# 业务
![](assets/Pasted%20image%2020241003162557.png)

# 架构设计

传统项目架构：只适合单体架构
- controller
- service（原子的）
- biz（聚合）
- dao

> 简单 DDD（Domain-Driven Design，领域驱动设计）架构

现有架构：
- api（所有对外提供接口）
	- 接口
	- req
	- resp
- application（整个项目的入口）
	- controller
	- mq
	- job
- domain（原子性的层）
- infra：基础设施层
- common
- starter

现有架构复杂版：
- api：对外接口层
	- 能力的接口
	- req
	- resp
- application：应用层
	- controller
	- mq：消费者
	- job
- aggressive：聚合层，聚合领域能力
- domain：领域层
	- bo
	- service：关注的是领域能力
- infra：基础设施层
	- basic
		- mapper
		- service
		- po（Persistent Object，永久对象）
	- rpc
	- service
		- entity
	- mq：生产者
- common
	- config
	- dict
	- enum
	- util
- starter：启动层，无关乎任何业务，纯启动；聚合当前包的能力


# 通用

## 依赖版本管理

start.aliyun.com

## 数据库密码加密

密码明文写在代码中，如果代码泄露会导致密码泄露，所以要把密码加密成密文写在代码中

```yml
spring:
  datasource:
    username: root
    password: e6AHHtvK2HkNe6cCk6s4HhyV9GLnexMx9W+hd6n/xZq+PvZDFQqkdeDxmqPb7vqbVQ3OcxTDzmS9g4aJcYb6eg==
    
    druid:
      connectionProperties: config.decrypt=true;config.decrypt.key=${publicKey};

publicKey: MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAIKxyqk6dhT5aR2iplF/pdPqN35iJuVhC9LvVxmRBFY0Gv8yvAOOMcjRMcIilLLozJU1jDW2KS9BOV00wOrZ9J8CAwEAAQ==
```

```java
package com.clear.subject.infra.basic.utils;

import com.alibaba.druid.filter.config.ConfigTools;

import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;

/**
 * 数据库加密
 */
public class DruidEncryptUtil {

    private static String publicKey;
    private static String privateKey;

    static {
        try {
            // 调用 druid 生成私钥、公钥、密文
            String[] keyPair = ConfigTools.genKeyPair(512);
            privateKey = keyPair[0];
            System.out.println("privateKey:" + privateKey);
            publicKey = keyPair[1];
            System.out.println("publicKey:" + publicKey);

        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException(e);
        } catch (NoSuchProviderException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 加密
     *
     * @param plainText 待加密的文本
     * @return 加密之后的密文
     */
    public static String encrypt(String plainText) throws Exception {
        String encrypt = ConfigTools.encrypt(privateKey, plainText);
        System.out.println("encrypt:" + encrypt);
        return encrypt;
    }

    /**
     * 解密
     *
     * @param encryptText 需要解密的密文
     * @return 解密后的文本
     */
    public static String decrypt(String encryptText) throws Exception {
        String decrypt = ConfigTools.decrypt(publicKey, encryptText);
        System.out.println("decrypt:" + decrypt);
        return decrypt;
    }

    /**
     * 测试
     * @param args
     * @throws Exception
     */
    public static void main(String[] args) throws Exception {
        String encrypt = encrypt("123456");
        System.out.println("encrypt:" + encrypt);
    }
}

```

## mapstruct 实体类转换工具 #todo

## 空值全局处理

把响应结果中值为 null 的属性都去掉

```java
/**
 * MVC全局处理
 */
@Configuration
public class GlobalConfig extends WebMvcConfigurationSupport {

    @Resource
    private LoginInterceptor loginInterceptor;

    @Override
    protected void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        super.configureMessageConverters(converters);
        converters.add(this.mappingJackson2HttpMessageConverter());
    }

    /**
     * 自定义mappingJackson2HttpMessageConverter
     * 目前实现：空值忽略、空字段可返回
     *
     * @return
     */
    private MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);

        // 全局空值处理
//        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter(objectMapper);
        return converter;
    }
}
```

## Nacos

### 云配置

oss 模块

```yml
storage:
  service:
    type: minio
```


# 刷题模块

![](assets/Pasted%20image%2020241003172022.png)

## 库表设计

### 分类表

```sql
create table subject_category
(
    id            bigint auto_increment comment '主键'
        primary key,
    category_name varchar(16) null comment '分类名称',
    category_type tinyint(2)  null comment '分类的类型 1表示大类 2表示二级分类',
    image_url     varchar(64) null comment '图标链接',
    parent_id     bigint      null comment '父级id',
    created_by    varchar(32) null comment '创建人',
    created_time  datetime    null comment '创建时间',
    update_by     varchar(32) null comment '更新人',
    update_time   datetime    null comment '更新时间',
    is_deleted    int         null comment '是否删除 0: 未删除 1: 已删除'
)
    comment '题目分类表' charset = utf8
                         row_format = DYNAMIC;
```

