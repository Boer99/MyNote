
# Java SPI 有没有破坏双亲委派模型呢？

参考：[(65 封私信 / 80 条消息) 为什么说java spi破坏双亲委派模型？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/49667892)

虽然有 SPI 破坏双亲委派模型的说法，但我不太认同。简单说下。

双亲委派模型（再次吐槽下这个翻译），是一种加载类的约定。这个约定的一个用处是保证安全。比如说你写 Java 用了 String类，你怎么保证你用的那个 String 类就是 JDK 里提供的那个 String 类呢？答案是对于 JDK 基础类，JDK 要用特殊的 ClassLoader 来保证在正确的位置加载。JDK 主要有 3个自带 ClassLoader：

- 最基础：Bootstrap ClassLoader（加载 JDK 的/lib 目录下的类）
- 次基础：Extension ClassLoader（加载 JDK 的/lib/ext 目录下的类）
- 普通：Application ClassLoader（程序自己 classpath 下的类）

双亲委派模型要求如果一个类可以被委派最基础的 ClassLoader 加载，就不能让高层的 ClassLoader 加载。这样你就知道你用的 String 类一定是被 BootstrapClasserLoader 加载的/lib 下的那个 rt. jar 的那个 java/lang/String.class.

但**这个模型不是强制的**。如果你自己写个自己的 ClassLoader，你可以不理会它。比如你可以写个自己的 ClassLoader 去自己规定的一个神怪的目录里加载自己写的 String. class。当然 Java Runtime 能够识别出这俩 String 不是一个类，哪怕他们的 Full Qualified Class Name 一模一样。所以如果你这做了，大概率是自作自受。当如果你真的知道自己在干啥，是能够玩出一些花的。

> 顺便说一句，这个机制的安全性是有限的。假如有人能登入服务器，能够直接替换 JDK 目录的文件。上述机制也就失效了。为了保证严格的安全，还应该保证系统文件要做[数字签名](https://www.zhihu.com/search?q=%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A690288611%7D)。

另外一点是，这个模式虽然“安全“，但是损失了一丢丢灵活性。就比如 `java.sql.Driver` 这个东西。JDK 只能提供一个规范接口，而不能提供实现。提供实现的是实际的[数据库提供商](https://www.zhihu.com/search?q=%E6%95%B0%E6%8D%AE%E5%BA%93%E6%8F%90%E4%BE%9B%E5%95%86&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A690288611%7D)。**提供商的库总不能放 JDK 目录里吧**。

Java 从 1.6 搞出了 SPI 就是为了优雅的解决这类问题——**JDK 提供接口，供应商提供服务。编程人员编码时面向接口编程**，然后 JDK 能够自动找到合适的实现，岂不是很爽？

但是便利的同时也带来了困扰。提供商提供的类不能放 JDK 里的 lib 目录下，所以也就没法用 BootstrapClassLoader 加载了。所以当你代码写了

```java
Class clz = Class.forName("java.sql.Driver");
Driver d = (Driver)clz.newInstance();
```

时，这个代码会用 Bootstrap ClassLoader 尝试去加载. 问题是 `java.sql.Driver` 是个接口，无法真的实例化，就报错了。

没有 SPI 时，你可以现在 classpath 里加一个 mysql-connector-java. jar，然后这样写

```java
Class clz = Class.forName("com.mysql.jdbc.Driver");
Driver d = (Driver) clz.newInstance();
```

这就没问题了，这里用了 Application Classloader 加载了 `mysql-connector-java.jar` 的 `com.mysql.jdbc.Driver`。问题是你 **hard code** 了一定要加载 `com. mysql. jdbc. Driver`，不是很优雅，不能实现“**用接口编程，自动实例化真的实现**“的这种编码形式。

使用 SPI 后，代码大致会这样

```java
Connection connection = 
DriverManager.getConnection("jdbc:mysql://xxxxxx/xxx", "xxxx", "xxxxx");
```

DriverManager 就根据"jdbc: mysql"这个提示去找具体实现去了。

然后

```java
 System.out.println(connection.getClass().getClassLoader());
```

就会看到这里的结果是 Application ClassLoader。这就好像 Application ClassLoader 加载了本来应该由 BootstrapClassLoader 加载的 java. sql. Connection 一样。看起来像是违反了双亲委派模型。但实际上，这里的 Connection 的类型实际上是 `com.mysql.jdbc.JDBC4Connection`“，也是个第三方类。AppClassLoader 加载一个第三方类看起来并没有违反模型。

再进一步调查下 Connection 接口自己的加载情况：

```java
System.out.println(java.sql.Connection.class.getClassLoader());
```

会发现返回的 null。说明 Connection 自己是被 Bootstrap ClassLoader 加载的。

**综上，并没有说 Bootstrap ClassLoader 加载了个[第三方库](https://www.zhihu.com/search?q=%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A690288611%7D)或者 Application ClassLoader 加载了 JDK 的库的情况发生。**

所以能否请题主给出具体哪里写了“SPI 破坏双亲委派模型“？我再仔细看看是不是前后哪里谁理解错了。可能是我错了，也可能是那个参考错了。
