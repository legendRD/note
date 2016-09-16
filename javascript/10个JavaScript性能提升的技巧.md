Nicholas Zakas是一位 JS 大师，Yahoo! 首页的前端主程。他是《高性能 Javascript》的作者，这本书值得每个程序员去阅读。

当谈到 JS 性能的时候，Zakas差不多就是你要找的，2010年六月他在Google Tech Talk发表了名为《Speed Up Your Javascript》的演讲。

但 Javascript 性能优化绝不是一种书面的技术，Nicholas 的技术演进列出了10条建议，帮助你写出高效的 JS 代码。

1. 定义局部变量

当一个变量被引用的时候，JavaScript将在作用域链中的不同成员中查找这个变量。作用域链指的是当前作用于下可用变量的集合，它在各种主流浏览器中至少包含两个部分：局部变量的集合和全局变量的集合。

简单地说，如果JavaScript引擎在作用域链中搜索的深度越大，那么操作也就会消耗更多的时间。引擎首先从 this 开始查找局部变量，然后是函数参数、本地定义的变量，最后遍历所有的全局变量。

因为局部变量在这条链的起端，所以查找局部变量总是比查找全局变量要块。所以当你想要不止一次地使用一个全局变量的时候，你应该将它定义成局部变量，就像这样：

var blah = document.getElementById('myID'),
blah2 = document.getElementById('myID2');

改写成

var doc = document,
blah = doc.getElementById('myID'),
blah2 = doc.getElementById('myID2');

2. 不要使用 with() 语句

这是因为 with() 语句将会在作用域链的开始添加额外的变量。额外的变量意味着，当任何变量需要被访问的时候，JavaScript引擎都需要先扫描with()语句产生的变量，然后才是局部变量，最后是全局变量。 So with() essentially gives local variables all the performance drawbacks of global ones, and in turn derails Javascript optimization. 因此with()语句同时给局部变量和全局变量的性能带来负面影响，最终使我们优化JavaScript性能的计划破产。

3. 小心使用闭包

虽然你可能还不知道“闭包”，但你可能在不经意间经常使用这项技术。闭包基本上被认为是JavaScript中的new，当我们定义一个即时函数的时候，我们就使用了闭包，比如：

document.getElementById('foo').onclick = function(ev) { };

闭包的问题在于：根据定义，在它们的作用域链中至少有三个对象：闭包变量、局部变量和全局变量。这些额外的对象将会导致第1和第2个建议中提到的性能问题。

但是我认为Nicholas并不是要我们因噎废食，闭包对于提高代码可读性等方面还是非常有用的，只是不要滥用它们（尤其在循环中）。

4. 对象属性和数组元素的速度都比变量慢

谈到JavaScript的数据，一般来说有4种访问方式：数值、变量、对象属性和数组元素。在考虑优化时，数值和变量的性能差不多，并且速度显著优于对象属性和数组元素。

因此当你多次引用一个对象属性或者数组元素的时候，你可以通过定义一个变量来获得性能提升。（这一条在读、写数据时都有效）

虽然这条规则在绝大多数情况下是正确的，但是Firefox在优化数组索引上做了一些有意思的工作，能够让它的实际性能优于变量。但是考虑到数组元素在其他浏览器上的性能弊端，还是应该尽量避免数组查找，除非你真的只针对于火狐浏览器的性能而进行开发。

5. 不要在数组中挖得太深

另外，程序员应该避免在数组中挖得太深，因为进入的层数越多，操作速度就越慢。

简单地说，在嵌套很多层的数组中操作很慢是因为数组元素的查找速度很慢。试想如果操作嵌套三层的数组元素，就要执行三次数组元素查找，而不是一次。

因此如果你不断地引用 foo.bar， 你可以通过定义 var bar = foo.bar 来提高性能。

6. 避免 for-in 循环（和基于函数的迭代）

这是另一条非常教条的建议：不要使用for-in循环。

这背后的逻辑非常直接：要遍历一个集合内的元素，你可以使用诸如for循环、或者do-while循环来替代for-in循环，for-in循环不仅仅可能需要遍历额外的数组项，还需要更多的时间。

为了遍历这些元素，JavaScript需要为每一个元素建立一个函数，这种基于函数的迭代带来了一系列性能问题：额外的函数引入了函数对象被创建和销毁的上下文，将会在作用域链的顶端增加额外的元素。

7. 在循环时将控制条件和控制变量合并起来

提到性能，在循环中需要避免的工作一直是个热门话题，因为循环会被重复执行很多次。所以如果有性能优化的需求，先对循环开刀有可能会获得最明显的性能提升。

