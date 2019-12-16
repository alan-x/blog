### 延迟栗子
### 更多延迟的栗子
延迟在`Ajax`幕后使用，但是并不是说不能在其他地方使用。这个章节概括了延迟会版主我们解决异步行为和代码耦合的场景。

### 缓存
### 异步缓存
谈到异步任务，缓存有点麻烦，因为你需要保证对于一个给定的key，任务只执行一次。作为结果，代码必须以某种方式跟踪到任务里面。
```
$.cachedGetScript( url, callback1 );
$.cachedGetScript( url, callback2 );
```
缓存机制必须保证 URL 只请求一次，尽管脚本还不在缓存中。这显示了一些逻辑去保持跟踪绑定在 URL 的回调，为了缓存系统合理的处理每一个完成和入站请求。
```
var cachedScriptPromises = {};
 
$.cachedGetScript = function( url, callback ) {
    if ( !cachedScriptPromises[ url ] ) {
        cachedScriptPromises[ url ] = $.Deferred(function( defer ) {
            $.getScript( url ).then( defer.resolve, defer.reject );
        }).promise();
    }
    return cachedScriptPromises[ url ].done( callback );
};
```
一个承诺缓存一个 URL，如果给定的 URL 还没有承诺，将会创建一个延迟并发生请求。当请求完成，延迟被解决（使用`defer.resolve()`）；如果错误发生，延迟被拒绝（使用`defer.reject`）。如果承诺已经存在，回调被绑定到存在的延迟；换句话说，承诺先被创建，然后回调被绑定。这种解决方案最大的好处是，它会透明的处理完成和入站请求，另一个好处是基于延迟的错误处理非常优雅。承诺将会以拒绝为结束，可以提供一个错误回调来测试：
```
$.cachedGetScript( url ).then( successCallback, errorCallback );
```
### 通用异步缓存
也可以让代码代码完全通用并且创建一个缓存工厂，当一个`key`还没被缓存的时候，抽象出将要执行的实际任务：
```
$.createCache = function( requestFunction ) {
    var cache = {};
    return function( key, callback ) {
        if ( !cache[ key ] ) {
            cache[ key ] = $.Deferred(function( defer ) {
                requestFunction( defer, key );
            }).promise();
        }
        return cache[ key ].done( callback );
    };
};
```
现在请求的逻辑被移除了，`$.cacheGetScript()`可以被这样重写：
```
$.cachedGetScript = $.createCache(function( defer, url ) {
    $.getScript( url ).then( defer.resolve, defer.reject );
});
```
这有效是因为每一次调用`$.createCache()`将会创建一个新的缓存仓库，并返回新的缓存检测函数：
### 图片加载
一个缓存也可以用来保证相同的图片不被多次加载。
```
$.loadImage = $.createCache(function( defer, url ) {
    var image = new Image();
    function cleanUp() {
        image.onload = image.onerror = null;
    }
    defer.then( cleanUp, cleanUp );
    image.onload = function() {
        defer.resolve( url );
    };
    image.onerror = defer.reject;
    image.src = url;
});
```
再一次，代码片段：
```
$.loadImage( "my-image.png" ).done( callback1 );
$.loadImage( "my-image.png" ).done( callback2 );
```
将会无视是否`my-image.png`是否已经被加载，或者实际上正在加载。
### 缓存数据 API 响应
在你页面的生命周期被认为是不可变的`API`请求也是完美的候选。比如，下面：
```
$.searchTwitter = $.createCache(function( defer, query ) {
    $.ajax({
        url: "http://search.twitter.com/search.json",
        data: {
            q: query
        },
        dataType: "jsonp",
        success: defer.resolve,
        error: defer.reject
    });
});
```
允许你在`Twitter`执行搜索同时缓存他们：
```
$.searchTwitter( "jQuery Deferred", callback1 );
$.searchTwitter( "jQuery Deferred", callback2 );
```
### 定时器
基于延迟的缓存不紧急限制与网络请求；也可以用于定时器目的。

比如，你可能需要在页面上等待一些时间后执行一个动作，捕捉用户的注意到一个他们可能意识不到的指定的特性或者处理超市（比如一个智力问题）。尽管`setTime()`适用于大部分场景，但是它并没有处理定时器被延迟执行的场景，就算它在理论上已经过期了。我们可以处理这种情况通过下列的缓存场景：
```
var readyTime;
 
$(function() {
    readyTime = jQuery.now();
});
 
$.afterDOMReady = $.createCache(function( defer, delay ) {
    delay = delay || 0;
    $(function() {
        var delta = $.now() - readyTime;
        if ( delta >= delay ) {
            defer.resolve();
        } else {
            setTimeout( defer.resolve, delay - delta );
        }
    });
});
```
新的`$.afterDOMReady()`工具方法在`DOM`准备好的时候提供适合的定时器，保证最少的定时器将被使用。如果延迟已经超时了，任何回调将会马上执行。
### 一次性事件
尽管`jQuery`提供了一个人需要的所有时间绑定，处理只处理一次的事件可能会很笨重。
比如，你可能希望有一个按钮，第一次点击的时候将会打开一个面板，然后保持打开状态，或者在第一次按钮被点击的时候采取特殊的初始化动作。当处理这种场景的时候，一般会这么解决：
```
var buttonClicked = false;
 
$( "#myButton" ).click(function() {
    if ( !buttonClicked ) {
        buttonClicked = true;
        initializeData();
        showPanel();
    }
});
```
然后，你可能希望去采取动作，但是只有在面板打开的时候。
```
if ( buttonClicked ) {
 
    // Perform specific action
 
}
```
这是一个非常耦合的解决方案。如果你想要采取其他动作，你需要去修改绑定的代码或者重复它。如果你不，你唯一的选择时测试`buttonClicked`，你可能丢失新动作，因为`buttonClicked`变量可能是`false`，你的新代码可能永远不会被执行。

