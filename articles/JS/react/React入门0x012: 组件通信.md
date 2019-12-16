### 0x000 概述
这一章讲组件间通信，组件通信分为很多种：
- 父组件向子组件发起通信
- 子组件向父组件发起通信
- 兄弟组件间通讯
- 远程组件通信
在组件通信中，有一种错误的说法，那就是父组件调用子组件，子组件调用父组件，这种说法其实是错误的，并没有什么父组件调用子组件，这是一种传统的说法，从传统`dom`的操作方式来的，但是在`react`中，使用这种方法描述组件间的通信是不合适的。在我看来，用**事件通知模**型来描述可能更合适。

### 0x001 父组件向子组件发起通信
> 父组件向子组件发起通信很简单，直接使用`props`
```
import React from 'react'
import ReactDom from 'react-dom'

class Sub extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
    }

    componentWillReceiveProps(nextProps) {
        this.setState({
            num: nextProps.num
        })
    }

    render() {
        return <div>
            <p>props.num:{this.props.num}</p>
            <p>state.num:{this.state.num}</p>
        </div>
    }
}

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
        setInterval(() => {
            this.setState({
                num: ++this.state.num
            })
        }, 1000)

    }

    render() {
        return <Sub num={this.state.num}/>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)

```
查看浏览器：
![图片描述][1]

说明：`App`组件每秒更新一次`num`，同时将`num`通过`props`传递给子组件`Sub`，`Sub`通过`props`访问父级传递下来的`num`，并渲染到界面上，同时也将该参数保存到自己的`state`中，也渲染到界面上。可以看到，父级更新了`num`，子组件也同步更新了，也就完成了父组件向子组件发起通信的目的。


### 0x002 子组件向父组件发起通信
子组件向父组件发起通信则是通过事件触发，我们可以想想平时的`html`：
```
<button onclick="ourFunc('you click me')">确认</button>
```
我们给`button`绑定了`click`事件，所以当我们点击按钮的时间，该事件就会被触发，然后执行我们的函数`ourFunc`并传递参数`'you click me'`，这就是事件通知。现在我们假设`button`是我们实现的自定义组件，那么`onclick`就是我们传递的`props`。众所周知，`js`中，函数也是变量，所以在`react`中，`props`也可以传递函数：
```
import React from 'react'
import ReactDom from 'react-dom'

class MyButton extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
    }

    handleOnClick() {
        let num = ++this.state.num
        this.setState({
            num: num
        })
        this.props.onClick(num)
    }

    render() {
        return <button onClick={() => this.handleOnClick()}>{this.props.text}</button>
    }
}

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
    }

    render() {
        return <div>
            <p>点击了 {this.state.num} 次</p>
            <MyButton text='确认' onClick={(text) => this.handleOnClick(text)}/>
        </div>
    }

    handleOnClick(num) {
        this.setState({
            num: num
        })
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)

```
查看浏览器：
![图片描述][2]

**说明：**我们声明了一个`MyButton`组件，并传递了两个属性，一个是`text`，一个是`onClick`，注意，这里的`onClick`只是一个名字叫做`onClick`的属性，而不是点击事件，只是我们借用了这个名字而已，真正的点击事件必须绑定在`dom`上。所以这里叫什么名字都无所谓，不过为了好看，还是叫着名字比较好。
在`MyButton`中，我们真正的绑定了`onClick`事件，并在该事件触发的时候，调用传递过来的`props.onClick`，并吧点击的次数作为参数，这样，`App`的`handleOnClick`就会被促发，同时接收到`num`参数，用于显示用户的点击次数。
从表面上看，我们就好像在`MyButton`上绑定了事件一般，和`web`中的十分类似，但是这种机制更加的灵活，比如我们不一定要叫`onClick`，我们可以将之和业务绑定，叫做`onUserAdd`，这样，似乎就实现了自定义事件，并且具有一定的业务特点，非常的灵活。

