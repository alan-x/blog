### 0x001 订阅一个或者多个频道
> 命令格式：`SUBSCRIBE channel [channel channel ...]`
```
$ redis-cli
# 进程1
127.0.0.1:6379> SUBSCRIBE chat chat2
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "chat"
3) (integer) 1
1) "subscribe"
2) "chat2"
3) (integer) 2
```
### 0x002 发布消息
> 命令格式：`PUBLISH channel message`

```
$ redis-cli
# 进程2
127.0.0.1:6379> PUBLISH chat hellochat1
(integer) 1
127.0.0.1:6379> PUBLISH chat2 hellochat2
(integer) 1
#此时的进程1
127.0.0.1:6379> SUBSCRIBE chat chat2
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "chat"
3) (integer) 1
1) "subscribe"
2) "chat2"
3) (integer) 2
1) "message"
2) "chat"
3) "hellochat1"
1) "message"
2) "chat2"
3) "hellochat2"
```
