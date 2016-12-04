# 4.2数组
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/arrays/index.html

----------

在本教程中，你将了解Dojo针对JavaScript数组的跨平台解决方案：`dojo/_base/array`。

##入门
数据存取和操作是你的web应用开发时非常重要的部分。JavaScript的实现者们了解这些，所以为了更加易用，他们在数组实例上加入了一些方法。可惜，不是所有的浏览器和环境都采取这些新方法。好消息是Dojo为这些数组新方法提供辅助方法，因此无论你在什么环境下运行，你都可以轻易驾驭数组。

##查找
你在用数组的时候会需要的一项操作是找到数组的某一项。Dojo在`dojo/_base/array`资源里提供两个函数：`indexOf` 和 `lastIndexOf`。`indexOf` 方法从数组的最小索引查找到最大索引，`lastIndexOf`则从大到小。它们接收同样的参数：查找的数组、查找的项、开始查找的索引（可选）。来看一些例子：
```
require(["dojo/_base/array"], function(arrayUtil) {
    var arr1 = [1,2,3,4,3,2,1,2,3,4,3,2,1];
    arrayUtil.indexOf(arr1, 2); // returns 1
    arrayUtil.indexOf(arr1, 2, 2); // returns 5
    arrayUtil.lastIndexOf(arr1, 2); // returns 11
});
```
如果没找到该项，两个函数都返回-1。需要注意的是，两个函数都使用严格比较（===），所以你能找到的不仅仅是原始值：
```
var obj1 = { id: 1 },
    arr2 = [{ id: 0 }, obj1, { id: 2 }, { id: 3 }];

// This search returns 1, as obj1 is the second item
// in the array.
arrayUtil.indexOf(arr2, obj1);

// This search returns -1. While the objects may look similar,
// they are entirely different objects, and so this object
// isn't found in the array.
arrayUtil.indexOf(arr2, { id: 1 });
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/demo/searching.html)

##循环
另一个常用操作是循环遍历数组的每一项。通常，会类似下面这样：
```
var item;
for(var i = 0; i &lt; arr.length; i++){
    item = arr[i];
    // do something with item
}
```
上面这样做的缺点是当你在一个事件处理器里访问`item`时，它可能不是你想要的项，而是所有事件处理器数组的最后一项。`forEach`给了我们一个设置循环的标准方式，同时在作用域链查找期间也会保留项目。Array.prototype.forEach有两个异常：

 - 如果你的索引之中存在未定义的索引，它也会进行迭代。就是说如果你的数组里有一个undefined值，它依然会对那个索引执行你的函数，而不能跳过它。
 - 循环是执行在数组本身的。而当在支持的浏览器里使用原生`forEach`方法时，它会遍历数组的一个副本而不是数组本身。你在函数里对原始数组做的任何改变在随后的函数执行的都能看到。

实际上，这两点差异适用于本教程中讨论的所有方法。我们来看一个例子：
```
var arr = ["one", "two", "three", "four"],
    // dom is from dojo/dom
    list1 = dom.byId("list1");

// Skip over index 4, leaving it undefined
arr[5] = "six";

arrayUtil.forEach(arr, function(item, index){
    // This function is called for every item in the array
    if(index == 3){
        // this changes the original array,
        // which changes the item passed to
        // the sixth invocation of this function
        arr[5] = "seven";
    }

    // domConstruct is available at dojo/dom-construct
    domConstruct.create("li", {
        innerHTML: item + " (" + index + ")"
    }, list1);
});
```
`arrayUtil.forEach`方法接收三个参数：一个用来迭代的数组、一个为数组每一项（包含已定义之间存在的未定义索引）调用的函数（或者回调）、一个对象（可选）用作调用回调的作用域。
你提供的回调将调用给数组的每一个索引直到最后分配的索引（包含在内），它接收三个参数：当前索引的对象或值、当前索引本身、要遍历的数组的引用。回调也将在`arrayUtil.forEach`第三个参数的作用域里调用，如果没提供第三个参数则为全局对象（浏览器下为`window`）。来看下作用域参数：

```
var list2 = dom.byId("list2"),
    myObject = {
        prefix: "ITEM: ",
        formatItem: function(item, index){
            return this.prefix + item + " (" + index + ")";
        },
        outputItems: function(arr, node){
            arrayUtil.forEach(arr, function(item, index){
                domConstruct.create("li", {
                    innerHTML: this.formatItem(item, index)
                }, node);
            }, this);
        }
    };

