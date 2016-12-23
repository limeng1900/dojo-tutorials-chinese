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

Dojo 1.7后版本的一个重要特点是使用has()模式进行特征检测。我们可以在`dojo.config`中为has()指定特征，只需要将特征的hash对象作为`has`的属性就可以。这个特性集合用来决定Dojo的某些支持能力。例如，我们可以禁用amd factory scan（扫描 CommonJS require（模块）语句的模块来作为deps进行加载）：

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

在Dojo1.7之前的版本或者其它教程里你可能已经熟悉`isDebug`配置项了，用它来开启调试信息。在1.7之后，它也在更高的粒度上用has()特征进行了指定。设置dojo-firebug 特征就可以在旧版本的IE浏览器里用Firebug Lite帮助调试（isDebug 依然可用，不过在异步模式下使用这个特征能够更早的加载）。如果你有Firebug或者其它控制台存在和开启，它就没什么用了。不过你要是没有控制台，它就会加载Dojo版本的 Firebug Lite，并在页面底部生成控制台界面。这在早期的IE和其他不带开发工具的浏览器里用起来很方便。

想要得到已弃用和实验性特征的调试信息，我们可以把 dojo-debug-messages设为true（默认`false`，除非设置了isDebug）。如果这个特征设置为false，相关的警告信息都不会显示。下面例子中，开启一个开发控制台（浏览器提供或者使用Firebug Lite）并且记录调试信息：

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
为了禁用一个有保证的控制台对象，我们可以dojo-guarantee-console特征设为false。这个特征默认为true，必要时会创建一个虚拟`console`对象，然后任何`console.*`记录语句都会安全而安静地执行，不会抛出异常。

以下附加项是用来进一步配置这个页内控制台的：

 - **debugContainerId：** 把控制台界面放在一个指定的元素里。
 - **popup：** 控制台放在新的弹出窗口，而不是当前页面里。

##Loader配置

1.7版本起Dojo针对toolkit的新AMD模块格式添加了一个新的加载器。新加载器加了几个新的配置选项，它们至关重要，用来定义packages、maps等。加载器的更多细节参考 [Advanced AMD Usage tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/)。比较重要的配置参数有：

 - **`baseUrl`：** 在将模块标识符转换成路径或URL时，模块标识符的基础URL预设：
 

```
 baseUrl: "/js"
```
 - **`packages`：** 提供package名称和位置的对象数组：
 

```
  packages: [{
        name: "myapp",
        location: "/js/myapp"
    }]
```
 - **`map`：** 可以把模块标识符映射到不同的路径：
 

```
map: {
        dijit16: {
            dojo: "dojo16"
        }
    }
```
 - **`paths`：** 模块id片段到文件路径的映射：
 

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

    // ...等同于:
var dojoConfig = {
    packages: [
        { name: "package1", location: "../lib/package1" },
        { name: "package2", location: "/js/package2" }
    ]
};
```
 - **`async`：** 定义Dojo core是否异步加载 ，它的值可以是 `true`, `false` 或 `legacyAsync`（将加载器永久设置为 旧跨域访问模式）：
 

```
   async: true
```
 - **`parseOnLoad`：** 如果为`true`，在DOM和所有初始依赖（包含在`dojoConfig.deps`数组里）加载完成后用`dojo/parser`解析页面：
 

```
    parseOnLoad: true
```
> 这里推荐`parseOnLoad`为`false`（默认,所以其实你可以忽略这个属性），开发人员可以require `dojo/parser`并调用`parser.parse()`。

 - **`deps`：** 一个资源路径的数组，这些资源要在Dojo加载后立即加载：
 

```
	deps: ["dojo/parser"]
```
 - **`callback`：** deps的回调：
 

```
  callback: function(parser) {
        // 使用这里提供的资源
    }
```
 - **`waitSeconds`：** 发出模块加载超时信号前的等待时间，默认为0（永远等待）：

```
    waitSeconds: 5
```
 - **`cacheBust`：** 如果为true，向每个模块URL附加时间作为查询字符串以避免模块缓存：
 

```
    cacheBust: true
```
让我们创建一个使用这些基本参数的简单演示。常用的一个方案是使用来自CDN的Dojo Toolkit和本地模块。下面使用Google CDN和来自`/documentation/tutorials/1.10/dojo_config/demo`的模块：

```
<!-- 首先配置 Dojo -->
<script>
    dojoConfig = {
        has: {
            "dojo-firebug": true,
            "dojo-debug-messages": true
        },
        // 不要试图为widget解析页面
        parseOnLoad: false,
        packages: [
            // 任何指向demo资源的引用都应该本地加载, 而*不是* 从 CDN
            {
                name: "demo",
                location: "/documentation/tutorials/1.10/dojo_config/demo"
            }
        ],
        // 10秒后超时
        waitSeconds: 10,
        map: {
            // 不用再输入 "dojo/domReady!", 我盟只需要用 "ready!" 代替
            "*": {
                ready: "dojo/domReady"
            }
        },
        // 获取 "新鲜" 资源
        cacheBust: true
    };
