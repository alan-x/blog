### 事件介绍
### 介绍
网页都是和交互相关的。用户在页面上执行大量的操作，比如移动鼠标，点击元素，在输入框输入-这些都是事件的栗子。除了这些用户事件之外，还有一些其他的事件会发生，比如页面加载，视频播放或者暂停等。每当一些有趣的事件发生在页面上的时候，一个事件被激活，这意味着流浪起宣告一些东西发生了。这些宣告允许开发者去`监听`这些事件并作出适当的反应。



### 什么是`DOM`事件？
就像提到的，存在无数的事件类型，但是可能最容易理解的一种类型就是用户事件，比如当一个人点击一个元素或者向表单输入。这些类型的事件发生在元素上，意味着比如当用户点击一个按钮的时候，按钮上有一个事件出现了。尽管用户交互不是唯一的`DOM`事件类型，但是在开始的时候他们确实是最容易理解的。`MDN`有关于[可用的DOM事件]()很好的参考。

### 监听事件的方式
有很多种方式监听事件。操作不断发生在网页上，但是开发者之后在监听他们的时候才会注意到他们。监听一个事件基本上意味着你在等待浏览器告诉你一个特俗的事件发生了，而你指定页面如何反应。

为了指定浏览器在事件发生的时候做什么，你需要提供一个函数，一般成为事件处理器。当事件发生的时候，这个函数就会执行（或者直到事件发生才会执行）。

比如，当用户点击一个按钮的时候提示一个消息，你可能会这么写：

```
<button onclick="alert('Hello')">Say hello</button>
```
我们想要监听的事件通过`button`的`onclick`属性被定义，事件处理器就是`alert`函数，它会提示`Hello`给用户。尽管这是可行的，但是由于以下两个原因，这种实现方式很糟糕：

- 首先，我们混淆了我们的视图代码(`HTML`)和我们的行为代码(`JS`)。这意味着当我们需要更新功能的时候，我们必须去编辑我们的`HTML`，这是一个非常不好的实践和维护噩梦。
- 其实，不可扩展。如果你需要将这个功能绑定在非常多的按钮上，你不仅让引入了一堆重复的代码让页面膨胀，还摧毁了可维护性。
像这样内联事件处理器一般被称为`侵入性JavaScript`，于此相反，`非侵入性JavaScript`是讨论这个主题更普遍的方式。`非侵入性JavaScript`的概念是指你的`HTML`和`JS`bacon分离，从而带来更好的维护性。关注点分离是很重要的，因为它让代码块保持聚合（比如`HTML`、`CSS`、`JS`）并且不像代码块分离，有利于更新，增强。`非侵入性JavaScript`非常强调尽可能能添加最少的代码。如果用户电脑不支持`JavaScript`，则不应该让他们插入到页面编辑中。此外，为了防止命名冲突，`JS`代码应该为不同的功能和库使用单一的命名空间。`jQuery`就是一个很好的栗子，为了这个目的，`jQuery`对象/构造器（`jQiery`的别名`$`也一样）只占用了一个全局命名空间，所有`jQuery`的功能都被打包进一个对象。

为了非侵入性的完成这个任务，让我们稍微改变以下我们的`HTML`，删除`onClick`属性并用`id`替换，我们将在脚本文件中`勾住`按钮。

