原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/index.html 

---

这篇教程中，你将了解使用Dojo's Deferred实现的基础知识，它是轻松地进行异步活动的一种方式，例如Ajax调用。

##入门
当你第一次听说“Deferred”时，它可能听起来像一个神秘对象，实际上它是异步操作（如Ajax）的有力工具。最简单的形式如一个Deferred等待直到一段时间后执行一个动作；本质上，你推迟该动作直到一个前置动作完成。Ajax就属于这种情况：在服务器成功返回信息之前， 我们不想执行一些动作。关键在于能够等待值的返回。在这篇教程中，我们将结合之前 [Ajax tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/ajax/) 的知识并探索如何使用Deferred来提高异步行为的交互能力。

##dojo/Deferred
Dojo对deferred对象的实现是`dojo/Deferred`（从0.3版本就开始出现），它在Dojo1.8版本进行了重构。现在起在实例化一个`Deferred`、动作或者将引用的回调之后，可以通过传递一个函数给`then`方法进行注册，在`Deferred`完成之后（a success）调用`then`方法。`then`方法也接收第二个参数：在`Deferred`被拒（an error）之后调用一个函数，该函数通常被称为`errback`。我们来看一个示例来帮助消化：

```
require(["dojo/Deferred", "dojo/request", "dojo/_base/array", "dojo/dom-construct", "dojo/dom", "dojo/domReady!"],
    function(Deferred, request, arrayUtil, domConstruct, dom) {

        // Create a deferred and get the user list
        var deferred = new Deferred(),
            userlist = dom.byId("userlist");

        // Set up the callback and errback for the deferred
        deferred.then(function(res){
            arrayUtil.forEach(res, function(user){
                domConstruct.create("li", {
                    id: user.id,
                    innerHTML: user.username + ": " + user.name
                }, userlist);
            });
        },function(err){
            domConstruct.create("li", {
                innerHTML: "Error: " + err
            }, userlist);
        });

        // Send an HTTP request
        request.get("users.json", {
            handleAs: "json"}).then(
            function(response){
                // Resolve when content is received
                deferred.resolve(response);
            },
            function(error){
                // Reject on error
                deferred.reject(error);
            }
        );
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/demo/deferred.html)

在示例中，我们的创建一个`Deferred`并且注册一个回调和一个errback。我们也调用`request.get`——一个异步操作，来获取“users.json”。如果取回成功，它解决这个deferred并调用回调函数；如果取回失败，它将拒绝deferred并调用errback。

你可能会问自己，“我真的每次都要这么用`dojo/Deferred`么？”答案是“No”。Dojo所有的Ajax方法都返回`dojo/promise/Promise`，成功时回调，被拒时error：

```
require(["dojo/request", "dojo/_base/array", "dojo/dom-construct", "dojo/dom", "dojo/domReady!"],
    function(request, arrayUtil, domConstruct, dom) {

        var deferred = request.get("users.json", {
            handleAs: "json"
        });

        deferred.then(function(res){
            var userlist = dom.byId("userlist");

            arrayUtil.forEach(res, function(user){
                domConstruct.create("li", {
                    id: user.id,
                    innerHTML: user.username + ": " + user.name
                }, userlist);
            });
        },function(err){
            // This shouldn't occur, but it's defined just in case
            alert("An error occurred: " + err);
        });

});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/demo/xhr.html)

我们用`then`来注册一个回调，如果Ajax调用成功，Deferred被解决，通常传递给`load`函数的第一个参数会传递给回调函数。如果Ajax调用失败，Deferred被拒，error会传递给errback。

在回调函数中，我们遍历服务器返回的users并且为每一个创建一个列表项，就像我们设置`load`属性。然而，使用一个Deferred，我们可以从动作中分离获取数据的取回动作（Ajax调用）。对动作的分离正是Deferred强大的一面。

##链式
虽然一旦你熟悉Deferred就会发现它是一个相当简单的概念，但`dojo/Deferred`还包含着一些强力的特性。其中之一就是链式：`then`调用动作的结果就像一个新的Deferred，而不是回调返回的值。初看会很容易困惑，所以先看个例子。

比方说服务器不在返回users对象，而是返回每一个user值组成的list。这并不是很实用，所以我们注册一个回调来将这个list转换成更有用的东西。针对第一个`then`的结果注册的每个连续回调都有可用的users list传递给它。

```
require(["dojo/request", "dojo/_base/array", "dojo/json", "dojo/dom-construct", "dojo/dom", "dojo/domReady!"],
    function(request, arrayUtil, JSON, domConstruct, dom) {

        var original = request.get("users-mangled.json", {
            handleAs: "json"
        });

        var result = original.then(function(res){
            var userlist = dom.byId("userlist1");

            return arrayUtil.map(res, function(user){
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

        // Our result object has a `then` method that accepts a callback,
        // like our original object -- but the value handed to the callback
        // we're registering here is *NOT* the data from the Ajax call,
        // but the return value from the callback above!
        result.then(function(objs){
            var userlist = dom.byId("userlist2");

            arrayUtil.forEach(objs, function(user){
                domConstruct.create("li", {
                    innerHTML: JSON.stringify(user)
                }, userlist);
            });
        });
});
```
`then`方法的返回值指定为一个promise，它实现一个特定API。你可以前往[the promises tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/promises/) 了解更多信息，不过现在，只要知道一个promise提供一个`then`方法，该方法和Deferred的`then`完全一致。

重点要注意：原始Deferred不受链式影响，并且如果原始Deferred注册了一个回调，服务器的list仍然完好。

```
original.then(function(res){
    var userlist = dom.byId("userlist3");

    arrayUtil.forEach(res, function(user){
        domConstruct.create("li", {
            innerHTML: JSON.stringify(user)
        }, userlist);
    });
});
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/demo/chaining.html)

这个例子很随意，但是链式可以用来为你应用的消耗修改数据。例子里还可以这么做：

```
require(["dojo/request", "dojo/_base/array", "dojo/dom-construct", "dojo/dom", "dojo/domReady!"],
    function(request, arrayUtil, domConstruct, dom) {

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

        getUserList().then(function(users){
            var userlist = dom.byId("userlist");
            arrayUtil.forEach(users, function(user){
                domConstruct.create("li", {
                    id: user.id,
                    innerHTML: user.username + ": " + user.name
                }, userlist);
            });
        });
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/deferreds/demo/chaining-practical.html)

现在任何使用`getUserList`的代码都将得到一个user对象列表。

##Deferred 列表
有时你需要并行从多个源取回数据，还想在请求完成时收到通知。你可以设置某种叫做Deferred系统的Deferred，一项项返回，不过就像本教程第一个例子，你不用手动来做。Dojo1.8之前，用`Dojo/DeferredList`处理。1.8之后用`dojo/promise/all`和`dojo/promise/first`来处理，这将在 [promises tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/promises)教程中涉及。

##小结
由于大部分JavaScript应用使用Ajax，就需要一个简单而优雅的注册操作，这正是`dojo/Deferred`所提供的。链式则让它变得更加简单。

##资源

 - [dojo/Deferred Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojo/Deferred.html)
 - [dojo/Deferred API](https://dojotoolkit.org/api/1.10/dojo/Deferred.html)
 - [Ajax with dojo/request Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/ajax)
 - [Dojo Deferreds and Promises Tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/promises)
 - [Future and Promises](http://en.wikipedia.org/wiki/Futures_and_promises) Wikipedia article