</script>

<!-- 从Google CDN 加载 Dojo, Dijit, and DojoX 资源 -->
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>

<!-- 加载一个 "demo" 模块  -->

<script>
    require(["demo/AuthoredDialog", "dojo/parser", "ready!"], function(AuthoredDialog, parser) {
        // 解析页面
        parser.parse();

        // 用 demo/AuthoredDialog 做些什么...
    });
</script>
```
我们使用`packages`配置让所有`demo/*`的引用指向`/documentation/tutorials/1.10/dojo_config/demo/`目录，对于`dojo`、`dijit`、`dojox`则从Google CND获得。如果`demo`包还没有定义，那么`demo/AuthoredDialog`的请求就会指向//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/demo/AuthoredDialog.js。我们还使用了别名，把`ready`关联到了`dojo/domReady`。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/demo/packages.html)

更多细节参考[关于新加载器的文档](https://dojotoolkit.org/reference-guide/1.10/loader/amd.html)。

> 新加载器也支持旧的`dojo.require()`资源加载和在Dojo 1.6教程里涉及的像`modolePaths`这样的配置属性，因此开发者可以轻易地安全升级已有的应用，不用太担心。

## 区域和国际化

Dojo的i18n系统是独立的，值得拥有它自己的教程，这里涉及它只是为了更好地展示`dojoConfig`。

你可以从`dojoConfig`使用Dojo的i18n基础设置来对任何widget或本地化内容进行区域配置。`locale`选项可以用来重写你的浏览器提供给Dojo的默认配置。一个简单的例子：

```
<script>
    var dojoConfig = {
        has: {
            "dojo-firebug": true,
            "dojo-debug-messages": true
        },
        parseOnLoad: true,
        //寻找一个 locale=xx 查询字符串参数，或者默认为 'en-us'
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
            // 为今天日期的对话框设置一个标题，使用本地化的日期格式
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

示例中，我们定义了`dojoConfig`对象的`locale`属性，从查询字符串里寻找`locale=xx`参数。这是一个手动演示，通常你可能会把区域硬编码。把`locale`设置放在任何模块加载之前，确保在需要的地方加载正确的本地化信息绑定依赖。案例中使用`dojo/date/locale`模块将日期对象格式化为本地化字符串，然后作为对话框的标题。

> 对于多语种页面，除了浏览器或`dojoConfig.locale`属性指定的那个，你还需要为其他区域加载bundles。在这个例子中，使用区域名字符串的数组来配置`extraLocale`属性。
> 使用`dojo/parser`时，`lang=`设置一个祖先DOMNode重写`dojoConfig.locale`的设置。这个行为将在Dojo2.0发生变化。你也可以为个别的widget指定`lang`，只为这个widget重写`dojoConfig.locale`设置。

##自定义属性

因为`dojo.config`总是存在，而且是提供页面范围配置的合理位置，Dojo的一些其它模块会使用它配置自己的特殊属性。我们会在Dijit看到这个，DojoX里更多，它们的模块标识和行为可以设置：

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

它对dojox模块有用，对你自己的应用和模块也一样。`dojoConfig`是一个提供行为、页面级或应用级属性配置的理想地方。看看下面的例子：

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

        // 从查询字符串推出配置信息并将它混入我们的app config
        var queryParams = ioQuery.queryToObject(location.search.substring(1));
        lang.mixin(config.app, queryParams);

        // 创建一个对话框
        var dialog = new Dialog({
            title: "Welcome back " + config.app.userName,
            content: "<pre>" + JSON.stringify(config, null, "\t") + "```"
        });

        // 利用 app config 来显示一个个性化信息
        dialog.show();

    });
</script>
```
>[View Application Config Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/demo/appConfig.html)

这个例子中，我们添加了一个`dojoConfig`属性`app`，随后通过`dojo.config`来弹出一个性化欢迎对话框。我们有很多方法来设置`dojoConfig.app`，可以使用默认预设或随后混入具体值。生产中，`dojoConfig`脚本块会写在服务器端。或者，你可以从cookie获取json格式的配置值，或像之前的例子那样，直接从查询字符串提取配置数据来设置它。在开发和测试模式下，你可以使用模板提供虚拟值或者加载一个脚本或模块来设置它。

##小结

本节教程中，我们提到了很多常见的`dojo.config`设置方式——通过`dojoConfig`或者`data-dojo-config`，还有它的值如何向Dojo模块提供属性及影响行为。

`dojo.config`在Dojo自展开阶段和全生命周期中具备良好的位置和角色，这也是说同样的思想也适用于Dojo模块甚至属于你自己的模块和应用。

##后记
- [dojoConfig (djConfig) 文档](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/config.html)
- [Dojo AMD 加载器配置参考](https://dojotoolkit.org/reference-guide/1.10/loader/amd.html#loader-amd-configuration)
- [i18n docs](https://dojotoolkit.org/reference-guide/1.10/dojo/i18n.html)

