### 0x001 概述
[上一章][1]讲的是`providerPlugin`，和这一章依旧没有丝毫关系，这一章讲的是`CommonsChunkPlugin`,说实在的，这个插件略复杂，我还没完全搞懂，大概是还没到那么深我就已经选择其他解决方案了吧，所以这里只讲一些基本用法。

### 0x002 插件介绍
这个插件就是用来提取公共代码的，可以这么说，如果一个方法被两个入口文件调用，那么这个方法将被打包到这两个文件中，会形成代码冗余，这个插件可以将这个方法提取出来，放到第三个文件中，通过在引入入口文件之前引入第三个文件，就可以避免冗余的代码打包

### 0x003 栗子
为了更深刻理解，需要举个栗子
1. 搭建环境
    ```
    + 0x007-common-chunk-plugin
      + src
      - webpack.config.js
    ```
2. 安装依赖
    ```
    $ npm init -y 
    $ npm install --save-dev webpack
    ```
3. 修改配置
    ```
    var path       = require('path')
    module.exports = {
        entry  : path.resolve(__dirname, 'src/index.js'),
        output : {
            path    : path.resolve(__dirname, 'dist'),
            filename: '[name].js'
        }
    }
    ```
4. 添加其他入口文件
    ```
    entry  : {
        index1: path.resolve(__dirname, 'src/index.js'),
        index2: path.resolve(__dirname, 'src/index2.js')
    },
    // ./src/index.js
    var timestamp = require('./utils')
    timestamp()
    // ./src/index2.js
    var timestamp = require('./utils')
    timestamp()
    ```
5. 添加工具类`./src/utils.js`
    ```
    export default function () {
        console.log(new Date().getTime())
    }
    ```
6. 打包并查看文件
    ```
    // ./dist.index1.js
    ...
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
            /* harmony default export */ __webpack_exports__["default"] = (function () {
                console.log(new Date().getTime())
            });
    
    /***/ }),
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var timestamp = __webpack_require__(0)
    timestamp()
    ...
    ```
    ```
    // ./dist/index2.js
    ...
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony default export */ __webpack_exports__["default"] = (function () {
        console.log(new Date().getTime())
    });
    
    /***/ }),
    /* 1 */,
    /* 2 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var timestamp = __webpack_require__(0)
    timestamp()
    ...
    ```
    可以看到，同样的的`utils`分别被打包到了`index1.js`和`index2.js`，存在代码冗余。
### 0x003 配置`commonChunkPlugin`
1. 修改配置文件
    ```
    var path       = require('path')
    var webpack    = require('webpack')
    module.exports = {
        entry  : {
            index1: path.resolve(__dirname, 'src/index.js'),
            index2: path.resolve(__dirname, 'src/index2.js')
        },
        output : {
            path    : path.resolve(__dirname, 'dist'),
            filename: '[name].js'
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: "vendor"
            })
        ]
    }
    ```
2. 再次打包，可以发现，多了一个`vendor.js`，里面是`utils.js`的方法，查看`index1.js`和`index2.js`可以发现，原先重复的地方没了，因此我们就可以通过引入`vendor.js`、`index1.js`和`vendor.js`、`index2.js`来达到对公共代码的提取和分离。
    ```
    // ./dist/vendor.js
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony default export */ __webpack_exports__["default"] = (function () {
        console.log(new Date().getTime())
    });
    ```
    ```
    // ./dist/index1.js
    webpackJsonp([1],[
    /* 0 */,
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var timestamp = __webpack_require__(0)
    timestamp()
    
    /***/ })
    ],[1]);
    ```
    ```
    // ./dist/index2.js
    webpackJsonp([0],{

    /***/ 2:
    /***/ (function(module, exports, __webpack_require__) {
    
    var timestamp = __webpack_require__(0)
    timestamp()
    
    /***/ })
    
    },[2]);
    ```
### 0x004 直接打包几个包
    ```
    new webpack.optimize.CommonsChunkPlugin({
                name     : ["jquery", "lodash"],
                minChunks: Infinity,
            })
    ```
    当然还有许多更加复杂的用法，还请看[webpack关于commonChunkPlugin章节][2]

### 0x005 资源
    - [源代码][3]
    
    
    


  [1]: https://segmentfault.com/a/1190000011882883
  [2]: https://webpack.js.org/plugins/commons-chunk-plugin/
  [3]: https://github.com/followWinter/webpack-study/tree/master/0x007-common-chunk-plugin
