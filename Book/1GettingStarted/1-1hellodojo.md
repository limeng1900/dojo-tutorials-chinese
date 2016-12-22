# 1.1 Hello Dojo

##简介
本篇入门，主要是关于Dojo的加载和一些核心功能，另外还有基于AMD的模块加载，还有如何在出现错误时寻求帮助。 

##入门
开始用Dojo只需要把dojo.js文件包含在web页面里，方式和其他javascript文件一样。Dojo在主流CDN上也有，先来把下面的代码放在一个文件里，以`hellodojo.html`命名该文件，然后在浏览器里打开。

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Tutorial: Hello Dojo!</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>
    <!-- load Dojo -->
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
            data-dojo-config="async: true"></script>
</body>
</html>
```
通常，当你了加载一个库的js文件就可以使用它的所有方法了。Dojo以前也是这样的，但1.7版后源码采取[异步模块定义(AMD)](https://github.com/amdjs/amdjs-api/wiki/AMD)，可以实现完全模块化的web应用开发。选择AMD是因为它使用纯javascript，使源文件在浏览器中正常工作的同时支持生产资源优化的build过程来提高部署时的应用性能。

那么在`dojo.js`加载之后提供什么功能呢？那就是Dojo的AMD加载器，它定义了[两个全局函数](https://dojotoolkit.org/reference-guide/1.10/loader/amd.html#the-amd-api)`require`和`define`。AMD细节见[Introduction to AMD tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/modules/)。在入门阶段你只需要知道require用来加载并使用模块、define用来自定义模块。一个模块就是一个单独的javascript源文件。

还有几个HTML DOM 操作的Dojo基本模块是[dojo/dom](https://dojotoolkit.org/reference-guide/1.10/dojo/dom.html) 和 [dojo/dom-construct](https://dojotoolkit.org/reference-guide/1.10/dojo/dom-construct.html)。下面我们来看看如何加载和使用这几个模块：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Tutorial: Hello Dojo!</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>
    <!-- load Dojo -->
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
            data-dojo-config="async: true"></script>

    <script>
        require([
            'dojo/dom',
            'dojo/dom-construct'
        ], function (dom, domConstruct) {
            var greetingNode = dom.byId('greeting');
            domConstruct.place('<em> Dojo!</em>', greetingNode);
        });
    </script>
</body>
</html>
```
`require`的第一个参数（14-17行）是一个你需要加载的模块id组成的数组。通常，它们直接对应文件名，如果你下载了[Dojo源代码](https://dojotoolkit.org/download/)，就能在Dojo目录里找到定义了这些模块的`dom.js`和`dom-constuct.js`。

AMD加载器进行异步操作，并且用以回调的方式实现了JavaScript中的异步操作，`require`（17行）的第二个参数就是一个回调函数。回调函数中，就是你提供的使用那些模块的代码。AMD加载器将模块作为参数传递给你的回调函数（它们和模块id数组中的顺序一致）。你可以随意命名这些参数，但为了保障代码一致性和可读性，我们推荐你基于模块id来命名。

在第18-19行可以看到，我们运用了`dom`和`dom-construct`模块，它通过id获取一个DOM节点并操作其内容。

对于你请求的模块，AMD加载器会自动加载它的所有依赖模块，所以要只需要列出你直接用到的模块即可。

##定义AMD模块
下面有一个加载和使用模块的示例。要定义和加载你自己的模块，先要确认你是从HTTP服务器加载HTML文件的（本地也可以，但是由于有一些安全敏感设置会阻止许多使用“file:///”协议的内容，所以你需要一个HTTP服务器）。对于这些例子，你的web服务器只需要提供文件的能力，不需要其他任何花哨的功能。在你的目录下添加一个`demo`目录，它包含你的`hellodojo.html`文件，并在`demo`目录下创建一个`myModule.js`文件。

```
demo/
    myModule.js
hellodojo.html
```
现在在myModule.js中输入：

```
define([
    // 这个模块需要用到dojo/dom模块，所以要把它放进依赖模块列表中。
    'dojo/dom'
], function(dom){
    // 一旦依赖模块列表中的模块全都加载完成，就会调用这个函数来定义demo/myModule模块。
    //
    // dojo/dom 作为第一个参数传递给这个函数，依赖列表中的其他模块会作为随后的参数传入。

    var oldText = {};

    // 这个返回的对象成为该模块定义的值
    return {
        setText: function (id, text) {
            var node = dom.byId(id);
            oldText[id] = node.innerHTML;
            node.innerHTML = text;
        },

        restoreText: function (id) {
            var node = dom.byId(id);
            node.innerHTML = oldText[id];
            delete oldText[id];
        }
    };
});
```
ADM `define`函数的参数和`require`函数相似，一个模块id数组和一个回调函数。AMD加载器将回调函数的返回值存储为该模块的值，然后当其他代码通过require（或者define）加载该模块时，会接收到该模块定义的返回值。

##CDN的使用
通过CDN来使用Dojo时，加载本地模块需用一些额外配置（关于配置Dojo的AMD加载器和通过CDN使用Dojo的更多信息请参考[AMD进阶](https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/)和[在CDN下使用模块](https://dojotoolkit.org/documentation/tutorials/1.10/cdn/)的教程）。像下面这样更新`hellodojo.html`的代码：

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Tutorial: Hello Dojo!</title>
</head>
<body>
    <h1 id="greeting">Hello</h1>
    <!-- 配置 Dojo -->
    <script>
        // 为了代替data-dojo-config，我们在加载dojo.js之前创建一个dojoConfig对象；
        //它们在功能上完全相同，只是这种方法对于大型配置来说更容易读懂。
        var dojoConfig = {
            async: true,
            // 这个代码注册“demo”包的正确位置，这样我们就能够在从CDN加载Dojo的同时加载本地模块了
            packages: [{
                name: "demo",
                location: location.pathname.replace(/\/[^/]*$/, '') + '/demo'
            }]
        };
    </script>
    <!-- 加载 Dojo -->
    <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"></script>

    <script>
        require([
            'demo/myModule'
        ], function (myModule) {
            myModule.setText('greeting', 'Hello Dojo!');

            setTimeout(function () {
                myModule.restoreText('greeting');
            }, 3000);
        });
    </script>
</body>
</html>
```
除了添加Dojo配置之外，我们还重新定义了主要JS代码，现在它只加载`demo/myModule`，并利用该模块完成页面上的文本操作。由此可见，定义和加载模块其实很简单。我们也修改了指向`dojo.js`的URL，省略了协议（26行），它创建一个跟页面使用相同协议（http或https）的链接，这样可以防止会让一些浏览器发出安全警报的混合内容。

在AMD模块中组织代码，可让你创建模块化的JavaScript资源，它们可以立即在浏览器中执行，并且易于调试。AMD模块中变量使用局部作用域，避免搅乱全局命名空间的同时，还提供了更快的名称解析。AMD是一个有着多重实现的标准规范，所以你不会被局限在单一实现方式上 —— 任何AMD加载器都可以使用AMD模块。

##等待DOM

有一件事通常是web应用必须要实现的，那就是确保浏览器在执行代码前DOM是可用的。我们通过一个特殊的AMD模块—— plugin（插件）实现这个功能。插件可以像其他模块一样require，但是要在模块标识符后加个感叹号（!）来激活他们的特殊功能。对于DOM ready事件，Dojo提供`dojo/domReady`插件。只要在任何`require`或`define`调用时，将这个插件包含在依赖列表里，那么对应的回调在DOM准备好之前就不会触发：

```
require([
    'dojo/dom',
    'dojo/domReady!'
], function (dom) {
    var greeting = dom.byId('greeting');
    greeting.innerHTML += ' from Dojo!';
});
```
上面的例子简单地在`greeting`元素中添加一些文本，这件事只能在DOM加载后进行（先前不用这个是由于`script`元素放在`body`元素的底部，这样会将脚本的处理延迟到DOM加载之后）。注意，模块标识符以“!”结尾，否则`dojo/domReady`模块会变得和普通模块一样。

有时候，比如dojo/domReady，我们加载一个模块只是为了利用它的副作用，并不需要引用它。但是AMD加载器并不知道这个，它总是会将依赖数组里的每一个模块都引用给回调函数，所以任何你不需要返回值的模块都应该放在依赖数组的末尾，并且在回调函数的参数列表中忽略它们。

DOM操作函数的更多信息参见 [Dojo DOM Functions](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/)。

##添加视觉特效

现在我们可以给页面添加一些动画来让它更生动。我们可以加载模块`dojo/fx`来进行。下面使用`dojo/fx`的`slideTo`方法给欢迎辞添加一个滑动动画。

```
require([
    'dojo/dom',
    'dojo/fx',
    'dojo/domReady!'
], function (dom, fx) {
    // 我们之前有的
    var greeting = dom.byId('greeting');
    greeting.innerHTML += ' from Dojo!';

    // 现在，有了动画
    fx.slideTo({
        node: greeting,
        top: 100,
        left: 200
    }).play();
});
```
如你所见，我们已经添加了一个`dojo/fx`依赖，然后使用这个模块在`greeting`元素上播放动画。
> 特效和动画的更多信息请参考 [Dojo Effects](https://dojotoolkit.org/documentation/tutorials/1.10/effects/) 和 [Animations](https://dojotoolkit.org/documentation/tutorials/1.10/animation/)教程。

##使用Dojo资源

CDN很方便，我们在教程的例子中使用它是因为你直接复制代码就可以运行了，不需要做任何修改。不过它有一些劣势：

 - 为了保证性能，它们是一个Dojo的“build”版本，就是说每个模块都进行了压缩和优化以便于在网上高效传输。那么也就是说当出现问题时，调试会很困难。
 - 它要求你必须连接互联网来使用你的应用，很多情况下这不太现实。
 - 当你要包含自定义模块时就需要更多的努力。
 - 如果你想要将你的应用产品化，定制化build版本的Dojo能够针对你特定的应用和浏览器提高性能，但使用一刀切的CDN build你实现不了。
 
按照以下步骤使用Dojo资源，它们是你使用Dojo开发项目的通用方式：

1.[下载Dojo](https://dojotoolkit.org/download/)——查看本文结尾部分并下载发布源代码。
    
如果你熟悉[git](http://git-scm.com/)和[GitHub](https://github.com/dojo/)，你可以从[GitHub克隆Dojo](https://github.com/dojo/)。你至少要下载[dojo](https://github.com/dojo/dojo)，某种程度上你可能还会想要[dijit](https://github.com/dojo/dijit)、[dojox](https://github.com/dojo/dojox)和[util](https://github.com/dojo/util)（这些都包含在源码下载中）。

2.把Dojo放到你的项目文件中，例如：

```
demo/
    myModule.js
dojo/
dijit/
dojox/
util/
hellodojo.html
```

3.本地加载dojo.js，比CDN更好。

```
<script src="dojo/dojo.js"></script>
```

4.更新你的配置文件包

```
var dojoConfig = {
    async: true,
    baseUrl: '.',
    packages: [
        'dojo',
        'dijit',
        'dojox',
        'demo'
    ]
};

```

##获取帮助

任何时候当你困惑或遇到棘手问题时，你都不是一个人在战斗！志愿者们随时准备着通过[dojo-interest mailing list ](http://mail.dojotoolkit.org/mailman/listinfo/dojo-interest)的邮件和通过 #dojo on irc.freenode.net[](https://dojotoolkit.org/chat)的IRC来提供帮助。如果你在我们的文档中发现了问题，或者觉得哪里有所误导或混淆，可以用文档底部的反馈页面来告知我们。

如果你需要紧急或者秘密的帮助，或者有志愿者团队解决不了的问题，可以通过SitePen找到 [commercial Dojo support](http://www.sitepen.com/support/)和[training workshops](http://www.sitepen.com/workshops/)。

##下一步做什么？

Dojo Toolkit的入门只要简单的添加一个脚本标签和require一些模块就可以，但是Dojo巨大的领域和力量意味着我们这才刚刚接触到了皮毛。根据你的需求，本系列教程提供几个不同的学习路径：

- 如果你之前用过Dojo，并且想要更好地理解AMD的世界及旧Dojo，理解其他已经改变的概念，你应该看一看“[新一代Dojo](https://dojotoolkit.org/documentation/tutorials/1.10/modern_dojo/)”教程。
    - 如果你
