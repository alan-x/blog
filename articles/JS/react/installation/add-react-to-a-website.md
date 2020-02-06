[原文地址](https://reactjs.org/docs/add-react-to-a-website.html)

### 添加 React 到一个网站

按照你需要的来添加 React 的多少

React 被设计为渐近引入，**你可以按照你的需要，或多或少的引入 React**。可能你只要去添加一些"可交互的洒水车"到已存在的页面。React 组件是做这个的好方法。

大部分的网站不是并且不需要是单页面应用。在你的网站小部分尝试 React，**不需要太多的代码和构建工具**。你可以逐渐扩展它的存在，或者保持它包含在一些动态的组件中。

### 一分钟加入 React

在这个章节，我们将展示怎样添加一个 React 组件到已存在的 HTML 页面。你可以使用你自己的网站，或者创建一个空的 HTML 文件去练习。

没有复杂的工具或者安装需要 -- **完成这个章节，你只需要一个网络链接，和你的一分钟时间。**

可选：[下载完整例子（压缩后 2KB）](https://gist.github.com/gaearon/6668a1f6986742109c00a581ce704605/archive/f6c882b6ae18bde42dcf6fdb751aae93495a2275.zip)

### 步骤1：添加 DOM 容器到 HTML

首先，打开你要编辑的 HTML 页面。添加空的`<div>`标签去标记你要使用 React 显示一些东西的点。比如：
```jsx harmony
<!-- ... existing HTML ... -->

<div id="like_button_container"></div>

<!-- ... existing HTML ... -->
```

我们给这个`<div>`一个唯一的`id` HTML 属性。这将允许我们之后在 JavaScript 代码中找到它并在它内部显示一个 React 组件。

提示：你可以放置一个像这样的"容器"`<div>`在你的`<body>`标签内部。你可以按照你需要的放置多个容器在你的一个页面中。他们通常是空的 -- React 将替换任何 DOM 容器内已经存在的内容。

### 步骤2：添加脚本标签

下一步，添加三个`<script>`标签到 HTML 页面的`<body>`标签之后：
```jsx harmony
 <!-- ... other HTML ... -->

  <!-- Load React. -->
  <!-- Note: when deploying, replace "development.js" with "production.min.js". -->
  <script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script>
  <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>

  <!-- Load our React component. -->
  <script src="like_button.js"></script>

</body>
```
前两个加载 React，第三个加载你的组件代码

### 步骤3：创建 React 组件

创建一个叫做`like_button.js`到你的 HTML 页面后面。

打开[这个启动代码](https://gist.github.com/gaearon/0b180827c190fe4fd98b4c7f570ea4a8/raw/b9157ce933c79a4559d2aa9ff3372668cce48de7/LikeButton.js)并复制到你创建的文件。

提示：这个代码定义了一个 React 组件，叫做`LikeButton`。不要担心如果你不理解它 -- 我们将在我们的[实践教程](https://reactjs.org/tutorial/tutorial.html)和[主要概念指南](https://reactjs.org/docs/hello-world.html)中覆盖构建的 React 块。现在，只是让他显示在屏幕上！

在[启动器代码](https://gist.github.com/gaearon/0b180827c190fe4fd98b4c7f570ea4a8/raw/b9157ce933c79a4559d2aa9ff3372668cce48de7/LikeButton.js)之后，添加两行到`like_button.js`的结尾：
```jsx harmony
// ... the starter code you pasted ...

const domContainer = document.querySelector('#like_button_container');
ReactDOM.render(e(LikeButton), domContainer);
```
这两行的代码找到我们第一步添加到 HTML 的`<div>`，然后在内部显示我们的"Like"按钮 React 组件。

### 这就完成了

没有第四步。**你已经添加了第一个 React 组件到你的网站。**

查阅下一个章节了解更多关于 React 集成的提示

[查看完整源代码](https://gist.github.com/gaearon/6668a1f6986742109c00a581ce704605)

[下载完整例子（压缩后 2KB）](https://gist.github.com/gaearon/6668a1f6986742109c00a581ce704605/archive/f6c882b6ae18bde42dcf6fdb751aae93495a2275.zip)

### 注意：重用一个组件
通常，你可能想要在 HTML 页面的多个地方显示 React 组件。这是一个显示"Like"按钮 3 次并传递一些数据给它的例子：

[查看完整源代码](https://gist.github.com/gaearon/faa67b76a6c47adbab04f739cba7ceda)

[下载完整例子（压缩后 2KB）](https://gist.github.com/gaearon/faa67b76a6c47adbab04f739cba7ceda/archive/9d0dd0ee941fea05fd1357502e5aa348abb84c12.zip)

注意：这个策略只有当 React 驱动一个页面的部分，并且相互独立的时候有用。在 React 代码内部，使用[组件合成](https://reactjs.org/docs/components-and-props.html#composing-components)替代更简单。

### 注意：为生产环境压缩 JavaScript

在部署你的网站到生产环境之前，注意未压缩的 JavaScript 会大大降低用户的页面速度。

如果你已经压缩了应用脚本，如果你确保部署的 HTML 加载的 React 版本以`production.min.js`结尾，**你的网站就可以投入生产**：

 ```jsx harmony
<script src="https://unpkg.com/react@16/umd/react.production.min.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js" crossorigin></script>
```
如果你没有对你的脚本执行压缩步骤，[这里有一个方式去设置它](https://gist.github.com/gaearon/42a2ffa41b8319948f9be4076286e1f3)。

### 可选：使用 JSX 尝试 React

在前面的例子中，我们只依赖了浏览器原生支持的功能。这是为什么我们使用一个 JavaScript 函数调用去告诉 React 显示什么：
```jsx harmony
const e = React.createElement;

// Display a "Like" <button>
return e(
  'button',
  { onClick: () => this.setState({ liked: true }) },
  'Like'
);
```

然而，React 也提供了一个可选的 [JSX](https://reactjs.org/docs/introducing-jsx.html) 替代：
```jsx harmony
// Display a "Like" <button>
return (
  <button onClick={() => this.setState({ liked: true })}>
    Like
  </button>
);
```

这两个代码脚本都是相等的。尽管 **JSX 是[完全可选的](https://reactjs.org/docs/react-without-jsx.html)**，很多人发现它对写 UI 代码很有帮助 -- 不管是使用 React 还是其他库。

你可以使用[这个在线转化器](https://babeljs.io/en/repl#?babili=false&browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=DwIwrgLhD2B2AEcDCAbAlgYwNYF4DeAFAJTw4B88EAFmgM4B0tAphAMoQCGETBe86WJgBMAXJQBOYJvAC-RGWQBQ8FfAAyaQYuAB6cFDhkgA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=es2015%2Creact%2Cstage-2&prettier=false&targets=&version=7.4.3)去玩玩 JSX。

### 快速尝试 JSX

在你的项目中尝试 JSX 最快的方式是添加这个`<script>`标签到你的页面：
```jsx harmony
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
```

现在你可以使用 JSX 在任何`<script>`标签，通过添加`type="text/babel"`属性到它之上。这是一个[HTML 文件使用 JSX 的例子](https://raw.githubusercontent.com/reactjs/reactjs.org/master/static/html/single-file-example.html)，你可以下载并试试。

这个方式对于学习和创建简单例子很好。然而，它让你的网站变慢，而且**不适合生产环境**。当你准备继续的时候，移除你添加的这个新的`<script>`标签和`type="text/babel"`属性。相反，在下一个章节你将设置一个 JSX 预处理器去自动转化你的`<script>`标签。


### 添加 JSX 到项目

添加 JSX 到一个项目不需要复杂的工具，比如一个构建器或者一个开发服务器。基本上，添加**JSX 不像添加一个 CSS 预处理器。** 只需要你在你的电脑安装了[Node.js](https://nodejs.org/)。

在终端进入你的项目文件夹，并张贴这两个命令：

1. 步骤1：执行`npm init -y`（如果失败，[这有一个修复](https://gist.github.com/gaearon/246f6380610e262f8a648e3e51cad40d)）

2. 步骤2：执行`npm install babel-cli@6 babel-preset-react-app@3`

提示：我们**这里使用 npm 只是安装 JSX 预处理器；** 你不需要其他东西。React 和应用代码可以呆在`<script>`标签中，不需要任何改变。

恭喜！你添加了一个**可用于生产环境的 JSX 设置** 到你的项目。

### 运行 JSX 预处理器

创建一个文件夹叫做`src`并运行这个终端命令：
```jsx harmony
npx babel --watch src --out-dir . --presets react-app/prod 
```
注意：`npx`不是一个错误 -- 它是一个[包运行工具，从 npm 5.2+ 引入]()。

如果你看到一个错误消息说：你安装`babel`包失败，你可能跳过了[前一个步骤]()。在相同目录执行它，然后再次尝试。

不要等他完成 -- 这个命令启动了一个 JSX 的自动监听器。

如果你现在使用[JSX 启动器代码](JSX starter code)创建一个文件叫做`src/like_button.js`，监听器将会使用普通 JavaScript 代码创建一个适合你浏览器的预处理的`like_button.js`。当你使用 JSX 编辑源文件，转化将会再次执行。

作为奖励，这也让你使用现代 JavaScript 语法特性，比如类，不需要担心破坏旧浏览器。我们刚刚使用的工具叫做 Babel，你可以从[它的文档](https://babeljs.io/docs/en/babel-cli/)学到更多。

如果你注意到你开始适应构建工具，并且想要他们为你做更多，[下一个章节](https://reactjs.org/docs/create-a-new-react-app.html)描述一些最流行并且可实现的工具链。如果不 -- 这些脚本标签也能做的很好。













