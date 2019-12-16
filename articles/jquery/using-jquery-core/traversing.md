### 遍历
一旦你使用`jQuery`创建了一个选择器，你就可以深入遍历所选中的。遍历可以分为3个基本部分：祖先，子孙，兄弟。`jQuery`在这部分提供了大量使用简单的方法。值得注意的是，这些方法都可以提供一个可选的选择器表达式，还有一些可以接受其他`jQuery`对象去过滤选择器。注意查阅[遍历相关的API文档]()查找你可以使用的参数。
### 祖先

从一个选择器茶轩它的祖先的方法包括`.parent()`、`.parents()`、`.parentsUtil()`、`.closet()`。
```
<div class="grandparent">
    <div class="parent">
        <div class="child">
            <span class="subchild"></span>
        </div>
    </div>
    <div class="surrogateParent1"></div>
    <div class="surrogateParent2"></div>
</div>
```
```
// Selecting an element's direct parent:
 
// returns [ div.child ]
$( "span.subchild" ).parent();
 
// Selecting all the parents of an element that match a given selector:
 
// returns [ div.parent ]
$( "span.subchild" ).parents( "div.parent" );
 
// returns [ div.child, div.parent, div.grandparent ]
$( "span.subchild" ).parents();
 
// Selecting all the parents of an element up to, but *not including* the selector:
 
// returns [ div.child, div.parent ]
$( "span.subchild" ).parentsUntil( "div.grandparent" );
 
// Selecting the closest parent, note that only one parent will be selected
// and that the initial element itself is included in the search:
 
// returns [ div.child ]
$( "span.subchild" ).closest( "div" );
 
// returns [ div.child ] as the selector is also included in the search:
$( "div.child" ).closest( "div" );
```
### 子孙
从一个选择器查找他的子孙节点的方法包括`.children()`、`.find()`。这两个方法的区别在他们子孙节点结构的深度。`.children()`只操作他们的直接子孙节点，`.find()`可以递归遍历子孙节点、子孙节点的子孙节点...
1
2
3
4
5
6
7
8
9
```
// Selecting an element's direct children:
 
// returns [ div.parent, div.surrogateParent1, div.surrogateParent2 ]
$( "div.grandparent" ).children( "div" );
 
// Finding all elements within a selection that match the selector:
 
// returns [ div.child, div.parent, div.surrogateParent1, div.surrogateParent2 ]
$( "div.grandparent" ).find( "div" );
```
### 兄弟节点
`jQuery`中剩下的遍历方法都是处理兄弟节点的。就遍历方向而言，只有几个基本的方法需要关心。可以使用`.prev()`查找前面的元素，可以使用`.next()`，可以使用`.silbings()`查找前面和后面。当然还有一些其他方法组成了基本的方法：`.nextAll()`、`.nextUtil()`、`.prevAll()`、`.prevUtil()`。
```
// Selecting a next sibling of the selectors:
 
// returns [ div.surrogateParent1 ]
$( "div.parent" ).next();
 
// Selecting a prev sibling of the selectors:
 
// returns [] as No sibling exists before div.parent
$( "div.parent" ).prev();
 
// Selecting all the next siblings of the selector:
 
// returns [ div.surrogateParent1, div.surrogateParent2 ]
$( "div.parent" ).nextAll();
 
// returns [ div.surrogateParent1 ]
$( "div.parent" ).nextAll().first();
 
// returns [ div.surrogateParent2 ]
$( "div.parent" ).nextAll().last();
 
// Selecting all the previous siblings of the selector:
 
// returns [ div.surrogateParent1, div.parent ]
$( "div.surrogateParent2" ).prevAll();
 
// returns [ div.surrogateParent1 ]
$( "div.surrogateParent2" ).prevAll().first();
 
// returns [ div.parent ]
$( "div.surrogateParent2" ).prevAll().last();
```
使用`.siblings()`选择所有的兄弟节点：
```
// Selecting an element's siblings in both directions that matches the given selector:
 
// returns [ div.surrogateParent1, div.surrogateParent2 ]
$( "div.parent" ).siblings();
 
// returns [ div.parent, div.surrogateParent2 ]
$( "div.surrogateParent1" ).siblings();
```
去[api.jquery.com 关于遍历]()查看关于这些方法完整的文档
当在文档中做长距离遍历的时候要注意-复杂的遍历需要保证文档结构不变，就算你是创建了从服务端到客户端完整应用的人也很难保证。一到两步的的遍历比较适合，最好避免从一个容器到另一个容器的遍历。
