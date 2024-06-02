# 服务网关 Gateway

## Gateway概述

### Gateway是什么

Cloud全家桶中有个很重要的组件就是网关，在1.x版本中都是采用Zuul网关；

但在2.x版本中，zuul的升级就是一直跳票，SpringCloud最后自己研发了一个网关代替Zuul

那就是 SpringCloud Gateway  ，gateway是zuul 1.x版本的替代。

<img src="图片/05_服务网关 Gateway/1632809950569-9d0e8382-c673-4191-9245-299a2ff4f767.png" alt="img" style="zoom: 67%;" />

- SpringCloud Gateway 是 Spring Cloud 的一个全新项目，基于 Spring 5.0+Spring Boot 2.0 和 Project Reactor 等技术开发的网关，它旨在为微服务架构提供一种简单有效的统一的 API 路由管理方式。
- SpringCloud Gateway作为Spring cloud生态系统中的网关，目标是代替 Zuul，在SpringCloud2.0以上版本中，没有对新版本的Zuul 2.0以上实现最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是基于WebFlux框架实现的，而WebFlux框架底层则使用了高性能的Reactor模式通信框架Netty，【说穿了就是 SpringCloud Gateway是异步非阻塞式，响应式的框架】
- Spring Cloud Gateway的目标提供统一的路由方式且基于 Filter 链的方式提供了网关基本的功能，例如：安全，监控/指标，和限流。

**一句话：SpringCloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架。**



### 能干嘛

反向代理、鉴权、流量控制、熔断、日志监控等



### 微服务架构中网关在哪里

<img src="图片/05_服务网关 Gateway/1632810706535-71a5e3e9-f84c-4543-9bbe-5da3d0c070e2.png" alt="img" style="zoom:67%;" />



### 为什么选择gateway？

 一方面因为Zuul1.0已经进入了维护阶段，同时Zuul2.0疯狂跳票。而且Gateway是SpringCloud团队研发的，是亲儿子产品，值得信赖。

而且很多功能Zuul都没有用起来也非常的简单便捷。

Gateway是基于异步非阻塞模型上进行开发的，性能方面不需要担心。虽然Netflix早就发布了最新的 Zuul 2.x，

但 Spring Cloud 貌似没有整合计划。而且Netflix相关组件都宣布进入维护期；不知前景如何？

多方面综合考虑Gateway是很理想的网关选择。



#### SpringCloud Gateway具有如下特性

- 基于Spring Frameword5 ,Project Reactor 和 SpringBoot 2.0进行构建；
- 动态路由：能够匹配任何请求属性
- 可以对路由指定Predicate(断言)和Filter（过滤器）
- 集成Hystrix的断路器功能；
- 集成Spring Cloud的服务发现功能
- 易于编写的Predicate（断言）和Filter（过滤器）
- 请求限流功能；
- 支持路径重写



#### SpringCloud Gateway 与 Zuul的区别

在SpringCloud Finchley 正式版之前（现在H版），SpringCloud推荐的网关是Netflix提供的zuul。

1. Zuul1.x 是一个基于阻塞 I/O的API网关
2. Zuul1.x 基于Servlet2.5使用阻塞架构它不支持任何长连接 （如WebSocket）Zuul的设计模式和Nginx较像，每次I/O操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是Nginx用C++实现，Zuul用java实现，而JVM本身会有第一次加载较慢的情况，使得Zuul的性能相对较差。

3. Zuul 2.x理念更加先进，像基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。Zuul2.x的性能较Zuul 1.x有较大的提升。在性能方面，根据官方提供的基准测试，Spring Cloud Gateway的RPS（每秒请求次数）是Zuul的1.6倍。
4. Spring Cloud Gateway建立在Spring Framework 5、project Reactor和Spring Boot2 之上，使用非阻塞API
5. Spring Cloud Gateway 还支持WebSocket，并且与Spring紧密集成拥有更好的开发体验。



### Zuul1.x模型

