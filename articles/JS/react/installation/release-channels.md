### 发布通道

React 依赖于繁荣的开源社区去提交 bug 报告，提交 pull request，和[提交 RFC](https://github.com/reactjs/rfcs)。为了鼓励反馈，我们有时候分享特定的包含未发布的功能 React 构建。

这个文档主要面向框架，库，或者开发工具的开发者。使用 React 的开发者主要用于构建面向用户的应用不需要担心我们的预发行通道。

每一个 React 的发行通道都是为了单独的用户场景：

- [Latest]()是为了稳定，语义化的 React 发行。这是你从 npm 安装 React 时候得到的。这是你现在使用的通道。**为所有面向用户的 React 应用使用这个。**

- [Next]()跟踪 React 源代码仓库的主分支。把这个认为下一个副语义发行的候选发行。

- [Experimental]()包含体验性的 API和在稳定发行不可用的特性。这也跟踪主分支，但是有些额外的功能标志打开了。使用这个在发行之前尝试还未到来的功能。

所有的发行都发布到 npm，但是只有 Latest 使用[语义化版本](https://reactjs.org/docs/faq-versioning.html)。预发行（在 Next 和 Experimental 通道）有一个从他们内容生成的哈希作为版本，比如为 Next 生成的`0.0.0-1022ee0ec`，和为 Experimental 生成的`0.0.0-experimental-1022ee0ec`。

**唯一官方支持的面向用户应用的发行通道是 Latest**。Next 和 Experimental 发行都是只为测试目的提供的，并且我们不保证行为在发行之间不会改变。他们不遵循我们为 Latest 使用的语义化协议。

通过发布预发行到和稳定发行相同的仓库，我们可以充分利用很多支持 npm 工作流的工具，比如[unpkg](https://unpkg.com/)和[CodeSandbox](https://codesandbox.io/)。

### Latest 通道

Latest 是用于稳定的 React 发行的通道。它表示 npm 中的`latest`标签。这是所有面向真实用户的 React 应用的推荐通道。

**如果你不确定哪一个渠道你应该使用，那就是 Latest。** 如果你是一个 React 开发者，这就是你正在使用的。

你可以认为 Latest 更新是最稳定的。遵循语义化方案的版本。在我们的[版本策略](https://reactjs.org/docs/faq-versioning.html)了解更多关于我们提交稳定性和增量升级相关信息。

### Next 通道

Next 通道是一个预发行通道，跟踪 React 仓库的主分支。我们在 Next 使用预发行作为 Latest 的发行候选者通道。你可以认为 Next 是 Latest 的更新更为频繁的超集。最新的 Next 发行和最新的 Latest 发行之间的差距大概和两个副语义发行之间的差异差不多。然而，**Next 通道不符合语义化版本**。 你应该期待在成功的 Next 通道发行之间可能存在破坏性改变。

**不要在面向用户的应用使用预发行。**

发行在 Next 的在 npm 中使用`next`标签。版本号从构建的内容哈希生成，比如，`0.0.0-1022ee0ec`。

### 使用 Next 通道做集成测试

Next 通道是用于支持 React 和其他项目的集成测试。

React 所有的改变在发布到公共之前都经过非常多个内部测试。然而，有无数的环境和配置在 React 生态中使用，对于我们，测试所有的情况是不可能的。

如果你是 React 的第三方框架，库，开发工具，或者类似的基础设施类型项目的作者，你可以帮助我们保证 React 对你的用户和整个 React 社区是稳定的，通过周期性的对最新改变执行你的测试套件。如果你有兴趣，使用下列步骤：

- 使用你喜欢的持续集成平台设置一个定时认为。[CicleCI](https://circleci.com/docs/2.0/triggers/#scheduled-builds)和[Travis CI](https://docs.travis-ci.com/user/cron-jobs/)都支持定时任务。

- 在定时任务内，更新你的 React 包到最新的 Next 通道中的 React 发行，在 npm 使用`next`标签。使用 npm cli：
```jsx harmony
npm update react@next react-dom@next
```
或者 yarn：
```jsx harmony
yarn upgrade react@next react-dom@next
```

- 对更新的包执行你的测试套件。

- 如果所以东西都通过了，很好！你可以认为你的项目可以用于下一个 React 发行副版本

- 如果有些东西被意外破坏了，请让我们知道，通过[提交一个 issue](https://github.com/facebook/react/issues)。

一个使用这个工作流的项目是 Next.js。（不是开玩笑，说真的）—— 你可以查阅他们的 [CycleCI 配置作为一个例子](https://github.com/zeit/next.js/blob/c0a1c0f93966fe33edd93fb53e5fafb0dcd80a9e/.circleci/config.yml)

### Experimental 通道

像 Next，Experimental 通道是一个预发行通道，跟踪 React 仓库的主分支。不像 Next，Experimental 发行包含额外的特性并且 API 还没为广泛发行做好准备。

通常，一个到 Next 的更新有一个对应的更新到 Experimental。他们基于相同的源版本，但是使用不同集合的特性标识构建。

Experimental 发行可能和 Next 和 Latest 发行有非常大的不同。**不要使用 Experimental 发行在一个面向用户的应用**。你应认为待在 Experimental 渠道之间经常有破坏性改变。

Experimental 的发行通常在 npm 中使用`experimental`标签发行。版本是从构建的内容生成的哈希，比如，` 0.0.0-experimental-1022ee0ec`。

### 什么是 Experimental 发行

Experimental 功能是还没准备好广泛公开的特性，并且可能在完成之前彻底改变。一些体验可能永远不会完成 -- 我们进行实验的目的是测试提案改变的存活性。

比如，当我们宣布 Hooks 的时候，如果 Experimental 渠道已经存在了。在我们在 Latest 可用的时候，我们已经发布 Hooks 到 Experimental 渠道好几周了。

 你会发现对 Experimental 执行集成测试是非常有用的。这取决于你。然而，注意 Experimental 比 Next 不稳定。**我们在 Experimental 发行之间不保证任何稳定性**。

### 怎样学到更多关于 Experimental 功能

Experimental 功能可能没有文档。通常体验没有文档直到他们接近 Next 或者 Stable。

如果一个功能没有文档，他们可能有 [RFC](https://github.com/reactjs/rfcs)。

当你我们去宣布新的体验，我们将会发表到[React 博客](https://reactjs.org/blog)，但这不意味着我们将发布每一个实验。

你总是可以查阅我们的公共 GitHub 仓库的[历史](https://github.com/facebook/react/commits/master)了解全部的改变列表。


