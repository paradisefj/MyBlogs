翻译自：[Understanding Scopes](https://github.com/angular/angular.js/wiki/Understanding-Scopes#directives)

**摘要**

在AngularJS中，子作用域通常会原型继承于其父作用域。有一个例外是当指令使用`scope: { ... }`来定义－－这创建了一个没有原型继承的“独立“作用域，这会在创建“可重复使用的组件“的指令时经常使用。如果你设置了`scope:true`（而不是`scope: { ... }`），这个指令会使用原型继承。

通常情况下作用域继承非常直白，你甚至不需要知道它正在发生。。。直到在一个定义在父作用域上原始类型（例如：number, string, boolean）在子作用域中使用了**双向数据绑定**（即：表单元素，ng-model）。这并不会像大多数人期望的那样工作，而是子作用域得到了它自己的属性，从而覆盖了父作用域上的同名属性。这不是AngularJS做的事情－这是JavaScript的原型继承起作用了。新入门的AngularJS开发者通常情况下不会意识到ng-repeat、 ng-switch、 ng-view 和 ng-include都创建了新的子作用域，所以当使用这些指令的时候，经常会有这种问题发生。

关于原始类型的这个问题通过下面的这个最佳建议很容易避免：[在你的模型中始终使用'.' ](http://www.youtube.com/watch?v=ZhfUv0spHCY&feature=youtu.be&t=30m)

在模型中使用'.' 会确保原型继承始终发生。所以，使用代码`<input type="text" ng-model="someObj.prop1">`而不是`<input type="text" ng-model="prop1">`。

如果你必须要使用原始类型，有以下两种解决方法：


- 在子作用域上使用$parent.parentScopeProperty。这会阻止子作用域创建自己的属性。
- 在父作用域上定义一个函数，在子作用域上调用，通过该函数传递父作用域上的原始值。

#JavaScript原型继承

----------


首先对JavaScript原型继承有一个深入的了解很有必要，尤其你具有服务器端开发的背景，并且对于传统的继承很熟悉。让我们先来复习一下。

假设parentScope具有如下属性， aString, aNumber, anArray, anObject, and aFunction。如果childScope 原型继承于parentScope，如下：

![图1](https://camo.githubusercontent.com/85ec776a0dd4acbe687f3db6367fa56872abb87f/687474703a2f2f692e737461636b2e696d6775722e636f6d2f61544147672e706e67)

（注意：为了节省空间，我把anArray展示成一个蓝色的有三个值的对象，而不是一个蓝色的拥有三个分离的灰色的对象）

如果我们在childScope上获取parentScope上定义的对象，JavaScript会首先在childscope上查找，没有找到该属性，查找其继承的scope，找到这个属性（如果在parentScope上没有找到该属性，会继续查找原型链。。。直到到达root scope）。所以，以下全都为真：

```
childScope.aString === 'parent string'
childScope.anArray[1] === 20
childScope.anObject.property1 === 'parent prop1'
childScope.aFunction() === 'parent output'
```
假设我们有如下代码：
```
childScope.aString = 'child string'
```
原型链并没有被遍历，一个新的属性会被添加到childScope上。同时，这个新属性隐藏了和parentScope具有同样名称的属性。这对我们下面讨论ng-repeat 和 ng-include非常重要

![图2](https://camo.githubusercontent.com/be4cd25951780638cf181b743b723ba6fb6ddb47/687474703a2f2f692e737461636b2e696d6775722e636f6d2f4f795650572e706e67)

假设我们又做了如下操作
```
childScope.anArray[1] = 22
childScope.anObject.property1 = 'child prop1'
```

原型链被访问了，因为对象（anArray和anObject）在childScope中没有被找到。这两个对象在parentScope中被找到了，属性的值在原对象上被更新了。childScope上不会增加新的属性，没有新的对象被创建（注意：在JavaScript中数组和函数同样是对象）

![图3](https://camo.githubusercontent.com/57f1938d241122bf49583ed64ee093e45e5dd012/687474703a2f2f692e737461636b2e696d6775722e636f6d2f32516365552e706e67)

假设我们做如下操作：
```
childScope.anArray = [100, 555]
childScope.anObject = { name: 'Mark', country: 'USA' }
```

原型链不会被访问，childScope会创建两个新的对象属性，隐藏了和parentScope具有相同名称的属性。

![图3](https://camo.githubusercontent.com/4679189c134e25e7dd7fcee18bc676cf9ec6b295/687474703a2f2f692e737461636b2e696d6775722e636f6d2f684b6574482e706e67)


**重要结论：**

 - 如果我们读取childScope的某个属性childScope.propertyX，
   并且childScope具有属性propertyX，那么原型链不会被访问。
 - 如果我们设置childScope的某个属性propertyX，那么原型链不会被访问。

最后一个场景：
```
delete childScope.anArray
childScope.anArray[1] === 22  // true
```
首先我们删除了childScope的属性anArray，然后我们尝试再次去获得该属性，原型链被访问了。

![图4](https://camo.githubusercontent.com/7f2d4d76e472463fa2980802a08cb3bedca7e0cd/687474703a2f2f692e737461636b2e696d6775722e636f6d2f3536756f652e706e67)

这个[jsfiddle](http://jsfiddle.net/5qjLd/)中你可以看到JavaScript原型继承的例子和结果（打开你的浏览器的控制台查看输出）

#Angular Scope 继承

----------

- 如下的指令创建了新的scope，并且基于原型继承：ng-repeat, ng-include, ng-switch, ng-view, ng-controller，使用scope:true的指令，使用transclude: true的指令。

- 如下的指令创建了新的scope，并且没有基于原型继承：使用scope: { ... }的指令。这创建了“孤立”的scope
（注意： 默认情况下，指令不创建新的scope，默认值是scope: false)

##ng-include
假设我们的controller中的代码如下：
```
$scope.myPrimitive = 50;
$scope.myObject    = {aNumber: 11};
```
HTML 如下:
```
<script type="text/ng-template" id="/tpl1.html">
    <input ng-model="myPrimitive">
</script>
<div ng-include src="'/tpl1.html'"></div>

<script type="text/ng-template" id="/tpl2.html">
    <input ng-model="myObject.aNumber">
</script>
<div ng-include src="'/tpl2.html'"></div>
```

每一个ng-include生成了一个新的基于其父作用域原型继承的子作用域。

![图5](https://camo.githubusercontent.com/67fc2d40487725fde10b669426c8b6b74213e6c6/687474703a2f2f692e737461636b2e696d6775722e636f6d2f7a694466782e706e67)

修改第一个textbox中的值为‘77’会导致子作用域创建一个新的myPrimitive 属性，并且隐藏了父作用域的同名属性。这可能不是你所希望的。

![图6](https://camo.githubusercontent.com/f1c9d54bd5b13d1e479b41ca6062b4b9fecc8fe2/687474703a2f2f692e737461636b2e696d6775722e636f6d2f376c3864672e706e67)

修改第二个textbox中的值为‘99’不会导致创建一个新的子属性。因为tpl2.html 绑定了一个对象的属性，当ngModel查找对象myObject时原型继承起作用了，最终在parentScope中找到了该属性。

![图7](https://camo.githubusercontent.com/5a6ff2644b1b7a15621c2a20928abfce0a2018bb/687474703a2f2f692e696d6775722e636f6d2f6f764a6547706f2e706e67)

如果不想将model从原始类型改为对象类型的话，我们可以使用$parent来重写第一个模板。

```
<input ng-model="$parent.myPrimitive">
```

这次修改第一个textbox的值不会导致生成一个新的子属性。模型现在绑定了parentScope中的属性（因为$parent是子作用域指向父作用域的一个引用）。

![图8](https://camo.githubusercontent.com/40767f9e9cc824e5c9ef178e385c9daa40ade6ba/687474703a2f2f692e737461636b2e696d6775722e636f6d2f6b6438706a2e706e67)

对于所有的作用域（不管是否是基于原型继承），Angular会通过作用域上的属性\$parent, \$\$childHead 和 \$$\childTail 始终跟踪其父-子关系（即层次结构）。在图中我并没有展示出这些属性。

对于不涉及表单元素的情况，另一个解决方案是在父作用域中定义一个函数来修改原始数据类型。然后保证子作用域总是调用这个函数，由于原型继承子作用域能够访问到该函数。例如，

```
// in the parent scope
$scope.setMyPrimitive = function(value) {
    $scope.myPrimitive = value;
}
```

这是一个使用了“parent function”的简单的[jsfiddle](http://jsfiddle.net/mrajcok/jNxyE/)（部分来自于[Stack OverFlow](http://stackoverflow.com/a/14104318/215945)）
还可以参考 http://stackoverflow.com/a/13782671/215945 和
https://github.com/angular/angular.js/issues/1267.

##ng-switch

ng-switch scope 的继承和ng-include类似。因此如果你需要**双向数据绑定**到父作用域中的一个原始数据类型上，使用$parent或者将model改为对象的某个属性。这会避免子作用域隐藏了父作用域的属性。

可以查看[Stack Overflow](http://stackoverflow.com/questions/12405005/angularjs-bind-scope-of-a-switch-case/12414410)

##ng-repeat

ng-repeat 和以上指令有点差别。假设我们的controller如下：

```
$scope.myArrayOfPrimitives = [ 11, 22 ];
$scope.myArrayOfObjects    = [{num: 101}, {num: 202}]
```

HTML 如下:

```
<ul><li ng-repeat="num in myArrayOfPrimitives">
       <input ng-model="num"></input>
    </li>
</ul>
<ul><li ng-repeat="obj in myArrayOfObjects">
       <input ng-model="obj.num"></input>
    </li>
</ul>
```
对于每次 item的iteration，ng-repeate创建了一个从父作用域原型继承的新的作用域，但是**它也将item的值分配给新的子作用域上的一个新的属性**（新的属性的名称是循环变量的名称）。如下是ng-repeate的源代码。

```
childScope = scope.$new(); // child scope prototypically inherits from parent scope ...     
childScope[valueIdent] = value; // creates a new childScope property
```

如果item是一个原始类型（例如上面的myArrayOfPrimitives），本质上该值的一个拷贝被分配给新的子scope。改变了子scope的属性值（即使用ng-model、也就是子scope属性num）并没有改变父scope引用的数组。所以，在上面第一个ng-repeate，每一个子scope会得到一个独立于myArrayOfPrimitives 的num属性。

![图9](https://camo.githubusercontent.com/3254baf91afdd969e6f167eeeb59950a0399a8f1/687474703a2f2f692e737461636b2e696d6775722e636f6d2f6e4c6f69572e706e67)

因此这个ng-repeat不会像你希望的那样工作。在Angular1.0.2（包含）以前，修改textbox的值会改变上图中灰色框的值，并且只在child scope中可见。在Angular 1.0.3以上，修改textbox的值不会有任何影响（参考Artem在[Stack Overflow](http://stackoverflow.com/a/13723990/215945)的解释）（此处说法有点不太准确，在较新的Angular版本中，修改textbox的值会改变图中灰色框中的值－－译者注）。我们所希望的是修改input的值能够改变数组myArrayOfPrimitives，而不是子scope的一个原始类型的属性。为了达到这个目的，我们需要将模型改为对象的数组（见第2个例子）。

因此，如果item是一个对象，原始对象的引用（非拷贝）会被分配成为新的子scope上的属性。修改子scope的属性值（例如，使用ng-model，obj.num）会**修改**父scope上的值。在上面的第二个ng-repeat中，我们有如下结论：

![图10](https://camo.githubusercontent.com/881318ad2d70364cf61d50faf536a7ce08f39777/687474703a2f2f692e737461636b2e696d6775722e636f6d2f51536a544a2e706e67)

（注意图中的灰线，能清楚的看到发生了什么）

按照预期工作了。修改textbox的值改变了灰色框中的值，同时对子作用域和父作用域都可见。

可以参考[ Difficulty with ng-model, ng-repeat, and inputs](http://stackoverflow.com/questions/13714884/difficulty-with-ng-model-ng-repeat-and-inputs) 和[ng-repeat and databinding](http://stackoverflow.com/a/13782671/215945) 

##ng-view

和ng-include类似

##ng-controller

和ng-include、ng-switch的原理一致，使用ng-controller的嵌套的控制器会引起正常的原型继承。然而，**“不建议在两个控制器中通过$scope的继承关系来共享信息“**－－http://onehungrymind.com/angularjs-sticky-notes-pt-1-architecture/。在控制器中共享数据应该使用服务。

（如果你必须通过控制器的scope的继承关系共享数据，你不需要做任何操作。子scope能够取到父scope的所有属性。参考[Controller load order differs when loading or navigating](http://stackoverflow.com/questions/13825419/controller-load-order-differs-when-loading-or-navigating/13843771#13843771)

##指令
1. 默认（```scope:false```）－指令没有创建任何新的作用域，因此不存在任何的原型继承。这很简单，但是同样存在隐患，例如：一个指令可能以为它在作用域上创建了一个新的属性，但实际上它修改了一个现有的属性的值。这对于书写可重复使用的组件来说并不是一个好的选择。
2. ```scope: true```－指令创建了一个从父作用域基于原型继承的子作用域。如果在同一个DOM上有多个指令需要创建新的作用域，那么只有一个新的子作用域会被创建。既然有“正常“的原型继承，和ng-include 、ng-switch类似，警惕在父作用域上的原始数据类型的双向数据绑定，子作用域会覆盖掉父作用域上的属性。
3. ```scope: { ... }```－指令创建了一个新的独立作用域。并且没有原型继承。当你创建可以复用的组件时这是一个好的选择，因为指令不能够直接读取或修改父作用域。然而，通常这种指令需要读取父作用域的某些属性。该对象可以在父作用域和独立作用域上使用“＝“创建双向数据绑定，使用“@“创建**单向绑定**（父作用域改变会影响子作用域，子作用域改变并不会影响父作用域－－译者注）。也可以使用“&“绑定父作用域上的表达式。所以，这些方法同样给子作用域创建了从父作用域衍生的属性。注意这些属性被用来帮助设置绑定－－在对象中你不能直接引用父作用域的属性名称，你需要使用一个HTML属性。例如：如下，你想要在独立作用域上绑定父作用域的属性`parentProp`将不会起作用：代码`<div my-directive>` 和`scope: { localProp: '@parentProp' }`。指令想要绑定的父属性必须要有明确的HTML属性名：代码`<div my-directive the-Parent-Prop=parentProp>`和`scope: { localProp: '@theParentProp' }`。独立作用域的__proto__ 引用了一个Scope对象。独立作用域的$parent引用了父作用域，尽管这是一个没有原型继承的独立作用域，但他还是一个子作用域。
如下图片中：我们有代码`<my-directive interpolated="{{parentProp1}}" twoway－binding="parentProp2">` 和
`scope: { interpolatedProp: '@interpolated', twowayBindingProp: '=twowayBinding' } `。同样假设在指令的link函数中有代码`scope.someIsolateProp = "I'm isolated"`
![图11](https://camo.githubusercontent.com/0c650e5b62347beec5ebbb4990673a523a80968c/687474703a2f2f692e737461636b2e696d6775722e636f6d2f45586a5a712e706e67)
最后注意：使用link函数中`attrs.$observe('attr_name', function(value) { ... })`来得到独立作用域中使用‘@‘绑定的属性的值。例如：在link函数中有代码--`attrs.$observe('interpolated', function(value) { ... })` -- `value`会被设置为11。（`scope.interpolatedProp`在link函数中**没有定义**（该文章写的时间较早，译者通过测试Angular1.4.7发现在该版本中，这个属性已经有定义了，值为11）。而`scope.twowayBindingProp`有定义，因为他使用了‘＝‘ ）。
关于独立作用域的更多信息请查看：http://onehungrymind.com/angularjs-sticky-notes-pt-2-isolated-scope/
4. `transclude: true`－指令创建了一个新的 "transcluded" 子作用域，并且原型继承于父作用域。因此，如果你的嵌入的内容（即ng-transclude将被替换的内容）需要双向数据绑定到父作用域上的一个原始类型上，使用$parent，或者将模型改为对象，绑定到改对象的某个属性上。这会避免子作用域覆盖父作用域的属性。
内嵌作用域和独立作用域是同胞的－－每个scope的\$parent属性指向同一个父作用域。当内嵌作用域和独立作用域同时存在，独立作用域的\$\$nextSibling 属性会指向内嵌作用域。
关于内嵌作用域的更多信息，请查看[AngularJS two way binding not working in directive with transcluded scope](http://stackoverflow.com/a/14484903/215945) 
假设上面的指令增加了属性`transclude: true` ，scope的示意图如下：
![图12](https://camo.githubusercontent.com/4d9a7cbb029bb29d66cbbef0f0527b2d40202d90/687474703a2f2f692e737461636b2e696d6775722e636f6d2f41684f47482e706e67)

这个[jsfiddle](http://jsfiddle.net/mrajcok/7g3QM/)有一个用来检查独立作用域和他相关的内嵌作用域的`showScope()`函数。参考该fiddle中的注释中的说明。
#总结


----------
有四种类型的作用域：
1. 普通原型继承作用域－－ng-include、ng-switch、ng-controller和使用`scope: true`定义的指令
2. 含有拷贝属性的普通原型继承作用域－－ng-repeat。每次迭代ng-repeat都会创建一个新的子作用域，同时新的子作用域会得到一个新的属性。
3. 独立作用域－－使用`scope: {...}`定义的指令。这次没有原型继承，但是 '=', '@', and '&'提供了一种通过HTML属性获取父作用域属性的机制。
4. 内嵌作用域－－使用`transclude: true`定义的指令。这次依旧是正常的基于原型的继承，但是同时他也是任意独立作用域的兄弟作用域。

对于所有的作用域（不论是否原型继承），Angular总是通过\$parent、\$\$childHead 和\$\$childTail追踪父－子关系（即层级结构）