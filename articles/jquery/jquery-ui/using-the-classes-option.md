### 使用类选项
当 1.12 版本发布的时候， jQuery UI 组件工厂可以通过类选项来管理 CSS 类。这篇文章让你过一遍类配置的工作方式，并讨论你可以怎样操作它。

### 语法概述
类配置用来映射结构类名和你定义的主题相关的类名。为了了解这到底是什么意思，让我们看一个栗子。瞎买呢的代码用`classes`配置创建一个红色弹窗。
```
<style>
    .custom-red { background: red; }
</style>
<script>
    var dialog = $( "<div>Red</div>" ).dialog({
        classes: {
            "ui-dialog": "custom-red"
        }
    });
</script>
```
这里`custom-red`类名和结构`ui-dialog`类名关联起来了。现在，当弹窗应用`ui-dialog`类名的时候，也会添加一个`costom-red`类名。
然而，除了添加一个`custom-red`类之外，还发生了一些东西，我们将在接下来看到。下面的代码也移除了“ui-corner-all”的值。你可以使用空格分离的多个类名作为对象的值来关联多个类名。比如下面的代码创建了一个弹窗，它是红色的摒弃有圆角。
```
<style>
    .custom-red { background: red; }
</style>
<script>
    var dialog = $( "<div>Big and red</div>" ).dialog({
        classes: {
            "ui-dialog": "ui-corner-all custom-red"
        }
    });
</script>
```
为了得到你可以在`classed`选项可以使用的完整类名列表，查看你感兴趣的 jQuery UI 组件文档。比如，这是`dialog`可以使用的`classes`列表：[]()。
`classes`配置和组件工厂的其他配置一样工作，这意味着所有的组件工厂配置机制依旧应用。比如，瞎买呢的代码使用`option()`方法移除了所有正和`ui-dialig`类名关联的类名。
```
dialog.dialog( "option", "classes.ui-dialog", null );
```
瞎买呢创建了一个组件扩展，自动关联`custom-red`类和`ui-dialog`类：
```
<style>
    .custom-red { background: red; }
</style>
<script>
    $.widget( "custom.dialog", $.ui.dialog, {
        options: {
            classes: {
                "ui-dialog": "ui-corner-all custom-red custom-big"
            }
        }
    });
    $( "<div>Big and red</div>" ).dialog();
</script>
```
组件工厂也在组件摧毁的时候移除了所有`classed`指定的的类名，作为添加的好处。
link Theming
### 主题
就像前一个栗子显示的，`classes`配置提供了一个关联组件内使用的主题相关的类名和结构化类名的快速方式。这个方式对简单的场景有用，但也可以用于接受第三方主题组件工厂构建的组件。比如，如果你使用 Bootstrap 和 jQuery UI，你可以使用下面的代码去创建使用 Bootstrap 主题的 jQuery UI 弹窗：
```
$.extend( $.ui.dialog.prototype.options.classes, {
    "ui-dialog": "modal-content",
    "ui-dialog-titlebar": "modal-header",
    "ui-dialog-title": "modal-title",
    "ui-dialog-titlebar-close": "close",
    "ui-dialog-content": "modal-body",
    "ui-dialog-buttonpane": "modal-footer"
});
```
查询更多这种方式的栗子，可以检出Alexander Schmitz 使用`classes`配置适配 jQuery UI 和 Bootstrap 的[仓库]()。
### 结论
`classes`配置的介绍带我们进一步的分离结构和主题相关的类，让它更简单的让 jQuery UI 组件的外观和感觉更贱配合你的站点。同时，这让 jQuery UI 和其他 CSS 框架一起使用，就像 jQuery 可以和其他 JavaScript 框架一起使用一样。