### 0x001 概述
[上一章][1]讲的是`CommonChunkPlugin`，和这一章依旧没有丝毫关系，这一章讲的是`DllPlugin`和`DllReferencePlugin`。

### 0x002 插件介绍
这个插件啊，用来预打包一些第三方库，因为他们不经常修改，而每次我们引用他们之后都要将他们不断的打包一次又一次，不但浪费了调试编译的时间，还浪费了....时间。

### 0x003 栗子又来了
1. 初始化项目
    ```
    $ midkir 0x007-dll
    $ cd 0x007-dll
    $ cnpm init -y
    $ cnpm install angular lodash jquery
    ```
2. 这次需要两个配置文件，一个用于打包`dll`，一个用于打包`dll-user`，先配置用来打包`dll`的`webpack.dll.config.js`
    ```
    $ vim ./webpack.dll.config.js
    // ./webpack.dll.config.js
    const path = require('path');
    const webpack = require('webpack');
    
    module.exports = {
        entry: {
            vendor: ['angular', 'jquery','lodash']// 用这三个库打包成dll做测试
        },
        output: {
            path: path.join(__dirname, 'dist'),
            filename: '[name].dll.js',
            library: '[name]_library'
        },
        plugins: [
            new webpack.DllPlugin({
                path: path.join(__dirname, 'dist', '[name]-manifest.json'),
                name: '[name]_library' // 需要与output.library相同
            })
        ]
    };
    ```
3. 打包dll，将会在`./dist`目录下生成`vendor-minifest.json`、`vendor.dll.js`
   ```
   $ webpack --config webpack.dll.config.js 
   ```

4. 配置`dll-user`
    ```
    $ vim ./webpack.config.js
    # ./webpack.config.js
    const path = require('path');
    const webpack = require('webpack');
    module.exports = {
        entry: {
            'dll-user': ['./src/index.js']
        },
        output: {
            path: path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
    
        plugins: [
            new webpack.DllReferencePlugin({
                context: __dirname,
                manifest: require('./dist/vendor-manifest.json')
            })
        ]
    
    };

    ```
5. 添加入口文件
    ```
    $ vim ./src/index.js
    // ./src/index.js
    var angular = require('angular');
    console.log(angular);
    ```
6. 打包`dll-user`
    ```
    $ webpack 
    Hash: 1aa3feec9d1827de7d5a
    Version: webpack 3.8.1
    Time: 70ms
                 Asset     Size  Chunks             Chunk Names
    dll-user.bundle.js  2.88 kB       0  [emitted]  dll-user
       [0] multi ./src/index.js 28 bytes {0} [built]
       [2] ./src/index.js 56 bytes {0} [built]
       // 注意这行 ↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓↓
       [3] delegated ./node_modules/_angular@1.6.6@angular/index.js from dll-reference vendor_library 42 bytes {0} [built]
        + 1 hidden module
    ```
    **注意**：这里我们引用了`angular`但是在打包完的`index.js`中，并不存在`angular`，因为我们通过引用`dll`来引入`angular`，在打包的信息输出中，对于angular的处理也变成了`delegated`,
    
7. 更多详细信息，请查看[webpack关于DllPlugin章节][4]

### 0x004 资源
- [源代码][5]


  [1]: https://segmentfault.com/a/1190000011883352
  [2]: https://segmentfault.com/a/1190000011883352
  [3]: https://segmentfault.com/a/1190000011883352
  [4]: https://webpack.js.org/plugins/dll-plugin/
  [5]: https://github.com/followWinter/webpack-study/tree/master/0x008-dll
