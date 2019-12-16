### 0x001 添加元素
> 命令格式:`ZADD score member [score member ...]`
```
127.0.0.1:6379> ZADD star 100 game1 200 game2 300 game3  
(integer) 3
```
### 0x002 查看成员的数量
> 命令格式:`ZCARD key`
```
127.0.0.1:6379> ZCARD star
(integer) 3
```
### 0x003 查看某一区间分数的成员数量
> 命令格式：`ZCOUNT key min max` 
```
127.0.0.1:6379> ZCOUNT star 1 200
(integer) 2
```
### 0x004 查看某一区间索引的数量
> 命令格式：`ZLEXCOUNT key min max`
```
127.0.0.1:6379> ZLEXCOUNT star (game1 (game3
(integer) 1
127.0.0.1:6379> ZLEXCOUNT star [game1 [game3
(integer) 3
127.0.0.1:6379> ZLEXCOUNT star - [game3
(integer) 3
```
### 0x005 查看某一元素的索引
> 命令格式：`ZRANK key member`
```
127.0.0.1:6379> ZRANK star game1
(integer) 0
```
### 0x006 获取某个索引区间的值
> 命令格式：`ZRANK key start stop [WITHSCORES]`
```
127.0.0.1:6379> ZRANGE star 0 4 WITHSCORES
1) "game1"
2) "100"
3) "game2"
4) "200"
5) "game3"
6) "300"
```
### 0x007 获取某个分数区间的值
> 命令格式：`ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]`
```
127.0.0.1:6379> ZRANGEBYSCORE star 0 201 WITHSCORES
1) "game1"
2) "100"
3) "game2"
4) "200"
```
### 0x008 获取某个索引区间的值
> 命令格式：`ZRANGEBYLEX key min max [LIMIT offset count]`
```
127.0.0.1:6379> ZRANGEBYLEX star (game1 (game3
1) "game2"
```
### 0x009 获取某个元素的分数
> 命令格式：`ZSCORE key member`
```
127.0.0.1:6379> ZSCORE star game1
"100"
```
### 0x010 移除指定元素
> 命令格式：`ZREM key member [member member ...]`
```
127.0.0.1:6379> ZREM star game1 game2
(integer) 2
127.0.0.1:6379> ZRANGE star 0 100
1) "game3"
```
### 0x011 根据索引删除元素
> 命令格式：`ZREMRANGEBYLEX key min max`
```
127.0.0.1:6379> DEL star
(integer) 1
127.0.0.1:6379> ZADD star 100 game1 200 game2 300 game3
(integer) 3
127.0.0.1:6379> ZREMRANGEBYLEX star game1 game2
(error) ERR min or max not valid string range item
127.0.0.1:6379> ZREMRANGEBYLEX star [game1 [game2
(integer) 2
127.0.0.1:6379> ZRANGE star 0 100
1) "game3"
```
### 0x013 根据分数删除元素
> 命令格式：`ZREMRANGEBYSCORE key min max`
```
127.0.0.1:6379> DEL star
(integer) 1
127.0.0.1:6379> ZADD star 100 game1 200 game2 300 game3
(integer) 3
127.0.0.1:6379> ZREMRANGEBYSCORE star 0 201
(integer) 2
127.0.0.1:6379> ZRANGE star 0 100
1) "game3"

```
### 0x014 根据排行分数元素
> 命令格式：`ZREMRANGEBYSCORE key star stop`
```
127.0.0.1:6379> DEL star
(integer) 1
127.0.0.1:6379> ZADD star 100 game1 200 game2 300 game3
(integer) 3
127.0.0.1:6379> ZREMRANGEBYRANK star 1 2
(integer) 2
127.0.0.1:6379> ZRANGE star 0 100
1) "game1"
```
### 0x015 根据索引获取元素，按分数从高到底
> 命令格式：`ZREVRANGE key star stop [WITHSCORES]`
```
127.0.0.1:6379> ZREVRANGE star 0 100 WITHSCORES
1) "game3"
2) "300"
3) "game2"
4) "200"
5) "game1"
6) "100"
```
### 0x016 根据分数区间获取元素，按分数从高到底

> 命令格式：`ZREVRANGE key max min [WITHSCORES] [LIMIT offset count]`
```
127.0.0.1:6379> ZREVRANGEBYSCORE star 201 0 WITHSCORES
1) "game2"
2) "200"
3) "game1"
4) "100"
```
### 0x016 返回元素的排行
> 命令格式：`ZREVRANK key member`
```
127.0.0.1:6379> ZREVRANK star game2 
(integer) 1
```
### 0x017 增加元素的分数
> 命令格式：`ZLEXCOUNT key min max`
```
127.0.0.1:6379> ZINCRBY star 10 game2
"210"
127.0.0.1:6379> ZSCORE star game2
"210"
```
