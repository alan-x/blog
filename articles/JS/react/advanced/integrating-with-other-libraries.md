[原文地址](https://reactjs.org/docs/integrating-with-other-libraries.html)
### 整合其他库

React 可以用在任何网页应用。它可以嵌入到其他应用，并且，小心一点，其他应用可以嵌入到 React。这个指南检测一些更常见的例子，聚焦于[jQuery]()和[Backbone]()，但是相同的想法可以应用于整合组件和任何已存在的代码。

### 整合 DOM 操作插件

React 意识不到 React 之外的 DOM 的改变，它基于它自己的内部表示决定更新，如果相同的 DOM 节点被其他库操作，React 会被迷惑并且无法恢复。

这不意味着将 React 和其他影响 DOM 的方式结合是不可能或者甚至是困难的，你只需要小心他们做了什么。

避免混淆最简单的方式是防止 React 组件更新。你可以通过渲染 React 没有理由更新的元素，比如一个空的`<div />`。

### 怎样解决这个问题

为了展示这个，来为一个通用 jQuery 插件设计一个包裹器。

我们嫁给你绑定一个[ref]()到根 DOM 元素。在`componentDidMount()`内，我们将得到一个到它的引用，这样我们可以传递它到 jQuery 插件。

为了防止 React 在挂载之后触摸 DOM，我们将在`render()`返回一个空的`<div />`，`<div />`元素没有属性或者子元素，因此 React 没有理由去更新它，让 jQuery 插件自由管理这部分的 DOM。

```jsx harmony
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);·
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

注意我们定义了`componentDidMount`和`componentWillUnmount`[生命周期方法]()。很多 jQuery 插件绑定事件监听器到 DOM，因此在`componentWillUnmount`解绑他们是很重要的。如果插件不提供方法去清理，你将可能需要自己提供，记住去移除插件注册的任何事件监听器，防止内存泄漏。

### 整合 jQuery 选择插件

作为这些概念更具体的例子，编写一个[Chosen]()插件的迷你包裹器，扩展`<select>`输入。

注意：只是因为这可能，不意味着这是 React 应用最好的方式。我们鼓励尽可能去使用 React 组件。React 组件更容易在 React 应用中复用，通常提供对他们行为和外观更多的控制。

首先，看看 Chosen 对 DOM 做了什么。

如果你在`<select>` DOM 节点调用它，它读取原始 DOM 节点的属性，使用内联样式隐藏它，然后在`<select>`之后使用自己的可视化表现添加一个独立的 DOM 节点。然后触发 jQuery 事件去通知我们改变。

假设这是我们为`<Chosen>`包裹器 React 组件抓取的 API：
```jsx harmony
function Example() {
  return (
    <Chosen onChange={value => console.log(value)}>
      <option>vanilla</option>
      <option>chocolate</option>
      <option>strawberry</option>
    </Chosen>
  );
}
```
为了简单起见，我们将作为[非受控组件]()实现。

首先我们创建一个空的组件，它的`render()`方法返回包裹`<select>`的`<div>`：
```jsx harmony
class Chosen extends React.Component {
  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```
注意我们是如何包裹`<select>`到一个额外的`<div>`。这是必须的，因为`<Chosen>`将拼接其他 DOM 元素到我们传递的`<select>`节点后。然而，对 React 而言，`<div>`总是只有单一的一个子节点。这是我们确保 React 更新不会和 Chosen 额外拼接的 DOM 节点冲突的方式。如果你在 React 流之外修改 DOM，你必须确保 React 没有理由去触碰这些 DOM 节点。

下一步，我们将实现生命周期方法。我们需要在`componentDidMount`中使用`<select>`节点的 ref 初始化 Chosen，并在`componentWillUnmount`中卸载它：
```jsx harmony
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();
}

componentWillUnmount() {
  this.$el.chosen('destroy');
}
``` 
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/qmqeQx?editors=0010)

注意，React 没有赋予`this.el`域任何特殊的含义。它可以工作只是因为我们前面在`render()`方法中的`ref`赋值了这个：
```jsx harmony
<select className="Chosen-select" ref={el => this.el = el}>
```
这里已经足够让我们的组件去渲染，但是我们也希望在值改变的时候被通知。为了做到这个，我们将在`<select>`上定于订阅 Chose 管理的 jQuery `change`事件。

我们不直接传递`this.props.onCHange`到  Chosen，因为组件的属性可能随着事件改变，包括事件处理器。相反，我们将声明一个`handleChange()`方法，调用`this.props.onChange`，并订阅它到`change`事件：

```jsx harmony
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();

  this.handleChange = this.handleChange.bind(this);
  this.$el.on('change', this.handleChange);
}

