# 常见问题
### 我该怎么使用`class`或者`ID`选择一项

这段代码使用`ID``myDivId`选择一个元素。因为`ID`是唯一的，所以该表达式的结果一般是0或者一个元素，这取决于是否存在一个`ID`为`myDivId`的元素。
```
$( "#myDivId" );
```
这段代码使用类`myCssClass`选择一个元素。因为有大量的元素可以有相同的类，这个表达式将会选择很多元素。
1
```
$( ".myCssClass" );
```
一个包含选中元素的`jQuery`对象可以像普通一样赋值给一个`JavaScript`变量

```
var myDivElement = $( "#myDivId" );
```
通常，在`jQuery`对象中的元素被其他`jQuery`函数操作：
```
var myValue = $( "#myDivId" ).val(); // Get the value of a form input.
 
$( "#myDivId" ).val( "hello world" ); // Set the value of a form input.
```

### 当我已经有一个`DOM`元素的时候，我该如何选择元素？
如果你有一个包含`DOM`元素的变量，想要相对于这个`DOM`元素选择元素，可以简单的将它包裹进`jQuery`对象。
```
var myDomElement = document.getElementById( "foo" ); // A plain DOM element.
 
$( myDomElement ).find( "a" ); // Finds all anchors inside the DOM element.
```
很多人尝试拼接一个`DOM`元素或`jQuery`对象和`CSS`选择器。比如：
```
$( myDomElement + ".bar" ); // This is equivalent to $( "[object HTMLElement].bar" );
```
不幸的是，你无法拼接字符串和对象。

### 如何测试一个元素是否有一个指定的类？
`.hasClass()`（在版本1.2中添加）可以解决这种常见的使用场景：
```
$( "div" ).click(function() {
 
    if ( $( this ).hasClass( "protected" ) ) {
 
        $( this )
            .animate({ left: -10 })
            .animate({ left: 10 })
            .animate({ left: -10 })
            .animate({ left: 10 })
            .animate({ left: 0 });
 
    }
 
});
```
你可以使用`.is()`方法配合适当的选择器进行高级的匹配：
5
```
if ( $( "#myDiv" ).is( ".pretty.awesome" ) ) {
 
    $( "#myDiv" ).show();
 
}
```
注意，这个方法也允许你去测试其他的东西。比如你可以测试一个元素是否隐藏（通过使用自定义的`:hidden`选择器）：
```
if ( $( "#myDiv" ).is( ":hidden" ) ) {
 
    $( "#myDiv" ).show();
 
}
```
### 我该怎么测试一个元素是否存在
你可以通过使用`:visible`和`:hidden`选择器来决定一个元素是否被折叠。
```
var isVisible = $( "#myDiv" ).is( ":visible" );
 
var isHidden = $( "#myDiv" ).is( ":hidden" );
```
如果你只是简单的基于元素是否可见来操作元素，可以在选择器表达式包含`:visible`或者`:hidden`。比如
```
$( "#myDiv:visible" ).animate({
 
    left: "+=200px"
 
}, "slow" )
```

### 如何确元素切换的状态
你可以使用`：visible`、`:hidden`选择器来确定元素是否展开。

```
var isVisible = $( "#myDiv" ).is( ":visible" );
 
var isHidden = $( "#myDiv" ).is( ":hidden" );
```
如果你指示像在元素可见的时候操作元素，可以将`:visible`或者`:hidden`包含到选择器表达式，比如：
```
$( "#myDiv:visible" ).animate({
 
    left: "+=200px"
 
}, "slow" );
```
### 我该如何通过ID选中一个ID中有符号的元素
因为`jQuery`使用`CSS`语法选择元素，一些字符被解释为`CSS`声明，为了告诉`jQuery`对待这些字符像对待字面量一样，而不是`CSS`声明，他们必须通过在前面添加两个反斜杆。
查阅[选择器文档]()产看更多详细信息。
```
// Does not work:
$( "#some:id" )
 
// Works!
$( "#some\\:id" )
 
// Does not work:
$( "#some.id" )
 
// Works!
$( "#some\\.id" )
```
下面的函数考虑到了这些字符串，并在ID前添加了一个`#`：
```
function jq( myid ) {
 
    return "#" + myid.replace( /(:|\.|\[|\]|,|=|@)/g, "\\$1" );
 
}
```
该函数可以这么使用：
```
$( jq( "some.id" ) )
```
### 如何启用/禁用一个表单元素
你可以使用`.prop()`方法启用、禁用一个表单元素：
```
// Disable #x
$( "#x" ).prop( "disabled", true );
 
// Enable #x
$( "#x" ).prop( "disabled", false );
```
如何选中/取消选中一个`checkbox`或者`radio`？
你可以使用`.prop()`方法选中/取消选中一个`checkbox`元素或者`radio`元素：
```
// Check #x
$( "#x" ).prop( "checked", true );
 
// Uncheck #x
$( "#x" ).prop( "checked", false );
```
### 如何获取选中`option`的文本内容
`select`元素通常有两个值经常被获取。第一个是将要被发送到服务端的值，这很简单：
1
2
```
$( "#myselect" ).val();
// => 1
```
第二是`select`的文本内容。比如，使用下面的`select`:
```
<select id="myselect">
    <option value="1">Mr</option>
    <option value="2">Mrs</option>
    <option value="3">Ms</option>
    <option value="4">Dr</option>
    <option value="5">Prof</option>
</select>
```
如果你想要获取第一个选中的`option`的`Mr`字符串(而不是1)，你可以这么做：
```
$( "#myselect option:selected" ).text();
// => "Mr"
```
### 我如何在一个拥有10项数据的列表里替换第三个元素的文本？
不管是`:eq()`还是`.eq()`方法都可以让你选择正确的元素，然而，为了替换文本，你必须在设置之前获取到它：
```
// This doesn't work; text() returns a string, not the jQuery object:
$( this ).find( "li a" ).eq( 2 ).text().replace( "foo", "bar" );
 
// This works:
var thirdLink = $( this ).find( "li a" ).eq( 2 );
 
var linkText = thirdLink.text().replace( "foo", "bar" );

thirdLink.text( linkText );
```
第一个栗子只是替换找到的文本。第二个栗子保存找到的文本并使用新的文本替换旧的文本。记住`.text()`获取，`.text("foo")`设置。

### 如何从`jQuery`对象中获取原生`DOM`对象？

一个`jQuery`对象是一个类数组包裹着一个或者多个`DOM`元素。为了获取真实的`DOM`引用（而不是`jQuery`对象）。你有两个选择。第一个（最快的）方法是使用数组索引：
```
$( "#foo" )[ 0 ]; // Equivalent to document.getElementById( "foo" )
```
第二个方法是使用`.get()`方法：
```
$( "#foo" ).get( 0 ); // Identical to above, only slower.
```
你可以调用无参的`.get()`去获取真实的`DOM`元素数组。