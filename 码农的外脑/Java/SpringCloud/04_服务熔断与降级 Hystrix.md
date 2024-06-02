# 服务熔断与降级 Hystrix

> Hystrix官网
>
> https://github.com/Netflix/Hystrix/wiki/How-To-Use

## 概述

### 分布式系统面临的问题

复杂分布式体系结构中的应用程序有数十个依赖关系，每个依赖关系在某些时候将不可避免地失败。

**服务雪崩：**

多个微服务之间调用的时候，假设微服务A调用微服务B和微服务C，微服务B和微服务C又调用其它的微服务，这就是所谓的“扇出”。如果扇出的链路上某个微服务的调用响应时间过长或者不可用，对微服务A的调用就会占用越来越多的系统资源，进而引起系统崩溃，所谓的“雪崩效应”.

对于高流量的应用来说，单一的后端依赖可能会导致所有服务器上的所有资源都在几秒钟内饱和。比失败更糟糕的是，这些应用程序还可能导致服务之间的延迟增加，备份队列，线程和其他系统资源紧张，导致整个系统发生更多的级联故障。这些都表示需要对故障和延迟进行隔离和管理，以便单个依赖关系的失败，不能取消整个应用程序或系统。

所以，通常当你发现一个模块下的某个实例失败后，这时候这个模块依然还会接收流量，然后这个有问题的模块还调用了其他的模块，这样就会发生级联故障，或者叫雪崩。



### Hystrix是什么？

Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。



### Hystrix能做什么

- 服务降级
- 服务熔断
- 接近实时的监控
- ......



## Hystrix 重要概念

### 服务降级：fallback

加入对方的系统不可用了，需要一个兜底的解决方案或备选响应；向调用方返回一个符合预期的、可处理的备选响应。

服务器繁忙，请稍后再试，不让客户端等待并立刻返回一个好友提示。`fallback`

哪些情况会触发降级：

- 程序运行异常
- 超时
- 服务熔断触发服务降级
- 线程池/信号量打满也会导致服务降级

### 服务熔断：break

类比保险丝达到最大服务访问后，直接拒绝访问，拉闸限电，然后调用服务降级的方法并返回友好提示 break

就是保险丝   ||    服务的降级--->进而熔断--->恢复调用链路

### 服务限流：flowlimit

秒杀高并发等操作，严禁一窝蜂的过来拥挤，大家排队，一秒钟N个，有序进行  flowlimit



## Hystrix案例

### 构建模块

#### 构建服务提供者模块 cloud-provider-hystrix-payment8001

pom

```xml
<dependencies>
        <!--hystrix-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
        <!--eureka client-->
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
  port: 8001

spring:
  application:
    name: cloud-provider-hystrix-payment

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      #集群版
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

主启动类

```java
@SpringBootApplication
@EnableEurekaClient
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}
```

service

```java
@Service
public class PaymentService {
    /**
     * 正常访问，一定ok的方法
     */
    public String paymentInfo_Ok(Integer id) {
        return "线程池：  " + Thread.currentThread().getName() + "   paymentInfo_OK,  id:  " + id
                + "\t"+"O(∩_∩)O哈哈~";
    }

    public String paymentInfo_Timeout(Integer id) {
        int timeNumber = 3;
        try {
            TimeUnit.SECONDS.sleep(timeNumber);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        return "线程池：  " + Thread.currentThread().getName() + "   payment_Timeout,  id:  " + id
                + "\t"+"O(∩_∩)O哈哈~耗时(s):" + timeNumber;
    }
}
```

controller

```java
@RestController
@Slf4j
public class PaymentController {
    @Resource
    private PaymentService paymentService;
    @Value("${server.port}")
    private String serverPort;

    @GetMapping("/payment/hystrix/ok/{id}")
    private String paymentInfo_Ok(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_Ok(id);
        log.info("******ok****result：" + result);
        return result;
    }

