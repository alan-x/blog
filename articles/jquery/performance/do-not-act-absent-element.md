### 不要操作不存在的元素
如果你在一个空的选择器上执行一系列的代码，`jQuery`也不会告诉你的-它不会报错。需要你自己去验证选择器是否包含一些元素。
```
// Bad: This runs three functions before it
// realizes there's nothing in the selection
$( "#nosuchthing" ).slideUp();
 
// Better:
var elem = $( "#nosuchthing" );
 
if ( elem.length ) {
 
    elem.slideUp();
 
}
 
// Best: Add a doOnce plugin.
jQuery.fn.doOnce = function( func ) {
 
    this.length && func.apply( this );
 
    return this;
 
}
 
$( "li.cartitems" ).doOnce(function() { 
 
    // make it ajax! \o/ 
 
});
```
这个直营对`jQuery UI`组件非常有用，当选择器不包含元素的时候，他们也有很大的开销。