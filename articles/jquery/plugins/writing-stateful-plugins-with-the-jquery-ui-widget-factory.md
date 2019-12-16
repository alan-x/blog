### 使用`jQuery UI Widget Factory`编写带状态的插件
### ### 使用`jQuery UI Widget Factory`编写带状态的插件
尽管大部分存在的`jQuery`插件都是无状态的-我们在元素上调用，这就是我们和插件的交互方式-很大一部分的功能不符合基本的插件模式。

为了填满这个沟壑，`jQuery UI`实现了一个额更加高级的插件系统。这个新系统管理状态，允许多个功能通过个简单的插件暴露，并提供许多的扩展点。这个系统叫做`Widget Factory`，它作为`jQuery UI 1.8`的一部分-`jQuery.widget`暴露。它的使用依赖于`jQuery UI`

为了证明`Widget Factory`的能咯，我们将会创建一个简单的进度条插件。
为了开始，我们将会创建一个进度条，只允许我们设置进度一次。就像我们将看到的，通过调用2个参数调用`jQuery.widget`就可以完成：要创建插件的名字和包含要支持我们插件的功能的对象字面量。

当我们的插件被调用，他将会创建一个新的插件实例，所有的函数将会在这个实例的上下文执行。这和标准的`jQuery`插件有两个不同的地方。第一，上下文是一个对象，不是一个`DOM`元素。第二，上下文总是一个单独的元素，不会是集合。
A simple, stateful plugin using the jQuery UI Widget Factory:
一个使用`jQuery UI Widget Factory`简单，有状态的插件：
```
$.widget( "nmk.progressbar", {
 
    _create: function() {
        var progress = this.options.value + "%";
        this.element.addClass( "progressbar" ).text( progress );
    }
 
});
```
插件的名字必须包含命名空间：在这个场景中，我们使用`npm`命名空间。命名空间的等级深度只能是一-这也是为什么我们无法使用类似`nmk.foo`类似的命名空间。我们也可以看到`Widget Factory`提供了两个属性给我们。`this.element`是一个`jQuery`对象只包含一个元素。如果我们的插件在一个`jQuery`对象对象包含很多的元素，分离的插件实例将会为每个元素创建，每一个实例都有他自己的`this.element`。第二个属性，`this.option`是一个哈希，包含我们插件所有配置的键值对。这些配置可以像显示的那样传递给我们的插件。

注意：在我们的栗子中，我们使用`nmk`命名空间。`ui`命名空间是为官方的`jQuery UI`插件准备的。当创建你自己的插件的时候，你应该创建你自己的命名空间。这让插件的涞源变得清晰，并且表明他是属于哪个大集合的一部分

传递配置给插件：
```
$( "<div />" ).appendTo( "body" ).progressbar({ value: 20 });
```
当我们调用`jQuery.widget`的时候，他通过添加一个方法到`jQuery.fn`来扩展`jQuery`（和我们创建标准的插件一样）。添加的函数的名字取决你你传递给`jQuery.widget`的名字，不带命名空间；在这个栗子中我们创建了`jQuery.fn.progressbar`。传递给我们插件的配置可以在我们的插件内通过`this.options`获取和设置。


如下所示，当设计API的时候，我们可以为我们的配置指定默认值，你应该描绘出你的插件最常用的使用场景，这样你就可以设置何时的默认值，让所有的配置都是可信的配置。
为插件设置默认配置：
```
$.widget( "nmk.progressbar", {
 
    // Default options.
    options: {
        value: 0
    },
 
    _create: function() {
        var progress = this.options.value + "%";
        this.element.addClass( "progressbar" ).text( progress );
    }
 
});
```
### 添加方法到插件
现在我们可以初始化我们的进度条，我们将会通过调用在我们插件实例上的能力来添加能力去执行操作。为了定义插件方法，我们只要将函数包裹进我们传递给`jQuery.widget`的对象字面量。我们也可以定义`私有`方法通过在函数名字前面添加下划线。
Creating widget methods:
创建插件方法：
```
$.widget( "nmk.progressbar", {
    options: {
        value: 0
    },
 
    _create: function() {
        var progress = this.options.value + "%";
        this.element.addClass("progressbar").text( progress );
    },
 
    // Create a public method.
    value: function( value ) {
 
        // No value passed, act as a getter.
        if ( value === undefined ) {
 
            return this.options.value;
 
        // Value passed, act as a setter.
        } else {
 
            this.options.value = this._constrain( value );
            var progress = this.options.value + "%";
            this.element.text( progress );
 
        }
 
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
为了在你的插件实例上调用方法，你传递了方法的名字给`jQuery`插件。如果你调用的方法接收参数，你简单的在方法名后面传递这些参数。
Calling methods on a plugin instance:
在插件实例上调用方法：
```
var bar = $( "<div />" ).appendTo( "body").progressbar({ value: 20 });
 
// Get the current value.
alert( bar.progressbar( "value" ) );
 
// Update the value.
bar.progressbar( "value", 50 );
 
