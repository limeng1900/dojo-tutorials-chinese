# 3.7 NodeList扩展
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/nodelist_extensions/index.html
----------

对于`dojo/query`使用的`NodeList`集合Dojo有一系列的扩展。本教程中，我们来看看有什么可用的拓展功能和如何加以利用。

##入门
在前面[`dojo/query`](https://dojotoolkit.org/reference-guide/1.10/dojo/query.html)的教程中，我们已经看到如何得到一个匹配查询或选择器的节点集合，还有如何使用[`dojo/NodeList`](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList.html)的方法来使用这些节点。先让我们快速回顾一下。下面是将要用到的标签：
`dojo/NodeList`对象不同于DOM的[`NodeList`](https://developer.mozilla.org/en-US/docs/DOM/NodeList)对象。Dojo的`NodeList`是带有额外方法的数组的一个实例。ES5迭代方法确保甚至可以用在非ES5环境里，本教程中你将看到有各种模块可以扩展`dojo/NodeList`，给它增加很多有用的方法。

```
<button type="button" id="btn">Pick out fresh fruits</button>

<h3>Fresh Fruits</h3>
<ul id="freshList"></ul>

<h3>Fruits</h3>
<ul>
    <li class="fresh">Apples</li>
    <li class="fresh">Persimmons</li>
    <li class="fresh">Grapes</li>
    <li class="fresh">Fresh Figs</li>
    <li class="dried">Dates</li>
    <li class="dried">Raisins</li>
    <li class="dried">Prunes</li>
    <li class="fresh dried">Apricots</li>
    <li class="fresh">Peaches</li>
    <li class="fresh">Bananas</li>
    <li class="fresh">Cherries</li>
</ul>
```
`dojo/query`实例，创建一个按钮的点击事件：
```
require(["dojo/query", "dojo/domReady!"], function(query){
    query("#btn").on("click", function(){
        var nodes = query("li.fresh");
        nodes.on("click", function(){
            alert("I love fresh " + this.innerHTML);
        });
    });
});
```
`query("li.fresh")`返回一个`NodeList`，它是一个标准的JavaScript数组，带有可以更容易操作DOM节点集合的方法。因为每个`query`调用都返回一个`NodeList`，我们可以用链式方法让它更简单（不用再一次又一次输入`var nodes`）：

```
require(["dojo/query", "dojo/domReady!"], function(query){
    query("#btn").on("click", function(){
        query("li.fresh").on("click", function(event){
            alert("I love fresh " + event.target.innerHTML);
        });
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/nodelist_extensions/demo/nodelist_extensions-queryRecap.html)

由于没有地方可以添加日志语句或调试的断点，调用链式方法的故障排除会比较困难。需要把链式方法拆成分离的步骤来检查每个方法的返回值。

##NodeList更多用途
“获得一些节点然后对它们做一些事”的模式非常普遍，`NodeList`的很多潜在特性解决了Dojo的模块化性质和“可组合”的功能之间的冲突。因此，在Dojo和DojoX中，有很多的NodeList扩展模块可以加载来给它添加新功能。让我们看一看。

##关于文档
在API viewer中展示了[NodeList](https://dojotoolkit.org/api/1.10/dojo/NodeList.html)对象的声明在Dojo和DojoX的扩展模块的全部扩展方法。尽管资源模块是可以识别的，但是它更加“复杂”。另外扩展这个对象的个人模块本质上是一片空白。在参考指南里，每个模块都有它自己的页面（例如：[`dojo/NodeList-data`](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList-data.html)页面），如此各个模块提供什么方法就很清楚。

##操作样式和DOM
Dojo1.7之前，基础`NodeList`具备一些DOM方法，例如`addClass`、`removeClass`、`attr`、`style`、`empty` 和 `place`。随着AMD和Dojo 1.7+的出现，这些方法移到了[`dojo/NodeList-dom`](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList-dom.html)。这里有一个关于你如何使用这个模块的示例：

```
require(["dojo/query", "dojo/NodeList-dom"], function(query){
    query("li.fresh")
        .addClass("fresher")
        .attr("title", "freshened")
        .style("background", "lightblue")
        .on("click", function(){
            alert("I love fresh " + this.innerHTML);
        });
});
```
简单的加载`dojo/NodeList-dom`模块可以将这些方法添加到`NodeList`。它们的使用跟`dojo/dom`和相关模块一样。

##动画元素
[`dojo/NodeList-fx`](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList-fx.html)模块参数`NodeList`带有一系列的方法可以用来将Dojo的特效系统应用到节点集合中。这些方法和非NodeList的方法对应，如果你不熟悉的话先看看 [Dojo Effects](https://dojotoolkit.org/documentation/tutorials/1.10/effects/) 和 [Animation](https://dojotoolkit.org/documentation/tutorials/1.10/animation/) 教程。
在这个例子中，我们将使用之前的list和点击时执行以下代码的一个按钮：
```
require(["dojo/query", "dojo/NodeList-fx", "dojo/domReady!"], function(query){
    query("#btn").on("click", function(){
        query("li.fresh")
            .slideTo({
                left: 200, auto: true
            })
            .animateProperty({
                properties: {
                    backgroundColor: { start: "#fff", end: "#ffc" }
                }
            })
            .play();
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/nodelist_extensions/demo/nodelist_extensions-fx.html)

不像大多数的`Nodelist`方法，`NodeList-fx`方法默认返回一个动画对象，和常见的`Nodelist`的链式行为相矛盾。这是因为Dojo的动画函数通常返回一个动画对象，需要你对用对象的`play`方法来启动动画。在传递给函数的对象里设置`auto: true`可以使`NodeList-fx`的方法自动播放并且返回一个`Nodelist`，例如上面`slideTo`的调用。

##联合元素与数据
[`dojo/NodeList-data`](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList-data.html)模块通过`data`方法增加了一个可以将任意数据附加到元素的机制。这有一个例子，在每次点击元素时将一个`Data`对象存储到元素上。
```
require(["dojo/query", "dojo/NodeList-data", "dojo/domReady!"], function(query, NodeList){
    function mark(){
        var nodeList = new NodeList(this);        // make a new NodeList from the clicked element
        nodeList.data("updated", new Date());    // update the 'updated' key for this element via the NodeList
    }

    query("li")                            // get all list items
        .data("updated", new Date())    // set the initial data for each matching element
        .on("click", mark);                // add the event handler

    query("#btn").on("click", function(){
        query("li").data("updated").forEach(function(date){
            console.log(date.getTime());
        });
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/nodelist_extensions/demo/nodelist_extensions-data.html)

这里我们做了三件事：
 - 将初始化的`Date`对象关联到每个元素。
 - 设置一个`click`处理器来调用`mark()`函数
 - 设置一个按钮来查看每一项的数据

在点击事件处理器里，我们仍需要一个`NodeList`来获得被点击元素的事件data属性，因此我们将点击过的元素创建一个新的`NodeList`。在被点击的元素中存在的`Date`对象随后会被新的替代。
使用`NodeList-data`时，**特别重要**的是当你从DOM移除节点时，你需要调用`NodeList`的`removeData`方法。如果不这么做，你的应用会**内存泄漏**，因为数据实际并不存储在元素上，当节点被垃圾回收时它无法进行回收。

##DOM遍历
 `dojo/NodeList-traverse`模块为NodeList添加的方法可以让你轻易的在DOM上查找引用节点的父节点、兄弟节点和子节点。
 为了展示我们使用一个长长的水果分类列表。其中一些水果已经标记了可口（使用`yum`类），然后我们想要：
 1. 高亮它们。
 2. 在列表的开头显示这里面有好东西。

使用`NodeList`、`dojo/NodeList-traverse`、`dojo/NodeList-dom`提供的方法，如下是一个快捷的方式：
```
require(["dojo/query", "dojo/NodeList-traverse", "dojo/NodeList-dom",
        "dojo/domReady!"], function(query){
    query("li.yum")                // get LI elements with the class 'yum'
        .addClass("highlight")    // add a 'highlight' class to those LI elements
        .closest(".fruitList")    // find the closest parent elements of those LIs with the class 'fruitList'
        .prev()                    // get the previous sibling (headings in this case) of each of those fruitList elements
        .addClass("happy")        // add a 'happy' class to those headings
        .style({backgroundPosition: "left", paddingLeft: "20px"}); // add some style properties to those headings
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/nodelist_extensions/demo/nodelist_extensions-traverse.html)

这个示例的关键是使用`clone`方法创建原始元素的副本。通过`NodeList-traverse`方法，`clone`返回一个新的NodeList包括所有新克隆的元素，这些元素随后修改和追加到DOM。如果没有创建这个副本，原始的元素会修改并移进来。

##内容注入进阶
`[dojo/NodeList-html](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList-html.html)`模块给`NodeList`带来了`[dojo/html::set()](https://dojotoolkit.org/reference-guide/1.10/dojo/html.html)`的高级功能。这里有一个简单的例子，它用`dijit/form/CheckBox`把一个简单的列表转变成一个复选框列表：
```
require(["dojo/query", "dojo/_base/lang", "dijit/form/CheckBox", "dojo/NodeList-html", "dojo/domReady!"],
function(query, lang){
    var demo = {
        addCheckboxes: function(q) {
            query(q).html('<input name="fruit" value="" data-dojo-type="dijit/form/CheckBox">', {
                onBegin: function(){
                    var label = lang.trim(this.node.innerHTML),
                        cont = this.content + label;
                    cont = cont.replace('value=""', 'value="'+lang.trim(this.node.innerHTML) + '"');

                    this.content = cont;
                    return this.inherited("onBegin", arguments);
                },
                parseContent: true
            });
        }
    }

    query("#btn").on("click", lang.hitch(demo, "addCheckboxes", "li.alkaline"));
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/nodelist_extensions/demo/nodelist_extensions-htmlSet.html)

由于其他NodeList方法提供了丰富的功能，尤其是在`NodeList-manipulate`里面，你可能不会经常使用`NodeList-html`模块，甚至可能不用。这里提它主要是因为对于某一类问题它还是能够作为专业的工具来解决，相比之下其它方式会困难的多。

##小结
`NodeList`模块扩展了已有的`NodeList`API，但它没有让你的代码变得臃肿。在你的代码使用这些扩展可以更加高效的处理DOM。