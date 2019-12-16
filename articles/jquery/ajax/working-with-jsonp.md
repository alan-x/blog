### 使用`JSONP`
`JSONP`的出现-本质上是一种双方协商的跨站脚本攻击-打开了威力强大的内容混合的大门。很多著名的占带你都提供了`JSONP`服务，允许你通过预定义的`API`获取他们的内容。一个特别大的`JSONP`格式数据的来源是[Yahoo! Query Language]()，也是接下来我们栗子中用来获取关于猫的新闻的地方。
```
// Using YQL and JSONP
$.ajax({
    url: "http://query.yahooapis.com/v1/public/yql",
 
    // The name of the callback parameter, as specified by the YQL service
    jsonp: "callback",
 
    // Tell jQuery we're expecting JSONP
    dataType: "jsonp",
 
    // Tell YQL what we want and that we want JSON
    data: {
        q: "select title,abstract,url from search.news where query=\"cat\"",
        format: "json"
    },
 
    // Work with the response
    success: function( response ) {
        console.log( response ); // server response
    }
});
```
`jQuery`在幕后处理了`JSPNP`所有复杂的方面-所有我们要做的就是告诉`jQuery`一个`JSONP`回调参数的名字，它定义在`YQL`（在这个栗子中叫做`callback`），而其他的处理看起来就像一个普通的`Ajax`请求。