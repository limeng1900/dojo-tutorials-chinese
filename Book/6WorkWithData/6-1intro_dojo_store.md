#6.1 Dojo Object Store

原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/intro_dojo_store/index.html

---

关注点分离是良好编程的基础。让展示从数据模型中区分出来是要做的关键分离。Dojo Object Store框架受HTML5对象存储API启发，为数据交互建立了一致的接口。

##为什么要用Dojo对象存储

关注点的分离是组织化和易于管理的编程的基础方面，而web应用的一个基本分离就是数据模型从用户界面的分离（在模型-视图-控制器（MVC）架构中用户界面通常定义为视图和控制器）。Dojo Object Store框架受HTML5对象存储API启发，为数据交互建立了一致的接口。这个API用以推进松耦合开发，不同的widget和用户界面可以统一的方式与各种来源的数据进行交互。

你可以通过Dojo Object Store接口开发和使用封装好的组件，这些组件可以轻易连接到各种数据提供者。Dojo Object Store是一个API，它有多种实现，叫做stores。store包括一个简单的Memory store、一个JSON/REST store、旧的`dojo.data` stores和提供额外功能的store封装器。

##入门

从最简单的`dojo/store/Memory`开始。我们可以简单地向构造器提供一组对象来开始进行交互。一旦创建了store，我们就可以是所有`query`来查询它。查询的一个简单方法是提供一个带有name/value 的对象，name/value 指出匹配对象所需要的值。`query`方法通常返回一个带有`forEach`方法（还有`map`和`filter`）的对象或者数组：

```
require(["dojo/store/Memory"],
    function(Memory){

        var employees = [
            {name:"Jim", department:"accounting"},
            {name:"Bill", department:"engineering"},
            {name:"Mike", department:"sales"},
            {name:"John", department:"sales"}
        ];
        var employeeStore = new Memory({data:employees, idProperty: "name"});
        employeeStore.query({department:"sales"}).forEach(function(employee){
            // this is called for each employee in the sales department
            alert(employee.name);
        });

});
```

它将弹出销售部门的每个雇员的名字。

>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/intro_dojo_store/demo/demo.html)

我们可以继续在store中创建新的对象和删除对象：

```
// add a new employee
employeeStore.add({name:"George", department:"accounting"});
// remove Bill
employeeStore.remove("Bill");
```

我们可以检索和更新对象。store中的对象是简单的纯JavaScript对象，所以我们可以直接访问和修改属性（当你修改属性时，确保你做了put()来保存更改）：

```
// retrieve object with the name "Jim"
var jim = employeeStore.get("Jim");
// show the department property
console.log("Jim's department is " + jim.department);
// iterate through all the properties of jim:
for(var i in jim){
    console.log(i, "=", jim[i]);
}
// update his department
jim.department = "engineering";
// and store the change
employeeStore.put(jim);
```

回到查询，我们可以给查询添加额外的参数。这些额外参数可以让我们限定查询对象的序号或者对对象进行排序，它放在`query`方法的第二的参数。这个参数可以是一个带有`start`和`count`属性来定义返回对象的限定序号的对象。限定结果集是大规模数据集使用分页插widget（比如grid）的关键所在，只在需要时请求新一页数据。第二个参数也可以包含一个`sort`属性，来指定查询中排序的属性和方向：

```
employeeStore.query({department:"sales"}, {
    // the results should be sorted by department
    sort:[{attribute:"department", descending: false}],
    // starting at an offset of 0
    start: 0,
    // with a limit of 10 objects
    count: 10
}).map(function(employee){
    // return just the name, mapping to an array of names
    return employee.name;
}).forEach(function(employeeName){
    console.log(employeeName);
});
```

Memory store 是一个同步的store，就是说它直接返回行动的结果（`get`返回对象）。

##dojo/store/JsonRest

另一个很常用的store是`JsonRest` store，它通过JSON使用基于标准HTTP/REST向你的服务器委托各种store动作。这些store动作直接映射HTTP GET、PUT、POST和DELETE方法。服务器端的更多细节请看[JsonRest 文档](https://dojotoolkit.org/reference-guide/1.10/dojo/store/JsonRest.html)。

它也是一个异步store的例子。异步store的方法返回[promise](https://dojotoolkit.org/documentation/tutorials/1.10/promises/)。我们可以通过给返回的promise提供一个回调来使用它。

```
require(["dojo/store/JsonRest"],
    function(JsonRest){
            employeeStore = new JsonRest({target:"/Employee/"});
            employeeStore.get("Bill").then(function(bill){
                // called once Bill was retrieved
            });
});
```

也可以使用`Deferred.when()`（通过`dojo/_base/Deferred`模块给出）来使用异步和同步的方法，不管哪种实现都使用一致的行为。

这些例子展示如何与store交互。现在我们摆脱对特定实现的依赖，开始构建与store交互的widget和组件。我们还可以把store加入到已存在的使用store的组件中。

例如，StoreSeries 可以让我们把store作为一个图表的数据源。大多数使用store的组件需要你提供一个查询：

```
// Note that while the Default plot2d module is not used explicitly, it needs to
// be loaded to be able to create a Chart when no other plot is specified.
require(["dojox/charting/Chart", "dojox/charting/StoreSeries" /*, other deps */,
        "dojox/charting/plot2d/Default"],
    function(Chart, StoreSeries /*, other deps */){
        /* create stockStore here... */

        new Chart("lines").
            /* any other config of chart */
            // now use a data series from my store
            addSeries("Price", new StoreSeries(
                stockStore, {query: {sector:"technology"}}, "price")).
            render();

});
```

Dojo store框架的另一个重要概念是分层store封装器的组合功能。Dojo提供一些store封装器来添加功能，包括一个缓存封装器和一个在数据改变时触发时间的观察封装器。

## 本地存储

Dojo1.10在dojox中加入了本地存储dojo/store提供者，支持IndexedDB和WebSQL。

##dstore：dojo/store的未来

新的[dstore](https://github.com/sitepen/dstore)包是dojo/store的继承者，在Dojo 1.8+可用，也是Dojo 2的计划API。如果你刚接触Dojo，我们推荐你看一看dstore。

##小结
Dojo Object Store 自1.6实现以来成为了一个有用的工具，它可以帮助我们在数据和用户界面之间保持一个清晰地分离。它为轻松开发自定义store提供一个直接的API。请浏览参考指南和下面的信息来了解更多。

##额外资源

 - [JsonRest Reference Guide](https://dojotoolkit.org/reference-guide/1.10/dojo/store/JsonRest.html)
 - [SitePen blog post on Dojo Object Stores](https://www.sitepen.com/blog/2011/02/15/dojo-object-stores/)
