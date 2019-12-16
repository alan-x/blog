### Data 

`Data`模块很简单，就是在元素上保存一个属性，该属性指向一个对象，这个对象管理所有你添加到这个元素上的所有数据，就是这样！

`Data`相关的代码从4002～4324行。

从4002～4004行声明了`Data`对象：
```
	function Data() {
		this.expando = jQuery.expando + Data.uid++;
	}
```
其中包含一个属性`expando`，这个属性是`jQuery.expando`和`Data.uid`属性拼接而成。其中`jQuery.expando`在第`297`行声明：
```
expando: "jQuery" + (version + Math.random()).replace(/\D/g, ""),
```
`Data.uid`在第4006行声明：
```
	Data.uid = 1;
```
初始值为1，每次调用`new Data`之后都会递增1，为每一个`Data`实例带来不一样的`expando`属性，而这个属性将作为属性名将挂载在元素上


第4004～4326行声明了`Data`的系列方法：
```
Data.prototype={
    cache: ...
    set: ...
    access: ...
    remove: ...
    hasData: ...
}
```

第4154～4156行声明了两个`Data`实例：
```
var dataPriv = new Data();

var dataUser = new Data();
```

我们常常使用的方法定义在4245～4326行：
```
jQuery.fn.extend( {
    data: function( key, value ){},

    removeData: function( key ){}
}
```
只有两个方法，`data`用来获取或者设置属性：
```
data: function( key, value ) {
		var i, name, data,
			elem = this[ 0 ],
			attrs = elem && elem.attributes;

		// Gets all values
		if ( key === undefined ) {
			if ( this.length ) {
				data = dataUser.get( elem );

				if ( elem.nodeType === 1 && !dataPriv.get( elem, "hasDataAttrs" ) ) {
					i = attrs.length;
					while ( i-- ) {

						// Support: IE 11 only
						// The attrs elements can be null (#14894)
						if ( attrs[ i ] ) {
							name = attrs[ i ].name;
							if ( name.indexOf( "data-" ) === 0 ) {
								name = camelCase( name.slice( 5 ) );
								dataAttr( elem, name, data[ name ] );
							}
						}
					}
					dataPriv.set( elem, "hasDataAttrs", true );
				}
			}

			return data;
		}

		// Sets multiple values
		if ( typeof key === "object" ) {
			return this.each( function() {
				dataUser.set( this, key );
			} );
		}

		return access( this, function( value ) {
			var data;

			// The calling jQuery object (element matches) is not empty
			// (and therefore has an element appears at this[ 0 ]) and the
			// `value` parameter was not undefined. An empty jQuery object
			// will result in `undefined` for elem = this[ 0 ] which will
			// throw an exception if an attempt to read a data cache is made.
			if ( elem && value === undefined ) {

				// Attempt to get data from the cache
				// The key will always be camelCased in Data
				data = dataUser.get( elem, key );
				if ( data !== undefined ) {
					return data;
				}

				// Attempt to "discover" the data in
				// HTML5 custom data-* attrs
				data = dataAttr( elem, key );
				if ( data !== undefined ) {
					return data;
				}

				// We tried really hard, but the data doesn't exist.
				return;
			}

			// Set the data...
			this.each( function() {

				// We always store the camelCased key
				dataUser.set( this, key, value );
			} );
		}, null, value, arguments.length > 1, null, true );
	},
```
我们现考虑`data(key,value)`的情况，这种情况时候，直接进入第4284行，通过`access`，而`access`之后将会进入到第4313行代码：
```
			// Set the data...
			this.each( function() {

				// We always store the camelCased key
				dataUser.set( this, key, value );
			} );
```
遍历当前`jQuery`对象的所有元素并调用`Data.prototype.set`方法为每一个元素设置值，`Data.prototype.set`在第4045～4063行：
```
	set: function( owner, data, value ) {
		var prop,
			cache = this.cache( owner );

		// Handle: [ owner, key, value ] args
		// Always use camelCase key (gh-2257)
		if ( typeof data === "string" ) {
			cache[ camelCase( data ) ] = value;

		// Handle: [ owner, { properties } ] args
		} else {

			// Copy the properties one-by-one to the cache object
			for ( prop in data ) {
				cache[ camelCase( prop ) ] = data[ prop ];
			}
		}
		return cache;
	},
```
这里调用了`Data.prototype.cache`方法获取元素上的数据，这个方法声明在第4012～4044行：
```

cache: function( owner ) {

		// Check if the owner object already has a cache
		var value = owner[ this.expando ];

		// If not, create one
		if ( !value ) {
			value = {};

			// We can accept data for non-element nodes in modern browsers,
			// but we should not, see #8335.
			// Always return an empty object.
			if ( acceptData( owner ) ) {

				// If it is a node unlikely to be stringify-ed or looped over
				// use plain assignment
				if ( owner.nodeType ) {
					owner[ this.expando ] = value;

				// Otherwise secure it in a non-enumerable property
				// configurable must be true to allow the property to be
				// deleted when data is removed
				} else {
					Object.defineProperty( owner, this.expando, {
						value: value,
						configurable: true
					} );
				}
			}
		}

		return value;
	},
```
这个方法访问元素上的`[this.expando]`属性，`this.expando`的值就是在构造`Data` 实例的时候生成的。如果这个属性值为空，则赋值为`{}`，这中间对于赋值方式还有一定的判断，首先只能对`Node`和`Object`对象赋值，这利用了`acceptData`，`acceptData`声明在第3990～39999行：
```
var acceptData = function( owner ) {

	// Accepts only:
	//  - Node
	//    - Node.ELEMENT_NODE
	//    - Node.DOCUMENT_NODE
	//  - Object
	//    - Any
	return owner.nodeType === 1 || owner.nodeType === 9 || !( +owner.nodeType );
};
```
其次在赋值的时候对不同的类型使用不同的方法，对于`Node`，直接赋值。对于对象，则使用`Object.defineproperty`。

