### Building Your Own Hooks

Hooks 是 React 16.8 引入的新内容。它们允许你不写一个类来使用状态和其他 React 功能。

创建你自己的 Hooks 让你提取组件逻辑到可重用的函数。

当我们学习关于使用 Effect Hook，我们看到这个从聊天应用来的组件显示一个信息，只是一个朋友是否在线或者离线：
```jsx harmony
import React, { useState, useEffect } from 'react';

function FriendStatus(props) {
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

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```
现在，假设我们的聊天应用也有一个联系人列表，我们想要使用绿色渲染在线用户的名字。我们可以扶着和张贴同样的逻辑到我们的`FriendListItem`组件，但是它不理想：
```jsx harmony
import React, { useState, useEffect } from 'react';

function FriendListItem(props) {
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

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```
相反，我们希望在`FriendStatus`和`FrientListItem`分享这个逻辑。

通常在 React 中，我们有两个流行的方式在组件之间去分享状态逻辑：render props 和 higher-order components。我们看看 Hooks 如何解决相同的问题，但是不需要强制你去添加更多的组件到树中。

### 抽取一个自定义 Hook

当我们想要在 JavaScript 函数之间分享逻辑，我们抽取到第三个函数。组件和 Hooks 都是函数，所以这丢他们有有用。

**一个自定义 Hook 是一个 JavaScript 函数，它的名字以"use"开头并且可能调用其他 Hooks。**比如，下面的`useFriendStatus`是我们的第一个自定义 Hook。
```jsx harmony
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

前面没有新的东西在里面 -- 逻辑是从前面的组件复制的。就像在一个续建内，确保只有非条件话的在你的自定义 Hook 调用其他的 Hook。

不像一个 React 组件，一个自定义 Hook 不需要去有一个前面特殊的前面，我们可以决定它采取什么参数，和什么，如果有，它应该返回。换句话说，它就先给一个普通的函数。它的名字应该总是使用`use`开头，这样你可以一眼看出应用的 Hook 的规则。

我们`useFriendStatus` Hook 的目标是去订阅我们朋友的状态。这是为什么它需要`friedndID`作为一个参数，并且返回这个朋友是否在线：
```jsx harmony
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  return isOnline;
}
```
现在来看看我们可以怎么使用自定义 Hook。

### 使用自定义 Hook

在开始的时候，我们状态的目标是从`FriendStatus`和`FriendListItem`组件移除重复的逻辑。他们都想要知道一个朋友是否在线。

现在我们提取这个逻辑到一个`useFriendStatus` hook，我们可以这么用：
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

**这个代码和原始的例子一样吗？** 是的，它以相同的方式工作。如果你更近一点看，你会主要到我们没有改变任何行为。我们做的只是将相同的普通代码从两个函数抽取到一个分离的函数。**自定义 Hook 是一个约定，自然遵循了 Hooks 的设计，而不是一个 React 功能。**

**我必须把我的自定义 Hooks 以"use"开头命名吗？** 请这么做。这个约定是非常重要的。没有他，我们无法自动检查违背 Hooks 规则，因为我们无法告诉如果一个确认的函数内部包含 Hooks 调用。

**两个组件使用相同 Hook 分享状态吗？** 不。自定义 Hooks 是重用状态逻辑的机制（比如设置一个定过并记住当前值），但是每次你使用一个自定义 Hook，所有内部的的状态和 effect 都是完全独立的。

**自定义 Hook 是如何得到独立的状态的？** 每次调用 Hook 都得到独立的状态。因为我们直接调用`useFriendStatus`，从 React 的视角来看，我们的组件只调用`useState`和`useEffect`。就像我们之前学到的，我们可以在一个组件中调用`useState`和`useEffect`很多次，并且他们将完全独立。

### 提示：在 Hooks 之间传递信息

因为 Hooks 是函数，我们可以在他们之间传递信息。

为了描述这个，我们将使用从我们假设的聊天李忠中的其他组件。这是一个聊天消息接收选择器，显示当前选择的嗯有是否在线：
```jsx harmony
const friendList = [
  { id: 1, name: 'Phoebe' },
  { id: 2, name: 'Rachel' },
  { id: 3, name: 'Ross' },
];

function ChatRecipientPicker() {
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);

  return (
    <>
      <Circle color={isRecipientOnline ? 'green' : 'red'} />
      <select
        value={recipientID}
        onChange={e => setRecipientID(Number(e.target.value))}
      >
        {friendList.map(friend => (
          <option key={friend.id} value={friend.id}>
            {friend.name}
          </option>
        ))}
      </select>
    </>
  );
}
```
我们保存当前选择的好友 ID 在`recipientID`状态变量，并且在用户从`<select>`选择器选择一个不同朋友的时候更新。

因为`useState` Hook 调用给我们最新的`recipientID`状态变量，我们可以传递它到我们自定义的`useFriendStatus`Hook 作为参数：
```jsx harmony
  const [recipientID, setRecipientID] = useState(1);
  const isRecipientOnline = useFriendStatus(recipientID);
```
这让我们知道当前选中的朋友是否在线。如果我们选择一个不同的朋友，并更新`recipientID`状态变量，我们的`useFriendStatus` Hook 将会从前一个选择的朋友取消订阅，并订阅新选择的一个的状态。

### useYourImagination()

自定义 Hooks 提供之前 React 组件不可能的分享逻辑的弹性。你可以编写自定义 Hook，覆盖广泛的用户场景，类似表单处理，动画，声明式订阅，定时器，和可能很多我们没有考虑到的。甚至，你可以创建 Hooks 就像使用 React 内置功能一样简单。

不要过早添加抽象。现在函数组件可以做的更多，可能函数组件的平均长度会变长。这是很正常的 -- 不要觉得你应该分离他到 Hooks。但是我们也鼓励你去开始关注场景，一个自定义 Hook 可以隐藏复杂的逻辑在一个简单接口，或者帮助解决混乱的额组件。

比如，肯恩给你需要一个浮渣的组件包含很多本地状态，被管理在一个 ad-hoc 更简单，所以你可能更偏向于协程一个 Redux reducer：
```jsx harmony
function todosReducer(state, action) {
  switch (action.type) {
    case 'add':
      return [...state, {
        text: action.text,
        completed: false
      }];
    // ... other actions ...
    default:
      return state;
  }
}
```

Reducer 独立测试非常方便，可以伸缩的表示复杂的更新逻辑。你可以更深的分离他们到更小的 reducer，如果需要。然而，你可能享受使用 React 本地状态的好处，或者可能不想要去安装其他库。

所以，如果我们可以写一个`useReducer` Hook 使用 reducer 让我们管理组件的本地状态？一个简单的版本可能看起来像这样：
```jsx harmony
function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}

```

现在我们可以在我们组件内这么用，让 reducer 分发状态管理：
```jsx harmony
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer, []);

  function handleAddClick(text) {
    dispatch({ type: 'add', text });
  }

  // ...
}

```

在浮渣组件内使用 reducer 管理本地状态是一个足够常见的，我们创建了`useReducer` Hook 到 React。你可以和其他内置 Hooks 在 Hooks API reference 找到。
