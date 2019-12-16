### 入口
我们通常使用`$()`来选择元素，`$()`虽然只有一个方法，根据参数的不同却有多个用法。
该方法的完整签名位于第135~140行：
```
	jQuery = function( selector, context ) {

		// The jQuery object is actually just the init constructor 'enhanced'
		// Need init if jQuery is called (just allow error to be thrown if not included)
		return new jQuery.fn.init( selector, context );
	},
```
该方法接受两个参数：
- `selector`：选择器表达式
- `context`：上下文

在调用这个方法的时候，内部执行的是`return new jQuery.fn.init( selector, context )`，所以实际执行的是`jQuery.fn.init`，位于第2911～3088，以下是完整代码：
```
init = jQuery.fn.init = function( selector, context, root ) {
		var match, elem;

		// HANDLE: $(""), $(null), $(undefined), $(false)
		if ( !selector ) {
			return this;
		}

		// Method init() accepts an alternate rootjQuery
		// so migrate can support jQuery.sub (gh-2101)
		root = root || rootjQuery;

		// Handle HTML strings
		if ( typeof selector === "string" ) {
			if ( selector[ 0 ] === "<" &&
				selector[ selector.length - 1 ] === ">" &&
				selector.length >= 3 ) {

				// Assume that strings that start and end with <> are HTML and skip the regex check
				match = [ null, selector, null ];

			} else {
				match = rquickExpr.exec( selector );
			}

			// Match html or make sure no context is specified for #id
			if ( match && ( match[ 1 ] || !context ) ) {

				// HANDLE: $(html) -> $(array)
				if ( match[ 1 ] ) {
					context = context instanceof jQuery ? context[ 0 ] : context;

					// Option to run scripts is true for back-compat
					// Intentionally let the error be thrown if parseHTML is not present
					jQuery.merge( this, jQuery.parseHTML(
						match[ 1 ],
						context && context.nodeType ? context.ownerDocument || context : document,
						true
					) );

					// HANDLE: $(html, props)
					if ( rsingleTag.test( match[ 1 ] ) && jQuery.isPlainObject( context ) ) {
						for ( match in context ) {

							// Properties of context are called as methods if possible
							if ( isFunction( this[ match ] ) ) {
								this[ match ]( context[ match ] );

							// ...and otherwise set as attributes
							} else {
								this.attr( match, context[ match ] );
							}
						}
					}

					return this;

				// HANDLE: $(#id)
				} else {
					elem = document.getElementById( match[ 2 ] );

					if ( elem ) {

						// Inject the element directly into the jQuery object
						this[ 0 ] = elem;
						this.length = 1;
					}
					return this;
				}

			// HANDLE: $(expr, $(...))
			} else if ( !context || context.jquery ) {
				return ( context || root ).find( selector );

			// HANDLE: $(expr, context)
			// (which is just equivalent to: $(context).find(expr)
			} else {
				return this.constructor( context ).find( selector );
			}

		// HANDLE: $(DOMElement)
		} else if ( selector.nodeType ) {
			this[ 0 ] = selector;
			this.length = 1;
			return this;

		// HANDLE: $(function)
		// Shortcut for document ready
		} else if ( isFunction( selector ) ) {
			return root.ready !== undefined ?
				root.ready( selector ) :

				// Execute immediately if ready is not present
				selector( jQuery );
		}

		return jQuery.makeArray( selector, this );
	};
```
### 情况1：处理空参数情况
第2014～2917行处理`selector`为`falsy`的情况，在注释中也有标明。该情况直接返回空`jQuery`对象，不包含任何选中元素，但是返回空`jQuery`对象也是有意义的，这样我们之后就可以继续操作了。
```
		// HANDLE: $(""), $(null), $(undefined), $(false)
		if ( !selector ) {
			return this;
		}
```
### 情况2：处理`html`
如果`selector`参数是字符串，就进入了这个流程：
```
            if ( selector[ 0 ] === "<" &&
				selector[ selector.length - 1 ] === ">" &&
				selector.length >= 3 ) {

				// Assume that strings that start and end with <> are HTML and skip the regex check
				match = [ null, selector, null ];

			} else {
				match = rquickExpr.exec( selector );
			}
```
第2925～2934行对`selector`做了判定，如果是以`<`开头，以`>`结尾并且长度大于3，则认为是一个`HTML`，不需要做正则匹配。如果不满足条件，则对其做正则匹配，其中的`rquickExpr`解析可以看[这里]()，它的作用是匹配出`html`或者`id`。为了保持后续调用的统一，两个流程最后的结果保持正则匹配结果的格式，但是有所不同。如果`selector`是`#id`，那么正则的结果是`['#id', undefind, 'id', ...]`，结果在数组的下标3，如果是`html`，则是`[ null, selector, null ]`，结果在数组的下标2。通过判断下表1和2就能区分命中的是`#id`还是`html`。这种保持数据结构一致的设计模式不错！

