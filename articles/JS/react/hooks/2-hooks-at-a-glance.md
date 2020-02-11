 [原文地址](https://reactjs.org/docs/hooks-overview.html)
### Hooks at a Glance

Hooks 是 React 16.8 新添加的功能。他们让你不写类就能使用状态和其他 React 功能。

Hooks 是[向后兼容的](https://reactjs.org/docs/hooks-intro.html#no-breaking-changes)。这个页面为有经验的 React 用户提供了一个关于 Hooks 的概览。这是一个快速概览。如果你感觉困惑，查看像这样的黄色盒子：

> **详细解释**
> 阅读[动机](https://reactjs.org/docs/hooks-intro.html#motivation)去学习为什么在 React 中引入 Hooks

↑↑↑ 每一个章节使用像这样的黄色盒子结束。他们链接到详细的解释。

### 📌 State Hook

这个例子渲染一个计数器。当你点击按钮，增加值：
```jsx harmony
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

这里，`useState`是一个 Hook（我们将在稍后讨论这意味着什么）。我们在一个函数组件的内部调用它去添加一个本地状态。React 将会在每次重新渲染的时候保持这个状态。`useState`返回一对：当前状态值和一个让你更新它的函数。你可以在事件处理器或者其他调用这个函数。它和类的`this.setState`很像，除了它不合并新的和旧的状态。（我们将会在 [Using the State Hook](https://reactjs.org/docs/hooks-state.html) 展示一个例子来比较`userState`和`this.state`）。


`useState`唯一的参数是初始状态。在前面的例子中，它是`0`是因为我们的计数器从 0 开始。注意，不像`this.state`，这里的状态不必是个对象 - 尽管如果你需要，它可以是。初始化状态参数只有在第一次渲染使用。

### 声明多个状态变量

你可以在一个单独的组件多次使用 State Hook：
```jsx harmony
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

数组结构语法让我们给我们通过`setState`声明的变量赋予不同的名字。这些名字不是`useState`API 的一部分。相反，React 假设如果你调用`useState`API 很多次，你在每一次渲染都以相同的顺序来做。稍后我们将会讨论为什么这个有效和什么时候有用。

### 但是什么是一个 Hook？

Hooks 是函数，让你在函数组件"勾住"React 状态和生命周期功能。Hooks 无法在类内部工作 -- 他们让你不使用类来使用 React。（我们不推荐突然重写已经存在的组件，但是你可以在新的组件开始使用 Hooks，如果你喜欢。）

React 提供了少量类似`useState`的内置 Hooks。你也可以创建你自己的 Hooks，在不同组件间去重用有状态的行为。我们先关注内置的 Hooks。

> 详细解释
> 你可以在独立的章节学习更多关于 State Hook：Using the State Hook。

### ⚡️ Effect Hook

你之前可能在 React 组件执行数据获取，订阅，或者在 React 之前手动改变 DOM。我们称这种操作为"副作用"（或者简称为"作用"）因为他们影响其他组件，并且不能在渲染期间执行。

Effect Hook ，`useEffects`，添加从函数组件中执行副作用的能力。它和 React 类中`componentDidMount`，`componentDidUpdate`，`componentWillMount`的目标相同，但集合称一个单一的 API（我们将会在 Using the Effect Hook 使用例子展示`useEffect`和这些方法的比较。）

比如，这个组件在 React 更新 DOM 之后设置文档标题：
```jsx harmony
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

当你调用`useEffect`，在刷新你的改变到 DOM 之后，你告诉 React 执行你的"作用"函数。作用声明在组件内部，这样他们可以访问它的组件和状态。默认，React 在每一次渲染之后执行作用 -- 包括第一次执行。（我们将在 [Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html) 讨论更多和类声明周期的比较。）

作用可能也可选的指定之后怎样去"清理"他们，通过返回一个函数。比如，这个组件使用一个作用去订阅一个朋友的在线状态，并通过取消订阅清理：
```jsx harmony
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

在这个例子中，React 将会在组件卸载的时候取消订阅我们的`ChatAPI`，同时在后续渲染的时候再次执行作用。（如果你想要，有一个方式去告诉 React 跳过重新订阅，如果我们传递给`ChartAPI`测`props.friend.id`没有改变。）

就像`useState`，你可以在一个组件中多次使用一个作用：
```jsx harmony
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```
> 详细解释
> 你可以在一个独立的页面学习更多关于`useEffect`：[Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html)。


### ✌️ Hooks 的规则

Hooks 是 JavaScript 函数，但是他们有两个额外规则约束：

- 只**在顶层**调用 Hooks。不要在循环，条件，或者嵌套函数在调用 Hooks。

- 只**在 React 函数组件**内调用 Hooks。不要在常规 JavaScript 函数内调用 Hooks。（只有唯一一个调用 Hooks 的有效地方 -- 你自己的自定义 Hooks。我们将在稍后学习他们。）

我们提供一个 [linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks) 去自动强制这些规则。我们理解这些规则可能看起来限制或者让人困惑，但是但是他们对于创建 Hooks 非常有用。

> 详细解释
> 你可以在独立的页面学习更多关于这些规则：Rule Of Hooks。

### 💡 Building Your Own Hooks

有时候，我们想要在组件中去重用一些有状态的逻辑。通常，有两种解决这个问题的解决方案：higher-order components 和 render props。自定义 Hooks 让你做到这个，但是没有添加更多组件到你的树中。

在这个页面的前面，我们介绍了一个 `FriendStatus`组件，调用`userState`和`userEffect` Hooks 去订阅一个朋友的在线状态。假设我们也想要重用这个订阅逻辑到其他组件。

首先，我们提取这个逻辑到一个自定义的 Hook，叫做 useFriendStatus：
```jsx harmony
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```
它接收一个`friendId`作为参数，并返回我们的朋友是否在线。

现在我们可以在多个组件中使用它：
```jsx harmony
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}

```
```jsx harmony
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```
这些组件的状态是完全独立的。Hooks 是重用状态逻辑的方式，不是状态本身。实际上，每一个 Hook 调用有一个完全独立的状态 -- 所以你可以在组件中多次使用自定义 Hook 。

自定义 Hooks 更像是一个约定，而不是一个功能。如果一个函数的名字使用"use"开头，并且它调用其他 Hooks，我们称它为自定义 Hook。`useSomething`命名约定让我们的 linter 插件可以在使用 Hooks 代码中找到 bug。

你可以编写自定义 Hook，覆盖大范围的场景，像表单处理，动画，声明订阅，定时器，和可能很多我们没有考虑到的。我们很高兴可以看到 React 社区可以推出什么自定义 Hooks。

> 详细解释
> 你可以在独立的页面学习到更多关于自定义 Hooks：Building Your Own Hooks。

### 🔌 Other Hooks

还有一些不常用的内置钩子可能非常有用。比如，[useContext](https://reactjs.org/docs/hooks-reference.html#usecontext)让你订阅 React 上下文，不需要引入嵌套：

```jsx harmony
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

[useReducer](https://reactjs.org/docs/hooks-reference.html#usereducer)让你使用一个 reducer 管理复杂对象的本地状态：
```jsx harmony
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...
```

> 详细解释
> 你可以在独立页面学习到更多关于内置 Hooks：[Hooks API Reference](https://reactjs.org/docs/hooks-reference.html)。

### 下一步

哦，真快！如果没有太大的意义或者你想要更详细的学习，你可以阅读下一个页面，从 [State Hooks](https://reactjs.org/docs/hooks-state.html) 文档开始。

你也可以查阅 [Hooks API 索引](https://reactjs.org/docs/hooks-reference.html) 和 [Hooks FAQ](https://reactjs.org/docs/hooks-faq.html)。

最后，不要错过[介绍页面](https://reactjs.org/docs/hooks-intro.html)，它解释为什么我们添加 Hooks 并且我们怎样和类一起开始用他们 -- 不重写我们的应用。
