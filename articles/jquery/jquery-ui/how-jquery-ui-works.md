### jQuery UI 是怎样工作的
jQuery UI 包含很多的保持状态的组件，因此使用模式和传统的 jQuery 插件有一些不同，你已经习惯了。尽管和大部分 jQuery 插件的初始化方式差不多，jQuery UI 组件构建在组件工厂之上，它为他们所有都提供了通用的相同的 API。如果你学会了使用一个，你就直到怎样使用全部。这个文档建辉带你领略基本的功能，使用进度条组件作为代码栗子。

### 初始化
为了跟踪组件的状态，我们必须引入完整的组件生命周期。生命周期在组件初始化的时候开始，为了初始化一个组件，我们简单的在一个或者多个元素上调用组件。

```
$( "#elem" ).progressbar();
```
这将在会 jQuery 对象中初始化每一个元素，在这个场景中，id 为“elem”的元素。因为我们调用无参的`.progressbar()`方法，组件将会使用默认的配置初始化。我们可以在初始化的过程中传递一系列的配置去覆盖默认的配置。
```
$( "#elem" ).progressbar({ value: 20 });
```
我们可以在初始化的过程中尽可能多或者少的配置。任何我们没有传递的配置都使用默认配置。
配置是组件状态的一部分，所以我们也可以在初始化后设置配置。我们可以在后面的配置防范看到。

### 方法
现在组件已经被初始化了，我们可以查询它的状态或者在组件上执行操作。所有的动作在初始化之后表现为方法调用。为了在组件上调用防范，我们船渡方法的名称给组件插件。比如为了在 我们的 progressbar 组件上调用 value 方法，我们可以使用：

```
$( "#elem" ).progressbar( "value" );
```
如果方法接收参数，我们可以在方法名后面传递他们。比如传递 40 这个参数给 value 方法，我们可以这么做：
```
$( "#elem" ).progressbar( "value", 40 );
```
就像 jQuery 中的其他方法，大部分的组件方法返回 jQuery 对象用于链式。

```
$( "#elem" )
    .progressbar( "value", 90 )
    .addClass( "almost-done" );
```
普通方法
每一个组件有他们自己基于组件提供功能的方法。然而，有一些方法存在于所有的组件。

### 配置
就像我们之前提到的，我们可以在初始化之后通过`option`方法改变选项。比如，我们可以通过`option`方法将进度条的值改变为30。
```
$( "#elem" ).progressbar( "option", "value", 30 );
```
注意这和之前的我们调用的`value`方法的栗子不同。在这个栗子中，我们调用`option`方法并告诉他我们想要改变`value`配置为30。
我们也可以得到配置当前的值。
```
$( "#elem" ).progressbar( "option", "value" );
```
此外，我们可以更新多个选项，通过一次性传递一个对象到`option`方法。
```
$( "#elem" ).progressbar( "option", {
    value: 100,
    disabled: true
});
```
你可能注意到`option`方法和 jQuery 核心 getter 和 setter 有相同的签名，比如`.css()`和`.attr()`。唯一的不同就是你需要传递字符串“option”作为第一个参数。
link disable
### disable
就像你可能猜测的，`disable`方法禁用组件。在进度条的栗子中，它改变样式让进度条看起来被禁用。
```
$( "#elem" ).progressbar( "disable" );
```
调用`diable`方法和设置`disabled`配置效果一样。
link enable
### enbale
`enable`方法是`disable`方法的对立。
```
$( "#elem" ).progressbar( "enable" );
```
调用`enable`方法和设置`disabled`配置为`false`效果一样。

### destory
如果你不需要组件，你可以摧毁它并还原成原始的标签。这结束了组件的生命周期。
```
$( "#elem" ).progressbar( "destroy" );
```
一旦你摧毁了一个元素，你就没有办法调用任何方法，直到你再一次初始化组件。如果你移除了元素，无论是直接通过`.remove()`或者使用`.html()`或`.empty()`修改祖先元素，元素将会自动摧毁它自己。
### widget
一些组件生成包裹元素，或者元素和原始元素断开链接，在这个栗子中，`widget`方法将会放回生成的元素。在类似进度条的场景中，没有生成包装，`widget`方法返回元素元素。
```
$( "#elem" ).progressbar( "widget" );
```
link Events
### 事件
所有的组件都有事件，和他们的各种在状态改变的时候通知你的行为关联。对于大部分组件来说，当事件被触发，组件名将作为名字的前缀。比如，我们可以绑定进度条的`change`事件，当`value`被改变的时候就会触发它。

```
$( "#elem" ).bind( "progressbarchange", function() {
    alert( "The value has changed!" );
});
```
m一个事件都有代表的函数，暴露有选项。我们可以勾住进度条的改变回调，而不是绑定`progressbarchange`事件，如果我们想要。
```
$( "#elem" ).progressbar({
    change: function() {
        alert( "The value has changed!" );
    }
});
```
普通事件
尽管大部分事件都是组件指定的，所有的组件都有`create`事件。这个事件将会在组件创建的时候立刻触发。