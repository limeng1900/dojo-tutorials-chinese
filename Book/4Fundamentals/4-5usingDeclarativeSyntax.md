# 4.5使用声明式语法

原文地址：[https://dojotoolkit.org/documentation/tutorials/1.10/declarative/index.html](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/index.html)

---

最快开始使用Dojo的方式之一，尤其是使用Dijit这样的widget，是利用`dojo/parser`和声明式语法。本教程将帮你充分利用这种风格的编程。

##入门
使用Dojo时又两种编码方式。第一种叫做编程式，第二种叫声明式。编程式是只使用JavaScript来实例化你的对象，你所有的行为代码都在JavaScript里实现。声明式是使用`dojo/parser`来读取DOM并解析用特殊属性修饰的节点，以及解释某些`<script>`标签来扩展widget的行为。

这两种风格都各有利弊，有时你会看到开发者混合使用它们。这个教程着眼于提供对声明式语法的全部特性。如果你打算在你的项目中使用声明式语法，你需要考虑下面这些事：

 - 声明式语法非常简单，它不需要深入了解JavaScript。它的功能也变得相当丰富，你几乎可以用它实现在JavaScript里做的一切，但是通常也会受限制。
 - 因为它的工作本质，它的性能总是比编程式风格要差。这是由于`dojo/parser`必须解析DOM来查找它要处理的节点。
 - 它非常适合原型界面，在生产应用里你回发现它难以管理。

本教程主要利用Dojo1.10，教程中许多例子只适用于Dojo 1.8+。

##实例化对象
声明式编程的大部分使用方式是实例化widget。这种方式通过在你的标签添加一个特殊属性（`data-dojo-type`），然后用`dojo/parser`读取文档并实例化widget。例如以下标记将创建一个Dijit按钮：

```
<button type="button" data-dojo-type="dijit/form/Button">
    <span>Click Me!</span>
</button>
```
在上面的片段中，我们提供代表Dojo“类型”的Module ID（MID），它将告知`dojo/parser`于该节点在DOM的所处位置上实例化一个`dijit/form/Button`。

这很棒，不过如果我们未来想用按钮来做一些事，就需要获取到widget的引用。对于基于widget的Dijit，当它实例化以后，它将查找被它代替的节点。如果它找到`id`属性，就将在`dijit/registry`里用这个ID注册自己，这样我们将来就可以得到它的引用了。所以为了更好用应该这么改：

```
<button type="button" id="myButton" data-dojo-type="dijit/form/Button">
    <span>Click Me!</span>
</button>
```
当选择标签作为Dijit的占位符时，你应该使用最接近要用的widget的原生HTML标签。在这个例子中`dijit/form/Button`是一个按钮，所以我们使用`<button>`标签。这样你的应用可以更好地降低要求，它避免了在某些浏览器中替换节点时出现的一些问题。同样，你应该遵循HTML的最佳实践，就是说你应该给按钮分配一个`type`属性。

现在，我们要做的是调用`dojo/parser`。在采用Dojo 1.7的AMD之前，你可以安全的使用一个Dojo配置选项`parseOnLoad: true`。不过现在有一些边界情况，如果你那么做会出现一些意想不到的结果，因此现在建议直接在你的JavaScript代码里调用解析。你可以这样运行解析：

```
<script type="text/javascript" src="lib/dojo/dojo.js"
    data-dojo-config="async: true"></script>
<script type="text/javascript">
    require(["dojo/parser", "dijit/form/Button", "dojo/domReady!"],
    function(parser){
        parser.parse();
    });
</script>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/button.html)

注意我们require了`dijit/form/Button`，但在`require`的回调参数里忽略了它。因为我们并没有直接在代码块里引用它，但我们需要确保在调用`dojo/parser`之前加载了这个模块。其实解析是能自动加载的模块，我们随后会讨论这个，最好的明确你的需求项。

##配置对象
拥有`dojo/parser`实例对象是很棒，但是如果你不能配置widget，那它就没什么用了。所以你需要一种机制来给实例传递某些配置信息。

在早期版本的Dojo，我们在标签属性之后添加属性，甚至这些属性可能对标记无效。随着HTML5的引进，规范允许使用以`data-`开头的自定义属性，这样文档仍然严格有效。我们专门用`data-dojo-props`属性来包含需要传递给实例构造器的配置信息。

我们想要创建一个TabContainer，它有一个tab，里面包含一个按钮。用编程式风格的话，我们这么做：

```
require([
    "dijit/form/Button",
    "dijit/layout/TabContainer",
    "dijit/layout/ContentPane",
    "dojo/domReady!"
], function(Button, TabContainer, ContentPane){
    var tc = new TabContainer({
            style: {
                height: "200px",
                width: "400px"
            },
            id: "tc"
        }),
        atab = new ContentPane({
            title: "A Tab",
            closable: false,
            id: "atab"
        }),
        myButton = new Button({
            label: "Click Me!",
            id: "myButton"
        });
    atab.addChild(myButton);
    tc.addChild(atab);
    tc.startup();
});
```
声明式的话，我们的标签应该是这样：

```
<div id="tc" data-dojo-type="dijit/layout/TabContainer"
        data-dojo-props="style: { height: '200px', width: '400px' }">
    <div id="atab" data-dojo-type="dijit/layout/ContentPane"
            data-dojo-props="title: 'A Tab', closable: false">
        <button type="button" id="myButton"
                data-dojo-type="dijit/form/Button">
            <span>Click Me!</span>
        </button>
    </div>
</div>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/tab.html)

