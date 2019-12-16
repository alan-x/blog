### jQuery 和 jQuery.fn
`jQuery`定义在第134～140行，内容如下：
```
	// Define a local copy of jQuery
	jQuery = function( selector, context ) {

		// The jQuery object is actually just the init constructor 'enhanced'
		// Need init if jQuery is called (just allow error to be thrown if not included)
		return new jQuery.fn.init( selector, context );
	},
```
这里其实就是我们最常用的`jQuery`对象，配合第10357行`window.jQuery = window.$ = jQuery;`，就可以直接使用`$('#selector')`这种形式了。

可以看到，`jQuery`方法内返回的是`new jQuery.fn.init( selector, context )`，这是啥？

`jQuery.fn`定义在第146行，代码省略部分如下：
```
jQuery.fn = jQuery.prototype = {

	// The current version of jQuery being used
	jquery: version,

	constructor: jQuery,

	// The default length of a jQuery object is 0
	length: 0,

    toArray: function(){...}

    map: function(){...}

    each: function(){...}

    ...
}
```
其实`jQuery.fn`只是一个对象，但是`jQuery.fn = jQuery.prototype = {...}`让它变成了`jQuery`的原型，所以`jQuery.fn`上的所有方法`jQuery`对象都能调用。

`jQuery.fn.init`在第2903~3011行定义，代码省略如下：
```
var rootjQuery,

	// A simple way to check for HTML strings
	// Prioritize #id over <tag> to avoid XSS via location.hash (#9521)
	// Strict HTML recognition (#11290: must start with <)
	// Shortcut simple #id case for speed
	rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]+))$/,

	init = jQuery.fn.init = function( selector, context, root ) {
		...
	};

// Give the init function the jQuery prototype for later instantiation
init.prototype = jQuery.fn;
```
`jQuery.fn.init`只是`fn`上的一个函数属性，但是`init.prototype = jQuery.fn`让`jQuery.fn`变成了`init`的原型。所以`jQuery.fn`上的方法`init`都可以调用，结合`jQuery.fn = jQuery.prototype = {...}`其实就可以知道`init.prototype = jQuery.fn = jQuery.prototype = {...}`，那么`$()`就是`new jQuery.fn.init( selector, context )`就是`new init( selector, context )`，而`init`方法的原型是`jQuery.fn`，也是`jQuery.prototype`，因此让`$()`返回的`jQuery`对象拥有众多的方法，而我们加在`jQuery.fn`上的方法也会让所有的`jQuery`对象共享。

这里其实不复杂，如果用多个变量来表示就没有那么多弯弯绕绕了，只是这里偏偏把所有的都关联起来了，比如我们可以这样：
```
jQuery.prototype={ }

fn = jQuery.prototype

init.prototype = fn
// 也可以这么表示
init.prototype = jQuery.prototype
```
`jQuery.fn`就是`jQuery.prototype`的一个引用，而`init`也不过是`jQuery`的一个别名

在后面的代码中，我们最常见的就是如下形式：
```
jQuery.fn.extend({...})
```
`extend`方法是一个属性拷贝方法，用来将对象的属性拷贝到当前对象上，这里就是将对象的属性拷贝到`jQuery.fn`上，相当于扩展了`jQuery`的原型方法，这样所有的`jQuery`对象都可以享受到这些方法。

我们也可以看到如下形式：
```
jQuery.extend({...})
```
作用和上面一样，但是为什么有些方法是扩展到`jQuery`上，有些是扩展到`jQuery.fn`上呢？

在我们使用`$()`方法之后会返回一个`jQuery`对象，而上面的一系列方法都来自于`jQuery.prototype`，也就是`jQuery.fn`，我们可以通过`$().each()`之类来直接调用。
而`jQuery`上的方法则只能通过`jQuery.each（）`来调用。

看第146～225行的`jQuery.fn`的声明（省略部分）：
```
jQuery.fn = jQuery.prototype = {
	...
	each: function( callback ) {
		return jQuery.each( this, callback );
	},
	...
}
```
这里声明的一个`each`方法，而内部却调用了`jQuery.each`方法，再看第296~471行的`jQuery.extend()`中（省略部分）：
```
jQuery.extend( {
	each: function( obj, callback ) {
		...
	},
} )
```
可以看到，`jQuery.fn.each`只需要一个参数，而`jQuery.each`则需要两个参数，`jQuery.fn.each`的使用方式是`$('.className').each(()=>...)`。而`jQuery.each`的使用场景则是`jQuery.each([], ()=>...)`。`jQuery.each`方法的功能是遍历`obj`并在每个项上调用`callback`，而`jQuery.fn.each`将`this`作为`obj`参数传输，调用`jQuery.each`，这样既保持了代码简约，又方便调用，贼棒。