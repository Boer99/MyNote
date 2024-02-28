
# -------------------- 基础篇

# ---------- 快速入门

## 引言

学习任意一项技术，首先要知道这个技术的作用是什么，不然学完以后，你都不知道什么时候使用这个技术，也就是技术对应的应用场景。SpringBoot 技术由 Pivotal 团队研发制作，功能的话简单概括就是加速 Spring 程序的开发，这个加速要从如下两个方面来说

- Spring 程序初始搭建过程
- Spring 程序的开发过程

通过上面两个方面的定位，我们可以产生两个模糊的概念：

1. SpringBoot 开发团队认为原始的 Spring 程序初始搭建的时候可能有些繁琐，这个过程是可以简化的，那原始的 Spring 程序初始搭建过程都包含哪些东西了呢？为什么觉得繁琐呢？最基本的 Spring 程序至少有一个配置文件或配置类，用来描述 Spring 的配置信息，莫非这个文件都可以不写？此外现在企业级开发使用 Spring 大部分情况下是做 web 开发，如果做 web 开发的话，还要在加载 web 环境时加载时加载指定的 spring 配置，这都是最基本的需求了，不写的话怎么知道加载哪个配置文件/配置类呢？那换了 SpringBoot 技术以后呢，这些还要写吗？

2. SpringBoot 开发团队认为原始的 Spring 程序开发的过程也有些繁琐，这个过程仍然可以简化。开发过程无外乎使用什么技术，**导入对应的 jar 包（或坐标）然后将这个技术的核心对象交给 Spring 容器管理**，也就是配置成 Spring 容器管控的 bean 就可以了。这都是基本操作啊，难道这些东西 SpringBoot 也能帮我们简化？

## 入门案例

> 创建 SpringBoot 工程的四种方式：
> 
> 1. idea 的 Spring Initializr
> 2. 官网 https://start.spring.io
> 3. 阿里云
> 4. 手工创建 Maven 工程修改为 SpringBoot 工程
>    
> 这里采用第四种

创建 maven 工程，导入坐标
- 继承 spring-boot-starter-parent
- 添加依赖 spring-boot-starter-web

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.4</version>
    </parent>

    <groupId>com.itheima</groupId>
    <artifactId>springboot_01_04_quickstart</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

编写控制器

```java
@RestController
@RequestMapping("/books")
public class BookController {
    @GetMapping
    public String getById() {
        System.out.println("springboot is running...");
        return "springboot is running...";
    }
}
```

制作引导类 Application

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

启动项目，可以访问了！

## 简介

SpringBoot程序的核心功能及优点：

- 起步依赖（简化依赖配置）

- 自动配置（简化常用工程相关配置）

- 辅助功能（内置服务器，……）

### parent

Springboot 继承了 spring-boot-starter-parent

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.4</version>
</parent>
```

spring-boot-starter-parent 又继承了 spring-boot-dependencies，这个坐标中定义了两组信息，
- 第一组是各式各样的依赖版本号属性
- 第二组是各式各样的的依赖坐标信息，依赖版本号引用了第一组信息中定义的依赖版本属性值

```java
<properties>
    <activemq.version>5.16.3</activemq.version>
    <aspectj.version>1.9.7</aspectj.version>
    <assertj.version>3.19.0</assertj.version>
    <commons-codec.version>1.15</commons-codec.version>
    <commons-dbcp2.version>2.8.0</commons-dbcp2.version>
    <commons-lang3.version>3.12.0</commons-lang3.version>
    <commons-pool.version>1.6</commons-pool.version>
    <commons-pool2.version>2.9.0</commons-pool2.version>
    <h2.version>1.4.200</h2.version>
    <hibernate.version>5.4.32.Final</hibernate.version>
    <hibernate-validator.version>6.2.0.Final</hibernate-validator.version>
    <httpclient.version>4.5.13</httpclient.version>
    <jackson-bom.version>2.12.4</jackson-bom.version>
    <javax-jms.version>2.0.1</javax-jms.version>
    <javax-json.version>1.1.4</javax-json.version>
    <javax-websocket.version>1.1</javax-websocket.version>
    <jetty-el.version>9.0.48</jetty-el.version>
    <junit.version>4.13.2</junit.version>
</properties>
```

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

> 上面的依赖坐标定义是出现在 `<dependencyManagement>` 标签中的，其实是对引用坐标的依赖管理，并不是实际使用的坐标。因此当项目中继承了这组 parent 信息后，在不使用对应坐标的情况下，前面的这组定义是不会具体导入某个依赖的

在 maven 中继承机会只有一次，上述继承的格式还可以切换成**导入**的形式进行，在阿里云的 starter 创建工程时就使用了此种形式

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

总结
1. 开发 SpringBoot 程序要继承 spring-boot-starter-parent
2. spring-boot-starter-parent 中定义了若干个依赖管理
3. 继承 parent 模块可以**避免多个依赖使用相同技术时出现依赖版本冲突**
4. 继承 parent 的形式也可以采用引入依赖的形式实现效果

### starter

> SpringBoot 关注到开发者在实际开发时，对于依赖坐标的使用往往都有一些固定的组合方式，比如使用 spring-webmvc 就一定要使用 spring-web。每次都要固定搭配着写，非常繁琐，而且格式固定，没有任何技术含量。
> 
> SpringBoot 一看这种情况，看来需要给开发者带来一些帮助了。安排，把所有的技术使用的固定搭配格式都给开发出来，以后你用某个技术，就不用一次写一堆依赖了，还容易写错，我给你做一个东西，代表一堆东西，开发者使用的时候，直接用我做好的这个东西就好了，对于这样的固定技术搭配，SpringBoot 给它起了个名字叫做**starter**。

starter 定义了使用某种技术时对于**依赖的固定搭配格式**，也是一种最佳解决方案，使用 starter 可以帮助开发者**减少依赖配置**

|starter所属|命名规则|示例|
|---|---|---|
|官方提供|spring-boot-starter-技术名称|spring-boot-starter-web spring-boot-starter-test|
|第三方提供|第三方技术名称-spring-boot-starter|druid-spring-boot-starter|
|第三方提供|第三方技术名称-boot-starter（第三方技术名称过长，简化命名）|mybatis-plus-boot-starter|

实际开发
- 使用任意坐标时，仅书写 GAV 中的 G 和 A，**V 由 SpringBoot 提供**，
	- 除非 SpringBoot 未提供对应版本 V（Druid之类的）
- 如发生坐标错误，再指定 Version (要小心版本冲突)
- 如果发现坐标出现了冲突现象，**覆盖** SpringBoot 提供给我们的配置管理
	- 方式一：直接写坐标
	- 方式二：覆盖 `<properties>` 中定义的版本号

---
例如：spring-boot-starter-web

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

在 spring-boot-starter-web 中又定义了若干个具体依赖的坐标

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-json</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>2.5.4</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.9</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

### 引导类

SpringBoot 的引导类是 Boot 工程的执行入口，运行 main 方法就可以启动项目
- 最典型的特征就是当前类上方声明了一个注解 `@SpringBootApplication`

SpringBoot 工程运行后初始化 Spring 容器，扫描**引导类所在包**加载 bean

```java
@SpringBootApplication
public class Springboot0101QuickstartApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctx = SpringApplication.run(Springboot0101QuickstartApplication.class, args);
        BookController bean = ctx.getBean(BookController.class);
        System.out.println("bean======>" + bean);
    }
}
```

### 内嵌 Tomcat

内嵌 Tomcat 服务器是 
- SpringBoot 辅助功能之一
- 工作原理是**将 Tomcat 服务器作为对象运行，并将该对象交给 Spring 容器管理**

---
> 内嵌 Tomcat 定义位置在哪？

导入的 web 相关的 starter

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring-boot-starter-web 其中又包含了 spring-boot-starter-tomcat

```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<version>2.5.4</version>
	<scope>compile</scope>
