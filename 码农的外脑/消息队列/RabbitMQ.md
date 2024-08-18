
```ad-summary
项目地址：[GitHub - Boer99/learn-rabbitmq](https://github.com/Boer99/learn-rabbitmq)

参考：heima2022
```

# 背景

同步通信
![[Pasted image 20240815092831.png]]

异步通信
![[Pasted image 20240815093105.png]]

大纲
![[Pasted image 20240815093306.png]]


# 同步和异步

## 同步调用

已支付服务为例

优点：
- 时效性强，等待到结果才返回

缺点：
- 拓展性差
- 性能下降
- 级联失败（中间错一步整个都错）

![[Pasted image 20240815095140.png]]

## 异步调用

异步调用方式其实就是基于消息通知的方式，一般包含三个角色：

- 消息发送者：投递消息的人，就是原来的调用方（点外卖）
- 消息代理：管理、暂存、转发消息，你可以把它理解成微信服务器（外卖柜）
- 消息接收者：接收和处理消息的人，就是原来的服务提供方（送外卖）

支付服务不再同步调用业务关联度低的服务，而是发送消息通知到Broker，具备下列优势：

- 解除耦合，拓展性强
- 无需等待，性能好
- ==故障隔离==
- 缓存消息，流量==削峰填谷==（激增的流量存在队列中，服务按自己的能力取任务）

异步调用的问题是：

- 不能立刻得到结果，时效性差
- 不确定下游业务是否执行成功
- 业务安全依赖于 Broker 的可靠性

![[Pasted image 20240815101033.png]]

```ad-tip
削峰填谷

![[Pasted image 20240815101002.png]]
```

# MQ 介绍

## 技术选型

MQ(MessageQueue)，中文是消息队列，字面来看就是存放消息的队列。也就是异步调用中的 Broker。

> Broker：中间人，经纪人
> 
> Erlang：一门面向并发的语言，常用来开发游戏

![[Pasted image 20240815102030.png]]


RabbitMQ 和 RocketMQ 同时保证高可用和高可靠，kafka 的吞吐量非常高，常用于日志服务

## 整体架构及核心概念

- virtual-host：虚拟主机
- publisher：消息发送者
- consumer：消息的消费者
- queue：队列，存储消息
- exchange：交换机，负责路由消息

> 公司为了节约成本，多个项目都用同一套mq，可以通过virtual-host隔离
![[Pasted image 20240815111512.png]]

## 安装

Docker：

