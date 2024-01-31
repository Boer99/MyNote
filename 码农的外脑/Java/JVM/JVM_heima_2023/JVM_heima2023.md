> 参考资料

黑马程序员 JVM 虚拟机入门到实战全套视频教程 2023 版

链接： [基础篇-1-初识JVM_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1r94y1b7eS/?p=2&spm_id_from=pageDriver&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa)

# ----- 基础篇

# 初识 JVM

## 基本概念

> 什么是 JVM？

JVM 全称是 Java Virtual Machine，中文译名 Java虚拟机。

JVM 本质上是一个运行在计算机上的程序，他的职责是运行 Java 字节码文件。

![](assets/Pasted%20image%2020240115215011.png)

> JVM 的三大核心功能

解释和运行
- 对字节码文件中的指令，实时的解释成机器码，让计算机执行

内存管理
- 自动为对象、方法等分配内存
- 自动的垃圾回收机制，回收不再使用的对象

即时编译
- 对热点代码进行优化，提升执行效率

> 即时编译

由于 JVM 需要实时解释虚拟机指令，Java 语言如果不做任何优化，性能不如 C、C++等语言。
![](assets/Pasted%20image%2020240115220857.png)

Java 需要实时解释，主要是为了支持跨平台特性
![](assets/Pasted%20image%2020240115221017.png)

【即时编译】JVM 提供了即时编译 (Just-ln-Time 简称 JIT) 进行性能的优化，最终能达到接近 C、C++语言的运行性能，甚至在特定场景下实现超越。
![](assets/Pasted%20image%2020240115221341.png)

##  常见的 JVM

常见的 JVM 有 HotSpot、GraalVM、OpenJ9 等，另外 DragonWell 龙井 JDK 也提供了一款功能增强版的 JVM。其中使用最广泛的是 HotSpot 虚拟机。（课程主要讲解的是 Oracle JDK 版本的 HotSpot 虚拟机）
![](assets/Pasted%20image%2020240115221533.png)

Java 虚拟机规范
- 《Java 虚拟机规范》由 Oracle 制定，内容主要包含了 Java 虚拟机在设计和实现时需要遵守的规范，主要包含 class 字节码文件的定义、类和接口的加载和初始化、指令集等内容。
- 《Java 虚拟机规范》是对虚拟机设计的要求，而不是对 Java 设计的要求，也就是说虚拟机可以运行在其他的语言比如 Groovy、Scala 生成的 class 字节码文件之上。

HotSpot 的发展历程
![](assets/Pasted%20image%2020240115224426.png)

## JVM 的组成

- 字节码文件
- 类加载器
- 运行时数据区域
- 执行器引擎
- 本地接口

![](assets/Pasted%20image%2020240116180941.png)

# 字节码文件详解

## 字节码文件的查看

字节码文件中保存了==源代码编译之后的内容==，以==二进制的方式存储==，无法直接用记事本打开阅读。

