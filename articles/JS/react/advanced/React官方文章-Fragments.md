### Fragments
React 中一个常见模式是一个组件返回多个元素。Fragment 让你分组一个列表的子组件，不太贱额外节点到 DOM。
```jsx harmony
render() {
  return (
    <React.Fragment>
      <ChildA />
      <ChildB />
      <ChildC />
    </React.Fragment>
  );
}
```
还有一个新的[缩写语法]()去声明他们。

### 动机
组件的一个常见模式是返回一个列表的子组件。使用这个例子的 React 片段：
```jsx harmony
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```
`<Columns />` 需要去返回多个`<td>`元素，为了让渲染的 HTML 合法。如果父`div`在`<Columns />`的`render()`的内部使用，将会导致 HTML 不合法。
```jsx harmony
class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>Hello</td>
        <td>World</td>
      </div>
    );
  }
}
```
导致`<Table />`输出：
```jsx harmony
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```
Fragment 解决了这个问题。

### 使用
```jsx harmony
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Hello</td>
        <td>World</td>
      </React.Fragment>
    );
  }
}
```
导致一个正确的`<Table />`输出：
```jsx harmony
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```
### 缩写语法

有一个新的，短的语法你可以用来声明片段，它看起来像空标签：
```jsx harmony
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Hello</td>
        <td>World</td>
      </>
    );
  }
}
```
你可以像使用任何其他元素一样使用`<></>`，除了它不支持 key 或者属性。

### 有 Key 的 Fragment
Fragment 使用明确的`<React.Fragment>`语法声明可以有 key。一个使用场景是映射一个几何到一个片段数组 -- 比如，去创建一个描述列表：
```jsx harmony
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Without the `key`, React will fire a key warning
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```
`key`是唯一的可以传递给`Fragment`的属性。在未来，我们可能田间对其他属性的支持，比如事件处理器。

### 在线例子：
你可以尝试新的 JSX 片段语法，使用这个[CodePen]()。









