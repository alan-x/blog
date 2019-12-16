### jQuery.extend
`jQuery.extend`函数在`jQuery`源码中随处可见，这个方法的的作用就是拷贝，浅拷贝或者深拷贝。源码位于第277～297行：
```
jQuery.extend = jQuery.fn.extend = function() {
	var options, name, src, copy, copyIsArray, clone,
		target = arguments[ 0 ] || {},
		i = 1,
		length = arguments.length,
		deep = false;

	// Handle a deep copy situation
	if ( typeof target === "boolean" ) {
		deep = target;

		// Skip the boolean and the target
		target = arguments[ i ] || {};
		i++;
	}

	// Handle case when target is a string or something (possible in deep copy)
	if ( typeof target !== "object" && !isFunction( target ) ) {
		target = {};
	}

	// Extend jQuery itself if only one argument is passed
	if ( i === length ) {
		target = this;
		i--;
	}

	for ( ; i < length; i++ ) {

		// Only deal with non-null/undefined values
		if ( ( options = arguments[ i ] ) != null ) {

			// Extend the base object
			for ( name in options ) {
				src = target[ name ];
				copy = options[ name ];

				// Prevent never-ending loop
				if ( target === copy ) {
					continue;
				}

				// Recurse if we're merging plain objects or arrays
				if ( deep && copy && ( jQuery.isPlainObject( copy ) ||
					( copyIsArray = Array.isArray( copy ) ) ) ) {

					if ( copyIsArray ) {
						copyIsArray = false;
						clone = src && Array.isArray( src ) ? src : [];

					} else {
						clone = src && jQuery.isPlainObject( src ) ? src : {};
					}

					// Never move original objects, clone them
					target[ name ] = jQuery.extend( deep, clone, copy );

				// Don't bring in undefined values
				} else if ( copy !== undefined ) {
					target[ name ] = copy;
				}
			}
		}
	}

	// Return the modified object
	return target;
};
```
第228～232行声明了许多的变量：
```
	var options, name, src, copy, copyIsArray, clone,
		target = arguments[ 0 ] || {},
		i = 1,
		length = arguments.length,
		deep = false;
```
- options：要合并的对象
- name：对象中的键名
- src：要复制的源
- copy：要复制的数据
- target：要复制的目标，默认选中第一个参数，如果第一个，如果没有，默认为`{}`
- i：要合并的数据的参数下标
- length：参数列表长度
- deep：是否深拷贝，默认`false`

第234～241行处理深拷贝的情况：
```
	// Handle a deep copy situation
	if ( typeof target === "boolean" ) {
		deep = target;

		// Skip the boolean and the target
		target = arguments[ i ] || {};
		i++;
	}
```
`target`默认选中第一个，即`arguments[0]`，但是如果`arguments[0]`是一个布尔类型的参数，那么这个参数的意义就是指定是否深拷贝，同时，第二个参数，下标1就变成了要复制到的目标，例如
```
jQuery.extend(true, a, b)
```
那么`deep=true`，`target=a`，`i=2`。

第243～246行处理`target`不是对象或者不是函数的的时候，直接将`target`赋值为`{}`
```
	// Handle case when target is a string or something (possible in deep copy)
	if ( typeof target !== "object" && !isFunction( target ) ) {
		target = {};
	}
```

第248～252行代码则处理只有一个参数传递的时候，这表示要拷贝到`jQuery`之上：
```
	// Extend jQuery itself if only one argument is passed
	if ( i === length ) {
		target = this;
		i--;
	}
```
注意1：这里说的一个参数，不是整个方法只有一个参数，而是要合并的对象只有一个。
比如
```
jQuery.extend({})
jQuery.extend(false, {})
```
注意2：这里的i用的很巧，如果第一个参数不传入布尔，则`i=1`，正好是要被合并的对象在`arguments`的开始下标，`i=0`则是`target`，如果传入了布尔，则`target`位于下标1，而要合并的参数的开始下标递增1，对应第240行`i++`；如果要合并的参数只有一个，则参数列表中没有`target`，`target`为`this`，则要合并的参数的开始下标应该前移一位，对应第251行的`i--`。


第254～290遍历参数，使用的正好是上面的`i`作为开始，这细节超帅，不是单纯记录位置，而是用游标一样的东西来实现，超帅：
```
for ( ; i < length; i++ ) {

		// Only deal with non-null/undefined values
		if ( ( options = arguments[ i ] ) != null ) {

			// Extend the base object
			for ( name in options ) {
				src = target[ name ];
				copy = options[ name ];

				// Prevent never-ending loop
				if ( target === copy ) {
					continue;
				}

				// Recurse if we're merging plain objects or arrays
				if ( deep && copy && ( jQuery.isPlainObject( copy ) ||
					( copyIsArray = Array.isArray( copy ) ) ) ) {

					// ...

				// Don't bring in undefined values
				} else if ( copy !== undefined ) {
					target[ name ] = copy;
				}
			}
		}
	}
```
省略深拷贝的部分，先看看浅拷贝。浅拷贝很简单，如果`options`非空，就遍历`options`的属性，如果该属性不是`undefined`，就复制到`target`同名属性上，第284～287行。


深拷贝则是使用递归实现，位于第270～283行：
```
if ( deep && copy && ( jQuery.isPlainObject( copy ) ||
	( copyIsArray = Array.isArray( copy ) ) ) ) {

	if ( copyIsArray ) {
		copyIsArray = false;
		clone = src && Array.isArray( src ) ? src : [];

	} else {
		clone = src && jQuery.isPlainObject( src ) ? src : {};
	}

	// Never move original objects, clone them
	target[ name ] = jQuery.extend( deep, clone, copy );

} 
```
分别将要被覆盖的属性和要拿来覆盖的属性保存为`src`和`copy`，如果`target`和`copy`是同一个引用，则跳过，防止无限递归。如果`copy`存在并且是对象或者数组，就进行深拷贝。第273~279行代码的作用是初始化一个存放新数据的容器，分为两种情况考虑，
如果`src`和`copy`的类型相同，则容器的数据初始化为`src`的数据，之后的数据都将合并进来。但是如果两个类型不同，则容器将初始化为空的`copy`对应的类型，比如数组就初始化为`[]`，对象就初始化为`{}`，然后递归调用`jQuery.extend`：
```
if ( copyIsArray ) {
	copyIsArray = false;
	clone = src && Array.isArray( src ) ? src : [];

} else {
	clone = src && jQuery.isPlainObject( src ) ? src : {};
}

// Never move original objects, clone them
target[ name ] = jQuery.extend( deep, clone, copy );
```

浅拷贝不递归复制对象和数组，也就是说两个对象的对象属性或者数组属性合并只是引用合并，修改他们内部的数据都将使得两个对象变化，而深拷贝则不会，它会递归遍历数组和对象，并拷贝。