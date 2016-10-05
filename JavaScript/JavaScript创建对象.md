# JavaScript创建对象

虽然`Object`构造函数或对象字面量可以用来创建单个对象，但是使用同一个接口创建多个对象时，会产生大量重复的代码。本文将介绍JavaScript中创建多个对象的几种方式，及其优缺点。

- 工厂模式
- 构造函数模式
- 原型模式
- 组合使用构造函数模式和原型模式
- 动态原型模式
- 寄生构造函数模式
- 稳妥构造函数模式



## 工厂模式

使用函数来模拟类，用函数来封装使用特定接口来创建对象。

```javascript
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        return this.name;
    };
    
    return o;
}

var person1 = createPerson("Tom", 39, "Engineer");
var person2 = createPerson("Greg", 27, "Doctor");

```

可以无数次调用上面的代码，每次都会返回一个包含三个属性和一个方法的对象。

> 工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（即怎样知道一个对象的类型）

## 构造函数模式

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
        return this.name;
    };
    
}
var person1 = new Person("Tom", 39, "Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```

> 按照惯例，构造函数始终都应以一个大写字母开头，而非构造函数则应以一个小写字母开头，主要是为了区别JS中的其他函数：因为构造函数本身也是函数，只不过可以用来创建对象而已。

要创建Person的新实例，必须使用new操作符。以这种方式调用构造函数会经历一下四个步骤：

1. 创建一个新对象；
2. 将构造函数的作用域赋给新对象(因此`this`就指向了这个新对象);
3. 执行构造函数中的代码(为这个新对象添加属性)；
4. 返回新对象。

以该方式创建的对象，可以使用`instanceof`来检测对象类型。在上例中创建的对象既是`Object`的实例，也是`Person`的实例。

```javascript
alert(person1 instanceof Object); // true
alert(person1 instanceof Person); // true
alert(person2 instanceof Object); // true
alert(person2 instanceof Person); // true
```

> 创建自定义的构造函数意味着将来可以将它的实例识别为一种特定的类型。

### 构造函数模式的问题

1. 将构造函数当成函数来调用

    构造函数和其他函数唯一的区别，就是可以使用`new`操作符来调用；而构造函数可以普通函数，也可以像普通函数那样调用。
    
    ```javascript
    // 当做构造函数调用
    var person = new Person("Greg", 27, "Doctor");
    person.sayName(); //Greg
    
    //作为普通函数调用
    Person("Greg", 27, "Doctor"); // 添加到window
    window.sayName(); // Greg
    
    // 在另一个对象的作用域中调用
    var o = new Object();
    Person.call(o, "Greg", 27, "Doctor");
    o.sayName(); // Greg
    
    ```
    
    > 避免直接调用构造函数，这样，属性和方法都会被添加到`window`上，从而污染全局变量，而且调用还没有返回值，造成对象构建失败。

2. 构造函数的另一个问题，就是方法的复用性。每个方法都要在每个实例上重新创建一遍，方法没有得到复用。

    由于JS中的函数是对象，因此每定义一个函数，也就是实例化了一个对象。所以上例中，每个Person对象都包含一个不同的`sayName`方法。因此不同实例上的同名函数是不同的。
    
    `alert(person1.sayName == person2.sayName); // false`
    
    可以通过把函数定义转移到构造函数外部来解决这个问题。
    
    ```javascript
    function Person(name, age, job) {
        this.name = name;
        this.age = age;
        this.job = job;
        this.sayName = sayName;
    }
    
    function sayName() {
        return this.name;
    }
    
    var person1 = new Person("Tom", 38, "Engineer");
    var person2 = new Person("Greg", 39, "Doctor");
    ```
    每个person实例的`sayName`都指向同一个函数，共享了在全局变量中定义的同一个`sayName()`函数。
    
    > 这样做也会存在问题，在全局作用域中定义的函数只能被某个对象使用，这样有点名不副实。如果对象需要定义很多方法，那么就要定义很多个全局函数，这样我们定义的引用类型就没有封装性可言了。
    

## 原型模式

每个函数都有一个`prototype`(原型)属性，该属性是一个指针，指向一个对象，这个对象的用途是可以包含由特定类型的所有实例共享的属性和方法。

```javascript
function Person() {
}

Person.prototype.name = 'Tom';
Person.prototype.age = 29;
Person.prototype.job = "Engineer";
Person.prototype.sayName = function() {
    return this.name;
};

var person1 = new Person();
person1.sayName(); // Tom

var person2 = new Person();
person2.sayName(); // Tom

console.log(person1.sayName === person2.sayName); // true
```

原型对象省略了构造函数传递初始化参数这一环节，结果所有的实例在默认的情况下都将取得相同的属性值。原型中多有的属性都是被实例共享的，这种共享对于函数非常合适，对于那些包含基本值的属性也说的过去。但是，对于包含引用类型的属性来说，就存在问题。

```javascript
function Person() {
}

Person.prototype = {
    constructor: Person,
    name: "Tom",
    age: 29,
    job: "Doctor",
    friends: ["Greg", "Lily"],
    sayName: function() {
        return this.name;
    }
};

var person1 = new Person();
var person2 = new Person();

person1.friends.push("Van");

console.log(person1.friends); // ["Greg", "Lily", "Van"]
console.log(person2.friends); // ["Greg", "Lily", "Van"]
console.log(person1.friends === person2.friends); // true

```

上述代码中，创建了两个Person的实例，这两个实例的friends属性指向了同一个数据，即`Person.prototype.friends`，person1中对friends的修改同样也反应在person2中。

> 实例一般都是要有属于自己的全部属性的。

即我们需要一种方法来创建对象，保证所有的实例中，属性是各自的，方法是共享的。

## 组合使用构造函数模式和原型模式

创建自定义类型的最常见的方式，就是组合使用构造函数模式与原型模式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限制的节省了内存。

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

Person.prototype = {
    constructor: Person,
    sayName: function() {
        return this.name;
    }
}

var person1 = new Person("Tom", 29, "Engineer");
var person2 = new Person("Greg", 29, "Doctor");

person1.friends.push("Van");
console.log(person1.friends); // ["Shelby", "Court", "Van"]
console.log(person2.friends); // ["Shelby", "Court"]
console.log(person1.friends === person2.friends); // false
console.log(person1.sayName === person2.sayName); // true
```

上述代码中，`person1`和`person2`拥有各自的`friends`属性，对其进行修改时，互不影响。

> 组合模式使用到了构造函数和原型模式的有点，避免了各自的问题，是目前JS中使用最广泛、认同度最高的一种创建自定义类型的方法。

## 动态原型模式

动态原型模式将所有的信息都封装在构造函数中，通过在构造函数中初始化原型（仅在必要的情况下），又保存了同时使用构造函数和原型的有点。

```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    
    // 方法
    if(typeof this.sayName != "function") {
        Person.prototype.sayName = function() {
            return this.name;
        };
    }
}
var person = new Person("Tom", 29, "Doctor");
person.sayName();

