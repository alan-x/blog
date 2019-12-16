### 0x000 概述
上一章说明了生命周期的概念，本质上就是框架在操作组件的过程中暴露出来的一系列钩子，我们可以选择我们需要的钩子，完成我们自己的业务，以下讲的是`react v16.3`以下的生命周期，`v16.3`以及以上的版本有所不同

### 0x001 组件挂载
以下是组件挂载的过程中触发的声明周期：
```
class App extends React.Component {
    constructor(props) {
        super(props)
        console.log('constructor', props)

    }

    componentWillMount() {
        console.log('componentWillMount')
    }

    componentDidMount() {
        console.log('componentDidMount')
    }

    render() {
        console.log('render')
        return <p>{Date()}</p>
    }

    componentDidMount() {
        console.log('componentDidMount')
    }
}
```
![clipboard.png](/img/bVbf0q8)

### 0x002 组件更新
以下是组件更新的时候触发的生命周期：
```
class App extends React.Component {
    constructor() {
        super()
        this.state = {
            date: Date()
        }
        setTimeout(() => {
            this.setState({date: Date()})
        }, 3000)
    }

    componentWillReceiveProps() {
        console.log('componentWillReceiveProps')
    }

    shouldComponentUpdate() {
        console.log('shouldComponentUpdate')
        return true

    }

    render() {
        console.log('render')
        return <p>{this.state.date}</p>
    }

    componentWillUpdate() {
        console.log('componentWillUpdate')

    }

    componentDidUpdate() {
        console.log('componentDidUpdate')

    }

}
```
第一次的`render`是由组件挂载引起的，而其他的方法则是由`setState`引起的
![clipboard.png](/img/bVbf0rS)

### 0x003 组件卸载
以下是由组件卸载的时候触发的生命周期：
```

class Content extends React.Component {
    render(){
        console.log('Content::render')
        return <p>content</p>
    }
    componentWillUnmount() {
        console.log('Content::componentWillUnmount')
    }
}

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            show: true
        }
        setTimeout(() => {
            this.setState({show: false})
        }, 3000)
    }

    render() {
        console.log('App::render')

        return (
            this.state.show
                ? <Content/>
                : null
        )
    }



}
```

![clipboard.png](/img/bVbf0sk)


### 0x004 完整生命周期
```
挂载：
constructor
componentWillMount
render
componentDidMount
更新：
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate
卸载：
componentWillUnmount
错误处理（这里不说）：
componentDidCatch
```
### 0x005 声明周期使用场景
1. `constructor(props)`：
    > 该方法主要用来初始化`state`，或者初始化一些资源，可以访问`prop`，但是访问不了 `setState`，会报错：Warning: Can't call setState on a component that is not yet mounted. This is a no-op, but it might indicate a bug in your application. Instead, assign to `this.state` directly or define a `state = {};` class property with the desired state in the App component.
    ```
    class App extends React.Component {
        constructor() {
            super()
            this.state = {
                show: true
            }
        }
    
        render() {
            return (
                this.state.show
                    ? <Content/>
                    : null
            )
        }
    
    }
    ```
2. `componentWillMount()`
    > 没啥卵用，可以在这个方法中调用`setState()`，并且设置的`state`可以在本次渲染生效，推荐使用`constructor`
3. `render（）`
    > 你懂的，每次更新状态都会触发，但是不要在这里调用会触发组件更新的函数，比如`setState()`，否则可能陷入无尽阿鼻地狱。
4. `componentDidMount()`
    > 这个方法常用，触发这个生命周期，意味着`dom`和子组件都挂载好了，`refs`也可以用了，一般在这儿做网络请求。
5. `componentWillReceiveProps(nextProps)`
    > 这个组件也常用，一般用于父组件状态更新，导致传递给子组件的`props`更新，当时子组件又不是直接绑定父组件的`props`的时候使用，比如
    ```
    class Content extends React.Component {
        constructor(props) {
            super(props)
            this.state = {
                content: props.content
            }
        }
   
        render() {
            return this.state.content
        }
    }
    
    class App extends React.Component {
        constructor() {
            super()
            this.state = {
                content: "1"
            }
            setTimeout(() => {
                this.setState({
                    content: 10
                })
            }, 1000)
        }
    
        render() {
            return (
                <Content content={this.state.content}/>
            )
        }
    
    }
    ```
    我们接受父组件的`state.content`作为子组件的初始化`state.content`，但是1s 之后，父组件发生了变化，`state.content`从1变成了10，我们期望子组件也同时更新，可惜子组件的`constructor`只会执行一次，为了解决这个问题，我们可以添加这个生命周期：
    ```
    class Content extends React.Component {
        constructor(props) {
            super(props)
            this.state = {
                content: props.content
            }
        }
    
        componentWillReceiveProps(nextProps) {
            this.setState({
                content:nextProps.content
            })
        }
    
        render() {
            return this.state.content
        }
    }
    ```
    这样就可已在父组件`props`更新的时候，触发子组件的更新
