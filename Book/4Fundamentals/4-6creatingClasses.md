#4.6 创建类
 原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/declarative/index.html 

---

`dojo/_base/declare`模块是Dojo Toolkit里类创建的基础。`declare`允许多重继承来让开发者编写灵活的代码并避免重复同样的代码。Dojo、 Dijit 和 Dojox 模块都使用`declare`；在本教程中，你将了解为什么你也要用它。

##入门
确保你了解[模块教程](https://dojotoolkit.org/documentation/tutorials/1.10/modules)中提出的概念。

##Dojo的基础类创建
`declare`函数定义在`dojo/_base/declare`模块里。`declare`接收三个参数：`className`、`superClass`和`properties`。

### ClassName
`className`参数表示要创建的类的名字，包括命名空间。命名的类放在全局作用域内。`className`也可以通过命名空间表示继承链。

#### 命名类

```
// Create a new class named "mynamespace.MyClass"
declare("mynamespace.MyClass", null, {

    // Custom properties and methods here

});
```
一个名叫`mynamespace.MyClass`的类现在存在于应用的全局作用域。
如果用在Dojo解析器，应该只创建命名类。其他类应该忽略`className`参数。

#### “匿名”类

```
// Create a scoped, anonymous class
var MyClass = declare(null, {

    // Custom properties and methods here

});
```
`MyClass`现在它给出的作用域的可用。

### 超类
超类的参数可以是`null`、一个已有类或者一个已有类的数组。如果一个新类继承自多个类，那么列表中的第一个类将作为基本原型，其余的作为“混合”。

#### 无继承类

```
var MyClass = declare(null, {

    // Custom properties and methods here

});
```
`null`意味着这个类没有类来继承。

#### 单继承的类

```
var MySubClass = declare(MyClass, {

    // MySubClass now has all of MyClass's properties and methods
    // These properties and methods override parent's

});
```
新的`MySubClass`将继承`MyClass`的属性和方法。一个父类的方法或属性可以进行重写，只要在第三个参数里添加它的新定义的键值，随后再细说。

#### 多继承的类

```
var MyMultiSubClass = declare([
    MySubClass,
    MyOtherClass,
    MyMixinClass
],{

    // MyMultiSubClass now has all of the properties and methods from:
    // MySubClass, MyOtherClass, and MyMixinClass

});
```
一个类数组表示多继承。属性和方法从左到右继承。数组的第一个类作为基础原型，随后的类混入该类。
如果一个属性或方法在多个被继承的类里指定过，则使用最后一个被继承的该属性或方法。

### 属性和方法对象
`declare`最后一个参数包含了类原型的方法和属性。如果被继承的类有同样的属性或方法，这个参数将会对同名的属性和方法进行重写。

#### 自定义属性和方法

```
// Class with custom properties and methods
var MyClass = declare(MyParentClass, {
    // Any property
    myProperty1: 12,
    // Another
    myOtherProperty: "Hello",
    // A method
    myMethod: function(){

        // Perform any functionality here

        return result;
    }
});
```

#### 示例：基类的创建和继承
下面的代码创建一个继承自`dijit/form/Button`的widget：

```
define([
    "dojo/_base/declare",
    "dijit/form/Button"
], function(declare, Button){
    return declare("mynamespace.Button", Button, {
        label: "My Button",
        onClick: function(evt){
            console.log("I was clicked!");
            this.inherited(arguments);
        }
    });
});
```
从上面的代码，很容易得到以下结论：

 - 类名是`mynamespace.Button`
 - 该类可以通过全局变量`mynamespace.Button`来引用，或者从模块的返回值
 - 该类继承自`dijit/form/Button`（因此Button是依赖项）
 - 该类设置了几个自定义属性和方法

让我们通过学习`constructor`方法来深入了解Dojo类的创建。

## 构造器方法
`constructor`方法是一个特殊的类方法。`constructor`方法在类实例化时触发，在新对象的作用域内执行。这就是说`this`关键字引用该实例，而不是原始类。`constructor`方法也接收任意数量实例指定的参数。

```
// Create a new class
var Twitter = declare(null, {
    // The default username
    username: "defaultUser",

    // The constructor
    constructor: function(args){
        declare.safeMixin(this,args);
    }
});
```
如下创建实例：

```
var myInstance = new Twitter();
```
该实例中的username在指定设置之前都将是“defaultUser”。利用`safeMixin`方法提供一个username参数：

```
var myInstance = new Twitter({
    username: "sitepen"
});
```
现在该实例将username设为`sitepen`。

`declare.safeMixin`在类的创建和继承中也很有用。如API文档中：
> 这个函数用来像lang._mixin 一样混入属性，但是它跳过一个构造器属性并且像dojo/_base/declare那样修饰函数。就是说它用来和dojo/_base/declare进行类和对象的生成。和declare.safeMixin混合的函数可以想普通方法一样使用this.inherited()。这个函数用来实现declare()产生的构造器的方法的extend()。

`declare.safeMixin`是创建类是的好帮手。

##继承
如上所述，`declare`第二个参数定义继承。父类的属性和方法从左到右混入类，如果属性已定义后来的具有优先权。如下：

```
// Define class A
var A = declare(null, {
    // A few properties...
    propertyA: "Yes",
    propertyB: 2
});

// Define class B
var B = declare(A, {
    // A few properties...
    propertyA: "Maybe",
    propertyB: 1,
    propertyC: true
});

// Define class C
var C = declare([mynamespace.A, mynamespace.B], {
    // A few properties...
    propertyA: "No",
    propertyB: 99,
    propertyD: false
});
```
继承类最后的属性是：

```
// Create an instance
var instance = new C();

// instance.propertyA = "No" // overridden by B, then by C
// instance.propertyB = 99 // overridden by B, then by C
// instance.propertyC = true // kept from B
// instance.propertyD = false // created by C
```
理解原型继承很重要。当读取一个对象示例的属性时，示例先检查自己有没有定义它。如果没有则遍历原型链，并返回链上具有该属性的第一个对象的值。当属性值已指定时，总是用对象实例上指定的，从来不用原型的。就是说共享一个原型的对象将返回相同原型定义的值，除非该实例上指定了该值。这在类声明式就很容易为原始数据类型（number、string、boolean）设定默认值，并且需要的时候可以在实例中更新它们。然而，如果你在原型的属性上指定一个对象值（Object、Array），每个实例都将操作这个共享的值。如下：

```
var MyClass = declare(null, {
    primitiveVal: 5,
    objectVal: [1, 2, 3]
});

var obj1 = new MyClass();
var obj2 = new MyClass();

// both return the same value from the prototype
obj1.primitiveVal === 5; // true
obj2.primitiveVal === 5; // true

// obj2 gets its own property (prototype remains unchanged)
obj2.primitiveVal = 10;

// obj1 still gets its value from the prototype
obj1.primitiveVal === 5; // true
obj2.primitiveVal === 10; // true

// both point to the array on the prototype,
// neither instance has its own array at this point
obj1.objectVal === obj2.objectVal; // true

// obj2 manipulates the prototype's array
obj2.objectVal.push(4);
// obj2's manipulation is reflected in obj1 since the array
// is shared by all instances from the prototype
obj1.objectVal.length === 4; // true
obj1.objectVal[3] === 4; // true

// only assignment of the property itself (not manipulation of object
// properties) creates an instance-specific property
obj2.objectVal = [];
obj1.objectVal === obj2.objectVal; // false
```
为了避免无意地在实例之间共享数组或对象，对象属性应该声明为空并在构造器函数里进行初始化。

```
declare(null, {
    // not strictly necessary, but good practice
    // for readability to declare all properties
    memberList: null,
    roomMap: null,

    constructor: function () {
        // initializing these properties with values in the constructor
        // ensures that they ready for use by other methods
        // (and are not null or undefined)
        this.memberList = [];
        this.roomMap = {};
    }
});
```
更多信息参考 [dojo/_base/declare](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/declare.html#arrays-and-objects-as-member-variables)

### this.inherited
完全重写方法当然很有用，有时候每个类的构造器执行继承链时需要保留其原始功能。这里`this.inherited(arguments)`就很方便。`this.inherited(arguments)`调用父类的同名方法。如下：

```
// Define class A
var A = declare(null, {
    myMethod: function(){
        console.log("Hello!");
    }
});

// Define class B
var B = declare(A, {
    myMethod: function(){
        // Call A's myMethod
        this.inherited(arguments); // arguments provided to A's myMethod
        console.log("World!");
    }
});

// Create an instance of B
var myB = new B();
myB.myMethod();

// Would output:
//        Hello!
//        World!
```
`this.inherited`方法在子类代码中任何时候都可以调用。有些情况你会想要在子函数中间或者末尾调用`inherited()`。即便如此，你不能在构造器里调用它。

##小结
`declare`函数是创建模块化和重用Dojo Toolkit 类的关键。`declare`允许多重继承和任意数量的属性和方法来生成复杂类。更简单的是`declare`易于学习并且可以让开发者避免重复代码。

##dojo/_base/declare资源
`declare`和类创建的更多细节请查看一下资源：

 - [dojo/declare](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/declare.html)
 - [dojo/_base/lang::mixin](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/lang.html#dojo-base-lang-mixin)
 - [Writing Your Own Widget](https://dojotoolkit.org/reference-guide/1.10/quickstart/writingWidgets.html)
