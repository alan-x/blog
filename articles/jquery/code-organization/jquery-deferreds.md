### `jQuery`延迟
延迟为`Ajax`模块大规模重写之后的添加的一部分，由 Julian Aubourg 引导，遵循`CommonJS Promise/A`设计。从1.5和以上版本包含了延迟能力，`jQuery.ajax()`早先的版本接收将在请求完成或者失败的时候执行的回调，但是会导致高耦合-同样的原则将会驱使开发者使用其他的语言或者其他工具去操作延迟执行。

通常来说，`jQuery`的版本提供你管理回调的一些列增强，给你更加弹性的方式去提供回调，不管原始的回调分发是否被激活。`jQuery`的延迟对象支持多种回调绑定特别的任务结果（不止一个）也就没啥了，而不管这任务本身是同步还是异步的。

`jQuery`的核心是现实`jQuery.Deferred`-一个链式构造，可以创建新的延迟对象，可以检测可以被观测的承诺是否存在。它也可以执行传递同步和异步成功的回调队列。有一点必须要注意，延迟默认的状态是未解决。可能会通过`.then()`台那集或者`.fail()`的回调是排队的，并且在稍后的流程中执行。
你可以在同时发生的时候使用延迟对象的`when()`概念，`jQuery`的实现是`$.when()`去等待所有的延迟对象请求执行完成（比如，所有的承诺都是被处理的）。从技术来说，`$.when()`是基于多个代表异步事件的承诺执行回调的有效方式，
一个`$.when()`的栗子，接收多个参数可以看作是同时执行，还有`.then()`：
```
function successFunc() {
    console.log( "success!" );
}
 
function failureFunc() {
    console.log( "failure!" );
}
 
$.when(
    $.ajax( "/main.php" ),
    $.ajax( "/modules.php" ),
    $.ajax( "/lists.php" )
).then( successFunc, failureFunc );
```
`jQuery`提供的`$.when()`实现是非常有趣的，因为它不仅仅实现了延迟对象，在传递的参数非延迟对象的时候，它将他们看作已经解决的延迟，并立马执行回调（`doneCallbacks`）。`jQuery`的延迟实现不算什么，此外，还提供了`deferred.then()`，`deferred.done()`，`deferred.fail()`方法，可以用来添加回调到延迟队列。
我们将在在一个栗子中看许多延迟特性。这是一个非常简单的脚本，消耗（1）一个外部喂食的新闻，（2）通过`$.get()`拉取最新的评论（将会返回一个类承诺对象）。当两个请求都接收到的时候，的时候，`showAjaxedContent()`函数将会被调用。`showAjaxedContent()`返回一个承诺，当容器的动画完成的时候，将会被解决。当`showAjaxedContent()`承诺被解决，`removeActiveClass()`将会被调用。`removeActiveClass()`返回一个承诺，将会在`setTimeout()`中4秒后解决。最后，在`removeActiveClass()`承诺被解决的时候，如果没有错误发生，最后的`.then()`回调被调用，
```
function getLatestNews() {
    return $.get( "latestNews.php", function( data ) {
        console.log( "news data received" );
        $( ".news" ).html( data );
    });
}
 
function getLatestReactions() {
    return $.get( "latestReactions.php", function( data ) {
        console.log( "reactions data received" );
        $( ".reactions" ).html( data );
    });
}
 
function showAjaxedContent() {
    // The .promise() is resolved *once*, after all animations complete
    return $( ".news, .reactions" ).slideDown( 500, function() {
        // Called once *for each element* when animation completes
        $(this).addClass( "active" );
    }).promise();
}
 
function removeActiveClass() {
    return $.Deferred(function( dfd ) {
        setTimeout(function () {
            $( ".news, .reactions" ).removeClass( "active" );
            dfd.resolve();
        }, 4000);
    }).promise();
}
 
$.when(
    getLatestNews(),
    getLatestReactions()
)
.then(showAjaxedContent)
.then(removeActiveClass)
.then(function() {
    console.log( "Requests succeeded and animations completed" );
}).fail(function() {
    console.log( "something went wrong!" );
});
```