最后返回的是元素上的`[expando]`属性的值，或者是一个空对象。

接着往下，如果`data`为字符串，加入`cache`中，如果不是，则遍历`data`，然后将每个属性加入到`cache`中，最后返回`cache`。

`data(key, value)`的流程就是这样了，结果就是在元素上添加一个属性，这个属性的名是`dataUser.expando`的值，属性的值为一个对象，对象中包含所有用`data(key,value)`保存的值，

如果使用的是`data(key)`形式，而`key`是一个对象，则将会走入批量赋值流程，第4277～4282行：
```
		// Sets multiple values
		if ( typeof key === "object" ) {
			return this.each( function() {
				dataUser.set( this, key );
			} );
		}
```
依旧调用的是`Data.prototype.data`方法，但此时`key`是对象，所以将会遍历`key`属性，将`key`的每个属性和值赋值到`cache`中。





当`key`为`undefined`的时候，`data()`方法的作用是获取元素上所有的数据：
```
		var i, name, data,
			elem = this[ 0 ],
			attrs = elem && elem.attributes;

		// Gets all values
        if ( key === undefined ) {
			if ( this.length ) {
				data = dataUser.get( elem );

				if ( elem.nodeType === 1 && !dataPriv.get( elem, "hasDataAttrs" ) ) {
					i = attrs.length;
					while ( i-- ) {

						// Support: IE 11 only
						// The attrs elements can be null (#14894)
						if ( attrs[ i ] ) {
							name = attrs[ i ].name;
							if ( name.indexOf( "data-" ) === 0 ) {
								name = camelCase( name.slice( 5 ) );
								dataAttr( elem, name, data[ name ] );
							}
						}
					}
					dataPriv.set( elem, "hasDataAttrs", true );
				}
			}

			return data;
		}
```
通过`dataUser.get(elem)`来获取，而`elem = this[0]`，所以`data()`方法其实只会获取`jQuery`对象中第一个元素的`data`。

