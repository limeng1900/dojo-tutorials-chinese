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
layer本质上是一个单独JavaScript文件，它一般会包含几个模块，有时也包含其它资源。build的主要输出通常是一个layer，创建这个文件然后让你应用变得可分配。一个layer可以是一个引导layer，它包含Dojo的引导代码可以让Dojo加载其它模块。你需要什么layer以及它们的内容都由你的应用和设计来决定。这不一定有一个“正确”的方式。
