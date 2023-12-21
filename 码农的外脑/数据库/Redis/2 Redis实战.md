![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693397930689-f148c778-065b-4c08-8dab-254c6328a33f.png?x-oss-process=image%2Fresize%2Cw_1299%2Climit_0#averageHue=%23f4f0f0&from=url&id=GIBYp&originHeight=682&originWidth=1299&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&status=done&style=none&title=)<br />前端启动：在nginx目录里开个终端
```
start nginx.exe
```
# Redis共享Session登录（单点登录）
## 基于Session实现
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694103595146-f3c6f7ab-e830-40fa-8e24-556a2fdb7d8e.png#averageHue=%23f0eeee&clientId=u7bca12c5-0781-4&from=paste&height=400&id=uc3cb349f&originHeight=693&originWidth=1421&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=417511&status=done&style=none&taskId=u48a867ff-ba4a-452d-82e9-942acc47a48&title=&width=820.2020405190493)<br />基于Session实现的登录/注册成功后，不需要返回登录凭证的，因为每个Session都一个唯一的SessionId，会保存到Cookie里，客户端每次请求都会携带SessionId，获取Session对象。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694352259577-e83fc09f-d46f-4d7d-ac3b-90fbd6da6030.png#averageHue=%23f9fcf9&clientId=u13004d99-a575-4&from=paste&height=436&id=u9ec50a36&originHeight=755&originWidth=1708&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=851822&status=done&style=none&taskId=u0fa1b17d-48a7-4ae6-97fb-377febcde3b&title=&width=985.8586102790543)
## 基于Redis实现
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694355762551-51409f6a-faef-49f0-b674-76279f098d3e.png#averageHue=%23f7f4f3&clientId=u13004d99-a575-4&from=paste&height=405&id=u9820fc73&originHeight=701&originWidth=1391&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=383288&status=done&style=none&taskId=ucba88eef-879a-4ba1-ad8f-96df80f6b44&title=&width=802.8860227741011)

Redis独立于tomcat服务器，基于Redis实现共享Session登录<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694356169143-e05bdbcd-1a2c-4bc4-a12c-2ab2c37418fd.png#averageHue=%23f1eded&clientId=u13004d99-a575-4&from=paste&height=417&id=u0cec14b5&originHeight=722&originWidth=1447&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=404345&status=done&style=none&taskId=u5d431b23-89d5-46e4-b3bd-5c37ef5f57b&title=&width=835.2092558980045)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694356290609-aec11291-adb9-41af-b3fa-8d9d08827f6d.png#averageHue=%23f0ecec&clientId=u13004d99-a575-4&from=paste&height=380&id=u21b138cf&originHeight=659&originWidth=1354&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=372544&status=done&style=none&taskId=u4b2f710e-f547-48a1-b43e-bc8eac1a989&title=&width=781.5296008886648)<br />注意事项：

1. 存入Redis的数据一定要设置保存时间
2. 存入 Redis 的数据尽量保证精简和安全，比如存入用户信息时可以移除密码等敏感数据
### 短信验证码登录
1、发送验证码<br />路径：`@PostMapping("code")`
```java
@Override
public Result sendCode(String phone) {
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误！");
    }
    String code = RandomUtil.randomNumbers(6);
    // 过期时间为2分钟
    stringRedisTemplate.opsForValue()
            .set(LOGIN_CODE_KEY + phone, code, LOGIN_CODE_TTL, TimeUnit.MINUTES);
    // 模拟发送验证码给用户
    log.info("发送短信验证码成功，验证码：{}", code);
    return Result.ok();
}
```
2、短信验证码登录、注册<br />路径：
```java
@Override
public Result login(LoginFormDTO loginForm) {
    String phone = loginForm.getPhone();
    if (RegexUtils.isPhoneInvalid(phone)) {
        return Result.fail("手机号格式错误！");
    }
    // 校验验证码
    String cacheCode = stringRedisTemplate.opsForValue().get(LOGIN_CODE_KEY + phone);
    // ！！！这里不能用 cacheCode != loginForm.getCode()
    if (cacheCode == null || !cacheCode.equals(loginForm.getCode())) {
        return Result.fail("验证码错误！");
    }
    // 获取用户（不存在就创建）
    User user = query().eq("phone", phone).one();
    if (user == null) {
        user = createUserWithPhone(phone);
    }
    // 保存用户信息到Redis，key=token, value=userMap
    // 1、key
    String token = UUID.randomUUID().toString();
    String tokenKey = LOGIN_USER_KEY + token;
    // 2、value
    UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
    // userDTO的id是Long类型，转成String会报错
    // BeanUtil是hutool工具包
//        Map<String, Object> userMap = BeanUtil.beanToMap(userDTO);
    Map<String, Object> userMap = BeanUtil.beanToMap(
            userDTO,
            new HashMap<>(),
            CopyOptions.create()
                    // 是否忽略空字段
                    .setIgnoreNullValue(true)
                    // 自定义属性值转换规则
                    .setFieldValueEditor((fieldName, fieldValue) -> fieldValue.toString())
    );
    // 3、保存
    stringRedisTemplate.opsForHash().putAll(tokenKey, userMap);
    stringRedisTemplate.expire(tokenKey, LOGIN_USER_TTL, TimeUnit.MINUTES);
    return Result.ok(token);
}
```
### 校验登录状态
已登录用户访问系统后，记得刷新 token 过期时间（续期）。并且访问任何路径时都要刷新 token<br />不是所有路径都需要登录校验，登录注册页面、首页等就不需要登录，所以可以分开设置token刷新拦截器和登录拦截器，要给登录拦截器设置排除的路径<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694509596581-9eddc3c5-d6ba-457b-b86a-b03adca216ff.png#averageHue=%23e5cdc9&clientId=u3c385b45-4012-4&from=paste&height=373&id=u23300e9d&originHeight=647&originWidth=1391&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=298461&status=done&style=none&taskId=ub789d568-6dc2-48df-ae45-289d9056821&title=&width=802.8860227741011)<br />请求->token刷新拦截器->登录拦截器
```java
public class RefreshTokenInterceptor implements HandlerInterceptor {
    StringRedisTemplate stringRedisTemplate;

    // RefreshTokenInterceptor没有注册为组件，只能构造器传入stringRedisTemplate
    public RefreshTokenInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String token = request.getHeader("authorization");
        if (StrUtil.isEmpty(token)) { // return str == null || str.length() == 0;
            return true;
        }
        // 1、基于token获取用户
        String key  = LOGIN_USER_KEY + token;
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(key);
        // token可能会过期或者是假的
        if (userMap.isEmpty()) {
            return true;
        }
        // 参数3：是否忽略注入错误
        UserDTO userDTO = BeanUtil.fillBeanWithMap(userMap, new UserDTO(), false);
        // 2、保存到ThreadLocal
        UserHolder.saveUser(userDTO);
        // 3、刷新token有效期
        stringRedisTemplate.expire(key, LOGIN_USER_TTL, TimeUnit.MINUTES);
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 移除用户
        UserHolder.removeUser();
    }
}
```
```java
public class LoginInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        if (UserHolder.getUser() == null) {
            response.sendError(401);
            // 拦截
            return false;
        }
        return true;
    }
}
```
配置拦截器
```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {
    @Resource
    StringRedisTemplate stringRedisTemplate;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // token刷新拦截器
        registry
                .addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate))
                .addPathPatterns("/**")
                .order(0);
        // 登录校验拦截器
        registry
                .addInterceptor(new LoginInterceptor())
                // 登录校验排除路径
                .excludePathPatterns(
                        "/shop/**",
                        "/shop-type/**",
                        "/blog/hot",
                        "/user/code",
                        "/user/login"
                )
                .order(1);
    }
}
```
前端通过axios的拦截器，将token存到请求头里，key为authorization<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694356591427-96296aaf-553c-4447-b095-302e48fe9705.png#averageHue=%23fbf9f9&clientId=u13004d99-a575-4&from=paste&height=503&id=U5Yis&originHeight=872&originWidth=798&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=449895&status=done&style=none&taskId=u6d2f848e-b58b-4d9a-a84e-0e7f4f2c985&title=&width=460.60607201562374)
# 缓存
缓存就是数据交换的缓冲区(称作cache)，是存储数据的临时地方，一般读写性能较高<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694525691075-0784d641-ee32-4b99-bd23-45a4c697c72b.png#averageHue=%23fbfbfb&clientId=u3c385b45-4012-4&from=paste&height=284&id=u8ee342ea&originHeight=492&originWidth=1244&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=85031&status=done&style=none&taskId=u8d1fd0f7-1e14-470d-9e3e-00b68c9b2da&title=&width=718.0375358238546)<br />缓存的作用（优点）：

1. 降低后端负载
2. 提高读写效率，降低响应时间

缓存的成本（缺点）

1. 保证数据一致性
2. 额外开发和解决缓存带来的问题，增加代码成本
3. 额外引入中间件，增加运维成本
## 添加Redis缓存
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694526188574-10b306a9-7cda-457c-ae72-4a2ccf440202.png#averageHue=%23f0efef&clientId=u3c385b45-4012-4&from=paste&height=399&id=u4853f72f&originHeight=691&originWidth=1378&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=320295&status=done&style=none&taskId=u6abaa87a-56a1-4b5c-8124-198e5d2856c&title=&width=795.3824150846234)<br />给商铺查询添加缓存<br />请求路径：`@GetMapping("/{id}")`
```java
@Resource
StringRedisTemplate stringRedisTemplate;

@Override
public Result queryShopById(Long id) {
    // 1 查询缓存
    String shopKey = CACHE_SHOP_KEY + id;
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    // 2 命中直接返回
    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // 3 未命中查数据库
    Shop shop = getById(id);
    // 3.1 不存在，返回404
    if (shop == null) {
        return Result.fail("商户不存在");
    }
    // 3.2 存在商铺，写入Redis，返回商铺信息
    stringRedisTemplate.opsForValue().set(shopKey, JSONUtil.toJsonStr(shop));
    return Result.ok(shop);
}
```
## 缓存更新策略
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694530148845-78bb51ce-2d0c-4d02-8a0f-34b8f506c42a.png#averageHue=%23e3d4d3&clientId=u8b7321a1-0d1f-4&from=paste&height=387&id=ubf5044ef&originHeight=670&originWidth=1379&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=281783&status=done&style=none&taskId=u2cddcf1a-6828-448a-95b5-ae3d4a23f96&title=&width=795.9596156761218)

主动更新策略<br />Cache Aside Pattern：企业里最常用<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694530791106-012699b6-56b2-4df9-abee-155c7ba73d1a.png#averageHue=%23efeeee&clientId=u8b7321a1-0d1f-4&from=paste&height=287&id=u428e4ace&originHeight=497&originWidth=1404&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=212456&status=done&style=none&taskId=u23dec05b-9739-4080-a41b-8ef8f64e5ff&title=&width=810.3896304635787)

操作缓存和数据库时有三个问题需要考虑<br />1.）删除缓存还是更新缓存?

- ❌更新缓存：每次更新数据库都更新缓存，无效写操作较多
- ✔️删除缓存：**更新数据库时让缓存失效，查询时再更新缓存**

2.）如何保证缓存与数据库的操作的同时成功或失败?

- 单体系统：将缓存和数据库操作放在一个事务里
- 分布式系统：TCC等分布式事务方案

3.）先操作缓存还是先操作数据库?<br />两种方案都有可能发生**线程安全**问题，导致**缓存和数据库不一致**的问题<br />方案一发生的可能性高，**方案二发生的可能行很低**，因为缓存的速度高于数据库<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694533096441-4c365ba0-c0fa-4832-aa99-62b8f33ee912.png#averageHue=%23f2f1f1&clientId=u74749200-d9d1-4&from=paste&height=412&id=u0ae05ed9&originHeight=714&originWidth=1305&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=254032&status=done&style=none&taskId=u3d57ebd4-7597-4935-bab6-8f5f6a001ad&title=&width=753.2467719052494)

缓存更新策略的最佳实践方案<br />1.）低一致性需求：使用Redis自带的内存淘汰机制<br />2.）高一致性需求：主动更新，并以超时剔除作为兜底方案

- 读操作
   - 缓存命中则直接返回
   - 缓存未命中则查询数据库，并写入缓存，设定超时时间
- 写操作
   - 先写数据库，然后再删除缓存
   - 要确保数据库与缓存操作的原子性（写操作要在一个事务里）
### 案例
给查询商铺的缓存添加超时剔除和主动更新的策略

- 根据id查询店铺时，如果缓存未命中，则查询数据库，将数据库结果写入缓存，并设置超时时间
- 根据id修改店铺时，先修改数据库，再删除缓存（**添加事务！**）

1）查询商铺，在上一小节的基础上增加一个TTL即可
```java
// 3.2 存在商铺，写入Redis，返回商铺信息
// 超时剔除策略
stringRedisTemplate.opsForValue()
        .set(shopKey, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
```
2）修改店铺
```java
@Override
@Transactional
public Result updateShop(Shop shop) {
    Long shopId = shop.getId();
    if (shopId == null) {
        return Result.fail("店铺id不能为空");
    }
    // 1 更新数据库
    updateById(shop);
    // 2 删除缓存
    stringRedisTemplate.delete(CACHE_SHOP_KEY + shopId);
    return Result.ok();
}
```
测试<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694536289070-7fae89bc-88ec-42ed-9dd7-991a8efac8c5.png#averageHue=%23fbfafa&clientId=u74749200-d9d1-4&from=paste&height=243&id=u914ec4f8&originHeight=421&originWidth=1284&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=46382&status=done&style=none&taskId=u3414a1af-196d-4075-a283-8ff460f7f18&title=&width=741.1255594837855)
## 缓存穿透
客户端请求的数据在缓存和数据库中都不存在，这样缓存永远不生效，请求都会打到数据库里

常见的解决方案：

- 缓存空对象✔️
   - 优点：简单
   - 缺点：
      - 额外的内存消耗（解决：设置TTL）
      - 短期的数据不一致（解决：真正插入的时候覆盖）
- 布隆过滤
   - 优点：内存占用少，没有多余key
   - 缺点：
      - 实现复杂
      - 存在误判可能
> 上面都是一些被动的解决方案，主动的解决方案（预防）：
> - 增强id的复杂度
> - 增强对请求数据的格式校验
> - 增强用户权限校验
> - 热点参数限流

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694537402355-8fa90427-83d2-44ca-b528-c68be4e2e57e.png#averageHue=%23f9f7f6&clientId=u74749200-d9d1-4&from=paste&height=283&id=u3673937c&originHeight=491&originWidth=891&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=154147&status=done&style=none&taskId=u431d0104-3220-41df-95b0-af18638fc87&title=&width=514.2857270249633)
### 案例
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694576401431-bf50cd76-a0bb-4a31-a840-2b6850c5f07a.png#averageHue=%23efecec&clientId=u5b7e824b-fefd-4&from=paste&height=354&id=uc91a7168&originHeight=613&originWidth=1440&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=389281&status=done&style=none&taskId=u26c3768d-6c9b-41d9-b0b2-ed78a916358&title=&width=831.1688517575166)<br />id查询商铺
```java
@Override
public Result queryShopById(Long id) {
    // 1 查询缓存
    String shopKey = CACHE_SHOP_KEY + id;
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    // 2 命中直接返回
    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return Result.ok(shop);
    }
    // ===== 缓存穿透，命中为空值""，返回错误
    if(shopJson!=null){
        return Result.fail("商户不存在");
    }
    // 3 未命中查数据库
    Shop shop = getById(id);
    // 3.1 不存在，返回404
    if (shop == null) {
        // ===== 缓存穿透，将空值""写入Redis
        stringRedisTemplate.opsForValue()
                .set(shopKey, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
        return Result.fail("商户不存在");
    }
    // 3.2 存在商铺，写入Redis，返回商铺信息
    // ===== 超时剔除策略兜底
    stringRedisTemplate.opsForValue()
            .set(shopKey, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
    return Result.ok(shop);
}
```
测试：`[http://localhost:8081/shop/0](http://localhost:8081/shop/0)`
## 缓存雪崩
在同一时段大量的缓存key同时失效或者Redis服务宕机导致大量请求到达数据库

解决方案：

- 给不同key的TTL添加随机值
- Redis集群（针对宕机）
- 给缓存业务添加降级限流策略（缓存全崩完了）
- 业务添加多级缓存（浏览器缓存、nginx缓存、jvm本地缓存）
## 缓存击穿
也叫热点Key问题，一个被高并发访问并且缓存重建业务较复杂（重建时间较长）的key突然失效了，无数的请求会瞬间给数据库带来巨大的压力<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694584838789-4082b818-2705-40a5-b956-bdbb64e81cac.png#averageHue=%23eeeceb&clientId=u5b7e824b-fefd-4&from=paste&height=322&id=u331ea91b&originHeight=558&originWidth=914&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=192287&status=done&style=none&taskId=uca4793c1-2e63-4cb9-82d4-621e0ea364f&title=&width=527.5613406294237)

解决方案（根据需求选择）：

|  | 优点 | 缺点 |
| --- | --- | --- |
| 互斥锁 | <br />- 没有额外的内存开销<br />- 保证**一致性**<br />- 实现简单<br /> | <br />- 线程要等待，性能受影响<br />- 有死锁风险<br /> |
| 逻辑过期（由后端添加一个expire字段） | <br />- 线程无需等待，**性能较好**<br /> | <br />- 不保证一致性<br />- 有额外的内存开销<br />- 实现复杂<br /> |

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694585382353-eecf2dee-e80c-49b4-aa09-8d72c804487f.png#averageHue=%23ece8e7&clientId=u5b7e824b-fefd-4&from=paste&height=431&id=u7ca2e247&originHeight=746&originWidth=1468&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=469782&status=done&style=none&taskId=u1a402659-452b-4c6f-ba52-098cc1579a5&title=&width=847.3304683194682)
### 互斥锁案例
 ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694587911788-f9e41e90-9a18-46c9-818a-60b60c1082ce.png#averageHue=%23f7f2f2&clientId=ud0ff78dd-bf30-4&from=paste&height=320&id=uf92cabf6&originHeight=554&originWidth=749&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=181673&status=done&style=none&taskId=u7a49fa2c-fb2d-43be-8913-0ba8403cec0&title=&width=432.32324303220827)<br />利用 Redis 的操作String类型的命令`SETNX`（添加一个String类型的键值对，前提是这个key不存在，否则不执行）作为互斥锁
```java
@Override
public Result queryShopById(Long id) {
    Shop shop = queryWithMutex(id);
    if (shop == null) {
        return Result.fail("商铺不存在");
    }
    return Result.ok(shop);
}

/**
 * id查询商铺——缓存击穿解决
 *
 * @param id
 * @return
 */
public Shop queryWithMutex(Long id) {
    // 1 查询缓存
    String shopKey = CACHE_SHOP_KEY + id;
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    // 2 命中直接返回
    if (StrUtil.isNotBlank(shopJson)) {
        Shop shop = JSONUtil.toBean(shopJson, Shop.class);
        return shop;
    }
    // <<缓存穿透>> 命中为空值""，返回错误
    if (shopJson != null) {
        return null;
    }
    // 3 未命中，实现缓存重建
    String lockKey = LOCK_SHOP_KEY + id;
    Shop shop = null;
    try {
        // 3.1 <<缓存击穿>> 获取互斥锁
        boolean isLock = tryLock(lockKey);
        if (!isLock) {
            // 3.2 <<缓存击穿>> 未获取到，等待，递归调用
            Thread.sleep(50);
            return queryWithMutex(id);
        }
        // 3.3 获取到，查数据库
        shop = getById(id);
        // <<缓存击穿>> 模拟重建的延迟
        Thread.sleep(200);
        // 3.3.1 不存在商铺
        if (shop == null) {
            // <<缓存穿透>> 将空值""写入Redis
            stringRedisTemplate.opsForValue()
                    .set(shopKey, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
        // 3.3.2 存在商铺，写入Redis，返回商铺信息
        // <<超时剔除策略兜底>> CACHE_SHOP_TTL
        stringRedisTemplate.opsForValue()
                .set(shopKey, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    } finally {
        // <<缓存击穿>> 释放锁
        unLock(lockKey);
    }
    return shop;
}

/**
 * 获取锁
 *
 * @param key
 * @return
 */
private boolean tryLock(String key) {
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
    // 不要直接返回flag，flag自动拆箱可能会空指针异常
    return BooleanUtil.isTrue(flag);
}

/**
 * 释放锁
 *
 * @param key
 */
private void unLock(String key) {
    stringRedisTemplate.delete(key);
}
```
测试：`[http://localhost:8081/shop/](http://localhost:8081/shop/0)3`<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694595004445-b827085b-5cc1-46f5-9d13-747a18cc4ea6.png#averageHue=%23f6f5f4&clientId=ud0ff78dd-bf30-4&from=paste&height=248&id=u7da44f8d&originHeight=430&originWidth=947&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=66376&status=done&style=none&taskId=uf6c41d5e-2cbb-4d65-8aca-e087e240ee1&title=&width=546.6089601488668)
### 逻辑过期案例
> 需要把数据提前写入Redis，不然永远命中不了

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694595772240-cf56e0b6-40a7-4217-8988-9d724d36a0c2.png#averageHue=%23f8f4f3&clientId=ud0ff78dd-bf30-4&from=paste&height=326&id=ua3f08a00&originHeight=564&originWidth=1086&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=245019&status=done&style=none&taskId=uc17e96cd-9d90-4ffd-8cf0-7daca7e326b&title=&width=626.8398423671271)<br />测试类提前插入过期数据
```java
@Test
public void testSaveShopToRedis() throws InterruptedException {
    // saveShopToRedis是ShopServiceImpl的新方法
    // 插入一条10s过期的测试数据
    shopService.saveShopToRedis(1L, 10L);
}
```
业务逻辑
```java
// 线程池
private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

/**
 * id查询商铺——缓存击穿解决——逻辑过期
 *
 * @param id
 * @return
 */
public Shop queryWithLogicalExpire(Long id) {
    String shopKey = CACHE_SHOP_KEY + id;
    // # 查询缓存
    String shopJson = stringRedisTemplate.opsForValue().get(shopKey);
    // ## 未命中，返回空
    if (StrUtil.isBlank(shopJson)) {
        return null;
    }
    // # 命中，判断缓存是否过期
    RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
    Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);
    // 获取逻辑过期时间
    LocalDateTime expireTime = redisData.getExpireTime();
    // ## 未过期，返回shop
    if (expireTime.isAfter(LocalDateTime.now())) {
        return shop;
    }
    // # 过期，获取互斥锁
    boolean isLock = tryLock(LOCK_SHOP_KEY + id);
    // ## 获取到锁，开启独立线程重建缓存
    if (isLock) {
        CACHE_REBUILD_EXECUTOR.submit(() -> {
            try {
                // 缓存重建
                this.saveShopToRedis(id, 20L);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                // 释放锁
                unLock(LOCK_SHOP_KEY + id);
            }
        });
    }
    // ## 没获取到锁，返回过期shop
    return shop;
}

// 重建缓存
public void saveShopToRedis(Long id, Long expireSeconds) throws InterruptedException {
    // # 查询shop
    Shop shop = getById(id);
    // 模拟延时
    Thread.sleep(200);
    // # 封装逻辑过期时间
    RedisData redisData = new RedisData();
    redisData.setData(shop);
    redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireSeconds));
    // # 写入Redis，不要设置TTL
    stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(redisData));
}

/**
 * 获取锁
 */
private boolean tryLock(String key) {
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
    // 不要直接返回flag，flag自动拆箱可能会空指针异常
    return BooleanUtil.isTrue(flag);
}

/**
 * 释放锁
private void unLock(String key) {
    stringRedisTemplate.delete(key);
}
```
## 缓存工具封装
> [https://www.bilibili.com/video/BV1cr4y1671t/?p=46&spm_id_from=pageDriver&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa](https://www.bilibili.com/video/BV1cr4y1671t/?p=46&spm_id_from=pageDriver&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa)

基于StringRedisTemplate封装一个缓存工具类，满足下列需求:<br />方法1：将任意Java对象序列化为json并存储在string类型的key中，并且可以设置TTL过期时间<br />方法2：将任意]ava对象序列化为ison并存储在string类型的key中，并且可以设置**逻辑过期**时间，用于处理缓存击穿问题<br />方法3：根据指定的key查询缓存，并反序列化为指定类型，利用**缓存空值的方式解决缓存穿透问题**<br />方法4：根据指定的key查询缓存，并反序列化为指定类型，需要利用逻辑过期解决缓存击穿问题

技术点：

1. 泛型
2. 函数式编程

工具类
```java
@Slf4j
@Component
public class CacheClient {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    // 线程池
    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool(10);

    /**
     * Java对象序列化为json并存储在string类型的key中，并且可以设置TTL过期时间
     *
     * @param key
     * @param value
     * @param time
     * @param unit
     */
    public void set(String key, Object value, Long time, TimeUnit unit) {
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value), time, unit);
    }

    /**
     * 将任意]ava对象序列化为ison并存储在string类型的key中
     * 并且可以设置逻辑过期时间，用于处理缓存击穿问题
     *
     * @param key
     * @param value
     * @param time
     * @param unit
     */
    public void setWithLogicalExpire(String key, Object value, Long time, TimeUnit unit) {
        RedisData redisData = new RedisData();
        redisData.setData(value);
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(unit.toSeconds(time)));
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
    }

    /**
     * 根据指定的key查询缓存，并反序列化为指定类型
     * 利用缓存空值的方式解决缓存穿透问题
     *
     * @param keyPrefix
     * @param id
     * @param type       返回值类型
     * @param dbFallback id查询数据库逻辑，需实现dbFallback接口
     * @param time       缓存保存时间
     * @param unit       时间单位
     * @param <R>
     * @param <ID>
     * @return
     */
    public <R, ID> R queryWithPassThrough(
            String keyPrefix,
            ID id,
            Class<R> type,
            Function<ID, R> dbFallback,
            Long time, TimeUnit unit) {

        // # Redis查询缓存
        String key = keyPrefix + id;
        String json = stringRedisTemplate.opsForValue().get(key);
        // ## 命中且值非空，直接返回
        if (StrUtil.isNotBlank(json)) {
            return JSONUtil.toBean(json, type);
        }
        // ## 命中为空值""，返回null
        if (json != null) {
            return null;
        }
        // # 未命中查数据库
        R r = dbFallback.apply(id);
        // ## 不存在，返回null
        if (r == null) {
            // 将空值""写入Redis
            stringRedisTemplate.opsForValue()
                    .set(key, "", CACHE_NULL_TTL, TimeUnit.MINUTES);
            return null;
        }
        // # 存在商铺，写入Redis，返回商铺信息
        // 超时剔除策略兜底
        this.set(key, r, time, unit);
        return r;
    }


    public <R, ID> R queryWithLogicalExpire(
            String keyPrefix,
            ID id,
            Class<R> type,
            Function<ID, R> dbFallback,
            Long time, TimeUnit unit) {

        String key = keyPrefix + id;
        // # 查询缓存
        String json = stringRedisTemplate.opsForValue().get(key);
        // ## 未命中，返回空
        if (StrUtil.isBlank(json)) {
            return null;
        }
        // # 命中，判断缓存是否过期
        RedisData redisData = JSONUtil.toBean(json, RedisData.class);
        R r = JSONUtil.toBean((JSONObject) redisData.getData(), type);
        // 获取逻辑过期时间
        LocalDateTime expireTime = redisData.getExpireTime();
        // ## 未过期，返回r
        if (expireTime.isAfter(LocalDateTime.now())) {
            return r;
        }
        // # 过期，获取互斥锁
        String lockKey = LOCK_SHOP_KEY + id;
        boolean isLock = tryLock(lockKey);
        // ## 获取到锁，开启独立线程重建缓存
        if (isLock) {
            CACHE_REBUILD_EXECUTOR.submit(() -> {
                try {
                    // 查数据库
                    R r1 = dbFallback.apply(id);
                    // 存入Redis
                    this.setWithLogicalExpire(key, r1, time, unit);
                } catch (Exception e) {
                    throw new RuntimeException(e);
                } finally {
                    // 释放锁
                    unLock(lockKey);
                }
            });
        }
        // ## 没获取到锁，返回过期r
        return r;
    }

    /**
     * 获取锁
     *
     * @param key
     * @return
     */
    private boolean tryLock(String key) {
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, "1", 10, TimeUnit.SECONDS);
        // 不要直接返回flag，flag自动拆箱可能会空指针异常
        return BooleanUtil.isTrue(flag);
    }

    /**
     * 释放锁
     *
     * @param key
     */
    private void unLock(String key) {
        stringRedisTemplate.delete(key);
    }
}

```
使用工具类
```java
@Override
public Result queryShopById(Long id) {
    // 解决缓存穿透
    /*Shop shop = cacheClient.queryWithPassThrough(
            CACHE_SHOP_KEY,
            id,
            Shop.class,
            // shopId -> getById(shopId),
            this::getById,
            CACHE_SHOP_TTL,
            TimeUnit.MINUTES
    );*/

    // 互斥锁解决缓存击穿
    // Shop shop = queryWithMutex(id);

    // 逻辑过期解决缓存击穿
    Shop shop = cacheClient.queryWithLogicalExpire(
            CACHE_SHOP_KEY,
            id,
            Shop.class,
            // shopId -> getById(shopId),
            this::getById,
            CACHE_SHOP_TTL,
            TimeUnit.MINUTES
    );

    if (shop == null) {
        return Result.fail("商铺不存在");
    }
    return Result.ok(shop);
}
```

# 优惠券秒杀（分布式锁）
##  全局唯一ID
当用户抢购时，就会生成订单并保存到tb_voucher_order这张表中，而订单表如果使用数据库自增ID就存在一些问题：

- id的规律性太明显，容易透露给用户信息（比如一天产生了多少单）
- 受单表数据量的限制，每天产生的订单都很多，分多张表存储，ID会重复

全局ID生成器：一种在分布式系统下用来生成全局唯一ID的工具，一般要满足以下特性：

- 唯一性
- 高可用：任何时候都不能挂
- 高性能：生成速度要快
- 递增性：要确保全局逐渐增大，有利于数据库创建索引，提高插入的速度
- 安全性：id规律性不能太明显
> 一些全局ID生成策略：
> - UUID（16进制字符串，缺少单调递增的特性）
> - Redis自增
> - snowflake算法（比较依赖时钟）
> - 数据库自增（专门弄一张表来自增，性能不如Redis自增）


为了增加ID的安全性，我们可以不直接使用Redis自增的数值，而是拼接一些其他信息<br />思想：时间戳+计数器<br />ID的组成部分：

- 符号位：1bit，永远为0
- 时间戳：31bit，以秒为单位，可以使用69年
- 序列号：32bit，秒内的计数器，支持每秒产生2^32个不同ID

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694623693347-ac625f6a-f79e-4e10-aabd-2970f1a1554f.png#averageHue=%23f8f2f1&clientId=u1d3a7e9d-c1c7-4&from=paste&height=116&id=ueb76dcb9&originHeight=201&originWidth=1160&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=79621&status=done&style=none&taskId=u3a6d22e0-1094-4420-8976-fc34c2131a3&title=&width=669.5526861379994)

使用 Redis 的 `Incr` 命令，可以实现后 32 位的原子性递增。<br />Redis的key设计为`icr:业务前缀:当前日期`，每天都会从1开始生成序列号，每天一个key方便统计订单量<br />单key设计可能会出现生成序列号数溢出 2^32 的情况，多key的话单日不可能超过2^32次方个订单
```java
@Component
public class RedisIdWorker {
    private static final long BEGIN_TIMESTAMP = 1672531200L;

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final int COUNT_BITS = 32;

    public long nextId(String keyPrefix) {
        // # 生成时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timeStamp = nowSecond - BEGIN_TIMESTAMP;
        // # 生成序列号
        // ## 获取当前日期
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        // ## 序列号自增长
        Long count = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);
        // # 拼接
        return timeStamp << COUNT_BITS | count;
    }

    public static void main(String[] args) {
        // 生成起始时间
        LocalDateTime time = LocalDateTime.of(2023, 1, 1, 0, 0, 0);
        long second = time.toEpochSecond(ZoneOffset.UTC);
        System.out.println("second=" + second);
    }
}
```
```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class HmDianPingApplicationTests {
    @Resource
    private RedisIdWorker redisIdWorker;

    private ExecutorService es = Executors.newFixedThreadPool(500);

    @Test
    public void testRedisIdWorker() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(300);

        Runnable task = () -> {
            for (int i = 0; i < 100; i++) {
                long id = redisIdWorker.nextId("testOrder");
                System.out.println("id=" + id);
            }
            // 减少计数器
            latch.countDown();
        };

        long begin = System.currentTimeMillis();
        // 共生成30k个全局id
        for (int i = 0; i < 300; i++) {
            es.submit(task);
        }
        // 挂起线程，计数器为0的时候恢复线程
        latch.await();
        long end = System.currentTimeMillis();
        System.out.println("time=" + (end - begin));
    }
}
```
## 优惠券秒杀下单实现
每个店铺都可以发布优惠券，分为平价券和特价券。平价券可以任意购买，而特价券需要秒杀抢购<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694702580019-c47f2a7e-1b00-493b-a2d6-266ed8b1bee6.png#averageHue=%23edeeef&clientId=u7b3a1905-9ec7-4&from=paste&height=109&id=uc753ebd1&originHeight=189&originWidth=1178&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=121607&status=done&style=none&taskId=ue0570866-0c0e-4e54-8840-6e245f8afec&title=&width=679.9422967849683)<br />表关系如下:<br />`tb voucher`：优惠券的基本信息，优惠金额、使用规则等<br />`tb seckill voucher`：优惠券的库存、开始抢购时间，结束抢购时间。特价优惠券才需要填写这些信息

数据库插入一条秒杀券的数据<br />post：`[http://localhost:8081/voucher/seckill](http://localhost:8081/voucher/seckill)`
```
{
    "shopId":1,
    "title":"100元代金券",
    "subTitle":"周一至周五均可使用",
    "rules":"全场通用\\n无需预约\\n可无限叠加\\不兑现、不找零\\n仅限堂食",
    "payValue":8000,
    "actualValue":10000,
    "type":1,
    "stock":100,
    "beginTime":"2023-09-10T10:09:17",
    "endTime":"2023-12-26T12:09:04"
}
```

实现优惠券秒杀的下单功能的基本实现<br />下单时要判断两点：

1. 秒杀是否开始或者结束
2. 库存是否充足

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694704985970-e6c3618b-e8c0-4a18-9ab8-ad62b2851d3d.png#averageHue=%23fbf8f8&clientId=u7b3a1905-9ec7-4&from=paste&height=298&id=u392eda0e&originHeight=516&originWidth=855&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=130623&status=done&style=none&taskId=ub680e5ce-6d5d-4afe-8572-e3f648d49be&title=&width=493.5065057310254)
```java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService voucherService;
    @Resource
    private RedisIdWorker redisIdWorker;

    @Override
    public Result seckillVoucher(Long voucherId) {
        // # 查询优惠券
        SeckillVoucher voucher = voucherService.getById(voucherId);
        // # 判断秒杀时间是否开始
        if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
            return Result.fail("时间未开始");
        }
        // # 判断秒杀时间是否结束
        if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
            return Result.fail("时间已结束");
        }
        // # 判断库存是否充足
        if (voucher.getStock() < 1) {
            return Result.fail("库存不足");
        }
        // # 抽检库存
        voucher.setStock(voucher.getStock() - 1);
        voucherService.updateById(voucher);
        // # 创建订单
        VoucherOrder voucherOrder = new VoucherOrder();
        // ## 订单id
        long orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        // ## 用户id
        voucherOrder.setUserId(UserHolder.getUser().getId());
        // ## 代金券id
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);
        // # 返回订单id
        return Result.ok(orderId);
    }
}
```
## 超卖问题（乐观锁）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694713344523-ab9defa6-a816-4321-ab07-c2e5a0f79290.png#averageHue=%23f3f2f2&clientId=u7b3a1905-9ec7-4&from=paste&height=371&id=ufd74d508&originHeight=643&originWidth=1269&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=168147&status=done&style=none&taskId=u4236f6f8-bcc8-45d3-a1f2-076e0efed16&title=&width=732.4675506113115)

  超卖问题是典型的多线程安全问题，针对这一问题的常见解决方案就是加锁

- 乐观锁：不加锁，在更新时判断是否有其他线程修改
   - 性能好，存在成功率低的问题
- 悲观锁：添加同步锁，让线程串行执行
   - 简单粗暴，性能一般

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694713542990-22f19e1f-9c77-4a0f-b0a1-236c6a0d4ddd.png#averageHue=%23f2f0f0&clientId=u7b3a1905-9ec7-4&from=paste&height=304&id=uccbdec90&originHeight=526&originWidth=1256&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=231511&status=done&style=none&taskId=ube4c43b0-6ebd-4402-9c8a-2f8f5d578bd&title=&width=724.9639429218339)

乐观锁的关键是判断之前查询得到的数据是否又被修改过，常见的方式有两种

1. 版本号法
2. CAS法（直接将stock做为版本号）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694714022439-75603a08-eab4-4ca1-a54c-a3b72d770da0.png#averageHue=%23f9f5f4&clientId=u7b3a1905-9ec7-4&from=paste&height=332&id=u14664a26&originHeight=575&originWidth=1419&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=199860&status=done&style=none&taskId=ubee20e0e-356a-450e-9463-27357bf0270&title=&width=819.0476393360527)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694714255754-e0680a6a-f2a9-44e2-9b21-ca4a75da6b0f.png#averageHue=%23faf8f8&clientId=u7b3a1905-9ec7-4&from=paste&height=325&id=u0f03097e&originHeight=563&originWidth=1393&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=159846&status=done&style=none&taskId=ue9912aec-4288-4732-a18d-ff775c22997&title=&width=804.0404239570976)

CAS法解决超卖问题
```java
// # 判断库存是否充足
if (voucher.getStock() < 1) {
    return Result.fail("库存不足");
}
// # 抽检库存
boolean success = seckillVoucherService.update()
        .setSql("stock=stock-1")
        .eq("voucher_id", voucherId)
        .gt("stock",0).update(); // 乐观锁
if (!success) {
    return Result.fail("库存不足");
}
```
## 一人一单（悲观锁）
### 单机实现
同一个特惠优惠券，一个用户只能下一单<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694753880224-beabe84e-2080-4404-b898-ba04e6580302.png#averageHue=%23faf8f8&clientId=u4105754c-416e-4&from=paste&height=302&id=u97781aa2&originHeight=524&originWidth=1140&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=168995&status=done&style=none&taskId=u2f4655e0-1eae-42d6-bc90-06d29cd0946&title=&width=658.008674308034)

以上方案存在线程安全问题，可能同一个用户多个线程都未创建过订单，并行执行判断订单是否存在，结果是都不存在，还是出现一人多单<br />解决方案：加锁

注意事项：<br />1）synchronized的同步对象是this，将userId作为同步对象，给每个用户加锁<br />2）Long的`toString()`内部还是会返回一个新的字符串对象，`userId.toString()`锁不上。`userId.toString().intern()`作为同步对象，可以确保返回的都是同一个String对象
> `intern()`会去字符串常量池里找和当前字符串一样的字符串对象的引用，并返回

3）确保事务提交了之后，才释放锁！<br />4）Spring的事务是通过拿到`VoucherOrderServiceImpl`的代理对象操作的，而下面这段代码中调用`createOrder`的对象是this，不是`VoucherOrderServiceImpl`的代理对象，事务会失效
```java
synchronized (userId.toString().intern()) {
    return createOrder(voucherId);
}
```
允许暴露代理对象
```xml
<!--aspectJ-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```
```java
// exposeProxy 是否暴露代理对象
@EnableAspectJAutoProxy(exposeProxy = true)
@MapperScan("com.hmdp.mapper")
@SpringBootApplication
public class HmDianPingApplication {
    public static void main(String[] args) {
        SpringApplication.run(HmDianPingApplication.class, args);
    }
}
```
获取代理对象
```java
synchronized (userId.toString().intern()) {
    // 获取代理对象（事务）
    IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
    return proxy.createVoucherOrder(voucherId);
}
```

完整实现
```java
@Service
public class VoucherOrderServiceImpl extends ServiceImpl<VoucherOrderMapper, VoucherOrder> implements IVoucherOrderService {

    @Resource
    private ISeckillVoucherService seckillVoucherService;
    @Resource
    private RedisIdWorker redisIdWorker;

    @Override
    public Result seckillVoucher(Long voucherId) {
        // ......
        
        // # 给每个用户上锁，订单创建完，事务提交了才释放锁
        synchronized (userId.toString().intern()) {
            // 获取代理对象（事务）
            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
            return proxy.createVoucherOrder(voucherId);
        }
    }

    @Transactional
    public Result createVoucherOrder(Long voucherId) {
        Long userId = UserHolder.getUser().getId();
        // # 一人一单，判断当前用户订单是否存在
        int count = query()
                .eq("user_id", userId)
                .eq("voucher_id", voucherId)
                .count();
        if (count > 0) {
            return Result.fail("用户已经购买过一次");
        }
        // # 抽检库存
        boolean success = seckillVoucherService.update()
                .setSql("stock=stock-1")
                .eq("voucher_id", voucherId)
                .gt("stock", 0).update(); // 乐观锁
        if (!success) {
            return Result.fail("库存不足");
        }
        // # 创建订单
        VoucherOrder voucherOrder = new VoucherOrder();
        // ## 订单id
        long orderId = redisIdWorker.nextId("order");
        voucherOrder.setId(orderId);
        // ## 用户id
        voucherOrder.setUserId(userId);
        // ## 代金券id
        voucherOrder.setVoucherId(voucherId);
        save(voucherOrder);
        // # 返回订单id
        return Result.ok(orderId);
    }

}
```
### 分布式或集群模式下的问题
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694783574756-7214b7a8-37d1-41bc-b297-0c623de73a29.png#averageHue=%23efefef&clientId=u475f6ad9-c04f-4&from=paste&height=315&id=ua5d1376e&originHeight=545&originWidth=977&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=277918&status=done&style=none&taskId=ub735611f-62e6-416c-b789-bc1dd7794a1&title=&width=563.924977893815)<br />端口号设置<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694783530242-df03898f-f67d-437a-9756-4c04e51c44c2.png#averageHue=%232c2f34&clientId=u475f6ad9-c04f-4&from=paste&height=515&id=u84a04d19&originHeight=893&originWidth=953&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=86684&status=done&style=none&taskId=u0a9b8f7f-31de-4dba-8762-8fe02d96f50&title=&width=550.0721636978565)

访问`[http://localhost:8080/api/voucher/list/1](http://localhost:8080/api/voucher/list/1)`，nginx负载均衡转发给8081和8082的服务，同一个用户请求不同的服务，都能拿到锁<br />Synchronized 关键字只对单个 JVM 有效，多机部署时不同的JVM的锁已经不一样了（8080和8081的线程可以同时获取锁）<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694786792380-4f881ca6-49c3-47eb-8178-f3e727dca4d1.png#averageHue=%23eeeceb&clientId=u475f6ad9-c04f-4&from=paste&height=376&id=u387ced87&originHeight=652&originWidth=1461&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=412858&status=done&style=none&taskId=ua7a37fcf-6254-41a1-86b4-a90083319f9&title=&width=843.2900641789803)

### 分布式锁
满足分布式系统或集群模式下多进程可见并且互斥的锁<br />特性：

- 多进程可见
- 互斥
- 高可用
- 高性能
- 安全性：服务挂了锁没释放、死锁

常见的分布式锁实现<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694790963910-6afd8422-cf5a-4763-b684-dac3f97cf6a9.png#averageHue=%23d7c1c0&clientId=u475f6ad9-c04f-4&from=paste&height=284&id=u3575735b&originHeight=492&originWidth=1362&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=187855&status=done&style=none&taskId=u38d1a295-597e-4b4b-86ad-bf137f13ddc&title=&width=786.1472056206511)

### Redis实现分布式锁
获取分布式锁需要实现两个基本方法<br />1）获取锁

1. 利用setnx的互斥性，`SETNX lock xxx`
2. 添加过期时间 `EXPIRE lock xxx`

两个操作要确保原子性 `SET lock xxx EX xxx NX`（NX：不存在才可以set）
> `Boolean setIfAbsent(K key, V value, long timeout, TimeUnit unit)`

非阻塞：尝试一次，成功返回true，失败返回false<br />2）释放锁<br />手动释放，`DEL lock`<br />超时释放<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694793876489-8dec06e8-1b57-42df-8135-2ee6c0cb0281.png#averageHue=%23f8f7f7&clientId=u475f6ad9-c04f-4&from=paste&height=352&id=u5ec22bd3&originHeight=609&originWidth=1305&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=255802&status=done&style=none&taskId=u3cb254fc-515a-4e9b-89fb-233d477c901&title=&width=753.2467719052494)
### 两种误删锁情况
> 解决方法：线程标识、lua保证原子性

**误删问题一：线程校验**<br />当业务阻塞时间过长会导致锁被提前释放，线程2可以获取锁，线程1执行完业务后会误放线程2的锁，因为是同一个用户的多个线程，锁的key是一样的<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694797795051-dd010bc4-5d40-48ca-b269-9d373e2bee01.png#averageHue=%23fafaf9&clientId=u475f6ad9-c04f-4&from=paste&height=314&id=u2d8e82d9&originHeight=544&originWidth=1359&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=160500&status=done&style=none&taskId=ucd62c3d8-4061-4571-9d2f-8b6c76b0a0c&title=&width=784.4156038461563)<br />解决方案：<br />1）获取锁的时候，将value设置为当前线程标识（UUID+线程id），两个JVM的线程id可能会冲突<br />2）释放锁的时候判断锁的线程标识是否和当前线程是一致的，一致才释放，避免误删<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694798832694-c85c1d91-f613-4a88-bddc-3d5b410c8274.png#averageHue=%23faf9f8&clientId=u475f6ad9-c04f-4&from=paste&height=319&id=ue99463aa&originHeight=552&originWidth=1362&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=198498&status=done&style=none&taskId=uc72e7025-03a2-4872-96b7-bd5352f03d2&title=&width=786.1472056206511)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694798897393-c1d36b0b-9616-4fd4-9127-a2c1c39dece8.png#averageHue=%23edecec&clientId=u475f6ad9-c04f-4&from=paste&height=344&id=u0c1a6bbd&originHeight=596&originWidth=573&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=122561&status=done&style=none&taskId=u6639877a-284e-4832-8434-afbc47c043b&title=&width=330.7359389285118)

**误删问题二：判断锁标识和释放锁的原子性**<br />线程1判断锁标识和当前线程一致后，出现业务阻塞（gc垃圾回收）导致没有及时释放锁，锁超时释放了以后，线程2成功获取锁，执行业务，线程1又把线程2的锁释放了，线程3有能获取锁执行业务<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694882900476-70bff775-9f0c-4144-8bd5-62c97feac5c2.png#averageHue=%23f6f4f4&clientId=u4124065c-0985-4&from=paste&height=462&id=ud1da0777&originHeight=801&originWidth=1898&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=524708&status=done&style=none&taskId=u0ec19670-05e0-4e34-976a-4e1496c157a&title=&width=1095.5267226637266)<br />解决方案：<br />Redis提供了Lua脚本功能，在一个脚本中编写多条Redis命令，确保多条命令执行时的原子性。Lua是一种编程语言，它的基本语法大家可以参考网站: [https://www.runoob.com/lua/lua-tutorial.html](https://www.runoob.com/lua/lua-tutorial.html)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694916769809-02275319-23a5-48f9-a572-3e86409476b4.png#averageHue=%23dee4df&clientId=u6c8c1437-1b33-4&from=paste&height=287&id=uf22a1964&originHeight=720&originWidth=1153&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=408871&status=done&style=none&taskId=ubcde4c0e-f80e-4fb1-bca3-034fdf4e899&title=&width=459.98516845703125)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694936406217-fad8b853-e701-4e97-83c1-2e4a1f2a66f3.png#averageHue=%23d9dcd1&clientId=u6c8c1437-1b33-4&from=paste&height=560&id=u4129a50d&originHeight=971&originWidth=2027&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=1011701&status=done&style=none&taskId=u1008cf4d-a120-4138-9ade-d4df61898b0&title=&width=1169.985598967004)<br />lua脚本实现锁的业务流程<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694937611623-17b7a006-3f6b-4bc8-8e36-cb2611899d66.png#averageHue=%23dcddd8&clientId=u6c8c1437-1b33-4&from=paste&height=410&id=u309ddd8d&originHeight=711&originWidth=1468&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=679155&status=done&style=none&taskId=uce02245d-5f8b-4984-83b1-38152b04904&title=&width=847.3304683194682)<br />RedisTemplate调用lua脚本的api<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1694937941095-e3b26303-bd39-4727-b1ab-b4540f0e0684.png#averageHue=%23eaefe8&clientId=u6c8c1437-1b33-4&from=paste&height=380&id=u1a33b13d&originHeight=658&originWidth=1478&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=677877&status=done&style=none&taskId=u5036d850-ab32-45f6-bdf6-ddef6789e9c&title=&width=853.102474234451)
### 完整实现
1）Redis分布式锁实现类
```java
public class SimpleRedisLock implements ILock {

    private String name;
    private StringRedisTemplate stringRedisTemplate;

    private static final String KEY_PREFIX = "lock:";
    private static final String THREAD_ID_PREFIX = UUID.randomUUID().toString(true) + '-';
    private static final DefaultRedisScript<Long> UNLOCK_SCRIPT;

    static {
        UNLOCK_SCRIPT = new DefaultRedisScript<>();
        UNLOCK_SCRIPT.setLocation(new ClassPathResource("unlock.lua"));
        UNLOCK_SCRIPT.setResultType(Long.class);
    }

    public SimpleRedisLock(String name, StringRedisTemplate stringRedisTemplate) {
        this.name = name;
        this.stringRedisTemplate = stringRedisTemplate;
    }

    @Override
    public boolean tryLock(long timeoutSec) {
        // 获取线程标识
        String threadId = THREAD_ID_PREFIX + Thread.currentThread().getId();
        // 获取锁
        Boolean success = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, threadId, timeoutSec, TimeUnit.SECONDS);
        // 避免空指针
        return BooleanUtil.isTrue(success);
    }

    @Override
    public void unLock() {
        // 调用lua脚本
        stringRedisTemplate.execute(
                UNLOCK_SCRIPT,
                Collections.singletonList(KEY_PREFIX + name),
                THREAD_ID_PREFIX + Thread.currentThread().getId()
        );
    }

//    @Override
//    public void unLock() {
//        // 获取线程标识
//        String threadId = THREAD_ID_PREFIX + Thread.currentThread().getId();
//        // 获取锁的线程标识
//        String id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
//        // 判断标识是否一致
//        if (threadId.equals(id)) {
//            // 释放锁
//            stringRedisTemplate.delete(KEY_PREFIX + name);
//        }
//    }
}
```
2）业务逻辑
```java
@Override
public Result seckillVoucher(Long voucherId) {
    // # 查询优惠券
    SeckillVoucher voucher = seckillVoucherService.getById(voucherId);
    // # 判断秒杀时间是否开始
    if (voucher.getBeginTime().isAfter(LocalDateTime.now())) {
        return Result.fail("时间未开始");
    }
    // # 判断秒杀时间是否结束
    if (voucher.getEndTime().isBefore(LocalDateTime.now())) {
        return Result.fail("时间已结束");
    }
    // # 判断库存是否充足
    if (voucher.getStock() < 1) {
        return Result.fail("库存不足");
    }

    // # 给每个用户上锁，订单创建完，事务提交了才释放锁
    // 1、单机业务实现
//        Long userId = UserHolder.getUser().getId();
//        synchronized (userId.toString().intern()) {
//            // 获取代理对象（事务）
//            IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
//            return proxy.createVoucherOrder(voucherId);
//        }

    // 2、分布式锁实现
    Long userId = UserHolder.getUser().getId();
    // # 获取锁
    SimpleRedisLock lock = new SimpleRedisLock("order:" + userId,stringRedisTemplate);
    boolean isLock = lock.tryLock(12000);
    // # 获取失败
    if (!isLock){
        return Result.fail("不允许重复下单");
    }
    // # 执行业务逻辑
    try {
        // 获取代理对象（事务）
        IVoucherOrderService proxy = (IVoucherOrderService) AopContext.currentProxy();
        return proxy.createVoucherOrder(voucherId);
    }finally {
        lock.unLock();
    }
}
```

# Redission
## 介绍/能解决的问题
除了误删之外，现在的分布式锁实现还存在以下几个问题：

- 不可重入：同一个线程无法多次获取同一把锁（递归调用或调用的子函数抢同一把锁时就会出现死锁）
- 不可重试：获取锁只尝试一次就返回false，没有重试机制
- 超时释放：虽然可以避免死锁，但如果是业务执行耗时较长，也会导致锁释放，存在安全隐患
- 主从一致性：Redis提供了主从集群，主从同步存在延迟，主节点设置锁成功，还未及时同步到从节点，这时主节点宕机，从节点被选为主节点。但此时从节点还没有锁，仍可以抢锁成功。

Redisson是一个在Redis的基础上实现的Java驻内存数据网格(In-Memory Data Grid)。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务，其中就包含了各种分布式锁的实现。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1695218312227-32fe6c86-e972-4855-b1fc-dec8edc616cf.png#averageHue=%23fdfdfb&clientId=u79b0bede-31dd-4&from=paste&height=285&id=ufb49f2ba&originHeight=299&originWidth=698&originalType=binary&ratio=1.0499999523162842&rotation=0&showTitle=false&size=129333&status=done&style=none&taskId=uac252ca2-38ad-4093-95bd-95192fb6bbe&title=&width=664.7619349507802)<br />官网地址: [https://redisson.org](https://redisson.orgGitHub)<br />GitHub地址: [https://github.com/redisson/redisson](https://github.com/redisson/redisson)
## 入门
不建议引入 springboot-starter，因为可能会和 springboot 内置的 redis 整合冲突<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1695799841977-9a290f86-6aeb-48e6-bf0d-3c8e2b565940.png#averageHue=%23edf0ea&clientId=u3543a423-d590-4&from=paste&height=443&id=u01799d03&originHeight=465&originWidth=1053&originalType=binary&ratio=1.0499999523162842&rotation=0&showTitle=false&size=203487&status=done&style=none&taskId=u44a22ec2-f8f9-4700-a7e0-c86f41f456f&title=&width=1002.8571883999592)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1695799991333-e395866b-ed44-4c2d-bc42-63ceeaa5d81b.png#averageHue=%23edf0ea&clientId=u3543a423-d590-4&from=paste&height=590&id=u6678290e&originHeight=620&originWidth=1221&originalType=binary&ratio=1.0499999523162842&rotation=0&showTitle=false&size=330050&status=done&style=none&taskId=u36b08022-82e9-4637-a657-e36beb993cd&title=&width=1162.8571956660496)
## Redis分布式锁实现原理
> 没细看

Redisson分布式锁原理

- 可重入：利用hash结构记录线程id和重入次数
- 可重试：利用信号量和Pubsub功能实现等待、唤醒，获取锁失败的重试机制
- 超时续约：利用watchDog，每隔一段时间 (releaseTime/3)，重置超时时间

**调用api**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696083677982-4e753b27-bd6c-4f30-8cc4-d7cc38104370.png#averageHue=%23e4e7e0&clientId=ub26dbcfb-ae41-4&from=paste&height=130&id=b6epR&originHeight=219&originWidth=834&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=120262&status=done&style=none&taskId=ucd432535-bf2a-43a3-9e1e-306a903dca7&title=&width=493.54351806640625)<br />`waitTime`：在等待时间内不断重试<br />`leaseTime`：在锁失效时间内自动释放

**1）可重入实现**<br />保证同一个线程可以多次获取同一把锁<br />通过Redis的哈希结构实现

- `key`：锁名称
- `field`：线程标识
- `value`：重入次数

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696068174668-bd15acaf-530e-49d0-b77e-773b00ed0ab6.png#averageHue=%23f4f6f5&clientId=ub26dbcfb-ae41-4&from=paste&height=151&id=u34667898&originHeight=208&originWidth=468&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=55683&status=done&style=none&taskId=u95ec56d0-9709-468a-86bb-dec7f72f655&title=&width=340.3636363636364)

实现原理<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696068352563-d200e0ec-1f1c-4a04-a850-9db219a2c4e8.png#averageHue=%23eee6e5&clientId=ub26dbcfb-ae41-4&from=paste&id=u44450430&originHeight=954&originWidth=1973&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=837388&status=done&style=none&taskId=ua2b33100-988b-40a6-a7a3-79ff1461af2&title=)

获取锁的lua脚本<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696069064075-c469b5e3-cf6b-4524-ba82-2ede3bffaa86.png#averageHue=%23eef0ea&clientId=ub26dbcfb-ae41-4&from=paste&height=704&id=u5a1f41d2&originHeight=968&originWidth=1929&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=1093924&status=done&style=none&taskId=u4a618302-d617-49f3-b611-3921613598b&title=&width=1402.909090909091)<br />释放锁的lua脚本<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696069220422-cae40820-d15a-4b8a-8400-0278b5263851.png#averageHue=%23edefe9&clientId=ub26dbcfb-ae41-4&from=paste&height=692&id=u8a9aad92&originHeight=952&originWidth=1920&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=1050648&status=done&style=none&taskId=u1c53cb07-1486-4c95-9aad-4db85e2c83e&title=&width=1396.3636363636363)<br />测试可重入锁，导包junit5
```java
@SpringBootTest
@Slf4j
public class RedissionTest {
    @Resource
    private RedissonClient redissonClient;

    private RLock lock;

    @BeforeEach
    public void setUp() {
        lock = redissonClient.getLock("order");
    }

    @Test
    public void method1() {
//        boolean isLock = lock.tryLock();
//        lock = redissonClient.getLock("order");
        boolean isLock = lock.tryLock();
        if (!isLock) {
            log.error("获取锁失败1");
            return;
        }
        try {
            log.info("获取锁成功1");
            method2();
            log.info("开始执行业务1");
        } finally {
            log.warn("准备释放锁1");
            lock.unlock();
        }
    }

    public void method2() {
//        boolean isLock = lock.tryLock();
//        lock = redissonClient.getLock("order");
        boolean isLock = lock.tryLock();
        if (!isLock) {
            log.error("获取锁失败2");
            return;
        }
        try {
            log.info("获取锁成功2");
            log.info("开始执行业务2");
        } finally {
            log.warn("准备释放锁2");
            lock.unlock();
        }
    }
}
```

**2）可重试和超时续约**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696084534145-4962b023-e1c1-4819-9ba0-e66044c44e1d.png#averageHue=%23f9f6f5&clientId=ub26dbcfb-ae41-4&from=paste&height=561&id=ub8c75e81&originHeight=771&originWidth=1761&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=422189&status=done&style=none&taskId=u459238b5-e6a3-4779-bcdf-35d73ffb962&title=&width=1280.7272727272727)
## 主从一致性问题
主节点负责写操作，从结点只负责读操作，主节点挂了以后选取一个从结点作为主节点，但还没来得及同步锁<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696085628817-4fc9e071-8e49-46ef-a434-e8ad64abc645.png#averageHue=%23f6eaea&clientId=ub26dbcfb-ae41-4&from=paste&id=uee68e32d&originHeight=674&originWidth=1457&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=257197&status=done&style=none&taskId=u9b59f688-0f41-4f6c-99d1-704ef750610&title=)<br />解决方法：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功（可以不设置主从）<br />**利用Redisson的multiLock（联锁）可以实现，联锁可以看做是多个Redis可重入锁的集合**<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696086191239-cd102a81-44e8-444e-ada9-270dce527102.png#averageHue=%23f2dfde&clientId=ub26dbcfb-ae41-4&from=paste&height=492&id=u53b44615&originHeight=677&originWidth=1906&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=458220&status=done&style=none&taskId=u10244d9d-deb1-48b7-bc09-cc72b004d4e&title=&width=1386.1818181818182)<br />使用方法：<br />1）配置多个`RedissionClient`<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696133038035-31faf9de-bc42-4a44-9bb5-a56133553393.png#averageHue=%23eff4ed&clientId=u30c8dc24-1b16-4&from=paste&id=uf1d8f4f4&originHeight=723&originWidth=1264&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=437439&status=done&style=none&taskId=u0b26d7b7-301c-4a31-bc6d-c5814120e52&title=)<br />2）创建联锁<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696133225965-ff187be5-e941-4461-bcf6-f89789d98e3e.png#averageHue=%23eff4ee&clientId=u30c8dc24-1b16-4&from=paste&height=370&id=u860fdc15&originHeight=611&originWidth=1021&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=378606&status=done&style=none&taskId=ue826b52a-3f07-4cbe-b053-0408e01f9c9&title=&width=618.7878430229047)<br />3）获取锁和释放锁的方式和之前一样

## 总结
> 全都用lua脚本保证原子性

1）不可重入Redis分布式锁<br />原理：利用setnx的互斥性；利用ex避免死锁；释放锁时判断线程标示<br />缺陷：不可重入、无法重试、锁超时失效<br />2）可重入的Redis分布式锁<br />原理：利用hash结构，记录线程标示和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试等待<br />缺陷：redis宕机引起锁失效问题<br />3）Redisson的multiLock<br />原理：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功<br />缺陷：运维成本高、实现复杂

# Redis优化秒杀（消息队列）
用户响应速度不够<br />优化思路：

1. 串行改并行：原本由 1 个线程的操作改为由 2 个或多个线程同时操作，比如 1 个线程负责判断秒杀资格，1 个线程负责减库存 + 创建订单（写）
2. 同步改异步：判断完秒杀资格后，就可以返回订单 id 给前端；其余的写库操作可以异步执行。
3. 提高判断秒杀资格的性能：读 DB 改为读 Redis

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696142608378-b70532e1-6708-4ef4-93a3-e1c1c356ebf1.png#averageHue=%23f9f5f5&clientId=u30c8dc24-1b16-4&from=paste&height=527&id=uba0ca84a&originHeight=870&originWidth=1811&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=469064&status=done&style=none&taskId=u6dfc43b7-6cfc-46a3-b1ee-6012fdc0f4e&title=&width=1097.575694137591)

优化后的流程<br />1）将秒杀券库存信息提前存入Redis，用set记录用户是否已下单<br />2）基于Lua脚本，判断秒杀库存、一人一单。仅在Redis中实现用户资格判断<br />3）确认有秒杀资格后，将订单等信息传递给阻塞队列，单个独立线程串行从队列中取出信息并异步下单<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696143250708-38fb4f4d-0790-48d6-87ba-c9ad4a615872.png#averageHue=%23f1eded&clientId=u30c8dc24-1b16-4&from=paste&height=548&id=u6c309be3&originHeight=905&originWidth=1898&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=475734&status=done&style=none&taskId=u13f363a0-2817-40cc-8701-eae0b9c999d&title=&width=1150.3029638173095)
> 具体实现没有细看

阻塞队列可以用 JDK 原生的 BlockingQueue 实现，记得指定队列容量。

## Redis消息队列
消息队列 (Message Queue)，字面意思就是存放消息的队列。最简单的消息队列模型包括3个角色

1. 消息队列：存储和管理消息，也被称为消息代理(Message Broker)
2. 生产者：发送消息到消息队列
3. 消费者：从消息队列获取消息并处理消息
> 可以看做是一个快递箱


JDK 阻塞队列可能存在哪些问题？

1. 服务器宕机，内存队列中的订单信息全部丢失
2. 线程处理错误，已取出单个订单信息，但没有入库
3. 受单 JVM 内存限制

Redis消息队列的优势

1. 独立于jvm，不受jvm内存限制
2. 消息队列会做持久化，避免服务器宕机数据丢失
3. 消息投递后需要消费者确认，重复投递知道确认位置

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696155119959-b167ddaa-9628-4044-a94c-6a2fac26f4ca.png#averageHue=%23f7f5f1&clientId=u30c8dc24-1b16-4&from=paste&height=345&id=u70e92a57&originHeight=474&originWidth=1478&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=194878&status=done&style=none&taskId=u703fa789-5cec-4ee2-a6ec-81004c8a2ef&title=&width=1074.909090909091)

Redis提供了三种不同的方式来实现消息队列

1. list结构：基于List结构模拟消息队列
2. Pubsub：基本的点对点消息模型
3. stream：比较完善的消息队列模型

1）基于list实现的消息队列<br />Redis的list数据结构是一个双向链表，很容易模拟出队列效果。<br />队列是入口和出口不在一边，因此我们可以利用:LPUSH结合RPOP、或者RPUSH结合LPOP来实现不过要注意的是，当队列中没有消息时RPOP或LPOP操作会返回null，并不像VM的阻塞队列那样会阻塞并等待消息，因此这里应该使用BRPOP或者BLPOP来实现阻塞效果。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696164574872-3d70e494-b072-4363-89ae-bd6a9cd6e5b6.png#averageHue=%23abc378&clientId=u30c8dc24-1b16-4&from=paste&height=110&id=u1e81c1e8&originHeight=151&originWidth=1573&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=120643&status=done&style=none&taskId=u825c103a-bd32-4571-a67c-3a299e07d93&title=&width=1144)<br />优点:

- 利用Redis存储，不受限于JVM内存上限
- 基于Redis的持久化机制，数据安全性有保证可以满足消息有序性

缺点:

- 无法避免消息丢失。服务取出消息后挂了，POP会直接移除消息，其他消费者也拿不到消息
- 只支持单消费者。无法实现一条消息被多个消费者消费

2）基于Pubsub实现的消息队列<br />Pubsub(发布订阅)是Redis2.0版本引入的消息传递模型。顾名思义，消费者可以订阅一个或多个channel，生产老向对应channel发送消息后，所有订阅者都能收到相关消息

- SUBSCRIBE channelchannell：订阅一个或多个频道
- PUBLISH channel msg：向一个频道发送消息
- PSUBSCRIBE pattern[pattern]：订阅与pattern格式匹配的所有频道

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696165776585-382acb55-2db7-48cf-8e63-64cf755417ba.png#averageHue=%23e9e6e6&clientId=u30c8dc24-1b16-4&from=paste&height=367&id=u11ef9f6c&originHeight=504&originWidth=1604&originalType=binary&ratio=1.6500000953674316&rotation=0&showTitle=false&size=185611&status=done&style=none&taskId=u961c5a12-0d8b-48d6-a215-11b59f8aafd&title=&width=1166.5454545454545)<br />优点:

- 采用发布订阅模型，支持多生产、多消费

缺点:

- 不支持数据持久化。
- 无法避免消息丢失。如果发出的消息没有被订阅，直接就丢失了
- 消息堆积有上限，超出时数据丢失

### 基于Stream的消息队列
Stream是Redis5.0引入的一种新数据类型，可以实现一个功能非常完善的消息队列。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696166864838-cd4e2688-e8aa-442c-af28-43e81ce17175.png#averageHue=%23627e75&clientId=ue1bd0761-17c2-4&from=paste&height=473&id=u877f42c7&originHeight=650&originWidth=1828&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=681330&status=done&style=none&taskId=ua4e452ed-2fb0-466c-8fd2-ae0fbca4566&title=&width=1329.4545454545455)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696167310411-402a506a-e26f-476f-9be9-b28cb1b35a80.png#averageHue=%23526b74&clientId=ue1bd0761-17c2-4&from=paste&height=584&id=uf39751b5&originHeight=803&originWidth=1749&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=916739&status=done&style=none&taskId=u4e0d460f-89b5-4051-b2dd-8bb6111a5af&title=&width=1272)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696167437895-0cb37d7d-4b12-428a-806b-53ddfdb050c4.png#averageHue=%23d6dfd6&clientId=ue1bd0761-17c2-4&from=paste&height=564&id=u66c86067&originHeight=776&originWidth=1736&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=507418&status=done&style=none&taskId=uc807b116-709a-4d3c-a7bc-583c7807222&title=&width=1262.5454545454545)<br />STREAM类型消息队列的XREAD命令特点

- 消息可回溯
- 一个消息可以被多个消费者读取
- 可以阻塞读取
- 有消息漏读的风险

**消费者组(Consumer Group)**<br />将多个消费者划分到一个组中，监听同一个队列。具备下列特点:

1. 队列中的消息会分流给组内的不同消费者，而不是重复消费，从而加快消息处理的速度（同一个组内消费者处于竞争关系，抢消息）
2. 消费者组会维护一个标示，记录最后一个被处理的消息哪怕消费者宕机重启，还会从标示之后读取消息。**确保每一个消息都会被消费，按顺序消费**
3. 消费者获取消息后，消息处于pending状态，并存入一个pending-list。当处理完成后需要通过XACK来确认消息，标记消息为已处理，才会从pendingList移除。

> 跳过，太枯燥，而且用不到

--
# 达人探店（SortedSet）

## 发布探店笔记
探店笔记类似点评网站的评价，往往是图文结合。对应的表有两个

1. tb_blog：探店笔记表，包含笔记中的标题、文字、图片等
2. tb_blog_comments：其他用户对探店笔记的评价

已实现发布、查看<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696171885329-cf3ecb10-7cbc-4344-8d73-88dfee523a63.png#averageHue=%23f3efed&clientId=ue1bd0761-17c2-4&from=paste&height=676&id=ua5eeb62f&originHeight=930&originWidth=1847&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=933515&status=done&style=none&taskId=uc9f64c32-f552-4657-8b2b-5bd1209b053&title=&width=1343.2727272727273)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696171867484-0761df93-0b29-4838-9d58-61b61dcc5aab.png#averageHue=%23eae4e1&clientId=ue1bd0761-17c2-4&from=paste&height=655&id=u1934b9d7&originHeight=900&originWidth=1643&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=861951&status=done&style=none&taskId=u51e3c9d2-defc-4668-9aaf-0267a9c3855&title=&width=1194.909090909091)

## 点赞
完善点赞功能：

- 同一个用户只能点赞一次，再次点击则取消点赞
- 如果当前用炉已经点赞，则点赞按钮高亮显示(前端已实现，判断字段Blog类的isLike属性)

实现步骤：

- 修改点赞功能，利用Redis的set集合判断是否点赞过，未点赞过则点赞数+1，已点赞过则点赞数-1
   - blogId作为key，set集合存储点赞过的userId
   - `SISMEMBER key member`命令可以判断userId是否在集合里存在
- 给Blog类中添加一个isLike字段（不是数据库里加的），标示是否被当前用户点赞
   - 修改根据id查询Blog的业务，判断当前登录用户是否点赞过，赋值给isLike字段
   - 修改分页查询Blog业务，判断当前登录用户是否点赞过，赋值给isLike字段

添加字段
```java
/**
 * 是否点赞过了
 */
@TableField(exist = false)
private Boolean isLike;
```
点赞实现
```java
/**
 * 点赞
 *
 * @param blogId
 * @return
 */
@Override
public Result likeBlog(Long blogId) {
    // 1 获取登录用户
    Long userId = UserHolder.getUser().getId();
    // 2 判断当前用户是否点赞，从Redis的set集合找
    String key = BLOG_LIKED_KEY + blogId;
    Boolean isMember = stringRedisTemplate.opsForSet().isMember(key, userId.toString());
    // 3 没点赞，点赞
    if (BooleanUtil.isFalse(isMember)) {
        // 3.1 数据库liked+1
        boolean isSuccess = update().setSql("liked = liked + 1").eq("id", blogId).update();
        // 3.2 保存用户到Redis的set集合
        if (isSuccess) {
            stringRedisTemplate.opsForSet().add(key, userId.toString());
        }
    } else {
        // 4 点赞了
        // 4.1 数据库liked-1
        update().setSql("liked = liked - 1").eq("id", blogId).update();
        // 4.2 用户从Redis的set集合移除
        stringRedisTemplate.opsForSet().remove(key, userId.toString());
    }
    return Result.ok();
}

```

## 点赞排行榜
需求：按照点赞时间先后返回Top5的用户<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696239768317-4723c618-038b-4b2b-b7e8-fcce3845e6bb.png#averageHue=%23e3dddb&clientId=ua5f5a85b-6dd0-4&from=paste&height=599&id=uc87ee8e9&originHeight=824&originWidth=1545&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=874540&status=done&style=none&taskId=u4150dd9a-c44b-4175-b4fb-49985dba94a&title=&width=1123.6363636363637)

选择一种合适的数据结构：能排序，且查找速度快 ——> SortedSet<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696240382533-7f3aeb36-84ed-4548-8ed9-3a571595df7d.png#averageHue=%23c6c6c3&clientId=ua5f5a85b-6dd0-4&from=paste&height=221&id=u1e1ca867&originHeight=267&originWidth=783&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=84951&status=done&style=none&taskId=u0bb40ea9-1444-45c3-aa0f-a6893c204d1&title=&width=647.1074176155463)<br />SortedSet命令：  <br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696240961626-4e85cbe3-b5cd-442f-aa77-dfed46e14629.png#averageHue=%23082236&clientId=ua5f5a85b-6dd0-4&from=paste&height=272&id=u0e9f6ceb&originHeight=329&originWidth=599&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=135509&status=done&style=none&taskId=u390dc1f9-7d34-4b78-977f-0aec6c99534&title=&width=495.0413067071676)

实现思路：<br />1）之前实现点赞时，将每个blog点赞的用户存入了Redis的Set集合里，改为存入SortedSet集合中，并将时间戳作为score排序

- `zadd  k score v`添加userId到集合中
```java
stringRedisTemplate.opsForZSet().add(key, userId.toString(), System.currentTimeMillis());
```

- `ZSCORE k v`查询集合里是否存在用户，存在返回score
```java
Double score = stringRedisTemplate.opsForZSet().score(key, userId.toString());
```
2）`ZRANGE`获取top5的userId
```java
@Override
    public Result queryBlogLikes(Long blogId) {
        // 1 查询top5的用户
        String key = BLOG_LIKED_KEY + blogId;
        Set<String> top5 = stringRedisTemplate.opsForZSet().range(key, 0, 4);
        if (top5 == null || top5.isEmpty()) {
            return Result.ok(Collections.emptyList());
        }
        // 2 解析userId
        List<Long> ids = top5.stream().map(Long::valueOf).collect(Collectors.toList());
        String strIds = StrUtil.join(",", ids);
        // 3 userId查询user信息
        // 注意：mysql的in不保证有序性，用order by field(id, id1, id2 ...) 保证有序
        List<UserDTO> userDTOS = userService
                .query().in("id", ids).last("ORDER BY FIELD(id," + strIds + ")").list()
                .stream()
                .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
                .collect(Collectors.toList());
        // 4 返回用户信息
        return Result.ok(userDTOS);
    }
```

# 好友关注（SortedSet）
## 关注和取关
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696252400255-99b8c0b5-6087-45b4-9aac-3150f2e631c6.png#averageHue=%23e6e1dd&clientId=ucf2f1bc2-449d-4&from=paste&height=856&id=u2519c825&originHeight=1036&originWidth=1830&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=1337883&status=done&style=none&taskId=u1735487e-fd96-453f-ad9a-b2b31a927c1&title=&width=1512.3966465344186)<br />关注是User之间多对多的关系，需要一张表来维护关注关系<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696252948704-d91d1bc2-bb36-4842-89af-081578df9302.png#averageHue=%23eeeeeb&clientId=ucf2f1bc2-449d-4&from=paste&height=202&id=u33fdfc7f&originHeight=245&originWidth=1162&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=261918&status=done&style=none&taskId=u3973fa14-5c42-4cfb-9333-d51d20feac7&title=&width=960.3305482366089)

关注和取关实现
```java
/**
 * 关注和取关
 *
 * @param followUserId
 * @param isFollow     是否关注了
 * @return
 */
@Override
public Result follow(Long followUserId, Boolean isFollow) {
    // 1 获取登录用户
    Long userId = UserHolder.getUser().getId();
    // 2 关注，新增关注数据
    if (isFollow) {
        Follow follow = new Follow();
        follow.setUserId(userId);
        follow.setFollowUserId(followUserId);
        save(follow);
    } else {
        // 3 取关，删除关注数据
        remove(new QueryWrapper<Follow>()
                .eq("user_id", userId)
                .eq("follow_user_id", followUserId));
    }
    return Result.ok();
}
```

## 共同关注
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696257266272-1c1ce2c6-571a-4c2b-a35d-92cfb6516f85.png#averageHue=%23f9f8f8&clientId=ueca89f5f-b08d-4&from=paste&height=763&id=u90793dab&originHeight=923&originWidth=1725&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=368321&status=done&style=none&taskId=u8a5994af-1a02-4f3c-9f19-9f7849bb5b9&title=&width=1425.6197897660502)

实现思路：<br />1）关注用户以后，把关注用户id存入redis的Set集合<br />2）借助Redis的Set集合求交集功能，求共同关注（两个用户各自的关注用户Set集合求交集）
```java
// 2 求关注交集
Set<String> intersectIds = stringRedisTemplate.opsForSet().intersect(key, key2);
```

查找共同关注实现
```java
/**
     * 查找共同关注
     *
     * @param userId2
     * @return
     */
    @Override
    public Result followCommon(Long userId2) {
        // 1 获取当前user
        Long userId = UserHolder.getUser().getId();
        String key = "follows:" + userId;
        String key2 = "follows:" + userId2;
        // 2 求关注交集
        Set<String> intersectIds = stringRedisTemplate.opsForSet().intersect(key, key2);
        // 2.1 无交集
        if (intersectIds == null || intersectIds.isEmpty()) {
            // 无交集
            return Result.ok(Collections.emptyList());
        }
        // 3 有交集，解析id交集
        List<Long> ids = intersectIds.stream().map(Long::valueOf).collect(Collectors.toList());
        // 4 查询用户并封装
        List<UserDTO> userDTOS = userService.listByIds(ids).stream()
                .map(user -> BeanUtil.copyProperties(user, UserDTO.class))
                .collect(Collectors.toList());
        // 5 返回
        return Result.ok(userDTOS);
    }
}
```

## 关注推送
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696301205007-61a93fa6-e98f-4105-9b0e-87a00e7b9ba3.png#averageHue=%23e9dede&clientId=uc4865e41-f4dd-4&from=paste&height=657&id=u7f8e920a&originHeight=904&originWidth=2035&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=453212&status=done&style=none&taskId=u22da3e70-11ac-4081-affe-755ca26cc01&title=&width=1480)

Feed流产品有两种常见模式:<br />1）Timeline<br />不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注。例如朋友圈

- 优点：信息全面，不会有缺失。并且实现也相对简单
- 缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低

2）智能排序<br />利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户

- 优点：投喂用户感兴趣信息，用户粘度很高，容易沉迷
- 缺点：如果算法不精准，可能起到反作用

本例中的个人页面，是基于关注的好友来做Feed流，因此采用Timeline的模式。该模式的实现方案有三种：<br />**1）拉模式**<br />也叫读扩散<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696300489737-9614df1e-eac1-40d0-a568-66092fd4eec5.png#averageHue=%23fbfbfb&clientId=uf6eea3f4-1efa-4&from=paste&height=331&id=u2497a1f2&originHeight=455&originWidth=1057&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=169638&status=done&style=none&taskId=u96c20df2-588a-411d-a7c8-b04448e7329&title=&width=768.7272727272727)<br />缺点：读取+排序耗时时间久

**2）推模式**<br />也叫写扩散<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696300665303-92acddd4-d33f-4969-8225-ed7088f492e6.png#averageHue=%23f6f6f6&clientId=uf6eea3f4-1efa-4&from=paste&height=366&id=u227f8189&originHeight=503&originWidth=727&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=129564&status=done&style=none&taskId=ufa91eb6c-767e-4239-9a18-21a74db4d6c&title=&width=528.7272727272727)<br />缺点：发消息以后，每个粉丝都会收到消息，粉丝很多的情况下特别耗费空间

**3）推拉结合**<br />也叫读写混合<br />普通人（粉丝少）发消息，直接采用推模式发送到收件箱<br />大V（粉丝多）发消息，对于活跃粉丝采用推模式，普通粉丝采用拉模式<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696301196217-7cdc14db-5b5d-4055-8481-f0a71d1029a3.png#averageHue=%23fafafa&clientId=uc4865e41-f4dd-4&from=paste&height=614&id=u7355c642&originHeight=844&originWidth=1760&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=442718&status=done&style=none&taskId=ud6c7f5a7-ecc2-44a4-88c9-eb93c58b6cb&title=&width=1280)

三种方案的对比![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696301733465-e19d8493-0c1e-4c22-8b6b-0e75bcef3167.png#averageHue=%23e5e4e2&clientId=u0e7bccfb-d4de-4&from=paste&height=621&id=u1f0f7c60&originHeight=854&originWidth=1954&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=298147&status=done&style=none&taskId=ue3c3aae5-78b7-48c7-82be-4fd0cf45f60&title=&width=1421.090909090909)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696596401052-728792fb-8bb3-43ef-ad12-5de17ba1b65d.png#averageHue=%23e0d9d7&clientId=u14818403-b2ea-4&from=paste&height=384&id=uc0f31c72&originHeight=528&originWidth=944&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=317621&status=done&style=none&taskId=ucc998b1e-9c5c-4c14-a916-7a9f17f500d&title=&width=686.5454545454545)<br />微博、抖音都有类似的功能，关注页面下拉刷新

**基于推模式实现关注推送功能**<br />实现思路：<br />1）收件箱满足可以根据时间戳排序，用Redis的`SortedSet`实现<br />key为前缀+userId，value为blogId，score为时间戳<br />2）修改新增探店blog的业务，在保存blog到数据库的同时，推送到粉丝的收件箱<br />3）查询收件箱数据时，可以实现分页查询，滚动分页

Feed流的分页问题<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696303340160-ba46df9a-1e15-4c88-9b7e-f72f1899113b.png#averageHue=%23f7f6f5&clientId=uaa1f8912-4175-4&from=paste&height=337&id=ue0e156f0&originHeight=464&originWidth=890&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=127198&status=done&style=none&taskId=u25b01fed-54c0-423f-b579-76a14700d81&title=&width=647.2727272727273)<br />**滚动分页**<br />lastId记录每次当前页查询到的最后一条数据的时间戳（类似游标）。查询下一页时，从当前时间戳的下一条开始查询即可<br />查询指令：`zrevrangebyscore key max min [WITHSCORES] [LIMIT offset count]`

- 从大到小排序
- max：分数最大值，<=，每次取上次查询的分数最小值。
- min：分数最小值（不变，始终是0）。
- offset：起始角标，取在上一次的结果中与最小值一样的元素个数
- count：查询数量

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696303726556-5cdf21c3-7337-43d8-9f4d-f57474bd8255.png#averageHue=%23f7f6f6&clientId=uaa1f8912-4175-4&from=paste&height=311&id=u059f77b7&originHeight=427&originWidth=963&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=98864&status=done&style=none&taskId=u800134c6-5118-4cd5-bd44-6e37b3490d8&title=&width=700.3636363636364)<br />关注推送实现
```java
/**
 * 查询关注列表里，关注用户的最新博文，下拉（滚动）刷新
 *
 * @param max    最大的时间戳
 * @param offset 偏移量，从第几条开始查
 * @return
 */
@Override
public Result queryBlogOfFollow(long max, Integer offset) {
    // 1 获取当前用户
    Long userId = UserHolder.getUser().getId();
    // 2 查询收件箱
    String key = FEED_KEY + userId;
    Set<ZSetOperations.TypedTuple<String>> typedTuples = stringRedisTemplate.opsForZSet().reverseRangeByScoreWithScores(key, 0, max, offset, 2);
    // 3 非空判断
    if (typedTuples == null || typedTuples.isEmpty()) {
        return Result.ok();
    }
    // 4 解析收件箱数据
    // zset={5,5,5,5,3,3,2}
    // 第一次：max=6, offset=0, res={5,5}, offset=2,
    // 第二次：
    long minTime = 0;
    int os = 1;
    List<Long> blogIds = new ArrayList<>(typedTuples.size());
    for (ZSetOperations.TypedTuple<String> typedTuple : typedTuples) {
        // 4.2 获取分数（时间戳）
        long time = typedTuple.getScore().longValue();
        // 4.1 获取blogId
        blogIds.add(Long.valueOf(typedTuple.getValue()));
        // 获取最小时间戳、偏移量
        if (time == minTime) {
            os++;
        } else {
            minTime = time;
            os = 1;
        }
    }
    // 5 根据id查询blog
    String strIds = StrUtil.join(",", blogIds);
    List<Blog> blogs = query().in("id", blogIds).last("ORDER BY FIELD(id," + strIds + ")").list();
    // 6 查询笔记的用户信息，被点赞信息
    for (Blog blog : blogs) {
        queryBlogUser(blog);
        isBlogLiked(blog);
    }
    // 7 封装返回
    ScrollResult r = new ScrollResult();
    r.setList(blogs);
    r.setOffset(os);
    r.setMinTime(minTime);
    return Result.ok(r);
}
```

# 附近商铺（GEO）
## GEO
GEO就是Geolocation的简写形式，代表地理坐标。Redis在3.2版本中加入了对GEO的支持，允许存储地理坐标信息，帮助我们根据经纬度来检索数据。常见的命令有:<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696663098878-c7bbaacc-108c-444a-9df7-4511ebb53508.png#averageHue=%23ececec&clientId=uea7f3b2e-47a7-4&from=paste&height=264&id=udcb5af30&originHeight=320&originWidth=1189&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=178726&status=done&style=none&taskId=u7859da94-2a4a-49fc-98a9-39c450c6e3b&title=&width=982.6445971199037)
> 示例：
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696663425094-d7150e2c-1591-4fe6-9f87-ee6a66610d06.png#averageHue=%2311273b&clientId=uea7f3b2e-47a7-4&from=paste&height=47&id=u634f3a00&originHeight=57&originWidth=1225&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=71626&status=done&style=none&taskId=u1ba65b53-7a4d-42a6-900b-807c8f34af7&title=&width=1012.39666229763)
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696663404527-dd6735d7-2d79-4b2c-9957-bbf523b43e27.png#averageHue=%23efefef&clientId=uea7f3b2e-47a7-4&from=paste&height=112&id=u50ca16b6&originHeight=135&originWidth=883&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=26874&status=done&style=none&taskId=u4546ab27-af93-41f8-b9b8-3c21f74603a&title=&width=729.7520431092304)
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664137200-4d9f7e33-c933-4298-b995-ee0828237fc0.png#averageHue=%23162a3e&clientId=uea7f3b2e-47a7-4&from=paste&height=40&id=u1d03a498&originHeight=49&originWidth=440&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=30148&status=done&style=none&taskId=u93f6227f-cb53-4f8c-ad6f-e28597df471&title=&width=363.6363521722099)
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664224298-b27f5b74-19f9-4051-9840-1cc1fda12f04.png#averageHue=%23203346&clientId=uea7f3b2e-47a7-4&from=paste&height=63&id=uf2844529&originHeight=76&originWidth=375&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=55096&status=done&style=none&taskId=u3b0c3174-9ddf-49a3-9f87-0ff01a78bbe&title=&width=309.9173456013153)
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664247137-d89b976d-8a14-4415-bc9a-bddd5ae284b0.png#averageHue=%231c2f42&clientId=uea7f3b2e-47a7-4&from=paste&height=38&id=ue2fd12ab&originHeight=46&originWidth=378&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=29909&status=done&style=none&taskId=u726fa3d3-5600-48b7-b713-d693b5de29b&title=&width=312.3966843661258)
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664204137-f232ec32-d99f-4009-8ee4-78a2292b4bd8.png#averageHue=%23092135&clientId=uea7f3b2e-47a7-4&from=paste&height=152&id=u79284ecc&originHeight=184&originWidth=1042&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=102550&status=done&style=none&taskId=u31d9a723-5ce9-4399-b896-890b631f2f3&title=&width=861.1569976441881)


## 附近商户搜索
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664423482-812c6f96-c0a8-4910-bd0f-6de855fe35d1.png#averageHue=%23bda18c&clientId=uea7f3b2e-47a7-4&from=paste&height=448&id=ucf00cbc7&originHeight=542&originWidth=1209&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=500754&status=done&style=none&taskId=ubd495c47-1749-490b-a524-0de9a6067d9&title=&width=999.1735222186404)<br />实现流程<br />1）导入商铺信息到GEO<br />按照商户类型做分组，类型相同的商户作为同一组，以typeld为key存入同一个GEO集合中即可<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696664622175-76871b4b-c5a9-4ff9-940b-7f07f89c7236.png#averageHue=%23d4c0bf&clientId=uea7f3b2e-47a7-4&from=paste&height=304&id=u4a4037bc&originHeight=368&originWidth=715&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=134933&status=done&style=none&taskId=u24ff9ff9-e062-430c-940a-c002b5f0e53&title=&width=590.9090722798411)
```java

  /**
     * 将店铺地理信息存入Redis
     */
    @Test
    public void loadShopData() {
        // 1 查询店铺信息
        List<Shop> shopList = shopService.list();
        // 2 店铺按照typeId分组
        Map<Long, List<Shop>> shopMap = shopList.stream().collect(Collectors.groupingBy(Shop::getTypeId));
        // 3 分批写入Redis，同一类型商铺一批
        for (Map.Entry<Long, List<Shop>> entry : shopMap.entrySet()) {
            // 获取商铺类型id
            Long typeId = entry.getKey();
            // 获取同一类型的全部商铺
            List<Shop> shops = entry.getValue();
            // 将商铺的地理坐标存起来
            List<RedisGeoCommands.GeoLocation<String>> locations = new ArrayList<>(shops.size());
            for (Shop shop : shops) {
                locations.add(new RedisGeoCommands.GeoLocation<String>(
                        shop.getId().toString(),
                        new Point(shop.getX(), shop.getY())
                ));
            }
            String key = "shop:geo:" + typeId;
            stringRedisTemplate.opsForGeo().add(key, locations);
        }
    }
```
 2）根据坐标查询附近商铺<br />`GEOSEARCH`查出坐标(x,y)附近的商铺
```java
GEOSEARCH key [FROMMEMBER member] [FROMLONLAT longitude latitude] [BYRADIUS radius m|km|ft|mi] [BYBOX width height m|km|ft|mi] [ASC|DESC] [COUNT count [ANY]] [WITHCOORD] [WITHDIST] [WITHHASH]
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696669325983-170ef964-0f11-425a-80b0-0d4d2c8f4769.png#averageHue=%23eaece8&clientId=uea7f3b2e-47a7-4&from=paste&height=608&id=ud504ccb6&originHeight=736&originWidth=1602&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=846125&status=done&style=none&taskId=u26f11a01-a715-4d79-b8d1-6afa7f0c169&title=&width=1323.966900408819)

# 用户签到（BitMap）
## BitMap
  ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696684374313-247c447b-8ecd-40ef-9f83-e493bbd6a2d7.png#averageHue=%23f2f2f2&clientId=uea7f3b2e-47a7-4&from=paste&height=330&id=ue5f2b4ef&originHeight=399&originWidth=913&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=245953&status=done&style=none&taskId=u6b5d750a-d1e0-477a-bc65-93cbb514244&title=&width=754.5454307573356)<br /> <br />bitmap用法<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696684732716-e1d8bd0a-472b-41fe-bd71-14f278aecd7d.png#averageHue=%23f3f3f3&clientId=uea7f3b2e-47a7-4&from=paste&height=343&id=u5eda6ac0&originHeight=415&originWidth=1075&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=100516&status=done&style=none&taskId=u39935152-c4e4-41b1-b41a-d000a1efec9&title=&width=888.4297240571038)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696685634522-fc6ce376-3238-4767-a39f-57a91bdb2821.png#averageHue=%23eeeeee&clientId=uea7f3b2e-47a7-4&from=paste&height=293&id=u06e8cd25&originHeight=354&originWidth=844&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=130208&status=done&style=none&taskId=u30d7bdd2-4a56-473c-b7a1-5f0b3f75983&title=&width=698)<br /> 
## 签到功能
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696689708214-6335f243-bc16-4ff1-8082-5144870b8020.png#averageHue=%23eae4e3&clientId=uea7f3b2e-47a7-4&from=paste&height=389&id=u40705630&originHeight=471&originWidth=913&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=158612&status=done&style=none&taskId=u022e11eb-a326-4903-ad76-ddb46e54e2e&title=&width=754.5454307573356)<br />实现流程：<br />1）获取用户、签到的年月<br />2）存入Redis的BitMap结构
```java
help setbit

SETBIT key offset value
```

   - key：前缀+userId+年月
   - value：1

业务实现
```java
/**
 * 用户签到
 * @return
 */
@Override
public Result sign() {
    // 1 获取当前登录用户
    Long userId = UserHolder.getUser().getId();
    // 2 获取日期
    LocalDateTime now = LocalDateTime.now();
    // 3 拼接key：前缀+userId+年月
    String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
    String key = USER_SIGN_KEY + userId + keySuffix;
    // 4 获取今天是本月的第几天
    int dayOfMonth = now.getDayOfMonth();
    // 5 写入Redis：SETBIT key offset 1
    stringRedisTemplate.opsForValue().setBit(key, dayOfMonth - 1, true);
    return Result.ok();
}
```

## 签到统计
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696761254399-cd80adfa-3d7e-40eb-be97-dc05346ccf5a.png#averageHue=%23ded2d2&clientId=ub2e9aa43-995d-4&from=paste&height=207&id=bhFzC&originHeight=284&originWidth=719&originalType=binary&ratio=1.2100000381469727&rotation=0&showTitle=false&size=64264&status=done&style=none&taskId=ud81e8d6f-1419-4883-b92c-34af1bfe5d2&title=&width=522.9090909090909)<br />1）连续签到天数：从最后一次签到开始向前统计，直到遇到第一次未签到为止，计算总的签到次数，就是连续签到天数

2）如何得到本月到今天的所有签到数据？<br />`BITFIELD`：操作(查询、修改、自增)BitMap中bit数组中的指定位置 (offset)的值
```java
help BITFIELD

BITFIELD key [GET type offset]
```
下面是最常规的做法，循环获取redis key中的偏移值，但是这种写法看着确实不太优雅....
```java
long offset = 0;
for (int i = 0; i < 10; i++) {
    redisTemplate.opsForValue().getBit("key", offset);
}
```
spring-boot-starter-data-redis早就考虑到了这一点，所以为我们提供了一种批量执行命令的方式。我们需要使用`BitFieldSubCommands`

3）如何获取连续签到天数？<br />Redis返回的是10进制数<br />与1做与运算可以得到最后一个bit位<br />循环：右移，直到某一天与运算结果为0停止

完整实现
```java
/**
     * 统计签到天数
     * @return
     */
    @Override
    public Result signCount() {
        // 1 获取当前登录用户
        Long userId = UserHolder.getUser().getId();
        // 2 获取日期
        LocalDateTime now = LocalDateTime.now();
        // 3 拼接key：前缀+userId+年月
        String keySuffix = now.format(DateTimeFormatter.ofPattern(":yyyyMM"));
        String key = USER_SIGN_KEY + userId + keySuffix;
        // 4 获取今天是本月的第几天
        int dayOfMonth = now.getDayOfMonth();
        // 5 获取本月截止到今天的所有签到记录，返回的是一个10进制数字
        // BITFIELD sign:5:202203 GET u14
        List<Long> result = stringRedisTemplate.opsForValue().bitField(
                key,
                BitFieldSubCommands.create() // 这里采用批量执行命令的方式
                        .get(BitFieldSubCommands.BitFieldType.unsigned(dayOfMonth)) // 无符号、截止到dayOfMonth
                        .valueAt(0) // 从角标0开始
        );
        if (result == null || result.isEmpty()) {
            // 没有签到结果
            return Result.ok(0);
        }
        Long num = result.get(0);
        if (num == null || num == 0) {
            // 十进制0代表没签到
            return Result.ok(0);
        }
        // 6 获取连续签到天数
        int count = 0;
        while (true) {
            // 和1做与运算，得到数字最后一个bit位
            if ((num & 1) == 0) {
                // 最后一位为0，改天未签到
                break;
            } else {
                count++;
            }
            // 右移，到前一天
            num >>>= 1;
        }
        return Result.ok(count);
    }
```

# UV统计（HyperLogLog）
## HyperLogLog用法

1. UV：全称Unique Visitor，也叫独立访客量，是指通过互联网访问、浏览这个网页的自然人。1天内同一个用户多次问该网站，只记录1次。
2. PV：全称Page View，也叫页面访问量或点击量，用户每访问网站的一个页面，记录1次PV，用户多次打开页面，则记录多次PV。往往用来衡量网站的流量。

UV统计在服务端做会比较麻烦，因为要判断该用户是否已经统计过了，需要将统计过的用户信息保存。但是如果每个访问的用户都保存到Redis中，数据量会非常恐怖。

Hyperloglog(HLL)是从Loglog算法派生的概率算法，用于确定非常大的集合的基数，而不需要存储其所有值。相关算法原理可以参考: <br />[HyperLogLog 算法的原理讲解以及 Redis 是如何应用它的 - 掘金](https://juejin.cn/post/6844903785744056333#heading-0Redis)<br />Redis中的HLL是基于string结构实现的，单个HLL的内存永远小于16kb，内存占用低的令人发指！作为代价，其测量结果是概率性的，有小于0.81%的误差。不过对于UV统计来说，这完全可以忽略。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696769581371-07f02f41-0160-4267-ab7d-e767102d9299.png#averageHue=%230b2236&clientId=ufa1e98a6-3214-4&from=paste&height=231&id=u2a6193b7&originHeight=317&originWidth=939&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=229402&status=done&style=none&taskId=u5352be5c-a548-41a2-86db-cff1807595a&title=&width=682.9090909090909)<br />重复元素只记录一次<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1696769665540-cc485f2b-574f-45f4-b467-b0eaac304a49.png#averageHue=%23102839&clientId=ufa1e98a6-3214-4&from=paste&height=151&id=u0440561c&originHeight=207&originWidth=504&originalType=binary&ratio=1.375&rotation=0&showTitle=false&size=115575&status=done&style=none&taskId=u455154a0-0b12-4820-8a02-32e0181f197&title=&width=366.54545454545456)

## UV统计实现
向HyperLogLog中添加100万条数据，看内存占用和统计效果
```java
 @Test
    public void testHyperLogLog() {
        String[] users = new String[1000];
        // 数组角标
        int index = 0;
        for (int i = 1; i <= 1000000; i++) {
            users[index++] = "user_" + i;
            if (i % 1000 == 0) {
                index = 0;
                stringRedisTemplate.opsForHyperLogLog().add("hll1", users);
            }
        }
        Long size = stringRedisTemplate.opsForHyperLogLog().size("hll1");
        System.out.println("size=" + size);
    }
```
1）输出：`size=997593`，和100万差距很小<br />2）内存占用前后对比，`info memory`查看内存占用情况<br />只占用了14KB
```
used_memory:1527480
used_memory_human:1.46M

used_memory:1541896
used_memory_human:1.47M
```



















