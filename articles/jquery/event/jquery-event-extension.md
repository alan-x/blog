### `jQuery`事件扩展
`jQuery`提供了许多方式去拓展它的事件系统，可以在事件绑定到元素之后提供自定义功能。在内部，这些扩展主要用来保证类似`submit`、`click`之类的标准事件在跨浏览器时表现一致。然而，他们也可以用来定义拥有新行为的新事件。

这个文档概括了从`jQuery 1.7`以来可用的扩展；从`jQuery 1.3`以来，关于该功能一些少量的文档集就已经存在，但是有很大的差别。了解早起版本更多特殊事件，请查阅[Ben Alma 的 jQuery 特殊事件文章]()。

注意：`jQuery`事件扩展是一个高级特性；他们需要相对于其他大部分`API`关于浏览器行为和`jQuery`内部更深入的了解。大部分的`jQuery`用户不需要去使用事件扩展，只有需要使用他们的人才需要关心。比如，一个有大量第三方插件的大项目，改变像`click`或`mouseover`之类的标准事件的行为将会导致严重的兼容性问题。
### 事件概述和一些建议
当编写事件扩展的时候，了解事件是如何在`jQuery`事件系统中流动是必要的。查看`API`等级的事件系统描述，包括事件代理的讨论，可以查看[.on()]()方法。
为了简化事件管理，`jQuery`为每个元素的每个事件只提供了一个简单的事件处理器（在遵循`W3C`标准的浏览器使用`addEventListener`，或者在旧的`IE`使用`attachEvent`），然后分发通过`jQuery`的`API`绑定的事件处理器。比如，如果三个`click`事件处理器绑定到一个元素上，`jQuery`在第一个处理器绑定的时候会添加紫的的事件处理器，然后将用户的事件处理器添加到一个列表，在事件触发的时候执行。对于剩下的事件处理器，`jQuery`只添加到他们内部的列表，因为它已经调用浏览器绑定了它唯一的处理器。反过来说，当特定类型的事件最终从元素中被删除的时候，`jQuery`从浏览器移除它自己的事件处理器。

一个事件可以是`W3C`定义的原生事件，为了响应一些东西，比如用户点击一个鼠标按钮或者按下一个键，浏览器将会触发它。也可以是一个自定义事件，只能通过`.trigger()`或者`.triggerHandler()`方法来触发它。代码也可以触发原生浏览器事件，可以很方便的模拟用户动作。

通常来说，`jQuery`内部没有关于一个事件名称是否会被浏览器触发的知识。所以默认情况下，`jQuery`总是会在`API`调用添加此类事件的事件处理器的时候绑定事件到浏览器。如果这个事件类型浏览器从来不会生成，那么唯一调用这个处理器的方式就是`JavaScript`代码调用激活这个事件。尽管绑定一个没有用的事件名称到浏览器是无害的，还是可以使用下面描述的`setup`钩子覆盖默认行为。

当元素通过`jQuery`被移出`document`的时候，事件系统将尝试保证事件和它关联的数据从元素上被移除以放置内存泄漏。（老版本的IE浏览器因为这种场景下因没有仔细管理而内存泄漏而臭名昭著）如果一个事件扩展绑定事件或者创建新对象，它在事件被移除的时候，应该在`remove`或者`teardown`钩子中被解绑和清理。

`jQuery`事件扩展开发者应该比变使用在`DOM`设置中有特殊意义的事件名。类似`click`、`change`或者`load`事件名称在`W3C`定义中有特殊的语义，所以使用他们作为自定义事件会导致非预期的行为。通常，`jQuery`事件扩展只能在为了跨浏览器行为一致的时候才使用`W3C`定义的事件名。一个避免冲突简单快捷的方法是在事件类型名称添加一个冒号或者破折号，因为`W3C`不使用这些符号



### `jQuery.event.props: Array`
当事件发生的时候，`jQuery`定义了一个事件对象，它表示了一系列跨浏览器信息子集。`jQuery.event.props`属性是一个属性名称字符串数组，它总是在`jQuery`处理原生浏览器事件的时候被复制。（`.trigger()`触发的时间不会使用这个列表，因为它会构造一个包含所需值的`jQuery.Event`对象并使用该对象触发事件。）


