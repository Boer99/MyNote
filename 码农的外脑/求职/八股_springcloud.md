## Spring Cloud

Spring Cloud 是**分布式微服务架构的一站式解决方案**，它提供了一套简单易用的编程模型，使我们能==在 Spring Boot 的基础上==轻松地实现微服务系统的构建。 Spring Cloud 提供以微服务为核心的分布式系统构建标准。

Spring Cloud 的主要目标是解决分布式系统中的常见问题，例如服务发现、负载均衡、配置管理、断路器、消息总线等。

Spring Cloud 本身并不是一个开箱即用的框架，它是一套微服务规范，共有两代实现。
- **Spring Cloud Netflix** 是 Spring Cloud 的第一代实现，主要由 Eureka、Ribbon、Feign、Hystrix 等组件组成。
	- Eureka：服务发现和注册中心，可以帮助服务消费者自动发现和调用服务提供者。
	- Ribbon：负载均衡组件，可以帮助客户端在多个服务提供者之间进行负载均衡。
	- OpenFeign：声明式 HTTP 客户端，可以帮助开发人员更容易地编写 HTTP 调用代码。
	- Hystrix：断路器组件，可以帮助应用程序处理服务故障和延迟问题。
	- Config：分布式配置管理组件，可以帮助应用程序从远程配置源获取配置信息。
	- Bus：消息总线组件，可以帮助应用程序实现分布式事件传递和消息广播。
	- Sleuth：Sleuth 是 Spring Cloud 生态系统中的一个分布式追踪解决方案，可以帮助开发人员实现对分布式系统中请求链路的追踪和监控。
	- Gateway：spring Cloud Gateway 是 Spring Cloud 推出的第二代网关框架，取代 Zuul 网关。提供了路由转发、权限校验、限流控制等作用。
	- Security：用于简化 OAuth2 认证和资源保护。
- **Spring Cloud Alibaba** 是 Spring Cloud 的**第二代实现**

## Spring Cloud Alibaba 有哪些组件？ #面过 

- Nacos：服务注册（发现）和配置中心
- Sentinel：熔断限流
- Seata：分布式事务
- RocketMQ：消息队列，削峰填谷
- Sidecar：异构服务接入
- SMS：阿里云短信服务
- OSS：阿里云存储服务

## nacos 怎么实现服务注册的？ #todo #面过2 

> nacos 的大致原理？ 




## HttpClient，RestTemplate，OpenFeign 的区别？

> HttpClient 和 OpenFeign 的区别？ #面过 

HttpClient、RestTemplate 和 OpenFeign 是三种常见的 Java HTTP 客户端工具，用于与外部服务进行通信

> **客户端工具**是指用于发起 HTTP 请求并处理响应的工具，而不是指使用它们的设备或应用类型。即便这些工具运行在服务器端，它们仍然充当 HTTP 客户端角色，因为它们主动向其他服务发起请求


|          | HttpClient                        | RestTemplate | OpenFeign                        |
| -------- | --------------------------------- | ------------ | -------------------------------- |
| 开发难度     | 较高                                | 较低           | 最低                               |
| 集成       | apache 推出，从 Java11 开始引入并成为标准库的一部分 | Spring 集成    | SpringCloud 集成                   |
| 声明式支持    | 无                                 | 支持           | 支持且风格统一                          |
| 性能 #todo | 高（支持异步非阻塞）                        | 一般（同步阻塞）     | 较好                               |
| 使用场景     | 高性能、高并发、复杂请求、非 Spring 项目          | 简单 REST 调用   | 与 Spring Cloud 生态结合（例如服务发现、负载均衡） |

## Ribbon 怎么实现负载均衡的？ #面过

[✅Ribbon是怎么做负载均衡的？](https://www.yuque.com/hollis666/krcpbs/umf7fkgc9purm9qb)

Ribbon 是一种客户端负载均衡的解决方案，它通常与 Spring Cloud 一起使用，以在微服务架构中实现负载均衡（OpenFeign 里自动集成了）

原理是通过注册中心，如 Nacos，将可用的服务列表拉取到本地（客户端），再通过客户端负载均衡器（设置的负载均衡策略）获取到某个服务器的具体 ip 和端口，然后再通过 Http 框架请求服务并得到结果。

![](https://cdn.nlark.com/yuque/0/2023/png/5378072/1683878166235-0fca0c8a-624e-4ec0-a906-6eed33ef387c.png)

工作流程：
- Ribbon 底层的拦截器拦截了客户端 HTTP 请求
- 根据请求 URL 将目标服务名称和 api 路径解析出来
- 从注册中心拉取可用的服务实例列表
- 根据负载均衡算法选择实例
- 修改请求地址，将 ip 替换服务名称
- 将请求发送给目标服务器