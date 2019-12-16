### 代码组织概念
当你不止是要使用`jQuery`向你的网站添加一些简单的增强，而是开始开发完全的客户端应用的时候，你需要考虑如何组织代码。在这个章节，我们将浏览一系列你的`jQuery`应用程序可以使用的的代码组织模式，并探索[RequireJS]()依赖管理和构建系统。
link Key Concepts
### 核心理念
在进入代码组织模式之前，理解一些好的代码组织模式基本的理念很重要。

- 你的代码必须按功能分离为单元-模块，服务，等。避免将所有的代码放在一个巨大的`$(document).ready()`块的诱惑。这个理念，一般来说，叫做封装。
Don't repeat yourself. Identify similarities among pieces of 
- 不要重复你自己。将相似的功能代码块标记，使用继承技术避免重复的代码。
- 尽管围绕者`jQuery`生态，`JavaScript`应用并不全是关于`DOM`的，记住不是所有的功能都需要-或应该-有`DOM`表示。

- 功能单元应该是[松散耦合]()的，也就是说，每一个功能单元都应该可以独立存在，并且单元致电的通讯应该通过像自定义事件或者发布订阅之类的消息系统来处理。尽可能避免功能单元之间的直接交流。

松散耦合的概念对第一次接触复杂应用的人来说特别麻烦，所以在你开始的时候务必要注意。

### 封装
代码组织的第一步是将你的应用分离成独立的部分；有时候，即使只做了这个也足够了

#### 对象字面量
一个对象字面量可能是封装相关代码最好的方式了。它不提供任何独立的属性和方法，但是对于消除你代码中的匿名函数很有用，集中控制配置选项，扫清复用和重构的道路。

```
// An object literal
var myFeature = {
    myProperty: "hello",
 
    myMethod: function() {
        console.log( myFeature.myProperty );
    },
 
    init: function( settings ) {
        myFeature.settings = settings;
    },
 
    readSettings: function() {
        console.log( myFeature.settings );
    }
};
 
myFeature.myProperty === "hello"; // true
 
myFeature.myMethod(); // "hello"
 
myFeature.init({
    foo: "bar"
});
myFeature.readSettings(); // { foo: "bar" }
```
前面的对象字面量只是赋值给一个变量的简单变量。这个对象有一个属性和一系列的方法。所有的属性和方法都是公共的，所以你应用的任何地方都可以在这个对象上访问属性和调用方法。尽管有`init`方法，但是没有任何要求在对象运行之前调用它。
我们该怎么在`jQuery`代码中应用这种模式呢？假设我们用传统的`jQuery`风格编写这些代码：
```
// Clicking on a list item loads some content using the
// list item's ID, and hides content in sibling list items
$( document ).ready(function() {
    $( "#myFeature li" ).append( "<div>" ).click(function() {
        var item = $( this );
        var div = item.find( "div" );
        div.load( "foo.php?item=" + item.attr( "id" ), function() {
            div.show();
            item.siblings().find( "div" ).hide();
        });
    });
});
```
如果这是我们应用的全部，那就这样把。反之，如果这是一个巨大的应用的一部分，我们最好将不相关的功能分离。我们可能想要将`URL`从代码移动到配置区域。最后，我们可能想要打破链式调用，让之后修改功能的每个部分更简单。
```
// Using an object literal for a jQuery feature
var myFeature = {
    init: function( settings ) {
        myFeature.config = {
            items: $( "#myFeature li" ),
            container: $( "<div class='container'></div>" ),
            urlBase: "/foo.php?item="
        };
 
        // Allow overriding the default config
        $.extend( myFeature.config, settings );
 
        myFeature.setup();
    },
 
    setup: function() {
        myFeature.config.items
            .each( myFeature.createContainer )
            .click( myFeature.showItem );
    },
 
    createContainer: function() {
        var item = $( this );
        var container = myFeature.config.container
            .clone()
            .appendTo( item );
        item.data( "container", container );
    },
 
    buildUrl: function() {
        return myFeature.config.urlBase + myFeature.currentItem.attr( "id" );
    },
 
    showItem: function() {
        myFeature.currentItem = $( this );
        myFeature.getContent( myFeature.showContent );
    },
 
    getContent: function( callback ) {
        var url = myFeature.buildUrl();
        myFeature.currentItem.data( "container" ).load( url, callback );
    },
 
    showContent: function() {
        myFeature.currentItem.data( "container" ).show();
        myFeature.hideContent();
    },
 
    hideContent: function() {
        myFeature.currentItem.siblings().each(function() {
            $( this ).data( "container" ).hide();
        });
    }
};
 
$( document ).ready( myFeature.init );
```
第一个你可以注意到的是，这种很明显比原来更长-再一次，如果这是我们应用的全部，使用对象字面量可能过头了。假设这不是我们应用的全部，我们将得到一些东西：
We've eliminated the use of anonymous functions.
- 我们将我们的功能分离成了小的方法。在这个功能中，我们想要改变怎样实现内容，很清楚就知道去哪里改变它。在原来的代码中，这个步骤定位更加困难。
- 我们消除了匿名函数
- 我们将配置选项从代码体中移除了，并集中放置。
- 我们消除了链式约束，让代码更加容易重构、混合、安排。
对于不小的功能，对象字面量比堆积一大堆的代码在`$(document).ready()`块中清晰多了，因此它让我们思考我们的功能点。然而，这并没有比简单的放置一系列函数声明在`$(document).read()`块高级很多。
### 模块模式
模块模式客服克服了一些对象字面量的限制，如果希望暴露一些公共的`API`的时候，它提供私有的变量和函数。
```
// The module pattern
var feature = (function() {
 
    // Private variables and functions
    var privateThing = "secret";
    var publicThing = "not secret";
 
    var changePrivateThing = function() {
        privateThing = "super secret";
    };
 
    var sayPrivateThing = function() {
        console.log( privateThing );
        changePrivateThing();
    };
 
    // Public API
    return {
        publicThing: publicThing,
        sayPrivateThing: sayPrivateThing
    };
})();
 
feature.publicThing; // "not secret"

// Logs "secret" and changes the value of privateThing
feature.sayPrivateThing();
```

