### 0x000 概述
前一章讲了`redux`的`Action Creator`，这一章讲`redux`中很神奇的中间件。
### 0x001 手写中间件
在项目中，我们经常会有记录一些事件或者在某些事件发生的时候做某些事的需求，比如`api`接口鉴权操作、日志记录操作等，一般我们都可以用中间件来完成，中间件具有可拔插、可扩展的特点。我们也可以使用中间件来扩展`redux`的功能。

- 记录每一次的`action`和`state`变化
    我们在之前是这么使用`redux`的：
    ```
    import {createStore} from 'redux'

    function counter(state = 0, action) {
        switch (action.type) {
            case 'INCREMENT':
                return state + 1
            default:
                return state
        }
    }
    
    let store = createStore(counter)
    
    store.subscribe(() => {
        // console.log(store.getState())
    })
    
    const ACTION_INCREMENT = 'INCREMENT'
    
    const increment = () => {
        return {
            type: ACTION_INCREMENT
        }
    }
    const action = increment()
    store.dispatch(action)
    store.dispatch(action)
    store.dispatch(action)

    ```
    通过`store.dispatch`完成对数据的修改，现在我们希望记录每一次对数据的修改，我们可以这么做
    ```
    console.log('action', action.type)
    store.dispatch(action)
    console.log('next state', store.getState())
    
    console.log('action', action.type)
    store.dispatch(action)
    console.log('next state', store.getState())
    
    console.log('action', action.type)
    store.dispatch(action)
    console.log('next state', store.getState())
    ```
    效果很明显，猪才会这么做
    ![clipboard.png](/img/bVbhfOE)
    
    封装1：封装成函数，可以，经常我们也是这么做的，但是不好
    ```
    const dispatch = (store, action) => {
        console.log('action', action.type)
        store.dispatch(action)
        console.log('next state', store.getState())
    }
    
    dispatch(store, action)
    ```
    封装2：`hack`掉`store.dispatch`，没毛病，但是不够优雅并且如果希望有多个中间件不太好办，并且希望中间键可以串联起来
    ```
    const storeDispatch=store.dispatch
    store.dispatch= (action) => {
        console.log('action', action.type)
        storeDispatch(action)
        console.log('next state', store.getState())
    }
    store.dispatch(action)
    ```
    - 封装3：多个中间件串联
        > 这里写了两个中间键，一个是前中间件，一个是后中间件，在执行` before(store)`的时候，其实我们已经将`store.dispatch`替换成了`before`的`dispatch`，所以我们在`after`对`dispatch`第二次替换的时候，`const storeDispatch = store.dispatch`中的` store.dispatch`其实是`before.dispatch`，所以，当我们执行`store.dispatch(increment())`的时候，调用链其实是：`store#dispatch=after#dispatch -> before#dispatch -> before#console.log -> store#dispatch -> after#console.log`
        ```
        const before = (store) => {
            const storeDispatch = store.dispatch
            const dispatch=(action) => {
                console.log('before', action.type,store.getState())
                storeDispatch(action)
            }
            store.dispatch = dispatch
        }
        
        const after = (store) => {
            const storeDispatch = store.dispatch
            const dispatch = (action) => {
                storeDispatch(action)
                console.log('after',action.type,store.getState())
            }
            store.dispatch=dispatch
        }
        before(store)
        after(store)
        store.dispatch(increment())
        ```
        查看输出：
        
        ![clipboard.png](/img/bVbhfVR)
    - 封装4：隐藏`hack`，减少样板代码
        ```
        
        const before = (store) => {
            const storeDispatch = store.dispatch
            return (action) => {
                console.log('before', action.type, store.getState())
                storeDispatch(action)
            }
        }
        
        const after = (store) => {
            const storeDispatch = store.dispatch
            return (action) => {
                storeDispatch(action)
                console.log('after', action.type, store.getState())
            }
        }
        
        const applyMiddleware = (store, ...middlewares) => {
            middlewares.reverse()
            middlewares.forEach(middleware => {
                store.dispatch = middleware(store)
            })
        }
        
        applyMiddleware(store, before, after)
        
        store.dispatch(increment())
        ```
    - 封装5：不使用 hack
        ```
        const before = (store) => {
            return (storeDispatch) => {
                return (action) => {
                    console.log('before', action.type, store.getState())
                    storeDispatch(action)
                }
            }
        }
        const after = (store) => {
            return (storeDispatch) => {
                return (action) => {
                    storeDispatch(action)
                    console.log('after', action.type, store.getState())
                }
            }
        }
        
        const applyMiddleware = (store, ...middlewares) => {
            middlewares.reverse()
            let storeDispatch = store.dispatch
            middlewares.forEach(middleware => {
                storeDispatch = middleware(store)(storeDispatch)
            })
            // store.dispatch = storeDispatch
            return {...store, ...{dispatch: storeDispatch}}
        }
        
        store = applyMiddleware(store, before, after)
        
        store.dispatch(increment())
        ```
    - 封装6：优化中间件写法
        ```
        const before = store => storeDispatch => action => {
            console.log('before', action.type, store.getState())
            return storeDispatch(action)
        }
        const after = store => storeDispatch => action => {
            let result = storeDispatch(action)
            console.log('after', action.type, store.getState())
            return result
        }
        ```
    - 最终的完整代码
        ```
        import {createStore} from 'redux'
        
        // reducer
        function counter(state = 0, action) {
            switch (action.type) {
                case 'INCREMENT':
                    return state + 1
                default:
                    return state
            }
        }
        // 创建 store
        let store = createStore(counter)
        
        // action
        const ACTION_INCREMENT = 'INCREMENT'
        
        // action creator
        const increment = () => {
            return {
                type: ACTION_INCREMENT
            }
        }
        
        // 前中间件
        const before = store => storeDispatch => action => {
            console.log('before', action.type, store.getState())
            return storeDispatch(action)
        }
        
        // 后中间件
        const after = store => storeDispatch => action => {
            let result = storeDispatch(action)
            console.log('after', action.type, store.getState())
            return result
        }
        
        // 应用中间件
        const applyMiddleware = (store, ...middlewares) => {
            middlewares.reverse()
            let storeDispatch = store.dispatch
            middlewares.forEach(middleware => {
                storeDispatch = middleware(store)(storeDispatch)
            })
            // store.dispatch = storeDispatch
            return {...store, ...{dispatch: storeDispatch}}
        }
        
        // 返回了新的 store
        store = applyMiddleware(store, before, after)
        
        // 发出 action 
        store.dispatch(increment())
        ```
