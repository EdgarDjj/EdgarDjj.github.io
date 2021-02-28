# Redis学习


# Redis学习

## 介绍

Redis 就是一个使用 C 语言开发的数据库，Redis 的数据就是存在内存中，所以读写速度非常快，因此多被用于缓存。

另外，**Redis 除了做缓存之外，Redis 也经常用来做分布式锁，甚至是消息队列。**

**Redis 提供了多种数据类型来支持不同的业务场景。Redis 还支持事务 、持久化、Lua 脚本、多种集群方案。**

Redis常用的5种数据类型（String，Hash，List，Set，sorted set）

## 流程

1. 如果用户请求的数据在缓存中就直接返回。
2. 缓存中不存在的话就看数据库中是否存在。
3. 数据库中存在的话就更新缓存中的数据。
4. 数据库中不存在的话就返回空数据。

## 为什么用Redis作为缓存

直接来说就是速度性能的提升，且直接操作缓存能大大提高请求数量，毕竟常用的数据就那些，如果每次查询都要经过数据库的话，会极大的影响整个系统的性能，而使用缓存，将部分数据从数据库移动到缓存中去，这样一部分用户请求不会经过数据库，而先经过缓存，提高了系统整体的并发性。

## Redis常见数据结构以及应用

### 开始使用

```
// 启动redis服务
redis-server
// 使用redis客户端
redis-cli
在远程服务上执行命令
redis-cli -h host -p port -a password
```

#### string（SDS）

1. **介绍** ：string 数据结构是简单的 key-value 类型。虽然 Redis 是用 C 语言写的，但是 Redis 并没有使用 C 的字符串表示，而是自己构建了一种 **简单动态字符串**（simple dynamic string，**SDS**）。相比于 C 的原生字符串，Redis 的 SDS 不光可以保存文本数据还可以保存二进制数据，并且获取字符串长度复杂度为 O(1)（C 字符串为 O(N)）,除此之外,Redis 的 SDS API 是安全的，不会造成缓冲区溢出。
2. **常用命令:** `set,get,strlen,exists,dect,incr,setex` 等等。
3. **应用场景** ：一般常用在需要计数的场景，比如用户的访问次数、热点文章的点赞转发数量等等。

```bash
127.0.0.1:6379> set key djj
OK
127.0.0.1:6379> get key
"djj"
127.0.0.1:6379> strlen key # 获取长度
(integer) 3
127.0.0.1:6379> exists key # 判断某个 key 是否存在
(integer) 1
127.0.0.1:6379> del key # 删除某个key对应的值
(integer) 1
127.0.0.1:6379> get key # 删除后返回 nil
(nil)
```

##### 批量设置

```bash
127.0.0.1:6379> mset k1 213 k2 432 # 批量设置
OK
127.0.0.1:6379> mget k1 k2 # 获取多个键值对
1) "213"
2) "432"
```

##### 计数器（字符串内容为整数的时候可以使用）

```bash
127.0.0.1:6379> set number1 1
OK
127.0.0.1:6379> incr number1 # 将存储的数字+1
(integer) 2
127.0.0.1:6379> get number1
"2"
127.0.0.1:6379> decr number1 # 将存储的数字-1
(integer) 1
127.0.0.1:6379> get number1
"1"
```

##### 过期

```bash
127.0.0.1:6379> expire key 60 # 数据在60秒后过期
(integer) 0
127.0.0.1:6379> setex key 60 432 # 数据在60秒后过期 值为432
OK
127.0.0.1:6379> get key
"432"
127.0.0.1:6379> ttl key # 查看数据还有多久过期
(integer) 54
127.0.0.1:6379> ttl key # 过期了
(integer) -2
127.0.0.1:6379> get key # 数据无效了
(nil)
```

#### list

1. **介绍** ：**list** 即是 **链表**。链表是一种非常常见的数据结构，特点是易于数据元素的插入和删除并且且可以灵活调整链表长度，但是链表的随机访问困难。许多高级编程语言都内置了链表的实现比如 Java 中的 **LinkedList**，但是 C 语言并没有实现链表，所以 Redis 实现了自己的链表数据结构。Redis 的 list 的实现为一个 **双向链表**，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。
2. **常用命令:** `rpush,lpop,lpush,rpop,lrange、llen` 等。
3. **应用场景:** 发布与订阅或者说消息队列、慢查询。

##### 通过 rpush/lpop 实现队列

```bash
127.0.0.1:6379> rpush mylist 123
(integer) 1
127.0.0.1:6379> rpush mylist 32 43
(integer) 3
127.0.0.1:6379> lpop mylist # 从最头部取出
"123"
127.0.0.1:6379> lrange mylist 0 1
1) "32"
2) "43"
127.0.0.1:6379> lrange mylist 0 -1 # -1表示倒数第一个元素
1) "32"
2) "43"
```

##### 通过 rpush/rpop 实现栈

