### 0x001 概述
[上一章][1]讲的是definePlugin的用法，和这一章依旧没有任何关系，这一章讲的时候`providerPlugin`。

### 0x002 插件介绍
就是提供全局变量啦

### 0x003 全局定义`jquery`栗子
1. 初始化项目
    ```
    + 0x006-provider-plugin
      + src
        - index.js
      - webpack.config.js
    ```
2. 安装依赖包
    ```
    $ cnpm init -y
    $ cnpm install webpack --save-dev
    $ cnpm install jquery --save
    ```
3. 编写`webpack.config.js`
    ```
    var path       = require('path')
    module.exports = {
        entry  : ['./src/index.js'],
        output : {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        }
    }
    
    ```
4. 添加插件，并定义`jQuery`
    ```
    var path       = require('path')
    var webpack    = require('webpack')
    module.exports = {
        entry  : ['./src/index.js'],
        output : {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        },
        plugins: [
            new webpack.ProvidePlugin({
                $     : 'jquery',
                jQuery: 'jquery'
            })
        ]
    }
    ```
5. 调用`jquery`
    ```
    // ./src/index.js
    $(document).ready(function () {
        console.log($('#name')[0].innerHTML)
    })
    // ./src/index.html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>providerPlugin</title>
    
    </head>
    <body>
    <p id="name">followWinter</p>
    </body>
    <script src="./../dist/index.js"></script>
    </html>
    ```
6. 打包并用浏览器打开`./src/index.html`,查看控制台
    
    ![clipboard.png](/img/bVX1ph)



### 0x004 全局定义自定义函数栗子
1. 添加定义
    ```
    timestamp: [path.resolve(__dirname, 'src/utils'), 'default']

    ```
2. 添加文件`./src/utils`
    ```
    export default function () {
        console.log(new Date().getTime())
    }
    ```
3. 调用
    ```
    // ./src/index.js
    timestamp()
    ```
4. 打包并执行
    ```
    $ webpack
    $ node ./dist/index.js
    # 输出
    1509977476685
    ```
### 0x005 资源
- [源代码][2]


  [1]: https://segmentfault.com/a/1190000011882514
  [2]: https://github.com/followWinter/webpack-study/tree/master/0x006-provider-plugin
