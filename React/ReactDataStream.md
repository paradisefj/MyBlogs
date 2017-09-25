# React 数据流

在 React 中，数据是自顶向下单向流动的，即从父组件到子组件。这条原则让组件之间的关系变得简单且可预测。
state 与 props 是 React 组件中最重要的概念。**如果顶层组件初始化 props，那么 React 会向下遍历整棵组件树，重新尝试渲染所有相关的子组件。** 而 state 只关心每个组件自己内部的状态， 这些状态只能在组件内改变。把组件看成一个函数，那么它接受了 props 作为参数，内部由 state 作为函数的内部参数，返回一个 Virtual DOM 的实现。

## state

当组件内部使用库内置的 setState 方法时，最大的表现行为就是该组件会尝试重新渲染。setState 是一个异步方法，一个生命周期内所有的 setState 方法会合并操作。

1. smartComponent 智能组件
: 通过state更新组件
2. dumb Component 木偶组件
: 通过回调函数，修改组件的props更新组件

## props

props 的传递过程，对于 React 组件来说是非常直观的。React 的单向数据流，主要的流动管 道就是 props。props 本身是不可变的。当我们试图改变 props 的原始值时，React 会报出类型错 误的警告，组件的props一定来自于默认属性或通过父组件传递而来。


