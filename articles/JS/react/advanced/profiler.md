[原文地址](https://reactjs.org/docs/profiler.html)
### Profiler API

`Profiler`测量 React 应用渲染的频率，和渲染的"损耗"。它的目的是帮助标识应用慢的部分，并且从[使用记忆优化](https://reactjs.org/docs/hooks-faq.html#how-to-memoize-calculations)受益。

注意：分析增加了额外的开销，因此**它在[产品构建](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build)中禁止。

 使用产品分析，React 提供了一个特殊的产品构建，它启用了分析。在[fb.me/react-profiling](https://fb.me/react-profiling)阅读更多关于如何使用。

### 使用

`Profiler`可以添加到 React 树的任何部分去测量渲染树的这部分的损耗。它需要两个属性：一个`id`(字符串)和一个`onRender`回调（函数），当树内的组件"提交"一个更新的时候，React 会调用它。

比如，为了分析一个`Navigation`组件和它的子孙：
```jsx harmony
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Main {...props} />
  </App>
);
```
多个`Profiler`组件可以用于测量应用的不同部分：
```jsx harmony
render(
  <App>
    <Profiler id="Navigation" onRender={callback}>
      <Navigation {...props} />
    </Profiler>
    <Profiler id="Main" onRender={callback}>
      <Main {...props} />
    </Profiler>
  </App>
);
```
`Profiler`组件可以在相同子树内嵌套测量不同组件：
```jsx harmony
render(
  <App>
    <Profiler id="Panel" onRender={callback}>
      <Panel {...props}>
        <Profiler id="Content" onRender={callback}>
          <Content {...props} />
        </Profiler>
        <Profiler id="PreviewPane" onRender={callback}>
          <PreviewPane {...props} />
        </Profiler>
      </Panel>
    </Profiler>
  </App>
);
```

注意：尽管`Profiler`是一个轻量组件，它只应该在必要的时候使用；每一次使用添加一些应用的 CPU 和内存开销。

### onRender 回调

`Profiler`需要一个`onRender`函数作为属性。React 在分析的树中"提交"一个更新的时候调用这个函数。它接收描述渲染的东西和花费的时间作为参数。
```jsx harmony
function onRenderCallback(
  id, // the "id" prop of the Profiler tree that has just committed
  phase, // either "mount" (if the tree just mounted) or "update" (if it re-rendered)
  actualDuration, // time spent rendering the committed update
  baseDuration, // estimated time to render the entire subtree without memoization
  startTime, // when React began rendering this update
  commitTime, // when React committed this update
  interactions // the Set of interactions belonging to this update
) {
  // Aggregate or log render timings...
}
```

近距离看看每一个属性：

- **id: string** - 刚刚被提交的`Profiler`树的`id`属性。如果你有多个分析器，这可以用来标识树的那一部分被提交。

- **phase: "mount" | "update"** - 标识树第一次被挂载或者因为属性，状态，或者 hooks 的改变导致重新渲染。

- **actualDuration: number** - 为这次更新渲染`Profiler`和它的子元素花费的时间。这指示子树对记忆的使用情况（比如，[React.memo](https://reactjs.org/docs/react-api.html#reactmemo)，[useMemo](https://reactjs.org/docs/hooks-reference.html#usememo)，[shouldComponentUpdate](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)）。理想情况下，这个值在初始化挂载之后极大的减小，因为大部分的子组件将只需要在属性改变的时候重新渲染。

- **baseDuration: number**  - `Profiler`树内每一个独立组件最近的`render`时间的持续时间。这个值建立熏染损耗的最坏情况（比如，初始化挂载或者没有记忆的树）。

- **startTIme: number** - React 开始渲染当前更新的时间戳

- **commitTime: number** - React 提交当前更新的时间戳。这个值在一个提交的所有分析器之间共享，如果需要可以分组。

- **interactions: Set** - 更新被调度的"[interaction](https://fb.me/react-interaction-tracing)"集合（比如，当`render`或者`setState`被调用）。

注意：交互可以用于标识一个更新的起源，尽管跟踪他们的 API 依旧是试验性的。

在[fb.me/react-interaction-tracing](https://fb.me/react-interaction-tracing)了解更多。
