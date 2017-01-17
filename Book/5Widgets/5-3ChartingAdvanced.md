# 5.3 图表进阶

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/charting_advanced/index.html

---

虽然大部分的开发者只需要基础图表，`dojox/charting`还提供了更加高级的功能：带动画的图表、响应数据变化的图表和响应事件的图表。在这篇教程中，你将学习如何使用`dojox/charting`中的一些高级功能。

##入门

用`dojox/charting`来创建高级、动态的图表可能比你想的还要容易。得益于Dojo Toolkit，创建能够处理数据变化的、能缩放、移动和滚动的图表很简单。

##Dojo Stores 和 Charting

Dojo提供一个优秀灵活的Store API，能够让程序员以高效的方式管理（增删改查）数据。由于Store在Dojo应用中的流行，`dojox/charting/StoreSeries`合并了从Store数据创建数据series的解决方案。

### StoreSeries
`dojox/charting/StoreSeries`专门用来将数据存储合并到图表中。在图表中使用数据存储的第一步是创建一个存储：

```
require(["dojo/store/Observable", "dojo/store/Memory"], function(ObservableStore, MemoryStore) {
    // Initial data
    var data = [
        // This information, presumably, would come from a database or web service
        // Note that the values for site are either 1 or 2
        { id: 1, value: 20, site: 1 },
        { id: 2, value: 16, site: 1 },
        { id: 3, value: 11, site: 1 },
        { id: 4, value: 18, site: 1 },
        { id: 5, value: 26, site: 1 },
        { id: 6, value: 19, site: 2 },
        { id: 7, value: 20, site: 2 },
        { id: 8, value: 28, site: 2 },
        { id: 9, value: 12, site: 2 },
        { id: 10, value: 4, site: 2 }
    ];

    // 创建 data store
    // 于客户端在 data store 中储存信息
    var store = new ObservableStore(new MemoryStore({
        data: {
            identifier: "id",
            label: "Users Online",
            items: data
        }
    }));
});
```
以方便查看的方式包装store是很重要的，它可以让你向store发送通知，转而通知我们将创建的StoreSeries。

有了store，就该像[基础教程](https://dojotoolkit.org/documentation/tutorials/1.10/charting/)里一样添加图表、plot和坐标轴了。创建图表、plot和坐标轴之后就需要实现StoreSeries：

```
// 给y轴添加一个StoreSeries ， 查询所有site为1的项
chart.addSeries("y", new StoreSeries(store, { query: { site: 1 } }, "value"));
```

有了StoreSeries，每次data store被通知改变的时候，series也会在图表中重新渲染。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting_advanced/demo/store-series.html)

##图表动画：Zooming、Scrolling 和 Panning
Dojo的图表解决方案很灵活，可以让数据随时改变，所以自然地，图表也要足够灵活适应数据的变化。Zooming、Scrolling 和 Panning应运而生。

每个动画扮演的角色是很直观的：

 - Zooming —— 允许开发者在不放大图表的前提下放大图表中的元素
 - Scrolling —— 允许用户以他们的方式在图表中点击和拖拽
 - Panning —— 允许用户在图表中调整不同的视野

这些特效通过两个图表方法完成：`setAxisWindow`和`setWindow`。

### setAxisWindow(name,scale,offset)
`setAxisWindow`通过带比例系数的轴定义一个窗口，设置数据坐标的偏移。`setAxisWindow`方法接收三个参数：

 - **name** —— 轴的名称
 - **scale** —— 图表变化的比例
 - **offset** —— 图表的目标偏移

`setAxisWindow`用法如下：

```
 // 将 x 轴 改为两倍比例, 偏移 100
    chart.setAxisWindow("x",2,100).render();

```

### setWindow(sx,sy,dx,dy)
`setWindow`设置整个图表绘制的比例和偏移。`setWindow`方法接收四个参数：

 - sx —— 水平轴的放大倍率
 - sy —— 垂直轴的放大倍率
 - dx —— 水平轴的偏移像素量
 - dy —— 垂直轴的偏移像素量

`setWindow`用法如下：

```
    // 将图表移回原始位置
    chart.setWindow(1, 1, 0, 0).render();
```

每个方法需要调用图表的`render`方法来将变化反映到图表中。

### 示例：Zooming、Scrolling 和 Panning
下面的例子展示如何使用滑块来zoom、pan和scroll图表。

