### 0x000 概述
开坑 `react` 系列文章，不知道会写到什么程度，毕竟写文章并不在行，存在当做笔记做，先不讲理论，实践先行。

### 0x001 创建项目
    ```
    $ mkdir 0x001-helloworld
    $ cd 0x001-helloworld
    $ yarn init -y
    ```
### 0x0002 添加依赖
```
$ yarn add @babel/core @babel/preset-react babel-loader html-webpack-plugin webpack webpack-cli --dev
```
- `@babel/core`: `babel` 核心包
- `@babel/preset-react`: `react` 的 `preset`，支持 `jsx` 等，具体看[这里](https://babeljs.io/docs/en/babel-preset-react)
- `babel-loader`: `babel` 的 `webpack loader`
- `webpack webpack-cli html-webpack-plugin`: `webpack`相关包和插件，其中 `html-webpack-plugin` 用来处理 `html` 模版


```
$ yarn add react react-dom
```
- `react react-dom`:`react`相关核心包

此时的`package.json`
```
    {
      "name": "0x001-helloworld",
      "version": "1.0.0",
      "description": "",
      "main": "index.js",
      "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "webpack-dev-server --color --process --hot"
      },
      "keywords": [],
      "author": "",
      "license": "ISC",
      "devDependencies": {
        "@babel/core": "^7.4.3",
        "@babel/preset-react": "^7.0.0",
        "babel-loader": "^8.0.5",
        "html-webpack-plugin": "^3.2.0",
        "webpack": "^4.30.0",
        "webpack-cli": "^3.3.0"
      },
      "dependencies": {
        "react": "^16.8.6",
        "react-dom": "^16.8.6"
      }
    }

```
### 0x003 配置 `babel`，添加`.babelrc`文件
    ```
    {
      "presets": [
        "@babel/preset-react"
      ]
    }
    ```
### 0x004 编写`webpack.config.js`
    ```
    const path = require('path')
    const HtmlWebpackPlugin = require('html-webpack-plugin');
    
    module.exports = {
        entry: path.resolve(__dirname, 'src/index.js'),
        mode: 'development',
        output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'bundle.js'
        },
        devServer: {
            open: true
        },
        module: {
            rules: [{
                test: /\.js$/,
                loader: "babel-loader"
            }]
        },
        plugins: [
            new HtmlWebpackPlugin({
                template: path.resolve(__dirname, "src/index.html")
            })
        ]
    }
    ```
### 0x005 编写源代码
- 创建目录
        ```
        $ mkdir src
        ```
- 编写`html`
        ```
        // index.html
        <!doctype html>
        <html>
        <head>
            <title>React Study</title>
        </head>
        <body>
        <div id="app"></div>
        </body>
        </html>
        ```
- 编写`js`
        ```
        //index.js
        import React from 'react'
        import ReactDom from 'react-dom'
        
        ReactDom.render(
            <h1>Hello, world!</h1>,
            document.getElementById('app')
        );
        ```
    
### 0x006 此时完整目录结构
```
+ react-study
    + 0x001-helloworld
        + src
            - index.html
            - index.js
        - .babelrc
        - package.json
        - webpack.config.js
        - yarn.lock
```


### 0x007 运行项目
```
npm start
```
查看浏览器：`http://localhost:8080/`

![clipboard.png](/img/bVbfD7b)

成功！！！


