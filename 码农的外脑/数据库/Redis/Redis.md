
# -------------------- 入门篇

> 学习教程：
> 黑马程序员Redis入门到实战教程，深度透析redis底层原理+redis分布式锁+企业解决方案+黑马点评实战项目

# ---------- Redis 介绍与安装

## NoSQL

什么是 NoSQL？

- NoSQL 仅仅是一个概念，泛指**非关系型**的数据库
- 区别于关系数据库，它们不保证关系数据的 ACID 特性
- 常见的 NoSQL 数据库有：`Redis`、`MemCache`、`MongoDB` 等

> NoSQL 最常见的解释是 "non-relational"， 很多人也说它是"**_Not Only SQL_**"

---

NoSQL 与 SQL 的差异：

| <br /> | **SQL** | **NoSQL** |
| --- | --- | --- |
| **数据结构** | 结构化 | 非结构化 |
| **数据关联** | 关联的 | 无关联的 |
| **查询方式** | SQL查询 | 非SQL |
| **事务特性** | ACID | BASE |
| **存储方式** | 磁盘 | 内存 |
| **扩展性** | 垂直 | 水平 |
| **使用场景** | 1）数据结构固定<br />2）相关业务对数据安全性、一致性要求较高 | 1）数据结构不固定<br />2）对一致性、安全性要求不高<br />3）对性能要求 |

## 介绍

> Redis 诞生于 2009 年全称是 Remote Dictionary Server，远程词典服务器，是一个**基于内存的键值型** NoSQL 数据库。

特征：

- *键值*（key-value）型：value 支持多种不同数据结构，功能丰富
- *单线程*：每个命令具备原子性，中途不会执行其他命令
	- 命令处理始终是单线程，自 6.x 起改为多线程接收网络请求
- *低延迟，速度快*：基于内存、IO多路复用、良好的编码
- *持久化*
- *集群*：支持主从、分片集群
- *支持多语言客户端*

## 安装与启动

> [Docker安装最新Redis6（redis-6.2.7）（参考官方文档）_docker安装redis6_大白有点菜的博客-CSDN博客](https://blog.csdn.net/u014282578/article/details/128223953?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169331447916777224475827%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169331447916777224475827&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-128223953-null-null.142^v93^insert_down28v1&utm_term=Docker%E5%AE%89%E8%A3%85Redis6&spm=1018.2226.3001.4187)

Redis的启动方式有很多种，例如：前台启动、后台启动、开机自启

### 前台启动（不推荐）

> 会阻塞整个会话窗口，窗口关闭或者按下 `CTRL + C` 则 Redis 停止。不推荐使用。

安装完成后，在任意目录输入 `redis-server` 命令即可启动 Redis 

![](https://image-bed-vz.oss-cn-hangzhou.aliyuncs.com/Redis/image-20220524191137983.png#id=qkhPZ&originHeight=466&originWidth=1024&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

### 后台启动

> 必须修改Redis配置文件

-  修改 `redis.conf` 文件中的一些配置 

> 0.0.0.0 已经不是一个真正意义上的 IP 地址了。它表示的是这样一个集合：所有不清楚的主机和目的网络

```properties
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes 
# 密码，设置后访问Redis必须输入密码
requirepass 1325
```

-  **Redis其他常用配置** 
```shell
# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.log"
```

-  **启动Redis** 
```shell
# 进入redis安装目录 
cd /usr/local/src/redis-6.2.6
# 启动
redis-server redis.conf
```

-  **停止Redis服务** 
```shell
# 通过kill命令直接杀死进程
kill -9 redis进程id
```

### 开机自启
> **我们也可以通过配置来实现开机自启**

-  **首先，新建一个系统服务文件** 
```shell
vi /etc/systemd/system/redis.service
```

-  **将以下命令粘贴进去** 
```nginx
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/src/redis-6.2.6/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

-  然后重载系统服务 
```shell
systemctl daemon-reload
```

-  现在，我们可以用下面这组命令来操作redis了 
```shell
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```

-  执行下面的命令，可以让redis开机自启 
```shell
systemctl enable redis
```

## 层级 key

Redis 的 key 允许有多个单词形成层级结构，多个单词之间用":"隔开

> 例如：`项目名:业务名:类型:id`
> 
> 格式不固定，可根据需求删除或添加词条
> 
> 例如：项目名称叫 `heima`，有 `user` 和 `product` 两种不同类型的数据
> 
> - `heima:user:1`
> - `heima:product:1`

## 序列化存储

如果 Value 是一个 Java 对象，则可以将对象序列化为 JSON 字符串后存储

| KEY               | VALUE                                     |
| ----------------- | ----------------------------------------- |
| `heima:user:1`    | `{"id":1, "name": "Jack", "age": 21}`     |
| `heima:product:1` | `{"id":1, "name": "小米11", "price": 4999}` |

```bash
set heima:user:1 '{"id":1, "name": "Jack", "age": 21}'
set heima:user:2 '{"id":2, "name": "Jack2", "age": 22}'
```

![400](assets/Pasted%20image%2020240315113641.png)

# ---------- Redis 数据结构

Redis 是一个 key-value 的数据库，key 一般是 String 类型，**value 的类型多种多样**

![600](assets/image-20220524205926164.png)

## 通用命令

> 不用记，忘了就查
> 
> - **Redis的中文文档**：[http://www.redis.cn/commands.html](http://www.redis.cn/commands.html)
> - 菜鸟教程官网：[https://www.runoob.com/redis/redis-keys.html](https://www.runoob.com/redis/redis-keys.html)
> - `redis-cli help` 查看，`help [command]` 查看一个命令的具体用法！

通用指令不分数据类型的，都可以使用

- `set key value`
- `keys [pattern]` 模糊搜索
	- 性能较差，生产环境(尤其是主节点)不建议使用
- `del key...`
- `exists key` 判断 key 是否存在
- `expire key` 设置过期时间，过期自动删除（默认永久）
- `ttl key` 查询剩余存活时间（未设置过期时间则为 -1）

## String

根据字符串的格式分为 3 类：

- `string`：普通字符串
- `int`：整数类型，可以做自增、自减操作
- `float`：浮点类型，可以做自增、自减操作

> 底层都是“字节数组”形式存储，只是编码方式不同。字符串类型的最大空间不能超过 512m

| **KEY** | **VALUE**   |
| ------- | ----------- |
| msg     | hello world |
| num     | 10          |
| score   | 92.5        |

String 的 常见命令

| **命令**          | **描述**               |     |
| --------------- | -------------------- | --- |
| `SET k v`       | 添加或者修改               |     |
| `SETEX k v`     | 添加，并指定有效期            |     |
| `SETNX k v`     | 添加，前提是 key 不存在，否则不执行 |     |
| `GET k`         | 根据 key 获取  value     |     |
| `MSET`          | 批量添加                 |     |
| `MGET`          | 批量获取                 |     |
| `INCR k`        | 让一个整型的 key 自增1       |     |
| `INCRBY k 步长`   | 让一个整型的 key 自增并指定步长   |     |
| `INCRBYFLOAT k` | 让一个浮点类型的数字自增并指定步长    |     |

## Hash

> value 一个无序字典，类似于 Java 中的 `HashMap` 结构。

可将对象中的每个字段独立存储，可以**针对单个字段做 CRUD**（如果保存为 json 字符串就不可以）

![500](assets/image-20220525001227167.png)

Hash 的常见命令：

| **命令**                 | **描述**                                    |
| ---------------------- | ----------------------------------------- |
| `HSET key field value` | 添加或修改 key 的 field 的 value                 |
| `HSETNX`               | 添加，前提是 field 不存在，否则不执行                    |
| `HGET key field`       | 获取 key 的 field 的 value                    |
| ~~`HMSET`~~            | ~~hmset 和 hset 效果相同 ，4.0 之后 hmset 可以弃用了~~ |
| `HMGET`                | 批量获取                                      |
| `HGETALL k`            | 获取 key 中的所有的 field 和 value                |
| `HKEYS k`              | 获取 key 中的所有 field                         |
| `HVALS k`              | 获取 key 中的所有 value                         |
| `HINCRBY k f 步长`       | 让 key 的 field 自增并指定步长                     |

## List

> 类似 Java 中的 `LinkedList` 双向链表。既可以支持正向检索和也可以支持反向检索。

常用来存储有序数据，特征也与 LinkedList 类似：

- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

> 例如：朋友圈点赞列表，评论列表等

List 的常见命令：

| **命令**               | **描述**                      |
| -------------------- | --------------------------- |
| `LPUSH k v ...`      | 向列表左侧插入一个或多个元素              |
| `LPOP k`             | 移除并返回列表左侧的第一个元素，没有元素则返回 nil |
| `BLPOP k TIMEOUT`    | 与 LPOP 类似，没有元素时可指定等待时间      |
| `LRANGE k start end` | 返回一段角标范围内的所有元素              |
| `RPUSH k v ...`      | 右侧                          |
| `RPOP key`           | 右侧                          |
| `BRPOP`              | 右侧                          |

![700](assets/new.gif)

思考问题

- List 模拟**栈**?
	- 先进后出，入口和出口在同一边
-  List 模拟**队列**?
	- 先进先出，入口和出口在不同边
-  List 模拟**阻塞队列**?
	- 入口和出口在不同边
	- 出队时采用 `BLPOP` 或 `BRPOP`

## Set

> 与 Java 中的 `HashSet` 类似，可以看做是一个 value 为 null 的 HashMap。因为也是一个 hash 表

特征：

- 无序
- 元素不可重复
- 查找快
- 支持交集、并集、差集等功能

Set 的常见命令：

| **命令**             | **描述**            |
| ------------------ | ----------------- |
| `SADD k v ...`     | 添加一个或多个元素         |
| `SREM k v ...`     | 移除指定元素            |
| `SCARD k`          | 返回 元素个数           |
| `SISMEMBER k v`    | 判断元素是否存在          |
| `SMEMBERS`         | 获取所有元素            |
| `SINTER k1 k2 ...` | 求 key1 与 key2 的交集 |
| `SDIFF k1 k2 ...`  | 求 key1 与 key2 的差集 |
| `SUNION k1 k2 ...` | 求 key1 和 key2 的并集 |

> 交集、差集、并集
> 
> ![400](assets/image-20220525112632214.png)

## SortedSet 类型

> 与 Java 中的 `TreeSet` 有些类似，但底层数据结构却差别很大

可排序的 set 集合

- 每一个元素都带有一个 score 属性，**基于 score 对元素排序**
- 底层的实现是一个跳表（SkipList）加 hash 表

特性：

- 可排序
- 元素不重复
- 查询速度快

> 因为 SortedSet 的可排序特性，经常被用来实现排行榜这样的功能。

SortedSet 的常见命令：

| **命令**                    | **描述**                       |
| ------------------------- | ---------------------------- |
| `ZADD k score v`          | 添加一个或多个元素，如果已经存在则更新 score    |
| `ZREM k v`                | 删除一个 v                       |
| `ZSCORE k v`              | 获取 v 的 score                 |
| `ZRANK k v`               | 获取 v 的排名                     |
| `ZCARD k`                 | 获取 v 个数                      |
| `ZCOUNT k min max`        | 统计 score 在 `min max` 内的 v 个数 |
| `ZINCRBY k increment v`   | 为 v 的 score 加上增量 increment   |
| `ZRANGE k min max`        | 获取指定 排名 范围内的 v               |
| `ZRANGEBYSCORE k min max` | 获取指定 score 范围内的 v            |
| `ZDIFF`、`ZINTER`、`ZUNION` | 求差集、交集、并集                    |

注意：所有的排名默认升序，要降序在命令的 Z 后面添加 `REV` 即可

> 例如 `ZREVRANK`

# ---------- Redis 客户端

## 命令行客户端

自带的命令行客户端：`redis-cli`，使用方式如下： 
 
```shell
redis-cli [options] [commonds]
```

常见的 `options` 有： 

   - `-h 127.0.0.1`：指定要连接的 redis 节点的 IP 地址，默认是 127.0.0.1
   - `-p 6379`：指定要连接的 redis 节点的端口，默认是6379
      - 注意：搭建了集群后这里别忘了
   - `-a 132537`：指定 redis 的访问密码

 `commonds` 就是 Redis 的操作命令，例如： 

   - `ping`：与 redis 服务端做心跳测试，服务端正常会返回 pong
   - 不指定 commond，会进入 `redis-cli` 的交互控制台

![700](assets/image-20220524201258092.png)

## 图形化客户端

> 下载地址：[https://pan.baidu.com/s/1sxQTOt-A5MCvVZnlgDf0eA?pwd=1234](https://pan.baidu.com/s/1sxQTOt-A5MCvVZnlgDf0eA?pwd=1234)

RESP、QuickRedis

## Java 客户端

> 可以在 Redis 官网查看所有客户端以及推荐的客户端：[https://redis.io/docs/clients](https://redis.io/docs/clients)

对于 Java，主要推荐以下 3 种：

1. Jedis：和原生 redis 命令行的命令一致，学习成本最低（注意它是线程不安全的，通常配合连接池使用）
2. lettuce：和 Spring 兼容最好（Spring Data Redis 默认集成）
3. Redission：提供了和 Java 集合用法一致的分布式集合，适用于**更复杂的业务场景**

![](assets/image%20(1).png)

### Jedis 使用

#### 快速入门

> Jedis的官网地址： [https://github.com/redis/jedis](https://github.com/redis/jedis)

引入依赖

```xml
<!--引入Jedis依赖-->
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>4.2.0</version>
</dependency>

<!--引入单元测试依赖-->
<dependency>
  <groupId>org.junit.jupiter</groupId>
  <artifactId>junit-jupiter</artifactId>
  <version>5.8.2</version>
  <scope>test</scope>
</dependency>
```

建立连接、操作数据、释放资源

```java
public class JedisTest {
    private Jedis jedis;

    /**
     * 被test注解修饰的方法每次执行其他方法前自动执行
     * 1、建立连接
     */
    @BeforeEach
    void setUp() {
        // 1. 获取连接
        jedis = new Jedis("192.168.111.154", 6379);
        // 2. 设置密码
        jedis.auth("990117");
        // 3. 选择库（默认是下标为0的库）
        jedis.select(0);
    }

    /**
     * 2、操作数据
     */
    @Test
    public void testString() {
        // 1、操作String
        String result = jedis.set("url", "https://www.oz6.cn");
        System.out.println("result = " + result);
        String url = jedis.get("url");
        System.out.println("url = " + url);
	    // result = OK
	    // url = https://www.oz6.cn

		// 2、操作hash
        jedis.hset("user:1", "name", "Jack");
        jedis.hset("user:1", "age", "21");
        Map<String, String> map = jedis.hgetAll("user:1");
        System.out.println("map = " + map);
    }

    /**
     * 被该注解修饰的方法会在每次执行其他方法后执行
     * 3、释放资源
     */
    @AfterEach
    void tearDown() {
        if (jedis != null) {
            jedis.close();
        }
    }
}
```

#### Jedis 连接池

Jedis 本身是**线程不安全**的，并且频繁的创建和销毁连接会有性能损耗，因此推荐使用 Jedis 连接池代替 Jedis 的直连方式

> 多线程情况下让每个线程有自己独立的 jedis 实例，可变为线程安全

```java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;

    static {
        // 配置连接池
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 最大连接
        jedisPoolConfig.setMaxTotal(8);
        // 最大空闲连接
        jedisPoolConfig.setMaxIdle(8);
        // 最小空闲连接
        jedisPoolConfig.setMinIdle(0);
        // 最长等待时间
        jedisPoolConfig.setMaxWaitMillis(200);
        // 创建连接池对象
        jedisPool = new JedisPool(
            jedisPoolConfig,
            "192.168.111.154",
            6379,
            1000,
            "990117");
    }

    public static Jedis getJedis() {
        return jedisPool.getResource();
    }
}
```
```java
@BeforeEach
void setUp() {
//        // 1. 获取连接
//        jedis = new Jedis("192.168.111.154", 6379);
//        // 2. 设置密码
//        jedis.auth("990117");
//        // 3. 选择库（默认是下标为0的库）
//        jedis.select(0);
    jedis = JedisConnectionFactory.getJedis();
}
```

# ---------- Spring Data Redis
## 介绍

Spring Data 是 Spring 中数据操作的模块，包含对各种数据库的集成，其中对 Redis 的集成模块就叫做 Spring Data Redis
`
> 官网地址：[https://spring.io/projects/spring-data-redis](https://spring.io/projects/spring-data-redis)

- 整合 Redis 客户端 **Lettuce（默认） 和 Jedis**
- RedisTemplate 统一 API 操作 Redis
- 支持 Redis 的 发布订阅模型
- 支持 Redis 哨兵 和  Redis 集群
- 支持基于 Lettuce 的响应式编程
- 支持基于 JDK、JSON、字符串、Spring 对象的数据*序列化及反序列化*
- 支持基于 Redis 的 JDKCollection 实现

![500](assets/image%20(2).png)

RedisTemplate 工具类封装了各种对 Redis 的操作，并且将不同数据类型的操作 API 封装到了不同的类型中：

![700](assets/image-20220525140217446%201.png)

## 快速入门

> SpringBoot 已经提供了对 Spring Data Redis 的支持

引入依赖（连接池）

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--
连接池依赖
spring-boot-starter-data-redis里的commons-pool2设置了<optional>true</optional>
所以还要引入
-->
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-pool2</artifactId>
</dependency>
```

配置文件

```yaml
spring:
   redis:
     host: 192.168.230.88 #指定redis所在的host
     port: 6379  #指定redis的端口
     password: 132537  #设置redis密码
     lettuce:
       pool:
         max-active: 8 #最大连接数
         max-idle: 8 #最大空闲数
         min-idle: 0 #最小空闲数
         max-wait: 100ms #连接等待时间
```

```yaml
spring:
  data:
    redis:
      host: 192.168.111.154
      password: 990117
      port: 6379
      lettuce:
        pool:
          max-active: 8
          max-idle: 8
          min-idle: 0
          max-wait: 100ms
```

测试

```java
@SpringBootTest
class Redis02SpringdataredisApplicationTests {
    @Resource
    private RedisTemplate redisTemplate;

    @Test
    void testString() {
        // redisTemplate.opsForValue().set("name","虎哥");
        // Object name = redisTemplate.opsForValue().get("name");
        ValueOperations ops = redisTemplate.opsForValue();
        ops.set("name","虎哥2");
        Object name = ops.get("name");
        System.out.println(name);
    }
}
```

## RedisSerializer（序列化）

> 实际开发中，RedisTemplate 通常是 `<String, Object>` 这种泛型

RedisTemplate 可以接收任意 Object 作为 value 写入 Redis，只不过写入前会把 Object 序列化为【字节形式】

> RedisTemplate 默认序列化方式是 jdk 的（可读性差、内存占用较大），所以我们存入的 Object 要实现序列化接口，不然会报错，但在实际开发中我们并不想用这种序列化方式，我们都会采用 JSON 方式的序列化。

![600](assets/image-20220525170205272%201.png)

RedisTemplate 的两种 json 序列化实践方案，两种方案各有优缺点

方案一：

1. 自定义 RedisTemplate
2. 修改 RedisTemplate 的序列化器为 `GenericJackson2JsonRedisSerializer`

方案二：

1. 使用 StringRedisTemplate
2. 写 Redis ，**手动**把对象**序列化**为 JSON
3. 读 Redis ，**手动**把读取到的 JSON **反序列化**为对象

### 自定义 RedisTemplate 序列化

> 这里没有引入 springmvc，所以要手动导入 json 的包

```xml
<!-- Jackson依赖 -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

配置类自定义 RedisTemplate

```java
@Configuration
public class RedisConfig {
    @SuppressWarnings("all") // connectionFactory爆红，但是能运行
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        // 1.创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 2.设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 3.创建序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 4.设置key和hashKey采用String的序列化方式
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 5.设置value和hashValue采用json的序列化方式
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        return template;
    }
}
```

已经将 RedisTemplate 的 key 设置为 String 序列化，value 设置为 Json 序列化的方式，因此可以直接向 redis 中插入一个对象

```java
@SpringBootTest
class Redis02SpringdataredisApplicationTests {
    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    @Test
    void testSaveUser() {
        ValueOperations ops = redisTemplate.opsForValue();
        // 序列化
        ops.set("user:100", new User("虎哥",21));
        // 反序列化
        User user = (User) ops.get("user:100");
        System.out.println(user);
    }
}
```

![600](assets/image-20220525170925364.png)

![600](assets/image-20220525171340322.png)

为了在**反序列化**时知道对象的类型，JSON 序列化器会**将类的 class 类型写入 json 结果中**，存入 Redis，会带来**额外的内存开销**

> 通过下文的 `StringRedisTemplate` 可以解决这个问题

反序列化的时候可以向下转型为 User 对象

![600](assets/image%20(3).png)

### StringRedisTemplate

统一使用 String 序列化器处理 value，只能存储 **String 类型的 key 和 value**。当需要存储 Java 对象时，手动完成对象的序列化和反序列化。

![700](assets/image-20220525172001057.png)

Spring 默认提供了一个 StringRedisTemplate 类，它的 key 和 value 的序列化方式默认就是 String 方式

---
使用 StringRedisTemplate
`
```java
@SpringBootTest
class Redis02SpringdataredisApplicationTests {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final ObjectMapper mapper = new ObjectMapper();

    @Test
    void testSaveUser2() throws JsonProcessingException {
		// ValueOperations<String, String> ops = stringRedisTemplate.opsForValue();
        User user = new User("虎哥", 21);
        String json = mapper.writeValueAsString(user);
        stringRedisTemplate.opsForValue().set("user:200", json);
        String jsonUser = stringRedisTemplate.opsForValue().get("user:200");
        System.out.println(jsonUser);
    }
}
```

![600](assets/image-20220525172508234.png)

# -------------------- 实战篇

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693397930689-f148c778-065b-4c08-8dab-254c6328a33f.png?x-oss-process=image%2Fresize%2Cw_1299%2Climit_0#averageHue=%23f4f0f0&from=url&id=GIBYp&originHeight=682&originWidth=1299&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&status=done&style=none&title=)

前端启动：在nginx目录里开个终端

```bash
start nginx.exe
```

# ---------- Redis 共享 Session 登录

## 基于 Session 实现

基于 Session 实现的登录/注册成功后，**不需要返回登录凭证**的，

> 每个 Session 都一个唯一的 SessionId，会保存到 Cookie 里，客户端每次请求都会携带 SessionId，获取 Session 对象

![](assets/image%20(4).png)

![](assets/image%20(5).png)

## 基于 Redis 实现

多台 tomcat 不共享 session 存储空间

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694355762551-51409f6a-faef-49f0-b674-76279f098d3e.png#averageHue=%23f7f4f3&clientId=u13004d99-a575-4&from=paste&height=405&id=u9820fc73&originHeight=701&originWidth=1391&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=383288&status=done&style=none&taskId=ucba88eef-879a-4ba1-ad8f-96df80f6b44&title=&width=802.8860227741011)

Redis 独立于 tomcat 服务器，基于 Redis 实现共享 Session 登录

![](assets/image%20(6).png)


![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694356290609-aec11291-adb9-41af-b3fa-8d9d08827f6d.png#averageHue=%23f0ecec&clientId=u13004d99-a575-4&from=paste&height=380&id=u21b138cf&originHeight=659&originWidth=1354&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=372544&status=done&style=none&taskId=u4b2f710e-f547-48a1-b43e-bc8eac1a989&title=&width=781.5296008886648)

注意事项：

1. 存入 Redis 的数据一定要设置保存时间
2. 存入 Redis 的数据尽量保证精简和安全，比如存入用户信息时可以移除密码等敏感数据

### 短信验证码登录

1、发送验证码

> 路径：`@PostMapping("code")`

```java
@Override
public Result sendCode(String phone) {
	// 校验手机号
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误！");
    }
    String code = RandomUtil.randomNumbers(6);
    // 设置TTL
    stringRedisTemplate.opsForValue()
            .set(LOGIN_CODE_KEY + phone, code, LOGIN_CODE_TTL, TimeUnit.MINUTES);
    // 模拟发送验证码
    log.info("发送短信验证码成功，验证码：{}", code);
    return Result.ok();
}
```

2、短信验证码登录、注册

采用 hash 代替 json 字符串 存储用户信息

```java
@Override
public Result login(LoginFormDTO loginForm) {
	// 校验手机号
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误！");
    }
    
    // ---------- 校验验证码
    String cacheCode = stringRedisTemplate.opsForValue().get(LOGIN_CODE_KEY + phone);
    // ！！！这里不能用 cacheCode != loginForm.getCode()
    if (cacheCode == null || !cacheCode.equals(loginForm.getCode())) {
        return Result.fail("验证码错误！");
    }
    
    // ---------- db：手机号查询用户是否存在
    User user = query().eq("phone", phone).one();
    if (user == null) {
	    // 不存在创建新用户
        user = createUserWithPhone(phone);
    }
    
    // ---------- Redis存入用户信息
    // 1、key=token
    String token = UUID.randomUUID().toString();
    String tokenKey = LOGIN_USER_KEY + token;
    // 2、value=userMap
    // 用户信息脱敏，User --> UserDTO
    UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
	// 采用 hash 代替 json字符串 存储用户信息。UserDTO --> Map
    Map<String, Object> userMap = BeanUtil.beanToMap(
		    // BeanUtil是hutool工具包
            userDTO,
            new HashMap<>(),
            CopyOptions.create()
                    // 是否忽略空字段
                    .setIgnoreNullValue(true)
                    // 自定义属性值转换规则
                    // userDTO的id是Long类型，String序列化会报错
                    .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString())
    );
    stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);
    stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);
    
    // ---------- token 返回给前端用户
    return Result.ok(token);
}
```

### 校验登录状态

已登录用户访问系统（任何路径）后，要刷新 token 的 TTL

不是所有路径都需要校验登录状态（登录注册页面、首页 不需要）

所以分开设置 **token 刷新拦截器**和**登录校验拦截器（设置排除的路径）**

![](assets/image%20(7).png)

请求-->token 刷新拦截器-->登录拦截器

```java
// token 刷新拦截器
public class RefreshTokenInterceptor implements HandlerInterceptor {  
    StringRedisTemplate stringRedisTemplate;  
  
