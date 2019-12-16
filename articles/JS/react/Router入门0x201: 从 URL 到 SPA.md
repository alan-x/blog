### 0x000 概述
从这一章开始就进入路由章节了，并不直接从如何使用`react-route`来讲，而是从路由的概念和实现来讲，达到知道路由的本质，而不是只知道如何使用`react-route`库的目的，毕竟`react-route`只是一个库，是路由的一个实现而已，而不是路由本身。

### 0x001 URL的概念
很多人对`url`的理解就是网址，我们在浏览器地址栏输入网址，便可以访问到特定网页，但其实`url`的含义远远不止是网址。`url`的全称是统一资源定位符（英文：Uniform Resource Locator），可以这么说，`url`是一种标准，而网址则是符合`url`标准的一种实现而已。

让我们做几个实验：
1. 打开浏览器，访问`segmentfault`的主页，此时地址栏显示的是：
    ```
    https://segmentfault.com
    ```
2. 桌面新建`from-url-to-spa.txt`文件，输入内容`from url to spa`，并拖拽到浏览器，此时浏览器显示的是
    ```
    file:///Users/FollowWinter/Desktop/from-url-to-spa.txt
    ```
3. 打开一个github项目，并选择ssh访问，我们可以得到以下地址：
    ```
    git@github.com:followWinter/flex-layout.git
    ```
**说明**：其中，1访问了一个网页，2访问了一个本地文件，3访问了一个开源项目，从以上可以看出，url有多种用途各异的实现，但是我们可以这么归纳，网络上（包括本地和远程）所有的的东西都看作资源，我们可以通过一种符合某种标准的格式来访问这种资源，从而忽略**设备类型**(服务器、路由器、硬盘......)、**网络类型**（远程、本地......）、**资源类型**（文本、图片、音乐、电影......），而这种标准就是url，也就是我对统一资源定位符的理解。

- 统一资源定位符的标准格式如下：
    ```
    协议类型:[//服务器地址[:端口号]][/资源层级UNIX文件路径]文件名[?查询][#片段ID]
    ```

- 统一资源定位符的完整格式如下：
    ```
    协议类型:[//[访问资源需要的凭证信息@]服务器地址[:端口号]][/资源层级UNIX文件路径]文件名[?查询][#片段ID]
    ```

### 0x002 spa是什么
`SPA`全称是`single page web application`，也就是只有一个页面的`web`应用程序，我们访问一个网页，能够在这个网页上完成所有的业务操作，我们就可以称之为`SPA`，是和框架无关、技术无关的一个概念。并不是说用`angular`、`vue`、`react`实现的`web`应用才叫`SPA`，因为这些框架也可以在多页应用中使用。

### 0x003 如何实现`spa`
只要在一个页面完成所有业务操作，就可以称之为`SPA`了，所以实现所谓的`SPA`也很简单，就是将原本多页的步骤转化为一个页面就行了。

### 0x004 `SPA`和路由有啥关系啊
回答：没有关系。`SPA`不一定要使用路由，不使用也没有关系，但是随着单页应用了扩大，将所有的逻辑都卸载一个页面上，会导致逻辑爆炸，维护痛苦，所以在逻辑上又分为多个页面，达到好维护的效果。

### 0x005 路由出现
一开始是没有路由的，但是做的应用多了，便有了路由。对于路由的需求有两个：
1. 维护上的需求，过多的逻辑写在一个页面上，容易混乱，所以用路由分离单独逻辑和页面。
2. 状态保存的需求，比如一个`SPA`，我们有文章和文章详情页，有一天我们需要分享一个文章，希望可以通过一个链接直接访问到这篇文章。但是单页应用是无状态的，而网址又是唯一的，比如`a.com/index.html`，无法做到直接访问详情页，所以就出现了一些方案：
    - hash：`a.com/index.html#detail/1`，访问 id 为1的文章详情页
    - url：`a.com/index/detail/1`，访问 id 为1的文章详情页
这样我们就可以分享一篇文章给其他用户了，方案1实现比较简单，但是路由丑陋并且占用了 hash 符，页面中就不能乱用 hash 符了。方案2好但是需要后端配合，实现也很简单，不管这个 url 是什么，都返回单页应用的 html 就好了。