mac：[MacBook M1 Pro 安装 RabbitMQ 保姆级教程，亲测有效~_mac下载rabbitmq-CSDN博客](https://blog.csdn.net/weixin_44719880/article/details/135551169)

## 交换机和队列 

需求：在 RabbitMQ 的控制台完成下列操作:

- 新建队列 hello.queue1 和 hello.queue2
- 向默认的 amp.fanout 交换机发送一条消息
- 查看消息是否到达 hello.queue1 和 hello.queue2
	- 需要让交换机绑定队列，队列才能收到消息

![[Pasted image 20240815114902.png]]

![[Pasted image 20240815114940.png|600]]

## 数据隔离

1）添加用户并登录
![[Pasted image 20240815115619.png|600]]

2）添加一个虚拟主机

![[Pasted image 20240815115841.png]]

3）测试不同 virtual host 之间的数据隔离现象

# Java 客户端

AMQP：Advanced Message Queuing Protocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求、

Spring AMQP：Spring AMQP 是基于 AMQP 协议定义的一套 API 规范，提供了模板来发送和接收消息。包含两部分，其中 spring-amqp 是基础抽象，spring-rabbit 是底层的默认实现。

## hello world

需求：
- 利用控制台创建队列 simple.queue
- 在 publisher 服务中，利用 SpringAMOP 直接向 simple.queue 发送消息
- 在 consumer 服务中，利用 SpringAMOP 编写消费者，监听 simple.queue 队列

> 跳过了交换机

1）引入 Spring AMQP 依赖

2）配置 RabbitMQ 服务端信息

```yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    virtual-host: /hmall
    username: hmall
    password: 123
```

3）发送消息

```java
@RunWith(SpringRunner.class)  
@SpringBootTest  
public class SpringAMQPTest {  
  
    @Resource  
    private RabbitTemplate rabbitTemplate;  
  
    @Test  
    public void testSendMessage2Queue() {  
        String queueName = "simple.queue";  
        String message = "hello, spring amqp2!";  
        rabbitTemplate.convertAndSend(queueName, message);  
    }  
}
```

4）接收消息：SpringAMQP 提供声明式的消息监听，我们只需要通过注解在方法上声明要监听的队列名称，将来 SpringAMOP 就会把消息传递给当前方法

```java
@Slf4j
@Component
public class MqListener {
    @RabbitListener(queues = "simple.queue")
    public void listenSimpleQueueMessage(String msg) {
        log.info("消费者收到了simple.queue的消息: {}", msg);
    }
}
```

## Work Queues

Work Queues，任务模型。简单的说就是==让多个消费者绑定到一个队列，共同消费队列中的消息==

模拟 WorkQueue，实现一个队列绑定多个消费者，基本思路如下:
1. 在 RabbitMQ 的控制台创建一个队列，名为 work.queue
2. 在 publisher 服务中定义测试方法，在 1 秒内产生 50 条消息，发送到 work.queue 
3. 在 consumer 服务中定义两个消息监听者，都监听 work.queue 队列 
4. 消费者 1 每秒处理 50 条消息，消费者 2 每秒处理 5 条消息 

1）publisher 服务
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAMQPTest {

    @Resource
    private RabbitTemplate rabbitTemplate;

    @Test
    public void testSendMessage2WorkQueue() throws InterruptedException {
        String queueName = "work.queue";
        for (int i = 1; i <= 50; i++) {
            String message = "hello, work.queue---" + i;
            rabbitTemplate.convertAndSend(queueName, message);
            Thread.sleep(20);
        }
    }
}
```

2）consumer 服务编写监听器监听 work.queue

```java
@Slf4j
@Component
public class MqListener {
    @RabbitListener(queues = "work.queue")
    public void listenWorkQueueMessage1(String msg) throws InterruptedException {
        log.info("消费者收到了work.queue的消息: {}", msg);
        Thread.sleep(20);
    }

    @RabbitListener(queues = "work.queue")
    public void listenWorkQueueMessage2(String msg) throws InterruptedException {
        log.error("消费者收到了work.queue的消息: {}", msg);
        Thread.sleep(200);
    }
}
```

默认情况下，RabbitMQ 的会将消息==依次轮询==投递给绑定在队列上的每一个消费者。但这并没有考虑到消费者是否已经处理完消息，可能出现消息堆积。

3）修改 application.yml，设置 preFetch 值为 1，确保==同一时刻最多投递给消费者 1 条消息==：

```yml
spring:
  rabbitmq:
    listener:
      simple:
        prefetch: 1
```

```ad-summary
Work 模型的使用:
- 多个消费者绑定到一个队列，可以加快消息处理速度
- 同一条消息只会被一个消费者处理
- 通过设置 prefetch 来控制消费者预取的消息数量，处理完一条再处理下一条，实现==能者多劳==
```

## Fanout Exchange 

Fanout Exchange 会将接收到的消息广播到每一个跟其绑定的 queue，所以也叫广播模式

![[Pasted image 20240815155953.png]]

利用 SpringAMQP 演示 FanoutExchange 的使用，实现思路如下:
1. 在 RabbitMQ 控制台中，声明队列 fanout.queue1 和 fanout.queue2
2. 在 RabbitMQ 控制台中，声明交换机 hmall.fanout，将两个队列与其绑定 
3. 在 consumer 服务中，编写两个消费者方法，分别监听 fanout.queue1 和 fanout.queue2
4. 在 publisher 中编写测试方法，向 hmall.fanout 发送消息

添加交换机
![[Pasted image 20240815160900.png|600]]

consumer 服务编写监听器监听 fanout.queue1 和 fanout.queue2

```java
@Slf4j
@Component
public class MqListener {
    @RabbitListener(queues = "fanout.queue1")
    public void listenFanoutQueueMessage1(String msg) {
        log.info("消费者收到了fanout.queue的消息: {}", msg);
    }

    @RabbitListener(queues = "fanout.queue2")
    public void listenFanoutQueueMessage2(String msg) {
        log.error("消费者收到了fanout.queue的消息: {}", msg);
    }
}
```

向hmall.fanout交换机发送消息

```java
@Test
    public void testSendMessage2FanoutQueue() {
        String exchangeName = "hmall.fanout";
        String queueName = "fanout.queue";
        String message = "hello, everyone!";
        rabbitTemplate.convertAndSend(exchangeName, queueName, message);
    }
```

```ad-summary
交换机的作用是什么?

- 接收publisher发送的消息
- 将消息==按照规则路由==到与之绑定的队列
- FanoutExchange的会将消息路由到每个绑定的队列
```

## Direct Exchange

Direct Exchange 会将接收到的消息根据规则路由到指定的 Queue，因此称为**定向路由**
- 每一个 Queue 都与 Exchange 设置一个 BindingKey
- 发布者发送消息时，指定消息的 RoutingKey
- Exchange 将消息路由到 BindingKey 与消息 RoutingKey 一致的队列



![[Pasted image 20240815164041.png]]

利用 SpringAMQP 演示 DirectExchange 的使用，需求如下:
1. 在 RabbitMQ 控制台中，声明队列 direct.queue1 和 direct.queue2
2. 在 RabbitMQ 控制台中，声明交换机 hmall.direct ，将两个队列与其绑定
3. 在 consumer 服务中，编写两个消费者方法，分别监听 direct.queue1 和 direct.queue2
4. 在 publisher 中编写测试方法，利用不同的 RoutingKey 向 hmall.direct 发送消息

交换机绑定队列：
![[Pasted image 20240815164621.png|500]]
```java
@Slf4j
@Component
public class MqListener {
    @RabbitListener(queues = "direct.queue1")
    public void listenDirectQueueMessage1(String msg) {
        log.info("消费者收到了direct.queue1的消息: {}", msg);
    }

    @RabbitListener(queues = "direct.queue2")
    public void listenDirectQueueMessage2(String msg) {
        log.error("消费者收到了direct.queue2的消息: {}", msg);
    }
}
```

利用不同的 RoutingKey 向 hmall.direct 发送消息
```java
@Test
    public void testSendMessage2DirectQueue() {
        String exchangeName = "hmall.direct";
        rabbitTemplate.convertAndSend(exchangeName, "red", "hello red!");
        rabbitTemplate.convertAndSend(exchangeName, "blue", "hello blue!");
        rabbitTemplate.convertAndSend(exchangeName, "yellow", "hello yellow!");
    }
```

```
08-15 16:53:15:108  INFO 72166 --- [ntContainer#5-1] cn.itcast.mq.listeners.MqListener        : 消费者收到了direct.queue1的消息: hello red!
08-15 16:53:15:108 ERROR 72166 --- [ntContainer#6-1] cn.itcast.mq.listeners.MqListener        : 消费者收到了direct.queue2的消息: hello red!
08-15 16:53:15:109  INFO 72166 --- [ntContainer#5-1] cn.itcast.mq.listeners.MqListener        : 消费者收到了direct.queue1的消息: hello blue!
08-15 16:53:15:109 ERROR 72166 --- [ntContainer#6-1] cn.itcast.mq.listeners.MqListener        : 消费者收到了direct.queue2的消息: hello yellow!
```

## Topic Exchange

Topic Exchange 与 Direct Exchange 类似，区别在于 routingKey 可以是多个单词的列表，并且以 `.` 分割。

Queue 与 Exchange 指定 BindingKey 时可以使用通配符:
- `#`：代指 0 个或多个单词
- `*`：代指一个单词

> Topic Exchange 非常灵活，推荐使用！

![[Pasted image 20240815170528.png]]

利用 SpringAMQP 演示 DirectExchange 的使用，需求如下:
1. 在 RabbitMQ 控制台中，声明队列 topic.queue1 和 topic.queue2
2. 在 RabbitMQ 控制台中，声明交换机 hmall.topic ，将两个队列与其绑定 
3. 在 consumer 服务中，编写两个消费者方法，分别监听 topic.queue1 和 topic.queue2
4. 在 publisher 中编写测试方法，利用不同的 RoutingKey 向 hmall.topic 发送消息

```java
@Slf4j
@Component
public class MqListener {
    @RabbitListener(queues = "topic.queue1")
    public void listenTopicQueueMessage1(String msg) {
        log.info("消费者收到了topic.queue1的消息: {}", msg);
    }

    @RabbitListener(queues = "topic.queue2")
    public void listenTopicQueueMessage2(String msg) {
        log.error("消费者收到了topic.queue2的消息: {}", msg);
    }
}
```

利用不同的 RoutingKey 向 hmall.topic 发送消息
```java
@Test
    public void testSendMessage2TopicQueue() {
        String exchangeName = "hmall.topic";
        rabbitTemplate.convertAndSend(exchangeName, "china.news", "hello china.news!");
        rabbitTemplate.convertAndSend(exchangeName, "japan.news", "hello japan.news!");
    }
```

## 声明队列和交换机

SpringAMQP 提供了几个类，用来声明队列、交换机及其绑定关系
- Queue：用于声明队列，可以用工厂类 QueueBuilder 构建
- Exchange：用于声明交换机，可以用工厂类 ExchangeBuilder 构建
- Binding：用于声明队列和交换机的绑定关系，可以用工厂类 BindingBuilder 构建

1）配置类方式

```java
@Configuration
public class FanoutConfig {
    @Bean
    public FanoutExchange fanoutExchange2() {
//        return ExchangeBuilder.fanoutExchange("hmall.fanout2").build();
        return new FanoutExchange("hmall.fanout2");
    }

    @Bean
    public Queue fanoutQueue3() {
        return new Queue("fanout.queue3");
    }

    @Bean
    public Queue fanoutQueue4() {
        return new Queue("fanout.queue4");
    }

    @Bean
    public Binding bindingQueue1(Queue fanoutQueue3, FanoutExchange fanoutExchange2){
        return BindingBuilder.bind(fanoutQueue3).to(fanoutExchange2);
    }

    @Bean
    public Binding bindingQueue2(){
        return BindingBuilder.bind(fanoutQueue4()).to(fanoutExchange2());
    }
}
```

```java
@Configuration  
public class DirectConfig {  
    @Bean  
    public DirectExchange directExchange2() {  
//        return ExchangeBuilder.fanoutExchange("hmall.fanout2").build();  
        return new DirectExchange("hmall.direct2");  
    }  
  
    @Bean  
    public Queue directQueue3() {  
        return new Queue("direct.queue3");  
    }  
  
    @Bean  
    public Queue directQueue4() {  
        return new Queue("direct.queue4");  
    }  
  
    @Bean  
    public Binding bindingRed(Queue directQueue3, DirectExchange directExchange2){  
        return BindingBuilder.bind(directQueue3).to(directExchange2).with("red");  
    }  
  
    @Bean  
    public Binding bindingBlue(){  
        return BindingBuilder.bind(directQueue3()).to(directExchange2()).with("blue");  
    }  
}
```

2）注解式，更方便

```java
@Slf4j
@Component
public class MqListener {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.ann.q1", durable = "true"),
            exchange = @Exchange(name = "hmall.ann.direct", type = ExchangeTypes.DIRECT),
            key = {"red", "blue"}
    ))
    public void listenDirectAnnQ1(String msg) {
        log.info("消费者收到了direct.ann.q1的消息: {}", msg);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.ann.q2", durable = "true"),
            exchange = @Exchange(name = "hmall.ann.direct", type = ExchangeTypes.DIRECT),
            key = {"red", "yellow"}
    ))
    public void listenDirectAnnQ2(String msg) {
        log.info("消费者收到了direct.ann.q2的消息: {}", msg);
    }
}
```

## 消息转换器

需求：测试利用 SpringAMOP 发送对象类型的消息
1. 声明一个队列，名为 object.queue 
2. 编写单元测试，向队列中直接发送一条消息，消息类型为 Map 

1）发送消息
```java
@Test
public void testSendMessage2ObjectQueue() {
	Map<String, Object> jack = new HashMap<>();
	jack.put("name", "jack");
	jack.put("age", 18);
	rabbitTemplate.convertAndSend("object.q", jack);
}
```

2）接收消息
```java
@RabbitListener(queues = "object.q")
public void listenTopicQueueMessage2(Map<String, Object> msg) {
	log.info("消费者收到了object.q的消息: {}", msg);
}
```

3）对象序列化方式

建议采用 JSON 序列化代替默认的 JDK 序列化（可读性差，体积大）

在生产者和消费者服务中引入 jackson 依赖，并配置 MessageConverter

```xml
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
</dependency>
```

```java
@Bean
public MessageConverter jackson2JsonMessageConverter() {
	return new Jackson2JsonMessageConverter();
}
```

![[Pasted image 20240816111939.png]]


# 生产者可靠性

## 生产者重连

有的时候由于网络波动，可能会出现客户端连接 MQ 失败的情况。通过配置我们可以开启连接失败后的重连机制：

```yml
spring:  
  rabbitmq:  
    template:  
      retry:  
        enabled: true # 开启超时重试  
        initial-interval: 1000ms # 失败后的初始等待时间  
        multiplier: 1 # 失败后下次的等待时长倍数  
        max-attempts: 3 # 最大重试次数
```

```ad-tip
当网络不稳定的时候，利用重试机制可以有效提高消息发送的成功率。不过 SpringAMQP 提供的重试机制是阻塞式的重试，也就是说多次重试等待的过程中，当前线程是被阻塞的，会影响业务性能。

如果对于业务性能有要求，建议==禁用重试机制==。如果一定要使用，请合理配置等待时长和重试次数，当然也可以考虑使==用异步线程来发送消息==。
```

## 生产者确认

RabbitMQ 提供 了 Publisher confirm 和 Publisher Return 两种确认机制。开启确机制认后，在 MQ 成功收到消息后会返回确认消息给生产者。返回的结果有以下几种情况:

- 消息投递到了 MQ，但是**路由失败**（代码错了、交换机配置错误）。此时会==通过 Publisher Return 返回路由异常原因，然后返回 ACK，告知投递成功==
- **临时**消息投递到了 MQ，并且入队成功，返回 ACK，告知投递成功
- **持久**消息投递到了 MQ，并且入队完成持久化，返回 ACK，告知投递成功
- 其它情况都会返回 **NACK**，告知投递失败

SpringAMQP 实现生产者确认

1）publisher 服务中添加开启生产者确认的配置

这里 publisher-confirm-type 有三种模式可选

- none: 关闭 confirm 机制
- simple: 同步阻塞等待 MQ 的回执消息
- correlated: MQ 异步回调方式返回回执消息

```yml
spring:  
  rabbitmq:  
    publisher-confirm-type: correlated # 开启publisher confirm机制  
    publisher-returns: true # 开启publisher return机制
```

2）每个RabbitTemplate只能配置一个ReturnCallback，因此需要在项目启动过程中配置

> 没理解这里的因果关系

```java
@Slf4j
@Configuration
public class MqConfirmConfig implements ApplicationContextAware {
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        RabbitTemplate template = applicationContext.getBean(RabbitTemplate.class);
        template.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            log.info("路由失败，收到消息的return callback，exchange：{}，routingKey：{}，message：{}，replyCode：{}，replyText：{}",
                    exchange, routingKey, message, replyCode, replyText);
        });
    }
}
```

3）发送消息，指定消息的ConfirmCallback

```java
@Test
public void testConfirmCallback() throws InterruptedException {
	CorrelationData cd = new CorrelationData(UUID.randomUUID().toString());
	cd.getFuture().addCallback(new ListenableFutureCallback<CorrelationData.Confirm>() {
		// spring内部的失败，几乎不会发生，不用管
		@Override
		public void onFailure(Throwable throwable) {
			log.error("消息回调失败: {}", throwable.getMessage());
		}

		// 回调成功
		@Override
		public void onSuccess(CorrelationData.Confirm confirm) {
			log.info("收到confirm callback回调");
			if (confirm.isAck()) {
				log.info("消息投递成功，收到ack");
			} else {
				log.error("消息投递失败，收nack，原因：{}", confirm.getReason());
			}
		}
	});
//        rabbitTemplate.convertAndSend("hmall.direct", "red", "测试确认机制", cd);
	rabbitTemplate.convertAndSend("hmall.direct", "不存在的routingKey", "测试确认机制", cd);
	Thread.sleep(2000); // JVM直接销毁了看不到，需要睡眠一下
}
```

```ad-note
如何处理生产者的确认消息？

- 生产者确认需要额外的网络和系统资源开销，尽量不要使用。
	- 如果一定要使用，无需开启 Publisher-Return 机制，因为一般路由失败是自己业务问题
- 对于 nack 消息可以==有限次数重试==，依然失败则记录异常消息
```

# MQ 的可靠性

在默认情况下，RabbitMQ 会将接收到的信息保存在内存中以降低消息收发的延迟。这样会导致两个问题:

- 一旦 MQ 宕机，内存中的消息会丢失
- 内存空间有限，当消费者故障或处理过慢时，会导致消息积压，引发 MQ 阻塞

## 数据持久化

rabbitmq实现持久化包括：

- 交换机持久化
- 队列持久化
- 消息持久化

