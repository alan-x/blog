### 0x001 特殊模式概述

目前为止，我们使用了`redis-cli`两种主要模式:

- 使用命令行执行`Redis`命令
- 类`REPL`交互模式

下一章节将会解释`Redis`怎样执行其他辅助任务：

- 持续监控`Redis`状态的监控工具
- 大体积`key`搜索
- `key`模糊匹配搜索
- 像发布/订阅客户端一样订阅通道
- 监控`Redis`实例执行的所有命令
- 用不同的方式检测延迟
- 检查本地系统的调度延迟
- 从远程`Redis`实例传输`RDB`备份到本地
- 扮演`Redis slave`角色，显示`slave`接收到了什么命令
- 模拟`LRU`工作演示`key`命中
- 一个`lua`调试客户端


### 0x002 持续监控状态模式
这可能是`redis-cli`其中一个很少人知道的特性了，一个实时监控`Redis`实例的工具。可以通过`--stat`选项来启用它，这种模式中可以很清楚的看到`redis-cli`的行为：
```

$ redis-cli --stat
------- data ------ --------------------- load -------------------- - child -
keys       mem      clients blocked requests            connections
506        1015.00K 1       0       24 (+0)             7
506        1015.00K 1       0       25 (+1)             7
506        3.40M    51      0       60461 (+60436)      57
506        3.40M    51      0       146425 (+85964)     107
507        3.40M    51      0       233844 (+87419)     157
507        3.40M    51      0       321715 (+87871)     207
508        3.40M    51      0       408642 (+86927)     257
508        3.40M    51      0       497038 (+88396)     257
```


这种模式中，每一秒就会输出一次非常有用的信息，通过这些信息你可以很简单的就知道内存占用、客户端连接等信息。


`-i <interval>`选项可以指定多长时间输出一次，默认`1`秒。

### 0x003 大体积`key`扫描

在这种模式中，`redis-cli`分析`key`的空间，他在扫描大体积`key`的同时也提供了关于组成数据库的数据类型的信息。可以使用`--bigkeys`来启用它：
```
$ redis-cli --bigkeys

# Scanning the entire keyspace to find biggest keys as well as
# average sizes per key type.  You can use -i 0.1 to sleep 0.1 sec
# per 100 SCAN commands (not usually needed).

[00.00%] Biggest string found so far 'key-419' with 3 bytes
[05.14%] Biggest list   found so far 'mylist' with 100004 items
[35.77%] Biggest string found so far 'counter:__rand_int__' with 6 bytes
[73.91%] Biggest hash   found so far 'myobject' with 3 fields

-------- summary -------

Sampled 506 keys in the keyspace!
Total key length in bytes is 3452 (avg len 6.82)

Biggest string found 'counter:__rand_int__' has 6 bytes
Biggest   list found 'mylist' has 100004 items
Biggest   hash found 'myobject' has 3 fields

504 strings with 1403 bytes (99.60% of keys, avg size 2.78)
1 lists with 100004 items (00.20% of keys, avg size 100004.00)
0 sets with 0 members (00.00% of keys, avg size 0.00)
1 hashs with 3 fields (00.20% of keys, avg size 3.00)
0 zsets with 0 members (00.00% of keys, avg size 0.00)
```


在输出的第一部分，报告中相同类型下的每个新的的`key`体积都大于上一个大体积`key`。`summary`部分提供了`Redis`实例中数目的基本信息


这个模式使用`SCAN`命令，所以他可以直接执行而不影响`Redis`实例的运行状态，当然`-i`选项可以通过指定秒的小数来限制每100个`key`扫描的进程数。比如，`-i 0.1`将会极大的放慢扫描速度，但是将会较少服务器负载到一个很小的程度。


注意`summary`也提供了一个格式清晰的报告来显示它找打的大体积`key`。如果在一个大数据集中使用这个命令，初始的输出只提供了一些有趣的信息。

### 0x005 获取`key`列表

