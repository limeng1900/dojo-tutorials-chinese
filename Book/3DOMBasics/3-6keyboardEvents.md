# 3.6键盘事件
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/key_events/index.html

----------

本教程中，我们将继续探索Dojo的标准事件和`dojo/keys`和Dojo如何处理键盘事件。

##入门
当键盘上的键被按下时出发键盘事件。它包括所有的键、字母、数字、符号、标点， 还有Escape、function、Enter、Tab和小键盘按键。所有的按键都触发一个可捕获和处理的事件。
浏览器对键盘事件的处理的支持和实现不尽相同。使用Dojo来处理键盘事件可以实现跨浏览器编程。

##键盘事件
实现用户界面时在浏览器中监听键盘事件不仅可以让你拥有原生app的感觉，还提供了针对UI更强大的控制。
**onkeypress**
> 按下任意键时出发，会一直重复直到放开按键。onkeypress 用在大部分的键盘事件处理中。

**onkeydown**
> 按下任意键时出发，会一直重复直到放开按键。大多数情况下onkeypress会在onkeydown之后触发。

**onkeyup**
> 按键释放事触发。
大部分按键三个事件都触发，但是浏览器之间可能有差异。接下来的例子将展示按键时触发的键盘事件。多花点时间试试不同的键和组合键。

Dojo将键盘事件标准化，你就可以使用`dojo/keys`来测试键盘上非输出的键。如果想要创建一个表格并用上下键或者enter键来遍历元素。可以从下面的例子开始：
```
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
                data-dojo-config="async: true"> </script>
<body>
    <h1>Press any key</h1>
    keyCode value: <input type="text" id="keyCode" size="2">
</body>
```

```
require(["dojo/on", "dojo/domReady!"], function(on) {
    on(document, "keyup", function(event) {
        document.getElementById("keyCode").value = event.keyCode;
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/key_events/demo/field_basic.html)

这个例子可见在表单元素中用Dojo标准化事件和`dojo/keys`捕捉键盘事件非常简单。用到了：

 - `dojo/on:` [api](https://dojotoolkit.org/api/?qs=1.10/dojo/on) [ref](https://dojotoolkit.org/reference-guide/1.10/dojo/on.html)
 - `dojo/keys:` [api](https://dojotoolkit.org/api/?qs=1.10/dojo/keys) [ref](https://dojotoolkit.org/reference-guide/1.10/dojo/keys.html)
你会注意到，例子并没有像预期的一样，还缺少一些功能，比如处理enter键。在下一个例子中将补充这些细节。
> 注意不同于`dojo/_base/connect`API，使用`dojo/on`模块时事件名前缀“on”必须省略。详见 [Events with Dojo](https://dojotoolkit.org/documentation/tutorials/1.10/events/)

##KeyboardEvent对象
下面可见，当键盘事件触发时，将一个`KeyboardEvent`传递给事件处理器。这个事件对象包含大量事件信息，其中最重要的是`keyCode`值。这是按键对应的Unicode值。
![](keyboardevent.png)
继续，我们可以使用Dojo的力量让这个简单的例子更加优雅和有效。

```
<body>
    <h1>Press Up or Down Arrow Keys</h1>
    <input type="text" id="input1" value="up">
    <input type="text" id="input2" value="down">
    <input type="submit" id="send" value="send">
