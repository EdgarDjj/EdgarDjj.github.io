# Redis学习（前置）


# Redis（前置）

## 缓存

### 基本思想

缓存就是用空间换时间，提高系统的性能以及减少请求相应时间。

像CPU与内存之间的速度不匹配的问题，用缓存进行平衡这种速度不匹配。在操作系统中，在页表的方案基础之上引入快表来加速虚拟地址到物理地址的映射转换。可以将快表理解为一种特殊的高速缓存存储器（CaChe）。

而在业务系统中，为了避免用户在请求数据时获取速度过慢，在数据库上添加了这一层缓存来弥补。

### 使用缓存带来的问题

> 引入本地缓存来做一些简单业务场景的话，实际带来的代价几乎可以忽略，下面主要是针对分布式缓存来说的。

1、系统复杂性增加：引入缓存后，需要维护缓存和数据库的数据一致性、维护热点缓存等等。

2、系统开发成本增加：引入缓存意味着系统需要一个单独的缓存服务，这是需要花费相应的成本的，并且这个成本还是很贵的，毕竟耗费的是宝贵的内存。但是，如果你只是简单的使用一下本地缓存存储一下简单的数据，并且数据量不大的话，那么就不需要单独去弄一个缓存服务。

### 本地缓存解决方案

#### 一、JDK自带的HashMap和ConcurrentHashMap

`ConcurrentHashMap`可以看作是线程安全版本的`HashMap`，两者都是以存放 Key/Value 键值对。

#### 二、Ehache、Guava Cache、Spring Cache 本地缓存框架

Ehache 比另外两者更加重量，但 Ehcache支持可以嵌入到 hibernate 和 mybatis 中作为多级缓存，并可以将缓存的数据持久化到本地磁盘中，同时提供集群方案（鸡肋）。

`Guava Cache` 和 `Spring Cache` 两者的话比较像。

`Guava` 相比于 `Spring Cache` 的话使用的更多一点，它提供了 API 非常方便我们使用，同时也提供了设置缓存有效时间等功能。它的内部实现也比较干净，很多地方都和 `ConcurrentHashMap` 的思想有异曲同工之妙。

使用 `Spring Cache` 的注解实现缓存的话，代码会看着很干净和优雅，但是很容易出现问题比如缓存穿透、内存溢出。

**三： 后起之秀 Caffeine。**

相比于 `Guava` 来说 `Caffeine` 在各个方面比如性能要更加优秀，一般建议使用其来替代 `Guava` 。并且， `Guava` 和 `Caffeine` 的使用方式很像！

本地缓存固然好，但是缺陷也很明显，比如多个相同服务之间的本地缓存的数据无法共享。

### 为什么要有分布式缓存

*我们可以把分布式缓存（Distributed Cache） 看作是一种内存数据库的服务，它的最终作用就是提供缓存数据的服务。*

本地的缓存的优势是低依赖，比较轻量并且通常相比于使用分布式缓存要更加简单。

再来分析一下本地缓存的局限性：

1. **本地缓存对分布式架构支持不友好**，比如同一个相同的服务部署在多台机器上的时候，各个服务之间的缓存是无法共享的，因为本地缓存只在当前机器上有。
2. **本地缓存容量受服务部署所在的机器限制明显。** 如果当前系统服务所耗费的内存多，那么本地缓存可用的容量就很少。

使用分布式缓存之后，缓存部署在一台单独的服务器上，即使同一个相同的服务部署在再多机器上，也是使用的同一份缓存。 并且，单独的分布式缓存服务的性能、容量和提供的功能都要更加强大。

使用分布式缓存的缺点呢，也很显而易见，那就是你需要为分布式缓存引入额外的服务比如 Redis 或 Memcached，你需要单独保证 Redis 或 Memcached 服务的高可用。

### 缓存读写模式/更新策略

#### Cache Aside Pattern （旁路缓存模式）

1、写：更新 DB，然后直接删除 cache

2、读：从 cache 中读取数据，读取到就直接返回，读取不到的话，就从 DB 中取数据返回，然后再把数据放到 cache 中。

Cache Aside Pattern 中服务端需要同时维系 DB 和 cache，并且是以 DB 的结果为准。另外，Cache Aside Pattern 有首次请求数据一定不在 cache 的问题，对于热点数据可以提前放入缓存中。

**适合读请求比较多的场景**

#### Read/Write Through Pattern （读写穿透）

Read/Write Through 套路是：服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中。cache 服务负责将此数据读取和写入 DB，从而减轻了应用程序的职责。

1. 写（Write Through）：先查 cache，cache 中不存在，直接更新 DB。 cache 中存在，则先更新 cache，然后 cache 服务自己更新 DB（**同步更新 cache 和 DB**）。
2. 读(Read Through)： 从 cache 中读取数据，读取到就直接返回 。读取不到的话，先从 DB 加载，写入到 cache 后返回响应。

Read-Through Pattern 实际只是在 Cache-Aside Pattern 之上进行了封装。在 Cache-Aside Pattern 下，发生读请求的时候，如果 cache 中不存在对应的数据，是由客户端自己负责把数据写入 cache，而 Read Through Pattern 则是 cache 服务自己来写入缓存的，这对客户端是透明的。

和 Cache Aside Pattern 一样， Read-Through Pattern 也有首次请求数据一定不再 cache 的问题，对于热点数据可以提前放入缓存中。

#### Write Behind Pattern （异步缓存写入）

Write Behind Pattern 和 Read/Write Through Pattern 很相似，两者都是由 cache 服务来负责 cache 和 DB 的读写。

但是，两个又有很大的不同：**Read/Write Through 是同步更新 cache 和 DB，而 Write Behind Caching 则是只更新缓存，不直接更新 DB，而是改为异步批量的方式来更新 DB。**

**Write Behind Pattern 下 DB 的写性能非常高，尤其适合一些数据经常变化的业务场景比如说一篇文章的点赞数量、阅读数量。** 往常一篇文章被点赞 500 次的话，需要重复修改 500 次 DB，但是在 Write Behind Pattern 下可能只需要修改一次 DB 就可以了。

但是，这种模式同样也给 DB 和 Cache 一致性带来了新的考验，很多时候如果数据还没异步更新到 DB 的话，Cache 服务宕机就 gg 了。


