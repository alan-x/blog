[原文地址](https://reactjs.org/docs/hooks-state.html)
### Using the State Hook

Hooks 是 React 16.8 新添加的功能。他们让你不写类就能使用状态和其他 React 功能。

[介绍页面](https://reactjs.org/docs/hooks-intro.html)使用这个例子去熟悉 Hooks：
```jsx harmony
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
我们通过比较这个代码和相同的类例子开始学习 Hooks。

### 相同的类例子

如果你曾在 React 中使用 class，这段代码应该很熟悉：
```jsx harmony
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
状态以`{ count: 0 }`开始，当用户点击按钮，通过调用`this.setState()`我们增加`state.count`。我们将在整个页面中使用这个类的片段。

> 注意：你可能想为什么这里我们使用一个计数器替代更加真实的例子。这是帮助我们聚焦 API，当我们还在使用 Hooks 创建我们的第一步。

### Hooks and Function Components
 
提醒一下，React 中的函数组件看起来像这样：
```jsx harmony
const Example = (props) => {
  // You can use Hooks here!
  return <div />;
}
```
或者这样：
```jsx harmony
function Example(props) {
  // You can use Hooks here!
  return <div />;
}
```
你可能已经知道这是"无状态组件"。我们现在引入使用 React 状态的能力，所以我们更偏向于叫做"函数组件"。

Hooks **无法**在类内部工作。但是你可以使用他们替代编写类。

#### What's a Hook

我们的新例子通过从 React 引入`useState` Hook 开始：
```jsx harmony
import React, { useState } from 'react';

function Example() {
  // ...
}
```
**啥是 Hook？** 一个 Hook 是一个特殊的函数让你"hook into"React 功能。比如，`useState`是一个 Hook，让你添加 React 状态到函数组件。我们将会在稍后学习其他 Hook。

**什么时候使用 Hook？** 如果你写一个函数组件，你意识到你需要添加一些状态，在之前，你需要转化它为一个类。现在你可以在已经存在的函数组件中使用一个 Hook。我们将马上要去这么做！

> 注意：关于在一个组件内你是否可以使用 Hooks 有一些特殊的规则。我们将会在 [Rule of Hooks](https://reactjs.org/docs/hooks-rules.html) 学到他们。

### 声明一个状态变量

在类内，我们初始化`count`状态为0，通过在构造器设置`this.state`到`{ state:0 }`：
```jsx harmony
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }
```
在一个函数组件，我们没有`this`，所以我们无法赋值或者读取`this.state`。相反，我们在我们的组件内部直接调用`sueState` Hook：
```jsx harmony
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
``` 

**调用 useState 做什么？** 它声明一个"状态变量"。我们的变量叫做`count`但是我们可以叫做任何东西，比如`banana`。这是在函数调用之间"保存"一些值的方式 -- `useState` 是使用和类内提供`this.stat`能力完成一样的新的方式。通常，变量"消失"，当函数推出但是状态变量被 React 保留。

**我们传递啥作为 useState 的参数？** `useState()`唯一的参数是初始状态。和类不同，状态不必是个对象。我们可以保存一个数字或者字符串，如果我们只需要这个。在我们的例子中，我们值要一个数字，表示用户点击的次数，所以为我们的变量传递`0`作为初始状态。（如果我们想要存储两个不同的值在状态中，我们可以调用`useState()`两次。）

**useState 返回啥？** 返回一对值：当前状态和更新它的函数。这就是为什么我们写`const [count, setCount] = useState()`。这和在类中的`this.state.count`和`this.setState`类似，除了你在一个对中获取他们。如果你对我们使用的语法不熟悉，我们将在这个页面的底部回顾它。

现在我们知道`setState` Hook 做啥，我们的例子应该更有意义：
```jsx harmony
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);
```

我们声明一个状态变量叫做`count`，并设置它为`0`。React 将会记住他当前的值，在每一次重新熏染，并提供最新的一次给我们的函数。如果我们想要更新当前的`count`，我们叫做`setCount`。

> 注意：你可能会像：为什么`useState`不叫做`createState`？"Create"并不十分精确因为状态只在我们组件的第一次渲染的时候创建。在下一次渲染，`useState`给我们当前的状态。否则就完全不是"state"！这也是为什么 Hook 总是以`use`开头命名的原因。我们将在 Rules of Hooks 中学到为啥。

### 读取状态
在类中，当我们想要显示当前计数的时候，我们读取`this.state.count`：
```jsx harmony
  <p>You clicked {this.state.count} times</p>
```
在函数中，我们可以直接使用`count`：
```jsx harmony
  <p>You clicked {count} times</p>
```

### 更新状态
在类中，我们需要调用`this.setState()`去更新`count`状态：
```jsx harmony
  <button onClick={() => this.setState({ count: this.state.count + 1 })}>
    Click me
  </button>
```
在一个函数中，我们已经有`setCount`和`count`作为变量，所以我们不需要`this`：
```jsx harmony
  <button onClick={() => setCount(count + 1)}>
    Click me
  </button>
```

### 回顾

现在让我们**一行一行回顾我们学到啥**并检查我们的理解。
```jsx harmony
 1:  import React, { useState } from 'react';
 2:
 3:  function Example() {
 4:    const [count, setCount] = useState(0);
 5:
 6:    return (
 7:      <div>
 8:        <p>You clicked {count} times</p>
 9:        <button onClick={() => setCount(count + 1)}>
10:         Click me
11:        </button>
12:      </div>
13:    );
14:  }
```

- **line 1**：我们从 React 导入一个`useState`Hook。让我们在函数组件中保存本地状态。

- **line 4**：在`Example`组件中，我们声明了一个新的状态变量叫做`useState`Hook。它返回一对值，赋值到我们给定的名字。我们把变量叫做`count`是因为它保存了按钮的点击次数。我们初始化为它为 0，通过传递 0 作为唯一的`useState`参数。第二个返回项是一个函数，它让我们更新`count`，所以我们叫他`setCount`。

- **line 9**：当用户点击的时候，我们使用一个新的值调用`setCount`。React 将会马上重新渲染`Example`组件，传递新的`count`值给它。 

这看起来需要先做很多工作。别急！如果你在解释中迷失，再次查看前面的代码并尝试从头到脚的阅读它。我们保持保证一旦你带着新眼光去审视代码，尝试去"忘记"类中状态是如何工作的，它将会有意义。

### 提示：方括弧意味着啥？
你可能注意到方括弧，当我们声明一个状态变量的时候：
```jsx harmony
  const [count, setCount] = useState(0);
```
左边的名字不是 React API 的一部分。你可以命名你自己的状态变量：
```jsx harmony
  const [fruit, setFruit] = useState('banana');
```
这个 JavaScript 语法叫做"array destructuring"。这意味着我们创建两个新的变量`fruit`和`setFruit`，`fruit`设置为`useState`返回的第一个值，`setFruit`是第二个。这和下面的代码相同：
````jsx harmony
  var fruitStateVariable = useState('banana'); // Returns a pair
  var fruit = fruitStateVariable[0]; // First item in a pair
  var setFruit = fruitStateVariable[1]; // Second item in a pair
````

当我们使用`useState`声明一个状态变量，它返回一对 -- 一个数组，它有两个项。第一项是当前的值，第二项是一个函数，让我们更新它。使用`[0]`和`[1]`去访问他们有一点迷惑因为他们有特定的意义。这就是我们使用结构替代的原因。

> 注意：你可能会好奇 React 怎样知道哪一个组件`useState`去对应，因为我们没有传递任何像`this`到 React。我们将会在 FAQ 章节回答这个问题和很多其他问题。

### 提示：使用多个状态变量

声明状态变量为`[something, setSomething]`很便利，因为它让我们为不同的状态变量赋予不同的名字，如果我们想要使用多个：
```jsx harmony
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```
在前面的组件，我们有`age`，`fruit`和`todos`作为本地变量，并且我们可以独立的更新他们：
```jsx harmony
  function handleOrangeClick() {
    // Similar to this.setState({ fruit: 'orange' })
    setFruit('orange');
  }
```

你**不需要**使用多个状态变量。状态变量也可以持有数据和对象，所以你依旧可以组合相关的数据。然而，不像类中的`this.setState`，更新状态变量总是替代它而不是合并它。

我们在 FAQ 提供更多关于分离独立状态变量的建议。

### 下一步
在这个页面我们学习了 React 提供的一个 Hook，叫做`setState`。有时候我们也称之为"State Hook"。它让我们添加本地状态到 React 函数组件 -- 前面我们第一次做的那样！

我们也学到了一些关于什么是 Hooks。Hooks 是函数，让你从函数内"hook into"React 功能他们的名字总是以`use`开头，并且还有很多 Hooks 没有见过。

**现在，继续学习下一个 Hook：useEffect。** 它让你在组件中执行副作用，并且和类中的生命周期方法类似。
