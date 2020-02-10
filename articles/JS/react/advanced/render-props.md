[原文地址](https://reactjs.org/docs/render-props.html)
### Render Props
术语["render prop"](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)指的是一个在组件间使用一个 prop 分享代码的技术，它的值是一个函数。

一个有 render prop 的组件接收一个函数，它返回一个 React 元素，调用它，而不是实现自己的渲染逻辑。
```jsx harmony
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```
使用 render props 的库包括[React ROuter](https://reacttraining.com/react-router/web/api/Route/render-func)，[Downshift](https://github.com/paypal/downshift)，和[Formik](https://github.com/jaredpalmer/formik)。

在这个文档，我们将讨论为什么 render prop 很有用，怎样编写你自己的渲染属性。

### 为了分离关注点使用渲染属性

组件是 React 的主要代码重用单元，但是它不总是明显的怎样去分享状态或者行为，从一个组件封装到另一个需要相同状态的组件里。

比如，下面的组件在 web app 中跟踪鼠标的位置。
```jsx harmony
class MouseTracker extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>
        <h1>Move the mouse around!</h1>
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}
```
当鼠标在屏幕上移动的时候，组件在`<p>`内显示它的 (x, y) 坐标。

现在问题是：我们怎样在其他组件重用这个行为？换句话说，如果其他组件需要去知道鼠标位置，我们可以封装这个行为，这样我们可以简单的使用这个组件分享它？

因为组件是 React 中代码重用的基本单元，让我们稍微重构代码去使用`<Mouse>`组件，封装我们需要重用的行为。
```jsx harmony
// The <Mouse> component encapsulates the behavior we need...
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/* ...but how do we render something other than a <p>? */}
        <p>The current mouse position is ({this.state.x}, {this.state.y})</p>
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse />
      </div>
    );
  }
}
```
现在`<Mouse>`组件封装所有的行为，绑定对`mousemove`事件的监听，并存储鼠标的 (x, y) 定位，但是这不是真正的重用。

比如，假设我们有一个`<Cat>`组件渲染一个猫的图片，在屏幕上追随鼠标。我们可能使用一个`<Cat mouse={{x, y}}>`属性去告诉组件鼠标的坐标，这样它知道如何在屏幕上定位这个图片。

作为第一个尝试，你可能尝试在`<Mouse>`内部的 render 方法渲染`<Cat>`，像这样：
```jsx harmony
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class MouseWithCat extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          We could just swap out the <p> for a <Cat> here ... but then
          we would need to create a separate <MouseWithSomethingElse>
          component every time we need to use it, so <MouseWithCat>
          isn't really reusable yet.
        */}
        <Cat mouse={this.state} />
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <MouseWithCat />
      </div>
    );
  }
}
```
这个方法对我们指定的使用场景有用，但是我们没有达到真正封装行为到一种可重用的方式的目的。现在，每一次我们想要为不同的使用场景提供鼠标定位，我们需要去创建一个新的组件（比如，建立另一个`<MouseWithCat>`）然后为这个使用场景渲染一些指定的行为。

这就是 render prop 使用的地方：替代在一个`<Mouse>`组件内部硬编码一个`<Cat>`，并且有效的改变它的渲染输出，我们可以提供`<Mouse>`一个函数属性，用来动态决定怎样渲染一个 render prop。

```jsx harmony
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```

现在，替代实际克隆`<Mouse>`组件并硬编码一些东西到它的`render`方法去解决特定使用场景，我们提供一个`render`属性，这样`<Mouse>`可以用于动态决定它的渲染。

更具体的说，**一个渲染属性是一个函数属性，一个组件用来知道渲染什么**。

这个技术让我们需要分享的行为变得及其方便。为了得到这个行为，使用一个`render`属性渲染一个`<Mouse>`，告诉他使用当前鼠标的 (x, y) 渲染啥。

关于渲染属性一个有趣的事需要注意的是你可以实现大部分[高阶组件](https://reactjs.org/docs/higher-order-components.html)（HOC）使用一个带渲染属性的常规组件。比如没如果你想要使用`withMouse`HOC 替代一个`<Mouse>`组件，你可以简单创建一个使用渲染属性的的常规组件。

```jsx harmony
// If you really want a HOC for some reason, you can easily
// create one using a regular component with a render prop!
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```
所以，使用一个渲染属性让他可以使用两种模式。

### 使用不是 render 的属性
有一点很重要，需要记住只是因为这个模式叫做"render props"，你不必使用一个叫做 render 的属性名去使用这个模式。实际上，[组件用来知道渲染什么的任何函数属性的技术叫做一个"render prop"](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)。

尽管前面的例子使用`render`，我们可以简单的使用`children`属性！
```jsx harmony
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)}/>
```

记住，`children`属性不需要实际列在你的 JSX 元素的"属性"列表。相反，你可以直接放置在元素内部。
```jsx harmony
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```
你可以在[react-motion](https://github.com/chenglou/react-motion)API 中看见这个技术。

因为这个技术不太常见，你可能想要去明确的状态`children`应该是一个函数，在你的`propTypes`当这样设计一个 API 的时候：
```jsx harmony
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```

### 注意

### 当 Render Props 和 React.PureComponent 一起使用的时候要注意

 使用一个渲染属性会抵消使用[React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)带来的优点，如果你在`render`方法内创建一个函数。这是因为浅属性比较将会总是为新属性返回`false`，这个场景中的每一个`render`将会为渲染属性生产新的值。

比如，继续前面的`<Mouse>`组件，如果`Mouse`是继承自`React.PureComponent`，而不是`React.Component`，我们的例子可能看起来像这样：
```jsx harmony
class Mouse extends React.PureComponent {
  // Same implementation as above...
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>

        {/*
          This is bad! The value of the `render` prop will
          be different on each render.
        */}
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```
在这个例子中，每一次`<MouseTracker>`渲染，它生成一个新的函数作为`<Mouse render>`属性的值，因此抵消了`<Mouse>`继承`React.PureComponent`带来的效果。

为了解决这个问题，你可以定属性作为实例方法，比如：
```jsx harmony
class MouseTracker extends React.Component {
  // Defined as an instance method, `this.renderTheCat` always
  // refers to *same* function when we use it in render
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```
如果你无法静态定义属性（比如，因为你需要关闭组件的属性和/或状态），`<Mouse>`应该继承`React.Component`替代。
