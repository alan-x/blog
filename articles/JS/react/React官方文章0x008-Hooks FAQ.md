### Hooks FAQ

Hooks 是 React 16.8 引入的新内容。它们允许你不写一个类来使用状态和其他 React 功能。


### 引入策略

#### 哪一个版本的 React 包含 Hooks？

开始于 16.8.0，React 包含一个稳定的 React Hooks 实现：

- React DOM
- React Native
- React DOM Server
- React Test Renderer
- React Shallow Renderer

注意去启用 Hooks，所有的 React 包需要是 16.8.0 或者更高。Hooks 将不会工作，如果你忘记去更新，比如，React DOM。

React Native 0.59 之上支持 Hooks。

#### 我需要重写我的所有类组件？

不，没有从 React 移除类的计划 -- 我们都需要保持产品运输并且不能承受重写。我们推荐在新代码尝试 Hooks。

#### 我可以使用 Hooks 做到什么我无法使用类做到的？

Hooks 提供一个强大和表达的新方式在组件间去重用功能，"构建你自己的 Hooks"提供一个可能的概述。React 核心团队成员提供的这个文章深入解析 Hooks 解锁的新能力。

#### 我的多少 React 知识还保留？
Hooks 是一个更加直接的方式去使用你已经知道的 React 功能 -- 比如状态，生命周期，上下文，和 refs。他们没有从根本上改变 React 的工作方式，并且你关于 component，props，和自顶向下数据流的只是都保留着。

Hooks 的确有自己的学习曲线。如果有什么在这个文档缺失，提出一个问题，我们会尝试去帮助。

#### 我应该使用 Hooks，类，或者混合他们？

当你准备好，我们鼓励你去开始在你写的新组件尝试 Hooks。确保你团队的每一个人使用他们，并熟悉这个文档。我们不推荐重写你已经存在的类到 Hooks 知道你计划去重写他们（比如，修复 bug）。

你不能在类组件内部使用 Hooks，但是你可以定义混合类和使用 Hooks 的函数组件在一个单独的树。无论一个组件是一个类或者一个使用 Hooks 的函数都是组件实现的细节。换句话说，我们期待 Hooks 成为热门编写 React 组件的主要方式。

#### Hooks 覆盖所有类的使用场景吗？

我们对于 Hooks 的目标是覆盖类的所有使用场景，越快越好。没有 Hook 和不常用的 `getSnapshotBeforeUpdate`和`componentDidCatch`生命周期一样，但是我们计划尽快添加他们。

这是 Hooks 的早期时间，一些第三方库可能还没有和 Hooks 兼容。

#### Hooks 替代 render props 和 higher-order 组件？

通常，render props 和 higher-order 组件只渲染一个子组件。我们觉得 Hooks是服务于这个使用场景的简单方式。依旧还有他们这些模式的用处（比如，一个虚拟滚动组件可能哟一个`renderItem`属性，或者可视容器组件肯恩更有自己的 DOM 结构）。但是在大部分用力，Hooks 足够并可以帮助减少你的树的嵌套。

#### Hooks 对于像 Redux connect() 和 React Router 这类流行的 API 来说意味着啥？

你可以和往常一样继续使用完全相同的 API；他们依旧继续工作。

React Redux 从 v7.1.0 支持 Hooks API，并暴露出类似`useDispatch`或者`useSelector`的 hook。

React Route 从 v5.1 开始支持 hooks。

其他库可能在未来也支持 hooks。


####  Hooks 和静态类型一起用吗？

Hooks 在设计的时候考虑了静态类型。因为他们是函数，他们比 higher-order component 模式更容易输入正确。最新的 Flow 和 TypeScript React 定义包含对 React Hooks 的支持。

重要的是，自定义 Hooks 给你去约束 React API 的能力，如果你要以某种形式约束更严格。React 给你这个原语，但是你可以以不同的方式结合他们，不同于我们提供的开箱即用的。

#### 怎么测试使用 Hooks 的组件？

从 React 的视角来看，使用 Hooks 的组件就只是一个常规组件。如果你的测试解决方案不依赖 React 内部，测试使用 Hooks 的组件和测试普通组件没啥区别。