    public RefreshTokenInterceptor(StringRedisTemplate srt) {  
        this.stringRedisTemplate = srt;  
    }  
  
    @Override  
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {  
        // 获取token  
        String token = req.getHeader("authorization");  
        if (StrUtil.isEmpty(token)) {  
            // return str == null || str.length() == 0;  
            // 没有token，放行  
            return true;  
        }  
        // redis取出用户信息  
        String key  = LOGIN_USER_KEY + token;  
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(key);  
        if (userMap.isEmpty()) {  
            // 过期 或 假token，放行  
            return true;  
        }  
        UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);  
        // 保存到ThreadLocal  
        UserHolder.saveUser(userDTO);  
        // 刷新token TTL  
        stringRedisTemplate.expire(key, LOGIN_USER_TTL, TimeUnit.MINUTES);  
        return true;  
    }  
  
    @Override  
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {  
        // 移除用户  
        UserHolder.removeUser();  
    }  
}
```

```java
// 登录校验拦截器  
public class LoginInterceptor implements HandlerInterceptor {  
    @Override  
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) throws Exception {  
        if (UserHolder.getUser() == null) {  
            res.sendError(401);  
            // 拦截  
            return false;  
        }  
        return true;  
    }  
}
```

配置拦截器

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Resource
    StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // token刷新拦截器
        registry
                .addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate))
                .addPathPatterns("/**")
                .order(0);
        // 登录校验拦截器
        registry
                .addInterceptor(new LoginInterceptor())
                // 登录校验排除路径
                .excludePathPatterns(
                        "/shop/**",
                        "/shop-type/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                )
                .order(1);
    }
}
```

前端通过 axios 的拦截器，将 token 存到请求头里，key 为 authorization

![500](assets/image%20(8).png)

# 缓存

缓存就是数据交换的缓冲区(称作 cache)，是存储数据的临时地方，一般读写性能较高

![](assets/image%20(9).png)

缓存的作用（优点）：

1. 降低后端负载
2. 提高读写效率，降低响应时间

缓存的成本（缺点）

1. 保证数据一致性
2. 额外开发和解决缓存带来的问题，增加代码成本
3. 额外引入中间件，增加运维成本

## 添加 Redis 缓存

![](assets/image%20(10).png)

给商铺查询添加缓存

> 请求路径：`@GetMapping("/{id}")`

```java
@Resource
StringRedisTemplate stringRedisTemplate;

@Override
public Result queryShopById(Long id) {
    // 1 查询缓存
    String shopKey = CACHE_SHOP_KEY + id;
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    if (StrUtil.isNotBlank(shopJson)) {
	     // 2 命中直接返回
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // 3 未命中查数据库
    Shop shop = getById(id);
    if (shop == null) {
		// 不存在，返回404
        return Result.fail("商户不存在");
    }
    // 写入Redis
    stringRedisTemplate.opsForValue().set(shopKey,JSONUtil.toJsonStr(shop));
    // 返回商铺信息
    return Result.ok(shop);
}
```

## 缓存更新策略

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694530148845-78bb51ce-2d0c-4d02-8a0f-34b8f506c42a.png#averageHue=%23e3d4d3&clientId=u8b7321a1-0d1f-4&from=paste&height=387&id=ubf5044ef&originHeight=670&originWidth=1379&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=281783&status=done&style=none&taskId=u2cddcf1a-6828-448a-95b5-ae3d4a23f96&title=&width=795.9596156761218)

缓存更新策略的最佳实践方案

- 低一致性需求：使用 Redis 自带的内存淘汰机制
- 高一致性需求：**主动更新**，并以**超时剔除作为兜底**方案

### 主动更新策略

> 主动更新策略 也叫 缓存读写策略

主动更新策略中，Cache Aside Pattern（旁路缓存模式） 企业里最常用

- 读操作
   - 缓存命中则直接返回
   - 未命中则查询 db，写入缓存，设定 TTL
- 写操作
   - 先写 db，再删除缓存

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694530791106-012699b6-56b2-4df9-abee-155c7ba73d1a.png#averageHue=%23efeeee&clientId=u8b7321a1-0d1f-4&from=paste&height=287&id=u428e4ace&originHeight=497&originWidth=1404&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=212456&status=done&style=none&taskId=u23dec05b-9739-4080-a41b-8ef8f64e5ff&title=&width=810.3896304635787)

操作缓存和数据库时有三个问题需要考虑

1）删除 or 更新 缓存?

- ❌更新缓存：每次更新数据库都更新缓存，**无效写操作较多**
- ✔️删除缓存：更新数据库时让缓存失效，查询时再更新缓存

2）如何保证缓存与数据库的操作的 同时成功或失败?

- 单体系统：将缓存和数据库操作放在一个事务里
- 分布式系统：TCC 等分布式事务方案

3）先操作 缓存 or 数据库?

都可能发生“线程安全”问题，导致缓存和数据库**不一致**的问题

- 先删缓存，后更新数据库：发生的**概率高**
- 先更新数据库，后删缓存：发生的概率很低，因为**缓存的速度高于数据库很多**

> 方案一不一致的情况：线程 2 删除缓存 --> 在线程 1 查询数据库写入缓存 --> 线程 2 更新数据库
> 
> 方案二不一致的情况：（刚好缓存失效）线程 1 查询缓存未命中，查数据库 --> 线程 2 更新数据库，删除缓存 --> 线程 1 写入缓存

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694533096441-4c365ba0-c0fa-4832-aa99-62b8f33ee912.png#averageHue=%23f2f1f1&clientId=u74749200-d9d1-4&from=paste&height=412&id=u0ae05ed9&originHeight=714&originWidth=1305&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=254032&status=done&style=none&taskId=u3d57ebd4-7597-4935-bab6-8f5f6a001ad&title=&width=753.2467719052494)

### shop 缓存案例

给查询商铺的缓存添加“超时剔除”和“主动更新”的策略

1）id 查询店铺：db 查询结果写入缓存，添加**设置 TTL**

```java
// 存在商铺，写入Redis，返回商铺信息
// 超时剔除
stringRedisTemplate.opsForValue()
        .set(shopKey, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
```

