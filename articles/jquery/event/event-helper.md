### 事件辅助
`jQuery`提供了一些事件相关的辅助函数，可以帮助你节约一些按键。这是`.hover()`函数的一个栗子。

### `.hover()`
`.hover()`方法让你可以传递1个或者2个函数在当`mouseenter`、`mouseleave`发生在一个元素的时候运行，如果你传递一个函数，它将会在两个事件都执行，如果你传递一个函数，第一个将在`mouseenter`的时候执行，第二个将会在`mouseleave`的时候执行。
注意：`jQuery` 1.4之前的版本，`.hover()`函数需要传递两个函数。
```
// The hover helper function
$( "#menu li" ).hover(function() {
    $( this ).toggleClass( "hover" );
});
```
你可以在[API站点]()找到更多的事件辅助函数。