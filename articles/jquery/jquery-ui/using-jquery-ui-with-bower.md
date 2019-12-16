### 在 Bower 中使用 jQuery UI
注意：这篇文档引用的功能在 jQuery UI 1.11 中可用。
Bower 是一个网页包管理器。你可以通过命令行使用 Bower 去下载包，比如 jQuery UI，不需要手动从他们各自的网站下载每个项目。
作为栗子，假设我们开始一个新项目，我们需要使用 jQuery UI 的手风琴组件。我们将为我们的项目创建一个新的文件夹，并添加下面的`index.html`模版文件。
```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>jQuery Projects</title>
</head>
<body>
 
<div id="projects">
    <h3>jQuery Core</h3>
    <p>jQuery is a fast, small, and feature-rich JavaScript library...</p>
    <h3>jQuery UI</h3>
    <p>jQuery UI is a curated set of user interface interactions...</p>
    <h3>jQuery Mobile</h3>
    <p>jQuery Mobile is a HTML5-based user interface system...</p>
</div>
 
<script>
    $( "#projects" ).accordion();
</script>
 
</body>
</html>
```
上面的栗子将会以一个 JavaScript 错误失败，因为既没有 jQuery Core 也没有 jQuery UI 被加载。让我们使用 Bower 加载他们。
### 使用 Bower 下载 jQuery UI
Bower 使用`bower install`命令下载库。为了安装 jQuery UI，运行`bower install jquery-ui`。这样子就会创建下面的文件夹结构（简化过）。
注意：如果你看到`bower`命名没有找到的错误，检查[Bower 的安装指南]()。
```
.
├── bower_components
│   ├── jquery
│   │   ├── dist
│   │   │   ├── jquery.js
│   │   │   └── jquery.min.js
│   │   └── src
│   └── jquery-ui
│       ├── themes
│       │   ├── smoothness
│       │   │   ├── jquery-ui.css
│       │   │   └── jquery-ui.min.css
│       │   └── [The rest of jQuery UI's themes]
│       ├── ui
│       │   ├── accordion.js
│       │   ├── autocomplete.js
│       │   └── ...
│       ├── jquery-ui.js
│       └── jquery-ui.min.js
└── index.html
```
这里发生了两件事，第一，Bower 知道 jQuery UI 依赖 jQuery core，所以它自动下载两个库。第二，所有的 jQuery UI 文件最新的发布都方便的放在一个 jquery-ui 文件夹，而这个文件在在一个新创建的`bower_components`文件夹内。

注意：如果你不想要最新的版本，你可以可选的提供一个版本号给`bower install`，比如，`bower install jquery-ui#1.10.4`安装版本 1.10.4 jQuery UI。
现在我们已经有文件了，我们可以去使用它们。
### 使用 Bower 下载文件
我们有几种不同的方式去使用 Bower 下载的文件。最简单的方式是使用压缩和混淆的文件，它在我们的 bower_component/jquery 和 bower_component/jquery-ui 文件夹。这种方式如下所示。
```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>jQuery Projects</title>
    <link rel="stylesheet" href="bower_components/jquery-ui/themes/smoothness/jquery-ui.min.css">
</head>
<body>
 
<div id="projects">
    <h3>jQuery Core</h3>
    <p>jQuery is a fast, small, and feature-rich JavaScript library...</p>
    <h3>jQuery UI</h3>
    <p>jQuery UI is a curated set of user interface interactions...</p>
    <h3>jQuery Mobile</h3>
    <p>jQuery Mobile is a HTML5-based user interface system...</p>
</div>
 
<script src="bower_components/jquery/dist/jquery.min.js"></script>
<script src="bower_components/jquery-ui/jquery-ui.min.js"></script>
<script>
    $( "#projects" ).accordion();
</script>
 
</body>
</html>
```
代码成功构建了手风琴组件，但是它需要包含整个 jQuery UI，而我们只需要手风琴组件。因为 jQuery UI 不仅仅只有手风琴组件，这强制用户下载超过他们需要的。
因为 Bower 也下载了 jQuery UI 的独立源文件，我们可以可选的使用他们，让用户只得到手风琴组件和它的依赖。下面的栗子使用这种方式构建了相同的手风琴组件。

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>jQuery Projects</title>
    <link rel="stylesheet" href="bower_components/jquery-ui/themes/smoothness/jquery-ui.min.css">
</head>
<body>
 
<div id="projects">
    <h3>jQuery Core</h3>
    <p>jQuery is a fast, small, and feature-rich JavaScript library...</p>
    <h3>jQuery UI</h3>
    <p>jQuery UI is a curated set of user interface interactions...</p>
    <h3>jQuery Mobile</h3>
    <p>jQuery Mobile is a HTML5-based user interface system...</p>
</div>
 
<script src="bower_components/jquery/dist/jquery.js"></script>
<script src="bower_components/jquery-ui/ui/core.js"></script>
<script src="bower_components/jquery-ui/ui/widget.js"></script>
<script src="bower_components/jquery-ui/ui/accordion.js"></script>
<script>
    $( "#projects" ).accordion();
</script>
 
</body>
</html>
```
到这里，你可以将 jQuery UI 的文件放进你的自定义构建系统去混淆和压缩你的资源和产品。如果你是一个 RequireJS 用户，查看[我们关于如何在 AMD 中使用 jQuery UI 的指南]()。