`dataUser`是上面声明的一个`Data`的实例，用户空间的数据都存储在这里，我们调用`Data.prototype.get`方法来设置数据，第4064～4070行：
```
	get: function( owner, key ) {
		return key === undefined ?
			this.cache( owner ) :

			// Always use camelCase key (gh-2257)
			owner[ this.expando ] && owner[ this.expando ][ camelCase( key ) ];
	},
```
很简单，如果`key`是`undefined`，就调用`this.cache(owner)`方法，由`set`流程可以知道，其实`Data`就是把值设置到元素的`[dataUser.expando]`属性上，这里就是把这个属性的值取出来，就是将元素上的所有数据取出来了。

如果`key`不为空，则走入了`access`流程的第4287行：
```
            // The calling jQuery object (element matches) is not empty
			// (and therefore has an element appears at this[ 0 ]) and the
			// `value` parameter was not undefined. An empty jQuery object
			// will result in `undefined` for elem = this[ 0 ] which will
			// throw an exception if an attempt to read a data cache is made.
			if ( elem && value === undefined ) {

				// Attempt to get data from the cache
				// The key will always be camelCased in Data
				data = dataUser.get( elem, key );
				if ( data !== undefined ) {
					return data;
				}

				// Attempt to "discover" the data in
				// HTML5 custom data-* attrs
				data = dataAttr( elem, key );
				if ( data !== undefined ) {
					return data;
				}

				// We tried really hard, but the data doesn't exist.
				return;
			}
```
调用`Data.prototype.get`方法获取某个`key`的数据，又走入了`Data.prototype.get`方法，但是这时候的`key !== undefined`，所以，调用的是
```
owner[ this.expando ] && owner[ this.expando ][ camelCase( key ) ];
```
也很简单，`owner[ this.expando ]`是获取元素上的`[dataUser.expando]`属性的值，该属性的值是一个对象，然后获取对象上的`key`属性的值返回，就是我们要的数据了。
如果这个时候数据依旧是`undefined`，也就是不存在这个数据，则会尝试获取元素自定义`data`属性的值，第4301～4306：
```
                / Attempt to "discover" the data in
				// HTML5 custom data-* attrs
				data = dataAttr( elem, key );
				if ( data !== undefined ) {
					return data;
				}
```
其中`dataAttr`方法定义在第4198～4219行：
```
function dataAttr( elem, key, data ) {
	var name;

	// If nothing was found internally, try to fetch any
	// data from the HTML5 data-* attribute
	if ( data === undefined && elem.nodeType === 1 ) {
		name = "data-" + key.replace( rmultiDash, "-$&" ).toLowerCase();
		data = elem.getAttribute( name );

		if ( typeof data === "string" ) {
			try {
				data = getData( data );
			} catch ( e ) {}

			// Make sure we set the data so it isn't changed later
			dataUser.set( elem, key, data );
		} else {
			data = undefined;
		}
	}
	return data;
}
```
如果`data`是`undefined`并且`elem`是元素，则使用`getAttribute`方法获取元素上的属性，获取的时候需要在属性名前面拼接上`data-`，因为这是`html`自定义属性的前缀。如果获取到了，则使用`getData`进行数据转化，因为`html`的属性只有字符串，但是`js`是有类型的。
获取到数据之后，将它添加到`dataUser`当中，调用的依旧是`Data.prototype.set`方法，这样接下来就可以不去属性中拿了。

其中`getData`方法声明在第4170～4196行:
```
var rbrace = /^(?:\{[\w\W]*\}|\[[\w\W]*\])$/,
	rmultiDash = /[A-Z]/g;

function getData( data ) {
	if ( data === "true" ) {
		return true;
	}

	if ( data === "false" ) {
		return false;
	}

	if ( data === "null" ) {
		return null;
	}

	// Only convert to a number if it doesn't change the string
	if ( data === +data + "" ) {
		return +data;
	}

	if ( rbrace.test( data ) ) {
		return JSON.parse( data );
	}

	return data;
}

```
其中`rbrace`是检测字符串是否以`{}`或者`[]`包裹，如果是，使用`JSON.parse`转化成对象或者数组。