    @GetMapping("/payment/hystrix/timeout/{id}")
    private String paymentInfo_Timeout(@PathVariable("id") Integer id) {
        String result = paymentService.paymentInfo_Timeout(id);
        log.info("******timeout****result：" + result);
        return result;
    }
}
```



#### 新建消费者模块 cloud-consumer-feign-hystrix-order80

pom

```xml
<dependencies>
    <!--openfeign-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <!--hystrix-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    <!--eureka client-->
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
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```

主启动类

```java
@SpringBootApplication
@EnableFeignClients //不作为Eureak客户端了，而是作为Feign客户端
public class OrderFeignHystrixMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeignHystrixMain80.class, args);
    }
}
```

service

```java
@Component
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT")
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_Ok(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_Timeout(@PathVariable("id") Integer id);
}	
```

controller

```java
@RestController
@Slf4j
public class OrderHystirxController {
    @Resource
    PaymentHystrixService paymentHystrixService;

    @GetMapping("/consumer/payment/hystrix/ok/{id}")
    public String paymentInfo_Ok(@PathVariable("id") Integer id) {
        return paymentHystrixService.paymentInfo_Ok(id);
    }

    @GetMapping("/consumer/payment/hystrix/timeout/{id}")
    public String paymentInfo_Timeout(@PathVariable("id") Integer id) {
        return paymentHystrixService.paymentInfo_Timeout(id);
    }
}
```



### Jmeter高并发压测

上述在非高并发情况下，还勉强可以满足。使用jmeter，设置20000个并发压死8001，20000个请求都去访问http://localhost:8001/payment/hystrix/timeout/31

<img src="图片/04_服务熔断与降级 Hystrix/1632296071791-701b786a-5d25-4c21-b5a9-c545ac74c8c6-16610835281885.png" alt="img" style="zoom: 67%;" />

<img src="图片/04_服务熔断与降级 Hystrix/1632296085411-98b8ca31-801c-4fc9-a5c2-8c68355dc6a4-16610835281897.png" alt="img" style="zoom:67%;" />

结果发现原来可以迅速响应的http://localhost:8001/payment/hystrix/ok/31 变得卡顿。两个都在转圈。

**原因：**

因为ok跟timeout都在一个微服务里，现在大量的资源被timeout占用，微服务必须集中资源去处理这些高并发的请求。那么ok方法，得到的资源就少，进而被影响。

SpringBoot默认集成的是tomcat容器，里面有一个tomcat的线程池，高并发下没有多余的线程来分解压力和处理ok方法。 每个线程都要等待3秒钟，才能拿到timeout的结果响应。那么2000个线程一起过来， Tomcat池子里的线程，马上就会被抢占完。



### Hystrix如何解决问题？

- 超时导致服务器变慢(转圈)：超时不再等待
- 出错(宕机或程序运行出错)：出错要有兜底
- 解决

- - 对方服务(8001)超时了，调用者(80)不能一直卡死等待，必须有服务降级
  - 对方服务(8001)down机了，调用者(80)不能一直卡死等待，必须有服务降级
  - 对方服务(8001)OK，调用者(80)自己出故障或有自我要求（自己的等待时间小于服务提供者），自己处理降级



## 服务降级

不管是在消费者，还是提供者，都可以进行服务降级，使用过**@HystrixCommand**注解指定降级后的方法

一般服务降级 是放在客户端

### 服务降级：8001生产者服务端 （服务端）

8001先从自身找问题，设置自身调用超时时间的峰值，峰值内可以正常运行。超过了就需要有兜底的方法，做服务降级fallback。向调用方法返回一个符合预期的，可以处理的备选响应（FallBack）



业务类(Service)方法上添加 `@HystrixCommand`，设置超时时间和兜底方法

```java
/**
 * 模拟超时
 */
