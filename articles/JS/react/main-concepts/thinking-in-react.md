[原文地址](https://reactjs.org/docs/thinking-in-react.html)
### Thinking in React

在我们的观点看来，React 是使用 JavaScript 构建大型，快速网页应用的首选方式。它在 Facebook 和 Instagram 的推广效果特别好。

React 很大的一部分是它怎样让你思考你构建他们的方式。在这个文档，我们将带你使用 React 贯穿构建一个可搜索产品数据表格的流程。

### 使用一个 Mock 开始

想象我们已经有一个 JSON API 和一个我们设计者的 mock。mock 看起来像这个：

![](https://reactjs.org/static/thinking-in-react-mock-1071fbcc9eed01fddc115b41e193ec11-4dd91.png)

我们的 JSON API 返回一些数据，就像这样：
```jsx harmony
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```
 
### 步骤1：打破 UI 到一个组件层级

你要做的第一件事是在 mock 中为每个组件（和子组件）周围绘制一个盒子，并给他们名字。如果你和一个设计师工作，他们可能已经这么做了，因此和他们聊聊！他们的 Photoshop 层的名字可能会成为你的 React 组件名！
 
 但是你如何知道什么是组件自己的组成部分？使用相同的技术去决定是否需要创建一个新的函数或者对象。一个技术是[单一职责原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)，那就是，理想情况下一个组件应该只做一件事。如果它组件成长，它应该分解为更小的组件。

因为你通常显示一个 JSON 数据模型给用户，你将发现，如果你的模型构建正确，你的 UI（你的组件结构）将会映射的非常好。因为 UI 和数据模型趋向于相同的信息架构。分离你的 UI 到组件，每一个组件都匹配你的数据模型的一部分。

![](https://reactjs.org/static/thinking-in-react-components-eb8bda25806a89ebdc838813bdfa3601-82965.png)

你在这里可以看到，在我们的应用，我们有五个组件。我们已经斜体了每个组件表示的数据：

1. **FilterableProductTable(orange)**：包含完整的例子

2. **SearchBar (blue)**：接受所有的用户输入

3. **ProductTable (green):**：基于用户输入显示和过滤数据集

4. **ProductCategoryRow (turquoise)**：显示每个类别的标题

5. **ProductRow（red）**：为每一个产品显示一行

如果你正在看`ProductTable`，你会看到表格头（包含"Name"和"Price"标签）不是它自己的组件。这只是偏好，还有其他实现方式。对于这个例子，我们将它当作`ProductTable`的一部分，因为它是渲染数据集的一部分，是`ProductTable`的责任。然而，如果这个头部变得复杂（比如，如果我们添加排序功能），让他成为`ProductTableHeader`组件是有意义的。

现在我们在我们的 mock 中标识除了组件，现在来将他们安排到层级。出现在其他组件的组件应该作为子组件出现：

- `FilterableProductTable`
    - `SearchBar`
    - `ProductTable`
        - `ProductCategoryRow`
        - `ProductRow`

### 步骤2：使用 React 构建一个静态版本

[CodePen](https://codepen.io/lacker/embed/BwWzwm?height=600&theme-id=0&slug-hash=BwWzwm&default-tab=js&user=lacker&embed-version=2&name=cp_embed_1)

现在已经有你的组件层级了，是时候去实现你的应用了。最简单的方式是使用你的数据模型，渲染 UI 但是无法交互。最好区分这些流程，因为创建一个静态版本不需要思考，只需要大量的输入，并且添加交互需要大量的思考，和少量的输入。我们来看看为啥。

为了构建一个渲染你的数据模型的静态版本的应用，你将构建组件，重用其他组件并使用属性传输数据。如果你对状态的概念很熟悉，**不要使用状态**去构建静态版本。状态是为交互保留的，也就是说，数据随着事件改变。因为这是一个静态版本的应用，不需要它。

你可以自顶向下或者自底向上。也就是说，你可以从层级的顶部构建组件（比如，从`FilterableProductTable`开始）或者更低（`ProductRow`）。在更简单的额例子中，自顶向下通常更简单，在大型项目，自底向上并编写测试更简单。

在这个步骤的底部，你将有一个可重用的组件库，渲染你的数据模型，组件将只会有`render()`方法，因为这是一个静态版本。层级顶部的组件（`FIlterableProductTable`）将会接受你的数据模型为一个属性。如果你创建一个修改到你的底层数据模型并调用`ReactDom.render()`，UI 将会被更新。 你可以看到你的 UI 是如何更新的，和哪里发生了更新。React 是**单向数据流**（也叫做单向绑定），让所有东西模块化和更快。

参考[React 文档](https://reactjs.org/docs/)，如果你执行这个步骤的时候需要帮助。

### 一个短暂的插曲：属性 vs 状态

在 React 中，有两种类型的"模型"数据，在 React 中：属性和状态。理解两者的区别很重要；浏览[官方 React 文档](https://reactjs.org/docs/state-and-lifecycle.html)，如果你不确认两者的不同。也看看[FAQ：状态和属性之间有什么不同？](https://reactjs.org/docs/faq-state.html#what-is-the-difference-between-state-and-props)

### 步骤3：标识最小（但完整）的 UI 状态表示

为了让你的 UI 可以交互，你需要能够触发你底层数据模型的改变。React 使用 state 达到这个。

为了正确构建你的应用，你首先需要去思考你应用需要的可变状态的最小集合。这里的关键词是：[DRY: Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)。找出你的应用需要的绝对最小的状态表示，并计算任何你按需需要的。比如，如果你在构建一个 TODO 列表，保存一个数组的 TODO 项；不要保存一个分离的状态用于统计。相反，当你要去渲染 TODO 的数量，使用 TODO 项数组的长度。

思考我们示例应用中所有的数据。我们有：

- 产品的原始列表

- 用户输入的搜索文字

- checkbox 的值

- 产品的过滤列表

让我们过一遍并指出哪一个是状态。对每一个数据问三个问题：

1.  通过属性从父组件传递过来？如果是，他可能不是属性。

2. 它需要一直保持不变吗？如果是，他可能部署状态。

3. 你可以基于组件的其他状态或者属性计算出它吗？如果可以，那不是状态。

产品的原始列表通过属性传递进来，因此不是状态；搜索文本和 chekbox 看起来是状态，因为他们随着时间改变并且无法从其他计算出来。最终，过滤的产品列表不是状态，因为它可以通过结合原始产品列表和搜索文本和 checkbox 的值计算出来。

所以最终，我们的状态是：

- 用户输入的搜索文本

- checkbox 的值

### 步骤4：标识你的状态应该生存在哪里

[codePen](https://codepen.io/gaearon/pen/qPrNQZ)

好了，我们已经标识出应用的最小集合的状态。下一步，我们需要去标识哪个组件操作，或者拥有这个状态。

记住：React 是组件层级向下的单向数据流。哪一个组件应该拥有什么状态可能不是很清楚。**这可能是对新人理解最具有挑战性，** 跟随下面记不指出：

对于你的应用里面的每一块状态：

- 标识出基于这个状态渲染的每个组件

- 找到一个公共组件（在层级中所有需要这个状态的组件之前单一的组件）

- 公共拥有者或者层级中更高的组件应该拥有这个状态

- 如果你无法找到一个组件，它拥有这个状态，创建一个新的组件单独持有这个状态，并添加到公共拥有者组件的前面的层级中。

在我们的应用中贯穿这个策略：

- `ProductTable`需要过滤产品列表，基于状态，`SearchBar`需要去显示搜索文字和选中的值

- 公共所有者组件是`FilterProductTable`

- 通常来说，让过滤文字和选中的值活在`FilterableProductTable`是有意义的。

帅，我们已经决定我们的状态活在`FilterableProductTable`。首先，添加一个属性示例`this.state = {filterText: '', inStockOnly: false}`到`FilterableProductTable`的`constructor`去反映你的应用的初始化状态。然后，传递`filterText`和`inStockOnly`到`ProductTable`和`SearchBar`作为属性。最后，使用这些属性去过滤`ProductTable`中的行，并设置表单域的值到`SearchBar`。

你可以开始看看你的应用的行为：设置`filterText`到`"ball"`并刷新你的应用。你将会看到数据表被正确更新。

### 步骤5：添加数据流

![CodePen](https://codepen.io/gaearon/pen/LzWZvb)

到目前为止，我们已经创建了一个应用，正确渲染作为函数的属性和状态，向下传递。现在，是时候使用其他方式支持数据流：层级深处的表单组件需要在`FilterableProductTable`中更新状态。

React 让数据流更明确的帮助你去理解你的程序是如何工作的，但是它需要比传统双向数据绑定更多的编码。

如果你尝试在当前版本的例子输入或者选择盒子，你将看到 React 忽略你的输入。这是预期内的，因为我们已经设置了`input`的`value`属性总是和`FilterableProductTable`传入的`state`相同。

现在来思考以下我们希望发生什么。我们想要去确保无论很湿，当用户改变表单，我们更新状态去响应用户输入。因为组件应该值更新他们自己的状态。我们可以使用输入框的`onChange`事件去通知他。`FilterableProductTable`传递的回调将会调用`setState()`，并且应用将会被更新。

### 这就是所有

希望，这给你一个关于怎样使用 React 构建组件和应用的想法。尽管它需要多一点输入，记住代码读的比写的多，读这种模块化的代码比较简单，在你开始构建大组件库的时候，你将感谢这个显式和模块化，当重用代码的时候，你的代码行数将开始缩减。


















