[原文地址](https://reactjs.org/docs/shallow-renderer.html)
### Shallow Renderer

### 引入
```jsx harmony
import ShallowRenderer from 'react-test-renderer/shallow'; // ES6
var ShallowRenderer = require('react-test-renderer/shallow'); // ES5 with npm
```

### 概述
当为 React 编写单元测试的时候，浅渲染帮助很大。浅渲染让你渲染一个组件"一层"并断垣它渲染方法返回的事实，不需要担心子组件的行为，他们不实例化或者渲染。这不需要一个 DOM。

比如，如果你有下列组件：
```jsx harmony
function MyComponent() {
  return (
    <div>
      <span className="heading">Title</span>
      <Subcomponent foo="bar" />
    </div>
  );
}
```
则你可以断言：
```jsx harmony
import ShallowRenderer from 'react-test-renderer/shallow';

// in your test:
const renderer = new ShallowRenderer();
renderer.render(<MyComponent />);
const result = renderer.getRenderOutput();

expect(result.type).toBe('div');
expect(result.props.children).toEqual([
  <span className="heading">Title</span>,
  <Subcomponent foo="bar" />
]);
```
浅测试当前有一些限制，就是不支持 ref。

注意：我们也推荐查阅 Enzyme 的[浅渲染 API](https://airbnb.io/enzyme/docs/api/shallow.html)。它提供一个更高级的 API，实现相同的功能。


### 索引

### shallowRenderer.render()

你可以认为 shallowRenderer 是一个渲染你测试的组件的"地方"，从这里你可以抽取出组件的输出。

`shallowRenderer.render()`和`ReactDOM.render()`很像，但是它不需要 DOM，并且只渲染一级。这意味着你可以独立测试组件和他们的子组件的实现。

### shallowRenderer.getRenderOutput()

在`shallowRenderer.render()`被调用自后，你可以使用`shallowRenderer.getRenderOutput()`去获取浅渲染的结果。

然后你可以去断言关于输出的事实。

