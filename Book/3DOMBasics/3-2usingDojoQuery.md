# 3.2 使用dojo/query

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/using_query/index.html

----------

本教程中，我们将学习DOM查询和如何使用`dojo/query`来选择节点。

##入门
在使用DOM时，快速和高效地检索节点很重要。我们之前提到了[一个选择](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/#byId)：`dom.byId`。但是，在应用中用单一的ID来查找所有的兴趣节点是不现实的。只用ID在查找和操作多节点时效率也很低。幸好，还有另一种解决方案：`dojo/query`。`dojo/query`模块使用类CSS查询（你在样式表里使用的）来检索一个节点列表，它也支持高级的CSS3选择器。

##查询
我们用下面的HTML来展示最常用的查询，里面主要网址链接的列表：

```
<ul id="list">
    <li class="odd">
        <div class="bold">
            <a class="odd">Odd</a>
        </div>
    </li>
    <li class="even">
        <div class="italic">
            <a class="even">Even</a>
        </div>
    </li>
    <li class="odd">
        <a class="odd">Odd</a>
    </li>
    <li class="even">
        <div class="bold">
            <a class="even">Even</a>
        </div>
    </li>
    <li class="odd">
        <div class="italic">
            <a class="odd">Odd</a>
        </div>
    </li>
    <li class="even">
        <a class="even">Even</a>
    </li>
</ul>

<ul id="list2">
    <li class="odd">Odd</li>
</ul>
```
第一件要做的事是获得整个列表的处理器。跟前面一样，可以用`dom.byId`，但你也可以使用`query`。初看这个方法不是很有用，我们会从这个例子开始：

```
// require the query, dom, and domReady modules
require(["dojo/query", "dojo/dom", "dojo/domReady!"], function (query, dom) {
    // 检索带有“list”ID的一组节点
    var list = query("#list")[0];
})
```
通过前置标识符“#”来告诉`query`用ID属性查找节点。这个规定和CSS相似。记住一件事：`query`总是返回一个数组。数组以后再细说，由于这里通过ID list只得到一个节点（也应该只有一个），就直接从数组取出这个元素。

通过ID获取节点很棒，但是并没有比`dom.byId`更强大。然而，`query`也可以通过类名来选择节点。我们打算检索只带“odd”类的节点：

```
// retrieve an array of nodes with the class name "odd"
var odds = query(".odd");
```
我们通过前置“.”的标识符告诉`query`查找className属性中含有该标识的节点，还是和CSS一样。示例中，`query`将返回一个包含4个`<li>`和3个`<a>`的数组。

##限制查询
你可以已经发现前面的例子中`odd`同时包含多个list的节点。如果只想要第一个list的odd节点，可以使用两种方式：

```
// retrieve an array of nodes with the class name "odd"
// from the first list using a selector
var odds1 = query("#list .odd");

// retrieve an array of nodes with the class name "odd"
// from the first list using a DOM node
var odds2 = query(".odd", dom.byId("list"));
```
两种通过不同的方法实现：第一种使用选择器语法让query引擎限制获取的结果，第二种将query引擎的作用域限制在一个特定的DOM节点。

当`query`不带第二个参数执行的时候，它将搜索全部的DOM结构的全部节点。第二个参数指定为一个DOM节点时，查询被限制在该节点和其子节点中。

如果你的DOM相对较小，比如跟例子中一样，可以省略第二个参数。然而，对于有着更大DOM结构的页面，最好使用第二个参数来限制`query`的作用域。在指定部分执行查询比搜索全部文档要快的多。

后面的示例中，我们会省略第二个范围参数，但是你用`query`时要记住前面的话，保持你的检索快速简洁来提供更快的代码和更好的用户体验。

##进阶部分
前面的查询结果混合了`<li>`和`<a>`节点，如果只想要`<a>`呢？你可以组合标签名和类名：

```
var oddA = query("a.odd");
```
代替分离的标识符，你可以组合标识符来指向更明确的节点；上面的组合类名在跨浏览器的样式表中有不同的效果，不过`query`能正常工作。

`query`还有另一个跨浏览器的选择器“>”，但不是样式表所有部分都支持。它只在第一个选择器下一级查找第二个：

```
//检索一组a元素，它的祖先中有li
var allA = query("li a");
// 检索一组a元素，它有li作为直接父节点
var someA = query("li > a");
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/using_query/demo/queries.html)

`allA`将包含全部的6个`<a>`，`someA`只有2个`<a>`。任何选择器都可以放在“>”的两边，包括类选择器。这里我们只涉及几个常见的选择器，`query`是完全遵循CSS3的，并且可以接受[更多选择器](https://dojotoolkit.org/reference-guide/1.10/dojo/query.html#standard-css3-selectors)，你可以自己试试。

##NodeList
如前面提到的，`query`返回一个匹配选择器的节点数组；这个数组实际上是一个[`dojo/NodeList`](https://dojotoolkit.org/reference-guide/1.10/dojo/NodeList.html)，它还有操作节点的方法。前面的例子已经用了几个方法，不过我们在看一个你更可能在应用里用到的。这一系列的示例使用下面的标记：
```
<div id="list">
    <div class="odd">One</div>
    <div class="even">Two</div>
    <div class="odd">Three</div>
    <div class="even">Four</div>
    <div class="odd">Five</div>
    <div class="even">Six</div>
</div>
```
`NodeList`拥有相当于Dojo数组辅助方法的方法。其中一个是`forEach`，会对数组里每一个节点执行函数：
```
// 等待DOM加载
require(["dojo/query", "dojo/dom-class", "dojo/domReady!"],
    function(query, domClass) {

        query(".odd").forEach(function(node, index, nodelist){
            // 为查询返回的数组中的每一个节点执行以下代码
            domClass.add(node, "red");
        });

});
```
传递给`forEach`的函数是一个回调，调用给数组的每一项，参数如下：当前它所在的节点，节点的索引和迭代的`NodeList`。对于大部分的开发者，第三个参数可以忽略；但是对于数组没有存储在变量中的例子，第三个参数可以用来获取数组的其他项。`forEach`方法也接收第二个参数来指定回调调用的作用域。

`NodeList`定义的其他数组帮助函数是`map`、`filter`、`every`和`some`。这些函数都返回一个`NodeList`，除了`every`和`some`返回布尔值。

`NodeList`还有几个拓展模块来添加额外的方法。类和样式帮助方法放在`dojo/NodeList-dom`模块里。`dojo/NodeList-dom`提供对应各种DOM方法的遍历方法， 这样前面的例子就可以简化成：

```
require(["dojo/query", "dojo/NodeList-dom", "dojo/domReady!"], function(query) {
    // 为所有匹配".odd"选择器的节点添加"red"类
    query(".odd").addClass("red");
    // 为所有匹配".even"选择器的节点添加"blue"类
    query(".even").addClass("blue");
});
```
在`NodeList`里对每个节点执行DOM方法，并且返回一个`NodeList`支持链接写法：
```
// 为所有匹配".odd"选择器的节点移除"red"类并添加"blue"类
query(".odd").removeClass("red").addClass("blue");
```
`dojo/NodeList-dom`定义的其它DOM方法有`style`、`toggleClass`、 `replaceClass`、 `place`和`empty`。这些方法也都返回一个`NodeList`：

```
// 将每一个匹配“.even”选择器的节点的font color变为“white”并添加“italic”类名
query(".even").style("color", "white").addClass("italic");
```

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/using_query/demo/nodelist.html)

##事件
`NodeList`提供的另一个便利的方法是`on`来连接DOM事件。虽然DOM事件将在下一个教程里涉及，我们会先涉及`NodeList`的`on`方法的语法。也要记住，虽然这是个方便的语法，这个方式不应该用在包含大量节点的`NodeList`，应该用一种叫做事件委托的技术来替代，见[事件教程](https://dojotoolkit.org/documentation/tutorials/1.10/events/)。

```
<button class="hookUp demoBtn">Click Me!</button>
<button class="hookUp demoBtn">Click Me!</button>
<button class="hookUp demoBtn">Click Me!</button>
<button class="hookUp demoBtn">Click Me!</button>
<script>
    // 在使用之前先等待DOM加载完成
    require(["dojo/query", "dojo/domReady!"], function(query) {
        query(".hookUp").on("click", function(){
            alert("This button is hooked up!");
        });
    });
</script>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/using_query/demo/events.html)

`on`方法附在查询返回的每一个节点上。

##小结
如你所见，大多时候使用DOM节点是很简单的。使用`query`可以快速、轻易地得到想要节点的集合。大多时候可以很方便地调整样式和改变类，这正是Dojo向页面添加交互的开始。我们已经展示了一个处理点击事件的简单例子，在下一个教程中，我们将深度解析Dojo事件处理。
