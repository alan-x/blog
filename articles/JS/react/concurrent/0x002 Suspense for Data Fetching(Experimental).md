### Suspense for Data Fetching(Experimental)

注意：

这个页面描述了**稳定发布中[还不可用]()的实验性功能**。不要在生产 app 中依赖 React 的试验性构建。在成为 React 的一部分之前，这些功能可能发生非常大的改变，并且没有警告。

这个文档面向早期接受者和好奇的人。**如果你刚接触 React，不要单鞋这些功能** -- 你不需要立马学习他们。比如，如果你在寻找现在使用的数据获取指南，阅读[这篇文章]()。

React 16.6 添加了一个`<Suspense>`组件，让你"等待"一些代码去加载，并声明式的指定一个加载状态（比如一个旋转器），当我们在等待的时候：
```jsx harmony
const ProfilePage = React.lazy(() => import('./ProfilePage')); // Lazy-loaded

// Show a spinner while the profile is loading
<Suspense fallback={<Spinner />}>
  <ProfilePage />
</Suspense>
```
因为数据获取而挂起是一个新功能，让你使用`<Suspense>`去**声明式的"等待"其他东西，包括数据**。这个页面聚焦于数据获取场景，但是也可以用于等待图像，脚本，或者其他异步工作。

- [What is Suspense, Exactly?]()
    - [What Suspense Is Not]()
    - [What Suspense Lets You Do]()