可以使用`jQuery.event.props.push('newPropertyName')`来添加一个属性名到列表中。然而，需要注意的是`jQuery`在处理每个事件的时候将会尝试从原生浏览器事件复制该属性到`jQuery`构造的事件。如果事件类型不存在该属性。它将会得到`undefined`。添加太多属性到这个列表将会降低事件分发性能，所以，使用频次低的属性使用`event.originalEvent`来直接访问更加有效。
如果一个属性必须被复制，你最好使用`1.7`版本的`jQuery.event.fixHooks`。

### `jQuery.event.fixHooks: Object`

`jQuery`在处理原生浏览器事件的时候，`fixHook`接口提供了一个针对每个事件类型的方式来扩展和规范事件对象。一个`fixHook`实体是一个对象，它拥有两个可选属性：


- `props:Array`: 表示应该从浏览器事件对象中复制到`jQuery`事件对象的属性字符串列表。如果缺省，除了标准的属性，将不会有额外的属性被`jQuery`复制和规范化（比如：`event.target`和`event.relatedTarget`）。

- `filter: Funciton(event, originalEvent)`：`jQuery`在构造`jQuery.Event`对象之后调用这个函数，从`jQuery.event.props`中复制标准属性，也从前面的醒的`fixHooks`的`props`复制。这个函数将会在事件对象上创建一个新的属性或者修改已经存在的属性。第二个参数是浏览器原生事件对象，也就是`event.originalEvent`。

注意：对于所有事件，浏览器原生事件兑现个都可以通过`event.originalEvent`访问，如果`jQuery`事件`jQuery`事件处理器明确要覆盖`jQuery`规范事件对象的属性，没哟必要创建一个`fixHook`实体去创建或者修改属性。

比如，为`drop`事件设置一个`hook`，复制`dataTrannsfer`属性，为`jQuery.event.fixHooks.drop`赋值一个对象：
```
jQuery.event.fixHooks.drop = {
    props: [ "dataTransfer" ]
};
```
因为`fixHooks`是一个高级特性，一般在内部使用，`jQuery`没有包含解决冲突的解决方案的代码或者接口。如果存在其他代码可能赋值`fixHooks`到相同事件的奉新，应该检测是否存在`hook`并采取适当的解决方案。一个简单的解决方案可能看起来像这样：
```
if ( jQuery.event.fixHooks.drop ) {
    throw new Error( "Someone else took the jQuery.event.fixHooks.drop hook!" );
}
 
jQuery.event.fixHooks.drop = {
    props: [ "dataTransfer" ]
};
```
当知道其他不同的插件将会绑定`drop hook`，这个解决方案可能更适合：
```
var existingHook = jQuery.event.fixHooks.drop;
 
if ( !existingHook ) {
    jQuery.event.fixHooks.drop = {
        props: [ "dataTransfer" ]
    };
} else {
    if ( existingHook.props ) {
        existingHook.props.push( "dataTransfer" );
    } else {
        existingHook.props = [ "dataTransfer" ];
    }
}
```
### 特殊事件钩子
`jQuery`特殊事件钩子是一系列以每个事件名为属性名的函数。允许`jQuery`代码控制事件行为。这个机制和`fixHooks`很类似，当时特殊事件的信息被存储到`jQuery.event.special.NAME`，其中`NAME是特殊事件的名称，事件名称是大小写敏感的。

就像`fixHooks`，特殊事件钩子的设计是基于假设两个不想管的代码部分想要处理相同的事件名是非常罕见的。特殊是假作者去修改已经存在钩子的事件需要采取预防措施去防止引入不需要的副作用。

### `noBubble: Boolean`
指示事件被`.trigger()`方法触发的时候是否冒泡；默认情况下是`false`，意味着被触发的事件将会冒泡到元素的副元素知道`document`（如果绑定到了`document`）然后到`window`。注意在事件上定义`noBubble`将会阻止事件被代理事件触发。

### `bindType: String`，`delegateType: String`
定义时，这些字符串属性指定一个特殊的事件将被处理成其他事件类型，直到事件被传递。`bindType`用于事件被直接绑定，`delegateType`用于事件代理。这些事件都是`DOM`事件类型，但是不因该被指定为事件本身。

这些属性的行为用一个栗子来解释最简单。假设一个特殊的事件被这么定义：
```
jQuery.event.special.pushy = {
    bindType: "click",
    delegateType: "click"
};
```
当这些属性被定义，`jQuery`事件系统就会出现下面的行为：

- `pushy`事件的事件处理器将会实际绑定`click`-不管是直接绑定还是代理事件。
- `click`的特殊事件钩子如果存在，将会被调用，如果存在事件，`pushy`的钩子将会在事件被传递的时候调用。
- `pushy`的事件处理器只能使用`pushy`事件名称来移除，如果相同元素上的`click`事件被移除，不会对他造成影响。
针对上面的特殊事件，下面的代码显示`push`在移除点击的时候不会被移除。这可能是防范不良行为插件的有效方式，即使没有为移除点击事件赋予命名空间。

```
var elem = $( "p" );
 
