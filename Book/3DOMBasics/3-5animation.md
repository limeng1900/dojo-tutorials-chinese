# 3.6动画

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/animation/index.html

----------

本教程中，你将学习如何使用Dojo来创建和组合页面元素的自定义动画。

## 入门
和所有的图形用户界面一样，Web界面也需要保持这样的幻觉，就是所有的像素和按钮都是和我们可以操控的真实事物相连的。在这种幻觉下，我们的大脑可以高效的处理数字和虚拟体验。当转变过程太过生硬时，这种幻觉就会被打破。动画过渡让界面看起来更加自然和直观，可以用来巧妙或不那么巧妙的为页面的变化吸引注意了。
或者换句话来说：你需要更多的牛铃！
本教程我们将更多地了解Dojo提供的动画工具，你可以用它调整和创建自定义动画来满足你特殊的界面需求。

##特效回顾
在之前的教程里我们已经讨论过一些内置、常用的[Dojo特效](https://dojotoolkit.org/documentation/tutorials/1.10/effects/)。可以使用`baseFx.fadeIn ` 和`baseFx.fadeOut`（都出自`dojo/_base/fx`模块）来实现元素的淡入淡出还有来自`dojo/fx`模块的 `fx.slideTo` 和`fx.wipeIn`。已经了解如何给这些函数传递一个带有节点属性的参数对象，来指示在哪出现动画：

```
require(["dojo/fx", "dojo/dom", "dojo/domReady!"], function(fx, dom) {
    fx.wipeIn({
        node: dom.byId("wipeTarget")
    }).play();
});
```
但是元素有许许多多的属性和值可以用来做动画。假如我们想要背景闪光或者让节点在屏幕上移动，我们需要Dojo的通用动画功能`baseFx.animateProperty`。

##动画属性
如果查看`fx.wipeIn`的源码，你回发现节点的style.height属性从0变到自动或者自然高度。我们通过节点的边框动画来看如何创建任意属性的动画，以下是用到的HTML：

```
<button id="startButton">Grow Borders</button>
<button id="reverseButton">Shrink Borders</button>

<div id="anim8target" class="box" style="border-style:outset">
    <div class="innerBox">A box</div>
</div>
```
`animateProperty`方法遵循我们使用过的模式。如下调用animateProperty来指定`border-wodth`属性动画：

```
require(["dojo/_base/fx", "dojo/dom", "dojo/domReady!"], function(baseFx, dom) {
    baseFx.animateProperty({
        node: dom.byId("anim8target"),
        properties: { borderWidth: 100 }
    }).play();
});
```
注意我们使用JavaScript的lower camelCase属性名称`borderWidth`，不是带连字符的CSS属性名`border-width`。我们还是用的节点属性，不过这次用的是新的‘`properties`’关键字来指定。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/animation/demo/animateBorder.html)
所有的适用的属性都有数值，你可以任意指定。在例子中，将同时进行top、left、opacity的动画。通过为每一项指定`start`和`end`属性，可以制作非常详细、可重复的动画。

```
baseFx.animateProperty({
    node: anim8target,
    properties: {
        top: { start: 25, end: 150 },
        left: 0,
        opacity: { start: 1, end: 0 }
    },
    duration: 800
}).play();
```
注意我们也提供一个`duration`属性，它是整个动画消耗的毫秒数。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/animation/demo/animateProperties.html)

##Easing
如果用动画生成的值来绘线，可以看到从开始到结束的值是一个曲线。这条曲线的形状称为“easing”。最简单的曲线式直线，例如，节点匀速从X:0移动到Y:100。但是如果节点开始很慢，加速然后再减速到结束，看起来会比较自然。很多地方这都是默认的，不过Dojo提供大范围的easing函数来获得正确的效果和感觉。`dojo/fx/easing`模块有几个可以加载的easing曲线：

```
require(["dojo/_base/fx", "dojo/dom", "dojo/fx/easing", "dojo/window", "dojo/on", "dojo/domReady!"], function(baseFx, dom, easing, win, on) {
    var dropButton = dom.byId("dropButton"),
        ariseSirButton = dom.byId("ariseSirButton"),
        anim8target = dom.byId("anim8target");

    // Set up a couple of click handlers to run our animations
    on(dropButton, "click", function(evt){
        // get the dimensions of our viewport
        var viewport = win.getBox(win.doc);
        baseFx.animateProperty({
            // use the bounceOut easing routine to have the box accelerate
            // and then bounce back a little before stopping
            easing: easing.bounceOut,
            duration: 500,
            node: anim8target,
            properties: {
                // calculate the 'floor'
                // and subtract the height of the node to get the distance from top we need
                top: { start: 0, end:viewport.h - anim8target.offsetHeight }
            }
        }).play();
    });
    on(ariseSirButton, "click", function(evt){
        baseFx.animateProperty({
            node: anim8target,
            properties: { top: 0 }
        }).play();
    });
});
```
在这个例子中，我们计算计算窗口高度以便于将盒子放在底部。它用bounceOut easing函数达到底部然后在设为最终值之前轻微弹起。注意top属性是一个带有`start`和`end`属性的对象，这让每一个属性的动画都在特定的范围内。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/animation/demo/easing.html)