第2937行有个判定`if ( match && ( match[ 1 ] || !context ) )`，如果此时`selector`是`html`，则`match[1]`是`selector`的值，该判定为真，进入该流程，第2938～2980行：
```
                // HANDLE: $(html) -> $(array)
				if ( match[ 1 ] ) { // 流程 1
					context = context instanceof jQuery ? context[ 0 ] : context;

					// Option to run scripts is true for back-compat
					// Intentionally let the error be thrown if parseHTML is not present
					jQuery.merge( this, jQuery.parseHTML(
						match[ 1 ],
						context && context.nodeType ? context.ownerDocument || context : document,
						true
					) );

					// HANDLE: $(html, props)
					if ( rsingleTag.test( match[ 1 ] ) && jQuery.isPlainObject( context ) ) { // 流程 2
						for ( match in context ) {

							// Properties of context are called as methods if possible
							if ( isFunction( this[ match ] ) ) {
								this[ match ]( context[ match ] );

							// ...and otherwise set as attributes
							} else {
								this.attr( match, context[ match ] );
							}
						}
					}

					return this;

				// HANDLE: $(#id)
				} else { // 流程3
					elem = document.getElementById( match[ 2 ] );

					if ( elem ) {

						// Inject the element directly into the jQuery object
						this[ 0 ] = elem;
						this.length = 1;
					}
					return this;
				}
```
这里其实处理了3个情况，我们先看第2940行`if ( match[ 1 ] ) `，这里判定了`match[1]`，由之前可以知道，这时候值为`selector`的值，所以进入流程1，第2941～2964行，而这里又有两种情况，我们会慢慢分析。

第2941行对`context`参数进行判定，如果`context`是`jQuery`实例，就取出`jQuery`内部的第一个`Dom`元素作为`context`，如果不是，则保持`context`原样。这里`context`并未传入，则`context`是`undefined`。
```
context = context instanceof jQuery ? context[ 0 ] : context;
```

第2945～2949行将`html`转化成了数组，并合并到了`this`中，其中`jQuery.parseHtml`方法的解析可以看[这里]()。
```
					jQuery.merge( this, jQuery.parseHTML(
						match[ 1 ],
						context && context.nodeType ? context.ownerDocument || context : document,
						true
					) );
```
而第2952行的代码判定`jQuery.isPlainObject(context)`其实是`false`，在这种情况下不会进入流程2，所以直接跳过。

到此对于`$(`<p>123123</p>`)`传入`html`这种场景其实已经可以返回`this`了。

### 情况3：处理`html`和属性
接上一种情况，在第2952行有一句判定：
`if ( rsingleTag.test( match[ 1 ] ) && jQuery.isPlainObject( context ) )`，用来处理`$(html, props)`的情况，其中`rsingleTag`是用来检测单个标签的，具体解析可以看[这里]() 。
```
for ( match in context ) {

	// Properties of context are called as methods if possible
	if ( isFunction( this[ match ] ) ) {
		this[ match ]( context[ match ] );

	// ...and otherwise set as attributes
	} else {
		this.attr( match, context[ match ] );
	}
}
```
如果是单标签，并且`context`是一个简单对象，遍历对象`context`，如果`context`的`key`在`this`中有同名属性，并且是一个函数，则调用该函数，并将对象的这个属性作为参数。如果没有，就调用`attr()`方法为自身添加属性。`attr()`方法解析可以看[这里]()。

