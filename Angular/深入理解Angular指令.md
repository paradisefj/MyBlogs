翻译自：[Understanding Directives](https://github.com/angular/angular.js/wiki/Understanding-Directives)

本篇文章旨在解释AngularJS指令和其相关的编译引擎是如何工作的，理解了以后你就不会像第一次吃面条那样手足无措。

#**Injecting, Compiling, and Linking functions**
----

当你创建指令的时候，本质上，有3个功能层级需要定义：（transcluding function是第四层，但是指令uiJq没有用到）

```
myApp.directive('uiJq', function InjectingFunction(){

  // === InjectingFunction === //
  // 对于每个应用，该逻辑会执行0或1次（取决于该指令是否被用到）
  // 对启动和全局配置很有用

  return {
    compile: function CompilingFunction($templateElement, $templateAttributes) {

      // === CompilingFunction === //
      // 在原始的未被渲染的模版中，对于每个ui-jq实例来说，该逻辑都会被执行一次
      // 在这里获取不到Scope，因为模板仅仅是被缓存起来了
      // 你可以查看DOM和将要使用变量或表达式的信息，但是你取不到他们的值。
      // Angular正在缓存模版，现在是一个很好的时机来注入新的模板作为孩子或未来的兄弟姐妹来自动运行。

      return function LinkingFunction($scope, $linkElement, $linkAttributes) {

        // === LinkingFunction === //
        // 对于每个渲染好的实例来说，该逻辑都会执行一次。
        // 对于ng-repeat创建的每一行都会执行一次（而compile在ng-repeat中只会执行一次）。
        // Note that ng-if or ng-switch may also affect if this is executed.
        // Scope能够被访问，因为控制器逻辑已经执行完毕。
        // 所有的变量和表达式值可以最终确定。
        // Angular正在渲染缓存好的模版。现在添加让angular自动运行的模版已经太晚了。如果你必须注入新的模板，请使用$compile。

      };
    }
  };
})
```

只能在link函数中取到`$scope`中的值。由于模版的逻辑可能删除活着复制元素，你只能依靠在LinkingFunction中最终生成的DOM结构。你不能依赖其子元素或者兄弟元素，因为在这时它们还没有进入到链接阶段。

#**Pre vs Post Linking Functions**
----
任何你使用`LinkingFunction()`的地方，你都可以选在一个有pre和post属性的link function的对象。[奇怪的是](https://github.com/angular/angular.js/issues/2592)，默认情况下，一个 `LinkingFunction()` 是一个 `PostLinkingFunction()` ：

```
link: function LinkingFunction($scope, $element, $attributes) { ... }
...
link: {
  pre: function PreLinkingFunction($scope, $element, $attributes) { ... },
  post: function PostLinkingFunction($scope, $element, $attributes) { ... },
}
```

区别是`PreLinkingFunction()`会先在parent上触发，然后是child等等。而`PostLinkingFunction()`与之相反，会在child上先出发，然后是parent等等。示例：http://plnkr.co/edit/qrDMJBlnwdNlfBqEEXL2?p=preview

**我什么时候想要反转`PostLinking`的行为？**有时候jQuery 插件想要知道子DOM元素的个数和大小。有一些方法来支持这种：

- （最差）使用$timeout来延迟插件的执行
- 嵌套指令。如果每一个元素都包含一个指令，可以使用 `require: '^parentDirective'` 来得到`parentDirective` 的控制器。
 - 如果在`parentDirective`上使用`PreLinkingFunction()` ，你可以实例化容器为空，然后使用的时候每次更新它。

**这不适应于异步变化，例如通过ajax来加载$scope的数据。**

如果你需要等到$scope数据加载完成，使用ng-if来阻止DOM的链接。

#**\$element === angular.element() === jQuery() === \$()**
----
为了操作DOM的时候更容易，AngularJS封装了一个名叫jqlite的小型的jQuery。这使用和jQuery几乎相同的API用来模拟jQuery的一些核心功能。任何时候你看到AngularJS DOM元素，这将是一个jquery()封装的DOM元素。

**你不需要使用jQuery来包装AngularJS元素。**

如果你注意到所有jQuery方法（或者插件）不能在AngularJS元素上使用，你不是忘了加载jQuery库，就是忘了在AngularJS之前加载jQuery库，这种情况下Angular会只用自己的jqlite来替代jQuery。

#**\$attributes.\$observe()**
---

如果你有一个包含`{{}}`的兄弟属性，这个属性需要被计算，而且可能改变多次。**不要手动做这个操作。**

用 `$attributes.$observe('myOtherAttribute', function(newValue))`代替`$scope.$watch()`没有任何问题，唯一的区别是第一个参数时属性名（而不是表达式），回调函数只有一个新的值（已经计算好了的值）。每次计算结果改变都会触发回调。

**注意：**这意味着你能够*异步*的获取属性

**注意：**如果你想要*可靠*的获取未计算的属性值，你应该在CompileFunction中操作


#**Extending Directives**
----
如果你想要使用第三方的指令，你希望不做任何修改地扩展它。以下有几种方法你可以参考：

##**Global Configurations**

一些精心设计的指令（例如AngularUI中的指令）可以被全局配置，所以在每个实例中，你不需要每次传递你的配置项。

##**Require Directives**

创建一个新的指令，假定第一个指令已经被应用。你可以要求它在父DOM元素上，或在同一个DOM元素中。如果你需要另一个指令的某些方法，通过指令的控制器暴露给另一个指令。


```javascript
// <div a b></div>
ui.directive('a', function(){
  return {
    controller: function(){
      this.data = {}
      this.changeData = function( ... ) { ... }
    },
    link: ($scope, $element, $attributes, controller) {
      controller.data = { ... }
    }
  }
})
myApp.directive('b', function(){
  return {
    require: 'a',
    link: ($scope, $element, $attributes, aController) {
      aController.changeData()
      aController.data = { ... }
    }
  }
})
```

##**Stacking Directives**

你可以创建一个和原始指令一样名称的指令，这两个指令都会被执行。但是，你可以通过使用优先级来控制那个指令先被触发。

```javascript
// <div a></div>
ui.directive('a', ... )
myApp.directive('a', ... )
```

##**Templating**

你可以发挥`<ng-include>`的优势，或者创建一个新的指令，生成的HTML属性中包含需要的指令。

```javascript
// <div b></div>
ui.directive('a', ... )
myApp.directive('b', function(){
  return {
    template: '<div a="someOptions"></div>'
  }
})
```

##**Specialized the Directive Configuration**

有时你通过一个特定版本的指令创建一个新的指令，该指令之和原始指令有一点配置项的不同（例如templateUrl），同时让原始的指令仍能正常工作。任何注册的指令都会创建一个将“Directive”加在指令名称后面的特殊的服务。如果你注册了 `<my-dir>` 的指令（指令名称 'myDir'），这会创建一个名为‘myDirDirective’的服务。如果你将其注入到一个新的指令中，你会得到一个指令的配置的数组（可能以优先级排序）（如果有同名的指令，会得到所有同名指令的服务）。选择你需要的服务（通常是第一个），复制它，改变某个配置项，然后返回。

```javascript
// <div b></div>
ui.directive('a', ... )
myApp.directive('b', function(aDirective){
   return angular.extend({}, aDirective[0], { templateUrl: 'newTemplate.html' });
})
```