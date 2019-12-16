Custom Effects with .animate()
### 使用`.animate()`自定义效果
jQuery makes it possible to animate arbitrary CSS properties via the .animate() method. The  .animate() method lets you animate to a set value, or to a value relative to the current value.
通过`.animate()`方法，`jQuery`可以为任意的`CSS`属性添加效果。`.animate()`让你的效果到某个设定的值或者相对于当前的值。
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
13
14
15
```
// Custom effects with .animate()
$( "div.funtimes" ).animate(
    {
        left: "+=50",
        opacity: 0.25
    },
 
    // Duration
    300,
 
    // Callback to invoke when the animation is finished
    function() {
        console.log( "done!" );
    }
);
```
Note: Color-related properties cannot be animated with .animate() using jQuery out of the box. Color animations can easily be accomplished by including the color plugin. We'll discuss using plugins later in the book.
注意：颜色相关的属性无法通过`jQuery`自带的`.animate()`添加效果。颜色动画可以通过添加颜色插件简单的完成。我们将在本书后面讨论插件的使用。

link Easing
### 缓动
Definition: Easing describes the manner in which an effect occurs — whether the rate of change is steady, or varies over the duration of the animation. jQuery includes only two methods of easing: swing and linear. If you want more natural transitions in your animations, various easing plugins are available.
定义：`Easing`描述的效果的发生方式-变化率是稳定的还是在持续时间内变化的。`jQuery`只有两种缓动方式：`swing`和`linear`。如果你想要更多自然的过渡，大量的缓动插件可以使用。

As of jQuery 1.4, it is possible to do per-property easing when using the .animate() method.
`jQuery 1.4`以后，在`.animate()`中，可以为每个属性设置缓动。

```
// Per-property easing
$( "div.funtimes" ).animate({
    left: [ "+=50", "swing" ],
    opacity: [ 0.25, "linear" ]
}, 300 );
```
For more details on easing options, see Animation documentation on api.jquery.com.
了解更多缓动选项，可以查阅[api.jquery.com 上的动画文档]()。

