#1.2 使用dojoConfig配置Dojo

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/index.html

---

`dojoConfig` 对象（原`djConfig`）可以用来设置toolkit各方面的选项和默认行为。本篇教程我们将探讨有什么可能以及如何在你的代码中使用`dojoConfig`。

##简介

`dojoConfig`对象（Dojo 1.6之前是`djConfig`）是在web页面或应用中配置Dojo的主要机制。和带有全局选项的Dojo组件一样，模块加载器会引用它。需要的话，它可以进一步作为自定义应用的配置。

旧对象名`djConfig`已弃用，不过使用它的代码直到2.0之前都可以继续工作。在编写本篇的事件里，大多数文档还在使用`djConfig`；这两个名称是等价的，不过现在起我们会采用并且鼓励大家使用新的`dojoConfig`。

##入门

先通过几个简单的例子了解`dojoConfig`在实际工作中是怎么运行的。首先，我们直接看一个`dojoConfig`的实例：

```
<!-- 设定 Dojo 配置, 加载 Dojo -->
<script>
    dojoConfig= {
        has: {
            "dojo-firebug": true
        },
        parseOnLoad: false,
        foo: "bar",
        async: true
    };
</script>
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>

<script>
// Require the registry, parser, Dialog, and wait for domReady
require(["dijit/registry", "dojo/parser", "dojo/json", "dojo/_base/config", "dijit/Dialog", "dojo/domReady!"]
, function(registry, parser, JSON, config) {
    // 显式地解析页面
    parser.parse();
    // 找到 dialog
    var dialog = registry.byId("dialog");
    // 将内容设置为 dojo.config 的内容
    dialog.set("content", "<pre>" + JSON.stringify(config, null, "\t") + "```");
    // 显示 dialog
    dialog.show();
});
</script>

<!-- 随后的页面 -->
<div id="dialog" data-dojo-type="dijit/Dialog" data-dojo-props="title: 'dojoConfig / dojo/_base/config'"></div>
```
请注意`dojoConfig`定义在脚本块里，这个脚本块要放在dojo.js加载之前。这非常重要，如果颠倒了的话，配置属性就会被忽略。

在该例中，设置了三项内容：`parseOnLoad: false`, `has` (`dojo-firebug` 子属性)和 `async: true`。另外还设置了一个自定义属性`foo: "bar"`。这个示例里往页面添加了一个`dijit/Dialog`。代码运行时require回调将`dojo.config`的值转换成JSON再放在对话框显示出来。结果里可以看到`parseOnLoad`、`has`和`foo`。另外还有其它一些跨域、Google-CDN-hosted 版本的dojo 1.10等信息。

提醒大家注意`dojoConfig`和`dojo/_base/config`之间的区别。*`dojoConfig`纯粹是为了输入——这就是我们如何将配置参数传递给加载器和模块。在引导过程中，`dojo/_base/config`则由这些参数填充，以便模块代码随后查找。*

下面是一个相同的声明式编写的例子：

```
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
        data-dojo-config="has:{'dojo-firebug': true}, parseOnLoad: false, foo: 'bar', async: 1">
</script>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/demo/data-dojo-config.html)


该例中，我们在Dojo的`script`标签内使用同样的`data-dojo-config`属性。它完全等同于之前的例子。两个例子中，我们提供的配置选项最终都混入`dojo/_base/config`对象中，该对象在`dojo.js`加载时发生的自展处理之后可以马上可以获取到。

你可以在`dojoConfig`中设置一些新的值，然后在控制台查看`dojo.config`对象来确认这件事。所以说，`dojoConfig`是Dojo的通用配置属性包。下面来了解它都有哪些选项和如何使用。

##has()配置

Dojo 1.7后版本的一个重要特点是使用has()模式进行特征检测。我们通过指定hash对象作为`has`的属性在`dojoConfig`里配置特性（features）。这个特性集合用来决定是否启用Dojo的支持功能。例如，可以禁用amd factory scan：

```
<script>
    dojoConfig = {
        has: {
            "dojo-amd-factory-scan": false
        }
    };
</script>

```

##Debug/Firebug Lite配置

从Dojo1.7之前的版本或者其他教程里你可以已经熟悉isDebug配置项，就是用它开启调试信息。在1.7之后，它也放入了has()特性中。设置dojo-firebug 特性就可以在旧版本的IE浏览器里开启调试（isDebug 依然可用，不过在异步模式下这个特性能够更早的加载）。它对于Firebug或者其他支持console的浏览器没什么用。不过要是没有console，它就会加载Dojo版本的 Firebug Lite，并在页面底部生成console界面。这在早期的IE和其他不带开发工具的浏览器用起来的很方便。
想要得到已弃用和实验性特性的调试信息，要把 dojo-debug-messages设为true（默认false，除非设置了isDebug）。如果这个特性设置为false，相关的警告信息都不会显示。下面例子中，开启一个开发控制台（浏览器提供或者使用Firebug Lite）并且记录调试信息：

