### 0x001 添加一个值
> 命令格式：
>    - `SADD key member [member1 member2 ...]`
```
SADD goods apple banana
```
### 0x002 获取set的数量
> 命令格式：
>    - `SCARD key`
```
127.0.0.1:6379> SCARD goods
(integer) 2
```
### 0x003判断是否是set的member
> 命令格式：
>    - `SISMEMBER key member [member1 member2 ...]`
```
127.0.0.1:6379> SISMEMBER uid1 1
(integer) 1
```
### 0x004 获取set的所有member
> 命令格式：
>    - `SMEMBERS key`
```
127.0.0.1:6379> SMEMBERS goods
1) "banana"
2) "apple"
```
### 0x005 随机弹出指定数量的元素
> 命令格式：
>    - `SPOP key`
```
127.0.0.1:6379> SADD uid 1 2 3 4 5 6 7 
(integer) 7
127.0.0.1:6379> SPOP uid
"4"
127.0.0.1:6379> 
127.0.0.1:6379> SPOP uid
"7"
127.0.0.1:6379> SPOP uid 2
1) "2"
2) "6"
```
### 0x006 移除指定元素
> 命令格式：
>    - `SREM key member [member1 member2 ...]`
```
127.0.0.1:6379> DEL uid
(integer) 1
127.0.0.1:6379> SADD uid 1 2 3 4 5 6 7 
(integer) 7
127.0.0.1:6379> SREM uid 1 2 3 4 
(integer) 4
127.0.0.1:6379> SMEMBERS uid
1) "5"
2) "6"
3) "7"
```
### 0x007 获取指定集合的并集
> 命令格式：
>    - `SUNION key member [member1 member2 ...]`
```
127.0.0.1:6379> SADD uid1 1 2 3 4 5 6 
(integer) 6
127.0.0.1:6379> SADD uid2 3 4 5 6 7 8 
(integer) 6
127.0.0.1:6379> SUNION uid1 uid2
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
```
### 0x008 将指定集合的并集存储到新的set
> 命令格式：
>    - `SADD key member [member1 member2 ...]`
```
127.0.0.1:6379> SUNIONSTORE  uid3 uid1 uid2
(integer) 8
127.0.0.1:6379> SMEMBERS uid3
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
8) "8"
```
### 0x009 返回指定集合的交集
> 命令格式：
>    - `SINTER key1 key2`
```
127.0.0.1:6379> SINTER uid1 uid2
1) "3"
2) "4"
3) "5"
4) "6"
```
### 0x010 将指定集合的交集存到新的set
> 命令格式：
>    - `SINTERSTORE key1 [key2 key3 ...]`
```
127.0.0.1:6379> SINTERSTORE uid3 uid1 uid2
(integer) 4
127.0.0.1:6379> SMEMBERS uid3
1) "3"
2) "4"
3) "5"
4) "6"
```
### 0x011 返回指定集合的差集
> 命令格式：
>    - `SDIFF key1 [key2 key3 ...]`
```
127.0.0.1:6379> SDIFF uid1 uid2
1) "1"
2) "2"
```
### 0x012 将指定集合的差集存储到新的set
> 命令格式：
>    - `SDIFFSTORE key1 [key2 key3 ...]`
```
127.0.0.1:6379> SDIFFSTORE uid3 uid1 uid2
(integer) 2
127.0.0.1:6379> SMEMBERS uid3
1) "1"
2) "2"
```
