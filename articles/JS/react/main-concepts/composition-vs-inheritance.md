### 组合 vs 继承

React 有一个强大的组合模型，并且推荐使用组合替代继承在组件间重用代码。

在这个章节，我们将考虑一些新的 React 开发者通常会使用继承问题，并显示我们如何使用组合解决他们。

### 容器

一些组件无法事先知道他们的子组件。这对于像`Sidebar`或者`Dialog`之类表示常用"盒子"的组件特别常见。

我们推荐这类组件使用特殊的`children`属性直接传递子元素到他们的输出：
```jsx harmony
function FancyBorder(props) {
  return (
    <div className={'FancyBorder FancyBorder-' + props.color}>
      {props.children}
    </div>
  );
}
```
这让其他组件传递任意子元素给他们，通过嵌套 JSX：
```jsx harmony
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        Welcome
      </h1>
      <p className="Dialog-message">
        Thank you for visiting our spacecraft!
      </p>
    </FancyBorder>
  );
}
```
[在 CodePen 尝试它]()
`<FancyBorder>` JSX 标签内的任何东西通过`children`属性传递到`FancyBorder`。因为`FancyBorder`在一个`<div>`渲染`{props.children}`，传递过来的元素出现在最终的输出。

尽管这不太常见，有时候组件内你可能需要多个"插槽"。在这类场景你可以使用你自己惯用的而不是`children`：
```jsx harmony
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```
[在 CodePen 中尝试它]()

类似`<Contacts />`和`<Chat />`之类的 React 元素只是对象，这样你可以像传递其他数据一样传递他们作为属性。这个方式可能让你想起其他库的"插槽"，但是在 React 你传递什么作为属性是没有限制的。


### 特殊

有时候我们认为一些组件是另一些组件的一个"特殊场景"。比如，我们可能说`WelcomeDialog`是`Dialog`的一个特殊场景。

在 React，这也可以通过组合事先，一个更"特殊"的组件渲染一个更"通用"的并使用属性配置它：
```jsx harmony
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog
      title="Welcome"
      message="Thank you for visiting our spacecraft!" />

  );
}
```
[在 CodePen 尝试它]()

组合对于定义为类的组件工作的也很好。

```jsx harmony
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">
        {props.title}
      </h1>
      <p className="Dialog-message">
        {props.message}
      </p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = {login: ''};
  }

  render() {
    return (
      <Dialog title="Mars Exploration Program"
              message="How should we refer to you?">
        <input value={this.state.login}
               onChange={this.handleChange} />

        <button onClick={this.handleSignUp}>
          Sign Me Up!
        </button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({login: e.target.value});
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```
[在 CodePen 中尝试它]()


### 继承怎么样？

在 Facebook，我们在成千上万个组件使用 React，我们没有找到任何使用场景推荐创建组件继承体系。

属性和组合提供给你所有的弹性，你需要去自定义一个组件的外观和行为，在一个显示和安全的方式。记住，组件可以接受任何属性，包括原生值，React 元素，或者函数。

如果你想要在组件间重用非 UI 功能，我们推荐抽取它到一个分离的 JavaScript 模块。组件可能引入它，并使用那个函数，对象，或者一个类，不需要继承它。
