//规定这个线程的超时时间是3s，3s后就由fallbackMethod指定的方法帮我“兜底”（服务降级）
@HystrixCommand(fallbackMethod = "paymentInfo_TimeoutHandler", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
})
public String paymentInfo_Timeout(Integer id) {
    int timeNumber = 3;
    // int age = 10 / 0;
    try {
        TimeUnit.SECONDS.sleep(timeNumber);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "线程池：  " + Thread.currentThread().getName() + "   payment_Timeout,  id:  " + id
            + "\t" + "O(∩_∩)O哈哈~耗时(s):" + timeNumber;
}

/**
 * 兜底方法
 */
public String paymentInfo_TimeoutHandler(Integer id) {
    return "线程池：" + Thread.currentThread().getName() + " , paymentInfo_TimeoutHandler,  id: " + id
            + "\t" + "o(╥﹏╥)o";
}
```

主启动类添加新注解 `@EnableCircuitBreaker`

```java
@SpringBootApplication
@EnableEurekaClient
@EnableCircuitBreaker
public class PaymentHystrixMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentHystrixMain8001.class, args);
    }
}
```

测试：我们故意制造的两个异常

1. int age = 10/0 计算异常 
2. 我们定义超时时间3s，服务运行5s的超时异常 

这些都会造成当前服务不可用了，就会进行服务降级，执行兜底的方案。



### 服务降级：80消费者服务端（客户端）

yml

```yml
feign:
  hystrix:
    enabled: true
```

主启动类添加 `@EnableHystrix`

Controller方法上添加 `@HystrixCommand`，设置超时时间和兜底方法

```java
@GetMapping("/consumer/payment/hystrix/timeout/{id}")
@HystrixCommand(fallbackMethod = "paymentTimeOutFallbackMethod", commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "1500")
})
public String paymentInfo_Timeout(@PathVariable("id") Integer id) {
    return paymentHystrixService.paymentInfo_Timeout(id);
}

/**
 * 兜底方法
 */
public String paymentTimeOutFallbackMethod(@PathVariable("id") Integer id) {
    return "我是消费者80,对方支付系统繁忙请10秒钟后再试或者自己运行出错请检查自己,o(╥﹏╥)o";
}
```

测试



### 进一步优化

目前存在的问题——代码臃肿、耦合度高

- 如果每个接口，每个方法都需要一个“兜底”方法，那么就会造成代码臃肿
- 我们现在服务降级的方法和业务处理的方法混杂在了一块，耦合度很高。

所以我们需要一个全局的服务降级：global fallback；

需要特殊照顾的方法，我们再进行精确的配置服务降级。



#### 解决代码臃肿

除了个别重要核心业务有专属，其它普通的可以通过@DefaultProperties(defaultFallback = "")  统一跳转到统一处理结果页面

以消费者服务端80为例：cloud-consumer-feign-hystrix-order80

- 在业务类Controller上加 `@DefaultProperties(defaultFallback = "method_name")` 注解，并指明`defaultFallback`
- ==在需要服务降级的方法上标注 `@HystrixCommand`  注解==

- - 如果@HystrixCommand里没有指明fallbackMethod，就默认使用全局降级服务

```java
@RestController
@Slf4j
// 配置全局的fallback兜底方法
@DefaultProperties(defaultFallback = "payment_Global_FallbackMethod")
public class OrderHystirxController {

    @Resource
    PaymentHystrixService paymentHystrixService;

    /**
     * 测试全局兜底方法
     */
    @GetMapping("/consumer/payment/hystrix/TestDefaultProperties")
    @HystrixCommand //加了@DefaultProperties属性注解，并且没有写具体方法名字，就用统一全局的
    public String TestDefaultProperties() {
        int a = 10 / 0;
        return "TestDefaultProperties";
    }

    /**
     * 全局兜底方法
     */
    public String payment_Global_FallbackMethod() {
        return "Global异常处理信息，请稍后再试，/(ㄒoㄒ)/~~";
    }
}
```

测试：localhost/consumer/payment/hystrix/TestDefaultProperties



#### 降低代码耦合度

修改cloud-consumer-feign-hystrix-order80，分离业务方法和服务降级方法

根据cloud-consumer-feign-hystrix-order80已经有的PaymentHystrixService接口，重新新建一个类(PaymentFallbackService)实现该接口，统一为接口里面的方法进行异常处理。

```java
@Component  
public class PaymentFallbackService implements PaymentHystrixService {

    @Override
    public String paymentInfo_Ok(Integer id) {
        return "------PaymentFallbackService-paymentInfo_Ok, fallback";
    }

