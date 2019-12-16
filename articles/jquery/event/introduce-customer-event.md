### 自定义事件

我们对基本的事件已经非常熟悉了-`click`、`mouseover`、`focue`、`blue`、`submit`等等-所以我们可以在用户和浏览器交互的时候插手。自定义事件为事件驱动编程打开了新世界。

当内建事件看起来非常符合你的需要的时候，第一次很难理解为什么要使用自定义事件。事实证明自定义事件为`JavaScript`事件驱动编程提供了一个新的思路。不是将焦点放在触发动作的元素上，而是聚焦于被触发的元素上。这带来许多好处，包括：

- 目标元素的行为可以被多个不同元素使用相同代码触发。
- 行为可以一次性跨越多种类似的目标元素触发
- 行为和元素的代码关联上更加清晰，让代码更加易读和维护。
为什么你需要关心？一个栗子可能可以作为最好的解释。假设你有房子的房间里有一个电灯。这个电灯现在是开的，它被两个三路开关和一个拍板控制
```
<div class="room" id="kitchen">
    <div class="lightbulb on"></div>
    <div class="switch"></div>
    <div class="switch"></div>
    <div class="clapper"></div>
</div>
```
触发拍板和开关将会改变电灯的状态。开关不关心电灯当前的状态，他们只是改变状态。
Without custom events, you might write some code like this:
没有自定义事件，你可能会像这样写代码：
```
$( ".switch, .clapper" ).click(function() {
    var light = $( this ).closest( ".room" ).find( ".lightbulb" );
    if ( light.is( ".on" ) ) {
        light.removeClass( "on" ).addClass( "off" );
    } else {
        light.removeClass( "off" ).addClass( "on" );
    }
});
```
使用自定义事件，你的代码可能会像这样：
```
$( ".lightbulb" ).on( "light:toggle", function( event ) {
    var light = $( this );
    if ( light.is( ".on" ) ) {
        light.removeClass( "on" ).addClass( "off" );
    } else {
        light.removeClass( "off" ).addClass( "on" );
    }
});
 
$( ".switch, .clapper" ).click(function() {
    var room = $( this ).closest( ".room" );
    room.find( ".lightbulb" ).trigger( "light:toggle" );
});
```
最后一点代码不是令人激动的，但是一些重要的事情发生了：我们从开关和拍板上将电灯的行为移动到了电灯上。

让我们的栗子更加有趣一点，为我们的房子添加更多的房间。还有一个主控，就像下面那样：
```
<div class="room" id="kitchen">
    <div class="lightbulb on"></div>
    <div class="switch"></div>
    <div class="switch"></div>
    <div class="clapper"></div>
</div>
<div class="room" id="bedroom">
    <div class="lightbulb on"></div>
    <div class="switch"></div>
    <div class="switch"></div>
    <div class="clapper"></div>
</div>
<div id="master_switch"></div>
```
如果房间内有等是开的，我们希望主开关关闭所有的电灯；否则，我们希望开启所有的电灯。为了完成这个功能，我们为电灯添加两个自定义事件：`light:on`和`light:off`，我们将会在`light:toggle`自定义事件中水用他们，并使用一些逻辑决定主开关触发哪个：
```
$( ".lightbulb" ).on( "light:toggle", function( event ) {
    var light = $( this );
    if ( light.is( ".on" ) ) {
        light.trigger( "light:off" );
    } else {
        light.trigger( "light:on" );
    }
}).on( "light:on", function( event ) {
    $( this ).removeClass( "off" ).addClass( "on" );
}).on( "light:off", function( event ) {
    $( this ).removeClass( "on" ).addClass( "off" );
});
```
 
$( ".switch, .clapper" ).click(function() {
    var room = $( this ).closest( ".room" );
    room.find( ".lightbulb" ).trigger( "light:toggle" );
});
 
$( "#master_switch" ).click(function() {
    var lightbulbs = $( ".lightbulb" );
 
    // Check if any lightbulbs are on
    if ( lightbulbs.is( ".on" ) ) {
        lightbulbs.trigger( "light:off" );
    } else {
        lightbulbs.trigger( "light:on" );
    }
});
注意主控开关的行为是如何绑定到主控开关的；电灯的行为属于电灯。
### 自定义事件命名
你可以为自定义事件使用任何名字，但是您要注意新的事件名可能会被未来`DOM`事件使用。因为这个原因，这篇文章中，我们选择使用`light:`作为我们所有事件的名字，带冒号的事件在未来的`DOM`标准不太可能被使用。

### 概括：`.on()`和`.trigger()`
在自定义事件的事件，有两个重要的`jQuery`方法：`.on()`和`.trigger()`。在事件章节，我们知道了如何使用这些方法处理用户事件；在这个章节，要记住两个重要的东西：

- `.on()`方法需要一个事件类型和一个事件处理函数作为参数。可选的，也可以接受一个事件相关的饿数据作为第二个参数，将事件处理函数作为第三个参数。任何传递进去的数据可以在事件处理函数的事件对象的`data`属性访问到。事件处理函数总是接受一个事件对象作为它的第一个参数

- `trigger()`方法接受一个事件类型作为参数。可选的，他也可以接受一个值数组。这些值将会被事件处理函数作为参数，并放置到事件对象之后。

这是一个`.on()`、`trigger()`中使用自定义数据的栗子：

```
$( document ).on( "myCustomEvent", {
    foo: "bar"
}, function( event, arg1, arg2 ) {
    console.log( event.data.foo ); // "bar"
    console.log( arg1 );           // "bim"
    console.log( arg2 );           // "baz"
});
 
$( document ).trigger( "myCustomEvent", [ "bim", "baz" ] );
```
### 结论
自定义事件提供了一个思考你代码的新方式：他们强调目标的行为，而不是触发它的元素。如果你花一些事件来描述你应用的各个部分以及每个部分需要展示的行为，自定义事件会为你提供一个强大的方式让你和这些部分沟通，一次一个或者一个集体。一旦这些部分的行为被描述，它从任何地方触发合格行为都将变得非常的简单。可以快速的创建体现和交互选项。最后，自定义事件清晰的关联元素和它的行为，会增强代码的可读性和可维护性。