</body>
```

```
require(["dojo/dom-construct", "dojo/on", "dojo/query", "dojo/keys", "dojo/domReady!"],
function(domConstruct, on, query, keys) {
    query("input[type='text']").on("keydown", function(event) {
        //query returns a nodelist, which has an on() function available that will listen
        //to all keydown events for each node in the list
        switch(event.keyCode) {
            case keys.UP_ARROW:
                event.preventDefault();
                //preventing the default behavior in case your browser
                // uses autosuggest when you hit the down or up arrow.
                log("up arrow has been pressed");
                break;
            case keys.DOWN_ARROW:
                event.preventDefault();
                //preventing the default behavior in case your browser
                // uses autosuggest when you hit the down or up arrow.
                log("down arrow has been pressed");
                break;
            case keys.ENTER:
                log("enter has been pressed");
                break;
            default:
                log("some other key: " + event.keyCode);
        }
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/key_events/demo/field_traverse.html)

只做了一些小的修改，我们已经消除了一些冗余代码，并让我们的脚本更加强大——可以在一个事件处理器中处理多种类型的按键，也可以处理多重元素。通过使用`dojo/query`，我们可以充分利`dojo/query`返回`NodeList`的`on`方法。除了`NodeList`里的节点都是定向的外，它和正常的`on()`函数是一样的。
前面的例子已经体现了Dojo的强大，不过为了全面完成我们使用键盘事件遍历表单的任务，还需要添加一些东西。来看一个全功能的新例子：

```
<body>
    <h1>Press Up/Down Arrow Or Enter Keys to traverse form.</h1>
    <h2>Home/End will go to the beginning or end.</h2>
    <form id="traverseForm">
        First Name: <input type="text" id="firstName">
        Last Name: <input type="text" id="lastName">
        Email Address: <input type="text" id="email">
        Phone Number: <input type="text" id="phone">
        <input type="submit" id="send" value="send">
    </form>
</body>
```

```
require(["dojo/dom", "dojo/dom-construct", "dojo/on", "dojo/query", "dojo/keys", "dojo/NodeList-traverse", "dojo/domReady!"],
function(dom, domConstruct, on, query, keys) {
    var inputs = query("input");

    on(dom.byId("traverseForm"), "keydown", function(event) {
        var node = query.NodeList([event.target]);
        var nextNode;

        //on listens for the keydown events inside of the div node, on all form elements
        switch(event.keyCode) {
            case keys.UP_ARROW:
                nextNode = node.prev("input");
                if(nextNode[0]){
                    //if not first element
                    nextNode[0].focus();
                    //moving the focus from the current element to the previous
                }
                break;
            case keys.DOWN_ARROW:
                nextNode = node.next("input");
                if(nextNode[0]){
                    //if not last element
                    nextNode[0].focus();
                    //moving the focus from the current element to the next
                }
                break;
            case keys.HOME:
                inputs[0].focus();
                break;
            case keys.END:
                inputs[inputs.length - 2].focus();
                break;
            case keys.ENTER:
                event.preventDefault();
                //prevent default keeps the form from submitting when the enter button is pressed
                //on the submit button
                if(event.target.type !== "submit"){
                    nextNode = node.next("input");
                    if(nextNode[0]){
                        //if not last element
                        nextNode[0].focus();
                        //moving the focus from the current element to the next
                    }
                }else {
                    // submit the form
                    log("form submitted!");
                }
                break;
            default:
                log("some other key: " + event.keyCode);
        }
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/key_events/demo/form_traverse.html)

最新的表单遍历使用`dojo/on`事件委托，我们在`form`是注册一个事件监听。它监听`form`的所有子元素的全部`keydown`事件。
事件委托的强大之处在于ROM我只需要做很小的修改就可以同时处理文本框和提交按钮。这个解决方案相当棒了，即使在往页面里添加更多的表单输入元素，它也100%有效，并且在`dojo/on`的标准化事件和`dojo/keys`的帮助下，它可以实现跨浏览器和跨平台。

##[dijit/_KeyNavMixin](https://dojotoolkit.org/reference-guide/1.10/dijit/_KeyNavMixin.html)的键盘导航
最后，如果你处理键盘事件的首要目标是方向键导航，你可以使用`dijit/_KeyNavMixin`。它需要在你的widget里定义的一些导航事件处理器，不过一旦你满足了这些需求，它就可以监听和回应有关方向导航的键盘事件。本教程中不会涉及更多细节，你可以在[reference guide](https://dojotoolkit.org/reference-guide/1.10/dijit/_KeyNavMixin.html)阅读相关内容，下面的例子是一个简单的实现。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/key_events/demo/keyNav.html)

##小结
Dojo的事件标准化使用`dojo/on`可以在多变和复杂的浏览器环境下更加简单的处理键盘事件。使用`dojo/on`和`dojo/keys`，我们创建一个扩充up/down箭头和enter键的默认行为的表单。想想如何利用这项新知识来使用Dojo的事件标准化和事件委托来提高你的web应用的可用性。

##资源
本教程中用到的`dojo/on`和其他工具的的更多细节：
 - [SitePen's Blog: dojo/on](http://www.sitepen.com/blog/2011/08/03/dojoon-new-event-handling-system-for-dojo/)
 - [Events with Dojo Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/events/)


