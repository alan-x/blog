[原文地址](https://reactjs.org/docs/react-dom.html)
### ReactDOM

如果你从一个`<script>`标签加载 React，这些顶级 API 可以在全局的`ReactDOM`访问。如果你使用 ES6 和 npm，你可以编写`import ReactDOM from 'react-dom'`。如果你使用 ES5 和 npm，你可以使用`var ReactDOM = require('react-dom')`。

### 概述
`react-dom`包提供 DOM 指定方法，可以在你的应用顶级使用，或者如果你需要，也可以作为 React 模型的紧急出口。你的大部分组件应该不需要用这个模块。

- [render()](https://reactjs.org/docs/react-dom.html#render)
- [hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)
- [unmountComponentAtNode()](https://reactjs.org/docs/react-dom.html#unmountcomponentatnode)
- [findDOMNode()](https://reactjs.org/docs/react-dom.html#finddomnode)
- [createPortal()](https://reactjs.org/docs/react-dom.html#createportal)


### 浏览器支持

React 支持所有的流行的浏览器，包括 IE9 和之前，尽管对于旧的浏览器，比如 IE9 和 IE10 需要[需要一个 polyfill](https://reactjs.org/docs/javascript-environment-requirements.html)。

注意：我们不支持那些不支持 ES5 方法的旧的浏览器，但是你可能，发现你的应用通过填充类似[es5-shim 和 es5 sham](https://github.com/es-shims/es5-shim)之后在旧的浏览器也能工作。如果你选择这条路，你就是在走自己的路。

### 索引

### render()
```jsx harmony
ReactDOM.render(element, container[, callback])
```
 渲染一个 React 元素到`container`提供的 DOM，并返回一个组件[索引](https://reactjs.org/docs/more-about-refs.html)（或者为[无状态组件](https://reactjs.org/docs/components-and-props.html#function-and-class-components)返回 null）。
 
 如果 React 元素前面已经渲染到`container`这将会执行一个更新，并且只操作必要的 DOM 去反映最新的 React 元素。
 
 如果提供了可选的回调，他将会在组件渲染和更新之后执行。

注意：`ReactDOM.render()`控制你传输的容器的内容。任何存在的 DOM 元素在第一次调用的时候都会被覆盖。后续调用使用 React 的 DOM diff 算法做高效更新。

`ReactDom.render()`不修改容器节点（只修改容器的子节点）。它可能插入组件到已存在的 DOM 节点，不需要重写存在的子元素。

`ReactDOM.render()`现在返回一个根`ReactComponent`实例的索引。然而，使用这个返回值是遗留的，并且需要避免，因为在未来的 React 版本中，可能在某些场景异步渲染。如果你需要引用根`ReactComponent`实例，更好的解决方案是去半丁一个[回调 ref](https://reactjs.org/docs/more-about-refs.html#the-ref-callback-attribute) 到根元素。

使用`ReactDOM.render()`去混合服务端渲染的容器是废弃的，将在 React 17 移除。使用[hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)代替。

### hydrate()
```jsx harmony
ReactDOM.hydrate(element, container[, callback])
```
和[render()](https://reactjs.org/docs/react-dom.html#render)一样，但是它用于混合一个容器，它的 HTML 内容是由[ReactDOMServer](https://reactjs.org/docs/react-dom-server.html)渲染的。React 将尝试绑定事件处理器到已存在的标签上。

React认为渲染的内容在服务端和客户端相同。他可以修补不同的文本内容，但是你应该对待没有匹配的为一个 bug 并修复他们。在卡法模式，React 在混合期间警告错误。不保证属性不同会被修复，为了防止错误匹配。在大部分应用，错误匹配是非常罕见的，因此验证所有标签会非常昂贵，这是非常重要的性能原因。

如果你需要在服务端和客户端渲染不同的内容，你可以做两次渲染。在客户端上渲染不同内容的组件可以读取像`this.state.isClient`的状态变量，你可以在`componentDidMount()`设置为`true`。这种方式，初始化渲染将会渲染和服务端相同的结果，避免错误匹配，但是此外，额外传输将会在混合之后同步发生。注意，这个方式将让你的组件更慢，因为他们渲染两次，因此，小心使用。

注意慢速链接的用户体验。JavaScript 代码可能比 HTML 渲染慢非常多，因此如果你针对客户端传输渲染不同的内容，过渡会很糟糕。然而，如果执行的很好，它对在服务端渲染一个应用"外壳"很有益，并只在客户端渲染额外组件。为了了解怎么做到这个并且不导致错误匹配问题，参考前面段落的的解释。

### unmountComponentAtNode()
```jsx harmony
ReactDOM.unmountComponentAtNode(container)
```
从 DOM 中移除已经挂载的 React 组件并清理它的事件处理器和状态。如果没有组件挂载在容器中，调用这个函数不做任何事儿。如果组件卸载了，返回`true`，如果没有组件被卸载，返回`false`。

### findDOMNode()

注意：`findDOMNode`是一个紧急出口，用于访问底层 DOM 节点。在大部分场景中，使用这个紧急出口是不鼓励的，因为它破坏了组件抽象。[它在 StrictMode 被废弃了](https://reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)

```jsx harmony
ReactDOM.findDOMNode(component)
```

如果组件已经挂载到 DOM 中，这返回对应的原生 DOM 元素。这个方法对于在 DOM 之外读取值很有帮助，比如表单域的值或者执行 DOM 测量。**在大部分场景中，你可以绑定一个 ref 到 DOM 节点，完全避免使用 findDOMNode**。

当一个组件渲染`null`或者`false`，`findDOMNode`返回`null`。当一个组件渲染为字符串，`findDOMNode`返回一个文本 DOM 节点，包含这个值。React 16 中，一个组件可能返回一个片段，包含多个子节点，这种场景，`findDOMNode`将返回 DOM 节点对应的第一个非空子元素。

注意：`findDOMNode`只在挂载的组件上工作（也就是，已经放置在 DOM 上的组件）。如果你尝试在一个还没挂载的组件上（比如在还没有创建的组件的`render()`调用`findDOMNode()`），将会抛出一个异常。

### createPortal()
```jsx harmony
ReactDOM.createPortal(child, container)
```
创建一个传送门。传送门提供一个方式去[渲染子节点到一个 DOM 节点，该节点存在于 DOM 组件层级之外](https://reactjs.org/docs/portals.html)。












