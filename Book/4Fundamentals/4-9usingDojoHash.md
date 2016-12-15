#4.9 使用dojo/hash和dojo/router

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/hash/index.html

----------

在JavaScript应用中，修改URL hash是提供书签和历史页面功能的好途径。用dojo/hash很容易添加这个功能。再搭配上dojo/router，dojo/hash将成为创建强健的交互应用的给力工具。

##入门
当编写基于JavaScript的web站点或应用时，重要的是要确保浏览器控件（如向前/向后按钮和书签）能正常工作，即使浏览器没有导航到一个全新的页面。使用`dojo/hash`模块可以提供跨浏览器的书签、历史页面功能。

##改变hash
我们的先创建一个简单页面，用Ajax向页面里加载新内容：

```
<!DOCTYPE html>
    <html>
        <head>
            <title>Welcome</title>
            <link rel="stylesheet" href="style.css">
        </head>
        <body>
            <ul id="menu">
                <li><a href="index.html">Home</a></li>
                <li><a href="about.html">About us</a></li>
            </ul>
            <div id="content">Welcome to the home page!</div>
            <!-- load dojo and provide config via data attribute -->
            <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js" data-dojo-config="isDebug: 1, async: 1, parseOnLoad: 1"></script>
            <script>
                require([
                    "dojo/dom",
                    "dojo/dom-attr",
                    "dojo/on",
                    "dojo/request",
                    "dojo/query"
                ], function(dom, domAttr, on, request){
                    var contentNode = dom.byId("content");

                    on(dom.byId("menu"), "a:click", function(event){
                        // prevent loading a new page - we're doing a single page app
                        event.preventDefault();
                        var page = domAttr.get(this, "href").replace(".html", "");
                        loadPage(page);
                    });

                    function loadPage(page){
                        // get the page content using an Ajax GET
                        request(page + ".json", {
                            handleAs: "json"
                        }).then(function (data) {
                            // update the page title and content
                            document.title = data.title;
                            contentNode.innerHTML = data.content;
                        });
                    }
                });
            </script>
        </body>
    </html>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hash/demos/demo1)

以上代码使用JavaScript更新页面内容和标题。事实上，它完全没有改变浏览器的历史记录或者URL。这就是说如果有人将这个页面做为书签或者想要浏览器的向前向后控件来导航，会发现都没有效果——没有页面可以返回，书签页也总是第一次加载的页面。通过两处小的改动，我们可以在请求新内容时得到一个更新后的URL：

```
 require([
        "dojo/dom",
        "dojo/dom-attr",
        "dojo/hash",
        "dojo/on",
        "dojo/request",
        "dojo/query" // for dojo/on event delegation
    ], function(dom, domAttr, hash, on, request){
        var contentNode = dom.byId("content");

        on(dom.byId("menu"), "a:click", function(event){
            // prevent loading a new page - we're doing a single page app
            event.preventDefault();
            var page = domAttr.get(this, "href").replace(".html", "");
            loadPage(page);
        });

        function loadPage(page){
            hash(page);

            // get the page content using an Ajax GET
            request(page + ".json", {
                handleAs: "json"
            }).then(function (data) {
                // update the page title and content
                document.title = data.title;
                contentNode.innerHTML = data.content;
            });
        }
    });
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hash/demos/demo2)

通过调用`dojo/hash`，页面的URL映射到新的状态，记录也加入到浏览器历史中。然而当用户想要使用back/forward按钮时，URL改变了但是没发生别的。为了实现它，我们添加代码来响应hash的变化。幸好，这很简单。

##响应hash变化：[dojo/topic](https://dojotoolkit.org/reference-guide/1.10/dojo/topic.html)
为了监听hash的变化，我们需要订阅`/dojo/hashchange`主题：

```
    topic.subscribe("/dojo/hashchange", function(hash){
        loadPage(hash);
    });
```
现在，无论何时页面的hash发生变化，页码都将收到一个包含新hash值的通知。这个信息只有在hash真正变化时才会发出，所以如果hash设定了一个相同的值，我们不用担心困在一个循环里或者做许多不必要的工作。（就是说，在我们的例子中，由于`/dojo/hashchange`监听器和`loadPage`函数之间的循环逻辑，我们将最后的页面请求存储起来，以避免在`loadPage`更新hash时向服务器发出多余的请求。）

我们最后要做的是，为了完成状态管理要确保关注页面加载时hash的初始值。有两种方式，最简单的是在第一加载时用页面的hash（如果存在）调用`dojo/hash`：

