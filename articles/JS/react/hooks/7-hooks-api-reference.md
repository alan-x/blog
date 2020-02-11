[原文地址](https://reactjs.org/docs/hooks-reference.html)
### Hooks API Reference

Hooks 是 React 16.8 引入的新内容。它们允许你不写一个类来使用状态和其他 React 功能。

这个页面描述了 React 内置的 Hooks。

如果你是 Hooks 新手，你可能想要先查阅概述。你也可能在常问问题章节找到有用的信息。

### 基本 Hooks
 
#### useState

```jsx harmony
const [state, setState] = useState(initialState);
```
返回一个有状态的值，和一个更新它的函数。

在初始化渲染期间，返回的状态（`state`）和作为第一个传输的值一样（`initialState`）。

`setState`函数用于更新状态。它接受一个新状态值，并且入队一个组件的重新渲染。
```jsx harmony
setState(newState);
```
在后续的重新渲染中，`useState`返回的第一个值将总会是应用更新后最近的值。

> 注意：React 保证`setState`函数标示是稳定的并且不会在重新渲染的时候改变。这也是为什么它从`useEffect`或`useCallback`依赖列表缺省是安全的。

##### 函数更新
如果新状态是使用前一个状态计算的，你可以传递一个函数到`setState`。函数将会接收前一个值，并返回更新的值。这是一个计数器组件的例子，使用两种形式的`setState`。
```jsx harmony
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```
"+"和"-"按钮使用函数形式，因为更新值基于前一个值。但是"Reset"按钮按钮使用普通形式，因为它总是设置计数到初始值。

> 注意：不像类组件中的`setState`方法，`useState`不会自动合并更新对象。你可以复制这个行为，通过结合函数更新形式和对象扩展语法：
```jsx harmony
setState(prevState => {
  // Object.assign would also work
  return {...prevState, ...updatedValues};
});
```

另一个选项是`useReducer`，更适合用于管理包含多个子值的状态对象。


##### 指定初始值

有两种初始化`useReducer`状态的方式。你可以根据场景选择一种方式。最简单的方式是传递初始状态作为第二参数：
```
  const [state, dispatch] = useReducer(
    reducer,
    {count: initialCount}
  );
```

注意：React 不使用由 Redux 推广的`state = initialState`参数。这个初始化值有时候需要去依赖属性，因此使用 Hook 调用替代。如果你觉得这很强制，你可以调用`useReducer(reducer, undefined, reducer)`去模拟 Redux 的行为，但是这不被鼓励。

##### 惰性初始化状态

`initialState`参数是在初始化渲染期间使用的状态。在后续渲染中，他被抛弃了。入股初始状态是一个昂贵计算的结果，你可能提供一个函数替代，它只会在初始化渲染被执行：
```jsx harmony
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

##### 摆脱状态更新
如果你更新一个 State Hook 到相同的值作为当前状态，React 将会不会渲染子组件或者触发 effect。（React 使用 [Object.is 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description)。）

 注意 React 在摆脱之前依旧需要再次渲染指定组件。这不必担心，因为 React 将不会"深入"到树中。如果你在渲染的时候执行昂贵的计算，你可以使用`useMemo`优化他们。

#### useEffect
```jsx harmony
useEffect(didUpdate);
```
接受一个包含重要的，可能是有作用的代码的函数。

操作，订阅，定时器，日志，和其他副作用不允许在一个函数组件的主体（称为 React 的 render phase）。这么做将会导致令人困惑的 bug，并在 UI 不一致。

作为替代，使用`useEffect`。传递给`useEffect`的函数将会在渲染被提交到屏幕之后执行。将 effect 认为是 React 纯净的函数世界到重要世界的逃生通道。

默认，effect 在每次计算的渲染之后执行，但是你可以选择触发他们[只有当某些值改变的时候]()。

##### 清理 effect

通常，effect 创建资源需要在组件离开屏幕前面清理，比如一个订阅或者定时器 ID。为了做到这个，传递给`useEffect`的函数可能返回一个清理函数，比如，创建一个订阅：
```jsx harmony
useEffect(() => {
  const subscription = props.source.subscribe();
  return () => {
    // Clean up the subscription
    subscription.unsubscribe();
  };
});
```
清理函数在组件从 UI 移除之前执行，为了防止内存泄漏。此外，如果一个组件渲染多次（就像他们通常做的），**前一个 effect 在执行下一个 effect 之前被清理**。在我们的例子中，这意味着新的订阅在每一次更新创建。为了避免在每一次更新触发 effect，查阅下一章节。


##### Timing of effects

不像`componentDidMount`和`componentDidUpdate`，传递给`useEffect`的函数在布局和绘制之后触发，在一个延迟事件。这让它适合很多常见的副作用，比如设置监听器和事件处理器，因为大部分累的工作不应该堵塞浏览器更新屏幕。

然而，不是所有的 effect 可以被延迟。比如，一个用户可见的 DOM 改变必须在下一次绘制的时候同步触发，这样用户不会察觉到视觉上的不一致。（和主动被动事件监听器概念上很像）对于这类的 effect，React 提供一个额外的 Hook 叫做[useLayoutEffect](https://reactjs.org/docs/hooks-reference.html#uselayouteffect)。它和`useEffect`有相同的签名，只有在何时触发上有区别。

尽管`useEffect`推迟到浏览器被绘制，它保证在任何渲染之前触发。React 将总是在开始一个新的更新之前刷新一个前一个渲染的 effect。

##### 按条件触发一个 effect
effect 的默认行为是在每一次完成渲染之后触发 effect。这种方式，一个 effect 总是在它的依赖改变的时候重新创建。

然而，在某些场景中可能太过火，比如前一个章节的订阅例子。我们不需要在每一次更新的时候创建一个新的订阅，只有如果`source`属性变化。

为了实现这个，传递一个第二参数给`useEffect`，这是一个数组值，这个 effect 依赖它。我们更新的例子现在看起来像这样：
```jsx harmony
useEffect(
  () => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  },
  [props.source],
);

