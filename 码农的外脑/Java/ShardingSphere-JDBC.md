
参考：[尚硅谷ShardingSphere5实战教程（快速入门掌握核心）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ta411g7Jf?spm_id_from=333.788.videopod.episodes&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa)

官方文档：[概览 :: ShardingSphere](https://shardingsphere.apache.org/document/5.3.2/cn/overview/)

代码地址：[Boer99/ShardingSphere-learn](https://github.com/Boer99/ShardingSphere-learn)


# 高性能架构模式

##  读写分离

## 数据库分片架构

> **读写分离的问题**：读写分离分散了数据库读写操作的压力，但没有分散存储压力，为了满足业务数据存储的需求，就需要 将存储分散到多台数据库服务器上。

**数据分片**：
- 将存放在单一数据库中的数据分散地存放至多个数据库或表中，以达到提升性能瓶颈以及可用性的效果。 数据分片的有效手段是对关系型数据库进行分库和分表。
- 数据分片的拆分方式又分为垂直分片和水平分片。

### 垂直分片

按照业务拆分的方式称为垂直分片，又称为纵向拆分

垂直拆分可以缓解数据量和访问量带来的问题，但无法根治。如果垂直拆分之后，表中的数据量依然超过单节点所能承载的阈值，则需要水平分片来进一步处理。

**垂直分库**：按照业务将表进行归类，分布到不同的数据库中，从而将压力分散至不同的数据库。
![](../assets/Pasted%20image%2020241117005406.png)
**垂直分表**：
- 垂直分表适合将表中某些不常用的列，或者是占了大量空间的列拆分出去
- 缺点：表操作的数量要增加

### 水平分片

水平分片又称为横向拆分，通过某个字段 (或某几个字段)，根据某种规则将数据分散至多个库或表中，每个分片仅包含数据的一部分。

> 例如：根据主键分片偶数主键的记录放入 0 库(或表)，奇数主键的记录放入 1 库(或表)

![](../assets/Pasted%20image%2020241117010228.png)
**水平分表**：单表切分为多表后，新的表即使在同一个数据库服务器中，也可能带来可观的性能提升

**水平分库**：如果单表拆分为多表后，单台服务器依然无法满足性能要求，那就需要将多个表分散在不同的数据库服务器中。

> 阿里巴巴 Java 开发手册：
> 
> 【推荐】单表行数超过 50 万行或者单表容量超过 2GB，才推荐进行分库分表
> 
> 说明：如果预计三年后的数据量根本达不到这个级别，请不要在创建表时就分库分表


# ShardingSphere

Apache shardingSphere 由 JDBC、Proxy 和 Sidecar (规划中) 这 3 款既能够独立部署，又支持混合部署配合使用的产品组成。

# ShardingSphere-JDBC

## 版本，依赖

[概览 :: ShardingSphere](https://shardingsphere.apache.org/document/5.3.2/cn/overview/)

这里采用SpringBoot3，ShardingSphere5.3.2

## 基础配置

[ShardingSphere-JDBC :: ShardingSphere](https://shardingsphere.apache.org/document/5.3.2/cn/quick-start/shardingsphere-jdbc-quick-start/)

## 垂直分片

[数据分片 :: ShardingSphere](https://shardingsphere.apache.org/document/5.3.2/cn/user-manual/shardingsphere-jdbc/yaml-config/rules/sharding/)

```yaml
dataSources:
  ds_1: # 数据源名称
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://116.198.253.55:3306/db_user
    username: root
    password: gong990117
  ds_2:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://116.198.253.55:3306/db_order
    username: root
    password: gong990117

rules:
  - !SHARDING
    tables: # 数据分片规则配置
      t_user: # 逻辑表名称
        actualDataNodes: ds_1.t_user
      t_order: # 逻辑表名称
        actualDataNodes: ds_2.t_order

# 打印SQL
props:
  sql-show: true
```

测试

```java
@SpringBootTest
public class ShardingTest {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private OrderMapper orderMapper;

    /**
     * 垂直分片：插入数据测试
     */
    @Test
    public void testInsertOrderAndUser(){
        User user = new User();
        user.setUname("强哥");
        userMapper.insert(user);

        Order order = new Order();
        order.setOrderNo("ATGUIGU001");
        order.setUserId(user.getId());
        order.setAmount(new BigDecimal(100));
        orderMapper.insert(order);
    }

    /**
     * 垂直分片：查询数据测试
     */
    @Test
    public void testSelectFromOrderAndUser(){
        User user = userMapper.selectById(1L);
        Order order = orderMapper.selectById(1L);
    }
}
```

查看控制台日志输出：

```java
Logic SQL: INSERT INTO t_user  ( uname )  VALUES  ( ? )

Actual SQL: ds_1 ::: INSERT INTO t_user  ( uname )  VALUES  (?) ::: [强哥]

Logic SQL: INSERT INTO t_order  ( order_no, user_id, amount )  VALUES  ( ?, ?, ? )

Actual SQL: ds_2 ::: INSERT INTO t_order  ( order_no, user_id, amount ) VALUES  (?, ?, ?) ::: [ATGUIGU001, 3, 100]
```

## 水平分片

![](../assets/Pasted%20image%2020241117231751.png)

配置

```yaml
dataSources:
  db_order0: # 数据源名称
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://116.198.253.55:3306/db_order0
    username: root
    password: gong990117
  db_order1:
    dataSourceClassName: com.zaxxer.hikari.HikariDataSource
    driverClassName: com.mysql.jdbc.Driver
    jdbcUrl: jdbc:mysql://116.198.253.55:3306/db_order1
    username: root
    password: gong990117

rules:
  - !SHARDING
    # --------------------数据分片规则配置
    tables:
      t_order: # 逻辑表名称
        # actualDataNodes: db_order0.t_order0, db_order0.t_order1, db_order1.t_order0, db_order1.t_order1
        # inline表达式简化
        actualDataNodes: db_order$->{0..1}.t_order$->{0..1}
        # ----------分库策略
        databaseStrategy:
          standard:
            # 分片列名称
            shardingColumn: user_id
            # 分片算法名称
            shardingAlgorithmName: alg_inline_userid
        # ----------分表策略
        tableStrategy:
          standard:
            # 分片列名称
            shardingColumn: order_no
            # 分片算法名称
            shardingAlgorithmName: alg_hash_mod
    # --------------------分片算法配置
    shardingAlgorithms:
      # ----------行表达式分片算法（分库）
      alg_inline_userid: # 分片算法名称
        type: INLINE
        props: # 分片算法属性配置
          algorithm-expression: db_order$->{user_id % 2}
      # ----------取模分片算法
#      alg_mod:
#        type: MOD
#        props:
#          sharding-count: 2
      # ----------哈希取模分片算法
      alg_hash_mod:
        type: HASH_MOD
        props:
          sharding-count: 2
# 打印SQL
props:
  sql-show: true
```

修改 Order 实体类的主键策略：

```java
//@TableId(type = IdType.AUTO)//依赖数据库的主键自增策略
@TableId(type = IdType.ASSIGN_ID)//分布式id
```

测试

```java
@Test
public void testInsertOrder() {
	Order order;
	for (int i = 11; i <= 20; i++) {
		order = new Order();
		order.setOrderNo("aaa" + i);
		order.setUserId((long) i);
		order.setAmount(new BigDecimal(100));
		orderMapper.insert(order);
	}
}
```

