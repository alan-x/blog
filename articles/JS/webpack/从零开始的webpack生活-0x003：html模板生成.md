### 0x001 概述
[上一章][1]之后已经可以自动刷新浏览器了，大大方便了我们的开发，这章开始讲插件，第一个插件将会解决上一章节的一个问题，那就是`index.html`需要手动插入打包好的js，同时不会将`index.html`一起放到`dist`文件夹下的问题。

### 0x002 初始化项目
1. 创建项目文件夹`0x003-html-webpack-plugin`，我们将在这个文件夹底下开始这一章节的所有编码
2. 复制上一章的文件及其目录，除了`dist`和`index.html`
    ```
    + webpack-study
      + 0x001-begin
      + 0x002-dev-server
      + 0x003-html-webpack-plugin
        + src
          - index.js
        - package.json
        - webpack.config.js
    ```

### 0x003 简单配置`html-webpack-plugin`并使用
1. 安装
    ```
    # 终端输入
    $ cnpm install --save-dev html-webpack-plugin
    # 输出
    ✔ Installed 3 packages
    ...
    ✔ All packages installed (473 packages installed from npm registry, used 7s, speed 1.08MB/s, json 434(768.7kB), tarball 7.09MB)
    ```
2. 配置
- 初始化插件
    ```
    var HtmlWebpackPlugin = require('html-webpack-plugin');
    
    ```
- 添加插件
    ```
    plugins  : [
        new HtmlWebpackPlugin()
    ]
    ```
- 最终配置文件
    ```
    var path              = require('path')
    var HtmlWebpackPlugin = require('html-webpack-plugin');
    
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
        plugins  : [
            new HtmlWebpackPlugin()
        ]
    }
    
    ```
4. 打包
    ```
    $ npm run build
    Hash: 1d3993547a22839c5053
    Version: webpack 3.8.1
    Time: 413ms
         Asset       Size  Chunks             Chunk Names
      index.js  510 bytes       0  [emitted]  main
    index.html  181 bytes          [emitted]  
       [0] ./src/index.js 32 bytes {0} [built]
    Child html-webpack-plugin for "index.html":
         1 asset
           [2] (webpack)/buildin/global.js 488 bytes {0} [built]
           [2] (webpack)/buildin/module.js 495 bytes {0} [built]
            + 2 hidden modules
    
    ```
此时查看`dist`，会发现生成了两个文件
- `index.js`：`webpack`打包生成的`js`文件
- `index.html`：`htmlWebpackPlugin`自动生成
观察`index.html`可以发现，`htmlWebpackPlugin`不只是生成了一个`html`文件，还添加了对我们打包生成的`index.js`的引用。
    ```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <title>Webpack App</title>
      </head>
      <body>
      <script type="text/javascript" src="index.js"></script></body>
    </html>
    ```

### 0x004 插件配置
1. 自定义`title`
    - 添加配置
        ```
        // package.json
        new HtmlWebpackPlugin({
                    title: '自动插入标题'
                })
        ```
    - 打包
        ```
        <!-- dist/index.html -->
        <!DOCTYPE html>
        <html>
          <head>
            <meta charset="UTF-8">
            <title>自动插入标题</title>
          </head>
          <body>
          <script type="text/javascript" src="index.js"></script></body>
        </html>
        ```
        
2. 自定义文件名
    - 添加配置
        ```
            new HtmlWebpackPlugin({
                title   : '自动插入标题',
                filename: 'admin.html',
                }）
        ```
    - 打包查看文件
        ```
        + dist
          - admin.html
          - index.js
         
        ```
    
3. 根据模板生成
    - 添加模板文件`./index.html`
        ```
    <!-- ./index.html -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta charset="UTF-8">
        </head>
        <body>
            <p>这是一个模板文件</p>
        </body>
    </html>
        ```
    - 添加配置
        ```
         new HtmlWebpackPlugin({
                    title   : '自动插入标题',
                    filename: 'admin.html',
                    template      : path.resolve(__dirname, 'index.html')
                })
        ```
    - 打包
        ```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
      </head>
      <body>
          <p>这是一个模板文件</p>
      <script type="text/javascript" src="index.js"></script>
      </body>
    </html>
        ```
