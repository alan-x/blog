---
title: '[ES6]0x021-Promise'
date: 2018-11-26 20:11:41
categories: ES6
tags: ES6
---
### 0x000 概述
`Promise`是一个异步编程的解决方案，他经常和`ajax`一起出现，导致很多人以为`Promise`是一种新的网络请求技术，其实不然。`Promise`是一种思考方式、编程方式。

### 0x000 栗子
先写一个栗子
```
setTimeout(()=>{
    console.log('here')
},3000)
```
很简单，3s之后将会打印出`here`，现在换成`Promise`：
```
new Promise((resolve, reject)=>{
    setTimeout(()=>{
        resolve()
    }, 3000)
}).then(()=>{
    console.log('here')
})
```
执行结果也是一样，3s之后将会输出`here`，还可以这么写
```
let proxy=new Promise((resolve, reject)=>{
    setTimeout(()=>{
        resolve()
    }, 3000)
})

setTimeout(()=>{
    proxy.then(()=>{
        console.log('here')
    })
}, 10000)
```
13s后才输出`here`

### 0x002 初始化
- 语法
    ```
    new Promise(executor)
    ```
    - 参数：
        - executor：处理器函数，函数的完整签名是`(resolve, reject)=>{}`
    - 返回值：一个`Promise`实例
- 栗子
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            resolve()
        }, 3000)
    })
    console.log(promise) // Promise {<pending>}
    ```
- 说明：
    使用`new Promise`实例化一个`Promise`之后，将它输出出来，可以看到他有一个`pending`，这是说明这个`promise`的状态，称为`PromiseStatus`，`promise`一共有3种状态，一个`promise`必定处于下面三个状态之一：
    - `pending`：初始状态
    - `fulfilled`：操作成功
    - `rejected`：操作失败
    
### 0x003 `then`
- 语法：
    ```
    promise.then(onFulfilled, onRejected)
    ```
    - `onFulfilled`：操作成功的回调
    - `onRejected`：操作失败的回调
- 栗子1：
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            resolve()
        }, 3000)
    })
    console.log(promise) // Promise {<pending>}
    promise.then(()=>{
        console.log(promise) // Promise {<resolved>: undefined}
    })
    ```
- 说明1：
    当调用`resolve`之后，`then`函数执行了，同时`promise`的`PromiseStatus`变成了`resolved`。`onFulfilled`同时接受一个变量，称之为`PromiseValue`：
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            resolve(1)
        }, 3000)
    })
    promise.then((value)=>{
        console.log(value) // 1
    })
    ```
- 栗子2:
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            reject()
        }, 3000)
    })
    console.log(promise) // Promise {<pending>}
    promise.then(()=>{},()=>{
        console.log(promise) // Promise {<rejected>: undefined}
    })
    ```
    当调用`reject`之后，`then`执行了，此时`promise`的`PromiseStatus`变成了`rejected`，同时，`onRejected`回调接受一个`reason`，作为操作失败的原因说明：
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            reject('nothing')
        }, 3000)
    })
    promise.then(()=>{},(reason)=>{
        console.log(reason) // nothing
    })
    ```
### 0x004 `catch`
- 语法：
    ```
    promise.catch(onRejected)
    ```
    - `onRejected`：回调
- 栗子：
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            resolve()
        }, 3000)
    }).then(()=>{
	    throw 'error'
    }).catch((e)=>{
        console.log(`i catch you: ${e}`) // i catch you error 
    })
    ```
- 注意1：在异中的错误不会执行catch
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            throw 'error'
        }, 3000)
    }).catch((e)=>{
        console.log(`i catch you: ${e}`) // Uncaught error 
    })
    ```
- 注意2：`resolve`之后会被忽略
    ```
     let promise=new Promise((resolve, reject)=>{
        resolve()
        throw 'error'
    }).catch((e)=>{
        console.log(`i catch you: ${e}`)  // 不会输出
    }) 
    
    ```
### 0x005 `finally`
- 语法：
    ```
    p.finally(onFinally)
    ```
    - `onFainally`：回调
- 栗子：
    ```
    let promise=new Promise((resolve, reject)=>{
        setTimeout(()=>{
            resolve()
        }, 3000)
    }).then(()=>{
	    console.log('resolve')
	    throw 'error'
    }).catch((e)=>{
        console.log(`i catch you: ${e}`)
    }).finally(()=>{
        console.log('finally')
    })
    // resolve
    // i catch you error
    // finally
    ```