### 0x000 概述
前面虽然简单的讲了如何在`react`中集成`redux`，但是那只是简单的讲讲而已，这一章将会仔细讲讲如何在`react`中使用`reudx`
### 0x001 问题分析
查看前边的栗子：
```
import {createStore} from 'redux'
import React from 'react'
import ReactDom from 'react-dom'

//reducer
const counter = (state = 0, action) => {
    switch (action.type) {
        case ACTION_INCREMENT:
            return state + 1
        case ACTION_DECREMENT:
            return state - 1
        default:
            return state
    }
}
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

// store
const store = createStore(counter)

// react
// // 组件
class App extends React.Component {
    constructor() {
        super()
        // 初始化 state
        this.state = {
            counter: 0
        }
        // 监听 store 变化, store 变化的时候更新 counter
        this.unSub=store.subscribe(() => {
            this.setState({
                counter: store.getState()
            })
        })
    }
    // 组件销毁的时候取消订阅
    componentWillUnmount(){
        this.unSub()
    }

    render() {
        return <div>
            <p>{this.state.counter}</p>
            <button
                onClick={() => {
                    store.dispatch(increment())
                }}>+
            </button>
            <button
                onClick={() => {
                    store.dispatch(decrement())
                }}>-
            </button>
        </div>
    }
}

ReactDom.render(
    <App/>,
    document.getElementById('app')
)
```
为了让组件可以响应`redux`的变化，我们写了一些代码：
```
    ....
    // 监听 store 变化, store 变化的时候更新 counter
    this.unSub=store.subscribe(() => {
        this.setState({
                counter: store.getState()
            })
        })
    ....
    // 组件销毁的时候取消订阅
    componentWillUnmount(){
        this.unSub()
    }
```
如果我们有大量的组件需要绑定`redux`，那么写这些代码就显得非常冗余了
这一章要做的事就是优化掉这个东西

### 0x002 `connect`方法
> 这里用了一个`react`的`HOC`，参数是一个组件，返回值也是一个组件，但是返回的组件被添加了一个`props`，也就是`state`。`connect`方法为每个组件添加了响应`store`数据变化的能力，在`store.dispatch`调用的时候，会修改组件的`props`，让组件重绘，从而达到`react`组件和`redux`绑定但是又不需要写太多样板代码
- `connect`
    ```
    const connect = (WrappedComponent) => {
        return class Control extends React.Component {
            constructor() {
                super()
                this.state = {
                    state: 0
                }
                this.unSub = store.subscribe(() => {
                    this.setState({
                        state: store.getState()
                    })
                })
            }
    
            componentWillUnmount() {
                this.unSub()
            }
    
            render() {
                return <WrappedComponent state={this.state.state}/>
            }
    
    
        }
    }
    ```
- 完整源码
    ```
    import {createStore} from 'redux'
    import React from 'react'
    import ReactDom from 'react-dom'
    
    //reducer
    const counter = (state = 0, action) => {
        switch (action.type) {
            case ACTION_INCREMENT:
                return state + 1
            case ACTION_DECREMENT:
                return state - 1
            default:
                return state
        }
    }
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
    
    // store
    const store = createStore(counter)
    
    const connect = (WrappedComponent) => {
        return class Control extends React.Component {
            constructor() {
                super()
                this.state = {
                    state: 0
                }
                this.unSub = store.subscribe(() => {
                    this.setState({
                        state: store.getState()
                    })
                })
            }
    
            componentWillUnmount() {
                this.unSub()
            }
    
            render() {
                return <WrappedComponent state={this.state.state}/>
            }
    
    
        }
    }
    // 子组件
    class SubCom extends React.Component {
        render(){
            return <p>{this.props.state}</p>
        }
    }
    // 包裹这个组件
    let ReduxSubCom=connect(SubCom)
    
    // react 组件
    class App extends React.Component {
        constructor() {
            super()
        }
    
        render() {
            return <div>
                <ReduxSubCom/>
                <button
                    onClick={() => {
                        store.dispatch(increment())
                    }}>+
                </button>
                <button
                    onClick={() => {
                        store.dispatch(decrement())
                    }}>-
                </button>
            </div>
        }
    }
    // 包裹组件
    let ReduxApp = connect(App)
    
    ReactDom.render(
        <ReduxApp/>,
        document.getElementById('app')
    )
    ```