Springcloud中所集成的Zuul版本，采用的是Tomcat容器，使用的是传统的Servlet IO处理模型。

Servlet的生命周期?servlet由servlet container进行生命周期管理。

container启动时构造servlet对象并调用servlet init()进行初始化；

container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用service()。

container关闭时调用servlet destory()销毁servlet；

<img src="图片/05_服务网关 Gateway/1632811268454-33791eff-eede-43f5-bb23-a3409274793a.png" alt="img" style="zoom:80%;" />

上述模式的缺点：

servlet是一个简单的网络IO模型，当请求进入servlet container时，servlet container就会为其绑定一个线程，在并发不高的场景下这种模型是适用的。但是一旦高并发(比如抽风用jemeter压)，线程数量就会上涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。在一些简单业务场景下，不希望为每个request分配一个线程，只需要1个或几个线程就能应对极大并发的请求，这种业务场景下servlet模型没有优势

所以Zuul 1.X是基于servlet之上的一个阻塞式处理模型，即spring实现了处理所有request请求的一个servlet（DispatcherServlet）并由该servlet阻塞式处理处理。所以Springcloud Zuul无法摆脱servlet模型的弊端



### gateway模型

https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-new-framework

传统的Web框架，比如说：struts2，springmvc等都是基于Servlet API与Servlet容器基础之上运行的。

但是在Servlet3.1之后有了异步非阻塞的支持。而WebFlux是一个典型非阻塞异步的框架，它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty，Undertow及支持Servlet3.1的容器上。非阻塞式+函数式编程（Spring5必须让你使用java8）

Spring WebFlux 是 Spring 5.0 引入的新的响应式框架，区别于 Spring MVC，它不需要依赖Servlet API，它是完全异步非阻塞的，并且基于 Reactor 来实现响应式流规范。



## Gateway的三大核心概念与工作流程

### 1、Route(路由)

路由是构建网关的基本模块，它由ID，目标URI（Uniform Resource Identifier，统一资源标识符），一系列的断言和过滤器组成，如果断言为true则匹配该路由。

### 2、Predicate(断言)

开发人员可以匹配Http请求中的所有内容（例如请求头或者请求参数），如果请求参数与断言相匹配则进行路由。

### 3、Filter(过滤)

指的是Spring框架中的GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。



### 总结

![img](图片/05_服务网关 Gateway/1632811965373-bba8c0f8-c6ed-4fbb-97d0-796f73410f6c.png)

- web 请求，通过一些匹配条件，定位到真正的服务节点，并在这个转发过程的前后，进行一些精细化控制
- predicate 就是我们的匹配条件
- filter：就可以理解为一个无所不能的拦截器，有了这两个元素，再加上目标的uri，就可以实现一个具体的路由了。



### 工作流程

![img](图片/05_服务网关 Gateway/1632812491354-6516ee5d-8dd1-4a81-b02e-7d3b4d6fc625.png)

- 客户端向 Spring Cloud Gateway 发出请求。然后在 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。
- Handler 再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
- 过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。
- Filter在 “pre” 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等，在 “post” 类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

**核心逻辑**：路由转发+执行过滤器链



## 入门配置

创建cloud-gateway-gateway-9527 模块

pom：做网关不需要添加  web starter  否则会报错

```xml
<dependencies>
    <!--gateway-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-gateway</artifactId>
    </dependency>
    <!--eureka-client-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ......
</dependencies>
```

yml

```yml
server:
  port: 9527

spring:
  application:
    name: cloud-gateway

eureka:
  instance:
    hostname: cloud-gateway-service
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
#      defaultZone: http://eureka7001.com:7001/eureka # 单机版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka #集群版
```

