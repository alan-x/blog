### 0x000 概述
在`React`中，渲染数据的形式有多种多样，但是万变不离其中，都是用`js`。

### 0x001 渲染文本
```
// 渲染文本
ReactDom.render(
    <div>这是一个文本</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`
```
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    '\u8FD9\u662F\u4E00\u4E2A\u6587\u672C'
), document.getElementById('app'));

```
查看浏览器

![clipboard.png](/img/bVbfP3D)

### 0x002 渲染数字字面量

```
// 渲染文本
ReactDom.render(
    <div>{111}</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    111
), document.getElementById('app'));

```
查看浏览器

![clipboard.png](/img/bVbfP3M)

### 0x003 渲染字符串字面量
```
ReactDom.render(
    <div>{"hello world"}</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    "hello world"
), document.getElementById('app'));

```
查看浏览器
![clipboard.png](/img/bVbfP36)

### 0x004 渲染表达式
```
ReactDom.render(
    <div>{1 + 1}</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    1 + 1
), document.getElementById('app'));

```
查看浏览器

![clipboard.png](/img/bVbfP39)
### 0x005 渲染变量
```
const name = 'world'
ReactDom.render(
    <div>{name}</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
var name = 'world';
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    name
), document.getElementById('app'));

```
![clipboard.png](/img/bVbfP4b)

### 0x006 渲染函数
```
const getName = () => 'world'
ReactDom.render(
    <div>{getName()}</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
var getName = function getName() {
    return 'world';
};
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    getName()
), document.getElementById('app'));

```
查看浏览器

![clipboard.png](/img/bVbfP4b)

### 0x007 三元运算符
```
ReactDom.render(
    <div>{
        1 == 1 ? 1 : 0
    }</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    1 == 1 ? 1 : 0
), document.getElementById('app'));

```
查看浏览器

![clipboard.png](/img/bVbfP4C)

### 0x008 渲染数组
```
// 方式1
ReactDom.render(
    <div>{
        [1, 2, 3].map((item, index) => {
            return <p key={index}>{item}</p>
        })
    }</div>,
    document.getElementById('app')
)
// 方式2
const arr = [1, 2, 3].map((item, index) => {
    return <p key={index}>{item}</p>
})
ReactDom.render(
    <div>{arr}</div>,
    document.getElementById('app')
)
```
使用`babel`转义：`babel --plugins transform-react-jsx index.js`

```
// 方式1
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    [1, 2, 3].map(function (item, index) {
        return _react2.default.createElement(
            'p',
            { key: index },
            item
        );
    })
), document.getElementById('app'));
// 方式2
var arr = [1, 2, 3].map(function (item, index) {
    return _react2.default.createElement(
        'p',
        { key: index },
        item
    );
});
_reactDom2.default.render(_react2.default.createElement(
    'div',
    null,
    arr
), document.getElementById('app'));

```
查看浏览器

![clipboard.png](/img/bVbfP4K)

### 0x009 总结
通过转义前转义后对比，我们可以看出来，其实没有`html`的东西存在在其中，完全都是`js`，所以我们可以用代码的形式来构建整个`UI`，将模板抛弃脑后，我们可以在使用`js`定制组件，随意使用`js`的形式组合组件，形成更大的组件，一切都是`js`，自由的 js。

当然，自由也带来混乱，需要将这种自由化为思想的自由，而不是工程的自由、代码的自由，否则将会带来噩梦。

### 0x010 资源
- [react](https://reactjs.org/)
- [transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)
- [源码](https://github.com/followWinter/react-study)


