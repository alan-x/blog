### `jQuery Ajax`相关的方法
`jQuery`没有提供很多`Ajax`相关的简便方法，核心的`$.ajax`方法是他们的心脏，理解它非常重要，我们会先复习它，然后简单介绍其他的简便方法。

一般来说使用`$.ajax()`相对`jQuery`提供的简便方法来说是一个很好的实践。就像你看到的，它提供了很多简便方法没有的特性，它的语法更具有可读性。

### `$.ajax()`
`jQuery`的核心`$.ajax()`方法是创建一个`Ajax`强大的和直接的方式。它接收一个配置对象包含`jQuery`所有的执行去完成一个请求。`$.ajax()`方法特别有价值，因为它提供了指定成功和失败回调的能力。他可以通过一个配置对象分开定义，可以简单的写出复用的代码。关于配置项完整的文档，可以查阅`http://api.jquery.com/jQuery.ajax/`。
```
// Using the core $.ajax() method
$.ajax({
 
    // The URL for the request
    url: "post.php",
 
    // The data to send (will be converted to a query string)
    data: {
        id: 123
    },
 
    // Whether this is a POST or GET request
    type: "GET",
 
    // The type of data we expect back
    dataType : "json",
})
  // Code to run if the request succeeds (is done);
  // The response is passed to the function
  .done(function( json ) {
     $( "<h1>" ).text( json.title ).appendTo( "body" );
     $( "<div class=\"content\">").html( json.html ).appendTo( "body" );
  })
  // Code to run if the request fails; the raw request and
  // status codes are passed to the function
  .fail(function( xhr, status, errorThrown ) {
    alert( "Sorry, there was a problem!" );
    console.log( "Error: " + errorThrown );
    console.log( "Status: " + status );
    console.dir( xhr );
  })
  // Code to run regardless of success or failure;
  .always(function( xhr, status ) {
    alert( "The request is complete!" );
  });
```
注意：关于`dataType`设置，如果服务端返回的数据和你指定的不同，你的代码可能会失败，原因可能不总是很清晰，因为`HTTP`响应代码不会显示错误。当使用`Ajax`请求的时候，请确保你的服务器返回的数据类型是你想要的，验证`Content-type`头是否和数据类型匹配。比如，对于`JSON`数据，`Content-Type`头应该是`application/json`。
### `$.ajax()`配置项
`$.ajax()`方法有很多的配置项，这是他强大的一部分。查阅完整的配置列表，访问`http://api.jquery.com/jQuery.ajax/`；以下是你最常用的一些配置：
- `async`
如果请求要同步，设置为`false`，默认是`true`。注意如果这个选项设置为`false`，你的请求将会堵塞其他的代码执行，直到响应被接收。
- `cache`
如果有缓存，使用缓存。默认为`true`，除了`dataType`为`script`和`jsonp`。当设置为`false`的时候，`URL`将会添加一个简单的缓存破坏参数。
- `done`
请求成功的回调函数。函数接收响应数据（如果`dataType`是`json`，将会转化成`JavaScript`对象），还有请求的文本状态和原始请求对象。

- `fail`
请求失败的回调函数。该函数接收原始请求对象和请求的文本状态。

- `always`
请求完成的回调函数，无论成功还是失败。该函数接收原始请求对象和请求的文本状态。

- `context`
回调函数应该运行的上下文（回调函数中的`this`）。回调函数内的`this`指向传递给`$.ajax()`的对象。

link data
- `data`
发送给服务端的数据。可以是一个对象或者查询字符串，比如`foo=bar&amp;baz=bim`。
- `dataType`
你希望从服务端获取到的数据的类型。默认情况下，如果`dataType`没有指定，`jQuery`将会查看响应的`MIME`类型。
- `jsonp`
发送一个`JSONP`的时候放在查询字符串中的回调名。默认是`callback`。
- `timeout`
请求失败的等待时间，毫秒级别。

- `traditional`
在`jQuery 1.4`中设置为`true`可以使用参数序列化风格。了解详情，可以访问`http://api.jquery.com/jQuery.param/`。
- `type`
请求的类型，`POST`或者`GET`，默认是`GET`。其他请求类型，比如`PUT`、`DELETE`可以使用，但是好他们可能不被所有的浏览器支持。

- `url`
请求的`URL`
`url`参数是`$.ajax()`配置对象的唯一的必须属性。其他属性都是可选的。这可以作为`$.ajax()`的第一个参数，配置对象作为第二个参数。

- 简便方法
如果你不需要`$.ajax()`可扩展的配置能力，并且不需要处理错误，`jQuery`提供的`Ajax`简便方法非常有用，简便的方式去完成`Ajax`请求。这些方法只是包装了核心的`$.ajax()`方法，并且为`$.ajax()`预置了一些选项。
`jQuery`提供的简便方法有：
link $.get
- `$.get()`
执行一个`GET`请求到提供的`URL`。
- `post`
执行一个`POST`请求到提供的`URL`。
- `$.getScript`
添加一个`script`到页面。
- `$.getJSON`
执行一个`GET`请求，期待返回`JSON`。
在每个场景中，方法使用了一下参数，按顺序来：
- `url`
请求的`URL`。必须。
link data
- `data`
要发送给服务端的数据。可选。可以是对象，也可以是查询字符串，比如`foo=bar&amp;baz=bim`
注意：这个选项`$.getScript`不可用。
- 成功回调
请求成功的回调函数。可选的。函数接收响应数据（如果数据类型是`JSON`就会转化成`JavaScript`对象）。还有请求的文本状态和原始请求对象。
- 数据类型
你希望服务端返回的数据的类型。可选。
注意：这个选项只能被还没有指定数据类型的方法应用。
```
// Using jQuery's Ajax convenience methods
 
// Get plain text or HTML
$.get( "/users.php", {
    userId: 1234
}, function( resp ) {
    console.log( resp ); // server response
});
 
// Add a script to the page, then run a function defined in it
$.getScript( "/static/js/myScript.js", function() {
    functionFromMyScript();
});
 
// Get JSON-formatted data from the server
$.getJSON( "/details.php", function( resp ) {
 
    // Log each key in the response data
    $.each( resp, function( key, value ) {
        console.log( key + " : " + value );
    });
});
```
### `$.fn.load`
`.load()`方法是`jQuery`唯一一个在选择器上调用的`Ajax`方法。`.load()`方法从`URL`获取`HTML`，并使用返回的`HTML`去填充选择的元素。除了提供一个`URL`，你还可以可选的提供一个选择器表达式；`jQuery`只会从返回的`HTML`获取命中的内容。

```
// Using .load() to populate an element
$( "#newContent" ).load( "/foo.html" );
```
```
// Using .load() to populate an element based on a selector
$( "#newContent" ).load( "/foo.html #myDiv h1:first", function( html ) {
    alert( "Content updated!" );
});
```