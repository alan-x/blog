### 0x001 `redis-cli`--`Redis`命令行接口
`redis`是一个简单的命令行接口程序，他允许你在终端直接向`Redis`发送命令，并且读取`Redis`返回的数据。

它有两种模式：
    - 交互模式（`REPL`）：用户输入命令，获取结果。
    - 参数模式：命令作为`redis-cli`的参数传输，执行，并且以标准输出流输出

在交互模式中，redis-cli的只能提示提供了一个非常好的输入体验。

然而，`redis-cli`不仅仅能做到这样，它有非常多的选项让你选择，可以通过不同的选项让他进入某种特殊的模式。所以，`redis-cli`肯定是可以做非常复杂的任务的，比如模拟`master`复制到`slave`的数据同步，并且将同步流点出来。检查`Redis`服务端的潜在危险并且作出统计，设置可以以`ASCII-art`形式显示潜在危险案例和概率，初次之外还有许多功能。

这个指南将会从最简单的特性开始，并且以将会以高级特性结束，覆盖了`redis-cli`各个不同的方面。

如果你想广泛使用`Redis`，又或者你已经在这么做了，那么这篇文章将有很大的可能让你得到这种机会。花一些时间熟悉这些特性可能是一个非常好的注意，如果你知道`redis-cli`命令行接口诀窍，你将会看到如何更有效的使用Redis。

### 0x002 命令行使用

将命令当做`redis-cli`分离的参数来执行命令非常的简单，同时它可以在屏幕上以标准输出命令执行的结果
```
$ redis-cli incr mycounter
(integer) 7
```
命令的返回值是`7`，因为`Redis`的返回值是类型化的（`strings`、`arrays`、`integers`、`NULL`、`errors`等），你将会在返回结果中看到类型，他们在两个括号之间，所以讲Redis的返回值作为另一个命令的输入值不是一个好主意，除非我们想将结果重定向到文件中。

通常情况下，`redis-cli`如果在`ttf`模式中，将会显示一些而外的消息，以增强阅读性。在其他模式中，将会自动使用原始输出，就像接下来的例子

```
$ redis-cli incr mycounter > ./output
$ cat ./output 
2
```
此时，`(integer)`将会被忽略，因为`redis-cli`发现并不是在终端上输出，当然，我们也可以强制在终端上直接显示原始信息，使用`--raw`参数
```
$ redis-cli --raw incr mycounter
3
```
同理，你可以强制在输入文件或者其他管道中启用增强模式，使用`--no-raw`
```
$ redis-cli --no-raw incr mycounter > ./output
FollowWinter:~ FollowWinter$ cat output 
(integer) 4
```
### 0x003 主机，端口，密码，数据库



默认情况下，`redis-cli`链接的服务端主机地址是`127.0.0.1:6379`，就像你想的那样，通过命令行，你可以简单的修改这些参数，去指定一个不同的主机名字或者`IP`，使用`-h`，指定不同的端口，使用`-p`。

```
$ redis-cli -h redis15.localnet.org -p 6390 ping
PONG
```

如果你的实例是受密码保护的，`-a <password>`选项将会实现认证，明企鹅不需要再显示的调用AUTH命令。
```
$ redis-cli -a myUnguessablePazzzzzword123 ping
PONG
```

最后，不使用默认的0号数据库而使用其他数据库也是可能的，可以使用` -n <dbnum>`选项：
```
$ redis-cli flushall
OK
$ redis-cli -n 1 incr a
(integer) 1
$ redis-cli -n 1 incr a
(integer) 2
$ redis-cli -n 2 incr a
(integer) 1
```
这些信息也可以以`URI`的形式给出，使用` -u <uri>`选项
```
$ redis-cli -u redis://p%40ssw0rd@redis-16379.hosted.com:16379/0 ping
PONG
```
### 0x004 从其他输入获取输入数据

有两种方式可以从其他程序获取数据（一般从标准输入）。一种方式是将从标准输入读取的数据装载到`redis-cli`的最后一个选项。比如，我想将我电脑上的`/etc/services`文件内容作为一个`key`的值，我可以使用-x选项：

```
$ redis-cli -x set foo < /etc/services
OK
$ redis-cli getrange foo 0 50
"#\n# Network services, Internet style\n#\n# Note that "
```

