[原文地址](https://reactjs.org/docs/forms.html)
### Forms

HTML 表单元素和其他 DOM 元素有点不同，因为表单元素天生保持一些内部状态。比如，这个表单在简单 HTML 接受一个单一的名字：
```jsx harmony
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

这个表单有默认的 HTML 表单行为，当用户提交表单的时候，跳转到新的页面。如果你在 React 中想要这个行为，它就会工作。但是在大部分场景中，有一个 JavaScript 函数处理表单提交，并且可以访问用户输入到表单的数据是很方便的。达到这个目标的标准方式是叫做"受控组件"的技术。

### 受控组件

在 HTML 中，表单元素，比如`<input>`，`<textarea>`，和`<select>`通常基于用户输入，维护他们自己的状态和更新。在 React，操作状态通常保存在组件的状态属性，并且只能使用[setState()](https://reactjs.org/docs/react-component.html#setstate)更新。

我们可以通过让 React 状态变成"单一数据源"来绑定两者。然后渲染表单的组件也控制在之后的用户输入发生的事情。一个输入表单元素的值被 React 以这种方式控制叫做"受控组件"。

比如，如果我们想要让前一个例子记录名字，当它被提交，我们可以编写表单为受控组件：
```jsx harmony
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/VmmPgp?editors=0010)

因为`value`属性设置在我们的表单元素，显示的值将总是`this.state.value`，让 React 状态为单一数据源。因为`handleChange`在每次键击都更新 React 状态，显示的值将会变成用户输入。

使用一个受控组件，每个状态操作都将和一个处理器函数关联。这导致它直接修改或者验证用户输入。比如，如果我们想要去强制名字使用全部大写，我们可以这样编写`handleChange`：
```jsx harmony
handleChange(event) {
  this.setState({value: event.target.value.toUpperCase()});
}
```

### textarea 标签

在 HTML，一个`<textarea>`元素通过它的子元素定义它的文本：
```jsx harmony
<textarea>
  Hello there, this is some text in a text area
</textarea>
```
在 React 中，一个`<textarea>`使用一个`value`属性替代。这种方式，一个表单使用一个`<textarea>`可以编写非常像是单行输入的表单：
```jsx harmony
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: 'Please write an essay about your favorite DOM element.'
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('An essay was submitted: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
注意`this.state.value`在构造器初始化。因此文本域使用它里面的值开始。

### select 标签

在 HTML，`<select>`创建下拉列表。比如，这个 HTML 创建一个兴趣的下拉列表：
```jsx harmony
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">Coconut</option>
  <option value="mango">Mango</option>
</select>
```

注意 Coconut 选项是初始化选中的，因为`selected`属性。React，与其使用`selected`属性，使用一个`value`属性在根`select`标签。这在受控组件中更方便，因为你只需要在一个地方更新它。比如：
```jsx harmony
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: 'coconut'};

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({value: event.target.value});
  }

  handleSubmit(event) {
    alert('Your favorite flavor is: ' + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```
[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/JbbEzX?editors=0010)

以上，这让`<input type="text">`，`<textarea>`，和`<select>`都工作的很像 - 他们所有接受一个`value`属性，这样你可以使用去实现一个受控组件。

注意：你可以传递一个数组到`value`属性，允许你在`select`标签选择多个选项。
```jsx harmony
<select multiple={true} value={['B', 'C']}>
```

### 文件输入标签

在 HTML 这种，一个`<input type="file">`让用户选择一个或者多个文件，从他们的设备存储，上传到服务端或者通过[File API](https://developer.mozilla.org/en-US/docs/Web/API/File/Using_files_from_web_applications)被 JavaScript 操作。

```jsx harmony
<input type="file" />
```

因为它的值是只读的，他在 React 中是一个非受控组件。这在[之后的文档](https://reactjs.org/docs/uncontrolled-components.html#the-file-input-tag)和其他非受控组件一起讨论。

### 处理多个输入

当你需要去处理多个受控`input`元素，你可以添加一个`name`属性到每个元素，并让处理器函数选择做什么，基于`event.target.name`。

比如：
```jsx harmony
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```
[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/wgedvV?editors=0010)

注意我们怎么使用 ES6 [计算属性名](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names)语法去更新状态的 key 对应的给定输入名：
```jsx harmony
this.setState({
  [name]: value
});
```
它和 ES5 代码是相同的：
```jsx harmony
var partialState = {};
partialState[name] = value;
this.setState(partialState);
```
同时，因为`setState()`自动[合并部分状态到当前状态](https://reactjs.org/docs/state-and-lifecycle.html#state-updates-are-merged)，我们只需要使用改变的部分调用它。

### 控制输入的空值

指定值属性在一个[受控组件](https://reactjs.org/docs/forms.html#controlled-components)，防止用户改变输入，除非你期望。如果你指定给一个`value`，但是输入狂依旧可以边界，你可能意外设置`value`为`undefined`或者`null`。

下面的代码展示了这个。（输入框先是锁定的，但是一会之后就可以编辑）
```jsx harmony
ReactDOM.render(<input value="hi" />, mountNode);

setTimeout(function() {
  ReactDOM.render(<input value={null} />, mountNode);
}, 1000);
```

### 受控组件的替代方式

有时候使用受控组件很啰嗦，因为它需要去编写一个事件处理器，为每一种你的数据可以改变并派发所有的输入状态到一个 React 组件。这会特别烦躁，当你转化一个已经存在的代码库到 React，或者使用 React 和非 React 库交互的时候。在这些场景，你可能想要查阅[非受控组件](https://reactjs.org/docs/uncontrolled-components.html)，一个实现输入表单的可选技术。

### 完全成熟的解决方案

如果你在寻找一个完整的解决方案，包含验证，保持可访问域的跟踪，表单提交的处理，[Formik](https://jaredpalmer.com/formik)是一个流行的选择。然而，它基于受控组件和状态管理的相同原则 -- 因此不要忽视去学习他们。