> 注意：Resting Recipes 包含很多你可以复制粘贴的例子。

比如，假设我们有一个计数器组件：
```jsx harmony
function Example() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
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

我们将使用 React DOM 测试它。为了确保和浏览器的行文一致，我们将使用`ReactTestUtils.act()`调用包裹代码的渲染和更新：
```jsx harmony
import React from 'react';
import ReactDOM from 'react-dom';
import { act } from 'react-dom/test-utils';
import Counter from './Counter';

let container;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
});

afterEach(() => {
  document.body.removeChild(container);
  container = null;
});

it('can render and update a counter', () => {
  // Test first render and effect
  act(() => {
    ReactDOM.render(<Counter />, container);
  });
  const button = container.querySelector('button');
  const label = container.querySelector('p');
  expect(label.textContent).toBe('You clicked 0 times');
  expect(document.title).toBe('You clicked 0 times');

  // Test second render and effect
  act(() => {
    button.dispatchEvent(new MouseEvent('click', {bubbles: true}));
  });
  expect(label.textContent).toBe('You clicked 1 times');
  expect(document.title).toBe('You clicked 1 times');
});
```

`act()`的调用也将会更新他们内部的 effect。

如果你需要测试自定义 Hook，你可以通过在你的测试创建一个组件，并在它内部使用 Hook。然后你就可以测试你写的组件。

为了减少样板，我们推荐使用 React Testing Library，它为鼓励使用你的组件就像最终用户使用一样写测试而设计。

了解更多信息，查阅 Testing Recipes。


#### lint rule 具体约束什么？

我们提供一个 ESLint plugin 应用 Hooks 的规则去避免 bug。假设任何函数以"use"开头并使用大写字符作为后续的开头，则这是一个 Hook。我们启发式的认为这不是最好并且可能存在误报，但是没有一个一个生态系统宽度的约定，无法让 Hooks 好好工作 -- 长名字将会阻止人份引入 Hooks 或者遵守约定。

特别是，规则执行：
- 在 `PascalCase` 函数内部（假设是一个组件）或者其他`useSomething`函数内部（假设是一个自动以 Hook）调用 Hook。
- Hooks 在每次渲染都以相同的顺序执行。

还有一些启发，他们可能随着时间改变，我们对规则的微调去平衡寻找 bug 和避免误报。

### 从类到 Hooks

#### 生命周期方法和 Hooks 如何对应？

- `constructor`：函数组件不需要一个构造器。你可以初始化在`useState`调用的状态。如果计算初始状态很昂贵，你可以传递一个函数给`userState`。

- `getDerivedStateFromProps`：使用更新时渲染。

- `shouldComponentUpdate`：查阅下面的`React.memo`。

- `render`：这就是函数组件体本身。

- `componentDidMount`，`componentDidUpdate`，`componentWillUnmount`：useEffect Hook 可以表示所有的结合（包含少量常见例子）。

- `componentDidCatch`和`getDerivedStateFromError`：还没有 Hook 和这些方法相同，但是他们马上就会被加进来。

#### Hooks 如何实现数据获取？

这里有一个小例子去开始。学习更多，可以查阅这篇文章关于使用 Hooks 获取数据。

#### 有类似 instance 的变量吗？

是的！`useRef()` Hook 不仅仅是一个 DOM refs。"ref"对象是一个通用容器，它的`current`属性是可变的，并且可以保存任何值，就像类的一个实例。

你可以在`useEffect()`内部写入它：
```jsx harmony
function Timer() {
  const intervalRef = useRef();

  useEffect(() => {
    const id = setInterval(() => {
      // ...
    });
    intervalRef.current = id;
    return () => {
      clearInterval(intervalRef.current);
    };
  });

  // ...
}
```

如果我们想要设置一个定时器，我们不需要这个 ref（`id`可以是 effect 的本地变量），但是如果我们需要从事件处理器清理定时器，它就很有用。

概念上，你可以认为 refs 和类中的实例变量一样。除非你使用 lazy initialization，避免在渲染的时候设置 refs -- 这会导致惊喜的行为。相反，通常你要在事件处理器和 effect 修改 refs。

#### 应该使用一个或者多个状态变量？

如果你从 class 来的，你可能会尝试总是调用`useState`一次放入所有的状态到一个单独的对象。你可以按你喜欢的做。这是一个组件跟随鼠标移动的例子。我们保存它的定位和大小在本地状态:
```jsx harmony
function Box() {
  const [state, setState] = useState({ left: 0, top: 0, width: 100, height: 100 });
  // ...
}
```

现在，假设我们想要写一些逻辑改变`left`和`top`，当用户移动他们的鼠标的时候。注意我们是怎样必须去手动合并这些域和之前的状态对象的：
```jsx harmony
  // ...
  useEffect(() => {
    function handleWindowMouseMove(e) {
      // Spreading "...state" ensures we don't "lose" width and height
      setState(state => ({ ...state, left: e.pageX, top: e.pageY }));
    }
    // Note: this implementation is a bit simplified
    window.addEventListener('mousemove', handleWindowMouseMove);
    return () => window.removeEventListener('mousemove', handleWindowMouseMove);
  }, []);
  // ...
