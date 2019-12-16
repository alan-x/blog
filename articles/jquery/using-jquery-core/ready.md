### $( document ).ready()

一个页面不能被安全的操作直到`document`是`ready`状态。`jQuery`为你检测了这种就绪状态。包含在`$(document).ready()`的代码将会在页面的`Document Object Model(Dom)`为`JavaScript`代码执行准备好的时候执行一次。

```
// A $( document ).ready() block.
$( document ).ready(function() {
    console.log( "ready!" );
});
```
有经验的开发者有时候会使用简短的`$()`来代替`$(document).ready()`。如果你写的代码将会被没有`jQuery`经验的人看到，最好使用完整的格式。
```
// Shorthand for $( document ).ready()
$(function() {
    console.log( "ready!" );
});
```
你也可以传递一个具名函数给`$(document).ready()`来代替传递匿名函数。
```
// Passing a named function instead of an anonymous function.
 
function readyFn( jQuery ) {
    // Code to run when the document is ready.
}
 
$( document ).ready( readyFn );
// or:
$( window ).on( "load", readyFn );
```

下面这个栗子显示了`$(document).ready()`和`$(window).on("load")`的行为。它尝试通过`iframe`加载一个站点`URL`，并检测这两个事件：
```
<html>
<head>
    <script src="https://code.jquery.com/jquery-1.9.1.min.js"></script>
    <script>
    $( document ).ready(function() {
        console.log( "document loaded" );
    });
 
    $( window ).on( "load", function() {
        console.log( "window loaded" );
    });
    </script>
</head>
<body>
    <iframe src="http://techcrunch.com"></iframe>
</body>
</html>
```