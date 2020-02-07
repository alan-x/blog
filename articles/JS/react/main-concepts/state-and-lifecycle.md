[原文地址](https://reactjs.org/docs/state-and-lifecycle.html)
### 状态和生命周期

这个页面引入了 React 组件的状态和生命周期的概念。你可以[在这里找到组件 API 引用的详情](https://reactjs.org/docs/react-component.html)。

思考[前一个章节](https://reactjs.org/docs/rendering-elements.html#updating-the-rendered-element)的时钟例子。在[渲染元素](https://reactjs.org/docs/rendering-elements.html#rendering-an-element-into-the-dom)，我们只学到一种方式去更新 UI。我们调用`ReactDOM.render()`去改变渲染的结果。
```jsx harmony
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```
[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/gwoJZk?editors=0010)

在这个章节，你将学习怎样去创建一个真正可重用的封装的`Clock`组件。它将设置自己的定时器并每秒更新它。

我们可以通过封装时钟的外观开始：
```jsx harmony
function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  ReactDOM.render(
    <Clock date={new Date()} />,
    document.getElementById('root')
  );
}

setInterval(tick, 1000);
```
[在 CodePen 中尝试它]()

然而，它缺少了一个重要的需求：`Clock`设置定时器并每秒更新 UI 应该是`Clock`的实现详情。

理想情况下，我们只编写一次，并让`Clock`自己更新：
```jsx harmony
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

为了实现这个，我们需要去添加"状态"到`Clock`组件。

状态和属性类似，但是它是私有的，并且完全由组件控制。

### 转化一个函数到类
你可以在五步之内转化类似`Clock`的函数组件到一个类：

1. 创建一个 [ES6 类](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes)，使用相同的名字，继承`React.Component`。

2. 添加一个单独的空方法到它，叫做`render()`。

3. 移动函数体到`render()`内

4. 在`render()`内使用`this.props`替换`props`。

5. 删除剩下的空的函数声明。

```jsx harmony
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/zKRGpo?editors=0010)

`Clock`现在定义为一个类而不是一个函数。

`render`方法将在一次更新发生的时候被调用，但是当我们渲染`<Clock />`到相同的 DOM 节点，只有一个单独的`Clock`类实例被使用。这让我们说你用额外的特性，比如本地状态和生命周期方法。

### 添加本地状态到类

我们将在三步内从属性中移动`date`到状态：

1. 在`render()`方法内使用`this.state.date`替换`this.props.date`。
```jsx harmony
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2. 添加一个类构造器，赋值初始的`this.state`：
```jsx harmony
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
注意我们是怎样基于构造器传递`props`：
```jsx harmony
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }
```

3. 从`<Clock />`元素移除`date`属性：
```jsx harmony
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

我们稍后将添加定时器代码到组件自身。

结果看起来像这样：
```jsx harmony
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/KgQpJd?editors=0010)

下一步，我们将创建`Clock`设置他自己的定时器并每秒更新它自己。

### 添加生命方法到一个类

在有很多组件的应用中，当组件被摧毁的时候释放被组件使用的资源很重要。

当`Clock`被第一次渲染到 DOM 的时候，我们要[设置一个定时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/setInterval)。这在 React 中叫做"挂载"。

当`Clock`产生的 DOM 被移除的时候，我们也要[清理定时器](https://developer.mozilla.org/en-US/docs/Web/API/WindowTimers/clearInterval)。这在 React 中叫做"卸载"。

我们可以在组件类中声明特定的方法去执行一些代码，当组件挂载和卸载的时候：
```jsx harmony
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {

  }

  componentWillUnmount() {

  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

这些方法叫做"生命周期方法"。

`componentDidMount()`方法在组件输出被渲染到 DOM 之后执行的。这是设置定时器的好地方：
```jsx harmony
  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }
```

注意我们是如何保存定时器 ID 到`this`（`this.timerID`）。

尽管`this.props`是被 React 自己设置的，并且`this.state`有特定的含义，如果你需要存储一些不需要参与数据流（比如一个定时器 ID）的东西，你可以自由手动添加额外的域到类。

我们将在`componentWillUnmount()`生命周期方法卸载定时器：
```jsx harmony
  componentWillUnmount() {
    clearInterval(this.timerID);
  }
```
最后，我们将实现一个叫做`tick()`的方法，`Clock`将会每秒执行它。

它将会使用`this.setState()`去调度更新到组件本地状态：
```jsx harmony
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```

[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/amqdNA?editors=0010)

现在时钟每秒执行一次。

快速回顾发生了什么，还有哪些方法按顺序被调用：

1. 当`<Clock />`传递给`ReactDOM.render()`，React 调用`Clock`组件的构造器，因为`Clock`需要去显示当前的时间，它使用包含当前时间的对象初始化`this.state`。我们将在稍后更新这个状态。

2. React 之后调用`Clock`组件的`render()`方法。这是 React 知道显示什么到屏幕上的方式。React 更新 DOM 去匹配`Clock`的渲染结果。

3. 当`Clock`输出被插入到 DOM，React 调用`componentDidMount()`生命周期方法。在方法内，`Clock`组件询问浏览器设置一个定时器去调用组件的`tick()`方法每秒一次。

4. 浏览器调用`tick()`方法每秒一次。在方法内，`Clock`组件调度一个 UI 更新，通过使用一个包含当前时间的对象调用`setState()`。感谢`setState()`调用，React 知道状态改变了，再次调用`render()`方法了解应该显示什么到屏幕上。这个时间，`render()`方法内的`this.state.date`将会不同，因此渲染输出将包含更新的时间。React 基于次更新 DOM。

5. 如果`Clock`组件从 DOM 移除，React 调用`componentWillUnmount()`生命周期方法，因此定时器停止了。

### 正确使用状态

有三个关于`setState()`的东西需要知道。

### 不要直接修改状态

比如，这不会重新渲染一个组件：
```jsx harmony
// Wrong
this.state.comment = 'Hello';
```
相反，使用`setState()`：
```jsx harmony
// Correct
this.setState({comment: 'Hello'});
```
唯一你可以赋值`this.state`的地方是构造器。

### 状态更新可能是异步的

React 可能会为了性能批量将`setState()`调用合并到单一的更新。

因为`this.props`和`this.state`可能异步更新，你不应该依赖他们的值去计算下一个状态。

比如，这个代码可能会更新定时器失败：
```jsx harmony
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

为了修复它，使用`setState()`的第二种格式，它接受一个函数而不是一个对象。这个函数将会接受前一个状态作为参数，和更新被接受的时候的 props 作为第二个参数：
```jsx harmony
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```
前面我们使用[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，但是也可以使用常规函数：
```jsx harmony
// Correct
this.setState(function(state, props) {
  return {
    counter: state.counter + props.increment
  };
});
```

### 状态更新是合并的

当你调用`setState()`，React 合并你提供的对象到当前状态。

比如，你的状态可能包含多个独立的变量：
```jsx harmony
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```
然后你可以独立更新他们，使用分离的`setState()`调用：
```jsx harmony
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```
合并是浅合并的，因此`this.setState({comments})`留下完整的`this.state.posts`，但是完全替换`this.state.comments`。

### 自顶向下的数据流

父组件或者子组件都无法知道某个组件是有状态的还是无状态的，他们不应该关心它定义为函数或者类。

这也是为什么状态通常叫做本地或者封装的。它无法被任何组件访问，不管是拥有它的还是设置它的。

一个组件可能选择向下传递它的状态作为属性到子组件：
```jsx harmony
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```
这对用户定义的组件也有用：
```jsx harmony
<FormattedDate date={this.state.date} />
```
[在 CodePen 中尝试它]()

这通常叫做"自顶向下"或者"单向"数据流。任何状态总是被一些特定组件拥有，任何从状态中派发的数据或者 UI 只影响他们树中的后面组件。


如果你想象一个组件树是属性的瀑布，每一个组件的状态就像一个额外的水源，在任意点加入它但是也下落。

为了显示所有组件是真正独立的，我们可以创建一个`App`组件渲染三个`<Clock>`：
```jsx harmony
function App() {
  return (
    <div>
      <Clock />
      <Clock />
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/vXdGmd?editors=0010)

每一个`Clock`设置他自己的定时器并独立更新。

在 React 应用中，一个组件是否有状态是一个组件的实现详情，可能随着时间改变。你可以使用无状态组件在有状态组件内，或者相反。