</dependency>
```

spring-boot-starter-tomcat 里面有一个核心的坐标，tomcat-embed-core，叫做 tomcat 内嵌核心。就是这个东西把 tomcat 功能引入到了我们的程序中。

```java
<dependencies>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-core</artifactId>
        <version>9.0.52</version>
        <scope>compile</scope>
    </dependency>
</dependencies>
```

> 内嵌 Tomcat 运行原理是什么？
> 
> Tomcat 服务器是一款软件，而且是一款使用 java 语言开发的软件，tomcat 安装目录中保存有好多个 jar。

tomcat 服务器运行其实是以对象的形式在 Spring 容器中运行的，具体运行的就是上前面提到的那个 tomcat 内嵌核心

> 是否可以换个服务器呢？根据 SpringBoot 的工作机制，用什么技术，加入什么依赖就行了。SpringBoot 提供了 3 款内置的服务器
> 
> - *tomcat*(默认)：apache 出品，粉丝多，应用面广，负载了若干较重的组件
> - *jetty*：更轻量级，负载性能远不及 tomcat
> - *undertow*：负载性能勉强跑赢 tomcat

变更内嵌服务器思想是去除现有服务器，添加全新的服务器

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jetty</artifactId>
    </dependency>
</dependencies>
```

# ---------- 基础配置

> SpringBoot 没有具体的功能，它在辅助加快 Spring 程序的开发效率。我们发现现在几乎不用做任何的配置，功能就有了，确实很好用。
> 
> 但是仔细想想，没有做配置意味着什么？意味着**配置已经做好了**，不用你自己写了。但是新的问题又来了，如果不想用已经写好的默认配置，该如何干预呢？

## 属性配置

SpringBoot 通过配置文件 **application.properties** 就可以修改默认的配置
- 书写规范是 key=value

SpringBoot 中**导入对应 starter 后，提供对应配置属性**

---
> SpringBoot 内置属性查询：[https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties)
> 
> 这里面罗列了能写的配置项

例如：
- 修改服务器端口：`server.port=80`
- 关闭运行日志图标：`spring.main.banner-mode=off`
- 设置运行日志的显示级别：`logging.level.root=debug`

> 这个配置项和什么有关。在 pom 中注释掉导入的 spring-boot-starter-web，然后刷新工程，会发现配置的提示消失了。
> 
> 服务器端口和 web 的相关坐标有关，日志应该是大家通用的，到底和什么坐标有关呢？

所有的 starter 中都会依赖 spring-boot-starter。这个 starter 是所有的 SpringBoot 的 starter 的基础依赖，里面定义了 SpringBoot 相关的基础配置，

> 关于这个 starter 应用篇和原理篇中再深入理解。

```xml
 <dependency>  
     <groupId>org.springframework.boot</groupId>  
     <artifactId>spring-boot-starter</artifactId>  
     <version>2.5.4</version>  
     <scope>compile</scope>  
 </dependency>
```

## 配置文件分类

> properties 格式的配置写起来总是觉得看着不舒服，所以就期望存在一种书写起来更简便的配置格式提供给开发者使用

SpringBoot 提供了 3 种配置文件的格式
- properties（传统格式/默认格式）
- **yml**（主流格式）
- yaml

配置文件间的加载优先级 properties（最高）> yml > yaml（最低）

不同配置文件中**相同配置**按照加载优先级**相互覆盖**，不同配置文件中**不同配置全部保留**

---
application.properties

```properties
server.port=80
```

application.yml（yml格式）

```YML
server:
  port: 81
```

application.yaml（yaml格式）

```yaml
server:
  port: 82
```

> yml 格式和 yaml 格式除了文件名后缀不一样，格式完全一样，所以可以合并成一种格式来看

## yaml 

YAML（YAML Ain't Markup Language），一种数据序列化格式。
- 优点：
	- 容易阅读、
	- 容易与脚本语言交互
	- 以数据为核心
	- 重数据轻格式
- 常见的文件扩展名有两种：
	- `.yml` 格式（主流）
	- `.yaml` 格式

> 书写格式，自己查吧

