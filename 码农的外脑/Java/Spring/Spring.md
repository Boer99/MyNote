
# 参考资料

视频
- 尚硅谷Spring（lj）
- 黑马2022SSM（nb）

推荐文章
- IOC 源码：[Spring IOC 容器源码分析_Javadoop](https://javadoop.com/post/spring-ioc)
- springaop 和 aspectj 的区别 [springAOP 和 aspectJ 有什么区别_spring aop and aspectj aop 有什么区别?-CSDN博客](https://blog.csdn.net/u013452337/article/details/100981702)


# Spring 介绍

## 为什么学？

从使用和占有率看
- Spring 在市场的占有率与使用率高
- Spring 在企业的技术选型命中率高
- 所以说，Spring 技术是 JavaEE 开发必备技能，企业开发技术选型命中率 **>90%**

从专业角度看
- 随着时代发展，软件规模与功能都呈几何式增长，开发难度也在不断递增，该如何解决?
    - Spring 可以==简化开发==，降低企业级开发的复杂性，使开发变得更简单快捷
- 随着项目规模与功能的增长,遇到的问题就会增多，为了解决问题会引入更多的框架，这些框架如何协调工作?
    - Spring 可以==框架整合==，高效整合其他技术，提高企业级应用开发与运行效率

## 初识 Spring

### Spring 家族

- 官网：[https://spring.io](https://spring.io)，从官网我们可以大概了解到：
    - Spring 能做什么：用以开发 web、微服务以及分布式系统等，光这三块就已经占了 JavaEE 开发的九成多。
    - **Spring 并不是单一的一个技术，而是一个大家族**，可以从官网的 `Projects` 中查看其包含的所有技术。  
- Spring 发展到今天已经形成了一种开发的生态圈，Spring 提供了若干个项目，每个项目用于完成特定的功能。
    - Spring 已形成了完整的生态圈，也就是说我们可以完全使用 Spring 技术完成整个项目的构建、设计与开发。
    - Spring 有若干个项目，可以根据需要自行选择，把这些个项目组合起来，起了一个名称叫==全家桶==

![](assets/Pasted%20image%2020240207161035.png)

这些技术并不是所有的都需要学习，额外需要重点关注
- *Spring Framework*：Spring 框架，是 Spring 中最早最核心的技术，也是所有其他技术的基础。
- *SpringBoot*：Spring 是来简化开发，而 SpringBoot 是来帮助 Spring 在简化的基础上能更快速进行开发。
- *SpringCloud*：这个是用来做分布式之微服务架构的相关开发。

除了上面的这三个技术外，还有很多其他的技术，也比较流行，如 SpringData、SpringSecurity 等，这些都可以被应用在我们的项目中。

**我们今天所学习的 Spring 其实指的是 Spring 家族中 Spring Framework。**

Spring Framework 是 Spring 家族中其他框架的底层基础

###  Spring 发展史

![](assets/Pasted%20image%2020240207161637.png)

- IBM(IT 公司-国际商业机器公司)在 1997 年提出了 EJB 思想，早期的 JAVAEE 开发大都基于该思想。
- Rod Johnson(Java 和 J2EE 开发领域的专家)在2002年出版的 `Expert One-on-One J2EE Design and Development`，书中有阐述在开发中使用 EJB 该如何做。
- Rod Johnson 在2004年出版的 `Expert One-on-One J2EE Development without EJB`,书中提出了比 EJB 思想更高效的实现方案，并且在同年将方案进行了具体的落地实现，这个实现就是 Spring1.0。
- 随着时间推移，版本不断更新维护，目前最新的是 Spring5
    - Spring1.0 是纯配置文件开发
    - Spring2.0 为了简化开发引入了注解开发，此时是配置文件加注解的开发方式
    - Spring3.0 已经可以进行纯注解开发，使开发效率大幅提升，
	    - 课程会以注解开发为主
    - Spring4.0 根据 JDK 的版本升级对个别 API 进行了调整
    - Spring5.0 已经全面支持 JDK8，
	    - 现在 Spring 最新的是 5 系列，所以建议把 JDK 安装成 1.8 版

### Spring 系统架构

Spring Framework 是 Spring 生态圈中最基础的项目，是其他项目的根基。它的发展也经历了很多版本的变更，每个版本都有相应的调整。

5 版本目前没有最新的架构图，而最新的是 4 版本，所以接下来主要研究的是 4 的架构图

![](assets/Pasted%20image%2020240207162221.png)

> 上层依赖下面的层，除了 Test

- 核心层
	- *Core Container*：核心容器，**这个模块是 Spring 最核心的模块**，其他的都需要依赖该模块
- AOP 层
	- *AOP*：面向切面编程，它依赖核心层容器，目的是在不改变原有代码的前提下对其进行功能增强
	- *Aspects*：AOP 是思想，Aspects 是对 AOP 思想的具体实现
- 数据层
	- *Data Access*：数据访问，Spring 全家桶中有对数据访问的具体实现技术
	- *Data Integration*：数据集成，Spring 支持整合其他的**数据层解决方案**，比如 Mybatis
	- *Transactions*：事务，Spring 中事务管理是 **Spring AOP 的一个具体实现**，也是后期学习的重点内容
- Web 层
	- 这一层的内容将在 SpringMVC 框架具体学习
- Test 层
	- Spring 主要整合了 Junit 来完成单元测试和集成测试

> Spring是轻量级（体积小，引入的jar包少）的开源（免费提供源代码）的JavaEE框架（开发更简便，代码更简洁）
> 
> Spring可以解决企业应用开发的复杂性
> 
> Spring有两个核心部分：
> - ``IOC``：控制反转，把创建对象过程交给Spring进行管理（原始方式：new一个对象）
> - ``Aop``：面向切面，不修改源代码进行功能增强
> 
> Spring特点
> - 方便解耦，简化开发
> - Aop编程支持
> - 方便程序测试
> - 方便和其他框架进行整合
> - 方便进行事务操作
> - 降低JAVAEE API开发难度

学习路线
- Spring 的 IOC/DI
- Spring 的 AOP
- AOP 的具体应用，事务管理
- IOC/DI 的具体应用，整合 Mybatis

![](assets/Pasted%20image%2020240207162729.png)

## Spring 核心概念

### 引入

现在代码在编写的过程中存在的问题是：**耦合度偏高**
- 业务层需要调用数据层的方法，就需要在业务层 new 数据层的对象
- 如果数据层的实现类发生变化，那么业务层的代码也需要跟着改变，发生变更后，都需要进行编译打包和重部署

![](assets/Pasted%20image%2020240207164959.png)

如果能把框中的内容给去掉，不就可以降低依赖了么，但是又会引入新的问题，去掉以后因为 bookDao 没有赋值为 Null，强行运行就会出空指针异常。

所以现在的问题就是：业务层不想 new 对象，运行的时候又需要这个对象，该咋办呢?

针对这个问题，Spring 就提出了一个解决方案：使用对象时，在程序中不要主动使用 new 产生对象，转换为**由外部提供对象**

这种实现思想就是Spring的一个核心概念

### IOC、IOC 容器、Bean、DI

IOC（Inversion of Control），控制反转
- 使用对象时，由主动 new 产生对象转换为由==外部==提供对象，此过程中**对象创建控制权**由程序转移到外部，此思想称为控制反转

Spring 与 IOC 之间的关系：Spring 技术对 IOC 思想进行了实现
- Spring 提供了一个容器，称为==IOC 容器==，用来充当 IOC 思想中的"外部"
- IOC 思想中的 **别人[外部]** 指的就是 Spring 的 IOC 容器

IOC 容器负责对象的创建、初始化等一系列工作，其中包含了数据层和业务层的类对象
- 被创建或被管理的对象在 IOC 容器中统称为==Bean==
- IOC 容器中放的就是一个个的 Bean 对象

DI（Dependency Injection）依赖注入
- **在容器中建立 bean 与 bean 之间的依赖关系的整个过程，称为依赖注入**

---
最终目标就是：**充分解耦**
- 使用 IOC 容器管理 bean（IOC)
- 在 IOC 容器内将有依赖关系的 bean 进行关系绑定（DI）

最终结果：使用对象时不仅可以直接从 IOC 容器中获取，并且获取到的 bean 已经绑定了所有的依赖关系.

# 入门案例

## IOC 入门案例

思路分析：
- Spring 是使用容器来管理 bean 对象的，那么管什么?
	- 主要管理项目中所使用到的类对象，比如(Service 和 Dao)
- 如何将被管理的对象告知 IOC 容器?
	- 使用配置文件
- 被管理的对象交给 IOC 容器，要想从容器中获取对象，就先得思考如何获取到 IOC 容器?
	- Spring 框架提供相应的接口
- IOC 容器得到后，如何从容器中获取 bean?
	- 调用 Spring 框架提供对应接口中的方法
- 使用 Spring 导入哪些坐标?
	- 用别人的东西，就需要在 pom.xml 添加对应的依赖

---
依赖
```xml
<dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.10.RELEASE</version>
    </dependency>
```

定义 Spring 管理的类（接口）
```java
public interface BookDao {
    public void save();
}
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
}
public interface BookService {
    public void save();
}
public class BookServiceImpl implements BookService {
    private BookDao bookDao = new BookDaoImpl();
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

创建 Spring 配置文件，配置作为 Spring 管理的 Bean
- id 属性在同一个上下文中不能重复
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
    bean标签标示配置bean
        id属性标示给bean起名字
        class属性表示给bean定义类型
    -->
    <bean id="bookDao" class="org.example.dao.impl.BookDaoImpl"/>
    <bean id="bookService" class="org.example.service.impl.BookServiceImpl"/>
</beans>
```

初始化 IOC 容器，通过容器获取 Bean
```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext ctx=new ClassPathXmlApplicationContext("applicationContext.xml");

        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        BookService bookService=(BookService) ctx.getBean("bookService");

        bookDao.save();
        bookService.save();
    }
}
```

## DI 入门案例

> 在 `BookServiceImpl` 的类中依然存在 `BookDaoImpl` 对象的 new 操作，它们之间的耦合度还是比较高，这块该如何解决

思路分析
- 基于 IoC 管理 bean
- Service 中使用 new 形式创建的 Dao 对象是否保留? (否)
- Service 中需要的 Dao 对象如何进入到 Service 中? (提供方法)
- Service 与 Dao 间的关系如何描述? (配置)

---
删除使用 new 创建对象的代码；提供依赖对象对应的 setter 方法
```java
public class BookServiceImpl implements BookService {

//    private BookDao bookDao = new BookDaoImpl();
    private BookDao bookDao;

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
}
```

配置 service 与 dao 之间的关系
```xml
<bean id="bookDao" class="org.example.dao.impl.BookDaoImpl"/>

<bean id="bookService" class="org.example.service.impl.BookServiceImpl">
	<!-- 配置server与dao的关系 -->
	<!--
		property标签表示配置当前bean的属性
			name属性表示配置哪一个具体的属性
			ref属性表示参照哪一个bean
	-->
	<property name="bookDao" ref="bookDao"/>
</bean>
```

# IOC and DI 相关内容

## ----- IOC

## bean 基础配置

### id 与 class 属性

```xml
<bean id="" class=""/>
```

![](assets/Pasted%20image%2020240207183830.png)

### name 属性

![](assets/Pasted%20image%2020240207184009.png)

bean 别名配置
- ref 属性值也可以是另一个 bean 的 name 属性值，但还是建议使用 id 属性值
```xml
<!--name:为bean指定别名，别名可以有多个，使用逗号，分号，空格进行分隔-->
<bean id="bookService" name="service service4 bookEbi" class="org.example.service.impl.BookServiceImpl">
	<property name="bookDao" ref="bookDao"/>
</bean>

<!--scope：为bean设置作用范围，可选值为单例singloton，非单例prototype-->
<bean id="bookDao" name="dao" class="org.example.dao.impl.BookDaoImpl"/>
```

别名获取 Bean
- bean 依赖注入的 ref 属性指定 bean，必须在容器中存在。
- 如果不存在，抛出异常 `NoSuchBeanDefinitionException`
```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext ctx=new ClassPathXmlApplicationContext("applicationContext.xml");

        BookService bookService=(BookService) ctx.getBean("service4");
        BookService bookService2=(BookService) ctx.getBean("bookEbi");

        bookService.save();
        bookService2.save();
    }
}
```

### scope 属性

![](assets/Pasted%20image%2020240207210318.png)

使用 bean 的 `scope` 属性可以控制 bean 的创建是否为单例：
- `singleton` **默认**为单例，在 Spring 的 IOC 容器中只会有该类的一个对象
- `prototype` 为非单例

为什么 bean 默认为单例?
- 避免了对象的频繁创建与销毁，达到了 bean 对象的复用，性能高

哪些 bean 对象 适合/不适合 交给容器进行管理?
- 适合：
	- 表现层对象
	- 业务层对象
	- 数据层对象
	- 工具对象
- 不适合：封装实例的域对象，因为会引发线程安全问题，所以不适合。

bean 在容器中是单例的，会不会产生线程安全问题?
- 如果对象是有状态对象，即该对象有成员变量可以用来存储数据的，
	- 因为所有请求线程共用一个 bean 对象，所以会存在线程安全问题。
- 如果对象是无状态对象，即该对象没有成员变量没有进行数据存储的，
	- 因方法中的局部变量在方法调用完成后会被销毁，所以不会存在线程安全问题。

## bean 实例化

> 对象已经能交给 Spring 的 IOC 容器来创建了，但是容器是如何来创建对象的呢?

bean 本质上就是对象，对象在 new 的时候会使用构造方法完成，那创建 bean 也是使用**构造方法**完成的。

实例化 bean 的 3/4 种方式
- 构造方法 (常用)
- 静态工厂 (了解)
- 实例工厂 (了解)
	- **FactoryBean** (实用)

### 构造方法实例化

无参构造如果不存在，将抛出异常 `BeanCreationException`

构造方法在类中默认会提供，但是如果重写了构造方法，默认的就会消失，在使用的过程中需要注意，如果需要重写构造方法，最好把默认的构造方法也重写下。

---
使用 private 无参构造函数，能执行成功并打印，能访问到类中的私有构造方法,显而易见 Spring 底层用的是**反射**

```java
public class BookDaoImpl implements BookDao {
    private BookDaoImpl() {
        System.out.println("book dao constructor is running ....");
    }
    public void save() {
        System.out.println("book dao save ...");
    }
}
```

构造函数中添加一个参数测试，程序会报错，说明 Spring 底层使用的是类的无参构造方法。

```java
public class BookDaoImpl implements BookDao {
    private BookDaoImpl(int i) {
        System.out.println("book dao constructor is running ....");
    }
}
```

```bash
Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'bookDao' defined in class path resource [applicationContext.xml]: Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.itheima.dao.impl.BookDaoImpl]: No default constructor found; nested exception is java.lang.NoSuchMethodException: com.itheima.dao.impl.BookDaoImpl.`<init>`()。
```

> Spring 的报错信息**从下往上依次查看**，因为上面的错误大都是对下面错误的一个包装，最**核心错误是在最下面**

### 静态工厂实例化

静态工厂创建对象
```java
public class OrderDaoFactory {
    public static OrderDao getOrderDao(){
	    // 其他操作
        return new OrderDaoImpl();
    }
}
```

配置文件
- class：工厂类的全类名
- factory-method：具体工厂类中创建对象的方法名
```java
<bean id="orderDao" class="com.itheima.factory.OrderDaoFactory" factory-method="getOrderDao"/>
```

![](assets/Pasted%20image%2020240207234614.png)

> 你这种方式在工厂类中不也是直接new对象的，和我自己直接new没什么太大的区别，而且静态工厂的方式反而更复杂，这种方式的意义是什么?

在工厂的静态方法中，我们除了 new 对象还可以做其他的一些业务操作。

> 这种方式一般是用来兼容早期的一些老系统，所以==了解为主==

### 实例工厂实例化

实例工厂
```java
public class UserDaoFactory {
    public UserDao getUserDao(){
        return new UserDaoImpl();
    }
}
```

配置文件
- 因为不再是静态方法，而是实例方法，所以工厂本身也要实例化
- 属性：
	- factory-bean：工厂的实例对象
	- factory-method：工厂类中的创建对象的方法名

```xml
<bean id="userFactory" class="com.itheima.factory.UserDaoFactory"/>

<bean id="userDao" factory-method="getUserDao" factory-bean="userFactory"/>
```

![](assets/Pasted%20image%2020240208014224.png)

IOC 容器中获取 bean
```java
public class AppForInstanceUser {
    public static void main(String[] args) {
        ApplicationContext ctx = new 
            ClassPathXmlApplicationContext("applicationContext.xml");
        UserDao userDao = (UserDao) ctx.getBean("userDao");
        userDao.save();
    }
}
```

> 实例工厂实例化的方式就已经介绍完了，配置的过程还是比较复杂，所以 Spring 为了**简化这种配置方式**就提供了一种叫 `FactoryBean` 的方式来简化开发。

%% -- %%
### FactoryBean

> 这种方式在 Spring 去**整合其他框架**的时候会被用到，重点掌握！

创建一个 FactoryBean 接口的实现类，重写接口的方法

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
}
```

配置文件

```java
<bean id="userDao" class="com.itheima.factory.UserDaoFactoryBean"/>
```

运行类不用做任何修改

---
查看源码会发现，FactoryBean 接口其实会有三个方法

```java
T getObject() throws Exception;

Class<?> getObjectType();

default boolean isSingleton() {
		return true;
}
```

- `getObject()`，被重写后，在方法中进行对象的创建并返回
- `getObjectType()`，被重写后，主要返回的是被创建类的 Class 对象
- `isSingleton()`，已经给了默认值，设置对象是否为单例

## bean 的生命周期

- 生命周期：从创建到消亡的完整过程
- bean 生命周期：bean 对象从创建到销毁的整体过程。
- bean 生命周期控制：在 bean 创建后到销毁前做一些事情

### 添加生命周期_配置方式

添加初始化和销毁方法
```java
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
    //表示bean初始化对应的操作
    public void init(){
        System.out.println("init...");
    }
    //表示bean销毁前对应的操作
    public void destory(){
        System.out.println("destory...");
    }
}
```

配置生命周期控制方法，对应属性名：
- init-method
- destroy-method
```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl" init-method="init" destroy-method="destory"/>
```

### 添加生命周期_接口方式

> 分析上面的实现过程，会发现添加初始化和销毁方法，即需要编码也需要配置，实现起来步骤比较多也比较乱。

Spring 提供了两个接口来完成生命周期的控制，好处是可以不用再进行配置 

添加两个接口 `InitializingBean`， `DisposableBean` 并实现接口中的两个方法 `afterPropertiesSet()` 和 `destroy()`

```java
public class BookServiceImpl implements BookService, InitializingBean, DisposableBean {
    private BookDao bookDao;
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }
    public void save() {
        System.out.println("book service save ...");
        bookDao.save(); 
    }
    public void destroy() throws Exception {
        System.out.println("service destroy");
    }
    public void afterPropertiesSet() throws Exception {
        System.out.println("service init");
    }
}
```

配置文件中无需配置
```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl"/>
```

### bean 销毁时机

通过上述两种方法添加控制生命周期的方法后，加载 Spring 的 IOC 容器，并从中获取对应的 bean 对象

```java
public class AppForLifeCycle {
    public static void main( String[] args ) {
        ApplicationContext ctx = new 
        	ClassPathXmlApplicationContext("applicationContext.xml");
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();
    }
}
```

运行结果
```
init...
book dao save...
```

init 方法执行了，但是 destroy 方法却未执行，这是为什么呢?
- Spring 的 IOC 容器是运行在 JVM 中
- 运行 main 方法后,JVM 启动,Spring 加载配置文件生成 IOC 容器,从容器获取 bean 对象，然后调方法执行
- main 方法执行完后，JVM 退出，这个时候 IOC 容器中的 bean 还没有来得及销毁就已经结束了
- 所以没有调用对应的 destroy 方法

---
bean的销毁时机：容器关闭前触发 bean 的销毁

关闭容器的方式
- *手工关闭容器*：
	- `ConfigurableApplicationcontext` 接口的 `close()`
- *注册关闭钩子*
	- `ConfigurableApplicationcontext` 接口的 `registerShutdownHook()` 操作

`close()` 和 `registerShutdownHook()` 选哪个?
- 关闭时机：
	- `close()` 是在调用的时候关闭容器，
	- `registerShutdownHook()` 是在 JVM 退出前调用关闭容器。
- `registerShutdownHook()` 调用后依旧可以操作容器

> `ApplicationContext` 接口中没有定义这两个方法，`ConfigurableApplicationContext` 接口继承了 `ApplicationContext`，新增了 `close()` 方法和 `registerShutdownHook()` 方法

1）手工关闭容器
```java
ClassPathXmlApplicationContext ctx = new 
    ClassPathXmlApplicationContext("applicationContext.xml");
