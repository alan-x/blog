### 0x001 概述
上一章讲的是[分离样式][1]，这一章讲的是剩下的一些我常用的插件,和上一章是没有任何关系。
### 0x002 环境搭建
```
$ mkdir 0x0016-other-plugin
$ cd 0x0016-other-plugin
$ npm init -y
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
### 0x003 `EnvironmentPlugin`定义环境
1. 插件介绍
    这个插件用来定义环境变量的，直接定义在了`process.env`下。
2. 修改配置
    ```
    plugins: [
        new webpack.EnvironmentPlugin({
            NODE_ENV:'development',
            DEBUG:false
        })
    ]
    ```
3. 编写代码
    ```
    if (process.env.NODE_ENV==='production') {
        console.log('Welcome to production');
    }
    if (process.env.DEBUG) {
        console.log('Debugging output');
    }
    ```
4. 编译并查看结果
    ```
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    if (false) {
        console.log('Welcome to production');
    }
    if (false) {
        console.log('Debugging output');
    }
    
    /***/ })
    ```
5. 更多配置请查阅[webpack关于EnvironmentPlugin相关章节][2]
### 0x004 `CleanWebpackPlugin`清除文件夹
1. 插件介绍
    这个插件用来清除某个路径下的文件的，一般用来清理上次打包之后的残留文件。
2. 不使用插件打包文件
    这里我们修改一下`output.filename:`为`[name].[chunkhash].js'`，这样每次输出的文件就都不一样了
    - 打包代码
        ```
        $ webpack
        // ./dist
        index.dfa7ddd294976d60a25f.js
        ```
    - 修改代码
        ```
        // ./src/index.js
        if (process.env.NODE_ENV==='production') {
            console.log('Welcome to production');
        }
        if (process.env.DEBUG) {
            console.log('Debugging output');
        }
        console.log('clearwebpackplugin')
        ```
    - 再次打包
        ```
        $ webpack
        // ./dist
        index.69ed567b40b7234d1ea4.js
        index.dfa7ddd294976d60a25f.js
        ```
        可以看到，上次打包的文件依旧在，这不方面我们直接部署到线上，手动删除可不符合`webpack`的初衷，所以需要用到这个插件
3. 安装依赖
    ```
    $ cnpm install --save-dev clean-webpack-plugin
    ```
4. 修改依赖
    ```
    const path               = require('path');
    var webpack              = require('webpack')
    const CleanWebpackPlugin = require('clean-webpack-plugin')
    
    module.exports = {
        entry : {
            'index': ['./src/index.js'],
        },
        output: {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].[chunkhash].js'
        },
    
        plugins: [
            new webpack.EnvironmentPlugin({
                NODE_ENV: 'development',
                DEBUG   : false
            }),
            new CleanWebpackPlugin(path.resolve(__dirname, 'dist'))
        ]
    };
    ```
4. 打包编译
    ```
    $ webpack

    // ./dist
    index.69ed567b40b7234d1ea4.js
    ```
    啊世界清静了，以前的文件都没了!
5. 更多配置请查看[clean-webpack-plugin官方文档][3]
### 0x005 `copyWebpackPlugin`复制文件
1. 插件介绍
    用来直接复制文件的,比如资源文件，有一些文件我们希望他不打包到`js`中，但是又需要部署到生成环境下，为了方便部署，将它们和要部署的文件放在同一个文件夹下，方便部署。
2. 安装依赖
    ```
    $ cnpm install --save-dev copy-webpack-plugin
    ```
3. 添加资源
    ```
    $ mkdir ./asset
    $ cd ./asset
    $ vim resource.txt
    this is resource!
    ```
4. 修改配置
    ```
    const path               = require('path');
    const webpack              = require('webpack')
    const CleanWebpackPlugin = require('clean-webpack-plugin')
    const CopyWebpackPlugin=require('copy-webpack-plugin')
    module.exports = {
        entry : {
            'index': ['./src/index.js'],
        },
        output: {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].[chunkhash].js'
        },
    
        plugins: [
            new webpack.EnvironmentPlugin({
                NODE_ENV: 'development',
                DEBUG   : false
            }),
            new CleanWebpackPlugin(path.resolve(__dirname, 'dist')),
            new CopyWebpackPlugin([
                {
                    from:path.resolve(__dirname,'asset'),
                    to:path.resolve(__dirname,'dist/asset')
                }
            ])
        ]
    };
    ```
5. 打包
    ```
    $ webpack
    // ./dist
    asset
        -resource.txt
    index.69ed567b40b7234d1ea4.js
    ```
6. 其他更多配置请查阅[webpack关于CopyWebpackPlugin][4]

### 0x006 资源
- [源代码][5]


  [1]: https://segmentfault.com/a/1190000011884725
  [2]: https://webpack.js.org/plugins/environment-plugin/
  [3]: https://github.com/johnagan/clean-webpack-plugin
  [4]: https://webpack.js.org/plugins/copy-webpack-plugin/
  [5]: https://github.com/followWinter/webpack-study/tree/master/0x016-other-plugin