```
    hash(location.hash || lastPage, true);
```
有了这段代码，将在加载页面后立即获取给定hash的正确内容。注意我们将`true`作为第二个参数传递给`dojo/hash`；这意味着在浏览器历史里新页面将替换当前页面，而不是添加一个新页面。当响应一个动作的URL无效并需要从用户浏览器历史中移除时它也很有用（例如，在内容管理系统中，有人删除一条记录时）。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hash/demos/demo3)

##响应hash变化：[dojo/router](https://dojotoolkit.org/reference-guide/1.10/dojo/router.html)
如果你认为`/dojo/hashchange`主题处理器中的逻辑有点太多，你可能会想要使用`dojo/router`来代替。`dojo/router`将一些hash值的解析自动化，让你很容易将hash值映射到处理函数。它也提供模式匹配和hash的参数化，所以可以解析这部分hash字符串并作为变量用在处理函数里。

```
    router.register("/user/:id", function (event) {
        console.log("Hash change", event.params.id);
    });
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hash/demos/demo4)

如果你在使用`dojo/router`，你可能也会用到它的`go`方法，它是一个很方便改变hash的方法。

```
    router.go("hash");
```
下面是一个更加深入的例子，展示`dojo/router`模式匹配的实现：使用hash值模式`#/user/<id>`通过id加载users。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hash/demos/router)

##注意事项
将来，`dojo/hash`可能被支持新兴HTML特性（ [HTML5 session history and navigation](http://www.w3.org/html/wg/drafts/html/master/browsers.html#history)）的机制扩展或代替。然而，现在它完全运作在URL的hash部分，就是说需要注意一些事项（尽管它们相比完全没有状态管理没那么严重）。

第一个也是最重要的问题就是使用hash进行状态管理时创建的URL不会被搜索引擎索引。如果你想要内容被索引，你必须确保你的站点降低（像示例那样，连接指向有效的终点，我们ys AJAX就在数据代替通过URL变化加载），或者你可以按照Google在[Making AJAX Applications Crawlable](https://code.google.com/web/ajaxcrawling/docs/getting-started.html)发布的说明来做。

第二，使用hash不那么要紧的问题是，它原本用来基于一个元素的ID链接到一个页面的内容。如果在页面里有一个元素ID匹配你使用的hash，浏览器将滚动到它的位置。对这种情况，简单的解决是预先指定你的hash来确保它不会匹配任何元素ID。在上面的例子中可以这么改：

```
  require([
        "dojo/dom",
        "dojo/dom-attr",
        "dojo/hash",
        "dojo/on",
        "dojo/request",
        "dojo/topic",
        "dojo/query" // for dojo/on event delegation
    ], function(dom, domAttr, hash, on, request, topic){
        // find the content element and store a reference
        var contentNode = dom.byId("content"),
            prefix = "!",
            // store the last requested page so we do not make multiple requests for the same content
            lastPage = (/([^\/]+).html$/.exec(location.pathname) || [])[1] || "index";

        on(dom.byId("menu"), "a:click", function(event){
            // prevent loading a new page - we're doing a single page app
            event.preventDefault();
            var page = domAttr.get(this, "href").replace(".html", "");
            loadPage(page);
        });

        topic.subscribe("/dojo/hashchange", function(newHash){
            // parse the plain hash value, e.g. "index" from "!index"
            loadPage(newHash.substr(prefix.length));
        });

        hash(prefix + (location.hash || lastPage), true); // set the default page hash

        function loadPage(page){
            hash(prefix + page);

            // get the page content using an Ajax GET
            request(page + ".json", {
                handleAs: "json"
            }).then(function (data) {
                // update the page title and content
                document.title = data.title;
                contentNode.innerHTML = data.content;
            });
        }
    });
```
我们最终的示例包含以下重要功能，它们即用户友好又单页应用友好：

 - 当点击连接时，取消跟随它（`event.preventDefault`）并设置hash来替代
 - 监听hash变化并适当更新内容
 - 初始页面加载时设置hash和页面内容
 - 预先设置hash避免和页面中指向元素的链接冲突

这些将使应用可以书签，并使用浏览器自带的后退和前进空间，并使页面容易被搜索引擎索引。即便客户端禁用JavaScript也可以导航。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/hash/demo/index.html)

##小结
使用`dojo/hash`可以轻易地创建高度敏感同时又保持用户交互的所有正常浏览器功能的应用。只需要几行代码就可以给一个JavaScript增强Web站点或应用添加完整的状态管理。