// Get the current value again.
alert( bar.progressbar( "value" ) );
```
注意：通过传递方法名称给和我们用来初始化插件相同的`jQuery`函数看起来很怪，这是为了维护链式方法调用的时候避免污染`jQuery`命名空间。
link Working with Widget Options
### 使用组件配置
我们的插件有一个自动可用的方法，那就是选项方法。选项方法允许你在初始化之后设置和获取配置。这个方法工作起来很像`.css`和`.attr()`方法：你可以只传递一个名字用来作为一个`getter`，一个名字和一个值作为`setter`，或者一个键值哈希对去设置多个值。当作为一个`getter`的时候，插件将会返回传入的名字的当前的配置值。当作为一个`setter`的时候，插件的`_setOption`方法将在每一个被设置的选项上调用。我们可以在我们的插件内指定一个`_setoption`方法来响应选项的变化。
Responding when an option is set:
设置配置的时候的响应：
```
$.widget( "nmk.progressbar", {
 
    options: {
        value: 0
    },
 
    _create: function() {
        this.element.addClass( "progressbar" );
        this._update();
    },
 
    _setOption: function( key, value ) {
        this.options[ key ] = value;
        this._update();
    },
 
    _update: function() {
        var progress = this.options.value + "%";
        this.element.text( progress );
    }
 
});
```
### 添加回调
让你的插件变得可扩展最简单的方式之一就是添加回调，这样用户就可以响应你的插件的状态的变化。我们在下面将看到怎样添加一个回调到我们的进度条，表示进度到达了100%。`_trigger`方法接收三个参数：回调的名字，初始化回调的原生事件对象，和事件相关的哈希数据。回调名是唯一必须的参数，但是其他对想要在你的插件上实现自定义功能的用户来说也是非常有用的。比如，如果我们创建了拖拽插件，我们可以传递一个原生的`mousemove`事件当触发`drag`回调的时候；这允许用户基于事件对象提供的`x/y`去响应`drag`。
Providing callbacks for user extension:
为用户扩展提供回调：
```
$.widget( "nmk.progressbar", {
 
    options: {
        value: 0
    },
 
    _create: function() {
        this.element.addClass( "progressbar" );
        this._update();
    },
 
    _setOption: function( key, value ) {
        this.options[ key ] = value;
        this._update();
    },
 
    _update: function() {
        var progress = this.options.value + "%";
        this.element.text( progress );
        if ( this.options.value == 100 ) {
            this._trigger( "complete", null, { value: 100 } );
        }
    }
 
});
```
回调函数基本是只是额外的选项，所以你可以像其他选项那样获取和设置他们。当回调执行的时候，一个相关的事件也被触发了。事件类型取是插件名和回调名的结合。回调和事件都接收相同的两个参数，一个事件对象和事件相关的哈希数据，就像前面看到的。
如果你的插件想要有允许用户阻止的功能，最好的支持方式是创建一个可以取消的回调。用户可以取消回调或他关联的事件。就像取消任何原生事件一样：通过调用`event.preventDefault（）`或者返回`false`。如果用户取消了回调`_trigger`方法将会返回`false`，你可以在你的插件内实现适合的功能。
绑定组件事件：
```
var bar = $( "<div />" ).appendTo( "body" ).progressbar({
 
    complete: function( event, data ) {
        alert( "Callbacks are great!" );
    }
 
}).bind( "progressbarcomplete", function( event, data ) {
 
     alert( "Events bubble and support many handlers for extreme flexibility." );
 
     alert( "The progress bar value is " + data.value );
 
});
 
bar.progressbar( "option", "value", 100 );
```
### `Widget Factory`:面罩之下
当你调用`jQuery.widget`的时候，他为你的插件创建一个构造函数，并设置你传递进来的对象字面量为你的插件实例的原型。自动添加到你的插件的所有的功能都来自一个基本的组件原型，定义为`jQuery.Widget.prototype`。当一个插件实例创建的时候，他使用`jQuery.data`保存原始的`DOM`元素，并使用插件的全名（插件的命名空间，加上一个连字符，加上插件的名字）作为键。比如`jQuery UI`弹窗组件使用一个键，名为`ui-dialog`。
因为插件实例是直接链接到`DOM`元素的，如果你想要，你可以通过暴露的插件方法直接访问插件实例。这将允许你直接在插件实例上调用方法，而不是传递方法名作为一个字符串，同时也给你访问插件属性的直接权限。
```
var bar = $( "<div />")
    .appendTo( "body" )
    .progressbar()
    .data( "nmk-progressbar" );
 
// Call a method directly on the plugin instance.
bar.option( "value", 50 );
 
// Access properties on the plugin instance.
alert( bar.options.value );
```
一个插件拥有构造器和原型最大的好处就是扩展插件非常轻松。通过添加或者修改插件原型上的方法，我们可以修改我们插件实例的所有行为。比如。如果我们想要添加一个方法去设置我们的进度条的进度为0%，我们可以添加这个方法到原型，他将会直接允许在任何插件实例上调用。
```
$.nmk.progressbar.prototype.reset = function() {
    this._setOption( "value", 0 );
};
```
### 清理
在一些场景中，允许用户应用和取消应用你的插件是有意义的。你可以通过`_destory`方法来完成。在`_destory`方法内，你应该取消任何你的插件在厨师哈和后续时候中的操作。`_destory`方法将在你的插件实例尝试从`DOM`上移除元素的时候自动调用，所以这也可以用来做垃圾收集。最基本的`destory`方法调用插件实例的`_destory`。
Adding a destroy method to a widget:
添加`destory`方法到组件：
```
$.widget( "nmk.progressbar", {
 
    options: {
        value: 0
    },
 
    _create: function() {
        this.element.addClass("progressbar");
        this._update();
    },
 
    _setOption: function( key, value ) {
        this.options[ key ] = value;
        this._update();
    },
 
    _update: function() {
        var progress = this.options.value + "%";
        this.element.text( progress );
        if ( this.options.value === 100 ) {
            this._trigger( "complete", null, { value: 100 } );
        }
    },
 
    _destroy: function() {
        this.element
            .removeClass( "progressbar" )
            .text( "" );
    }
 
});
```
### 总结
`Widget Factory`是我们创建有状态插件的唯一方法。有几种不同的模型可以使用，每一种都有优点和缺点。`Widget Factory`为你解决了阿部分常见的问题，可以大幅度提高生产力，当然也大大提供代码复用，就像其他有状态的插件一样，让它更适合于`jQuery UI`。