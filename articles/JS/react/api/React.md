[原文地址](https://reactjs.org/docs/react-api.html)
### React 顶级 API

`React`是 React 库的进入点。如果你从`<script>`标签加载 React，这些顶级 API 在`React`全局可用。如果你使用`npm`和 ES6，你可以写`import React from 'react'`。如果你使用 ES5 和 npm，你可以写`var React = require('react')`。

### 概述

### 组件

React 让你分离 UI 到独立的，可重用的块，并在每一块独立思考。React 组件可以通过继承`React.Component`或者`React.PureComponent`定义：

- [React.Component](https://reactjs.org/docs/react-api.html#reactcomponent)

- [React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)

如果你不使用 ES6 类，你可以使用`create-react-class`模块代替。查阅[使用 React 不使用 ES6](https://reactjs.org/docs/react-without-es6.html)了解更多详情。

React 组件也能定义为函数，可以被包裹到：

- [React.memo](https://reactjs.org/docs/react-api.html#reactmemo)


### 创建 React 元素

我们推荐[使用 JSX](https://reactjs.org/docs/introducing-jsx.html) 去描述你的 UI 看起来的样子。每一个 JSX 元素都只是调用[React.createElement()](https://reactjs.org/docs/react-api.html#createelement)的语法糖。如果你使用 JSX，通常不会直接调用下面的方法：

- [createElement](https://reactjs.org/docs/react-api.html#createelement)

- [createFactory](https://reactjs.org/docs/react-api.html#createfactory)

查阅[使用 React 不使用 JSX](https://reactjs.org/docs/react-without-jsx.html)了解更多信息。

### 转化元素

`React`提供多个操作元素的 API：

- [cloneElement](https://reactjs.org/docs/react-api.html#cloneelement)

- [isValidElement](https://reactjs.org/docs/react-api.html#isvalidelement)

- [React.Children](https://reactjs.org/docs/react-api.html#reactchildren)


### Fragment

`React`也提供了一个组件用来渲染多个元素而不需要容器。

- [React.Fragment](https://reactjs.org/docs/react-api.html#reactfragment)

### Ref

- [React.createRef()](https://reactjs.org/docs/react-api.html#reactcreateref)

- [React.forwardRef()](https://reactjs.org/docs/react-api.html#reactforwardref)

### Suspense

Suspense 让组件在渲染之前"等待"一些东西。现在，Suspense 只支持一个场景：[使用 React.lazy 动态加载组件](https://reactjs.org/docs/code-splitting.html#reactlazy)。在未来，他将支持类似数据获取之类的场景

- [React.lazy](https://reactjs.org/docs/react-api.html#reactlazy)

- [React.Suspense](https://reactjs.org/docs/react-api.html#reactsuspense)

### Hook

Hook 是 React 16.8 新加入的功能。它让你使用状态和 React 特性而不需要编写一个类。Hook 有一个[分离的文档章节]()和一个分离的 API 索引：

- [Basic Hooks]()
    - [useState]()
    - [useEffect]()
    - [useContext]()
- [Additional Hooks]()
    - [useReducer]()
    - [useCallback]()
    - [useMemo]()
    - [useRef]()
    - [useImperativeHandle]()
    - [useLayoutEffect]()
    - [useDebugValue]()


### 索引

### React.Component
当使用 ES6 类定义的时候，`React.Component`是 React 组件的基本类：
```jshelllanguage
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

查阅[React Component API 索引]()了解基本`React.Component`的方法和属性。

### React.PureComponent
`React.PureComponent`和[React.Component](https://reactjs.org/docs/react-api.html#reactcomponent)类似。他们之间的不同是[React.Component](https://reactjs.org/docs/react-api.html#reactcomponent)不实现[shouldComponentUpdate()](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)，但是`React.PureComponent`使用一个浅属性和状态比较实现它。

如果你的 React 组件`render()`函数渲染使用相同的属性和状态渲染相同的结果，你可以在某些场景使用`React.PureComponent`做性能提升。

注意：`React.PureComponent`的`shouldComponentUpdate()`只浅比较对象。如果他们包含复杂的数据结构，他可能对深层产生伪错误。只有在你期待只有简单属性和状态的时候继承`PureComponent`，或者在你知道深层数据结构有变化的时候使用[forceUpdate()](https://reactjs.org/docs/react-component.html#forceupdate)。或者，假设使用[immutable 对象](https://facebook.github.io/immutable-js/)去促进嵌套数据的快速比较。

此外，`React.PureComponent`的`shouldComponentUpdate()`为整个组件子树跳过属性更新。确保所有的子组件也都是"纯净的"。

### React.memo
```jshelllanguage
const MyComponent = React.memo(function MyComponent(props) {
  /* render using props */
});
```
`React.memo`是一个[高阶组件](https://reactjs.org/docs/higher-order-components.html)。和[React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)类似，但是它是针对函数组件的，而不是类组件。

如果你的函数组件相同的属性渲染相同的结果，在某些场景，你可以将它包裹到`React.memo`调用，通过记住结果作为性能提升。这意味着 React 将跳过组件渲染，并重用上一次渲染的结果。

`React.memo`只受属性变化影响。如果包裹在`React.memo`的函数组件实现有一个[useState](https://reactjs.org/docs/hooks-state.html)或者[useContext](https://reactjs.org/docs/hooks-reference.html#usecontext) Hook，它将在状态和 context 改变的时候重新渲染。

默认，它只浅比较属性对象中的复杂对象。如果你想要控制比较，你可以提供一个自定义比较函数作为第二个参数。

```jshelllanguage
function MyComponent(props) {
  /* render using props */
}
function areEqual(prevProps, nextProps) {
  /*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */
}
export default React.memo(MyComponent, areEqual);
```
这个方法只作为[性能优化]()存在。不要依赖他去"防止"一个渲染，这将导致 bug。

注意：不像类中的[shouldComponentUpdate()]组件方法，`areEqual`函数返回`true`，如果属性相等，返回`false`如果属性不相等。这和`shouldComponentUpdate()`相反。

### createElement()

```jshelllanguage
React.createElement(
  type,
  [props],
  [...children]
)
```
创建和返回一个新的指定类型的[React 元素](https://reactjs.org/docs/react-api.html#reactfragment)。type 参数可以是一个标签名称字符串（比如'div'或者'span'），一个[React 组件](https://reactjs.org/docs/components-and-props.html)类型（一个类或者一个函数），或者一个[React fragment](https://reactjs.org/docs/react-api.html#reactfragment)类型。

使用 JSX 编写的代码将会被覆盖为`React.createElement()`。如果你使用 [JSX](https://reactjs.org/docs/introducing-jsx.html)，你通常不会直接调用`React.createElement()`。查阅[React 不使用 JSX](https://reactjs.org/docs/react-without-jsx.html)了解更多。

### cloneElement()

```jshelllanguage
React.cloneElement(
  element,
  [props],
  [...children]
)
```
克隆和返回一个新的 React 元素，它以`element`作为起点。结果元素将原始元素的属性和 props 的浅合并作为新属性。新的子元素将会替换存在的子元素。原始元素的`key`和`ref`将会被保留。

`React.cloneElement()`基本和下面相等：
```jshelllanguage
<element.type {...element.props} {...props}>{children}</element.type>
```
然而，它也保留`ref`。这意味着如果你使用`ref`获取一个子元素，你不会意外从它的祖先偷走。你将得到绑定在你新元素上相同的`ref`。

这个 API 的引入是为了替换废弃的`React.addons.clineWithProps()`。

### createFactory()
```jshelllanguage
React.createFactory(type)
```
返回一个产生指定类型 React 元素的函数。就像[React.createElement()](https://reactjs.org/docs/react-api.html#createelement)，类型参数可以是一个标签名称字符串（比如`'div'`或者`'span'`），一个[React 组件](https://reactjs.org/docs/components-and-props.html)类型（一个类或者一个函数）,或者一个[React fragment](https://reactjs.org/docs/react-api.html#reactfragment)类型。

这个助手可以认为是遗留的，我们鼓励你直接使用 JSX 或者`React.createElement()`。

如果你使用 JSX，通常你不会直接调用`React.createFactory()`。查阅[React 不使用 JSX](https://reactjs.org/docs/react-without-jsx.html)了解更多。

### isValidElement()
```jsx harmony
React.isValidElement(object)
```
验证对象是一个 React 元素。返回`true`或者`false`。

### React.Children

`React.Children`提供处理`this.props.children`不透明数据结构的工具。

### React.Children.map
```jsx harmony
React.Children.map(children, function[(thisArg)])
```
在包含在`children`的每个直接子元素上调用一个函数，该函数的`this`设置为`this.Arg`。如果`children`是一个数组，它将会遍历，并且在数组的每一个子元素上调用函数。如果 children 是`null`或者`undefeined`，这个方法将会返回`null`或者`undefined`，而不是一个数组。

注意：如果`children`是一个`Fragment`，他将会被认为是一个单一的子元素，而不会遍历。

### React.Children.forEach
```jsx harmony
React.Children.forEach(children, function[(thisArg)])
```
就像[React.Children.map()]()，但是不反悔一个数组。

### React.Children.count
```jsx harmony
React.Children.count(children)
```
返回`children`中的组件总数，等于传输给`map`或者`forEach`的函数的调用次数。

### React.Children.only
```jsx harmony
React.Children.only(children)
```
验证`children`只有一个子元素（一个 React 元素），并返回它。否则，这个方法抛出一个错误。

注意：`React.Children.only`不接受`React.Children.map()`的返回值，因为它是一个数组而不是一个 React 元素。

### React.Children.toArray
```jsx harmony
React.Children.toArray(children)
```
返回`children`不透明的数据结构为一个扁平化的数组，并为每一个子元素提供一个 key。如果你要在你的 render 方法中操作子元素集合就很有用，特别是你想要在向下传递之前重新排序或者分割`this.props.children`。

注意：`React.Children.toArray()`改变 key，为了在扁平化的时候保留嵌套数组的语义。也就是，`toArray`在返回的数组中为每一个 key 添加前缀，这样每一个元素的 key 都限定在包含它的数组中。

### React.Fragment

`React.Fragment`组件让你在`render()`方法中返回多个子元素，不需要创建一个额外的 DOM 元素：
```jsx harmony
render() {
  return (
    <React.Fragment>
      Some text.
      <h2>A heading</h2>
    </React.Fragment>
  );
}
```

你也可以使用它的缩写模式`<></>`语法。了解更多信息，查阅[React V16.2：改善对 Fragment 的支持](https://reactjs.org/blog/2017/11/28/react-v16.2.0-fragment-support.html)。

### React.createRef

`React.createRef`创建一个[ref](https://reactjs.org/docs/refs-and-the-dom.html)，可以通过 ref 属性和 React 元素绑定。
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

### React.forwardRef
`React.forwardRef`创建一个 React 组件，转发它接收到的[ref](https://reactjs.org/docs/refs-and-the-dom.html)属性到它下面的其他组件。这个技术不是很常见，但是他在两个场景中特别有用：

- [转发 ref 到 DOM 组件](https://reactjs.org/docs/forwarding-refs.html#forwarding-refs-to-dom-components)

- [在高阶组件中转发 ref](https://reactjs.org/docs/forwarding-refs.html#forwarding-refs-in-higher-order-components)

`React.forwardRef`接收一个渲染函数作为参数。React 将使用`props`和`ref`作为参数调用这个函数。这个函数应该返回一个 React 节点：
```
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

在前面的例子中，React 传递一个`ref`给`<FancyButton ref={ref}>`元素作为第二个参数到`React.forwardRef`调用内的渲染函数。这个渲染函数传递`ref`到`<button ref={ref}>`元素。

作为结果，在 React 绑定 ref 之后，`ref.current`将直接指向`<button>`DOM 元素实例。

了解更多详情，查阅[转发 ref](https://reactjs.org/docs/forwarding-refs.html)。

### React.lazy

`React.lazy`让你定一个动态加载的组件。这帮助减少包大小，延迟加载初始化渲染不需要的组件。

你可以从我们的[代码分割文档](https://reactjs.org/docs/code-splitting.html#reactlazy)了解如何使用。你可能想要查阅[这篇文章](https://medium.com/@pomber/lazy-loading-and-preloading-components-in-react-16-6-804de091c82d)详细解释怎样使用它。

```jsx harmony
// This component is loaded dynamically
const SomeComponent = React.lazy(() => import('./SomeComponent'));
```

注意渲染`lazy`组件需要在渲染树的高处有一个`<React.Suspense>`组件。这是你指定加载指示器的方式。

注意：使用`React.lazy`和动态导入需要 Promise 在 JS 环境中可用。这需要在 IE11 和以下版本有一个 polyfill。

### React.Suspense

`React.suspense`让你指定加载指示器，防止它下面的组件树中的组件还没有准备好渲染。现在，懒加载组件是`<React.Suspense>`支持的唯一场景：
```jsx harmony
// This component is loaded dynamically
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    // Displays <Spinner> until OtherComponent loads
    <React.Suspense fallback={<Spinner />}>
      <div>
        <OtherComponent />
      </div>
    </React.Suspense>
  );
}
```
这记录在我们的[代码分割指南](https://reactjs.org/docs/code-splitting.html#reactlazy)。注意`lazy`组件可以在`Suspense`树很深的地方 -- 他不需要去包裹他们的每一个。最好的实践方式是将`<Suspense>`放在你想要看到加载指示器的地方，但是`lazy()`可以放在任何你想要实现代码分割的地方。

景观这个当前还不支持，在未来，我们计划去让`Suspense`处理更多场景，比如数据获取。你可以在[我们的 roadmap](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html) 了解这些。

注意：`React.lazy()`和`<Reacrt.Suspense>`还不支持`ReactDOMServer`。这是一个已知的限制，将会在未来解决。
