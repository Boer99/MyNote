# 简介

**Spring Security** 是 Spring 家族中的一个安全管理框架。相比与另外一个安全框架**Shiro**，它提供了更丰富的功能，社区资源也比 Shiro 丰富。

一般来说中大型的项目都是使用 **SpringSecurity** 来做安全框架。小项目有 Shiro 的比较多，因为相比与 SpringSecurity，Shiro 的上手更加的简单。

一般 Web 应用的需要进行**认证**和**授权**， 而认证和授权也是 SpringSecurity 作为安全框架的核心功能
- 认证：验证当前访问系统的是不是本系统的用户，并且要确认具体是哪个用户
- 授权：经过认证后判断当前用户是否有权限进行某个操作

# 快速入门

## 准备工作

设置父工程 添加依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

创建启动类

```java
@SpringBootApplication
public class SecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SecurityApplication.class,args);
    }
}
```

创建Controller

```java
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

##  引入SpringSecurity

 在 SpringBoot 项目中使用 SpringSecurity 我们只需要引入依赖即可实现入门案例。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

 引入依赖后我们在尝试去访问之前的接口就会自动跳转到一个 SpringSecurity 的默认登陆页面，默认用户名是 user，密码会输出在控制台。

 必须登陆之后才能对接口进行访问。

# 认证

## 登陆校验流程（仔细看）

![](assets/2e1216c638db41c7b827598695ef8f15.png)

## 原理初探

想要知道如何实现自己的登陆流程就必须要先知道入门案例中 SpringSecurity 的流程。

### SpringSecurity 完整流程（仔细看）

SpringSecurity 的原理其实就是一个过滤器链，内部包含了提供各种功能的过滤器。这里我们可以看看入门案例中的过滤器。

![](assets/124867e0f3e84467ac9ead2419aa450c.png)

​图中只展示了核心过滤器，其它的非核心过滤器并没有在图中展示。

- `UsernamePasswordAuthenticationFilter`：负责处理我们在登陆页面填写了用户名密码后的登陆请求。入门案例的**认证工作**主要有它负责
- `ExceptionTranslationFilter`：处理过滤器链中抛出的任何 AccessDeniedException 和 AuthenticationException 
- `FilterSecurityInterceptor`：负责权限校验的过滤器

Debug 查看当前系统中 SpringSecurity 过滤器链中有哪些过滤器及它们的顺序

![](assets/684b4604a4404264bff0dcdae656288b.png)

### 认证流程详解（仔细看）

![](assets/fd1374d3fe6d40d69f1c6500239308b2.png)

概念速查:

- `Authentication接口`: 它的实现类，表示当前访问系统的用户，封装了用户相关信息。（==贯穿全文的线索！！！==）
- `AuthenticationManager接口`：定义了认证 Authentication 的方法
- `UserDetailsService接口`：加载用户特定数据的核心接口。里面定义了一个根据用户名查询用户信息的方法。
- `UserDetails接口`：提供核心用户信息。
	- 通过 UserDetailsService 根据用户名获取处理的用户信息要**封装成 UserDetails 对象**返回。
	- 然后将这些信息**封装到 Authentication 对象**中。
- `SecurityContextHolder`：存储安全上下文（security context）的信息

## 具体实现

### 思路分析

登录
- 自定义登录接口
	- 调用 ProviderManager 的方法进行认证，如果认证通过生成 jwt
	- 把用户信息存入 redis 中
- 自定义 UserDetailsService
	- 在这个实现类中去查询数据库

校验：
-  定义 Jwt 认证过滤器
	-  获取 token
	-  解析 token 获取其中的 userid
	-  从 redis 中获取用户信息
	-  存入 SecurityContextHolder

### 准备工作

添加依赖

```xml
<!--web-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<!--lombok-->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
<!--springsecurity-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--redis依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--fastjson依赖-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.33</version>
</dependency>
<!--jwt依赖-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
<!--MybatisPlus和mysql驱动的依赖-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<!--junit-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

添加Redis相关配置

```java
/**
 * Redis使用FastJson序列化
 */
public class FastJsonRedisSerializer<T> implements RedisSerializer<T> {

    public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");

    private Class<T> clazz;

    static {
        ParserConfig.getGlobalInstance().setAutoTypeSupport(true);
    }

    public FastJsonRedisSerializer(Class<T> clazz) {
        super();
        this.clazz = clazz;
    }

    @Override
    public byte[] serialize(T t) throws SerializationException {
        if (t == null) {
            return new byte[0];
        }
        return JSON.toJSONString(t, SerializerFeature.WriteClassName).getBytes(DEFAULT_CHARSET);
    }

    @Override
    public T deserialize(byte[] bytes) throws SerializationException {
        if (bytes == null || bytes.length <= 0) {
            return null;
        }
        String str = new String(bytes, DEFAULT_CHARSET);

        return JSON.parseObject(str, clazz);
    }


    protected JavaType getJavaType(Class<?> clazz) {
        return TypeFactory.defaultInstance().constructType(clazz);
    }
}
```