4. 自定义`js`文件注入位置
    - 添加配置
        ```
            new HtmlWebpackPlugin({
                        title   : '自动插入标题',
                        filename: 'admin.html',
                        template      : path.resolve(__dirname, 'index.html'),
                        inject        : 'head' 
                    })
        ```
    - 打包
        ```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <script type="text/javascript" src="index.js"></script>
      </head>
      <body>
        <p>这是一个模板文件</p>
      </body>
    </html>
        ```
    - 可配置的值：
        - `true`：注入
        - `false`：不注入
        - `'head'`：注入头部
        - `'body'`：注入`body`底部
5. 压缩`html`
    - 添加配置
        ```
                new HtmlWebpackPlugin({
            title         : '自动插入标题',
            filename      : 'admin.html',
            template      : path.resolve(__dirname, 'index.html'),
            inject        : 'head',
            minify:{
                collapseWhitespace:true,
            }
        }),
        ```
    - 打包
        ```
        <!-- ./dist/admin.html -->
        <!DOCTYPE html><html><head><meta charset="UTF-8"><script type="text/javascript" src="index1.87e0a7e4f4842924a964.js"></script><script type="text/javascript" src="index3.112a255042f8daa92065.js"></script></head><body><p>这是一个模板文件</p></body></html>
        ```
    - 可选配置
    这里的值是一个对象，具体可以配置哪些可以看[html-minifier的配置说明][3]
5. 多入口的情况下自定义插入的`chunk`
    - 添加入口文件`index2.js`、`index3.js`
        ```
        // ./src/index2.js
        document.write('hello index2')
        // ./src/index3.js
        document.write('hello index3')
        ```
    - 修改`entry`、`output`、`plugin`
        ```
        var path              = require('path')
        var HtmlWebpackPlugin = require('html-webpack-plugin');
        
        module.exports = {
            entry    : {
                index1: './src/index.js',
                index2: './src/index2.js',
                index3: './src/index3.js',
            },
            output   : {
                path    : path.resolve(__dirname, 'dist'),
                filename: '[name].[chunkhash].js'
            },
            devServer: {
                contentBase: path.resolve(__dirname, ''),
                port       : 9000,
                compress   : true,
                open       : true,
                host       : '0.0.0.0',
                index      : 'index.html'
            },
            plugins  : [
                new HtmlWebpackPlugin({
                    title   : '自动插入标题',
                    filename: 'admin.html',
                    template: path.resolve(__dirname, 'index.html'),
                    inject  : 'head',
                    chunks  : ['index', 'index3']
                })
            ]
        }
        
        ```
        - 打包
        ```
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <script type="text/javascript" src="index1.87e0a7e4f4842924a964.js"></script>
            <script type="text/javascript" src="index3.112a255042f8daa92065.js"></script>
        </head>
        <body>
        <p>这是一个模板文件</p>
        </body>
        </html>
        ```
    **注意**：注入的顺序和chunks的顺序相反

7. 自定义注入`chunk`的顺序

    - 修改配置
        ```
            new HtmlWebpackPlugin({
                title         : '自动插入标题',
                filename      : 'admin.html',
                template      : path.resolve(__dirname, 'index.html'),
                inject        : 'head',
                chunks        : ['index1', 'index3'],
                chunksSortMode: function (chunk1, chunk2) {
                    return -1;
                }
            })
        ```
    - 打包
        ```
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <script type="text/javascript" src="index3.112a255042f8daa92065.js"></script>
            <script type="text/javascript" src="index1.87e0a7e4f4842924a964.js"></script>
        </head>
        <body>
        <p>这是一个模板文件</p>
        </body>
        </html>
        ```
    - 可选值
        - `none`：不排序
        - `'auto'`：根据thunk的id排序
        - `'dependency'`：根据依赖排序
        - `'manual'`：找不到文档啊，不知道说的是啥
        - `function`：提供一个函数计算排序方式，会自动调用这个函数来计算排序，可以根据`chunk1.names[0]`和`chunk2.names[0]`对比计算出来，如果返回大于1的数，则chunk1在前，chunk2在后，反之亦然。调试的时候可以直接在函数中`console.log(chunk1, chunk2)`来调试。

