单点Redis的问题：

1. 数据丢失问题：Redis是内存存储，服务重启可能会丢失数据
   1. 实现数据持久化
2. 并发能力问题：单节点Redis并发能力虽然不错，但也无法满足如618这样的高并发场景
   1. 搭建主从集群，实现读写分离
3. 故障恢复问题：如果Redis宕机，则服务不可用，需要一种自动的故障恢复手段
   1. 利用Redis哨兵，实现健康监测和自动回复
4. 存储能力问题：Redis基于内存，单节点能存储的数据量难以满足海量数据需求
   1. 搭建分片集群，利用插槽机制实现动态扩容

# Redis持久化
## RDB持久化
### 使用和配置
RDB全称Redis Database Backup file (Redis数据备份文件)，也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据

---

修改redis.conf文件，将其中的持久化模式改为默认的RDB模式，AOF保持关闭状态。
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

快照文件称为RDB文件，默认是保存在当前运行目录<br />两种执行RDB的方式<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696819323538-3a64bd9f-fdc8-4000-a60d-8272c15fe80a.png#averageHue=%230c3049&clientId=u09d4932f-1b94-4&from=paste&height=117&id=uf7712e66&originHeight=161&originWidth=741&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=107699&status=done&style=none&taskId=u21c90b49-c8da-4d2d-a2bd-4da6e9ab8e6&title=&width=538.9090909090909)

Redis停机时会自动执行一次RDB，但如果遇到宕机的情况，会造成数据丢失<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696819735458-de82b368-76d7-4558-bc16-7a28adbf5474.png#averageHue=%2366685c&clientId=u09d4932f-1b94-4&from=paste&height=93&id=uf279e0c7&originHeight=128&originWidth=953&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=165813&status=done&style=none&taskId=u90408c3e-df8d-4f31-b4e0-22ab4f48d5d&title=&width=693.0909090909091)

Redis内部有触发RDB的机制，在Redis.conf文件中配置
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

### fork原理
bgsave开始时会fork主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入RDB 文件。异步持久化

- 当主进程执行读操作时，访问共享内存
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696823584775-60473194-aba1-4fcc-bb1c-3020c8fbc0c4.png#averageHue=%23f3eceb&clientId=u09d4932f-1b94-4&from=paste&height=272&id=uff99dd9c&originHeight=374&originWidth=1052&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=211653&status=done&style=none&taskId=u05c447e6-2cd9-4ad5-b77a-7a0904a8069&title=&width=765.0909090909091)

### 总结
1）RDB方式bgsave的基本流程?

- fork主进程得到一个子进程，共享内存空间
- 子进程读取内存数据并写入新的RDB
- 文件用新RDB文件替换旧的RDB文件

2）RDB会在什么时候执行？save 60 1000代表什么含义？

- 默认是服务停止时
- 代表60秒内至少执行1000次修改则触发RDB

3）RDB的缺点?

- RDB执行间隔时间长（短了可能第一次RDB还没结束，就开始第二次RDB了），两次RDB之间写入数据有丢失的风险
- fork子进程、压缩、写出RDB文件都比较耗时


## AOF
### 使用
AOF全称为Append 0nly File (追加文件)。Redis处理的每一个写命令都会记录在AOF文件，可以看做是命令日志文件