```yml
subject:
	- Java
	- 前端
	- 大数据
enterprise:
	name: itcast
    age: 16
    subject:
    	- Java
        - 前端
        - 大数据
likes: [王者荣耀,刺激战场]			#数组书写缩略格式
users:							 #对象数组格式一
  - name: Tom
   	age: 4
  - name: Jerry
    age: 5
users:							 #对象数组格式二
  -  
    name: Tom
    age: 4
  -   
    name: Jerry
    age: 5			    
users2: [ { name:Tom , age:4 } , { name:Jerry , age:5 } ]	#对象数组缩略格式
```

## 读取配置数据

### 读取单一数据

`@Value` 可以读取单个数据，属性名引用方式：`${一级属性名.二级属性名……}`

![](assets/Pasted%20image%2020240220183820.png)

### 读取全部属性

> 读取单一数据可以解决读取数据的问题，但是如果定义的数据量过大，这么一个一个书写肯定会累死人的

SpringBoot 提供了一个 Environment 对象，使用**自动装配注解**可以将所有的 yaml 数据封装到这个对象中

![](assets/Pasted%20image%2020240220204903.png)

### 读取对象数据

> 单一数据读取书写比较繁琐，全数据封装又封装的太厉害了，每次拿数据还要一个一个的 `getProperties()`。由于 Java 是一个面向对象的语言，很多情况下，我们会将一组数据封装成一个对象。SpringBoot 也提供了可以将一组 yaml 对象数据封装一个 Java 对象的操作

- 封装类定义为 Spring 管理的 bean
- 使用注解 `@ConfigurationProperties` 绑定配置信息到封装类中
	- prefix：加载的数据前缀
	- 数据属性名要与对象的变量名一一对应

> `org.springframework.boot.context.properties.ConfigurationProperties`，`@ConfigurationProperties` 是 Boot 的注解

![](assets/Pasted%20image%2020240220213110.png)

# --------- 整合 SSMP

## 整合 Junit

不使用 SpringBoot 技术时，Spring 整合 JUnit 的制作方式

> 前置步骤省略

```java
//加载spring整合junit专用的类运行器
@RunWith(SpringJUnit4ClassRunner.class)
//指定对应的配置信息
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTestCase {
    //注入你要测试的对象
    @Autowired
    private AccountService accountService;
    @Test
    public void testGetById(){
        //执行要测试的对象对应的方法
        System.out.println(accountService.findById(2));
    }
}
```

---

Springboot 整合 Junit 步骤
1. 导入测试对应的 starter
2. 测试类使用 `@SpringBootTest` 修饰
	1. 测试类如果存在于引导类所在包或子包中无需指定引导类
	2. 否则需要**通过 classes 属性指定引导类**
3. 使用**自动装配**的形式添加要测试的对象

> `@SpringBootTest` 找其所在包和父包中有没有 `@SpringBootConfiguration`

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

```java
//@SpringBootTest(classes = {Main.class})
@SpringBootTest()
public class BookDaoTest {
    @Autowired
    private BookDao bookDao;

    @Test
    public void testSave(){
        bookDao.save();
    }
}
```

## 整合 MyBatis

1. 导入 MyBatis 对应的 starter，以及数据库驱动
2. 数据库连接相关信息转换成配置
3. 数据库 SQL 映射需要添加 `@Mapper` 被容器识别到

```xml
<dependencies>
    <!--1.导入对应的starter-->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.0</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

```yml
# 配置数据源
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/spring_db?serverTimezone=UTC
    username: root
    password: root
```

>  MySQL 8.X 驱动强制要求设置时区
> - 修改 url，添加 serverTimezone 设定
> 
> 驱动类过时，提醒更换为 `com.mysql.cj.jdbc.Driver`

```java
public class Book {
    private Integer id;
    private String type;
    private String name;
    private String description;
}

@Mapper
public interface BookDao {
    @Select("select * from tbl_book where id = #{id}")
    public Book getById(Integer id);
}
```

```java
@SpringBootTest
class Springboot05MybatisApplicationTests {
    @Autowired
    private BookDao bookDao;
    
    @Test
    void contextLoads() {
        System.out.println(bookDao.getById(1));
    }
}
```

## 整合 MP

1. 添加 MyBatis-Plus 对应的 starter
2. 数据层接口使用 BaseMapper 简化开发
3. 配置通用前缀名

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
```

> 配置数据源、实体类

```java
@Mapper
public interface BookDao extends BaseMapper<Book> {
}
```

## 整合 Druid

在没有指定数据源时，SpringBoot 帮我们选了一个它认为最好的数据源对象 HiKari，通过启动日志可以查看到对应的身影

```java
2024-02-21 16:14:18.856  INFO 2400 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2024-02-21 16:14:19.967  INFO 2400 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
[Book(id=1, type=小说, name=活着, description=xxx), Book(id=2, type=小说, name=在细雨中呼喊, description=xxx)]
2024-02-21 16:14:20.026  INFO 2400 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2024-02-21 16:14:20.028  INFO 2400 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
```

---
更换 Druid 数据源
1. 导入 Druid 对应的 starter
2. 根据 Druid 提供的配置方式进行配置

```xml
<dependencies>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid-spring-boot-starter</artifactId>
        <version>1.2.6</version>
    </dependency>
</dependencies>
```

修改配置两种方式

```java
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
```

> 目前的数据源配置格式是一个通用格式，不管你换什么数据源都可以用这种形式进行配置。
> 
> 但是如果对数据源进行个性化的配置，例如配置数据源对应的连接数量，这个时候就有新的问题了。每个数据源技术对应的配置名称都一样吗？肯定不是，各个厂商不可能提前商量好都写一样的名字啊，怎么办？就要使用专用的配置格式了。

```yml
spring:
  datasource:
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/ssm_db?serverTimezone=UTC
      username: root
      password: root
```

查看日志

