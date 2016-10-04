## js函数

在JavaScript中函数就是**对象**。对象是“名/值”对的集合并拥有一个连到原型对象的隐藏链接。
对象字面量产生的对象连接到`Object.prototype`。函数对象链接到`Function.prototype`，`Function.prototype`本身仍链接到`Object.prototype`。
每个函数在创建时附有两个附件的隐藏属性：**函数的上下文（this）**和实现函数行为的代码。

因为函数是对象，所以他们可以像任何其他的值一样被使用。函数可以存放在变量、对象和数组中。函数可以被当做参数传递给其他函数，函数也可以在返回函数。而且，因为函数是对象，所以函数可以拥有方法。

函数的与众不同之处在于他们可以被调用。

### 函数调用
调用一个函数将暂停当前函数的执行，传递控制权和参数给新的函数。
在JS中一共有**四种调用模式**：方法调用、函数调用、构造器调用和apply调用。
这些模式的不同在于初始化**this**上的差异
- **方法调用**
    当一个函数被保存为一个对象的属性时，称为一个**方法**。当一个方法被调用时，this绑定到该对象上。并且可以取到该对象的上下文中的方法和属性。
- **函数调用**
    当一个函数并非一个对象的属性时，那么他被当做一个函数来调用：
    ` var sum = add(3, 4) `
    > 当函数以此模式调用时，this被绑定到全局对象上。这是语言设计上的一个错误。
- **构造器调用**
    构造器函数存在一个严重的危害，如果在调用时忘记了在前面加上new，那么this将不会绑定到一个新对象上，this将会绑定到全局对象上，所以不但没有扩充新对象，反而将破坏全局变量。而且既没有编译警告，也没有运行警告。

### 如何将函数科里化？？

```javascript
Function.prototype.curry =  function(){
	var slice = Array.prototype.slice,
		args = slice.apply(arguments), //为了将arguments 转为数组
		that = this; //指代执行curry的函数对象
	return function() {
		return that.apply(null, args.concat(slice.apply(arguments)));
	};
};
```
如何使用上面函数？

```javascript
var add = function(a, b){return a+b;}
var add1 = add.curry(1);
add1(2);//3
```

> 注意：上述科里化的过程只能科里化一次，如果一个函数有多个参数，结果可能是先将n-m个函数传给curry，然后用最后m个参数来调用curry后的结果。而不像lodash的curry，能够每次都返回一个函数。建议使用lodash



