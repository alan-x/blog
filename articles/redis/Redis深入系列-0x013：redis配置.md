`Redis`允许以没有配置文件的方式启动，他将会使用内置的默认配置，但是这种方式推荐只用来测试和开发。
最好的方式是提供一个`Redis`配置文件给`Redis`，通常命名为`redis.conf`。
`redis.conf`通常包含了一系列的指令，它们的格式很简单：
```
keyword argument1 argument2 ... argumentN
```
这是一个配置指令的示例：
```
slaveof 127.0.0.1 6380
```
当然，提供一个包含空白符的参数也是可以的，不过需要使用引号包裹，就像：
```
requirepass "hello world"
```
配置指令列表和每个指令的意义都写在他们自己的`redis.conf`文件中，通常被打包到`Redis`发行。
- [Redis 4.0 配置][1]
- [Redis 3.2 配置][2]
- [Redis 3.0 配置][3].
- [Redis 2.8 配置][4].
- [Redis 2.6 配置][5].
- [Redis 2.4 配置][6].

通过命令行传输参数
从`Redis 2.6`开始，就可以使用命令行直接传输配置参数。这对于测试来说非常有用，下面这个示例显示了如何启动一个新的使用`6380`端口的`Redis`实例作为另一个`host`为`127.0.0.1`端口为`6379`的实例的`slave`：
```
./redis-server --port 6380 --slaveof 127.0.0.1 6379
```
通过命令行参数形式传输配置的方式和提供一个`redis.conf`文件没有什么不同，唯一不同的就是`keyword`前面有`--`。
注意：`Redis`内部内存会生成一个临时配置文件，大部分的参数翻译成`redis.conf`形式的


在`Redis`服务端运行的时候改变`Redis`配置。

使用`CONFIG SET`和`CONFIG GET`命令可以在不停止`Redis`运行或者重启服务的情况下，重新配置`Redis`或者查询`Redis`当前配置。

并不是所有的配置指令都支持这种方式，但是大部分都支持。可以去`CONFIG SET`和`CONFIG GET`了解更多详情。

要注意的是在`Redis`在运行的时候改变它的配置不会影响到`redis.conf`文件，在下次启动的时候`Redis`将继续使用旧的配置。

确保修改`redis.conf`文件和你使用`CONFIG SET`一致。你可以手动，也可以使用`Redis 2.8`,你可以使用`CONFIG REWRITE`，他可以自动扫描你的`redis.cong`文件，并在配置的值错误的时候自动更新相应的域。域如果不存在但是设置为默认值将不会被添加。你配置文件中的注释将被保留。

配置`Redis`为缓存
Configuring Redis as a cache
如果你打算将`Redis`当做缓存并且每个`key`都将有一个过期时间，你最好考虑一下下面的配置：
```
maxmemory 2mb
maxmemory-policy allkeys-lru
```
在这个配置中，不需要使用`EXPIRE`为应用程序设置过期时间，因为当我们命中超过`2MB`内存限制的时候，所有的`key`都会被近似`LRU`算法驱逐。
一般来说，在这个配置下，`redis`表现的像`memcached`，我们还有更多关于将`Redis`作为`LRU cache`的文档。


  [1]: https://raw.githubusercontent.com/antirez/redis/4.0/redis.conf
  [2]: https://raw.githubusercontent.com/antirez/redis/3.0/redis.conf.
  [3]: https://raw.githubusercontent.com/antirez/redis/3.0/redis.conf
  [4]: https://raw.githubusercontent.com/antirez/redis/2.8/redis.conf
  [5]: https://raw.githubusercontent.com/antirez/redis/2.6/redis.conf
  [6]: https://raw.githubusercontent.com/antirez/redis/2.4/redis.conf
