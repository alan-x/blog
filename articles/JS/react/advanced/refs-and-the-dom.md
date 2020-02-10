[原文地址](https://reactjs.org/docs/refs-and-the-dom.html)
### Refs and the DOM

Refs 提供访问 render 方法创建的 DOM 或者 React 元素的方法。

在典型的 React 数据流中，Props 是父组件和他们子孙交互的唯一方式。为了修改一个组件，你使用新的 props 重新渲染它。然而，在一些场景中，你需要命令式的在典型数据流之外修改一个子组件。被修改的子组件可以是一个 React 组件的实例，也可以是一个 Dom element。对于这两种场景，React 提供一个紧急出口。

### 什么时候使用 Refs

这是 refs 一些好的使用场景：
- 管理聚焦，文本选择，或者媒体播放。
- 触发命令式动画。
- 和第三方 DOM 库交互

避免能够声明完成的任何情况下使用 refs。

比如，与其在`Dialog`组件上暴露`open()`和`close()`方法，不如传递一个`isOpen`属性给它。

### 不要过度使用 Refs

你的第一倾向可能是使用 refs "让东西发生"在你的应用中。如果这就是场景，花点时间，严谨的思考一下，组件树的哪里应该拥有状态。通常，很清晰的，拥有状态的适合点在层级的高处。查阅[Lifting State Up](https://reactjs.org/docs/lifting-state-up.html)指南了解这个例子。

注意：下面的例子已经更新为使用 React 16。3 引入的`React.createRef()` API。如果你使用更早版本的 React，我们推荐使用[callback refs](https://reactjs.org/docs/refs-and-the-dom.html#callback-refs)替代。

### 创建 Refs
Refs 使用`React.createRef()`，并且通过`ref`属性绑定 React 元素。Refs 通常在组件构造之后被赋予一个实例属性，这样他们可以贯穿整个组件被引用。

```jsx harmony
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```
### 访问 Refs
当一个 ref 传递到`render`中的一个元素，一个到节点的引用可以通过 ref 的`current`属性。
```jsx harmony
const node = this.myRef.current;
```

ref 的值根据节点的类型不同：

- 当`ref`属性用于 HTML 元素，构造器中使用`React.createRef()`创建的`ref`接收底层的 DOM 元素作为它的`current`属性。
- 当一个`ref`属性用于一个自定义类组件，`ref`对象接收挂载的组件的实例作为它的`current`。
- **你可能不使用 ref 属性在函数组件上**，因为他们没有实例。

下面的例子展示了不同。

### 添加一个 Ref 到一个 DOM 元素

这个代码使用一个`ref`去存储一个 DOM 节点的引用：
```jsx harmony
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // create a ref to store the textInput DOM element
    this.textInput = React.createRef();
    this.focusTextInput = this.focusTextInput.bind(this);
  }

  focusTextInput() {
    // Explicitly focus the text input using the raw DOM API
    // Note: we're accessing "current" to get the DOM node
    this.textInput.current.focus();
  }

  render() {
    // tell React that we want to associate the <input> ref
    // with the `textInput` that we created in the constructor
    return (
      <div>
        <input
          type="text"
          ref={this.textInput} />

        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
### 添加一个 Ref 到一个类组件
如果我们想要去包裹前面的`CustomTextInput`在挂载之后去模拟被点击，我们可以使用一个 ref 去获取自定义输入框，并手动调用它的`focusTextInput`方法。
```jsx harmony
class AutoFocusTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }

  componentDidMount() {
    this.textInput.current.focusTextInput();
  }

  render() {
    return (
      <CustomTextInput ref={this.textInput} />
    );
  }
}
```
注意，这只有在如果`CustomTextInput`被声明为一个类的时候才生效：
```jsx harmony
class CustomTextInput extends React.Component {
  // ...
}
```

### Refs 和函数组件
**你可能不能在函数组件上使用 ref 属性**，因为他们没有实例：
```jsx harmony
function MyFunctionComponent() {
  return <input />;
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.textInput = React.createRef();
  }
  render() {
    // This will *not* work!
    return (
      <MyFunctionComponent ref={this.textInput} />
    );
  }
}
```
如果你需要一个 ref，你应该将组件转化成一个类，就像当你需要生命周期方法或者状态时你做的。

你可以，然而，**在函数组件内使用 ref 属性**，只要你引用一个 DOM 元素或者一个类组件：
```jsx harmony
function CustomTextInput(props) {
  // textInput must be declared here so the ref can refer to it
  let textInput = React.createRef();

  function handleClick() {
    textInput.current.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={textInput} />

      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```
### 暴露 DOM 引用给父组件
在罕见的场景，你可能想要从父组件访问一个子组件的 DOM 节点。这通常不被推荐，因为它破坏了组件的封装，但是偶尔它很有用，对于触发聚焦或者测量子组件 DOM 节点的大小和位置。

尽管你可以[添加一个 ref 到子组件](https://reactjs.org/docs/refs-and-the-dom.html#adding-a-ref-to-a-class-component)。这不是一个理想的解决方案，因为你只能得到一个组件的实例而不是一个 DOM 节点。此外，它无法和函数组件一起工作。

如果你使用 React 16.3 或者更高，我们推荐为这种场景使用 [ref forwarding](https://reactjs.org/docs/forwarding-refs.html)。**ref forwarding 让组件可选的暴露任何子组件的 ref 作为他们自己的**。你可以[在 ref forwarding 文档](https://reactjs.org/docs/forwarding-refs.html#forwarding-refs-to-dom-components)找到如何暴露一个子组件的 DOM 给父组件的详细例子。

如果你使用 React 16.2 或者更低，或者如果你需要比 ref forwarding 提供的更加弹性，你可以使用[这个替代解决方法](https://gist.github.com/gaearon/1a018a023347fe1c2476073330cc5509)并且明确传递一个 ref 作为不同命名的属性。

当可能，我们反对暴露 DOM 节点，但是它是一个很有用的逃生舱口。注意这个方法需要你去添加一些代码到子组件。如果你需要对子组件实现绝对的不控制，你最后的选项是使用[findDOMNode()](https://reactjs.org/docs/react-dom.html#finddomnode)，但是它在[StrictMode](https://reactjs.org/docs/strict-mode.html#warning-about-deprecated-finddomnode-usage)是不被鼓励且被抛弃的。

### Callback Refs

React 也支持其他方式去设置 refs，叫做"callback refs"，可以在设置和取消的时候做到更细粒度的控制。

作为传递`createRef()`创建的`ref`的替代，你传递一个函数。函数接收 React 组件实例或者 HTML DOM 元素作为它的参数，它可以在其他地方被存储和访问。

下面的例子实现了一个常见的模式：使用`ref`回调去存储一个引用到一个 DOM 节点，在一个实例属性。
```jsx harmony
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // Focus the text input using the raw DOM API
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // autofocus the input on mount
    this.focusTextInput();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
    return (
      <div>
        <input
          type="text"
          ref={this.setTextInputRef}
        />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```
React 将会使用 DOM 元素调用`ref`回调，当组件挂载的时候，并在它卸载的时候使用`null`调用它。Refs 保证在`componentDidMount`或者`componentDidUpdate`触发的时候更新。

你可以在组件间传递 callback refs，就像你使用`React.createRef()`创建的对象 refs 那样。
```jsx harmony
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  render() {
    return (
      <CustomTextInput
        inputRef={el => this.inputElement = el}
      />
    );
  }
}
``` 

在前面的例子中，` Parent`传递它的 ref callback 作为一个`inputRef`属性给`CustomTextInput`，并且`CustomTextInput`传递相同的函数作为一个特殊的`ref`属性到`<input>`。作为结果，`Parent`的`this.inputElement`将会设置为`CustomTextInput`中的`<input>`元素对应的 DOM 节点。

### 遗留 API：String Refs

如果你在之前使用过 React，你可能对`ref`属性是一个字符的老的 API 很熟悉，像"textInput"，DOM 节点像`this.refs.textInput`一样被访问。我们强烈返回它因为 string refs 有[一些问题]()，被认为是遗留，并且**可能在未来的一个发行中被移除**。

注意：如果你现在使用`this.refs.textInput`去访问 refs。我们推荐使用[callback pattern](https://reactjs.org/docs/refs-and-the-dom.html#callback-refs)或者[createRef API](https://reactjs.org/docs/refs-and-the-dom.html#creating-refs)替代。

### callback refs 注意事项

如果`ref`回到定义为一个行内函数，它将会在更新的时候被调用两次，一次是使用`null`，一次使用 DOM 元素。这是因为函数的新实例在每一次渲染的时候都会被创建，所以 React 需要清理旧的 ref 并设置一个新的。你可以，通过定义`ref` callback 为一个类上的绑定方法，避免这个问题。但是注意这在大部分情况下没关系。