按照约定，`data-dojo-props`属性的值是一个JavaScript字面量，只是没有外面的大括号（{}）。

##Non-Widgets
`dojo/parser`通常用来实例化标签里的可视化元素（如Dijit Widgets），它也可以用来实例化其它不可见的对象。`dojo/parser`假设一个构造器，将它的配置信息作为第一个参数，传递节点的引用作为第二个参数。这对于基于`dojo/_base/declare`的不可见对象也很有用。

有一个“挑战”是正规对象没有类似于Dijit基于widget所做的注册机制，为了在实例化之后能够引用到它们，你必须在全局范围里创建它们的引用。`dojo/parser`通过查找`data-dojo-id`属性来实现。它的值将被设在全局范围。例如想要创建一个内存存储，要这么做：

```
<div data-dojo-id="myStore" data-dojo-type="dojo/store/Memory"></div>
```
例如，我们要创建一个内存存储来填充一个下拉列表，可以这么做：

```
<div data-dojo-id="myStore" data-dojo-type="dojo/store/Memory"
    data-dojo-props="data: [
        { name: 'Alabama', id: 'AL' },
        { name: 'Alaska', id: 'AK' },
        { name: 'Arizona', id: 'AZ' },
        { name: 'California', id: 'CA' },
        { name: 'Colorado', id: 'CO' },
        { name: 'Connecticut', id: 'CT' },
        { name: 'New York', id: 'NY' }
    ]"></div>
<select id="mySelect" name="state" value="CA"
    data-dojo-type="dijit/form/FilteringSelect"
    data-dojo-props="searchAttr: 'name', store: myStore"></select>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/nonwidget.html)

你需要知道一件事，因为这些不可见对象没有注册，并且引用是在全局范围，就不会发生垃圾回收，即便创建对象的DOM节点已经不在内存里了。这在大型应用里被视为内存泄漏。你应该确保不再使用这个变量时，将它从全局范围移除。

##更改行为
为了修改widget的行为，通常需要在时间发生时执行一些代码。例如在按钮点击时显示一个对话框，在JavaScript可能要这么做：

```
require(["dijit/form/Button", "dijit/Dialog"], function(Button, Dialog){
    var myButton = new Button({
            label: "Click Me!",
            id: "myButton"
        }, "myButton"),
        someDialog = new Dialog({
            title: "Hello World!",
            content: "<p>I am a dialog. That makes me happy.</p>"
        }, "someDialog");

    myButton.on("click", function(){
        someDialog.show();
    });

    myButton.startup();
    someDialog.startup();
});
```
为了全部在标签里面实现它们，我们要使用“声明式脚本”，它可以让`dojo/parser`获取行内代码段并附加到实例化的对象中。示例如下：

```
<div id="someDialog" data-dojo-type="dijit/Dialog"
        data-dojo-props="title: 'Hello World!'">
    <p>I am a dialog. That makes me happy.</p>
