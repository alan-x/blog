---
title: '[ES6]0x009-Set'
date: 2018-11-26 20:11:29
categories: ES6
tags: ES6
---
### 0x000 概述
`Set`是一个新的数据结构，和其他语言的特性差不多，当然，作为`js`中的`Set`，他还是有一些属于`js`的特点。

### 0x001 初始化
```
new Set([iterable]);
```
初始化一个`Set`有一个可选的参数，这个参数必须是一个`可迭代的对象`，`可迭代对象`包括`String`、`Array`、`Array-Like obejct(Arguments、NodeList)`、`Typped Array`、`Set`、`Map`和`用户定义的可迭代对象`

- 字符串
    ```
    new Set('1234') // Set(4) {"1", "2", "3", "4"}
    ```
- 数组
    ```
    new Set([1,2,3,4]) // Set(4) {1, 2, 3, 4}
    ```
- arguments
    ```
    function sum(){
      return new Set(arguments)
    }
    sum(1,2,3,4)  // Set(4) {1, 2, 3, 4}
    ```
- Set
    ```
    new Set(new Set([1,2,3,4])) // Set(4) {1, 2, 3, 4}
    ```

### 0x002 添加值
初始化一个`Set`之后，可以继续往里面添加值
```
let set=new Set()
set.add(1)
set.add(1)
set.add(1)
console.log(set) // Set(1) {1}
```
借用这个特性可以搞很多事，比如过滤重复值
```
new Set([1,1,2,3]) // Set(3){1,2,3}
```
但是注意，两个相同的对象字面量是不同的对象，具有不同的引用，所以`Set`是无法将两个不同引用的对象标记为同一个的，即使他们看过去是一样的
```
let a={num:1}
let b={num:1}
console.log(a===b) //false
new Set(a, b)// Set(2){{num:1},{num:2}}
let c=a
console.log(c===a)//true
new Set(a,c)// Set(1){{num:1}}
```
### 0x003 判断是否包含

```
let set=new Set([1,2,3])
set.has(1) // true
set.has(4) //false
```
### 0x004 获取数量
```
let set=new Set([1,2,3])
set.size //3
```

### 0x005 删除
```
let set=new Set([1,2,3])
set.delete(1)// true
set.delete(1)// false
```
### 0x006 清空
```
let set=new Set([1,2,3])
set.clear()
console.log(set) // Set(0){}
```
### 0x007 遍历
```
let set=new Set([1,2,3])
set.forEach((s)=>{console.log(s)})
// 1
// 2
// 3
```
### 0x008 获取迭代器
```
let set=new Set([1,2,3])
let entries=set.entries()
console.log(entries.next().value) // [1,1]
console.log(entries.next().value) // [2,2]
console.log(entries.next().value) // [3,3]
console.log(entries.next().value) // undefined

for(let item of set){
    console.log(item)
}
// 1
// 2
// 3
```

### 0x009 获取键迭代器
```
let set=new Set([1,2,3])
let keys=set.keys()
console.log(keys.next().value) // 1
console.log(keys.next().value) // 2
console.log(keys.next().value) // 3
console.log(keys.next().value) // undefined

for(let {key} of set){
    console.log(key)
}
// 1
// 2
// 3
```

### 0x010 获取值迭代器
```
let set=new Set([1,2,3])
let values=set.values()
console.log(values.next().value) // 1
console.log(values.next().value) // 2
console.log(values.next().value) // 3
console.log(values.next().value) // undefined

for(let {value} of set){
    console.log(value)
}
// 1
// 2
// 3
```