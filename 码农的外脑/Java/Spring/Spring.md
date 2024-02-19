
# 参考资料

视频
- 尚硅谷Spring（lj）
- 黑马2022SSM（nb）

推荐文章
- IOC 源码：[Spring IOC 容器源码分析_Javadoop](https://javadoop.com/post/spring-ioc)
- springaop 和 aspectj 的区别 [springAOP 和 aspectJ 有什么区别_spring aop and aspectj aop 有什么区别?-CSDN博客](https://blog.csdn.net/u013452337/article/details/100981702)

# -------------------- Spring

# ---------- Spring 介绍

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

# ---------- 入门案例

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

# ---------- IOC and DI 相关内容

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

# ---------- IOC/DI 注解开发

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

## AOP 工作流程

> SpringAOP 的本质或者可以说底层实现是通过代理模式。

- Spring 容器启动
	- 需要被增强的类、通知类会被加载
	- 此时bean对象还没有创建成功
- 读取所有切面配置中的切入点
	- 没有使用的切入点不会被读取
- 初始化 bean，判定 bean 对应的类中的方法是否匹配到任意切入点
	- 匹配失败，创建原始对象
	- 匹配成功，创建原始对象（目标对象）的代理对象
- 获取 bean 执行方法
	- bean 是原始对象时，调用方法并执行
	- bean 是代理对象时，根据代理对象的运行模式运行原始方法与增强的内容，完成操作


> 这个例子中有两个切入点的配置，但是第一个 `ptx()` 并没有被使用，所以不会被读取。
> 
> ![](assets/Pasted%20image%2020240214163532.png)

验证容器中是否为代理对象
- 如果目标对象中的方法会被增强，那么容器中将存入的是目标对象的代理对象
- 如果目标对象中的方法不被增强，那么容器中将存入的是目标对象本身。

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

public class Main {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = (BookDao) ctx.getBean("bookDaoImpl");
        System.out.println(bookDao); // org.example.dao.impl.BookDaoImpl@3e0e1046
        System.out.println(bookDao.getClass()); // class com.sun.proxy.$Proxy22
    }
}
```

> 代理对象内部内部对 toString 方法进行了重写，打印对象输出的是原始对象

## AOP 配置

### 切入表达式

> - 切入点:要进行增强的方法
> - 切入点表达式：要进行增强的方法的描述方式

切入点表达式标准格式：

```java
动作关键字(访问修饰符 返回值 包名.类/接口名.方法名(参数) 异常名）

@Pointcut(execution(public User com.itheima.service.UserService.findById(int)))
```

- execution：动作关键字，描述切入点的行为动作，例如 execution 表示执行到指定切入点
- public：访问修饰符，还可以是 public，private 等，可以省略
- User：返回值，写返回值类型
- com.itheima.service：包名，多级包使用点连接
- **UserService：类/接口名称**
- findById：方法名
- int：参数，直接写参数的类型，多个类型用逗号隔开
- 异常名：方法定义中抛出指定异常，可以省略

> 方法的定义会有很多，所以如果每一个方法对应一个切入点表达式，想想这块就会觉得将来编写起来会比较麻烦，有没有更简单的方式呢?

通配符
- `*`：**单个**独立的任意符号，可以独立出现，也可以作为前缀或者后缀的匹配符出现
- `..`：**多个**连续的任意符号，可以独立出现，常用于简化包名与参数的书写
- `+`：专用于匹配**子类类型**
	- 这个使用率较低，JavaEE 开发中，继承机会就一次，使用都很慎重

```java
execution（public * com.itheima.*.UserService.find*(*))
// 匹配com.itheima包下的任意包中的UserService类或接口中所有find开头的带有一个参数的方法

execution（public User com..UserService.findById(..)
// 匹配com包下的任意包中的UserService类或接口中所有名称为findById的方法

execution(* *..*Service+.*(..))
// *Service+，表示所有以Service结尾的接口的子类。

execution(* com.itheima.*.*Service.find*(..))
// 将项目中所有业务层方法的以find开头的方法匹配
```

书写技巧：
- 描述切入点通**常描述接口**，而不描述实现类,如果描述到实现类，就出现紧耦合了
- 访问控制修饰符针对接口开发均采用 public 描述（**可省略访问控制修饰符描述**）
- 返回值类型对于增删改类使用精准类型加速匹配，对于查询类使用 `*` 通配快速描述
- **包名**书写**尽量不使用 `..` 匹配**，效率过低，常用 `*` 做单个包描述匹配，或精准匹配
- **接口名/类名**书写名称与模块相关的**采用 `*` 匹配**，
	- 例如 UserService 书写成*`Service`，绑定业务层接口名
- **方法名**书写以**动词**进行**精准匹配**，名词采用 `_` 匹配，
	- 例如 getById 书写成 `getBy*`，selectAll 书写成 selectAll
- 参数规则较为复杂，根据业务方法灵活调整
- 通常**不使用异常**作为**匹配**规则

> 所有代码按照标准规范开发，否则以上技巧全部失效

### 通知类型

> 通知具体要添加到切入点的哪里?

5 种通知类型
- 前置通知
	- `@Before`
- 后置通知：不管方法执行的过程中有没有抛出异常都会执行
	- `@After`
- **环绕通知 (重点)**：它可以**实现其他四种通知类型的功能**
	- `@Around`
- 返回后通知 (了解)：只有方法正常执行结束后才进行
	- `@AfterReturning`
- 抛出异常后通知 (了解)：只有方法执行出异常才进行
	- `@AfterThrowing`

![](assets/Pasted%20image%2020240214202400.png)

@Around 注意事项
- 环绕通知必须依赖**形参 ProceedingJoinPoint** 才能实现对原始方法的调用，进而实现原始方法调用前后同时添加通知
	- 通知中如果未对原始方法进行调用，将跳过原始方法的执行
- 要根据原始方法的返回值来设置环绕通知的返回值：
	- 对原始方法的调用可以不接收返回值，通知方法设置成 void 即可；如果接收返回值，最好**设定为 Object 类型**
	- 原始方法的返回值如果是 void 类型，通知方法的返回值类型可以设置成 void，也可以设置成 Object
	- 在环绕通知中是可以对原始方法**返回值修改**的。
- 由于无法预知原始方法运行后是否会抛出异常，因此环绕通知方法**必须要处理 Throwable 异常**
	- 接口 ProceedingJoinPoint 中，`public Object proceed() throws Throwable;`

---
环绕通知演示

```java
@Component
@Aspect
public class MyAdvice {
    
    @Pointcut("execution(int com.itheima.dao.BookDao.select())")
    private void pt2(){}
    
    @Around("pt2()")
    public void aroundSelect(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("around before advice ...");
        //表示对原始操作的调用
        pjp.proceed();
        System.out.println("around after advice ...");
    }
}

public class App {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        BookDao bookDao = ctx.getBean(BookDao.class);
        int num = bookDao.select();
    }
}
```

运行后会报错，错误内容为:

```java
Exception in thread "main" org.springframework.aop.AopInvocationException: ==Null return value from advice does not match primitive return type for: public abstract int com.itheima.dao.BookDao.select()==
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:226)
	at com.sun.proxy.$Proxy19.select(Unknown Source)
	at com.itheima.App.main(App.java:12)
