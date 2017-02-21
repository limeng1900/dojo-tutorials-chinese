# 5.10 创建自定义widget

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/recipes/custom_widget/

译者按：本篇不属于系列教程内容，只是作为推荐内容，个人认为比较有意义因而翻译。
___

在这篇教程中，我们将谈及如何利用Dojo碎片和Dijit框架来创建你的自定义widget，特别是如何使用`dijit/_WidgetBase`和`dijit/_TemplatedMixin`来快速并轻易地构建你的widget。

##简介

与Dojo工具集的Dijit框架时一系列叫做widget的图形控件的集合。我们可以使用这些widget构建图形用户界面。

你有时可能需要一个Dojo没有提供的特特定的widget。在这里，你可以使用Dijit的核心来轻松购进这个widget。

##计划

比如说我们有一个JSON格式的数据资源，它列出了一系列作者，比如写了Dojo教程的。它看起来是这样的：

```
[
    {
        "name": "Brian Arnold",
        "avatar": "/includes/authors/brian_arnold/avatar.jpg",
        "bio": "Brian Arnold is a software engineer at SitePen, Inc., ..."
    },
    /* More authors here... */
]
```

我们还想要结果放在页面里，像这样：

```
<body>
    <!-- Headers and whatnot -->
    <h2>Authors</h2>
    <div id="authorContainer">
        <!-- Authors go here! -->
    </div>
</body>
```

我们还希望它能有一些技巧——或许在鼠标移入时改变背景色。最后，我们想要它看起来是这样的：

