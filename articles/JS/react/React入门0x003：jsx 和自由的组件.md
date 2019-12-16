### 0x000 概述

说起来`react`，我喜欢的还是他的思想，在`react`中，实际上没有`html`、`css`、`js`的区别，全部都是`js`，就和`webpack`一样，可以将所有的资源等同视之。但是这在一开始，就让很多人感觉很难受，毕竟在这之前，我们看过的大部分关于`web`的书、文章，提出的最佳实践就是`html`、`js`、`css`分开来，因为前端受够了在`js`中拼接模板，`php`、`java`之类的后端也受够了代码中拼接`html`，所以就有了模板引擎之类的存在。但是`react`这时候又出来说，我要把`html`写在`js`中，真是烦透咯！

不过，这种东西不过是50年一轮回，就和时尚一样。当时不容许这么做是受限于技术、时代之类的，而如今可以这么做是因为时机到了！

### 0x001 项目初始化
```
$ cp 0x001-helloworld 0x003-jsx-and-component
```
当前项目情况
```
+ 0x001-helloworld
+ 0x002-jsx
+ 0x003-jsx-and-component
    + src
        - index.html
        - index.js
    - .babelrc
    - package.json
    - webpack.config.js
    
```
### 0x002 自由的组件
1. 使用变量
    ```
    import React from 'react'
    import ReactDom from 'react-dom'
    
    const p = <p>这是`p`</p>
    
    ReactDom.render(
        p,
        document.getElementById('app')
    )
    ```
    查看浏览器：`http://localhost:8080/`

    ![clipboard.png](/img/bVbfMhD)

    说明：在这个案例中，使用了`ReactDom.render`方法，我们查看一下关于这个方法的[文档][1]：
    ```
    ReactDOM.render(element, container[, callback])
    ```
    
    参数：
        ```
         `element`: `React element`，可以使用`React.createElement`创建
         `container`: 容器
         `callback`: 回调，可选
        ```
    由参数1可知，我们只要提供一个`React element`便可，而由上一章可以知道，   `React.createElement`可以返回一个`React element`，所以我们可以这么调用：
    
            ```
            ReactDom.render(
                React.createElement(
                    "h1",
                    null,
                    "Hello, world!"
                ),
                document.getElementById('app')
            );
            ```
    
     而`jsx`经过`babel`翻译之后，又可以转化成`React.createElement`，所以，我们可以想这样调用：
    ```
    ReactDom.render(
        <h1>Hello, world!</h1>,
        document.getElementById('app')
    );
    ```
    自然，将它保存成一个变量，也是可以的了，
2. 使用方法
        ```
        const div = () => <div>这是`div`</div>
        
        ReactDom.render(
            div(),
            document.getElementById('app')
        )
        ```
    查看浏览器：`http://localhost:8080/`
        
       ![clipboard.png](/img/bVbfMiW)
        
    当然在`js`中的所有操作可以做到呢：
        ```
        
        import React from 'react'
        import ReactDom from 'react-dom'
        
        const aP = <p>这是`p`</p>
        
        const aDiv = () => <div>这是`div`</div>
        
        const divWithChildren = (...children) => {
            const len = children.length
            return <div>
                一共有{len}个子组件
                {
                    children.map((child, index) => {
                        return <div>
                            <span>这是第{index + 1}个子组件:</span>
                            {child}
                        </div>
                    })
                }
            </div>
        }
        
        ReactDom.render(
            divWithChildren(
                aP,
                aDiv(),
                <h1>h1</h1>
            ),
            document.getElementById('app')
        )
        ```
        
     查看浏览器：`http://localhost:8080/`
        
        
     ![clipboard.png](/img/bVbfMjP)


3. 使用组件
    ```
    class AComponent extends React.Component {
        render() {
            return <div>
                classComponent
            </div>
        }
    }
    
    ReactDom.render(
        <AComponent/>,
        document.getElementById('app')
    )
    ```
    查看浏览器：`http://localhost:8080/`
    
    ![clipboard.png](/img/bVbfMmY)
    
    说明：这种方式和上面两种有些不一样，之后会详细说来。

### 0x003 总结

- 案例一非常简单
- 案例二显得略复杂且杂乱，但是却能够很好的说明一点，那就是在`react`中，可以尽情的使用`js`的思想去构建一个简单组件，而简单组件又可以组合成复杂组件。相比于`angular`、`vue`等模板化的框架，`react`在构建`UI`方面让我们有更多的选择，同时也可以产生出许多非常具有突破性的思想！
- 案例三使用类来表示一个组件，非常符合面相对象的思想

当然，自由也带来混乱，需要将这种自由化为思想的自由，而不是工程的自由、代码的自由，否则将会带来噩梦。

### 0x004 资源
- [react](https://reactjs.org/)
- [transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)
- [源码](https://github.com/followWinter/react-study)


  [1]: https://reactjs.org/docs/react-dom.html#render