```

错误大概的意思是：空的返回 不匹配 原始方法的 int 返回
* void 就是返回 Null
* 原始方法就是 BookDao 下的 select 方法

所以使用环绕通知，要根据原始方法的返回值来设置环绕通知的返回值

### 获取签名信息

环绕通知的形参中 ProceedingJoinPoint 对象可以获取增强方法的签名信息：
- 获取签名信息
- 获取执行操作名称 (接口名)
- 获取执行操作名称(方法名)

```java
@Component
@Aspect
public class ProjectAdvice {
    //配置业务层的所有方法
    @Pointcut("execution(* com.itheima.service.*Service.*(..))")
    private void servicePt(){}
    //@Around("ProjectAdvice.servicePt()") 可以简写为下面的方式
    @Around("servicePt()")
    public void runSpeed(ProceedingJoinPoint pjp){
        //获取执行签名信息
        Signature signature = pjp.getSignature();
        //通过签名获取执行操作名称(接口名)
        String className = signature.getDeclaringTypeName();
        //通过签名获取执行操作名称(方法名)
        String methodName = signature.getName();
        
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000; i++) {
           pjp.proceed();
        }
        long end = System.currentTimeMillis();
        System.out.println("万次执行："+ className+"."+methodName+"---->" +(end-start) + "ms");
    } 
}
```

### 通知获取数据

数据：参数、返回值和异常
- “非环绕通知”：在方法上添加 JoinPoint 参数
- “环绕通知”：在方法上添加 ProceedingJoinPoint 参数

---
1）获取参数

JoinPoint 的 `Object[] getArgs();`

- “环绕通知”可以修改参数：ProceedingJoinPoint 的
	- `public Object proceed() throws Throwable;` 在调用中会自动传入原始方法的参数
	- `public Object proceed(Object[] args) throws Throwable` 手动传入方法参数

> ProceedingJoinPoint 是 JoinPoint 的子接口，JoinPoint 中是没有 `proceed()` 方法的

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

	// 非环绕通知
    @Before("pt()")
    public void before(JoinPoint jp) 
        Object[] args = jp.getArgs();
        System.out.println(Arrays.toString(args));
        System.out.println("before advice ..." );
    }
    
	// 环绕通知
	@Around("pt()")
    public Object around(ProceedingJoinPoint pjp)throws Throwable {
        Object[] args = pjp.getArgs();
        System.out.println(Arrays.toString(args));
        // 修改参数
        args[0] = 666;
        Object ret = pjp.proceed(args);
        return ret;
    }
}
```

---
2）获取返回值

- “环绕通知”获取返回值：`public Object proceed() throws Throwable;`，执行陨石方法并获取到返回值
- “非环绕通知”获取返回值：
	- `@AfterReturning(returning="参数名")` 
		- `String returning() default "";`
	- 通知方法上添加返回参数
	- 注意：
		- `returning` 和 方法参数名 要一致
		- 通知方法的参数类型建议写成 Object 类型
		- 通知方法如果有 JointPoint 参数，必须放第一位

