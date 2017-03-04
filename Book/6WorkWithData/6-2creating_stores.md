# 6.2 创建Store

原文地址：[https://dojotoolkit.org/documentation/tutorials/1.10/creating\\_stores/index.html](https://dojotoolkit.org/documentation/tutorials/1.10/creating_stores/index.html)

---

在这篇教程中，你将了解所有`dojo/store`遵循的基础API以及如何创建你自己的store——包括如何处理查询结果。

## 入门

新的[`dojo/store`](https://dojotoolkit.org/reference-guide/1.10/dojo/store.html\)是用来代替旧的`dojo.data`系统的；它部分基于新的 [W3C Object Store API]\(http://www.w3.org/TR/IndexedDB/#object-store)，希望能够让创建数据存储组件变得尽可能简单。以`dojo/store`创建store非常简单，因为大多数方法都是可选的，只要在你需要的时候应用它。这篇教程将帮助你入门，并告诉你新的Dojo Stores里那些是最重要的部分。

基础的Dojo Store只实现了部分的 IndexedDB API，这些方法主要集中在数据存取上。IndexedDB API的某些方面（比如索引和指针）并没有实现，主要因为他们在纯JavaScript环境下不是必须的。

## 创建第一个store

在创建你的store实现之前，最好看一看Dojo工具集中最简单的store—— [Memory store](https://dojotoolkit.org/api/?qs=1.10/dojo/store/Memory)，它用来处理最原始的数据管理任务。下面是`dojo/store/Memory`的基础API：

```
define(["../_base/declare", "dojo/store/util/SimpleQueryEngine"],
        function(declare, SimpleQueryEngine) {

    declare("dojo.store.Memory", null, {
        constructor: function(options){ },
        data: null,
        index: null,
        queryEngine: SimpleQueryEngine,

        //    what follows is the actual API signature
        idProperty: "id",
        get: function(id){ },
        getIdentity: function(object){ },
        put: function(object, options){ },
        add: function(object, options){ },
        remove: function(id){ },
        query: function(query, options){ },
        setData: function(data){ }
    });
});
```

如你所见，`dojo/store/Memory`的签名很简单。它用来获取任意类型的数据（`get`和`query`），处理身份问题（`idProperty`和`getIdentity`），并且可以新建、删除和更新条目（`add`、`remove`和`put`）。另外，它还提供了设置初始化数据集（`setData`）的方式。

你可能注意到没有工具来发送通知时间，例如当数据创建、删除或更新时。我们会在另一个教程中讨论如何对已存在的Dojo Store应用`dojo/store/Observable`。

## 内部数据结构

大家可能都用过一些JavaScript数据结构，你可能注意到store没有规定一组对象的实际数据结构。这是经过`dojo/store`深思熟虑的，由于你数据的结构通常取决于数据的应用，store不需要决定它的结构。

那就是说，在自定义store中有一件事你要实现：每一个数据对象的唯一标识。store的`idProperty`属性指出那个项目属性作为唯一标识；默认是“id”，不过你可以任意指定。

你可以不用**idProperty**来编写一个store处理数据，但是我们强烈反对你这么做。没有唯一标识的store每次都要查找你的数据结构中的全部元素，这是性能上的浪费。

## query：Store中最重要的方法

目前，store中最重要的方法是`query`方法。它是不改变内部数据结构来获取store信息的主要方式。这个方法接收两个参数：一个`query`对象和一个可选的`options`对象。

`query`对象包含查询条件，它依赖基础的查询引擎。`dojo/store`自带一个内置的叫做`dojo/store/util/SimpleQueryEngine`的查询引擎，这个处理大部分的基础查询需求，也可以作为模板来编写更复杂的查询引擎。让我们来了解一下。

### 创建一个查询引擎

`dojo/store/util/SimpleQueryEngine`示范了递减查询引擎的示例。它的想法是创建并返回一个函数，这个函数会通过将一些初始条件传递给查询引擎来过滤（或者需要的其他方式）一组对象。

要创建一个查询引擎，你得遵循SimpleQueryEngine的结构，在闭包中捕获查询参数并返回一个将一组元素作为唯一参数的函数。基本的例子如下：

```
require(["dojo/_base/array"],
        function(arrayUtil){
var myEngine = function(query, options){
    var filteringFunction = function(object){
        //    do something here based on the passed query object
    };

    var execute = function(array){
        var results = arrayUtil.filter(array, filteringFunction);
        //    do anything else needed, like sorting and pagination
        return results;
    }
    execute.matches = filteringFunction;
    return execute;
}
```

你总可以给予`SimpleQueryEngine`来编写你自己的查询引擎，来处理任意可能的明确需求。如果真的需要的话，你也可以直接在store的`query`方法中创建一些东西，例如让查询方法与远程服务器通信并让服务器返回结果。稍后我们会在这篇教程中看到一个关于`dojo/store/JsonRest`的例子。

确保有了一个查询引擎来操作一组数据只是创建你的store中`query`方法的第一步；第二步是确保`query`方法用`dojo/store/util/QueryResults`来包装返回的结构。

### QueryResults是什么？

`dojo/store/util/QueryResults`是一个应用于查询结果的包裹函数。他确保结果集上存在标准的迭代方法，包括`forEach`、`map`、`filter`（更多参见[Arrays Made Easy](https://dojotoolkit.org/documentation/tutorials/1.10/arrays/) ）。

这里正是有意思的地方，你传递给`QueryResults`的`results`对象既可以是一个数组也可以是一个promise。没错，你可以给QueryResults函数传递一个[promise对象](https://dojotoolkit.org/documentation/tutorials/1.10/promises/)，而且迭代方法仍然能用。

让我们分别看一看在Memory store和JsonRest store这两种store中的`query`方法。首先，  
Memory store：

```
define(["dojo/store/util/QueryResults"],
        function(QueryResults){
        ....
//    from dojo/store/Memory
query: function(query, options){
    return QueryResults(
        (this.queryEngine(query, options))(this.data)
    );
}
```

Memory store的内部数据结构是一组对象。通过调用`QueryResults`，所有重要的迭代方法都直接加给结果对象。就是说随后可以直接调用迭代方法，如下：

```
var results = myMemoryStore.query({ foo: "bar" });
results.forEach(function(item){
    //    do something with the item
});
```

现在再来看下来自 JsonRest store 的`query`方法：

```
//    from dojo/store/JsonRest
query: function(query, options){
    var headers = {Accept: "application/javascript, application/json"};
    options = options || {};

    if(options.start >= 0 || options.count >= 0){
        headers.Range = "items=" + (options.start || '0') + '-' +
            (("count" in options && options.count != Infinity) ?
                (options.count + (options.start || 0) - 1) : '');
    }
    // lang is from dojo/_base/lang
    if(lang.isObject(query)){
        // ioQuery from dojo/io-query
        query = ioQuery.objectToQuery(query);
        query = query ? "?" + query: "";
    }
    if(options && options.sort){
        query += (query ? "&" : "?") + "sort(";
        for(var i = 0; i < options.sort.length; i++) {
            var sort = options.sort[i];
            query += (i > 0 ? "," : "")
                + (sort.descending ? '-' : '+')
                + encodeURIComponent(sort.attribute);
        }
        query += ")";
    }
    // request from dojo/request
    var results = request.get(this.target + (query || ""), {
        handleAs: "json",
        headers: headers
    });
    results.total = results.then(function(){
        var range = results.response.getHeaders("Content-Range");
        return range && (range=range.match(/\/(.*)/)) && +range[1];
    });
    return QueryResults(results);
}
```

你会注意到 JsonRest 并没有是用查询引擎，它是用`dojo/request`调用一个REST服务，该服务返回一个promise。随后QueryResults 确保返回的promise可以使用常用的迭代方法，并且让这些方法以正确的方式运行。

`QueryResults`使用了[`dojo.when`](https://dojotoolkit.org/documentation/tutorials/1.10/promises/)，这里我们不多说。只要记得当你编写自己的store时，你应该总是确保`query`函数返回一个`dojo/store/util/QueryResults`包裹的对象。

## 我们来创建一个store

现在我们已经有了基本的查询，让我们继续来创建一个新的store。我们把它叫做“Example”，再添加一些东西。简单起见，这个store最后会看起来像Memory store，因为我们只是简单地使用内部的一组数据并操作它。我们来设置它：

```
define(["dojo/store/util/QueryResults", "dojo/_base/declare", "dojo/_base/lang", "dojo/request", "dojo/store/util/SimpleQueryEngine"],
        function(QueryResults, declare, lang, request, SimpleQueryEngine){

    //    Declare the initial store
    return declare(null, {
        data: [],
        index: {},
        idProperty: "id",
        queryEngine: SimpleQueryEngine,

        constructor: function(options){
            lang.mixin(this, options || {});
            this.setData(this.data || []);
        },
        query: function(query, options){
            return QueryResults(
                (this.queryEngine(query, options))(this.data)
            );
        },
        setData: function(data){
            this.data = data;
            //    index our data
            this.index = {};
            for(var i = 0, l = data.length; i < l; i++){
                var object = data[i];
                this.index[object[this.idProperty]] = object;
            }
        }
    });
});
```

你可能已经注意到构造器中的`lang.mixin`语句。这是一个惯例，通过构造器的`options`参数来规范实例属性；在这个例子中，最常见的是用来设置`data`和`idProperty`。

### 加入我们的getter

我们的Example store 中最重要的方法都实现了：设置store数据的一种方式和基于`SimpleQueryEngine`来查询数据的一种方式。我们也有一个索引机制来通过身份标识快速返回一项；我们继续来加入这些方法：

```
//    in our declare from above
get: function(id){
    return this.index[id];
},
getIdentity: function(object){
    return object[this.idProperty];
}
```

这两种方法允许数据的直接访问，不用通过一个查询，并可以让用户针对一个对象得到一个正确的唯一标识。如果我们store的目的是只读（本质上），那这些就是store定义中所需要的了。

### 加入写入能力

无论如何大部分store并不是只读的。通常，用户会需要修改已存在的对象，从我们的store中添加或移除对象。为此，我们添加三个新的方法：`put`、`add`和`remove`。

```
//    in our declare from above
put: function(object, options){
    var id = options && options.id
        || object[this.idProperty];
    this.index[id] = object;

    var data = this.data,
        idProperty = this.idProperty;
    for(var i = 0, l = data.length; i < l; i++){
        if(data[i][idProperty] == id){
            data[i] = object;
            return id;
        }
    }
    this.data.push(object);
    return id;
},
add: function(object, options){
    var id = options && options.id
        || object[this.idProperty];
    if(this.index[id]){
        throw new Error("Object already exists");
    }
    return this.put(object, options);
},
remove: function(id){
    delete this.index[id];
    for(var i = 0, l = this.data.length; i < l; i++){
        if(this.data[i][this.idProperty] == id){
            this.data.splice(i, 1);
            return;
        }
    }
}
```

它的理念是在你任何要修改一个对象的时候，你可以使用`put`方法，创建一个新对象并加入store时使用`add`，将一个对象从store删除时用`remove`。这里`put`方法是重点：在你要变更一个对象时可以使用它，这样store就可以进行更新一类的操作了。`put`和`add`实现的不同之处是`add`方法确保对象还没有存在于store。

### 最终实现

下面是我们最终的store：

```
define(["dojo/store/util/QueryResults", "dojo/_base/declare", "dojo/store/util/SimpleQueryEngine"],
        function(QueryResults, declare, SimpleQueryEngine){

    //    Declare the initial store
    return declare(null, {
        data: [],
        index: {},
        idProperty: "id",
        queryEngine: SimpleQueryEngine,

        constructor: function(options){
            lang.mixin(this, options || {});
            this.setData(this.data || []);
        },
        get: function(id){
            return this.index[id];
        },
        getIdentity: function(object){
            return object[this.idProperty];
        },
        put: function(object, options){
            var id = options && options.id
                || object[this.idProperty];
            this.index[id] = object;

            var data = this.data,
                idProperty = this.idProperty;
            for(var i = 0, l = data.length; i < l; i++){
                if(data[i][idProperty] == id){
                    data[i] = object;
                    return id;
                }
            }
            this.data.push(object);
            return id;
        },
        add: function(object, options){
            var id = options && options.id
                || object[this.idProperty];
            if(this.index[id]){
                throw new Error("Object already exists");
            }
            return this.put(object, options);
        },
        remove: function(id){
            delete this.index[id];
            for(var i = 0, l = this.data.length; i < l; i++){
                if(this.data[i][this.idProperty] == id){
                    this.data.splice(i, 1);
                    return;
                }
            }
        },
        query: function(query, options){
            return QueryResults(
                (this.queryEngine(query, options))(this.data)
            );
        },
        setData: function(data){
            this.data = data;
            //    index our data
            this.index = {};
            for(var i = 0, l = data.length; i < l; i++){
                var object = data[i];
                this.index[object[this.idProperty]] = object;
            }
        }
    });
});
```

如你所见，使用新的Dojo Store API创建一个基本的store既简单又直接。

## 小结

在这篇教程中，我们已经了解了新的Dojo Store API 背后的历史和基础，以及如何创建我们自己的store，还有Dojo Store API 的两块核心——查询引擎和`dojo/store/util/QueryResults`如何作用。我们鼓励你在Dojo 工具集中（`dojo/store`中）探索store，你也会在DojoX（`dojox/store`中）中发现一些另外的store。

下一步：使用`dojo/store/Observable`和一些store来处理通知事件，还有[使用 Dojo Store API 处理实时数据](https://dojotoolkit.org/documentation/tutorials/1.10/realtime_stores)。

