
# 基本概念

redis 是一个基于 C 语言开发的、开源、基于内存的键值型 **NoSQL 数据库**（BSD 许可）

> Remote Dictionary Server，远程词典服务器

## redis 为什么快？ #字节抖音24

[✅Redis为什么这么快？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/kc7dw3)

Redis 内部做了非常多的性能优化，比较重要的有下面 3 点：

- 基于**内存**，内存的访问速度是磁盘的上千倍；
- 基于 Reactor 模式设计开发了一套高效的**事件处理模型**，
	- 主要是 单线程事件循环 和 IO 多路复用 #todo扩展 
- 内置了多种优化过后的**数据类型/结构**实现

![|600](assets/Pasted%20image%2020240314002538.png)


## 为什么 Redis 设计成单线程？

[✅Redis为什么被设计成是单线程的？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/og6nf4)

[✅为什么Redis设计成单线程也能这么快？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/lrhzxqbur0eywnfu)

Redis中只有网络请求模块和数据操作模块是单线程的。而其他的如持久化存储模块、集群支撑模块等是多线程的。

Redis 并没有在网络请求模块和数据操作模块中使用多线程模型，主要是基于以下四个原因：
- 1、Redis 操作基于内存，绝大多数操作的性能瓶颈不在 CPU
- 2、使用单线程模型，可维护性更高，开发，调试和维护的成本更低
- 3、单线程模型，避免了线程间切换带来的性能开销
- 4、在单线程中使用多路复用 I/O 技术也能提升 Redis 的 I/O 利用率


## 为什么 Redis 6.0 引入了多线程？

[✅为什么Redis 6.0引入了多线程？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/zfpgxa93bmn9png9)

Redis 6.0中的多线程，也只是针对处理网络请求过程采用了多线程，而数据的读写命令，仍然是单线程处理的。

不是说多路复用技术已不是说多路复用技术已经大大的提升了 IO 利用率了么，为啥还需要多线程？主要是因为我们对 Redis 有着更高的要求。

根据测算，Redis 将所有数据放在内存中，内存的响应时长大约为 100 纳秒，对于小数据包，Redis 服务器可以处理 80,000 到 100,000 QPS，这么高的对于 80% 的公司来说，单线程的 Redis 已经足够使用了。

但随着越来越复杂的业务场景，有些公司动不动就上亿的交易量，因此==需要更大的 QPS==。为了提升 QPS，很多公司的做法是部署 Redis 集群，并且尽可能提升 Redis 机器数。但是这种做法的资源消耗是巨大的。

而经过分析，限制 Redis 的性能的主要瓶颈出现在网络 IO 的处理上，虽然之前采用了多路复用技术。但是我们前面也提到过，==多路复用的 IO 模型本质上仍然是同步阻塞型 IO 模型==。

在多路复用的IO模型中，在处理网络请求时，调用 select （其他函数同理）的过程是阻塞的，也就是说这个过程会阻塞线程，==如果并发量很高，此处可能会成为瓶颈==。

Redis 6.0 只有在网络请求的接收和解析，以及请求后的数据通过网络返回给时，使用了多线程。而数据读写操作还是由单线程来完成的，所以，这样就不会出现并发问题了。






## 为什么用 redis 做缓存？(2) #rep

> 说一下 Redis 和 Memcached 的区别和共同点

- *丰富的数据类型*：（支持更复杂的应用场景）不仅支持简单的 k/v 类型数据，还提供 list，set，zset，hash 等数据结构的存储
- *持久化*
	- 灾难恢复机制
	- 内存用完后，可以将不用的数据放到磁盘上
- *原生集群*
- *单线程*：单线程的多路 IO 复用模型（Redis 6.0 针对网络数据的读写引入了多线程）
- *支持发布订阅模型、Lua 脚本、事务*等功能（Memcached 不支持）
- *支持更多语言客户端*
- *过期策略*：同时使用了惰性删除与定期删除

> Memcached 
> - 只支持最简单的 k/v 数据类型。
> - 把数据全部存在内存之中，内存用完后直接报异常
> - 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据
> - Memcached 是多线程，非阻塞 IO 复用的网络模型；
> - Memcached 过期数据的删除策略只用了惰性删除


