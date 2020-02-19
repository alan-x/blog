[原文地址](https://reactjs.org/docs/test-renderer.html)
### Test Renderer

### 导入
```jsx harmony
import TestRenderer from 'react-test-renderer'; // ES6
const TestRenderer = require('react-test-renderer'); // ES5 with npm
```

### 概述

这个包提供一个 React renderer，可以用于渲染 React 组件到纯 JavaScript 对象，不需要依赖 DOM 或者一个原生手机环境。

基本上，这个包让抓取 React DOM 或者 React Native 渲染的平台视图层级镜像更简单，不需要使用一个浏览器或者[jsdom]()。

例子：
```jsx harmony
import TestRenderer from 'react-test-renderer';

function Link(props) {
  return <a href={props.page}>{props.children}</a>;
}

const testRenderer = TestRenderer.create(
  <Link page="https://www.facebook.com/">Facebook</Link>
);

console.log(testRenderer.toJSON());
// { type: 'a',
//   props: { href: 'https://www.facebook.com/' },
//   children: [ 'Facebook' ] }
```

你可以使用 Jest 镜像测试功能去自动保存 JSON 树的备份到文件，并在你的测试检测它没有改变：[了解更多](https://jestjs.io/docs/en/snapshot-testing)

你也可以遍历输出去找到特定节点并断言他们：

```jsx harmony
import TestRenderer from 'react-test-renderer';

function MyComponent() {
  return (
    <div>
      <SubComponent foo="bar" />
      <p className="my">Hello</p>
    </div>
  )
}

function SubComponent() {
  return (
    <p className="sub">Sub</p>
  );
}

const testRenderer = TestRenderer.create(<MyComponent />);
const testInstance = testRenderer.root;

expect(testInstance.findByType(SubComponent).props.foo).toBe('bar');
expect(testInstance.findByProps({className: "sub"}).children).toEqual(['Sub']);
```

### 索引

### [TestRenderer.create()]()
```jsx harmony
TestRenderer.create(element, options);
```
使用传递的 React element 创建一个`TestRenerer`实例。它不使用一个真正的 DOM，但是它依旧完整渲染组件树到内存，因此你可以创建关于它的断言。返回一个[TestRenderer 实例](https://reactjs.org/docs/test-renderer.html#testrenderer-instance)。

### [TestRenderer.act()]()
```jsx harmony
TestRenderer.act(callback);
```
和[react-dom/test-utils 的 act() 助手](https://reactjs.org/docs/test-utils.html#act)类似，`TestRenderer.act`准备一个组件用于断言。使用这个版本的`act()`去包裹回调到`TestRenderer.create`和`testRenderer.update`。
```jsx harmony
import {create, act} from 'react-test-renderer';
import App from './app.js'; // The component being tested

// render the component
let root; 
act(() => {
  root = create(<App value={1}/>)
});

// make assertions on root 
expect(root.toJSON()).toMatchSnapshot();

// update with some different props
act(() => {
  root = root.update(<App value={2}/>);
})

// make assertions on root 
expect(root.toJSON()).toMatchSnapshot();
```
### [testRenderer.toJSON()]()

```jsx harmony
testRenderer.toJSON()
```
返回一个对象，表示渲染的树。这个树只包含平台指定的节点，比如`<div>`或者`<View>`和他们的属性，但是不包含任何用户编写的组件。这用来[快照测试](https://facebook.github.io/jest/docs/en/snapshot-testing.html#snapshot-testing-with-jest)

### [testRenderer.toTree()]()
```jsx harmony
testRenderer.toTree()
```

返回一个代表渲染的树的对象。这个表示比`toJSON()`提供的更加详细，包括用户编写的组件。你可能不需要这个方法，除非你正在基于 test renderer 编写你自己的断言库。

### [testRenderer.update()]()

```jsx harmony
testRenderer.update(element)
```
使用新的根元素在内存重新渲染树。这在根上模拟一个 React 更新。如果新的元素和前一个元素有相同的类型和 key，树将会被更新，否则，它将会重新挂载新的树。

### [testRenderer.unmount()]()

```jsx harmony
testRenderer.unmount()
```
卸载内存中的树，触发适当的生命周期事件。

### [testRenderer.getInstance()]()

```jsx harmony
testRenderer.getInstance()
```
返回对应的根元素对应的实例，如果可用。如果根元素是一个函数组件，则无效，因为他们没有实例。

### [testRenderer.root]()

```jsx harmony
testRenderer.root
```
 返回根"test instance"对象，对创建树中指定节点的断言很有帮助。你可以使用它去找更深处的"test instance"。

### [testInstance.find()]()

```jsx harmony
testInstance.find(test)
```
寻找单一的`test(testInstance)`返回`true`d的子孙测试实例。如果`test(testInstance)`对某个测试实例不返回`true`，它将会抛出一个错误。

### [testInstance.findByType()]()

```jsx harmony
testInstance.findByType(type)
```

使用提供的`type`寻找单一的子孙测试实例。如果没有确切的一个所提供`type`的测试实例，他将抛出一个错误。

### [testInstance.findByProps()]()

```jsx harmony
testInstance.findByProps(props)
```
使用提供的`props`寻找一个单独的子孙测试实例。如果没有确切的一个所提供的`props`测试实例，它将抛出一个错误。

### [testInstance.findAll()]()

```jsx harmony
testInstance.findAll(test)
```
寻找所有的`test(testInstance)`返回`true`的测试实例。

### [testInstance.findAllByType()]()

```jsx harmony
testInstance.findAllByType(type)
```
使用提供的`type`寻找所有的子孙测试实例。

### [testInstance.findAllByProps()]()

```jsx harmony
testInstance.findAllByProps(props)
```

使用提供的`props`寻找所有的子孙测试实例。

### [testInstance.instance]()

```jsx harmony
testInstance.instance
```

组件实例对应这个测试实例。只有类组件可用，因为函数组件没有测试实例。它匹配给定组件中的`this`值。

### [testInstance.type]()

```jsx harmony
testInstance.type
```
组件类型对应这个测试实例。比如，`<Button />`组件有`Button`类型。

### [testInstance.props]()

```jsx harmony
testInstance.props
```

属性对应这个测试实例。比如，`<Button size="small" />`组件有`{size:'small"}`作为属性。

### [testInstance.parent]()

```jsx harmony
testInstance.parent
```

这个测试实例的父测试实例

### [testInstance.children]()

```jsx harmony
testInstance.children
```
这个测试实例的子孙测试实例

### 思想

你可以传递`createNodeMock`函数到`TestRenderer.create`作为选项，这允许自定义 mock ref。`createNodeMock`接收当前元素并返回一个 mock 的 ref 对象。当你测试一个依赖于 ref 的组件的时候很有用。
```jsx harmony
import TestRenderer from 'react-test-renderer';

class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.input = null;
  }
  componentDidMount() {
    this.input.focus();
  }
  render() {
    return <input type="text" ref={el => this.input = el} />
  }
}

let focused = false;
TestRenderer.create(
  <MyComponent />,
  {
    createNodeMock: (element) => {
      if (element.type === 'input') {
        // mock a focus function
        return {
          focus: () => {
            focused = true;
          }
        };
      }
      return null;
    }
  }
);
expect(focused).toBe(true);
```
















