# 2.1 CDN
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/cdn/index.html
本翻译GitBook地址：https://www.gitbook.com/book/limeng1900/dojo1-11-tutorials-translation-in-chinese/details

----------

有时候，从CDN加载Dojo模块会很有用，但是这么做的同时再使用本地模块几乎不可能。本教程将演示如何实现它。

## 简介
有些时候，从内容发布网络加载Dojo模块是很有用的，例如当需要在任意地方做一个简单的测试案例，或者用来提供易于分发和运行的示例代码。可惜，由于模块路径的组织方式导致从CDN使用Dojo及自定义模块并非一个直观的过程。为了在CDN下使用自定义和本地模块，需要进行一些额外的配置。
经研究CDN库表明，使用CDN通常比本地脚本性能上要差很多，尤其是本地脚本可以通过内置层来显著减少HTTP往返。如果你想用CDN库来提高应用性能，最好还是仔细想想。

## 加载自己的模块
开始先来一个简单页面，它包含了源于CDN的Dojo加载器：

```
<!DOCTYPE html>
<html>
    <head>
        <title>Demo</title>
    </head>
    <body>
        <script data-dojo-config="async: 1"
            src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>
    </body>
</html>
```
这段代码确保Dojo加载器开启了AMD功能，从而可以用require来加载其他模块。
Dojo1.7之前，跨域Dojo 加载器脚本是 `dojo.xd.js`。得益于AMD对跨域加载的原生支持，这个脚本就不再是必需的了。另外，注意在URL脚本里没有`http:`，这意味着对于当前页面从CDN加载脚本时也会使用相同的协议（例如：如果当前页面通过HTTPS加载也一样）。
接下来，我们需要通过设置`dojoBlankHtmlUrl` 属性使Dojo访问` dojo/resources/blank.html` 的本地副本文件，从而确保模块（如` dojo/hash`）开启跨域功能：

```
<script data-dojo-config="async: 1, dojoBlankHtmlUrl: '/path/to/blank.html'"
    src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>
```
最后，需要定义本地模块包的位置：

```
<script data-dojo-config="async: 1, dojoBlankHtmlUrl: '/blank.html',
        packages: [ {
            name: 'custom',
            location: location.pathname.replace(/\/[^/]+$/, '') + '/js/custom'
        } ]"
    src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>
```
注意本地包的位置是用从当前HTM文件获取的路径生成的绝对路径伪装的。Dojo1.10加载器要正确处理本地模块必须使用绝对路径。需要的话 `dojoBlankHtmlUrl` 关键字也可以使用同样的伪装。
现在完成了包含本地模块的包的定义，就可以像普通模块一样require了：

```
require([ 'custom/thinger' ], function(thinger){ … });
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/cdn/demo/index.html)

## 警告
不像旧的Dojo加载器，你在使用CDN创建模块时不用做别的事。但是，你使用源于CDN的Dojo加载器时可能会有一个问题：

 - 由于跨源的安全限制，使用`dojo/text ` 插件加载未创建或远程AMD模块会失败。（AMD的编译版本不受影响，因为编译系统已淘汰了对`dojo/text ` 调用。）

## 小结
基于CDN版本的Dojo在某些情况下是很有用的。感谢基于AMD的新系统，让我们在从CDN加载Dojo时使用自定义本地模块时，只需要做几个简单的配置变化。

## 链接

 - [Dojo configuration reference guide](http://dojotoolkit.org/reference-guide/1.10/dojo/_base/config.html)
 -  更多CDN库的信息： [Google CDN](http://code.google.com/apis/libraries/devguide.html) 和[Yandex CDN](http://api.yandex.ru/jslibs/)  