2）id 修改店铺：先修改数据库，再删除缓存，要**添加事务！**

> /shop

```java
@Override
@Transactional
public Result updateShop(Shop shop) {
    Long shopId = shop.getId();
    if (shopId == null) {
        return Result.fail("店铺id不能为空");
    }
    // 更新数据库
    updateById(shop);
    // 删除缓存
    stringRedisTemplate.delete(CACHE_SHOP_KEY + shopId);
    return Result.ok();
}
```

无前台页面，后台测试

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694536289070-7fae89bc-88ec-42ed-9dd7-991a8efac8c5.png#averageHue=%23fbfafa&clientId=u74749200-d9d1-4&from=paste&height=243&id=u914ec4f8&originHeight=421&originWidth=1284&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=46382&status=done&style=none&taskId=u3414a1af-196d-4075-a283-8ff460f7f18&title=&width=741.1255594837855)

## 缓存穿透

客户端请求的数据在**缓存和数据库中都不存在**，这样缓存永远不生效，请求都会打到数据库里

常见的解决方案：

- 缓存空对象✔️
   - 优点：简单
   - 缺点：
      - 额外的内存消耗（解决：设置TTL）
      - 短期的数据不一致（解决：真正插入的时候覆盖）

- 布隆过滤
   - 优点：内存占用少，没有多余key
   - 缺点：
      - 实现复杂
      - 存在误判可能

> 上面都是一些被动的解决方案，主动的解决方案（预防）：
> 
> - 增强 id 的复杂度
> - 增强对请求数据的格式校验
> - 增强用户权限校验
> - 热点参数限流

![600](assets/image%20(11).png)

### shop 查询案例

![](assets/image%20(12).png)

id 查询商铺

```java
@Override
public Result queryShopById(Long id) {
    // 查询缓存
    String shopKey = CACHE_SHOP_KEY + id;
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    if (StrUtil.isNotBlank(shopJson)) {
	    // 命中直接返回
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // 【缓存穿透】，命中为空值""，返回错误
    if(shopJson!=null){
        return Result.fail("商户不存在");
    }
    // 未命中查数据库
    Shop shop = getById(id);
    if (shop == null) {
        // 【缓存穿透】，将空值""写入Redis，设置TTL
        stringRedisTemplate.opsForValue()
                .set(shopKey, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
        return Result.fail("商户不存在");
    }
    // 存在商铺，写入Redis
    // 【超时剔除】
    stringRedisTemplate.opsForValue()
            .set(shopKey, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
    // 返回商铺信息
    return Result.ok(shop);
}
```

测试：`http://localhost:8081/shop/0`

## 缓存雪崩

在同一时段大量的缓存 key 同时失效 或者 Redis 服务宕机 导致大量请求到达数据库

解决方案：

- 给不同 key 的 TTL 添加随机值
- Redis 集群（针对宕机）
- 给缓存业务添加降级限流策略（缓存全崩完了）
- 业务添加多级缓存（浏览器缓存、nginx 缓存、jvm 本地缓存）

## 缓存击穿

也叫热点 Key 问题，一个**被高并发访问**并且缓存重建业务较复杂（重建时间较长）的 key 突然失效了（过期），无数的请求会瞬间给数据库带来巨大的压力

![600](assets/image%20(13).png)

解决方案（根据需求选择）：

1）*互斥锁*：

缓存未命中，先获取互斥锁，查询 db 重建缓存数据后，再释放互斥锁。保证**只有一个请求会落到数据库上**

- 优点：
	- **保证一致性**
- 缺点：
	- 线程要等待，性能受影响
	- 有死锁风险

2）*互斥锁+逻辑过期*

key 是永久的，由后端添加一个 expire 字段（LocalDateTime 实现）表示过期时间；

发现逻辑过期后，获取锁成功的线程，另开一个线程异步重建缓存，获取锁失败的线程直接返回过期数据

- 优点：
	- 线程无需等待，**性能较好**
- 缺点
	- 不保证一致性
	- 有额外的内存开销

![](assets/image%20(15).png)

### 互斥锁案例

![500](assets/image%20(14).png)

 利用命令 `SETNX` 实现互斥锁，对应 StringRedisTemplate 的方法如下

```java
Boolean setIfAbsent(K key, V value, long timeout, TimeUnit unit);
```

> SETNX：添加一个 String 类型的键值对，前提是这个 key 不存在，否则不执行
 
```java
@Override
public Result queryShopById(Long id) {
    Shop shop = queryWithMutex(id);
    if (shop == null) {
        return Result.fail("商铺不存在");
    }
    return Result.ok(shop);
}

/**  
 * id查询商铺——缓存击穿解决——互斥锁  
 */  
public Shop queryWithMutex(Long id) {  
    // 查询缓存  
    String shopKey = CACHE_SHOP_KEY + id;  
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);  
    if (StrUtil.isNotBlank(shopJson)) {  
        // 命中直接返回  
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);  
        return shop;  
    }  
    if (shopJson != null) {  
        // 【缓存穿透】 命中为空值""  
        return null;  
    }  
    // 未命中，查询 db    String lockKey = LOCK_SHOP_KEY + id;  
    Shop shop = null;  
    try {  
        // 获取【互斥锁】  
        boolean isLock = tryLock(lockKey);  
        if (!isLock) {  
            // 获取失败，等待缓存数据重建，递归调用  
            Thread.sleep(50);  
            return queryWithMutex(id);  
        }  
        shop = getById(id);  
        Thread.sleep(200); // 模拟重建的延迟  
        if (shop == null) {  
            // 【缓存穿透】 创建无效key  
            stringRedisTemplate.opsForValue()  
                    .set(shopKey, "", CACHE_NULL_TTL, TimeUnit.MINUTES);  
            return null;  
        }  
        // 写入缓存，【超时剔除】  
        stringRedisTemplate.opsForValue()  
                .set(shopKey, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);  
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    } finally {  
        // 释放【互斥锁】  
        unLock(lockKey);  
    }  
    return shop;  
}

/**
 * 获取锁
 */
private boolean tryLock(String key) {
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
    // 直接返回flag，自动拆箱可能会空指针
    return BooleanUtil.isTrue(flag);
}

/**
 * 释放锁
 */
private void unLock(String key) {
    stringRedisTemplate.delete(key);
}
```

测试：`http://localhost:8081/shop/3`

![600](assets/image%20(16).png)

### 逻辑过期案例

> 需要把数据提前写入Redis，不然永远命中不了

![](assets/image%20(17).png)

提前插入过期数据

```java
@Test
public void testSaveShopToRedis() throws InterruptedException {
    // saveShopToRedis是ShopServiceImpl的新方法
    // 插入一条10s过期的测试数据
    shopService.saveShopToRedis(1L, 10L);
}
```

业务逻辑

```java
// 线程池
private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

/**
 * id查询商铺——缓存击穿解决——逻辑过期
 *
 * @param id
 * @return
 */
public Shop queryWithLogicalExpire(Long id) {
    String shopKey = CACHE_SHOP_KEY + id;
    // # 查询缓存
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    // ## 未命中，返回空
    if (StrUtil.isBlank(shopJson)) {
        return null;
    }
    // # 命中，判断缓存是否过期
    RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
    Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);
    // 获取逻辑过期时间
    LocalDateTime expireTime = redisData.getExpireTime();
    // ## 未过期，返回shop
    if (expireTime.isAfter(LocalDateTime.now())) {
        return shop;
    }
    // # 过期，获取互斥锁
    boolean isLock = tryLock(LOCK_SHOP_KEY + id);
    // ## 获取到锁，开启独立线程重建缓存
    if (isLock) {
        CACHE_REBUILD_EXECUTOR.submit(() -> {
            try {
                // 缓存重建
                this.saveShopToRedis(id, 20L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                // 释放锁
                unLock(LOCK_SHOP_KEY + id);
            }
        });
    }
    // ## 没获取到锁，返回过期shop
    return shop;
}

// 重建缓存
public void saveShopToRedis(Long id, Long expireSeconds) throws InterruptedException {
    // # 查询shop
    Shop shop = getById(id);
    // 模拟延时
    Thread.sleep(200);
    // # 封装逻辑过期时间
    RedisData redisData = new RedisData();
    redisData.setData(shop);
    redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireSeconds));
    // # 写入Redis，不要设置TTL
    stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(redisData));
}

/**
 * 获取锁
 */
private boolean tryLock(String key) {
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
    // 不要直接返回flag，flag自动拆箱可能会空指针异常
    return BooleanUtil.isTrue(flag);
}

/**
 * 释放锁
private void unLock(String key) {
    stringRedisTemplate.delete(key);
}
```
## 缓存工具封装