一种优化循环的方法是在定义循环的时候，将控制条件和控制变量合并起来，下面是一个没有将他们合并起来的例子：

for ( var x = 0; x < 10; x++ ) {
};

当我们要添加什么东西到这个循环之前，我们发现有几个操作在每次迭代都会出现。JavaScript引擎需要：

#1：检查 x 是否存在
#2：检查 x 是否小于 0 <span style="color: #888888;">（译者注：我猜这里是作者的笔误）</span>
#3：使 x 增加 1

然而如果你只是迭代元素中的一些元素，那么你可以使用while循环进行轮转来替代上面这种操作：

var x = 9;
do { } while( x-- );

如果你想更深入地了解循环的性能，Zakas提供了一种高级的循环优化技巧，使用异步进行循环（碉堡了！）

8. 为HTML集合对象定义数组

JavaScript使用了大量的HTML集合对象，比如 document.forms，document.images 等等。通常他们被诸如 getElementsByTagName、getElementByClassName 等方法调用。

由于大量的DOM selection操作，HTML集合对象相当的慢，而且还会带来很多额外的问题。正如DOM标准中所定义的那样：“HTML集合是一个虚拟存在，意味着当底层文档被改变时，它们将自动更新。”这太可怕了！

尽管集合对象看起来跟数组很像，他们在某些地方却区别很大，比如对于特定查询的结果。当对象被访问进行读写时，查询需要重新执行来更新所有与对象相关的组分，比如 length。

HTML集合对象也非常的慢，Nicholas说好像在看球的时候对一个小动作进行60倍速慢放。另外，集合对象也有可能造成死循环，比如下面的例子：

var divs = document.getElementsByTagName('div');
for (var i=0; i < divs.length; i++ ) {
   var div = document.createElement("div");
   document.appendChild(div);
}

这段代码造成了死循环，因为 divs 表示一个实时的HTML集合，并不是你所期望的数组。这种实时的集合在添加 <div> 标签时被更新，所以i < div.length 永远都不会结束。

解决这个问题的方法是将这些元素定义成数组，相比只设置 var divs = document.getElementsByTagName(‘div’) 稍微有点麻烦，下面是Zakas提供的强制使用数组的代码：

function array(items) {
   try {
       return Array.prototype.concat.call(items);
   } catch (ex) {
       var i       = 0,
           len     = items.length,
           result  = Array(len);
       while (i < len) {
           result[i] = items[i];
           i++;
       }
       return result;
   }
}
var divs = array( document.getElementsByTagName('div') );
for (var i=0l i < divs.length; i++ ) {
   var div = document.createElement("div");
   document.appendChild(div);
}

9. 不要碰DOM！

不使用DOM是JavaScript优化中另一个很大的话题。经典的例子是添加一系列的列表项：如果你把每个列表项分别加到DOM中，肯定会比一次性加入所有列表项到DOM中要慢。这是因为DOM操作开销很大。

Zakas对这个进行了细致的讲解，解释了由于回流（reflow）的存在，DOM操作是非常消耗资源的。回流通常被理解为浏览器重新选渲染DOM树的处理过程。比如说，如果你用JavaScript语句改变了一个div的宽度，浏览器需要重绘页面来适应变化。

任何时候只要有元素被添加到DOM树或者从DOM树移除，都会引发回流。使用一个非常方便的JavaScript对象可以解决这个问题——documentFragment，我并没有使用过，但是在Steve Souders也表示同意这种做法之后我感觉更加肯定了。

DocumentFragment 基本上是一种浏览器以非可视方式实现的类似文档的片段，非可视化的表现形式带来了很多优点，最主要的是你可以在 documentFragment 中添加任何结点而不会引起浏览器回流。

10. 修改CSS类，而不是样式

你也许听说过：修改CSS类必直接修改样式会更高效。这归结于回流带来的另一个问题：当布局样式发生改变时，会引发回流。

布局样式意味着任何影响改变布局的变化都会强制引起浏览器回流。比如宽度、高度、字号、浮动等。

但是别误会我的意思，CSS类并不会避免回流，但是可以将它的影响最小化。相比每次修改样式都会引起回流，使用CSS类一次修改多个样式，只需要承担一次回流带来的消耗。

因此在修改多个布局样式的时候，使用CSS类来优化性能是明智的选择。另外如果你需要在运行时定义很多歌CSS类，在DOM上添加样式结点也是不错的选择。

总结

Nicholas C. Zakas 是JavaScript界的权威。在写这篇文章的时候，我发现我引用的很多文章也是他写的——因为太难找到其他更好的文章。

Zakas的技术演进非常棒，他解释了很多JavaScript优化规则的原因，我已奉为圣经。