elem.on( "click", function( event ) {
    $( "body" ).append( "I am a " + event.type + "!" );
});
 
elem.on( "pushy", function( event ) {
    $( "body" ).append( "I am pushy but still a " + event.type + "!" );
});
 
elem.trigger( "click" ); // Triggers both handlers
 
elem.off( "click" );
 
elem.trigger( "click" ); // Still triggers "pushy"
 
elem.off( "pushy" );
```
这两个属性通常和处理器钩子函数一起使用；比如，钩子可能在调用事件处理器之前将事件名称从`click`改为`pushy`。看下面的栗子。
### `handleObj`对象
前面的很多特殊事件钩子函数都被传递了一个`handleObj`对象，该对象宝航了事件的很多信息，怎样被绑定的，当前的状态。这个对象应该被当作是只读的数据，只有前面记录的属性被特殊的事件处理器使用。在前面的讨论中，假设一个事件被如此绑定：

```
$( ".dialog" ).on( "click.myPlugin", "button", {
    mydata: 42
}, myHandler );
```
`type: String`：事件类型，比如`click`。当特殊事件使用`bindType`或者`delegateType`映射的时候，将会是映射的类型。

`origType: String`：原始的类型名称（在这个栗子中是`click`）无视它是否通过`bindType`或者`delegateType`映射。所以当一个`pushy`事件被映射成`click`的时候，它的`origType`将会是`pushy`。了解更多详情可以看前面这些特殊事件属性的栗子。

`namespace: String`：命名空间在事件绑定的时候提供，比如`myPlugin`。当有许多的命名空间的时候，他们将按句点符号分割并按照字母顺序排列，如果没有，则该属性是空字符串。

`selector: String`：对于代理事件，这个选择器用来过滤子孙元素，来判定处理器是否应该被调用。在这个栗子中是`button`。对于直接绑定事件来说，这个属性是`null`。

`data: Objext`：事件绑定的时候传递过来的数据，比如`{ myData: 42 }`，如果数据参数是缺省的或者`undefined`，这个属性也是`undfined`。

`handler: function( event: jQuery.Event )`：当`jQuery`处理事件绑定的时候传入的事件处理器函数；在这个栗子中是`myHandler`的引用。如果事件绑定的时候传入的是`false`，处理器指向一个独立的分享的函数，它简单的返回`false`。

### `setup: function( data: Object, namespaces, eventHandle: function )`
当一个特定类型的事件第一次绑定到元素上的时候，`setup`钩子将被调用。这个钩子提供一个去处理所有这个元素上这种类型的事件的机会。`this`关键字指向被绑定事件的元素，`eventHandle`是`jQuery`的事件处理函数。在大部分场景中，`namespace`参数不应该被使用，因为它只是表示第一个事件被绑定的`namespace`；剩下的事件并不一定会使用这个`namespace`。

这个钩子可以执行任何想要的东西，包括将它自己的事件处理器绑定到元素或者其他元素，使用`jQuery.data()`方法在元素上记录设置信息。如果`setup`钩子想要`jQuery`添加一个浏览器事件（通过`addEventListener`或者`attachEvent`，这取决于浏览器），它应该返回`false`。在其他所有的情况下，`jQuery`将不会添加浏览器事件，但是会继续为该事件记录。这是适合的，比如，如果事件从不会被浏览器激活但是被用`.trrigger()`调用。要绑定`jQuery`事件处理器到`setup`钩子，请使用`eventHandle`参数

### `teardown: function()`
`teardown`钩子在最后一个特定类型事件从一个元素移除的时候被调用。`this`关键字指向事件被清理的元素。如果想要`jQuery`从浏览器的事件系统移除事件，这个钩子应该返回`false`（通过`removeEventListener`或者`detachEvent`）。在大部分情况下，`setup`和`teardown`钩子应该返回相同的值。

如果`setup`钩子绑定事件处理器或者通过像`jQuery.data()`之类的机制添加数据到元素，那么`teardown`钩子就应该反过来移除他们。`jQuery`通常会在元素完全从`document`移除的时候移除数据和事件，但是如果元素还在`document`上，那么`teardown`中将无法移除`数据或者事件，这将导致内存泄漏。

