### `CSS`、`样式`、`尺寸`
`jQuery`有很便捷的方式去设置和获取元素的`CSS`属性
```
// Getting CSS properties.
 
$( "h1" ).css( "fontSize" ); // Returns a string such as "19px".
 
$( "h1" ).css( "font-size" ); // Also works.
```
```
// Setting CSS properties.
 
$( "h1" ).css( "fontSize", "100px" ); // Setting an individual property.
 
// Setting multiple properties.
$( "h1" ).css({
    fontSize: "100px",
    color: "red"
});
```

注意第二行的参数分割-他是一个包含多个属性的对象。这是传递多个参数到函数的一个普通的方式，很多`jQuery`的`setting`方法一次性接受对象来设置值。


`CSS`属性通常宝航连字符，在`JavaScript`中，通常需要驼峰化。比如。`CSS`属性`font-size`在`JavaScript`中作为属性使用的饿时候，表示为`fontSize`。然而，这样把`CSS`属性名当作字符串传递为`.css()`的时候并不适用-在这种情况下，驼峰或者连字符形式的都可以。

在产品代码中，并不推荐把`.css()`当作一个`setter`，当时当传递一个对象设置`CSS`的时候，`CSS`属性将会将连字符转化成驼峰形式。

### 使用`CSS`类去设置样式
作为一个`getter`，`.css()`方法是有值的，然而，在生产环境中要避免把`.css()`当作一个`setter`。作为替代，将`CSS`规则写为描述一系列虚拟状态的类，然后在元素上改变类。
```
// Working with classes.
 
var h1 = $( "h1" );
 
h1.addClass( "big" );
h1.removeClass( "big" );
h1.toggleClass( "big" );
 
if ( h1.hasClass( "big" ) ) {
    ...
}
```
类用来存储元素的状态信息信息非常有用，比如指示出元素被选中了。
### 尺寸

`jQuery`提供了一系列的方法去设置和获取一个元素的尺寸和定位信息。
下面的方法简单显示了`jQuery`尺寸相关的功能。关于`jQuery`尺寸更多完整的细节，访问在[api.jquert.com 中关于尺寸]()完整文档。
```
// Basic dimensions methods.
 
// Sets the width of all <h1> elements.
$( "h1" ).width( "50px" );
 
// Gets the width of the first <h1> element.
$( "h1" ).width();
 
// Sets the height of all <h1> elements.
$( "h1" ).height( "50px" );
 
// Gets the height of the first <h1> element.
$( "h1" ).height();
 
 
// Returns an object containing position information for
// the first <h1> relative to its "offset (positioned) parent".
$( "h1" ).position();
```