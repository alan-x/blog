[原文地址](https://reactjs.org/docs/components-and-props.html)
### 组件和属性

 组件让你分离 UI 到独立的，可重用的块，并且独立的思考每一块。这个页面提供了一个关于组件的概念介绍。你可以[在这里找到组件的详细 API 索引](https://reactjs.org/docs/react-component.html)。

概念上，组件就像 JavaScript 的函数。他们接受任意的输入（叫做"props"）并且返回描述什么应该出现在屏幕上的 React 元素。

### 函数和类组件
定义组件最简单的方式是写一个 JavaScript 函数：
```jsx harmony
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

这个函数是一个有效的 React 组件，因为它接受一个单独的"props"（表示 properties）对象参数，接受数据，并返回一个 React 元素。我们把这些组件叫做"函数组件"因为他们就是 JavaScript 函数。

你可以使用一个[ES6 类](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)去定义一个组件。

```jsx harmony
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

前面的两个组件从 React 的角度来看都是一样的。

类有一些额外的特性，我们将在[下一个章节](https://reactjs.org/docs/state-and-lifecycle.html)讨论。到那时，我们将使用函数组件，因为他们的简洁。

### 渲染一个组件

前面，我们只遇到表示 DOM 标签的 React 元素：
```jsx harmony
const element = <div />;
```
然而，元素也可以表示用户定义的组件：
```jsx harmony
const element = <Welcome name="Sara" />;
```
当 React 看到一个元素表示一个用户定义的组件，它传递 JSX 属性到这个组件为单一的对象。我们叫这个对象为"props"。

比如，这个代表渲染"Hello，Sara"到页面：
```jsx harmony
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
[在 CodePen 尝试它](https://reactjs.org/redirect-to-codepen/components-and-props/rendering-a-component)

让我们回顾以下在这个例子中发生了什么：

1. 我们使用`<Welcome name="Sara" />`调用`ReactDOM.render`。

2. React 调用使用`{name:'Sara'}`调用`Welcome`组件。

3. 我们的`Welcome`组件返回一个`<h1>Hello,Sara</h1>`元素作为结果。

4. React DOM 有效的更新 DOM 去匹配`<h1>Hello, Sara</h1>`。

注意：始终以大写字母开始作为组件名。

React 对待小写字母开头的组件为 DOM 标签。比如，`<div />`表示一个 HTML div 标签，但是`<Welcome />`表示一个组件，并且需要`Welcome`在范围中。

了解更多这个约定的原因，请阅读[深入 JSX](https://reactjs.org/docs/jsx-in-depth.html#user-defined-components-must-be-capitalized)。

### 组合组件

组件可以在他们的输出结果引用其他组件。这让我们为任意级别详细使用相同的组件抽象。一个按钮，一个表单，一个弹窗，一个屏幕：在 React 应用，这些所有通常表示为组件。

比如，我们可以创建一个`App`组件，渲染`Welcome`多次：
```jsx harmony
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试它](https://reactjs.org/redirect-to-codepen/components-and-props/composing-components)

通常，新 React 应用有一个单一`App`组件在最顶部。然而，如果你整合 React 到一个存在的应用，你可能使用一个小的组件，比如`Button`开始并逐渐以你的方式到扩展到视图顶级层次。

### 抽取组件

不要担心分离组件到更小的组件。

比如，假设这个`Component`组件：
```jsx harmony
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
[在 CodePen 中尝试](https://reactjs.org/redirect-to-codepen/components-and-props/extracting-components)

它接受`author`（一个对象），`text`（一个字符串）,和`data`（一个日期）作为属性，并表述一个社区媒体网站的评论。

这个组件改变很复杂，因为所有都是嵌套的，并且也很难重用它的部分。现在从它里面抽取一些组件。

首先，我们将抽取`Avatar`：
```jsx harmony
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />

  );
}
```

`Avatar`不需要知道它是被渲染到一个`Comment`内部。这也是为什么我们给它的属性一个更通用的名字：`user`而不是`author`。

我们推荐以组件的观点去命名属性，而不是使用的上下文。

我们现在可以简化`Comment`一点：
```jsx harmony
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
下一步，我们将抽取一个`UserInfo`组件，它在用户的姓名后面渲染一个`Avatar`：
```jsx harmony
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```
这让我们更加简化`Comment`：
```jsx harmony
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
[在 CodePen 尝试它](https://reactjs.org/redirect-to-codepen/components-and-props/extracting-components-continued)

抽取组件可能起先看起来像一个繁重的工作，但是有一个可重用的组件调色板是非常值得的。一个好的规则是如果你 UI 的一部分使用多次（`Button`，`Panel`，`Avatar`），或者它自己本身够复杂（`App`，`FeedStory`，`Comment`）,都是可重用组件的候选。

### props 是只读的

无论你声明一个组件为[一个函数或者一个类](https://reactjs.org/docs/components-and-props.html#function-and-class-components)，它必须永远不修改它自己的属性。假设这个`sum`函数：
```jsx harmony
function sum(a, b) {
  return a + b;
}
```

这类函数叫做["纯函数"](https://en.wikipedia.org/wiki/Pure_function)因为他们不尝试去改变他们的输入，并且总是为相同的输入返回相同的结果。

相反，这个函数是非纯净的，因为它改变了自己的输入：
```jsx harmony
function withdraw(account, amount) {
  account.total -= amount;
}
```

React 是足够灵活的，但是它有一个严格的规则：

**所有 React 组件必须表现的像一个纯函数，尊重他们的属性。**

当然，应用 UI 是动态的并随着时间改变。在[下一个章节](https://reactjs.org/docs/state-and-lifecycle.html)，我们将引入一个新的概念，关于"状态"。状态允许 React 组件在用户动作、网络响应，和其他的的响应中去改变他们的输出，不需要遵守这个规则。





























