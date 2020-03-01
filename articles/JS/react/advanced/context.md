[原文地址](https://reactjs.org/docs/context.html)
### Context

Context 提供了一个跨越组件树传输数据的方式，不需要在每一级手动传输数据。

在一个典型的 React 应用，数据通过属性从顶部到底部传输，但是这对于某些类型的属性很沉重（比如，本地化偏好，UI 主体），在一个应用内被很多组件需要的。Context 提供在组件间分享值的方式，不需要明确在树的每一级传输属性。

### 什么时候使用 Context

Context 设计用于分享数据，对于 React 组件树可以认为是"全局"，比如当前认证的用户，主题，或者偏好语言。比如，下面的代码中，我们手动传输一个"theme"属性为了装饰 Button 组件：
```jsx harmony
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  // The Toolbar component must take an extra "theme" prop
  // and pass it to the ThemedButton. This can become painful
  // if every single button in the app needs to know the theme
  // because it would have to be passed through all components.
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```
使用 context，我们可以避免通过中间元素传递属性：
```jsx harmony
// Context lets us pass a value deep into the component tree
// without explicitly threading it through every component.
// Create a context for the current theme (with "light" as the default).
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Use a Provider to pass the current theme to the tree below.
    // Any component can read it, no matter how deep it is.
    // In this example, we're passing "dark" as the current value.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// A component in the middle doesn't have to
// pass the theme down explicitly anymore.
function Toolbar(props) {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // Assign a contextType to read the current theme context.
  // React will find the closest theme Provider above and use its value.
  // In this example, the current theme is "dark".
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

### 在你使用 Context 之前

Context 主要用于在不同嵌套级别的组件需要访问数据的时候。很少使用它是因为它让组件重用更加艰难。

**如果你只是为了避免让一些属性穿过多个层级，[组件组合](https://reactjs.org/docs/composition-vs-inheritance.html)是比 context 更常见更简单的解决方案。**

比如，假设一个`Page`组件向下多个层级传递一个`user`和`avatarSize`属性，这样深层嵌套的`Link`和`Avatar`组件可以读取它：

```jsx harmony
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout user={user} avatarSize={avatarSize} />
// ... which renders ...
<NavigationBar user={user} avatarSize={avatarSize} />
// ... which renders ...
<Link href={user.permalink}>
  <Avatar user={user} size={avatarSize} />
</Link>
```

向下传递只有最后面的`Avatar`组件真的读取它的`user`和`avatarSize`，这可能会让人觉得冗余。这也表明当`Avatar`组件需要从顶部获取更多属性时，你必须在所有中间曾添加他们。

**不使用 context**解决这个问题的一种方式是[传递 Avatar 组件自己](https://reactjs.org/docs/composition-vs-inheritance.html#containment)，这样中间组件不需要知道关于`user`和`avartarSize`属性：
```jsx harmony
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// Now, we have:
<Page user={user} avatarSize={avatarSize} />
// ... which renders ...
<PageLayout userLink={...} />
// ... which renders ...
<NavigationBar userLink={...} />
// ... which renders ...
{props.userLink}
```
使用这种变化，只有最顶级的 Page 组件需要知道`Link`和`Avatar`组件使用`user`和`avatarSize`。

这种*控制倒置*让你的代码在很多场景中更加清晰，通过减少你需要穿过你的应用的属性的数量，并且给根组件更多的控制。然而，这不适用于每一个场景：移动复杂性到树的更高处让这些更高的组件更复杂，并强制低级别的组件比你想要的更灵活。

你不限制一个组件只有一个子元素。你可能传递多个子元素，甚至自元素还有多个分离的"插槽"。[正如这里记录的](https://reactjs.org/docs/composition-vs-inheritance.html#containment)：
```jsx harmony
function Page(props) {
  const user = props.user;
  const content = <Feed user={user} />;
  const topBar = (
    <NavigationBar>
      <Link href={user.permalink}>
        <Avatar user={user} size={props.avatarSize} />
      </Link>
    </NavigationBar>
  );
  return (
    <PageLayout
      topBar={topBar}
      content={content}
    />
  );
}
```

这个模式对于很多你想要分离子组件和它的直接父组件的场景已经足够了，你可以使用[渲染属性](https://reactjs.org/docs/render-props.html)更加深入，如果子元素需要需要在渲染之后钱和父元素交流。

然而，有时候相同的数据需要被树中的很多组件访问，并且有不同的嵌套级别。Context 让你"广播"这类数据，并改变它，在之后的所有组件。使用 context 比替换方案更简单的常见例子是管理当前的本地化，主题，或者数据缓存。

### API

### React.createContext
```jsx harmony
const MyContext = React.createContext(defaultValue);
```

创建一个 Context 对象。当 React 渲染一个定于这个 Context 对象的组件的时候，它会读取当前 context 的值，从它前面的树最近匹配的`Provider`。

`defaultValue`参数只有在组件没有从它所在的树的前面匹配 Provider 的时候使用。这对于在独立没有包裹他们的时候测试组件很有帮助。注意：传递`undefined`作为 Provider 值不会让消费者组件使用`defaultValue`。

### Context.Provider

```jsx harmony
<MyContext.Provider value={/* some value */}>
```
每一个 Context 对象有一个 Provider React 组件，允许消费组件去订阅 context 的改变。

接收一个`value`属性传递给 Provider 后续的消费组件。一个 Provider 可以被很多消费者连接。Provider 可以在树内嵌套覆盖值。

Provider 后代的所有消费者将会重新渲染，当 Provider 的`value`属性变化的时候。从 Provider 到它的后代消费者（包括[.contextType](https://reactjs.org/docs/context.html#classcontexttype)和[useContext](https://reactjs.org/docs/hooks-reference.html#usecontext)）的传播不受`shouldComponentUpdate`方法的约束，因此就算一个祖先组件跳过更新，消费者也会被更新。

改变由比较新的值和旧的值决定的，比较算法和[Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is#Description)一样。

注意：改变决定的方式会导致一些问题，当传递对象作为`value`的时候：查阅[警告](https://reactjs.org/docs/context.html#caveats)。

### Class.contextType
```jsx harmony
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* perform a side-effect at mount using the value of MyContext */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* render something based on the value of MyContext */
  }
}
MyClass.contextType = MyContext;
```

一个类中的`contextType`属性可以被赋值为一个 [React.createContext()](https://reactjs.org/docs/context.html#reactcreatecontext) 创建的 Context 对象，这让你使用`this.context`消费最近的 Context 类型的当前值。你可以在任何生命周期方法甚至是 render 函数这么引用它。

注意：
你只能使用这个 API 去订阅单个 context。如果你需要读取多个 context，查阅[消费多个 Context](https://reactjs.org/docs/context.html#consuming-multiple-contexts)。

如果你使用试验性的[public class fields syntax](https://babeljs.io/docs/plugins/transform-class-properties/)，你可以使用一个**static**类域去初始化你的`contextType`。

```jsx harmony
class MyClass extends React.Component {
  static contextType = MyContext;
  render() {
    let value = this.context;
    /* render something based on the value */
  }
```


### Context.Consumer
```jsx harmony
<MyContext.Consumer>
  {value => /* render something based on the context value */}
</MyContext.Consumer>
```

一个订阅 context 变化的 React 组件。这让你在一个[函数组件内](https://reactjs.org/docs/components-and-props.html#function-and-class-components)订阅一个 context。

需要一个[函数作为子组件](https://reactjs.org/docs/render-props.html#using-props-other-than-render)。函数接收当前 context 的值并返回一个 React 节点。传递给函数的参数`value`参数将会和树中前面最近的 Provider 的`value`属性相等。如果这个 context 之前没有 Provider，`value`参数将会和传递给`createContext()`的`defaultValue`值相等。

注意：
了解更多关于'函数作为子元素'模式的信息，查阅[渲染属性](https://reactjs.org/docs/render-props.html)。


### Context.displayName

Context 对象接收一个`displayName`字符串属性。React DevTool 使用这个字符串去决定为这个 context 显示什么。

比如，下面的组件将会以 MyDisplayName 出现在 DevTools 中：
```jsx harmony
const MyContext = React.createContext(/* some value */);
MyContext.displayName = 'MyDisplayName';

<MyContext.Provider> // "MyDisplayName.Provider" in DevTools
<MyContext.Consumer> // "MyDisplayName.Consumer" in DevTools
```

### 例子

### 动态 Context
一个使用动态的主题值的更加复杂的例子：
**theme-context.js**
```jsx harmony
export const themes = {
  light: {
    foreground: '#000000',
    background: '#eeeeee',
  },
  dark: {
    foreground: '#ffffff',
    background: '#222222',
  },
};

export const ThemeContext = React.createContext(
  themes.dark // default value
);
```

**theme-button.js**
```jsx harmony
import {ThemeContext} from './theme-context';

class ThemedButton extends React.Component {
  render() {
    let props = this.props;
    let theme = this.context;
    return (
      <button
        {...props}
        style={{backgroundColor: theme.background}}
      />
    );
  }
}
ThemedButton.contextType = ThemeContext;

export default ThemedButton;
```

**app.js**
```jsx harmony
import {ThemeContext, themes} from './theme-context';
import ThemedButton from './themed-button';

// An intermediate component that uses the ThemedButton
function Toolbar(props) {
  return (
    <ThemedButton onClick={props.changeTheme}>
      Change Theme
    </ThemedButton>
  );
}

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      theme: themes.light,
    };

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));
    };
  }

  render() {
    // The ThemedButton button inside the ThemeProvider
    // uses the theme from state while the one outside uses
    // the default dark theme
    return (
      <Page>
        <ThemeContext.Provider value={this.state.theme}>
          <Toolbar changeTheme={this.toggleTheme} />
        </ThemeContext.Provider>
        <Section>
          <ThemedButton />
        </Section>
      </Page>
    );
  }
}

