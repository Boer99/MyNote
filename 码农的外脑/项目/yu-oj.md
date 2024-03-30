
# 前端
## 前端组件库

[Arco Design - 企业级产品的完整设计和开发解决方案](https://arco.design/)

# 系统功能

- 用户模块
	- 注册(后端已实现)
	- 登录(后端已实现，前端已实现)

- 题目模块
	- 创建题目(管理员)
	- 删除题目(管理员)
	- 修改题目(管理员)
	- 搜索题目(用户)
	- 在线做题(题目详情页)

- 判题模块
	- 提交判题(结果是否正确与错误)
	- 错误处理(内存溢出、安全性、超时)自主实现 代码沙箱(安全沙箱 C .
	- 开放接口(提供一个独立的新服务)

# 后端

## 后端初始化

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

