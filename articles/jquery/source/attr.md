### attr
`jQuery`可以通过`attr`来获取`html`元素的属性，这部分代码定义在第7514～7641行，其中第7517～7527定义的是`jQuery.fn`上的`attr`方法和`removeAttr`方法，可以直接在`jQuery`对象上调用，他是`jQuery`上`attr`和`removeAttr`的便捷封装：
```
jQuery.fn.extend( {
	attr: function( name, value ) {
		return access( this, jQuery.attr, name, value, arguments.length > 1 );
	},

	removeAttr: function( name ) {
		return this.each( function() {
			jQuery.removeAttr( this, name );
		} );
	}
} );
```
其中`access`方法是一个通用方法，具体解析可以看[这里]()。`attr`是通过`access`方法调用`jQuery.attr`，而`removeAttr`则是通过`each`方法在每个元素上调用`removeAttr`方法，关于`each`可以看[这里]()。


第7529～7606行定义了真正的`attr`和`removeAttr`方法，基本声明如下：
```
jQuery.extend({
    attr: function(elem, name, value){...},
    attrHooks: {...},
    removeAttr: function(elem, value){...},
})
```
首先是`attr`方法，第7530～7564行：
```
	attr: function( elem, name, value ) {
		var ret, hooks,
			nType = elem.nodeType;

		// Don't get/set attributes on text, comment and attribute nodes
		if ( nType === 3 || nType === 8 || nType === 2 ) {
			return;
		}

		// Fallback to prop when attributes are not supported
		if ( typeof elem.getAttribute === "undefined" ) {
			return jQuery.prop( elem, name, value );
		}

		// Attribute hooks are determined by the lowercase version
		// Grab necessary hook if one is defined
		if ( nType !== 1 || !jQuery.isXMLDoc( elem ) ) {
			hooks = jQuery.attrHooks[ name.toLowerCase() ] ||
				( jQuery.expr.match.bool.test( name ) ? boolHook : undefined );
		}

		if ( value !== undefined ) {
			if ( value === null ) {
				jQuery.removeAttr( elem, name );
				return;
			}

			if ( hooks && "set" in hooks &&
				( ret = hooks.set( elem, value, name ) ) !== undefined ) {
				return ret;
			}

			elem.setAttribute( name, value + "" );
			return value;
		}

		if ( hooks && "get" in hooks && ( ret = hooks.get( elem, name ) ) !== null ) {
			return ret;
		}

		ret = jQuery.find.attr( elem, name );

		// Non-existent attributes return null, we normalize to undefined
		return ret == null ? undefined : ret;
	},
```
第7534～7537行：不在`text`，`comment`， `attribute`节点上执行这个方法，完整的`nodeType`列表及其对应的数字可以在[这里]()找到：
```
// Don't get/set attributes on text, comment and attribute nodes      
if ( nType === 3 || nType === 8 || nType === 2 ) {                    
	return;                                                           
}                                                                     
```

第7539～7542 行：如果不支持`getAttribute`方法，则使用`jQuery.prop`方法，关于`jQuery.prop`方法解析，可以看[这里]()：
```
// Fallback to prop when attributes are not supported      
if ( typeof elem.getAttribute === "undefined" ) {          
	return jQuery.prop( elem, name, value );               
}                                                          
```

