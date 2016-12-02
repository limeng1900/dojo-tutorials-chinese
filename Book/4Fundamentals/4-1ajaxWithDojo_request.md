# 4.1 dojo/request和Ajax
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/ajax/index.html 

----------

[`dojo/request`](http://dojotoolkit.org/reference-guide/1.10/dojo/request.html)是一个新的从客户端向服务器发送请求的API（Dojo1.8 引入）。本教程主要介绍`dojo/request`API：你将学习如何从服务器请求一个text文件，如果发生错误如何处理，向服务器post信息，利用notify API和使用registry来用同样的代码从不同的位置请求数据。

##入门
`dojo/request`可以在不重载页面的前提下向服务器发送数据和接收服务器数据（通常称为AJAX）。新特性的引入让代码编写更加紧凑并更快。本教程会提到`dojo/promise`和`dojo/Deferred`，`dojo/request`用它们来进行异步编程。不过不可能一口吃成胖子，只要先记着promise和Deffreed可以让非阻塞的异步编程更加容易。在这个教程之后，你会想看看这些相关的教程。

##dojo/request介绍
先从简单的例子开始：
```
require(["dojo/request"], function(request){
    request("helloworld.txt").then(
        function(text){
            console.log("The file's content is: " + text);
        },
        function(error){
            console.log("An error occurred: " + error);
        }
    );
});
```
在浏览器中，上面的代码会使用一个`XMLHttpRequest`执行HTTP GET请求`helloworld.txt`并返回一个[`dojo/promise/Promise`](https://dojotoolkit.org/reference-guide/1.10/dojo/promise/Promise.html)，如果请求成功，传递给`then()`的第一个函数将文件的text作为唯一参数执行；如果请求失败，传递给`then()`的第二个函数会将error对象作为唯一参数执行。不过如果要向服务器发送表单数据呢？或者返回的是JSON或XML？没问题，`dojo/request`API允许自己定制。

##dojo/request API
每一个请求都需要一个终点。所以`dojo/request`的第一个参数是请求的URL。
为了适应应用需求和多重环境，Web开发者需要灵活的工具。`dojo/request`都考虑到了：首先，`dojo/request`必须的参数是请求的URL。第二个参数可以用一个`object`来自定义请求。一些最常用的选项是：

 - method —— 一个大写的字符串代表请求的HTTP方法。提供了一些帮助函数来指定这项（`request.get`,
`request.post`, `request.put`, `request.del`）。
 - sync —— 一个布尔值，如果为true，则让请求阻塞直到**服务器回应或者请求超时**。
 - query —— 一个字符串或者key-value对象包含了附加给**URL**的查询参数。
 - data —— 一个字符串、key-value对象或者`FormData`对象包含了要**传递给服务器**的数据。
 - timeout —— 请求失败和触发**错误处理器**前等待的毫秒数。
 - handleAs —— 一个字符串，代表着在将数据传递给成功处理器前，将应答数据转换成哪种载体方式。可用的格式有“text”（默认）、“json”、“javascript”和“xml”。
 - headers —— 一个key-value对象包含了请求发送的额外头部信息。

再来看看使用了这些选项的例子：

```
require(["dojo/request"], function(request){
    request.post("post-content.php", {
        data: {
            color: "blue",
            answer: 42
        },
        headers: {
            "X-Something": "A value"
        }
    }).then(function(text){
        console.log("The server returned: ", text);
    });
});
```
这个示例向`post-content.php`发送一个HTTP POST请求；请求POST的数据是一个简单对象（`data`），还有一个“X-Something”标头。当服务器回应时，按照约定的载体方式从`request.post`返回。

##示例：request.get and request.post
下面是`dojo/request`的一些常见用法。

### 在页面上显示一个文本文件的内容
这个例子使用`dojo/request.get`来请求一个文本文件。这个方式用在提供条款和协议的文本或者网站的隐私，因为文本文件只在明确被请求是菜发送给客户端，将文本保存在文件里比放在代码里要容易的多。

```
require(["dojo/dom", "dojo/on", "dojo/request", "dojo/domReady!"],
    function(dom, on, request){
        // Results will be displayed in resultDiv
        var resultDiv = dom.byId("resultDiv");

        // Attach the onclick event handler to the textButton
        on(dom.byId("textButton"), "click", function(evt){
            // Request the text file
            request.get("../resources/text/psalm_of_life.txt").then(
                function(response){
                    // Display the text file content
                    resultDiv.innerHTML = "<pre>"+response+"</pre>";
                },
                function(error){
                    // Display the error returned
                    resultDiv.innerHTML = "<div class=\"error\">"+error+"<div>";
                }
            );
        });
    }
);
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/demo/dojo-request-xhr.html)

### 登陆示例
下面的例子中，使用POST请求向服务器发送用户名和密码，并显示服务器返回的结果。

```
require(["dojo/dom", "dojo/on", "dojo/request", "dojo/dom-form"],
    function(dom, on, request, domForm){

        var form = dom.byId('formNode');

        // Attach the onsubmit event handler of the form
        on(form, "submit", function(evt){

            // prevent the page from navigating after submit
            evt.stopPropagation();
            evt.preventDefault();

            // Post the data to the server
            request.post("../resources/php/login-demo.php", {
                // Send the username and password
                data: domForm.toObject("formNode"),
                // Wait 2 seconds for a response
                timeout: 2000

            }).then(function(response){
                dom.byId('svrMessage').innerHTML = response;
            });
        });
    }
);

```

### 标头示例
在下面的例子中，用了和上面一样的POST请求，带有一个Auth-Token标头。
标头的访问使用了原始`Promise`的`promise.response.getHeader`方法（`XHR`返回的`Promise`**没有**这个属性）。另外，使用`promise.response.then`时，响应不是一个data，而是一个带有data属性的对象。

```
require(["dojo/dom", "dojo/on", "dojo/request", "dojo/dom-form"],
    function(dom, on, request, domForm){
        // Results will be displayed in resultDiv

        var form = dom.byId('formNode');

        // Attach the onsubmit event handler of the form
        on(form, "submit", function(evt){

            // prevent the page from navigating after submit
            evt.stopPropagation();
            evt.preventDefault();

            // Post the data to the server
            var promise = request.post("../resources/php/login-demo.php", {
                // Send the username and password
                data: domForm.toObject("formNode"),
                // Wait 2 seconds for a response
                timeout: 2000
            });

            // Use promise.response.then, NOT promise.then
            promise.response.then(function(response){

                // get the message from the data property
                var message = response.data;

                // Access the 'Auth-Token' header
                var token = response.getHeader('Auth-Token');

                dom.byId('svrMessage').innerHTML = message;
                dom.byId('svrToken').innerHTML = token;
            });
        });
    }
);

```

## JSON（Javascript Object Notation）
[JSON](http://json.org/)是AJAX请求的数据编码常用方式，因为它很易读、易用而且很紧凑。JSON可以用来对任意类型的数据进行编码：许多语言都支持JSON，包括： PHP、 Java、Perl、 Python、Ruby和ASP。

### JSON 编码对象

```
{
    "title":"JSON Sample Data",
    "items":[{
        "name":"text",
        "value":"text data"
    },{
        "name":"integer",
        "value":100
    },{
        "name":"float",
        "value":5.65
    },{
        "name":"boolean",
        "value":false
    }]
}
```
当`handleAs`设为`"json"`时，`dojo/request`以JSON数据作为响应载体，并且将它解析成一个JavaScript对象。

```
require(["dojo/dom", "dojo/request", "dojo/json",
        "dojo/_base/array", "dojo/domReady!"],
    function(dom, request, JSON, arrayUtil){
        // Results will be displayed in resultDiv
        var resultDiv = dom.byId("resultDiv");

        // Request the JSON data from the server
        request.get("../resources/data/sample.json.php", {
            // Parse data from JSON to a JavaScript object
            handleAs: "json"
        }).then(function(data){
            // Display the data sent from the server
            var html = "<h2>JSON Data</h2>" +
                "<p>JSON encoded data:</p>" +
                "<p><code>" + JSON.stringify(data) + "</code></p>"+
                "<h3>Accessing the JSON data</h3>" +
                "<p><strong>title</strong> " + data.title + "</p>" +
                "<p><strong>items</strong> An array of items." +
                "Each item has a name and a value.  The type of " +
                "the value is shown in parentheses.</p><dl>";

            arrayUtil.forEach(data.items, function(item,i){
                html += "<dt>" + item.name +
                    "</dt><dd>" + item.value +
                    " (" + (typeof item.value) + ")</dd>";
            });
            html += "</dl>";

            resultDiv.innerHTML = html;
        },
        function(error){
            // Display the error returned
            resultDiv.innerHTML = error;
        });
    }
);
```
除了在响应里将数据编码为JSON之外，还要将[Content-Type](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.17)设置为application/json，也可以使用服务器配置如 [Apache's AddType](http://httpd.apache.org/docs/2.0/mod/mod_mime.html#addtype)或者将它加在服务器端代码的标头。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/demo/dojo-request-json.html)

##JSONP(Javascript Object Notation with Padding)
AJAX请求限制在当前域里。如果你想要从一个不同的域请求数据，你可以使用[JSONP](http://json-p.org/)。当时用JSONP时，在当前页面插入一个脚本标记，请求src文件，服务器将data包装在回调函数里，当解析响应时，回调函数将data作为第一个参数调用。JSONP请求放在[dojo/request/script](https://dojotoolkit.org/reference-guide/1.10/dojo/request/script.html)。
让我们看几个例子：

### 使用JSONP从服务器请求数据并处理回应

```
require(["dojo/dom", "dojo/on", "dojo/request/script",
        "dojo/json", "dojo/domReady!"
], function(dom, on, script, JSON){
    // Results will be displayed in resultDiv
    var resultDiv = dom.byId("resultDiv");

    // Attach the onclick event handler to the makeRequest button
    on(dom.byId('makeRequest'),"click", function(evt){

        // When the makeRequest button is clicked, send the current
        // date and time to the server in a JSONP request
        var d = new Date(),
            dateNow = d.toString();
        script.get("../resources/php/jsonp-demo.php",{
            // Tell the server that the callback name to
            // use is in the "callback" query parameter
            jsonp: "callback",
            // Send the date and time
            query: {
                clienttime: dateNow
            }
        }).then(function(data){
            // Display the result
            resultDiv.innerHTML = JSON.stringify(data);
        });
    });
});
```
由于响应是JavaScript，不是JSON，响应的**Content-Type**标头应该是application/javascript。

### 使用JSONP从GitHub API请求Dojo pull requests

```
require(["dojo/dom", "dojo/on", "dojo/request/script",
        "dojo/dom-construct", "dojo/_base/array",
        "dojo/domReady!"
], function(dom, on, script, domConstruct, arrayUtil){
    var pullsNode = dom.byId("pullrequests");

    // Attach the onclick event handler to tweetButton
    on(dom.byId("pullrequestsButton"), "click", function(evt){
        // Request the open pull requests from Dojo's GitHub repo
        script.get("https://api.github.com/repos/dojo/dojo/pulls", {
            // Use the "callback" query parameter to tell
            // GitHub's services the name of the function
            // to wrap the data in
            jsonp: "callback"
        }).then(function(response){
            // Empty the tweets node
            domConstruct.empty(pullsNode);

            // Create a document fragment to keep from
            // doing live DOM manipulation
            var fragment = document.createDocumentFragment();

            // Loop through each pull request and create a list item
            // for it
            arrayUtil.forEach(response.data, function(pull){
                var li = domConstruct.create("li", {}, fragment);
                var link = domConstruct.create("a", {href: pull.url, innerHTML: pull.title}, li);
            });

            // Append the document fragment to the list
            domConstruct.place(fragment, pullsNode);
        });
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/demo/dojo-request-script-pulls.html)

##报告状态
[dojo/request/notify](https://dojotoolkit.org/reference-guide/1.10/dojo/request/notify.html)提供一个机制来报告`dojo/request`（或者任何`dojo/request`里提供的）请求的状态。require`dojo/request/notify`可以让提供者发出一个可以监听的事件来报告请求的状态。为了监听这个事件，调用`dojo/request/notify`的返回值，它有两个参数：一个事件名和一个监听函数。下面是`dojo/request`提供者发出的事件：

### 支持的dojo/request/notify 事件
 
 - start —— 第一个请求启动时发出
 - send —— 提供者发出请求前发出
 - load —— 提供者接收到一个成功响应时发出
 - error —— 提供者接收到一个error时发出
 - done —— 提供者完成一个请求时发出，不管成功还是失败
 - stop —— 当全部执行的请求完成时发出

`"start"` 和`"stop"`的监听器不接收任何参数。`"send"`的监听器接收两个参数：一个代表请求的对象和一个cancel 函数。调用cancel 函数将在请求开始之前取消它。`"load"`、`"error"`和 `"done"`的监听器接收一个参数：一个代表服务器响应的对象。来看一个示例：

### 使用dojo/request/notify来监听请求过程

```
require(["dojo/dom", "dojo/request", "dojo/request/notify",
        "dojo/on", "dojo/dom-construct", "dojo/query",
        "dojo/domReady!"],
    function(dom, request, notify, on, domConstruct){
        // Listen for events from request providers
        notify("start", function(){
            domConstruct.place("<p>Start</p>","divStatus");
        });
        notify("send", function(data, cancel){
            domConstruct.place("<p>Sent request</p>","divStatus");
        });
        notify("load", function(data){
            domConstruct.place("<p>Load (response received)</p>","divStatus");
        });
        notify("error", function(error){
            domConstruct.place("<p class=\"error\">Error</p>","divStatus");
        });
        notify("done", function(data){
            domConstruct.place("<p>Done (response processed)</p>","divStatus");
            if(data instanceof Error){
                domConstruct.place("<p class=\"error\">Error</p>","divStatus");
            }else{
                domConstruct.place("<p class=\"success\">Success</p>","divStatus");
            }
        });
        notify("stop", function(){
            domConstruct.place("<p>Stop</p>","divStatus");
            domConstruct.place("<p class=\"ready\">Ready</p>", "divStatus");
        });

        // Use event delegation to only listen for clicks that
        // come from nodes with a class of "action"
        on(dom.byId("buttonContainer"), ".action:click", function(evt){
            domConstruct.empty("divStatus");
            request.get("../resources/php/notify-demo.php", {
                query: {
                    success: this.id === "successBtn"
                },
                handleAs: "json"
            });
        });
    }
);

```

##dojo/request/registry
[dojo/request/registry](https://dojotoolkit.org/reference-guide/1.10/dojo/request/registry.html)提供了一个机制基于请求的URL来为请求提供路由。registry 常用于指定一个provider ，它基于这个请求是使用JSON在当前域还是使用JSONP在另一个域。如果URL将随着业务的进展变化，你也可以使用这个方法。

### dojo/request/registry 语法

```
request.register(url, provider, first);
```

### dojo/request/registry parameters

 - url —— url可以是一个字符串、正则表达式或者函数。
	- string —— 如果url是一个字符串，当url能够精准匹配时使用这个provider 。
	- regExp —— 如果url是一个正则表达式，当请求的URL匹配正则表达式时使用这个provider 。
	- function —— 如果url是一个function，request的URL和选项对象将传递给函数。如果函数返回一个真值，这个provider 将被用在请求上。 
 - provider —— 用来处理请求的provider。
 - first —— 一个可选的布尔值参数。如果为真， 在其它注册的provider之前注册这个providers。

让我们看最后一个例子：
### 使用dojo/request/registry基于request的URL指定provider
```
require(["dojo/request/registry", "dojo/request/script", "dojo/dom",
        "dojo/dom-construct", "dojo/on", "dojo/domReady!"],
    function(request, script, dom, domConstuct, on){
        // Registers anything that starts with "http://"
        // to be sent to the script provider,
        // requests for a local search will use xhr
        request.register(/^https?:\/\//i, script);

        // When the search button is clicked
        on(dom.byId("searchButton"), "click", function(){
            // First send a request to twitter for all tweets
            // tagged with the search string
            request("http://search.twitter.com/search.json", {
                query: {
                    q:"#" + dom.byId("searchText").value,
                    result_type:"mixed",
                    lang:"en"
                },
                jsonp: "callback"
            }).then(function(data){
                // If the tweets node exists, destroy it
                if (dom.byId("tweets")){
                    domConstuct.destroy("tweets");
                }
                // If at least one result was returned
                if (data.results.length > 0) {
                    // Create a new tweet list
                    domConstuct.create("ul", {id: "tweets"},"twitterDiv");
                    // Add each tweet as an li
                    while (data.results.length>0){
                        domConstuct.create("li", {innerHTML: data.results.shift().text},"tweets");
                    }
                }else{
                    // No results returned
                    domConstuct.create("p", {id:"tweets",innerHTML:"None"},"twitterDiv");
                }
            });
            // Next send a request to the local search
            request("../resources/php/search.php", {
                query: {
                    q: dom.byId("searchText").value
                },
                handleAs: "json"
            }).then(
                function(data){
                    dom.byId('localResourceDiv').innerHTML =
                        "<p><strong>" + data.name + "</strong><br />" +
                        "<a href=\"" + data.url + "\">" + data.url + "</a><br />";
                },
                function(error){
                    // If no results are found, the local search returns a 404
                    dom.byId('localResourceDiv').innerHTML = "<p>None</p>";
                }
            );
        });
    }
);

```

##最佳实践
`dojo/request`的最佳实践包括：

 - 小心选择request方法。通常，GET是用在不考虑安全的简单数据请求。GET通常比POST快。POST通常用来发送表单数据，数据不会在URL上传递。
 - 对于需要保护的数据在HTTPS页面上使用HTTPS。
 - 由于AJAX请求不刷新页面，大多数用户喜欢看到从*加载* 到*完成*的状态更新。
 - 在请求失败的检测和恢复使用error回调。
 - 使用开发工具来更快地解决问题。
 - 在尽可能多的浏览器里测试你的代码。

##小结
`dojo/request`为当前域和其他域的请求提供一个跨浏览器的AJAX接口，包括错误处理、通知支持和基于URL的请求路由。dojo/request返回的promise是一个[promise](https://dojotoolkit.org/reference-guide/1.10/dojo/promise/Promise.html)，允许发布一系列的请求和异步处理响应。页面可以包含多源的内容并使用能尽快使用请求得到的数据。用`dojo/request`加速你的页面！

##资源

 - [dojo/request 文档](https://dojotoolkit.org/reference-guide/1.10/dojo/request.html)
 - [JSONP 教程](https://dojotoolkit.org/documentation/tutorials/1.10/jsonp)
 - [Deferreds入门教程](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds)
 - [Dojo Deferreds 和Promises 教程](https://dojotoolkit.org/documentation/tutorials/1.10/promises)
 - [JSON](http://json.org/) 介绍JSON
 - [JSONP](http://json-p.org/) JSON-P文档
 - [比较GET 和 POST](http://www.diffen.com/difference/Get_vs_Post)
 - [Future and Promises](http://en.wikipedia.org/wiki/Futures_and_promises) 维基百科
