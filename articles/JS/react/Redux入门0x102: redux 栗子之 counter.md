### 0x000 概述
这一章讲一些栗子

### 0x001 原生+`redux`实现`counter`
- 修改模板
    ```
    <!doctype html>
    <html>
    <head>
        <title>React Study</title>
    </head>
    <body>
    <div id="app">
        <p id="counter">0</p>
        <button id="btnAdd">增加</button>
        <button id="btnSub">减少</button>
    </div>
    
    </body>
    </html>
    ```
- 修改入口文件
    ```
    import {createStore} from 'redux'
    
    // 声明两个 action
    const ACTION_NUM_INCREMENT = 'ACTION_NUM_INCREMENT'
    const ACTION_NUM_DECREMENT = 'ACTION_NUM_DECREMENT'
    
    // 声明一个 reducer
    const counter = (state = 0, action) => {
        switch (action.type) {
            case ACTION_NUM_INCREMENT: {
                return ++state
            }
            case ACTION_NUM_DECREMENT: {
                return --state
            }
            default : {
                return state
            }
        }
    }
    
    let store = createStore(counter)
    
    // 设置监听, 当 reducer 发生改变的时候获取新的 counter, 并更新界面
    store.subscribe(() => {
        document.getElementById('counter').innerText = store.getState()
    })
    
    // 绑定事件
    document.getElementById('btnAdd').addEventListener('click', () => {
        store.dispatch({type: ACTION_NUM_INCREMENT})
    })
    document.getElementById('btnSub').addEventListener('click', () => {
        store.dispatch({type: ACTION_NUM_DECREMENT})
    })
    
    ```

查看浏览器

![图片描述][1]

说明：`dispatch`其实就是发出一个请求，比如`store.dispatch({type: ACTION_NUM_INCREMENT})`发出的其实是一个`增加`的请求，当然这个只是一个名字而已，如何处理还看我们自己，是一个简单的`js`对象，我们发出这个请求以后，`reducer`将会受到这个请求，`counter`中的`action`对象其实就是你发出的这个东西，经过`switch`处理以后，将会吧`reducer`返回值作为新的`state`保存起来，并通知订阅了`store`的地方--`subscribe`的回调函数将会执行。

### 0x002 使用`ledux`
使用`ledux`实现也很简单上诉的代码只需要该两个地方就行了
- 导入`Ledux`
    ```
    import Ledux from "../../0x101-hello-redux/src/ledux";
    
    ```
- 修改`createStore`为`Ledux.createStore`
    ```
    //redux
    // let store = createStore(counter)
    //ledux
    let store = Ledux.createStore(counter)
    ```
这样就行了

### 0x003 使用`MyEvent`实现
使用`MyEvent`实现无法复用现有的，但是也很简单
```
import MyEvent from "../../0x012-component-communication/src/MyEvent";

// 声明两个 action
const ACTION_NUM_INCREMENT = 'ACTION_NUM_INCREMENT'
const ACTION_NUM_DECREMENT = 'ACTION_NUM_DECREMENT'

let counter = 0
MyEvent.sub(ACTION_NUM_INCREMENT, () => {
    document.getElementById('counter').innerText = counter
})
MyEvent.sub(ACTION_NUM_DECREMENT, () => {
    document.getElementById('counter').innerText = counter
})
// 绑定事件
document.getElementById('btnAdd').addEventListener('click', () => {
    ++counter
    MyEvent.pub(ACTION_NUM_INCREMENT)
})
document.getElementById('btnSub').addEventListener('click', () => {
    --counter
    MyEvent.pub(ACTION_NUM_DECREMENT)
})
```
### 0x004 总结
无
### 0x005 资源
- [源码](https://github.com/followWinter/react-study)

  [1]: /img/bVbgm2P
