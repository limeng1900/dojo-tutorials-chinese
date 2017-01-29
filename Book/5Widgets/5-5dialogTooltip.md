原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/index.html

---

对于构建响应式、交互式的web应用来说，用户交互是极为重要的。web浏览器在警告和对话框方面为用户交互提供了基本的方法，但是这些功能既不优雅也不灵活。`dijit/Tooltip`、`dijit/Dialog`和`dijit/TooltipDialog`、Dijit、Dojo工具集的UI框架提供了跨浏览其、可扩展和主题化的解决方案，来应对浏览器基础功能的缺陷。在这篇教程中，你讲了解这些widget和相关示例，以及如何创建他们的来龙去脉。

##工具提示入门

原生“Tooltips”是通过使用DOM节点的`title`属性在浏览器中创建的。这些工具提示毫无特色：无法控制显示的时间，没有富文本能力，还有很小的跨浏览器一致性。Dijit的`dijit/Tooltip`类可以解决所有这些问题：

 - 允许HTML包含自定义的提示
 - 提供控制提示显示位置和时间的方法
 - 在浏览器大小变化时调整提示的位置和大小
 - 为Flash元素的提示显示实现可靠的跨浏览器策略
 - 提示的“惰性”创建——直到Tooltip 必须显示时才创建Tooltip 节点

`dijit/Tooltip`的用法和其他任意Dijit widget是一样的：将需要的主题样式添加到页面，将主题的名称作为一个CSS类添加到body节点，require widget的JavaScript类：

```
<head>
<!-- 使用 "claro" 主题 -->
<link rel="stylesheet" href="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dijit/themes/claro/claro.css">
<!-- 加载 dojo 并通过data属性提供配置 -->
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js" data-dojo-config="async: true, parseOnLoad:true"></script>
<script>
    // 加载 Tooltip widget 类
    require(["dijit/Tooltip",  "dojo/parser", "dojo/domReady!"], function(Tooltip, parser){
        parser.parse();
    });
</script>
</head>
<!-- 给body添加 "claro" CSS 类 -->
<body class="claro">

</body>
```

主题和widget类的加载之后，`dijit/Tooltip`编程式用法的一个基本示例如下：

```
// 创建一个新的 Tooltip
var tip = new Tooltip({
    // 标签 —— 放进 Tooltip 的HTML或者文本
    label: '&lt;div class="myTipType"&gt;This is the content of my Tooltip!&lt;/div&gt;',
    // Tooltip 显示前的延时 （毫秒为单位）
    showDelay: 250,
    // 要附加提示的节点
    // 可以是一个字符串数组或 domNodes
    connectId: ["myElement1","myElement2"]
});
```

`dijit/Tooltip`的重要属性包括：

 - **connectId** —— Tooltip要连接的一个ID数组或者DOM节点
 - **label** —— 放入Tooltip的HTML或者文本内容
 - **showDelay** —— Tooltip的显示延时

`dijit/Tooltip`的重要方法包括：

 - **addTarget** —— 如果还没有连接则添加一个Tooltip目标
 - **close** —— 关闭一个Tooltip实例（视觉上隐藏）
 - **open** —— 打开一个Tooltip实例（使Tooltip可见）
 - **removeTarget** —— 从Tooltip的目标列表中移除一个节点
 - **set** ——允许改变属性，尤其是Tooltip内容（`myTip.set("label","New content!")`）

`dijit/Tooltip`对象也拥有一个可配置的`defaulltPosition`数组，包含一个Tooltip实例应该展示的顺序：

```
Tooltip.defaultPosition = ["above", "below", "after-centered", "before-centered"];
```

这个数组可以按需改变 。

>注意改变`Tooltip.defaultPosition`会改变所有提示的显示位置。

##dijit/Tooltip 示例

下面是`dijit/Tooltip`非常自定义化的运用。

### 声明式（HTML）Tooltip创建

