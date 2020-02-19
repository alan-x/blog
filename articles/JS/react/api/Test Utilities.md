[原文地址](https://reactjs.org/docs/test-utils.html)
### 测试工具

### 导入
```jsx harmony
import ReactTestUtils from 'react-dom/test-utils'; // ES6
var ReactTestUtils = require('react-dom/test-utils'); // ES5 with npm
```

### 概述
`ReactTestUtils`让你在你选择的测试框架中测试 React 组件十分简单。在 Facebook，我们使用[Jest](https://facebook.github.io/jest/)无痛的 JavaScript。了解如何使用 Jest，可以通过 Jest 网站的[React 教程](https://jestjs.io/docs/tutorial-react)。

注意：我们推荐使用[React 测试库]()是为了让你并鼓励你编写测试，就像最终用户使用一样。

或者，Airbnb 发行了一个测试工具，叫做`Enzyme`，让断言，操作，遍历你的 React 组件的输出更简单。

### 索引

### act()

为了准备一个组件去测试，包裹渲染它的代码并在一个`act()`调用内执行更新。这让你的测试运行更接近 React 在浏览器中的工作方式。

注意：如果你使用`react-test-renderer`，它也提供了一个相同行为的`act`导出。

比如，假设我们有一个`Counter`组件：
```jsx harmony
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 0};
    this.handleClick = this.handleClick.bind(this);
  }
  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }
  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }
  handleClick() {
    this.setState(state => ({
      count: state.count + 1,
    }));
  }
  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={this.handleClick}>
          Click me
        </button>
      </div>
    );
  }
}
```
这是我们测试它的方式：
```jsx harmony
import React from 'react';
import ReactDOM from 'react-dom';
import { act } from 'react-dom/test-utils';
import Counter from './Counter';

let container;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
});

afterEach(() => {
  document.body.removeChild(container);
  container = null;
});

it('can render and update a counter', () => {
  // Test first render and componentDidMount
  act(() => {
    ReactDOM.render(<Counter />, container);
  });
  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');

  // Test second render and componentDidUpdate
  act(() => {
    button.dispatchEvent(new MouseEvent('click', {bubbles: true}));
  });
  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```
- 不要忘记派发的 DOM 事件只有在 DOM 容器被添加到`document`之后才能工作。你可以使用类似[React 测试库](https://testing-library.com/react)之类的库去产生样板代码。

- [recipes](https://reactjs.org/docs/testing-recipes.html)文档包含`act()`行为方式更详细的描述，还有例子和使用。

### mockComponent()
```jsx harmony
mockComponent(
  componentClass,
  [mockTagName]
)
```
传递一个 mock 组件模块到这个方法，使用有用的方法扩展它，让它作为一个虚拟 React 组件。与其像平常一样渲染，组件将会是一个简单的`<div>`（如果`mockTagName`提供了，则是其他标签），包含任何提供的子组件。

注意：`mockComponent()`是一个遗留 API。我们推荐使用[jest.mock()](https://facebook.github.io/jest/docs/en/tutorial-react-native.html#mock-native-modules-using-jestmock)替代。

### isElement()
```jsx harmony
isElement(element)
```
如果`element`是任意 React 元素，就返回`true`。

### isElementOfType()
```jsx harmony
isElementOfType(
  element,
  componentClass
)
```
如果`element`是一个 React 元素，它的类型是一个 React `componentClass`，返回`true`。

### isDOMComponent()

```jsx harmony
isDOMComponent(instance)
```
如果`instance`是一个 DOM 组件（比如`<div>`或者`<span>`），返回`true`。

### isCompositeComponent()

```jsx harmony
isCompositeComponent(instance)
```
如果`instance`是一个用户定义的组件，比如类或者函数，返回`true`。

### isCompositeComponentWithType()

```jsx harmony
isCompositeComponentWithType(
  instance,
  componentClass
)
```
如果`instance`是一个组件，它的类型是 React `componentClass`，返回`true`。

### findAllInRenderedTree()

```jsx harmony
findAllInRenderedTree(
  tree,
  test
)
```
遍历`tree`中所有的组件，并累计所有`test(component)`是`true`的组件。这对它自己没什么用，但是对其他测试工具很重要。

### scryRenderedDOMComponentsWithClass()
```jsx harmony
scryRenderedDOMComponentsWithClass(
  tree,
  className
)
```
在渲染的树中寻找所有类名称和`className`匹配的 DOM 组件。

### findRenderedDOMComponentWithClass()

```jsx harmony
findRenderedDOMComponentWithClass(
  tree,
  className
)
```

类似[scryRenderedDOMComponentsWithClass()]()，但是只有一个结果，并返回这个结果，否则，如果有任何其他数量的命中，抛出异常。

### scryRenderedDOMComponentsWithTag()

```jsx harmony
scryRenderedDOMComponentsWithTag(
  tree,
  tagName
)
```
在渲染的树中寻找所有标签名称和`tagName`匹配的 DOM 组件。

### findRenderedDOMComponentWithTag()

```jsx harmony
findRenderedDOMComponentWithTag(
  tree,
  tagName
)
```
类似[scryRenderedDOMComponentsWithTag()]()，但是只有一个结果，并返回这个结果，否则，如果有任何其他数量的命中，抛出异常。

### scryRenderedComponentsWithType()
```jsx harmony
scryRenderedComponentsWithType(
  tree,
  componentClass
)
```

查找所有组件的实例，它的类型和`componentClass`相同。

### findRenderedComponentWithType()

```jsx harmony
findRenderedComponentWithType(
  tree,
  componentClass
)
```

类似[scryRenderedComponentsWithType()]()，但是只有一个结果，并返回这个结果，否则，如果有任何其他数量的命中，抛出异常。

### renderIntoDocument()

```jsx harmony
renderIntoDocument(element)
```

渲染一个 React 元素到一个独立的 DOM 节点。**这个函数需要一个 DOM**。它和下面同等有效：

```jsx harmony
const domContainer = document.createElement('div');
ReactDOM.render(element, domContainer);
```

注意：你需要在你导入`React`之前保证`window`，`window.document`和`window.document.createElement`全局可用。否则 React 将认为它无法访问 DOM 和类似`setState`的方法。

### 其他工具

### Simulate
```jsx harmony
Simulate.{eventName}(
  element,
  [eventData]
)
```
模拟一个事件在 DOM 节点上派发，使用可选的`eventData`事件数据。

`Simulate`有为[React 理解的每一个事件](https://reactjs.org/docs/events.html#supported-events) 的方法

**点击一个元素**
```jsx harmony
// <button ref={(node) => this.button = node}>...</button>
const node = this.button;
ReactTestUtils.Simulate.click(node);
```
**改变一个输入框的值穿厚点击 ENTER**

```jsx harmony
// <input ref={(node) => this.textInput = node} />
const node = this.textInput;
node.value = 'giraffe';
ReactTestUtils.Simulate.change(node);
ReactTestUtils.Simulate.keyDown(node, {key: "Enter", keyCode: 13, which: 13});
```
注意：你需要提供你在你的组件使用的任何事件属性（比如，keyCode，等...），因为 React 不为你创建这些。
