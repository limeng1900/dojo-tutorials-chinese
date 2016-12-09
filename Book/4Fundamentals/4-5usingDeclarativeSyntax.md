# 4.5使用声明式语法
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/declarative/index.html

----------

开始使用Dojo的最快方式之一，尤其是用Dijit这样的widget，是利用`dojo/parser`和声明式语法。本教程将帮你充分利用这种风格的编程。

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

