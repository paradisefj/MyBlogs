翻译自：[Anti Patterns](https://github.com/angular/angular.js/wiki/Anti-Patterns)


 1. 不要用$()来包装`element`。所有的AngularJS 元素都已经是jquery对象。
 2. 不要使用`if (!$scope.$$phase) $scope.$apply()`，这意味着你的`$scope.$apply()`没有在调用栈足够高的位置。
 3. 不要使用jQuery生成模板或者DOM
 4. 不要重复创造轮子（先在互联网上查找是否有开源的控件）
 5. 不要在隔离作用域（例如 `ng-if`）中使用一个标量（null 是一个标量）作为模型（例子：http://embed.plnkr.co/qRhLfw/preview）