```

这是因为当我饿们更新状态变量的时候，我们替换它的值。这和类中的`this.setState`不同，它合并更新的域到对象中。

如果你想念自动合并，你可以写一个自定义的`useLagacyState` Hook，合并对象状态更新。然而，相反，**我们推荐去分割状态到多个状态变量，基于哪些值将一起改变**。

比如，我们可以分割我们的组件状态到`position`和`size`对象，并且总是替换`position`，而不需要合并。
```jsx harmony
function Box() {
  const [position, setPosition] = useState({ left: 0, top: 0 });
  const [size, setSize] = useState({ width: 100, height: 100 });

  useEffect(() => {
    function handleWindowMouseMove(e) {
      setPosition({ left: e.pageX, top: e.pageY });
    }
    // ...
```
分离独立状态边领还有其他好处。它让它可以很简单的抽取一些相关逻辑到一个自定义 Hook，比如：
```jsx harmony
function Box() {
  const position = useWindowPosition();
  const [size, setSize] = useState({ width: 100, height: 100 });
  // ...
}

function useWindowPosition() {
  const [position, setPosition] = useState({ left: 0, top: 0 });
  useEffect(() => {
    // ...
  }, []);
  return position;
}
```
注意我们是怎样移动`position`状态变量的`useState`调用和相关 effect 到一个自定义 Hook 而没有改变他们的代码的。如果所有的状态都在一个单一的对象，抽取他们将更加困难。

将所有的状态放在一个单独的`useState`调用，或者为每一个域都有一个`useState`调用都行。组件趋向于易读，当你找到两种实践中的平衡，并分组相关状态到一些独立的状态变量的时候。如果状态逻辑变得复杂，我们推荐使用一个 reducer 管理，或者一个自定义 Hook。

#### 可以只在更新的时候执行 effect 吗？

这是一个罕见的例子。如果你需要它，你可以使用一个可变 ref 去手动存储一个布尔值表示你是否在第一次或者后续渲染，然后在你的 effect 检查这个标示（如果你发现你经常这么做，你可以为他创建一个自定义 Hook）。

#### 怎么获取前一个 props 或者 state？

现在，你可以手动使用 ref 来完成。
```jsx harmony
function Counter() {
  const [count, setCount] = useState(0);

  const prevCountRef = useRef();
  useEffect(() => {
    prevCountRef.current = count;
  });
  const prevCount = prevCountRef.current;

  return <h1>Now: {count}, before: {prevCount}</h1>;
}
```

```jsx harmony
function Counter() {
  const [count, setCount] = useState(0);

  const prevCountRef = useRef();
  useEffect(() => {
    prevCountRef.current = count;
  });
  const prevCount = prevCountRef.current;

  return <h1>Now: {count}, before: {prevCount}</h1>;
}

```

这可能有点复杂，但是你可以抽取到一个自定义 Hook：
```jsx harmony
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  return <h1>Now: {count}, before: {prevCount}</h1>;
}

function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```
注意它是怎样为 props，state，或者其他计算值生效的。
```jsx harmony
function Counter() {
  const [count, setCount] = useState(0);

  const calculation = count + 100;
  const prevCalculation = usePrevious(calculation);
  // ...

```
未来 React 可能会提供一个开箱即用的`userPrevious` Hook，因为这是一个常见的场景。

同时查阅分发状态的推荐模式。

#### 为什么在函数内部看到不新鲜的属性或者状态？

任何组件内的函数，包括事件处理器和 effect，都可以从创建它的渲染者"看到" props 和 state。比如，考虑这样的代码：
```jsx harmony
function Example() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```
如果你第一次点击"Show Alert"，然后增加计数器，alert 将会显示**你点击"Show alert"按钮时的**`count`变量。这防止因为 props 和 state 不改变的 bug。

如果你想要从异步回调读取最新的状态，你可以保存它到一个 ref，改变它，然后从它那里读取。

最后，你看到不新鲜的 props 或者 state 的其他可能的原因是如果你使用"依赖数组"优化，但是不正确的指定所有的依赖。比如，如果一个 effect 指定`[]`作为第二个参数，但是在内部读取`someProps`，它将会保持"看到"`someProp`的初始值。解决方案是移除依赖数组，或者修复它。如何解决函数，和其他常见策略尽可能少的执行 effect 没有正确跳过依赖。

> 注意：我们提供一个`exhaustive-deps` ESLint 规则作为`eslint-plugin-react-hooks`包的一部分。它在依赖指定不正确的时候警告，并建议一个修复。

#### 如何实现 getDerivedStateFromProps？

尽管你可能不需要它，在很罕见的场景你需要（比如实现一个`<Transition>`组件），你可以在渲染期间更新状态。React 将会使用更新的状态立即重新运行组件，在存在第一次渲染之后，所以它会很昂贵。
这里，我们存储`row`属性的前一个值在状态变量，这样我们就可以比较：
```jsx harmony
function ScrollView({row}) {
  let [isScrollingDown, setIsScrollingDown] = useState(false);
  let [prevRow, setPrevRow] = useState(null);

  if (row !== prevRow) {
    // Row changed since last render. Update isScrollingDown.
    setIsScrollingDown(prevRow !== null && row > prevRow);
    setPrevRow(row);
  }

  return `Scrolling down: ${isScrollingDown}`;
}
```
刚看起来可能很陌生，但是在渲染中更新完全和`getDerivedStateFromProps`的行为一致。

#### 有类似 forceUpdate 的吗？ 

`useState`和`useEffect` Hooks 跳伞更新，如果下一个值和前一个值一样。同样的操作状态调用`setState`也不会导致重新渲染。

通常你不应该在 React 中操作本地状态。然而，作为一个逃生舱，你可以使用一个自增的计数器去强制重新渲染，就算状态没有改变。
```jsx harmony
  const [ignored, forceUpdate] = useReducer(x => x + 1, 0);

  function handleClick() {
    forceUpdate();
  }

```
尽可能避免这种模式。

#### 可以为函数组件创建 ref？
尽管你并不需要经常雪瑶这个，你可以使用`useImperativeHandle` Hook 暴露一些紧急的方法给父组件。

#### 如何测量 DOM 节点？

为了测量 DOM 节点的位置和大小，你可以使用 callback ref。React 将会调用回调当 ref 绑定到不同的节点上的时候。这是一个小例子：
```jsx harmony
function MeasureExample() {
  const [height, setHeight] = useState(0);

  const measuredRef = useCallback(node => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  }, []);

  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  );
}
```
我们没有在这个例子选择`useRef`是因为一个对象 ref 不会提示我们当前 ref 值的改变。使用一个 callback  ref 确保就算如果一个子组件显示在测量节点之后（比如，点击响应），我们依旧得到关于负组件的通知，并且更新测量。

注意我们传递`[]`作为一个依赖数组到`useCallbak`。这确保我们的 ref callback 不会在重新渲染的时候改变，并且这样 React 不会在不需要的时候调用它。

如果你需要，你可以抽取这个逻辑到一个可重用的 Hook：
```jsx harmony
function MeasureExample() {
  const [rect, ref] = useClientRect();
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      {rect !== null &&
        <h2>The above header is {Math.round(rect.height)}px tall</h2>
      }
    </>
  );
}

