### 导读
这篇文章基于[[RFC 6265]](https://tools.ietf.org/html/rfc6265)，简单说明 Cookie 的使用和特性。大概包括如下四个内容：1）介绍 Cookie 的使用；2）详解 Cookie 的格式；3）测试 Cookie 在各种情况下的反应；4）CSRF 攻击说明


### 环境、工具及前置知识
1. 系统：`macOS Mojava`
2. IDE：`IDEA`
3. `SwitchHosts`
4. `OpenSSL`
5. `Chrome`
5. `js` 基础
6. `NodeJS` 基础
7. `HTTP` 基础

### 0x001 Cookie 的使用

`cookie` 的使用非常简单，可以归纳为 4 步：
1. 前端发送 `HTTP` 请求到后端
2. 后端生成要放到 `cookie` 中的信息并设置到响应中的 `Set-Cookie` 头部
3. 前端取出响应中的 `Set-Cookie` 的内容，保存 `cookie` 信息到本地
4. 前端继续访问后端页面，将保存到本地的 `cookie` 放到请求的 `Cookie` 头部

用图说明：
![Cookie 简单使用](https://user-gold-cdn.xitu.io/2019/12/4/16ecccbe119229fe?w=1112&h=1144&f=png&s=96116)


用代码说明：
```
const http = require('http');

http.createServer((req, res) => {
    res.writeHead(200, {'Set-Cookie': 'name=123'})
    res.write(`
        <script>document.write(document.cookie)</script>
    `)
    res.end()
}).listen(3000)
```
这段代码的功能很简单：
1. 创建一个 `HTTP` 服务器
2. 为每一个响应添加一个 `Set-Cookie` 头部，它的值是 `name=123`
3. 响应的正文是一个 `html`，其中只有一个 `script` 标签，标签内的脚本会将 `cookie` 输出到当前页面

启动这个脚本，然后打开浏览器访问：
```
$ node index.js
$ open http://localhost:3000
```
就可以看到：
1. 页面上显示 `name=123`，正是我们 `Set-Cookie` 的内容
2. `HTTP` 的 `response` 头部有 `Set-Cookie: name=123`

![Cookie 简单使用](https://user-gold-cdn.xitu.io/2019/12/1/16ec1bb632cd9e9a?w=1522&h=1374&f=png&s=375759)

打开 `Chrome` 调试工具的 `Application -> cookies -> localhost:3000`，就可以看见我们设置的 `cookie` 了：

![存储在本地的 cookies](https://user-gold-cdn.xitu.io/2019/12/1/16ec1bdd16d3e77c?w=1620&h=568&f=png&s=99182)


此时再打开一个 `tab`，然后再访问这个页面（或者直接刷新一下就好，不过为了对比，可以再开一个 `tab`）：

![在 request 中携带 cookie](https://user-gold-cdn.xitu.io/2019/12/1/16ec1c452ae40c10?w=1476&h=1394&f=png&s=329799)

可以看到，此时，对比第一次访问的时候，在 `request` 中多了一个 `Cookie`，而 `Cookie` 的内容就是我们 `Set-Cookie` 的内容（这不表示 `Cookie === Set-Cookie`，只是说明 `Cookie` 的内容来自 `Set-Cookie`，在后续会有详细说明 `Set-Cookie` 到 `Cookie` 的转化）。

这就是最简单的 `cookies` 使用了。

### 0x002 cookies 的格式详解

![cookies 的属性](https://user-gold-cdn.xitu.io/2019/12/1/16ec1cbc4d20a965?w=1906&h=574&f=png&s=103229)

从 `Chrome` 的 `cookies` 管理工具可以看出一条 `cookie` 的属性是有很多的：
- `Name`
- `Value`
- `Domain`
- `Path`
- `Expire / Max-Age`
- `Size`（忽略，应该只是前端统计 Cookie 键值对的长度，比如"name"和"123"的长度是 7，如果有大神知道请告知）
- `HttpOnly`
- `Secure`
- `SameSite`

在接下来的章节，将会慢慢解释这些属性。

#### 1. Expire

`Expires` 是用来为一个 `cookie` 设置过期时间，它是一个 `UTC` 格式的时间，过了这个时间以后，这个 cookie 就失效了，浏览器不会将这条失效的 `cookie` 包含在 `Cookie` 请求头部中，并且会删除这条过期的 `cookie`。

代码：
```
const http = require('http');

http.createServer((req, res) => {
    const cookie =req.headers['cookie']
    const date = new Date();
    date.setMinutes(date.getMinutes()+1)
    if (!cookie || !cookie.length){
        res.writeHead(200, {'Set-Cookie': `name=123; expires=${date.toUTCString()}`})
    }
    res.write(`
        <script>document.write(document.cookie)</script>
    `)
    res.end()
}).listen(3000)

```
上面的代码在给 `response` 添加的 `Set-Cookie` 中多了一个 `expires`属性，就是用来设置过期时间，这里将它设置为一分钟以后:
```
    const date = new Date();
    date.setMinutes(date.getMinutes()+1)
```

同时做了一个判断，如果 `request` 中有 `cookie` 了，就不添加 `Set-Cookie` 头部，否则永远不会过期。

打开浏览器（先删除之前的 `cookies`），访问`localhost:3000`：


![带 epires 的 cookie](https://user-gold-cdn.xitu.io/2019/12/1/16ec210d125fa12e?w=1544&h=1284&f=png&s=278261)
可以看见第一次访问的时候
```
Date: Sun, 01 Dec 2019 15:24:47 GMT
Set-Cookie: name=123; expires=Sun, 01 Dec 2019 15:25:47 GMT
```
Set-Cookie 的格式变了，多了一个 `expires` 属性，并且时间是 `Date` 属性的 1分钟以后。在这一分钟之内，如果我们刷新页面，会发现 `request` 中有 `Cookie`，而 `response` 中没有 `Set-Cookie`：

![expires 内的 request](https://user-gold-cdn.xitu.io/2019/12/1/16ec2119a80c5519?w=1488&h=1278&f=png&s=263218)

而在 1 分钟以后，则会重新生成一个 `Set-Cookie`，并且 `request` 中的 `Cookie` 没了：

![重新生成的 cookie](https://user-gold-cdn.xitu.io/2019/12/1/16ec2122b378ed99?w=1542&h=1268&f=png&s=277556)



`expires` 属性的的格式是：
```
    expires-av = "Expires=" sane-cookie-date
    sane-cookie-date = <rfc1123-date, 定义在 [RFC2616], 章节 3.3.1>
```
`sane-cookie-date` 是一个时间格式，大概如下：
```
    Sun, 06 Nov 1994 08:49:37 GMT
```
`js` 可以使用如下获得：
```
    $ new Date().toUTCString()
    "Sun, 01 Dec 2019 15:18:12 GMT"
```

`expires` 的默认值是 `session`，也就是在当前浏览器关闭以后就会删除。

#### 2. Max-Age

`Max-Age` 也是用来设置 `cookie` 的过期时间，但是它设置的是相对于资源获取的时间的相对秒数。比如，第一次访问的时候，`response` 的 `Date` 是 `Sun, 01 Dec 2019 15:44:43 GMT`，那么这条 `cookie` 的过期时间就是 `Sun, 01 Dec 2019 15:45:43 GMT`

第一次请求的时候，可以看到，`cookie` 的格式是：`Set-Cookie: name=123; max-age=60`，多了一个 max-age=60。

![response 中带 max-age](https://user-gold-cdn.xitu.io/2019/12/1/16ec2230f23c57d6?w=1668&h=1252&f=png&s=284410)

如果在 60 s 内访问，则会看到 `request` 中包含了一个 `Cookie: name=123`，而 `response` 中没有`Set-Cookie`

![max 有效期 request 中带 cookie](https://user-gold-cdn.xitu.io/2019/12/1/16ec2238f638d19b?w=1520&h=1262&f=png&s=265536)

在 60 s 以后访问，则又创建了新的 cookie。

`max-age` 的默认值是 `session`，当前浏览器关闭以后就会被删除。

`max-age` 的格式是：
```
    max-age-av = "Max-Age=" non-zero-digit *DIGIT
```

#### 3. Domain

`Domain` 限制在哪个域名下会将这个 cookie 包含到 `request` 的 `Cookie` 头部中。

比如，如果我们设置一个 cookie 的 Domain 属性是 `example.com`，则用户代理会在发往 `example.com`，`www.example.com`，`www.corp.example.com` 的请求的 `Cookie` 中携带这个 `cookie`。

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    const cookie =req.headers['cookie']
    const date = new Date();
    date.setMinutes(date.getMinutes()+1)
    if (!cookie || !cookie.length){
        res.writeHead(200, {'Set-Cookie': 'name=123;domain=example.com'})
    }
    res.write(`
        <script>document.write(document.cookie)</script>
    `)
    res.end()
}).listen(3000)
```
这里我们添加了一个 `domain=example.com`。然后使用 `SwitchHost` 配置几个 `host`：
```
127.0.0.1 example.com
127.0.0.1 www.example.com
127.0.0.1 www.corp.example.com
```
访问 `example.com:3000`，第一次访问的时候，会返回
```
Set-Cookie: name=123;domain=example.com
```


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6b1411589257?w=2088&h=1006&f=png&s=271546)

然后我们刷新页面，就会发现 `cookie` 被携带到 `request` 中：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6b37248f79f0?w=1860&h=920&f=png&s=241060)

访问 `www.example.com:3000`，也存在这个 `cookie`


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6b53720e74fa?w=1634&h=940&f=png&s=249443)

访问 `www.corp.example.com:3000`，也存在这个 `cookie`


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6b5f7914f2bd?w=1528&h=922&f=png&s=261718)

但是如果访问 `exampel2.com:3000`，就不存在了，并且，因为 `cookie` 中的 `domain` 和当前的 `domain` 不同，用户代理会拒绝存储这个 `cookie`，所以我们看不到 `cookie` 的输出：


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6b6aed64d1b4?w=1664&h=932&f=png&s=282054)
在 `cookie` 管理中，也不存在这条 `cookie`：


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6b71882b6c0a?w=1650&h=516&f=png&s=73353)

`domain` 的默认值是当前的域名。

`domain` 的格式是：
```
    domain-av = "Domain=" domain-value
```

#### 4. Path

相对于 `Domain` 限于 `cookie` 的域名，`Path` 限制 `cookie` 的路径，比如，如果我们设置 `cookie` 的路径是 `/a`，则在 `/` 或者 `/b` 就无法使用

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    const cookie =req.headers['cookie']
    const date = new Date();
    date.setMinutes(date.getMinutes()+1)
    if (!cookie || !cookie.length){
        res.writeHead(200, {'Set-Cookie': 'name=123;path=/a'})
    }
    res.write(`
        <script>document.write(document.cookie)</script>
    `)
    res.end()
}).listen(3000)

```
这里添加了一个 `path=/a`，其他就没有变化了，访问 `localhost:3000/a`，得到一个 `cookie`：


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6bf944517707?w=1626&h=1034&f=png&s=224253)

刷新页面，发送刚刚得到的 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6c0807a0af92?w=1448&h=1098&f=png&s=229965)

访问 `/a/b`，却可以：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6c344364a232?w=1808&h=1054&f=png&s=266803)

访问 `/b`，不会发送 `cookie`，并且会生成一个新的 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6c1851c7e686?w=1518&h=1054&f=png&s=235395)

但是由于和当前路径不匹配，被拒绝：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6c22215835c1?w=1774&h=496&f=png&s=72901)


访问 `/`，则和访问 `/b` 一样的结果：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6c42811e900a?w=1712&h=1054&f=png&s=267707)

因此，Path 可以限制一个 `cookie` 在哪个路径及其子路径下可用，祖先路径和兄弟路径则不在允许之列，如果没有这个值，则是 `/`

`Path` 的默认值是 `/`，也就是对整个站是有效的。

`path` 的格式是：
```
    path-av = "Path=" path-value
```


#### 5. Secure

`Secure` 用来指示一个 `cookie` 只在“安全”通道发送，这里的安全通道，一般指的就是 `https`。意思就是只有在 `https` 的时候才发送，`http` 不发送。

上代码：
```
let https = require("https");
let http = require("http");
let fs = require("fs");

const options = {
    key: fs.readFileSync('./server.key'),
    cert: fs.readFileSync('./server.pem')
};

const app = (req, res) => {
    const cookie =req.headers['cookie']
    const date = new Date();
    date.setMinutes(date.getMinutes()+1)
    if (!cookie || !cookie.length){
        res.writeHead(200, {'Set-Cookie': 'name=123;secure'})
    }
    res.write(`
        <script>document.write(document.cookie)</script>
    `)
    res.end()
}

https.createServer(options, app).listen(443);
http.createServer(app).listen(80);
```
这里同时兼通 `80、443` 端口，提供 `http` 和 `https` 服务，其中，`https` 服务所需的证书由 `openssl` 生成，这里免去不讲。访问 `https://example.com`，得到 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6dfd703965c5?w=2010&h=1070&f=png&s=249312)

刷新页面，发送 `cookie`：


![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6e080a2f1d54?w=1496&h=1186&f=png&s=247034)

访问 `http://example.com`，得到 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6e13acdd4e34?w=1742&h=1034&f=png&s=244636)

但是被拒绝：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6e197498c399?w=1614&h=510&f=png&s=71094)