    @Override
    public String paymentInfo_Timeout(Integer id) {
        return "------PaymentFallbackService-paymentInfo_Timeout, fallback";
    }
}
```

修改PaymentHystrixService接口的 @FeignClient，指定其服务异常处理类

```java
@Component
// @FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT")
@FeignClient(value = "CLOUD-PROVIDER-HYSTRIX-PAYMENT", fallback = PaymentFallbackService.class)
public interface PaymentHystrixService {
    @GetMapping("/payment/hystrix/ok/{id}")
    public String paymentInfo_Ok(@PathVariable("id") Integer id);

    @GetMapping("/payment/hystrix/timeout/{id}")
    public String paymentInfo_Timeout(@PathVariable("id") Integer id);
}
```

确认yml配置文件中开启了hystrix

```yml
feign:
  hystrix:
    enabled: true  #在Feign中开启Hystrix
```

测试：启动7001、7002（我这里用了集群）、80，这里不启动8001服务，访问localhost/consumer/payment/hystrix/ok/8



## 服务熔断

熔断也会出发服务降级。



### 熔断机制概述

熔断机制是应对雪崩效应的一种微服务链路保护机制。当扇出链路的某个微服务出错不可用或者响应时间太长时，
会进行服务的降级，进而熔断该节点微服务的调用，快速返回错误的响应信息。

当检测到该节点微服务调用响应正常后，恢复调用链路。

在Spring Cloud框架里，熔断机制通过Hystrix实现。

Hystrix会监控微服务间调用的状况，当失败的调用到一定阈值，缺省是5秒内20次调用失败，就会启动熔断机制。熔断机制的注解是 `@HystrixCommand`



### Hystrix服务熔断实操

修改cloud-provider-hystrix-payment8001模块

对8001服务端的Service层进行改造 增加熔断机制

```java
@Service
public class PaymentService {
    /**
     * 服务熔断
     */
    @HystrixCommand(fallbackMethod = "paymentCircuitBreaker_fallback", commandProperties = {
            @HystrixProperty(name = "circuitBreaker.enabled",    value = "true"), //是否开启断路器
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),  //最小请求次数
            @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), //短路多久以后开始尝试是否恢复，默认5s
            @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60")  //失败率达到多少后跳闸
    })
    public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
        if (id < 0) {
            throw new RuntimeException("******id 不能负数");
        }
        String serialNumber = IdUtil.simpleUUID();  // 等价于UUID.randomUUID().toString()
        return Thread.currentThread().getName() + "\t" + "调用成功，流水号: " + serialNumber;
    }

    /**
     * 服务熔断的兜底方法
     */
    public String paymentCircuitBreaker_fallback(@PathVariable("id") Integer id) {
        return "id 不能负数，请稍后再试，/(ㄒoㄒ)/~~   id: " + id;
    }

}
```

controller

```java
//===================服务熔断====================
@GetMapping("/payment/circuit/{id}")
public String paymentCircuitBreaker(@PathVariable("id") Integer id) {
    String result = paymentService.paymentCircuitBreaker(id);
    log.info("****result: " + result);
    return result;
}
```

测试：

启动7001、7002（我这里启动了Eureka集群）和8001

访问 http://localhost:8001/payment/circuit/-5  ，达到10次/10s以上，在10s内执行了超过10*0.6 = 6次的异常请求，触发了Hystrix的断路器，即使传入正确的参数 http://localhost:8001/payment/circuit/5，也出现服务降级

![image.png](图片/04_服务熔断与降级 Hystrix/1632742150128-db56f776-0b6f-4ee6-bc55-5da7edd4645f.png)



### Hystrix服务熔断总结

#### 熔断类型

- 熔断打开：请求不再进行调用当前服务，再有请求调用时将不会调用主逻辑，而是直接调用降级fallback。实现了自动的发现错误并将降级逻辑切换为主逻辑，减少响应延迟效果。内部设置时钟一般为MTTR（Mean time to repair，平均故障处理时间)，当打开时长达到所设时钟则进入半熔断状态。
- 熔断关闭：熔断关闭不会对服务进行熔断，服务正常调用
- 熔断半开：部分请求根据规则调用当前服务，如果请求成功且符合规则则认为当前服务恢复正常，关闭熔断



#### 断路器在什么情况下开始起作用

涉及到断路器的四个重要参数：快照时间窗、请求总数阀值、窗口睡眠时间、错误百分比阀值。

![image-20220822160124301](图片/04_服务熔断与降级 Hystrix/image-20220822160124301.png)

- **快照时间窗（滚动窗口）**：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。  
  - `metrics.rollingStats.timeInMilliseconds`

- **请求总数阀值**：在快照时间窗内，必须满足请求总数阀值才有资格熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用次数不足20次，即使所有的请求都超时或其他原因失败，断路器都不会打开。
  - `circuitBreaker.requestVolumeThreshold`

- **窗口睡眠时间**：剩下一个表示窗口睡眠时间，即断路器触发多少秒（默认5s）后尝试恢复，进入半开状态。
  - `circuitBreaker.sleepWindowInMilliseconds`

- **错误百分比阀值**：当请求总数在快照时间窗内超过了阀值，比如发生了30次调用，如果在这30次调用中，有15次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%阀值情况下，这时候就会将断路器打开。
  - `circuitBreaker.errorThresholdPercentage`




#### 断路器开启或关闭条件

1、当满足一定的阈值的时候（默认10秒内超过20个请求次数）；

2、当失败率达到一定的时候（默认10秒内超过50%的请求失败）；

3、到达以上阈值，断路器将会开启；

4、当开启的时候，所有请求都不会进行转发；

5、一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发；如果成功，断路器会关闭，若失败，继续开启。重复4和5。



#### 断路器打开之后

##### 再有请求调用的时候，还会调用主逻辑吗？

将不会调用主逻辑，而是直接调用降级的fallback方法，通过断路器，实现了自动的发现错误并将降级逻辑升级为主逻辑，减少响应延迟的效果。

##### 原来的主逻辑要如何恢复？

对于这一问题mhystrix也为我们实现了自动恢复功能。

当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑。

当休眠时间窗到期，断路器将进入半开状态，释放给一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合。

主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时。



#### 所有的配置

```java
//========================All
@HystrixCommand(fallbackMethod = "str_fallbackMethod",
        groupKey = "strGroupCommand",
        commandKey = "strCommand",
        threadPoolKey = "strThreadPool",

        commandProperties = {
                // 设置隔离策略，THREAD 表示线程池 SEMAPHORE：信号池隔离
                @HystrixProperty(name = "execution.isolation.strategy", value = "THREAD"),
                // 当隔离策略选择信号池隔离的时候，用来设置信号池的大小（最大并发数）
                @HystrixProperty(name = "execution.isolation.semaphore.maxConcurrentRequests", value = "10"),
                // 配置命令执行的超时时间
                @HystrixProperty(name = "execution.isolation.thread.timeoutinMilliseconds", value = "10"),
                // 是否启用超时时间
                @HystrixProperty(name = "execution.timeout.enabled", value = "true"),
                // 执行超时的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnTimeout", value = "true"),
                // 执行被取消的时候是否中断
                @HystrixProperty(name = "execution.isolation.thread.interruptOnCancel", value = "true"),
                // 允许回调方法执行的最大并发数
                @HystrixProperty(name = "fallback.isolation.semaphore.maxConcurrentRequests", value = "10"),
                // 服务降级是否启用，是否执行回调函数
                @HystrixProperty(name = "fallback.enabled", value = "true"),
                // 是否启用断路器
                @HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
                // 该属性用来设置在滚动时间窗中，断路器熔断的最小请求数。例如，默认该值为 20 的时候，
                // 如果滚动时间窗（默认10秒）内仅收到了19个请求， 即使这19个请求都失败了，断路器也不会打开。
                @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "20"),
                // 该属性用来设置在滚动时间窗中，表示在滚动时间窗中，在请求数量超过
                // circuitBreaker.requestVolumeThreshold 的情况下，如果错误请求数的百分比超过50,
                // 就把断路器设置为 "打开" 状态，否则就设置为 "关闭" 状态。
                @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "50"),
                // 该属性用来设置当断路器打开之后的休眠时间窗。 休眠时间窗结束之后，
                // 会将断路器置为 "半开" 状态，尝试熔断的请求命令，如果依然失败就将断路器继续设置为 "打开" 状态，
                // 如果成功就设置为 "关闭" 状态。
                @HystrixProperty(name = "circuitBreaker.sleepWindowinMilliseconds", value = "5000"),
                // 断路器强制打开
                @HystrixProperty(name = "circuitBreaker.forceOpen", value = "false"),
                // 断路器强制关闭
                @HystrixProperty(name = "circuitBreaker.forceClosed", value = "false"),
                // 滚动时间窗设置，该时间用于断路器判断健康度时需要收集信息的持续时间
                @HystrixProperty(name = "metrics.rollingStats.timeinMilliseconds", value = "10000"),
                // 该属性用来设置滚动时间窗统计指标信息时划分"桶"的数量，断路器在收集指标信息的时候会根据
                // 设置的时间窗长度拆分成多个 "桶" 来累计各度量值，每个"桶"记录了一段时间内的采集指标。
                // 比如 10 秒内拆分成 10 个"桶"收集这样，所以 timeinMilliseconds 必须能被 numBuckets 整除。否则会抛异常
                @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "10"),
                // 该属性用来设置对命令执行的延迟是否使用百分位数来跟踪和计算。如果设置为 false, 那么所有的概要统计都将返回 -1。
                @HystrixProperty(name = "metrics.rollingPercentile.enabled", value = "false"),
                // 该属性用来设置百分位统计的滚动窗口的持续时间，单位为毫秒。
                @HystrixProperty(name = "metrics.rollingPercentile.timeInMilliseconds", value = "60000"),
                // 该属性用来设置百分位统计滚动窗口中使用 “ 桶 ”的数量。
                @HystrixProperty(name = "metrics.rollingPercentile.numBuckets", value = "60000"),
                // 该属性用来设置在执行过程中每个 “桶” 中保留的最大执行次数。如果在滚动时间窗内发生超过该设定值的执行次数，
                // 就从最初的位置开始重写。例如，将该值设置为100, 滚动窗口为10秒，若在10秒内一个 “桶 ”中发生了500次执行，
                // 那么该 “桶” 中只保留 最后的100次执行的统计。另外，增加该值的大小将会增加内存量的消耗，并增加排序百分位数所需的计算时间。
                @HystrixProperty(name = "metrics.rollingPercentile.bucketSize", value = "100"),
                // 该属性用来设置采集影响断路器状态的健康快照（请求的成功、 错误百分比）的间隔等待时间。
                @HystrixProperty(name = "metrics.healthSnapshot.intervalinMilliseconds", value = "500"),
                // 是否开启请求缓存
                @HystrixProperty(name = "requestCache.enabled", value = "true"),
                // HystrixCommand的执行和事件是否打印日志到 HystrixRequestLog 中
                @HystrixProperty(name = "requestLog.enabled", value = "true"),
        },
        threadPoolProperties = {
                // 该参数用来设置执行命令线程池的核心线程数，该值也就是命令执行的最大并发量
                @HystrixProperty(name = "coreSize", value = "10"),
                // 该参数用来设置线程池的最大队列大小。当设置为 -1 时，线程池将使用 SynchronousQueue 实现的队列，
                // 否则将使用 LinkedBlockingQueue 实现的队列。
                @HystrixProperty(name = "maxQueueSize", value = "-1"),
                // 该参数用来为队列设置拒绝阈值。 通过该参数， 即使队列没有达到最大值也能拒绝请求。
                // 该参数主要是对 LinkedBlockingQueue 队列的补充,因为 LinkedBlockingQueue
                // 队列不能动态修改它的对象大小，而通过该属性就可以调整拒绝请求的队列大小了。
                @HystrixProperty(name = "queueSizeRejectionThreshold", value = "5"),
        }
)
public String strConsumer() {
    return "hello 2020";
}