```java
@Configuration
public class RedisConfig {
    @Bean
    @SuppressWarnings(value = { "unchecked", "rawtypes" })
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory)
    {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        FastJsonRedisSerializer serializer = new FastJsonRedisSerializer(Object.class);

        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);

        // Hash的key也采用StringRedisSerializer的序列化方式
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

响应类，统一相应数据的格式

```java
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ResponseResult<T> {
    /**
     * 状态码
     */
    private Integer code;
    /**
     * 提示信息，如果有错误时，前端可以获取该字段进行提示
     */
    private String msg;
    /**
     * 查询到的结果数据，
     */
    private T data;

    public ResponseResult(Integer code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    public ResponseResult(Integer code, T data) {
        this.code = code;
        this.data = data;
    }

    public Integer getCode() {
        return code;
    }

    public void setCode(Integer code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public ResponseResult(Integer code, String msg, T data) {
        this.code = code;
        this.msg = msg;
        this.data = data;
    }
}
```

生成jwt的工具类

```java
/**
 * JWT工具类
 */
public class JwtUtil {

    //有效期为
    public static final Long JWT_TTL = 60 * 60 *1000L;// 60 * 60 *1000  一个小时
    //设置秘钥明文
    public static final String JWT_KEY = "sangeng";

    public static String getUUID(){
        String token = UUID.randomUUID().toString().replaceAll("-", "");
        return token;
    }
    
    /**
     * 生成jtw
     * @param subject token中要存放的数据（json格式）
     * @return
     */
    public static String createJWT(String subject) {
        JwtBuilder builder = getJwtBuilder(subject, null, getUUID());// 设置过期时间
        return builder.compact();
    }

    /**
     * 生成jtw
     * @param subject token中要存放的数据（json格式）
     * @param ttlMillis token超时时间
     * @return
     */
    public static String createJWT(String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, getUUID());// 设置过期时间
        return builder.compact();
    }

    private static JwtBuilder getJwtBuilder(String subject, Long ttlMillis, String uuid) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        SecretKey secretKey = generalKey();
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        if(ttlMillis==null){
            ttlMillis=JwtUtil.JWT_TTL;
        }
        long expMillis = nowMillis + ttlMillis;
        Date expDate = new Date(expMillis);
        return Jwts.builder()
                .setId(uuid)              //唯一的ID
                .setSubject(subject)   // 主题  可以是JSON数据
                .setIssuer("sg")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey) //使用HS256对称加密算法签名, 第二个参数为秘钥
                .setExpiration(expDate);
    }

    /**
     * 创建token
     * @param id
     * @param subject
     * @param ttlMillis
     * @return
     */
    public static String createJWT(String id, String subject, Long ttlMillis) {
        JwtBuilder builder = getJwtBuilder(subject, ttlMillis, id);// 设置过期时间
        return builder.compact();
    }

    public static void main(String[] args) throws Exception {
        String token = "eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiJjYWM2ZDVhZi1mNjVlLTQ0MDAtYjcxMi0zYWEwOGIyOTIwYjQiLCJzdWIiOiJzZyIsImlzcyI6InNnIiwiaWF0IjoxNjM4MTA2NzEyLCJleHAiOjE2MzgxMTAzMTJ9.JVsSbkP94wuczb4QryQbAke3ysBDIL5ou8fWsbt_ebg";
        Claims claims = parseJWT(token);
        System.out.println(claims);
    }

    /**
     * 生成加密后的秘钥 secretKey
     * @return
     */
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.getDecoder().decode(JwtUtil.JWT_KEY);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    
    /**
     * 解析
     *
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
    }
}
```

```java
@SuppressWarnings(value = { "unchecked", "rawtypes" })
@Component
public class RedisCache
{
    @Autowired
    public RedisTemplate redisTemplate;

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key 缓存的键值
     * @param value 缓存的值
     */
    public <T> void setCacheObject(final String key, final T value)
    {
        redisTemplate.opsForValue().set(key, value);
    }

    /**
     * 缓存基本的对象，Integer、String、实体类等
     *
     * @param key 缓存的键值
     * @param value 缓存的值
     * @param timeout 时间
     * @param timeUnit 时间颗粒度
     */
    public <T> void setCacheObject(final String key, final T value, final Integer timeout, final TimeUnit timeUnit)
    {
        redisTemplate.opsForValue().set(key, value, timeout, timeUnit);
    }

    /**
     * 设置有效时间
     *
     * @param key Redis键
     * @param timeout 超时时间
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout)
    {
        return expire(key, timeout, TimeUnit.SECONDS);
    }

    /**
     * 设置有效时间
     *
     * @param key Redis键
     * @param timeout 超时时间
     * @param unit 时间单位
     * @return true=设置成功；false=设置失败
     */
    public boolean expire(final String key, final long timeout, final TimeUnit unit)
    {
        return redisTemplate.expire(key, timeout, unit);
    }

    /**
     * 获得缓存的基本对象。
     *
     * @param key 缓存键值
     * @return 缓存键值对应的数据
     */
    public <T> T getCacheObject(final String key)
    {
        ValueOperations<String, T> operation = redisTemplate.opsForValue();
        return operation.get(key);
    }

    /**
     * 删除单个对象
     *
     * @param key
     */
    public boolean deleteObject(final String key)
    {
        return redisTemplate.delete(key);
    }

    /**
     * 删除集合对象
     *
     * @param collection 多个对象
     * @return
     */
    public long deleteObject(final Collection collection)
    {
        return redisTemplate.delete(collection);
    }

    /**
     * 缓存List数据
     *
     * @param key 缓存的键值
     * @param dataList 待缓存的List数据
     * @return 缓存的对象
     */
    public <T> long setCacheList(final String key, final List<T> dataList)
    {
        Long count = redisTemplate.opsForList().rightPushAll(key, dataList);
        return count == null ? 0 : count;
    }

    /**
     * 获得缓存的list对象
     *
     * @param key 缓存的键值
     * @return 缓存键值对应的数据
     */
    public <T> List<T> getCacheList(final String key)
    {
        return redisTemplate.opsForList().range(key, 0, -1);
    }

    /**
     * 缓存Set
     *
     * @param key 缓存键值
     * @param dataSet 缓存的数据
     * @return 缓存数据的对象
     */
    public <T> BoundSetOperations<String, T> setCacheSet(final String key, final Set<T> dataSet)
    {
        BoundSetOperations<String, T> setOperation = redisTemplate.boundSetOps(key);
        Iterator<T> it = dataSet.iterator();
        while (it.hasNext())
        {
            setOperation.add(it.next());
        }
        return setOperation;
    }

    /**
     * 获得缓存的set
     *
     * @param key
     * @return
     */
    public <T> Set<T> getCacheSet(final String key)
    {
        return redisTemplate.opsForSet().members(key);
    }

    /**
     * 缓存Map
     *
     * @param key
     * @param dataMap
     */
    public <T> void setCacheMap(final String key, final Map<String, T> dataMap)
    {
        if (dataMap != null) {
            redisTemplate.opsForHash().putAll(key, dataMap);
        }
    }

    /**
     * 获得缓存的Map
     *
     * @param key
     * @return
     */
    public <T> Map<String, T> getCacheMap(final String key)
    {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * 往Hash中存入数据
     *
     * @param key Redis键
     * @param hKey Hash键
     * @param value 值
     */
    public <T> void setCacheMapValue(final String key, final String hKey, final T value)
    {
        redisTemplate.opsForHash().put(key, hKey, value);
    }

    /**
     * 获取Hash中的数据
     *
     * @param key Redis键
     * @param hKey Hash键
     * @return Hash中的对象
     */
    public <T> T getCacheMapValue(final String key, final String hKey)
    {
        HashOperations<String, String, T> opsForHash = redisTemplate.opsForHash();
        return opsForHash.get(key, hKey);
    }

    /**
     * 删除Hash中的数据
     * 
     * @param key
     * @param hkey
     */
    public void delCacheMapValue(final String key, final String hkey)
    {
        HashOperations hashOperations = redisTemplate.opsForHash();
        hashOperations.delete(key, hkey);
    }

    /**
     * 获取多个Hash中的数据
     *
     * @param key Redis键
     * @param hKeys Hash键集合
     * @return Hash对象集合
     */
    public <T> List<T> getMultiCacheMapValue(final String key, final Collection<Object> hKeys)
    {
        return redisTemplate.opsForHash().multiGet(key, hKeys);
    }

    /**
     * 获得缓存的基本对象列表
     *
     * @param pattern 字符串前缀
     * @return 对象列表
     */
    public Collection<String> keys(final String pattern)
    {
        return redisTemplate.keys(pattern);
    }
}
```

```java
public class WebUtils
{
    /**
     * 将字符串渲染到客户端
     * 
     * @param response 渲染对象
     * @param string 待渲染的字符串
     * @return null
     */
    public static String renderString(HttpServletResponse response, String string) {
        try
        {
            response.setStatus(200);
            response.setContentType("application/json");
            response.setCharacterEncoding("utf-8");
            response.getWriter().print(string);
        }
        catch (IOException e)
        {
            e.printStackTrace();
        }
        return null;
    }
}
```

实体类

```java
import java.io.Serializable;
import java.util.Date;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {
    private static final long serialVersionUID = -40356785423868312L;
    
    /**
    * 主键
    */
    private Long id;
    /**
    * 用户名
    */
    private String userName;
    /**
    * 昵称
    */
    private String nickName;
    /**
    * 密码
    */
    private String password;
    /**
    * 账号状态（0正常 1停用）
    */
    private String status;
    /**
    * 邮箱
    */
    private String email;
    /**
    * 手机号
    */
    private String phonenumber;
    /**
    * 用户性别（0男，1女，2未知）
    */
    private String sex;
    /**
    * 头像
    */
    private String avatar;
    /**
    * 用户类型（0管理员，1普通用户）
    */
    private String userType;
    /**
    * 创建人的用户id
    */
    private Long createBy;
    /**
    * 创建时间
    */
    private Date createTime;
    /**
    * 更新人
    */
    private Long updateBy;
    /**
    * 更新时间
    */
    private Date updateTime;
    /**
    * 删除标志（0代表未删除，1代表已删除）
    */
    private Integer delFlag;
}
```

创建一个用户表， 建表语句如下：

```mysql
CREATE TABLE `sys_user` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '用户名',
  `nick_name` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '昵称',
  `password` VARCHAR(64) NOT NULL DEFAULT 'NULL' COMMENT '密码',
  `status` CHAR(1) DEFAULT '0' COMMENT '账号状态（0正常 1停用）',
  `email` VARCHAR(64) DEFAULT NULL COMMENT '邮箱',
  `phonenumber` VARCHAR(32) DEFAULT NULL COMMENT '手机号',
  `sex` CHAR(1) DEFAULT NULL COMMENT '用户性别（0男，1女，2未知）',
  `avatar` VARCHAR(128) DEFAULT NULL COMMENT '头像',
  `user_type` CHAR(1) NOT NULL DEFAULT '1' COMMENT '用户类型（0管理员，1普通用户）',
  `create_by` BIGINT(20) DEFAULT NULL COMMENT '创建人的用户id',
  `create_time` DATETIME DEFAULT NULL COMMENT '创建时间',
  `update_by` BIGINT(20) DEFAULT NULL COMMENT '更新人',
  `update_time` DATETIME DEFAULT NULL COMMENT '更新时间',
  `del_flag` INT(11) DEFAULT '0' COMMENT '删除标志（0代表未删除，1代表已删除）',
  PRIMARY KEY (`id`)
) ENGINE=INNODB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户表'
```

 配置数据库信息

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/sg_security?characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
```

 定义Mapper接口

