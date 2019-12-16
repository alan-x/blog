### 0x001 概述
[上一章][1]讲的是js压缩混淆，和这一章没有半毛钱关系，这章讲的是`DefinePlugin`，一个好像没有用，但其实很好用的一个插件，我很喜欢，嘿嘿嘿！

### 0x002 插件介绍
说白了，这是一个简单的字符串替换插件，将我们所有经过`webpack`打包的`js`文件的对应的字符串都替换为我们在这个插件中指定的字符串。

### 0x003 栗子来了
1. 初始化项目
    ```
    + 0x005-define-plugin
      + src
        - index.js
      - webpack.config.js
    ```
2. 安装依赖包
    ```
    $ npm init -y
    $ npm install --save-dev webpack
    ```
3. 编写`webpack.config.js`
    ```
    var path       = require('path')
    module.exports = {
        entry : ['./src/index.js'],
        output: {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        }
    }
    ```
4. 添加插件，并指定几个常量
    ```
    var path       = require('path')
    var webpack    = require('webpack')
    module.exports = {
        entry  : ['./src/index.js'],
        output : {
            path    : path.resolve(__dirname, 'dist'),
            filename: 'index.js'
        },
        plugins: [
            new webpack.DefinePlugin({
                PRODUCTION            : JSON.stringify(true),
                VERSION               : JSON.stringify("2.0.1"),
                BROWSER_SUPPORTS_HTML5: true,
                TWO                   : "1+1",
                "typeof window"       : JSON.stringify("object"),
                不想写代码                 : JSON.stringify("i like boss"),
                弹窗一下                  : "alert('我弹窗了')"
            })
        ]
    }
    ```
5. 调用这几个变量
    ```
    // src/index.js
    console.log("Running App version " + VERSION);
    if (!BROWSER_SUPPORTS_HTML5)
        console.log('BROWSER_SUPPORTS_HTML5')
    
    if (PRODUCTION) {
        console.log('Production log')
    }
    console.log(TWO)
    
    console.log(typeof window)
    
    console.log(不想写代码)
    
    
    弹窗一下
    ```
6. 打包并查看打包后的结果
    ```
    // dist/index.js
    ...
    console.log("Running App version " + "2.0.1");
    if (false)
        console.log('BROWSER_SUPPORTS_HTML5')
    
    if (true) {
        console.log('Production log')
    }
    console.log(1+1)
    
    console.log("object")
    
    console.log("i like boss")
    
    alert('我弹窗了')
    ...
    ```
    就是这种用处了，直接将匹配的字符串替换，当然后面两句是玩笑，万万不可这么做，直接用这个插件替换逻辑会造成维护上的问题的，推荐只使用这个插件做全局的变量替换，比如在不同环境之间切换等。

7. 更多细节，查阅[webpack关于definePlugin章节][2]。

### 0x004 注意
该插件是简单的字符串替换，所以如果是定义常量最好使用`""`包裹要替换的内容，或者使用`JSON.stringify()`转化，否则会变成代码直接插入，比如版本号：`'"1.2.1"'`,这样替换的时候就会变成`"1.2.1"`,而不会变成`1.2.1`,导致错误的数据格式。

### 0x005 最佳使用搭配（使用场景而已，日后会详细说明）
- `git`配合`jenkins`之类的`ci`做自动化发布的时候注入版本号
    - 每一次`git push`就自动递增`package.json`中的版本号
    - `git tag`读取`package.json`中的版本号发布新版本
    - `jenkins`接收到`webhook`之后，调用`webpack`构建发布版本，`webpack`将会注入`package.json`中的版本号，让`app`中显示当前的版本
    - 打包完成之后做资源上传，页面部署之类的
    - 这样就让`git`版本号、`app`版本号、`package.json`版本号统一。
- `切换环境`
    - 当拥有两套环境时，比如测试环境和正式环境，我们可以定义`PRODUCTION`变量来做判定，判定当前的环境，并对此作出不同的操作，比如替换`api`地址等

### 0x006 资源
- [源代码][3]


  [1]: https://segmentfault.com/a/1190000011868498
  [2]: https://webpack.js.org/plugins/define-plugin/
  [3]: https://github.com/followWinter/webpack-study/tree/master/0x005-define-pligin
