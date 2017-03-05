# 6.4 实时Store

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/realtime_stores/index.html

---

基于实时store的web应用所提供的时传统web应用不可能提供的，可以在数据变化时就让用户看到。Dojo对象存储接口作为Dojo应用的数据模型基础，它支持实时数据更新。在这篇教程中，我们将了解如何利用通知系统来和实时widget进行交互。

##入门

在这篇教程中我们将在[Dojo object store](https://dojotoolkit.org/documentation/tutorials/1.10/intro_dojo_store/)和[数据模型教程](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/)的基础上继续。在数据模型教程中我们看到了如何为查询结果创建一个视图渲染器，并使用`observe()`方法来监控数据变化。我们可以在store上调用`query()`方法并得到一个查询结果集；然后可以在查询结果集合上调用`forEach`来迭代结果集。我们也可以在结果集上调用`observe()`来监听变化。下面是示例实现：

```
require(["dojo/dom", "dojo/dom-construct"],
    function(dom, domConstruct){
function viewResults(results){
    var container = dom.byId("container");
    var rows = [];

    // functions called within observe callback below
    function addRow(market, i){
        // insert row into DOM, and also into our internal array
        rows.splice(i, 0, domConstruct.create("div", {
            innerHTML: market.name + " index: " + market.index.toFixed(2) + " at: " + market.date.toLocaleTimeString()
        }, container, i));
    }
    function removeRow(i){
        // remove row from DOM and array (splice returns the removed items)
        domConstruct.destroy(rows.splice(i, 1)[0]);
    }

    // add initial items, and handle future changes
    results.forEach(addRow);
    results.observe(function(market, removedFrom, insertedInto){
        // this will be called any time a market is added, removed, or updated
        if(removedFrom &gt; -1){
            removeRow(removedFrom);
        }
        if(insertedInto &gt; -1){
            addRow(market, insertedInto);
        }
    }, true); // we can indicate to be notified of object updates as well
}

var results = marketStore.query({});
viewResults(results);
```

监控store数据的切入点是**observe()**方法，它是查询结果的一个方法。传递给`observe()`的回调有三个参数：

 - **`object`**：被修改的对象
 - **`removedFrom`**：结果集合中之前的要移除或修改的对象的索引。`removedFrom`的-1值表示该对象要添加到结果集。
 - **`insertedInto`**：结果集合中现存的新的或者修改的对象的索引。`insertedInto`的-1值表示对象要从结果集中移除。

`observe()`方法属于结果集，通知的意义更适合结果集。通知表明加入到结果集中的并非必须是刚创建的对象；它可以是之前创建的或者以这种方式更新后现在才属于结果集的。删除也是如此；对象可能已经被更新或者删除触发从查询结果集中删除。

>注意`insertedInto`索引应用在结果集是在`removedFrom`索引位置移除之后（数组可能发生了变化）。

这个功能——提供了底层数据变化的通知——在任何store中在结果集提供了一个`observe()`方法。给store添加这个功能最简单的方式是用`dojo/store`的**Observable**方法封装它。例如，我们来创建一个数据集，实例化一个新Memory store并使用`dojo/store/Observable`封装它：

```
require(["dojo/store/Memory", "dojo/store/Observable"],
        function(Memory, Observable){
    var data = [
        {"name": "Dow Jones", "index": 12197.88, "date": new Date()},
        {"name": "Nasdaq", "index": 2730.68, "date": new Date()},
        {"name": "S&P 500", "index": 1310.19, "date": new Date()}
    ];
    // create the store with the data
    marketStore = new Memory({data: data, idProperty: "name"});
    // wrap the store with Observable to make it possible to monitor:
    marketStore = Observable(marketStore);
});
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/realtime_stores/demo/demo.html)

现在任何时候我们在本地通过调用`put()`、 `add()`或者 `remove()`修改数据，通知将传递给视图渲染器，从而就可以自动更新视图。

##远程发起的通知

使用`Observable`分装的store 在当数据更新时自动通知。然而，如果你创建一个实时应用，你可以还有来自其他用户的和由服务器传递的通知。这种情况下，在用`put()`、 `add()`和`remove()`就没有意义了，因为这些都表示本地用户执行要发送给服务器的操作。随着服务器发起调用，我们不需要更新操作返回到服务器，因为服务器已经知道了这个变化，而要在服务器端抑制这些“回声”实际上很有挑战。因此，`Observable`store封装器提供了一个**notify()**方法，这专门用来通知store发生了一个来自其它资源的变化。 `put()`和`notify()`之间的主要区别是`put()`请求一个改变，而`notify()`则表示已经发生了一个改变。

有了`notify()`方法，我们对于数据变化通知就有了一个单独的调用目标。`notify()`方法接收两个参数：第一个参数是被添加或更新的对象，第二个参数是被更新或删除的对象的标识。如果只给出了第一个参数，它表示创建了一个新的对象。如果第一个参数是`undefined`或者`null`（并且有第二个参数），则表示要移除被引用的对象。如果两个参数都有，它表示发生了一次更新。要通知更新了对象，我们可以调用`notify()`：

```
marketStore.notify(
    {"date": "2008-02-29", "name": "Dow Jones", "index": 12197.88},
            "Dow Jones");
```

为了演示，我们模拟远程触发一个简单的随机`setInterval`函数：

```
setInterval(function(){
    // choose a market randomly
    var market = data[Math.floor(Math.random() * 3)];
    // change it randomly
    market.index += Math.random() - 0.5;
    // update date
    market.date = new Date();
    // notify of the change
    marketStore.notify(market, market.name);
}, 1000); // every second
```

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/realtime_stores/demo/demo.html)

由于`notify()`方法通常与Comet-driven 消息结合，让我们看看如何和 [dojox/socket](http://www.sitepen.com/blog/2010/10/31/dojo-websocket/)来使用它，来基于WevSockets和回退到XHR长轮询的 Comet-style 通讯。简单地说，`dojox/socket`允许我们使用WebSocket 或者长期连接的XHR来连接服务器，并从服务器异步接收消息。这里我们创建一个socket连接并使用来自服务器的信息来将更新通知给store：

```
require(["dojox/socket"],
    function(Socket){
    var socket = Socket("/comet");
    socket.on("message", function(event){
      var data = event.data;
      switch(data.action){
        case "create": store.notify(data.object); break;
        case "update": store.notify(data.object, data.object.id); break;
        case "delete": store.notify(undefined, data.object.id); break;
        default: // some other action
      }
    });
});

```

##实现你自己的observe()

有些情况下直接使用你自己的`observe()`方法会更加高效。如果你有专门的缓存或者通知方案，或者你已经实现了你自己的`query()`方法，这就很重要了。我们直接在`query()`方法中实现附加的`observe()`方法。基本的实现模式如下：

```
require(["dojox/socket", "dojo/store/Memory", "dojo/store/util/QueryResults", "dojo/_base/array"],
        function(Socket, Memory, QueryResults, arrayUtil){
    new Memory({
        idProperty: "name",
                data: data,
        query: function(query, options){
            // execute the query and get the results as an array
            var resultsArray = this.queryEngine(query, options)(this.data));

            // create a results object with standard iterative methods
            var results = QueryResults(resultsArray);

            // keep track of listeners
            var listeners = [];

            // add the observe method
            results.observe = function(listener){
                listeners.push(listener);
            };
            socket.on("message", function(event){
                // ... process event
                arrayUtil.forEach(listeners, function(listener){
                    listener(object, insertedInto, removedFrom);
                });
            });
        }
    });
});

```

##小结

Dojo对象存储接口中的可观察模式为数据模型整合实时更新提供了一个强力的基础。数据视图可以直接连接查询结果而不用知道数据是如何变化的。使用一致的接口，视图可以应对各种数据变化，无论是本地启动的或是通过服务器端转发作为一个远程操作的结果。
