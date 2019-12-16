---
title: '[ES6]0x012-WeakMap'
date: 2018-11-26 20:11:32
categories: ES6
tags: ES6
---
### 0x000 概述
`WeakMap`和`Map`使用上类似，在特性上和`Set`类似，和`Map`相比，有一下特点
- 不可枚举
- `WeakMap`的`key`只能是对象
- `WeakMap`是弱引用，`WeakMap`内的`key`如果没有引用，将会被垃圾回收机制回收

### 0x001 初始化
```
new WeakMap([[{},1]])
```
### 0x002 添加
```
let weakmap=new WeakMap()
weakmap.add({},"1")
weakmap.add({num:1},()=>{})
```
### 0x003 删除
```
let obj={}
let weakmap=new WeakMap()
weakmap.add(obj,"1")
weakmap.add({},"2")
weakmap.delete(obj) //true
weakmap.delete({}) //false
```
###0x004 包含
```
let obj={}
let weakmap=new WeakMap()
weakmap.add(obj,"1")
weakmap.has(obj)//true
weakmap.has({})//false
```
### 0x005 弱引用特性
```
let weakmap=new WeakMap([[{},1]])
setTimeout(()=>{console.log(weakmap)},3000)
// 3s后输出一下内容，数据消失了
WeakMap {}
```