`secure` 的默认值是 `false`，也就是 `http/https`都能访问。

`secure` 的格式是：
```
    secure-av = "Secure"
```

#### 6. HttpOnly

`Secure` 限制 `cookie` 只能在安全通道中使用，别以为 `HttpOnly` 就是只能在 `Http` 中使用的意思，`HttpOnly` 的意思是只能通过 `http` 才能访问，在前面的例子中，我们一直通过`document.cookie`来在前端访问 `cookie`，但是如果指定了这个，就无法通过`document.cookie`来操作 `cookie` 了。

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    const cookie =req.headers['cookie']
    const date = new Date();
    date.setMinutes(date.getMinutes()+1)
    if (!cookie || !cookie.length){
        res.writeHead(200, {'Set-Cookie': 'name=123;httpOnly'})
    }
    res.write(`
        <script>document.write(document.cookie)</script>
    `)
    res.end()
}).listen(3000)
```
多了一个 `httpOnly`，访问 `localhost:3000`，得到 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6e67ba7b1580?w=1454&h=1066&f=png&s=222211)

刷新，发送 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6e6f786f0276?w=1462&h=1150&f=png&s=236917)

使用 `document.cookie` 操作 `cookie`：
```
> document.cookie = 'name=bar'
< "name=bar"
> document.cookie
< ""
```
然后查看 `cookie`，可以看见 `value` 依旧是 `123`，而 `HttpOnly` 则被打勾，无情。

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec6e9a0cdba5ac?w=1840&h=508&f=png&s=86215)

`httpOnly` 的默认值是 `fasle`，也就是可以用`document.cookie`来操作 `cookie`

`httpOnly` 的格式是
```
    httponly-av = "HttpOnly"
