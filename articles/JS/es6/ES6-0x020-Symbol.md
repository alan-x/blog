---
title: '[ES6]0x020-Symbol'
date: 2018-11-26 20:11:40
categories: ES6
tags: ES6
---
### 0x000 概述
`Symbol`是`es6`新的基本数据类型，所以`es`之后的数据类型如下：
- 基本数据类型：
    - `Boolean`
    - `Null`
    - `Undefined`
    - `Number`
    - `String`
    - `Symbol`
- 引用类型
    - `Object`
### 0x001 `Symbol()`
- 语法
    ```
    Symbol([description])
    ```
    - `description`：描述，可选字符串
- 例子
    ```
    Symbol()
    Symbol(1)
    Symbol('string')
    ```
- 说明：
    使用`Symbol()`初始化的变量是完全不同的两个变量，`description`只是一个描述而已，没有任何意义。
    ```
    Symbol(1)===Symbol(1) // false
    ```
    可以使用`typeof`来判断`Symbol`类型
    ```
    typeof Symbol('1')
    // "symbol"
    ```
    
### 0x002 `Symbol.for()`
- 语法：
    ```
    Symbol.for(key);
    ```
    - `key`：与该`Symbol`相关连的一个名字，可以通过这个名字获取`Symbol`实例。
- 例子：
    ```
    Symbol.for(1)
    Symbol.for('string')
    ```
- 说明：
    和`Symbol()`实例化的`Symbol`实例不同，使用`Symbol.for()`实例化的实例在全局保存，相同的两个`key`返回的`Symbol`实例是一样的。也就是说，使用`Symbol.for(key)`实例化一个`Symbol`数据类型的时候，如果全局不存在这个`key`对应的`symbol`，则全局创建一个`key`对应的`symbol`，如果全局存在，则直接返回这个`key`对应的`Symbol`。
    ```
    Symbol.for('string')===Symbol.for('string')
    // true
    ```

### 0x003 `Symbol.keyFor(key)`
- 语法
    ```
    Symbol.keyFor(sym);
    ```
    - 参数：
        - `sym`：`Symbal`实例
    - 返回值：
        - `string`：返回这个`Symbol`实例的`key`
- 例子
    ```
    let sym=Symbol.for('string')
    console.log(Symbol.keyFor(sym)) // 'string'
    ```
- 说明
    `Symbol(description)`的`description`和`Symbol.for(key)`的`key`是不一样的，`description`只是一个描述，除了调试没有任何实际用途，无法通过`description`获取这个`Symbol`实例：
    ```
    let sym= Symbol('sss')
    Symbol.for('sss')===sym  // false
    Symbol.keyFor(sym) // undefined 
    ```
    所以，`Symbol.keyFor`也只能获取`Symbol.for`实例化的`Symbol`的`key`。
    
    
    