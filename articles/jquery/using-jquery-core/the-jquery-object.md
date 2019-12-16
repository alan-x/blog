### jQuery 对象
当创建一个新的元素的时候（或者选择一个存在的元素），`jQuery`返回一个包含元素的集合。很多开发者以为这个集合就是一个数组。它有着以0为开始的索引的`DOM`元素序列，一些类似数组的方法，和一个`.length`属性。实际上，`jQuery`对象比这更加复杂。

### `DOM`和`DOM`元素
文档对象模型（简称`DOM`），是`HTML`文档的表示。它可能包含大量的`DOM`元素。从高层来看，一个`DOM`元素可以看作是网页的一个片段。它可能包含文字和/或其他`DOM`元素。`DOM`元素可以概括为一个类型，比如`<div>`、`<a>`、`<p>`，和任意数量的属性，比如`src`、`href`、`class`等。更多相关的描述，可以从W3C查阅[DOM 规范]()

元素也有属性，就像任意`JavaScript`对象。在这些属性中，有一些属性，比如`.tagName`和方法，比如`.appendChild()`。这些属性是`JavaScript`和网页交互的唯一方式。

### `jQuery`对象
事实证明，直接使用`DOM`元素会很棘手。`jQuery`对象定义了很多方法，让开发者的体验更加的顺滑。`jQuery`对象的好处包括：

- 兼容-元素方法的实现在哥哥浏览器提供商和版本之间各有差异。下面的代码片段尝试设置保存在`target`中的`<tr>`元素的内部`HTMl`，

```
var target = document.getElementById( "target" );
 
target.innerHTML = "<td>Hello <b>World</b>!</td>";
```
在大部分情况下，它是可以工作了，但是它将在大部分的`IE`版本中失效。在这种情况下，推荐使用纯粹的`DOM`方法代替。通过将目标元素包裹在`jQuery`对象中。这边边缘情况都将被处理，并在所有支持的浏览器中，展现出预期的结果：
```
// Setting the inner HTML with jQuery.
 
var target = document.getElementById( "target" );
 
$( target ).html( "<td>Hello <b>World</b>!</td>" );
```
便捷-还有很多常见的`DOM`操作使用场景使用纯`DOM`方法很难完成。比如，将存储在`newElement`的元素插入目标元素需要大量的`DOM`方法：
```
// Inserting a new element after another with the native DOM API.
 
var target = document.getElementById( "target" );
 
var newElement = document.createElement( "div" );
 
target.parentNode.insertBefore( newElement, target.nextSibling );
```
通过包裹目标元素到`jQuery`对象，同样的任务变得非常简单：
```
// Inserting a new element after another with jQuery.
 
var target = document.getElementById( "target" );
 
var newElement = document.createElement( "div" );
 
$( target ).after( newElement );
```
大部分情况下，这些细节只是横亘在你和目标之间的陷阱。
### 将元素倒入`jQuery`对象
当`jQuery`方法通过`CSS`选择器被调用的时候，它将返回一个`jQuery`对象，包裹一定数量满足选择器的元素
```
// Selecting all <h1> tags.
 
var headings = $( "h1" );
```
`heading`现在是一个`jQuery`元素，它包含页面上所有的`<h1>`标签。这个可以通过审查`heading`的`.length`属性来验证：
1
2
3
4
5
```
// Viewing the number of <h1> tags on the page.
 
var headings = $( "h1" );
 
alert( headings.length );
```
如果页面删更有不止一个`<h1>`标签，那么数量就会大于1，如果页面上没有`<h1>`标签，`.length`属性将会是0，检查`.length`属性是确保选择器成功命中一个或者更多个元素非常普通的方式。
如果目标只是获取第一个元素，需要做一些其他操作。有很多种方式可以完成，但是最直接的方式是调用`.eq`方法。
1
2
3
4
5
```
// Selecting only the first <h1> element on the page (in a jQuery object)
 
var headings = $( "h1" );
 
var firstHeading = headings.eq( 0 );
```
现在，`firstHeading`是一个包含第一个`<h1>`元素的`jQuery`对象。因为他是一个`jQuery`对象，所以它包含像`.html()`、`.after()`之类有用的方法。`jQuery`当然有一个叫做`.get(）`的方法提供相应的函数。它返回`DOM`元素自身，替代返回`jQuery`包裹的`DOM`元素。
```
// Selecting only the first <h1> element on the page.
 
var firstHeadingElem = $( "h1" ).get( 0 );
```
此外，因为`jQuery`对象是类数组的，它支持通过括号访问数组：
```
// Selecting only the first <h1> element on the page (alternate approach).
 
var firstHeadingElem = $( "h1" )[ 0 ];
```
不管那种情况，`firstHeadingEle`包含原生的`DOM`元素。这意味着它是拥有像`.innerHTML`的`DOM`属性和像`.appendChild()`的方法，但是没有像`.html()`和`.after()`之类的`jQuery`方法。`firstHeadingElem`元素更加难以操作，但是它的确是某些实例所需要的。有这样的一个实例更容易进行比较。
### 并不是所有的`jQuery`都想都被创建了