```
<script>
    dojoConfig = {
        has: {
            "dojo-firebug": true,
            "dojo-debug-messages": true
        }
    };
</script>
```
**dojo-guarantee-console**：默认为true，必要时会创建一个虚拟console对象，然后任何console.*语句都会静默执行，而不会抛出异常。
以下附加项是用来进一步配置页内控制台的：

 - **debugContainerId：** 把console界面放在一个指定的元素里。
 - **popup：** console放在新的弹出窗口，而不是当前页面里。

##Loader配置
1.7版本起Dojo针对toolkit的新AMD模块格式添加了一个新的加载器。新加载器加了几个新的配置选项，可以定义packages、maps等。比较重要的配置参数有：

 - **baseUrl：** 模块标识符的基础URL预设，用来转换成路径或URL：
 

```
 baseUrl: "/js"
```
 - **packages：** 提供package名称和位置的对象数组：
 

```
  packages: [{
        name: "myapp",
        location: "/js/myapp"
    }]
```
 - **map：** 可以把模块标识符映射到不同的路径：
 

```
map: {
        dijit16: {
            dojo: "dojo16"
        }
    }
```
 - **paths：** 模块id到文件路径的分段映射：
 

```
var dojoConfig = {
    packages: [
        "package1",
        "package2"
    ],
    paths: {
        package1: "../lib/package1",
        package2: "/js/package2"
    }
};

    // ...is equivalent to:
var dojoConfig = {
    packages: [
        { name: "package1", location: "../lib/package1" },
        { name: "package2", location: "/js/package2" }
    ]
};
```
 - **async：** 定义Dojo core是否异步加载 ，值可以是 true, false 或 legacyAsync（将加载器永久设置为 legacy cross-domain模式）：
 

```
   async: true
```
 - **parseOnLoad：** 如果为true，在DOM和所有初始依赖（包含在dojoConfig.deps数组里）加载完成后用dojo/parser解析页面：
 

```
    parseOnLoad: true
```
这里推荐parseOnLoad为false（默认），开发人员可以require dojo/parser并调用parser.parse()。
 - **deps：** 一个资源路径数组，这些资源要在Dojo加载后立即加载：
 

```
	deps: ["dojo/parser"]
```
 - **callback：** deps的回调：
 

```
  callback: function(parser) {
        // Use the resources provided here
    }
```
 - **waitSeconds：** 发出模块加载超时信号前的等待时间，默认为0（永远等待）：

```
    waitSeconds: 5
```
 - **cacheBust：** 如果为true，向每个模块URL附加时间字符串以避免模块缓存：
 

```
    cacheBust: true
```
下面是使用基本参数的简单例子。一个很常用的方案是使用来自CDN的Dojo Toolkit和本地模块。下面使用Google CDN和来自/documentation/tutorials/1.10/dojo_config/demo的模块：