> [https://www.bilibili.com/video/BV1cr4y1671t/?p=46&spm_id_from=pageDriver&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa](https://www.bilibili.com/video/BV1cr4y1671t/?p=46&spm_id_from=pageDriver&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa)

基于 StringRedisTemplate 封装一个缓存工具类 CacheClient，满足下列需求：

- 方法 1：将**对象**序列化为 json 并写入缓存，可设置 TTL 
- 方法 2：方法 1 的基础上，可以设置“逻辑过期”时间
- 方法 3：根据 key 查询缓存，并**反序列化**；利用“缓存空值”解决【缓存穿透】
- 方法 4：根据 key 查询缓存，并**反序列化**；利用“逻辑过期”解决【缓存穿透】

技术点：泛型、函数式编程

---
工具类

```java
@Slf4j
@Component
public class CacheClient {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    // 线程池
    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

    /**
     * Java对象序列化为json并存储在string类型的key中，并且可以设置TTL过期时间
     *
     * @param key
     * @param value
     * @param time
     * @param unit
     */
    public void set(String key, Object value, Long time, TimeUnit unit) {
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);
    }

    /**
     * 将任意]ava对象序列化为ison并存储在string类型的key中
     * 并且可以设置逻辑过期时间，用于处理缓存击穿问题
     *
     * @param key
     * @param value
     * @param time
     * @param unit
     */
    public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {
        RedisData redisData = new RedisData();
        redisData.setData(value);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
    }

    /**
     * 根据指定的key查询缓存，并反序列化为指定类型
     * 利用缓存空值的方式解决缓存穿透问题
     *
     * @param keyPrefix
     * @param id
     * @param type       返回值类型
     * @param dbFallback id查询数据库逻辑，需实现dbFallback接口
     * @param time       缓存保存时间
     * @param unit       时间单位
     * @param <R>
     * @param <ID>
     * @return
     */
    public <R, ID> R queryWithPassThrough(
            String keyPrefix,
            ID id,
            Class<R> type,
            Function<ID, R> dbFallback,
            Long time, TimeUnit unit) {

        // # Redis查询缓存
        String key = keyPrefix + id;
        String json = stringRedisTemplate.opsForValue().get(key);
        // ## 命中且值非空，直接返回
        if (StrUtil.isNotBlank(json)) {
            return JSONUtil.toBean(json, type);
        }
        // ## 命中为空值""，返回null
        if (json != null) {
            return null;
        }
        // # 未命中查数据库
        R r = dbFallback.apply(id);
        // ## 不存在，返回null
        if (r == null) {
            // 将空值""写入Redis
            stringRedisTemplate.opsForValue()
                    .set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
        // # 存在商铺，写入Redis，返回商铺信息
        // 超时剔除策略兜底
        this.set(key, r, time, unit);
        return r;
    }


    public <R, ID> R queryWithLogicalExpire(
            String keyPrefix,
            ID id,
            Class<R> type,
            Function<ID, R> dbFallback,
            Long time, TimeUnit unit) {

        String key = keyPrefix + id;
        // # 查询缓存
        String json = stringRedisTemplate.opsForValue().get(key);
        // ## 未命中，返回空
        if (StrUtil.isBlank(json)) {
            return null;
        }
        // # 命中，判断缓存是否过期
        RedisData redisData = JSONUtil.toBean(json, RedisData.class);
        R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
        // 获取逻辑过期时间
        LocalDateTime expireTime = redisData.getExpireTime();
        // ## 未过期，返回r
        if (expireTime.isAfter(LocalDateTime.now())) {
            return r;
        }
        // # 过期，获取互斥锁
        String lockKey = LOCK_SHOP_KEY + id;
        boolean isLock = tryLock(lockKey);
        // ## 获取到锁，开启独立线程重建缓存
        if (isLock) {
            CACHE_REBUILD_EXECUTOR.submit(() -> {
                try {
                    // 查数据库
                    R r1 = dbFallback.apply(id);
                    // 存入Redis
                    this.setWithLogicalExpire(key, r1, time, unit);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                } finally {
                    // 释放锁
                    unLock(lockKey);
                }
            });
        }
        // ## 没获取到锁，返回过期r
        return r;
    }

    /**
     * 获取锁
     *
     * @param key
     * @return
     */
    private boolean tryLock(String key) {
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
        // 不要直接返回flag，flag自动拆箱可能会空指针异常
        return BooleanUtil.isTrue(flag);
    }

    /**
     * 释放锁
     *
     * @param key
     */
    private void unLock(String key) {
        stringRedisTemplate.delete(key);
    }
}

```

使用工具类

```java
@Override
public Result queryShopById(Long id) {
    // 解决缓存穿透
    Shop shop = cacheClient.queryWithPassThrough(
            CACHE_SHOP_KEY,
            id,
            Shop.class,
            // shopId -> getById(shopId),
            this::getById,
            CACHE_SHOP_TTL,
            TimeUnit.MINUTES
    );

    // 逻辑过期解决缓存击穿
//        Shop shop = cacheClient.queryWithLogicalExpire(  
//                CACHE_SHOP_KEY,  
//                id,  
//                Shop.class,  
//                // shopId -> getById(shopId),  
//                this::getById,  
//                CACHE_SHOP_TTL,  
//                TimeUnit.MINUTES  
//        );

    if (shop == null) {
        return Result.fail("商铺不存在");
    }
    return Result.ok(shop);
}
```

# 优惠券秒杀（分布式锁）

## 全局唯一 ID

当用户抢购时，就会生成订单并保存到 tb_voucher_order 这张表中，而订单表如果使用数据库自增 ID 就存在一些问题：

- id 的规律性太明显，容易透露给用户信息（比如一天产生了多少单）
- 受单表数据量的限制，每天产生的订单都很多，分多张表存储，ID 会重复

全局 ID 生成器：一种在分布式系统下用来生成全局唯一 ID 的工具，一般要满足以下特性：

- 唯一性
- 高可用：任何时候都不能挂
- 高性能：生成速度要快
- 递增性：要确保全局逐渐增大，有利于数据库创建索引，提高插入的速度
- 安全性：id 规律性不能太明显

> 一些全局 ID 生成策略：
> 
> - UUID（16 进制字符串，缺少单调递增的特性）
> - Redis 自增
> - snowflake 算法（比较依赖时钟）
> - 数据库自增（专门弄一张表来自增，性能不如 Redis 自增）

为了增加 ID 的安全性，我们可以不直接使用 Redis 自增的数值，而是拼接一些其他信息

思想：时间戳+计数器

ID 的组成部分：

- 符号位：1bit，永远为0
- 时间戳：31bit，以秒为单位，可以使用69年
- 序列号：32bit，秒内的计数器，支持每秒产生2^32个不同ID

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694623693347-ac625f6a-f79e-4e10-aabd-2970f1a1554f.png#averageHue=%23f8f2f1&clientId=u1d3a7e9d-c1c7-4&from=paste&height=116&id=ueb76dcb9&originHeight=201&originWidth=1160&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=79621&status=done&style=none&taskId=u3a6d22e0-1094-4420-8976-fc34c2131a3&title=&width=669.5526861379994)

使用 Redis 的 `Incr` 命令，可以实现后 32 位的原子性递增。<br />Redis的key设计为`icr:业务前缀:当前日期`，每天都会从1开始生成序列号，每天一个key方便统计订单量<br />单key设计可能会出现生成序列号数溢出 2^32 的情况，多key的话单日不可能超过2^32次方个订单
```java
@Component
public class RedisIdWorker {
    private static final long BEGIN_TIMESTAMP = 1672531200L;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final int COUNT_BITS = 32;

    public long nextId(String keyPrefix) {
        // # 生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timeStamp = nowSecond - BEGIN_TIMESTAMP;
        // # 生成序列号
        // ## 获取当前日期
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        // ## 序列号自增长
        Long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);
        // # 拼接
        return timeStamp << COUNT_BITS | count;
    }

    public static void main(String[] args) {
        // 生成起始时间
        LocalDateTime time = LocalDateTime.of(2023, 1, 1, 0, 0, 0);
        long second = time.toEpochSecond(ZoneOffset.UTC);
        System.out.println("second=" + second);
    }
}
```
```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class HmDianPingApplicationTests {
    @Resource
    private RedisIdWorker redisIdWorker;

    private ExecutorService es = Executors.newFixedThreadPool(500);

    @Test
    public void testRedisIdWorker() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(300);

        Runnable task = () -> {
            for (int i = 0; i < 100; i++) {
                long id = redisIdWorker.nextId("testOrder");
                System.out.println("id=" + id);
            }
            // 减少计数器
            latch.countDown();
        };

        long begin = System.currentTimeMillis();
        // 共生成30k个全局id
        for (int i = 0; i < 300; i++) {
            es.submit(task);
        }
        // 挂起线程，计数器为0的时候恢复线程
        latch.await();
        long end = System.currentTimeMillis();
        System.out.println("time=" + (end - begin));
    }
}
```

## 优惠券秒杀下单实现

每个店铺都可以发布优惠券，分为平价券和特价券。平价券可以任意购买，而**特价券**需要秒杀抢购

![](assets/image%20(18).png)

> 优惠券相关表:
> 
> - `tb voucher`：基本信息，优惠金额、使用规则等
> - `tb seckill voucher`：库存、开始抢购时间，结束抢购时间。
> 	- 特价优惠券才需要

数据库插入一条秒杀券的数据 `http://localhost:8081/voucher/seckill`

```json
{
    "shopId":1,
    "title":"100元代金券",
    "subTitle":"周一至周五均可使用",
    "rules":"全场通用\\n无需预约\\n可无限叠加\\不兑现、不找零\\n仅限堂食",
    "payValue":8000,
    "actualValue":10000,
    "type":1,
    "stock":100,
    "beginTime":"2023-09-10T10:09:17",
    "endTime":"2023-12-26T12:09:04"
}
```

下单时要判断两点：

1. 秒杀是否开始或者结束
2. 库存是否充足

![600](assets/image%20(19).png)

```java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService voucherService;
    @Resource
    private RedisIdWorker redisIdWorker;

    @Override
    public Result seckillVoucher(Long voucherId) {
        // # db 查询优惠券
        SeckillVoucher voucher = voucherService.getById(voucherId);
        // # 秒杀是否 开始 或 结束
        if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
            return Result.fail("时间未开始");
        }
        if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
            return Result.fail("时间已结束");
        }
        // # 判断库存是否充足
        if (voucher.getStock() < 1) {
            return Result.fail("库存不足");
        }
        // # 扣减库存
        voucher.setStock(voucher.getStock() - 1);
        voucherService.updateById(voucher);
        // # 创建订单
        VoucherOrder voucherOrder = new VoucherOrder();
        long orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        voucherOrder.setUserId(UserHolder.getUser().getId());
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);

        return Result.ok(orderId);
    }
}
```

## 超卖问题（乐观锁）

![](assets/image%20(20).png)
 
超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案就是加锁

- 乐观锁：不加锁，在更新时判断是否有其他线程修改
	- 性能好，存在成功率低的问题

- 悲观锁：添加同步锁，让线程串行执行
	- 简单粗暴，性能一般

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694713542990-22f19e1f-9c77-4a0f-b0a1-236c6a0d4ddd.png#averageHue=%23f2f0f0&clientId=u7b3a1905-9ec7-4&from=paste&height=304&id=uccbdec90&originHeight=526&originWidth=1256&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=231511&status=done&style=none&taskId=ube4c43b0-6ebd-4402-9c8a-2f8f5d578bd&title=&width=724.9639429218339)

乐观锁的关键是判断之前查询得到的数据是否又被修改过，常见的方式有两种

1. 版本号法
2. CAS 法（直接将 stock 做为版本号）

![](assets/image%20(21).png)

![](assets/image%20(22).png)

CAS 法解决超卖问题

```java
// 判断库存是否充足
if (voucher.getStock() < 1) {
    return Result.fail("库存不足");
}
// 扣减库存
boolean success = seckillVoucherService.update()
        .setSql("stock=stock-1")
        .eq("voucher_id", voucherId)
        // 乐观锁，大于0
        .gt("stock",0).update(); 
if (!success) {
    return Result.fail("库存不足");
}
```

## 一人一单（悲观锁）

### 单机实现

同一个特惠优惠券，一个用户只能下一单

![](assets/image%20(23).png)

以上方案存在线程安全问题，可能**同一个用户多个线程都未创建过订单**，并行执行判断订单是否存在，结果是都不存在，出现一人多单

解决方案：加锁

- synchronized 将 userId 作为同步对象，**给每个用户加锁**
	- Long 的 `toString()` 内部还是会返回一个新的字符串对象
	- `userId.toString().intern()` 作为同步对象，可以确保返回的都是同一个 String 对象

- 确保事务提交了之后，才释放锁！
	- Spring 的事务是通过 `VoucherOrderServiceImpl` 的“代理对象”操作的，直接在方法内部调用 `@Transactional` 方法会使事务失效（因为使用 this 调用，而不是代理对象调用）
	- 获取到代理对象调用 `@Transactional` 方法

> `intern()` 会去字符串常量池里找和当前字符串一样的字符串对象的引用，并返回

```java
synchronized (userId.toString().intern()) {
    return createOrder(voucherId);
}
```

允许暴露代理对象

```xml
<!--aspectJ-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

```java
@EnableAspectJAutoProxy(exposeProxy = true) // 是否暴露代理对象
@MapperScan("com.hmdp.mapper")
@SpringBootApplication
public class HmDianPingApplication {
    public static void main(String[] args) {
        SpringApplication.run(HmDianPingApplication.class, args);
    }
}
```

完整实现

```java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService seckillVoucherService;
    @Resource
    private RedisIdWorker redisIdWorker;

    /**  
     * 秒杀下单_单机实现  
     */  
    public Result singleSeckillVoucher(Long voucherId) {  
        // 查询优惠券  
        SeckillVoucher voucher = seckillVoucherService.getById(voucherId);  
        // 判断秒杀是否开始或结束  
        if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {  
            return Result.fail("时间未开始");  
        }  
        if (voucher.getEndTime().isBefore(LocalDateTime.now())) {  
            return Result.fail("时间已结束");  
        }  
        // 优惠券库存是否充足  
        if (voucher.getStock() < 1) {  
            return Result.fail("库存不足");  
        }  
  
        // 给每个用户上锁，订单创建完，事务提交了才释放锁  
        
        Long userId = UserHolder.getUser().getId();  
        synchronized (userId.toString().intern()) {  
            // 获取代理对象（事务）  
            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();  
            return proxy.createVoucherOrder(voucherId);  
        }  
    }  
  
    @Transactional  
    public Result createVoucherOrder(Long voucherId) {  
        Long userId = UserHolder.getUser().getId();  
        // 一人一单，判断当前用户是否有订单记录  
        int count = query()  
                .eq("user_id", userId)  
                .eq("voucher_id", voucherId)  
                .count();  
        if (count > 0) {  
            log.info("用户已经购买过一次");  
            return Result.fail("用户已经购买过一次");  
        }  
        // 扣减库存  
        boolean success = seckillVoucherService.update()  
                .setSql("stock=stock-1")  
                .eq("voucher_id", voucherId)  
                .gt("stock", 0).update(); // CAS  
        if (!success) {  
            return Result.fail("库存不足");  
        }  
        // 创建订单  
        VoucherOrder voucherOrder = new VoucherOrder();  
        long orderId = redisIdWorker.nextId("order");  
        voucherOrder.setId(orderId);  
        voucherOrder.setUserId(userId);  
        voucherOrder.setVoucherId(voucherId);  
        save(voucherOrder);  
          
        return Result.ok(orderId);  
    }

}
```

### 分布式或集群模式下的问题

![600](assets/image%20(24).png)

访问 `http://localhost:8080/api/voucher/list/1`，nginx 负载均衡转发给 8081 和 8082 的服务，同一个用户请求不同的服务，都能拿到锁

Synchronized 只对单个 JVM 有效，多机部署时不同的 JVM 的锁已经不一样了（8080 和 8081 的线程可以同时获取锁）

![](assets/image%20(25).png)

### 分布式锁

满足 分布式系统 或 集群模式 下多进程可见并且互斥的锁

特性：

- 多进程可见
- 互斥
- 高可用
- 高性能
- 安全性：服务挂了锁没释放、死锁

常见的分布式锁实现

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694790963910-6afd8422-cf5a-4763-b684-dac3f97cf6a9.png#averageHue=%23d7c1c0&clientId=u475f6ad9-c04f-4&from=paste&height=284&id=u3575735b&originHeight=492&originWidth=1362&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=187855&status=done&style=none&taskId=u38d1a295-597e-4b4b-86ad-bf137f13ddc&title=&width=786.1472056206511)

### Redis 实现分布式锁

1）获取锁

- 利用 `SETNX` 的互斥性，`SET lock v EX time NX`
- 非阻塞：尝试一次，成功返回 true，失败返回 false

对应 RedisTemplate 的方法 `Boolean setIfAbsent(K key, V value, long timeout, TimeUnit unit)`

2）释放锁

- 手动释放，`DEL lock`
- 超时释放，获取锁时添加超时时间（服务宕机）

![](assets/image%20(26).png)

### 锁过期导致误删锁

> 解决方法：线程标识、lua保证原子性

1）业务阻塞时间过长，手动释放锁之前，锁过期

- 线程 1 获取锁，因为业务阻塞时间过长，导致 锁过期
- 线程 2 获取锁，线程 1 执行完业务后 误放 线程 2 的锁，**线程 3 又能获取锁执行业务**
	- 因为是同一个用户的多个线程，锁的 key 是一样的

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694797795051-dd010bc4-5d40-48ca-b269-9d373e2bee01.png#averageHue=%23fafaf9&clientId=u475f6ad9-c04f-4&from=paste&height=314&id=u2d8e82d9&originHeight=544&originWidth=1359&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=160500&status=done&style=none&taskId=ucd62c3d8-4061-4571-9d2f-8b6c76b0a0c&title=&width=784.4156038461563)

解决方案：线程标识

- 获取锁时，将 value 设置为**当前线程标识**
	- 隐患：两个 JVM 的线程 id 可能会冲突
		- 解决：UUID + 线程 id
- 释放锁时，锁的 value 和 当前线程 一致，才释放

![](assets/image%20(27).png)

![350](assets/image%20(28).png)

2）判断锁标识和释放锁的原子性

- 线程 1 判断锁标识和当前线程一致后，出现业务阻塞（gc 垃圾回收），锁过期
- 线程 2 成功获取锁，执行业务，线程 1 又把线程 2 的锁释放了，**线程 3 又能获取锁执行业务**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694882900476-70bff775-9f0c-4144-8bd5-62c97feac5c2.png#averageHue=%23f6f4f4&clientId=u4124065c-0985-4&from=paste&height=462&id=ud1da0777&originHeight=801&originWidth=1898&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=524708&status=done&style=none&taskId=u0ec19670-05e0-4e34-976a-4e1496c157a&title=&width=1095.5267226637266)

解决方案：

- Redis 提供了 Lua 脚本功能，在一个脚本中编写多条 Redis 命令，确保多条命令执行时的原子性

