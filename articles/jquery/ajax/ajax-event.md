### `Ajax`事件
经常，你想要在`Ajax`请求开始和结束的时候做一些操作，比如显示或者隐藏加载指示器。相对于将这个行为定义在每个`Ajax`请求，不如绑定`Ajax`事件到元素上，就像你绑定其他事件一样。了解`Ajax`事件的更多详情，访问[在 docs.jquery.com 的Ajax 事件文档]()。

```
// Setting up a loading indicator using Ajax Events
$( "#loading_indicator" )
    .ajaxStart(function() {
        $( this ).show();
    })
    .ajaxStop(function() {
        $( this ).hide();
    });
```
