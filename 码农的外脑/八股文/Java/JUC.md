# ---------- 基本概念
## 进程和线程

“进程”是**程序的一次执行过程**，是系统运行程序的基本单位，因此进程是动态的。
- 系统运行一个程序即是一个进程从创建，运行到消亡的过程。
- 在 Java 中，启动 main 函数时其实就是启动了一个 JVM 的进程，而 main 函数所在的线程就是这个进程中的一个线程，也称**主线程**。

线程与进程相似，操作系统进行调度的基本单元。线程也被称为“**轻量级进程**”

线程与进程不同的是，
- 一个进程在其执行的过程中可以产生多个线程。
- 是一个比进程更小的**执行单位**
- 线程执行开销小，但不利于资源的管理和保护；而进程正相反。
	- 系统在**产生一个线程**，或是在**各个线程之间作切换工作**时，负担要比进程小得多，
- 同类的多个线程**共享**进程的“堆”和“方法区”资源，但每个线程有自己的“程序计数器”、“虚拟机栈”和“本地方法栈”。
- 各进程是**独立**的，同一进程中的线程极有可能会**相互影响**

> 【拓展】
> 
> 程序计数器为什么是线程私有的?
> - 线程切换后能恢复到正确的执行位置
> 
> 虚拟机栈和本地方法栈为什么是线程私有的?
> - 保证线程中的局部变量不被别的线程访问到

## Java 线程和 os 里的线程的区别是什么？(2)