## redis 适用场景(2+) #rep

> - 为什么要用 Redis/缓存？
> - 用本地缓存可不可以 #todo
> - 什么时候使用 Redis，什么情况不适用 #todo

1）高性能

> 用户第一次访问数据库中的某些数据的话，这个过程是比较慢

高频数据并且不会经常改变的数据放入缓存，用户下一次访问速度快（还可以缓存预热）

2）高并发

> 一般像 MySQL 这类的数据库的 **QPS** 大概都在 1w 左右（4 核 8g） ，但是使用 Redis 缓存之后很容易达到 10w+，甚至最高能达到 30w+（就单机 Redis 的情况，Redis 集群的话会更高）。
> 
> QPS（Query Per Second）：服务器每秒可以执行的查询次数；

缓存能够承受的请求数量是==远远大于==直接访问数据库，所以我们可以考虑把数据库中的部分数据转移到缓存中去，进而提高了系统整体的并发


## Redis Module

#todo 


# 数据结构

## redis 常用数据结构 #面过

- String
	- 细分 string、int、float
	- 可存 json
- Hash
	- 特点：对 field 做 CRUD，json 字符串 不行
- List
	- 特点：有序、可重复、链表
	- 可模拟栈、队列、阻塞队列
- Set
	- 特点：无序、不重复、查找快、支持交并差集
- SortedSet
	- 特点：可排序的 Set

## bitmap 了解吗？有什么使用场景？

> bitmap原理

Bitmap 存储的是**连续的二进制数字（0 和 1）**，只需要一个 bit 位来表示某个元素对应的值或者状态，Bitmap 本身会极大的节省储存空间

可以将 Bitmap 看作是一个存储二进制数字（0 和 1）的数组，数组中每个元素的下标叫做 **offset**（偏移量）

Redis 中是利用 **string** 类型数据结构实现 BitMap，因此最大上限是 512MB，转换为 bit 则是 **2^32**个 bit 位

## Hyperloglog

Hyperloglog (HLL) 是从 Loglog 算法派生的概率算法，用于确定**非常大的集合的基数**（元素个数），且**不需要存储元素本身的全部**，所以 HLL 不能像集合那样，返回输入的各个元素。

- Redis 中的 HLL 是基于 **string** 结构实现的
- 单个 HLL 的内存永远**小于 16kb**，内存占用低的令人发指！
- 作为代价，**其测量结果是概率性的**，有小于 0.81%的误差。不过对于 UV 统计来说，这完全可以忽略。

## SortedSet 底层数据结构？时间复杂度？ #字节24 #面过

[✅Redis中的Zset是怎么实现的？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/uzqztzuicddlk95c)

> 当 ZSet 的元素数量比较少时，Redis 会采用 ZipList（ListPack）来存储 ZSet 的数据。当 ZSet 的元素数量增多时，Redis 会自动将 ZipList（ListPack）转换为 SkipList，以保持元素的有序性和支持范围查询操作。

**zset 是基于 dict（字典）和 zskiplist（跳表）实现的**

- SkipList 用来实现==有序==集合，跳表的**插入、删除和查找**操作的时间复杂度都是 $O(log n)$
- dict 用来实现==元素到 score 的映射关系==，哈希表的**插入、删除和查找**操作的时间复杂度都是 $O(1)$

```java
typedef struct zset 
{ 
    dict *dict; 
    zskiplist *zsl;
} zset;
```

### 跳表和二叉树区别？ #面过 

> 想问的是为什么要用跳表？一样的时间复杂度二叉树也可以