AOF默认是关闭的，需要修改redis.conf开启
```bash
# 是否开启AOF，默认是no
appendonly yes

# AOF文件的名称
appendfilename "appendonly.aof"
```
AOF的命令记录的频率也可以通过redis.conf文件来配
```bash
# 每执行一次写命令，立即记录到AOF文件
appendfsync always

# 默认方案，写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件
appendfsync everysec

# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘
appendfsync no
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696831023594-9e322c1d-95a3-4e78-8351-651c0af6fbca.png#averageHue=%23c2a9a8&clientId=u09d4932f-1b94-4&from=paste&height=118&id=u9d9fa883&originHeight=162&originWidth=941&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=79534&status=done&style=none&taskId=u7900e969-d381-48a1-9250-68cca247dc9&title=&width=684.3636363636364)

### aof文件重写
因为是记录命令，AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但只有最后一次写操作才有意义。通过执行`**bgrewriteaof**`命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696832332515-773796b8-e850-4334-87fa-423cb29c0b67.png#averageHue=%23141729&clientId=u09d4932f-1b94-4&from=paste&height=33&id=ufb6a6729&originHeight=46&originWidth=463&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=5210&status=done&style=none&taskId=u6f501757-d978-4f57-a121-648839712c2&title=&width=336.72727272727275)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696831784030-5c4c989f-e578-4751-a302-0f903baa623e.png#averageHue=%23e2e2dc&clientId=u09d4932f-1b94-4&from=paste&height=81&id=u062cddc6&originHeight=112&originWidth=873&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=45237&status=done&style=none&taskId=uf5cf1ee0-0654-4778-8e9e-8ef467ca188&title=&width=634.9090909090909)<br />  Redis也会在触发闯值时自动去重写AOF文件。闻值也可以在redis.conf中配置
```bash
# aof文件比上次文件，增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# aof文件触发重写的最小体积
auto-aof-rewrite-min-size 64mb
```

## RDB和AOF对比
RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会结合两者使用<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696837784755-ac3e52ae-17f7-4d09-93d7-60ff8e8dc8a1.png#averageHue=%23cacac8&clientId=u09d4932f-1b94-4&from=paste&height=295&id=u9ae0f93a&originHeight=406&originWidth=1117&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=182295&status=done&style=none&taskId=u0ec7b542-8b57-4c2e-820f-f4b05d55b21&title=&width=812.3636363636364)

# Redis主从
 单节点Redis的并发能力是有上限的，要进一步提高Redis的并发能力，就需要搭建主从集群，实现读写分离<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696838433752-350bb64b-e3d8-4057-b3f1-ad4e9d7bdbe7.png#averageHue=%23f9f3f2&clientId=u09d4932f-1b94-4&from=paste&height=312&id=u29aa49e1&originHeight=429&originWidth=906&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=115993&status=done&style=none&taskId=u32a04d01-64b4-42b7-b030-829abd93c23&title=&width=658.9090909090909)
## 搭建主从架构
我们搭建的主从集群结构如图：共包含三个节点，一个主节点，两个从节点。<br />7001端口作为master，7002和7003作为slave

1）配置文件<br />统一使用rdb
```properties
# 开启RDB
# save ""
save 3600 1
save 300 100
save 60 10000

# 关闭AOF
appendonly no
```
修改三个端口号为7001、7002、7003<br />虚拟机本身有多个IP，为了避免将来混乱，我们需要在redis.conf文件中指定每一个实例的绑定ip信息，格式如下：
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

3）开启主从关系<br />有临时和永久两种模式：

-  修改配置文件（永久生效） 
   - 在redis.conf中添加一行配置：`slaveof <masterip> <masterport>`
-  使用redis-cli客户端连接到redis服务，执行slaveof命令（重启后失效）： 
```properties
slaveof <masterip> <masterport>

