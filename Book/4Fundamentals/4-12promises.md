# 4.12 Dojo Deferreds 和 Promises

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/promises/index.html

----------

Deferred是一个绝妙又给力的东西，但是它其实是更棒的promis的实现。在这篇教程中，我们将讨论这个，还有一些额外的Dojo的API，它们以统一的方式使用promise和正则值。

##入门
现在你已经了解了[`dojo/request`](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/)、[`dojo/Deferred`](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/) 以及这些API相关的概念，我们打算给你来点更抽象的：promises。一个promise是代表一个操作完成的最终返回值的对象。`dojo/promise`API在1.8版本有了一次重大升级和重构，promise有以下特点：

 - 可以为三个状态中的一个：unfulfilled、resolved、 rejected
 - 可以从unfulfilled转变到resolved或者rejected
 - 实现了一个`then`方法来针对状态改变的通知注册回调
 - 回调不能改变promise产生的值
 - promise的`then`方法返回一个新的promise，在保持原promise的值不变的情况下提供链式操作

了解了这些，我们再来看Dojo如何实现promise。

##Deferred as a Promise
如果一个promise听起来很像一个`Deferred`，那你就要注意。实际上，`dojo/Deferred`模块是promise在Dojo的主要实现方式。让我们看一看最后一个教程中链式示例，在这里`Deferred`是一个promise：

```
require(["dojo/request"],
    function(request){

        // original is a Deferred
        var original = request.get("users-mangled.json", {
            handleAs: "json"
        });

});
```
我们之前提过，`request.get`（和Dojo的所有Ajax辅助）返回一个`promise`。你可以认为这个`promise`代表向服务器请求完成的最终返回值。首先它是一个unfulfilled（未实现）状态，随后会根据服务器返回的结果变成resolved（已解决）或者rejected（被拒绝）。

 对于请求调用返回的结果promise，我们可以通过`then`方法注册回调。不过，我们没有完全覆盖`then`的返回值，只是返回的值还有一个`then`方法。你可能想过它会返回原始的promise，但是它只是返回一个实现promise API的简单对象。这个对象常用的两个方法是`then`和`cancel`（表明不在需要结果）。来看下：
 

```
require(["dojo/_base/array", "dojo/dom", "dojo/dom-construct", "dojo/json"],
    function(arrayUtil, dom, domConstruct, JSON){

        // result is a new promise that produces a
        // new value
        var result = original.then(function(response){
            var userlist = dom.byId("userlist1");

            return arrayUtil.map(response, function(user){
                domConstruct.create("li", {
                    innerHTML: JSON.stringify(user)
                }, userlist);

                return {
                    id: user[0],
                    username: user[1],
                    name: user[2]
            };
        });
    });
});
```
`then`的调用产生一个promise对象，它的值将设为回调函数的返回值。我们可以看到这个新的promise产生的值不同于原来通过调用promise的`then`方法的`Deferred`。

```
// chaining to the result promise rather than
// the original deferred to get our new value
result.then(function(objs){
    var userlist = dom.byId("userlist2");

    arrayUtil.forEach(objs, function(user){
        domConstruct.create("li", {
            innerHTML: JSON.stringify(user)
        }, userlist);
    });
});
```
> `then`返回的promise值**总是**回调的返回值，如果你不返回一个值，promise的值将会是`undefined`。如果你在你的作用链里某处看到一个随机`undefined`，检查你的回调确保它提供了正确的返回值。如果你不关心作用链，就不用担心返回什么值了。

我们也可以查看原`Deferred`的值没有改变：

```
// creating a list to show that the original
// deferred's value was untouched
original.then(function(response){
    var userlist = dom.byId("userlist3");

    arrayUtil.forEach(response, function(user){
        domConstruct.create("li", {
            innerHTML: JSON.stringify(user)
        }, userlist);
    });
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/promises/demo/chaining.html)

如你所见，作用链很强大；当你知道链上的每一环都是不可变的，它就更棒了。

也要注意`Deferred`包含里一个重要属性：`promise`。这是一个只实现promise API的对象，它表示`Deferred`将产生的值。`promise`属性可以防止某人意外（或出于某些目的）调用`resolve`或`reject`来最小化来自你API用户的副作用，但是仍然允许他们获取原始`Deferred`的值。