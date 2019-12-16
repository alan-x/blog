### 高级插件概念
### 提供默认插件设置的公共访问权限
对于上面的代码，一个我们可以做，也应该做的提升是暴露默认的插件设置。这很重要，因为这让插件的用户使用最少的代码覆盖/自定义插件变得很简单。这就是我们开始使用函数对象的地方。

```
// Plugin definition.
$.fn.hilight = function( options ) {
 
    // Extend our default options with those provided.
    // Note that the first argument to extend is an empty
    // object – this is to keep from overriding our "defaults" object.
    var opts = $.extend( {}, $.fn.hilight.defaults, options );
 
    // Our plugin implementation code goes here.
 
};
 
// Plugin defaults – added as a property on our plugin function.
$.fn.hilight.defaults = {
    foreground: "red",
    background: "yellow"
};
```
现在用户可以在他们的脚本中添加一行代码，就像这样：
```
// This needs only be called once and does not
// have to be called from within a "ready" block
$.fn.hilight.defaults.foreground = "blue";
```
现在我们可以这样调用插件方法，它将会把前景色改为蓝色：

```
$( "#myDiv" ).hilight();
```
就像你看到的，我们允许用户写一行代码去修改插件默认的前景色。用户也可以可选的重写默认值为他们想要的：
```
// Override plugin default foreground color.
$.fn.hilight.defaults.foreground = "blue";
 
// ...
 
// Invoke plugin using new defaults.
$( ".hilightDiv" ).hilight();
 
// ...
 
// Override default by passing options to plugin method.
$( "#green" ).hilight({
    foreground: "green"
});
```
### 适当的提供独具第二功能的访问全县
这个栗子手把手的从前面的栗子过来，这是一个扩展你插件的有趣的方式（让其他人扩展你的插件）。比如，我们插件的实现可能定义了一个`format`方法，可以格式化`hilight`文本。我们的插件可能像这样，在`hilight`函数下定义`format`方法的默认实现：

```
// Plugin definition.
$.fn.hilight = function( options ) {
 
    // Iterate and reformat each matched element.
    return this.each(function() {
 
        var elem = $( this );
 
        // ...
 
        var markup = elem.html();
 
        // Call our format function.
        markup = $.fn.hilight.format( markup );
 
        elem.html( markup );
 
    });
 
};
 
// Define our format function.
$.fn.hilight.format = function( txt ) {
    return "<strong>" + txt + "</strong>";
};
```
我们也可以简单的在选项对象中提供其他属性支持回调函数去覆盖默认的格式化。那是另一种卓越的支持自定义你插件的方法。这里显示的技巧通过实际的暴露`format`函数，让它支持预定义。使用这个技术，其他人可以发送他们的自定义配置去覆盖你的参数-换句话说，这意味着其他人可以为你的插件写插件。

考虑到我们在这篇文章创建的小栗子，你可能会撕开这什么时候会有用。一个真实世界的栗子是[Cycle Plugin]()。`Cycle Plugin`是一个滑动显示的插件，它支持一系列内建的动画效果-滚动，滑动，淡化，等。但实际上，没有办法去定义每个人想要用于滑动变化的每种效果类型。这就是这种类型的扩展方式有用的地方了。`Cycle Plugin`暴露了`transitions`对象给用户添加他们自己的自定义变化。在插件中是这么定义的：
```
$.fn.cycle.transitions = {
 
    // ...
 
};
```
这技术允许其他人定义并传输变化定义并增强`Cycle Plugin`。
### 保持私有函数私有
暴露你插件的部分让用户去覆盖的技术非常有用。但是你需要非常小心的思考哪一部分的实现可以暴露，一旦暴露，你需要记住调用参数和语义的任何变化都有可能破坏向后兼容性。一般来说，如果你不确定是否要暴露特定功能，那你就不要暴露。

那么，我们要怎么做才能在不弄乱命名空间的情况下创建更多的功能并且不暴露实现呢？这是闭包的工作。为了演示，我们将添加其他函数到我们的插件，叫做`debug`。`debug`函数将会记录选中元素的数量到控制台。为了创建一个闭包，我们将整个插件定义在一个函数内（就像在`jQuery创作指南中那样`）。