</div>
<button type="button" id="myButton" data-dojo-type="dijit/form/Button">
    <span>Click Me!</span>
    <script type="dojo/on" data-dojo-event="click">
        var registry = require("dijit/registry");
        registry.byId("someDialog").show();
    </script>
</button>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/scripting.html)

所有的声明式脚本只在它们自己的作用域里运行，为了获取模块的引用它们就得进入全局作用域，这就是为什么我们要用`require("dijit/registry");`来耍一些小聪明。我们随后会讨论这种声明式将模块加入全局作用域的方式。

在上面的例子中你可以看到使用声明式脚本很容易修改widget行为。当结合widget动态加载内容是它变得更加强大，如`dijit/layout/ContentPane`，因为它可以通过`dojo/parser`传递加载的内容，不仅仅实例化更多的widget，还可以设置它们的行为。

例如，如果我有一个`content.html`的文件，它的内容如下：

```
<button type="button" id="myButton" data-dojo-type="dijit/form/Button">
    <span>Click Me!</span>
    <script type="dojo/on" data-dojo-event="click">
        console.log("I was clicked!");
    </script>
</button>
```
我想要动态地把它加载到一个tab，我会这么做：

```
<div id="tc" data-dojo-type="dijit/layout/TabContainer"
        data-dojo-props="style: { height: '200px', width: '400px' }">
    <div id="atab" data-dojo-type="dijit/layout/ContentPane"
        data-dojo-props="title: 'A Tab', href: 'content.html'"></div>
</div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/href.html)

这里有几个解析器支持的`<script type="dojo/*">`：
`dojo/on`
	用来设置事件处理器。`data-dojo-event`里提供的事件等同于调用`object.on()`，参数以`data-dojo-args`里的逗号分隔列表的方式传递。通常，它只是你命名标准事件的变量（例如：`data-dojo-args="e"`）。

`dojo/aspect`
	用来修改一个方法处理器。它等同于使用`dojo/aspect`模块。`data-dojo-advice`提供要处理的方法的特殊“advice”。一般，这是`after`，但是`dojo/parser`也支持`before`和`around`。在`data-dojo-event`支持这个方法，你需要传递给方法的参数变量都以逗号分隔列表的形式由`data-dojo-args`提供。

`dojo/watch`
用来在属性发生改变时执行一个处理器。由`data-dojo-prop`来设定属性，就像Dijits的`watch()`和基于`dojo/Stateful`的对象，处理器接收三个代表属性名的参数，以逗号分隔列表的方式在`data-dojo-args`属性里面命名的旧值和新值。

`dojo/method`
用于执行实例上的代码或者重写一个方法。如果没有指定的`data-dojo-event`属性，代码块就会在对象实例化时执行。如果指定了方法，那个所有已有的功能都会被代码块代替。

`dojo/connect`
它已经用`dojo/on`和`dojo/aspect`代替了，不过在连接一个函数和另一个函数时它本质上执行同样的功能。

在`dojo/parser`参考指南的 [Script Tags section](https://dojotoolkit.org/reference-guide/1.10/dojo/parser.html#script-tags)，你可以找到声明式脚本的更多信息，其中包括一些例子，

##声明式require模块
AMD最棒的事情是你可以非常模块化，你只要在代码真正需要某个模块时才加载它。当然如果你要完全利用声明式语法，你会发现很难再声明式脚本里获得模块。或者你可能只是想要生成你的标签并最小化需要写的JavaScript。`dojo/parser`中的声明式require可以帮你。

例如，当我点击一个按钮，我想要激活另一个按钮并设置它点击事件的处理器，如果不用该功能我不得不这样声明：

```
<button type="button" id="button1" disabled="disabled"
        data-dojo-type="dijit/form/Button">
    <span>I'm disabled</span>
