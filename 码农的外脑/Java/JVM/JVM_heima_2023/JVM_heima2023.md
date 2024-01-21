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

## 初始化阶段

执行**静态代码块**中的代码，并为**静态变量**赋值
- 执行字节码文件中 **clinit** 部分的字节码指令

> Demo 的源代码和 clinit 部分的字节码指令

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

==clinit 方法中的执行顺序与 Java 中编写的顺序是一致的。==
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

> 导致类的初始化的几种方式

1. 访问一个类的静态变量或者静态方法，注意==变量是 final 修饰的并且等号右边是常量不会触发初始化。==
2. 调用 `Class.forName(String className)`。
3. new 一个该类的对象时。
4. 执行 Main 方法的当前类。

添加 `-XX:+TraceClassLoading` 参数可以打印出加载并初始化的类

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

> 某大型互联网公司2019笔试题——考察了类的初始化执行顺序

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
  
    {        
	    System.out.println("C");  
    }  
  
    static {  
        System.out.println("D");  
    }  
}
```

> clinit 指令在特定情况下不会出现

比如，如下几种情况是不会进行初始化指令执行的：
1. 无静态代码块且无静态变量赋值语句。
2. 有静态变量的声明，但是没有赋值语句。
3. 静态变量的定义使用 final 关键字，这类变量会在准备阶段直接进行初始化。

--- 
- 直接访问父类的静态变量，不会触发子类的初始化。
- 子类的初始化 clinit 调用之前，会**先调用**父类的 clinit 初始化方法

【案例】某大型互联网公司2021笔试题
```java
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

结果：输出 2。如果把 `new B();` 去掉，输出 1。

--- 
数组的创建**不会导致**数组中元素的类进行初始化

