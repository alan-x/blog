---
title: '[ES6]0x010-Map'
date: 2018-11-26 20:11:30
categories: ES6
tags: ES6
---
### 0x000 概述
`Map`也是一个新的数据结构，在`js`中其实也经常用到，比如下面的栗子，我们经常这么使用一个对象，与其说他是对象，其实他更像一个`Map`，但是比起真正的`Map`，这个还是有点弱了，
```
let color={
    "red":"#FF0000",
    "green":"#00FF00",
    "blue":"#0000FFF"
}
color['red']
```
### 0x001 初始化

```
new Map([iterable])
``` 
初始化一个`Map`有一个可选的参数，该参数必须是一个`可迭代对象`，`可迭代对象`包括`String`、`Array`、`Array-Like obejct(Arguments、NodeList)`、`Typped Array`、`Set`、`Map`和`用户定义的可迭代对象`。
- 数组
    ```
    new Map([[1,2],[3,4]]) // Map(2) {1 => 2, 3 => 4}
    ```
### 0x002 添加
和对象作为`Map`相比，`Map`的键可以是任意值，甚至可以是`NaN`
```
var myMap = new Map();
 
var keyObj = {},
    keyFunc = function () {},
    keyString = "a string";
 
// 添加键
myMap.set(keyString, "和键'a string'关联的值");
myMap.set(keyObj, "和键keyObj关联的值");
myMap.set(keyFunc, "和键keyFunc关联的值");
```
### 0x003 获取`Map`的大小
```
myMap.size    // 3

```
### 0x004 获取
```
myMap.get(keyString)   // "和键'a string'关联的值"
myMap.get(keyObj)     // "和键keyObj关联的值"
myMap.get(keyFunc)      // "和键keyFunc关联的值"
```
### 0x005 是否包含
```
myMap.has(keyString)  // true
myMap.has('1')  // false
```

### 0x006 删除
```
myMap.delete(keyString)  // true
myMap.delete('')  // false
```
### 0x007 遍历
```
myMap.forEach(m=>{console.log(m)})
// 和键'a string'关联的值
//  和键keyObj关联的值
//  和键keyFunc关联的值
```

### 0x008 获取迭代器
```
let entries=myMap.entries()
entries.next().value // 和键'a string'关联的值
entries.next().value//  和键keyObj关联的值
entries.next().value//   和键keyFunc关联的值
```

### 0x009 获取 key 迭代器
```
let keys=myMap.keys()
keys.next().value //  "a string"
keys.next().value//  function () {}
keys.next().value//   {}
```
### 0x010 获取 value 迭代器
```
let values=myMap.values()
values.next().value // 和键'a string'关联的值
values.next().value//  和键keyObj关联的值
values.next().value//   和键keyFunc关联的值
```
### 0x011  清除
```
mySet.clear()
```