```
现在订阅将只会在`props.source`改变的时候创建。

> 注意：如果你使用这个优化，确保数组包含**被 effect 使用的随着时间改变的组件范围的所有值**。否则，你的代码将会引用上一次不新鲜的值。查阅更多关于[怎样处理函数](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)并且当[数组值频繁改变的时候做什么](https://reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)。

> 如果你想要只运行和清理一个 effect 一次（在挂载和卸载），你可以传递一个空数组（`[]`）作为第二个参数。这告诉 React 你的 effect 不依赖 props 和 state 的任何值，所以它不需要去重新执行。这不是一个特殊场景 -- 它直接遵循依赖数组一贯工作方式。

> 如果你传递一个空的数组（`[]`），effect 内部的 props 和 state 将总是他们的初始值。尽管传递`[]`作为第二个参数和`componentDidMount`和`componentWillUnMount`思想模式很像，有[更好的](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)[解决方案](https://reactjs.org/docs/hooks-faq.html#what-can-i-do-if-my-effect-dependencies-change-too-often)去避免频繁重新运行 effect。同时，别忘了 React 延迟执行`useEffect`直到浏览器被绘制，所以做额外的工作不是一个问题。

> 我们推荐使用[exhaustive-deps](https://github.com/facebook/react/issues/14920)规则作为我们[eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation)包的一部分。它在依赖指定错误的时候警告并建议一个修复。

> 依赖的数组不是作为参数传递。概念上，这是他们所标示的：effect 函数内部引用的每一个值应该出现在依赖数组中。在未来，一个足够该机的编译器可以自动创建这个数组。

#### useContext
```jsx harmony
const value = useContext(MyContext);
```
接受一个上下文对象（从`React.createContext`返回的值）并且为上下文返回当前上下文的的值。当前上下文的值取决于树中调用组件前面的最近的的`<MyContext.Provider>`的`value`属性。

当组件前面最近的`<MyContext.Provider>`更新，Hook 将会使用传递给`MyContext`最新的上下文`value`触发一个渲染。就算一个组件使用[React.memo](https://reactjs.org/docs/react-api.html#reactmemo)或者[shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)，一个渲染依旧发生在组件使用`useContext`。

不要忘记传递给`useContext`的参数必须是上下文对象本身：
- **正确**：`useContext(MyContext)`
- **错误**：`useContext(MyContext.Consumer)`
- **错误**：`useContext(MyContext.Provider)`

一个组件调用`useContext`将会总是重新渲染，当上下文值改变。如果重新渲染组件是昂贵的，你可以[通过使用记忆优化它](https://github.com/facebook/react/issues/15156#issuecomment-474590693)。

> 提示：
> 如果在 Hooks 之前对 context API 很熟悉，`useContext(MyContext)`和类中`static contextTyoe = MyContext`相同，或者`<MyContext.Consumer>`。

`useContext(MyContext)`只让你读取 context，并订阅它的改变。你依旧需要一个`<MyContext.Provider>`在树的前面为这个 context 提供值。

**将它和 Context.Provider 放在一起**
```jsx harmony
const themes = {
  light: {
    foreground: "#000000",
    background: "#eeeeee"
  },
  dark: {
    foreground: "#ffffff",
    background: "#222222"
  }
};

const ThemeContext = React.createContext(themes.light);

