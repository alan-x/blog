### 0x001 自动创建和移除`key`
到目前为止的例子中，我们从来没有去在推入元素之前创建一个空的列表，或者在列表为空的时候删除一个列表。当列表为空的时候删除列表或者在我们尝试使用类似`LPUSH`的命令插入元素的时候创建一个空的不存在的列表的`key`是`Redis`的责任，

这不仅仅特指`Lists`，这对由多种元素组成的`Redis`的所有数据类型都适用-- `Sets`，`Sorted Sets`和`Hashed`。

基本上，我们可以归纳为以下三个规则：

- 当我们添加一个元素到一个集合数据类型，如果目标`key`不存在，一个空的集合数据类型将在插入元素之前被创建。
- 当我们从一个集合数据类型移除一个元素的时候，如果值是空的，这个`key`将会被摧毁。
- 在一个空的`key`上执行一个类似`LLEN`命令的只读命令(他的作用是返回列表的长度)，或者一个移除元素的写命令总是产生相同的结果，总是产生一个相同的结果，就好像这个`key`拥有一个该命令想要找的集合类型的空的集合类型。
规则1的示例：
```
> del mylist
(integer) 1
> lpush mylist 1 2 3
(integer) 3
```
然而，如果`key`存在我们没办法使用对错误的数据类型做操作。
```
> set foo bar
OK
> lpush foo 1 2 3
(error) WRONGTYPE Operation against a key holding the wrong kind of value
> type foo
string
```
规则2的示例:
```
> lpush mylist 1 2 3
(integer) 3
> exists mylist
(integer) 1
> lpop mylist
"3"
> lpop mylist
"2"
> lpop mylist
"1"
> exists mylist
(integer) 0
```
`key`在元素被弹出以后就不再存在
规则3的示例：
```
> del mylist
(integer) 0
> llen mylist
(integer) 0
> lpop mylist
(nil)
```
### 0x002 `Redis Hashes`
`Redis hashes`看起来就是让一个元素由域值对组成`hash`:
```
> hmset user:1000 username antirez birthyear 1977 verified 1
OK
> hget user:1000 username
"antirez"
> hget user:1000 birthyear
"1977"
> hgetall user:1000
1) "username"
2) "antirez"
3) "birthyear"
4) "1977"
5) "verified"
6) "1"
```
使用`hashes`可以很简单的去表示对象，但是可以放置的域的数量没有客观的限制（相对于内存），所以你可以在你的应用中任意的使用hash。

`HMSET`可以设置`hash`的多个域，`HGET`可以获取一个域，`HMGET`命令和`HGET`很像，但是他返回值的数组：
```
> hmget user:1000 username birthyear no-such-field
1) "antirez"
2) "1977"
3) (nil)
```
还有其他命令也可以特殊的操作域，比如`HINCRBY`：
```
> hincrby user:1000 birthyear 10
(integer) 1987
> hincrby user:1000 birthyear 10
(integer) 1997
```
你可以在文档中找到完整的`hash`命令列表。
很小的`hashes`（很少的元素并且元素的值很小）被以一种特殊的方式编码，让它拥有很高效的内存利用是没有任何的意义的。

### 0x003 `Redis Sets`
Redis Sets
`Redis Sets`是一个字符串无序集合。`SADD`命令添加新的元素到`Redis Sets`。当然还有很多其他命令和集合交互，比如查看是否已经存在，获取多个集合的交集、差集、并集等。
```
> sadd myset 1 2 3
(integer) 3
> smembers myset
1. 3
2. 1
3. 2
```
这里我添加了三个元素到我的集合中并告诉`Redis`返回所有的元素。就像你看到的他们是无序的--`Redis`在每一次请求都自由返回所有元素，因为它并没有和用户约定关于元素的顺序。
`Redis`有测试成员资格的命令。比如，检查元素是否存在：

```
> sismember myset 3
(integer) 1
> sismember myset 30
(integer) 0
```
"3" is a member of the set, while "30" is not.
`3`是这个结合的成员，但`30`不是。

集合很擅长表达的对象之间的关系，比如，我们可以很简单的使用集合来实现标签。

去模拟这个问题的一个简单的方式是个体每一个我们想要打标签的对象一个集合。这个集合包含了关联到这个标签的`ID`

