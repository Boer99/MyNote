
```ad-summary
参考：
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










