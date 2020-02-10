 [原文地址](https://reactjs.org/docs/higher-order-components.html)
### Higher-Order Components

高阶组件（HOC）是 React 重用组件逻辑的高级技术。HOC 本身不是 React API 的一部分。他们是 React 的组成性质中产生的模式。

具体的，**一个高阶组件是一个函数，接收一个组件，并返回一个新的组件。**
```jsx harmony
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

一个组件传输属性到 UI，一个高阶组件传输一个组件到另一个组件。

HOC 在第三方 React 库很常见，比如 Redux 的[connect](https://github.com/reduxjs/react-redux/blob/master/docs/api/connect.md#connect)和 Relay 的[createFragmentContainer](http://facebook.github.io/relay/docs/en/fragment-container.html)。

在这个文档，我们将讨论为什么高阶组件很有用，还有如何编写你自己的。

### 为分离关注点使用 HOC。

注意：
我们之前推荐使用 mixin 作为处理分离关注点的方式。我们发现 mixin 比他们的价值导致了更多问题。[了解更多](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)关于为什么我们移除 mixin 和怎样转移你已经存在的组件。 

组件是 React 重用代码的主要单元。然而，你将会发现一些模式并不十分适合传统组件。

比如，假设你有一个`CommentList`组件订阅一个外部的数据源去显示一个组件列表：
```jsx harmony
class CommentList extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      // "DataSource" is some global data source
      comments: DataSource.getComments()
    };
  }

  componentDidMount() {
    // Subscribe to changes
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    // Clean up listener
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    // Update component state whenever the data source changes
    this.setState({
      comments: DataSource.getComments()
    });
  }

  render() {
    return (
      <div>
        {this.state.comments.map((comment) => (
          <Comment comment={comment} key={comment.id} />
        ))}
      </div>
    );
  }
}
```
然后，你编写了一个组件去订阅一个单独的博客文章，使用类似的模式：
```jsx harmony
class BlogPost extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {
      blogPost: DataSource.getBlogPost(props.id)
    };
  }

  componentDidMount() {
    DataSource.addChangeListener(this.handleChange);
  }

  componentWillUnmount() {
    DataSource.removeChangeListener(this.handleChange);
  }

  handleChange() {
    this.setState({
      blogPost: DataSource.getBlogPost(this.props.id)
    });
  }

  render() {
    return <TextBlock text={this.state.blogPost} />;
  }
}
```
`CommontList`和`BlogPost`不相同 -- 他们在`DataSource`上调用不同的方法，他们渲染不同的输出。但是他们大部分的实现是相同的。

- 在挂载的时候，添加一个`DataSource`的变化监听器。
- 在监听器内部，在数据源变化的时候调用`setState`
- 在卸载的时候，移除变化监听器

你可以想象到，在一个大型的 app 中，相同模式的订阅`DataSource`和调用`setState`将会重复出现。我们想要一个抽象允许我们去定义这个逻辑到一个单一的地方，跨越多个组件分享。这就是高阶组件擅长的。

我们可以编写一个函数去创建组件，像`CommentList`和`BlogPost`，它订阅`DataSource`。函数将会将接收一个参数作为接收订阅数据作为属性的子组件。现在调用函数`withSubscription`：
```jsx harmony
const CommentListWithSubscription = withSubscription(
  CommentList,
  (DataSource) => DataSource.getComments()
);

const BlogPostWithSubscription = withSubscription(
  BlogPost,
  (DataSource, props) => DataSource.getBlogPost(props.id)
);
```
第一个参数是包裹的组件。第二个参数获取我们感兴趣的数据，使用一个`DataSource`和当前的属性。

当`CommentListWithSubscription`和`BlogPostWithSubscription` 被渲染之后，`CommentList`和`BlogPost`将会被传递`data`属性和从`DataSource`获取的的最新数据。
```jsx harmony
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```
注意一个 HOC 没有修改输入的组件，也没有使用继承去复制行为。HOC 组合原始的组件，通过将它包裹在一个容器组件。一个 HOC 是一个纯函数，没有任何副作用。

这就是它！包裹的组件接收容器所有的属性，和一个新的属性，`data`，它用来渲染它的输出。HOC 不关心数据怎么或者为什么使用，并且包裹的组件不关心数据从哪里来。

因为`withSubscription`是一个普通函数，你可以添加你喜欢的参数数量，比如，你可能想要让`data`属性可配置，进一步隔离 HOC 和包裹的组件。或者你可以接收一个参数配置`shouldComponentUpdate`,或者一个可以配置数据源。这都是可能的，因为 HOC 完全受组件定义控制。

类似组件，`withSubscription`和包裹的组件完全是基于属性的。这让它可以很简单的从一个 HOC 切换到另一个，只要他们提供相同的属性给包裹的组件。比如，如果你改变数据获取库，这将会很有用。

### 不要操作原始的组件。使用组合。
 
抵制在一个 HOC 内部修改组件的原型的诱惑（或者操作它）。
```jsx harmony
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps = function(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  };
  // The fact that we're returning the original input is a hint that it has
  // been mutated.
  return InputComponent;
}

