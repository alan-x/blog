### Reconciliation

React 提供了一个声明式的 API，这样你就完全不需要去担心每一次更新改变了什么。这让编写应用变得更加简单，但是他可能变得不明显，React 是如何实现的。这篇文章解释我们在 React 中"diffing" 算法做的选择，这样组件更新是可预料的，并且在高性能应用也足够快。

### 动机

当你使用 React，在某个单一的事件点，你可以认为`render()`函数创建了一棵树的 React 元素。在下一个装在或者属性更新的时候，`render()`函数将会返回一个不同的 React 元素树。React 需要指出怎样有效的更新 UI 去匹配最近的树。

这个算法问题有一些通用解决方案，可以生成最小数量的操作去转化一颗树到另一颗。然而，[最先进的算法]()大约有 O(n^3) 的复杂度，n 指的是树的数量。

如果我们在 React 中使用这个，显示 1000 个元素将会需要大约一百万次比较。这太昂贵了。作为替代，React 实现了一个启发式 O(n)  算法，基于两个假设：

1. 两个不同类型的元素将会产生不同的树。

2. 开发者可以使用一个`key`属性来表示哪一个自元素可能在多次不同渲染是稳定的。

实际上，这些假设对于大部分实际实现用场景都有效。


### Diff 算法

当对比两棵树的时候，React 首先比较两个根元素。这个行为根据根元素的类型的不同而不同。

### 不同类型的元素

当根元素有不同类型的时候，React 将会卸载旧的树，并从头创建新的树。从`<a>`到`<img>`，或者从`<Article>`到`<Comment>`，或者从`<Button>`到`<div>` - 任何一个都将会导致完全重建。

当卸载一棵树的时候，旧的 DOM 节点被销毁。组件实例接收`componentWillUnmount()`。当建立一个新的树的时候，新的 DOM 节点被插入 DOM。组件实例接收`componentWillMount()`，然后是`componentDidMount()`。任何和旧的树关联的状态都会丢失。

根下的任何组件将会被卸载，并且状态被销毁，比如，当 diff：
```jsx harmony
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```
这将会摧毁旧的`Counter`然后重新挂载一个新的。

### 相同类型的 Dom 元素
当比较两个相同类型的 React DOM 元素，React 查找两个的属性，保存相同的底层 DOM 节点，并且只更新改变的属性。比如：

```jsx harmony
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过比较这两个元素，React 知道只修改底层 DOM 节点的`className`。

当更新`style`，React 也知道只更新改变的属性。比如：
```jsx harmony
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

当转化这两个元素，React 知道只修改`color`样式，不是`fontWeight`。

在处理完 DOM 节点之后，React 在子元素上递归。

### 相同类型的组件元素
当一个组件更新的时候，实例依旧相同，因此状态在多次渲染之间维持。React 更新底层组件实例的属性去匹配新的元素，并调用底层实例的`componentWillReceiveProps()`和`componentWillUpdate()`。

下一步，`render()`方法被调用，diff 算法在前一次结果和新结果之间递归调用。

### 在子组件递归

默认，当在一个 DOM 节点的子元素递归的时候，React 同时迭代两个子组件列表，生成一个操作，当他们不同的时候。

比如，当添加一个元素到子元素的最后，两棵树之间的转化工作的也很好：
```jsx harmony
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```
React 将会匹配两个`<li>first</li>`树，噗呸两个`<li>second</li>`树，然后插入`<li>third</li>`树。

如果你天真的实现它，插入一个元素到开头将会有一个很差的性能。比如，转化这两棵树工作的很差：
```jsx harmony
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```
React 将会操作每一个子元素，而不是意识到他可以保持`<li>Duke</li>`和`<li>Villanova</li>`子树完好。这种失效会成为一个问题。

### Key
为了解决这个问题，React 支持一个`key`属性。当子元素有 key，React 使用 key 去匹配源树中的子元素和后续的树。比如，添加一个`key`到我们前面无效的例子，会让树的转化有效：
```jsx harmony
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```
现在 React 知道有`'1024'`key 的是新的，带`'2015'`和`'2016'`key 的元素被移动了。

实际上，寻找一个 key 通常不难。你想要显示的元素可能已经有一个唯一 ID，因此可以来自你的数据：
```jsx harmony
<li key={item.id}>{item.name}</li>
```
如果不是这样，你可以添加一个新的 ID 属性到你的模型或者哈希内容的一部分去生成一个 key。key 只需要在相邻唯一，不是全局唯一。

作为最后手段，你可以传递一个项在数组中的索引作为一个 key。如果项不会再排序，这将工作的很好，但是重排序将会很慢。

重排序也会导致问题，当组件使用索引作为 key。组件实例基于 key 更新和重用。如果 key 是一个索引，移动一个项并改变它。作为结果，类似不受控的输入框的组件状态会以不期望的方式更新。

[这]是一个 CodePen 由于使用索引作为 key 导致的这个问题的例子，[这]()是一个相同例子的更新的版本，显示怎样不使用索引作为 key 将会修复这些重排序，排序和前置问题。

### 取舍
有一点很重要需要记住的是调和算法是一个实现细节。React 可以在每个动作渲染整个 app；最后的结果将会相同。要清楚，渲染器在上下文意味着为所有组件调用`render`，它不意味着 React 将会卸载和重装载他们。它将只会应用根据前一个章节标记出的不同。


我们将会经常提炼启发式，为了让常见使用场景更快。在当前的是想，你可以表述显示，那就是一个子树被从相邻元素移除，但是你不能告诉它被移除到哪里去了。算法将会渲染整个子树。

因为 React 依赖于启发式，如果他们背后的假设没有命中，表现将受影响。

1. 算法将不会尝试去噗呸不同组件类型的子树。如果你发现你的两个组件的更新有非常相近的输出，你可能想要去让他们有相同的类型。实际上，我们没有发现这会导致问题。

2. key 应该是稳定的，可预料的。不稳定的 key（比如通过`Math.random()`产生）将会导致很多组件实例和 DOM 节点被不必要的重新创建，这会导致性能降低并丢失子组件中的状态。



