推荐使用 jclasslib 工具查看字节码文件。Idea 有 jclasslib 插件
- [Releases · ingokegel/jclasslib (github.com)](https://github.com/ingokegel/jclasslib/releases)

## 字节码文件的核心组成

- 基本信息
- 常量池
- 字段
- 方法
- 属性

### 基本信息

- 魔数：固定为 0XCAFEBABE，不会改变
- 主次版本号：编译字节码文件的 JDK 版本号
- 访问标识：public final 等等
- 类、父类、接口索引：通过这些索引可以找到类、父类、接口的信息

![](assets/Pasted%20image%2020240115231118.png)

> Magic 模数

文件是无法通过文件扩展名来确定文件类型的，==文件扩展名可以随意修改，不影响文件的内容==。软件使用==文件的头几个字节 (文件头) 去校验文件的类型==，如果软件不支持该种类型就会出错

Java 字节码文件中，将文件头称为 magic 魔数。固定为 0XCAFEBABE，不会改变
![](assets/Pasted%20image%2020240116121218.png)

> 主副版本号

主副版本号指的是==编译字节码文件的 JDK 版本号==，
- 主版本号用来标识大版本号，JDK 1.0-1.1 使用了 45.0-45.3，JDK 1.2 是 46 之后每升级一个大版本就加 1; 
- 副版本号是当主版本号相同时作为区分不同版本的标识，==一般只需要关心主版本号==。

1.2 之后大版本号计算方法就是：==主版本号 - 44 ==
- 比如主版本号 52 就是 JDK8

版本号的作用：主要是判断当前字节码的版本和运行时的 JDK 是否兼容

---
【需求】解决以下由于主版本号不兼容导致的错误

【问题】类文件具有错误的版本 52.0 (JDK 1.8)，应为 50.0 (JDK 1.6) 。请删除该文件或确保该文件位于正确的类路径子目录中

两种方案：（改变环境或者改变自己）
1. 升级 JDK 版本（运行时），容易引发其他兼容性问题
3. 将第三方依赖的版本号降低或者更换依赖，以满足 JDK 版本的要求 √

### 常量池

保存了
- 字符串常量
- 类或接口名
- 字段名

主要在字节码指令中使用
![](assets/Pasted%20image%2020240115231438.png)

> 作用：

避免相同的内容重复定义，节省空间

> 原理

 常量池中的数据都有一个编号，编号从 1 开始。在字段或者字节码指令中通过编号可以快速的找到对应的数据。

字节码指令中通过编号引用到常量池的过程称之为**符号引用**。

```java
public class ConstantPoolTest {  
    public static final String a1 = "abc";  
    public static final String a2 = "abc";  
    public static final String abc = "abc";  
  
    public static void main(String[] args) {  
        ConstantPoolTest constantPoolTest = new ConstantPoolTest();  
    }  
}
```

以上代码中，三个变量的值都是“abc”，“abc”的字面量值存在 `cp_info # 10` 中，`abc = "abc"` 中的变量名也是 abc，他直接用的 `cp_info # 10`
![](assets/Pasted%20image%2020240116150908.png)

### 字段

当前类或接口声明的字段信息
![](assets/Pasted%20image%2020240115231407.png)

### 方法

当前类或接口声明的方法信息。源代码方法的内容都编译为**字节码指令**
![](assets/Pasted%20image%2020240115231709.png)

字节码中的方法区域是存放**字节码指令**的核心位置，字节码指令的内容存放在方法的 Code 属性中。

> 案例了解基本的字节码指令

**操作数栈**是==临时存放==数据的地方，**局部变量表**是存放方法中的==局部变量==的位置

局部变量表
![](assets/Pasted%20image%2020240116154337.png)

```java
源代码：
int i = 0;  
int j = i + 1;

字节码指令：
// int i = 0; 
0 iconst_0  //将常量0入操作数栈
1 istore_1  //从操作数栈取出放入局部变量表1号位置

// int j = i + 1;
2 iload_1  //将局部变量表1中的数据放入操作数栈
3 iconst_1  //将常量1放入操作数栈
4 iadd  //将操作数栈顶部的两个数据进行累加，结果放入栈中
5 istore_2  //从操作数栈取出放入局部变量表2号位置

6 return  //方法结束，返回
```

> 经典的面试题

下面这段代码中 i 最后输出多少？
```java
public class Demo {  
    public static void main(String[] args) {  
        int i = 0;  
        i = i++;  
        System.out.println(i); //0  
    }  
}
```

`i=i++` 会把 i 中的值先放到操作数栈里，对 i 赋值的时候再从操作数栈里取出来，中间局部变量表 i 的加 1被覆盖了。
- 【个人理解】还有其他操作数的情况下，例如 `j+i++`，肯定是要在 i 自增前先把 i 的值入栈的。
- 【个人理解】这也是为什么 `++` 效率高的原因。
```java
//int i = 0; 
 0 iconst_0
 1 istore_1

//i = i++; 
 2 iload_1  //将局部变量表1中的数据放入操作数栈（0入栈）
 3 iinc 1 by 1  //Increment local variable by constant，局部变量表1号位置增加1（0+1）
 6 istore_1  //从操作数栈取出放入局部变量表1号位置（0覆盖了1）

 7 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
10 iload_1
11 invokevirtual #3 <java/io/PrintStream.println : (I)V>
14 return
```

`i=i++` 换成 `i = ++i`，最终输出 i 就是 1 了。
```java
 0 iconst_0
 1 istore_1
 
 2 iinc 1 by 1
 5 iload_1
 6 istore_1
 
 7 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
10 iload_1
11 invokevirtual #3 <java/io/PrintStream.println : (I)V>
14 return
```

> 通过字节码指令分析下面三种“加一”的操作性能高低

```java
int i=0, j=0, k=0; 

i++;
i = j + 1; 
k += 1;
```

### 属性

类的属性，比如源码的文件名、内部类的列表等
![](assets/Pasted%20image%2020240115231742.png)

## 玩转字节码常用工具

### javap

`javap` 是 JDK 自带的反编译工具，可以通过控制台查看字节码文件的内容。==适合在服务器上查看字节码文件内容==。

直接输入 javap 查看所有参数
```bash
PS E:\E_Boer_workpace\study-project\JVM-learn\target\classes\com\boer> javap
用法: javap <options> <classes>
其中, 可能的选项包括:
  --help -help -h -?               输出此帮助消息
  -version                         版本信息
  -v  -verbose                     输出附加信息
  -l                               输出行号和本地变量表
  -public                          仅显示公共类和成员
  -protected                       显示受保护的/公共类和成员
  -package                         显示程序包/受保护的/公共类
```

输入 `javap -v 字节码文件名称 > 目录下的输出文件名`  查看具体的字节码信息。 (如果 jar 包需要先使用 ` jar -xvf ` 命令解压

### 阿里 arthas

Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，大大提升线上问题排查效率。

[简介 | arthas (aliyun.com)](https://arthas.aliyun.com/doc/)

> 功能

- 监控面板
- 查看字节码信息
- 方法监控
- 类的热部署
- 内存监控
- 垃圾回收监控
- 应用热点定位

> Dashboard 

 [dashboard | arthas (aliyun.com)](https://arthas.aliyun.com/doc/dashboard.html)

当前系统的实时数据面板

![](assets/Pasted%20image%2020240116173235.png)

`dashboard -i 2000 -n 1` 
![](assets/Pasted%20image%2020240116172746.png)

> Dump：dump 已加载类的 bytecode 到特定目录

 [dump | arthas (aliyun.com)](https://arthas.aliyun.com/doc/dump.html)

将 JVM 中实际运行的 class 的 byte code dump 到指定目录，适用场景批量下载指定包目录的 class 字节码

> Jad：反编译指定已加载类的源码

`jad` 命令将 JVM 中实际运行的 class 的 byte code 反编译成 java 代码，便于你理解业务逻辑
- 在 Arthas Console 上，反编译出来的源码是带语法高亮的，阅读更方便
- 当然，反编译出来的 java 代码可能会存在语法错误，但不影响你进行阅读理解

---
【案例】使用阿里 arthas 定位线上出现的字节码问题

【背景】
- 小李的团队昨天对系统进行了升级修复了某个 bug，但是升级完之后发现 bug 还是存在，
- 小李怀疑是因为没有把最新的字节码文件部署到服务器上，
- 请使用阿里的 arthas 去确认升级完的字节码文件是不是最新的。

【思路】
1. 在出问题的服务器上部署一个 arthas，并启动。
2. 连接 arthas 的控制台，==使用 jad 命令加上想要查看的类名，反编译出源码==。
3. 确认源码是否是最新的

### 小总结

- 本地文件可以使用 iclasslib 工具查看，开发环境使用 iclasslib 插件
- 服务器上文件使用 javap 命令直接查看，也可以通过 arthas 的 dump 命令导出字节码文件再查看本地文件。还可以使用 jad 命令反编译出源代码

# 类加载过程
## 类的生命周期

类的生命周期描述了一个类 **加载--->使用--->卸载** 的整个过程。

分为以下阶段：
- 加载
- 连接（验证--->准备--->解析）
- 初始化
- 使用
- 卸载

> 其中初始化阶段程序员可以干涉，是面试的高频考点

## 加载阶段

1）**类加载器**根据**类的全限定名**通过不同的渠道以二进制流的方式获取字节码信息（加载到内存）。程序员可以使用 Java 代码拓展的不同的渠道

渠道包括
- 本地文件：磁盘上的字节码文件
- 动态代理生成：程序运行时使用动态代理生成
- 通过网络传输的类：早期的 Applet 技术使用

2）类加载器在加载完类之后，Java 虚拟机会将字节码中的信息保存到**方法区**中。
- 方法区只是只是 JVM 规范中的一个虚拟概念。不同的虚拟机对方法区的实现不同

3）在**方法区**中生成一个 **InstanceKlass 对象**，保存类的所有信息，里边还包含实现特定功能比如多态的信息。

类的信息包括：
- 基本信息
- 常量池
- 字段
- 方法
- 虚方法表

4）同时，Java 虚拟机还会在**堆**中生成一份与方法区中数据类似的 **`java.Lang.Class` 对象**。

作用是在 Java 代码中去 获取**类的信息** 以及存储**静态字段**的数据 (JDK8及之后)

![](assets/Pasted%20image%2020240116211409.png)

对于开发者来说，只需要访问堆中的 Class 对象而不需要访问方法区中所有信息。这样 ==Java 虚拟机就能很好地控制开发者访问数据的范围==

## 连接阶段

可以细分为三个阶段：
- 验证：验证内容是否满足《Java 虚拟机规范》
- 准备：给静态变量赋初值
- 解析：将常量池中的符号引用替换成指向内存的直接引用

> 验证

主要目的是检测 Java 字节码文件是否遵守了《Java 虚拟机规范》中的约束。==这个阶段一般不需要程序员参与。==

主要包含如下四部分，具体详见《Java 虚拟机规范》
- 文件格式验证，
	- 比如文件是否以 `0xCAFEBABE` 开头，主次版本号是否满足当前 Java 虚拟机版本要求
- 元信息验证，
	- 例如类必须有父类 (super 不能为空)
- 程序执行指令的语义验证，
	- 比如方法内的指令执行到一半强行跳转到其他方法中去 
- 符号引用验证，
	- 例如是否访问了其他类中 private 的方法等

---
【验证案例】版本号检测

主版本号不能高于运行环境主版本号，如果主版本号相等，副版本号也不能超过。
![](assets/Pasted%20image%2020240116215407.png)

> 准备

准备阶段为**静态变量** (static) 分配内存并设置初始值（感觉就是默认值，By Boer.）
- 注意: 本章涉及到的内存结构只讨论 JDK 8 及之后的版本，8 之前的版本后续章节详述
![](assets/Pasted%20image%2020240116221253.png)

准备阶段只会给**静态变量**赋初始值，而每一种基本数据类型和引用数据类型都有其初始值。
![](assets/Pasted%20image%2020240116220638.png)

**Final** 修饰的**基本数据类型**的**静态变量**，准备阶段直接会将代码中的值进行赋值
![](assets/Pasted%20image%2020240116221304.png)

![](assets/Pasted%20image%2020240116222012.png)
![](assets/Pasted%20image%2020240116222107.png)

解析阶段主要是将常量池中的符号引用替换为直接引用。
- **符号引用**就是在字节码文件中==使用编号来访问常量池中的内容==
- **直接引用**不再使用编号，而是使用==内存中地址==进行访问具体的数据。

## 初始化阶段

执行字节码文件中 **clinit** 部分的字节码指令
- 即执行**静态代码块**中的代码，并为**静态变量**赋值

【案例】初始化演示
```java
public class Demo {
    public static int value = 1;

    static {
        value = 2;
    }

    public static void main(String[] args) {}
}

0 iconst_1
1 putstatic #2 <com/boer/Demo.value : I> //将操作数栈中的值放入静态变量value中
4 iconst_2
5 putstatic #2 <com/boer/Demo.value : I>
8 return
```

**clinit 方法中的执行顺序与 Java 中编写的顺序是一致的。**
```java
public class Demo {
    static {
        value = 2;
    }

    public static int value = 1;

    public static void main(String[] args) {}
}

0 iconst_2
1 putstatic #2 <com/boer/Demo.value : I>
4 iconst_1
5 putstatic #2 <com/boer/Demo.value : I>
8 return
```

导致类的初始化的几种方式：
- 访问一个类的静态变量或者静态方法，
	- 注意：变量是 final 修饰的并且等号右边是常量不会触发初始化。
- 调用 `Class.forName(String className)`
- new 一个该类的对象时。
- 执行 Main 方法的当前类。
- 子类的初始化 clinit 调用之前，会**先调用**父类的 clinit 初始化方法

【#VM参数】 `-XX:+TraceClassLoading` 打印出加载并初始化的类

【案例】初始化日志打印演示
```java
public class Demo {
    static {
        System.out.println("Demo初始化了");
    }

    public static void main(String[] args) throws ClassNotFoundException {
        //----- 访问静态变量
        //int i = Demo3.i;

        //----- 访问final修饰的静态常量:
        int i = Demo3.i2;

        //----- Class.forName():
        //Class<?> clazz = Class.forName("com.boer.Demo3");

        //----- new一个该类对象：
        Demo3 demo3 = new Demo3();
    }
}

class Demo3 {
    static {
        System.out.println("Demo3初始化了");
    }

    public static int i = 2;
    public static final int i2 = 2;
}
```

> 【案例】某大型互联网公司 2019 笔试题——考察了类的初始化执行顺序

```java
public class Demo {  
    public static void main(String[] args) {  
        System.out.println("A");  
        new Demo();  
        new Demo();  
        // DACBCB
    }  
  
    public Demo() {  
        System.out.println("B");  
    }  
  
    { System.out.println("C"); }  
  
    static {  System.out.println("D");  }  
}
```

> 【案例】子类的初始化 clinit 调用之前，会**先调用**父类的 clinit 初始化方法

```java
// 结果：输出 2。如果把 `new B();` 去掉，输出 1。
public class Demo {  
    public static void main(String[] args) {  
        new B();  
        System.out.println(B.a);  
    }  
}  
  
class A {  
    static int a = 0;  
  
    static {  
        a = 1;  
    }  
}  
  
class B extends A {  
    static {  
        a = 2;  
    }  
}
```

clinit 指令在特定情况下不会出现
- 无静态代码块且无静态变量赋值语句。
- 有静态变量的声明，但是没有赋值语句。
- 静态变量的定义使用 final 关键字，这类变量会在准备阶段直接进行初始化。
- 直接访问父类的静态变量，不会触发子类的初始化。
- 数组的创建不会导致数组中元素的类进行初始化

```java
/**
* 数组的创建不会导致数组中元素的类进行初始化
*/
public class Demo {
    public static void main(String[] args) {
        A[] arr = new A[10];
    }
}

class A {
    static {
        System.out.println("class A 初始化");
    }
}
```

final 修饰的变量，如果**赋值的内容需要执行指令才能得出结果**，会执行 clinit 方法进行初始化。

```java
public class Demo {
    public static void main(String[] args) {
        System.out.println(A.a);
    }
}

class A {
    public static final int a = Integer.valueOf(1);

    static {
        System.out.println("class A 初始化");
    }
}
```

# ---------- 类加载器

> #《深入理解Java虚拟机》  虚拟机设计团队有意把类加载阶段中的“通过一个类的全限定名来获取描述该类的二进制字节流”这个动作放到 Java 虚拟机外部去实现，以便让应用程序自己决定如何去获取所需的类。实现这个动作的代码被称为“类加载器”（Class Loader）。
> 
> 比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个 Class 文件，被同一个 Java 虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等

> #JavaGuide 类加载器从 JDK 1.0 就出现了，最初只是为了满足 Java Applet（已经被淘汰） 的需要。后来，慢慢成为 Java 程序中的一个重要组成部分，**赋予了 Java 类可以被动态加载到 JVM 中并执行的能力**。

类加载器（ClassLoader）是 Java 虚拟机**提供给应用程序**去实现**获取类和接口字节码数据**的技术。

类加载器只参与**加载**过程中的字节码获取并**加载到内存**这一部分。

>By Boer. 
>
>抽象的规范，实际的 Java 实现的 ClassLoader 类中存放了加载的类的 Class 对象的集合

![](assets/Pasted%20image%2020240117154818.png)

> 本地接口 JNI 是 Java Native Interface 的缩写，允许 Java 调用其他语言编写的方法。在 hotspot 类加载器中，主要用于调用 Java 虚拟机中的方法，这些方法使用 C++编写。

类加载器的应用场景：
1. 企业级应用
	1. SPI 机制
	2. 类的热部署
	3. Tomcat 类的隔离
2. 大量的面试题
	1. 什么是类的双亲委派机制
	2. 打破类的双亲委派机制
	3. 自定义类加载器
3. 解决线上问题
	1. 使用 Arthas 不停机
	2. 解决线上故障

### 类加载器的分类

类加载器分为两类，
- 一类是 **Java 代码**中实现的，
	- JDK 中默认提供或者自定义：JDK 中默认提供了多种处理不同渠道的类加载器，程序员也可以自己根据需求定制
	- 所有 Java 中实现的类加载器都需要**继承 ClassLoader 抽象类**
- 一类是 **Java 虚拟机底层源码**实现的。
	- 虚拟机底层实现：源代码位于 Java 虚拟机的源码中，实现语言与虚拟机底层语言一致，比如 Hotspot 使用 C++
	- 保证 Java 程序**运行中基础类**被正确地加载，比如 `java.lang.String`，确保其可靠性

类加载器的设计 JDK8 和 8 之后的版本差别较大，**JDK8 及之前**的版本中默认的类加载器有如下几种: 
- 虚拟机底层实现 c++：
	- **启动类加载器** BootStrapClassLoader`：加载 java 中最核心的类。
- Java 实现：
	- **扩展类加载器** ExtensionClassLoader：允许扩展 Java 中比较通用的类
	- **应用程序类加载器** AppClassLoader：加载应用使用的类

#Arthas `classloader` 命令查看类加载器具体信息
-  `classloader` ：查看
	- 类加载器 的继承树，
	- Urls，
	- 类加载信息，
- `classloader -l`：类加载实例进行统计
- `classloader -c 哈希值`：看 URLClassLoader 实际的 urls

![](assets/Pasted%20image%2020240117162352.png)

![](assets/Pasted%20image%2020240118143639.png)

#### 启动类加载器

启动类加载器（Bootstrap ClassLoader）是由 **Hotspot 虚拟机** 提供的、使用 C++编写的类加载器。
- 默认加载  `JDK 安装目录/jre/lib` 下的类文件，比如 `rt.jar`，`tools.jar`，`resources.jar` 等 jar 包和类。

> `rt.jar`（runtime）是 JDK 8 非常核心的 jar 包 （包括 String，Thread 等）

Java 程序员没法在代码中获取到启动类加载器

```java
public class BootstrapClassLoaderDemo {  
    public static void main(String[] args) throws IOException {  
        //String.class是堆上的class对象  
        ClassLoader classLoader = String.class.getClassLoader();  
        System.out.println(classLoader); //null，就是启动类加载器  
        System.in.read();  
    }  
}
```

通过启动类加载器去加载用户 jar 包：
- 【不推荐】放入 jre/lib 下进行扩展：尽可能不要去更改 JDK 安装目录中的内容，会出现即时放进去由于文件名不匹配的问题也不会正常地被加载
- 【推荐】使用 **参数** 进行扩展： `-Xbootclasspath/a:jar包目录/jar包名` 

#### 扩展类加载器 和 应用程序类加载器     

- 它们的源码都位于 sun. Misc. Launcher 类中，
- 都是**静态内部类**。
- **继承自 URLClassLoader。**
- 具备通过 目录或者指定 jar 包 将字节码文件加载到内存中。

```java
public class Launcher {
	static class ExtClassLoader extends URLClassLoader{}
	static class AppClassLoader extends URLClassLoader{}
}
```

![](assets/Pasted%20image%2020240117181243.png)

扩展类加载器 默认加载 `JDK安装目录/jre/lib/ext` 下的类文件。

```java
public class ExtClassLoaderDemo {
    public static void main(String[] args) {
        ClassLoader classLoader = ScriptEnvironment.class.getClassLoader();
        System.out.println(classLoader); //sun.misc.Launcher$ExtClassLoader@45ee12a7
    }
}
```

通过扩展类加载器去加载用户 jar 包：
- 【不推荐】放入 /jre/lib/ext 下进行扩展：尽可能不要去更改 JDK 安装目录中的内容
- 【推荐】 #VM参数  参数扩展： `-Djava.ext.dirs=jar包目录[分隔符]用户jar包目录` 
	- 这种方式会覆盖掉原始目录（应该是 `/jre/lib/ext`，By Boer.），所以用分隔符追加上原始目录。
		- Windows 分隔符 `;`
		- Macos/linux 分隔符 `:`

Jar 包中包含该类
```java
class ExtClassLoaderTest{  
    static {  
        System.out.println("ExtClassLoaderTest 初始化");  
    }  
}
```

```java
//VM参数：-Djava.ext.dirs="C:\tool_all\jdk1.8.0_161\jre\lib\ext;E:\E_Boer_workpace\study-project\JVM-learn\myjar"

public class ExtClassLoaderDemo {
    public static void main(String[] args) throws ClassNotFoundException {
        Class<?> clazz = Class.forName("com.boer.ExtClassLoaderTest");
        System.out.println(clazz);
        ClassLoader classLoader = clazz.getClassLoader();
        System.out.println(classLoader);

        ClassLoader classLoader1 = ScriptEnvironment.class.getClassLoader();
        System.out.println(classLoader1);
        /**
         * ExtClassLoaderTest 初始化
         * class com.boer.ExtClassLoaderTest
         * sun.misc.Launcher$ExtClassLoader@45ee12a7
         * sun.misc.Launcher$ExtClassLoader@45ee12a7
         */
    }
}
```

---
应用程序类加载器 加载 classpath 下的类文件

```java
public class AppClassLoaderDemo {
    public static void main(String[] args) {
        // 当前项目中创建的Student类
        ClassLoader classLoader = Student.class.getClassLoader();
        System.out.println(classLoader);

        //maven依赖中包含的类
        ClassLoader classLoader1 = FileUtils.class.getClassLoader();
        System.out.println(classLoader1);
        /**
         * sun.misc.Launcher$AppClassLoader@18b4aac2
         * sun.misc.Launcher$AppClassLoader@18b4aac2
         */
    }
}
```

### JDK9 之后的类加载器

JDK8 及之前的版本中，扩展类加载器和应用程序类加载器的源码位于 rt.jar 包中的 `sun.misc.Launcher.java`。

由于 JDK 9 引入了 module 的概念，类加载器在设计上发生了很多变化。
- 遵循**模块化**方式加载字节码文件，所以**继承关系从 URLClassLoader 变成了 BuiltinClassLoader**，BuiltinClassLoader 实现了**从模块中加载字节码文件。**

![](assets/Pasted%20image%2020240129194131.png)

启动类加载器
- 使用 **Java 编写**，位于 `jdk.internal.loader.ClassLoaders` 类中。
- 启动类加载器依然无法通过 java 代码获取到，返回的仍然是 null，保持了统一

扩展类加载器 被替换成了 **平台类加载器（Platform Class Loader）**。平台类加载器的存在更多的是为了与老版本的设计方案兼容，自身没有特殊的逻辑。

### 双亲委派机制

> #《深入理解Java虚拟机》 
> 
> 各种类加载器之间的层次（协作）关系被称为类加载器的“双亲委派模型（Parents Delegation Model）
> 
> 双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。不过这里类加载器之间的父子关系一般不是以继承（Inheritance）的关系来实现的，而是通常使用组合（Composition）关系来复用父加载器的代码。
> 
> 双亲委派模型的**工作过程**是：
> - 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，
> - 只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

> #JavaGuide 在面向对象编程中，有一条非常经典的设计原则：**组合优于继承，多用组合少用继承。**

【解决什么问题？】由于 Java 虚拟机中有多个类加载器，双亲委派机制的核心是解决**一个类到底由谁加载**的问题。
- 保证类加载的安全性：
	- 避免恶意代码替换 JDK 中的核心类库，比如 `Java.Lang.String`，确保核心类库的完整性和安全性
- 避免同一个类被重复加载

【是什么？】当一个类加载器接收到加载类的任务时，会**自底向上**查找是否加载过再**由顶向下**进行加载

![](assets/Pasted%20image%2020240118153027.png)

自底向上：
- 每个类加载器都有一个**父类加载器**，在类加载的过程中，每个类加载器都会**先检查**是否已经加载了该类，
	- 如果已经加载则直接返回，
	- 否则会将加载请求**委派给**父类加载器。
- 向上查找如果已经加载过，就直接返回 **Class 对象**，加载过程结束。

自顶向下：
- 如果**所有的**父类加载器都无法加载该类，则由**当前**类加载器自己尝试加载（是否在加载路径中）。
- 第二次再去加载相同的类，仍然会向上进行委派，如果某个类加载器加载过就会直接返回

【父类加载器】每个 Java 实现的类加载器中保存了一个**成员变量**叫 “父” (Parent) 类加载器，可以理解为它的上级，并不是继承关系。
- 应用程序类加载器的 parent 父类加载器是扩展类加载器，而扩展类加载器的 parent 是空，但是在代码逻辑上，扩展类加载器依然会把启动类加载器当成父类加载器处理。
- 启动类加载器使用 C++编写，没有父类加载器。

![](assets/Pasted%20image%2020240118162010.png)

#Arthas  `classloader -t`：打印所有 ClassLoader 的继承树

![](assets/Pasted%20image%2020240118162549.png)

【类加载器的使用】Java 中使用代码的方式主动加载一个类
- 方式1：使用 **`Class.forName()`** 方法，使用**当前类**的类加载器去加载指定的类。
- 方式2：获取到类加载器，通过类加载器的 **`loadClass()`** 方法指定某个类加载器加载。

```java
public class Demo {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        // ----- 方式一
        Class<?> clazz = Class.forName("java.lang.String");
        System.out.println(clazz);

        // ----- 方式二
        // 获取main方法所在类的类加载器，应用程序类加载器
        ClassLoader classLoader = Demo.class.getClassLoader();
        System.out.println(classLoader); // sun.misc.Launcher$AppClassLoader@18b4aac2

        Class<?> stringClazz = classLoader.loadClass("java.lang.String");
        System.out.println(stringClazz.getClassLoader()); // null 启动类加载器
    }
}
```

### 打破双亲委派机制

1）自定义类加载器（继承ClassLoader）并且重写 loadClass 方法，就可以将双亲委派机制的代码去除
- Tomcat 通过这种方式实现应用之间类隔离，《面试篇》中分享它的做法

2）利用上下文类加载器加载类，比如 JDBC 和 JNDI 等

3）历史上 Osgi 框架实现了一套新的类加载器机制，允许同级之间委托进行类的加载

#### 自定义类加载器

> #《深入理解Java虚拟机》 
> 
> 双亲委派模型的**第一次**“被破坏”其实发生在双亲委派模型出现之前——即 JDK 1.2 面世以前的“远古”时代。由于双亲委派模型在 JDK 1.2 之后才被引入，但是类加载器的概念和抽象类 java.lang.ClassLoader 则在 Java 的第一个版本中就已经存在，面对已经存在的用户自定义类加载器的代码，Java 设计者们引入双亲委派模型时不得不做出一些妥协，
> 
> 为了兼容这些已有代码，无法再以技术手段避免 loadClass()被子类覆盖的可能性，只能在 **JDK 1.2** 之后的 java.lang.ClassLoader 中**添加一个新的 protected 方法 findClass()**，并引导用户编写的类加载逻辑时**尽可能去重写这个方法**，而不是在 loadClass()中编写代码。
> 
> 上节我们已经分析过 loadClass()方法，双亲委派的具体逻辑就实现在这里面，按照loadClass()方法的逻辑，**如果父类加载失败，会自动调用自己的 findClass()方法来完成加载**，这样**既不影响用户按照自己的意愿去加载类，又可以保证新写出来的类加载器是符合双亲委派规则的**

> 一个 Tomcat 程序中是可以运行多个 Web 应用的，如果这两个应用中出现了**相同限定名的类**，比如 Servlet 类，Tomcat 要保证这两个类都能加载并且它们应该是不同的类。
> 
> 如果不打破双亲委派机制，当应用类加载器加载 Web 应用1中的 MyServlet 之后，Web 应用2中相同限定名的 MyServlet 类就无法被加载了
> 
> Tomcat 使用了自定义类加载器来实现应用之间类的隔离。每一个应用会有一个独立的类加载器加载对应的类。
> - 应用 1 的类加载器--->加载 Web 应用 1 的 MyServlet
> - 应用 2 的类加载器--->加载 Web 应用 2 的 MyServlet

分析 ClassLoader 的原理，ClassLoader 中包含了4个核心方法。双亲委派机制的核心代码就位于 **loadClass** 方法中。
- `public Class<?> loadClass(String name)`（**重要**）
	- 类加载的入口，提供了双亲委派机制，内部会调用 `findClass()`
- `protected Class<?> findClass(String name)`（**重要**）
	- 由类加载器子类实现，获取二进制数据调用 defineClass ，比如 URLClassLoader 会根据文件路径去获取类文件中的二进制数据。
- `protected final Class<?> defineClass(byte[] b, int off, int len)`
	- 做一些类名的校验，然后调用虚拟机底层的方法将字节码信息加载到虚拟机内存中
- `protected final void resolveClass(Class<?> c)`
	- 执行类生命周期中的连接阶段

打破双亲委派机制的核心就是将 `loadClass()` 中下面的代码重新实现
- 正确的实现一个自定义类加载器的方式是**重写 findClass 方法**，这样**不会破坏双亲委派机制**。
![](assets/Pasted%20image%2020240118181312.png)

【案例】自定义类加载器

```java
/**  
 * 打破双亲委派机制 - 自定义类加载器  
 */  
public class BreakClassLoader1 extends ClassLoader {  
  
    private String basePath;  
    private final static String FILE_EXT = ".class";  
  
    public void setBasePath(String basePath) {  
        this.basePath = basePath;  
    }  
  
    private byte[] loadClassData(String name) {  
        try {  
            String tempName = name.replaceAll("\\.", Matcher.quoteReplacement(File.separator));  
            FileInputStream fis = new FileInputStream(basePath + tempName + FILE_EXT);  
            try {  
                return IOUtils.toByteArray(fis);  
            } finally {  
                IOUtils.closeQuietly(fis);  
            }  
        } catch (Exception e) {  
            System.out.println("自定义类加载器加载失败，错误原因：" + e.getMessage());  
            return null;  
        }  
    }  
  
    @Override  
    public Class<?> loadClass(String name) throws ClassNotFoundException {  
        if (name.startsWith("java.")) {  
            return super.loadClass(name);  
        }  
        byte[] data = loadClassData(name);  
        return defineClass(name, data, 0, data.length);  
    }  
  
    public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, IOException {  
        BreakClassLoader1 bcl1 = new BreakClassLoader1();  
        bcl1.setBasePath("E:\\E_Boer_workpace\\study-project\\JVM-learn\\myjar\\");  
  
        Class<?> clazzA = bcl1.loadClass("com.boer.A");  
        ClassLoader clazzAClassLoader = clazzA.getClassLoader();  
        System.out.println(clazzA); // class com.boer.A  
        System.out.println(clazzAClassLoader); // com.boer.BreakClassLoader1@1ddc4ec2 
    }  
}
```

自定义类加载器的**默认父类加载器是 AppClassLoader**。以 Jdk8 为例，ClassLoader 类中提供了无参构造方法设置 parent 的内容：

> 【提示】当创建子类对象时，不管使用子类的哪个构造器，默认情况下总会去调用**父类的无参**构造

```java
protected ClassLoader() {  
    this(checkCreateClassLoader(), getSystemClassLoader());  
}

private ClassLoader(Void unused, ClassLoader parent) {
        this.parent = parent;
	    ......
}
```

![](assets/Pasted%20image%2020240118181454.png)

两个自定义类加载器加载相同限定名的类，不会冲突吗？
- 不会冲突，在同一个 Java 虚拟机中，只有**相同类加载器+相同的类限定名**才会被认为是同一个类。

#Arthas   `sc –d 类名` 命令查看类具体的情况

![](assets/Pasted%20image%2020240118193804.png)

#### 线程上下文类加载器

> #《深入理解Java虚拟机》 
> 
> JNDI 现在已经是 Java 的标准服务，它的代码由启动类加载器来完成加载（在 JDK 1.3 时加入到 rt.jar 的），肯定属于 Java 中很基础的类型了。但 JNDI 存在的目的就是对资源进行查找和集中管理，它需要调用由其他厂商实现并部署在应用程序的 ClassPath 下的 JNDI **服务提供者接口**（Service Provider Interface，SPI）的代码，现在问题来了，启动类加载器是绝不可能认识、加载这些代码的，那该怎么办？
> 
> 为了解决这个困境，Java 的设计团队只好引入了一个不太优雅的设计：**线程上下文类加载器**（Thread Context ClassLoader）。这个类加载器可以通过 java.lang.Thread 类的 setContextClassLoader()方法进行设置，如果创建线程时还未设置，它将会从**父线程中继承一个**，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器**默认就是应用程序类加载器。**
> 
> 有了线程上下文类加载器，程序就可以做一些“舞弊”的事情了。JNDI 服务使用这个线程上下文类加载器去加载所需的 SPI 服务代码，这是一种“**父类加载器去请求子类加载器**”完成类加载的行为，这种行为实际上是**打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型的一般性原则**，但也是无可奈何的事情。**Java 中涉及 SPI 的加载基本上都采用这种方式来完成**，例如 JNDI、JDBC、JCE、JAXB 和 JBI 等。
> 
> 不过，**当 SPI 的服务提供者多于一个的时候，代码就只能根据具体提供者的类型来“硬编码”判断**，为了消除这种极不优雅的实现方式，在 JDK 6 时，JDK 提供了 **java.util.ServiceLoader** 类，以 META-INF/services 中的配置信息，辅以责任链模式，这才算是给 SPI 的加载提供了一种相对合理的解决方案。

JDBC 中使用了 DriverManager 来管理项目中引入的不同数据库的驱动，比如 mysql 驱动、oracle 驱动

DriverManager 类位于 rt.jar 包中，由**启动类加载器**加载。而依赖中的 mysql 驱动对应的类，由**应用程序类加载器**来加载，这就**违反**了双亲委派机制。

![](assets/Pasted%20image%2020240119120228.png)

DriverManager 怎么知道 jar 包中要加载的驱动在哪儿？

> Spi 全称为 (Service Provider Interface, 服务提供者接口)，是 JDK 内置的一种服务提供发现机制。（补充：**JDK 提供接口，供应商提供服务。编程人员编码时面向接口编程**，然后 JDK 能够自动找到合适的实现）
> 
> Spi 的工作原理：
> 1. 在 ClassPath 路径下的 **META-INF/services** 文件夹中，以**接口的全限定名**来命名文件名，对应的文件里面写该 **接口的实现**。
> 2. 使用 **ServiceLoader.load()** 加载实现类
>
> ![](assets/Pasted%20image%2020240119121047.png)

DriverManager 使用 SPI 机制，最终加载 jar 包中对应的驱动类
- **启动类加载器**加载 DriverManager。
- 在初始化 DriverManager 时，通过 SPI 机制加载 jar 包中的 mysql 驱动。
	- `ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);`
- SPI 中利用了**线程上下文类加载器**（应用程序类加载器）去加载类并创建对象

> `Java.lang.Thread` 中的 `getContextClassLoader()` 和 `setContextClassLoader(ClassLoader cl)` 分别用来获取和设置线程的上下文类加载器

```java
public final class ServiceLoader<S> implements Iterable<S>{
	public static <S> ServiceLoader<S> load(Class<S> service) {
		// 获取线程上下文加载器
        ClassLoader cl = Thread.currentThread().getContextClassLoader();
        return ServiceLoader.load(service, cl);
    }
}
```

```java
public class DriverManager {
	static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }

	// 加载jar包中的驱动
	private static void loadInitialDrivers() {
        String drivers;
        try {
            drivers = AccessController.doPrivileged(new PrivilegedAction<String>() {
                public String run() {
                    return System.getProperty("jdbc.drivers");
                }
            });
        } catch (Exception ex) {
            drivers = null;
        }
        // If the driver is packaged as a Service Provider, load it.
        // Get all the drivers through the classloader
        // exposed as a java.sql.Driver.class service.
        // ServiceLoader.load() replaces the sun.misc.Providers()

        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
				// SPI
                ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
                Iterator<Driver> driversIterator = loadedDrivers.iterator();

                /* Load these drivers, so that they can be instantiated.
                 * It may be the case that the driver class may not be there
                 * i.e. there may be a packaged driver with the service class
                 * as implementation of java.sql.Driver but the actual class
                 * may be missing. In that case a java.util.ServiceConfigurationError
                 * will be thrown at runtime by the VM trying to locate
                 * and load the service.
                 *
                 * Adding a try catch block to catch those runtime errors
                 * if driver not available in classpath but it's
                 * packaged as service and that service is there in classpath.
                 */
                try{
                    while(driversIterator.hasNext()) {
                        driversIterator.next();
                    }
                } catch(Throwable t) {
                // Do nothing
                }
                return null;
            }
        });

        println("DriverManager.initialize: jdbc.drivers = " + drivers);

        if (drivers == null || drivers.equals("")) {
            return;
        }
        String[] driversList = drivers.split(":");
        println("number of Drivers:" + driversList.length);
        for (String aDriver : driversList) {
            try {
                println("DriverManager.Initialize: loading " + aDriver);
                // 不知道是不是这里才获取到class对象
                Class.forName(aDriver, true,
                        ClassLoader.getSystemClassLoader());
            } catch (Exception ex) {
                println("DriverManager.Initialize: load failed: " + ex);
            }
        }
    }
}
```

这种由启动类加载器加载的类，委派应用程序类加载器去加载类的方式，打破了双亲委派机制。

![](assets/Pasted%20image%2020240119143017.png)

JDBC 案例中真的打破了双亲委派机制吗？

1. 打破了：这种由启动类加载器加载的类，委派应用程序类加载器去加载类的方式。打破了双亲委派机制。
2. 没有打破：JDBC 只是在 DriverManager 加载完之后，通过**初始化阶段**触发了驱动类的加载，类的加载依然遵循双亲委派机制（应用程序类加载器还是要向上再向下）。

#### Osgi 框架的类加载器

> #《深入理解Java虚拟机》 
> 
> 双亲委派模型的第三次“被破坏”是由于用户对程序动态性的追求而导致的，这里说的“动态性”指的是一些非常“热”门的名词：**代码热替换**（Hot Swap）、**模块热部署**（Hot Deployment）等。说白了就是希望 Java 应用程序能像我们的电脑外设那样，接上鼠标、U 盘，不用重启机器就能立即使用，鼠标有问题或要升级就换个鼠标，不用关机也不用重启。对于个人电脑来说，重启一次其实没有什么大不了的，但对于一些生产系统来说，关机重启一次可能就要被列为生产事故，这种情况下热部署就对软件开发者，尤其是大型系统或企业级软件开发者具有很大的吸引力。历史上，OSGi 模块化框架。它存在**同级之间的类加载器的委托加载**。OSGi 还使用类加载器实现了热部署的功能。

**热部署**指的是在服务不停止的情况下，动态地更新字节码文件到内存中。

【热部署案例】使用 arthas 不停机解决线上问题

【背景】小李的团队将代码上线之后，发现存在一个小 bug，但是用户急着使用，如果重新打包再发布需要一个多小时的时间，所以希望能使用 arthas 尽快的将这个问题修复。

思路：

- 在出问题的服务器上部署一个 arthas，并启动。
- `jad --source-only 类全限定名 > 目录/文件名.java`
	- Jad 命令反编译，然后可以用其它编译器，比如 vim 来修改源码
- `mc –c 类加载器的 hashcode 目录/文件名. Java -d 输出目录`
	- Mc 命令用来编译修改过的代码
- `retransform class 文件所在目录/xxx. Class`
	- retransform 命令加载新的字节码

注意事项：

1. 程序重启之后，字节码文件会恢复，除非将 class 文件放入 jar 包中进行更新。
2. 使用 retransform 不能添加方法或者字段，也不能更新正在执行中的方法。

> 只是一种应急的手段，正确的方式还是重新打包部署

%%  %%
# ---------- JVM 的内存区域

Java 虚拟机在运行 Java 程序过程中管理的内存区域，称之为 **运行时数据区**。《Java 虚拟机规范》中规定了每一部分的作用。

总览：
- 线程不共享
	- 程序计数器
	- Java 虚拟机栈
	- 本地方法栈
- 线程共享
	- 方法区
	- 堆

## 程序计数器

程序计数器 (Program Counter Register) 也叫 PC 寄存器，每个线程会通过程序计数器记录**当前**要执行的的**字节码指令的地址**。

在**代码执行过程**中，程序计数器会记录下一行字节码指令的地址。执行完当前指令之后，虚拟机的执行引擎根据程序计数器执行下一行指令。

![](assets/Pasted%20image%2020240120160230.png)

程序计数器可以控制程序指令的进行，实现**分支、跳转、异常**等逻辑

在**多线程**执行情况下，Java 虚拟机需要通过程序计数器记录 CPU 切换前解释执行到哪一句指令并继续解释运行。

程序计数器在运行中会出现内存溢出吗？
- 因为每个线程只存储一个固定长度的内存地址，程序计数器是**不会发生内存溢出**的。
- 程序员无需对程序计数器做任何处理。

> 内存溢出指的是程序在使用某一块内存区域时，存放的数据需要占用的内存大小超过了虚拟机能提供的内存上限。

## Java 虚拟机栈

Java 虚拟机栈采用栈的数据结构来管理方法调用中的基本数据，先进后出（First In Last Out），每一个方法的调用使用一个**栈帧**（Stack Frame）来保存。

Java 虚拟机栈随着线程的创建而创建，而回收则会在线程的销毁时进行。由于方法可能会在不同线程中执行，**每个线程**都会包含一个自己的虚拟机栈。

栈帧的组成：
1. 局部变量表：在运行过程中存放所有的局部变量
2. 操作数栈：栈帧中虚拟机在执行指令过程中用来存放临时数据的一块区域
3. 帧数据：主要包含动态链接、方法出口、异常表的引用

#### 局部变量表

局部变量表的作用是在**方法**执行过程中存放所有的局部变量。**编译成字节码文件时**就可以确定局部变量表的内容。

栈帧中的局部变量表是一个**数组**，数组中每一个位置称之为槽 (slot) ，**long 和 double** 类型占用两个槽，其他类型占用一个槽。

局部变量表保存的内容有：
- 实例方法的 this 对象，
	- 存放在**序号为 0 的位置**，指的是当前调用方法的对象，运行时会在内存中存放实例对象的地址。
- 方法的参数，
	- 顺序与方法中参数定义的顺序一致。
- 方法体中声明的局部变量。

> 以下代码的局部变量表中会占用几个槽？
> 
> ![](assets/Pasted%20image%2020240120153009.png)

为了节省空间，局部变量表中的槽是可以**复用**的，一旦某个**局部变量不再生效**，当前槽就可以再次被使用。

#### 操作数栈

操作数栈是栈帧中虚拟机在执行指令过程中用来**存放中间数据**的一块区域。
- 他是一种栈式的数据结构，如果一条指令将一个值压入操作数栈，则后面的指令可以弹出并使用该值。
- 在**编译期**就可以确定**操作数栈的最大深度**，从而在执行时正确的分配内存大小。

当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。动态链接就保存了编号到运行时常量池的内存地址的映射关系。

【案例】加法运算中操作数栈的应用

![](assets/Pasted%20image%2020240120154418.png)

#### 帧数据

主要包含动态链接、方法出口、异常表的引用

**动态链接**：保存了 编号到 运行时常量池 的内存地址的映射关系
- 当前类的字节码指令引用了其他类的属性或者方法时，需要将**符号引用**（编号）转换成对应的**运行时常量池中的内存地址**。

![](assets/Pasted%20image%2020240120154811.png)

**方法出口**：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址。

![](assets/Pasted%20image%2020240120155557.png)

“异常表”存放的是代码中异常的处理信息，包含了
- 异常捕获的生效范围
- 异常发生后跳转到的字节码指令位置。

![](assets/Pasted%20image%2020240120160903.png)

#### 栈内存溢出 & 大小设置

Java 虚拟机栈如果栈帧过多，占用内存超过栈内存可以分配的最大大小就会出现内存溢出。

Java 虚拟机栈**内存溢出**时会出现 **`StackOverflowError` 的错误**

如果我们不指定栈的大小，JVM 将创建一个具有**默认大小**的栈。大小取决于操作系统和计算机的体系结构

![](assets/Pasted%20image%2020240120163257.png)

#VM参数 修改 Java 虚拟机栈的大小
- 语法：`-Xss栈大小` 
- 单位：字节（默认，必须是 1024 的倍数）、k 或者 K (KB)、m 或者 M (MB)、g 或者 G (GB)

>  `-XX:ThreadStackSize=1024` 调整标志来配置栈大小，推荐 `-Xss`

【案例】模拟栈内存的溢出
```java
public class StackOverflowDemo {
    public static int count;

