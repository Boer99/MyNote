

## 核心业务流程

![700](assets/Pasted%20image%2020240401003446.png)

![](assets/Pasted%20image%2020240401003823.png)

## 后端架构设计

采用**微服务架构**

整个系统的后端分为：  
- 用户服务：提供用户登录、用户的增删改查等管理功能  
- 题目服务：提供题目的增删改查管理、==题目提交==功能  
- 判题服务：提供判题功能，调用代码沙箱并==比对判题结果== 
- 代码沙箱：提供==编译执行代码、返回结果==的功能  
- 公共模块：提供公共代码，比如数据模型、全局请求响应封装、全局异常处理、工具类等  
- 网关服务：提供统一的 API 转发、聚合文档、全局跨域解决等功能  
  
各模块之间的关系，3 个核心：  
1. 由网关服务集中接受前端的请求，并转发到对应的业务服务。  
2. 判题服务需要调用题目服务获取题目信息和测试用例、调用代码沙箱完成代码的编译和执行并比对结果，服务间通过 Open Feign 相互调用。  
3. 所有服务都需要引入公共模块。

## 库表设计

项目主要包含 3 个核心表：用户表、题目表、题目提交表

### 用户表

- userRole：用户角色（只有管理员才能发布和管理题目，普通用户只能看题）

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

- 题目内容：存放题目的介绍、输入输出提示、描述、具体的详情
- 题目答案
- 题目标签（json 数组字符串）：栈、队列、链表、简单、中等、困难
- 提交数、通过数：便于分析统计（通过率、难度）

> 扩展字段：
> - 通过率
> - 判题类型

**判题相关字段（重点）：**

在题目表中，==通过 json 对象字符串存储 判题配置信息 judgeConfig（text 类型）==，而不是把每种具体的配置单独作为一列存储，==便于该字段的读写和扩展（不用修改数据库表字段）==。

> 如果说题目不是很复杂，用例文件大小不大的话，可以直接存在数据库表里。
> 但是如果用例文件比较大，>512 KB 建议单独存放在一个文件中，数据库中只保存文件 url（类似存储用户头像）

1）**judgeConfig**：判题配置（json 对象）

- 时间限制 timeLimit
- 内存限制 memoryLimit
- ...（扩展，例如不同的语言不同的限制）

2）**judgeCase**：判题用例（json 数组）

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

- questionld
- language：编程语言
- code
- **status**：判题状态（待判题、判题中、成功、失败）
- **judgelnfo**（json 对象）：执行信息（比如程序的失败原因）、程序执行消耗的时间、空间

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

判题状态枚举值：

```java
WAITING("等待中", 0),  
RUNNING("判题中", 1),  
SUCCESS("通过", 2),  
FAIL("失败", 3); //TODO
```

```java
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

程序执行信息 枚举值:

```java
ACCEPTED("成功", "Accepted"),
WRONG_ANSWER("答案错误", "Wrong Answer"),
//COMPILE_ERROR("Compile Error", "编译错误"),
MEMORY_LIMIT_EXCEEDED("", "内存溢出"),
TIME_LIMIT_EXCEEDED("Time Limit Exceeded", "超时"),
//PRESENTATION_ERROR("Presentation Error", "展示错误"),
//WAITING("Waiting", "等待中"),
//OUTPUT_LIMIT_EXCEEDED("Output Limit Exceeded", "输出溢出"),
//DANGEROUS_OPERATION("Dangerous Operation", "危险操作"),
//RUNTIME_ERROR("Runtime Error", "运行错误"),
//SYSTEM_ERROR("System Error", "系统错误");
```


### 实体类的 json 字段设计

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


## 判题模块

### 判题模块和代码沙箱关系

![500](assets/Pasted%20image%2020240401004249.png)


### 判题服务业务流程

判题服务业务流程：

1. 传入题目的提交 id，获取到对应的题目（判题配置、判题用例）、提交信息（包含代码、编程语言等）
2. 如果 题目提交状态 不为“等待中”，就不用重复执行了
3. 更改 题目提交状态 为“判题中”，防止重复执行
4. 调用沙箱，获取到执行结果
5. 根据沙箱的执行结果，设置题目的判题状态和信息

执行结果的判断逻辑：

1. 先判断沙箱执行的结果输出**数量**是否和预期输出数量相等
2. 依次判断**每一项**输出和预期输出是否相等
3. 判题题目的**限制**是否符合要求
4. 可能还有**其他**的异常情况


### 代码沙箱调用架构设计（工厂模式、代理模式） 

为了设计灵活、可扩展的判题机模块，我用了多种设计模式和方法：

1）为了提高系统的灵活性，对于代码沙箱的调用，没有选择硬编码的方式指定单一的代码沙箱，而是**定义了一个通用的代码沙箱调用接口**，提供了多种代码沙箱调用的实现类

这样在你使用其他的代码沙箱实现类时，就不用去修改名称了， 便于扩展。

```java
/**  
 * 代码沙箱接口定义  
 */  
