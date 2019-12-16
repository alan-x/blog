### 0x001 概述
上一章我们讲了[eslint-loader的配置][1]，常用类型的常用`loader`已经都讲完了，大体上其他的都大同小异，需要去各大`loader`的官方查阅用户手册就可以了。接下来将`loader`的本质和自定义`loader`。

### 0x002 环境配置
这一次需要两个项目，一个项目是`custome-loader`，实现了`custome-loader`，一个是`user`，使用了`custom-loader`。
```
$ mkdir 0x0014-custome-loader
$ cd 0x0014-custome-loader
```

### 0x003 `custome-loader`实现
1. `custome-loader`项目配置
    ```
    $ mkdir custome-loader
    $ cd custome-loader
    $ npm init -y
    ```
2. 实现`custome-loader`
    ```
    $ vim ./index.js
    /**
     * 实现一个函数
     * 作用是将`var`替换成`let`
     * @param content 字符串
     * @returns string 字符串 */
        module.exports = function (content) {
            console.log(content)
            content = content.replace(/var/g, 'let')
            console.log(content)
            return content;
             };
        ```
3. 测试
    ```
    $ vim ./test/index.js
    // ./test/index.js
    var custom = require('./../index')
    custom('var name="张三"')
    
    $ node  ./test/index.js
    元数据 var name="张三"
    处理之后 let name="张三"
    ```
    
### 0x004 `custome-loader`使用
1. 配置
    ```
    $ mkdir user
    $ cd user
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
2. 本地安装依赖
    ```
    $ npm install --sve-dev /MY_PROJECT/PROJECT_OWN/webpack_study/0x014-custom-loader/custom-loader
    # 输出
    pm WARN user@1.0.0 No description
    npm WARN user@1.0.0 No repository field.
    +custom-loader@1.0.6
    updated 1 package in 0.3s

    ```
3. 修改配置
    ```
    const path     = require('path');
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
                    test  : /\.js$/,
                    loader: "custom-loader",
                }
            ]
        }
    }
    ;

    ```
4. 打包
    ```
    $ webpack
    # 输出
    元数据 var name="张三"
    var age=14
    
    处理之后 let name="张三"
    let age=14
    
    Hash: 4040662a699dbc91f8dd
    Version: webpack 3.8.1
    Time: 68ms
              Asset     Size  Chunks             Chunk Names
    index.bundle.js  2.62 kB       0  [emitted]  index
       [0] multi ./src/index.js 28 bytes {0} [built]
       [2] ./src/index.js 25 bytes {0} [built]

    ```

5. 查看打包结果
    ```
    /* 1 */
    /***/ (function(module, exports) {
    
    let name="张三"
    let age=14
    
    
    /***/ })
    /******/ ]);
    ```
    可以看到，webpack在打包的时候，将`./src/index.js`的内容作为我们定义在`custome-loader`中的方法的参数，同时执行该方法，将处理后的返回值作为结果输出到`./dist/index.bunle.js`.
### 0x005 更多配置
这里只是实现了一个最简单的`loader`，而`webpack`官方还提供了许多其他的API来实现功能更加强大的`loader`，凭借这些API我们也可以写出自己的`file-loader`、`json-loader`之类的。当然没有必要去真的重复制造轮子，只是为了掌握这种造轮子的技术，对理解整个`webpack`工程，对理解`web`功能提供更多思路而已。
### 0x006 资源
- [源代码][3]


  [1]: https://segmentfault.com/a/1190000011931364
  [2]: https://segmentfault.com/a/1190000011931364
  [3]: https://github.com/followWinter/webpack-study/tree/master/0x014-custom-loader
