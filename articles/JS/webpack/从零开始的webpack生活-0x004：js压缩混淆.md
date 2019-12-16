### 0x001 概述
[上一章][1]讲了关于生成模板`html`的，并且最后将`html`压缩，这一章讲的是`js`压缩混淆

### 0x002 配置环境
1. 初始化项目
    ```
    $ npm init -y
    $ mkdir src
    $ mkdir src/index.js
    ```
2. 新建`webpack.config.js`
    ```
    var path       = require('path')
    module.exports = {
        entry : path.resolve(__dirname, 'index.js'),
        output: {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        }
    }
    
    ```
3. 修改配置
    - 安装依赖
        ```
        npm i -D uglifyjs-webpack-plugin
        ```
    - 修改`./src/index.js`
        ```
        document.write('hello webpack')
        ```
    - 初始化
        ```
        var UglifyJSPlugin = require('uglifyjs-webpack-plugin')

        ```
    - 添加插件
        ```
        plugins: [
            new UglifyJSPlugin()
        ]
        ```
    - 最终配置文件
        ```
        var path           = require('path')
        var UglifyJSPlugin = require('uglifyjs-webpack-plugin')
        
        module.exports = {
            entry  : path.resolve(__dirname, 'index.js'),
            output : {
                path    : path.resolve(__dirname, 'dist'),
                filename: 'index.min.js'
            },
            plugins: [
                new UglifyJSPlugin()
            ]
        }

        ```
    - 打包
    ```
    $ webpack
    
    // ./dist/index.min.js
    !function(e){function t(r){if(n[r])return n[r].exports;var o=n[r]={i:r,l:!1,exports:{}};return e[r].call(o.exports,o,o.exports,t),o.l=!0,o.exports}var n={};t.m=e,t.c=n,t.d=function(e,n,r){t.o(e,n)||Object.defineProperty(e,n,{configurable:!1,enumerable:!0,get:r})},t.n=function(e){var n=e&&e.__esModule?function(){return e.default}:function(){return e};return t.d(n,"a",n),n},t.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},t.p="",t(t.s=0)}([function(e,t){document.write("hello webpack")}]);
    ```
    
### 0x003 配置
1. 匹配上的文件才压缩
    
    - 添加`index2.js`
        ```
        document.write('hello webpack2')
        ```
    - 修改`entry`、`output`、`plugins`
        ```
        var path           = require('path')
        var UglifyJSPlugin = require('uglifyjs-webpack-plugin')
        
        module.exports = {
            entry  : {
                index : path.resolve(__dirname, 'src/index.js'),
                index2: path.resolve(__dirname, 'src/index2.js')
            },
            output : {
                path    : path.resolve(__dirname, 'dist'),
                filename: '[name].min.js'
            },
            plugins: [
                new UglifyJSPlugin({
                    test: /index2/i
                })
            ]
        }
    
        ```
    - 打包
        ```
        `index.min.js`未被压缩
        `index2.min.js`压缩了
        ```
    - 取值
        - `RegExp|Array<RegExp>`：正则匹配或者正则匹配数组
2. 需要压缩的文件才压缩(测试失败了？日后再详细测试)
    - 添加`index3.js`
        ```
        document.write('hello webpack3')
        ```
    - 修改`entry`、`output`、`plugins`
        ```
        var path           = require('path')
        var UglifyJSPlugin = require('uglifyjs-webpack-plugin')
        
        module.exports = {
            entry  : {
                index : path.resolve(__dirname, 'src/index.js'),
                index2: path.resolve(__dirname, 'src/index2.js'),
                index3: path.resolve(__dirname, 'src/index3.js')
            },
            output : {
                path    : path.resolve(__dirname, 'dist'),
                filename: '[name].min.js'
            },
            plugins: [
                new UglifyJSPlugin({
                    test: /index2/i,
                    include: /index3/i
                })
            ]
        }
    
        ```
    - 打包
        ```
        `index.min.js`未被压缩
        `index2.min.js`压缩了
        `index3.min.js`压缩了
        ```
    - 取值
        - `RegExp|Array<RegExp>`：正则匹配或者正则匹配数组
3. 排除某些文件(测试失败了？日后再详细测试)
    - 添加`index4.js`
        ```
        document.write('hello webpack4')
        ```
    - 修改`entry`、`output`、`plugins`
        ```
        var path           = require('path')
        var UglifyJSPlugin = require('uglifyjs-webpack-plugin')
        
        module.exports = {
            entry  : {
                index : path.resolve(__dirname, 'src/index.js'),
                index2: path.resolve(__dirname, 'src/index2.js'),
                index3: path.resolve(__dirname, 'src/index3.js'),
                index4: path.resolve(__dirname, 'src/index4.js')
            },
            output : {
                path    : path.resolve(__dirname, 'dist'),
                filename: '[name].min.js'
            },
            plugins: [
                new UglifyJSPlugin({
                    test: /index2/i,
                    include: /index3/i
                })
            ]
        }
    
        ```
    - 打包
        ```
        `index.min.js`未被压缩
        `index2.min.js`压缩了
        `index3.min.js`压缩了
        `index4.min.js`未被压缩
        ```
    - 取值
        - `RegExp|Array<RegExp>`：正则匹配或者正则匹配数组
4. 生成sourceMap
    - 修改配置
        ```
         new UglifyJSPlugin({
            test         : /\.js($|\?)/i,
            include      : /index3/,
            exclude      : /index4/,
            sourceMap    : true,
        })
        ```
     - 打包
5. 自定义压缩细节
    这里可以配置的项非常多，包括是否兼容ie8、是否支持es5、6、7、8等等，还可以配置压缩的各种细节，包括是否保留注释之类的，其实使用默认就差不多了，需要的时候再去配置细节，比如anuglar中如果压缩了变量名可能导致找不到注入的服务
    ```
    new UglifyJSPlugin({
            test         : /\.js($|\?)/i,
            include      : /index3/,
            exclude      : /index4/,
            sourceMap    : true,
            uglifyOptions: {
                ie8     : true,
                ecma    : 6,
                mangle  : true,
                compress: true,
                warnings: false
            }
        })
    ```
6. 更多细节配置，参考[webpack关于UglifyJSPlugin章节][2]

0x004 资源
- [源代码][3]


  [1]: https://segmentfault.com/a/1190000011868449
  [2]: https://webpack.js.org/plugins/uglifyjs-webpack-plugin/
  [3]: https://github.com/followWinter/webpack-study/tree/master/0x004-uglifyjs-webpack-plugin
