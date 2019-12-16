### 使用组件工厂扩展组件
jQuery UI 的组件工厂让构建扩展已存在的组件的组件变得很简单。这么做允许你在存在的基础上构建威力强大的组件，也可以稍微改动已存在的组件的功能。

注意：这篇文章假设你直到一些组件工厂的工作原理和组件的工作方式。如果你不熟悉，先阅读怎样是使用组件工厂。

#### 创建组件扩展
使用组件工厂创建组件是通过传递组件名称和原型对象给`$.widget()`来完成的。下面在“custome”命名空间创建了一个“superDialog”组件。
```
$.widget( "custom.superDialog", {} );
```
为了允许扩展，`$.widget()`可选的接收一个组件的构造函数作为父母。为了指定一个父级组件，传递第二个参数-在组件名称之后，在组件的原型对象之前。
就像上一个栗子，下面在“custom”命名空间下创建了一个“superDialog”组件。然而，这一次 jQuery UI 的弹窗组件（`$.ui.dialog`）的构造函数被船渡，指示 superDialog 组件应该使用 jQuery UI 的弹窗组件作为父级组件。

```
$.widget( "custom.superDialog", $.ui.dialog, {} );
```
这里 superDialog 和弹窗使用不同的名称和命名空间构建了相同的组件。为了让我们的组件更加有趣，我们可以添加方法到原型对象。

组件原型对象是最后一个传递给`$.widget()`的参数。所以，饿哦们的离职使用了空对象。让我们添加一些方法到原型对象。
```
$.widget( "custom.superDialog", $.ui.dialog, {
    red: function() {
        this.element.css( "color", "red" );
    }
});
 
// Create a new <div>, convert it into a superDialog, and call the red() method.
$( "<div>I am red</div>" )
    .superDialog()
    .superDialog( "red" );
```
现在，superDialog 有`red()`方法将会改变文字的颜色为红色。注意组件工厂是怎样设置`this`到组件实例对象的。要了解实例所有的方法和属性，查阅组件工厂的 API 文档。

### 扩展存在的方法。
有时候我们需要去修改或者添加已存在组件方法的行为。为了做到这个，在原型对象中指定一个和你想要覆盖的放的相同名字的方法。下面的栗子覆盖了弹窗的`open()`方法。因为弹窗默认自动打开，当代码运行的时候“open”将会被记录。
```
$.widget( "custom.superDialog", $.ui.dialog, {
    open: function() {
        console.log( "open" );
    }
});
 
// Create a new <div>, and convert it into a superDialog.
$( "<div>" ).superDialog();
```
尽管可以运行，这有一个问题。因为我们覆盖了`open()`默认的行为，弹窗不太显示在屏幕上。
当我们将方法放在原型对象中的时候，我们没有真的覆盖原本的方法-而是放置了一个新的方法在更高的原型链层级层级。
为了让父级方法可用，组件工厂提供了两个方法-`_super()`和`_superApply()`。
### 使用`_super()`和`_superApply()`去访问父级。
`_super()`和`_superApply()`调用父组件相同的方法。查看下面的栗子。就像前一个，这个栗子覆盖了`open()`方法并记录“open”。然而，这一次`_super()`方法调用了弹窗的`open()`方法并打开了弹窗。
```
$.widget( "custom.superDialog", $.ui.dialog, {
    open: function() {
        console.log( "open" );
 
        // Invoke the parent widget's open().
        return this._super();
    }
});
 
$( "<div>" ).superDialog();
```
`_super()`和`_superApply`被设计为表现的像原生的`Function.prototype.call()`和`Function.prototype.apply()`方法。然而，`_super()`接收参数列表，`_superApply`接受单个数组参数。这个不同在下面的栗子显示。
```
$.widget( "custom.superDialog", $.ui.dialog, {
    _setOption: function( key, value ) {
 
        // Both invoke dialog's setOption() method. _super() requires the arguments
        // be passed as an argument list, _superApply() as a single array.
        this._super( key, value );
        this._superApply( arguments );
    }
});
```
### 重新定义组件
jQuery UI 1.9 添加了重新定义组件的能力。然而，不是创建一个新的组件，我们传递`$.widget()`一个存在的组件方法和构造函数。瞎买呢的栗子在`open()`添加相同的日志，但是不是创建一个新的组件。
```
$.widget( "ui.dialog", $.ui.dialog, {
    open: function() {
        console.log( "open" );
        return this._super();
    }
});
 
$( "<div>" ).dialog();
```
使用这种方法可以扩展存在的组件方法，并可以访问原始的`_super()`方法-所有都不需要创建新的组件。
### 组件和垫片
当组件扩展和他们的插件交互的时候，有一个警告。父组件的插件不能在元素上执行防范，因为他是子组件。栗子显示如下：
```
$.widget( "custom.superDialog", $.ui.dialog, {} );
 
var dialog = $( "<div>" ).superDialog();
 
// This works.
dialog.superDialog( "close" );
 
// This doesn't.
dialog.dialog( "close" );
```
上面。父级组件插件`dialog()`无法在 superDialog 的元素上调用`close()`方法。了解更多组件方法，查阅[组件方法调用]()。
### 自定义独立实体
我们看过的所有的栗子都从组件的原型继承方法。方法覆盖原型会影响所有组件的实例。
为了显示这个，查看下面的栗子；所有的 dialog 实例都使用相同的`open()`方法。
```
$.widget( "ui.dialog", $.ui.dialog, {
    open: function() {
        console.log( "open" );
        return this._super();
    }
});
 
// Create two dialogs, both use the same open(), therefore "open" is logged twice.
$( "<div>" ).dialog();
$( "<div>" ).dialog();
```
尽管这很强大，有时候你只需要去改变单个组件实例的行为。为了做到这个，绑定一个实例的引用，并使用普通的 JavaScript 属性赋值。如下显示：
```
var dialogInstance = $( "<div>" )
    .dialog()
 
    // Retrieve the dialog's instance and store it.
    .data( "ui-dialog" );
 
// Override the close() method for this dialog
dialogInstance.close = function() {
    console.log( "close" );
};
 
// Create a second dialog
$( "<div>" ).dialog();
 
// Select both dialogs and call close() on each of them.
// "close" will only be logged once.
$( ":data(ui-dialog)" ).dialog( "close" );
```
这种为独立的实体覆盖方法的技术对单次自定义是完美的。