```java
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

---
final 修饰的变量，如果赋值的内容==需要执行指令才能得出结果==，会执行 clinit 方法进行初始化。

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

# 类加载器

类加载器（ClassLoader）是 Java 虚拟机提供给应用程序去实现获取类和接口**字节码数据**的技术。

类加载器只参与**加载**过程中的字节码获取并**加载到内存**这一部分。

>By Boer. 
>
>抽象的规范，实际的 Java 实现的 ClassLoader 类中存放了加载的类的 Class 对象的集合

![](assets/Pasted%20image%2020240117154818.png)
- 本地接口 JNI 是 Java Native Interface 的缩写，允许 Java 调用其他语言编写的方法。在 hotspot 类加载器中，主要用于调用 Java 虚拟机中的方法，这些方法使用 C++编写。

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
- 一类是 Java **代码**中实现的，
	- JDK 中默认提供或者自定义：JDK 中默认提供了多种处理不同渠道的类加载器，程序员也可以自己根据需求定制
	- 所有 Java 中实现的类加载器都需要继承 **ClassLoader 抽象类**
- 一类是 Java 虚拟机**底层源码**实现的。
	- 虚拟机底层实现：源代码位于 Java 虚拟机的源码中，实现语言与虚拟机底层语言一致，比如 Hotspot 使用 C++
	- 保证 Java 程序**运行中基础类**被正确地加载，比如 `java.lang.String`，确保其可靠性

类加载器的设计 JDK8 和 8 之后的版本差别较大，**JDK8 及之前**的版本中默认的类加载器有如下几种: 
- 虚拟机底层实现 c++：
	- **启动类加载器** BootStrap：加载 java 中最核心的类。
- Java 实现：
	- **扩展类加载器** Extension：允许扩展 Java 中比较通用的类
	- **应用程序类加载器** Application：加载应用使用的类

【Arthas】 `classloader` 命令查看类加载器具体信息

-  `classloader` ：查看
	- 类加载器 的继承树，
	- Urls，
	- 类加载信息，
- `classloader -l`：类加载实例进行统计
- `classloader -c 哈希值`：看 URLClassLoader 实际的 urls

![](assets/Pasted%20image%2020240117162352.png)

![](assets/Pasted%20image%2020240118143639.png)

#### 启动类加载器

- 启动类加载器（Bootstrap ClassLoader）是由 **Hotspot 虚拟机** 提供的、使用 C++编写的类加载器。
- 默认加载 Java **安装目录 `/jre/lib`** 下的类文件，比如 rt.jar，tools.jar，resources.jar 等。
	- rt.jar（runtime）是 JDK 8 非常核心的 jar 包 （包括 String，Thread 等）

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

- 都是 JDK 中提供的、使用 Java 编写的类加载器。
- 它们的源码都位于 **sun. Misc. Launcher 类**中，都是**静态内部类**。**继承**自 **URLClassLoader**。具备通过 **目录或者指定 jar 包** 将字节码文件加载到内存中。

```java
public class Launcher {
	static class ExtClassLoader extends URLClassLoader{}
	static class AppClassLoader extends URLClassLoader{}
}
```

![](assets/Pasted%20image%2020240117181243.png)

扩展类加载器（Extension Class Loader）是 
- JDK 中提供的、使用 Java 编写的类加载器。
- 默认加载 Java **安装目录 /jre/lib/ext** 下的类文件。

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
- 【推荐】使用 **参数** 扩展： `-Djava.ext.dirs=jar包目录[分隔符]用户jar包目录` ，这种方式会覆盖掉原始目录（应该是 `/jre/lib/ext`，By Boer.），所以用分隔符追加上原始目录。
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

### 双亲委派机制

【解决什么问题？】由于 Java 虚拟机中有多个类加载器，双亲委派机制的核心是解决**一个类到底由谁加载**的问题。
- 保证类加载的安全性：通过双亲委派机制避免恶意代码替换 JDK 中的核心类库，比如`Java.Lang.String`，确保核心类库的完整性和安全性
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

【Arthas 命令】 `classloader -t`：打印所有 ClassLoader 的继承树

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

1）自定义类加载器并且重写 loadClass 方法，就可以将双亲委派机制的代码去除
- Tomcat 通过这种方式实现应用之间类隔离，《面试篇》中分享它的做法

2）利用上下文类加载器加载类，比如 JDBC 和 JNDI 等

3）历史上 Osgi 框架实现了一套新的类加载器机制，允许同级之间委托进行类的加载

#### 自定义类加载器

一个 Tomcat 程序中是可以运行多个 Web 应用的，如果这两个应用中出现了**相同限定名的类**，比如 Servlet 类，Tomcat 要保证这两个类都能加载并且它们应该是不同的类。

如果不打破双亲委派机制，当应用类加载器加载 Web 应用1中的 MyServlet 之后，Web 应用2中相同限定名的 MyServlet 类就无法被加载了

Tomcat 使用了自定义类加载器来实现应用之间类的隔离。每一个应用会有一个独立的类加载器加载对应的类。
- 应用 1 的类加载器--->加载 Web 应用 1 的 MyServlet
- 应用 2 的类加载器--->加载 Web 应用 2 的 MyServlet

> 分析 ClassLoader 的原理

ClassLoader 中包含了4个核心方法。双亲委派机制的核心代码就位于 **loadClass** 方法中。

- `public Class<?> loadClass(String name)`（**重要**）
	- 类加载的入口，提供了双亲委派机制，内部会调用 `findClass()`
- `protected Class<?> findClass(String name)`（**重要**）
	- 由类加载器子类实现，获取二进制数据调用 defineClass ，比如 URLClassLoader 会根据文件路径去获取类文件中的二进制数据。
- `protected final Class<?> defineClass(byte[] b, int off, int len)`
	- 做一些类名的校验，然后调用虚拟机底层的方法将字节码信息加载到虚拟机内存中
- `protected final void resolveClass(Class<?> c)`
	- 执行类生命周期中的连接阶段

> 打破双亲委派机制 

核心就是将 loadClass 中下面的代码重新实现

![](assets/Pasted%20image%2020240118181312.png)

> 自定义类加载器案例

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

> 自定义类加载器的默认父类加载器是 **AppClassLoader** 

【提示】当创建子类对象时，不管使用子类的哪个构造器，默认情况下总会去调用**父类的无参**构造

以 Jdk8 为例，ClassLoader 类中提供了无参构造方法设置 parent 的内容：

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

> 两个自定义类加载器加载相同限定名的类，不会冲突吗？

不会冲突，在同一个 Java 虚拟机中，只有**相同类加载器+相同的类限定名**才会被认为是同一个类。

> 在 Arthas 中使用 `sc –d 类名` 的方式查看类具体的情况

![](assets/Pasted%20image%2020240118193804.png)

> 正确的实现一个自定义类加载器的方式

重写 **findClass** 方法，这样不会破坏双亲委派机制。

#### 线程上下文类加载器

案例：JDBC 和 JNDI 等

JDBC 中使用了 DriverManager 来管理项目中引入的不同数据库的驱动，比如 mysql 驱动、oracle 驱动

DriverManager 类位于 rt.jar 包中，由**启动类加载器**加载。而依赖中的 mysql 驱动对应的类，由**应用程序类加载器**来加载，这就**违反**了双亲委派机制。

![](assets/Pasted%20image%2020240119120228.png)

DriverManager怎么知道jar包中要加载的驱动在哪儿？

> Spi 全称为 (Service Provider Interface)，是 JDK 内置的一种服务提供发现机制。
> 
> Spi 的工作原理：
> 1. 在 ClassPath 路径下的 **META-INF/services** 文件夹中，以**接口的全限定名**来命名文件名，对应的文件里面写该 **接口的实现**。
> 2. 使用 **ServiceLoader** 加载实现类
> 
> ![](assets/Pasted%20image%2020240119121047.png)

DriverManager 使用 SPI 机制，最终加载 jar 包中对应的驱动类
- **启动类加载器**加载 DriverManager。
- 在初始化 DriverManager 时，通过 SPI 机制加载 jar 包中的 mysql 驱动。
- SPI 中利用了**线程上下文类加载器**（应用程序类加载器）去加载类并创建对象

这种由启动类加载器加载的类，委派应用程序类加载器去加载类的方式，打破了双亲委派机制。

![](assets/Pasted%20image%2020240119143017.png)

JDBC 案例中真的打破了双亲委派机制吗？

1. 打破了：这种由启动类加载器加载的类，委派应用程序类加载器去加载类的方式。打破了双亲委派机制。
2. 没有打破：JDBC 只是在 DriverManager 加载完之后，通过**初始化阶段**触发了驱动类的加载，类的加载依然遵循双亲委派机制（应用程序类加载器还是要向上再向下）。

#### Osgi 框架的类加载器

历史上，OSGi 模块化框架。它存在同级之间的类加载器的委托加载。OSGi 还使用类加载器实现了热部署的功能。

**热部署**指的是在服务不停止的情况下，动态地更新字节码文件到内存中。

--- 
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

### JDK9 之后的类加载器

JDK8 及之前的版本中，扩展类加载器和应用程序类加载器的源码位于 rt.jar 包中的 `sun.misc.Launcher.java`。

由于 JDK 9 引入了 module 的概念，类加载器在设计上发生了很多变化。

启动类加载器

- 使用 **Java 编写**，位于 `jdk.internal.loader.ClassLoaders` 类中。
- Java 中的 BootClassLoader **继承自 BuiltinClassLoader** 实现从**模块**中找到要加载的字节码资源文件。
- 启动类加载器依然无法通过 java 代码获取到，返回的仍然是 null，保持了统一。

扩展类加载器 被替换成了 **平台类加载器（Platform Class Loader）**

- 遵循**模块化**方式加载字节码文件，所以**继承**关系从 URLClassLoader 变成了 **BuiltinClassLoader**，BuiltinClassLoader 实现了从模块中加载字节码文件。
- 平台类加载器的存在更多的是为了与老版本的设计方案兼容，自身没有特殊的逻辑。

# JVM 的内存区域

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
- 因为每个线程只存储一个固定长度的内存地址，程序计数器是不会发生内存溢出的。
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

主要包含**动态链接、方法出口、异常表**的引用

**动态链接**：保存了 编号到 运行时常量池 的内存地址的映射关系
- 当前类的字节码指令引用了其他类的属性或者方法时，需要将**符号引用**（编号）转换成对应的**运行时常量池中的内存地址**。

![](assets/Pasted%20image%2020240120154811.png)

**方法出口**：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址。

![](assets/Pasted%20image%2020240120155557.png)

**异常表**存放的是代码中异常的处理信息，包含了异常**捕获的生效范围**以及异常**发生后跳转**到的字节码指令位置。

![](assets/Pasted%20image%2020240120160903.png)

#### 栈内存溢出 and 大小设置

Java 虚拟机栈如果栈帧过多，占用内存超过栈内存可以分配的最大大小就会出现内存溢出。

Java 虚拟机栈内存溢出时会出现 **StackOverflowError** 的错误

如果我们不指定栈的大小，JVM 将创建一个具有**默认大小**的栈。大小取决于操作系统和计算机的体系结构

![](assets/Pasted%20image%2020240120163257.png)

要修改 Java 虚拟机栈的大小，可以使用虚拟机参数 `-Xss` 。

- 语法：`-Xss` 栈大小
- 单位：字节（默认，必须是 1024 的倍数）、k 或者 K (KB)、m 或者 M (MB)、g 或者 G (GB)

模拟栈内存的溢出
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
- `-XX:ThreadStackSize=1024` 调整标志来配置堆栈大小，推荐 `-Xss`
- HotSpot JVM 对栈大小的**最大值和最小值**有要求。Windows（64 位）下的 JDK 8 测试最小值为 180 k，最大值为 1024m
- 局部变量**过多**、操作数栈**深度过大**也会影响栈内存的大小
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
- 栈上的局部变量表中，可以存放堆上**对象的引用**。静态变量也可以存放堆对象的引用，通过**静态变量**就可以实现对象在**线程之间共享**。
- 堆内存大小是有上限的，一直向堆中放入对象达到上限之后，就会抛出 **OutOfMemoryError**

 > 类的生命周期---加载阶段
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




## 方法区

## 直接内存





# JVM 的垃圾回收
