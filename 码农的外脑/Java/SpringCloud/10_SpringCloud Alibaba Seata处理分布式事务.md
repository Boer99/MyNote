# SpringCloud Alibaba Seata处理分布式事务

## 分布式事务问题

==只要用到分布式，必然会提及分布式的事务==。

在分布式之前，一切组件全都在一台机器上。

在使用分布式之后，单体应用被拆分成微服务应用，原来的三个模块被拆分成三个独立的应用，分别使用三个独立的数据源。

业务操作需要调用三个服务来完成。此时每个服务内部的数据一致性由**本地**事务来保证，但是**全局**的数据一致性问题没法保证。

![img](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636468367331-5feffbdc-ec8f-4620-9c50-d8c5485c2871.png)

一句话：==一次业务操作需要跨多个数据源或需要跨多个系统进行远程调用，就会产生分布式事务问题==。



## Seata简介与安装

[官网](http://seata.io/zh-cn/)

Seata是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。 

### 相关术语

一个典型的分布式事务过程，可以用分布式处理过程的 ==一ID+三组件== 模型来描述。

- 一ID（全局唯一的事务ID）：Transaction ID `XID`，在这个事务ID下的所有事务会被统一控制
- 三组件：
  - `Transaction Coordinator (TC)`：事务协调器，维护全局事务的运行状态，负责协调并驱动全局事务的提交或回滚；（Server端，为单独服务器部署）
  - `Transaction Manager (TM)`：事务管理器，控制全局事务的边界，负责开启一个全局事务，并最终发起全局提交或全局回滚的决议；
  - `Resource Manager (RM)`：资源管理器，控制分支事务，负责分支注册、状态汇报，并接收事务协调器的指令，驱动分支（本地）事务的提交和回滚

Seata分TC、TM和RM三个角色，TC（Server端）为==单独服务端部署==，TM和RM（Client端）由==业务系统集成（微服务）==

### 典型的分布式控制事务流程

1. TM 向 TC 申请开启一个全局事务，全局事务创建成功并生成一个**全局唯一的 XID**；
2. XID 在微服务调用链路的上下文中传播；（也就是在多个TM，RM中传播）
3. RM 向 TC 注册分支事务，将其纳入 XID 对应全局事务的管辖；
4. TM 向 TC 发起针对 XID 的全局提交或回滚决议；
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求。

<img src="图片/10_SpringCloud Alibaba Seata处理分布式事务/1636511629414-0e4a7788-1488-40b8-8b5f-93f330a481b7.png" alt="img" style="zoom:80%;" />



### Seata-Server的下载与配置

https://github.com/seata/seata/releases，下载的是seata-server-0.9.0.zip



解压到指定目录并修改conf目录下的file.conf配置文件。

主要修改：

- 事务日志存储模式为db
- 自定义事务组名称
- 数据库连接信息

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636531841434-f5abb282-86e4-48b1-b7a0-71ad75de1679.png)



数据库新建库seata，建表db_store.sql在\seata-server-0.9.0\seata\conf目录里面



修改seata-server-0.9.0\seata\conf目录下的registry.conf配置文件

目的是：指明注册中心为nacos，及修改nacos连接信息

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636533822408-2ed7382f-c9ba-41f2-af4d-c7a7313b4c1a.png)



启动nacos和seata，seata-server-0.9.0\seata\bin\seata-server.bat

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636540290726-7072e4b8-6751-4779-9ed6-829a06a63ee6.png)

nacos中成功注册了seata

<img src="图片/10_SpringCloud Alibaba Seata处理分布式事务/1636540463325-cae9956d-6610-43ac-ae3c-60c81e803afc.png" alt="image.png" style="zoom:80%;" />



## 订单/库存/账户业务数据库准备

### 分布式事务业务说明

这里我们会创建三个服务，一个订单服务，一个库存服务，一个账户服务。

