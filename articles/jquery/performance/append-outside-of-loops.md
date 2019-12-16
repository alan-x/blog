### 在循环外拼接
操作`DOM`带来很大的损耗。如果你拼接大量的元素到`DOM`中，你会想要一次性全部拼接，而不是每次拼接一个。在循环内拼接元素是一个很常见的问题。
```
$.each( myArray, function( i, item ) {
 
    var newListItem = "<li>" + item + "</li>";
 
    $( "#ballers" ).append( newListItem );
 
});
```
一个常见的技术是利用文档片段。在每一次循环迭代中，你拼接元素到片段，而不是`DOM`元素。在循环之后，只要拼接片段到`DOM`元素
```
var frag = document.createDocumentFragment();
 
$.each( myArray, function( i, item ) {
 
    var newListItem = document.createElement( "li" );
    var itemText = document.createTextNode( item );
 
    newListItem.appendChild( itemText );
 
    frag.appendChild( newListItem );
 
});
 
$( "#ballers" )[ 0 ].appendChild( frag );
```
另一个简单的技术是在每一个循环迭代中拼接字符串。在循环之后，设置字符串为`DOM`元素的`HTML`。
```
var myHtml = "";
 
$.each( myArray, function( i, item ) {
 
    myHtml += "<li>" + item + "</li>";
 
});
 
$( "#ballers" ).html( myHtml );
```
当然还有其他的技术你可以测试。一个测试这些很好的方式是通过[jsperf]()这个站点。这个站点允许你对每个技术进行基准测试，并直观的看到它在所有浏览器的表现。