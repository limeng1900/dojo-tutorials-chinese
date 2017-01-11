# 3.5 Dojo特效

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/effects/index.html

----------

本教程中，我们将探索Dojo提供的特效，它可以让你的页面或应用生动起来。

## 入门
现在为止，我们已经可以很轻松地通过DOM节点来操作DOM和处理事件。但是，在我们执行某些动作时，过渡显得非常生硬：移除一个节点让它突然从页面消失，可能会让用户迷惑。使用Dojo提供的标准特效，我们可以创建更加平滑的用户体验，为应用添加额外的闪光点。如果我们利用Dojo的`dojo/_base/fx`和`dojo/fx`可以对这些特效进行连接和组合，进而提供一些真正炫酷的动态体验。

> Dojo1.10有两个`fx`模块：`dojo/_base/fx` 和 `dojo/fx`。
    > `dojo/_base/fx`提供基础特效方法，之前在Dojo base里看到过，包括：`animateProperty`，`anim`，`fadeIn`和`fadeOut`。
    > `dojo/fx`提供更多高阶特效，包括：`chain`，`combine`，`wipeIn`，`wipeOut` 和`slideTo`。


##淡入淡出
应用中你可能见过或者用过淡入和淡出。这个特效很常见，它包含在`dojo/_base/fx`的核心特效部分。可以用它来平滑的显示或隐藏页面中的元素。请看示例：

```
<button id="fadeOutButton">Fade block out</button>
<button id="fadeInButton">Fade block in</button>

<div id="fadeTarget" class="red-block">
    A red block
</div>
<script>
    require(["dojo/_base/fx", "dojo/on", "dojo/dom", "dojo/domReady!"], function(fx, on, dom) {
        var fadeOutButton = dom.byId("fadeOutButton"),
            fadeInButton = dom.byId("fadeInButton"),
            fadeTarget = dom.byId("fadeTarget");

        on(fadeOutButton, "click", function(evt){
            fx.fadeOut({ node: fadeTarget }).play();
        });
        on(fadeInButton, "click", function(evt){
            fx.fadeIn({ node: fadeTarget }).play();
        });
    });
</script>
```
所有的动画函数都需要一个参数：一个带属性的对象。你要用的最重要的属性是`node`属性：一个DOM节点或一个节点ID字符串。另一个属性是`duration`——动画的持续时间，以毫秒为单位，默认为350毫秒。其他动画还有别的属性，不过淡入淡出不需要。

动画函数返回一个[`dojo/_base/fx::Animation`](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/fx.html#dojo-base-fx-animation)对象，它有几个方法：paly、pause、stop和gotoPercent。如上，动画在创建时不会立即播放必须用play方法播放。

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/effects/demo/fade.html)

##Wiping
另一个动画是擦拭：改变一个节点的高度。看起来像是在节点上使用雨刷器。通常，它用来创建页面下拉特效，用在较少访问的内容部分，或者你想要收缩的地方。

```
<button id="wipeOutButton">Wipe block out</button>
<button id="wipeInButton">Wipe block in</button>

<div id="wipeTarget" class="red-block wipe">
    A red block
</div>
<script>
    require(["dojo/fx", "dojo/on", "dojo/dom", "dojo/domReady!"], function(fx, on, dom) {
        var wipeOutButton = dom.byId("wipeOutButton"),
            wipeInButton = dom.byId("wipeInButton"),
            wipeTarget = dom.byId("wipeTarget");

        on(wipeOutButton, "click", function(evt){
            fx.wipeOut({ node: wipeTarget }).play();
        });
        on(wipeInButton, "click", function(evt){
            fx.wipeIn({ node: wipeTarget }).play();
        });
    });
</script>
```
擦拭特效位于`dojo/fx`模块。本例中，将“wipe”类添加到目标节点。由于wipe函数操作节点内容的高度，“wipe”类将目标节点的高度设为“auto”。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/effects/demo/wipe.html)

##滑动
目前已经涉及了隐藏节点，但是如果要移动它们呢?淡入淡出和擦拭都不能改变节点的位置。可以用`fx.slideTo`。节点的位移可以用来创建页面上的运动，`fx.slideTo`创建一个节点的平滑动画，通过指定左上角的坐标来移动它。

```
<button id="slideAwayButton">Slide block away</button>
<button id="slideBackButton">Slide block back</button>

<div id="slideTarget" class="red-block slide">
    A red block
</div>
<script>
    require(["dojo/fx", "dojo/on", "dojo/dom", "dojo/domReady!"], function(fx, on, dom) {
        var slideAwayButton = dom.byId("slideAwayButton"),
            slideBackButton = dom.byId("slideBackButton"),
            slideTarget = dom.byId("slideTarget");

        on(slideAwayButton, "click", function(evt){
            fx.slideTo({ node: slideTarget, left: "200", top: "200" }).play();
        });
        on(slideBackButton, "click", function(evt){
            fx.slideTo({ node: slideTarget, left: "0", top: "100" }).play();
        });
    });
</script>
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/effects/demo/slide.html)

##动画事件
之前说过所有的动画方法都返回一个`dojo/_base/fx::Animation`对象。这些对象不只用来控制动画的播放和暂停，为了控制动作的执行顺序如之前、期间、之后，这些对象还可以设置监听事件。两个常用的重要事件处理器是`beforeBegin` 和`onEnd`。

```
<button id="slideAwayButton">Slide block away</button>
<button id="slideBackButton">Slide block back</button>

<div id="slideTarget" class="red-block slide">
    A red block
