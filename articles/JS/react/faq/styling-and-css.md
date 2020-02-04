### 样式和 CSS
### 怎样添加 CSS 类到组件？
传递字符串作为`className`属性：
```jsx harmony
render() {
  return <span className="menu navigation-menu">Menu</span>
}
```
CSS 类依赖于组件 props 和 state 很常见。
```jsx harmony
render() {
  let className = 'menu';
  if (this.props.isActive) {
    className += ' menu-active';
  }
  return <span className={className}>Menu</span>
}
```

提示：如果你发现你经常这么写代码，[classnames]()包可以简化它。

### 我可以使用行内样式？
是的，查阅[这里]()了解样式文档。

### 行内样式坏吗？

CSS 类通常比行内样式好。

### 什么是 CSS-in-JS？

"CSS-in-JS"指的是 CSS 使用 JavaScript 组合，而不是定义额外的文件。在[这里]()阅读 CSS-in-JS 的比较。

注意这个功能不是 React 的一部分，但是第三方库有提供。React 关于样式如何定义没有什么一件；如果有疑惑，一个好的开始是定义你的样式到分离的`*.css`文件作为常用，并使用[classname]()引用他们。

### 我可以在 React 中使用动画吗？
React 可以使用动画。查阅 [React Transition Group]() 和 [React Motion]() 或 [React Spring]()，比如。