componentWillUnmount() {
  this.$el.off('change', this.handleChange);
  this.$el.chosen('destroy');
}

handleChange(e) {
  this.props.onChange(e.target.value);
}
```

[在 CodePen 中尝试它](https://codepen.io/gaearon/pen/bWgbeE?editors=0010)

最后，还有一件事需要做。在 React，属性可以随着事件改变。比如，`<Chosen>`组件在父组件状态改变的时候得到不同的子元素。这意味着在集成点手动更新 DOM 响应属性更新很重要，因为我们不再让 React 为我们管理 DOM。

Chosen 的文档建议我们使用 jQuery 的`trigger` API 去通知源 DOM 元素的改变。我们将让 React 注意`<select>`内`this.props.children`的更新，但是我们也添加`componentDidUpdate()`生命周期方法通知 Chosen 关于子元素列表的改变：
```jsx harmony
componentDidUpdate(prevProps) {
  if (prevProps.children !== this.props.children) {
    this.$el.trigger("chosen:updated");
  }
}
```
这种方式，Chosen 将会知道去更新它的 DOM 元素，当 React 管理的`<select>`子元素发生改变的时候。

`Chosen`组件完整的实现看起来像这样：
```jsx harmony
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }
  
  componentDidUpdate(prevProps) {
    if (prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }
  
  handleChange(e) {
    this.props.onChange(e.target.value);
  }

  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

[在 CodePen 中尝试](https://codepen.io/gaearon/pen/xdgKOz?editors=0010)

### 整合其他视图库

感谢[ReactDOM.render()](https://reactjs.org/docs/react-dom.html#render)的灵活性，React 可以嵌入到其他应用中。

尽管 React 通常用于加载一个单一的根 React 组件到 DOM，`ReactDOM.render()`可以为 UI 的多个独立部分调用多次，小到按钮，大到一个应用。

实际上，这是 React 在 Facebook 的使用方式。这让我们使用 React 一块一块的编写应用，并将他们绑定到我们已存在的服务端生成的模版和其他客户端代码。

## 使用 React 替换基于字符串的渲染

在旧的网页应用中，一个常见的模式是将描述 DOM 的块作为字符串，并将它像这样插入 DOM：`$el.html(htmlString)`：代码库中的这些点特别时候引入 React。只要重写字符串，基于渲染为一个 React 组件。

因此下面的 jQuery 实现...
```jsx harmony
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```
... 可以重写为使用一个 React 组件：
```jsx harmony
function Button() {
  return <button id="btn">Say Hello</button>;
}

ReactDOM.render(
  <Button />,
  document.getElementById('container'),
  function() {
    $('#btn').click(function() {
      alert('Hello!');
    });
  }
);
```

从这里，你可以开始移动更多逻辑到组件，并且开始接收更多 React 实践。比如，在组件中，最好别依赖 ID，因为相同的组件可以渲染多次。相反，我们将使用[React 事件系统](https://reactjs.org/docs/handling-events.html)并直接注册点击事件在 React `<button>`元素：

```jsx harmony
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/RVKbvW?editors=1010)

你可以拥有你想要数量的独立的组件，并使用`ReactDOM.render()`去渲染他们到不同的 DOM 容器。渐渐的，随着你转化你的应用到 React，你讲可以去绑定他们到大型组件中，并移动`ReactDOM.render()`调用到层级的上面。

### 将 React 嵌入到 Backbone 视图

[Backbone](https://backbonejs.org/)视图通常使用 HTML 字符串，或者字符串产生模板函数，为了给他们的 DOM 元素创建内容。这个流程也可以替换为渲染成一个 React 组件。

下面，我们将创建一个 Backbone 视图，叫做`ParagraphView`。它将会重写 Backbone 的`render()`函数，去渲染一个 React`<Paragraph>`组件到 Backbone（`this.el`）提供的 DOM 元素。这里，我们也使用[ReactDOM.render()](https://reactjs.org/docs/react-dom.html#render)：
```jsx harmony
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extend({
  render() {
    const text = this.model.get('text');
    ReactDOM.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/gWgOYL?editors=0010)

我们也在`remove`方法中调用`ReactDOM.unmountComponentAtNode()`，因此在解绑的时候， React 卸载事件处理器和其他关联组件树的资源，

当一个组件在 React 树中被移除的时候，清理被自动化执行，但是因为我们手动移除了整棵树，我们必须调用这个方法。

### 整合模型层

尽管通常推荐使用类似[React 状态](https://reactjs.org/docs/lifting-state-up.html)，[Flex](https://facebook.github.io/flux/)，或者[Redux](https://redux.js.org/) 之类的单向数据流，React 组件可以使用其他框架和库的模型层。

### 在 React 组件中使用 Backbone 模型

从 React 组件消费[Backbone]()模型和集合最简单的额方式是舰艇大量改变事件并手动强制更新。

负责渲染模型的组件将监听'change'事件，然而负责渲染集合的组件将监听'add'和'remove'事件。两种情况下，调用[this.forceUpdate()]()使用新的数据去渲染组件。

在瞎买呢的例子中，`List`组件渲染一个 Backbone 集合，使用`Item`组件去渲染独立的项。

```jsx harmony
class Item extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.model.on('change', this.handleChange);
  }

  componentWillUnmount() {
    this.props.model.off('change', this.handleChange);
  }

  render() {
    return <li>{this.props.model.get('text')}</li>;
  }
}

class List extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.collection.on('add', 'remove', this.handleChange);
  }

  componentWillUnmount() {
    this.props.collection.off('add', 'remove', this.handleChange);
  }

  render() {
    return (
      <ul>
        {this.props.collection.map(model => (
          <Item key={model.cid} model={model} />
        ))}
      </ul>
    );
  }
}
```

[在 CodePen 尝试](https://codepen.io/gaearon/pen/GmrREm?editors=0010)

### 从 Backbone 模型中解析数据

前面的解决方案需要你的 React 组件意识到 Backbone 的模型和集合。如果你之后计划升级到其他数据管理解决方案，你可能想要去较少 Backbone 的知识，越少越好。

这个问题的一个解决方案是当它改变的时候，抽取模型的属性作为普通数据，并保持这个逻辑到单独的地方。下面是[一个高阶组件]()，抽取所有的 Backbone 模型的属性到状态，传递数据到包裹的组件。

这种方式，只有高阶组件需要知道 Backbone 模型内部，并且应用中大部分组件可以保持对 Backbone 的无感知。

在瞎买呢的例子中，我们将赋值模型的属性去组织初始化状态，我们订阅`change`事件（并在卸载的时候取消订阅），当它发生的时候，我们使用模型当前属性更新状态。最后，我们确保，如果`modal`属性它自己改变，我们不会忘记从旧的模型取消订阅，并订阅到新的一个。

注意这个例子不意味着和 Backbone 一起用的全部，但是它给我们一个关于通常实现这个的方式：
```jsx harmony
function connectToBackboneModel(WrappedComponent) {
  return class BackboneComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = Object.assign({}, props.model.attributes);
      this.handleChange = this.handleChange.bind(this);
    }

    componentDidMount() {
      this.props.model.on('change', this.handleChange);
    }

    componentWillReceiveProps(nextProps) {
      this.setState(Object.assign({}, nextProps.model.attributes));
      if (nextProps.model !== this.props.model) {
        this.props.model.off('change', this.handleChange);
        nextProps.model.on('change', this.handleChange);
      }
    }

    componentWillUnmount() {
      this.props.model.off('change', this.handleChange);
    }

    handleChange(model) {
      this.setState(model.changedAttributes());
    }

    render() {
      const propsExceptModel = Object.assign({}, this.props);
      delete propsExceptModel.model;
      return <WrappedComponent {...propsExceptModel} {...this.state} />;
    }
  }
}
```

为了展示怎么使用它，我们将链接一个`NameInput` React 组件到一个 Backbone 模型，并更新它的`firstName`属性，当输入框改变的时候：
```jsx harmony
function NameInput(props) {
  return (
    <p>
      <input value={props.firstName} onChange={props.handleChange} />
      <br />
      My name is {props.firstName}.
    </p>
  );
}

const BackboneNameInput = connectToBackboneModel(NameInput);

function Example(props) {
  function handleChange(e) {
    props.model.set('firstName', e.target.value);
  }

  return (
    <BackboneNameInput
      model={props.model}
      handleChange={handleChange}
    />

  );
}

const model = new Backbone.Model({ firstName: 'Frodo' });
ReactDOM.render(
  <Example model={model} />,
  document.getElementById('root')
);
```
[在 CodePen 中尝试](https://codepen.io/gaearon/pen/PmWwwa?editors=0010)

这个技术不局限于 Backbone。你可以使用 React 和任何模型库，通过在生命周期订阅到它的改变，并且，可选的，赋值数据到本地的 React 状态。