```

### 7. SameSite

注意，在说这个属性之前，有一点需要说明，那就是请求头中发送 `cookie` 并不是只有在地址栏输入这个网址的时候才会发送这个 `cookie`，而是在整个浏览器的所有页面内，发送的所有 `http` 请求，比如 `img` 的 `src`，`script` 的 `src`，`link` 的 `src`，`ifram` 的 `src`，甚至 `ajax` 配置之后，只要满足 上面提到的条件，这个 `http` 请求都会包含这个请求地址的 `cookie`，就算你是在其他网站访问也是一样，这也就是后面讲的 `csrf` 有机可趁的原因。

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    if (req.headers.host.startsWith('example')) {
        res.writeHead(200, {'Set-Cookie': `host=${req.headers.host}`})
    }
    if (req.headers.host.startsWith('localhost')) {
        res.write(`
            <script src="http://example.com:3000/script.js"></script>
            <img src="http://example.com:3000/img.jpg"/>
            <iframe src="http://example.com:3000/"/>
        `)
    }
    res.end()
}).listen(3000)

```
如果访问的地址是 `example` 开头，设置了一个 `cookie`，名字为 `host`，值为当前访问的地址。
如果访问的地址是`localhost`开头，就返回 3 个元素，他们的 `src` 指向 `example.com:3000` 三个不同的资源（尽管不存在，但是足够了）。
访问 `example.com:3000`，得到 `cookie：host=example.com:3000`

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec70d5bdca99a4?w=1424&h=1012&f=png&s=216516)

