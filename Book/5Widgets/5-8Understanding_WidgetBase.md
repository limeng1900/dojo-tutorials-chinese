#5.8 理解 _WidgetBase

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/understanding_widgetbase/

---

在这篇教程中你将了解Dijit的`_WidgetBase`模块，以及它是如何做为Dojo工具集所有widget的基础的。

##入门
Dijit的基础和创建自己的widget时，都依赖于一个基类，它定义在`dijit/_WidgetBase`模块。使用Dojo工具集的web应用开发通常依赖几个其他关键的部分（例如 [Dojo parser](https://dojotoolkit.org/reference-guide/1.10/dojo/parser.html)和 [Dijit 模板系统](https://dojotoolkit.org/reference-guide/1.10/dijit/_TemplatedMixin.html)），而这个模块则是使用Dojo工具集创建任何自定义widget的关键。在这篇教程中，你将了解Dijit的widget的基础框架如何工作。

>如果你从早期版本的Dojo过来，你可能会熟悉`dijit/_Widget`模块。现在`dijit/_Widget`模块仍然存在并且继承自`dijit/_WidgetBase`，当前建议你在自定义widget直接从`dijit/_WidgetBase`继承。`dijit/_Widget`可能在Dojo 2.0逐步淘汰。

理解Dijit系统最重要的一点是一个widget的生命周期。生命周期是一个widget全面启动首要考虑的——也就是说，从开始构想到完全被你的应用使用——直到销毁widget和相关的DOM元素。

>如果你疑惑为什么“Widget”和“WidgetBase”前面都有“_”，是因为它们都不能直接被实例化；相反，它们作为使用Dojo工具集`declare`机制的基类。

为了实现这一点，`dijit/_WidgetBase`定义了两组概念：在创建过程中连续调用的一套方法，widget在应用中getting/setting 字段的最小化绑定数据的一种方式。让我们先看第一种机制：Dijit的widget 生命周期。

##Dijit 生命周期

每个以`_WidgetBase`为基类声明的widget在实例化期间都会运行几个方法。下面按它们被调用的顺序；列了出来：

 - `constructor` （所有原型共有，实例化时调用）
 - `postscript` （所有使用`decale`构建的原型共有）
	 - `creat`
		 - `postMixInProperties`
		 - `buildRendering`
		 - `postCreate`
 - `startup`

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/understanding_widgetbase/demo/lifecycle.html)

这些方法用来处理一系列事务：

 - 由默认和运行值来初始化widget数据
 - 为widget的可视化表示生成DOM结构
 - 将widget的DOM结构放进页面
 - 处理那些依赖文档中存在的DOM结构的逻辑（例如DOM元素的尺寸）

### postCreate()

目前为止，在创建自己的widget时你要记住的最重要的方法是`porsCreate`方法。它在widget所有的属性定义完成和代表widget的文档片段创建后触发 —— 但是在文档片段添加到主文档之前。这个方法重要的原因在于它是widget呈现给用户之前开发者能够进行最后修改的主要地方，包含设置任何自定义属性等等。当开发一个自定义widget时，你的大部分定制都发生在这里。

### startup()

Dijit生命周期中第二重要的方法可能就是`startup`方法了。这个方法用来处理DOM片段实际添加到文档中之后的过程；它将在所有的子widget被创建和开启之后才触发。它对于合成widget和布局widget尤其有用。

>当编程式实例化一个widget时，通常在将它放进文档之后调用widget的`startup()`方法。常见的错误是编程式创建了widget但是忘了调用`startup`，让你抓耳挠腮为啥你的widget不能正确显示。

### Tear-down 方法

作为实例化方法的补充，`dijit/_WidgetBase`也定义了许多销毁方法（也按调用顺序列出）：

 - `destroyRecursive`
	 -  `destroyDescendants`
	 -  `destroy` 
		 - `uninitialize`
		 - `destroyRendering`

当编写你自己的widget时，任何必须的拆除行为都应该定义在**destory**方法中。（不要忘了调用`this.inherited(arguments)`！）Dijit本身已经为解决了节点和大多数对象的管理（使用前面提到的销毁方法），因此你通常不需要担心需要从头编写这些方法的自定义版本。

>虽然`destroy`是所有widget的主要销毁方法，在明确的销毁一个widget时也可以调用`destroyRecursive`。它确保不只销毁widget本身还销毁任何的子widget。

##节点引用

一个widget通常是某种类型的用户界面，没有某些DOM表示它可能不会完整。`_Widget`定义一个名叫`domNode`的标准属性，它是widget本身的整体父节点的一个引用。如果需要（例如在文档中移动整个widget），你可以编程式的获得这个节点的引用，它在调用`postCreate`方法时是可用的。