![](assets/Pasted%20image%2020240215204548.png)

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

	// 环绕通知
    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable{
        Object ret = pjp.proceed(args);
        return ret;
    }
    
	// 返回后通知
	@AfterReturning(value = "pt()", returning = "ret")
    public void afterReturning(Object ret) {
        System.out.println("afterReturning advice ..."+ret);
    }
}
```

---
3）获取异常

- “环绕通知”获取异常：直接捕获
- “抛出异常后通知”获取异常：
	- `@AfterThrowing(throwing ="")` 
	- 通知方法上添加异常参数 Throwable
	- 注意：
		- `throwing` 和 异常参数名 一致

```java
@Component
@Aspect
public class MyAdvice {
    @Pointcut("execution(* com.itheima.dao.BookDao.findName(..))")
    private void pt(){}

	// 环绕通知
    @Around("pt()")
    public Object around(ProceedingJoinPoint pjp){
        Object ret = null;
        try{
            ret = pjp.proceed(args);
        }catch(Throwable throwable){
            t.printStackTrace();
        }
        return ret;
    }
    
	// 抛出异常后通知
	@AfterThrowing(value = "pt()",throwing = "t")
    public void afterThrowing(Throwable t) {
        System.out.println("afterThrowing advice ..."+t);
    }
}
```

# ---------- 事务

## Spring 事务简介

- 事务作用：在数据层保障一系列的数据库操作同成功同失败
- Spring 事务作用：在数据层或业务层保障一系列的数据库操作同成功同失败

Spring 为了管理事务，提供了一个平台事务管理器 `PlatformTransactionManager`
- commit 是用来提交事务，rollback 是用来回滚事务。

```java
public interface PlatformTransactionManager extends TransactionManager {
	TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;
	
	void commit(TransactionStatus status) throws TransactionException;
	
