### 0x001 概述
上一章讲的是[装载模板][1]，这一章讲的是装载样式

### 0x002 配置环境
```
$ mkdir 0x011-styling-loader
$ cd 0x011-styling-loader
$ npm init -y
$ npm install --save-dev webpack
$ touch ./src/index.js
$ vim webpack.config.js

// ./webpack.config.js
const path = require('path');

module.exports = {
    entry : {
        'index': ['./src/index.js'],
    },
    output: {
        path    : path.join(__dirname, 'dist'),
        filename: '[name].bundle.js'
    }
;
```
### 0x003 栗子1-`css-loader`装载`css`
1. 安装依赖
    ```
    $ cnpm install --save-dev css-loader
    ```
2. 添加`style.css`
    ```
    $ vim ./src/style.css
    p {
        color: red;
    }
    ```
3. 引入`style.css`
    ```
    // ./src/index.js
    var style = require("./style.css")
    console.log(style.toString())
    ```
4. 打包并查看结果
    ```
    /* 2 */
    /***/ (function(module, exports, __webpack_require__) {

    exports = module.exports = __webpack_require__(3)(false);
    exports.push([module.i, "p{color:red}", ""]);
    /***/ }),
    /* 3 */
    /***/ (function(module, exports) {
    ...
    module.exports = function(useSourceMap) {
    	var list = [];
    
    	// return the list of modules as css string
    	list.toString = function toString() {
    		return this.map(function (item) {
    			var content = cssWithMappingToString(item, useSourceMap);
    			if(item[2]) {
    				return "@media " + item[2] + "{" + content + "}";
    			} else {
    				return content;
    			}
    		}).join("");
    	};
    	....
    ```
    可以看到，生成了两个新的模块，模块2包含我们的`style`文件中的内容，模块3导出了一个`toString`,它的作用是可以将`style.css`中的内容转化成字符串来使用，所以我们在`index.js`中可以使用`style.toString()`来得到样式字符串，执行结果
    ```
    $ node ./dist/index.bundle.js
    p {
        color: red;
    }
    ```
5. 其他配置
    - `minimize`：压缩css
6. 更多配置配置，请查阅[webpack关于css-loader章节][2]
### 0x004 栗子2-`style-loader`配合`css-loader`插入
1. 安装依赖
    ```
    $ cnpm install --save-dev css-loader
    ```
2. 修改配置
    ```
            {
                test: /\.css$/,
                use : [
                    {
                        loader : 'css-loader',
                        options: {}
                    },
                    {
                        loader : 'css-loader',
                        options: {
                            minimize: true
                        }
                    }
                ]
            }
    ```
3. 添加`index.html`
    ```
    <html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport"
              content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
        <script src="./../dist/index.bundle.js"></script>
    </head>
    <body>
    <p>hello webpack</p>
    </body>
    </html>
    ```
4. 浏览器打开`./src/index.html`,可以看到我们写的`style.css`内容已经被插入到`head`的`style`标签中

    ![clipboard.png](/img/bVYdlV)

5. 其他更多配置请查阅[webpack关于style-loader章节][3]

### 0x005 栗子3-添加`file-loader`独立出`style`文件
1. 安装依赖
    ```
    $ cnpm install --save-dev file-loader
    ```
2. 修改配置
    ```
    {                           
        loader : 'style-loader',
        options: {}             
    },                          
    {                           
        loader : 'file-loader', 
        options: {              
            name:'[name].[ext]' 
        }                       
    },                          
    ```
3. 打包并用浏览器打开`./src/index.html`，可以看见，`style.css`被以文件的形式导出并在`head`中以外链的形式导入
    ![clipboard.png](/img/bVYdlE)
    
4. 其他更多配置查阅[webpack关于style-loader章节][4]

### 0x006 栗子4-`sass-loader`装载sass
1. 安装依赖
    ```
    $ npm install sass-loader node-sass webpack --save-dev
    ```
2. 修改配置
    ```
    {                             
        test: /\.(scss|css)$/,    
        use : [{                  
            loader: "style-loader"
        }, {                      
            loader: "css-loader"  
        }, {                      
            loader: "sass-loader" 
        }]                        
    }                             
    ```
3. 打包并使用浏览器打开`index.html`，可以看到，不管是`style.css`还是`style.scss`都被`style-loader`插入到了`head`

    ![clipboard.png](/img/bVYdpG)
4. 更多设置查阅关于[webpack关于sass-loader章节][5]

### 0x007 栗子5-`postcss-loder`搞定兼容性
1. 安装依赖
```
$ cnpm install --save-dev postcss-loader precss autoprefixer
```
2. 添加配置
```
{                                    
    test: /\.(scss|css)$/,           
    use : [                          
        {                            
            loader: "style-loader"   
        },                           
        {                            
            loader: "css-loader"     
        },                           
        {                            
            loader: "postcss-loader" 
        },                           
        {                            
            loader: "sass-loader"    
        }]                           
}                                    
```
3. 添加`postcss`配置
```
$ vim ./postcss.config.js
// ./postcss.config.js
const precss = require('precss');
const autoprefixer = require('autoprefixer');
module.exports = ({ file, options, env }) => ({
    plugins: [precss, autoprefixer]
})
```
4. 打包并使用浏览器打开`./src/index.html`，可以看到，自动给我们添加了前缀。

    ![clipboard.png](/img/bVYdxU)

5. 其他更多配置请查阅[webpack关于`postcss-loder`章节][6]

### 0x008 资源
- [源代码][7]





  [1]: https://webpack.js.org/loaders/style-loader/
  [2]: https://webpack.js.org/loaders/style-loader/
  [3]: https://webpack.js.org/loaders/style-loader/
  [4]: https://webpack.js.org/loaders/style-loader/
  [5]: https://webpack.js.org/loaders/sass-loader/
  [6]: https://webpack.js.org/loaders/postcss-loader/
  [7]: https://github.com/followWinter/webpack-study/tree/master/0x011-styling-loader
