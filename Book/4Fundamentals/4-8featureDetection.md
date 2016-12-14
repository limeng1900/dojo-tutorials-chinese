# 4.8 特征检测和设备优化的Build

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/device_optimized_builds/index.html 

----------

Dojo现在使用流行的`has()`模式进行特征检测，并结合到`has()`-aware（“有意识”）的build系统。使用专门JavaScript表达式很容易编写特征监测测试，`has()`模式定义了一个特定语法，这样build系统就可以监测到这些基于特征的分支，并且随着已知功能垫片的剔除，你可以为特定设备创建高度优化的应用build。

##入门
>确保你了解[Creating Builds tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/build)给出的概念。

移动设备的革命为web应用提出了新的需求。移动设备普遍具有更低的带宽和更低的CPU运算能力，迫使我们避免大型复杂代码。幸运的是相比桌面端，移动设备上有非常高比例的用户使用新一代浏览器，所以针对移动浏览器用更少的代码编写类似应用是可行的。然而，处理多平台也不容易，为移动设备创建合适的小代码包，同时又为旧桌面浏览器提供足够的能力是很有挑战性的。处理平台差异有几种不同的方式，过去十年的教训表明特征检测是分支的重要机制。

幸好，Dojo1.7+已经进化出了新的强大的特征检测基础。现在Dojo使用流行的[`has()`](https://github.com/phiggins42/has.js) 模式进行特征监测，并结合到`has()`-aware（“有意识”）的build系统。使用专门JavaScript表达式很容易编写特征监测测试，`has()`模式定义了一个特定语法，这样build系统就可以监测到这些基于特征的分支，并且随着已知功能垫片的剔除，你可以为特定设备创建高度优化的应用build。

为了使用`has()`模式，Dojo在1.8的基本代码已经进行了代码重构，我们不需要在代码中使用`has()`就可以快速开始构建平台优化的build。当然最常见的优化build是在主流移动设备使用的新一代WebKit平台。目前在移动世界里使用的WebKit版本之间有几个小的变更，但是我们可以依赖大量重要的已知特征来为WebKit浏览器和移动设备创建build。为了指定已知特征，我们在build配置文件的`staticHasFeatures`属性引入特征的对象。这里有一个build配置文件的简单入门，它涉及Dojo用到的主要特征：

```
var profile = {
    // ...
    action: "release",
    layerOptimize: "closure",

    staticHasFeatures: {
        "dom-addeventlistener": true,
        "dom-qsa": true,
        "json-stringify": true,
        "json-parse": true,
        "bug-for-in-skips-shadowed": false,
        "dom-matches-selector": true,
        "native-xhr": true,
        "array-extensible": true,
        "quirks": false,
        "dom-quirks": false
    },
    // ...
```
使用这个配置文件，build系统将找到代码中的特征分支，替换提供的已知特征（或bugs）。

> 注意上面配置文件的`layerOptimize: "closure"`。使用闭包编译器是包含`staticHasFeatures`的build配置文件的关键，它能够执行废代码移除——移除那些已知条件分支不用的代码块。

运行build之后，我们现在有一个Dojo（或者我们应用）的build版本，它不包含任何W3C缺少的`addEventListener()`、`querySelectorAll()`的补充代码和其他早期版本IE中缺失的标准特征的额外代码。当这个优化build运行在基础dojo.js时，相比全浏览器支持的Dojo版本它将为我们节省9KB空间。对于大小敏感的应用，9KB是很大的节约。我们可以为应用的移动版本使用这个build，或者在检测到WebKit浏览器时使用它。

当使用设备指定build时，我们通常需要为每种特性设置运行单独的build（build系统不支持一个build运行不同的静态特征）。例如，我们可以创建一个脚本：

```
# run build with webkit static features
./build.sh --profile /path/to/webkit-profile.js --releaseDir /target/dojo-webkit
# run build without any features, to work on any other browser
./build.sh --profile /path/to/standard-profile.js --releaseDir /target/dojo-standard
```
如果我们想要创建一个在运行时基于主机浏览器页面选择适当的build的页面，我们可以使用简单的浏览器检测。有几种不同的方式来实现，下面可能是最简单的：

```
<script>
    // choose the appropriate dojo script based on the user agent;
    // will match FF, Safari, Chrome, mobile browsers, not IE
    var dojoScript = /Gecko/.test(navigator.userAgent) ?
        "dojo-webkit/dojo/dojo.js" : "dojo-standard/dojo/dojo.js";
    // now create and append a script element to load it:
    var head = document.getElementsByTagName("head")[0],
        element = document.createElement("script");

    element.async=true;

    // configure Dojo for async mode
    var dojoConfig = {
        async: true
    };
    element.src = "path/to/dojo/" + dojoScript;
    // insert the script so it will load
    head.insertBefore(element, head.firstChild);
</script>
```
上面的脚本将异步加载Dojo，可以让你的页面加载地更快。然而，如果你想要同步地加载Dojo，你可以使用document.write来代替：

```
<script>
    // choose the appropriate dojo script based on the user agent
    // will match FF, Safari, Chrome, mobile browsers, not IE
    var dojoScript = /Gecko/.test(navigator.userAgent) ?
        "dojo-webkit/dojo/dojo.js" : "dojo-standard/dojo/dojo.js";
    document.write('<script src="path/to/dojo/' + dojoScript + '"></s' + 'cript>');
</script>
```

> 你可能注意到尽管我们提倡特征检测，但在这个例子中我们却用了浏览器嗅探。一般来说，在你的源代码里绝对优先使用特征检测，因为它让你的代码对于浏览器来说更加健壮。然而，像上面的例子中使用基于用户代理的代码，它避免了运行多重特征检测的高额开销（它们会消耗大量的时间和空间），进行了有价值的优化。当你这么做时，确保优化不同于使用特征检测的代码，便于目标分离。将它从模块中分离，放在HTML里，是实现这种组织方式的好办法。

因为build系统是基于特征设置的，我们可以进一步创建更多平台特定的build。我们可以定义额外的特征，为不同版本的IE创建特定build（新版IE包含更多项特征），或从WebKit中分理处Firefox和Opera。基于build的特征设置允许设备特定优化的无限排列。

另一个build设置是选择器引擎，我们也可以用它创建轻量化build。默认情况，Dojo使用“acme”引擎build，它很早就成为了Dojo的一部分。然而，1.7引入了一个“lite”选择器引擎来替代。lite”选择器引擎侧重于新一代浏览器原生的`querySelectorAll`功能，它对于旧浏览器没有完全的CSS3支持。然而，它支持大多数应用主要使用的核心CSS2特性（关于lite引擎功能更多细节请看[dojo/query documentation](https://dojotoolkit.org/reference-guide/1.10/dojo/query.html)）。你可以使用lite引擎如果你的目标是新一代浏览器或者你的应用不需要使用任何花哨的CSS3查询。如下在你的build配置文件里选择lite引擎：

```
var profile = {
    selectorEngine:"lite",
    ...
};
```
相比另一个，lite引擎将为dojo.js节省6KB空间。

>注意在运行时（build之前），lite引擎默认为异步模式，acme引擎默认为同步模式。如果你在build里指定一个选择器引擎，它将在build应用时使用。如果你想要确认在开发时和build应用中使用相同的浏览器引擎，你可以在页面的dojo配置中明确地选择lite引擎。

```
var dojoConfig = {
    async: true,
    selectorEngine:"lite"
}

```

##使用has()
在用已知特征运行build方面，目前我们已经利用了Dojo基础代码中存在的特征检测分支。然而，我们可能想要在自己的应用中使用`has()`。Dojo规范了浏览器之间的大部分重要差异，但仍然有这样的情况，你的应用需要检测一个浏览器特征或bug并对应作出响应。我们可以使用`dojo/has`模块来使用`has()`功能。如果我们使用一个存在的Dojo检测的特征，会很简单：

```
require(["dojo/has"], function(has){
    if(has("touch")){
        // show our touch interface
    }else{
        // show our mouse-driven interface
    }
});
```
[dojo/has reference page](https://dojotoolkit.org/reference-guide/1.10/dojo/has.html)提供一个Dojo检测特征的列表。如果Dojo测试特征不够用，你也可以通过调用`has.ass()`很容易地创建你自己的特征检测测试：

```
require(["dojo/has"], function(has){
    // test if we have video
    has.add('html5-video', !!document.createElement('video').canPlayType);
    if(has('html5-video')){
        // show our video with a &lt;video&gt; element
    }else{
        // use flash or something
    }
});
```
这些例子都使用`has()`模式，这样build系统就可以正确地识别它们的特征分支，你在创建build时就可以用已知特征来消除指定浏览器不用的分支。

##小结
为了跨浏览器web应用开发和移动web应用的高度优化，使用最新和更加高级的技术——新的特征监测基础构造和build系统的集成，它也帮助Dojo更加先进。

##Dojo Build 资源
build的更多细节请查看：

 - [Creating Builds tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/build)
 - [The Dojo Build System](https://dojotoolkit.org/reference-guide/1.10/build/index.html)