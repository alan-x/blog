### 入队和出队解释
队列是`jQuery`所有动画的基础功能，它允许一些列的函数在元素上异步执行。像`.slideUp()`、`.slideDown()`、`.fadeIn()`、`.fadeOut`的方法都使用`.animate()`，它操控队列创建一系列的不走，可以在整个动画的持续时间内变化一个或多个`CSS`值。

我们可以传递一个回调函数到`.animate()`方法，它将会在动画完成的时候调用。
```
$( ".box" )
    .animate( {
        height: 20
    }, "slow", function() {
        $( "#title" ).html( "We're in the callback, baby!" );
    } );
```
### 队列就像回调
我们可以添加其他的函数到队列中，而不是作为参数传递回调函数，这将表现的像我们的回调函数。它将会在所有的动画步骤完成之后执行。
```
$( ".box" )
    .animate( {
        height: 20
    }, "slow")
    .queue( function() {
        $( "#title" ).html( "We're in the animation, baby!" );
 
        // This tells jQuery to continue to the next item in the queue
        $( this ).dequeue();
    } );
```
在这个栗子中，传递到队列的函数将会在动画完成之后立马执行。

`jQuery`没有办法了解队列项函数的具体细节，所以我们需要调用`.dequeue()`告诉`jQuery`什么时候去移除队列中的下一项。

另一个出队的方式是调用传递给你的函数参数。这个函数将会自动为你调用`.dequeue()`。
```
.queue( function( next ) {
    console.log( "I fired!" );
    next();
} );
```
### 自定义队列
上面我们所有的动画和队列栗子都使用默认的队列名称，叫做`fx`。元素可以有多个队列绑定，，我们可以给每个队列一个不同的名字。我们可以指定一个自定义队列名字作为`.queue()`方法的第一个参数。
```
$( ".box" )
    .queue( "steps", function( next ) {
        console.log( "Step 1" );
        next();
    } )
    .queue( "steps", function( next ) {
        console.log( "Step 2" );
        next();
    } )
    .dequeue( "steps" );
```
注意我们需要传递我们自定义队列启动的名字给`.dequeue()`。每一个队列都有一个默认名字`fx`，调用`.dequeue()`的时候也会默认传递。

### 清理队列
因为队列只是一系列有序的操作，我们的应用程序可能有一些逻辑需要防止剩下的队列继续执行。我们可以调用`.clearQueue()`方法来清空队列。
```
$( ".box" )
    .queue( "steps", function( next ) {
        console.log( "Will never log because we clear the queue" );
        next();
    } )
    .clearQueue( "steps" )
    .dequeue( "steps" );
```
在这个栗子中，当我们从`steps`队列中移除所有东西的时候，任何事都不会发生。
其他情况队列的方式是调用`.stop(true)`。这将会停止当前的动画，并清理队列。
### 替换队列
当你传递了一个函数数组作为`.queue()`的第二个参数的时候，这个数组将会替代整个队列。
```
$( ".box" )
    .queue( "steps", function( next ) {
        console.log( "I will never fire as we totally replace the queue" );
        next();
    } )
    .queue( "steps", [
        function( next ) {
            console.log( "I fired!" );
            next();
        }
    ] )
    .dequeue( "steps" );
```
你可以调用`.queue()`不传递函数参数，它将会返回元素的队列数组。
```
$( ".box" ).queue( "steps", function( next ) {
    console.log( "I fired!" );
    next();
} );
 
console.log( $( ".box" ).queue( "steps" ) );
 
$( ".box" ).dequeue( "steps" );
```