function useClientRect() {
  const [rect, setRect] = useState(null);
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, []);
  return [rect, ref];
}
```

#### `const [thing, setThing] = useState()` 意味这啥？

如果你对这个语法不熟悉，查阅 State Hook 文档的解释。

### 性能优化

#### 可以在更新的时候跳过 effect？

可以，查阅条件化触发一个 effect。注意忘记去处理更新经常导致 bug，这也是为什么这不是默认行为。

#### 从依赖列表中缺省函数安全吗？

通常来说，不。
```jsx harmony
function Example({ someProp }) {
  function doSomething() {
    console.log(someProp);
  }

  useEffect(() => {
    doSomething();
  }, []); // 🔴 This is not safe (it calls `doSomething` which uses `someProp`)
}
```
很难去记住哪一个 props 或者 state 在 effect 外面的函数使用。这是为什么**通常你要声明 effect 函数内部需要的。** 很容易看出 effect 需要依赖函数范围内的哪些值：
```jsx harmony
function Example({ someProp }) {
  useEffect(() => {
    function doSomething() {
      console.log(someProp);
    }

    doSomething();
  }, [someProp]); // ✅ OK (our effect only uses `someProp`)
}
```
如果在这之后，我们依旧不使用组件范围的任何值，指定`[]`是安全的：
```jsx harmony
useEffect(() => {
  function doSomething() {
    console.log('hello');
  }

  doSomething();
}, []); // ✅ OK in this example because we don't use *any* values from component scope
```
取决于你的用户场景，还有一些选项表述在下面：

> 注意：我们提供 exhaustive-deps ESLint 规则作为 eslint-plugin-react-hooks 包的一部分。它帮助你去找出不需要处理更新的组件。

俩看看为什么这很重要。

如果你指定一个依赖列表作为`useEffect`，`useMemo`，`useCallback`，或者`useImperativeHandle`，它必须包含参与 React 数据流的所有值。包含 props，state，和任何从他们派生出来的。

缺省一个函数的依赖列表只有在它（或者函数通过它调用）没有任何对 props，state 或者从他们派生出来的值的引用的时候，才是安全的。这个例子有一个 bug：
```jsx harmony
function ProductPage({ productId }) {
  const [product, setProduct] = useState(null);

  async function fetchProduct() {
    const response = await fetch('http://myapi/product' + productId); // Uses productId prop
    const json = await response.json();
    setProduct(json);
  }

  useEffect(() => {
    fetchProduct();
  }, []); // 🔴 Invalid because `fetchProduct` uses `productId`
  // ...
}
```
**推荐的修复方式是移动函数到 effect 内部。** 这让看出哪一个 props 或者 state 被使用很简单，并确保他们都被声明
```jsx harmony
function ProductPage({ productId }) {
  const [product, setProduct] = useState(null);

  useEffect(() => {
    // By moving this function inside the effect, we can clearly see the values it uses.
    async function fetchProduct() {
      const response = await fetch('http://myapi/product' + productId);
      const json = await response.json();
      setProduct(json);
    }

    fetchProduct();
  }, [productId]); // ✅ Valid because our effect only uses productId
  // ...
}
```

这也允许你在 effect 内部使用一个本地变量处理无序的响应：
```jsx harmony
  useEffect(() => {
    let ignore = false;
    async function fetchProduct() {
      const response = await fetch('http://myapi/product/' + productId);
      const json = await response.json();
      if (!ignore) setProduct(json);
    }

    fetchProduct();
    return () => { ignore = true };
  }, [productId]);
