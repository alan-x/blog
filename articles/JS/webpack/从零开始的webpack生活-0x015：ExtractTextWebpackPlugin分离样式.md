### 0x001 概述
[上一章][1]讲的是`Dll`，这一章讲的是`ExtractTextWebpackPlugin`,依旧是没有任何关系。

### 0x002 插件介绍
这个插件用来将`css`导出到单独文件

### 0x003 栗子-不使用该插件的情况
这次涉及到`loader`的使用，可以暂时忽略这方面配置
1. 初始化项目
    ```
    $ mkdir 0x008-extract-text-webpack-plugin
    $ cd 0x008-extract-text-webpack-plugin
    $ cnpm init -y
    $ cnpm install webpack css-loader style-loader extract-text-webpack-plugin --save-dev
    ```
2. 配置`webpack`，先不使用插件
    ```
    $ vim webpack.config.js
    // webpack.config.js
    const path    = require('path');
    const webpack = require('webpack');
    
    module.exports = {
        entry  : {
            'index': ['./src/index.js']
        },
        output : {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
        module : {
            rules: [
                {
                    test: /\.css$/,
                    use:[
                        { loader: "style-loader" },
                        { loader: "css-loader" }
                    ]
                }
            ]
        }
    };
    ```
3. 添加入口文件和样式文件
    ```
    $ vim ./src/index.js
    // ./src/index.js
    require('./../style2.css')
    
    $ vim style1.css
    p{
        color: red;
    }
    $ vim style2.css
    p{
        font-size: 30px;
    }
    ```
4. 打包并查看打包结果`index.bundle.js`，可以发现，`css`被打包到了`js`里面，以字符串的形式存在，并且整个`index.bundle.js`比平常打了不少。
    ```
    // module
    exports.push([module.i, "p{\n    color: red;\n}", ""]);
    
    // exports
    ```
    此时如果有`html`引用该`js`
    ```
    $ vim ./src/index.html
    
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <script src="./../dist/index.bundle.js"></script>
    </head>
    <body>
    <p>text</p>
    </body>
    </html>
   
    ```
5 浏览器打开`index.html`，就会发现`css`以`style`的形式被插到了`head`
    
![clipboard.png](/img/bVX1UE)

### 0x004 使用该插件
1. 修改配置
    ```
    const path    = require('path');
    const webpack = require('webpack');
    const ExtractTextPlugin = require("extract-text-webpack-plugin");
    
    module.exports = {
        entry  : {
            'index': ['./src/index.js']
        },
        output : {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
        module : {
            rules: [
                {
                    test: /\.css$/,
                    // use:[
                    //     { loader: "style-loader" },
                    //     { loader: "css-loader" }
                    // ]
                    use : ExtractTextPlugin.extract({
                        fallback: "style-loader",
                        use     : "css-loader"
                    })
                }
            ]
        },
        plugins: [
            new ExtractTextPlugin("styles.css"),
        ]
    };
    ```
2. 打包并查看`dist`，可以发现，`index.bundle.js`文件恢复了正常，并且多出来一个`style.css`文件。
    ```
    $ webpack
    // ./dist/style.css
    p{
        color: red;
    }p{
        font-size: 30px;
    }
    ```
### 0x005 配合`HtmlWebpackPlugin`插件自动插入`index.html`
1. 修改配置
    ```
    const path              = require('path');
    const webpack           = require('webpack');
    const ExtractTextPlugin = require("extract-text-webpack-plugin");
    var HtmlWebpackPlugin   = require('html-webpack-plugin');
    
    module.exports = {
        entry  : {
            'index': ['./src/index.js']
        },
        output : {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
        module : {
            rules: [
                {
                    test: /\.css$/,
                    // use:[
                    //     { loader: "style-loader" },
                    //     { loader: "css-loader" }
                    // ]
                    use : ExtractTextPlugin.extract({
                        fallback: "style-loader",
                        use     : "css-loader"
                    })
                }
            ]
        },
        plugins: [
            new ExtractTextPlugin("styles.css"),
            new HtmlWebpackPlugin({
                title   : 'extract-text-webpack-plugin',
                filename: 'index.html',
                template: path.resolve(__dirname, 'src/index.html'),
                inject  : 'head'
            })
        ]
    };
    ```
2. 打包并查看`./dist/index.html`,它以`link`的形式被插入头部
    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <link href="styles.css" rel="stylesheet">
        <script type="text/javascript" src="index.bundle.js"></script>
    </head>
    <body><p>text</p></body>
    </html>
    ```

### 0x006 多入口文件打包
1. 添加入口
    ```
    entry  : {
            'index': ['./src/index.js'],
            'index2': ['./src/index2.js']
        }
        
    ```
2. 修改插件命名方式
    ```
    new ExtractTextPlugin("[name].css"),
    ```
3. 打包并查看`dist`
    ```
    $ ls ./dist
    index.bundle.js
    index.css
    index.html
    index2.bundle.js
    index2.css
    ```
4. 更多用法，请查阅[webpack关于ExtractTextWebpapckPlugin章节][2]

### 0x007 资源
- [源代码][3]


  [1]: https://segmentfault.com/a/1190000011884606
  [2]: https://webpack.js.org/plugins/extract-text-webpack-plugin/
  [3]: https://github.com/followWinter/webpack-study/tree/master/0x009-extract-text-webpack-plugin
