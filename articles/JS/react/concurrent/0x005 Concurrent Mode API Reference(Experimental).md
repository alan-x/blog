### Adopting Concurrent Mode(Experimental)

注意：

这个页面描述了**稳定发布中[还不可用](https://reactjs.org/docs/concurrent-mode-adoption.html)的实验性功能**。不要在生产应用中依赖 React 的试验性构建。在成为 React 的一部分之前，这些功能可能发生非常大的改变，并且没有警告。

这个文档面向早期接受者和好奇的人。**如果你刚接触 React，不要担心这些功能** -- 你不需要立马学习他们。

这个页面是 React [Concurrent 模式](https://reactjs.org/docs/concurrent-mode-intro.html) 的 API 索引。如果你在需要一个指南，查阅[Concurrent UI 模式](https://reactjs.org/docs/concurrent-mode-patterns.html)

**注意：这是一个社区预览，不是最终稳定版。这些 API 将会有更多改变。使用风险自负！**

### 启用 Concurrent 模式

#### creatRoot
```jsx harmony
ReactDOM.createRoot(rootNode).render(<App />);
```
替换`ReactDOm.render(<App />, rootNode)`并启用 Concurrent 模式。

了解更多关于 Concurrent 模式，查阅[Concurrent 模式文档](https://reactjs.org/docs/concurrent-mode-intro.html)。

#### createBlockingRoot
```jsx harmony
ReactDOM.createBlockingRoot(rootNode).render(<App />)
```
替换`ReactDom.render(<App />, rootNode)`并启用[Blocking 模式](https://reactjs.org/docs/concurrent-mode-adoption.html#migration-step-blocking-mode)。

选择 Concurrent 模式导致 React 工作原理的语义化改变。这意味着你不能只在部分组件使用 Concurrent 模式。因为这个，有些应用可能无法直接升级到 Concurrent 模式。

Blocking 模式只包含一个小的 Concurrent 功能子集，并且只是作为无法直接升级的应用中间升级的步骤。

### Suspense API

#### Suspense
```jsx harmony
<Suspense fallback={<h1>Loading...</h1>}>
  <ProfilePhoto />
  <ProfileDetails />
</Suspense>
```
`Suspense`让你的组件"等待"一些东西，在他们可以被渲染之前，当等待的时候显示一个 fallback。

在这个例子，`ProfileDetails`等待一个异步 API 调用去获取一些数据。当我们等待`ProfileDetails`和`ProfilePhoto`，我们将显示`Loading` fallback 替代。有一点很重要需要注意，直到`<Suspense>`所有的子组件被加载，我们将持续显示 fallback。

`Suspense`接收两个属性：

- fallback 接收一个加载指示器，fallback 会显示直到`Suspense`组件的子组件全部完成渲染。

- unstable_avoidThisFallback 接收一个布尔值。它告诉 React 是否"跳过"获取这个边界，在初始化加载的时候。这个 API 将可能在未来发行中移除。

### <SuspenseList>
```jsx harmony
<SuspenseList revealOrder="forwards">
  <Suspense fallback={'Loading...'}>
    <ProfilePicture id={1} />
  </Suspense>
  <Suspense fallback={'Loading...'}>
    <ProfilePicture id={2} />
  </Suspense>
  <Suspense fallback={'Loading...'}>
    <ProfilePicture id={3} />
  </Suspense>
  ...
</SuspenseList>
```
`SuspenseList`帮助协调过的组件，可以通过编排顺序挂起这些和用户相关的组件。

当多个组件需要获取数据，这个数据可能以非预期顺序获取。然而，如果你包裹这些项目在一个`SuspenseList`，React 将不显示列表中的任何项，直到前一个项被显示（这个行为是自适应的）。

`SuspenseList`接收两个属性：

- revealOrder(forwards, backwards, together) 定义`SuspenseList`子组件显示的顺序
    - together 显示他们所有，当他们一个一个都准备好
- tail(collapsed, hidden) 只是怎样卸载`SuspenseList`显示的项目
    - 默认，`SuspenseList`将会显示列表中所有的 fallback
    - `collapsed`只显示列表下一个 fallback。
    - `hidden`不限时任何卸载的项目


注意`SuspenseList`只操作它下面最近的`Suspense`和`SuspenseList`组件。它不寻找深度超过一级的边界。然而，它可能去嵌套多个`SuspenseList`组件去构建网格。

#### useTransition
```jsx harmony
const SUSPENSE_CONFIG = { timeoutMs: 2000 };

const [startTransition, isPending] = useTransition(SUSPENSE_CONFIG);
```

`useTransition`通过等待内容去加载，允许组件去避免不期待的加载状态，在**过渡到下一个屏幕**之前。它也允许组件去延迟更久，数据获取更新直到子序列渲染，这样更多关键更新可以立即渲染。

`useTransition` hook 返回一个包含两个值的数组。

- `startTransition`是一个函数，接收一个回调。我们可以使用它去告诉 React 哪一个状态我们要去延迟。

- `isPending`是一个布尔。是 React 告诉我们我们等待的组件过渡完成的方式。

**如果一些状态更新导致一个组件挂起，这个状态更新应该被包裹到一个过渡**。

```jsx harmony
const SUSPENSE_CONFIG = { timeoutMs: 2000 };

function App() {
  const [resource, setResource] = useState(initialResource);
  const [startTransition, isPending] = useTransition(SUSPENSE_CONFIG);
  return (
    <>
      <button
        disabled={isPending}
        onClick={() => {
          startTransition(() => {
            const nextUserId = getNextId(resource.userId);
            setResource(fetchProfileData(nextUserId));
          });
        }}
      >
        Next
      </button>
      {isPending ? " Loading..." : null}
      <Suspense fallback={<Spinner />}>
        <ProfilePage resource={resource} />
      </Suspense>
    </>
  );
}
```
在这个代码，我们包裹我们的数据获取到`startTransition`。这允许我们去马上开始获取 profile 状态，当延迟渲染下一个 profile 页面，并且它关联的`Spinner`两秒（时间展示在`timeoutMs`）。

`isPending`布尔让 React 直到我们的组件在过渡，这样我们可以让用户直到这个，通过显示一些加载文字在上一个 profile 页面。

**了解更多关于过渡的信息，你可以阅读[Concurrent UI 模式]https://reactjs.org/docs/concurrent-mode-patterns.html#transitions)**。

#### useTransition Config
```jsx harmony
const SUSPENSE_CONFIG = { timeoutMs: 2000 };
```
`useTransition`接收一个带`timeoutMs`的**可选 Suspense 配置**。这个超时（毫秒形式）告诉 React 显示下一个状态之前等待多久（前一个例子是新 Profile 页面）。

**注意：我们推荐你在不同模块之前共享 Suspense 配置**。

### useDeferredValue
```jsx harmony
const deferredValue = useDeferredValue(value, { timeoutMs: 2000 });
```
返回一个延迟版本的值，这个值最多延迟`timeoutMS`时间。

这个通常用于保持接口响应，当你有些东西立即渲染，基于用户输入并且需要等待数据获取。

关于这点一个好的例子是文本输入：
```jsx harmony
function App() {
  const [text, setText] = useState("hello");
  const deferredText = useDeferredValue(text, { timeoutMs: 2000 }); 

  return (
    <div className="App">
      {/* Keep passing the current text to the input */}
      <input value={text} onChange={handleChange} />
      ...
      {/* But the list is allowed to "lag behind" when necessary */}
      <MySlowList text={deferredText} />
    </div>
  );
 }
```
这允许我们去开始立即为`input`显示新的文本，让网页感觉更加积极。同时，`MyShowList`在更新前延迟最多两秒，基于`timeoutMs`，允许它在背后渲染当前文本。

#### useDeferredValue 配置
```jsx harmony
const SUSPENSE_CONFIG = { timeoutMs: 2000 };
```
`useDeferredValue`接收一个带 `timeoutMs`**可选的 Suspense 配置**。这个超时（毫秒形式）告诉 React 显示下一个状态之前等待多久（前一个例子是新 Profile 页面）。

React 将允许尝试去使用一个更短的延迟，当网络和设备允许的时候。











