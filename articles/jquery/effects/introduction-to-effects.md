### 动效介绍
### 显示隐藏内容
`jQuery`的`.show()`和`.hide()`方法可以瞬间隐藏内容。
```
// Instantaneously hide all paragraphs
$( "p" ).hide();
 
// Instantaneously show all divs that have the hidden style class
$( "div.hidden" ).show();
```
当`jQuery`隐藏元素的时候，它设置它的`CSS`的`display`属性为`none`。这意味着，内容将会有0的宽度和高度；它不意味着内容只是简单的变透明，并留下一个空的区域在页面上。
`jQuery`也可以在显示和隐藏内容的时候添加动画效果。你有两种方式告诉`.show`和`.hide()`使用动画。一是传递`slow`、`normal`或者`fast`作为参数：
```
// Slowly hide all paragraphs
$( "p" ).hide( "slow" );
 
// Quickly show all divs that have the hidden style class
$( "div.hidden" ).show( "fast" );
```
如果你想要更直接的控制动画效果的持续事件，你可以传递毫秒作为持续时间给`.show()`和`.hide()`：
 ```
// Hide all paragraphs over half a second
$( "p" ).hide( 500 );
 
// Show all divs that have the hidden style class over 1.25 seconds
$( "div.hidden" ).show( 1250 );
```
大部分开发者都会传递一个毫秒的数字，为了更加精细的控制持续时间。
### 淡化和滑动动画
你可能注意到`.show()`和`.hide()`使用滑动和淡化的混合效果来显示和隐藏内容。如果你更愿意显示和隐藏都使用一种效果或者其他，有更多的方法可以达到。`.slideDown()`和`.slideUp()`显示和隐藏内容，各自都只使用滑动效果。滑动效果通过快速的改变元素的`CSS`的`height`属性来完成。
```
// Hide all paragraphs using a slide up animation over 0.8 seconds
$( "p" ).slideUp( 800 );
 
// Show all hidden divs using a slide down animation over 0.6 seconds
$( "div.hidden" ).slideDown( 600 );
```
`fadeIn()`和`fadeOut`也类似的，各自通过淡化动画显示和隐藏元素。淡化动哈通过快速的修改元素的`CSS`的`opactiy`属性来实现。
```
// Hide all paragraphs using a fade out animation over 1.5 seconds
$( "p" ).fadeOut( 1500 );
 
// Show all hidden divs using a fade in animation over 0.75 seconds
$( "div.hidden" ).fadeIn( 750 );
```
### 通过当前的可视状态改变显示
`jQuery`允许你根据内容当前的可视状态来改变内容的可视状态。`.toggle()`将会在当前内容隐藏的时候显示内容。你可以传递给`.toggle()`和之前效果方法相同的参数。
```
// Instantaneously toggle the display of all paragraphs
$( "p" ).toggle();
 
// Slowly toggle the display of all images
$( "img" ).toggle( "slow" );
 
// Toggle the display of all divs over 1.8 seconds
$( "div" ).toggle( 1800 );
```
`.toggle()`将会混合使用滑动和淡化效果，就像`.show()`和`.hide()`。你可以只使用滑动或者淡化来切换内容的状态，通过使用`.slideToggle()`和`.fadeToggle()`。
```
// Toggle the display of all ordered lists over 1 second using slide up/down animations
$( "ol" ).slideToggle( 1000 );
 
// Toggle the display of all blockquotes over 0.4 seconds using fade in/out animations
$( "blockquote" ).fadeToggle( 400 );
```
### 在动画完成之后做些什么
在实现`jQuery`的动画的时候有一个常见的误会，那就是假设你的链中下一个方法将会等到动画执行完成才执行
```
// Fade in all hidden paragraphs; then add a style class to them (not quite right)
$( "p.hidden" ).fadeIn( 750 ).addClass( "lookAtMe" );
```
要意识到前面的`.fadeIn()`只是启动了动画是很重要的。一旦开始，动画通过`JavaScript`的`setInterval()`循环快速修改`CSS`属性实现的。当你调用`.fadeIn()`的时候，它开始动画循环然后返回`jQuery`对象，将它传递给添加`lookAtMe`样式类的`.addClass()`的时候动画循环刚刚开始。
如果想要在动画运行完成之后才执行动作，你需要使用动画的回调函数。你可以指定动画的回调作为第二个参数传递给再买呢提到的任何动画方法。对于前面的代码端，我们可以像这样实现回调：
```
// Fade in all hidden paragraphs; then add a style class to them (correct with animation callback)
$( "p.hidden" ).fadeIn( 750, function() {
    // this = DOM element which has just finished being animated
    $( this ).addClass( "lookAtMe" );
});
```
注意你可以使用`this`关键字来引用正在动画的`DOM`元素。也要注意回调将会在`jQuery`对象的每个元素上被调用。这意味着如果你的选择器表达式没有任何元素，你的动画回调将不会执行！你可以通过测试你的选择器是否返回元素来解决这个问题；如果没有，你可以立刻运行回调。
```
// Run a callback even if there were no elements to animate
var someElement = $( "#nonexistent" );
 
var cb = function() {
    console.log( "done!" );
};
 
if ( someElement.length ) {
    someElement.fadeIn( 300, cb );
} else {
    cb();
}
```
### 管理动画效果
`jQuery`提供了一些额外的特性来控制你的动画：
### `.stop()`
`.stop()`将会马上终止你的选择器中所有元素的正在执行的动画。你可以通过提供给最终用户一个可以点击的按钮让那个他们控制页面动画。
```
// Create a button to stop all animations on the page:
$( "<button type='button'></button>" )
    .text( "Stop All Animations" )
    .on( "click", function() {
        $( "body *" ).filter( ":animated" ).stop();
    })
    .appendTo( document.body );
```
### `.delay()`
`.delay()`用于在连续动画之间引入延迟
```
// Hide all level 1 headings over half a second; then wait for 1.5 seconds
// and reveal all level 1 headings over 0.3 seconds
$( "h1" ).hide( 500 ).delay( 1500 ).show( 300 );
```
### `jQuery.fx`
`jQuery.fx`对象有一系列属性可以控制好动画的实现。`jQuery.fx.speeds`映射前面提到的`slow`、`normal`、`fast`持续参数到一个指定的毫秒数字。默认的`jQuery.fx.speeds`值是：
```
{
    slow: 600,
    fast: 200,
    _default: 400 // Default speed, used for "normal"
}
```
你可以修改这些设置，设置设置一些你自己的：
```
jQuery.fx.speeds.fast = 300;
jQuery.fx.speeds.blazing = 100;
jQuery.fx.speeds.excruciating = 60000;
```
`jQuery.fx.interval`控制动画显示的每秒帧的数量。帧间距的默认值是13毫秒。你可以为更快的浏览器设置更低的值让动画更加平滑。然而每秒更多的帧对浏览器意味着更高的计算负载，所以你需要确保测试这么做的性能影响。

最后，`jQuery.fx.off`可以设置为`true`来禁止所有的动画，元素将立即设置为目标的最终状态。这对于处理旧的浏览器特别有用；你也可以为你的用户提供选项去禁用所有动画。
```
$( "<button type='button'></button>" )
    .text( "Disable Animations" )
    .on( "click", function() {
        jQuery.fx.off = true;
    })
    .appendTo( document.body );
```