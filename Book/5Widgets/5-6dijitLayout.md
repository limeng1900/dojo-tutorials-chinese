# 5.6 Dijit布局

原文地址：[https://dojotoolkit.org/documentation/tutorials/1.10/dijit\_layout/index.html](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/index.html)

---

创建动态和交互式的布局是所有图形用户界面的共同挑战。通过HTML和CSS我们已经拥有了创建布局的许多能力。在CSS之外，Dojo挑选了一系列可扩展的widget作为Dijit的一部分——Dojo的UI框架。在这片及教程中，我们将阐述Dijit如何应对常见的布局需求，并了解它如何只用几个灵活的widget创建更加复杂的布局。

## 布局管理介绍

> “CSS不是布局的语言么？为什么布局还需要JavaScript和widget来解决问题？”

布局widget没有代替CSS在常规目标安排和页面内容流上的作用。它们针对页面区域有着更加严格的安置和管理，这正式我们需要的：

* 响应调整大小事件
* 为用户提供布局控制，以及如何安排可用区域
* 使空间 and/or 内容适应当前可用的水平或垂直空间

布局管理是在页面加载后主动控制布局、响应和传播事件，然后在页面中而推动布局的过程。在Dijit中，布局管理由专业的布局widget完成。有这样一些widget，它们的首要目标就是作为一个或多个内容区域或子widget的容器，控制这些子元素的大小的和显示。

## 入门

你可以对整个页面的布局进行管理，或者只针对其中一小部分。在这个教程中，我们将开发一个类桌面应用的UI布局，来让页面中的一些控件和内容可以修改。它最后会像下面这样：

> [View Complete Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/appLayout.html)

Dijit提供了一小撮灵活的widget来应对这样的常见布局需求。我们先准备一些HTML和CSS，然后引入这些widget来构建一个典型的应用布局。

```
<!DOCTYPE HTML>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Demo: Layout with Dijit</title>
        <link rel="stylesheet" href="style.css" media="screen">
        <link rel="stylesheet" href="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dijit/themes/claro/claro.css" media="screen">
    </head>
    <body class="claro">
        <div id="appLayout" class="demoLayout">
            <div class="centerPanel">
                <div>
                    <h4>Group 1 Content</h4>
                    <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
                </div>
                <div>
                    <h4>Group 2 Content</h4>
                </div>
                <div>
                    <h4>Group 3 Content</h4>
                </div>
            </div>

            <div class="edgePanel">Header content (top)</div>
            <div id="leftCol" class="edgePanel">Sidebar content (left)</div>
        </div>
        <!-- load dojo and provide config via data attribute -->
        <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
                data-dojo-config="async: 1, parseOnLoad: 1">
        </script>
    </body>
</html>
```

这些标签中有top、sidebar和center内容包含在div中，并且我们也在相应的位置获得了Dojo `script`标记。在`<head>`中我们加载Cloro主题样式和我们的页面样式表。在`body`中，注意要给内容应用Claro CSS主题的话，`claro`类是必需的。常常会忽略它。

定义布局时，我们只需要用到样式中的几个规则：

```
html, body {
    height: 100%;
    margin: 0;
    overflow: hidden;
    padding: 0;
}

#appLayout {
    height: 100%;
}
#leftCol {
    width: 14em;
}

.claro .demoLayout .edgePanel {
    background-color: #d0e9fc;
}

#viewsChart {
    width: 550px;
    height: 550px;
}
```

> 这里展示的所有样例中包含一个demo.css文件，它包含了body、button和h1元素的几个样式。查看任意样例的代码来了解这个文件的内容。

想要得到预期的内容区域的布局和行为，我们要用布局来填充视窗。我们明确地设置文档和最外层的元素为100%视窗的高度。`overflow:hidden`用来设置不出现滚动条；滚动将在必要时出现在我们布局的不同区域。我们已经给将成为左边一列的`DIV`一个`em`为单位的可变宽度。其他可变区域将有它们的初始化内容来确定它们的大小。

## 添加Widget

我们将使用Dijit的三个widget类：`dijit/layout/BorderContainer`、`dijit/layout/TabContainer`和`dijit/layout/ContentPane`来实现布局。

首先，添加一个`require`调用来加载这些依赖项。

```
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
        data-dojo-config="async:true, parseOnLoad:true">
</script>
<script>
    require(["dojo/parser", "dijit/layout/BorderContainer", "dijit/layout/TabContainer",
        "dijit/layout/ContentPane"]);
</script>
```

注意Dojo脚本标签的`data-dojo-config`属性中，我们把`parseOnLoad`设置为true。这告诉Dojo为它找到的元素运行解析器自动进行“部件化”。有了这个我们就可以完全依靠解析器，不需要`dojo/domReady!`或者类似的其它，我们只加载要用到的。

> 注意我们也明确地加载了`dojo/parser`模块。这很重要，尽管经常被误解，但在`parseOnLoad`设置为`true`时，`dojo、parser`并不会自动加载，也从来不会。它只是因为在1.7之前的许多widget加载`dijit/_Templated`（它会加载`dojo/parser`）。

widget会在后台加载，解析器将遍历DOM。但是实际上什么都还没有发生，我们需要创建这些布局widget。

这个例子中，我们将使用标签或声明方式来实例化widget。每个元素的`data-dojo-`属性为Dojo解析器提供指令，来指导实例化哪个widget类，并给出widget实例化的配置属性。

```
<body class="claro">
    <div
            id="appLayout" class="demoLayout"
            data-dojo-type="dijit/layout/BorderContainer"
            data-dojo-props="design: 'headline'">
        <div
                class="centerPanel"
                data-dojo-type="dijit/layout/ContentPane"
                data-dojo-props="region: 'center'">
            <div>
                <h4>Group 1 Content</h4>
                <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
            </div>
            <div>
                <h4>Group 2 Content</h4>
            </div>
            <div>
                <h4>Group 3 Content</h4>
            </div>
        </div>
        <div
                class="edgePanel"
                data-dojo-type="dijit/layout/ContentPane"
                data-dojo-props="region: 'top'">Header content (top)</div>
        <div
            id="leftCol" class="edgePanel"
            data-dojo-type="dijit/layout/ContentPane"
            data-dojo-props="region: 'left', splitter: true">Sidebar content (left)</div>
    </div>
</body>
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/borderContainer.html)

将外部的`appLayout`元素设为一个`BorderContainer`，并将其每一个子div设为`ContentPane`。这就有了一个全屏的灵活布局。试着调整示例窗口的大小，观察左边如何保持一个固定宽度而让右边变化。你可能还注意到在左边和中心区域之间有一个垂直分割的处理器，它可以让你手动设定它们的相对宽度。

这就是我们所说的动态交互式布局。我们将在初始的例子中添加标签，不过先回过来仔细看看这些个别的布局widget和他们的使用。

## BorderContainer

如果你用过其他用户界面工具集的布局管理和容器，就应该很容易熟悉`dijit/layout/BorderContainer`，就算没有也能很快上手。

BorderContainer 可以定义将一个布局细分。`center`区域通常是灵活和自动大小的，其他区域则是固定大小：“`top`”“`bottom`”、“`leading`”、“`trailing`”、“`left`”、“`right`”。

所有的Dijit widget支持国际化（i18n），所以Dijit不会假定页面上从左到右的内容和控制。对于从左向右阅读的地域，`leading`部分会放在左边，而`trailing`部分放在右边。从右向左阅读的地域，（例如阿拉伯语、希伯来语），则是相反的。话虽如此，你仍可以使用`left`和`right`来确保这些部分放在你选定的区域，忽略掉地域。总之，要适合你的内容。

每一个区域使用一个子widget表示，像我们在App布局示例中看到的那样。所有的Dijit 部件支持`region`属性，所以原则上，你可以在这些位置使用任何widget，只是有一些用起来明显比其他的好。固定大小区域（除了`center`以外的全部）可以通过设置一个`splitter`属性来获得一个手动的分配器。

当你使用BorderContainer，区域的初始大小以正常方式使用CSS来指定——使用样式表规则或者行内样式。注意虽然你可以设置50%这样的初始大小，渲染时它会被`px`覆盖，所以百分比单位在BorderContainer调整大小时不会保留。center区域不应该给出高度或宽度属性，它总是占据剩余的空间。

目前为止已经构建了我们的布局，所有的区域都是`ContentPane`——一个很常用的内容加载和容纳的widget，但是像我们在第一个App 布局中看到的一个TabContainer占据center区域其实不必如此。实际上，BorderContainer嵌套很好用。这里有一个BorderContainer嵌套进行复杂布局的例子：

```
<div class="demoLayout" style="height: 300px; width: 300px" data-dojo-type="dijit/layout/BorderContainer" data-dojo-props="design: 'headline'">
    <div class="centerPanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="region: 'center'">center</div>
    <div class="demoLayout" style="height: 50%" data-dojo-type="dijit/layout/BorderContainer" data-dojo-props="region: 'top', splitter: true, design: 'headline'">
        <div class="centerPanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="region: 'center'">center</div>
        <div class="edgePanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="region: 'bottom'">bottom</div>
    </div>
    <div class="edgePanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="splitter: true, region: 'left'">left</div>
    <div class="demoLayout" style="width: 50%" data-dojo-type="dijit/layout/BorderContainer" data-dojo-props="region: 'right', design: 'headline'">
        <div class="centerPanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="region: 'center'">center</div>
        <div class="edgePanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="region: 'left'">left</div>
    </div>
    <div class="edgePanel" data-dojo-type="dijit/layout/ContentPane" data-dojo-props="splitter: true, region: 'bottom'">bottom</div>
</div>
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/nestedBorderContainer.html)

