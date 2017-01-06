# 3.3 Dojo事件

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/events/index.html

----------

本教程中，我们将探索`dojo/on`以及Dojo如何更容易地关联DOM事件。我们也将探索Dojo的publish/subscribe框架：`dojo/topic`。

##入门

你许多的JavaScript代码都用来为事件做准备：包括生成和响应新事件。就是说构建响应式、交互式web应用的关键就是创建高效的事件连接。事件连接可以让你的应用响应用户交互和等待动作的发生。Dojo中DOM事件的主力是`dojo/on`，下面一起来看如何使用这个模块。

## DOM事件

你会问自己“DOM不是已经提供了一个机制来注册事件处理程序么？” 答案是yes，但并不是所有的浏览器都遵循 W3C DOM 规范。在各种主流浏览器里主要有三种方式来注册事件处理器：`addEventListener`、`attachEvent`和 DOM 0。除此之外，还有两个不同的事件对象实现和至少一种 [fires registered listeners in random order](http://msdn.microsoft.com/en-us/library/ms536343%28v=vs.85).aspx) 和[leaks memory](http://msdn.microsoft.com/en-us/library/bb250448%28VS.85).aspx)的浏览器引擎。Dojo改善了你使用DOM事件的方式，用一个名叫`dojo/on`的单一、直白的事件API将各种原生API标准化并阻止内存泄漏。

假设我们有以下标记：

```
<button id="myButton">Click me!</button>
<div id="myDiv">Hover over me!</div>
```

再想象一下，你想要在点击按钮的时候让`div`变成蓝色，鼠标经过时变为红色，完成悬停后变回白色。`dojo/on`很容易实现：

```
require(["dojo/on", "dojo/dom", "dojo/dom-style", "dojo/mouse", "dojo/domReady!"],
    function(on, dom, domStyle, mouse) {
        var myButton = dom.byId("myButton"),
            myDiv = dom.byId("myDiv");

        on(myButton, "click", function(evt){
            domStyle.set(myDiv, "backgroundColor", "blue");
        });
        on(myDiv, mouse.enter, function(evt){
            domStyle.set(myDiv, "backgroundColor", "red");
        });
        on(myDiv, mouse.leave, function(evt){
            domStyle.set(myDiv, "backgroundColor", "");
        });
});
```

> 注意我们还require了`dojo/mouse`。不是所有的浏览器原生支持`mouseenter`和`mouseleave`事件，所以用`dojo/mouse`来添加支持。你可以自己编写像`dojo/mouse`一样的模块来支持其他自定义事件类型。

这个例子阐述了常见模式： `on(`element`,`event name`,`handler`)`。或者换个说话就是某个元素上的某个事件连接某个处理器。该模式适用于全部的window、document、node、 form、mouse和keyboard事件。

> 注意和旧的`dojo.connect`API不一样，使用`dojo/on`模块时，事件名称的“on”前缀必须省略。
`on`方法不仅仅规范了事件注册的API，还规范了事件处理器的工作方式：

 - 事件处理器总是按注册的顺序调用。
 - 它们调用时都将一个事件对象作为第一个参数，
 - 事件对象总是规范的包含常见W3C事件对象属性、类似`target`属性、一个`stopPropagation`方法和一个`preventDefault`方法。

就像DOM API，Dojo提供一个方法来移除一个事件处理器：`_handle_.remove`。`on`的返回值是一个带有`remove`方法的简单对象，调用该方法是将移除事件监听。例如，如果你想要一个一次性的事件，你就可以这么做：

```
var handle = on(myButton, "click", function(evt){
    // 使用处理器移除该事件
    handle.remove();

    // Do other stuff here that you only want to happen one time
    alert("This alert will only happen one time.");
});
```
> 顺便提下，`dojo/on`还有一个简便的方法：`on.once`。它接收和`on`一样的参数，但是会在它触发一次之后移除处理程序。

最后要说明的是：默认情况下，`on`将会在第一个参数节点的上下文中运行事件处理器。例外是当它用来事件委托时，稍后就会讨论。

无论如何，你可以使用`lang.hitch`（来自`dojo/_base/lang`模块）来指定运行处理器的上下文。当使用对象方法时，hitch非常有用：

```
require(["dojo/on", "dojo/dom", "dojo/_base/lang", "dojo/domReady!"],
    function(on, dom, lang) {

        var myScopedButton1 = dom.byId("myScopedButton1"),
            myScopedButton2 = dom.byId("myScopedButton2"),
            myObject = {
                id: "myObject",
                onClick: function(evt){
                    alert("The scope of this handler is " + this.id);
                }
            };

        // 这个将弹出 "myScopedButton1"
        on(myScopedButton1, "click", myObject.onClick);
        // 这个将弹出 "myObject" 而不是 "myScopedButton2"
        on(myScopedButton2, "click", lang.hitch(myObject, "onClick"));

});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/events/demo/on.html)

>不像它的前任`dojo.connect`，`on`不接受处理器作用域和方法参数。你需要为第三个参数使用`lang.hitch`，如果你希望保持执行语境。

## NodeList事件
前面的教程提到过，`NodeList`提供一种多节点注册事件的方式：`on`方法。这个方法遵循`dojo/on`相同的模式，但没有第一个参数（由于`NodeList`的节点就是你将要连接的对象）。

> `on`方法包含在`dojo/query`里，所以你用`NodeList.on`时不需要专门require`dojo/on`。

让我们看一个更高级的例子：

```
<button id="button1" class="clickMe">Click me</button>
<button id="button2" class="clickMeAlso">Click me also</button>
<button id="button3" class="clickMe">Click me too</button>
<button id="button4" class="clickMeAlso">Please click me</button>
<script>
require(["dojo/query", "dojo/_base/lang", "dojo/domReady!"],
    function(query, lang) {

        var myObject = {
            id: "myObject",
            onClick: function(evt){
                alert("The scope of this handler is " + this.id);
            }
        };
        query(".clickMe").on("click", myObject.onClick);
        query(".clickMeAlso").on("click", lang.hitch(myObject, "onClick"));

});
</script>
```
> 记住不同于`NodeList.connect`，`NodeList.on`方法并不返回NodeList，反之，它返回一个稍后可`remove`的`on`处理器的数组。 这个数组也包含一个便利的最高阶remove方法，可以一次移除全部的监听器。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/events/demo/query.html)

## 事件委托

上面讨论过，NodeList的on方法可以很容易的把相同事件的同一处理器连接到多个DOM节点上。dojo/on也可以通过更高效的事件委托来实现这个。

事件委托背后的理念不是对每一个独立兴趣节点绑定一个事件监听，而是给一个节点在更高层面上单独附加一个监听器，它会检查捕获的事件的目标，看这些目标是否是从真实的兴趣节点冒泡来的，如果是则执行处理器逻辑。

以前这个可以（现在仍可以）通过`dojox/NodeList/delegate`来扩展`NodeList`。在1.10中就可以用`dojo/on`模块，语法是`on(`parent enement`, "`selector` : `event name `",` handler`)`。

为了更好的说明，来看另一个基于和之前相同premise的例子：

```
<div id="parentDiv">
    <button id="button1" class="clickMe">Click me</button>
    <button id="button2" class="clickMe">Click me also</button>
    <button id="button3" class="clickMe">Click me too</button>
    <button id="button4" class="clickMe">Please click me</button>
</div>
<script>
require(["dojo/on", "dojo/dom", "dojo/query", "dojo/domReady!"],
    function(on, dom){

        var myObject = {
            id: "myObject",
            onClick: function(evt){
                alert("The scope of this handler is " + this.id);
            }
        };
        var div = dom.byId("parentDiv");
        on(div, ".clickMe:click", myObject.onClick);

});
</script>
```
> 请注意，我们也需要`require` `dojo/query`模块，虽然并不直接使用它。因为`dojo/on`需要`dojo/query`提供的选择器引擎来实现事件委托的选择器匹配。为了减少封装和避免不经常使用的特性困扰开发者，它没有被`dojo/on`自动引入。

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/events/demo/delegation.html)

在运行上面的例子时，注意`this`仍然引用实际的兴趣节点，不是`parentDiv`节点。这是使用on进行事件委托的重要区别：this不再引用第一个参数传入的节点，而是选择器匹配的节点。这会非常有用。

##对象方法

`dojo/on`的前任——`dojo.connect`曾经也可实现 function-to-function的事件连接。这个功能已经划分到了名叫`dojo/aspect`中。你很快会在后面的`dojo/aspect`的教程中看到。

##Publish/Subscribe

目前为止的所有例子都是用已存在的对象作为一个事件产生者，当发生一些事情时注册的这个对象将会发觉。不过，如果你没有节点的处理器或者不知道一个对象是否已经创建时应该怎么办？这里引入Dojo的发布和订阅（pub/sub）框架，它在1.10版本的`dojo/topic`模块里。你可以用pub/sub为“topic”（这是一个有趣的事件名称，可以有多种资源，表示为一个字符串）注册一个处理器（订阅）。

想象一下在我们正在开发的一个应用中，我们想要确保按钮会在用户的动作下弹出信息，但是不打算多次编写程序来弹出信息，也不想要创建一个包装对象来在这个按钮上注册这个小程序。这里可以使用pub/sub ：

```
<button id="alertButton">Alert the user</button>
<button id="createAlert">Create another alert button</button>

<script>
require(["dojo/on", "dojo/topic", "dojo/dom-construct", "dojo/dom", "dojo/domReady!"],
    function(on, topic, domConstruct, dom) {

        var alertButton = dom.byId("alertButton"),
            createAlert = dom.byId("createAlert");

        on(alertButton, "click", function() {
            // 当这个按钮被点击
            // 发布"alertUser" 主题
            topic.publish("alertUser", "I am alerting you.");
        });

        on(createAlert, "click", function(evt){
            // 创建另一个按钮
            var anotherButton = domConstruct.create("button", {
                innerHTML: "Another alert button"
            }, createAlert, "after");

            // 当另一个按钮被点击
            // 发布"alertUser" 主题
            on(anotherButton, "click", function(evt){
                topic.publish("alertUser", "I am also alerting you.");
            });
        });

        // 为"alertUser" 主题注册警告程序
        topic.subscribe("alertUser", function(text){
            alert(text);
        });

});
</script>
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/events/demo/pubsub.html)

这种模式的一个优势在于我们不需要创建任何DOM对象就可以在单元测试中测试我们的弹出程序：这个程序和事件产生者之间不挂钩。

如果你想要停止获取一个topic的通知，`topic.subscribe`返回一个带有`remove`方法的对象，这个方法可以用来移除单个处理器（就像`on`）。

> 注意和`dojo.publish`不同，`topic.publish`不希望通过数组来传递发布的参数。例如，`topic.publish("someTopic", "foo", "bar")`等同于`dojo.publish("someTopic", ["foo", "bar"])`。

##小结

Dojo事件虽然用起来很简单，但是非常强大。`on`方法可以将不同浏览器间DOM事件的差异规范化。Dojo的pub/sub框架、`dojo/topic`向开发者提供了将事件处理器与事件产生者分离开来的方法。花点时间来熟悉这些工具吧，它们在创建web应用时非常有用。
