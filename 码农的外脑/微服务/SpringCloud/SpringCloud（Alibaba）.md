```ad-summary
参考：heima2021

官方文档：[Spring Cloud Alibaba 是什么 - Spring Cloud Alibaba官网 (aliyun.com)](https://sca.aliyun.com/docs/2021/overview/what-is-sca/?spm=5176.29160081.0.0.74807a3ck9HpqP)
```

# 导学

微服务技术 > springcloud

微服务技术栈：

![](assets/Pasted%20image%2020240812235104.png)

学习路线：

![](assets/Pasted%20image%2020240812235826.png)

# 认识微服务

## 服务架构演变

单体架构：将业务的所有功能集中在一个项目中开发，打成一个包部署

- 优点：架构简单、部署成本低
- 缺点：耦合度高

分布式架构：根据业务功能对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务

- 优点：降低服务耦合、有利于服务升级拓展
- 缺点：架构复杂，运维、监控、部署难度提高
- 分布式架构的要考虑的问题：
	- 服务拆分粒度如何?
	- 服务集群地址如何维护?
	- 服务之间如何实现远程调用?
	- 服务健康状态如何感知?

微服务是一种经过良好架构设计的==分布式架构方案==，微服务架构特征：

- 单一职责：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
- 面向服务：微服务对外暴露业务接口
- 自治：团队独立、技术独立、数据独立、部署独立
- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

微服务技术对比：

![](assets/Pasted%20image%2020240813235429.png)

企业需求：

![](assets/Pasted%20image%2020240813235606.png)

## SpringCloud

```ad-note
官方说明：

Spring Cloud 是分布式微服务架构的一站式解决方案，它提供了一套简单易用的编程模型，使我们能在 Spring Boot 的基础上轻松地实现微服务系统的构建。 **Spring Cloud 提供以微服务为核心的分布式系统构建标准。**

Spring Cloud 本身并不是一个开箱即用的框架，它是一套微服务规范，共有两代实现。

- Spring Cloud Netflix 是 Spring Cloud 的第一代实现，主要由 Eureka、Ribbon、Feign、Hystrix 等组件组成。
- Spring Cloud Alibaba 是 Spring Cloud 的第二代实现，主要由 Nacos、Sentinel、Seata 等组件组成。
```

