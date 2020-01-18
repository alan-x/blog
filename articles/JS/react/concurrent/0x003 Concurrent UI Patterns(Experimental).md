### Concurrent UI Patterns(Experimental)

注意：

这个页面描述了**稳定发布中[还不可用]()的实验性功能**。不要在生产 app 中依赖 React 的试验性构建。在成为 React 的一部分之前，这些功能可能发生非常大的改变，并且没有警告。

这个文档面向早期接受者和好奇的人。**如果你刚接触 React，不要单鞋这些功能** -- 你不需要立马学习他们。比如，如果你在寻找现在使用的数据获取指南，阅读[这篇文章]()。

通常，当我们更新状态，我们期待立即在屏幕上看到改变。这是有意义的，因为我们想要保持我们的应用响应用户输入。然而，有一些场景，我们可能偏向于**延迟一些更新出现在屏幕**。

比如，如果我们切换一个页面到另一个，下一个屏幕的代码或者数据还没有被加载，立即看到一个带加载指示器的空页面是令人沮丧的。我们可能更偏向于呆在前一个应用更长时间。在历史上，React 实现这个模式很难。Concurrent 模式提供一个新的工具集去做到它。

- [Transitions]()
    - [Wrapping setState in a Transition]()
    - [Adding a Pending Indicator]()
    - [Reviewing the Changes]()
    - [Where Does the Update Happen?]()
    - [Transitions Are Everywhere]()
    - [Baking Transitions Into the Design System]()

- [The Three Steps]()
    - [Default: Receded → Skeleton → Complete]()
    - [Preferred: Pending → Skeleton → Complete]()
    - [Wrap Lazy Features in <Suspense>]()
    - [Suspense Reveal “Train”]()
    - [Delaying a Pending Indicator]()
    - [Recap]()
    
- [Other Patterns]()
    - [Splitting High and Low Priority State]()
    - [Deferring a Value]()
    - [SuspenseList]()
    - [Next Steps]()

