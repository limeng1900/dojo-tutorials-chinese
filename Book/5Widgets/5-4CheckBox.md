# 5.4 复选框

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/index.html

----------

Dijit的表单widgets的集合提供一个方便和灵活的一系列选项来创建丰富的表单。在本篇教程中，我们将着眼于复选框式交互的选项。

##入门
复选框和单选按钮是用户交互中输入和选择的主要选项。Dijit包含`dijit/form/CheckBox`和`dijit/form/RadioButton`widget模块，可以用来嵌入替换原生的复选框和单选按钮元素。这些widget都是主体化的，可以在创建构建表单时创建一致的外观，并为输入值和状态的管理提供一系列便利的方法。

Dijit的表单widget的驱动原理是在保留现有的语义和使用模式的前提下提高原生控制。因此，你可以预期CheckBox 和RadioButton widget支持你使用过的原生复选框和单选按钮类型相同的全部功能。

### 定义一个CheckBox
和全部的Dijit widget一样，`dijit/form/CheckBox`可以使用标记（声明式）或者代码（编程式）进行实例化。以下两个例子创建一个初始化选中的CheckBox：

```
<input type="checkbox" id="dbox1" checked
    data-dojo-type="dijit/form/CheckBox">
<label for="dbox1">Want</label>
```

```
require(["dijit/form/CheckBox"], function(CheckBox) {
    var box1 = new CheckBox({
        id: "pbox1",
        checked: true
    });

    // place the widget on the page
    box1.placeAt("pbox1_container", "first");
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/demo/CheckBox.html)

点击标签和输入框都可以让它在已选和未选之间切换。你也可以使用tab键来通过键盘控制导航，用空格键控制选中/未选元素，就和原生控制一样。通过引用相应的CheckBox widget的ID，HTML标签也能和原生表单一样工作。

Dojo1.6引入的一个新特性是使用HTML5 `data-dojo-type`属性来代替`dojoType`属性。在Dojo1.6中，`data-dojo-type`会阻止Dojo从标准HTML属性中获取值，也就是说如果你需要使用该属性，你就要将正常HTML属性值（`checked`，`disabled`等）复制到 `data-dojo-type`中。这个问题在Dojo1.7中得到了解决。

##复选框的值

所有的Dijit表单widget都有存取器方法来获得和更新widget的值（`<var>widget</var>.get("value")` 和 `<var>widget</var>.set("value")`）。记着对于原生复选框，一个复选框的值如果是checked只能发送给服务器。Dijit的CheckBox相似：如果它是选中状态，`<var>widget</var>.get("value")`将返回widget的值属性。