### 0x000 概述
上一章讲咯生命周期，这一章就是事件处理咯

### 0x001 事件绑定

```
// 绑定一个外部函数
function handleClick(event) {
    console.log('handleClick', event)
}

class App extends React.Component {

    render() {
        return <button onClick={handleClick}>
            按钮
        </button>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
```
// 绑定一个内部函数
class App extends React.Component {

    handleClick(event) {
        console.log('handleClick', event, this)
    }

    render() {
        return <button onClick={this.handleClick}>
            按钮
        </button>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
### 0x002 解决函数绑定的`this`问题
上面的栗子有个问题：在`handleClick`内无法访问`App`内的资源，比如`this.state`
```

class App extends React.Component {
    constructor(){
        super()
        this.state={
            name:1
        }
    }

    handleClick(event) {
        console.log('handleClick', event, this.state.name)
    }

    render() {
        return <button onClick={this.handleClick}>
            按钮
        </button>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```

![clipboard.png](/img/bVbf4L1)
可以这么解决这个问题：
```
class App extends React.Component {
    constructor() {
        super()
        this.state = {
            name: 1
        }
    }

    handleClick(event) {
        console.log('handleClick', event, this.state.name)
    }

    render() {
        return <button onClick={(e) => this.handleClick(e)}>
            按钮
        </button>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
或者
```

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            name: 1
        }
        this.handleClick = this.handleClick.bind(this)
    }

    handleClick(event) {
        console.log('handleClick', event, this.state.name)
    }

    render() {
        return <button onClick={this.handleClick}>
            按钮
        </button>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
或者
```
class App extends React.Component {
    constructor() {
        super()
        this.state = {
            name: 1
        }
    }

    handleClick = (event) => {
        console.log('handleClick', event, this.state.name)
    }

    render() {
        return <button onClick={this.handleClick}>
            按钮
        </button>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```
第三中方式需要`babel-plugin-transform-class-properties`支持：
```
$ npm install --save-dev babel-plugin-transform-class-properties
$ vim .babelrc
{
  "presets": [
    "env"
  ],
  "plugins": [
    "transform-react-jsx",
    "transform-class-properties"
  ]
}
```
### 0x003 做几个栗子
1. 栗子：计数器
```
class App extends React.Component {
    constructor() {
        super()
        this.state = {
            num: 1
        }
    }

    handleClick(event) {
        this.setState({
            num: ++this.state.num
        })
    }

    render() {
        return <div>
            <button onClick={() => this.handleClick()}>
                按钮
            </button>
            <p>{this.state.num}</p>
        </div>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```

![图片描述][1]

2. 栗子2：`tab` 切换
```

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            tab: 1
        }
    }

    handleClick(tab) {
        this.setState({
            tab: tab
        })
    }

    render() {
        return <div>
            <button onClick={() => this.handleClick(1)}>
                tab1
            </button>
            <button onClick={() => this.handleClick(2)}>
                tab2
            </button>
            {
                this.state.tab === 1
                    ? <div>tab1</div>
                    : <div>tab2</div>
            }
        </div>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)
```

![图片描述][2]

### 0x004 事件说明
- 事件相关的属性名为`on`开头的驼峰写法，和`html`中的事件一致


### 0x005 资源

- [react](https://reactjs.org/)
- [源码](https://github.com/followWinter/react-study)
    



  [1]: /img/bVbf4OQ
  [2]: /img/bVbf4Pl