刷新页面，正常发送 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec70e2e86bf238?w=2050&h=1036&f=png&s=276853)

此时访问 `localhost:3000`，因为前面的代码，所以会返回 `script`、`img`、`iframe`，然后用户代理会加载这三个资源：

![](https://user-gold-cdn.xitu.io/2019/12/2/16ec7164d3f0cc01?w=2228&h=505&f=png&s=141366)

打开这三个资源，可以看到，每个资源都携带了一个 `cookie`。

- script.js
![](https://user-gold-cdn.xitu.io/2019/12/2/16ec716f2188913f?w=1784&h=948&f=png&s=241838)

- img.jpg
![](https://user-gold-cdn.xitu.io/2019/12/2/16ec717795abbe8b?w=1674&h=1006&f=png&s=247438)

- index.html
![](https://user-gold-cdn.xitu.io/2019/12/2/16ec717cef61b9b8?w=1802&h=1038&f=png&s=257103)

而 `SameSite` 就是用来限制这种情况：

##### Strict
当 `SameSite` 为 `Strict` 的时候，用户代理只会发送和网站 `URL` 完全一致的 `cookie`，就算是子域名都不行。

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    if (req.headers.host.startsWith('example')) {
        res.writeHead(200, {'Set-Cookie': `host=${req.headers.host};SameSite=Strict`})
    }
    if (req.headers.host.startsWith('localhost')) {
        res.write(`
            <a href="http://example.com:3000/index.html">http://example.com:3000/index.html</a>
            <script src="http://example.com:3000/script.js"></script>
            <img src="http://example.com:3000/img.jpg"/>
            <iframe src="http://example.com:3000/index.html"/>
        `)
    }
    res.end()
}).listen(3000)

```

- 访问 `example.com:3000` 获得 `cookie`:
![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbb69810d93c5?w=1590&h=902&f=png&s=207403)

- 刷新正常发送 `cookie`：
![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbb77ab8eba6f?w=1384&h=992&f=png&s=229913)

- 访问 `localhost:3000` 任何指向 `example.com:3000` 的资源都不会发送 `cookie`：
![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbb89a6646236?w=1580&h=914&f=png&s=246206)

- 包括从这个页面跳转过去：
![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbb9386aef8e0?w=1558&h=946&f=png&s=277703)

- 子域名
![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbbc9efd2528a?w=1746&h=1072&f=png&s=287588)

##### Lax
新版浏览器默认的 `SameSite` 是 `Lax`，如果 `SameSite` 是 Lax，则将会为一些跨站子请求保留，如图片加载或者 `frames` 的调用，但只有当用户从外部站点导航到URL时才会发送。如 `link` 链接。（我没能测试出来，具体看[阮一峰-Cookie 的 SameSite 属性](http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)和[MDN-HTTP cookies](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)）

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    if (req.headers.host.startsWith('example')) {
        res.writeHead(200, {'Set-Cookie': `host=${req.headers.host};SameSite=Lax`})
    }
    if (req.headers.host.startsWith('localhost')) {
        res.write(`
            <a href="http://example.com:3000/index.html">http://example.com:3000/index.html</a>
            <form action="http://example.com:3000/form" method="post">
                <button>post</button>
            </form>
            <form action="http://example.com:3000/form" method="get">
                <button>get</button>
            </form>
            <script src="http://example.com:3000/script.js"></script>
            <img src="http://example.com:3000/img.jpg"/>
            <iframe src="http://example.com:3000/index.html"/>
        `)
    }
    res.end()
}).listen(3000)

```

以下是我的测试结果
- a 链接跳转：带 cookie
- form[action=get]：带 cookie
- form[action=post]：不带 cookie
- iframe：不带 cookie
- script：不带 cookie
- img：不带 cookie

也就是简单导航跳转带，而资源加载不带。

#### None
在新版浏览器，如果要让 `cookie` 支持同站和跨站，需要明确指定 `SameSite=None`（我测试出来的结果是和没有添加这个属性是一致的）。

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    if (req.headers.host.startsWith('example')) {
        res.writeHead(200, {'Set-Cookie': `host=${req.headers.host};SameSite=None`})
    }
    if (req.headers.host.startsWith('localhost')) {
        res.write(`
            <a href="http://example.com:3000/index.html">http://example.com:3000/index.html</a>
            <form action="http://example.com:3000/form" method="post">
                <button>post</button>
            </form>
             <form action="http://example.com:3000/form" method="get">
                <button>get</button>
            </form>
            <script src="http://example.com:3000/script.js"></script>
            <img src="http://example.com:3000/img.jpg"/>
            <iframe src="http://example.com:3000/index.html"/>
            
        `)
    }
    res.end()
}).listen(3000)
```

#### 8. Set-Cookie 的格式
其实 `Set-Cookie` 的格式是由 `cookie` 键值对和属性列表构成的，所以可以用如下表示(不用 ABNF)：
```
    Set-Cookie: cookie-pair ";" cookie-av ";" cookie-av....
