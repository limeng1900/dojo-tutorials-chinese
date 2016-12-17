# 4.11 JSONP和dojo/request

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/jsonp/index.html
**本翻译项目放在GitBook上，欢迎参与。** 
GitBook地址：https://www.gitbook.com/book/limeng1900/dojo1-11-tutorials-translation-in-chinese/details 
转载请注明出处：http://blog.csdn.net/taijiedi13/ – 碎梦道

----------

JSON with Padding （JSONP）已经成为浏览器跨域访问的常用技术。这篇教程里，你将了解什么是JSONP以及如何用它从跨域资源获取数据。

##入门
Dojo的Ajax功能提供一个简单但是强大的接口来获取动态资源。然而，浏览器跨域安全性的限制妨碍了你向另一个域的XHR请求。怎么办？更多新一代浏览器按照W3C [Cross-Origin Resource Sharing](http://www.w3.org/TR/cors)规范提供了跨域请求的能力。但是并不是所有的浏览器都可用，有大量已有的服务还没有利用这个规范。

跨域交流的解决方式是JSON with Padding，或者叫JSONP。Bob Ippolito 在2005年[最先介绍了JSONP技术](http://bob.pythonmac.org/archives/2005/12/05/remote-json-jsonp/)，今天许多来自Google、GitHub、Facebook的服务都提供获得API。Dojo的`dojo/request/script`模块（1.8引入，代替`dojo/io/script`）提供JSONP资源的无缝获取，而不需要乱七八糟的设置细节。

JSONP技术到底是什么？不像XHR，浏览器不会阻止脚本跨域加载。JSONP通过动态地在页面插入一个`<script>`元素，并将它的资源设为我们将要查询的跨域URL。通过传递URL上的查询字符串上的相关参数，它可以返回一个动态响应，该响应以JavaScript格式表示我们请求的数据。例如，一个请求`endpoint?q=dojo&callback=callme`，它的响应如下：

```
callme({id: "dojo", some: "parameter"})
```
随后浏览器解析脚本里的代码时，它将调用`callme()`方法——传入它的数据。已经定义了`callme`方法的本地应用随后将接收到。注意这本质上是从第三方执行脚本；由于你从第三方服务执行脚本，这表示你的应用程序信任该第三方服务。这不是说JSONP不好或者应该避免使用，只是它应该限制在与信任对象的交流上使用。

>用JSONP使用跨域资源也会减少你应用的web服务的连接数。浏览器限制同一时间发送给服务器的请求数。最糟糕的时候，IE6只能并行两个连接。新的浏览器默认为6到8个连接。当获取跨域资源时，它不计入你服务的连接总数里。

`dojo/request/script`将脚本元素的创建和回调方法自动化，并且提供你熟悉的[`Deferred`](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/)接口。

```
//include the script modules
require(["dojo/request/script"], "dojo/dom-construct", "dojo/dom", "dojo/_base/array", "dojo/domReady!"
], function(script, domConstruct, dom, arrayUtil){
        // wait for the page and modules to be ready...
        // Make a request to GitHub API for dojo pull requests
            script.get("https://api.github.com/repos/dojo/dojo/pulls", {
                jsonp: "callback"
        });
});
```
这个代码和你在`dojo/request/xhr`里看到的基础模式是一样的。你会注意到唯一增加的是`jsonp`；这个属性告诉Dojo终点希望你指定的回调函数的名字放在哪个参数上（不是回调函数名称本身）。这往往在服务之间各不相同。这个代码获取Dojo GitHub rep上最近的合并请求。来个充实点的例子：

```
//first do the io script request
script.get("https://api.github.com/repos/dojo/dojo/pulls", {
    jsonp: "callback"
}).then(function(response){
    return response.data;
}).then(function(results){
    // Iterate through the response results and add them to the DOM
    var fragment = document.createDocumentFragment();
    arrayUtil.forEach(results, function(pull){
        var li = domConstruct.create("li", {}, fragment);
        var link = domConstruct.create("a", {href: pull.url, innerHTML: pull.title}, li);
    });
    domConstruct.place(fragment, dom.byId("pullrequests"));
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/jsonp/demo/demo.html)

>驱动JSONP的机制（动态插入一个`<script>`标签）无法以标准Ajax请求一样的方式处理error。当脚本由于HTTP error（404、500等）加载失败时，浏览器不会向应用发出信号，所以`dojo/request/script`回调也不会收到任何信号。为了不让你的应用一直等待脚本的返回，你可以为`dojo/request/script`请求设置一个`timeout`属性。如果在超时之前没有完成回调，`Deferred`将被拒绝，这样你的应用就可以做出正确的行动。

##小结
JSONP可以让你获取丰富的资源，你就可以混搭自己的应用来轻松创建高效有趣的接口。大多数主流web服务供应商都提供一些可以用JSONP访问的服务。即使在同一个组织里，通过JSONP获取不同子域的服务可以减少连接数，以适应某些限制并发连接数的浏览器。沿着和你用过的标准`dojo/request`相同的模式，现在你应该能够获取跨域资源了。

如果你想做一些实践，你可以尝试访问 Flickr JSON API并显示结果图像。为了帮助你入门，这里有一个返回 Dojo Toolkit-tagged图像的Flickr URL：http://api.flickr.com/services/feeds/photos_public.gne?tags=dojotoolkit&lang=en-us&format=json

##进一步参考

 - [教程: Ajax with Dojo](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/)
 - [Dojo Toolkit 参考指南: dojo/request/script](https://dojotoolkit.org/reference-guide/1.10/dojo/request/script.html)
 - [Dojo Toolkit API 文档: dojo/request/script](https://dojotoolkit.org/api/?qs=1.10/dojo/request/script)
 - [克服Ajax 应用安全威胁](http://www.ibm.com/developerworks/xml/library/x-ajaxsecurity.html)