### `add: function( handleObj )`

每次通过`.on()`之类的方法添加一个事件处理器到一个元素的时候，`jQuery`就会调用这个钩子。`this`关键字将会指向将被添加事件处理器的元素，`handleObj`参数就是之前章节描述的。这个钩子的返回值将会被忽略。


### `remove: function( handleObj )`
当使用`.off()`之类的`API`从元素上移除一个事件处理器的时候，这个钩子将会被调用。`this`关键字将会指向处理器被移除的元素，`handleObj`参数就是之前章节所描述的。这个钩子的返回值将会被忽略。

### `trigger: function( event: jQuery.Event, data: Object )`
当`.trigger()`或者`.triggerHandle()`方法在代码中被用来触发特定类型的是事件，而不是源自浏览器事件的时候将会被调用。`this`关键字指向被触发的元素，`event`参数由`jQuery.Event`对象构造，从调用者输入。至少，`event`拥有事件类型、数据、命名空间和目标等属性。`data`参数代表`.trigger()`调用的时候传入的数据。

`trigger`钩子在触发事件的过程中调用的比较早，就在`jQuery.Event`构造完成之后，任何事件处理器调用之前。他可以用任何方式触发事件，比如在返回前调用`event.stopPropagation()`或者`event.preventDefault()`。如果这个钩子返回`false`。`jQuery`将不会嗲用任何触发动作，并且立马返回。否则，他执行正常的触发流程，调用元素的事件处理器，将事件冒泡（直到特定事件的扩展被停止，或者`noBubble`被指定）。并调用绑定父元素的事件处理器。

### `_default: function( event: jQuery.Event, data: Object )`
当`.trigger()`方法执行事件的所有事件处理器的时候，也会寻找并执行`target`对象上的任何同名方法，直到处理器调用`event.preventDefault()`。所以`.trigger("submit")`将会执行元素的`submit()`方法，如果这个方法存在。如果`_default`钩子被指定，这个钩子将会在检查执行元素默认方法之前执行。如果这个钩子返回`false`，元素默认方法将会执行，否则不执行。


### `handle: function( event: jQuery.Event, data: Object )`
`jQuery`在事件发生的时候会调用`handle`方法。也会正常调用由`.on()`或者其他事件绑定方法指定的事件处理器。如果这个钩子存在，`jQuery`将会调用这个钩子，而不是事件处理器。如果不是一个原生事件，该事件和通过`trigger()`传递进来的数据将会传递到这个钩子。`this`关键之执行被处理的`DOM`元素，`event.handleObj`属性拥有事件的详细信息。

基于他拥有的信息，`handle`钩子应该决定是否调用位于`event.handleObj.handle`的源处理器函数。在调用源处理器的时候，可以修改`event`对象的信息，但是在返回之前必须恢复数据，否则接下来不相关的事件处理器的行为将无法预测。在大部分场景中，`handle`钩子应该返回源处理器的结果，但是这决定于钩子。`handle`钩子的特殊之处在于当使用`bindType`和`delegateType`映射，他在源特殊事件名被调用，是唯一的特殊事件函数钩子。因为这个原因，如果特殊事件定义了`bindType`和`delegateType`
，那么除了`handle`钩子，其他钩子总是错误的，因为其他钩子将不会被调用。
### 栗子：多次点击事件
这个`multiclick`特殊事件将自己映射成标准的`click`事件，但是使用`handle`钩子，这样他就可以监控事件，在只有用户多次点击一个元素到达指定次数的时候，才会分发事件。

这个钩子在`data`对象中存储的当前的点击次数统计。所以`multiclick`处理器绑定到不同元素将不会互相影响。他在调用处理器之前改变事件类型为`multiclick`并在返回之前恢复为`click`：
```
jQuery.event.special.multiclick = {
    delegateType: "click",
    bindType: "click",
    handle: function( event ) {
        var handleObj = event.handleObj;
        var targetData = jQuery.data( event.target );
        var ret = null;
 
        // If a multiple of the click count, run the handler
        targetData.clicks = ( targetData.clicks || 0 ) + 1;
 
        if ( targetData.clicks % event.data.clicks === 0 ) {
            event.type = handleObj.origType;
            ret = handleObj.handler.apply( this, arguments );
            event.type = handleObj.type;
            return ret;
        }
    }
};
 
// Sample usage
$( "p" ).on( "multiclick", {
    clicks: 3
}, function( event ) {
    alert( "clicked 3 times" );
});
```