```
我们移动函数到 effect 内部，这样它不需要在它的依赖列表。

> 提示：检出这个小例子和这篇文章了解更多关于关于使用 Hooks 获取数据。

如果因为一些原因你不能移动一个函数到一个 effect 内部，这又一些选项：

- **你可以尝试移动这个函数到你的组件外部。** 在这个场景中，函数保证不引用任何属性和状态，并且不需要被列在依赖中。

- 如果你调用的函数是一个纯计算并且在渲染的时候是安全调用的，你可能**在 effect 外面调用它，** 让 effect 依赖返回值。

- 作为最后手段，你可以**添加一个函数到 effect 依赖但是包裹它的定义**到`useEffect`Hook。这确保它在每一次渲染不改变明知道它自己的依赖也改变。

```jsx harmony
function ProductPage({ productId }) {
  // ✅ Wrap with useCallback to avoid change on every render
  const fetchProduct = useCallback(() => {
    // ... Does something with productId ...
  }, [productId]); // ✅ All useCallback dependencies are specified

  return <ProductDetails fetchProduct={fetchProduct} />;
}

function ProductDetails({ fetchProduct }) {
  useEffect(() => {
    fetchProduct();
  }, [fetchProduct]); // ✅ All useEffect dependencies are specified
  // ...
}
```
注意在前面的例子中，我们需要保持函数在依赖列表，这确保`productPage`的`productId`属性的改变自动触发`productDetails`组件的重新获取。

#### 如果我的 effect 依赖变动频繁该怎么办？

有时候，你的 effect 可能使用改变很频繁的状态。你可能尝试去从依赖列表缺省这个状态，但是这经常导致 bug。
```jsx harmony
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1); // This effect depends on the `count` state
    }, 1000);
    return () => clearInterval(id);
  }, []); // 🔴 Bug: `count` is not specified as a dependency

  return <h1>{count}</h1>;
}
```
空的依赖集合，`[]`，意味着 effect 只会运行一次，当组件挂载的时候，而不是每次渲染。这个问题在`setinterval`回调内部，`count`值没有改变，因为我们创建了一个闭包，它的`count`值为`0`，当这个 effect 回调执行的时候。每秒，回调都会调用`setCount(0 + 1)`，所以，计数器永远不会大于 1。

指定`[count]`作为依赖列表可以修复这个 bug，但是导致定时器在每一次改变都重新设置。实际上，每一次`setInterval`都会有一次机会在清理前被执行（和`setTimeout`类似）这可能不是希望的。为了修复它，我们可以使用`setState 的函数方式更新格式`。它让我们指定状态怎样不引用当前状态改变。
```jsx harmony
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c + 1); // ✅ This doesn't depend on `count` variable outside
    }, 1000);
    return () => clearInterval(id);
  }, []); // ✅ Our effect doesn't use any variables in the component scope

  return <h1>{count}</h1>;
}

