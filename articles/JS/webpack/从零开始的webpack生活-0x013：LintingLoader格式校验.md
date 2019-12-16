### 0x001 概述
上一章讲的是[脚本装载相关的loader][1]，这一章见得是脚本格式校验相关的loader

### 0x002 环境配置
```
$ mkdir 0x013-linting-loader
$ cd 0x013-linting-loader
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
### 0x002 使用`eslint`校验js格式（这份配置是偷`vue`的）
1. 安装依赖包
    ```
    cnpm install --save-dev eslint eslint-loader eslint-config-standard eslint-friendly-formatter eslint-plugin-html eslint-plugin-import eslint-plugin-node eslint-plugin-promis eslint-plugin-standard
    ```
2. 修改配置文件
    ```
    ./webpack.config.js
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
            test   : /\.js$/,
            exclude: /node_modules/,
            enforce: "pre",
            include: [path.resolve(__dirname, 'src')],
            loader : "eslint-loader",
            options: {
              formatter: require('eslint-friendly-formatter')
            }
          }
        ]
      }
    }
    ;
    ```
3. 添加`eslint`配置文件
    ```
    // .eslintrc.js
    module.exports = {
      root         : true,
      parser       : 'babel-eslint',
      parserOptions: {
        sourceType: 'module'
      },
      env          : {
        browser: true,
      },
      // https://github.com/feross/standard/blob/master/RULES.md#javascript-standard-style
      extends      : 'standard',
      // required to lint *.vue files
      plugins      : [
        'html'
      ],
      // add your custom rules here
      'rules'      : {
        // allow paren-less arrow functions
        'arrow-parens'          : 0,
        // allow async-await
        'generator-star-spacing': 0,
        // allow debugger during development
        'no-debugger'           : 0
      }
    }
    
    // .eslintignore
    dist/*.js
    
    // ./.editconfig
    root = true
    
    [*]
    charset = utf-8
    indent_style = space
    indent_size = 2
    end_of_line = lf
    insert_final_newline = true
    trim_trailing_whitespace = true
    
    ```
4. 编写测试代码
    ```
    let  name="张三"
    ```
5. 打包
    ```
    $ webpack
    # 输出
    ERROR in ./src/index.js
    
      ✘  http://eslint.org/docs/rules/no-multi-spaces  Multiple spaces found before 'name'        
      src/index.js:1:6
      let  name="张三"
            ^
    
      ✘  http://eslint.org/docs/rules/no-unused-vars   'name' is assigned a value but never used  
      src/index.js:1:6
      let  name="张三"
            ^
    
      ✘  http://eslint.org/docs/rules/space-infix-ops  Infix operators must be spaced             
      src/index.js:1:10
      let  name="张三"
                ^
    
      ✘  http://eslint.org/docs/rules/quotes           Strings must use singlequote               
      src/index.js:1:11
      let  name="张三"
                 ^
    
    
    ✘ 4 problems (4 errors, 0 warnings)
    ```
这里爆出4个问题
- `let`和`name`之间只能有一个空格
- `name`变量没有使用过
- `张三`前边没有空格
- `张三`使用了双引号，应该用单引号
6. 修复
    ```
    let name = '张三'
    console.log(name)
    ```
7. 再次打包
    ```
    $ webpack
    # 输出
    Hash: 4014a2bcb0ede78b2270
    Version: webpack 3.8.1
    Time: 616ms
              Asset     Size  Chunks             Chunk Names
    index.bundle.js  2.63 kB       0  [emitted]  index
       [0] multi ./src/index.js 28 bytes {0} [built]
       [2] ./src/index.js 34 bytes {0} [built]
    ```
8. 更多配置，请查阅[eslint文档][3]
### 0x006 资源
- [源代码][4]


  [1]: https://segmentfault.com/a/1190000011930547
  [2]: https://github.com/MoOx/eslint-loader
  [3]: https://github.com/MoOx/eslint-loader
  [4]: https://github.com/followWinter/webpack-study/tree/master/0x013-linting-loader
