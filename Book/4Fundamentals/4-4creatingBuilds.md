# 4.4创建Build
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/build/index.html

----------

Dojo的build系统提供了build Dojo和其它的JavaScript资源以及CSS文件的方法，从而让你的应用在生产环境下更加高效的使用它们。

##"Building" Dojo 还是 JavaScript？
如果你用过其它编程语言，你可能纳闷为什么我们要讨论"Building" Dojo 还是 JavaScript，因为build通常意味将代码编译成机器字节码。但是当我们讨论生成Dojo时，通常是关于简化、优化、串联和废代码删除这些概念。
当你从服务器端将代码发送给客户端解析时，比如JavaScript、HTML和CSS，它们将占用带宽和时间。你有多少要发送，就对应在用户知觉的延迟上加上多少对服务器的请求。不管你的代码执行的有多快，在它抵达客户端之前你都无法开始执行。
Dojo Builder是一个用来将代码尽可能最有效地配送到客户端，它包含处理自定义代码、模块和CSS的能力。如果做的好，你可以为应用创建一个易于管理和维护的build系统。

##基础
Dojo的build系统是综合和可高度定制化的。它还可以进行扩展，不过本教程不会涉及。另外，虽然在Dojo1.7引入的新bulid系统大体是向后兼容的，但它完全进行了重写，多数情况下你最好重新看待你的build，尤其是把应用转向AMD的情况。
在开始之前，你需要先理解教程中用到的几个核心概念。