// EnhancedComponent will log whenever props are received
const EnhancedComponent = logProps(InputComponent);
```
这里有一些问题。一个是输入框无法从增强的组件中分离出来使用。关键是，如果你应用其他 HOC 到`EnhancedComponent`，它也操作了`componentWillReceiveProps`，第一个 HOC 的功能将会被覆盖！这个 HOC 也无法和函数组件一起使用，它没有生命周期方法。

操作 HOC 是一个封装泄漏 -- 消费者必须知道他们怎样实现，为了去避免和其他 HOC 冲突。

与其操作，HOC 应该使用组合，通过包裹输入组件到一个容器组件：
```jsx harmony
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // Wraps the input component in a container, without mutating it. Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```
这个 HOC 和操作版本有相同的功能，避免了潜在的冲突。它和类和函数组件都能一起使用。并且因为它是纯函数，它可以和其他 HOC 组合，甚至和它自己。

你可能注意到 HOC 和**容器组件**模式的类似。容器组件是分离高级别和低级别关注点的职责的部分的策略。容器管理像订阅或者状态的东西，并传递属性到组件处理类似渲染 UI。HOC 使用容器作为他们实现的一部分。你可以认为 HOC 是参数化的容器组件定义。

### 约定：传递不相关的属性到包裹的组件

HOC 添加特性到组件。他们不应该大幅度修改它的约定。它期待从 HOC 返回的组件和包裹的组件有类似的接口。

HOC 应该传递和他不相关的属性。大部分 HOC 包含一个 render 方法，看起来像这样：
```jsx harmony
render() {
  // Filter out extra props that are specific to this HOC and shouldn't be
  // passed through
  const { extraProp, ...passThroughProps } = this.props;

  // Inject props into the wrapped component. These are usually state values or
  // instance methods.
  const injectedProp = someStateOrInstanceMethod;

  // Pass props to wrapped component
  return (
    <WrappedComponent
      injectedProp={injectedProp}
      {...passThroughProps}
    />
  );
}
```
这个约定帮助确保 HOC 是尽可能灵活和可重用的。

### 约定：最大化组合

不是所有的 HOC 看起来都一样。有时候他们只接受一个参数，包裹的组件：
```jsx harmony
const NavbarWithRouter = withRouter(Navbar);
```
通常，HOC 接收额外的参数，在这个从 Relay 的例子中，一个配置对象用于指定一个组件的数据依赖：
```jsx harmony
const CommentWithRelay = Relay.createContainer(Comment, config);
```
最常见的 HOC 签名看起来像这样：
```jsx harmony
// React Redux's `connect`
const ConnectedComment = connect(commentSelector, commentActions)(CommentList);
```
什么？如果你分开来看，它很简单就可以看出发生了什么。
```jsx harmony
// connect is a function that returns another function
const enhance = connect(commentListSelector, commentListActions);
// The returned function is a HOC, which returns a component that is connected
// to the Redux store
const ConnectedComment = enhance(CommentList);
```
换句话说，`connect`是一个高阶函数，返回一个高阶组件！

这个形式看起来迷惑人或者不需要，但是它有一个很有帮助的属性。单参数 HOC，比如`connect`函数返回的有签名`Component => Component`。输出类型和输入类型相同的函数很容易组合。
```jsx harmony
// Instead of doing this...
const EnhancedComponent = withRouter(connect(commentSelector)(WrappedComponent))

// ... you can use a function composition utility
// compose(f, g, h) is the same as (...args) => f(g(h(...args)))
const enhance = compose(
  // These are both single-argument HOCs
  withRouter,
  connect(commentSelector)
)
const EnhancedComponent = enhance(WrappedComponent)
```
(相同的属性也允许`connect`和其他增强风格的 HOC 作为装饰器使用，一个 experimental JavaScript proposal。)

