### 0x001 概述
这一章讲多个`reducer`怎么处理，并将`ledux`搞成支持支持多个`reducer`
### 0x002 拆封`reducer`
```
import {createStore, combineReducers} from 'redux'

const ACTION_NUM1_INCREMENT = 'ACTION_NUM1_INCREMENT'
const ACTION_NUM2_INCREMENT = 'ACTION_NUM2_INCREMENT'

const num1 = (state = 0, action) => {
    switch (action.type) {
        case ACTION_NUM1_INCREMENT: {
            return ++state
        }
        default: {
            return state
        }
    }
}

const num2 = (state = 0, action) => {
    switch ((action.type)) {
        case ACTION_NUM2_INCREMENT: {
            return ++state
        }
        default: {
            return state
        }
    }
}

const reducer = combineReducers({
    num1: num1,
    num2: num2
})
let store = createStore(reducer)

store.subscribe(() => {
    console.log(store.getState())
})

store.dispatch({type: ACTION_NUM1_INCREMENT})
store.dispatch({type: ACTION_NUM2_INCREMENT})
```
查看浏览器
![clipboard.png](/img/bVbgqOy)

就是这么简单了，核心函数是：`combineReducers(reducers)`，将多个`reducer`并成一个，拆分之后呢，每个`reducer`单独管理一个`state`

### 0x002 改造`ledux`使之支持`combineReducers`
- 添加`combineReducers`并修改`state`计算逻辑
```
class Ledux {
    static createStore(reduer) {
        return new Store(reduer)
    }

    static combineReducers(reducers) {
        return reducers
    }
}

class Store {
    constructor(reducer) {
        this.state = this.calculateState(reducer)
        this.callbacks = []
        this.reducer = reducer
    }

    subscribe(callback) {
        this.callbacks.push(callback)
    }

    getState() {
        return this.state
    }

    dispatch(action) {
        this.state = this.calculateState(this.reducer, action)
        this.callbacks.forEach(callback => callback())
    }

    /**
     * reducer 可能是一个经过 combineReducers 的对象
     * 所以需要判断 reducer 是否是一个对象
     * 如果是
     *  那说明这是一个复合的 reducer
     *  需要循环计算出每个 state
     *  并以对象的形式保存到 state
     * 如果不是对象并且是函数
     *  那说明这是一个单一的 reducer
     *  直接计算就行了
     *  然后保存到 state
     *
     * @param reducer 单一的 reducer 函数或者 combineReducers 之后的对象
     * @param action
     * @returns {*}
     */
    calculateState(reducer, action = {}) {
        if (typeof reducer === 'object') {
            return Object.keys(reducer).map((key) => {
                return {
                    [key]: reducer[key](undefined, action)
                }
            }).reduce((pre, current) => {
                return {...pre, ...current}
            })
        } else if (typeof reducer === 'function') {
            return reducer(undefined, action)
        }
    }
}
/**
 * 添加几个函数导出
 * 保持和 redux 一致的 api
 * 这样就可以不修改调用的函数了
 */
const createStore = Ledux.createStore
const combineReducers = Ledux.combineReducers
export {createStore, combineReducers}
export default Ledux
```
- 修改调用
```
// 直接修改 redux 引入就好了
import {createStore, combineReducers} from './ledux'
```

### 0x003 总结

无

### 0x004 资源
- [源码](https://github.com/followWinter/react-study)