### Modules and Packages
希望你在尝试本教程之前已经能理解模块。如果没有请先复习 [Defining Modules](https://dojotoolkit.org/documentation/tutorials/1.10/modules/) 教程，它们是Dojo 1.10的基础。这些模块组织成包从而构成模块的逻辑分组。在Dojo1.10中，每个包应该有一个描述包的`package.json`。许多包也包含一个`package.js`文件来提供Dojo特定的build信息。

### Dojo 配置
 Dojo内部有许多可以在应用中利用的配置选项。这些配置信息不仅对于你应用的正确运行很重要，在build应用时也可能需要提供这些配置信息作为build过程的一部分。如果你不熟悉如何配置Dojo，你需要复习[Configuring Dojo with dojoConfig](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config/) 教程。

### Layers
layer本质上是一个单独JavaScript文件，它一般会包含几个模块，有时还有其它资源。一个layer通常build的主要输出，创建这个文件然后成为你应用的可分配之处。一个layer可以是一个引导layer，它包含Dojo的引导代码可以让Dojo加载其它模块。你需要什么layer以及它们的内容都由你的应用和设计来决定。不一定有一个“正确”的方式。

### build配置文件
build配置文件是一个给builder提供代码处理相关信息的很小的JavaScript文件。在旧的build系统中，你可能只有一个配置文件，它包含了所有你需要的东西，放在`util/buildscripts/profiles`目录。Dojo1.7以后，它变得高度分散，每个包都有它自己的build配置文件，然后你还有一个主build配置文件，它指导package和layer的builder，也提供用来优化代码的配置选项。

### 压缩
这个概念是关于在不改变代码功能的前提下获取和压缩JavaScript代码。从效率的角度看它很强大并且一些开发者喜欢它让代码变的模糊，但是它让调试变得困难。这将是你不愿对“build”过的代码做开发的原因之一。Dojo Builder利用两个工具集进行压缩。第一个是ShrinkSafe，只有它能够用在1.7之前、1.7及之后的builder里。builder还可以利用Google的 [Closure Compiler](https://developers.google.com/closure/compiler/)。

### 废代码路径的移除
Google Closure Compiler的优点在于它能检测到不可访问的代码，并在文件的压缩版里将它移除。几年前，我们启动了一个名叫Dojo Linker的项目来解决相似的问题，但一直没有时间完成它，所以很高兴有一个替代方案。Dojo Builder设计的时候考虑到了这个特性。在build时可以设置几个“旋钮”来让builder输出代码的路径上“硬编码”，随后在优化时，Closure Compiler检测到某些代码一直访问不到，就会在压缩文件里将它们移除。

### Build Controls, Transforms and Resolvers
这些是build系统里的基础build模块。通常你不需要用到它们，不过如果你对高级build感兴趣，可以通过修改它们来做一些有趣的事。build control是build“指令”的设置，它读取配置文件并确定使用哪种 transform 和 resolver。transform 如字面意思一样，用于对某些事物进行转换，resolver是用来在build 时对AMD插件模块进行解析。例如，当你在用`dojo/text`了加载一个widget模板时，build时插件解析会读取这个文本并将它内联到压缩文件里。

##你需要的
使用Dojo build系统，你必须有一个[full Dojo SDK](https://dojotoolkit.org/download/#sdk)副本。Dojo的“标准发行版”已经用build系统build过了。build工具本身依赖于Java（可选，Node.js用在更快的build），所以确认下你已经安装了它。

##布置你的应用
这是最大的挑战之一，尤其是当你已经超越了非常基础的应用。我们已经注意到许多从Dojo1.6和更早版本迁移过来的人，他们设计了我们预期之外非常奇怪的应用结构。这就是说许多人在将他们的build迁移到Dojo 1.7及之后版本时遇到了挑战。所以在你的应用偏离之前，你需要考虑如何布置它们。
形式上需要一个`src`目录，所有你的包都要放在这个根目录下。通常你需要像这样：
![](4.4file_list.png)
Dojo的主体包（`dojo`、`dijit`和`dojox`）和其它包并列，包括你的自定义包，在加上Dojo实用工具（`util`）。

如果你不想从头开始， [Dojo Boilerplate](https://github.com/csnover/dojo-boilerplate)包含了你起步所需的一切，不仅有基本应用程序的布局，而且已经为build进行了配置。

##包
为了用build系统来创建一个build，你的主应用包目录里必须包含两个文件。第一个是 [CommonJS Packages/1.0](http://wiki.commonjs.org/wiki/Packages/1.0)  包描述，它通常命名为`package.json`，放在包根目录下。第二个文件是一个build配置文件，它包含了如何用build工具处理包目录的相关信息。这个文件的命名有两个主要约定。在Dojo Toolkit里它叫做`<package name>.profile.js`，放在包的根目录下。另一个约定是将配置文件命名为`package.js`放在包的根目录下。你会在 [Dojo Foundation Packages](http://packages.dojofoundation.org/)看到这种约定。

通常，你会只有一个包来内含你全部的应用，但是如果你已经把你的代码分解到多个包里（例如，在不同的包里使用shared/common模块），你需要对每个包都创建这些文件。


##包描述 
包描述文件(package.json)提供当前包的信息，例如包的名字、它依赖包的信息、许可的链接和bug追踪信息等。鉴于Dojo build系统的目的，真正重要的是一个特殊键`dojoBuild`，尽管它通常只提供一个名称、版本和描述。`dojoBuild`键用来指向包的build配置文件。给出一个名为`app`的包作为例子，一个基础`package.json`像下面这样：
```
{
    "name": "app",
    "description": "My Application.",
    "version": "1.0",
    "keywords": ["JavaScript", "Dojo", "Toolkit", "DojoX"],
    "maintainers": [{
        "name": "Kitson Kelly"
    }],
    "contributors": [{
        "name": "Kitson Kelly"
    },{
        "name": "Colin Snover"
    }],
    "licenses": [{
        "type": "AFLv2.1",
        "url": "http://bugs.dojotoolkit.org/browser/dojox/trunk/LICENSE#L43"
    },{
        "type": "BSD",
        "url": "http://bugs.dojotoolkit.org/browser/dojox/trunk/LICENSE#L13"
    }],
    "bugs": "https://github.com/example/issues",
    "repositories": [{
        "type": "git",
        "url": "http://github.com/example.git",
        "path": "packages/app"
    }],
    "dependencies": {
        "dojo": "~1.10.4",
        "dijit": "~1.10.4",
        "dojox": "~1.10.4"
    },
    "main": "src",
    "homepage": "http://example.com/",
    "dojoBuild": "app.profile.js"
}
```
 [CommonJS Packages/1.0](http://wiki.commonjs.org/wiki/Packages/1.0)规范为包描述提供了可选项的一个全部列表。如果你计划只在内部使用你的代码，你可以将它退到最小配置，不过你至少要包含`dojoBuild`来让builder能找到你的build配置文件。

##包build配置文件
build profile是build系统的主要配置文件。它是一个JavaScript文件，存放着创建全部build必须的指令的`profile`对象。一个包的最基本build profile如下：
```
var profile = (function(){
    return {
        resourceTags: {
            amd: function(filename, mid) {
                return /\.js$/.test(filename);
            }
        }
    };
})();
```
注意我们如何使用匿名函数。这可以帮你确认环境中的其它代码不会干扰你的配置文件。这也给你提供了做更复杂的自包含计算来生成配置文件的机会。
对于build profile的链接到`dojoBuild`只需要包含一个`resourceTags`来检测它是否不是应用本身的首要build profile。
这里提供了构建你包的builder的“最少”信息。当builder读取这个包时，会将包里的每个文件都传递给`resourceTags`函数来检测标记是否适合该文件。这个过程会传递两个参数，文件名和MID（Module ID）。如果函数返回`true`那么标记适用（当然`false`就是不适用）。
这里在做的就是将每一个以`.js`结尾的文件标记为一个AMD模块，然后builder就会以此处理它，而不是当做一个旧Dojo模块。下面是标记包资源的其他标签：
`amd`
	该资源是一个AMD模块。
`declarative`
	该资源使用声明式标记你想要扫描的依赖项。
`test`
	该资源是包的部分测试代码。
`copyOnly`
	该资源应该复制到目标位置，不能改变。
`miniExclude`
	如果配置选项`mini`为真，该资源不应该被复制到目标位置。
如果你的模块是AMD但你没做`amd`标记，builder通常会提出模块是AMD并且处理它，但是最好还是准确标记。
使用`declarative`标记已超出了本教程的范围。关于这个主体的更多信息，你可以查看 [depsDeclarative](https://dojotoolkit.org/reference-guide/1.10/build/transforms/depsDeclarative.html)。
重要的是确保恰当地标记你的资源以便于builder能正确处理它们的属性。假设你在`src/app/tests`有一个测试（所有游戏的代码都应为它们的包生成单元测试），加上`profile.json`和几个其它类型的只用来复制的文件。一个相对完整的包配置文件应该像下面这样：
```
var profile = (function(){
    var testResourceRe = /^app\/tests\//,
        // checks if mid is in app/tests directory

        copyOnly = function(filename, mid){
            var list = {
                "app/app.profile": true,
                // we shouldn't touch our profile
                "app/package.json": true
                // we shouldn't touch our package.json
            };
            return (mid in list) ||
                (/^app\/resources\//.test(mid)
                    && !/\.css$/.test(filename)) ||
                /(png|jpg|jpeg|gif|tiff)$/.test(filename);
            // Check if it is one of the special files, if it is in
            // app/resource (but not CSS) or is an image
        };

    return {
        resourceTags: {
            test: function(filename, mid){
                return testResourceRe.test(mid) || mid=="app/tests";
                // Tag our test files
            },

            copyOnly: function(filename, mid){
                return copyOnly(filename, mid);
                // Tag our copy only files
            },

            amd: function(filename, mid){
                return !testResourceRe.test(mid)
                    && !copyOnly(filename, mid)
                    && /\.js$/.test(filename);
                // If it isn't a test resource, copy only,
                // but is a .js file, tag it as AMD
            }
        }
    };
})();
```
如你所见，这很快变得相当复杂，但是它思想的本质是配置文件对象需要包含一个`resourceTags`散列，这个散列包含了一个代表不同标记函数的集合。你可以借助JavaScript来找出什么资源用什么标记。
独立包配置文件的示例详见[dgrid Profile](https://github.com/SitePen/dgrid/blob/master/package.js) 。

##应用配置文件
想要在整体build中包含一个包，你只需要按照上一节中说的那样，标记你的资源。但真正创建一个用于生产环境的build，你需要一些额外的设置。这里有两条思路。如果你的应用比较简单并且你有一个包含全部自定义代码的自定义包（如：`app`），你可以在你的包配置文件里创建一个完整配置文件。如果你的应用比较复杂，比如有多个包，或者你想要为不同的build准备不同的build配置文件，那你应该创建一个“应用”build配置文件。

对于剩余的教程，我们假设你将创建一个应用级的配置文件，它位于你应用文件夹根目录下名为`myapp.profile.js`。
创建一个完整build配置文件结构的一些关键设置有：
<table>
    <thead>
        <tr><th>Option</th><th>Type</th><th>Description</th></tr>
    </thead>
    <tbody>
        <tr><td><code>basePath</code></td><td>Path</td><td>This is the "root" of the build, from where the rest of the build will be calculated from.  This is relative to where the build profile is located.</td></tr>
        <tr><td><code>releaseDir</code></td><td>Path</td><td>This is the root directory where the build should go.  The builder will attempt to create this directly and will overwrite anything it finds there.  It is relative to the <code>basePath</code></td></tr>
        <tr><td><code>releaseName</code></td><td>String</td><td>This provides a name to a particular release when outputting it.  This is appended to the <code>releaseDir</code>.  For example if you are going to release your code in <code>release/prd</code> you could set your <code>releaseDir</code> to <code>release</code> and your <code>releaseName</code> to <code>prd</code>.</td></tr>
        <tr><td><code>action</code></td><td>String</td><td>This should be set to <code>release</code>.</td></tr>
        <tr><td><code>packages</code></td><td>Array</td><td>This is an array of hashes of package information which the builder uses when mapping modules.  This provides flexibility in locating in different places and the pulling it together when you build.</td></tr>
        <tr><td><code>layers</code></td><td>Object</td><td>This allows you to create different "layer" modules as part of a build that contain discreet functionality all built into single file.</td></tr>
    </tbody>
</table>

假设我们将要创建一个应用配置文件，这个单页应用将加载两个文件。一个包含了它依赖项中的大部分代码，第二个layer将在一定情况下有条件的加载：

```
var profile = (function(){
    return {
        basePath: "./src",
        releaseDir: "../../app",
        releaseName: "lib",
        action: "release",

        packages:[{
            name: "dojo",
            location: "dojo"
        },{
            name: "dijit",
            location: "dijit"
        },{
            name: "dojox",
            location: "dojox"
        },{
            name: "app",
            location: "app"
        }],

        layers: {
            "dojo/dojo": {
                include: [ "dojo/dojo", "dojo/i18n", "dojo/domReady",
                    "app/main", "app/run" ],
                customBase: true,
                boot: true
            },
            "app/Dialog": {
                include: [ "app/Dialog" ]
            }
        }
    };
})();
```
如果现在就build这个配置文件，它可以运行，我们将得到一个`app/lib`的build，它从四个包获得包含的全部模块，外加两个名为`app/lib/dojo/dojo.js`和`app/lib/app/Dialog.js`特殊文件，它包含全部必需模块及其资源的“build”版本。
现在对于layer的复杂性可能有点糊涂，不过下面我们的将继续深入了解。

##build优化
只生成一个build可能不会满足你一切需求。builder默认压缩任何的layer build，但是不涉及其余的build。这里有几个其它的build配置文件旋钮/选项供你参考：
<table>
    <thead>
        <tr><th>Option</th><th>Type</th><th>Description</th></tr>
    </thead>
    <tbody>
        <tr><td><code>basePath</code></td><td>Path</td><td>This is the "root" of the build, from where the rest of the build will be calculated from.  This is relative to where the build profile is located.</td></tr>
        <tr><td><code>releaseDir</code></td><td>Path</td><td>This is the root directory where the build should go.  The builder will attempt to create this directly and will overwrite anything it finds there.  It is relative to the <code>basePath</code></td></tr>
        <tr><td><code>releaseName</code></td><td>String</td><td>This provides a name to a particular release when outputting it.  This is appended to the <code>releaseDir</code>.  For example if you are going to release your code in <code>release/prd</code> you could set your <code>releaseDir</code> to <code>release</code> and your <code>releaseName</code> to <code>prd</code>.</td></tr>
        <tr><td><code>action</code></td><td>String</td><td>This should be set to <code>release</code>.</td></tr>
        <tr><td><code>packages</code></td><td>Array</td><td>This is an array of hashes of package information which the builder uses when mapping modules.  This provides flexibility in locating in different places and the pulling it together when you build.</td></tr>
        <tr><td><code>layers</code></td><td>Object</td><td>This allows you to create different "layer" modules as part of a build that contain discreet functionality all built into single file.</td></tr>
    </tbody>
</table>

基于上面的特性，你可能想要创建一个build配置文件来提供一个高度优化的应用和其他所需代码的版本。这个配置文件可能要加入下面这些内容：

```
 layerOptimize: "closure",
    optimize: "closure",
    cssOptimize: "comments",
    mini: true,
    stripConsole: "warn",
    selectorEngine: "lite",
```
这可以确保我们的layer经过了闭包压缩，所有其它模块也进行了压缩，我们提高了CSS资源的性能，不会复制演示和测试等额外资源，我们将控制台日志设回最小，并明确要使用哪种CSS选择器。

你可能会问自己“如果将我们所需要的一切build到一个layer，为什么还要管其余的模块？”如果你打算只保留layer文件，不用其余的模块，你可能失去这样一个选项，而将来为了这些模块不得不再做一次完整的build。

输出会包含每一个模块和layer的未压缩格式和一个剥离控制台信息的版本（`.uncompressed.js`和` .consoleStripped.js`）。这些都用来潜在的调试。通常你可以保留它们，但是如果你想要移除（例如web服务器文件空间受限）也可以。

##废代码路径移除
如上所述，`staticHasFeatures`可以结合Closure Compiler很好的优化代码。Dojo使用`has`API来进行特性检测。当特性相关的代码必需时，将`if(has("some-feature")){...}`放在代码里。`staticHasFeatures`可以让你在build时将特性“硬编码”到build，当特性不可用时，闭包将检测到代码不可达并从build文件移除它。

重点要注意，这样创建的build不会完全和非生成版本一样（故意的）。因此你需要仔细考虑你的目标环境并对你的build代码做测试来确保它和预期的一样。

有很多特性可以用，这里有一个特性列表，它普遍适用于完全AMD的Dojo build，为生产浏览器环境设计，并且可配置：
<table class="options">
    <thead>
        <tr><th>Feature</th><th>Setting</th><th>Description</th></tr>
    </thead>
    <tbody>
        <tr><td><code>config-deferredInstrumentation</code></td><td>0</td><td>Disables automatic loading of code that reports un-handled rejected promises</td></tr>
        <tr><td><code>config-dojo-loader-catches</code></td><td>0</td><td>Disables some of the error handling when loading modules.</td></tr>
        <tr><td><code>config-tlmSiblingOfDojo</code></td><td>0</td><td>Disables non-standard module resolution code.</td></tr>
        <tr><td><code>dojo-amd-factory-scan</code></td><td>0</td><td>Assumes that all modules are AMD</td></tr>
        <tr><td><code>dojo-combo-api</code></td><td>0</td><td>Disables some of the legacy loader API</td></tr>
        <tr><td><code>dojo-config-api</code></td><td>1</td><td>Ensures that the build is configurable</td></tr>
        <tr><td><code>dojo-config-require</code></td><td>0</td><td>Disables configuration via the <code>require()</code>.</td></tr>
        <tr><td><code>dojo-debug-messages</code></td><td>0</td><td>Disables some diagnostic information</td></tr>
        <tr><td><code>dojo-dom-ready-api</code></td><td>0</td><td>Ensures that the DOM ready API is available</td></tr>
        <tr><td><code>dojo-firebug</code></td><td>0</td><td>Disables Firebug Lite for browsers that don't have a developer console (e.g. IE6)</td></tr>
        <tr><td><code>dojo-guarantee-console</code></td><td>1</td><td>Ensures that the console is available in browsers that don't have it available (e.g. IE6)</td></tr>
        <tr><td><code>dojo-has-api</code></td><td>1</td><td>Ensures the has feature detection API is available.</td></tr>
        <tr><td><code>dojo-inject-api</code></td><td>1</td><td>Ensures the cross domain loading of modules is supported</td></tr>
        <tr><td><code>dojo-loader</code></td><td>1</td><td>Ensures the loader is available</td></tr>
        <tr><td><code>dojo-log-api</code></td><td>0</td><td>Disables the logging code of the loader</td></tr>
        <tr><td><code>dojo-modulePaths</code></td><td>0</td><td>Removes some legacy API related to loading modules</td></tr>
        <tr><td><code>dojo-moduleUrl</code></td><td>0</td><td>Removes some legacy API related to loading modules</td></tr>
        <tr><td><code>dojo-publish-privates</code></td><td>0</td><td>Disables the exposure of some internal information for the loader.</td></tr>
        <tr><td><code>dojo-requirejs-api</code></td><td>0</td><td>Disables support for RequireJS</td></tr>
        <tr><td><code>dojo-sniff</code></td><td>1</td><td>Enables scanning of data-dojo-config and djConfig in the dojo.js script tag</td></tr>
        <tr><td><code>dojo-sync-loader</code></td><td>0</td><td>Disables the legacy loader</td></tr>
        <tr><td><code>dojo-test-sniff</code></td><td>0</td><td>Disables some features for testing purposes</td></tr>
        <tr><td><code>dojo-timeout-api</code></td><td>0</td><td>Disables code dealing with modules that don't load</td></tr>
        <tr><td><code>dojo-trace-api</code></td><td>0</td><td>Disables the tracing of module loading.</td></tr>
        <tr><td><code>dojo-undef-api</code></td><td>0</td><td>Removes support for module unloading</td></tr>
        <tr><td><code>dojo-v1x-i18n-Api</code></td><td>1</td><td>Enables support for v1.x i18n loading (required for Dijit)</td></tr>
        <tr><td><code>dom</code></td><td>1</td><td>Ensures the DOM code is available</td></tr>
        <tr><td><code>host-browser</code></td><td>1</td><td>Ensures the code is built to run on a browser platform</td></tr>
        <tr><td><code>extend-dojo</code></td><td>1</td><td>Ensures pre-Dojo 2.0 behavior is maintained</td></tr>
    </tbody>
</table>

关于可用特性和特性检测详见[dojo/has](https://dojotoolkit.org/reference-guide/1.10/dojo/has.html) 参考指南。

为了实现它，需要在你的build配置文件里加入以下内容：

```
staticHasFeatures: {
    "config-deferredInstrumentation": 0,
    "config-dojo-loader-catches": 0,
    "config-tlmSiblingOfDojo": 0,
    "dojo-amd-factory-scan": 0,
    "dojo-combo-api": 0,
    "dojo-config-api": 1,
    "dojo-config-require": 0,
    "dojo-debug-messages": 0,
    "dojo-dom-ready-api": 1,
    "dojo-firebug": 0,
    "dojo-guarantee-console": 1,
    "dojo-has-api": 1,
    "dojo-inject-api": 1,
    "dojo-loader": 1,
    "dojo-log-api": 0,
    "dojo-modulePaths": 0,
    "dojo-moduleUrl": 0,
    "dojo-publish-privates": 0,
    "dojo-requirejs-api": 0,
    "dojo-sniff": 1,
    "dojo-sync-loader": 0,
    "dojo-test-sniff": 0,
    "dojo-timeout-api": 0,
    "dojo-trace-api": 0,
    "dojo-undef-api": 0,
    "dojo-v1x-i18n-Api": 1,
    "dom": 1,
    "host-browser": 1,
    "extend-dojo": 1
},
```
虽然这对于大多数web应用已经大体很安全，但没什么能取代build版本在目标平台上的测试。再次提醒你，使用`staticHasFeatures`你将在根本上改变路径。

##Layers
很多情况下多层文件是有益的。最常见的情况是你的代码分几个不同的部分并可以按需加载。例如，在一个Web邮件应用中也有一个日历组件，通用代码、邮件部分、日历组件可以分解到分离的layers。这可以让只使用邮件服务的用户避免加载日历代码，同时也确保邮件和日历都用到的用户不会两次下载共享的代码。创建这些layer非常简单：

```
var profile = {
    layers: {
        "app/main": {
            include: [ "app/main" ],
            exclude: [ "app/mail", "app/calendar" ]
        },
        "app/mail": {
            include: [ "app/mail" ],
            exclude: [ "app/main" ]
        },
        "app/calendar": {
            include: [ "app/calendar" ],
            exclude: [ "app/main" ]
        }
    }
};
```
在上面的例子中，build系统创建个layer：一个包含main应用，一个包含邮件组件，一个包含日历组件。通过从`app/main`排除`app/mail`和`app/calendar`，这些模块（和它们的所有依赖项）都从main layer排除了。（同样也将`app/main`从`app/mail`和`app/calendar`layer排除了。）

注意如果还有其它没有被`app/main`的非排除依赖链require的共享组件，你需要把它们添加到layer的include的模块列表里。例如，如果邮件和日历组件都使用DataGrid，但是在`app/main`没有什么要引用它，那就需要显式的指定在main layer来避免分别编译到`app/mail`和`app/calendar`里。

```
var profile = {
    layers: {
        "app/main": {
            include: [ "app/main", "dojox/grid/DataGrid" ],
            exclude: [ "app/mail", "app/calendar" ]
        },
        "app/mail": {
            include: [ "app/mail" ],
            exclude: [ "app/main" ]
        },
        "app/calendar": {
            include: [ "app/calendar" ],
            exclude: [ "app/main" ]
        }
    }
};
```
也可以创建一个`dojo.js`的自定义build；特别在使用AMD时，由于默认（为了向后兼容），build系统自动将`dojo/main`模块添加到`dojo.js`，但你的代码可能实际并不适用这些加载模块，因此浪费了空间。为了创建一个`dojo.js`的自定义build，你只要将它定义为一个分离的layer，把 `customBase` 和`boot`设置为`true`：

```
var profile = {
    layers: {
        "dojo/dojo": {
            include: [ "dojo/dojo", "app/main" ],
            customBase: true,
            boot: true
        }
    }
};
```
`customBase`指令阻止 `dojo/main`的自动添加， `boot`确保文件包含必须的AMD加载器代码。将`dojo/main`添加到`include`列表直接将`dojo/main`（和它的依赖项）build到`dojo.js`里。

设计你的layer很有挑战性，尤其是当你想要在build里利用其它包时。在 [SitePen blog post](http://www.sitepen.com/blog/2012/06/11/dgrid-and-dojo-nano-build/)有一个关于处理layers和优化build的很好的示例。

##默认配置
最后一项是关于build的默认配置，它们可以在build里设定。它可以被应用或代码重写，但是如果你在为一个特定的应用做一个特定的build，使用它会非常方便。另外，你可以指定一个`hasCache`。这类似于静态has特性，但是不同于硬编码，它简单的“告诉”代码：特性的相关设置是什么，所以它不需要花周期计算出来。我们可以在build配置文件里加入下面内容：

```
defaultConfig: {
    hasCache:{
        "dojo-built": 1,
        "dojo-loader": 1,
        "dom": 1,
        "host-browser": 1,
        "config-selectorEngine": "lite"
    },
    async: 1
},
```
以上适用于一个全AMD的应用。如果在你的生成应用中值适用一个配置信息，你可以在build时轻易地扩展其它内容，然后在你加载应用时忽略它们。

##整合
我们把上面的全部整合成一个“应用”build配置文件，如下：

```
var profile = (function(){
    return {
        basePath: "./src",
        releaseDir: "../../app",
        releaseName: "lib",
        action: "release",
        layerOptimize: "closure",
        optimize: "closure",
        cssOptimize: "comments",
        mini: true,
        stripConsole: "warn",
        selectorEngine: "lite",

        defaultConfig: {
            hasCache:{
                "dojo-built": 1,
                "dojo-loader": 1,
                "dom": 1,
                "host-browser": 1,
                "config-selectorEngine": "lite"
            },
            async: 1
        },

        staticHasFeatures: {
            "config-deferredInstrumentation": 0,
            "config-dojo-loader-catches": 0,
            "config-tlmSiblingOfDojo": 0,
            "dojo-amd-factory-scan": 0,
            "dojo-combo-api": 0,
            "dojo-config-api": 1,
            "dojo-config-require": 0,
            "dojo-debug-messages": 0,
            "dojo-dom-ready-api": 1,
            "dojo-firebug": 0,
            "dojo-guarantee-console": 1,
            "dojo-has-api": 1,
            "dojo-inject-api": 1,
            "dojo-loader": 1,
            "dojo-log-api": 0,
            "dojo-modulePaths": 0,
            "dojo-moduleUrl": 0,
            "dojo-publish-privates": 0,
            "dojo-requirejs-api": 0,
            "dojo-sniff": 1,
            "dojo-sync-loader": 0,
            "dojo-test-sniff": 0,
            "dojo-timeout-api": 0,
            "dojo-trace-api": 0,
            "dojo-undef-api": 0,
            "dojo-v1x-i18n-Api": 1,
            "dom": 1,
            "host-browser": 1,
            "extend-dojo": 1
        },

        packages:[{
            name: "dojo",
            location: "dojo"
        },{
            name: "dijit",
            location: "dijit"
        },{
            name: "dojox",
            location: "dojox"
        },{
            name: "app",
            location: "app"
        }],

        layers: {
            "dojo/dojo": {
                include: [ "dojo/dojo", "dojo/i18n", "dojo/domReady",
                    "app/main", "app/run" ],
                customBase: true,
                boot: true
            },
            "app/Dialog": {
                include: [ "app/Dialog" ]
            }
        }
    };
})();
```
这个build脚本执行如下：

 1. 为`./src`的build指定一个基础路径。
 2. 指定一个发布目录和名称，它们将输出到`app/lib`。
 3. 使用 Closure Compiler优化layers和模块。
 4. 通过移除注释和行内`@import`来优化layers。
 5. 不复制测试和示例这些额外资源。
 6. 从代码移除`console.log`行。
 7. 为build过的代码使用`lite`CSS选择器引擎。
 8. 提供了一个默认的`dojoConfig`。
 9. 将某些特性“固定”进或出代码，这样不用的特性可以通过Closure Compiler来移除
 10. 识别我们build的每一个包。
 11. 构建两个layer，一个叫做`app/lib/dojo/dojo.js`包含我们应用的大部分代码，在一个文件里加入必须的Dojo代码，一个叫做`app/lib/app/Dialog.js`包含支持`app/Dialog`模块任何额外代码，应用可以动态地加载这些模块。

我们已经提供了build文件里包的位置。有好几种提供包位置的方式：

 - 使用`--require`命令行标记来指向一个脚本，它包含一个`require`对象或`require(config)`调用。
 - 使用`--dojoConfig`命令行标记来指向一个脚本，它包含一个带有包配置数据的`dojoConfig`对象。
 - 使用多重`--package`命令行标记来指向每一个包目录（注意它们必须都有`package.json`描述文件）。
 - 在build配置文件里指定包配置信息，像我们上面所做的一样。

##Building
现在各种配置文件都已经设置好了，到了真正创建build的时候。感谢之前所做的准备，实际运行build非常简单。在OSX或者Linux上，你可以在应用的根目录运行以下命令：

```
$ src/util/buildScripts/build.sh --profile myapp.profile.js
```
或者在Windows上：

```
> src\util\buildScripts\build.bat --profile myapp.profile.js
```
这将使用我们为应用创建的build配置文件启动一个build。其实可以在命令行指定很多不同的配置选项；运行`build.sh --help`（或者Windows上`build.bat --help`）来获取列表，不过我们建议你保持简洁并将你的选项放在build配置文件里。最好在build命令后附加`--check-args`来查看build系统的详细信息，这样会输出一个表示build配置过程的JSON。

当使用Closure Compiler运行时，你将看见一些关于`dojox`的警告和错误信息。这些可以先忽略，在工具集的未来版本将会进行修正。当使用`staticHasFeatures`时你将看到关于不可达代码的警告信息，这其实正是它应该做的。

##小结
build系统是部署应用的关键。即时Dojo1.10内置了异步加载机制，build过的应用也不没build过的明显快的多。加载时间是用户体验的关键因素，所以不要发布一个未经build的应用。

##其他资源

 - [Defining Modules](https://dojotoolkit.org/documentation/tutorials/1.10/modules/) 教程
 - [Configuring Dojo with dojoConfig](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config)教程
 - [dojo/has](https://dojotoolkit.org/reference-guide/1.10/dojo/has.html)参考指南
 - [the build system](https://dojotoolkit.org/reference-guide/1.10/build/index.html)参考指南
 - [generator-dojo](https://github.com/bryanforbes/generator-dojo)
 - [Working with Dojo and AMD in production](http://www.sitepen.com/blog/2012/08/27/working-with-dojo-and-amd-in-production/)
