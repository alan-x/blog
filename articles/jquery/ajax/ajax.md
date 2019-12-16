Ajax
Traditionally webpages required reloading to update their content. For web-based email this meant that users had to manually reload their inbox to check and see if they had new mail. This had huge drawbacks: it was slow and it required user input. When the user reloaded their inbox, the server had to reconstruct the entire web page and resend all of the HTML, CSS, JavaScript, as well as the user's email. This was hugely inefficient. Ideally, the server should only have to send the user's new messages, not the entire page. By 2003, all the major browsers solved this issue by adopting the XMLHttpRequest (XHR) object, allowing browsers to communicate with the server without requiring a page reload.
传统的网页需要重新加载才能更新他们的内容。对于基于网页的邮件来说这以为这需要用户手动刷新收件箱才能查看是否有新的邮件。这有很大的缺点：很慢并且需要用户输入。当用户重新加载他们的收件箱的时候，服务器需要重构构建整个网页并且发送所有的`HTML`、`CSS`和`JavaScript`，还有用户的邮件。这是非常低效的。理想情况下，服务器应该只发送用户的新信息，而不是整个页面。到了2003年，所有的主要浏览器采用`XMLHttpRequest(XHR)`对象解决了这个问题，允许用户和服务端交流而不通过重新加载页面。

The XMLHttpRequest object is part of a technology called Ajax (Asynchronous JavaScript and XML). Using Ajax, data could then be passed between the browser and the server, using the XMLHttpRequest API, without having to reload the web page. With the widespread adoption of the XMLHttpRequest object it quickly became possible to build web applications like Google Maps, and Gmail that used XMLHttpRequest to get new map tiles, or new email without having to reload the entire page.
`XMLHttpRequest`对象是`Ajax`技术的一部分。使用`Ajax`，数据可以在浏览器和服务器之间传输。使用`XMLHttpRequest`这个`API`，不需要去重新加载网页。随着`XMLHttpRequest`对象的广泛应用，构建像`Google Maps`和`Gmail`一样快的应用变得可能，他们使用`XMLHttpRequest`去获取新的地图切片，或者新的邮件，而不需要去重新加载整个页面。

Ajax requests are triggered by JavaScript code; your code sends a request to a URL, and when it receives a response, a callback function can be triggered to handle the response. Because the request is asynchronous, the rest of your code continues to execute while the request is being processed, so it's imperative that a callback be used to handle the response.
`Ajax`请求使用`JavaScript`代码触发；你的代码发送一个请求到一个`URL`，当收到响应的时候，一个回调函数将会被触发去处理响应。应为请求是异步的，接下来的代码将会继续运行，直到请求被处理，所以使用一个回调去处理响应是非常重要的。

Unfortunately, different browsers implement the Ajax API differently. Typically this meant that developers would have to account for all the different browsers to ensure that Ajax would work universally. Fortunately, jQuery provides Ajax support that abstracts away painful browser differences. It offers both a full-featured $.ajax() method, and simple convenience methods such as $.get(), $.getScript(), $.getJSON(), $.post(), and $().load().
很遗憾，不同的浏览器对于`Ajax Api`的实现都不同。这通常意味着开发者需要去考虑所有不同的浏览器，以保证`Ajax`可以正常工作。
幸运的是，`jQuery`提供了`Ajax`支持，可以脱离浏览器不同带来的痛苦。它不但提供了全功能的`$.ajax()`方法，也提供了简单方便的方法，比如`$.get()`、`$.getScript()`、`$.getJSON()`、`$.post()`、`$().load()`等。

Most jQuery applications don't in fact use XML, despite the name "Ajax"; instead, they transport data as plain HTML or JSON (JavaScript Object Notation).
尽管名字叫做`Ajax`，大部分的`jQuery`应用都不会使用`XML`；相反，他们传输普通的`HTML`或者`JSON(JavaScript Object Notation)`。

In general, Ajax does not work across domains. For instance, a webpage loaded from example1.com is unable to make an Ajax request to example2.com as it would violate the same origin policy. As a work around, JSONP (JSON with Padding) uses <script> tags to load files containing arbitrary JavaScript content and JSON, from another domain. More recently browsers have implemented a technology called Cross-Origin Resource Sharing (CORS), that allows Ajax requests to different domains.
通常来说，`Ajax`无法跨站。比如，一个`example1.com`加载的网页无法发送一个`Ajax`请求到`example2.com`，因为违反了同源策略。`JSONP(JSON with Padding)`使用`<script>`标签从其他站点加载包含`JavaScript`内容和`JSON`的文件可以作为一种变通的方式。越来越多的现代浏览器开始实现一个叫做`Cross-Origin Resource Sharing(CORS)`的技术，它允许`Ajax`请求其他站点。

Key Concepts
- [核心概念]()
jQuery’s Ajax-Related Methods
- [jQuery Ajax 相关方法]()
Ajax and Forms
- [Ajax 和 表单]()
Working with JSONP
- [使用 JSONP]()
Ajax Events
- [Ajax 事件]()
