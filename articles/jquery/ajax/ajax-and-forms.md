### `Ajax`和表单
`Jquery ajax`的能力在处理表单的时候特别有用。这里有很多一件，从序列化到客户端校验（比如：`对不起，用户名已存在`），预过滤（接下来会解释），还有跟多。
### 序列化
`jQuery`序列化表单输入非常简单。两个方法原生就支持：`.serialize()`和`.seriallizeArray()`。尽管这个名字非常具有自解释性，还有很多使用他们的建议。

`.serialize()`方法将表单数据序列化成查询字符串。要被序列化的元素，必须要有`name`属性。请注意`checkbox`和`radio`表单控件的值只有在选中的时候才会被包含在其中。
```
// Turning form data into a query string
$( "#myForm" ).serialize();
 
// Creates a query string like this:
// field_1=something&field2=somethingElse
```
尽管普通的旧的序列化很棒，但是有时候你的应用需要将会工作的更好，如果你发送一个对象或者数组，而不只是一个查询字符串。为此，`jQuery`有`.serializeArray()`方法。这和前面列出的`.serialize()`方法很像，除了产生一个数组或对象，而不是一个字符串。
```
// Creating an array of objects containing form data
$( "#myForm" ).serializeArray();
 
// Creates a structure like this:
// [
//   {
//     name : "field_1",
//     value : "something"
//   },
//   {
//     name : "field_2",
//     value : "somethingElse"
//   }
// ]
```
 ### 客户端校验
客户端校验就像其他东西一样，使用`jQuery`实现非常简单。尽管有很多个场景开发者可以测试，但是最常用的一个是：必须的输入，有效的`username/emails/phone numbers/`等，或者检测`我同意...`盒子。
请注意你也可以为你的输入做做服务端校验。然而，通常来说可以不提交表单就校验一些东西将会得到更好的用户体验。
就像常说的，来点栗子把。首先，我们将看到简单检测必须的输入域并不需要什么东西，如果它不是，将会返回`false`，然后阻止表单执行。
```
// Using validation to check for the presence of an input
$( "#form" ).submit(function( event ) {
 
    // If .required's value's length is zero
    if ( $( ".required" ).val().length === 0 ) {
 
        // Usually show some kind of error message here
 
        // Prevent the form from submitting
        event.preventDefault();
    } else {
 
        // Run $.ajax() here
    }
});
```
看看检测电话好吗中的无效字符是多么简单：
```
// Validate a phone number field
$( "#form" ).submit(function( event ) {
    var inputtedPhoneNumber = $( "#phone" ).val();
 
    // Match only numbers
    var phoneNumberRegex = /^\d*$/;
 
    // If the phone number doesn't match the regex
    if ( !phoneNumberRegex.test( inputtedPhoneNumber ) ) {
 
        // Usually show some kind of error message here
 
        // Prevent the form from submitting
        event.preventDefault();
    } else {
 
        // Run $.ajax() here
    }
});
```
### 预过滤
一个预过滤器在每个请求发送前是修改`ajax`选项的方法（所以，叫做预过滤器）
比如，我们想要通过一个代理修改所有的跨域请求。通过一个预过滤器来实现非常简单：
```
// Using a proxy with a prefilter
$.ajaxPrefilter(function( options, originalOptions, jqXHR ) {
    if ( options.crossDomain ) {
        options.url = "http://mydomain.net/proxy/" + encodeURIComponent( options.url );
        options.crossDomain = false;
    }
});
```
在回调函数参数之前，你可以传递一个可选的参数，指定你想要应用过滤器的`dataType`。比如，如果我们想要我们过滤器只应用于`JSON`和`script`请求，我们可以这么做：
```
// Using the optional dataTypes argument
$.ajaxPrefilter( "json script", function( options, originalOptions, jqXHR ) {
 
    // Do all of the prefiltering here, but only for
    // requests that indicate a dataType of "JSON" or "script"
});
```