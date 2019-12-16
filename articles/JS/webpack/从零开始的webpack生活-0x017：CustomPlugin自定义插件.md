### 0x001 概述
上一章讲的是[其他一些常用的小插件][1]，这一章讲的是自定义插件。
### 0x002 环境配置
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
        filename: '[name].[chunkhash].js'
    }
;
```
### 0x003 栗子1-最简单的插件
1. 编写插件，这个插件会子安控制台输出一句`hello plugin`
    ```
    // ./CustomPlugin.js
    function UpdatePackageVersionPlugin(options) {
        console.log('hello plugin')
    }
    CustomPlugin.prototype.apply = function (compiler) {
    };
    
    module.exports = CustomPlugin;
    ```
2. 引用该插件
    ```
    const path                     = require('path');
    var CustomPlugin = require('./CustomPlugin')
    
    module.exports = {
        entry : {
            'index': ['./src/index.js'],
        },
        output: {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
    
        plugins: [
            new CustomPlugin({options:true})
        ]
    };
    ```
3. 打包并查看控制台
    ```
    $ webpack
    # ↓↓↓↓↓↓↓↓↓↓↓插件输出
    hello plugin
    # ↑↑↑↑↑↑↑↑↑↑↑插件输出
    The compiler is starting to compile...
    The compiler is starting a new compilation...
    The compilation is starting to optimize files...
    The compilation is going to emit files...
    Hash: 8daa4edb5544a8f81c35
    Version: webpack 3.8.1
    Time: 58ms
              Asset    Size  Chunks             Chunk Names
    index.bundle.js  2.6 kB       0  [emitted]  index
       [0] multi ./src/index.js 28 bytes {0} [built]
       [2] ./src/index.js 14 bytes {0} [built]

    ```
### 0x004 栗子2-偷偷添加资源
    1. 修改插件，这个插件会读取打包好的资源文件，并写入到`filelist.md`文件，保存到`dist`目录下。
    ```
    function CustomPlugin(options) {
        console.log('hello plugin')
    }
    
    CustomPlugin.prototype.apply = function (compiler) {
        compiler.plugin('emit', function (compilation, callback) {
            // Create a header string for the generated file:
            var filelist = 'In this build:\n\n';
    
            // Loop through all compiled assets,
            // adding a new line item for each filename.
            for (var filename in compilation.assets) {
                filelist += ('- '+ filename +'\n');
            }
    
            // Insert this list into the webpack build as a new file asset:
            compilation.assets['filelist.md'] = {
                source: function() {
                    return filelist;
                },
                size: function() {
                    return filelist.length;
                }
            };
    
            callback();
        })
    };
    
    module.exports = CustomPlugin;
    ```
2. 打包并查看文件
    ```
    $ webpack
    $ ls ./dist
    filelist.md
    index.bundle.js
    $ cat filelist.md
    // ./filelist.md
    In this build:

    - index.bundle.js

    ``` 
3. 更多配置请查阅[webpack关于自定义plugin章节][3]

### 0x005 资源
- [源代码][4]


  [1]: https://segmentfault.com/a/1190000011976221
  [2]: https://segmentfault.com/a/1190000011976221
  [3]: https://webpack.js.org/contribute/writing-a-plugin/
  [4]: https://github.com/followWinter/webpack-study/tree/master/0x017-custom-plugin