	void rollback(TransactionStatus status) throws TransactionException;
}
```

PlatformTransactionManager 是一个接口，DataSourceTransactionManager 是它的一个具体实现类

从名称上可以看出，我们只需要给它一个 DataSource 对象，它就可以帮你去在业务层管理事务。其内部采用的是 JDBC 的事务。所以说如果你持久层采用的是 JDBC 相关的技术，就可以采用这个事务管理器来管理你的事务，例如 Mybatis

## Spring事务基本使用

开启注解式事务驱动

```java
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:jdbc.properties")
@Import({JdbcConfig.class,MybatisConfig.class
// 开启注解式事务驱动
@EnableTransactionManagement
public class SpringConfig {
}
```

业务层添加 Spring 事务管理注解
- `@Transactional` 可以写在接口类上、接口方法上、实现类上和实现类方法上
	- 加在类或接口上代表所有方法或实现类方法都会有事务
- 通常添加在**业务层接口**中而不是实现类中，降低耦合

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    // 配置当前接口方法具有事务
    public void transfer(String out, String in, Double money) ;
}

@Service
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;
    
	@Transactional
    public void transfer(String out,String in ,Double money) {
        accountDao.outMoney(out,money);
        int i = 1/0;
        accountDao.inMoney(in,money);
    }
}
```

配置事务管理器
- 事务管理器要根据实现技术进行选择，例如 MyBatis 框架使用的是 JDBC 事务
- 注意：`DataSourceTransactionManager` 和 `SqlSessionFactoryBean` 使用的是同一个数据源 DataSource

```java
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

    // 配置事务管理器，mybatis使用的是jdbc事务
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource){
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(dataSource);
        return transactionManager;
    }
}
```

> MyBatis 需要注入的 SqlSessionFactoryBean：
> 
> ```java
>     @Bean
>     public SqlSessionFactoryBean sqlSessionFactory(DataSource dataSource) {
>         SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
>         // 设置模型类的别名扫描
>         ssfb.setTypeAliasesPackage("org.example.domain");
>         // 设置数据源
>         ssfb.setDataSource(dataSource);
>         return ssfb;
>     }
> ```

%% -- %%
## Spring事务角色

- 事务管理员：**发起**事务方，在 Spring 中通常指代业务层开启事务的方法
- 事务协调员：**加入**事务方，在 Spring 中通常指代数据层方法，也可以是业务层方法

---
未开启Spring事务之前：

![](assets/Pasted%20image%2020240216231324.png)

- AccountDao 的 outMoney 因为是修改操作，会开启一个事务 T1
- AccountDao 的 inMoney 因为是修改操作，会开启一个事务 T2
- AccountService 的 transfer 没有事务，
    - 运行过程中如果没有抛出异常，则 T1 和 T2 都正常提交，数据正确
    - 如果在两个方法中间抛出异常，T1 因为执行成功提交事务，T2 因为抛异常不会被执行，就会导致数据出现错误

开启 Spring 的事务管理后

![](assets/Pasted%20image%2020240216231643.png)

- transfer 上添加了 `@Transactional` 注解，在该方法上就会有一个事务 T
- AccountDao 的 outMoney 方法的事务 T1 加入到 transfer 的事务 T 中
- AccountDao 的 inMoney 方法的事务 T2 加入到 transfer 的事务 T 中
- 这样就保证他们在同一个事务中，当业务层中出现异常，整个事务就会回滚，保证数据的准确性。

## Spring事务属性

> `@Transactional` 的属性

### 事务配置

这些属性都可以在`@Transactional`注解的参数上进行设置。

- `boolean readOnly() default false;`：true只读事务，false读写事务
	- 增删改要设为false，查询设为true。
- `int timeout() default -1`：设置事务超时时间（单位秒），在设置时间之内事务没有提交成功就自动回滚，-1 表示不设置超时时间。
- `Class<? extends Throwable>[] rollbackFor() default {};`：当出现指定异常进行事务回滚

![](assets/Pasted%20image%2020240217113856.png)

### 事务传播属性

引入案例：转账业务追加日志

需求：实现任意两个账户间转账操作，并对每次转账操作在数据库进行留痕

创建日志表

```java
create table tbl_log(
   id int primary key auto_increment,
   info varchar(255),
   createDate datetime
)
```

日志业务类
```java
public interface LogDao {
    @Insert("insert into tbl_log (info,createDate) values(#{info},now())")
    void log(String info);
}

public interface LogService {
    void log(String out, String in, Double money);
}

@Service
public class LogServiceImpl implements LogService {
    @Autowired
    private LogDao logDao;
    
	@Transactional
    public void log(String out,String in,Double money ) {
        logDao.log("转账操作由"+out+"到"+in+",金额："+money);
    }
}
```

转账业务中添加记录日志

```java
public interface AccountService {
    /**
     * 转账操作
     * @param out 传出方
     * @param in 转入方
     * @param money 金额
     */
    public void transfer(String out,String in ,Double money)throws IOException ;
}

@Service
public class AccountServiceImpl implements AccountService {
    @Autowired
    private AccountDao accountDao;
    @Autowired
    private LogService logService;
    
	@Transactional
    public void transfer(String out,String in ,Double money) {
        try{
            accountDao.outMoney(out,money);
            // int a = 1/0;
            accountDao.inMoney(in,money);
        }finally {
            logService.log(out,in,money);
        }
    }
}
```

当程序正常运行，tbl_account 表中转账成功，tbl_log 表中日志记录成功。当转账业务之间出现异常(int i =1/0)，转账失败，tbl_account 成功回滚，但是 tbl_log 表未添加数据

失败原因：日志的记录与转账操作隶属同一个事务，同成功同失败

---
事务传播行为：事务协调员对事务管理员所携带事务的处理态度
- `@Transactional` 注解的 propagation 属性

propagation的属性值：

![](assets/Pasted%20image%2020240217152744.png)

修改logService改变事务的传播行为

```java
@Service
public class LogServiceImpl implements LogService {

    @Autowired
    private LogDao logDao;
    
	// propagation设置事务属性：传播行为设置为当前操作需要新事务
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String out,String in,Double money ) {
        logDao.log("转账操作由"+out+"到"+in+",金额："+money);
    }
}
```

![](assets/Pasted%20image%2020240217154034.png)

-----







# IOC 其他


## IOC 容器

IOC 思想基于 IOC 容器完成，IOC 容器底层就是对象工厂

在Spring中，IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map中存放的是各种对象（以后可以看看IOC源码）

## Bean作用域

在 Spring 里面，设置创建 bean 实例是单实例还是多实例
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





# AOP其他

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

# -------------------- SpringMVC

# SpringMVC概述

SpringMVC 是隶属于 Spring 框架的一部分，主要是用来进行 Web 开发，是对 Servlet 进行了封装。

![](assets/Pasted%20image%2020240217162507.png)

SpringMVC 主要负责的就是
- controller 如何接收请求和数据
- 如何将请求和数据转发给业务层
- 如何将响应数据转换成 json 发回到前端


SpringMVC 是一种基于 Java 实现 MVC 模型的轻量级 Web 框架
- 优点：
	- 使用简单、开发便捷(相比于Servlet)
	- 灵活性强

# 入门案例

## 基本使用

新建一个 web 项目

导入坐标
- servlet 的坐标添加 `<scope>provided</scope>`
- 只导入了 `spring-webmvc` jar 包的原因是它会自动依赖 spring 相关坐标

> - scope 是 maven 中 jar 包依赖作用范围的描述，
> - 如果不设置默认是 `compile` 在在编译、运行、测试时均有效
> - 如果运行有效的话就会和 tomcat 中的 servlet-api 包发生冲突，导致启动报错
> - provided 代表的是该包只在编译和测试的时候用，运行的时候无效直接使用 tomcat 中的，就避免冲突

```xml
<packaging>war</packaging>

<dependencies>
<dependency>
  <groupId>javax.servlet</groupId>
  <artifactId>javax.servlet-api</artifactId>
  <version>3.1.0</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-webmvc</artifactId>
  <version>5.2.10.RELEASE</version>
</dependency>
</dependencies>

<build>
<plugins>
  <plugin>
	<groupId>org.apache.tomcat.maven</groupId>
	<artifactId>tomcat7-maven-plugin</artifactId>
	<version>2.1</version>
	<configuration>
	  <port>80</port>
	  <path>/</path>
	</configuration>
  </plugin>
</plugins>
</build>
```

![](assets/Pasted%20image%2020240217170016.png)

创建 springmvc 控制器类

```java
@Controller
public class UserController {
    @RequestMapping("/save")
    @ResponseBody
    public String save() {
        System.out.println("user save ...");
        return "{'info':'springmvc'}";
    }
}
```

初始化 springmvc 环境，设定 springmvc 加载指定的 bean

```java
@Configuration
@ComponentScan("com.itheima.controller")
public class SpringMvcConfig {
}
```

初始化 Servlet 容器，加载 SpringMVC 环境，并设置 SpringMVC 技术处理的请求
- 将 web.xml 删除，换成 ServletContainersInitConfig
- AbstractDispatcherServletInitializer 类是 SpringMVC 提供的**快速初始化 Web3.0 容器的抽象类**，它提供了三个接口方法供用户实现
    - `createServletApplicationContext()` ：创建 Servlet 容器时，加载 SpringMVC 对应的 bean 并放入 WebApplicationContext 对象范围中，而 WebApplicationContext 的作用范围为 ServletContext 范围，即整个 web 容器范围
    - `getServletMappings()`：设定 SpringMVC 对应的请求映射路径，即 SpringMVC 拦截哪些请求
    - `createRootApplicationContext()`：如果创建 Servlet 容器时需要加载非 SpringMVC 对应的 bean，使用当前方法进行，使用方式同 createServletApplicationContext
- createServletApplicationContext 用来加载 SpringMVC 环境， createRootApplicationContext 用来加载 Spring 环境

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    //加载springmvc配置类
    protected WebApplicationContext createServletApplicationContext() {
        //初始化WebApplicationContext对象
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        //加载指定配置类
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }

    //设置由springmvc控制器处理的请求映射路径
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //加载spring配置类
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }
}
```

三个基本注解

|名称|@Controller|
|---|---|
|类型|类注解|
|位置|SpringMVC 控制器类定义上方|
|作用|设定 SpringMVC 的核心控制器 bean|

|名称|@RequestMapping|
|---|---|
|类型|类注解或方法注解|
|位置|SpringMVC 控制器类或方法定义上方|
|作用|设置当前控制器方法请求访问路径|
|相关属性|value (默认)，请求访问路径 |

|名称|@ResponseBody|
|---|---|
|类型|类注解或方法注解|
|位置|SpringMVC 控制器类或方法定义上方|
|作用|设置当前控制器方法响应内容为当前返回值，无需解析|

## 工作流程

启动服务器初始化过程
- 服务器启动，执行 ServletContainersInitConfig 类，初始化 web 容器
    - 功能类似于以前的 web.xml
- 执行 createServletApplicationContext 方法，创建了 WebApplicationContext 对象
    - 该方法加载 SpringMVC 的配置类 SpringMvcConfig 来初始化 SpringMVC 的容器
- 加载 SpringMvcConfig 配置类
- 执行 `@ComponentScan` 加载对应的 bean
    - 扫描指定包及其子包下所有类上的注解，如 Controller 类上的 `@Controller` 注解
- 加载 UserController，每个 `@RequestMapping` 的名称对应一个具体的方法
	- 加载 UserController，每个 `@RequestMapping` 的名称对应一个具体的方法
- 执行 getServletMappings 方法，设定 SpringMVC 拦截请求的路径规则

![](assets/Pasted%20image%2020240217195113.png)

单次请求过程
- 发送请求 `http://localhost/save`
- web 容器发现该请求满足 SpringMVC 拦截规则，将请求交给 SpringMVC 处理
- 解析请求路径 `/save`
- 由/save 匹配执行对应的方法 `save()`
- 执行 `save()`
- 检测到有 `@ResponseBody` 直接将 `save()` 方法的返回值作为响应体返回给请求方