myObject.outputItems(arr, list2);
```
这可能是作用域参数最常用的模式：传递`this`以便于回调函数将在调用它的方法的作用域里调用。当你使用widget时，这个模式非常有用，建议保存起来以备后用。
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/demo/looping.html)

##操作
Dojo让循环变的很简单，不过你就会经常想要获得一个数组的数据，然后用它来做一些事再得到一个新数组。比如我们有一个字符串数组，想要将转换成对象，并将对象的“name”属性设为字符串的值。我们可能这么做：

```
var original = ["one", "two", "three", "four", "five"],
    transformed = [];

arrayUtil.forEach(original, function(item, index){
    transformed.push({
        id: index * 100,
        text: item
    });
}); // [ { id: 0, text: "one" }, { id: 100, text: "two" }, ... ]
```
这没什么错误，不过新版的JavaScript和Dojo有一个相当的功能：`arrayUtil.map`。我们来看看：

```
var mapped = arrayUtil.map(original, function(item, index){
    return {
        id: index * 100,
        text: item
    };
}); // [ { id: 0, text: "one" }, { id: 100, text: "two" }, ... ]
```
`map`的参数和`forEach`一样。不同点在于返回值会被存储在一个新数组里，它的索引和原始数组一样。从`map`返回了新的数组。
Dojo覆盖新版JavaScript的另一个常见转换是`filter`。`filter`的想法是你有一个数组指向选择其中一些符合条件的项。用`forEach`也可以实现，不过`filter`更简单。
参数相比`map`还是没有改变，不过在`filter`里，会评估回调返回的值，如果为真则将该项附加到`filter`返回的数组里。让我们看另一个例子：

```
var filtered = arrayUtil.filter(mapped, function(item, index){
    return item.id &gt; 50 && item.id &lt; 350;
}); // [ { id: 100, text: "two" }, { id: 200, text: "three" },
    //   { id: 300, text: "four" } ]
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/demo/manipulating.html)

##匹配
有时候你想知道一个数组的项是否能匹配一定的条件：或许你想知道一些对象是否有一个error属性，或者你想确认所有的对象有一个text属性。这就是`some`和`every`的用处了。它们的函数签名很像`filter`(包括回调的返回)，不过不是返回一个数组，而是一个布尔值：如果数组每一项的回调都返回`true`则`every`返回`true`，如果数组至少有一项的回调返回`true`则`some`返回`true`。下面的例子应该能解释地更清楚：

```
var arr1 = [1,2,3,4,5],
    arr2 = [1,1,1,1,1];

arrayUtil.every(arr1, function(item){ return item == 1; }); // returns false
arrayUtil.some(arr1, function(item){ return item == 1; });  // returns true

arrayUtil.every(arr2, function(item){ return item == 1; }); // returns true
arrayUtil.some(arr2, function(item){ return item == 1; });  // returns true

arrayUtil.every(arr2, function(item){ return item == 2; }); // returns false
arrayUtil.some(arr2, function(item){ return item == 2; });  // returns false
```
简单的理解就是`every`就像是在`if`声明里用的`&&`，`som`就像是`||`。
> [ View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/demo/matching.html)

##小结
JavaScript规范已经提供了一些强大的数组方法，但不是所有的浏览器和环境都支持。Dojo的`dojo/_base/array`模块在数组的新旧方法之间搭起桥梁，让你可以用更少的代码更快、更高效的做更多的事。

