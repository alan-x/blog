### 0x000 概述
这一章终于大集成了
### 0x001 集成`react`
- 源码
    ```
    import React from 'react'
    import ReactDom from 'react-dom'
    
    class App extends React.Component {
        render() {
            return (<div>react</div>)
        }
    }
    
    ReactDom.render(
        <App/>,
        document.getElementById('app')
    )
    ```
- 效果
    
    ![clipboard.png](/img/bVbhT93)

- 说明：
    - 集成`react`主要是用到`react`、`react-router`库

### 0X002 集成`react-router`
- 源码
    ```
    import React from 'react'
    import ReactDom from 'react-dom'
    import {BrowserRouter, Route, withRouter} from 'react-router-dom'
    
    class Article extends React.Component {
        render() {
            return (
                <p>article</p>
            )
        }
    }
    
    let MyArticle = withRouter(Article)
    
    class App extends React.Component {
        render() {
            return (<div>
                <Route component={MyArticle}></Route>
            </div>)
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
    ![clipboard.png](/img/bVbhUaO)
- 说明
    - 主要用到`react-router-dom`库，是针对`react-router`库在`dom`环境下的封装
    - `withRouter`组件，注入`match`、`location`、`history`等属性
    - `BrowserRouter`接管跟组件
    - `Route`指定路由和组件的对应关系

### 0x003 集成`redux`
- 源码
    - 引入`redux`相关的包
        ```
        import {Provider, connect} from 'react-redux'
        import {createStore} from 'redux'
        ```
    - `reducer`
        ```
        const counter = (state = {counter: 0, num: 0}, action) => {
            switch (action.type) {
                case ACTION_INCREMENT:
                    return {...state, ...{counter: ++state.counter}}
                case ACTION_DECREMENT:
                    return {...state, ...{counter: --state.counter}}
                default:
                    return state
            }
        }
        ```
    - `action`和`actionCreators`
        ```
        // action
        const ACTION_INCREMENT = 'INCREMENT'
        const ACTION_DECREMENT = 'DECREMENT'
        // action creator
        const increment = () => ({
            type: ACTION_INCREMENT
        })
        const decrement = () => ({
            type: ACTION_DECREMENT
        })
        ```
    - 链接组件
        ```
        
        // store
        const store = createStore(counter)
        
        class Article extends React.Component {
            render() {
                return (
                    <p>{this.props.counter}</p>
                )
            }
        }
        
        let MyArticle = withRouter(connect((state) => {
            return {
                counter: state.counter
            }
        })(Article))
        
        class App extends React.Component {
            render() {
                return (
                    <div>
                        <Route component={MyArticle}></Route>
                        <button onClick={() => this.props.increment()}>+</button>
                        <button onClick={() => this.props.decrement()}>-</button>
                    </div>
                )
            }
        }
        
        let MyApp = withRouter(connect(() => ({}), (dispatch) => {
            return {
                increment: () => dispatch(increment()),
                decrement: () => dispatch(decrement())
            }
        })(App))
        
        ReactDom.render(
            <Provider store={store}>
                <BrowserRouter>
                    <MyApp/>
                </BrowserRouter>
            </Provider>,
        
            document.getElementById('app')
        )
        ```
- 效果

![clipboard.png](/img/bVbhUbV)
- 说明
    - 主要用到`redux`、`react-redux`库
    - `reducer`、`action`、`actionCreators`都是`redux`用的
    - `Provider`接管`BrowserRouter`，并注入`store`
    - `connect`连接组件和`store`，为组件注入`reducer`
### 0x005 总结
每一步都搞懂，下一步才能更明确。

### 0x006 资源
- [源码](https://github.com/followWinter/react-study)
