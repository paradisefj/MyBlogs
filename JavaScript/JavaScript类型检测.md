# JavaScript 类型检测

本文介绍JavaScript的几种类型检测的方法，以及其各自的适用范围。

## JavaScript 的数据类型

JavaScript中的数据类型有`null`、`undefined`、布尔值、字符串、数字、对象，这6种数据类型，其中前五种是基本数据类型，对象是复杂数据类型。

JavaScript中检测类型有如下几种方法：
1. `typeof`
1. `instanceof`
1. `constructor`
1. `duck type` (鸭子类型)

### typeof

`typeof`返回字符串，适用于**函数对象**和**基本类型**的检测（注：遇`null`失效），各种类型的返回值如下：

| 代码 | 结果 |
|------|------|
|`typeof 1`| "number" |
|`typeof true`| "boolean" |
|`typeof function`| "function" |
|`typeof [3,4,5] `| "object" |
|`typeof new Object()`| "object"|
|`typeof NaN`| "number" |
|`typeof undefined`| "undefined" |
|***`typeof null`***| ***"object"(`null`被认为是一个空的对象引用)***|

**不适用的范围：**

1. `typeof null`返回"object"，所以不适合判断`null`

如需判断某个对象是否是`null`，可以使用`obj === null`

## instanceof

instanceof 是一个二元**操作符**，左边是要判断的对象，右面是函数对象或者构造器

instanceof 是基于**原型链**进行判断，判断左边的原型链上是否有右边的类型（和Java的intanceof类型）

```javascript
function Dog() {}  
  
  
// Dog继承于Animal  
Dog.prototype = new Animal();  
Dog.prototype.constructor = Dog;  
  
  
var dog = new Dog();  
console.log(dog instanceof Dog); // true dog直接用Dog初始化，所以为真  
  
  
var cat = new Animal();  
console.log(cat instanceof Animal); // true cat直接用Animal初始化，所以为真  
  
  
console.log(cat instanceof Dog); //false   
console.log(dog instanceof Animal); //true  
```

在判断dog是否是Animal类型时，dog对象有一个原型`_proto_`，该原型会指向其构造器 Dog 的`_proto_`，但是Animal不等于Dog，所以该原型链会继续向上查找，Dog的`_proto_`指向了Animal，所以最终`dog instanceof Animal` 返回`true`

**不适用的范围：在多窗口和多框架的子页面的web应用中兼容性不好。**

**原因**：在两个不同框架页面创建的两个相同的对象继承与两个相同但互相独立的原型对象（例如：创建两个数组），其中一个框架页面中的数组不是另一个框架页面中的Array()构造函数的实例，instanceof 运算结果返回false。

## constructor

不适用范围：
1. 不适合判断null和undefined，因为没有contructor属性
2. contructor可能被改写
3. 和instanceof类型，在多窗口和多框架下无法工作
4. 并非所有的对象都包含contructor 属性

contructor的使用如下:

| 代码 | 结果 |
|------|------|
|`var x=1; x.constructor`| `Number`|
|`var x=true; x.constructor`|`Boolean`|
|`var x = function(){};x.constructor`|`Function`|
|`var x='string';x.constructor`|`String`|


注意：`null`和`undefined`没有`constructor`

## duck type

不要关注对象的类是什么，而是关注对象能做什么。

例如，我们在判断一个对象是否是Array的实例的时候，可以通过判断对象是否含有一个非负的length属性来判断，这是一个必要非充分条件。