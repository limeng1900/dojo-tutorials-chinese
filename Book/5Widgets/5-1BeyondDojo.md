# 5.1 超越Dojo

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/index.html

---

Dojo区别于其他JavaScript库的一点就是它的作用范围。你可以简单的使用Dojo基础功能、[DOM](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/)、[Ajax](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/)、[特效](https://dojotoolkit.org/documentation/tutorials/1.10/effects/)和其他常见功能，但其实Toolkit
提供的要多的多。在这篇教程中，我们将快速浏览Dojo Toolkit，介绍每个发行版本的一些其他装配组件。

##入门

目前为止，你可能只使用过Dojo的基本功能。你可能已经冒险用过Dojo核心的其他部分内容了，它们可能是包含在分发的`/dojo/`目录下的模块并且模块ID是以`dojo/*`开头的。如果你[下载](https://dojotoolkit.org/download/)了一个DOjo的SDK分发，或者查看一个来自SVN的版本，你会注意到`dojo`不是唯一的包：

![](/assets/dojoRelease.png)

不大可能说你已经走了这么远都没有嗅到Dojo中有点不同的部分，不过以防万一，我们再来回顾一遍基础知识。

##包和模块

Dojo里的包是严格的AMD包，它需要在根目录下包含一个`package.json`文件来描述这个包。Dojo源码分发中有三个包：`dojo`、`dijit`和`dojox`。在源码分发中的第四个目录直接就是`util`，它不是一个包，包含了实用程序来更容易地管理Dojo程序。

模块随后被聚合到包里。一个模块是一个JavaScript文件，它可以通过Dojo加载器（或者其他AMD加载器）加载到应用中。这些模块通常以一个`define()`函数作为开始。由于Dojo的可扩展性，你会在三个“root”包里找到许多的子目录。有时我们将它们作为“子包”（sub-packages）来引用，但严格来说他们不能仅靠自己就成为一个AMD包。一个模块的模块ID（MID）会直接映射到包内所占的路径。例如`/dojo/fx.js`模块有一个`dojo/fx`的MID。经常会有其他资源包含在包目录中，包括tests、CSS、语言包和图片。

##Base vs. Core
传统上（Dojo1.7之前）,曾经当你加载`dojo/dojo.js`文件到你的应用时，有一组功能是默认可用的。 这些功能被称为“base”Dojo。自Dojo1.7起，这些功能中`dojo/dojo.js`中打破并放进`dojo/_base`目录下的独立模块中。如果你在旧模式下（`async:false`）运行你的应用，那么当你加载`dojo/dojo.js`时这些模块将自动加载。如果你以异步的模式运行（`async:true`），那么这些模块将只在被直接require时加载，或者其他模块的加载依赖该模块时。

`dojo`包里其余的模块被称为Core。如果你已经读过其他一些教程，你可能已经用过Core的其他部分，比如[动画教程](https://dojotoolkit.org/documentation/tutorials/1.10/animation)中的`dojo/fx`。

Dojo Core 一系列广泛的功能，包括特征检测、deferreds 和promises、事件处理、数据存储、DOM操作、查询、DOM特效、窗口生命周期管理、鼠标、触摸和键盘事件、拖放、日期和国际化。

### 一个传统的示例
这个例子使用一些Dojo中更加“传统的”方式（可能还有其他JavaScript库）来实现一个DOM层面的查询和操作：

```
require([
    "dojo/query",
    "dojo/_base/array",
    "dojo/dom-construct",
    "dojo/domReady!"
], function(query, array, domConst){
    function topLinks(){
        var headings = query('h2,h3');

        array.forEach(headings, function(elm){
            var topLink = domConst.create("a", {
                href: "#top",
                innerHTML: "^top"
            });

            domConst.place(topLink, elm, "before");
        });
    }
});
```

查询DOM的片段使用一个CSS选择器同时找到`<h2>`和`<h3>`元素，并将它们中每一个作为引用节点插入一个新创建的`<a>`元素。还有一些关于[`dojo/query`](https://dojotoolkit.org/documentation/tutorials/1.10/using_query/)、[DOM functions](https://dojotoolkit.org/documentation/tutorials/1.10/dom_functions/)和[array helpers](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/)专门的教程。

### Dojo Core示例
一个简单的示例使用一个核心模块来创建一个函数比较两个数据并返回不同之处：

```
require([
    "dojo/date",
    "dojo/dom",
    "dojo/domReady!"
], require(date, dom){
    function daysSince(fromDate, target){

        if(!(fromDate instanceof Date)){
            fromDate = new Date(fromDate);
        }
        console.log("from date: ", fromDate);
        var now = new Date();
        console.log("From Date: " + fromDate.toUTCString());

        console.log("Difference in days: " +
            date.difference(fromDate, now, "day"));

        var days  = date.difference(fromDate, now, "day");

        dom.byId(target).innerHTML = days;
    }
});
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/demo/basecore.html)

在Dojo Base和Core为常见web和通用程序设计任务准备了丰富的功能。这些教程旨在向你介绍其中的一些，但是它替代不了[参考指南](https://dojotoolkit.org/reference-guide/1.10/dojo/)和[API参考](https://dojotoolkit.org/api/?qs=1.10)。

##Dijit：Forms、Layout和界面友好

`dijit`包是Dojo Core的一个兄弟包。它作为Dojo Toolkit的一个子程序管理和运行，它有着自己的拥有者、策略和指导方针。Dijit被设计为一组“企业级”、稳定、可达和国际化（包含了 right-to-left 和 left-to-right的支持）的视觉元素。它增强了像表单字段这样的原生HTML控件，提供先进的布局功能以及其他常用的UI元素接口。作为一组可视化UI元素，它也是用来定义自定义widget的一个框架。

Dijit提供了一系列控件来增强HTM表单控件的可用性、格式化和验证，还有使用dojo/store API的数据绑定。更多内容请看 [Dijit Themes、Buttons和Textboxes](https://dojotoolkit.org/documentation/tutorials/1.10/themes_buttons_textboxes/) 教程。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/demo/dijitForm.html)

Dijit还拥有适应视窗大小来创建动态用户界面和应对调和和用户交互的强力工具。它包括创建类似桌面应用程序布局的widget、像Tab和手风琴控件这样合理利用空间的标准项。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/demo/dijitLayout.html)

在创建交互时不是非要用Dijit，但是这些控件和模型包含了多年构建web应用整合和积累的经验，让它成为一个很棒的构建基础。

更多请阅读 [Layout with Dijit](https://dojotoolkit.org/documentation/tutorials/1.10/dijit_layout/) 。

##DojoX：Dojo扩展
`dojox`包是一个工具集常用和不常用扩展的子包集合。每一个子包是`dojox/`下的一个目录，提供它自己的`README`文件作为程序功能和状态的概述。其中一些代码是稳定且可生产化的，还有一些则不是。这里有DojoX中的一些亮点：

### DojoX：Data Grid
DojoX最常用的一个模块是`dojox/grid`。为了适应不断变化的网络浏览器以及不断广泛使用的移动浏览器，Dojo社区创建了新的网格：

 - [dgrid](http://dgrid.io/) - 由[SitePen](http://sitepen.com/)创建和维护，它提供为新一代浏览器和Dojo Store或dstore API设计的下一代网格组件。
 - [GridX](http://oria.github.com/gridx/) - 快速渲染、模块化和插件架构的网格组件。

### DojoX：GFX
一个跨平台的矢量图像API。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/demo/dojoxGfx.html)

更多内容请阅读[Vector Graphics with Dojo's GFX](https://dojotoolkit.org/documentation/tutorials/1.10/gfx/)教程。

### DojoX：Charting
构建于dojox/gfx的原生图表，为可视化数据提供广泛的图表类型、主体和特色。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/demo/dojoxCharting.html)

更多内容请阅读[Dojo Charting](https://dojotoolkit.org/documentation/tutorials/1.10/charting/)。

### DojoX：Mobile
为移动设备提供的一个轻量级、跨平台、移动widget和应用框架。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/beyond_dojo/demo/dojoxMobile.html)

更多内容请阅读[Getting Started with Dojox Mobile](https://dojotoolkit.org/documentation/tutorials/1.10/mobile/flickrview/part1/)。

还有些不上镜但是非常有用的：

**dojox/lang/functional**
一个函数式编程工具库，补充了Dojo Base的数组方法让你的代码更具表现性、紧凑且无副作用。

**dojox/widget | dojox/layout | dojox/form**
Dijit提供的更多可选控件。

DojoX的概述请看[DojoX Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojox/) 。

##Util
`util/`目录包含你项目打包、测试和文档的脚本及资源。Util不是一个Dojo命名空间，他的内容通常用在浏览器和你的页面之外。没有一个“完全正确”的方式来创建和部署web应用，但是多年来Dojo努力提供工具以促进一些重点的最佳实践。

每一个utile都 [documented in its own right](https://dojotoolkit.org/reference-guide/1.10/util/)，不过我们下面会做一个简短的介绍：

### Dojo Build System
Dojo 的包系统让我们可以模块化toolkit中的代码，并让文件系统布局追随模块结构。也就是说即使是一个简单的应用，你的浏览器也得为这些独立模块发出几十个HTTP请求。对于开发人员这是一个可以接受的折中之举，但是对于生产，你应该总是使用build工具来将这些独立模块文件整合带一个或多个“layers”。

build系统是一个运行在Rhino或NodeJS的JavaScript应用。更多请阅读 [build system itself](https://dojotoolkit.org/reference-guide/1.10/build/index.html)以及如何在[你自己的代码上使用](https://dojotoolkit.org/documentation/tutorials/1.10/build/)。

### Testing
[D.O.H.](https://dojotoolkit.org/reference-guide/1.10/util/doh.html)是Dojo用来做单元测试的测试框架。它包含一个浏览器内测试运行器和一个非浏览器、Rhino驱动的测试运行器。DOH支持一组简单的断言功能，并强烈支持异步测试，包含一个专门的Deferred实现，就是说它也可以用来测试非Dojo JavaScript。

 [DOH Robot](https://dojotoolkit.org/reference-guide/1.10/util/dohrobot.html)用来UI 测试，它是一个基于Java的web驱动器，可以记录和重演你页面的交互。你随后可以获取记录脚本并适当地插入断言来进行更加完整的测试。

为了更好的支持新一代web开发生态系统，SitePen在[2013年中](http://www.sitepen.com/blog/2013/05/01/intern-javascript-testing/)发布了一个新的测试框架—— [Intern](http://theintern.io/)。2014年Intern代替了DOH成为Dojo 的自测工具。更多Intern相关信息：

 - [Documentation: Intern wiki](https://github.com/theintern/intern/wiki)
 - [Tutorial](https://github.com/theintern/intern-tutorial)
 - [Examples](https://github.com/theintern/intern-examples)
 - [Help: Stack Overflow](http://stackoverflow.com/questions/tagged/intern)

##dstore：dojo/store的未来
新的[dstore](https://github.com/sitepen/dstore)包是dojo/store的继承者，适用于Dojo1.8+，而且是Dojo2计划内的API。如果你刚入门Dojo，我们推荐你看看dstore。

##小结
Dojo Toolkit不只是一个DOM和Ajax库。虽然它的特点和API在这块与其他库有所重叠，但是它拓展的广泛，在某些区域更加深入，提供了有用的抽象、widget和工具来帮助解决大型和小型、简单和复杂、常见和不常见的程序。

你不可能一次性了解全部——或者永远做不到——但是随着你的需求的增长，Dojo伴你成长。