```
<!-- Configure Dojo first -->
<script>
    dojoConfig = {
        has: {
            "dojo-firebug": true,
            "dojo-debug-messages": true
        },
        // Don't attempt to parse the page for widgets
        parseOnLoad: false,
        packages: [
            // Any references to a "demo" resource should load modules locally, *not* from CDN
            {
                name: "demo",
                location: "/documentation/tutorials/1.10/dojo_config/demo"
            }
        ],
        // Timeout after 10 seconds
        waitSeconds: 10,
        map: {
            // Instead of having to type "dojo/domReady!", we just want "ready!" instead
            "*": {
                ready: "dojo/domReady"
            }
        },
        // Get "fresh" resources
        cacheBust: true
    };
</script>

<!-- Load Dojo, Dijit, and DojoX resources from Google CDN -->
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>

<!-- Load a "demo" module -->

<script>
    require(["demo/AuthoredDialog", "dojo/parser", "ready!"], function(AuthoredDialog, parser) {
        // Parse the page
        parser.parse();

        // Do something with demo/AuthoredDialog...
    });
</script>
```
使用packages配置令所有demo/*的引用指向/documentation/tutorials/1.10/dojo_config/demo/目录，对于dojo、dijit、dojox则来自Google CND。如果demo package没有定义，那么demo/AuthoredDialog的请求就会指向//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/demo/AuthoredDialog.js。例中也用了别名，把dojo/domReady绑定到ready。

## 区域和国际化
Dojo的i18n系统是独立的，有自己的教程，这里在展示dojoConfig时会涉及到它。
你可以从dojoConfig使用Dojo的i18n基础设置来对任何widgets或局部目录进行区域配置。locale选项可以用来重写你的浏览器提供给Dojo的默认配置。示例：

```
<script>
    var dojoConfig = {
        has: {
            "dojo-firebug": true,
            "dojo-debug-messages": true
        },
        parseOnLoad: true,
        // look for a locale=xx query string param, else default to 'en-us'
        locale: location.search.match(/locale=([\w\-]+)/) ? RegExp.$1 : "en-us"
    };
</script>
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>
<script>
    require(["dojo/date/locale", "dijit/Dialog", "dojo/json", "dojo/_base/config",
    "dojo/_base/window", "dojo/i18n", "dojo/domReady!"]
    , function(locale, Dialog, JSON, config, win) {
        var now = new Date();
        var dialog = new Dialog({
            id: "dialog",
            // set a title on the dialog of today's date,
            // using a localized date format
            title: "Today: " + locale.format(now, {
                    formatLength:"full",
                    selector:"date"
            })
        }).placeAt(win.body());
        dialog.startup();

        dialog.set("content", "<pre>" + JSON.stringify(config, null, "\t") + "```");
        dialog.show();
    });
</script>
```
[Demo with dojo.config.locale ='zh' (Chinese)](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/demo/localeConfig.html?locale=zh)
示例中，定义了dojoConfig对象的`locale`属性，我们从查询字符串里寻找`locale=xx`参数。把locale设置放在任何模块加载之前，确保正确的区域信息绑定依赖在需要的时候已加载（Setting the locale ahead of any module loading ensures that the correct localized message bundle dependencies are loaded where necessary.）。案例中使用`dojo/date/locale`模块来格式化数据对象，然后传递给Dialog title的本地字符串（a localized string）。

##自定义属性
因为`dojo.config`总是存在而且是提供页面配置的合理位置，Dojo的一些其它模块会使用它配置自己的特殊属性。Dijit，尤其是DojoX都会这样：
**Dijit Editor**
	allowXdRichTextSave
**dojox GFX**
	dojoxGfxSvgProxyFrameUrl, forceGfxRenderer, gfxRenderer
**dojox.html metrics**
	fontSizeWatch
**dojox.io transports and plugins**
	xipClientUrl, dojoCallbackUrl
**dojox.image**
	preloadImages
**dojox.analytics plugins**
	sendInterval, inTransitRetry, analyticsUrl, sendMethod, maxRequestSize, idleTime, watchMouseOver, sampleDelay, targetProps, windowConnects, urchin
**dojox.cometd**
	cometdRoot
**dojox.form.FileUploader**
	uploaderPath
**dojox.mobile**
	mblApplyPageStyles, mblHideAddressBar, mblAlwaysHideAddressBar, mobileAnim, mblLoadCompatCssFiles

对dojox模块起作用，对你自己的应用和模块也一样。`dojoConfig`是一个提供行为、页面、应用或广泛性能配置的理想位置。看看下面的例子：

```
<script>
    dojoConfig = {
        has: {
            "dojo-firebug": true
        },
        app: {
            userName: "Anonymous"
        }
    };
</script>
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>
<script>
    require(["dijit/Dialog", "dijit/registry", "dojo/parser", "dojo/_base/lang",
    "dojo/json", "dojo/_base/config", "dojo/io-query", "dojo/domReady!"]
    , function(Dialog, registry, parser, lang, JSON, config, ioQuery) {

        // pull configuration from the query string
        // and mix it into our app config
        var queryParams = ioQuery.queryToObject(location.search.substring(1));
        lang.mixin(config.app, queryParams);

        // Create a dialog
        var dialog = new Dialog({
            title: "Welcome back " + config.app.userName,
            content: "<pre>" + JSON.stringify(config, null, "\t") + "```"
        });

        // Draw on the app config to put up a personalized message
        dialog.show();

    });
</script>
```
示例中，添加了一个dojoConfig属性`app`，通过dojo.config来弹出一个性化欢迎的Dialog。有很多方法来填充dojoConfig.app，可以使用默认预设和插入特性值。生产中，dojoConfig会在服务器端输出。或者可以从cookie获取json，或者像之前的例子直接中query string提取配置数据。在开发和测试模式下，你可以使用模板提供虚拟值或者加载脚本/模块来填充它。

##小结
本节教程中包含了多种dojo.config使用方式——通过dojoConfig或者data-dojo-config。还有dojo.config的值如何向Dojo模块提供属性和影响其行为。
dojo.config在 Dojo bootstrap阶段和全生命周期良好的位置和角色意味着它可以巧妙的适用于Dojo模块甚至属于你自己的模块和应用。