当用户下单时，会在订单服务中创建一个订单，然后通过远程调用库存服务来扣减下单商品的库存，再通过远程调用账户服务来扣减用户账户里面的余额，最后在订单服务中修改订单状态为已完成。

 该操作跨越三个数据库，有两次远程调用，很明显会有分布式事务问题。

下订单--->扣库存--->减账户(余额)



### 创建业务数据库与表

```sql
CREATE DATABASE seata_order;
CREATE DATABASE seata_storage;
CREATE DATABASE seata_account;
```

### 按照上述3库分别创建对应业务表

```mysql
# seata_order库下建t_order表：
CREATE TABLE seata_order.`t_order` (
  `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
  `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
  `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
  `count` INT(11) DEFAULT NULL COMMENT '数量',
  `money` DECIMAL(11,0) DEFAULT NULL COMMENT '金额',
  `status` INT(1) DEFAULT NULL COMMENT '订单状态：0：创建中；1：已完结' 
) ENGINE=INNODB AUTO_INCREMENT=7 DEFAULT CHARSET=utf8;

# seata_storage库下建t_storage 表:
CREATE TABLE `seata_storage`.`t_storage` (
 `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY,
 `product_id` BIGINT(11) DEFAULT NULL COMMENT '产品id',
 `total` INT(11) DEFAULT NULL COMMENT '总库存',
 `used` INT(11) DEFAULT NULL COMMENT '已用库存',
 `residue` INT(11) DEFAULT NULL COMMENT '剩余库存'
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

INSERT INTO seata_storage.t_storage(`id`, `product_id`, `total`, `used`, `residue`)
VALUES ('1', '1', '100', '0', '100');

# seata_account库下建t_account 表
CREATE TABLE `seata_account`.t_account (
  `id` BIGINT(11) NOT NULL AUTO_INCREMENT PRIMARY KEY COMMENT 'id',
  `user_id` BIGINT(11) DEFAULT NULL COMMENT '用户id',
  `total` DECIMAL(10,0) DEFAULT NULL COMMENT '总额度',
  `used` DECIMAL(10,0) DEFAULT NULL COMMENT '已用余额',
  `residue` DECIMAL(10,0) DEFAULT '0' COMMENT '剩余可用额度'
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
 
INSERT INTO seata_account.t_account(`id`, `user_id`, `total`, `used`, `residue`)  VALUES ('1', '1', '1000', '0', '1000');
```



### 按照上述3库分别建对应的回滚日志表

订单-库存-账户3个库下都需要建各自的回滚日志表，\seata-server-0.9.0\seata\conf目录下的db_undo_log.sql；

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636548808495-fa194c4f-7fc4-4863-871d-b044e93b9500.png)



## 订单/库存/账户业务微服务准备

业务需求：下订单->减库存->扣余额->改(订单)状态



### 新建订单Order-Module

新建seata-order-service2001

pom

```xml
<dependencies>
    <!--nacos-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--seata-->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
        <!-- 因为兼容版本问题,所以需要剔除它自带的seata的包 -->
        <exclusions>
            <exclusion>
                <artifactId>seata-all</artifactId>
                <groupId>io.seata</groupId>
            </exclusion>
        </exclusions>
    </dependency>
        <!-- 要跟我们安装SeaTa的一致！ -->
    <dependency>
        <groupId>io.seata</groupId>
        <artifactId>seata-all</artifactId>
        <version>0.9.0</version>
    </dependency>
    <!--feign-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <!--web-actuator-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--mysql-druid-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.22</version>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.1.10</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.0.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

yml

```yml
server:
  port: 2001

spring:
  application:
    name: seata-order-service
  cloud:
    alibaba:
      seata:
        # 自定义事务组名称需要与seata-server中file.conf中配置的事务组ID对应
        # vgroup_mapping.my_test_tx_group = "my_group"
        tx-service-group: my_group
    nacos:
      discovery:
        server-addr: localhost:8848
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3307/seata_order?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=UTC
    username: root
    password: root

feign:
  hystrix:
    enabled: false

logging:
  level:
    io:
      seata: info