使用延迟可以做的更好（为了简化，下面的代码只能在一个元素和一个事件类型工作，但对于多种事件类型的完整集合，他可以很简单的推广）：
```
$.fn.bindOnce = function( event, callback ) {
    var element = $( this[ 0 ] ),
        defer = element.data( "bind_once_defer_" + event );
 
    if ( !defer ) {
        defer = $.Deferred();
        function deferCallback() {
            element.unbind( event, deferCallback );
            defer.resolveWith( this, arguments );
        }
        element.bind( event, deferCallback )
        element.data( "bind_once_defer_" + event , defer );
    }
 
    return defer.done( callback ).promise();
};
```
代码如下工作：
- 检查元素对于所给事件是否已经有了延迟绑定了
- 如果没有，创建一个并在第一次触发的时候解决
- 然后绑定所有给回调到延迟并返回承诺
```
$.fn.firstClick = function( callback ) {
    return this.bindOnce( "click", callback );
};
```
然后逻辑可以冲构成这样：
```
var openPanel = $( "#myButton" ).firstClick();
 
openPanel.done( initializeData );
openPanel.done( showPanel );
```
如果一个动作只能在面板打开之后才执行：
```
openPanel.done(function() {
 
    // Perform specific action
 
});
```
如果面板没有打开，也不会丢失什么，动作将会延迟到按钮被点击。
### 绑定帮助
上面所有的栗子分开来看好想有点儿局限。然而，承诺真正的威力在于混合他们使用。
### 首次点击并打开上述面板的时候请求面板内容
下面是一个按钮的代码，当点击的时候，打开一个面板。通过铜线请求它的内容，然后淡入内容。使用帮助定义更加简单，他可以这么定义
```
$( "#myButton" ).firstClick(function() {
    var panel = $( "#myPanel" );
    $.when(
        $.get( "panel.html" ),
        panel.slideDownPromise()
    ).done(function( ajaxResponse ) {
        panel.html( ajaxResponse[ 0 ] ).fadeIn();
    });
});
```
### 第一次点击的时候打开上述面板并在面板中加载图片
另一个目标是只有在按钮被点击并且所有的图片被加载好的时候才淡入面板。
这个例子的`HTML`代码可能如下：

```
<div id="myPanel">
    <img data-src="image1.png" alt="">
    <img data-src="image2.png" alt="">
    <img data-src="image3.png" alt="">
    <img data-src="image4.png" alt="">
</div>
```
我们使用`data-src`属性保持对真实图片地址的跟踪。使用承诺处理我们的使用场景的代码如下：
```
$( "#myButton" ).firstClick(function() {
    var panel = $( "#myPanel" ),
        promises = [];
 
    panel.find( "img" ).each(function() {
        var image = $( this ),
            src = element.attr( "data-src" );
        if ( src ) {
            promises.push(
                $.loadImage( src ).then(function() {
                    image.attr( "src", src );
                }, function() {
                    image.attr( "src", "error.png" );
                })
            );
        }
    });
 
    promises.push( panel.slideDownPromise() );
 
    $.when.apply( null, promises ).done(function() {
        panel.fadeIn();
    });
});
```
这里的巴西是保持对所有`$.loadImage()`承诺的跟踪。我们在之后使用`$.when`将面板的`.slideDown()`动画加入他们。所以当按钮第一次被点击，面板将会滑下，图片开始加载，一旦面板滑下完成，所有的图片加载完成，面板将会淡入。

### 在页面上延迟指定的事件加载图片
为了实现延迟加载图片显示在整个页面上，瞎买呢格式的`HTML`可以被使用：
1
2
3
4
```
<img data-src="image1.png" data-after="1000" src="placeholder.png" alt="">
<img data-src="image2.png" data-after="1000" src="placeholder.png" alt="">
<img data-src="image1.png" src="placeholder.png" alt="">
<img data-src="image2.png" data-after="2000" src="placeholder.png" alt="">
```
它表达的东西很简单
- 加载`image1.png`并在第三张图片马上显示它，1秒后在第一张图片显示。
- 加载`image2.png`并在1秒后在第二张图片处显
```
$( "img" ).each(function() {
    var element = $( this ),
        src = element.attr( "data-src" ),
        after = element.attr( "data-after" );
    if ( src ) {
        $.when(
            $.loadImage( src ),
            $.afterDOMReady( after )
        ).then(function() {
            element.attr( "src", src );
        }, function() {
            element.attr( "src", "error.png" );
        }).done(function() {
            element.fadeIn();
        });
    }
});
```
为了延迟加载图片：
```
$( "img" ).each(function() {
    var element = $( this ),
        src = element.attr( "data-src" ),
        after = element.attr( "data-after" );
    if ( src ) {
        $.afterDOMReady( after, function() {
            $.loadImage( src ).then(function() {
                element.attr( "src", src );
            }, function() {
                element.attr( "src", "error.png" );
            }).done(function() {
                element.fadeIn();
            });
        });
    }
});
```
这样，在延迟被解决后，图片被加载。这对于你要限制页面加载的网络请求非常有意义。
