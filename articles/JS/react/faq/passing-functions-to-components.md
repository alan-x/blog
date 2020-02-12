[原文地址](https://reactjs.org/docs/faq-functions.html)
### 传递函数到组件

**如何传递一个时间处理器（像 onClick）到一个组件？**

传递事件处理器和其他函数为属性到子组件：
```jsx harmony
<button onClick={this.handleClick}>
```

如果你需要在处理器内访问父组件，你需要绑定函数到组件实例（看下面）。

**如何绑定函数到组件实例？**

有多种方式可以确保函数可以访问组件属性，比如`this.props`和`this.state`，依赖于你使用的语法和构建步骤。

### 在构造器中绑定（ES2015）

```jsx harmony
class Foo extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  handleClick() {
    console.log('Click happened');
  }
  render() {
    return <button onClick={this.handleClick}>Click Me</button>;
  }
}
```

### Class 属性（Stage 3 提案）
```jsx harmony
class Foo extends Component {
  // Note: this syntax is experimental and not standardized yet.
  handleClick = () => {
    console.log('Click happened');
  }
  render() {
    return <button onClick={this.handleClick}>Click Me</button>;
  }
}
```

### 在 render 中绑定
```jsx harmony
class Foo extends Component {
  handleClick() {
    console.log('Click happened');
  }
  render() {
    return <button onClick={this.handleClick.bind(this)}>Click Me</button>;
  }
}
```

注意：在 render 中使用`Function.prototype.bing`在组件每一次 render 创建一个新的函数，这可能影响性能（看下面）。

### 在 render 中使用箭头函数
```jsx harmony
class Foo extends Component {
  handleClick() {
    console.log('Click happened');
  }
  render() {
    return <button onClick={() => this.handleClick()}>Click Me</button>;
  }
}
```
注意：在 render 中使用箭头函数在每一次组件渲染的时候都创建一个新的函数，可能会破坏基于严格标识比较的优化。

### 在 render 方法中使用箭头函数 OK 吗？

通常来说，可以，没问题，也是传递回调函数作为参数最常用的方式。

如果你有性能问题，务必，优化！

### 为什么绑定是必须的？

在 JavaScript，这两个代码片段不相等：
```jsx harmony
obj.method();
```
```jsx harmony
var method = obj.method;
method();
```

绑定方法帮助确保第二个片段和第一个以相同方式工作。

使用 React，通常你只需要去绑定你传递给其他组件的方法。比如，`<button onClick={this.handleClick} >` 传递的`this.handleClick`，你需要去绑定它。然而，不需要去绑定`render`方法或者生命方法：我们不能传递他们到其他组件。

[Yehuda Katz 的这篇文章](https://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)详细解释了绑定是什么，并且函数在 JavaScript 中是怎样工作的。 

**为什么我的函数在每次组件 render 的时候都被调用？**

确保你没有调用函数，当你传递它到组件的时候：

```jsx harmony
render() {
  // Wrong: handleClick is called instead of passed as a reference!
  return <button onClick={this.handleClick()}>Click Me</button>
}
```
相反，传递函数自身（没有括号）：
```jsx harmony
render() {
  // Correct: handleClick is passed as a reference!
  return <button onClick={this.handleClick}>Click Me</button>
}
```


### 如何传递一个参数到一个事件处理器或者回调？

你可以使用一个箭头函数去包裹事件处理器并传递参数：
```jsx harmony
<button onClick={() => this.handleClick(id)} />
```
这和调用`.bind`一样：
```jsx harmony
<button onClick={this.handleClick.bind(this, id)} />
```
### 例子：使用箭头函数传递参数
```jsx harmony
const A = 65 // ASCII character code

class Alphabet extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      justClicked: null,
      letters: Array.from({length: 26}, (_, i) => String.fromCharCode(A + i))
    };
  }
  handleClick(letter) {
    this.setState({ justClicked: letter });
  }
  render() {
    return (
      <div>
        Just clicked: {this.state.justClicked}
        <ul>
          {this.state.letters.map(letter =>
            <li key={letter} onClick={() => this.handleClick(letter)}>
              {letter}
            </li>
          )}
        </ul>
      </div>
    )
  }
}
```

### 例子：使用 data-attributes 传递参数

可替换的，你可以使用 DOM API 去存储事件处理器需要的数据。假设这个方式你需要去优化大量的元素或者或者有一个依赖于 React.PureComponent 相同的检查的渲染树。
```jsx harmony
const A = 65 // ASCII character code

class Alphabet extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.state = {
      justClicked: null,
      letters: Array.from({length: 26}, (_, i) => String.fromCharCode(A + i))
    };
  }

  handleClick(e) {
    this.setState({
      justClicked: e.target.dataset.letter
    });
  }

  render() {
    return (
      <div>
        Just clicked: {this.state.justClicked}
        <ul>
          {this.state.letters.map(letter =>
            <li key={letter} data-letter={letter} onClick={this.handleClick}>
              {letter}
            </li>
          )}
        </ul>
      </div>
    )
  }
}
```

### 如何防止一个函数调用太快或者太多次？

如果你有一个类似`onClick`或者`onScroll`并且想要防止回调调用太快，则你可以限制执行的回调的频率。这个可以通过使用：

- throttling：样本基于事件频率变化（比如[_.throttle](https://lodash.com/docs#throttle)）

- debouncing：在一段不活跃之后发布改变（比如[_.debounce](https://lodash.com/docs#debounce)）

- requestAnimationFrame throttling：样本基于 [requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) 变化（比如 [raf-schd](https://github.com/alexreardon/raf-schd)）

查看 [这个可视化]() 了解`throttle`和`debounce`函数的比较。

注意：`_.debounce`，`_.throttle`和`raf-schd`提供一个`cancel`方法去取消延迟的回调。你应该在`componentWillUnmount`调用这个方法或者检车确保组件依旧在延迟的函数中挂载。

### Throttle

throttling 防止一个函数在给定事件窗口内调用超过一次。下面的例子 throttle 一个"点击"处理器去防止每秒调用超过一次。

```jsx harmony
import throttle from 'lodash.throttle';

class LoadMoreButton extends React.Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
    this.handleClickThrottled = throttle(this.handleClick, 1000);
  }

  componentWillUnmount() {
    this.handleClickThrottled.cancel();
  }

  render() {
    return <button onClick={this.handleClickThrottled}>Load More</button>;
  }

  handleClick() {
    this.props.loadMore();
  }
}
```

### Debounce
Debouncing 确保一个函数从它上一次调用，直到某个时间过去之后，不会被调用。当你需要在一个派发快速的事件响应中执行一些昂贵的计算（比如，滚动或者键盘事件）的时候，这很有用。下面的例子使用 250ms 延迟 debounce 文本输入框。
```jsx harmony
import debounce from 'lodash.debounce';

class Searchbox extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.emitChangeDebounced = debounce(this.emitChange, 250);
  }

  componentWillUnmount() {
    this.emitChangeDebounced.cancel();
  }

  render() {
    return (
      <input
        type="text"
        onChange={this.handleChange}
        placeholder="Search..."
        defaultValue={this.props.value}
      />
    );
  }

  handleChange(e) {
    // React pools events, so we read the value before debounce.
    // Alternately we could call `event.persist()` and pass the entire event.
    // For more info see reactjs.org/docs/events.html#event-pooling
    this.emitChangeDebounced(e.target.value);
  }

  emitChange(value) {
    this.props.onChange(value);
  }
}
```

### requestAnimationFrame throttling

[requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame) 是入队一个函数到浏览器在适当时间被执行的优化渲染性能的方式。使用`requestAnimationFrame`入队的函数将会在下一帧激活。浏览器将会努力确保 60 帧每秒（60fps）。然而，如果浏览器无法做到，将会压缩每秒的帧数。比如，一个设备可能只能处理 30 fps，你将只得到 30 帧每秒。使用`requestAnimationFrame`去节流是一个非常有用的技术，防止你执行超过 60 次更新每秒。如果你执行 100 次更新每秒，这创建额外的工作给浏览器，用户将无法看到任何东西。

注意：使用这个技术将只捕获一帧最后发布的值。你可以在 [MDN](https://developer.mozilla.org/en-US/docs/Web/Events/scroll) 上看这个例子是优化是如何工作的。

```jsx harmony
import rafSchedule from 'raf-schd';

class ScrollListener extends React.Component {
  constructor(props) {
    super(props);

    this.handleScroll = this.handleScroll.bind(this);

    // Create a new function to schedule updates.
    this.scheduleUpdate = rafSchedule(
      point => this.props.onScroll(point)
    );
  }

  handleScroll(e) {
    // When we receive a scroll event, schedule an update.
    // If we receive many updates within a frame, we'll only publish the latest value.
    this.scheduleUpdate({ x: e.clientX, y: e.clientY });
  }

  componentWillUnmount() {
    // Cancel any pending updates since we're unmounting.
    this.scheduleUpdate.cancel();
  }

  render() {
    return (
      <div
        style={{ overflow: 'scroll' }}
        onScroll={this.handleScroll}
      >
        <img src="/my-huge-image.jpg" />
      </div>
    );
  }
}
```

### 测试你的频率限制

当测试你的频率限制代码工作正常，向前加快时间的能力很有帮助。如果你使用[jest](https://facebook.github.io/jest/)则你可以使用[mock timer](https://facebook.github.io/jest/docs/en/timer-mocks.html)去向前加快时间。如果你使用`requestAnimationFrame`节流，则你可能找到[raf-stub](https://github.com/alexreardon/raf-stub)是一个控制动画帧非常有用的工具。











