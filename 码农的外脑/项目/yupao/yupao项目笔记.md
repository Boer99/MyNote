
# 技术栈

前端
1. Vue3 开发框架(提高页面开发的效率)
2. Vant Ul (基于 Vue 的移动端组件库)(React 版 Zent)
3. Vite 2 (打包工具，快! ) 
   1. 明明是3！我不小心下成4了
4. Nginx 来单机部署

后端
1. Java 编程语言 + SpringBoot 框架
2. SpringMVC + MyBatis + MyBatis Plus (提高开发效率）
3. MySQL 数据库
4. Redis 缓存
5. Swagger + Knife4i 接口文档

# 需求分析

1. 用户去添加标签，标签的分类 (要有哪些标签、、怎么把标签进行分类) 学习方向 java /c++，工作 /大学
2. 主动搜索：允许用户根据标签去搜索其他用户
   1. Redis 缓存
3. 组队
   1. 创建队伍
   2. 加入队伍
   3. 根据标签查询队伍
   4. 邀请其他人
4. 允许用户去修改标签
5. 推荐
   1. 相似度计算算法 +本地分布式计算

# 通用问题

## mybatisX插件生成模板代码

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1698395131913-b5150639-9623-42b6-be64-e5cccf06f1ec.png#averageHue=%233d4144&clientId=u49d80f0e-0646-4&from=paste&height=415&id=TR26P&originHeight=578&originWidth=1164&originalType=url&ratio=1.2540000677108765&rotation=0&showTitle=false&status=done&style=none&taskId=udc7804f1-5bb7-4037-910c-747b02162bc&title=&width=835.9940185546875)

## 跨域

origins存储一个数组

```java
@CrossOrigin(origins={"http://127.0.0.1:5173"})
```

坑：别写localhost

## 枚举类

`values()`可以获取一个枚举类中的所有对象

```java
public enum TeamStatusEnum {

    PUBLIC(0, "公开"),
    PRIVATE(1, "私有"),
    SECRET(2, "加密");

    private int value;

    private String text;

    TeamStatusEnum(int value, String text) {
        this.value = value;
        this.text = text;
    }

    public static TeamStatusEnum getEnumByValue(Integer value) {
        if (value == null) {
            return null;
        }
        TeamStatusEnum[] values = TeamStatusEnum.values();
        for (TeamStatusEnum teamStatusEnum : values) {
            if (teamStatusEnum.getValue() == value) {
                return teamStatusEnum;
            }
        }
        return null;
    }

    public int getValue() {
        return value;
    }

    public void setValue(int value) {
        this.value = value;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }
}
```
## Optional字段校验

很优雅~<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1699351954858-e01ad639-69e5-48b3-9934-f5a8b45dd58c.png#averageHue=%23373733&clientId=u23dd3ae5-37de-4&from=paste&height=171&id=u6465c723&originHeight=257&originWidth=762&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=97444&status=done&style=none&taskId=ua7320c6c-1b0f-498d-8798-fad4ec7a7e3&title=&width=508)
# 前端开发

## 前端项目初始化

1）vite快速初始化项目 [Vite](https://cn.vitejs.dev/guide/)

```latex
npm create vite@latest
npm install
```

2）安装vant3 [Vant 3 - Lightweight Mobile UI Components built on Vue](https://vant-contrib.gitee.io/vant/v3/#/zh-CN/quickstart)<br />vant是一个轻量、可靠的移动端组件

> 在基于 **vite**、**webpack** 或 **vue-cli** 的项目中使用 Vant 时，可以使用 [unplugin-vue-components](https://github.com/antfu/unplugin-vue-components) 插件，它可以自动引入组件，并按需引入组件的样式。
> 相比于常规用法，这种方式可以按需引入组件的 CSS 样式，从而减少一部分代码体积，但使用起来会变得繁琐一些。如果业务对 CSS 的体积要求不是特别极致，我们推荐使用更简便的常规用法。

```latex
npm i vant

按需引入
npm i unplugin-vue-components -D

测试一下
<van-button type="primary">主要按钮</van-button>
<van-button type="success">成功按钮</van-button>
<van-button type="default">默认按钮</van-button>
<van-button type="warning">警告按钮</van-button>
<van-button type="danger">危险按钮</van-button>
```
## 页面设计

1）导航条：展示当前页面名称<br />采用NavBar组件<br />2）主页搜索框 => 搜索页 => 搜索结果页(标签选页)<br />3）内容<br />4）tab栏：<br />采用Tabbar组件

- 主页 (推荐页 +广告 )
   - 搜索框
   - banner
   - 推荐信息流
- 队伍页
- 用户页(消息 暂时考虑发邮件)
## 配置路由
### Vue Router
[Vue Router](https://router.vuejs.org/zh/guide/)
```latex
npm install vue-router@4
```
main.ts中配置路由

vant3自带了和vue-router的整合<br />底部tabbar使用路由模式，参照官网
> 标签栏支持路由模式，用于搭配 **vue-router** 使用。路由模式下会匹配页面路径和标签的 **to** 属性，并自动选中对应的标签。

```html
<router-view />

<van-tabbar route>
  <van-tabbar-item replace to="/home" icon="home-o">标签</van-tabbar-item>
  <van-tabbar-item replace to="/search" icon="search">标签</van-tabbar-item>
</van-tabbar>
```
### 前端页面跳转传值（路由传值）

1. query => url searchParams，url 后附加参数，传递的值长度有限
2. vuex(全局状态管理)，搜索页将关键词塞到状态中，搜索结果页从状态取值

## axios
[起步 | Axios中文文档 | Axios中文网](https://www.axios-http.cn/docs/intro)<br />配置
```javascript
import axios from "axios";

const myAxios = axios.create({
    baseURL: 'http://127.0.0.1:8080/api',
    // `withCredentials` 表示跨域请求时是否需要携带凭证，默认false
    withCredentials: true,
});

// 全局配置
// axios.defaults.withCredentials = true

// 添加请求拦截器
myAxios.interceptors.request.use(function (config) {
    // 在发送请求之前做些什么
    return config;
}, function (error) {
    // 对请求错误做些什么
    return Promise.reject(error);
});

// 添加响应拦截器
myAxios.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么，直接把Axios对象里的data取出来，这样then拿到的直接是后端传过来的数据
    console.log('axios response\n', response)
    return response.data;
}, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
});

export default myAxios;

```
axios，params路径传参为数组时需要序列化
```vue
onMounted(() => {
    myAxios.get('/user/search/tags', {
      params: {
        tagNameList: tags
      },
      // tagNameList=xxx & tagNameList=xxx & tagNameList=xxx
      paramsSerializer: params => {
        return qs.stringify(params, {indices: false})
      }
    }).then(function (response) {
      console.log('/user/search/tags succeed', response);
    }).catch(function (error) {
      console.log('/user/search/tags error', error);
    }).finally(function () {
      // 总是会执行
    });
  })
```

## 前端导航栏标题动态变化
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1699930680417-ec8fe8dc-c28a-409c-b382-bbfa6f7f43c2.png#averageHue=%239bd6e8&clientId=u7a4aca4f-8430-4&from=paste&height=562&id=uf0a70aaa&originHeight=1124&originWidth=570&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=263993&status=done&style=none&taskId=u848037c7-eb02-46b7-bc1f-e85bbf05968&title=&width=285)![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1699930722252-ef2dde0b-164d-4a01-b466-ac7e88151f78.png#averageHue=%23fdfdfc&clientId=u7a4aca4f-8430-4&from=paste&height=564&id=uefdfc2d4&originHeight=1127&originWidth=567&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=82413&status=done&style=none&taskId=u8e6bf5db-b5ab-4309-835d-57979adc024&title=&width=284)<br />采用vue-router提供的全局前置路由守卫
```vue
// 全局前置路由守卫————初始化的时候被调用、每次路由切换之前被调用
  router.beforeEach((to, from) => {
    // console.log('前置路由守卫', to, from, next)
    const toPath = to.path
    // todo 不造有没有遍历路由表以外的办法
    const route = routes.find((route) => {
      return toPath == route.path
    })
    title.value = route?.title ?? DEFAULT_TITLE
  })
```
通过遍历路由表找到每个路由对应的标题
```javascript
const routes = [
    {path: '/', component: IndexPage},
    {path: '/team', title: '找队伍', component: TeamPage},
    {path: '/team/add', title: '创建队伍', component: TeamAddPage},
    {path: '/team/update', title: '更新队伍', component: TeamUpdatePage},
    {path: '/user', title: '个人', component: UserPage},
    {path: '/user/edit', title: '编辑个人信息', component: UserEditPage},
    {path: '/user/list', title: '伙伴', component: SearchResultPage},
    {path: '/user/login', title: '登录', component: UserLoginPage},
    {path: '/user/update', title: '个人信息', component: UserUpdatePage},
    {path: '/user/team/create', title: '我创建的队伍', component: UserTeamCreatePage},
    {path: '/user/team/join', title: '我加入的队伍', component: UserTeamJoinPage},
    {path: '/search', title: '找伙伴', component: SearchPage},
]
```

## 强制登录，自动跳转登录页
> 有bug，回退会回到登录页

axios全局配置响应拦截、并且添加重定向
```javascript
// 添加响应拦截器
myAxios.interceptors.response.use(function (response) {
    // 对响应数据做点什么，直接把Axios对象里的data取出来，这样then拿到的直接是后端传过来的数据
    console.log('axios response\n', response)
    // 后端40100是未登录状态码
    if (response?.data?.code === 40100) {
        // 跳转登录页
        // 1 hash模式路由跳转
        // window.location.href = '/#/user/login'
        // 2 history模式路由跳转
        const redirectUrl = window.location.href
        // 将当前页面路径作为参数传递
        window.location.href = `/user/login?redirect=${redirectUrl}`
    }
    return response.data;
}, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    return Promise.reject(error);
});
```
登录页添加重定向
```javascript
const onSubmit = async () => {
  // 请求后端 用户密码登录
  const res = await myAxios.post('/user/login', {
    userAccount: userAccount.value,
    userPassword: userPassword.value
  })
  // console.log('/user/login res',res)
  // 结果判断
  if (res.code === 0 && res.data) {
    showSuccessToast("登录成功")
    // 方式一：路由传参
    // 跳转到之前的页面，没有就回到主页
    console.log('route\n', route)
    const redirectUrl = route.query?.redirect ?? '/'
    window.location.href = redirectUrl
    // 方式二：
    // router.back()
  } else {
    showFailToast("登录失败")
  }
}
```


# 后端接口开发

## 用户搜索标签（可）

### 需求分析

允许用户传入多个标签
- 多个标签都存在才搜索出来
- 有任何一个标签存在就能搜索出来

### 接口设计

标签以作为 user 表的一个字段，json字符串方式存储 `["java","python","c++","男"]`

> 解析 JSON 字符串:
> 1. 序列化：java 对象转成 json
> 2. 反序列化：把 json 转为 java 对象
> 
> java json 序列化库有很多
> 1. gson (google 的)
> 2. fastjson alibaba (ali 出品，快，但是漏洞太多)
> 3. jackson
> 4. kryo

两种查询方式：
- SQL查询（实现简单）
	- `like %Java%' and like%C++%'
	- `like %Java%' or like"%C++%'``
- 内存查询（灵活，可以通过并发进一步优化）

选择依据：
- 如果参数可以分析，根据用户的参数去选择查询方式，比如标签数
- 如果参数不可分析，并且数据库连接足够、内存空间足够，可以并发同时查询，谁先返回用谁.
- 还可以SQL 查询与内存计算相结合，比如先用SQL 过滤掉部分 tag

> 建议通过实际测试来分析哪种查询比较快，数据量大的时候验证效果更明显！

### 库表设计

```sql
create table if not exists yupao.user
(
    username     varchar(256)                       null comment '用户昵称',
    id           bigint auto_increment comment 'id'
        primary key,
    userAccount  varchar(256)                       null comment '账号',
    avatarUrl    varchar(1024)                      null comment '用户头像',
    gender       tinyint                            null comment '性别',
    userPassword varchar(512)                       not null comment '密码',
    phone        varchar(128)                       null comment '电话',
    email        varchar(512)                       null comment '邮箱',
    userStatus   int      default 0                 not null comment '状态 0 - 正常',
    createTime   datetime default CURRENT_TIMESTAMP null comment '创建时间',
    updateTime   datetime default CURRENT_TIMESTAMP null on update CURRENT_TIMESTAMP,
    isDelete     tinyint  default 0                 not null comment '是否删除',
    userRole     int      default 0                 not null comment '用户角色 0 - 普通用户 1 - 管理员',
    planetCode   varchar(512)                       null comment '星球编号',
    tags         varchar(1024)                      null comment '标签列表',
    profile      varchar(256)                       null comment '个人简介'
)
    comment '用户';
```

## 整合 Swagger+Knife4j 接口文档（可）

### 接口文档介绍

什么是接口文档？写接口信息的文档，每条接口包括
- 请求参数
- 响应参数
- 错误码0
- 接口地址
- 接口名称
- 请求类型
- 请求格式
- 备注

who 谁用？一般是后端或者负责人来提供，后端和前端都要使用

为什么需要接口文档?
- 有个书面内容(背书或者归档)，便于大家参考和查阅，便于 沉淀和维护，拒绝口口相传
- 接口文档便于前端和后端开发对接，前后端联调的 介质。后端=> 接口文档<= 前端
- 好的接口文档支持在线调试、在线测试，可以**作为工具提高我们的开发测试效率**

怎么做接口文档?
- 手写(比如腾讯文档、Markdown 笔记)
- 自动化接口文档生成: 自动根据项目代码生成完整的文档或在线调试的网页。SwaggerPostman (侧重接口管理) (国外);apifox、apipost、eolink (国产)

接口文档有哪些技巧?
### 使用

> [1.6 快速开始 | knife4j](https://doc.xiaominfo.com/v2/documentation/get_start.html)
> 
> [解决 高版本SpringBoot整合Swagger 启动报错Failed to start bean ‘documentationPluginsBootstrapper‘ 问题_摸鱼佬的博客-CSDN博客](https://blog.csdn.net/weixin_39792935/article/details/122215625)

访问地址：[http://localhost:8080/api/doc.html#/home](http://localhost:8080/api/doc.html#/home)

线上环境屏蔽（重要）：在配置类上加 `@Profile()` 注解，使其只在部分环境生效

```java
@Profile({"dev","test"}) // 该Bean只在开发、测试环境生效
//@Profile("prod")  // 该Bean只在生产环境生效
public class Knife4jConfiguration
```

`application.yml` 文件中指定当前环境

```yaml
spring:
  profiles:
    active: dev
```

## 存量用户信息导入及同步

爬虫

yupi没实现

## 分布式共享 Session/单点登录

Redis (基于内存的 K/V 数据库) 

此处选择 Redis，因为用户信息读取/是否登录的判极其频繁，Redis 基于内存，读写性能很高，简单的数据单机 qps 5w-10w

> 这张图非常经典，适合理解
> 
> ![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694352259577-e83fc09f-d46f-4d7d-ac3b-90fbd6da6030.png#averageHue=%23f9fcf9&from=url&id=xYp15&originHeight=755&originWidth=1708&originalType=binary&ratio=1.5&rotation=0&showTitle=false&status=done&style=none&title=)


1）安装依赖

```xml
<!--Redis-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

2）修改Session的存储配置，`store-type`默认是none，表示存储在单台服务器

```yaml
  spring:
    # session 失效时间
    session:
      timeout: 86400
      store-type: redis # Session存储在Redis中
```

3）测试。在 8081 另起一个服务，通过打包项目，打包时可以跳过测试

启动：

```bash
java -jar .\yupao-backend-0.0.1-SNAPSHOT.jar --server.port=8081
```

登录后 8080 和 8081 的 swagger 里获取用户都能获得，cookie 中的 jsessionid 都是一样的

## 多线程导入千万用户数据（todo）

```java
@Component
public class InsertUsers {
    @Resource
    private UserMapper userMapper;

    public void doInsertUsers(){
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        final int INSERT_NUM=10000000;
        for (int i = 9; i <INSERT_NUM; i++) {
            User user = new User();
            user.setUsername("user"+i);
            // 省略
            
            userMapper.insert(user);
        }
        stopWatch.stop();
        // 计时
        System.out.println(stopWatch.getTotalTimeMillis());
    }

    public static void main(String[] args) {
        new InsertUsers().doInsertUsers();
    }
}
```

以下的方式执行 main 函数会有如下报错，因为拿不到 UserMapper 实例。

`Exception in thread "main" java.lang.NullPointerException`

---
用“测试类”实现并发批量导入

```java
/**
 * 并发批量插入用户
 */
@Test
public void doConcurrencyInsertUsers() {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // 分十组
    int batchSize = 5000;
    int j = 0;
    List<CompletableFuture<Void>> futureList = new ArrayList<>();
    for (int i = 0; i < 100; i++) {
        List<User> userList = new ArrayList<>();
        while (true) {
            j++;
            User user = new User();
            user.setUsername("假鱼皮");
            user.setUserAccount("fakeyupi");
            user.setAvatarUrl("https://636f-codenav-8grj8px727565176-1256524210.tcb.qcloud.la/img/logo.png");
            user.setGender(0);
            user.setUserPassword("12345678");
            user.setPhone("123");
            user.setEmail("123@qq.com");
            user.setTags("[]");
            user.setUserStatus(0);
            user.setUserRole(0);
            user.setPlanetCode("11111111");
            userList.add(user);
            if (j % batchSize == 0) {
                break;
            }
        }
        // 异步执行
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            System.out.println("threadName: " + Thread.currentThread().getName());
            userService.saveBatch(userList, batchSize);
        }, executorService);
        futureList.add(future);
    }
    CompletableFuture.allOf(futureList.toArray(new CompletableFuture[]{})).join();
    // 20 秒 10 万条
    stopWatch.stop();
    System.out.println(stopWatch.getTotalTimeMillis());
}
```


## 缓存主页和缓存预热

数据库User表里插入100w条数据

第一次从数据库里查主页的伙伴，很慢，需要2.4s，后续的查询也都需要1.6~1.7s

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1698847190402-d475e742-352e-4ec7-9a8d-3a2145562602.png#averageHue=%23888561&clientId=ubb496d6b-192a-4&from=paste&height=763&id=a0Rfy&originHeight=1145&originWidth=2135&originalType=binary&ratio=1.3200000524520874&rotation=0&showTitle=false&size=388606&status=done&style=none&taskId=u80279e01-9ee3-43f7-8a36-d729401bb52&title=&width=1423.3333333333333)
解决方案：
1. Redis缓存
2. 解决第一次查询数据库慢的问题：预加载缓存，定时更新缓存。(定时任务)
   1. 多个机器都要执行任务么? (分布式锁: 控制同一时间只有一台机器去执行定时任务，其他机器不用重复执行了)

缓存的实现：
1. Redis (分布式缓存)
2. memcached (分布式)
3. Etcd (云原生架构的一个分布式存储，存储配置，扩容能力)
4. ehcache (单机)
5. 本地缓存 (Java 内存Map)
6. Caffeine (Java 内存缓存，高性能)
7. Google Guava

spring data redis
> Spring Data: 通用的数据访问框架，定义了一组增删改查的接口，mysql、redis、jpa都可以用

### Redis key设计
多个系统可能共用一个Redis<br />设计规则：`systemId:moduleId:func:[options]`<br />例：`yupao:user:rocommend:userId`
### 缓存实现
**Redis内存不能无限增加，一定要设置过期时间！**<br />自定义RedisTemplate
```java
@Configuration
public class RedisTemplateConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(RedisSerializer.string());
        // value使用jackson的序列化器，starter-web里已经引入了这个包
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        return redisTemplate;
    }
}
```
业务
```java
/**
     * 分页查询用户信息
     *
     * @param pageSize 页大小
     * @param current  当前页
     * @param request
     * @return
     */
    @GetMapping("/recommend")
    private BaseResponse<Page<User>> recommendUsers(long pageSize, long current, HttpServletRequest request) {
        User loginUser = userService.getLoginUser(request);
        // 1 先查缓存
        String key = String.format("yupao:user:recommend:%s", loginUser.getId());
        ValueOperations<String, Object> ops = redisTemplate.opsForValue();
        Object o = ops.get(key);
        Page<User> userPage = (Page<User>) ops.get(key);
        if (userPage != null) {
            return ResultUtils.success(userPage);
        }
        // 2 查数据库
        Page<User> page = new Page<>(current, pageSize);
        userPage = userService.page(page);
        userPage.setRecords(userPage.getRecords().stream()
                .map(user -> userService.getSafetyUser(user))
                .collect(Collectors.toList()));
        // 3 写缓存
        // 这里要直接捕获异常，即便写入失败，数据库也查出来，可以响应数据
        try {
            ops.set(key, userPage,30000, TimeUnit.MILLISECONDS);
        } catch (Exception e) {
            log.error("redis set error");
        }
        return ResultUtils.success(userPage);
    }
```
现在的性能，无敌<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1698894344539-d0ff0c93-4fc3-45a7-940a-97d845270694.png#averageHue=%238d9266&clientId=ud0cd7dda-29a4-4&from=paste&height=784&id=u3189fe82&originHeight=1176&originWidth=1907&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=424878&status=done&style=none&taskId=u123ca302-1853-4852-80cf-fbbda0c7ef9&title=&width=1271.3333333333333)
### 缓存预热
问题:第一个用户访问还是很慢 (加入第一个老板)，也能一定程度上保护数据库<br />缓存预热缺点：

1. 增加开发成本(你要额外的开发、设计)
2. 预热的时机和时间如果错了，有可能你缓存的数据不对或者太老
3. 需要占用额外空间

定时任务实现：

1. Spring Scheduler (spring boot 默认整合了)
2. Quartz (独立于 Spring 存在的定时任务框架)
3. XXLJob 之类的分布式任务调度平台 (界面 + sdk)

Spring Scheduler实现缓存预热：

1. 主类开启 `@EnableScheduling`
2. 给要定时执行的方法添加 `@Scheduling`注解，指定`cron`表达式或者执行频率
> [Cron - 在线Cron表达式生成器](http://cron.ciding.cc/)

对于主要用户`mainUserList`执行缓存预热
```java
package com.yupi.yupao.job;

import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.yupi.yupao.model.domain.User;
import com.yupi.yupao.service.UserService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;

@Component
@Slf4j
public class PreCacheJob {
    @Resource
    private UserService userService;
    @Resource
    private RedisTemplate redisTemplate;

    // 重点用户
    private List<Long> mainUserList = Arrays.asList(1L);

    @Scheduled(cron = "0 0/5 * * * *") // 每5分钟执行一次
    public void doCacheRecommendUser() {
        ValueOperations<String, Object> ops = redisTemplate.opsForValue();
        for (Long userId : mainUserList) {
            String key = String.format("yupao:user:recommend:%s", userId);
            Page<User> page = new Page<>(1, 10);
            Page<User> userPage = userService.page(page);
            userPage.setRecords(userPage.getRecords().stream()
                    .map(user -> userService.getSafetyUser(user))
                    .collect(Collectors.toList()));
            // 写缓存
            // 这里要直接捕获异常，即便写入失败，数据库也查出来，可以响应数据
            try {
                ops.set(key, userPage, 5, TimeUnit.MINUTES);
            } catch (Exception e) {
                log.error("redis set error");
            }
            log.info("定时任务 doCacheRecommendUser---" + LocalDateTime.now());
        }
    }

}

```

## 分布式锁
### 业务背景
**控制定时任务的执行，为啥?**

1. 浪费资源，想象 10000 台服务器同时“打鸣”
2. 脏数据，比如重复插入

**要控制定时任务在同一时间只有 1个服务器能执行，怎么做?**

1. 分离定时任务程序和主程序，只在1个服务器运行定时任务。成本太大
2. 写死配置，每个服务器都执行定时任务，但是只有 ip 符合配置的服务器才真实执行业务逻辑，其他的直接返回。成本最低，但是我们的IP 可能是不固定的，把IP 写的太死了
3. 动态配置，配置是可以轻松的、很方便地更新的(**代码无需重启**)，但是只有 ip 符合配置的服务器才真实执行业务逻辑.
   1. 数据库
   2. Redis
   3. 配置中心(Nacos、Apollo、Spring Cloud Config)
>    4. 问题：服务器多了、IP 不可控还是很麻烦，还是要人工修改

4. 分布式锁，只有抢到锁的服务器才能执行业务逻辑。坏处: 增加成本，好处: 不用手动配置，多少个服务器都一样。
### 分布式锁实现思路

1. MySQL 数据库: select for update 行级锁 (最简单)(观锁)
2. ✔️Redis 实现: 内存数据库，读写速度快。支持 setnx、lua 脚本，比较方便我们实现分布式锁`setnx`: set if not exists 如果不存在，则设置；只有设置成功才会返回 true，否则返回 false

**核心思想**：先来的人先把数据改成自己的标识（服务器ip），后来的人发现标识已存在，就抢锁失败，继续等待。等先来的人执行方法结束，把标识清空，其他的人继续抢锁。

**注意事项：** 

1. 用完锁要释放    
2. 设置过期时间（万一服务挂了没释放锁）
3. 方法执行时间过长，锁提前释放
   1. 连锁效应：释放掉别人的锁
   2. 还是存在多个方法同时执行

解决方案：续期、加锁标识

4. 释放锁的时候，先判断出是自己的锁，然后锁过期了（别人拿到锁），释放了别人的锁
```
if(get lock == 当前线程标识) {
	// lock 过期
  // 其他线程 setnx 成功
	del lock;
}
```
解决方案：lua脚本，保证原子性
### Redisson
[https://github.com/redisson/redisson](https://github.com/redisson/redisson)<br />Java 客户端，数据网格<br />实现了很多Java 里支持的接口和数据结构（继承List等集合接口）

Redisson 是一个java 操作 edis 的客户端，提供了大量的分布式数据集来**简化对 Redis 的操作和使用**，可以让开发者像**使用本地集合样使用 Redis**，完全感知不到 Redis 的存在<br />实例代码：
```java
@Test
    void addTest() {
        List<Object> list = new ArrayList<>();
        list.add("test");
        System.out.println("list[0] =" + list.get(0));

        RList<String> rList = redissonClient.getList("test-list");
        rList.add("test");
        System.out.println("rlist[0] =" + rList.get(0));
    }
    
    @Test
    void removeTest() {
        RList<String> rList = redissonClient.getList("test-list");
        rList.remove(0);
    }
```
### 定时任务+分布式锁
多个服务每天只要只要一个服务执行一次即可

1. waitTime设置为0，抢一次，抢不到第二天再抢
2. finally里释放锁

业务代码：
```java
 @Scheduled(cron = "0 8 22 * * *") // 每天22:08分执行
    public void doCacheRecommendUser() {
        RLock rLock = redissonClient.getLock("yupao:precache:doCacheRecommendUser:lock");
        ValueOperations<String, Object> ops = redisTemplate.opsForValue();
        try {
            // 等待时间为0，不需要可重入，一天只要一个服务执行就够了，抢不到就第二天再抢
            if (rLock.tryLock(0, 30000L, TimeUnit.MILLISECONDS)) {
                // 获取锁成功
                log.info("getLock:" + Thread.currentThread().getId());
                for (Long userId : mainUserList) {
                    String key = String.format("yupao:user:recommend:%s", userId);
                    Page<User> page = new Page<>(1, 10);
                    Page<User> userPage = userService.page(page);
                    userPage.setRecords(userPage.getRecords().stream()
                            .map(user -> userService.getSafetyUser(user))
                            .collect(Collectors.toList()));
                    // 写缓存
                    // 这里要直接捕获异常，即便写入失败，数据库也查出来，可以响应数据
                    try {
                        ops.set(key, userPage, 5, TimeUnit.MINUTES);
                    } catch (Exception e) {
                        log.error("redis set error");
                    }
                    log.info("定时任务 doCacheRecommendUser---" + LocalDateTime.now());
                }
            }else {
                log.info("fail to getLock");
            }
        } catch (InterruptedException e) {
            log.error(e.getMessage());
        } finally {
            if (rLock.isHeldByCurrentThread()) {
                rLock.unlock();
                log.info("unLock:" + Thread.currentThread().getId());
            }
        }
    }

```

## 组队功能
### 需求分析

1. 用户可以 创建 一个队伍，设置队伍的人数、队伍名称 (标题)描述、超时时间
> 队长、剩余的人数
> 聊天?
> 公开或 private 或加密

2. 修改队伍信息
3. 用户可以加入队伍 (其他人、未满、未过期)
> 是否需要队长同意? 筛选审批?

4. 用户可以退出队伍 (如果队长退出，权限转移给第二早加入的用户 -- 先来后到)
5. 队长可以解散队伍
6. 分享队伍=> 邀请其他用户加入队伍
7. 队伍人满后发送消息通知
### 库表设计
**队伍表team**<br />字段:

- id 主键：bigint(最简单、连续，放 url 上比较简短，但缺点是爬虫)
- name 队伍名称
- description 描述
- maxNum 最大人数
- expireTime过期时间
- userld 用户id
- status 0- 公开，1-私有，2- 加密
- password 密码
- createTime创建时间
- updateTime 更新时间
- isDelete 是否删除

**两个关系:**

1. 用户加了哪些队伍?
2. 队伍有哪些用户?

方式

1. ✔️建立用户队伍关系表teamld userld 
   1. 优点：便于修改
2. 用户表补充已加入的队伍字段，队伍表补充已加入的用户字段
   1. 优点：便于查询，不用写多对多的代码，可以直接根据队伍查用户、根据用户查队伍
   2. 缺点：要维护两张表，队伍用户的关系没法逻辑删除

**用户队伍表 user_team**<br />字段:

- id 主键
- userld 用户 id
- teamld 队伍id
- joinTime 加入时间
- createTime 创建时间
- updateTime 更新时间
- isDelete 是否删除

### 系统（接口）设计
#### 创建队伍
用户可以 创建 一个队伍，设置队伍的人数、队伍名称 (标题)、描述、超时时间 PO

1. 请求参数是否为空?
2. 是否登录，未登录不允许创建
3. 校验信息
   1. 队伍人数 >1 且 <= 20
   2. 队伍标题 <= 20
   3. 描述 <= 512
   4. status 是否公开 (int) 不传默认为 0(公开)
   5. 如果 status 是加密状态，一定要有密码，且密码<= 32
   6. 超时时间>当前时间
   7. 校验用户最多创建5个队伍
4. 插入队伍信息到队伍表
5. 插入用户 =>队伍关系到关系表
#### 查询队伍列表
分页展示队伍列表，根据名称搜索队伍，信息流中不展示已过期的队伍

1. 从请求参数中取出队伍名称，如果存在则作为查询条件
2. 不展示已过期的队伍（根据过期时间筛选）
3. 可以通过某个关键词同时对名称和描述查询
4. 只有管理员才能查看加密还有非公开的房间
5. 关联查询队伍创建人信息
6. todo 关联查询已加入队伍的用户信息（很耗费性能）
#### 修改队伍信息

1. 判断请求参数是否为空
2. 查询队伍是否存在
3. 只有管理员或者队伍的创建者可以修改
4. 队伍状态为加密，必须要有密码
5. todo 如果用户传入的新值和老值一致，就不用 update 了(可自行实现，降低数据库使用次数)
#### 加入队伍（分布式锁）
**分布式锁控制并发请求，避免重复加入队伍**<br />请求参数：

1. 队伍id
2. 密码（非必要）

其他人、未满、未过期，允许加入多个队伍，但是要有个上限PO

1. 用户最多加入 5 个队伍
2. 队伍必须存在，只能加入未满、未过期的队伍
3. 不能重复加入已加入的队伍 (幂等性)
4. 禁止加入私有的队伍
5. 如果加入的队伍是加密的，必须密码匹配才可以
6. 新增队伍- 用户关联信息

#### 退出队伍
请求参数：队伍id

1. 校验请求参数
2. 校验队伍是否存在
3. 校验我是否已加入队伍
4. 如果队伍
   1. 只剩一人，队伍解散
   2. 还有其他人
      1. 如果是队长退出队伍，权限转移给第早加入的用户
      2. 非队长，自己退出队伍
#### 队长解散队伍
请求参数：队伍id<br />业务流程：

1. 校验请求参数
2. 校验队伍是否存在
3. 校验你是不是队伍的队长
4. 移除所有加入队伍的关联信息
5. 删除队伍
#### 查看登录用户创建的队伍/加入的队伍
复用listTeam()方法
## 随机匹配（todo）
> 视频（12）~（13）

### 需求分析
> 为了帮大家更快地发现和自己兴趣相同的朋友

匹配多个，并且按照匹配的相似度从高到低排序<br />根据tags匹配
> 还可以根据user_team 匹配加入相同队伍的用户

本质：找到有相似标签的用户
> 用户 A: [Java,大一,男]
> 用户 B:[Java,大二,男]
> 用户 C: [Python,大二,女]

1. 找到有共同标签最多的用户(TopN)
2. 共同标签越多，分数越高，越排在前面
3. 如果没有匹配的用户，随机推荐几个 (降级方案)

编辑距离算法<br />余弦相似度算法：(如果需要带权重计算，比如学什么方向最重要，性别相对次)
### 接口设计
直接取出所有用户，依次和当前用户计算分数，取 TOP N（54 秒）<br />**优化方法**

1. 切忌不要在数据量大的时候循环输出日志（取消掉日志后 20 秒）
2. Map 存了所有的分数信息，占用内存
   1. 解决: 维护一个固定长度的有序集合 (sortedSet) ，只保留分数最高的几个用户 todo
   2. e.g.`[3,4,5,6,7]`取 TOP 5，id 为1的用户就不用放进去了
3. 细节：剔除自己
4. 尽量只查需要的数据
   1. 过滤掉标签为空的用户
   2. 根据部分标签取用户
   3. 只查需要的数据 (比如 id 和 tags)（ 7秒）
5. 提前查? todo
   1. 提前把所有用户给缓存（不适用于经常更新的用户）
   2. 提前运算出来结果，缓存


**大数据推荐**<br />比如说有几亿个商品，难道要查出来所有的商品?<br />难道要对所有的数据计算一遍相似度?<br />检索 => 召回 => 粗排 => 精排 => 重排序等等

- 检索：尽可能多地查符合要求的数据 (比如按记录查）
- 召回：查询可能要用到的数据 (不做运算)
- 粗排：粗略排序，简单地运算 (运算相对轻量)
- 精排：精细排序，确定固定排位

# 项目上线

前端：Vercel

后端：微信云托管

前端区分线上和开发接口

# Todo

1. 多线程插入用户数据，第六期
2. 标签查用户，优化，全表查询卡爆了直接
3. 前端退出登录