[Java并发常见面试题总结（上） | JavaGuide](https://javaguide.cn/java/concurrent/java-concurrent-questions-01.html#java-%E7%BA%BF%E7%A8%8B%E5%92%8C%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%BA%BF%E7%A8%8B%E6%9C%89%E5%95%A5%E5%8C%BA%E5%88%AB)

**现在的 Java 线程的本质其实就是操作系统的线程。**

JDK 1.2 之前，Java 线程是基于绿色线程（Green Threads）实现的，这是一种**用户级线程**（用户线程），也就是说 JVM 自己模拟了多线程的运行，而不依赖于操作系统。

在 JDK 1.2 及以后，Java 线程改为**基于“原生线程”（Native Threads）实现**，也就是说 JVM 直接使用**操作系统原生的内核级线程**（内核线程）来实现 Java 线程，由操作系统内核进行线程的调度和管理。

> 【提示】
> 
> 绿色线程和原生线程比起来在使用时有一些限制（比如绿色线程不能直接使用操作系统提供的功能如异步 I/O、只能在一个内核线程上运行无法利用多核）
> 
> 用户线程创建和切换成本低，但不可以利用多核。内核态线程，创建和切换成本高，可以利用多核。

在 **Windows** 和 **Linux** 等主流操作系统中，Java 线程采用的是 **“一对一”的线程模型**，也就是一个 Java 线程对应一个系统内核线程。

> **Solaris** 系统是一个特例（Solaris 系统本身就支持多对多的线程模型），HotSpot VM 在 Solaris 上支持**多对多**和**一对一**。具体可以参考 R 大的回答: [JVM 中的线程模型是用户级的么？open in new window](https://www.zhihu.com/question/23096638/answer/29617153)。

> 虚拟线程在 **JDK 21** 顺利转正，关于虚拟线程、平台线程（也就是我们上面提到的 Java 线程）和内核线程三者的关系可以参考：[Java 20 新特性概览](/java/new-features/java20.html)。

## 并发 和 并行

#字节_23_秋招_Java  

并发：
- 是在**同一实体**上的多个事件,
- 是在同一台处理器上的**同一时间段**内处理多个任务，
- 同一时刻，其实是只有一个事件在发生。

并行：
- 是在**不同实体**上的多个事件，
- 是在多台处理器上“**同时**”处理多个任务，
- 同一时刻，大家都真的在做事情，你做你的，我做我的

## 同步 和 异步

- 同步：发出一个调用之后，在没有得到结果之前， 该调用就不可以返回，一直等待。
- 异步：调用在发出之后，不用等待返回结果，该调用直接返回。

## 为什么要使用多线程？

先从总体上来说：

- *从计算机底层来说*： 
	- 线程可以比作是轻量级的进程，是程序执行的最小单位,线程间的切换和调度的成本远远小于进程。
	- 另外，多核 CPU 时代意味着多个线程可以同时运行，这减少了线程上下文切换的开销。
- *从当代互联网发展趋势来说*： 现在的系统动不动就要求百万级甚至千万级的并发量，而多线程并发编程正是开发高并发系统的基础，利用好多线程机制可以大大提高系统整体的并发能力以及性能。

再深入到计算机底层来探讨：

- *单核时代* ：
	- 主要是为了**提高单进程利用 CPU 和 IO 系统的效率**。 
		- 假设只运行了一个 Java 进程的情况，当我们请求 IO 的时候，如果 Java 进程中只有一个线程，此线程被 IO 阻塞则整个进程被阻塞。CPU 和 IO 设备只有一个在运行，那么可以简单地说系统整体效率只有 50%。当使用多线程的时候，**一个线程被 IO 阻塞，其他线程还可以继续使用 CPU**。从而**提高了 Java 进程利用系统资源的整体效率**。
	- #Boer 用户交互及时（例子：wx）
- *多核时代* : 
	- 主要是为了**提高进程利用多核 CPU 的能力**。
		- 举个例子：假如我们要计算一个复杂的任务，我们只用一个线程的话，不论系统有几个 CPU 核心，都只会有一个 CPU 核心被利用到。而创建多个线程，这些线程可以被映射到底层多个 CPU 上执行，在任务中的**多个线程没有资源竞争的情况下，任务执行的效率会有显著性的提高**，约等于（单核时执行时间/CPU 核心数）。

## 使用多线程可能带来什么问题？

**并发编程并不总是能提高程序运行速度的**，且可能会遇到很多问题，比如：
- 内存泄漏、
- 死锁、
- 线程不安全
- 等等。

如何理解线程安全和不安全？
- 线程安全和不安全是在**多线程环境**下对于**同一份数据的访问是否能够保证其正确性和一致性**的描述。
- 线程安全：不管有多少个线程同时访问，都能保证这份数据的正确性和一致性。
- 线程不安全：多个线程同时访问时可能会导致数据混乱、错误或者丢失。

## 什么是线程上下文切换？

> 哪些情况引起线程上下文切换

上下文：线程在执行过程中会有自己的**运行条件和状态**（也称"上下文"），比如程序计数器，栈信息等。

当出现如下情况的时候，线程会从占用 CPU 状态中退出

- 主动让出 CPU，比如调用了 `sleep()`, `wait()` 等
- 时间片用完，
	- 因为操作系统要防止一个线程或者进程长时间占用 CPU 导致其他线程或者进程饿死。
- 调用了阻塞类型的系统中断，比如请求 IO，线程被阻塞。
- 被终止或结束运行

上下文切换：这其中**前三种**都会发生线程切换，意味着需要**保存当前线程的上下文**，留待线程下次占用 CPU 的时候**恢复现场**。并加载**下一个**将要占用 CPU 的线程上下文。

> 切出去还要切回来

上下文切换是现代操作系统的基本功能，因其每次需要保存信息恢复信息，这将会占用 CPU，内存等系统资源进行处理，也就意味着效率会有一定损耗，如果**频繁切换就会造成整体效率低下**

## 单核 CPU 上运行多个线程效率一定会高吗？

单核 CPU 同时运行多个线程的效率是否会高，取决于**线程的类型**和**任务的性质**。

一般来说，有两种类型的线程：

- *CPU 密集型*：线程主要进行计算和逻辑处理，需要占用大量的 CPU 资源
- *IO 密集型*：线程主要进行输入输出操作，如读写文件、网络通信等，需要等待 IO 设备的响应，而不占用太多的 CPU 资源。

在单核 CPU 上，同一时刻只能有一个线程在运行，其他线程需要等待 CPU 的时间片分配

- 如果线程是 CPU 密集型的，那么多个线程同时运行会导致**频繁的线程切换**，增加了系统的开销，降低了效率。
- 如果线程是 IO 密集型的，那么多个线程同时运行可以**利用 CPU 在等待 IO 时的空闲时间**，提高了效率。

# ---------- Java 线程

## 线程的生命周期和状态？线程状态的转换？

#B站SRE_Java后端 

> 其他问法：
> - 有实战监控过吗？(1)

Java 线程在运行的生命周期中共有6 种状态（根据 `Thread.State` 枚举类）：

- *NEW* : 初始状态，线程被**创建**出来但没有被调用 `start()` 。
- *RUNNABLE* : 运行/就绪状态，线程被调用了 `start()` **等待运行/运行**的状
- *BLOCKED* ：阻塞状态，**需要等待锁释放**
- *WAITING* ：等待状态，表示该线程需要等待其他线程做出一些特定动作（通知或中断）
- *TIME_WAITING* ：超时等待状态，可以在指定的时间后自行返回而不是像 WAITING 那样一直等待
- *TERMINATED* ：终止状态，表示该线程已经运行完毕

![|600](assets/Pasted%20image%2020240204194329.png)

- 当线程进入 `synchronized` 方法/块或者调用 `wait` 后（被 `notify`）重新进入 `synchronized` 方法/块，但是锁被其它线程占有，这个时候线程就会进入 **`BLOCKED（阻塞）`** 状态

## 线程创建方式(1)

- Thread：自定义线程类继承 Thread 类，重写 `run()` 方法
- Runnable 配合 Thread：
	- 实现 Runnable 接口重写 `run()` 方法，
	- 创建 Thread 对象，构造器传入 Runnable 对象
- FutureTask 配合 Thread
	- 实现 Callable 接口重写 `call()` 方法
	- 创建 FutureTask 的对象，构造器传入 Callable 对象
	- 创建 Thread 对象，构造器传入 FutureTask 对象
- 创建任务对象，交给线程池

严格来说，Java 就只有一种方式可以创建线程，那就是通过 `new Thread().start()` 创建。不管是哪种方式，最终还是依赖于 `new Thread().start()`

## start & run

> 可以直接调用 Thread 类的 run 方法吗？

`start()` 会执行线程的相应准备工作，然后自动执行 `run()` 方法的内容，这是真正的多线程工作。 

但是，直接执行 `run()` 方法，会把 `run()` 方法当成一个 **main 线程下的普通方法**去执行，并不会在某个线程中执行它，所以这并不是多线程工作

## sleep & wait

> 区别？ #PDD_23_秋招_后端

共同点：

- 两者都可以暂停线程的执行。
- 线程状态都变成 `TIMED_WAITING`

区别：

- *锁*：
	- `wait()` 需要**和 syncronized 配合使用**，`sleep()` 不强制
	- `wait()` 会释放锁，`sleep()` 不会
- *使用场景*：`wait()` 通常被用于线程间交互/通信，`sleep()` 通常被用于暂停执行
- *唤醒机制*：
	- `wait()` 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 `notify()` 或者 `notifyAll()` 方法。
		- 或者也可以使用 `wait(long timeout)` 超时后线程会自动苏醒。
	- `sleep()` 方法执行完成后，线程会自动苏醒
- *所属类*：`sleep()` 是 `Thread` 类的**静态本地**方法，`wait()` 则是 `Object` 类的**本地**方法

> 为什么 `wait()` 方法不定义在 `Thread` 中？
> - `wait()` 是让获得对象锁的线程实现等待，会自动释放当前线程占有的对象锁。每个对象 `Object` 都拥有对象锁，既然要释放当前线程占有的对象锁并让其进入 WAITING 状态，自然是要操作对应的对象（`Object`）而非当前的线程（`Thread`）。
> 
> 为什么 `sleep()` 方法定义在 `Thread` 中？
> - 因为 `sleep()` 是让当前线程暂停执行，不涉及到对象类，也不需要获得对象锁

# ---------- 共享模型_管程  

## synchronized

###  介绍下 synchronized 关键字

#得物_实习_Java

主要解决的是多个线程之间访问资源的同步性，可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行

### 如何使用 Synchronized？

> synchronized怎么去使用，有哪几种用法？ #PDD_暑期实习_后端 
> 
> synchronized 加在普通方法上和静态方法上有什么区别？ #PDD_暑期实习_后端 #字节_23_秋招_Java 
> 
> #todo 加在 final 修饰的方法上 #字节_23_秋招_Java 
> 
> JDK1.8 里的 sychronized 锁是锁的哪里？ #小红书_实习_后端 

修饰实例方法：线程进入方法前要获得 **当前对象实例（this）的锁**

修饰静态方法：线程进入方法前要获得 **当前 class 对象 的锁**

- 静态同步方法与普通同步方法之间无竞态条件

修饰代码块

- `synchronized(object)` 表示进入同步代码块前要获得 **给定对象的锁**。
- `synchronized(类.class)` 表示进入同步代码前要获得 **给定 Class 的锁**

> 构造方法不能用 syncronized 修饰吗？
> 
> - 构造方法本身就属于线程安全的，不存在同步的构造方法一说

### 底层原理

> 底层原理？ #字节_23_秋招_Java 
> 
> synchronized 上锁解锁流程 #字节_商业化_23_秋招_Java 
> 
> synchronized 是如何用字节码表达的？虚拟机是怎么支持它的？ #小红书_23_秋招_后端
> 
> synchronized 关联的 monitor 信息存储在哪？ #得物_实习_Java 

#### 锁得是什么？

每个对象都可以关联一个 Monitor，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象的指针

> 在 HotSpot 虚拟机中，monitor 采用 `ObjectMonitor` 实现，位于 HotSpot 虚拟机源码 `objectMonitor.hpp` 文件，C++实现的
> 
> `ObjectMonitor.java` --> `ObjectMonitor.cpp` --> `objectMonitor.hpp` 

ObjectMonitor 中的几个关键属性

![](assets/Pasted%20image%2020231127220305.png)

![](assets/Pasted%20image%2020240307135941.png)

> `wait/notify` 等方法也依赖于 `monitor` 对象，所以只有在同步块或者方法中才能调用 `wait/notify` 等方法，否则会抛出 `java.lang.IllegalMonitorStateException` 的异常

实现原理：
- Monitor 拥有一个锁计数器 count 和 一个指向持有该锁的线程的指针 owner
- 当执行 `monitorenter` 时，
	- count 为零，那么说明它没有被其他线程持有
		- owner 设置为当前线程，并将 count+1
	- count 不为零，
		- 如果 owner 是当前线程，那么 count+1，
		- 否则进入 EntryList，状态【BLOCKED】
- 当执行 `monitorexit` 时，count-1。count 为零代表锁已被释放
	- 唤醒 EntryList 中等待的线程来竞争锁，竞争是**非公平**的
- WaitSet 中是之前获得过锁，释放后在等待的线程

#### 字节码

```java
static final Object lock = new Object();
static int counter = 0;

public static void main(String[] args) {
	synchronized (lock) {
		counter++;
	}
}

Code:
	stack=2, locals=3, args_size=1
		0: getstatic #2 // <- lock引用 （synchronized开始）
		3: dup
		4: astore_1 // lock引用 -> slot 1
		5: monitorenter // 将 lock对象 MarkWord 置为 Monitor 指针
		6: getstatic #3 // <- i
		9: iconst_1 // 准备常数 1
		10: iadd // +1
		11: putstatic #3 // -> i
		14: aload_1 // <- lock引用
		15: monitorexit // 将 lock对象 MarkWord 重置, 唤醒 EntryList
		16: goto 24
		19: astore_2 // e -> slot 2 
		20: aload_1 // <- lock引用
		21: monitorexit // 将 lock对象 MarkWord 重置, 唤醒 EntryList
		22: aload_2 // <- slot 2 (e)
		23: athrow // throw e
		24: return
 Exception table:
	 from to target type
		 6 16 19 any
		 19 22 19 any
```

> 方法级别的 synchronized 不会在字节码指令中有所体现

1）同步代码块

`synchronized` 同步代码块对应的字节码指令的是 `monitorenter` 和 `monitorexit` 指令

- `monitorenter` 指向同步代码块的开始位置，`monitorexit` 指向结束位置
- 一般情况是 1 个 enter 对应 2 个 exit，一个正常退出、一个异常退出
- owner 线程才可以执行 `monitorexit` 指令来释放锁

2）同步方法

`synchronized` 修饰的方法并**没有** `monitorenter` 指令和 `monitorexit` 指令

- JVM 通过该 `ACC_SYNCHRONIZED` **访问标志**来辨别一个方法是否声明为同步方法（静态同步方法还有 `ACC STATIC` 标识）
- 如果设置了，执行线程会将先持有 monitor，然后再执行方法，最后在方法完成（无论是正常完成还是非正常完成）时释放 monitor

### synchronized 锁优化

> #todo 为什么 Synchronized 效率高？ #小红书_23_秋招_后端
> 
> #todo 锁优化机制 #PDD_23_秋招_后端 

### synchronized 和 volatile 有什么区别？

`synchronized` 关键字和 `volatile` 关键字是两个互补的存在

- 性能：`volatile` 是线程同步的轻量级实现，性能肯定比 `synchronized` 要好 。
- 作用对象：
	- `volatile` 只能用于变量
	- 而 `synchronized` 关键字可以修饰方法以及代码块 
- 线程安全：
	- `volatile` 关键字能保证数可见性、有序性，不能保证原子性
	- `synchronized` 关键字三者都能保证
- 适用场景：
	- `volatile` 关键字主要用于解决变量在多个线程之间的可见性
	- `synchronized` 关键字解决的是多个线程之间访问资源的同步性。

## ReentrantLock

### 可重入锁

> 可重入锁是什么 #得物_训练营_Java 

*可重入锁* 也叫递归锁，指的是**线程可以再次获取自己的内部锁**。

> 如果是不可重入锁的话，就会造成死锁

Lock 实现类（显式锁），包括 synchronized 关键字锁（隐式锁）都是可重入的

> synchronized 为什么设计为可重入锁？ #字节_商业化_23_秋招_Java 

调用完一个类的同步方法后还能再 调用这个类其他同步方法 或者 递归调用自身

### 公平 & 非公平锁

#PDD_23_秋招_后端 #小红书_23_秋招_后端

公平锁：是指多个线程按照**申请锁的顺序**来获取锁

- 优缺点：
	- 保证时间上的绝对顺序
	- 性能较差一些，**上下文切换更频繁**
- `Lock lock = new ReentrantLock(true)`：表示公平锁，先来先得

非公平锁：有可能后申请的线程比先申请的线程优先获取锁

- 优缺点：
	- 性能较好
	- **在高并发环境**下，有可能造成 优先级反转 或者 饥饿 的状态
- `Lock lock = new ReentrantLock(false)`：表示非公平锁，**默认为非公平锁**

> 为什么默认非公平？
> 
> - 非公平锁能**更充分的利用 CPU 的时间片，尽量减少 CPU 空闲状态时间**
> 
> 恢复挂起的线程到真正锁的获取还是有时间差的，从开发人员来看这个时间微乎其微，但是从 CPU 的角度来看，这个时间差存在的还是很明显的。
> 
> 使用多线程很重要的考量点是线程切换的开销，当采用非公平锁时，当 1 个线程请求锁获取同步状态，然后释放同步状态，所以**刚释放锁的线程在此刻再次获取同步状态的概率就变得非常大**，所以就**减少了线程的开销**。

### synchronized 和 ReentrantLock 区别？

> synchronized 和 ReentrantLock 区别？ #PDD_服务端研发 #小红书_23_秋招_后端
> 
> Sychronized 和 ReetrantLock 哪个性能好些？ #小红书_23_秋招_后端 

#todo 

共同点：

- 都是可重入锁

区别：

1）底层实现

- `synchronized` 是依赖于 **JVM** 实现的，它的优化都是在虚拟机层面实现的，并没有直接暴露给我们
- `ReentrantLock` 是 **JDK** 层面（AQS）实现的（也就是 API 层面，需要 lock() 和 unlock() 方法配合 try/finally 语句块来完成），所以我们可以通过查看它的源代码，来看它是如何实现的

2）ReentrantLock 比 synchronized 增加了一些高级功能

- *等待可中断* : 能够中断等待锁的线程，通过 `lock.lockInterruptibly()` 来实现
- *可实现公平锁* : ReentrantLock 可以指定是公平锁还是非公平锁，而 synchronized 只能是非公平锁
- *可实现选择性通知*（多条件变量）: 
	- `synchronized` 与 `wait()` 和 `notify()` / `notifyAll()` 方法相结合可以实现等待/通知机制，只能唤醒一个或全部唤醒
	- ReentrantLock 结合 Condition 实例可以实现“选择性通知”

### reentrantlock 原理

> lock 底层原理(2) #得物_训练营_Java 
> 
> 可重入的底层是如何实现的？ #得物_训练营_Java 
> 
> reentrantlock 原理 #得物_训练营_Java 

#todo

## 死锁

> 什么是死锁？ #小红书_23_秋招_后端 #字节_商业化_24_Java 

**多个线程**同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。若无外力干涉那它们都将无法推进下去。

产生死锁主要原因：

- 系统资源不足
- 进程运行推进的顺序不合适
- 资源分配不当

### 代码案例

> 代码模拟死锁，要百分之百会出现死锁，而不是偶现 #小红书_23_秋招_后端

![300](assets/Pasted%20image%2020240205160141.png)

> 来源于《并发编程之美》

```java
public class DeadLockDemo {
    private static Object resource1 = new Object();//资源 1
    private static Object resource2 = new Object();//资源 2

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resource1) {
                sleep(1000);
                
                synchronized (resource2) {
                    
                }
            }
        }, "线程 1").start();

        new Thread(() -> {
            synchronized (resource2) {
                sleep(1000);
                
                synchronized (resource1) {
                    
                }
            }
        }, "线程 2").start();
    }
}
```

### 必要条件

> 死锁的必要（产生）条件 #顺丰_22_秋招_Java #字节_商业化_24_Java  #字节_国际电商_23_秋招_Java 构造一个场景 #字节_国际电商_23_秋招_Java 

上面的例子符合产生死锁的四个必要条件：

1. *【互斥】条件* ：该资源任意一个时刻只由**一个线程**占用。
2. *【请求与保持】条件* ：一个线程因请求资源而阻塞时，对已获得的资源**保持不放**。
3. *【不剥夺】条件* ：线程已获得的资源在未使用完之前**不能被其他线程强行剥夺**，只有自己使用完毕后才释放资源
4. *【循环等待】条件* ：若干线程之间形成一种**头尾相接**的循环等待资源关系 #todo 

### 如何预防

> √ 如何预防（避免）死锁的发生？(2) #顺丰_22_秋招_Java  #字节_国际电商_23_秋招_Java 

如何“预防”线程死锁？破坏死锁的产生的必要条件即可：

1. *破坏请求与保持条件*：一次性申请**所有**的资源
2. *破坏不剥夺条件*：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以**主动释放**它占有的资源
3. *破坏循环等待条件*：靠**按序申请资源**来预防。按某一顺序申请资源，释放资源则反序释放（AB，BA）

### 排查死锁

> √ 发生死锁怎么排查？(2)
>
> 怎么解决死锁？ #字节_国际电商_23_秋招_Java 

1）命令方式

- `jps -l` 查出进程号
- `jstack <进程号>` 打印进程的栈信息

2）图形化界面：jconsole工具

### 如何避免

注意加锁顺序！

在资源分配时，借助于算法（比如银行家算法）对资源分配进行计算评估，使其进入安全状态。

> *安全状态* 指的是系统能够按照某种线程推进顺序（P1、P2、P3……Pn）来为每个线程分配所需资源，直到满足每个线程对资源的最大需求，使每个线程都可顺利完成。称 `<P1、P2、P3.....Pn>` 序列为安全序列。

我们对线程 2 的代码修改成下面这样就不会产生死锁了。

```java
new Thread(() -> {
	synchronized (resource1) {
		System.out.println(Thread.currentThread() + "get resource1");
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread() + "waiting get resource2");
		synchronized (resource2) {
			System.out.println(Thread.currentThread() + "get resource2");
		}
	}
}, "线程 2").start();
```

> 线程 1 首先获得到 resource1 的监视器锁,这时候线程 2 就获取不到了。然后线程 1 再去获取 resource2 的监视器锁，可以获取到。然后线程 1 释放了对 resource1、resource2 的监视器锁的占用，线程 2 获取到就可以执行了。这样就破坏了破坏循环等待条件，因此避免了死锁。

## java 里有哪几种上锁方式，了解他们的区别吗？

#pdd_暑期实习_后端 

#todo 
  
## Future 获得结果怎么处理？

#B站SRE_Java后端 

#todo 

# ---------- 内存

## 介绍一下 java 内存模型 JMM

#得物_实习_Java #顺丰_23_秋招_Java #字节_24_实习_Java 

### 是什么 & 能干嘛？

JMM 即 *Java Memory Model*，它定义了**主存、工作内存抽象概念**，底层对应着 CPU 寄存器、缓存、硬件内存、CPU 指令优化等。

JMM 体现在以下几个方面

- 原子性 - 保证指令不会受到**线程上下文**切换的影响
- 可见性 - 保证指令不会受 **cpu 缓存**的影响
- 有序性 - 保证指令不会受 **cpu 指令并行优化**的影响

为什么需要 JMM？/ JMM 能干嘛？

- Java 语言是跨平台的，它需要自己提供一套内存模型，来**屏蔽**掉各种硬件和操作系统的内存访问差异
- 并发编程下，像 CPU 多级缓存和指令重排这类设计可能会导致程序运行出现一些问题。为此，JMM 抽象了 happens-before 原则来解决这个指令重排序问题

JMM 说白了，**定义了一些规范来解决这些问题**，开发者可以利用这些规范更方便地开发多线程程序。对于 Java 开发者说，你不需要了解底层原理，直接使用并发相关的一些关键字和类（比如 `volatile`、`synchronized`、各种 `Lock`）即可开发出并发安全的程序

**简化多线程编程，增强程序可移植性的**

> 一般来说，编程语言也可以直接复用操作系统层面的内存模型。不过，不同的操作系统内存模型不同。如果直接复用操作系统层面的内存模型，就可能会导致同样一套代码换了一个操作系统就无法执行了。

### JMM 是如何抽象线程和主内存之间的关系？

> 共享变量的副本是存储在 jvm 哪里的？ #得物_实习_Java

- 所有的共享变量都存储在主内存（Main Memory）中
- 每条线程还有自己的工作内存，
    - 保存了被该线程使用的**变量的主内存副本**，
    - 线程对变量的所有操作（读取、赋值等）都**必须在工作内存中进行**，而不能直接读写主内存中的数据
- 不同的线程之间也无法直接访问对方工作内存中的变量，**线程间变量值的传递均需要通过主内存**来完成

### Java 内存区域和 JMM 有何区别？

> 这是一个比较常见的问题，很多初学者非常容易搞混

- JVM 内存结构和 Java 虚拟机的运行时区域相关，定义了 JVM 在运行时如何**分区存储程序数据**，就比如说堆主要用于存放对象实例
- Java 内存模型和 Java 的并发编程相关，**简化多线程编程，增强程序可移植性的**

### happens-before

> 区分可见和可见性

happens-before 规定了对共享变量的写操作对其它线程的读操作可见，它是**可见性与有序性**的一套规则总结

- 线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见
- 线程对 volatile 变量的写，对接下来其它线程对该变量的读可见
- 线程 start 前对变量的写，对该线程开始后对该变量的读可见
- 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束）
- 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过 t2.interrupted 或 t2.isInterrupted）
- 对变量默认值（0，false，null）的写，对其它线程对该变量的读可见
- 具有传递性，如果 x hb-> y 并且 y hb-> z 那么有 x hb-> z ，配合 volatile 的防指令重排，有下面的例子

### 三大特性

> #todo 从计算机的操作系统的角度讲一下为什么会出现数据可见性

可见性：当一个线程修改了某一个共享变量的值，其他线程是否能够知道立即该变更

- volatile 保证
- synchronized 的可见性是由“对一个变量执行 **unlock 操作之前**，必须先把此变量同步回主内存中（执行 store、write 操作）”这条规则获得的

原子性：指一个操作是不可打断的，即**多线程**环境下，操作不能被其他线程干扰

- synchronized 块之间的操作具备原子性（时间片用完了还是会上下文切换）
- CAS（原子操作）

> 同步代码块内不能有指令交错

有序性：JVM 会在**不影响正确性**的前提下，可以调整语句的执行顺序

- volatile 修饰的变量，可以禁用指令重排
- synchronized #todo 写屏障

> 保证有序性就是保证不影响正确性

#todo final 变量

## 说一下 volatile 关键字

#B站Java后端23_秋招 #得物_实习_Java

> volatile可以保证线程安全吗？ #B站Java后端23_秋招 

volatile（易变关键字）：它可以用来修饰**成员变量**和**静态成员变量**，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存

volatile 保证可见性和有序性，不保证原子性

volatile 的底层实现原理是【内存屏障】（Memory Barrier/Memory Fence）

- 对 volatile 变量的写指令**后**会加入写屏障
- 对 volatile 变量的读指令**前**会加入读屏障

### 如何保证变量的可见性？

每次使用它都到主存中进行读取

> #JavaGuide 
> 
> `volatile` 关键字其实并非是 Java 语言特有的，在 C 语言里也有，它最原始的意义就是**禁用 CPU 缓存**。如果我们将一个变量使用 `volatile` 修饰，这就指示 编译器，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。

- 写屏障（sfence）保证在该屏障**之前**的，对共享变量的改动，都同步到主存当中
- 读屏障（lfence）保证在该屏障**之后**，对共享变量的读取，加载的是主存中最新数据

### 如何禁止指令重排序

仅针对单个线程内

- 写屏障会确保指令重排序时，不会将写屏障**之前**的代码排在写屏障之后
- 读屏障会确保指令重排序时，不会将读屏障**之后**的代码排在读屏障之前

### volatile 可以保证原子性吗？

不能，多线程环境下还是会发生指令交错

多线程 `inc++` 案例

### 应用场景

- 单一赋值可以，但是含复合运算赋值不可以（i++之类的）
	- 状态标志，判断业务是否结束
- 当读远多于写，结合使用内部锁和 volatile 变量来减少同步的开销
- DCL 单例模式

### DCL 单例模式

> double check实现单例模式 #B站

DCL 的特点

- 懒惰实例化
- 创建完了就不需要加锁了，synchronized 外面会有对 `INSTANCE == null` 的判断
- INSTANCE 用 **volatile** 修饰保证有序性
	- 禁止 **初始化对象** 和 **对象引用赋值给 INSTANCE** 之间的指令

# ---------- 无锁

## 乐观锁 和 悲观锁

> 乐观锁与悲观锁 #顺丰_22_秋招_Java  #字节_飞书_24_实习_Java 
> 
> 两个用户修改数据，怎么防止另一个用户负载上一个用户的修改？ #PDD_23_秋招_基础电商_后端 
> 
> 乐观锁实现方式（版本号、CAS） #顺丰_23_秋招_Java 

悲观锁：每次在获取资源操作的时候都会上锁，共享资源每次只给一个线程使用，其它线程**阻塞**

- 实现：synchronized 和 Lock 实现类
- 适用场景：“**多写，竞争激烈**”，悲观锁的开销是固定的
- 缺点：
	- 高并发的场景下，激烈的锁竞争会造成大量线程阻塞，导致频繁系统的**上下文切换**，增加系统的性能开销
	- 存在死锁

乐观锁：不添加锁，只是在提交修改的时候去验证对应的资源是否被其它线程修改了

- 实现：版本号机制、CAS 算法（直接把共享变量当做 verison）
- 适用场景：
	- “**多读，竞争较少**”
	- 乐观锁主针对**单个共享变量**（参考`java.util.concurrent.atomic`包下面的原子变量类）
- 缺点：
	- 写占比非常多的情况，会频繁失败和重试，这样同样会非常影响性能，导致 CPU 飙升。（像 `LongAdder` 以空间换时间的方式就解决了这个问题 #todo ）

## CAS

> #todo 操作系统层面，CAS 操作是怎么做的？ #B站cdn_服务端开发

全称是 *Compare And Swap*（比较与交换） ，用于实现乐观锁。思想就是用一个**预期值和要更新的变量值进行比较**，两值相等才会进行更新

CAS 涉及到三个操作数：

- **V**：要更新的变量值(Var)
- **E**：预期值(Expected)
- **N**：拟写入的新值(New)

> 当且仅当 V 的值等于 E 时，CAS 通过原子方式用新值 N 来更新 V 的值。如果不等，说明已经有其它线程更新了 V，则当前线程放弃更新。

CAS 是一个原子操作，底层依赖一条原子指令 `lock cmpxchg` 指令（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的原子性。

> 在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的

CAS 必须**借助 volatile** 才能读取到共享变量的最新值来实现【比较并交换】的效果

### 优缺点 & 适用场景

> CAS 的优缺点是什么？ #PDD_服务端研发 #PDD_23_秋招_基础电商_后端
>
> CAS 和普通的锁的区别，什么时候用 cas 什么时候用到锁 #PDD_服务端研发

优点：

- 无锁、无阻塞并发（操作失败也不会上下文切换）

缺点：

- ABA
- *循环时间长开销大*：CAS 经常会用到自旋操作来进行重试，也就是不成功就一直循环执行直到成功。如果长时间不成功，会给 CPU 带来非常大的执行开销
- *只能保证一个共享变量的原子操作*：当操作涉及跨多个共享变量时 CAS 无效
	- AtomicReference 类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作

适用场景：

- 线程数少（竞争较少）
- 多读（自旋时间短）

### ABA

> ABA如何解决？ #PDD_23_秋招_后端  #顺丰_22_秋招_Java 

在共享变量之外再追加一个**版本号**，不仅能解决，还能知道引用变量中途被修改了多少次

`AtomicStampedReference` 类就是用来解决 ABA 问题的，

- 类中维护了一个静态内部类 Pair，里面存了对象引用和版本号
- 其中 `compareAndSet()` 方法就首先检查**当前引用是否等于预期引用**，并且**当前标志是否等于预期标志**，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

### 原理

#todo

> 原理？ #PDD_23_秋招_后端
> 
> CAS底层怎么实现？ #字节_飞书_24_实习_Java

Java 语言并没有直接实现 CAS，CAS 相关的实现是通过 C++ 内联汇编的形式实现的（JNI 调用）。因此， CAS 的具体实现和操作系统以及 CPU 都有关系。

`sun.misc` 包下的 **Unsafe** 类提供了 `compareAndSwapObject`、`compareAndSwapInt`、`compareAndSwapLong` 方法来实现的对 `Object`、`int`、`long` 类型的 CAS 操作

```java
/**
  *  CAS
  * @param o         包含要修改field的对象
  * @param offset    对象中某field的偏移量
  * @param expected  期望值
  * @param update    更新值
  * @return          true | false
  */
public final native boolean compareAndSwapObject(Object o, long offset,  Object expected, Object update);

public final native boolean compareAndSwapInt(Object o, long offset, int expected,int update);

public final native boolean compareAndSwapLong(Object o, long offset, long expected, long update);

```

# ---------- 线程池

线程池就是管理一系列线程的资源池。当有任务要处理时，直接从线程池中获取线程来处理，处理完之后线程并不会立即被销毁，而是等待下一个任务。

使用线程池的好处：

- *降低资源消耗*：通过**重复利用**已创建的线程 降低线程创建和销毁造成的消耗
- *提高响应速度*：当任务到达时，任务可以**不需要等待线程创建**就能立即执行
- *提高线程的可管理性*：线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行**统一的分配，调优和监控**

## 线程池参数

### 线程池有哪些参数？

#PDD_23_秋招_基础电商_后端 #PDD_23_秋招_后端 #字节_国际电商_23_秋招_Java
 
> 线程池的常见配置？ #B站Java后端23_秋招 

- *corePoolSize*：任务队列**未达到队列容量**时，最大可以同时运行的线程数量
- *maximumPoolSize*：任务队列中存放的任务**达到队列容量**的时候，当前可以同时运行的线程数量变为最大线程数
- *workQueue*：阻塞队列，当前运行的线程数量达到 corePoolSize，新任务会放入阻塞队列

- *keepAliveTime*：救急线程的生存时间
- *unit*：keepAliveTime 的时间单位
- *threadFactory*：线程工厂 - 可以为线程创建时起个好名字
- *handler*：拒绝（饱和）策略

### 拒绝策略

> 拒绝策略 #顺丰_22_秋招_Java 

ThreadPoolExecutor 定义一些策略，静态内部类，都实现了 RejectedExecutionHandler:

```java
public interface RejectedExecutionHandler {  
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);  
}
```

- *AbortPolicy*（默认）： 抛出 `RejectedExecutionException` 来拒绝新任务的处理
- *CallerRunsPolicy*： 让调用者线程运行任务
- *DiscardPolicy*： 不处理新任务，直接丢弃掉
- *DiscardOldestPolicy*： 放弃队列中最早的任务，本任务取而代之

### 线程池常用的阻塞队列


### corePoolSize 怎么确定

> 怎么 保证/确定（核心）线程数的？ #PDD_23_秋招_后端
> 
> core 线程数目一般如何确定：计算密集性、IO 密集性 #滴滴

设置的 corePoolSize 

- 过小：
	- 程序不能充分地利用系统资源
	- 大量任务都堆积在任务队列里，可能会队满、OOM 的情况

- 过大：大量线程可能会同时在争取 CPU 资源，这样会导致**大量的上下文切换**，从而增加线程的执行时间，影响了整体执行效率

1）CPU 密集型运算

`cpu 核数 + 1` 能够实现最优的 CPU 利用率

- +1 是保证当线程由于页缺失故障（操作系统）或其它原因导致暂停时，额外的这个线程就能顶上去，充分利用 CPU 的空闲时间

2） I/O 密集型运算

CPU 不总是处于繁忙状态（几乎全是线程等待时间）

> 例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。

公式：`线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间`

或者直接 `cpu 核数 * 2`

> 例如 4 核 CPU 计算时间是 50% ，其它等待时间是 50%，期望 cpu 被 100% 利用，套用公式 `4 * 100% * 100% / 50% = 8`
> 
> 例如 4 核 CPU 计算时间是 10% ，其它等待时间是 90%，期望 cpu 被 100% 利用，套用公式 `4 * 100% * 100% / 10% = 40`

### 如何给线程池命名？

实现 ThreadFactory

```java
public final class NamingThreadFactory implements ThreadFactory {

    private final AtomicInteger threadNum = new AtomicInteger();
    private final String name;

    /**
     * 创建一个带名字的线程池生产工厂
     */
    public NamingThreadFactory(String name) {
        this.name = name;
    }

    @Override
    public Thread newThread(Runnable r) {
        Thread t = new Thread(r);
        t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
        return t;
    }
}
```

## 线程池工作流程？

#得物_训练营_Java #字节_国际电商_23_秋招_Java

> 线程池设置 2 个核心线程数，4 个最大线程数，什么时候用核心线程数，什么时候用最大线程数？(1)
> 
> 线程池没有新任务进来，线程池会如何变化？ #得物_训练营_Java 
> 
> 核心线程数不够会怎么处理？(1)
> 
> 线程池新增任务的执行流程是怎么样的(1)

- 当 运行的线程数 `<` corePoolSize，会新建一个线程来执行任务
- 当 运行的线程数达到 corePoolSize，新任务放入 workQueue 排队
- 当 任务队列满了，最多会创建 maximumPoolSize - corePoolSize 数目的线程救急
- 当 运行的线程数达到 maximumPoolSize 仍然有新任务，会执行拒绝策略
- 当高峰过去后，超过 corePoolSize 的救急线程如果一段时间（keepAliveTime 和 unit 来控制）没有任务做，需要结束节省资源

> 假如现在有 15 个任务 5 个核心线程 最大线程是 10 工作队列是 5，请问执行顺序是怎样的？ #B站Java后端23_秋招 

假如都执行不玩，5 核心 5 队列 5 救急

## 线程池创建方法

#得物_训练营_Java

> fixed、single、cache、schedule 线程池原理(1)
> 
> 线程池原理(1)

1）通过 `ThreadPoolExecutor` 构造函数来创建

2）通过 Executor 框架的工具类 Executors 来创建

*FixedThreadPool*：固定线程数量的线程池

- 创建：`newFixedThreadPool(int nThreads, ThreadFactory threadFactory)`
- 特点：
	- 核心线程数 `==` 最大线程数
	- 阻塞队列是无界的 LinkedBlockingQueue
- 适用场景：适用于任务量已知，相对耗时的任务

*SingleThreadExecutor*：只有一个线程的线程池

- 创建：`newSingleThreadExecutor(ThreadFactory threadFactory)`
- 特点：
	- 核心线程数 `==` 最大线程数 `==` 1，且线程数不能修改
	- 阻塞队列是无界的 LinkedBlockingQueue
- 适用场景：希望多个任务**排队**执行

*CachedThreadPool*：可根据实际情况调整线程数量的线程池

- 创建：`newCachedThreadPool(ThreadFactory threadFactory)
- 特点：
	- 核心线程数为 0，最大线程数是 `Integer.MAX_VALUE`（全都是救急线程）
	- 救急线程的空闲生存时间是 60s
	- #todo 队列采用了 SynchronousQueue，没有容量，没有线程来取是放不进去的
- 适用场景：任务数比较**密集**，但每个任务**执行时间较短**的情况

*ScheduledThreadPool*：在给定的**延迟**后运行任务或者**定期**执行任务的线程池 

- #todo 源码和教程有争议

## 使用场景

> 什么时候用的线程池？ #得物_训练营_Java 
>
> 怎么用线程池，应用场景(1)
> 
> 线程池在项目中是做什么的？(1)
> 
> 你平常用哪个线程池，和别的有什么区别？ #字节_国际电商_23_秋招_Java

#todo 

## 线程池是怎么实现线程的保活和停止管理的？(1)

> 答：不是很理解面试官的意思，沉默了几分钟说了一下 shutdown()、shutdownNow()以及 awaitTermination()三种方法的使用，面试官没有反驳，点了点头，估计瞎猫碰上死耗子了

#todo 啥是保活？

### 停止管理

#todo 

调用 `shutdown()`，线程池状态变为 SHUTDOWN，不会接收新任务

- 但**已提交任务会执行完**
- 此方法**不会阻塞**调用线程的执行

调用 `shutdownNow()`，线程池状态变为 STOP，不会接收新任务

- 会将队列中的任务返回 `List<Runnable>`
- 并用 interrupt 的方式**中断正在执行的任务**

`awaitTermination()`：由于调用线程并**不会等待所有任务运行结束**，因此如果它想在线程池 TERMINATED 后做些事情，可以利用此方法等待

```java
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
```

## 自定义线程池

> 线程池是使用的自己实现的线程池吗？(1)
> 
> 自定义线程池需要重写什么(1)
> 
> 如果针对业务场景设计线程池参数应该根据哪些？(1)

#todo
## 在线程池中多个线程的结果是如何去合并的，说出两种解决方式(1)


## 线程池中提交任务想要得到运行结果怎么办，其原理是什么(1)




# ---------- 工具

## 说一下 ThreadLocal

#B站Java后端23_秋招 #B站后端日常实习 

> 其他问法：
> 
> - ThreadLocal 是什么，怎么使用的？ #PDD_服务端研发
> - 可能会有哪些问题？ #PDD_服务端研发 #PDD_23_秋招_基础电商_后端 
> - 怎么避免内存泄漏？ #PDD_服务端研发
> - 底层怎么优化的？ #PDD_服务端研发
> - key是弱引用，可以为null，那么value呢？ #PDD_服务端研发
> - 用 ThreadLocal 有什么好处？ #PDD_服务端研发
> - 主线程的 ThreadLocal 如何向子线程的 ThreadLocal 传递数据？(1)
> - 如何实现多个线程修改同一个变量，互不影响？ #PDD_23_秋招_基础电商_后端 

## AQS

> aqs原理(2)，可以举一个具体的实现来说？ #携程

#PDD_23_秋招_后端 

*AbstractQueuedSynchronizer*，翻译 抽象队列同步器，为 阻塞式锁 和 相关的同步器工具 提供了一些通用功能的实现

AQS 核心思想是，

- 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
- 如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，即 **CLH 锁** （Craig, Landin, and Hagersten locks） 实现的。

CLH 锁是对自旋锁的一种改进，是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系），暂时获取不到锁的线程将被加入到该队列中。AQS 将每条请求共享资源的线程封装成一个 CLH 队列锁的一个结点（Node）来实现锁的分配。在 CLH 队列锁中，一个节点表示一个线程，它保存着线程的引用（thread）、 当前节点在队列中的状态（waitStatus）、前驱节点（prev）、后继节点（next）


## JUC 工具类用过哪些？

#B站SRE_Java后端  #字节_飞书_24_实习_Java 


## CountDownlaunch 使用场景和原理

#得物_训练营_Java 

## 信号量

> 信号量可以用来当锁用吗 #滴滴
> 
> 信号量和 CAS 的区别，这两个的设计思想分别对应的是乐观锁还是悲观锁
> 
> 信号量当锁是能防止指令重排问题吗，能够保证数据的可见性吗

# ---------- 线程安全

## java 中线程安全的类有哪些？(1) 

> map、set、list中哪些是线程安全的？(1)
> 
> HashTable为什么线程安全 #小红书_实习_后端

## List、Map 要使用线程安全要怎么办？

#得物_后端


## ConcurrentHashMap 原理(1)

#PDD_服务端研发

> ConcurrentHashMap1.7 和 1.8线程安全怎么做的？ #小红书_实习_后端
> 
> HashMap 和 ConcurrentHashMap 的对比和区别 #顺丰_23_秋招_Java 
> 
> ConcurrentHashMap 是通过什么手段保证（怎么实现）线程安全的？ #顺丰_23_秋招_Java  #字节_飞书_24_实习_Java #字节_商业化_23_秋招_Java 
> 
> 底层结构？ #字节_飞书_24_实习_Java 
> 
> ConcurrentHashMap 和 HashTable 有什么区别？ #得物_后端




# ---------- 多线程业务和场景

## 平时怎么用多线程？遇到的多线程问题怎么解决？(1)

## 3 个线程交替打印 1-100 伪代码，则么让 main 线程等待这三个线程执行完再执行？(1)



# ---------- 线程通信
## 线程通信的方式有哪些？

#字节_24_实习_Java 

> 大概的原理？(2)


# ---------- 未知

## 真正使用时，Java 里的线程和进程是如何调度？(1)

## java怎么处理多线程并发的问题？

#PDD_暑期实习_后端 


## 多线程的底层实现原理(1)


## 进程有哪些状态？

> 有一个问过，忘了哪个了

## 多线程事务失效(1)

## Spring多线程支持(1)

## 救急线程是什么时候过期的？

## 协程和线程区别，对应关系

#pdd_暑期实习_后端