```
<button id="helloBtn">Say hello</button>
```
如果我们想要在用户点击按钮的时候被非侵入性的通知，我们可能需要在分离的脚本文件中像下面这么做：
```
// Event binding using addEventListener
var helloBtn = document.getElementById( "helloBtn" );
 
helloBtn.addEventListener( "click", function( event ) {
    alert( "Hello." );
}, false );
```
这里我们通过调用`getElementById`方法，将它的返回值赋给一个变量来保存按钮的引用。然后我们调用`addEventListener`方法，并提供一个当事件被触发的时候就会调用的事件处理器函数。这些代码没有任何的问题，并且将会在现代浏览器上工作的很好，除了`IE9`以前的版本。因为微软采用了一种不同的方法，`attachEvent`，并不是W3C标准的`addEventListener`，并且在`IE9`发布之前没有去改变它。因为这个原因，使用`jQuery`是非常有用的，因为`jQuery`抽象了浏览器的差异，允许开发者为这些任务类型使用单一的`API`，比如。
```
// Event binding using a convenience method
$( "#helloBtn" ).click(function( event ) {
    alert( "Hello." );
});
```
代码`$('#helloBtn')`使用`$`函数选择所有的按钮元素，并返回一个`jQuery`对象。这个`jQuery`包含大量的方法（函数），其中一个叫做`click`，它位于`jQuery`对象的原型上。我们可以在`jQuery`对象上调用`click`方法，并传递一个匿名函数事件处理器，当用户点击按钮的时候触发，它会提示用户`Hello.`。
使用`jQuery`还可以i 坚挺一大堆的事件：
```
// The many ways to bind events with jQuery
// Attach an event handler directly to the button using jQuery's
// shorthand `click` method.
$( "#helloBtn" ).click(function( event ) {
    alert( "Hello." );
});
 
// Attach an event handler directly to the button using jQuery's
// `bind` method, passing it an event string of `click`
$( "#helloBtn" ).bind( "click", function( event ) {
    alert( "Hello." );
});
 
// As of jQuery 1.7, attach an event handler directly to the button
// using jQuery's `on` method.
$( "#helloBtn" ).on( "click", function( event ) {
    alert( "Hello." );
});
 
// As of jQuery 1.7, attach an event handler to the `body` element that
// is listening for clicks, and will respond whenever *any* button is
// clicked on the page.
$( "body" ).on({
    click: function( event ) {
        alert( "Hello." );
    }
}, "button" );
 
// An alternative to the previous example, using slightly different syntax.
$( "body" ).on( "click", "button", function( event ) {
    alert( "Hello." );
});
```
从`jQuery 1.7`开始，所有的事件绑定都是通过`on`方法，无论你是直接调用或者使用昵称/简写方法，比如`bind`或者`click`，在内部都是使用`on`方法。记住这一点，使用`on`方法非常有好处，因为其他的所有方法都不过是语法糖，使用`on`方法更快，并且让代码更加一致。

注意前面栗子中的`on`，并讨论他们有什么不同。在第一个栗子中，`click`字符串作为`on`的第一个参数，第二个参数则是一个匿名函数。这个之前的`bind`很像。这里，我们直接向`#helloBtn`绑定了一个事件处理器。如果页面上还有其他的按钮，当他们点击的时候，将不会弹出`Hello.`，因为事件只绑定在`#helloBtn`上。


在第二个栗子中，我们传递了一个对象（由`{}`定义），有一个`click`属性，他的值是一个匿名函数。`on`的第二个参数是一个按钮的`jQuery`选择器表达式。栗子1-3的功能是相同的，栗子4不同在于`body`元素监听了发生在`button`上的`click`事件，不仅仅是`#helloBtn`。前面的最后一个栗子完全和前一个一摸一样，但是传递的是一个对象，我们传递一个事件字符串，一个选择器表达式字符串，一个回调。这些栗子都是事件代理的栗子，`Dom`树中较高的元素监听它的子元素的事件的一种处理方式。

### 事件代理
事件代理的工作原理就是事件冒泡。对于所有事件来说，无论何时，页面上发生的事件（比如元素被点击），事件从它诞生的元素，上升到它的父元素，父元素的父元素，等。一直到根元素，众所周知的`window`。所以在我们的`table`栗子中，无论那个`td`被点击了，他的父元素`tr`都会收到`click`的通知，`table`也会，`body`也会，直到`window`也会。尽管事件冒泡和事件代理工作的很好，但是代理元素（在我们的栗子中是`table`）离被代理的元素越近越好，这样事件就不需要在处理函数调用之前在`Dom`树中遍历太久。

