# 5.4 复选框

原文地址：[https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/index.html](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/index.html)

---

Dijit的表单widgets的集合提供一个方便和灵活的一系列选项来创建丰富的表单。在本篇教程中，我们将着眼于复选框式交互的选项。

## 入门

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

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/demo/CheckBox.html)

点击标签和输入框都可以让它在已选和未选之间切换。你也可以使用tab键来通过键盘控制导航，用空格键控制选中/未选元素，就和原生控制一样。通过引用相应的CheckBox widget的ID，HTML标签也能和原生表单一样工作。

Dojo1.6引入的一个新特性是使用HTML5 `data-dojo-type`属性来代替`dojoType`属性。在Dojo1.6中，`data-dojo-type`会阻止Dojo从标准HTML属性中获取值，也就是说如果你需要使用该属性，你就要将正常HTML属性值（`checked`，`disabled`等）复制到 `data-dojo-type`中。这个问题在Dojo1.7中得到了解决。

## 复选框的值

所有的Dijit表单widget都有存取器方法来获得和更新widget的值（`<var>widget</var>.get("value")` 和 `<var>widget</var>.set("value")`）。记着对于原生复选框，一个复选框的值如果是checked只能发送给服务器。Dijit的CheckBox相似：如果它是选中状态，`<var>widget</var>.get("value")`将返回widget的值属性。否则，它将返回`false`。如果没有提供或设置值属性，`dijit/form/CheckBox`有一个默认为“on”的值。我们可以通过请求这个值来看是否得到一个布尔值`false`来推断选中状态，或者我们可以检查`checked`属性：

```
require(["dijit/registry"], function(registry){

    var toppings = [];
    if(registry.byId("topping1").get("checked")){
        toppings.push(registry.byId("topping1").get("value"));
    }

    if(registry.byId("topping2").get("value") !== false){
        toppings.push(registry.byId("topping2").get("value"));
    }
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/demo/checkboxValues.html)

这个示例也连接了另外一个复选框，来修改另一个的值，作为`value`属性设置的例子：

```
registry.byId("deluxe").on("change", function(isChecked){
    registry.byId("topping2").set("value", isChecked ? "kalamata olives" : "olives");
}, true);
```

当你向`dijit/form/CheckBox`传递一个真（例如：非空、非零）`value`，widget会自动转为选中状态。

##单选按钮

目前我们在前文中讨论的关于复选框和`dihit/form/CheckBox`，对于Dijit的单选按钮widget——`dijit/form/RadioButton`也适用。单选按钮和复选框的不同之处在于：

 - 通常会渲染成一个圆盘，用在里面的圆点代表它的选中状态
 - 用在单一选择上，同一时间在一系列选项中只能有一个被选中。

除此之外，它的适用和复选框很相似。让我们看个例子：

```
<ul>
    <li>
        <input id="topping1" type="radio" name="topping" value="anchovies" checked
            data-dojo-type="dijit/form/RadioButton">
        <label for="topping1">Anchovies</label>
    </li>
    <li>
        <input id="topping2" type="radio" name="topping" value="olives"
            data-dojo-type="dijit/form/RadioButton">
        <label for="topping2">Olives</label>
    </li>
    <li>
        <input id="topping3" type="radio" name="topping" value="pineapple"
            data-dojo-type="dijit/form/RadioButton">
        <label for="topping3">Pineapple</label>
    </li>
</ul>
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/demo/radioButtons.html)

这里关键的不同是使用`name`属性。使用原生的HTML单选控制，你可以同过共享一个`name`属性来联合一组Dijit单选按钮。现在当你选中列表中的一个选项时，其他的会自动取消选中。

将一系列单选按钮作为一个单独控制器并获取选中的值是很常见的。 [dojox/form/CheckedMultiSelect](https://dojotoolkit.org/reference-guide/1.10/dojox/form/CheckedMultiSelect.html) 提供了这个功能。

##事件

因此，我们可以用这些看起来很棒的控制器来进行创建和交互。还能做些什么呢？像Dijit提供的许多widget一样，单选按钮和复选框也提供一些方法，让你在活动发生时得到通知。这些事件的完整列表可以在[API docs](https://dojotoolkit.org/api/?qs=1.10/dijit/form/CheckBox)找到。下一个示例中，我们将重点关注你可能最常用到的：`change`。

```
registry.byId("topping1").on("change", function(isChecked){
    if(isChecked){
        summaryNode.innerHTML = "Likes the salty!";
    }
}, true);

registry.byId("topping2").on("change", function(isChecked){
    if(isChecked){
        summaryNode.innerHTML = "Likes the sweet!";
    }
}, true);

registry.byId("crust").on("change", function(isChecked){
    remarkNode.innerHTML = isChecked ? "Healthy gums!" : "";
}, true);
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/demo/onChange.html)

我们在之前`value`存取器示例中已经对它有了一个预览。这个模式很简单，因为`dojo/aspect`可以让我们用处理正规元素上的DOM事件那样的方式来处理widget方法。监听器函数接受选中状态作为一个单独的参数，我们通过更新屏幕上显示的信息做出回应。这些widget提供的大量事件开辟了广泛的交互和表单逻辑选项。

##dijit/form/ToggleButton
我们再来介绍一个布尔状态按钮的变形：`dijit/form/ToggleButton`。开关按钮是一个两种状态的按钮。它在功能上与复选框和单选按钮很像，不过用户界面不同。每种状态都可以包含一个icon、文本或两者皆有。这里icon使用一个CSS类来定义。

```
<input type="checkbox" dojoType="dijit/form/ToggleButton" checked iconClass="dijitCheckBoxIcon" label="Toggle Me">
```

```
var myToggleButton = new ToggleButton({
    checked: true,
    iconClass: "dijitCheckBoxIcon",
    label: "Toggle Me, Too."
}, "toggleButtonProgrammatic");
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/checkboxes/demo/ToggleButton.html)

在这个widget，`label`属性是强制的。像所有其他属性一样，在初始化之后，`label`和`iconClass`可以通过`set`方法来更新：

```
// 为按钮提供一个新的标签
myToggleButton.set("label", "New Label");

// 链接样式表中的 .recent 规则
myToggleButton.set("iconClass", "recent");
```

注意你必须使用`<var>widget</var>.set()`方法；简单的指定属性并不能正确更新widget。

##小结
Dijit开箱即用的表单widget解决了许多用户很常见的输入和选择需求。复选框、单选按钮、开关按钮三个widget类可以帮你提供一个更丰富的、更加美观和一致的用户体验。如果表单是你项目的重要部分，Dijit里的其他widget你也可以熟悉下：`dijit/form/Menu`（和`dijit/CheckedMenuItem`），`dijit/form/Select`和相关的widget`FilteringSelect`、`ComboBox`和`MultiSelect`。`dojox/form`包里有更多选项，如果这些还不够，`dijit/form/_FormWidget`可以作为你构建时有用的基础。不过这些都是另一个教程的主题了。