主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class GateWayMain9527 {
    public static void main(String[] args) {
        SpringApplication.run(GateWayMain9527.class, args);
    }
}
```

我们以前访问cloud-provider-payment8001中的controller方法,通过：

localhost:8001/payment/get/id  和 localhost:8001/payment/lb 就能访问到相应的方法。

现在我们不想暴露8001端口号，希望在8001外面套一层9527

**yml新增网关配置**

```yml
spring:
  application:
    name: cloud-gateway
    # ================新增===================
  cloud:
    gateway:
      routes:
        - id: payment_routh # 路由的ID，没有固定规则但要求唯一，建议配合服务名
          uri: http://localhost:8001 # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/** # 断言，路径相匹配的进行路由
        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**
```

测试：

- 添加网关前：http://localhost:8001/payment/get/31
- 添加网关后：http://localhost:9527/payment/get/31



### Gateway网关路由有两种配置方式

1. 在yml中配置——之前的方式
2. 配置类：注入 `RouteLocator` 的Bean



**案例：** 通过9527网关访问到百度新闻的网址；http://news.baidu.com/guonei

在config包下创建一个配置类 路由规则是：我现在访问/guonei，将会转发到http://news.baidu.com/guonei

```java
@Configuration
public class GatewayConfig {
    /**
     * 配置了一个id为 path_route_boer 的路由规则，
     * 当访问地址 http://localhost:9527/guonei时会自动转发到地址：http://news.baidu.com/guonei
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        // 构建一个路由器，这个routes相当于yml配置文件中的routes
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        // 路由器的id是：path_route_boer，规则是我现在访问/guonei，将会转发到http://news.baidu.com/guonei
        // 这里的id、path、uri都可以跟yml中的配置对上
        routes.route("path_route_boer",
                r -> r.path("/guonei").uri("http://news.baidu.com")).build();
        return routes.build();
    }
}
```

测试：访问 http://localhost:9527/guonei



## 通过微服务名实现动态路由

目前面临的问题：

一个路由规则仅仅只对应一个接口方法，即我们将请求地址写死了。 试想一下：在分布式集群的情况下，会有多少个主机，多少个端口，多少个接口？ 难道我们要为每一个接口都定义一个路由规则吗？

解决思路：我们前面用80调用8001和8002中的接口时，只认微服务名。访问接口时没有指定哪个端口。 那么我们在定义路由规则时也可以通过微服务名实现动态路由和负载均衡。



修改9527实现动态路由

默认情况下Gateway会根据注册中心注册的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能。

pom：之前已经添加过了

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

yml：spring.cloud.gateway.discovery.locator.enabled:true;

在添加uri的时候，**lb:// **开头代表从注册中心中获取服务，后面接的就是你需要转发到的服务名称，而且找到的服务实现负载均衡。

```yml
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh # 路由的ID，没有固定规则但要求唯一，建议配合服务名
          # uri: http://localhost:8001
          # lb://开头代表从注册中心中获取服务，后面接的就是你需要转发到的服务名称，而且找到的服务实现负载均衡。
          uri: lb://cloud-payment-service  # 匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**  # 断言，路径相匹配的进行路由

        - id: payment_routh2
          # uri: http://localhost:8001
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**
```

测试：访问 http://localhost:9527/payment/get/31，可以看到轮询访问8001/8002





## Predicate 断言的使用

说白了，==Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理==

Predicates 这是一个复数，其实有多种Predicate。 我们这里用的Predicate的是[Path]，它是路由规则的其中一个。

启动我们的gateway9527，查看控制台日志

![image-20220822222307462](图片/05_服务网关 Gateway/image-20220822222307462.png)

Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。

Spring Cloud Gateway包括许多内置的Route Predicate工厂。所有这些Predicate都与HTTP请求的不同属性匹配。多个Route Predicate工厂可以进行组合

Spring Cloud Gateway 创建 Route 对象时， 使用 RoutePredicateFactory 创建 Predicate 对象，Predicate 对象可以赋值给 Route。 Spring Cloud Gateway 包含许多内置的Route Predicate Factories。

所有这些谓词都匹配HTTP请求的不同属性。多种谓词工厂可以组合，并通过逻辑and。



### 常用的Predicate 断言

#### After Route Predicate

匹配改断言时间之后的uri请求

yml

<img src="图片/05_服务网关 Gateway/1632827933056-82dc1db5-504b-46d9-ab84-b95638a3eeff.png" alt="image.png" style="zoom: 67%;" />

获取时间格式

```java
public class TimeTest {
    public static void main(String[] args) {
        ZonedDateTime zbj = ZonedDateTime.now(); // 默认时区
        System.out.println(zbj); //2021-09-28T19:14:51.514+08:00[Asia/Shanghai]
//        ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York")); // 用指定时区获取当前时间
//        System.out.println(zny);
    }
}
```



#### Before Route Predicate

匹配改断言时间之前的uri请求



#### Between Route Predicate：

匹配改断言时间之间的uri请求

```yml
predicates:
    - Path=/payment/lb/**         # 断言，路径相匹配的进行路由
    - Between=2020-02-02T17:45:06.206+08:00[Asia/Shanghai],2020-03-25T18:59:06.206+08:00[Asia/Shanghai]
```



#### Cookie Route Predicate：

需要两个参数，一个是Cookie name，一个是正则表达式。

<img src="图片/05_服务网关 Gateway/image-20220822223626968.png" alt="image-20220822223626968" style="zoom:50%;" />

测试：

<img src="图片/05_服务网关 Gateway/image-20220822223849930.png" alt="image-20220822223849930" style="zoom:50%;" />



#### Header Route Predicate

两个参数：一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

<img src="图片/05_服务网关 Gateway/image-20220822224103086.png" alt="image-20220822224103086" style="zoom: 50%;" />



#### Host Route Predicate

只有指定主机可以访问，可以指定多个用“，”分隔开。

<img src="图片/05_服务网关 Gateway/image-20220822224344797.png" alt="image-20220822224344797" style="zoom:50%;" />

测试：

`curl http://localhost:9527/payment/lb -H "Host: java.atguigu.com"`  正确

`curl http://localhost:9527/payment/lb -H "Host: java.atguigu.net" `  错误 



#### Method Route Predicate

请求方式get/post匹配

```yml
predicates:
    - Path=/payment/lb/**
    - Method=GET
```



#### Path Route Predicate

用过了



#### Query Route Predicate

请求参数匹配

支持传入两个参数，一个是属性名，一个为属性值，属性值可以是正则表达式

```yml
predicates:
    - Path=/payment/lb/** 
    - Query=username, \d+  # 要有参数名username并且值还要是整数才能路由
```



## Filter的使用

Filter指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。 

Filter链：同时满足一系列的过滤链。

路由过滤器可用于修改进入的HTTP请求和返回的HTTP响应，路由过滤器只能指定路由进行使用。Spring Cloud Gateway 内置了多种路由过滤器，他们都由GatewayFilter的工厂类来产生。

生命周期 ： 

- pre  在业务逻辑之前
- post  在业务逻辑之后

种类：

- 单一的：GatewayFilter
- 全局的：GlobalFilter



常用的GatewayFilter：AddRequestParameter 等

![image-20220822232318161](图片/05_服务网关 Gateway/image-20220822232318161.png)



### 自定义全局过滤器（GlobalFilter）

作用：全局日志记录、统一网关鉴权 ......

自定义过滤器实现两个接口 GlobalFilter , Ordered

```java
package com.boer.springcloud.filter;

@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("*********************come in MyLogGateWayFilter:  " + new Date());
        //取出请求参数的uname对应的值
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        //如果uname为空，就直接过滤掉，不走路由
        if (uname == null) {
            log.info("************* 用户名为Null 非法用户 o(╥﹏╥)o");
            //判断该请求不通过时：给一个回应，返回
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }

        //反之，调用下一个过滤器，也就是放行：在该环节判断通过的exchange放行，交给下一个filter判断
        return chain.filter(exchange);
    }

    /**
     * 这个过滤器的加载顺序，数字越小，优先级越高
     * 设置这个过滤器在Filter链中的加载顺序。
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

测试：http://localhost:9527/payment/get/31?uname=boer



