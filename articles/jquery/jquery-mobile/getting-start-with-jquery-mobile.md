### 开始使用 jQuery Mobile
jQUery Mobile 提供了一系列触摸友好的 UI 组件和一个 AJAX 驱动的导航系统去支持动画页面切换。这个指南将指引你构建你的第一个 jQuery Mobile 应用。

link Create a Basic Page Templat
### 创建一个基本的页面模板
为了开始，你可以简单的粘贴下面的模版到你最喜欢的文本编辑器，保存，并在浏览器中打开文档。
在模版的`<head>`中，一个`mete viewport`标签设置屏幕的宽度为设备的像素宽度。从 CDDM 引用 jQuery， jQuery Mobile，和手机主题样式等样式和脚本。jQuery Mobile 1.4 可以和 jQuery 1.8 或者更新的版本一起使用。
在`<body>`内，一个`data-role`是`page`的`div`是一个包裹器，用来划出一个页面。一个头部条(data-role="header")，一个内容去（role="main" class="ui-content"）和一个底部区域(data-role="footer")被添加进去创建一个基本的页面（这三个都是可选的）。这些`data-`属性是`HTML5`属性，jQuery Mobile 可以将基本标签转化为增强和样式组件。
```
<!doctype html>
<html>
<head>
    <title>My Page</title>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="https://code.jquery.com/mobile/[version]/jquery.mobile-[version].min.css">
    <script src="https://code.jquery.com/jquery-[version].min.js"></script>
    <script src="https://code.jquery.com/mobile/[version]/jquery.mobile-[version].min.js"></script>
</head>
<body>
    <div data-role="page">
 
        <div data-role="header">
            <h1>My Title</h1>
        </div><!-- /header -->
 
        <div role="main" class="ui-content">
            <p>Hello world</p>
        </div><!-- /content -->
 
        <div data-role="footer">
            <h4>My Footer</h4>
        </div><!-- /footer -->
 
    </div><!-- /page -->
</body>
</html>
```
### 添加内容
下一步是添加内容到内容容器内部。任何标准的 HTML 元素 - 标题，段落，等，都可以添加。你可以编写你自定义的样式去创建一个自定义的布局，通过添加额外的样式到`<head>`中 jQuery Mobiel 样式之后。
### 做一个列表
jQuery Mobile 包含一系列各种个样添加`data-role="listview"`属性的列表。`data-inset="true"`属性让列表看起来像嵌入的模块，`data-filter="true"`添加一个动态搜索过滤。
```
<ul data-role="listview" data-inset="true" data-filter="true">
    <li><a href="#">Acura</a></li>
    <li><a href="#">Audi</a></li>
    <li><a href="#">BMW</a></li>
    <li><a href="#">Cadillac</a></li>
    <li><a href="#">Ferrari</a></li>
</ul>
```
### 添加一个滑动
框架包含了一整个系列的表单元素，自动增强的触摸友好样式组件。这是一个滑动器，使用 HTML5 type 为`range`的`input`实现的，不需要添加`data-role`。所有的表单样式都必须很好的和`<label>`关联，一组的表单必须被包裹在`<form>`标签。
```
<form>
    <label for="slider-0">Input slider:</label>
    <input type="range" name="slider" id="slider-0" value="25" min="0" max="100" />
</form>
```
link Make a Button
### 制作一个按钮
有很多中方式去制作按钮，一个普通的方式是转化一个链接为按钮，让他可以更容易的点击。从一个链接开始，为他添加`data-role="button"`属性。你可以使用`data-icon`属性添加一个图标，并可选的使用`data-iconpos`属性设置图标的位置。
```
<a href="#" data-role="button" data-icon="star">Star button</a>
```
link Choose a Theme Swatch
### 选择一个主题切换
jQuery Mobile 有一个很鲁棒的主题框架，支持多达26个系列的顶部条，内容，按钮颜色，叫做一个“swatch”。你可以添加一个`data-theme="b"`属性去任何你页面的组件：`page`，`header`，`list`，滑动器的`input`，或者`button`，去将它变成一个暗色阴影到黑色。默认主题中不同的`swatch`字母从a-b可以用于混合和命中`swatch`。
如果你添加主题`swatch`到页面，内容内部所有的组件都会自动继承这个主题。

```
<a href="#" data-role="button" data-icon="star" data-theme="a">Button</a>
```
如果你想要去创建一个自定义主题，你可以使用 ThemeRoller，它允许用户创建他们自己的主题，痛殴简单的使用拖拽界面。你可以i 下载和使用你新创建的主题。
### 更深入的构建一些东西
这个指南提供你一个 jQuery Mobile 页面基本的结构，和一些增强的组件。你可以探索完整的 jQuery Mobile API 文档和 jQuery Mobile 栗子中心去学习链接页面，添加动画到页面切换，创建弹窗和浮层。

如果你更倾向于实际写 JavaScript 去构建你的 app，你不想要使用`data-`属性配置系统，你可以完全控制任何点东西，并直接调用插件，因为所有的标准 jQuery 插件使用 UI 组件工厂构建的。对这些场景特别有用的信息可以从全局配置，事件和方法章节找到。
最后，你可以阅读脚本页面，生成动态动态页面，和构建 PhoneGap 应用。