- [Using Suspense is Practice]()
    - [What If I Dont't Use Relay?]()
    - [For Library Authors]()
- [Traditional Approaches vs Suspense]()
    - [Approach 1: Fetch-on-Render(not using Suspense)]()
    - [Approach 2: Fetch-Then-Render(not using Suspense)]()
    - [Approach 3: Render-as-You-Fetch(using Suspense)]()
- [Start Fetching Early]()
    - [We're Still Figuring This Out]()
- [Suspense and Race Condition]()
    - [Race Conditions with useEffect]()
    - [Race Conditions with componentDidUpdate]()
    - [The Problem]()
    - [Solving Race Conditions with Suspense]()    
- [Handing Errors]()
- [Next Steps]()


### 挂起到底是啥？
挂起让你的组件"等待"一些东西，在他们可以渲染之前。在[这个例子]()，里那个组件等待一个获取一些数据的异步 API 调用。

```jsx harmony
const resource = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails() {
  // Try to read user info, although it might not have loaded yet
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  // Try to read posts, although they might not have loaded yet
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandBox 尝试]()

这个例子是一个预告。不要担心没如果没有啥意义。我们将在下面讨论它如何工作。记住 Suspense 更像一个机制，前面例子中像`fetchProfileData()`或者`resource.posts.read()`特定的 API 并不是很重要。如果你有兴趣，你可以直接在[沙箱例子]()中直接找到他们的定义。

Suspense 不是一个数据获取库。它是一个机制，用于数据获取库和 React 交流，说明组件正在读取的数据还没准备好。React 可以等待它准备好并更新 UI。在 Facebook，我们使用 Relay 和它的[新 Suspense 集成]()。我们期待其他库，比如 Apollo 可以提供类似的集成。

### Suspense 不是什么

Suspense 和已存在的解决这类问题的方式有重大的不同，因此第一次阅读他们将导致误解。现在来澄清一下最常见的：

- **不是一个数据获取实践**。它不假设你使用 GraphQL，REST，或者其他特定的数据格式，库，传输，或者协议。

- **不是一个准备好去使用的客户端**。你不能用 Suspense "替代"`fetch`或者 Relay。但你你可以使用库和 Suspense 集成（比如，[新的 Relay API]()）。

- **它不绑定数据获取到视图层**。它帮助编排显示一个加载状态到你的 UI，但是它不绑定你的网络逻辑到 React 组件。

### Suspense 让你做什么？

所以，Suspense 的关键是什么？关于这个我们有多种方式回答：

- **它让数据获取库和 React 深度集成**。如果一个数据获取库实现对 Suspense 的支持，在 React 组件使用它感觉非常自然。

- **它仍你编排想要的设计的加载状态**。它不关心数据怎样获取，当时它让你更好控制你应用的可视的加载序列。

- **它帮你避免竞态条件**。就算和`await`，异步代码总是容易出错的。Suspense 更像同步获取数据 -- 就像它已经加载完成。


### 在实践中使用 Suspense
在 Facebook，到目前为止，在产品中，我们只使用 Relay 集成 Suspense。**如果你寻找一个实践指南，[查阅 Relay 指南！]() 它展示已经在我们产品中工作很好的模式。

**这个页面的代码使用一个"fake" API 实现，而不是 Relay**。这让他们很容易理解，如果你对 GraphQL 不熟悉。但是他们不会告诉你使用 Suspense 构建你的应用的"正确方式"。这个页面更偏向于概念化，为帮助你了解为什么 Suspense 以某种方式工作，和它解决了什么问题。


### 如果不使用 Relay？

如果你不使用 Relay，你可能可能需要等待，在你真的尝试在你的应用使用 Suspense 之前。到目前为止，这是唯一个实现，我们在产品中测试的和有信心的。

再过几个月，很多库将会以不同的实现来表示 Suspense API。**如果你偏向于在事物更加稳定的时候学习，你可能偏向于现在忽略这个，并在 Suspense 生态更加成熟的时候回来。**

你也可以编写你自己的数据获取集成库，如果你喜欢。


### 对于库作者

我们期待看到社区使用其他库的大量实验。对于数据获取库作者有一点很重要。

尽管技术上可行，Suspense 当前的异步不是成为一个在组件渲染的时候开始获取数据的方式。而是，它让组件表示他们"等待"已经被获取的数据。**[使用 Concurrent 模式和 Suspense 构建好的用户体验]()**描述了为什么这种方式和在实践中如何实现这个模式。

除非你有解决方案帮助防止 waterfall，我们推荐使用支持或者强制在渲染之前获取数据的 API。对于一个具体的例子，你可以看看[Relay Suspense API]() 如何强制提前获取。我们关于这个的信息在过去并不一致。数据获取的 Suspense 依旧是试验性的，所以你可以期待我们的推荐随着实践改变，正如我们更好的从产品使用和理解问题空间中学习到更多。

我们可以不提到流行的数据获取方式来介绍 Suspense。然而，这让它更加困难的去了解 Suspense 解决了什么问题，为什么这些问题值得解决，Suspense 和已经存在的解决方案有啥区别。

相反，我们将看到 Suspense 作为一系列方案的逻辑下一步：

- **渲染中获取（比如，在 useEffect 中获取）：** 开始渲染组件。每一个组件可能在他们的副作用和生命周期方法中触发数据获取。这些方法通常导致"waterfall"。

- **在渲染之后获取（比如，不使用 Suspense 的 Relay）：** 尽可能尽快开始获取下一个屏幕需要的所有数据。当数据准备好，渲染新屏幕。我们无法做任何事情，知道数据到达。

- **在获取的时候获取（比如，Relay 和 Suspense）：** 尽快开始获取下一个屏幕需要的所有数据，然后立即开始渲染新屏幕 -- 在我们获取一个网络响应之前。随着数据流入，React 尝试重新渲染依旧需要数据的组件，知道他们全部准备好。

注意：这有一点简化，实际的例子中，可能偏向使用混合的方法。我们依旧独立的看待他们，为了更好的对比他们的利弊。

为了比较这些方式，我们使用每一种实现了一个 profile 页面。

### 方式 1：渲染的时候获取（不使用 Suspense）

React 中现在常用的数据获取方式是使用一个副作用：
```jsx harmony
// In a function component:
useEffect(() => {
  fetchSomething();
}, []);

// Or, in a class component:
componentDidMount() {
  fetchSomething();
}
```
我们吧这个方式叫做"渲染的时候获取**，因为它在组件被渲染到屏幕之前不获取数据。这导致了被称为"waterfall"的问题。

假设有`<ProfilePage>`和`<ProfileTimeLine>`组件：
```jsx harmony
function ProfilePage() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser().then(u => setUser(u));
  }, []);

  if (user === null) {
    return <p>Loading profile...</p>;
  }
  return (
    <>
      <h1>{user.name}</h1>
      <ProfileTimeline />
    </>
  );
}