BorderContainer 不同选项和使用的更多细节参见 [BorderContainer documentation](https://dojotoolkit.org/reference-guide/1.10/dijit/layout/BorderContainer.html) 。

## 创建选项卡

一个布局widget的任务是在可用的空间内布局和展示内容。最希望它的内容是一个或多个子widget。一个常见需求是一次只显示这些子widget中的一个，并把它们当做用户可以互动的堆栈。这样可以最大化利用空间，还能实现只在选择该条目时才加载其内容。Dijit在这里提供了StackContainer、TabContainer和AccordionContainer。

我们尝试创建的布局将不同组的div以选项卡面板展示，并在中心区域底部放一个标签。这是一个可以追溯到模拟文件系统的常见、直观的UI模式。`dijit/layout/TabContainer`实现了这个模式。它用选项卡表示它包含的子widget，以一次一个的方式在预留空间中展示他们的内容。

```
<div class="centerPanel"
        data-dojo-type="dijit/layout/TabContainer"
        data-dojo-props="region: 'center', tabPosition: 'bottom'">
    <div
            data-dojo-type="dijit/layout/ContentPane"
            data-dojo-props="title: 'Group 1'">
        <h4>Group 1 Content</h4>
        <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>
    </div>
    <div
            data-dojo-type="dijit/layout/ContentPane"
            data-dojo-props="title: 'Group Two'">
        <h4>Group 2 Content</h4>
    </div>
    <div
            data-dojo-type="dijit/layout/ContentPane"
            data-dojo-props="title: 'Long Tab Label for this One'">
        <h4>Group 3 Content</h4>
    </div>
</div>
```

要将选项卡放进TabContainer，我们首先要把容器元素设为一个TabContainer。这个widget本身就是BorderContainer的一个子元素，所以它仍然需要`region`属性。TabContainer提供了许多选项来配置如何显示标签及内容；这里我们通过设置`tabPosition`属性把标签放在底部。TabContainer是另一个容器widget——它管理子widget——所以我们要将每一个部分放进一个合适的widget。它们用于做太多，所以选择ContentPane就可以。注意每一个都提供一个“`title`”属性。TabContainer将这些标题作为每个子widget创建的对应选项卡的标签。

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/appLayout.html)

