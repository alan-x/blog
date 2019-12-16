### 0x000 概述
哈哈哈，看到很有意思的三个词，仓库放[这里](https://github.com/sindresorhus/ponyfill)。
以及三篇引用的文章：
- [Ponyfill definition - Sillicon Valley Dictionary](http://svdictionary.com/words/ponyfill)
- [Polyfills or Ponyfills? - Pony Foo](https://ponyfoo.com/articles/polyfills-or-ponyfills)
- [Polyfill, Ponyfill and Prollyfill - Kikobeats](https://kikobeats.com/polyfill-ponyfill-and-prollyfill/)


### 0x001 polyfill

polyfill 是针对原生 API 的 [monkey patch](https://www.jianshu.com/p/f1060b22aab8)，他会影响整个运行环境。比如：
```
// polyfill
Number.isNaN = Number.isNaN || function (value) {
	return value !== value;
};

// 使用
require('is-nan-polyfill');

Number.isNaN(5);
```

### 0x002 ponyfill

像 polyfill 却带着小马的纯净（Like polyfill but with pony pureness）？

ponyfill 是以模块的方式导出功能，因此只会在本地，而不会影响整个环境：
```
// ponyfill
module.exports = function (value) {
	return value !== value;
};
// 使用
var isNanPonyfill = require('is-nan-ponyfill');

isNanPonyfill(5);
```

### 0x003 prollyfill
> 注意：这里主要来自上面第 3 篇文章，但是我觉得原文语义好像有问（prollyfill 写成 polyfill？）。

prollyfill 表示一个还未标准化的 API，它虽然现在不再规范中，但是将来会支持。

比如，我希望`JSON.save`或者`JSON.load` 成为原生方法，但是现在不在。尽管你可以使用 [JSON Future](https://github.com/Kikobeats/json-future) pollyfill。


### 0x004 带货

帮大佬带货：[随心秀-微场景编辑器](https://juejin.im/post/5db718b7f265da4d1e26cb9c)。

---