```java
public interface UserMapper extends BaseMapper<User> {}
```

 修改User实体类

```java
类名上加 @TableName(value = "sys_user") ,id字段上加 @TableId
```

 配置Mapper扫描

```java
@SpringBootApplication
@MapperScan("com.boer.mapper")
public class SimpleSecurityApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext run = SpringApplication.run(SimpleSecurityApplication.class);
        System.out.println(run);
    }
}
```

 测试MP是否能正常使用

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class MapperTest {

    @Autowired
    private UserMapper userMapper; //usermapper上加 @repository

    @Test
    public void testUserMapper(){
        List<User> users = userMapper.selectList(null);
        System.out.println(users);
    }
}
```



### 实现

#### 数据库校验用户

从之前的分析我们可以知道，我们可以自定义一个UserDetailsService，让SpringSecurity使用我们的UserDetailsService。我们自己的UserDetailsService可以从数据库中查询用户名和密码。



创建一个类实现UserDetailsService接口，重写其中的方法。更加用户名从数据库中查询用户信息

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //根据用户名查询用户信息
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(wrapper);
        //如果查询不到数据就通过抛出异常来给出提示
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
        //TODO 根据用户查询权限信息 添加到LoginUser中
        
        //封装成UserDetails对象返回 
        return new LoginUser(user);
    }
}
```

