### 0x001 概述
redis的事务不具备原子性，在事务中，多条命令执行时，如果其中一条或者多条命令执行失败，并不会影响其他命令的执行，之前执行成功的命令也不会回滚，而之后尚未执行的命令将会继续执行。
### 0x002 执行一个事务
> 命令格式：
>         - `MULTI`：开始事务
>         - `command`：命令入队
>         - `EXEC`：执行
```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET name j
QUEUED
127.0.0.1:6379> SET age 11
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) OK
127.0.0.1:6379> GET name
"j"
127.0.0.1:6379> get age
"11"
```
### 0x003 放弃执行事务
> 命令格式：
>         - `MULTI`：开始事务
>         - `command`：命令入队
>         - `DISCARD`：取消事务
```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> SET num 100
QUEUED
127.0.0.1:6379> DISCARD
OK
127.0.0.1:6379> GET num
(nil)
```
### 0x004 非原子性
```
# 先放入一个String
127.0.0.1:6379> SET name lyxxxx
OK
# 此时从新设置`name`为`hash`将会报错，因为类型不匹配
127.0.0.1:6379> hset name first lin
(error) WRONGTYPE Operation against a key holding the wrong kind of value
# 此时开始事务
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set age 25
QUEUED
127.0.0.1:6379> set name first l
QUEUED
127.0.0.1:6379> set sex male
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (error) ERR syntax error 
3) OK
# 从上面的(2)可以看出`set name first l`执行失败了，但是(3)还是成功了
127.0.0.1:6379> get sex
"male"
```
### 0x005 监听`key`改变，取消事务
> 命令格式：
>         - `WATCH key [key key ...]`：监听`key`
>         - `UNWATCH`：取消监听

- 监听`name`：
    ```
    127.0.0.1:6379> watch name
    OK
    ```
- 把`name`改变
    ```
    127.0.0.1:6379> set name lyxxx2
    OK
    ```
- 执行事务,因为`name` 已经被改变了事务执行失败了，返回`nil`，并且值未被改变
    ```
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> set name lxxxx3
    QUEUED
    # name 已经被改变了，所以事务执行失败了
    127.0.0.1:6379> EXEC
    (nil)
    127.0.0.1:6379> get name
    "lyxxx2"
    ```
- 取消监听并再次执行事务，此时事务将会成功
    ```
     127.0.0.1:6379> UNWATCH 
    OK
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> set name lxxxx3
    QUEUED
    127.0.0.1:6379> EXEC
    1) OK
    127.0.0.1:6379> GET name
    "lxxxx2"
    ```