```
jQuery.fn.superGallery = function( options ) {
 
    // Bob's default settings:
    var defaults = {
        textColor: "#000",
        backgroundColor: "#fff",
        fontSize: "1em",
        delay: "quite long",
        getTextFromTitle: true,
        getTextFromRel: false,
        getTextFromAlt: false,
        animateWidth: true,
        animateOpacity: true,
        animateHeight: true,
        animationDuration: 500,
        clickImgToGoToNext: true,
        clickImgToGoToLast: false,
        nextButtonText: "next",
        previousButtonText: "previous",
        nextButtonTextColor: "red",
        previousButtonTextColor: "red"
    };
 
    var settings = $.extend( {}, defaults, options );
 
    return this.each(function() {
        // Plugin code would go here...
    });
 
};
```
你的第一个想法（也可能不是第一个）可能是多大的插件才能满足这种等级的自定义。如果这个插件不是虚构的，可能大大超过必要。因为人们只愿意花费那么多的字节。

现在我们的`Bob`觉得这很棒；他对插件和插件的自定义性等级非常印象深刻。它相信，所有的选项让解决方案更加多种多样，可以使用在各种不同的场景。

`Sue`，我们的另一个朋友，决定使用这个新的插件。她设置了所有需要的的选项，现在有一个可以工作的解决方案在她面前。在玩了五分钟的插件后，她意识到如果画廊的每张图都放慢一点将会更加漂亮。她匆忙的查找`Bob`的文档，但是没有发现`animateWidthDuration`选项。

### 你发现问题了吗？
你的拆案有多少选项并不重要；重要的时候有什么选项。

`Bob`有点过头了。他提供的自定义程度看起来很高，但其实很低，特别是考虑一个人使用这个插件的时候可能想控制的所有的东西。`Bob`错误提供了很多滑稽特别的选项，让他的插件更难自定义。

### 一个更好的模型
所以，和明显：`Bob`需要一个新的自定义模型，一个不需要放弃控制或者抽象必要细节的模型。
`Bob`被这种高等级的简介所吸引的原因是`jQuery`框架非常适合这种思维方式。提供一个`previousButtonTextColor`选项非常简单和漂亮，但是我们需要面对的是大部分的插件用户都想要更多的控制！
这有一些小提示，可以帮助你为你的插件创建更好的自定义选项集合：
- 不要创建插件指定语法
开发部不需要学习新的语言或则术语就可以完成工作。
`Bob`觉得他提供的`delay`选项提供了最大的自定义性（看前面）。他让他的差劲可以指定4个不同的延迟，`quite short`，`very short`，`quite long`，`very long`：

```
var delayDuration = 0;
 
switch ( settings.delay ) {
 
    case "very short":
        delayDuration = 100;
        break;
 
    case "quite short":
        delayDuration = 200;
        break;
 
    case "quite long":
        delayDuration = 300;
        break;
 
    case "very long":
        delayDuration = 400;
        break;
 
    default:
        delayDuration = 200;
 
}
```
不仅仅限制了用户控制的等级，还增加了许多的空间。12行的代码只是去定义延迟事件有点太过了，你觉得呢？构造这个选项更好的方式是让插件用户指定数字事件（毫秒），这样这个配置的处理就不需要关心了。

这里的关键不是降低抽象的控制水平。无论你的你的抽象是神恶魔，都可以像你想要的那样简单，但是确保人们使用你的插件仍然可以有受人追捧的低级别的控制（低级别意味着非抽象的）。

- 提哦功能元素完整的控制
如果你的插件创建了元素并在`DOM`中使用，提供某种方式让插件用户可以访问这些元素是一个很好的想法。有时候这意味着提供给一个确定的元素`ID`或者类。但是注意你的插件内部不能依赖这种钩子：
一个坏的实现：

