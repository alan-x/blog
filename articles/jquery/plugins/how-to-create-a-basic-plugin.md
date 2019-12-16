### 这样创建一个基础的插件
有时候你想要让你的一部分功能贯穿你的代码。比如，可能你想要一个方法，这个方法可以在`jQuery`选择器上调用，它执行一些列的操作。这种情况下。你可能想要写一个插件。

### `jQuery`是怎么工作的 101：`jQuery`对象方法
在编写我们的插件之后钱，我们必须要了解一点关于`jQuery`的工作原理。看一下这个代码：
```
$( "a" ).css( "color", "red" );
```
这是一些非常基本的`jQuery`代码，但是你直到这背后发生了什么吗？当你使用`$`函数选择元素的时候，它返回一个`jQuery`对象。这个对象包含所有你使用过的方法（`.css()`，`.click()`之类的），还有所有满足你选择器表达式的元素。`jQuery`对象从`$.fn`对象上获取这些方法。这个对象包含所有的的`jQuery`对象的方法，如果我们想要写我们自己的方法，它还需要包含这些方法。

### 基本的插件编写
让我们编写一个插件，它可以让捕获的元素内部的文本编程绿色。所有我们要做的就是添加一个叫做`grennify`的函数到`$.fn`，它将会像其他`jQuery`对象方法一样可用。

```
$.fn.greenify = function() {
    this.css( "color", "green" );
};
 
$( "a" ).greenify(); // Makes all the links green.
```
注意使用`.css()`，其他方法，我们使用`this`，不是`$(this)`。这是因为我们的`greenify`函数是和`.css()`属于一个对象。

### 链式
这可以共奏 但是想要让我们的插件在真实世界存活，还需要做两件事。`jQuery`的特性之一是链式，当你链接五到六个动作到一个选择器的时候。这通过让所有的`jQuery`对象方法再一次返回原始的`jQuery`对象来完成（有一些例外：`.width()`没有参数的时候返回选择元素的宽度，它不是可链式调用的）。让我们的插件方法支持链式只需要添加一行代码：
```
$.fn.greenify = function() {
    this.css( "color", "green" );
    return this;
}
 
$( "a" ).greenify().addClass( "greenified" );
```
### 保护`$`别名，添加作用域。
`$`变量在`JavaScript`库非常的流行，如果你使用其他库和`jQuery`，你需要使用`jQuery.noConflict()`让`jQuery`不使用`$`。然而这将会打破我们的插件，因为它的编写建立在`$`是`jQuery`函数的别名的假设智商。为了和其他插件一起工作，并继续使用`jQuery`的`$`别名，我们需要将我们的所有代码放置到[立即执行函数表达式]()内，然后场地给函数`jQuery`，并把参数名叫做`$`：
```
(function ( $ ) {
 
    $.fn.greenify = function() {
        this.css( "color", "green" );
        return this;
    };
 
}( jQuery ));
```
此外，立即执行函数的主要目的是允许我们拥有自己的私有变量。比如我们想要一个绿色，我们想要将它存储在变量中：
```
(function ( $ ) {
 
    var shade = "#556b2f";
 
    $.fn.greenify = function() {
        this.css( "color", shade );
        return this;
    };
 
}( jQuery ));
```
### 最小化插件痕迹
编写插件只占用`$.fn`一个插槽是一个非常好的实践。这可以减少你的插件被覆盖的风险，和你的插件覆盖别人的插件的风险。换句话说，这非常坏：

```
(function( $ ) {
 
    $.fn.openPopup = function() {
        // Open popup code.
    };
 
    $.fn.closePopup = function() {
        // Close popup code.
    };
 
}( jQuery ));
```
有一个插件会更好，并使用参数来控制插槽表现的行为。
```
(function( $ ) {
 
    $.fn.popup = function( action ) {
 
        if ( action === "open") {
            // Open popup code.
        }
 
        if ( action === "close" ) {
            // Close popup code.
        }
 
    };
 
}( jQuery ));
```
### 使用`each()`方法
通常情况下，你的`jQuery`对象会包含大量的`DOM`元素引用，这就是为什么`jQuery`对应通常被称为集合。如果你想要使用指定元素做一些操作（比如获取数据属性，计算特定位置），你需要使用`.each()`遍历元素。
```
$.fn.myNewPlugin = function() {
 
    return this.each(function() {
        // Do something to each element here.
    });
 
};
```
注意我们返回了`.each()`的结果，而不是`this`。因为`.each()`已经是可链式调用的了，它返回`this`，我们返回它。这是一个委会链式能力很好的方式，我们一直都是这么做的。

### 接收选项
当你的插件越来越浮渣，让你的插件根据接收到的配置来自定义是一个很好的想法。特别是我们有很多配置的时候，这么做最简单的方式就是接收一个对象子棉量。让我们该拜年我们的`greenify`插件去接收一些配置。
```
(function ( $ ) {
 
    $.fn.greenify = function( options ) {
 
        // This is the easiest way to have default options.
        var settings = $.extend({
            // These are the defaults.
            color: "#556b2f",
            backgroundColor: "white"
        }, options );
 
        // Greenify the collection based on the settings variable.
        return this.css({
            color: settings.color,
            backgroundColor: settings.backgroundColor
        });
 
    };
 
}( jQuery ));
```
使用栗子：
```
$( "div" ).greenify({
    color: "orange"
});
```
颜色的默认是`#556b2f`被`$.extend()`的`orange`覆盖。
### 将它放在一起
这是一个简单的插件栗子，综合了我们讨论的所有技术：

```
(function( $ ) {
 
    $.fn.showLinkLocation = function() {
 
        this.filter( "a" ).each(function() {
            var link = $( this );
            link.append( " (" + link.attr( "href" ) + ")" );
        });
 
        return this;
 
    };
 
}( jQuery ));
 
// Usage example:
$( "a" ).showLinkLocation();
```
这个方便的插件遍历集合内所有的锚点将他们的`href`属性拼接到括号内。
```
<!-- Before plugin is called: -->
<a href="page.html">Foo</a>
 
<!-- After plugin is called: -->
<a href="page.html">Foo (page.html)</a>
```
我们的拆那可以优化：
```
(function( $ ) {
 
    $.fn.showLinkLocation = function() {
 
        this.filter( "a" ).append(function() {
            return " (" + this.href + ")";
        });
 
        return this;
 
    };
 
}( jQuery ));
```
当使用`.append()`接收回调的方法的时候，回调的返回值将会决定什么被添加到记恨中的每个元素。注意我们没有使用`.attr()`方法去捕获`href`属性，因为原生的`DOM API`提供了使用适当命名的`hef`属性快速访问。