    public static void main(String[] args) {
        recusion();
        /**
         * 默认 10531
         * -Xss1024k 10549
         * -Xss10m 108830
         *
         */
    }

    public static void recusion() {
        System.out.println(++count);
        recusion();
    }
}
```

注意事项：
- HotSpot 虚拟机 对栈大小的**最大值和最小值**有要求。Windows（64 位）下的 JDK 8 测试最小值为 180 k，最大值为 1024m
- **局部变量**过多、**操作数栈深度**过大也会影响栈内存的大小
	-  一般情况下，工作中即便使用了递归进行操作，栈的深度最多也只能到几百，不会出现栈的溢出。所以此参数可以**手动指定为 `-Xss256k` 节省内存**。

## 本地方法栈

【是啥？】本地方法栈存储的是 **native 本地方法的栈帧**。

> Java 虚拟机栈存储了 Java 方法调用时的栈帧

在 Hotspot 虚拟机中，Java 虚拟机栈和本地方法栈实现上使用了**同一个栈空间**。

本地方法栈会在栈内存上生成一个栈帧，临时保存方法的参数同时方便出现异常时也把本地方法的栈信息打印出来。

![|200](assets/Pasted%20image%2020240120175329.png)

```java
public class NativeDemo {
    public static void main(String[] args) {
        try {
            FileOutputStream fileOutputStream = new FileOutputStream("F:\\123.txt");
            fileOutputStream.write(1);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        /**
         * java.io.FileNotFoundException: F:\123.txt (系统找不到指定的路径。)
         * 	at java.io.FileOutputStream.open0(Native Method)
         * 	at java.io.FileOutputStream.open(FileOutputStream.java:270)
         * 	at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
         * 	at java.io.FileOutputStream.<init>(FileOutputStream.java:101)
         * 	at com.boer.NativeDemo.main(NativeDemo.java:15)
         */
    }
}
```

## 堆

【是啥？】一般 Java 程序中堆内存是空间最大的一块内存区域。创建出来的**对象**都存在于堆上。
- 栈上的局部变量表中，可以存放堆上**对象的引用**。
- 静态变量也可以存放堆对象的引用，通过**静态变量**就可以实现对象在**线程之间共享**。
- 堆内存大小是有上限的，一直向堆中放入对象达到上限之后，就会**抛出 `OutOfMemoryError` 错误**

 > 1）最容易内存溢出的位置
 > 
 > 2）【提示】类的生命周期---加载阶段
 > 
 > 同时，Java 虚拟机还会在**堆**中生成一份与方法区中数据类似的 **`java.Lang.Class` 对象**。作用是在 Java 代码中去获取**类的信息** 以及存储**静态字段**的数据 (JDK 8 及之后)

【案例】模拟堆内存的溢出
```java
public class OutOfMemoryError {
    public static int count;

