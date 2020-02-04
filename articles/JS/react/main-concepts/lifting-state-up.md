### 状态提升

通常，多个组件需要反映相同的改变数据。我们推荐提升共享的状态到他们最近的组件。看看这是这么做到的把。

在这个章节，我们将创建一个温度计算器，计算水是否在某个给定稳定沸腾。

我们将从一个叫做`BoilingVerdict`的组件开始。它接受`celisus`温度作为属性，打印水是否足够沸腾：
```jsx harmony
function BoilingVerdict(props) {
  if (props.celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
}
```
下一步，我们将创建一个组件叫做`Calculator`。它渲染一个`<input>`让你输入温度，并保存它的值到`this.state.temperature`。

此外，它为当前输入值渲染`BoilingVerdict`。

```jsx harmony
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    return (
      <fieldset>
        <legend>Enter temperature in Celsius:</legend>
        <input
          value={temperature}
          onChange={this.handleChange} />

        <BoilingVerdict
          celsius={parseFloat(temperature)} />

      </fieldset>
    );
  }
}
```
[在 CodePen 中尝试它]()

### 添加第二个输入

我们新的需求是，除了一个 Celsius 输入，我们还提供一个 Fahrenheit 舒徐，并让他们保持同步。

我们可以开始从`Calculator`抽取一个`TemperatureInput`组件。我们将添加一个新的`scale`属性，可以是`"c"`或者`"f"`。

```jsx harmony
const scaleNames = {
  c: 'Celsius',
  f: 'Fahrenheit'
};

class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

我们现在可以改变`Calculator`去渲染两个分离的温度入参：
```jsx harmony
class Calculator extends React.Component {
  render() {
    return (
      <div>
        <TemperatureInput scale="c" />
        <TemperatureInput scale="f" />
      </div>
    );
  }
}
```

[在 CodePen 尝试它]

我们现在有两个输入框，但是当你输入温度到其中一个的时候，另一个不更新。这不符合我们的要求：我想要保持他们同步。

我们也不能从`Calculator`显示`BoilingVerdict`。`Calculator`不知道当前的温度，因为它藏在`TemperatureInput`。


### 编写转化函数

首先，我们将编写两个函数去转化 Celsius 和 Fahrenheit：
```jsx harmony
function toCelsius(fahrenheit) {
  return (fahrenheit - 32) * 5 / 9;
}

