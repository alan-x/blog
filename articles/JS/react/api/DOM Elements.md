[原文地址](https://reactjs.org/docs/dom-elements.html)
### DOM 元素

为了性能和跨浏览器兼容，React 实现了一个不依赖浏览器的 DOM 系统。我们借此机会清理了一些浏览器 DOM 实现粗糙的边角。

在 React，所有的 DOM 属性和特征（包括事件处理器）应该是驼峰。比如，HTML 特征`tabindex`对应 React 的`tabIndex`。除了`aria-*`和`data-*`属性，应该是小写。比如，你可以保持`aria-label`为`aria-label`。

### 属性的不同

### checked

`checked`属性被`<input>`组件的`checkbox`或者`radio`类型支持。你可以使用它设置组件是否选中。这对于构建受控组件很有用。`defaultChecked`相当于未受控，它设置组件第一次挂载的时候是否选中。

### className

为了指定一个 CSS 类，使用`className`属性。这应用于所有的常规 DOM 和 SVG 元素，比如`<div>`，`<a>`，和其他。

如果使用 React 和 Web Component（不常见），使用`class`属性代替。

### dangerouslySetInnerHTML

`dangerouslySetInnerHTML`是 React 替换浏览器 DOM 中`innerHTML`的使用。通常，从代码设置 HTML 是有风险的，因为这很容易无意的让你的用户暴露在[跨站脚本（SXX）]()攻击中。因此，你可以直接从 React 设置 HTML，但是你需要打出`dangerouslySetInnerHTML`并传递一个携带`__html`键的对象，为了让你意识到这是危险的。比如：
```jsx harmony
function createMarkup() {
  return {__html: 'First &middot; Second'};
}

function MyComponent() {
  return <div dangerouslySetInnerHTML={createMarkup()} />;
}
```

### htmlFor

因为`for`是 JavaScript 的保留关键字，React 元素使用`htmlFor`替代。

### onChange

`onChange`事件行为如你期待的：当表单域改变的时候，这个事件被触发。我们特意不使用已存在的浏览器行为。因为`onChange`是它行为的错用，并且 React 依赖这个事件去实时处理用户输入。

### selected

`selected`属性被`<option>`组件支持。你可以使用它设置组件是否被选择。这对构建受控组件很有用。

### style

注意：这个文档的一些例子为了方便，使用`style`，但是**使用 style 属性作为排版元素的主要手段通常是不推荐的**。在大部分场景，[className]()应该用来引用定义在外部 CSS 样式表的类。`style`大部分在 React 应用中用来在渲染的时候添加动态计算的样式。查阅[FAQ:样式和 CSS]()

`style`属性接收一个携带驼峰属性而不是 CSS 字符串的 JavaScript 对象。这和 DOM `style` JavaScript 属性保持一致，更有效，并且防止 XSS 安全漏洞。比如：
```jsx harmony
const divStyle = {
  color: 'blue',
  backgroundImage: 'url(' + imgUrl + ')',
};

function HelloWorldComponent() {
  return <div style={divStyle}>Hello World!</div>;
}
```

注意样式不是自动添加前缀的。为了支持旧的浏览器，你需要支持对应的样式属性：
```jsx harmony
const divStyle = {
  WebkitTransition: 'all', // note the capital 'W' here
  msTransition: 'all' // 'ms' is the only lowercase vendor prefix
};

function ComponentWithTransition() {
  return <div style={divStyle}>This should work cross-browser</div>;
}

```
样式键是驼峰的，为了和 JS 中访问 DOM 属性保持一致（比如，`node.style.backgroudImage`）。提供商前缀[除了 ms](https://www.andismith.com/blogs/2012/02/modernizr-prefixed/)应该以大写字母开头。这也是`WebkitTransition`有一个大写的"W"。

React 将会自动添加一个"px"后缀到某些内联样式属性。如果你想要去使用非"px"的单位，指定值作为字符串，携带期望的单位。比如：
```jsx harmony
// Result style: '10px'
<div style={{ height: 10 }}>
  Hello World!
</div>

// Result style: '10%'
<div style={{ height: '10%' }}>
  Hello World!
</div>
```
并不是所有的样式属性都转化为像素字符串。某些依旧没有单位，（比如`zoom`，`order`，`flex`）。一个完整的无单位属性可以在[这里](https://github.com/facebook/react/blob/4131af3e4bf52f3a003537ec95a1655147c81270/src/renderers/dom/shared/CSSProperty.js#L15-L59)看见

### suppressContentEditableWarning

通常，当元素的子元素被标记为`contentEditable`，有这个警告，因为它不工作。这个属性覆盖这个警告。不要使用这个，除非你构建一个类似[Draft.js](https://facebook.github.io/draft-js/)这样手动管理`contentEditable`的类。

### suppressHydrationWarning

如果你使用 React 服务端渲染，当服务端和客户端渲染不同内容的时候有这个警告。然而，在某些罕见的场景，很难或者不可能去保证精确匹配。比如，时间戳在服务端和客户端被认为是不同的。

如果你设置`suppressHydrationWarning`为`true`，React 将不会警告你关于错误匹配元素的内容和属性。它只对一层深度起作用，并且只作为紧急出口。不要过分使用这个。你可以在[ReactDOM.hydrate() 文档](https://reactjs.org/docs/react-dom.html#hydrate)了解更多关于混合。

### value

`<input>`和`<textarea>`组件支持`value`属性。你可以使用它设置组件的值。这对于构建受控组件有用。`defaultValue`相当于未受控，设置组件第一次挂载的值。

### 所有支持的 HTML 属性。

自 React 16 起，任何标准[或者自定义]()DOM 属性都是完全支持的。

React 总是提供一个以 JavaScript 为中心的 DOM API。因为 React 组件总是接收自定义和 DOM 相关的属性，React 使用`驼峰`约定，就像 DOM API：
```jsx harmony
<div tabIndex="-1" />      // Just like node.tabIndex DOM API
<div className="Button" /> // Just like node.className DOM API
<input readOnly={true} />  // Just like node.readOnly DOM API
```
这些属性和对应的 HTML 属性工作类似，除了前面记录的特定例子。

React 支持的 DOM 属性包括：
```jsx harmony
accept acceptCharset accessKey action allowFullScreen alt async autoComplete
autoFocus autoPlay capture cellPadding cellSpacing challenge charSet checked
cite classID className colSpan cols content contentEditable contextMenu controls
controlsList coords crossOrigin data dateTime default defer dir disabled
download draggable encType form formAction formEncType formMethod formNoValidate
formTarget frameBorder headers height hidden high href hrefLang htmlFor
httpEquiv icon id inputMode integrity is keyParams keyType kind label lang list
loop low manifest marginHeight marginWidth max maxLength media mediaGroup method
min minLength multiple muted name noValidate nonce open optimum pattern
placeholder poster preload profile radioGroup readOnly rel required reversed
role rowSpan rows sandbox scope scoped scrolling seamless selected shape size
sizes span spellCheck src srcDoc srcLang srcSet start step style summary
tabIndex target title type useMap value width wmode wrap
``` 
同样的，所有的 SVG 属性是完全支持的：
```jsx harmony
accentHeight accumulate additive alignmentBaseline allowReorder alphabetic
amplitude arabicForm ascent attributeName attributeType autoReverse azimuth
baseFrequency baseProfile baselineShift bbox begin bias by calcMode capHeight
clip clipPath clipPathUnits clipRule colorInterpolation
colorInterpolationFilters colorProfile colorRendering contentScriptType
contentStyleType cursor cx cy d decelerate descent diffuseConstant direction
display divisor dominantBaseline dur dx dy edgeMode elevation enableBackground
end exponent externalResourcesRequired fill fillOpacity fillRule filter
filterRes filterUnits floodColor floodOpacity focusable fontFamily fontSize
fontSizeAdjust fontStretch fontStyle fontVariant fontWeight format from fx fy
g1 g2 glyphName glyphOrientationHorizontal glyphOrientationVertical glyphRef
gradientTransform gradientUnits hanging horizAdvX horizOriginX ideographic
imageRendering in in2 intercept k k1 k2 k3 k4 kernelMatrix kernelUnitLength
kerning keyPoints keySplines keyTimes lengthAdjust letterSpacing lightingColor
limitingConeAngle local markerEnd markerHeight markerMid markerStart
markerUnits markerWidth mask maskContentUnits maskUnits mathematical mode
numOctaves offset opacity operator order orient orientation origin overflow
overlinePosition overlineThickness paintOrder panose1 pathLength
patternContentUnits patternTransform patternUnits pointerEvents points
pointsAtX pointsAtY pointsAtZ preserveAlpha preserveAspectRatio primitiveUnits
r radius refX refY renderingIntent repeatCount repeatDur requiredExtensions
requiredFeatures restart result rotate rx ry scale seed shapeRendering slope
spacing specularConstant specularExponent speed spreadMethod startOffset
stdDeviation stemh stemv stitchTiles stopColor stopOpacity
strikethroughPosition strikethroughThickness string stroke strokeDasharray
strokeDashoffset strokeLinecap strokeLinejoin strokeMiterlimit strokeOpacity
strokeWidth surfaceScale systemLanguage tableValues targetX targetY textAnchor
textDecoration textLength textRendering to transform u1 u2 underlinePosition
underlineThickness unicode unicodeBidi unicodeRange unitsPerEm vAlphabetic
vHanging vIdeographic vMathematical values vectorEffect version vertAdvY
vertOriginX vertOriginY viewBox viewTarget visibility widths wordSpacing
writingMode x x1 x2 xChannelSelector xHeight xlinkActuate xlinkArcrole
xlinkHref xlinkRole xlinkShow xlinkTitle xlinkType xmlns xmlnsXlink xmlBase
xmlLang xmlSpace y y1 y2 yChannelSelector z zoomAndPan
```
你可能也使用自定义属性，只要他们是完全小写的。




