第7544～7549行：如果不是元素节点或者不是`xml`，则查找是否有`hook`:
```
// Attribute hooks are determined by the lowercase version               
// Grab necessary hook if one is defined                                 
if ( nType !== 1 || !jQuery.isXMLDoc( elem ) ) {                         
	hooks = jQuery.attrHooks[ name.toLowerCase() ] ||                    
		( jQuery.expr.match.bool.test( name ) ? boolHook : undefined );  
}                                                                      
```
这一段需要特别解释，`jQuery.attrHooks`定义在第7576～7590行，它为`type`属性定义了一个`set`的`hook`，当我们设置`type`属性的时候，将会调用这个`hook`：  
```
attrHooks: {                                                
	type: {                                                 
		set: function( elem, value ) {                      
			if ( !support.radioValue && value === "radio" &&
				nodeName( elem, "input" ) ) {               
				var val = elem.value;                       
				elem.setAttribute( "type", value );         
				if ( val ) {                                
					elem.value = val;                       
				}                                           
				return value;                               
			}                                               
		}                                                   
	}                                                       
},                          
这里有一个`support`，具体解析可以看[这里]()，它的作用就是判断是否支持某个特性，可以称为特性嗅探。

这里的作用是在设置一个`input`的`type`属性之前将值保存，然后再之后恢复。这么做是因为`IE<=11`的时候`input`变成`radio`将会丢失`value`。
```
除了检查`attrHooks`之外，还会检测属性的值是否是布尔类型的，用的是`jQuery.expr.match.bool`正则去校验，如果是布尔类型的，则使用`boolHook`，其中`jQuery.expr.match.bool`定义在第2757行：
```
jQuery.expr = Sizzle.selectors;
```
`Sizzle.selectors`定义在第1612~2114行，`match`属性定义在1619行，值为`matchExpr`： 
```
Expr = Sizzle.selectors = {                 
                                            
	// Can be adjusted by the user          
	cacheLength: 50,                        
                                            
	createPseudo: markFunction,             
                                            
	match: matchExpr,                      
}
```
`matchExpr`定义在第609~622行，这里定义了许多的正则，其中`jQuery.expr.match.bool`就是其中的`bool`：
```
matchExpr = {                                                                                               
	"ID": new RegExp( "^#(" + identifier + ")" ),                                                           
	"CLASS": new RegExp( "^\\.(" + identifier + ")" ),                                                      
	"TAG": new RegExp( "^(" + identifier + "|[*])" ),                                                       
	"ATTR": new RegExp( "^" + attributes ),                                                                 
	"PSEUDO": new RegExp( "^" + pseudos ),                                                                  
	"CHILD": new RegExp( "^:(only|first|last|nth|nth-last)-(child|of-type)(?:\\(" + whitespace +            
		"*(even|odd|(([+-]|)(\\d*)n|)" + whitespace + "*(?:([+-]|)" + whitespace +                          
		"*(\\d+)|))" + whitespace + "*\\)|)", "i" ),                                                        
	"bool": new RegExp( "^(?:" + booleans + ")$", "i" ),                                                    
	// For use in libraries implementing .is()                                                              
	// We use this for POS matching in `select`                                                             
	"needsContext": new RegExp( "^" + whitespace + "*[>+~]|:(even|odd|eq|gt|lt|nth|first|last)(?:\\(" +     
		whitespace + "*((?:-\\d)?\\d*)" + whitespace + "*\\)|)(?=[^-]|$)", "i" )                            
```
`bool`的正则为`new RegExp( "^(?:" + booleans + ")$", "i" )` ，其中`booleans`的定义在第569行：
```
	booleans = "checked|selected|async|autofocus|autoplay|controls|defer|disabled|hidden|ismap|loop|multiple|open|readonly|required|scoped",
```
所以`jQuery.expr.match.bool.test( name )`就是测试属性是否是上述的属性之一，如果是，说明这个属性的值是`bool`类型，需要使用`boolHook`来做处理。

`boolHook`的定义在第7608～7620行，定义了一个`set`操作：
```
// Hooks for boolean attributes                                 
boolHook = {                                                    
	set: function( elem, value, name ) {                        
		if ( value === false ) {                                
                                                                
			// Remove boolean attributes when set to false      
			jQuery.removeAttr( elem, name );                    
		} else {                                                
			elem.setAttribute( name, name );                    
		}                                                       
		return name;                                            
	}                                                           
};                                                              
```
这里也需要特别解释，在`html`中，如果是布尔类型的数据，是不需要设置值的，比如
```
<input hidden/>
<input hidden=''/>
<input hidden='true'/>
<input hidden='false'/>
```
`input`都是隐藏的，因为在`html`中，属性都是字符串，没有数据类型，如果要让属性为`false`，则需要移除属性；如果要让属性为`true`，只要属性存在或者设置任意值就行了，通常设置为属性同名的值。这是有点反常规的，所以`jQuery`做了个`hook`，让我们可以使用常规的方法去操作，而不关心这种细节处理。

