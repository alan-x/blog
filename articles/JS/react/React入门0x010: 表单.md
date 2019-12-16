### 0x000 概述
这一章讲表单处理

### 0x001 栗子
```
import React from 'react'
import ReactDom from 'react-dom'


class App extends React.Component {
    constructor() {
        super()
        this.state = {
            name: '',
            sex: '',
            content: '',
            formData: '',
        }
    }

    handleSubmit(e) {
        e.preventDefault()
        this.setState({
            formData: `name=${this.state.name}&sex=${this.state.sex}&content=${this.state.content}`
        })
    }

    render() {
        return <div>
            <h1>填写表单</h1>
            <form action="" onSubmit={(e) => this.handleSubmit(e)}>
                <div>
                    <label htmlFor="">
                        姓名:
                        <input type="text"
                               value={this.state.name}
                               onChange={(event) => this.setState({name: event.target.value})}
                        />
                    </label>
                </div>
                <div>
                    <label htmlFor="">
                        性别:
                        <select name="sex" id="" value={this.state.sex}
                                onChange={(event) => this.setState({sex: event.target.value})}>
                            <option value="男">男</option>
                            <option value="女">女</option>
                        </select>
                    </label>
                </div>
                <div>
                    <label htmlFor="">
                        简介:
                        <textarea name="" id="" cols="30" rows="10" value={this.state.content}
                                  onChange={(event) => this.setState({content: event.target.value})}>
                    </textarea>
                    </label>
                </div>
                <button type='submit'>提交</button>
            </form>
            <div>
                <hr/>
                <h1>预览</h1>
                <div>
                    姓名:{this.state.name}
                </div>
                <div>
                    性别:{this.state.sex}
                </div>
                <div>
                    简介:{this.state.content}
                </div>
            </div>
            <div>
                <hr/>
                <h1>表单提交</h1>
                <p>{this.state.formData}</p>
            </div>

        </div>
    }
}

ReactDom.render(
    <App></App>,
    document.getElementById('app')
)

```
打开浏览器查看效果：
![图片描述][1]
### 0x002 总结
如果你只给`input`绑定了`value`，会发现无法输入任何内容，因为在`react`中，有**受控组件**的说法，有点不大好理解，直接换种说法比较好，在`form`表单中我们需要完成数据的双向绑定。如果你只给`input`绑定了`value`，那么`state`的数据将会被绑定到`input`上就和你使用`{this.state.name}`一样，但是这只是完成了数据到视图的绑定，我们还好完成视图到数据的绑定，以完成数据的双向流动。形成一个闭环，所以我们要绑定`onChange`事件，并且将`input`输入的值赋值给`state`中对应的键。

这种模式在`react`中，特别是组件嵌套中经常用到，我更愿意称之为数据双向流动、数据流动闭环。而非受控组件......

### 0x003 资源

- [react](https://reactjs.org/)
- [源码](https://github.com/followWinter/react-study)
    

  [1]: /img/bVbf8Jk
