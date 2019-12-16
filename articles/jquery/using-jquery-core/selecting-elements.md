### 选择元素

`jQuery`最基本的概念就是“选择一些元素并且对他们做一些操作”。`jQuery`支持大部分的`CSS3`选择器以及一些非标准的选择器。可以访问[api.jquery.com 中关于选择器的章节]()查看完整的选择器索引。

### 根据`ID`选择元素
```
$( "#myId" ); // Note IDs must be unique per page.
```
### 根据`class`选择元素
```
$( ".myClass" );
```
### 根据属性选择元素
```
$( "input[name='first_name']" );
```
### 根据复合`CSS`选择器选择元素
```
$( "#contents ul.people li" );
```
### 根据逗号分隔的选择器列表选择元素
```
$( "div.myClass, ul.people" );
```
### 伪元素选择器
```
$( "a.external:first" );
$( "tr:odd" );
 
// Select all input-like elements in a form (more on this below).
$( "#myForm :input" );
$( "div:visible" );
 
// All except the first three divs.
$( "div:gt(2)" );
 
// All currently animated divs.
$( "div:animated" );
```
> 注意：当使用`:visible`和`:hidden`伪元素选择器的时候，`jQuery`测试元素的真实可见行，而不是`CSS`的`visibility`或者`display`属性。`jQuery`查找物理的高度和宽度是否都大于0。

然而，这种验证对`<tr>`元素不生效。对于`<tr>`，`jQuery`会检查它`CSS`的`display`属性，如果`display`属性是`none`，则判定该元素是隐藏的。

没有添加到`DOM`的元素也被认为是隐藏的，即使影响他们的`CSS`会将他们渲染为可见。查看[元素操作]()章节学习如何创建和添加元素到`DOM`。

### 选择选择器
选择一个好的选择器是提高`JavaScript`性能的一个好方法。指定太多可能会是一件坏事情。如果`#myTable th.special`就可以完成工作，那么`#myTable thead tr th.special`就是太过了。

### 我的选择器选中元素了吗？
一旦你生成了一个选择器，你通常会想知道你能否对他们做一些操作。一个普通的错误用法：
```
// Doesn't work!
if ( $( "div.foo" ) ) {
    ...
}
```
这将不会共奏。使用`$()`创建一个选择器总是会返回一个对象，对象判定总是`true`。尽管选择器没有包含任何元素，包含在`if`语句中的代码依旧会执行。
测试一个选择器是否包含元素最好的方式是通过`.length`属性，它会告诉你有多少元素被选中。如果返回`0`，`.length`作为`boolean`使用的时候将会被判定为`false`。

```
// Testing whether a selection contains elements.
if ( $( "div.foo" ).length ) {
    ...
}
```
### 保存选择器
`jQuery`不会为你缓存元素。如果你创建了一个选择器，并且你可能需要再次创建，最好将这个选择器保存到变量中，而不是重复创建。
```
var divs = $( "div" );
```
一旦你将选择器保存到变量总，你总是可以在变量上调用`jQuery`方法，就像你在原生选择器上面调用那样。

一个选择器只会在选择器创建的时候查询页面上的元素。如果元素是在之后添加到页面的，你必须重新创建选择，或者将他们添加到存储选择器的变量中。被存储的选择器不会在`DOM`变化的时候神奇的更新。

### 提取和过滤选择器
有时候选择器包含的不仅仅是你想要的。`jQuery`提供了一系列的方法用来提取和过滤选择器。

```
// Refining selections.
$( "div.foo" ).has( "p" );         // div.foo elements that contain <p> tags
$( "h1" ).not( ".bar" );           // h1 elements that don't have a class of bar
$( "ul li" ).filter( ".current" ); // unordered list items with class of current
$( "ul li" ).first();              // just the first unordered list item
$( "ul li" ).eq( 5 );              // the sixth
```

### 选择表单元素
`jQuery`提供了一系列的伪元素选择器去帮助我们在表单中找到元素。这些对我们的帮助特别大，因为使用标准的`CSS`选择器很难格局表单元素的状态和类型区分。

- `:checked`

不要和`:checkbox`混淆，`:cheked`目标是所有选中的`checkbox`，但是要注意，这个选择器对选中的`radio`和`<select>`元素都有效（`:selected`选择器只对`<select>`元素有效）
```
$( "form :checked" );
```
`:checked`伪元素对`checkbox`、`radio`和`select`都有效。

- `:disabled`

使用`:disabled`伪元素目标是所有带有`disabled`属性的`<input>`元素。
```
$( "form :disabled" );
```
为了获得更好的性能，使用`:disabled`的时候，首选通过标准的`jQuery`选择器选择元素，然后使用`.filter(':disabled')`，也可以使用标签名或者其他选择器和伪元素选择器一起使用。

- `:enabled`

单纯的和`:disabled`伪元素选择器相反，`:enabled`伪元素选择器指向任意没有`disabled`属性的元素：
```
$( "form :enabled" );
```
为了获得更好的性能，使用`:disabled`的时候，首选通过标准的`jQuery`选择器选择元素，然后使用`.filter(':enabled')`，也可以使用标签名或者其他选择器和伪元素选择器一起使用。

- `:input`

使用`:input`选择器可以选择所有的`<input>`、`<textarea>`、`<select>`、`<button>`。
```
$( "form :input" );
```

- `:selected`

使用`:selected`伪元素选择器指向任意选中的`<option>`元素
```
$( "form :selected" );
```
为了获得更好的性能，使用`:disabled`的时候，首选通过标准的`jQuery`选择器选择元素，然后使用`.filter(':selected')`，也可以使用标签名或者其他选择器和伪元素选择器一起使用。

- 根据类型选择

`jQuery`提供了根据类型来选择特定的表单元素的伪元素选择器：
- `:password`
- `:reset`
- `:radio`
- `:text`
- `:submit`
- `:checkbox`
- `:button`
- `:image`
- `:file`
对于以上这些，都有关于性能的附加说明，请确保去查看`API`文档获取更多深入的信息。