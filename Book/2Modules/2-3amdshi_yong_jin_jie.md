#2.3 AMD使用进阶

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/index.html
本翻译GitBook地址：https://www.gitbook.com/book/limeng1900/dojo1-11-tutorials-translation-in-chinese/details

----------

Dojo现在支持以异步模块定义（AMD）格式来编写模块，从而更容易编写和调试代码。本教程中，我们将全面了解这个新模块格式并探索如何用它来编写一个应用。

> 本教程是 [Introduction to AMD](https://dojotoolkit.org/documentation/tutorials/1.10/modules/)的后续，所以请先确认你已理解AMD基础。

本教程中，我们将涉及一个假想的应用，它的文件系统结构如下：

```
/
    index.html
    js/
        lib/
            dojo/
            dijit/
            dojox/
        my/
        util/
```
你可以看到这个结构和之前教程中讨论的结构并不一样，我们将解释如何配置加载器让它工作。不过首先通过更多的细节先回顾下`require`和`define`…

## 深入require
`require` 函数接受以下参数：

 1.  配置（可选，default=undefined）：一个带加载器配置选项的对象——它可以让你在运行时重新配置加载器。
 
 2.  依赖关系（可选，default=[]）：一个模块标识符的数组。指定的模块会在你的代码生成之前解析 。它们会按照列出的顺序加载，并且按顺序以参数的方式传递给你的回调函数。
 
 3.  回调：包含你想要运行的代码的一个函数，这个函数依赖于依赖关系里的模块。你需要将你的代码包裹在回调函数里以支持异步加载和对这些模块使用非全局的引用。
 
> 配置参数可以省略，不需要加上空的占位符。

接下来将涉及加载器配置的更多细节；现在有一个使用配置参数的`require`：

```
require({
    baseUrl: "/js/",
    packages: [
        { name: "dojo", location: "//ajax.googleapis.com/ajax/libs/dojo/1.10.4/" },
        { name: "my", location: "my" }
    ]
}, [ "my/app" ]);
```

这里我们略微修改配置信息让`dojo`包指向Google CDN。跨域加载支持隐含在AMD格式里。

>  请注意不是所有的配置选项可以在运行时配置。尤其是` async` 、`tlmSiblingOfDojo`和已有的`has`测试，一旦加载器加载完成它们就不能改变。此外，大部分的配置数据是浅拷贝，就是说你不能使用这种机制，例如向自定义配置对象添加更多的键值——对象将被覆盖。

## 深入define
`define` 函数接收以下参数：

 1. moduleId （可选，default=undefined）：一个模块标识符。这个参数很大程度上是早期AMD加载器的历史遗留或为支持 pre-AMD Dojo，**不应该提供它**。
 2.  dependencies （可选，default=[]）：是你模块依赖关系的模块标识符的一个数组。如果指定它，这些模块将在你的模块求值之前解析，它们将作为参数按顺序传递给你的工厂函数。
 3.  factory：你模块的值，或者一个将返回值的工厂函数。
 
重要的是要记住，在定义模块的时候，工厂函数只调用一次——返回值会被加载器缓存。在实践层面，这意味着模块可以通过加载相同的模块轻易地共享对象（类似其他语言的静态属性）。

在定义模块的时候，值可以是一个简单的对象：

```
// in "my/nls/common.js"
define({
    greeting: "Hello!",
    howAreYou: "How are you?"
});
```
请记住，如果你不使用工厂函数定义模块。你将无法引用任何依赖关系，所以上面这种定义的类型是很少见的，通常只有被用在i18n绑定或者简单配置对象。

##加载器如何工作？
当你调用`require` 来加载模块，加载器会找到模块的代码然后将它当做一个参数传递给你的回调函数，这样你就可以使用这些模块了。

 1.  首先加载器要先解决你传递的模块标识符。这涉及到将模块标识符和`baseUrl` 联系起来，还要考虑加载的其他配置选项的修改，比如`map`(稍后讨论更多细节)。
 
 2.  此时加载器拥有了模块的URL并且可以加载实际的文件，只需要通过在页面上创建一个新的`script` 元素来并将其`src`属性设置为模块的URL。
 
 3.  一旦文件被加载并解析，其结果将被设为模块的值。
 
 4.  加载器会保留每个模块的引用，在下次请求该模块时，加载器会返回已经存在的引用。

当一个AMD模块加载完成时，代码被插入到页面里一个新的`script`元素，然后调用`define` 函数。在加载传递给`define` 的任何依赖模块时，都会经过上面同样的处理过程，然后将加载器对模块的引用设置为你传递给`define` 的工厂函数的返回值。（如果你向`define` 传递一个值，而不是函数，加载器对你的模块的引用将被设为该值。）

## 配置加载器
因为遗留的兼容性原因，Dojo的加载器默认为同步模式。为了实现异步，我们需要进行显式的配置。通过将`async` 属性配置为`true` ：

```
<script data-dojo-config="async: true" src="js/lib/dojo/dojo.js"></script>
```

你要习惯把开启异步作为标准做法——只有你知道你需要同步行为的时候再关闭它。接下来要做的是将模块的位置信息配置给加载器：

```
var dojoConfig = {
    baseUrl: "/js/",
    tlmSiblingOfDojo: false,
    packages: [
        { name: "dojo", location: "lib/dojo" },
        { name: "dijit", location: "lib/dijit" },
        { name: "dojox", location: "lib/dojox" },
        { name: "my", location: "my", main: "app" }
    ]
};
```
>  记住你必须在加载`dojo.js` 之前设置`dojoConfig`。先看看[Configuring Dojo tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/dojo_config) 如果你还没看的话。
让我们检查下我们使用的配置选项：

 - `baseUrl` （default=dojo.js文件加载的路径）：为库的加载定义基础URL。例如，如果你想要加载"my/widget/Person"，加载器会试着从下面的路径加载：
```
/js/my/widget/Person.js
```
我们可以在文件系统里方便的在任何地方存放我们的文件（在本例中，“js”文件夹），并且只使用模块id相关的部分路径。比如我们不需要`require(["js/my/widget/Person"])`，用`require(["my/widget/Person"])`更加简单。因为在实际加载资源文件的时候，我们将“/js/”配置为所有模块id的基础预设。

 - `tlmSiblingOfDojo` (default = true):默认情况下，加载器会希望从加载器来源文件夹的兄弟文件夹来查找模块（记住，在Dojo中当你的script元素加载`dojo.js`时，就加载了加载器）。如果你的文件结构像下面这样：
```
/
    js/
        dojo/
        dijit/
        dojox/
        my/
        util/
```
这样你就不需要配置`baseUrl` 或者 `tlmSiblingOfDojo`，你的最高级模块都是`dojo.js` 所在文件夹的兄弟文件夹，所以`tlmSiblingOfDojo` 就是true。

 - packages：一个包配置对象的数组。在最基本的层面上，包是模块的简单集合。dojo， dijit和dojox都是包的范例。不像放在一个目录里下一个简单模块集合，packages还有一些额外功能，它能显著提高模块的可移植性和易用性。一个便携包是独立的，也可以通过[cpm](https://github.com/kriszyp/cpm)这样的工具安装。你可以指定一个包的以下各项：
     
     - name：包的名字，应该和包含该模块的文件名一致。
     
     - location：包的位置；可以是相对于`baseUrl` 的路径或者一个绝对路径。相比"lib/dojo/dom" ，我们更希望从  "dojo/dom"来加载模块（再看一眼本教程开头的文件结构）。所以我们将`location` 属性指定为"lib/dojo"。就是说尝试加载 "dojo/dom"模块的加载器会加载 "/js/lib/dojo/dom.js" 文件（记住，因为` baseUrl` 里预设了“js”）。
     
     - main（可选，default=main.js）:当试图require包本身时，用来加载正确的模块。例如，如果你想要require“dojo”，实际加载的文件是"/js/dojo/main.js"。由于我们已经为“my”包重写了这个属性，如果require“my”，实际会加载"/js/my/app.js"。

> 如果我们尝试require "util"，它是一个还没有定义的包，加载器会试着加载"/js/util.js"。你应该每次都在加载器配置里定义你全部的包。

## 使用便携模块

新AMD加载器的一个最重要的特性是能够创建完全的便携包。例如，如果你有一个应用需要用到Dojo的两个不同版本中的模块，新加载器就非常方便。

假设你有一个建立在旧版本Dojo上的应用，你想要更新到最新的1.10版本，但是Dojo有一些更新导致你的旧代码无法使用。在为旧代码使用旧版Dojo的同时，你可以将新代码升级到当前Dojo的发布版本。这可以通过`map`  配置属性完成：

```
dojoConfig = {
    packages: [
        { name: "dojo16", location: "lib/dojo16" },
        { name: "dijit16", location: "lib/dijit16" },
        { name: "dojox16", location: "lib/dojox16" },
        { name: "dojo", location: "lib/dojo" },
        { name: "dijit", location: "lib/dijit" },
        { name: "dojox", location: "lib/dojox" },
        { name: "myOldApp", location: "myOldApp" },
        { name: "my", location: "my" }
    ],
    map: {
        myOldApp: {
            dojo: "dojo16",
            dijit: "dijit16",
            dojox: "dojox16"
        }
    }
};
```
这里发生了什么？

 - （3-5行）首先定义了三个包，它们指向包含旧版Dojo的文件夹。
 
 - （6-8行）接下来定义三个当前发行版本的包。
 
 - （9-10行）为旧代码和当前代码定义包
 
 - （12-18行）定义一个`map` 配置：它将应用到 "myOldApp"模块，并且将模块对 "dojo"、"dijit"和"dojox" 包的请求分别映射到"dojo16"、"dijit16"和"dojox16"。
 
 - 来源于“my“包的模块将从当前Dojo发行版本的 dojo、 dijit、 dojox加载模块。

你可以从 [AMD Configuration documentation](https://github.com/amdjs/amdjs-api/wiki/Common-Config#map-) 获得更多关于`map` 的信息。

如果你已经很熟悉加载器，特别是 `packageMap` 属性，请注意它已经弃用了，更先进的配置选项是`map` 。

## 编写便携模块

你可以（也应该）确保的是，你创建的包内部的模块总是从该包内部加载文件，通过用**相对** 的模块标识符指定依赖关系。下面给出在“my”包里的模块的代码：

```
// in "my/widget/NavBar.js"
define([
    "dojo/dom",
    "my/otherModule",
    "my/widget/InfoBox"
], function(dom, otherModule, InfoBox){
    // …
});
```
用相对关系标识符代替从my包明确的模块请求：

```
// in "my/widget/NavBar.js"
define([
    "dojo/dom",
    "../otherModule",
    "./InfoBox"
], function(dom, otherModule, InfoBox){
    // …
});
```
相对于"my/widget/NavBar"：

 - "dojo/dom"在一个分离的包里，所以使用全的标识符
 
 - "my/otherModule"在上一级目录，所以使用“../”
 
 - "my/widget/InfoBox"在同一目录，所以使用"./"
    > 如果你只指定"InfoBox"，它会将其理解为一个包的名字，所以标识符必须以"./"开头。

> 记住相对标识符只能用来引用同一个包里的模块。相对模块id也只在定义模块时有效，在传递给`require` 的依赖列表中是不起作用的。

考虑相对标识符对同一个包的作用，再回头看`map` 的例子是不是发现了一些问题？为了简单起见，我们把重点放在让你的应用一部分使用旧版Dojo而另一部分用当前版本的实现上。但是，我们漏掉了一些重要的东西，Dijit依赖于Dojo，DojoX 又同时依赖于Dojo和Dijit。下面的配置将确保这些依赖项正确解析。为了安全起见，将Dojo包映射到他们自己 (`map: { dojo16: { dojo: "dojo16" } }`)，以防止任何模块无法使用相对标识符。

```
var map16 = {
    dojo: "dojo16",
    dijit: "dijit16",
    dojox: "dojox16"
};

dojoConfig = {
    packages: [
        { name: "dojo16", location: "lib/dojo16" },
        { name: "dijit16", location: "lib/dijit16" },
        { name: "dojox16", location: "lib/dojox16" },
        { name: "dojo", location: "lib/dojo" },
        { name: "dijit", location: "lib/dijit" },
        { name: "dojox", location: "lib/dojox" },
        { name: "myOldApp", location: "myOldApp" },
        { name: "my", location: "my" }
    ],
    map: {
        dojo16: map16,
        dijit16: map16,
        dojox16: map16,
        myOldApp: map16
    }
};

```

##有条件地require模块
有时候，你可能想要有条件地require一个模块来回应一些状况。例如，你想要延迟加载一个可选模块，直到一个事件发生。如果你使用显式模块定义，这就很简单：

```
define([
    "dojo/dom",
    "dojo/dom-construct",
    "dojo/on"
], function(dom, domConstruct, on){
    on(dom.byId("debugButton"), "click", function(){
        require([ "my/debug/console" ], function(console){
            domConstruct.place(console, document.body);
        });
    });
});
```
不过，为了变的完全便携，"my/debug/console"需要变成一个相对标识符。单改变它的话不起作用，因为` require ` 调用的时候丢失了原始模块的上下文。为了解决这个问题，Dojo加载器提供一个叫做**上下文相关require（context-sensitive require）**的功能。在你初始化`define` 时，将特殊标识符“require”作为依赖项传递来实现它：

```
// in "my/debug.js"
define([
    "dojo/dom",
    "dojo/dom-construct",
    "dojo/on",
    "require"
], function(dom, domConstruct, on, require){
    on(dom.byId("debugButton"), "click", function(){
        require([ "./debug/console" ], function(console){
            domConstruct.place(console, document.body);
        });
    });
});
```
现在，内部的`require` 使用局部绑定、上下文相关的`require` 函数，所以我们能够相对于“my/debug”来安全地require模块。
> `require` 的上下文是怎么消失的？
>  记住`require` 是一个全局定义的函数。当click时间的处理器执行时，它唯一从模块获得的上下文是局部定义的。它并不知道自己定义在什么模块里。在本地作用域没有“require”，所以将调用全局的“require”。回想本教程自始至终的文件结构，如果我们传递 "./debug/console"给`require` ，它将尝试加载并不存在的"/js/debug/console.js"文件。通过使用上下文相关的`require` ，我们拥有一个改进的`require` 的本地引用，它可以保持模块的上下文，所以能正确的加载"/js/my/debug/console.js"。

上下文相关的`require`在模块加载资源（images, templates, CSS）时也非常有用。先给出以下文件结构：

```
/
    js/
        my/
            widget/
                InfoBox.js
                    images/
                        info.png
```
在`InfoBox.js` 里我们可以调用`require.toUrl` 得到一个定位"info.png"的完整URL，这个URL可以用在`img` 元素里设置`src` 属性。

```
// in my/widget/InfoBox.js
define([
    "dojo/dom",
    "require"
], function(dom, require){
    // 假设 DOM 结构中#infoBoxImage 是一个 img 元素
    dom.byId("infoBoxImage").src = require.toUrl("./images/info.png");
});

```

## 处理循环依赖项
你编程的时候偶尔会碰到这样的情况，两个模块之间互相引用，而这种引用形成了一个循环依赖。为了解决这样的循环依赖，加载器优先解决第一个递归的模块。例如下面的例子：

```
// in "my/moduleA.js"
define([ "./moduleB" ], function(moduleB){
    return {
        getValue: function(){
            return "oranges";
        },

        print: function(){
            // 依赖 moduleB
            log(moduleB.getValue());
        }
    };
});

// in "my/moduleB.js"
define([ "./moduleA" ], function(moduleA){
    return {
        getValue: function(){
            // 依赖 moduleA
            return "apples and " + moduleA.getValue();
        }
    };
});

// in "index.html"
require([
    "my/moduleA"
], function(moduleA) {
    moduleA.print();
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/demo/circular.html)

它看起来应该print "apples and oranges"，但是却出现了` moduleB: Object has no method 'getValue'` 的错误。下面看下当你加载和运行“index.html”时，加载器做了什么：

 1.  解析传递给require的依赖项（在`index.html`里）：`moduleA` 
 2.  解析`moduleA` 的依赖项：`moduleB`
 3.  解析`moduleB` 的依赖项：`moduleA` 
 4.  检测到程序试图解析`moduleA` 
 5.  通过临时将`moduleA` 解析成一个空对象来打破循环依赖。
 6.  通过调用`moduleB` 的工厂函数重新解析它，空对象将作为`moduleA` 传递给工厂函数。
 7.  将工厂函数的返回值传递给加载器作为对`moduleB` 的引用。 
 8.  调用工厂函数重新解析`moduleA` 。
 9. 将`moduleA` 工厂函数的返回值传递加载器作为对`moduleA` 的引用，现在加载器都引用了有效值。`moduleB` 仍引用空对象。
 10.  执行`moduleA.print`，由于 `moduleB` 对`moduleA` 有一个坏的引用，当它调用`moduleA.getValue`时就抛出一个错误。
为了解决这个问题，加载器提供一个特殊的“exports”模块识别符。使用时，这个模块会返回一个指向一个持久对象的引用，这个持久对象代表着已定义的模块，它会初始化为空，不过任何参与循环引用解决方案的模块都将传递向它传递一个引用。同样的引用将传递给将“exports”列为依赖项的模块。下面事情的先后顺序会有一些不同，请看下面更新的代码和随后的解释。

```
// in "my/moduleA.js"
define([ "./moduleB", "exports" ], function(moduleB, exports){
    exports.getValue = function(){
        return "oranges";
    };

    exports.print = function(){
        log(moduleB.getValue());
    };
});

// in "my/moduleB.js"
define([ "./moduleA" ], function(moduleA){
    return {
        getValue: function(){
            return "apples and " + moduleA.getValue();
        }
    };
});

// in "index.html"
require([
    "my/moduleA"
], function(moduleA) {
    moduleA.print();
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/demo/exports.html)
这次加载运行"index.html"时发生的是：

 1.  解析传递给require的依赖项（在`index.html`里）：`moduleA` 
 2.  解析`moduleA` 的依赖项：`moduleB`
 3.  解析`moduleB` 的依赖项：`moduleA` 
 4.  检测到程序试图解析`moduleA` 
 5.  通过临时将`moduleA` 解析成一个空对象来打破循环依赖。
 6.  通过调用`moduleB` 的工厂函数重新解析它，空对象将作为`moduleA` 传递给工厂函数。
 7.  将工厂函数的返回值传递给加载器作为对`moduleB` 的引用。 
 8.  通过调用工厂函数重新解析`moduleA` ，空对象将作为`moduleA` 的占位符以`exports` 参数的方式传递给工厂函数。
 9.  在解析将“exports”列为依赖项的模块之后，加载器对模块的引用不在指向工厂函数的返回值。相反，加载器假设模块在创建的作为占位符的空对象上设置了一些必要属性，并将它作为`exports`参数传递工厂函数。
 10. 执行`moduleA.print`，由于`moduleB` 拥有一个指向`moduleA` 填充对象的有效引用，当它调用`moduleA.getValue` 时就可以像预期的那样执行。

重要的是要记住，虽然使用exports提供一个最终有效的引用，但是在依赖模块（moduleB）解析的时候它仍然是一个空对象。当你的工厂函数（moduleB的）执行时，它接收到指向一个空对象（moduleA）的引用。只有在循环依赖完全解析之后（moduleA临时解析为{ }，moduleB 解析，然后moduleA完全解析），对象添加模块（moduleA）的方法和属性，然后才可以用在你的（moduleB的）工厂函数的函数定义，最后被调用。下面的代码演示了这种差别：

```
// in "my/moduleA.js"
define([ "./moduleB", "exports" ], function(moduleB, exports){
    exports.isValid = true;

    exports.getValue = function(){
        return "oranges";
    };

    exports.print = function(){
        // 依赖 moduleB
        log(moduleB.getValue());
    }
});

// in "my/moduleB.js"
define([ "./moduleA" ], function(moduleA){
    // this code will run at resolution time, when the reference to
    // moduleA is an empty object, so moduleA.isValid will be undefined
    if(moduleA.isValid){
        return {
            getValue: function(){
                return "won't happen";
            }
        };
    }

    // this code returns an object with a method that references moduleA
    // the "getValue" method won't be called until after moduleA has
    // actually been resolved, and since it uses exports, the "getValue"
    // method will be available
    return {
        getValue: function(){
            return "apples and " + moduleA.getValue();
        }
    };
});

// in "index.html"
require([
    "my/moduleA"
], function(moduleA) {
    moduleA.print();
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/demo/exports2.html)

## 加载非AMD代码
在模块标识符部分提到过，AMD加载器也可以通过将JavaScript文件的路径作为标识符传递来加载非AMD代码。加载器这些同属标识符的三种方式：

 - 以“/”开头的标识符
 - 以协议（如“http：”、“https：”）开头的标识符
 - 以“.js”结尾的标识符

当任意代码作为模块加载的时候，模块的解析值为`undefined` ，你需要直接访问定义为全局脚本的代码。

Dojo加载器的一个专属特性是混搭旧Dojo模块和AMD风格模块的能力。这让它可以稳步有序地从旧的代码库转换到AMD代码库，而不是立马改变所有的代码。加载器在同步和异步模式下都可以这样。异步模式下，旧模块的解析值是全局作用域里的对象，这个对象对应`dojo.provide` 调用文件的脚本的解析值。例如：

```
// in "my/legacyModule.js"
dojo.provide("my.legacyModule");
my.legacyModule = {
    isLegacy: true
};
```
当使用`require(["my/legacyModule"])`通过AMD加载器加载这个代码时，分配给`my.legacyModule`的对象将作为这个模块的解析值。

##服务器端JavaScript
新AMD加载器的最后一个特性时能够在服务器端加载JavaScript使用node.js or Rhino。如下通过命令行加载Dojo：

```
# node.js:
node path/to/dojo.js load=my/serverConfig load=my/app

# rhino:
java -jar rhino.jar path/to/dojo.js load=my/serverConfig load=my/app
```
更多细节见 [Dojo and Node.js](https://dojotoolkit.org/documentation/tutorials/1.10/node) 。
一个加载器准备完毕，每个`load=`参数都会向自动解析的依赖项列表里添加一个模块。在浏览器里，同样的代码是这样的：

```
<script data-dojo-config="async: true" src="path/to/dojo.js"></script>
<script>require(["my/serverConfig", "my/app"]);</script>

```

## 小结
新AMD给Dojo带来了许多激动人心的特性和能力，篇幅有限，本教程只对新加载器做了一个简单概述。想了解AMD加载器的新特性请查看 [Dojo loader reference guide](https://dojotoolkit.org/reference-guide/1.10/loader/)。

## 资源

 - [AMD Specification](https://github.com/amdjs/amdjs-api/wiki/AMD)