```bash
127.0.0.1:6379> rpush mylist 123
(integer) 3
127.0.0.1:6379> lrange mylist 0 2
1) "32"
2) "43"
3) "123"
127.0.0.1:6379> rpop mylist # 弹出顶部元素
"123"
127.0.0.1:6379> lrange mylist 0 1
1) "32"
2) "43"
```

总结： lpush/lpop 为对链表的左侧进行操作，rpush/rpop 为对链表的右侧进行操作。

![list结构](Redis学习/list数据结构.png)

##### 查看列表范围 lrange/llen

```bash
127.0.0.1:6379> lrange mylist 0 1 # 可以通过lrange来进行分页查询
1) "32"
2) "43"
127.0.0.1:6379> llen mylist # 查看链表长度
(integer) 2
```

#### hash

1. **介绍** ：hash 类似于 JDK1.8 前的 HashMap，内部实现也差不多(数组 + 链表)。不过，Redis 的 hash 做了更多优化。另外，hash 是一个 string 类型的 field 和 value 的映射表，**特别适合用于存储对象**，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。
2. **常用命令：** `hset,hmset,hexists,hget,hgetall,hkeys,hvals` 等。
3. **应用场景:** 系统中对象数据的存储。

```bash
127.0.0.1:6379> hset user name "djj" age 20
(integer) 2
127.0.0.1:6379> hexists user name # 查看hash表中的字段是否存在
(integer) 1
127.0.0.1:6379> hexists user age
(integer) 1
127.0.0.1:6379> hget user name # 获取hash表中的字段
"djj"
127.0.0.1:6379> hget user age
"20"
127.0.0.1:6379> hset user age 21 # 修改hash表中的字段值
(integer) 0
127.0.0.1:6379> hget user age
"21"
```

### set

1. **介绍 ：** set 类似于 Java 中的 `HashSet` 。Redis 中的 set 类型是一种无序集合，集合中的元素没有先后顺序。当你需要存储一个列表数据，又不希望出现重复数据时，set 是一个很好的选择，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作。比如：你可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis 可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程。
2. **常用命令：** `sadd,spop,smembers,sismember,scard,sinterstore,sunion` 等。
3. **应用场景:** 需要存放的数据不能重复以及需要获取多个数据源交集和并集等场景

```bash
127.0.0.1:6379> sadd mySet 123 432 32
(integer) 3
127.0.0.1:6379> sadd mySet 123 # 插入失败，set集合中不允许有重复的元素
(integer) 0
127.0.0.1:6379> smembers mySet # 查询set集合中的所有元素
1) "32"
2) "123"
3) "432"
127.0.0.1:6379> scard mySet # 查看set集合的长度
(integer) 3
127.0.0.1:6379> sismember mySet 123 # 查看set集合中的某个元素是否存在 只接受单个元素
(integer) 1

127.0.0.1:6379> sadd myset1 1 2 3 4
(integer) 4
127.0.0.1:6379> sadd myset2 2 3 4 5
(integer) 4
127.0.0.1:6379> sinterstore myset myset1 myset2 # 将myset1 和 myset2 中的交集存放到myset中
(integer) 3
127.0.0.1:6379> smembers myset
1) "2"
2) "3"
3) "4"
127.0.0.1:6379> sunionstore myset3 myset1 myset2 # 获取myset1 和 myset2 的并集并存放到myset3中
(integer) 5
127.0.0.1:6379> smembers myset3 
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
```

### sorted sort

1. **介绍：** 和 set 相比，sorted set 增加了一个权重参数 score，使得集合中的元素能够按 score 进行有序排列，还可以通过 score 的范围来获取元素的列表。有点像是 Java 中 HashMap 和 TreeSet 的结合体。
2. **常用命令：** `zadd,zcard,zscore,zrange,zrevrange,zrem` 等。
3. **应用场景：** 需要对数据根据某个权重进行排序的场景。比如在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息。

```bash
127.0.0.1:6379> zadd myzet 2.0 v1 # 添加一个权重为2.0的值v1
(integer) 1
127.0.0.1:6379> zadd myzet 3.0 v2
(integer) 1
127.0.0.1:6379> zadd myzet 1.0 v3
(integer) 1
127.0.0.1:6379> zcard myzet # 显示sorted set中的数量
(integer) 3
127.0.0.1:6379> zscore myzet v2 # 查询key中某个值的权值
"3"
127.0.0.1:6379> zrange myzet 0 3 # 顺序输出sorted set中的值 （按照权值从小到大）
1) "v3"
2) "v1"
3) "v2"
```

### 总结

Xcommand，X没有的时候为string类型，X为l的时候为list类型，X为s的时候为set类型，X为h的时候为hash类型，X为z的时候为sorted set类型。

Xcard：一般用于查询集合中的数量

Xrange：输出某一数据结构某一范围中的值

Xset：设置或更改

Xget：获取