function ProfileTimeline() {
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    fetchPosts().then(p => setPosts(p));
  }, []);

  if (posts === null) {
    return <h2>Loading posts...</h2>;
  }
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandbox 中尝试]()

如果你运行代码并在控制台中观察，你将会注意到以下序列：

1. 我们开始获取用户详情
2. 我们等待...
3. 我们结束获取用户详情
4. 我们开始获取文章
5. 我们等待...
6. 我们结束获取文章

如果获取用户详情需要 3 秒，我只能在三秒后获取文章！这就是"waterfall"：应该并行的请求变成序列。

waterfall 在渲染的石斛获取的代码中很常见。他们可能解决，但是随着产品发展，很多人偏向使用一个解决方案来规避这个问题。

### 方案2：获取之后渲染（不使用 Suspense）

库可以防止 waterfall，通过提供更加集中的方式去获取数据。比如，Relay 解决这个问题，通过移动一个组件需要的数据信息到静态的可分析的片段，然后组合到单个查询。

在这个页面，我们不假设任何关于 Relay 的只是，因此，我们将手动编写类似的，通过结合我们的数据获取方法：
```jsx harmony
function fetchProfileData() {
  return Promise.all([
    fetchUser(),
    fetchPosts()
  ]).then(([user, posts]) => {
    return {user, posts};
  })
}
```

在这个例子中，`<ProfilePage>`等待两个请求，但是同时开始他们：
```jsx harmony
// Kick off fetching as early as possible
const promise = fetchProfileData();

function ProfilePage() {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    promise.then(data => {
      setUser(data.user);
      setPosts(data.posts);
    });
  }, []);

  if (user === null) {
    return <p>Loading profile...</p>;
  }
  return (
    <>
      <h1>{user.name}</h1>
      <ProfileTimeline posts={posts} />
    </>
  );
}

// The child doesn't trigger fetching anymore
function ProfileTimeline({ posts }) {
  if (posts === null) {
    return <h2>Loading posts...</h2>;
  }
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandbox 中尝试]()

事件序列开始像这样：
1. 我们开始获取用户详情
2. 我们开始获取文章
3. 我们等待...
4. 我们结束用户详情获取
5. 我们结束文章获取

我们解决了前面的网络"waterfall"，但是意外引入了不同的一个。我们等待`fetchProfileData`中的`Promise.all()`的所有数据回来，现在我们不能渲染 profile 详情，直到 post 也被获取倒了。我们需要去等待两者。

当然，修复这个特定的例子也是可能的，我们可以移除`Promise.all()`调用，并分别等待两个 Promise。然而，这个方案随着我们数据和组件树的增常更加复杂。很难编写可信赖的组件，当数据树的任意一部分丢失或者不新鲜的时候。为新屏幕获取所有数据，然后渲染通常是一个实践选项

### 方案3：随着你的获取渲染（使用 Suspense）
在前面的方案中，我们在调用`setState`之前获取数据：
1. 开始获取
2. 结束获取
3. 开始渲染

使用 Suspense，我们依旧先开始获取数据，但是我们吧后面两个步骤反过来：
1. 开始获取
2. 开始渲染
3. 结束获取

**使用 Suspense，我们不等待响应回来，在我们开始渲染之前。** 实际上，我们在启动网路哦请求后立即开始渲染：
```jsx harmony
// This is not a Promise. It's a special object from our Suspense integration.
const resource = fetchProfileData();