ReactDOM.render(<App />, document.root);

```

### 从一个嵌套的组件更新 Context
通常需要从一个组件树深度嵌套的组件中去更新 context。在这个场景中，你可以传递一个函数到 context，去允许消费者更新上下文。

**theme-context.js**
```jsx harmony
// Make sure the shape of the default value passed to
// createContext matches the shape that the consumers expect!
export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
```
**theme-toggler-button.js**
```jsx harmony
import {ThemeContext} from './theme-context';

function ThemeTogglerButton() {
  // The Theme Toggler Button receives not only the theme
  // but also a toggleTheme function from the context
  return (
    <ThemeContext.Consumer>
      {({theme, toggleTheme}) => (
        <button
          onClick={toggleTheme}
          style={{backgroundColor: theme.background}}>
          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemeTogglerButton;
```
**app.js**
```jsx harmony
import {ThemeContext, themes} from './theme-context';
import ThemeTogglerButton from './theme-toggler-button';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));
    };

    // State also contains the updater function so it will
    // be passed down into the context provider
    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme,
    };
  }

  render() {
    // The entire state is passed to the provider
    return (
      <ThemeContext.Provider value={this.state}>
        <Content />
      </ThemeContext.Provider>
    );
  }
}

function Content() {
  return (
    <div>
      <ThemeTogglerButton />
    </div>
  );
}