## StackContainer and Friends

我们已经完成了布局，但是如何变化这些方案并构建你自己的布局——甚至于你自己的布局widget——我们需要在深入一点。TabContainer 实际上是`dijit/layout/StackContainer`的一个子类；它从StackContainer （也是`dijit/layout/_LayoutWidget`的一个扩展）借用了许多功能。TabContainer 指定了内容面板如何排列和表示，而StackContainer是一个更通用的widget。没有提供内部控制器可以在子widget之间导航，但是`dijit/layout/StackController`可以简单的实现。下面是我们如何将TabContainer 替换为StackContainer，并且把控制器widget放在一个新的`bottom`区域。这里我们也改变了BorderContainer 使用侧边栏布局。

> [StackContainer Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/stackContainerAppLayout.html)

Dijit也提供一个易用的`dijit/layout/AccordionContainer`，在`dojox/layout`中还有其他StackContainer子类可能满足你的需求。同样的，`ContentPane`也可以用`dojox/layout/Expandopane`替换，从而在布局中提供一个可折叠的面板。通常在你开始考虑自定义之前，先熟悉一下所提供的选项是值得的。

## 启动与调整尺寸

目前我们已经了解如何用标签布局以及Dojo解析器实例化及启动序列。你已经走了很长一段路，这些可能是关于Dijit布局你真正需要了解的全部内容。然而，如果你想要编程式地创建和插入widget，它就没用了。我们需要更多地了解布局如何与何时发生。

