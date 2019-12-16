### 0x001 概述
其实我不知道怎么写，所以决定就一块一块的写点平常配置的过程，根据不同东西稍微分区一下就好了

### 0x002 初始化项目结构
```
$ mkdir webpack_study
$ cd webpack_study
$ mkdir 0x001-begin
$ cd 0x001-begin
```

### 0x003 安装`webpack`，并检查环境
```
$ node -v
# 输出
v8.4.0
$ npm -v
# 输出
v5.4.2
$ cnpm -v
# 输出
cnpm@5.1.1 (/usr/local/lib/node_modules/cnpm/lib/parse_argv.js)
npm@5.4.1 (/usr/local/lib/node_modules/cnpm/node_modules/npm/lib/npm.js)
node@8.4.0 (/usr/local/bin/node)
npminstall@3.1.4 (/usr/local/lib/node_modules/cnpm/node_modules/npminstall/lib/index.js)
prefix=/usr/local 
darwin x64 16.7.0 
registry=http://registry.npm.taobao.org

$ cnpm install -g webpack
# 输出
Downloading webpack to /usr/local/lib/node_modules/webpack_tmp
...
[webpack@3.8.1] link /usr/local/bin/webpack@ -> /usr/local/lib/node_modules/webpack/bin/webpack.js

$ webpack -v
# 输出
3.8.1
```

### 0x004 第一个栗子-最简单的使用方式
> 这个栗子使用的是命令行打包形式，纯粹用来做测试而已，在正式项目中，我们的配置将会达到非常复杂的程度虽然命令行依旧可以实现，但是将会带来维护上的和形式上的麻烦，所以我们通常会采用配置文件的形式。

1. 编写一个`js`文件

    ```
    # 0x001-begin/src/index.js
    console.log('hello webpack')
    
    ```
2. 在`0x001-begin`文件夹下执行命令
    
    ```
    $ cd 0x001-begin
    $ webpack ./src/index.js ./dist/index.js
    # 控制台输出
    Hash: d25cd23c431cf01c6a5b
    Version: webpack 3.6.0
    Time: 53ms
       Asset    Size  Chunks             Chunk Names
    index.js  2.5 kB       0  [emitted]  main
       [0] ./src/index.js 29 bytes {0} [built]
    ```
3. 此时查看`0x001-begin`可以发现，多了个`dist`文件夹，文件夹下有一个`index.js`文件,这个文件就是我们通过`webpack`打包之后的文件，打开这个文件，我们可以发现，里面的代码都不是我们写的，这是`webpack`自动生成的，暂时不管，以后再去分析他，在文件末尾找一找便可以发现我们写的那句话也在里面。
    ```
    ...
    /***/ (function(module, exports) {
    console.log('hello webpack')
    /***/ })
    ...
    ```
4. 文件引用
    平常我们在做项目的时候，会将不同的功能、不同的类写在不同的文件，使用的时候互相引用，提升项目的可维护性和可扩展性。
    ```
    # 0x001-begin/user.js
    export default function (first_name, second_name) {
        console.log('hello ' + first_name + '-' + second_name)
    }
    # 0x001-begin/index.js
    let getUserName = require('./user')
    console.log('hello webpack')
    getUserName('hello','webpack')
    ```
    在`user.js`中我们编写了一个方法并`export`，然后在`index.js`中引入并调用，此时查看打包之后的`dist/index.js`文件可以发现所有的代码都打包进去了。
    ```
    ...
    /***/ (function(module, exports, __webpack_require__) {
    let getUserName = __webpack_require__(1)
    console.log('hello webpack')
    getUserName('hello','webpack')
    /***/ }),
    /* 1 */
    /***/ (function(module, __webpack_exports__, __webpack_require__) {
    "use strict";
    Object.defineProperty(__webpack_exports__, "__esModule", { value: true });
    /* harmony default export */ __webpack_exports__["default"] = (function (first_name, second_name) {
        console.log('hello ' + first_name + '-' + second_name)
    });
    /***/ })
    /******/ ]);
    ...
    ```
