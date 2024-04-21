
参考：

- 尚硅谷 SpringCloud 框架开发教程 (SpringCloud Alibaba 微服务分布式架构丨 Spring Cloud)
- 讲师：周阳

Boot 和 Cloud 课程使用版本选择

- 上篇：
	- SpringBoot：2.x
	- Spring Cloud：H 版
- 下篇：SpringCloud Alibaba
	- SpringBoot: 2.2.2.release
	- SpringCloud: Hoxton.SR1
	- Cloud Alibaba: 2.1.0.release
	- Java: Java8
	- Maven: 3.5 及以上
	- Mysql: 5.7 及以上

# ---------- 01_微服务介绍及工程搭建

# SpringCloud 简介

SpringCloud 是分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合体，俗称微服务全家桶

各模块常用的技术支撑：

- 服务注册与发现：eureka
- 服务负载与调用：ribbon、feign
- 服务熔断降级：hystrix
- 服务网关：zuul
- 服务分布式配置：Spring Cloud Config
- 服务开发：SpringBoot

Cloud 各组件的停更/升级/替换

![img](assets/01_微服务介绍及工程搭建/1630904566825-e71eedb9-4525-49f9-b324-95b047ca5440.png)

# SpringCloud Alibaba 简介

> 学习资料的获取
> 
> - 官网： https://spring.io/projects/spring-cloud-alibaba#overview
> - 英文：
> 	- https://github.com/alibaba/spring-cloud-alibaba
> 	- https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html
> - 中文：[https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md]()

### 为什么出现 SpringCloud Alibaba？

Spring Cloud Netflix 项目进入维护模式，意味着 Spring Cloud Netflix 将不再开发新的组件

https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now

Spring Cloud 版本迭代算是比较快的，因而出现了很多重大 ISSUE 都还来不及 Fix 就又推另一个 Release 了。进入维护模式意思就是目前一直以后一段时间 Spring Cloud Netflix 提供的服务和功能就这么多了，不再开发新的组件和功能了。以后将以维护和 Merge 分支 Full Request 为主

新组件功能将以其他替代平代替的方式实现

### SpringCloud Alibaba 带来了什么？ 

> 官网 https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md

能干什么 

- 服务限流降级：默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。

- 服务注册与发现：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。

- 分布式配置管理：支持分布式系统中的外部化配置，配置更改时自动刷新。

- 消息驱动能力：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。

- 阿里云对象存储：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。

- 分布式任务调度：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。

去哪里下载：[https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md]()

### 依赖

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```


# Rest 微服务工程构建

## 构建 cloud-consumer-order80 模块

使用 80 端口号的原因：消费者不用写端口号

> 百度百科：
>
> 80端口是为 [HTTP](https://baike.baidu.com/item/HTTP)（HyperText Transport Protocol)即[超文本传输协议](https://baike.baidu.com/item/超文本传输协议/8535513)开放的，此为上网冲浪使用次数最多的协议，主要用于 WWW（World Wide Web）即万维网传输信息的协议。
>
> 可以通过 HTTP 地址（即常说的“网址”）加“: 80”来访问网站，因为浏览网页服务默认的[端口号](https://baike.baidu.com/item/端口号/10883658)都是 80，因此==只需输入网址即可，不用输入“: 80”了==。

## RestTemplate

RestTemplate 是 Spring 提供的一种用于简单便捷访问 restful 服务的**模板类**

```java
@Configuration
public class ApplicationContextConfig
{
    @Bean
    public RestTemplate restTemplate()
    {
        return new RestTemplate();
    }
}
```

RestTemplate 中方法的三个常用参数：

`url`：REST 请求地址
`requestMap`：请求参数
`ResponseBean.class`：HTTP 响应转换被转换成的对象类型

```java
@RestController
public class OrderController {

