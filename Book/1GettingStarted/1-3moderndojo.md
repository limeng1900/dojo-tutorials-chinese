# 1.4 新一代Dojo

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/modern_dojo/index.html

你或许已经很久没用Dojo了，或者你在1.10版本下用你1.6版的应用挣扎了很久却不知到底怎么回事。你可能经常听说AMD和baseless，但你不知道怎么做或者从哪开始。那么这个教程将会有所帮助。  

## 入门

Dojo1.7是Dojo Toolkit向更加现代的架构的一个重大转变，Dojo1.10在这条道路上更进了一步。虽然它广泛地向后兼容，但是为了充分发挥Dojo1.10的优势，一些基本概念还是发生了变化。这些概念将作为Dojo2.0的基础，采用这些新概念将帮你在正确的路上走得更远。当然为了从这些新的要素（如 `dojo/on`）中直接获益，你需要接受其中一些新概念。

本教程将解释一些Dojo中引入的概念，主要侧重于新的Dojo。我会尽力阐明为什么作出改变以及是如何改变的。一些变化是从根本上进行的，初看会很迷惑，但它们都有着充分的理由——让你的代码更高效、运行更快、玩转Javascript、更棒的可维护性。总之，花时间去理解新一代Dojo是非常值得的。

本教程不是一个手把手的版本迁移指导，如果你已经很熟悉Dojo，它对你来说远胜于一个初级概念读本。更多的技术细节请参考 [Dojo 1.X to 2.0 Migration Guide](https://dojotoolkit.org/reference-guide/1.10/releasenotes/migration-2.0.html) 。

## Hello新世界

新一代Dojo的一个核心理念就是全局命名空间是件坏事。这有很多原因，在一个复杂的web应用中，全局命名空间很容易被各种代码污染，特别是当许多组织使用多重Javascript框架时。我甚至不用从安全的角度去提那些故意修改全局命名空间产生的恶果。如果你打算在新一代Dojo中的全局命名空间里访问一些东西，请剁手。由于向后兼容的原因大量的toolkit暂时是全局范围的，但请不要用在新的开发中。

>当你发现你在输入 `dojo.*` 或 `dijit.*` 或 `dojox.*` ，你正步入歧途。 

就是说，那些只是引入 `dojo.js` 、获取一个核心功能、引用几个模块，然后在你的核心内容中输入`dojo.something` 的开发者们，你们真的要改改了，因为这么干真的很糟。

再来一次，跟着我念“全局命名空间真烂， 全局命名空间真烂 ，我才不用全局命名空间， 我才不用全局命名空间 ”。

另一个核心理念是同步作业很慢，而异步通常更快。旧Dojo已经从dojo.Deferred获得了异步JavaScript强大血脉，但是对于“新一代”Dojo，最好都一切都异步操作。

为了加强Dojo的模块化并利用上述理念，在Dojo1.7中采用被称为异步模块定义（AMD）的CommonJS模块定义。就是说Dojo模块加载器从根本上的重写通常都离不开 `require()` 和`define()` 函数。完整文档在这里 [loader in the reference guide](https://dojotoolkit.org/reference-guide/1.10/loader) 。这从根本上改变了代码结构。

先举个以旧方式做的例子：

```
 dojo.ready(function(){
    dojo.byId("helloworld").innerHTML = "Hello World!";
  });
```

然后我们看看新一代版本：

```
 require(["dojo/dom", "dojo/domReady!"], function(dom){
    dom.byId("helloworld").innerHTML = "Hello New World!";
  });
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/modern_dojo/demo/modern_dojo-helloworld.html)

欢迎来到美丽新世界。 `require()` 是新一代Dojo的基础。它创建Javascript闭包提供给需要的模块使用，就像通过参数将变量传递给函数一样。通常，第一个参数是一个模块ID组成的数组，第二个是一个函数。在 `require()` 的闭包里，我们可以通过在参数从声明的变量来引用这些模块。在调用模块时，通常会有一些惯例，一般在参考指南中会说明。

加载器——正如旧的那种，会负责查找、加载、和管理模块的所有累活。

你可能会发现在require的数组中有一个名为`dojo/domReady!`的模块没有返回变量，它是一种加载器插件，用来控制加载器的行为。本模块的作用是让加载器等待DOM结构加载完成。在异步的世界里，当你想要操作页面DOM结构时假设它存在可不是个好主意，所以如果你要在代码中对DOM做点什么的话，先确定你包含了这个插件。因为我们在代码中并不使用这个插件，惯例是把它放在数组的最后，且不提供它的返回变量。

> 现在这里只有`dojo/domReady!`插件，但是可能并不足以让你的代码正常工作（因为还有其他的事情，比如一些a11y特征检测，可能无法运行）。所以，当你使用`dojo/parser`和Dijit这些时，使用`dojo/ready`比较明智。

在模块已经加载后，你可以只提供MID作为一个字符串参数来使用 require() 引用该模块。它不会加载模块，如果模块还没有加载就会抛出一个错误。你会在Dojo Toolkit里看到这种编码风格，因为我们希望能够在代码中集中管理依赖关系。该风格编码示例如下：

```
  require(["dojo/dom"], function(){
    // 一些代码
    var dom = require("dojo/dom");
    // 另一些代码
  });
```
> AMD的另外一个核心功能是define()，用来定义模块。详细教程看 [Defining Modules](https://dojotoolkit.org/documentation/tutorials/1.10/modules/) 。

## Dojo Base和Core

你在用“新一代”Dojo的时候可能听过 "baseless" 这个术语，就是确保一个模块它需要的基本Dojo功能之外，不会依赖其它更多的东西。在旧世界里，仍然有大量的函数放在基础`dojo.js`里，而且至少在2.0之前它们依然存在。不过要是你希望确保你的代码将来如愿地易于迁移，那就别用`dojo.*`。就是说你可能并不了解现在的一部分命名空间在哪。

`dojoConfig` 有一个选项是 `async` ，它默认为`false`，就是所有的Dojo基础模块都会自动加载。如果你设为`true`并利用加载器的异步性质，这些模块就不会自动加载。这样的组合是为了应用更快的响应和加载。

此外，Dojo 遵循EcmaScript 5规范， 在可能的情况下，不赞成模仿ES5功能的部分Dojo，只是为了将ES5的功能带给旧一代浏览器时使用它才合适。就是说以Dojo的方式处理问题在某些情况下是指完全不使用Dojo。

虽然更新的参考指南会告诉你函数的位置，不过还有一个特定的文档来标明[基础函数](https://dojotoolkit.org/reference-guide/1.10/releasenotes/migration-2.0.html#basic-functions)的位置。

你一旦跳出了 Dojo Base 和 Core， 几乎一切都会像下面这样。以前你这么用`dojo.require()`：

```
  dojo.require("dojo.string");

  dojo.byId("someNode").innerHTML = dojo.string.trim("  I Like Trim Strings ");
```

现在用require()则是这样：

```
require(["dojo/dom", "dojo/string", "dojo/domReady!"], function(dom, string){
    dom.byId("someNode").innerHTML = string.trim("  I Like Trim Strings ");
  });
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/modern_dojo/demo/modern_dojo-string.html)

## Events and Advice

`dojo.connect()` 和 `dojo.disconnect()` 被移入到 `dojo/_base/connect` 模块，新Dojo使用 `dojo/on` 来进行事件处理， `dojo/aspect` 则针对方法advice。 [Events](https://dojotoolkit.org/documentation/tutorials/1.10/events/) 有更深层次的教程，这里将涉及一些差异。

在旧Dojo中，事件和修正方法行为之间没有明确的区别，都用的 `dojo.connect()` 。事件是事情发生在于对象之间的关系，例如一个点击事件。 `dojo/on` 完美处理DOM原生时间以及Dojo对象或小部件引发的事件。 advice 是源于面向方面编程（AOP）的概念外加连接点或方法的行为。Dojo的很多部分都符合AOP，

`dojo/aspect` 则为此提供了一个集中机制。

在旧Dojo中，我们有多种方式来完成一个点击事件的处理：

```
<script>
    dojo.require("dijit.form.Button");

    myOnClick = function(evt){
      console.log("I was clicked");
    };

    dojo.connect(dojo.byId("button3"), "onclick", myOnClick);
  </script>
  <body>
    <div>
      <button id="button1" type="button" onclick="myOnClick">Button1</button>
      <button id="button2" data-dojo-type="dijit.form.Button" type="button"
        data-dojo-props="onClick: myOnClick">Button2</button>
      <button id="button3" type="button">Button3</button>
      <button id="button4" data-dojo-type="dijit.form.Button" type="button">
        <span>Button4</span>
        <script type="dojo/connect" data-dojo-event="onClick">
          console.log("I was clicked");
        </script>
    </div>
  </body>
```

在新Dojo中只使用 `dojo/on`, 你可以用编程式或声明式的来编写你的代码，不用管你处理的是DOM事件还是 Dijit\/widget ：

```
 <script>
    require([
        "dojo/dom",
        "dojo/on",
        "dojo/parser",
        "dijit/registry",
        "dijit/form/Button",
        "dojo/domReady!"
    ], function(dom, on, parser, registry){
        var myClick = function(evt){
            console.log("I was clicked");
        };

        parser.parse();

        on(dom.byId("button1"), "click", myClick);
        on(registry.byId("button2"), "click", myClick);
    });
  </script>
  <body>
    <div>
      <button id="button1" type="button">Button1</button>
      <button id="button2" data-dojo-type="dijit/form/Button" type="button">Button2</button>
      <button id="button3" data-dojo-type="dijit/form/Button" type="button">
        <div>Button4</div>
        <script type="dojo/on" data-dojo-event="click">
          console.log("I was clicked");
        </script>
      </button>
    </div>
  </body>
```

> 注意没有用 `dijit.byId` ，在新Dojo中widgets使用 `dijit/registry` ， `registry.byId()` 用来取得widget的引用。另外，注意 `dojo/on` 处理DOM节点和widget事件的方式是一样的。

旧方式给方法添加功能时你可能要这么做：

```
  var callback = function(){
    // ...
  };
  var handle = dojo.connect(myInstance, "execute", callback);
  // ...
  dojo.disconnect(handle);
```

新Dojo中， dojo\/aspect 可以让你获取一个方法的advice并且在另一方法之前、之后或周围添加行为。比如，你可用 `aspect.after()` 代替 `dojo.connect()` ：
```
  require(["dojo/aspect"], function(aspect){
    var callback = function(){
      // ...
    };
    var handle = aspect.after(myInstance, "execute", callback);
    // ...
    handle.remove();
  });
```

## 要点

Dojo中另一块经过小修正的是 publish\/subscribe 功能，它在 `dojo/topic` 模块下进行了模块化和改进。

旧Dojo中这么做：

```
  // To publish a topic
  dojo.publish("some/topic", [1, 2, 3]);

  // To subscribe to a topic
  var handle = dojo.subscribe("some/topic", context, callback);

  // And to unsubscribe from a topic
  dojo.unsubscribe(handle);
```

新Dojo中，利用 `dojo/topic` ：
```
  require(["dojo/topic"], function(topic){
    // To publish a topic
    topic.publish("some/topic", 1, 2, 3);

    // To subscribe to a topic
    var handle = topic.subscribe("some/topic", function(arg1, arg2, arg3){
      // ...
    });

    // To unsubscribe from a topic
    handle.remove();
  });
```
> 从 `dojo/topic` 指南去获取更多细节。

>  注意publish参数不再是一个数组，而只是简单的传递。

## Promises

Dojo 的一个核心概念是Deffrred类，改变了Dojo1.5中设定的架构，这里需要讨论下。另外，在1.8和更新的版本中，promise API进行了重写。它大部分只是在语义上和之前一样，但不在支持旧的API，如果你要使用它，就需要采用新API。在就Dojo中可以看到Deferred如此工作：

```
function createMyDeferred(){
    var myDeferred = new dojo.Deferred();
    setTimeout(function(){
      myDeferred.callback({ success: true });
    }, 1000);
    return myDeferred;
  }

  var deferred = createMyDeferred();
  deferred.addCallback(function(data){
    console.log("Success: " + data);
  });
  deferred.addErrback(function(err){
    console.log("Error: " + err);
  });
```

新Dojo这么做：
```
require(["dojo/Deferred"], function(Deferred){
    function createMyDeferred(){
      var myDeferred = new Deferred();
      setTimeout(function(){
        myDeferred.resolve({ success: true });
      }, 1000);
      return myDeferred;
    }

    var deferred = createMyDeferred();
    deferred.then(function(data){
      console.log("Success: " + data);
    }, function(err){
      console.log("Error: " + err);
    });
  });
```

> dojo/DeferredList 依然还在，只是不赞成再用。你能在 `dojo/promise/all` 和`dojo/promise/first`. 找到更健壮和相似的功能。

## Requests

任何一个Javascript库都将Ajax作为其核心基础之一。Dojo1.8之后，这一基本构建API更加优秀——跨平台运行，易于拓展，提高代码重用。以前，你常常为了获取外来数据只能在XHR、 Script 和 IFrame IO communication 之间挣扎。 `dojo/request` 可以帮你让整个过程更加容易。

就像 `dojo/promise` 一样，就的实现方法依然保留，但是你可以很容易的利用新的方法重构你的代码。例如，旧Dojo中你可能这么做：

```
  dojo.xhrGet({
    url: "something.json",
    handleAs: "json",
    load: function(response){
      console.log("response:", response);
    },
    error: function(err){
      console.log("error:", err);
    }
  });
```

新Dojo你要这么做：

```
 require(["dojo/request"], function(request){
    request.get("something.json", {
      handleAs: "json"
    }).then(function(response){
      console.log("response:", response);
    }, function(err){
      console.log("error:", err);
    });
  });
```

> `dojo/request` 会加载对于你的平台最合适的请求处理器，比如浏览器的XHR。上面的代码可以在NodeJS上轻易的运行，你不需要做任何修改。

这也是一个很大的话题，详细参见 [Ajax with dojo\/request](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/) 。

## DOM操作

到现在你可能已经发现了这样的趋势，Dojo不仅舍弃了对全局命名空间的依赖，采用了一些新模式，而且还打破了一些模块的核心功能，这点在 JavaScript toolkit 比DOM操作上做的更多。

模块摘要和内容（略）。

贯穿新Dojo doolkit的的一个模式就是围绕访问器的逻辑分离。下面这些已经被替代了：

```
 var node = dojo.byId("someNode");

  // Retrieves the value of the "value" DOM attribute
  var value = dojo.attr(node, "value");

  // Sets the value of the "value" DOM attribute
  dojo.attr(node, "value", "something");
```

下面的示例用使用完全不同的事情实现的同样的功能：

```
  require(["dojo/dom", "dojo/dom-attr"], function(dom, domAttr){
    var node = dom.byId("someNode");

    // Retrieves the value of the "value" DOM attribute
    var value = domAttr.get(node, "value");

    // Sets the value of the "value" DOM attribute
    domAttr.set(node, "value", "something");
  });
```

在新Dojo中，你在代码里做什么事很清晰的，很难由于参数的多余或缺失在你的代码里出现你不打算做的事。访问器的分离始终贯穿新Dojo。

## DataStores 与  Stores

在Dojo1.6中，引入了新的 `dojo/store` API，丢弃了 `dojo/data` API。 `dojo/data` 数据存储至少会维持到Dojo2.0，可能的话还是迁移到新API更好。本教程不多赘述，详情参见 [Dojo Object Store](https://dojotoolkit.org/documentation/tutorials/1.10/intro_dojo_store/) 。

## Dijit 和 Widgets

Dijit 也在新Dojo作出了改变，不过随着功能被打破到离散的构建模块在组合出复杂的功能，多数的改变放在了toolkit的基础上。如果要创建自定义widget，参见 [Creating a custom widget](https://dojotoolkit.org/documentation/tutorials/1.10/recipes/custom_widget) 。

如果只是用dijit或其他widget开发，那么在 `dojo/Stateful` 和`dojo/Evented` 类中引入了一些核心概念。

`dojo/Stateful` 提供widget的离散访问器，这些属性的改变可以看见。例如：

```
  require(["dijit/form/Button", "dojo/domReady!"], function(Button){
    var button = new Button({
      label: "A label"
    }, "someNode");

    // Sets up a watch on button.label
    var handle = button.watch("label", function(attr, oldValue, newValue){
      console.log("button." + attr + " changed from '" + oldValue + "' to '" + newValue + "'");
    });

    // Gets the current label
    var label = button.get("label");
    console.log("button's current label: " + label);

    // This changes the value and should call the watch
    button.set("label", "A different label");

    // This will stop watching button.label
    handle.unwatch();

    button.set("label", "Even more different");
  });
```

`dojo/Evented` 为类提供 `emit()` 和`on()` 函数，这是Dijit和widget的一步更。尤其是，使用 `widget.on()` 来设置你的事件处理更加先进。例如：

```
 require(["dijit/form/Button", "dojo/domReady!"], function(Button){
    var button = new Button({
      label: "Click Me!"
    }, "someNode");

    // Sets the event handling for the button
    button.on("click", function(e){
      console.log("I was clicked!", e);
    });
  });
```

## Parser

最后是关于 `dojo/parser` 。Dojo同时加强了编程式和声明式标记方式， `dojo/parser` 对声明式标记进行解析，并转化成对象和widget的实例。之前提到的所有新思想同样影响着 `dojo/parser` ，并让它发生了一些新变化。
虽然Dojo依然支持 `parseOnLoad: true` 配置， 但显式地调用解析器通常更有意义 。例如：

```
    require(["dojo/parser", "dojo/domReady!"], function(parser){
        parser.parse();
    });
```
解析器的另一大变化就是支持了HTML5的属性标记节点。你可以在HTML5中标记HTML。特别是 `dojoType` 变成了 `data-dojo-type` ，从而用指定对象参数替代无效的 HTML\/XHTML 属性， `data-dojo-props` 属性指定的所有参数都将传递给对象构造器。例如：

```
  <button data-dojo-type="dijit/form/Button" tabIndex=2
      data-dojo-props="iconClass: 'checkmark'">OK</button>
```

> Dojo 支持在 `data-dojo-type` 里使用Module ID，例如 `dojoType="dijit.form.Button"` 变成`data-dojo-type="dijit/form/Button"`。

上面提到的引入 `dojo/Evented` and `dojo/Stateful` 的概念变化在 "watch" 和"on" 的功能上已经赶上了尚明脚本和添加合适的脚本类型。例如，现在你可以这么做：

```
  <button data-dojo-type="dijit/form/Button" type="button">
    <span>Click</span>
    <script type="dojo/on" data-dojo-event="click" data-dojo-args="e">
      console.log("I was clicked!", e);
      this.set("label", "Clicked!");
    </script>
    <script type="dojo/watch" data-dojo-prop="label" data-dojo-args="prop, oldValue, newValue">
      console.log("button: " + prop + " changed from '" + oldValue + "' to '" + newValue + "'");
    </script>
  </button>
```

另外，解析器也支持 `dojo/aspect` 引入的概念，你可以向 "before"、"after" 和"around" advice加入代码。更多见 [dojo\/parser](https://dojotoolkit.org/reference-guide/1.10/dojo/parser.html#changing-the-behavior-of-a-method) 。

> `dojo/parser` 也支持模块的 auto-requiring 。就是说你没必要在模块调用前引入它。不过如果把 `isDebug` 设为true，你这么引入模块会出现警告。

## Builder

最后再简单提下 Dojo builder，它在 Dojo 1.7 里完全重写了。一方面是随着AMD的显著变化，同时它也被设计的更加现代化和富于特色。这里不多说，详见 [Creating Builds](https://dojotoolkit.org/documentation/tutorials/1.10/build/) ，不过为了拥抱新的 builder 请忘记你对旧builder 的一切印象。

## 小结

希望你经历了一场有意思的新Dojo旅程。任何熟悉旧款的人在新世界里重新开始思考都需要花点时间，一旦你开动了，就很难再回去，而且你会发现对于应用有了更加结构化的方法。

总之，记得新Dojo的方式是：

* **粒度相关和模块化**——只require你需要的。为了更快、更智能、更安全的应用。

* **异步**——事情不是必须按顺序发生的，为代码的异步操作做好计划。

* **全局作用域很糟**——再来一次，我再也不用 全局作用域 了。

* **离散访问器**——一个函数只做一件事，尤其是对于访问器。为你准备了 `get()` 和 `set()` 。

* **Dojo补充 ES5 **—— EcmaScript 5 做的，Dojo不做。

* **Events and Advice,not Connections**——Dojo正从通用连接转向时间和面向方面编程。

* **Builder 大有不同了**——变得更加强大和富有特色，但是它只会更加彰显旧应用的设计缺陷，而不是修复它们。

祝好运。












