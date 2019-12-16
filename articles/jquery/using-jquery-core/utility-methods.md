Utility Methods
### 工具方法
`jQuery`在`$`命名空间上提供了许多的工具玩法。这些方法对完成日常编程任务来说非常有帮助。了解更多关于`jQuery`方法类，可以查阅[在 api.jquery.com 上的工具类文档]()
下面是一些工具类方法的栗子：

### `$.trim()`
移除开始和结束的空白字符
```
// Returns "lots of extra whitespace"
$.trim( "    lots of extra whitespace    " );
```
### `$.each()`
遍历对象和数组：
```
$.each([ "foo", "bar", "baz" ], function( idx, val ) {
    console.log( "element " + idx + " is " + val );
});
 
$.each({ foo: "bar", baz: "bim" }, function( k, v ) {
    console.log( k + " : " + v );
});
```
`.each()`方法可以在选择器上被调用，用来遍历选择器包含的方法。选择器使用`.each()`方法来遍历选择器中的元素，而不是`$.each()`。
link$.inArray()
### `$.inArray()`
返回值在数组中的索引，如果没找到就返回`-1`。

```
var myArray = [ 1, 2, 3, 5 ];
 
if ( $.inArray( 4, myArray ) !== -1 ) {
    console.log( "found it!" );
}
```
### `$.extend()`
使用后面的对象属性改变第一个对象属性：

```
var firstObject = { foo: "bar", a: "b" };
var secondObject = { foo: "baz" };
 
var newObject = $.extend( firstObject, secondObject );
 
console.log( firstObject.foo ); // "baz"
console.log( newObject.foo ); // "baz"
```
如果你不想要该拜年任何你传给`$.extend()`的对象，可以传递一个空的对象为第一个参数：

```
var firstObject = { foo: "bar", a: "b" };
var secondObject = { foo: "baz" };
 
var newObject = $.extend( {}, firstObject, secondObject );
 
console.log( firstObject.foo ); // "bar"
console.log( newObject.foo ); // "baz"
```
### `$.proxy()`
返回一个函数，该函数只会运行在提供的范围内-也就是说将传递的函数的`this`设置为第二个参数。

```
var myFunction = function() {
    console.log( this );
};
var myObject = {
    foo: "bar"
};
 
myFunction(); // window
 
var myProxyFunction = $.proxy( myFunction, myObject );
 
myProxyFunction(); // myObject
```
如果你有一个包含方法的对象，你可以传递一个对象和方法的名字，该方法返回一个函数，该函数总数运行在对象的作用域下。
```
var myObject = {
    myFn: function() {
        console.log( this );
    }
};
 
$( "#foo" ).click( myObject.myFn ); // HTMLElement #foo
$( "#foo" ).click( $.proxy( myObject, "myFn" ) ); // myObject
```
### 类型检测
有时候`typeof`操作符容易[混淆或不一致]()，作为替代`typeof`的使用，`jQuery`提供了工具方法去帮助决定值的类型。

首先，你有方法去检测特定的值是特定的类型。

```
$.isArray([]); // true
$.isFunction(function() {}); // true
$.isNumeric(3.14); // true
```
其次，`$.type()`可以用来检测创建值的内部类。你可以将该方法作为`typeof`操作符更好的替代方法。

```
$.type( true ); // "boolean"
$.type( 3 ); // "number"
$.type( "test" ); // "string"
$.type( function() {} ); // "function"
 
$.type( new Boolean() ); // "boolean"
$.type( new Number(3) ); // "number"
$.type( new String('test') ); // "string"
$.type( new Function() ); // "function"
 
$.type( [] ); // "array"
$.type( null ); // "null"
$.type( /test/ ); // "regexp"
$.type( new Date() ); // "date"
```
就像往常一样，你可以查看[API]()文档做更深入的了解。