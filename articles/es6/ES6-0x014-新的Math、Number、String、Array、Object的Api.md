---
title: '[ES6]0x014-新的Math、Number、String、Array、Object的Api'
date: 2018-11-26 20:11:34
categories: ES6
tags: ES6
---
### 0x001 Math
`Math`更新了几个方法，但是一般情况下没有太大的用处
- 反双曲线函数，返回一个数字的反双曲余弦值
    ```
    Math.acosh(-1);  // NaN
    Math.acosh(0);   // NaN
    Math.acosh(0.5); // NaN
    Math.acosh(1);   // 0
    Math.acosh(2);   // 1.3169578969248166
    ```
- 算数平方根函数，返回所有参数的算术平方根
    ```
    Math.hypot(3, 4)        // 5
    Math.hypot(3, 4, 5)     // 7.0710678118654755
    Math.hypot()            // 0
    Math.hypot(NaN)         // NaN
    Math.hypot(3, 4, "foo") // NaN, +"foo" => NaN
    Math.hypot(3, 4, "5")   // 7.0710678118654755, +"5" => 5
    Math.hypot(-3)          // 3, the same as Math.abs(-3)
    ```
- 类C的32位整数乘法运算函数
    ```
    Math.imul(2, 4);          // 8
    Math.imul(-1, 8);         // -8
    Math.imul(-2, -2);        // 4
    Math.imul(0xffffffff, 5); // -5
    Math.imul(0xfffffffe, 5); // -10
    ```
### 0x002 Number
- `Number.EPSILON`
    该常量表示`1`与`Number`可表示的大于`1`的最小的浮点数之间的差值，那有什么用呢？可以用来解决`浮点数`的比较问题
    ```
    x = 0.2;
    y = 0.3;
    z = 0.1;
    equal = (Math.abs(x - y + z) < Number.EPSILON); // true 
    ```
- `Number.isInteger`
    该函数接受一个参数，如果该参数是整数，则返回`true`，否则返回`false`，`NaN`、`+Infinity`、`-Infinity`不是整数
    ```
    Number.isInteger(0);         // true
    Number.isInteger(1);         // true
    Number.isInteger(-100000);   // true
    
    Number.isInteger(0.1);       // false
    Number.isInteger(Math.PI);   // false
    
    Number.isInteger(Infinity);  // false
    Number.isInteger(-Infinity); // false
    Number.isInteger("10");      // false
    Number.isInteger(true);      // false
    Number.isInteger(false);     // false
    Number.isInteger([1]);       // false
    ```
### 0x003 String
- `String.protorype.includes(searchString[, position])`
    判断字符串是否包含子串，该函数有两个参数，返回值为`boolean`
    - `searchString`：要搜索的子串
    - `position`：可选的起始索引位置，默认就是0
    ```
    `123456`.includes(1) // true
    `123456`.includes(1, 2) // false
    `123456`.includes(7) // true
    ```
- `String.protorype.repeat(count)`
    将一个字符串重复多次
    - `count`：重复的次数
    ```
    `12`.repeat(10) // "12121212121212121212"
    `12`.repeat(-10) // Uncaught RangeError: Invalid count value
    ```
### 0x004 Array
- `Array.from(arrayLike[, mapFn[, thisArg]])`
    该函数可以从一个伪数组对象或者可迭代对象中创建一个数组。
    - `arrayLike`：目标对象
    - `mapFn`：`arrayLike`到数组的映射方式
    - `thisArg`：映射函数中的`this`
    ```
    Array.from('123') //[1,2,3]
    Array.from([1,2,3]) //[1,2,3]
    Array.from(new Set([1,2,3])) //[1,2,3]
    Array.from(new Map([[1,2],[3,4],[5,6]])) // [[1,2],[3,4],[5,6]]
    Array.from('123',n=>n*2})// [2,4,6]
    
    function func(){
        return Array.from(arguments)
    }
    func(1,2,3,4) // [1,2,3,4]
    ```
- `Array.of(element0[, element1[, ...[, elementN]]])`
    根据给的元素返回包含这些元素的数组
    ```
    Array.of(1) // [1]
    Array.of(1,2,3,4,5) // [1,2,3,4,5]
    ```
- `Array.fill(value[, start[, end]])`
    用指定元素填充数组
    - `value`：要填充的元素
    - `start`：开始填充的位置
    - `end`：借宿填充的位置
    ```
    [1,2,3].fill(1,1)//[1,1,1]
    [1,2,3].fill(1,2,1)//[1,1,1]
    ```
- `Array.findIndex(callback[, thisArg])`
    查找指定元素的索引
    - `callback`：指定命中的方式
    - `thisArg`：`callback`中的`this`
    ```
    [1,2,3].find(n=>n===2) // 1
    [1,2,3].find(n=>n===8) // -1
    ```
- `Array.entries()`
    获取数组迭代器
    ```
    let entries=[1,2,3].entries()
    for(let e of entries){
        console.log(e)
    }
    // (2)[0,1]
    // (2)[1,2]
    // (2)[2,3]
    ```

- `Array.keys()`
    获取数组的键迭代器
    ```
    let keys=[1,2,3].keys()
    for(let k of keys){
        console.log(k)
    }
    // 0
    // 1
    // 2
    ```
- `Array.values`
    获取数组的值迭代器
    ```
    let values=[1,2,3].values()
    for(let v of values){
        console.log(v)
    }
    // 1
    // 2
    // 3
    ```
### 0x005 Object
- `Object.assign(target, ...sources)`
    对象合并，将第二个开始的参数合并到第一个，并返回一个新的对象，不存在的属性将会添加，存在的属性将会覆盖。
    - `target`：目标对象
    - `sources`：源对象
    ```
    Object.assign({}, {a:1},{a:2,b:2})// {a:2,b:2}
    ```