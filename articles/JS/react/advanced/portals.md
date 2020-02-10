[原文地址](https://www.w3.org/TR/wai-aria-practices-1.1/#dialog_modal)
### Portals

Portal 提供了一个第一等的方式去渲染子组件到一个 DOM 节点，这个 DOM 节点存在在父组件 DOM 层级之外。
```jsx harmony
ReactDOM.createPortal(child, container)
```
第一个参数（`child`）是任意[可渲染的 React 子组件](https://reactjs.org/docs/react-component.html#render)，比如一个元素，字符串，或者片段。第二个参数（`container`）是一个 DOM 元素。

### 使用
通常，当你从一个组件的 render 方法返回一个元素，它挂载到 DOM 作为最近父节点的子组件。
```jsx harmony
render() {
  // React mounts a new div and renders the children into it
  return (
    <div>
      {this.props.children}
    </div>
  );
}
```
然而，有时候，插入一个子组件到 DOM 的不同定位很有用：
```jsx harmony
render() {
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
  return ReactDOM.createPortal(
    this.props.children,
    domNode
  );
}

```
portal 一个典型的使用常见是当父组件有一个`overflow:hidden`或者`z-index`样式，但你是你需要子组件在它的容器外可见。比如，弹窗，悬浮卡，工具提示。

> 注意：当使用 portal 的时候，记住[管理键盘聚焦](https://reactjs.org/docs/accessibility.html#programmatically-managing-focus)很重要。对于模态弹窗，确保每个人可以和他们交互，通过[WAI-ARIA 模态创作实践](WAI-ARIA Modal Authoring Practices)。

### 事件冒泡穿过 portal
尽管一个 portal 可以在 DOM 树的任何地方，它表现的像一个任何其他地方可见普通的 React 子组件。像 context 类似的特性同样起效，无论子组件是否是一个 portal，就像 portal 依旧存在在 React 树中，无论在 DOM 树中的啥地方。

这包括事件冒泡。从一个 portal 触发的一个事件将会传播到包含的 React 树，就算这些元素在 DOM 树中不是组件。假设下面的 HTML 结构：
```jsx harmony
<html>
  <body>
    <div id="app-root"></div>
    <div id="modal-root"></div>
  </body>
</html>
```

一个在`#app-root`中的`Parent`组件可能可以去捕获或者不捕获，从兄弟节点`#modal-root`冒泡的事件。
```jsx harmony
// These two containers are siblings in the DOM
const appRoot = document.getElementById('app-root');
const modalRoot = document.getElementById('modal-root');

class Modal extends React.Component {
  constructor(props) {
    super(props);
    this.el = document.createElement('div');
  }

  componentDidMount() {
    // The portal element is inserted in the DOM tree after
    // the Modal's children are mounted, meaning that children
    // will be mounted on a detached DOM node. If a child
    // component requires to be attached to the DOM tree
    // immediately when mounted, for example to measure a
    // DOM node, or uses 'autoFocus' in a descendant, add
    // state to Modal and only render the children when Modal
    // is inserted in the DOM tree.
    modalRoot.appendChild(this.el);
  }

  componentWillUnmount() {
    modalRoot.removeChild(this.el);
  }

  render() {
    return ReactDOM.createPortal(
      this.props.children,
      this.el,
    );
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {clicks: 0};
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This will fire when the button in Child is clicked,
    // updating Parent's state, even though button
    // is not direct descendant in the DOM.
    this.setState(state => ({
      clicks: state.clicks + 1
    }));
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        <p>Number of clicks: {this.state.clicks}</p>
        <p>
          Open up the browser DevTools
          to observe that the button
          is not a child of the div
          with the onClick handler.
        </p>
        <Modal>
          <Child />
        </Modal>
      </div>
    );
  }
}

function Child() {
  // The click event on this button will bubble up to parent,
  // because there is no 'onClick' attribute defined
  return (
    <div className="modal">
      <button>Click</button>
    </div>
  );
}

ReactDOM.render(<Parent />, appRoot);
```
在父组件捕获一个从 portal 冒泡的事件允许不依赖于 portal 更灵活抽象的开发。比如，如果你渲染一个`<Modal />`组件，父组件可以捕获它的事件，无视它是否使用 portal 实现。
