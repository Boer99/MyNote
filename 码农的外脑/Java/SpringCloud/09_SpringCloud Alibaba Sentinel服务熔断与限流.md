# SpringCloud Alibaba Sentinel 服务调用、熔断与限流

## Sentinel介绍

官方文档：https://github.com/alibaba/Sentinel   [中文](https://github.com/alibaba/Sentinel/wiki/介绍)

Sentinel 是轻量级的流量控制、熔断降级Java库；功能类似于Hystrix

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636121571580-6b6a7838-1def-4a6a-8780-9dced8b563e1.png" alt="image.png" style="zoom: 67%;" />



<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636120420628-55b29acd-23e4-4a77-8963-7e46ed37d7f2.png" alt="image.png" style="zoom: 80%;" />



怎么玩：[入门文档](https://spring-cloud-alibaba-group.github.io/github-pages/greenwich/spring-cloud-alibaba.html#_spring_cloud_alibaba_sentinel)

服务使用中的各种问题：服务雪崩、服务降级、服务熔断、服务限流



## 安装和访问Sentinel控制台

Sentinel分为两个部分：

- 核心库（Java客户端）不依赖任何框架/库，能够云星宇所有Java运行时环境，同时对Dubbo/Spring Cloud等框架也有较好的支持——后台；
- 控制台（Dashboard）基于Spring Boot开发，打包后可以直接运行，不需要额外的Tomcat等应用容器——前台 8080



下载到本地sentinel-dashboard-1.7.0.jar（[下载地址](https://github.com/alibaba/Sentinel/releases) ）

运行命令：`java -jar sentinel-dashboard-1.7.0.jar` （前提需要Java8，且8080端口不能被占用）

访问 `localhost:8080`，账号密码均为 sentinel



## 搭建演示工程

新建Module：cloudalibaba-sentinel-service8401

pom：以后基本上nacos 跟sentinel一起配置

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
    <dependency>
        <groupId>com.alibaba.csp</groupId>
        <artifactId>sentinel-datasource-nacos</artifactId>
    </dependency>
    <!--SpringCloud ailibaba sentinel -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
    <!--openfeign-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <!-- SpringBoot整合Web组件+actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <!--日常通用jar包配置-->
    ......
</dependencies>
```

yaml：

spring.cloud.sentinel.transport.port：指定与Sentinel控制台交互的端口，应用本地会启动一个占用该端口的Http Server，该 Server 会与 Sentinel 控制台做交互。

比如 Sentinel 控制台添加了1个限流规则，会把规则数据push给这个Http Server接收，Http Server再将规则注册到Sentinel中。

```yml
server:
  port: 8401

spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719
        
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

主启动类：

```java
@EnableDiscoveryClient
@SpringBootApplication
public class SentinelMainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(SentinelMainApp8401.class, args);
    }
}
```

业务类：

```java
@RestController
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA() {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB() {
        return "------testB";
    }
}
```

测试：

启动nacos、Sentinel、微服务8401

查看Sentinel控制台，发现什么也没有。原因：Sentinel采用懒加载机制

访问：http://localhost:8401/testA ，再查看Sentinel控制台，sentinel8080正在监控微服务8401



## 流控规则

### 基本介绍

即 流量限制控制规则，分为 流控模式 和 流控效果

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636175547549-6c575fa5-1aef-405d-b939-a4d68029c825.png" alt="image.png" style="zoom:67%;" />



各选项含义：

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636175587333-c9adeac1-c4a0-48be-b5f8-b174b1a4051c.png" alt="image.png" style="zoom: 80%;" />



### 流控模式

流控模式有三种：直接、关联、链路

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636176700047-5a4c8c69-02ef-4ab9-a2a1-2f118d4b814b.png" alt="image.png" style="zoom:67%;" />



#### 直接（默认）

这里留空的效果用 快速失败（默认）

##### QPS直接快速失败

QPS：query per second，每秒钟的请求数量，当调用该api的QPS达到阈值时，进行限流。

下面设置表示1秒钟内查询一次就是OK，若QPS>1，就直接快速失败，报默认错误

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636176644611-c8d5c126-80b4-40ae-a6d2-fa3de2cd58e0.png" alt="image.png" style="zoom: 50%;" />



测试一下，当/testA的访问超过1次/s是，页面报错。被Sentinel限流

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220906202008542.png" alt="image-20220906202008542" style="zoom:50%;" />



##### 线程数直接快速失败

当调用该api的线程数达到阈值的时候，进行限流。

与QPS直接快速失败不同的是，QPS情况下限制的是流量，比如银行的人流量只能是1人/s，也就是说每次只能一个人进入银行办理业务；而线程数就好比银行只有一个窗口开放，一群人都可以进入银行，但是每次只能处理一个人的业务。

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636178641049-10ce2a40-c119-496b-a555-f1d68ebf23cf.png" alt="image.png" style="zoom:67%;" />

修改一下8401的业务类，增加线程睡眠

```java
@GetMapping("/testA")
public String testA() {
    try {
        TimeUnit.MILLISECONDS.sleep(2000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "------testA";
}
```

然后重启8401，测试 /testA，最好用两个浏览器访问，效果更明显（我用一个浏览器测不出来）

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220906203038686.png" alt="image-20220906203038686" style="zoom:50%;" />



#### 关联

当关联的资源达到阈值时，就限流自己。比如当与A关联的资源B达到阈值后，就限流A自己。

支付接口达到阈值，限流下订单的接口。



设置效果：当==关联资源/testB的qps阀值超过1时==，就限流/testA的Rest访问地址

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636185529204-c4973c0a-7ca9-4fe9-857c-ed28a602c17d.png" alt="image.png" style="zoom: 67%;" />



**postman模拟并发密集访问testB**

先创建一个集合

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636186406791-1d909ef7-5650-4260-b198-56ad5bd4adef.png" alt="image.png" style="zoom: 80%;" />

然后将创建的访问/testB的请求保存在创建的集合中，或者直接在该集合下添加请求

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220906205545236.png" alt="image-20220906205545236" style="zoom:67%;" />

设定集合运行参数，20个线程，每次间隔0.3s访问一次（QPS>1），执行

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636186554384-4d416f2c-da41-4642-b726-f4852ebb648c.png" alt="image.png" style="zoom:80%;" />

然后再访问/testA，发现被限流，等postman执行完毕，testA又可以访问了



#### 链路

没讲，自己尝试吧



### 流控效果

#### 快速失败

快速失败在上面的流控模式演示过了，他是默认的流控效果，直接失败，抛出异常。

源码：com.alibaba.csp.sentinel.slots.block.flow.controller.DefaultController

#### warm up 预热

[官网](https://github.com/alibaba/Sentinel/wiki/限流---冷启动)         

源码：com.alibaba.csp.sentinel.slots.block.flow.controller.WarmUpController

公式：==阈值除以coldFactor(默认值为3)==，经过预热时长后才会达到阈值

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636204182838-32d2e249-f24f-4837-a0f0-3c4be43a5b8c.png" alt="image.png" style="zoom:80%;" />



<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636204711029-22aea67e-cc6b-4cfd-994c-dd51bb5fd0b6.png" alt="image.png" style="zoom:80%;" />



测试：狂点请求，可以看通过的QPS逐渐增加，最开始会报错限流，之后就可以抗住10/s的QPS了

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636204985038-e380529f-cc06-4fd7-9d61-ca191748bf9c.png" alt="image.png" style="zoom:80%;" />



应用场景：秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来，慢慢的把阀值增长到设置的阀值



#### 排队等待

[官网](https://github.com/alibaba/Sentinel/wiki/流量控制)  		

源码：com.alibaba.csp.sentinel.slots.block.flow.controller.RateLimiterController

匀速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效。

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636205316214-62574c1a-6bb6-47d6-915f-b02211bf4281.png" alt="img" style="zoom:80%;" />



以下设置含义：/testA每秒1次请求，超过的话就排队等待，等待的超时时间为20000毫秒。

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636205407842-08de7a67-8015-421a-9181-e3fa123cf8be.png" alt="image.png" style="zoom:80%;" />



修改一下业务代码，把线程名打印出来以验证是否排队。

```java
@GetMapping("/testB")
public String testB() {
    log.info(Thread.currentThread().getName());
    return "------testB";
}
```



测试：postman密集发送请求，10次请求，间隔500ms

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907005324019.png" alt="image-20220907005324019" style="zoom: 50%;" />

可以看到刚好满足1s一个请求，说明请求的执行进行了排队

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907005422702.png" alt="image-20220907005422702" style="zoom:80%;" />



## 降级规则（熔断规则）

### 基本介绍

[官网](https://github.com/alibaba/Sentinel/wiki/熔断降级)

Sentinel 熔断降级会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。

 当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 DegradeException）。

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907105418379.png" alt="image-20220907105418379" style="zoom:80%;" />



> 老版本的Sentinel的断路器是没有半开状态的，半开的状态系统自动去检测是否请求有异常，没有异常就关闭断路器恢复使用，有异常则继续打开断路器不可用。具体可以参考Hystrix。（在Hystrix中 快照时间窗口是值 阈值检测时间 ，而休眠时间窗口是指 断路器从开启到半开状态间隔的时间）
>
> **新版本的Sentinel加入了半开状态**！



### RT

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636378468415-d870b7ac-5a40-47c1-b6ca-13ae193bd2b4.png)

> 新版本：
>
> 慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，**并且**慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断。（跟豪猪科类似）



#### 演示

业务类

```java
@GetMapping("/testD")
public String testD() {
    //暂停1秒
    try {
        TimeUnit.MILLISECONDS.sleep(1000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    log.info(Thread.currentThread().getName() + ": testD 测试慢调用比例 RT");
    return "------testD";
}
```

添加降级规则

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907113154344.png" alt="image-20220907113154344" style="zoom: 50%;" />

jmeter测试

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907115150136.png" alt="image-20220907115150136" style="zoom:67%;" />

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907115226699.png" alt="image-20220907115226699" style="zoom:67%;" />

永远一秒钟打进来10个线程（大于5个了）调用testD，我们希望200毫秒处理完本次任务，如果超过200毫秒还没处理完，在未来s秒钟的时间内，断路器打开，微服务不可用，testD被熔断了

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907115315022.png" alt="image-20220907115315022" style="zoom:50%;" />

停止jmeter，没有这么大的访问量了，微服务恢复OK



### 异常比例

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636378442709-b1dd34e0-96d1-4be1-bbfd-26f9986b2a07.png" alt="image.png" style="zoom:90%;" />

> 新版本：
>
> 异常比例 (ERROR_RATIO)：当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。



#### 演示

业务类添加异常方法

```java
@GetMapping("/testER")
public String testER() {
    log.info("testER 测试异常比例");
    int age = 10 / 0;
    return "testER 测试异常比例";
}
```

配置降级规则

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907152712064.png" alt="image-20220907152712064" style="zoom:50%;" />

jmeter压测

![image-20220907152820726](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907152820726.png)

单独访问一次，必然来一次报错一次(int age = 10/0)，调一次错一次；

开启jmeter后，直接高并发发送请求，多次调用达到我们的配置条件了。断路器开启(保险丝跳闸)，微服务不可用了，不再报错error而是服务降级了



### 异常数

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636378394872-1cfc747b-422a-4df5-a551-e21137e49095.png" alt="image.png" style="zoom:90%;" />

- 时间窗口一定要大于等于60秒
- 异常数是按照分钟统计的

> 新版本
>
> 异常数 (ERROR_COUNT)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。



#### 演示

```java
@GetMapping("/testE")
public String testE() {
    log.info("testE 测试异常比例");
    int age = 10 / 0;
    return "------testE 测试异常比例";
}
```

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220907154530516.png" alt="image-20220907154530516" style="zoom:50%;" />



## 热点Key限流

> 具体到某个参数的

### 基本介绍

[官网](https://github.com/alibaba/Sentinel/wiki/热点参数限流)    

何为热点：热点即经常访问的数据，很多时候我们希望统计或者限制某个热点数据中访问频次最高的TopN数据，并对其访问进行限流或者其它操作。比如：

- 商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
- 用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限制会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限制。热点参数限流可以看作是一种特殊的流量控制，仅对包含热点参数的资源调用生效。 

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636271319999-dd06109b-159d-417c-ab3a-8ff219009af4.png" alt="img" style="zoom:67%;" />

Sentinel利用LRU策略统计最近最常访问的热电参数，结合令牌桶算法来进行参数级别的流控。热点参数限流支持集群模式。



### 基本使用

兜底防范分为系统默认和客户自定义；两种，根据之前的case，都是使用sentinel系统默认的提示：Blocked by Sentinel (flow limiting)。那我们能不能自定义兜底方法呢？类似hystrix，某个方法出问题了，就找对应的兜底降级方法？

类似于 `@HystrixCommand` ， 引入 `@SentinelResource` 注解。

热点规则共有资源名、限流模式（只支持QPS模式）、参数索引、单机阈值、统计窗口时长、是否集群6种参数，还有一些高级选项，用到时会详细介绍。这里会用到注解中的value作为资源名，兜底方法会在后面详细介绍

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636426167642-e877c00d-bcc0-4fdf-9b3d-a67bb51fd640.png" alt="image.png" style="zoom:50%;" />

**注意：**

资源名此处必须是 `@SentinelResource` 注解的 value 属性值

配置  `@GetMapping` 的请求路径无效



### 演示

还是在8401的controller中，加入热点测试方法。	

```java
@GetMapping("/testHotKey")
@SentinelResource(value = "testHotKey", blockHandler = "del_testHotKey") // 这里的名称可以随便写，但是一般跟rest地址一样
public String testHotkey(@RequestParam(value = "p1", required = false) String p1,
                         @RequestParam(value = "p1", required = false) String p2) {
    return "------testHotkey";
}

/**
 * 自定义的兜底方法
 */
public String del_testHotKey(String p1, String p2, BlockException e) {
    return "这次不用默认的兜底提示Blocked by Sentinel(flow limiting)，自定义提示：del_testHotKeyo(╥﹏╥)o...";
}
```

注解 `@SentinelResource(value = "testHotKey", blockHandler = "del_testHotKey")`

- 其中 `value = "testHotKey"` 是一个标识（Sentinel资源名），与rest的 `/testHotKey` 对应，这里value的值可以任意写，但是我们约定与rest地址一致，唯一区别是没有 `/`。
- `blockHandler = "del_testHotKey"` 则表示如果违背了Sentinel中配置的流控规则，就会调用我们自己的兜底方法 `del_testHotKey`



**配置热点key限流规则**

方法testHotKey里面第一个参数只要QPS超过每秒1次，马上降级处理。

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636425843025-86f932d2-ce64-4c9a-baa1-b67ed8db5eec.png" alt="image.png" style="zoom:80%;" />



**测试：**

访问 http://localhost:8401/testHotKey?p1=a&p2=b ，1次/s正常显示，迅速点击两次，触发热点限流，执行自定义兜底方法

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636275834807-7920316a-2d72-40fd-a595-60140e381d43.png" alt="img" style="zoom: 80%;" />



仅传入参数p2没有任何影响： http://localhost:8401/testHotKey?p2=b

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636275869802-3bdd8cb1-e801-45df-9953-c5a5969e1c5f.png" alt="img" style="zoom:80%;" />



现在开两个访问，一个通过jmeter压测 http://localhost:8401/testHotKey?p1=a&p2=b，另外再单独使用浏览器访问http://localhost:8401/testHotKey?p2=b，发现只带参数p2访问没有任何影响。





### 参数例外项

上述案例演示了第一个参数p1，当QPS超过1秒1次点击后马上被限流

特例情况：我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样，比如当p1的值等于5时，它的阈值可以达到200。



**演示：**

别忘了点添加

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636276820213-7a189398-d408-4696-9175-bddfbf2345a1.png)



测试：狂点 http://localhost:8401/testHotKey?p1=5&p2=b 没有限流



### 其他

在方法中添加异常试试

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636283487769-27477123-c4db-4bb0-870d-86d588b5a972.png" alt="image.png" style="zoom:80%;" />

测试直接错误页面



@SentinelResource 处理的是Sentinel控制台配置的违规情况，有blockHandler方法配置的兜底处理；

int age = 10/0，这个是java运行时报出的运行时异常RunTimeException，@SentinelResource不管

==@SentinelResource主管配置出错，运行出错该走异常走异常==



## 系统规则（系统自适应限流）

[官网](https://github.com/alibaba/Sentinel/wiki/系统自适应限流)

Sentinel 系统自适应限流从整体维度对应用入口流量进行控制，结合应用的 Load、CPU 使用率、总体平均 RT、入口 QPS 和并发线程数等几个维度的监控指标，通过自适应的流控策略，让系统的入口流量和系统的负载达到一个平衡，让系统尽可能跑在最大吞吐量的同时保证系统整体的稳定性。

系统保护规则是应用整体维度的，而不是资源维度的，并且**仅对入口流量生效**。入口流量指的是进入应用的流量（EntryType.IN），比如 Web 服务或 Dubbo 服务端接收的请求，都属于入口流量。

系统规则支持以下的模式：

- **Load 自适应**（仅对 Linux/Unix-like 机器生效）：系统的 load1 作为启发指标，进行自适应系统保护。当系统 load1 超过设定的启发值，且系统当前的并发线程数超过估算的系统容量时才会触发系统保护（BBR 阶段）。系统容量由系统的 maxQps * minRt 估算得出。设定参考值一般是 CPU cores * 2.5。
- **CPU usage**（1.5.0+ 版本）：当系统 CPU 使用率超过阈值即触发系统保护（取值范围 0.0-1.0），比较灵敏。
- **平均 RT**：当单台机器上所有入口流量的平均 RT 达到阈值即触发系统保护，单位是毫秒。
- **并发线程数**：当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护。
- **入口 QPS**：当单台机器上所有入口流量的 QPS 达到阈值即触发系统保护。



### 案例——配置全局QPS

![img](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636283979732-9ec624d5-b538-4e97-b131-800c32624945.png)

不管是/testA还是/testB 只要QPS > 1 整个系统就不能用。

这个粒度太粗，就相当于一个窗口人很多，整个银行就不接待人了，不太建议使用。



## @SentinelResource 注解详解

### 按资源名称限流+后续处理（自定义兜底）

后续处理采用自定义兜底方法

> 自己测试发现：按资源名称限流，不写兜底方法会报错，不会走默认处理

8401模块 pom

```xml
<dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
  <groupId>com.atguigu.springcloud</groupId>
  <artifactId>cloud-api-commons</artifactId>
  <version>${project.version}</version>
</dependency>
```

业务类

```java
@RestController
public class RateLimitController {
    @GetMapping("/byResource")
    @SentinelResource(value = "byResource", blockHandler = "handleException")
    public CommonResult byResource() {
        return new CommonResult(200, "按资源名称限流测试OK", new Payment(2020L, "serial001"));
    }

    public CommonResult handleException(BlockException exception) {
        return new CommonResult(444, exception.getClass().getCanonicalName() + "\t 服务不可用");
    }
}
```

配置流控规则——按资源名称添加流控规则

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636285393933-ddc0a9ee-ed0e-440f-bc83-48cd579dcc76.png" alt="image.png" style="zoom:80%;" />

自测



### 按照Url地址限流+后续处理（默认处理）

通过访问的URL来限流，会返回Sentinel自带默认的限流处理信息

> 自己测试发现：按照Url地址限流，不会走自定义的兜底方法，会走默认处理



业务类

```java
@GetMapping("/rateLimit/byUrl")
@SentinelResource(value = "byUrl")
public CommonResult byUrl(){
    return new CommonResult(200,"按url限流测试OK",new Payment(2020L,"serial002"));
}
```

设置流控规则

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636285921229-af984a98-bdca-4013-ba83-a43c63fdfe1a.png" alt="image.png" style="zoom:80%;" />

测试：不知道为什么我这里流控规则就是不生效，重启也没用，==但是把业务类和流控规则的 /rateLimit/byUrl 改成 /byUrl就可以了？？？==



### 客户自定义限流处理逻辑

##### 上面兜底方案面临的问题

- 系统默认的，没有体现我们自己的业务要求
-  依照现有条件，我们自定义的处理方法又和业务代码耦合在一块，不直观
-  每个业务方法都添加一个兜底的，那代码膨胀加剧
- 全局统一的处理方法没有体现

##### 演示自定义限流处理逻辑

创建CustomerBlockHandler类用于自定义限流处理逻辑

注意：

- 静态方法
- 参数和原方法一致，多一个 BlockException e

```java
package com.boer.springcloud.myhandler;

import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.boer.springcloud.entities.CommonResult;

public class CustomerBlockHandler {
    public static CommonResult handlerException(BlockException e) {
        return new CommonResult(4444, "按客户自定义, global handlerException----1");
    }

    public static CommonResult handlerException2(BlockException e) {
        return new CommonResult(4444, "按客户自定义, global handlerException----2");
    }
}
```

业务类 自定义通用的限流处理逻辑

```java

@GetMapping("/rateLimit/customerBlockHandler")
@SentinelResource(value = "customerBlockHandler",
        blockHandlerClass = CustomerBlockHandler.class,
        blockHandler = "handlerException2")  // 找CustomerBlockHandler类里的handleException2方法进行兜底处理
public CommonResult customerBlockHandler() {
    return new CommonResult(200, "按客户自定义限流处理逻辑");
}
```



设置流控规则

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636287621476-3a3eaf6f-4507-468f-bde2-decdc7f78973.png" alt="image.png" style="zoom:80%;" />



测试：狂点 http://localhost:8401/rateLimit/customerBlockHandler，触发流控（不生效就重启Sentinel）

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636287665130-41bc38f3-b6cd-483a-b3f4-7fbe7462cb0a.png)



结构说明

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636287798408-543e1ede-a827-49c3-92f0-5c0b13725223.png" alt="image.png" style="zoom:80%;" />



### 更多属性说明

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636289727930-10fdb9a1-5ddf-47a5-bd1a-8c62f33357ed.png)

Sentinel主要有三个核心Api

1. SphU定义资源
2. Tracer定义统计
3. ContextUtil定义了上下文



### fallback 和 blockHandler

见下方Ribbon系列，这里老师的章节安排有点奇怪！



## 服务调用与熔断功能

sentinel分别整合ribbon+openFeign以及设置fallback

### Ribbon系列演示

nacos中整合了Ribbon，所以直接使用nacos就行。启动nacos和Sentinel。

#### 服务提供者9003/9004构建

新建 cloudalibaba-provider-payment9003/9004 两个一样的做法

pom

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!-- SpringBoot整合Web组件 -->
    ......
    <!--日常通用jar包配置-->
    ......
    </dependency>
</dependencies>
```

yml

```yml
server:
  port: 9003

spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 # 配置Nacos地址

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

主启动类

```java
@EnableDiscoveryClient
@SpringBootApplication
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9003.class, args);
    }
}
```

业务类

```java
@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    public static HashMap<Long, Payment> hashMap = new HashMap<>();

    static {
        hashMap.put(1L, new Payment(1L, "28a8c1e3bc2742d8848569891fb42181"));
        hashMap.put(2L, new Payment(2L, "bba8c1e3bc2742d8848569891ac32182"));
        hashMap.put(3L, new Payment(3L, "6ua8c1e3bc2742d8848569891xt92183"));
    }

    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
        Payment payment = hashMap.get(id);
        CommonResult<Payment> result = new CommonResult(200, "from mysql,serverPort:  " + serverPort, payment);
        return result;
    }
}
```

测试

http://localhost:9003/paymentSQL/1     

http://localhost:9004/paymentSQL/1



####  服务消费者84构建

新建 cloudalibaba-consumer-nacos-order84

pom

```xml
<dependencies>
    <!--SpringCloud ailibaba nacos -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    <!--SpringCloud ailibaba sentinel -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
    </dependency>
    <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
    ...
    <!-- SpringBoot整合Web组件 -->
    ...
    <!--日常通用jar包配置-->
    ...
<dependencies>
```

yml

```yml
server:
  port: 84

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        # 配置Sentinel dashboard地址
        dashboard: localhost:8080
        # 默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

# 消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
# 方便controller的@value获取
service-url:
  nacos-user-service: http://nacos-payment-provider
```

主启动类

```java
@EnableDiscoveryClient
@SpringBootApplication
public class OrderNacosMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain84.class, args);
    }
}
```

config

```java
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced  //不要忘了
    public RestTemplate getRestemplate() {
        return new RestTemplate();
    }
}
```

业务类

```java
@RestController
public class CircleBreakerController {
    @Value("${service-url.nacos-user-service}")
    private String SERVICE_URL;

    @Autowired
    RestTemplate restTemplate;

    @RequestMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback")  //没有配置兜底方法
    public CommonResult<Payment> fallback(@PathVariable Long id) {
        CommonResult<Payment> result = restTemplate.getForObject(SERVICE_URL + "/paymentSQL/" + id, CommonResult.class, id);

        if (id == 4) {
            throw new IllegalArgumentException("IllegalArgumentException,非法参数异常....");
        } else if (result.getData() == null) {
            throw new NullPointerException("NullPointerException,该ID没有对应记录,空指针异常");
        }

        return result;
    }
}
```

测试：负载均衡实现

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636295900018-2e6c309b-8454-4cab-a0e8-d00d83fba367.png)

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636295908562-9d50031c-7b65-437b-9494-d3f4454f0ddd.png)



#### fallback 和 blockHandler

> 这块应该放在@SentinelResource 注解详解

```java
@SentinelResource(value = "xxx", fallback = "fffff", blockHandler = "bbbbbb")
```

fallback管运行异常，blockHandler管配置违规。



##### 没有任何配置

前面我们的84消费端，@SentinelResource里面只配置了value，fallback和blockHandler都没有配置，该情况下我们测试一下 http://localhost:84/consumer/fallback/4

出现了错误页面，error page对客户不友好，所以我们需要有兜底方法。



##### 只配置fallback

服务可以正常访问，但是业务逻辑出现错误，需要降级兜底。

业务类：

![image-20220908130458066](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220908130458066.png)

测试：

![image-20220908130301397](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220908130301397.png)

![image-20220908130335664](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220908130335664.png)



##### 只配置blockHandler

blockHandler对应服务熔断，当前sentinel配置已经违规（RT数过多、异常过多），服务熔断后不可用，需要给客户提示，进行一个熔断的兜底。

业务类：

![image-20220908131724587](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220908131724587.png)

测试：访问该地址五次以后，第六次服务熔断了

![image-20220908131533611](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220908131533611.png)



##### fallback和blockHandler都配置

![image-20220909004906648](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909004906648.png)

在没有违反sentinel规则时，出现业务异常（降级）走fallback方法；违反了sentinel规则时，直接微服务不可用（熔断），走blockHandler指定的自定义方法



##### 异常忽略属性

```java
@SentinelResource(
        value = "fallback",
        fallback = "handleFallback",
        blockHandler = "blockHandler",
    	// 设置忽略的异常
        exceptionsToIgnore = {IllegalArgumentException.class}
)
```

可以选择性的配置当某些异常发生时，不触发fallback的兜底方法。

![image-20220909005857895](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909005857895.png)

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909005711226.png" alt="image-20220909005711226" style="zoom:50%;" />



### Feign系列演示

修改84模块

pom：加入feign的依赖

```xml
<!--SpringCloud openfeign -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

yml：激活Sentinel对Feign的支持

```yml
feign:
  sentinel:
    enabled: true
```

主启动类：加上 `@EnableFeignClient` 注解开启OpenFeign

业务类：

```java
@FeignClient(value = "nacos-payment-provider", fallback = PaymentFallbackService.class)
public interface PaymentService {
    @GetMapping(value = "/paymentSQL/{id}")
    public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id);
}
```

```java
@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult<Payment> paymentSQL(Long id) {
        return new CommonResult<>(444, "服务降级返回,没有该流水信息", new Payment(id, "errorSerial......"));
    }
}
```

controller

```java
// ==================OpenFeign==================
@Resource
private PaymentService paymentService;

