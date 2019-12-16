### 0x001 概述
Hash的存储形式大概如下:

|--
|key1|field1|value1
|    |field2|value2
|key2|field1|value1
|    |field2|value2

###0x0002 设置或者获取单个field值
```
# 设置单个field的value
127.0.0.1:6379> HSET user address here
(integer) 1
# 获取单个field的value
127.0.0.1:6379> HGET user address
"here"
# 只有当field不存在的时候才设置，否则返回0
127.0.0.1:6379> HSETNX user address there
(integer) 0
127.0.0.1:6379> HGET user address
"here"
127.0.0.1:6379> HSETNX user money 0
(integer) 1
127.0.0.1:6379> HGET user money
"0"
```
### 0x003 批量添加新的值
> 命令格式 
>    - 批量设置：`HMSET key field value [field value field value...]`
>    - 批量获取：`HMGET  key field value [field value field value...]`
```
# 设置多个属性
127.0.0.1:6379> HMSET user name lyxxxx age 22
OK
# 如果key已经存在并且field也存在，则会覆盖旧的值
127.0.0.1:6379> HMSET user name lyxxxx2 age 222
OK
127.0.0.1:6379> HMGET user name
1) "lyxxxx2"
# 如果key已经存在但是field不存在，将会添加新的field值，其他field不变
127.0.0.1:6379> HMSET user sex male
OK
127.0.0.1:6379> HMGET user sex age
1) "male"
2) "22"
```
### 0x004  判断指定`field`是否存在
> 命令格式：
>    - 判断`field`是否存在：`HEXISTS key field`
```
127.0.0.1:6379> HMSET user name lyxxxx age 22 sex male
OK
127.0.0.1:6379> HEXISTS user name
(integer) 1
127.0.0.1:6379> HEXISTS user work
(integer) 0
```

### 0x005 获取`key`的内容
> 命令格式：
>    - 获取这个`key`的长度:`HLEN key`
>    - 获取这个`key`的所有`field`和`value`：`HGETALL key`
>    - 获取这个`key`的所有`field`:`HKEYS key`
>    - 获取这个`key`的所有`value`:`HVALS key`
```
127.0.0.1:6379> HLEN user
(integer) 3
127.0.0.1:6379> HGETALL user
1) "name"
2) "lyxxxx"
3) "age"
4) "22"
5) "sex"
6) "male"
127.0.0.1:6379> HKEYS user
1) "name"
2) "age"
3) "sex"
127.0.0.1:6379> HVALS user
1) "lyxxxx"
2) "22"
3) "male"
```
