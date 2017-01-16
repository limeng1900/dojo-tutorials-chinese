# 5.2 图表

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/charting/index.html

---

以可读的、引人注目的方式呈现统计数据是很重要的，但也是困难的。`dojox/charting`系统的创建就是为了减轻这种痛苦，开发者可以用它从不同的数据集创建动态、独特且具备功能的图表。另外，`dojox/charting`提供大量的主题和图表类型，可以让开发者以喜欢的方式展示数据。这篇教程将向你展示如何以不同的数据、绘制、轴线和主题来创建基本图表。

##入门

Dojo的图表库位于`dojox/charting`资源中。`dojox/charting`非常独特：

 - 允许图表以HTML（声明式）或者JavaScript（编程式）来创建
 - 适用几乎所有的设备
 - 可以在SVG、VML、Silverlight和Canvas里渲染图表。正努力实现SVGWeb渲染。
 - 开发者可以决定使用哪种渲染器
 - 评估客户端并且使用基于客户端支持的合适的渲染器
 - 用[dojox/gfx](https://dojotoolkit.org/reference-guide/1.10/dojox/gfx.html)创建图表，它是一个强大的矢量图像库能够让你的图表以各种各样的方式生动起来。
 - 打包了几十种多种多样的、有吸引力的主题
 - 在图表主题中可以使用线性和放射渐变（甚至IE里也可以）

##配置dojox/charting

在创建这些绝妙的图表之前，重点是要将图表资源放进页面。

对于任何Dojo Toolkit源码，都使用`require`来加载依赖项。开发人员通常需要的两个依赖项是图表资源和想要的[主题](http://archive.dojotoolkit.org/nightly/dojotoolkit/dojox/charting/tests/theme_preview.html)：

```
require([
     // Require the basic 2d chart resource
    "dojox/charting/Chart",

    // Require the theme of our choosing
    "dojox/charting/themes/Claro",

], function(Chart, theme){
    // ....
}
```
如果要优先指定一个渲染器，需要在加载Dojo之前加入创建的dojoConfig对象中：

```
<script>
    dojoConfig = {
        parseOnLoad: true, //enables declarative chart creation
        gfxRenderer: "svg,silverlight,vml" // svg is first priority
    };
</script>
<script src="/path/to/dojo/dojo/dojo.js"></script>
```
注意在1.7+，Chart2D 加载所有的轴线和绘制类型的“全方位”模块被废弃了。它还是可以使用的，但是首选的方法是使用Chart并只包含你需要的轴线和绘制模块。

随着这些最小化的依赖项加载，你的应用现在可以创建图表了。

##创建一个基本图表

### 声明式
有两种方式来创建一个基本图表：声明式和编程式。无论如何，在创建图表之前，重点是先创建/获取数据。我们将用到下面的数据样例来创建基本图表：

```
//x和y坐标用来更方便理解他们应该显示在哪 
// 数据表示一周里网站的访问
chartData = [
    { x: 1, y: 19021 },
    { x: 1, y: 12837 },
    { x: 1, y: 12378 },
    { x: 1, y: 21882 },
    { x: 1, y: 17654 },
    { x: 1, y: 15833 },
    { x: 1, y: 16122 }
];
```
有了正确格式化和可用的数据，就可以像这样声明式创建图表：

```
<!-- 创建图表 -->
<div
    data-dojo-type="dojox/charting/widget/Chart"
    data-dojo-props="theme:dojox.charting.themes.Claro" id="viewsChart" style="width: 550px; height: 550px;">

    <!-- Pie Chart: add the plot -->
    <div class="plot" name="default" type="Pie" radius="200" fontColor="#000" labelOffset="-20"></div>

    <!-- pieData is the data source -->
    <div class="series" name="Last Week&#x27;s Visits" array="chartData"></div>
</div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/basic-declarative.html)

使用声明式图表创建，主要的图表设置都放在容器节点。plot和series获得它们自己的带有自定义属性节点，这些自定义属性包含了图表设置。

重点记住：虽然可以用声明式创建图表，但是高度建议开发者使用编程式创建。dojox/charting还没有完全支持Dojo 1.6+引入的`data-dojo`属性。

### 编程式
编程式图表创建需要更多一点的代码提供更强的稳定性和控制。编程式创建与上面相同的图表可以使用下面的代码：

```
<script>
    require([
         // 加载模块...
    ], function(Chart, theme, PiePlot){

        // 在节点内创建图表
        var pieChart = new Chart("chartNode");

        // 设置主题
        pieChart.setTheme(theme);

        // 添加 only/default plot
        pieChart.addPlot("default", {
            type: PiePlot, // our plot2d/Pie module reference as type value
            radius: 200,
            fontColor: "black",
            labelOffset: -20
        });

        // 添加数据 series
        pieChart.addSeries("January",chartData);

        // 渲染图表
        pieChart.render();

    });
</script>

<!-- create the chart -->
<div id="chartNode" style="width: 550px; height: 550px;"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/basic-programmatic.html)

上面的代码是`dojox/charting`的一个基础示例，但是图表创建远不止如此。让我们来深度挖掘`dojox/charting`和它的功能。

##图表主题

Dojo图表图为开发者提供了大量的主题。这些主题多种多样；一些主题使用可靠的十六进制颜色，同时更多复杂图表使用高级立即来计算线性甚至径向渐变来增强外观。主题都放在`dojox/charting/themes`资源里。主题是一个简单的带有主题特定数据的JavaScript文件。下面的代码将创建“Miami Nice”主题：

```
define([
    // Require the SimpleTheme class which is used by all non-gradient based themes
    "../SimpleTheme",
    "./common"
], function(Theme, themes){

    themes.MiamiNice=new Theme({
        colors: [
            "#7f9599",
            "#45b8cc",
            "#8ecfb0",
            "#f8acac",
            "#cc4482"
        ]
    });

    return themes.MiamiNice;
});
```
通过require`dojox/charting/themes/Theme`能得到更复杂的主题。一个例子是使用渐变和自定义字体设置的“Claro”主题：

```
define(["../Theme", "dojox/gfx/gradutils", "./common"], function(Theme, gradutils, themes){
    // created by Tom Trenka

    var g = Theme.generateGradient,
        defaultFill = {type: "linear", space: "shape", x1: 0, y1: 0, x2: 0, y2: 100};

    themes.Claro = new Theme({
        chart: {
            fill:       {
                type: "linear",
                x1: 0, x2: 0, y1: 0, y2: 100,
                colors: [
                    { offset: 0, color: "#dbdbdb" },
                    { offset: 1, color: "#efefef" }
                ]
            },
            stroke:    {color: "#b5bcc7"}
        },
        plotarea: {
            fill:       {
                type: "linear",
                x1: 0, x2: 0, y1: 0, y2: 100,
                colors: [
                    { offset: 0, color: "#dbdbdb" },
                    { offset: 1, color: "#efefef" }
                ]
            }
        },
        axis:{
            stroke:    { // the axis itself
                color: "#888c76",
                width: 1
            },
            tick: {    // used as a foundation for all ticks
                color:     "#888c76",
                position:  "center",
                font:      "normal normal normal 7pt Verdana, Arial, sans-serif",    // labels on axis
                fontColor: "#888c76"                                // color of labels
            }
        },
        series: {
            stroke:  {width: 2.5, color: "#fff"},
            outline: null,
            font: "normal normal normal 7pt Verdana, Arial, sans-serif",
            fontColor: "#131313"
        },
        marker: {
            stroke:  {width: 1.25, color: "#131313"},
            outline: {width: 1.25, color: "#131313"},
            font: "normal normal normal 8pt Verdana, Arial, sans-serif",
            fontColor: "#131313"
        },
        seriesThemes: [
            {fill: g(defaultFill, "#2a6ead", "#3a99f2")},
            {fill: g(defaultFill, "#613e04", "#996106")},
            {fill: g(defaultFill, "#0e3961", "#155896")},
            {fill: g(defaultFill, "#55aafa", "#3f7fba")},
            {fill: g(defaultFill, "#ad7b2a", "#db9b35")}
        ],
        markerThemes: [
            {fill: "#2a6ead", stroke: {color: "#fff"}},
            {fill: "#613e04", stroke: {color: "#fff"}},
            {fill: "#0e3961", stroke: {color: "#fff"}},
            {fill: "#55aafa", stroke: {color: "#fff"}},
            {fill: "#ad7b2a", stroke: {color: "#fff"}}
        ]
    });

    themes.Claro.next = function(elementType, mixin, doPost){
        var isLine = elementType == "line";
        if(isLine || elementType == "area"){
            // custom processing for lines: substitute colors
            var s = this.seriesThemes[this._current % this.seriesThemes.length],
                m = this.markerThemes[this._current % this.markerThemes.length];
            s.fill.space = "plot";
            if(isLine){
                s.stroke  = { width: 4, color: s.fill.colors[0].color};
            }
            m.outline = { width: 1.25, color: m.fill };
            var theme = Theme.prototype.next.apply(this, arguments);
            // cleanup
            delete s.outline;
            delete s.stroke;
            s.fill.space = "shape";
            return theme;
        }
        else if(elementType == "candlestick"){
            var s = this.seriesThemes[this._current % this.seriesThemes.length];
            s.fill.space = "plot";
            s.stroke  = { width: 1, color: s.fill.colors[0].color};
            var theme = Theme.prototype.next.apply(this, arguments);
            return theme;
        }
        return Theme.prototype.next.apply(this, arguments);
    };

    themes.Claro.post = function(theme, elementType){
        theme = Theme.prototype.post.apply(this, arguments);
        if((elementType == "slice" || elementType == "circle") &amp;&amp; theme.series.fill &amp;&amp; theme.series.fill.type == "radial"){
            theme.series.fill = gradutils.reverse(theme.series.fill);
        }
        return theme;
    };

    return themes.Claro;
});
```
不管你实现（或创建）的主题是基础还是复杂，在你的图表中都可以很简单地使用它。简单地require该源码并在图表上调用“setTheme”：

```
require([
     // Require 基础 2d 图表资源: Chart2D
    "dojox/charting/Chart",

    // Require 我们选择的主题
    "dojox/charting/themes/Claro",

], function(Chart, theme){

    // 在节点中创建图表
    var pieChart = new Chart("chartNode");

    // 设置主题
    pieChart.setTheme(theme);

    // ...
});
```
你可以在一个给定的页面上使用任意数量的主题。如果你想要了解如何为你的web应用创建一个自定义图表主题，请阅读[Dojo Charting: Dive Into Theming](http://www.sitepen.com/blog/2012/11/09/dojo-charting-dive-into-theming/)。

##图表组件：Plots、Axes、Series

定义一个基础图表并实现主题是非常简单的。真正的工作在于定义plots（绘制类型）、axes（轴线）和series。它们每一块都独特且重要。

### Plots
在`dojox/charting`中plots的重点是定义添加的图表的类型，并提供特定图表类型的设置内容。`dojox/charting`包含众多的2D图表：

 - **Default** —— 渲染线的普通折线图，填充位于线下的区域，并在数据点放置标记。当添加图表时如果没有指定绘制类型，就会使用这个类型。
 - **Lines**—— 基础折线图。使用Default。
 - **Areas**—— 折线下的区域将被填充。使用Default。
 - **Markers**—— 带有标记的折线。使用Default。
 - **MarkersOnly**—— 标记，无折线。使用Default。
 - **Stacked**—— 数据集相对于之前的数据集设置图表。对Default的扩展。
 - **StackedLines **—— 使用折线叠放数据集。使用Stacked。
 - **StackedAreas**—— 使用线下填充区域叠放数据集。使用Stacked。
 - **Bars**—— 水平直方图。
 - **ClusteredBars **—— 群数据集的水平直方图。使用Bars。
 - **StackedBars **—— 使用水平直方图叠放数据集。使用Bars。
 - **Columns**—— 垂直直方图。
  - **ClusteredColumns **—— 群数据集的垂直直方图。使用Columns。
 - **StackedColumns**—— 使用垂直直方图叠放数据集。使用Columns。
 - **Pie**—— 传统饼图。
 - **Scatter **—— 类似MarkerOnly，不过图表使用梯度字段。

你可以在[Dojo's nightly charting tests](http://archive.dojotoolkit.org/nightly/dojotoolkit/dojox/charting/tests/test_chart2d.html)查看每一种图表类型。

使用图表的`addPlot`方法将Plots添加到图表，并将图表名（通常“default”）和 指定绘制选项传递给该方法：

```
// Add the default plot
chart.addPlot("default",{
    // Add the chart type
    type: "Pie"
});
```
当使用`dojox/charting/Chart`时你需要`require`每一个你要用到的图表类型模块。

一些标准的绘制选项如下：

 - **type** —— 图表类型（Pie、Bars、Scatter等）
 - **lines** —— 表示是否将折线添加到图表中
 - **markers** —— 表示是否将数据点标记添加到图表中
 - **areas** —— 表示是否将图表中的区域着色
 - **shadows** —— 表示是否为折线添加阴影（例如：`dx:4, dy:4}`）
 - **tension** —— 为折线添加曲线调整来提高平滑度。值可以是：
	 - **X** —— 三次贝塞尔曲线
	 - **x** ——  类似“X”但是假设数据集是闭合的（回路）。可以在绘制真实XY数据时使用。
	 - **S** —— 二次贝塞尔曲线
 - **gap** —— 表示直方图之间的间隔像素数

每一中图表类型都可能有它的自定义选项。例如，Pie绘制类型有一个`radius`设置来定义图表的半径大小。

```
// Add the default plot
chart.addPlot("default",{
    // 添加图表类型
    type: "Pie",
    // 因为是个Pie，添加半径大小
    radius: 200 //pixels
});
```
在你创建图表之前，花点事件看看 [dojox/charting参考指南](https://dojotoolkit.org/reference-guide/1.10/dojox/charting.html)了解你喜欢的图表类型的特殊设置和自定义选项。

### Axes
大多数图表都有坐标轴，其中许多都是传统的`x`和`y`。坐标轴可以是水平（默认）或垂直。使用`addAxis`方法给图表添加轴线。下面的代码给一个图表添加`x`和`y`轴：

```
// Add the X axis
chart.addAxis("x");
// Ad the Y axis
chart.addAxis("y",{
    vertical: true // y is vertical!
});
```

你也可以在图表中创建自定义轴：

```
// 添加一个自定义 "dw" axis
chart.addAxis("dw",{
    vertical: true,
    leftBottom: false
});
```
下面是一些标准的轴选项：

 - **fixUpper** —— 排列图表记号（可以是 "major"、"minor"、"micro"和"none"）
 - **fixLower** —— 排列图表记号（可以是 "major"、"minor"、"micro"和"none"）
 - **leftBottom** —— 确定轴防止在图表的哪一边（默认为`true`）
 - **min** —— 轴开始的最小数
 - **max** —— 轴结束的最大数

你可以在Dojo的参考指南了解更多关于[axis设置](https://dojotoolkit.org/reference-guide/1.10/dojox/charting.html#enabling-and-disabling-tick-marks)，包括自定义颜色、字体、步进和精度等。

##Series
这个元素简单指一系列要渲染到图表的数据。使用`addSeries`来向图表添加series。`addSeries`接收三个参数：

 - name —— series的名称。当使用Legend插件时也代表series标签。
 - data —— 数据数组。
 - options —— 一个包含series选项的对象，它包含：
	 - stroke —— 线的颜色和宽度（例如：`{ color:"red", width: 2 }`）
	 - fill ——  bar / line / pie 块的填充色

添加一个series像下面这么容易：

```
// Add a simple axis to the chart
chart.addSeries("Visits",[10,20,30,40,50],{
    stroke: {
        color: "blue",
        width: 3
    },
    fill: "#123456"
});
```

一个多轴series数据如下这样：

```
// 给图表添加一个多轴数据 series
chart.addSeries("Visits",[
    { x: 1, y: 200 },
    { x: 2, y: 185 },
    // and so on...
],{
    stroke: {
        color: "blue",
        width: 3
    },
    fill: "#123456"
});

```
一个如表可以拥有任意数量的重叠series。

##dojox/charting Examples
随着`dojox/charting`一块一块的定义，也到了创建一些基础图表的时候了。

### 折线图：月销售量
月销售量图是一个折线图，它有多重轴和“Tom”主题。

```
<script>
require([
     // Require 基础图表类
    "dojox/charting/Chart",

    // Require 我们选择的主题
    "dojox/charting/themes/Tom",

    // 图表插件:

    //     我们打算绘制折线图
    "dojox/charting/plot2d/Lines",

    //    要使用Markers
    "dojox/charting/plot2d/Markers",

    //   将使用默认的 x/y 轴
    "dojox/charting/axis2d/Default",

    // 等待DOM准备好
    "dojo/domReady!"
], function(Chart, theme) {
    // When the DOM is ready and resources are loaded...

    // 定义数据
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];

    //在节点创建图表
    var chart = new Chart("chartNode");

    // 设置主题
    chart.setTheme(theme);

    // 添加 only/default plot
    chart.addPlot("default", {
        type: "Lines",
        markers: true
    });

    // 添加 axes
    chart.addAxis("x");
    chart.addAxis("y", { min: 5000, max: 15000, vertical: true, fixLower: "major", fixUpper: "major" });

    // 添加 数据 series 
    chart.addSeries("SalesThisDecade",chartData);

    // 渲染图表！
    chart.render();

});

</script>

<div id="chartNode" style="width:800px;height:400px;"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/monthly-sales.html)

### 叠放区域图：月销售量
这个图表基于上一个图表，但是它添加了第二个坐标轴来展示多个数据集。这个图表使用Dollar主题。

```
<script>
require([
     // Require 基础2D图表资源: Chart2D
    "dojox/charting/Chart",

    // Require 选择的主题
    "dojox/charting/themes/Dollar",

    // Charting plugins:

    //    使用 StackedAreas
    "dojox/charting/plot2d/StackedAreas",

    //    使用 Markers
    "dojox/charting/plot2d/Markers",

    //    使用 default x/y 坐标轴
    "dojox/charting/axis2d/Default",

    // Wait until the DOM is ready
    "dojo/domReady!"
], function(Chart, theme) {

    // 定义数据
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];
    var chartData2 = [3000,12000,17733,9876,12783,12899,13888,13277,14299,12345,12345,15763];

    // 在节点中创建图表
    var chart = new Chart("chartNode");

    // 设置主题
    chart.setTheme(theme);

    // Add the only/default plot
    chart.addPlot("default", {
        type: "StackedAreas",
        markers: true
    });

    // 添加 axes
    chart.addAxis("x");
    chart.addAxis("y", { min: 5000, max: 30000, vertical: true, fixLower: "major", fixUpper: "major" });

    //  添加 the series of data
    chart.addSeries("Monthly Sales - 2010",chartData);
    chart.addSeries("Monthly Sales - 2009",chartData2);

    // 渲染图表！
    chart.render();

});</script>

<div id="chartNode" style="width:800px;height:400px;"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/monthly-sales-stacked.html)

### 列图表：月销售额
这个图表基于原始的折线图但是使用列图表。列之间间隔5像素，使用MiamiNice主题。

```
<script>
require([
     // Require 基本图表类
    "dojox/charting/Chart",

    // Require 选择的主题
    "dojox/charting/themes/MiamiNice",

    // Charting plugins:

    //     使用Columns
    "dojox/charting/plot2d/Columns",

    //    使用Markers
    "dojox/charting/plot2d/Markers",

    //   使用 默认 x/y 坐标轴
    "dojox/charting/axis2d/Default",

    // 等待DOM准备
    "dojo/domReady!"
], function(Chart, theme) {

    // 定义数据
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];

    // 在节点中创建图表
    var chart = new Chart("chartNode");

    // 设置主题
    chart.setTheme(theme);

    // 添加 only/default plot
    chart.addPlot("default", {
        type: "Columns",
        markers: true,
        gap: 5
    });

    // 添加坐标轴
    chart.addAxis("x");
    chart.addAxis("y", { vertical: true, fixLower: "major", fixUpper: "major" });

    // 添加 series of data
    chart.addSeries("Monthly Sales",chartData);

    // 渲染图表！
    chart.render();

});

</script>

<div id="chartNode" style="width:800px;height:400px;"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/monthly-sales-column.html)

##图表插件
`dojox/charting`库提供功能强大和美观的插件来加强你的表单。

### 图例
图表经常会包含图例来进一步明确图表提供的数据。[`dojox/charting/widget/Legend`](https://dojotoolkit.org/api/?qs=1.10/dojox/charting/widget/Legend)很容易用：require源码并创建一个实例，给它指定一个图表：

```
<script>
require([
    // Require 基础图表类
    "dojox/charting/Chart",

    // 加载 Legend widget 类
    "dojox/charting/widget/Legend"
], function(Chart, Legend) {
    // 在节点中创建图表
    var chart = new Chart("chartNode");

    //图例放在 "legend1" ID的元素
    var legend = new Legend({ chart: chart }, "legend1");
});
</script>
<!-- 为图表创建进DOM节点-->
<div id="chartNode" style="width:800px;height:400px;"></div>
<div id="legend"></div>

```

### Tooltip
[Tooltip插件](https://dojotoolkit.org/api/?qs=1.10/dojox/charting/action2d/Tooltip)在鼠标悬停在标记和图表块时根据布标类型展示值。使用Tooltip插件和图例插件一样简单，只要简单地包含插件源码，并给图表和plot分配一个新的实例：

```
<script>
require([
    // Require 基础图表类
    "dojox/charting/Chart",

    // 加载 Tooltip 类
    "dojox/charting/action2d/Tooltip"
], function(Chart, Tooltip) {
    // 在节点内创建图表
    var chart = new Chart("chartNode");

    var tip = new Tooltip(chart, "default");
});
</script>
```
包含你选择的Dijit主题来定义你的tooltips样式。也要注意Tooltip插件必须在图表调用渲染方法之前给图表分配实例。

### MoveSlice and Magnify
[MoveSlice](https://dojotoolkit.org/api/?qs=1.10/dojox/charting/action2d/MoveSlice)和[Magnify](https://dojotoolkit.org/api/?qs=1.10/dojox/charting/action2d/Magnify)插件使用疑点动画来响应鼠标移动事件。MoveSlice 移动一个饼图的块，Magnify 稍微放大图表标记。它们都是通过创建一个新的实例并将图表和plot传递给它来实现。

```
// 移动一个饼图切片: 使用"pieChart" 图表, 和"default" plot
var slice = new MoveSlice(pieChart, "default");

// 放大一个 marker: 使用 "chart" 图表, and "default" plot
var magnify = new Magnify(chart, "default");});
```
像Tooltip插件，MoveSlice 和 Magnify 插件必须在图表调用渲染方法之前分配给图表。

### Highlight
当鼠标移到区域时，高亮插件改变该区域的颜色。

```
// 高光一个区域: 使用"chart" 图表, 和"default" plot
var highlight = new Highlight(chart, "default");

```

### 使用 Legend、 Tooltips 和 Magnify的月销售额
让我们给一个折线表添加Legend、 Tooltips 和 Magnify：

```
<link rel="stylesheet" href="/path/to/dijit/themes/claro/claro.css" media="screen">
<script>
require([
     // Require 基础图表类
    "dojox/charting/Chart",

    // Require 选择的主题
    "dojox/charting/themes/Claro",

    //     We want to plot Lines
    "dojox/charting/plot2d/Lines",

    // 加载 Legend, Tooltip, and Magnify classes
    "dojox/charting/widget/Legend",
    "dojox/charting/action2d/Tooltip",
    "dojox/charting/action2d/Magnify",

    //    使用 Markers
    "dojox/charting/plot2d/Markers",

    //    使用默认 x/y 坐标轴
    "dojox/charting/axis2d/Default",

    // 等待DOM加载
    "dojo/domReady!"
], function(Chart, theme, LinesPlot, Legend, Tooltip, Magnify) {

    // 定义 data
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];
    var chartData2 = [3000,12000,17733,9876,12783,12899,13888,13277,14299,12345,12345,15763];
    var chartData3 = [3000,12000,17733,9876,12783,12899,13888,13277,14299,12345,12345,15763].reverse();

    // 在节点中创建图表
    var chart = new Chart("chartNode");

    // 设置主题
    chart.setTheme(theme);

    // Add the only/default plot
    chart.addPlot("default", {
        type: LinesPlot,
        markers: true
    });

    // 添加坐标轴
    chart.addAxis("x");
    chart.addAxis("y", { min: 5000, max: 30000, vertical: true, fixLower: "major", fixUpper: "major" });

    // 添加 series of data
    chart.addSeries("Monthly Sales - 2010",chartData);
    chart.addSeries("Monthly Sales - 2009",chartData2);
    chart.addSeries("Monthly Sales - 2008",chartData3);

    // 创建 tooltip
    var tip = new Tooltip(chart,"default");

    // 创建 magnifier
    var mag = new Magnify(chart,"default");

    // 渲染图表！
    chart.render();

    // 创建 legend
    var legend = new Legend({ chart: chart }, "legend");
});</script>

