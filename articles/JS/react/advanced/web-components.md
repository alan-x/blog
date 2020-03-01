[原文地址](https://reactjs.org/docs/web-components.html)
### web component

React 和[Web Component](https://developer.mozilla.org/en-US/docs/Web/Web_Components) 是为了解决不同的问题。Web Component 提供了对可重用组件的强力封装，然而 React 提供了保持 DOM 和你的数据同步的声明式库。两个目标是相辅相成的。作为开发者，你可以自由的在你的 Web Component 使用 React，或者在 React 使用 Web Component，或者同时。


### 在 React 中使用 Web Component

```jsx harmony
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```

注意：Web Component 通常暴露一个命令式 API，比如，一个`video` Web Component 可能暴露`play()`和`pause()`函数。为了访问一个 Web Component 的命令式 API，你需要去使用一个 ref 和 DOM 节点直接交互。如果你使用第三方 Web Component，最好的解决方案是编写一个 React 组件，表现的像你的 Web Component 的包装器。

Web Component 发出的事件可能不会正确的在 React 渲染树中传播。你需要在你的 React 组件中手动绑定事件处理器。

一个常见的混淆是 Web Component 使用"class"替代"className"。
```jsx harmony
function BrickFlipbox() {
  return (
    <brick-flipbox class="demo">
      <div>front</div>
      <div>back</div>
    </brick-flipbox>
  );
}
```

### 在 Web Component 中使用 React
```jsx harmony
class XSearch extends HTMLElement {
  connectedCallback() {
    const mountPoint = document.createElement('span');
    this.attachShadow({ mode: 'open' }).appendChild(mountPoint);

    const name = this.getAttribute('name');
    const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
    ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
  }
}
customElements.define('x-search', XSearch);
```
注意：如果你使用 Babel 转化类，这个代码**不会**生效。查看[这个问题](https://github.com/w3c/webcomponents/issues/587)的讨论。在你加载你的 web component 之前包含[custom-element-es5-adapter](https://github.com/webcomponents/polyfills/tree/master/packages/webcomponentsjs#custom-elements-es5-adapterjs)修复这个问题。