```
<button id="TooltipButton" onmouseover="dijit.Tooltip.defaultPosition=['above', 'below']">Tooltip Above</button>
<div class="dijitHidden"><span data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'TooltipButton'">I am <strong>above</strong> the button</span></div>

<button id="TooltipButton2" onmouseover="dijit.Tooltip.defaultPosition=['below','above']">Tooltip Below</button>
<div class="dijitHidden"><span data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'TooltipButton2'">I am <strong>below</strong> the button</span></div>

<button id="TooltipButton3" onmouseover="dijit.Tooltip.defaultPosition=['after','before']">Tooltip After</button>
<div class="dijitHidden"><span data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'TooltipButton3'">I am <strong>after</strong> the button</span></div>

<button id="TooltipButton4" onmouseover="dijit.Tooltip.defaultPosition=['before','after']">Tooltip Before</button>
<div class="dijitHidden"><span data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'TooltipButton4'">I am <strong>before</strong> the button</span></div>
```
 >[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/tooltip-mixed.html)

### 编程式Tooltip创建

```
// 添加图片的提示
new Tooltip({
    connectId: ["nameTip"],
    label: "&lt;img src='rod-stewart.jpg' alt='Rod Stewart' width='300' height='404' /&gt;"
});
// 添加 North London 的提示
new Tooltip({
    connectId: ["londonTip"],
    label: "&lt;img src='emirates-stadium.jpg' alt='The Emirates in London' width='400' height='267' /&gt;"
});
//Add Tooltip of record
new Tooltip({
    connectId: ["recordsTip"],
    label: "&lt;img src='every-picture.jpg' alt='Every Picture Tells a Story' width='200' height='197' /&gt;"
});
// 添加自定义提示
var myTip = new Tooltip({
    connectId: ["hoverLink"],
    label: "Don't I look funky?",
    "class": "customTip"
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/tooltip-mixed.html)

### 产品详细信息

```
<ul>
    <li><a href="http://www.imdb.com/title/tt0112573/" id="movieBraveheart">Braveheart</a></li>
    <li><a href="http://www.imdb.com/title/tt0237534/" id="movieBrotherhood">Brotherhood of the Wolf</a></li>
    <li><a href="http://www.imdb.com/title/tt0245844/" id="movieCristo">The Count of Monte Cristo</a></li>
</ul>
<div class="dijitHidden">
    <div data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'movieBraveheart'">
        <img style="width:100px; height:133px; display:block; float:left; margin-right:10px;" src="../images/braveheart.jpg" />
        <p style="width:400px;"><strong>Braveheart</strong><br />Braveheart is the partly historical, partly mythological, story of William Wallace, a Scottish common man who fights for his country's freedom from English rule around the end of the 13th century...</p>
        <br style="clear:both;">
    </div>
</div>
<div class="dijitHidden">
    <div data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'movieBrotherhood'">
        <img style="width:100px; height:133px; display:block; float:left; margin-right:10px;" src="../images/brotherhood.jpg" />
        <p style="width:400px;"><strong>Brotherhood of the Wolf</strong><br />In 1765 something was stalking the mountains of central France. A 'beast' that pounced on humans and animals with terrible ferocity...</p>
        <br style="clear:both;">
    </div>
</div>
<div class="dijitHidden">
    <div data-dojo-type="dijit/Tooltip" data-dojo-props="connectId:'movieCristo'">
        <img style="width:100px; height:133px; display:block; float:left; margin-right:10px;" src="../images/count.jpg" />
        <p style="width:400px;"><strong>The Count of Monte Cristo</strong><br />'The Count of Monte Cristo' is a remake of the Alexander Dumas tale by the same name. Dantes, a sailor who is falsely accused of treason by his best friend Fernand, who wants Dantes' girlfriend Mercedes for himself...</p>
        <br style="clear:both;">
    </div>
</div>
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/tooltip-details.html)

基础Tooltip widget对于提供丰富信息非常有用，但是如果你需要一个更加突出的widget呢？Dijit的Dialog widget是一个完美的选择！

## Dialog入门

