### 0x000 概述
之前写的所有关于`redux`的文章都是纯粹的`redux`，是和框架无关、环境无关的`redux`，所以我没有将`redux`和`react`一起讲，为的是吧`redux`和`react`分开，作为独立的个体来分析，`redux`提现的是一种思想，而不是一个思维定式。而现在我们可以尝试在`react`中来使用了。
### 0x001 `react`集成`redux`栗子-`counter`
- 源码
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

// 组件
class App extends React.Component {
    constructor() {
        super()
        // 初始化 state
        this.state = {
            counter: 0
        }
        // 监听 store 变化, store 变化的时候更新 counter
        store.subscribe(() => {
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

- 效果
![图片描述][1]
### 0x002 原生集成 redux
- `src/index.html`
    ```
    <!doctype html>
    <html>
    <head>
        <title>React Study</title>
    </head>
    <body>
    <div id="app">
        <p id="counter">0</p>
        <button id="incrementBtn">+</button>
        <button id="decrementBtn">-</button>
    </div>
    </body>
    </html>
    ```
- `src/index.js`
    ```
    import {createStore} from 'redux'
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
    
    // 初始化一些 dom 变量
    let counterP = document.getElementById('counter')
    let incrementBtn = document.getElementById('incrementBtn')
    let decrementBtn = document.getElementById('decrementBtn')
    
    // 监听变化, 在 store 变化的时候修改计数器显示
    store.subscribe(() => {
        counterP.innerText = store.getState()
    })
    // 添加点击事件, 当+被点击的时候修改 state
    incrementBtn.onclick = () => {
        store.dispatch(increment())
    }
    // 添加点击事件, 当-被点击的时候修改 state
    decrementBtn.onclick = () => {
        store.dispatch(decrement())
    }
    
    ```
- 效果和上图一致
### 0x003 `vue`集成`redux`
- 源码：
    ```
    <template>
        <div id="app">
            <p>{{counter}}</p>
            <button v-on:click="increment">+</button>
            <button v-on:click="decrement">-</button>
        </div>
    </template>
    
    <script>
        import {createStore} from 'redux'
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
        export default {
            name: 'app',
            data: function () {
                return {
                    counter: 0
                }
            },
            created: function () {
                // 组件创建的时候监听 store 变化, 更新到 data
                this.unSub=store.subscribe(() => {
                    this.counter = store.getState()
                })
            },
            beforeDestroy:function(){
                // 组件销毁的时候取消订阅
                this.unSub()
            },
            methods: {
                increment: function () {
                    store.dispatch(increment())
                },
                decrement: function () {
                    store.dispatch(decrement())
                }
            }
        }
    </script>
    
    <style>
    
    </style>
    
    ```
- 效果和上图一致
### 0x004 总结
`redux`是独立的，就和`react`、`vue`一样都是独立的框架，如何组合他们是我们的选择，而不是必然和唯一的，要让框架成为我们的生产力工具，而不是束缚我们的存在。

### 0x005 资源
- [源码](https://github.com/followWinter/react-study)




  [1]: /img/bVbhgr4
