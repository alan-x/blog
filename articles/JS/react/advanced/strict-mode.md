[原文地址](https://reactjs.org/docs/strict-mode.html)
### 严格模式

`StrictMode`是一个用于高亮一个应用潜在问题的工具。像`Fragement`，`StrictMode`不渲染任何可见 UI。它启用额外的检测并为它的子孙警告。

注意：严格模式检测只在开发模式运行；他们不影响产品构建。

你可以在应用的任何部分启用严格模式：比如：
```jsx harmony
import React from 'react';

function ExampleApplication() {
  return (
    <div>
      <Header />
      <React.StrictMode>
        <div>
          <ComponentOne />
          <ComponentTwo />
        </div>
      </React.StrictMode>
      <Footer />
    </div>
  );
}
```
在前面的例子中，严格模式检测不会影响`Header`和`Footer`组件。然而，`ComponentOne`和`ComponentTwo`，还有它们的子组件，将会被检查。

`StrictMode`现在对以下有帮助：

- [标识有不安全生命周期的组件]()

- [警告遗留的字符串 ref API 使用]()

- [警告关于废弃的 findDOMNode 使用]()

- [检测意外的副作用]()

- [检测遗留的 context  API]()

其他的功能将会 React 的未来发行中添加。

### 标识不安全的生命周期

正如[在这篇博文](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)解释的，某些遗留的生命周期方法在异步 React 生命周期是不安全的。然而，如果你的应用使用第三方库，很难保证这些生命周期不被使用。幸运的是，严格模式对此有帮助。

当启用严格模式，React 编译使用不安全生命周期的类组件列表，并使用这些组件的信息记录一个警告信息，比如：
![](https://reactjs.org/static/strict-mode-unsafe-lifecycles-warning-e4fdbff774b356881123e69ad88eda88-2535d.png)

通过严格模式指出这些问题让你在 React 的未来发行中充分利用 concurrent 渲染。

### 警告遗留的字符串 ref API 使用

之前，React 提供两种方式的 ref：遗留的字符串 ref API 和回调 API。尽管字符串 ref API 更方便，它有[一些缺点](https://github.com/facebook/react/issues/1373)，因此我们的官方推荐是[使用回调替代](https://reactjs.org/docs/refs-and-the-dom.html#legacy-api-string-refs)。

React 16.3 添加了一个第三方选项，提供便捷的字符串 ref，没有任何缺点：
```jsx harmony
class MyComponent extends React.Component {
  constructor(props) {
    super(props);

    this.inputRef = React.createRef();
  }

  render() {
    return <input type="text" ref={this.inputRef} />;
  }

  componentDidMount() {
    this.inputRef.current.focus();
  }
}
```

因为对象 ref 是为了替代字符串 ref 遗留添加的，严格模式现在警告字符串 ref 的使用。

注意：除了新的`createRef` API，回调 ref 将继续被支持。

你不需要在你的组件中替换回调 ref。他们稍微灵活些，因此他们将会保留作为高级特性。

[了解更多关于 createRef API](https://reactjs.org/docs/refs-and-the-dom.html)

### 警告废弃的 findDOMNode 使用

React 过去支持`findDOMNode`为一个类实例在树中搜索 DOM 节点。通常你不需要这个，因为你可以[直接绑定 ref 到一个 DOM 节点](https://reactjs.org/docs/refs-and-the-dom.html#creating-refs)。

`findDOMNode`可以用于类组件，但是这将会打破抽象级别，它需要父组件要求某些子元素被渲染。它产生了重构风险，你不能改变组件的详情，因为一个父组件可能搜索它的 DOM 节点。`findDOMNode`只返回第一个子元素，但是使用 Fragment，一个组件渲染多个 DOM 节点是可能的。`findDOMNode`是单次读取的 API。它只会在你查询的时候响应。如果子组件渲染了一个不同的节点，没有办法处理这个改变。因此，`findDOMNode`只又在组件总是返回但一个的 DOM 节点并且永远不会改变的时候才生效。

你可以通过明确传递一个 ref 到你的自定义组件，并使用[ref 转发](https://reactjs.org/docs/forwarding-refs.html#forwarding-refs-to-dom-components)传递到这个 DOM。

你也可以添加个包裹 DOM 节点到你的组件并直接绑定一个 ref。
```jsx harmony
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.wrapper = React.createRef();
  }
  render() {
    return <div ref={this.wrapper}>{this.props.children}</div>;
  }
}
```

注意：在 CSS，[display: contents]()属性可以被使用，如果你不想要让节点成为布局的一部分。

### 检测不期待的副作用

概念上，React 工作分为两个阶段：

- **渲染**阶段确定哪些该改变需要被创建，比如，DOM。在这个极端，React 调用`render`，然后和前一次渲染比较。

- **提交**阶段是 React 应用改变的时候。（在 React DOM 的场景中，这是 React 插入，更新，和移除 DOM 节点的时候。）React 也在这个阶段调用像`componentDidMount`和`componentDidUpdate`之类的生命周期。

提交阶段通常很快，但是渲染节能很慢。因为这个原因，将来的 concurrent 模式（默认不启用）将渲染工作分割到块，暂停和恢复工作，以避免堵塞浏览器。这意味着 React 可能在提交之前调用渲染阶段生命周期多次，或者执行他们而完全不提交（因为一个错误或者一个更高优先级的中断）。

渲染阶段生命周期包含下面的类方法：

- `constructor`

- `componentWillUnmount`（或`UNSAFE_componentWillMount`）

- `componentWillReceiveProps`（或者`UNSAFE_componentWillReceiveProps`）
- `componentWillUpdate`（或者`UNSAFE_componentWillUpdate`）
- `getDerivedStateFromProps`
- `shouldComponentUpdate`
- `render`
- `setState`更新器函数（第一个参数）

因为前面的方法可能被调用多次，不包含副作用是很重要的。忽略这个规则会导致多种问题，包含内存泄漏和非法的应用状态。不幸的是，很难检测这些问题，因为他们通常是[不确定的](https://en.wikipedia.org/wiki/Deterministic_algorithm)。

严格模式无法为你自动检测副作用，但是它可以帮你监控他们，通过让他们更确定一点。这是通过有意的调用下面方法两次：

- 类组件`constructor`方法

- `render`方法

- `setState`更新器函数（第一个参数）

- 静态`getDerivedStateFromProps`生命周期

注意：这只在开发模式应用。生命周期不会在生产模式调用两次。

比如，考虑以下代码：

```jsx harmony
class TopLevelRoute extends React.Component {
  constructor(props) {
    super(props);

    SharedApplicationState.recordEvent('ExampleComponent');
  }
}
```

第一眼，这代码看过去好像没有问题。但是如果`SharedApplicationState.recordEvent`是[非幂等性](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning)的，则实例化这个组件多次会导致非法的应用状态。这类不易察觉的 bug 在开发中可能不会被注意，或者它可能太不一致因此被忽略。

通过特意调用两次像组件构造器，严格模式让类似这样的模式更容易监控。

### 解绑遗留的 context  API

遗留的 context API 是容易出错的，将会在未来的大版本中移除。它依旧在所有 16.x 发行中可用，但是将会在严格模式中显示这个警告信息：
![](https://reactjs.org/static/warn-legacy-context-in-strict-mode-fca5c5e1fb2ef2e2d59afb100b432c12-4b612.png)

阅读[新 Context API 文档](https://reactjs.org/docs/context.html)帮助升级到新的版本。