除`domNode`属性以外，一些widget还定义了一个`ContainerNode`属性。它是一个widget中的子节点的引用，这个widget可能包含了在你的widget定义之外定义的内容或者widget，例如一个声明实例化的widget源代码节点。

>我们的将在另一个教程中讨论`ContainerNode`属性；现在只要知道这个属性的存在和定义（显然，它是定义在所有继承自`dijit/_Container`的widget上）。

##Getters and Setters

除了启动和销毁的基本框架，`_WidgetBase`不仅提供了许多所有widget需要的预定义属性，也提供了一种方式来让你自定义的存取器和标准`get`和`set`方法一起工作来预定义所有的widget。它通过在你的代码中自定义“私有”方法完成，模式如下：

```
// 对于你的widget的“foo”字段:

// 自定义 getter
_getFooAttr: function(){ /* do something and return a value */ },

//     自定义 setter
_setFooAttr: function(value){ /* do something to set a value */ }
```

如果你以这种方式自定义了方法，你随后还可以在你的widget实例上使用标准`_WidgetBase`的`get`和`set`方法。鉴于上面的例子，你可以这样做：

```
// 假定widget实例是 "myWidget":

// 获取 "foo" 的值:
var value = myWidget.get("foo");

// 设置 "foo" 的值:
myWidget.set("foo", someValue);
```

这个标准可以让其他的widget和控制代码来通过一致的方法来作用于一个widget，也让你能够在访问一个字段时执行自定义逻辑（例如修改一个DOM片段，等），还可以让你启动其他任何方法（例如一个事件处理器或通知）。例如，你的widget有一个自定义`value`，你想要通知其他人这个值发生了变化（可能通过你定义过的一个`onChange`方法）：

```
// 假设我们的字段叫 "value":

_setValueAttr: function(value){
    this.onChange(this.value, value);
    this._set("value", value);
},

// a function designed to work with dojo/on
onChange: function(oldValue, newValue){ }
```

如你所见这给了我们一个便利的方式来在自己的widget中自定义存取行为。

>当定义你自己的widget时，每当你需要定义一个自定义属性的检索或修改背后的自定义逻辑时，你应该创建自定义存取方法。当使用你自己的widget时，为了正确地与自定义的存取方法沟通，你应该总使用`get()`和`set()`来进行字段访问。另外，当定义一个自定义setter方法是，你应该总使用内部的`_set`方法来更新内部值，这是为了对接所有widget继承的`dojo/Stateful`的`watch`功能。

##Owning handles

`_WidgetBase`基本框架提供了一个方法来将handles注册给widget自己。这可以用在任何被widget创建的处理上，通常设置在postCreate()监听DOM节点时间.

将handles 附给 widget的方法是`.own()`，它的用法很简单：

```
this.own(
    on(someDomNode, "click", lang.hitch(this, "myOnClickHandler)"),
    aspect.after(someObject, "someFunc", lang.hitch(this, "mySomeFuncHandler)"),
    topic.subscribe("/some/topic", function(){ ... }),
    ...
);
```

在widget基础框架使用`own()`方法的优点在于它是内部的，widget可以跟踪所有的handles，确保在widget销毁时一切都分离和取消订阅——防止任何类型的内存泄漏。

##预定义的属性和事件

最后，`_WidgetBase`提供了一组预定义属性，和相应的getter、setter方法：

 - `id`：识别widget的唯一字符串
 - `lang`：很少用到的字符串，可以重写默认的Dojo语言环境
 - `dir`：用于双向支持
 - `class`：widget的`domNode`的HTML `class` 属性
 - `style`：widget的`domNode`的HTML `style` 属性
 - `title`：最常用，HTML的`title`属性用来原生的提示信息
 - `baseClass`：widget的根CSS类
 - `srcNodeRef`：如果提供了它，原始节点将在它部件化之前存在。注意根据widget的类型（例如模板widget），它可能在`postCreate`之后复原，伴随原始节点的丢弃。

##小结

如你所见，Dijit的`_WidgetBase`基础框架提供了一个坚固的基础来创建和使用widget；一个widget的所有方面（生命周期、DOM节点引用、存取器、预定义属性和事件）都被涉及到了。我们已经看到了widget的`postCreate()`方法如何成为开发自定义widget时最重要的方法，以及编程式实例化widget时如何调用`startup()`方法。我们也涉及到了Dijit的getter/setter框架，还有widget`domNode`属性的重要性。

##资源

 - [创建一个自定义widget](https://dojotoolkit.org/documentation/tutorials/1.10/recipes/custom_widget/)
 - [dijit/_WidgetBase参考指南](https://dojotoolkit.org/reference-guide/1.10/dijit/_WidgetBase.html)