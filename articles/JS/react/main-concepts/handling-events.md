[原文地址](https://reactjs.org/docs/handling-events.html)
### 处理事件

使用 React 处理事件和 DOM 元素处理事件非常像。有一些语法上的不同：

- React 事件使用驼峰命名，而不是小写。

- 使用 JSX 你传递一个函数作为事件处理器，而不是一个字符串

比如，HTML：
```jsx harmony
<button onclick="activateLasers()">
  Activate Lasers
</button>
```
和 React 有点不同
```jsx harmony
<button onClick={activateLasers}>
  Activate Lasers
</button>
```
另一个不同是在 React 你不能返回`false`去防止默认的行为。你必须显式调用`preventDefault`。比如，简单 HTML，为了防止默认链接行为打开一个新的页面，你可以这么写：
```jsx harmony
<a href="#" onclick="console.log('The link was clicked.'); return false">
  Click me
</a>
```
在 React，这可以替代为：
```jsx harmony
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}
```
这里，`e`是一个合成事件，React 基于[W3C 规格](https://www.w3.org/TR/DOM-Level-3-Events/)定义这些合成事件，这样你不需要去担心跨浏览器兼容。查阅[SyntheticEvent](https://reactjs.org/docs/events.html)引用指南去学习更多。

当使用 React，你通常不需要调用`addEventListener`去添加监听器到 DOM 元素，在他创建之后。相反，只提供一个监听器，当元素初始化渲染的时候。

当你使用[ES6 类](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)定义一个组件，事件处理器的一个常见模式是类中的一个方法。比如，这个`Toggle`组件渲染一个按钮，让用户在"ON"和"OFF"状态切换：
```jsx harmony
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // This binding is necessary to make `this` work in the callback
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}

ReactDOM.render(
  <Toggle />,
  document.getElementById('root')
);
```
[在 CodePen 尝试它](https://codepen.io/gaearon/pen/xEmzGg?editors=0010)

你需要小心`this`在 JSX 回调中的意义。在 JavaScript，类方法并不默认[绑定](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind)。如果你忘记绑定`this.handleClick`并传递它到`onClick`，当函数实际执行的时候，`this`将会是`undefined`。

这不是 React 特定的行为；这是[JavaScript 函数工作方式](https://www.smashingmagazine.com/2014/01/understanding-javascript-function-prototype-bind/)的一部分。通常来说，如果你引用一个方法，没有`()`跟随它，比如`onClick={this.handleClick}`，你应该绑定这个方法。

如果调用`bind`惹恼你，有两个方式可以解决它。如果你使用体验性的[public class field 语法](https://babeljs.io/docs/plugins/transform-class-properties/)，你可以使用类域去正确绑定回调。

```jsx harmony
class LoggingButton extends React.Component {
  // This syntax ensures `this` is bound within handleClick.
  // Warning: this is *experimental* syntax.
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```
这个语法在[Create React App](https://github.com/facebookincubator/create-react-app) 中默认启用。

如果你不使用类域语法，你可以在回调中使用[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)：
```jsx harmony
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    // This syntax ensures `this` is bound within handleClick
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```
这个语法的问题是每一次`LoggingButton`渲染的时候都会创建不同的回调。在大部分场景，这是好的。但是，如果这个回调作为一个属性传递到底层组件，这些组件可能会执行一次额外的重新渲染。我们通常推荐在构造器中绑定或者使用类域语法，去避免这类的性能问题。


### 传递参数到事件处理器。

在循环中，传递一个额外的参数到一个事件处理器是很常见的。比如，如果`id`是行 ID，下面都会工作：
```jsx harmony
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```
前面的两行都是相同的，并且各自使用[箭头函数](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)和[Function.prototype.bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind)。

在两个场景中，`e`参数表示 React 事件，将会作为 ID 之后的第二个参数传递。使用一个箭头函数，我们需要显式传递，但是使用`bind`，任何其他参数将自动转发。