function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails() {
  // Try to read user info, although it might not have loaded yet
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline() {
  // Try to read posts, although they might not have loaded yet
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandbox 中尝试]()

这是当我们渲染`<ProfilePage>`到屏幕的时候发生的事：
1. 我们已经在`fetchProfileData()`中发起请求。它给我们一个指定的"资源"，而不是一个 Promise。在更早的例子，它通过我们数据数据库的 Suspense 集成提供，比如 Relay。

2. React 尝试渲染`<ProfilePage>`。它返回`<ProfileDetails>`和`<ProfileTimeline>`作为子元素。

3. React 尝试渲染`<ProfileDetails>`。它调用`resource.user.read()`。没有数据被获取，所以这个组件"挂起"。React 跳过它，并尝试渲染树中的其他组件。

4. React 尝试渲染`<ProfileTimeline>`它调用`resource.posts.read()`。同样，没有数据，因此这个组件也被"挂起"。React 也跳过它，并尝试渲染树中的其他组件。

5. 没有其他可以渲染。因为`<ProfileDetails>`挂起，React 显示树前面最近的`<Suspense>` fallback：`<h1>Loading Profile</h1>`。我们现在完成了。

这个`resource`对象表示数据还没有，或者事实上已经加载了。当我们调用`read()`，我们获取数据，或者组件"挂起"。

**随着更多数据流入，React 将会尝试渲染，并且每次它可能进展更"深入"。当`resource.user`被获取，`<ProfileDetails>`组件将会成功渲染，将不再需要`<h1>Loading profile...</h1>` fallback。事实上，我们将会获取所有数据，并且将不会有 fallback 在屏幕上。

这有一个有趣的暗示。就算我们使用一个 GraphQL 客户端，收集单个请求需要的所有数据，响应的流入让我们尽快显示更多内容。因为我们边获取边渲染（而不是在获取之后），如果`user`比`posts`更早出现在响应中，我们将解锁外边的`<Suspense>`边界，在响应还没结束之前。我们可能丢失它，但是就算获取然后渲染解决方案包含一个 waterfall：在渲染和获取之间，Suspense 不受 waterfall 的影响，Relay 之类的库就利用了这点。

注意我们是怎样从我们的组件消除`if(...)`"is loading"检查的。这不仅仅移除了样板代码，它还简化了设计改变。比如没如果我们想要 profile 详情和 post 总是同时"弹出"，我们可以删除他们之间的`<Suspense>`边界，我们可以删除他们之间的`<Suspense>`边界。或者我们可以让他们相互独立痛殴给他们自己的`<Suspense>`边界。Suspense 让我们改变改变加载力度的排序，并编排他们的序列而不需要侵入我们的代码。


### 更早开始获取

注意这个例子中的`read()`调用没有开始获取。它只尝试读取**已经获取到的**。这个不同对使用 Suspense 创建更快的应用至关重要。我们不想要延迟加载数据，直到组件开始渲染。作为一个数据获取库作者，你可以强制这个，通过创建一个`resource`对象而没有开始获取。这个页面的每一个例子使用我们的"fake API"强制这个。

你可能觉得像这个例子在"顶部获取"获取是不现实的。如果我们导航到其他 profile 页面，我们该做什么呢？我们可能基于属性获取。回答是**我们想要在事件处理器开始获取**。这是一个在用户页面导航的简单例子：
```jsx harmony
// First fetch: as soon as possible
const initialResource = fetchProfileData(0);

function App() {
  const [resource, setResource] = useState(initialResource);
  return (
    <>
      <button onClick={() => {
        const nextUserId = getNextId(resource.userId);
        // Next fetch: when the user clicks
        setResource(fetchProfileData(nextUserId));
      }}>
        Next
      </button>
      <ProfilePage resource={resource} />
    </>
  );
}
```
[在 CodeSandbox 中尝试]()

使用这个方式，我们可以**同步获取数据和代码**。当我们在页面之间导航，我们不需要等待页面代码加载去开始获取数据。我们可以同时获取代码和数据（在链接点击的时候），提供更好的用户体验。

这有一个问题，那就是我们怎么知道在渲染下一个屏幕之前需要获取什么数据。有多种方式去解决这个（比如，通过将数据获取和路由更好的集成解决）。[使用 Concurrent 模式和 Suspense 构建更好的用户体验]()深入介绍了怎样实现这个和为什么重要。

### 我们将依旧指出这个

Suspense 它作为一个机制是弹性的，并且没有很多约束。产品代码需要更多越苏确保没有 waterfall，但是使用不同的方式提供保证。我们现在探索的问题包括：

- 更早的获取很难去表述。我们如何做去更简单的避免 waterfall ？

- 当我们为一个页面获取数据的时候， API 鼓励包含数据而不是及时转换？

- 响应的生命周期是什么？缓存是全局的还是本地的？谁管理缓存？

- 代理可以帮助表示懒加载 API，不需要插入`read()`？

- 和 GraphQL 查询组合之后，任何 Suspense 数据看起来像啥？

Relay 对这些问题有自己的回答。不仅仅只有一种单一的方式去实现，我们很激动可以在 React 社区看到新的想法。


### Suspense 和竞态条件

竞态条件是 bug，因为错误的假设我们代码可能允许的顺序。在`useEffect` Hook 或者在类生命周期方法`componentDidUpdate`中获取数据通常导致这些。Suspense 在这里也有帮助，-- 看看怎样做到。

为了演示这个问题，我们将添加顶级的`<App>`组件渲染我们的`<ProfilePage>`，它有一个按钮让我们**在不同 profile 之间切换**：
```jsx harmony
function getNextId(id) {
  // ...
}

function App() {
  const [id, setId] = useState(0);
  return (
    <>
      <button onClick={() => setId(getNextId(id))}>
        Next
      </button>
      <ProfilePage id={id} />
    </>
  );
}
```
比较以下不同获取策略如何解决这个需求。

### 竞态条件和 useEffect
首先，我们将尝试我们原始的"在副作用中获取"版本例子。我们将修改它，传递一个`id`参数，来自`<ProifilePage>`属性到`fetch(id)`和`fetchPosts(id)`：
```jsx harmony
function ProfilePage({ id }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(id).then(u => setUser(u));
  }, [id]);

  if (user === null) {
    return <p>Loading profile...</p>;
  }
  return (
    <>
      <h1>{user.name}</h1>
      <ProfileTimeline id={id} />
    </>
  );
}

function ProfileTimeline({ id }) {
  const [posts, setPosts] = useState(null);

  useEffect(() => {
    fetchPosts(id).then(p => setPosts(p));
  }, [id]);

  if (posts === null) {
    return <h2>Loading posts...</h2>;
  }
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandbox 中尝试]()

注意我们怎样将副作用的依赖从`[]`改成`[id]` -- 因为我们要让副作用在`id`改变的时候重新运行。否则，我们将获取新的数据。

如果我们尝试这个代码。它可能开起来可行。然而，如果我们在我们的"fake API"实现中随机延迟，并足够快的点击"Next"按钮，我们将看到输入日志有一些错误。**上一次的请求有时候在我们已经切换到其他 ID 之后"回来" -- 在这个例子中他们可以为一个不同的 ID 使用不新鲜的响应重写状态。**

这个问题可以被修复（你可以使用副作用清理函数去或略获取取消不新鲜的亲故），但是不直观，并且很难调试。

### 竞态条件和 componentDidUpdate

可能认为这是`useEffect`和 Hooks 才有的问题。如果如果我们使用这个代码到类或者使用方便的语法，比如`async`/`await`，它将会解决这个问题？

让我们来试试：
```jsx harmony
class ProfilePage extends React.Component {
  state = {
    user: null,
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const user = await fetchUser(id);
    this.setState({ user });
  }
  render() {
    const { id } = this.props;
    const { user } = this.state;
    if (user === null) {
      return <p>Loading profile...</p>;
    }
    return (
      <>
        <h1>{user.name}</h1>
        <ProfileTimeline id={id} />
      </>
    );
  }
}

class ProfileTimeline extends React.Component {
  state = {
    posts: null,
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const posts = await fetchPosts(id);
    this.setState({ posts });
  }
  render() {
    const { posts } = this.state;
    if (posts === null) {
      return <h2>Loading posts...</h2>;
    }
    return (
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.text}</li>
        ))}
      </ul>
    );
  }
}
```
[在 CodeSandbox 中试试]()

这个代码看起来很容易阅读。

不幸的是，使用类或者`async`/`await`语法帮助我们解决这个问题。这个版本遭受了同样的竞态条件，因为一些原因。

### 问题

React 组件有他们自己的"生命周期"。他们可能接收属性或者更新状态，在任意时间点。然而，每一个异步请求都有自己的"生命周期"。它在我们启动它的时候开始，在我们得到响应的时候结束。我们经历的困难是同时"同步"多个互相影响的进程。这很难去思考。

### 使用 Suspense 解决竞态条件

让我们重写这个例子，但是只使用`Suspense`：
```jsx harmony
const initialResource = fetchProfileData(0);

