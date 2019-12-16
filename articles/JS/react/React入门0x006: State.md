### 0x000 概述
这一章讲`state`，`state`是`MVVM`的核心，也算是`React`的核心思想......都很核心啊。

### 0x001 问题
在上一章节的栗子中，我们取出一个栗子稍作修改：
```
class App extends React.Component {
    constructor() {
        super()
    }

    render() {
        return <p>{this.props.date}</p>
    }
}

let date=Date()

ReactDom.render(
    <App date={date}></App>,
    document.getElementById('app')
)
```
查看浏览器：

![clipboard.png](/img/bVbfVsu)

已经知道了如何渲染数据，但是怎样更新数据呢？比如我希望栗子中的`date`根据时间变化，我们可以如下做：
```
setInterval(() => {
    date = Date()
    ReactDom.render(
        <App date={date}></App>,
        document.getElementById('app')
    )
}, 1000)
```
但是`React`提供了一个更优雅的方式，那就是`state`。
### 0x002 `state`
看栗子：
```

class App extends React.Component {
    constructor() {
        super()
        this.state = {
            date: Date()
        }
        setInterval(() => {
            this.setState({
                date: Date()
            })
        }, 1000)
    }

    render() {
        return <p>{this.state.date}</p>
    }
}
ReactDom.render(
    <App></App>,
    document.getElementById('app')
)

```
我们声明了一个`state`为`{date:Date()}`，然后绑定到视图上，这样视图就显示了`state.date`了，这是毋庸置疑的，我们一直都是这么做。但是接着我们又搞了一个定时器，每秒执行一直`setState`函数，将`date`修改为最新的时间。就完成了视图的更新。

### 0x003 `setState`
查看`setState`[文档](https://reactjs.org/docs/react-component.html#setstate)：
```
React.Component.setState(updater[,callback])
- updater: 更新的数据
- callback: 可选，更新之后的回调
```
该函数是由`React.Component`提供的，而我们在声明该组件的时候继承了`Component`，所以也就有了这个方面。
参数一是要更新的数据，我们只需要传输我们要更新的数据即可，对于不需要更新的数据，则不需要理睬。
参数二是可选的，接受一个回调函数，因为该方法是异步的，如果需要再数据更新完成之后再执行某些操作，可以在这里完成

我们可以这么理解这个函数，我们是这么调用的:
```
this.setState({
    date: Date()
})        
```
我们假想它在执行的时候是这么执行的
```
this.state.date=Date()
this.render()
```
还可以假想他的实现是这么实现的：
```
class Component{
    ...
    setState(updator, callback){
        this.state={...this.state, ...updator}
        this.callback?callback():null
    }
    ...
}
```
当然不管是`setState`的执行流程，还是完整声明，都不是这样的，但是现在我们都不必深入，只是为了简单而这么理解而已。
### 0x004 资源

- [react](https://reactjs.org/)
- [transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx)
- [源码](https://github.com/followWinter/react-study)