## bean加载控制

![](assets/Pasted%20image%2020240217205257.png)

controller、service 和 dao 这些类都需要被容器管理成 bean 对象，那么到底是该让 SpringMVC 加载还是让 Spring 加载呢?

- SpringMVC 加载其相关 bean(表现层 bean),也就是 controller 包下的类
- Spring 控制的 bean
    - 业务 bean(Service)
    - 功能 bean(DataSource,SqlSessionFactoryBean,MapperScannerConfigurer 等)
        
如何让 Spring 和 SpringMVC 分开加载各自的内容？

---

避免 Spring 错误加载到 SpringMVC 的 bean
- 加载 Spring 控制的 bean 的时候**排除掉 SpringMVC 控制的 bean**
	- 方式一：Spring 加载的 bean 设定扫描范围为精准范围，
		- 例如 service 包、dao 包等
	- 方式二：Spring 加载的 bean 设定扫描范围为 `com.itheima`，排除掉 controller 包中的 bean
- 不区分 Spring 与 SpringMVC 的环境，加载到同一个环境中

方式一：

```java
@Configuration
@ComponentScan({"com.itheima.service","com.itheima.dao"})
public class SpringConfig {}
```

方式二：

```java
@Configuration  
@ComponentScan(value="com.itheima",  
	excludeFilters=@ComponentScan.Filter(  
		type = FilterType.ANNOTATION,  
		classes = Controller.class  
	)  
)  
public class SpringConfig {}
```

