[原文地址](https://reactjs.org/docs/react-without-jsx.html)
### React without JSX

JSX 对于使用 React 不是必须的。使用 React 不使用 JSX 特别方便，当你不想要在你的构建环境设置编译的时候。

每一个 JSX 元素都只是`React.createElement(component, props, ...children)`调用的语法糖。因此，任何你能够使用 JSX 完成的，也能用普通 
JavaScript 完成。

比如，这段使用 JSX 编写的代码：
```jsx harmony
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.toWhat}</div>;
  }
}

ReactDOM.render(
  <Hello toWhat="World" />,
  document.getElementById('root')
);
```
可以转化为不使用 JSX 的代码：
```jsx harmony
class Hello extends React.Component {
  render() {
    return React.createElement('div', null, `Hello ${this.props.toWhat}`);
  }
}

ReactDOM.render(
  React.createElement(Hello, {toWhat: 'World'}, null),
  document.getElementById('root')
);
```
如果你好奇想要看更多关于 JSX 如何转化到 JavaScript 的例子，你可以尝试[在线 Babel 编译器]()。

组件可以提供为字符串，为`React.Component`的子类，或者普通函数。


如果你经常输入`React.createElement`，一个常见的模式是赋值到缩写：
```jsx harmony
const e = React.createElement;

ReactDOM.render(
  e('div', null, 'Hello World'),
  document.getElementById('root')
);
```
如果你使用`React.createElement`的缩写形式，他是使用 React 不使用 JSX 最方便的方式。

此外，你可以查阅社区项目，比如[react-hyperscript](https://github.com/mlmorg/react-hyperscript)和[hyperscript-helpers](https://github.com/ohanhi/hyperscript-helpers)，它提供更简洁的语法。