事件代理比直接绑定到元素（或元素集）的两个有点是性能和前面提到的事件冒泡。想象一下，一个有1000个空间的`table`，要为每个空间绑定一个事件。这1000个分离的事件处理器浏览器都需要去绑定，尽管他们都是指向同一个函数。与其为每一个独立的空间都绑定事件，不如使用代理去监听发生在`table`父元素的事件，然后去做相应的响应。用一个事件代替1000个，带来更好的性能和内存管理。

事件冒泡的出现的副作用是我们可以通过`Ajax`添加更多的空间，不需要去直接为这些空间绑定事件，因为父元素`table`已经监听了由它的子元素触发的点击事件。如果我们不实用事件代理，我们不得不去为每一个添加的空间去绑定事件，这不仅仅是性能问题，还会成为一个维护地狱。

### 事件对象
在上面所有的栗子中，我们都是使用匿名的函数并指定一个`event`参数。让我们做一些小改变
```
// Binding a named function
function sayHello( event ) {
    alert( "Hello." );
}
 
$( "#helloBtn" ).on( "click", sayHello );
```
在这个稍微有点不同的栗子中，我们定义了一个函数称为`sayHello`，并且将它传递给`on`方法传，替代了匿名函数。很多在线的栗子都显示了匿名函数作为事件处理器，但是很重要的一点要意识到，那就是你也可以传递一个定义好的函数作为事件处理器。这对于如果很多元素或者很多事件要执行相同的功能很重要。这帮助你保持你的代码`DRY`。

但是`sayHello`函数的`event`参数是什么-他是什么？它能做什么？在所有的`DOM`事件回调中，`jQuery`传递了一个`event`对象当作参数，它包含事件的信息，比如精确的指出什么时候、哪里发生的，它是什么类型的事件，在那个元素发生的，还有很多其他的信息。当然你不必要把它叫做`event`，`e`或者其他你想要的都行，但是`event`是一个很好的很常见的命名。

如果元素特定的事件有默认的功能（比如`a`链接打开新的页面，表单中的按钮提交表单等），这个默认功能可以被取消。这在`Ajax`请求中很有用，如果一个用户点击按钮通过`Ajax`提交表单，我们需要去取消`button/form`的默认动作（提交到表单的`action`属性）,然后我们用一个`Ajax`请求完成同样的任务，这样体验更加无缝。为了做到这一点，我们使用`event`对象并调用他的`.preventDefault()`方法。我们同样可以使用`.stopPrpagation()`阻止事件冒泡到`DOM`树，这样父级元素将不会收到通知（如果事件代理被使用的话）。
```
// Preventing a default action from occurring and stopping the event bubbling
$( "form" ).on( "submit", function( event ) {
 
    // Prevent the form's default submission.
    event.preventDefault();
 
    // Prevent event from bubbling up DOM tree, prohibiting delegation
    event.stopPropagation();
 
    // Make an AJAX request to submit the form data
});
```
如果`.preventDefault()`和`.stopPropagation()`都要使用的话，你可以返回一个`false`来达到同样的效果，但建议两者都需要的时候才返回，而不是为了简洁。关于`.stopPropagation()`最后要注意的一点是当在代理事件中使用的时候，最快可以阻止事件冒泡的时机是当事件到达被代理的元素的时候。

还有一个很重要的点需要注意的是`event`对象包含一个`originalEvent`属性，这是浏览器自己创建的事件对象。`jQuery`包裹这个原生事件对象，并添加了许多有用的方法和属性，但是在某些情况下，你需要通过`event.originalEvent`去访问原生的事件对象。这对于移动设备和平板上的`touch`事件非常有用。

最后，为了知道`event`自身并查看所有它包含的数据，你应该使用`console.log`在浏览器控制台打印它。它允许你查看`event`所有的属性（包括`originalEvent`），这对于调试非常有帮助
```
// Logging an event's information
$( "form" ).on( "submit", function( event ) {
 
    // Prevent the form's default submission.
    event.preventDefault();
 
    // Log the event object for inspectin'
    console.log( event );
 
    // Make an AJAX request to submit the form data
});
```