一个示例是标记新闻文章。如果`ID`为`1000`的文章被`ID`为`1`，`2`，`5`，`77`的标签标记，可以将这些标签的`ID`关联到新闻集合去：
```
> sadd news:1000:tags 1 2 5 77
(integer) 4
```
我们可以反过来做：所有的新闻都标记为给定标签：
```
> sadd tag:1:news 1000
(integer) 1
> sadd tag:2:news 1000
(integer) 1
> sadd tag:5:news 1000
(integer) 1
> sadd tag:77:news 1000
(integer) 1
```
获取给定对象的所有的标签是很简单的
```
> smembers news:1000:tags
1. 5
2. 1
3. 77
4. 2
```
**注意：**我们假设你有一个其他数据结构，比如`Redis hash`，将标签的`ID`映射到姓名。


还有很多无关紧要的操作使用正确的`Redis`命令是很容易事项的。比如我们想过要一个被`1,2,10,27`标签标记的列表。我们可以使用`SINTER`命令，它可以在多个集合之间求交集。我们可以使用：
```
> sinter tag:1:news tag:2:news tag:10:news tag:27:news
... results here ...
```
除了可以求交集，你也可以求合集、差集、提取随机元素等。

获取元素的命令是`SPOP`，我们可以很简单的模拟实际问题。比如，我们要实现一个网页版本的扑克游戏，你可能会使用`Sets`来表示你的牌组。想想我们使用一个字符前缀(C)lubs, (D)iamonds, (H)earts, (S)pades：
```
>  sadd deck C1 C2 C3 C4 C5 C6 C7 C8 C9 C10 CJ CQ CK
   D1 D2 D3 D4 D5 D6 D7 D8 D9 D10 DJ DQ DK H1 H2 H3
   H4 H5 H6 H7 H8 H9 H10 HJ HQ HK S1 S2 S3 S4 S5 S6
   S7 S8 S9 S10 SJ SQ SK
   (integer) 52
```
现在我们要提供每个玩家5张卡片。`SPOP`随机移除一个元素，并返回给客户端。所以在这种场景下这是一个非常完美的操作。
然而，如果我们直接这么操作，下一次游戏的时候，我们又要再初始化一次牌组，这不是一个好主意。所以在开始的时候，我们可以做一个备份，存储在另一个牌组，使用`game:1:deck`作为`key`的名字。

使用`SUNIONSTORE`是很成熟的方案，它可以求多个集合的并集，并且将它存储到其他集合。然后，因为现在只有一个集合，所以他的并集就是他自己，我可以这么复制我的牌组：
```
> sunionstore game:1:deck deck
(integer) 52
```
现在，我已经做好提供我的第一个玩家5张牌的准备了：
```
> spop game:1:deck
"C6"
> spop game:1:deck
"CQ"
> spop game:1:deck
"D1"
> spop game:1:deck
"CJ"
> spop game:1:deck
"SJ"
```
一对`J`，不是很好......

这是一个介绍可以获取集合内元素数量的命令的好机会。这在理论上一般叫做集合的基数，所以`Redis`的命令叫做`SCARD`。
```
> scard game:1:deck
(integer) 47
```
计算一下： 52 - 5 = 47
当你需要的只是随机获取元素二不需要将他们从集合中移除的时候，`SRANDMEMBER`命令比较适合这样的任务。返回重复的和不重复的。


### 0x004 `Redis Sorted sets`
`Sorted Set`是一种和`Set`混合`Hash`之后很像的数据结构。 和`Set`一样，他是由唯一的、不重复的字符串元素组成，所以在某种场景中，有序集合也可以看做集合。

尽管在集合内的元素是没有顺序的，但是集合内的每个元素都和一个福点值关联，叫做分数（这就是为什么这种类型和哈希很像，因为每个元素都映射到一个值）。

此外有序集合中的元素是按顺序获取的（他们在请求中是无序的，顺序是表现有序集合的数据结构的特性）。他们的循序遵循以下规则：

- 如果`A`和`B`两个元素有不通的分数，如果`A.score > B.score`那么`A > B`
- 如果`A`和`B`有相同的分数，如果`A`的词典顺序大于`B`，则`A > B`。`A`和`B`不可能相等，因为有序你和元素是唯一的。
让我们用一个简单的示例开始，添加一些选中的黑客名单作为有序集合的元素，他们的生日作为分数。
```
> zadd hackers 1940 "Alan Kay"
(integer) 1
> zadd hackers 1957 "Sophie Wilson"
(integer) 1
> zadd hackers 1953 "Richard Stallman"
(integer) 1
> zadd hackers 1949 "Anita Borg"
(integer) 1
> zadd hackers 1965 "Yukihiro Matsumoto"
(integer) 1
> zadd hackers 1914 "Hedy Lamarr"
(integer) 1
> zadd hackers 1916 "Claude Shannon"
(integer) 1
> zadd hackers 1969 "Linus Torvalds"
(integer) 1
> zadd hackers 1912 "Alan Turing"
(integer) 1
```
就像你看到的，`ZADD`和`SADD`很像，但是多了一个额外的分数参数（放在将要被填的的元素的前面）。`ZADD`也是参数可变的，所以你可以只有指定多个分数-值对，即使没有在上面的例子提现出来。