在上面的栗子中，我们自执行了一个匿名函数，返回一个对象。在函数内部，我们定义了一些变量。因为变量是定义在函数内部的，我们无法在函数的外部访问他们，直到我们将他们作为返回对象。这意味着外面的代码不能访问`privateThing`或者`changePrivateThing`函数。然而，`sayPrivateThing`有权限访问`privateThing`，因为这两个定义在了相同的作用域内。
这个模式非常强大，因为你可以因为你可以从变量名收集到，返回的对象和属性构成了暴露有限的`API`，它可以提供私有变量和函数。

下面是上面栗子的一个修正版本，显示了我们怎样使用模块模式创建相同的功能，但是该模块只暴露一个公共的方法，`showItemIndex()`。
```
// Using the module pattern for a jQuery feature
$( document ).ready(function() {
    var feature = (function() {
        var items = $( "#myFeature li" );
        var container = $( "<div class='container'></div>" );
        var currentItem = null;
        var urlBase = "/foo.php?item=";
 
        var createContainer = function() {
            var item = $( this );
            var _container = container.clone().appendTo( item );
            item.data( "container", _container );
        };
 
        var buildUrl = function() {
            return urlBase + currentItem.attr( "id" );
        };
 
        var showItem = function() {
            currentItem = $( this );
            getContent( showContent );
        };
 
        var showItemByIndex = function( idx ) {
            $.proxy( showItem, items.get( idx ) );
        };
 
        var getContent = function( callback ) {
            currentItem.data( "container" ).load( buildUrl(), callback );
        };
 
        var showContent = function() {
            currentItem.data( "container" ).show();
            hideContent();
        };
 
        var hideContent = function() {
            currentItem.siblings().each(function() {
                $( this ).data( "container" ).hide();
            });
        };
 
        items.each( createContainer ).click( showItem );
 
        return {
            showItemByIndex: showItemByIndex
        };
    })();
 
    feature.showItemByIndex( 0 );
});
```