### 标签表

```sql
create table subject_label
(
    id           int(20) auto_increment comment '主键'
        primary key,
    label_name   varchar(32)   null comment '标签名称',
    sort_num     int           null comment '排序',
    category_id  bigint        null comment '分类id',
    created_by   varchar(32)   null comment '创建人',
    created_time datetime      null comment '创建时间',
    update_by    varchar(32)   null comment '更新人',
    update_time  datetime      null comment '更新时间',
    is_deleted   int default 0 null
)
    comment '题目标签表' charset = utf8
                         row_format = DYNAMIC;
```

### 题目表

```sql
create table subject_info
(
    id                bigint auto_increment comment '主键'
        primary key,
    subject_name      varchar(128) null comment '题目名称',
    subject_difficult bigint(4)    null comment '题目难度',
    settle_name       varchar(32)  null comment '出题人名',
    subject_type      bigint(2)    null comment '题目类型 1单选 2多选 3判断 4简答',
    subject_score     int(4)       null comment '题目分数',
    subject_parse     varchar(512) null comment '题目解析',
    created_by        varchar(32)  null comment '创建人',
    created_time      datetime     null comment '创建时间',
    update_by         varchar(32)  null comment '更新人',
    update_time       datetime     null comment '更新时间',
    is_deleted        int          null comment '题目状态（1：已删除，0未删除）'
)
    comment '题目信息表' charset = utf8
                         row_format = DYNAMIC;
```

### 题目分类关系表

这张表记录了题目分类，标签与题目的mapping关系
```sql
CREATE TABLE `subject_mapping`
(
    `id`           bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `subject_id`   bigint(20)  DEFAULT NULL COMMENT '题目id',
    `category_id`  bigint(20)  DEFAULT NULL COMMENT '分类id',
    `label_id`     bigint(20)  DEFAULT NULL COMMENT '标签id',
    `created_by`   varchar(32) DEFAULT NULL COMMENT '创建人',
    `created_time` datetime    DEFAULT NULL COMMENT '创建时间',
    `update_by`    varchar(32) DEFAULT NULL COMMENT '修改人',
    `update_time`  datetime    DEFAULT NULL COMMENT '修改时间',
    `is_deleted`   int(11)     DEFAULT '0',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 479
  DEFAULT CHARSET = utf8 COMMENT ='题目分类关系表';
```

### 单选表

```sql
CREATE TABLE `subject_radio`
(
    `id`             bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `subject_id`     bigint(20)   DEFAULT NULL COMMENT '题目id',
    `option_type`    tinyint(4)   DEFAULT NULL COMMENT 'a,b,c,d',
    `option_content` varchar(128) DEFAULT NULL COMMENT '选项内容',
    `is_correct`     tinyint(2)   DEFAULT NULL COMMENT '是否正确',
    `created_by`     varchar(32)  DEFAULT NULL COMMENT '创建人',
    `created_time`   datetime     DEFAULT NULL COMMENT '创建时间',
    `update_by`      varchar(32)  DEFAULT NULL COMMENT '修改人',
    `update_time`    datetime     DEFAULT NULL COMMENT '修改时间',
    `is_deleted`     int(11)      DEFAULT '0',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8 COMMENT ='单选题信息表';
```

### 多选题表

```sql
CREATE TABLE `subject_multiple`
(
    `id`             bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `subject_id`     bigint(20)  DEFAULT NULL COMMENT '题目id',
    `option_type`    bigint(4)   DEFAULT NULL COMMENT '选项类型',
    `option_content` varchar(64) DEFAULT NULL COMMENT '选项内容',
    `is_correct`     tinyint(2)  DEFAULT NULL COMMENT '是否正确',
    `created_by`     varchar(32) DEFAULT NULL COMMENT '创建人',
    `created_time`   datetime    DEFAULT NULL COMMENT '创建时间',
    `update_by`      varchar(32) DEFAULT NULL COMMENT '更新人',
    `update_time`    datetime    DEFAULT NULL COMMENT '更新时间',
    `is_deleted`     int(11)     DEFAULT '0',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8 COMMENT ='多选题信息表';
```

### 简答题表

```sql
CREATE TABLE `subject_brief`
(
    `id`             bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `subject_id`     int(20)     DEFAULT NULL COMMENT '题目id',
    `subject_answer` text COMMENT '题目答案',
    `created_by`     varchar(32) DEFAULT NULL COMMENT '创建人',
    `created_time`   datetime    DEFAULT NULL COMMENT '创建时间',
    `update_by`      varchar(32) DEFAULT NULL COMMENT '更新人',
    `update_time`    datetime    DEFAULT NULL COMMENT '更新时间',
    `is_deleted`     int(11)     DEFAULT '0',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 232
  DEFAULT CHARSET = utf8 COMMENT ='简答题';
```