SpringCloud 是目前国内使用最广泛的微服务框架。官网地址：[Spring Cloud](https://spring.io/projects/spring-cloud)

SpringCloud 集成了各种微服务功能组件，并基于 SpringBoot 实现了这些组件的自动装配，从而提供了良好的开箱即用体验：

![](assets/Pasted%20image%2020240813235944.png)

SpringCloud 和 SpringBoot 版本兼容关系：

![600](assets/Pasted%20image%2020240814000537.png)

# 服务拆分及远程调用

## 服务拆分

注意事项：
1. 不同微服务，不要重复开发相同业务
2. 微服务数据独立，不要访问其它微服务的数据库
3. 微服务可以将自己的业务暴露为接口，供其它微服务调用

项目结构：
- cloud-demo
	- order-service（根据 id 查询订单）
	- user-service（根据 id 查询用户）

## 远程调用

查询订单对应的用户信息，order-service远程调用user-service

1）注册RestTemplate：在 order-service 的 OrderApplication 中注册 RestTemplate

```java
@MapperScan("cn.itcast.order.mapper")
@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

2）远程调用

```java
@Service
public class OrderService {

    @Resource
    private OrderMapper orderMapper;
    @Resource
    private RestTemplate restTemplate;

    private final static String URL = "http://localhost:8081/user/";

    public Order queryOrderById(Long orderId) {
        Order order = orderMapper.findById(orderId);
        User user = restTemplate.getForObject(URL + order.getUserId().toString(), User.class);
        order.setUser(user);
        return order;
    }
}
```

## 服务调用关系

- *服务提供者*：暴露接口给其它微服务调用
- *服务消费者*：调用其它微服务提供的接口

> 服务 A 调用服务 B，服务 B 调用服务 C，那么服务 B 是什么角色?

提供者与消费者角色是相对的，一个服务可以同时是服务提供者和服务消费者


# Eureka 注册中心

## 服务调用出现的问题

- 服务消费者该如何获取服务提供者的地址信息?
- 如果有多个服务提供者，消费者该如何选择?
- 消费者如何得知服务提供者的健康状态?

## Eureka 的作用

在 Eureka 架构中，微服务角色有两类：
- Eureka Server：服务端，**注册中心**
	- 记录服务信息
	- 心跳监控
- Eureka Client：客户端
	- Provider：服务提供者（例如案例中的 user-service）
		- 注册自己的信息到 Eureka Server
		- 每隔 30 秒向 EurekaServer **发送心跳**
	- consumer：服务消费者（例如案例中的 order-service ）
		- 根据服务名称从 Eureka server 拉取服务列表
		- 基于服务列表做负载均衡，选中一个微服务后发起远程调用

Eureka 的作用：
- 消费者该如何获取服务提供者具体信息?
	- 服务提供者启动时向 eureka **注册**自己的信息
	- eureka 保存这些信息
	- **消费者**根据服务名称向 eureka 拉取提供者信息
- 如果有多个服务提供者，消费者该如何选择?
	- 服务消费者利用**负载均衡**算法，从服务列表中挑选一个
- 消费者如何感知服务提供者健康状态?
	- 服务提供者会每隔 30 秒向 EurekaServer 发送心跳请求，报告健康状态 
	- eureka 会更新记录服务列表信息，心跳不正常会被剔除
	- 消费者就可以拉取到最新的信息

![](assets/Pasted%20image%2020240415191728.png)

## 动手实践

整体流程：
- 搭建 EurekaServer
- 将 user-service、order-service 都注册到 eureka
- 在 order-service 中完成服务拉取，然后通过负载均衡挑选一个服务，实现远程调用

### 搭建 eureka-server

eureka-server 必须是一个独立的微服务

1）引入 eureka-server 依赖

```xml
<dependencies>
    <!--eureka-server-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
```

2）编写启动类，添加 `@EnableEurekaServer` 注解

```java
@SpringBootApplication
@EnableEurekaServer  //表示当前是Eureka的服务注册中心
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```

3）配置文件

```yml
server:
  port: 10086
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url:
      # EurekaServer的地址，现在是自己的地址，如果是集群，需要加上其它Server的地址
      defaultZone: http://127.0.0.1:10086/eureka/
```

测试：浏览器访问 http://localhost:7001

- instances currently registered with euraka：已经注册到 Eureka 的实例

![](assets/Pasted%20image%2020240814231910.png)

### 服务注册

将 user-service 服务注册到 EurekaServer（order-service 同理）

1）添加 Eureka-Client 依赖

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2）配置文件添加如下配置：

```yml
spring:
  application:
    name: user-service # 微服务的注册名称
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

另外，可以将服务多次启动模拟多实例部署，但为了避免端口冲突，需要修改端口设置

- 再启动一个服务，VM options 设置 `-Dserver.port=8082`

![](assets/Pasted%20image%2020240814234025.png)

### 服务发现 & 负载均衡

服务拉取：基于服务名称获取**服务列表**，然后再对服务列表做**负载均衡**

1）修改 order-service 中调用的 url 路径，==用服务名代替 ip、端口==

```java
private final static String URL = "http://user-service/user/";
```

2）使用 `@LoadBalanced` 注解赋予 RestTemplate 负载均衡的能力

```java
@Bean
@LoadBalanced //赋予 RestTemplate 负载均衡的能力
public RestTemplate restTemplate() {
	return new RestTemplate();
}
```

> 查看两个 user-service 的 mybatis 日志，都被调用了


# Ribbon 负载均衡

## 流程

![](assets/Pasted%20image%2020240416233649.png)

```ad-tip
Ribbon 负载均衡 VS Nginx 负载均衡？

- Nginx 是服务器负载均衡，客户端所有请求都会交给 nginx，然后由 nginx 实现转发请求。即负载均衡是由==服务端实现==的。
- Ribbon 本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到 JVM 本地，从而在==本地实现== RPC 远程服务调用技术
```

## 原理

SpringCloud Ribbon 的底层采用了一个拦截器，拦截了 RestTemplate 发出的请求，对地址做了修改

基本流程如下：

- 拦截我们的 RestTemplate 请求 [http://userservice/user/1](http://userservice/user/1)
- RibbonLoadBalancerClient 会从请求 url 中获取服务名称，也就是 user-service
- DynamicServerListLoadBalancer 根据 user-service 到 eureka 拉取服务列表
	- eureka 返回列表，localhost:8081、localhost:8082
- 根据具体的 **IRule** 负载均衡策略，从列表中选择一个
- RibbonLoadBalancerClient 修改请求地址

![](assets/Pasted%20image%2020240417000140.png)

## 负载均衡策略

负载均衡的规则都定义在 IRule 接口中，而 IRule 有很多不同的实现类，每一个实现类都代表一个策略

![](assets/Pasted%20image%2020240417001034.png)

| **内置负载均衡规则类**             | **规则描述**                                                                                                                                                                                                                                                                |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RoundRobinRule            | 简单轮询服务列表来选择服务器。它是 Ribbon **默认**的负载均衡规则。                                                                                                                                                                                                                                 |
| AvailabilityFilteringRule | 对以下两种服务器进行忽略：<br> （1）在默认情况下，这台服务器如果 3 次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续 30 秒，如果再次连接失败，短路的持续时间就会几何级地增加。 <br>（2）并发数过高的服务器。如果一个服务器的并发连接数过高，配置了 AvailabilityFilteringRule 规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的 `<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit` 属性进行配置。 |
| WeightedResponseTimeRule  | 为每一个服务器赋予一个权重值。服务器响应时间越长，这个服务器的权重就越小。这个规则会随机选择服务器，这个权重值会影响服务器的选择。                                                                                                                                                                                                       |
| **ZoneAvoidanceRule**     | 以区域可用的服务器为基础进行服务器的选择。使用 Zone 对服务器进行分类，这个 Zone 可以理解为一个机房、一个机架等。而后再对 Zone 内的多个服务做轮询。                                                                                                                                                                                      |
| BestAvailableRule         | 忽略那些短路的服务器，并选择并发数较低的服务器。                                                                                                                                                                                                                                                |
| RandomRule                | 随机选择一个可用的服务器。                                                                                                                                                                                                                                                           |
| RetryRule                 | 重试机制的选择逻辑                                                                                                                                                                                                                                                               |

## 配置负载均衡策略

服务调用方通过定义 IRule 实现可以修改负载均衡规则，有两种方式：

1）代码方式：在配置类中，定义一个新的 IRule：

```java
@Configuration  
public class ApplicationContextConfig {  
  
    @Bean  
    @LoadBalanced //使用@LoadBalanced注解赋予RestTemplate负载均衡的能力  
    public RestTemplate restTemplate() {  
        return new RestTemplate();  
    }  
  
    @Bean  
    public IRule randomRule(){  
        return new RandomRule();  
    }  
}
```

2）配置文件方式：在 application.yml 文件中，添加新的配置也可以修改规则

```yml
cloud-payment-service: # 给某个微服务配置负载均衡规则，这里是userservice服务
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则 
```

## 饥饿加载

Ribbon 默认是采用**懒加载**，即第一次访问时才会去创建 LoadBalanceClient，==请求时间会很长==。

而**饥饿加载**则会在项目启动时创建，降低第一次访问的耗时

通过下面配置开启饥饿加载：

```yml
ribbon:  
  eager-load:  
    enabled: true # 开启饥饿加载
    clients: userservice # 指定对userservice服务饥饿加载
```


# Nacos 注册中心

官网文档：[https://nacos.io/zh-cn/index.html]()

## Nacos 简介

[Nacos](https://nacos.io/)（**Na**ming **Co**nfiguration **S**ervice）是阿里巴巴的产品，现在是 [SpringCloud](https://spring.io/projects/spring-cloud) 中的一个组件。相比 [Eureka](https://github.com/Netflix/eureka) 功能更加丰富，在国内受欢迎程度较高。

官网：一个更易于构建云原生应用的动态服务发现、配置管理和服务的管理平台。

## 安装和运行

下载地址：[https://github.com/alibaba/Nacos]()

1）运行 bin 目录下的 startup.cmd，默认的是集群模式启动

```bash
# 单机模式启动
# windows
startup.cmd -m standalone
# mac
sh startup.sh -m standalone
```

2）运行成功，访问 http://localhost:8848/nacos ，默认账号密码都是 nacos

## 服务注册到 Nacos

1）父工程中添加管理依赖

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>com.alibaba.cloud</groupId>
			<artifactId>spring-cloud-alibaba-dependencies</artifactId>
			<version>2.2.5.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

2）注释掉 order-service 和 user-service 中原有的 eureka 依赖

3）添加 nacos 客户端依赖

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

4）客户端添加 nacos 地址

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848 # 配置Nacos地址 ，注册到 Nacos
```

5）启动并测试

![](assets/Pasted%20image%2020240816002259.png)


## 服务分级存储模型

> 一个**服务**可以有多个**实例**，例如 user-service，可以有:
> 
> - 127.0.0.1:8081
> - 127.0.0.1:8082
> - 127.0.0.1:8083
> 
> 假如这些实例分布于全国各地的不同机房，例如：
> 
> - 127.0.0.1:8081，在上海机房
> - 127.0.0.1:8082，在上海机房
> - 127.0.0.1:8083，在杭州机房

Nacos 就将同一机房内的实例 划分为一个**集群**

也就是说，user-service 是服务，一个服务可以包含多个集群，如杭州、上海，每个集群下可以有多个实例，形成分级模型，如图：

![](assets/Pasted%20image%2020240417172307.png)

Nacos 服务分级存储模型

1. 一级是**服务**，例如 userservice
2. 二级是**集群**，例如 杭州或上海
3. 三级是**实例**，例如 杭州机房的某台部署了 userservice 的服务器

服务调用尽可能选择本地集群的服务，跨集群调用延迟较高。本地集群不可访问时，再去访问其它集群

### 服务集群属性

1）配置文件中添加集群属性

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

2）复制多个实例，通过以下参数启动

```bash
-Dserver.port=8082 -Dspring.cloud.nacos.discovery.cluster-name=HZ
-Dserver.port=8083 -Dspring.cloud.nacos.discovery.cluster-name=HZ
-Dserver.port=8084 -Dspring.cloud.nacos.discovery.cluster-name=SH
```

3）查 Nacos 看控制台

![](assets/Pasted%20image%2020240417181122.png)

### NacosRule 负载均衡策略

默认的 ZoneAvoidanceRule 并不能根据**同集群优先**来实现负载均衡。

Nacos 中提供了一个 NacosRule
- 优先从同集群中挑选实例
- 本地集群找不到提供者，才去其它集群寻找，并且会报警告
- 确定了可用实例列表后，再采用随机负载均衡挑选实例

1）在 order-service 中配置集群属性，并配置调用 user-service 的负载均衡的 IRule 为 NacosRule：

```yml
spring:  
  cloud:  
    nacos:  
      server-addr: localhost:8848 # 配置Nacos地址 ，注册到 Nacos      discovery:  
        cluster-name: HZ  
user-service:  
  ribbon:  
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule
```

2）将 user-service 的 权重都设置为 1

## 根据权重负载均衡

> 服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求。

默认情况下 NacosRule 是同集群内随机挑选，不会考虑机器的性能问题。Nacos 提供了权重配置来控制访问频率，==权重越大则访问频率越高==。

1）在 nacos 控制台编辑实例的权重

![500](assets/Pasted%20image%2020240417202003.png)

2）将实例的权重设置为 0.1，测试可以发现该实例被访问到的频率大大降低

## 环境隔离-namespace

Nacos 提供了 namespace 来实现环境隔离功能

- nacos 中可以有多个 namespace
- namespace 下可以有 group、service 等
- 不同 namespace 之间相互隔离（不可见）

![500](assets/Pasted%20image%2020240417203257.png)

默认情况下，所有 service、data、group 都在名为 **public** 的 namespace 下

1）在 Nacos 控制台创建 namespace

2）给微服务配置 namespace：

```yml
spring:  
  cloud:  
    nacos:  
      discovery:  
        namespace: 65527313-eef9-439e-a3e0-bf2187879a5d # namespace的id
```

> 将 consumer 服务配置在 dev namespace 下，provider 服务配置在 public namespace 下，consumer 调用不到 provider 服务

## Nacos 和 Eureka 的异同

Nacos 的服务实例分为两种类型：

- *临时实例*（默认）：如果实例宕机超过一定时间，会从服务列表剔除
- *非临时实例 / 永久实例*：如果实例宕机，不会从服务列表剔除

配置一个服务实例为永久实例：

```yml
 spring:  
   cloud:  
     nacos:  
       discovery:  
         ephemeral: false # 设置为永久实例
```

Nacos 和 Eureka 整体结构类似，服务注册、服务拉取、心跳等待，但是也存在一些差异：

- 共同点
    - 都支持服务注册和服务拉取
    - 都支持服务提供者**心跳方式**做健康检测
- 区别
    - Nacos 支持服务端主动检测提供者状态：
	    - 临时实例（干儿子）采用**心跳**模式，
	    - 非临时实例（亲儿子）采用**主动检测**模式
    - Nacos 支持服务列表变更的**消息推送**模式，服务列表更新更及时
    - CAP：
	    - Nacos 集群**默认**采用 AP 方式。当集群中存在非临时实例时，采用 CP 模式；
	    - Eureka 采用 AP 方式

![](assets/Pasted%20image%2020240417204836.png)

# Nacos 配置管理

## 统一配置管理

配置更改热更新（开关、模板等类型的配置）

1）在nacos中添加配置信息

![](assets/Pasted%20image%2020240817120106.png)

```yml
pattern:
  dateformat: yyyy-MM-dd HH:mm:ss
```

## 微服务配置拉取

配置获取的步骤：

![](assets/Pasted%20image%2020240817121353.png)

> bootstrap 的读取先于 application，用于配置 nacos 地址

1）引入Nacos的配置管理客户端依赖

```xml
<dependency>
	<groupId>com.alibaba.cloud</groupId>
	<artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

2）在 userservice 中的 resource 目录添加一个 bootstrap.yml 文件

```yml
spring:
  application:
    name: user-service
  profiles:
    active: dev # 开发环境
  cloud:
    nacos:
      server-addr: localhost:8848
      config:
        file-extension: yaml # 文件后缀名
```

3）在 user-service 中将pattern.dateformat 这个属性注入到 UserController 中做测试

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @Resource
    private UserService userService;

	@Value("${pattern.dateformat}")
    private String dateformat;

    @GetMapping("now")
    public String now() {
        log.debug(dateformat);
        return LocalDateTime.now().format(DateTimeFormatter.ofPattern(dateformat));
    }
}
```


## 配置热更新

Nacos 中的配置文件变更后，微服务无需重启就可以感知，可以通过以下两种配置实现：

1）方式一：在 `@Value` 注入的变量所在类上添加注解 `@RefreshScope`

```java
@RefreshScope  
public class UserController {}
```

2）使用 `@ConfigurationProperties` 注解（推荐）

```java
@Data
@Component
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
}
```

```java
@Resource
private PatternProperties patternProperties;

@GetMapping("now")
public String now() {
	return LocalDateTime.now().format(DateTimeFormatter.ofPattern(patternProperties.getDateformat()));
}
```


## 多环境配置共享

微服务启动时会从 nacos 读取多个配置文件:

- `[spring.application.name]-[spring.profiles.active].yaml`，例如: `userservice-dev.yaml `
- `[spring.application.name].yaml`，例如: `userservice.yaml` 

无论 profile 如何变化，`[spring.application.name].yaml` 这个文件一定会加载，因此多环境**共享配置**可以写入这个文件

**多种配置的优先级**：`[服务名]-[profile].yaml` > `[服务名].yaml` > 本地配置


## nacos 集群搭建

[集群部署说明 | Nacos 官网](https://nacos.io/docs/v2/guide/admin/cluster-mode-quick-start/)

官方的 nacos 集群图：

![](assets/Pasted%20image%2020240817162757.png)

其中包含 3 个 nacos 节点，然后一个负载均衡代理 3 个 Nacos，这里的负载均衡器可以使用 nginx

![400](assets/Pasted%20image%2020240817161935.png)

搭建集群的基本步骤：[集群部署说明 | Nacos 官网](https://nacos.io/docs/v2/guide/admin/cluster-mode-quick-start/)

1）下载

2）搭建数据库，初始化数据库表结构

[nacos/distribution/conf/mysql-schema.sql at master · alibaba/nacos · GitHub](https://github.com/alibaba/nacos/blob/master/distribution/conf/mysql-schema.sql)


3）配置集群配置文件

nacos/conf/cluster.conf

```yml
# ip:port
127.0.0.1:8845
127.0.0.1:8846
127.0.0.1:8847
```

在 nacos/conf/application.properties 配置
- nacos 的端口
- 数据库连接地址

```properties
server.port=8845
```

4）启动 nacos 集群 #todo 

5）nginx 反向代理 #todo


# http 客户端 Feign

官方地址： https://github.com/OpenFeign/feign

## 介绍

用 RestTemplate 发起远程调用代码存在以下问题:
- 代码可读性差，编程体验不统一
- 参数复杂 URL 难以维护

```java
User user = restTemplate.getForObject(URL + order.getUserId().toString(), User.class);
```

Feign 是一个声明式的 http 客户端，其作用就是帮助我们优雅的实现 http 请求的发送 ，解决上面提到的问题。

## 基本使用

1）引入依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2）启动类添加注解开启 Feign 的功能

```java
@EnableFeignClients
@SpringBootApplication
public class OrderApplication {}
```

3）编写 Feign 客户端

```java
@FeignClient("user-service")
public interface UserClient {
    @GetMapping("/user/{id}")
    User queryById(@PathVariable("id") Long id);
}
```

主要是基于 SpringMVC 的注解来声明远程调用的信息，比如

- 服务名称：userservice
- 请求方式：GET
- 请求路径：/user/{id}
- 请求参数：Long id
- 返回值类型：User\

> 直接从 controller 复制，别忘了路径前缀 /user

4）用Feign客户端代替 RestTemplate

```java
@Service
public class OrderService {
    @Resource
    private OrderMapper orderMapper;
	@Resource
    private UserClient userClient;

    public Order queryOrderById(Long orderId) {
        Order order = orderMapper.findById(orderId);
        User user = userClient.queryById(order.getUserId());
        order.setUser(user);
        return order;
    }
}
```

## 自定义配置

Feign 运行自定义配置来覆盖默认配置，可以修改的配置如下：

![](assets/Pasted%20image%2020240817194546.png)

一般我们需要配置的就是日志级别，配置Feign日志有两种方式：

1）方式一：配置文件

全局生效：

```yml
feign:
  client:
    config:
      default: # 全局配置
        logger-level: FULL
```

局部生效：

```yml
feign:
  client:
    config:
      user-service: # 服务名称
        logger-level: FULL
```

2）方式二：代码

![700](assets/Pasted%20image%2020240817195911.png)

## 性能优化

Feign 底层的客户端实现：

- URLConnection: 默认实现，不支持连接池
- Apache HttpClient: 支持连接池
- OKHttp: 支持连接池

因此优化 Feign 的性能主要包括：

- 使用连接池代替默认的 URLConnection 
- 日志级别，最好用 basic 或 none

### 连接池配置

Feign 添加 HttpClient 的支持

1）引入依赖

```xml
<dependency>
	<groupId>io.github.openfeign</groupId>
	<artifactId>feign-httpclient</artifactId>
</dependency>
```

2）配置连接池

```yml
feign:
  client:
    config:
      default: # 全局配置
        logger-level: BASIC # 日志级别
  httpclient:
    enabled: true
    max-connections: 200 # 最大连接数
    max-connections-per-route: 50 # 每个路径的最大连接数
```

## 最佳实践

1）方式一（继承）: 给消费者的 FeignClient 和提供者的 controller 定义统一的父接口作为标准。

![](assets/Pasted%20image%2020240817202453.png)

官方不推荐这种方式：

- 服务紧耦合
- 父接口参数列表中的映射不会被继承

![](assets/Pasted%20image%2020240817203050.png)

2）方式二（抽取）: 将 FeignClient 抽取为独立模块，并且把接口有关的 POJO、默认的 Feign 配置都放到这个模块中，提供给所有消费者使用

![](assets/Pasted%20image%2020240817203419.png)

### 方式二实操

> 具体见代码

实现最佳实践方式二的步骤如下:

1. 首先创建一个 module，命名为 feign-api，然后引入 feign 的 starter 依赖
2. 将 order-service 中编写的 UserClient、User、DefaultFeignConfiguration 都复制到 feign-api 项目中
3. 在 order-service 中引入 feign-api 的依赖
4. 修改 order-service 中的所有与上述三个组件有关的 import 部分，改成导入 feign-api 中的包
5. 重启测试

当定义的 FeignClient 不在 SpringBootApplication 的扫描包范围时，这些 FeignClient 无法使用。有两种方式解决: 

方式一：指定 FeignClient 所在包
```java
@EnableFeignClients(basePackages = {"cn.itcast.feign.clients"})
@SpringBootApplication  
public class OrderApplication {}
```

方式二：指定 FeignClient 字节码

```java
@EnableFeignClients(clients = {UserClient.class})
@SpringBootApplication  
public class OrderApplication {}
```


# 统一网关 Gateway

## 介绍

网关功能：

- 身份认证和权限校验
- 服务路由、负载均衡
- 请求限流

![](assets/Pasted%20image%2020240817222537.png)

Zuul 是基于 Servlet 的实现，属于阻塞式编程。而 SpringCloudGateway 则是基于 Spring5 中提供的 WebFlux，属于响应式编程的实现，具备更好的性能。

## 搭建

1）创新的 module，引入SpringCloudGateway 的依赖和 nacos 的服务发现依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-gateway</artifactId>
	</dependency>
	<dependency>
		<groupId>com.alibaba.cloud</groupId>
		<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
	</dependency>
</dependencies>
```

2）配置

路由配置包括：

- 路由 id：路由的唯一标示
- 路由目标(uri)：路由的目标地址，http 代表固定地址，lb 代表根据服务名负载均衡
- 路由断言(predicates)：判断路由的规则，符合则转发到目标地址

```yml
server:
  port: 10010
spring:
  application:
    name: gateway # 微服务的注册名称
  cloud:
    nacos:
      server-addr: localhost:8848 # 配置Nacos地址 ，注册到 Nacos
    gateway:
      routes:
        - id: user-service # 路由标识，必须唯一
          uri: lb://user-service # 路由的目标地址
          predicates:
            - Path=/user/** # 路径断言，/user开头的符合
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/order/**
```

请求转发流程：
![](assets/Pasted%20image%2020240817231655.png)

## 路由断言工厂

我们在配置文件中写的断言规则（predicates）只是字符串，这些字符串会被 Predicate Factory 读取并处理，转变为路由判断的条件

Spring 提供了 11 种基本的 Predicate 工厂：

![](assets/Pasted%20image%2020240817233513.png)

## 路由过滤器

GatewayFilter 是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理

![](assets/Pasted%20image%2020240817235849.png)


Spring 提供了 31 种过滤器工厂

- AddRequestHeader：给当前请求添加一个请求头
- RemoveRequestHeader：移除请求中的一个请求头
- AddResponseHeader：给响应结果中添加一个响应头
- RemoveResponseHeader：从响应结果中移除有一个响应头
- RequestRateLimiter：限制请求的流量

---

给所有进入 userservice 的请求添加一个请求头：Truth=itcast is freaking awesome!

实现方式：在 gateway 中修改 application.yml 文件，给 userservice 的路由添加过滤器

```yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service # 路由标识，必须唯一
          uri: lb://user-service # 路由的目标地址
          predicates:
            - Path=/user/** # 路径断言，/user开头的符合
          filters:
            - AddRequestHeader=Truth, hello gateway!
```

### 默认过滤器

如果要对所有的路由都生效，则可以将过滤器工厂写到 default 下

```yml
spring:
  cloud:
    gateway:
      default-filters:
        - AddRequestHeader=Truth, hello gateway!
```

### 全局过滤器

全局过滤器的作用也是处理一切进入网关的请求和微服务响应。

- 与 GatewayFilter 的作用一样，区别在于 GatewayFilter 通过配置定义，处理逻辑是固定的。而 GlobalFilter 的逻辑需要自己写代码实现。
- 定义方式是实现 GlobalFilter 接口

---

需求：定义全局过滤器，拦截请求，判断请求的参数是否满足下面条件，如果同时满足则放行，否则拦截

- 参数中是否有 authorization
- authorization 参数值是否为 admin

1）实现 GlobalFilter 接口

2）添加 `@Order` 注解或实现 Ordered 接口

3）编写处理逻辑

```java
@Component
public class AuthorizeFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> queryParams = request.getQueryParams();
        String auth = queryParams.getFirst("authorization");
        if ("admin".equals(auth)) {
            return chain.filter(exchange);
        }
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### 过滤器执行顺序

请求进入网关会碰到三类过滤器：当前路由的过滤器、DefaultFilter、GlobalFilter 

请求路由后，会将当前路由过滤器和 DefaultFilter、GlobalFilter，合并到一个过滤器链(集合)中，排序后依次执行每个过滤器

![](assets/Pasted%20image%2020240818122903.png)

过滤器的执行顺序：

- 每一个过滤器都必须指定一个 int 类型的 order 值，order 值越小，执行顺序越靠前
	- GlobalFilter 通过实现 Ordered 接口，或者添加 `@Order` 注解来指定 order 值，由我们自己指定
	- 路由过滤器和 defaultFilter 的 order 由 Spring 指定，默认是按照声明顺序从 1 递增。
- 当过滤器的 order 值一样时，会按照 ==defaultFilter > 路由过滤器 > GlobalFilter== 的顺序执行

> 可以参考下面几个类的源码来查看:
> - `org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator#getFilters()` 方法是先加载 defaultFilters，然后再加载某个 route 的 filters，然后合并。
> - `org.springframework.cloud.gateway.handler.FilteringWebHandler#handle()` 方法会加载全局过滤器与前面的过滤器合并后根据 order 排序，组织过滤器链

## 跨域问题处理

跨域：域名不一致就是跨域，主要包括

- 域名不同: [www.taobao.com]() 和 [www.taobao.org]() 和 [www.jd.com]() 和 [miaosha.jd.com]()
- 域名相同，端口不同：[localhost:8080]() 和 [localhost:8081]()

跨域问题：浏览器禁止请求的发起者与服务端发生**跨域 ajax 请求**，请求被浏览器拦截的问题

- 解决方案：CORS

网关处理跨域采用的同样是 CORS 方案，并且只需要简单配置即可实现

- 允许哪些域名跨域?
- 允许哪些请求头?
- 允许哪些请求方式?
- 是否允许使用 cookie?
- 有效期是多久?

![600](assets/Pasted%20image%2020240818125516.png)

