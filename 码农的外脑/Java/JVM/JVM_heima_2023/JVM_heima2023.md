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

## 字节码文件的组成

### 字节码文件的查看

字节码文件中保存了==源代码编译之后的内容==，以==二进制的方式存储==，无法直接用记事本打开阅读。

推荐使用 jclasslib 工具查看字节码文件。Idea 有 jclasslib 插件
- [Releases · ingokegel/jclasslib (github.com)](https://github.com/ingokegel/jclasslib/releases)

### 字节码文件的核心组成

- 基本信息
- 常量池
- 字段
- 方法
- 属性

#### 基本信息

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

#### 常量池

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

#### 字段

当前类或接口声明的字段信息
![](assets/Pasted%20image%2020240115231407.png)

#### 方法

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

#### 属性

类的属性，比如源码的文件名、内部类的列表等
![](assets/Pasted%20image%2020240115231742.png)

### 玩转字节码常用工具

#### javap

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

#### 阿里 arthas

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

#### 小总结

- 本地文件可以使用 iclasslib 工具查看，开发环境使用 iclasslib 插件
- 服务器上文件使用 javap 命令直接查看，也可以通过 arthas 的 dump 命令导出字节码文件再查看本地文件。还可以使用 jad 命令反编译出源代码

## 类的生命周期

类的生命周期描述了一个类加载、使用、卸载的整个过程。

分为以下阶段：
- 加载
- 连接
	- 验证
	- 准备
	- 解析
- 初始化
- 使用
- 卸载

其中 初始化阶段 程序员可以干涉，是面试的高频考点

### 加载阶段

1）**类加载器**根据类的全限定名通过不同的渠道以二进制流的方式获取字节码信息（加载到内存）。==程序员可以使用 Java 代码拓展的不同的渠道==

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

作用是在 Java 代码中去 获取类的信息 以及 ==存储静态字段的数据 (JDK8及之后)==
![](assets/Pasted%20image%2020240116211409.png)

对于开发者来说，只需要访问堆中的 Class 对象而不需要访问方法区中所有信息。这样 ==Java 虚拟机就能很好地控制开发者访问数据的范围==

### 连接阶段

可以细分为三个阶段：
- 验证：验证内容是否满足《Java 虚拟机规范》
- 准备：给静态变量赋初值
- 解析：将常量池中的符号引用替换成指向内存的直接引用

> 验证

主要目的是检测 Java 字节码文件是否遵守了《Java 虚拟机规范》中的约束。==这个阶段一般不需要程序员参与。==

主要包含如下四部分，具体详见《Java 虚拟机规范》
1. 文件格式验证，比如文件是否以 `0xCAFEBABE` 开头，主次版本号是否满足当前 Java 虚拟机版本要求
2. 元信息验证，例如类必须有父类 (super 不能为空)
3. 验证程序执行指令的语义，比如方法内的指令执行到一半强行跳转到其他方法中去 
4. 符号引用验证，例如是否访问了其他类中 private 的方法等

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

> 解析

解析阶段主要是将常量池中的符号引用替换为直接引用。
- **符号引用**就是在字节码文件中==使用编号来访问常量池中的内容==
- **直接引用**不再使用编号，而是使用==内存中地址==进行访问具体的数据。

### 初始化阶段





# JVM 的内存区域


# JVM 的垃圾回收
