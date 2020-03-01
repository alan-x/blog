[原文地址](https://reactjs.org/docs/jsx-in-depth.html)
### 深入 JSX

基本上，JSX 只提供`React.createElement(component, props, ... children)`函数的语法糖。JSX 代码：
```jsx harmony
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```
编译为：
```jsx harmony
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```
如果没有子元素，你可以使用子关闭的形式。因此：
```jsx harmony
<div className="sidebar" />
```
编译为：
```jsx harmony
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
```
如果你想要测试一些指定 JSX 转化到 JavaScript，你可以尝试[在线 Babel 编译器](https://babeljs.io/repl/#?presets=react&code_lz=GYVwdgxgLglg9mABACwKYBt1wBQEpEDeAUIogE6pQhlIA8AJjAG4B8AEhlogO5xnr0AhLQD0jVgG4iAXyJA)。


### 指定 React 元素类型

JSX 标签的第一部分指定 React 元素的类型。

大写的类型指定 JSX 标签引用一个 React 组件。这些标签直接编译到命名的变量，因此，如果你使用 JSX `<Foo />`表达式，`Foo`必须在范围内。

### React 必须在范围中

因为 JSX 编译为`React.createElement`调用，`React`库必须总是在你的 JSX 代码的范围内。

比如，尽管`React`和`CustomButton`没有被 JavaScript 直接引用，引入这两者也是必须的：
```jsx harmony
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```

如果你没有使用 JavaScript 打包器并使用`<script>`标签加载，它已经作为`React`全局可用。

### 使用点符号表示 JSX 类型

你也可以时候点符号在 JSX 内引用 React 组件。如果你有一个模块导出多个 React 组件，这很方便。比如，如果`MyComponents.DatePicker`是一个组件，你可以直接在 JSX 中这么使用：
```jsx harmony
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}

```

### 用户定义的组件必须大写

当一个元素类型以小写字母开头，它引用类似`<div>`或者`<span>`之类的内置组件并导致一个字符串`'div'`或者`'span'`传递到`React.createElement`。使用大写字母开头的类型，类似`<Foo />`编译为`React.creatElement(Foo)`表示在你的 JavaScript 文件定义或者引入的一个组件。

我们推荐使用使用一个大写字母来命名组件。如果你有一个组件实现用小写字母，在 JSX 使用它之前将它赋值到一个大写变量。

比如，这个代码不会如期运行：

```jsx harmony
import React from 'react';

// Wrong! This is a component and should have been capitalized:
function hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Wrong! React thinks <hello /> is an HTML tag because it's not capitalized:
  return <hello toWhat="World" />;
}
```

为了修复这个，我们将重命名`hello`到`Hello`，并使用`<Hello />`引用它：
```jsx harmony
import React from 'react';

// Correct! This is a component and should be capitalized:
function Hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Correct! React knows <Hello /> is a component because it's capitalized.
  return <Hello toWhat="World" />;
}
```

### 在运行时选择类型

你不能使用一个普通表达式作为 React 元素类型。如果你的确想要使用普通表达式去指定元素的类型，先赋值他到一个大写变量。当你想要基于属性渲染一个不同组件的时候常用：
```jsx harmony
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Wrong! JSX type can't be an expression.
  return <components[props.storyType] story={props.story} />;
}
```

为了修复这个，我们可以赋值类型到大写变量：
```jsx harmony
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

### JSX 中的属性

在 JSX 中指定属性有多种方式。

### JavaScript 表达式作为属性

你可以传递任何 JavaScript 表达式作为属性，通过使用`{}`包裹。比如，在 JSX 中：
```jsx harmony
<MyComponent foo={1 + 2 + 3 + 4} />
```

对于`MyComponent`，`props.foo`的值将会是`10`，因为表达式`1 + 2 + 3 + 4 + 5`被执行了。

`if`语句和`for`循环在 JavaScript 中不是表达式，因此他们不能直接在 JSX 中直接使用。相反，你可以在包裹代码中使用。比如：
```jsx harmony
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

你可以在对应的章节学习更多关于[条件化渲染](https://reactjs.org/docs/conditional-rendering.html)和[循环](https://reactjs.org/docs/lists-and-keys.html)。

### 字符串字面量

你可以传递字符串字面量作为属性。这两个 JSX 表达式是相同的：
```jsx harmony
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

当你传递字符串字面量，它的值是非转移的 HTML。因此这两个 JSX 表达式是相同的。

```jsx harmony
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

这种行为通常是无关的。这里只是为了完备性提及。

### 属性默认是 true

如果你没有传递任何值给属性，它默认是`true`。这两个 JSX 表达式是相同的：
```jsx harmony
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

通常，我们不推荐使用这个，因为它会和[ES6 对象缩写]()混淆，`{foo}`表示`{foo: foo}`，而不是`{foo: true}`。这个行为只是为了和 HTML 匹配。

### 属性展开

如果你已经有`props`作为对象，你想要传递到 JSX，你可以使用`...`作为"展开"操作符去传递整个属性对象。这两个组件是相等的：
```jsx harmony
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```
当使用扩展操作符传递其他属性时候，你也可以选择组件将会消费的特定属性。

```jsx harmony
const Button = props => {
  const { kind, ...other } = props;
  const className = kind === "primary" ? "PrimaryButton" : "SecondaryButton";
  return <button className={className} {...other} />;
};

const App = () => {
  return (
    <div>
      <Button kind="primary" onClick={() => console.log("clicked!")}>
        Hello World!
      </Button>
    </div>
  );
```

在前面的例子中，`king`属性是安全消费的，它不会传递到 DOM 的`<button>`元素中。所有的其他属性通过`...other`对象传递，让这个组件更加灵活。你可以看到它传递一个`onClick`和`children`属性。

展开属性很有用，但是也容易传递不需要的属性到组件，或者传递非法的 HTML 属性到 DOM。我们推荐少使用这个语法。

### JSX 中的子元素

在 JSX 表达式中，包含一个开放标签和一个关闭标签，两个标签之间的内容作为特殊的属性传递：`props.children`。有多种不同方式传递子元素：

### 字符串字面量

你可以将一个字符串放到开始和关闭标签之间，`props.children`将会是这个字符串。这对于很多内置 HTML 元素很有用。比如：
```jsx harmony
<MyComponent>Hello world!</MyComponent>
```
这是有效的 JSX，`MyComponent`的`props.children`将会是简单的字符串`"Hello world!"`。HTML 是未转译的，因此你可以像编写 HTML 一样编写 JSX。

```jsx harmony
<div>This is valid HTML &amp; JSX at the same time.</div>
```
JSX 移除行首和行尾的空白。也移除空白行。标签附近新行被移除；字符串字面量中间出啊先的新航将被压缩为单个空格。因此，下面渲染同一个东西：
```jsx harmony
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX 子元素

你可以使用多个 JSX 元素作为子元素。这对于显示嵌套组件很有用：
```jsx harmony
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

你可以混合多种类型的子元素，因此你可以使用字符串字面量和 JSX 子元素。这是 JSX 和 HTML 很像的另一个方式，这是有效的 JSX 和 HTML：
```jsx harmony
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

一个 React 组件可以返沪一个数组的元素：
```jsx harmony
render() {
  // No need to wrap list items in an extra element!
  return [
    // Don't forget the keys :)
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
```

### JavaScript 表达式作为子元素

你可以传递 JavaScript 表达式作为子元素，通过在`{}`封闭它。比如，这些表达式是相等的：
```jsx harmony
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

这对于渲染人一场的列表的 JSX 表达式很有用。比如，这首先渲染一个 HTML 列表：
```jsx harmony
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

JavaScript 表达式可以和其他类型的子元素混合。这通常在部署字符串模板很有用：
```jsx harmony
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

### 函数作为子元素

通常，JavaScript 表达式插入到 JSX 将执行到一个字符串，一个 React 组件，或者一个列表的这些东西。然而，`props.children`和其他属性一样工作，他可以传递任何类型的数据，不只是 React 知道如何渲染的那些。比如，如果你有一个自定义组件，你可以让他接收一个回调作为`props.children`：
```jsx harmony
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```
传递到自定义组件的子元素可以是任何东西，只要组件转化他们到一些 React 可以理解的。这中使用不常见，但是它有效，如果你想要扩展 JSX 的能力。

### Boolean，Null，和 Undefined 被忽略

`false`，`null`，`undefined`，和`true`都是有效的子元素。他们直接不渲染。这些 JSX 表达式将会渲染相同的东西：
```jsx harmony
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

这对于条件化渲染 React 元素很有用。这个 JSX 只有在`showHeader`是`true`的时候渲染`<Header />`组件。

```jsx harmony
<div>
  {showHeader && <Header />}
  <Content />
</div>
```

有一个警告是["falsy"值]()，比如`0`数字，依旧被 React 渲染。比如，这个代码将不是你表现的那样，因为`0`将会在`props.message`是空数组的时候打印：
```jsx harmony
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```
为了修复这个，确保`&&`前的表达式总是 boolean：
```jsx harmony
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```
相反，如果你想要值类似`false`，`true`，`null`，或者`undefined`出现在输出，你要先[转化到字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#String_conversion)：
```jsx harmony
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```








