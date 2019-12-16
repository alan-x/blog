### 延迟
从高层次来看，延迟可以认为是需要长时间完成的异步操作的表示。他是阻塞函数的异步的替代方法，通常来说，与其让应用应为等到某个请求完成返回一个结果而堵塞，不如立马返回一个延迟对象。你可以为延迟对象绑定一个回调：它将会在请求真的完成的时候调用。

### 承诺
在最基本的形式中，一个“承诺”是一个软件工程模型，提供一个延迟（或未来）的概念的解决方案。它背后的主要思想是我们已经见过的：与其执行一个将会导致堵塞的调用，不如返回一个未来值将被满足的承诺
如果这里需要一个栗子，假设你正在构建一个 web 应用，它非常依赖第三方的 API。一个基本的将要面对的问题是不知道 API 服务的延迟，所以你的应用的其他部分可能堵塞，直到结果返回。延迟提供了一个解决这个问题更好的解决方案，它避免了“堵塞”效果并完全分离
[Promise/A]()协议定义了一个方法叫做`then`，可以用来注册注册回调到某个承诺，在可获取的时候获取未来的值。解决第三方 API 返回一个承诺的伪代码看起来可能是这样的：
```
var promise = callToAPI( arg1, arg2, ...);
 
promise.then(function( futureValue ) {
 
    // Handle futureValue
 
});
 
promise.then(function( futureValue ) {
 
    // Do something else
 
});
```
此外：一个承诺实际上可以以两种不同的状态结束：
- `Resolved`：这种情况下值可以获取
- `Rejected`：这种情况下出错了，并且没有值可以获取
谢天谢地，`then`方法接收两个参数：一个是当承诺被解决的时候，一个是当承诺被拒绝的时候。如果我们使用伪代码表示，可能是这么做：
```
promise.then(function( futureValue ) {
 
    // We got a value
 
}, function() {
 
    // Something went wrong
 
});
```
在实际应用场景中，在多个结果没有返回之前，应用无法继续（比如，在用户可以选择他们感兴趣的选项的之前在屏幕上显示一系列的选项）。`when`方法的存在可以解决这个场景，所有的承诺一旦被解决，它可以用来行一些操作。
```
when(
    promise1,
    promise2,
    ...
).then(function( futureValue1, futureValue2, ... ) {
 
    // All promises have completed and are resolved
 
});
```
一个可能的栗子是你可能有多个同时运行的动画。不跟踪每一个完成回调，很难确认你的动画已经完成了。使用承诺和`when`，你的每一个动画都可以非常直接的说“一旦我们运行完成，我们承诺让你知道”。这些组合的结果意味一旦动画哦借宿，将会执行一个回调。比如：
```
var promise1 = $( "#id1" ).animate().promise();
var promise2 = $( "#id2" ).animate().promise();
when(
    promise1,
    promise2
).then(function() {
 
    // Once both animations have completed
    // we can then run our additional logic
 
});
```
这意味着可以编写同步运行的无堵塞的逻辑。直接传递回调给函数，会导致接口紧密耦合，使用承诺允许我们将同步或异步代码的关注点分离。
- [jQuery 延迟]()
- [延迟栗子]()
