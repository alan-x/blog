0x001：mac上安装
`mac`上安装使用[homebrew][1]即可，安装中有许多重要的信息，所以这里不做省略，并在以下的输出中做一些注释。
```
# 输入
$ brew install redis

# 输出
Updating Homebrew...
==> Downloading https://homebrew.bintray.com/bottles/redis-4.0.9.high_sierra.bot
######################################################################## 100.0%
==> Pouring redis-4.0.9.high_sierra.bottle.tar.gz
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/redis-benchmark
Target /usr/local/bin/redis-benchmark
already exists. You may want to remove it:
  rm '/usr/local/bin/redis-benchmark'

To force the link and overwrite all conflicting files:
  brew link --overwrite redis

To list all files that would be deleted:
  brew link --overwrite --dry-run redis

Possible conflicting files are:
# 这里是一些工具的位置，又需要可以直接从下面路径找到相应工具
/usr/local/bin/redis-benchmark
/usr/local/bin/redis-check-aof
/usr/local/bin/redis-check-rdb
/usr/local/bin/redis-cli
/usr/local/bin/redis-sentinel -> /usr/local/bin/redis-server
/usr/local/bin/redis-server
==> Caveats
To have launchd start redis now and restart at login:
  brew services start redis
Or, if you don't want/need a background service you can just run:
# 这里是启动服务端的方法，其中包含了默认配置文件的位置
  redis-server /usr/local/etc/redis.conf
==> Summary
🍺  /usr/local/Cellar/redis/4.0.9: 13 files, 2.8MB

```
0x002：启动服务端
看到以下`redis`图标图案说明启动成功，并且可以从输出中看出启动的进程号和端口号

```
# 输入
$ redis-server

# 输出
...
# 省略部分输出
...
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379 # 端口
 |    `-._   `._    /     _.-'    |     PID: 72438 # 进程
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

...
...
```
### 0x0003 启动客户端
```
# 输出
$ redis-cli

# 输出，进入`redis`交互界面
127.0.0.1:6379> 
```
### 0x004 测试
```
# 输入
127.0.0.1:6379> PING
# 输出
PONG
```
### 0x005 注意
以上测试都是基于基本配置，不涉及更改配置和远程连接的情况。


  [1]: https://brew.sh/index_zh-cn
