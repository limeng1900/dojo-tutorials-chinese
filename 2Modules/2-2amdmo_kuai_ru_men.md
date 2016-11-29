#2.2 AMD模块入门

Dojo支持模块以异步模块定义（AMD）的格式写入，这使得代码更容易编写和调试。在本教程中，我们将解释AMD的基本理解和使用。
如果你正从低于1.7的版本进行迁移，本教程的[1.8版本](https://dojotoolkit.org/documentation/tutorials/1.8/modules/)会比较有用，它提供一些从Dojo的旧模块系统向AMD迁移的指导。本教程只关注AMD。

## 概述
异步模块定义（AMD）格式是Dojo从1.7版本开始摘取的模块格式。它为旧的Dojo模块风格提供了许多增强，包括完全异步操作、真正的可移植性包、更好的依赖管理和对调试支持的改善。这也是一个社区驱动的标准，就是说写入模块的AMD规范可以使用任何其他的AMD-compliant加载器或库。在本教程中，我们将解释AMD并解释如何使用它。

## 模块是什么？
一个模块是一个可以由引用访问的值。如果你想在一个模块里展开多个数据或函数，它们必须是代表模块的单一对象的属性。实际上来说，它为一个简单的值多余的创建了一个模块，例如`var tinyModule = 'simple value';` ，但这是有效的。模块更多的意义在于将你的代码模块化，将它分解成处理特定功能的逻辑子集。如果你想用名字和地址这样的信息来待代表一个人，或者为你的人添加方法，把所有代码放在一个位置是很有意义的。在文件系统中，一个模块存储为一个单独文件。

## 如何创建一个模块
在AMD里，你通过向加载器注册来创建一个模块。
什么是加载器？加载器就是定义和加载模块背后控制逻辑的代码（是的，只用Javascript）。当你可以通过加载 `dojo.js` 或`require.js` 得到一个AMD加载器。加载器定义了交互函数，分别是require和define。
使用全局函数`define` 可以通过加载器注册一个模块，先看几个例子：
```
define(5);
```
不是很复杂但是有用，这个模块的值是数字5。

```
define({
    library: 'dojo',
    version: 1.10
});
```
来个更有意思点的，这个模块加载的时候，能够得到一个带2个属性的对象。

```
define(function(){
    var privateValue = 0;
    return {
        increment: function(){
            privateValue++;
        },

        decrement: function(){
            privateValue--;
        },

        getValue: function(){
            return privateValue;
        }
    };
});
```
这个例子里，向`define` 传递了一个函数。这个函数已经评定并且结果被加载器存储为一个模块。这段代码使用闭包创建了一个私有值，它不会被外部代码直接访问到，但是可以通过对象提供的方法进行检查和操作，这个对象作为模块的值返回。

## 如何加载模块？
首先，我们需要了解如何识别模块。为了加载模块，你需要一些识别它的方法。类似于其他编程语言的module/package系统，AMD模块通过它的路径和文件名进行识别。让我们把上面示例的代码保存在一个文件夹里：

```
app/counter.js
```
在加入一个加载器和index.html——应用接入点。文件结构如下：

```
/
    index.html
    /dojo/
    /app/
        counter.js
```
index 页面像这样：

```
<html>
    <body>
        <script src="dojo/dojo.js" data-dojo-config="async: true"></script>
        <script>
            require([
                "app/counter"
            ], function(counter){
                log(counter.getValue());
                counter.increment();
                log(counter.getValue());
                counter.decrement();
                log(counter.getValue());
            });
        </script>
    </body>
</html>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/modules/demo/demo.html)

下面回顾一下：

 1.  在`app/counter.js` 里，我们调用`define` 用加载器注册一个模块。注意我们定义的模块是一个对象的引用，不是一个构造函数，也就是说这个模块加载的每一部分代码都会引用相同的对象。总之，模块返回构造函数，但是在某些情况下他适当返回一个单例对象。
 2.  通过文件系统定位我们的模块，它在包含着`index.html`的文件夹的一个子文件夹里，这个子文件夹是AMD加载器（`dojo/dojo.js`）的兄弟文件夹。不需要我们额外配置，加载器知道module id“app/counter”意思是它应该加载`app/counter.js` 文件，并将它的返回值作为一个模块。
 3.  在`index.html` 里，我们调用`require` 来加载“app/counter”模块。你可以简单地通过`require(["app/counter"])` 加载模块。如果模块中的代码有其他副作用（比如扩增其他模块），你完全不需要再引用这些模块。不过，如果你需要一个模块的引用，你需要提供一个回调函数。加载器会确保模块已经被加载，一旦加载完成，它会将传递给它的任意模块作为参数调用回调函数。与任何其他函数一样，你可以自由地为参数命名，没有要求参数名要与模块名有关系。即便如此，使用和模块名相似的命名仍是一种比较好的做法。

## 模块加载模块
到目前为止，我们的例子只是展示了`define`函数的简单运用。当一些结构清晰的模块组合成应用，这些模块之间自然有很多依赖关系。`define` 函数可以自动的为你的模块加载依赖关系模块。在模块生成之前要将依赖关系列表传递给 `define` .

```
define([
    "dojo/_base/declare",
    "dojo/dom",
    "app/dateFormatter"
], function(declare, dom, dateFormatter){
    return declare(null, {
        showDate: function(id, date){
            dom.byId(id).innerHTML = dateFormatter.format(date);
        }
    });
});
```
这个例子展示了AMD应用的一些特点：

 1.  多重依赖—— 依赖关系列表里同时指定了"dojo/dom" 和 (假设的) "app/dateFormatter" 。
 2.  返回一个构造器——这个模块可以叫做"app/DateManager"这样的名字。使用它的代码如下：

```
require([
    "app/DateManager"
], function(DateManager){
    var dm = new DateManager();
    dm.showDate('dateElementId', new Date());
});
```
在你用Dojo开发之前首先要熟悉的主题之一是AMD，还有一个重要功能是` declare `，如果你还没有熟悉 `dojo/_base/declare` ，接下来可以先看看它的[教程](https://dojotoolkit.org/documentation/tutorials/1.10/declare/)。

## 使用插件
除了常规模块，AMD加载器还有一种称为插件的新模块。插件通过其超越简单AMD模块的功能对加载器进行扩展。插件的加载或多或少与常规模块一样，不过在模块标识符的末尾使用一个特殊符号“!”来标识它是一个插件请求。“！”之后的数据会直接传递给插件来处理。通过几个例子来学习会比较清楚点。Dojo提供几个默认插件，最重要的四个是` dojo/text` 、`dojo/i18n`、`dojo/has`、和`dojo/domReady` 。让我们来看看如何使用它们。

### [dojo/text](https://dojotoolkit.org/reference-guide/1.10/dojo/text.html)
当你需要中文件（比如HTML模板）加载字符串的时候可以使用`dojo/text`。 获取的值会被缓存起来，这样后面再调用同一文件的时候就不会再额外进行请求。生成器使用`dojo/text`加载内联字符串。例如为一个模板化的widget加载一个模板，你要像这样定义你的模块：

```
// in "my/widget/NavBar.js"
define([
    "dojo/_base/declare",
    "dijit/_WidgetBase",
    "dijit/_TemplatedMixin",
    "dojo/text!./templates/NavBar.html"
], function(declare, _WidgetBase, _TemplatedMixin, template){
    return declare([_WidgetBase, _TemplatedMixin], {
        // template contains the content of the file "my/widget/templates/NavBar.html"
        templateString: template
    });
});

```
### [dojo/i18n](https://dojotoolkit.org/reference-guide/1.10/dojo/i18n.html)
`dojo/i18n` 根据web浏览器的用户语言环境加载语言包。示例：

```
// in "my/widget/Dialog.js"
define([
    "dojo/_base/declare",
    "dijit/Dialog",
    "dojo/i18n!./nls/common"
], function(declare, Dialog, i18n){
    return declare(Dialog, {
        title: i18n.dialogTitle
    });
});
```
如何使用 `i18n` 的更多信息见 [internationalization tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/i18n/)。

### [dojo/has](https://dojotoolkit.org/reference-guide/1.10/dojo/has.html)
Dojo加载器包含一个has.js实现的API特征检测器；`dojo/has` 插件利用此功能有条件的加载模块。像下面这么用：

```
// in "my/events.js"
define([
    "dojo/dom",
    "dojo/has!dom-addeventlistener?./events/w3c:./events/ie"
], function(dom, events){
    // events is "my/events/w3c" if the "dom-addeventlistener" test was true, "my/events/ie" otherwise
    events.addEvent(dom.byId("foo"), "click", function(){
        console.log("Foo clicked!");
    });
});

```
### [dojo/domReady](https://dojotoolkit.org/reference-guide/1.10/dojo/domReady.html)
dojo/domReady用来替代` dojo.ready`。它是直到DOM准备好才运行的模块。像下面这么用：

```
// in "my/app.js"
define(["dojo/dom", "dojo/domReady!"], function(dom){
    // This function does not execute until the DOM is ready
    dom.byId("someElement");
});
```
注意在回调函数里没有定义参数来返回dojo/domReady的值。因为返回值没有什么作用，我们只是用它来延迟回调。由于模块的本地变量名称取决于模块之间的顺序，不使用返回值的加载模块或插件应放在require依赖关系列表的末尾。
即使没有数据传递给插件，感叹号还是必须的。没有它，你就只会将dojo/domReady模块作为一个依赖加载，而不是激活它的特殊插件功能。

## 小结
本教程提供的对AMD的基本理解将带你入门Dojo开发，但你很快会发现自己陷入了更复杂的情况。阅读[Advanced AMD Usage](https://dojotoolkit.org/documentation/tutorials/1.10/modules_advanced/) 教程来了解更多：

 - 加载器和库位于不同的位置甚至服务时如何配置加载器
 - 创建便携式模块的包
 - 加载同一模块或库的多个版本
 - 加载非AMD代码

## 资源

 -  [AMD规范](https://github.com/amdjs/amdjs-api/wiki/AMD)