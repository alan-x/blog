### 0x001 PUSH和POP
> 命令格式：
>    - `LPUSH key value [value2 value3...]`:推入值
>    - `LPOP key value`:推出值
>    - `LLEN key`:获取list长度
>    - `LPUSHX key value [value2 value3...]`:只有存在这个`key`，才推入值：
>    - `RPUSH key value [value2 value3...]`:反向推入值 
>    - `RPUSHX key value [value2 value3...]`:只有存在这个`key`，才反向推入值
```
# 推入一个值
127.0.0.1:6379> LPUSH order 1
(integer) 1
# 推入另一个值
127.0.0.1:6379> LPUSH order 2
(integer) 2
# 弹出一个值
127.0.0.1:6379> LPOP order 
"2"
# 弹出另一个值
127.0.0.1:6379> LPOP order 
"1"
# list为空的时候，弹出的为nil
127.0.0.1:6379> LPOP order 
(nil)
# 推入一个值
127.0.0.1:6379> LPUSH order 2
(integer) 1
# 推入多个值
127.0.0.1:6379> LPUSH order 2 3 4
(integer) 4
# 获取list长度
127.0.0.1:6379> LLEN order
(integer) 4
# 获取索引为0-10的值
127.0.0.1:6379> LRANGE order 0 10
1) "10000"
2) "4"
3) "3"
4) "2"
5) "2"
# 推入不存在的list，返回0
127.0.0.1:6379> LPUShX goods 1
(integer) 0
# 推如存在的list
127.0.0.1:6379> LPUShX order -1
(integer) 6
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "10000"
3) "4"
4) "3"
5) "2"
6) "2"
# 反序推入值
127.0.0.1:6379> RPUSH goods 1 2 3 4
(integer) 4
127.0.0.1:6379> LRANGE goods 0 10
1) "1"
2) "2"
3) "3"
4) "4"
# 只有存在才推入值
127.0.0.1:6379> RPUSHX goods2 1 2
(integer) 0
```
### 0x002 获取指定范围的值
> 命令格式：
>    - `LRANGE key start value`:获取某个区间索引的值
```
127.0.0.1:6379> LRANGE order 0 10
1) "4"
2) "3"
3) "2"
4) "2"
127.0.0.1:6379> LRANGE order 0 2
1) "4"
2) "3"
3) "2"

```
### 0x003 根据索引获取值
> 命令格式：
>    - `LINDEX key index`
```
127.0.0.1:6379> LINDEX order 0
"10000"
```
### 0x004 根据索引修改值
> 命令格式：
>    - `LINDEX key index value`
```
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "10000"
3) "3"
127.0.0.1:6379> LSET order 1 1
OK
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "1"
3) "3"
```
### 0x005 根据索引插入值
> 命令格式：
>    - `LINDEX key index BEFORE|AFTER key value`
```
127.0.0.1:6379> LINSERT order BEFORE 10000 -10
(integer) 7
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "-10"
3) "10000"
4) "4"
5) "3"
6) "2"
7) "2"
127.0.0.1:6379> LINSERT order BEFORE s -10
(integer) -1
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "-10"
3) "10000"
4) "4"
5) "3"
6) "2"
7) "2"
```
### 0x006 移除
> 命令格式：
>    - `LREM key count value`：根据指定值删除指定值及其周围指定数量的值
>        - count > 0 : 正序删除|count|数量的value值
>        - count < 0 : 倒序删除|count|数量的value值
>        - count = 0 : 删除所有指定的值
```
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "-10"
3) "10000"
4) "4"
5) "3"
6) "2"
7) "2"
127.0.0.1:6379> LREM order 2 -10
(integer) 1
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "10000"
3) "4"
4) "3"
5) "2"
6) "2"
127.0.0.1:6379> LREM order -2 4
(integer) 1
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "10000"
3) "3"
4) "2"
5) "2"
127.0.0.1:6379> LREM order 0 2
(integer) 2
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "10000"
3) "3"
```
### 0x007 修剪
> 命令格式：
>    - `LTRIM key start stop`:将区间内的值删除
```
127.0.0.1:6379> LRANGE order 0 10
1) "-1"
2) "1"
3) "3"
127.0.0.1:6379> LTRIM order 1 2
OK
127.0.0.1:6379> LRANGE order 0 10
1) "1"
2) "3"
```
