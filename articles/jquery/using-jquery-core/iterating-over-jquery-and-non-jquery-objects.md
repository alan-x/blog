### 遍历`jQuery`和非`jQuery`对象
`jQuery`提供了一个对象迭代工具，叫做`$.each()`，同时也提供了一个`jQuery`集合迭代器：`.each()`。两者不可交换。此外，还提供了一对有帮助的方法叫做`$.map()`和`.map()`，可以简化我们普通的迭代场景。

### `$.each()`
`$.each()`是一个通用的迭代器函数，可以遍历对象、数组、类数组对象。简单对象通过对象属性的名字来遍历，数组和类数组通过下标索引来遍历。
`$.each()`其实是传统的`for`或者`for-in`循环的替换。给定：

```
var sum = 0;
 
var arr = [ 1, 2, 3, 4, 5 ];
```
然后：
```
for ( var i = 0, l = arr.length; i < l; i++ ) {
    sum += arr[ i ];
}
 
console.log( sum ); // 15
```
可以替换为：
```
$.each( arr, function( index, value ){
    sum += value;
});
 
console.log( sum ); // 15
```
注意，我们没有通过`arr[index]`访问，因为值已经很放方便的被传递给`$.each()`的回调函数

此外，给定：
```
var sum = 0;
var obj = {
    foo: 1,
    bar: 2
}
```
然后：
```
for (var item in obj) {
    sum += obj[ item ];
}
 
console.log( sum ); // 3
```
可以替换为：
```
$.each( obj, function( key, value ) {
    sum += value;
});
 
console.log( sum ); // 3
```
再一次，我们没有通过`obj[key]`访问，因为值已经直接传递给回调函数。
注意`$.each()`是用来遍历简单对象、数组、类数组对象的，而不是用来遍历`jQuery`集合的。
这将被视为错误：
```
// Incorrect:
$.each( $( "p" ), function() {
    // Do something
});
```
对于`jQuery`结婚，使用`.each()`。
### `.each()`
`.each()`可以直接在`jquery`集合上使用。它遍历所有集合中命中的元素，并在每个对象像调用回调函数。当前元素在集合中的索引被作为参数传递给回调函数。值（当前场景表示`DOM`元素）当然也被传递了，但是因为回调是在当前命中的元素的上下文中执行的，所以`this`关键字被指向当前元素，就像其他`jQuery`回调所期待的那样。

比如，给定下面的标签
```
<ul>
    <li><a href="#">Link 1</a></li>
    <li><a href="#">Link 2</a></li>
    <li><a href="#">Link 3</a></li>
</ul>
```
`.each()`可能被这么使用：
```
$( "li" ).each( function( index, element ){
    console.log( $( this ).text() );
});
// Logs the following:
// Link 1
// Link 2
// Link 3
 ```

### 第二个参数
有一个问题经常被提起，“如果`this`指的是元素，为什么第二个参数要传递给回调？”
无论有意或者无意，执行上下文可能被该拜年。当考虑使用关键字`this`的时候，我们或者其他开发者阅读并不容易被混淆。即使执行上下文一直保持不变，使用第二个参数作为一个具名参数将会更具有可读性。比如
```
$( "li" ).each( function( index, listItem ) {
 
    this === listItem; // true
 
    // For example only. You probably shouldn't call $.ajax() in a loop.
    $.ajax({
        success: function( data ) {
            // The context has changed.
            // The "this" keyword no longer refers to listItem.
            this !== listItem; // true
        }
    });
 
});
```
### 有时候不需要`.each()`

很多方法隐式迭代整个集合，将行为应用到每一个匹配的元素。比如，这是不必要的：

```
$( "li" ).each( function( index, el ) {
    $( el ).addClass( "newClass" );
});
```
这样就行了：
```
$( "li" ).addClass( "newClass" );
``` 
文档中的每一个`<li>`将会被添加`newClass`类。
另一方 main，一些方法不会遍历整个集合。当我们在设置新值之前需要从一个元素获取信息的时候，就需要`.each()`。
这将不会工作：

