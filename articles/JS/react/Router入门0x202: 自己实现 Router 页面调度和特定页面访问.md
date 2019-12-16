### 0x000 概述
上一章讲了`SPA`如何实现组件/页面切换，这一章讲如何解决上一章出现的问题以及如何优雅的实现页面切换。
### 0x001 问题分析
回顾一下上一章讲的页面切换，我们通过`LeactDom.render(new ArticlePage(),document.getElementById('app'))`来切换页面，的确做到了列表页和详情页的切换，但是我们可以看到，浏览器的网址始终没有变化，是`http://localhost:8080`，那如果我们希望直接访问某个页面，比如访问某篇文章呢？也就是我们希望我们访问的地址是`http://localhost:8080/article/1`，进入这个地址之后，可以直接访问id 为1的文章。
问题1：无法做到访问特定页面并传递参数
问题2：通过`LeactDom.render(new ArticlePage(),document.getElementById('app'))`太冗余了

### 0x002 简单的路由实现并和原生`js`结合：
> 基本上也是基于`发布-订阅`模式，
> register: 注册路由
> push: 路由跳转

- 源码
```

class Router {
    static routes = {}

    /**
     * 如果是数组
     * 就遍历数组并转化成 {"/index":{route:{...},callback:()=>{....}}} 形式
     * 并执行 init 方法
     * 如果是对象
     * 就转化成 {"/index":{route:{...},callback:()=>{....}}} 形式
     * 并和原来的 this.route 合并
     * 注意: 如果用对象形式必须手动执行 init 方法
     * 最终 this.route 形式为
     * [
     *  {"/index":{route:{...},callback:()=>{....}}}
     *  {"/detail":{route:{...},callback:()=>{....}}}
     * ]
     * @param routes
     * @param callback
     */
    static register(routes, callback) {
        if (Array.isArray(routes)) {
            this.routes = routes.map(route => {
                return {
                    [route.path]: {
                        route: route,
                        callback: callback
                    }
                }
            }).reduce((r1, r2) => {
                return {...r1, ...r2}
            })
        }
        this.routes = {
            ...this.routes,
            ...{
                [routes.path]: {
                    route: routes,
                    callback: callback
                }
            }
        }
    }

    /**
     * 跳转到某个路由
     * 本质是遍历所有路由并执行 callback
     *
     * @param path
     * @param data
     */
    static push(path, data) {
        Object.values(this.routes).forEach(route => {
            route.callback(data, this.routes[ path].route, path)
        })
    }

}

export default Router

```
- 使用

    
    ```
    import Router from "./core/Router";

    Router.register([
        {
            path: "/index",
            name: "主页",
            component: (props) => {
                return document.createTextNode(`这是${props.route.name}`)
            }
        },
        {
            path: "/detail",
            name: "详情页",
            component: (props) => {
                return document.createTextNode(`这是${props.route.name}`)
            }
        }
    ], (data, route, match) => {
        if (route.path === match) {
            let app = document.getElementById('app')
            app.childNodes.forEach(c => c.remove())
            app.appendChild(new route.component({data,route,match}))
        }
    })
    
    Router.push('/index')
    
    setTimeout(()=>{
        Router.push('/detail')
    },3000)

    ```
- 说明：
    当`push`方法调用的时候，会触发`register`的时候传入的`callback`，并找到`push`传入的`path`匹配的路由信息，然后将该路由信息作为`callback`的参数，并执行`callback`。
    在上面的流程中，我们注册了两个路由，每个路由的配置信息大概包含了`path`、`name`、`component`三个键值对，但其实只有`path`是必须的，其他的都是非必须的，可以结合框架、业务来传需要的参数；在注册路由的同时传入了路由触发时的动作。这里设定为将父节点的子节点全部移除后替换为新的子节点，也就达到了组件切换的功能，通过`callback`的`props`参数，我们可以获取到当前触发的路由配置和触发该路由配置的时候的数据，比如后面调用`Route.push('/index',{name:1})`的时候，`callback`的`props`为
    ```
    {
        data:{
            name:1
        },
        route:{ 
            path: "/index",
            name: "主页",
            component: (props) => {
                    return document.createTextNode(`这是${props.route.name}`)
                }
        }
    }
    ```