```
<script>

    // Require the dependencies
    require(["dijit/form/HorizontalSlider", "dijit/form/HorizontalRule", "dijit/form/HorizontalRuleLabels", "dojox/charting/Chart", "dojox/charting/themes/Claro", "dojox/lang/functional/object", "dijit/registry", "dojo/on", "dojo/dom", "dojo/_base/event", "dojo/parser", "dojox/charting/axis2d/Default", "dojox/charting/plot2d/Areas", "dojox/charting/plot2d/Grid", "dojo/domReady!"], function(HorizontalSlider, HorizontalRule, HorizontalRuleLabels, Chart, Claro, functionalObject, registry, on, dom, baseEvent, parser) {

        // Initialize chart, scales, and offsets
        var chart, moveable, scaleX = 1, scaleY = 1, offsetX = 0, offsetY = 0;

        // Updates the slider values, animates the change in scale and offsets
        var reflect = function(){
            functionalObject.forIn(chart.axes, function(axis){
                var scale  = axis.getWindowScale(),
                    offset = Math.round(axis.getWindowOffset() * axis.getScaler().bounds.scale);
                if(axis.vertical){
                    scaleY  = scale;
                    offsetY = offset;
                }else{
                    scaleX  = scale;
                    offsetX = offset;
                }
            });
            setTimeout(function(){
                registry.byId("scaleXSlider").set("value", scaleX);
                registry.byId("offsetXSlider").set("value", offsetX);
                registry.byId("scaleYSlider").set("value", scaleY);
                registry.byId("offsetYSlider").set("value", offsetY);
            }, 25);
        };

        // Update the scale and offsets of *all* plots on the chart
        var update = function(){
            chart.setWindow(scaleX, scaleY, offsetX, offsetY, { duration: 1500 }).render();
            reflect();
        };

        // The following four methods are fired when the corresponding sliders are  changed
        var scaleXEvent = function(value){
            scaleX = value;
            dom.byId("scaleXValue").innerHTML = value;
            update();
        };

        var scaleYEvent = function(value){
            scaleY = value;
            dom.byId("scaleYValue").innerHTML = value;
            update();
        };

        var offsetXEvent = function(value){
            offsetX = value;
            dom.byId("offsetXValue").innerHTML = value;
            update();
        };

        var offsetYEvent = function(value){
            offsetY = value;
            dom.byId("offsetYValue").innerHTML = value;
            update();
        };

        // Function called when the mouse goes down
        var _init = null;
        var onMouseDown = function(e){
            console.warn("mousedown");
            _init = {x: e.clientX, y: e.clientY, ox: offsetX, oy: offsetY};
            baseEvent.stop(e);
        };

        // Function called when the mouse is released
        var onMouseUp = function(e){
            if(_init){
                // Clears the click/drag, updates the chart
                console.warn("mouseup");
                _init = null;
                reflect();
                baseEvent.stop(e);
            }
        };

        // Create the base chart
        chart = new Chart("chart");
        chart.setTheme(Claro);
        chart.addAxis("x", {fixLower: "minor", natural: true, stroke: "grey",
            majorTick: {stroke: "black", length: 4}, minorTick: {stroke: "gray", length: 2}});
        chart.addAxis("y", {vertical: true, min: 0, max: 30, majorTickStep: 5, minorTickStep: 1, stroke: "grey",
            majorTick: {stroke: "black", length: 4}, minorTick: {stroke: "gray", length: 2}});
        chart.addPlot("default", {type: "Areas", animate: {duration: 1800}});
        chart.addSeries("Series A", [0, 25, 5, 20, 10, 15, 5, 20, 0, 25]);
        chart.addAxis("x2", {fixLower: "minor", natural: true, leftBottom: false, stroke: "grey",
            majorTick: {stroke: "black", length: 4}, minorTick: {stroke: "gray", length: 2}});
        chart.addAxis("y2", {vertical: true, min: 0, max: 20, leftBottom: false, stroke: "grey",
            majorTick: {stroke: "black", length: 4}, minorTick: {stroke: "gray", length: 2}});
        chart.addPlot("plot2", {type: "Areas", hAxis: "x2", vAxis: "y2", animate: {duration: 1800}});
        chart.addSeries("Series B", [15, 0, 15, 0, 15, 0, 15, 0, 15, 0, 15, 0, 15, 0, 15, 0, 15], {plot: "plot2"});
        chart.addPlot("grid", { type: "Grid", hMinorLines: true });
        chart.render();

        parser.parse();

        // Add change events to the sliders to know when chart changes should be triggered
        registry.byId("scaleXSlider").on("Change", scaleXEvent, true);
        registry.byId("scaleYSlider").on("Change", scaleYEvent, true);
        registry.byId("offsetXSlider").on("Change", offsetXEvent, true);
        registry.byId("offsetYSlider").on("Change", offsetYEvent, true);

        // Add mouse events to the chart to allow click and drag
        var chartNode = dom.byId("chart");
        on(chartNode, "mousedown", onMouseDown);
        on(chartNode, "mouseup",   onMouseUp);
    });
</script>

<!-- create the sliders to control chart scale and offsets -->
<table>
    <tr><td align="center" class="pad">Scale X (<span id="scaleXValue">1</span>)</td></tr>
    <tr><td>
        <div id="scaleXSlider" data-dojo-type="dijit/form/HorizontalSlider" data-dojo-props="
                value: 1, minimum: 1, maximum: 5, discreteValues: 5, showButtons: false"
                style="width: 600px;">
            <div data-dojo-type="dijit/form/HorizontalRule" data-dojo-props="
                container: 'bottomDecoration', count: 5" style="height:5px;"></div>
            <div data-dojo-type="dijit/form/HorizontalRuleLabels" data-dojo-props="
                container: 'bottomDecoration', count: 5, minimum: 1, maximum: 5, constraints: {pattern: '##'}" style="height:1.2em;font-size:75%;color:gray;"></div>
        </div>
    </td></tr>
    <tr><td align="center" class="pad">Scale Y (<span id="scaleYValue">1</span>)</td></tr>
    <tr><td>
        <div id="scaleYSlider" data-dojo-type="dijit/form/HorizontalSlider" data-dojo-props="
                value: 1, minimum: 1, maximum: 5, discreteValues: 5, showButtons: false"
                style="width: 600px;">
            <div data-dojo-type="dijit/form/HorizontalRule" data-dojo-props="
                container: 'bottomDecoration', count: 5" style="height:5px;"></div>
            <div data-dojo-type="dijit/form/HorizontalRuleLabels" data-dojo-props="
                container: 'bottomDecoration', count: 5, minimum: 1, maximum: 5, constraints: {pattern: '##'}" style="height:1.2em;font-size:75%;color:gray;"></div>
        </div>
    </td></tr>
    <tr><td align="center" class="pad">Offset X (<span id="offsetXValue">0</span>)</td></tr>
    <tr><td>
        <div id="offsetXSlider" data-dojo-type="dijit/form/HorizontalSlider" data-dojo-props="
                value: 1, minimum: 0, maximum: 500, discreteValues: 501, showButtons: false"
                style="width: 600px;">
            <div data-dojo-type="dijit/form/HorizontalRule" data-dojo-props="
                container: 'bottomDecoration', count: 6" style="height:5px;"></div>
            <div data-dojo-type="dijit/form/HorizontalRuleLabels" data-dojo-props="
                container: 'bottomDecoration', count: 6, minimum: 0, maximum: 500, constraints: {pattern: '####'}" style="height:1.2em;font-size:75%;color:gray;"></div>
        </div>
    </td></tr>
    <tr><td align="center" class="pad">Offset Y (<span id="offsetYValue">0</span>)</td></tr>
    <tr><td>
        <div id="offsetYSlider" data-dojo-type="dijit/form/HorizontalSlider" data-dojo-props="
                value: 1, minimum: 0, maximum: 500, discreteValues: 501, showButtons: false"
                style="width: 600px;">
            <div data-dojo-type="dijit/form/HorizontalRule" data-dojo-props="
                container: 'bottomDecoration', count: 6" style="height:5px;"></div>
            <div data-dojo-type="dijit/form/HorizontalRuleLabels" data-dojo-props="
                container: 'bottomDecoration', count: 6, minimum: 0, maximum: 500, constraints: {pattern: '####'}" style="height:1.2em;font-size:75%;color:gray;"></div>
        </div>
    </td></tr>
</table><br /><br />

<!-- the chart node -->
<div id="chart" style="width: 800px; height: 400px;"></div>
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting_advanced/demo/zooming-scrolling-panning.html)（译者注：Demo中滑块失效）

##dojox/charting 事件
所有交互接口中时间连接都是重要的。重点是它们要能有效且高效地传递，并且完全可用。针对这个目标创建了一个API，开发者可以用它来连接和相应用户触发的事件。

在一个给定的图表中，事件监听通过`connectToPlot`方法分配给指定的plots：

```
//将事件连接到 "default" plot
chart.connectToPlot("default", function(evt) {
    // Use console to output information about the event
    console.info("Chart event on default plot!",evt);
    console.info("Event type is: ",evt.type);
    console.info("The element clicked was: ",evt.element);
});
```
connectToPlot的`event`对象不同时传统DOM事件。这个时间对象包含以下键值属性：

 - type —— 事件类型（`onclick`、`onmouseover` 或 `onmouseleave`）
 - element —— 悬停的元素类型（`marker`、 `bar`、 `column`、`circle`、 `slice`）
 - x —— 点的`x`值
 - y —— 点的`y`值
 - shape —— 代表数据点的gfx 形状对象

你可以在[dojox/charting参考指南](https://dojotoolkit.org/reference-guide/1.10/dojox/charting.html#chart-events)查看全部的事件属性。

这个插件涉及在基础图表教程使用图表事件解决方案来触发形状的运动。

##示例：使用图表事件
这个示例展示在鼠标悬停时用图表事件改变饼图块的颜色，并在点击时将它旋转360度。

```
// Require the basic 2d chart resource: Chart
// Retrieve the Tooltip class
// Require the theme of our choosing
require(["dojox/charting/Chart", "dojox/charting/action2d/Tooltip", "dojox/charting/themes/Claro", "dojox/charting/plot2d/Pie", "dojox/charting/axis2d/Default", "dojo/domReady!"], function(Chart, Tooltip, Claro) {

    // Define the data
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];

    // Create the chart within it's "holding" node
    var chart = new Chart("chartNode");

    // Set the theme
    chart.setTheme(Claro);

    // Add the only/default plot
    chart.addPlot("default", {
        type: "Pie",
        markers: true
    });

    // Add axes
    chart.addAxis("x");
    chart.addAxis("y", { min: 5000, max: 30000, vertical: true, fixLower: "major", fixUpper: "major" });

    // Add the series of data
    chart.addSeries("Monthly Sales - 2010", chartData);

    // Create the tooltip
    var tip = new Tooltip(chart, "default");

    // Render the chart!
    chart.render();

    // Add a mouseover event to the plot
    chart.connectToPlot("default",function(evt) {
        // Output some debug information to the console
        console.warn(evt.type," on element ",evt.element," with shape ",evt.shape);
        // Get access to the shape and type
        var shape = evt.shape, type = evt.type;
        // React to click event
        if(type == "onclick") {
            // Update its fill
            var rotateFx = new gfxFx.animateTransform({
                duration: 1200,
                shape: shape,
                transform: [
                    { name: "rotategAt", start: [0,240,240], end: [360,240,240] }
                ]
            }).play();
        }
        // If it's a mouseover event
        else if(type == "onmouseover") {
            // Store the original color
            if(!shape.originalFill) {
                shape.originalFill = shape.fillStyle;
            }
            // Set the fill color to pink
            shape.setFill("pink");
        }
        // If it's a mouseout event
        else if(type == "onmouseout") {
            // Set the fill the original fill
            shape.setFill(shape.originalFill);
        }

    });

});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting_advanced/demo/chart-events.html)

重点要里理解图表中的每一个元素只是一个GFX图形，因此我们可以像这样处理图表元素和添加动画，从而创建一些独特的特效。

##小结
基础图表不总是完全能满足要求。为动态图表调用动态数据，Dojo Toolkit的`dojox/charting`库提供全套的工具来让你的图表想你的数据一样灵活多变。

##dojox/charting 资源

 - [dojox/charting 参考指南](https://dojotoolkit.org/reference-guide/1.10/dojox/charting.html)
 - [主题预览](http://download.dojotoolkit.org/release-1.10.4/dojo-release-1.10.4/dojox/charting/tests/theme_preview.html)
 - [GFX 动画示例](http://www.sitepen.com/labs/code/london-ajax/london-ajax.html)