![这里写图片描述](http://img.blog.csdn.net/20170218115510229?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFpamllZGkxMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##解决方案

我们可以通过以下几个步骤来创建自定义的widget。

 1. 为自定义widget创建一些文件结构
 2. 创建表示一个作者的标签
 3. 扩充我们的作者标签让它成为一个Dijit模板
 4. `declare`创建我们widget类
 5. 适当的样式

### 步骤1：为自定义widget创建一些文件结构

这一步其实是可选的，不过通常认为给你的自定义Dijit工作（或广义上的自定义代码）建立一个合适的文件结构是种很好的实践。在这个例子中，“myApp”是我们所有“自定义”代码的总文件夹——这里“自定义”是为这个app写的特定代码。通用的和第三方库（如dojo、dijit等）会放在“myApp”的兄弟文件夹中。它的名称完全由你决定，但是让名称富有意义，比如你的组织的名称或者widget所属的应用。我们想要将widget组织起来，所以将在“myApp”下创建一个“widget”文件。我们将调用我没的新widget AuthorWidget——它的模块id是`myApp/widget/AuthorWidget`。widget常常会使用外部资源，所以我们将在“widget”文件夹下添加一些文件夹来组织它们—— css、imagets和templates。我们最后的文件结构如下：

![这里写图片描述](http://img.blog.csdn.net/20170218145126046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFpamllZGkxMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们还没有实际地在自定义控件创建任何文件——现在只是一些层次结构。

### 步骤2：创建表示一个作者的标签

现在我们有了一些结构来储存碎片，再来创建一些简单标签来表示一个单独的作者。对于你的第一个widget，可能只是最简单地建立一个基本页面，再直接放入例子的值。

当你制作模板时，你应该总是递减一个父级包装元素来包含其他所有的元素。这个元素可以是任何你想要的，但是重点是你只能有一个跟元素。对于我们的数据，用一个`div`作为包装元素。我们将使用一个`H3`元素来放入我们的作者姓名，图像使用一个`img`元素，个人简介放在一个`p`元素里。

```
<div>
    <h3>Brian Arnold</h3>
    <img src="/includes/authors/brian_arnold/avatar.jpg">
    <p>Brian Arnold is a software engineer at SitePen, Inc., ...</p>
</div>
```

### 步骤3：扩充我们的作者标签让它成为一个Dijit模板

当使用`dijit/_TemplatedMixin`时，你可以用不同的方法调整你的标签：

 - 你可以让你的widget的值自动插入
 - 你可以将模板中的元素指定为Attach Point，来提供widget中节点的编程引用。
 - 你可以给特定节点的DOM事件设置方法来调用

对于我们来说，现在还不用操心事件——不过我们想要利用一些自动插入。在`myApp/widget/templates/`下创建一个名叫`AuthorWidget.html`的文件。以上只是基本的标签定义，但是做了一些简单添加。

```
<div>
    <h3 data-dojo-attach-point="nameNode">${name}</h3>
    <img class="${baseClass}Avatar" src="" data-dojo-attach-point="avatarNode">
    <p data-dojo-attach-point="bioNode">${!bio}</p>
</div>
```

这里有几件事情要注意：

 - 我们可以使用`${attribute}`语法来直接插入一些值，比如我们的名字。
 - 我们也可以使用`${!attribute}`语法来直接给widget插入一些值，比如我们用在个人简介的。`${attribute}`和`${!attribute}`之间的主要区别是我们的个人简介内容是HTML，而我们想要避免`dijit/_TemplatedMixin`对插入的内容执行自动转义。
 - 所有基于`dijit/_WidgetBase`的widget都默认有一个`baseClass`属性，通过这个我们可一个给头像提供一个自定义类。
 - 我们已经给所有节点提供了一个附加点，就是说在我们widget代码中，我们可以使用名称来直接引用节点。它就好像在做`getElementById`类型的工作而不需要ID，我们提前设置了引用—— 所以在AuthorWidget的一个实例中，我们可以使用`myAuthor.nameNode`来直接为widget引用H3的DOM节点。

你可能发现我们没有直接设置头像的资源。如果我们有一个作者没有指定头像呢？又不想显示一个破裂图。我们将在创建widget时为它处理一个默认值，这正是下面要做的。

### 步骤4：使用 dojo/_base/declare 创建我们widget类

这里在我们上面的文件结构中，我们将要在`widget`文件夹中创建一个名叫`AouthorWidget.js`的文件。我们还将添加一个默认的头像图片。文件结构开始被填充起来。之后，它将更加丰满。我们会在这里做大量的工作。

![这里写图片描述](http://img.blog.csdn.net/20170219212639836?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFpamllZGkxMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

现在我们可以简单地构建我们widget。把下面代码放在你的`AuthorWidget.js`文件中。

```
// myApp/widget/AuthorWidget.js
define(["dojo/_base/declare","dijit/_WidgetBase", "dijit/_TemplatedMixin"],
    function(declare, _WidgetBase, _TemplatedMixin){
        return declare([_WidgetBase, _TemplatedMixin], {
        });
}); // and that's it!
```

使用`declare`我们可以由`dijit/_WidgetBase`和`dijit/_TemplatedMixin`轻松地创建我们的自定义AuthorWidget。现在，它还跑不起来。我们需要为我们的widget添加几个自定义属性，还有为实例化时可能没指定值的属性设置一些默认值。下面是我们的定义：

```
define([
    "dojo/_base/declare",
    "dojo/_base/fx",
    "dojo/_base/lang",
    "dojo/dom-style",
    "dojo/mouse",
    "dojo/on",
    "dijit/_WidgetBase",
    "dijit/_TemplatedMixin",
    "dojo/text!./templates/AuthorWidget.html"
], function(declare, baseFx, lang, domStyle, mouse, on, _WidgetBase, _TemplatedMixin, template){
    return declare([_WidgetBase, _TemplatedMixin], {
        // 我们的作者的一些默认值
        // 这些将映射你传递给构造器的内容
        name: "No Name",
        // 使用 require.toUrl， 我们可以得到一个指向AuthorWidget位置的路径
        // 以防万一，我们需要有一个默认的头像
        avatar: require.toUrl("./images/defaultAvatar.png"),
        bio: "",

        // 我们的模板 - important!
        templateString: template,

        // 模板中应用到根节点的类
        baseClass: "authorWidget",

        // 背景动画的引用
        mouseAnim: null,

        // 背景动画的颜色
        baseBackgroundColor: "#fff",
        mouseBackgroundColor: "#def"
    });
});
```

这里发生了几件事，我们来分解一下。

 - 我们从作者相关的一些属性开始——`name`、`bio`、`avatar`——设置默认值。通过使用`require.toUrl`，我们可以得到AuthorWidget所在的路径，并访问它下面的图像文件夹。
 - 使用`templateString`属性和`dojo/text`，我们指定模板的内容。
 - 设置我们的`baseClass`。它适用于我们的根节点，对应这个案例是`div`。
 - 为动画设置引用，还有一对动画的颜色

现在很好了，到这里它已经可以作为一个基本信息展示的widget运行了。不过我们可以添加一些方法来让它更安全。我们将添加：

 - `postCreate`中的一些逻辑（来自`_WidgetBase`最常见的声明周期方法）
 - `avatar`属性的一个自定义属性设置器
 - 一个改变背景颜色的功能函数

让我们来一个一个看。

我们大部分的工作在`postCreate`方法中完成。当widget的DOM结构准备好时就会调用它，不过会在插入页面之前。它是放实例化代码最好的位置。

```
postCreate: function(){
    // Get a DOM node reference for the root of our widget
    var domNode = this.domNode;

    // Run any parent postCreate processes - can be done at any point
    this.inherited(arguments);

    // Set our DOM node's background color to white -
    // smoothes out the mouseenter/leave event animations
    domStyle.set(domNode, "backgroundColor", this.baseBackgroundColor);
    // Set up our mouseenter/leave events
    // Using dijit/Destroyable's "own" method ensures that event handlers are unregistered when the widget is destroyed
    // Using dojo/mouse normalizes the non-standard mouseenter/leave events across browsers
    // Passing a third parameter to lang.hitch allows us to specify not only the context,
    // but also the first parameter passed to _changeBackground
    this.own(
        on(domNode, mouse.enter, lang.hitch(this, "_changeBackground", this.mouseBackgroundColor)),
        on(domNode, mouse.leave, lang.hitch(this, "_changeBackground", this.baseBackgroundColor))
    );
}
```

这里我们基于`baseBackgroundColor`属性设置一些样式，然后设置一些 onmouseenter/onmouseleave 事件，这样鼠标移入DOM节点时，就会调用自定义`_changeBackground`函数。如下：

```
_changeBackground: function(newColor) {
    // 如果我们有一个动画, 先停止它
    if (this.mouseAnim) {
        this.mouseAnim.stop();
    }

    // 开启一个新动画
    this.mouseAnim = baseFx.animateProperty({
        node: this.domNode,
        properties: {
            backgroundColor: newColor
        },
        onEnd: lang.hitch(this, function() {
            // 清除mouseAnim 属性
            this.mouseAnim = null;
        })
    }).play();
}
```

>这个方法为什么叫做`_changeBackground`而不是`changeBackground`？前置的下划线表示使用者应该把它当做一个私有方法，而不是可以直接拿来用。这是一种常见的方法，在对象或方法前加下划线来表示他们不应该直接使用。它不直接阻止用户使用这些代码，而是含蓄地暗示“嗨，这不是通用的”。

我们查看`mouseAnim`属性来看是否已经有一个动画，如果有，安全起见就调用`stop`来停止它。然后我们设置一个新的动画并把它存进`mouseAnim`，随后开始播放。这个例子和动画教程中展示的很像，只是用了一些不同的颜色。

最后，还记得我们之前考虑过如果一个用户没有头像，就设置一个默认的吧？我们可以为属性设置一个自定义setter函数，在设置值的时候将自动被调用，还有当创建widget时或者调用`myWidget.set("avatar", somePath)`。方法的命名也特殊，对应属性的名字——在这个例子中，对于`avatar`，我们命名为`_setAvatarAttr`。

```
_setAvatarAttr: function(imagePath) {
    // 我们只在它是一个非空字符串时设置它
    if (imagePath != "") {
        // 把它存到widget实例 - 注意我们使用了_set 来支持让任何人使用widget的Watch功能来查看值的变化
        this._set("avatar", imagePath);

        // 使用 avatarNode attach point, 设置它的 src 值
        this.avatarNode.src = imagePath;
    }
}
```

Dojo1.6开始，所有的`dijit/_WidgetBase`widget的继承链都包含`dojo/Stateful`，就是说用户可以主动观察值的变化。我们在setter中使用`_set`来确保所有的`watch`调用都正确触发，然后使用我们的`avatarNode`附加点来设置图像的`src`从而设置值。通过在外围检查字符串是否非空，我们避免了有头像属性但是值为空字符串的情况。这样，当没有值时就使用默认图像。

要使用这个widget，我们这么做：

```
<div id="authorContainer"></div>
```

```
require(["dojo/request", "dojo/dom", "dojo/_base/array", "myApp/widget/AuthorWidget", "dojo/domReady!"],
    function(request, dom, arrayUtil, AuthorWidget){
    // 加载我们的作者
    request("myApp/data/authors.json", {
        handleAs: "json"
    }).then(function(authors){
        // 得到容器的引用
        var authorContainer = dom.byId("authorContainer");

        arrayUtil.forEach(authors, function(author){
            // 创建widget并放置它
            var widget = new AuthorWidget(author).placeAt(authorContainer);
        });
    });
});
```

有了所有这些，我们就有了可以运行的！不过，如你所见它还不够漂亮。

![这里写图片描述](http://img.blog.csdn.net/20170221154834242?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFpamllZGkxMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/recipes/custom_widget/demo/unstyled.html)

### 步骤5：合适的样式

使用`dijit/_WidgetBase`的好处，是它给了我们一个`baseClass`来处理样式。使用这个，我们可以创建一些相当简单的演示。在我们的AuthorWidget下的文件夹中有一个是给CSS的，让我们在这创建一个`AuthorWidget.css`文件。

```
/* myApp/widget/css/AuthorWidget.css */
.authorWidget {
    border: 1px solid black;
    width: 400px;
    padding: 10px;
    overflow: hidden; /* I hear this helps clear floats inside */
}

.authorWidget h3 {
    font-size: 1.5em;
    font-style: italic;
    text-align: center;
    margin: 0px;
}

.authorWidgetAvatar {
    float: left;
    margin: 4px 12px 6px 0px;
    max-width: 75px;
    max-height: 75px;
}
```

由于我们知道`baseClass`是`authorWidget`，我们就可以这么做。还有如果你还记得，在模板里，我们给头像设置了一个`${baseClass}Avatar`类，所以在样式中可以使用`authorWidgetAvatar`。

现在，有了这些，我们只需要将CSS添加到页面的`head`中，就有了一个更漂亮的作者列表。

![这里写图片描述](http://img.blog.csdn.net/20170221155635494?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGFpamllZGkxMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/recipes/custom_widget/demo/index.html)

## 概要
如你所见，使用`dijit/_WidgetBase`和`dijit/_TemplatedMixin`让创建自定义widget非常容易。我们可以快速建立一个模板，在做一点工作，就可以创建我们基于`dijit/_WidgetBase`和`dijit/_TemplatedMixin`的AuthorWidget 类。

值得注意的是Dijit中的大多数widget同样是使用这些工具构建的，我们这里只简单涉及一些皮毛。下面的链接可以获取更多信息！

## 资源
- [dijit/_WidgetBase](https://dojotoolkit.org/reference-guide/1.10/dijit/_WidgetBase.html)
- [dijit/_TemplatedMixin](https://dojotoolkit.org/reference-guide/1.10/dijit/_TemplatedMixin.html)
- [Tutorial on dojo/declare](https://dojotoolkit.org/documentation/tutorials/1.10/declare/)
- [Dojo Reference Guide: Writing Your Own Widget](https://dojotoolkit.org/reference-guide/1.10/quickstart/writingWidgets.html)