[Redis为什么用跳表实现有序集合 | JavaGuide](https://javaguide.cn/database/redis/redis-skiplist.html#%E5%92%8C%E5%85%B6%E4%BD%99%E4%B8%89%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%9A%84%E6%AF%94%E8%BE%83)

平衡树每一次插入或者删除操作都需要保证整颗树左右节点的绝对平衡，只要不平衡就要通过旋转操作来保持平衡，这个过程是比较耗时的。

跳表诞生的初衷就是为了克服平衡树的一些缺点

> 跳表的发明者在论文[《Skip lists: a probabilistic alternative to balanced trees》](https://15721.courses.cs.cmu.edu/spring2018/papers/08-oltpindexes1/pugh-skiplists-cacm1990.pdf)中有详细提到
> 
> Skip lists are a data structure that can be used in place of balanced trees. Skip lists use probabilistic balancing rather than strictly enforced balancing and as a result the algorithms for insertion and deletion in skip lists are much simpler and significantly faster than equivalent algorithms for balanced trees.
> 
> 跳表是一种可以用来代替平衡树的数据结构。跳表使用概率平衡而不是严格强制的平衡，因此，跳表中的插入和删除算法比平衡树的等效算法简单得多，速度也快得多。

红黑树（Red Black Tree）也是一种自平衡二叉查找树，它的查询性能略微逊色于 AVL 树，但插入和删除效率更高

相比较于红黑树来说，跳表的实现也更简单一些。并且，按照区间来查找数据这个操作，红黑树的效率没有跳表高。


## zset 插入过程 #字节24 #todo


## 哈希冲突怎么解决的？ #面过

链地址法



# 缓存

缓存就是数据交换的缓冲区(称作 cache)，是存储数据的临时地方，一般读写性能较高


## Redis 缓存主要针对哪些内容？ #美团24

热点数据


## 缓存读写策略

### Cache Aside Pattern

> 旁路缓存模式

服务端需要同时维系 db 和 cache，并且是以 db 的结果为准

- 使用比较多，
- 适合场景：读请求比较多

具体步骤：

- 写操作：
	- 先更新 db
	- 然后删除 cache 
- 读操作:
	- 从 cache 中读数据，
		- 读取到就直接返回
		- 读取不到的话，就从 db 中获取
	- 写操作

问题：

- 删除 or 更新 缓存?
	- ❌更新缓存：每次更新数据库都更新缓存，**无效写操作较多**
	- ✔️删除缓存：更新数据库时让缓存失效，查询时再更新缓存

- 如何保证缓存与数据库的操作的 同时成功或失败?
	- 单体系统：将缓存和数据库操作放在一个事务里
	- 分布式系统：TCC 等分布式事务方案

- 先操作 缓存 or 数据库?
	- 都可能发生“线程安全”问题，导致缓存和数据库**不一致**的问题
	- 先删缓存，后更新数据库：发生的**概率高**
	- 先更新数据库，后删缓存：发生的概率很低，因为**缓存的速度高于数据库很多**

缺陷：

- **首次**请求数据一定不在 cache 的问题
	- 解决办法：可以将热点数据可以提前放入 cache 中（缓存预热）

- 写操作比较频繁 会导致 cache 被频繁被删除，影响**缓存命中率**
	- 数据库和缓存*强一致场景*：
		- 更新 db 的时候同样更新 cache，
		- **锁/分布式锁** 保证更新 cache 的时不存在线程安全问题
	- 可以*短暂地允许不一致*的场景：
		- 更新 db 的时候同样更新 cache，
		- 给缓存加一个**比较短的过期时间**，即使数据不一致影响也比较小

### Read/Write Through Pattern

> 读写穿透模式

服务端把 cache 视为主要数据存储，从中读取数据并将数据写入其中。cache 服务负责将此数据写入 db，从而减轻了应用程序的职责

> 这种缓存读写策略非常少见。抛去性能方面的影响，大概率是因为 **Redis 并没有提供 cache 将数据写入 db 的功能**。

具体操作：

- 写（Write Through）：
	- 先查 cache，
		- 不存在，直接更新 db
		- 存在，则先更新 cache，然后 cache 服务自己更新 db（**同步更新** cache 和 db）
- 读(Read Through)：
	- 从 cache 中读取数据，
		- 读取到就直接返回
		- 读取不到，cache 从 db 加载，写入到 cache 后返回响应

![|500](assets/Pasted%20image%2020240315110241.png)

缺点：首次请求数据一定不在 cache 的问题

### Write Behind Pattern

> 异步缓存写入

cache 服务来负责 cache 和 db 的读写

具体操作：

- Write Behind：只更新缓存，不直接更新 db，而是改为**异步**批量的方式来更新 db

## 缓存穿透(2)

> 布隆过滤器怎么实现(1) #todo

客户端请求的数据在**缓存和数据库中都不存在**，这样缓存永远不生效，请求都会打到数据库里

方案一：缓存空值（无效 key）

> 缓存空值以后，第二次就能从缓存里获取空值，直接返回不存在

- 优点：简单
- 缺点：
	- 额外的内存消耗（大量恶意攻击，缓存大量无效 key）
		- 解决：设置较短的 TTL
	- 短期的数据不一致
		- 解决：真正插入的时候覆盖

方案二：布隆过滤器 #todo 

- 优点：内存占用少，没有多余 key
- 缺点：
	- 实现复杂
	- 存在误判可能

## 缓存击穿(2)  #rep 

> 类似问题：
> 
> - 给数据库加互斥锁，如果有些就是请求因为一直抢不到锁，出现饿死情况怎么解决(1)

一个**被高并发访问**并且缓存重建业务较复杂（**重建时间较长**）的 key 突然失效了（过期），无数的请求会瞬间给数据库带来巨大的压力

> 也叫热点 Key 问题
> 
> 举个例子：秒杀过程中，缓存中的某个秒杀商品的数据突然过期，导致瞬时大量对该商品的请求直接落到数据库上

解决方案：

- 预防
	- 设置合理的 TTL
		- 热点 key 永不过期 或者 TTL 比较长
		- 秒杀场景下的数据在秒杀结束之前不过期
	- 热点数据缓存预热
- 主动解决
	- 互斥锁、互斥锁+逻辑过期
		- 互斥锁保证**只有一个请求会落到数据库上**

> 常见的缓存预热方式有两种：
> 
> 1. 使用定时任务，比如 xxl-job，来定时触发缓存预热的逻辑，将数据库中的热点数据查询出来并存入缓存中
> 2. 使用消息队列，比如 Kafka，来异步地进行缓存预热，将数据库中的热点数据的主键或者 ID 发送到消息队列中，然后由缓存服务消费消息队列中的数据，根据主键或者 ID 查询数据库并更新缓存

1）*互斥锁*：

缓存未命中，先获取互斥锁，查询 db 重建缓存数据后，再释放互斥锁。抢锁失败的线程循环查询缓存，直到从缓存中成功获取数据。

- 优点：
	- **保证一致性**
- 缺点：
	- 线程要等待，性能受影响
	- 有死锁风险

2）*互斥锁+逻辑过期*

> key 是永久的，由后端添加一个 expire 字段（LocalDateTime 实现）表示过期时间；
> 
> LocalDateTime提供了
> - `now.plusSeconds(TTL)` 设置逻辑过期时间
> - `expireTime.isAfter(now)` 判断是否过期

发现逻辑过期后，
- 获取锁成功的线程，另开一个线程异步重建缓存（不会等待了），然后**返回过期数据**，
- 获取锁失败的线程直接**返回过期数据**。
- 异步线程执行完后**释放锁**

优点：
- 线程无需等待，**性能较好**

缺点
- 不保证一致性
- 有额外的内存开销


## 缓存雪崩(2) #rep

在同一时段大量的缓存 key 同时失效 或者 Redis 服务宕机 导致大量请求到达数据库

针对 Redis 服务不可用的情况：

- 采用 Redis 集群，避免单机出现问题整个缓存服务都没办法使用
- 限流，避免同时处理大量的请求
- 降级（缓存全崩完了）
- 多级缓存
	- 例如：jvm 本地缓存+Redis 缓存（浏览器缓存、nginx 缓存）

针对缓存失效的情况：

- 设置不同的 TTL
	- 例如：给不同 key 的 TTL 添加随机值
- 热点 key，缓存预热


# 分布式锁

满足 分布式系统 或 集群模式 下 多进程可见并且互斥的锁

特性：

- 多进程可见
- 互斥
- 高可用
- 高性能
- 安全性：服务挂了锁没释放、死锁

## 如何实现分布式锁(2+) #rep

> 底层实现（基于什么命令）(2)
> 
> redis 并发锁内部实现，分段锁好处 #todo

1）获取锁

- 利用 `SETNX` 的互斥性，`SET lock v EX time NX`

对应 RedisTemplate 的方法 `Boolean setIfAbsent(K key, V value, long timeout, TimeUnit unit)`

2）释放锁

- 手动释放，`DEL lock`
- 超时释放，获取锁时添加超时时间（服务宕机）


## 可重入锁怎么实现

#todo完善 

通过 Redis 的哈希结构实现

- `field`：线程标识
- `value`：重入次数

## 用 setnx 实现可重入锁？ #字节24 #面过

[✅如何用setnx实现一个可重入锁？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/ponw7kdrqasbrgoz#Ubunj)



## 超时续约

> 看门狗机制是怎么实现的 #滴滴 


## Redisson

> Redisson的底层机制了解吗? 与redis实现分布式锁有什么区别？解决了哪些问题？ #携程


## lua

> Lua 脚本的作用？ #携程
> 
> lua 为什么能保证原子性？ #腾讯



# 内存管理

## Redis 是如何判断数据是否过期的呢？

## redis 的过期策略有哪些(1)

## 过期数据的删除策略(1) #rep 

常用的过期数据的删除策略

- *惰性删除*：取出 key 的时候才进行过期检查
	- 对 CPU 更友好
	- 但是可能会造成太多过期 key 没有被删除
- *定期删除*：每隔一段时间抽取一批 key， 删除过期 key
	- 对内存更友好
	- Redis 底层会通过限制删除操作执行的时长和频率来减少删除操作对 CPU 时间的影响

Redis 采用的是 【定期删除 + 惰性/懒汉式删除】 

仅仅通过给 key 设置 TTL，还是可能漏掉很多过期 key，导致大量过期 key 堆积在内存里，然后就 Out of memory 了

> 怎么解决这个问题呢？答案就是：Redis 内存淘汰机制。


## 内存淘汰策略 #面过

[说说Redis的内存淘汰策略 (yuque.com)](https://www.yuque.com/tulingzhouyu/db22bv/fn1agyxs0t15wlvu)

[✅Redis的内存淘汰策略是怎么样的？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/xw99lcraocebx1mk)

Redis 的内存淘汰策略用于在内存满了之后，决定哪些 key 要被删除。Redis 支持多种内存淘汰策略，可以通过配置文件中的 maxmemory-policy 参数来指定。

Redis 的内存淘汰策略只有在运行内存达到了==配置的最大内存阈值==时才会触发

Redis 提供了 6 种内存淘汰策略：

- 从已设置过期时间的数据集（`server.db[i].expires`）中
	- **volatile-lru（least recently used）**：挑选==最近最少使用==的数据淘汰。
	- **volatile-ttl**：挑选==将要过期==的数据淘汰。
	- **volatile-random**：任意选择数据淘汰。
	- **volatile-lfu（least frequently used）**：挑选==最不经常使用（使用频率最低）==的数据淘汰。
- 从数据集（`server.db[i].dict`）中
	- **allkeys-lru（least recently used）**：移除==最近最少使用==的数据淘汰。
	- **allkeys-random**：从数据集（`server.db[i].dict`）中任意选择数据淘汰。
	- **allkeys-lfu（least frequently used）**：移除==最不经常==使用的数据淘汰。
- **no-eviction**（默认内存淘汰策略）：禁止驱逐数据，当内存不足以容纳新写入数据时，新写入操作会报错。


## 怎么选择内存淘汰策略？

以下是腾讯针对 Redis 的淘汰策略设置给出的建议：

- 当 Redis 作为==缓存==使用的时候，推荐使用 allkeys-lru 淘汰策略。该策略会将最近最少使用的 Key 淘汰。默认情况下，使用频率最低则后期命中的概率也最低，所以将其淘汰。

- 当 Redis 作为==半缓存半持久化==使用时，可以使用 volatile-lru。但因为 ==Redis 本身不建议保存持久化数据，所以只作为备选方案==。

> 个人理解：持久化的场景下,新加的 key 可能来不及持久化就被删了，所以从过期数据里删

阿里云 Redis 默认是 volatile-lru （[https://www.alibabacloud.com/help/zh/redis/user-guide/how-does-apsaradb-for-redis-evict-data-by-default](https://www.alibabacloud.com/help/zh/redis/user-guide/how-does-apsaradb-for-redis-evict-data-by-default) ）

腾讯云默认是 noeviction，即不删除键。在内存占满后会出现 OOM 问题，所以建议==创建好实例后修改淘汰策略==，减少 OOM 问题的出现。（[https://cloud.tencent.com/document/product/239/90960](https://cloud.tencent.com/document/product/239/90960) ）

# 持久化

使用缓存的时候，经常需要对内存中的数据进行持久化，大部分原因是为了之后

- **重用数据**（比如重启机器、机器故障之后恢复数据），
- 或者是为了做**数据同步**（比如 Redis 集群的主从节点通过 RDB 文件同步数据）

## 持久化机制 #面过

Redis 支持三种持久化方式

1）**RDB**：内存快照，将 Redis 的内存中的数据==定期==保存到磁盘上

- 优点：快照文件小、恢复速度快
- 缺点：定期更新可能会丢数据
- 适用场景：适合做备份和灾难恢复。

2）**AOF**：将 Redis 的所有写操作追加到 AOF 文件（Append Only File）的末尾

- 优点：可以实现更高的数据可靠性、支持更细粒度的数据恢复
- 缺点：
	- 文件大占用空间更多
	- 每次写操作都需要写磁盘导致负载较高
- 适用场景：数据存档和数据备份

3）RDB 和 AOF 的**混合持久化** (Redis 4.0 新增)

AOF 重写时会把 Redis 的持久化数据，以 RDB 的格式写入到 AOF 文件的开头，之后的数据再以 AOF 的格式化追加的文件的末尾。


| **特性**    | **RDB**                 | **AOF**            |
| --------- | ----------------------- | ------------------ |
| 数据可靠（完整）性 | 可能会丢失最后一次快照之后的数据        | 保证最后一次写操作之前的数据不会丢失 |
| 存储空间占用    | 快照文件较小，占用空间较少（压缩的二进制数据） | AOF文件较大，占用空间较多     |
| 恢复速度      | 较快                      | 较慢                 |
| 系统资源占用    | 高，大量 CPU 和内存消耗          | 低，主要是磁盘 IO 资源      |

RDB和AOF的选择：

- Redis 保存的数据**丢失一些也没什么影响**的话，可以选择使用 RDB。
- **不建议单独使用 AOF**，因为时不时地创建一个 RDB 快照可以进行数据库备份、更快的重启以及解决 AOF 引擎错误。
- 如果保存的数据要求安全性比较高的话，建议同时开启 RDB 和 AOF 持久化或者开启 RDB 和 AOF 混合持久化。



## AOF 的三种写回策略

- `appendfsync always`：每执行一次写命令，立即记录到AOF文件
- `appendfsync everysec`（默认）：写命令执行完先放入 AOF 缓冲区，然后表示每隔 1 秒将缓冲区数据写到 AOF 文件
- `appendfsync no`：写命令执行完先放入 AOF 缓冲区，由**操作系统**决定何时将缓冲区内容写回磁盘


# 集群

## 主从复制

如何保证 Redis 服务高可用? 最简单的一种办法就是==基于 主从复制 搭建一个 Redis 集群==，master（主节点）要负责处理写请求，slave（从节点）主要负责处理读请求

### 什么是主从复制？ #rep 

==将一台 Redis 主节点的数据复制到其他的 Redis 从节点中==，尽**最大可能**保证 Redis 主节点和从节点的数据是**一致**的。

- 主从复制这种方案不仅保障了 Redis 服务的高可用，还实现了**读写分离**，提高了系统的并发量，尤其是读并发量。
- Redis Sentinel 以及 Redis Cluster 都**依赖**于主从复制

在指定从节点执行 replicaof 命令开启主从关系

```properties
replicaof <masterip> <masterport>

# 7002和7003执行
replicaof 192.168.111.154 7001
```

> 复制品

### 主从复制下从节点会主动删除过期数据吗?

#todo

### 主从复制过程(1)

- *全量同步*：master 将完整内存数据生成 RDB，发送 RDB 到 slave。后续命令则记录在 `repl_baklog`，逐个发送给 slave
- *增量同步*：slave 提交自己的 offset 到 master，master 获取 repl baklog 中从 offset 之后的命令给 slave

每一个 master 都有唯一的 replid，slave 则会**继承** master 节点的 replid（第一次 master 会把 replid 给 slave）

流程

- slave 节点执行 `replicaof`，请求数据同步，把 replid 和 offset 传给 master
- master 节点判断 replid，
	- replid 不一致，说明是第一次同步数据，**全量同步**
		- master 将完整内存数据生成 RDB（bgsave），发送 RDB 到 slave
		- slave 清空本地数据，加载 master 的 RDB
		- master 将 RDB 期间的命令记录在 `repl baklog`，并持续将 log 中的命令发送给 slave
		- slave 执行接收到的命令，保持与 master 之间的同步
	- replid 一致，master 查看 slave 传来的 offset，master 的同步进度必须比 slave 更快并且两者的同步进度必须在规定范围，
		- 未超过 `repl backlog` 的大小（能在 `repl_baklog` 中能找到 offset 时），**增量同步**
		- 否则，**全量同步**操作

### 为什么主从全量复制使用 RDB 而不是 AOF?

- RDB 文件更节省带宽，传输速度快
- 恢复大数据集的时候，RDB 速度更快
- AOF 需要选择合适的刷盘策略，如果刷盘策略选择不当的话，会影响 Redis 的正常运行。并且，根据所使用的刷盘策略，AOF 的速度可能会慢于 RDB。

## 哨兵

### 什么是哨兵，有什么用？

Sentinel 只是 Redis 的一种**运行模式**，

- 不提供读写服务，
- 默认运行在 26379 端口上，
- 依赖于 Redis 工作

Redis Sentinel 实现 Redis 集群高可用，只是在主从复制实现集群的**基础**上，多了一个 Sentinel 角色来帮助我们

- _监控_：Sentinel 会不断检查 redis 节点状态
- _自动故障恢复_（故障转移）：如果 master 故障，Sentinel 会将一个 slave 提升为 master。当故障实例恢复后也以新的 master 为主
- _通知_：Sentinel 充当 Redis 客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给 Redis 的客户端

### 如何检测节点是否下线

Sentinel 基于**心跳机制**监测服务状态，每隔 1 秒向集群的每个实例发送 **ping 命令**：

- _主观下线_：sentinel 发现某实例未在规定时间（down-after-milliseconds）响应，则认为该实例主观下线
- _客观下线_：若**超过指定数量**（quorum）的 sentinel 都认为该实例主观下线，则该实例客观下线（quorum 最好超过总数量一半）

如果被认定为主观下线的是 slave 的话， sentinel不会做什么事情

如果是 master 被认定为主观下线，所有 sentinel 节点要以每秒一次的频率确认 master 的确下线了，当法定数量的 sentine 节点认定 master 已经下线， master 才被判定为 客观下线

### 哨兵选主过程(1)

sentinel 如何在 slave 中选择一个新的 master？

- *筛选出在线的 slave*：判断 slave 节点与 master 节点断开时间长短，如果超过指定值(`down-after-milliseconds*10`)则会排除该 slave 节点
- *slave-prority 优先级*：slave 节点的 slave-priority 越小优先级越高（特殊值 0 表示永不参与选举）
- *复制进度 offset*：如果 slave-prority 一样，则判断 slave 节点的 offset 值，越大说明数据越新，优先级越高
- *runid*：最后判断 slave 节点的运行 id 大小，越小优先级越高

当选中了其中一个 slave 为新的 master 后，**故障转移的步骤**如下：

- sentinel 给**备选 slave** 节点发送 `slaveof no one` 命令，让该节点成为 master
- sentinel 给所有**其它 slave** 发送 `slaveof <masterip> <masterport>` 命令，让这些 slave 成为新 master 的从节点，开始从新的 master 上同步数据。
- 最后，sentinel 将**故障节点**标记为 slave（强行修改配置文件），当故障节点恢复后会自动成为新的 master 的 slave 节点

### sentinel 可以防止脑裂吗？

#todo

## 分片(1)

#todo



## redis 集群，批量获取 key 会有什么问题

> 如果保证了相同业务场景的key都写入了一个主节点，这是再使用mget，会不会有什么问题
> 
> 解决集群中批量获取的问题，应该怎么做？


# 未知

## CAP 理论

## redis 怎么优化内存占用的？

> redis数据结构是怎么优化内存空间使用的

## 网络模型