//容器操作
ctx.close();
```

2）注册关闭钩子
```java
ClassPathXmlApplicationContext ctx = new 
    ClassPathXmlApplicationContext("applicationContext.xml");
ctx.registerShutdownHook();
//容器操作
```

## 核心容器

核心容器，可以简单地理解为 `ApplicationContext`
- 如何创建容器? 
- 创建好容器后，如何从容器中获取 bean 对象?
- 容器类的层次结构是什么?
- BeanFactory 是什么?

### 容器的创建

方式一：类路径加载配置文件
```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
```

方式二：文件路径加载配置文件
- 这种方式是从项目路径下开始查找`applicationContext.xml`配置文件的
```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("applicationContext.xml");
```

加载多个配置文件
```java
ApplicationContext ctx = new FileSystemXmlApplicationContext("bean1.xml", "bean2.xml"); 
```

### Bean 的获取

方式一：使用 bean 名称获取

```java
BookDao bookDao = (BookDao) ctx.getBean("bookDao");
```

方式二：使用 bean 名称获取，并指定类型
- 可以解决类型强转问题，但是参数又多加了一个，相对来说没有简化多少

```java
BookDao bookDao = ctx.getBean("bookDao"，BookDao.class);
```

方式三：使用 bean 类型获取
- 必须要确保 IOC 容器中该类型对应的 bean 对象只能有一个

```java
BookDao bookDao = ctx.getBean(BookDao.class);
```

### 容器类层次结构

![](assets/Pasted%20image%2020240208231338.png)

- BeanFactory 接口是 IoC 容器的顶层接口
- ApplicationContext 接口是 Spring 容器的核心接口
	- 提供基础的 bean 操作相关方法，通过其他接口扩展其功能
	- 常用初始化类
		- ClassPathXmlApplicationContext
		- FileSystemXmlApplicationContext（了解）

### BeanFactory

使用 BeanFactory 来创建 IOC 容器
```java
public class AppForBeanFactory {
    public static void main(String[] args) {
        Resource resources = new ClassPathResource("applicationContext.xml");
        BeanFactory bf = new XmlBeanFactory(resources);
        
        BookDao bookDao = bf.getBean(BookDao.class);
        bookDao.save();
    }
}
```

- BeanFactory 是延迟加载，只有在获取 bean 对象的时候才会去创建
- ApplicationContext 是立即加载，容器加载的时候就会创建 bean 对象

ApplicationContext 配置延迟加载
- `<bean>` 的 lazy-init 属性

```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl"  lazy-init="true"/>
```

## ----- DI 

向一个类中传递数据的方式有几种?
- 普通方法 (set 方法)
- 构造方法
        
依赖注入描述了在容器中建立 bean 与 bean 之间的依赖关系的过程，如果 bean 运行需要的是数字或字符串呢?
- 引用类型    
- 简单类型 (基本数据类型与 String)

Spring 为我们提供了两种注入方式，分别是:
- setter 注入
    - 简单类型
    - 引用类型（见[DI 入门案例](#DI%20入门案例)）
- 构造器注入
    - 简单类型
    - 引用类型

## setter 注入

- 对于引用数据类型使用的是 `<property name="" ref=""/>`
- 对于简单数据类型使用的是 `<property name="" value=""/>