### 0x003 加强`connect`方法，消除订阅整个`state`树的影响
> 虽然已经实现了将`state`和组件绑定，但是我们绑定的是整个`state`，如果`state`树很大并且组件很多，那这个无畏的性能消耗太凶了。
- 修改`redux`结构
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
- 修改`connect`方法，返回一个函数，并修改`props`传参：
```
const connect = (mapStateToProps) => {
    return (WrappedComponent) => class Control extends React.Component {
        constructor() {
            super()
            this.state = {
                state: {}
            }
            this.unSub = store.subscribe(() => {
                let state = mapStateToProps(store.getState())
                this.setState({
                    state: state
                })
            })
        }
        componentWillUnmount() {
            this.unSub()
        }
        render() {
            return <WrappedComponent {...this.state.state}/>
        }
    }
}
```
- 修改`APP`组件中的`props`访问方式
```
class App extends React.Component {
    constructor() {
        super()
    }
    componentWillReceiveProps(nextProps) {
        console.log(nextProps)
    }
    render() {
        return <div>
            <p>{this.props.counter}</p>
            <button
                onClick={() => {
                    store.dispatch(increment())
                }}>+
            </button>
            <button
                onClick={() => {
                    store.dispatch(decrement())
                }}>-
            </button>
        </div>
    }
}
```
- 修改`connect`调用
```
let ReduxApp = connect((state) => {
    return {
        counter: state.counter
    }
})(App)
```
### 0x004: 加强`connect`，让代码中不再调用`store.dispatch`，不在依赖`redux`
- 修改`connect`方法，除了吧`state`映射到`props`上，也把`dispatch`给映射上去了，这样组件就感受不到`redux`的存在了
    ```
    const connect = (mapStateToProps, mapDispatchToProps) => {
    
        return (WrappedComponent) => class Control extends React.Component {
            constructor() {
                super()
                // 第一次初始化
                let props = mapStateToProps(store.getState())
                let actions = mapDispatchToProps(store.dispatch)
                this.state = {
                    props: {...props,...actions}
                }
    
                this.unSub = store.subscribe(() => {
                    // 变化的时候再次计算
                    let props = mapStateToProps(store.getState())
                    let actions = mapDispatchToProps(store.dispatch)
                    this.setState({
                        props: {...props,...actions}
                    })
                })
            }
    
            componentWillUnmount() {
                this.unSub()
            }
    
            render() {
                return <WrappedComponent {...this.state.props}/>
            }
        }
    }
    
    ```
- 修改`connect`调用，将`dispatch`映射到组件上
    ```
    let ReduxApp = connect(
        (state) => {
            return {
                counter: state.counter
            }
        },
        (dispatch) => {
            return {
                increment: () => dispatch(increment()),
                decrement: () => dispatch(decrement()),
            }
        }
    )(App)
    ```
- 修改`App`组件，不再使用`store.dispatch`，而是使用`connect`传递过来的`dispatch`，让组件不依赖`redux`
    ```
    class App extends React.Component {
        constructor(props) {
            super()
        }
    
        render() {
            const {counter,increment,decrement}=this.props
            return <div>
                <p>{counter}</p>
                <button
                    onClick={increment}>+
                </button>
                <button
                    onClick={decrement}>-
                </button>
            </div>
        }
    }
    ```
- 完整源码
    ```
    import {createStore} from 'redux'
    import React from 'react'
    import ReactDom from 'react-dom'
    
    //reducer
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
    
    // store
    const store = createStore(counter)
    
    const connect = (mapStateToProps, mapDispatchToProps) => {
    
        return (WrappedComponent) => class Control extends React.Component {
            constructor() {
                super()
                // 第一次初始化
                let props = mapStateToProps(store.getState())
                let actions = mapDispatchToProps(store.dispatch)
                this.state = {
                    props: {...props,...actions}
                }
    
                this.unSub = store.subscribe(() => {
                    // 变化的时候再次计算
                    let props = mapStateToProps(store.getState())
                    let actions = mapDispatchToProps(store.dispatch)
                    this.setState({
                        props: {...props,...actions}
                    })
                })
            }
    
            componentWillUnmount() {
                this.unSub()
            }
    
            render() {
                return <WrappedComponent {...this.state.props}/>
            }
        }
    }
    
    // react 组件
    class App extends React.Component {
        constructor(props) {
            super()
        }
    
        render() {
            const {counter,increment,decrement}=this.props
            return <div>
                <p>{counter}</p>
                <button
                    onClick={increment}>+
                </button>
                <button
                    onClick={decrement}>-
                </button>
            </div>
        }
    }
    
    let ReduxApp = connect(
        (state) => {
            return {
                counter: state.counter
            }
        },
        (dispatch) => {
            return {
                increment: () => dispatch(increment()),
                decrement: () => dispatch(decrement()),
            }
        }
    )(App)
    
    ReactDom.render(
        <ReduxApp/>,
        document.getElementById('app')
    )
    ```
### 0x004 `reat-redux`
以上效果就和上一章的效果一致，是一个`counter`，在这里我们一步一步去除了样板代码，并将`redux`从组件中移除，如果只看`App`组件，根本感觉不到`redux`的存在，但`redux`又确实存在，如果有一天你要去掉`redux`，就可以做到不影响组件了。这里的`connect`其实不需要自己写，已经有好的实现了：[react-redux][1]
    ```
    // 引入`react-redux`
    import {Provider, connect} from 'react-redux'
    // 修改组件
    ReactDom.render(
        <Provider store={store}>
            <ReduxApp/>
        </Provider>,
        document.getElementById('app')
    )
    ```
- 完整源码
    ```
    import {createStore} from 'redux'
    import React from 'react'
    import ReactDom from 'react-dom'
    import {Provider, connect} from 'react-redux'
    //reducer
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
    
    // store
    const store = createStore(counter)
    
    
    // react 组件
    class App extends React.Component {
        constructor(props) {
            super()
        }
    
        render() {
            const {counter, increment, decrement} = this.props
            return <div>
                <p>{counter}</p>
                <button
                    onClick={increment}>+
                </button>
                <button
                    onClick={decrement}>-
                </button>
            </div>
        }
    }
    
    let ReduxApp = connect(
        (state) => {
            return {
                counter: state.counter
            }
        },
        (dispatch) => {
            return {
                increment: () => dispatch(increment()),
                decrement: () => dispatch(decrement()),
            }
        }
    )(App)
    
    ReactDom.render(
        <Provider store={store}>
            <ReduxApp/>
        </Provider>,
        document.getElementById('app')
    )
    ```
### 0x005 资源
- [源码](https://github.com/followWinter/react-study)


  [1]: https://github.com/reduxjs/react-redux