    public static void main(String[] args) {
        ArrayList<Object> list = new ArrayList<>();
        while (true) {
            list.add(new byte[1024 * 1024]); // 1MB
            System.out.println(++count); // 3285MB
        }
        /**
         * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
         * 	at com.boer.OutOfMemoryError.main(OutOfMemoryError.java:16)
         */
    }
}
```

堆空间有三个需要关注的值：
1. `used`：已使用的堆内存
2. `total`：java 虚拟机已经分配的可用堆内存
3. `max`：java 虚拟机可以分配的最大堆内存

#Arthas  `memory` 命令监控下面程序的内存情况

```java
public class OutOfMemoryError {
    public static int count;

    public static void main(String[] args) throws IOException {
        List<Object> list = new ArrayList<>();
        while (true) {
            System.in.read();
            System.out.println("添加一次"); // 回车即可
            list.add(new byte[1024 * 1024 * 100]); // 100MB
        }
    }
}
```

![](assets/Pasted%20image%2020240121145335.png)

随着堆中对象的增多，当 total 可以使用的内存即将不足时，java 虚拟机会继续分配内存给堆。

如果堆内存不足，java 虚拟机就会不断地分配内存，**total 会变大**，total 最多只能和 max 相等。

> 当 used=max=total 的时候，堆内存就溢出了吗？
> 
> 不是，堆内存溢出的判断条件比较复杂，在下一章《垃圾回收器》中会详细介绍。

#VM参数 堆大小设置 `-Xmx` （max）和 `-Xms` （total）
- 单位：字节（默认，必须是 1024 的倍数）、k 或者 K (KB)、m 或者 M (MB)、g 或者 G (GB)
- 限制：Xmx 必须大于 2 MB，Xms 必须大于 1MB

> 例： `-Xmx4g -Xms4g`
> 
> ![](assets/Pasted%20image%2020240121151655.png)
> 
> 为什么 Arthas 中显示的 heap 大小和我们设置的值不一样呢？
> 
> Arthas 中的 heap 堆内存使用了 JMX 技术中内存获取方式，这种方式与垃圾回收器有关，计算的是**可以分配对象的内存**，而不是整个内存

注意：
- Java 服务端开发时，建议将 **max 和 total 设置为相同**，
	- 无需向 java 虚拟机再次申请，减少了**申请并分配内存**时间上的开销，
	- 同时也不会出现内存过剩后**堆收缩**的情况
- Max 具体设置的值与实际的应用程序运行环境有关，在后续《实战篇》会给出设置方案

## 方法区

> #《深入理解Java虚拟机》 
> 
> 方法区（Method Area）与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等数据。虽然《Java 虚拟机规范》中把**方法区描述为堆的一个逻辑部分**，但是它却有一个别名叫作“**非堆**”（Non-Heap），目的是与 Java 堆区分开来。
> 
> 在 JDK 6 的时候 HotSpot 开发团队就有放弃永久代，逐步改为采用本地内存（Native Memory）来实现方法区的计划了
> 
> 到了 JDK 7 的 HotSpot，已经**把原本放在永久代的字符串常量池、静态变量等移出**，
> 
> 而到了 JDK 8，终于完全**废弃了永久代**的概念，改用与 JRockit、J9 一样在本地内存中实现的**元空间**（Metaspace）来代替，**把 JDK7 中永久代还剩余的内容（主要是类型信息）全部移到元空间中**。

【是啥】方法区就是存放**基础信息**的位置，**线程共享**，主要包含三部分的内容。
- 类的元信息
- 运行时常量池
- 字符串常量池

> 字符串常量池在某些 JVM 中和运行时常量池是一起的

方法区是《JVM 规范》中设计的**虚拟概念**，每款 JVM 在实现上都各不相同。**Hotspot** 设计如下：
- Jdk7 及之前的版本，将方法区放在**堆**中的**永久代**空间，堆的大小由 JVM 参数来控制。
- Jdk8 及之后的版本，将方法区放在**元空间**中，元空间位于操作系统的**直接内存**中，默认情况下只要不超过操作系统承受的上限，可以一直分配。
	- #VM参数  `-XX:MaxMetaspaceSize=值` 对元空间大小进行限制

> 【案例】模拟方法区的溢出
> 
> Todo。就是通过产生大量类来填满方法区

元信息，一般称之为 **InstanceKlass** 对象，在类的**加载**阶段完成。

![](assets/Pasted%20image%2020240121153519.png)

#### 运行时常量池

> #《深入理解Java虚拟机》 
> 
> 运行时常量池（Runtime Constant Pool）是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池表（Constant Pool Table），用于存放编译期生成的各种字面量与符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。
> 
> 符号引用翻译出来的**直接引用**也存储在运行时常量池中[1]。
> 
> **运行期间也可以将新的常量放入池中**，这种特性被开发人员利用得比较多的便是 String 类的 `intern()` 方法。
> 
> 既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会**抛出 `OutOfMemoryError` 异常**。

> #JavaGuide 
> 
> 字面量是源代码中的固定值的表示法，即通过字面我们就能知道其值的含义。字面量包括**整数、浮点数和字符串**字面量。
> 
> 常见的符号引用包括**类符号引用、字段符号引用、方法符号引用、接口方法符号**。

存放的是字节码中的常量池内容

- 字节码文件中，通过**编号查表**的方式找到常量，这种常量池称为**静态常量池**
- 当常量池加载到内存中之后，通过内存地址快速地定位到常量池中的内容，这种常量池称为**运行时常量池**

#### 字符串常量池

> #JavaGuide 
> 
> “字符串常量池”是 JVM 为了提升性能和减少内存消耗针对字符串（String 类）专门开辟的一块区域，主要目的是为了避免字符串的重复创建。

StringTable，存储代码中定义的**常量**字符串内容。

![](assets/Pasted%20image%2020240122131434.png)

字符串常量池和运行时常量池有什么关系？
- 早期设计时，字符串常量池是属于运行时常量池的一部分，他们存储的位置也是一致的。
	- JDK 7 之前：运行时常量池**逻辑包含**字符串常量池，Hotspot 虚拟机对方法区的实现为**永久代**
- 后续做出了调整，对他们做了拆分。
	- JDK 7：字符串常量池被从方法区拿到了堆，运行时常量池剩下的东西还在永久代。
	- JDK 8 及之后：hotspot 移除了永久代用**元空间**取而代之，**字符串常量池还在堆**。

静态变量存储在哪里？
- JDK 6 之前存储在方法区，即永久代
- JDK 7 及之后，存储在堆中的 Class 对象

#### 字符串拼接

- 变量连接使用 StringBuilder
- 常量连接，编译阶段直接连接（编译优化）

【案例-变量连接 】
```java
public static void main(String[] args) throws Exception {
	String a = "1";
	String b = "2";
	String c = "12"; // 字符串常量池
	String d = a + b; // 堆中对象
	System.out.println(c == d); // false
}