当从用户获取信息或者发出通知时，浏览器的原生`alert`和`confirm`方法还不够好用。它们死板又难看。幸运的是，Dojo Toolkit由`dijit/Dialog`提供一个了替代。和`dijit/Tooltip`很像，`dijit/Dialog`允许内含HTML并且轻松主体化。`dijit/Dialog`的一个简单应用如下：

```
// 创建一个新的 dijit/Dialog 实例
var myDialog = new Dialog({
    // The dialog's title
    title: "The Dojo Toolkit",
    // The dialog's content
    content: "This is the dialog content.",
    // Hard-code the dialog width
    style: "width:200px;"
});
```

关于`dijit/Dialog`重点要知道的是实例是添加到一个“堆栈”的，所以你可以一个上面再有一个实例。对话框显示也是由一个iFrame支持的，因此可以确保总在其他元素的“上面”。一个单独的`dijit/DialogUnderlay`实例将被所有的Dialog共享。

`dijit/Dialog`的重要属性包括：

 - **content** —— Dialog的HTML或文本内容
 - **draggable** —— 表示Dialog是否可以拖拽
 - **href** —— 如果内容通过Ajax（`xhrGet`）加载，则为指向内容文件的路径
 - **loadingMessage** —— Ajax内容加载时显示的信息
 - **open** —— 如果Dialog实例目前被打开则返回true
 - **title** ——在Dialog顶上显示的标题

`dijit/Dialog`的重要方法包括：

 - **hide** —— 隐藏对话框和底层
 - **refresh** —— 如果对话框是基于Ajax的，刷新它的内容
 - **show** —— 显示对话框和底层

`dijit/Dialog`也提供你期待的回调方法：onShow、onHide、onLoad、onClick以及更多。

##dijit/Dialog 示例

下面是`dijit/Dialog`的一个自定义示例：

### 条款和协议

```
<script>
    // Require the Dialog class
    require(["dijit/registry", "dojo/parser", "dijit/Dialog", "dijit/form/Button", "dojo/domReady!"], function(registry, parser){
        // Show the dialog
        showDialog = function() {
            registry.byId("terms").show();
        }
        // Hide the dialog
        hideDialog = function() {
            registry.byId("terms").hide();
        }

        parser.parse();
    });
</script>
<button onclick="showDialog();">View Terms and Conditions</button>

<div class="dijitHidden">
    <div data-dojo-type="dijit/Dialog" style="width:600px;" data-dojo-props="title:'Terms and Conditions'" id="terms">
        <p><strong>Please agree to the following terms and conditions:</strong></p>
        <div style="height:160px;overflow-y:scroll;border:1px solid #769dc4;padding:0 10px;width:600px"><p>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed sed suscipit massa. Aenean vel turpis tincidunt velit gravida venenatis. In iaculis urna non quam tincidunt elementum. Nunc pellentesque aliquam dui, ac facilisis massa sollicitudin et. Donec tincidunt vulputate ultrices. Duis eu risus ut ipsum auctor scelerisque non quis ante. Nam tempor lobortis justo, et rhoncus mauris cursus et. Mauris auctor congue lectus auctor ultrices. Aenean quis feugiat purus. Cras ornare vehicula tempus. Nunc placerat, lorem adipiscing condimentum sagittis, augue velit ornare odio, eget semper risus est et erat....
        </p></div>

        <button onclick="hideDialog();">I Agree</button>
        <button onclick="alert('You must agree!');">I Don't Agree</button>
    </div>
</div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/dialog-terms.html)

### 叠放对话框

```
<script>
    // Require the Dialog class
    require(["dijit/Dialog"], function(Dialog) {
        // Create counter
        var counter = 1;
        // Create a new Dialog
        createDialog = function(first) {
            // Create a new dialog
            var dialog = new Dialog({
                // Dialog title
                title: "New Dialog " + counter,
                // Create Dialog content
                content: (!first ? "I am a dialog on top of other dialogs" : "I am the bottom dialog") + "<br /><br /><button onclick='createDialog();'>Create another dialog.</button>"
            });
            dialog.show();
            counter++;
        }
    });

