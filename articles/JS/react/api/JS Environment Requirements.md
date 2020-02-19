[原文地址](https://reactjs.org/docs/javascript-environment-requirements.html)
### JavaScript 环境要求

React 16 依赖于集合类型[Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)和[Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)。如果你支持还没有原生（比如，IE < 11）提供或者兼容实现（比如，IE 11）这些的旧的浏览器和设备，考虑全局包含 polyfill 到你打包的应用中，比如[core-js](https://github.com/zloirock/core-js)或者[babel-polyfill](https://babeljs.io/docs/usage/polyfill/)。

一个填充过的使用 core-js 的支持旧浏览器的 React 16 环境可能看起来像这样：
```jsx harmony
import 'core-js/es/map';
import 'core-js/es/set';

import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
``` 

React 也依赖`requestAnimationFrame`（甚至在测试环境中）。你可以使用[raf](https://www.npmjs.com/package/raf)包去模拟`requestAnimationFrame`：
```jsx harmony
import 'raf/polyfill';
```