8. 生成多页面
    - 修改配置
        ```
         plugins  : [
            new HtmlWebpackPlugin({
                title         : '自动插入标题',
                filename      : 'admin.html',
                template      : path.resolve(__dirname, 'index.html'),
                inject        : 'head',
                chunks        : ['index1', 'index3'],
                chunksSortMode: function (chunk1, chunk2) {
                    return 1;
                }
            }),
            new HtmlWebpackPlugin({
                title         : '第二个页面',
                filename      : 'index.html',
                template      : path.resolve(__dirname, 'index.html'),
                inject        : 'head',
                chunks        : ['index1', 'index2'],
                chunksSortMode: function (chunk1, chunk2) {
                    return 1;
                }
            })
    
        ]
        ```
    - 打包并查看`dist`
        ```
        # dist 文件夹结构
        + dist
          - index.html
          - admin.html
          - ...
          
        <!-- ./dist/admin.html -->
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <script type="text/javascript" src="index1.87e0a7e4f4842924a964.js"></script>
            <script type="text/javascript" src="index3.112a255042f8daa92065.js"></script>
        </head>
        <body>
        <p>这是一个模板文件</p>
        </body>
        </html>
        
        <!-- ./dist/index.html -->
        <!DOCTYPE html>
        <html>
          <head>
            <meta charset="UTF-8">
          <script type="text/javascript" src="index1.87e0a7e4f4842924a964.js"></script><script type="text/javascript" src="index2.72cf280e7df62106422b.js"></script></head>
          <body>
          <p>这是一个模板文件</p>
          </body>
        </html>
        ```
        
9. 此时的配置
    ```
    // webpack.config.js
    var path              = require('path')
    var HtmlWebpackPlugin = require('html-webpack-plugin');
    
    module.exports = {
        entry    : {
            index1: './src/index.js',
            index2: './src/index2.js',
            index3: './src/index3.js',
        },
        output   : {
            path    : path.resolve(__dirname, 'dist'),
            filename: '[name].[chunkhash].js'
        },
        devServer: {
            contentBase: path.resolve(__dirname, ''),
            port       : 9000,
            compress   : true,
            open       : true,
            host       : '0.0.0.0',
            index      : 'index.html'
        },
        plugins  : [
            new HtmlWebpackPlugin({
                title         : '自动插入标题',
                filename      : 'admin.html',
                template      : path.resolve(__dirname, 'index.html'),
                inject        : 'head',
                minify:{
                    collapseWhitespace:true,
                },
                chunks        : ['index1', 'index3'],
                chunksSortMode: function (chunk1, chunk2) {
                    return 1;
                }
            }),
            new HtmlWebpackPlugin({
                title         : '第二个页面',
                filename      : 'index.html',
                template      : path.resolve(__dirname, 'index.html'),
                inject        : 'head',
                minify:{
                    collapseWhitespace:true,
                },
                chunks        : ['index1', 'index2'],
                chunksSortMode: function (chunk1, chunk2) {
                    return 1;
                },
            })
    
        ]
    }
    
    ```
10 其他配置看[这里][4]
### 0x005 资源
- [源代码][5]


  [1]: https://segmentfault.com/a/1190000011867390
  [2]: https://github.com/jantimon/html-webpack-plugin#configuration
  [3]: https://github.com/kangax/html-minifier#options-quick-reference
  [4]: https://github.com/jantimon/html-webpack-plugin#configuration
  [5]: https://github.com/followWinter/webpack-study/tree/master/0x003-html-webpack-plugin