public interface CodeSandbox {  
  
    /**  
     * 执行代码  
     *  
     * @param executeCodeRequest  
     * @return  
     */  
    ExecuteCodeResponse executeCode(ExecuteCodeRequest executeCodeRequest);  
}

public class ExecuteCodeRequest {  

    private List<String> inputList;  
  
    private String code;  
  
    private String language;  

	// 自行扩展：时间限制、空间限制
}

public class ExecuteCodeResponse {  
  
    private List<String> outputList;  
    /**  
     * 接口信息  
     */  
    private String message;  
    /**  
     * 执行状态  
     */  
    private Integer status;  
    /**  
     * 判题信息  
     */  
    private JudgeInfo judgeInfo;  
}
```

> 扩展思路：增加一个查看代码沙箱状态的接口

2）定义多种不同的代码沙箱调用实现

- 示例代码沙箱：仅为了跑通业务流程
- 远程代码沙箱：自己开发的沙箱接口
- 第三方代码沙箱：调用网上现成的代码沙箱， https://github.com/criyle/go-judge

远程代码沙箱调用实现：

```java
public class RemoteCodeSandbox implements CodeSandbox {

    // 定义鉴权请求头和密钥
    private static final String AUTH_REQUEST_HEADER = "auth";

    private static final String AUTH_REQUEST_SECRET = "secretKey";

    @Override
    public ExecuteCodeResponse executeCode(ExecuteCodeRequest executeCodeRequest) {
        System.out.println("远程代码沙箱");
        String url = "http://localhost:8090/executeCode";
        String json = JSONUtil.toJsonStr(executeCodeRequest);
        String responseStr = HttpUtil.createPost(url)
                .header(AUTH_REQUEST_HEADER, AUTH_REQUEST_SECRET)
                .body(json)
                .execute()
                .body();
        if (StringUtils.isBlank(responseStr)) {
            throw new BusinessException(ErrorCode.API_REQUEST_ERROR, "executeCode remoteSandbox error, message = " + responseStr);
        }
        return JSONUtil.toBean(responseStr, ExecuteCodeResponse.class);
    }
}

```

3）使用**静态工厂模式+配置文件**的方式，来生成对应的代码沙箱实现类，**解决代码沙箱调用硬编码问题**

用户在配置文件中配置的字符串参数（沙箱类别）

```yml
# 代码沙箱配置
codesandbox:
  type: remote
```

此处使用静态工厂模式，实现比较简单，符合我们的需求。

```java
/**
 * 代码沙箱工厂（根据字符串参数创建指定的代码沙箱实例）
 */
public class CodeSandboxFactory {

    /**
     * 创建代码沙箱示例
     *
     * @param type 沙箱类型
     */
    public static CodeSandbox newInstance(String type) {
        switch (type) {
            case "example":
                return new ExampleCodeSandbox();
            case "remote":
                return new RemoteCodeSandbox();
            case "thirdParty":
                return new ThirdPartyCodeSandbox();
            default:
                return new ExampleCodeSandbox();
        }
    }
}
```

service 代码中创建代码沙箱：

```java
@Value("${codesandbox.type:example}")
private String type;
    
