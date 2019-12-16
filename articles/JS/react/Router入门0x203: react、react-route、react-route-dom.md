### 0x000 概述
上一章使用的是自己实现的`route`，当然已经有现成的库给我们用了，那就是[react-route][1]。

### 0x001 `history Api`说明
在说这个库之前，得先对`history`新的`api`做一个了解
1. `window.history.pushState(data,title,?url)`
    - `data`：数据
    - `title`：标题
    - `url`：地址

        当我们调用该函数的时候，将改变地址栏的地址，但是却不会导致页面重新去后台获取，如下图操作，在第一次初始化完成之后，我们依次调用
        ```
        window.history.pushState({name:'a'},'','a')
        window.history.pushState({name:'b'},'','b')
        window.history.pushState({name:'c'},'','c')
        window.history.pushState({name:'d'},'','d')
        ```
        将会看到地址栏分别变成了：
        ```
        https://github.com/a
        https://github.com/b
        https://github.com/c
        https://github.com/d
        ```
        但是网络请求却没有发出
        
        ![clipboard.png](/img/bVbhQNa)

2. `window.history.state`：该变量可以获取到跳转当前所处`state`时传入的`data`:
    
    ![clipboard.png](/img/bVbhQNF)
    
3. `window.onpopstate`：这是一个事件，可以设置监听器，监听状态被`pop`出来的时候的事件，其中`go`、`back`会触发该事件
    
    ![clipboard.png](/img/bVbhQOj)


4. 总结
    根据以上几个特性，就可以和上一章一样，做出一个基于`history`模式的路由库了

###0x002 `history`库说明
[history][2]是一个针对该特性封装的库，以下是示例代码
```
import createHistory from 'history/createBrowserHistory'

const history = createHistory()

// 监听 location 的变化
const unlisten = history.listen((location, action) => {
    // location is an object like window.location
    console.log(action, location.pathname, location.state)
})
// 修改 location
history.push("/home", { some: "state" })
// 取消监听
unlisten()
```
查看浏览器

![clipboard.png](/img/bVbhQRI)

    
### 0x003 react-route
> `react-route`+`history`+`react`就可以形成一个套餐了
- 源码
    ```
    import React from 'react'
    import ReactDom from 'react-dom'
    import { Router, Route, Switch,withRouter } from 'react-router'
    import createHistory from 'history/createBrowserHistory'
    
    class App extends React.Component{
        render(){
            console.log(this)
            return(
                <div>
                    <div>
                        <button className="btn btn-primary" onClick={()=>{this.props.history.push('/index')}}>首页</button>
                        <button className="btn btn-primary" onClick={()=>{this.props.history.push('/article')}}>文章</button>
                        <button className="btn btn-primary" onClick={()=>{this.props.history.push('/mine')}}>我的</button>
                    </div>
                    <hr/>
                    <Switch>
                        <Route path='/index' component={()=>({render:()=><p>首页</p>})}></Route>
                        <Route path='/article' component={()=>({render:()=><p>文章</p>})}></Route>
                        <Route path='/mine' component={()=>({render:()=><p>我的</p>})}></Route>
                    </Switch>
                </div>
            )
        }
    }
    
    const history = createHistory()
    
    let MyApp=withRouter(App)
    
    ReactDom.render(
        <Router history={history}>
            <MyApp/>
        </Router>,
        document.getElementById('app')
    )
    ```
- 效果
    ![图片描述][3]


- 说明
    1. `App`组件说明
        `App`组件是跟组件，所有的组件都挂载在这个组件之下，在这个组件中，使用了两个`react-route`的组件，一个是`Switch`，用来在路由变化的时候切换显示的路由；一个是`Route`组件，一个`Route`代表一个页面，也代表一个组件，这里用了三个`Route`,每个`Route`对应一个路由，也对应一个组件，这里的组件为了方便，直接用匿名函数实现，分别是：
        ```
        ()=>({render:()=><p>首页</p>}) // 对应`/index`
        ()=>({render:()=><p>文章</p>}) // 对应`/article`
        ()=>({render:()=><p>我的</p>}) // 对应`/mine`
        ```
        当我们点击相应的按钮的时候，将会调用`this.props.history.push('${path}')`来跳转到对应的页面，其中`${path}`是我们设置的`Route`组件的`path`属性。
    2. `history`
        通过`history`库来做`location`监听和跳转
    3. `withRoute`是一个`HOC`，为`App`组件注入了`history`对象和路由相关的属性，这样可以屏蔽路由的存在，将`App`组件变成一个纯粹的组件
    4. `Router`组件接管了`histoy`对象，在该组件中完成了`history`的监听和路由的渲染
    
###0x004 react-route-dom
> 上面的调用太复杂了，需要手动创建`history`、调用`this.props.history.push('/index')`跳转，那有没有简单点的呢？那就是[react-router-dom][4]，这个库封装了`react-route`、`history`库，并提供了几个实用的组件

- 源码
```

import React from 'react'
import ReactDom from 'react-dom'
import {BrowserRouter, Switch, Route, Link, withRouter} from 'react-router-dom'

class App extends React.Component {
    render() {
        return (
            <div>
                <div>
                    <Link to='/index'>首页</Link>
                    <Link to='/article'>文章</Link>
                    <Link to='/mine'>我的</Link>
                </div>
                <hr/>
                <Switch>
                    <Route path='/index' component={() => ({render: () => <p>首页</p>})}></Route>
                    <Route path='/article' component={() => ({render: () => <p>文章</p>})}></Route>
                    <Route path='/mine' component={() => ({render: () => <p>我的</p>})}></Route>
                </Switch>
            </div>
        )
    }
}

let MyApp = withRouter(App)


ReactDom.render(
    <BrowserRouter>
        <MyApp/>
    </BrowserRouter>,
    document.getElementById('app')
)
```

- 说明：
    1. 使用`BrowserRouter`替代`Router`，并且不再手动创建`history`
    2. 使用`Link`直接跳转
    3. 该库是在`react-route`和`history`库的基础之上封装的，只是为了在`dom`环境下快速调用，并且提供了一个更加实用的组件而已，不能应为这个库而忘记了本质。

### 0x005 总结
看透它，然后掌握它

### 0x006 资源
- [源码](https://github.com/followWinter/react-study)



  [1]: https://github.com/ReactTraining/react-router
  [2]: https://github.com/ReactTraining/history
  [3]: /img/bVbhQJm
  [4]: https://github.com/ReactTraining/react-router/tree/master/packages/react-router-dom