`comopse`工具函数被 lodash 在内的许多三方库包含（比如[lodash.flowRight](https://lodash.com/docs/#flowRight)，[Redux](https://redux.js.org/api/compose)，[Ramda](https://ramdajs.com/docs/#compose)）。

### 约定：为了方便调试，包裹显示名字
HOC 创建的容器组件就像任何其他组件显示在[React Developer Tools](https://github.com/facebook/react-devtools)。为了方便调试，选择一个名字显示表示它是 HOC 的结果。

最常见的技术是包裹包裹组件的显示名字。因此如果你的高阶组件命名为`withSubscription`，包裹的组件的显示名字是`CommentList`，使用显示名`WithSubscriptiion(CommentList)`：
```jsx harmony
function withSubscription(WrappedComponent) {
  class WithSubscription extends React.Component {/* ... */}
  WithSubscription.displayName = `WithSubscription(${getDisplayName(WrappedComponent)})`;
  return WithSubscription;
}

function getDisplayName(WrappedComponent) {
  return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

### 注意
如果你刚接触 React，高级组件有一些不可见的警告。

### 不要在 render 方法内使用 HOC
React 的 diff 算法（叫做 reconciliation）使用组件表示去决定它是否更新存在的子树或者抛弃它重新挂载一个。如果从`render`返回的组件和前一个渲染的相等（`===`），React 递归更新子树，通过将它和新的 diff。如果他们不相等，前一个子树完全被卸载。

通常，你不应该去关心这个。但是对 HOC 和重要，因为它意味着你无法应用一个 HOC 到一个组件的 render 方法：

```jsx harmony
render() {
  // A new version of EnhancedComponent is created on every render
  // EnhancedComponent1 !== EnhancedComponent2
  const EnhancedComponent = enhance(MyComponent);
  // That causes the entire subtree to unmount/remount each time!
  return <EnhancedComponent />;
}
```
这里的问题不仅仅是性能 -- 重新挂载一个组件导致组件的状态和它的所有子元素全部丢失。

相反，在组件定义之外应用 HOC，让组件只创建一次。然后，它的定义将会在贯穿渲染一致。无论如何，这通常是你需要的。

在这些罕见的场景，你需要动态应用一个 HOC，你可以在一个组件声明周期方法或者构造器中这么做。

### 静态方法必须被复制

有时候，在 React 组件定义一个静态方法是非常有用的。比如，Relay 容器暴露一个静态方法`getFragment`去促进 GraphQL 片段的组合。

当你应用一个 HOC 到一个组件，尽管，原始的组件被容器组件包裹。这意味新组件没有任何原始组件的静态方式：
```jsx harmony
// Define a static method
WrappedComponent.staticMethod = function() {/*...*/}
// Now apply a HOC
const EnhancedComponent = enhance(WrappedComponent);

// The enhanced component has no static method
typeof EnhancedComponent.staticMethod === 'undefined' // true
```

为了解决这个，你可以复制方法到容器，在返回之前：
```jsx harmony
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  // Must know exactly which method(s) to copy :(
  Enhance.staticMethod = WrappedComponent.staticMethod;
  return Enhance;
}
```
然而，这需要你明确知道那个方法需要被复制。你可以使用[hoist-non-react-statics](https://github.com/mridgway/hoist-non-react-statics)去自动复制所有非 React 静态方法：
```jsx harmony
import hoistNonReactStatic from 'hoist-non-react-statics';
function enhance(WrappedComponent) {
  class Enhance extends React.Component {/*...*/}
  hoistNonReactStatic(Enhance, WrappedComponent);
  return Enhance;
}
```
其他可能的解决方案是分离的从组件自己导出静态方法。
```jsx harmony
// Instead of...
MyComponent.someFunction = someFunction;
export default MyComponent;

// ...export the method separately...
export { someFunction };

// ...and in the consuming module, import both
import MyComponent, { someFunction } from './MyComponent.js';

```
### Refs 不传递
尽管高阶组件约定传递所有的属性到包裹的组件，这对于 ref 无效。这是因为`ref` 不是一个真正的属性 -- 像`key`，它被 React 特殊处理。如果你添加一个 ref 到一个元素，他的组件是 HOC 的结果，ref 引用是最外层的容器组件的实例，而不是包裹的组件。
这个问题的解决方案是使用`React.forwardRef`API（从 React 16.3 引入）。[在转发 ref 了解更多关于它的消息](https://reactjs.org/docs/forwarding-refs.html)。
