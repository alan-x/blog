[原文地址](https://reactjs.org/docs/react-dom-server.html)
### ReactDOMServer

`ReactDOMServer`对象让你渲染组件到静态标签。通常，它用于一个 NODE 服务端：
```jsx harmony
// ES modules
import ReactDOMServer from 'react-dom/server';
// CommonJS
var ReactDOMServer = require('react-dom/server');
```

### 概述
下面的方法可以用于服务端和浏览器环境。

- [renderToString()](https://reactjs.org/docs/react-dom-server.html#rendertostring)

- [renderToStaticMarkup()](https://reactjs.org/docs/react-dom-server.html#rendertostaticmarkup)

这些额外的方法依赖于包（`stream`），只能在服务端使用，在浏览器端无法使用。

- [renderToNodeStream()](https://reactjs.org/docs/react-dom-server.html#rendertonodestream)

- [renderToStaticNodeStream()](https://reactjs.org/docs/react-dom-server.html#rendertostaticnodestream)

### 索引

### renderToString()
```jsx harmony
ReactDOMServer.renderToString(element)
```

渲染一个 React 元素到它的初始化 HTML。React 将返回一个 HTML 字符串。你可以使用这个方法在服务端去生成 HTML 并发送标签到快速页面加载的初始化请求，并允许搜索引擎为 SEO 目的爬取你的页面。

如果你调用[ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)到一个已经有服务端渲染标签的节点，React 将保留它并只绑定事件处理器，允许你拥有高性能的首次加载体验。

### renderToStaticMarkup()
```jsx harmony
ReactDOMServer.renderToStaticMarkup(element)
```
和[renderToString](https://reactjs.org/docs/react-dom-server.html#rendertostring)，除了这不创建 React 内部使用的额外 DOM 属性，比如`data-reactroot`。如果你想要使用 React 作为简单的静态页面生成器，这很有用，因为去掉额外的属性可以节省一些字节。

如果你计划在客户端使用 React，并保持标签交互，不要使用这个方法。相反，在服务端使用[renderToString](https://reactjs.org/docs/react-dom-server.html#rendertostring)，在客户端使用[ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)

### renderToNodeStream()

渲染一个 React 元素到它的初始化 HTML。返回一个输出一个 HTML 字符串的[可读的流](https://nodejs.org/api/stream.html#stream_readable_streams)。这个流输出的 HTML 和[ReactDOMServer.renderToString](https://reactjs.org/docs/react-dom-server.html#rendertostring)返回的完全相同。你可以使用这个方法在服务端生成 HTML，发送标签到快速页面加载的初始化请求，并允许搜索引擎为 SEO 目的抓取你的页面。

如果你在一个已经有服务端渲染的标签的节点调用[ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)，React 将会保留它，并绑定事件处理器，允许你去有一个高性能的首次加载体验。

注意：只能用于服务端。这个 API 在浏览器不可用。

这个方法放回的流将会返回 utf-8 编码的字节。如果你需要一个其他编码的流，了解一下类似[iconv-lite](https://www.npmjs.com/package/iconv-lite)之类的项目，提供转化文本的转化流。

### renderToStaticNodeStream()

```jsx harmony
ReactDOMServer.renderToStaticNodeStream(element)
```

和[renderToNodeStream](https://reactjs.org/docs/react-dom-server.html#rendertonodestream)类似，除了这不创建 React 内部使用的额外的 DOM 标签，比如`data-reactroot`。如果你想要使用 React 作为简单的静态页面生成器，这很有用，因为去掉额外的属性可以节省一些字节。

这个流输出的 HTML 和[ReactDOMServer.renderToStaticMarkup](https://reactjs.org/docs/react-dom-server.html#rendertostaticmarkup)返回的一致。

如果你计划在客户端使用 React，并保持标签交互，不要使用这个方法。相反，在服务端使用[renderToNodeStream](https://reactjs.org/docs/react-dom-server.html#rendertonodestream)，在客户端使用[ReactDOM.hydrate()](https://reactjs.org/docs/react-dom.html#hydrate)



注意：只能用于服务端。这个 API 在浏览器不可用。

这个方法放回的流将会返回 utf-8 编码的字节。如果你需要一个其他编码的流，了解一下类似[iconv-lite](https://www.npmjs.com/package/iconv-lite)之类的项目，提供转化文本的转化流。





