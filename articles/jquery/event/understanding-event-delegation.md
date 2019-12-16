### 理解事件代理
### 介绍

事件代理允许我们绑定单个事件监听器到父元素，它将会被所有的子元素激活，不管子孙元素现在存在还是未来被添加。
link Example
### 栗子
在接下来的课程中，我们将引用下面的`HTML`
```
<html>
<body>
<div id="container">
    <ul id="list">
        <li><a href="http://domain1.com">Item #1</a></li>
        <li><a href="/local/path/1">Item #2</a></li>
        <li><a href="/local/path/2">Item #3</a></li>
        <li><a href="http://domain4.com">Item #4</a></li>
    </ul>
</div>
</body>
</html>
```
当`#list`组中的链接被点击的时候，我们希望在控制台打印出它的文字。正常情况下我们使用`.on()`可以直接绑定点击事件到每个链接元素。
```
// Attach a directly bound event handler
$( "#list a" ).on( "click", function( event ) {
    event.preventDefault();
    console.log( $( this ).text() );
});
```
尽管这工作的很好，但是有缺陷。思考当我们在绑定监听器之后添加一个新的链接会发生什么：
1
2
```
// Add a new element on to our existing list
$( "#list" ).append( "<li><a href='http://newdomain.com'>Item #5</a></li>" );
```
如果我们点击新添加的项目，任何事情都不会发生。因为我们在前面直接绑定了事件处理器。`.on()`方法只会在元素存在的时候为元素绑定事件。在这里栗子中，因为我们在调用`.on`的时候，新的链接并不存在，它没有得到事件处理器。

### 事件冒泡
了解事件冒泡是利用事件代理的一个重要因素。当我们的链接被点击的时候，一个点击事件将会在链接上触发，然后冒泡到`DOM`树，触发它每一个父元素的点击事件处理器：
- <a>
- <li>
- <ul #list>
- <div #container>
- <body>
- <html>
- document root
这意味着，任何时候你点击了我们其中一个链接，相当于点击了整个`document body`，这叫做事件冒泡或者事件传播。

由于我们知道了事件是怎样冒泡的，我们就可以创建事件代理：
```
// Attach a delegated event handler
$( "#list" ).on( "click", "a", function( event ) {
    event.preventDefault();
    console.log( $( this ).text() );
});
```
注意我们吧`a`从选择器表达式分离并作为第二个参数传递给`.on()`方法。第二个选择器表达式参数告诉处理器监听指定的事件，如果监听到了，检查触发的元素是否命中第二个参数。在这个栗子中，触发的事件就是我们的链接标签，正好命中了第二个参数。因为它命中了，所以匿名函数将会执行。现在我们绑定了一个单一的点击事件监听器去监听我们的`<ul>`，就可以监听到它的所有链接子元素的点击事件，而不需要为未知数量的链接直接绑定事件。

link Using the Triggering Elemen
### 使用触发元素
如果我们想要在链接是外部链接的时候在新窗口打开该怎么做（外部链接在这里表现为以`http`开头）？
```
// Attach a delegated event handler
$( "#list" ).on( "click", "a", function( event ) {
    var elem = $( this );
    if ( elem.is( "[href^='http']" ) ) {
        elem.attr( "target", "_blank" );
    }
});
```
这里我们给`.is()`方法传递了一个选择器表达式作为参数来判断这个元素的`href`属性是不是以`http`开头。我们也移除了`event.preventDefault()`语句，因为我们想要默认的行为发生（将会跳转到`href`）。
我们也可以通过修改`.on()`的选择器表达式参数简化我们的代码，完成我们的逻辑：
```
// Attach a delegated event handler with a more refined selector
$( "#list" ).on( "click", "a[href^='http']", function( event ) {
    $( this ).attr( "target", "_blank" );
});
```
### 总结
事件代理指的是通过使用事件传播(事件冒泡)在`DOM`中高级别元素中处理低级别事件的方式。这允许我们为现在存在的元素和未来添加的元素绑定单一的事件处理器。