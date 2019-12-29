### Using the Effect Hook

Hooks 是 React 16.8 引入的新内容。它们允许你不写一个类来使用状态和其他 React 功能。

Hooks 是 JavaScript 函数，但是使用他们的时候你需要遵循两个规则。我们提供一个 linter plugin 去自动推行这些规则：

### 只有在顶级调用 Hooks

**不要在循环，条件，或者嵌套函数内调用 loops。** 相反，总是在你的 React 函数顶部调用 Hooks。通过遵循这个规则，你确保 Hooks 在每一次组件渲染都以相同的顺序调用。这是 React 允许多个`useState`和`useEffect`调用正确保留。（如果你好奇，我们将会在下面更加深入解释）

### 只有在 React Function 内调用 Hooks

**不要在常规 JavaScript 函数内调用 Hooks。** 相反，你可以：
- 在 React 函数组件内调用 Hooks
- 在自定义 Hooks 内调用 Hooks（我们将在下一个页面学习他们）

通过遵循这个规则，你确保一个组件中的所有状态逻辑都在源代码中清晰可见。

### ESLint plugin
我们发布了一个 ESLint 插件叫做 eslint-plugin-react-hooks，推行这两个规则。你可以添加插件到你的项目，如果你去尝试它：
```jsx harmony
npm install eslint-plugin-react-hooks --save-dev
```
```jsx harmony
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

在未来，我们希望去包含这个插件到 Create React App 和相同的工具箱。

**现在你可以跳到下一个解释怎样编写你自己的 Hooks 页面**。在这个页面，我们将继续解释这些规则背后的原因。

### 解释

就像我们前面学到的，我们可以在一个组件内使用多个 State 或者 Effect Hook：
```jsx harmony
function Form() {
  // 1. Use the name state variable
  const [name, setName] = useState('Mary');

  // 2. Use an effect for persisting the form
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. Use the surname state variable
  const [surname, setSurname] = useState('Poppins');

  // 4. Use an effect for updating the title
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

所以 React 怎么知道哪一个状态对应哪一个`useState`调用？回答是**React 依赖 Hooks 调用的顺序。** 我们的例子可以工作是因为 Hook 调用的顺序在每一次顺序都一样：
```jsx harmony
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

只要 Hook 调用的顺序在渲染之间都相同，React 可以关联一些本地状态和他们的每一个。但是如果我们在一个条件内调用 Hook 会发生什么（比如，`persistForm` effect）?
```jsx harmony
  // 🔴 We're breaking the first rule by using a Hook in a condition
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
```

`name !== ''`在第一次渲染的时候为`true`，所以我们执行这个 Hook。然而，在下一次渲染，用户可能清理表单，让条件为`false`。现在我们在渲染间跳过 Hook，调用 Hook 的顺序开始不同：
```jsx harmony
useState('Mary')           // 1. Read the name state variable (argument is ignored)
// useEffect(persistForm)  // 🔴 This Hook was skipped!
useState('Poppins')        // 🔴 2 (but was 3). Fail to read the surname state variable
useEffect(updateTitle)     // 🔴 3 (but was 4). Fail to replace the effect
```

React 无法知道第二个`useState` Hook 调用返回啥。React 期待这个组件内第二个 Hook 调用对应`persistForm` effect，就像在前一个渲染期间，但是它不再是。从这个点开始，每一个我们跳过的下一个 Hook 调用，都会少一个，导致 bug。

**这也是为什么 Hooks 必须在我们组件的顶部嗲用**。如果我们想要按条件去执行一个 effect，我们可以将条件放在我们的 Hook 中。
```jsx harmony
  useEffect(function persistForm() {
    // 👍 We're not breaking the first rule anymore
    if (name !== '') {
      localStorage.setItem('formData', name);
    }
  });
```

**注意你不需要当心这个问题，如果你使用提供的 lint rule。** 但是现在你也知道为什么 Hook 按这种方式工作，和规则防止什么问题。

### 下一步

最后，我们准备去学习编写你自己的 Hooks！自定义 Hooks 让你结合 React 提供的 Hooks 到你自己的抽象，并在不同组件重用常见状态逻辑。
