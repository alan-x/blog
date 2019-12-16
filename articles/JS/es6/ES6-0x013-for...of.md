---
title: '[ES6]0x013-for...of'
date: 2018-11-26 20:11:33
categories: ES6
tags: ES6
---
### 0x000 概述
`for...of`是一个迭代`可迭代对象`的方式，可迭代对象包括`Array`、`Map`、`Set`、`String`、`TypedArray`、`arguments` 对象等等

### 0x001 语法
```
for(variable of iterable){
    // statement
}
```

### 0x001 迭代数组
```
for(let a of [1,2,3]){
	console.log(a)
} 
// 1
// 2
// 3
```

### 0x002 迭代字符串
```
for(let s of 'hello'){
	console.log(s)
}
// h
// e
// l
// l
// o
```

### 0x003 迭代`Set`
```
for(let s of new Set([1,2,3])){
	console.log(s)
}
// 1
// 2
// 3
```

### 0x004 迭代`Map`
```
for(let s of new Map([[1,1],[2,2]])){
	console.log(s)
}
// (2) [1, 1]
// (2) [2, 2]
```

### 0X005 迭代`arguments`
```
(function() {
  for (let argument of arguments) {
    console.log(argument);
  }
})(1, 2, 3);
```

### 0x006 迭代`Dom`集合
```
for(let p of document.getElementsByTagName('p')){
	console.log(p)
}
// <p>...<p>
// <p>...<p>
// <p>...<p>
// <p>...<p>
...
```
### 0x007 总结
`for...of`只能迭代`可迭代对象`