```
其中 `cookie-pair` 就是我们上面写的类似 `name=123` 的格式：
```
    cookie-pair = cookie-name "=" cookie-value
```
后面可以跟一系列的属性，`cookie-pair` 和 `cookie-av` 之间使用 `;` 分割，而 `cookie-av` 可以表示为：
```
    cookie-av = expires-av / max-age-av / domain-av /
                path-av / secure-av / httponly-av /
                extension-av
```
其中的` expires-av`、`max-age-av`、`domain-av`、`path-av`、`secure-av`、`httponly-av` 就对应前面一张图的各个属性，但是有点区别，`Chrome` 实现了 `same-origin`属性，但是在 `RFC 6265` 并没有这个属性,可以看作是 `extension-av`，

再做一次对应：
- Name：cookie-name
- Value：cookie-value
- Domain：domain-av
- Path：path-av
- Expire / Max-Age ：expires-av / max-age-av
- Size（忽略，应该只是前端统计 Cookie 键值对的长度，比如"name"和"123"的长度是 7，如果有大神知道请告知）
- HttpOnly：httponly-av
- Secure：secure-av
- SameSite：extension-av

可以看到其实 cookie 的格式并不是和 `Chrome` 中的 `cookies` 一致，它将 `cookie-pair` 和 `cookie-av` 平铺开来。

### 0x003 `Cookie` 在各种情况下的反映

#### 1. 每种状态码下 `Set-Cookie` 都会被处理吗？包括 `301`、`404`、`500`？

根据 `RFC 6265`，用户代理可能忽略包含在 `100` 级别的状态码，但是必须处理其他响应中的 `Set-Cookie`（包括 `400` 级别和 `500` 级别）。

根据测试，除了 `100`，`101`，还有一个 `407`（未知为啥）不会处理 `Set-Cookie`，甚至 999 都会处理

![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbebe38cc88ef?w=2094&h=1578&f=png&s=429619)

#### 2. 如何发送多个 cookie 到 Set-Cookie，Cookie 又是如何处理的？

上代码：
```
const http = require('http');

