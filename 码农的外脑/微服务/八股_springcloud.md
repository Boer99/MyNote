# Spring Cloud，Spring Cloud Alibaba？

Spring Cloud 是**分布式微服务架构的一站式解决方案**，它提供了一套简单易用的编程模型，使我们能在 Spring Boot 的基础上轻松地实现微服务系统的构建。 Spring Cloud 提供以微服务为核心的分布式系统构建标准。

Spring Cloud 本身并不是一个开箱即用的框架，它是一套微服务规范，共有两代实现。
- **Spring Cloud Netflix** 是 Spring Cloud 的第一代实现，主要由 Eureka、Ribbon、Feign、Hystrix 等组件组成。
	- Eureka：服务发现和注册中心，可以帮助服务消费者自动发现和调用服务提供者。
	- Ribbon：负载均衡组件，可以帮助客户端在多个服务提供者之间进行负载均衡。
	- OpenFeign：声明式 HTTP 客户端，可以帮助开发人员更容易地编写 HTTP 调用代码。
	- Hystrix：断路器组件，可以帮助应用程序处理服务故障和延迟问题。
	- Config：分布式配置管理组件，可以帮助应用程序从远程配置源获取配置信息。
	- Bus：消息总线组件，可以帮助应用程序实现分布式事件传递和消息广播。
	- Sleuth：Sleuth是Spring Cloud生态系统中的一个分布式追踪解决方案，可以帮助开发人员实现对分布式系统中请求链路的追踪和监控。
	- Gateway：spring Cloud Gateway是Spring Cloud推出的第二代网关框架，取代Zuul网关。提供了路由转发、权限校验、限流控制等作用。
	- Security：用于简化 OAuth2 认证和资源保护。
- **Spring Cloud Alibaba** 是 Spring Cloud 的**第二代实现**，主要组件：
	- Nacos：服务注册（发现）和配置中心
	- Sentinel：熔断限流
	- Seata：分布式事务
	- RocketMQ：消息队列，削峰填谷
	- Sidecar：异构服务接入
	- SMS：阿里云短信服务
	- OSS：阿里云存储服务

