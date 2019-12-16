### 选择器操作
#### `Getter`和`Setter`
选择器的一些`jQuery`方法既可以赋值，也可以获值。当这些方法调用的时候传入参数，就会被当作`setter`，因为它会设置（赋值）为这个值。当调用方法不传递参数的时候，它会获取（或读取）这个元素的值。`setter`会影响选择器中的所有值，而`getter`只会返回选择器中的第一个值，除了`.text()`，它返回所有元素的值。

```
// The .html() method sets all the h1 elements' html to be "hello world":
$( "h1" ).html( "hello world" );
```
```
// The .html() method returns the html of the first h1 element:
$( "h1" ).html();
// > "hello world"
```
`setter`返回一个`jQuery`对象，允许你在选择器上继续调用`jQuery`方法。`getter`返回他们要求的内容，所以你无法在`getter`返回的值上继续调用`jQuery`方法。
1
2
3
```
// Attempting to call a jQuery method after calling a getter.
// This will NOT work:
$( "h1" ).html().addClass( "test" );
```
#### 链式调用

如果你在选择器上调用一个方法，并且这个方法返回一个`jQuery`对象，你可以在这个对象上继续调用`jQuery`方法，而不需要用分号暂停。这种做法称为链式调用
```
$( "#content" ).find( "h3" ).eq( 2 ).html( "new text for the third h3!" );
```
将链式调用分为多行可以提高代码的可读性：
```
$( "#content" )
    .find( "h3" )
    .eq( 2 )
    .html( "new text for the third h3!" );
```
`jQuery`也提供了`.end()`方法在你链式调用中途改变选择器的时候返回到最原始的选择器。
```
$( "#content" )
    .find( "h3" )
    .eq( 2 )
        .html( "new text for the third h3!" )
        .end() // Restores the selection to all h3s in #content
    .eq( 0 )
        .html( "new text for the first h3!" );
```
链式调用非常强大，它随着`jQuery`流行以后就被很多库接受，并称为他们的特性。然而，必须非常小心的使用--广泛使用链式调用会使得代码非难以修改和调试。没有什么严格且快速的规则去规定链应该要多长--只要知道它很容易被带偏。