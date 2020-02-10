[原文地址（未完结）](https://reactjs.org/docs/optimizing-performance.html)
### Optimizing Performance

内部，React 使用一系列灵活的技术去最小化更新 UI 需要的 DOM 操作的损耗。对于很多应用程序，使用 React 会导致一个快速的用户接口，而不需要做太多的工作去优化性能。然而，有一些方式可以让你加速你的 React 应用。

### 使用生产构建

如果你在为你的 React 应用做基准测试或者遇见性能问题，确保你使用的是压缩过的生产构建。

通常，React 包含很多有帮助的警告。这些警告在开发的时候非常有用。然而，他们让 React 又大又慢，因此当你部署你的应用的时候，确保你使用生产版本。

如果你不确保你的构建进程正确设置，你可以通过安装[ Chrome React 开发者工具](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)。如果你访问一个使用 React 产品模式的网站，图标将会有一个暗色背景。

![](https://reactjs.org/static/devtools-prod-d0f767f80866431ccdec18f200ca58f1-1e9b4.png)

如果你访问一个使用 React 开发模式的网站，图标将会是红色背景。

![](https://reactjs.org/static/devtools-dev-e434ce2f7e64f63e597edf03f4465694-1e9b4.png)

当你开发你的应用的时候，希望你使用开发者模式，当你部署你的 app 给用户的时候，希望你使用产品模式。

你可以在下面找到使用产品构建你的应用的指令。

### Create React App

如果你的项目使用[Create React App](https://github.com/facebookincubator/create-react-app)，执行：
```jsx harmony
npm run build
```
这将会在你的项目的`build/`文件夹下为你的 app 创建一个产品构建。

记住这只在部署到生产前需要。对于普通开发，使用`npm start`。

### 单文件构建
我们提供单文件生产就绪版本的 React 和 React DOM：
```jsx harmony
<script src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
```

记住只有以`.production.min.js`结尾的 React 文件适合生产。

### Branch

最有效的 Brunch 生产构建，安装[terser-brunch](https://github.com/brunch/terser-brunch)：
```jsx harmony
# If you use npm
npm install --save-dev terser-brunch

# If you use Yarn
yarn add --dev terser-brunch
```

然后，为了创建生产构建，添加`-p`表示到`build`命令：
```jsx harmony
brunch build -p
```

记住你只需要去为生产构建这么做。你不应该传递`-p`表示或者应用这个插件在开发，因为这将隐藏有用的 React 警告，让构建更加缓慢。

### Browserify
最有效的 Browserify 产品构建，安装一些插件：
```jsx harmony
# If you use npm
npm install --save-dev envify terser uglifyify 

# If you use Yarn
yarn add --dev envify terser uglifyify 
```

为了创建一个生产构建，确认你添加了这些 transform **（顺序很重要）**。

- [envify](https://github.com/hughsk/envify) transform 确保设置正确的环境，确保他全局(`-g`)。

- [uglifyify](https://github.com/hughsk/uglifyify) transform 移除开发引入。确保它全局（`-g`）。

- 最后，生成的包流向 [terser](https://github.com/terser-js/terser) 管理（[阅读为什么](https://github.com/hughsk/uglifyify#motivationusage)）。

比如：
```jsx harmony
browserify ./index.js \
  -g [ envify --NODE_ENV production ] \
  -g uglifyify \
  | terser --compress --mangle > ./bundle.js
```

记住你只需要去为生产构建做这个。你不需要在开发的时候应用这些插件，因为他们将会隐藏有用的 React 警告，并且让构建更慢。

### Rollup


