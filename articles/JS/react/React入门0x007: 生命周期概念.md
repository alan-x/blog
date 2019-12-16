### 0x000 概述

上一章只是稍微了解了一下`state`和`setState`相关的简单用法，这一章需要讲一下组件的生命周期。

### 0x001 生命周期的概念
这玩意似乎很高大上，其实就是一个假概念罢了，直接来实现一个类似的吧。大凡事物从出现到消亡都有个过程，而这个过程大抵可以分为：
```
出现->变化->消亡
```
而`web`开发中，组件也有类似的过程，当然作为组件的创造者，我们可以干预这个过程，在每个过程加入自己的行为，也就是`hook`这个过程：
```
         (出现之前)->出现->（出现之后）
                     ↓
->(循环)[（变化之前）->变化->(变化之后)]
                    ↓
->      （消亡之前）->消亡->（消亡之后）
```
这个就是生命周期了，转化成组件：
```
（组件声明->构造组件->组件挂载之前）->组件挂载->(组件挂载之后)
                                   ↓
->          (循环)[(组件更新之前)->组件更新->(组件更新之后)]
                                   ↓
->               （组件卸载之前）->组件卸载->（组件卸载之后）
```
上面`(...)`中的便是我们的`hook`，接下来我们来实现它
### 0x002  初始化框架
1. 复制项目
    ```
    $ cp 006-state 007-life-cycle
    ```
2. 完成框架主文件-`ReactLikeDom.js`
    ```
    $ cd 0x007-life-cycle/src
    $ vim ReactLike.js
    // ReactLikeDom.js 模拟 ReactDom
    class ReactLikeDom {
        static render(component, container) {
        }
    }
    export default ReactLikeDom   
    ```
3. 完成框架主文件-`ReactLike.js`
    ```
    // ReactLike.js 用来模拟 React
    class ReactLike {
        static createElement(type, props) {
        }
    }
    
    export default ReactLike
    ```
3. 完成框架主文件-`Component.js`

    ```
    // 用来模拟 React.Component
    class Component {
    
    }
    export default Component
    ```
4. 完成框架主文件-`ReactLikeNode.js`
    ```
    // 用来模拟 React Element ，每一个组件都将会被抽象成一个 ReatLikeElement
    class ReactLikeElement {
        constructor() {

        }
    
    }
    
    export default ReactLikeElement
    ```

5. 当前项目文件

        + 0x007-life-cycle
        + src
            - Component.js
            - index.html
            - index.js
            - ReactLike.js
            - ReactLikeDom.js
            - ReactLikeElement.js
        - .babelrc
        - package.json
        - webpack.config.js
        
### 0x003 我们先看看我们想要干什么
```
// index.js
import ReactLikeDom from "./ReactLikeDom";
import ReactLike from "./ReactLike";
import Component from "./Component";

// 声明一个组件
class App extends Component {
    render() {
        return "hello " + this.props.name
    }
}
// 将组件挂载到容器中
ReactLikeDom.render(
    ReactLike.createElement(App, {name: "reactLike"}, null),
    document.getElementById('app')
)
``` 
打开浏览器查询最终的效果：

![clipboard.png](/img/bVbfWIp)

### 0x004 完成`Compoennt.js`

> 这个类其实没有太多需要做的，只是为了继承而已，
> 而在`React`中每一个组件都必须有一个`render`方法，该方法返回一个`React Element`，
> 而这个方法实在具体的组件完成的，而不是在这里完成，所以我们这里声明一下就好了


```
class Component {
    render() {
        
    }
}

export default Component
```

