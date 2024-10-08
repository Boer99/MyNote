# ---------- JavaWeb

##  项目中从前端来的请求如何最终到达 mysql 数据库 #得物_24_后端 

# ---------- Spring

# 概念

Spring 是一款开源的轻量级 Java 开发框架，旨在提高开发人员的开发效率以及系统的可维护性

Spring 是一个家族，通常 Spring 框架指的都是 Spring Framework，是 Spring 家族中其他框架的底层基础

它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发，比如说 Spring 支持 IoC 和 AOP、可以很方便地对数据库进行访问、可以很方便地集成第三方组件（电子邮件，任务，调度，缓存等等）、对单元测试支持比较好、支持 RESTful Java 应用程序的开发

Spring 最核心的思想就是不重新造轮子，开箱即用，提高开发效率

## Spring 包含的模块有哪些？ #rep 

> 讲一下你知道的模块 #得物_24_实习_Java 

![600](assets/Pasted%20image%2020240320011936.png)

各模块间的依赖关系

![600](assets/Pasted%20image%2020240320011739.png)

*Core Container*：Spring 框架的核心模块、基础模块，主要提供 IoC 依赖注入功能的支持。Spring 其他所有的功能基本都需要依赖于该模块

- spring-**core**：Spring 框架基本的核心工具类
- spring-**beans**：提供对 bean 的创建、配置和管理等功能的支持
- spring-context：提供对国际化、事件传播、资源加载等功能的支持
- spring-expression：提供对表达式语言（Spring Expression Language） SpEL 的支持，只依赖于 core 模块，不依赖于其他模块，可以单独使用

> maven 里导入 spring-context
> 
> ![600](assets/Pasted%20image%2020240320014712.png)

*AOP*

- spring-**aspects**：该模块为与 AspectJ 的集成提供支持。
- spring-**aop**：提供了面向切面的编程实现
- spring-instrument：提供了为 JVM 添加代理（agent）的功能。 具体来讲，它为 Tomcat 提供了一个织入代理，能够为 Tomcat 传递类文件，就像这些文件是被类加载器加载的一样。没有理解也没关系，这个模块的使用场景非常有限

*Data Access/Integration*

- spring-**jdbc**：提供了对数据库访问的抽象 JDBC。不同的数据库都有自己独立的 API 用于操作数据库，而 Java 程序只需要和 JDBC API 交互，这样就屏蔽了数据库的影响。
- spring-**tx**：提供对事务的支持。
- spring-orm：提供对 Hibernate、JPA、iBatis 等 ORM 框架的支持。
- spring-oxm：提供一个抽象层支撑 OXM(Object-to-XML-Mapping)，例如：JAXB、Castor、XMLBeans、JiBX 和 XStream 等。
- spring-jms : 消息服务。自 Spring Framework 4.1 以后，它还提供了对 spring-messaging 模块的继承

*Spring Web*

- spring-web：对 Web 功能的实现提供一些最基础的支持。
- spring-**webmvc**：提供对 Spring MVC 的实现。
- spring-**websocket**：提供了对 WebSocket 的支持，WebSocket 可以让客户端和服务端进行双向通信。
- spring-webflux：提供对 WebFlux 的支持。WebFlux 是 Spring Framework 5.0 中引入的新的响应式框架。与 Spring MVC 不同，它不需要 Servlet API，是完全异步

*Messaging*：spring-messaging 是从 Spring4.0 开始新加入的一个模块，主要职责是为 Spring 框架集成一些基础的报文传送应用。

*Spring Test*：

- Spring 团队提倡测试驱动开发（TDD）。有了控制反转 (IoC)的帮助，单元测试和集成测试变得更简单
- Spring 的测试模块对 **JUnit**（单元测试框架）、TestNG（类似 JUnit）、Mockito（主要用来 Mock 对象）、PowerMock（解决 Mockito 的问题比如无法模拟 final, static， private 方法）等等常用的测试框架支持的都比较好。

## 为什么要用 spring，静态实现方式可不可以

#携程

#todo 

# IOC、DI

## 介绍 Spring IoC(1)  #rep

#PDD_23_秋招_后端 

*IOC*：Inversion of Control，控制反转，是一种**设计思想**，而不是一个具体的技术实现

- IoC 的思想就是将原本在程序中手动创建对象的**控制权**（实例化、管理），**转移**到外部（Spring 框架）

*Spring IoC*：Spring 对 IOC 思想进行了实现，提供了一个 **IoC 容器**

- 将对象之间的相互**依赖关系**交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。可以从容器里获取对象，并且对象已经绑定了所有依赖关系

- 很大程度上**简化应用的开发**，把应用从复杂的依赖关系中解放出来。 IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要**配置好配置文件/注解即可**，完全不用考虑对象是如何被创建出来的。

- IoC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象

*Bean*：被 IoC 容器所管理的对象

## 注册 Bean

### 注册 Bean 的方式 #rep 

config 配置类上加 `@Configuration` + `@ComponentScan` 
- `@Component` 及其衍生注解
- Bean 类中通过 `@Bean` 方法
- config 配置类上加 `@Import({class1, class2})`
	- 手动加入配置类到核心配置