有序集合返回一个根据黑客出生年份排序的列表是微不足道的，因为他们本来就是排好序的。

实践笔记：游戏集合通过实现双重港口数据结构，包含一个可跳过的列表和一个哈希表。所以，每次我们添加一个元素到`Redis`是一个0(log(N))操作。这很好，但是我们请求有序列表，`Redis`不需要做任何工作，因为他们本来就是有序的。
```
> zrange hackers 0 -1
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
6) "Richard Stallman"
7) "Sophie Wilson"
8) "Yukihiro Matsumoto"
9) "Linus Torvalds"
```
**注意：**如果我想要反向排序呢，最新或者最旧？使用`ZREVRANGE`代替`ZRANGE`。
```
> zrevrange hackers 0 -1
1) "Linus Torvalds"
2) "Yukihiro Matsumoto"
3) "Sophie Wilson"
4) "Richard Stallman"
5) "Anita Borg"
6) "Alan Kay"
7) "Claude Shannon"
8) "Hedy Lamarr"
9) "Alan Turing"
```
返回分数也是可以的，使用`WITHSCORES`参数：
```
> zrange hackers 0 -1 withscores
1) "Alan Turing"
2) "1912"
3) "Hedy Lamarr"
4) "1914"
5) "Claude Shannon"
6) "1916"
7) "Alan Kay"
8) "1940"
9) "Anita Borg"
10) "1949"
11) "Richard Stallman"
12) "1953"
13) "Sophie Wilson"
14) "1957"
15) "Yukihiro Matsumoto"
16) "1965"
17) "Linus Torvalds"
18) "1969"
```
### 0x005 范围操作
Operating on ranges
有序集合的威力不知如此。他们可以范围操作。获取所有1950以前出生的人。我们使用`ZRANGEBYSCORE`命令来做到：
```
> zrangebyscore hackers -inf 1950
1) "Alan Turing"
2) "Hedy Lamarr"
3) "Claude Shannon"
4) "Alan Kay"
5) "Anita Borg"
```
我们让`Redis`返回所有分数在无穷小到1950之间的元素（两个临界值都包含在内）。

移除一个范围内的元素也是可以做大的。让我们从有序集合中移除出生在1940-1960的黑客吧：
```
> zremrangebyscore hackers 1940 1960
(integer) 4
```
`ZREMRANGEBYSCORE`可能不是一个好命令名字，但是他是非常有用的，它返回被移除元素的数量。
另一个对有序集合中的元素非常有用的操作是获取等级操作。它可以得到一个元素在有序集合中的位置。
```
> zrank hackers "Anita Borg"
(integer) 4
```
`ZREVRANK`命令当然也可以获得等级，但是顺序是倒过来的。