# 7002和7003执行
SLAVEOF 192.168.111.154 7001
```
`info replication`查看集群状态信息
> **注意**：在5.0以后新增命令replicaof，与salveof效果一致。


4）测试<br />主节点执行`set num 666`，从节点`get num`都能获取到<br />从结点`set num`会失败<br />实现了读写分离

## 主从数据同步原理
### 全量同步
Redis主从同步的第一次同步是**全量同步**

1. 第一阶建立连接，完成了初步沟通，保存版本信息
2. 第二阶段master执行`bgsave`，生成RDB，发送给slave；同时记录生成rdb期间的所有命令到`repl_baklog`
3. master将`repl_baklog`中的命令发送给slave

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696842806491-353d7b37-da3b-4a60-bc69-c9130c134a22.png#averageHue=%23e8edde&clientId=u09d4932f-1b94-4&from=paste&height=367&id=uab496b92&originHeight=505&originWidth=1033&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=183365&status=done&style=none&taskId=u9514190e-0fb9-4804-84f4-f01ca6eba35&title=&width=751.2727272727273)

在第一阶段涉及到两个概念

1. `Replication ld`: 简称replid，是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid，slave则会继承master节点的replid。（第一次master会把replid给slave）
2. `offset`：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新。  

因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据<br /> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696849373813-438c548b-b301-4cf4-a128-0d9a6592fc4c.png#averageHue=%23e9d4d3&clientId=u09d4932f-1b94-4&from=paste&height=148&id=ue38bcd2a&originHeight=204&originWidth=846&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=61100&status=done&style=none&taskId=u34cc37e1-3807-46cf-9703-a50ab8991ff&title=&width=615.2727272727273)<br />**master如何判断slave是不是第一次来同步数据? **<br />replid不一样就是第一次

**全量同步的流程**

1. slave节点请求增量同步
2. master节点判断replid，发现不一致，拒绝增量同步
3. master将完整内存数据生成RDB，发送RDB到slave
4. slave清空本地数据，加载master的RDB
5. master将RDB期间的命令记录在repl baklog，并持续将log中的命令发送给slave
6. slave执行接收到的命令，保持与master之间的同步

### 增量同步
 主从第一次同步是全量同步，但如果slave重启后同步，则执行增量同步<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696852120879-f84f2cdb-9bc3-4254-afb4-423cb5b2941a.png#averageHue=%23ede7de&clientId=u09d4932f-1b94-4&from=paste&height=263&id=u98f44995&originHeight=362&originWidth=1186&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=167520&status=done&style=none&taskId=u980f6e24-1e5d-46c9-a8a1-0dea27795b7&title=&width=862.5454545454545)<br />repl_baklog大小有上限，写满后会覆盖最早的数据。如果slave断开时间过久致尚未备份的数据被覆盖，则无法基于log做增量同步，只能再次全量同步。

 可以从以下几个方面来优化Redis主从就集群

- 提高全量同步的性能
   - 在master中配置repl-diskless-sync yes启用无磁盘复制，避免全量同步时的磁盘IO
   - Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 减少、避免全量同步
   - 适当提高repl baklog的大小，发现slave宕机时尽快实现故障恢复
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用主-从-从链式结构，减少master压力

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696852511863-577e7997-57ca-488f-8d60-14d6df309fe3.png#averageHue=%23f4e7e7&clientId=u09d4932f-1b94-4&from=paste&height=223&id=uca7bf5d4&originHeight=306&originWidth=980&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=152156&status=done&style=none&taskId=u22de1312-68e8-4304-b52e-9d8b5430eb5&title=&width=712.7272727272727)<br /> 
### 总结
简述全量同步和增量同步区别?

- 全量同步：master将完整内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_baklog，逐个发送给slave
- 增量同步：slave提交自己的offset到master，master获取repl baklog中从offset之后的命令给slave

什么时候执行全量同步?

- slave节点第一次连接master节点时
- slave节点断开时间太久，repl baklog中的offset已经被覆盖时

什么时候执行增量同步?

- slave节点断开又恢复，并且在repl_baklog中能找到offset时

# Redis哨兵
slave节点宕机恢复后可以找master节点同步数据，那master节点宕机怎么办?
## 哨兵的作用和原理
1）Redis提供了哨兵(Sentinel)机制来实现主从集群的自动故障恢复。哨兵的结构和作用如下

- 监控：Sentinel 会不断检查您的master和slave是否按预期工作
- 自动故障恢复（故障转移）：如果master故障，Sentinel会将一个slave提升为master。当故障实例恢复后也以新的master为主
- 通知：Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696854208629-2c94bf54-42db-41b7-b7ee-9337367cb7ff.png#averageHue=%23f2dcd2&clientId=u09d4932f-1b94-4&from=paste&height=332&id=ub1c32957&originHeight=456&originWidth=570&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=177771&status=done&style=none&taskId=u1c6fecec-2690-47ec-9313-ed0f3607cd8&title=&width=414.54545454545456)<br />2）Sentinal如何判断一个Redis实例是否健康？<br />Sentinel基于心跳机制监测服务状态，每隔1秒向集群的每个实例发送ping命令：

- 主观下线：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例主观下线。
- 客观下线（挂了）：若**超过指定数量(quorum)的sentinel都认为该实例主观下线**，则该实例客观下线。quorum值最好超过Sentinel实例数量的一半

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696854495226-be36cd39-31ce-49e8-9df7-2e78000d8cd9.png#averageHue=%23f1d5c9&clientId=u09d4932f-1b94-4&from=paste&height=264&id=uee3dc7c0&originHeight=363&originWidth=563&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=176693&status=done&style=none&taskId=u5d5222fb-5035-4b77-be47-e29dc367416&title=&width=409.45454545454544)

3）一旦发现master故障，sentinel需要在salve中选择一个作为新的master，**选择依据**是：

- 首先会判断slave节点与master节点断开时间长短，如果超过指定值(down-after-milliseconds*10)则会排除该slave节点
- 然后判断slave节点的slave-priority值，越小优先级越高，如果是0则永不参与选举
- 如果slave-prority一样，则判断slave节点的offset值，越大说明数据越新，优先级越高
- 最后是判断slave节点的运行id大小，越小优先级越高。

4）当选中了其中一个slave为新的master后(例如slave1)，**故障的转移的步骤**如下：

- sentinel给备选的slave节点发送`slaveof no one`命令，让该节点成为master
- sentinel给所有其它slave发送`slaveof 192.168.150.101 7002`命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
- 最后，sentinel将故障节点标记为slave（强行修改配置文件），当故障节点恢复后会自动成为新的master的slave节点

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696855044177-04250e7d-0b0a-43e9-b0eb-ce87522a9680.png#averageHue=%23e9d8d5&clientId=u09d4932f-1b94-4&from=paste&height=281&id=uc11a2201&originHeight=387&originWidth=559&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=130605&status=done&style=none&taskId=ufe499654-b7e8-46dc-afa7-dd0b2f1f497&title=&width=406.54545454545456)

## 搭建哨兵集群
[docker环境搭建redis sentinel哨兵集群_docker redis sentinel_抹香鲸之海的博客-CSDN博客](https://blog.csdn.net/u010797364/article/details/122561349)

1）配置文件sentinel-27001.conf，其他两个节点同理，把sentinel实例的端口号改了就行（别把主节点的改了）
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
2）Docker创建哨兵
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

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696861017453-d86c4bc2-fba0-44b1-8cd7-62834bc825aa.png#averageHue=%23252422&clientId=u09d4932f-1b94-4&from=paste&height=688&id=u3c0292cc&originHeight=832&originWidth=1670&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=203005&status=done&style=none&taskId=uc1952535-f588-49b6-bfac-c752e16efbe&title=&width=1380.165245744524)<br />7002日志<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696864345824-72f98a2c-10ec-4381-9fd8-d7870bf62536.png#averageHue=%232d2a27&clientId=u4e88a21d-c6bf-4&from=paste&height=369&id=uaa28e3c2&originHeight=446&originWidth=914&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=66503&status=done&style=none&taskId=ue2664476-0670-4073-8dc6-801cd27a6b7&title=&width=755.3718770122724)<br />7003日志<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696864353045-2a2b5ae3-2a98-41fc-aa01-60e5cd234c81.png#averageHue=%232d2a27&clientId=u4e88a21d-c6bf-4&from=paste&height=307&id=u27fc9494&originHeight=371&originWidth=950&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=67652&status=done&style=none&taskId=u0f963ced-2302-4fc0-b377-ff56445bcbb&title=&width=785.1239421899987)

## RedisTemplate的哨兵模式
在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须感知这种变化及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

使用流程<br />1）依赖
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
这里的ReadFrom是配置Redis的读取策略，是一个枚举，包括下面选择:

- `MASTER`：从主节点读取
- `MASTER PREFERRED`：优先从master节点读取，master不可用才读取replica
- `REPLICA`：从slave (replica)节点读取
- `REPLICA PREFERRED（推荐）`：优先从slave (replica)节点读取，所有的slave都不可用才读取master

# Redis分片集群
主从和哨兵可以解决高可用、高并发读的问题。但是依然有两个问题没有解决

1. 海量数据存储问题
2. 高并发写的问题

使用分片集群可以解决上述问题，分片集群特征

- 集群中有多个master，每个master保存不同数据
- 每个master都可以有多个slave节点
- master之间通过ping监测彼此健康状态 
- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696906237717-f163bf58-6cf4-4411-89ed-2b318d65d175.png#averageHue=%23f3e3de&clientId=ue8397173-0857-4&from=paste&height=379&id=u17ff3a52&originHeight=458&originWidth=451&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=153522&status=done&style=none&taskId=u1959bced-ec9d-4d39-8b59-a6a768447d8&title=&width=372.7272609765152)
## 搭建分片集群
搭建的分片集群结构：3个master节点，每个master包含一个slave节点<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696906422303-b143143d-4182-481f-82cb-83fedd146610.png#averageHue=%23f7ebea&clientId=ue8397173-0857-4&from=paste&height=492&id=ub3df273a&originHeight=595&originWidth=518&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=52966&status=done&style=none&taskId=u26dab5d9-765c-4bd6-95d6-97de73e4402&title=&width=428.0991600572835)

| **IP** | **PORT** | **角色** |
| --- | --- | --- |
| 192.168.111.154 | 7001 | master |
| 192.168.111.154 | 7002 | master |
| 192.168.111.154 | 7003 | master |
| 192.168.111.154 | 8001 | slave |
| 192.168.111.154 | 8002 | slave |
| 192.168.111.154 | 8003 | slave |


1）配置文件，端口号要修改<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696945158986-47830297-4ddc-45cf-ad10-4c672d8b6c45.png#averageHue=%23f5f4f2&clientId=ue8397173-0857-4&from=paste&height=245&id=u907ed774&originHeight=297&originWidth=461&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=49222&status=done&style=none&taskId=u100cf27f-a62f-4475-9ad8-45014865a40&title=&width=380.9917235258836)
```
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

2）docker创建Redis节点<br />-p开启两个端口映射，一个外网交互，一个集群间通信
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

- `redis-cli --cluster`或者`./redis-trib.rb`：代表集群操作命令
- `create`：代表是创建集群
- `--replicas 1`或者`--cluster-replicas 1` ：指定集群中每个master的副本个数为1，此时`节点总数 ÷ (replicas + 1)` 得到的就是master的数量。因此节点列表中的前n个就是master，其它节点都是slave节点，随机分配到不同master

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

4）查看集群状态<br />随便进入一个结点`cluster nodes`
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
Redis会把每一个master节点映射到0~16383共16384个插槽(hash slot)上，查看集群信息时就能看到:
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
数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况:

- key中包含"{}”，且“0”中至少包含1个字符，“{}”中的部分是有效部分
- key中不包含“{}”，整个key都是有效部分p

例如: key是num，那么就根据num计算，如果是{itcast}num，则根据itcast计算。计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot值。

## 集群伸缩



## 故障转移



## RedisTemplate访问分片集群















