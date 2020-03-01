[原文地址](https://reactjs.org/docs/accessibility.html)
### 代码分割

### 打包

大部分 React 应用都有他们的文件"打包"，使用类似[Webpack](https://webpack.js.org/)，[Rollup](https://rollupjs.org/)，[Browserify](http://browserify.org/)。打包是导入文件并合并他们到单一的文件的流程：一个"包"。这个包可以包含在一个网页去一次性加载整个应用。

### 例子

### APP：
```jsx harmony
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```
```jsx harmony
// math.js
export function add(a, b) {
  return a + b;
}
```
### Bundle:
```jsx harmony
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

注意：你的包最终看起来和这个大不一样。

如果你使用[Create React APP](https://github.com/facebookincubator/create-react-app)，[Next.js](https://github.com/zeit/next.js/)，[Gatsby](https://www.gatsbyjs.org/)，或者类似工具，你将会有一个开箱即用的 Webpack 设置打包你的应用。

如果你不是，你需要自己设置打包。比如，查阅 Webpack 文档的[安装](https://webpack.js.org/guides/installation/)和[开始](https://webpack.js.org/guides/getting-started/)指南。

### 代码分割

打包是很好的，但是随着你的应用增长，你的包也会增长。特别是你包含了巨大的第三方库。你需要去注意包含到你的包的代码，这样你不会意外让它太大，导致花费太多时间加载。

为了避免最后有一个巨大的包，最好先解决这个问题并开始"分割"你的包。代码分割是打包器支持的功能，比如[Webpack](https://webpack.js.org/guides/code-splitting/)，[Rollup](https://rollupjs.org/guide/en/#code-splitting)和 Browserify（通过[factor-bundle](https://github.com/browserify/factor-bundle)），可以创建多个包，并在运行时动态加载。

代码分割你的应用可以帮助你"懒加载"用户当前需要的，可以显著的提高你的应用的性能。尽管你没有减少你的应用的代码总量，你避免了加载用户可能永远不会需要的，并减少初始化加载需要的代码量。

### import()

引入代码分割到你的应用的最好方式是通过动态`import()`语法。

之前：
```jsx harmony
import { add } from './math';

console.log(add(16, 26));
```
之后：
```jsx harmony
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

当 Webpack 发现这个语法，它自动开始代码分割你的应用。如果你使用 Create React App，这已经为你配置好了，比可以立即[开始使用它](https://facebook.github.io/create-react-app/docs/code-splitting)。这在[Next.js](https://github.com/zeit/next.js/#dynamic-import)中也开箱即用。

如果你手动设置 Webpack，你可能想要阅读 Webpack 的[代码分割指南](https://webpack.js.org/guides/code-splitting/)。你的 Webpack 配置应该看起来[像这个](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)。

当使用[Babel](https://babeljs.io/)，你需要去确保 Babel 可以转化动态导入语法，但是不转化它。因此你将需要[babel-plugin-syntax-dynamic-import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import)。

### React.lazy

注意：`React.lazy`和 Suspense 对于服务端还不可用，如果你要去代码分割一个服务端渲染应用，我们推荐[Loadable 组件](https://github.com/gregberge/loadable-components)。它有一个很棒的[服务端渲染包分割指南](https://loadable-components.com/docs/server-side-rendering/)。

`React.lazy`函数让你渲染一个动态导入为一个常规组件。

之前：
```jsx harmony
import OtherComponent from './OtherComponent';
```

之后：
```jsx harmony
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```
当这个组件第一次渲染的时候，将自动加载包含在`OtherComponent`中的包。
[原文地址](https://reactjs.org/docs/accessibility.html)
### 代码分割

### 打包

大部分 React 应用都有他们的文件"打包"，使用类似[Webpack](https://webpack.js.org/)，[Rollup](https://rollupjs.org/)，[Browserify](http://browserify.org/)。打包是导入文件并合并他们到单一的文件的流程：一个"包"。这个包可以包含在一个网页去一次性加载整个应用。

### 例子

### APP：
```jsx harmony
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```
```jsx harmony
// math.js
export function add(a, b) {
  return a + b;
}
```
### Bundle:
```jsx harmony
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

注意：你的包最终看起来和这个大不一样。

如果你使用[Create React APP](https://github.com/facebookincubator/create-react-app)，[Next.js](https://github.com/zeit/next.js/)，[Gatsby](https://www.gatsbyjs.org/)，或者类似工具，你将会有一个开箱即用的 Webpack 设置打包你的应用。

如果你不是，你需要自己设置打包。比如，查阅 Webpack 文档的[安装](https://webpack.js.org/guides/installation/)和[开始](https://webpack.js.org/guides/getting-started/)指南。

### 代码分割

打包是很好的，但是随着你的应用增长，你的包也会增长。特别是你包含了巨大的第三方库。你需要去注意包含到你的包的代码，这样你不会意外让它太大，导致花费太多时间加载。

为了避免最后有一个巨大的包，最好先解决这个问题并开始"分割"你的包。代码分割是打包器支持的功能，比如[Webpack](https://webpack.js.org/guides/code-splitting/)，[Rollup](https://rollupjs.org/guide/en/#code-splitting)和 Browserify（通过[factor-bundle](https://github.com/browserify/factor-bundle)），可以创建多个包，并在运行时动态加载。

代码分割你的应用可以帮助你"懒加载"用户当前需要的，可以显著的提高你的应用的性能。尽管你没有减少你的应用的代码总量，你避免了加载用户可能永远不会需要的，并减少初始化加载需要的代码量。

### import()

引入代码分割到你的应用的最好方式是通过动态`import()`语法。

之前：
```jsx harmony
import { add } from './math';

console.log(add(16, 26));
```
之后：
```jsx harmony
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

当 Webpack 发现这个语法，它自动开始代码分割你的应用。如果你使用 Create React App，这已经为你配置好了，比可以立即[开始使用它](https://facebook.github.io/create-react-app/docs/code-splitting)。这在[Next.js](https://github.com/zeit/next.js/#dynamic-import)中也开箱即用。

如果你手动设置 Webpack，你可能想要阅读 Webpack 的[代码分割指南](https://webpack.js.org/guides/code-splitting/)。你的 Webpack 配置应该看起来[像这个](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269)。

当使用[Babel](https://babeljs.io/)，你需要去确保 Babel 可以转化动态导入语法，但是不转化它。因此你将需要[babel-plugin-syntax-dynamic-import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import)。

### React.lazy

注意：`React.lazy`和 Suspense 对于服务端还不可用，如果你要去代码分割一个服务端渲染应用，我们推荐[Loadable 组件](https://github.com/gregberge/loadable-components)。它有一个很棒的[服务端渲染包分割指南](https://loadable-components.com/docs/server-side-rendering/)。

`React.lazy`函数让你渲染一个动态导入为一个常规组件。

之前：
```jsx harmony
import OtherComponent from './OtherComponent';
```

之后：
```jsx harmony
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```
当这个组件第一次渲染的时候，将自动加载包含在`OtherComponent`中的包。

`React.lazy`接收一个函数，必须调用动态`import()`。这必须返回一个`Promise`，它解析为一个模块，有一个`default`导出，包含一个 React 组件。

懒组件应该在一个`Suspense`组件内渲染，当我们等待懒组件加载的时候，允许我们显示一些回滚内容（比如一个加载指示器）。

```jsx harmony
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

`fallback`属性接收任何在等待组件加载时你想要显示的 React 组件。你可以将`Suspense`组件放在懒组件前面的任何地方。你甚至包裹多个懒组件到一个单一的`Suspense`组件。

```jsx harmony
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```

### 错误边界

如果其他模块加载失败（比如，因为网络失败），他将会触发一个错误。你可以处理这些错误去显示一个很好的用户体验，并使用[错误边界](https://reactjs.org/docs/error-boundaries.html)管理恢复。一旦你创建你的错误边界，当网络错误的时候，你可以在懒组件前面的任何地方使用它去显示一个错误状态。

```jsx harmony
import MyErrorBoundary from './MyErrorBoundary';
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

### 基于路由的代码分割

决定在你的应用什么地方导入代码分割可能有点棘手。你要确保你选择的地方均匀的分割包，但是不破坏用户体验。

路由是开始的好地方。网页上的大部分人习惯页面过渡需要一些时间去加载。你还倾向于一次性重新渲染整个页面，因此你的用户不太可能同时和页面的其他元素交互。

这是一个基于路由设置代码分割的例子，使用像[React Route](https://reacttraining.com/react-router/)的`React.lazy`。

```jsx harmony
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

### 具名导出

`React.lazy`当前只支持默认导出。如果你想要导入的模块使用具名导出，你可以创建一个中介模块，重新导出它，作为默认。这确保 tree shaking 依旧工作，不会拉到未使用的组件。

```jsx harmony
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;
```
```jsx harmony
// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";
```
```jsx harmony
// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```

`React.lazy`接收一个函数，必须调用动态`import()`。这必须返回一个`Promise`，它解析为一个模块，有一个`default`导出，包含一个 React 组件。

懒组件应该在一个`Suspense`组件内渲染，当我们等待懒组件加载的时候，允许我们显示一些回滚内容（比如一个加载指示器）。

```jsx harmony
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

`fallback`属性接收任何在等待组件加载时你想要显示的 React 组件。你可以将`Suspense`组件放在懒组件前面的任何地方。你甚至包裹多个懒组件到一个单一的`Suspense`组件。

```jsx harmony
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```

### 错误边界

如果其他模块加载失败（比如，因为网络失败），他将会触发一个错误。你可以处理这些错误去显示一个很好的用户体验，并使用[错误边界](https://reactjs.org/docs/error-boundaries.html)管理恢复。一旦你创建你的错误边界，当网络错误的时候，你可以在懒组件前面的任何地方使用它去显示一个错误状态。

```jsx harmony
import MyErrorBoundary from './MyErrorBoundary';
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

### 基于路由的代码分割

决定在你的应用什么地方导入代码分割可能有点棘手。你要确保你选择的地方均匀的分割包，但是不破坏用户体验。

路由是开始的好地方。网页上的大部分人习惯页面过渡需要一些时间去加载。你还倾向于一次性重新渲染整个页面，因此你的用户不太可能同时和页面的其他元素交互。

这是一个基于路由设置代码分割的例子，使用像[React Route](https://reacttraining.com/react-router/)的`React.lazy`。

```jsx harmony
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

### 具名导出

`React.lazy`当前只支持默认导出。如果你想要导入的模块使用具名导出，你可以创建一个中介模块，重新导出它，作为默认。这确保 tree shaking 依旧工作，不会拉到未使用的组件。

```jsx harmony
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;
```
```jsx harmony
// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";
```
```jsx harmony
// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```














