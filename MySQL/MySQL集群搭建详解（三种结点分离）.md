本文将搭建一个最简化的MySQL Cluster系统，配置方法中的所有命令都是以root账户运行。这个MySQL Cluster包含一个管理结点、两个数据结点、两个SQL结点，这五个结点会分别安装在五个虚拟机上，虚拟机的名称和IP如下所示：
管理结点
mysql-mgm
192.168.124.141
数据结点 1
mysql-ndbd-1
192.168.124.142
数据结点 2
mysql-ndbd-2
192.168.124.143
SQL 结点1
mysql-sql-1
192.168.124.144
SQL 结点2
mysql-sql-2
192.168.124.145
  
    
一、公共配置
请在三个虚拟机上分别配置此处的配置项。

1. 安装虚拟机
虚拟机操作系统安装CentOS 6.4的x86_64版本，使用NAT网络，并且还要安装vmware-tools，具体安装方法此处不详述。

2. 拷贝mysql cluster
下载以下版本的MySQL-Cluster：
http://cdn.mysql.com/Downloads/MySQL-Cluster-7.3/mysql-cluster-gpl-7.3.4-linux-glibc2.5-x86_64.tar.gz

下载得到的压缩包拷贝至虚拟机的/root/Downloads目录，然后在shell中运行以下命令：
cd /root/Downloads
tar -xvzf mysql-cluster-gpl-7.3.4-linux-glibc2.5-x86_64.tar.gz
mv mysql-cluster-gpl-7.3.4-linux-glibc2.5-x86_64 /usr/local/mysql

3. 关闭安全策略
关闭iptables防火墙（或者打开防火墙的1186、3306端口），在Shell中运行以下命令：
chkconfig --level 35 iptables off

关闭SELinux，在Shell中运行以下命令：
gedit /etc/selinux/config

将config文件中的SELINUX项改为disabled，修改后的config文件的内容如下：

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

最后重启系统

二、配置管理结点（192.168.124.141）
1. 配置config.ini配置文件
在shell中运行以下命令：
mkdir /var/lib/mysql-cluster
cd /var/lib/mysql-cluster
gedit config.ini


配置文件config.ini内容如下：
[ndbd default]
NoOfReplicas=2
DataMemory=80M
IndexMemory=18M

[ndb_mgmd]
NodeId=1
hostname=192.168.124.141
datadir=/var/lib/mysql-cluster

[ndbd]
NodeId=2
hostname=192.168.124.142
datadir=/usr/local/mysql/data

[ndbd]
NodeId=3
hostname=192.168.124.143
datadir=/usr/local/mysql/data

[mysqld]
NodeId=4
hostname=192.168.124.144

[mysqld]
NodeId=5
hostname=192.168.124.145

2. 安装管理结点
安装管理节点，不需要mysqld二进制文件，只需要MySQL Cluster服务端程序(ndb_mgmd)和监听客户端程序(ndb_mgm)。在shell中运行以下命令：
cp /usr/local/mysql/bin/ndb_mgm* /usr/local/bin
cd /usr/local/bin
chmod +x ndb_mgm*

三、配置数据结点（192.168.124.142、192.168.124.143）
1. 添加mysql组和用户
在shell中运行以下命令：
groupadd mysql
useradd -g mysql mysql

2. 配置my.cnf配置文件
在shell中运行以下命令：
gedit /etc/my.cnf

配置文件my.cnf的内容如下：
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/sock/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

[mysql_cluster]
ndb-connectstring=192.168.124.141

3. 创建系统数据库
在shell中运行以下命令：
cd /usr/local/mysql
mkdir sock
scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

4. 设置数据目录
在shell中运行以下命令：
chown -R root .
chown -R mysql.mysql /usr/local/mysql/data
chown -R mysql.mysql /usr/local/mysql/sock
chgrp -R mysql .

5. 配置MySQL服务
在shell中运行以下命令：
cp support-files/mysql.server /etc/rc.d/init.d/
chmod +x /etc/rc.d/init.d/mysql.server
chkconfig --add mysql.server

四、配置SQL结点（192.168.124.144、192.168.124.145）
1. 添加mysql组和用户
在shell中运行以下命令：
groupadd mysql
useradd -g mysql mysql