### 0x002 `redux applyMiddleware`
> 前面写了一个`applyMiddleware`方法，虽然可以用，但是官方其实也提供了这个方法，并且比我们写的更好一点
    ```
    const before = store => storeDispatch => action => {
        console.log('before', action.type, store.getState())
        return storeDispatch(action)
    }
    const after = store => storeDispatch => action => {
        let result = storeDispatch(action)
        console.log('after', action.type, store.getState())
        return result
    }
    let store = createStore(counter, applyMiddleware(before, after))
    store.dispatch(increment())
    ```
    可以看出来，相较于我们自己写的`applyMiddleware`，官方提供的可以直接传递给`createStore`，而无需在次对`store`进行操作。
### 0x003 异步`action`
```
const before = store => storeDispatch => action => {
    console.log('before', action.type, store.getState())
    return storeDispatch(action)
}
const after = store => storeDispatch => action => {
    let result = storeDispatch(action)
    console.log('after', action.type, store.getState())
    return result
}
const asyncAction=()=>{
    return (dispatch)=>{
        setInterval(()=>{
            dispatch(increment())
        },1000)
    }
}
const asyncMiddleware = store => storeDispatch => action => {
    if (typeof action === "function") {
        return action(storeDispatch)
    } else {
        return storeDispatch(action)
    }
}
let store = createStore(counter, applyMiddleware(asyncMiddleware,before,after))
store.dispatch(asyncAction())
```
这里写了一个`asyncMiddleware`，他判断传入的`action`是否是一个函数，如果是一个函数，那就直接执行这个函数，同时将`dispatch`作为参数，则在`asyncAction`我们就能直接访问到`dispatch`了，就可以在`asyncAction`适当的时候再次`dispatch`了。
当然一样的，这样的中间件也已经存在了：[redux-thunk][1]，日志的中间件也已经存在了：[redux-logger][2]
```
import thunkMiddleware from 'redux-thunk'
import { createLogger } from 'redux-logger'

const store = createStore(
    counter,
    applyMiddleware(
        thunkMiddleware,
        createLogger()
    )
)
store.dispatch(increment())
```
查看效果

![clipboard.png](/img/bVbhgjp)

### 0x004 资源
- [源码](https://github.com/followWinter/react-study)


  [1]: https://github.com/reduxjs/redux-thunk
  [2]: https://github.com/evgenyrodionov/redux-logger