> Lua 是一种编程语言，基本语法参考: [https://www.runoob.com/lua/lua-tutorial.html](https://www.runoob.com/lua/lua-tutorial.html) 

![500](assets/image%20(30)%201.png)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694936406217-fad8b853-e701-4e97-83c1-2e4a1f2a66f3.png#averageHue=%23d9dcd1&clientId=u6c8c1437-1b33-4&from=paste&height=560&id=u4129a50d&originHeight=971&originWidth=2027&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=1011701&status=done&style=none&taskId=u1008cf4d-a120-4138-9ade-d4df61898b0&title=&width=1169.985598967004)

lua 脚本实现锁的业务流程

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694937611623-17b7a006-3f6b-4bc8-8e36-cb2611899d66.png#averageHue=%23dcddd8&clientId=u6c8c1437-1b33-4&from=paste&height=410&id=u309ddd8d&originHeight=711&originWidth=1468&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=679155&status=done&style=none&taskId=uce02245d-5f8b-4984-83b1-38152b04904&title=&width=847.3304683194682) 

RedisTemplate 调用 lua 脚本的 api

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694937941095-e3b26303-bd39-4727-b1ab-b4540f0e0684.png#averageHue=%23eaefe8&clientId=u6c8c1437-1b33-4&from=paste&height=380&id=u1a33b13d&originHeight=658&originWidth=1478&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=677877&status=done&style=none&taskId=u5036d850-ab32-45f6-bdf6-ddef6789e9c&title=&width=853.102474234451)

### 完整实现

1）Redis 分布式锁实现类

```java
public class SimpleRedisLock implements ILock {

    private String name;
    private StringRedisTemplate stringRedisTemplate;

    private static final String KEY_PREFIX = "lock:";
    private static final String THREAD_ID_PREFIX = UUID.randomUUID().toString(true) + '-';
    private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;

    static {
        UNLOCK_SCRIPT = new DefaultRedisScript<>();
        UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));
        UNLOCK_SCRIPT.setResultType(Long.class);
    }

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean tryLock(long timeoutSec) {
        // 获取线程标识
        String threadId = THREAD_ID_PREFIX + Thread.currentThread().getId();
        // 获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        // 避免空指针
        return BooleanUtil.isTrue(success);
    }

    @Override
    public void unLock() {
        // 调用lua脚本
        stringRedisTemplate.execute(
                UNLOCK_SCRIPT,
                Collections.singletonList(KEY_PREFIX + name),
                THREAD_ID_PREFIX + Thread.currentThread().getId()
        );
    }

//    @Override
//    public void unLock() {
//        // 获取线程标识
//        String threadId = THREAD_ID_PREFIX + Thread.currentThread().getId();
//        // 获取锁的线程标识
//        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
//        // 判断标识是否一致
//        if (threadId.equals(id)) {
//            // 释放锁
//            stringRedisTemplate.delete(KEY_PREFIX + name);
//        }
//    }
}
```

2）业务逻辑

```java
@Override
public Result seckillVoucher(Long voucherId) {
    // 查询优惠券
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
    // 判断秒杀时间是否开始/结束
    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        return Result.fail("时间未开始");
    }
    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("时间已结束");
    }
    // 判断库存是否充足
    if (voucher.getStock() < 1) {
        return Result.fail("库存不足");
    }
    
    // 分布式锁实现
    Long userId = UserHolder.getUser().getId();
    SimpleRedisLock lock = new SimpleRedisLock("order:" + userId,stringRedisTemplate);
    // 获取锁
    boolean isLock = lock.tryLock(12000);
    if (!isLock){
        return Result.fail("不允许重复下单");
    }
    // 执行业务
    try {
        IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
        return proxy.createVoucherOrder(voucherId);
    }finally {
	    // 释放锁
        lock.unLock();
    }
}
```

# Redission

## 介绍/能解决的问题

除了误删之外，现在的分布式锁实现还存在以下几个问题：

- 不可重入：同一个线程无法多次获取同一把锁（递归调用或调用的子函数抢同一把锁时就会出现死锁）
- 不可重试：获取锁只尝试一次就返回 false，没有重试机制
- 超时释放：虽然可以避免死锁，但如果是业务执行耗时较长，也会导致锁释放，存在安全隐患
- 主从一致性：Redis 提供了主从集群，主从同步存在延迟，主节点设置锁成功，还未及时同步到从节点，这时主节点宕机，从节点被选为主节点。但此时从节点还没有锁，仍可以抢锁成功。

Redisson 是一个在 Redis 的基础上实现的 Java 驻内存数据网格 (In-Memory Data Grid)。它不仅提供了一系列的分布式的 Java 常用对象，还提供了许多分布式服务，其中就包含了各种分布式锁的实现。

![](assets/image%20(31).png)

> 官网地址：[https://redisson.org](https://redisson.orgGitHub)
> 
> GitHub地址：[https://github.com/redisson/redisson](https://github.com/redisson/redisson)

## 基本使用

不建议引入 springboot-starter，因为可能会和 springboot 内置的 redis 整合冲突

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1695799841977-9a290f86-6aeb-48e6-bf0d-3c8e2b565940.png#averageHue=%23edf0ea&clientId=u3543a423-d590-4&from=paste&height=443&id=u01799d03&originHeight=465&originWidth=1053&originalType=binary&ratio=1.0499999523162842&rotation=0&showTitle=false&size=203487&status=done&style=none&taskId=u44a22ec2-f8f9-4700-a7e0-c86f41f456f&title=&width=1002.8571883999592)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1695799991333-e395866b-ed44-4c2d-bc42-63ceeaa5d81b.png#averageHue=%23edf0ea&clientId=u3543a423-d590-4&from=paste&height=590&id=u6678290e&originHeight=620&originWidth=1221&originalType=binary&ratio=1.0499999523162842&rotation=0&showTitle=false&size=330050&status=done&style=none&taskId=u36b08022-82e9-4637-a657-e36beb993cd&title=&width=1162.8571956660496)

## 实现原理

> 没细看

Redisson 分布式锁原理

- 可重入：利用 hash 结构记录线程 id 和重入次数
- 可重试：利用信号量和 Pubsub 功能实现等待、唤醒，获取锁失败的重试机制
- 超时续约：利用 watchDog，每隔一段时间 (releaseTime/3)，重置超时时间

获取锁 api

- `waitTime`：在等待时间内不断重试
- `leaseTime`：在锁失效时间内自动释放

![500](assets/image%20(34).png)

### 可重入

保证同一个线程可以多次获取同一把锁

通过 Redis 的哈希结构实现

- `field`：线程标识
- `value`：重入次数

![250](assets/image%20(35).png)

实现原理

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696068352563-d200e0ec-1f1c-4a04-a850-9db219a2c4e8.png#averageHue=%23eee6e5&clientId=ub26dbcfb-ae41-4&from=paste&id=u44450430&originHeight=954&originWidth=1973&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=837388&status=done&style=none&taskId=ua2b33100-988b-40a6-a7a3-79ff1461af2&title=)

获取锁的 lua 脚本

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696069064075-c469b5e3-cf6b-4524-ba82-2ede3bffaa86.png#averageHue=%23eef0ea&clientId=ub26dbcfb-ae41-4&from=paste&height=704&id=u5a1f41d2&originHeight=968&originWidth=1929&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=1093924&status=done&style=none&taskId=u4a618302-d617-49f3-b611-3921613598b&title=&width=1402.909090909091)

释放锁的 lua 脚本

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696069220422-cae40820-d15a-4b8a-8400-0278b5263851.png#averageHue=%23edefe9&clientId=ub26dbcfb-ae41-4&from=paste&height=692&id=u8a9aad92&originHeight=952&originWidth=1920&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=1050648&status=done&style=none&taskId=u1c53cb07-1486-4c95-9aad-4db85e2c83e&title=&width=1396.3636363636363)

测试可重入锁，导包 junit5

```java
@SpringBootTest
@Slf4j
public class RedissionTest {
    @Resource
    private RedissonClient redissonClient;

    private RLock lock;

    @BeforeEach
    public void setUp() {
        lock = redissonClient.getLock("order");
    }

    @Test
    public void method1() {
//        boolean isLock = lock.tryLock();
//        lock = redissonClient.getLock("order");
        boolean isLock = lock.tryLock();
        if (!isLock) {
            log.error("获取锁失败1");
            return;
        }
        try {
            log.info("获取锁成功1");
            method2();
            log.info("开始执行业务1");
        } finally {
            log.warn("准备释放锁1");
            lock.unlock();
        }
    }

    public void method2() {
//        boolean isLock = lock.tryLock();
//        lock = redissonClient.getLock("order");
        boolean isLock = lock.tryLock();
        if (!isLock) {
            log.error("获取锁失败2");
            return;
        }
        try {
            log.info("获取锁成功2");
            log.info("开始执行业务2");
        } finally {
            log.warn("准备释放锁2");
            lock.unlock();
        }
    }
}
```

2）可重试和超时续约

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696084534145-4962b023-e1c1-4819-9ba0-e66044c44e1d.png#averageHue=%23f9f6f5&clientId=ub26dbcfb-ae41-4&from=paste&height=561&id=ub8c75e81&originHeight=771&originWidth=1761&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=422189&status=done&style=none&taskId=u459238b5-e6a3-4779-bcdf-35d73ffb962&title=&width=1280.7272727272727)

## 主从一致性问题

主节点负责写操作，从结点只负责读操作，主节点挂了以后选取一个从结点作为主节点，但还没来得及同步锁

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696085628817-4fc9e071-8e49-46ef-a434-e8ad64abc645.png#averageHue=%23f6eaea&clientId=ub26dbcfb-ae41-4&from=paste&id=uee68e32d&originHeight=674&originWidth=1457&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=257197&status=done&style=none&taskId=u9b59f688-0f41-4f6c-99d1-704ef750610&title=)

解决方法：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功（可以不设置主从）

利用 Redisson的 multiLock（联锁）可以实现，联锁可以看做是多个 Redis 可重入锁的集合

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696086191239-cd102a81-44e8-444e-ada9-270dce527102.png#averageHue=%23f2dfde&clientId=ub26dbcfb-ae41-4&from=paste&height=492&id=u53b44615&originHeight=677&originWidth=1906&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=458220&status=done&style=none&taskId=u10244d9d-deb1-48b7-bc09-cc72b004d4e&title=&width=1386.1818181818182)<br />使用方法：<br />1）配置多个`RedissionClient`<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696133038035-31faf9de-bc42-4a44-9bb5-a56133553393.png#averageHue=%23eff4ed&clientId=u30c8dc24-1b16-4&from=paste&id=uf1d8f4f4&originHeight=723&originWidth=1264&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=437439&status=done&style=none&taskId=u0b26d7b7-301c-4a31-bc6d-c5814120e52&title=)<br />2）创建联锁<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696133225965-ff187be5-e941-4461-bcf6-f89789d98e3e.png#averageHue=%23eff4ee&clientId=u30c8dc24-1b16-4&from=paste&height=370&id=u860fdc15&originHeight=611&originWidth=1021&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=378606&status=done&style=none&taskId=ue826b52a-3f07-4cbe-b053-0408e01f9c9&title=&width=618.7878430229047)<br />3）获取锁和释放锁的方式和之前一样

## 总结
> 全都用lua脚本保证原子性

1）不可重入Redis分布式锁<br />原理：利用setnx的互斥性；利用ex避免死锁；释放锁时判断线程标示<br />缺陷：不可重入、无法重试、锁超时失效<br />2）可重入的Redis分布式锁<br />原理：利用hash结构，记录线程标示和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试等待<br />缺陷：redis宕机引起锁失效问题<br />3）Redisson的multiLock<br />原理：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功<br />缺陷：运维成本高、实现复杂

# Redis 优化秒杀（消息队列）

用户响应速度不够

优化思路：

1. 串行改并行：原本由 1 个线程的操作改为由 2 个或多个线程同时操作，比如 1 个线程负责判断秒杀资格，1 个线程负责减库存 + 创建订单（写）
2. 同步改异步：判断完秒杀资格后，就可以返回订单 id 给前端；其余的写库操作可以异步执行。
3. 提高判断秒杀资格的性能：读 DB 改为读 Redis

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696142608378-b70532e1-6708-4ef4-93a3-e1c1c356ebf1.png#averageHue=%23f9f5f5&clientId=u30c8dc24-1b16-4&from=paste&height=527&id=uba0ca84a&originHeight=870&originWidth=1811&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=469064&status=done&style=none&taskId=u6dfc43b7-6cfc-46a3-b1ee-6012fdc0f4e&title=&width=1097.575694137591)

优化后的流程<br />1）将秒杀券库存信息提前存入Redis，用set记录用户是否已下单<br />2）基于Lua脚本，判断秒杀库存、一人一单。仅在Redis中实现用户资格判断<br />3）确认有秒杀资格后，将订单等信息传递给阻塞队列，单个独立线程串行从队列中取出信息并异步下单<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696143250708-38fb4f4d-0790-48d6-87ba-c9ad4a615872.png#averageHue=%23f1eded&clientId=u30c8dc24-1b16-4&from=paste&height=548&id=u6c309be3&originHeight=905&originWidth=1898&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=475734&status=done&style=none&taskId=u13f363a0-2817-40cc-8701-eae0b9c999d&title=&width=1150.3029638173095)
> 具体实现没有细看

阻塞队列可以用 JDK 原生的 BlockingQueue 实现，记得指定队列容量。

## Redis 消息队列
消息队列 (Message Queue)，字面意思就是存放消息的队列。最简单的消息队列模型包括3个角色

1. 消息队列：存储和管理消息，也被称为消息代理(Message Broker)
2. 生产者：发送消息到消息队列
3. 消费者：从消息队列获取消息并处理消息
> 可以看做是一个快递箱


JDK 阻塞队列可能存在哪些问题？

1. 服务器宕机，内存队列中的订单信息全部丢失
2. 线程处理错误，已取出单个订单信息，但没有入库
3. 受单 JVM 内存限制

Redis消息队列的优势

1. 独立于jvm，不受jvm内存限制
2. 消息队列会做持久化，避免服务器宕机数据丢失
3. 消息投递后需要消费者确认，重复投递知道确认位置

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696155119959-b167ddaa-9628-4044-a94c-6a2fac26f4ca.png#averageHue=%23f7f5f1&clientId=u30c8dc24-1b16-4&from=paste&height=345&id=u70e92a57&originHeight=474&originWidth=1478&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=194878&status=done&style=none&taskId=u703fa789-5cec-4ee2-a6ec-81004c8a2ef&title=&width=1074.909090909091)

Redis提供了三种不同的方式来实现消息队列

1. list结构：基于List结构模拟消息队列
2. Pubsub：基本的点对点消息模型
3. stream：比较完善的消息队列模型

1）基于list实现的消息队列<br />Redis的list数据结构是一个双向链表，很容易模拟出队列效果。<br />队列是入口和出口不在一边，因此我们可以利用:LPUSH结合RPOP、或者RPUSH结合LPOP来实现不过要注意的是，当队列中没有消息时RPOP或LPOP操作会返回null，并不像VM的阻塞队列那样会阻塞并等待消息，因此这里应该使用BRPOP或者BLPOP来实现阻塞效果。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696164574872-3d70e494-b072-4363-89ae-bd6a9cd6e5b6.png#averageHue=%23abc378&clientId=u30c8dc24-1b16-4&from=paste&height=110&id=u1e81c1e8&originHeight=151&originWidth=1573&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=120643&status=done&style=none&taskId=u825c103a-bd32-4571-a67c-3a299e07d93&title=&width=1144)<br />优点:

- 利用Redis存储，不受限于JVM内存上限
- 基于Redis的持久化机制，数据安全性有保证可以满足消息有序性

缺点:

- 无法避免消息丢失。服务取出消息后挂了，POP会直接移除消息，其他消费者也拿不到消息
- 只支持单消费者。无法实现一条消息被多个消费者消费

2）基于Pubsub实现的消息队列<br />Pubsub(发布订阅)是Redis2.0版本引入的消息传递模型。顾名思义，消费者可以订阅一个或多个channel，生产老向对应channel发送消息后，所有订阅者都能收到相关消息

- SUBSCRIBE channelchannell：订阅一个或多个频道
- PUBLISH channel msg：向一个频道发送消息
- PSUBSCRIBE pattern[pattern]：订阅与pattern格式匹配的所有频道

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696165776585-382acb55-2db7-48cf-8e63-64cf755417ba.png#averageHue=%23e9e6e6&clientId=u30c8dc24-1b16-4&from=paste&height=367&id=u11ef9f6c&originHeight=504&originWidth=1604&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=185611&status=done&style=none&taskId=u961c5a12-0d8b-48d6-a215-11b59f8aafd&title=&width=1166.5454545454545)

优点:

- 采用发布订阅模型，支持多生产、多消费

缺点:

- 不支持数据持久化。
- 无法避免消息丢失。如果发出的消息没有被订阅，直接就丢失了
- 消息堆积有上限，超出时数据丢失

### 基于Stream的消息队列
Stream是Redis5.0引入的一种新数据类型，可以实现一个功能非常完善的消息队列。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696166864838-cd4e2688-e8aa-442c-af28-43e81ce17175.png#averageHue=%23627e75&clientId=ue1bd0761-17c2-4&from=paste&height=473&id=u877f42c7&originHeight=650&originWidth=1828&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=681330&status=done&style=none&taskId=ua4e452ed-2fb0-466c-8fd2-ae0fbca4566&title=&width=1329.4545454545455)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696167310411-402a506a-e26f-476f-9be9-b28cb1b35a80.png#averageHue=%23526b74&clientId=ue1bd0761-17c2-4&from=paste&height=584&id=uf39751b5&originHeight=803&originWidth=1749&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=916739&status=done&style=none&taskId=u4e0d460f-89b5-4051-b2dd-8bb6111a5af&title=&width=1272)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696167437895-0cb37d7d-4b12-428a-806b-53ddfdb050c4.png#averageHue=%23d6dfd6&clientId=ue1bd0761-17c2-4&from=paste&height=564&id=u66c86067&originHeight=776&originWidth=1736&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=507418&status=done&style=none&taskId=uc807b116-709a-4d3c-a7bc-583c7807222&title=&width=1262.5454545454545)<br />STREAM类型消息队列的XREAD命令特点

- 消息可回溯
- 一个消息可以被多个消费者读取
- 可以阻塞读取
- 有消息漏读的风险

**消费者组(Consumer Group)**<br />将多个消费者划分到一个组中，监听同一个队列。具备下列特点:

1. 队列中的消息会分流给组内的不同消费者，而不是重复消费，从而加快消息处理的速度（同一个组内消费者处于竞争关系，抢消息）
2. 消费者组会维护一个标示，记录最后一个被处理的消息哪怕消费者宕机重启，还会从标示之后读取消息。**确保每一个消息都会被消费，按顺序消费**
3. 消费者获取消息后，消息处于pending状态，并存入一个pending-list。当处理完成后需要通过XACK来确认消息，标记消息为已处理，才会从pendingList移除。

> 跳过，太枯燥，而且用不到


# 达人探店（SortedSet）

## 发布探店笔记

探店笔记类似点评网站的评价，往往是图文结合。对应的表有两个

1. tb_blog：探店笔记表，包含笔记中的标题、文字、图片等
2. tb_blog_comments：其他用户对探店笔记的评价

已实现发布、查看

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696171885329-cf3ecb10-7cbc-4344-8d73-88dfee523a63.png#averageHue=%23f3efed&clientId=ue1bd0761-17c2-4&from=paste&height=676&id=ua5eeb62f&originHeight=930&originWidth=1847&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=933515&status=done&style=none&taskId=uc9f64c32-f552-4657-8b2b-5bd1209b053&title=&width=1343.2727272727273)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696171867484-0761df93-0b29-4838-9d58-61b61dcc5aab.png#averageHue=%23eae4e1&clientId=ue1bd0761-17c2-4&from=paste&height=655&id=u1934b9d7&originHeight=900&originWidth=1643&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=861951&status=done&style=none&taskId=u51e3c9d2-defc-4668-9aaf-0267a9c3855&title=&width=1194.909090909091)

## 点赞

> blog：探店笔记、liked：点赞数

完善 blog 点赞功能：

- 显示点赞数
- 一个用户只能点赞一次 ，当前用户 点赞/取消点赞

点赞功能实现步骤：

- 利用 Redis 的 **set 集合**判断 是否点赞过，
	- db：未点赞 liked+1，已点赞 liked-1
	- kv 设计：k=blogId，v 存放 userId
	- `SISMEMBER k v` 判断 userId 是否在集合里存在
		- `RedisTemplate.opsForSet().isMember(k,v)`

- Blog 类添加一个 isLike 字段，标识是否被当前用户点赞
	- id 查询 blog，判断当前登录用户是否点赞过，赋值给 isLike 字段
	- 修改分页查询 Blog 业务，判断当前登录用户是否点赞过，赋值给 isLike 字段

---
添加字段

```java
/**
 * 是否点赞过了
 */
@TableField(exist = false)
private Boolean isLike;
```

点赞实现

```java
/**
 * 点赞
 */
@Override
public Result likeBlog(Long blogId) {
    // Set，判断当前用户是否点赞过
    Long userId = UserHolder.getUser().getId();
    String key = BLOG_LIKED_KEY + blogId;
    Boolean isMember = stringRedisTemplate.opsForSet().isMember(key, userId.toString());
    if (BooleanUtil.isFalse(isMember)) {
        // 没点赞，liked+1
        boolean isSuccess = update().setSql("liked = liked + 1").eq("id", blogId).update();
        // Set添加userId
        if (isSuccess) {
            stringRedisTemplate.opsForSet().add(key, userId.toString());
        }
    } else {
        // 点赞了，liked-1
        update().setSql("liked = liked - 1").eq("id", blogId).update();
        // Set移除userId
        stringRedisTemplate.opsForSet().remove(key, userId.toString());
    }
    return Result.ok();
}
```

## 点赞排行榜

需求：按照点赞时间先后返回Top5的用户

![600](assets/image%20(32).png)

选择一种合适的数据结构：能排序、查找速度快、可去重 ——> SortedSet

![600](assets/image%20(33).png)

SortedSet 命令：  

![image.png|500](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696240961626-4e85cbe3-b5cd-442f-aa77-dfed46e14629.png#averageHue=%23082236&clientId=ua5f5a85b-6dd0-4&from=paste&height=272&id=u0e9f6ceb&originHeight=329&originWidth=599&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=135509&status=done&style=none&taskId=u390dc1f9-7d34-4b78-977f-0aec6c99534&title=&width=495.0413067071676)

实现思路：

1）当前用户 点赞/取消点赞  实现

每个 blog 点赞的用户改为存入 SortedSet，并将**时间戳作为 score 排序**

- `ZSCORE k v`：查询集合里是否存在用户，**存在返回 score**
	- 存在：`zadd k score v` 添加 userId 到集合中
	- 不存在：`DEL k`

```java
/**  
 * 当前用户 点赞/取消点赞  
 */  
@Override  
public Result likeBlog(Long blogId) {  
    Long userId = UserHolder.getUser().getId();  
    String key = BLOG_LIKED_KEY + blogId;  
    // zscore k，查询 ZSET 里是否存在 userId    Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());  
    if (score == null) {  
        // 没点赞点赞  
        boolean isSuccess = update().setSql("liked = liked + 1").eq("id", blogId).update();  
        if (isSuccess) {  
            // zadd k v score，添加 userId 到 ZSET 中，score为时间戳  
            stringRedisTemplate.opsForZSet()  
                    .add(key, userId.toString(), System.currentTimeMillis());  
        }  
    } else {  
        // 点赞了取消  
        update().setSql("liked = liked - 1").eq("id", blogId).update();  
        // zset 集合移除 userId        stringRedisTemplate.opsForZSet().remove(key, userId.toString());  
    }  
    return Result.ok();  
}
```

2）点赞排行榜 实现

`ZRANGE k start end`：获取 top5 的 userId

```java
/**  
 * 点赞排行榜：获取最先点赞的top5用户  
 */  
@Override  
public Result queryBlogLikes(Long blogId) {  
    String key = BLOG_LIKED_KEY + blogId;  
    // ZRANGE k start end，获取top5  
    Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);  
    if (top5 == null || top5.isEmpty()) {  
        return Result.ok(Collections.emptyList());  
    }  
    // userId，String-->Long  
    List<Long> ids = top5.stream().map(Long::valueOf).collect(Collectors.toList());  
    String strIds = StrUtil.join(",", ids);  
    // db 查询用户信息  
    List<UserDTO> userDTOS = userService  
            // 注意：mysql的in不保证有序性，order by field(id, id1, id2 ...) 保证有序  
            .query().in("id", ids).last("ORDER BY FIELD(id," + strIds + ")").list()  
            .stream()  
            .map(user -> BeanUtil.copyProperties(user, UserDTO.class))  
            .collect(Collectors.toList());  
  
    return Result.ok(userDTOS);  
}
```

# 好友关注（SortedSet）

## 关注和取关

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696252400255-99b8c0b5-6087-45b4-9aac-3150f2e631c6.png#averageHue=%23e6e1dd&clientId=ucf2f1bc2-449d-4&from=paste&height=856&id=u2519c825&originHeight=1036&originWidth=1830&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=1337883&status=done&style=none&taskId=u1735487e-fd96-453f-ad9a-b2b31a927c1&title=&width=1512.3966465344186)

关注是 User 之间多对多的关系，需要一张表 tb_follow 来维护关注关系

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696252948704-d91d1bc2-bb36-4842-89af-081578df9302.png#averageHue=%23eeeeeb&clientId=ucf2f1bc2-449d-4&from=paste&height=202&id=u33fdfc7f&originHeight=245&originWidth=1162&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=261918&status=done&style=none&taskId=u3973fa14-5c42-4cfb-9333-d51d20feac7&title=&width=960.3305482366089)

关注和取关实现

```java
/**
 * 关注和取关
 */
@Override
public Result follow(Long followUserId, Boolean isFollow) {
    // 获取登录用户
    Long userId = UserHolder.getUser().getId();
    if (isFollow) {
	    // 关注，新增关注数据
        Follow follow = new Follow();
        follow.setUserId(userId);
        follow.setFollowUserId(followUserId);
        save(follow);
    } else {
        // 取关，删除关注数据
        remove(new QueryWrapper<Follow>()
                .eq("user_id", userId)
                .eq("follow_user_id", followUserId));
    }
    return Result.ok();
}
```

## 共同关注

![600](assets/image%20(36).png)

实现思路：

- 关注用户以后，把 关注用户 id 存入 redis 的 Set 集合
- 借助 Set 集合**求交集**功能，求共同关注
	- `SINTER k1 k2 ...`

查找共同关注实现

```java
/**
 * 查找共同关注
 */
@Override
public Result followCommon(Long userId2) {
	// 获取当前user
	Long userId = UserHolder.getUser().getId();
	String key = "follows:" + userId;
	String key2 = "follows:" + userId2;
	// SINTER k1 k2 ...，求关注交集
	Set<String> intersectIds = stringRedisTemplate.opsForSet()
		.intersect(key, key2);
	if (intersectIds == null || intersectIds.isEmpty()) {
		// 无交集
		return Result.ok(Collections.emptyList());
	}
	// 封装获取共同关注用户信息
	List<Long> ids = intersectIds.stream().map(Long::valueOf).collect(Collectors.toList());
	List<UserDTO> userDTOS = userService.listByIds(ids).stream()
			.map(user -> BeanUtil.copyProperties(user, UserDTO.class))
			.collect(Collectors.toList());

	return Result.ok(userDTOS);
}
```

> 取关以后要从 Redis 里删除

## 关注推送

### feed 流

![](assets/image%20(37).png)

Feed 流产品有两种常见模式：

1）Timeline

不做内容筛选，按照内容**发布时间排序**

- 常用于好友或关注（例如朋友圈）
- 优点：信息全面，不会有缺失。并且实现也相对简单
- 缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低

2）智能排序

利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户

- 优点：投喂用户感兴趣信息，用户粘度很高，容易沉迷
- 缺点：如果算法不精准，可能起到反作用

### Timeline 模式

本例中的个人页面，是基于关注的好友来做 Feed 流，因此采用 **Timeline** 的模式。该模式的实现方案有三种：

1）拉模式（也叫读扩散）

- 缺点：读取+排序 耗时时间久

![](assets/image%20(38).png)

2）推模式（也叫写扩散）

发消息以后，每个粉丝都会收到消息

- 缺点：粉丝很多的情况下特别耗费空间

![500](assets/image%20(39).png)

3）推拉结合（也叫读写混合）

- 普通人（粉丝少）发消息，直接采用推模式发送到收件箱
- 大V（粉丝多）发消息，对于活跃粉丝采用推模式，普通粉丝采用拉模式

![](assets/image%20(40).png)

三种方案的对比

![](assets/image%20(41).png)

### 拉模式实现

![600](assets/image%20(42).png)

> 微博、抖音、pyq 都有类似的功能，关注页面下拉刷新

基于“推模式”实现关注推送功能

- 1）收件箱用 Redis 的 **SortedSet** 实现
	- kv 设计： key 为前缀+userId，value 存放 blogId，**score 为时间戳**

- 2）保存 blog 到 db 的同时，推送到粉丝的收件箱

- 3）查询收件箱数据时，可以实现分页查询，滚动分页

Feed 流的分页问题：角标在不断变化

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696303340160-ba46df9a-1e15-4c88-9b7e-f72f1899113b.png#averageHue=%23f7f6f5&clientId=uaa1f8912-4175-4&from=paste&height=337&id=ue0e156f0&originHeight=464&originWidth=890&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=127198&status=done&style=none&taskId=u25b01fed-54c0-423f-b579-76a14700d81&title=&width=647.2727272727273)

滚动分页

lastId 记录每次**当前页查询到的最后一条数据的时间戳**（类似游标）。查询下一页时，从当前时间戳的下一条开始查询即可

查询指令：`zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]`

- rev：从大到小排序，时间从近到远
- max：score 最大值，<=，每次取上次查询的 score 最小值
- min：score 最小值（不变，始终是0）
- offset：起始角标，取在上一次的结果中**与最小值一样**的元素个数
- count：查询数量
- WITHSCORES：显示分数值

![](assets/image%20(43).png)

关注推送实现

```java
/**
 * 查询关注列表里，关注用户的最新博文，下拉（滚动）刷新
 *
 * @param max    游标，最大的时间戳
 * @param offset 偏移量，从第几条开始查
 */
@Override
public Result queryBlogOfFollow(long max, Integer offset) {
    // 获取当前用户
    Long userId = UserHolder.getUser().getId();
    // zset 查询收件箱
    String key = FEED_KEY + userId;
    Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet()
	    .reverseRangeByScoreWithScores(key, 0, max, offset, 2);
    if (typedTuples == null || typedTuples.isEmpty()) {
        return Result.ok();
    }
    // 解析收件箱数据
    // zset={5,5,5,5,3,3,2}
    // 第一次：max=6, offset=0, res={5,5}, offset=2,
    // 第二次：
    long minTime = 0;
    int os = 1;
    List<Long> blogIds = new ArrayList<>(typedTuples.size());
    for (ZSetOperations.TypedTuple<String> typedTuple : typedTuples) {
        // 获取分数（时间戳）
        long time = typedTuple.getScore().longValue();
        // 获取blogId
        blogIds.add(Long.valueOf(typedTuple.getValue()));
        // 获取最小时间戳、偏移量
        if (time == minTime) {
            os++;
        } else {
            minTime = time;
            os = 1;
        }
    }
    // db 查询blog
    String strIds = StrUtil.join(",", blogIds);
    List<Blog> blogs = query().in("id", blogIds).last("ORDER BY FIELD(id," + strIds + ")").list();
    // db 查询blog的用户信息，被点赞信息
    for (Blog blog : blogs) {
        queryBlogUser(blog);
        isBlogLiked(blog);
    }

    ScrollResult r = new ScrollResult();
    r.setList(blogs);
    r.setOffset(os);
    r.setMinTime(minTime);
    return Result.ok(r);
}
```

# 附近商铺（GEO）

## GEO

GEO 就是 Geolocation 的简写形式，代表地理坐标。Redis 在 3.2 版本中加入了对 GEO 的支持，允许存储地理坐标信息，帮助我们根据经纬度来检索数据。常见的命令有:

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696663098878-c7bbaacc-108c-444a-9df7-4511ebb53508.png#averageHue=%23ececec&clientId=uea7f3b2e-47a7-4&from=paste&height=264&id=udcb5af30&originHeight=320&originWidth=1189&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=178726&status=done&style=none&taskId=u7859da94-2a4a-49fc-98a9-39c450c6e3b&title=&width=982.6445971199037)

相关命令

```bash
# 添加一个地理位置坐标，[精度 纬度 位置名称]
GEOADD key longitude latitude member [longitude latitude member ...]

# 返回指定member的坐标
GEOPOS key member [member ...]

# 返回两个member间的距离，可选单位
GEODIST key member1 member2 [m|km|ft|mi]

# 指定圆心坐标、半径，返回给定点范围内的member
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]

# 返回member的hash字符串
GEOHASH key member [member ...]
```

## 附近商户搜索

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664423482-812c6f96-c0a8-4910-bd0f-6de855fe35d1.png#averageHue=%23bda18c&clientId=uea7f3b2e-47a7-4&from=paste&height=448&id=ucf00cbc7&originHeight=542&originWidth=1209&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=500754&status=done&style=none&taskId=ubd495c47-1749-490b-a524-0de9a6067d9&title=&width=999.1735222186404)

实现流程

1）导入商铺信息到 GEO

按照商户类型做分组，类型相同的商户作为同一组，以 typeld 为 key 存入同一个 GEO 集合中即可

![400](assets/image%20(44).png)

```java
/**
 * 将店铺地理坐标存入Redis
 */
@Test
public void loadShopData() {
	List<Shop> shopList = shopService.list();
	// 店铺按照typeId分组
	Map<Long, List<Shop>> shopMap = shopList.stream()
		.collect(Collectors.groupingBy(Shop::getTypeId));
	// 按照typeId，分批写入Redis
	for (Map.Entry<Long, List<Shop>> entry : shopMap.entrySet()) {
		Long typeId = entry.getKey();
		List<Shop> shops = entry.getValue();
		// 获取商铺的地理坐标
		List<RedisGeoCommands.GeoLocation<String>> locations = new ArrayList<>(shops.size());
		for (Shop shop : shops) {
			locations.add(new RedisGeoCommands.GeoLocation<String>(
					// member
					shop.getId().toString(),
					// 坐标
					new Point(shop.getX(), shop.getY())
			));
		}
		String key = "shop:geo:" + typeId;
		stringRedisTemplate.opsForGeo().add(key, locations);
	}
}
```
 
 2）根据坐标查询附近商铺 `GEOSEARCH` 查出坐标 `(x,y)` 附近的商铺
 
```java
GEOSEARCH key [FROMMEMBER member] [FROMLONLAT longitude latitude] [BYRADIUS radius m|km|ft|mi] [BYBOX width height m|km|ft|mi] [ASC|DESC] [COUNT count [ANY]] [WITHCOORD] [WITHDIST] [WITHHASH]
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696669325983-170ef964-0f11-425a-80b0-0d4d2c8f4769.png#averageHue=%23eaece8&clientId=uea7f3b2e-47a7-4&from=paste&height=608&id=ud504ccb6&originHeight=736&originWidth=1602&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=846125&status=done&style=none&taskId=u26f11a01-a715-4d79-b8d1-6afa7f0c169&title=&width=1323.966900408819)

# 用户签到（BitMap）

## BitMap

![600](assets/image%20(47).png)

Redis 中利用 String 类型实现 BitMap

![](assets/image%20(45).png)

操作 String 的指令

![500](assets/image%20(46).png)

## 签到功能

![600](assets/image%20(48).png)

实现流程：

- 获取用户签到的年月日
- 存入 Redis 的 BitMap 结构 
	- `SETBIT key offset value`
		- key：前缀+userId+年月
		- offset：日
		- value：1

业务实现

```java
/**
 * 用户签到
 */
@Override
public Result sign() {
    // 获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    // 获取日期
    LocalDateTime now = LocalDateTime.now();
    // 拼接key：前缀+userId+年月
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 获取今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();
    // 写入Redis：SETBIT key offset 1
    stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
    return Result.ok();
}
```

## 月签到统计

![500](assets/image%20(49).png)

1）*连续签到天数*：从最后一次签到开始向前统计，直到遇到**第一次未签到**为止，计算总的签到次数，就是连续签到天数

2）如何得到本月到今天的所有签到数据？

`BITFIELD key [GET type offset]`：操作 (type：查询、修改、自增) BitMap 中 bit 数组中指定位置（offset）的值

第一种循环获取 redis key 中的偏移值，但是这种写法看着确实不太优雅....

```java
long offset = 0;
for (int i = 0; i < 10; i++) {
    redisTemplate.opsForValue().getBit("key", offset);
}
```

spring-boot-starter-data-redis 早就考虑到了这一点，所以为我们提供了一种批量执行命令的方式。我们需要使用 `BitFieldSubCommands`

3）如何获取连续签到天数？

- Redis 返回的是 10 进制数
- 与 1 做与运算可以得到最后一个 bit 位（最后一次签到）
- 循环：右移，直到某一天与运算结果为 0 停止（第一次未签到）

```java
/**
 * 统计签到天数
 * @return
 */
@Override
public Result signCount() {
	// 1 获取当前登录用户
	Long userId = UserHolder.getUser().getId();
	// 2 获取日期
	LocalDateTime now = LocalDateTime.now();
	// 3 拼接key：前缀+userId+年月
	String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
	String key = USER_SIGN_KEY + userId + keySuffix;
	// 4 获取今天是本月的第几天
	int dayOfMonth = now.getDayOfMonth();
	// 5 获取本月截止到今天的所有签到记录，返回的是一个10进制数字
	// BITFIELD sign:5:202203 GET u14
	List<Long> result = stringRedisTemplate.opsForValue().bitField(
			key,
			BitFieldSubCommands.create() // 这里采用批量执行命令的方式
					.get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth)) // 无符号、截止到dayOfMonth
					.valueAt(0) // 从角标0开始
	);
	if (result == null || result.isEmpty()) {
		// 没有签到结果
		return Result.ok(0);
	}
	Long num = result.get(0);
	if (num == null || num == 0) {
		// 十进制0代表没签到
		return Result.ok(0);
	}
	// 6 获取连续签到天数
	int count = 0;
	while (true) {
		// 和1做与运算，得到数字最后一个bit位
		if ((num & 1) == 0) {
			// 最后一位为0，改天未签到
			break;
		} else {
			count++;
		}
		// 右移，到前一天
		num >>>= 1;
	}
	return Result.ok(count);
}
```

# UV 统计（HyperLogLog）

- UV：全称 Unique Visitor，也叫独立访客量，是指通过互联网访问、浏览这个网页的自然人。**1 天内同一个用户多次问该网站，只记录 1 次。**
- PV：全称 Page View，也叫页面访问量或点击量，用户每访问网站的一个页面，记录 1 次 PV，用户多次打开页面，则记录多次 PV。往往用来衡量网站的流量。

UV 统计在服务端做会比较麻烦，因为要判断该用户是否已经统计过了，需要将统计过的用户信息保存。但是如果每个访问的用户都保存到 Redis 中，数据量会非常恐怖。

## HyperLogLog

> 相关算法原理可以参考：[HyperLogLog 算法的原理讲解以及 Redis 是如何应用它的 - 掘金](https://juejin.cn/post/6844903785744056333#heading-0Redis)

Hyperloglog (HLL) 是从 Loglog 算法派生的概率算法，用于确定非常大的集合的基数（元素个数），且不需要存储元素本身的全部，所以 HLL 不能像集合那样，返回输入的各个元素。

- Redis 中的 HLL 是基于 **string** 结构实现的
- **单个 HLL 的内存永远小于 16kb**，内存占用低的令人发指！
- 作为代价，**其测量结果是概率性的**，有小于 0.81%的误差。不过对于 UV 统计来说，这完全可以忽略。

常见指令：

![image.png|650](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696769581371-07f02f41-0160-4267-ab7d-e767102d9299.png#averageHue=%230b2236&clientId=ufa1e98a6-3214-4&from=paste&height=231&id=u2a6193b7&originHeight=317&originWidth=939&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=229402&status=done&style=none&taskId=u5352be5c-a548-41a2-86db-cff1807595a&title=&width=682.9090909090909)

重复元素只记录一次

![image.png|400](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696769665540-cc485f2b-574f-45f4-b467-b0eaac304a49.png#averageHue=%23102839&clientId=ufa1e98a6-3214-4&from=paste&height=151&id=u0440561c&originHeight=207&originWidth=504&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=115575&status=done&style=none&taskId=u455154a0-0b12-4820-8a02-32e0181f197&title=&width=366.54545454545456)

## UV 统计实现

向 HyperLogLog 中添加 100 万条数据，看内存占用和统计效果

```java
 @Test
public void testHyperLogLog() {
	String[] users = new String[1000];
	// 数组角标
	int index = 0;
	for (int i = 1; i <= 1000000; i++) {
		users[index++] = "user_" + i;
		if (i % 1000 == 0) {
			index = 0;
			stringRedisTemplate.opsForHyperLogLog().add("hll1", users);
		}
	}
	Long size = stringRedisTemplate.opsForHyperLogLog().size("hll1");
	System.out.println("size=" + size);
}
```

查看误差：打印 size=997593，和 100 万差距很小

内存占用前后对比，`info memory` 查看内存占用情况，只占用了 14KB

```
used_memory:1527480
used_memory_human:1.46M

used_memory:1541896
used_memory_human:1.47M
```

`memory usage key` 查看字节数大小 --> 14384B

# -------------------- 高级篇

单点Redis的问题：

1. 数据丢失问题：Redis是内存存储，服务重启可能会丢失数据
   1. 实现数据持久化
2. 并发能力问题：单节点Redis并发能力虽然不错，但也无法满足如618这样的高并发场景
   1. 搭建主从集群，实现读写分离
3. 故障恢复问题：如果Redis宕机，则服务不可用，需要一种自动的故障恢复手段
   1. 利用Redis哨兵，实现健康监测和自动回复
4. 存储能力问题：Redis基于内存，单节点能存储的数据量难以满足海量数据需求
   1. 搭建分片集群，利用插槽机制实现动态扩容

# Redis 持久化

## RDB 持久化

RDB 全称 *Redis Database Backup file* (Redis 数据备份文件)，也被叫做 Redis 数据快照。简单来说就是**把内存中的所有数据都记录到磁盘中**。当 Redis 实例故障重启后，从磁盘读取快照文件，恢复数据

### 使用 & 配置

修改 redis.conf 文件，将其中的持久化模式改为默认的 RDB 模式，AOF 保持关闭状态。

```properties
# 开启RDB
# save ""
save 3600 1
save 300 100
save 60 10000

# 关闭AOF
appendonly no
```

---

快照文件称为 RDB 文件，默认是保存在当前运行目录

两种执行 RDB 的方式

- `save` 命令：由 Redis 主进程来执行 RDB，会阻塞所有命令
- `bgsave` 命令：开启子进程执行 RDB，避免主进程收到影响

Redis **停机时会自动执行一次 RDB**，但如果遇到宕机的情况，会造成数据丢失

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696819735458-de82b368-76d7-4558-bc16-7a28adbf5474.png#averageHue=%2366685c&clientId=u09d4932f-1b94-4&from=paste&height=93&id=uf279e0c7&originHeight=128&originWidth=953&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=165813&status=done&style=none&taskId=u90408c3e-df8d-4f31-b4e0-22ab4f48d5d&title=&width=693.0909090909091)

Redis 内部有**触发 RDB 的机制**，在 Redis.conf 文件中配置

```bash
# 禁用rdb
# save ""

# 900s内，如果至少有一个key被修改，则执行bgsave
save 900 1
save 300 10
save 60 10000
```

其他配置

```bash
# 是否压缩，建议不开启，压缩也会消耗cpu，磁盘不值钱
rdbcompression yes

# RDB文件名称
dbfilename dump.rdb

# 文件保存路径
dir ./
```

### fork 原理

**bgsave** 开始时会 fork 主进程得到子进程，子进程共享主进程的内存数据。完成 fork 后读取内存数据并写入 RDB 文件。**异步**持久化

- 当主进程执行读操作时，访问共享内存
- 当主进程执行写操作时，则会**拷贝**一份数据，执行写操作

![](assets/image%20(50).png)

### 总结

1）RDB 方式 bgsave 的基本流程?

- fork 主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的 RDB
- 文件用新 RDB 文件替换旧的 RDB 文件

2）RDB 会在什么时候执行？save 60 1000 代表什么含义？

- 默认是服务停止时
- 代表 60 秒内至少执行 1000 次修改则触发 RDB

3）RDB 的缺点?

- RDB 执行间隔时间长（短了可能第一次 RDB 还没结束，就开始第二次 RDB 了），两次 RDB 之间写入数据有丢失的风险
- fork 子进程、压缩、写出 RDB 文件都比较耗时


## AOF

AOF 全称为 *Append 0nly File* (追加文件)。Redis 处理的每一个**写命令都会记录在 AOF 文件**，可以看做是命令日志文件

### 使用 & 配置

AOF **默认是关闭**的，需要修改 redis.conf 开启

```bash
# 是否开启AOF，默认是no
appendonly yes

# AOF文件的名称
appendfilename "appendonly.aof"
```

AOF 的命令记录的频率也可以通过 redis.conf 文件来配

```bash
# 每执行一次写命令，立即记录到AOF文件
appendfsync always

# 默认方案，写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件
appendfsync everysec

# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```

![](assets/image%20(51).png)

### AOF 文件重写

因为是记录命令，**AOF 文件会比 RDB 文件大的多**

AOF 会记录对同一个 key 的多次写操作，但只有最后一次写操作才有意义。通过执行 `bgrewriteaof` 命令，可以让 AOF 文件**执行重写功能**，用最少的命令达到相同效果

![](assets/image%20(52).png)

Redis 也会在触发阈值时自动去重写 AOF 文件，阈值可以在 redis.conf 中配置

```bash
# aof文件比上次文件，增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# aof文件触发重写的最小体积
auto-aof-rewrite-min-size 64mb
```

## RDB 和 AOF 对比

RDB 和 AOF 各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会结合两者使用

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696837784755-ac3e52ae-17f7-4d09-93d7-60ff8e8dc8a1.png#averageHue=%23cacac8&clientId=u09d4932f-1b94-4&from=paste&height=295&id=u9ae0f93a&originHeight=406&originWidth=1117&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=182295&status=done&style=none&taskId=u0ec7b542-8b57-4c2e-820f-f4b05d55b21&title=&width=812.3636363636364)

# Redis 主从

单节点 Redis 的并发能力是有上限的，要进一步提高 Redis 的并发能力，就需要搭建主从集群，实现**读写分离**

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696838433752-350bb64b-e3d8-4057-b3f1-ad4e9d7bdbe7.png#averageHue=%23f9f3f2&clientId=u09d4932f-1b94-4&from=paste&height=312&id=u29aa49e1&originHeight=429&originWidth=906&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=115993&status=done&style=none&taskId=u32a04d01-64b4-42b7-b030-829abd93c23&title=&width=658.9090909090909)

## 搭建主从架构

搭主从集群结构如图：共包含三个节点，一个主节点，两个从节点。

7001 端口作为 master，7002 和 7003 作为 slave

1）配置文件

统一使用 rdb

```properties
# 开启RDB
# save ""
save 3600 1
save 300 100
save 60 10000

# 关闭AOF
appendonly no
```

修改三个端口号为7001、7002、7003

虚拟机本身有多个 IP，为了避免将来混乱，我们需要在 redis.conf 文件中指定每一个实例的绑定 ip 信息，格式如下：

```properties
# redis实例的声明 IP
replica-announce-ip 192.168.111.154
```

2）docker搭建集群

```bash
docker run --name redis7001 -p 7001:7001 \
-v /mydata/redis/7001/conf/:/usr/local/etc/redis \
-v /mydata/redis/7001/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7002 -p 7002:7002 \
-v /mydata/redis/7002/conf/:/usr/local/etc/redis \
-v /mydata/redis/7002/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7003 -p 7003:7003 \
-v /mydata/redis/7003/conf/:/usr/local/etc/redis \
-v /mydata/redis/7003/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf
```

进入Redis客户端

```bash
docker exec -it redis7001 redis-cli -p 7001
docker exec -it redis7002 redis-cli -p 7002
docker exec -it redis7003 redis-cli -p 7003
```

3）开启主从关系

有临时和永久两种模式：

-  修改配置文件（永久生效） 
	   - 在 `redis.conf` 中添加一行配置：`slaveof <masterip> <masterport>`
-  使用 redis-cli 客户端连接到 redis 服务，执行 slaveof 命令（重启后失效）：

```properties
slaveof <masterip> <masterport>

# 7002和7003执行
SLAVEOF 192.168.111.154 7001
```

`info replication` 查看集群状态信息

> **注意**：在 5.0 以后新增命令 replicaof，与 slaveof 效果一致。

4）测试

主节点执行 `set num 666`，从节点 `get num` 都能获取到

从结点 `set num` 会失败

## 主从数据同步原理

### 全量同步

Redis 主从同步的**第一次**同步是全量同步

1. 第一阶建立连接，完成了初步沟通，保存版本信息
2. 第二阶段 master 执行 `bgsave`，生成 RDB，发送给 slave；同时记录生成 rdb 期间的所有命令到 `repl_baklog`
3. master 将 `repl_baklog` 中的命令发送给 slave

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696842806491-353d7b37-da3b-4a60-bc69-c9130c134a22.png#averageHue=%23e8edde&clientId=u09d4932f-1b94-4&from=paste&height=367&id=uab496b92&originHeight=505&originWidth=1033&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=183365&status=done&style=none&taskId=u9514190e-0fb9-4804-84f4-f01ca6eba35&title=&width=751.2727272727273)

在第一阶段涉及到两个概念

- `Replication ld`：简称 replid，是数据集的标记，
	- id 一致则说明是同一数据集
	- 每一个 master 都有唯一的 replid，slave 则会**继承** master 节点的 replid。（第一次 master 会把 replid 给 slave）

- `offset`：偏移量
	- 随着记录在 `repl_baklog` 中的数据增多而逐渐增大。
	- slave 完成同步时也会记录当前同步的 offset。
	- 如果 slave 的 offset 小于 master 的 offset，说明 slave 数据落后于 master，需要更新。

> Replication 复制

因此 slave 做数据同步，必须向 master 声明自己的 `replication id` 和 offset，master 才可以判断到底需要同步哪些数据

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696849373813-438c548b-b301-4cf4-a128-0d9a6592fc4c.png#averageHue=%23e9d4d3&clientId=u09d4932f-1b94-4&from=paste&height=148&id=ue38bcd2a&originHeight=204&originWidth=846&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=61100&status=done&style=none&taskId=u34cc37e1-3807-46cf-9703-a50ab8991ff&title=&width=615.2727272727273)

> master 如何判断 slave 是不是第一次来同步数据? 
> 
> - replid 不一样就是第一次

全量同步的流程

1. slave 节点请求增量同步
2. master 节点判断 replid，发现不一致，**拒绝**增量同步
3. master 将完整内存数据生成 RDB，发送 RDB 到 slave
4. slave 清空本地数据，加载 master 的 RDB
5. master 将 RDB 期间的命令记录在 `repl baklog`，并持续将 log 中的命令发送给 slave
6. slave 执行接收到的命令，保持与 master 之间的同步

### 增量同步

 主从第一次同步是全量同步，但如果 slave 重启后同步，则执行增量同步
 
![](assets/image%20(53).png)

`repl_baklog` 大小有上限，写满后会覆盖最早的数据。如果 slave 断开时间过久致尚未备份的数据被覆盖，则无法基于 log 做增量同步，只能再次全量同步。

 可以从以下几个方面来优化 Redis 主从集群

- 提高全量同步的性能
	- 在 master 中配置 `repl-diskless-sync yes` 启用无磁盘复制，避免全量同步时的磁盘 IO
	- Redis 单节点上的内存占用不要太大，减少 RDB 导致的过多磁盘 IO
- 减少、避免全量同步
	- 适当提高 `repl baklog` 的大小，发现 slave 宕机时尽快实现故障恢复
- 限制一个 master 上的 slave 节点数量，如果实在是太多 slave，则可以采用主-从-从链式结构，减少 master 压力

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696852511863-577e7997-57ca-488f-8d60-14d6df309fe3.png#averageHue=%23f4e7e7&clientId=u09d4932f-1b94-4&from=paste&height=223&id=uca7bf5d4&originHeight=306&originWidth=980&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=152156&status=done&style=none&taskId=u22de1312-68e8-4304-b52e-9d8b5430eb5&title=&width=712.7272727272727)

### 总结

简述全量同步和增量同步区别?

- 全量同步：master 将完整内存数据生成 RDB，发送 RDB 到 slave。后续命令则记录在 repl_baklog，逐个发送给 slave
- 增量同步：slave 提交自己的 offset 到 master，master 获取 repl baklog 中从 offset 之后的命令给 slave

什么时候执行全量同步?

- slave 节点第一次连接 master 节点时
- slave 节点断开时间太久，repl baklog 中的 offset 已经被覆盖时

什么时候执行增量同步?

- slave 节点断开又恢复，并且在 repl_baklog 中能找到 offset 时

# Redis 哨兵

slave 节点宕机恢复后可以找 master 节点同步数据，那 master 节点宕机怎么办?

## 哨兵的作用和原理

1）Redis 提供了哨兵(Sentinel)机制来实现主从集群的**自动故障恢复**。哨兵的结构和作用如下

- *监控*：Sentinel 会不断检查您的 master 和 slave 是否按预期工作
- *自动故障恢复*（故障转移）：如果 master 故障，Sentinel 会将一个 slave 提升为 master。当故障实例恢复后也以新的 master 为主
- *通知*：Sentinel 充当 Redis 客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给 Redis 的客户端

![500](assets/image%20(54).png)

2）Sentinal 如何判断一个 Redis 实例是否健康？

Sentinel 基于**心跳机制**监测服务状态，每隔 1 秒向集群的每个实例发送 **ping 命令**：

- *主观下线*：如果某 sentinel 节点发现某实例未在规定时间响应，则认为该实例主观下线
- *客观下线*（挂了）：若**超过指定数量**(quorum)的 sentinel 都认为该实例主观下线，则该实例客观下线。quorum 值最好超过 Sentinel 实例数量的一半

![500](assets/image%20(55).png)

3）一旦发现 master 故障，sentinel 需要在 salve 中选择一个作为新的 master，**选择依据**是：

- 首先会判断 slave 节点与 master 节点断开时间长短，如果超过指定值(`down-after-milliseconds*10`)则会排除该 slave 节点
- 然后判断 slave 节点的 slave-priority 值，越小优先级越高，如果是 0 则永不参与选举
- 如果 slave-prority 一样，则判断 slave 节点的 offset 值，越大说明数据越新，优先级越高
- 最后是判断 slave 节点的运行 id 大小，越小优先级越高。

4）当选中了其中一个 slave 为新的 master 后，**故障转移的步骤**如下：

- sentinel 给备选的 slave 节点发送 `slaveof no one` 命令，让该节点成为 master
- sentinel 给所有其它 slave 发送 `slaveof 192.168.150.101 7002` 命令，让这些 slave 成为新 master 的从节点，开始从新的 master 上同步数据。
- 最后，sentinel 将故障节点标记为 slave（强行修改配置文件），当故障节点恢复后会自动成为新的 master 的 slave 节点

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696855044177-04250e7d-0b0a-43e9-b0eb-ce87522a9680.png#averageHue=%23e9d8d5&clientId=u09d4932f-1b94-4&from=paste&height=281&id=uc11a2201&originHeight=387&originWidth=559&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=130605&status=done&style=none&taskId=ufe499654-b7e8-46dc-afa7-dd0b2f1f497&title=&width=406.54545454545456)

## 搭建哨兵集群

> [docker环境搭建redis sentinel哨兵集群_docker redis sentinel_抹香鲸之海的博客-CSDN博客](https://blog.csdn.net/u010797364/article/details/122561349)

1）配置文件 sentinel-27001.conf，其他两个节点同理，把 sentinel 实例的端口号改了就行（别把主节点的改了）

```properties
# sentinel实例的端口
port 27001
sentinel announce-ip 192.168.111.154
# 输出日志目录
dir "/home/redis_sentinel/log"
logfile "/home/redis_sentinel/log/27001.log"
# 指定主节点信息
#   mymaster:主节点名称，任意写
#   2:选举master时的quorum值
sentinel monitor mymaster 192.168.111.154 7001 2
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```

2）Docker 创建哨兵

```bash
docker run -p 27001:27001 --restart=always --name sentinel-27001 \
-v /mydata/redis/sentinel/sentinel-27001/conf/sentinel-27001.conf:/etc/redis/sentinel.conf \
-v /mydata/redis/sentinel/sentinel-27001/data/:/data \
-v /mydata/redis/sentinel/sentinel-27001/log/:/home/redis_sentinel/log \
-d redis:6.2.7 redis-sentinel /etc/redis/sentinel.conf

docker run -p 27002:27002 --restart=always --name sentinel-27002 \
-v /mydata/redis/sentinel/sentinel-27002/conf/sentinel-27002.conf:/etc/redis/sentinel.conf \
-v /mydata/redis/sentinel/sentinel-27002/data/:/data \
-v /mydata/redis/sentinel/sentinel-27002/log/:/home/redis_sentinel/log \
-d redis:6.2.7 redis-sentinel /etc/redis/sentinel.conf

docker run -p 27003:27003 --restart=always --name sentinel-27003 \
-v /mydata/redis/sentinel/sentinel-27003/conf/sentinel-27003.conf:/etc/redis/sentinel.conf \
-v /mydata/redis/sentinel/sentinel-27003/data/:/data \
-v /mydata/redis/sentinel/sentinel-27003/log/:/home/redis_sentinel/log \
-d redis:6.2.7 redis-sentinel /etc/redis/sentinel.conf
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696861017453-d86c4bc2-fba0-44b1-8cd7-62834bc825aa.png#averageHue=%23252422&clientId=u09d4932f-1b94-4&from=paste&height=688&id=u3c0292cc&originHeight=832&originWidth=1670&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=203005&status=done&style=none&taskId=uc1952535-f588-49b6-bfac-c752e16efbe&title=&width=1380.165245744524)

7002日志

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696864345824-72f98a2c-10ec-4381-9fd8-d7870bf62536.png#averageHue=%232d2a27&clientId=u4e88a21d-c6bf-4&from=paste&height=369&id=uaa28e3c2&originHeight=446&originWidth=914&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=66503&status=done&style=none&taskId=ue2664476-0670-4073-8dc6-801cd27a6b7&title=&width=755.3718770122724)<br />7003日志<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696864353045-2a2b5ae3-2a98-41fc-aa01-60e5cd234c81.png#averageHue=%232d2a27&clientId=u4e88a21d-c6bf-4&from=paste&height=307&id=u27fc9494&originHeight=371&originWidth=950&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=67652&status=done&style=none&taskId=u0f963ced-2302-4fc0-b377-ff56445bcbb&title=&width=785.1239421899987)

## RedisTemplate 的哨兵模式

在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须感知这种变化及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

使用流程

1）依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2）配置sentinal相关信息

```yaml
spring:
  redis:
    sentinel:
      master: mymaster
      nodes:
        - 192.168.111.154:27001
        - 192.168.111.154:27002
        - 192.168.111.154:27003
```

3）配置读写分离，在任意配置类中，这里以主启动类

```java
@SpringBootApplication
public class RedisDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisDemoApplication.class, args);
    }

    @Bean
    public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer() {
        // return new LettuceClientConfigurationBuilderCustomizer() {
        //     @Override
        //     public void customize(LettuceClientConfiguration.LettuceClientConfigurationBuilder clientConfigurationBuilder) {
        //         clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
        //     }
        // };
        return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
    }
}
```

这里的 ReadFrom 是配置 Redis 的读取策略，是一个枚举，包括下面选择:

- `MASTER`：从主节点读取
- `MASTER PREFERRED`：优先从 master 节点读取，master 不可用才读取 replica
- `REPLICA`：从 slave (replica)节点读取
- `REPLICA PREFERRED（推荐）`：优先从 slave (replica)节点读取，所有的 slave 都不可用才读取 master

# Redis 分片集群

主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决

1. 海量数据存储问题
2. 高并发写的问题

使用分片集群可以解决上述问题，分片集群特征

- 集群中有多个 master，每个 master 保存不同数据
- 每个 master 都可以有多个 slave 节点
- master 之间通过 ping 监测彼此健康状态 
- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点

![450](assets/image%20(56).png)

## 搭建分片集群

搭建的分片集群结构：3 个 master 节点，每个 master 包含一个 slave 节点

![450](assets/image%20(57).png)

| IP              | PORT | 角色     |
| --------------- | ---- | ------ |
| 192.168.111.154 | 7001 | master |
| 192.168.111.154 | 7002 | master |
| 192.168.111.154 | 7003 | master |
| 192.168.111.154 | 8001 | slave  |
| 192.168.111.154 | 8002 | slave  |
| 192.168.111.154 | 8003 | slave  |

1）配置文件，端口号要修改

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696945158986-47830297-4ddc-45cf-ad10-4c672d8b6c45.png#averageHue=%23f5f4f2&clientId=ue8397173-0857-4&from=paste&height=245&id=u907ed774&originHeight=297&originWidth=461&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=49222&status=done&style=none&taskId=u100cf27f-a62f-4475-9ad8-45014865a40&title=&width=380.9917235258836)

```properties
port 7001

