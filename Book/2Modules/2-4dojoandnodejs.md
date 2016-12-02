# 2.4Dojo和Node.js
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/node/index.html

----------
JavaScript不仅仅用在客户端。随着Node.js这样的技术的出现，服务器端JavaScript成为一个很好的选择。不止JavaScript能用于服务器端，AMD和Dojo Toolkit也可以。本教程主要关于如何在Node.js下使用Dojo。

## 入门
你应该已经熟悉一些Dojo基础了，比如 [Defining Modules](https://dojotoolkit.org/documentation/tutorials/1.10/modules)和 [Classy JavaScript with dojo/_base/declare](https://dojotoolkit.org/documentation/tutorials/1.10/declare/) 。另外你也应该对 [Node.js ](http://nodejs.org/)有了一定的熟悉，并在本地安装了Node.js和[npm](http://npmjs.org/)包管理工具。在开始之前,你还需要一个Dojo的本地源代码（(Dojo Toolkit SDK）。我们将利用Dojo1.8引进的一些特性，所以确保你用的是最新的发行版本。
另一方面，我准备了一个[GitHub库](https://github.com/kitsonk/dojo-node-boilerplate)，它包含了一个子模块，可以将正确的Dojo部分引入到项目中。它也遵循我们在这个教程中建议的文件布局和目录。
> Dojo的Built distributions或者CDN distributions 在Node.js不起作用，因为他们是基于你在使用浏览器的用户代理的假设生成的。生成和已压缩的代码也会让你的服务器端应用调试变得异常困难，所以还是坚持用源码distribution。

## AMD
你或许会对自己说“AMD是给客户端准备的，我为什么要让它在服务器端运行？”。相比客户端，服务器端没那么需要AMD，AMD在服务器端也没有劣势并且开销很小。这也意味着你可以在客户端和服务器端使用相同的模块和编码风格。使用同一种语言不会让你走的更远，但是如果你只用一种代码风格，就可以更快地编写代码。
遗憾的是对于开发人员来说，如果想要使用自己的包和模块同时在服务器端和浏览器端运行，必须同时支持CommonJS  packaging和模块以及AMD模块。越来越多的包同时支持 CommonJS 和AMD，将来说不定会成为标准。
好消息是Dojo AMD加载器在Node.js下运行很好，只需要一点工作量，你就可以在 AMD/Dojo可用环境下开始编程。

## Bootstrapping
为了在NodeJS下加载AMD模块，我们需要一个模块加载器。Node.js默认的require()只能加载CommonJS模块。Dojo AMD加载器（还有其它加载器，像RequireJS）可以 "bootstrapped"来让你拥有一个“AMD环境”。
但是，在实际加载加载器之前，我们需要组织应用的结构。我建议你的程序使用以下布局：
![这里写图片描述](http://img.blog.csdn.net/20161120145859160)
下面我们要做的就是进一步配置Dojo加载器，让它可以加载另一个模块。一旦我们加载了Dojo加载器，它会进行接管，然后加载的任何模块都会以AMD模块一样的方式运行。下面看看应该在叫做`/server.js`自引导文件里放些什么：

```
// Configuration Object for Dojo Loader:
dojoConfig = {
    baseUrl: "src/", // Where we will put our packages
    async: 1, // We want to make sure we are using the "modern" loader
    hasCache: {
        "host-node": 1, // Ensure we "force" the loader into Node.js mode
        "dom": 0 // Ensure that none of the code assumes we have a DOM
    },
    // While it is possible to use config-tlmSiblingOfDojo to tell the
    // loader that your packages share the same root path as the loader,
    // this really isn't always a good idea and it is better to be
    // explicit about our package map.
    packages: [{
        name: "dojo",
        location: "dojo"
    },{
        name: "app",
        location: "app"
    },{
        name: "app-server",
        location: "app-server"
    }]
};

// Now load the Dojo loader
require("./src/dojo/dojo.js");

```

## 加载模块
现在我们可以运行这个文件，但其实它并不会做什么，因为还没有加载任何模块或执行任何功能代码。那么，再来创建一个名叫`src/app-server/server.js` 的简单模块，它会在控制台记录一条信息。

```
require([], function(){
    console.log("Hello World!");
});
```
我们也需要引导脚本来加载这个模块，最简单的方法是将模块包含在配置信息的`deps`里。我喜欢用更容易修改和配置的方法：

```
// The module to "bootstrap"
var loadModule = "app-server/server";

// Configuration Object for Dojo Loader:
dojoConfig = {
    baseUrl: "src/", // Where we will put our packages
    async: 1, // We want to make sure we are using the "modern" loader
    hasCache: {
        "host-node": 1, // Ensure we "force" the loader into Node.js mode
        "dom": 0 // Ensure that none of the code assumes we have a DOM
    },
    // While it is possible to use config-tlmSiblingOfDojo to tell the
    // loader that your packages share the same root path as the loader,
    // this really isn't always a good idea and it is better to be
    // explicit about our package map.
    packages: [{
        name: "dojo",
        location: "dojo"
    },{
        name: "app",
        location: "app"
    },{
        name: "app-server",
        location: "app-server"
    }],
    deps: [ loadModule ] // And array of modules to load on "boot"
};

// Now load the Dojo loader
require("./src/dojo/dojo.js");
```
现在我们只需要在项目的根目录下运行应用：

```
$ node server
Hello World!
$
```
> 上面提到的样板也有一个名叫`boot.js`的简便工具，用来从命令行引导模块。例如，你可以运行`node boot "app-server/server"`来运行服务。

## 使用Node.js模块
一旦你进入AMD的世界，你就可以用AMD的方式来require()和define()。你可以像客户端那样加载模块。当你使用Node.js使用的标准CommonJS模块时，挑战随之而来。由于`require()`以AMD重写了，所以只调用`require()`不再能加载CommonJS模块。Dojo有相应的解决方案，在Dojo1.8之后，有一个名叫[`dojo/node`](https://dojotoolkit.org/reference-guide/1.10/dojo/node.html) 的AMD插件模块可以用来加载CommonJS模块。
插件的参数（!后面的字符串）就是当你使用原生Node.js require()的模块。例如：

```
// This:
var fs = require("fs");

// Would become:
require([ "dojo/node!fs" ], function(fs){
    // Utilise the "fs" module
});
```
让我们修改`app-server/server.js`来加载Node.js `fs`模块，读取一个JSON文件，解析并记录到控制台：

```
require([
    "dojo/node!fs"
], function(fs){
    var helloworld = fs.readFileSync("helloworld.json");
    console.log(JSON.parse(helloworld));
});
```
在项目的根目录，创建·require([
    "dojo/node!fs"
], function(fs){
    var helloworld = fs.readFileSync("helloworld.json");
    console.log(JSON.parse(helloworld));
});·

```
{
    "Hello": "World!"
}
```
然后再次运行代码，会得到下面的结果：

```
$ node server
{ Hello: 'World!' }
$
```
> 加载模块的`dojo/node`和调用`node`使用的文件系统都需要路径。所以虽然我们的模块`./app-server/server.js`，我们还是相对于项目的根目录来使用路径。例外的是`define()`的相对路径，它和浏览器里一样，是相对于模块的路径。

> 基于Google V8  JavaScript引擎的Node.js是完全遵守ES5的，就是说你不用担心你代码的解码。例如，在上面，我们不像在浏览器里一样使用`dojo/json`模块来解析JSON字符串。不过如果你想要你的代码在老式浏览器上正常工作，你当然可以继续在代码中使用这些功能。

现在我们开始在Node.js下使用Dojo加载器来运行AMD模块了。但是除非我们以Dojo的方式来改变我们的编码风格，否则并不会从中获益。不只是在你的服务器端和客户端共享同样的代码，你还可以利用 Dojo Promises、 Dojo Requests、 Dojo i18n、 Dojo Topics 和Events 等其它丰富的AMD模块。

## Deferred and Promises
Node.js的一个强大特性是它的非阻塞性质。Node.js绝大多数的核心API是异步运行的。尽管如此，通过异步回调函数仍是默然方式。如果你用Dojo工作了一段时间之后，你就会了解Deferreds 和 Promises实现的异步功能的强大处理能力。还有一些其它框架为基于Node.js接口提供一个promise ，但是由于Dojo已经有一个promise框架，我们就用它了。
上面，我们用`fs.readFileSync` 来加载我们的文件并显示结果，但是这个方法同步运行并且会阻塞。为什么不来个异步的：

```
require([
    "dojo/node!fs"
], function(fs){
    fs.readFile("helloworld.json", function(err, data){
        if(err) console.error("I was robbed!");
        console.log(JSON.parse(data));
    });
});
```
它正常运行，不过我们用非常不Dojo的回调函数作为结束，再把它变成返回promise的：

```
require([
    "dojo/node!fs",
    "dojo/Deferred"
], function(fs, Deferred){

    function readFilePromise(filename){
        var dfd = new Deferred();
        fs.readFile(filename, function(err, data){
            if(err) dfd.reject(err);
            dfd.resolve(data);
        });
        return dfd.promise;
    }

    readFilePromise("helloworld.json").then(function(helloworld){
        console.log(JSON.parse(helloworld));
    }, function(err){
        console.error("I was robbed!");
    });
});
```
现在配合`readFilePromise`，我们可以使用promise模式来处理异步函数。这包括改变promise的能力，使代码更加可读，而如果你用Node.js异步模块你就只能“在回调里回调”。
实际上，你不会每次都创建一个函数来作为promise使用它。你可是创建一个模块来exposes 你想要返回promise的函数，然后加载这个模块来使用这个函数的promise版本。
> 我已经创建了一个包，叫做setten[这里写链接内容](https://github.com/kitsonk/setten)，它提供很多的“开箱即用”功能。它和Dojo Toolkit拥有一样的许可，而且全部是由Dojo基金会的CLA贡献的。希望它会随时间继续成长，提供更多的让Node.js下的工作更像Dojo的可用模块。特别的，一个名为`setten/util`的模块可以通过使用`util.asDeferred()`将异步回调函数转换成基于promise的。

##构建应用
现在我们有了一些基本的方式，可能你阅读本教程是想要在Node.js里创建一个客户端/服务器端组合应用，就是在服务器端和客户端都使用Dojo风格的代码。但是构建完整的应用程序超出了本教程的范围，我会试着给你一些建议和结构。
第一件要做的事就是为你已部署应用的客户端代码进行优化生成，这样你的服务器端可以提供一个建立层文件和串联的CSS。这就是为什么我建议你为应用程序的特定代码建立三个不同的包：

 - `src/app` - 在服务器端和客户端共享的模块。
 - `src/app-server` - 你的基础服务模块和其它值用在服务器端的模块。
 - `src/app-client` - 你的客户端模块。

你可以为`app`和`app-client`包创建一个Dojo生成配置文件，这样它们就可以用Dojo生成器来生成，你还可以创建一个整体应用的生成配置文件用来识别你的层，它们都将有你的服务器端提供。我上面提到的样本文件提供了这个框架，可以在构建过程中加入一个 `build.sh`和`build.bat`来生成。构建将会输出到`lib`路径。
另外一个方法是使用`[dojo-boilerplate](https://github.com/csnover/dojo-boilerplate)`项目提供的方法，它可以让客户端和服务器端代码在同一个模块里共存。这个方法里，`dojo/has`用来分离客户端和服务器端代码。例如，我们可以编写一个模块，它在客户端加载和在服务器端的node下加载时会做分别完全不同的事情：

```
define([
    "dojo/has", // Dojo's feature detection module
    "require"   // The "context" require, that is aware of the current context
], function(has, require){
    var app = {};

    if(has("host-browser")){
        // we are running under a browser
        require([ "dojo/request" ], function(request){
            request.get("/helloworld.json", {
                handleAs: "json"
            }).then(function(data){
                console.log(data);
            });
        });
    }

    if(has("host-node")){
        // we are running under node
        require([ "dojo/node!fs" ], function(fs){
            var data = fs.readFileSync("helloworld.json");
            console.log(JSON.parse(data));
        });
    }

    return app;
});
```
这些代码由Dojo生成器生成时，如果我们用Google Closure 来优化代码，会移除无效分支，就是说服务器端代码会从模块中移除。
就我个人而言，我更关注对同时用在服务器端和客户端的模块的分离，它可以更“干净”地生成，而不是完全依赖于代码优化来确保服务器端的代码不会分发给客户端。虽然这种替代模式是完全可行的。
> 代码生成的更多细节见 [Creating Builds](https://dojotoolkit.org/documentation/tutorials/1.10/build)教程。

## 小结
希望这个教程可以给你一个在Node.js下使用Dojo的总体印象。我个人认为在服务器端和客户端使用相同的编程语言和模式大有可为。在其他技术和语言都在试图模糊客户端和服务器端间的界限时，我认为我们开始看到这样的情况——我们的代码可以真正的实现同构，而不用再担心执行它的特定上下文。
这里有一些我或者其他人在Node.js下做的事情，或许能给你一些启发。我个人已经在NodeJS下用Dojo代替了所有的服务器端代码。

 - 构建“全栈”网站和全栈应用。（[My personal homepage](https://github.com/kitsonk/kitsonkelly.com)）
 - 文档服务器和文档生成工具。（[Dojoment](https://github.com/kitsonk/dojoment)）
 - 自动化测试。（[Teststack](https://github.com/csnover/dojo2-teststack)）
 - 服务器端widget渲染。（[Server Side Dijit](https://github.com/jthomas/server_side_dijit)）

在NodeJS下用Dojo可以做无数的事。祝你好运，看看你能想到什么有趣的事情可以做。