### 0x003 兄弟组件通信
> 兄弟组件通信其实很简单，借助公共的父组件作为中间桥梁，然后通过上面说的`props`和事件机制，就完成了兄弟组件的通信，
稍微修改一下`0x002`的栗子就可以搞定了
```

class MyView extends React.Component {
    render() {
        return <p>点击了 {this.props.num} 次</p>
    }
}

class MyButton extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
    }

    handleOnClick() {
        let num = ++this.state.num
        this.setState({
            num: num
        })
        this.props.onClick(num)
    }

    render() {
        return <button onClick={() => this.handleOnClick()}>{this.props.text}</button>
    }
}

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
    }

    render() {
        return <div>
            <MyView num={this.state.num}/>
            <MyButton text='确认' onClick={(text) => this.handleOnClick(text)}/>
        </div>
    }

    handleOnClick(num) {
        this.setState({
            num: num
        })
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)

```
查看浏览器：
![图片描述][2]

**说明：**`MyButton`和`MyView`是挂载在`App`的子组件，他们是同级的兄弟组件，而他们之间的通信可以借助`App`作为中转，将数据从一个`MyButton`传递到`MyView`。具体过程：
- `MyButton`点击事件触发后触发父组件传递进来的`onClick`
- `App`监听`MyButton`的`onClick`事件并将该事件的`num`保存到自己的`state`中
- 而`state.num`又传递到了`MyView`中
所以当我们点击按钮的时候，点击次数就增加了。


### 0x004 远程组件通信
从上克制，组件间的通信可以通过`props`传递普通变量和函数来实现，但是如果事件嵌套的层级太深，这套模型就显得麻烦，必须要不断的传递`props`，不但冗余而且不好维护。
当然我们也可以通过适当的组件设计来避免过深的组件嵌套通信。
那如果无法避免呢？
那就需要用其他方式来实现了，比如，使用`订阅-发布模型`可以实现`无视距离和组件的通信`，甚至`无视框架和位置`，但是很容易引起混乱，适用于全局通知。
以下是简单实现：
- 实现通用的`订阅-发布模型框架`
```
let eventMap = {}

class MyEvent {

    static pub(name, data) {
        if (!eventMap.hasOwnProperty(name)) return
        let callbacks = eventMap[name]
        if (callbacks.length === 0) return
        callbacks.forEach((callback) => {
            callback(data)
        })


    }

    static sub(name, callback) {
        let callbacks = []
        if (eventMap.hasOwnProperty(name)) {
            callbacks = eventMap[name]
        }
        callbacks.push(callback)
        eventMap[name] = callbacks
    }
}

export default MyEvent
```
该框架只有两个方法，和一个事件队列，`pub`用来发布一个事件，`sub`用来订阅一个事件，调用`pub`发布一个事件以后，`sub`了该事件的方法就会被执行
```
MyEvent.sub('EventA',(data)=>{console.log(data)})
MyEvent.pub('EventA',{num:1})
// 查看控制台
{num:1}
```
- 编写组件
```

const EVENT_BUTTON_CLICK = 'EVENT_BUTTON_CLICK'

class MyView extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 0
        }
        MyEvent.sub(EVENT_BUTTON_CLICK, (num) => {
            this.setState({
                num: num.num
            })
        })

    }

    render() {
        return <p>点击了 {this.state.num} 次</p>
    }
}

class MyButton extends React.Component {

    constructor() {
        super()
        this.state = {
            num: 0
        }
    }

    handleOnClick() {
        let num = ++this.state.num
        this.setState({
            num: num
        })
        MyEvent.pub(EVENT_BUTTON_CLICK, {num: num})
    }

    render() {
        return <button onClick={() => this.handleOnClick()}>{this.props.text}</button>
    }
}

class App extends React.Component {
    constructor() {
        super()
    }

    render() {
        return <div>
            <MyView/>
            <MyButton text='确认'/>
        </div>
    }


}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)

```

可以看到，我们使用的是`发布-订阅模型`，并没有通过父组件来做兄弟组件的传递中介，没有太多的`props`嵌套。所以这就解决了远程组件通信、深度组件嵌套的问题。当然也带来了另一个问题，那就是事件维护。如果事件太多，将会导致无法维护和流程混乱......不可乱用，可用于系统的中心事件控制，也就是单一的事件源输出。

组件间通信可以选择像`redux`这样的库来处理。

### 0x005 资源

- [react](https://reactjs.org/)
- [源码](https://github.com/followWinter/react-study)
  [1]: /img/bVbggAM
  [2]: /img/bVbggC4
