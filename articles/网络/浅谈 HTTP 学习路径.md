### 0x000 概述
最近专注于 HTTP 的学习，这篇文章浅谈一下我学习 HTTP 过程中的心得，当作过渡性总结。如果能对各位有所帮助，最好不过。

### 0x001 HTTP 协议

说起 HTTP，自然逃不过 HTTP 协议本身，以下是我的学习资料：
- [《图解 HTTP》](https://book.douban.com/subject/25863515/)
- [《HTTP 权威指南》](https://search.douban.com/book/subject_search?search_text=HTTP+%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97&cat=1001)
- [http/0.9](https://www.w3.org/Protocols/HTTP/AsImplemented.html)
- [http/1.0](https://tools.ietf.org/html/rfc1945)
- [http/1.1 系列](https://tools.ietf.org/html/rfc7230)
- [从 0.9 到 QUIC](https://zhuanlan.zhihu.com/p/23366045)

《图解 HTTP》是入门精品，是初学者的福音。

《HTTP 权威指南》对于深入学习的需求也可以满足。

深入学习之后我觉得应该回归本质，也就是 HTTP 规范自身，rfc7230-7235 就是这接下来的不二选择。

其实前两本书已经够了，可是我发现单纯看书无法掌握技术发展的脉络，譬如 HTTP/0.9、HTTP/1.0、HTTP/1.1、HTTP/2、HTTP/3
是如何发展的。规范虽然晦涩，却有很明显的历史发展性；虽然不够详细，却非常全面。

http/0.9 协议很简单，只有一行，就是[simple-request](https://www.w3.org/Protocols/HTTP/Request.html)：
```
SimpleRequest     =        GET <uri> CrLf
```

http/1.0 和 和 HTTP/1.1 变化不大。
- 多部署服务器
- 默认 keep-alive 持久连接
- 引入传输编码

有人说作为前端，HTTP 协议使用的并不深，不需要掌握太深太细。但是我在学习之后却发现并不是这样，学习是一回事，实践是另一回事。

比如：
1. 在往常思考 HTTP 的时候，总是将请求分为 client 和 server 来思考。

![client-network-server](https://user-gold-cdn.xitu.io/2019/11/4/16e3654cf787b198?w=826&h=384&f=png&s=31704)

对于 network 中的一切都是忽略不计的，但深入学习之后才会发现，真实世界的网络拓扑可是能是这样的

![client-device-server](https://user-gold-cdn.xitu.io/2019/11/4/16e365890501afb9?w=1156&h=544&f=png&s=66108)

并且最后处理我们请求的并不一定是目标服务器，而是中间的某个设备，可能是代理，也可能是缓存服务，或者网关。

2. 我以为 HTTP 的报文结构从来是这样的：
```
request-ling/status-line
header-section

request-body/response-body
```
但其实也有可能是(Range Request)：
```
HTTP/1.1 206 Partial Content
Date: Wed, 15 Nov 1995 06:25:24 GMT
Last-Modified: Wed, 15 Nov 1995 04:58:08 GMT
Content-Length: 1741
Content-Type: multipart/byteranges; boundary=THIS_STRING_SEPARATES

--THIS_STRING_SEPARATES
Content-Type: application/pdf
Content-Range: bytes 500-999/8000

...the first range...
--THIS_STRING_SEPARATES
Content-Type: application/pdf
Content-Range: bytes 7000-7999/8000

...the second range
--THIS_STRING_SEPARATES--
```
看见了吗？HTTP 的 Header 并不是只能出现在 request-line/response-line 下面


3. Last-Modified、If-Modified-Since、ETag、If-Match...

我总是在缓存相关的文章中看到这些头部，所以我以为这些头部就是专门用来做缓存控制的，但是其实这些头部应该是属于 Preconditional Headers Field，是用来发起 Conditional Request，是为了防止多人同时编辑导致“update lost”问题的。比如两个人两个了 GET 了一个资源，该资源的 ETag 是 xx001，两个人修改资源之后 POST 到服务端，这两个 POST 请求带有 If-Match: xx001，其中一个请求 a 被接收了，资源更新，则新资源的 ETag 为 xx002，第二个人请求到来的时候，就可以通过检测资源的 ETag 和 If-Match 中是否一致，如果不一致，说明资源已经被修改。则这个资源不应该被更新，否则将会覆盖第一个人的更新。相当于一个锁。当然也用来做缓存控制，但是容易产生概念上的误解。

所以为什么要深入学习 HTTP，甚至学习 HTTP 协议本身呢？原因如下：
1. 从大局全面理解技术，避免一叶障目，不见泰山的情况，脱离信息茧房。（不能因为你只玩过游乐园的摩天轮，就认为这是一个摩天轮游乐园，毕竟人家可能还有海底世界和动物园呢）
2. 掌握技术在历史发展中的脉络，更好理解技术的存在的原因和未来发展的方向。
3. 不再人云亦云，对技术持有自己的观点。

### 0x002 基于 HTTP 之上的技术
在前端，通常我们不会直接操作 HTTP，而是通过浏览器提供的接口来实现这个过程：
- [URI](https://tools.ietf.org/html/rfc3986)
- [URL](https://url.spec.whatwg.org/)
- [cookie](https://tools.ietf.org/html/rfc6265)
- [同源策略](https://tools.ietf.org/html/rfc6454)
- [跨域资源共享](https://www.w3.org/TR/cors/)
- [XMLHTTPRequest](https://xhr.spec.whatwg.org/)
- [Fetch](https://fetch.spec.whatwg.org/)
- [jQuery ajax](https://learn.jquery.com/ajax/)
- [axios](https://github.com/axios/axios)
- [curl](https://curl.haxx.se/)

URI 和 URL 是不一样，应该详细了解 URL 的构成。

同源策略是用户代理安全基石之一，许多功能都是基于同源策略之上，在理解跨域资源共享之前还是先理解一下同源策略吧。

注意：cookie 是一个痛苦的例外，一来它可以在头部中出现多次，二来它并没有使用同源策略。

XMLHTTPRequest 叫这名字存粹是历史原因，和功能无关。

Fetch 和 XMLHTTPRequest 就是我们最常用的底层对象了。

有人觉得 jQuery 已经落后于时代了，完全不需要去了解。但是作为当年最火的项目，它的每一个实现在当时都是最佳实践。我们学习历史的原因是为了在当时历史条件下，分析一个技术解决问题的思路，而不仅仅看它在当代能有啥用。

axios 真是前端瑞士军刀。

还有一些工具：
- [Fiddler](https://www.telerik.com/fiddler)
- [Charles](https://www.charlesproxy.com/)
- [wireshark](https://www.wireshark.org/)
- [postman](https://www.getpostman.com/)

Fiddler和 Charles 一样，都是网络调试利器，我常用 Charles 抓包修改请求和响应测试各种数据情况下的交互。

wireshark 也是抓包，但是只是在需要分析底层报文的时候才用。

postman 则是测试 restful 接口、团队合作的好东西。

curl 在无界面环境是不错的选择，比如 linux。

### 0x003 HTTPS
HTTP 之后自然轮到 HTTPS 了：
- [阮一峰 https 相关](http://www.ruanyifeng.com/blog/2016/08/migrate-from-http-to-https.html)
- [HTTPS 权威指南](https://book.douban.com/subject/10746113/)

我们需要知道浏览器如何和 HTTPS 建立连接，并传输数据的。

### 0x004 TSL
- [HTTP Over TLS](https://tools.ietf.org/html/rfc5246)

在 HTTPS 的底层是 TSL，本身是可以作为独立的技术存在的，有握手协议和消息协议。

### 0x005 X.509
- [X.509 数字证书的基本原理及应用](https://zhuanlan.zhihu.com/p/36832100)
- [Internet X.509 ](https://tools.ietf.org/html/rfc5280)

浏览器在建立 HTTPS 连接的时候，需要验证证书，想要知道证书的详细格式，就需要了解 X.509。

### 0x006 openssl
- [OpenSSL](https://www.openssl.org/)
- [证书的信任链校验](https://www.zybuluo.com/blueGhost/note/807076)

生成、转化、验证证书的库/工具，可以用来模拟验证过程，或者自己成为 CA 颁发证书。

### 0x007 http 服务
- [Nginx](http://nginx.org/)
- [Apache](http://www.apache.org/)
- [Node http 模块](http://nodejs.cn/api/http.html)
- [http-server](https://www.npmjs.com/package/http-server)

Nginx 和 Apache 搭建 HTTP 服务器，配合 openSSL 做配置 HTTPS 实验。

Node http 模块可以自己搭建一个 web 服务器。

http-server 则是静态服务器的命令行工具。

### 0x008 HTTP2
- [HTTP/2](https://tools.ietf.org/html/rfc7540)

支持帧传输、二进制流、头部压缩、服务端推送等。

### 0x009 TCP/IP
- [IP](https://tools.ietf.org/html/rfc791)
- [TCP](https://tools.ietf.org/html/rfc793)
- [A TCP/IP Tutorial](https://tools.ietf.org/html/rfc1180)
这里只做记录。

### 0x010 HTTP3
- [draft-ietf-quic-transport-23](https://tools.ietf.org/html/draft-ietf-quic-transport-23)
- [draft-ietf-quic-http-23](https://tools.ietf.org/html/draft-ietf-quic-http-23)

底层抛弃了 TCP 协议，转而使用基于 UDP 协议的 Quic 协议。目前还是草稿版本。

### 0x011 UDP
- [UDP](https://tools.ietf.org/html/rfc768)
- [UDP Usage Guidelines](https://tools.ietf.org/html/rfc8085)
这里只做记录。

### 0x012 DNS
...待续

### 0x013 CDN
...待续

### 0x014 总结
以上，将随着我的学习持续更新。

### 0x015 带货
帮大佬带货：[【随心秀】开篇 - 开源微场景编辑器介绍](https://juejin.im/post/5db718b7f265da4d1e26cb9c)