- 编程式注册：通过 AnnotationConfigApplicationContext 实例的 `registerBean()` 方法

### @Component 和 @Bean 的区别是什么 #rep

1）作用对象：`@Component` 作用于类，`@Bean` 作用于方法

2）使用方式：

- `@Component` 通常配合 `@ComponentScan(包扫描路径)` 自动注册到容器中
- `@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean

3）自定义性：

- `@Bean` 注解比 `@Component` 注解的**自定义性更强**，而且很多地方我们只能通过 `@Bean` 注解来注册 bean。比如需要将第三方库的类注册为 bean 只能通过 `@Bean` 来实现

## 依赖注入

### Spring 注入有哪几种方式？(1)

- 字段（属性）注入
	- 暴力反射
- 构造器注入
- setter 注入

> 注解实现：@Autowired 加在字段、构造器方法、setter 方法上
> 
> Spring 官方文档：强制依赖使用构造器注入，可选依赖使用 setter 注入

### @Autowired 和 @Resource 的区别是什么？ #rep

> 自动装配具体是怎么实现的？(1)

1）`@Autowired` 是 Spring 提供的注解，`@Resource` 是 JDK 提供的注解

2）`Autowired` 默认的注入方式为 byType，`@Resource` 默认注入方式为 byName

- byName 默认会找首字母小写的类名

3）当一个接口存在多个实现类的情况下，`@Autowired` 和 `@Resource` 都需要通过名称才能正确匹配到对应的 Bean

- `@Autowired` 可以通过 `@Qualifier` 注解来显式指定名称
- `@Resource` 可以通过 `name` 属性来显式指定名称

4）`@Autowired` 支持在构造函数、方法、字段和参数上使用。`@Resource` 主要用于**字段和方法**上的注入，不支持在构造函数或参数上使用

## Bean 的作用域有哪些? #rep

- *singleton*（默认） : IoC 容器中只有唯一的 bean 实例
- *prototype* : 每次获取都会创建一个新的 bean 实例。也就是说，连续 `getBean()` 两次，得到的是不同的 Bean 实例

仅 Web 应用可用： #todo

- *request* : 每一次 HTTP 请求都会产生一个新的 bean（**请求 bean**），该 bean 仅在当前 HTTP request 内有效
- *session* : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（**会话 bean**），该 bean 仅在当前 HTTP session 内有效
- *application/global-session*：每个 Web 应用在启动时创建一个 Bean（**应用 Bean**），该 bean 仅在当前应用启动时间内有效
- *websocket*：每一次 WebSocket 会话产生一个新的 bean

## Bean 是线程安全的吗？ #rep

Bean 是否线程安全，取决于其**作用域和状态**

- prototype 作用域下，每次获取都会创建一个新的 bean 实例，不存在资源竞争问题，所以不存在线程安全问题

- **singleton** 作用域下，
	- 如果这个 bean 是**有状态**的话，那就存在线程安全问题
	- 大部分 Bean 实际都是**无状态**的（比如 Dao、Service），是线程安全的

> - 有状态：包含可变的成员变量的对象
> - 无状态：没有定义可变的成员变量

对于有状态单例 Bean 的线程安全问题，常见的有两种解决办法：

1. 在 Bean 中尽量避免定义**可变**的成员变量
2. 在类中定义一个 ThreadLocal 成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）

## bean 生命周期 #rep

> #饿了么

bean 生命周期控制：在 bean 创建后到销毁前做一些事情

![](assets/Pasted%20image%2020240320162438.png)

实例化：

- Bean 容器找到配置文件中 Spring Bean 的定义
- Bean 容器利用 Java **Reflection** API 调用bb创建 Bean 的实例

初始化：

- 设置属性值：自动装配、属性注入、依赖检查

- 调用 Bean 实现的 **`*.Aware` 接口**的方法，列举三个：
	- *BeanNameAware* 接口：调用 `setBeanName()` 方法，传入 Bean 的名字
	- *BeanClassLoaderAware* 接口：调用 `setBeanClassLoader()` 方法，传入 ClassLoader 对象的实例
	- *BeanFactoryAware* 接口：调用 `setBeanFactory()` 方法，传入 BeanFactory 对象的实例

- 如果有和加载这个 Bean 的 Spring 容器相关的 **BeanPostProcessor** 对象（Bean），执行 `postProcessBeforeInitialization()` 方法

- 如果 Bean 实现了 `InitializingBean` 接口，执行 `afterPropertiesSet()` 方法
- 如果 Bean 在【配置文件】中的定义包含 init-method 属性，执行 method 指定的自定义初始化方法

- 如果有和加载这个 Bean 的 Spring 容器相关的 **BeanPostProcessor** 对象（Bean），执行 `postProcessAfterInitialization()` 方法

Bean 准备就绪，开始使用

销毁：

- 当要销毁 Bean 的时候，如果 Bean 实现了 `DisposableBean` 接口，执行 `destroy()` 方法
- 当要销毁 Bean 的时候，如果 Bean 在【配置文件】中的定义包含 destroy-method 属性，执行 destroy-method 指定的自定义销毁方法

#todo 查看源码

```java
@Component("BeanLifecycle666")
public class BeanLifecycle implements InitializingBean, DisposableBean, BeanNameAware, BeanClassLoaderAware {
    @Value("field1")
    public String field1;