### 0x006 实现简单的`SPA`
1. 架构：
    - 组件，每个组件都是独立的，可以渲染出自己的`dom`，并且可以绑定事件，拥有生命周期。
    - 渲染器，将组件渲染到页面上。
    - 服务，做数据管理等一些逻辑服务。
2. 项目初始化:
   > 整个项目起始没有啥特别的，只是支持了`es6`而已，而整个项目我们也将会用`es6`来实现
    - 初始化项目及其目录
        + 0x021-spa
            + src 
                + core
                + page
                + services
                - index.html
                - index.js
            - .babelrc
            - package.json
            - webpack.config.js
    - `index.html`:
        ```
        <!doctype html>
        <html>
        <head>
            <title>React Study</title>
            <!--直接引入`bootstrap`样式，让 `demo` 好看一点-->
            <link href="https://cdn.bootcss.com/bootstrap/4.1.1/css/bootstrap.min.css" rel="stylesheet">
        </head>
        <body class="container">
        <div id="app">
        </div>
        </body>
        </html>
        ```
    - `.babelrc`
        ```
        {
          "presets": [
            "env",
            "stage-3"
        
          ]
        }
        ```
    - `package.json`
        ```
        {
          "name": "0x021-spa",
          "version": "1.0.0",
          "description": "",
          "main": "index.js",
          "scripts": {
            "test": "echo \"Error: no test specified\" && exit 1",
            "start": "webpack-dev-server --color --process "
          },
          "keywords": [],
          "author": "",
          "license": "ISC",
          "devDependencies": {
            "babel-cli": "^6.26.0",
            "babel-core": "^6.26.3",
            "babel-loader": "^7.1.5",
            "babel-preset-env": "^1.7.0",
            "babel-preset-react": "^6.24.1",
            "babel-preset-stage-3": "^6.24.1",
            "html-webpack-plugin": "^3.2.0",
            "webpack": "^4.16.5",
            "webpack-cli": "^3.1.0",
            "webpack-dev-server": "^3.1.5"
          }
         
        }

        ```
     - `webpack.config.js`:
         ```
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
        
                ]
            },
            plugins: [
                new HtmlWebpackPlugin({
                    template: path.resolve(__dirname, "src/index.html")
                })
            ]
        }
         ```
3. 渲染器实现
    > 渲染器的作用起始就是渲染组件而已，而每个组件都有一个`render`方法，该方法返回一个`dom`字符串，也就是说，渲染器的本质就是将`dom`字符串挂载和卸载。
    - `core/LeactDom.js`
        ```
        class LeactDom {
            static render(child, parent) {
                parent.innerHTML=child
            }
        }
        
        export default LeactDom
        ```
    - 测试`index.js`
        ```
        import LeactDom from "./core/LeactDom";
        import LeactDom from "./core/LeactDom";
        
        LeactDom.render(`<p id="p">这是一个p</p>`, document.getElementById('app'))
        document.getElementById('p').addEventListener('click', () => {
            LeactDom.render("<span>这是一个span</span>", document.getElementById('app'))
        
        })

        ```
    - 查看浏览器
         如图，我们已经实现了切换了，只需要将之封装为组件就行了
        ![图片描述][1]    
4. 组件
    - `core/Component.js`
        ```
        // 这是组件根类, 所有的组件都继承这个根
        class Component {
            // 返回 dom 字符串
           render() {
                return ''
            }
        
            // dom 挂载上去以后 执行该方法, 可以在这个方法上执行 dom 查询和事件绑定
            componentDidMount() {
        
            }
        
        }
        
        export default Component
        ```
    - 自定义组件`page/Hello.js`
        ```
        import Component from "../core/Component";

        class Hello extends Component {
        
            render() {
                return `<p id='hello'>hello</p>`
            }
        
            componentDidMount() {
                document.getElementById('hello').addEventListener('click', () => {
                    alert('hello')
                })
            }
        }
        
        export default Hello
        ```
    - 引入`Hello`组件
        ```
        import LeactDom from "./core/LeactDom";
        import Hello from "./page/Hello";
        
        LeactDom.render(Hello,document.getElementById('app'))
        ```
    - 修改`LeactDom`
        ```
        class LeactDom {
        
            static render(child, parent, props={}) {
                if (typeof child === 'function') {
                    let comp = new child()
                    comp.props = props
                    parent.innerHTML = comp.render()
                    comp.componentDidMount()
                } else {
                    parent.innerHTML = child
                }
            }
        }
        
        export default LeactDom
        ```
    - 查看效果
        
     ![clipboard.png](/img/bVbhksQ)