### 0x006 字典性分数
`Redis 2.8`版本的时候，引入了一个新的特性，那就是获取根据字典排序获取区间数据，假设有序集合中所有的元素都有同一个分数（元素的比较将使用C的`memcmp`函数，所以可以保证没有整理，并且每个`Redis`实例返回的都是一样的输出）。
操作字典性区间的命令主要有：`ZRANGEBYLEX`，`ZREVRANGEBYLEX`，`ZREMRANGEBYLEX` 和 `ZLEXCOUNT`。
The main commands to operate with lexicographical ranges are ZRANGEBYLEX, ZREVRANGEBYLEX, ZREMRANGEBYLEX and ZLEXCOUNT.
比如，再次添加著名的黑客到我的列表，但是所有代表时间的分数全部设置成0：
```
> zadd hackers 0 "Alan Kay" 0 "Sophie Wilson" 0 "Richard Stallman" 0
  "Anita Borg" 0 "Yukihiro Matsumoto" 0 "Hedy Lamarr" 0 "Claude Shannon"
  0 "Linus Torvalds" 0 "Alan Turing"
```
因为排序规则，他们已经完全按照字典排序了：
```
> zrange hackers 0 -1
1) "Alan Kay"
2) "Alan Turing"
3) "Anita Borg"
4) "Claude Shannon"
5) "Hedy Lamarr"
6) "Linus Torvalds"
7) "Richard Stallman"
8) "Sophie Wilson"
9) "Yukihiro Matsumoto"
```
使用`ZRANGEBYLEX`我们可以获取字典性排序的区间：
Using ZRANGEBYLEX we can ask for lexicographical ranges:
```
> zrangebylex hackers [B [P
1) "Claude Shannon"
2) "Hedy Lamarr"
3) "Linus Torvalds"
```
区间可以包含或者不包含（取决于第一个字符），字符串`infinite`和`minus infinite`是特俗的，代表`+`、`-`。获取更新信息请看文档。
这个特性非常重要，因为它允许我们使用有序集合像一般的索引。比如，你想要使用128位无符号整形参数作为元素索引，你要做的就是添加元素到一个有序集合，并设置相同的分数（比如0），同时在大字节序使用16比特前缀组成的128位数字，当字典性排序（原始比特顺序），就像数字排序一样，你可以在128位的空间里获取一个区间，然后抛弃前缀就可以获取元素的值了。
如果你想在知道更多这个特性在环境中更严谨的特性，查看`Redis`自动完成的案例。
### 0x007 更新分数：排行榜
在跳转到下一个话题之前，这是最后一个关于有序集合的笔记。
有序集合的分数可以在任何时候被更新，只要使用`ZADD`命令就可以更新已经存在在有序集合中的元素的分数了，这是一个O(log(N))的操作。所以，有序集合很适合大量的更新。
因为这个特点，所以一个普通的使用场景就是排行榜。最典型的应用就是`Facebook`游戏，当你结合按照高分获取用户和获取排行的操作，为了显示前N个用户，和用户在排行板中的等级（比如：你的分数在第4932名）。
Bitmaps
### 0x008 `Bitmap`
`Bitmap`不是一个真是的数据类型，而是定义在`String`类型上一个面向比特操作的集合。因为字符串是二进制安全的二进制大对象，最大的长度是512MB，他们很适合创建232不同的比特。
比特操作分为两种：常量时间单比特操作，比如设置一个bite为0或者1，或者获取他的值，和操作一组比特，比如统计给定区间比特的数量（比如，人口统计）。
关于`Bitmap`其中一个最大的优势是他们经常提供极大的空间保存信息。比如在一个系统中，不同的用户表示为自动增长的用户id，使用一个单独的比特就可以记住一些信息（比如，这个用户是否想收到消息），40亿的用户只需要512MB。
比特的设置和获取使用`SETBIT`、`GETBIT`命令：
```
> setbit key 10 1
(integer) 1
> getbit key 10
(integer) 1
> getbit key 11
(integer) 0
```
`SETBIT`的第一个参数是第几个比特，第二个参数是想要设置到这个比特的值，1或者0。这个命令在比特的索引超出去的时候自动扩大字符串的长度。
`GETBIT`返回指定索引的比特的值，超出范围的比特（如果访问的比特超出了字符串的长度，将自动存入这个`key`）被认为是0。
有三个命令可以操作比特组：

- `BITOP`是在不同字符串比特级别的操作，提供的操作有`AND`，`OR`，`XOR`和`NOT`
- `BITCOUNT`是人口统计，返回值为1的比特的数量
- `BITPOS`找到第一个指定的值的索引
`BITPOS`和`BITCOUNT`都可以去操作字符串区间内的比特，替代运行整个字符串。下面是一关于`BITCOUNT`个简单的操作：
```
> setbit key 0 1
(integer) 0
> setbit key 100 1
(integer) 0
> bitcount key
(integer) 2
```
`Bitmap`基本的应用场景是：

- 实时分析所有的种类
- 有效的存储空间但是高性能的关联到对象ID的布尔信息
比如，想想你想知道每天访问你网站的用户来了多少天。你从0开始统计，然后开放你的网站，使用`SETBIT`命令设置用户访问你网站的时间，比特的索引可以使用`Unix`时间，减去初始化的偏移，再除以3600*24
使用这个方式，每个用户你可以用最小的字符串包含了每天的访问信息。使用`BITCOUNT`命令可以简单的得到给定用户访问网站的天数，使用`BITPOS`命令或者简单的在客户端获取并分析`Bitmap`，就可以简单的统计访问时间。

`Bitmap`可以很简单的分割成很多个`key`，比如，分片数据集的理由是因为通常情况下会比一个大`key`好，吧一个`Bitmap`分割成不同的`key`，相对于设置所有的bit为到一个key要好，一个小策略是每个`key`只存储M比特，然后`key`的名字添加上`bit-number/M`，这样第N个比特的地址就是`bit-num MOD M`。