http.createServer((req, res) => {
    res.setHeader('Set-Cookie', ['cookie1=1','cookie1=2','cookie2=2','cookie3=3'])
    res.end()
}).listen(3000)
```
访问 `localhost:3000`，得到 `cookie`：
![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbf54a0611309?w=2180&h=1326&f=png&s=300228)

可以看到，`cookie` 被放到多个 `Set-Cookie` 头部中，而在 `RFC 7230` 中其实规定，一个报文中不应该出现重复的头部，如果一个头部有多个值，应该使用 , 分隔，就像下面的` Accept-Encoding` 和 `Accept-Language`。但问题是，`,` 可以作为 `cookie-value` 的合法值存在，如果降它作为分隔符会破坏 `cookie` 的解析😢，所以就这样咯。


刷新发送 `Cookie`，可以看到，被乖乖折叠了，并且重复的 `cookie-name` 的 `cookie` 被按顺序覆盖了：

![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbf84ed5a9005?w=1800&h=1247&f=png&s=318360)

#### 3. ajax 如何发送 Cookie，ajax 返回的 Set-Cookie 会被接受吗？

答案是会，这里以 `axios` 为例子，上代码：
```
const http = require('http');

http.createServer((req, res) => {
    res.writeHead(200, {'Set-Cookie': 'name=ajax'})
    res.write(`
        <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
        <script >
        axios.get('http://example.com:3000/', {
            withCredentials: true
        })        
        </script>
    `)
    res.end()
}).listen(3000)
```

访问 `localhost:3000`，得到 `cookie`：

![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbfd12382c664?w=1936&h=1118&f=png&s=271994)

查看 `cookie` 存储，已经存上了：

![](https://user-gold-cdn.xitu.io/2019/12/3/16ecbfd7ae3330eb?w=1860&h=564&f=png&s=102766)

查看 `ajax` 请求，已经发送了：

![](https://user-gold-cdn.xitu.io/2019/12/3/16ecc060395ecadd?w=1588&h=1118&f=png&s=261477)

这里遵循同源策略，如果是同源，则默认会发送，如果是跨域，则需要添加`withCredentials: true`才行，`XMLHttpRequest` 和 `Fetch` 配置可能有所不同。

#### 暂时想不到了

### 0x004 CSRF

#### 例子

- 打开登陆页面 `localhost:3000/login`，登陆之后，后端返回一个 `Set-Cookie`: `name=bob`，后端读取 `Cookie` 判断用户是否登陆，如果登陆，就会显示账户金额，和一个转账表单


![](https://user-gold-cdn.xitu.io/2019/12/4/16eccb0130fce5ef?w=2876&h=1376&f=png&s=454757)

- 转账是通过 `http://localhost:3000/transfer?name=lucy&money=1000` 来转账的，`name` 是转账目标用户，`money` 是金额。这里为了方便，直接使用 `GET`。