    public static final String PaymentSrv_URL = "http://localhost:8001";

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/create")
    public CommonResult create(Payment payment) {
        //客户端用浏览器是get请求，但是底层实质发送post调用服务端8001
        return restTemplate.postForObject(PaymentSrv_URL + "/payment/create", payment, CommonResult.class);
    }


    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult getPayment(@PathVariable("id") Long id) {
        return restTemplate.getForObject(PaymentSrv_URL + "/payment/get/" + id, CommonResult.class);
    }
}
```

# ---------- 02_服务注册与发现


# 基本概念

## 服务调用关系

- *服务提供者*：暴露接口给其它微服务调用
- *服务消费者*：调用其它微服务提供的接口

> 服务 A 调用服务 B，服务 B 调用服务 C，那么服务 B 是什么角色?

提供者与消费者角色是相对的，一个服务可以同时是服务提供者和服务消费者


## 什么是服务注册与发现？

在服务注册与发现中，有一个**注册中心**。

- *服务注册*：每个服务节点在启动运行的时候，都会向注册中心注册服务，
	- 即将自己的地址信息 (ip、端口以及服务名字等) 上报给注册中心，注册中心负责将地址信息保存起来

![700](assets/Pasted%20image%2020240415174645.png)

- *服务发现*：**服务消费者**拿着服务的信息找注册中心要对方的地址信息。
	- 通常情况下，服务节点拿到地址信息之后，还会在**本地缓存**一份，保证在注册中心宕机时仍然可以正常调用服务。

![700](assets/Pasted%20image%2020240415174733.png)

- *服务健康检查*：为了保证服务地址列表中都是可用服务的地址信息，注册中心通常会通过 **心跳机制** 来检测服务是否可用，
- *不可用服务剔除*：注册中心会主动剔除不可用服务
- *服务状态变更通知*：如果服务信息发生变更，注册中心会将变更**推送**给相关联的服务，更新服务地址信息

![500](assets/Pasted%20image%2020240415175553.png)

> 调用 RPC 远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系 (服务治理概念)。在**任何 rpc 远程框架**中，都会有一个注册中心

## 为什么需要服务注册与发现？

#todo

## 常见的注册中心

比较常用的注册中心有 ZooKeeper、Eureka、Nacos，这三个都是使用 Java 语言开发，相对来说更适合 Java 技术栈

> 其他的还有像 ETCD、Consu，这里就不做介绍了。

1）严格意义上来说，*ZooKeeper* 设计之初并不是未来做注册中心的，只是前几年国内使用 Dubbo 的场景下比较喜欢使用它来做注册中心。

- 对于 CAP 理论来说，ZooKeeper 保证的是 **CP**。任何时刻对 ZooKeeper 的读请求都能得到一致性的结果，
- 但是，ZooKeeper 不保证每次请求的可用性（比如在 Leader 选举过程中或者半数以上的机器不可用的时候服务就是不可用的）

> 一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）

针对注册中心这个场景来说，重要的是可用性，**AP** 会更合适一些。ZooKeeper 更适合做分布式协调，服务注册中心就交给专业的来做吧!

2）*Eureka*，一款非常值得研究的注册中心。Eureka 是 Netflix 公司开源的一个注册中心，配套的还有 Feign、Ribbon、Zuul、Hystrix 等知名的微服务系统构建所必须的组件。

- 对于 CAP 理论来说，Eureka 保证的是 **AP**。Eureka 集群只要有一台 Eureka 正常服务，整个注册中心就是可用的，
- 只是查询到的数据可能是过期的（集群中的各个节点异步方式同步数据，不保证强一致性）

> Spring Cloud 2020.0.0 版本移除了 Netflix 除 Eureka 外的所有组件，因为在 2018 年的时候。Netflix 宣布其开源的核心组件 Hystrix、Ribbon、Zuul、Eureka 等进入维护状态，不再进行新特性开发，只修 BUG。于是，Spring 官方不得不考虑移除 Netflix 的组件。

不推荐使用 Eureka 作为注册中心，阿里开源的 Nacos 或许是更好的选择。

3）Nacos，一款即可以用来做**注册中心**，又可以用来做**配置中心**的优秀项目

- Nacos 属实是后起之秀，借鉴吸收了其他注册中心的优点，与 Spring Boot、Dubbo、Spring CloudKubernetes **无缝对接**，兼容性很好
- Nacos 不仅支持 **CP** 也支持 **AP**
- Nacos 性能强悍 (比 Eureka 能支持更多的服务实例)
- 易用性较强 (文档丰富、数据模型简单且自带后台管理界面)
- 支持 99.9% 高可用

推荐使用 Nacos 来做注册中心

## 注册中心的异同点

![image-20220818122943977](assets/02_服务注册与发现/image-20220818122943977.png)

### CAP 理论

`Consistency（一致性）`：即更新操作成功并返回客户端后，所有节点在同一时间的数据完全一致。

- 对于客户端，一致性指的是并发访问时更新过的数据如何获取的问题。
- 对于服务端，则是更新如何复制分布到整个系统，以保证数据最终一致。

`Avaliability（可用性）`：即服务一直可用，而且是正常响应时间。系统能够很好的为用户服务，不出现用户操作失败或者访问超时等用户体验不好的情况。

`Partition Tolerance（分区容错性）`：即分布式系统在遇到某节点或网络故障的时候，仍然能够对外提供满足一致性和可用性的服务。分区容错性要求应用虽然是一个分布式系统，但看上去切好像是在一个可以运转正常的整体。比如现在的分布式系统中有某一个或者几个机器宕掉了，其他剩下的机器还能够正常运转满足系统要求，对于用户而言并没有什么体验上的影响。

==CAP 理论关注粒度是数据，而不是整体系统设计的策略==

![500](assets/02_服务注册与发现/image-20220818123335449.png)

CAP 理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三 大类：

- CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。
- CP - 满足一致性，分区容忍性的系统，通常性能不是特别高。
- AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。


### AP 架构

当网络分区出现后，为了保证可用性，系统 B 可以返回旧值，保证系统的可用性。

**结论：违背了一致性 C 的要求，只满足可用性和分区容错，即 AP**

![500](assets/02_服务注册与发现/image-20220818123427722.png)

### CP 架构

当网络分区出现后，为了保证一致性，就必须拒接请求，否则无法保证一致性

**结论：违背了可用性 A 的要求，只满足一致性和分区容错，即 CP**

![500](assets/02_服务注册与发现/image-20220818123440290.png)



# Eureka 注册中心

## Eureka 介绍

Spring Cloud 封装了 Netflix 公司开发的 Eureka 模块来实现服务治理

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

在 Eureka 架构中，微服务角色有两类：

1）Eureka Server：服务端，**注册中心**

- 记录服务信息
- 心跳监控

2）Eureka Client：客户端

- Provider：服务提供者，例如案例中的 user-service
	- 注册自己的信息到 Eureka Server
	- 每隔 30 秒向 EurekaServer **发送心跳**
- consumer：服务消费者，例如案例中的 order-service 
	- 根据服务名称从 Eureka server 拉取服务列表
	- 基于服务列表做负载均衡，选中一个微服务后发起远程调用

![](assets/Pasted%20image%2020240415191728.png)

## 搭建 eureka-server

eureka-server 必须是一个独立的微服务

1）创建 cloud-eureka-server7001 模块，引入 eureka-server 依赖

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
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class, args);
    }
}
```

3）配置文件

```yml
server:  
  port: 7001  
spring:  
  application:  
    name: eureka-server7001  
eureka:  
  client:  
    service-url:  
      # EurekaServer的地址，现在是自己的地址，如果是集群，需要加上其它Server的地址  
      defaultZone: http://127.0.0.1:7001/eureka
```

测试：浏览器访问 http://localhost:7001

- instances currently registered with euraka：已经注册到 Eureka 的实例

![](assets/02_服务注册与发现/1630991672466-c5132bb3-724d-4111-9375-c3df5c264cd8.png)

