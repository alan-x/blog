### Forwarding Refs

Ref forwarding 是通过一个组件自动传递一个[ref]()到它的一个子组件的技术。在应用的大部分组件中，这通常不是必须的。然而，它对于某些组件很有用，特别在重用组件库。最常见的场景描述在下面。

### 转发 refs 到 DOM 组件
假设一个`FancyButton`组件渲染一个原生`buttong` DOM 元素：
```jsx harmony
function FancyButton(props) {
  return (
    <button className="FancyButton">
      {props.children}
    </button>
  );
}
```
React 组件隐藏他们的实现细节，包含他们的渲染输出。其他组件使用`FancyButton`**通常不需要去**[获取一个ref]()到内部的`button`DOM 元素。这很好，因为它防止组件互相依赖 DOM 结构。

尽管这类封装在应用级别是合理的，比如`FeedStory`或者`Comment`，它对于高重用的`叶子`组件很不方便，比如`FancyButton`或者`MyTextInput`。这些组件趋向于贯穿整个应用使用，在一个相似的方式，作为常规 DOM `button`和`input`，并且访问他们的 DOM 节点可能无法避免，比如管理聚焦，选择，或者动画。

**ref forwarding 是一个可选的功能，让一些组件采用一个它接收到 的 ref，并且向下更深的传递（换句话说，"转发"它）到子组件。

在下面的例子中，`FancyButton`使用`React.forwardRef`去获取传递到它的`ref`，然后传递它到它渲染的`button`：
```jsx harmony
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

这种方式，组件使用`FancyButton`可以获取一个到底层`button` DOM 节点的 ref，并在需要的时候访问它 -- 就像他们直接使用 DOM `button`。

接下来一步一步的解释前面的例子发生了什么：

1. 我们创建一个[React ref]()，通过调用`React.createRef`并且赋值它到一个`ref`变量。

2. 我们向下传递我们的`ref`到`<FancyButton ref={ref}>`，通过指定它为一个 JSX 属性。

3. React 传递`ref`到`(props, ref) => ...`函数内部的`forwardRef`作为第二个参数。

4. 我们向下转发这个`ref`参数到`<button ref={ref}>`，通过指定为一个 JSX 属性。

5. 当一个 ref 被绑定，`ref.current`将会指向`<button>`DOM 节点。

注意：第二个`ref`参数只有在你使用`React.forwardRef`调用定义一个组件的时候存在。常规函数或者类组件不接收`ref`参数，并且 ref 也无法在 props 中访问。

Ref 转发不局限于 DOM 组件。你也可以转发 refs 到类组件实例。

### 注意：对组件库维护者

**当你开始使用 forwardRef 在一个组件库，你应该对待她就像一个 break change，并且使用主版本发行你的库。** 这是因为你的库可能有明显的不同行为（比如被赋值的 refs，导出的类型），这可以会破坏依赖旧行为的 app 和其他库。

当`React.forwardRef`存在的时候才应用也因为相同的原因不推荐：它改变你的库的行为并且破坏你的用户的 app，当他们更新 React 本身。


### 在高阶组件中转发 refs

这个技术对于[higher-order components]()（也叫做 HOC）也特别有用。以一个记录组件属性到控制台的 HOC 例子开始：
```jsx harmony
function logProps(WrappedComponent) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      return <WrappedComponent {...this.props} />;
    }
  }

  return LogProps;
}
```
"logProps"HOC 传递所有`prop`穿到它包裹的组件，所以渲染的输出是一样的。比如，我们可以使用这个 HOC 去记录所有传递给我们"fancy button"组件的 props：
```jsx harmony
class FancyButton extends React.Component {
  focus() {
    // ...
  }

  // ...
}

// Rather than exporting FancyButton, we export LogProps.
// It will render a FancyButton though.
export default logProps(FancyButton);
```

对于前面的例子有一个警告：refs 将不会传递通过。这是因为`ref`不是`props`。像`key`，它被 React 特殊处理。如果你添加一个 ref 到一个 HOC，ref 将会引用最外层的容器组件，不是被包裹的组件。

这意味着目标是`FancyButton`的组件将会实际上绑定到`LogProps`组件：
```jsx harmony
import FancyButton from './FancyButton';

const ref = React.createRef();

// The FancyButton component we imported is the LogProps HOC.
// Even though the rendered output will be the same,
// Our ref will point to LogProps instead of the inner FancyButton component!
// This means we can't call e.g. ref.current.focus()
<FancyButton
  label="Click Me"
  handleClick={handleClick}
  ref={ref}
/>;
```

幸运的是，我们可以明确的穿法 refs 到内部的`FancyButton`组件，使用`React.forwardRef`API。`React.forwardRef`接收一个渲染函数，接收`props`和`ref`参数并且返回一个 React 节点。比如：
```jsx harmony
function logProps(Component) {
  class LogProps extends React.Component {
    componentDidUpdate(prevProps) {
      console.log('old props:', prevProps);
      console.log('new props:', this.props);
    }

    render() {
      const {forwardedRef, ...rest} = this.props;

      // Assign the custom prop "forwardedRef" as a ref
      return <Component ref={forwardedRef} {...rest} />;
    }
  }

  // Note the second param "ref" provided by React.forwardRef.
  // We can pass it along to LogProps as a regular prop, e.g. "forwardedRef"
  // And it can then be attached to the Component.
  return React.forwardRef((props, ref) => {
    return <LogProps {...props} forwardedRef={ref} />;
  });
}
```

### 在 DevTools 显示一个自定义名字

`React.forwardRef`接收一个渲染函数。React DevTools 使用这个函数去决定为这个 ref 转发的组件显示什么。

比如，下面的组件将会在 DevTools 中作为`ForwardRef`出现：
```jsx harmony
const WrappedComponent = React.forwardRef((props, ref) => {
  return <LogProps {...props} forwardedRef={ref} />;
});
```

如果你命名了渲染函数，DevTools 将会包含它的名字（比如，"ForwardRef(MyFunction)"）：
```jsx harmony
const WrappedComponent = React.forwardRef(
  function myFunction(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }
);
```
你甚至可以设置`diaplayName`属性到你包裹的组件内部：
```jsx harmony
function logProps(Component) {
  class LogProps extends React.Component {
    // ...
  }

  function forwardRef(props, ref) {
    return <LogProps {...props} forwardedRef={ref} />;
  }

  // Give this component a more helpful display name in DevTools.
  // e.g. "ForwardRef(logProps(MyComponent))"
  const name = Component.displayName || Component.name;
  forwardRef.displayName = `logProps(${name})`;

  return React.forwardRef(forwardRef);
}
```


