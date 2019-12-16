### 0x001 概述
上一章讲的是[DLL加速编译][1]，这一章开始讲`loader`相关的部分，但是关于`plugin`的部分善未完结，因为即将要讲的`ExtractTextWebpackPlugin`用到了一些`loader`，所以觉得先讲一下`loder`比较好。

### 0x002 `loader`介绍
我比较喜欢装载器这个翻译，`loder`说白了就是对各种文件的转化而已，比如`json-loader`可以将loader文件中的内容转化为`js`对象，也就是可以模拟为读取`json`文件然后做`JSAON.parse`而已，而对于其他的装载器则也类似，是对不同文件的不同处理方式，只是他们保持了相同的接口形式。就像一个加工的机器，有一个入口和一个出口，入口放猪肉，出来猪肉制品，入口放鸡肉，出来鸡肉制品，里面如何实现或许不一样，但是操作方式基本一致。

### 0x003 栗子1-`raw-loader`：读取文件，并封装成模块，导出唯一的内容为文件内容的字符串
1. 初始化项目
    ```
    $ mkdir 0x009-loader
    $ cd 0x009-loader
    $ npm init -y
    $ cnpm install --save-dev webpack raw-loader
    ```
2. 添加配置
    ```
    const path              = require('path');
    
    module.exports = {
        entry  : {
            'index': ['./src/index.js'],
        },
        output : {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        }
    };
    ```
3. 添加示例文件
    ```
    // ./srcindex.txt
    hello loader
    
    // ./src/index.j
    var content = require('raw-loader!./index.txt')
    console.log(content)
    ```
4. 打包并查看结果
    ```
    $ webpack
    // ./dist/index.js
    /***/ }),
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    var content = __webpack_require__(2)
    
    console.log(content)
    
    
    /***/ }),
    /* 2 */
    /***/ (function(module, exports) {
    
    module.exports = "hello loader"
    
    /***/ })
    /******/ ]);
    ```
    可以看到，文件的内容被以一个模块的形式导出，而在引入的文件中，变得不再是引入一个文件，而是一个模块。
 
5. 不在`require`中使用`loader`，因为麻烦且不美观，我们可以把它迁移到`webpack.conf.js`中。
    - 修改配置文件
        ```
        // ./wenpack.conf.js
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
                        test: /\.txt$/,
                        use : 'raw-loader'
                    }
                ]
            }
        };
        ```
        - `test`：匹配的文件名，这里匹配后缀为`.txt`的，只要`require`了该文件名结尾的文件，都将使用这个`raw-loader`来处理
        - `use`：命中后使用的加载器
    
    - 查看结果，和之前一致，推荐使用`wenpack`配置文件形式，可以保持引入文件格式的一致性。有利于维护和美观
6. 更多配置，可以查阅[webpack关于raw-loader部分][2]。

### 0x004 栗子2-`json-loader`：将`json`文件转化成`js`对象
1. 安装依赖
```
$ cnpm install --save-dev json-loader
```
2. 添加`rule`配置
    ```
    {
        test: /\.json$/,
        use : loader : 'json-loader'
    }
    ```
3. 引用
    ```
    //./src/index.json
    {
      "name": "张三",
      "age": "21"
    }
    // ./src/index.js
    require('./index.json')
    ```
4. 打包并查看结果
    ```
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    __webpack_require__(2)
    
    /***/ }),
    /* 2 */
    /***/ (function(module, exports) {
    
    module.exports = {"name":"张三","age":"21"}
    
    /***/ })
    /******/ ]);
    ```
    可见，`json-loder`将文件内的`json`字符串转化成了js对象，相当于使用`raw-loader`获取文件内容字符串，再调用`JSON.parse`，然后封装成模块并导出。
    
### 0x005 栗子3-`file-loader`：导出文件并返回文件的`URL`
1. 安装依赖包
```
$ cnpm install --save-dev file-loader
```
2. 添加`rule`配置
    ```
    {
                test: /\.(png|jpg|gif)$/,
                use : [
                    {
                        loader : 'file-loader',
                        options: {}
                    }
                ]
            }
    ```
3. 引用
    ```
    // ./src/index.js
    var funny = require('./../res/funny.png')
    ```