```java
2024-02-21 16:53:03.311  INFO 21732 --- [           main] c.a.d.s.b.a.DruidDataSourceAutoConfigure : Init DruidDataSource
2024-02-21 16:53:03.523  INFO 21732 --- [           main] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} inited
 _ _   |_  _ _|_. ___ _ |    _ 
| | |\/|_)(_| | |_\  |_)||_|_\ 
     /               |         
                        3.4.3 
2024-02-21 16:53:04.578  INFO 21732 --- [           main] com.boer.dao.BookDaoTest                 : Started BookDaoTest in 2.052 seconds (JVM running for 3.136)
[Book(id=1, type=小说, name=活着, description=xxx), Book(id=2, type=小说, name=在细雨中呼喊, description=xxx)]
2024-02-21 16:53:05.867  INFO 21732 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closing ...
2024-02-21 16:53:05.870  INFO 21732 --- [ionShutdownHook] com.alibaba.druid.pool.DruidDataSource   : {dataSource-1} closed
```

## 综合案例

> 跳过

# -------------------- 运维实用篇

# ---------- SpringBoot 程序的打包与运行

## 打包与运行

打包
- 插件
- `mvn package`，本操作可以在 Idea 环境下执行
- 排除测试
- 打包后会产生一个与工程名类似的 jar 文件，其名称是由`模块名+版本号+.jar` 组成的

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

运行
- `java -jar <工程包名>.jar
`
## maven 打包插件与 jar 包结构

如果没有配置插件会报错：`xxx.jar` 中没有主清单属性

配置插件前后的 jar 包有3处比较明显的不同
- 打包后文件的大小不同
	- 带有配置的程序包体积比不带配置的大了 30 倍
	- 不仅将项目中自己开发的内容进行了打包，还把当前工程运行需要使用的 jar 包全部打包进来了，就是为了可以独立运行
- 打包后所包含的内容不同
- 打包程序中个别文件内容不同

> #todo 并没有研究清楚 具体的打包结构

%% 手动换行 %%
## linux 系统操作

> #todo 

%% 手动换行 %%
# ---------- 配置高级

SpringBoot 提供了“配置文件”和“临时属性”的方式来对程序进行配置。

## 临时属性

### 基本使用

> 目前我们的程序包打好了，可以发布了。但是程序包打好以后，里面的配置都已经是固定的了，比如配置了服务器的端口是 8080。如果我要启动项目，发现当前我的服务器上已经有应用启动起来并且占用了 8080 端口，这个时候就尴尬了。难道要重新把打包好的程序修改一下吗？比如我要把打包好的程序启动端口改成 80。

使用 jar 命令启动 SpringBoot 工程时可以使用临时属性**替换配置文件中的属性**    
- 临时属性添加方式：`java –jar [工程名].jar –-[属性名]=值`
	- 多个临时属性之间使用**空格**分隔
- 临时属性必须是当前 boot 工程支持的属性，否则设置无效

例：

```java
java –jar springboot.jar –-server.port=80 --logging.level.root=debug
```

### 开发环境中使用临时属性

idea 中的临时属性配置

![](assets/Pasted%20image%2020240222001755.png)

在程序中引导类的 main 方法的 args 参数能够获取到配置的临时属性
- 可以修改 args 参数
- 支持**断开外部传递临时属性的入口**，即 `run()` 方法不带 args 参数

```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        System.out.println(Arrays.toString(args));
        SpringApplication.run(Main.class);
    }
}
```

## 属性加载优先级

> 官方文档：[https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html)

![](assets/Pasted%20image%2020240222000026.png)

- 从上到下优先级增高
- 第 3 条 Config data 说的就是使用配置文件，第 11 条 Command line arguments 说的就是使用命令行临时参数

## 配置文件分类

配置文件分为 4 种，优先级由低到高
- **类路径**配置文件：服务于开发人员本机开发与测试
- **类路径 config 目录**中配置文件：服务于项目经理整体调控
- **工程路径**配置文件：服务于运维人员配置涉密线上环境
- **工程路径 config 目录**中配置文件：服务于运维经理整体调控

即：
1. file ：config/application.yml **【最高】**
2. file ：application.yml
3. classpath：config/application.yml
4. classpath：application.yml **【最低】**

级别 1 和 2 用于程序打包以后，无论程序里面配置写的是什么，都可以覆盖

多层级配置文件间的属性采用**叠加并覆盖**的形式作用于程序

> 那为什么设计这种多种呢？说一个最典型的应用吧。
> 
> - 场景 A：你作为一个开发者，你做程序的时候为了方便自己写代码，配置的数据库肯定是连接你自己本机的，咱们使用 4 这个级别，也就是之前一直用的 application.yml。
>     
> - 场景 B：现在项目开发到了一个阶段，要联调测试了，连接的数据库是测试服务器的数据库，肯定要换一组配置吧。你可以选择把你之前的文件中的内容都改了，目前还不麻烦。
>     
> - 场景 C：测试完了，一切 OK。你继续写你的代码，你发现你原来写的配置文件被改成测试服务器的内容了，你要再改回来。现在明白了不？场景 B 中把你的内容都改掉了，你现在要重新改回来，以后呢？改来改去吗？

## 自定义配置文件

临时属性配置
1. 配置文件可以修改**名称**
2. 配置文件可以修改**路径**

```java
--spring.config.name=<无后缀名称>
--spring.config.location=classpath:<类路径下目录>
```

可以设置加载多个配置文件

```java
--spring.config.location=classpath:/myconfig.yml,classpath:/myconfig2.yml
```

> 我们现在研究的都是 SpringBoot 单体项目，就是单服务器版本。其实企业开发现在更多的是使用基于 SpringCloud 技术的多服务器项目。这种配置方式和我们现在学习的完全不一样，所有的服务器将**不再设置自己的配置文件，而是通过配置中心获取配置**，动态加载配置信息。

# ---------- 多环境开发

## 单文件版

使用 `---` 区分环境设置边界

```yml
spring:
	profiles:
		active: pro		# 启动pro
---
spring:
	profiles: pro
server:
	port: 80
---
spring:
	profiles: dev
server:
	port: 81
---
spring:
	profiles: test
server:
	port: 82
```

上述格式是过时格式，标准格式如下

```yml
spring:
	config:
    	activate:
        	on-profile: pro
```

## 多文件版

主配置文件 application.yml

```yml
 spring:  
     profiles:  
         active: pro # 启动pro
```

使用文件名区分环境配置文件，文件的命名规则为：`application-环境名.yml`

application-pro.yaml