</div>
<script>
    require(["dojo/fx", "dojo/on", "dojo/dom-style", "dojo/dom", "dojo/domReady!"], function(fx, on, style, dom) {

        var slideAwayButton = dom.byId("slideAwayButton"),
            slideBackButton = dom.byId("slideBackButton"),
            slideTarget = dom.byId("slideTarget");

            on(slideAwayButton, "click", function(evt){
                // 注意我们指定了beforeBegin作为动画的一个属性，而没有使用connect
                // 这确保我们的beforeBegin处理器会优先执行
                var anim = fx.slideTo({
                    node: slideTarget,
                    left: "200",
                    top: "200",
                    beforeBegin: function(){

                        console.warn("slide target is: ", slideTarget);

                        style.set(slideTarget, {
                            left: "0px",
                            top: "100px"
                        });
                    }
                });

                //除了beforeBegin，我们也可以指定onEnd ，不过他和这样的连接一样容易
                on(anim, "End", function(){
                    style.set(slideTarget, {
                        backgroundColor: "blue"
                    });
                }, true);

                // 不要忘了启动动画
                anim.play();
            });

            on(slideBackButton, "click", function(evt){
                var anim = fx.slideTo({
                    node: slideTarget,
                    left: "0",
                    top: "100",
                    beforeBegin: function(){

                        style.set(slideTarget, {
                            left: "200px",
                            top: "200px"
                        });
                    }
                });

                on(anim, "End", function(){
                    style.set(slideTarget, {
                        backgroundColor: "red"
                    });
                }, true);

                anim.play();
            });
    });
</script>
```
`beforeBegin`作为一个参数对象的属性传递。直接原因是确保在动画创建的时候就连接`beforeBegin`。因此，如果你在动画创建之后连接`beforeBegin`，你的处理器将在动画的`beforeBegin`处理器之后执行，你可能并不想这样。通过将处理器作为参数对象属性传递，可以确保你的处理器最先执行。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/effects/demo/connect.html)


##连接
如果我们想要连续播放动画呢？可以使用之前说过的`end`事件来开启下一个特效，但是不太方便。`dojo/fx`提供了几个方法设置特效顺序播放或并行，每个方法都返回一个`dojo/_base/fx::Animation`对象，它带有一组事件和代表整个序列的方法。先看`fx.chain`来一个接一个播放动画：

```
<button id="slideAwayButton">Slide block away</button>
<button id="slideBackButton">Slide block back</button>

<div id="slideTarget" class="red-block slide chain">
    A red block
</div>
<script>
    require(["dojo/_base/fx", "dojo/fx", "dojo/on", "dojo/dom", "dojo/domReady!"], function(baseFx, fx, on, dom) {

        var slideAwayButton = dom.byId("slideAwayButton"),
            slideBackButton = dom.byId("slideBackButton"),
            slideTarget = dom.byId("slideTarget");

        // Set up a couple of click handlers to run our chained animations
        on(slideAwayButton, "click", function(evt){
            fx.chain([
                baseFx.fadeIn({ node: slideTarget }),
                fx.slideTo({ node: slideTarget, left: "200", top: "200" }),
                baseFx.fadeOut({ node: slideTarget })
            ]).play();
        });
        on(slideBackButton, "click", function(evt){
            fx.chain([
                baseFx.fadeIn({ node: slideTarget }),
                fx.slideTo({ node: slideTarget, left: "0", top: "100" }),
                baseFx.fadeOut({ node: slideTarget })
            ]).play();
        });
    });
</script>
```
如你所见，直接在`fx.chain`的回调里创建了一些特效，然后立即调用返回的`dojo/_base/fx::Animation`的`play`来开启链式动画。我们不对每一个独立特效启动播放，`fx.chain`会进行处理。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/effects/demo/chain.html)

##组合
`dojo/fx`提供的第二个方便的方法是`combine`，可以同时开启多个动画。在持续最长的动画结束后，返回的` dojo/_base/fx::Animation`对象会启动`onEnd`事件。来看另一个例子：

```
<button id="slideAwayButton">Slide block away</button>
<button id="slideBackButton">Slide block back</button>

<div id="slideTarget" class="red-block slide chain">
    A red block
</div>
<script>
    require(["dojo/_base/fx", "dojo/fx", "dojo/on", "dojo/dom", "dojo/domReady!"], function(baseFx, fx, on, dom) {

        var slideAwayButton = dom.byId("slideAwayButton"),
            slideBackButton = dom.byId("slideBackButton"),
            slideTarget = dom.byId("slideTarget");

        // Set up a couple of click handlers to run our combined animations
        on(slideAwayButton, "click", function(evt){
            fx.combine([
                baseFx.fadeIn({ node: slideTarget }),
                fx.slideTo({ node: slideTarget, left: "200", top: "200" })
            ]).play();
        });
        on(slideBackButton, "click", function(evt){
            fx.combine([
                fx.slideTo({ node: slideTarget, left: "0", top: "100" }),
                baseFx.fadeOut({ node: slideTarget })
            ]).play();
        });
    });
</script>
```
这个例子中，同时进行滑动和淡入淡出。

你可以使用`fx.chain`和`fx.combine`来创建一些精致的动画序列。而且，连接和组合都返回一个动画对象，这些方法还可以进行连接和组合，你就可以用简单的动画来创建更加丰富多彩的序列。

> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/effects/demo/combine.html)

##小结
使用Dojo可以很容易地为你的页面增色。 [`dojo/_base/fx`](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/fx.html) 和 [`dojo/fx`](https://dojotoolkit.org/reference-guide/1.10/dojo/fx.html)可以让你轻易的实现DOM节点的淡入淡出、擦除等，链接和组合意味着你可以更快更容易地创建高阶动画。

然而，如果你想要做一些更有难度的事，比如调整一个DOM节点的高但并不一定缩减到零，或者通过动画改变背景颜色？下一节教程[`fx.animateProperty`](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/fx.html#id8)会涉及到这些。