function App() {
  return (
    <ThemeContext.Provider value={themes.dark}>
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);

  return (
    <button style={{ background: theme.background, color: theme.foreground }}>
      I am styled by theme context!
    </button>
  );
}
```
这个例子是为了 hooks 修改，从在[Context Advanced Guide](https://reactjs.org/docs/context.html)的例子，你可以在那里找到更多关于何时和怎样使用 Context 的信息。

### 额外 Hooks

下面的 Hooks 是前面章节的基本 Hooks 的变体。只在特定边缘场景需要。不要强制预先学习。

#### useReducer
```jsx harmony
const [state, dispatch] = useReducer(reducer, initialArg, init);
```
`useState`的替代。接受一个类型为`(state, action) => newState`的 reducer，并返回当前值和一个`dispatch`方法的对。（如果你对 Redux 很熟悉，你已经知道它是如何工作的了）。

`useReducer`比`useState`更适合，当你有复杂状态逻辑调用多个子值或者当下一个状态依赖前一个的时候。`useReducer`也让你为触发深层更新的组件优化性能，因为[你可以向下传递 dispatch 替代一个 callback](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)。

这是[useState](https://reactjs.org/docs/hooks-reference.html#usestate)章节的计数器例子，使用 reducer 重写：
```jsx harmony
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

> 注意：React 保证`dispatch`函数标示是稳定的，并且不会在重新渲染的时候改变。这是为什么它从`useEffect`或者`useCallback`依赖列表缺省是安全的。

##### 惰性初始化

你也可以惰性创建初始化。为了做到这个，你可以传递一个`init`函数作为第三个参数。初始化状态将会设置为`init(initialArg)`。

它让你抽取计算初始状态的逻辑到 reducer 之外。这当然也方便之后在动作的响应中重置状态。
```jsx harmony
function init(initialCount) {
  return {count: initialCount};
}

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    case 'reset':
      return init(action.payload);
    default:
      throw new Error();
  }
}

function Counter({initialCount}) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  return (
    <>
      Count: {state.count}
      <button
        onClick={() => dispatch({type: 'reset', payload: initialCount})}>

        Reset
      </button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
    </>
  );
}
```

##### 摆脱 dispatch

如果你从一个 Reducer Hook 返回相同的值作为当前状态，React 将摆脱，不会重新渲染子孙或者触发 effect。（React 使用[Object.is 比较算法](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description)）。

注意 React 可能依旧需要再次渲染指定组件，在摆脱之前。这不需要关心，因为 React 不需要去"深入"树。如果你在渲染的时候执行昂贵的计算，你可以使用`useMemo`优化他们。

