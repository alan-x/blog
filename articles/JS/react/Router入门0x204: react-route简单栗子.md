### 0x000 概述
这一章仔细讲一讲 react-route 的使用栗子
### 0x001 简单使用
- 源码
    ```
    import React from 'react'
    import ReactDom from 'react-dom'
    import {BrowserRouter, Switch, Route, Link, withRouter} from 'react-router-dom'
    
    // 简单使用
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
- 效果

    ![clipboard.png](/img/bVbhTYx)


### 0x002 带导航激活效果
- 源码
    ```
    import React from 'react'
    import ReactDom from 'react-dom'
    import {BrowserRouter, Switch, Route, NavLink, withRouter} from 'react-router-dom'
    import './App.css'
    // 带导航效果
    class App extends React.Component {
        render() {
            return (
                <div>
                    <div>
                        <NavLink  to='/index' activeStyle=\{\{ color:'#eeeeee'\}\} >首页</NavLink>
                        <NavLink to='/article'  activeClassName='active'>文章</NavLink>
                        <NavLink to='/mine'  activeStyle=\{\{ color:'#eeeeee'\}\} isActive={(match,location)=>location.pathname==='/mine'}>我的</NavLink>
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
- 效果
    
    ![clipboard.png](/img/bVbhTZN)
    
- 说明
    `NavLink`: `Link`增强版，如果当前路由命中，将会启用`activeStyle`或者`activeClassName`。
    
    ```
    - activeStyle:?Object: 当链接激活的时候的样式
    - activeClassName:?String: 当链接激活的时候的样式类
    - isActive?Function: 可以手动判断该链接是否激活，该函数的签名是：function(match, location):boolean
    ```
    
    
### 0x003 重定向
- 源码
    ```
    import {BrowserRouter, Switch, Route, NavLink, Redirect, withRouter} from 'react-router-dom'
    
    class App extends React.Component {
        render() {
            return (
                <div>
                    <div>
                        <NavLink to='/index'>首页</NavLink>
                        <NavLink to='/article'>文章</NavLink>
                        <NavLink to='/mine'>我的</NavLink>
                    </div>
                    <hr/>
                    <Switch>
                        <Route path='/index' component={() => ({render: () => <p>首页</p>})}></Route>
                        <Route path='/article' component={() => ({render: () => <p>文章</p>})}></Route>
                        <Route path='/mine' component={() => ({render: () => <p>我的</p>})}></Route>
                        <Redirect from="/" to="/index"/>
                    </Switch>
                </div>
            )
        }
    }
    ```
- 效果
    当我们访问`http://localhost:8081/`时，会自动跳转到`http://localhost:8081/index`

### 0x004 没找到匹配的路由
- 源码
    ```
    import {BrowserRouter, Switch, Route, NavLink, withRouter} from 'react-router-dom'
    
    class App extends React.Component {
        render() {
            return (
                <div>
                    <div>
                        <NavLink to='/index'>首页</NavLink>
                        <NavLink to='/article'>文章</NavLink>
                        <NavLink to='/mine'>我的</NavLink>
                    </div>
                    <hr/>
                    <Switch>
                        <Route path='/index' component={() => ({render: () => <p>首页</p>})}></Route>
                        <Route path='/article' component={() => ({render: () => <p>文章</p>})}></Route>
                        <Route path='/mine' component={() => ({render: () => <p>我的</p>})}></Route>
                        <Route component={() => ({render: () => <p>未找到这个页面</p>})}/>
                    </Switch>
                </div>
            )
        }
    }
    ```
- 效果

    ![clipboard.png](/img/bVbhT1r)

### 0x005 url传参
- 源码
    ```
    import {BrowserRouter, Switch, Route, NavLink, withRouter} from 'react-router-dom'
    
    class Article extends React.Component{
        render(){
            return(<p>{this.props.match.params.id}</p>)
        }
    }
    class App extends React.Component {
        render() {
            return (
                <div>
                    <div>
                        <NavLink to='/index'>首页</NavLink>
                        <NavLink to='/article/1'>文章1</NavLink>
                        <NavLink to='/article/2'>文章2</NavLink>
                        <NavLink to='/mine'>我的</NavLink>
                    </div>
                    <hr/>
                    <Switch>
                        <Route path='/index' component={() => ({render: () => <p>首页</p>})}></Route>
                        <Route path='/article/:id' component={Article}></Route>
                        <Route path='/mine' component={() => ({render: () => <p>我的</p>})}></Route>
                    </Switch>
                </div>
            )
        }
    }
    
    
    ```
- 效果
    ![clipboard.png](/img/bVbhT2y)
- 说明
    声明`Route`的时候使用`:${name}`表示要作为动态参数，之后可以通过`this.props.match.params.${name}`获取
    
### 0x006 页面手动跳转并传参
- 源码
    ```
    class Article extends React.Component{
        render(){
            console.log(this)
            return(<p>文章 id:{this.props.location.state.id}</p>)
        }
    }
    class App extends React.Component {
        render() {
            return (
                <div>
                    <div>
                        <button className='btn btn-primary' onClick={()=>{this.props.history.push('/article',{id:1})}}>文章1</button>
                        <button className='btn btn-primary'  onClick={()=>{this.props.history.push('/article',{id:2})}}>文章2</button>
    
                    </div>
                    <hr/>
                    <Switch>
                        <Route path='/article' component={Article}></Route>
                    </Switch>
                </div>
            )
        }
    }
    
    ```
- 效果

    ![clipboard.png](/img/bVbhT22)

- 说明：
    - 跳转：this.props.history.push(path:String,data:?Object)
    - 获取参数：this.props.location.state

### 0x007 何时使用`Switch`
做个试验，假设我们有两个路由：
```
class App extends React.Component {
    render() {
        return (
            <div>
                <Route path='/article' component={()=><p>article</p>}></Route>
                <Route path='/:name' component={()=><p>:name</p>}></Route>
            </div>
        )
    }
}
```
此时看效果

![clipboard.png](/img/bVbhT3E)

会发现两个都命中了，这个时候可以使用`Switch`，他只会命中第一个命中的路由
```
class App extends React.Component {
    render() {
        return (
            <div>
                <Switch>
                    <Route path='/article' component={() => <p>article</p>}></Route>
                    <Route path='/:name' component={() => <p>:name</p>}></Route>
                </Switch>
            </div>
        )
    }
}
```
- 命中`/article`
    ![clipboard.png](/img/bVbhT3U)
- 命中`/:name`
    ![clipboard.png](/img/bVbhT3X)

### 0x008  资源
- [源码](https://github.com/followWinter/react-study)