## 服务注册

注册实例

- 服务消费者：cloud-consumer-order80 模块
- 服务生产者：cloud-provider-payment8001 模块

1）添加 Eureka-Client 依赖

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2）配置文件

```yml
server:  
  port: 8001  
spring:  
  application:  
    name: cloud-payment-service # 微服务的注册名称  
eureka:  
  client:  
    service-url:  
      defaultZone: http://127.0.0.1:7001/eureka  
```

```yaml
server:  
  port: 80
spring:  
  application:  
    name: cloud-order-service  
eureka:  
  client:  
    service-url:  
      defaultZone: http://localhost:7001/eureka
```

另外，可以将 cloud-payment-service 多次启动模拟多实例部署，但为了避免端口冲突，需要修改端口设置

- 再启动一个 cloud-payment-service，VM options 设置 `-Dserver.port=8082`

测试：访问 [http://localhost:7001]()，可以看到注册进来的实例

![](assets/Pasted%20image%2020240416231114.png)

## 服务发现 & 负载均衡

服务拉取：基于服务名称获取**服务列表**，然后再对服务列表做**负载均衡**

在 cloud-consumer-order80 中完成服务拉取

1）修改消费者服务调用的 url 路径，用**服务名**代替 ip、端口

```java
@RestController
public class OrderController {
    //    public static final String PaymentSrv_URL = " http://localhost:8001" ;
    // 通过在 eureka 上注册过的微服务名称调用
    public static final String PaymentSrv_URL = " http://cloud-payment-service" ;
}
```

2）使用 `@LoadBalanced` 注解赋予 RestTemplate 负载均衡的能力

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced //赋予 RestTemplate 负载均衡的能力
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

测试：访问 [http://localhost/consumer/test]()，负载均衡效果达到，8001/8002 端口交替出现

## actuator 微服务信息完善

#### 主机名称：服务名称的规范和修改

按照规范的要求只暴露服务名，不带有主机名。 

修改生产者 8001/8002 的 yml

![img](assets/02_服务注册与发现/1631704450338-fc5f7714-c957-486d-b04e-4e695aa2d235.png) ![img](assets/02_服务注册与发现/1631704456633-f75ccbc5-51e6-416e-83a0-af005591f4c9.png)

测试：仅暴露服务名

![](assets/02_服务注册与发现/1631703666885-18109b60-277c-4942-ad91-569682e434f0-166082498556021.png)

#### 访问信息有 IP 信息提示

![](assets/02_服务注册与发现/1631704600433-da87fdc9-56cc-45b2-907c-b1be857f81a6.png)

我现在点击这个微服务的链接，没有 ip 信息提示。 实际工作中，我们都会说这个微服务是部署在几号机器上面的几号端口。我们要让访问信息 有 ip 信息提示。 

在每个微服务的 yml 中添加如下

![img](assets/02_服务注册与发现/1631704693453-7470f0d0-fe04-4283-a267-78c02a129522.png)

## 服务发现 Discovery

服务发现：对于注册进 eureka 里面的微服务，可以通过服务发现来获得该服务的信息

不排除我们微服务要对外提供一种功能。那么我们就需要拿到在 eureka 上注册了的微服务的信息，例如：主机名称、端口号。

修改 cloud-provider-payment8001 的 controller，注入`DiscoveryClient`（注意写在 80 和 8002 中也都可以获得所有的微服务信息，这里只是在 8001 上进行测试）

```java
@RestController
@Slf4j
public class PaymentController {

    @Value("${server.port}")
    private String serverPort;

    @Resource
    private PaymentService paymentService;

    @Resource
    private DiscoveryClient discoveryClient;

    @GetMapping(value = "/payment/discovery")
    public Object discovery() {
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info(element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance element : instances) {
            log.info(element.getServiceId() + "\t"
                    + element.getHost() + "\t" + element.getPort() + "\t" + element.getUri());
        }
        return this.discoveryClient;
    }
}
```

8001 主启动类上添加 `@EnableDiscoveryClient`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient //服务发现
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class, args);
    }
}
```

测试： http://localhost:8001/payment/discovery

![image.png](assets/02_服务注册与发现/1631707101134-9f79116e-e133-46ef-bca6-68312f355846.png)

![image.png](assets/02_服务注册与发现/1631707370541-72b5368b-b30b-4c1a-b05d-c29ac27b7016.png)

## eureka 自我保护

#### 概述

什么是保护模式？

保护模式主要用于一组客户端和 EurekaServer 之间存在网络分区场景下的保护。一旦进入保护模式 EurekaServer 将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。 

如果在 EurekaServer 的首页看到以下这段提示，则说明 Eureka 进入了保护模式：

![img](assets/02_服务注册与发现/1631707780822-bd921fea-2ace-4224-a1ca-1c6f8ae7dcf7.png)

综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让 Eureka 集群更加的健壮、稳定。

一句话：某时刻某一个微服务不可用了，Eureka 不会立刻清理，依旧会对该微服务的信息进行保存。



为什么会产生 Eureka 自我保护机制？

因为可能存在这样的情况： EurekaClient 可以正常运行，但是与 EurekaServer 网络不通。 此时 EurekaServer 不会立刻将 EurekaClient 服务剔除。 

默认情况下，如果 EurekaServer 在一定时间内没有接收到某个微服务实例的心跳，EurekaServer 将会注销该实例（默认 90 秒）。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与 EurekaServer 之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka 通过“自我保护模式”来解决这个问题——当 EurekaServer 节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。



属于 CAP 里面的 AP 分支。



#### 禁止自我保护？

##### 修改注册中心 eurekaServer 端 7001

出厂默认，自我保护机制是开启的，`eureka.server.enable-self-preservation=true`

修改 yml

```yml
eureka:
  server:
    # 关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

测试关闭效果：访问 7001

