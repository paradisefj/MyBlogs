# 理解JS执行环境和作用域

## 执行环境及作用域链

执行环境（Context）是每中编程语言的一个重要概念，同时也是JavaScript种的一个重要概念，其定义了变量或函数有权访问的其他数据，决定了他们各自的行为。每个执行环境都有一个与之关联的**变量对象**，环境中定义的所有变量和函数都存在这个对象中。

每个函数都有自己的**执行环境**。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。

全局执行环境是最外围的一个执行环境。在浏览器中，为windows对象，NodeJS中，为global对象。所有的全局变量和函数都是作为全局对象的属性和方法创建的。某个执行环境中多有代码执行完毕，该环境被销毁，保存在其中的所有变量和函数定义也随之销毁。

当代码在一个环境中执行时，会创建变量对象的一个**作用域链（scope chain）**。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是函数，则将其**活动对象（activation object）**作为变量对象。活动对象最开始时，只包含一个变量，即`arguments`对象。作用域链中的下一个变量对象来自包含它执行的外部环境，而再下一个变量对象则来自下一个包含环境。这样，一直延伸到全局执行环境；全局执行环境始终都是作用域链中的最后一个对象。

标识符解析是沿着作用域链一级一级的搜索标识符的过程。搜索过程始终是从作用域链的前端开始，然后逐级的向后回溯，直到找到标识符为止。

### 例子 
```javascript
var color = "blue";

function changeColor() {
    var anotherColor = "red";
    
    function swapColors() {
        var tempColor = anotherColor;
        anotherColor = color;
        color = tempColor;
        
        // 这里可以访问 color, anotherColor, tempColor
    }
    
    // 这里可以访问color，anotherColor，但不能访问tempColor
    swapColors();
}

// 这里只能访问color
changeColor();

```

以上代码共涉及三个执行环境：全局环境、`changeColor()`的局部环境和`swapColor()`的局部环境。其中作用域链如下图。

<div style="width: 250px;">
    window
	<ul style="background-color: #999;list-style-type:none;">
		<li style="border-left:2px solid #000;">------color</li>
		<li style="border-left:2px solid #000;">------changeColor()</li>
		<ul style="background-color: #DDD;list-style-type:none;">
			<li style="border-left:2px solid #000;">----anotherColor</li>
			<li style="border-left:2px solid #000;">----swapColor()</li>
			<ul style="background-color: #999;list-style-type:none;">
				<li style="border-left:2px solid #000;">--tempColor</li>
			</ul>
		</ul>
	</ul>
</div>

如上去所示，内部环境可以通过作用域访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数。这些环境之间的联系是线性的、有次序的。每个环境都可以向上搜索作用域链，以查询变量和函数名；但任何环境不能通过向下搜索作用域链而进入另一个执行环境。

## 延长作用域链

有如下两种语句能够使作用域链得到延长：
- `try-catch`语句的`catch`块
- `with`语句

两个语句都会在作用域链的前端添加一个变量对象。对`with`来说，会将执行的对象添加到作用域链中。对`catch`语句来说，会创建一个新的变量对象，其中包含的是被抛出的错误对象的声明。

```javascript
function buildUrl() {
    var qs = "?debug=true";
    
    with(location) {
        var url = href + qs;
    }
    return url;
}
```

`with`语句接受的是`location`对象，因此在其变量对象中就包含了`location`对象的所有属性和方法，当在`with`语句中引用`href`时（实际上是访问`location.href`），可以在当前执行的环境的变量对象中找到。

> 由于`with`能造成代码的易读性降低，代码的效率降低，所以在代码中不建议使用`with`语句。