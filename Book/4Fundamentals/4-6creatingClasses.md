#4.6 创建类
 原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/declarative/index.html 

---


`dojo/_base/declare`模块是Dojo Toolkit里类创建的基础。`declare`允许多重继承来让开发者编写灵活的代码并避免重复同样的代码。Dojo、 Dijit 和 Dojox 模块都使用`declare`；在本教程中，你将了解为什么你也要用它。

##入门
确保你了解[模块教程](https://dojotoolkit.org/documentation/tutorials/1.10/modules)中提出的概念。

##Dojo的基础类创建
`declare`函数定义在`dojo/_base/declare`模块里。`declare`接收三个参数：`className`、`superClass`和`properties`。

### ClassName