<img src="assets/02_服务注册与发现/image-20220818203551421.png" alt="image-20220818203551421" style="zoom:67%;" />



##### 修改生产者客户端 eurekaClient 端 8001

默认：

```yml
eureka.instance.lease-renewal-interval-in-seconds=30
eureka.instance.lease-expiration-duration-in-seconds=90
```

修改 yml

```yml
eureka:
  instance:
    instance-id: payment8001
    prefer-ip-address: true # 访问路径可以显示 IP 地址
    # Eureka 客户端向服务端发送心跳的时间间隔，单位为秒(默认是 30 秒)
    lease-renewal-interval-in-seconds: 1
    # Eureka 服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是 90 秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

# Ribbon 负载均衡

Spring Cloud Ribbon 是基于 Netflix Ribbon 实现的一套客户端**负载均衡**的工具

负载均衡流程

![](assets/Pasted%20image%2020240416233649.png)

> Ribbon 负载均衡 VS Nginx 负载均衡？
> 
> - Nginx 是服务器负载均衡，客户端所有请求都会交给 nginx，然后由 nginx 实现转发请求。即负载均衡是由==服务端实现==的。
> - Ribbon 本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到 JVM 本地，从而在==本地实现== RPC 远程服务调用技术

## 原理

> 源码跟踪 #todo

SpringCloudRibbon 的底层采用了一个拦截器，拦截了 RestTemplate 发出的请求，对地址做了修改

基本流程如下：

- 拦截我们的 RestTemplate 请求 [http://userservice/user/1](http://userservice/user/1)
- RibbonLoadBalancerClient 会从请求 url 中获取服务名称，也就是 user-service
- DynamicServerListLoadBalancer 根据 user-service 到 eureka 拉取服务列表
	- eureka 返回列表，localhost:8081、localhost:8082
- IRule 利用内置负载均衡规则，从列表中选择一个，例如 localhost:8081
- RibbonLoadBalancerClient 修改请求地址

![](assets/Pasted%20image%2020240417000140.png)

## 负载均衡策略

负载均衡的规则都定义在 IRule 接口中，而 IRule 有很多不同的实现类，每一个实现类都代表一个策略

![](assets/Pasted%20image%2020240417001034.png)

| **内置负载均衡规则类**             | **规则描述**                                                                                                                                                                                                                                                                |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RoundRobinRule            | 简单轮询服务列表来选择服务器。它是 Ribbon 默认的负载均衡规则。                                                                                                                                                                                                                                     |
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
    clients: cloud-payment-service # 指定对cloud-payment-service服务饥饿加载
```


# Consul

### Consul 简介 

**1、是什么：微服务的服务注册中心 [Consul官网](https://www.consul.io/docs/intro)**

Consul 是一套开源的分布式服务发现和配置管理系统，由 HashiCorp 公司采用 Go 语言开发

- 提供了微服务系统中心的服务治理，配置中心，控制总线等功能。这些功能中的每一个都可以根据需要单独使用，也可以一起使用，以构建全方位的服务网格。
- 总之 Consul 提供了一种完整的服务网格解决方案。
- 它具有很多优点：
- 
  - 包括，基于 raft 协议，比较简洁；支持健康检查，同时支持 HTTP 和 DNS 协议支持跨数据中心的 WAN 集群。
  - 提供图形界面 跨平台，支持 Linux，MAC，Windows

**2、能干嘛：The key features of Consul are:**  

- Service Discovery：服务发现，提供 HTTP 和 DNS 两种发现方式
- Health Checking: 健康检查，支持多种方式，HTTP，TCP，Docker，Shell 脚本定制化
- KV Store: Key，Value 的存储方式
- Secure Service Communication: 安全的服务交流
- Multi Datacenter:多数据中心
- 可视化 web 界面

**3、[下载页面**](https://www.consul.io/downloads)

**4、[中文文档**](https://www.springcloud.cc/spring-cloud-consul.html)



### 安装并运行 Consul 

下载 zip 完成后解压，只有一个 exe 文件。在解压目录下打开 cmd，输入`consul --version` 即可查看 consul 版本

![image.png](assets/02_服务注册与发现/1632134305155-6ee9b4de-0fff-45b6-9cec-a64398e35ddf.png)

使用开发者模式启动 `consul agent -dev`

访问可视化页面：`localhost:8500`

![image.png](assets/02_服务注册与发现/1632134497396-98976f5e-978c-452f-8e8a-4c2db48ca5b8.png)



### 生产者服务注册：consul 

新建 Module 支付服务：cloud-providerconsul-payment8006 

pom

```xml
<dependencies>
        <!--SpringCloud consul-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

yml 

```yml
###consul 服务端口号
server:
  port: 8006

spring:
  application:
    name: consul-provider-payment
  ####consul 注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        service-name: ${spring.application.name}
```

主启动类 

```java
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain8006 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8006.class, args);
    }
}
```

 业务类

```java
@RestController
@Slf4j
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @RequestMapping(value = "/payment/consul")
    public String paymentConsul() {
        return "springcloud with consul: " + serverPort + "\t" + UUID.randomUUID().toString();
    }
}
```

测试：启动8006服务，浏览器输入 [localhost:8006/payment/consul](http://localhost:8006/payment/consul)

<img src="assets/02_服务注册与发现/image-20220818121050912.png" alt="image-20220818121050912" style="zoom:50%;" />



### 消费者服务注册

新建 cloud-consumerconsul-order80	

pom

```xml
<dependencies>
    <!--SpringCloud consul-server -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-consul-discovery</artifactId>
    </dependency>
    <!-- SpringBoot整合Web组件 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--日常通用jar包配置-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

yml

```yml
###consul 服务端口号
server:
  port: 80

spring:
  application:
    name: cloud-consumer-order
  ####consul 注册中心地址
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        #hostname : 127.0.0.1
        service-name: ${spring.application.name}
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class OrderConsulMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderConsulMain80.class, args);
    }
}
```

