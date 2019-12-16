### 0x001 概述
redis是存储键值对的数据库，存储形式可以表达为如下：

|---|---|
|-|-|
|key|value|
|key|value|
|key|value|
|key|value|
|key|value|

- `key`:
    `key`是二进制安全的，这意味着可以使用任意的二进制序列作为`key`，从类似`foo`的字符串到一个`JPEG`文件内容，甚至空字符串也可以。
- `value`
    `value`有多种的数据结构，并不是单纯的字符串
    - Strings： 字符串
    - Lists：列表
    - Sets：集合
    - Hashes：哈希
    - Sorted sets：有序哈希
    - Bitmaps and HyperLogLogs：不讨论
### 0x002 添加`key`

> 语法格式：SET key value

```
127.0.0.1:6379> SET name helloworld
OK
# 如果`key`已经存在了将会覆盖原先的`key`
127.0.0.1:6379> SET name helloworld2
OK
127.0.0.1:6379> GET name helloworld2
"helloworld2"
```
### 0x003 判断`key`是否存在
> 语法格式：EXITS key
```
127.0.0.1:6379> EXISTS name
(integer) 1

# 如果`key`不存在，将会返回0
127.0.0.1:6379> EXISTS name2
(integer) 0
```
### 0x004 获取`key`的`value`
> 语法格式：GET key
```
127.0.0.1:6379> GET name
"helloworld"

# 如果`key`不存在，将会返回`nil`
127.0.0.1:6379> GET name2
(nil)

```
### 0x005 删除`key`的`value`
> 命令格式：DEL KEY_NAME
```
# 该命令返回删除的条数，如果`key`不存在，则返回0
127.0.0.1:6379> DEL name
(integer) 1
127.0.0.1:6379> DEL name2
(integer) 0
```
### 0x006 修改`key`的值，是`key`的名字，不是`value`
> 命令格式： RENAME name newName
```
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> RENAME name newName
OK
127.0.0.1:6379> Get name
(nil)
127.0.0.1:6379> GET newName
"helloworld"
# 如果`key`不存在，将会报错
127.0.0.1:6379> RENAME name2 name
(error) ERR no such key
# 如果`newKey`已经存在，则将覆盖
127.0.0.1:6379> set name2 helloworld2
OK
127.0.0.1:6379> RENAME name name2
OK
127.0.0.1:6379> GET name2
"helloworld"
# 只有在`newKey`不存在的时候，才重命名，防止被覆盖要用以下命令
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> SET name2 helloworld2
OK
127.0.0.1:6379> RENAMENX name name2
(integer) 0
127.0.0.1:6379> GET name
"helloworld"
127.0.0.1:6379> GET name2
"helloworld2"
```
### 0x007 获取`key`存储`value`的数据类型
```
127.0.0.1:6379> set name helloworld
OK
127.0.0.1:6379> TYPE name
string
# 如果`key`不存在，将会返回`none`
127.0.0.1:6379> TYPE name2
none
```

### 0x008 设置过期时间
> 语法格式： EXPIRE key seconde|timestamp , PEXPIRE secode|timestamp
```
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> GET name
"helloworld"
# 设置5s过期
127.0.0.1:6379> EXPIRE name 5
(integer) 1
# 5s内
127.0.0.1:6379> GET name
"helloworld"
# 5s后
127.0.0.1:6379> GET name
(nil)

```

### 0X009 获取剩余时间
> 命令格式：TTL key | PTTL key
```
# 返回单位：秒
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> EXPIRE name 5
(integer) 1
127.0.0.1:6379> TTL name
(integer) 3
127.0.0.1:6379> TTL name
(integer) 2
127.0.0.1:6379> TTL name
(integer) 1
127.0.0.1:6379> TTL name
(integer) -2
127.0.0.1:6379> GET name
(nil)
# 返回单位：毫秒
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> EXPIRE name 5
(integer) 1
127.0.0.1:6379> PTTL name
(integer) 1106
127.0.0.1:6379> PTTL name
(integer) 445
127.0.0.1:6379> PTTL name
(integer) -2
127.0.0.1:6379> PTTL name
(integer) -2
127.0.0.1:6379> PTTL name
(integer) -2
127.0.0.1:6379> GET name
(nil)

```
### 0x010 移除过期时间
> 命令格式： PERSIST key
```
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> EXPIRE name 10
(integer) 1
127.0.0.1:6379>  TTL name
(integer) 7
127.0.0.1:6379>  PTTL name
(integer) 4323
127.0.0.1:6379> PERSIST name
(integer) 1
127.0.0.1:6379> GET name
"helloworld"
```
0x011 注意
redis的命令不区分大小写，所以`get name`和`GET name`一样。