因为UserDetailsService方法的返回值是UserDetails类型，所以需要定义一个类，实现该接口，把用户信息封装在其中。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class LoginUser implements UserDetails {
    private User user;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return null;
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUserName();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

注意：如果要测试，需要往用户表中写入用户数据，并且如果你想让用户的密码是`明文`存储，需要在密码前加`{noop}`。例如

![在这里插入图片描述](assets/a4557f33e1c54795aa0a6ca6fb302ce9.png)

这样登陆的时候就可以用sg作为用户名，1234作为密码来登陆了。



#### 密码加密存储

实际项目中我们不会把密码明文存储在数据库中。

默认使用的PasswordEncoder要求数据库中的密码格式为：{id}password 。它会根据id去判断密码的加密方式。但是我们一般不会采用这种方式。所以就需要==替换PasswordEncoder==

我们一般使用SpringSecurity为我们提供的 `BCryptPasswordEncoder`

我们只需要使用把BCryptPasswordEncoder对象注入Spring容器中，SpringSecurity就会使用该PasswordEncoder来进行密码校验

我们可以定义一个SpringSecurity的配置类，SpringSecurity要求这个配置类要继承WebSecurityConfigurerAdapter

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

测试下`BCryptPasswordEncoder`生成的密文，每次都是不一样的，但都能和原来的明文匹配上

```java
@Test
public void testBCryptPasswordEncoder() {
    BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    //加密
    String encode = passwordEncoder.encode("1234");
    String encode2 = passwordEncoder.encode("1234");
    System.out.println(encode); // $2a$10$A5enuqtFU3HF9i25c026O.mL0XoiTOionL0Fw4rwRZKm.5pqihJHO
    System.out.println(encode2); // $2a$10$bGD0K0VTfvS1A1u/LoK.QOGJT/MDWTipBk25iEDxe7x36cwOFEgk6
    //解密
    boolean matches = passwordEncoder.matches("1234", "$2a$10$A5enuqtFU3HF9i25c026O.mL0XoiTOionL0Fw4rwRZKm.5pqihJHO");
    boolean matches1 = passwordEncoder.matches("1234", "$2a$10$bGD0K0VTfvS1A1u/LoK.QOGJT/MDWTipBk25iEDxe7x36cwOFEgk6");
    System.out.println(matches); //true
    System.out.println(matches1); //true
}
```

可以将这串密文放入数据库中，测试一下是否能通过校验



#### jwt工具类的使用

```java
@Test
public void testJWT() throws Exception {
    //对userid加密
    String jwt = JwtUtil.createJWT("2123");
    System.out.println(jwt);
    //解密
    Claims claims = JwtUtil.parseJWT(jwt);
    String subject = claims.getSubject();
    System.out.println(subject); //2123
}
```



#### 登陆接口

接下我们需要自定义登陆接口，然后让SpringSecurity对这个接口放行，让用户访问这个接口的时候不用登录也能访问

在接口中我们通过AuthenticationManager的authenticate方法来进行用户认证，所以需要在SecurityConfig中配置==把AuthenticationManager注入容器==

认证成功的话要生成一个jwt，放入响应中返回。并且为了让用户下回请求时能通过jwt识别出具体的是哪个用户，我们需要把用户信息存入redis，可以把用户id作为key



在 SecurityConfig 中配置把 AuthenticationManager 注入容器，并对登录接口放行

> 这里我们重写了父类的configure方法，在WebSecurityConfigurerAdapter的configure方法会调用http.formLogin()方法去new一个`UsernamePasswordAuthenticationFilter`，所以我们重写以后没有调用该方法，就没有`UsernamePasswordAuthenticationFilter`这个过滤器了

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
    }
}
```

编写 LoginController

```java
@RestController
public class LoginController {
    @Autowired
    private LoginService loginService;

    @PostMapping("/user/login")
    public ResponseResult login(@RequestBody User user) {
        return loginService.login(user);
    }
}
```

编写service

```java
public interface LoginService {
    public ResponseResult login(User user);
}
```

```java
@Service
public class LoginServiceImpl implements LoginService {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private RedisCache redisCache;

    @Override
    public ResponseResult login(User user) {
        // Authentication authenticate(Authentication authentication)
        // Authentication是一个接口，我们创建它的实现类UsernamePasswordAuthenticationToken，传入用户名和密码
        UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user.getUserName(), user.getPassword());
        Authentication authenticate = authenticationManager.authenticate(authenticationToken);
        // authenticate为null，说明认证没通过
        if (Objects.isNull(authenticate)) {
            throw new RuntimeException("登录失败");
        }
        // 使用userid生成token
        // loginUser会被封装成 Principal，需要强转回来
        LoginUser loginUser = (LoginUser) authenticate.getPrincipal();
        String userId = loginUser.getUser().getId().toString();
        String jwt = JwtUtil.createJWT(userId);
        //将完整的用户信息存入redis，userId作为key
//        redisCache.setCacheObject("login:" + userId, loginUser);
        //把token响应给前端
        HashMap<String, String> map = new HashMap<>();
        map.put("token", jwt);
        return new ResponseResult(200, "登陆成功", map);
    }
}
```

测试接口

<img src="assets/image-20220511114702313.png" alt="image-20220511114702313" style="zoom:50%;" />



#### 认证过滤器

> 1、可以回顾下javaweb的过滤器链
>
> 2、SecurityContextHolder：
>
> ​	SecurityContextHolder用于存储安全上下文（security context）的信息。当前操作的用户是谁，该用户是否已经被认证，他拥有哪些角色权限…这些都被保存在SecurityContextHolder中。SecurityContextHolder默认使用ThreadLocal 策略来存储认证信息。看到ThreadLocal 也就意味着，这是一种与线程绑定的策略。Spring Security在用户登录时自动绑定认证信息到当前线程，在用户退出时，自动清除当前线程的认证信息。
>
> ​	SecurityContextHolder是所有过滤器共享的

我们需要自定义一个过滤器，这个过滤器会去获取请求头中的token，对token进行解析取出其中的userid。

使用userid去redis中获取对应的LoginUser对象。

然后封装Authentication对象存入SecurityContextHolder



自定义一个过滤器

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //获取token
        String token = request.getHeader("token");
        if (!StringUtils.hasText(token)) {
            //放行
            filterChain.doFilter(request, response);
            return;
        }
        //解析token
        String userid;
        try {
            Claims claims = JwtUtil.parseJWT(token);
            userid = claims.getSubject();
        } catch (Exception e) {
          	//token超时或者非法
            e.printStackTrace();
            //响应告诉前端需要重新登录，因为不是springmvc管理filter的，这里需要自己封装response
            ResponseResult result = ResponseResult.errorResult(AppHttpCodeEnum.NEED_LOGIN);
            WebUtils.renderString(response, JSON.toJSONString(result));
            return; //不需要执行后面的代码了
        }
        //从redis中获取用户信息
        String redisKey = "login:" + userid;
        LoginUser loginUser = redisCache.getCacheObject(redisKey);
        if(Objects.isNull(loginUser)){
            throw new RuntimeException("用户未登录");
        }
        //存入SecurityContextHolder
        //TODO 获取权限信息封装到Authentication中
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser,null,null);
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        //放行
        filterChain.doFilter(request, response);
    }
}
```