    @Value("111")
    public int field2;

    public Dog dog;

    @Override
    public void setBeanName(String name) {
        System.out.println("BeanNameAware接口的---setBeanName---bean的名字是：" + name);
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("BeanClassLoaderAware接口的---setBeanClassLoader---" + classLoader);
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean接口的---afterPropertiesSet");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean接口的---destroy");
    }

    @Autowired
    public void setDog(Dog dog) {
        System.out.println("设置属性---setDog---" + dog);
        this.dog = dog;
    }

    public String getField1() {
        System.out.println("getField1");
        return field1;
    }

    public void setField1(String field1) {
        System.out.println("设置属性---setField1---" + field1);
        this.field1 = field1;
    }

    public int getField2() {
        System.out.println("getField2");
        return field2;
    }

    public void setField2(int field2) {
        System.out.println("设置属性---setField2---" + field2);
        this.field2 = field2;
    }
}

@Component  
public class CustomBeanPostProcessor implements BeanPostProcessor {  
  
    public CustomBeanPostProcessor() {  
        System.out.println("CustomBeanPostProcessor---construct");  
    }  
  
    @Override  
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
        if (bean instanceof BeanLifecycle) {  
            System.out.println("CustomBeanPostProcessor---process bean before initialization");  
        }  
        return bean;  
    }  
  
    @Override  
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
        if (bean instanceof BeanLifecycle) {  
            System.out.println("CustomBeanPostProcessor---process bean after initialization");  
        }  
        return bean;  
    }  
}