![](https://user-gold-cdn.xitu.io/2019/12/4/16eccb3fbd0ab7e9?w=2878&h=1642&f=png&s=480899)

- 重新开一个服务，假装是另一个站点，这个站点只有一个 `img`，该 `img` 的 `src` 指向了上面的转账地址：
```
const http = require('http');
http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type':'text/html'})
    res.write(`
        <img src="http://localhost:3000/transfer?name=bad&money=10000000">
    `)
    res.end()
}).listen(3001)
```

- 已经在 `localhost:3000`登陆的 `bob` 访问 `example.com:3001`，就会发现，虽然只是一个图片指向了 `localhost:3000`，但是因为能够获取到 `localhost:3000` 的 `cookie`，所以会被带过去，而这个地址根据 `cookie` 判断用户，并执行转账操作：

![](https://user-gold-cdn.xitu.io/2019/12/4/16eccb62239fe4f0?w=2872&h=1642&f=png&s=415744)

- 如果再 bob 回到自己的账户页面，就会发现，他已然破产，被限制高消费：
![](https://user-gold-cdn.xitu.io/2019/12/4/16eccb913dcd2b54?w=1234&h=558&f=png&s=55383)


#### 防御
（本文章主要讲 `cookie`，`csrf` 原理和防御不是重点）

- 检测 `Referrer` 头：但是一些浏览器可以禁用这个头部（我也没找到哪个浏览器可以设置）
- `SameSite`：尚未完全普及
- `csrftoken`：一般 `csrf` 发起攻击都是类似上面，搭建一个钓鱼网站，其实该钓鱼网站是无法访问源站的 `cookie` 的，发送 `cookie` 是浏览器自带的行为，所以只要生成一个随机的 `token`，在每个表单发送的时候都带上就行了，因为钓鱼网站无法预测这个 `token` 的值。甚至可以将这个 `token` 直接存储在 `cookie` 中，在表单发送的时候取出来和表单一起发送。也可以后端生成表单的时候注入到一个 `inputp[typehidden]` 域。
- 二次验证，比如验证码、支付密码等


### 0x005 资源
- [案例源码](https://github.com/followWinter/cookie-test)
- [RFC 6265-HTTP State Management Mechanism](https://tools.ietf.org/html/rfc6265)
- [MDN HTTP Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookie)
- [阮一峰-Cookie SameSite](http://www.ruanyifeng.com/blog/2019/09/cookie-samesite.html)

### 0x006 带货

最近发现一个好玩的库，作者是个大佬啊--[基于 React 的现象级微场景编辑器](https://juejin.im/post/5db718b7f265da4d1e26cb9c)。

---