<div id="chartNode" style="width:800px;height:400px;"></div>
<div id="legend"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/monthly-sales-legend.html)

我们使用的Tooltips是基于Dijit Tooltip的，因此我们需要确保加载一个Dijit主题样式。在这个例子中我们加载了Claro。

### 使用MoveSlice的月销售额饼图
用MoveSlice来让你的饼图添点活力：

```
<link rel="stylesheet" href="/path/to/dijit/themes/claro/claro.css" media="screen">
<script>
require([
     // Require 基础图表类
    "dojox/charting/Chart",

    // Require 我们选择的主题
    "dojox/charting/themes/Claro",

    // Charting plugins:

    //     We want to plot a Pie chart
    "dojox/charting/plot2d/Pie",

    // 获取Legend, Tooltip, 和 MoveSlice 类
    "dojox/charting/action2d/Tooltip",
    "dojox/charting/action2d/MoveSlice",

    //    使用的 Markers
    "dojox/charting/plot2d/Markers",

    //    使用 默认 x/y 坐标轴
    "dojox/charting/axis2d/Default",

    // 等待DOM加载
    "dojo/domReady!"
], function(Chart, theme, Pie, Tooltip, MoveSlice) {

    // 定义 data
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];

    // Create the chart within it's "holding" node
    var chart = new Chart("chartNode");

    // 设置 theme
    chart.setTheme(theme);

    // 添加 only/default plot
    chart.addPlot("default", {
        type: Pie,
        markers: true,
        radius:170
    });

    // 添加 axes
    chart.addAxis("x");
    chart.addAxis("y", { min: 5000, max: 30000, vertical: true, fixLower: "major", fixUpper: "major" });

    // 添加 series of data
    chart.addSeries("Monthly Sales - 2010",chartData);

    // 创建 tooltip
    var tip = new Tooltip(chart,"default");

    // 创建 slice mover
    var mag = new MoveSlice(chart,"default");

    // 渲染图表！
    chart.render();

});
</script>

<div id="chartNode" style="width:800px;height:400px;"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/monthly-sales-moveslice.html)

### 使用Highlights的月销售额
高亮插件用在Columns图表看起来很棒：

```
<script>
require([
     // Require the basic chart class
    "dojox/charting/Chart",

    // Require the theme of our choosing
    "dojox/charting/themes/MiamiNice",

    //     We want to plot Columns
    "dojox/charting/plot2d/Columns",

    // Require the highlighter
    "dojox/charting/action2d/Highlight",

    //    We want to use Markers
    "dojox/charting/plot2d/Markers",

    //    We&#x27;ll use default x/y axes
    "dojox/charting/axis2d/Default",

    // Wait until the DOM is ready
    "dojo/domReady!"
], function(Chart, theme, ColumnsPlot, Highlight) {

    // Define the data
    var chartData = [10000,9200,11811,12000,7662,13887,14200,12222,12000,10009,11288,12099];

    // Create the chart within it&#x27;s "holding" node
    var chart = new Chart("chartNode");

    // Set the theme
    chart.setTheme(theme);

    // Add the only/default plot
    chart.addPlot("default", {
        type: ColumnsPlot,
        markers: true,
        gap: 5
    });

    // Add axes
    chart.addAxis("x");
    chart.addAxis("y", { vertical: true, fixLower: "major", fixUpper: "major" });

    // Add the series of data
    chart.addSeries("Monthly Sales",chartData);

    // 高亮！
    new Highlight(chart,"default");

    // Render the chart!
    chart.render();
});</script>
<div id="chartNode" style="width:800px;height:400px;"></div>
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/charting/demo/monthly-sales-highlight.html)

##小结
 Dojo Toolkit提供了一个完整的图表库，能够创建优雅、引人注目的各种类型的动态图表。相比其他图表库，`dojox/charting `在灵活性、功能性和可扩展性上是无与伦比的。不要将你的数据以无聊的方式提供给用户，用`dojox/charting `来为你的数据盛装打扮吧。

##dojox/charting 资源
更多关于Dojo图表中库的细节请查看以下资源：

 - [Advanced Charting with Dojo](https://dojotoolkit.org/documentation/tutorials/1.10/charting_advanced/)
 - [dojox/charting 参考指南](https://dojotoolkit.org/reference-guide/1.10/dojox/charting.html)
 - [Theme Preview](http://archive.dojotoolkit.org/nightly/dojotoolkit/dojox/charting/tests/test_themes.html)
 - [Dive Into Dojo Charting Again](http://www.sitepen.com/blog/2012/11/09/dive-into-dojo-charting-again/)
 - [Dojo Charting: Dive Into Theming](http://www.sitepen.com/blog/2012/11/09/dojo-charting-dive-into-theming/)