```
`setCount`表示保证是稳定的，所以缺省是安全的。

现在，`setInterval`回调执行一秒一次，但是每次内部嗲用`setCount`都是用更新的`count`值（在回调内叫做`c`）。

在更复杂的场景（比如，如果状态依赖其他状态），尝试移动状态更新逻辑到 effect 之外，使用`useReducer`Hook。这篇文章提供一个怎样做的例子。**useReducer 中的分发函数的表示总是稳定的** -- 就算分发函数在组件内声明并读取它的属性。

最后方案，如果你想要一请类内的`this`，你可以使用一个 ref 去保存可变变量。则你可以写入和读取它，比如：
```jsx harmony
function Example(props) {
  // Keep latest props in a ref.
  let latestProps = useRef(props);
  useEffect(() => {
    latestProps.current = props;
  });

  useEffect(() => {
    function tick() {
      // Read latest props at any time
      console.log(latestProps.current);
    }

    const id = setInterval(tick, 1000);
    return () => clearInterval(id);
  }, []); // This effect never re-runs
}
```
只有在你找不到更好的替代的时候可以这么做，因为依赖可变让组件不可预料。如果有一个指定的模式不能很好的转化，使用可执行的代码提一个 issue，我们会尝试去帮助。 

#### 我该如何实现 shouldComponentUpdate？

你可以使用`React.memo`去浅比较它的属性：
```jsx harmony
const Button = React.memo((props) => {
  // your component
});
```
这不是一个 Hook，因为它不像 Hook 那样组合。`React.memo`和`Pure Component`相同，但是它只比较属性。（你也可以添加一个第二参数去指定一个自定义的比较函数，使用旧的和新的属性。如果返回 true，更新就会被跳过）。

`React.memo`不比较状态是因为没有单一的状态对象比较。但是你也可以创建子孙纯组件，或者使用`useMemo`优化独立的子孙。

#### 怎么记住计算？

`useMemo` Hook 让你在多次渲染之间缓存计算，通过"记住"起那一次计算：
```jsx harmony
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```
这个代码调用`computeExpensiveValue(a, b)`，但是如果依赖`[a, b]`从上一次值开始就没有改变，`useMemo`第二次跳过调用它，并简单使用前一个值返回。

记住传递给`useMemo`的函数在渲染期间执行。不要做你不会在渲染期间做的任何事情。比如，属于`useEffect`的副作用，不是`useMemo`。

**你可能依赖 useMemo 作为性能优化，不是语法保证。** 在未来，React 可能选择一些之前记录的值和计算他们在下一次渲染，比如，释放不再屏幕内的组件的内存。在没有`useMemo`的情况下你编写的代码也能执行 -- 并添加它到优化性能。（在很少见的场景，当一个值必须不被重新计算，你可以 lazily initialize 一个 ref）。

方便的，`useMemo`也让你跳过昂贵的子组件重渲染：
```jsx harmony
function Parent({ a, b }) {
  // Only re-rendered if `a` changes:
  const child1 = useMemo(() => <Child1 a={a} />, [a]);
  // Only re-rendered if `b` changes:
  const child2 = useMemo(() => <Child2 b={b} />, [b]);
  return (
    <>
      {child1}
      {child2}
    </>
  )
}
```
注意这个实现无法在循环内工作没因为 Hook 调用无法在循环内。但你可以为列表项抽取一个独立的组件，并调用`useMemo`。

#### 如何惰性创建昂贵的对象？
`useMemo`让你记住一个昂贵的计算，如果依赖都是相同的。然而，它只作为一个窍门使用，并且不保证计算不会重新执行。但是有时候你需要保证一个对象只创建一次。

第一个常见的使用场景就是当创建一个出事状态很昂贵：
```jsx harmony
function Table(props) {
  // ⚠️ createRows() is called on every render
  const [rows, setRows] = useState(createRows(props.count));
  // ...
}
```
为了避免重新创建忽略出事状态，我们可以传递一个函数到`useState`：
```jsx harmony
function Table(props) {
  // ✅ createRows() is only called once
  const [rows, setRows] = useState(() => createRows(props.count));
  // ...
}
```
React 将会只调用这个函数一次，在第一次渲染的时候，查看`useState` API
索引。

**你可能偶尔也需要避免重新创建 useRef() 初始状态**。比如，可能你要保证一些重要的类实例只创建一次：
```jsx harmony
function Image(props) {
  // ⚠️ IntersectionObserver is created on every render
  const ref = useRef(new IntersectionObserver(onIntersect));
  // ...
}
```
`useRef`**不**接收一个指定的函数，像`useState`。相反，你可以编写你自己的函数惰性创建和设置：
```jsx harmony
function Image(props) {
  const ref = useRef(null);

  // ✅ IntersectionObserver is created lazily once
  function getObserver() {
    if (ref.current === null) {
      ref.current = new IntersectionObserver(onIntersect);
    }
    return ref.current;
  }

  // When you need it, call getObserver()
  // ...
}