在使用像`KEYS *`的时候将会阻塞`Redis`服务端，当然在不阻塞`Redis`服务端的同时扫描`key`空间也是可以做到的，它可以答应出所有的`key`的名称，甚至可以指定模式匹配。这种模式中，像`--bigkeys`选项，使用`SCAN`命令，如果数据集一直在改变，`key`可能会多次的出现，但是只要`key`在开始遍历的时候存在，就不会被遗漏。因为这个命令使用的选项叫做`--scan`

```
$ redis-cli --scan | head -10
key-419
key-71
key-236
key-50
key-38
key-458
key-453
key-499
key-446
key-371
```

注意`head -10`是为了获取头10个key


可以通过` --pattern`选项来启用模糊匹配
```
$ redis-cli --scan --pattern '*-11*'
key-114
key-117
key-118
key-113
key-115
key-112
key-119
key-11
key-111
key-110
key-116
```


管道输出到`wc`命令可以用来统计指定类型的对象的数量：
```
$ redis-cli --scan --pattern 'user:*' | wc -l
3829433
```
### 0x006 发布/订阅模式




`redis-cli`能够使用`PUBLISH`命令在`Redis`发布/订阅通道中发布消息。`PUBLISH`命令的使用和其他命令很像。但是为了接受消息而去订阅通道--这种情况下我们要阻塞并等待消息，所以这是`redis-cli`的一种特俗模式。这个模式不像其他特殊模式一样通过其他特殊的选项启用，只是简单的使用`SUBSCRIBE`或者`PSUBSCRIBE`命令，在交互模式或者非交互模式都可以使用

```
$ redis-cli psubscribe '*'
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "*"
3) (integer) 1
```
`Reading messages...`标志我们进入了发布/订阅模式。当其他客户端在通道中发布消息的时候，比如你可以使用`redis-cli`执行`PUBLISH mychannel mymessage`，发布订阅模式下的命令行窗口将会显示一些信息：

```
1) "pmessage"
2) "*"
3) "mychannel"
4) "mymessage"
```

这对于调试发布/订阅模式非常有益，退出这种模式可以按`CTRL-C`

### 0x007 监控`Redis`中执行的命令

和发布订阅模式很想，你使用`MOBITOR`模式将会自动进入这个监控模式，它将会输出`Redis`接收到的所有命令。

```
$ redis-cli monitor
OK
1460100081.165665 [0 127.0.0.1:51706] "set" "foo" "bar"
1460100083.053365 [0 127.0.0.1:51707] "get" "foo"
```

注意：这里可以使用管道操作符重定向结果，所以你可以使用类似`grep`的工具去监控指定模式的命令。


### 0x008 监控`Redis`实例的延迟
`redis`对延迟是非常挑剔的，延迟涉及到应用中非常多的可移动部分，从客户端库到网络栈，再到`Redis`实例本身。

`redis-cli`有很多机制可以知道`Redis`实例的延迟的最大值、平均值和分配
最基本的延迟检测工具是`--latency`选项，使用这个选项将会不停的发送`PING`命令到`Redis`实例，返回的时间将会被统计。每秒100次的时候，状态实时更新在控制台将会如下显示：

```
$ redis-cli --latency
min: 0, max: 1, avg: 0.19 (427 samples)
```
这个状态的单位是毫秒。通常情况下，在一个非常快的实例中，平均延迟是被过分估计的，因为言辞取决于系统运行`redis-cli`自身的内核调度，所以`0.19`的平均延迟可以看做是`0.01`或者更低。不管如何，这通常不是什么大问题，我们通常关心几毫秒或者更多。


有时候我们更关心最大延迟和平均延迟随着时间是如何变化的，`--latency-history`选项提供给我们这种实现：他运行的像`--latency`，但是默认情况下每`15s`，新的对话将会被抓取：

