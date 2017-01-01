# 3.1 Dojo DOM函数

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/index.html

----------

在本教程中，你将学到如何通过使用Dojo来轻松地浏览器操作DOM。只需要基本DOM知识和几个Dojo函数，你就能够飞速且高效地创建、读取、更新和删除页面里的元素。

##入门
对于基于浏览器的JavaScript而言，文档对象模型（DOM）就是一块可以画的玻璃板，我们把内容和用户界面放在上面展示给用户。如果我们想要在浏览器加载HTML时对它进行扩展、替换或者添加，就要用JavaScript和DOM来实现。Dojo旨在通过提供几个便利的函数使DOM相关的工作更加容易和高效，它可以填补跨浏览器不兼容的问题并让常见操作更加简单。

下面通过一个包含无序列表的简单页面来学习这些函数：

```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Demo: DOM Functions</title>
        <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
            data-dojo-config="async: true">
        </script>
        <script>
            require(["dojo/domReady!"], function() {

            });
        </script>
    </head>
    <body>
        <ul id="list">
            <li id="one">One</li>
            <li id="two">Two</li>
            <li id="three">Three</li>
            <li id="four">Four</li>
            <li id="five">Five</li>
        </ul>
    </body>
</html>
```
这个页面已经有Dojo脚本标签了，你应该认识`require`代码块。所有操作DOM的代码必须等DOM准备好了才能执行。

##检索

想要使用DOM，我们首先需要知道如何从DOM获取元素。最简单的方法是使用`dojo/dom`的`byId`方法。当你向`dom.byId`传递一个ID，你将获得带有这个ID的DOM节点对象。如果没有匹配到节点，将返回空对象。
它等同于使用`document.getElementById`，但是有两个优点：一是它更加简短，在一些浏览器`getElementById`实现带bug时它也能正常工作。`dom.byId`的另一个优秀特性是当给它传递一个节点时，它会立刻返回那个节点。这可以帮助我们创建同时使用字符串和DOM节点的API。下面看一个示例：

```
// Require the DOM resource
require(["dojo/dom", "dojo/domReady!"], function(dom) {

    function setText(node, text){
        node = dom.byId(node);
        node.innerHTML = text;
    }

    var one = dom.byId("one");
    setText(one, "One has been set");
    setText("two", "Two has been set as well");

});
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/demo/byid.html)

 `setText` 函数给节点设置文本，不过因为它传递节点参数给`dom.byId`，所以它输入一个节点ID的字符串或者一个DOM节点都可以。

## 创建
你经常会做的另一件事就是创建元素。Dojo不会禁止你用原生的`document.createElement`方法来创建元素，不过用它创建元素并设置必须的属性和特性非常啰嗦。而且跨浏览器也让我们有了更多的理由使用更便利的`dojo/dom-construct`的`create`方法来设置属性。

`domConstruct.create`参数如下：节点名称字符串，节点属性对象，可选的父节点或兄弟节点，相对父节点和兄弟节点的一个可选的位置（默认为“last”），它返回一个新的DOM元素节点。下面先看示例;

```
require(["dojo/dom", "dojo/dom-construct", "dojo/domReady!"],
    function(dom, domConstruct) {

        var list = dom.byId("list"),
            three = dom.byId("three");

        domConstruct.create("li", {
            innerHTML: "Six"
        }, list);

        domConstruct.create("li", {
            innerHTML: "Seven",
            className: "seven",
            style: {
                fontWeight: "bold"
            }
        }, list);

        domConstruct.create("li", {
            innerHTML: "Three and a half"
        }, three, "after");
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/demo/create.html)

创建了一个内容为“Six”的列表项，并放进列表里。另一个新加列表项的内容是“Seven”，它的类名属性设为“seven”，它的样式是字体加粗。最后创建一个内容为“Three and a half”的列表项，来插入到ID是“three”的列表项后面。
你什么时候可以设置一个元素的内容属性`innerHTML`来创建元素？如果你已经有一个HTML字符串式的内容，设置`innerHTML`会比较快。无论如何，`domConstruct.create`可以用来创建一个元素但不用马上把它放在DOM里，或者当你想要在不干扰其他兄弟节点的情况下插入或追加一个新元素时使用它。

## 布局

如果你已经有一个节点并且打算布置它，你需要使用`domConstruct.place`。参数如下：用来放置的DOM节点或者节点的字符串ID、一个作为引用的DOM节点或者节点的字符串ID、可选的位置字符串默认为“last”。跟`domConstruct.create` 里很像，其实`domConstruct.create`底层用的就是`domConstruct.place`。示例中，我们往页面添加了几个按钮：

```
<button id="moveFirst">The first item</button>
<button id="moveBeforeTwo">Before Two</button>
<button id="moveAfterFour">After Four</button>
<button id="moveLast">The last item</button>
```
这个例子定义了一个函数，使用`domConstruct.place`来移动列表的第三个节点：

```
require(["dojo/dom", "dojo/dom-construct", "dojo/on", "dojo/domReady!"],
    function(dom, domConstruct, on){

        function moveFirst(){
            var list = dom.byId("list"),
                three = dom.byId("three");

            domConstruct.place(three, list, "first");
        }

        function moveBeforeTwo(){
            var two = dom.byId("two"),
                three = dom.byId("three");

            domConstruct.place(three, two, "before");
        }

        function moveAfterFour(){
            var four = dom.byId("four"),
                three = dom.byId("three");

            domConstruct.place(three, four, "after");
        }

        function moveLast(){
            var list = dom.byId("list"),
                three = dom.byId("three");

            domConstruct.place(three, list);
        }

        // Connect the buttons
        on(dom.byId("moveFirst"), "click", moveFirst);
        on(dom.byId("moveBeforeTwo"), "click", moveBeforeTwo);
        on(dom.byId("moveAfterFour"), "click", moveAfterFour);
        on(dom.byId("moveLast"), "click", moveLast);
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/demo/place.html)

位置参数的值可以是 "before"、"after"、"replace"、"only"、"first"和"last"。请看 [reference guide for domConstruct.place](https://dojotoolkit.org/reference-guide/1.10/dojo/dom-construct.html#dojo-dom-construct-place) 了解每项的效果。

在这个简单的例子里， domConstruct.place比原生的`parentNode.appendChild(node)`做的更多一点。它的价值在于不管它引用的是一个父节点或者兄弟节点，都只用同一个API就能轻松地指定位置。

## 销毁
通常你会创建节点，但是偶尔你也要移除节点。在Dojo里有两种方式：`domConstruct.destroy`会销毁节点和它所有的子节点，`domConstruct.empty`将销毁给出的节点的子节点。它们都将一个DOM节点或者一个节点的字符串ID作为唯一参数。下面我们将往页面里添加两个按钮：

```
<button id="destroyFirst">Destroy the first list item</button>
<button id="destroyAll">Destroy all list items</button>
```

```
function destroyFirst(){
    var list = dom.byId("list"),
        items = list.getElementsByTagName("li");

    if(items.length){
        domConstruct.destroy(items[0]);
    }
}
function destroyAll(){
    domConstruct.empty("list");
}

// Connect buttons to destroy elements
on(dom.byId("destroyFirst"), "click", destroyFirst);
on(dom.byId("destroyAll"), "click", destroyAll);
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/demo/destroy.html)
第一个按钮每次点击会销毁列表中的第一项，第二个完全清空列表。

## 小结
目前，我们有一个很全面的工具集，可以进行简单的DOM操作，从节点的创建、移动到销毁，但它们都只在一个节点上工作。如果你想要处理的节点没有ID呢？下一个教程关于`dojo/query`，基于CSS选择器来查找和处理节点。