6. `shouldComponentUpdate(nextProps, nextState)`
    > 这个组件也很常用，用来判断是否要进行某次更新，如果返回`true`则执行更新，如果返回`false`则不更新，常用与性能优化
    ```
    class A extends React.Component {
        render() {
            console.log("A::render")
            return "A"
        }
    }
    
    class B extends React.Component {
        render() {
            console.log("B::render")
            return "A"
        }
    }
    
    class App extends React.Component {
        constructor(props) {
            super(props)
            this.state = {
                num: 1
            }
            setTimeout(() => {
                this.setState({
                    num: 10,
                    name: 1
                })
            })
        }
    
        render() {
            console.log("App::render")
            return <div>
                    <A name={this.state.name}/>
                    <B name={this.state.name}/>
                <div>
                    {this.state.num}
                </div>
            </div>
        }
    }
    
    ReactDom.render(
        <App></App>,
        document.getElementById('app')
    )
    ```
    我们在`App`组件中挂载了`A`、`B`组件，`App`绑定了`state.num`，`A`、`B`绑定了`state.name`，然后在1s 后触发`app`组件的更新，此时查看浏览器，可以看到，`APP`、`A`、`B`都执行了`render`，但其实`A`、`B`依赖的`name`并没有发生改变。如果是小组件还好，但如果是大组件，那就糟糕了，所以需要避免这种无所谓的更新：
    ```
    class A extends React.Component {
        shouldComponentUpdate(nextProps, nextState) {
            return nextProps.name === this.props.name ? true : false
        }
    
        render() {
            console.log("A::render")
            return "A"
        }
    }
    
    class B extends React.Component {
        shouldComponentUpdate(nextProps, nextState) {
            return nextProps.name === this.props.name ? false : true
        }
    
        render() {
            console.log("B::render")
            return "A"
        }
    }
    ```
    我在促发这个方法的时候，`A`组件返回 `true`，`B` 组件返回`false`，查看浏览器，可以发现，触发该方法的时候 `A`渲染了，而`B`没有渲染
    
    ![clipboard.png](/img/bVbf0xi)
    
7. `componentWillUpdate(nextProps, nextState)`
    > 没啥卵用，和`componentDidUpdate`组成一对儿，如果业务需要，就用

8. `componentDidUpdate()`
    > 没啥卵用，和`componentWillUpdate`组成一对儿，如果业务需要，就用
    
9. `componentWillUnmount`
    > 用于清理资源或者事件
    ```
    class Content extends React.Component {
        constructor() {
            super()
            this.state = {
                num: 1
            }
            setInterval(() => {
                this.setState({
                    num: ++this.state.num
                })
                console.log(this.state.num)
            }, 1000)
        }
    
        render() {
            return this.state.num
        }
    }
    
    class App extends React.Component {
        constructor() {
            super()
            this.state = {
                show: true
            }
            setTimeout(() => {
                this.setState({
                    show: false
                })
            },2000)
        }
    
        render() {
            return this.state.show
                ? <Content/>
                : null
        }
    
    }
    
    ReactDom
        .render(
            <App></App>,
            document
                .getElementById(
                    'app'
                )
        )

    ```
    我们在`App`组件中挂载`Content`组件，而`Content`组件则使用定时器每秒更新一次，但是在2s 之后，我们在`App`组件中卸载这个组件，但是，查看浏览器可以发现，定时器依旧在工作，并且报错：
    
![clipboard.png](/img/bVbf0yO)
    如果我们返回显示隐藏组件，就会积累越来越多的定时器。然后就爆炸了。
    
    ```
    class Content extends React.Component {
        constructor() {
            super()
            this.state = {
                num: 1
            }
            this.interval=setInterval(() => {
                this.setState({
                    num: ++this.state.num
                })
                console.log(this.state.num)
            }, 1000)
        }
    
        render() {
            return this.state.num
        }
        componentWillUnmount(){
            clearInterval(this.interval)
        }
    }
    ```
    这么解决
    
### 0x006 总结
在不同的生命周期做不同的事。

### 0x007 资源：

- [react](https://reactjs.org/)
- [源码](https://github.com/followWinter/react-study)
    
    
    