4. 打包并查看结果
    ```
    $ ls ./dist
    4e4e36593ce458606ffd851043749eec.png
    index.bundle.js
    ```
    ```
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    // var content = require('raw-loader!./index.txt')
    // var content = require('./index.txt')
    
    var funny = __webpack_require__(2)
    
    
    /***/ }),
    /* 2 */
    /***/ (function(module, exports, __webpack_require__) {
    
    module.exports = __webpack_require__.p + "4e4e36593ce458606ffd851043749eec.png";
    
    /***/ })
    /******/ ]);
    ```
    可以看出，文件被导出到了`dist`，并且将文件的路径封装成了模块并导出。
5. `option`其他配置
    - `name`：名字
        - `[path]`：文件路径
        - `[name]`：文件名称
        - `[hash]`：文件`hash`
        - `[ext]`：文件后缀
        - 以上变量可以随机组合，形成文件名,推荐：`[name].[hash].[ext]`
6. 注意：每引入一个文件，就会生成一个模块，即便该文件只是文件名不同，但是内容相同
7. 更多配置，可以[查阅webpack关于file-loader部分][3]。
    
### 0x006 栗子4-`url-loader`：根据文件大小类型判断是否`DATAURL`
1. 删除`file-loader`，添加配置
    ```
{
                test: /\.(png|jpg|gif)$/,
                use : [
                    {
                        loader : 'url-loader',
                        options: {
                            limit   : 1048576, // 低于1m， 这里的单位是Byte
                            mimetype: 'image/jpg', // 类型是`jpg`
                            name    : '[name].[hash].[ext]',
                            fallback: 'file-loader' //否则使用`file-loader`
                        }
                    }
                ]
            }
    ```
2. 引入一张高于1048576的jpg图片和1张低于81920的jpg图片，还有一张png图片，da
    ```
    // 低于1048576
    require('./../res/icon.jpg')
    // 高于1048576
    require('./../res/big.jpg')
    ```
3. 打包并查看结果
    ```
    // ./dist
    big.f22f26599897a8f5003aea3d070135bf.jpg
    index.bundle.js
    ```
    ```
    // ./index.bundle.js
    
    /***/ }),
    /* 2 */
    /***/ (function(module, exports) {
    
    module.exports = "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQABAAD/2wBDAAcFBQYFBAcGBgYIBwcICx...
    tDEx6snNS15Z9DcRe9a1n/AMew+tZI/qK1rP8A49h9a1p7Es//2Q=="
    
    /***/ }),
    /* 3 */
    /***/ (function(module, exports, __webpack_require__) {
    
    module.exports = __webpack_require__.p + "big.f22f26599897a8f5003aea3d070135bf.jpg";
    
    /***/ })
    /******/ ]);
    ```
    可以看出大于`1m`的那个图片文件被以文件的形式导出，而小于`1m`的文件被以`dataurl`的形式封装成模块
4. 参数说明
    - `limit`：限制文件大小，如果小于这个数，则转化成DATAURL，如果大于这个数，则使用`fallback`指定的`loader`再次装载，如果没有配置`fallback`，则默认调用`file-loader`
    - `mimetype`：这个只是用来指定`文件的mimetype`，因为有些文件没有后缀，或者后缀和文件不符合。
    - `fallback`：文件超出`limit`之后的加载器
    - **注意**：`url-loader`自身只有这3个参数，但是如果超出`limit`大小，将会执行下一个`loader`并且自动将配置的参数往下传，所以文中的案例的`name`其实是`file-loader`的参数，其他`loader`的参数同理也可以往下传
    - **注意**：这里的`use`其实还有另外一种`querystring`写法，不过不推荐
        ```
        {
                test: /\.(png|jpg|gif)$/,
                use:'url-loader?limit=1045876&name=[name].[hash].[ext]'
            }
        ```
5. 更多配置，可以查阅[webpack关于url-loader部分][4]。

### 0x007. 资源
- [源代码][5]


  [1]: https://segmentfault.com/a/1190000011884606
  [2]: https://webpack.js.org/loaders/raw-loader/
  [3]: https://webpack.js.org/loaders/file-loader/
  [4]: https://webpack.js.org/loaders/url-loader/
  [5]: https://github.com/followWinter/webpack-study/tree/master/0x009-loader-first-part