```
$ redis-cli --latency-history
min: 0, max: 1, avg: 0.14 (1314 samples) -- 15.01 seconds range
min: 0, max: 1, avg: 0.18 (1299 samples) -- 15.00 seconds range
min: 0, max: 1, avg: 0.20 (113 samples)^C
```
你可以通过`-i <interval>`选项来修改每次对话抓取的时间

大多数高级的延迟分析工具，都非常努力的像没有经验的用户展示一个带有颜色的频谱图。你可以看到被颜色渲染的输入显示了不同示例的百分比，不同的`ASCII`字符来表示不同的延迟图像。可以通过`--latency-dist`选项来使用：

```
$ redis-cli --latency-dist
```

![clipboard.png](/img/bVbaVMT)


这是`redis-cli`中另一个非常漂亮的不常用的延迟工具。他不检测`redis-cli`实例的延迟，它检测的是你运行`redis-cli`电脑的延迟，如果你要问什么延迟？这延迟是内核调度固有的延迟，虚拟架构实例的管理程序等。


我们叫它固有延迟是因为他对开发者是不透明的，通常情况下，你排除了所有的可观察的原因，但是你的`Redis`实例依旧有很高的延迟，那基本就是固有延迟引起的，在你运行`Redis`服务端的电脑上，运行这个特殊的模式是非常有意义的。


通过测量固有延迟，你将会知道基线在哪里，`Redis`不可能超过你的系统。`--intrinsic-latency <test-time>`选项可以启动这种模式，`<test-time>`单位是秒：
```
$ ./redis-cli --intrinsic-latency 5
Max latency so far: 1 microseconds.
Max latency so far: 7 microseconds.
Max latency so far: 9 microseconds.
Max latency so far: 11 microseconds.
Max latency so far: 13 microseconds.
Max latency so far: 15 microseconds.
Max latency so far: 34 microseconds.
Max latency so far: 82 microseconds.
Max latency so far: 586 microseconds.
Max latency so far: 739 microseconds.

65433042 total runs (avg latency: 0.0764 microseconds / 764.14 nanoseconds per run).
Worst run took 9671x longer than the average latency.
```

重要：这个命令必须运行在你想要运行`Redis`服务端的电脑上，而不是其他主机，它不会链接到`Redis`实例，只会测试本地。
在上面的例子中，我系统的最高延迟是`739`微妙，所以我可以确定每次执行的延迟不会高于`1`毫秒

### 0x009 远程备份`RDB`文件
当`Redis`复制集第一次同步时，主从双机通过`RDB`文件交换数据集。这个特性利用`redis-cli`提供一个远程备份机制，允许从`Redis`传输`RDB`文件到本地。使用这个模式，可以使用`--rdb <dest-filename>`参数：

```
$ redis-cli --rdb /tmp/dump.rdb
SYNC sent to master, writing 13256 bytes to '/tmp/dump.rdb'
Transfer finished with success.
```
这是一个保证你有从你的`Redis`实例做灾难恢复`RDB`备份的简单且行之有效的方式。当在脚本或者`cron`使用这个选项时候，请检查命令的返回值适不适合`0`，如果不是`0`，说明有错误发生：


```
$ redis-cli --rdb /tmp/dump.rdb
SYNC with master failed: -ERR Can't SYNC while not connected with my master
$ echo $?
1
```
### 0x010 `slave`模式
Slave mode
对于`Redis`开发者和调试操作来说，从模式是一个高级特性。它允许查看一个`master`在复制流传输的时候给他的`slave`发送了什么。可以使用`--slave`来启用它：
```
$ redis-cli --slave
SYNC with master, discarding 13256 bytes of bulk transfer...
SYNC done. Logging commands from master.
"PING"
"SELECT","0"
"set","foo","bar"
"PING"
"incr","mycounter"
```
这个命令一开始先抛弃了第一次同步的`RDB`文件，接着以`CSV`格式记录每一次收到的命令。
如果你发现并不是所有的命令都正确的复制到你的`slave`机子上，这将是一个检查错误非常好的方式，对于改善`bug`发现也是非常有用的信息。

