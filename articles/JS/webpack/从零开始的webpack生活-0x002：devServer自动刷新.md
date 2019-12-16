### 0x001 概述
[上一章][1]已经实现了最简单的`webpack`配置文件使用和`webpack`监听功能，这一章要开始实现自动刷新。

### 0x002 浏览器自动刷新
1. 创建新的项目框架
    ```
    - webpack_study
        + 0x001-begin
        - 0x002-dev-server
           - src
    ```
我们将在`0x002-dev-server`下做所有的开发

2. 初始化项目，这里使用`npm init`指令初始化，生成`package.json`，以便以后管理依赖包
    ```
    $ npm init -y
    # 输出
    Wrote to /MY_PROJECT/PROJECT_OWN/webpack_study/0x002-dev-server/package.json:
    
    {
      "name": "1-web-auto-refresh",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
      },
      "keywords": [],
      "author": "",
      "license": "ISC"
    }
    ```
3. 创建`./src/index.js`和`./index.html`,并添加对`js`的引用
    ```
    # ./src/index.js
    console.log('hello wbpack');
    ```
    
    ```html
    <!--./index.html-->
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>1-web-auto-refresh</title>
        <script src="./index.js"></script>
    </head>
    <body>
    
    </body>
    </html>
    ```
4. 复制上一章的`webpack.config.js`配置文件
    ```
    var path       = require('path')
    module.exports = {
        entry : './src/index.js',
        output: {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        }
    }
    ```
5. 配置`devServer`
    ```
    var path       = require('path')
    module.exports = {
        entry    : './src/index.js',
        output   : {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        },
        devServer: {
            contentBase: path.join(__dirname, "dist"),
            port       : 9000
        }
    }
    ```
    `devServer`提供了一个`web`服务器
    - `contentBase`为该服务器的根目录，它将以此来寻找资源
    - `port`为端口，我们通过该端口访问该服务器的资源
6. 安装依赖包`webpack-dev-server`、`webpack`
    ```
    $ cnpm install --save-dev webpack
    $ cnpm install -g webpack-dev-server 
    # 输出
    Downloading webpack-dev-server to /usr/local/lib/node_modules/webpack-dev-server_tmp
    ...
    [webpack-dev-server@2.9.4] link /usr/local/bin/webpack-dev-server@ -> /usr/local/lib/node_modules/webpack-dev-server/bin/webpack-dev-server.js

    $ webpack-dev-server -v
    # 输出
    webpack-dev-server 2.9.4
    webpack 3.8.1
    

    
    ```
7. 启动`devServer`
    ```
    $ webpack-dev-server
    # 输出
    Project is running at http://localhost:9000/
    webpack output is served from /
    Content not from webpack is served from /MY_PROJECT/PROJECT_OWN/webpack_study/0x002-dev-server/1-web-auto-refresh/dist
    Hash: ab62a2a6acbc29a572c6
    Version: webpack 3.8.1
    Time: 338ms
       Asset    Size  Chunks                    Chunk Names
    index.js  324 kB       0  [emitted]  [big]  main
       [2] multi (webpack)-dev-server/client?http://localhost:9000 ./src/index.js 40 bytes {0} [built]
    ...
        + 11 hidden modules
    webpack: Compiled successfully.

    ```
5. 接着我们就可以通过`http://loaclhost:9000`来访问这个网站了！

**注意**
- `webpack-dev-server`指令拥有与`webpack -w`相同的功能，在代码修改的时候根据webpack配置文件自动打包，在出错的时候输出错误信息，还能在修改代码的时候自动刷新浏览器。
- 自动刷新浏览器只有在修改配置文件中`entry`配置的入口文件及其所引用的文件有效，其他则无效，比如没有引用到的文件和`index.html`
- 自动打包出来的`index.js`此时在内存中，是看不见的

### 0x003 其他配置
`devServer`还有许多其他常用配置，这里只讲常用的，暂时不讲热更新
- `compress`:是否`gzip`压缩
- `host`:访问地址，默认是`localhost`，但是可以配置成'0.0.0.0',就可以使用`127.0.0.1`访问了
- `index`:不指定访问资源时，默认访问`contenBase`路径下的`index.html`文件，但是也可以通过指令`index`改变默认访问文件
- `open`:执行`webpack-dev-server`后，我们需要自己打开浏览器访问`localhost:9000`访问，配置该指令可以在启动`webpack-dev-server`后自动打开一个浏览器窗口
- `progress`:显示打包进度，用于命令行
其他配置可查阅[webpack-devServer][2]章节
注意：
- 配置指令分为两种，一种是只用在终端使用，比如`progress`，只能在终端指定`webpack-dev-server --progress`；一种既可以在终端，也可以在配置文件，比如`compress`,既可以在配置文件中指定`compress:true`，也可以在终端中指定`webpack-dev-server --compress`，在终端中指定使用`-`连接每个指令，在配置文件中，使用驼峰法指定。只能在终端中使用的在[webpack-devServer][3]章节中指令后标有`CLI only`
- 可以使用`npm`的`script`功能，快速调用`webpack-dev-server`
    ```
    # package.json
      "scripts": {
        "dev":"webpack-dev-server --progress",
        "build":"webpack -p"
      }
    # 终端
    $ npm run dev
    $ npm run build
    ```
### 0x004 最终项目
1. 文件夹结构
    ```
    - 0x002-dev-server
        - src
            index.js
        package.json
        webpack.config.js
       
    ```
2. `index.js`
    ```
    document.write('hello webpack')
    ```
3. `package.json`
    ```
    {
      "name": "1-web-auto-refresh",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "dev":"webpack-dev-server --progress"
      },
      "keywords": [],
      "author": "",
      "license": "ISC",
      "devDependencies": {
        "webpack": "^3.8.1"
      }
    }
    
    ```
4. `webpack.config.js`
    ```
    var path       = require('path')
    module.exports = {
        entry    : './src/index.js',
        output   : {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        },
        devServer: {
            contentBase: path.resolve(__dirname, ''),
            port       : 9000,
            compress   : true,
            open       : true,
            host       : '0.0.0.0',
            index      : 'index.html'
        },
    }
    ```
### 0x005 资源
- [源代码][4]


  [1]: https://segmentfault.com/a/1190000011865966
  [2]: https://webpack.js.org/configuration/dev-server/
  [3]: https://webpack.js.org/configuration/dev-server/
  [4]: https://github.com/followWinter/webpack-study/tree/master/0x002-dev-server