```
// Doesn't work:
$( "input" ).val( $( this ).val() + "%" );
 
// .val() does not change the execution context, so this === window
```

相反，下面显示了如果编写：
```
$( "input" ).each( function( i, el ) {
    var elem = $( el );
    elem.val( elem.val() + "%" );
});
```
下面是需要使用`.each()`的方法：
- `.attr() (getter)`
- `.css() (getter)`
- `.data() (getter)`
- `.height() (getter)`
- `.html() (getter)`
- `.innerHeight()`
- `.innerWidth()`
- `.offset() (getter)`
- `.outerHeight()`
- `.outerWidth()`
- `.position()`
- `.prop() (getter)`
- `.scrollLeft() (getter)`
- `.scrollTop() (getter)`
- `.val() (getter)`
- `.width() (getter)`
注意，在大部分场景中，`getter`签名返回`jQuery`集合中第一个元素的结果，但是`setter`作用域整个集合中命中的实体。`.text()`是一个例外，它返回所有命中元素的文字的拼接。
除了值的`setter`，属性、成员、`CSS setters`和`DOM`插入`setter`方法(比如`.text()`和`.html()`)接受匿名回调函数应用到命中的集合中的每一个元素。命中元素在集合中的索引和`getter`签名的结果将会被当作参数传递给回调函数。
For example, these are equivalent:
比如，下面是等价的：
```
$( "input" ).each( function( i, el ) {
    var elem = $( el );
    elem.val( elem.val() + "%" );
});
 
$( "input" ).val(function( index, value ) {
    return value + "%";
});
```
另一点需要注意的是，像`.children()`或者`.parent()`这样的隐式迭代方法将会作用于集合中的每个命中元素，返回所有子元素或者父元素节点的集合。
link.map()
### `.map90`
还有一种常见的迭代应用场景用`.map()`更容易处理。任何时候我们想要在我们的`jQuery`选择器中基于选中的元素创建一个数组或者拼接字符串，我们最好使用`.map()`
For example instead of doing this:
比可代替的做法栗子：
```
var newArr = [];
 
$( "li" ).each( function() {
    newArr.push( this.id );
});
```
我们可以这么做
```
$( "li" ).map( function(index, element) {
    return this.id;
}).get();
```
注意，在`.map()`的结尾有一个`.get()`链式调用。`.map()`实际上返回的是一个`jQuery`包裹的集合，即使我们从回调中返回的是一个字符串。为了返回一个可以为我们所用的基本的`JavaScript`数组，我们需要使用无参数版本的`.get()`。为了拼接为一个字符串，我们可以在`.get()`后面链式调用普通`JS`的`join()`数组方法。
### `$.map`
就像`$.each()`和`.each()`，也存在`$.map()`和`.map()`。他们之间的差别和`.each()`方法之间的差别很像。`$.map()`可以在普通`JavaScript`上工作。`.map()`可以工作在`jQuery`元素集合上。因为工作在普通数组上，`$.array()`返回一个普通数组，所以不需要使用`.get()`-实际上，它会抛出一个错误，因为这不是一个原生的`JavaScript`方法。

警告：`$.map()`交换的回调的参数。为了符合`ECMAScript 5`中可用的原生的`JavaScript`的`.map()`方法。

比如：
```
<li id="a"></li>
<li id="b"></li>
<li id="c"></li>
 
<script>
 
var arr = [{
    id: "a",
    tagName: "li"
}, {
    id: "b",
    tagName: "li"
}, {
    id: "c",
    tagName: "li"
}];
 
// Returns [ "a", "b", "c" ]
$( "li" ).map( function( index, element ) {
    return element.id;
}).get();
 
// Also returns [ "a", "b", "c" ]
// Note that the value comes first with $.map
$.map( arr, function( value, index ) {
    return value.id;
});
 
</script>
```