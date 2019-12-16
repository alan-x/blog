### 0x001 概述
本篇文章[承接上文][1]，记录的是如何发布自己的Node.js模块

### 0x002 编写模块
- 新建项目并初始化
```
$ mkdir 0x005-publish-own-module
$ cd 0x005-publish-own-module
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help json` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (0x005-publish-own-module) 
version: (1.0.0) 
description: 
entry point: (index.js) 
test command: 
git repository: 
keywords: 
author: 
license: (ISC) 
About to write to /MY_PROJECT/PROJECT_OWN/NodeJS/npm/0x005-publish-own-module/package.json:

{
  "name": "0x005-publish-own-module",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}


Is this ok? (yes)
```
- 编写模块代码
```
$ vim index.js

// index.js
exports.printMsg = function() {
  console.log("This is a message from the demo package");
}
```
- 发布模块
```
$ npm publish --access=public
+ 0x005-publish-own-module@1.0.0
```
- 测试模块
```
$ mkdir 0x006-use-own-package
$ cd 0x006-use-own-package
$ npm install 0x005-publish-own-module@1.0.0
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN npm@1.0.0 No description
npm WARN npm@1.0.0 No repository field.

+ 0x005-publish-own-module@1.0.0

$ vim index.js

// index.js
var myModule = require('0x005-publish-own-module');
console.log(myModule);
myModule.printMsg();

$ node index.js
{ printMsg: [Function] }
This is a message from the demo package
```

### 0x003 命名空间
每个人都可以发布自己的包，难免会有包名相同的情况，如果想要使用一个已经存在的包的包名，可以使用命名空间
将`package.json`中的包名该为`@scope/package_name`就行，比如`@followwinter/lodash`
其中，scope为当前登录的用户名，package_name便是包名，则在安装、更新、移除、require包的时候都必须该为这种格式

### 0x004 包版本管理
项目的初始化版本号为`1.0.0`，当然也可以自行修改，也可以不遵守以下规范
- 主版本号：版本更新，具有颠覆式的改变或者架构的改变
- 次版本号：新功能更新
- bug修复版本号：bug修复

### 0x005 tag使用
- 为一个版本添加一个`tag`
`npm dist-tag add <pkg>@<version> [<tag>]`
- 发布一个有`tag`的版本
```
npm publish --tag beta --access=public
```
- 安装
```
npm install 0x005-publish-own-module@beta
```

0x006 总结
- `npm publish [[--tag beta] [--access public]]` 
    - 发布一个包
    - 如果`access=public`，则这个包为公共的，所有人都可以通过`npm`安装这个包
    - 如果携带了`tag`参数，则可以通过`npm install <package_name@<tag_name>>`来安装这个版本的包
- `npm dist-tag add <pkg>@<version> [<tag>]`为某个版本添加一个tag

#### 0x007 资源
- [项目github][2]


  [1]: https://segmentfault.com/a/1190000011101938
  [2]: https://github.com/followWinter/npm-study.git