</script>
<button onclick="createDialog(true);">Create New Dialog</button>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/dialog-stacked.html)

### 带有黑色垫层的Ajax Dialog

```
<style>
    /* colors the underlay black instead of white
     * We're using '.claro .dijitDialogUnderlay' as our selector,
     * to match the specificity in claro.css */
    .claro .dijitDialogUnderlay { background:#000; }
</style>

<script>
    // Require the Dialog class
    require(["dijit/registry", "dojo/parser", "dijit/Dialog", "dojo/domReady!"], function(registry, parser){
        // Show the dialog
        showDialog = function() {
            registry.byId("ajaxDialog").show();
        }

        parser.parse();
    });
</script>

<button onclick="showDialog();">Load Ajax Dialog</button>

<div class="dijitHidden">
    <!-- dialog that gets its content via ajax, uses loading message -->
    <div data-dojo-type="dijit/Dialog" style="width:600px;" data-dojo-props="title:'Ajax Dialog',href:'dialog-ajax-content.html',loadingMessage:'Loading dialog content...'" id="ajaxDialog"></div>
</div>
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/dialog-ajax.html)

##dijit/TooltipDialog 入门

Dijit的`TooltipDialog`的widget完美混合了Tooltip和Dialog，创建了一个可设定焦点的、富“弹出”元素。`TooltopDialog` widget用其他widget打开为下拉菜单，通常使用`dijit/form/DropDownButton`。`Tooltip`和`TooltipDialog` widget之间的区别是`TooltipDialog`将保持打开状态直到用户点击widget以外的地方，这样“Tooltip”中就能拥有可点击的链接、表单元素等widget，而不是像Tooltip那样在你鼠标移出的时候就关闭。

`dijit/TooltipDialog` widget 的属性、方法和事件大部分与Tooltip和Dialog一样。

##dijit/TooltipDialog 示例

下面是`dijit/TooltipDialog`的一个自定义用法。

### 按钮下拉菜单

```
<script>
    // Require the Button, TooltipDialog, DropDownButton, and TextBox classes
    require(["dojo/parser", "dijit/form/DropDownButton", "dijit/TooltipDialog", "dijit/form/TextBox", "dijit/form/Button", "dojo/domReady!"],
    function(parser){
        parser.parse();
    });

</script>
<div data-dojo-type="dijit/form/DropDownButton">
    <span>Login</span><!-- Text for the button -->
    <!-- The dialog portion -->
    <div data-dojo-type="dijit/TooltipDialog" id="ttDialog">
        <strong><label for="email" style="display:inline-block;width:100px;">Email:</label></strong>
        <div data-dojo-type="dijit/form/TextBox" id="email"></div>
        <br />
        <strong><label for="pass" style="display:inline-block;width:100px;">Password:</label></strong>
        <div data-dojo-type="dijit/form/TextBox" id="pass"></div>
        <br />
        <button data-dojo-type="dijit/form/Button" data-dojo-props="onClick:doAlert" type="submit">Submit</button>
    </div>
</div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/dialogs_tooltips/demo/ttd-button.html)

在不需要完整对话框的情况下，`TooltipDialog`是弹出内容交互的重要方式。

##小结

Dojo Toolkit不止让你更容易地完成基本任务，同时也为你提供了跨浏览器一致性的、灵活的、主体化的widget。这里描述的widget提供了浏览器基础功能的强大代替。使用Dijit的Tooltip、Dialog和TooltipDialog来丰富你的网站吧！

##Dialog 和 Tooltip资源

关于Dijit的Dialog和Tooltip的更多细节请查看一下资源：

 - [dijit/Tooltip API Documentation](https://dojotoolkit.org/api/1.10/dijit/Tooltip.html)
 - [dijit/Dialog API Documentation](https://dojotoolkit.org/api/1.10/dijit/Dialog.html)
 - [dijit/TooltipDialog API Documentation](https://dojotoolkit.org/api/1.10/dijit/TooltipDialog.html)