2. 配置my.cnf配置文件
在shell中运行以下命令：
gedit /etc/my.cnf

配置文件my.cnf的内容如下：
[client]
socket=/usr/local/mysql/sock/mysql.sock

[mysqld]
ndbcluster
datadir=/usr/local/mysql/data
socket=/usr/local/mysql/sock/mysql.sock
ndb-connectstring=192.168.124.141

[mysql_cluster]
ndb-connectstring=192.168.124.141

3. 创建系统数据库
在shell中运行以下命令：
cd /usr/local/mysql
mkdir sock
scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data

4. 设置数据目录
在shell中运行以下命令：
chown -R root .
chown -R mysql.mysql /usr/local/mysql/data
chown -R mysql.mysql /usr/local/mysql/sock
chgrp -R mysql .

5. 配置MySQL服务
在shell中运行以下命令：
cp support-files/mysql.server /etc/rc.d/init.d/
chmod +x /etc/rc.d/init.d/mysql.server
chkconfig --add mysql.server

五、Cluster环境启动
注意启动顺序：首先是管理节点，然后是数据节点，最后是SQL节点。

1. 启动管理结点
在shell中运行以下命令：
ndb_mgmd -f /var/lib/mysql-cluster/config.ini

还可以使用ndb_mgm来监听客户端，如下：
ndb_mgm

2. 启动数据结点
首次启动，则需要添加--initial参数，以便进行NDB节点的初始化工作。在以后的启动过程中，则是不能添加该参数的，否则ndbd程序会清除在之前建立的所有用于恢复的数据文件和日志文件。
/usr/local/mysql/bin/ndbd --initial

如果不是首次启动，则执行下面的命令。
/usr/local/mysql/bin/ndbd

3. 启动SQL结点
若MySQL服务没有运行，则在shell中运行以下命令：
/usr/local/mysql/bin/mysqld_safe --user=mysql &

4. 启动测试
查看管理节点，启动成功：

六、集群测试
1. 测试一
现在我们在其中一个SQL结点上进行相关数据库的创建,然后到另外一个SQL结点上看看数据是否同步。

在SQL结点1（192.168.124.144）上执行：
shell> /usr/local/mysql/bin/mysql -u root -p
mysql>show databases;
mysql>create database aa;
mysql>use aa;
mysql>CREATE TABLE ctest2 (i INT) ENGINE=NDB; //这里必须指定数据库表的引擎为NDB,否则同步失败
mysql> INSERT INTO ctest2 () VALUES (1);
mysql> SELECT * FROM ctest2;

然后在SQL结点2上看数据是否同步过来了

经过测试，在非master上创建数据，可以同步到master上
查看表的引擎是不是NDB，>show create table 表名；

2. 测试二
关闭一个数据节点 ，在另外一个节点写输入，开启关闭的节点，看数据是否同步过来。

首先把数据结点1重启，然后在结点2上添加数据
在SQL结点2(192.168.124.145)上操作如下：
mysql> create database bb;
mysql> use bb;
mysql> CREATE TABLE ctest3 (i INT) ENGINE=NDB;
mysql> use aa;
mysql> INSERT INTO ctest2 () VALUES (3333);
mysql> SELECT * FROM ctest2;

等数据结点1启动完毕，启动数据结点1的服务
#/usr/local/mysql/bin/ndbd --initial#service mysqld start


然后登录进去查看数据
# /usr/local/mysql/bin/mysql -u root –p


可以看到数据已经同步过来了，说明数据可以双向同步了。

七、关闭集群
1. 关闭管理节点和数据节点，只需要在管理节点（ClusterMgm--134）里执行：
shell> /usr/local/mysql/bin/ndb_mgm -e shutdown

显示
Connected to Management Server at: localhost:1186
2 NDB Cluster node(s) have shutdown.
Disconnecting to allow management server to shutdown.

2. 然后关闭Sql节点（135,136），分别在2个节点里运行：
shell> /etc/init.d/mysql.server stop
Shutting down MySQL... SUCCESS!

注意：要再次启动集群，就按照第五部分的启动步骤即可，不过这次启动数据节点的时候就不要加”-initial”参数了。