### 引用数据类型注入

声明引用类型属性并提供可访问的 set() 方法
```java
public class BookServiceImpl implements BookService{
    private BookDao bookDao;
    private UserDao userDao;
    
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
        userDao.save();
    }
}
```

使用 property 标签的 ref 属性注入
```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl"/>
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/>

<bean id="bookService" class="com.itheima.service.impl.BookServiceImpl">
	<property name="bookDao" ref="bookDao"/>
	<property name="userDao" ref="userDao"/>
</bean>
```

### 简单数据类型注入

声明简单数据类型属性并提供可访问的 set() 方法
```java
public class BookDaoImpl implements BookDao {

    private String databaseName;
    private int connectionNum;

    public void setConnectionNum(int connectionNum) {
        this.connectionNum = connectionNum;
    }

    public void setDatabaseName(String databaseName) {
        this.databaseName = databaseName;
    }

    public void save() {
        System.out.println("book dao save ..."+databaseName+","+connectionNum);
    }
}
```

使用 property 标签的 value 属性注入
```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
	<property name="databaseName" value="mysql"/>
	<property name="connectionNum" value="10"/>
</bean>
```

> 对于参数类型，Spring 在注入的时候会自动转换，也不能 int 类型的 value 属性值为字符串，spring 在转换的时候会报错

## 构造器注入

### 引用数据类型

在 bean 中定义引用类型属性并提供可访问的构造方法

```java
public class BookServiceImpl implements BookService{
    private BookDao bookDao;
    private UserDao userDao;

    public BookServiceImpl(BookDao bookDao,UserDao userDao) {
        this.bookDao = bookDao;
        this.userDao = userDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
        userDao.save();
    }
}
```

使用 constructor-arg 标签 ref 属性注入引用类型对象
- ref 属性值是**构造方法的参数名**
- constructor-arg 标签的顺序任意
```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl"/>
<bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"/>

<bean id="bookService" class="com.itheima.service.impl.BookServiceImpl">
	<constructor-arg name="bookDao" ref="bookDao"/>
	<constructor-arg name="userDao" ref="userDao"/>
</bean>
```

### 简单数据类型

使用 constructor-arg 标签 的 value 属性，其他同上

```java
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
	<constructor-arg name="databaseName" value="mysql"/>
	<constructor-arg name="connectionNum" value="666"/>
</bean>
```

### 参数适配的其他两种方式（了解）

> 上面已经完成了构造函数注入的基本使用，但是会存在一些问题:
> - 当构造函数中方法的参数名发生变化后，配置文件中的name属性也需要跟着变
> - 这两块存在紧耦合，具体该如何解决?
>   
>  ![](assets/Pasted%20image%2020240208173122.png)
>   
>   在解决这个问题之前，需要提前说明的是，这个参数名发生变化的情况并不多，所以上面的还是比较主流的配置方式，下面介绍的，以了解为主。

方式一：type 属性按形参类型注入
- 如果构造方法参数中有类型相同的参数，这种方式就不太好实现了

