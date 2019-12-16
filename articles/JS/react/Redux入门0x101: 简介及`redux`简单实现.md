### 0x000 概述
这一章开始讲`redux`，其实是承接前面的`react`，但其实作为一个框架来说，`redux`和`react`并没有太多的关系，本身是独立存在的。在我看来它们的关系不会比共用`re`开头更深了，所以我就重新开了一个头，但其实是基于前面写的...
### 0x001 redux资源
- [中文文档](https://cn.redux.js.org/)
- [英文文档](https://redux.js.org/)
- [官方视频](https://egghead.io/series/getting-started-with-redux)

### 0x002 学习历程
当初为了学习`redux`，看了许多的材料，中途曾经放弃两次，但是最后还是勇敢的拿起了它，现在终于勉强弄懂。
- 第一次是大家都说`redux`牛逼，所以就打算学习一下，看了许多遍了`redux`中文文档，也看了英文文档--因为有些`demo`总是跑的不成功，怕是中文更新不及时的原因。但最后还是不了了之，因为不知道用它来干啥，反倒弄出了一堆的`reducer`、`action`之类的东西。
- 第二次是在做毕业设计的时候，做了一个类似看板一样的`api自动测试及性能统计分析`项目--类似`postman`，在这里遇到了一些超级棘手的问题，其中的最大的问题就是`组件间通信`和`统一的状态管理`。因为是有一个超级复杂的组件操作方式和组件嵌套，在做组件间通信的时候，为了保持状态的一致性，不断的将事件和属性一层一层往下传、或者往上传。真是糟糕的回忆啊....... 当时用的是`Angular2`。而我为了解决这个问题，写了一套[基于订阅-发布模式的事件通知框架](https://github.com/followWinter/event-notify)，原本想解决这些问题，结果，确实解决了`组件间通讯`和`统一状态管理`的问题，但是最终带来的确实更加复杂的`事件管理`和更加无无序的组件关系....
- 后来使用`vue`+`vuex`做了一些项目，反倒是突然领悟的`redux`的思想，就开始了第三次的学习，直到现在，我觉稍微缓解了我在前端项目所受到的伤害，`redux`的文档有点像上帝视角，他好像在说，我知道你应该知道的，所以我这么说你应该懂得，其实我不懂。所以我要从凡人视角说说`redux`，`扒开他的神袍，看看他的胸毛......`



### 0x003 再说一些废话

在前面`react`的文章中，我一直在重复`react`中都是`js`，就是为了把思想引回`react`的本质，`react`是基于`js`写的一个框架而已，只是一个框架，并不是一门独立的语言。很多人总是用了`jQuery`就忘了原生，用了`React`就忘了`jQuery`和原生。而来了`redux`，就必须和`react`绑定。这是一个极大的思想误区，从个体上来说，`vue`是独立的、自由的，`react`也是独立的、自由的，`redux`也是独立的、自由的。

的确有`redux在react中的最佳实践`，却绝对没有`redux的唯一实践`这种说法，在`js`的世界中，各大框架各放异彩，他们既可以兼容并济，也可以相互排斥，不过就是`js`中的一员罢了。每个框架就像每个人一样，有自己的特点，`react`可以构建`ui`，`redux`能够管理状态，`axios`在行网络，`angular`啥都行！你可以在`vue`中使用`redux`，也可以在`vue`中使用`jQuery`，甚至也可在你自己的项目中同时使用`vue`和`react`，语言也好，框架也罢，都是为了向伟大航线进发而服务的。

一句话总结：`自由，然后秩序，接着是世间万物`。

### 0x004 订阅-发布模式
`redux`本质上也是基于`订阅-发布模式`的产物（我记得没错的话，如果记错了，之后回来改），和我写的那个小框架一样.....，所以在使用`redux`之前，先来研究一下这个模式，看看我之前写的那个小东西：
```
let eventMap = {}

class MyEvent {

    /**
     * 发布一个事件并附带一份数据
     *
     * @param name 发布的事件名
     * @param data 附带的数据
     */
    static pub(name, data) {
        if (!eventMap.hasOwnProperty(name)) return
        let callbacks = eventMap[name]
        if (callbacks.length === 0) return
        callbacks.forEach((callback) => {
            callback(data)
        })


    }

    /**
     * 订阅一个事件并附带一个回调
     * 说明这个事件发生的时候所要做的事情
     *
     * @param name 订阅的事件名称
     * @param callback 回调
     * @returns {function(): *} 返回一个函数, 执行这个函数将会取消订阅
     */
    static sub(name, callback) {
        let callbacks = []
        if (eventMap.hasOwnProperty(name)) {
            callbacks = eventMap[name]
        }
        callbacks.push(callback)
        eventMap[name] = callbacks
        return () => callbacks.shift(callback)
    }

}

export default MyEvent
```
这个库一共只有两个接口：
- `pub(name:String,data:data):void`：发布一个事件，这个事件附带一些数据，当这个事件发布的时候，所有订阅这个事件的都将会收到通知，并执行订阅这个事件的时候定义的操作，即回调函数。
- `sub(name:String,callback:Function):Fuction`：订阅一个事件，当这个事件发生的时候，即调用`pub`的时候，该`callback`就会执行，并且在`callback`中可以收到这个事件发生的时候的附带数据。该函数还返回一个新的函数，调用这个函数可以取消订阅该事件

案例：
```
import MyEvent from '../../0x012-component-communication/src/MyEvent'

// 定义一个变量
let num = 1
// 定义一个事件名
const EVENT_INCREMENT = 'EVENT_INCREMENT'

// 订阅这个事件，并将取消订阅的函数保存起来
let unSub = MyEvent.sub(EVENT_INCREMENT, (data) => {
    console.log(data)
})
// 当 num 发生变化的时候，发布这个时间
num += 1
MyEvent.pub(EVENT_INCREMENT, {num: num})

// 当 num 发生变化的时候，发布这个时间
num += 1
MyEvent.pub(EVENT_INCREMENT, {num: num})

// 取消订阅
unSub()

// 当 num 发生变化的时候，发布这个时间
num += 1
MyEvent.pub(EVENT_INCREMENT, {num: num})
// 当 num 发生变化的时候，发布这个时间
num += 1
MyEvent.pub(EVENT_INCREMENT, {num: num})
console.log({num})

```
查看浏览器，可以看到，我们收到了两次通知，因为我们在中途取消了订阅
![clipboard.png](/img/bVbgjAA)

原理就是保存了`MyEvent`中的`eventMap`保存了一个`Map`，该`Map`是一个`String=>Array<Function>`，当我们订阅事件的时候，即调用`sub`的时候，就会形成如下的数据结构：
```
name                 |    callbacks
EVENT_INCRECEMENT    |    (data)=>{...}
-                    |    (data)=>{...}
-                    |    (data)=>{...}
EVENT_DECRECEMENT    |    (data)=>{...}
-                    |    (data)=>{...}
```
当我们调用`pub`的时候，就或寻找到这个事件名，并循环将该事件名下挂载的`callback`队列执行。
当我们调用`unsub`的时候，则会将这个`callback`从队列中移除，这样就不会执行了。

`redux`的原理和这个简陋的框架类似，就是比这精巧多了，不过本质还是一样的，我们可以订阅某个值的变化，然后在某个值变化并收到这个通知的时候作出我们自己的逻辑。

接下来我们会使用`redux`，然后通过`redux`来改造这个我们的`ledux`，打造成至少表面上类似的......

### 0x005 `redux`栗子
```
import {createStore} from 'redux'

// 定义一个 reducer
function counter(state = 0, action) {
    switch (action.type) {
        case 'INCREMENT':
            return state + 1
        case 'DECREMENT':
            return state - 1
        default:
            return state
    }
}

// 定义一个store，持有全局的 state
let store = createStore(counter)

// 订阅，并输出 state
store.subscribe(() =>
    console.log(store.getState())
)
// 发布一个事件
store.dispatch({type: 'INCREMENT'})
store.dispatch({type: 'INCREMENT'})
store.dispatch({type: 'DECREMENT'})
```
查看浏览器
![clipboard.png](/img/bVbgjC5)

可以看到，其实模式差不多：
- 定义数据：
    - `redux`:`reducer` 部分
    - `MyEvent`:`let num`部分
- 订阅：
    - `redux`:`subscribe` 部分
    - `MyEvent`:`sub`部分
- 发布：
    - `redux`:`dispatch` 部分
    - `MyEvent`:`pub`部分
但是其实内部差别又非常大，在`MyEvent`中，都是很随意的，我可以随便定义事件，随便修改数据，随便发布事件，所以我写的东西叫做`事件通知`，而不是状态管理。`redux`是用来做状态管理的，可以说是在`事件通知`之中再一次做了封装。
- 在`MyEvent`中数据是可以随意修改的，但是在`redux`中，数据的修改只能通过`dispatch`提交一个修改请求，而在`reducer`中处理这个请求。

### 0x006 `ledux`实现：
- 编写`redux`
```
class Ledux {
    static createStore(reduer) {
        return new Store(reduer)
    }
}

class Store {
    constructor(reducer) {
        this.state = reducer(null, {})
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
        this.state = this.reducer(this.state, action)
        this.callbacks.forEach(callback => callback())
    }

}

export default Ledux
```
- 使用`Ledix`
```

import Ledux from "./ledux";

function counter(state = 0, action) {
    switch (action.type) {
        case 'INCREMENT':
            return state + 1
        case 'DECREMENT':
            return state - 1
        default:
            return state
    }
}
let store=Ledux.createStore(counter)

store.subscribe(()=>{
    console.log(store.getState())
})

store.dispatch({type: 'INCREMENT'})
store.dispatch({type: 'INCREMENT'})
store.dispatch({type: 'DECREMENT'})
```
- 查看浏览器

![clipboard.png](/img/bVbgjFw)

可以看到，效果是一样的，而过程除了我喜欢用`class`封装以外也没有太大的区别，就这样实现了`ledux`了，当然随着对`redux`的`api`的深入学习，这个框架也可以不断的深入发展。

### 0x007 资源

- [源码](https://github.com/followWinter/react-study)
