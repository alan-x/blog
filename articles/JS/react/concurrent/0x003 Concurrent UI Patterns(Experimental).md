[原文地址](https://reactjs.org/docs/concurrent-mode-patterns.html)
### Concurrent UI Patterns(Experimental)

注意：

这个页面描述了**稳定发布中[还不可用](https://reactjs.org/docs/concurrent-mode-adoption.html)的实验性功能**。不要在生产 app 中依赖 React 的试验性构建。在成为 React 的一部分之前，这些功能可能发生非常大的改变，并且没有警告。

这个文档面向早期接受者和好奇的人。**如果你刚接触 React，不要担心这些功能** -- 你不需要立马学习他们。比如，如果你在寻找现在使用的数据获取指南，阅读[这篇文章](https://www.robinwieruch.de/react-hooks-fetch-data/)。

通常，当我们更新状态，我们期待立即在屏幕上看到改变。这是有意义的，因为我们想要保持我们的应用响应用户输入。然而，有一些场景，我们可能偏向于**延迟一些更新出现在屏幕**。

比如，如果我们切换一个页面到另一个，下一个屏幕的代码或者数据还没有被加载，立即看到一个带加载指示器的空页面是令人沮丧的。我们可能更偏向于呆在前一个应用更长时间。在历史上，React 实现这个模式很难。Concurrent 模式提供一个新的工具集去做到它。

### Transitions

再次访问前一个页面关于[数据获取的 Suspense 的](https://reactjs.org/docs/concurrent-mode-suspense.html)[这个 demo](https://codesandbox.io/s/infallible-feather-xjtbu)。

当我们点击"Next"按钮去切换活跃的 profile，存在的页面数据立马消失，我们再次看到整个页面的加载状态。我们称之为"不期待"的加载状态。**如果我们可以"跳过"它并等待一些内容加载，再过渡到新屏幕之前不是很好嘛？**。

React 提供了一个新的内建的`useTransition()`Hook 去帮助做到这个。

我们可以在三个步骤内使用它。

首先，我们确保我们真的使用 Concurrent 模式。我们将在之后讨论更多关于[使用 Concurrent 模式](https://reactjs.org/docs/concurrent-mode-adoption.html)，但是现在，我们知道我们需要使用`ReactDom.createRoot()`而不是`ReactDOM.render()`让这个功能可用就够了。
```jsx harmony
const rootElement = document.getElementById("root");
// Opt into Concurrent Mode
ReactDOM.createRoot(rootElement).render(<App />);
```
下一步，我们将从 React 添加一个`userTransition`导入：
```jsx harmony
import React, { useState, useTransition, Suspense } from "react";
```
最后，我们将在`App`组件内使用：
```jsx harmony
function App() {
  const [resource, setResource] = useState(initialResource);
  const [startTransition, isPending] = useTransition({
    timeoutMs: 3000
  });
  // ...
```
**代码本身还没有做任何东西**。我们将需要去使用 Hook 的返回值去设置状态过渡。`useTransition`返回两个值：

- `startTransition`是一个函数。我们将用它告诉 React 哪一个状态更新我们想要去延迟。

- `isPending`是一个布尔值。它是 React 告诉我们过渡当前是否正在进行。

我们在下面马上就会使用他们。

注意我们传输一个配置对象给`useTransition`。它的`timeoutMS`属性指定**我们愿意等待多长时间去完成过渡**。通过传递`{timeoutMs: 3000}`，我们说"如果下一个 profile 页面需要超过 3 秒加载，显示大 spinner -- 但是正在超时之前，保持显示前一个屏幕是没问题的"。

### 在一个过渡中包裹 setState

我们的"Next"按钮点击处理器设置状态切换当前 profile：
```jsx harmony
<button
  onClick={() => {
    const nextUserId = getNextId(resource.userId);
    setResource(fetchProfileData(nextUserId));
  }}
>
```
我们将包裹状态更新到`startTransition`。这是我们告诉 React **我们不关心 React 延迟状态更新**如果它导致一个不期待的加载状态：
```jsx harmony
<button
  onClick={() => {
    startTransition(() => {
      const nextUserId = getNextId(resource.userId);
      setResource(fetchProfileData(nextUserId));
    });
  }}
>
``` 
[在 CodeSandbox 尝试](https://codesandbox.io/s/musing-driscoll-6nkie)

点击"Next"一些时间。注意他已经感觉非常不同。**点击的时候不是立即看到一个空的屏幕，我们现在保持前一个屏幕在一段时间内可见**。当数据被加载，React 让我们过渡到新的屏幕。

如果我们让我们的 API 响应超过 5 秒，[我们可以确认](https://codesandbox.io/s/relaxed-greider-suewh)现在 React"放弃"并且无论如何，3 秒后过渡到下一个屏幕。这是因为我们传输`{timeout: 3000}`给`useTransition()`。比如，如果我们传递`{timeout: 6000}`替代，它将会等待整整 1 分钟。

### 添加 Pending 指示器

关于[我们最后的例子](https://codesandbox.io/s/musing-driscoll-6nkie)依旧有一些东西让人感觉断开。的确，看不到"坏的"加载状态很好。**但是看不到完全进度指示感觉更糟糕**！当我们点击"Next"，什么都没有发生让它感觉应用坏了。

我们`useTransition()`调用返回两个值：`startTransition`和`isPending`。

```jsx harmony
  const [startTransition, isPending] = useTransition({ timeoutMs: 3000 });
```

我们已经使用`startTransition`去包裹状态更新。现在我们将使用`isPending`。React 给这个布尔给我们，这样我们可以告诉是否**我们当前等待过渡完成**。我们将使用它指示一些东西发生了：
```jsx harmony
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
    <ProfilePage resource={resource} />
  </>
);
```
[在 CodeSandbox 尝试它](https://codesandbox.io/s/jovial-lalande-26yep)

现在，这感觉好多了！当我们点击下一步，它被禁用，因为多次点击没有意义。并且新的"Loading..."告诉用户应用没有冻结。

### 回顾改变

让我们看看我们从[原始例子](https://codesandbox.io/s/infallible-feather-xjtbu)开始采取的所有改变：
```jsx harmony
function App() {
  const [resource, setResource] = useState(initialResource);
  const [startTransition, isPending] = useTransition({
    timeoutMs: 3000
  });
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
      <ProfilePage resource={resource} />
    </>
  );
}
```
[在 CodeSandbox 尝试](https://codesandbox.io/s/jovial-lalande-26yep)

添加这个过渡只需要添加七行代码：

- 我们导入`useTransition`Hook 并使用它，组件更新状态。

- 我们传递`{timeout: 3000}`去呆在前一个屏幕最多 3 秒。

- 我们包裹我们的状态更新到`startTransition`去告诉 React 延迟它没事。

- 我们使用`isPending`去传递状态更新进度给用户，并禁止按钮。

作为结果，点击"Next"不执行一个立即状态过渡到一个"不期待"加载状态，但是呆在前一个屏幕并传递进度。

### 更新发生在哪里？

这实现并不是很难。然而，如果你开始觉得这怎么可能运行，它可能变得有一点混乱。如果我们设置状态，怎么做到我们没有立即看到结果？下一次`<ProfilePage>`在哪里渲染。

很明显，多"版本"的`<ProfilePage>`同时存在。我们知道旧的存在因为我们在屏幕上看到它，甚至显示一个进度指示器在它之上。并且我们知道新版本也存在在其他地方，因为它是我们等待的！

**但是相同组件的同一个版本如何同时存在？**

这涉及到 Concurrent 模式的根本。[前面说](https://reactjs.org/docs/concurrent-mode-intro.html#intentional-loading-sequences)有点像 React 在"分支"上更新状态。另一方面，我们可以认为包含在`startTransition`的状态更新"在另一个不同的宇宙"渲染，很像科幻电影。我们不会直接"看到"宇宙 -- 但是我们可以从它那里得到一个信号，告诉我们有些事情发生了（`isPending`）。当更新准备好了，我们的"宇宙"合并回去，然后我们在屏幕上看到结果。

多玩一会[demo](https://codesandbox.io/s/jovial-lalande-26yep)，尝试去想象所发生的。

当然，两个版本的树在同一时间渲染是假象，就像你的电脑上所有的程序同时运行也是一个假象。一个操作系统在不同应用之间切换非常快。同样的，React 可以在不同版本的树之间切换，你在屏幕上看到的和它"准备"下一次显示的。

像`useTranstion`之类的 API 让你聚焦于期望的用户体验，不用去思考这机制是怎样实现的。依旧，想象包裹在`startTransition`的更新发生在"一个分支"或者"在另一个宇宙"的概念依旧很有帮助。

### 过渡在任何地方
正如我们在[Suspense 演练](https://reactjs.org/docs/concurrent-mode-suspense.html)学习到的，任何组件可以在任何时候"挂起"，如果一些需要的数据还没准备好。我们可以直接防止`<Suspense>`边界在树的不同地方去处理这个，但是并不一定足够。

让我们回到[第一个 Suspense 例子](https://codesandbox.io/s/frosty-hermann-bztrp)，只有一个 profile 的那个。同时，它只获取一次数据。我们将添加一个"Refresh"按钮去检查服务端更新。

我们的第一次尝试看起来像这样：
```jsx harmony
const initialResource = fetchUserAndPosts();

function ProfilePage() {
  const [resource, setResource] = useState(initialResource);

  function handleRefreshClick() {
    setResource(fetchUserAndPosts());
  }

  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails resource={resource} />
      <button onClick={handleRefreshClick}>
        Refresh
      </button>
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
    </Suspense>
  );
}
```
[在 CodeSandbox 尝试它](https://codesandbox.io/s/boring-shadow-100tf)

在这个例子中，我们在加载和每一次点击"Refresh"的时候调用数据获取。我们将`fetchUserAndPosts()`的调用结果防止到状态中，这样后面的组件可以从我们发起的请求中读取获取到的数据。

我们可以在[这个例子](https://codesandbox.io/s/boring-shadow-100tf)看到点击"Refresh"是如何工作的。`<ProfileDetails>`和`<ProfileTimeline>`组件接收一个新的`resource`属性，表示刷新的数据，他们"挂起"是因为我们还没有一个响应，我们看到 fallback。当响应加载，我们可以看到更新的文章（我们伪装的 API 每 3 s 添加一次）。

然而，体验非常糟糕。我们浏览一个页面，但是它我们和它交互的时候立即被一个加载状态替代。这很混乱。**就像之前，为了避免显示一个不期待的加载状态，我们可以包裹装载更新在一个过渡中：**
```jsx harmony
function ProfilePage() {
  const [startTransition, isPending] = useTransition({
    // Wait 10 seconds before fallback
    timeoutMs: 10000
  });
  const [resource, setResource] = useState(initialResource);

  function handleRefreshClick() {
    startTransition(() => {
      setResource(fetchProfileData());
    });
  }

  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails resource={resource} />
      <button
        onClick={handleRefreshClick}
        disabled={isPending}
      >
        {isPending ? "Refreshing..." : "Refresh"}
      </button>
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
    </Suspense>
  );
}
```
[在 CodeSandbox 中尝试](https://codesandbox.io/s/sleepy-field-mohzb)

这个感觉好一点！点击"Refresh"没有让我们脱离我们浏览的页面。我们看到一些加载"内联"，当数据准备好，它就会显示。

### 将过渡融合到设计系统

我们现在可以看到`useTransition`的需要非常常见。大量的按钮点击或者交互会导致组件挂起，需要包裹在`useTranstion`去避免意外隐藏用户正在交互的元素。

这会导致组件间大量的重复代码。这是为什么**我们通常推荐将 useTransition 融合到你的应用的设计系统组件**。比如，我们可以抽取逻辑到我们的`<Button>`组件：
```jsx harmony
function Button({ children, onClick }) {
  const [startTransition, isPending] = useTransition({
    timeoutMs: 10000
  });

  function handleClick() {
    startTransition(() => {
      onClick();
    });
  }

  const spinner = (
    // ...
  );

  return (
    <>
      <button
        onClick={handleClick}
        disabled={isPending}
      >
        {children}
      </button>
      {isPending ? spinner : null}
    </>
  );
}
```
[在 CodeSandbox 中尝试](https://codesandbox.io/s/modest-ritchie-iufrh)

注意按钮不关心我们在更新什么状态。它包裹任何发生在`onClick`处理器的状态更新到一个过渡。注意我们`<Button>`关注过渡的设置，`<ProfilePage>`组件不需要设置自己的：
```jsx harmony
function ProfilePage() {
  const [resource, setResource] = useState(initialResource);

  function handleRefreshClick() {
    setResource(fetchProfileData());
  }

  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails resource={resource} />
      <Button onClick={handleRefreshClick}>
        Refresh
      </Button>
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
    </Suspense>
  );
}
```
[在 CodeSandbox 中尝试](https://codesandbox.io/s/modest-ritchie-iufrh)

当按钮被点击的时候，它开始一个过渡，并在它的内部调用`props.onClick()` -- 它触发`<ProfilePage>`组件中的`handleRefreshClick`。我们开始获取新鲜数据，但是它不触发一个 fallback 因为我们在一个过渡内部，在`useTransition`内部指定的 10 秒超时还没过去之前。当过渡是 pending，按钮显示一个内联加载指示器。

我们可以看到 Concurrent 模式如何帮助我们达到一个好的用户体验而不损失组件的独立性和模块化。React 和过渡协调。

### 三步走

到目前位置，我们讨论和一个更新经过的不同虚拟状态。在这个章节，我们将给他们命名，并讨论他们直接的处理过程。
![](https://reactjs.org/static/cm-steps-simple-46bf8ed031b93548272a405e1fb0f1ed-50dad.png)

在最后，我们有"Complete"状态。这才是我们真正想要的。它表示下一个屏幕完全渲染，并且不再加载更多数据。

但是在 Complete 之前，我们可能需要去加载一些数据或者代码。当我们在下一个屏幕，但是它的其中一部分还在加载，我们调用一个 Skeleton 状态。

最后，有两个主要方式让我们到 Skeleton 状态。我们将使用具体例子描述他们之前的不同。

### 默认：Receded -> Skeleton -> Complete

打开[这个例子](https://codesandbox.io/s/prod-grass-g1lh5)并点击"Open Profile"。你将一个一个看到虚拟状态：

- Receded： 一会儿，你将会看到`<h1>Loading the app ...</h1>`fallback。

- Skeleton：你将看到`<ProfilePage>`组件，内部带有`<h1>Loading posts...</h2>`。

- Complete：你将看到`<ProfilePage>`组件，内部没有 fallback。任何东西都被获取。

我们如何分离 Receded 和 Skeleton 状态？他们之间的不同在于 Receded 状态给用户感觉像"退后异步"，当 Skeleton 状态感觉像"前进一步"，在我们显示更多内容的过程中。

在这个例子，我们在`<HomePage>`开始我们的旅程：
```jsx harmony
<Suspense fallback={...}>
  {/* previous screen */}
  <HomePage />
</Suspense>
```
在点击之后，React 开始渲染下一个屏幕：
```jsx harmony
<Suspense fallback={...}>
  {/* next screen */}
  <ProfilePage>
    <ProfileDetails />
    <Suspense fallback={...}>
      <ProfileTimeline />
    </Suspense>
  </ProfilePage>
</Suspense>
```
`<ProfileDetails>`和`<ProfileTimeline>`需要数据去渲染，所以他们挂起：
```jsx harmony
<Suspense fallback={...}>
  {/* next screen */}
  <ProfilePage>
    <ProfileDetails /> {/* suspends! */}
    <Suspense fallback={<h2>Loading posts...</h2>}>
      <ProfileTimeline /> {/* suspends! */}
    </Suspense>
  </ProfilePage>
</Suspense>
```
当一个组件挂起，React 需要去显示最近的 fallback。但是`<ProfileDetails>`最近的 fallback 是在顶部：
```jsx harmony
<Suspense fallback={
  // We see this fallback now because of <ProfileDetails>
  <h1>Loading the app...</h1>
}>
  {/* next screen */}
  <ProfilePage>
    <ProfileDetails /> {/* suspends! */}
    <Suspense fallback={...}>
      <ProfileTimeline />
    </Suspense>
  </ProfilePage>
</Suspense>
```
这就是为什么当我们点击按钮。感觉我们"回到了上一步"，`<Suspense>`边界是前面显示的有用的内容（`<HomePage />`）已经"recede"去显示一个 fallback（`<h1>Loading the app ...</h1>`）。我们叫做 Receded 状态。

随着我们加载更多数据，React 将会尝试渲染，并且`<ProfileDetails>`可以成功渲染。最后，我们在 Skeleton 状态。我们看到带着丢失部分的新页面：
```jsx harmony
<Suspense fallback={...}>
  {/* next screen */}
  <ProfilePage>
    <ProfileDetails />
    <Suspense fallback={
      // We see this fallback now because of <ProfileTimeline>
      <h2>Loading posts...</h2>
    }>
      <ProfileTimeline /> {/* suspends! */}
    </Suspense>
  </ProfilePage>
</Suspense>
```
最后，他们也加载了，我们得到了 Complete 状态。

这个方案（Receded -> Skeleton -> Complete）是默认的一个。然而，Receded 状态并不令人愉快，因为它"隐藏"的存在的信息。这也是为什么 React 使用`useTransition`让我们可选的到一个不同的序列（Pending -> Skeleton -> Complete）。


### Preferred: Pending -> Skeleton -> Complete
当我们`useTransition`，React 将会让我们"呆在"前一个屏幕 -- 并且显示一个进度指示器。我们叫做 Pending 状态。它感觉比 Receded 状态更好，因为没有一个已存在的内容丢失，并且页面依旧可以交互。

- Default: [Receded -> Skeleton -> Complete](https://codesandbox.io/s/prod-grass-g1lh5)

- Preferred: [Pending -> Skeleton -> Complete](https://codesandbox.io/s/focused-snow-xbkvl)

两个例子的不同在于第一个使用常规的`<button>`，但是第二个使用我们自定义的带`useTransition`的`<Button>`组件。

### 在 <Suspense> 包裹 Lazy 功能

打开[这个例子](https://codesandbox.io/s/nameless-butterfly-fkw5q)。当你点击按钮，你将看到 Pending 状态，在你继续之前。这个过渡感觉非常好和新鲜。

我们现在将添加一个新功能到 profile 页面 -- 关于个人的兴趣列表：
```jsx harmony
function ProfilePage({ resource }) {
  return (
    <>
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h2>Loading posts...</h2>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
      <ProfileTrivia resource={resource} />
    </>
  );
}

function ProfileTrivia({ resource }) {
  const trivia = resource.trivia.read();
  return (
    <>
      <h2>Fun Facts</h2>
      <ul>
        {trivia.map(fact => (
          <li key={fact.id}>{fact.text}</li>
        ))}
      </ul>
    </>
  );
}
```
[在 CodeSandbox 尝试](https://codesandbox.io/s/focused-mountain-uhkzg)

如果你现在点击"Open Profile"，你可以告诉一些东西是错误的。它需要七秒去过渡！这是因为我们的 trivia API 太慢了。加入我们无法让 API 更快。我们如何在这个约束下提高用户体验？

如果我们不想要呆在 Pending 状态太久，我们第一个本能是在`useTransition`设置一个更小的`timeoutMS`，比如`3000`。你可以在[这里](https://codesandbox.io/s/practical-kowalevski-kpjg4)尝试。这然我们逃离太长的 Pending 状态，但是我们依旧有任何有效的去显示！

有一个简单的方式去解决这个。**与其让过渡更短，我们可以在过渡中"不连接"慢的组件**通过包含它到`<Suspense>`。
```jsx harmony
function ProfilePage({ resource }) {
  return (
    <>
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h2>Loading posts...</h2>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
      <Suspense fallback={<h2>Loading fun facts...</h2>}>
        <ProfileTrivia resource={resource} />
      </Suspense>
    </>
  );
}
```
[在 CodeSandbox 尝试](https://codesandbox.io/s/condescending-shape-s6694)

这透露了一个重要的见解。React 总是偏向于尽可能尽快的到达 Skeleton 状态。就算我们使用带长时间的过渡，React 将不呆在 Pending 状态太长。去避免 Receded 状态。

**如果一些功能不是下一个屏幕重要的一部分，包裹它到 <Suspense> 并让他惰性初始化**。这强制我们尽可能快的显示剩下的内容。反过来，如果一个屏幕不值得显示一些组件，比如`<ProfileDetails>`的例子，不要包裹在`<Suspense>`。然后过渡将会"等待"它到准备好。

### Suspense Reveal "Train"

当我们已经在下一个屏幕，有时候数据被需要去成功快速"解锁"不同`<Suspense>`边界。比如，两个不同响应可能在 1000 毫秒和 1050 毫秒后各自到达。如果你已经等到 1 秒，等待另一个 50 毫秒是不可擦觉的。这也是为什么 React 使用一个调度展示`<Suspense>`边界，就像一个"火车"按序到达。这需要一点延迟，用来减少布局抖动和可视化改变的数量。

你可以[在这里](https://codesandbox.io/s/admiring-mendeleev-y54mk)看到这个例子。"文章"和"兴趣"响应在 100 毫秒内返回。但是 React 合并他们并"展示"他们的 Suspense 边界。

### Delay a Pending Indicator
我们的`Button`组件在点击的时候将立即显示 Pending 状态指示器：
```jsx harmony
function Button({ children, onClick }) {
  const [startTransition, isPending] = useTransition({
    timeoutMs: 10000
  });

  // ...

  return (
    <>
      <button onClick={handleClick} disabled={isPending}>
        {children}
      </button>
      {isPending ? spinner : null}
    </>
  );
}
```
[在 CodeSandbox 尝试它](https://codesandbox.io/s/floral-thunder-iy826)

这指示用户有些东西发生了。然而，如果过渡相当短（少于 500 毫秒），这可能太迷惑人并且让过渡感觉更慢。

一个可能的解决方案是延迟显示 spinner 本身。

```jsx harmony
.DelayedSpinner {
  animation: 0s linear 0.5s forwards makeVisible;
  visibility: hidden;
}

@keyframes makeVisible {
  to {
    visibility: visible;
  }
}
```
```jsx harmony
const spinner = (
  <span className="DelayedSpinner">
    {/* ... */}
  </span>
);

return (
  <>
    <button onClick={handleClick}>{children}</button>
    {isPending ? spinner : null}
  </>
);

```
[在 CodeSandbox 中尝试它](https://codesandbox.io/s/gallant-spence-l6wbk)

使用这个改变，就算我们在 Pending 状态，我们不显示任何指示器给用户，知道 500 毫秒过去。这可能看起来不是一个提高，当 API 响应慢的时候。但是在 API 调用快的时候对比之前和之后的感觉。尽管剩下的代码没有改变，压制一个"太快的"加载状态提高感知性能，通过不调用注意到延迟。

### 回顾

目前位置我们学到最重要的东西是：

- 默认，我们加载序列是 Receded -> Skeleton -> Complete。

- Receded 状态不是非常好，因为它隐藏了隐藏的存在的内容。

- 使用`useTransition`，我们可以可选的显示一个 Pending 指示器。这将会让我们保持在上一个屏幕，当下一个屏幕正在准备的时候。
- 如果我们不想要一些组件去延迟过渡，我们可以将它包裹到它自己的`<Suspense>`边。

- 与其在每个组件使用`useTransition`，我们可以构建他到我们设计系统。

### 其他模式
过渡可能是你遇到过最普通的 Concurrent 模式，但是还有一些模式你会发现很有用。

### 分离高和低优先级状态
当你设计 React 组件，最好找到状态的"最小表示"。比如，与其保存`firstName`,`lastName`，和`fullName`到状态中，通常最好只保存`firstName`和`lastName`，然后在渲染的时候计算`fullName`。这让我们避免错误，在我们更新状态，但是忘记更新其他状态。

然而，在 Concurrent 模式中，有些场景，你可能想要让一些数据在不同状态变量中"重复"。假设有一个小的翻译应用：
```jsx harmony
const initialQuery = "Hello, world";
const initialResource = fetchTranslation(initialQuery);

function App() {
  const [query, setQuery] = useState(initialQuery);
  const [resource, setResource] = useState(initialResource);

  function handleChange(e) {
    const value = e.target.value;
    setQuery(value);
    setResource(fetchTranslation(value));
  }

  return (
    <>
      <input
        value={query}
        onChange={handleChange}
      />
      <Suspense fallback={<p>Loading...</p>}>
        <Translation resource={resource} />
      </Suspense>
    </>
  );
}

function Translation({ resource }) {
  return (
    <p>
      <b>{resource.read()}</b>
    </p>
  );
}
```
[在 CodeSandbox 中尝试](https://codesandbox.io/s/brave-villani-ypxvf)

注意当我们输入的时候，`<Translation>`，`<Translation>`组件是怎么挂起的，我们看到`<p>Loading...</p>` fallback，直到我们得到新鲜的结果。这不理想。当我们获取下一个的时候，如果我们可以看到前一个翻译多一会就会更好。

实际上，如果我们打开控制台，我们将看到一个警告：
```jsx harmony
Warning: App triggered a user-blocking update that suspended.

The fix is to split the update into multiple parts: a user-blocking update to provide immediate feedback, and another update that triggers the bulk of the changes.

Refer to the documentation for useTransition to learn how to implement this pattern.
```


前面我们提到，如果一些状态更新导致组件挂起，状态更新应该包裹到一个过渡。添加`useTransition`到我们的组件:
```jsx harmony
function App() {
  const [query, setQuery] = useState(initialQuery);
  const [resource, setResource] = useState(initialResource);
  const [startTransition, isPending] = useTransition({
    timeoutMs: 5000
  });

  function handleChange(e) {
    const value = e.target.value;
    startTransition(() => {
      setQuery(value);
      setResource(fetchTranslation(value));
    });
  }

  // ...

}
```
[在 CodeSandbox 试试](https://codesandbox.io/s/zen-keldysh-rifos)

现在尝试输入。有些问题！输入更新的非常慢。我们已经修复了第一个问题（在一个过渡外边挂起）。但是现在因为过渡，我们的状态没有立即更新，它不能"驱动"一个受控的输入框。

这个问题的回答是**分离状态到两部分**：一个立即更新的"高优先级"部分，和一个可能等待过渡的"低优先级"部分。

在我们的例子中，我们已经有两个状态变量。输入框文字在`query`中，我们从`resource`中读取翻译。我们要立即改变`query`，但是改变`resource`（比如，获取一个新的翻译）应该触发一个过渡。

所以，正确的修复是方式`setQuery`（不挂起）到过渡外部，但是`setResource`(挂起)到内部。

```jsx harmony
function handleChange(e) {
  const value = e.target.value;
  
  // Outside the transition (urgent)
  setQuery(value);

  startTransition(() => {
    // Inside the transition (may be delayed)
    setResource(fetchTranslation(value));
  });
}
```

[在 CodeSandbox 中尝试](https://codesandbox.io/s/lively-smoke-fdf93)

使用这个改变，它如期工作。我们可以立即输入到输入框，并且翻译稍后"跟上"我们输入的。

### 延迟一个值

默认，React 总是渲染一个一致性的 UI。假设代码如下：
```jsx harmony
<>
  <ProfileDetails user={user} />
  <ProfileTimeline user={user} />
</>
```

React 保证任何时候我们在屏幕上看到的组件，他们将反映相同`user`的数据。如果一个不同的`user`因为一个更新传输下来，你应该可以看到他们一起改变。你不能记录一个屏幕并找出一个显示不同`user`的值的帧。（如果你曾遇到过这样的例子，提一个 bug！）

这在大部分场景下都有意义。不一致的 UI 是令人困惑的，并且会错误引导用户。（比如，它将会很糟糕，如果一个 messenger 的 Send 按钮和会话选择器面板"不同意"当前选择的线程）

然而，有时候有意识的引入不一致可能有帮助。我们可以通过前面一样的手动`分割`状态，但是 React 提供了一个内置的 Hook。

```jsx harmony
import { useDeferredValue } from 'react';

const deferredValue = useDeferredValue(value, {
  timeoutMs: 5000
});
```

为了演示这个功能，我们将使用[profile 切换例子](https://codesandbox.io/s/musing-ramanujan-bgw2o)。点击"Next"按钮并注意它如何使用 1 秒去完成一个过渡。

假设获取用户详情非常快，只花费 300 毫秒。现在，我们等待整整一秒因为我们需要用户详情和文章去显示一个连续的 profile 页面。但是如果我们希望更快的显示详情呢？

如果我们想要去贡献一致性，我们可以**传递潜在不新鲜数据给组件，延迟他们的过渡**。这就是`useDeferedValue()`让我们做的：
```jsx harmony
function ProfilePage({ resource }) {
  const deferredResource = useDeferredValue(resource, {
    timeoutMs: 1000
  });
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline
          resource={deferredResource}
          isStale={deferredResource !== resource}
        />
      </Suspense>
    </Suspense>
  );
}

function ProfileTimeline({ isStale, resource }) {
  const posts = resource.posts.read();
  return (
    <ul style={{ opacity: isStale ? 0.7 : 1 }}>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandbox 尝试](https://codesandbox.io/s/vigorous-keller-3ed2b)

这里我们做的交易是`<ProfileTimeline>`将会和其他组件不一致，并且显示一个旧的项目。点击"Next"一点时间以后，你将会注意到。但是感谢它，我们能将延迟过渡时间从 1000 毫秒到 300 毫秒。

不管如何躲着这个场景来说，这交易是很值得的。但是这是一个灵巧的工具，特别是当内容改变不改变注意，并且用户可能没有意识到他们短时间看到的是不新鲜的版本。

值得注意的是`useDefferdValue`不仅仅用于数据获取。当一个昂贵的组件树导致交互（比如，输入一个输入框）缓慢时也有帮助。就像我们可以"延迟"一个需要很长时间去获取的值（并且显示它的值，尽管其他组件更新），我们可以这么做当需要长时间去渲染的时候。

比如，假设一个可过滤的列表：
```jsx harmony
function App() {
  const [text, setText] = useState("hello");

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <div className="App">
      <label>
        Type into the input:{" "}
        <input value={text} onChange={handleChange} />
      </label>
      ...
      <MySlowList text={text} />
    </div>
  );
}
```

[在 CodeSandbox 中尝试](https://codesandbox.io/s/pensive-shirley-wkp46)

在这个例子，**<MySlowList>中的每一项都有一个人工延迟 -- 每一个延迟线程一些时间**。我们不会在真实的应用这么做，但是这帮助我们模拟没有可以优化的组件树中发生了什么。

我们可以看到输入到输入框如何导致卡顿。现在添加`useDefferedValue`：
```jsx harmony
function App() {
  const [text, setText] = useState("hello");
  const deferredText = useDeferredValue(text, {
    timeoutMs: 5000
  });

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <div className="App">
      <label>
        Type into the input:{" "}
        <input value={text} onChange={handleChange} />
      </label>
      ...
      <MySlowList text={deferredText} />
    </div>
  );
}
```
[在 CodeSandbox 中尝试](https://codesandbox.io/s/infallible-dewdney-9fkv9)

现在卡顿少了很多 -- 尽管我们通过使用一个延迟显示一个结果。

这和节流有啥区别？我们的例子有一个固定的人工延迟（80 项每一项都有 3 毫秒），因此总是有一个延迟，不管我们的计算机有多快。然而，`useDefferrdValue`值只在渲染一段时间后"延迟"。对于 React 没有最小的延迟。使用更实际的工作负载，你可以认为延迟是根据设备调整的。在快的机器，延迟会更小或者不存在，在慢的机器，它更容易被注意。在两场场景中，应用将保持响应。这是这个机制相对于防抖和节流的优势，他们总是有最小的延迟，无法在渲染的时候避免堵塞线程。

尽管在响应性方面还可以提高，这个例子没有很好的展示，因为 Concurrent Mode 对于这个场景缺少一些关键的优化。它依旧显示了像`useDeferredValue`(或者`useTransition`)之类的特性，他们很有用，无论我们等待网络或者计算工作完成。

这里的问题是现在我们总是等待他们全部被获取，如果文章先返回，没有理由延迟显示他们。当兴趣后面加载的时候，他们不会改变布局，因为他们已经在文章后面


### SuspenseList
`<SuspenseList>`是和加载状态编排有关的最后模式。

考虑这个例子：
```
function ProfilePage({ resource }) {
  return (
    <>
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h2>Loading posts...</h2>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
      <Suspense fallback={<h2>Loading fun facts...</h2>}>
        <ProfileTrivia resource={resource} />
      </Suspense>
    </>
  );
}
```

[在 CodeSandbox 尝试它](https://codesandbox.io/s/proud-tree-exg5t)

在这个例子调用的 API 是随机的。如果你保持刷新它，你将注意到有时候文章先到，有时候"兴趣"先到。

这存在一个问题。如果兴趣的响应先到，我们将在文章的 fallback `<h2>Loading post...</h2>`看到兴趣。我们可能开始阅读他们，但是文章的响应将会 回来，并且将所有的兴趣推后。这很糟糕。

修复它的一种放回事将他们都放在一个的边界：
```jsx harmony
<Suspense fallback={<h2>Loading posts and fun facts...</h2>}>
  <ProfileTimeline resource={resource} />
  <ProfileTrivia resource={resource} />
</Suspense>
```
[在 CodeSandbox 尝试它](https://codesandbox.io/s/currying-violet-5jsiy)

这里的问题是我们总是等来他们全部被获取，如果文章先回来，没有理由去显示他们。当兴趣后加载，他们不会下推布局，因为他们总是在文章之后。

其他解决方式，比如以特定方式组合 Promise，当加载状态在树的不同组件，增加获取的复杂度。

为了解决这个问题，我们引入`SuspenseList`：
```jsx harmony
import { SuspenseList } from 'react';

```
`<SuspenseList>`位于最近的`<Suspense>`节点后的"展示顺序"：
```jsx harmony
function ProfilePage({ resource }) {
  return (
    <SuspenseList revealOrder="forwards">
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h2>Loading posts...</h2>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
      <Suspense fallback={<h2>Loading fun facts...</h2>}>
        <ProfileTrivia resource={resource} />
      </Suspense>
    </SuspenseList>
  );
}
```
[在 CodeSandbox 尝试它](https://codesandbox.io/s/black-wind-byilt)

`revealOrder="forwards"`选项意味着最近的`<Suspense>`节点内部**将只"展示"他们的内容，以出现在树的顺序 -- 就算他们的数据以不同的顺序到达**。`<SuspenseList>`有其他有趣的模式：尝试改变`"forwards"`到`"backforwards"`或者`"together"`，并查看发生了啥。

使用`tail`属性，你可以控制多少加载状态同时可见。如果我们指定`tail="collapsed"`，我们最多同时可以看见一个 fallback。你可以在[这里](https://codesandbox.io/s/adoring-almeida-1zzjh)试试。

记住`<SuspenseList>`是可以组合的，和 React 中的任何东西。比如，你可以创建一个网格，通过防止多行`<SuspenseList>`在一个`<SuspenseList>`表格内。

### 下一步

Concurrent 模式提供一个强大的 UI 变成模型和一系列可组合的原语去帮助你去编排愉快的用户体验。

这是多年开发和调研的结果，但是这还没完。在[使用 Concurrent 模式](https://reactjs.org/docs/concurrent-mode-adoption.html)，我们将描述你如何尝试和你可以期待什么。