### 判断题表

```sql
CREATE TABLE `subject_judge`
(
    `id`           bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `subject_id`   bigint(20)  DEFAULT NULL COMMENT '题目id',
    `is_correct`   tinyint(2)  DEFAULT NULL COMMENT '是否正确',
    `created_by`   varchar(32) DEFAULT NULL COMMENT '创建人',
    `created_time` datetime    DEFAULT NULL COMMENT '创建时间',
    `update_by`    varchar(32) DEFAULT NULL COMMENT '更新人',
    `update_time`  datetime    DEFAULT NULL COMMENT '更新时间',
    `is_deleted`   int(11)     DEFAULT '0',
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8 COMMENT ='判断题';
```


## 新增题目（工厂+策略模式）

采取工厂+策略的模式去做扩展，现在有四种题型，未来无论加多少种，我们都可以==不用动主流程==。

针对不同的题型的保存，封装不同的策略类，策略类通过工厂模式创建

```java
/**
 * 题目类型工厂
 */
@Component
public class SubjectTypeHandlerFactory implements InitializingBean {

    @Resource
    private List<SubjectTypeHandler> subjectTypeHandlerList;

    // 定义一个map，缓存所有的 策略
    private Map<SubjectInfoTypeEnum, SubjectTypeHandler> handlerMap = new HashMap<>();

    public SubjectTypeHandler getHandler(int subjectType) {
        SubjectInfoTypeEnum subjectInfoTypeEnum = SubjectInfoTypeEnum.getByCode(subjectType);
        return handlerMap.get(subjectInfoTypeEnum);
    }

    // 利用spring提供的InitializingBean接口将所有的 策略 做缓存
    @Override
    public void afterPropertiesSet() throws Exception {
        for (SubjectTypeHandler subjectTypeHandler : subjectTypeHandlerList) {
            handlerMap.put(subjectTypeHandler.getHandlerType(),subjectTypeHandler);
        }
    }
}
```

## 题目详情（工厂+策略模式）

查标签，难度啊，出题人啊，等等，这些就直接查，不做 join。


# oss 模块

## 技术选型

minio

## 模块设计

注意:考虑 oss 的扩展性和切换性,

目前对接的 minio，要考虑，如果作为公共的 oss 服务，如何切换到其他的阿里云 oss 或者对接京东云的 oss。

作为基础的 oss 服务，切换等等动作，不应该要求业务方进行改造，以及对切换有感知。

## 适配器模式 #todo 

## 配合 Nacos 实现 Bean 动态加载

Nacos Config 配置文件中配置 storageType

```yml
storage:
  service:
    type: minio
```

配置读取类添加动态刷新注解 `@RefreshScope`

```java
@Configuration
@RefreshScope
public class StorageConfig {

    @Value("${storage.service.type}")
    private String storageType;

    @Bean
    @RefreshScope
    public StorageAdapter storageService() {
        if ("minio".equals(storageType)) {
            return new MinioStorageAdapter();
        } else if ("aliyun".equals(storageType)) {
            return new AliStorageAdapter();
        } else {
            throw new IllegalArgumentException("未找到对应的文件存储处理器");
        }
    }

}

```


# auth 模块

## 技术选型

