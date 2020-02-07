[原文地址](https://reactjs.org/docs/conditional-rendering.html)
### 条件化请求

在 React，你可以创建不同的组件封装你需要的行为。然后，你可以只渲染他们的一部分，取决于你应用的状态。

React 中的条件化渲染和 JavaScript 中的条件化工作方式一致。使用 JavaScript 操作符，比如[if](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/if...else)或者[conditional operator](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)去创建元素表示当前状态，让 React 更新 UI 去匹配他们。

假设有两个组件：
```jsx harmony
function UserGreeting(props) {
  return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
  return <h1>Please sign up.</h1>;
}
```
我们将创建一个`Greeting`组件，是否显示这些组件依赖于一个用户是否登陆：
```jsx harmony
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/ZpVxNq?editors=0011)

这个例子渲染一个不同的欢迎，依赖于`isLoggedIn`属性。

你可以使用变量去存储元素。这可以帮助你条件化渲染组件的一部分，当输出的其他部分没有改变的时候。

假设这两个新组件表示登出和登陆按钮：
```jsx harmony
function LoginButton(props) {
  return (
    <button onClick={props.onClick}>
      Login
    </button>
  );
}

function LogoutButton(props) {
  return (
    <button onClick={props.onClick}>
      Logout
    </button>
  );
}
```
在下面的例子，我们将创建一个[有状态的组件](https://reactjs.org/docs/state-and-lifecycle.html#adding-local-state-to-a-class)，叫做`LoginButton`。

它将渲染`<LoginButton />`或者`<LogoutButton/>`，取决于它当前的状态。它将渲染前一个例子中的`<Greeting />`。

```jsx harmony
class LoginControl extends React.Component {
  constructor(props) {
    super(props);
    this.handleLoginClick = this.handleLoginClick.bind(this);
    this.handleLogoutClick = this.handleLogoutClick.bind(this);
    this.state = {isLoggedIn: false};
  }

  handleLoginClick() {
    this.setState({isLoggedIn: true});
  }

  handleLogoutClick() {
    this.setState({isLoggedIn: false});
  }

  render() {
    const isLoggedIn = this.state.isLoggedIn;
    let button;

    if (isLoggedIn) {
      button = <LogoutButton onClick={this.handleLogoutClick} />;
    } else {
      button = <LoginButton onClick={this.handleLoginClick} />;
    }

    return (
      <div>
        <Greeting isLoggedIn={isLoggedIn} />
        {button}
      </div>
    );
  }
}

ReactDOM.render(
  <LoginControl />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/QKzAgB?editors=0010)

当声明一个变量并使用一个`if`语句是一个很好的方式去渲染一个组件，有时候你可能想要使用一个更短的语法。还有一些其他方式在 JSX 内联条件，在下面解释。


### 使用逻辑 && 操作符内联
通过包裹他们到花括号，你可能[在 JSX 中嵌入任何表达式](https://reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)。这包含 JavaScript 逻辑`&&`操作符。他可以处理包含一个元素的条件化：
```jsx harmony
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```
[在 CodePen 尝试它](https://codepen.io/gaearon/pen/ozJddz?editors=0010)

它可以工作因为在 JavaScript 中，`true && expression`总是执行`expression`，并且`false && expression`总是执行`false`。

然而，如果条件是`true`，`&&`之后的元素总是出现在输出。如果它是`false`，React 将会忽略并跳过它。

### 使用条件化操作符内联 If-Else

内联条件化渲染元素的另一个方法是使用 JavaScript 的条件化操作符[condition ? true: false](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Conditional_Operator)。

在下面的例子中，我们使用它去条件化渲染一个更小的块的文字。
```jsx harmony
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```

它也可以用于大表达式，尽管不太明显：
```jsx harmony
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      {isLoggedIn ? (
        <LogoutButton onClick={this.handleLogoutClick} />
      ) : (
        <LoginButton onClick={this.handleLoginClick} />
      )}
    </div>
  );
}
```
就像在 JavaScript，这取决于你去选择一个方式风格，基于你和你的团队觉得什么更易读。记住当条件太复杂的时候，最好[抽取一个组件](https://reactjs.org/docs/components-and-props.html#extracting-components)

### 防止组件渲染

在很罕见的场景中，你可能想要一个组件去隐藏自身，尽管他被其他组件渲染。为了做到这个，返回`null`替代原本的渲染输出。

在下面的例子中，`<WarningBanner />`的渲染取决于`warn`属性的值。如果这个属性的值是`false`，则组件不渲染：
```jsx harmony
function WarningBanner(props) {
  if (!props.warn) {
    return null;
  }

  return (
    <div className="warning">
      Warning!
    </div>
  );
}

class Page extends React.Component {
  constructor(props) {
    super(props);
    this.state = {showWarning: true};
    this.handleToggleClick = this.handleToggleClick.bind(this);
  }

  handleToggleClick() {
    this.setState(state => ({
      showWarning: !state.showWarning
    }));
  }

  render() {
    return (
      <div>
        <WarningBanner warn={this.state.showWarning} />
        <button onClick={this.handleToggleClick}>
          {this.state.showWarning ? 'Hide' : 'Show'}
        </button>
      </div>
    );
  }
}

ReactDOM.render(
  <Page />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/Xjoqwm?editors=0010)

从一个组件的`render`方法返回`null`不影响触发组件的生命周期方法。比如`componentDidUpdate`将依旧被调用。












