### 0x000 概述
写长文章有点累啊，偶尔写点短的文章吧
### 0x001 概念
其实很多框架在技术上没有太大的难度，真正难的是思想，思想的突破远远比技术突破难多了。`redux`就为我带来了一种应用状态管理的新思想，其间充斥着许多个概念，`state`、`reduce`等，乍一看头大，等到仔细理解了它的思想，或许就很简单了，`Action Creators`也是其中一种。

### 0x002 栗子
看看前面我们是怎么使用`redux`的:
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
    console.log(store.getState())
})

store.dispatch({type: 'INCREMENT'})

```
我们使用`store.dispatch({type: 'INCREMENT'})`发出了一个`action`，但是这种方式实在是太不优雅了，存在两个问题：
1. 容易产生重复代码，特别是复杂的`action`
2. 使用字符串作为`type`容易出错，重构也不易（200个地方使用了`INCREMENT`，有一天突然要修改，那就gg了）
对于第二个问题，解决很简单，使用常量就好了
```
const ACTION_INCREMENT='INCREMENT'
store.dispatch({type: ACTION_INCREMENT})
```
第一个问题，也很简单，使用函数分装起来就好了
```
const increment = () => {
    return {
        type: ACTION_INCREMENT
    }
}

store.dispatch(increment())
```
这个封装起来的函数就是`Action Creator`了，这种做法就是用函数封装了一下而已。但是在维护性上却有很大提升：
1. 不需要重复的写`{....}`，防止大的`aciton`写到吐
2. 动态变化时候隐藏细节，异步`action`、数据拼装...
```
const increment = (step) => {
    return {
        type: ACTION_INCREMENT,
        step:step
    }
}

store.dispatch(increment(5))
```
### 0x003 总结

在学习一个东西了时候，可以通过实践来理解概念，纯粹的概念很让人头疼啊。

### 0x004 资源
- [源码](https://github.com/followWinter/react-study)


