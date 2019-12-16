### 0x001 设置值并获取
> 命令格式：SET key | GET key
```
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> GET name
"helloworld"
127.0.0.1:6379> GET name2
(nil)
```
### 0x002 获取指定区间的值
> 命令格式： GRANGE name start end
```
127.0.0.1:6379> GETRANGE name 2 6
"llowo"

# 如果`key`不存在，则返回空
127.0.0.1:6379> GETRANGE name2 1 10
""
```


### 0X003 设置新值，并返回旧的值
> 命令格式： GETSET key value
```
127.0.0.1:6379> GETSET name helloworld2
"helloworld"
127.0.0.1:6379> get name
"helloworld2"

# 如果值不存在，则返回nil，但是设置的新值是成功的
127.0.0.1:6379> GETSET name2 helloworld2
(nil)
127.0.0.1:6379> GET name2
"helloworld2"
```

### 0x004 获取多个KEY
> 命令格式： MGET key [key,key,key]
```
127.0.0.1:6379> set name helloworld
OK
127.0.0.1:6379> set name2 helloworld2
OK
127.0.0.1:6379> MGET name name2 name3
1) "helloworld"
2) "helloworld2"
3) (nil)
```

### 0x005 不存在的时候才设置key的值
> 命令格式： SETNX key
```
127.0.0.1:6379> SET name helloworld
OK
127.0.0.1:6379> SETNX name helloworld2
(integer) 0
127.0.0.1:6379> GET name
"helloworld"

```


### 0x006 获取字符串长度
> 命令格式： STRLEN key
```
127.0.0.1:6379> STRLEN name
(integer) 10
```


### 0x007 设置多个键值对
> 命令格式： MSET key value key value

```
127.0.0.1:6379> MSET name helloworld name2 helloworld2
OK
127.0.0.1:6379> MGET name name2
1) "helloworld"
2) "helloworld2"
```
### 0x008 value是数字的情况下，增加value的值
> 命令格式：INCR key | INCRBY key
```
127.0.0.1:6379> set num 1
OK
127.0.0.1:6379> INCR num
(integer) 2
127.0.0.1:6379> GET num
"2"
# 如果value不是数字，将会报错
127.0.0.1:6379> INCR name
(error) ERR value is not an integer or out of range
# 如果key不存在，则自动创建
127.0.0.1:6379> INCR num2
(integer) 1
# 增加指定量
127.0.0.1:6379> INCRBY num 5
(integer) 7
127.0.0.1:6379> GET num
"7"
```

### 0x009 value是数字的情况下，减少value的值
> 命令格式：DECR key | DECRBY key
```
127.0.0.1:6379> set num 1
OK
127.0.0.1:6379> DECR num
(integer) 0
127.0.0.1:6379> DECR num
(integer) -1
127.0.0.1:6379> DECRBY num 4
(integer) -5
127.0.0.1:6379> DECRBY num2
(error) ERR wrong number of arguments for 'decrby' command
127.0.0.1:6379> DECR num2
(integer) 0
127.0.0.1:6379> DECRBY num3 2
(integer) -2
```
### 0x010 追加value
> 命令格式：APPEND key | DECRBY key
```
127.0.0.1:6379> set name helloworld
OK
127.0.0.1:6379> APPEND name 2
(integer) 11
127.0.0.1:6379> GET name
"helloworld2"
# 如果key不存在将报错
127.0.0.1:6379> APPEND name2
(error) ERR wrong number of arguments for 'append' command

```

