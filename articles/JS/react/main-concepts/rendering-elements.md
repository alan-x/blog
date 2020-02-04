### 渲染元素

元素是 React 应用最小的构建块。

一个元素描述你在屏幕上看到什么：
```jsx harmony
const element = <h1>Hello, world</h1>;
```
不像浏览器 DOM 元素，React 元素是简单对象，并且创建很简单。React DOM 负责更新 DOM 去匹配 React 元素。

注意：元素和更广为人知的概念"组件"容易混淆。我们将会在[下一个章节]()介绍组件。元素是组件的"构成"。我们推荐你阅读这个章节在跳过之前。


### 渲染一个元素到 DOM
假设有一个`<div>`在你的 HTML 文件的某个地方：
```jsx harmony
<div id="root"></div>
```

我们把这个 DOM 节点叫做"root"，因为任何在它之内的将被 React DOM 管理。

只使用 React 构建的应用通常只有一个单一的根 DOM 节点。如果你整合 React 到一个存在的应用，如果你喜欢，你可能有多个独立的根 DOM 节点。
```jsx harmony
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```
[在 CodePen 尝试它]()

它打印"Hello，world"在页面上。


### 更新渲染的元素

React 元素是[不可变的]()。一旦你创建一个元素，你不能改变它的子元素或者属性。一个元素就像电影的一帧：它表示 在某个时间点的 UI。

 根据我们目前的只是，更新 UI 的唯一方式是创建一个新的元素，传递它到`ReactDOM.render()`。
 
 考虑这个时钟例子：
 ```jsx harmony
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```

[在 CodePen 尝试它]()

它在[setInterval()]()回调调用`ReactDOM.render()`每秒。

注意：实际上，大部分 React 应用只调用`ReactDOM.render()`一次。在下一章节，我们将学习怎样封装这些代码到[有状态的组件]()。

我们推荐你不要跳过话题，因为他们建立在彼此之上。

### React 只在需要的时候更新

React DOM 和前一个比较元素和它的子元素，并且只应用需要的 DOM 更新让 DOM 到期待的状态。

你可以通过使用浏览器工具检查[最后的例子]()验证：

![](https://reactjs.org/granular-dom-updates-c158617ed7cc0eac8f58330e49e48224.gif)

尽管我们每秒创建一个元素描述整个 UI 树，只有文本节点的内容有改变，并且被 React DOM 更新。

在我们经验中，思考 UI 在给定时刻的样子比如何随着时间改变，会角山一大类的 bug。






















