### CDN 链接

React 和 ReactDOM 都可以通过 CDN 访问。
```jsx harmony
<script crossorigin src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
```

前面的版本只对开发环境有意义，不适合于生产环境。React 压缩的和优化的生产版本可以在这里获取：
```jsx harmony
<script crossorigin src="https://unpkg.com/react@16/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@16/umd/react-dom.production.min.js"></script>
```

加载指定版本的`react`和`react-dom`，替换`16`为版本号。

### 为什么添加 crossorigin 属性？

如果你从 CDN 获取 React，我们推荐保留[crossorigin](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_settings_attributes)属性的设置：
```jsx harmony
<script crossorigin src="..."></script>
```
我们推荐去验证你使用的 CDN 设置了`Access-Control-Allow-Origin: *`HTTP 头部：

![](https://reactjs.org/static/cdn-cors-header-89baed0a6540f29e954065ce04661048-dd807.png)

这在 React 16 和之后启用更好的[错误处理体验](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)。