function toFahrenheit(celsius) {
  return (celsius * 9 / 5) + 32;
}
```
这两个函数转化数字。我们将编写另一个函数接受一个字符串的`temperature`和一个转化函数作为参数并返回一个字符串。我们将使用它基于一个输入去计算另一个输入。

它为无效的`temperature`返回一个空字符串，并保存输出为三位小数：
```jsx harmony
function tryConvert(temperature, convert) {
  const input = parseFloat(temperature);
  if (Number.isNaN(input)) {
    return '';
  }
  const output = convert(input);
  const rounded = Math.round(output * 1000) / 1000;
  return rounded.toString();
}
```
比如，`tryConvert('abc', toCelsius)`返回一个空字符串，`tryConvert('10.22', toFahrenheit)`返回`'50.396'`。

### 状态提升

现在，`TemperatureInput`组件独立保存他们的值到本地状态：
```jsx harmony
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.state = {temperature: ''};
  }

  handleChange(e) {
    this.setState({temperature: e.target.value});
  }

  render() {
    const temperature = this.state.temperature;
    // ...  
```
然而，我希望这两个输入框相互同步。当我们更新 Celsius 输入框，Fahrenheit 输入应该反映转化的温度，反过来也是。

在 React，分享状态通过移动它到需要它的组件的最近的公共祖先组件来完成。这叫做"状态提升"。我们将从`TemperatureInput`移除本地状态到`Calculator`。

如果`Calculator`拥有共享状态，它就变成两个输入框的温度的"数据源"。它可以指示他们有一致的值。因为`TemperatureInput`组件的属性都来自相同父组件`Calculator`组件，两个输入框将总是同步。

来一步步看看这是怎样工作的把。

第一，我们将在`TemperatureInput`中使用`this.state.temperature`替换`this.state.temperature`。对于现在，假装`this.props.temperature`已经存在，尽管之后我们需要从`Calculator`传递进来：

```jsx harmony
  render() {
    // Before: const temperature = this.state.temperature;
    const temperature = this.props.temperature;
    // ...
```

我们知道[props 是只读的]()。当`temperature`在本地状态，`TemperatureInput`可以只调用`this.setState()`去改变它。然而，现在`temperature`作为属性来自父组件，`TemperatureInput`无法控制它。

在 React，这通常通过让组件变成"受控"。可以解决。就像 DOM`input`接受`value`和`onChange`属性，因此，自定义的`TemperatureInput`接受来自`Calculator`的`temperature`和`onTemperatureChange`属性。

现在，当`TemperatureInput`想要去更新它的温度，它调用`this.props.onTemperatureChange`：

```jsx harmony
  handleChange(e) {
    // Before: this.setState({temperature: e.target.value});
    this.props.onTemperatureChange(e.target.value);
    // ...
```

注意：在自定义组件中，`temperature`或者`onTemperatureChange`属性名都没有特殊的意义。我们也可以把他们叫做任何其他东西，比如命名为`value`和`onChagne`也很常见方便。

`onTemperatureChange`属性和`temperature`属性将会同时被`Calculator`组件提供。它将处理改变，通过修改它的本地状态，因此将使用新的值去重新渲染两个输入框。我们很快将看到新的`Calculator`实现。

在深入`Calculator`的改变之前，回顾我们对`TemperatureInput`组件的改变。我们从它移除了本地状态，不再读取`this.state.temperature`，我们现在读取`this.props.temperature`。不是调用`this.setState()`，当我们创建改变的时候，我们现在调用`this.props.onTemperatureChange()`，他们将被`Calculator`提供：
```jsx harmony
class TemperatureInput extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange(e) {
    this.props.onTemperatureChange(e.target.value);
  }

  render() {
    const temperature = this.props.temperature;
    const scale = this.props.scale;
    return (
      <fieldset>
        <legend>Enter temperature in {scaleNames[scale]}:</legend>
        <input value={temperature}
               onChange={this.handleChange} />
      </fieldset>
    );
  }
}
```

现在转向`Calculator`组件。

我们将存储当前输入的`temperature`和`scale`到它的本地存储。这是我们从输入框中"提升"的状态，它将作为他们的"数据源"。这是为了渲染两个输入框需要知道的所有数据的最小表示。

比如，如果我们输入 37 到 Celsius 输入框，`Calculator`组件的状态将会是：
```jsx harmony
{
  temperature: '37',
  scale: 'c'
}
```

如果我们编辑 Fahrenheit 域 为 212，`Calculator`的状态将会是：
```jsx harmony
{
  temperature: '212',
  scale: 'f'
}
```
我们可以存储两个输入的值，但是这被证明是非必须的。存储最近改变的输入框和它表示的单位已经足够了。我们可以基于当前的`temperature`和`scale`值去推断其他输入框的值。

输入框保持同步是因为他们的值是从相同状态计算出来的：
```jsx harmony
class Calculator extends React.Component {
  constructor(props) {
    super(props);
    this.handleCelsiusChange = this.handleCelsiusChange.bind(this);
    this.handleFahrenheitChange = this.handleFahrenheitChange.bind(this);
    this.state = {temperature: '', scale: 'c'};
  }

  handleCelsiusChange(temperature) {
    this.setState({scale: 'c', temperature});
  }

  handleFahrenheitChange(temperature) {
    this.setState({scale: 'f', temperature});
  }

  render() {
    const scale = this.state.scale;
    const temperature = this.state.temperature;
    const celsius = scale === 'f' ? tryConvert(temperature, toCelsius) : temperature;
    const fahrenheit = scale === 'c' ? tryConvert(temperature, toFahrenheit) : temperature;

    return (
      <div>
        <TemperatureInput
          scale="c"
          temperature={celsius}
          onTemperatureChange={this.handleCelsiusChange} />

        <TemperatureInput
          scale="f"
          temperature={fahrenheit}
          onTemperatureChange={this.handleFahrenheitChange} />

        <BoilingVerdict
          celsius={parseFloat(celsius)} />

      </div>
    );
  }
}
```
[在 CodePen 中尝试它]()

现在，不论你编辑哪一个输入框，`this.state.temperature`和`this.state.scale`都被更新。一个输入框按原样取值，因此任何用户输入都将保留，另一个输入框基于它计算。

回顾以下我们编辑输入框的时候，发生了什么：

- React 调用函数`<input>`上指定为`onCHange`的函数。在我们的场景中，这是`TemperatureInput`组件上的`handleChange`方法。

- `TemperatureInput`组件的`handleChange`方法调用`this.props.onTemperatureChange()`，使用新的期待的值。它的属性，包括`onTemperatureChange`，通过它的父组件提供，`Calculator`。

- 当他前一次渲染的时候，`Calcularot`指定`TemperatureInput`的`onTemperatureChange`为`Calculator`的`handleCelsiusChange`方法，Farenheit 的`onTemperatureChange`方法为`Calculator`的`handleFahrenheitChange`方法。因此，这两个`Calculator`方法的调用取决于哪一个输入框我们编辑了。

- 在这些方法内，`Calculator`组件让 React 去重新渲染它自己，通过调用`this.setState()`，使用新的输入值和当前我们编辑的输入框的比例。

- React 调用`Calculator`组件的`render`方法去了解 UI 应该长什么样。输入框的值基于当前温度和活跃的比例来计算。温度转换在这里进行。

- React 调用分别使用`Calculator`指定的新的属性值去更新`TemperatureInput`组件。它知道 UI 应该长什么样。

- React DOM 更新 DOM，使用沸腾判断并去匹配期望的输入值。我们编辑饿的输入框接受它当前的额值，另一个输入框在转化之后更新温度。

每一个更新都经历相同的步骤，一次输入框保持同步。

### 课程学习

React 应用中的任何数据应该有单一的"数据源"。通常，状态先被添加到需要它去渲染的组件。然后没如果其他组件也需要它，你可以提升他们到最近的共同祖先。与其尝试在不同组件间同步组件，你应该依赖[自顶向下数据流]()。

状态提升相对于双向绑定方式，导致编写更多"样板"代码，但是作为好处，它较少寻找和解决 bug 的工作。因为任何状态"活在一些组件中，并且组件可以改变它，遭遇 bug 的表面积大大减少。此外，你可以实现任意自定义逻辑去拒绝转化用户输入。

如果一些东西可以从属性或者状态分发，它可能不应该在状态中。比如，与其存储`celsiusValue`和`fahrenheitValue`，我们存储最后编辑的`temperature`和它的`scale`。其他输入框的值总是可以在`render()`方法中被计算出来。这让我们清理或者应用绕过其他域，不丢失任何用户输入的任何精度。

当你看到 UI 上出现一些错误的时候，你可以使用[React Developer Tools]()去检查属性并向上查找树，知道你发现负责更新状态的组件。这让你跟踪 bug 到他们的源头：

![](https://reactjs.org/react-devtools-state-ef94afc3447d75cdc245c77efb0d63be.gif)

