CodeSandbox codeSandbox = CodeSandboxFactory.newInstance(type);
```

4）为了在调用代码沙箱前后进行统一的日志操作，使用**静态代理模式**对代码沙箱调用进行封装，能够在不改变代码沙箱调用实现类的前提下，集中地增强代码沙箱的能力

日志记录代码沙箱的请求和响应信息

```java
@Slf4j
public class CodeSandboxProxy implements CodeSandbox {

    private final CodeSandbox codeSandbox;
    
    public CodeSandboxProxy(CodeSandbox codeSandbox) {
        this.codeSandbox = codeSandbox;
    }

    @Override
    public ExecuteCodeResponse executeCode(ExecuteCodeRequest executeCodeRequest) {
        log.info("代码沙箱请求信息：" + executeCodeRequest.toString());
        ExecuteCodeResponse executeCodeResponse = codeSandbox.executeCode(executeCodeRequest);
        log.info("代码沙箱响应信息：" + executeCodeResponse.toString());
        return executeCodeResponse;
    }
}
```

获取增强的代码沙箱调用

```java
CodeSandbox codeSandbox = CodeSandboxFactory.newInstance(type);
codeSandbox = new CodeSandboxProxy(codeSandbox);
```


### 策略模式优化判题策略代码

由于不同的编程语言可能会有不同的判题逻辑，比如Java 语言的内存和时把这些判题逻辑如果都写在 service 中通过 if... else..来区分，整个代码文件的圈复杂度读和维护。

可以采用**策略模式**，针对不同的情况，定义独立的策略

1）定义判题策略接口 JudgeStrategy，提供 doJudge 判题方法，让代码更==通用化==

```java
public interface JudgeStrategy {  
  
    /**  
     * 执行判题  
     * @param judgeContext  
     * @return  
     */  
    JudgeInfo doJudge(JudgeContext judgeContext);  
}
```

2）将不同语言的判题算法分别封装成不同的策略类，实现上述接口，比如JavaLanguageStrategy、DefaultJudgeStrategy。这样来，需要修改某个策略时只需要修改对应的类即可，==便于维护和扩展==。

3）定义了 JudgeContext 上下文类，用于存储判题策略需要用到的信息，传递给策略类

```Java
/**
 * 上下文（用于定义在策略中传递的参数）
 */
@Data
public class JudgeContext {

    private JudgeInfo judgeInfo;

    private List<String> inputList;

    private List<String> outputList;

    private List<JudgeCase> judgeCaseList;

    private Question question;

    private QuestionSubmit questionSubmit;
}
```

4）JudqeManager 类，在这个类中根据具体的编程语言来执行对应的策略，从而 简化了 service 的调用

```java
/**
 * 管理判题策略的选择（简化对具体判题策略的调用）
 */
