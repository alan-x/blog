[原文地址](https://reactjs.org/docs/hooks-effect.html)
### Using the Effect Hook

Hooks 是 React 16.8 引入的新内容。它们允许你不写一个类来使用状态和其他 React 功能。

Effect Hook 让你在函数组件内执行副作用：

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
这个片段基于上一个页面的计数器例子，但是我们为它添加了新的功能：我们设置文档的标题为自定义信息，包含点击的次数。

数据获取，设置一个监听器，和手动改变 React 组件 DOM 都是副作用的例子。不管你是否叫这些操作为"副作用"（或者只是"作用"），你都可能曾在你的组件执行过他们。

> 提示：如果你对 React 的生命周期方法很熟悉，你可以把`useEffect`认为是`componentDidMount`，`componentDidUpdate`，和`componentWillUnmount`结合。

在 React 组件中，有两种常见类型的副作用：一种需要清理，一种不需要。让我们详细看看区别。

### 不需要清理的作用

有时候，我们想要去**在 React 更新 DOM 之后运行额外的代码**。网络请求，手动 DOM 操作，和记录都是不需要清理的副作用的普通例子。我们说因为我们可以执行他们并且马上忘记他们。让我们比较一下类和 Hooks 表示副作用。

### 使用类的例子

在 React 类组件，`render`方法它不应该导致副作用。它可能太早 -- 我们通常想要执行我们的作用在 React 更新 DOM 之后。

这也是为什么在 React 类中，我们将副作用放到`componentDidMount`和`componentDidUpdate`。回到我们的例子，这是一个 React 计数器类组件，在 React 改变 DOM 之后更新 DOM：
```jsx harmony
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
注意我们如何在这个类的两个生命周期方法中重复代码。

这是因为在很多场景中我们要去执行相同的副作用，无视组件是否刚挂载，或者已经被更新。概念上，我们想要它在每一次渲染之后发生 -- 但是 React 类组件没有这类的方法。我们可以提取一个分离的方法但是我们依旧要在两个地方调用。

现在来看看我们怎样使用 useEffect Hook 做到相同的事儿。

### 使用 Hooks 的例子

我们已经在这个页面的顶部看过这个例子，但是来近距离看看：
```jsx harmony
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
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

**useEffect 做了啥子？** 通过使用这个 Hook，你告诉 React 你的组件需要需要在渲染之后做一些事儿。React 将会记住你传递的函数（我们称之为"effect"），并且在执行 DOM 更新之后调用它。在这个作用中，我们设置文档标题，但是我们也可以执行数据获取或者调用一些其他紧急 API。

**为什么在组件内调用 useEffect？** 放置`useEffect`在组件内部让我们访问`count`状态变量（或者其他属性）。我们不需要特别的 API 去读取它 -- 它已经在函数作用域。Hooks 拥抱 JavaScript 闭包并避免引入 JavaScript 已经提供的解决方案 React-指定 API。

**useEffect 在每次渲染之后都执行吗？** 是的！默认，它在第一次渲染和每一次更新都执行。（我们将会在之后讨论怎样自定义这个）预期思考术语"挂载"和"更新"，你可能更容易去思考作用发生在"渲染之后"。React 保证 DOM 更新之后执行作用。

### 详细解释

现在我们更加了解作用了，这些行应该有意义：
```jsx harmony
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
}
```
我们声明了`count`状态变量，然后我们告诉 React 我们需要使用 effect。我们传递一个函数给`useEffect`Hook。我们传递的这个函数就是我们的 effect。在我们的 effect 中，我们使用`document.title`浏览器 API 设置文档标题。我们可以在 effect 内部读取最新的`count`，因为它在我们函数的作用域内。当 React 渲染我们的组件，它将会记住我们使用的 effect，然后在更新 DOM 之后执行我们的 effect。这发生于每一次渲染，包括第一次。

有经验的 JavaScript 开发者可能注意到，传输给`useEffect`的函数每一次渲染都不同。这是故意的，实际上，这让我们从 effect 内部读取`count`不需要担心它过期的原因。每一次我们重新渲染，我们调度一个不同的 effect，替代前一个。某种程度上，这让 effect 表现的更像渲染结果的一部分 -- 每一个 effect"属于"一个特定的渲染。我们将会在这页面的后面更清楚的了解为什么这很有用。

> 提示：和`componentDidMount`或者`componentDidUpdate`不像，`useEffect`调度 effect 不会堵塞浏览器更新屏幕。这让你的应用感觉更灵敏。大部分的 effect 不需要同步发生。在不普通的场景中他们这么做（比如测量布局），有一个分离的和`useEffect` API
 相同的`useLayoutEffect` Hook。
 
### 需要清理的 effect

前面，我们看过怎样表示不需要清理的副作用。然而，一些 effect 需要。比如，我们可能想要设置一个订阅到一些外部的数据资源。在这种场景中，清理很重要，这样才不会导致内存泄漏！来比较一下我们如何使用类和 Hooks 来完成。

### 使用类的例子
在 React 类中，你通常在`componentDidMount`设置监听，在`componentWillUnmount`中清理。比如，我们有一个`ChatAPI`模块让我们订阅朋友的在线状态。这是我们可能如何在类中订阅和像是状态：
```jsx harmony
class FriendStatus extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }

  render() {
    if (this.state.isOnline === null) {
      return 'Loading...';
    }
    return this.state.isOnline ? 'Online' : 'Offline';
  }
}
```
注意`componentDidMount`和`componentWillUnmount`需要互为镜像。生命周期方法强制我们分离这个逻辑，尽管概念上这两者的代码和同一个副作用相关。

> 注意：有鹰眼的读者可能注意到这个例子也需要一个`componentDidUpdate`方法才完全正确。我们现在先忽略这个，但是这个页面的后续章节将会再回顾。

### 使用 Hooks 的例子

看看我们怎样使用 Hooks 写这个组件。

你可能觉得我们需要一个分离的 effect 去执行清理。但是添加或者移除一个订阅关联的如此紧密，`useEffect`设计为让他们在一次。如果你的 effect 返回一个函数，React 将会在需要清理的时候执行它。

```jsx harmony
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

**为什么我们从 effect 返回一个函数？** 这是 effect 可选的清理机制。每一个 effect 可能返回一个函数在之后清理它。这让我们保持添加或者移除监听的逻辑更接近。他们是同一个 effect 的一部分。

**React 具体在啥时候清理 effect？** React 在组件卸载的时候执行清理。然而，就像我们前面学习的，effect 在每一次渲染都执行，而不是只有一次。这是为什么 React 总是在上一次渲染，限一次执行 effect 之前清理 effect。我们将会在后面讨论为神恶魔这帮助米面 bug 和如何选择性推出这个行为防止它导致性能问题。

> 注意：我们不需要从 effect 返回具名函数。这里我们叫他`cleanup`是为了表明它的目的，但是你可以返回一个箭头函数或者叫做其他不同的。

### 回顾

我们已经学习到`useEffect`让我们在组件渲染之后表达不同类型的副作用。一些 effect 可能需要清理，所以他们返回一个函数：
```jsx harmony
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
```
其他副作用可能没有一个清理阶段，并且不返回任何东西。
```jsx harmony
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```

Effect Hook 使用一个单独的 API 统一两个场景。

**如果你对 Effect Hook 如何工作有点把握，或者如果你感觉不知所措，你可以现在跳到下一个关于 Hooks 的规则页面。**

### 使用 effect 的提示

我们将在这个页面继续深入`useEffect`的一些方面，有经验的 React 用户将可能很好奇。不要觉得有责任去挖掘他们。你总是可以回到这个页面学习 Effect Hook 的更多细节。

### 提示：使用多个副作用去分离关注点

我们在 Hooks 的动机中标示出的一个问题是类生命周期总是包含不想管的额逻辑，但是相关的逻辑被分离到几个方法。这是一个包含前面例子的计数器的组件，和朋友状态指示器逻辑。

```jsx harmony
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
```

注意设置`document.title`的逻辑被分离在`componentDidMount`和`componentDidUpdate`。订阅逻辑也扩散在`componentDidMount`和`componentWillUnmount`。并且`componentDidMount`包含两个任务。

所以，Hooks如何解决这个问题？就像你可以多次使用 State Hook，你可以使用多个 effect，这让我们分离不相关的逻辑到不同的副作用。
```jsx harmony
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

**Hooks 让我们基于他们所做分离代码**，而不是生命周期名。React 将会以他们被指定的顺序，应用这个组件的每一个副作用。

### 解释：为什么 effect 在每一次更新都执行

如果你使用过类，你可能想为什么 effect 清理步骤在每一次重新渲染之后都发生，而不是在卸载的时候一次。看看一个实际的例子，为什么这个设计帮助我们创建 bug 更少的组件。

这个页面的前面，我们引入一个例子`FriendStatus`组件显示一个朋友是否在线。我们的类从`this.props`读取`friend.id`，在组件挂载之后订阅一个朋友状态，在卸载的是欧取消订阅。

```jsx harmony
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```
但是组件在屏幕上的时候，如果`friend 属性发生变化会发生什么`？我们的组件将会继续显示一个不同朋友的在线状态。这是一个 bug。我们也导致一个内存泄漏或者奔溃，在卸载的时候使用错误的调用取消订阅。

在一个类组件，我们需要添加`componentDidUpdate`去处理这个场景：
```jsx harmony
  componentDidMount() {
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate(prevProps) {
    // Unsubscribe from the previous friend.id
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // Subscribe to the next friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
```
忘记去处理`componentDidUpdate`属性是一个 React 应用常见的 bug 来源。

现在考虑这个组件使用 Hooks 的版本：
```jsx harmony
function FriendStatus(props) {
  // ...
  useEffect(() => {
    // ...
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
``` 
它没有遇到这个 bug。（但是我们也没有做任何改变。）

没有特殊的代码去处理更新，因为`useEffect`默认处理他们。它在应用下一个 effect 之前清理前一个 effect。为了指出这个，这个组件会随着时间产生一系列的订阅和取消订阅调用。
```jsx harmony
// Mount with { friend: { id: 100 } } props
ChatAPI.subscribeToFriendStatus(100, handleStatusChange);     // Run first effect

// Update with { friend: { id: 200 } } props
ChatAPI.unsubscribeFromFriendStatus(100, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(200, handleStatusChange);     // Run next effect

// Update with { friend: { id: 300 } } props
ChatAPI.unsubscribeFromFriendStatus(200, handleStatusChange); // Clean up previous effect
ChatAPI.subscribeToFriendStatus(300, handleStatusChange);     // Run next effect

// Unmount
ChatAPI.unsubscribeFromFriendStatus(300, handleStatusChange); // Clean up last effect
```

这个行为确保默认一致性并防止类中因为缺少更新逻辑导致的常见的 bug。


### 提示：通过跳过 effect 优化性能

在一些场景，在每一次渲染清理或者应用 effect 可能导致性能问题。在类组件，我们可以解决这个问题，通过在`componentDidUpdate`内编写额外的`preProps`和`prevState`的比较。

```jsx harmony
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```
这个需求是足够常见的，被内置到`useEffect` Hook API。你可以告诉 React 去跳过应用一个 effect，如果确认值在重新渲染期间没有改变。为了做到它，传递一个数组作为可选的第二参数给`useEffect`。
```jsx harmony
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```
 在前面的例子中，我们传递`[ count ]`作为第二参数。这意味着啥？如果`count`是`5`，并且我们的组件重新渲染的时候`count`依旧是`5`，React 将会比较前一次渲染的`[5]`和下一次渲染的`[5]`。因为数组中所有的项都是相同的（`5 === 5`），React 将会跳过 effect。这是我们的优化。
 
 当我们渲染的时候`count`更新为`6`，React 将会比较前一次渲染`[5]`数组中的项和下一次渲染`[6]`中的项。如果有多个项在数组中，React 将会重新执行 effect，就算他们只有一项不同。
 
 这对于有清理阶段的作用也有效：
 ```jsx harmony
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
}, [props.friend.id]); // Only re-subscribe if props.friend.id changes
```
在未来，第二个参数可能通过构建时转化被自动添加。

> 注意
> 如果你使用这个优化，确保数组包含组件范围的所有被 effect 使用的随着时间改变值（比如 props 和 state）。否则，你的代码将会从前一次渲染引用不新鲜的值。了解更多关于怎样[处理函数](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)和[当数组改变太频繁的时候怎么做](https://reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)。

> 如果你想要执行和清理一个 effect 只有一次（在挂载和卸载），你可以传递一个空的数组(`[]`)作为第二个参数。这告诉 React 你的 effect 不依赖任何 props 和 state 中的值，所以它不需要重新执行。这不是一个特例 -- 它遵循依赖的数组的工作原理。

> 如果你传递一个空数组（`[]`），effect 内的 props 和 state 将会总是有他们的初始值。当传递`[]`作为第二个参数和`componentDidMount`和`componentDidUmmount` 模型很像，通常有[更好](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)的[解决方案](https://reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)去避免重新运行 effect 太频繁。同样，不要忘记 React 延迟运行`useEffect`，直到浏览器被绘制，所以额外工作不是啥问题。

> 我们推荐使用 [exhaustive-deps](https://github.com/facebook/react/issues/14920) 规则作为我们 [eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation) 包的一部分。它在依赖指定错误的时候警告并提示修复。


### 下一步

祝贺！这是一个长页面，但是希望在结束的时候，你所有关于 effect 的问题都被回答。你已经学了 State Hook 和 Effect Hook，你可以结合他们使用做很多事情。他们覆盖类的大部分用力 -- 如果他们没有，你可能找到有帮助的[额外的 Hooks](https://reactjs.org/docs/hooks-reference.html)。

我们也开始看到 Hooks 如何解决我们[动机中](https://reactjs.org/docs/hooks-intro.html#motivation)标示出的问题。我们看到 effect 清理如何避免`componentDidUpdate`和`componentWillUnmount`中的重复，让相关代码更接近，帮助我们避免 bug。我们也看到我们如何通过他们的目的分离 effect，这是我们在类中无法做到的。

在这个时候，你可能疑惑 Hooks 是如何工作的。React 是怎么知道哪一个`useState`调用代表哪一个状态变量，每一次重新渲染？React 在每一次更新怎么"匹配"前一个和下一个 effect？**在下一个页面，我们将会学校以关于 Hooks 的规则 -- 他们是 Hooks 工作的关键。**