就想你再上面的对话中的第一行看到的，`SET`命令的最后一个选项并没有指定。这个参数并没有直接把`key`设置成我想要设置的值。

作为替代，-x选项指定了一个文件，并被重定向为`CLI`的标准输入，所以，输入被读取，并且被当做命令的最后一个参数，这对脚本很很有用。

另一种方式是将一长串的命令写入文件，直接喂给`redis-cli`
```
$ cat /tmp/commands.txt
set foo 100
incr foo
append foo xxx
get foo
$ cat /tmp/commands.txt | redis-cli
OK
(integer) 101
(integer) 6
"101xxx"
```
所有在这个文件中的命将会一个接着一个的执行，就像是用户输入一样。字符串在必要的时候，可以使用引号包裹，所以可以使用特殊的符号，比如空格、换行或者其他特殊字符：

```
$ cat /tmp/commands.txt
set foo "This is a single argument"
strlen foo
$ cat /tmp/commands.txt | redis-cli
OK
(integer) 25
````
### 0x005 连续执行多次相同命令

在执行时用户连续多次执行相同的命令是很有可能的。在不同的上下文中，这是非常有用的，比如当我们需要连续监控一些`key`的内容或者输出内容，或者我们需要模拟一些重复写入事件（比如每5s向一个`list`推入数据）

这个特性受控于两个选项：`-r <count>`和`-i <delay>`。第一个标明要执行多少次，第二个配置两次命令执行的延迟事件，单位是秒（但是也可以指定像`0.1`这样的小数，用来表示`100`毫秒）。

默认延迟是`0`，所以命令会尽可能快的执行
```
$ redis-cli -r 5 incr foo
(integer) 1
(integer) 2
(integer) 3
(integer) 4
(integer) 5
```
如果要永远执行某个命令，count选项值为-1，如果为了一直监控RSS的内存大小，或许可以这样使用命令：

````
$ redis-cli -r -1 -i 1 INFO | grep rss_human
used_memory_rss_human:1.38M
used_memory_rss_human:1.38M
used_memory_rss_human:1.38M
... a new line will be printed each second ...
```
### 0x006 使用`redis-cli`插入混合的数据

使用`redis-cli`插入混合的数据是一个重要的课题，将会在独立的文章说明，请阅读[混合数据插入指南][2]

### 0x007 导出CSV文件

有时候，你需要使用`redis-cli`快速的导出`Redis`中的数据给其他程序使用，可以使用`CSV`输出特性：

```
$ redis-cli lpush mylist a b c d
(integer) 4
$ redis-cli --csv lrange mylist 0 -1
"d","c","b","a"
```
此时不可能把整个数据项这样导出，但是只有一个简单的命令就可以做到导出数据到`csv`文件。

### 0x008 执行`lua`脚本


从`Redis 3.2`开始，我们就广泛的支持使用新的`Lua`调试器来调试和编写`Lua`脚本，关于这个特性，可以查阅[Redis Lua 调试文档][3]。


然而，就算没有使用调试器，你也可以使用`redis-cli`从一个文件执行脚本，比在`shell`交互中输入脚本或者作为参数舒服多了

```
$ cat /tmp/script.lua
return redis.call('set',KEYS[1],ARGV[1])
$ redis-cli --eval /tmp/script.lua foo , bar
OK
```
`Redis`的`EVAL`命令携带了脚本需要用到的`key`，另一个却没有`key`参数,当调用`EVAL`的时候，你以数字的方式提供了一个`key`的数字。然而在前面`redis-cli`使用`--eval`选项的时候，不需要特意指定`key`的序号。它性惯性的使用逗号分隔的`key`序列作为替代。这就是为什么你再前面看到`foo, bar`作为参数。

所以 `foo`将会由`KSYS`数组组成，`bar`由`ARGV`数组组成
So foo will populate the KEYS array, and bar the ARGV array.

`--eval`选项在编写简单脚本的时候特别有用。其他复杂的工作，使用`Lua`调试器将会更加舒适，这两种方式可以混合使用，因为调试器也是执行外部脚本文件。


  [1]: https://redis.io/topics/rediscli
  [2]: https://redis.io/topics/mass-insert
  [3]: https://redis.io/topics/ldb