把token校验过滤器添加到过滤器链中，并且要在UsernamePasswordAuthenticationFilter的前面

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                //关闭csrf
                .csrf().disable()
                //不通过Session获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                // 对于登录接口 允许匿名访问
                .antMatchers("/user/login").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated();
        //把token校验过滤器添加到过滤器链中
        http.addFilterBefore(jwtAuthenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
```

测试：

访问/user/login，拿到token

访问/hello，无效

将token放入请求头，访问/hello，成功

<img src="assets/image-20220511162511380.png" alt="image-20220511162511380" style="zoom: 50%;" />

> debug查看当前的过滤器链
>
> <img src="assets/image-20220513160543203.png" alt="image-20220513160543203" style="zoom:50%;" />



#### 退出登陆

我们只需要定义一个登出接口，然后获取SecurityContextHolder中的认证信息（userid），删除redis中对应的数据即可

```java
@RestController
public class LoginController {
    @Autowired
    private LoginService loginService;

    @GetMapping("/user/logout")
    public ResponseResult logout() {
        return loginService.logout();
    }
}
```

```java
public interface LoginService {
    //登出
    public ResponseResult logout();
}
```

```java
@Service
public class LoginServiceImpl implements LoginServcie {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private RedisCache redisCache;

    @Override
    public ResponseResult logout() {
        //从SecurityContextHolder中获取userid
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        Long userid = loginUser.getUser().getId();
        //删除redis中的值
        redisCache.deleteObject("login:" + userid);
        return new ResponseResult(200, "退出成功");
    }
}
```


# 授权

## 权限系统的作用

 例如一个学校图书馆的管理系统，如果是普通学生登录就能看到借书还书相关的功能，不可能让他看到并且去使用添加书籍信息，删除书籍信息等功能。但是如果是一个图书馆管理员的账号登录了，应该就能看到并使用添加书籍信息，删除书籍信息等功能。

 总结起来就是**不同的用户可以使用不同的功能**。这就是权限系统要去实现的效果。

 我们不能只依赖前端去判断用户的权限来选择显示哪些菜单哪些按钮。因为如果只是这样，如果有人知道了对应功能的接口地址就可以不通过前端，直接去发送请求来实现相关功能操作。

 所以我们还需要在后台进行用户权限的判断，判断当前用户是否有相应的权限，必须具有所需权限才能进行相应的操作。



## 授权基本流程（重要）

在SpringSecurity中，会使用默认的 `FilterSecurityInterceptor` 来进行权限校验。在FilterSecurityInterceptor中会从`SecurityContextHolder ` 获取其中的 `Authentication`，然后获取其中的权限信息。当前用户是否拥有访问当前资源所需的权限。

所以我们在项目中只需要把当前登录用户的权限信息也存入 Authentication

然后设置我们的资源所需要的权限即可



## 授权实现

### 限制访问资源所需权限

SpringSecurity为我们提供了基于注解的权限控制方案，这也是我们项目中主要采用的方式。我们可以使用注解去指定访问对应的资源所需的权限。

但是要使用它我们需要先开启相关配置

```java
// 开启基于方法的安全认证机制，也就是说在web层的controller启用注解机制的安全确认
// securedEnabled 确定是否应该启用Spring Security的安全注释
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {}
```

然后就可以使用对应的注解 `@PreAuthorize`

```java
@GetMapping("/hello")
//实际执行的时候会去调用hasAuthority这个方法，判断用户是否具有 test 权限
@PreAuthorize("hasAuthority('test')")
public String hello() {
    return "hello";
}
```

### 封装权限信息

我们前面在写UserDetailsServiceImpl的时候说过，在查询出用户后还要获取对应的权限信息，封装到UserDetails中返回。

我们先直接把权限信息写死封装到UserDetails中进行测试。

我们之前定义了UserDetails的实现类LoginUser，想要让其能封装权限信息就要对其进行修改，增加 permissions 字段存储权限信息

重写 getAuthorities 方法，把permissions中字符串类型的权限信息转换成GrantedAuthority对象存入authorities中

```java
@Data
@NoArgsConstructor
public class LoginUser implements UserDetails {
  
    private User user;
        
    //存储用户的权限信息
    private List<String> permissions;

    public LoginUser(User user, List<String> permissions) {
        this.user = user;
        this.permissions = permissions;
    }

    //存储SpringSecurity所需要的权限信息的集合
    @JSONField(serialize = false) //不让序列化，redis不存
    //将authorities作为全局变量，不用每次都转换 permissions 为 authorities
    //GrantedAuthority 是 GrantedAuthority 的实现类
    private List<GrantedAuthority> authorities;

    @Override
    //接受的返回类型是 GrantedAuthority 实现类的集合
    public Collection<? extends GrantedAuthority> getAuthorities() {
        if (authorities != null) {
            return authorities;
        }
        //把permissions中字符串类型的权限信息转换成GrantedAuthority对象存入authorities中
        //函数式编程写法
        authorities = permissions.stream().
                map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
        return authorities;
        /*
        原写法：
        authorities=new ArrayList<>();
        for (String permission : permissions) {
            SimpleGrantedAuthority authority = new SimpleGrantedAuthority(permission);
            authorities.add(authority);
        }
        */
    }
}
```

LoginUser修改完后我们就可以在UserDetailsServiceImpl中去把权限信息封装到LoginUser中了。我们写死权限进行测试，后面我们再从数据库中查询权限信息

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(wrapper);
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
        //TODO 根据用户查询权限信息 添加到LoginUser中
        List<String> list = new ArrayList<>(Arrays.asList("test"));
        return new LoginUser(user,list);
    }
}
```

在自定义过滤器中将获取的权限信息封装到Authentication中

```java
@Component
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    @Autowired
    private RedisCache redisCache;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        //获取token
        ...
        //解析token
        ...
        //从redis中获取用户信息
     		...
        //存入SecurityContextHolder
        //获取权限信息 loginUser.getAuthorities() 封装到Authentication中
        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(loginUser, null, loginUser.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(authenticationToken);
        //放行
        filterChain.doFilter(request, response);
    }
}
```



### 从数据库查询权限信息

#### RBAC权限模型

RBAC权限模型（Role-Based Access Control）即：基于角色的权限控制。这是目前最常被开发者使用也是相对易用、通用权限模型。

用户和角色是多对多关系，角色和权限是多对多关系

![700](assets/71bd9c6dddf74daba1429818049b7703.png)

#### 准备工作

数据库创建 user、role、menu 以及 关联表

```
sql脚本
```

sql测试语句，根据userid查询perms，对应的role和menu都必须是是正常状态

```mysql
SELECT 
	DISTINCT m.`perms`
FROM
	sys_user_role ur
	LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
	LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
	LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
WHERE
	user_id = 2
	AND r.`status` = 0
	AND m.`status` = 0
```

菜单表(Menu)实体类

```java
/**
 * 菜单表(Menu)实体类
 */
@TableName(value="sys_menu")
@Data
@AllArgsConstructor
@NoArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Menu implements Serializable {
    private static final long serialVersionUID = -54979041104113736L;
    
    @TableId
    private Long id;
    /**
    * 菜单名
    */
    private String menuName;
    /**
    * 路由地址
    */
    private String path;
    /**
    * 组件路径
    */
    private String component;
    /**
    * 菜单状态（0显示 1隐藏）
    */
    private String visible;
    /**
    * 菜单状态（0正常 1停用）
    */
    private String status;
    /**
    * 权限标识
    */
    private String perms;
    /**
    * 菜单图标
    */
    private String icon;
    
    private Long createBy;
    
    private Date createTime;
    
    private Long updateBy;
    
    private Date updateTime;
    /**
    * 是否删除（0未删除 1已删除）
    */
    private Integer delFlag;
    /**
    * 备注
    */
    private String remark;
}
```



#### 代码实现

我们只需要根据用户id去查询到其所对应的权限信息即可

所以我们可以先定义个mapper，其中提供一个方法可以根据userid查询权限信息

```java
public interface MenuMapper extends BaseMapper<Menu> {
    List<String> selectPermsByUserId(Long id);
}
```

这是自定义方法，没法使用mp自带的方法，所以需要创建对应的mapper映射文件，定义对应的sql语句

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.boer.mapper.MenuMapper">
    <select id="selectPermsByUserId" resultType="java.lang.String">
        SELECT
            DISTINCT m.`perms`
        FROM
            sys_user_role ur
            LEFT JOIN `sys_role` r ON ur.`role_id` = r.`id`
            LEFT JOIN `sys_role_menu` rm ON ur.`role_id` = rm.`role_id`
            LEFT JOIN `sys_menu` m ON m.`id` = rm.`menu_id`
        WHERE
            user_id = #{userid}
            AND r.`status` = 0
            AND m.`status` = 0
    </select>
</mapper>
```

在application.yml中配置mapperXML文件的位置

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/sg_security?characterEncoding=utf-8&serverTimezone=UTC
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  redis:
    host: localhost
    port: 6379
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*.xml # 可以不配，MybatisPlusProperties中默认值就是这个
```

测试一下mapper的功能

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class MapperTest {
    @Autowired
    private MenuMapper menuMapper;

    @Test
    public void testSelectPermsByUserId() {
        List<String> list = menuMapper.selectPermsByUserId(2L);
        System.out.println(list);
    }
}
```

然后我们可以在UserDetailsServiceImpl中去调用该mapper的方法查询权限信息封装到LoginUser对象中即可。

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private UserMapper userMapper;

    @Autowired
    private MenuMapper menuMapper;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getUserName,username);
        User user = userMapper.selectOne(wrapper);
        if(Objects.isNull(user)){
            throw new RuntimeException("用户名或密码错误");
        }
      
        //根据用户查询权限信息 添加到LoginUser中
        List<String> list =  menuMapper.selectPermsByUserId(user.getId());
        //封装成UserDetails对象返回
        //List<String> list = new ArrayList<>(Arrays.asList("test")); //原来是写死的
        return new LoginUser(user, list);
    }
}
```

重新为控制器设置权限，按照数据库里的标准

```java
@GetMapping("/hello")
//@PreAuthorize("hasAuthority('test')")
@PreAuthorize("hasAuthority('system:dept:list')") //直接从menu表复制过来
//实际执行的时候会去调用hasAuthority这个方法，判断用户是否具有 test 权限
public String hello() {
    return "hello";
}
```



# 自定义失败处理

我们还希望在认证失败或者是授权失败的情况下也能和我们的接口一样返回相同结构的json，这样可以让前端能对响应进行统一的处理。要实现这个功能我们需要知道SpringSecurity的异常处理机制。

在SpringSecurity中，如果我们在认证或者授权的过程中出现了异常会被 `ExceptionTranslationFilter` 捕获到。在ExceptionTranslationFilter中会去判断是认证失败还是授权失败出现的异常。

如果是认证过程中出现的异常会被封装成 `AuthenticationException` 然后调用 `AuthenticationEntryPoint` 对象的方法去进行异常处理。

如果是授权过程中出现的异常会被封装成 `AccessDeniedException` 然后调用 `AccessDeniedHandler` 对象的方法去进行异常处理。

所以如果我们需要自定义异常处理，我们只需要自定义 `AuthenticationEntryPoint` 和 `AccessDeniedHandler` 然后配置给SpringSecurity即可。



自定义 `AuthenticationEntryPoint` 和 `AccessDeniedHandler` 的实现类

```java
@Component
public class AuthenticationEntryPointImpl implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        // HttpStatus.UNAUTHORIZED.value() 就是 401
        ResponseResult result = new ResponseResult(HttpStatus.UNAUTHORIZED.value(), "认证失败请重新登录");
        // com.alibaba.fastjson下的JSON，toJSONString()方法返回一个json格式的字符串
        String json = JSON.toJSONString(result);
        WebUtils.renderString(response, json);
    }
}
```

```java
@Component
public class AccessDeniedHandlerImpl implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        // HttpStatus.FORBIDDEN.value() 就是 403
        ResponseResult result = new ResponseResult(HttpStatus.FORBIDDEN.value(), "权限不足");
        String json = JSON.toJSONString(result);
        WebUtils.renderString(response, json);
    }
}
```

配置异常处理器给SpringSecurity，先注入我们实现的异常处理器

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private AuthenticationEntryPoint authenticationEntryPoint;
    @Autowired
    private AccessDeniedHandler accessDeniedHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // 配置异常处理器
        http.exceptionHandling().authenticationEntryPoint(authenticationEntryPoint).
                accessDeniedHandler(accessDeniedHandler);
    }
}
```

测试认证失败，输入错误密码或者不存在的用户名

![](assets/image-20220512193208296.png)

测试授权失败，把controller的权限改改就行





# 跨域

浏览器出于安全的考虑，使用 XMLHttpRequest对象发起 HTTP请求时必须遵守同源策略，否则就是跨域的HTTP请求，默认情况下是被禁止的。 同源策略要求源相同才能正常进行通信，即协议、域名、端口号都完全一致。

前后端分离项目，前端项目和后端项目一般都不是同源的，所以肯定会存在跨域请求的问题。

所以我们就要处理一下，让前端能进行跨域请求。



先对SpringBoot配置，允许跨域请求

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        // 设置允许跨域的路径
        registry.addMapping("/**")
                // 设置允许跨域请求的域名
                .allowedOriginPatterns("*")
                // 是否允许cookie
                .allowCredentials(true)
                // 设置允许的请求方式
                .allowedMethods("GET", "POST", "DELETE", "PUT")
                // 设置允许的header属性
                .allowedHeaders("*")
                // 跨域允许时间
                .maxAge(3600);
    }
}
```

