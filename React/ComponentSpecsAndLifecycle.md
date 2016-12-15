# React组件规范和生命周期

## 组件规范

当你通过调用`React.createClass()`创建一个组件类时，应该提供一个包含render方法和其他可选的生命周期方法的说明对象。

> 请注意：可以使用简单的JavaScript类作为组件类。尽管有所不同，但是这些类应该实现大多数相同的函数。关于这些不同的更多信息，请参考我们关于[ES6 class](http://reactjs.cn/react/docs/reusable-components.html#es6-classes)的文章。

### render

```javvascirpt
ReactElement render()
```

`render()`方法是必须的。

当调用该方法，他应该检查`this.props`和`this.state`，并且返回单一的子元素。子元素可以是一个真实的DOM组件（例如：`<div />`或者`React.DOM.div()`）的虚拟的代表，也可以是你自己定义的另一个符合组件。

你也可以返回`null`或者`false`来表明你不想任何组件被渲染。在这种情况下，React选软了一个`<noscript>`标记来适配我们当前的差异算法。当返回`null`或者`false`时，`ReactDOM.findDOMNode(this)`会返回`null`。

`render()`应该是纯函数，这意味着他不能修改组件状态，每次被调用应该返回同样的结果，它不应该读写DOM或者用其他的方式和浏览器交流（例如，使用`setTimeout`）。如果你需要作用于浏览器，请在`componentDidMount`或者其他声明周期中执行。保持`render()`作为一个纯函数能让服务端渲染更加实际，并且让组件更容易被理解。

### getInitialState

```javascript
object getInitialState()
```

在组件被加载之前触发一次。返回值会被当做组件初始的`this.state`的值。

### getDefaultProps 

```javascript
object getDefaultProps()
```

当类被创建的时候执行一次，并被缓存下来。当props没有被父组件指定时，映射中的值会被设置为`this.props`（即使用`in`检查）

该方法在所有实例被创建之前触发，因此不能依赖`this.props`。除此之外，请注意`getDefaultProps()`返回的任何复杂对象对象都会在所有实例中共享，而不是拷贝。

### propTypes

```javascript
object propTypes
```

`propTypes`对象允许在props在传递给组件之后做校验。关于`propTypes`的更多信息，请查看[Reusable Components](http://reactjs.cn/react/docs/reusable-components.html)

### mixins

```javascript
array mixins
```

`mixins`数组允许在多个组件之间共享行为。关于`mixins`的更多信息，请查看[Reusable Components](http://reactjs.cn/react/docs/reusable-components.html)

### statics

```javascript
object statics
```

`statics`对象允许定义被组件类调用的静态方法。例如：
```javascript
var MyComponent = React.createClass({
  statics: {
    customMethod: function(foo) {
      return foo === 'bar';
    }
  },
  render: function() {
  }
});

MyComponent.customMethod('bar');  // true
```

在该块内定义的方法是静态的，这意味着你可以在任意组件实例被创建之前调用该方法，该方法不能获取到组件的props或者state。如果在一个静态方法中你想要检查props的值，请调用者将props作为静态方法的参数传递。

### displayName

```javascript
string displayName
```

`displayName`字符创被用作调试信息。JSX自动设置该值；请查看[JSX in Depth](http://reactjs.cn/react/docs/jsx-in-depth.html#the-transform)


## 生命周期方法

在一个组件的生命周期中，不同的方法在不同的时间点被执行。


### Mounting: componentWillMount

```javascript
void componentWillMount()
```

被触发一次，在客户端和服务端都会被执行，在初次渲染发生之前立即被触发。如果在该方法中调用`setState`，`render()`会使用更新后的state，尽管state发生了改变，仍然只执行一次。

### Mounting: componentDidMount 

```javascipt
void componentDidMount()
```

只在客户端（非服务端）被触发一次，在初次渲染发生之后立即被处罚。在声明周期的该点，你可以获取到你的子节点的任何引用（例如：访问DOM）。子组件的`componentDidMount()`在父组件之前被调用。

如果你想要集成其他的JavaScript框架、使用`setTimeout`或者`setInterval`、发送AJAX请求，请在该方法中执行。

### Updating: componentWillReceiveProps

```javascript
void componentWillReceiveProps(
  object nextProps
)
```

每次组件接受到新的props都会被触发。在首次渲染时，该方法不会被调用。

在`render()`被调用之前，使用该函数当做props的过渡，并且通过调用`this.setState()`来更新state。旧的props可以通过`this.props`获取到。在该方法中调用`this.setState()`不会触发额外的渲染。

```javascript
componentWillReceiveProps: function(nextProps) {
  this.setState({
    likesIncreasing: nextProps.likeCount > this.props.likeCount
  });
}
```

> 请注意：一个很常见的错误是假设在该方法执行过程中props已经改变了。要立即这为什么无效，请查看[A implies B does not imply B implies A](http://reactjs.cn/react/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html)
> 并没有一个叫做`componentWillReceiveState`的类似方法。props转换可能引起state的改变，但是反过来并不成立。如果你需要在state改变时执行操作，请使用`componentWillUpdate`。

### Updating: shouldComponentUpdate

```javascript
boolean shouldComponentUpdate(
  object nextProps, object nextState
)
```

当接受到新的props或者state渲染之前被调用。在首次渲染或者调用`forceUpdate`时，该方法不会被调用。

当你确定新的props或者state不会引起组件更新时，返回`false`。

```javascript
shouldComponentUpdate: function(nextProps, nextState) {
  return nextProps.id !== this.props.id;
}
```

如果`shouldComponentUpdate`返回`false`，`render()`函数会被完全跳过，知道下一次state改变。除此之外，`componentWillUpdate`和`componentDidUpdate`也不会被调用。

默认情况下，当state发生变化`shouldComponentUpdate`永远返回`true`来阻止微妙的bug，但是如果你等将state处理为不可变的，在`render()`中仅仅读取props和state，然后可以通过比较旧的props和state来实现`shouldComponentUpdate`。

如果性能是一个瓶颈，特别是有成百上千个组件时，使用`shouldComponentUpdate`来加速应用。

### Updating: componentWillUpdate

```javascript
void componentWillUpdate(
  object nextProps, object nextState
)
```

当接受到新的props和state时，进行渲染之前立刻被调用。该方法在初次渲染时不会被调用。

使用此方法作为在更新发生之前执行准备的机会。

> 请注意：在该方法中不能使用`this.setState()`。如果你需要响应prop的改变，请使用`componentWillReceiveProps`。

### Updating: componentDidUpdate

```javascript
void componentDidUpdate(
  object prevProps, object prevState
)
```

当组件更新生成DOM之后立即被触发。该方法在首次渲染时不会被调用。

使用该方法作为组件更新后操作DOM的机会。

### Unmounting: componentWillUnmount

```javascript
void componentWillUnmount()
```

当一个组件被从DOM移除之后立即被触发。

在该方法中执行清理工作，例如删除在`componentDidMount`时添加的定时器和清理DOM元素。
