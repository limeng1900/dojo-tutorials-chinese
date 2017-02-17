#5.9 创建基于模板的widget

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/templated/index.html

---

这篇教程你将了解Dijit的`_TemplateMixin`混合的重要性，以及如何使用模板快速创建自定义widget。

##入门
如果你还不熟悉Dijit创建widget的基础，先阅读下[理解 _WidgetBase教程](https://dojotoolkit.org/documentation/tutorials/1.10/understanding_widgetbase/)。[创建自定义的widget教程](https://dojotoolkit.org/documentation/tutorials/1.10/recipes/custom_widget)和[编写你自己的Widget指南](https://dojotoolkit.org/reference-guide/1.10/quickstart/writingWidgets.html)也会对你有所帮助。

Dijit的`_WidgetBase`为创建widget提供了一个了不起的基础，但是`_TemplateMixin`混合才是Dijit真正的亮点所在。使用`_TemplatedMixin`和`_WidgetsInTemplateMixin`，你可以快速地创建高可维护性、快速修改和易于操作的widget。

`_TemplatedMixin`的基本概念很简单：它可以让开发人员剑姬一个带有一些小扩展的小HTML文件，并在运行时（或在build过程中）将这个HTML文件当做字符串加载，提供给这个模板widget的所有实例进行重复使用。

我们略过`_TemplatedMixin`的定义，先从头用它的功能来创建一个简单的widget。

注意`_TemplatedMixin`旨在用作混入，而不是直接继承。在基于类的语法中，意味着它相比于类更像是一个接口（虽然在JavaScript中，两者的区别很模糊）。更多信息请看[Dojo Declare Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/declare/) 。

##_TemplatedMixin提供什么

对于开发者，将`_TemplatedMixin`混入widget定义为你的widget提供了一下额外属性：

```
templateString        //    表示模板HTML的字符串
```

这个属性看起了很简单，它如何起到巨大作用？答案在于`_TemplatedMixin`向你的widget定义中添加了什么。

一个小提示：`templatePath`也被加入，但不在用于模板加载。它是为了向后兼容。后面我们将展示如何使用`dojo/text!`来加载一个widget的模板。

### 重写方法

除了上面的属性，`_TemplatedMixin`重写了定义在Dijit widget架构的两个方法： `buildRendering`和`destroyRendering`。这两个方法处理模板的解析与填充（`buildRendering`）和正确地销毁widget的DOM（`destroyRendering`）。

因为这两个方法都是模板处理的关键，如果你在自定义代码中重写了两者之一 —— 确保在重写的方法中添加`this.inherited(arguments)`来包含一个对父版本的调用。更多信息请看 [Understanding _WidgetBase Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/understanding_widgetbase/)中的widget声明周期。

### 使用 _TemplatedMixin

要让你的自定义widget模板化，你要做的就是将`dijit/_TemplatedMixin`作为你widget类声明数组的第二个或之后的参数。例如一个SomeWidget定义如下：

```
define([
    "dojo/_base/declare",
    "dijit/_WidgetBase",
    "dijit/_TemplatedMixin",
    "dojo/text!./templates/SomeWidget.html"
], function(declare, _WidgetBase, _TemplatedMixin, template) {

    return declare([_WidgetBase, _TemplatedMixin], {
        templateString: template
    });

});
```

Dijit坚持这样的标准，在文件夹中创建一个单独的名为`templates`的目录来包含widget模块—— 建议在你的代码中也遵循这个标准。

注意在我们上面简单定义中，我们使用了`templateString`属性来与通过`dojo/text!{path}`加载的模板相结合。这是引用模块文件的推荐方式，它可以确保文件异步加载，并在创建Dojo工具集的build时正确地整合。

现在已进行了基于模板的widget声明，我们来编写一个模板并讨论它们中的一些特殊挂钩。

##编写模板

模板是你定义的DOM结构中的一个HTML文档片段，它有一些特殊的“挂钩”来将一些事情绑定到你的widget声明。在我们深入每一挂钩之前先看一个例子，如何在一个模板中进行变量替换。这是SomeWidget假设的一个模板：

```
<div class="${baseClass}">
    <div class="${baseClass}Title" data-dojo-attach-point="titleNode"
            data-dojo-attach-event="onclick:_onClick"></div>
</div>
```

这个模板展示了Dijit模板系统最重要的三个方面：变量替代、附加点和事件附着。

注意当你定义一个模板时，它只能有一个根节点定义（就像XML文档）。不允许在最高级存在多个节点。

### 变量替代

通过使用简单的变量占位符语法，模板可以在DOM渲染的时候设置值，像这样：

```
${property}
```

你widget中声明的任何属性或字段定义都可以作为变量名；上面的例子使用`baseClass`属性（任何widget都有），不过自定义字段也一样——例如，如果在SomeWidget中我们定义了一个叫'foo'的属性，我们可以在模板中使用`${foo}`。如果属性是引用自一个对象，而你想要使用这个对象中一个属性的值，你可以使用正常的对象引用符号：

```
${propertyObject.property}
```

要防止`_TemplatedMixin`从超出字符串，在完整的变量名前放一个“!”，像这样：

```
    ${!property}
```
只推荐针对在widget生命周期中不会发生变化的值在模板中使用变量替代。也就是说，如果在widget生命周期期间你想要编程式地设置一个属性值，推荐你使用widget的`postCreate`方法来通过`set()`方法编程式地设置任何变量。

### Attach Points

Dijit的模板系统有一个特殊属性，它会在你的模板中查找一个叫做attach point ——用HTML5 data 属性语法实现。一个附加点告诉模板渲染器当一个`data-dojo-attach-point`属性的DOM元素创建时，将这个属性的值设置为你widget的一个属性，作为创建的DOM元素的引用。例如，SomWidget模板定义了两个DOM元素。主元素（外部的`div`）可以通过`domNode`属性在你的代码中引用，内部`div`元素可以在你的代码中通过`titleNode`引用。

通常，你模板的根节点会变成你的widget的`domNode`属性，因此你通常不需要在你的定义中包含这个附加点。然而，有时也会这么做来让根节点在其他子系统中也起作用，例如Dijit的focus管理。

### The containerNode Attach Point

Dijit也定义一个名叫containerNode的神奇附加点。容器节点的基本思想是当声明式创建widget时，为额外的标签提供位置。例如SomeWidget的模板：

```
<div class="${baseClass}">
    <div class="${baseClass}Title" data-dojo-attach-point="titleNode"
            data-dojo-attach-event="ondijitclick:_onClick"></div>
    <!-- 我们的容器: -->
    <div class="${baseClass}Container"
            data-dojo-attach-point="containerNode"></div>
</div>
```

我们可以在声明式标签中这么使用它：

```
<div data-dojo-type="demo/SomeWidget"
        data-dojo-props="title: 'Our Some Widget'">
    <p>This is arbitrary content!</p>
    <p>More arbitrary content!</p>
</div>
```

当Dojo解析器解析文档是，它将找到我们的示例widget并将其实例化——作为实例化的一部分，widget中的所有标签都将追加到containerNode中。所以当widget完成启动时，DOM会是这样：

```
<div id="demo_SomeWidget_0" class="someWidgetBase">
    <div class="someWidgetTitle">Our Some Widget</div>
    <div class="someWidgetContainer">
        <p>This is arbitrary content!</p>
        <p>More arbitrary content!</p>
    </div>
</div>
```

注意为了简洁我们移除了一些自定义属性，Dijit在渲染模板的时候不会移除。

还要注意如果你在主标签中插入其他widget定义，并且你的widget有一个`containerNode`，所有widget都将在容器节点里实例化。例如，下面是一个典型案例：

```
<div data-dojo-type="demo/SomeWidget">
    <p>This is arbitrary content!</p>
    <div data-dojo-type="dijit/form/Button">My Button</div>
    <p>More arbitrary content!</p>
</div>

```

### 事件附着

除了附加点，Dijit模板系统还给你提供了一个方式来将原生的DOM事件附到自定义widget的方法中。它使用HTML5 data属性`data-dojo-attach-event`。这是一个逗号分隔的键值对（用冒号分隔）字符串；键是要附加处理器的原生DOM事件，值是事件触发时你的widget要执行的方法名。如果只有一个事件需要处理，省略尾部逗号。例如，这是Dijit的MenuBarItem定义的`data-dojo-attach-event`属性：

```
data-dojo-attach-event="onmouseenter:_onHover,onmouseleave:_onUnhover,ondijitclick:_onClick"
```

当你的widget实例化并且由模板创建了DOM片段，Dijit模板系统将随后检查所有附加时间定义，并自动将这些事件（使用`dojo/on`）与结果DOM和你的widget对象相连接——让可视化表示与控制代码之间的连接变得非常简单。另外，当这些事件处理器触发时，原生DOM事件机制传递的参数也会传递给你的widget处理器，如此你就可以完全地访问浏览器的报告。

我们想要使用`dijit/_OnDijitClickMixin`，它添加了一个改进事件，比标准的DOM `onclick`事件支持更多的功能。所以我们需要修改widget声明：

```
define([
    "dojo/_base/declare",
    "dijit/_WidgetBase",
    "dijit/_OnDijitClickMixin",
    "dijit/_TemplatedMixin",
    "dojo/text!./templates/SomeWidget.html"
], function(declare, _WidgetBase, _OnDijitClickMixin, _TemplatedMixin,
        template) {

    return declare([_WidgetBase, _OnDijitClickMixin, _TemplatedMixin], {
        templateString: template
        //    any custom code goes here
    });

});
```

还需要修改我们的widget模板：

```
<div class="${baseClass}">
    <div class="${baseClass}Title"
        data-dojo-attach-point="titleNode"
        data-dojo-attach-event="ondijitclick:_onClick"></div>
    <div>And our container:</div>
    <div class="${baseClass}Container"
        data-dojo-attach-point="containerNode"></div>
</div>
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/templated/demo/templated-demo.html)

>Domo files:[`demo/SomeWidget.js`](https://dojotoolkit.org/documentation/tutorials/1.10/templated/demo/SomeWidget.js) [`demo/templates/SomeWidget.html`](https://dojotoolkit.org/documentation/tutorials/1.10/templated/demo/templates/SomeWidget.html)

##_WidgetsInTemplateMixin 混合

最后，使用`_WidgetsInTemplateMixin`混合，Dijit的模板系统还能用模板来创建更加复杂的widget。这个混合类告诉模板系统你的模板中还有其他的widget，当你的widget实例化时也将他们实例化。

例如，让我们修改定义来包含一个Dijit按钮：

```
define([
    "dojo/_base/declare",
    "dijit/_WidgetBase",
    "dijit/_OnDijitClickMixin",
    "dijit/_TemplatedMixin",
    "dijit/_WidgetsInTemplateMixin",
    "dijit/form/Button",
    "dojo/text!./templates/SomeWidget.html"
], function(declare, _WidgetBase, _OnDijitClickMixin, _TemplatedMixin,
            _WidgetsInTemplateMixin, Button, template) {

    return declare("example.SomeWidget", [_WidgetBase, _OnDijitClickMixin,
        _TemplatedMixin, _WidgetsInTemplateMixin
    ], {
        templateString: template
        //    your custom code goes here
    });

});
```

然后创建一个模板：

```
<div class="${baseClass}" data-dojo-attach-point="focusNode"
        data-dojo-attach-event="ondijitclick:_onClick"
        role="menuitem" tabIndex="-1">
    <div data-dojo-type="dijit/form/Button"
        data-dojo-attach-point="buttonWidget">
        My Button
    </div>
    <span data-dojo-attach-point="containerNode"></span>
</div>
```

注意在我们修改的模板中，我们已经连同按钮的标记添加了一个名叫`buttonWidget`的附加点。这是Dijit的附加点系统的额外奖励；因为给我们的widget添加的属性定义是一个widget——`myWidget.buttonWidget` —— 它实际是按钮widget的一个引用，而不是一个DOM元素的引用。这就可以让你用简单的积木搭建更好的widget，比如一个查看邮件列表的widget，一个预置widget的工具条，还有许多。

还要注意你要把模板用到的widget require到模块中。对于widget模板你不能使用`dojo/parser`Dojo 1.8引入的自动require特性，因为创建的生命周期是异步的，但是自动require特性必须异步运行。

除非你有一个明确的需求要在模板中定义一个widget，否则不要混入`dijit/_WidgetsInTemplateMixin`。如果滥用会招致对widget和你的应用性能的影响。

##小结

在这篇教程中，我们学习了Dijit强大的模板系统，它通过混合`_TemplatedMixin` 和`_WidgetsInTemplateMixin`实现，还有如何用这个系统来快速创建自定义widget用在你的应用中。我们也讨论了如何用模板系统的附加点和事件附着来快速地将DOM元素绑定到你的代码，以及如何提盒你模板中的值 —— 还有如何在你的widget模板中包含其他widget来创建超级widget。

Happy widget building!