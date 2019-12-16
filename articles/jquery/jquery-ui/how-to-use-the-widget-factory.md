### 怎样使用组件工厂
为了开始，我们将会创建一个进度条，只允许我们设置进度条一次。就像我们下面看到的，这通过调用`jQuery.widget()`方法并传递两个参数实现的：要创建的插件的名称，支持我们插件的包含函数的对象字面量。当我们的插件被调用的时候，它将会创建一个新的插件实例，所有的函数都会在这个实例的上下文执行。这和标准的 jQuery 方法有亮点重要的不同。第一，上下文是一个对象，而不是 DOM 元素。第二，上下文总是一个对象，永远不会是集合。
```
$.widget( "custom.progressbar", {
    _create: function() {
        var progress = this.options.value + "%";
        this.element
            .addClass( "progressbar" )
            .text( progress );
    }
});
```
插件的名字必须包含一个命名空间，在这个场景中，我们使用`custom`作为命名空间。你只能创建一个等级深度的命名空间，因此`custom.progressbar`是一个有效的插件名称，而`very.custome.progressbar`不是。

我们可以看见组件工厂提供了两个属性给我们。`this.element`是一个 jQuery 对象，只宝航一个元素。如果插件在包含多个元素的 jQuery 对象上调用，每一个元素都会创建一个分离的插件实例。第二个属性，`this.options`，是一个哈希，包含我们插件配置的所有键值对。这些选项可以这么传递给我们插件：
```
$( "<div></div>" )
    .appendTo( "body" )
    .progressbar({ value: 20 });
```
当我们调用`jQuery.widget()`的时候，它通过添加一个函数到`jQuery.fn`扩展 jQuery（创建标准插件的系统）。添加的函数的名字是基于你传递给`jQuery.widget()`的名字，不包含命名空间-在这个场景中是“processbar”。传递给我们插件的配置在我们的插件实例内部可以通过`this.options`获取或者设置。就像瞎买呢显示的，我们可以为我们的配置指定默认值。当设置你的 API 的时候，你应该绘制出你的插件最常用的场景，你可以设置适当的默认值，让所有的选项都是可信任的。
```
$.widget( "custom.progressbar", {
 
    // Default options.
    options: {
        value: 0
    },
    _create: function() {
        var progress = this.options.value + "%";
        this.element
            .addClass( "progressbar" )
            .text( progress );
    }
});
```
### 调用插件方法
现在我们可以初始化我们的极度条，我们将添加在我们插件实例调用放的执行动作的能力。为了定义个插件方法，我们只要在传递给`jQuery.widget()`的对象字面量包含函数就行了。我们也可以定义“私有”方法通过添加一个下划线到函数名字前面。
```
$.widget( "custom.progressbar", {
 
    options: {
        value: 0
    },
 
    _create: function() {
        var progress = this.options.value + "%";
        this.element
            .addClass( "progressbar" )
            .text( progress );
    },
 
    // Create a public method.
    value: function( value ) {
 
        // No value passed, act as a getter.
        if ( value === undefined ) {
            return this.options.value;
        }
 
        // Value passed, act as a setter.
        this.options.value = this._constrain( value );
        var progress = this.options.value + "%";
        this.element.text( progress );
    },
 
    // Create a private method.
    _constrain: function( value ) {
        if ( value > 100 ) {
            value = 100;
        }
        if ( value < 0 ) {
            value = 0;
        }
        return value;
    }
});
```
为了调在插件实例上调用方法，你可以给 jQuery 插件传递插件的名字。如果你调用的方法接收参数，你简单传递这些阐述在名字后面就好了。
注意：通过传递方法名称到用来初始化插件相同的 jQuery 函数来执行方法看起来很怪。这是为了维护链式方法调用放置 jQuery 命名空间污染。稍候的文章中我们将看到优化的使用防范，看起来感觉更自然。
```
var bar = $( "<div></div>" )
    .appendTo( "body" )
    .progressbar({ value: 20 });
 
// Get the current value.
alert( bar.progressbar( "value" ) );
 
// Update the value.
bar.progressbar( "value", 50 );
 
// Get the current value again.
alert( bar.progressbar( "value" ) );
```
### 操作配置
我们插件的一个开箱即用的方法是`option()`。`opiton()`方法允许你在初始化之后获取和设置配置。这个方法工作起来就像`jQuery`的`.css()`和`.attr()`方法：你可以只传递一个名字，将它作为一个`getter`使用，或者传递一名字和值作为`setter`使用，或者一个键值对哈希去设置多个值。当作用`getter`的时候，差劲阿将会返回代表我们传入名字的当前的值。当用作`setter`的时候，插件的`_setOption`方法会在每一个设置的配置上调用。我们可以在我们的插件上指定一个`_setOption`方法去响应配置的变化。对于要指定不依赖配置数量改变的操作，我们可以覆盖`_setOptions()`.
```
$.widget( "custom.progressbar", {
    options: {
        value: 0
    },
    _create: function() {
        this.options.value = this._constrain(this.options.value);
        this.element.addClass( "progressbar" );
        this.refresh();
    },
    _setOption: function( key, value ) {
        if ( key === "value" ) {
            value = this._constrain( value );
        }
        this._super( key, value );
    },
    _setOptions: function( options ) {
        this._super( options );
        this.refresh();
    },
    refresh: function() {
        var progress = this.options.value + "%";
        this.element.text( progress );
    },
    _constrain: function( value ) {
        if ( value > 100 ) {
            value = 100;
        }
        if ( value < 0 ) {
            value = 0;
        }
        return value;
    }
});
```
### 添加回调
让你的插件变得可扩展的方式是添加回调，这样用户就可以在你的状态变化的时候响应。我们可以在瞎买呢看到如何添加一个回调到我们的进度条插件，示意进度达到了100%。`_trigger（）`方法接收三个参数：回调的名字，初始化回调的 jQuery 事件对象，和该事件相关的哈希数据。回调名称是唯一的必要参数，但是其他参数对于想要在你的插件上实现自己自定义功能的用户来说非常用用。比如，如果我们正在创建一个拖拽插件，当触发`drag`事件时我们可以传递`mouseover`事件；这允许用户基于事件对象提供的`x/y`响应拖拽事件。注意传递给`_trigger()`的原始事件对象必须是一个`jQuery`事件，不是原生浏览器事件。　
```
$.widget( "custom.progressbar", {
    options: {
        value: 0
    },
    _create: function() {
        this.options.value = this._constrain(this.options.value);
        this.element.addClass( "progressbar" );
        this.refresh();
    },
    _setOption: function( key, value ) {
        if ( key === "value" ) {
            value = this._constrain( value );
        }
        this._super( key, value );
    },
    _setOptions: function( options ) {
        this._super( options );
        this.refresh();
    },
    refresh: function() {
        var progress = this.options.value + "%";
        this.element.text( progress );
        if ( this.options.value == 100 ) {
            this._trigger( "complete", null, { value: 100 } );
        }
    },
    _constrain: function( value ) {
        if ( value > 100 ) {
            value = 100;
        }
        if ( value < 0 ) {
            value = 0;
        }
        return value;
    }
});
```
回调函数只是一个额外的配置，所以你可以获取和设置它，就像其他配置一样。当一个回调执行的时候，一个代表的事件也被触发了。事件类型决定于插件名称和回调名称的结合。互调和事件都接收相同的两个参数：一个事件对象和事件相关的哈希数据，就像我们下面看到的。你的插件可能有允许用户去阻止的功能。支持这个功能的最好方式是创建可取消的回调。用户可以取消一个回调，或者它关联的事件，就像取消任何原生事件，通过调用`event.perventDefault()`或者返回`false`。如果用户取消了回调，`_trigger()`方法将返回`false`，所以你可以在你的插件内实现适当的功能。
```
var bar = $( "<div></div>" )
    .appendTo( "body" )
    .progressbar({
        complete: function( event, data ) {
            alert( "Callbacks are great!" );
        }
    })
    .bind( "progressbarcomplete", function( event, data ) {
        alert( "Events bubble and support many handlers for extreme flexibility." );
        alert( "The progress bar value is " + data.value );
    });
 
bar.progressbar( "option", "value", 100 );
```
### 面纱之下
现在我们可以看见如何使用组件工厂创建插件，让我们看看它实际上如何工作的。当你调用`jQuery.widget()`的时候，它为你的插件创建一个构造函数，并设置你传递进来的对象字面量作为你的插件的实例。你的插件所有自动获取的功能都是从一个基本的组件原型得到的，它定义在`jQuery.wedget.prototype`。当插件实例被创建，它被`jQuery.data`存储在原始`DOM`元素上，将插件的名字作为`key`。

