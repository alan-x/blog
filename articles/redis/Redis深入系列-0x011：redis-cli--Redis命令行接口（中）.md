### 0x001 交互模式

到目前为止，我们探索了如何像使用命令行程序一样使用`redis-cli`。这种方式在使用脚本或者测试的时候的确是一种好的方式，但是大多数人大多使用的是交互模式

用户在交互模式中，用户在提示符的帮助下输入`Redis`命令。命令将会发送到服务端，运行完成之后将会返回并渲染为简单的可阅读性好的输出。


在交互模式中，不需要有什么特别的输入--只需要启动它并且不添加任何参数：
```
$ redis-cli
127.0.0.1:6379> ping
PONG
```

`127.0.0.1:6379>`是提示符，它提示你你已经连接到了`Redis`实例


提示符将会在你的连接发生变化的时候改变，或者你选择操作不是`0`号数据库的时候：
```
127.0.0.1:6379> select 2
OK
127.0.0.1:6379[2]> dbsize
(integer) 1
127.0.0.1:6379[2]> select 0
OK
127.0.0.1:6379> dbsize
(integer) 503
```
### 0x002 处理连接和重新连接



在交互模式中使用连接命令并指定`hostname`和`port`可以实现连接到其他实例：
```
127.0.0.1:6379> connect metal 6379
metal:6379> ping
PONG
```

你可以看到提示符相应的改变了。如果用户尝试连接一个不可达的实例，`redis-cli`将会进入不可连接模式并在每次输入新的命令的时候重新尝试连接一次。

```
127.0.0.1:6379> connect 127.0.0.1 9999
Could not connect to Redis at 127.0.0.1:9999: Connection refused
not connected> ping
Could not connect to Redis at 127.0.0.1:9999: Connection refused
not connected> ping
Could not connect to Redis at 127.0.0.1:9999: Connection refused
```

通常情况下，如果一个连接失败了，`redis-cli`总会在后台尝试重新连接：如果尝试连接失败了，它将会显示错误并进入不可连接状态。下面是例子：
```
127.0.0.1:6379> debug restart
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected> ping
PONG
127.0.0.1:6379> (now we are connected again)
```

当重新连接成功了，`redis-cli`自动重新选择最新的数据库。然而，其他的状态将会丢失，比如我们正在执行的事务：
```
$ redis-cli
127.0.0.1:6379> multi
OK
127.0.0.1:6379> ping
QUEUED

( 此时`redis-server正好在重启` )

127.0.0.1:6379> exec
(error) ERR EXEC without MULTI
```

在交互模式中测试不是议题，但是你必须知道这种模式的局限性。

### 0x003 编辑，历史和自动完成

因为`redis-cli`使用了[linenoise line editing library][2]，所以他具有命令行自动提示和完成能力，并没有依赖`libreadline`或者其他可选的库


为了防止一次又一次的输入重复的命令，你可以按`↑`、`↓`箭头来获取到已经执行过的历史命令。历史命令在重启`redis-cli`的时候将会消失，它保存在`.rediscli_history`文件中，该文件位于用户的`home`文件夹，并定义在环境变量中。可以通过设置`REDISCLI_HISTFILE`环境变量来设定其他的文件，如果要禁用它，请设置到`/dev/null`。

通过`TAB`键，`redis-cli`也可以做到命令名的自动完成：
```
127.0.0.1:6379> Z<TAB>
127.0.0.1:6379> ZADD<TAB>
127.0.0.1:6379> ZCARD<TAB>
```

### 0x004 重复执行相同命令

通过在命令前面添加数字前缀来重复执行命令：

```
127.0.0.1:6379> 5 incr mycounter
(integer) 1
(integer) 2
(integer) 3
(integer) 4
(integer) 5
```
### 0x005 显示命令帮助

`Redis`有很多的命令，当你测试的时候，如果你忘记了参数的准确顺序，`redis-cli`通过`help`为绝大多数的`Redis`命令提供了在线的帮助。使用`help`命令有两种形式：

- `help @<category>`： 显示某个分类的所有命令. 这些分类包括： @generic, @list, @set, @sorted_set, @hash, @pubsub, @transactions, @connection, @server, @scripting, @hyperloglog.
- `help <commandname>`：显示参数中所给命令的帮助 
比如获取`PFADD`命令的帮助：
```

  PFADD key element [element ...]
  summary: Adds the specified elements to the specified HyperLogLog.
  since: 2.8.9
  group: hyperloglog
```
`help`命令同样支持`TAB`来自动完成

### 0x006 清理终端屏幕

在交互模式中使用`clear`命令可以清理终端的屏幕



  [1]: https://redis.io/topics/rediscli
  [2]: http://github.com/antirez/linenoise