public String str_fallbackMethod(){
    return "*****fall back str_fallbackMethod";
}
```



## 服务限流

在spring cloud alibaba Sentinel时再说！



## Hystrix工作流程

> 官网地址：https://github.com/Netflix/Hystrix/wiki/How-it-Works

<img src="图片/04_服务熔断与降级 Hystrix/1632753962812-5995b053-e5d5-438a-8310-190ccaff6852.png" alt="image.png" style="zoom:80%;" />

|      |                                                              |
| ---- | ------------------------------------------------------------ |
| 1    | 创建 HystrixCommand（用在依赖的服务返回单个操作结果的时候） 或 HystrixObserableCommand（用在依赖的服务返回多个操作结果的时候） 对象。 |
| 2    | 命令执行。其中 HystrixComand 实现了下面前两种执行方式；而 HystrixObservableCommand 实现了后两种执行方式：execute()：同步执行，从依赖的服务返回一个单一的结果对象， 或是在发生错误的时候抛出异常。queue()：异步执行， 直接返回 一个Future对象， 其中包含了服务执行结束时要返回的单一结果对象。observe()：返回 Observable 对象，它代表了操作的多个结果，它是一个 Hot Obserable（不论 "事件源" 是否有 "订阅者"，都会在创建后对事件进行发布，所以对于 Hot Observable 的每一个 "订阅者" 都有可能是从 "事件源" 的中途开始的，并可能只是看到了整个操作的局部过程）。toObservable()： 同样会返回 Observable 对象，也代表了操作的多个结果，但它返回的是一个Cold Observable（没有 "订阅者" 的时候并不会发布事件，而是进行等待，直到有 "订阅者" 之后才发布事件，所以对于 Cold Observable 的订阅者，它可以保证从一开始看到整个操作的全部过程）。 |
| 3    | 若当前命令的请求缓存功能是被启用的， 并且该命令缓存命中， 那么缓存的结果会立即以 Observable 对象的形式 返回。 |
| 4    | 检查断路器是否为打开状态。如果断路器是打开的，那么Hystrix不会执行命令，而是转接到 fallback 处理逻辑（第 8 步）；如果断路器是关闭的，检查是否有可用资源来执行命令（第 5 步）。 |
| 5    | 线程池/请求队列/信号量是否占满。如果命令依赖服务的专有线程池和请求队列，或者信号量（不使用线程池的时候）已经被占满， 那么 Hystrix 也不会执行命令， 而是转接到 fallback 处理逻辑（第8步）。 |
| 6    | Hystrix 会根据我们编写的方法来决定采取什么样的方式去请求依赖服务。HystrixCommand.run() ：返回一个单一的结果，或者抛出异常。HystrixObservableCommand.construct()： 返回一个Observable 对象来发射多个结果，或通过 onError 发送错误通知。 |
| 7    | Hystrix会将 "成功"、"失败"、"拒绝"、"超时" 等信息报告给断路器， 而断路器会维护一组计数器来统计这些数据。断路器会使用这些统计数据来决定是否要将断路器打开，来对某个依赖服务的请求进行 "熔断/短路"。 |
| 8    | 当命令执行失败的时候， Hystrix 会进入 fallback 尝试回退处理， 我们通常也称该操作为 "服务降级"。而能够引起服务降级处理的情况有下面几种：第4步： 当前命令处于"熔断/短路"状态，断路器是打开的时候。第5步： 当前命令的线程池、 请求队列或 者信号量被占满的时候。第6步：HystrixObservableCommand.construct() 或 HystrixCommand.run() 抛出异常的时候。 |
| 9    | 当Hystrix命令执行成功之后， 它会将处理结果直接返回或是以Observable 的形式返回。 |

tips：如果我们没有为命令实现降级逻辑或者在降级处理逻辑中抛出了异常， Hystrix 依然会返回一个 Observable 对象， 但是它不会发射任何结果数据， 而是通过 onError 方法通知命令立即中断请求，并通过onError()方法将引起命令失败的异常发送给调用者。

