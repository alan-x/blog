[原文地址](https://reactjs.org/docs/react-component.html)
### React.Component

这个页面包含 React 组件类定义的详细 API 索引。它假设你熟悉基本的 React 概念，比如[组件和属性](https://reactjs.org/docs/components-and-props.html)，还有[状态和生命周期](https://reactjs.org/docs/state-and-lifecycle.html)。如果你不，先读他们。


### 概述

React 让你定义组件为类或者函数。组件定义为类当前提供的更多功能，在这个页面详细米哦啊叔。为了定义一个 React 组件类，你需要继承`React.Component`：
```jsx harmony
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

你在一个`React.Component`子类必须定义的方法叫做[render()](https://reactjs.org/docs/react-component.html#render)。所有描述在这个页面的其他方法都是可选的。

**我们再次强烈推荐基于组件类创建你自己的组件**。在 React 组件，[代码重用是主要架构通过组合，而不是继承](https://reactjs.org/docs/composition-vs-inheritance.html)。

> 注意：React 不强制你使用 ES6 类语法。如果你更偏向于避免它，你可能使用`create-react-class`模块或者类似自定义抽象替代。查阅[Using React without ES6](https://reactjs.org/docs/react-without-es6.html)了解更多。

### 组件生命周期

每一个组件有一系列的"生命周期方法"，你可以覆盖去运行代码在处理的特殊时间。**你可以使用[这个生命周期图](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)作为一个小抄**。在下面的列表，常用的生命周期方法被标记为**粗体**。剩下的都是为了不常用使用场景相关存在。

#### 挂载

这些方法以下列的顺序被调用，当一个组件实例被创建并且插入到 DOM：

- **[constructor()](https://reactjs.org/docs/react-component.html#constructor)**
- [static getDerivedStateFromProps](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
- **[render()](https://reactjs.org/docs/react-component.html#render)**
- **[componentDidMount()](https://reactjs.org/docs/react-component.html#componentdidmount)**

**注意：**
这些方法被认为是遗留的，并且你应该在新代码[避免他们](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)：
- [UNSAFE_componentWillMount()](https://reactjs.org/docs/react-component.html#unsafe_componentwillmount)


#### 更新
一个更新可以通过改变 props 或者 state 触发。这些方法以下列的顺序触发，当一个组件正在重新渲染。

- [static getDerivedStateFromProps()](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
- [shouldComponentUpdate()](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)
- **[render()](https://reactjs.org/docs/react-component.html#render)**
- [getSnapshotBeforeUpdate()](https://reactjs.org/docs/react-component.html#getsnapshotbeforeupdate)
- **[componentDidUpdate()](https://reactjs.org/docs/react-component.html#componentdidupdate)**

**注意：**
这些方法被认为是遗留的，并且你应该在新代码[避免他们](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)：
- [UNSAFE_componentWillUpdate()](https://reactjs.org/docs/react-component.html#unsafe_componentwillupdate)
- [UNSAFE_componentWillReceiveProps()](https://reactjs.org/docs/react-component.html#unsafe_componentwillreceiveprops)


#### 挂载
这个方法在一个组件正在从 DOM 上移除的时候调用：
- **[componentWillUnmount](https://reactjs.org/docs/react-component.html#componentwillunmount)**

#### 错误处理
这些方法在渲染期间有一个错误的时候调用，在一个生命周期方法，或者在任何子组件的构造器。
- [static getDerivedStateFromError()](https://reactjs.org/docs/react-component.html#static-getderivedstatefromerror)
- [componentDidCatch](https://reactjs.org/docs/react-component.html#componentdidcatch)

### 其他 API
每一个组件也提供了一些其他 API：
- [setState()](https://reactjs.org/docs/react-component.html#setstate)
- [forceUpdate](https://reactjs.org/docs/react-component.html#forceupdate)

### 类属性
- [defaultProps](https://reactjs.org/docs/react-component.html#defaultprops)
- [displayName](https://reactjs.org/docs/react-component.html#displayname)

### 实例属性
- [props](https://reactjs.org/docs/react-component.html#props)
- [state](https://reactjs.org/docs/react-component.html#state)

### 索引

### 常用生命周期
这个章节方法覆盖你将会创建 React 组件绝大多数使用场景。**对于一个可视化引用，查阅[这个生命周期图](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)**。

#### render()
```jsx harmony
render()
```
`render()`方法是一个类组件中唯一需要的方法。

当调用的时候，它应该检查`this.props`和`this.state`并返回下列的其中一个类型：

- **React elements**。通常通过[JSX](https://reactjs.org/docs/introducing-jsx.html)创建。比如，`<div />`和`<MyComponent />`都是 React 元素，指导 React 去渲染一个 DOM 节点，或者其他用户定义的组件，分别。

- **数组和片段**。让你从 render 返回多个元素。查阅在[fragments](https://reactjs.org/docs/fragments.html)的文档了解更多。

- **Portals**。让你渲染子组件们到一个不同的 DOM 子树。查阅在[portals](https://reactjs.org/docs/portals.html)的文档了解更多。

- **字符串和数字**。这些渲染为 DOM 的文字节点。

- **布尔或者 null**。不渲染任何东西。（主要是为了支持`return test && </Child />`模式，`test`是一个布尔。）

`render`方法应该是纯净的，意味着它不修改组件状态，每次调用的时候它返回相同的结果，并且它不直接和浏览器交互。

如果你需要和浏览器交互，在`componentDidMount`或者其他生命周期方法执行你的工作。保持`render()`纯净，让组件更容易思考。

**注意：**
`render()`不会执行，如果[shouldComponentUpdate()](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)返回 false。

#### constructor()
```jsx harmony
constructor(props)
```
**如果你不初始化状态并且你不绑定方法，你不需要为你的 React 组件实现一个 constructor**。

React 元素的 constructor 在它还没被挂载之前被调用。当为`React.Component`子组件实现 constructor 的时候，你应该在其他任何语句之前调用`super(props)`。 否则，`this.props`在 constructor 中将会是未定义，将会导致 bug。

通常，React 构造器只用于两个目的：
- 通过赋值一个对象到`this.state`初始化[本地状态](https://reactjs.org/docs/state-and-lifecycle.html)。
- 绑定[事件处理](https://reactjs.org/docs/handling-events.html)方法到一个实例。

在`constructor()`中你**不应该调用 setState()**。相反，如果你的组件需要使用本地状态，直接在构造器中为**this.state**赋初始值：
```jsx harmony
constructor(props) {
  super(props);
  // Don't call this.setState() here!
  this.state = { counter: 0 };
  this.handleClick = this.handleClick.bind(this);
}
``` 
构造器是你可以直接赋值`this.state`的唯一地点。在其他方法，你需要使用`this.setState`替代。

避免在构造器引入任何副作用或者定义。为了这些使用场景，使用`componentDidMount`替代。

**注意**
**避免赋值 props 到 state！这是一个创建错误：**
```jsx harmony
constructor(props) {
 super(props);
 // Don't do this!
 this.state = { color: props.color };
}
```
这个问题是他们都是不需要的（你可以使用`this.props.color`直接替代），并且创建 bug（更新`color`属性不会反映在状态）。

**如果你想要忽略 props更新，只有使用这个模式。** 在这个场景，重命名属性为`initialColor`或者`defaultColor`是有意义的。然后你可以强制一个组件去"重置"它的内部状态，通过在需要的时候[改变它的 key](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)。

阅读我们[避免传递状态的博客文章](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)了解更多如果你觉得你需要一些状态依赖于属性要做什么。

#### componentDIdMount()
```jsx harmony
componentDidMount()
```
`componentDidMount()`在组件被挂载（插入到树中）之后立即执行。需要 DOM 节点的初始化可以在这里进行。如果你需要去从远端加载数据，这是实例话网络请求的好地方。

这个方法是设置任何监听的好地方。如果你这么做，不要忘记在`componentWillUnmout`取消订阅。

你在`componentDidMount()`**可能立即调用 setState()**。它将会触发一个额外的渲染，但是它会在浏览器更新屏幕前发生。这保证就算`render()`在这个长江中将调用两次，用户将不会看到中间状态。小心使用这个模式，因为它总是导致性能问题。在大部分场景，你应该能够在`constructor()`中赋值初始状态代替。他可以，然而，对于像模态和工具提示之类的场景是需要的，当你需要去测量一个 DOM 节点，在渲染一些东西依赖它的大小和位置的时候。

#### componentDidUpdate()
```jsx harmony
componentDidUpdate(prevProps, prevState, snapshot)
```
`componentDidUpdate()`在更新发生之后立即调用。这个方法在初始化渲染的时候不会被调用。

使用这个作为一个机会去操作 DOM，当组件被更新的时候。这也是一个执行网络请求的好机会，当你比较当前 props 和之前的 props（比如，网络请求可能不需要，如果属性没有改变）。

```jsx harmony
componentDidUpdate(prevProps) {
  // Typical usage (don't forget to compare props):
  if (this.props.userID !== prevProps.userID) {
    this.fetchData(this.props.userID);
  }
}
```
你在`componentDidUpdate()`**可能立即调用 setState()`**，但是主要以**它必须被一个条件包裹**，就像前面的例子，否则你将会导致一个无限循环。

它将会导致一个额外的重新渲染，尽管不会被用户看见，但是它会影响组件性能。如果你尝试去"镜像"一些状态到一个前面得来的属性，考虑直接使用 props。了解更多关于[为什么复制属性到状态导致 bug](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)。

如果你的组件实现了`getSnapshotBeforeUpdate()`生命周期（很罕见），它返回的值将会作为`componentDidUpdate()`的第三个"snapshot"参数。否则这个参数是未定义。

**注意**
`componentDidUpdate()`将不会调用，如果`shouldComponentUpdate()`返回 false。

#### componentWillUnmount()
```jsx harmony
componentWillUnmount()
```
`componentWillUnmount()`在组件卸载和销毁之前立即调用。在这个方法执行任何需要的清理操作，比如无效化定时器，取消网络请求，或者清理任何我们在`componentDidMount()`的订阅。

你在`componentWillUnmout`中**不应该调用 setState()**，因为组件将不会被重新渲染。一旦一个组件实例被卸载，你将永远无法再次挂载。


### 不常使用的生命周期方法

这个章节的方法标示不常见的使用场景。他们很方便，但是你的大部分组件可能不需要他们。**你可以看见大部分的方法在瞎买呢[这个生命周期图](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)中，如果你点击它头部的"显示不常见生命周期"选择框。


#### shouldComponentUpdate()
```jsx harmony
shouldComponentUpdate(nextProps, nextState)
```
使用`shouldComponentUpdate()`让 React 知道如果一个组件的输出不受当前 state 和 props 的改变。默认行为是在每一次状态改变重新渲染，在大部分场景，你应该依赖默认行为。

`shouldComponentUpdate()`在渲染前调用，当新的 props 或者 state 被接受。默认是`true`。这个方法不会在初始化渲染或者当`forceUpdate()`被使用。

这个方法只作为[性能优化](https://reactjs.org/docs/optimizing-performance.html)存在。不要依赖它"阻止"渲染，因为这会导致 bug。**考虑使用内置的[PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent) 替代手写`shouldComponentUpdate()`。`PureComponent`执行一个 props 和 state 的浅比较，并且减少你将会跳过必须的更新的计划。

如果你很肯定你要去手写，你可能比较`this.props`和`nextProps`，`this.state`和`nextState`并返回`false`去告诉 React 更新可以被跳过。注意返回`false`不会防止子组件重新渲染，当他们的状态改变。

我们不推荐执行深相等检查或者在`shouldComponentUpdate()`使用`JSON.stringify()`。它非常低效，并且将会损害性能。

当前，如果`shouldComponentUpdate()`返回`false`，[UNSAFE_componentWillUpdate()](https://reactjs.org/docs/react-component.html#unsafe_componentwillupdate)，[render()](https://reactjs.org/docs/react-component.html#render)，和[componentDidUpdate()](https://reactjs.org/docs/react-component.html#componentdidupdate)将不会执行。在未来，React 可能对待`shouldComponentUpdate()`为一个技暗示，而不是一个严格指令，并且返回`false`可能依旧会导致组件的重新渲染。

#### static getDerivedStateFromProp()
```jsx harmony
static getDerivedStateFromProps(props, state)
```
`getDerivedStateFromProps`在 render 方法调用之前调用，在初始化挂载和后续更新中都是。它应该返回一个对象去更新状态，或者 null 去不更新。

这个方法是为[不常见使用场景](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#when-to-use-derived-state)存在的，state 依赖于随着时间改变的 props。比如，它可能是实现一个`<Transition>`组件的窍门，比较它前面和后面子组件，去决定他们的哪一个进来或者出去。

分发状态导致冗长的代码并且然你的组件很难去思考。[确保你熟悉简单的替代方案：](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)
- 如果你需要去**执行一个副作用**（比如，数据获取或者一个动画），为了响应 props 的改变，使用[componentDidUpdate](https://reactjs.org/docs/react-component.html#componentdidupdate)生命周期替代。

- 如果你想要去**在 props 改变的时候重新计算一个数据**，[使用一个记忆的 helper 替代](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#what-about-memoization)。

- 如果你想要**在 props 改变的时候"重置"一些状态**，考虑将组件变成[完全控制](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-controlled-component)或者[使用一个 key 完全控制](https://reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)替代。

这个方法无法访问组件实例。如果你想，你可以重用一些代码，在`getDerivedStateFromProps()`和其他类方法，通过组件的 props 和 state 抽取纯函数到类定义之外。

注意这个方法在每一次渲染触发，无视原因。这是在对标`UNSAFE_componentWillReceiveProps`，它缀有在父组件导致一个重新渲染并且不是作为本地`setState`的结果。

#### getSnapshotBeforeUpdate()
```jsx harmony
getSnapshotBeforeUpdate(prevProps, prevState)
```
`getSnapshotBeforeUpdate()`在最近渲染的输出被提交之前调用，比如，DOM。它让你的组件从 DOM（比如，滚动位置） 捕获一些信息，在它潜在改变之前。这个生命周期返回的任何值将被传递到`componentDidUpdate()`作为参数。

这个使用场景不常见，但是它可能出现在像聊天线程的 UI 中，需要以特殊的方式去处理滚动位置。

一个快照值（或者 null） 应该被返回。

比如：
```jsx harmony
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the scroll position so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    // (snapshot here is the value returned from getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```
在前面的例子，在`getSnapshotBeforeUpdate`读取`scrollHeight`属性很重要，因为在"render"阶段生命周期（比如`render`）和"commit"阶段生命周期（像`getSnapshotBeforUpdate`和`componentDidUpdate`）之间可能有延迟。

### 错误边界
[错误边界](https://reactjs.org/docs/error-boundaries.html)是捕获他们子组件树任何 JavaScript 错误的 React 组件，记录这些错误，并且显示一个回退 UI 替代崩溃的组件树。错误边界他们后面的整棵树的渲染，生命周期方法和构造器期间捕捉错误。

一个类组件成为一个错误边界，如果它定义了生命周期方法`static getDerivedStateFromError()`或者`componentDidCatch()`。从这些生命周期更新状态然你捕捉后面的树未处理的 JavaScript 错误并显示一个回退 UI。

只将错误边界用于恢复不期待的异常；**不要尝试使用他们作为控制流**。

了解更多细节，查阅[React 16 中的错误处理](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)。

**注意**
错误处理只捕获在组件后面的树的组件错误。一个错误边界不能在它自身捕获错误。

### static getDerivedStateFromError()
```jsx harmony
static getDerivedStateFromError(error)
```
这个生命周期在一个错误被子组件抛出以后才调用。它接受一个错误，被抛出作为一个参数并且应该返回一个值去更新 state。

```jsx harmony
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```
**注意**
`getDerivedStateFromError()`在"render"阶段被调用，所以富足用是不允许的。对于这些使用场景，使用`componentDidCatch()`替代。

### componentDidCatch()
```jsx harmony
componentDidCatch(error, info)
```
这个生命周期在一个错误被一个后代组件抛出之后执行。它接受两个参数：
1. `error` - 被抛出的错误
2. `info` - 带一个`componentStack` key 的对象，包含[抛出错误的组件的信息](https://reactjs.org/docs/error-boundaries.html#component-stack-traces)。

`componentDidCatch()`在"commit"阶段被调用，所以副作用是被允许的。它应该用于记录日志：
```jsx harmony
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logComponentStackToMyService(info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

**注意**
在一个错误的事件，你可以使用`componentDidCatch()`渲染一个回退 UI，通过调用`setState`，但是这个将在未来版本被废弃。使用`static getDerivedStateFromError()`去处理回退渲染替代。

