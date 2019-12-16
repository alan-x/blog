### 0x001 概述
上一章讲的是[样式装载相关的loader][1]，这一章见得是脚本加载相关的`loader`
### 0x002 环境配置
    ```
    $ mkdir 0x012-transliling-loader
    $ cd 0x012-transliling-loader
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
### 0x003 栗子1-`babel-laoder`加载`es6`
1. 安装依赖
    ```
    $ cnpm install --save-dev babel-loader babel-core babel-preset-env
    ```
2. 修改配置
    ```
    const path         = require('path');
    
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
                    test   : /\.js$/,
                    exclude: /node_modules/,
                    use    : {
                        loader: "babel-loader"
                    }
                }
            ]
        }
    }
    ;
    ```
3. 添加`babel`配置文件
    ```
    $ vim ./.babelrc
    // ./.babelrc
    {
      "presets": ["es2015"]
    }
    ```
4. 使用`es6`编写脚本
    ```
    $ vim ./src/index.js
    // ./src/index.js
    let sayHello = () => {
        console.log('hello webpack')
    }
    sayHello();
    ```
5. 打包并查看结果
    ```
    // ./dist/index.bundle.js
    /***/ }),
    /* 1 */
    /***/ (function(module, exports, __webpack_require__) {
    
    "use strict";
    
    
    var sayHello = function sayHello() {
        console.log('hello webpack');
    };
    sayHello();
    
    /***/ })
    /******/ ]);
    ```
    可以看到，`es6`语法被自动转化成了`es5`
6. 更多配置请查阅[webpack关于babel-loader章节][2]
### 0x004 栗子2-`awesome-typescript-loader `加载`typescript`
1. 安装依赖
    ```
    $ cnpm install --save-dev awesome-typescript-loader
    ```
2. 修改配置
    ```
    const path = require('path');
    
    module.exports = {
        entry  : {
            'index': ['./src/index.ts'],
        },
        output : {
            path    : path.join(__dirname, 'dist'),
            filename: '[name].bundle.js'
        },
        resolve: {
            extensions: ['.ts', '.tsx', '.js', '.jsx']
        },
        module : {
            rules: [
                {
                    test  : /\.tsx?$/,
                    loader: 'awesome-typescript-loader'
                }
            ]
        }
    }
    ;
    ```
3. 添加`typescript`的配置
    ```
    $ vim ./tsconfig.json
    // ./tsconfig.json
    {
      "compilerOptions": {
        "noImplicitAny": true,
        "removeComments": true
      },
      "awesomeTypescriptLoaderOptions": {
        /* ... */
      }
    }
    ```
4. 编写`index.ts`
    ```
    class Greeter {
        greeting: string;
    
        constructor(message: string) {
            this.greeting = message;
        }
    
        greet() {
            return "Hello, " + this.greeting;
        }
    }
    
    let greeter = new Greeter("world");
    ```
5. 编译并查看结果
    ```
    /* 1 */
    /***/ (function(module, exports) {
    
    var Greeter = (function () {
        function Greeter(message) {
            this.greeting = message;
        }
        Greeter.prototype.greet = function () {
            return "Hello, " + this.greeting;
        };
        return Greeter;
    }());
    var greeter = new Greeter("world");
    
    
    /***/ })
    /******/ ]);
    ```
    可以看出，`typescript`被自动转化成了`js`
6. 其他更多配置，请查阅[awesome-typescript-loader相关说明][3]

### 0x005 资源
    - [源代码][4]


  [1]: https://segmentfault.com/a/1190000011929459
  [2]: https://webpack.js.org/loaders/babel-loader/
  [3]: https://github.com/s-panferov/awesome-typescript-loader
  [4]: https://github.com/followWinter/webpack-study/tree/master/0x012-transpiling-loader
