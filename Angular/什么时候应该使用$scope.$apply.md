翻译自：[When to use \$scope.\$apply()](https://github.com/angular/angular.js/wiki/When-to-use-%24scope.%24apply%28%29)

AngularJS对JS原生的异步事件进行了封装：

- Events => `ng-click`
- Timeouts => `$timeout`
- jQuery.ajax() => `$http`

这是一个传统的异步函数，仅仅在最后调用了`$scope.$apply()` 来通知AngularJS异步事件正在发生。

**`$scope.$apply()`应该在尽可能接近异步事件绑定的地方被调用。**

**`$scope.$apply()` should occur as close to the async event binding as possible.**

不要随意的在你的代码中使用它，如果你使用了`if (!$scope.$$phase) $scope.$apply()`，因为你没有处在调用栈的高层。

尽可能使用AngularJS的服务来代替原始的JS。如果你在创建一个AngularJS服务（例如为套接字创建服务），你应该在触发回调的任何位置都使用`$scope.$apply()`

**注：**不知道翻译的对不对，附上后两段的原文：

Do **NOT** randomly sprinkle it throughout your code. If you are doing
`if (!$scope.$$phase) $scope.$apply()` it's because you are not high enough in the call stack.

Whenever possible, use AngularJS services instead of native. If you're creating an AngularJS service (such as for sockets) it should have a `$scope.$apply()` anywhere it fires a callback.

**注：**其实我没有太明白作者想要表达的意图，个人觉得下面这篇文章写的更详细且更符合实际：[深入理解ANGULAR中的\$APPLY()以及\$DIGEST()](http://www.cnphp6.com/archives/64167)
