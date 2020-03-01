[原文地址](https://reactjs.org/docs/react-without-es6.html)
### React without ES6

通常，你会定义一个 React 组件为普通 JavaScript 类：
```jsx harmony
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```
 如果你不使用 ES6，你可能使用`create-react-class`模块替代：
 ```jsx harmony
var createReactClass = require('create-react-class');
var Greeting = createReactClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```
ES6 类的 API 和`createReactClass`类似，但是更少异常。

### 声明默认属性

对于函数和 ES6 类，`defaultProps`都定义为组件自身的属性：
```jsx harmony
class Greeting extends React.Component {
  // ...
}

Greeting.defaultProps = {
  name: 'Mary'
};
```
使用`createReactClass()`，你需要定义`getDefaultProps()`作为传递进来的对象的一个函数：
```jsx harmony
var Greeting = createReactClass({
  getDefaultProps: function() {
    return {
      name: 'Mary'
    };
  },

  // ...

});
```

### 设置初始状态

在 ES6 类中，你可以通过在构造器定义`this.state`的值来定义初始状态：
```jsx harmony
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: props.initialCount};
  }
  // ...
}
```
使用`createReactClass()`，你需要提供一个独立的`getInitialState`方法，并返回初始状态：
```jsx harmony
var Counter = createReactClass({
  getInitialState: function() {
    return {count: this.props.initialCount};
  },
  // ...
});
```

### 自动绑定

在 ES6 类定义的 React 组件中，方法遵循常规 ES6 类的语义。这意味着他们不自动绑定`this`到实例。你需要在构造器中明确使用`bind(this)`：
```jsx harmony
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
    // This line is important!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    alert(this.state.message);
  }

  render() {
    // Because `this.handleClick` is bound, we can use it as an event handler.
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

使用`createReactClass()`，这不是必须的，因为它绑定所有的方法：
```jsx harmony
var SayHello = createReactClass({
  getInitialState: function() {
    return {message: 'Hello!'};
  },

  handleClick: function() {
    alert(this.state.message);
  },

  render: function() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
});
```

这意味着编写 ES6 类对于事件处理器多一点样板代码，但是在另一方面，在大型应用中有稍微更好的性能。

如果样板代码十分影响你，你可以使用 babel 和**试验性的**[类属性](https://babeljs.io/docs/plugins/transform-class-properties/)语法提案：
```jsx harmony
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
  }
  // WARNING: this syntax is experimental!
  // Using an arrow here binds the method:
  handleClick = () => {
    alert(this.state.message);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

请注意前面的语法是**试验性的**，并且语法可能改变，或者提案不被纳入语言。

如果你希望安全一点，你有一些选项：
- 在构造器中绑定方法
- 使用箭头函数，比如，`onClick={(e) => this.handleClick(e)}
- 继续使用`createReactClass`

### mixin

注意：ES6 发布不支持任何 mixin。因此，当你使用 ES6 类和 React 的时候，也不支持任何 mixin。

我们也在代码中发现使用 mixin 的一些问题，[并且不推荐在新代码中使用他们。](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)

这个章节的存在只是为了引用。

有时候，非常不同的组件可能共享一些普通的功能。有时候叫做[切面关注点]()。`createReactClass`让你为此使用遗留的`mixin`系统。

一个常见使用场景是一个组件在一个定时器等待去更新他自己。使用`setInterval()`很简单，但是当你不需要的时候，为了节约内存清理这个定时器是很重要的。React 提供[生命周期方法]()让你知道组件何时去创建和销毁。创建一个简单的 mixin 来使用这些方法去提供一个简单的`setInterval()`函数，自动在组件销毁的时候清理：
```jsx harmony
var SetIntervalMixin = {
  componentWillMount: function() {
    this.intervals = [];
  },
  setInterval: function() {
    this.intervals.push(setInterval.apply(null, arguments));
  },
  componentWillUnmount: function() {
    this.intervals.forEach(clearInterval);
  }
};

var createReactClass = require('create-react-class');

var TickTock = createReactClass({
  mixins: [SetIntervalMixin], // Use the mixin
  getInitialState: function() {
    return {seconds: 0};
  },
  componentDidMount: function() {
    this.setInterval(this.tick, 1000); // Call a method on the mixin
  },
  tick: function() {
    this.setState({seconds: this.state.seconds + 1});
  },
  render: function() {
    return (
      <p>
        React has been running for {this.state.seconds} seconds.
      </p>
    );
  }
});

ReactDOM.render(
  <TickTock />,
  document.getElementById('example')
);
```
如果一个组件使用多个 mixin 并且多个 mixin 定义了相同的生命周期方法（比如，多个 mixin 想要在组件被销毁之后做一些清理），所有的生命周期方法都保证被调用。定义在 mixin 的方法以 mixin 列出的顺序调用，跟随在组件方法调用之后。
