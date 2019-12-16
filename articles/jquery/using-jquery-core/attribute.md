### 属性
对你的应用来说，一个元素属性可以包含非常有用的信息，所以可以获取和设置它们是非常重要的。

### `.attr()`方法

`.attr()`方法可以表现为`getter`和`setter`。作为一个`setter`的时候，`.attr()`可以接受一个`key`和`value`，也可以接受一个包含一个或者多个键值对的对象。

`.attr()`作为一个`setter`：

```
$( "a" ).attr( "href", "allMyHrefsAreTheSameNow.html" );
 
$( "a" ).attr({
    title: "all titles are the same too!",
    href: "somethingNew.html"
});
```
`.attr()`作为一个`getter`：
```
$( "a" ).attr( "href" ); // Returns the href for the first a element in the document
```