测试 controller 类已经被排除掉了
- 方式二要把 SpringMvcConfig 配置类上的 `@ComponentScan` 注解去掉。因为 SpringMvcConfig 上有一个 `@Configuration` 注解，也会被 Spring 扫描到，此外还有一个 `@ComponentScan`，把 controller 类又给扫描进来了

```java
public class App{
	public static void main (String[] args){
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
        System.out.println(ctx.getBean(UserController.class));
    }
}
```

> SpringMvcConfig 和 SpringConfig 都在 config 包下，所以方式二还是扫描到了 SpringMvcConfig，解决方案是把 SpringMVC 的配置类移出 Spring 配置类的扫描范围即可。

---
tomcat 服务器启动后分别加载 SpringConfig（业务 bean）和 SpringMvcConfig（控制器）

```java
public class ServletContainersInitConfig extends AbstractDispatcherServletInitializer {
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }
    
    protected WebApplicationContext createRootApplicationContext() {
      AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }
    
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```

## 简化配置

对于上述的配置方式，Spring 还提供了一种更简单的配置方式，可以不用再去创建 `AnnotationConfigWebApplicationContext` 对象，不用手动 `register` 对应的配置类

```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {  
     protected Class<?>[] getRootConfigClasses() {  
         return new Class[]{SpringConfig.class};  
     }  
 ​  
     protected Class<?>[] getServletConfigClasses() {  
         return new Class[]{SpringMvcConfig.class};  
     }  
 ​  
     protected String[] getServletMappings() {  
         return new String[]{"/"};  
     }  
 }
```

![](assets/Pasted%20image%2020240217213403.png)

# 请求与响应

### 设置请求映射路径

`@RequestMapping`
- 类型：方法注解、类注解
- 位置：控制器方法上、控制器类上
- 作用：
	- 设置当前控制器方法请求访问路径，
	- 如果设置在类上统一设置当前控制器方法请求访问路径前缀
- 属性：value（默认），请求访问路径，或访问路径前缀
	- value 属性前面加不加 `/` 都可以

```java
@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("user save ...");
        return "{'module':'user save'}";
    }
    
    @RequestMapping("/delete")
    @ResponseBody
    public String save(){
        System.out.println("user delete ...");
        return "{'module':'user delete'}";
    }
}

@Controller
@RequestMapping("/book")
public class BookController {

    @RequestMapping("/save")
    @ResponseBody
    public String save(){
        System.out.println("book save ...");
        return "{'module':'book save'}";
    }
}
```

## 请求参数传递（接收）

|名称|@RequestParam|
|---|---|
|类型|形参注解|
|位置|SpringMVC 控制器方法形参定义前面|
|作用|绑定请求参数与处理器方法形参间的关系|
|相关参数|required：是否为必传参数，defaultValue：参数默认值 |

### 中文乱码

1）Get 请求乱码

> Tomcat 8 及之后的版本，处理 get 请求的编码默认为 UTF-8
> 
> Tomcat8.5 以后的版本已经处理了中文乱码的问题，但是 IDEA 中的 Tomcat 插件目前只到 Tomcat7，所以需要修改 pom.xml 来解决 GET 请求中文乱码问题

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.tomcat.maven</groupId>
        <artifactId>tomcat7-maven-plugin</artifactId>
        <version>2.1</version>
        <configuration>
          <port>80</port><!--tomcat端口号-->
          <path>/</path> <!--虚拟目录-->
          <uriEncoding>UTF-8</uriEncoding><!--访问路径编解码字符集-->
        </configuration>
      </plugin>
    </plugins>
  </build>
```

2）Post 请求乱码

配置过滤器

```java
public class ServletContainersInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    protected Class<?>[] getRootConfigClasses() {
        return new Class[0];
    }

    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }

    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    //乱码处理
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        filter.setEncoding("UTF-8");
        return new Filter[]{filter};
    }
}
```

### 4 种类型参数传递

在约定中
- Get 请求：url 地址 传参
- Post 请求：form 表单 传参

> 并不是强制的

表单传参：

![](assets/Pasted%20image%2020240218144208.png)

---
1）普通参数：

请求参数名 与 形参参数名
- 相同：定义形参即可接收参数
- 不同：使用 `@RequestParam` 注解

> 写上 `@RequestParam` 注解框架就不需要自己去解析注入，能提升框架处理性能

```http
Get发送请求：
localhost:81/user/commonParam?name=张三&age=11
```

```java
@Controller
@RequestMapping("/user")
public class UserController {
	@RequestMapping("/commonParamDifferentName")
	@ResponseBody
	public String commonParamDifferentName(@RequestParam("name") String userName , int age){
	    System.out.println("普通参数传递 userName ==> "+userName);
	    System.out.println("普通参数传递 age ==> "+age);
	    return "{'module':'common param different name'}";
	}
}
```

2）pojo/嵌套pojo 参数

> GET 和 POST 发送请求数据的方式不变

- 请求参数 key 的名称要和 **POJO 中属性的名称一致**，否则无法封装，定义 POJO 类型形参即可接收参数
- 嵌套 POJO 参数：按照对象层次结构关系即可接收嵌套 POJO 属性参数

```http
Get 发送请求：
localhost:81/user/pojoParam?name=张三&age=11&address.province=山东&address.city=烟台
```

```java
@Data
public class Address {
    private String province;
    private String city;
}