Sa-Token [框架介绍 (sa-token.cc)](https://sa-token.cc/doc.html#/)

## 库表设计

RABC-0 模型：用户 多对多 角色 多对多 权限

```sql
-- ----------------------------
-- Table structure for auth_permission
-- ----------------------------
DROP TABLE IF EXISTS `auth_permission`;
CREATE TABLE `auth_permission` (
  `id` bigint(20) NOT NULL,
  `name` varchar(64) DEFAULT NULL,
  `parent_id` bigint(20) DEFAULT NULL,
  `type` tinyint(4) DEFAULT NULL,
  `menu_url` varchar(255) DEFAULT NULL,
  `status` tinyint(2) DEFAULT NULL,
  `show` tinyint(2) DEFAULT NULL,
  `icon` varchar(128) DEFAULT NULL,
  `permission_key` varchar(64) DEFAULT NULL,
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `is_deleted` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for auth_role
-- ----------------------------
DROP TABLE IF EXISTS `auth_role`;
CREATE TABLE `auth_role` (
  `id` bigint(20) NOT NULL,
  `role_name` varchar(32) DEFAULT NULL,
  `role_key` varchar(64) DEFAULT NULL,
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `is_deleted` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for auth_role_premission
-- ----------------------------
DROP TABLE IF EXISTS `auth_role_premission`;
CREATE TABLE `auth_role_premission` (
  `id` bigint(20) NOT NULL,
  `role_id` bigint(20) DEFAULT NULL,
  `permission_id` bigint(20) DEFAULT NULL,
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `is_deleted` int(11) DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for auth_user
-- ----------------------------
DROP TABLE IF EXISTS `auth_user`;
CREATE TABLE `auth_user` (
  `id` bigint(20) DEFAULT NULL,
  `user_name` varchar(32) DEFAULT NULL,
  `nick_name` varchar(32) DEFAULT NULL,
  `email` varchar(32) DEFAULT NULL,
  `phone` varchar(32) DEFAULT NULL,
  `password` varchar(64) DEFAULT NULL,
  `sex` tinyint(2) DEFAULT NULL,
  `avatar` varchar(255) DEFAULT NULL,
  `status` tinyint(2) DEFAULT NULL,
  `introduce` varchar(255) DEFAULT NULL,
  `ext_json` varchar(255) DEFAULT NULL,
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `is_deleted` int(11) DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for auth_user_role
-- ----------------------------
DROP TABLE IF EXISTS `auth_user_role`;
CREATE TABLE `auth_user_role` (
  `id` bigint(20) DEFAULT NULL,
  `user_id` bigint(20) DEFAULT NULL,
  `role_id` bigint(20) DEFAULT NULL,
  `created_by` varchar(32) DEFAULT NULL COMMENT '创建人',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` varchar(32) DEFAULT NULL COMMENT '更新人',
  `update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `is_deleted` int(11) DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

## 鉴权架构设计

1）**ic-club-auth**：这个服务承载了我们所有的基础数据源。他不管鉴权，只管数据相关的持久化操作以及业务操作，提供出各种各样的权限相关的接口。
	采用 DDD 架构

2）**nacos**：将 auth 服务以及 subiect 服务都注册到上面。内部进行调用，不对外暴露。通过 nacos 实现我们的服务发现。

3）**gateway**：网关层会对外提供服务，内部实现路由，==鉴权==。整体我们采取 token 的方式来与前端进行交互。由网关来决定当前用户是否可以操作到后面的业务逻辑。

好处：不放在网关，导致我们的每个微服务，全要引入的鉴权的框架，不断的去写重复的代码

数据的权限获取产生问题：
- 1、网关直接对接数据库，实现查询
- 2、redis 中获取数据，获取不到的时候还是要像第一种一样去数据库里查,
- 3、redis 中获取缓存，没有的话，从 auth 服务里面获取相关的信息。

## 鉴权功能设计

### 用户验证

- **短信的方式**，通过向手机号发送验证码，来实现用户的验证并登录
	- （考虑的成本是短信的费用）

- **邮箱的注册登录**。用户注册的时候，留一个邮箱，我们往邮箱里通过邮箱服务器发送一个链接，用户点击之后，实现一个激激活成功之后就完成了注册。
	- 0 成本，坏处是这种发送的邮件很容易进垃圾箱

- **个人公众号模式**：用户登录的时候，弹出我们的这个公众号的码。扫码后，用户输入我们提示的验证码。可以随机比如说 nadbuge，通过我们的公众号对接的回调。能拿到一定的信息，用户的 openld。进而记录用户的信息
	- 个人开发者无公司的，比较适合使用，0 成本

- **企业的服务号**：必须要有营业执照，自己玩的不上线的话，也可以用测试号
	- 好处就是不仅打通了各种回调，而且还能拿到用户的信息。

### 登录功能

传统的 pc 形式，都是登录之后，写入 cookie。前端再次请求的时候，带着 cookie 一个身份识别就可以完成人证。坏处是什么?小程序呀，app 呀，其实是没有 cookie 这个概念的。为了更好的扩展，我们就直接选择 token 的模式。token 放入 header 来实现用户身份的识别与鉴权。

### 集成 redis

如果说我们选择了 token，然后不做 token 的保存，服务重启呀，分布式微服务啊，数据是无法共享并且会产生丢失问题，所以用 redis 来存储一些信息，实现共享。

### 踢人下线

发现风险用户，可以通过后台直接把用户踢掉，禁止其再访问，token 也可以直接置为失效的形式。

## 基于 Redis 实现分布式 Session

## 用户注册

密码加密加盐

加盐：摘要算法比如 md5，光加密 123456，结果都是一样的，如果是破解的库里正好有这个 md5 就很容易知道》面是 123456。来一手加盐。盐是随机的字符串，他来与原密码进行一波二次加密。这样获取到的很难破解出来。


# gateway 模块

## 路由配置

## 全局异常处理

## 配合 Redis 实现鉴权

auth 服务中，添加/删除/更新 用户的角色和权限 会同步到 Redis 中

网关读取角色和权限时完全信任缓存



# 部署 #todo

## 购买云服务器

## 安装

Docker，MySQL

## 购买域名


## 传统打包

## Jenkins 自动打包

## Jenkins 配合 shell 实现自动部署