开启SpringSecurity的跨域访问

由于我们的资源都会收到SpringSecurity的保护，所以想要跨域访问还要让SpringSecurity运行跨域访问

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        ......
        //允许跨域
        http.cors();
    }
}
```



# 遗留小问题

## 其它权限校验方法

我们前面都是使用@PreAuthorize注解，然后在在其中使用的是hasAuthority方法进行校验。SpringSecurity还为我们提供了其它方法例如：hasAnyAuthority，hasRole，hasAnyRole等。

这里我们先不急着去介绍这些方法，我们先去理解hasAuthority的原理，然后再去学习其他方法你就更容易理解，而不是死记硬背区别。并且我们也可以选择定义校验方法，实现我们自己的校验逻辑。

debug下 `hasAuthority()` 方法：

- hasAuthority方法实际是执行到了SecurityExpressionRoot的hasAuthority，大家只要断点调试既可知道它内部的校验原理。

- 它内部其实是调用authentication的getAuthorities方法获取用户的权限列表。然后判断我们存入的方法参数数据是否在权限列表中。



### hasAnyAuthority

方法可以传入多个权限，只有用户有其中任意一个权限就可以访问对应资源。

```java
@PreAuthorize("hasAnyAuthority('admin','test','system:dept:list')")
public String hello(){
    return "hello";
}
```

### hasRole（不好用）

要求有对应的角色才可以访问，但是它内部会把我们传入的参数拼接上 **ROLE_** 后再去比较。所以这种情况下要用用户对应的权限也要有 **ROLE_** 这个前缀才可以

```java
@PreAuthorize("hasRole('system:dept:list')")
public String hello(){
    return "hello";
}
```

### hasAnyRole（不好用）

有任意的角色就可以访问。它内部也会把我们传入的参数拼接上 **ROLE_** 后再去比较。所以这种情况下要用用户对应的权限也要有 **ROLE_** 这个前缀才可以。

```java
@PreAuthorize("hasAnyRole('admin','system:dept:list')")
public String hello(){
    return "hello";
}
```



## 自定义权限校验方法

我们也可以定义自己的权限校验方法，在@PreAuthorize注解中使用我们的方法。

```java
@Component("ex") //bean的名字是 ex
public class SGExpressionRoot {
    //自定义的hasAuthority()方法
    public boolean hasAuthority(String authority) {
        //获取当前用户的权限
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        LoginUser loginUser = (LoginUser) authentication.getPrincipal();
        List<String> permissions = loginUser.getPermissions();
        //判断用户权限集合中是否存在authority
        return permissions.contains(authority);
    }
}
```

在==SPEL表达式==使用 @ex相当于获取容器中bean的名字未ex的对象。然后再调用这个对象的hasAuthority方法

```java
@GetMapping("/hello")
//@PreAuthorize("hasAuthority('system:dept:list')") //直接从menu表复制过来
@PreAuthorize("@ex.hasAuthority('system:dept:list')")
//实际执行的时候会去调用hasAuthority这个方法，判断用户是否具有 test 权限
public String hello() {
    return "hello";
}
```



## 基于配置的权限控制

我们也可以在配置类中使用使用配置的方式对资源进行权限控制。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
            // 对于登录接口 允许匿名访问
            .antMatchers("/user/login").anonymous()
            // 对于/hello2接口，设置权限 system:dept:list
            .antMatchers("/hello2").hasAuthority("system:dept:list")
            // 除上面外的所有请求全部需要鉴权认证
            .anyRequest().authenticated();

}
```



