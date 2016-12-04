# 4.3对象扩张
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/augmenting_objects/index.html

----------

当你使用JavaScript时，你是在和对象打交道。`dojo/_base/lang`可以让你在使用`dojo/_base/declare`时用`lang.mixin`、 `lang.extend`和`declare.safeMixin`来扩张对象和原型。

##入门
`lang.mixin`、 `lang.extend`和`declare.safeMixin`用来从一个或多个其它对象的属性扩张原始对象。在本文中其它对象称为“混入类”（mixins）。这些函数有一些小差异，来适用不同的应用场景。在我们深入之前先看一个简要概述：
<table >
    <tr>
        <th>Method:</th>
        <th>lang.mixin</th>
        <th>declare.safeMixin</th>
        <th>lang.extend</th>
    </tr>
    <tr>
        <th >Operates on</th>
        <td>object</td>
        <td>object</td>
        <td>object.prototype</td>
    </tr>
    <tr>
        <th >Mixes in <code>constructor</code> property</th>
        <td>yes</td>
        <td>no</td>
        <td>yes</td>
    </tr>
    <tr>
        <th >Mixes in multiple objects at once</th>
        <td>yes</td>
        <td>no</td>
        <td>yes</td>
    </tr>
    <tr>
        <th >Annotates functions to support <code>this.inherited</code></th>
        <td>no</td>
        <td>yes</td>
        <td>no</td>
    </tr>
    <tr>
        <th >Speed</th>
        <td>fast</td>
        <td>slow</td>
        <td>fast</td>
    </tr>
    <tr>
        <th >Use primarily with</th>
        <td>A plain object</td>
        <td>A declare instance</td>
        <td>A constructor</td>
    </tr>
</table>

##lang.mixin
`lang.mixin`方法是一个简单的通用函数，可以用任意数量的对象作为参数，把随后的对象的属性添加到第一个对象上并返回它。例如，我们已经有一个对象需要添加一些额外属性。它可能是表单数据的集合、模板的一些数据、一个命名空间对象或者一个设置对象。在任何例子中如果不用`lang.mixin`，复制属性得像下面这样：
```
var formData = domForm.formToObject(dom.byId('form'));
formData.name = currentUser.name;
formData.phone = currentUser.phone;
formData.address = currentUser.address;
formData.city = currentUser.city;
formData.province = currentUser.province;
formData.country = currentUser.country;
formData.postalCode = currentUser.postalCode;
```
这样从一个对象复制数据到另一个需要输入很多内容。假设第二个对象的属性的都要复制到第一个里，`lang.mixin`可以很简单：

```
var formData = domForm.formToObject(dom.byId("form"));
lang.mixin(formData, currentUser);
```
注意这个调用的返回值被丢弃了。因为`lang.mixin`直接作用在传入的第一个对象里，所以我们不用再手动更新`formData`变量。
`lang.mixin`也可以同时混入多个对象。例如，如果你设置的对象有一个原型，在它创建新实例时经常需要包含默认的值，你可以这么写：

```
var defaultSettings = {
    useTheForce: true,
    isEvil: false,
    length: 75,
    color: "blue"
};

function Lightsaber(settings){
    // `defaultSettings` is first mixed into the blank object,
    // then `settings` is mixed into the blank object, overriding
    // any properties from `defaultSettings` without altering
    // the `defaultSettings` object
    this.settings = lang.mixin({}, defaultSettings, settings);
}

var darthsaber = new Lightsaber({
    isEvil: true,
    color: "red"
});

// { useTheForce: true, isEvil: true, length: 75, color: "red" }
console.log("darthsaber:", darthsaber.settings);
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/augmenting_objects/demo/mixin.html)

##declare.safeMixin
`declare.safeMixin`主要完成`lang.mixin`相同的任务，但有三点不同：
 - 它一直只能混入一个对象
 - 它不能混入`constructor`属性
 - 它会为函数添加注释便于`declare`的`this.inherited`功能属性正常功能

尽管如名称暗示的没有原始`lang.mixin`的不安全成分，但除了：
 - 你在往`declare`创建的对象的一个实例里添加功能，并且：
 - 你依赖调用`this.inherited`，或者
 - 混入的一个或多个对象包含一个`constructor`属性。

因此，在`declare`过的类的实例的实例的情况之外，`lang.mixin`基本就够用了。
`declare`过的构造器有一个`extend`方法，它是将带有declare的构造器的原型作为第一个参数的`declare.safeMixin`。这个不要和下面要说的`lang.extend`混淆。

##lang.extend
`lang.extend`跟`lang.mixin`非常相似，处理它是给第一个对象的`prototype`添加属性而不是直接在对象本身添加。对于想要将改变立即在全部继承的实例上生效的情况，直接扩张对象的原型很有用：

```
// Assume Lightsaber is defined as in the previous example

