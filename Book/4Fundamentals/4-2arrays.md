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
