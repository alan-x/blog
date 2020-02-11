[原文地址](https://reactjs.org/docs/hooks-intro.html)
### 介绍 Hooks

Hooks 是 React 16.8 引入的新内容。它们允许你不写一个类来使用状态和其他 React 功能。

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
`usestate`这个新函数是我们将要学习的第一个"Hook"。如果它没有意义请不要担心。

**你可以在下一个页面开始学习 Hooks**。在这个页面，我们将会继续解释为什么我们添加 Hooks 到 React 和他们是如何帮助你写出棒棒的应用。

> 注意：React 16.8.0 是支持 Hooks 的第一版本。当升级的时候，不要忘了升级所有的包，包括 React Dom。[React Native 从 0.59 开始支持 Hooks](https://facebook.github.io/react-native/blog/2019/03/12/releasing-react-native-059)。

### 视频介绍

在 React Conf 2018，Sophie Alpert 和 Dan Abramov 介绍了 Hooks，然后 Ryan Florence 示范了怎样重构一个应用去使用他们。在[这里](https://youtu.be/dpw9EHDh2bM)观看视频：

### 无破坏性改变改变

在我们继续之前，注意 Hooks 是：
- **完全可选**。你可以尝试在少量组件使用 Hooks，而不需要重写任何已经存在的代码。但是如果你不想要，你不必马上学习或者使用 Hooks。

- **100% 向后兼容**。Hooks 不包含任何中断改变。

- **立马可用**。Hook 在 v16.8.0 就可以使用。

**没有从 React 移除类的计划**。你可以在这个页面的底部章节阅读更多关于渐近引入 Hooks 的策略。

**Hooks 没有替换你关于 React 理念的知识**。相反，Hooks 提供一个你已经知道的 React 理念更加直接的 API：props，state，context，refs，和 lifecycle。正如接下来展示的，Hooks 也提供了新的强大的方式去结合他们。

**如果你只是要开始学习 Hooks，尽管直接跳转到下一个页面**！你也可以继续阅读这个页面去学习更多关于为什么我们要添加 Hooks，和如何在不重写我们应用的情况下开始使用他们。


### 动机

Hooks 解决了 React 中五年多来编写和维护数以万计的组件的一系列看起来不相关的问题。不管你是正在学习 React，每天使用它，或者更偏爱其他有类似组件模型的库，你可能意识到这些问题。

#### 很难在组件间复用有状态的逻辑

React 没有提供一个方式去"附加"可重用行为到组件（比如，连接它到一个 store）。如果你使用 React 一段时间，你可能会对 render props 和 higher-order components 这类尝试去解决这个问题的模式。但是这些模式在使用的时候，需要你去调整你的组件，这会很麻烦，并且让代码很难跟踪。如果你在 React DevTools 中查看一个典型的 React 应用，你可能找到一个由 provider 层，consumer，higher-order components，render props，和其他抽象概念包裹的组件的"wrapper hell"，尽管我们可以在 DevTools 中过滤他们，但是这指出了一个更底层的问题：React 需要一个更好的原语实现分享状态逻辑。

使用 Hooks，你可以从一个组件提取状态逻辑，这样它可以被单独测试和重用。**Hooks 允许你去重用有状态的逻辑，而不需要去改变你的组件层级**。这让它更简单的在很多组件和社区中去分享 Hooks。

我们将在[构建你自己的的 Hooks](https://reactjs.org/docs/hooks-custom.html)中讨论这个。

#### 复杂的组件开始变得更难理解

我们通常需要去维护组件，它一开始很简单，但是慢慢变成不能管理的混乱的状态逻辑和副作用。每一个生命周期方法通常包含混合的不相关的逻辑。比如，组件可能在`componentDidMount`和`componentDidUpdate`执行一些数据获取。然而，同一个`componentDidMount`可能包含一些不相关的逻辑，比如设置事件监听，并且在`componentWillUnmount`执行清理。一起更改的相互关联的代码被分离，但是完全不相关的代码被结合到一个方法。这很容易导致 bug 和不一致。
 
在很多场景中，很难去分离这些组件到更小的组件中，因为状态逻辑在所有的地方都存在。也很难测试他们。这也是很多人更偏向于去将 React 和一个分离的状态管理库结合。然而，这通常引入更多抽象，需要你在不同的文件中间跳转，让组件重用更加困难。
 
 为了解决这个，**Hooks 让你基于相关的部分分离一个组件到更小的函数（比如设置监听或者获取数据）**，而不是基于生命周期方法强制分离。你也可能可选的使用一个 reducer 去管理组件的本地状态，让它更加可预见。
 
 我们将会在 Using the Effect Hooks 继续讨论这个。
 
 #### 类让人和机器迷惑
 
 除了让代码重用和代码管理更加困难，我们发现类是学习 React 的一大障碍。你需要去理解`this`在 JavaScript 中是如何工作的，这和大部分其他语言可能不太一样。你需要去记住去绑定事件处理器。没有不稳定的[语法提案](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/)，代码非常冗长。人们可以很好的理解属性，状态，和自顶向下的数据流，但是依旧和类争斗。React 中函数和类组件的区别和何时使用他们甚至在有经验的 React 开发者之间都导致分歧。
 
此外，React 已经推出五年了，我们希望确保它和下一个五年保持联系。正如 [Svelte](https://svelte.dev/)，[Angular](https://angular.io/)，[Glimmer](https://glimmerjs.com/)，和其他展现的，组件的 [ahead-of-time compilation](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) 有很大的发展潜力。特别是如果它不限制于模板。最近，我们使用 [Prepack](https://prepack.io/) 实验 [component folding](https://github.com/facebook/react/issues/7323)，并且看到了预期的早期结果。然而，我们发现类组件会鼓励无意义模式，让这些优化回滚到一个更慢的方式。类在现在的工具也存在问题。比如，类没有很好的压缩，让热重载块状化和不可信赖。我们希望存在一个 API，尽可能让代码保持在可压缩的路径。
 
为了解决这些问题，**Hooks 让你不使用类使用更多 React 功能**。理念上，React 组件更加接近函数。Hooks 拥抱函数，但是不违背 React 的实践精神。Hooks 提供访问紧急逃生舱的方式，不需要你去学习复杂的功能或者响应式编程技术。
 
> 例子：[Hooks at a Glance](https://reactjs.org/docs/hooks-overview.html) 是开始学习 Hooks 的好地方。
 
### 渐近引入策略
 
> TLDR：没有从 React 引入类的计划。
 
我们知道 React 开发者聚焦于产品开发，没有时间去查阅每一个新发布的 API。Hooks 是非常新的，最好在考虑学习或者引入他们之前等待更多例子和教程。
 
 
我们也知道添加一个新的原语到 React 的难度非常高。对于好奇的读者，我们准备了一个[详细的 RFC](https://github.com/reactjs/rfcs/pull/68)，更加详细的解释动机，针对指定设计决策和相关的先前的技术提供额外的观点。

**重要的是，Hooks 和存在的代码同时工作，这样你可以渐近引入他们**。不要急于升级 Hooks。我们推荐避免"大重写"，特别是对已存在的，复杂的类组件。在开始"思考 Hooks"之前，它需要一些思维转换。在我们的经验中，最好在新的和不重要组件先实践，并确保你团队的每一个人都感觉舒服。在你给 Hooks 一个尝试的时候，请自由的[给我们一个反馈](https://github.com/facebook/react/issues/new)，正面的和负面的。

我们希望 Hooks 去覆盖所有已经存在的用户场景，但是**我们在可预见的未来将会保持对类组件的支持**。在 Facebook，我们有数以万计的组件使用类写的，并且我们绝对没有计划去重写他们。相反，我们开始在新代码中使用 Hooks， 和类一起。

### 常见问题

我们准备了一个 [Hooks FAQ page](https://reactjs.org/docs/hooks-faq.html) 回答大部分关于 Hooks 的常见问题。

### 下一步

在这个页面的底部，你应该有一个关于 Hooks 解决什么问题的大概的想法，但是很多细节可能不太清楚。不要担心！**现在到[下一个页面](https://reactjs.org/docs/hooks-overview.html)，开始通过例子学习 Hooks。**

