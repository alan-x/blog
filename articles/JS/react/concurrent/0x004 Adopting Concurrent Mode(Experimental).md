[原文地址](https://reactjs.org/docs/concurrent-mode-adoption.html)
### Adopting Concurrent Mode(Experimental)

注意：

这个页面描述了**稳定发布中还不可用的实验性功能**。不要在生产应用中依赖 React 的试验性构建。在成为 React 的一部分之前，这些功能可能发生非常大的改变，并且没有警告。

这个文档面向早期接受者和好奇的人。**如果你刚接触 React，不要担心这些功能** -- 你不需要立马学习他们。

### 安装

Concurrent 模式只在 React 的[体验构建](https://reactjs.org/blog/2019/10/22/react-release-channels.html#experimental-channel)可用。为了安装他们，执行：
```jsx harmony
npm install react@experimental react-dom@experimental
```
**没有语义化版本保证体验构建。**

`@experimental`发行的 API 可能被添加，改变，或者移除。


**体验发行将会经常断代改变。**

你可以在个人项目或者在一个分支尝试这些构建，但是我们不推荐在生产运行他们。在 Facebook，我们的确在产品中运行他们，但是这是因为我们也修复 bug，当出现问题的时候。你将被警告。

### 这个体验发行是为了谁

这个发行主要目标是早期接受者，库作者和好奇的人们。

我们在生产中使用这个代码（这对我们有效），但是依旧有一些 bug，丢失的特性，和文档的缺失。我们希望听到更多关于 Concurrent 模式的问题，这样我们可以更好准备它，让它在未来成为稳定的发行。


### 启用 Concurrent 模式

通常，当我们添加功能到 React， 你可以立即使用他们。Fragment，Context，甚至 Hook 也是这类功能的例子。你可以在新代码中使用，不需要改变任何存在的代码。

Concurrent 模式不同。它导致 React 怎样工作的语义变化。否则，[新功能](https://reactjs.org/docs/concurrent-mode-patterns.html)无法通过它启用。这也是为什么他们分组到新的"模式"，而不是独立的一个一个发行。

你可以可选的基于每个子树上启用 Concurrent 模式。相反，为了插入，你需要在你调用`ReactDOM.render()`的地方这么做：

**这将会为整个<App />启用 Concurrent 模式。**

```jsx harmony
import ReactDOM from 'react-dom';

// If you previously had:
//
// ReactDOM.render(<App />, document.getElementById('root'));
//
// You can opt into Concurrent Mode by writing:

ReactDOM.createRoot(
  document.getElementById('root')
).render(<App />);
```
注意：
像`createRoot`之类的 Concurrent API 只存在在 React 的体验构建。

在 Concurrent 模式中，[前面标记](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)为"unsafe"的生命周期方法真的不安全，将导致比现在的 React 更多的 bug。我们不推荐尝试 Concurrent 模式，直到你的应用兼容[Strict 模式](https://reactjs.org/docs/strict-mode.html)。


### 有啥期待

如果你有一个巨大的存在的应用，或者你的应用依赖大量的第三方包，不要期待你可以立即使用 Concurrent 模式。**比如，在 Facebook 我们为新的网站使用 Concurrent 模式，但是我们没有在旧的网站启用的计划。** 这是因为我们旧的网站依旧使用不安全的生命周期方法，在产品代码中，不兼容的三方库，并且模式和 Concurrent 模式无法很好的工作。

### 升级步骤：阻塞模式

对于老的代码库，Concurrent 模式可能是很遥远的步骤。这也是为什么我们提供一个新的"堵塞模式"在体验版本构建。你可以尝试使用`createBlockingRoot`替代`createRoot`。它只提供一个 Concurrent 模式小的子集，但是它很接近 React 如今的工作原理，并且可以用于一系列的升级步骤。

回顾：

- Legacy Mode：`ReactDOM.render(<App />, rootNode)`。这是 React 应用如今使用的。在可预见的未来没有移除遗留模式的计划 -- 但是它无法支持这些新的功能。

- Blocking Mode：`ReactDOM.createBlockingRoot(rootNode).render(<App />)`。这当前是体验的。它的目标是作为应用获取 Concurrent 模式功能的第一个升级步骤。

- Concurrent Mode：`ReactDOM.creatRoot(rootNdoe).render(<App />)`。这当前是体验的。在未来，在它稳定以后，我们目标是将它变成默认模式。这个模式启用所有的新功能。

### 为什么这么多模式

我们觉得最好提供[优雅升级策略](https://reactjs.org/docs/faq-versioning.html#commitment-to-stability)，而不是创建巨大的断代改变 -- 或者让 React 停滞不前。

在实践中，我们期待现在大部分使用 Legacy 模式的应用可以至少升级到 Blocking 模式（如果不是 Concurrent 模式）。这种碎片化将会让想要在短时间内支持所有模式的库感觉烦恼。然而，逐步从 Legacy 模式中移除生态系统将会解决影响 React 生态系统，比如[当读取布局的时候迷惑 Suspense 的行为](https://github.com/facebook/react/issues/14536)和[缺少一致性的批量保证](https://github.com/facebook/react/issues/15080)。有一系列的 bug 无法在 Legacy 模式修复，如果不改变语义，但是不存在在 Blocking 和 Concurrent 模式。

你可以认为 Blocking 模式为 Concurrent 模式一个"优雅降级"版本。**作为结果，在长周期我们应该可以覆盖和停止思考其他不同的模式。** 但是现在，模式是重要的升级步骤。它让任何一个决定升级的时候值得，并且升级到他们自己的步骤。







