### Uncontrolled Components

在大部分场景，我们推荐使用[受控组件]()去实现表单。在一个受控组件，表单数据被 React 组件处理。可替代的是非受控组件，表单数据被 DOM 自身处理。

编写一个受控组件，[使用 ref]() 从 DOM 获取表单数据，而不是为每一个状态更新编写一个事件处理器。

比如，这个代码中的非受控组件接受一个单独的名字：
```jsx harmony
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.input = React.createRef();
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.current.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
[在 CodePen 中尝试它]()

因为一个非受控组件将事实的圆头保存在 DOM，有时候可以很简单的让 React 和 非 React 代码混合。它也可以很少的代码，如果你想要又快又脏。否则，你应该使用受控组件。

如果你依旧不清楚在一个特定场景你应该使用哪种类型的组件，你可能可以在[这篇关于受控和非受控输入的文章]()找到帮助。


### 默认值
在 React 渲染生命周期，表单元素的`value`属性将会覆盖 DOM 的值。在一个非受控组件，你通常要 React 去指定初始值，但是让后续 更新不受控。为了处理这个场景，你可以指定一个`defaultValue`属性替代`value`。
```jsx harmony
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={this.input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```
同样，`<input type="checkbox">`和`<input type="radio">` 支持`defaultChecked`，并且`<select>`和`<textarea>`支持`defaultValue`。

### 文件输入标签
在 HTML，一个`<input type="file">`让用户从他们的设备存储选择一个或者多个文件上传到一个服务器或者使用 JavaScript 通过[File API]()操作。
```jsx harmony
<input type="file" />
```
在 React，一个`<input type="file">`总是一个非受控组件，因为它的值可以被用户设置，并且不是 可编程的。

你应该使用 File API 去和文件交互。下面的例子显示了怎样创建一个[ref 到 DOM 节点]()在提交处理器去访问文件：
```jsx harmony
class FileInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
    this.fileInput = React.createRef();
  }
  handleSubmit(event) {
    event.preventDefault();
    alert(
      `Selected file - ${
        this.fileInput.current.files[0].name
      }`
    );
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Upload file:
          <input type="file" ref={this.fileInput} />
        </label>
        <br />
        <button type="submit">Submit</button>
      </form>
    );
  }
}

ReactDOM.render(
  <FileInput />,
  document.getElementById('root')
);
```
[在 CodePen 尝试]








