# 4.10 用hitch和partial生成函数

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/hitch/index.html

----------

`dojo/_base/lang`包含了在JavaScript中使用函数的辅助方法。在这篇教程中，你将了解函数对象的基础——如何使用`lang.hitch`为函数绑定上下文。从那里，你将了解如何使用`lang.partial`来给函数绑定特定参数，以及`lang.hitch`如何结合这两种操作。

##入门
关于这篇教程，我们假设你已经有Dojo Toolkit 构建的基础知识，如[dojo/query](https://dojotoolkit.org/documentation/tutorials/1.10/using_query/)和其他基于语言的帮助函数如Dojo的[Array helpers](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/)。

在我们理解何时及怎样使用`lang.hitch`和`lang.partial`之前，我们必须先理解它们解决什么样的问题。JavaScript最常被误解的概念就体现在简单、频繁的提问里，“What is `this`”？通常，在面向对象的编程中，当调用一个对象的方法时，我们希望‘this’指代的是该对象。但是这个答案在JavaScript里很微妙，为了牢牢掌握它我们需要理解“执行上下文”。

### JavaScript执行上下文
在JavaScript里无论何时调用一个函数，都将创建一个执行上下文（更多细节见 [this article from Tuenti](http://blog.tuenti.com/dev/functions-and-execution-contexts-in-javascript-2/)）。上下文经由以下过程创建：

 - 创建**参数**对象；
 - 创建函数**作用域**；
 - 初始化函数的**变量**；
 - 创建*this*属性（为上下文本身）。

开发者最糊涂的是this属性；它是对调用函数的上下文（作用域）的对象的引用。理解这个是理解JavaScript如何工作的关键，因为在JavaScript中，函数执行的实际上下文是在函数调用的时候决定的。

>JavaScript里作用域常常混淆不清，一方面，它表示调用或执行的东西下的对象，另一方面它又表示定义的东西下的对象。后者称为词法作用域，它才是JavaScript里真正的作用域。词法作用域下可以实现[闭包](http://en.wikipedia.org/wiki/Closure_%28computer_science)编程技术；可以看看Richard Cornford的这篇[文章](http://www.jibbering.com/faq/notes/closures/)参考。调用时作用域就是JavaScript的执行上下文。

让我们先看一个常见的例子。我们有一个对象，还有一个该对象的方法，它用来作为文档中若干节点的事件处理器。我们可以这么定义它：

```
// Require the query resource, and wait until the DOM is ready
require(["dojo/query", "dojo/domReady!"],
    function(query) {

        var myObject = {
            foo: "bar",
            myHandler: function(evt){
                //    this is very contrived but will do.
                alert("The value of 'foo' is " + this.foo);
            }
        };

        //    later on in the script:
        query(".myNodes").forEach(function(node){
            node.onclick = myObject.myHandler;
        });

});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hitch/demo/demo.html)

当点击带有“myNodes”CSS类的任一节点时，你期望上面的函数显示一个JavaScript警告框信息“The value of 'foo' is bar”；但是由于我们设置myObject.myHandler 的方式，对每个节点的事件处理器我们将得到“**The value of 'foo' is undefined**”。我们来看看为什么会这样：

 - `node.onclick = myObject.myHandler`
	 - `myObject.myHandler`表达式的结果为一个函数——`myHandler`函数。但是，实际上`myHandler`作为定义在`myObject`上的方法被废弃了。
	 - 所以现在`node.onclick`指向函数 `myHandler`——只是这个函数，没有`myObject`上下文。
	 - DOM事件处理器运行在触发事件节点的上下文，就是说函数作为节点的一个方法执行，忽略了它本身在哪定义，结果就是运行时函数中`this`的值是这个节点。

>如果你觉得晕头转向，记住原因是函数对象——像其他JavaScript非基本类型一样，是通过*引用*传递的而不是值传递；在我们上面的例子中，我们将节点的onclick方法设置为` myObject.myHandler`的直接引用。记住：JavaScript的函数是一阶对象，可以看做和JavaScript的其他对象一样——包括可以作为一个参数传递给另一个函数。

### 使用.apply和.call转换执行上下文
由于JavaScript在调用函数时能够定义执行的上下文，这个语言通过`Function.apply`和`Function.call`提供改变上下文的方式——也就是this的含义。简而言之，这两个方法都允许你执行一个函数，这个函数可接受一个对象作为作为其上下文。例如，如果我们想要确保上面的处理器执行在myObject的上下文中，就要用`Function.call`方法来包裹我们的引用，像这样：

```
query(".myNodes").forEach(function(node){
    node.onclick = function(evt){
        myObject.myHandler.call(myObject, evt);
    };
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hitch/demo/call.html)

>网上大多数的例子都使用`Function.apply`，通常从外部函数传递参数对象。但是，如果提前知道函数的参数，我们推荐使用call；它在JavaScript编译器没有直接获得参数对象时会有少量的性能提升。我们也可以直接调用 myObject.myHandler，但是使用`.call`形式可以实时展示 myHandler 设置的上下文。

现在我们已经回顾了JavaScript执行上下文的基础，让我们继续了解Dojo Toolkit如何通过`lang.hitch`简化这个过程。

##使用lang.hitch绑定执行上下文
 Dojo Toolkit 通过[`lang.hitch`](https://dojotoolkit.org/api/?qs=1.10/dojo/_base/lang)提供简化的绑定上下文和函数的方式。简单来说，`lang.hitch`创建一个绑定（或攀上）指定上下文的新函数对象，你随后可以安全的调用它，不用担心上下文的改变。用`lang.hitch`很简单：
 
```
// `foo` is intentionally global
var foo = "bar";
require(["dojo/_base/lang"],
    function(lang) {

        var myFunction = function(){
            return this.foo;
        };
        var myObject = { foo: "baz" };

        // later on in your application
        var boundFunction = lang.hitch(myObject, myFunction);

        // the first value will be "bar", the second will be "baz";
        // the third will still be "bar".
        myFunction();        // "bar"
        boundFunction();    // "baz"
        myFunction();        // "bar"

});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hitch/demo/hitch.html)	 

如你所见，`lang.hitch`确保一个绑定到特定执行上下文的特定函数在调用时上下文不会被切换掉。

##参数对象
还记得当我们解释的创建执行上下文的过程么？第一步是创建参数对象。该对象是一个数组式的对象，将有序list的值传递给一个函数。另外，在第三步创建函数的全部变量——包含任何已命名的参数，这样这些值可以通过名称获得，就像它们只是函数体内的另一个变量。

记住参数对象不是一个真正的JavaScript数组对象；虽然它共享一些相同的aspects（如通过数字访问成员和length属性），它是只读的，也就是说数组的其它方法（如`Array.prototype.slice`）是不适用的。

当定义一个函数时，函数的签名是固定的。你不能添加或移除已命名的参数，除非重新定义函数本身。这有时会成为一个问题，特别是当你想要在不复制或者重写原始函数的情况下，匹配一个函数签名时（一个在一个Dojo Toolkit这样的库里的预定义处理器）。Dojo Toolkit通过`lang.partial`方法提供一个简单的方式来完成。

##lang.partial改变函数签名
开发者会面临的一个问题是有一个多参数的函数，但使用时指需要一小部分的参数。例如，我们有一个4个参数的函数，像这样（我们将为本例使用`dojo/data`）：

```
var putValue = function(store, item, attr, value){
    return store.setValue(item, attr, value);
}
```
但是在你应用的某些地方，另一个开发者（或者库）编写了一组对象，它们调用类似的处理器但只有3个参数：

```
someObject.setValueHandler = function(item, attr, value){
    //    placeholder function to be overridden
};
```
你可以使用`lang.partial`创建一个新的（未绑定）函数，它使用预设值参数（或绑定参数）。为了完成上面的例子，我们打算使用一个特定store的引用来预设store参数，然后将`someObject.setValueHandler`设为我们partial（局部）函数的引用，如下;

```
// assuming we have a dojo/data store called "myStore"

// our function
var putValue = function(store, item, attr, value){
    return store.setValue(item, attr, value);
}

// ...
// their function signature
someObject.setValueHandler = function(item, attr, value){
    //    placeholder function to be overridden
};

// ...
// our solution using lang.partial
someObject.setValueHandler = lang.partial(putValue, myStore);

// ...
// somewhere in the application when setValueHandler is invoked,
// our putValue function will already have the "store" arg
// set to a reference to "myStore"
someObject.setValueHandler(someItem, "foo", "bar");
```
>[View demo](https://dojotoolkit.org/documentation/tutorials/1.10/hitch/demo/partial.html)

上面可能比较乱，让我们来分解一下：

 1. 我们定义4个参数的putValue函数；
 2. 我们发现setValueHandler设计为接收3个参数，并且（为了例子）不能改变它；
 3. 我们基础putValue创建一个新函数，它的第一个参数store预设为myStore；
 4. 随后调用新的局部函数，只需要传递3个参数，但是我们的局部函数已经将myStore设置为第一个参数。

需要重点注意的是不像`lang.hitch`，`lang.partial`**不为返回的局部函数设置执行作用域**。也就是说，this关键字的含义可以改变，它依赖于你如何使用新的局部函数。

你可以用`lang.partial`来实现的有趣之处是你可以将引用预设为你要用的对象，这样你就可以让上下文变化同时又拥有一个通过函数参数绑定的引用。

##hitch和partial最佳结合
那么当你既想要`hitch`的优点（强制一个执行上下文），又想要`partial`的有点（预设参数）呢？其实`lang.hitch`就可以这样，你可以在上下文和方法参数之后包含任意数量的值，`lang.hitch`将用绑定的上下文和预设参数来装配新函数。来看一个和上面相似的例子：

```
someObject.setValueHandler = lang.hitch(someObject, putValue, myStore);

// ...
// later on in the application, the setHandler is invoked
// again--this time in the context of someObject
someObject.setValueHandler(someItem, "foo", "bar");
```
`hitch`和`partial`是通往[函数式编程](http://en.wikipedia.org/wiki/Functional_programming)技术的大门；Dojo Toolkit通过` dojox/lang/functional`命名空间提供需要函数式编程的技术，我们鼓励你多去了解一些。

##小结
在本教程中，我们回顾了JavaScript函数对象，包括一个函数的调用过程。然后引入了`lang.hitch`，你可以用它将函数绑定到一个指定的执行上下文。随后我们了解了如何使用`lang.partial`将参数绑定到函数，最后展示了如何使用`lang.hitch`同时绑定上下文和参数。

`lang.hitch`对于事件驱动的编程非常有用（也叫基于回调的编程），它可以让你预设函数的执行上下文，而不用担心会改变this关键字。

不要忘了如果你需要Dojo Toolkit的相关帮助，你可以 进入irc.freenode.net的[ #dojo IRC channel](https://dojotoolkit.org/chat) 。不要忘记我们这个频道的座右铭：Don't Ask To Ask, Just Ask®!