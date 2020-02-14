[实现地址](https://reactjs.org/docs/implementation-notes.html)
### 实现笔记

这是章节是一系列[stack reconciler](https://reactjs.org/docs/codebase-overview.html#stack-reconciler)的实现笔记。

这很有技术性，并且假设对 React 公共 API 有很深的理解，比如如何区分 core，renderer，和 reconciler。如果你对 React 代码库非常熟悉，首先阅读[代码库概述](https://reactjs.org/docs/codebase-overview.html)。

也假设对[React 组件，实例，元素的不同](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)的理解。

stack reconciler 在 React 15 和更早之前使用。它位于[src/renderers/shared/stack/reconciler](https://github.com/facebook/react/tree/15-stable/src/renderers/shared/stack/reconciler)。

### 视频：从头构建 React

[Paul O’Shannessy](https://twitter.com/zpao)有一个关于[从头构建 React]()的谈话，很大程度上启发了这个文档。

这个文档和它的糖化简化了真实代码库，因此对这两个属性的化，你可以更好的理解。

### 概述

reconciler 本身没有公共 API。像 React DOM 和 React Native 之类的[Renderer]()使用它去有效更新用户接口，基于用户编写的 React 组件。

### 递归过程挂载

考虑一下你第一次挂载一个组件：
```jsx harmony
ReactDOM.render(<App />, rootEl);
```

React DOM 将传递`<App />`到 reconciler。记住`<App />`是一个 React 元素，也就是，描述渲染什么。你可以认为它是一个普通对象：
```jsx harmony
console.log(<App />);
// { type: App, props: {} }
```
reconciler 将会检查`App`是一个类还是一个函数。

如果`App`是一个函数，reconciler 将调用`App(props)`去获取渲染的元素。

如果`App`是一个类，reconciler 将使用`new App(props)`实例化一个`App`，调用`componentWillMount()`生命周期方法，然后调用`render()`方法去获取渲染的元素。

无论怎样，reconciler 将了解`App`渲染的元素。

这个过程是递归的。`App`可能渲染一个`<Greeting />`，`Greeting`可能渲染一个`<Button />`，等等。reconciler 将"向下"递归穿过用户定义的组件，了解每一个组件渲染的。

你可以想象这个过程如下伪代码：
```jsx harmony
function isClass(type) {
  // React.Component subclasses have this flag
  return (
    Boolean(type.prototype) &&
    Boolean(type.prototype.isReactComponent)
  );
}

// This function takes a React element (e.g. <App />)
// and returns a DOM or Native node representing the mounted tree.
function mount(element) {
  var type = element.type;
  var props = element.props;

  // We will determine the rendered element
  // by either running the type as function
  // or creating an instance and calling render().
  var renderedElement;
  if (isClass(type)) {
    // Component class
    var publicInstance = new type(props);
    // Set the props
    publicInstance.props = props;
    // Call the lifecycle if necessary
    if (publicInstance.componentWillMount) {
      publicInstance.componentWillMount();
    }
    // Get the rendered element by calling render()
    renderedElement = publicInstance.render();
  } else {
    // Component function
    renderedElement = type(props);
  }

  // This process is recursive because a component may
  // return an element with a type of another component.
  return mount(renderedElement);

  // Note: this implementation is incomplete and recurses infinitely!
  // It only handles elements like <App /> or <Button />.
  // It doesn't handle elements like <div /> or <p /> yet.
}

var rootEl = document.getElementById('root');
var node = mount(<App />);
rootEl.appendChild(node);
```

注意：这真的是一个伪代码。它和真正的实现不类似。他也会导致一个栈溢出，因为我们没有讨论何时停止递归。

来回顾一下前面例子的关键点：

- React 元素是普通对象，表示组件类型（比如，`App`）和属性。

- 用户定义的组件（比如，`App`）可以是类或者函数，但是他们全部"渲染成"元素

- "挂载"是递归的过程，为给定顶级 React 元素创建一个 DOM 或者 Native 树（比如，`<App />`）。

### 挂载宿主元素

如果我们最后没有渲染一些东西到屏幕上，这个过程是无用的。

除了用户定义（"组合"）组件，React 元素可能表示平台-指定（"宿主"）组件。比如，`Button`可能在它的 render 方法返回`<div />`。

如果元素的`type`是一个字符串，我们就是在处理宿主元素：
```jsx harmony
console.log(<div />);
// { type: 'div', props: {} }
```

没有用户定义的代码和宿主元素关联。

当 reconciler 遭遇一个宿主元素，它让 renderer 注意挂载它。比如，React DOM 将会创建 DOM 节点。

如果宿主元素有子元素，reconciler 使用和前面相同的算法递归挂载他们。子元素是宿主（比如`<div><hr /></div>`)，合成（比如`<div><Button /></div>`），还是两者都没有啥关系。

子元素产生的 DOM 节点将拼接到父元素的 DOM 节点，并递归，完整的 DOM 结构将会被组装。

注意：reconciler 本身和 DOM 不关联。确切的挂载结果（有时候在源代码中叫做"挂载镜像"）取决于 renderer，可以是一个 DOM 节点（React DOM），一个字符串（React DOM Server），或者表述原生视图的数字（React Native）。

如果我们扩展代码去处理宿主元素，它看起来会是这样：
```jsx harmony
function isClass(type) {
  // React.Component subclasses have this flag
  return (
    Boolean(type.prototype) &&
    Boolean(type.prototype.isReactComponent)
  );
}

// This function only handles elements with a composite type.
// For example, it handles <App /> and <Button />, but not a <div />.
function mountComposite(element) {
  var type = element.type;
  var props = element.props;

  var renderedElement;
  if (isClass(type)) {
    // Component class
    var publicInstance = new type(props);
    // Set the props
    publicInstance.props = props;
    // Call the lifecycle if necessary
    if (publicInstance.componentWillMount) {
      publicInstance.componentWillMount();
    }
    renderedElement = publicInstance.render();
  } else if (typeof type === 'function') {
    // Component function
    renderedElement = type(props);
  }

  // This is recursive but we'll eventually reach the bottom of recursion when
  // the element is host (e.g. <div />) rather than composite (e.g. <App />):
  return mount(renderedElement);
}

// This function only handles elements with a host type.
// For example, it handles <div /> and <p /> but not an <App />.
function mountHost(element) {
  var type = element.type;
  var props = element.props;
  var children = props.children || [];
  if (!Array.isArray(children)) {
    children = [children];
  }
  children = children.filter(Boolean);

  // This block of code shouldn't be in the reconciler.
  // Different renderers might initialize nodes differently.
  // For example, React Native would create iOS or Android views.
  var node = document.createElement(type);
  Object.keys(props).forEach(propName => {
    if (propName !== 'children') {
      node.setAttribute(propName, props[propName]);
    }
  });

  // Mount the children
  children.forEach(childElement => {
    // Children may be host (e.g. <div />) or composite (e.g. <Button />).
    // We will also mount them recursively:
    var childNode = mount(childElement);

    // This line of code is also renderer-specific.
    // It would be different depending on the renderer:
    node.appendChild(childNode);
  });

  // Return the DOM node as mount result.
  // This is where the recursion ends.
  return node;
}

function mount(element) {
  var type = element.type;
  if (typeof type === 'function') {
    // User-defined components
    return mountComposite(element);
  } else if (typeof type === 'string') {
    // Platform-specific components
    return mountHost(element);
  }
}

var rootEl = document.getElementById('root');
var node = mount(<App />);
rootEl.appendChild(node);
```

这可以工作，但是和 reconciler 实际实现还是有差别的。缺少的主要成分是对更新的支持。

### 引入内部实例

React 的核心特性是你可以重新渲染任何东西，并且不会重新创建 DOM 或者重置状态：
```jsx harmony
ReactDOM.render(<App />, rootEl);
// Should reuse the existing DOM:
ReactDOM.render(<App />, rootEl);
```

然而，我们前面的实现只知道怎样挂载初始化树。它不执行更新，因为它不存储任何必须的信息，比如，所有的`publicInstance`，或者哪一个 DOM `node`表示哪一个组件。

stack reconciler 代码库解决了这个问题，通过让`mount()`函数成为类中的一个方法。这是这个方式的缺点，我们正在以相反的方式[重写 reconciler](https://reactjs.org/docs/codebase-overview.html#fiber-reconciler)。然而，这就是它现在的工作方式。

与其区分`mountHost`和`mountComposite`函数，我们将创建两个类：`DOMComponent`和`CompositeComponent`。

两个类都有构造器，接受`element`，同时在`mount()`方法中返回挂载的节点。我们将使用一个工厂替换顶级`mount()`函数，初始化正确的类：
```jsx harmony
function instantiateComponent(element) {
  var type = element.type;
  if (typeof type === 'function') {
    // User-defined components
    return new CompositeComponent(element);
  } else if (typeof type === 'string') {
    // Platform-specific components
    return new DOMComponent(element);
  }  
}
```

首先，假设这么实现`CompositeComponent`：
```jsx harmony
class CompositeComponent {
  constructor(element) {
    this.currentElement = element;
    this.renderedComponent = null;
    this.publicInstance = null;
  }

  getPublicInstance() {
    // For composite components, expose the class instance.
    return this.publicInstance;
  }

  mount() {
    var element = this.currentElement;
    var type = element.type;
    var props = element.props;

    var publicInstance;
    var renderedElement;
    if (isClass(type)) {
      // Component class
      publicInstance = new type(props);
      // Set the props
      publicInstance.props = props;
      // Call the lifecycle if necessary
      if (publicInstance.componentWillMount) {
        publicInstance.componentWillMount();
      }
      renderedElement = publicInstance.render();
    } else if (typeof type === 'function') {
      // Component function
      publicInstance = null;
      renderedElement = type(props);
    }

    // Save the public instance
    this.publicInstance = publicInstance;

    // Instantiate the child internal instance according to the element.
    // It would be a DOMComponent for <div /> or <p />,
    // and a CompositeComponent for <App /> or <Button />:
    var renderedComponent = instantiateComponent(renderedElement);
    this.renderedComponent = renderedComponent;

    // Mount the rendered output
    return renderedComponent.mount();
  }
}
```
这和前面我们实现的`mountComposite()`没有太大不同，但是现在我们可以保存一些信息，比如`this.currentElement`，`this.renderedComponent`，和`this.publicInstance`，用于更新使用。

注意`CompositeComponent`的实例和用户支持的`element.type`实例不是同一个东西。`CompositeComponent`是我们 reconciler 的实现细节，不会暴露给用户。用户定义的类是我们从`element.type`中读取到的，并且`CompositeComponent`创建一个它的实例。

为了避免迷惑，我们将调用`CompositeComponent`和`DOMComponent`的实例的"内部实例"。他们存在，因此我们可以关联一些长时间存在的数据。只有 renderer 和 reconciler 意识到他们的存在。

相反，我们调用用户定义的类的实例为"公共实例"。公共实例是你在`render()`和你的自定义组件的其他方法中看到的`this`。

`mountHost()`函数重构为`DOMComponent`中的`mount()`方法，看起来也是类似的：
```jsx harmony
class DOMComponent {
  constructor(element) {
    this.currentElement = element;
    this.renderedChildren = [];
    this.node = null;
  }

  getPublicInstance() {
    // For DOM components, only expose the DOM node.
    return this.node;
  }

  mount() {
    var element = this.currentElement;
    var type = element.type;
    var props = element.props;
    var children = props.children || [];
    if (!Array.isArray(children)) {
      children = [children];
    }

    // Create and save the node
    var node = document.createElement(type);
    this.node = node;

    // Set the attributes
    Object.keys(props).forEach(propName => {
      if (propName !== 'children') {
        node.setAttribute(propName, props[propName]);
      }
    });

    // Create and save the contained children.
    // Each of them can be a DOMComponent or a CompositeComponent,
    // depending on whether the element type is a string or a function.
    var renderedChildren = children.map(instantiateComponent);
    this.renderedChildren = renderedChildren;

    // Collect DOM nodes they return on mount
    var childNodes = renderedChildren.map(child => child.mount());
    childNodes.forEach(childNode => node.appendChild(childNode));

    // Return the DOM node as mount result
    return node;
  }
}
```

`mountHost()`重构之后主要的不同是我们现在保存`this.node`和`this.renderedChildren`和内部 DOM 组件实例的关联。我们将在未来应用不破坏的更新的时候使用他们。

最后，每一个内部实例，合成或者宿主，现在指向它的子节点的内部实例。为了帮助可视化这个，如果一个函数`<App>`组件渲染一个`<Button>`类组件，并且`Button`类渲染一个`<div>`，内部实例树看起来可能想这样：
```jsx harmony
[object CompositeComponent] {
  currentElement: <App />,
  publicInstance: null,
  renderedComponent: [object CompositeComponent] {
    currentElement: <Button />,
    publicInstance: [object Button],
    renderedComponent: [object DOMComponent] {
      currentElement: <div />,
      node: [object HTMLDivElement],
      renderedChildren: []
    }
  }
}
```

在 DOM 中，你可能只看到`<div>`。然而，内部实例树包含合成和数组内部实例。

合成内部实例需要存储：

- 当前元素

- 如果元素类型是一个类，需要存储公共实例

- 单独的渲染的内部实例。他可以是一个`DOMComponent`或者一个`CompositeComponent`。

宿主内部实例需要存储：

- 当前元素

- DOM 节点

- 所有子内部实例。他们的每一个可以是一个`DOMComponent`或者一个`CompositeComponent`。

如果你注意去想象在一个更复杂的应用中，内部实例树是如何构造的，[React DevTools](https://github.com/facebook/react-devtools)可以给你一个近似的，它使用灰色高亮宿主实例，使用紫色高亮合成实例。

![](https://reactjs.org/static/implementation-notes-tree-d96fec10d250eace9756f09543bf5d58-a6f54.png)

为了完成这个重构，我们将引入一个函数，挂载一个完整的树到容器节点，就像`ReactDOM.render()`。它返回一个公共实例，就像`ReactDOM.render()`。

```jsx harmony
function mountTree(element, containerNode) {
  // Create the top-level internal instance
  var rootComponent = instantiateComponent(element);

  // Mount the top-level component into the container
  var node = rootComponent.mount();
  containerNode.appendChild(node);

  // Return the public instance it provides
  var publicInstance = rootComponent.getPublicInstance();
  return publicInstance;
}

var rootEl = document.getElementById('root');
mountTree(<App />, rootEl);
```

### 卸载

现在我们有内部实例，持有他们的子孙和 DOM 节点，我们可以实现卸载。对于一个合成组件，卸载调用一个生命周期方法并递归。

```jsx harmony
class CompositeComponent {

  // ...

  unmount() {
    // Call the lifecycle method if necessary
    var publicInstance = this.publicInstance;
    if (publicInstance) {
      if (publicInstance.componentWillUnmount) {
        publicInstance.componentWillUnmount();
      }
    }

    // Unmount the single rendered component
    var renderedComponent = this.renderedComponent;
    renderedComponent.unmount();
  }
}
```
对于`DOMComponent`，卸载告诉每一个子元素去卸载。

```jsx harmony
class DOMComponent {

  // ...

  unmount() {
    // Unmount all the children
    var renderedChildren = this.renderedChildren;
    renderedChildren.forEach(child => child.unmount());
  }
}
```
在实践中，卸载一个 DOM 组件也移除事件监听器和清理一些缓存，但是我们将跳过这些详情。

我们可以添加一个新的顶级函数，叫做`unmountTree(containerNode)`，和`ReactDOM.unmountComponentAtNode()`：
```jsx harmony
function unmountTree(containerNode) {
  // Read the internal instance from a DOM node:
  // (This doesn't work yet, we will need to change mountTree() to store it.)
  var node = containerNode.firstChild;
  var rootComponent = node._internalInstance;

  // Unmount the tree and clear the container
  rootComponent.unmount();
  containerNode.innerHTML = '';
}
```
为了让这个可以工作，我们需要从 DOM 节点去读取一个内部根实例。我们将修改`mountTree()`去添加一个`_internalInstance`属性到根 DOM 节点。我们阿井告诉`mountTree()`去解构任何存在的树，因此它可以嗲用多次：
```jsx harmony
function mountTree(element, containerNode) {
  // Destroy any existing tree
  if (containerNode.firstChild) {
    unmountTree(containerNode);
  }

  // Create the top-level internal instance
  var rootComponent = instantiateComponent(element);

  // Mount the top-level component into the container
  var node = rootComponent.mount();
  containerNode.appendChild(node);

  // Save a reference to the internal instance
  node._internalInstance = rootComponent;

  // Return the public instance it provides
  var publicInstance = rootComponent.getPublicInstance();
  return publicInstance;
}
```
现在，重复执行`unmountTree()`，或者运行`mountTree()`，移除旧的树并在组件上运行`componentWillUnmount()`生命周期。

### 更新

在前面的章节，我们实现了卸载。然而，如果每一个属性改变都卸载或者挂载整个树，那么 React 就不是很有用。reconciler 的目标是是重用存在的可能保存 DOM 和状态的实例：
```jsx harmony
var rootEl = document.getElementById('root');

mountTree(<App />, rootEl);
// Should reuse the existing DOM:
mountTree(<App />, rootEl);
```
我们将使用一个或者多个方法扩展我们的内部实例的合约。除了`mount()`和`unmount()`，`DOMComponent`和`CompositeComponent`将实现一个新的方法，讲座`receive(nextElement)`：
```jsx harmony
class CompositeComponent {
  // ...

  receive(nextElement) {
    // ...
  }
}

class DOMComponent {
  // ...

  receive(nextElement) {
    // ...
  }
}
```

它的工作是做一些必要的工作让组件（和它的所有子元素）更新到`nextElement`提供的描述。

这个部分通常描述为"虚拟 DOM 对比"，尽管真正发生的是递归遍历树，让每一个内部实例接受一个更新。

### 更新组合组件

当一个合成组件接受一个新的元素，我们执行`componentWillUpdate()`生命周期方法。

然后我们使用新的属性重新渲染组件，然后获取下一个渲染的元素。

```jsx harmony
class CompositeComponent {

  // ...

  receive(nextElement) {
    var prevProps = this.currentElement.props;
    var publicInstance = this.publicInstance;
    var prevRenderedComponent = this.renderedComponent;
    var prevRenderedElement = prevRenderedComponent.currentElement;

    // Update *own* element
    this.currentElement = nextElement;
    var type = nextElement.type;
    var nextProps = nextElement.props;

    // Figure out what the next render() output is
    var nextRenderedElement;
    if (isClass(type)) {
      // Component class
      // Call the lifecycle if necessary
      if (publicInstance.componentWillUpdate) {
        publicInstance.componentWillUpdate(nextProps);
      }
      // Update the props
      publicInstance.props = nextProps;
      // Re-render
      nextRenderedElement = publicInstance.render();
    } else if (typeof type === 'function') {
      // Component function
      nextRenderedElement = type(nextProps);
    }

    // ...
```

下一步，我们可以查看渲染的元素的`type`。如果`type`从上一次渲染没有更新，后面的组件可以就地更新。

比如，如果它第一次返回的`<Button color="red" />`，第二次返回`<Button color="blue" />`，我们可以告诉对应的内部实例去`receive()`下一个元素：
```jsx harmony
    // ...

    // If the rendered element type has not changed,
    // reuse the existing component instance and exit.
    if (prevRenderedElement.type === nextRenderedElement.type) {
      prevRenderedComponent.receive(nextRenderedElement);
      return;
    }

    // ...
```

然而，如果下一个渲染的元素有一个和前一个渲染的元素不同的`type`，我们不能更新内部实例。一个`<button>`不能"变成"一个`<input>`。

相反，我们卸载已经存在的内部实例，并挂载新的一个对应的渲染的元素类型。比如，这是当一个前一个渲染为`<button />`的元素渲染为一个`<input />`所发生的：

```jsx harmony
    // ...

    // If we reached this point, we need to unmount the previously
    // mounted component, mount the new one, and swap their nodes.

    // Find the old node because it will need to be replaced
    var prevNode = prevRenderedComponent.getHostNode();

    // Unmount the old child and mount a new child
    prevRenderedComponent.unmount();
    var nextRenderedComponent = instantiateComponent(nextRenderedElement);
    var nextNode = nextRenderedComponent.mount();

    // Replace the reference to the child
    this.renderedComponent = nextRenderedComponent;

    // Replace the old node with the new one
    // Note: this is renderer-specific code and
    // ideally should live outside of CompositeComponent:
    prevNode.parentNode.replaceChild(nextNode, prevNode);
  }
}
```

总结一下，当一个合成组件接受一个新的元素，他可能委派这个更新到它渲染的内部实例，或者卸载它并原地挂载一个新的。

还有一个条件存在让一个组件重新挂载，而不是接受一个新的元素，那就是当元素的`key`变化的时候。我们在这个文档讨论`key`的处理，因为它让原本很复杂的指南更复杂。

注意我们需要添加一个`getHostNode()`方法到内部实例合约，这样才可能在更新的死后定位平台指定节点并替换它。它的实现对于两个类痘痕直接：

```jsx harmony
class CompositeComponent {
  // ...

  getHostNode() {
    // Ask the rendered component to provide it.
    // This will recursively drill down any composites.
    return this.renderedComponent.getHostNode();
  }
}

class DOMComponent {
  // ...

  getHostNode() {
    return this.node;
  }  
}

```

### 更新宿主元素

宿主元素实现，比如`DOMComponent`，更新不同。当他们接受一个元素，他们需要去更新底层平台指定的视图。比如 React DOM，这意味着更新 DOM 属性：
```jsx harmony
class DOMComponent {
  // ...

  receive(nextElement) {
    var node = this.node;
    var prevElement = this.currentElement;
    var prevProps = prevElement.props;
    var nextProps = nextElement.props;    
    this.currentElement = nextElement;

    // Remove old attributes.
    Object.keys(prevProps).forEach(propName => {
      if (propName !== 'children' && !nextProps.hasOwnProperty(propName)) {
        node.removeAttribute(propName);
      }
    });
    // Set next attributes.
    Object.keys(nextProps).forEach(propName => {
      if (propName !== 'children') {
        node.setAttribute(propName, nextProps[propName]);
      }
    });

    // ...
```

然后，宿主组件需要更新他们的子元素。不像合成组件，他们可能包含多余一个子元素。

在这个简单的例子，我们使用一个数组的内部实例并迭代它。更新或者替换内部实例取决于接收到的`type`是否匹配前一个`type`。真实的 reconcoler 也接受元素的`key`，跟踪移动，为了防止插入和删除，但是我们缺省这部分的逻辑。

我们收集子元素上的 DOM 操作到一个列表，这样我们可以批量执行他们：
```jsx harmony
    // ...

    // These are arrays of React elements:
    var prevChildren = prevProps.children || [];
    if (!Array.isArray(prevChildren)) {
      prevChildren = [prevChildren];
    }
    var nextChildren = nextProps.children || [];
    if (!Array.isArray(nextChildren)) {
      nextChildren = [nextChildren];
    }
    // These are arrays of internal instances:
    var prevRenderedChildren = this.renderedChildren;
    var nextRenderedChildren = [];

    // As we iterate over children, we will add operations to the array.
    var operationQueue = [];

    // Note: the section below is extremely simplified!
    // It doesn't handle reorders, children with holes, or keys.
    // It only exists to illustrate the overall flow, not the specifics.

    for (var i = 0; i < nextChildren.length; i++) {
      // Try to get an existing internal instance for this child
      var prevChild = prevRenderedChildren[i];

      // If there is no internal instance under this index,
      // a child has been appended to the end. Create a new
      // internal instance, mount it, and use its node.
      if (!prevChild) {
        var nextChild = instantiateComponent(nextChildren[i]);
        var node = nextChild.mount();

        // Record that we need to append a node
        operationQueue.push({type: 'ADD', node});
        nextRenderedChildren.push(nextChild);
        continue;
      }

      // We can only update the instance if its element's type matches.
      // For example, <Button size="small" /> can be updated to
      // <Button size="large" /> but not to an <App />.
      var canUpdate = prevChildren[i].type === nextChildren[i].type;

      // If we can't update an existing instance, we have to unmount it
      // and mount a new one instead of it.
      if (!canUpdate) {
        var prevNode = prevChild.getHostNode();
        prevChild.unmount();

        var nextChild = instantiateComponent(nextChildren[i]);
        var nextNode = nextChild.mount();

        // Record that we need to swap the nodes
        operationQueue.push({type: 'REPLACE', prevNode, nextNode});
        nextRenderedChildren.push(nextChild);
        continue;
      }

      // If we can update an existing internal instance,
      // just let it receive the next element and handle its own update.
      prevChild.receive(nextChildren[i]);
      nextRenderedChildren.push(prevChild);
    }

    // Finally, unmount any children that don't exist:
    for (var j = nextChildren.length; j < prevChildren.length; j++) {
      var prevChild = prevRenderedChildren[j];
      var node = prevChild.getHostNode();
      prevChild.unmount();

      // Record that we need to remove the node
      operationQueue.push({type: 'REMOVE', node});
    }

    // Point the list of rendered children to the updated version.
    this.renderedChildren = nextRenderedChildren;

    // ...
```

最后一步，我们执行 DOM 操作。再一次，真实的 reconciler 代码更加复杂，因为它也处理移动。

```jsx harmony
    // ...

    // Process the operation queue.
    while (operationQueue.length > 0) {
      var operation = operationQueue.shift();
      switch (operation.type) {
      case 'ADD':
        this.node.appendChild(operation.node);
        break;
      case 'REPLACE':
        this.node.replaceChild(operation.nextNode, operation.prevNode);
        break;
      case 'REMOVE':
        this.node.removeChild(operation.node);
        break;
      }
    }
  }
}
```

这就是更新宿主组件的过程。

### 顶级更新

现在`CompositeComponent`和`DOMComponent`实现了`receive(nextElement)`方法，当元素的`type`和最后一次相同的时候，我们可以改变顶级的`mountTree()`函数去使用它。
```jsx harmony
function mountTree(element, containerNode) {
  // Check for an existing tree
  if (containerNode.firstChild) {
    var prevNode = containerNode.firstChild;
    var prevRootComponent = prevNode._internalInstance;
    var prevElement = prevRootComponent.currentElement;

    // If we can, reuse the existing root component
    if (prevElement.type === element.type) {
      prevRootComponent.receive(element);
      return;
    }

    // Otherwise, unmount the existing tree
    unmountTree(containerNode);
  }

  // ...

}
```
现在，调用`mountTree()`两次使用相同的类型不会破坏：
```jsx harmony
var rootEl = document.getElementById('root');

mountTree(<App />, rootEl);
// Reuses the existing DOM:
mountTree(<App />, rootEl);
```

这就是 React 内部的基本工作远离。

### 我们遗漏了什么

这个文档是和真实代码库的简单比较。还有一些重要的方面我们没有提及：

- 组件可以渲染`null`，reconciler 可以处理数组和渲染的输出的"空槽"。

- reconciler 也从元素读取`key`，并且使用它去建立内部实例到元素的对应。真实的 React 中大量的复杂性于此相关。

- 除了合成和宿主内部实例类，还有一些类似"text"和"empty"组件。他们表示文本节点和"空槽"，你会从渲染`null`中得到。

- renderer 使用[注入](https://reactjs.org/docs/codebase-overview.html#dynamic-injection)传递宿主内部类到 reconciler。比如，React DOM 告诉 reconciler 去使用`ReactDOMComponent`作为宿主内部实例实现。

- 更新子元素列表的逻辑被抽取到一个混合，叫做`ReactMultiChild`， 被 React DOM 和 React Native 宿主内部实例类实现使用。

- 在合成组件中，reconciler 也实现对`setState()`的支持。事件处理器内的多个更新在一个更新中批量更新。

- 生命周期方法在 DOM 准备好之后调用，比如`componentDidMount()`和`componentDidUpdate()`，集成到"调用队列"，在单个批处理内执行。

- React 将当前更新的相关信息放置到内部对象，叫做"事务"。事务对跟踪挂起的生命周期，当前的 DOM 嵌套的警告，和任何其他"全局"的指定更新很有用。事务也保证 React 在更新后"清理"。比如，React DOM 提供的事务类存在任何更新之后储输入选择。

### 进入代码

- [ReactMount](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/dom/client/ReactMount.js)是这个指南中类似`mountTree()`和`unmountTree()`的代码。它关注挂载和卸载顶级组件。[ReactNativeMount](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/native/ReactNativeMount.js)是它的 React Native 模拟。
- [ReactDOMComponent](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/dom/shared/ReactDOMComponent.js)和这个教程的`DOMComponent`相同。它为 React DOM renderer 实现宿主组件类。[ReactNativeBaseComponent](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/native/ReactNativeBaseComponent.js)是它的 React Native 模拟。
- [ReactCompositeComponent](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/shared/stack/reconciler/ReactCompositeComponent.js)和这个教程的`CompositeComponent`相同。它处理用户定义组件的调用和维护他们的装套。
- [instantiateReactComponent](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/shared/stack/reconciler/instantiateReactComponent.js)包含选择正确的内部实例类去构造一个元素的开关。它和这个教程中`instantiateComponent()`相等。
- [ReactReconciler](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/shared/stack/reconciler/instantiateReactComponent.js)是`mountComponent()`，`receiveComponent()`，和`unmountComponent()`方法的包装。它在内部实例调用底层实现，但是也包含所有内部实例实现相同的代码。
- [ReactChildReconciler](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/shared/stack/reconciler/ReactChildReconciler.js)基于元素的`kye`实现挂载，更新，和卸载子孙的逻辑。
- [ReactMultiChild](https://github.com/facebook/react/blob/83381c1673d14cd16cf747e34c945291e5518a86/src/renderers/shared/stack/reconciler/ReactMultiChild.js)独立于 renderer，实现子元素插入，删除，移动的操作队列的处理。
- `mount()`，`receive()`，和`unmount()`在 React 代码库中，因为遗留原因，其实叫做`mountComponent()`，`receiveComponent()`，和`unmountComponent()`，但是他们接受元素。
- 内部实例的属性使用一个下划线开始，比如，`_currentElement`。他们被认为是代码库中只读的。

### 未来方向

Stack reconciler 有固有的局限，比如同步，无法中断工作或者分离它到块。正在进行中的[新的 Fiber reconciler](https://reactjs.org/docs/codebase-overview.html#fiber-reconciler)使用[完全不同的架构](https://github.com/acdlite/react-fiber-architecture)。未来，我们会使用它替代 stack reconciler，但是现在，他还不是功能的一部分。

### 下一步

阅读[下一个章节](https://reactjs.org/docs/design-principles.html)去学习关于我们用于 React 开发的原则指南。