## CSRF

 CSRF是指跨站请求伪造（Cross-site request forgery），是web常见的攻击之一。

 https://blog.csdn.net/freeking101/article/details/86537087

 SpringSecurity去防止CSRF攻击的方式就是通过csrf_token。后端会生成一个csrf_token，前端发起请求的时候需要携带这个csrf_token,后端会有过滤器进行校验，如果没有携带或者是伪造的就不允许访问。

 我们可以发现CSRF攻击依靠的是cookie中所携带的认证信息。但是在前后端分离的项目中我们的认证信息其实是token，而token并不是存储中cookie中，并且需要前端代码去把token设置到请求头中才可以，所以CSRF攻击也就不用担心了。



## 认证成功处理器（了解）

前提：当前系统中有UsernamePasswordAuthenticationFilter这个过滤器

实际上在UsernamePasswordAuthenticationFilter进行登录认证的时候，如果登录成功了是会调用AuthenticationSuccessHandler的方法进行认证成功后的处理的。AuthenticationSuccessHandler就是登录成功处理器。

我们也可以自己去自定义成功处理器进行成功后的相应处理。

```java
@Component
public class SGSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        System.out.println("认证成功了");
    }
}
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private AuthenticationSuccessHandler successHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
      	//调用了formLogin()，才有UsernamePasswordAuthenticationFilter这个过滤器，才能调用AuthenticationSuccessHandler
        http.formLogin().successHandler(successHandler);

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```