```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
    <constructor-arg type="int" value="10"/>
    <constructor-arg type="java.lang.String" value="mysql"/>
</bean>
```

方式一：index 属性按形参位置注入
- 构造方法参数顺序发生变化后，这种方式又带来了耦合问题

```xml
<bean id="bookDao" class="com.itheima.dao.impl.BookDaoImpl">
    <constructor-arg index="1" value="100"/>
    <constructor-arg index="0" value="mysql"/>
</bean>
```

## 两种依赖注入方式选择

- “强制依赖”使用构造器进行，使用 setter 注入有概率不进行注入导致 null 对象出现
    - 强制依赖：对象在创建的过程中必须要注入指定的参数
- “可选依赖”使用 setter 注入进行，灵活性强
    - 可选依赖：对象在创建过程中注入的参数可有可无
- Spring 框架倡导使用构造器，**第三方框架内部**大多数采用构造器注入的形式进行数据初始化，相对严谨
- 如果有必要可以两者同时使用，使用构造器注入完成强制依赖的注入，使用 setter 注入完成可选依赖的注入
- 实际开发过程中还要根据实际情况分析，如果受控对象没有提供 setter 方法就必须使用构造器注入
- **自己开发的模块推荐使用 setter 注入**

## 自动配置

依赖自动装配：IoC 容器根据 bean 所依赖的资源在容器中自动查找并注入到 bean 中的过程称为自动装配

自动装配的方式
- **按类型（常用）**
- 按名称
- 按构造方法（几乎不用）

注意事项：
- 需要注入属性的类中对应**属性的 setter 方法不能省略**
- 自动装配用于**引用类型**依赖注入，不能对简单类型进行操作
- 自动装配**优先级低于**“setter 注入”与“构造器注入”，同时出现时自动装配配置失效

---
自动装配需要 set() 方法注入

```java
public class BookServiceImpl implements BookService{
    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

在 `<bean>` 标签中添加 autowire 属性，属性值：
- 按类型注入：byType
	- 必须保障容器中**相同类型的bean 唯一**，推荐使用
	- 按照类型在 Spring 的 IOC 容器中如果找到多个对象，会报 `NoUniqueBeanDefinitionException`
- 按名称注入：byName
	- set 后首字母小写是其属性名，通过这个属性名找要装配的 bean
	- 必须保障容器中具有指定名称的bean，因变量名与配置耦合，不推荐使用

byType
```xml
<bean class="com.itheima.dao.impl.BookDaoImpl"/>

<bean id="bookService" class="com.itheima.service.impl.BookServiceImpl" autowire="byType"/>
```

byName
```xml
<bean class="com.itheima.dao.impl.BookDaoImpl"/>

<bean id="bookService" class="com.itheima.service.impl.BookServiceImpl" autowire="byName"/>
```

![](assets/Pasted%20image%2020240208192039.png)

> 按照名称注入中的名称指的是什么?
> - bookDao 是 private 修饰的，外部类无法直接方法，外部类只能通过属性的 set 方法进行访问
> - 对外部类来说，setBookDao 方法名，去掉 set 后首字母小写是其属性名
>     - 为什么是去掉 set 首字母小写？这个规则是 set 方法生成的默认规则，set 方法的生成是把属性名首字母大写前面加 set 形成的方法名
> - 所以按照名称注入，其实是和对应的 set 方法有关，但是如果按照标准起名称，属性名和 set 对应的名是一致的
> - 如果按照名称去找对应的 bean 对象，找不到则注入 Null

%% -- %%
## 注入集合类型数据

- property 标签表示 setter 方式注入，构造方式注入 constructor-arg 标签内部也可以写 `<array>`、`<list>`、`<set>`、`<map>`、`<props>` 标签
- List 的底层也是通过数组实现的，所以 `<list>` 和 `<array>` 标签是可以混用
- 集合中要添加引用类型，只需要把 `<value>` 标签改成 `<ref>` 标签，这种方式用的比较少

注入数组类型数据

```xml
<property name="array">
    <array>
        <value>100</value>
        <value>200</value>
        <value>300</value>
    </array>
</property>
```

注入List类型数据

```xml
<property name="list">
    <list>
        <value>itcast</value>
        <value>itheima</value>
        <value>boxuegu</value>
        <value>chuanzhihui</value>
    </list>
</property>
```

注入Set类型数据

```xml
<property name="set">
    <set>
        <value>itcast</value>
        <value>itheima</value>
        <value>boxuegu</value>
        <value>boxuegu</value>
    </set>
</property>
```

注入Map类型数据

```xml
<property name="map">
    <map>
        <entry key="country" value="china"/>
        <entry key="province" value="henan"/>
        <entry key="city" value="kaifeng"/>
    </map>
</property>
```

注入Properties类型数据

```xml
<property name="properties">
    <props>
        <prop key="country">china</prop>
        <prop key="province">henan</prop>
        <prop key="city">kaifeng</prop>
    </props>
</property>
```

## 配置管理第三方 bean

### 案例：Druid 数据源对象管理

需求：使用 Spring 的 IOC 容器来管理 Druid 连接池对象

添加依赖
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```

配置第三方 bean：添加 `DruidDataSource` 的配置

> 查看 DruidDataSource 类中，只有 set()方法能注入

```xml
<!--管理DruidDataSource对象-->
<bean class="com.alibaba.druid.pool.DruidDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver"/>
	<property name="url" value="jdbc:mysql://localhost:3306/spring_db"/>
	<property name="username" value="root"/>
	<property name="password" value="root"/>
</bean>
```

从 IOC 容器中获取对应的 bean 对象
```java
public class App {
    public static void main(String[] args) {
       ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
       DataSource dataSource = (DataSource) ctx.getBean("dataSource");
       System.out.println(dataSource);
    }
}
```

### 加载 properties 文件

> 数据源中使用到了一些固定的常量如数据库连接四要素，把这些值写在 Spring 的配置文件中不利于后期维护
>     
> 需要将这些值提取到一个**外部的 properties 配置文件**中
>     
> Spring 框架如何**从配置文件中读取属性值来配置**就是接下来要解决的问题。

主要分为三个步骤
1. 开启 context 命名空间
2. 加载 properties 配置文件
3. 在 applicationContext.xml 中读取配置文件中的值并注入属性

---
resources 下创建一个 jdbc.properties 文件,并添加对应的属性键值对
```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://127.0.0.1:3306/spring_db
jdbc.username=root
jdbc.password=root
```

开启 `context` 命名空间（5处修改）
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

使用 `context` 命名空间下的标签来加载 properties 配置文件
```xml
<context:property-placeholder location="jdbc.properties"/>
```

![](assets/Pasted%20image%2020240208221412.png)

使用 `${key}` 来读取 properties 配置文件中的内容并完成属性注入

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">
    
    <context:property-placeholder location="jdbc.properties"/>
    
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

## bean 配置总结

bean 相关

![](assets/Pasted%20image%2020240208232407.png)

依赖注入相关

![](assets/Pasted%20image%2020240208232530.png)

# IOC/DI 注解开发

> Spring 的 IOC/DI 对应的配置开发就已经讲解完成，但是使用起来相对来说还是比较复杂的，复杂的地方在==配置文件==。

要想真正简化开发，就需要用到 Spring 的注解开发，Spring 对注解支持的版本历程:
- 2.0 版开始支持注解
- 2.5 版注解功能趋于完善  
- 3.0 版支持**纯注解开发**

注解开发定义 bean 用的是 2.5 版提供的注解，纯注解开发用的是 3.0 版提供的注解。

## 注解开发定义 bean

`@Component` 及其三个衍生注解

|名称|@Component/@Controller/@Service/@Repository|
|---|---|
|类型|类注解|
|位置|类定义上方|
|作用|设置该类为spring管理的bean|
|属性|value（默认）：定义bean的id|

---
生成 bean 的类上 添加 `@Component` 注解
- 注解会有默认空值 `String value() default "";`
- 注解如果不起名称，会有一个默认值是 **当前类名首字母小写**
	- 例：bookServiceImpl

```java
@Component("bookDao")
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ..." );
    }
}

@Component
public class BookServiceImpl implements BookService {
    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}
```

XML 与注解配置的对应关系:

![](assets/Pasted%20image%2020240209012411.png)

核心配置文件中通过组件扫描加载 bean
- base-package 指定 Spring 框架扫描的包路径，它会扫描**指定包及其子包**中的所有类上的注解。
- 包路径越多[如:com.itheima.dao.impl]，扫描的范围越小速度越快
- 包路径越少[如:com.itheima]，扫描的范围越大速度越慢
- 一般扫描到项目的组织名称即 Maven 的 groupId 下[如:com.itheima]即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>
</beans>
```

BookServiceImpl 类没有起名称，
- 按照**类型**来获取 bean 对象
- 按照默认的名称获取 bean 对象

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

		// 按 @Component 里的名称获取
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        bookDao.save();

		// 按类型获取
        BookService bookService = ctx.getBean(BookService.class);
        // 按默认名称获取获取
        BookService bookService2 = (BookService) ctx.getBean("bookServiceImpl");

        System.out.println(bookService == bookService2);
        bookService.save();
    }
}
```

## 纯注解开发定义 Bean

