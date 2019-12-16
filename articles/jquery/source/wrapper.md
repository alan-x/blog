### 包装器
上一章讲到`jQuery`外层的`IIFE`是为了兼容各种环境而出现的，而且这层代码并不是打包器(`RequireJS`)生成的，而是`jQuery`自己实现的，具体代码在`src/wrapper.js`中，其中 49~50 行有如下注释：
```
// @CODE
// build.js inserts compiled jQuery here
```
这表示代码生成后，将会吧代码插入到这里来。

那这段代码是如何兼容的呢？
在`()()`的第一个`()`传入了一个函数表达式，完整逻辑如下：
```

( function( global, factory ) {

	"use strict";

	if ( typeof module === "object" && typeof module.exports === "object" ) {

		// For CommonJS and CommonJS-like environments where a proper `window`
		// is present, execute the factory and get jQuery.
		// For environments that do not have a `window` with a `document`
		// (such as Node.js), expose a factory as module.exports.
		// This accentuates the need for the creation of a real `window`.
		// e.g. var jQuery = require("jquery")(window);
		// See ticket #14549 for more info.
		module.exports = global.document ?
			factory( global, true ) :
			function( w ) {
				if ( !w.document ) {
					throw new Error( "jQuery requires a window with a document" );
				}
				return factory( w );
			};
	} else {
		factory( global );
	}

// Pass this if window is not defined yet
} )(typeof window !== "undefined" ? window : this, function ( window, noGlobal ) {...})
```
其中特意保留了注释：
- 对于拥有`window`属性的`CommonJS`和类似`CommonJS`的环境，执行`factory`获得`jQuery`。
- 对于没有拥有`document`属性的`window`属性的环境（比如`Node.js`），将`factory`暴露为`module.export`。
- 需要强调的是这要创建一个真实的`window`。
- 比如`var jQuery = require("jquery")(window)`;

这里分两中情况讨论：
- 浏览器环境
- `Node.js`

### 浏览器环境

在浏览器环境下，第18行`typeof module === "object" && typeof module.exports === "object"`为`false`，所以走第35行`else`分支，直接执行`factory( global )`：

- `factory`是该函数表达式的第二个参数，对应的是第二个`()`的第二个函数`function ( window, noGlobal ) `，执行该函数，并将`global`作为该函数的第一个参数`window`传入
- `global`是该函数表达式的第一个参数，对应的的是第二个`()`的第一个参数`ypeof window !== "undefined" ? window : this`，这个表达式在浏览器环境的结果就是`window`

所以，在浏览器环境下，可以简化为如下：
```
(function( global, factory ){
    factory( global )
    })( window, function( window, noGlobal ){ })
```
继续简化：
```
factory( window )
```

### Node.js 环境

我们从上面的栗子可以看出参数是怎么传输的了，在浏览器中，第一个`()`中的函数表达式接受到的参数其实是`this`和`function( window, noGlobal ){ }`。这里的`this`是`Node.js`的`this`，可以使用`Node`的`REPL`来输出看看到底是什么：
```
$ node
> this
Object [global] {
    ...
}
```
可以看到一大堆的变量和方法，比如我们`process`，`setTimeout`等，再尝试访问`this.module`
```
> this.module
Module {
    id: '<repl>',
    exports: {},
    parent: undefined,
    filename: null,
    loaded: false,
    children: [],
    paths:
    [ ... ] }
 ```

可以看到，`this.module`是一个对象，并且`this.module.exports`也是一个对象。所以，第18行的`typeof module === "object" && typeof module.exports === "object" `执行结果为`true`，走第19-37行。

在第27行，判定`global.document`是否存在，这里存在两种可能：

- 存在，说明是类`CommonJS`的环境，比如使用`webpack`或者`RequireJS`，和`Node.js`一样，存在`module`属性，并且`module`拥有`exports`属性。可以通过判定`document`来区别是`CommonJS`还是类`CommonJS`环境。如果已经拥有了`document`，那就直接通过`factory(global, true)`获取`jQuery`。可以简化为：
    ```
    module.exports = (function(global, factory){
        factory(global, true)
    })(function(this, function(window, global){...})
    ```
    继续简化：
    ```
    module.exports = factory(this, true)
    ```

- 不存在，则是在`Node.js`中，返回一个函数：
    ```
    function( w ) {
        if ( !w.document ) {
            throw new Error( "jQuery requires a window with a document" );
        }
        return factory( w );
    };
    ```
    该函数接受一个参数`w`，这个`w`必须拥有一个`document`属性，否则将抛出一个异常。如果的确有`document`属性，就执行`factory(w)`获取`jQuery`。
    可以简化为
    ```
    module.exports = function( w ) {
        if ( !w.document ) {
            throw new Error( "jQuery requires a window with a document" );
        }
        return factory( w );
    };
    ```
    `jquery`在[npm仓库](https://www.npmjs.com/package/jquery)的页面有一个栗子说明了这种情况下的使用方式：
    ```
    require("jsdom").env("", function(err, window) {
        if (err) {
            console.error(err);
            return;
        }
    
        var $ = require("jquery")(window);
    });
    ```
### 总结
其实上面的判断都是为了确保有一个`window`对象，并且该`window`对象下有一个`document`对象，因为`jQuery`需要挂载在`window`上并且需要调用`window`上和`document`上的许多方法，比如`querySelector`和`document.getElementById`。