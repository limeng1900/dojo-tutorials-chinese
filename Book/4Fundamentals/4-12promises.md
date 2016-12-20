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

##dojo/when
现在我们已经理解什么是promise以及它有什么用，我们再来谈谈`dojo/when`。它是一个Dojo提供的强大功能，让你可以用一致的API来处理promise和标准值。

`dojo/when`函数接收4个参数：一个promise或值，一个回调（可选），一个错误处理（可选）和一个progress处理器（可选）。它分两种用法：

 - 如果第一个参数不是一个promise并且提供回调，回调将在提供第一个参数之后立即被调用，并返回回调的结果。如果没有提供回调，直接返回第一个参数。
 - 如果第一个参数是一个promise，回调、错误处理和progress处理都将传递给promise的`then`方法，并将你的回调设置为当promise准备好时执行，返回结果promise。

让我们再看下[Deferred tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/)教程的`getUserList`函数：

```
function getUserList(){
    return request.get("users-mangled.json", {
        handleAs: "json"
    }).then(function(response){
        return arrayUtil.map(response, function(user){
            return {
                id: user[0],
                username: user[1],
                name: user[2]
            };
        });
    });
}
```
比如说users列表不会经常改变，并且可以缓存在客户端而不用每次调用时都要获取。在这个例子中，因为`dojo/when`接收一个只或promise，可以改变`getUserList`来返回一个promise或users数组，随后我们可以用`dojo/when`处理返回值：

```
require(["dojo/_base/array", "dojo/when", "dojo/request",
        "dojo/dom", "dojo/dom-construct", "dojo/json"],
    function(arrayUtil, when, request, dom, domConstruct, JSON){
        var getUserList = (function(){
            var users;
            return function(){
                if(!users){
                    return request.get("users-mangled.json", {
                        handleAs: "json"
                    }).then(function(response){
                        // Save the resulting array into the users variable
                        users = arrayUtil.map(response, function(user){
                            return {
                                id: user[0],
                                username: user[1],
                                name: user[2]
                            };
                        });

                        // Make sure to return users here,
                        // for valid chaining
                        return users;
                    });
            }
            return users;
        };
    })();

    when(getUserList(), function(users){
        // This callback will be run after the request completes

        var userlist = dom.byId("userlist1");
        arrayUtil.forEach(users, function(user){
            domConstruct.create("li", {
                innerHTML: JSON.stringify(user)
            }, userlist);
        });

        when(getUserList(), function(user){
            // This callback will run right away since it's already in cache

            var userlist = dom.byId("userlist2");
            arrayUtil.forEach(users, function(user){
                domConstruct.create("li", {
                    innerHTML: JSON.stringify(user)
                }, userlist);
            });
        });
    });
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/promises/demo/when.html)

也可能你主管创建user列表的API，想要一个干净的API来让你的开发人员从服务器（通过一个Deferred）或者数组传递给你一个users列表。在这个例子中，你可以提出一个像下面这样的函数：

```
function createUserList(node, users){
    var nodeRef = dom.byId(node);

    return when(
        users,
        function(users){
            arrayUtil.forEach(users, function(user){
                domConstruct.create("li", {
                    innerHTML: JSON.stringify(user)
                }, nodeRef);
            });
        },
        function(error){
            domConstruct.create("li", {
                innerHTML: "Error: " + error
            }, nodeRef);
        }
    );
}

var users = request.get("users-mangled.json", {
    handleAs: "json"
}).then(function(response){
    return arrayUtil.map(response, function(user){
        return {
            id: user[0],
            username: user[1],
            name: user[2]
        };
    });
});

createUserList("userlist1", users);
createUserList("userlist2",
    [{ id: 100, username: "username100", name: "User 100" }]);
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/promises/demo/when-create.html)

如你所见，`doio/when`可是让开发者使用一个API优雅地处理生产者和用户两端的同步和异步用例。

##用dojo/promise/all管理promise列表
`dojo/promise/all`用来代替`dojo/promise/all`，它本质上是将几个promise的结果合并到一个promise来提供一个管理多重异步进程的机制。有时你需要从多个资源并行获取数据，并在全部请求完成后得到通知。你可能设置某些调用Deferred系统统计返回数的Deferred，但是你不用手动实现它。Dojo在这里为你提供了`dojo/promise/all`。

只需要简单地传递一个Deferred对象或数组给`dojo/promise/all`的构造器就可以使用它。同样，结果要么是一个使用和参数相同键的对象，要么是一个跟传递给构造器参数顺序相同的数组。我们来看一个例子：

```
require(["dojo/promise/all", "dojo/Deferred", "dojo/request", "dojo/_base/array", "dojo/dom-construct", "dojo/dom", "dojo/json", "dojo/domReady!"],
function(all, Deferred, request, arrayUtil, domConstruct, dom, JSON){
    var usersDef = request.get("users.json", {
        handleAs: "json"
    }).then(function(response){
        var users = {};

        arrayUtil.forEach(response, function(user){
            users[user.id] = user;
        });

        return users;
    });

    var statusesDef = request.get("statuses.json", {
        handleAs: "json"
    });
    all([usersDef, statusesDef]).then(function(results){
        var users = results[0],
            statuses = results[1],
            statuslist = dom.byId("statuslist");

        if(!results[0] || !results[1]){
            domConstruct.create("li", {
                innerHTML: "An error occurred"
            }, statuslist);
            return;
        }
        arrayUtil.forEach(statuses, function(status){
            var user = users[status.userId];
            domConstruct.create("li", {
                id: status.id,
                innerHTML: user.name + ' said, "' + status.status + '"'
            }, statuslist);
        });
    });
});
```
这里我们想要从服务器得到一个users列表，并且将它和一个状态列表合并。在注册回调之后会返回一个用户hash，我们将Deferred都传递给`dojo/promise/all`并用它注册回调。回调随后就会检查错误，如果没有找到什么，它会遍历这些状态并匹配到用户。无所谓那个请求首先完成，`dojo/promise/all`总会给我们一个按照传递给它的Deferreds的顺序排列的结果。
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/promises/demo/all.html)

##小结
Dojo附加的promise API以两种方式让开发者有机会创建更加强大的应用：避免由于`Deferred`这样的函数因保证promise不变产生的副作用；同时`dojo/when`为基于promise的和基于值的编程之间的鸿沟提供一座桥梁。最重要的是，有了`dojo/promise/all`，你就可以用一个回调来处理多个deferreds/promises。

##资源

 - [dojo/promise Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojo/promise/Promise.html)
 - [dojo/promise/all Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojo/promise/all.html)
 - [dojo/promise/Promise API](https://dojotoolkit.org/api/?qs=1.10/dojo/promise/Promise)
 - [dojo/promise/all API](https://dojotoolkit.org/api/?qs=1.10/dojo/promise/all)
 - [Ajax with dojo.request Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/ajax)
 - [Getting Start with Deferreds Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds)
 - [Future and Promises](http://en.wikipedia.org/wiki/Futures_and_promises) Wikipedia article