关于这个`包裹`行为一个很重要的细节是每一个包装对象都是唯一的。就算对象是由同一个选择器或者相同引用的`DOM`元素创建的，也是如此。

1
2
3
4
```
// Creating two jQuery objects for the same element.
 
var logo1 = $( "#logo" );
var logo2 = $( "#logo" );
```
尽管`logo1`和`logo2`是通过相同方式创建的（包裹相同的`DOM`元素），他们依旧不是相同的对象。比如：
```
// Comparing jQuery objects.
 
alert( $( "#logo" ) === $( "#logo" ) ); // alerts "false"
```
但是，两个对象都宝航相同的`DOM`元素。`.get()`方法可以用来测试两个`jQuery`对象是否拥有相同的`DOM`元素:
```
// Comparing DOM elements.
 
var logo1 = $( "#logo" );
var logo1Elem = logo1.get( 0 );
 
var logo2 = $( "#logo" );
var logo2Elem = logo2.get( 0 );
 
alert( logo1Elem === logo2Elem ); // alerts "true"
```
为了区分`jQuery`对象，很多开发者在变量名前添加`$`符号。这个行为没有任何的魔法-只是帮助人们跟踪变量到底包含了什么。上面那个栗子可以被重写为下面的方式：
```
// Comparing DOM elements (with more readable variable names).
 
var $logo1 = $( "#logo" );
var logo1 = $logo1.get( 0 );
 
var $logo2 = $( "#logo" );
var logo2 = $logo2.get( 0 );
 
alert( logo1 === logo2 ); // alerts "true"
```

这段代码的功能和上面的栗子没有区别，但是更加清晰和易读。

无论使用哪种命名方式，区分`jQuery`对象和原生`DOM`元素是非常重要的。原生`DOM`方法和属性在`jQuery`对象上并不存在，反之亦然。像`event.target.closet is not a function`和`TypeError: Object [object Object] has no method 'setAttribute'`这种错误消息指示出这种普通错误的存在。
### `jQuery`对象不是`活的`
将页面上所有的`p`元素放到一个`jQuery`对象中：
1
2
3
```
// Selecting all <p> elements on the page.
 
var allParagraphs = $( "p" );
```
人们或许期待元素的内容将会随着`<p>`元素的从页面中添加、移除而增加、减少，但是`jQuery`对象不会以这种方式运行。`jQuery`对象内包含的元素不会改变，除非明确的修改。这意味的集合不是`活的`-当文档变化的时候它不会自动更新。如果文档在创建`jQuery`对象后更新了，集合应该像创建一个新的一样更新。它就像重新运行相同的选择器一样简单。
1
2
3
```
// Updating the selection.
 
allParagraphs = $( "p" );
```
### 包裹
尽管`DOM`元素提供了所有和网页交互需要的基础设施，但是他们可能很麻烦。`jQuery`对象包裹这些元素让体验更加顺滑，让普通任务更加简单。当使用`jQuery`创建或者选择元素的时候，总是会被包裹到一个新的`jQuery`对象。如果需要原声的`DOM`元素，可以通过`.get()`方法和/或数组风格访问。