```yml
 server:  
     port: 80
```

application-dev.yaml

```yml
 server:  
     port: 81
```

配置规则：
- 主配置文件中设置**公共**配置（全局）
- 环境分类配置文件中常用于设置**冲突属性**（局部）

> properties 格式同理，且多环境配置仅支持多文件格式

## 分组管理

> 作为程序员在搞配置的时候往往处于一种分久必合合久必分的局面。开始先写一起，后来为了方便维护就拆分

SpringBoot 从 2.4 版开始使用 **group 属性**替代 include 属性，降低了配置书写量。

> 准备好devDB.yml，proDB.yml，testDB.yml

```yml
spring:
	profiles:
    	active: dev
        group:
        	"dev": devDB,devRedis,devMVC
      		"pro": proDB,proRedis,proMVC
      		"test": testDB,testRedis,testMVC
```

当主环境 dev 与其他环境有相同属性时，主环境属性生效；其他环境中有相同属性时，最后加载的环境属性生效

## maven 设置开发环境

> 就是 maven 和 SpringBoot 同时设置多环境的话怎么办？
> 
> maven 是做什么的？项目构建管理的，最终生成代码包的，SpringBoot 是干什么的？简化开发的。简化，又不是其主导作用。最终还是要靠 maven 来管理整个工程，所以 SpringBoot 应该听 maven 的。

1. 当 Maven 与 SpringBoot 同时对多环境进行控制时，以 Maven 为主
2. 基于 SpringBoot 读取 Maven 配置属性的前提下，如果在 Idea 下测试工程时 pom.xml 每次更新需要手动 **compile** 方可生效

---
maven 中设置多环境

```xml
<profiles>
    <profile>
        <id>env_dev</id>
        <properties>
            <profile.active>dev</profile.active>
        </properties>
        <activation>
	        <!--默认启动环境-->
            <activeByDefault>true</activeByDefault>		
        </activation>
    </profile>
    <profile>
        <id>env_pro</id>
        <properties>
            <profile.active>pro</profile.active>
        </properties>
    </profile>
</profiles>
```

SpringBoot 中读取 maven 设置值
- `@..@` 占位符

```yml
spring:
	profiles:
    	active: @profile.active@
```

# ---------- 日志

> 日志其实就是记录程序日常运行的信息，主要作用如下：
> 
> - 编程期调试代码
> - 运营期记录信息
> - 记录日常运营重要信息（峰值流量、平均响应时长……）
> - 记录应用报错信息（错误堆栈）
> - 记录运维过程数据（扩容、宕机、报警……）
> 
> ​或许各位小伙伴并不习惯于使用日志，没关系，慢慢多用，习惯就好。想进大厂，这是最基本的，别去面试的时候说没用过，完了，没机会了。

## 基本使用

### 添加日志记录

- 获取日志对象
- 指定日志级别添加日志

```java
@RestController
@RequestMapping("/books")
public class BookController extends BaseClass{
    private static final Logger log = LoggerFactory.getLogger(BookController.class);
    
    @GetMapping
    public String getById(){
        log.debug("debug...");
        log.info("info...");
        log.warn("warn...");
        log.error("error...");
        return "springboot is running...2";
    }
}
```

每个类都要写创建日志记录对象，这个可以优化一下

1）第一种优化方式：自定义 BaseClass，继承

```java
public class BaseClass {
    private Class clazz;
    public Logger log;

    public BaseClass() {
        clazz = this.getClass();
        log = LoggerFactory.getLogger(clazz);
    }

}

@RestController
@RequestMapping("/books")
public class BookController extends BaseClass{
    @Autowired
    private BookDao bookDao;

    @GetMapping
    public List<Book> getAll() {
        log.debug("debug...");
        log.info("info...");
        log.warn("warn...");
        log.error("error...");
        return bookDao.selectList(null);
    }
}
```

2）第二种方式：基于 lombok 提供的 `@Slf4j` 注解

```java
@Slf4j
@RestController
@RequestMapping("/books")
public class BookController{

    @Autowired
    private BookDao bookDao;

    @GetMapping
    public List<Book> getAll() {
        log.debug("debug...");
        log.info("info...");
        log.warn("warn...");
        log.error("error...");
        return bookDao.selectList(null);
    }
}
```

### 设置日志输出级别

日志的级别分为 6 种，分别是：
- TRACE：运行堆栈信息，使用率低
- **DEBUG**：程序员调试代码使用
- **INFO**：记录运维过程数据
- **WARN**：记录运维过程报警数据
- **ERROR**：记录错误堆栈信息
- FATAL：灾难信息，合并计入 ERROR

由低到高，日志输出 `>=` 当前级别

> 一般情况下，开发时候使用 DEBUG，上线后使用 INFO，运维信息记录使用 WARN 即可。

下面的方式更细粒度

```yml
# 开启debug模式，输出调试信息，常用于检查系统运行状况
debug: true

# 设置日志级别，root表示根节点，即整体应用日志级别
logging:
	level:
    	root: debug
```

可以通过**日志组**或**代码包**的形式进行日志显示级别的控制

```yml
logging:
	# 设置日志组
    group:
    	# 自定义组名，设置当前组中所包含的包
        ebank: com.itheima.controller
        iservice: com.alibaba
    level:
    	root: warn
        # 为对应组设置日志级别
        ebank: debug
    	# 为对包设置日志级别
        com.itheima.controller: debug
```

### 日志输出格式控制

![](assets/Pasted%20image%2020240222164746.png)

官方模板

```yml
logging:
  pattern:
    console: "%d %clr(%p) --- [%16t] %clr(%-40.40c){cyan} : %m %n"
```

> #todo 

### 日志文件

#todo 

# -------------------- 开发实用篇

# ---------- 热部署

#todo 

# ---------- 绑定属性

## 第三方 bean 绑定属性

> 第三方开发的 bean 源代码不是你自己书写的，也不可能到源代码中去添加`@ConfigurationProperties` 注解

第一种方式：`@Bean` + `@ConfigurationProperties`
- `@ConfigurationProperties` 可以添加在类或者方法上
	- 添加到类上是为 spring 容器管理的当前类的对象绑定属性，
	- 添加到方法上是为 spring 容器管理的当前方法的返回值对象绑定属性

