### 0x000 概述
这一章讲`react`中的样式
### 0x001 样式也可以是`js`
再声明一遍，在`js`中，什么都是`js`，样式也是，但是样式分为两种，内联样式和外联样式
1. 内联样式：
    > 内联样式可以写在标签的`style`属性中
    
    - 我们先尝试传统写法
    
        ```
        import React from 'react'
        import ReactDom from 'react-dom'
        
        class App extends React.Component {
            render() {
                return <div style="background:red">
                    <p>hello react</p>
                </div>
            }
        }
        
        ReactDom.render(
            <App></App>,
            document.getElementById('app')
        )
    
        ```
        
        查看浏览器，会发现报错，因为`style`期待的是一个像`{background:'red'}`一样键值对对象

        ![clipboard.png](/img/bVbgcIh)
        修改写法：
        ```jsx harmony
        import React from 'react'
        import ReactDom from 'react-dom'
        
        class App extends React.Component {
            render() {
                return <div style=\{\{background: 'red'\}\}>
                    <p>hello react</p>
                </div>
            }
        }
        
        ReactDom.render(
            <App></App>,
            document.getElementById('app')
        )

        ```
        查看浏览器，可以了
        ![clipboard.png](/img/bVbgcIJ)
        那为什么要两个`{}`呢？其实不是两个大括弧，第一个大括弧代表这里边执行`js`表达式，第二个括弧则代表对象，我们换一种写法就会更清晰一点了
        ```
        class App extends React.Component {
            constructor() {
                super()
                this.state = {
                    background: 'red'
                }
            }
        
            render() {
                return <div style={this.state}>
                    <p>hello react</p>
                </div>
            }
        }
        ```
        甚至还可以：
        ```
        
        class App extends React.Component {
            constructor() {
                super()
            }
            createStyle(){
                return{
                    background: 'red'
                }
            }
        
            render() {
                return <div style={this.createStyle()}>
                    <p>hello react</p>
                </div>
            }
        }
        
        ReactDom.render(
            <App></App>,
            document.getElementById('app')
        )
        
        ```
        记住咯，在`react`中，什么都可以是`js`，对于`style`，我们只需要返回一个对象就行了，不论是直接返回一个对象，还是通过复杂的函数创建对象，或者是其他野路子，只要给`style`一个对象就好了。当然这个对象的键值取值必须在`css`的有效取值范围内才行，否则，浏览器可解析不了。
2. 外链样式文件
    > 外联样式文件可以将样式存储在独立的文件中
    - 编写样式文件
        ```
        // style.css
        div {
            background: red;
        }
        ```
    - 编写组件
        ```
        // index.js
        import React from 'react'
        import ReactDom from 'react-dom'
        import './style.css'
        class App extends React.Component {
            constructor() {
                super()
            }
        
            render() {
                return <div>
                    <p>hello react</p>
                </div>
            }
        }
        
        ReactDom.render(
            <App></App>,
            document.getElementById('app')
        )
        ```
     - 添加对`css`的支持
       ```
       $ npm install --save-dev css-loader style-loader
       $ vim webpack.config.js
       
       const path = require('path')
        var HtmlWebpackPlugin = require('html-webpack-plugin');
        
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
                rules: [
                    {
                        test: /\.js$/,
                        loader: "babel-loader"
                    },
                    {
                        test: /\.css$/,
                        loaders: [
                            'style-loader',
                            'css-loader'
                        ]
                    }
                ]
            },
            plugins: [
                new HtmlWebpackPlugin({
                    template: path.resolve(__dirname, "src/index.html")
                })
            ]
        }
       ```
        **说明**：外联样式的能力不是`react`提供的，而是`webpack`，`webpack`可以将`style.css`插入到文件中，从而渲染到`react`最后生成的`dom`上，可以查看浏览器：
        
        
        ![clipboard.png](/img/bVbgcKj)

### 0x003 总结
`react`中都是`js`，样式也是`js`，所以`react`中，样式也可以编程的，可以完全动态的方式生成样式，当然还是那句话：不受控制的自由将带来灾难，思想的自由才能铸就自我。

### 0x004 资源

- [react](https://reactjs.org/)
- [源码](https://github.com/followWinter/react-study)
        