var person2 = new Person("Greg", 29, "Doctor");
person.sayName();

console.log(person.sayName === person2.sayName);
```

上述代码中的方法不封，只有在`sayName`不存在的情况下，才会将它添加到原型中。这段代码只有在第一次调用构造函数时，才会执行。

> 对原型所做的修改，都会立即在所有实例中得到反应。

## 寄生构造函数模式

寄生函数构造模式的基本思想是创建一个函数，该函数的作用仅仅是封装创建对象的代码，然后返回新创建的对象。

```javascript
function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        return this.name;
    };
    return o;
}
var friend = new Person("Tom", 29, "Engineer");
friend.sayName();
```
该模式除了使用`new`操作符并把使用的包装函数叫做构造函数之外，和工厂模式其实是一样的。（也可以不使用`new`操作符）

> 该模式中，返回的对象与构造函数或者与构造函数的原型属性没有任何关系；因此，不能依赖`instanceof`操作符来确定对象的类型。因此，这种模式并不推荐。

## 稳妥构造函数模式

所谓**稳妥对象**，指的是没有公共属性，而且其方法也不引用`this`的对象。适合在一些安全环境，或者防止数据被其他应用程序修改时使用。

稳妥构造函数遵循于构造函数类似的模式，但有两点不同：一是新创建的对象的实例方法不引用`this`；二是不使用`new`操作符构造函数。

```javascript
function Person(name, age, job) {
    // 要返回的对象
    var o = new Object();
    
    //可以在这里定义私有变量和函数
    
    //方法
    o.sayName = function() {
        return name;
    };
    
    return o;
}

var friend = Person("Tom", 39, "Engineer");
friend.sayName();

```

> 注意，在这种模式创建的对象中，除了使用sayName()方法外，没有其他方法能够访问name的值。

变量friend中保存的是一个稳妥对象。即使有其他代码会给这个对象添加方法或者数据成员，但也不可能有别的方法访问传入到构造函数中的原始数据。稳妥构造函数模式提供的这种安全性，使得他非常适合在某些安全执行环境下使用。

