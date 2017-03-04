# 6.3 MVC应用的数据模型

原文地址：[https://dojotoolkit.org/documentation/tutorials/1.10/data\_modeling/index.html](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/index.html)

---

模型-视图-控制器（MVC）是应用开发的主流范式。这里我们将了解Dojo提供给MVC应用的基础。我们将了解如何在基本模型中利用Dojo对象存储和Stateful对象，以及如何在模型之上构建模块化视图和控制器代码。

\#\#MVC应用的数据模型

MVC通过分离关注点来组织和管理应用代码。Dojo是基于MVC理念的，它为MVC结构的应用提供了强力的帮助。优良设计的MVC应用的基础是一个稳固的数据模型。这里我们将了解如何利用Dojo对象存储和Stateful对象来创建一个可以用在视图和控制器代码的强健的模型。

\#\#模型

模型是MVC中的M。数据模型代表你应用中访问和操作的核心信息。模型是你应用的中心，视图和控制器则以友好的方式将用户和数据模型连接起来。模型封装了存储和验证过程。

\[Dojo对象存储\]\([https://dojotoolkit.org/documentation/tutorials/1.10/intro\_dojo\_store/\) 在Dojo应用中充当模型角色。存储接口设计用来将数据从应用的其余部分分离出来。可以在不改变存储接口的前提下使用不同的存储媒介。Store还可以拓展出更多功能。让我们先看一个基本存储的结构。我们将使用一个JsonRest存储并缓存它返回的项：](https://dojotoolkit.org/documentation/tutorials/1.10/intro_dojo_store/\)在Dojo应用中充当模型角色。存储接口设计用来将数据从应用的其余部分分离出来。可以在不改变存储接口的前提下使用不同的存储媒介。Store还可以拓展出更多功能。让我们先看一个基本存储的结构。我们将使用一个JsonRest存储并缓存它返回的项：)

\`\`\`

require\(\["dojo/store/JsonRest", "dojo/store/Memory", "dojo/store/Cache", "dojo/store/Observable"\],

```
    function\(JsonRest, Memory, Cache, Observable\){

masterStore = new JsonRest\({

    target: "/Inventory/"

}\);

cacheStore = new Memory\({}\);

inventoryStore = new Cache\(masterStore, cacheStore\);
```

\`\`\`

inventoryStore代表我们的基础数据模型。我们可以使用get\(\)来获取数据，使用query\(\)来查询数据，使用put\(\)来修改数据。store通过处理服务器的交互封装了这些信息。

我们可以将视图与查询结果相连接：

\`\`\`

results = inventoryStore.query\("some-query"\);

viewResults\(results\);

// pass the results on to the view

function viewResults\(results\){

```
var container = dom.byId\("container"\);



// results object provides a forEach method for iteration

results.forEach\(addRow\);



function addRow\(item\){

    var row = domConstruct.create\("div",{

        innerHTML: item.name + " quantity: " + item.quantity

    }, container\);

}
```

}

\`\`\`

现在我们的\`viewResult\`扮演一个数据模型的视图。我们也可以利用\`dojo/string\`的\`substitute\`函数来制作简单的模板。

\`\`\`

```
function addRow\(item\){

    var row = domConstruct.create\("div",{

        innerHTML: string.substitute\(tmpl, item\);

    }, container\);

}
```

\`\`\`

\#\#集合数据绑定

用视图监控数据模型并随时响应变化是MVC中的一个重要方面。它可以避免控制器和视图之间不必要的耦合。控制器更新模型，然后视图观察到并响应这种变化。我们可以使用\`dojo/store/Observable\`包装来让数据模型可以观察。

\`\`\`

masterStore = new Observable\(masterStore\);

...

inventoryStore = new Cache\(masterStore, cacheStore\);

\`\`\`

现在我们的视图可以通过observe方法来监听查询结果。

\`\`\`

function viewResults\(results\){

```
var container = dom.byId\("container"\);

var rows = \[\];



results.forEach\(insertRow\);



results.observe\(function\(item, removedIndex, insertedIndex\){

    // this will be called any time a item is added, removed, and updated

    if\(removedIndex &gt; -1\){

        removeRow\(removedIndex\);

    }

    if\(insertedIndex &gt; -1\){

        insertRow\(item, insertedIndex\);

    }

}, true\); // we can indicate to be notified of object updates as well



function insertRow\(item, i\){

    var row = domConstruct.create\("div", {

        innerHTML: item.name + " quantity: " + item.quantity

    }\);

    rows.splice\(i, 0, container.insertBefore\(row, rows\[i\] \|\| null\)\);

}



function removeRow\(i\){

    domConstruct.destroy\(rows.splice\(i, 1\)\[0\]\);

}
```

}

\`\`\`

&gt;\[View Demo\]\([https://dojotoolkit.org/documentation/tutorials/1.10/data\_modeling/demo/demo.html\](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/demo/demo.html\)\)

我们现在有了一个可以直接响应模型变化的视图，控制器代码可以改变store中的数据来响应用户交互。控制器可以用\`put\(\)\`、\`add\(\)\`和\`remove\(\)\`方法来改变数据。通常控制器代码用来处理事件，例如，在用户点击添加按钮时可以创建一个新的data对象：

\`\`\`

on\(addButton, "click", function\(\){

```
inventoryStore.add\({

    name: "Shoes",

    category: "Clothing",

    quantity: 40

}\);
```

}\);

\`\`\`

&gt;\[View Demo\]\([https://dojotoolkit.org/documentation/tutorials/1.10/data\_modeling/demo/demo.html\](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/demo/demo.html\)\)

这将在视图中触发一个更新，我们完全不需要直接对视图起作用。只需要控制器代码来响应用户动作和处理模型。模型的数据存储和视图的渲染都完全从这些代码中分离了出来。

\#\#丰富数据模型

对于store我们现在都用的很简单，除了简单对象储存之外还没有包含任何逻辑（虽然服务器端可能有额外的逻辑和验证）。不影响应用的其他不见，我们可以添加更多的功能。

\#\#\# 验证

我们要给store添加的第一个扩展是验证。对于\`JsonRest\`store这很简单，因为所有的更新都通过\`put\(\)\`方法（\`add\(\)\`调用\`put\(\)\`）。我们可以在构造器调用中添加一个\`put\`方法来扩展inventoryStore ：

\`\`\`

var oldPut = inventoryStore.put;

inventoryStore.put = function\(object, options\){

```
if\(object.quantity &lt; 0\){

    throw new Error\("quantity must not be negative"\);

}

// now call the original

oldPut.call\(this, object, options\);
```

};

\`\`\`

现在更新时会经过验证逻辑检查：

\`\`\`

inventoryStore.put\({

```
name: "Donuts",

category: "Food",

quantity: -1
```

}\);

\`\`\`

&gt;\[View Demo\]\([https://dojotoolkit.org/documentation/tutorials/1.10/data\_modeling/demo/demo.html\](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/demo/demo.html\)\)

它会抛出一个错误并拒绝改变。

\#\#\# 层级

当我们给数据模型添加逻辑时，就是在给原始数据增添意义。其中一个就是展示层级。对象存储定义了一个\`getChildren\(\)\`方法来实现父子关系的可视化。有几种不同的方式来存储这些关系。

存储对象可以拥有一组指向子集的引用。这是一个很好的设计，它需要小的有序列表。或者，对象可以记录他们的父节点。后者是一种可伸缩性更好的设计。

我们可以简单地添加一个\`gerChildren\(\)\`方法来实现第二种设计。在这个例子中我们的层级从拥有个体项作为子集的类别对象开始。我们会创建一个\`gerChildren\(\)\`方法将查找所有的对象来对比谁的category 属性能够匹配父对象的名称，就将父子关系定义为子集的一个属性。

\`\`\`

inventoryStore.getChildren = function\(parent, options\){

```
return this.query\({

    category: parent.id

}, options\);
```

};

\`\`\`

现在分层视图可以调用\`gerChildren\(\)\`来得到一个对象的子集列表，不需要理解数据的结构。子集的检索可能是这样的：

\`\`\`

require\(\["dojo/\_base/Deferred"\], function\(Deferred\){

```
Deferred.when\(inventoryStore.get\("Food"\), function\(foodCategory\){

    // retrieved the food category object, now get it's children

    inventoryStore.getChildren\(foodCategory\).forEach\(function\(food\){

        // handle each item in the food category

    }\);

}\);
```

}\);

\`\`\`

我们得到了一个对象的子集，现在来看如何改变它。我们知道当使用inventoryStore时层级管理是通过category 属性来定义的。如果想要改变子集的类别，只需要简单地改变category 属性：

\`\`\`

donut.category = "Junk Food";

inventoryStore.put\(donut\);

\`\`\`

Dojo store的一个核心概念是在数据模型与其他组件之间提供一个一致的接口。如果我们想要在不需要对象内部结构的情况下让组件设置一个对象的父级进而定义层级关系，可以使用\`put\(\)\`方法的\`options\`参数的\`parent\`属性：

\`\`\`

inventoryStore.put = function\(object, options\){

```
if\(options.parent\){

    object.category = options.parent;

}

// ...
```

};

\`\`\`

现在我们可以这么改变父级：

\`\`\`

inventoryStore.put\(donut, {parent: "Junk Food"}\);

\`\`\`

\#\#\# 有序存储

默认情况下，一个store代表对象的一个无序集合。但是如果各项有一个潜在顺序我们就可以轻易实现对store的排序。一个有序store的第一需求是从\`query\(\)\`调用（当没有定义一个替代排序时）返回一个顺序对象。这不需要额外扩展，只要你正确按顺序响应查询。

有序store也希望能够提供一个接口来让对象可以不同方式排序。应用希望提供一种手段来上下前后移动对象。这可以通过使用\`put\(\)\`的可选参数中的\`before\`属性：

\`\`\`

inventoryStore.put = function\(object, options\){

```
if\(options.before\){

    // we set the reference object's name in the object's "insertBefore"

    // so the server can put the object in the right order

    object.insertBefore = options.before.id;

}

// ...
```

};

\`\`\`

服务器可以响应insertBefore 属性来给对象排序。控制器代码可以移动对象（我们使用事件委托并且假设我们在创建时设置了节点的\`itemIdentity\`和\`beforeId\`属性）：

\`\`\`

require\(\["dojo/on"\],

```
    function\(on\){

on\(moveUpButton, ".move-up:click", function\(\){

    // \|this\| in event delegation is the node

    // matching the given selector

    inventoryStore.put\(inventoryStore.get\(this.itemId\), {

        before: inventoryStore.get\(this.beforeId\)

    }\);

}\);
```

\`\`\`

\#\#事务处理

事务处理是许多应用的关键部分，应用逻辑通常需要定义要自动你整合什么操作。一种方式是收集事务期间的所有操作并在事务提交时把他们放进一个请求中。这里有一个例子：

\`\`\`

require\(\["dojo/\_base/lang"\],

```
    function\(lang\){

lang.mixin\(inventoryStore, {

    transaction: function\(\){

        // start a transaction, create a new array of operations

        this.operations = \[\];

        var store = this;

        return {

            commit: function\(\){

                // commit the transaction, sending all the operations in a single request

                return xhr.post\({

                    url:"/Inventory/",

                    // send all the operations in the body

                    postData: JSON.stringify\(store.operations\)

                }\);

            },

            abort: function\(\){

                store.operations = \[\];

            }

        };

    },

    put: function\(object, options\){

        // ... any other logic ...



        // add it to the queue of operations

        this.operations.push\({action:"put", object:object}\);

    },

    remove: function\(id\){

        // add it to the queue of operations

        this.operations.push\({action:"remove", id:id}\);

    }

}\);
```

\`\`\`

随后我们可以使用它创建自定义操作：

\`\`\`

```
removeCategory: function\(category\){

    // atomically remove entire category and the items within the category

    var transaction = this.transaction\(\);



    var store = this;

    this.getChildren\(category\).forEach\(function\(item\){

        // remove each child

        store.remove\(item.id\);

    }, this\).then\(function\(\){

        // now remove the category

        store.remove\(category.id\);

        // all done, commit the changes

        transaction.commit\(\);

    }\);

}
```

\`\`\`

\#\# 对象数据绑定：dojo/Stateful

在数据模型的集合层面和实体层面之间Dojo有着清晰的描述。Dojo store提供集合层面的架构。现在我们查看单个对象的建模。Dojo在单个对象建模中使用同一的接口。我们可以使用\`dojo/Stateful\`接口来操作对象。这个接口很简单，它有三个关键方法：

* \`get\(name\)\`—— 检索给定名称属性的值

* \`set\(name，value\)\`—— 设置给定名称属性的值

* \`watch\(name，listener\)\`—— 为给定属性的变化注册一个回调（可以忽略第一个参数来监听所有变化）

这个结构像store一样让视图可以渲染来响应数据的变化。让我们创建一个简单的HTML表单绑定对象的视图。首先HTML是这样的：

\`\`\`

&lt;form id="itemForm"&gt;

```
Name: &lt;input type="text" name="name" /&gt;

Quantity: &lt;input type="text" name="quantity" /&gt;
```

&lt;/form&gt;

\`\`\`

然后我们可以绑定给HTML：

\`\`\`

function viewInForm\(object, form\){

```
// copy initial values into form inputs

for\(var i in object\){

    updateInput\(i, null, object.get\(i\)\);

}

// watch for any future changes in the object

object.watch\(updateInput\);

function updateInput\(name, oldValue, newValue\){

    var input = query\("input\[name=" + name + "\]", form\)\[0\];

    if\(input\){

        input.value = newValue;

    }

}
```

}

\`\`\`

现在我们可以用来自store的对象来初始化这个表单：

&gt;\[View Demo\]\([https://dojotoolkit.org/documentation/tutorials/1.10/data\_modeling/demo/demo.html\](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/demo/demo.html\)\)

现在控制器代码可以修改这个对象，视图会立即响应：

\`\`\`

item.set\("quantity", 4\);

\`\`\`

在表单中我们还想添加\`onchange\`事件监听器，在输出改变时更新对象，所以让数据双向绑定（对象的改变反映在表单里，表单的改变也反映到对象中）。Dojo在\[表单管理\]\([https://dojotoolkit.org/documentation/tutorials/1.10/form\_manager/\)中也提供了更多高阶表单交互功能。](https://dojotoolkit.org/documentation/tutorials/1.10/form_manager/\)中也提供了更多高阶表单交互功能。)

还要记住在改变准备提交是，包裹的对象可以也应该\`put\(\)\`到store。代码如下：

\`\`\`

on\(saveButton, "click", function\(\){

```
inventoryStore.put\(currentItem\); // save the current state of the Stateful item
```

}\);

\`\`\`

&gt;\[View Demo\]\([https://dojotoolkit.org/documentation/tutorials/1.10/data\_modeling/demo/demo.html\](https://dojotoolkit.org/documentation/tutorials/1.10/data_modeling/demo/demo.html\)\)

\#\#dstore:dojo/store的未来

新的\[dstore\]\([https://github.com/sitepen/dstore\)包是dojo/store的继承者，在Dojo](https://github.com/sitepen/dstore\)包是dojo/store的继承者，在Dojo) 1.8+可用，也是Dojo 2的计划API。如果你刚接触Dojo，我们推荐你看一看dstore。注意它也包含集合和数据建模的支持。

\#\#总结

使用Dojo store框架和stateful接口，我们拥有了一个坚固的数据模型来构建我们的MVC应用。视图可以直接渲染数据模型和直接监听响应数据的变化。控制器可以以一致的方式与数据交互，而不用耦合到特定的数据结构，也不用显式地操作视图。集合和实体结构泾渭分明。所有这些帮助你清晰分离关注点，快速构建有组织的、易于管理的应用。

