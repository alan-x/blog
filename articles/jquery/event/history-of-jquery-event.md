### `jQuery`事件的历史
贯穿整个`jQuery`的发展，事件绑定的意义为了性能和语法等一系列原因发生了改变。从`就Query 1.7`开始，`.on()`方法接受直接数据绑定和创建事件代理。这篇文章旨在探索从`jQuery 1.7`到现在位置事件代理的历史以及每个版本如何使用他们。

给出如下`HTML`，我们的栗子是当我们点击每个`<li>`元素的时候，打印出该元素的文本。
```
<div id="container">
    <ul id="list">
        <li>Item #1</li>
        <li>Item #2</li>
        <li>Item #3</li>
        <li>...</li>
        <li>Item #100</li>
    </ul>
</div>​
```
### `.bind()`（废弃）
在`jQuery v1.0`出现
使用`.bind()`可以为每个元素绑定一个处理器。
```
​$( "#list li" ).bind( "click", function( event ) {
    var elem = $( event.target );
    console.log( elem.text() );
});​​​​​​​​​​​​​​​​​​​​​
```
就像在[事件代理]()文章讨论的，这不是最优的。
### `liveQuery`

`liveQuery`是一个非常流行的`jQuery`插件，允许创建的事件将会被现在存在的元素或者将来才存在的元素触发。这个插件不使用事件代理，而是耗费昂贵的`CPU`，每20ms查询一次`DOM`改变来触发响应的事件。

### `.bind()`代理（废弃）
在`jQuery v1.0`引入

通常我们不会将`.bind()`和事件代理绑定起来，但是在`jQuery v1.3`之前，这是我们可以使用代理唯一的方式。
```
​$( "#list" ).bind( "click", function( event ) {
    var elem = $( event.target );
    if ( elem.is( "li" ) ) {
        console.log( elem.text() );
    }
});​
```​​​​​​​​​​​​​​​​​​​​
这里我们可以利用事件冒泡的有力因素绑定点击事件到`<ul>`元素。当`<li>`被点击的时候，事件冒泡到它的父元素，`<ul>`将会执行我们的事件处理器。我们的事件处理器检测`event.taget`（触发事件的元素）是否命中我们的选择器表达式。
### `.live()`(废弃)
`jQuery v1.3`引入
All .live() event handlers are bound to the document root by default.
所有`.live()`事件处理器默认被绑定到`document root`
```
​$( "#list li" ).live( "click", function( event ) {
    var elem = $( this );
    console.log( elem.text() );
});​
```​​​​​​​​​​​​​​​​​​​​
当我们使用`.live()`的时候，我们的事件被绑定到`$(document)`。当`<li>`被点击，冒泡发生并且我们的点击事件在下面的元素中被激活：
- <ul>
- <div>
- <body>
- <html>
- document root
最后接受到点击事件的元素是`document`，这是我们`.live()`事件被绑定的元素。`.live()`将会检测我们的`#list li`选择器表达式是否是触发事件的元素，如果是，我们的事件处理器将会执行。

### `.live()`上下文（废弃）
`jQuery v1.4`引入
从`v1.0`开始，支持传入`context`作为`$()`方法的第二个可选参数。然而，在`$.live()`中使用这个`context`是在`v1.4`才加入的。

如果拿我们前面的`.live()`栗子并提供一个默认的`context`，它可能是这样的：
```
​$( "#list li", document ).live( "click", function( event ) {
    var elem = $( this );
    console.log( elem.text() );
});​
```​​​​​​​​​​​​​​​​​​​​
当我们使用`.live()`方法的时候，可以覆盖`context`，所以我们可以指定一个在`DOM`层级上离指定元素更近的元素作为`context`。
```
$( "li", "#list" ).live( "click", function( event ) {
    var elem = $( this );
    console.log( elem.text() );
});​​​​​​​​​​​​​​​​​​​​​
```
在这个实例中，当`<li>`被点击的时候，事件依旧会像之前一样的方式冒泡到`document`树。但是，我们的饿事件处理器是被绑定在`<ul>`父元素的，所以我们不需要等到事件冒泡到`document`跟元素。
### `.delegate()`（废弃）
在`jQiery v1.4.2`中第一次引入。
`.delegate()`方法在绑定事件代理处理器的上下文和事件冒泡的捕获的代理元素的选择器有非常大的差别。
```
$( "#list" ).delegate( "li", "click", function( event ) {
    var elem = $( this );
    console.log( elem.text() );
});​
```​​​​​​​​​​​​​​​​​​​​
### `.on()`
在`jQuery v1.7`中第一次引入

`.on()`方法给我们一个语义话的方式创建直接事件绑定和事件代理。它消除了使用废弃的`.bind()`、`.live()`、`.delegate()`方法的需要，提供了一个创建事件的简单方式。
```
$( "#list" ).on( "click", "li", function( event ) {
    var elem = $( this );
    console.log( elem.text() );
});​
```​​​​​​​​​​​​​​​​​​​​
###  总结
所有的事件代理方式在他们发布的时候都是创新的，并被视为最佳实践。根据你使用的`jQuery`版本，所择适合的事件代理。