@Service
public class JudgeManager {
    /**
     * 执行判题
     *
     * @param judgeContext
     * @return
     */
    public JudgeInfo doJudge(JudgeContext judgeContext) {
        QuestionSubmit questionSubmit = judgeContext.getQuestionSubmit();
        String language = questionSubmit.getLanguage();
        // 默认判题策略
        JudgeStrategy judgeStrategy = new DefaultJudgeStrategy();
        // 根据语言切换的判题策略
        if ("java".equals(language)) {
            judgeStrategy = new JavaLanguageJudgeStrategy();
        }
        return judgeStrategy.doJudge(judgeContext);
    }
}
```


### api 接口加密

由于代码沙箱服务提供了暴露在外的接口，所以必须通过某种权限校验机制保证安全。  

有 2 种方式：  
1. 调用方与服务提供方之间约定一个加密字符串，调用方发送请求时必须在请求头中携带该字符串，服务提供方会从请求头中取出该字符串并进行比对。  
2. 使用 API 签名认证机制，给可信的调用方分配 accessKey、secretKey。在调用方发送请求时，会使用 secretKey 对请求参数等内容进行签名，并设置在请求头中；服务提供方会用同样的 secretKey 对请求参数等内容进行签名，然后校验签名是否一致。  
  
第 1 种方式的优点是简单易实现，但是不够安全，比较适合相对可信的内网环境调用；如果需要将服务暴露在公网，那么推荐第 2 种方式。

方法一实现：调用方与服务提供方之间约定一个字符串（最好**加密**）

- 优点：实现最简单，比较适合**内部系统之间**相互调用（相对可信的环境内部调用）
- 缺点：不够灵活，如果 key 泄露或变更，需要重启代码

代码沙箱服务，先定义约定的字符串：

```java
// 定义鉴权请求头和密钥  
private static final String AUTH_REQUEST_HEADER = "auth";  
private static final String AUTH_REQUEST_SECRET = "secretKey";
```


## Java 原生代码沙箱实现

> 以 **Java** 编程语言为主，带大家实现代码沙箱，重要的是学思想、学关键流程
> 
> 扩展：可以自行实现 C++ 语言的代码沙箱


### 模板方法模式定义沙箱执行流程

Java 原生代码沙箱和 Docker 代码沙箱这两种实现方式的核心业务流程是相同的，都需要经历以下几个步骤：  
- 1 把用户的代码保存为文件  
- 2 编译代码，得到 class 文件  
- 3 执行 Java 代码，收集整理输出结果  
- 5 文件清理，释放空间  
- 6 错误处理，提升程序健壮性  

区别在于：
- Java 原生代码沙箱是通过 Runtime.exec 执行命令行操作来执行代码，并通过 Process 对象的流来获取输出结果，不够安全；
- Docker 代码沙箱是通过创建隔离的 Java 容器并且通过 exec 命令在容器内执行 Java 代码和获取输出，更加安全。

实现步骤如下：  
1. 先定义了一个==抽象==的代码沙箱模板类，用一个方法定义了核心流程（保存文件、编译代码、执行代码、收集结果、文件清理、错误处理）  
	1. 核心流程方法 execute 应该为 final 的，不可被重写
2. 在模板类中，把每个环节单独定义为一个方法，可供子类覆写。  
3. 每种代码沙箱的实现类继承模板类，可以复用模板类的默认环节实现（比如保存文件），也可以重写某一个环节（比如 Docker 代码沙箱重写执行逻辑）。  

使用模板方法模式后，大幅减少了重复代码，并且让整个系统的流程更加清晰，提升了项目的可扩展性。


### 核心流程实现

核心实现思路：用程序代替人工，用程序来操作命令行，去编译执行代码 Java 

进程执行管理类：Process

实际 OJ 系统中，对用户输入的代码会有一定的要求，便于系统统一地处理。所以此处，我们把用户输入代码的==类名限制为 Main==（参考 Poj），可以减少类名不一致的风险，而且不用从用户代码中提取类名

---

核心流程：

1）把用户的代码保存为文件

- java 语言的代码保存为 `.java` 后缀
- 不同的用户代码通过 uuid 后缀的文件夹 隔离存放

```java
 public File saveFile(String code) {
	String userDir = System.getProperty("user.dir");
	String globalCodePathName = userDir + File.separator + GLOBAL_CODE_DIR_NAME;
	// 判断全局代码目录是否存在，没有则新建
	if (!FileUtil.exist(globalCodePathName)) {
		FileUtil.mkdir(globalCodePathName);
	}
	// 把用户的代码隔离存放
	String userCodeParentPath = globalCodePathName + File.separator + UUID.randomUUID();
	String userCodePath = userCodeParentPath + File.separator + GLOBAL_JAVA_CLASS_NAME;
	File userCodeFile = FileUtil.writeString(code, userCodePath, StandardCharsets.UTF_8);
	return userCodeFile;
}
```

2）编译代码，得到 class 文件

通过 **Runtime 类**执行 `javac -encoding utf-8 .\Main.java` 命令，返回 **Process 对象**

```java
public ExecuteMessage compileFile(File userCodeFile) {
	String compileCmd = String.format("javac -encoding utf-8 %s", userCodeFile.getAbsoluteFile());
	try {
		Process compileProcess = Runtime.getRuntime().exec(compileCmd);
		ExecuteMessage executeMessage = ProcessUtils.runProcessAndGetMessage(compileProcess, "编译");
		if (executeMessage.getExitValue() != 0) {
			throw new RuntimeException("编译错误");
		}
		return executeMessage;
	} catch (Exception e) {
		throw new RuntimeException(e);
	}
}
```

通过 exitValue 判断程序是否正常返回
```java
 int exitValue = process.waitFor();

