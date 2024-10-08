
# 基础


## 用户态和内核态

CPU 的运行模式被分为了用户态和内核态

- **用户态**：应用程序运行在用户态，应用程序执行非特权指令
	- 非特权指令：仅限于访问用户的地址空间
- **核心态**：操作系统内核程序运行在核心态，内核程序要执行一些特权指令
	- 特权指令，是指不允许用户直接使用的指令，如 I0 指令、置中断指令，存取用于内存保护的寄存器、送程序状态字到程序状态字寄存器等的指令。

内核态相比用户态拥有更高的特权级别，因此能够执行更底层、更敏感的操作。不过，由于==进入内核态需要付出较高的开销（需要进行一系列的上下文切换和权限检查）==，应该尽量减少进入内核态的次数，以提高系统的性能和稳定性。


> JavaGuide：
> - **用户态(User Mode)** : 用户态运行的进程可以直接读取用户程序的数据，拥有较低的权限。当应用程序需要执行某些需要特殊权限的操作，例如读写磁盘、网络通信等，就需要向操作系统发起系统调用请求，进入内核态。
> - **内核态(Kernel Mode)**：内核态运行的进程几乎可以访问计算机的任何资源包括系统的内存空间、设备、驱动程序等，不受限制，拥有非常高的权限。当操作系统接收到进程的系统调用请求时，就会从用户态切换到内核态，执行相应的系统调用，并将结果返回给进程，最后再从内核态切换回用户态。


## 系统调用

所谓系统调用，是指==用户在程序中调用操作系统所提供的一些子功能==，系统调用可视为特殊
的公共子程序。

用户程序可以执行==陷入指令(又称访管指令或 trap 指令)==来发起系统调用

> #Bo 用户态进程==主动==要求切换到内核态的一种方式，核心是利用 内中断/异常

![](assets/Pasted%20image%2020240913102142.png)

这些系统调用按功能大致可分为如下几类：

- 设备管理：完成设备（输入输出设备和外部存储设备等）的请求或释放，以及设备启动等功能。
- 文件管理：完成文件的读、写、创建及删除等功能。
- 进程管理：进程的创建、撤销、阻塞、唤醒，进程间的通信等功能。
- 内存管理：完成内存的分配、回收以及获取作业占用内存区大小及地址等功能。

系统调用和普通库函数调用非常相似，只是系统调用由操作系统内核提供，运行于内核态，而普通的库函数调用由函数库或用户自己提供，运行于用户态。

总结：系统调用是应用程序与操作系统之间进行交互的一种方式，通过系统调用，应用程序可以访问操作系统底层资源例如文件、设备、网络等。


# 进程和线程

## 进程和线程的区别

- 并发性：不光是进程间，线程间可以并发执行
- 并行性（支持多处理机系统）：进程只能运行在一个处理及上
- 开销：
	- 创建开销
	- 切换开销
	- 通信开销：线程通信不需要操作系统干预
- 独立性：每个进程有独立的地址空间和资源，同一进程的线程共享进程的地址空间和资源

## 进程/线程状态

- 创建
- 运行
- 就绪
- 阻塞
- 终止

## 进程通信

- 共享存储：共享内存，需要通过系统调用访问
- 消息传递：消息发送到接收进程的消息缓冲队列，需要通过发送消息和接受消息的系统原语
- 管道通信：管道是一种文件

## 线程同步方式

线程同步是两个或多个共享关键资源的线程的并发执行。应该同步线程以避免关键的资源使用冲突。

下面是几种常见的线程同步的方式：

