---
title: '[ES6]0x002-var、let、const、块级作用域、暂存死区'
date: 2018-11-26 20:11:22
categories: ES6
tags: ES6
---
### 0x001 `var`
1. 语法
    ```
    var varname1 [= value1 [, varname2 [, varname3 ... [, varnameN]]]];
    ```
2. 使用
    ```
    var a, b=2 // 声明多个变量，可以赋值，也可以不赋值
    a=1 // 先声明，再赋值
    ```
3. `var`变量提升
    使用`var`声明的变量将会被提升到函数的顶部
    ```
    console.log(a) // undefined
    var a =2  
    console.log(a) // 2
    console.log(b) //Uncaught ReferenceError: b is not defined...
    ```
    以上代码相当于
    ```
    var a
    console.log(a) // undefined
    a=2
    console.log(a) // 2
    console.log(b) //Uncaught ReferenceError: b is not defined...
    ```
    
### 0x002 `let`
1. 语法
    ```
    let var1 [= value1] [, var2 [= value2]] [, ..., varN [= valueN]];
    ```
2. 使用
    ```
    let a, b = 2 // 声明多个变量，赋值不赋值无所谓
    a = 2 // 声明之后再赋值
    ```
3. 不再提升
    ```
    console.log(a) // Uncaught ReferenceError: a is not defined...
    let a=1 
    ```
    **注意：**猜测， 使用`babel`翻译一下代码发现，只是`let`变成了`var`，所以使用`babel`转义之后的代码依旧会提升
4. 不能重复声明
    ```
    let a=1
    let a=2 // Uncaught SyntaxError: Identifier 'a' has already been declared
    ```
### 0x003 const
1. 语言
    ```
    const name1 = value1 [, name2 = value2 [, ... [, nameN = valueN]]];
    ```
2. 使用
    ```
    const a=1, b=2 // 不能省略的值
    ```
3. 不能省略的值
    ```
    const c // Uncaught SyntaxError: Missing initializer in const declaration
    ```
4. 不能重复赋值
    ```
    const d=4
    d=5 // Uncaught TypeError: Assignment to constant variable.
    ```
5. 可以修改的引用
    ```
    const e=[]
    e[0]=0
    console.log(e) //[0]
    ```
### 0x004 块级作用域
    
块级作用域是随着`let`、`const`而来最有用的特性了，在之前的`js`中，`js`的作用域是函数级的，由此带来的几个臭名昭著的问题：
- 意外被修改的值
    ```
    function varTest() {
      var x = 1;
      if (true) {
        var x = 2;  // 同样的变量!
        console.log(x);  // 2
      }
      console.log(x);  // 2
    }
    ```
    可以使用`let`避免了
    ```
    function letTest() {
      let x = 1;
      if (true) {
        let x = 2;  // 不同的变量
        console.log(x);  // 2
      }
      console.log(x);  // 1
    }
    ```
- 万恶的`for`循环和点击事件
    ```
    var list = document.getElementById("list");

    for (var i = 0; i < 5; i++) {
      var item = document.createElement("LI");
      item.appendChild(document.createTextNode("Item " + i));
    
      item.onclick = function (ev) {
        console.log("Item " +i + " is clicked.");
      };
      list.appendChild(item);
    }
    console.log(i) // 5
    ```
    如果点击上面，不管点击哪个，显示出来的都是`Item 5 is clicked.`，虽然可以使用闭包解决，但是现在有了更好的方案
    ```
    let list = document.getElementById("list");

    for (let i = 0; i < 5; i++) {
      let item = document.createElement("LI");
      item.appendChild(document.createTextNode("Item " + i));
    
      item.onclick = function (ev) {
        console.log("Item " +i + " is clicked.");
      };
      list.appendChild(item);
    }
    ```
### 0x005 作用域规则很简单
1. `{}`块内形成一个作用域，包括`if`、`else`、`while`、`class`、`do...while`、`{}`、`function`
    ```
    {
        const f=6
    }
    console.log(f) // Uncaught ReferenceError: f is not defined
    ```
2. `for`循环中用`let`声明一个初始因子，该因子在每个循环内都是一个新的作用域
    ```
    for (let i = 0; i < 10; i++) {
      console.log(i);
    }
    console.log(i) // Uncaught ReferenceError: i is not defined
    ```
3. `switch`只有一个作用域
    ```
    switch (x) {
      case 0:
        let foo;
        break;
        
      case 1:
        let foo; 
        break;
    }
    // Uncaught SyntaxError: Identifier 'foo' has already been declared
    ```
### 0x006 暂存死区-`Temporal Dead Zone`-`TDZ`
随着`let`和`const`的引入，也引入了暂存死区的概念。使用`var`的时候，作用域内（函数作用域），在还没使用`var`声明一个变量的时候，访问该变量，将会获得`undefined`。但是如果使用`let`，作用域（块级作用域）内，在还没使用`let`声明一个变量的时候，访问该变量，将会获得`ReferenceError`，从作用域开始到`let`语句之间，就是暂存死区。
    ```
    {
     console.log(a) // Uncaught ReferenceError: a is not defined
     console.log(b) // Uncaught ReferenceError: b is not defined
     console.log(c) // undefined
     // 暂存死区
     let a =1
     const b=2
     var c=3
    }
    
    ``` 
**注意：**猜测， 使用`babel`翻译一下代码发现，只是`let`变成了`var`，所以使用`babel`转义之后可能不存在暂存死区
### 0x007 总结
尽量使用`let`和`const`，如果希望改变该变量的值，则使用`let`，如果希望不再改变该变量的值或者引用，则使用`const`，让`var`成为历史。