```
// Plugin code
$( "<div class='gallery-wrapper' />" ).appendTo( "body" );
 
$( ".gallery-wrapper" ).append( "..." );
```
为了允许用户访问和操作这些信息，你可以将它存储到变量并保存到你的插件设置。前面的栗子更好的实现方式就像这样：

```
// Retain an internal reference:
var wrapper = $( "<div />" )
    .attr( settings.wrapperAttrs )
    .appendTo( settings.container );
 
// Easy to reference later...
wrapper.append( "..." );
```
注意我们创建了注入包裹的引用，并且我们调用`.attr()`方法天啊急一些特定的属性到元素。所以，我们的设置可能可以这么操作：

```
var defaults = {
    wrapperAttrs : {
        class: "gallery-wrapper"
    },
    // ... rest of settings ...
};
 
// We can use the extend method to merge options/settings as usual:
// But with the added first parameter of TRUE to signify a DEEP COPY:
var settings = $.extend( true, {}, defaults, options );
```
`$.extend()`方法会遍历潜逃的对象，给我们一个默认的和传递进来的配置的合并版本，并给予传递配置更高的优先级。

现在插件用户有能力去指定包裹元素的任何属性，如果他们需要，则有一个`CSS`样式钩子，他们可以非常简单的添加一个类或者改变`ID`的名字，而没必要深入挖掘插件的源码。
The same model can be used to let the user define CSS styles:
相同的模型可以用来让用户定义`CSS`样式：

```
var defaults = {
    wrapperCSS: {},
    // ... rest of settings ...
};
 
// Later on in the plugin where we define the wrapper:
var wrapper = $( "<div />" )
    .attr( settings.wrapperAttrs )
    .css( settings.wrapperCSS ) // ** Set CSS!
    .appendTo( settings.container );
```
你的插件可能有一个关联的样式，开发者可以添加`CSS`属性。尽管在这种场景使用`JavaScript`提供一些设置样式的方便方法是一个很好的想法，不需要在元素上使用一个选择器表达式去获取元素。
### 提供回调能力
什么是回调？-一个回调是一个建立的函数，稍候他将被调用，一般被事件触发。被当作参数传递，一般被组件初始化调用，在这个栗子中，是`jQuery`插件。

如果你的插件被事件驱动，提供一个回调能力可能是一个好主意。甚至，你可以创建你自己的自定义事件然后为这些事件提供回调。在这个画廊创建，添加一个`onImageShow`回调是有意义的。

```
var defaults = {
 
    // We define an empty anonymous function so that
    // we don't need to check its existence before calling it.
    onImageShow : function() {},
 
    // ... rest of settings ...
 
};
 
// Later on in the plugin:
 
nextButton.on( "click", showNextImage );
 
function showNextImage() {
 
    // Returns reference to the next image node
    var image = getNextImage();
 
    // Stuff to show the image here...
 
    // Here's the callback:
    settings.onImageShow.call( image );
}
```
相对于通过传统意义初始化回调，我们在关联图片节点的图片上下文调用它。这意味着你可以在回调中通过`this`关键字访问到真实的图片节点。
```
$( "ul.imgs li" ).superGallery({
    onImageShow: function() {
        $( this ).after( "<span>" + $( this ).attr( "longdesc" ) + "</span>" );
    },
 
    // ... other options ...
});
```
同样的你可以添加`onImageHide`回调和一系列其他的回调。关于回调的观点是给插件用户一个简单的方式添加而外的功能，无需深入挖掘源码。
link Remember, It's a Compromise
### 注意，这是一个妥协
你的插件不是为了能够在各种场景工作。同样的，只提供少量的控制方式是没啥用的。所示，记住，做个妥协。三个事情你必须要时刻注意：
- 伸缩性：你的插件要处理多少种场景？
- 大小：你的插件的大小是否和你的功能等级匹配？比如，你会使用一个非常基本的提示插件如果他有`20K`?-可能不会！
- 性能：你的插件处理插件使用了大量的性能吗？速度影响大吗？对于最终用户来说，过头了吗？