配置类：配置 Bean  RestTemplate

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

业务类 controller

```java
@RestController
@Slf4j
public class OrderConsulController {

    public static final String INVOKE_URL = " http://consul-provider-payment" ;

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/consul")
    public String paymentInfo() {
        String result = restTemplate.getForObject(INVOKE_URL + "/payment/consul", String.class);
        log.info("消费者调用支付服务(consule)--->result:" + result);
        return result;
    }
}
```

测试：启动80服务，浏览器输入 [localhost/consumer/payment/consul](http://localhost/consumer/payment/consul)

![600](assets/02_服务注册与发现/image-20220818121623909.png)


# Nacos 服务注册和配置中心

> 对应着我们前面学的：Eureka/Consul/Zookeeper（服务注册） Config+Bus（配置中心）

## Nacos 简介

> Nacos——**Na**ming **Co**nfiguration **S**ervice

[Nacos](https://nacos.io/)是阿里巴巴的产品，现在是[SpringCloud](https://spring.io/projects/spring-cloud)中的一个组件。相比[Eureka](https://github.com/Netflix/eureka)功能更加丰富，在国内受欢迎程度较高。

- 一个更易于构建云原生应用的动态服务发现、配置管理和服务的管理平台。
- Nacos 就是注册中心 + 配置中心的组合， **Nacos = Eureka + Config + Bus**

官网文档：[https://nacos.io/zh-cn/index.html]()

### 安装并运行 Nacos

下载地址：[https://github.com/alibaba/Nacos]()

运行 bin 目录下的 startup.cmd，默认的是集群模式

```bash
# 单机模式启动
startup.cmd -m standalone
```

运行成功，访问 [http://localhost:8848/nacos]()，默认账号密码都是 nacos


## Nacos 注册中心

### 服务注册到 Nacos

> 新建生产者服务 cloudalibaba-provider-payment9001（复制一份实例，端口9002）
> 
> 新建消费者服务 cloudalibaba-consumer-nacos-order83

1）依赖

父工程管理版本

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.1.0.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

spring-cloud-alibaba-dependencies中规定了版本号

```xml
<properties>  
    <sentinel.version>1.7.1</sentinel.version>  
    <oss.version>3.1.0</oss.version>  
    <seata.version>1.0.0</seata.version>  
    <nacos.client.version>1.1.4</nacos.client.version>  
    <nacos.config.version>0.8.0</nacos.config.version>  
    <acm.version>1.0.9</acm.version>  
    <ans.version>1.0.1</ans.version>  
    <aliyun.sdk.version>4.4.1</aliyun.sdk.version>  
    <alicloud.context.version>1.0.5</alicloud.context.version>  
    <aliyun.sdk.edas.version>2.44.0</aliyun.sdk.edas.version>  
    <schedulerX.client.version>2.1.6</schedulerX.client.version>  
    <aliyun.java.sdk.dysmsapi>1.1.0</aliyun.java.sdk.dysmsapi>  
    <aliyun.sdk.mns>1.1.8.6</aliyun.sdk.mns>  
    <aliyun.java.sdk.dyvmsapi>1.1.1</aliyun.java.sdk.dyvmsapi>  
	<spring.context.support.version>1.0.5</spring.context.support.version>  
</properties>
```

需要注册的服务引入 nacos-discovery 依赖

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
</dependencies>
```

2）配置 nacos 地址

```yml
server:
  port: 9001

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      server-addr: localhost:8848 # 配置Nacos地址 ，注册到 Nacos
```

```yml
server:
  port: 83

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      server-addr: localhost:8848

service-url:
  # 消费者要访问的微服务名称
  nacos-user-service: http://nacos-payment-provider
```

3）业务类

 服务提供者：返回端口号功能

```java
@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: " + serverPort + "\t id" + id;
    }
}
```

服务消费者

```java
@RestController  
@RequestMapping("/consumer")  
@Slf4j  
public class OrderNacosController {  
    @Resource  
    private RestTemplate restTemplate;  
  
    private String serverURL = "http://nacos-payment-provider";  
  
    @GetMapping("/payment/{id}")  
    public String paymentInfo(@PathVariable("id") Long id) {  
        String result = restTemplate.getForObject(serverURL + "/payment/" + id, String.class);  
        return result;  
    }  
}
```


4）启动成功，访问 http://localhost:9001/payment/1


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

1）Nacos 服务分级存储模型

1. 一级是**服务**，例如 userservice
2. 二级是**集群**，例如 杭州或上海
3. 三级是**实例**，例如 杭州机房的某台部署了 userservice 的服务器

### 实例集群配置

如何设置实例的集群属性：修改 `application.yml` 文件，添加集群配置属性

```yml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      discovery:
        cluster-name: HZ # 集群名称
```

---

复制多个 nacos-payment-provider 实例，通过以下参数启动

```bash
-Dserver.port=9001 -Dspring.cloud.nacos.discovery.cluster-name=HZ
-Dserver.port=9002 -Dspring.cloud.nacos.discovery.cluster-name=HZ
-Dserver.port=9003 -Dspring.cloud.nacos.discovery.cluster-name=SH
```

查看控制台

![](assets/Pasted%20image%2020240417181122.png)

### NacosRule 负载均衡策略

默认的 `ZoneAvoidanceRule` 并不能实现根据同集群优先来实现负载均衡。

Nacos 中提供了一个 `NacosRule` 的实现，可以**优先从同集群中挑选实例**

在服务消费者中配置：

```yml
nacos-payment-provider:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则
```

> 将 cloudalibaba-consumer-nacos-order83 的集群也配置为 HZ，设置 NacosRule 负载均衡策略，调用生产者服务返回的端口号都是 HZ 集群的

## 权重配置

> 服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求。

默认情况下 NacosRule 是同集群内随机挑选，不会考虑机器的性能问题。

Nacos 提供了权重配置来控制访问频率，==权重越大则访问频率越高==。

直接在 nacos 控制台编辑权重即可：

![500](assets/Pasted%20image%2020240417202003.png)

## 环境隔离

Nacos 提供了 namespace 来实现环境隔离功能

- nacos 中可以有多个 namespace
- namespace 下可以有 group、service 等
- 不同 namespace 之间相互隔离，例如==不同 namespace 的服务互相不可见==

![500](assets/Pasted%20image%2020240417203257.png)

默认情况下，所有 service、data、group 都在名为 **public** 的 namespace 下

给微服务配置 namespace ，通过修改配置文件：

```yml
spring:  
  cloud:  
    nacos:  
      discovery:  
        namespace: 65527313-eef9-439e-a3e0-bf2187879a5d # namespace的id，开发环境
```

> 将 consumer 服务配置在 dev namespace 下，provider 服务配置在 public namespace 下，consumer 调用不到 provider 服务


## Nacos 和 Eureka 的异同

Nacos 的服务实例分为两种类型：

- *临时实例*（默认）：如果实例宕机超过一定时间，会从服务列表剔除
- *非临时实例*：如果实例宕机，不会从服务列表剔除，也可以叫**永久实例**

配置一个服务实例为永久实例：

```yml
 spring:  
   cloud:  
     nacos:  
       discovery:  
         ephemeral: false # 设置为非临时实例
```

Nacos 和 Eureka 整体结构类似，服务注册、服务拉取、心跳等待，但是也存在一些差异：

- 共同点
    - 都支持服务注册和服务拉取
    - 都支持服务提供者**心跳方式**做健康检测
- 区别
    - Nacos 支持服务端主动检测提供者状态：
	    - 临时实例（干儿子）采用**心跳**模式，
	    - 非临时实例（亲儿子）采用**主动检测**模式
	    - 临时实例心跳不正常会被剔除，非临时实例则不会被剔除
    - Nacos 支持服务列表变更的**消息推送**模式，服务列表更新更及时
    - CAP：
	    - Nacos 集群**默认**采用 AP 方式。当集群中存在非临时实例时，采用 CP 模式；
	    - Eureka 采用 AP 方式

![](assets/Pasted%20image%2020240417204836.png)





## Nacos 作为服务配置中心演示

### Nacos 作为配置中心——基础配置

以前我们将所有的配置信息写到了 GitHub 上，用 Config+Bus 来进行自动刷新和动态的更新。

现在我们可以直接把配置文件写进 Nacos，然后再用 Nacos 做类似于 config 这样的功能，直接从 Nacos 上抓取我们的配置信息。

#### 操作流程

建：module cloudalibaba-config-nacos-client3377

pom

```xml
<!--nacos-config-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
<!--nacos-discovery-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<!--web + actuator-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
......
```

**yml：这里需要配置两个，一个 bootstrap 和一个 application。**

原因：Nacos 同 springcloud-config 一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。

springboot 中配置文件的加载是存在优先级顺序的，bootstrap 优先级高于 application。

全局的放在 bootstrap.yml，自己的放在 application.yml

bootstrap.yml

```yml
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    # nacos 配置
    nacos:
      discovery:
        server-addr: localhost:8848 # Nacos 服务注册中心地址
      config:
        server-addr: localhost:8848 # Nacos 作为配置中心地址
        file-extension: yaml # 指定 yaml 格式的配置

# ${spring.application.name}-${spring.profile.active}.${spring.cloud.nacos.config.file-extension}
```

application.yml

```yml
spring:
  profiles:
    active: dev # 表示开发环境
```

主启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigClientMain3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigClientMain3377.class, args);
    }
}
```

业务类

```java
@RestController
@RefreshScope //支持 Nacos 的动态刷新功能。
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo() {
        return configInfo;
    }
}
```



**在 Nacos 中添加配置信息**

==Nacos 中的匹配规则：==

```
${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
```

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635684258256-cda8e9fc-3b88-42cc-86cb-080698853a3c.png" alt="image.png" style="zoom: 80%;" />

结果： nacos-config-client-dev.yaml



配置新增：

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635684133899-cb940e70-fed6-4893-9136-a1605f864ac6.png" alt="img" style="zoom: 67%;" />

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635683995494-e18c9f15-33c4-486a-ab85-a986c9810fd4.png" alt="img" style="zoom: 80%;" />



测试：

发送请求： http://localhost:3377/config/info ，

![image.png](图片/08_SpringCloud Alibaba Nacos 服务注册和配置中心/1635684417158-e8ca283d-c66e-4edd-8489-76e65673925e.png)



自带动态刷新：修改 nacos 中的 yaml 配置文件，再次调用查看配置，发现配置刷新了

![image.png](图片/08_SpringCloud Alibaba Nacos 服务注册和配置中心/1635684484023-62bb8360-f694-4ece-81c7-16bda7005a02.png)



### Nacos 作为配置中心——分类配置

#### 分布式开发中的多环境多项目管理问题

**问题 1：**

实际开发中，通常一个系统会准备

- dev 开发环境
- test 测试环境
- prod 生产环境

如何保证指定环境启动时服务能正确读取到 Nacos 上相应环境的配置文件呢？

**问题 2：**

一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境......那怎么对这些微服务配置进行管理呢？



#### Namespace+Group+Data ID 三者关系？为什么这么设计？

类似 Java 里面的 package 名和类名，最外层的 namespace 是可以用于区分部署环境的，Group 和 DataID 逻辑上区分两个目标对象。

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635685398335-e647481f-d32d-4833-b7f9-4fcbfa6bf693.png" alt="img" style="zoom:67%;" />

**默认情况：**

**Namespace=public，Group=DEFAULT_GROUP, 默认 Cluster 是 DEFAULT**

Namespace 主要用来实现隔离。比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个 Namespace，不同的 Namespace 之间是隔离的。

Group 默认是 DEFAULT_GROUP，Group 可以把不同的微服务划分到同一个分组里面去

Service 就是微服务；一个 Service 可以包含多个 Cluster（集群），Nacos 默认 Cluster 是 DEFAULT，Cluster 是对指定微服务的一个虚拟划分。比方说为了容灾，将 Service 微服务分别部署在了杭州机房和广州机房，这时就可以给杭州机房的 Service 微服务起一个集群名称（HZ），给广州机房的 Service 微服务起一个集群名称（GZ），还可以尽量让同一个机房的微服务互相调用，以提升性能。

最后是 Instance，就是微服务的实例。



#### 三种方案加载配置

##### DataID 方案

指定 spring.profile.active 和配置文件的 DataID 来使不同环境下读取不同的配置

默认空间+默认分组+新建 dev 和 test 两个 DataID

- dev 配置 DataID，上一讲配置过
- 新建 test 配置 DataID

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635687085675-43bff20e-5d0f-4140-8356-41d5774614a2.png" alt="img" style="zoom:67%;" />

这里命名空间是默认的 public，Group 也是默认的。



通过 spring.profile.active 属性就能进行多环境下配置文件的读取

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635687246140-40f3010d-2e3c-4959-a537-42d5e0f21b2e.png" alt="img" style="zoom: 80%;" />

测试：访问 http://localhost:3377/config/info ，成功读取到 test 配置下的 config.info



##### Group 方案

默认 Group 是 DEFAULT_GROUP，现在通过 Group 实现环境分区



新建一个配置文件，添加到 DEV_GROUP 分组

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635687829633-f170e566-7692-42cd-9524-eef3337aeffe.png" alt="image.png" style="zoom: 67%;" />



新建一个配置文件，添加到 TEST_GROUP 分组

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635687928881-aa121233-ef4d-4868-b2aa-3a12017af496.png" alt="image.png" style="zoom: 67%;" />

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635687983063-3062040c-8821-4fac-b868-f570612d6a36.png" alt="image.png" style="zoom: 67%;" />



在 config 下增加一条 group 的配置即可。可配置为 DEV_GROUP 或 TEST_GROUP

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635688159372-2a81a5bd-4797-4e5c-a1fb-37fca2c56aee.png" alt="image.png" style="zoom:67%;" />



测试：

![image.png](图片/08_SpringCloud Alibaba Nacos 服务注册和配置中心/1635688181202-bf3709c6-29f3-49fe-95a9-bc55925d375b.png)



##### namespace 方案

新建 dev/test 的 Namespace

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635688594593-dcfc88bc-b792-421a-a01f-0ac785be0650.png" alt="image.png" style="zoom:67%;" />



回到服务管理-服务列表查看

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635688736719-d5c116d7-f8b1-4637-8488-0174a8b463b6.png" alt="image.png" style="zoom:67%;" />



在这两个新建的 namespace 中分别新建三个不同分组的配置文件（克隆）

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635690723423-42a30b93-e4c1-42d2-b349-277dd6a068fd.png" alt="image.png" style="zoom:67%;" />



修改 3377 的 yml 文件

bootstrap：

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635689312883-38e75de9-0081-4b50-82bb-be5b057b7d41.png" alt="image.png" style="zoom:67%;" />

application：

![image.png](图片/08_SpringCloud Alibaba Nacos 服务注册和配置中心/1635689345327-7fb0d48a-aba1-4814-9c54-d1d1bfc6f7a0.png)



测试：

dev namespace

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/image-20220905175850425.png" alt="image-20220905175850425" style="zoom:67%;" />



##### 总结

> 前两条默认改成相同比较好

DataID 方案是在默认 namespace 和默认 Group 下，创建两个不同的 DataID。

Group 方案是在默认 namespace 下，新建两个 DataID 相同的配置文件，通过指定不同的分组来读取不同的配置。

Namespace 方案，是相同的 Group，相同的 DataID，创建并指定不同的 namespace 来读取不同配置。



## Nacos 集群和持久化配置（重点）

我们之前在学 eureka 的时候，配置了两个 eureka 注册中心微服务。学 Nacos 时，不用我们单独新建注册中心微服务模块了，直接安装使用即可，很方便。

但是：如果这个注册中心挂了怎么办？我们目前就开了一个 Nacos 程序，也没有配置集群。显然，实际情况中不可能只有一个 Nacos 注册中心，因此需要用到 nacos 集群。

[集群官网文档](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)



### 官网说明

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635774952980-508e21d6-48a1-4eb9-94b9-ac493e7ac4d4.png" alt="image.png" style="zoom:67%;" />

这里 vip 表示 virtual ip（虚拟 IP）



要将配置持久化到数据库中 MySQL，不用 nacos 内嵌的数据库

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635775247554-92fa7cbf-573e-4385-9efd-1ad111ae0371.png" alt="image.png" style="zoom: 80%;" />



默认 Nacos 使用嵌入式数据库实现数据的存储，我们重启 Nacos 后，以前的配置文件不会小时。

但是，如果启动多个默认配置下的 Nacos 节点，数据存储是存在一致性问题的。每个 nacos 都有自己独立的嵌入式数据库，存放的数据不一致。

为了解决这个问题，Nacos 采用了集中式存储的方式来支持集群化部署，目前只支持 MySQL 的存储。

nacos 支持的三种部署模式：

- 单机模式：用于测试和单机使用
- 集群模式：用于生产环境，确保高可用
- 多集群模式：用于多数据中心场景



### Nacos 嵌入式数据库 derby 切换到 mysql（单机版）

Nacos 默认自带的是嵌入式数据库 derby，那么如果做集群时每个 nacos 都自带一个 derby，那么就有三个存储配置稳健的数据库，显然数据的统一性存在问题。



首先在 nacos 安装目录的 conf 目录下找到一个名为 `nacos-mysql.sql` 的 sql 脚本，执行

注意：数据库 nacos_config 需要自己创建

```mysql
 
CREATE DATABASE nacos_config;
USE nacos_config;
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(255) DEFAULT NULL,
  `content` LONGTEXT NOT NULL COMMENT 'content',
  `md5` VARCHAR(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` TEXT COMMENT 'source user',
  `src_ip` VARCHAR(20) DEFAULT NULL COMMENT 'source ip',
  `app_name` VARCHAR(128) DEFAULT NULL,
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` VARCHAR(256) DEFAULT NULL,
  `c_use` VARCHAR(64) DEFAULT NULL,
  `effect` VARCHAR(64) DEFAULT NULL,
  `type` VARCHAR(64) DEFAULT NULL,
  `c_schema` TEXT,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(255) NOT NULL COMMENT 'group_id',
  `datum_id` VARCHAR(255) NOT NULL COMMENT 'datum_id',
  `content` LONGTEXT NOT NULL COMMENT '内容',
  `gmt_modified` DATETIME NOT NULL COMMENT '修改时间',
  `app_name` VARCHAR(128) DEFAULT NULL,
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';
 
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(128) NOT NULL COMMENT 'group_id',
  `app_name` VARCHAR(128) DEFAULT NULL COMMENT 'app_name',
  `content` LONGTEXT NOT NULL COMMENT 'content',
  `beta_ips` VARCHAR(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` VARCHAR(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` TEXT COMMENT 'source user',
  `src_ip` VARCHAR(20) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(128) NOT NULL COMMENT 'group_id',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` VARCHAR(128) NOT NULL COMMENT 'tag_id',
  `app_name` VARCHAR(128) DEFAULT NULL COMMENT 'app_name',
  `content` LONGTEXT NOT NULL COMMENT 'content',
  `md5` VARCHAR(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  `src_user` TEXT COMMENT 'source user',
  `src_ip` VARCHAR(20) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` BIGINT(20) NOT NULL COMMENT 'id',
  `tag_name` VARCHAR(128) NOT NULL COMMENT 'tag_name',
  `tag_type` VARCHAR(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` VARCHAR(255) NOT NULL COMMENT 'data_id',
  `group_id` VARCHAR(128) NOT NULL COMMENT 'group_id',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` BIGINT(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键 ID',
  `group_id` VARCHAR(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '配额，0 表示使用默认值',
  `usage` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0 表示使用默认值',
  `max_aggr_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0 表示使用默认值',
  `max_aggr_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0 表示使用默认值',
  `max_history_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各 Group 容量信息表';
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` BIGINT(64) UNSIGNED NOT NULL,
  `nid` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT,
  `data_id` VARCHAR(255) NOT NULL,
  `group_id` VARCHAR(128) NOT NULL,
  `app_name` VARCHAR(128) DEFAULT NULL COMMENT 'app_name',
  `content` LONGTEXT NOT NULL,
  `md5` VARCHAR(32) DEFAULT NULL,
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00',
  `src_user` TEXT,
  `src_ip` VARCHAR(20) DEFAULT NULL,
  `op_type` CHAR(10) DEFAULT NULL,
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';
 
 
/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` BIGINT(20) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键 ID',
  `tenant_id` VARCHAR(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '配额，0 表示使用默认值',
  `usage` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0 表示使用默认值',
  `max_aggr_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0 表示使用默认值',
  `max_history_count` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '创建时间',
  `gmt_modified` DATETIME NOT NULL DEFAULT '2010-05-05 00:00:00' COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';
 
 
CREATE TABLE `tenant_info` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` VARCHAR(128) NOT NULL COMMENT 'kp',
  `tenant_id` VARCHAR(128) DEFAULT '' COMMENT 'tenant_id',
  `tenant_name` VARCHAR(128) DEFAULT '' COMMENT 'tenant_name',
  `tenant_desc` VARCHAR(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` VARCHAR(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` BIGINT(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` BIGINT(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';
 
CREATE TABLE users (
    username VARCHAR(50) NOT NULL PRIMARY KEY,
    PASSWORD VARCHAR(500) NOT NULL,
    enabled BOOLEAN NOT NULL
);
 
CREATE TABLE roles (
    username VARCHAR(50) NOT NULL,
    role VARCHAR(50) NOT NULL
);
 
INSERT INTO users (username, PASSWORD, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);
 
INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```



在 conf 目录下找到 application.properties，添加

```properties
spring.datasource.platform=mysql
 
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=数据库账号
db.password=数据库密码
```



在 bin 目录下的 cmd 输入 `startup.cmd -m standalone` 执行，这里还是单机模式，只不过数据库从 derby 迁移到了 mysql。可以看到，重启之后之前的配置都没了，说明迁移成功。

在 naocs 新建配置时，会自动保存到 mysql 数据库中

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1635778680246-a20e51f7-4fd9-4c94-8196-840cd3a33a3b.png" alt="image.png" style="zoom:80%;" />



#### 1.1.4 版本 nacos 使用外部数据库 Mysql8 报错问题解决

尝试了如下方法：

1、在 nacos\plugins\mysql（自己创建）文件夹下放 mysql8 对应版本的 jar 包（不管用）

2、修改 application.properties 文件名为 bootstrap.properties：可以正常启动 nacos，但是数据库并没有从 derby 切换为 mysql

3、添加时区：

<img src="图片/08_SpringCloud Alibaba Nacos服务注册和配置中心/1636688994869-43048817-50a7-486e-90d9-c0a2d589e64e.png" alt="img" style="zoom:80%;" />



解决方案：安装 mysql5.0+版本

https://www.bbsmax.com/A/QV5ZRk375y/

报错可看: https://blog.csdn.net/WindNolose/article/details/123238309



### Linux 版 Nacos+MySQL 生产环境配置