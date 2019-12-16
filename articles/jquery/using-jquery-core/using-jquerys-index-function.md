### 使用`jQuery`的`.index()`方法

`.index()`是`jQuery`对象的一个方法，通常用来搜索调用该方法的`jQuery`对象中指定的元素。这个方法有四种不同的签名，每一种都有令人疑惑的不同的语义。这篇文章尝试说明`.index()`方法的每一种方法。

### 无参数的`.index()`
```
<ul>
    <div></div>
    <li id="foo1">foo</li>
    <li id="bar1">bar</li>
    <li id="baz1">baz</li>
    <div></div>
</ul>
```
```
var foo = $( "#foo1" );
 
console.log( "Index: " + foo.index() ); // 1
 
var listItem = $( "li" );
 
// This implicitly calls .first()
console.log( "Index: " + listItem.index() ); // 1
console.log( "Index: " + listItem.first().index() ); // 1
 
var div = $( "div" );
 
// This implicitly calls .first()
console.log( "Index: " + div.index() ); // 0
console.log( "Index: " + div.first().index() ); // 0
```
 在第一个栗子中，`.index()`返回`#foo1`在父元素中以0为技术的顺序。以为`#foo1`是其父元素的第二个子元素，所以`index()`返回1。

注意：在`jQuery`1.9之前，`.index()`只在一个元素上工作的时候是可以心来的，这就是为什么我们在每一个例子中使用`.first()`。在`jQuery`1.9+，这可以被忽略，这个`API`被更新为只能工作在第一个元素上。
### 字符串参数的`.index()`
```
<ul>
    <div class="test"></div>
    <li id="foo1">foo</li>
    <li id="bar1" class="test">bar</li>
    <li id="baz1">baz</li>
    <div class="test"></div>
</ul>
<div id="last"></div>
```
```
var foo = $( "li" );
 
// This implicitly calls .first()
console.log( "Index: " + foo.index( "li" ) ); // 0
console.log( "Index: " + foo.first().index( "li" ) ); // 0
 
var baz = $( "#baz1" );
console.log( "Index: " + baz.index( "li" )); // 2
 
var listItem = $( "#bar1" );
console.log( "Index: " + listItem.index( ".test" ) ); // 1
 
var div = $( "#last" );
console.log( "Index: " + div.index( "div" ) ); // 2
```
当`.index()`使用字符串参数调用时，有两件事需要考虑。第一，`jQuery`将会在`jQuery`对象内部自动调用`.first()`方法。在这个场景中，它将会寻找第一个元素而不是最后一个元素的索引。这是易变的，所以要非常小心。

第二点需要注意的是`jQuery`使用传递的字符串选择器查询整个`DOM`，并在新的查询出来的`jQuery`对象上检查元素的索引。比如，在上面的栗子中，当使用`.index("div")`的时候，`jQuery`选中了文档中的所有的`<div>`元素。然后在`jQuery`对象查找第一个元素的索引。
### 对象参数调用`.index()`
```
<ul>
    <div class="test"></div>
    <li id="foo1">foo</li>
    <li id="bar1" class="test">bar</li>
    <li id="baz1">baz</li>
    <div class="test"></div>
</ul>
<div id="last"></div>
```
```
var foo = $( "li" );
var baz = $( "#baz1" );
 
console.log( "Index: " + foo.index( baz ) ); // 2
 
var tests = $( ".test" );
var bar = $( "#bar1" );
 
// Implicitly calls .first() on the argument.
console.log( "Index: " + tests.index( bar ) ); // 1
 
console.log( "Index: " + tests.index( bar.first() ) ); // 1
```
在这个场景中，传递到`.index()`的`jQuery`对象的第一个元素将和所有原始的`jQuery`对象对比。在`.index()`左边的原始的`jQuery`对象是类数组的，将会从索引0开始搜索参数的`jQuery`对象的第一个元素，一直到`length-1`。

### `DOM`元素作为`.index()`的参数
在这个场景中， 被传递到`.index()`的`DOM`元素将被和原始`jQuery`对象所有元素比较。一旦所有其他的场景都理解了，这将会是一个简单的栗子。这个和前一个场景很像，除了`DOM`元素是被直接传递的，并不是从`jQuery`对象容器中获取的。