```

这避免创建一个昂贵的对象，直到第一次它真的需要。如果你使用 Flow 或者 TypeScript，你也可以方便的给`getObserver`一个非可空类型。

#### Hooks 会因为在 render 创建函数变慢吗？
不。在现代浏览器，闭包的原始性能和类相比，没有很大的差别，除非在极端的情况下。

此外，考虑到 Hooks 的设置在两个方面更有效：
- Hooks 避免大量类的开销，比如创建类实例和在构造器内绑定事件处理的消耗
- **惯用代码使用 Hooks 不需要深组件树内嵌**，防止使用 higher-order component，render props，和 context。更小的组件树，React 工作更少。

一般来说，React 性能关心行内函数和怎样传递新的回调到每一个子组件渲染打破`shouldComponentUpdate`优化。Hooks 从树的方面解决这个问题。

- `useCallback` Hook 然你保持相同的回调引用，在重渲染之间，这样`shouldComponentUpdate`继续工作：
```jsx harmony
// Will not change unless `a` or `b` changes
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```
- `useMemo` Hook 让它更容易控制，当独立子孙更新，减少纯组件的需要。

- 最后，`useReducer` Hook 减少深层传递回调的需要，就像下面解释的。

#### 怎么避免向下传递回调？

#### 怎么从 useCallback 读取经常改变的值？

> 注意：
> 我们推荐在上下文向下传递分发，而不是属性中独立的回调。下面的实现只是在这里提示完备性和作为逃生舱。
> 同时注意这个模式可能在`concurrent mode`导致问题。我们计划在未来提供更人性化的替代，但是现在最安全的解决方案是总是无效回调，如果一些值的依赖改变了。

在一些罕见的场景，你需要记住一个`userCallback`使用的回调，但是记录并不工作，因为内部的函数经常被冲创建。如果你要记住的函数是一个事件处理器，并且不再渲染期间使用，你可以使用 ref 作为一个实例变量，手动保存最新提交的值到它。
```jsx harmony
function Form() {
  const [text, updateText] = useState('');
  const textRef = useRef();

  useEffect(() => {
    textRef.current = text; // Write it to the ref
  });

  const handleSubmit = useCallback(() => {
    const currentText = textRef.current; // Read it from the ref
    alert(currentText);
  }, [textRef]); // Don't recreate handleSubmit like [text] would do

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}
```
这是一个相当复杂的模式，但是它显示你可以这么优化，如果你需要它。它会更容易忍受，如果你将它抽取到一个自定义 Hook。
```jsx harmony
function Form() {
  const [text, updateText] = useState('');
  // Will be memoized even if `text` changes:
  const handleSubmit = useEventCallback(() => {
    alert(text);
  }, [text]);

  return (
    <>
      <input value={text} onChange={e => updateText(e.target.value)} />
      <ExpensiveTree onSubmit={handleSubmit} />
    </>
  );
}

