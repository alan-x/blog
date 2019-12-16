使用样式去改变大量元素的`CSS`
如果你要使用`.css()`改变超过20个元素，考虑添加一个演示标签到页面，将会提升60%的速度
```
// Fine for up to 20 elements, slow after that:
$( "a.swedberg" ).css( "color", "#0769ad" );
 
// Much faster:
$( "<style type=\"text/css\">a.swedberg { color: #0769ad }</style>")
    .appendTo( "head" );
```