> 上面已经可以使用注解来配置bean,但是依然有用到配置文件，在配置文件中对包进行了扫描

Spring3.0 开启了纯注解开发模式，使用 **Java 类替代配置文件**，开启了 Spring 快速开发赛道

---
创建一个配置类，在配置类上添加 `@Configuration` 注解，将其标识为一个配置类，替换 `applicationContext.xml`

在配置类上添加包扫描注解 `@ComponentScan` 替换 `<context:component-scan base-package=""/>`

```java
@Configuration
@ComponentScan("com.itheima")
public class SpringConfig {}
```

读取 Spring 核心配置文件初始化容器对象切换为读取 **Java 配置类初始化容器对象**
- `AnnotationConfigApplicationContext(配置类.class)`

```java
public class AppForAnnotation {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        
        BookDao bookDao = (BookDao) ctx.getBean("bookDao");
        System.out.println(bookDao);
        BookService bookService = ctx.getBean(BookService.class);
        System.out.println(bookService);
    }
}
```

## bean 作用范围 与 生命周期管理

`@Scope` 设置 bean 的作用范围
- 类型：类注解
- 属性：
	- 默认值 singleton（单例）
	- 可选值 prototype（非单例）

`@PostConstruct` 和 `@PreDestroy` 指明 bean 的生命周期钩子
- 类型：类注解
- 作用：标识 初始化方法 和 销毁方法
- 属性：无

```java
@Repository
@Scope("prototype")
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ...");
    }
    
	@PostConstruct
    public void init() {
        System.out.println("init ...");
    }
    
    @PreDestroy
    public void destroy() {
        System.out.println("destroy ...");
    }
}
```

![](assets/Pasted%20image%2020240210211949.png)

## 依赖注入
### 自动装配（引用类型）

> Spring 为了使用注解简化开发，并没有提供 `构造函数注入`、`setter注入` 对应的注解，只提供了自动装配的注解实现。

`@Autowired` 注解
- 位置：
	- 属性上
	- setter 方法上
- 注入方式：
	- 按照**类型**注入优先
	- 容器里有多个类型相同的对象，则按照**属性名**注入
- 无需提供 setter 方法，基于反射设计创建对象并通过**暴力反射**为私有属性进行设值
- 注意：自动装配建议使用**无参构造**方法创建对象（默认），如果不提供对应构造方法，请提供唯一的构造方法

> 普通反射只能获取 public 修饰的内容，暴力反射除了获取 public 修饰的内容还可以获取 private 修改的内容

`@Qualifier` 必须配合 `@Autowired` 注解使用，
- value 属性：指定注入哪个名称的 bean 对象

```java
@Service
public class BookServiceImpl implements BookService {
    @Autowired // 按属性名注入
    private BookDao bookDao; // 注入的是bookDao
    
	@Autowired  
	@Qualifier("bookDao2") // 按名称注入
	private BookDao bookDao3; // 注入的是bookDao2
    
    public void save() {
        System.out.println("book service save ...");
        bookDao.save();
    }
}

@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    public void save() {
        System.out.println("book dao save ..." );
    }
}

@Repository("bookDao2")
public class BookDaoImpl2 implements BookDao {
    public void save() {
        System.out.println("book dao save ...2" );
    }
}
```

### 简单数据类型注入

`@Value` 注解，将值写入注解的参数中就行了

```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    @Value("itheima")
    private String name;
    public void save() {
        System.out.println("book dao save ..." + name);
    }
}
```

### 注解读取 properties 配置文件

`@PropertySource`
- 类注解
- 路径仅支持单一文件配置，多文件请使用数组格式配置，
- 不允许使用通配符 `*`

---
resource 下准备 properties 文件

```properties
name=itheima888
```

在配置类上添加 `@PropertySource` 注解

```java
@Repository("bookDao")
public class BookDaoImpl implements BookDao {
    @Value("${name}")
    private String name;
    public void save() {
        System.out.println("book dao save ..." + name);
    }
}
```

## 管理第三方 bean

依赖准备

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.16</version>
</dependency>
```

### 注册第三方 Bean

> 前面定义 bean 的时候都是在自己开发的类上面写个注解就完成了，但如果是第三方的类，这些类都是在 jar 包中，我们没有办法在类上面添加注解，这个时候该怎么办?
> 
> 遇到上述问题，我们就需要有一种更加灵活的方式来定义 bean，这种方式不能在原始代码上面书写注解，一样能定义 bean，这就用到了一个全新的注解 `@Bean`。

`@Bean` 注解
- 位置：方法上
- 作用：将方法的返回值制作为 Spring 管理的一个 bean 对象

```java
@Configuration
public class SpringConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

> 不能使用 `DataSource ds = new DruidDataSource()`，因为 DataSource 接口中没有对应的 setter 方法来设置属性。

### 将独立配置类加入核心配置

> 如果把所有的第三方 bean 都配置到 Spring 的配置类 `SpringConfig` 中，虽然可以，但是不利于代码阅读和分类管理，所有我们就想能不能按照类别将这些 bean 配置到不同的配置类中?

新建一个 `JdbcConfig` 配置类，并把数据源配置到该类下。

```java
public class JdbcConfig {
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/spring_db");
        ds.setUsername("root");
        ds.setPassword("root");
        return ds;
    }
}
```

---
方式一：导入式（推荐），`@Import` 
- 手动加入配置类到核心配置
- 此注解只能添加一次，多个数据请用数组格式

```Java
@Configuration
@Import({JdbcConfig.class})
public class SpringConfig {}
```

方式二：扫描式，`@Configuration` + `@ComponentScan`
- 在外部配置类上加 `@Configuration` 注解
- 在在 Spring 的配置类上添加包扫描 `@ComponentScan`

```java
@Configuration
public class JdbcConfig {}
```

```java
@Configuration
@ComponentScan("com.itheima.config")
public class SpringConfig {}
```

### 为第三方 bean 注入资源

简单数据类型注入使用 `@Value` 注解

```java
@Configuration
@PropertySource("classpath:jdbc.properties")
public class JdbcConfig {
    @Value("${driver}")
    private String driver;
    @Value("${url}")
    private String url;
    @Value("${userName}")
    private String userName;
    @Value("${password}")
    private String password;
    
	@Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        System.out.println(driver);
        return ds;
    }
}
```

引用类型注入只需要为 bean 定义方法**设置形参即可**，容器会根据类型**自动装配**对象。

```java
@Bean
public DataSource dataSource(BookDao bookDao){
    System.out.println(bookDao);
    
    DruidDataSource ds = new DruidDataSource();
    ds.setDriverClassName(driver);
    ds.setUrl(url);
    ds.setUsername(userName);
    ds.setPassword(password);
    return ds;
}
```

## 总结

![](assets/Pasted%20image%2020240211183236.png)

## Spring 整合

### Mybatis

依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-context</artifactId>
		<version>5.2.10.RELEASE</version>
	</dependency>
	<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>druid</artifactId>
		<version>1.1.16</version>
	</dependency>
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.47</version>
	</dependency>
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.5.6</version>
	</dependency>
	<!--Spring操作数据库需要该jar包-->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-jdbc</artifactId>
		<version>5.2.10.RELEASE</version>
	</dependency>
	<!--
		Spring与Mybatis整合的jar包
		这个jar包mybatis在前面，是Mybatis提供的
	-->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis-spring</artifactId>
		<version>1.3.0</version>
	</dependency>
</dependencies>
```

数据库表建立

```sql
create database spring_db character set utf8;
use spring_db;
create table tbl_account(
    id int primary key auto_increment,
    name varchar(35),
    money double
);
```

对应的模型类

```java
public class Account implements Serializable {

    private Integer id;
    private String name;
    private Double money;
	//setter...getter...toString...方法略    
}
```

Dao 接口，Service接口及实现类
```java
public interface AccountDao {

    @Insert("insert into tbl_account(name,money)values(#{name},#{money})")
    void save(Account account);

    @Delete("delete from tbl_account where id = #{id} ")
    void delete(Integer id);

    @Update("update tbl_account set name = #{name} , money = #{money} where id = #{id} ")
    void update(Account account);

    @Select("select * from tbl_account")
    List<Account> findAll();

    @Select("select * from tbl_account where id = #{id} ")
    Account findById(Integer id);
}

public interface AccountService {

    void save(Account account);

    void delete(Integer id);

    void update(Account account);

    List<Account> findAll();

    Account findById(Integer id);

}

@Service
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao;

    public void save(Account account) {
        accountDao.save(account);
    }

    public void update(Account account){
        accountDao.update(account);
    }

    public void delete(Integer id) {
        accountDao.delete(id);
    }

    public Account findById(Integer id) {
        return accountDao.findById(id);
    }

    public List<Account> findAll() {
        return accountDao.findAll();
    }
}
```

> 准备工作结束

---

配置类

```java
// 数据源的配置类
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.username}")
    private String userName;
    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        ds.setDriverClassName(driver);
        ds.setUrl(url);
        ds.setUsername(userName);
        ds.setPassword(password);
        return ds;
    }
}