 0 ldc #2 <1> //从常量池中获取字符串1的地址放入操作数栈
 2 astore_1 //将操作数栈中的值放入局部变量表中
 3 ldc #3 <2> //
 5 astore_2
 6 ldc #4 <12> //
 8 astore_3
 9 new #5 <java/lang/StringBuilder>
12 dup
13 invokespecial #6 <java/lang/StringBuilder.<init> : ()V>
16 aload_1
17 invokevirtual #7 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;>
20 aload_2
21 invokevirtual #7 <java/lang/StringBuilder.append : (Ljava/lang/String;)Ljava/lang/StringBuilder;>
24 invokevirtual #8 <java/lang/StringBuilder.toString : ()Ljava/lang/String;>
27 astore 4
```

【案例-常量连接】
```java
public static void main(String[] args) throws Exception {
	String a = "1";
	String b = "2";
	String c = "12";
	String d = "1" + "2";
	System.out.println(c == d); // true
}
    
 0 ldc #2 <1>
 2 astore_1
 3 ldc #3 <2>
 5 astore_2
 6 ldc #4 <12>
 8 astore_3
 9 ldc #4 <12> // 直接拼接
11 astore 4
```

#### intern()

JDK 6 中 intern 会把第一次遇到的字符串**实例**复制到永久代的字符串常量池中，返回的是常量池中字符串实例的引用。

```java
public static void main(String[] args) throws Exception {
	String s = new StringBuilder().append("aaa").append("bb").toString();
	System.out.println(s.intern() == s); //false
	String s1 = new StringBuilder().append("ja").append("va").toString();
	System.out.println(s1.intern() == s1); //false
}
```

![](assets/Pasted%20image%2020240122212703.png)

JDK 7 及之后，由于字符串常量池在**堆**上，所以 intern 方法会把第一次遇到的字符串的**引用**放入字符串常量池。

```java
public static void main(String[] args) throws Exception {
	String s = new StringBuilder().append("aaa").append("bb").toString();
	System.out.println(s.intern() == s); //true
	String s1 = new StringBuilder().append("ja").append("va").toString();
	System.out.println(s1.intern() == s1); //false
}
```

![](assets/Pasted%20image%2020240122213127.png)

> JVM 启动时就会把“java”加入到常量池中

## 直接内存

直接内存不在《JVM 规范》中存在，所以不属于 Java 运行时的内存区域

Jdk 1.4 中引入了 NIO 机制，使用了直接内存，主要是为了解决以下两个问题：
1. Java 堆中的对象如果不再使用要回收，回收时会影响对象的创建和使用
2. IO 操作比如读文件，需要先把文件读入直接内存（缓冲区），再把数据复制到 Java 堆中。

现在直接放入直接内存即可，同时 Java 堆上维护直接内存的引用，减少了数据复制的开销，写文件也是类似的思路。

---
要创建直接内存上的数据，可以使用 **ByteBuffer**
- 语法：`ByteBuffer buffer = ByteBuffer.allocateDirect(size);`

【案例】Arthas 的 dashboard 监控以下代码：

```java
public class Demo {
    static int size = 1024 * 1024 * 100;
    static ArrayList<ByteBuffer> list = new ArrayList<>();
    static int count;

