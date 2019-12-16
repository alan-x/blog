### 优化选择器
选择器优化的重要性比以前要低许多，因为越来越多的浏览器实现了`document.querySelectAll()`，选择器的负载从`jQuery`移到了浏览器。然而，还是有一些提示要记住，当选择器成为性能瓶颈的时候。

### `jQuery`扩展
如果可能，尽量避免使用`jQuery`扩展的选择器。这些扩展不能很好的利用原生的`querySelectorAll()`方法的性能，而是需要使用`jQuery`提供的`Sizzle`选择器引擎。
```
// Slower (the zero-based :even selector is a jQuery extension)
$( "#my-table tr:even" );
 
// Better, though not exactly equivalent
$( "#my-table tr:nth-child(odd)" );
```
要记住很多`jQuery`的扩展，包括前面栗子的`:even`，没有在`CSS`规范中。在有些场景中，这些插件的遍历会加重性能损耗。
link Avoid Excessive Specificity
### 避免过分的指定
```
$( ".data table.attendees td.gonzalez" );
 
// Better: Drop the middle if possible.
$( ".data td.gonzalez" );
```
一个`扁平`的`DOM`同样也对提升选择器性能有帮助，选择器引擎将减少寻找元素遍历的层。

### 基于`ID`的选择器
使用`ID`选择器作为开始是一个很安全的方式。
```
// Fast:
$( "#container div.robotarm" );
 
// Super-fast:
$( "#container" ).find( "div.robotarm" );
```
第一种方式，`jQuery`使用`document.querySelectAll()`查询`DOM`。第二种，`jQuery`使用`document.getElementById()`，它更快，尽管后面的`.find`将会降低它的速度。
### 对于旧的浏览器的提示
当必须要支持类似`IE8`和之前的浏览器的时候，考虑下面的提示：
### 具体
右侧要具体，左侧可以稍微不那么具体。
```
// Unoptimized:
$( "div.data .gonzalez" );
 
// Optimized:
$( ".data td.gonzalez" );
```
如果可以，在你最右边的选择器使用`tag.class`，在左边最好只有`tag`或者`.class`。
### 避免通用选择器
指定或者暗示可以在任何地方找到的选择器可能会非常慢。
```
$( ".buttons > *" ); // Extremely expensive.
$( ".buttons" ).children(); // Much better.
 
$( ":radio" ); // Implied universal selection.
$( "*:radio" ); // Same thing, explicit now.
$( "input:radio" ); // Much better.
```