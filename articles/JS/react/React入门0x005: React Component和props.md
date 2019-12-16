### 0x000 概述
这一章讲组件，组件才是`React`的核心，也是`React`构建的项目中最小的单元。

### 0x001 组件
上面的章节有一个类似下面的栗子：
```
const App = () => {
    return <p>hello</p>
}
ReactDom.render(
    App(),
    document.getElementById('app')
)
```
查看浏览器

![clipboard.png](/img/bVbfTG4)

我们可以给他参数
```
const App = (name) => {
    return <p>hello {name}</p>
}
ReactDom.render(
    App("world"),
    document.getElementById('app')
)
```
查看浏览器

![clipboard.png](/img/bVbfTG6)

由此，我们可以自定义一些特别的组件了，比如：
```
const Article = (title, content) => {
    return <div>
        <h1>{title}</h1>
        <p>{content}</p>
    </div>
}
ReactDom.render(
    Article("送方外上人","孤云将野鹤，岂向人间住。莫买沃洲山，时人已知处。"),
    document.getElementById('app')
)
```
查看浏览器

![clipboard.png](/img/bVbfTHk)

### 0x002 组件的函数写法和参数传递
组件也可以使用jsx 来写
```
const App = () => {
    return <p>hello</p>
}
ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
用`babel`转码试试：`babel --plugins transform-react-jsx index.js`:
```
var App = function App() {
    return _react2.default.createElement(
        'p',
        null,
        'hello'
    );
};
_reactDom2.default.render(_react2.default.createElement(App, null), document.getElementById('app'));
```
可以看到依旧被转成了`createElement`函数，由`React.createElement`的[文档](https://reactjs.org/docs/react-api.html#createelement)说明可知，该函数的第一个参数可以是类似`div`、`p`之类的`html`元素，也可以是`React Component`，而`React Component`可以是一个类或者一个函数。该栗子中就是函数

那如果我们要实现传参呢，我们可以想想`html`如何传参，比如`img`：
```
<img src="XXX.png">
```
那么写法和`html`及其类似的 `jsx` 呢？可想而知：
```
ReactDom.render(
    <App name="world"></App>,
    document.getElementById('app')
)
```
我们使用`babel`转码看看：`babel --plugins transform-react-jsx index.js`:
```
_reactDom2.default.render(_react2.default.createElement(App, { name: 'world' }), document.getElementById('app'));

```
可以看到，被转化成了键值对作为`React.createElement`的第二个参数，那我们应该如何访问这个参数呢？
```
const App = (props) => {
    console.log(props)
    return <p>hello {props.name}</p>
}
ReactDom.render(
    <App name="world"></App>,
    document.getElementById('app')
)
```

查看浏览器：

![clipboard.png](/img/bVbfTIP)

![clipboard.png](/img/bVbfTIL)


### 0x003 组件的类写法和传参
在之前的文章，也已经写过这么一个类似的栗子：
```
class App extends React.Component{
    render(){
        return <p>hello</p>
    }
}
ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
查看浏览器：

![clipboard.png](/img/bVbfTI2)

那如何传参呢？
```
class App extends React.Component {
    render() {
        console.log(this)
        return <p>hello</p>
    }
}

ReactDom.render(
    <App name="world"></App>,
    document.getElementById('app')
)
```
查看浏览器：

![clipboard.png](/img/bVbfTI8)

我们可以看到，参数被放在了`props`中，所以，我们可以像这样访问：
```
class App extends React.Component {
    render() {
        console.log(this)
        return <p>hello {this.props.name}</p>
    }
}

ReactDom.render(
    <App name="world"></App>,
    document.getElementById('app')
)
```
查看浏览器

![clipboard.png](/img/bVbfTJg)

### 0x004 `jsx`也是`js`
因为`jsx`也是`js`，所以，上面的栗子我们也可以如此改造
```
class App extends React.Component {
    render() {
        console.log(this)
        return <p>hello {this.props.name}</p>
    }
}

ReactDom.render(
    <App name={Date()}></App>,
    document.getElementById('app')
)
```

使用`babel`转码看看：`babel --plugins transform-react-jsx index.js`:
```
_reactDom2.default.render(_react2.default.createElement(App, { name: Date() }), document.getElementById('app'));

```

查看浏览器：


![clipboard.png](/img/bVbfTJH)



### 0x005 总结

- 在项目中，我们一般使用类的形式组织整个项目，但是在某些情况下，使用函数也是一种不错的选择。
- `jsx`是使用类`html`的语法来写`js`，所以很多`html`的思想可以套用到`jsx`，但是一定要记得，这是`js`，而不是`html`

### 0x006 资源

- [react](https://reactjs.org/)
- [transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)
- [源码](https://github.com/followWinter/react-study)
