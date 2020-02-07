[原文地址](https://reactjs.org/docs/lists-and-keys.html)
### 列表和 key

首先，回顾一下 JavaScript 如何转化一个列表

给定下面的代码。我们使用[map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)函数去接受一个`numbers`数组，并翻倍他们的值。我们赋值`map()`返回新的数组到变量`doubled`并记录它：
```jsx harmony
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map((number) => number * 2);
console.log(doubled);
```
这些代码打印`[2, 4, 6, 8, 10]`到控制台。

在 React 中，转化数组到[元素](https://reactjs.org/docs/rendering-elements.html)列表基本相等。

### 渲染多个组件

你可以构建一个集合的元素，并使用花括号`{}`[包含他们到 JSX](https://reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)。

下面，我们使用 JavaScript 的[man()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)函数遍历`numbers`数组。我们为每一个项返回一个`<li>`元素。最终，我们赋值元素数组到`listItems`：
```jsx harmony
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li>{number}</li>
);
```

我们包含完整的`listItems`数组到一个`<ul>`元素，并[渲染它到 DOM](https://reactjs.org/docs/rendering-elements.html#rendering-an-element-into-the-dom)：
```jsx harmony
ReactDOM.render(
  <ul>{listItems}</ul>,
  document.getElementById('root')
);

```

[在 CodePen 中尝试](https://codepen.io/gaearon/pen/GjPyQr?editors=0011)

这段代码显示 1 到 5 之间的数字列表。

### 基本列表组件

通常你会在一个[组件](https://reactjs.org/docs/components-and-props.html)内渲染列表。

我们可以重构前面的例子到一个组件，接受一个数组的`numbers`并输出一个元素列表。
```jsx harmony
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li>{number}</li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
当你运行这个代码，你将会得到一个警告，列表的每一项都应该提供一个 key。一个"key"是一个特殊字符串属性，当创建元素列表的时候，你需要包含它。我们将在下一个章节讨论为什么这很重要。
 
在`numbers.map()`内赋值一个`key`到我们的列表项并修复 key 丢失的问题。
```jsx harmony
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <li key={number.toString()}>
      {number}
    </li>
  );
  return (
    <ul>{listItems}</ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/jrXYRR?editors=0011)

### Keys

key 帮助 React 标识示哪一个项改变了，添加了，或者移除了。key 应该在数组内给元素一个稳定的标识：
```jsx harmony
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) =>
  <li key={number.toString()}>
    {number}
  </li>
);
```
获取一个 key 最好的方式是使用一个字符串标识唯一的列表项。通常你使用你的数据的 ID 作为 key：
```jsx harmony
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
);
```
当你没有稳定的 ID 作为渲染项，你可能使用项目索引作为作新排序的 key：
```jsx harmony
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
);
```

我们不推荐使用索引作为 key，如果项目的顺序可能改变。这会消极的导致性能问题并且可能导致组件状态问题。查阅 Robin Pokorny 的文章，为了[更深入解释使用 index 作为 key 的消极影响]()。如果你选择不去赋值一个明确的 key 给列表项，则 React 将会默认使用索引作为 key。

这是一个[深入解释为什么 key 是必须的](https://reactjs.org/docs/reconciliation.html#recursing-on-children)，如果你有兴趣了解更多。

### 使用 key 抽取组件

key 只有在数组周围的上下文才有意义。

比如，如果你[抽取](https://reactjs.org/docs/components-and-props.html#extracting-components)一个`ListItem`组件，你应该保持 key 在数组中的`<ListItem />`元素，而不是在`ListItem`的`<li>`元素。

#### 例子：不正确的 key 使用
```jsx harmony
function ListItem(props) {
  const value = props.value;
  return (
    // Wrong! There is no need to specify the key here:
    <li key={value.toString()}>
      {value}
    </li>
  );
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Wrong! The key should have been specified here:
    <ListItem value={number} />
  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```

#### 例子：正确的 key 使用
```jsx harmony
function ListItem(props) {
  // Correct! There is no need to specify the key here:
  return <li>{props.value}</li>;
}

function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    // Correct! Key should be specified inside the array.
    <ListItem key={number.toString()}
              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}

const numbers = [1, 2, 3, 4, 5];
ReactDOM.render(
  <NumberList numbers={numbers} />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/ZXeOGM?editors=0010)

一个好的规则是`map()`内的元素调用需要 key。

### key 只在兄弟间保持唯一

数组内使用的 key 应该在他们的兄弟间唯一。然而他们不需要是全局唯一。当我们产生两个不同的数组，我们可以使用相同的 key：

```jsx harmony
function Blog(props) {
  const sidebar = (
    <ul>
      {props.posts.map((post) =>
        <li key={post.id}>
          {post.title}
        </li>
      )}
    </ul>
  );
  const content = props.posts.map((post) =>
    <div key={post.id}>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
  return (
    <div>
      {sidebar}
      <hr />
      {content}
    </div>
  );
}

const posts = [
  {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
  {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];
ReactDOM.render(
  <Blog posts={posts} />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/NRZYGN?editors=0010)

key 是给 React 的提示，他们不传递给你的组件。如果你的组件需要相同的只，显式使用不同名字传递为一个属性：
```jsx harmony
const content = posts.map((post) =>
  <Post
    key={post.id}
    id={post.id}
    title={post.title} />
);
```
使用前面的例子，`Post`组件可以读取`props.id`，但是不能读取`props.key`。

### 在 JSX 中嵌入 map()

在前面的例子中，我们声明一个分离的`listItems`变量并包含它到 JSX：
```jsx harmony
function NumberList(props) {
  const numbers = props.numbers;
  const listItems = numbers.map((number) =>
    <ListItem key={number.toString()}
              value={number} />

  );
  return (
    <ul>
      {listItems}
    </ul>
  );
}
```
JSX 允许在花括号内[嵌入任何表达式](https://reactjs.org/docs/introducing-jsx.html#embedding-expressions-in-jsx)，因此我们可以内联`map()`结果：
```jsx harmony
function NumberList(props) {
  const numbers = props.numbers;
  return (
    <ul>
      {numbers.map((number) =>
        <ListItem key={number.toString()}
                  value={number} />

      )}
    </ul>
  );
}
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/BLvYrB?editors=0010)
有时候这导致更清晰的代码，但是这个风格也是一种滥用。比如在 JavaScript 中，这取决于你认为为了可读性抽取到一个变量是否值得。记住，如果`map()`体太复杂，是时候去[抽取一个组件](https://reactjs.org/docs/components-and-props.html#extracting-components)。