# 开启集群功能（重要）
cluster-enabled yes

cluster-announce-ip 192.168.111.154
cluster-announce-port 7001
# 集群间通信端口
cluster-announce-bus-port 17001

# 集群的配置文件名称，不需要我们创建，由redis自己维护
cluster-config-file nodes.conf

# 节点心跳失败的超时时间
cluster-node-timeout 5000

# 持久化文件存放目录
# dir /tmp/6379

# 绑定地址
bind 0.0.0.0

# 让redis后台运行（千万别）
# daemonize yes

# 注册的实例ip
replica-announce-ip 192.168.111.154

# 保护模式
protected-mode no

# 数据库数量
databases 1

# 日志
logfile "redis.log"
```

2）docker 创建 Redis 节点

`-p` 开启两个端口映射，一个外网交互，一个集群间通信

```bash
docker run --name redis7001 -p 7001:7001 -p 17001:17001 \
-v /mydata/redis/sharding/7001/conf/:/usr/local/etc/redis \
-v /mydata/redis/sharding/7001/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7002 -p 7002:7002 -p 17002:17002 \
-v /mydata/redis/sharding/7002/conf/:/usr/local/etc/redis \
-v /mydata/redis/sharding/7002/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7003 -p 7003:7003 -p 17003:17003 \
-v /mydata/redis/sharding/7003/conf/:/usr/local/etc/redis \
-v /mydata/redis/sharding/7003/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7004 -p 7004:7004 -p 17004:17004 \
-v /mydata/redis/sharding/7004/conf/:/usr/local/etc/redis \
-v /mydata/redis/sharding/7004/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7005 -p 7005:7005 -p 17005:17005 \
-v /mydata/redis/sharding/7005/conf/:/usr/local/etc/redis \
-v /mydata/redis/sharding/7005/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf

docker run --name redis7006 -p 7006:7006 -p 17006:17006 \
-v /mydata/redis/sharding/7006/conf/:/usr/local/etc/redis \
-v /mydata/redis/sharding/7006/data/:/data \
-d redis:6.2.7 redis-server /usr/local/etc/redis/redis.conf
```

3）创建集群
```powershell
# 进入容器
#  docker exec -it 容器id /bin/sh，因为redis容器里没有/bin/bash
docker exec -it redis7001 /bin/sh

redis-cli --cluster create --cluster-replicas 1 192.168.111.154:7001 192.168.111.154:7002 192.168.111.154:7003 192.168.111.154:7004 192.168.111.154:7005 192.168.111.154:7006
```

- `redis-cli --cluster` 或者 `./redis-trib.rb`：代表集群操作命令
- `create`：代表是创建集群
- `--replicas 1` 或者 `--cluster-replicas 1` ：指定集群中每个 master 的副本个数为1，此时 `节点总数 ÷ (replicas + 1)` 得到的就是 master 的数量。因此节点列表中的前 n 个就是 master，其它节点都是 slave 节点，随机分配到不同 master

日志：

```bash
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.111.154:7005 to 192.168.111.154:7001
Adding replica 192.168.111.154:7006 to 192.168.111.154:7002
Adding replica 192.168.111.154:7004 to 192.168.111.154:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 4a36e8ea09638830252f8fd893f1074f25330504 192.168.111.154:7001
   slots:[0-5460] (5461 slots) master