public class Main {  
    public static void main(String[] args) {  
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(MyConfig.class);  
        
        BeanLifecycle beanLifecycle = ctx.getBean(BeanLifecycle.class);  
        System.out.println(beanLifecycle.getField1());  
        
        String[] names = ctx.getBeanDefinitionNames();  
        for (String name : names) {  
            System.out.println(name);  
        } 
    }
}
```

### beanPostProcessor #rep

> Spring 中的后置处理器的作用是什么？ #顺丰_23_秋招_Java 
> 
> 知不知道 beanPostProcessor #饿了么 （自定义实例化的逻辑，进行一些 Bean 实例的定制操作）

BeanPostProcessor (BPP) 是 Spring IOC 容器给我们提供的一个**扩展接口**

- 主要作用：帮我们**在 Bean 的初始化前后添加一些自己的逻辑处理**
- Spring 内置了很多 BPP，我们也可以定义多个 BPP 接口的实现，然后**注册到容器中**

- BPP 对象会在**普通对象创建前被创建**

- 在 bean 的初始化前后，容器会遍历所有的 BPP 的实现类的对象，执行他们的 前置处理方法 (postProcessBeforeInitialization) 和 后置处理方法 (postProcessAfterInitialization)

BPP 的典型使用：Abstract**AutoProxyCreator**

- 服务于 Spring AOP 的 BBP（连接 IOC 和 AOP 的桥梁）
- 它的后置处理方法完成 AOP 的代理对象的创建（带有切面逻辑的对象，注入进来之后，都不是原来的对象了）

# AOP

## 介绍 Spring aop  #rep


#得物_24_实习_Java 

_AOP (Aspect Oriented Programming)_ 面向切面编程，一种**编程范式**，指导开发者如何组织程序结构

- 作用：在不惊动原始设计的基础上为其进行功能增强（代理模式课可以实现）

- 应用场景：能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性

AOP 切面编程涉及到的一些专业术语：

| 术语             | 含义                                                                           |
| -------------- | ---------------------------------------------------------------------------- |
| 目标(Target)     | 被通知的对象                                                                       |
| 代理(Proxy)      | 向目标对象应用通知之后创建的代理对象                                                           |
| 连接点(JoinPoint) | 目标对象的所属类中，定义的所有**方法**均为连接点                                                   |
| 切入点(Pointcut)  | 对连接点进行拦截的条件定义（在哪些连接点切入）                                                      |
| 通知(Advice)     | 增强的逻辑 / 代码，也即拦截到目标对象的连接点之后要做的事情                                              |
| 切面(Aspect)     | 切入点(Pointcut)+通知(Advice)                                                     |
| Weaving(织入)    | 将通知应用到目标对象，进而生成代理对象的过程动作。织入可以在编译时，类加载时和运行时完成。在编译时进行织入就是静态代理，而在运行时进行织入则是动态代理。 |

## AspectJ 定义的通知类型有哪些？ #rep

> AOP 的通知类型和这个差不多

| 术语              | 含义                                                                                                          |
| --------------- | ----------------------------------------------------------------------------------------------------------- |
| @Before         | 连接点执行前执行的逻辑                                                                                                 |
| @AfterReturning | 连接点**正常执行**（未抛出异常）后执行的逻辑                                                                                    |
| @AfterThrowing  | 连接点**抛出异常**后执行的逻辑                                                                                           |
| @After          | 无论连接点是正常执行还是抛出异常，在连接点执行完毕后执行的逻辑                                                                             |
| @Around         | 编程式控制目标对象的方法调用。环绕通知是所有通知类型中**可操作范围最大**的一种，因为它可以**直接拿到目标对象，以及要执行的方法**，所以环绕通知可以任意的在目标对象的方法调用前后搞事，甚至不调用目标对象的方法 |

> 前、后、前+后

## Spring AOP 和 AspectJ AOP 有什么区别？ #rep

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了

区别：

- **Spring AOP 属于运行时增强（动态代理），而 AspectJ 是编译时增强（支持编译时、编译后和加载时织入）** 
- Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作（在执行应用程序之前 (在运行时) 前, 各方面直接在代码中进行织入）
- AspectJ 相比于 Spring AOP 功能更加强大
	- Spring AOP 仅支持方法切入点，AspectJ 支持所有切入点
- 如果切面比较少，那么两者性能差异不大。如果**切面太多的话，最好选择 AspectJ** ，它比 Spring AOP 快很多

## Spring AOP 底层原理/怎么实现？(3) #rep

#PDD_23_秋招_基础电商_后端 #字节_23_实习_Java 

从 Bean 的生命周期中的初始化流程来讲，Spring 的 AOP 会在 bean 实例的实例化已完成，进行初始化**后置处理**时创建代理对象，

- 通过 BBP 的实现类 **AbstractAutoProxyCreator** 的后置处理方法完成 AOP 代理对象的创建

Spring AOP 就是基于**动态代理**的

- 如果要代理的对象，**实现了某个接口**，那么 Spring AOP 会使用 **JDK Proxy**，去创建代理对象，
- 而对于**没有实现接口**的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 **Cglib** 生成一个被代理对象的**子类来作为代理**

![](assets/Pasted%20image%2020240320233941.png)

## Cglib 和 JDK 动态代理 #rep

> #todo 原理？ #小红书_电商_实习_后端
> 
> 除了 JDK 动态代理，还有什么动态代理方法？ #小红书_电商_实习_后端

在Java中，实现动态代理有两种方式：

1. *JDK 动态代理*：`Java.lang.reflect` 包中的 Proxy 类和 InvocationHandler 接口提供了生成动态代理类的能力
2. *Cglib 动态代理*：Cglib (Code Generation Library )是一个**第三方代码生成类库**，运行时在内存中动态生成一个**子类对象**从而实现对目标对象功能的扩展

> 有什么区别？ #PDD_23_秋招_后端 

- 使用 JDK 动态代理的对象必须实现一个或多个接口；

- 而使用 cglib 代理的对象则无需实现接口，达到代理类无侵入（底层是通过使用一个小而快的字节码处理框架 ASM，来转换字节码并生成新的类）
	- 通过继承的方式做的动态代理，如果某个类被标记为 final，那么它是无法使用 CGLIB 做动态代理的

> 动态代理和静态代理的区别 #携程

最大的区别：静态代理是**编译期**确定的，动态代理是**运行期**确定的

使用静态代理模式需要程序员手写很多代码，这个过程是比较浪费时间和精力的。一旦需要代理的类中方法比较多，或者需要同时代理多个对象的时候，这无疑会增加很大的复杂度

反射是动态代理的实现方式之一

> JDK 代理对象与被代理对象之间是什么关系？ #小红书_电商_实习_后端

#todo 

实现了同一个接口，代理对象依赖被代理对象（组合关系）

> 动态代理的用途

Java 的动态代理的最主要的用途就是应用在各种框架中。因为使用动态代理可以很方便的运行期生成代理类，通过代理类可以做很多事情，比如 **AOP**，比如**过滤器**、**拦截器**、**mybatis 分页插件**，以及**日志**拦截、**事务**拦截、**权限**拦截这些几乎全部由动态代理的身影

## 多个切面的执行顺序如何控制？

1）通知类上使用 `@Order` 注解直接定义切面顺序

```java
// 值越小优先级越高
@Order(3)
@Component
@Aspect
public class LoggingAspect implements Ordered {
```

2）通知类实现 `Ordered` 接口重写 `getOrder()` 方法

```java
@Component
@Aspect
public class LoggingAspect implements Ordered {

    // ....