function useEventCallback(fn, dependencies) {
  const ref = useRef(() => {
    throw new Error('Cannot call an event handler while rendering.');
  });

  useEffect(() => {
    ref.current = fn;
  }, [fn, ...dependencies]);

  return useCallback(() => {
    const fn = ref.current;
    return fn();
  }, [ref]);
}
```
这两种场景，我们**都不推荐这种模式**并且只为了完备性展示它。相反，最好避免向下传递回调。

### Hook 底层

#### React 如何关联 Hook 调用和组件？

React 跟踪当前渲染的组件。感谢 Hooks 的规则，我们知道 Hooks 只在 React 组件内调用（或者自定义 Hooks -- 也只在 React 组件内调用）。

有一个内部的"内存单元"列表关联每一个组件。他们只是 JavaScript 对象，我们输入一下数据。当你调用一个 Hook，比如`useState()`，它读取当前但愿或者初始化它，在第一次渲染（，然后移动指针到下一个）。这是多个`useState()`调用每一个都有独立本地状态的原因。

#### Hooks 的先前的艺术是啥？

Hooks 综合多个不同来源的想法。

- 我们关于函数 API 老的经验在 react-future 库。

- React 社区关于 render props API 的经验，包含 Ryan Florence 的 Reaction Component。

- Dominic Gannaway 的使用关键字作为渲染属性语法糖的提案。

- DisplayScript 中状态变量和状态单元。

- ReasonReact 中的 Reducer components。

- Rx 中的 Subscriptions

- Multicore OCaml 中的 Algebraic effect 

Sebastian Markbåge 提出了 Hooks 的最初设计，然后被 Andrew Clark，Sophie Alpert，Dominic Gannaway，和 React team 中的其他成员重新定义。

