[原文地址](https://reactjs.org/docs/accessibility.html)
### 为什么可访问？

网页可访问性（也称为[a11y](https://en.wiktionary.org/wiki/a11y)）是可以被任何人使用的网站设计和创作。可访问性支持对于允许访问技术去解释网站是非常必要的。

React 完全支持构建可访问性网站，通常通过使用标砖 HTML 技术。

### 标准和指南

### WCAG

[网页内容可访问性指南](https://www.w3.org/WAI/intro/wcag)提供创建可访问性网站的指南。

下面 WCAG 清单提供一个概述：

- [Wuhcag 的 WCAG 清单](https://www.wuhcag.com/wcag-checklist/)
- [WebAIM 的 WCAG 清单](https://webaim.org/standards/wcag/checklist)
- [A11Y 项目的清单](https://a11yproject.com/checklist.html)

### WAI-ARIA

[网页可访问性倡议-可访问性富互联网应用](https://www.w3.org/WAI/intro/aria)文档包含构建完全可访问性 JavaScript 插件的技术。

注意所有`aria-*`HTML 在 JSX 中被完全支持。虽然 React 中大部分 DOM 特性和属性都是驼峰的，这些属性应该是连字符链接的（kebab-case，lisp-case，等），就像他们在普通 HTML 中：
```jsx harmony
<input
  type="text"
  aria-label={labelText}
  aria-required="true"
  onChange={onchangeHandler}
  value={inputValue}
  name="name"
/>
```

### HTML 语义化

HTML 语义化在网页应用中是可访问性的基础。使用不同的 HTML 元素增强我们网站信息的含义，通常让我们免费得到可访问性。

- [MDN HTML 元素索引](https://developer.mozilla.org/en-US/docs/Web/HTML/Element)

有时候我们破坏 HTML 语义化，当我们添加`<div>`元素到我们的 JSX，让我们的 React 代码可用，特别是使用列表（`<ol>`，`<ul>`和`<di>`）和 HTML `<table>`。在这些场景，我们应该使用[React Fragment](https://reactjs.org/docs/fragments.html)去分组多个元素。

比如，
```jsx harmony
import React, { Fragment } from 'react';

function ListItem({ item }) {
  return (
    <Fragment>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </Fragment>
  );
}

function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        <ListItem item={item} key={item.id} />
      ))}
    </dl>
  );
}
```

你可以映射一个集合的项目到一个片段数组，就像其他类型的元素一样：
```jsx harmony
function Glossary(props) {
  return (
    <dl>
      {props.items.map(item => (
        // Fragments should also have a `key` prop when mapping collections
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```
如果你在 Fragment 标签上不需要任何属性，可以使用[短语法](https://reactjs.org/docs/fragments.html#short-syntax)，如果你的工具支持它：
```jsx harmony
function ListItem({ item }) {
  return (
    <>
      <dt>{item.term}</dt>
      <dd>{item.description}</dd>
    </>
  );
}
```

了解更多信息，查阅[Fragment 文档](https://reactjs.org/docs/fragments.html)。

### 可访问性表单

### 标签

每一个 HTML 表单控件，比如`<input>`和`<textarea>`，需要标记可访问性。我们需要去提供暴露给屏幕阅读器的描述性标签。

下面的资源给我们展示如何做到这个：

- [W3C 给我们展示如何标记元素](https://www.w3.org/WAI/tutorials/forms/labels/)

- [WebAIM 给我们展示如何标记元素](https://webaim.org/techniques/forms/controls)

- [Paciello Group 解释可访问性命名](https://www.paciellogroup.com/blog/2017/04/what-is-an-accessible-name/)

尽管这些标准 HTML 实践可以直接用于 React，注意`for`属性在 JSX 写作：
```jsx harmony
<label htmlFor="namedInput">Name:</label>
<input id="namedInput" type="text" name="name"/>
```

### 通知用户错误

错误场景需要被所有的用户理解。下面的链接给我们展示怎样暴露错误文字给屏幕阅读器：

- [W3C 展示用户通知](https://www.w3.org/WAI/tutorials/forms/notifications/)

- [WebAIM 查看表单验证](https://webaim.org/techniques/formvalidation/)

### 关注控制

确保你的网页应用可以完全被键盘操作：

- [WebAIM 讨论键盘可访问性](https://webaim.org/techniques/keyboard/)

### 键盘焦点和焦点轮廓

键盘焦点指向 DOM 当前选中的接收键盘输入的元素。我们在任何地方看到他，焦点轮廓类似下面显示的图片：

![](https://reactjs.org/static/keyboard-focus-dec0e6bcc1f882baf76ebc860d4f04e5-9d63d.png)

只使用删除这个轮廓的 CSS，比如，通过设置`outline: 0`，如果你使用其他焦点轮廓实现来替代它。

### 跳转到期待内容的机制

在你的应用提供一个机制允许用户跳过导航章节，因为这帮助和加速键盘导航。

跳过链接和跳过导航链接是隐含的导航链接，只有当键盘用户和页面交互的时候才可见。使用内部页面锚和一些样式很容易实现：

- [WebAIM - 跳转导航链接](https://webaim.org/techniques/skipnav/)

也可以使用地标元素和角色，比如`<main>`和`<aside>`，去划分页面区域，因为辅助技术允许用户快速导航到这些章节。

了解更多关于这些元素的使用去增强可访问性：

- [可访问性标志](https://www.scottohara.me/blog/2018/03/03/landmarks.html)

### 编码方式管理焦点

我们的 React 应用在运行期持续修改 HTML DOM，有时候导致键盘焦点丢失或者设置到不期待的元素。为了修复这个，我们需要去程序化推动键盘焦点正确。比如，通过在模态窗口关闭之后重设键盘焦点到一个打开一个模态窗口的按钮。

MDN 网页文档关注了这个，并描述我们怎样构建[可键盘导航的 JavaScript 元素](https://developer.mozilla.org/en-US/docs/Web/Accessibility/Keyboard-navigable_JavaScript_widgets)。

为了在 React 中设置焦点，我们可以使用[ DOM 元素引用](https://reactjs.org/docs/refs-and-the-dom.html)。

使用这个，首先在一个组件类的 JSX 创建一个 ref 到元素：
```jsx harmony
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    // Create a ref to store the textInput DOM element
    this.textInput = React.createRef();
  }
  render() {
  // Use the `ref` callback to store a reference to the text input DOM
  // element in an instance field (for example, this.textInput).
    return (
      <input
        type="text"
        ref={this.textInput}
      />
    );
  }
}
```
然后需要的时候，我们可以在我们组件中聚焦它：
```jsx harmony
focus() {
  // Explicitly focus the text input using the raw DOM API
  // Note: we're accessing "current" to get the DOM node
  this.textInput.current.focus();
}
```

有时候，一个父组件需要设置焦点到一个子组件的元素。我们可以通过[暴露 DOM ref 到父组件](https://reactjs.org/docs/refs-and-the-dom.html#exposing-dom-refs-to-parent-components)来做到，通过一个子组件上一个特殊的属性，转发父组件的 ref 到子组件的 DOM 节点。

```jsx harmony
function CustomTextInput(props) {
  return (
    <div>
      <input ref={props.inputRef} />
    </div>
  );
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.inputElement = React.createRef();
  }
  render() {
    return (
      <CustomTextInput inputRef={this.inputElement} />
    );
  }
}

// Now you can set focus when required.
this.inputElement.current.focus();
```

当使用一个 HOC 到扩展组件，推荐使用 React 的`forwardRef`函数去[转发 ref](https://reactjs.org/docs/forwarding-refs.html)到包裹的组件。如果一个第三方 HOC 没有实现 ref 转发，前面的模式依旧可以作为一个回滚。

一个好的焦点管理例子是[react-aria-modal](https://github.com/davidtheclark/react-aria-modal)。这是一个相当罕见的完全可访问的模态窗口例子。它不仅仅设置初始的焦点到取消按钮（防止键盘用户意外激活成功动作），并捕捉键盘焦点到模态内，它也重设焦点到最初触发模态窗口的元素。

注意：尽管这是非常重要的可访问性特性，这也是一个需要谨慎使用的技术。使用它去修复键盘焦点流，不要去尝试和预期用户怎样去使用应用。

### 鼠标和指针事件

确保所有暴露给鼠标或者指针实践的功能也可以被键盘访问。只取决于指针设备会导致很多键盘用户无法使用你的应用的场景。

为了描述这个，看看这个点击事件导致可访问性破坏的丰富例子。这是外部点击模式，一个用户可以通过点击元素外部禁用一个打开的 popover。

![](https://reactjs.org/outerclick-with-mouse-5523b05b22210c5a2fa0bd1f01339cb3.gif)

通常这通过绑定一个关闭 popover 的`click`事件到`window`对象：
```jsx harmony
class OuterClickExample extends React.Component {
  constructor(props) {
    super(props);

    this.state = { isOpen: false };
    this.toggleContainer = React.createRef();

    this.onClickHandler = this.onClickHandler.bind(this);
    this.onClickOutsideHandler = this.onClickOutsideHandler.bind(this);
  }

  componentDidMount() {
    window.addEventListener('click', this.onClickOutsideHandler);
  }

  componentWillUnmount() {
    window.removeEventListener('click', this.onClickOutsideHandler);
  }

  onClickHandler() {
    this.setState(currentState => ({
      isOpen: !currentState.isOpen
    }));
  }

  onClickOutsideHandler(event) {
    if (this.state.isOpen && !this.toggleContainer.current.contains(event.target)) {
      this.setState({ isOpen: false });
    }
  }

  render() {
    return (
      <div ref={this.toggleContainer}>
        <button onClick={this.onClickHandler}>Select an option</button>
        {this.state.isOpen && (
          <ul>
            <li>Option 1</li>
            <li>Option 2</li>
            <li>Option 3</li>
          </ul>
        )}
      </div>
    );
  }
}
```

这在指针设备的用户可能没问题，比如鼠标，但是使用键盘可能导致功能破坏，当跳转到下一个元素的时候，因为`window`对象没有接收到一个`click`事件。这个会导致功能模糊，阻塞用户使用你的应用。

![](https://reactjs.org/outerclick-with-keyboard-eca0ca825c8c5e2aa609cee72ef47e27.gif)

相同的功能可以通过使用适合的事件处理器替代来达到，比如`onBlur`和`onFocus`：
```jsx harmony
class BlurExample extends React.Component {
  constructor(props) {
    super(props);

    this.state = { isOpen: false };
    this.timeOutId = null;

    this.onClickHandler = this.onClickHandler.bind(this);
    this.onBlurHandler = this.onBlurHandler.bind(this);
    this.onFocusHandler = this.onFocusHandler.bind(this);
  }

  onClickHandler() {
    this.setState(currentState => ({
      isOpen: !currentState.isOpen
    }));
  }

  // We close the popover on the next tick by using setTimeout.
  // This is necessary because we need to first check if
  // another child of the element has received focus as
  // the blur event fires prior to the new focus event.
  onBlurHandler() {
    this.timeOutId = setTimeout(() => {
      this.setState({
        isOpen: false
      });
    });
  }

  // If a child receives focus, do not close the popover.
  onFocusHandler() {
    clearTimeout(this.timeOutId);
  }

  render() {
    // React assists us by bubbling the blur and
    // focus events to the parent.
    return (
      <div onBlur={this.onBlurHandler}
           onFocus={this.onFocusHandler}>

        <button onClick={this.onClickHandler}
                aria-haspopup="true"
                aria-expanded={this.state.isOpen}>
          Select an option
        </button>
        {this.state.isOpen && (
          <ul>
            <li>Option 1</li>
            <li>Option 2</li>
            <li>Option 3</li>
          </ul>
        )}
      </div>
    );
  }
}
```
这段代码暴露功能给指针设备和键盘用户。也要注意添加`aria-*`属性去支持屏幕阅读器用户。为了简单起见，`箭头键·
和弹窗交互的功能并未实现。

![](https://reactjs.org/blur-popover-close-28ce2067489843caf05fe7ce22494542.gif)

这是众多只依赖指针和鼠标事件会破坏键盘用户的功能的场景中的一个。总是使用键盘测试将立马发现问题区域，并且使用键盘事件处理器修复。

### 更复杂的组件

一个更复杂的用户体验不意味着更少的可访问性。尽管可访问性尽可能编码为 HTML 最简单，就算是最复杂的组件也可以编写的可访问。

这里我们需要[ARIA 角色](https://www.w3.org/TR/wai-aria/#roles)和[ARIA 状态和属性](https://www.w3.org/TR/wai-aria/#states_and_properties)的知识。有一个使用 HTML 填充的工具箱，完全在 JSX 中支持，并可以构建完全可访问的，高功能的 React 组件。

每一种类型的组件都有一个特定的设计模式，用户和用户代理希望以某种方式工作。

- [WAI-ARIA 作者实践 - 设计模式和组件](https://www.w3.org/TR/wai-aria-practices/#aria_ex)

- [Heydon Pickering - ARIA 例子](https://heydonworks.com/article/practical-aria-examples/)

- [元组件](https://inclusive-components.design/)

### 其他考虑要点

### 设置语言

指示页面文本的人类语言，因为屏幕阅读器软件使用这个去选择正确的语音设置：

- [WebAIM - 文档语言](https://webaim.org/techniques/screenreader/#language)

### 设置文档标题

设置文档的`<title>`到当前页面内容正确的描述，因为这确保用户意识到当前页面的上下文：

- [WCAG - 理解文档标题要求](https://www.w3.org/TR/UNDERSTANDING-WCAG20/navigation-mechanisms-title.html)

我们可以使用[React 文档标题组件](https://github.com/gaearon/react-document-title)在 React 中设置这个。

### 颜色对比

确保你的网站所有可读的文字有足够的颜色对照去尽可能保持低视力的用户可读：

- [WCAG - 理解颜色对照要求](https://www.w3.org/TR/UNDERSTANDING-WCAG20/visual-audio-contrast-contrast.html)

- [关于颜色对照的任何东西和为什么你需要重新思考它](https://www.smashingmagazine.com/2014/10/color-contrast-tips-and-tools-for-accessibility/)

- [A11yProject - 什么是颜色对照](https://a11yproject.com/posts/what-is-color-contrast/)

在你的网站手动计算适合的颜色结合所有的场景是很冗长的，因此，你可以[使用 Colorable 去计算整个可访问性色板](https://jxnblk.com/colorable/)。

下面提到的 aXe 和 WAVE 工具都包含颜色对照测试并报告对照错误。

- [WebAIM - 颜色对照检查器](https://webaim.org/resources/contrastchecker/)

- [Paciello 组 - 颜色对照分析器](https://www.paciellogroup.com/resources/contrastanalyser/)

### 开发和测试工具

有多种工具可以用于协助创建可访问的网页应用。 

### 键盘

到目前为止也是最重要的一个检测是测试你的整个网站可以只使用键盘单独访问。通过这样做：

1. 断开对鼠标的连接

2. 在浏览器使用`Tab`和`Shift+Tab`。

3. 使用`Enter`去激活元素。

4. 在需要的地方，使用你的键盘的箭头键去和一些元素交互，比如菜单和下拉。

### 开发助手

我们可以直接在我们的 JSX 代码检测一些可访问性功能。通常能识别 JSX 的 IDE 已经提供了对 ARIA 角色，状态和属性的敏感检测。我们也有下列工具可以使用：

#### eslint-plugin-jsx-a11y

[eslint-plugin-jsx-a11y](https://github.com/evcohen/eslint-plugin-jsx-a11y) ESLint 插件提供了 JSX 中可访问性问题的 SDT 反馈。很多 IDE 允许你去直接将这些发现加入到代码分析和源代码窗口。

[Create React App](https://github.com/facebookincubator/create-react-app)已经内置这个插件并启用了一些规则集合。如果你想要启用更多可访问性规则，你可以创建一个`.eslintrc`文件到你的项目的根目录，内容如下：
```jsx harmony
{
  "extends": ["react-app", "plugin:jsx-a11y/recommended"],
  "plugins": ["jsx-a11y"]
}
```

### 在浏览器测试可访问性

存在大量工具可以在你的浏览器上对你的网页运行可访问性检测。请使用他们和其他这里提到的可访问性检测结合，因为他们只能检测你的 HTML 的技术的可访问性。

#### aXe，aXe-core 和 react-axe

Deque System 提供[aXe-core](https://github.com/dequelabs/axe-core)自动和端到端可访问性测试。这个模块包含完整的 Selenium。

[可访问性引擎](https://www.deque.com/products/axe/)或 aXe，是一个基于`aXe-core`构建的可访问性检查器浏览器插件。

你也可以在开发和调试的时候使用[react-axe](https://github.com/dylanb/react-axe)模块去报告这些可访问性发现到控制台。

#### WebAIM WAVE
[网页可访问性执行工具](https://wave.webaim.org/extension/)是另一个浏览器可访问性插件。

#### 可访问性检查器和可访问性树

[可访问性树](https://www.paciellogroup.com/blog/2015/01/the-browser-accessibility-tree/)是包含每一个 DOM 元素的可访问对象的 DOM 树集合，应该暴露给辅助技术，比如屏幕阅读器。

在一些浏览器，我们可以简单的浏览可访问树中的每一个元素的可访问信息。

- [在 Firefox 中使用可访问性检查器](https://developer.mozilla.org/en-US/docs/Tools/Accessibility_inspector)
- [在 Chrome 中使用可访问性检查器](https://developers.google.com/web/tools/chrome-devtools/accessibility/reference#pane)
- [在 OS X Safari 中使用可访问性检查器](https://developer.apple.com/library/content/documentation/Accessibility/Conceptual/AccessibilityMacOSX/OSXAXTestingApps.html)

### 屏幕阅读者

### 常用屏幕阅读者

#### Firefox 的 NVDA

[NonVisual Desktop Access](https://www.nvaccess.org/)或者 NVDA 是一个开源的广泛使用的 Windows 屏幕阅读器。

查询下列指南了解怎样最好的使用 NVDA：

- [WebAIM - 使用 NVDA 去评估网页可访问性](https://webaim.org/articles/nvda/)

- [Deque - NVDA 键盘快捷键](https://dequeuniversity.com/screenreaders/nvda-keyboard-shortcuts) 

#### Safari 的 VoiceOver

VoiceOver 是 Apple 设备上集成的屏幕阅读器。

查阅下列指南了解怎样激活和使用 VoiceOver：

- [WebAIM - 使用 VoiceOver 去评估网页可访问性](https://webaim.org/articles/voiceover/)

- [Deque - VoiceOver 的 OS X 键盘快捷键](https://dequeuniversity.com/screenreaders/voiceover-keyboard-shortcuts)
- [Deque - VoiceOVer 的 IOS 快捷键](https://dequeuniversity.com/screenreaders/voiceover-ios-shortcuts)

#### Internet Explore 的 JAWS

[Job Access With Speech](https://www.freedomscientific.com/Products/software/JAWS/)或者 JAWS，是 windows 上经常使用的屏幕阅读器。

查询下列指南了解怎样最好的使用 JAWS：

- [WebAIM - 使用 JAWS 去评估网页可访问性](https://webaim.org/articles/jaws/)
- [Deque - JAWS 键盘快捷键](https://dequeuniversity.com/screenreaders/jaws-keyboard-shortcuts)

### 其他屏幕阅读者

### Google Chrome 的 ChromeVox

[ChromeVox](https://www.chromevox.com/) 是 Chromebooks 中集成的屏幕阅读器，可以作为 Google Chrome 的[插件](https://chrome.google.com/webstore/detail/chromevox/kgejglhpjiefppelpmljglcjbhoiplfn?hl=en)。

查询下列指南，了解如何最好的使用 ChromeVox：

- [Google Chromebook 帮助 - 使用内置屏幕阅读器](https://support.google.com/chromebook/answer/7031755?hl=en)

- [ChromeVox 经典键盘快捷键索引](https://www.chromevox.com/keyboard_shortcuts.html) 