// Mybatis配置类
public class MybatisConfig {
    //定义bean，SqlSessionFactoryBean，用于产生SqlSessionFactory对象
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        //设置模型类的别名扫描
        ssfb.setTypeAliasesPackage("com.itheima.domain");
        //设置数据源
        ssfb.setDataSource(dataSource);
        return ssfb;
    }
    //定义bean，返回MapperScannerConfigurer对象
    @Bean
    public MapperScannerConfigurer mapperScannerConfigurer(){
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        msc.setBasePackage("com.itheima.dao");
        return msc;
    }
}

// 主配置类
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class})
public class SpringConfig {
}
```

说明：
- SqlSessionFactoryBean 是前面我们讲解 FactoryBean 的一个子类，在该类中**将 SqlSessionFactory 的创建进行了封装**，简化对象的创建，我们只需要将其需要的内容设置即可。
    - 方法中有一个参数为 dataSource,当前 Spring 容器中已经创建了 Druid 数据源，类型刚好是 DataSource 类型，此时在初始化 SqlSessionFactoryBean 这个对象的时候，发现需要使用 DataSource 对象，而容器中刚好有这么一个对象，就自动加载了 DruidDataSource 对象。

![](assets/Pasted%20image%2020240211232726.png)

- 使用 MapperScannerConfigurer 加载 Dao 接口，创建代理对象保存到 IOC 容器中

![](assets/Pasted%20image%2020240211232828.png)

从 IOC 容器中获取 Service 对象，调用方法获取结果

```java
public class App2 {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);

        AccountService accountService = ctx.getBean(AccountService.class);

        Account ac = accountService.findById(1);
        System.out.println(ac);
    }
}
```

支持 Spring 与 Mybatis 的整合就已经完成了，其中主要用到的两个类分别是:
- SqlSessionFactoryBean
- MapperScannerConfigurer

### Junit

依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.2.10.RELEASE</version>
</dependency>
```
 
`@RunWith` 设置JUnit运行器
- 测试类注解
- 属性：`value`，运行所使用的运行器

> Junit 运行后是基于 Spring 环境运行的，所以 Spring 提供了一个专用的类运行器，这个务必要设置，这个类运行器就在 Spring 的测试专用包中提供的，导入的坐标就是这个东西 `SpringJUnit4ClassRunner`

`@ContextConfiguration` 设置 JUnit加载的 Spring 核心配置
- 测试类注解
- 属性
	- classes：核心配置类，可以使用数组格式设定加载多个配置类
	- locations：配置文件，可以使用数组的格式设定加载多个配置文件名称

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {SpringConfig.class}) // 加载配置类
public class AccountServiceTest {
    // 支持自动装配注入bean
    @Autowired
    private AccountService accountService;

    @Test
    public void testFindById() {
        System.out.println(accountService.findById(1));

    }
}
```

# ---------- AOP

## AOP 简介

*AOP (Aspect Oriented Programming)* 面向切面编程，一种**编程范式**，指导开发者如何组织程序结构。
- OOP(Object Oriented Programming)面向对象编程

作用：在不惊动原始设计的基础上为其进行功能增强，前面咱们有技术就可以实现这样的功能即代理模式。

核心概念：
- *连接点* (JoinPoint)：程序执行过程中的任意位置，粒度为执行方法、抛出异常、设置变量等
    - 在 SpringAOP 中，理解为方法的执行
- *切入点* (Pointcut)：匹配连接点的式子
    - 在 SpringAOP 中，一个切入点可以描述一个具体方法，也可也匹配多个方法
        - 一个具体的方法：如 com.itheima.dao 包下的 BookDao 接口中的无形参无返回值的 save 方法
        - 匹配多个方法：所有的 save 方法，所有的 get 开头的方法，所有以 Dao 结尾的接口中的任意方法，所有带有一个参数的方法
    - 连接点范围要比切入点范围大，是切入点的方法也一定是连接点，但是是连接点的方法就不一定要被增强，所以可能不是切入点。
- *通知* (Advice)：在切入点处执行的操作，也就是共性功能
    - 在 SpringAOP 中，功能最终以方法的形式呈现
- *通知类*：定义通知的类
- *切面* (Aspect)：描述通知与切入点的对应关系。

## 入门案例

> 注解版

需求：在方法执行前输出当前系统时间

导入坐标
- 因为 `spring-context` 中已经导入了 `spring-aop`，所以不需要再单独导入 `spring-aop`

> AspectJ 是 AOP 思想的一个具体实现，Spring 有自己的 AOP 实现，但是相比于 AspectJ 来说比较麻烦，所以我们直接采用 Spring 整合 ApsectJ 的方式进行 AOP 开发。

```java
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

需要增强的类

```java
public interface BookDao {
    public void save();
    public void update();
}

@Repository
public class BookDaoImpl implements BookDao {

    public void save() {
        System.out.println(System.currentTimeMillis());
        System.out.println("book dao save ...");
    }

    public void update(){
        System.out.println("book dao update ...");
    }
}
```

通知类
- 定义通知 method
- 定义切入点 `@Pointcut`
	- 切入点定义依托一个不具有实际意义的方法进行，即无参数、无返回值、方法体无实际逻辑。
- **绑定**切入点与通知关系，并指定通知添加到原始连接点的具体执行**位置** `@Before`
- 定义通知类受Spring容器管理 `@Component`
- 定义当前类为切面类 `@Aspect`

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(void com.itheima.dao.BookDao.update())")
    private void pt(){}
    
    @Before("pt()")
    public void method(){
        System.out.println(System.currentTimeMillis());
    }
}
```

开启 Spring 对 AOP 注解驱动支持 `@EnableAspectJAutoProxy`

```java
@Configuration
@ComponentScan("org.example")
@Import({JdbcConfig.class, MybatisConfig.class})
@EnableAspectJAutoProxy
public class SpringConfig {}
```

获取 bean 执行方法，看增强效果

```java
public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = ctx.getBean(BookDao.class);
        bookDao.update();
    }
}
```



# IOC


## IOC 容器

IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂

在Spring中，IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map中存放的是各种对象（以后可以看看IOC源码）



## IOC 操作 Bean 管理——基于 xml 方式


### 注入属性

DI：依赖注入，就是注入属性

#### 第一种：set方法 + 无参（重要）

```java
public class Book {
    private String bname;
    private String bauthor;

    //创建属性对应的 set 方法
    public void setBname(String bname) {
        this.bname = bname;
    }
    public void setBauthor(String bauthor) {
        this.bauthor = bauthor;
    }

    @Override
    public String toString() {
        return "Book{" +
                "bname='" + bname + '\'' +
                ", bauthor='" + bauthor + '\'' +
                '}';
    }
}
```

```xml
<bean id="book" class="com.boer.spring5.Book">
    <!--使用 property 完成属性注入
    name：类里面属性名称  value：向属性注入的值-->
    <property name="bname" value="易筋经"></property>
    <property name="bauthor" value="达摩老祖"></property>
</bean>
```

```java
@Test
public void testBook() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    Book book = context.getBean("book", Book.class);
    System.out.println(book);
}
```



#### 第二种：有参构造方法注入

```java
public class Orders {
    private String oname;
    private String address;

    //有参数构造
    public Orders(String oname, String address) {
        this.oname = oname;
        this.address = address;
    }

    @Override
    public String toString() {
        return "Orders{" +
                "oname='" + oname + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```

```xml
<bean id="orders" class="com.boer.spring5.Orders">
    <constructor-arg name="oname" value="电脑"></constructor-arg>
    <constructor-arg name="address" value="China"></constructor-arg>
</bean>
```

```java
@Test
public void testOrders() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    Orders orders = context.getBean("orders", Orders.class);
    System.out.println(orders);
}
```



### 工厂Bean

Spring有两种类型bean，一种普通bean，另外一种工厂bean（FactoryBean） 

- 普通 bean：在配置文件中定义 bean 类型就是返回类型
- 工厂 bean：在配置文件定义 bean 类型可以和返回类型不一样

**实现工厂bean：**

1. 创建类，让这个类作为工厂bean，实现接口FactoryBean
2. 实现接口里面的方法getObject()，在实现的方法中定义返回的bean类型

```java
public class MyBean implements FactoryBean<Orders> { //实现factory接口，泛型类型为Orders
    @Override
    public Orders getObject() throws Exception {
        Orders orders=new Orders("boer","aaa");
        return orders;
    }

//    @Override
//    public Class<?> getObjectType() {
//        return null;
//    }
//
//    @Override
//    public boolean isSingleton() {
//        return FactoryBean.super.isSingleton();
//    }
}
```

```xml
<bean id="myBean" class="com.boer.spring5.MyBean"></bean>
```

```java
@Test
public void testOrders() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    //返回类型为Orders，xml配置的类为myBean
    Orders orders = context.getBean("myBean", Orders.class);
    System.out.println(orders);
}
```



## Bean作用域

在 Spring 里面，设置创建 bean 实例是单实例还是多实例

### 默认作用域

在 Spring 里面，默认情况下，bean 是单实例对象

```java
@Test
public void testOrders() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    Orders orders = context.getBean("orders", Orders.class);
    Orders orders2 = context.getBean("orders", Orders.class);
    System.out.println(orders); //com.boer.spring5.Orders@7a30d1e6
    System.out.println(orders2); //com.boer.spring5.Orders@7a30d1e6
}
```

### 如何设置单实例还是多实例

在 spring 配置文件 bean 标签里面有属性（scope）用于设置单实例还是多实例

scope 属性值

- singleton：==单例模式==，默认值，表示是单实例对象
- prototype：==原型模式==，每次从容器中get对象时，都重新创建
- request、session、application、websocket这些只能在web开发中使用，不常用，见八股文

```java
@Test
public void testOrders() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    Orders orders = context.getBean("orders", Orders.class);
    Orders orders2 = context.getBean("orders", Orders.class);
    System.out.println(orders); //com.boer.spring5.Orders@7a30d1e6
    System.out.println(orders2); //com.boer.spring5.Orders@5891e32e
}
```

```xml
<bean id="orders" class="com.boer.spring5.Orders" scope="prototype"></bean>
```

### singleton 和 prototype 区别

singleton：加载spring配置文件时候就会创建单实例对象

> 加载spring配置文件：
>
> ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");

prototype：不是在加载spring配置文件时候创建对象，而是在调用getBean方法时候创建多实例对象



## Bean的生命周期

生命周期：从对象创建到对象销毁的过程

**bean生命周期**

1. 通过构造器创建 bean 实例（无参数构造）
2. 为 bean 的属性设置值和对其他 bean 引用（调用 set 方法）
3. 把 bean 实例传递 bean 后置处理器的方法==postProcessBeforeInitialization()==
4. 调用 bean 的初始化的方法（需要进行配置初始化的方法）
5. 把 bean 实例传递 bean 后置处理器的方法==postProcessAfterInitialization()==
6. bean 可以使用了（对象获取到了）
7. 当容器关闭时候，调用 bean 的销毁的方法（需要进行配置销毁的方法）

```java
public class Orders {
    private String oname;
    private String address;