```yml
mydatasource:
  driver-class-name: com.mysql.cj.jdbc.Driver
  url: jdbc:mysql://localhost:3306/spring_db?serverTimezone=UTC
  username: root
  password: root
```

```java
@SpringBootApplication
public class Main {

    @Bean
    @ConfigurationProperties("mydatasource")
    public DataSource dataSource(){
        DruidDataSource ds = new DruidDataSource();
        return ds;
    }

    public static void main(String[] args) {
        ConfigurableApplicationContext ct = SpringApplication.run(Main.class);
        DataSource dataSource = ct.getBean(DataSource.class);
        System.out.println(dataSource);
    }
}
```

## @EnableConfigurationProperties

> 因为 `@ConfigurationProperties` 不仅可以写在类上，还可以写在方法上，所以找起来就比较麻烦了。
> 
> 为了解决这个问题，spring 给我们提供了一个全新的注解，**专门标注使用 `@ConfigurationProperties` 注解绑定属性的 bean 是哪些**。这个注解叫做 `@EnableConfigurationProperties`

`@EnableConfigurationProperties` + `@ConfigurationProperties`
- 步骤：
	- 在配置类上添加 `@EnableConfigurationProperties` ，并标注要使用 `@ConfigurationProperties` 绑定属性的类
	- 在**对应的类**上直接使用 `@ConfigurationProperties` 进行属性绑定
- 使用 `@EnableConfigurationProperties` 之后，**不需要 `@Component` 了**，否则会导致生成两个 bean，容器不知道装配哪个

```yml
servers:
  ip-address: 192.168.0.1 
  port: 2345
  timeout: -1
```

```java
@Data
@ConfigurationProperties(prefix = "servers")
public class ServerConfig {
    private String ipAddress;
    private int port;
    private long timeout;
}
```

```java
@SpringBootApplication
@EnableConfigurationProperties(ServerConfig.class)
public class Springboot13ConfigurationApplication {
}
```

## 宽松绑定

> springboot 进行编程时人性化设计的一种体现，即配置文件中的命名格式与变量名的命名格式可以进行格式上的最大化兼容。

下面的配置属性名规则全兼容，最终都可以匹配到 ipAddress 这个属性名
- 在进行匹配时，配置中的名称 和 java 代码中的属性名 要**去掉中划线和下划线** 并 **忽略大小写** 的情况下去进行等值匹配，即都是 ipaddress
- springboot 官方推荐使用烤肉串模式

```yml
servers:
  ipAddress: 192.168.0.2       # 驼峰模式
  ip_address: 192.168.0.2      # 下划线模式
  ip-address: 192.168.0.2      # 烤肉串模式
  IP_ADDRESS: 192.168.0.2      # 常量模式
```

- `@ConfigurationProperties` 绑定属性时支持**属性名宽松绑定**，这个宽松体现在属性名的命名规则上
	- 绑定**前缀名**推荐采用**烤肉串命名规则**（仅能使用纯小写字母、数字、下划线作为合法的字符）
- `@Value` 注解**不支持**松散绑定规则

## 常用计量单位

> 在前面的配置中 `servers.timeout=-1`，超时时间 timeout 描述了服务器操作超时时间，-1 表示永不超时。
> 
> 但是每个人都这个值的理解会产生不同，比如线上服务器完成一次主从备份，配置超时时间 240，这个 240 如果单位是秒就是超时时间 4 分钟，如果单位是分钟就是超时时间 4 小时。面对一次线上服务器的主从备份，设置4分钟，简直是开玩笑，别说拷贝过程，备份之前的压缩过程4分钟也搞不定，这个时候问题就来了，怎么解决这个误会？

SpringBoot 支持 JDK8 提供的时间与空间计量单位 Duration 和 DataSize
- 通过 `@DurationUnit` 注解描述时间单位
- 通过 `@DataSizeUnit` 注解描述存储空间单位

```yml
servers:
	timeout: -1
	data-size: 10
```

```java
@Component
@Data
@ConfigurationProperties(prefix = "servers")
public class ServerConfig {
    @DurationUnit(ChronoUnit.HOURS)
    private Duration serverTimeOut;
    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize dataSize;
}
```

## bean 属性校验

> #Boer 感觉不是 boot 范畴的，boot 只是设定了 version，且 validation-api 这个坐标不导入也能使用

SpringBoot 给出了强大的数据校验功能，可以有效的避免此类问题的发生。

在 JAVAEE 的 JSR303 规范中给出了具体的数据校验标准，开发者可以根据自己的需要选择对应的校验框架，

> 此处使用 Hibernate 提供的校验框架来作为实现进行数据校验

---
导入坐标

```xml
<!--1.导入JSR303规范-->
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
</dependency>
<!--使用hibernate框架提供的校验器做实现-->
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
</dependency>
```

实现 bean 属性校验功能
- `@Validated` 注解启用校验功能
- 对具体的字段设置校验规则

```java
@Component
@Data
@ConfigurationProperties(prefix = "servers")
// 开启对当前bean的属性注入校验
@Validated
public class ServerConfig {
    // 设置具体的规则
    @Max(value = 8888,message = "最大值不能超过8888")
    @Min(value = 202,message = "最小值不能低于202")
    private int port;
}
```

## 数据类型转换

#todo 

# ---------- 测试

`@SpringBootTest` 所有的属性

![](assets/Pasted%20image%2020240224164037.png)

## 加载测试专用属性

测试过程本身并不是一个复杂的过程，但是很多情况下测试时需要模拟一些线上情况，或者模拟一些特殊情况。如果当前环境按照线上环境已经设定好了，例如是下面的配置

```yml
 env:  
   maxMemory: 32GB  
   minMemory: 16GB
```

但是你现在想测试对应的兼容性，需要测试如下配置

```yml
 env:  
   maxMemory: 16GB  
   minMemory: 8GB
```

这个时候如果每次测试前改过来，每次测试后改回去，这太麻烦了。需要在测试环境中创建一组**测试专用属性**，去覆盖源码中设定的属性