mybatis:
  mapperLocations: classpath:mapper/*.xml
```



file.conf

- 程序中依赖的是 seata-all，对应于 *.conf 文件，所以需要在resource新建.conf文件
- 高版本的支持yml、properties配置。
- 这里仅仅是seata-order-service2001模块的file.conf（配置2001的分布式事务），seata软件那里配置的是总控file.conf。

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636715541883-b0548d85-0e4a-440e-821b-f07c2d75f270.png)



registry.conf：指明注册到nacos中

<img src="图片/10_SpringCloud Alibaba Seata处理分布式事务/1636553697753-f5c87f38-aeed-42e2-8e09-5c8daf7f81bb.png" alt="image.png" style="zoom:80%;" />



domain（实体类）：新建Order类与CommonResult类



Dao接口及实现(SQL映射文件)

```java
@Mapper
public interface OrderDao {
    /**
     * 创建订单
     */
    void create(Order order);

    /**
     * 修改订单状态，从0改为1
     */
    void update(@Param("userId") Long userId, @Param("status") Integer status);
}
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="com.atguigu.cloudalibaba.dao.OrderDao">
    <!--定义一个结果集和实体类的映射表-->
    <resultMap id="BaseResultMap" type="com.atguigu.cloudalibaba.domain.Order">
        <id column="id" property="id" jdbcType="BIGINT"/>
        <result column="user_id" property="userId" jdbcType="BIGINT"/>
        <result column="product_id" property="productId" jdbcType="BIGINT"/>
        <result column="count" property="count" jdbcType="INTEGER"/>
        <result column="money" property="money" jdbcType="DECIMAL"/>
        <result column="status" property="status" jdbcType="INTEGER"/>
    </resultMap>

    <insert id="create">
        INSERT INTO `t_order` (`id`, `user_id`, `product_id`, `count`, `money`, `status`)
        VALUES (NULL, #{userId}, #{productId}, #{count}, #{money}, 0);
    </insert>

    <update id="update">
        UPDATE `t_order`
        SET status = 1
        WHERE user_id = #{userId} AND status = #{status};
    </update>
</mapper>
```



service

```java
public interface OrderService {
    /**
     * 创建订单
     * @param order
     */
    void create(Order order);
}
```

```java
/**
 * 通过OpenFeign远程调用库存的微服务
 */
@FeignClient(value = "seata-storage-service")
public interface StorageService {

    /**
     * 扣减库存，比如买了5个1号商品：对1号商品库存减5
     */
    @PostMapping(value = "/storage/decrease")
    CommonResult decrease(@RequestParam("productId") Long productId, @RequestParam("count") Integer count);
}
```

```java
/**
 * 通过OpenFeign远程调用账号微服务
 */
@FeignClient(value = "seata-account-service")
public interface AccountService {
    /**
     * 扣减账户余额，需要传入用户ID跟扣除的金额
     */
    @PostMapping("/account/decrease")
    CommonResult decrease(@RequestParam("userId") Long userId, @RequestParam("money") BigDecimal money);
}
```

```java
@Service
@Slf4j
public class OrderServiceImpl implements OrderService {
    @Resource
    private OrderDao orderDao;

    @Resource
    private StorageService storageService;

    @Resource
    private AccountService accountService;

    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 简单说：下订单->减库存->减余额->改状态
     */
    @Override
    public void create(Order order) {
        log.info("------->下单开始");
        //本应用创建订单
        orderDao.create(order);

        //远程调用库存服务扣减库存
        log.info("------->订单微服务调用库存微服务，扣减库存开始");
        storageService.decrease(order.getProductId(), order.getCount());
        log.info("------->订单微服务调用库存微服务，扣减库存结束");

        //远程调用账户服务扣减余额
        log.info("------->订单微服务调用账户微服务，扣减余额开始");
        accountService.decrease(order.getUserId(), order.getMoney());
        log.info("------->订单微服务调用账户微服务，减余额结束");

        //修改订单状态为已完成
        log.info("------->order-service中修改订单状态开始");
        // 这里的话是不是应该是orderId？
        orderDao.update(order.getUserId(), 0);
        log.info("------->order-service中修改订单状态结束");

        log.info("------->下单结束");
    }
}
```



controller

```java
@RestController
public class OrderController {
    @Autowired
    private OrderService orderService;

    /**
     * 创建订单
     */
    @GetMapping(value = "/order/create")
    public CommonResult create(Order order) {
        orderService.create(order);
        return new CommonResult(200, "订单创建成功!");
    }
}
```



config

```java
@Configuration
@MapperScan({"com.boer.springcloud.dao"})
public class MyBatisConfig {}
```

```java
/**
 * 使用Seata对数据源进行代理
 */
@Configuration
public class DataSourceProxyConfig {

    @Value("${mybatis.mapperLocations}")
    private String mapperLocations;

    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource druidDataSource() {
        return new DruidDataSource();
    }

    @Bean
    public DataSourceProxy dataSourceProxy(DataSource dataSource) {
        return new DataSourceProxy(dataSource);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactoryBean(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSourceProxy);
        sqlSessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(mapperLocations));
        sqlSessionFactoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return sqlSessionFactoryBean.getObject();
    }

}
```



主启动类

```java
@EnableFeignClients
@EnableDiscoveryClient
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class) //取消数据源的自动创建
public class SeataOrderMainApp2001 {
    public static void main(String[] args) {
        SpringApplication.run(SeataOrderMainApp2001.class, args);
    }
}
```



### 新建订单Storage-Module

省略



### 新建订单Account-Module

省略



## 测试

### Seata全局事务怎么使用

Spring提供的本地事务：@Transactional

Seata提供的全局事务：`@GlobalTransactional`

<img src="图片/10_SpringCloud Alibaba Seata处理分布式事务/1636880513744-d81289ab-be90-4832-b76c-facc47ef00d9.png" alt="img" style="zoom:80%;" />



### 测试正常下单

http://localhost:2001/order/create?userId=1&productId=1&count=10&money=100

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636885386660-f73be4c7-8f96-4436-928d-1b8924c3e812.png)

![image.png](图片/10_SpringCloud Alibaba Seata处理分布式事务/1636886967312-d6cb850f-eef7-43bb-be02-6c9e46b69755.png)

自行查看数据库的情况



### 测试超时异常：不加@GlobalTransactional

AccountServiceImpl添加超时：我们使用的是Openfeign，默认超时时长是1s，这里我们延迟30s。

<img src="图片/10_SpringCloud Alibaba Seata处理分布式事务/1636887315777-49f3f304-95a5-4e84-84e1-db98d2063d0a.png" alt="image.png" style="zoom:80%;" />



报错超时异常

<img src="图片/10_SpringCloud Alibaba Seata处理分布式事务/1636887516874-e984a427-fd14-49c6-a025-d8a2ec94f904.png" alt="image.png" style="zoom:80%;" />



数据库情况：当库存和账户金额扣减后，订单状态并没有设置为已经完成，没有从零改为1；而且由于feign的重试机制，账户余额还有可能被多次扣减。



### 测试超时异常：加@GlobalTransactional

OrderServiceImpl添加 `@GlobalTransactional` 注解，注意改注解只能用在方法上！

该注解的属性：

- name：给定全局事务实例的名称，随便取，唯一即可
- rollbackFor：当发生什么样的异常时，进行回滚
- noRollbackFor：发生什么样的异常不进行回滚。

```java
@Override
// 全局事务，发生异常进行回滚
@GlobalTransactional(name = "fsp-create-order",rollbackFor = Exception.class)
public void create(Order order) {
    ......
}
```



访问：localhost:2001/order/create?userId=1&productId=1&count=10&money=100

依然超时异常，数据库中的数据根本就没有变化，记录都添加不进来，说明回滚成功！



## 一部分补充

先跳过啦
