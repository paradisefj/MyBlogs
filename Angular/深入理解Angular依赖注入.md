翻译自：[Understanding Dependency Injection](https://github.com/angular/angular.js/wiki/Understanding-Dependency-Injection)

AngularJS依赖注入非常有用，并且是创建可测试组件的关键。本文解释了AngularJS依赖注入如何工作的。

#**The Provider (`$provide`)**
----

`$provide`负责告诉Angular如何创建新的可注入的事物，即服务。服务是被providers定义的，即当你用`$provide`创建的。通过`$provide`服务来定义provider，你可以通过在应用的配置函数中注入`$provide`来持有该服务。如下：

```
myMod.config(function($provide) {
  $provide.provider('greeting', function() {
    this.$get = function() {
      return function(name) {
        alert("Hello, " + name);
      };
    };
  });
});
```

这里我们定义了一个名为`greeting`的provider；我们可以在任何函数中（例如controllers）注入一个名为`greeting` 的变量，Angular会调用这个provider 的$get方法来返回该服务的一个实例。在上面的例子中，注入了一个函数，该函数有一个参数`name`，弹出了一个和`name`有关的消息框。我们可以像下面这样使用它：

```
myMod.controller('MainController', function($scope, greeting) {
  $scope.onClick = function() {
    greeting('Ford Prefect');
  };
});
```

现在有一个诀窍：`factory`, `service`, `value`都是定义provider的快捷方式--即它们提供了不同的方式来定义provider而不用使用很长的定义。例如，你可以像下面那样定义一个和上例一样的provider

```
myMod.config(function($provide) {
  $provide.factory('greeting', function() {
    return function(name) {
      alert("Hello, " + name);
    };
  });
});
```

理解这个很重要，因此我换一种说法：在底层，不论我们写了上面的哪一种方法，AngularJS调用了同样的代码。字面上的意思是：上面的两种版本没有任何差别。`value` 也是同样工作的--如果我们在`$get`方法（也就是`factory`方法） 中返回的一直相同，我们可以通过使用`value` 方法来写更少的代码。例如，因为在`greeting` 中，我们一直返回了同样的函数，我们可以通过`value` 来这样定义它：

```
myMod.config(function($provide) {
  $provide.value('greeting', function(name) {
    alert("Hello, " + name);
  });
});
It'
```

再次强调，这种方法和以上两种100%一样，我们使用它定义方法仅仅是为了精简代码。

你可能注意到了在`myMod.config(function($provide) { ... })`中我是用了匿名函数。既然通过上面的方法定义provider很常用，AngularJS直接在模块对象上暴露了`$provide`方法，来更大程度上精简代码。

```
var myMod = angular.module('myModule', []);

myMod.provider("greeting", ...);
myMod.factory("greeting", ...);
myMod.service("greeting", ...);
myMod.value("greeting", ...);
```

这和之前我们使用的`app.config(...)` 做了一样的事情。

到目前为止我跳过了`constant`。到目前为止，简单来说，它和`value`工作的很像。后面我们还会看到一些差别。

回顾一下，这些代码片段都做了同样的事情：

```
myMod.provider('greeting', function() {
  this.$get = function() {
    return function(name) {
      alert("Hello, " + name);
    };
  };
});

myMod.factory('greeting', function() {
  return function(name) {
    alert("Hello, " + name);
  };
});

myMod.service('greeting', function() {
  return function(name) {
    alert("Hello, " + name);
  };
});

myMod.value('greeting', function(name) {
  alert("Hello, " + name);
});
```

#**The Injector (`$injector`)**
----
injector 负责通过使用用`$provide`提供的代码创建服务实例。一旦你创建含有注入参数的函数，injector就开始工作了。每个AngularJS应用在启动的时候会创建一个`$injector`；你可以在任何可以注入的函数中用`$injector`注入它从而得到其实例（是的，`$injector`知道如果注入它自己）。

一旦你有了`$injector`，你可以调用它通过一个已定义的服务的名称取到该服务的实例。例如：

```
var greeting = $injector.get('greeting');
greeting('Ford Prefect');
```

injector也负责注入服务到函数中。例如：你可以使用injector的`invoke`方法将服务注入到函数中。


```
var myFunction = function(greeting) {
  greeting('Ford Prefect');
};
$injector.invoke(myFunction);
```

应该注意到`injector`只会创建一次某个服务的示例。然后，它通过服务的名称将provider返回值缓存起来，下次你请求该服务，你会得到同一个对象。

因此，显而易见地，你可以通过调用`$injector.invoke`**注入服务到任何函数中**。包括

- 控制器定义函数
- 指令定义函数
- 过滤器定义函数
- provider的`$get `方法（也就是服务定义函数）

因为`constants`和`values`总是返回一个固定值，它们不是通过injector调用的，因此你不能给他们注入任何东西。

#**Configuring Providers**
----
你可能会有疑问，既然`factory`、`value`等方法如此方面，为什么有人还用provider的`provide`方法创建一个完整的provider。答案是：providers允许大量的配置。我们之前提到当你通过provider创建服务（或者通过Angular提供的任何快捷方式），你创建了该服务的构造函数。我没有提到的是这些服务可以被注入到你的应用的config中，从而你可以和他们交互。

首先，Angular运行你的应用程序在两个阶段－－`config`和`run`运行阶段。配置阶段，上面我们看到的，你可以设置任何需要的provider。这也是指令、控制器、过滤器设置的地方。运行阶段，你可能猜测，是Angular编译DOM然后开始运行应用的地方。

你可以通过`myMod.config`和`myMod.run`方法在这两个阶段中添加额外的代码运行。正如我们在第一个章节中看到的，这些函数是可以被注入的－－在第一个例子中我们注入了内置的\$provide服务。然而，值得注意的是，**在配置阶段，只有provider可以被注入**（`$provide` 和 `$injector`例外）。

例如下面的例子中，是错误的：

```
myMod.config(function(greeting) {
  // 不会工作 －－ greeting是某个服务的实例
  // 只有服务的provider才可以被注入到config块中
});
```

你能获取到的是你创建的服务的provider，如下：

```
myMod.config(function(greetingProvider) {
  // a-ok!
});
```
有一个很重要的例外：`constant`，既然它们不能够改变，被允许注入到配置块中（这是它们和`value`的差别）。可以通过它们的名字获取到（不用加`Provider`前缀）


当你给你的服务定义了一个provider，这个provider会被命名为`serviceProvider`，service即服务的名称。我们可以使用这个来做一些更复杂的事情。

```
myMod.provider('greeting', function() {
  var text = 'Hello, ';

  this.setText = function(value) {
    text = value;
  };

  this.$get = function() {
    return function(name) {
      alert(text + name);
    };
  };
});

myMod.config(function(greetingProvider) {
  greetingProvider.setText("Howdy there, ");
});

myMod.run(function(greeting) {
  greeting('Ford Prefect');
});
```

上例中，在provider中有一个名为`setText`的函数来定制`alert`；我们可以在配置块中获取到provider，调用该函数来定制服务。当我们最终运行我们的应用的时候，我们可以得到`greeting`服务，调用它从而发现我们的定制起作用了。

完整的例子请参考：http://jsfiddle.net/BinaryMuse/9GjYg/

#**Controllers ($controller)**
----
你可以注入事物到控制器中，但是你不能把控制器注入到其他事物中。这是因为控制器不是通过`provider`创建的，而是通过一个名为`$controller`的Angular内置服务，该服务负责创建控制器。当你调用了`myMod.controller(...)`，你获取了该服务的provider，类似于上个章节所描述的。

例如，我们定义了一个控制器：

```
myMod.controller('MainController', function($scope) {
  // ...
});
```

你实际上正在做这样的操作：

```
myMod.config(function($controllerProvider) {
  $controllerProvider.register('MainController', function($scope) {
    // ...
  });
});
```

随后，当Angular创建控制器的实例的时候，它使用了`$controller`服务（它反过来会使用`$injector`来调用你的控制器，从而得到其依赖）。

#**Filters and Directives**
----

`filter`和`directive`工作的方式和 `controller`一样；`filter`使用一个名为`$filter`的服务，它的provider名为`$filterProvider`；`directive`使用一个名为`$compile`的服务，它的provider名为`$compileProvider`。一些链接如下：

- \$filter: http://docs.angularjs.org/api/ng.$filter
- \$filterProvider: http://docs.angularjs.org/api/ng.$filterProvider
- \$compile: http://docs.angularjs.org/api/ng.$compile
- \$compileProvider: http://docs.angularjs.org/api/ng.$compileProvider

#总结
----
总结一下，任何被`$injector.invoke`调用的函数都可以被注入。包含但不局限于以下：

- controller
- directive
- factory
- filter
- provider $get （当定义provider为对象时）
- provider function （当定义provider为构造函数时）
- service

provider创建了新的可以注入到其他事物中的服务。包括：

- constant
- factory
- provider
- service
- value

也就是说，内置服务（例如`$controller`和`$filter`）可以被注入，你可以通过这些服务来创建新的控制器和过滤器（即使你没有定义它们，它们也是能够被注入的）。

除了这个，任何injector触发的函数可以被注入provider提供的服务－－这是没有限制的（除了配置和运行处的差异）。

改编自：[Stack Overflow 回答](http://stackoverflow.com/a/16829270/62082)

可以关注AngularJS公众号
![AngularJS公众号](http://img.blog.csdn.net/20160227213402563)