// 正常退出
if (exitValue == 0) {
	
} else {

}
```

从 Process 对象 的 inputStream 和 errorStream 获取**控制台输出**

```java
// 分批获取进程的正常输出
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(process.getInputStream()));

// 分批获取进程的错误输出  
BufferedReader errorBufferedReader = new BufferedReader(new InputStreamReader(process.getErrorStream()));
```

3）执行代码，得到输出结果

- 通过 Runtime 类执行 `java -cp` 命令，返回 Process 对象
- 循环遍历每个输入用例，获取输出结果
- 输入用例直接拼接在命令后面

```java
for (String inputArgs : inputList) {
            String runCmd = String.format("java -Xmx256m -Dfile.encoding=UTF-8 -cp %s Main %s",
                    userCodeParentPath,
                    inputArgs);
            try {
                Process runProcess = Runtime.getRuntime().exec(runCmd);
                // 超时控制
//                new Thread(() -> {
//                    try {
//                        Thread.sleep(OVER_TIME);
//                        System.out.println("编译超时，已终止编译进程");
//                        runProcess.destroy();
//                    } catch (InterruptedException e) {
//                        throw new RuntimeException(e);
//                    }
//                }).start();
                ExecuteMessage executeMessage = ProcessUtils.runProcessAndGetMessage(runProcess, "运行");
                System.out.println(executeMessage);
                executeMessageList.add(executeMessage);
            } catch (IOException e) {
                throw new RuntimeException("执行错误", e);
            }
        }
