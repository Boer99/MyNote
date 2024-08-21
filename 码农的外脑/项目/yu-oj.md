
# 项目设计

## 核心业务流程

![700](assets/Pasted%20image%2020240401003446.png)

![](assets/Pasted%20image%2020240401003823.png)

## 系统功能

- 公共模块
	- common 公共模块 (yuoj-backend-common)：全局异常处理器、请求响应封装类、公用的工具类等
	- model 模型模块 (yuoj-backend-model)：很多服务公用的实体类
	- 公用接口模块 (yuoj-backend-service-client)：只存放接口，不存放实现(多个服务之间要共享)

- 用户模块
	- 注册
	- 登录
	- 用户管理

- 题目模块
	- 创建题目（管理员）
	- 删除题目（管理员）
	- 修改题目（管理员）
	- 搜索题目（用户）
	- 在线做题（题目详情页）
	- 题目提交

- 判题模块
	- 执行判题逻辑
	- 错误处理（内存溢出、安全性、超时）
	- 自主实现 代码沙箱（安全沙箱）
	- 开放接口（提供一个独立的新服务）



# 前端

## 前端组件库

[Arco Design - 企业级产品的完整设计和开发解决方案](https://arco.design/)

## Markdown 编辑器

Markdown 编辑器组件：[bytedance/bytemd: ByteMD v1 repository (github.com)](https://github.com/bytedance/bytemd)

## 代码编辑器

[microsoft/monaco-editor: A browser based code editor (github.com)](https://github.com/microsoft/monaco-editor)



# 后端

## 初始化项目

名字替换

- 项目文件名 yuoj-backend
- 全局替换：springboot-init --> yuoj-backend，springbootinit-->yuoj
- 包名替换

## 库表设计

### 用户表

```sql
-- 用户表  
create table if not exists user  
(  
    id           bigint auto_increment comment 'id' primary key,  
    userAccount  varchar(256)                           not null comment '账号',  
    userPassword varchar(512)                           not null comment '密码',  
    unionId      varchar(256)                           null comment '微信开放平台id',  
    mpOpenId     varchar(256)                           null comment '公众号openId',  
    userName     varchar(256)                           null comment '用户昵称',  
    userAvatar   varchar(1024)                          null comment '用户头像',  
    userProfile  varchar(512)                           null comment '用户简介',  
    userRole     varchar(256) default 'user'            not null comment '用户角色：user/admin/ban',  
    createTime   datetime     default CURRENT_TIMESTAMP not null comment '创建时间',  
    updateTime   datetime     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',  
    isDelete     tinyint      default 0                 not null comment '是否删除',  
    index idx_unionId (unionId)  
) comment '用户' collate = utf8mb4_unicode_ci;
```

*userRole*：用户角色（只有管理员才能发布和管理题目，普通用户只能看题）

### 题目表

```sql
-- 题目表  
create table if not exists question  
(  
    -- 通用字段  
    id           bigint auto_increment comment 'id' primary key,  
    title        varchar(512)                       null comment '标题',  
    tags         varchar(1024)                      null comment '标签列表（json 数组）',  
    thumbNum     int      default 0                 not null comment '点赞数',  
    favourNum    int      default 0                 not null comment '收藏数',  
    userId       bigint                             not null comment '创建用户 id',  
    createTime   datetime default CURRENT_TIMESTAMP not null comment '创建时间',  
    updateTime   datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',  
    isDelete     tinyint  default 0                 not null comment '是否删除',  
    -- 独有字段  
    content      text                               null comment '内容',  
    answer       text                               null comment '答案',  
    submitNum    int      default 0                 not null comment '提交数',  
    acceptedNum  int      default 0                 not null comment '通过数',  
    judgeCase    text                               null comment '判题用例（json数组）',  
    judgeConfig  text                               null comment '判题配置（json对象）',  
  
    index idx_userId (userId)  
) comment '题目' collate = utf8mb4_unicode_ci;
```

- *题目内容*：存放题目的介绍、输入输出提示、描述、具体的详情
- *题目答案*
- *题目标签*（json 数组字符串）：栈、队列、链表、简单、中等、困难
- *提交数、通过数*：便于分析统计（通过率、难度）

> 扩展字段：
> 
> - 通过率
> - 判题类型

---

> 如果说题目不是很复杂，用例文件大小不大的话，可以直接存在数据库表里
> 
> 但是如果用例文件比较大，>512 KB 建议单独存放在一个文件中，数据库中只保存文件 url（类似存储用户头像）

判题相关字段：

1）*judgeConfig*：判题配置（json 对象）

- 时间限制 timeLimit
- 内存限制 memoryLimit
- ...

存 json 对象的好处：便于扩展，只需要改变对象内部的字段，而不用修改数据库表字段（可能会影响数据库）

2）*judgeCase*：判题用例（json 数组）

- 每一个元素是：一个输入用例 对应 一个输出用例

> 存 json 的前提：
> 
> - 不需要根据某个字段去倒查这条数据
> - 字段含义相关，属于同一类的值
> - 字段存储空间占用不能太大

### 题目提交表

```sql
-- 题目提交表
create table if not exists question_submit
(
    -- 通用字段
    id           bigint auto_increment comment 'id' primary key,
    userId       bigint                             not null comment '创建用户 id',
    createTime   datetime default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime   datetime default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    isDelete     tinyint  default 0                 not null comment '是否删除',
    -- 独有字段
    questionId   bigint                             not null comment '题目 id',
    language     varchar(128)                       not null comment '编程语言',
    code         text                               not null comment '代码',
    status       int      default 0                 not null comment '判题状态（0-待判题、1-判题中、2-成功、3-失败）',
    judgeInfo    text                               null     comment '判题信息（json对象）',

    index idx_userId (userId),
    index idx_questionId (questionId)
) comment '题目提交表' collate = utf8mb4_unicode_ci;
```

字段：

- *questionld*
- *language*：编程语言
- *code*
- *status*：判题状态（待判题、判题中、成功、失败）
- *judgelnfo*（json 对象）：判题信息。比如程序的失败原因、程序执行消耗的时间、空间

索引：分别给 userId、questionId 单独创建索引

> **什么情况下适合加索引？如何选择给哪个字段加索引？**
> 
> 答：首先从业务出发，无论是单个索引、还是联合索引，都要从你实际的查询语句、字段枚举值的区分度、字段的类型考虑（where 条件指定的字段）
> 
> 比如：`where userld=1 and questionld =2` 
> 
> - 可以选择根据 userld 和 questionld 分别建立索引（需要分别根据这两个字段单独查询）;
> - 也可以选择给这两个字段建立联合索引(所查询的字段是绑定在一起的)。
> 
> 原则上：能不用索引就不用索引；能用单个索引就别用联合/多个索引；不要给没区分度的字段加索引（比如性别，就男/女），因为索引也是要占用空间的。


判题信息枚举值:

- Accepted 成功
- Wrong Answer 答案错误
- Compile Error 编译错误
- Memory Limit Exceeded 内存溢出
- Time Limit Exceeded 超时
- Presentation Error 展示错误
- Output Limit Exceeded 输出溢出
- Waiting 等待中
- Dangerous Operation 危险操作
- Runtime Error 运行错误(用户程序的问题)
- System Error 系统错误(做系统人的问题)

## 实体类的 json 字段设计

1）json 数组用 List 操作

```java
public class Question implements Serializable {  
    /**  
     * 标签列表（json 数组）  
     */  
    private String tags;
}

public class QuestionAddRequest implements Serializable {    
    /**  
     * 标签列表（json 数组）  
     */  
    private List<String> tags;
}

public class QuestionVO implements Serializable {
	/**  
	 * 标签列表（json 数组）  
	 */  
	private List<String> tags;
}
```

2）为了更方便地处理 json 字段中的某个字段，需要给对应的 json 字段编写独立的类，比如 judgeConfig、judgeInfo、judgeCase

```java
/**  
 * 判题用例（json数组）
 */  
public class JudgeCase {  
    /**  
     * 输入用例  
     */  
    private String input;  
  
    /**  
     * 输出用例  
     */  
    private String output;  
}

/**
 * 判题配置（json对象）
 */
public class JudgeConfig {
    /**
     * 时间限制(ms)
     */
    private Long timeLimit;

    /**
     * 内存限制(KB)
     */
    private Long memoryLimit;

    /**
     * 堆栈限制(KB)
     */
    private Long stackLimit;
}

/**  
 * 判题信息（json对象）
 */  
public class JudgeInfo {  
    /**  
     * 程序执行信息  
     */  
    private String message;  
  
    /**  
     * 消耗内存  
     */  
    private Long memory;  
  
    /**  
     * 消耗时间  
     */  
    private Long time;  
}
```

## 判题模块和代码沙箱

关系：

![500](assets/Pasted%20image%2020240401004249.png)

### 代码沙箱架构设计（工厂、代理）

1）定义代码沙箱的接口，提高通用性

项目代码只调用接口，不调用具体的实现类，这样在你使用其他的代码沙箱实现类时，就不用去修改名称了， 便于扩展。

> 代码沙箱的请求接口中，timeLimit 可加可不加，可自行扩展，即时中断程序

> 扩展思路：增加一个查看代码沙箱状态的接口

2）定义多种不同的代码沙箱实现

- 示例代码沙箱：仅为了跑通业务流程
- 远程代码沙箱：实际调用接口的沙箱
- 第三方代码沙箱：调用网上现成的代码沙箱， https://github.com/criyle/go-judge

3）使用工厂模式，根据用户传入的字符串参数（沙箱类别），来生成对应的代码沙箱实现类

此处使用**静态工厂模式**，实现比较简单，符合我们的需求。

4）`@Value` 配置文件中用户指定的代码沙箱

5）代码沙箱能力增强，**静态代理模式**

### 判题服务业务流程开发

判题服务业务流程：

1. 传入题目的提交 id，获取到对应的题目、提交信息（包含代码、编程语言等）
2. 如果题目提交状态不为等待中，就不用重复执行了
3. 更改判题（题目提交）的状态为“判题中”，防止重复执行
4. 调用沙箱，获取到执行结果
5. 根据沙箱的执行结果，设置题目的判题状态和信息

执行结果的判断逻辑：

1. 先判断沙箱执行的结果输出**数量**是否和预期输出数量相等
2. 依次判断**每一项**输出和预期输出是否相等
3. 判题题目的**限制**是否符合要求
4. 可能还有**其他**的异常情况

### 策略模式优化判题策略代码

判题策略：根据代码沙箱返回的执行结果 `List<String> outputList`，更新“题目提交”的 判题状态 Status 和 判题信息 JudgeInfo

可能会有很多种，比如：我们的代码沙箱本身执行程序需要消耗时间，这个时间可能不同的编程语言是不同的，比如沙箱执行 Java 要额外花 10 秒。

我们可以采用**策略模式**，针对不同的情况，定义独立的策略，而不是把所有的判题逻辑、if。。。else 代码全部混在一起写。

1）首先编写默认判题策略

2）如果**选择**某种判题策略的过程比较复杂，都写在调用判题服务的代码中，代码会越来越复杂，会有很多 if。。。else，所以呢，建议单独编写一个**选择策略**的类

## 代码沙箱实现（Java）

代码沙箱：只负责接受代码和输入，返回编译运行的结果，不负责判题(可以作为独立的项目/服务，提供给其他的需要执行代码的项目去使用)

> 以 **Java** 编程语言为主，带大家实现代码沙箱，重要的是学思想、学关键流程
> 
> 扩展：可以自行实现 C++ 语言的代码沙箱

### Java 原生代码沙箱实现

新建一个 Spring Boot Web 项目，最终这个项目要提供一个能够执行代码、操作代码沙箱的接口

原生：尽可能不借助第三方库和依赖，用最干净、最原始的方式实现代码沙箱

代码沙箱需要：接收代码 => 编译代码（javac）=> 执行代码（java）

编译代码：

```bash
javac -encoding utf-8 .\Main.java
```

执行：

```bash
java -cp 路径 Main 参数
```

> cp：classpath

实际 OJ 系统中，对用户输入的代码会有一定的要求，便于系统统一地处理。所以此处，我们把用户输入代码的==类名限制为 Main==（参考 Poj），可以减少类名不一致的风险，而且不用从用户代码中提取类名

#### 核心流程实现

核心实现思路：用程序代替人工，用程序来操作命令行，去编译执行代码 Java 

进程执行管理类：Process

1. 把用户的代码保存为文件
2. 编译代码，得到 class 文件
	1. 通过 Runtime 类执行 `javac` 命令，返回 Process 对象
	2. 通过 exitValue 判断程序是否正常返回
	3. 从 Process 的 inputStream 和 errorStream 获取**控制台输出**
3. 执行代码，得到输出结果
	1. 通过 Runtime 类执行 `java -cp` 命令，返回 Process 对象
4. 收集整理输出结果
	1. stopwatch 计时，最大运行时间取耗时最长的测试用例
6. 文件清理
7. 错误处理，提升程序健壮性

#### 异常情况演示

到目前为止，核心流程已经实现，但是想要上线的话，安全么?

用户提交恶意代码，怎么办?

- 1）执行阻塞，占用资源不释放（sleep，线程阻塞）
- 2）占用内存，不释放（堆内存溢出）
- 3）读取服务器文件（文件信息泄露）
- 4）写文件，越权植入木马
- 5）直接通过 Process 执行其他程序
- 6）执行高危命令

解决方法：

1）超时控制

通过创建一个守护线程，超时后自动中断 process（Runtime 执行命令实际开启了一个子进程，让守护线程睡眠指定时间后，销毁子进程即可）

2）限制内存

我们**不能**让每个 java 进程的执行占用的 JM 最大堆内存空间都和系统的一致，实际上应该小一点，比如说 256MB

在启动 Java 时，可以指定 JVM 的参数：`-Xmx256m`（最大堆空间大小）`-Xms`（初始堆空间大小）

```java
java -Xmx256m
```

3）限制代码

定义一个黑白名单，比如哪些操作是禁止的，可以就是一个列表

HuTool 字典树工具类：WordTree，可以用更少的空间存储更多的敏感词汇，实现更高效的敏感

一开始就可以通过 WordTree 检查代码中是否有敏感词

缺点：

- 你无法遍历所有的黑名单
- 不同的编程语言，你对应的领域、关键词都不一样，限制人工成本很大

4）限制用户对文件、内存、CPU、网络等资源的操作和访问。

Java 安全管理器（SecurityManager）是 Java 提供的保护 JVM、Java 安全的机制，可以实现更严格的资源和操作限制。

实际情况下，我们只需要限制子程序的权限即可，没必要限制开发者自己写的程序。

在运行 java 程序时，指定安全管理器：

```java
java -Xmx256m -Dfile.encoding=UTF-8 -cp E:\E_Boer_workpace\boer-project\yuoj\yuoj-code-sandbox-master\tempCode\b2e241ba-3b04-4d19-a13a-3e1597fb2c15:src/main/java/com/yupi/yuojcodesandbox/security -Djava.security.manager=MySecurityManager Main 1 2
```

缺点

- 如果要做比较严格的权限限制，需要自己去判断哪些文件、包名需要允许读写，**粒度太细**了，难以精细化控制
- 安全管理器本身也是 Java 代码，也有可能存在漏洞（还是程序上的限制，没到系统的层面）

5）运行环境隔离

系统层面上，把用户程序封装到沙箱里，和宿主机（我们的电脑/服务器）隔离开

Docker 容器技术能够实现（底层是用 cgroup、namespace 等方式实现的）

### 代码沙箱 Docker 实现

#### Java 操作 Docker

使用 Docker-Java

- *DockerClientConfig*：用于定义初始化 DockerClient 的配置（类比 MySQL 的连接、线程数配置）
- *DockerHttpClient*：用于向 Docker 守护进程（操作 Docker 的接口）发送请求的客户端，低层封装（不推荐使用），你要自己构建请求参数（简单地理解成 JDBC）
- *DockerClient*：才是真正和 Docker 守护进程交互的、最方便的 SDK，高层封装，对 DockerHttpClient 再进行了一层封装（理解成 MyBatis）

#todo 

### 模板方法优化代码沙箱

模板方法：定义一套通用的执行流程（抽象类），让子类负责每个执行步骤的具体实现

模板方法的适用场景：适用于有规范的流程，且执行流程可以复用

作用：大幅节省重复代码量，便于项目扩展、更好维护

### 提供代码沙箱 API

直接在 controller 暴露 CodeSandbox 定义的接囗（不安全，别人也可以调用代码沙箱）

#### api 接口加密

1）调用方与服务提供方之间约定一个字符串（最好**加密**）

- 优点：实现最简单，比较适合**内部系统之间**相互调用（相对可信的环境内部调用）
- 缺点：不够灵活，如果 key 泄露或变更，需要重启代码

代码沙箱服务，先定义约定的字符串：

```java
// 定义鉴权请求头和密钥  
private static final String AUTH_REQUEST_HEADER = "auth";  
private static final String AUTH_REQUEST_SECRET = "secretKey";
```

## 单体项目改造为微服务

用户登录功能：需要改造为分布式登录其他内容（代码：git）

> 其他内容：
> - 有没有用到单机的锁? 改造为分布式锁
> - 有么有用到本地缓存? 改造为分布式缓存(Redis)
> - 需不需要用到分布式事务? 比如操作多个库

### 模块划分

模块划分见 [系统功能](#系统功能)（代码：git）

### 路由划分

用 springboot 的 context-path 统一修改各项目的接口前缀，比如：

- 用户服务:
	- /api/user
	- /api/user/inner（内部调用，网关层面要做限制）
- 题目服务：
	- /api/question（也包括题目提交信息）
	- /api/question/inner（内部调用，网关层面要做限制）
- 判题服务：
	- /api/judge
	- /api/judge/inner（内部调用，网关层面要做限制）

需要修改每个服务提供者的 context-path 全局请求路由（配置文件中）

```java
server:
  servlet:
    context-path: /api/user
```

```java
@RestController
//@RequestMapping("/user")
@RequestMapping("/") // controller上的统一路径去掉
public class UserController {}
```

### 初始化项目

[Cloud Native App Initializer (aliyun.com)](https://start.aliyun.com/)

### 服务调用

现在的问题是，题目服务依赖用户服务，但是代码已经分到不同的包，找不到对应的 Bean。
可以使用 Open Feign 组件实现跨服务的远程调用。

1）梳理服务的调用关系，确定哪些服务(接口)需要给内部调用

- 用户服务:没有其他的依赖
- 题目服务:
	- userService.getByld(userld)
	- userService.getUserVO(user)
	- userService.listBylds(userldSet)
	- userService.isAdmin(loginUseruserService.getLoginUser(request)
	- judgeService.doJudge(questionSubmitld)
- 判题服务:
	- questionService.getByld(questionld)
	- questionSubmitService.getByld(questionSubmitld)
	- questionSubmitService.updateByld(questionSubmitUpdate)

2）确认要提供哪些服务

- 用户服务：
	- userService.getByld(userld)
	- userService.getUserVO(user)
	- userService.listBylds(userldSet)
	- userService.isAdmin(loginUser)
	- userService.getLoginUser(request)
- 题目服务:
	- questionService.getByld(questionld)
	- questionSubmitService.getByld(questionSubmitld)
	- questionSubmitService.updateByld(questionSubmitUpdate)
- 判题服务
	- judgeService.doJudge(questionSubmitld)

3）实现 client 接口（代码见 git）

注意：
- 像 isAdmin() 这种方法没必要远程调用，用默认方法实现即可
- 为什么 getById()，getLoginUser() 这种前端也能访问的接口，要再提供一个 inner 接口呢？
	- 这两个解耦前端访问是需要认证和权限校验的，同理 getQuestionById() 也是，只有本人和管理员才能看到题目的完整信息（比如答案）

```java
@FeignClient(name = "yuoj-backend-user-service",path = "/api/user/inner")
public interface UserFeignClient {

    @GetMapping("/get/id")
    User getById(@RequestParam("userId") long userId);

    @GetMapping("/get/ids")
    List<User> listByIds(@RequestParam("idList") Collection<Long> idList);

    default User getLoginUser(HttpServletRequest request) {
        // 先判断是否已登录
        Object userObj = request.getSession().getAttribute(USER_LOGIN_STATE);
        User currentUser = (User) userObj;
        if (currentUser == null || currentUser.getId() == null) {
            throw new BusinessException(ErrorCode.NOT_LOGIN_ERROR);
        }
        return currentUser;
    }

    default boolean isAdmin(User user) {
        return user != null && UserRoleEnum.ADMIN.getValue().equals(user.getUserRole());
    }

    default UserVO getUserVO(User user){
        if (user == null) {
            return null;
        }
        UserVO userVO = new UserVO();
        BeanUtils.copyProperties(user, userVO);
        return userVO;
    }
}
```

4）修改各业务服务的调用代码为 feignClient（代码见 git）

5）编写服务的实现类

```java
@RestController
@RequestMapping("/inner")
public class UserInnerController implements UserFeignClient {
    @Resource
    private UserService userService;

    @Override
    @GetMapping("/get/id")
    public User getById(@RequestParam("userId") long userId) {
        return userService.getById(userId);
    }

    @Override
    @GetMapping("/get/ids")
    public List<User> listByIds(@RequestParam("idList") Collection<Long> idList) {
        return userService.listByIds(idList);
    }
}
```

6）主启动类 Feign 注解、配置文件配置服务发现 nacos 地址