@Data
public class User {
    private String name;
    private int age;
    private Address address;
}

@Controller
@RequestMapping("/user")
public class UserController {
	// 
	@RequestMapping("/pojoParam")  
	@ResponseBody  
	public String pojoParam(User user) {  
	    System.out.println("pojo参数传递 user ==> " + user);  
	    return "{'module':'pojo param'}";  
	}
}
```

3）数组 参数

> GET 和 POST 发送请求数据的方式不变

请求参数名 与 形参对象属性名 **相同** 且 请求参数为**多个**，定义数组类型即可接收参数

```http
Get 发送请求：
localhost:81/user/arrayParam?likes=唱&likes=跳&likes=rap&likes=篮球
```

```java
@Controller
@RequestMapping("/user")
public class UserController {
	@RequestMapping("/arrayParam")  
	@ResponseBody  
	public String arrayParam(String[] likes) {  
	    System.out.println("数组参数传递 likes ==> " + Arrays.toString(likes));  
	    return "{'module':'array param'}";  
	}
}
```

4）集合类型参数

> GET 和 POST 发送请求数据的方式不变

使用 `@RequestParam` 注解，请求参数名 与 形参集合对象名 **相同** 且 请求参数为**多个**，

```http
Get 发送请求：
localhost:81/user/listParam?likes=唱&likes=跳&likes=rap&likes=篮球
```

```java
@Controller
@RequestMapping("/user")
public class UserController {
	@RequestMapping("/listParam")  
	@ResponseBody  
	    public String listParam(@RequestParam List<String> likes) {  
	    System.out.println("集合参数传递 likes ==> " + likes);  
	    return "{'module':'list param'}";  
	}
}
```

> 不加 `@RequestParam` 运行会报错，原因是 SpringMVC 将 List 看做是一个 POJO 对象来处理，将其创建一个对象并准备把前端的数据封装到对象中，但是 List 是一个接口无法创建对象，所以报错。
> 
> ![](assets/Pasted%20image%2020240218172404.png)

### json 数据参数传递


| 名称 | `@RequestBody` |
| ---- | ---- |
| 类型 | 形参注解 |
| 作用 | 将请求中请求体所包含的数据传递给请求参数，此注解一个处理器方法只能使用一次 |

`@RequestBody` 与 `@RequestParam`
- 区别
    - `@RequestParam` 用于接收 url 地址传参，表单传参【application/x- www-form-urlencoded】
    - `@RequestBody` 用于接收 json 数据【application/json】
- 应用
    - 后期开发中，发送 json 格式数据为主，`@RequestBody` 应用较广
    - 如果发送非 json 格式数据，选用 `@RequestParam` 接收请求参数

---
一次性工作：
- SpringMVC 默认使用的是 jackson 来处理 json 的转换，所以需要添加 jackson 依赖
- SpringMVC 的配置类中添加 `@EnableWebMvc` 注解开启 SpringMVC 多项辅助功能，这里面**包含**了将 JSON 转换成对象的功能。

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
```

```java
@Configuration
@ComponentScan("com.itheima.controller")
// 开启json数据类型自动转换
@EnableWebMvc
public class SpringMvcConfig {}
```

---
1）json 普通数组

```json
[
    "唱",
    "跳",
    "rap",
    "篮球"
]
```

```java
@Controller
@RequestMapping("/user")
public class UserController {
	@RequestMapping("/listParamForJson")  
	@ResponseBody  
	public String listParamForJson(@RequestBody List<String> likes) {  
	    System.out.println("list common(json)参数传递 list ==> " + likes);  
	    return "{'module':'list common for json param'}";  
	}
}
```

2）json 对象

```java
{
    "name": "张三",
    "age": "11",
    "address": {
        "province": "beijing",
        "city": "beijing"
    }
}
```

```java
@Controller
@RequestMapping("/user")
public class UserController {
	@RequestMapping("/pojoParamForJson")  
	@ResponseBody  
	public String pojoParamForJson(@RequestBody User user) {  
	    System.out.println("pojo(json)参数传递 user ==> " + user);  
	    return "{'module':'pojo for json param'}";  
	}
}
```

3）json 对象数组

```java
[
    {
        "name": "itcast",
        "age": 15
    },
    {
        "name": "itheima",
        "age": 12
    }
]
```

```java
@Controller
@RequestMapping("/user")
public class UserController {
	@RequestMapping("/listPojoParamForJson")
	@ResponseBody
	public String listPojoParamForJson(@RequestBody List<User> list){
	    System.out.println("list pojo(json)参数传递 list ==> "+list);
	    return "{'module':'list pojo for json param'}";
	}
```

### 日期类型参数传递

> 日期类型比较特殊，因为对于日期的格式有N多中输入方式，比如:
> - 2088-08-18
> - 2088/08/18
> - 08/18/2088
> - ......

| 名称 | `@DateTimeFormat` |
| ---- | ---- |
| 类型 | 形参注解 |
| 作用 | 设定日期时间型数据格式 |
| 相关属性 | pattern：指定日期时间格式字符串 |

