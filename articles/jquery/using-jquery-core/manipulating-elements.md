### 操作元素

获取更完整的`jQuery`操作方法，可以访问[api.jquery.com`操作文档]()。

### 获取和设置元素的信息


有很多方式可以改变一个存在的元素。大部分普通的任务都是改变一个元素内部的`HTML`和或属性。对于这些操作，`jQuery`提供了简单的、跨浏览器的方法。你也可以在元素的`getter`返回值使用那些相同的方法。获取更多关于`getter`和`setter`的信息，可以看[选择器操作]()章节。下面是可以用来获取或者操作元素信息的部分方法：

- `.html()` – 获取或者设置`HTML`内容。
- `.text()` – 获取或者设置文字内容；`HTML`将会被排除。
- `.attr()` – 获取或者设置提供的属性。
- `.width()` – 获取或者设置选择器第一个元素的像素宽度，并转化为数字。
- `.height()` – 获取或者设置选择器第一个元素的像素高度，并转化为数字。
- `.position()` – 获取选择器第一个元素相对于第一个祖先的定位信息。这是一个`getter`。
- `.val()` – 获取或设置一个表单元素的值。
改变元素的信息是很简单的，但是注意这会影响选择器内所有的元素。如果你只是想要改变一个元素。确保在调用`setter`之前指定选择器内的特定元素。

```
// Changing the HTML of an element.
$( "#myDiv p:first" ).html( "New <strong>first</strong> paragraph!" );
```
### 移动、复制和删除元素
索然有非常多的方式在`DOM`中移动元素，但是有两个常用的方式：
- 将选择的元素相对于其他元素放置。
- 将一个元素放置到选择的元素附近。
比如，`jQuery`提供了`.insertAfter()`和`.after()`方法。`.insertAfter()`方法可以将选择的元素放置到参数提供的元素之后。`.after()`可以可以放置元素到提供的参数的元素的后面。其他方法遵循一样的规范：`.insertBefore()`和`.before()`，`.appendTo()`、`.append()`、`.prependTo()`和`.prepend()`。

这些方法的意义取决于你选择了什么元素和是否需要存储你要添加到页面的元素。如果你需要存储一个引用，你通常需要使用第一种方式-将选中元素放置到其他元素附近-它返回你放置的元素。在这种情况下，`.insertAfter()`，`.insertBefore()`，`.appendTo()`和`.prependTo()`也是可以用选择的工具。
```
// Moving elements using different approaches.
 
// Make the first list item the last list item:
var li = $( "#myList li:first" ).appendTo( "#myList" );
 
// Another approach to the same problem:
$( "#myList" ).append( $( "#myList li:first" ) );
 
// Note that there's no way to access the list item
// that we moved, as this returns the list itself.
```

### 克隆元素
像`.appendTo()`这样的方法移动了元素，但是有时候需要复制一份元素。在这种情况下，首先使用`.close（）`：
```
// Making a copy of an element.
 
// Copy the first list item to the end of the list:
$( "#myList li:first" ).clone().appendTo( "#myList" );
```
如果你需要复制相关的数据和事件，确保传递`true`作为`.clone()`的参数。

### 移除元素

有两种方式可以从页面中移除元素：`.remove()`和`.detach()`。当你想要永久的从页面移除选择器的元素的时候，使用`.remove()`。`.remove()`确实返回被移除的元素，但是如果你将这些元素再添加到页面上，关联的数据和事件已不复存在。

如果你需要将数据和事件持久化，可以使用`.detach()`。就像`.remove()`，它返回选择器，但是依旧保持关联的数据和事件，所以你可以稍候在这个页面恢复这个选择器。

如果你需要对元素进行大量的操作，`.detach()`方法非常有用。在这种情况下，使用`.detach()`将元素从页面移除，然后通过代码操作，完成之后将它恢复到页面是非常有益的。抑制了昂贵的`DOM`操作，同时保留了数据和事件。

如果你想要保留元素但是移除内容，可以使用`.empty()`处理元素的内部`HTML`。

### 创建新的元素
`jQuery`提供了简单且优雅的方式去创建一个新的元素，使用相同的`$()`方法去创建选择器
```
// Creating new elements from an HTML string.
$( "<p>This is a new paragraph</p>" );
$( "<li class=\"new\">new list item</li>" );
```
```
// Creating a new element with an attribute object.
$( "<a/>", {
    html: "This is a <strong>new</strong> link",
    "class": "new",
    href: "foo.html"
});
```
注意上面第二个参数的属性对象，`class`属性名称是用引号包起来的，但是`html`和`href`却没有。属性名称通常不需要用引号包裹，除非他们是关键字（就像这个栗子中的`class`）。

当你创建一个新的元素，他不是立即添加到页面。当一个元素创建后，有很多个方式可以添加一个元素到页面。

```
// Getting a new element on to the page.
 
var myNewElement = $( "<p>New element</p>" );
 
myNewElement.appendTo( "#content" );
 
myNewElement.insertAfter( "ul:last" ); // This will remove the p from #content!
 
$( "ul" ).last().after( myNewElement.clone() ); // Clone the p so now we have two.
```
创建的元素不需要保存到一个变量中-你可以直接在`$()`后调用方法将元素添加到页面。然后大部分情况下，你最好将要添加的元素引用保存下来，免得在之后还要再选择它。
你也可以创建一个元素就像你添加它到页面一样，但是注意这种情况你没办法得到新创建元素的引用。

```
// Creating and adding an element to the page at the same time.
$( "ul" ).append( "<li>list item</li>" );
```
添加一个新元素到页面的语法非常简单，所以很容易忘记重复添加`DOM`有很严重的性能损耗。如果你添加很多元素到相同的容器，你最好将所有的`HTML`链接成单一的字符串，然后将字符串添加到容器中，而不是一次添加一个元素。使用数组收集所有的片段，将他们合并到单一的字符串，然后做拼接操作：
```
var myItems = [];
var myList = $( "#myList" );
 
for ( var i = 0; i < 100; i++ ) {
    myItems.push( "<li>item " + i + "</li>" );
}
 
myList.append( myItems.join( "" ) );
```
### 操作属性
`jQuery`属性操作能力非常强。`.attr()`方法的基本操作很简单，但是它也支持很复杂的操作。它可以设置显示值，也可以使用函数返回值设置值。当使用函数语法的时候，函数接受两个参数：要被改变属性的元素的0开始的元素索引，和要被该拜年的属性当前的值。
```
// Manipulating a single attribute.
$( "#myDiv a:first" ).attr( "href", "newDestination.html" );
```
```
// Manipulating multiple attributes.
$( "#myDiv a:first" ).attr({
    href: "newDestination.html",
    rel: "nofollow"
});
```
```
// Using a function to determine an attribute's new value.
$( "#myDiv a:first" ).attr({
    rel: "nofollow",
    href: function( idx, href ) {
        return "/new/" + href;
    }
});
 
$( "#myDiv a:first" ).attr( "href", function( idx, href ) {
    return "/new/" + href;
});
```