    @Override
    public int getOrder() {
        // 返回值越小优先级越高
        return 1;
    }
}
```

## Spring AOP 在什么场景下会失效？ #rep

#todo_完善

> 私有方法可以被代理吗(1)

Spring 的 AOP 是通过动态代理实现的，如果失效了说明==没有调用到代理对象的方法==

- 私有方法调用
- 静态方法调用
- final方法调用
- 类内部自调用（被调用方代理失效）
	- 不要嵌套调用，将需要一个类中嵌套调用的方法，移到另一个类中
	- 在代码中使用 AopContext 获取到代理对象，使用代理对象进行嵌套调用
- 内部类方法调用

[Spring专题: 4. AOP失效的场景与原理 - 简书 (jianshu.com)](https://www.jianshu.com/p/5df09b132abd)

# 事务

- 事务作用：在数据层保障一系列的数据库操作同成功同失败
- Spring 事务作用：在数据层或业务层保障一系列的数据库操作同成功同失败

## Spring 管理事务的方式有几种？ #rep

- **编程式事务**：在代码中硬编码(在分布式系统中推荐使用) : 通过 `TransactionTemplate` 或者 `TransactionManager` 手动管理事务，事务范围过大会出现事务未提交导致超时，因此事务要比锁的粒度更小。
- **声明式事务**：在 XML 配置文件中配置或者直接基于注解（单体应用或者简单业务系统推荐使用） : 实际是通过 AOP 实现（基于 `@Transactional` 的全注解方式使用最多）

## spring 事务有哪些传播行为？ #rep

#PDD_23_秋招_后端 

事务传播行为是为了**解决业务层方法之间互相调用的事务问题**

当事务方法被另一个事务方法调用时，必须**指定事务应该如何传播**（例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行）

正确的事务传播行为可能的值，即 `@Transactional` 的 Propagation 属性如下:

- *REQUIRED*（默认）：如果当前存在事务，则**加入**该事务；否则创建一个新的事务

- *REQUIRES_NEW*（独立新事务）：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
	- 不管外部方法是否开启事务，内部方法会新开启自己的事务，且**相互独立，互不干扰**

- *NESTED*（嵌套）：如果当前存在事务，则创建一个事务作为当前事务的**嵌套事务**来运行，否则创建一个新的事务
	- 嵌套事务出错回滚不会影响到主事务（部分回滚），主事务回滚会将嵌套事务一起回滚了

- *MANDATORY*（强制）：如果当前存在事务，则**加入**该事务，否则抛出异常（很少用）

若是错误的配置以下 3 种事务传播行为，事务将不会发生回滚：

- *SUPPORTS*（有就支持一下）：如果当前存在事务，则加入该事务，否则以**非事务**的方式继续运行
- *NOT_SUPPORTED*：以**非事务**方式运行，如果当前存在事务，则把当前事务**挂起**
- *NEVER*：以**非事务**方式运行，如果当前存在事务，则**抛出异常**

## Spring 事务的隔离级别 #rep 

- *DEFAULT*：使用后端数据库默认的隔离级别，
	- MySQL 默认 `REPEATABLE_READ`，Oracle 默认 `READ_COMMITTED`

- *READ_UNCOMMITTED*：最低的隔离级别，允许读取尚未提交的数据变更
	- 可能发生：脏读、幻读、不可重复读（通常不会用）

- *READ_COMMITTED*：允许读取并发事务已经提交的数据，不会发生脏读
	- 可能发生：幻读、不可重复读

- *REPEATABLE_READ*：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，不会发生脏读、不可重复读
	- 可能发生：幻读

- *SERIALIZABLE*（可串行化）：最高的隔离级别，完全服从 ACID 的隔离级别。所有的事务依次逐个执行，事务之间不可能产生干扰，但严重影响程序的性能（通常不会用）

## 事务注解的 rollbackFor 属性 #rep

`@Transactional(rollbackFor = Exception.class)` 设置事务的回滚策略

默认回滚策略：只有遇到 RuntimeException (运行时异常) 或者 Error 时才会回滚事务，而不会回滚 Checked Exception（受检查异常）

> 这是因为 Spring 认为 RuntimeException 和 Error 是不可预期的错误，而受检异常是可预期的错误，可以通过业务逻辑来处理

修改默认的回滚策略：rollbackFor 和 noRollbackFor 属性指定需要/不要回滚的异常

## 事务失效


# 设计模式

## Spring 框架有哪些比较经典的设计模式 #秋招24 #rep

- **工厂设计模式** : Spring 使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象 #todo
- **代理设计模式** : Spring AOP 功能的实现 #todo
- **单例设计模式** : Spring 中的 Bean 默认都是单例的 #todo
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式 #todo
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源 #todo
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用 #todo
- **适配器模式** : Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配 `Controller` #todo


## 工厂模式 #rep

> BeanFactory 和 FactoryBean 有什么区别？

==Spring IOC 容器就像是一个工厂一样==，我们完全不用考虑对象是如何被创建出来的。 IOC 容器负责创建、管理对象（生命周期、作用域）

**1）通过 BeanFactory 或 ApplicationContext 创建 bean 对象**

- **BeanFactory**：IoC 容器的顶层接口
	- 特点：延迟加载（获取 bean 的时候才创建）
	- 优势：相比于 ApplicationContext 来说会占用更少的内存，程序启动速度更快

- **ApplicationContext**：BeanFactory 的子接口
	- 特点：容器启动的时候，一次性创建所有 bean（可以配置延迟加载）
	- 优势：`BeanFactory` 仅提供了最基本的依赖注入支持，ApplicationContext 扩展了 BeanFactory
	- 实现类：
		- ClassPathXmlApplication：从类路径下的 XML 文件载入上下文定义信息
		- FileSystemXmlApplication：从文件系统中的 XML 文件载入上下文定义信息
		- XmlWebApplicationContext：从 Web 系统中的 XML 文件载入上下文定义信信息
		- AnnotationConfigApplicationContext

> #Bo 工厂方法模式，BeanFactory、ApplicationContext 是抽象工厂，底层实现是具体工厂

**2）FactoryBean**

FactoryBean 是一个==接口==，用于定义一个**工厂 Bean**，FactoryBean 接口的类==造出来的对象不是当前类的对象，而是**泛型类型**的对象==

- 如果某个 Bean 实现了 FactoryBean 接口，那么 Spring 容器不直接返回这个 Bean 实例，而是返回 `FactoryBean#getObject()` 方法所返回的对象

FactoryBean 通常用于创建过程**很复杂**的对象

- 这种方式在 Spring 去**整合其他框架**的时候会被用到。例如，创建与 JNDI 资源的连接或与代理对象的创建。就如我们的 Dubbo 中的 ReferenceBean #todo

> [✅BeanFactory和FactroyBean的关系？ (yuque.com)](https://www.yuque.com/hollis666/krcpbs/cnhqfg#tTg7F)

```java
public class UserDaoFactoryBean implements FactoryBean<UserDao> {
    //代替原始实例工厂中创建对象的方法
    public UserDao getObject() throws Exception {
        return new UserDaoImpl();
    }
    
    //返回所创建类的Class对象
    public Class<?> getObjectType() {
        return UserDao.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```


# 循环依赖

## 循环依赖 Spring 循环依赖了解吗，怎么解决？

#饿了么 #蔚来

循环依赖是指 Bean 对象循环引用，是两个或多个 Bean 之间相互持有对方的引用

Spring 框架通过使用三级缓存来解决这个问题，确保即使在循环依赖的情况下也能正确创建 Bean。三级缓存其实就是三个 Map：
- **一级缓存（singletonObjects）**：
	- 存放最终形态的 Bean（已经实例化、属性填充、初始化），单例池，为“Spring 的单例属性”而生。
	- 一般情况我们获取 Bean 都是从这里获取的，但是并不是所有的 Bean 都在单例池里面，例如原型 Bean
- **二级缓存（earlySingletonObjects）**：
	- 存放过渡 Bean（半成品，尚未属性填充），也就是==三级缓存中 `ObjectFactory` 产生的对象==，
	- 与三级缓存配合使用的，可以防止 AOP 的情况下，每次调用 `ObjectFactory#getObject()` 都是会产生新的代理对象的。
- **三级缓存（singletonFactories）**：
	- 存放 `ObjectFactory`，
		- `ObjectFactory` 的 `getObject()` 方法（最终调用的是 `getEarlyBeanReference()` 方法）可以生成原始 Bean 对象或者代理对象（如果 Bean 被 AOP 切面代理）
	- 三级缓存只会对单例 Bean 生效

```java
// 一级缓存
/** Cache of singleton objects: bean name to bean instance. */
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

// 二级缓存
/** Cache of early singleton objects: bean name to bean instance. */
private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);

// 三级缓存
/** Cache of singleton factories: bean name to ObjectFactory. */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
```


# 未知

## spring 怎么去做监控，错误码体系？

#PDD_23_秋招_后端 


# ---------- Spring MVC

# 概念

## MVC

MVC 是模型(Model)、视图(View)、控制器(Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码，是一种设计模式。

这种模式可以可以促进代码的复用和分工，从而提高代码的可读性和可维护性

![500](assets/Pasted%20image%2020240321121454.png)

## Spring MVC

Spring MVC 是一款很优秀的 MVC 框架，可以帮助我们进行更**简洁的 Web 层的开发**

- 对 Servlet 进行了封装，相比于 Servlet，使用更简单，开发更便捷
- 与 Spring 框架集成

> Spring MVC 下我们一般把后端项目分为 Service 层（处理业务）、Dao 层（数据库操作）、Entity 层（实体类）、Controller 层(控制层，返回数据给前台页面)

## Spring MVC 的核心组件 #rep

记住了下面这些组件，也就记住了 SpringMVC 的工作原理。

- `DispatcherServlet`：**核心的中央处理器**，负责接收请求、分发，并给予客户端响应
- `HandlerMapping`：**处理器映射器**，根据 URL 去匹配查找能处理的 `Handler` ，并会将请求涉及到的拦截器和 `Handler` 一起封装
- `HandlerAdapter`：**处理器适配器**，根据 `HandlerMapping` 找到的 `Handler` ，适配执行对应的 `Handler`
- `Handler`：**请求处理器**，处理实际请求的处理器
- `ViewResolver`：**视图解析器**，根据 `Handler` 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 `DispatcherServlet` 响应客户端

# 原理

## SpringMVC 是如何将不同的 Request 路由到不同 Controller 中的？ #rep

> 在计算机程序处理中，但凡涉及到路由，那包含到的数据结构一定是和 map 相关的。所以对于 url 和 controller 之间的映射，如果交给我们来设计的话，可能会用一个大的 **map 将 url 和 controller 中对应的方法作为键值对**存储起来，以此来达到路由的目的。

![600](assets/Pasted%20image%2020240321185323.png)

Spring MVC 在启动的时候，会把带有 `@RequestMapping` 注解的方法和类封装成 RequestMappingInfo 和 HandlerMethod，然后注册到 **MappingRegistry**

- MappingRegistry 存放了一个 **HashMap**，key 是 RequestMappingInfo，value 是 HandlerMethod
- HandlerMethod 封装了对应的 Method（controller 的方法的**反射方法对象**） 和持有它的 **Bean**（controller） 的名字

![](assets/Pasted%20image%2020240321192147.png)

http 请求进入 tomcat，解析 request 中的数据，拿到对应的 **HandlerMethod**，这一步就是
HandlerMapping 的 `getHandler()` 方法

```java
HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;
```

当 HttpServletRequest 访问时，会通过 `AbstractHandlerMethodMapping#lookupHandlerMethod()` 方法获取对应的 `HandlerMethod`

类之间的关系总图

![](assets/Pasted%20image%2020240321192118.png)


## Spring MVC 的执行流程(1) #rep

#小红书_暑期实习_Java后端 #PDD_23_秋招_后端 

对于 Http 请求来说，

- tomcat 执行了 HttpServlet 的 `service()` 方法，
- 继承了 HttpServlet 的 FrameWorkServlet 则是执行 `doService()` 方法
- DispatcherServlet **继承了 FrameworkServlet**

进入到 Spring MVC 的流程中，在 DispatcherServlet 中的流程如下：

1）请求到达 **DispatcherServlet** 的 `doDispatch()` 方法

2）根据 **HandlerMapping** 拿到 request 对应的 HandlerAdapter 对象，并且返回 **HandlerExecutionChain**

- `getHandler()` 方法中，先通过 `HandlerMapping` 拿到 request 对应的 HandlerExecutionChain
	- HandlerExecutionChain 封装了 HandlerMethod 和 存放拦截器的 ArrayList
- `getHandlerAdapter()` 中 拿到 HandlerMethod 对应的 HandlerAdapter，

3）通过 **HandlerExecutionChain** 执行拦截器的 `prehandle()` 方法（责任链模式）

4）通过 **HandlerAdapter** 去执行 **HandlerMethod**，返回一个 **ModelAndView** 对象

> - handlerMethod 里面封装的映射的真正方法 handler 还有可能是原生的 Servlet，要执行 `handler.invoke`
> - 在这之前要去判断参数，通过参数解析器 HandlerMethodArgumentResolver
> - 反射调用完之后，需要调用返回值解析器 `HandlerMethodReturnValueHandler`（适配器模式&组合模式&策略模式）

> #Bo 返回 json 数据，mav 是空的

5）通过 **HandlerExecutionChain** 执行拦截器的 `posthandle()` 方法

> Spring MVC 执行完之后返回的是 `ModelAndView`，我们还需要对 `ModelAndView` 进行 render，即把 `ModelAndView` 中的 view 渲染到 response 中

6）当发生异常时，会将异常拉到 用户定制 的**异常处理方法**中

- ==用户标记的 `@ExceptionHandler` 方法==已经被 ExceptionHandlerMethodResolver 找到，并且注册（==key 为对应异常，value 为对应方法==），只需要调用该方法就可以对异常进行处理，此时的方法调用和之前的 handler 几乎没有区别 #todo完善

![](assets/Pasted%20image%2020240321131100.png)

## 全局异常处理器怎么做的（项目）？ #rep 

#小红书_实习_后端 

`@RestControllerAdvice` (包含 `@ResponseBody`, `@Component`) 配合 `@ExceptionHandler`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(SystemException.class)
    public Result doSystemException(SystemException ex) {
        return new Result(ex.getCode(), null, ex.getMessage());
    }

    @ExceptionHandler(BusinessException.class)
    public Result doBusinessException(BusinessException ex) {
        return new Result(ex.getCode(), null, ex.getMessage());
    }
    
    @ExceptionHandler(Exception.class)
    public Result doOtherException(Exception ex) {
        return new Result(Code.SYSTEM_UNKNOW_ERR, null, "其他异常：系统繁忙，请稍后再试！");
    }
}
```

这种异常处理方式下，会给所有或者指定的 Controller 织入异常处理的逻辑（AOP），当 Controller 中的方法抛出异常时，由被 `@ExceptionHandler` 注解修饰的方法进行处理

项目中的异常处理方式：

- 自定义异常，
	- 自定义异常类**继承 RuntimeException** ，RuntimeException 不用显示处理
	- 添加 code 属性，便于区分异常来自的业务

- 对异常分类：
	- *业务异常*（BusinessException）：**用户行为**产生的异常
		- 解决方案：发送消息给用户，提醒规范操作
	- *系统异常*（SystemException）：**可预计但无法避免**的异常，比如数据库或服务器宕机
		- 解决方案：发送固定消息安抚用户、日志、提醒运维
	- *其他异常*（Exception）：编程人员**未预期**到的异常
		- 解决方案：同上

- 业务中 throw 自定义异常

![500](assets/Pasted%20image%2020240219204200.png)

# ---------- SpringBoot

# 概念

## Spring、Spring MVC、Spring Boot 之间什么关系?

> Spring Boot 和 Spring 有哪些区别？(1) #字节_23_秋招_Java 
>
> SpringBoot 和 SpringMVC 的区别？ #顺丰_23_秋招_Java 

Spring 包含了多个功能模块，其中最重要的是 **Spring-Core**（主要提供 IoC 依赖注入功能的支持） 模块， Spring 中的其他模块（比如 Spring MVC）的功能实现基本都需要依赖于该模块。

Spring MVC 是 Spring 中的一个很重要的模块，主要**赋予 Spring 快速构建 MVC 架构的 Web 程序的能力**。MVC 是模型 (Model)、视图 (View)、控制器 (Controller)的简写，其核心思想是通过将业务逻辑、数据、显示分离来组织代码

> 使用 Spring 进行开发各种配置过于麻烦比如开启某些 Spring 特性时，需要用 XML 或 Java 进行显式配置。于是，Spring Boot 诞生了！

Spring 旨在简化 J2EE 企业应用程序开发。Spring Boot **旨在简化 Spring 开发**（减少配置文件，开箱即用！）

Spring Boot 只是简化了配置，如果你需要构建 MVC 架构的 Web 程序，你还是需要使用 Spring MVC 作为 MVC 框架，只是说 Spring Boot 帮你**简化了 Spring MVC 的很多配置**，真正做到开箱即用！

## 为什么要有 SpringBoot（优点）？ #rep 

Spring 旨在简化 J2EE 企业应用程序开发。Spring Boot **旨在简化 Spring 开发**（减少配置文件，开箱即用！）

- *起步依赖*：帮我们规定了依赖的版本号（parent、dependencyManagement）、固定搭配（starter）
- *自动配置*：减少了样板代码、XML 配置、注解
- *统一配置文件*：简化配置、修改默认配置
- *辅助功能*：提供嵌入式 HTTP 服务器、如 Tomcat 和 Jetty
- *命令行接口（CLI）工具*：开发和测试 SpringBoot 程序
- *插件*：使用内置工具（maven、Gradle）开发和测试 SpringBoot 程序

## spring-boot-starter是什么？

starter 定义了使用某种技术时对于**依赖的固定搭配格式**，也是一种最佳解决方案，使用 starter 可以帮助开发者**减少依赖配置**

# 原理

## springboot 怎么实现自动化配置？ #rep 

#PDD_23_秋招_后端 

自动配置：Spring Boot 会根据类路径中的 jar 包、类、bean，自动地注册一些 Bean 到 Spring 的 Context 中

SpringBoot 通过 Spring 的条件配置决定哪些 bean 可以被配置，将这些条件定义成具体的 `xxxAutoConfiguration`，然后将这些 Configuration 配置到 `spring.factories` 文件中

![500](assets/Pasted%20image%2020240322154048.png)

---
主配置类上 `@SpringBootApplication` 注解包含了以下三个注解：

- `@EnableAutoConfiguration`：启用 springboot 的自动配置
- `@SpringBootConfiguration`：包含 `@Configuration`
- `@ComponentScan`：默认扫描该类所在的包下的所有的类

```java 
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),  
       @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })  