### 0x004 完成`ReactLike`类
> 该类有一个方法：`createElement`，作用是将一个类组件转化成`ReactLikeElement`
```
import ReactLikeElement from "./ReactLikeElement";

class ReactLike {
    static createElement(type, props) {
        // 直接创建一个 ReactLikeElement 并返回, 具体的实现由 ReactLikeElement 自己完成
        return new ReactLikeElement(type, props)
    }
}

export default ReactLike
```
### 0x005 完成`ReactLikeElement`
> 该类的所有事物都在构造器中完成
```
class ReactLikeElement {
    constructor(type, props) {
        // 实例化组件
        let component = new type(props)
        // 将 props 赋予 该组件实例
        component.props = props
        // 将该组件的真实 dom 保存在 ref 中
        this.ref = component.render()
    }

}

export default ReactLikeElement
```

### 0x006 自定义一个组件
```
// index.js
class App extends Component {
    render() {
        let div = document.createElement('div')
        div.append("hello " + this.props.name)

        return div
    }
}
```
该组件返回一个`div`，`div`中包含一个字符串，该字符串的构成是`hello `和这个组件的`props`中的`name`构成，我们期望外部传入一个`name`属性。

可以使用该自定义组件测试一下`ReactLikeElement`，看看一个组件究竟会被转化成什么样：
```
console.log(new ReactLikeElement(App, {name: "reactLike"}))
```
查看浏览器
![clipboard.png](/img/bVbfWJk)

### 0x007 完成`ReactDom`，将该组件挂载到容器中
```
class ReactLikeDom {
    static render(reactLikeElement, container) {
        container.append(reactLikeElement.ref)
    }

}

export default ReactLikeDom
```
`ReactLikeDom.render`接受一个元素和一个容器，组件是`ReactLikeElement`，容器是`dom`，这个方法会将组件挂载到容器上。
这个时候整个流程就已经完成了，查看浏览器

![clipboard.png](/img/bVbfWJ1)

我们梳理一下整个流程:
```
// index.js
class App extends Component {
    // 该方法返回一个`dom`，构建这个`dom`用到了`props`中的属性`name`
    render() {
        let div = document.createElement('div')
        div.append("hello " + this.props.name)

        return div
    }
}

/*
 * ReactLikeDom.render 可以将一个`ReactLikeElement`挂载到容器上
 * 而 ReactLike.createElement 正好可以创建一个`ReactLikeElement`
 * 在 ReactLike.createElement 中会构造一个 App 实例
 * 然后将 props 传递给该 App 实例
 * 然后调用 App.render
 * 这时候 render 中便可以访问 props 中的属性了
 * 接着将 App.render 返回的 dom 保存在 ref 变量中
 * ReactLikeDom.render 其实是把 ref 中的 dom 挂载到 容器上
 */

ReactLikeDom.render(
    ReactLike.createElement(App, {name: "react"}, null),
    document.getElementById('app')
)
```

### 0x008 实现生命周期

在`Component`中添加两个方法，代表其中两个生命周期
```
class Component {
    componentWillMount() {

    }

    componentDidMount() {

    }


    render() {

    }
}

export default Component
```

然后在`App`组件中实现它：
```
class App extends Component {
    componentDidMount() {
        console.log('componentDidMount')
    }

    componentWillMount() {
        console.log('componentWillMount')
    }

    render() {
        let div = document.createElement('div')
        div.append("hello " + this.props.name)

        return div
    }
}
```

接着让他起作用：
```
// ReactLikeDom.js
class ReactLikeDom {
    static render(reactLikeElement, container) {
        reactLikeElement.componentWillMount()
        container.append(reactLikeElement.ref)
        reactLikeElement.componentDidMount()
    }

}

export default ReactLikeDom

// ReactLikeElement
class ReactLikeElement {
    constructor(type, props) {
        let component = new type(props)
        component.props = props
        this.ref = component.render()
        this.component = component // 新添加一个属性
    }

}

export default ReactLikeElement
```
查看浏览器

![clipboard.png](/img/bVbfWLg)

### 0x009 总结

我们已经实现了其中两个生命周期，其他的声明周期也是如此类似的实现，只是插入的位置和时期不同而已，也造成在每个阶段我们所能访问的资源也不同。



### 0x010 资源
- [react](https://reactjs.org/)
- [transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)
- [源码](https://github.com/followWinter/react-study)