ReactDOM.render(<App />, document.root);
```
为了保证 context 重新渲染够快，React 需要去为这个 context 消费者在树中创建一个分离的节点。
```jsx harmony
// Theme context, default to light theme
const ThemeContext = React.createContext('light');

// Signed-in user context
const UserContext = React.createContext({
  name: 'Guest',
});

class App extends React.Component {
  render() {
    const {signedInUser, theme} = this.props;

    // App component that provides initial context values
    return (
      <ThemeContext.Provider value={theme}>
        <UserContext.Provider value={signedInUser}>
          <Layout />
        </UserContext.Provider>
      </ThemeContext.Provider>
    );
  }
}

function Layout() {
  return (
    <div>
      <Sidebar />
      <Content />
    </div>
  );
}

// A component may consume multiple contexts
function Content() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <UserContext.Consumer>
          {user => (
            <ProfilePage user={user} theme={theme} />
          )}
        </UserContext.Consumer>
      )}
    </ThemeContext.Consumer>
  );
}
```
如果两个或者多个 context 值通常一起使用，你可能想要考虑去创建你自己的渲染属性组件提供两者。

### 警告

因为 context 使用相同引用去决定何时重新渲染，有一些陷阱会触发消费者不再意图中的渲染，当提供者的父组件重新渲染。比如下面的代码在 Provider 重新渲染的时候每一次都会重新渲染所有的消费者，因为`value`总是一个新建的对象：
```jsx harmony
class App extends React.Component {
  render() {
    return (
      <Provider value={{something: 'something'}}>
        <Toolbar />
      </Provider>
    );
  }
}
```
为了解决这个，提升这个值到父组件的状态：
```jsx harmony
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: {something: 'something'},
    };
  }

  render() {
    return (
      <Provider value={this.state.value}>
        <Toolbar />
      </Provider>
    );
  }
}
```
### 遗留的 API
注意
React 之前提供了一个实验性的 context API。旧的 API 将会在所有 16.x 发行支持，但是使用它的应用应该升级到新版本。遗留的 API 将会在未来的 React 主版本移除。阅读[在这里的遗留的 context 文档](https://reactjs.org/docs/legacy-context.html)。
