### 0x000 概述
`WeakSet`和`Set`差不多，但是有一些不同：
- `WeakSet`只能存储对象，不能存储任意值
- `WeakSet`不可迭代
- `WeakSet`是弱引用，也就是如果没有变量引用`WeakSet`内的值，很容易被回收

### 0x001 初始化
```
 new WeakSet([iterable]);
```
因为只能存储对象，所以这里我想只能传入类似对象数组之类的东西
- 对象数组
```javascript
    new WeakSet([{name:1},{name:2}])
```
### 0x002 添加
```
let weakset=new WeakSet()
weakset.add({num:1})
weakset.add({num:2})
```
### 0x003 判断是否已经有了
```
let data={num:1}
let weakset=new WeakSet()
weakset.add(data)
weakset.add({num:2})
weakset.has(data) //true
weakset.has({num:2}) //false
```
### 0x004 删除
```
let data={num:1}
let weakset=new WeakSet()
weakset.add(data)
weakset.add({num:2})
weakset.delete(data) //true
weakset.delete({num:2}) //false

```
### 0x005 弱引用特性
```
let weakset=new WeakSet([{num:1}])
setTimeout(()=>console.log(weakset),3000)
// 3s 后输出，可以看到，数据没了
WeakSet {}
```