### 0x003 和上一章的`SPA`结合
```
import Router from "./core/Router";
import DetailPage from "./page/DetailPage";
import ArticlePage from "./page/ArticlePage";
import LeactDom from "./core/LeactDom";

Router.register([
    {
        path: "/index",
        name: "主页",
        component: ArticlePage
    },
    {
        path: "/detail",
        name: "详情页",
        component: DetailPage
    }
], (data, route,match) => {
    if (route.path !== match) return
    LeactDom.render(new route.component(data), document.getElementById('app'))
})


```
然后在页面跳转的地方，修改为`Route`跳转
```
    // ArticlePage#componentDidMount
    componentDidMount() {
            let articles = document.getElementsByClassName('article')
            ;[].forEach.call(articles, article => {
                    article.addEventListener('click', () => {
                        // LeactDom.render(new DetailPage({articleId: article.getAttribute('data-id')}), document.getElementById('app'))
                        Router.push('/detail',{articleId:article.getAttribute('data-id')})
                    })
                }
            )
    
        }

    // DetailPage#componentDidMount
    componentDidMount() {
        document.getElementById('back').addEventListener('click', () => {
            LeactDom.render(new ArticlePage(), document.getElementById('app'))
            Router.push('/index')
        })
    }
```

### 0x004 指定跳转页面-hash
先看结果，我们希望我们在访问`http://localhost:8080/#detail?articleId=2`的时候跳转到`id=2`的文章的详情页面，所以我们需要添加几个方法:
```
    import Url from 'url-parse'

class Router {
    static routes = {}

    /**
     * 初始化路径
     * 添加 hashchange 事件, 在 hash 发生变化的时候, 跳转到相应的页面
     * 同时根据访问的地址初始化第一次访问的页面
     *
     */
    static init() {
        Object.values(this.routes).forEach(route => {
            route.callback(this.queryStringToParam(), this.routes['/' + this.getPath()].route,'/'+this.getPath())
        })

        window.addEventListener('hashchange', () => {
            Object.values(this.routes).forEach(route => {
                route.callback(this.queryStringToParam(), this.routes['/' + this.getPath()].route,'/'+this.getPath())
            })
        })

    }

    /**
     * 如果是数组
     * 就遍历数组并转化成 {"/index":{route:{...},callback:()=>{....}}} 形式
     * 并执行 init 方法
     * 如果是对象
     * 就转化成 {"/index":{route:{...},callback:()=>{....}}} 形式
     * 并和原来的 this.route 合并
     * 注意: 如果用对象形式必须手动执行 init 方法
     * 最终 this.route 形式为
     * [
     *  {"/index":{route:{...},callback:()=>{....}}}
     *  {"/detail":{route:{...},callback:()=>{....}}}
     * ]
     * @param routes
     * @param callback
     */
    static register(routes, callback) {
        if (Array.isArray(routes)) {
            this.routes = routes.map(route => {
                return {
                    [route.path]: {
                        route: route,
                        callback: callback
                    }
                }
            }).reduce((r1, r2) => {
                return {...r1, ...r2}
            })
            this.init()
        }
        this.routes = {
            ...this.routes,
            ...{
                [routes.path]: {
                    route: routes,
                    callback: callback
                }
            }
        }
    }

    /**
     * 跳转到某个路由
     * 其实只是简单的改变 hash
     * 触发 hashonchange 函数
     *
     * @param path
     * @param data
     */
    static push(path, data) {
        window.location.hash = this.combineHash(path, data)
    }

    /**
     * 获取路径
     * 比如 #detail => /detail
     * @returns {string|string}
     */
    static getPath() {
        let url = new Url(window.location.href)
        return url.hash.replace('#', '').split('?')[0] || '/'
    }

    /**
     * 将 queryString 转化成 参数对象
     * 比如 ?articleId=1 => {articleId: 1}
     * @returns {*}
     */
    static queryStringToParam() {
        let url = new Url(window.location.href)
        let hashAndParam = url.hash.replace('#', '')
        let arr = hashAndParam.split('?')
        if (arr.length === 1) return {}
        return arr[1].split('&').map(p => {
            return p.split('=').reduce((a, b) => ({[a]: b}))
        })[0]
    }

    /**
     * 将参数变成 queryString
     * 比如 {articleId:1} => ?articleId=1
     * @param params
     * @returns {string}
     */
    static paramToQueryString(params = {}) {
        let result = ''
        Object.keys(params).length && Object.keys(params).forEach(key => {
            if (result.length !== 0) {
                result += '&'
            }
            result += key + '=' + params[key]
        })
        return result
    }

    /**
     * 组合地址和数据
     * 比如 detail,{articleId:1} => detail?articleId=1
     * @param path
     * @param data
     * @returns {*}
     */
    static combineHash(path, data = {}) {
        if (!Object.keys(data).length) return path.replace('/', '')
        return (path + '?' + this.paramToQueryString(data)).replace('/', '')
    }
}

export default Router


```
说明：这里修改了`push`方法，原本`callback`在这里调用的，但是现在换成在`init`调用。在`init`中监听了`hashchange`事件，这样就可以在`hash`变化的时候，需要路由配置并调用`callback`。而在监听变化之前，我们先调用了一次，是因为如果我们第一次进入就有`hash`，那么就不会触发`hanshchange`，所以我们需要手动调用一遍，为了初始化第一次访问的页面，这样我们就可以通过不同的地址访问不同的页面了，而整个站点只初始化了一次（在不使用按需加载的情况下），体验非常好，还要另外一种实行这里先不讲，日后有空独立出来讲关于路由的东西。