---
应用范围仅适用于当前测试用例：`@SpringBootTest` 的 
- properties 属性添加配置属性，
- args 属性添加临时参数
- 两者共存，命令行参数的加载优先级高于配置属性

```yml
# 会被覆盖的属性
test:
  prop: testProp1
```

```java
@SpringBootTest(properties = {"test.prop=testProp2"},args = {"--test.args=testArgs2"})
public class PropertiesAndArgsTest {
    @Value("${test.prop}")
    private String prop;

    @Value("${test.args}")
    private String args;

    @Test
    public void testPropertiesAndArgs(){
        System.out.println(prop);
        System.out.println(args);
    }
}
```

## 加载测试专用 bean

> 一个 spring 环境中可以设置若干个配置文件或配置类，若干个配置信息可以同时生效。现在我们的需求就是在测试环境中再添加一个配置类，然后启动测试环境时，生效此配置就行了

步骤：
- 在测试包 test 中创建专用的测试环境配置类
- `@Import` 注解在具体的测试中导入临时的配置

```java
@Configuration
public class MsgConfig {
    @Bean
    public String msg(){
        return "bean msg";
    }
}
```

```java
@SpringBootTest
@Import({MsgConfig.class})
public class ConfigurationTest {

    @Autowired
    private String msg;

    @Test
    void testConfiguration(){
        System.out.println(msg);
    }
}
```

## Web 环境模拟测试

#todo 

# ---------- 数据层解决方案

#todo 

# ---------- 整合第三方技术

#todo 

# ---------- 监控

#todo 

# -------------------- 原理篇

# ---------- 自动配置

## bean 的加载控制

> 企业级开发中不可能在 spring 容器中进行 bean 的饱和式加载的。什么是饱和式加载，就是不管用不用，全部加载。比如 jdk 中有两万个类，那就加载两万个 bean，显然是不合理的，因为你压根就不会使用其中大部分的 bean。那合理的加载方式是什么？肯定是必要性加载，就是用什么加载什么。
> 
> 继续思考，加载哪些 bean 通常受什么影响呢？最容易想的就是你要用什么技术，就加载对应的 bean。用什么技术意味着什么？就是加载对应技术的类。所以在 spring 容器中，通过判定是否加载了某个类来控制某些 bean 的加载是一种常见操作。
> 
> 下例给出了对应的代码实现，其实思想很简单，先判断一个类的全路径名是否能够成功加载，加载成功说明有这个类，那就干某项具体的工作，否则就干别的工作。

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        try {
            Class<?> clazz = Class.forName("com.itheima.bean.Mouse");
            if(clazz != null) {
                return new String[]{"com.itheima.bean.Cat"};
            }
        } catch (ClassNotFoundException e) {
//            e.printStackTrace();
            return new String[0];
        }
        return null;
    }
}
```

---
springboot 定义了若干种控制 bean 加载的条件设置注解，由 spring 固定加载 bean 变成了可以根据情况**选择性的加载 bean**
- `@Condition` 的衍生注解

`@ConditionalOnXxx`
- 可以加在 **`@Component` 类上** 或者 **`@Bean` 方法上**
- 可以做**并且的逻辑关系**，
	- 写 2 个就是 2 个条件都成立，写多个就是多个条件都成立。
- `@ConditionalOnClass` 实现了当虚拟机中加载了某个类时加载对应的 bean。
	- `@ConditionalOnMissingClass` 反之
- `@ConditionalOnWebApplication` 判定当前容器环境是否是web环境。
	- `@ConditionalOnNotWebApplication`反之
- `@ConditionalOnBean` 判定当前容器环境是否包含某个 bean
- 等等

例：判定当前是否加载了 mysql 的驱动类，如果加载了，就注册搞一个 Druid 的 bean

```java
public class SpringConfig {
    @Bean
    @ConditionalOnClass(name="com.mysql.jdbc.Driver")
    public DruidDataSource dataSource(){
        return new DruidDataSource();
    }
}
```

## bean 的依赖属性配置管理

> bean 在运行的时候，实现对应的业务逻辑时有可能需要开发者提供一些设置值，有就是属性了。
> - 如果使用构造方法将参数固定，灵活性不足，
> - 这个时候就可以使用前期学习的 bean 的属性配置相关的知识进行灵活的配置了。先通过 yml 配置文件，设置 bean 运行需要使用的配置信息。

需求：
- 业务 bean 的属性可以为其设定默认值
- 当需要设置时通过配置文件传递属性
- 业务 bean 应尽量**避免设置强制加载**，而是根据需要导入后加载，降低 spring 容器管理 bean 的强度

---
配置文件中使用固定格式为属性类注入属性

```yml
cartoon:
  cat:
    name: "图多盖洛"
    age: 5
  mouse:
    name: "泰菲"
    age: 1
```

将业务功能 bean 运行需要的资源抽取成独立的属性类（`*Properties`），读取配置文件信息

```java
@ConfigurationProperties(prefix = "cartoon")
@Data
public class CartoonProperties {
    private Cat cat;
    private Mouse mouse;
}
```

定义业务功能 bean
- 通常使用 `@Import` 导入，解耦强制加载 bean
- `@EnableConfigurationProperties` 设定**使用属性类时加载 bean**
	- 只用默认值的话就不需要加载属性类的 bean

```java
@Data
@EnableConfigurationProperties(CartoonProperties.class)
public class CartoonCatAndMouse implements ApplicationContextAware {
    private Cat cat;
    private Mouse mouse;

    private CartoonProperties cartoonProperties;

	// 构造器注入
    public CartoonCatAndMouse(CartoonProperties cartoonProperties){
        this.cartoonProperties = cartoonProperties;
        cat = new Cat();
        // 配置了就注入配置的属性值，否则就用默认值
        cat.setName(cartoonProperties.getCat()!=null && StringUtils.hasText(cartoonProperties.getCat().getName()) ? cartoonProperties.getCat().getName() : "tom");
        cat.setAge(cartoonProperties.getCat()!=null && cartoonProperties.getCat().getAge()!=null ? cartoonProperties.getCat().getAge() : 3);
        mouse = new Mouse();
        mouse.setName(cartoonProperties.getMouse()!=null && StringUtils.hasText(cartoonProperties.getMouse().getName()) ? cartoonProperties.getMouse().getName() : "jerry");
        mouse.setAge(cartoonProperties.getMouse()!=null && cartoonProperties.getMouse().getAge()!=null ? cartoonProperties.getMouse().getAge() : 4);
    }

