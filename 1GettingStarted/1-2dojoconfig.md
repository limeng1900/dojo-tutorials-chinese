#1.2 使用dojoConfig配置Dojo
[原文地址](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/index.html)：https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/index.html
dojoConfig 对象（原djConfig，1.6之前）可以用来设置工具包的选项和默认项。本篇关于如何使用dojoConfig 。

##简介
dojoConfig 对象是在web页面或应用中配置Dojo的主要机制。它和全局选项的Dojo组件一样被模块加载器引用。需要的话，可以进一步作为自定义应用的配置点。
旧对象名djConfig已弃用，不过2.0之前还在使用它的代码运行无碍。在写本篇的同时，大多数文档还在使用djConfig；这两个名称是等价的，不过现在起鼓励使用新的dojoConfig。

##准备开始
先通过几个简单的例子了解dojoConfig在实际工作中是怎么运行的。首先，我们直接看一个dojoConfig的实例：

```
<!-- set Dojo configuration, load Dojo -->
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
    // Explicitly parse the page
    parser.parse();
    // Find the dialog
    var dialog = registry.byId("dialog");
    // Set the content equal to what dojo.config is
    dialog.set("content", "<pre>" + JSON.stringify(config, null, "\t") + "```");
    // Show the dialog
    dialog.show();
});
</script>

<!-- and later in the page -->
<div id="dialog" data-dojo-type="dijit/Dialog" data-dojo-props="title: 'dojoConfig / dojo/_base/config'"></div>
```
请注意dojoConfig定义在脚本块里，这个脚本块要放在dojo.js加载之前。这非常重要，如果颠倒了的话，配置属性就会被忽略。
在该示例中，设置了三项：parseOnLoad: false, has (dojo-firebug sub-property)和 async: true。另外还设置了一个自定义属性foo: "bar"。这个案例里往页面加入了一个dijit/Dialog。代码运行时require回调将dojo.config的值转换成JSON再传入对话框显示出来。结果里可以看到parseOnLoad、has和foo。另外还有其它一些跨域、CDN dojo版本等信息。
提醒各位看官注意dojoConfig和dojo/_base/config有很大的区别。*dojoConfig纯粹以输入将配置参数传递给加载器和模块为目的。dojo/_base/config用于在引导过程中加入参数以便模块代码随后查找。*
下面是一个相同的dojo/_base/config例子：

```
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
        data-dojo-config="has:{'dojo-firebug': true}, parseOnLoad: false, foo: 'bar', async: 1">
</script>
```
该案例中，在Dojo的script标签内使用data-dojo-config属性，完全等同于之前的例子。两个例子中，配置选项最终都插入dojo/_base/config对象中，该对象在dojo.js加载后的引导过程中被迅速获取。
你可以在dojoConfig中设置新的值，然后在控制台查看dojo.config对象。dojoConfig是Dojo的通用配置属性包。下面来了解它都有哪些选项和如何使用。

###has()配置
Dojo 1.7后版本的一个重要特点是使用has()模式进行特性提取。我们通过指定hash对象作为has的属性在dojoConfig里配置特性（features）。这个特性集合用来决定是否启用Dojo的支持功能。例如，可以禁用amd factory scan：

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