### 0x005 将自己实现的路由和`React`集成
- 重构`ArticlePage`
```
class ArticlePage extends React.Component {

    render() {
        return <div>
            <h3>文章列表</h3>
            <hr/>
            {
                ArticleService.getAll().map((article, index) => {
                    return <div key={index} onClick={() => this.handleClick(article)}>
                        <h5>{article.title}</h5>
                        <p>{article.summary}</p>
                        <hr/>
                    </div>
                })
            }
        </div>

    }

    handleClick(article) {
        Router.push('/detail', {articleId: article.id})
    }
}
```
- 重构`DetailPage`
```

class DetailPage extends React.Component {
    render() {
        const {title, summary, detail} = ArticleService.getById(this.props.data.articleId)
        return <div>
            <h3>{title}</h3>
            <p>{summary}</p>
            <hr/>
            <p>{detail}</p>
            <button className='btn btn-success'  onClick={() => this.handleClick()}>返回</button>
        </div>
    }

    handleClick() {
        Router.push('/index')
    }
}
```
- 重构路由配置和渲染
```

const routes = [
    {
        path: "/index",
        name: "主页",
        component: ArticlePage
    },
    {
        path: "/detail",
        name: "详情页",
        component: DetailPage
    }
];


Router.register(routes, (data, route) => {
    let Component = route.component
    ReactDom.render(
        <Component {...{data, route}}/>,
        document.getElementById("app")
    )
})
```
### 0x006 为`React`定制`Router`组件
> 在上面每调用一次`Router.push`，就会执行一次`ReactDom.render`，并不符合`React`的思想，所以，需要为`React`定义一些组件
- `RouteApp`组件
```
class RouterApp extends React.Component {
    componentDidMount(){
        Router.init()
    }
    render() {
        return {...this.props.children}
    }

}
```

- `Route`组件
```

class Route extends React.Component {
    constructor(props) {
        super()
        this.state={
            path:props.path,
            match:'',
            data:{}
        }
    }

    componentDidMount() {
        Router.register({
            path: this.props.path
        }, (data, route) => {
            this.setState({
                match:route.path,
                data:data
            })
        })

    }

    render() {
        let Component = this.props.component
        if (this.state.path===this.state.match){
            return <Component {...this.state.data}/>
        }
        return null
    }
}
```
- 使用
```

class App extends React.Component {
    render() {
        return (<div>
            <Route path="/index" component={ArticlePage}/>
            <Route path="/detail" component={DetailPage}/>
        </div>)
    }
}


ReactDom.render(
    <RouterApp>
        <App/> 
    </RouterApp>,
    document.getElementById('app')
)
```

- 说明
在`RouterApp`组件中调用了`Route.init`来初始化调用,然后在每个`Route`中注册路由，每次路由变化的时候都会导致`Route`组件更新，从而使组件切换。

### 0x007 总结

路由本身是不带有任何特殊的属性的，在与框架集成的时候，应该考虑框架的特点，比如`react`的时候，我们可以使用`react`和`react-route`直接结合，也可以通过使用`react-route-dom`来结合。

### 0x008 资源
- [源码](https://github.com/followWinter/react-study)