因为插件实例是直接和`DOM`元素关联的，如果你想要，你可以直接访问插件实例，而不需要通过暴露的插件方法。这将允许你直接在插件实例上调用方法，而不需要传递方法名为字符串，这将会给你访问插件属性的直接权限。
```
var bar = $( "<div></div>" )
    .appendTo( "body" )
    .progressbar()
    .data( "custom-progressbar" );
 
// Call a method directly on the plugin instance.
bar.option( "value", 50 );
 
// Access properties on the plugin instance.
alert( bar.options.value );
```
你也可以创建一个实例，不需要通过插件方法，通过直接调用构造方法，将配置和元素作为参数：
```
var bar = $.custom.progressbar( {}, $( "<div></div>" ).appendTo( "body") );

// Same result as before.
alert( bar.options.value );
```
### 扩展插件的原型
插件拥有构造方法和原型最大的好处之一就是可以很简单的扩展插件。通过添加或者修改插件原型的方法，我们可以定义我们插件的所有行为。比如，如果我们想要给我们的进度条添加一个方法，用来设置进度为0%，我们可以添加一个方法到原型，它可以直接被任意的插件实例直接调用。
```
$.custom.progressbar.prototype.reset = function() {
    this._setOption( "value", 0 );
};
```
了解更多扩展插件，包括怎样在一存在的插件上构建一个完全新的组件，查阅[使用组件工厂扩展组件]()。
### 清理
在某些场景，让用户去应用，而在之后让用户去卸载你的插件是有意义的。你可以通过`_destory()`方法完成这个功能。在`_destory()`方法内，你应该撤销任何你的插件在初始化和之后所做的任何操作。`_destory()`被`destory()`方法调用，如果你的实例的绑定的元素从`DOM`中被移除，它将会自动调用，所以这也可以用来做垃圾收集。基础的`desotry`方法也处理一些常用的清理操作，比如从组件的`DOM`元素移除实例的引用，解绑元素命名空间所有的事件，解绑通过`_bind()`绑定的所有常见的事件。
```
$.widget( "custom.progressbar", {
    options: {
        value: 0
    },
    _create: function() {
        this.options.value = this._constrain(this.options.value);
        this.element.addClass( "progressbar" );
        this.refresh();
    },
    _setOption: function( key, value ) {
        if ( key === "value" ) {
            value = this._constrain( value );
        }
        this._super( key, value );
    },
    _setOptions: function( options ) {
        this._super( options );
        this.refresh();
    },
    refresh: function() {
        var progress = this.options.value + "%";
        this.element.text( progress );
        if ( this.options.value == 100 ) {
            this._trigger( "complete", null, { value: 100 } );
        }
    },
    _constrain: function( value ) {
        if ( value > 100 ) {
            value = 100;
        }
        if ( value < 0 ) {
            value = 0;
        }
        return value;
    },
    _destroy: function() {
        this.element
            .removeClass( "progressbar" )
            .text( "" );
    }
});
```
### 关闭评论
组件工厂是创建状态插件的唯一方式。还有一些其他不同的模型可以使用，每一种都有不同的有点和缺点。组件工厂为你解决了大量普通问题，让你可以大大提高生产力，也大大提高代码复用，让它更加适合`jQuery UI`，就像其他状态插件。

你可能注意到在这个文章中，我们使用自定义命名空间。`ui`命名空间是`jQuery UI`插件官方使用的。当构建你自己的插件的时候，你应该创建自己的命名空间。这让插件的来源和它所属的大集合变得更清晰。