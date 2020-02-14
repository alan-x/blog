[原文地址](https://reactjs.org/docs/codebase-overview.html)
### Codebase Overview

这个章节将给你一个 React 代码库组织的概述，它的惯例，和实现。

如果你想要[贡献 React](https://reactjs.org/docs/how-to-contribute.html)，我们希望这个指南将会帮助你在创建改变的时候更加舒适。

我们不需要在 React 应用中推荐这些约定。他们很多都是因为历史原因存在，并且可能随着时间改变。

### 外部依赖

React 基本没有额外的依赖。通常，一个`require()`指向 React 自己的代码库文件。然而，有一些罕见的例外。

[fbjs](https://github.com/facebook/fbjs) 库存在是因为 React 和一些类似 [Relay](https://github.com/facebook/relay) 的库共享一些小工具，我们保持他们同步。我们不依赖 Node 生态中的相同的小模块，因为我们想要 Facebook 工程师可以随时改变他们。fbjs 内的任何工具都不会考虑变成公共 API，他们只用于 Facebook 项目，比如 React。

### 顶级文件夹

在克隆[React 仓库](https://github.com/facebook/react) 之后，你将在仓库内看到一些顶级文件夹：

- [packages](https://github.com/facebook/react/tree/master/packages) 包含元数据（比如`package.json`），和 React 仓库内的所有包的源代码（`src`子文件夹）。**如果你的改变和代码相关，src 每一个包的子文件夹是你花费最多时间的地方。**

- [fixtures](https://github.com/facebook/react/tree/master/fixtures) 包含一些为贡献者准备的小的 React 测试应用。

-`build`是 React 的构建输出。它不在仓库中，但是它将会出现在你的 React 克隆，在你第一次[构建它](https://reactjs.org/docs/how-to-contribute.html#development-workflow)的时候。

文档部署在[从 React 分离出来的仓库](https://github.com/reactjs/reactjs.org)。

还有一些其他顶级文件夹，但是大部分用于工具，并且你贡献的时候可能将永远不会遇到他们。

### Colocated Tests
我们没有为单元测试设置顶级文件夹。相反，我们将他们放到和他们测试的文件接近的一个叫做`__tests__`的文件夹。

比如，[setInnerHTML.js](https://github.com/facebook/react/blob/87724bd87506325fcaf2648c70fc1f43411a87be/src/renderers/dom/client/utils/setInnerHTML.js) 的测试位于[__tests__/setInnerHTML-test.js](https://github.com/facebook/react/blob/87724bd87506325fcaf2648c70fc1f43411a87be/src/renderers/dom/client/utils/__tests__/setInnerHTML-test.js)附近。

### Warning and Invariants

React 代码库使用`warning`模块去显示警告：
```jsx harmony
var warning = require('warning');

warning(
  2 + 2 === 4,
  'Math is not working today.'
);
```
**警告将在警告条件为 false 的时候。**

一种思考方式是反映普通场景而不是异常。

避免大量向控制台发送重复警告是一个好主意：
```jsx harmony
var warning = require('warning');

var didWarnAboutMath = false;
if (!didWarnAboutMath) {
  warning(
    2 + 2 === 4,
    'Math is not working today.'
  );
  didWarnAboutMath = true;
}
```

警告只有在开发环境启用。在生产环境，他们完全移除。如果你需要去禁止一些代码路径执行，使用`invariant`模块替代：
```jsx harmony
var invariant = require('invariant');

invariant(
  2 + 2 === 4,
  'You shall not pass!'
);
```

invariant 将在 invariant 条件为 false 的时候抛出。

"invariant"是说明"这个条件总是 true"的一种方式。你可以可以认为它创建了一个断言。

保持开发和生产行为像是很重要，因此`invariant`在开发和生产都抛出。错误信息在生产环境自动替换为错误码，避免对字节大小的负面影响。

### 开发和生产

你可以在代码库中使用`__DEV__`伪全局变量去确保代码库只在开发生效。

它在编译步骤内联化，并且在 CommonJS 构建转化为`process.env.NODE_ENV !== 'production'`检查。

单独构建的时候，他在未压缩的构建是`true`，并且使用`if`块在确保压缩构建完全移除。

```jsx harmony
if (__DEV__) {
  // This code will only run in development.
}
```

### Flow
我们最近引入 [Flow](https://flow.org/) 检查代码库。使用`@flow`标注的头部注释表示使用类型检查。

我们接受 pull request [添加 Flow 标注存在的代码](https://github.com/facebook/react/pull/7600/files)。Flow 声明看起来像这样：
```jsx harmony
ReactRef.detachRefs = function(
  instance: ReactInstance,
  element: ReactElement | string | number | null | false,
): void {
  // ...
}
```
当可能的时候，新代码应该使用 Flow 声明。你可以执行`yarn flow`本地去使用 Flow 检查你的代码。


### 动态注入

React 在一些模块使用动态注入。尽管他总是明确的，它依旧是不幸的，因为它妨碍了对代码的理解。它存在的主要原因是因为 React 原来只支持 DOM 作为目标。React Native 开始是 React 的一个 fork。我们需要去添加动态注入让 React Native 覆盖一些行为。

你可能看到模块声明他们的动态依赖是这样的：
```jsx harmony
// Dynamically injected
var textComponentClass = null;

// Relies on dynamically injected value
function createInstanceForText(text) {
  return new textComponentClass(text);
}

var ReactHostComponent = {
  createInstanceForText,

  // Provides an opportunity for dynamic injection
  injection: {
    injectTextComponentClass: function(componentClass) {
      textComponentClass = componentClass;
    },
  },
};

module.exports = ReactHostComponent;
```
`injection`域没有做任何特殊处理。但是为了方便，它意味着这个模块想要有一些（假设是平台指定）依赖在运行时注入。

在代码库中有多个注入。在未来，我们目标是去掉所有动态依赖机制并且在构建时静态连接所有块。

### 多个包

React 是一个 [monorepo](https://danluu.com/monorepo/)。它的仓库包含多个分离的包，这样他们的改变可以同时一起，并且 issue 在同一个地方。

### React Core
React 的"核心"包含所有的 [React 顶级 API](https://reactjs.org/docs/top-level-api.html#react)，比如：

- `React.createElement()`

- `React.Component`

- `React.Children`

**React 核心只包含定义组件需要的 API。** 它不包含 [reconciliation](https://reactjs.org/docs/reconciliation.html) 算法或者任何平台指定算法。它被 React DOM 和 React Native 组件使用。

React 核心位于代码树的 [packages/react](https://github.com/facebook/react/tree/master/packages/react)。它作为 [react](https://www.npmjs.com/package/react) 包在 npm 上可用。 对应的单独的浏览器构建叫做`react.js`，它导出一个叫做`React`的全局。

### Renderer

React 起初是为 DOM 创建的，但是它之后也使用 [React Native](https://facebook.github.io/react-native/) 支持原生平台。这引入了"渲染器"的概念到 React 内部。

**渲染器管理 React 树如何转化为底层平台调用。**

渲染器也位于 [packages/](https://github.com/facebook/react/tree/master/packages/)：

- [React DOM 渲染器](https://github.com/facebook/react/tree/master/packages/react-dom)渲染 React 组件到 DOM。它实现[顶级 ReactDOM API](https://reactjs.org/docs/react-dom.html)，作为 [react-dom](https://www.npmjs.com/package/react-dom) npm 包可用。它也作为单独的浏览器构建使用，叫做 `react-dom.js`，导出一个`ReactDOM`全局。

- [React Native 渲染器](https://github.com/facebook/react/tree/master/packages/react-native-renderer) 渲染 React 组件到原生视图。它在 React Native 内部使用。

- [React Test 渲染器](https://github.com/facebook/react/tree/master/packages/react-test-renderer) 渲染 React 组件到 JSON 树。它用于 [Jest](https://facebook.github.io/jest) [Snapshot 测试](https://facebook.github.io/jest/blog/2016/07/27/jest-14.html)功能，并且作为[react-test-renderer](https://www.npmjs.com/package/react-test-renderer) npm 包使用。

其他官方支持的渲染器还有 [react-art](https://github.com/facebook/react/tree/master/packages/react-art)。它用于分离的 [GitHub 仓库](https://github.com/reactjs/react-art)，但是现在我们移动它到主代码树。

注意：技术上，[react-native-renderer]() 是非常小的层，告诉 React 和 React Native 实现交互。真实的平台指定代码视图存在在 [React Native 仓库]()，和它的组件一起。

### Reconcilers

尽管像 React DOM 和 React Native 这样差别很大的渲染器需要分享大量的逻辑。特别是，[reconciliation](https://reactjs.org/docs/reconciliation.html) 算法应该尽可能类似，这样声明式渲染，自定义组件，状态，生命周期方法，和 refs 跨平台保持一致。

为了解决这个，不同渲染器在他们之间分享不同代码。我们吧 React 的这部分叫做一个"reconciler"。当一个更新，比如`setState()`被调用，reconciler 调用树中组件的`render()`并挂载，更新，或者卸载他们。

reconciler 没有独立分包，因为他们当前没有公共 API。相反，他们只被类似 React DOM 和 React Native 之类的渲染器使用。

### Stack Reconciler

"stack" reconciler 是 React 15 和更早的实现。我们开始停止使用它，但是它在 [下一个章节](https://reactjs.org/docs/implementation-notes.html) 详细记录。

### Fiber Reconciler

"fiber" reconciler 是一个新的尝试，为了解决从 stack reconciler 继承来的问题，并修复一些长期存在的问题。它从 React 16 开始成为默认 reconciler。

它的主要目标是：

- 将可中断工作分割到块的能力。

- 在流程中优先级，变基和重用工作的能力。

- 在父亲和子孙之后来回跳转在 React 中支持布局的能力。

- 从`render()`中返回多个元素的能力。

- 更好的支持错误边界。

你可以在[这里](https://github.com/acdlite/react-fiber-architecture)和[这里](https://github.com/acdlite/react-fiber-architecture)了解更多关于 React Fiber 架构。当你使用 React 16 ，默认情况下没有启用异步特性。

它的源代码位于[packages/react-reconciler](https://github.com/facebook/react/tree/master/packages/react-reconciler)

### 事件系统

React 实现一个合成事件系统，它和渲染器和 React DOM 和 React Native 无关。它的代码位于[packages/legacy-events](https://github.com/facebook/react/tree/master/packages/legacy-events)。

这是一个[深入它的视频](https://www.youtube.com/watch?v=dRo_egw7tBc)（66 分钟）。

### 下一步是啥？

阅读[下一章节](https://reactjs.org/docs/implementation-notes.html)了解更多关于 pre-React 16 实现的 reconciler。我们还没有详细记录内部的新 reconciler。














