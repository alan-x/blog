[原文地址](https://reactjs.org/docs/how-to-contribute.html)
### 怎样贡献

React 是 Facebook 首先开源的项目之一，在非常活跃的开发之下，用于向 [facebook.com](https://www.facebook.com/) 的每个人发送代码。我们依旧在处理问题，为让这个项目贡献尽可能简单和透明，但是我们还没到达这一步。希望这个文档让贡献流程更清晰并且回答一些你可能存在的问题。

### [行为准则](https://github.com/facebook/react/blob/master/CODE_OF_CONDUCT.md)

Facebook 接受 [Contributor Covenant](https://www.contributor-covenant.org/) 作为它的行为准则，并且我们期待项目成员坚持它。请阅读[完整文字](https://github.com/facebook/react/blob/master/CODE_OF_CONDUCT.md)，这样你可以理解什么行为可以接受和不能接受。

### Open Development
React 所有的工作都直接发生在[GitHub](https://github.com/facebook/react)。核心团队成员和外部贡献者发送 pull request 都经历相同的 review 流程。


### 语义化版本
React 遵循[语义化版本](https://semver.org/)。我们为重要 bug 修复发布修订版本，为新功能或者非必须改变发布次版本，为任何破坏性改变发布主版本。当我们创建破坏性版本，我们也在主版本引入废弃警告，这样我们的用户了解还没来的改变并且事先升级他们的代码。了解更多我们关于[版本策略](https://reactjs.org/docs/faq-versioning.html)稳定和增量升级的承诺。

每一个重要的改变都记录在 [changelog 文件](https://github.com/facebook/react/blob/master/CHANGELOG.md)。

### 分支管理

直接提交所有的改变到[主分支](https://github.com/facebook/react/tree/master)。我们不为开发或者即将到来的功能使用分离的分支。我们尽我们最大的努力保持`master`良好，让所有的测试通过。

`master`上的代码必须和最新的稳定发行兼容。它可能包含额外的特性，但是没有破坏性改变。我们应该随时可以在`master`的顶部可以发行一个新的副版本。

### 功能标志


为了让`master`分支保持在可发行状态，破坏性改变和试验性功能必须使用一个功能标志开关。

功能标志定义在[packages/shared/ReactFeatureFlags.js](https://github.com/facebook/react/blob/master/packages/shared/ReactFeatureFlags.js)。React 的一些构建可能开启不同的功能标志；比如，React Native 构建可能和 React DOM 配置不同。这些标志可以在[packages/shared/forks](https://github.com/facebook/react/tree/master/packages/shared/forks)找到。功能标志是通过 Flow 静态输入的，因此你可以执行`yarn flow`去确保你更新了所有必须的文件。

React 的构建系统在发布之前将去除关闭的功能分支。一个持续的交互工作在每一个提交执行，检查包大小的改变。你可以使用大小的改变作为一个功能是否正确开关的信号。

### bug

#### 哪里去找已知的问题

我们为公共 bug 使用[GitHub Issues](https://github.com/facebook/react/issues)。我们密切关注这个，并尝试去弄清楚，当我们有一个内部修复流程。在填写一个新任务之前，尝试确认你的问题还没存在。

#### 报告新问题

让你的 bug 修复最好的方式是提供一个精简的测试用例。这个[JSFiddle 模版]()是一个很好的开始。

#### 安全 bug

Facebook 有一个关于揭露安全问题的[悬赏计划](https://www.facebook.com/whitehat/)。考虑到这点，请不要提交公共问题；通过页面中的流程处理。

### 联系方式

- IRC：[freenode 上的 #reactjs](https://webchat.freenode.net/?channels=reactjs)
- [讨论论坛](https://reactjs.org/community/support.html#popular-discussion-forums)

同时也有[一个活跃的 React 用户社区在 Discord 聊天平台](https://www.reactiflux.com/)，防止你需要帮助。

### 提出一个改变

如果你想要改变公共 API，或者创建一些不重要的改变到实现，我们推荐[提交一个问题](https://github.com/facebook/react/issues/new)。这让你在投入重要努力之前和我们达成一直。

如果你只修复一个 bug，直接提交一个 pull request 也可以，但是我们依旧推荐提交一个你修复的问题详情给我们。这对防止我们不接受指定修复但是保持对这个问题的跟踪很有帮助。

### 你的第一个 pull request

正在处理你的第一个 pull request？你可以从下面这个免费的系列视频学习怎么做：

[怎么在 GitHub 贡献一个开源项目](https://egghead.io/series/how-to-contribute-to-an-open-source-project-on-github)

为了帮助你一步一步熟悉我们的贡献流程。我们有一个列表的[良好的第一个问题]()，包含相关有限范围的 bug。这是开始的好地方。

如果你决定修复这个问题，请确认检查过评论线程，防止其他人已经在处理这个修复。如果这时候没有人在处理，请留下一个你想要处理的评论，这样其他用户不会重复你的努力。

如果有些人提出一个问题，但是两周内没有跟进，处理它没问题，但是应该留下一个评论。

### 发送一个 pull request

核心团队监控 pull request。我们将审阅你的 pull request 并且合并它，请求改变它，或者使用解释关闭它。API 的改变我们需要修复我们在 Facebook.com 内部的使用，这可能导致一些延迟。我们将尽我们最大的努力去提供更新并通过流程反馈。

**在提交一个 pull request 之前**，请确保下面完成了：
1. fork [仓库](https://github.com/facebook/react)，并从`master`创建你的分支。

2. 执行`yarn`在你的仓库根。

3. 如果你修复一个 bug，或者添加应该被测试的代码，添加测试！

4. 确保测试套件通过（`yarn test`）。提示：`yarn test --watch TestName` 在开发中很有帮助。

5. 执行`yarn test-prod`去测试产品环境。它提供和`yarn test`一样的配置。

6. 如果你需要一个调试器，执行`yarn debug-test --watch TestName`，打开`chrome://inspect`，并点击"Inspect"。

7. 使用[prettier](https://github.com/prettier/prettier)格式化你的代码（`yarn prettier`）

8. 确保你的代码 lint（`yarn lint`）。提示：`yarn linc`只检测该拜年的文件。

9. 执行[Flow](https://flowtype.org/)类型检测（`yarn flow`）

10. 如果你没有做好，完成 CLA

### 贡献者许可协议（CLA）

为了接受你的 pull request，我们需要你提交一个 CLA。你只需要做一次，因此，如果你为其他的 
Facebook 开源项目完成了这个，你就不用做了。如果你第一次提交 pull request，只要让我们知道你已经完成了 CLA，并且我们可以使用你的 GitHub 用户名检查。

[在这里完成你的 CLA](https://code.facebook.com/cla)

### 贡献的先决条件

- 安装了[Node](https://nodejs.org/) v8.0.0+ 和[Yarn]() v1.2.0+。

- 安装了[JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)。

- 安装了`gcc`或者适当安装了需要的编译器。我们的一些依赖可能需要一些编译步骤。我们的一些依赖可能需要编译步骤，在 OS X，XCode 命令行工具将会覆盖这个。在 Ubuntu，`apt-get install build-essential`将会安装需要的包。类似的命令应该也可以在其他 Linux 执行。Window 需要一些额外的步骤，查看[node-gyp 安装指令](https://github.com/nodejs/node-gyp#installation)了解详情。

- 你对 Git 很熟悉。

### 开发工作流

克隆 React 之后，执行`yarn`去获取它的依赖，然后，你可以运行一系列命令：

- `yarn lint`检查代码风格。

- `yarn linc`和`yarn lint`很像，但是更快，因为它只检查你的分支中不同的文件。

- `yarn test`运行完整测试套件。

- `yarn test --watch`运行一个交互式测试监听器。

- `yarn test <pattern>`运行匹配的文件名的测试。

- `yarn test-prod`在生产环境运行测试。它和`yarn test`运行相同的选项。

- `yarn debug-test`和`yarn test`很像，但是有一个调试器。打开`chrome://inspect`并点击"inspect"。

- `yarn flow`执行[Flow](https://flowtype.org/)类型检查。

- `yarn build`使用所有的包创建`build`文件夹。

- `yarn build react/index,react-dom/index --type=UMD`只创建 React 和 ReactDOM 的 UMD 构建。

我们推荐运行`yarn test`(或者它的变体)，确保你没有在你的改变中引入任何变体。然而，在一个真实的项目中尝试你的构建是一个技巧。

首先，运行`yarn build`。这会在`build`文件夹生产预构建，同时在`build/packages`内准备好了 npm 包。

尝试你的改变最简单的方式是运行`yarn build react/index,react-dom/index --type=UMD`，然后打开`fixture/packaging/babel-standalone/dev.html`。这个文件已经使用`build`文件夹的`react.development.js`文件，因此它会使用你的改变。

如果你想要在你已经存在的 React 项目去尝试你的改变，你可以复制`build/dist/react.development.js`，`build/dist/react-dom.development.jd`，或者任何其他构建产品到你的应用，并使用他们体改稳定版本，如果你的项目使用 npm 的 React，你可能删除依赖中的`react`和`react-dom`，并使用`yarn link`去指出他们到你本地的`build`文件夹。

```jsx harmony
cd ~/path_to_your_react_clone/build/node_modules/react
yarn link
cd ~/path_to_your_react_clone/build/node_modules/react-dom
yarn link
cd /path/to/your/project
yarn link react react-dom
```

每一次你在 React 文件夹执行`yarn build`，更新的版本将会出现在你项目的`node_modules`。你可以宠幸构建你的项目并尝试改变。

我们依旧需要你的 pull request 为任何新的功能包含单元测试。这个方式，我们可以确保我们不会在未来破坏你的代码。

### 风格指南

我们使用自动的代码格式化，叫做[Prettier](https://prettier.io/)。在对代码做任何改变之后执行`yarn prettier`。

然后，我们的 linter 将会捕获大部分存在在你的代码中的问题。你可以通过简单运行`yarn linc`检查你的代码风格的状态。

然而，依旧有一些风格 linter 无法检查。如果你对一些东西不确定，查阅[Airbnb 的风格指南](https://github.com/airbnb/javascript)，将会指导你到正确的方向。

### 征求意见（RFC）

很多改变，包括 bug 修复和文档改善，可以通过普通的 GitHub pull request 工作流实现和审阅。

一些改变被认为是"重大的"，我们需要让这些经过一些设计过程，并在 React 核心团队达成一致。

"RFC"（请求评议）流程的目的是为新功能进入项目提供一致和可控的路径。你可以通过浏览[rfcs 仓库](https://github.com/reactjs/rfcs)贡献。

### 许可证

通过为 React 贡献，你同意你的贡献在 MIT 协议之下。

### 下一步是什么？
阅读[下一个章节](https://reactjs.org/docs/codebase-overview.html)，学习关于代码库是如何组织的。