var darthsaber = new Lightsaber({
    isEvil: true,
    color: "red"
});

var weaponMixin = {
    hp: 5,
    maxHp: 10,
    repair: function() {
        if(this.hp >= this.maxHp) {
            console.log("Can't repair!");
            return;
        }

        this.hp++;
    },
    swing: function() {
        if(!this.hp) {
            console.log("Weapon is broken!");
            return;
        }

        this.hp--;
        console.log(Math.random() >= 0.5 ? "hit!" : "miss!");
    }
};

lang.extend(Lightsaber, weaponMixin);

// Now we can call swing() on our Lightsaber instance,
// even though we augmented the prototype after creating the instance.
darthsaber.swing(); // "hit!" (or "miss!" if you are unlucky)
```
> [View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/augmenting_objects/demo/extend.html)

这比在创建的每一个新对象上调用`lang.mixin`更加高效，它只修改一个被继承的对象而不是几个子对象。它也比用`declare.safeMixin`更快。然而，当你需要对`this.inherited`调用的功能增加一个`declare`过的构造器功能，你应该使用`declare.safeMixin`或者构造器上的`extend`方法：
```
var Lightsaber = declare({
    constructor: function(settings){
        this.settings = lang.mixin({}, defaultSettings, settings);
    }
});

// same augmentation, but calls to this.inherited won't break:
Lightsaber.extend(weaponMixin);
```
注意使用`lang.extend` 或者`declaredConstructor.extend`有效修改构造器的原型会有问题，它会影响所有创建的示例。因此，将这些功能的使用限制在你确认合适的地方非常重要，不然就会引起不必要的副作用。尤其是，记住当创建自定义的out-of-the-box 组件时，推荐使用`declare`创建派生，而不是扩张已有的原型。

重要的是，注意所有的混入函数执行浅拷贝。就是说以下是正确的：
重要的是要注意我们引入的混入函数执行浅拷贝。例如：

```
var a = {
    name: "a",
    subObject: {
        foo: "bar"
    }
};
var b = lang.mixin({}, a);

b.name = "b";
b.subObject.foo = "baz";

console.log("a b, as expected:",
    a.name, b.name);
console.log("true - both subObjects reference the exact same object:",
    a.subObject === b.subObject);
console.log("baz baz - a change to one subObject affects both:",
    a.subObject.foo, b.subObject.foo);
```
在简单的例子中，这种行为不是问题，甚至在某些情况下很合适。然而在你不想要共享对象的地方，你可以使用一个`lang.clone`进行深拷贝。

```
var a = {
    name: "a",
    subObject: {
        foo: "bar"
    }
};
var b = lang.clone(a);

b.name = "b";
b.subObject.foo = "baz";

console.log("a b, same as before:",
    a.name, b.name);
console.log("false - the subObjects are different now:",
    a.subObject === b.subObject);
console.log("bar baz - a change to one subObject no longer affects all:",
    a.subObject.foo, b.subObject.foo);
```
>[View Demo](https://dojotoolkit.org/documentation/tutorials/1.10/augmenting_objects/demo/clone.html)

记住深拷贝明显要比浅拷贝慢，而且用在递归数据结构上会引起脚本挂起。深拷贝应该只在绝对必须的时候使用。

##小结
Dojo简化了创建和扩张对象和类的过程。`lang.mixin`和`declare.safeMixin`方法为在一个对象上添加和瞎改属性提供了便捷的方法，`lang.extend`让修改对象原型变得更加容易。请记住，这些函数执行浅拷贝。

##dojo/_base/lang资源
想了解 `lang.mixin`、 `lang.extend`、 `declare.safeMixin` 和`lang.clone`的更多细节？查看这些资源：

 - [lang.mixin](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/lang.html#dojo-base-lang-mixin)
 - [lang.extend](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/lang.html#dojo-base-lang-extend)
 - [lang.clone](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/lang.html#dojo-base-lang-clone)
 - [declare.safeMixin](https://dojotoolkit.org/reference-guide/1.10/dojo/_base/declare.html#dojo-base-declare-safemixin)