```

执行：

```bash
java -cp 路径 Main 参数
```

> cp：classpath

4）收集整理输出结果

- 运行时间取所有用例中的最大值

```java
public ExecuteCodeResponse getOutputResponse(List<ExecuteMessage> executeMessageList) {
	ExecuteCodeResponse executeCodeResponse = new ExecuteCodeResponse();
	ArrayList<String> outputList = new ArrayList<>();
	long maxTime = 0; // 取用时最大值，便于判断是否超时
	for (ExecuteMessage executeMessage : executeMessageList) {
		String errorMessage = executeMessage.getErrorMessage();
		// 判断是否编译错误
		if (StrUtil.isNotBlank(errorMessage)) {
			// 设置返回结果
			executeCodeResponse.setMessage(errorMessage);
			executeCodeResponse.setStatus(3); // 用户提交的代码执行中存在错误
			// 有一个用例执行错误，就不用继续遍历了
			break;
		}
		outputList.add(executeMessage.getMessage());
		Long time = executeMessage.getTime();
		if (time != null) {
			maxTime = Math.max(maxTime, executeMessage.getTime());
		}
	}
	executeCodeResponse.setOutputList(outputList);
	if (outputList.size() == executeMessageList.size()) {
		executeCodeResponse.setStatus(1);
	}
	JudgeInfo judgeInfo = new JudgeInfo();
	judgeInfo.setTime(maxTime);
	// 要借助第三方库来获取内存占用
	// judgeInfo.setMemory();
	executeCodeResponse.setJudgeInfo(judgeInfo);
	return executeCodeResponse;
}
```

5）文件清理

6）错误处理，提升程序健壮性


### 异常情况演示

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

通过创建一个守护线程，超时后自动中断 process（Runtime 执行命令实际开启了一个子进程，==让守护线程睡眠指定时间后，销毁子进程即可==）

2）限制内存

我们**不能**让每个 java 进程的执行占用的 JM 最大堆内存空间都和系统的一致，实际上应该小一点，比如说 256MB

在启动 Java 时，可以指定 JVM 的参数：`-Xmx256m`（最大堆空间大小）`-Xms`（初始堆空间大小）

```java
java -Xmx256m
```

3）限制代码

定义一个黑白名单，比如哪些操作是禁止的，可以就是一个列表

**HuTool 字典树工具类：WordTree**，可以用更少的空间存储更多的敏感词汇，实现更高效的敏感

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


## 代码沙箱 Docker 实现

#### Java 操作 Docker

使用 Docker-Java

- *DockerClientConfig*：用于定义初始化 DockerClient 的配置（类比 MySQL 的连接、线程数配置）
- *DockerHttpClient*：用于向 Docker 守护进程（操作 Docker 的接口）发送请求的客户端，低层封装（不推荐使用），你要自己构建请求参数（简单地理解成 JDBC）
- *DockerClient*：才是真正和 Docker 守护进程交互的、最方便的 SDK，高层封装，对 DockerHttpClient 再进行了一层封装（理解成 MyBatis）

#todo 


## 单体项目改造为微服务

用户登录功能：需要改造为分布式登录其他内容（代码：git）

解决 cookie 跨路径问题

> 其他内容：
> - 有没有用到单机的锁? 改造为分布式锁
> - 有么有用到本地缓存? 改造为分布式缓存(Redis)
> - 需不需要用到分布式事务? 比如操作多个库


### 模块划分

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

### 网关

微服务网关(yuoj-backend-gateway)：Gateway 聚合所有的接口，统一接受处理前端的请求

为什么要用?
- 所有的服务==端口==不同，增大了前端调用成本
- 所有服务是分散的，你可需要集中进行管理、操作，比如集中解决跨域、鉴权、接口文档、服务的路由、接口安全性、流量染色

> Gateway 是应用层网关：会有一定的业务逻辑(比如根据用户信息判断权限)
> 
> Nginx 是接入层网关：比如每个请求的日志，通常没有业务逻辑

#### 聚合文档

以一个全局的视角集中查看管理接口文档

1）给所有业务服务引入依赖，并开启配置

[快速开始 | Knife4j (xiaominfo.com)](https://doc.xiaominfo.com/docs/quick-start#spring-boot-2)

2）给网关引入依赖，并开启配置

[Spring Cloud Gateway网关聚合 | Knife4j (xiaominfo.com)](https://doc.xiaominfo.com/docs/middleware-sources/spring-cloud-gateway/spring-gateway-introduction)

#### 全局配置跨域

在网关服务中添加以下配置：
```java
@Configuration
public class CosConfig {
    @Bean
    public CorsWebFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedMethod("*");
        config.setAllowCredentials(true);
        // todo 实际改为线上真实域名、本地域名
        config.setAllowedOriginPatterns(Arrays.asList("*"));
        config.addAllowedHeader("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource(new PathPatternParser());
        source.registerCorsConfiguration("/**", config);
        return new CorsWebFilter(source);
    }
}
```

#### 权限校验

可以使用 Spring Cloud Gateway 的 Filter 请求拦截器，接受到请求后根据请求的路径判断能否访问

> 扩展：可以在网关实现接口限流


## 消息队列解耦

由于判题操作是一个比较重的服务（需要调用代码沙箱），使用 RabbitMQ 消息队列对判题操作进行异步化解耦。  

改造后的业务流程：用户提交题目时，由题目服务发送一条消息（题目提交id）到队列，然后判题服务监听到该队列的消息并进行判题处理，并且异步更改题目的判题状态。  

这样做的好处：  
- 对用户来说：不需要在前端同步等待，优化了体验（==加快响应速度==）
- 对系统来说：==解耦了题目服务和判题服务==，两者不需要相互调用，即使判题服务繁忙或宕机，题目服务依然可以发送判题任务到队列，等判题服务恢复后继续处理。


# 前端

## 前端组件库

[Arco Design - 企业级产品的完整设计和开发解决方案](https://arco.design/)

## Markdown 编辑器

Markdown 编辑器组件：[bytedance/bytemd: ByteMD v1 repository (github.com)](https://github.com/bytedance/bytemd)

## 代码编辑器

[microsoft/monaco-editor: A browser based code editor (github.com)](https://github.com/microsoft/monaco-editor)