第7551～7564行是一个`setter`，当`value`不为`undefined`的时候就会进入这个流程：
```
if ( value !== undefined ) {                                       
	if ( value === null ) {                                        
		jQuery.removeAttr( elem, name );                           
		return;                                                    
	}                                                              
                                                                   
	if ( hooks && "set" in hooks &&                                
		( ret = hooks.set( elem, value, name ) ) !== undefined ) { 
		return ret;                                                
	}                                                              
                                                                   
	elem.setAttribute( name, value + "" );                         
	return value;                                                  
}                                                                  
```
如果`value`不是`undefind`但是是`null`，它的作用其实和`removeAttr`是一样的，直接移除了属性。如果有`hook`并且`hook`中有`set`操作，则执行该操作，并且判断操作的返回值是否是`undefined`，如果不是，就返回操作的结果。如果是，则继续往下，调用`setAttribute`方法为元素设置属性，然后返回值。

注意：`boolHook`中的`set`方法的声明是`function( elem, value, name )`，和我们通常见到的`function( elem, name, value )`后两个参数反了，为啥？因为`attrHooks`是关于属性的`hook`，它的声明是`function( elem, value )`，是针对特定属性的`hook`，它已经知道自身所针对的属性，所以它不需要知道属性名。而`boolHook`是对所有布尔类型属性使用的，它需要知道属性名。但是获取`hook`的过程是相同的：
```
hooks = jQuery.attrHooks[ name.toLowerCase() ] ||                    
	( jQuery.expr.match.bool.test( name ) ? boolHook : undefined );  
```
保持一致的声明让使用也一致，不需要关心到底是`attrHook`还是`boolHook`：
```
hooks.set( elem, value, name )
```


如果不是一个`setter`，那就是一个`getter`，第7566～7573行：
```
if ( hooks && "get" in hooks && ( ret = hooks.get( elem, name ) ) !== null ) {
	return ret;                                                               
}                                                                             
                                                                              
ret = jQuery.find.attr( elem, name );                                         
                                                                              
// Non-existent attributes return null, we normalize to undefined             
return ret == null ? undefined : ret;                                  
```
它依旧去`hook`中尝试寻找`get`操作，如果有，就执行该操作并返回值。如果没有或者没有获取到值，则调用`jQuery.find.attr`方法。
最后对返回值做一个处理，不返回`null`，而返回结果或者`undefined`。




`removeAttr`定义在第7592～7606行：
```
removeAttr: function( elem, value ) {                                      
	var name,                                                              
		i = 0,                                                             
                                                                           
		// Attribute names can contain non-HTML whitespace characters      
		// https://html.spec.whatwg.org/multipage/syntax.html#attributes-2 
		attrNames = value && value.match( rnothtmlwhite );                 
                                                                           
	if ( attrNames && elem.nodeType === 1 ) {                              
		while ( ( name = attrNames[ i++ ] ) ) {                            
			elem.removeAttribute( name );                                  
		}                                                                  
	}                                                                      
}                                                                          
```
代码很简单，`value.match(rnothtmlwhite)`将`value`分割成多个字符串，其实就是多个属性，然后遍历属性数组，在元素上调用`removeAttribute`方法来删除属性。这里关于`while`的用法很有意思，`while`会在条件为`false`的时候停止运行，而`(name=attrNames[i++])`会在获取不到的时候返回`undefined`，所以就自然跳出了循环，`jQuery`中有许多这种用法，有点好玩。

