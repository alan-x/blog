### jQuery 是怎样工作的
#### jQuery：基础
这是一个基础指南，用来帮你入门`jQuery`，如果你还没有一个测试页面，请从创建下面的`HTML`页面开始：

```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo</title>
</head>
<body>
    <a href="http://jquery.com/">jQuery</a>
    <script src="jquery.js"></script>
    <script>
 
    // Your code goes here.
 
    </script>
</body>
</html>
```


`<script>`的`src`属性必须指向`jQuery`文件，可以从[jQuery 下载页面]()下载`jQuery`，保存成`jquery.js`，并放置在`HTML`文件的相同目录下。




> 注意：当你下载`jQuery`的时候，文件名包含版本号，比如：`jquery-x.y.z.js`。确保文件被重命名为`jquery.js`，或者更新`<script>`标签的`src`属性为文件名

### 在`Document Ready`的时候执行代码


为了确保他们的代码在浏览器加载完`document`的时候才执行，很多`JavaScript`工程师会将他们的代码包含在`onload`中：

```
window.onload = function() {
 
    alert( "welcome" );
 
};
```


不幸的是，这些代码必须要在所有图片都加载完的时候才执行，包括`banner`广告图。为了在`document`一加载完就执行代码，`jQuery`准备了[ready 事件]()
```
$( document ).ready(function() {
 
    // Your code here.
 
});
```
> 注意：`jQuery`框架通过`window`对象的两个属性暴露它的方法和属性，一个是`jQuery`，一个是`$`。`$`是`jQuery`的简写，它经常被使用，因为它很简短并且写起来很快。

例如，在`ready`事件的内部，你可以为链接添加点击事件：


```
$( document ).ready(function() {
 
    $( "a" ).click(function( event ) {
 
        alert( "Thanks for visiting!" );
 
    });
 
});
```



复制上面的`jQuery`代码到你的`HTML`文件中`// Your code goes here`所在的位置。然后，保存你的`HTML`文件并刷新你的浏览器。点击链接，首先会出现一个弹窗，然后将会跳转到`http://jquery.com`-这是链接默认的行为。


对于点击事件和其他事件来说，在事件监听中调用`event.preventDefault()`来阻止默认的行为：

```
$( document ).ready(function() {
 
    $( "a" ).click(function( event ) {
 
        alert( "As you can see, the link no longer took you to jquery.com" );
 
        event.preventDefault();
 
    });
 
});
```

尝试替换你的第一个`jQuery`代码片段，就是前一个粘贴到`HTML`文件中的那个。再次保存，刷新浏览器并再试一次。


#### 完整的栗子 🌰

下面的栗子说明前面讨论的点击事件的处理代码，它直接嵌入到了`HTML`的`<body>`中。请注意，在实践中，将代码放置在单独的`js`文件中，并通过`<script>`元素的`src`属性引入更加优雅。


```
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Demo</title>
</head>
<body>
    <a href="http://jquery.com/">jQuery</a>
    <script src="jquery.js"></script>
    <script>
 
    $( document ).ready(function() {
        $( "a" ).click(function( event ) {
            alert( "The link will no longer take you to jquery.com" );
            event.preventDefault();
        });
    });
 
    </script>
</body>
</html>
```



#### 添加和移除`HTML`的`Class`
> 重要：你必须将下面的`jQuery`栗子放置在`ready`事件中，这样的你代码才会在`document`准备好的时候就工作。


另一个普通的任务是添加/移除一个`class`。

首先，添加一些样式信息到`document`的`<head>`中，就像这样：
```
<style>
a.test {
    font-weight: bold;
}
</style>
```
然后，在脚本中添加`.addClass()`的调用
```
$( "a" ).addClass( "test" );
```
所有的`<a>`元素现在都是粗体了。

为了移除存在的`class`，可以使用`.removeClass()`:
```
$( "a" ).removeClass( "test" );
```

#### 特殊的效果
`jQuery`海提供了一些方便的效果，帮助你让你的网站更加出众。比如如果你创建了一个如下点击处理器：
```
$( "a" ).click(function( event ) {
 
    event.preventDefault();
 
    $( this ).hide( "slow" );
 
});
```
当点击的时候，链接将会慢慢消失。

### 回调和函数

不像其他编程语言，`JavaScript`允许你自由的传递函数，并在之后执行。一个回调就是一个函数，它通过参数船体给其他函数，并在它的父函数执行完成后才执行。回调是很特别的，因为他们耐心的等待他们父函数执行完毕。同时，浏览器可以执行其他函数或者做其他一系列的工作。


为了使用回调，知道如何传递到他们的父函数很重要。


### 没有参数的回调
如果一个回调没有参数，你可以像这样传递：
```
$.get( "myhtmlpage.html", myCallBack );
```
当`$.get()`完成对`myhtmlpage.html`页面的获取的时候，它就会执行`myCallback()`函数。

> 注意：第二个参数是简单的函数名（但是不是一个字符串，并且没有圆括号）

### 有参数的回调


执行有参数的回调比较棘手：

#### 错误
以下栗子将不会工作
```
$.get( "myhtmlpage.html", myCallBack( param1, param2 ) );
```
执行失败的原因是因为这段代码立即执行了`myCallBack( param1, param2 )`并将执行的结果作为`$.get()`的第二个参数。我们想要的是将`myCallBack()`作为参数，而不是`myCallBack( param1, param2 )`的结果（它可能是函数，也可能不是）。所以，应该如何传递`myCallBack()`并包含它的参数？


#### 正确
为了延迟执行带参数的`myCallBack()`,你可以用一个匿名函数包裹。注意使用`function(){`。这个匿名函数只做一件事：使用`param1`和`param2`作为参数调用`myCallBack()`。
```
$.get( "myhtmlpage.html", function() {
 
    myCallBack( param1, param2 );
 
});
```
当`$.get()`完成对`myhtmlpage.html`页面的获取的时候，它会执行那个匿名函数，而这个匿名函数内部则执行`myCallBack(param1, param2)`