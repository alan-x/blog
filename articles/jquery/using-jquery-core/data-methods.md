### 数据方法
我们经常会在元素上存储一些数据。在原生`JavaScript`中，你可能会在`DOM`元素中添加一个属性，但是你需要在一些浏览器上处理内存泄漏问题。`jQuery`提供了一个直接的方式去存储和元素相关的数据，它负责帮你管理内存。

```
// Storing and retrieving data related to an element.
 
$( "#myDiv" ).data( "keyName", { foo: "bar" } );
 
$( "#myDiv" ).data( "keyName" ); // Returns { foo: "bar" }
```
任何类型的数据都可以存储在元素上。就本文而言，`.data()`可以用存储对其他元素的引用。


例如，你可能想要在列表项和`<div>`之间建立联系。每次列表项被访问，都可以建立一次联系，但是最好的方式是只建立一次联系，可以在列表项上使用`.data()`来存储指向`<div>`的指针:

````
// Storing a relationship between elements using .data()
 
$( "#myList li" ).each(function() {
 
    var li = $( this );
    var div = li.find( "div.content" );
 
    li.data( "contentDiv", div );
 
});
 
// Later, we don't have to find the div again;
// we can just read it from the list item's data
var firstLi = $( "#myList li:first" );
 
firstLi.data( "contentDiv" ).html( "new content" );
```
除了给`.data()`传递一个简直对去存储数据，你也可以传递一个包含多个简直对的对象。