#### useCallback
```jsx harmony
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```
返回一个[记忆的](https://en.wikipedia.org/wiki/Memoization)回调。

传递一个行内回调和一个依赖数组。`useCallback`将返回一个记忆的回调版本，它只有在其中一个依赖改变的时候改变。每当传递回调去优化依赖引用相同阻止不需要的渲染的子组件（比如，`shouldComponentUpdate`）的时候很有用。

`useCallback(fn, deps)`和`useMemo(() => fn, deps)`相同。

> 依赖的数组不是作为参数传递给 callback。概念上，这是他们所标示的：callback 函数内部引用的每一个值应该出现在依赖数组中。在未来，一个足够高级的编译器可以自动创建这个数组。

> 我们推荐使用[exhaustive-deps](https://github.com/facebook/react/issues/14920)规则作为我们[eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation)包的一部分。它在依赖指定错误的时候警告并建议一个修复。

#### useMemo
```jsx harmony
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

返回一个[记忆的](https://en.wikipedia.org/wiki/Memoization)值。

传递一个"创建"函数和一个数组依赖。`useMemo`将只重新计算记忆的值，当其中一个依赖改变的时候。这个优化帮助在每次渲染避免昂贵的计算。

记住传递给`useMemo`的函数在每次渲染都执行。不要做任何你在渲染通常不会做的事情。比如，副作用属于`useEffect`，不是`useMemo`。

如果没有提供数组，一个新的值将会在每次渲染计算。

**你可能依赖 useMemo 作为性能优化，不是一个语义保证**。在未来，React 可能选择"忘记"一些之前记忆的值并在下一次渲染重新计算他们，比如，为屏幕之外的组件释放内存。编写你的代码就算没有`useMemo`也能工作 -- 然后添加它作为性能优化。

> 依赖的数组不是作为参数传递给函数。概念上，这是他们所标示的：callback 函数内部引用的每一个值应该出现在依赖数组中。在未来，一个足够该机的编译器可以自动创建这个数组。
 
> 我们推荐使用[exhaustive-deps](https://github.com/facebook/react/issues/14920)规则作为我们[eslint-plugin-react-hooks](https://www.npmjs.com/package/eslint-plugin-react-hooks#installation)包的一部分。它在依赖指定错误的时候警告并建议一个修复。

#### useRef
```jsx harmony
const refContainer = useRef(initialValue);
```
`useRef`返回一个可变的对象，它的`.current`属性初始化为传递的参数（`initialValue`）。返回的对象将会在整个组件的生命周期持久化。

一个常见场景是命令式访问一个子组件：
```jsx harmony
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```
基本上，`useRef`类似一个"box"，可以保存一个可变值在它的`.current`属性。

你可能对 ref [访问 DOM](https://reactjs.org/docs/refs-and-the-dom.html) 的方式很熟悉。如果你使用`<div ref={myRef} />`传递一个 ref 对象到 React，React 将设置`.current`属性到对应的 DOM 节点，无论何时节点发生改变。


然而，`useRef()`比`ref`属性还有用。它是[保存可变值的技巧](https://reactjs.org/docs/hooks-faq.html#is-there-something-like-instance-variables)，类似你在类中使用的实例域。

这起作用是因为`useRef()`创建一个普通 JavaScript 对象。`useRef()`和你自己创建一个`{current: ...}`对象的不同在于`useRef`在每次渲染都给你相同的 ref 对象。

记住，`useRef`不会在内容改变的时候提示你。操作`.current`属性不会触发一个重新渲染。如果你想要执行代码，当 React 绑定或者解绑一个 ref 到 DOM 节点，你可能想要使用[callback ref](https://reactjs.org/docs/hooks-faq.html#how-can-i-measure-a-dom-node)替代。

#### useImperativeHandle
```jsx harmony
useImperativeHandle(ref, createHandle, [deps])
```
`useImperativeHandle`自定义使用`ref`暴露给父组件的实例值。一如往常，使用 ref 的命令式代码应该在大部分场景避免。`useImperativeHandle`应该和`forwardRef`一起使用。
```jsx harmony
function FancyInput(props, ref) {
  const inputRef = useRef();
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
}
FancyInput = forwardRef(FancyInput);
```

在这个例子，渲染`<FancyInput ref={inputRef} />`的父组件可以调用`inputRef.current.focus()`


#### useLayoutEffect

这个签名和`useEffect`相同，但是它在 DOM 变化之后同步触发。使用这个从 DOM 读取布局，并且同步重新渲染。`useLayoutEffect`内部嗲用的更新将会同步刷新，在浏览器有机会绘制之前。

尽可能选择标准的`useEffect`避免堵塞可视化更新。

> 提示：如果你从类组件升级代码，注意`useLayoutEffect`与`componentDidMount`和`componentDidUpdate`触发的阶段相同。然而，**我们推荐首先使用 useEffect** 并且只尝试`useLayoutEffect`如果它导致一个问题。

> 如果你使用服务端渲染，记住`useLayoutEffect`和`useEffect`直到 JavaScript 下载完成才会运行。这就是为什么当服务端渲染组件包含`useLayoutEffect`的时候 React 警告。为了修复这个，移动逻辑到`useEffect`(如果它对第一次渲染是必须的)，或延迟显示组件，直到客户端渲染（如果 HTML 看起来是破损的，直到`useLayoutEffect`运行）。

> 为了从服务端渲染的 HTML 排除一个需要布局作用的组件，使用`showChild && <Child />`条件话渲染组件并且使用`useEffect(() => { setShowChild(true); })`延迟显示它。这种方式，UI 在 hydration 之前不会出现破损。

#### useDebugValue
`useDebugValue`可以用于在 React DevTools 为自定义 Hooks 显示一个标签。

比如，描述在"[创建你自己的 Hooks](https://reactjs.org/docs/hooks-custom.html)"的`useFriendStatus`自定义 Hook。
```jsx harmony
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  // ...

  // Show a label in DevTools next to this Hook
  // e.g. "FriendStatus: Online"
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```
> 提示：我们不推荐为每一个自定义 Hook 添加调试值。它对于属于共享的库的一部分的自定义 Hook 很有价值。

##### 延迟格式化调试值
在某些场景，为了显示格式化一个值可能是一个昂贵操作。他不是必须的，除非一个 Hook 真的被检查。

为了这个原因，`useDebugValue`接受一个格式化函数作为一个可选的第二参数。这个函数只有在如果 Hooks 被检查的时候才调用。它接受调试值作为参数，并且应该返回一个格式化的显示值。

比如，一个返回`Date`值的自定义 Hook 应该避免不必要的调用`toDateString`函数，通过传递下面的格式化：
```jsx harmony
useDebugValue(date, date => date.toDateString());
```