5. 命令格式说明
    ```
    webpack <entry> [<entry>] <output>
    - entry: 入口文件，可以一个也可以多个
    - output: 输出文件，可以是
    ```
    注意：
    - `entry`可以有多个，但是一个和多个的写法不一样,必须以`entryname=filename`的形式指定。同时不能直接单纯的指定输出的文件名称，比如`./dist/index.js`，将会报错，可以换成以下方式指定，或者其他类似方式。
    ```
    webpack index1=./src/index.js index2=./src/index2.js ./dist/[name].js
    ```
    - `entry`文件名为`index.js`的时候可以省略，将会自动在指定文件夹下寻找该文件。

### 0x005-使用配置文件
> 现在开始使用配置文件的形式来配置`webpack`吧！
1. 最简单的配置文件

    ```
    # 0x001-begin/webpack.config.js
    var path       = require('path')
    module.exports = {
        entry : './src/index.js',
        output: {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        }
    }
    ```
2. 执行命令
    ```
    $ webpack
    # 输出
    Hash: 731e5b2dd5c8e29150e0
    Version: webpack 3.6.0
    Time: 59ms
       Asset     Size  Chunks             Chunk Names
    index.js  2.94 kB       0  [emitted]  main
       [0] ./src/index.js 96 bytes {0} [built]
       [1] ./src/user.js 112 bytes {0} [built]

    ```
    这里实现了我们之前使用的命令`webpack ./src/index.js ./dist/index.js`的功能，webpack命令自动读取文件夹下名为`webpack.config.js`的配置文件。
    注意：
    - 配置文件的名称不一定是`webpack.config.js`，如果不是该名称，需要显式的指定配置文件名称：`webpack --config <path/to/config/file>`
    - `output`的`path`必须为绝对目录，所以这里使用了`nodejs`的`path`模块。


3.监听文件变化
使用 `webpack -w` 可以监听入口文件变化，当然包括入口文件引入的所有文件的变化，可以利用该特性做到开发网页的时候的浏览器自动刷新
```
$ webpack -w
# 输出
Webpack is watching the files…

Hash: 731e5b2dd5c8e29150e0
Version: webpack 3.6.0
Time: 64ms
   Asset     Size  Chunks             Chunk Names
index.js  2.94 kB       0  [emitted]  main
   [0] ./src/index.js 96 bytes {0} [built]
   [2] ./src/user.js 112 bytes {0} [built]


```
注意：
- 此时webpack执行完并不会退出，而是一直执行并监听文件改变，当文件发生改变，将会触发再次打包，同时控制台将将会输出重新打包之后的信息，除非你在控制台按了`CTRL+C`
```
$ webpack -w
# 输出

Webpack is watching the files…

Hash: 7ec0e3e1cfaf0227c38b
Version: webpack 3.6.0
Time: 66ms
   Asset     Size  Chunks             Chunk Names
index.js  2.94 kB       0  [emitted]  main
   [0] ./src/index.js 96 bytes {0} [built]
   [3] ./src/user.js 113 bytes {0} [built]
Hash: 731e5b2dd5c8e29150e0
Version: webpack 3.6.0
Time: 9ms
   Asset     Size  Chunks             Chunk Names
index.js  2.94 kB       0  [emitted]  main
   [1] ./src/user.js 112 bytes {0} [built]
    + 1 hidden module
^C
```
- 修改`webpack`配置文件之后需要手动重启`webpack`，新的配置才会生效。
- 如果打包过程出现错误，比如语法错误，将会在控制台以红色文字显示，并且在你修复之后会再次打包。

![clipboard.png](/img/bVXW1f)
### 0x006 详解`entry`
entry可以取一下类型的值：
- string：'index.js'
- array：['index1.js','index2.js']
- object：{index:'index1.js',index2:'index2.js'}
- function：function(){return 'index1.js'}
具体配置可以查看[webpack关于entry的章节][4]
### 0x007 详解`output`
当只有一个`entry`时，可以直接指定`output.filename`，但是有多个`entry`时不能直接指定，否则将会报错
```
ERROR in chunk *** [entry]
***
Conflict: Multiple assets emit to the same filename ***
```
需要指定其他方式
- [name].js：使用`entry`名字
- [id].js：使用`chunk id`
- [hash].js：使用哈希
- [chunkhash].js：使用生成的文件的哈希（推荐）
当然，以上可以结合来用，推荐使用`[name].[chunkhash].js`，既能知道是哪个`entry`，也能让文件没有修改时候保持文件名不变，让用户在网站更新后访问时不需要重新获取该文件，直接从缓存读取文件
还有许多其他的配置，暂时使用不到，具体配置可以查看[webpack关于output的章节][5]