public @interface SpringBootApplication
```

`@EnableAutoConfiguration` 是启动自动配置的关键，其中 `@Import` 导入了 AutoConfigurationImportSelector 类，类中的 `getCandidateConfigurations()` 方法决定了要注册哪些自动配置类

- 通过读取类路径下 `META-INF\spring.factories` 文件中的 `EnableAutoConfiguration` 配置项，将所有自动配置类的全类名以 List 的形式返回
- List过滤
	- 自动配置类上的 `@ConditionalOnXxx` 注解，会根据类路径中的 jar 包、类、容器中的bean，决定是否要加载
	- exclude 属性主动剔除
- 满足条件的自动配置类会被注册到容器里，生效

```java
@AutoConfigurationPackage  
@Import(AutoConfigurationImportSelector.class)  
public @interface EnableAutoConfiguration
```

自动配置类会导入配置属性类，配置属性类上通过 `@ConfigurationProperties` 注解和 Application 文件绑定，替换默认的配置值




# 未知

## 如果小红书内部需要做一个 starter，你会从哪些方面去考虑、设计

#小红书_实习_后端 

## 你有用过 Spring Boot 提供的哪些钩子函数或者特性来方便编码的吗(1)




# ---------- MyBatis

Mybatis 和 MybtisPlus，在使用 mybaits 中设计到的端口问题 #得物_24_后端 


# ---------- 未知

注解机制 #小红书_23_秋招_后端 