### 0x011 执行 LRU 模拟器

`Redis`经常被用来做`LRU`缓存回收。取决于`key`的数量和为缓存申请的内存总量（通过`maxmemory`指定），缓存命中和缓存失效的总量将会改变。有时，模拟命中率是正确提供缓存非常有效的方式。

`redis-cli`提供了一个特俗的方式去执行一个`GET`和`SET`的模拟操作，用`80%`-`20%`的请求比例分布。这意味着`20%`的key将会在`80%`的请求内被请求，这在缓存方案中是一个很常见的分布。

从理论上讲，给`Redis`的请求分发达到`Redis`的内存峰值，就可能在数理上算出命中率。然而`Redis`可以配置不同的`LRU`设置(数量或者示例)和`LRU`的执行，这几乎在`Redis`中，每个版本都有很大的改变。同样的，每个`key`所占用的空间在每一个版本也可能不同。这就是这个工具被创建的理由：他主要的动机是为了测试`Redis``LRU`的执行质量，但是现在在测试给定版本给定配置中也很有用，在你的开发中要记住。

为了使用这个模式，你需要指定测试中`key`的数量。同时，在第一次测试中，你要正确配置`maxmemory`。

重要提示：配置`maxmemory`设置在`Redis`配置中是非常重要的：如果没有设置最大内存使用量，命中率最终将达到`100%`，因为所有的`key`都可以配存储到内存中。如果你设置了太多的`key`但是没有最大内存，最终，电脑所有的`RAM`江北使用，同样你要配置最大内存策略，大部分时间你要的是`allkeys-lru`。

接下来的例子我将内存限制为`100MB`，`LRU`模拟器使用1千万个`key`

警告：这个测试使用的管道，将会给服务器带来压力，不要在生产环境中的实例使用。
```
$ ./redis-cli --lru-test 10000000
156000 Gets/sec | Hits: 4552 (2.92%) | Misses: 151448 (97.08%)
153750 Gets/sec | Hits: 12906 (8.39%) | Misses: 140844 (91.61%)
159250 Gets/sec | Hits: 21811 (13.70%) | Misses: 137439 (86.30%)
151000 Gets/sec | Hits: 27615 (18.29%) | Misses: 123385 (81.71%)
145000 Gets/sec | Hits: 32791 (22.61%) | Misses: 112209 (77.39%)
157750 Gets/sec | Hits: 42178 (26.74%) | Misses: 115572 (73.26%)
154500 Gets/sec | Hits: 47418 (30.69%) | Misses: 107082 (69.31%)
151250 Gets/sec | Hits: 51636 (34.14%) | Misses: 99614 (65.86%)
```
这个程序每秒显示一次状态，就像你看到的，在第一秒的时候缓存开始填充。一段时间后缓存失效率渐渐稳定下来：
```
120750 Gets/sec | Hits: 48774 (40.39%) | Misses: 71976 (59.61%)
122500 Gets/sec | Hits: 49052 (40.04%) | Misses: 73448 (59.96%)
127000 Gets/sec | Hits: 50870 (40.06%) | Misses: 76130 (59.94%)
124250 Gets/sec | Hits: 50147 (40.36%) | Misses: 74103 (59.64%)
```
在我们的使用场景中`59%`的缓存失效率是不被接受的。所以我们知道`100MB`的内存是不够的。让我们阐释一下半个`G`，几分钟后我们将看到如下输出：
```
140000 Gets/sec | Hits: 135376 (96.70%) | Misses: 4624 (3.30%)
141250 Gets/sec | Hits: 136523 (96.65%) | Misses: 4727 (3.35%)
140250 Gets/sec | Hits: 135457 (96.58%) | Misses: 4793 (3.42%)
140500 Gets/sec | Hits: 135947 (96.76%) | Misses: 4553 (3.24%)
```
所以我们知道 `500MB`是满足我们1千万个`key`和`80-20`分布使用的。


  [1]: https://redis.io/topics/rediscli