大多数的easing名称都带有“In”或者“Out”或者“InOut”。名字表示easing特效加在开始（In）、结束（Out）或者都有（InOut）。更多细节见[the dojo/fx/easing Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojo/fx/easing.html)。

##整合
传统动画软件通常使用时间轴来指定在什么时间段里发生什么变化，同时运动和先后运动都很常见。在前面的特效教程里，Dojo分别提供了一个机制：`fx.combine`和`fx.chain`。下面看如何化零为整。
在这个示例中，设置了两个用来交换的容器。为了更加明显，我们也会闪烁它们的背景。首先是使用的HTML：

```
<button id="swapButton">Swap</button>

<div class="container" id="container">
    <div id="content1" class="contentBox" style="top: 0; left: 0">
        <div class="innerBox">1: Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident.</div>
    </div>
    <div id="content2" class="contentBox" style="top: 0; left: 250px">
        <div class="innerBox">2: Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.</div>
    </div>
</div>
```
跟平时一样，我们加载Dojo，require所需模块，在传给`require`的函数里进行初始化。
```
<script src="//ajax.googleapis.com/ajax/libs/dojo/1.10.4/dojo/dojo.js"
        data-dojo-config="isDebug: true, async: true">
<script>

require(["dojo/_base/fx", "dojo/fx", "dojo/fx/easing", "dojo/dom-style", "dojo/dom", "dojo/on", "dojo/aspect", "dojo/domReady!"], function(baseFx, fx, easing, domStyle, dom, on, aspect) {

    function swapAnim(node1, node2) {
        // create & return animation which swaps the positions of 2 nodes
    }

    var originalOrder = true; // track which order our content nodes are in

    var swapButton = dom.byId("swapButton"),
        c1 = originalOrder ? dom.byId("content1") : dom.byId("content2"),
        c2 = originalOrder ? dom.byId("content2") : dom.byId("content1"),
        container = dom.byId("container");

        // Set up a click handler to run our animations
        on(swapButton, "click", function(evt){
            // pass the content nodes into swapAnim to create the node-swapping effect
            // and chain it with a background-color fade on the container
            // ensure the originalOrder bool gets togged properly for next time
        });
});
</script>
```
用不同部分组成复杂动画是非常有用的。我们已经把动画进行了分解，这样位置交换的代码就可以重复使用了。`swapAnim`函数的实现如下：

```
function swapAnim(node1, node2) {
    var posn1 = parseInt(domStyle.get(node1, "left")),
        posn2 = parseInt(domStyle.get(node2, "left"));

    return moveNodes = fx.combine([
        fx.slideTo({
            duration: 1200,
            node: node2,
            left: posn1
        }),
        fx.slideTo({
            duration: 1200,
            node: node1,
            left: posn2
        })
    ]);
}
```
`slideTo`方法用`left`属性移动各个节点，我们也可以使用`animateProperty`来达到类似的效果。两个动画应该同步进行，因此节点要同时移动。`fx.combine`方法实现了二合一。注意和`animateProperty`和其他Dojo方法一样返回动画对象。在需要的时候调用代码给`play()`。

```
// Set up a click handlers to run our animations
on(swapButton, "click", function(evt){

    // chain the swap nodes animation
    // with another to fade out a background color in our container
    var anim = fx.chain([
        swapAnim(c1, c2),
        baseFx.animateProperty({
            node: container,
            properties: {
                backgroundColor: "#fff"
            }
        }),

    ]);
    // before the animation begins, set initial container background
    aspect.before(anim, "beforeBegin", function(){
        domStyle.set(container, "backgroundColor", "#eee");
    });

    // when the animation ends, toggle the originalOrder
    on(anim, "End", function(n1, n2){
        originalOrder = !originalOrder;
    });

    anim.play();
});
```
这是点击事件处理器调用代码。在`fx.combine`之前，传递给`fx.chain`的数组包含两个分解动画。我们希望它们连续运行，节点移动然后是背景颜色动画。容器的初始背景色设置连接到`beforeBegin`事件上，在`onEnd`期间，需要一些小处理来确保下次点击时节点会交换。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/animation/demo/index.html)

最终代码是灵活合理且容易扩展的。如果想要背景动画在交换的同时进行你该怎么做？如何在内容向右移动时降低透明度？和往常一样，原来最难的一点是在哪里停止。

##小结
Dojo动画工具通常都可以为你提供便利，只是对于自定义转换和其它特效，你需要指定全部的控制。动画可以从简单的部分开始组建，并且提供一组有用的生命周期事件来实现同步改变。
在现实世界中，没有什么会从一个状态直接到另一个状态，所以能够控制运动和视觉变化对于提高用户体验非常重要。在以后的教程中，我们将看到贯穿Dojo Toolkit的相同模式：让简单的事更容易，让困难的事变可能。

