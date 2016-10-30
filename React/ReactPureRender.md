# React纯组件渲染性能反模式

React纯组件的渲染可以非常高效，但是需要用户将其数据作为不可变的对象，才能正常工作。但是由于JavaScript的原因，有时做到这点可能非常具有挑战性。

> 反模式是在`Render`函数或者Redux的`connect(mapState)`中创建新的数组、对象、函数或者其他新的对象

#### 纯渲染？

说起React的纯渲染，我指的是组件应该通过浅比较来实现`shouldComponentUpdate`方法。例如[PureRenderMixin](https://facebook.github.io/react/docs/pure-render-mixin.html)，[recompose/pure](https://github.com/acdlite/recompose#optimize-rendering-performance) ，等等

#### 为什么？

这可能是你能在React中做的最显著的一个性能优化。这也是ClojureScript默认对React所做的封装，并且声称其比普通的React速度快的原因。因为ClojureScript必须使用不变的数据结构来存储state，所以在判断是否重新渲染组件时花费很小。然而使用可变的数据来深比较数据是否相等将非常耗费性能。在ClojureScript中，这很简单，因为所有的对象总是不可变的，但是在Javascript中不是如此。

公平的说，即使没有使用纯渲染优化，React也会很快，但是当使用基于JavaScript的动画（例如react-motion），在1s内组件会被成千上万次渲染，或者使用大的组件，例如有上千个单元格的可编辑的表格，在这些情况下，优化将变得至关重要。同样在低配置的移动设备中，你会从中得到巨大的性能提升。

### 反模式

几个月之前，我写了一个可编辑的表格，用来从电子表格中导入用户数据。一张表格很容易就有超过500个用户。在最上层的组件中，我写的代码如下：

```javascript
class Table extends PureComponent {
  render() {
    return (
      <div>
        {this.props.items.map(i =>
          <Cell data={i} options={this.props.options || []} />
         )}
       </div>
     );
  }
}
```

实际上，代码比这更加复杂。`Cell`组件非常复杂，对于每个用户渲染了好多的单元格。所以在应用中有上千个`Cell`元素。

在应用中我载入了500个用户，并且尝试修改一个单元格，修改的动作在我高性能的电脑上竟然花费了几秒时间才完成！后来使用了`console.log()`来调试代码后，我发现当一个很小的单元格改变后，几乎整个应用都会被重新渲染。这怎么可能？我使用的是Redux，冻结了应用的状态，并且使用了不可变的数据。

经过几个小时抓破头皮的思考，我意识到，这其中的一个改变时我使用的数组的默认值：

```javascript
    this.props.options || []
```

可以看到`options`数组传递给`Cell`元素。通常来说，这没有任何问题。其他的`Cell`元素也不会被渲染，因为他们可以做浅比较来检查属性是否一致，并且在一致时跳过渲染，但是万一props是`null`，就会使用默认的数组。正如你所知道的那样，数组字面量和`new Array()`都会创建一个新的数组实例。这会彻底的破坏掉`Cell`元素内纯组件渲染优化。Javascript的不同实例是不相等的，浅比较是否相等总是会返回`false`，并且告诉React来重新渲染组件。修改的方法非常简单：

```javascript
const default = [];
class Table extends PureComponent {
  render() {
    return (
      <div>
        {this.props.items.map(i =>
          <Cell data={i} options={this.props.options || default} />
         )}
       </div>
     );
  }
}
```

现在修改操作只需要几十毫秒！并且`defaultProps`的作用和以前一样。

### 函数也会创建新对象

在render中创建函数也会有同样的问题，好多代码是如下这样写的：

```javascript
class App extends PureComponent {
  render() {
    return <MyInput
      onChange={e => this.props.update(e.target.value)} />;
  }
}
```

或者

```javascript
class App extends PureComponent {
  update(e) {
    this.props.update(e.target.value);
  }
  render() {
    return <MyInput onChange={this.update.bind(this)} />;
 }
}
```

和上面的数组字面量类似，在这两种情况下，都会创建一个新的函数对象。你应该尽早的执行绑定`this`：

```javascript
class App extends PureComponent {
  constructor(props) {
    super(props);
    this.update = this.update.bind(this);
  }
  update(e) {
    this.props.update(e.target.value);
  }
  render() {
    return <MyInput onChange={this.update} />;
  }
}
```

还需要重复一点。也有其他的方法来解决这个问题，使用`React.createClass()`来自动绑定所有的方法或者使用Babel来箭头函数，还有使用自动绑定装饰器。

ESLint rule [react/jsx-no-bind](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md) 是一个用来捕获该问题的工具。

#### 在Reduc`connect(mapState)`中使用Reselect

起初，我并不认为Reselect（一个在Redux官方文档中提到的类库）会如此重要，因为我很少在Redux`connect()`方法中写性能低下的map state函数。我错的是如此离谱。这和函数的性能没有关系，关键是新对象（吃惊吧！），考虑如下的map state函数：

```javascript
let App = ({otherData, resolution}) => (
  <div>
    <DataContainer data={otherData} />
    <ResolutionContainer resolution={resolution} />
  </div>
);
const doubleRes = (size) => ({
  width: size.width*2,
  height: size.height*2
});
App = connect(state => {
  return {
    otherData: state.otherData,
    resolution: doubleRes(state.resolution)
  }
})(App);
```

在这个例子中，state中`otherData`每次发生变化，`DataContainer`和`ResolutionContainer`都会重新渲染，即使state中的`resolution`没有发生变化。这是因为函数`doubleRes`总是会返回一个新的`resolution`对象。如果使用Reselect重写`doubleRes`，问题就会变为如下的情况：

```javascript
import {createSelector} from “reselect”;
const doubleRes = createSelector(
  r => r.width,
  r => r.height,
  (width, height) => ({
    width: width*2,
    height: heiht*2
  })
);
```

Reselect会记住上一次函数的结果，在传入参数没有改变的情况下，将其返回。

### 结论

当你注意到的时候，反模式很明显，但是仍然很容易陷进去。比较好的方面是，如果你搞砸了，就像我之前那样，这不会破坏你的应用，只是会运行的比较慢一点，大多数情况下，并不重要。但是我希望在这篇文章中给你指向了应该去深入研究的某些内容。

翻译自[React.js pure render performance anti-pattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f#.t93mzbo77)