1. **互斥锁**(Mutex)：采用互斥对象机制，只有拥有互斥对象的线程才有访问公共资源的权限。因为互斥对象只有一个，所以可以保证公共资源不会被多个线程同时访问。比如 Java 中的 `synchronized` 关键词和各种 `Lock` 都是这种机制。
2. 读写锁（Read-Write Lock）：允许多个线程同时读取共享资源，但只有一个线程可以对共享资源进行写操作。
3. **信号量**(Semaphore)：它允许同一时刻多个线程访问同一资源，但是需要控制同一时刻访问此资源的最大线程数量。
4. 屏障（Barrier）：屏障是一种同步原语，用于等待多个线程到达某个点再一起继续执行。当一个线程到达屏障时，它会停止执行并等待其他线程到达屏障，直到所有线程都到达屏障后，它们才会一起继续执行。比如 Java 中的 `CyclicBarrier` 是这种机制。
5. 事件(Event) :Wait/Notify：通过通知操作的方式来保持多线程同步，还可以方便的实现多线程优先级的比较操作。

管程

## 协程

[什么是协程？ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/172471249)

[✅进程，线程和协程的区别 (yuque.com)](https://www.yuque.com/hollis666/krcpbs/gnieul)

所谓的协程，是英语翻译过来的（Coroutine），也叫纤程。通过 Coroutine 来理解，是**协作的程序**。

它其实是不能和进程，线程相提并论的，因为==协程是用户态的东西，不会被 OS 感知到的==。但是因为 GoLang 的大火，大家总是把他们放到一起来比较，所以在这里也一起讲了。

对于多次 IO 操作来说，我们可以用多线程来完成，但是多线程会==引起资源竞争，导致 CPU 算力的浪费==。为了避免这种情况，我们也可以用异步的方式，但是异步的方式会引起 callback hell，严重影响了代码的可读性。

所以就需要协程，==不让 OS 通过竞争的方式决定调用哪些线程==，而是==由用户自己决定如何去执行逻辑==。它可以让我们用逻辑流的顺序去写控制流，而且还不会导致操作系统级的线程阻塞。因为协程是由应用程序决定的，所以任务切换的上下文也会交给了用户态来保存。

目前 GO 和 Ruby 等编程语言都实现了协程，Java 也在 19 的时候引入了协程

> 将多个协程映射到少量操作系统线程中，通过有效的调度来避免那些上下文切换。



# 死锁

## 什么是死锁

> 什么是死锁？ #小红书_23_秋招_后端 #字节_商业化_24_Java 

**多个线程**同时被阻塞，它们中的一个或者全部都在等待某个资源被释放。由于线程被无限期地阻塞，因此程序不可能正常终止。若无外力干涉那它们都将无法推进下去。


## 代码案例

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


## 必要条件

> 死锁的必要（产生）条件 #顺丰_22_秋招_Java #字节_商业化_24_Java  #字节_国际电商_23_秋招_Java 构造一个场景 #字节_国际电商_23_秋招_Java 

上面的例子符合产生死锁的四个必要条件：

1. *【互斥】条件* ：该资源任意一个时刻只由**一个线程**占用。
2. *【请求与保持】条件* ：一个线程因请求资源而阻塞时，对已获得的资源**保持不放**。
3. *【不剥夺】条件* ：线程已获得的资源在未使用完之前**不能被其他线程强行剥夺**，只有自己使用完毕后才释放资源
4. *【循环等待】条件* ：若干线程之间形成一种**头尾相接**的循环等待资源关系 #todo 


## 如何预防

> √ 如何预防（避免）死锁的发生？(2) #顺丰_22_秋招_Java  #字节_国际电商_23_秋招_Java 

如何“预防”线程死锁？破坏死锁的产生的必要条件即可：

1. *破坏请求与保持条件*：一次性申请**所有**的资源
2. *破坏不剥夺条件*：占用部分资源的线程进一步申请其他资源时，如果申请不到，可以**主动释放**它占有的资源
3. *破坏循环等待条件*：靠**按序申请资源**来预防。按某一顺序申请资源，释放资源则反序释放（AB，BA）


## 排查死锁

> √ 发生死锁怎么排查？(2)
>
> 怎么解决死锁？ #字节_国际电商_23_秋招_Java 

1）命令方式

- `jps -l` 查出进程号
- `jstack <进程号>` 打印进程的栈信息

2）图形化界面：jconsole工具


## 如何避免

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



