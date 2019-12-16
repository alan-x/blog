### 0x001 概述
上一章讲的时候关于[文件相关的loder][1]，这一章讲的是模板相关的`loder`。

### 0x002 环境配置
    ```
    $ mkdir 0x010-templating-loader
    $ cd 0x010-templating-loader
    $ npm init -y
    $ npm install --save-dev webpack
    ```
### 0x003 栗子1-`html-loader`-加载html
1. 安装依赖包
    ```
    $ cnpm install --save-dev html-loader
    ```
2. 编写`index.html`并引入
    ```
    // ./src/content.html
    <p>hello webpack</p>
    <img src="./../res/icon.jpg"/>
    
    // ./src/index.html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>templating-loader</title>
    </head>
    <body>
    
    </body>
    <script src="./../dist/index.bundle.js"></script>
    
    </html>
    // ./src/index.js
    require('./index.html')
    ```
3. 配置`webpack.config.js`
    ```
    const path = require('path');
    
    module.exports = {
        entry : {
            'index': ['./src/index.js'],
        },
        output: {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
        module: {
            rules: [
                {
                    test: /\.html/,
                    use : {
                        loader:'html-loader',
                        options: {
                            attrs: [':data-src'],
                            minimize          : true,
                            removeComments    : false,
                            collapseWhitespace: false
                        }
                    }
                },
                {
                    test: /\.(png|jpg|gif)$/,
                    use: [
                        {
                            loader : 'url-loader',
                            options: {
                                limit   : 1048576, // 低于1m
                                name    : '[name].[hash].[ext]',
                                fallback: 'file-loader' //否则使用`file-loader`
                            }
                        }
                    ]
                }
            ]
        }
    };
    ```
4. 打包并查看结果
    ```
    // ./dist/index.bundle.js
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var content = __webpack_require__(2)
    document.write(content)
    
    
    /***/ }),
    /* 2 */
    /***/ (function(module, exports) {
    
    module.exports = "<p>hello webpack</p>\n<img src=\"./../res/icon.jpg\"/>";
    
    /***/ })
    /******/ ]);
    ```
    可以看到，html被打包成了字符串，并封装成模块导出。
    **注意**：看配置文件，除了配置一个`html-loader`，还配置了一个`url-loader`，因为当`<img src="./../res/icon.jpg"/>`时，会解析为`require('./../res/icon.jpg')`,因此，需要一个`loader`来解析这个资源，所以配置了一个`url-loader`

5. 其他配置
    - `removeComments`：移除评论
    - `collapseWhitespace`：删除空格
    - `removeAttributeQuotes`：删除属性的`"`号
    - `keepClosingSlash`：保持标签闭合
    - `minifyJS`：压缩JS
    - `minifyCSS`:压缩CSS
    
#### 0x004 栗子2-配合`extra-loader`将`html`导出成文件
1. 修改文件
    ```
            {
                test: /\.html/,
                use : [
                    {
                        loader : "file-loader",
                        options: {
                            name: "[name]-dist.[ext]",
                        },
                    },
                    {
                        loader: "extract-loader",
                    },
                    {
                        loader : 'html-loader',
                        options: {
                            attrs             : [':data-src'],
                            minimize          : true,
                            removeComments    : false,
                            collapseWhitespace: false
                        }
                    }
                ]
            },
    ```
2. 安装依赖
    ```
    $ cnpm install --save-dev extact-loader file-loader
    ```
3. 打包并查看结果
    ```
    // ./dist
    content.dist.html
    index.bundle.js
    
    // ./dist/index.bundle.js    
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var content = __webpack_require__(2)
    document.write(content)
    
    
    /***/ }),
    /* 2 */
    /***/ (function(module, exports, __webpack_require__) {
    
    module.exports = __webpack_require__.p + "content-dist.html";
    
    /***/ })
    /******/ ]);
    ```
4. 其他更多配置，查看[webpack关于html-loader部分][2]

### 0x005 栗子3-`pug-loader`：`pug`模板解析
1. 安装依赖
    ```
            {
                test: /\.pug$/,
                use : "pug-loader"
            },
    ```
2. 添加配置
    ```
    {
         test: /\.pug$/,
         use : "pug-loader"
    },
    ```
4. 调用
    ```
    var content = require('./content.pug')
    document.write(content())
    ```
5. 打包并查看结果
    ```
    
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
   
    var content = __webpack_require__(2)
    document.write(content())
   
    /***/ }),
    /* 2 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var pug = __webpack_require__(3);
    
    function template(locals) {var pug_html = "", pug_mixins = {}, pug_interp;pug_html = pug_html + "\u003Cp id=\"name\"\u003E张三\u003C\u002Fp\u003E";;return pug_html;};
    module.exports = template;
    
    /***/ }),
    ...
    ```
    可以看到`pug`内容被打包成了一个返回`html`字符串的函数，并且该函数被封装成模块。

6. 更多资源，请查阅[pug-loader的仓库][3]和[pug官网][4]

### 0x006 资源
- [源代码][5]


  [1]: https://segmentfault.com/a/1190000011926527
  [2]: https://webpack.js.org/loaders/html-loader/
  [3]: https://github.com/pugjs/pug-loader/tree/0298112c0504169f53cc18fcafd319dacb5a6bda#options
  [4]: https://pugjs.org/api/getting-started.html
  [5]: https://github.com/followWinter/webpack-study/tree/master/0x010-templating-loader
