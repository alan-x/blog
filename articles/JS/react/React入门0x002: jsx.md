### 0x000 概述

`jsx`也是`js`， 如是说。

### 0x001 语法
在上文[React入门0x001-环境配置和 helloworld][1]中， 出现了一句奇怪的代码：
```
<h1>Hello, world!</h1>
```
这在`html`中没有任何问题，但问题是他出现在了`js`中，这就是`jsx`了，它的语法非常简单，却也很神奇：

    ```
    // 示例，之后会解析
    // 保存到变量
    let p = <p>this is tag p</p>
    // 嵌套
    let div = <div>
        {p}
    </div>
    // 执行语句
    let div2 = <div>
    {
        div === undefined
            ? <p>undefined</p>
            : {div}
    }
    </div>
    ```
### 0x002 说明
首先是为什么 `js` 中能够识别 `jsx` 呢？这倒不是`react`的功劳，而是 `babel` 的功劳，在`.babelrc`中配置了一个插件：`transform-react-jsx`，就是这个插件，才能解析`jsx`，而这个插件是如果和解析的呢？我们可以查看这个插件的[文档](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)：

- 输入
    ```
    var profile = <div>
      <img src="avatar.png" className="profile" />
      <h3>{[user.firstName, user.lastName].join(' ')}</h3>
    </div>;
    ```
- 输出
    ```
    var profile = React.createElement("div", null,
      React.createElement("img", { src: "avatar.png", className: "profile" }),
      React.createElement("h3", null, [user.firstName, user.lastName].join(" "))
    );
    ```
其中，`jsx`中的每一个标签变成了一个 `React.createElement(...)`函数，而标签的名字，变成了该函数的第一个参数，而`img`标签的`src`和`className`等属性变成了该函数的第二个参数，`jsx`中的嵌套元素，比如`div`中的`img`、`h3`变成了第三个参数。

具体是否是这样，可以编译来看看:
- 源代码：
    ```
    // src/jsx.js
    var user = {
        firstName: "",
        lastName: ""
    }
    var profile = <div>
        <img src="avatar.png" className="profile"/>
        <h3>{[user.firstName, user.lastName].join(' ')}</h3>
    </div>;
    ```
- 编译：
    ```shell
    $ npm install -g babel-cli
    $ babel --plugins transform-react-jsx jsx.js 
    
    "use strict";

    var user = {
        firstName: "",
        lastName: ""
    };
    var profile = React.createElement(
        "div",
        null,
        React.createElement("img", { src: "avatar.png", className: "profile" }),
        React.createElement(
            "h3",
            null,
            [user.firstName, user.lastName].join(' ')
        )
    );

    
    ```


### 0x003 `React.createElement`

该函数是由`React`提供的，所以我们可以看看它的[文档][2]说明：
```
React.createElement(
  type,
  [props],
  [...children]
)
```
参数： 
- `type`：类型，可以是一个标签名字，比如`p`、`div`等`html`标签，也可以是一个`React Component`，或者`React Fragment`，具体之后再表。
    
- `props`：属性集合，比如`src`、`className`等`html`属性（`className`对应`class`，`class`是关键词，所以用`className`代替），也可以是自定义的属性。
    
- `children`：`React element`子元素集合。
    
返回值：`React element`


### 0x004 总结

`jsx`只是一层语法糖，在`babel`的转化下变成了相应的`js`代码，本质上还是`js`，所以，在`react`中用`jsx`或者不用`jsx`都是没有本质区别的。上一篇文章中的代码可以改为如下形式：
```
import React from 'react'
import ReactDom from 'react-dom'

ReactDom.render(
    React.createElement(
        "h1",
        null,
        "Hello, world!"
    ),
    document.getElementById('app')
);
```
查看浏览器：`http://localhost:8080/`

![clipboard.png](/img/bVbfHNR)

那为什么`React`推荐构建`UI`的时候使用`jsx`，而不是使用`js`呢，用两种形式实现对比一下就好了
- `React.createElement` 形式
    ```
    ReactDom.render(
        React.createElement(
            "div",
            null,
            React.createElement(
                "h1",
                null,
                "送方外上人 / 送上人"
            ),
            React.createElement(
                "p",
                null,
                "孤云将野鹤，岂向人间住。莫买沃洲山，时人已知处。"
            )
        ),
        document.getElementById('app')
    )
    ```
- `jsx`形式
    ```
    ReactDom.render(
        <div>
            <h1>送方外上人 / 送上人</h1>
            <p>孤云将野鹤，岂向人间住。莫买沃洲山，时人已知处。</p>
        </div>,
        document.getElementById('app')
    )
    ;
    ```

可以看到，使用`js`形式有太多的`''`、`)`之类影响布局视觉的符号，相对于`html`形式的`jsx`显得繁杂而又不直观，对原本的`web`开发者也不友好，但这也只是一家之言，`flutter`在布局方面就采用的代码形式的，视个人爱好而言的东西罢了！


  [1]: https://segmentfault.com/a/1190000016083117
  [2]: https://reactjs.org/docs/react-api.html#createelement