然后返回回到情况
### 情况4：处理`id`选择器
接情况1，如果不是`html`，而是一个`id`选择器，则进入这个流程，第2970～2978：
```
elem = document.getElementById( match[ 2 ] );

if ( elem ) {

	// Inject the element directly into the jQuery object
	this[ 0 ] = elem;
	this.length = 1;
}
return this;
```
调用`document.getElementById`，并将选择器作为参数，如果找到元素，就将该元素赋值给`0`属性，并将`length`设置为1，这是因为`document.getElementById`返回值为`HTMLElement | null`。

然后返回`this`。

### 情况5：处理表达式 + jQuery 对象
接下来是处理`selector`为表达式，`context`是一个`jQuery`对象的情况，在第2981～2983行：
```
// HANDLE: $(expr, $(...))                      
} else if ( !context || context.jquery ) {      
	return ( context || root ).find( selector );
```
前面已经判定过了，既不是`html`，也不是`id`选择器，那么就是选择器表达式。就调用`jQuery.find`方法去寻找匹配的元素。
其中`!context || context.jquery`这个判定表示`context`为空，或者拥有`context.jquery`属性，`context.jquery`其实就代表这是一个`jQuery`对象，其中`jquery`属性是`jQuery`的版本号，定义在第132～139行：
```
var                                                                                                          
	version = "3.3.1",                                                                                       
....                                                                                                             
jQuery.fn = jQuery.prototype = {                                                                             
                                                                                                             
	// The current version of jQuery being used                                                              
	jquery: version,                                                                                         
```
`( context || root )`表示如果传入了`jQuery`对象，就使用这个`jQuery`对象，并在这个对象下查找元素，而`root`定义在`2921`行，它接受`jQuery.fn.init`的第三个参数，同时也默认是`rootjQuery`。而`rootjQuery`其实是`rootjQuery = jQuery( document );`，定义在第3014行。

总结，这里处理的是`$(selector, $())`的情况，作用是在`context`下查找`selector`命中的元素，而如果`context`没有传输，默认为`jQuery( document )`，也就是默认在`document`下面查找命中的元素。然后返回`this`

### 情况6：处理表达式 + Dom
```
// HANDLE: $(expr, context)                             
// (which is just equivalent to: $(context).find(expr)  
} else {                                                
	return this.constructor( context ).find( selector );
}                                                       
```
第2985～2989行处理的是`$(expr, context)`的情况，因为上一种情况对`context`不存在或者是`jQuery`对象的情况进行了判定，所以这里就是`context`存在或者不是`jQuery`对象的分支了。就调用`this.constructor( context )`将`context`转化成`jQuery`对象，并在这之上调用`find`方法，查找匹配的元素。然后返回`this`。

### 情况7：处理 Dom 元素
```
// HANDLE: $(DOMElement)             
} else if ( selector.nodeType ) {    
	this[ 0 ] = selector;            
	this.length = 1;                 
	return this;                     
```
第2991~2995处理直接传入`Dom`的情况，如果`selector`不是字符串并且`selector`具有`nodeType`属性，说明他是`Dom`元素。那么直接将它赋值给`0`属性，并设置`length`为1，然后返回`this`。


### 情况8：处理函数 - ready
```
// HANDLE: $(function)                                 
// Shortcut for document ready                         
} else if ( isFunction( selector ) ) {                 
	return root.ready !== undefined ?                  
		root.ready( selector ) :                       
                                                       
		// Execute immediately if ready is not present 
		selector( jQuery );                            
}		
```
第2997～3005行处理`selector`为一个函数的情况，就是我们最常用的`$(()=>{})`情况，这种情况下，表现为`document ready`。将在文档准备好的时候执行。这将具体在[jQuery ready 流程说明]()。

### 情况9：其他情况
不是`html`、不是`ID`选择器、不是表达式、没有传入`context`，不是`falsy`的值，那是什么？还可能是一个对象，这里调用`jQuery.makeArray`将对象变成一个数组然后返回。如果是类数组对象，将被解析为数组，如果是单个对象将直接变成长度为1的数组。在`jQuery`中这挺有意义的，1是可以在类数组上调用`each`、`map`等遍历方法，2是`jQuery`支持事件驱动编程，可以在任意对象上触发自定义事件，帅！