### 0x008 `webpack`指令说明
- `webpack`：根据`webpack.config.js`打包，如果没有该文件将会报错
- `webpack --config <path/to/config/file>`：根据指定的配置文件打包，如果没有该文件将报错
- `webpack -w`：根据指定默认配置文件打包，并监听文件变化，在文件变化后自动打包
- `webpack -p`：打包的时候对`js`混淆压缩
其他更多指令说明，查看[webpack关于CLI的章节][6]
### 0x009 资源
- [源代码][7]

### 0x010 文章整理
- [从零开始的webpack生活-0x001：webpack初次相逢][8]
- [从零开始的webpack生活-0x002：devServer自动刷新][9]
- [从零开始的webpack生活-0x003：html模板生成][10]
- [从零开始的webpack生活-0x004：js压缩混淆][11]
- [从零开始的webpack生活-0x005：DefinePlugin奇妙用处][12]
- [从零开始的webpack生活-0x006：providerPlugin全局定义][13]
- [从零开始的webpack生活-0x007：CommonsChunkPlugin基本用法][14]
- [从零开始的webpack生活-0x008：DLL加速编译][15]
- [从零开始的webpack生活-0x009：FilesLoader装载文件][17]
- [从零开始的webpack生活-0x010：TemplatingLoader装载模板][18]
- [从零开始的webpack生活-0x011：StylingLoader装载样式][19]
- [从零开始的webpack生活-0x012：TranspilingLoader装载脚本][20]
- [从零开始的webpack生活-0x013：LintingLoader格式校验][21]
- [从零开始的webpack生活-0x014：CustomLoader自定义loader][22]
- [从零开始的webpack生活-0x015：ExtractTextWebpackPlugin分离样式][16]
- [从零开始的webpack生活-0x016：OtherPlugin其他常用][23]
- [从零开始的webpack生活-0x017：CustomPlugin自定义插件][24]


  [1]: https://github.com/followWinter/webpack-study/tree/master/0x001-begin
  [2]: https://github.com/followWinter/webpack-study/tree/master/0x001-begin
  [3]: https://github.com/followWinter/webpack-study/tree/master/0x001-begin
  [4]: https://webpack.js.org/configuration/entry-context/
  [5]: https://webpack.js.org/configuration/output/
  [6]: https://webpack.js.org/api/cli/
  [7]: https://github.com/followWinter/webpack-study/tree/master/0x001-begin
  [8]: https://segmentfault.com/a/1190000011865966
  [9]: https://segmentfault.com/a/1190000011867390
  [10]: https://segmentfault.com/a/1190000011868449
  [11]: https://segmentfault.com/a/1190000011868498
  [12]: https://segmentfault.com/a/1190000011882514
  [13]: https://segmentfault.com/a/1190000011882883
  [14]: https://segmentfault.com/a/1190000011883352
  [15]: https://segmentfault.com/a/1190000011884606
  [16]: https://segmentfault.com/a/1190000011884725
  [17]: https://segmentfault.com/a/1190000011926527
  [18]: https://segmentfault.com/a/1190000011926542
  [19]: https://segmentfault.com/a/1190000011929459
  [20]: https://segmentfault.com/a/1190000011930547
  [21]: https://segmentfault.com/a/1190000011931364
  [22]: %E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E7%9A%84webpack%E7%94%9F%E6%B4%BB-0x014%EF%BC%9ACustomLoader%E8%87%AA%E5%AE%9A%E4%B9%89loader
  [23]: https://segmentfault.com/a/1190000011976221
  [24]: https://segmentfault.com/a/1190000011976249