</button>
<button type="button" id="button2" data-dojo-type="dijit/form/Button">
    <span>Click Me!</span>
    <script type="dojo/on" data-dojo-event="click">
        require(["dijit/registry"], function(registry){
            var button1 = registry.byId("button1");
            button1.on("click", function(){
                console.log("I was clicked!");
            });
            button1.set("label", "I'm enabled");
            button1.set("disabled", false);
        });
    </script>
</button>
```
如果你确定你之前已经require了`dijit/registry`，你可以使用`var registry = require("dijit/registry");`来引用这个模块。

当然，你还可以用声明式require。前面提到过，所有的声明式脚本只在全局作用域内执行，所以声明式require得到的任何东西都会放到全局作用域内，进而提供给任何声明式脚本。声明式require的语法本质是一个不带大括号（{}）的JavaScript对象。属性名是全局变量的名称，值应该是一个包含模块ID（MID）的字符串。为了require `dijit/registry`并把它映射到一个名叫“registry”的全局变量，我们要这么做：

```
<script type="dojo/require">
    registry: "dijit/registry"
</script>
```
>[View demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/require.html)

如果你担心它和全局范围模块的命名空间冲突，你可以在属性名中使用点标记，把模块放在它自己的命名空间：`"myApp.registry": "dijit/registry"`。解析器会创建所需对象来让你随后在代码中像`myApp.registry.byId("someId")`这样使用registry。

##自动require
Dojo 1.8中引入`dojo/parser`的另一特性时“自动require”模块。这让拼凑页面变得很容易，因为你在调用`dojo/parser`之前不必确认是否require了你的模块。如果解析器遇到看起来像MID（例如里面含有`/`）的`data-dojo-type`值，而且这个模块还没有加载，解析器会在页面开始实例化对象之前尝试require该模块。

这个特性很有用，但是如果你不小心使用，它可能会对你的应用性能造成负面影响。例如，你可能在你的成果站点使用一个build过的版本，你希望builder会把所有你需要的模块build到一个自定义layer。那么，默认情况下，builder不会扫描文件来查找声明式依赖项，因此任何`dojo/parser`自动require的模块都不会在你layer里。这就是说对于每一个模块的require，你的应用都得执行一次单独的服务器请求，这可能导致你的应用运行缓慢。

为了让你知道这个，以便于你可以决定是否使用这个特性，你可以将Dojo设为调试模式（`isDebug: true`），然后当模块自动require时`dojo/parser`会在控制台记录。

看它的实际应用，上面TabContainer 示例已经重写成自动require它需要的Dijits。同时它也在调试模式下，你可以在JavaScript控制台查看警告信息。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/declarative/demo/autorequire.html)

这是一种让builder分析你的静态HTML文件来得到依赖项并将它们build到一个layer的方式。在builder转换 [depsDeclarative](https://dojotoolkit.org/reference-guide/1.10/build/transforms/depsDeclarative.html)参考指南里有涉及到。

##小结
希望这篇教程已经让你对如何利用声明式语法的特性和`dojo/parser`有了很好的了解。它非常灵活，也可以又快又容易地编写相对复杂的应用。我个人认为当你可以战略性组合声明式语法和编程式风格时将会更加给力。

这里有一些资源你可能想要看看：
 - [`dojo/parser `Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojo/parser.html)
 - [Prototyping with dijit/Declaration Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/declaration)
 - [Builder Transform `depsDeclarative` Reference Guide](https://dojotoolkit.org/reference-guide/1.10/build/transforms/depsDeclarative.html)
 - [The Auto-Require Demonstration Application](http://demos.dojotoolkit.org/demos/parserAutoRequire/)