    public static void main(String[] args) throws IOException, InterruptedException {
        System.in.read();
        while (true) {
            ByteBuffer buffer = ByteBuffer.allocateDirect(size);
            list.add(buffer);
            System.out.println(++count);
            Thread.sleep(5000);
        }
    }
}
```

---
手动调整直接内存的大小
- #VM参数：`-XX:MaxDirectMemorySize=大小`
- 默认不设置该参数的情况下，JVM 自动选择最大分配的大小

# JVM 的垃圾回收

> 在 C/C++这类没有自动垃圾回收机制的语言中，一个对象如果不再使用，需要手动释放，否则就会出现内存泄漏。我们称这种释放对象的过程为垃圾回收，而需要程序员编写代码进行回收的方式为**手动回收**。

**内存泄漏**：不再使用的对象在系统中未被回收  
- 内存泄漏的积累可能会导致**内存溢出**

Java 的内存管理：
- Java 中为了简化对象的释放，引入了**自动的垃圾回收**（Garbage Collection 简称 GC）机制。
- 通过垃圾回收器来对**不再使用**的对象完成自动的回收，
- 垃圾回收器主要负责对**堆**上的内存进行回收。

> 其他很多现代语言比如 C#、Python、Go 都拥有自己的垃圾回收器。

“手动”/“自动”垃圾回收的对比：
- 自动：自动根据对象是否使用由虚拟机来回收对象
	- 优点：降低程序员实现难度、**降低对象回收 bug 的可能性**
	- 缺点：程序员无法控制内存回收的及时性
- 手动：由程序员编程实现对象的删除
	- 优点：**回收及时性高**，由程序员把控回收的时机
	- 缺点：编写不当容易出现悬空指针、重复释放、内存泄漏等问题

线程**不共享**的部分，都是伴随着线程的创建而创建，线程的销毁而销毁。而方法的栈帧在执行完方法之后就会**自动弹出栈并释放掉对应的内存**。

## 方法区的回收

> 类卸载

方法区中能回收的内容主要就是不再使用的类。

判定一个类可以被**卸载**。需要同时满足下面三个条件：
1. 此类所有实例对象都已经被回收，在堆中不存在任何该类的**实例对象以及子类对象**。
2. 加载该类的**类加载器**已经被回收。
3. 该类对应的 **`java.lang.Class` 对象**没有在任何地方被**引用**。

手动触发回收：
- 调用 `System.gc()` 方法。
- 并**不一定会立即回收垃圾**，仅仅是向 Java 虚拟机发送一个垃圾回收的请求，具体是否需要执行垃圾回收 Java 虚拟机会自行判断。

> 开发中此类场景一般很少出现，主要在如 OSGi、JSP 的**热部署**等应用场景中。每个 jsp 文件对应一个唯一的类加载器，当一个 jsp 文件修改了，就直接卸载这个 jsp 类加载器。重新创建类加载器，重新加载 jsp 文件。

【案例】类卸载

```java
/**
 * 类的卸载
 */
public class ClassUnload {
    public static void main(String[] args) throws InterruptedException {
        try {
            ArrayList<Class<?>> classes = new ArrayList<>();
            ArrayList<URLClassLoader> loaders = new ArrayList<>();
            ArrayList<Object> objs = new ArrayList<>();

            while (true) {
                URLClassLoader loader = new URLClassLoader(new URL[]{
                        new URL("file:E:\\E_Boer_workpace\\study-project\\JVM-learn\\myjar\\")});
                // 不要是类路径下的类，会双亲委派给父类加载器，卸载不掉
                Class<?> clazz = loader.loadClass("com.boer.A");
                Object o = clazz.newInstance();

//                objs.add(o); // 引用该类的实例对象
//                classes.add(clazz); //引用该类的Class对象
//                 loaders.add(loader); //引用该类的类加载器

                System.gc();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## 堆回收

### 引用计数法和可达性分析法

Java 中的对象是否能被回收，是根据对象是否被引用来决定的。如果对象被引用了，说明该对象还在使用，不允许被回收

引用计数法会为每个对象维护一个引用计数器，当对象被引用时加1，取消引用时减1。
- 优点：实现简单，C++中的智能指针就采用了引用计数法，
- 缺点：
	- 每次引用和取消引用都需要**维护计数器**，对系统性能会有一定的影响
	- 存在**循环引用**问题，所谓循环引用就是当A引用B，B同时引用A时会出现对象无法回收的问题。

---
#VM参数  `-verbose:gc` 查看垃圾回收的日志

【案例】循环引用

```java
public class Demo {
    public static void main(String[] args) {
        while (true) {
            A a = new A();
            B b = new B();
            a.b = b;
            b.a = a;
            a = null;
            b = null;
            System.gc();
        }
    }
}

class A {
    B b;
}

class B {
    A a;
}
```

以上代码输出的日志如下。

```java
[GC (System.gc())  5161K->944K(247296K), 0.0009940 secs]
[Full GC (System.gc())  944K->726K(247296K), 0.0053908 secs]
```

日志信息包含三部分：
- 垃圾回收的类型
- 回收前的年轻代大小、回收后的年轻代大小、整个堆的大小
- 垃圾回收消耗的时间

---
Java 使用的是“**可达性分析算法**”来判断对象是否可以被回收。

可达性分析将对象分为两类：
- 垃圾回收的“**根对象**”（GC Root）
- 普通对象，对象与对象之间存在引用关系。

下图中 A 到 B 再到 C 和 D，形成了一个引用链，可达性分析算法指的是如果从**某个对象到 GC Root 对象是可达的**，对象就不可被回收。

![](assets/Pasted%20image%2020240123143636.png)

哪些对象被称之为 GC Root 对象呢？
 - 1）**线程 Thread 对象。**
	- 引用线程栈帧中的方法参数、局部变量等。
- 2）**系统类加载器加载的 `java.Lang.Class` 对象。**
	- 引用类中的静态变量。
- 3）**监视器对象**，即用来保存同步锁 synchronized 关键字持有的对象。
- 4）本地方法调用时使用的**全局对象**。

> #《深入理解Java虚拟机》 
> 
> 在 Java 技术体系里面，固定可作为 GC Roots 的对象包括以下几种：
> - 在虚拟机栈（栈帧中的**本地变量表**）中引用的对象，
> 	- 譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。
> - 在方法区中**类静态属性**引用的对象，
> 	- 譬如 Java 类的引用类型静态变量。
> - 在方法区中**常量**引用的对象，
> 	- 譬如字符串常量池（String Table）里的引用。
> - 在本地方法栈中 **JNI**（即通常所说的 Native 方法）引用的对象。
> - Java **虚拟机内部**的引用，
> 	- 如基本数据类型对应的 Class 对象，一些常驻的异常对象（比如 NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器。
> - 所有被“**同步锁**”（synchronized 关键字）持有的对象。
> - 反映 Java 虚拟机内部情况的 JMXBean、JVMTI 中注册的回调、本地代码缓存等
>   
>   本地变量表 `==` 局部变量表？

1） A 的实例对象和 B 的实例对象无法到达 **线程对象**
![](assets/Pasted%20image%2020240123145702.png)

2）
![](assets/Pasted%20image%2020240123150246.png)

3）
![](assets/Pasted%20image%2020240123151717.png)

查看 GC Root
- 通过 arthas 和 eclipse Memory Analyzer (MAT) 工具可以查看 GC Root，MAT 工具是 eclipse 推出的 Java 堆内存检测工具。
- 具体操作步骤如下：
	- 1、使用 arthas 的 `heapdump` 命令将堆内存快照保存到本地磁盘中。
	- 2、使用 MAT 工具打开堆内存快照文件。
	- 3、选择 GC Roots 功能查看所有的 GC Root。

![](assets/Pasted%20image%2020240123160339.png)

### 五种对象引用

可达性算法中描述的对象引用，一般指的是**强引用**，即是 **GCRoot 对象对普通对象有引用关系**，只要这层关系存在，普通对象就不会被回收。

除了强引用之外，Java中还设计了几种其他引用方式：
- 软引用
- 弱引用
- 虚引用
- 终结器引用

#### 软引用

【是啥】如果一个对象只有软引用关联到它，**当程序内存不足时**，就会将软引用中的数据进行回收。

在 JDK 1.2 版之后提供了 **`java.lang.ref.SoftReference`** 来实现软引用，软引用常用于缓存中。

SoftReference 对象像一个盒子一样包在可回收对象外面，需要注意的是，**GC Root 对象还是通过强引用指向 SoftReference 对象**，不然 SoftReference 对象会被回收掉。

![](assets/Pasted%20image%2020240123161252.png)

软引用的执行过程：
- 将对象使用软引用包装起来，`new SoftReference<对象类型>(对象)`。
- 内存不足时，虚拟机尝试进行垃圾回收。
- 如果垃圾回收仍**不能解决内存不足**的问题，回收软引用中的对象。
- 如果依然内存不足，抛出 **`OutOfMemory` 异常**

SoftReference 对象本身的回收：
- 软引用中的对象如果在内存不足时回收，SoftReference 对象**本身也需要被回收**。如何知道哪些 SoftReference 对象需要回收呢？
- SoftReference 提供了一套**队列机制**：
	- 软引用创建时，通过构造器传入引用队列
	- 在软引用中**包含的对象被回收时**，该软引用对象会被**放入引用队列** `ReferenceQueue`
	- 通过代码遍历引用队列，将 SoftReference 的强引用删除

【案例】队列机制演示

```java
/**
 *  VM参数：-Xmx200m（堆最大内存200mB）
 */
public class Demo {
    public static void main(String[] args) {
        // 存放软引用
        ArrayList<SoftReference> softReferences = new ArrayList<>();
        // 引用队列
        ReferenceQueue<byte[]> queue = new ReferenceQueue<>();
        for (int i = 0; i < 10; i++) {
            byte[] bytes = new byte[1024 * 1024 * 100]; //100m
            SoftReference<byte[]> studentRef = new SoftReference<>(bytes, queue);
            // 堆只有200m，最多只能放一个100m的字节数组（还有其它内存开销）
            softReferences.add(studentRef);
        }

        int count = 0;
        while (true) {
            Reference<? extends byte[]> poll = queue.poll();
            if (poll != null) {
                count++;
            } else {
                break;
            }
        }
        System.out.println(count); //9，说明前9个SoftReference对象都要被回收
//        System.out.println(softReferences.size()); //10
    }
}
```

软引用也可以使用**继承 `SoftReference` 类**的方式来实现，
- StudentRef 类就是一个软引用对象。
- 通过构造器传入
	- 软引用包含的对象，
	- 以及引用队列。

【案例】软引用适用场景-缓存

![](assets/Pasted%20image%2020240123202108.png)

```java
/**
 * 软引用案例-学生信息的缓存
 * - VM options: -Xmx10m
 * - 通过输出map的size查看回收情况
 *
 * @author Boer
 */
public class StudentCache {

    public static void main(String[] args) {
        for (int i = 0; ; i++) {
            StudentCache.getInstance().cacheStudent(new Student(i, String.valueOf(i)));
        }
    }

    private static StudentCache cache = new StudentCache();
    private Map<Integer, StudentRef> StudentRefs; // 用于Cache内容的存储
    private ReferenceQueue<Student> q; // 垃圾Reference的队列

    // 继承SoftReference，使得每一个实例都具有可识别的标识。
    // 并且该标识与其在HashMap内的key相同。
    private class StudentRef extends SoftReference<Student> {
        private Integer _key = null;

        public StudentRef(Student em, ReferenceQueue<Student> q) {
            super(em, q);
            _key = em.getId();
        }
    }

    // 构建一个缓存器实例
    private StudentCache() {
        StudentRefs = new HashMap<>();
        q = new ReferenceQueue<>();
    }

    // 取得缓存器实例
    public static StudentCache getInstance() {
        return cache;
    }

    // 以软引用的方式对一个Student对象的实例进行引用并保存该引用
    private void cacheStudent(Student em) {
        cleanCache(); // 清除垃圾引用
        StudentRef ref = new StudentRef(em, q);
        StudentRefs.put(em.getId(), ref);
        System.out.println(StudentRefs.size());
    }

    // 依据所指定的ID号，重新获取相应Student对象的实例
    public Student getStudent(Integer id) {
        Student em = null;
        // 缓存中是否有该Student实例的软引用，如果有，从软引用中取得。
        if (StudentRefs.containsKey(id)) {
            StudentRef ref = StudentRefs.get(id);
            em = ref.get();
        }
        // 如果没有软引用，或者从软引用中得到的实例是null，重新构建一个实例，
        // 并保存对这个新建实例的软引用
        if (em == null) {
            em = new Student(id, String.valueOf(id));
            System.out.println("Retrieve From StudentInfoCenter. ID=" + id);
            this.cacheStudent(em);
        }
        return em;
    }

    // 清除那些已经被回收的StudentRef对象
    private void cleanCache() {
        StudentRef ref = null;
        while ((ref = (StudentRef) q.poll()) != null) {
            //
            StudentRefs.remove(ref._key);
        }
    }

//    // 清除Cache内的全部内容
//    public void clearCache() {
//        cleanCache();
//        StudentRefs.clear();
//        //System.gc();
//        //System.runFinalization();
//    }
}

@Data
@AllArgsConstructor
class Student {
    int id;
    String name;
}
```

#### 弱引用

【是啥】弱引用的整体机制和软引用基本一致，区别在于弱引用包含的对象在垃圾回收时，**不管内存够不够**都会直接被回收。
- 在 JDK 1.2版之后提供了 **WeakReference 类**来实现弱引用，
- 弱引用主要在 **ThreadLocal** 中使用。
- 弱引用对象本身也可以使用**引用队列进行回收**。

【案例】触发弱引用对象的回收

```java
/**
 * 案例：触发弱引用对象的回收
 * - VM参数：-Xmx200m（堆最大内存200mB）
 */
public class Demo {
    public static void main(String[] args) {
        byte[] bytes = new byte[1024 * 1024 * 100];
        WeakReference<byte[]> weakReference = new WeakReference<byte[]>(bytes);
        bytes = null;
        System.out.println(weakReference.get());

        // 内存充足，也会触发回收
        System.gc();
        System.out.println(weakReference.get()); //null
    }
}
```

#### 虚引用和终结器引用

> 这两种引用在常规开发中是不会使用的。

**虚引用**也叫幽灵引用/幻影引用，**不能**通过虚引用对象获取到包含的对象。虚引用唯一的用途是当对象被垃圾回收器回收时可以接收到对应的通知。
- Java 中使用 **PhantomReference 类** 实现了虚引用，
- 直接内存中为了及时知道直接内存对象不再使用，从而回收内存，使用了虚引用来实现。

```java
案例跳过
```

**终结器引用**指的是在对象需要被回收时，终结器引用会关联对象并放置在 **Finalizer 类中的引用队列中**，在稍后由一条由 **FinalizerThread 线程**从队列中获取对象，然后执行对象的 **finalize 方法**，在对象**第二次**被回收时，该对象才真正的被回收。
- 在这个过程中可以在 finalize 方法中再将自身对象使用强引用关联上，但是**不建议**这样做。

> 注意下面的案例代码只是帮助理解，实际中是不会有这样的代码的
> 
> 【提示】一个对象的 finalize 方法最多只会执行一次

【案例】终结器引用

```java
/**
 * 案例：终结器引用
 */
public class FinalizeReferenceDemo {

    public static void main(String[] args) throws Throwable {
        reference = new FinalizeReferenceDemo();
        test();
        test();
        // finalize()执行了...
        // 当前对象还存活
        // 对象已被回收
    }

    // 自救强引用
    public static FinalizeReferenceDemo reference = null;

    public void alive() {
        System.out.println("当前对象还存活");
    }

    @Override
    protected void finalize() throws Throwable {
        try {
            System.out.println("finalize()执行了...");
            // 设置强引用自救
            reference = this;
        } finally {
            super.finalize();
        }
    }

    private static void test() throws InterruptedException {
        reference = null;
        // 回收对象
        System.gc();
        // 执行finalize方法的优先级比较低，休眠500ms等待一下
        Thread.sleep(500);
        if (reference != null) {
            reference.alive();
        } else {
            System.out.println("对象已被回收");
        }
    }
}
```

### 垃圾回收算法

Java 是如何实现垃圾回收的呢？简单来说，垃圾回收要做的有两件事：
1. 找到内存中存活的对象
2. 释放不再存活对象的内存，使得程序能再次利用这部分空间

垃圾回收算法的历史和分类：
- 1960 年 John McCarthy 发布了第一个 GC 算法：标记-清除算法（Mark Sweep GC）。
- 1963 年 Marvin L. Minsky 发布了复制算法。
- 本质上后续所有的垃圾回收算法，都是在上述两种算法的基础上优化而来。

标记-清楚算法 (Mark Sweep GC) ---> 复制算法 (Coping GC) ---> 标记-整理算法 (Mark Compact GC) ---> 分代 GC (Generational GC)

#### 垃圾回收算法的评价标准

STW：
- Java 垃圾回收过程会通过**单独的 GC 线程**来完成，但是不管使用哪一种 GC 算法，都会有部分阶段需要**停止所有的用户线程**。这个过程被称之为 **Stop The World** 简称 STW，
- 如果 STW 时间过长则会影响用户的使用。

> 执行用户代码 ---> GC, STW ---> 执行用户代码 ---> GC, STW ---> 执行用户代码

【案例】STW 测试

```java
/**
 * 案例：STW测试
 * - VM Options: -XX:+UseSerialGC -Xmx10g -verbose:gc
 */
public class StopWorldTest {
    public static void main(String[] args) {
        new PrintThread().start();
        new ObjectThread().start();
    }
}

/**
* 计时线程
*/
class PrintThread extends Thread {
    @SneakyThrows
    @Override
    public void run() {
        // 记录开始时间
        long last = System.currentTimeMillis();
        while (true) {
            long now = System.currentTimeMillis();
            System.out.println(now - last);
            last = now;
            Thread.sleep(100);
        }
    }
}

class ObjectThread extends Thread {
    @SneakyThrows
    @Override
    public void run() {
        List<byte[]> bytes = new LinkedList<>();
        while (true) {
            // 最多存放8g，然后删除强引用，垃圾回收时释放8g
            if (bytes.size() >= 80) {
                bytes.clear();
            }
            bytes.add(new byte[1024 * 1024 * 100]); // 100m
            Thread.sleep(10);
        }
    }
}
```

垃圾回收算法的评价标准：
- 吞吐量：
	- CPU 用于执行用户代码的时间与 CPU 总执行时间的比值，即 `吞吐量 = 执行用户代码时间 /（执行用户代码时间 + GC 时间）`。
	- 吞吐量数值越高，垃圾回收的效率就越高。
- 最大暂停时间：
	- 所有在垃圾回收过程中的 **STW 时间最大值**。
	- 最大暂停时间越短，用户使用系统时受到的影响就越短
- 堆使用效率
	- 不同垃圾回收算法，对堆内存的使用方式是不同的。比如标记清除算法，可以使用完整的堆内存。而复制算法会将堆内存一分为二，每次只能使用一半内存。从堆使用效率上来说，标记清除算法要优于复制算法。

上述三种评价标准不可兼得。
- 一般来说，堆内存越大，最大暂停时间就越长。
- 想要减少最大暂停时间（比如拆分成多次，准备的时间变多了，gc 的总时间变长了），就会降低吞吐量。

不同的垃圾回收算法，适用于不同的场景。

#### 标记-清除算法

核心思想分为两个阶段：
1. 标记阶段：将所有**存活的对象**进行标记。Java 中使用可达性分析算法，从 GC Root 开始通过引用链遍历出所有存活对象。
2. 清除阶段：从内存中删除没有被标记也就是非存活对象。

优点：实现简单，只需要在第一阶段给每个对象维护标志位，第二阶段删除对象即可。

缺点：
1. **碎片化问题**：由于内存是连续的，所以在对象被删除之后，内存中会出现很多细小的可用内存单元。如果我们需要的是一个比较大的空间，很有可能这些内存单元的大小过小无法进行分配。
2. **分配速度慢**：由于内存碎片的存在，需要维护一个空闲链表，极有可能发生每次需要遍历到链表的最后才能获得合适的内存空间

![](assets/Pasted%20image%2020240123224804.png)

#### 复制算法

> #《深入理解Java虚拟机》 
> 
> 为了解决标记-清除算法面对大量可回收对象时执行效率低的问题，1969 年 Fenichel 提出了一种称为“半区复制”（Semispace Copying）的垃圾收集算法
> 
> 将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉
> 
> 优点：
> - 对于多数对象都是可回收的情况，算法需要复制的就是占少数的存活对象
> - 不用考虑内存碎片
>  
>  缺点：
>  - 如果内存中多数对象都是存活的，这种算法将会产生大量的内存间复制的开销
>  - 代价是将可用内存缩小为了原来的一半，空间浪费未免太多了一点
>  
>  现在的商用 Java 虚拟机大多都优先采用了这种收集算法去**回收新生代**，IBM 公司曾有一项专门研究对新生代“朝生夕灭”的特点做了更量化的诠释——新生代中的对象有 98% 熬不过第一轮收集。因此**并不需要按照 1∶1 的比例来划分新生代的内存空间**。

核心思想：
1. 准备两块空间 *From* 空间和 *To* 空间，每次在对象分配阶段，只能使用其中一块空间（From 空间）。
2. 在垃圾回收 GC 阶段，将 From 中存活对象复制到 To 空间。
3. 将两块空间的 **From 和 To 名字互换**。

具体流程：
1. 将堆内存分割成两块 From 空间 To 空间，对象分配阶段，创建对象。
2. GC 阶段开始，将 *GC Root* 搬运到 To 空间
3. 将 GC Root **关联的对象**，搬运到 To 空间
4. 清理 *From* 空间，并把**名称互换**

优点：
- *吞吐量高*：复制算法只需要遍历一次存活对象复制到 To 空间即可，
	- 比*标记-整理*算法少了一次遍历的过程，因而性能较好，
	- 但是不如*标记-清除*算法，因为标记清除算法不需要进行对象的移动
- *不会发生对象的移动*：复制算法在复制之后就会将对象按顺序放入 To 空间中，所以对象以外的区域都是可用空间，不存在碎片化内存空间。

缺点：
- *内存的使用效率低*：每次只能让一半的内存空间来为创建对象使用

#### 标记-整理算法

也叫标记压缩算法，是对标记清理算法中容易产生内存碎片问题的一种解决方案。

核心思想分为两个阶段：
1. *标记阶段*：将所有存活的对象进行标记。Java 中使用可达性分析算法，从 GC Root 开始通过引用链遍历出所有存活对象。
2. *整理阶段*：将存活对象**移动到堆的一端**。清理掉存活对象的内存空间。

![](assets/Pasted%20image%2020240123224741.png)

优点：
- *内存使用效率高*：整个堆内存都可以使用，不会像*复制算法*只能使用半个堆内存
- *不会发生碎片化*：在整理阶段可以将对象往内存的一侧进行移动，剩下的空间都是可以分配对象的有效空间

缺点：
- *整理阶段的效率不高*：整理算法有很多种，比如 Lisp2 整理算法需要对整个堆中的对象搜索3次，整体性能不佳。可以通过 TwoFinger、表格算法、ImmixGC 等高效的整理算法优化此阶段的性能

#### 分代 gc 算法

现代优秀的垃圾回收算法，会将上述描述的垃圾回收算法组合进行使用，其中应用最广的就是分代垃圾回收算法 (Generational GC)。

分代垃圾回收将整个内存区域划分为*年轻代*和*老年代*

分代 gc 过程：
- 分代回收时，创建出来的对象，首先会被放入 *Eden 伊甸园区*。
- 随着对象在 Eden 区越来越多，如果 **Eden 区满**，新创建 1的对象已经无法放入，就会触发年轻代的 GC，称为 *Minor GC* 或者 Young GC。
	- Minor GC 会把 **eden 和 From** 中需要回收的对象回收，把**没有回收的对象放入 To 区**
	- 接下来，S0 会变成 To 区，S1 变成 From 区。当 eden 区满时再往里放入对象，依然会发生 Minor GC。
	- 此时会回收 eden 区和 S1 (from) 中的对象，并把 eden 和 from 区中剩余的对象放入 S0。
	- 注意：每次 Minor GC 中都会为对象记录他的年龄，初始值为 0，每次 GC 完加 1。
- 如果 Minor GC 后对象的年龄达到**阈值（最大 15，默认值和垃圾回收器有关）**，对象就会被晋升至*老年代*。
	- 【By Boer】如果 To 区放不下了没到年龄的应该也会提前晋升老年代
- 当老年代中空间不足，无法放入新的对象时，
	- **先尝试** minor gc，
	- 如果还是不足，就会触发 *Full GC*，Full GC 会**对整个堆进行垃圾回收**。
- 如果 Full GC 依然无法回收掉老年代的对象，那么当**对象继续放入老年代时**，就会抛出 *Out Of Memory 异常*

> 年轻代的 gc 思想是基于复制 gc 算法。
> 
> 【By Boer】幸存者区应该就是 S0 和  S1 中的 to 区
> 
> 【By Boer】为什么要区分 from 和 Eden？复制 gc 算法不是俩区就够了？
> - Eden 区要大一些，内存的利用率高，不然对半分始终有一般的内存没法用，会频繁触发 gc

![](assets/Pasted%20image%2020240124153809.png)

---
【#VM参数】在 JDK8中， `-XX:+UseSerialGC` ：使用分代 gc 的垃圾回收器运行程序。

| 参数名 | 参数含义 |  |
| ---- | ---- | ---- |
| `-Xms` | 设置堆的最小和初始大小，必须是 1024 倍数且大于 1 MB |  |
| `-Xmx` | 设置最大堆的大小，必须是 1024 倍数且大于 2 MB |  |
| `-Xmn` | *新生代* 的大小 |  |
| `-XX:SurvivorRatio` | *伊甸园区* 和 *幸存区* 的比例，默认为 8<br><br>例如：新生代 1 g 内存，伊甸园区 800 MB, S0 和 S1各100MB |  |
| `-XX:+PrintGCDetails` or  `-verbose:gc` | 打印 GC 日志日志 |  |

【案例】分代 gc 演示

```java
/**
 * 案例：分代gc演示
 * - VM参数：-XX:+UseSerialGC -Xms60m -Xmx60m -Xmn20m -XX:SurvivorRatio=3 -XX:+PrintGCDetails
 * - 理论的内存分配：堆 60，年轻代 20，老年代 40，Eden 12，s0 4，s1 4
 */
public class GgcDemo {
    public static void main(String[] args) throws IOException {
        ArrayList<Object> list = new ArrayList<>();
        int count = 0;
        while (true) {
            System.in.read();
            System.out.println(++count);
            // 每次添加1m数据
            list.add(new byte[1024 * 1024]);
        }
    }
}
```

查看日志

```java
9
[GC (Allocation Failure) [DefNew: 12194K->4095K(16384K), 0.0207687 secs] 12194K->9251K(57344K), 0.0210071 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
// 放第10个触发垃圾回收。
// [DefNew: gc前年轻代大小-->gc后年轻代大小, time] gc前堆大小-->gc后堆大小, time]
// 幸存者区只有4m，超出的就会放到老年代里

20
[GC (Allocation Failure) [DefNew: 15570K->3072K(16384K), 0.0097464 secs] 20726K->20515K(57344K), 0.0097842 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

31
[GC (Allocation Failure) [DefNew: 14559K->3072K(16384K), 0.0089644 secs] 32002K->31780K(57344K), 0.0090065 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

42
[GC (Allocation Failure) [DefNew: 14565K->14565K(16384K), 0.0000155 secs][Tenured: 28707K->39972K(40960K), 0.0135928 secs] 43273K->43044K(57344K), [Metaspace: 3748K->3748K(1056768K)], 0.0143287 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

54
[Full GC (Allocation Failure) [Tenured: 40933K->40933K(40960K), 0.0062248 secs] 55751K->55269K(57344K), [Metaspace: 3751K->3751K(1056768K)], 0.0062861 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[Full GC (Allocation Failure) [Tenured: 40933K->40933K(40960K), 0.0022187 secs] 55269K->55269K(57344K), [Metaspace: 3751K->3751K(1056768K)], 0.0022367 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
Heap
 def new generation   total 16384K, 
  eden space 12288K,  93% 
  from space 4096K,  75% 
  to   space 4096K,   0% 
 tenured generation   total 40960K, 
   the space 40960K,  99% 
 Metaspace       used 3782K, capacity 4536K, committed 4864K, reserved 1056768K
  class space    used 415K, capacity 428K, committed 512K, reserved 1048576K
// 老年代满了触发full gc，但还是放不下，抛出 out Of Memory 异常

Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.boer.gc.GgcDemo.main(GgcDemo.java:19)
```

【arthas】 `memory` 命令查看内存，显示出三个区域的内存情况

![](assets/Pasted%20image%2020240124160503.png)

### 垃圾回收器

> 系统中的大部分对象，都是创建出来之后很快就不再使用可以被回收，比如用户获取订单数据，订单数据返回给用户之后就可以释放了。
> 
> 老年代中会存放长期存活的对象，比如 Spring 的大部分 bean 对象，在程序启动之后就不会被回收了。
> 
> 在虚拟机的默认设置中，**新生代大小要远小于老年代的大小**

为什么分代 GC 算法要把堆分成年轻代和老年代？
- 可以通过调整他们的**比例**来适应不同类型的应用程序，**提高内存的利用率和性能**。
- 新生代和老年代使用不同的垃圾回收算法，
	- 新生代 一般选择 **复制 gc** 算法，
	- 老年代 可以选择 **标记-清除 gc** 算法 和 **标记-整理 gc** 算法，由程序员来选择灵活度较高。
- 分代的设计中**允许只回收新生代**（minor gc），如果能满足对象分配的要求就不需要对整个堆进行回收 (full gc)，STW 时间就会减少。

---
【是啥】垃圾回收器是垃圾回收算法的具体实现。

由于垃圾回收器分为年轻代和老年代，**除了 G1 之外**其他垃圾回收器必须**成对组合进行使用**。具体的关系图如下：

![](assets/Pasted%20image%2020240124173211.png)

> 蓝色虚线箭头代表 CMS 会调用 Serial Old 垃圾回收器

#### Serial 和 SerialOld

*Serial* 是一种**单线程串行**回收**年轻代**的垃圾回收器。
- 复制算法
- 优点：单 CPU 处理器下吞吐量非常出色
- 缺点：多 CPU 下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长时间的等待
- 适用场景：Java 编写的**客户端程序**或者**硬件配置有限**的场景

![](assets/Pasted%20image%2020240125160945.png)

*SerialOld* 是 Serial 垃圾回收器的**老年代**版本，采用**单线程串行**回收
- 标记-整理算法
- 优点：单 CPU 处理器下吞吐量非常出色
- 缺点：多 CPU 下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长时间的等待
- 适用场景：与 **Serial 垃圾回收器搭配使用**，或者在 **CMS 特殊情况**下使用

【#VM参数】 `-XX:+UseSerialGC` 新生代、老年代都使用串行回收器。

#### ParNew

*ParNew* 本质是对 Serial 在多 CPU 下的优化，**多线程**垃圾回收**年轻代**的垃圾回收器。
- 复制算法
- 优点：多 CPU 处理器下停顿时间较短
- 缺点：吞吐量和停顿时间不如 G1，所以在 JDK 9 之后不建议使用
- 适用场景：**JDK8 及之前**的版本中，与 CMS 老年代垃圾回收器搭配使用
- 【#VM参数】 `-XX:+UseParNewGC `， 新生代使用 ParNew 回收器，老年代使用串行回收器

![](assets/Pasted%20image%2020240125151027.png)

> 从名字 par 就能看出是并行的（Paralled），多条垃圾收集线程并行工作

#### CMS

*CMS* (Concurrent Mark Sweep) 垃圾回收器关注的是系统的**暂停时间**，允许用户线程和垃圾回收线程在某些步骤中同时执行，减少了用户线程的等待时间。
- 老年代
- 标记清除算法
- 优点：系统由于垃圾回收出现的停顿时间较短，用户体验好
- 缺点：内存碎片问题、退化问题、浮动垃圾问题
- 适用场景：大型的互联网系统中**用户请求数据量大、频率高的场景**比如订单接口、商品接口等
- 【#VM参数】：`-XX:+UseConcMarkSweepGC`

> 关注点更多的是用户线程的停顿时间（提高用户体验）

CMS 执行步骤：
1. *初始标记*，用极短的时间标记出 GC Roots 能直接关联到的对象。
2. *并发标记*，标记所有的对象，**用户线程不需要暂停**。
3. *重新标记*，由于并发标记阶段有些对象会发生了变化，存在错标、漏标等情况，需要重新标记。
4. *并发清理*，清理死亡的对象，**用户线程不需要暂停**

![](assets/Pasted%20image%2020240125161108.png)

缺点：
- CMS 使用了标记-清除算法，在垃圾收集结束之后会出现大量的内存碎片，CMS 会在 Full GC 时进行碎片的整理。这样会导致用户线程暂停，
	- 可以使用 `-XX:CMSFullGCsBeforeCompaction=N` 参数（默认0），调整 N 次 Full GC 之后再整理。
- 无法处理在并发清理过程中产生的 “浮动垃圾”，不能做到完全的垃圾回收。
- 如果老年代内存不足无法分配对象，CMS 就会**退化成 Serial Old** 单线程回收老年代。

#### Parallel Scavenge 和 Parallel Old

*Parallel Scavenge* 是 **JDK8默认**的**年轻代**垃圾回收器，**多线程并行**回收，
- 相较于 ParNew 的特别之处
	- 关注的是系统的**吞吐量**。
	- 具备**自动调整堆内存**大小的特点。
- 复制算法
- 优点：吞吐量高，而且手动可控。为了提高吞吐量，虚拟机会动态调整堆的参数
- 缺点：不能保证**单次**的停顿时间
- 适用场景：后台任务，**不需要与用户交互**，并且**容易产生大量的对象**。比如：大数据的处理，大文件导出

> 并行

![](assets/Pasted%20image%2020240125163925.png)

*Parallel Old* 是为 Parallel Scavenge 收集器设计的**老年代**版本，利用**多线程**并发收集。
- 标记整理算法
- 优点：并发收集，在多核 CPU 下效率较高
- 缺点：暂停时间会比较长
- 适用场景：**与 Parallel Scavenge 配套使用**

> 教案写错了应该，并行

【#VM参数】 
-  `-XX:+UseParallelGC`：Parallel 处理器+老年代串行
- `-XX:+UseParallelOldGC` ：以使用 Parallel Scavenge + Parallel Old 这种组合。

---
Parallel Scavenge 允许手动设置**最大**暂停时间和吞吐量。
- Oracle 官方建议在使用这个组合时，**不要设置堆内存的最大值**，垃圾回收器会根据最大暂停时间和吞吐量自动调整内存大小。

【#VM参数】
- 最大暂停时间： `-XX:MaxGCPauseMillis=n` 设置每次垃圾回收时的最大停顿毫秒数
- 吞吐量：`-XX:GCTimeRatio=n` 设置吞吐量为 n（用户线程执行时间 = `n/(n + 1)`）
- 自动调整内存大小：`-XX:+UseAdaptiveSizePolicy` 设置可以让垃圾回收器根据吞吐量和最大停顿的毫秒数自动调整内存大小

#### G1

**JDK9 之后默认**的垃圾回收器是 G1（Garbage First）垃圾回收器。

> 1）*Parallel Scavenge* 关注吞吐量，允许用户设置最大暂停时间，但是会**减少年轻代可用空间**的大小。
> 
> 2）*CMS* 关注暂停时间，但是**吞吐量方面会下降**。

而 G1 **设计目标**就是将上述两种垃圾回收器的**优点融合**：
1. 支持巨大的堆空间回收，并有较高的吞吐量。
2. 支持多CPU并行垃圾回收。
3. 允许用户设置最大暂停时间。

> JDK9 之后强烈建议使用 G1 垃圾回收器

内存结构
- G1 的整个堆会被划分成多个**大小相等**的区域，称之为区 Region，区域**不要求是连续**的。分为 Eden、Survivor、Old 区。
- Region 的大小通过 `堆空间大小/2048` 计算得到，
- 【#VM参数】 `-XX:G1HeapRegionSize=32m` 也可以指定
- Region size 必须是2的指数幂，取值范围从1M 到32M。

---
G1 垃圾回收有两种方式：

1）年轻代回收（Young GC）

回收 **Eden 区和 Survivor 区**中不用的对象。

会导致 **STW**，
- 【#VM参数】 `-XX:MaxGCPauseMillis=n`（默认 200） 设置每次垃圾回收时的**最大暂停时间毫秒数**，G1 垃圾回收器会尽可能地保证暂停时间。

执行流程：
- 新创建的对象会存放在 Eden 区。当 G1 判断年轻代区不足（max 默认 60%），无法分配对象时需要回收时会执行 Young GC。
- **标记**出 Eden 和 Survivor 区域中的存活对象，
- 根据配置的最大暂停时间**选择某些区域**将存活对象**复制到一个新的 Survivor 区中（年龄+1）**，**清空**这些区域。
	- 如何选择区域？：G1 在进行 Young GC 的过程中会去记录每次垃圾回收时每个 Eden 区和 Survivor 区的平均耗时，以作为下次回收时的参考依据。这样就可以**根据配置的最大暂停时间计算出本次回收时最多能回收多少个 Region 区域了**。
	- 比如 `-XX:MaxGCPauseMillis=n`（默认 200），每个 Region 回收耗时 40 ms，那么这次回收最多只能回收 4 个 Region。
- 后续 Young GC 时与之前相同，只不过 Survivor 区中存活对象会被**搬运到另一个 Survivor 区**。
- 当某个存活对象的年龄到达阈值（默认 15），将被放入老年代
- 部分对象如果**大小超过 Region 的一半，会直接放入老年代**，这类老年代被称为 **Humongous 区**。
	- 比如堆内存是 4G，每个 Region 是 2M，只要一个大对象超过了 1M 就被放入 Humongous 区，如果对象过大会横跨多个 Region。

2）混合回收（Mixed GC）

多次回收之后，会出现很多 Old 老年代区，此时**总堆占有率**达到阈值时会触发混合回收。
- 【#VM参数】总堆占有率阈值 `-XX:InitiatingHeapOccupancyPercent` (默认 45%)

回收区域：
- 所有年轻代
- 部分老年代的对象
- 大对象区

执行流程：
- 初始标记（initial mark）：标记 GC Roots 引用的对象为存活
- 并发标记（concurrent mark）：将第一步中标记的对象引用的对象，标记为存活
- 最终标记（remark 或者 FinalizeMarking）：标记一些引用改变漏标的对象不管新创建、不再关联的对象
- 并发清理（cleanup）：将存活对象复制到别的 Region
	- 采用**复制算法**来完成，不会产生内存碎片

![](assets/Pasted%20image%2020240127235723.png)

G1 对老年代的清理会选择**存活度最低**的区域来进行回收，这样可以保证回收效率最高，这也是 G1（Garbage first）名称的由来。

注意：如果清理过程中发现没有足够的空 Region 存放转移的对象，会出现 **Full GC**。单线程执行**标记-整理**算法，此时会导致用户线程的暂停。所以尽量保证应该用的堆内存有一定多余的空间

【案例】G 1 垃圾回收器演示

```java
/**
 * -XX:+PrintGCDetails -XX:+UseG1GC -XX:+PrintFlagsFinal
 */
public class G1GCDemo {
    public static void main(String[] args) {
        int count = 0;
        ArrayList<Object> list = new ArrayList<>(); // 有索引连着对象
        while (true) {
            if (count++ % 1024 * 10 == 0) { // 0.5M*10K=5G
                list.clear();
            }
            list.add(new byte[1024 * 1024 * 1 / 2]); // 0.5M
        }
    }
}
```

---
G1 总结：
- 【#VM参数】
	- `-XX:+UseG1GC` 打开 G1 的开关，JDK 9 之后默认不需要打开
	- `-XX:MaxGCPauseMillis=毫秒值` 最大暂停的时间
- 回收年代和算法
	- 年轻代+老年代
	- 复制算法
- 优点：
	- 对比较大的堆如超过 6G 的堆回收时，延迟可控
	- 不会产生内存碎片
	- 并发标记的 SATB 算法效率高
- 缺点：JDK 8 之前还不够成熟
- 适用场景：JDK 8 最新版本、JDK 9 之后建议默认使用

#### 总结

JDK8 及之前：
- ParNew + CMS（关注暂停时间）、
- Parallel Scavenge + Parallel Old (关注吞吐量)、 
- G1（JDK8之前不建议，较大堆并且关注暂停时间）

JDK9 之后:
- G1（默认）

从 JDK9 之后，由于 G1 日趋成熟，JDK 默认的垃圾回收器已经修改为 G1，所以**强烈建议在生产环境上使用 G1。**



