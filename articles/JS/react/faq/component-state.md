[原文地址](https://reactjs.org/docs/faq-state.html)
### Component State

### setState 做了什么？

`setState()`调度一个更新到组件的`state`对象。当状态改变，组件通过重新渲染响应。

### state 和 props 有什么不同？

[props](https://reactjs.org/docs/components-and-props.html)（"properties"的简称）和[state](https://reactjs.org/docs/state-and-lifecycle.html)都是简单 JavaScript 对象。尽管都持有影响渲染输出的信息，他们有一个重要的不同：`props`传递到组件（类似函数参数），`state`在组件内管理（和函数内的变量声明类似）。

有些好的资源可以用于在使用`props`和`state`的时候深入阅读：

- [Props vs State](https://github.com/uberVU/react-guide/blob/master/props-vs-state.md)

- [ReactJS: Props vs. State](https://lucybain.com/blog/2016/react-state-vs-pros/)

### 为什么 setState 给我错误的值？

在 React，`this.props`和`this.state`表示渲染的值，比如，当前屏幕的值。

调用`setState`是异步的 - 不要依赖`this.state`去在调用`setState`之后立即反映新的值。传递一个更新函数，而不是一个对象，如果你需要去比较值，基于当前状态（查阅下面了解详情）。

例子中的代码将不会如预期表现：
```jsx harmony
incrementCount() {
  // Note: this will *not* work as intended.
  this.setState({count: this.state.count + 1});
}

handleSomething() {
  // Let's say `this.state.count` starts at 0.
  this.incrementCount();
  this.incrementCount();
  this.incrementCount();
  // When React re-renders the component, `this.state.count` will be 1, but you expected 3.

  // This is because `incrementCount()` function above reads from `this.state.count`,
  // but React doesn't update `this.state.count` until the component is re-rendered.
  // So `incrementCount()` ends up reading `this.state.count` as 0 every time, and sets it to 1.

  // The fix is described below!
}
```
查阅下面了解怎样修复这个问题。

### 如何依赖当前状态值更新状态？

传递一个函数替代一个对象到`setState`去确保调用总是使用最新版本的状态（查阅下面）。

### 传递一个对象或者一个函数到`setState`有什么不同？

传递一个更新函数允许你在更新器内部去访问当前状态值。因为`setState`调用是批量的，这让你链式更新并且确保建立在彼此之上，而不是互相冲突。
```jsx harmony
incrementCount() {
  this.setState((state) => {
    // Important: read `state` instead of `this.state` when updating.
    return {count: state.count + 1}
  });
}

handleSomething() {
  // Let's say `this.state.count` starts at 0.
  this.incrementCount();
  this.incrementCount();
  this.incrementCount();

  // If you read `this.state.count` now, it would still be 0.
  // But when React re-renders the component, it will be 3.
}
```
[了解更多关于 setState](https://reactjs.org/docs/react-component.html#setstate)

### setState 什么时候是异步的？

当前，`setState`在事件处理器内部是异步的。

这确保，比如，如果`Parent`和`Child`在一个点击事件内调用`setState`，`Child`不会调用两次重新渲染。相反，React 在浏览器事件结束的时候"刷新"状态更新。这在大型应用导致性能提升。

这是一个实现详情，避免直接依赖它。在未来的版本，React 将在更多场景批量更新。

### 为什么 React 异步更新 this.state？

正如前一个章节解释的，React 有意"等待"直到所有组件在他们的事件处理器调用`setState()`，在开始重新渲染之前。这通过避免不需要的从新渲染促进性能。

然而，你可能依旧思考为什么 React 不只是立即更新`this.state`，而不重新渲染。

有两个主要原因：

- 这将破坏`props`和`state`之间的一致性，导致问题非常难调试。

- 这将导致我们正在实现的一些新功能无法实现。

这个 [GitHub comment](https://github.com/facebook/react/issues/11527#issuecomment-360199710) 深入指定例子。

### 我应该使用一个类似 Redux 或者 MobX 之类的状态管理库？

[可能](https://redux.js.org/faq/general#when-should-i-use-redux)

在添加额外的库之前，最好先知道 React。只使用 React，你可以构建更复杂的应用。




























