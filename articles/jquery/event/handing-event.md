### 事件处理
`jQuery`提供了`.on()`方法去响应发生在选中元素上的事件。这叫做事件绑定。尽管`.on()`不是提供事件绑定的唯一方法，但是这在`jQuery 1.7+`上是最好的实践。想知道更多，可以阅读[jQuery事件绑定的演变]()。
`.on()`方法提供了一系列有用的特性
- [将选择元素上触发的所有事件绑定到一个事件处理器上]()。
- [将多个事件绑定到一个事件处理器上]()。
- [将多个事件和多个处理器绑定到所选元素上]()。
- [在事件处理器中了解事件的详情]()。
- [在自定义事件中传递数据到事件处理器]()。
- [为未来出现的元素绑定事件]()。
### 栗子
### 简单的事件绑定
```
// When any <p> tag is clicked, we expect to see '<p> was clicked' in the console.
$( "p" ).on( "click", function() {
    console.log( "<p> was clicked" );
});
```
### 绑定多个事件，但是只有一个是事件处理器。
假设你想要在鼠标移出或者移入选中元素的时候触发相同的事件。最好的方法是使用`mouseenter`和`mouseleave`。注意这个和下一个栗子的不同。
```
// When a user focuses on or changes any input element,
// we expect a console message bind to multiple events
$( "div" ).on( "mouseenter mouseleave", function() {
    console.log( "mouse hovered over or left a div" );
});
```
### 多个事件和多个处理器
假设你想要为鼠标进入和鼠标移出元素提供不同的事件处理器。这是相对于之前更普通的栗子。比如，如果你想要提供一个在`hover`的时候隐藏和显示的提示栏，你可以这么使用。

.on() accepts an object containing multiple events and handlers.
`.on()`接受一个对象包含多个事件和事件处理器。
```
$( "div" ).on({
    mouseenter: function() {
        console.log( "hovered over a div" );
    },
    mouseleave: function() {
        console.log( "mouse left a div" );
    },
    click: function() {
        console.log( "clicked on a div" );
    }
});
```
### 事件对象
处理事件是非常棘手的。使用传递给事件处理器的事件对象中包含的额外信息可以达到更好的控制。为了变得对事件对象更加了解，使用下面的代码在你的浏览器点击页面上的`<div>`之后观察控制台。打一个断点，看看[事件处理函数里面有什么]()
```
$( "div" ).on( "click", function( event ) {
    console.log( "event object:" );
    console.dir( event );
});
```
### 传递数据到事件处理器
你可以传递自己的数据到事件对象。
```
$( "p" ).on( "click", {
    foo: "bar"
}, function( event ) {
    console.log( "event data: " + event.data.foo + " (should be 'bar')" );
});
```
### 绑定事件到还不存在的元素
这叫做事件代理。下面是一个完整的栗子，可以查阅[事件代理]()了解详细的解释。
```
$( "ul" ).on( "click", "li", function() {
    console.log( "Something in a <ul> was clicked, and we detected that it was an <li> element." );
});
```
### 连接事件并只执行一次
有时候你需要一个特殊的只运行一次的处理器-之后，你希望没有处理器执行，或者你可能想要一个不同的处理器去执行。`jQuery`为了这个目的提供了`.one()`方法。
```
// Switching handlers using the `.one()` method
$( "p" ).one( "click", function() {
    console.log( "You just clicked this for the first time!" );
    $( this ).click(function() {
        console.log( "You have clicked this before!" );
    });
});
```
`.one()`方法在你需要在元素第一次被点击的时候做一些复杂的设置的时候特别有用，但是之后它不再执行。
`.one()`和`.on()`接受一样的参数，这意味着它加收多个事件和一个或者多个处理器，传递自定义数据和事件代理
### 断开事件连接
尽管`jQuery`有趣的地方都发生在`.on()`方法上，但是如果你是一个负责人的开发者，它的同行也十分的重要。`.off()`在你不需要事件绑定的时候可以清理掉它。复杂的用户界面和大量的事件绑定将会降低浏览器性能，仔细的使用`.off()`方法是保证你只在你需要他们的时候才拥有你需要的事件绑定最好的实践。
```
// Unbinding all click handlers on a selection
$( "p" ).off( "click" );
```
```
// Unbinding a particular click handler, using a reference to the function
var foo = function() {
    console.log( "foo" );
};
 
var bar = function() {
    console.log( "bar" );
};
 
$( "p" ).on( "click", foo ).on( "click", bar );
 
// foo will stay bound to the click event
$( "p" ).off( "click", bar );
```
### 事件命名空间
为了复杂的应用和你要分享给别人的插件，为你的事件创建命名空间是非常有用的，这样你就不会在无意间断开你没有或者你不知道的事件。了解更多详情，查看[事件命名空间]()。