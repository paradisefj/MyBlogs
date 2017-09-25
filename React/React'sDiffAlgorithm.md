文章来源：[React’s diff algorithm](http://calendar.perfplanet.com/2013/diff/)

#React的差异算法

[React](http://facebook.github.io/react/)是Facebook推出的用来开发构建用户界面的JavaScript库。在这篇文章中我将介绍差异算法和React是如何渲染的，这样你可能最大的优化你的应用。

##Diff算法

在进入到算法实现细节之前，我们先来回顾一下React是如何工作的。

```javascript
var MyComponent = React.createClass({
	render: function(){
		if(this.props.first){
			return <div className="first"><span>A Span</span></div>;
		}else {
			return <div className="second"><p>A Paragraph</p></div>;
		}
	}
});
```

在任何时间点，你描述了你的UI的样子。理解渲染的结果不是一个真正的DOM节点很重要。他们仅仅是轻量的JavaScript对象。我们称之为虚拟DOM。

React会使用该表示方法（注：上文中的轻量的JavaScript对象）来寻找从先前的渲染到下一次渲染的最小的步骤。例如，如果我们先加载`<MyComponent first={true} />`，然后用`<MyComponent first={false} />`替换它，最后将其移除，下面是该过程的DOM指令：

> 从无到第一次
- 创建节点：`<div className="first"><span>A Span</span></div`

> 从第一次到第二次
- 替换属性：用`className="second"` 来替换 `className="first"`
- 替换节点：用`<p>A Paragraph</p>` 来替换 `<span>A Span</span>`

> 从第二次到移除
- 移除节点：`<div className="second"><p>A Paragraph</p></div>`

###一级一级

在任意两个树中找到修改的最小改变是一个O(n^3)时间复杂度的算法。正如你所想的，这不适用于我们的使用情况。React使用简单的但是非常强大的搜索算法，能够在接近O(n)的时间复杂度内找到差异。

React仅仅是一层一层的访问树结构。这戏剧化的降低了复杂度，并且没有大的损失，因为在web应用中，几乎没有将一个组件移动到树中不同的层级上的情况。他们通常在孩子之间移动。

![图1](http://img.blog.csdn.net/20160528210806399)

###列表

让我们先来举个例子，如果我们有一个组件，在一次遍历中渲染了五个组件，并且在下一次中在列表的中间插入了一个新的组件。通过这些信息来知道这两个列表的逐渐是如何映射的会非常困难。

默认情况下，React将前一个列表的第一个组件和下一个列表的第一个组件进行关联，以此类推。你可以提供一个`key`属性来帮助React发现映射。在实际中，在孩子中很容易会发现一个唯一的`key`.

![图2](http://img.blog.csdn.net/20160528210941350)

###组件

通常情况下，一个React应用是由许多用户自定义的组件组成的，最终变成一个由`div`组成的树。React会考虑这种额外的信息，并且只匹配有相同的class的组件。

例如，如果一个`<Header>`被一个`<ExampleBock>`替换了，React会移除header，并且创建一个新的example block。我们不会花费时间来试图寻找这两个组件不相同的部分。

![图3](http://img.blog.csdn.net/20160528211026038)

##事件委派

在DOM节点中添加事件监控是非常痛苦的慢，并且耗费内存。然而，React实现了一个名为“事件委派”的流行的技术。React更近一步，并且实现了W3C的事件系统。这意味着IE8的事件处理bug已经是过去式了，并且所有的事件名称在不同的浏览器中是一致的。

让我来解释一下这是如何实现的。一个单事件监听被添加在document的根部。当一个事件被触发时，浏览器会给我们目标DOM节点。为了能够沿着DOM层级结构传播事件，React不会遍历虚拟DOM的层级结构。

作为代替，我们使用了一个事实：每个React组件有一个唯一的id来编码层级结构。我们可以使用一个简单的字符串来得到所有父节点的id。通过在一个hash map中存储所有的事件监听，我们发现这个方法的性能比将其添加在虚拟DOM上要好的多。下面这个例子展示了当一个事件通过虚拟DOM传播是发生了什么。

```javascript
// dispatchEvent('click', 'a.b.c', event) 
clickCaptureListeners['a'](event); 
clickCaptureListeners['a.b'](event); 
clickCaptureListeners['a.b.c'](event); 
clickBubbleListeners['a.b.c'](event); 
clickBubbleListeners['a.b'](event); 
clickBubbleListeners['a'](event);
```

浏览器为每个事件和事件监听创建了一个新的对象。这些对象有很好的属性，你可以保存其引用，甚至修改它。然后，这意味着有很高的内存花费。React启动时为这些对象分配了一个内存池。当需要一个时间对象时，可以在内存池中取到。这显著的减少了内存回收。

##渲染

###合并渲染

当你在一个组件上调用`setState`时，React将其标记为脏的。当循环结束时，React查询所有脏的组件，并且重新渲染他们。

该合并意味着在事件循环中，只有一个确切的DOM被更新的时间点。这是构建一个高性能的应用的关键，并且在其他JavaScript代码中很难实现。在React应用中，你自然而然的使用它。

![图4](http://img.blog.csdn.net/20160528211056070)

###子树重新渲染

当`setState`被调用，组件会为其孩子重新构建虚拟DOM。如果你在根元素上调用`setState`，整个React应用会被重新渲染。所有的组件，甚至你从来没改变的组件，都会调用其`render`方法。这听起来很恐怖和低效，但是实际中，这工作起来很好，因为我们没有触摸实际的DOM。

首先，我们讨论一下展示用户界面。因为屏幕的空间是有限的，很多情况下你需要一次展示成百上千个元素。JavaScript对于整个业务逻辑的接口管理来说已经足够的快了。

另一个重要的点事当编写React代码时，每次事情发生变化时，你通常不会再根节点上调用`setState`。你在接收到事件变化的节点或者上面一些节点上调用它。很少情况下你会到达根节点。这意味着更新只在用户交互的局部发生。

![图5](http://img.blog.csdn.net/20160528211125778)

###选择子树重新渲染

最后，你又可能阻止一些子树的重新渲染。如果你实现了组件中的下面的方法：

```javascript
boolean shouldComponentUpdate(object nextProps, object nextState)
```

基于组件的前一个和下一个props/state，你可以告诉React这个组件没有改变并且不需要重新渲染。当正确实现时，会给你的应用带来巨大的性能的提升。

为了能够使用它，你不得不比较JavaScript对象。有许多问题会浮现出来，例如应该是浅比较还是深比较；如果是深比较，我们应该使用不可改变的数据结构还是深度拷贝。

并且你需要记住，即使重新渲染不是必须的，该函数每次都会被调用，所以你应该确保其花费的计算时间比React的搜索和重新渲染该组件的时间少。

![图6](http://img.blog.csdn.net/20160528211232545)

##总结

让React更快的技术不是新技术。很长时间以来我们已经知道了触摸DOM的花费是昂贵的，你应该打包你的读写操作，时间委派更快...

人们仍然在讨论他，因为在实际中，使用平常的JavaScript代码实现地很困难。让React能够脱颖而出的是所有的优化都是默认发生的。

React性能消耗的模型理解起来很简单：每次`setState`会重新渲染整个子树。如果你想要提升性能，调用`setState`的频率变低，并且使用`shouldComponentUpdate`来阻止一个大的子树的重新渲染。