## 认证失败处理器（了解）

实际上在UsernamePasswordAuthenticationFilter进行登录认证的时候，如果认证失败了是会调用AuthenticationFailureHandler的方法进行认证失败后的处理的。AuthenticationFailureHandler就是登录失败处理器。

我们也可以自己去自定义失败处理器进行失败后的相应处理。

```java
@Component
public class SGFailureHandler implements AuthenticationFailureHandler {
    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        System.out.println("认证失败了");
    }
}
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticationSuccessHandler successHandler;

    @Autowired
    private AuthenticationFailureHandler failureHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
//                配置认证成功处理器
                .successHandler(successHandler)
//                配置认证失败处理器
                .failureHandler(failureHandler);

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```



## 登出成功处理器（了解）

```java
@Component
public class SGLogoutSuccessHandler implements LogoutSuccessHandler {
    @Override
    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        System.out.println("注销成功");
    }
}
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private AuthenticationSuccessHandler successHandler;

    @Autowired
    private AuthenticationFailureHandler failureHandler;

    @Autowired
    private LogoutSuccessHandler logoutSuccessHandler;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin()
//                配置认证成功处理器
                .successHandler(successHandler)
//                配置认证失败处理器
                .failureHandler(failureHandler);
        http.logout()
                //配置注销成功处理器
                .logoutSuccessHandler(logoutSuccessHandler);

        http.authorizeRequests().anyRequest().authenticated();
    }
}
```



# spring security配置详解

```java
/**
 * spring security配置
 *
 * @author ruoyi
 */
//@EnableGlobalMethodSecurity(prePostEnabled=true)
// 开启基于方法的安全认证机制，也就是说在web层的controller启用注解机制的安全确认，而securedEnabled
// 确定是否应该启用Spring Security的安全注释。
// 返回值: 如果应该启用安全注释，则为true，否则为false。默认是假的。
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter
{
    /**
     * 自定义用户认证逻辑
     * 主要方法为loadUserByUsername，依据username去数据库查出该用户
     * 再根据该用户Status字段来判断该用户目前的状况
     */
    @Autowired
    private UserDetailsService userDetailsService;

    /**
     * 认证失败处理类
     * 主要方法为commence，用于返回response，该返回状态码为200，返回消息为认证失败
     */
    @Autowired
    private AuthenticationEntryPointImpl unauthorizedHandler;

    /**
     * 退出处理类
     * 主要方法为onLogoutSuccess，主要做两件事：
     *  1. 从redis中删除用户缓存记录
     *  2. 通过异步的方式记录用户退出日志，输出log日志，并将相关信息写到数据库中
     */
    @Autowired
    private LogoutSuccessHandlerImpl logoutSuccessHandler;

    /**
     * token认证过滤器
     * 主要是从request中获取用户，并及时刷新redis中的用户信息
     */
    @Autowired
    private JwtAuthenticationTokenFilter authenticationTokenFilter;

    /**
     *
     * 跨域过滤器
     * 跨域配置，实际上配置了所有请求都允许
     */
    @Autowired
    private CorsFilter corsFilter;

    /**
     * 解决 无法直接注入 AuthenticationManager
     * 用于处理认证请求，提供authenticate(认证)方法等
     * 使用样例如下：
     * @Resource
     * private AuthenticationManager authenticationManager;
     *
     * @return
     * @throws Exception
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception
    {
        return super.authenticationManagerBean();
    }

    /**
     * anyRequest          |   匹配所有请求路径
     * access              |   SpringEl表达式结果为true时可以访问
     * anonymous           |   匿名可以访问
     * denyAll             |   用户不能访问
     * fullyAuthenticated  |   用户完全认证可以访问（非remember-me下自动登录）
     * hasAnyAuthority     |   如果有参数，参数表示权限，则其中任何一个权限可以访问
     * hasAnyRole          |   如果有参数，参数表示角色，则其中任何一个角色可以访问
     * hasAuthority        |   如果有参数，参数表示权限，则其权限可以访问
     * hasIpAddress        |   如果有参数，参数表示IP地址，如果用户IP和参数匹配，则可以访问
     * hasRole             |   如果有参数，参数表示角色，则其角色可以访问
     * permitAll           |   用户可以任意访问
     * rememberMe          |   允许通过remember-me登录的用户访问
     * authenticated       |   用户登录后可访问
     */
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception
    {
        httpSecurity
                // CSRF禁用，因为不使用session
                .csrf().disable()
                // 认证失败处理类
                .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
                // 基于token，所以不需要session，
                // STATELESS：Spring Security永远不会创建HttpSession，也永远不会使用它来获取SecurityContext
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                // 过滤请求
                .authorizeRequests()
                // 对于登录login 验证码captchaImage 允许匿名访问
                .antMatchers("/login", "/captchaImage").anonymous()
//                对于静态资源，用户可以任意访问
                .antMatchers(
                        HttpMethod.GET,
                        "/*.html",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js"
                ).permitAll()
                .antMatchers("/profile/**").anonymous()
                .antMatchers("/common/download**").anonymous()
                .antMatchers("/common/download/resource**").anonymous()
                .antMatchers("/swagger-ui.html").anonymous()
                .antMatchers("/swagger-resources/**").anonymous()
                .antMatchers("/webjars/**").anonymous()
                .antMatchers("/*/api-docs").anonymous()
                .antMatchers("/druid/**").anonymous()
                .antMatchers("/screen/**").anonymous()
                //Todo:调试开放接口
//                .antMatchers("/communityManager/**/**").anonymous()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated()
                .and()
//                禁用安全响应头
                .headers().frameOptions().disable();
        httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler);
        // 添加JWT filter
        httpSecurity.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        // 添加CORS filter
//        CORS它允许浏览器向跨源服务器，发出XMLHttpRequest请求，从而克服了AJAX只能同源使用的限制。
//        同源：协议相同、域名相同、端口相同
        httpSecurity.addFilterBefore(corsFilter, JwtAuthenticationTokenFilter.class);
        httpSecurity.addFilterBefore(corsFilter, LogoutFilter.class);
    }


    /**
     * 强散列哈希加密实现
     */
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder()
    {
        return new BCryptPasswordEncoder();
    }

    /**
     * 身份认证接口
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception
    {
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }
}
```