    public Orders() {
        System.out.println("第一步：无参");
    }

    public void setOname(String oname) {
        this.oname = oname;
        System.out.println("第二步：注入属性");
    }

    public void initMethod(){
        System.out.println("第三步：执行初始化方法");
    }

    public void destoryMethod(){
        System.out.println("第五步：执行销毁方法");
    }
}
```

```java
/**
 * 配置后置处理器
 */
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}
```

```xml
<bean id="orders2" class="com.boer.spring5.Orders" init-method="initMethod" destroy-method="destoryMethod">
    <property name="oname" value="手机"></property>
</bean>

<!--配置后置处理器-->
<bean id="myBeanPost" class="com.boer.spring5.MyBeanPost"></bean>
```

```java
@Test
public void testOrders2() {
    //ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
  	//ApplicationContext里没有close()
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
  	//System.out.println("......"); //加载配置文件以后，对象已经创建且初始化
    Orders orders = context.getBean("orders2", Orders.class);
    System.out.println("第四步：获取实例对象");
    System.out.println(orders);
    context.close();
    //第一步：无参
    //第二步：注入属性
    //在初始化之前执行的方法
    //第三步：执行初始化方法
    //在初始化之后执行的方法
    //第四步：获取实例对象
    //com.boer.spring5.Orders@4a87761d
    //第五步：执行销毁方法
}
```



## Bean的自动装配

自动装配是 spring 满足 bean 依赖的一种方式，Spring 会在上下文中自动寻找，并自动给 bean 装配属性

在Spring中由三种装配方式

1. 在xml中显式配置
2. 在java中显式配置
3. 隐式的自动装配bean（重要）

### byName与byType自动装配

```xml
<!--
	autowire的属性值：
  	byName:会在容器上下文中查找，和自己对象set方法后面的值相对应的beanid，比如：setCat()找的beanid就是cat
  	byType：会自动在容器上下文中查找，和自己对象属性类型相同的bean
-->
<bean id="people" class="com.boer.spring5.pojo.People" autowire="byName">
    <property name="name" value="boer"></property>
</bean>

<bean id="cat" class="com.boer.spring5.pojo.Cat"></bean>
<bean id="dog" class="com.boer.spring5.pojo.Dog"></bean>
```

```java
public class People {
    private Cat cat;
    private Dog dog;
    private String name;
    //get、set
}
public class Dog {
    public void shout(){
        System.out.println("wang~");
    }
}
public class Cat {} //同dog
```

```java
@Test
public void test1() {
    ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
    People people = context.getBean("people", People.class);
    people.getCat().shout(); //miao~
    people.getDog().shout(); //wang~
}
```

**小结：**

byName：需要保证所有bean的id唯一，并且这个bean需要和自动注入的属性的set方法的值一致

byType：需要保证所有bean的class唯一，并且这个bean需要和自动注入的属性的类型一致



### 注解实现自动装配

配置xml，导入context约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xmlns:aop="http://www.springframework.org/schema/aop"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

   <!--开启注解支持-->
   <context:annotation-config/>
</beans>
```

**@Autowired**（常用）

直接在属性上使用即可！也可以在set方法上使用

默认ByType匹配（多个同类型好像好像也能匹配到）,要求依赖对象对象存在（可通过required更改）

出现多个同类型的Bean，可以指定唯一Bean对象，通过==@Qualifier==，设置value值为bean的id

**@Resource**

JDK原生注解（没有Spring也能用），默认ByName匹配，找不到ByType

```java
public class People {
    @Autowired(required = false) //说明这个对象可以为null，否则不允许为空
    @Qualifier(value = "cat1") //点击导航图标会跳转到 beanid=dog1
    private Cat cat;

    @Resource
    private Dog dog;

    private String name;
		//get、set
}
```

```java
<bean id="people" class="com.boer.spring5.pojo.People"></bean>

<bean id="cat" class="com.boer.spring5.pojo.Cat"></bean>
<bean id="cat1" class="com.boer.spring5.pojo.Cat"></bean>

<bean id="dog" class="com.boer.spring5.pojo.Dog"></bean>
<bean id="dog1" class="com.boer.spring5.pojo.Dog"></bean>
```



## 注解开发

在Spring4之后，要使用注解开发，必须保证aop的包导入了

使用注解需要导入context约束，增加注解的支持

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       https://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context
       https://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/aop
       https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启注解支持-->
    <context:annotation-config/>

    <!--这个标签已经暗含上面的注解支持标签了，可以点进component-scan查看注释-->
    <context:component-scan base-package="com.boer.spring5.pojo"></context:component-scan>
</beans>
```



### 装配bean==@Component==

@Component有几个衍生注解，在web开发中，会按照mvc三层架构分层

- dao层 ==@Repository==
- service层 ==@Service==
- controller层 ==@Controller==

这四个注解功能都是一样的，都是代表将某个类注册到Spring中，装配Bean

```java
@Component //等价于<bean id="user" class="com.yang.entity.User"/>
public class User {}
```



### 属性注入==@Value==

对于复杂的属性值还是要用xml装配Bean

```java
@Component
public class User {
    @Value("Boer") //等价于<property name="name" value="yang"/>
    private String name;
}
```



### 自动装配==@Autowired==

见Bean的自动装配



### 作用域==@Scope==

```java
@Component 
@Scope("prototype") //Bean作用域：原型模式
public class User {}
```



### XML与注解

xml更加万能，适用于任何场合，维护简单方便

注解不是自己类使用不了，维护相对复杂

XML与注解最佳实践：xml用来管理bean，注解只负责完成属性的注入



## 完全注解开发（抛弃xml）

> 这种纯java的配置方式在Spring Boot中随处可见

编写一个配置类

```java
//@Configuration该注解代表了这是一个配置类，与applicationContext.xml一样
@Configuration //这个也会被Spring容器托管，注册到容器中，打开注解里面有@Component
@ComponentScan("com.boer.spring5.pojo") //开启包扫描
@Import(BoerConfig2.class) //引入其他的配置类
public class BoerConfig {
    //注册一个Bean，就相当于我们之前写的一个bean标签
    //方法名字：bean标签的id
    //方法的返回值：bean标签中的class属性
    @Bean
    public User getUser () {
        return new User(); //就是返回要注入到bean的对象
    }
}
```

注意：@Bean和@ComponentScan 会生成不一样的对象

测试：

```java
//测试
@Component //@Component 等价于<bean id="user" class="com.yang.entity.User"/>
public class User {
    @Value("Boer") //@Value("Boer") 等价于<property name="name" value="yang"/>
    private String name;
   //get、set
}
```

```java
@Test
public void test3() {
    //AnnotationConfigApplicationContext
    ApplicationContext context1 = new AnnotationConfigApplicationContext(BoerConfig.class);
    User user1 = context1.getBean("getUser", User.class); //getUser就是Beanid
    User user = context1.getBean("user", User.class);

    System.out.println(user.getName()); //boer
    System.out.println(user1.getName()); //boer
    //不是同一个对象
    System.out.println(user); //com.boer.spring5.pojo.User@470f1802
    System.out.println(user1); //com.boer.spring5.pojo.User@63021689
}
```



# AOP

## 问题引入

现有三个类，`Horse`、`Pig`、`Dog`，这三个类中都有 eat 和 run 两个方法

通过 OOP 思想中的继承，我们可以提取出一个 Animal 的父类，然后将 eat 和 run 方法放入父类中，`Horse`、`Pig`、`Dog`通过继承`Animal`类即可自动获得 `eat()` 和 `run()` 方法。这样将会少写很多重复的代码。

OOP 编程思想可以解决大部分的代码重复问题。但是有一些问题是处理不了的。

比如在父类 Animal 中的多个方法的相同位置出现了重复的代码，OOP 就解决不了。

```java
/**
 * 动物父类
 */
