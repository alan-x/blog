### 组件方法调用
组件工厂创建的组件在初始化后使用方法去改变他们的状态执行动作。有两种方法可以被执行-通过组件工厂穿过插件，或者在元素实例对象上调用方法。

### 插件调用
为了使用组件插件调用方法，传递方法名作为字符串。比如，这是调用弹窗组件的`.close()`方法。
```
$( ".selector" ).dialog( "close" );
```
如果方法需要参数，传递他们作为额外的参数到插件，这是调用弹窗的`option()`方法。
```
$( ".selector" ).dialog( "option", "height" );
```
这返回弹窗的高度配置的值。
### 实例调用
在面纱之下，每一个组件的实例通过`jQuery.data()`被存储在元素上。为了抓住对象实例，`jQuery.data()`使用了组件的全部名称作为键。如下：
```
var dialog = $( ".selector" ).data( "ui-dialog" );
```
在你拥有了实例对象的引用之后，方法可以被直接调用
```
var dialog = $( ".selector" ).data( "ui-dialog" );
dialog.close();
```
在`jQuery UI 1.11`，新的`instance()`方法将会让操作更加简单。
```
$( ".selector" ).dialog( "instance" ).close();
```
### 返回类型
大部分通过组件插件调用的方法将会返回一个 jQuery 对象，所以方法调用可以链上其他额外的 jQuery 方法。就算在实例上调用方法返回`undefined`也是这样。如下：
```
var dialog = $( ".selector" ).dialog();
 
// Instance invocation - returns undefined
dialog.data( "ui-dialog" ).close();
 
// Plugin invocation - returns a jQuery object
dialog.dialog( "close" );
 
// Therefore, plugin method invocation makes it possible to
// chain method calls with other jQuery functions
dialog.dialog( "close" )
    .css( "color", "red" );
```
返回组件信息的方法是一个例外。比如弹窗`isOpen()`方法。
```
$( ".selector" )
    .dialog( "isOpen" )
    // This will throw a TypeError
    .css( "color", "red" );
```
这里产生了一个错误，`isOpen()`返回一个布尔，不是一个 jQuery 对象。