Triggering Event Handlers
### 触发事件处理器
jQuery provides a way to trigger the event handlers bound to an element without any user interaction via the .trigger() method.
`jQuery`通过`.trigger()`方法提供了一个不需要用户交互就能触发元素绑定的事件处理器的方式。

link What handlers can be .trigger()'d?
### 什么处理器可以被`.trigger()`?
jQuery's event handling system is a layer on top of native browser events. When an event handler is added using .on( "click", function() {...} ), it can be triggered using jQuery's .trigger( "click" ) because jQuery stores a reference to that handler when it is originally added. Additionally, it will trigger the JavaScript inside the onclick attribute. The .trigger() function cannot be used to mimic native browser events, such as clicking on a file input box or an anchor tag. This is because, there is no event handler attached using jQuery's event system that corresponds to these events.
`jQuery`事件处理系统是在原生浏览器事件之上的一层封装。当一个事件处理器是使用`.on("click", function(){...})`添加的时候，它就可以被`jQuery`使用`.trigger("click")`触发，因为`jQuery`在它刚被绑定的时候就保存了引用。此外，它还会触发`onClick`属性中的`JavaScritpt`。`.trigger()`无法模仿原生浏览器事件，比如点击一个文件输入框，或者锚点标签。这是因为`jQuery`事件系统没有这些事件事件处理器。
1
```
<a href="http://learn.jquery.com">Learn jQuery</a>
```
1
2
```
// This will not change the current page
$( "a" ).trigger( "click" );
```
link How can I mimic a native browser event, if not .trigger()?
### 如果没有`.trigger()`，我要如何模拟原生浏览器事件？
In order to trigger a native browser event, you have to use document.createEventObject for < IE9 and document.createEvent for all other browsers. Using these two APIs, you can programmatically create an event that behaves exactly as if someone has actually clicked on a file input box. The default action will happen, and the browse file dialog will display.
为了触发一个原生浏览器事件，你需要在IE9以下的版本使用`document.createEventObject`，而在其他浏览器使用`document.createEvent`。使用这两个API，你可以编程创建一个事件，让他看起来就想有人点击了文件输入框一样。默认的行为将会发生，浏览器将会弹出一个文件选择框。

The jQuery UI Team created jquery.simulate.js in order to simplify triggering a native browser event for use in their automated testing. Its usage is modeled after jQuery's trigger.
`Jquery UI`团队创建了`jquery.simulate.js`，为了在他们的自动化测试中简化触发原生浏览器事件。他是按照`jQuery trigger`为模型实现的。

1
2
```
// Triggering a native browser event using the simulate plugin
$( "a" ).simulate( "click" );
```
This will not only trigger the jQuery event handlers, but also follow the link and change the current page.
这将不仅仅触发`jQuery`事件处理器，还会跟随链接并改变当前页面。

link.trigger() vs .triggerHandler()
### `.trigger()`和`.triggerHandler()`
There are four differences between .trigger() and .triggerHandler()
`.trigger()`和`.triggerHandler()`有四点不同：
.triggerHandler() only triggers the event on the first element of a jQuery object.
- `.triggerHandler()`只会触发`jQuery`对象的第一个元素的事件。
.triggerHandler() cannot be chained. It returns the value that is returned by the last handler, not a jQuery object.
- `.triggerHandler()`可以链式调用。它返回最后一个处理器的返回值，而不是`jQuery`对象。
.triggerHandler() will not cause the default behavior of the event (such as a form submission).
- `.triggerHandler()`将不会引起事件的默认行为（比如表单提交）。
Events triggered by .triggerHandler(), will not bubble up the DOM hierarchy. Only the handlers on the single element will fire.
- 被`.triggerHandler()`触发的事件，不会冒泡到`DOM`层级中。只会触发被激活的元素上的处理器。
For more information see the triggerHandler documentation
了解更多信息，可以查看[triggerHandler文档]()。
link Don't use .trigger() simply to execute specific functions
### 不要使用`.trigger()`去执行指定的函数。
While this method has its uses, it should not be used simply to call a function that was bound as a click handler. Instead, you should store the function you want to call in a variable, and pass the variable name when you do your binding. Then, you can call the function itself whenever you want, without the need for .trigger().
尽管这个方法可以这么用，它应该不仅仅为了简答的调用我们绑定成点击处理器的函数。你应该保存这个你想要调用的函数到变量中，并在你绑定的时候传递变量。然后，你可以在任何时候调用函数，而不需要`trigger()`。

1
2
3
4
5
6
7
8
9
10
11
12
```
// Triggering an event handler the right way
var foo = function( event ) {
    if ( event ) {
        console.log( event );
    } else {
        console.log( "this didn't come from an event!" );
    }
};
 
$( "p" ).on( "click", foo );
 
foo(); // instead of $( "p" ).trigger( "click" )
```
A more complex architecture can be built on top of trigger using the publish-subscribe pattern using jQuery plugins. With this technique, .trigger() can be used to notify other sections of code that an application specific event has happened.
使用[jQuery插件]()的[发布-订阅模式]()，可以在触发器之上构建跟家复杂的架构。使用这个技术，`.trigger()`可以用来通知其他章节的代码一个应用指定的事件发生了。

 Understanding Event Delegation