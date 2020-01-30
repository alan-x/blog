### 虚拟 DOM 和内部

### 什么是虚拟 DOM？

虚拟 DOM（VDOM）是编程理念，理想，或者"虚拟"，表示 UI 在内存的表示，并同步到"真实" DOM，通过像 ReactDOM 之类的库。这个过程叫做 [reconciliation]()。

这个方式启用了 React 的声明式 API：你告诉 React 你想要 UI 是什么状态，它保证 DOM 和状态匹配。这个抽象出了状态操作，事件处理，和手动 DOM 更新，否则你必须用这些来构建你的应用。

因为"虚拟 DOM"更像是一个模式，而不是一个特定技术，人们有时候说他是不同的东西。在 React 世界，术语"虚拟 DOM"通常和 [React elements]()联系，因为他们表示用户接口的对象。React，然而，也适用内部对象，叫做"fiber"去持有关于组件树的额外信息。他们也可以认为是 React "虚拟 DOM"的一部分。

### Shadow DOM 和虚拟 DOM 是一样的吗？

不，他们不同。Shadow DOM 是浏览器技术，主要是为了 web 组件的变量和 CSS 的范围。虚拟 DOM 是一个概念，基于浏览器 API 之上，使用 JavaScript 库实现的。

### 什么是"React Fiber"？
Fiber 是 React 16 的新 reconciliation 引擎。它的目标是启用虚拟 DOM 的增量渲染。[了解更多]()
