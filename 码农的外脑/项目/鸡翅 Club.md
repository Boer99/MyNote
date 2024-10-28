项目地址：[Boer99/jc-club (github.com)](https://github.com/Boer99/jc-club)

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

标签和==一级分类==绑定
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

这张表记录了题目==二级分类==、标签与题目的mapping关系
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

### 点赞表

```sql
CREATE TABLE `subject_liked`
(
    `id`           bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `subject_id`   bigint(20) DEFAULT NULL COMMENT '题目id',
    `like_user_id` varchar(32) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '点赞人id',
    `status`       int(11) DEFAULT NULL COMMENT '点赞状态 1点赞 0不点赞',
    `created_by`   varchar(32) CHARACTER SET utf8  DEFAULT NULL COMMENT '创建人',
    `created_time` datetime                        DEFAULT NULL COMMENT '创建时间',
    `update_by`    varchar(32) CHARACTER SET utf8  DEFAULT NULL COMMENT '修改人',
    `update_time`  datetime                        DEFAULT NULL COMMENT '修改时间',
    `is_deleted`   int(11) DEFAULT '0',
    PRIMARY KEY (`id`),
    UNIQUE KEY `uniq_like` (`subject_id`,`like_user_id`) USING BTREE COMMENT '点赞唯一索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='题目点赞表';
```


## 新增题目（Spring 工厂+策略模式）

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


## 题目详情（Spring 工厂+策略模式）

查标签，难度啊，出题人啊，等等，这些就直接查，不做 join。


## 多线程优化分类和标签查询 #todo 

![600](../assets/Pasted%20image%2020241010125250.png)

FutureTask，CompetableFuture，自定义线程池工厂

### 线程池设置多少数量？ #todo 


## guava 本地缓存优化分类和标签查询 #todo


## 题目贡献排行榜

根据创建题目数量对用户排名

选择 redis 的 sorted set 来实现

###  实时方案

1）数据库统计

现在数据库里面的 createby 字段。用户的标识是唯一的，直接通过 group by 的形式统计 count

数据量比较小，并发也比较小。这种方案是 ok 的。保证可以走到索引，返回速度快，不要产生慢 sql

2）redis 的 sorted set

有序集合，不允许重复的成员，然后每一个 key 都会包含一个 score 分数的概念。redis 根据分数可以帮助我们做从小到大，和从大到小的一个处理。有序集合的 key 不可重复，score 重复。它通过我们的一个哈希表来实现的，添加，删除，查找，复杂度 $O(1)$，最大数量是 $2^{32}-1$

这种的好处在于，完全不用和数据库做任何的交互，纯纯的通过缓存来做，速度非常快，要避免一些大 key 的问题。

### 非实时方案

定时任务 xxl-job

统计数据库的数据形式，帮助我们统计完成后，直接写入缓存。缓存的外部的交互展示。


## 题目点赞

需求：获取题目的点赞数，当前用户是否点赞

给一个题目点赞后，在 Redis 中分三个部分保存

1. **hash**: key=subject.liked, hkey=[subjectId].[loginId], hvalue=[是否点赞 0/1]

2. **String**: key=subject.liked.count.[subjectId], value=[点赞数]

3. **String**: key=subject.liked.detail.[subjectId].[loginId]

第一个 hash 结构是用来做转存数据库的，不能用 set，因为转存的时候不知道哪些取消点赞了

第二个用来记录题目点赞数，第三个用来记录用户是否点赞

> 感觉后面两个设计的不好，应该用 set 来实现

### 点赞数据从 Redis 同步到数据库

使用 xxl-job 

![](../assets/Pasted%20image%2020241012130154.png)

```java
public void syncLiked() {
	Map<Object, Object> subjectLikedMap = redisUtil.getHashAndDelete(SUBJECT_LIKED_KEY);
	if (log.isInfoEnabled()) {
		log.info("syncLiked.subjectLikedMap:{}", JSON.toJSONString(subjectLikedMap));
	}
	if (MapUtils.isEmpty(subjectLikedMap)) {
		return;
	}
	//批量同步到数据库
	List<SubjectLiked> subjectLikedList = new LinkedList<>();
	subjectLikedMap.forEach((key, val) -> {
		SubjectLiked subjectLiked = new SubjectLiked();
		String[] keyArr = key.toString().split(":");
		String subjectId = keyArr[0];
		String likedUser = keyArr[1];
		subjectLiked.setSubjectId(Long.valueOf(subjectId));
		subjectLiked.setLikeUserId(likedUser);
		subjectLiked.setStatus(Integer.valueOf(val.toString()));
		subjectLikedList.add(subjectLiked);
	});
	subjectLikedService.batchInsert(subjectLikedList);
}
```

```java
@Component
@Slf4j
public class RedisUtil {

    @Resource
    private RedisTemplate redisTemplate;

    private static final String CACHE_KEY_SEPARATOR = ".";

    /**
     * 构建缓存key
     */
    public String buildKey(String... strObjs) {
        return Stream.of(strObjs).collect(Collectors.joining(CACHE_KEY_SEPARATOR));
    }

    public Map<Object, Object> getHashAndDelete(String key) {
        Map<Object, Object> map = new HashMap<>();
        Cursor<Map.Entry<Object, Object>> cursor = redisTemplate.opsForHash().scan(key, ScanOptions.NONE);
        while (cursor.hasNext()) {
            Map.Entry<Object, Object> entry = cursor.next();
            Object hashKey = entry.getKey();
            Object value = entry.getValue();
            map.put(hashKey, value);
            redisTemplate.opsForHash().delete(key, hashKey);
        }
        return map;
    }
}
```

### 我的点赞

从数据库点赞表中做分页条件查询

### 待优化 #todo 

用户对同一个题目的点赞没有任何逻辑处理的，数据库里对 subjectId 和 like_user_id 做了联合唯一索引，重复插入就会报错

```
### SQL: insert into subject_liked(id, subject_id, like_user_id, status, created_by, created_time, update_by,         update_time, is_deleted)         values                        (?,             ?,             ?,             ?,             ?,             ?,             ?, ?, ?)
### Cause: java.sql.SQLIntegrityConstraintViolationException: Duplicate entry '115-oexP_6UGOlXIIB_Ld61JwW5CrH3Q' for key 'uniq_like'
; Duplicate entry '115-oexP_6UGOlXIIB_Ld61JwW5CrH3Q' for key 'uniq_like'; nested exception is java.sql.SQLIntegrityConstraintViolationException: Duplicate entry '115-oexP_6UGOlXIIB_Ld61JwW5CrH3Q' for key 'uniq_like'
```


## 上一题下一题

新增属性
```java
@Data
public class SubjectInfoBO extends PageInfo implements Serializable {
    /**
     * 下一题
     */
    private Long nextSubjectId;

    /**
     * 上一题
     */
    private Long lastSubjectId;
}
```

sql 写法（分页）
```sql
<select id="querySubjectIdCursor" resultType="java.lang.Long">
	select a.id
	from subject_info a,
	subject_mapping b
	where a.id = b.subject_id
	and b.category_id = #{categoryId}
	and b.label_id = #{labelId}
	<if test="cursor !=null and cursor == 1">
		and a.id > #{subjectId}
	</if>
	<if test="cursor !=null and cursor == 0">
		and a.id &lt; #{subjectId}
	</if>
	limit 0,1
</select>
```


# 认证与鉴权

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


## 架构设计

1）**jc-club-auth**：注册和登录，用户、角色、权限的 CRUD

2）**jc-club-gateway**：认证与鉴权

好处：不放在网关，导致我们的每个微服务，全要引入的鉴权的框架，不断的去写重复的代码

3）**jc-club-wx**：发送验证码

---
Redis：完全信赖缓存，Sa-Token 默认把 token 存 Redis 中，角色、权限、验证码由我们手动存入


## 用户验证方式选择

- **短信的方式**，通过向手机号发送验证码，来实现用户的验证并登录
	- （考虑的成本是短信的费用）

- **邮箱的注册登录**。用户注册的时候，留一个邮箱，我们往邮箱里通过邮箱服务器发送一个链接，用户点击之后，实现一个激激活成功之后就完成了注册。
	- 0 成本，坏处是这种发送的邮件很容易进垃圾箱

- **个人公众号模式**：用户登录的时候，弹出我们的这个公众号的码。扫码后，用户输入我们提示的验证码。可以随机比如说 nadbuge，通过我们的公众号对接的回调。能拿到一定的信息，用户的 openld。进而记录用户的信息
	- 个人开发者无公司的，比较适合使用，0 成本

- **企业的服务号**：必须要有营业执照，自己玩的不上线的话，也可以用测试号
	- 好处就是不仅打通了各种回调，而且还能拿到用户的信息。


## 整体流程

1）用户扫公众号码

微信公众号收到用户发的“验证码”，回调 wx 服务。wx 服务回复一个随机的验证码，并存入 redis（key：验证码，value：openId）

核心代码：
```java
public String dealMsg(Map<String, String> messageMap) {
	log.info("接收到文本消息事件");
	
	String content = messageMap.get("Content");
	if (!KEY_WORD.equals(content)) {
		return "";
	}
	
	String fromUserName = messageMap.get("FromUserName");
	String toUserName = messageMap.get("ToUserName");
	// 生成随机验证码
	Random random = new Random();
	int num = random.nextInt(1000);
	// 存入Redis
	String numKey = redisUtil.buildKey(LOGIN_PREFIX, String.valueOf(num));
	redisUtil.setNx(numKey, fromUserName, 5L, TimeUnit.MINUTES);
	// 回复
	String numContent = "您当前的验证码是：" + num + "！ 5分钟内有效";
	String replyContent = "<xml>\n" +
			"  <ToUserName><![CDATA[" + fromUserName + "]]></ToUserName>\n" +
			"  <FromUserName><![CDATA[" + toUserName + "]]></FromUserName>\n" +
			"  <CreateTime>12345678</CreateTime>\n" +
			"  <MsgType><![CDATA[text]]></MsgType>\n" +
			"  <Content><![CDATA[" + numContent + "]]></Content>\n" +
			"</xml>";

	return replyContent;
}
```

2）用户输入验证码，点击登录

auth 服务根据验证码获取到 openId；如果用户没有注册过，自动注册一个用户，并为其关联普通用户角色，将角色（role_key）和权限（permission_key）存入 Redis

通过 sa-token 完成用户登录，sa-token 会将 token 存入 Redis，并返回给前端

> openId 作为 userName 字段存放在数据库中

```java
@Override
public SaTokenInfo doLogin(String validCode) {
	String loginKey = redisUtil.buildKey(LOGIN_PREFIX, validCode);
	String openId = redisUtil.get(loginKey);
	if (StringUtils.isBlank(openId)) {
		return null;
	}
	AuthUserBO authUserBO = new AuthUserBO();
	authUserBO.setUserName(openId);
	this.register(authUserBO);
	StpUtil.login(openId);
	SaTokenInfo tokenInfo = StpUtil.getTokenInfo();
	return tokenInfo;
}

@Override  
@SneakyThrows  
@Transactional(rollbackFor = Exception.class)  
public Boolean register(AuthUserBO authUserBO) {  
    //校验用户是否存在  
    AuthUser existAuthUser = new AuthUser();  
    existAuthUser.setUserName(authUserBO.getUserName());  
    List<AuthUser> existUser = authUserService.queryByCondition(existAuthUser);  
    if (existUser.size() > 0) {  
        return true;  
    }  
    AuthUser authUser = AuthUserBOConverter.INSTANCE.convertBOToEntity(authUserBO);  
    if (StringUtils.isNotBlank(authUser.getPassword())) {  
        authUser.setPassword(SaSecureUtil.md5BySalt(authUser.getPassword(), salt));  
    }  
    authUser.setStatus(AuthUserStatusEnum.OPEN.getCode());  
    authUser.setIsDeleted(IsDeletedFlagEnum.UN_DELETED.getCode());  
    Integer count = authUserService.insert(authUser);  
  
    //建立一个初步的角色的关联  
    AuthRole authRole = new AuthRole();  
    authRole.setRoleKey(AuthConstant.NORMAL_USER);  
    AuthRole roleResult = authRoleService.queryByCondition(authRole);  
    Long roleId = roleResult.getId();  
    Long userId = authUser.getId();  
    AuthUserRole authUserRole = new AuthUserRole();  
    authUserRole.setUserId(userId);  
    authUserRole.setRoleId(roleId);  
    authUserRole.setIsDeleted(IsDeletedFlagEnum.UN_DELETED.getCode());  
    authUserRoleService.insert(authUserRole);  
  
    String roleKey = redisUtil.buildKey(authRolePrefix, authUser.getUserName());  
    List<AuthRole> roleList = new LinkedList<>();  
    roleList.add(authRole);  
    redisUtil.set(roleKey, new Gson().toJson(roleList));  
  
    AuthRolePermission authRolePermission = new AuthRolePermission();  
    authRolePermission.setRoleId(roleId);  
    List<AuthRolePermission> rolePermissionList = authRolePermissionService.  
            queryByCondition(authRolePermission);  
  
    List<Long> permissionIdList = rolePermissionList.stream()  
            .map(AuthRolePermission::getPermissionId).collect(Collectors.toList());  
    //根据roleId查权限  
    List<AuthPermission> permissionList = authPermissionService.queryByRoleList(permissionIdList);  
    String permissionKey = redisUtil.buildKey(authPermissionPrefix, authUser.getUserName());  
    redisUtil.set(permissionKey, new Gson().toJson(permissionList));  
  
    return count > 0;  
}
```

> 这里好像有事物失效的 bug

![](../assets/Pasted%20image%2020241011025711.png)

3）用户登录成功之后，前端的所有请求都带着 token

网关统一认证和鉴权

配置 sa-token 的鉴权
```java
@Configuration
public class SaTokenConfigure {
    @Bean
    public SaReactorFilter getSaReactorFilter() {
        return new SaReactorFilter()
                // 拦截地址
                .addInclude("/**")
                // 鉴权方法：每次访问进入
                .setAuth(obj -> {
                    System.out.println("-------- 前端访问path：" + SaHolder.getRequest().getRequestPath());
                    // 登录校验 -- 拦截所有路由，并排除/user/doLogin 用于开放登录
//                    SaRouter.match("/auth/**", "/auth/user/doLogin", r -> StpUtil.checkRole("admin"));
                    SaRouter.match("/oss/**", r -> StpUtil.checkLogin());
                    SaRouter.match("/subject/subject/add", r -> StpUtil.checkPermission("subject:add"));
                    SaRouter.match("/subject/**", r -> StpUtil.checkLogin());
                })
                ;
    }
}
```

实现 sa-token 提供的接口，从 Redis 中获取用户的角色和权限
```java
@Component
public class StpInterfaceImpl implements StpInterface {
    @Resource
    private RedisUtil redisUtil;

    private String authPermissionPrefix = "auth.permission";

    private String authRolePrefix = "auth.role";

    @Override
    public List<String> getPermissionList(Object loginId, String loginType) {
        return getAuth(loginId.toString(), authPermissionPrefix);
    }

    @Override
    public List<String> getRoleList(Object loginId, String loginType) {
        return getAuth(loginId.toString(), authRolePrefix);
    }

    private List<String> getAuth(String loginId, String prefix) {
        String authKey = redisUtil.buildKey(prefix, loginId.toString());
        String authValue = redisUtil.get(authKey);
        if (StringUtils.isBlank(authValue)) {
            return Collections.emptyList();
        }
        List<String> authList = new LinkedList<>();
        if (authRolePrefix.equals(prefix)) {
            List<AuthRole> roleList = new Gson().fromJson(authValue, new TypeToken<List<AuthRole>>() {
            }.getType());
            authList = roleList.stream().map(AuthRole::getRoleKey).collect(Collectors.toList());
        } else if (authPermissionPrefix.equals(prefix)) {
            List<AuthPermission> permissionList = new Gson().fromJson(authValue, new TypeToken<List<AuthPermission>>() {
            }.getType());
            authList = permissionList.stream().map(AuthPermission::getPermissionKey).collect(Collectors.toList());
        }
        return authList;
    }
}
```


## 用户上下文打通 #todo 

网关转发，feign 调用


# Auth 模块

## 用户注册

密码加密加盐

加盐：摘要算法比如 md5，光加密 123456，结果都是一样的，如果是破解的库里正好有这个 md5 就很容易知道》面是 123456。来一手加盐。盐是随机的字符串，他来与原密码进行一波二次加密。这样获取到的很难破解出来。


# Gateway 模块

## 路由配置

## 全局异常处理

## 配合 Redis 实现鉴权

auth 服务中，添加/删除/更新 用户的角色和权限 会同步到 Redis 中

网关读取角色和权限时完全信任缓存


# WX 模块

## 接口测试号申请

[开始开发 / 接口测试号申请 (qq.com)](https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Requesting_an_API_Test_Account.html)

## 验证消息

[开始开发 / 接入指南 (qq.com)](https://developers.weixin.qq.com/doc/offiaccount/Basic_Information/Access_Overview.html#%E7%AC%AC%E4%BA%8C%E6%AD%A5%EF%BC%9A%E9%AA%8C%E8%AF%81%E6%B6%88%E6%81%AF%E7%9A%84%E7%A1%AE%E6%9D%A5%E8%87%AA%E5%BE%AE%E4%BF%A1%E6%9C%8D%E5%8A%A1%E5%99%A8)

![](../assets/Pasted%20image%2020241009133607.png)

```java
@RestController
@Slf4j
public class CallBackController {

    private static final String token = "adwidhaidwoaid";

    /**
     * 回调消息校验
     */
    @GetMapping("callback")
    public String callback(@RequestParam("signature") String signature,
                           @RequestParam("timestamp") String timestamp,
                           @RequestParam("nonce") String nonce,
                           @RequestParam("echostr") String echostr) {
        log.info("get验签请求参数：signature:{}，timestamp:{}，nonce:{}，echostr:{}",
                signature, timestamp, nonce, echostr);
        String shaStr = SHA1.getSHA1(token, timestamp, nonce, "");
        if (signature.equals(shaStr)) {
            return echostr;
        }
        return "unknown";
    }
}
```

## 收发消息（Spring 工厂+策略）

接收和发送的消息都是 xml 格式

![](../assets/Pasted%20image%2020241009160232.png)

针对用户不同的动作（订阅、文本），设计不同的策略类，返回不同的消息给用户

创建策略类的对象依旧可以用 Spring+工厂模式的方式



# OSS 模块


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


# 练题模块

## 库表设计

套卷表
```sql
CREATE TABLE `practice_set`
(
    `id`                  bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `set_name`            varchar(255) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '套题名称',
    `set_type`            int(11) DEFAULT NULL COMMENT '套题类型 1实时生成 2预设套题',
    `set_heat`            int(11) DEFAULT NULL COMMENT '热度',
    `set_desc`            varchar(255) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '套题描述',
    `primary_category_id` bigint(20) DEFAULT NULL COMMENT '大类id',
    
    `created_by`          varchar(32) CHARACTER SET utf8   DEFAULT NULL COMMENT '创建人',
    `created_time`        datetime                         DEFAULT NULL COMMENT '创建时间',
    `update_by`           varchar(32) CHARACTER SET utf8   DEFAULT NULL COMMENT '更新人',
    `update_time`         datetime                         DEFAULT NULL COMMENT '更新时间',
    `is_deleted`          int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',
    PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='套题信息表';
```

套卷题目表
```sql
CREATE TABLE `practice_set_detail`
(
    `id`           bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `set_id`       bigint(20) NOT NULL COMMENT '套题id',
    `subject_id`   bigint(20) DEFAULT NULL COMMENT '题目id',
    `subject_type` int(11) DEFAULT NULL COMMENT '题目类型',
    
    `created_by`   varchar(32) CHARACTER SET utf8 DEFAULT NULL COMMENT '创建人',
    `created_time` datetime                       DEFAULT NULL COMMENT '创建时间',
    `update_by`    varchar(32) CHARACTER SET utf8 DEFAULT NULL COMMENT '更新人',
    `update_time`  datetime                       DEFAULT NULL COMMENT '更新时间',
    `is_deleted`   int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='套题内容表';
```

练习表（练习一份套卷）
```sql
CREATE TABLE `practice_info`
(
    `id`              bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `set_id`          bigint(20) DEFAULT NULL COMMENT '套题id',
    `complete_status` int(11) DEFAULT NULL COMMENT '是否完成 1完成 0未完成',
    `time_use`        varchar(32) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '用时',
    `submit_time`     datetime                        DEFAULT NULL COMMENT '交卷时间',
    `correct_rate`    decimal(10, 2)                  DEFAULT NULL COMMENT '正确率',
    
    `created_by`      varchar(32) CHARACTER SET utf8  DEFAULT NULL COMMENT '创建人',
    `created_time`    datetime                        DEFAULT NULL COMMENT '创建时间',
    `update_by`       varchar(32) CHARACTER SET utf8  DEFAULT NULL COMMENT '更新人',
    `update_time`     datetime                        DEFAULT NULL COMMENT '更新时间',
    `is_deleted`      int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='练习表';
```

练习明细表（用户每做一题都会保存记录，用户交卷的时候其实答题记录都已经保存好了）
```sql
CREATE TABLE `practice_detail`
(
    `id`             bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
    `practice_id`    bigint(20) DEFAULT NULL COMMENT '练题id',
    `subject_id`     bigint(20) DEFAULT NULL COMMENT '题目id',
    `subject_type`   int(11) DEFAULT NULL COMMENT '题目类型',
    `answer_status`  int(11) DEFAULT NULL COMMENT '回答状态，1-正确',
    `answer_content` varchar(64) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '回答内容',
    
    `created_by`     varchar(32) CHARACTER SET utf8  DEFAULT NULL COMMENT '创建人',
    `created_time`   datetime                        DEFAULT NULL COMMENT '创建时间',
    `update_by`      varchar(32) CHARACTER SET utf8  DEFAULT NULL COMMENT '更新人',
    `update_time`    datetime                        DEFAULT NULL COMMENT '更新时间',
    `is_deleted`     int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',
    PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='练习详情表';
```

## 架构设计

企业里的三层架构模式

![](../assets/Pasted%20image%2020241013175350.png)

## 专项练习-新建套卷

整体流程
1. 找到用户选择标签下面的题目，通过动态配置的设定，来去进行题目的选择。比如说选择 10 套，多选 6 道，判断 4 道。
2. 存入我们的 practice set（套卷）, practice set detail（套卷明细）
3. 要根据选择的标签（assembleIds，“二级分类”-“标签”）来做一波默认的套卷的名称。

获取套卷题目的 sql。
- 通过 `order by rand()` 随机获取。可优化为已做过的题不再生成
- 通过 `CONCAT(a.category_id,'-',a.label_id)` 筛选用户选择的标签
```sql
<select id="getPracticeSubject"
		resultType="com.jingdianjichi.practice.server.entity.po.SubjectPO">
	select distinct a.subject_id as id,
	b.subject_type as subjectType
	from subject_mapping a,
	subject_info b
	where a.subject_id = b.id
	<if test="subjectType!= null">
		and b.subject_type = #{subjectType, jdbcType= INTEGER}
	</if>
	<if test="excludeSubjectIds!= null and excludeSubjectIds.size()>0">
		and a.subject_id not in
		<foreach collection="excludeSubjectIds" item="item" open="(" close=")" separator=",">
			#{item}
		</foreach>
	</if>
	and CONCAT(a.category_id,'-',a.label_id) in
	<foreach collection="assembleIds" item="item" open="(" close=")" separator=",">
		#{item}
	</foreach>
	order by rand() LIMIT #{subjectCount, jdbcType= INTEGER};
</select>
```

## 专项练习-交卷

整体流程：
- 保存、交卷时间、用时、完成状态
- 计算正确率
- 套题热度+1
- 补充没有答题的记录到 practice_detail

计算正确率核心 sql，其实就是比对答案计算正确题数
```sql
<select id="selectCorrectCount" resultType="java.lang.Integer">
	select count(1)
	from practice_detail
	where is_deleted = 0
	  and answer_status = 1
	  and practice_id = #{practiceId,jdbcType=BIGINT}
</select>
```

## 专项练习-保存答题记录

## 答题情况-答题卡

## 答题情况-每题情况


# 鸡圈模块

## 库表设计

圈子信息表
```sql
CREATE TABLE `share_circle`  
(  
    `id`           bigint(20) NOT NULL AUTO_INCREMENT COMMENT '圈子ID',  
    `parent_id`    bigint(20) NOT NULL COMMENT '父级ID,-1为大类',  
    `circle_name`  varchar(16) NOT NULL COMMENT '圈子名称', 
     
    `icon`         varchar(255) DEFAULT NULL COMMENT '圈子图片',  
    `created_by`   varchar(32)  DEFAULT NULL COMMENT '创建人',  
    `created_time` datetime     DEFAULT NULL COMMENT '创建时间',  
    `update_by`    varchar(32)  DEFAULT NULL COMMENT '更新人',  
    `update_time`  datetime     DEFAULT NULL COMMENT '更新时间',  
    `is_deleted`   int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',  
    PRIMARY KEY (`id`),  
    KEY            `idx_parent_id` (`parent_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='圈子信息';
```

评论及回复信息
```sql
CREATE TABLE `share_comment_reply`
(
    `id`             bigint(20) NOT NULL AUTO_INCREMENT COMMENT '评论ID',
    `moment_id`      int(11) NOT NULL COMMENT '原始动态ID',
    `reply_type`     int(11) NOT NULL COMMENT '回复类型 1评论 2回复',
    `to_id`          bigint(20) DEFAULT NULL COMMENT '评论目标id',
    `to_user`        varchar(32)   DEFAULT NULL COMMENT '评论人',
    `to_user_author` int(11) DEFAULT NULL COMMENT '评论人是否作者 1=是 0=否',
    `reply_id`       bigint(20) DEFAULT NULL COMMENT '回复目标id',
    `reply_user`     varchar(32)   DEFAULT NULL COMMENT '回复人',
    `replay_author`  int(11) DEFAULT NULL COMMENT '回复人是否作者 1=是 0=否',
    `content`        varchar(1024) DEFAULT NULL COMMENT '内容',
    
    `pic_urls`       varchar(1024) DEFAULT NULL COMMENT '图片内容',
    `created_by`     varchar(32)   DEFAULT NULL COMMENT '创建人',
    `created_time`   datetime      DEFAULT NULL COMMENT '创建时间',
    `update_by`      varchar(32)   DEFAULT NULL COMMENT '更新人',
    `update_time`    datetime      DEFAULT NULL COMMENT '更新时间',
    `is_deleted`     int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',
    `parent_id`      bigint(20) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY              `idx_moment_id` (`moment_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='评论及回复信息';
```

```sql
CREATE TABLE `share_message`  
(  
    `id`           int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',  
    `from_id`      varchar(32) NOT NULL COMMENT '来自人',  
    `to_id`        varchar(32) NOT NULL COMMENT '送达人',  
    `content`      varchar(256) DEFAULT NULL COMMENT '消息内容',  
    `is_read`      int(11) DEFAULT '0' COMMENT '是否被阅读 1是 2否',  
    `created_by`   varchar(32)  DEFAULT NULL COMMENT '创建人',  
    `created_time` datetime     DEFAULT NULL COMMENT '创建时间',  
    `update_by`    varchar(32)  DEFAULT NULL COMMENT '更新人',  
    `update_time`  datetime     DEFAULT NULL COMMENT '更新时间',  
    `is_deleted`   int(11) DEFAULT '0' COMMENT '是否被删除 0为删除 1已删除',  
    PRIMARY KEY (`id`),  
    KEY            `idx_to_id` (`to_id`)  
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='消息表';
```

# 全文检索 #todo

## 技术选型

elasticsearch

## 检索形式

三种方式：

- 同步的实现方式：新增题目->MYSQL->es
- 异步 mq 的实现方式：mysql 存储完后，发送 mq。
- 异步 canal 方式：监听 mysql 变更的 binlog，实现 es 的存储。

先做同步的实现。





# 部署

## 购买云服务器

京东轻量云服务器

## 安装

Docker，MySQL

## 购买域名 #todo


## 传统部署

### 打包

pom 配置打包插件
```xml
 <build>
    <finalName>${project.artifactId}</finalName>
	<!--打包成jar包时的名字-->
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<version>2.3.0.RELEASE</version>
			<executions>
				<execution>
					<goals>
						<goal>repackage</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

maven clean install

把 jar 包复制到服务器上

服务器环境安装 JDK8，运行 jar 包（不要忘了开发安全组端口）

### 部署

[Linux nohup后台启动/ 后台启动命令中nohup 、&、重定向的使用-CSDN博客](https://blog.csdn.net/weixin_49114503/article/details/134266408)

```java
[root@lavm-mzsinjda6q home]# nohup java -jar -Xmx256m jc-club-wx.jar > wx.log &
[1] 29756
[root@lavm-mzsinjda6q home]# nohup: ignoring input and redirecting stderr to stdout
```


## Jenkins 自动打包 #todo

内存不够

## Jenkins 配合 shell 实现自动部署 #todo



# 前端

## 安装和运行

安装 pnpm
```
npm install pnpm

pnpm config set registry https://registry.npmmirror.com/

pnpm install
```

运行
```bash
npm run dev
```

## 开发

`.env.development` 文件里配置后端地址

## 打包

```bash
npm run build
```

打包完会生成一个 dist 目录，放到 nginx 容器的 html 挂载目录里

## nginx

docker 安装 nginx
```bash
# 一般前端项目都会用80端口，默认端口是不需要显式地指定的
docker run \
-p 80:80 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest
```

配置 nginx.conf
```conf
http {
    server {
        listen       80;
        server_name  116.198.253.55;

        #charset utf-8;
        #access_log  logs/localhost.access.log  access;

        location / {
            root   /usr/share/nginx/html/dist;
            index  index.html index.htm;
            try_files $uri $uri/ /index.html;
        }

		location ~ /auth/ {
			proxy_pass http://116.198.253.55:5000;
		}
		location ~ /subject/ {
			proxy_pass http://116.198.253.55:5000;
		}
    } 
}
```


# 简历

熟悉 Rocketmq 的使用，掌握持久化机制，消息可靠性，延迟消费等，解决过消息积压，消息逆序等问题,

## 项目描述

鸡翅 Club 是一款专门为程序员打造的沟通交流社区，采用主流的微服务框架+主流 C 端技术栈来做为技术架构。旨在统一程序员信息差，进行平台统一化，程序员可以在平台，完善自身知识，刷自身薄弱点面试题，配合练习，模拟面试，简历分析模块来提升程序员面试能力。

## 技术架构

SpringBoot+SpringCloud Alibaba+SSM+Mysql+Redis+Nacos+Gateway+Minio

开源地址：xxx

## 工作职责

一期：
- ~~独立从 0 到 1 负责项目的架构设计，技术选型，功能设计，数据建，调研用户常用业务场景，~~
- 采用微服务领域拆分思想，对项目模块进行领域设计，划分为 4 个微服务，业务解，专注自身职责，
- 基于 Nacos 来实现业务项目的服务注册与发现及业务动态配置切换;
- 选取主流鉴权框架 Satoken 来替代传统的 secruity，提高开发效率，降低上手难度
- 采用 Gateway 配合 redis 实现统一的鉴权及分布式会话共享功能，在网关层实现统一的全局异常处理
- 为了解决原有部署多机器拖拽 jar 包的痛点，采用 Jenkins 配合 shell 脚本实现多机器自动部署;
- 整体项目中间件采取 Docker 形式进行容器化搭建，配合数据挂载实现重要数据抽离;
- ~~采用元数建模配合 easycode 实现模型搭建及代码自动生成，提升原有建模效率;~~
- 登录模块抽取微信微服务，实现微信的对接回调与 sdk 的统一封装，沉淀出无业务性的微信对接服务
- 重构原有复杂代码，采取工厂+策略模式实现微信的消息解耦处理，采取适配器模式实现 oss 对接;
- ~~独立从 0 到 1 通过云服务器搭建整体项目的环境及各依赖的安装;~~

二期：
- 基于 futuretask 及 completablefuture 实现了分类标签的并发查询，提升性能 80%。
- ~~封装了自定义的线程工厂，实现了线程池间的日志区分，提升了日志排查效率。~~
- ~~协同测试同学进行了线程池数据压测，确定出合理线程池数量，探索线程池与 cpu 关系公式~~
- 使用 threadlocal 配合网关拦截器，feign 拦截器，封装用户上下文全局工具。
- 针对高并发接口，采取了 guava 本地缓存配合函数式编程，泛型封装本地缓存工具，提升性能及通用性;
- 封装自定义的 esclient，支持多集群，多索引切换，封装了常用的业务函数，实现网站的高亮搜索功能,
- 基于 redis 的 zset 实现实时排行榜功能，解决传统数据库大量交互的瓶颈点;
- ~~提高项目开发效率，开发代码生成器组件，生成从 controller 到 dao 的整套基础代码，提高开发效率;~~
- 选取 xxl-job 配合 redis 的 hash 接口实现点赞收藏功能的开发及数据持久化，减轻数据库交互压力;

三期：
- ~~设计优化原有三层架构，拆分为 api 与 server 模块，职责清晰，对外与对内解耦~~
- 负责练题模块的从 0 到 1 落地实现，从需求分析、功能设计到数据建模，交付练题功能
- 使用 rocketmq，优化原有点赞功能，解决了 redis 存储点赞可能丢数据的问题
- 优化多线程相关代码，引入 ansyctool 线程编排，实现线程代码解耦和执行顺序编排


四期：
1、设计优化原有三层架构，拆分为 api 与 server 模块，职责清晰，对外与对内解耦
2、负责圈子模块的从 0 到 1 落地实现，从需求分析、功能设计到数据建，交付圈子功能
3、识别圈子数据可变性弱，使用 caffine 本地缓存，存储圈子数据，提高了查询性能
4、采用 dfa 算法，封装敏感词工具，实现发布内容时，毫秒级的敏感词检测，支持大量内容
5、根据数据量情况，将评论与回复设计为单表多类型，盖楼采用树工具进行内存方式聚合
6、使用 websocket 实现前端与后端的实时信息推送，扩展心跳机制，连接鉴权等