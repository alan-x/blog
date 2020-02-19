[原文地址](https://reactjs.org/docs/events.html)
### SyntheticEvent

这个索引指南记录了`SyntheticEvent`包装器，它构成 React 的事件系统的一部分。查阅[处理事件]()指南去了解更多。

### 概述

你的事件处理器将接收到`SyntheticEvent`的实例，一个跨浏览器包裹着浏览器的原生事件，包括`stopPropagation()`和`preventDefault()`，除了事件在所有的浏览器工作一致。

如果你发现你因为某些原因需要底层浏览器事件，简单使用`nativeEvent`属性去获取它，每一个`SyntheticEvent`都有下列的属性：
```jsx harmony
boolean bubbles
boolean cancelable
DOMEventTarget currentTarget
boolean defaultPrevented
number eventPhase
boolean isTrusted
DOMEvent nativeEvent
void preventDefault()
boolean isDefaultPrevented()
void stopPropagation()
boolean isPropagationStopped()
DOMEventTarget target
number timeStamp
string type
```
注意：v0.14 之后，从事件处理器返回`false`不会再阻止事件冒泡。相反，`e.stopPropagation()`或者`e.preventDefault()`应该酌情手动触发。

### 事件池
`SyntheticEvent`是池的。这意味着`SyntheticEvent`对象将会被重用，并且所有的属性将在事件回调被调用之后置空。这是为了性能。比如，你不能异步传递一个事件：
```jsx harmony
function onClick(event) {
  console.log(event); // => nullified object.
  console.log(event.type); // => "click"
  const eventType = event.type; // => "click"

  setTimeout(function() {
    console.log(event.type); // => null
    console.log(eventType); // => "click"
  }, 0);

  // Won't work. this.state.clickEvent will only contain null values.
  this.setState({clickEvent: event});

  // You can still export event properties.
  this.setState({eventType: event.type});
}
```

注意：如果你想要在异步中访问事件属性，你应该在事件上调用`event.persist()`，它将会从池中移除事件，并允许事件被用户代码维护。

### 支持的事件

React 规范化事件，因此他们在不同浏览器之间保持一致的属性。

下面的事件处理器在冒泡阶段被事件触发。注册事件处理器到捕获阶段，添加`Capture`到事件名，比如，与其使用`onClick`，你应该使用`onClickCapture`在捕获阶段处理点击事件。

### 索引

### Clipboard Event

事件名：
```jsx harmony
onCopy onCut onPaste
```
属性：
```jsx harmony
DOMDataTransfer clipboardData
```

### Composition Events

事件名：
```jsx harmony
onCompositionEnd onCompositionStart onCompositionUpdate
```
属性：
```jsx harmony
string data
```

### Keyboard Events

事件名：
```jsx harmony
onKeyDown onKeyPress onKeyUp
```
属性：
```jsx harmony
boolean altKey
number charCode
boolean ctrlKey
boolean getModifierState(key)
string key
number keyCode
string locale
number location
boolean metaKey
boolean repeat
boolean shiftKey
number which
```
`key`属性可以接受任何记录在[DOM Level 3 Event spec](https://www.w3.org/TR/uievents-key/#named-key-attribute-values)上的值。

### Focus Events

事件名：
```jsx harmony
onFocus onBlur
```
属性：
```jsx harmony
DOMEventTarget relatedTarget
```

### Mouse Events

事件名：
```jsx harmony
onClick onContextMenu onDoubleClick onDrag onDragEnd onDragEnter onDragExit
onDragLeave onDragOver onDragStart onDrop onMouseDown onMouseEnter onMouseLeave
onMouseMove onMouseOut onMouseOver onMouseUp
```
`onMouseEnter`和`onMouseLeave`事件从被保留的元素传播到被进入的元素，而不是从普通的冒泡或者没有捕获阶段。
属性：
```jsx harmony
boolean altKey
number button
number buttons
number clientX
number clientY
boolean ctrlKey
boolean getModifierState(key)
boolean metaKey
number pageX
number pageY
DOMEventTarget relatedTarget
number screenX
number screenY
boolean shiftKey
```

### Pointer Events

事件名：
```jsx harmony
onPointerDown onPointerMove onPointerUp onPointerCancel onGotPointerCapture
onLostPointerCapture onPointerEnter onPointerLeave onPointerOver onPointerOut
```
`onPointerEnter`和`onPointerLeave`事件从被保留的元素传播到被进入的元素，而不是从普通的冒泡或者没有捕获阶段。

属性：
正如[W3 spec](https://www.w3.org/TR/pointerevents/)定义，pointer 事件继承[Mouse Event](https://reactjs.org/docs/events.html#mouse-events)，并扩展下列属性：
```jsx harmony
number pointerId
number width
number height
number pressure
number tangentialPressure
number tiltX
number tiltY
number twist
string pointerType
boolean isPrimary
```

一个关于跨浏览器支持的笔记：

pointer 事件并没有在所有的浏览器支持（）。React 特意不填充对其他浏览器的支持，因为一个标准兼容的填充将会极大增大`react-dom`的包大小。

如果你的应用需要 pointer 事件，我们推荐推荐第三方 pointer 事件填充。

### Selection Events

事件名：
```jsx harmony
onSelect
```


### Touch Events

事件名：
```jsx harmony
onTouchCancel onTouchEnd onTouchMove onTouchStart

```
属性：
```jsx harmony
boolean altKey
DOMTouchList changedTouches
boolean ctrlKey
boolean getModifierState(key)
boolean metaKey
boolean shiftKey
DOMTouchList targetTouches
DOMTouchList touches
```

### UI Events

事件名：
```jsx harmony
onScroll
```
属性：
```jsx harmony
number detail
DOMAbstractView view
```

### Wheel Events

事件名：
```jsx harmony
onWheel
```
属性：
```jsx harmony
number deltaMode
number deltaX
number deltaY
number deltaZ
```
### Media Events

事件名：
```jsx harmony
onAbort onCanPlay onCanPlayThrough onDurationChange onEmptied onEncrypted
onEnded onError onLoadedData onLoadedMetadata onLoadStart onPause onPlay
onPlaying onProgress onRateChange onSeeked onSeeking onStalled onSuspend
onTimeUpdate onVolumeChange onWaiting
```
### Image Events

事件名：
```jsx harmony
onLoad onError
```

### UI Events

事件名：
```jsx harmony
onScroll
```
属性：
```jsx harmony
number detail
DOMAbstractView view
```
### Animation Events

事件名：
```jsx harmony
onAnimationStart onAnimationEnd onAnimationIteration
```
属性：
```jsx harmony
string animationName
string pseudoElement
float elapsedTime
```

### Transition Events

事件名：
```jsx harmony
onTransitionEnd
```
属性：
```jsx harmony
string propertyName
string pseudoElement
float elapsedTime
```
### Transition Events

事件名：
```jsx harmony
onTransitionEnd
```
属性：
```jsx harmony
string propertyName
string pseudoElement
float elapsedTime
```
### Other Events

事件名：
```jsx harmony
onToggle
```