先复习下，我们知道：

* 定义一个widget涉及一个良好定义的顺序
* 布局本质上是关联可用空间的测量
* 在`start`发生之前，widget的`domNode`不能保证已经存在于DOM
* 布局widget主动地布置它们的子widget

当编程式创建widget是，我们需要通过调用`startup`方法完成序列。这一步包括一旦widget放在DOM时任何值发生一次的事——它包含测量和定型。布局widget将在它们的子集中调用`startup`，这样他就可以从最顶端的widget往下滚。

按照定义，所有的布局widget有一个`resize`方法。这个方法在启动期间调用，在发生需要调整尺寸的变化时或者添加一个新的子widget时也会调用。类似`startup`，`resize`也会向下传递，让布局中的每个widget调整，并向它的子集传递新的尺寸。

记住这些，我们来看一些代码。下面是基础框架：

```
<head>
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
        data-dojo-config="async:1">
    </script>
    <script>
    require(["dijit/registry", "dijit/layout/BorderContainer",
            "dijit/layout/TabContainer", "dijit/layout/ContentPane", "dojo/domReady!"],
        function(registry, BorderContainer, TabContainer, ContentPane){
            // create the main appLayout BorderContainer
            // create the TabContainer
            // create the BorderContainer edge regions
        });
    </script>
</head>
<body class="claro">
    <div id="appLayout" class="demoLayout"></div>
</body>
```

这里我们已经忽略了`parseOnLoad`，它默认为false；作为代替，我们使用`dojo/domReady!`来等待DOM加载。

```
// 创建 BorderContainer 并将它附到 appLayout div
var appLayout = new BorderContainer({
    design: "headline"
}, "appLayout");

// 创建 TabContainer
var contentTabs = new TabContainer({
    region: "center",
    id: "contentTabs",
    tabPosition: "bottom",
    "class": "centerPanel"
});

// 将 TabContainer 作为 BorderContainer 的一个子widget
appLayout.addChild( contentTabs );

// 创建并添加 BorderContainer 边缘区域
appLayout.addChild(
    new ContentPane({
        region: "top",
        "class": "edgePanel",
        content: "Header content (top)"
    })
);
appLayout.addChild(
    new ContentPane({
        region: "left",
        id: "leftCol", "class": "edgePanel",
        content: "Sidebar content (left)",
        splitter: true
    })
);

// 给 TabContainer 添加初始化内容
contentTabs.addChild(
    new ContentPane({
        href: "contentGroup1.html",
        title: "Group 1"
    })
);

// 启动并布局
appLayout.startup();
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/programmaticLayout.html)

每一个widget都是用我们之前定义的`data-dojo-props`属性的等价属性实例化的。不同于标签的隐式包含，每个widget都显示地通过使用`addChild`方法添加给父级。

注意当所有的widget都添加后，调用appLayout 的 startup方法。在startup之前，`addChild`只是简单地widget注册为一个子元素。在startup之后，`addChild`可能代表一个布局变化，所以它将触发父级和其它全部子集的`resize`。

我们可以通过在布局渲染完成后添加一个新的子元素来了解它。下面是它的一个快速功能测试：

```
function addTab(name) {
    var pane = new ContentPane({
        title: name,
        content: "<h4>" + name + "</h4>"
    });
    // add the new pane to our contentTabs widget
    registry.byId("contentTabs").addChild(pane);
}
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/demo/addTabs.html)

## 小结

我们已经看过了Dijit为创建动态布局提供的模块，以及如何使用声明式标签风格进行组合和使用编程式来创建。这个方式可以让你全方位的选择如何定义和装配你的UI。在我们更多探索Dijit你也将发现同样的灵活性。还可以通过在Dijit提供的基础上创建自己的布局widget来丰富我们的选择。这将是未来的教程主题。