public class Animal {
    /** 身高 */
    private String height;
    /** 体重 */
    private double weight;

    public void eat() {
        // 性能监控代码
        long start = System.currentTimeMillis();
        // 业务逻辑代码
        System.out.println("I can eat...");
        // 性能监控代码
        System.out.println("执行时长：" + (System.currentTimeMillis() - start)/1000f + "s");
    }

    public void run() {
        // 性能监控代码
        long start = System.currentTimeMillis();
        // 业务逻辑代码
        System.out.println("I can run...");
        // 性能监控代码
        System.out.println("执行时长：" + (System.currentTimeMillis() - start)/1000f + "s");
    }
}
```

这部分重复的代码，一般统称为==横切逻辑代码==

横切逻辑代码存在的问题：

- 代码重复问题
- 横切逻辑代码和业务代码混杂在一起，代码臃肿，不变维护

AOP就是用来解决这些问题的，AOP另辟蹊径，提出横向抽取机制，将横切逻辑代码和业务逻辑代码分离

代码拆分比较容易，难的是如何在不改变原有业务逻辑的情况下，悄无声息的将横向逻辑代码应用到原有的业务逻辑中，达到和原来一样的效果

<img src="图片/640.png" alt="640" style="zoom: 50%;" />



## AOP相关概念 

### 什么是AOP？

AOP：Aspect oriented programming 面向切面编程，AOP 是 OOP（面向对象编程）的一种延续

### AOP解决了什么问题？

在不改变原有业务逻辑的情况下，增强横切逻辑代码，根本上解耦合，避免横切逻辑代码重复

不通过修改源代码方式，在主干功能里面添加新功能

### AOP 为什么叫面向切面编程？

**切** ：指的是横切逻辑，原有业务逻辑代码不动，只能操作横切逻辑代码，所以面向横切逻辑

**面** ：横切逻辑代码往往要影响的是很多个方法，每个方法如同一个点，多个点构成一个面。这里有一个面的概念



## Spring AOP底层原理

AOP 底层使用动态代理，有两种情况动态代理

1. 有接口情况，使用`JDK动态代理`，创建==接口实现类代理对象==，增强类的方法
2. 没有接口情况，使用`CGLIB动态代理`，创建==子类的代理对象==，增强类的方法



### JDK 动态代理

使用 `Proxy` 类（java.lang.reflect.Proxy）里面的方法创建代理对象

调用 `newProxyInstance()`，方法的三个参数：

1. 类加载器
2. 增强方法所在的类实现的接口，支持多个接口
3. 实现接口`InvocationHandler`，写增强的部分

```java
//创建接口
public interface UserDao {
    public int add(int a, int b);
    public String update(String id);
}
```

```java
//创建接口实现类
public class UserDaoImpl implements UserDao {
    @Override
    public int add(int a, int b) {
        return a + b;
    }

    @Override
    public String update(String id) {
        return id;
    }
}
```

```java
/**
 * 创建代理对象
 */
class UserDaoProxy implements InvocationHandler {
    //把被代理对象传过来
    //有参数构造传递
    private Object obj;
    public UserDaoProxy(Object obj) {
        this.obj = obj;
    }

    //增强的逻辑，加入横切的代码
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //方法之前
        System.out.println("方法之前执行...." + method.getName() + " :传递的参数..." + Arrays.toString(args));
        //被增强的方法执行
        Object res = method.invoke(obj, args);
        //方法之后
        System.out.println("方法之后执行...." + obj);
        return res;
    }
}
```

```java
/**
 * 使用Proxy类
 */
public class JDKProxy {
    public static void main(String[] args) {
        Class[] interfaces = {UserDao.class};
        UserDao userDao = new UserDaoImpl();
        //使用 Proxy 类创建接口代理对象
        UserDao dao = (UserDao) Proxy.newProxyInstance(
                JDKProxy.class.getClassLoader(),
                interfaces,
                new UserDaoProxy(userDao));

        System.out.println(dao.add(1, 2));
        System.out.println(dao.update("id"));
    }
}
```



## AspectJ 

> Springaop和aspectj的区别：
>
> [(9条消息) springAOP 和 aspectJ 有什么区别_awesome_go的博客-CSDN博客_aspectj和springaop区别](https://blog.csdn.net/u013452337/article/details/100981702)

### 相关概念

#### 介绍

Spring 框架一般都是基于 AspectJ 实现 AOP 操作

AspectJ 不是 Spring 组成部分，是独立 AOP 框架，一般把 AspectJ 和 Spring 框架一起使用，进行 AOP 操作

基于 AspectJ 实现 AOP 操作

1. 基于 xml 配置文件实现
2. 基于注解方式实现**（使用）**



#### 术语

`连接点`：类里面哪些方法可以被增强，这些方法称为连接点

`切入点`：实际被真正增强的方法，称为切入点

`通知`：实际增强的逻辑部分

- 前置通知
- 后置通知
- 环绕通知
- 异常通知
- 最终通知



#### **切入点表达式**

作用：知道对哪个类里面的哪个方法进行增强

语法结构： execution(权限修饰符 返回类型 类全路径 方法名称 参数列表)

> 举例1：对 com.atguigu.dao.BookDao 类里面的 add 进行增强
>
> ```java
> execution(* com.atguigu.dao.BookDao.add(..))
> ```
>
> 举例2：对 com.atguigu.dao.BookDao 类里面的所有的方法进行增强
>
> ```
> execution(* com.atguigu.dao.BookDao.*(..))
> ```
>
> 举例3：对 com.atguigu.dao 包里面所有类，类里面所有方法进行增强
>
> ```
> execution(* com.atguigu.dao.*.*(..))
> ```



### AOP操作

#### 依赖准备

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.7.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjtools</artifactId>
    <version>1.9.5</version>
</dependency>

<dependency>
    <groupId>aopalliance</groupId>
    <artifactId>aopalliance</artifactId>
    <version>1.0</version>
</dependency>

<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.0</version>
</dependency>

<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>
```

#### xml文件配置

配置类的方式见下一节

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd
 http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd
 http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!-- 开启注解扫描 -->
    <context:component-scan base-package="com.boer.spring5.aopanno"></context:component-scan>
    <!-- Enables the use of the @AspectJ style of Spring AOP.-->
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

#### 创建被增强类

```java
@Component
public class User {
    public void add() {
        System.out.println("add.......");
    }
}
```

#### 创建增强的类，加上@Aspect

#### 对相同切入点抽取

#### 配置不同类型的通知

```java
@Component
@Aspect //生成代理对象
public class UserProxy {
    //相同切入点抽取
    @Pointcut(value = "execution(* com.boer.spring5.aopanno.User.add(..))")
    public void pointDemo() {
    }

    //前置通知
    //@Before(value = "execution(* com.boer.spring5.aopanno.User.add(..))")
    @Before(value = "pointDemo()")
    public void before() {
        System.out.println("before......");
    }

    //后置通知（返回通知），返回结果后执行，有异常不执行
    @AfterReturning(value = "pointDemo()")
    public void afterReturning() {
        System.out.println("afterReturning.........");
    }

    //最终通知，有异常也执行
    @After(value = "pointDemo()")
    public void after() {
        System.out.println("after.........");
    }

    //异常通知
    @AfterThrowing(value = "pointDemo()")
    public void afterThrowing() {
        System.out.println("afterThrowing.........");
    }

    //环绕通知
    @Around(value = "pointDemo()")
    public void around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕之前.........");
        //被增强的方法执行
        proceedingJoinPoint.proceed();
        System.out.println("环绕之后.........");
    }
}
```

#### 设置增强类优先级

有多个增强类多同一个方法进行增强，设置增强类优先级

在增强类上面添加注解 `@Order(数字类型值)`，数字类型值越小优先级越高

```java
@Component
@Aspect
@Order(1) //设置优先级
public class UserProxy2 {
    @Before(value = "execution(* com.boer.spring5.aopanno.User.add(..))")
    public void before() {
        System.out.println("UserProxy2：before......");
    }
}
```

#### 测试

```java
@Test
public void test4() {
    ApplicationContext context1 = new ClassPathXmlApplicationContext("bean1.xml");
    User user = context1.getBean("user", User.class);
    user.add();
}
```



### AspectJ 配置文件

1. xml配置文件（不了解了）
2. 配置类（完全注解开发）

```java
@Configuration
@ComponentScan(basePackages = {"com.boer.spring5.aopanno"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {}
```

```java
@Test
public void test5() {
    ApplicationContext context = new AnnotationConfigApplicationContext(ConfigAop.class);
    User user = context.getBean("user", User.class);
    user.add();
}
```



# 事务

