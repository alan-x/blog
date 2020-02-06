[原文地址](https://reactjs.org/docs/create-a-new-react-app.html)
### 创建一个新的 React 应用

使用一个集成的工具链可以得到最好的用户和开发体验。

这个页面描述了一些流行的 React 工具链，可以在以下的任务帮助你：

- 压缩很多文件和组件

- 使用 npm 的第三方库

- 尽早捕捉常见错误

- 在开发者模式实时编辑 CSS 和 JS

- 为生产环境优化输出

这个页面推荐的工具链**不需要配置就可以开始。**

### 你可能不需要一个工具链

如果你没有遇见过前面的问题或者对使用 JavaScript 工具不舒服，考虑[添加 React 为一个简单的<script>标签到 HTML 页面中](https://reactjs.org/docs/add-react-to-a-website.html)，可选的[使用 JSX](https://reactjs.org/docs/add-react-to-a-website.html)。

这也是**集成 React 到一个已存在的网站最简单的方式**。你总是可以添加一个巨大的工具链，如果你发现这很有帮助。

### 推荐工具链

React 团队主要推荐这些解决方案：

- 如果你在**学习 React**或者创建一个新的[单页应用](https://reactjs.org/docs/glossary.html#single-page-application)，使用[Create React App](https://reactjs.org/docs/create-a-new-react-app.html#create-react-app)

- 如果你在创建一个**使用 Node.js 的服务端渲染网站**，尝试[Next.js](https://reactjs.org/docs/create-a-new-react-app.html#nextjs)

- 如果你在创建一个**静态面向内容的网站**，尝试[Gatsby](https://reactjs.org/docs/create-a-new-react-app.html#gatsby)

- 如果你在创建一个**组件库**或者**集成到已有代码库**，尝试[更多有弹性的工具链](https://reactjs.org/docs/create-a-new-react-app.html#more-flexible-toolchains)。

### 创建 React 应用

[Create React App](https://github.com/facebookincubator/create-react-app)是**学习 React** 非常好的环境，并且也是开始构建**[一个新的单页应用](https://reactjs.org/docs/glossary.html#single-page-application)**的最好的方式。

它设置你的开发环境，因此你可以使用最新的 JavaScript 功能，提供一个非常好的开发者体验，并为生产环境优化你的应用。你将需要 Node >= 8.10 和 npm >=5.6 在你的设备。为了创建一个项目，执行：

```jsx harmony
npx create-react-app my-app
cd my-app
npm start
```
提示：第一行的`npx`不是一个编写错误 -- 它是[包运行工具，在 npm 5.2+ 引入](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)。

创建 React 应用不处理后端逻辑或者数据库；它只创建一个前端构建流程，因此你可以和任何你想要的后端一起使用。在这个前提下，它使用[Babel](https://babeljs.io/)和[Webpack](https://webpack.js.org/)，但是你不需要知道关于他们的任何东西。

当你准备部署到生产环境，执行`npm run build`将会创建一个优化好的构建好的你的应用在你的`build`文件夹。你可以[从它的 README](https://github.com/facebookincubator/create-react-app#create-react-app--)和[用户指南](https://facebook.github.io/create-react-app/)了解到关于创建 React 应用的资料。

### Next.js

[Next.js](https://nextjs.org/)对于使用 React 构建**静态和服务端渲染应用**来说，是一个流行且轻量的框架。它包含**样式和路由解决方案**超出了范围，假设你使用[Node.js](https://nodejs.org/)作为服务端环境。

从[它的官方指南](https://nextjs.org/learn/)学习 Next.js。

### Gatsby

[Gatsby](https://www.gatsbyjs.org/) 是创建使用 React 创建**静态网站**最好的方式。它让你使用 React 组件，但是输出预渲染的 HTML 和 CSS 去保证最快的加载时间。

从[它的官方指南](https://www.gatsbyjs.org/docs/)和[启动器工具画廊](https://www.gatsbyjs.org/docs/gatsby-starters/)学习 Gatsby。

### 更多灵活的工具链

下面的工具链提供更灵活和选择。我们推荐他们给更有经验的用户：

- [Neutrino](https://neutrinojs.org/)结合[webpack](https://webpack.js.org/)的威力，使用简单的 preset，为 [React 应用](https://neutrinojs.org/packages/react/)和[React 组件](https://neutrinojs.org/packages/react-components/)包含一个 preset。

- [Parcel](https://parceljs.org/)是一个快速的，零配置的，和 [React 一起用](https://parceljs.org/recipes.html#react)的网页应用构建器。

- [Razzle](https://github.com/jaredpalmer/razzle)是一个服务端渲染框架，不需要任何配置，但是比 Next.js 更灵活。

### 从头开始创建工具链

一个 JavaScript 构建工具通常包含：

- 一个**包管理器**，比如[Yarn](https://yarnpkg.com/)和[npm](https://www.npmjs.com/)。它让你充分利用第三方包的生态环境，并简单的安装和更新他们。

- 一个**构建器**，比如[webpack](https://webpack.js.org/)或[Parcel](https://parceljs.org/)。它让你编写模块化代码并打包它一起到小的包去优化加载时间。

- 一个**编译器**，比如[Babel](https://babeljs.io/)。它让你编写模块化 JavaScript 代码，并能在旧的浏览器工作。

如果你偏向于从零设置你自己的 JavaScript 工具链，[查阅这个指南](https://blog.usejournal.com/creating-a-react-app-from-scratch-f3c693b84658)重新创建一些 Create React App 的功能。

不要忘记确保你自定义的工具链[正确为产品环境设置好了](https://reactjs.org/docs/optimizing-performance.html#use-the-production-build)。