@GetMapping(value = "/consumer/paymentSQL/{id}")
public CommonResult<Payment> paymentSQL(@PathVariable("id") Long id) {
    if (id == 4) {
        throw new RuntimeException("没有该id");
    }
    return paymentService.paymentSQL(id);
}
```

测试：

负载均衡生效

![image-20220909130623269](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909130623269.png)

![image-20220909130645487](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909130645487.png)

关闭9003、9004微服务提供者，看到84消费者自动执行降级兜底方法。

![image-20220909130723941](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909130723941.png)



### 熔断框架比较

![image.png](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636440701197-692a61f8-63d8-494d-8ead-f44acefab3df.png)



## 规则持久化

前面我们微服务新增的限流规则后，微服务关闭后就会丢失，当时配置都限流规则都是临时的。 将限流配置规则持久化进Nacos保存，只要刷新8401某个rest地址，sentinel控制台的流控规则 就能看到。只要nacos里面的配置不删除，针对8401上的sentinel上的流控规则就持续存在。 （也可以持久化到文件，redis，数据库等）

### 案例——修改8401已完成持久化设置

pom：导入持久化所需依赖

```xml
<!--SpringCloud ailibaba sentinel-datasource-nacos 持久化-->
<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

yaml：添加nacos数据源配置

```yml
spring:
  sentinel:
    datasource:
      ds1:
        nacos:
          server-addr: localhost:8848
          dataId: cloudalibaba-sentinel-service
          groupId: DEFAULT_GROUP
          data-type: json
          rule-type: flow
```

添加nacos业务规则配置：我们将sentinel的流控配置保存在nacos中，因为nacos的配置持久化在了数据库中。

<img src="图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/1636443367569-59d065b4-ec4e-43c0-8f50-8f805c8c818f.png" alt="image.png" style="zoom: 67%;" />

配置内容详解

```json
[
    {
        "resource": "/rateLimit/byUrl",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
```

- resource：资源名称；
- limitApp：来源应用；
- grade：阈值类型，0表示线程数，1表示QPS；
- count：单机阈值；
- strategy：流控模式，0表示直接，1表示关联，2表示链路；
- controlBehavior：流控效果，0表示快速失败，1表示Warm Up，2表示排队等待；
- clusterMode：是否集群。



随意访问8401的接口，可以看到sentinal已经有的流控规则

![image-20220909171334986](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909171334986.png)

持久化到数据库了

![image-20220909171309606](图片/09_SpringCloud Alibaba Sentinel服务熔断与限流/image-20220909171309606.png)
