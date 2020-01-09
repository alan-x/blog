### Error Boundaries

在过去，组件内的 JavaScript 错误干扰 React 的内部状态，导致它在在下一次渲染产生神秘的错误。这些错误总是由应用程序代码中更早的错误导致，但是 React 没有提供在组件内优雅的处理他们的方式，并且无法恢复。

### 介绍错误边界
一个在局部 UI 的 JavaScript 错误不应该破坏整个 app。为了为 React 用户解决这个问题，React 16 引入了一个新的概念，叫做"错误边界"。

错误边界是 React 组件，捕获在他们子组件树的任何 JavaScript 错误，记录这些错误，并显示一个回退 UI，替换崩溃的组件树。错误边界在渲染期间捕获错误，在生命周期方法，和之后的整个树的构造器中。

注意：
错误边界不为以下捕获错误：
- 事件处理器（[了解更多]()）
- 异步代码（比如，`setTimeout`或者`requestAnimationFrame`回调）
- 服务端渲染
- 错误边界自己跑出来的错误（而不是他们的子孙）。

一个类组件成为一个错误边界，如果它定义了生命周期方法[static getDerivedStateFromError]()或者[componentDidCache]()。使用`static getDerivedStateFromError()`去渲染一个回退 UI，在一个错误被抛出。使用`componentDidCache()`去记录错误信息。
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

  componentDidCatch(error, errorInfo) {
    // You can also log the error to an error reporting service
    logErrorToMyService(error, errorInfo);
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
然后你可以像一个常规组件一样使用它：
```jsx harmony
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```
错误边界像 JavaScript 的`catch {}`块一样工作，但是是用于组件。只有类组件可以成为错误边界。在实践中，大部分的事件，你将声明一个错误的边界组件，并贯穿你的应用使用。

注意，**错误边界只捕获在他们下面的树的错误**。一个错误边界尝试渲染错误信息失败，错误将会扩散到离它最近的前面的错误边界。这也 JavaScript 和 catch {} 块的很像。

### 演示例子
查阅使用[React 16]()[声明和使用错误边界的例子]()。

### 错误边界放哪里

错误边界的力度取决于你。你可能包裹顶级路由组件去显示一个"一些东西出现错误"消息给用户，就像服务端框架通常处理奔溃。你也可能包裹独立的组件在一个错误边界去保护他们不让应用的其他地方奔溃。

### 未捕获错误的新行为
这个改变有一个重要的含义。**在 React 16 之前，没有被任何错误边界捕获的错误将会导致整个 React 组件树的卸载**。
我们讨论了这个决定，但是我们的经验告诉我们，保留一个损坏的 UI 比整个移除它更加糟糕。比如，Messenger 类似的产品保留损坏的 UI 可见将会导致一些人发送消息给错误的人。同样，对于一个 payment app 去显示一个错误的总数比什么都不渲染更糟糕。

这个该改变意味着，当你升级到 React 16，你将可能发现你的应用存在之前没有注意到的奔溃。添加错误边界让你提供更好的用户体验，当一些东西发生错误的时候。

比如，Facebook Messenger 包裹侧边栏的内容，信息板，绘画日志，和信息输入框到分离的错误边界。如果这些 UI 区域的一些组件奔溃，他们剩下的依旧可以交互。


我们鼓励你去使用 JS 错误报告服务（或者构建你自己的），这样你可以了解到产品中发生的未处理的异常，并且修复他们。

### 组件栈跟踪
React 16 在开发模式打印所有出现在渲染期间的错误到控制台，就算应用不小心吃了他们。此外，错误信息和 JavaScript 栈，也提供了组件栈跟踪。现在你可以精确的看看错误发生在组件树的哪里。

![https://reactjs.org/static/error-boundaries-stack-trace-f1276837b03821b43358d44c14072945-71000.png]()

你也可以在组件栈跟踪看见文件名和行号。这个默认在[Create React App]()项目可用。

如果你不使用 Create React App，你可以手动添加[这个插件]()到你的 Babel 配置。注意它只用于开发，并且必须在**生产环境禁止**。

注意：
跟踪栈中的组件名依赖于[Function.name]()属性。如果你支持旧的浏览器并且它没有原生提供这个功能（比如 IE 11）。考虑包含一个`Function.name` polyfill 在你的打包的应用，比如[function.name-polyfill]()。此外，你可能明确的设定[displayName]()属性在你的所有组件。


### try/cache 怎么样？
`try`/`catch`很好，但是只为命令式代码工作：
```jsx harmony
try {
  showButton();
} catch (error) {
  // ...
}
```
然而，React 组件是声明的，并指定什么应该被渲染：
```jsx harmony
<Button />
```

错误边界保留了 React 的自然声明，并且行为就像你期待的。比如，就算一个错误发生在`componentDidUpdate`方法，是由于树中更深的`setState`导致的，它依旧正确的扩散到最近的错误边界。

### 事件处理器怎么样？
错误边界**不**捕获事件处理器内部的错误。  

React 不需要错误边界去从事件处理器的错误中恢复。不像 render 方法和生命周期方法，事件处理器不再渲染的时候发生。所以如果他们抛出，React 依旧知道显示什么到屏幕上。

如果你需要捕获事件处理器内的错误，使用常规的`try`/`catch`语句。
```jsx harmony
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    try {
      // Do something that could throw
    } catch (error) {
      this.setState({ error });
    }
  }

  render() {
    if (this.state.error) {
      return <h1>Caught an error.</h1>
    }
    return <div onClick={this.handleClick}>Click Me</div>
  }
}
```

注意前面的例子表示常规的 JavaScript 行为不使用错误边界。

从 React 15 以后的命名变化

React 15 包含一个非常优先支持的错误边界，在一个不同的方法名下面：`unstable_handleError`。这个方法不再工作，你需要改为`componentDidCatch`在你的代码从第一个 16 beta 发行开始。

对于这个例子，我们提供了一个[codemod]()去自动升级你的代码。