5. 框架完成开始编写服务
    - 文章获取服务`service/AticleService.js`
        ```
        const articles = [
            {
                id: 1,
                title: "Redux入门0x101: 简介及`redux`简单实现",
                summary: "简介及`redux`简单实现",
                detail: "详情1"
            },
            {
                id: 2,
                title: "Redux入门0x102: redux 栗子之 counter",
                summary: "redux 栗子之 counter",
                detail: "详情2"
            },
            {
                id: 3,
                title: "Redux入门0x103: 拆分多个 reducer",
                summary: "拆分多个 reducer",
                detail: "详情3"
            },
            {
                id: 4,
                title: "Redux入门0x104: Action Creators",
                summary: "Action Creators",
                detail: "详情4"
            },
            {
                id: 5,
                title: "Redux入门0x105: redux 中间件",
                summary: "redux 中间件",
                detail: "详情5"
            },
        
        ]
        
        class ArticleService {
        
            static getAll() {
                return articles
            }
        
            static getById(id) {
                return articles.find((article) => {
                    return id == article.id
                })
            }
        }
        
        export default ArticleService
        ```
6. 开始编写自定义组件
    - 文章列表组件
        ```
        import ArticleService from "../services/ArticleService";
        import DetailPage from "./DetailPage";
        import LeactDom from "../core/LeactDom";
        
        class ArticlePage {
            render() {
                let articlesListString = ArticleService.getAll()
                    .map(article => {
                        return `<div class="article" data-id="${article.id}">
                        <h5>${article.title}</h5>
                        <p>${article.summary}</p>
                        <hr>
                    </div>`
                    })
                    .reduce((article1, article2) => {
                        return article1 + article2
                    })
                let articleListContrinerString = `<div>
                    <h3>文章列表</h3>
                    <hr>
                    <div>
                    ${articlesListString}
                    </div>
                </div>`
        
                return articleListContrinerString
            }
        
            componentDidMount() {
                let articles = document.getElementsByClassName('article')
                ;[].forEach.call(articles, article => {
                        article.addEventListener('click', () => {
                            LeactDom.render(new DetailPage({articleId: article.getAttribute('data-id')}), document.getElementById('app'))
                        })
                    }
                )
        
            }
        
        }
        
        export default ArticlePage
        ```
    - 文章详情组件
        ```
        import ArticleService from "../services/ArticleService";
        import Component from "../core/Component";
        import LeactDom from "../core/LeactDom";
        import ArticlePage from "./ArticlePage";
        
        class DetailPage extends Component {
            constructor(props) {
                super()
                this.article = ArticleService.getById(props.articleId)
            }
        
            render() {
                const {title, summary, detail} = this.article
                return `<div>
                    <h3>${title}</h3>
                    <p>${summary}</p>
                    <hr>
                    <p>${detail}</p>
                    <button id="back" type="button" class="btn btn-success">返回</button>
                </div>`
            }
        
        
            componentDidMount() {
                document.getElementById('back').addEventListener('click', () => {
                    LeactDom.render(new ArticlePage(), document.getElementById('app'))
                })
            }
        }
        
        export default DetailPage

        ```
7. 加载组件`index.js`
    ```
    import LeactDom from "./core/LeactDom";
    import ArticlePage from "./page/ArticlePage";
    
    LeactDom.render(new ArticlePage(),document.getElementById('app'))
    ```
8 查看最终效果

![图片描述][2]

### 0x007 总结
这里要做的只是一个案例，而不是写一个完整的框架，所以在很多地方并没有完善，只是为了验证实现`SPA`的方式，而结果也确实验证了。也将一些问题暴露出来了，其他的问题我们不关心，我们只关心我们之前提出的问题，只有一个网址，如何将某个页面分享出去，很明显，做成`SPA`之后，无法将文章详情页面分享给他人。解决 方法也已经给出来了：
- hash
- url
将在下一张讲述如何解决
### 0x008 资源
- [源码](https://github.com/followWinter/react-study)

  [1]: /img/bVbhklF
  [2]: /img/bVbhkwF