    public void play(){
        String[] beans = applicationContext.getBeanDefinitionNames();
        for (String bean : beans) {
            System.out.println(bean);
        }
        System.out.println(cat.getAge()+"岁的"+cat.getName()+"和"+mouse.getAge()+"岁的"+mouse.getName()+"打起来了");
    }

}
```

```java
@Import(CartoonCatAndMouse.class)
@Configuration
public class App {}
```

## 自动配置原理

思想：
 - 准备阶段
	- 收集 Spring 开发者的编程习惯，整理开发过程经常使用的技术列表 ---> **技术集 A**
	- 收集常用技术 (**技术集 A**) 的使用参数，整理开发过程中每个技术的常用设置列表 ---> **设置集 B**
- 加载阶段
	- springboot 初始化 Spring 容器基础环境，形成**初始化环境**
		- 读取用户的配置信息
		- 加载用户自定义的 bean 
		- 导入的其他坐标
	- springboot 将**技术集 A**包含的所有技术在 SpringBoot 启动时默认全部加载，
		- 这时肯定加载的东西有一些是无效的，没有用的
	- springboot 会对**技术集 A**中具有使用条件的技术约定出来，设置成按条件加载，由开发者决定是否使用该技术
		- 与初始化环境对比
	- 将**设置集 B** 作为**默认配置加载**(约定大于配置)，减少开发者配置工作量
	- 开放**设置集 B**的配置覆盖接口，由开发者根据自身需要决定是否覆盖默认配置


原理：
- springboot 启动时先加载 `spring.factories` 文件（`spring-boot-autoconfigure-2.5.4.jar` 包下）中的 `org.springframework.boot.autoconfigure.EnableAutoConfiguration` 配置项，将其中配置的所有的类都加载成 bean
	- 在加载 bean 的时候，bean 对应的类定义上都设置有加载条件（`@ConditionalOnXxx`），因此有可能加载成功，也可能条件检测失败不加载 bean
- 对于可以正常加载成 bean 的类，通常会通过 `@EnableConfigurationProperties` 注解初始化对应的配置属性类并加载对应的配置
	- 配置属性类上通常会通过 `@ConfigurationProperties` 加载指定前缀的配置，当然这些配置通常都有默认值。如果没有默认值，就强制你必须配置后使用了

如何让 springboot 启动的时候去加载自定义的类呢？
- springboot 为我们开放了一个配置入口，在配置目录中**创建 META-INF 目录**，并**创建 spring.factories 文件**，在其中**添加设置**，说明哪些类要启动自动配置就可以了

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.itheima.bean.CartoonCatAndMouse
```

其实这个文件就做了一件事，通过这种配置的方式加载了指定的类。转了一圈，就是个普通的 bean 的加载，和最初使用 xml 格式加载 bean 几乎没有区别，格式变了而已。

自动配置的核心究竟是什么呢？自动配置其实是一个小的生态，可以按照如下思想理解：

1. 自动配置从根本上来说就是**一个 bean 的加载**
2. 通过 bean 加载条件的控制给开发者一种感觉，自动配置是自适应的，可以根据情况自己判定，但实际上就是最**普通的分支语句的应用**，这是蒙蔽我们双眼的第一层面纱
3. 使用 bean 的时候，如果不设置属性，就有默认值，如果不想用默认值，就可以自己设置，也就是可以修改部分或者全部参数，感觉这个过程好屌，也是一种自适应的形式，其实还是需要使用**分支语句**来做判断的，这是蒙蔽我们双眼的第二层面纱
4. springboot 技术提前将大量开发者有可能使用的技术提前做好了，条件也写好了，用的时候你**导入了一个坐标，对应技术就可以使用了**，其实就是**提前帮我们把 `spring.factories` 文件写好了**，这是蒙蔽我们双眼的第三层面纱

> 现在 springboot 程序启动时，在后台偷偷的做了这么多次检测，这么多种情况判定，不用问了，效率一定是非常低的，毕竟它要检测 100 余种技术是否在你程序中使用。

## 变更自动配置

启用自动配置只需要满足自动配置条件即可（导入坐标），可以根据需求开发自定义自动配置项

1）通过配置文件 exclude 属性排除指定的自动配置类

```yml
spring:
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration
```

2）通过注解 `@EnableAutoConfiguration` 属性排除自动配置项
- `@SpringBootApplication` 继承了这个属性

```java
@SpringBootApplication(excludeName = "org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration")
```

3）排除坐标（应用面较窄）

如果**当前自动配置中包含有更多的自动配置功能**，也就是一个套娃的效果。此时可以通过检测条件的控制来管理自动配置是否启动。

例如 **web 程序启动时会自动启动 tomcat 服务器**，可以通过排除坐标的方式，让加载 tomcat 服务器的条件失效。不过需要提醒一点，你把 tomcat 排除掉，记得再加一种可以运行的服务器。

```xml
 <dependencies>  
     <dependency>  
         <groupId>org.springframework.boot</groupId>  
         <artifactId>spring-boot-starter-web</artifactId>  
         <!--web起步依赖环境中，排除Tomcat起步依赖，匹配自动配置条件-->  
         <exclusions>  
             <exclusion>  
                 <groupId>org.springframework.boot</groupId>  
                 <artifactId>spring-boot-starter-tomcat</artifactId>  
             </exclusion>  
         </exclusions>  
     </dependency>  
     <!--添加Jetty起步依赖，匹配自动配置条件-->  
     <dependency>  
         <groupId>org.springframework.boot</groupId>  
         <artifactId>spring-boot-starter-jetty</artifactId>  
     </dependency>  
 </dependencies
```

# ---------- 自定义 starter 开发