function App() {
  const [resource, setResource] = useState(initialResource);
  return (
    <>
      <button onClick={() => {
        const nextUserId = getNextId(resource.userId);
        setResource(fetchProfileData(nextUserId));
      }}>
        Next
      </button>
      <ProfilePage resource={resource} />
    </>
  );
}

function ProfilePage({ resource }) {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails resource={resource} />
      <Suspense fallback={<h1>Loading posts...</h1>}>
        <ProfileTimeline resource={resource} />
      </Suspense>
    </Suspense>
  );
}

function ProfileDetails({ resource }) {
  const user = resource.user.read();
  return <h1>{user.name}</h1>;
}

function ProfileTimeline({ resource }) {
  const posts = resource.posts.read();
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>{post.text}</li>
      ))}
    </ul>
  );
}
```
[在 CodeSandBox 中尝试]()
在前面 Suspense 例子中，我们只有一个`resource`，因此我们在顶层变量持有它。现在我们有多个资源，我们移动它到`<App>`的组件状态：
```jsx harmony
const initialResource = fetchProfileData(0);

function App() {
  const [resource, setResource] = useState(initialResource);
```
当我们点击"next"，`<App>`组件启动一个请求，为下一个 profile，并向下传递对象到`<ProfilePage>`组件：
```jsx harmony
  <>
    <button onClick={() => {
      const nextUserId = getNextId(resource.userId);
      setResource(fetchProfileData(nextUserId));
    }}>
      Next
    </button>
    <ProfilePage resource={resource} />
  </>
```
再一次，注意**我们不等待响应去设置状态。有其他方式去解决：我们在启动一个请求之后立即设置状态（并开始渲染）。** 然后我们有更多数据，React "填充" `<Suspense>`组件内部的内容。

这个代码是可读的，但是不像之前的例子，Suspense 版本没有竞争条件，你可能会思考为什么。回答是在 Suspense 版本，在我们代码中我们没有太多思考时间。我们有竞态条件的原始代码需要设置状态，在一段时间后，或者其他可能错误。但是使用 Suspense，我们立即设置状态 -- 所以很难去理清。

### 错误处理

当我们使用 Promise 编写代码，我们可能使用`catch()`去处理错误。Suspense 如何做到的，不等待 Promise 就开始渲染？

使用 Suspense，处理获取异常和渲染异常一样 -- 你可以渲染一个[错误边界]()去"捕获"后面组件的错误。

首先，我们将定义一个错误边界组件去跨项目使用：
```jsx harmony
// Error boundaries currently have to be classes.
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };
  static getDerivedStateFromError(error) {
    return {
      hasError: true,
      error
    };
  }
  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```
然后我们可以将它放在树的任何地方去捕获错误：
```jsx harmony
function ProfilePage() {
  return (
    <Suspense fallback={<h1>Loading profile...</h1>}>
      <ProfileDetails />
      <ErrorBoundary fallback={<h2>Could not fetch posts.</h2>}>
        <Suspense fallback={<h1>Loading posts...</h1>}>
          <ProfileTimeline />
        </Suspense>
      </ErrorBoundary>
    </Suspense>
  );
}
```
[在 CodeSandbox 中尝试]()
它可以捕获渲染异常和错误，从 Suspense 数据获取。我们可以拥有我们想要的数量的粗五编辑，但是最好根据目的放置。

### 下一步
现在我们覆盖了 Suspense 数据获取的基本！重要的是，我们现在更加理解为什么 Suspense 这么做，和它怎样填充数据获取空间。

Suspense 回答了一些问题，但是也给出了他自己的问题：

- 如果组件"挂起"，应用会冻结吗？怎样避免？

-如果我们想要现实一个旋转在一个不同的地方，而不是组件所在的树的`前面`？

- 除了现实一个旋转器，我们可以添加可视化效果，比如"greying out"当前屏幕？

- 为什么我们[最后一个 Suspense 例子]()在点击"Next"按钮的时候打出一个警告？

为了回答这些问题，我们将在下一个章节[Concurrent UI 模式]()引用。
