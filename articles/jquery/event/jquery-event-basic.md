### `jQuery`事件基础
### `jQuery`事件基础
### 设置`DOM`元素的事件响应
`jQuery`设置页面元素的事件驱动相应很简单。这些页面上的事件事件经常被终端用户触发，比如文字被输入到一个表单元素或者鼠标指向移动。在一些场景下，比如页面加载或者卸载事件，浏览器自己将会触发。

`jQuery`为大部分的原生浏览器事件提供了简便的方法。这些方法-包括`.click()`、`.focus()`、`.blur()`、`.change()`等-是`jQuery`方法`.on()`的简写。当你想要为事件处理器提供数据，当你使用自定义事件，或者当你想要传递多个事件和处理的对象时，`.on()`方法对于绑定相同处理器到多个事
件是非常有用的。
```
// Event setup using a convenience method
$( "p" ).click(function() {
    console.log( "You clicked a paragraph!" );
});
```
```
// Equivalent event setup using the `.on()` method
$( "p" ).on( "click", function() {
    console.log( "click" );
});
```
### 拓展事件到新的页面元素
要着重注意的是，`.on()`方法只能为在你设置好监听器时候已经存在的元素设置事件监听器。在事件监听器创建之后的相同的元素将不会自动拥有你之间设置的行为。比如

```
$( document ).ready(function(){
 
    // Sets up click behavior on all button elements with the alert class
    // that exist in the DOM when the instruction was executed
    $( "button.alert" ).on( "click", function() {
        console.log( "A button with the alert class was clicked!" );
    });
 
    // Now create a new button element with the alert class. This button
    // was created after the click listeners were applied above, so it
    // will not have the same click behavior as its peers
    $( "<button class='alert'>Alert!</button>" ).appendTo( document.body );
});
```
查阅关于事件代理的文章，了解如何使用`.on()`让事件行为可以扩展到新建的元素，而不到重新绑定。
### 事件处理器函数内部
每一个事件处理函数接受一个事件对象，宝航了许多的属性和方法。这个事件对象一般通过`.preventDefault()`方法用来阻止事件的默认动作。不过，这个事件对象还包含了许多其他有用的属性和方法，包括：

- `pageX`，`pageY`
事件发生的时候的鼠标位置，相对于页面的左上角（不是整个浏览器窗口）。
- `type`
事件的类型（比如：`click`）

- `which`
被按下的按钮或者键

- `data`
绑定事件的时候传入的数据，比如：
```
// Event setup using the `.on()` method with data
$( "input" ).on(
    "change",
    { foo: "bar" }, // Associate data with event binding
    function( eventObject ) {
        console.log("An input value has changed! ", eventObject.data.foo);
    }
);
```
- `target`
初始化该事件的`DOM处理元素`

- `namespace`
事件触发时候指定的命名空间

- `timeStamp`
从1970年1月1日到事件发生的时候的毫秒数。

- `preventDefault()`
阻止事件的默认行为（比如：链接跳转）
link stopPropagation()
- `stopPropagation()`
阻止事件冒泡到其他元素

除了事件对象，事件处理函数可以通过事件处理函数内部的关键字`this`访问`DOM`元素的。要将`DOM`元素转化为`jQuery`对象可以使用`jQuery`方法，可以使用`$(this)`，一般形式如下：

```
var element = $( this );
```
一个完整的栗子如下：
```
// Preventing a link from being followed
$( "a" ).click(function( eventObject ) {
    var elem = $( this );
    if ( elem.attr( "href" ).match( /evil/ ) ) {
        eventObject.preventDefault();
        elem.addClass( "evil" );
    }
});
```
### 设置多个事件响应
在你的应用中，一个元素几个女孩仓绑定多个事件，如果多个事件分享同一个事件处理函数，你可以为`.on()`方法提供一个用空格隔开的事件列表：
```
// Multiple events, same handler
$( "input" ).on(
    "click change", // Bind handlers for multiple events
    function() {
        console.log( "An input was clicked or changed!" );
    }
);
```
当每一个事件有它自己的处理器，你可以给`.on()`传递一个对象，这个对象包含多个键值对，键是事件的名称，值是事件处理函数。
```
// Binding multiple events with different handlers
$( "p" ).on({
    "click": function() { console.log( "clicked!" ); },
    "mouseover": function() { console.log( "hovered!" ); }
});
```
link Namespacing Events
### 事件命名空间
为了复杂的应用程序，也为了你能分享插件给别人，可以为你的事件指定命名空间，这样可以避免无意间断开你没有或者不知道的事件。
```
// Namespacing events
$( "p" ).on( "click.myNamespace", function() { /* ... */ } );
$( "p" ).off( "click.myNamespace" );
$( "p" ).off( ".myNamespace" ); // Unbind all events in the namespace
```
### 卸载事件监听器
为了移除事件监听器，你可以使用`.off()`方法，并传递事件类型。如果你绑定了一个具名函数给事件，可以通过将这个函数作为第二个参数传递给`.off()`来单独移除这个具名函数。
```
// Tearing down all click handlers on a selection
$( "p" ).off( "click" );
```
```
// Tearing down a particular click handler, using a reference to the function
var foo = function() { console.log( "foo" ); };
var bar = function() { console.log( "bar" ); };
 
$( "p" ).on( "click", foo ).on( "click", bar );
$( "p" ).off( "click", bar ); // foo is still bound to the click event
```
### 设置一次性事件
有时候你需要一个特殊的只运行一次的处理器-之后你希望没有处理器运行，或者你希望一个不同的处理器去处理。`jQuery`提供了`.one()`方法来打到这个目的。

```
// Switching handlers using the `.one()` method
$( "p" ).one( "click", firstClick );
 
function firstClick() {
    console.log( "You just clicked this for the first time!" );
 
    // Now set up the new handler for subsequent clicks;
    // omit this step if no further click responses are needed
    $( this ).click( function() { console.log( "You have clicked this before!" ); } );
}
```
注意上面的代码片段，`firstClick`函数将会在每个段落元素第一次被点击的时候执行，而不是第一次点击任意一个段落元素将所有元素的的函数删除。
`.one()`也可以用来绑定多个事件：
6
```
// Using .one() to bind several events
$( "input[id]" ).one( "focus mouseover keydown", firstEvent);
 
function firstEvent( eventObject ) {
    console.log( "A " + eventObject.type + " event occurred for the first time on the input with id " + this.id );
}
```
在这种场景中，`firstEvent`函数将会为每个事件执行一次。在上面的代码段中，这意味着一旦`input`元素获得焦点，处理器函数将会继续执行第一个`keydown`事件。