---
把参数设置为日期类型，
- SpringMVC 默认支持的字符串转日期的格式为 `yyyy/MM/dd`
- `@DateTimeFormat` 自定义日期时间的格式

```http
localhost:81/user/dataParam?date=2088/08/08&date1=2088-08-08&date2=2088/08/08 8:08:08
```

```java
@RequestMapping("/dataParam")
@ResponseBody
public String dataParam(Date date,
						@DateTimeFormat(pattern = "yyyy-MM-dd") Date date1,
						@DateTimeFormat(pattern = "yyyy/MM/dd HH:mm:ss") Date date2) {
	System.out.println("参数传递 date ==> " + date);
	System.out.println("参数传递 date1(yyyy-MM-dd) ==> " + date1);
	System.out.println("参数传递 date2(yyyy/MM/dd HH:mm:ss) ==> " + date2);
	return "{'module':'data param'}";
}
```

## Converter

> - 前端传递字符串，后端使用日期Date接收
> - 前端传递JSON数据，后端使用对象接收
> - 前端传递字符串，后端使用Integer接收
> - 后台需要的数据类型有很多中
> - 在数据的传递过程中存在很多类型的转换
> 
> 问：谁来做这个类型转换?

SpringMVC 中提供了类型转换接口和实现类

```java
/**
*	S: the source type
*	T: the target type
*/
@FunctionalInterface
public interface Converter<S, T> {
    @Nullable
    //该方法就是将从页面上接收的数据(S)转换成我们想要的数据类型(T)返回
    T convert(S source);
}
```

Converter 所属的包为 `org.springframework.core.convert.converter`

Converter 接口的实现类，用来实现不同数据类型之间的转换，如
- 请求参数年龄数据（String --> Integer）
- 日期格式转换（String --> Date）

![](assets/Pasted%20image%2020240218202444.png)

## 响应

响应主要包括：
- 响应页面
- 响应数据
    - 文本数据
    - json 数据

> 因为异步调用是目前常用的主流方式，所以我们需要更关注的就是如何返回 JSON 数据，对于其他只需要认识了解即可。
> 
> 响应页面跳过了

|名称|`@ResponseBody` |
|---|---|
|类型|方法\类注解|
|位置|SpringMVC 控制器方法定义上方和控制类上|
|作用|设置当前控制器返回值作为响应体, 写在类上，该类的所有方法都有该注解功能|
|相关属性|pattern：指定日期时间格式字符串|
- 方法的返回值为“字符串”，会将其作为**文本内容**直接响应给前端
- 方法的返回值为“对象”，会将对象**转换成 JSON** 响应给前端

###  响应纯文本数据

```java
@RequestMapping("/toText")
@ResponseBody
public String toText() {
	System.out.println("返回纯文本数据");
	return "response text";
}
```

### 响应json数据

> jackson 依赖要导入，`@EnableWebMvc` 注解要加上

1）响应 pojo 对象

```java
@RequestMapping("/toJsonPOJO")
@ResponseBody
public User toJsonPOJO(){
	System.out.println("返回json对象数据");
	User user = new User();
	user.setName("itcast");
	user.setAge(15);
	return user;
} 
```

2）响应 POJO 集合对象

```java
@RequestMapping("/toJsonList")
@ResponseBody
public List<User> toJsonList() {
	System.out.println("返回json集合数据");
	User user1 = new User();
	user1.setName("传智播客");
	user1.setAge(15);

	User user2 = new User();
	user2.setName("黑马程序员");
	user2.setAge(12);

	List<User> userList = new ArrayList<>();
	userList.add(user1);
	userList.add(user2);

	return userList;
}
```

### HttpMessageConverter

HttpMessageConverter 接口是实现对象与 JSON 之间的转换工作

实现类如下：

![](assets/Pasted%20image%2020240218214515.png)

# Rest 风格

## 简介

*REST（Representational State Transfer）*，表现形式状态转换，它是一种软件架构风格
    
当我们想表示一个网络资源的时候，可以使用两种方式：

* 传统风格资源描述形式
	* `http://localhost/user/getById?id=1` 查询 id 为 1 的用户信息
	* `http://localhost/user/saveUser` 保存用户信息
* REST 风格描述形式
	* `http://localhost/user/1` 
	* `http://localhost/user`

REST 的优点有：
- 隐藏资源的访问行为，无法通过地址得知对资源是何种操作
- 书写简化

按照 REST 风格访问资源时使用**行为动作**区分对资源进行了何种操作

* `http://localhost/users`	查询全部用户信息 GET（查询）
* `http://localhost/users/1`  查询指定用户信息 GET（查询）
* `http://localhost/users`    添加用户信息    POST（新增/保存）
* `http://localhost/users`    修改用户信息    PUT（修改/更新）
* `http://localhost/users/1`  删除用户信息    DELETE（删除）

**描述模块的名称通常使用复数**，也就是加 s 的格式描述，表示此类资源，而非单个资源，例如：users、books、accounts......

*RESTful*：根据 REST 风格对资源进行访问称为 RESTful。

> 上述行为是约定方式，约定不是规范，可以打破，所以称 REST 风格，而不是 REST 规范

## 基本使用

