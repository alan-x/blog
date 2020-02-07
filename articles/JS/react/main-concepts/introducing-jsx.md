### 介绍 JSX

考虑这个变量声明：
```jsx harmony
const element = <h1>Hello, world!</h1>;
```

这个有趣的标签语法既不是一个字符串，也不是一个 HTML。

这叫做 JSX，是 JavaScript的语法扩展。我们推荐使用它和 React 去描述 UI 应该是啥样。JSX 可能会想你想起模版语言，但是它的能力完全来自 JavaScript。

JSX 产生 React "元素"。我们将在[下一章节]()探索他们渲染成 DOM。下面，你可以找到让你开始的 JSX 基础。


### 为什么是 JSX？
React 拥抱渲染逻辑和 UI 逻辑耦合的事实：事件是如何处理的，状态是如何随着时间改变，数据是如何准备用于显示的。

与其通过将标签和逻辑放到分离的文件的人工分离技术，React 使用叫做"组件"的组合单元包含两者，[分离关注点](https://en.wikipedia.org/wiki/Separation_of_concerns)。我们将在[更深入的章节](https://reactjs.org/docs/components-and-props.html)回顾组件，但是如果你还没习惯在 JS 中放置标签，[这个讨论](https://www.youtube.com/watch?v=x7cQ3mrcKaY)可能使你信服。

React [不需要](https://reactjs.org/docs/react-without-jsx.html) 使用 JSX，但是在 JavaScript 代码内配合 UI的时候，大部分人觉得他作为一个视觉辅助很有帮助。它允许 React 去显示更多有用的错误和警告信息。

不用担心，开始吧！

### 在 JSX 中嵌入表达式

在下面的例子中，我们声明一个变量叫做`name`并在 JSX 内通过包裹它到花括号中使用它：
```jsx harmony
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
基于 JSX 你可以放置任何有效的[JavaScript 表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_Operators#Expressions)到花括号中。比如，`2 + 2`，`user.firstName`，或者`formatName(user)`都是有效的 JavaScript 表达式。

在下面的例子中，我们嵌入了 JavaScript 函数`formatName(result)`调用的结果到一个`<h1>`元素。

```jsx harmony
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
[在 CodePen 中尝试它](https://reactjs.org/redirect-to-codepen/introducing-jsx)

为了可读性，我们分离 JSX 到多行。尽管它不是必须的，我们也推荐包裹它到括号中，避免[自动封号插入](https://stackoverflow.com/q/2846283)的陷阱。

### JSX 也是表达式

编译之后，JSX 表达式变成常规 JavaScript 函数调用，并得到 JavaScript 对象。

这意味着你可以在 `if` 语句和`for`循环使用 JSX，赋值它到变量，作为参数接受它，从函数返回它：
```jsx harmony
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

### 使用 JSX 指定属性

你可能使用引号去指定字符串字面量作为属性：
```jsx harmony
const element = <div tabIndex="0"></div>;
```
你也可以在一个属性中使用花括号去嵌入一个 JavaScript 表达式：
```jsx harmony
const element = <img src={user.avatarUrl}></img>;
```
当嵌入一个 JavaScript 表达式到一个属性的时候，不要使用引号包裹花括号。你应该使用引号（为字符串值）或者花括号（为表达式），但是不要对相同属性同时使用两者。

警告：因为 JSX 更接近 JavaScript 而不是 HTML，React DOM 使用`驼峰`属性命名约定而不是 HTML 属性名。

比如，JSX 中`class`变成[className](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)，并且`tabIndex`变成[tabIndex](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabIndex)。

### 使用 JSX 指定子元素

如果一个标签是空的，你可能使用 `/>` 立即关闭它，就像 XML：
```jsx harmony
const element = <img src={user.avatarUrl} />;
```
JSX 标签可能包含子元素：
```jsx harmony
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX 防止注入工具

在 JSX 嵌入用户输入是安全的：

```jsx harmony
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```

默认情况下，React DOM [转义](https://stackoverflow.com/questions/7381974/which-characters-need-to-be-escaped-on-html)嵌入到 JSX 中的任何值，在渲染他们之前。因此它确保你可以不注入任何你没有显式卸载应用中的东西。任何东西都会转化为字符串，在被渲染之前。这帮助防止[XSS(cross-site-scripting)](https://en.wikipedia.org/wiki/Cross-site_scripting)攻击。

### JSX 表示对象

Babel 编译 JSX 到 `React.createElement()` 调用。

这两个例子是相同的：
```jsx harmony
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```jsx harmony
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

`React.createElement()`执行一些检查去帮助你编写没有 bug 的代码，但是本质上，它创建一个类似这个的对象：
```jsx harmony
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```
这些对象叫做"React 元素"。你可以认为他们描述你在屏幕上看到的东西。React 读取这些对象并使用他们去构造 DOM 并保持它更新。

我们将在下一章节探索渲染 React 元素到 DOM。

提示：我们推荐你选择的的编辑器使用["Babel"语言定义](https://babeljs.io/docs/editors)这样 ES6 和 JSX 代码都会适合的高亮。 





