M: 6651dc848be04e1077fb6004549a79489b4e160e 192.168.111.154:7002
   slots:[5461-10922] (5462 slots) master
M: d52926c51a321c9c888d1fb17d4f1167e5595cef 192.168.111.154:7003
   slots:[10923-16383] (5461 slots) master
S: bce0b3c7c108c35e828cb492d21c55c8336471f7 192.168.111.154:7004
   replicates 6651dc848be04e1077fb6004549a79489b4e160e
S: 125bcf993d5116579585cea096bccfeda212ed49 192.168.111.154:7005
   replicates d52926c51a321c9c888d1fb17d4f1167e5595cef
S: 89bc73e6141056cff2b161feac94637be0c40b4f 192.168.111.154:7006
   replicates 4a36e8ea09638830252f8fd893f1074f25330504
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 192.168.111.154:7001)
M: 4a36e8ea09638830252f8fd893f1074f25330504 192.168.111.154:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: d52926c51a321c9c888d1fb17d4f1167e5595cef 192.168.111.154:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 125bcf993d5116579585cea096bccfeda212ed49 192.168.111.154:7005
   slots: (0 slots) slave
   replicates d52926c51a321c9c888d1fb17d4f1167e5595cef
S: bce0b3c7c108c35e828cb492d21c55c8336471f7 192.168.111.154:7004
   slots: (0 slots) slave
   replicates 6651dc848be04e1077fb6004549a79489b4e160e
S: 89bc73e6141056cff2b161feac94637be0c40b4f 192.168.111.154:7006
   slots: (0 slots) slave
   replicates 4a36e8ea09638830252f8fd893f1074f25330504
M: 6651dc848be04e1077fb6004549a79489b4e160e 192.168.111.154:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

4）查看集群状态

随便进入一个结点 `cluster nodes`

```bash
127.0.0.1:7002> cluster nodes
6651dc848be04e1077fb6004549a79489b4e160e 192.168.111.154:7002@17002 myself,master - 0 1696948517000 2 connected 5461-10922
89bc73e6141056cff2b161feac94637be0c40b4f 192.168.111.154:7006@17006 slave 4a36e8ea09638830252f8fd893f1074f25330504 0 1696948518179 1 connected
125bcf993d5116579585cea096bccfeda212ed49 192.168.111.154:7005@17005 slave d52926c51a321c9c888d1fb17d4f1167e5595cef 0 1696948517000 3 connected
4a36e8ea09638830252f8fd893f1074f25330504 10.1.1.5:7001@17001 master,fail - 1696947232298 1696947232200 1 connected 0-5460
d52926c51a321c9c888d1fb17d4f1167e5595cef 192.168.111.154:7003@17003 master - 0 1696948516666 3 connected 10923-16383
bce0b3c7c108c35e828cb492d21c55c8336471f7 192.168.111.154:7004@17004 slave 6651dc848be04e1077fb6004549a79489b4e160e 0 1696948517170 2 connected
```

## 散列插槽

Redis 会把每一个 master 节点映射到 0~16383 共 16384 个插槽(hash slot)上，查看集群信息时就能看到:

```bash
M: 4a36e8ea09638830252f8fd893f1074f25330504 192.168.111.154:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: d52926c51a321c9c888d1fb17d4f1167e5595cef 192.168.111.154:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
M: 6651dc848be04e1077fb6004549a79489b4e160e 192.168.111.154:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
```

数据 key 不是与节点绑定，而是与插槽绑定。redis 会根据 key 的有效部分计算插槽值，分两种情况:

- key 中包含"`{}`”，且“0”中至少包含 1 个字符，“`{}`”中的部分是有效部分
- key 中不包含“`{}`”，整个 key 都是有效部分 p

例如: key 是 num，那么就根据 num 计算，如果是{itcast}num，则根据 itcast 计算。计算方式是利用 CRC16 算法得到一个 hash 值，然后对 16384 取余，得到的结果就是 slot 值。

## 集群伸缩



## 故障转移



## RedisTemplate 访问分片集群



# -------------------- 原理篇






