由以上可知，我们可以通过`$(selector).data(name)`来直接获取元素的自定义属性，而且不需要添加`data-`前缀。

`removeData`定义在`4321~4325`行：
```
	removeData: function( key ) {
		return this.each( function() {
			dataUser.remove( this, key );
		} );
	}
```
遍历`jQuery`对象中所有的元素并调用`Data.prototype.remove`方法移除对应`key`的数据，其中`remove`方法定义在第4102～4148行：
```
	remove: function( owner, key ) {
		var i,
			cache = owner[ this.expando ];

		if ( cache === undefined ) {
			return;
		}

		if ( key !== undefined ) {

			// Support array or space separated string of keys
			if ( Array.isArray( key ) ) {

				// If key is an array of keys...
				// We always set camelCase keys, so remove that.
				key = key.map( camelCase );
			} else {
				key = camelCase( key );

				// If a key with the spaces exists, use it.
				// Otherwise, create an array by matching non-whitespace
				key = key in cache ?
					[ key ] :
					( key.match( rnothtmlwhite ) || [] );
			}

			i = key.length;

			while ( i-- ) {
				delete cache[ key[ i ] ];
			}
		}

		// Remove the expando if there's no more data
		if ( key === undefined || jQuery.isEmptyObject( cache ) ) {

			// Support: Chrome <=35 - 45
			// Webkit & Blink performance suffers when deleting properties
			// from DOM nodes, so set to undefined instead
			// https://bugs.chromium.org/p/chromium/issues/detail?id=378607 (bug restricted)
			if ( owner.nodeType ) {
				owner[ this.expando ] = undefined;
			} else {
				delete owner[ this.expando ];
			}
		}
	},
```
移除的步骤如下：
先将元素的整个数据拿出来，如果本来就不存在数据，就算了：
```
var i,                             
	cache = owner[ this.expando ]; 
                                   
if ( cache === undefined ) {       
	return;                        
}                                  
```
如果有数据并传入了`key`，说明要移除这个`key`及其对应的数据。如果`key`是数组，则将所有的`key`转化成驼峰；如果`key`是字符串，并且`cache`中存在这个`key`，则将这个`key`变成`[key]`，便于之后的统一操作，而如果`key`是空白符，则让`key`为`[]`。这里的一波操作最终都将`key`变成了数组，所以在最后可以通过遍历来移除缓存中所有的`key`及其对应的值:
```
		if ( key !== undefined ) {

			// Support array or space separated string of keys
			if ( Array.isArray( key ) ) {

				// If key is an array of keys...
				// We always set camelCase keys, so remove that.
				key = key.map( camelCase );
			} else {
				key = camelCase( key );

				// If a key with the spaces exists, use it.
				// Otherwise, create an array by matching non-whitespace
				key = key in cache ?
					[ key ] :
					( key.match( rnothtmlwhite ) || [] );
			}

			i = key.length;

			while ( i-- ) {
				delete cache[ key[ i ] ];
			}
		}
```
如果`key`是`undefined`或者`cache`是一个空的对象，则将会从元素上断开引用，而断开引用的方式不同的对象使用不同的方法。如果是`Node`则使用直接赋值为`undefined`，如果是对象，则使用`delete`，第4135～4147行:
```
		// Remove the expando if there's no more data
		if ( key === undefined || jQuery.isEmptyObject( cache ) ) {

			// Support: Chrome <=35 - 45
			// Webkit & Blink performance suffers when deleting properties
			// from DOM nodes, so set to undefined instead
			// https://bugs.chromium.org/p/chromium/issues/detail?id=378607 (bug restricted)
			if ( owner.nodeType ) {
				owner[ this.expando ] = undefined;
			} else {
				delete owner[ this.expando ];
			}
		}
```



