
> 参考资料：尚硅谷2022版JUC并发编程（对标阿里P6-P7）
> 
> 讲师：周阳
> 
> [https://www.bilibili.com/video/BV1ar4y1x727?p=4&share_source=copy_web&vd_source=eb6ade7f410b79ba275b327b6011d56e](https://www.bilibili.com/video/BV1ar4y1x727?p=4&share_source=copy_web&vd_source=eb6ade7f410b79ba275b327b6011d56e)

# 线程基础知识复习

### 用户线程or守护线程

**用户线程：** 是系统的工作线程，它会完成这个程序需要完成的业务操作，一般不做特别说明配置，**默认都是用户线程**

**守护线程：** 是一种特殊的线程，为其他线程服务的，在后台默默地完成一些系统性的服务，比如垃圾回收线程。
- 守护线程作为一个服务线程，没有服务对象就没有必要继续运行了，如果用户线程全部结束了，意味着程序需要完成的业务操作已经结束了，系统可退出了。假如当系统只剩下守护线程的时候，==java虚拟机会自动退出==。

线程的daemon属性：
- true 表示是守护线程
- false 表示是用户线程

判断用户线程or守护线程
```java
// public class Thread
public final boolean isDaemon() {  
    return daemon;  
}
```

设置用户线程：
```java
public final void setDaemon(boolean on) {  
    checkAccess();  
    if (isAlive()) {  
        throw new IllegalThreadStateException();  
    }  
    daemon = on;  
}
```
- 注意：**`setDaemon(true)` 方法必须在`start()`之前设置**，否则报`IIIegalThreadStateException`异常。即不能把正在运行的常规线程设置为守护线程

演示用户/守护线程区别：
- main方法执行完了，主线程结束后，用户线程t1还在运行
- 而用户线程main方法结束后，守护线程也结束了
```java
/**
 * 演示用户/守护线程
 */
public class Demo {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t 开始执行，" +
                    (Thread.currentThread().isDaemon() ? "守护线程" : "用户线程"));
            while (true) {
            }
        }, "t1");

        // 设置守护线程
        t1.setDaemon(true);
        t1.start();

        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName() + "\t ---end 主线程");
    }
}
```

# 说说 Java "锁"事

## 乐观锁 和 悲观锁

- 悲观锁： 
	- 认为自己在使用数据的时候==一定有别的线程来修改数据==，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改，**synchronized**和**Lock**的实现类都是悲观锁
	- 特点：
		- 适合写操作多的场景，先加锁可以保证写操作时数据正确，显示的锁定之后再操作同步资源
		- 狼性锁
- 乐观锁： 
	- 认为自己在使用数据的时候==不会有别的线程修改数据或资源==，不会添加锁
	- Java 中使用无锁编程来实现，只是在更新的时候去判断，之前有没有别的线程更新了这个数据
		- 如果这个数据没有被更新，当前线程将自己修改的数据成功写入
		- 如果已经被其他线程更新，则根据不同的实现方式执行不同的操作，比如：放弃修改、重试抢锁等等。
	- 实现方式：
		- 版本号机制 **Version**
		- 最常采用的是 **CAS** (Compare-and-Swap，比较并替换)算法
			- Java 原子类中的递增操作就通过 CAS 自旋实现的。
	- 特点
		- 适合==读操作多==的场景，不加锁的特性能够使其读操作的性能大幅提升
		- 乐观锁则直接去操作同步资源，是一种无锁算法，得之我幸不得我命，再努力就是
		- 佛系锁

## Syncronized

### 8锁案例，锁到底是什么？

- 标准访问 a 和 b 两个线程
	- 先邮件，后短信  共用一个对象锁
- sendEmail 中加入暂停 3 秒钟
	- 先邮件，后短信  共用一个对象锁
- 添加一个普通的 hello 方法
	- 先 hello，再邮件
- 有两部手机
	- 先短信后邮件  资源没有争抢，不是同一个对象锁
- 有两个静态同步方法，一步手机
	- 先邮件后短信  共用一个类锁
- 有两个静态同步方法，两部手机， 请问先打印邮件还是短信？ 
	- 先邮件后短信 共用一个类锁
- 有一个静态同步方法 一个普通同步方法，请问先打印邮件还是短信？
	- 先短信后邮件   一个用类锁一个用对象锁
- 有一个静态同步方法，一个普通同步方法，两部手机
	- 先短信后邮件 一个类锁一个对象锁

```java
class Phone {
    public synchronized void sendEmail() {
        // TimeUnit.SECONDS.sleep(2);
        System.out.println("---sendEmail");
    }

    public synchronized void sendSMS() {
        System.out.println("---sendSMS");
    }

    public static synchronized void sendEmail2() {
        System.out.println("---sendEmail");
    }

    public static synchronized void sendSMS2() {
        System.out.println("---sendSMS");
    }

    public void hello(){
        System.out.println("---hello");
    }
}

public class Synchronized8Demo {
    public static void main(String[] args) throws Exception {
        Phone phone = new Phone();
        Phone phone2 = new Phone();

//        new Thread(phone::sendEmail).start();
//        // 保证线程的启动顺序
//        TimeUnit.SECONDS.sleep(1);
//        phone.hello();
//        new Thread(phone::sendSMS).start();

        new Thread(Phone::sendEmail2).start();
        TimeUnit.SECONDS.sleep(1);
        phone.hello();
        new Thread(Phone::sendSMS2).start();
    }
}
```

结论：
- 所有普通同步方法用的是同一把锁，即==当前实例对象==，通常指this
	- 一个普通同步方法拿到锁，其他都得等
- 所有静态同步方法用的是同一把锁，锁的时==当前类的Class对象（唯一模板class）==，如Phone.class
	- 静态同步方法与普通同步方法之间是不会有竞态条件的
- 对于同步方法块，锁的时synchronized==括号内的对象==

> 阿里巴巴开发手册
> 
> 能锁区块不锁方法，能用对象锁，不用类锁
>![](assets/Pasted%20image%2020231127185348.png)

### 字节码角度分析 synchronized

> 使用 Java 编译器（`javac` 命令），将 Java 源代码编译成字节码。`javac HelloWorld.java`
> 
> 上述命令将生成 `HelloWorld.class` 文件，其中包含了编译后的字节码。
> 
> `javap HelloWorld.class` 反编译

1）synchronized 同步代码块

实现使用的是 `monitorenter` 和 `monitorexit` 指令
- 一般情况是 1 个 enter 对应 2 个 exit
	- 一个正常退出、一个异常退出
	- 方法里面抛了异常了，那就只有一个 exit

```java
 public void m1();
    Code:
       0: aload_0
       1: getfield      #3                  // Field object:Ljava/lang/Object;
       4: dup
       5: astore_1
       6: monitorenter
       7: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
      10: ldc           #5                  // String ---hello synchronized code block
      12: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      15: aload_1
      16: monitorexit
      17: goto          25
      20: astore_2
      21: aload_1
      22: monitorexit
      23: aload_2
      24: athrow
      25: return
```

2）普通同步方法

调用指令将会检查方法的 `ACC SYNCHRONIZED` 访问标志是否被设置

如果设置了，执行线程会将先持有 monitor 锁，然后再执行方法，最后在方法完成（无论是正常完成还是非正常完成）时释放 monitor

```java
  public synchronized void m2();
    descriptor: ()V
    flags: (0x0021) ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String ---hello synchronized method
         5: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 18: 0
        line 19: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/boer/lock/LockSyncDemo;
```

3）静态同步方法

`ACC STATIC`,` ACC SYNCHRONIZED` 访问标志区分该方法是否是静态同步方法

```java
  public static synchronized void m3();
    descriptor: ()V
    flags: (0x0029) ACC_PUBLIC, ACC_STATIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=0, args_size=0
         0: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String ---hello static synchronized method
         5: invokevirtual #6                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 22: 0
        line 23: 8
```

### 反编译 synchronized 锁的是什么？

> 面试题：为什么任何一个对象都可以成为一个锁？

> 管程（英语:Monitors，也称为监视器）是一种程序结构，结构内的多个子程序(对象或模块)形成的多个工作线程互斥访问共享资源。
> 
> 这些共享资源一般是硬件设备或一群变量。对共享变量能够进行的所有操作集中在一个模块中。(把信号量及其操作原语“封装”在一个对象内部）管程实现了在一个时间点，最多只有一个线程在执行管程的某个子程序。管程提供了一种机制，管程可以看做一个软件模块，它是将共享的变量和对于这些共享变量的操作封装起来，形成一个具有一定接口的功能模块，进程可以调用管程来实现进程级别的并发控制。

> Java 虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步结构都是使用管程（Monitor，更常见的是直接将它称为“锁”）来实现的。
> 
> 方法级的同步是隐式的，无须通过字节码指令来控制，它实现在方法调用和返回操作之中。虚拟机可以从方法常量池中的方法表结构中的 `ACC SYNCHRONIZED` 访问标志得知一个方法是否被声明为同步方法。当方法调用时，调用指令将会检查方法的 `ACC SYNCHRONIZED` 访问标志是否被设置，如果设置了，执行线程就要求先成功持有管程，然后才能执行方法，最后当方法完成 (无论是正常完成还是非正常完成)时释放管程。在方法执行期间，执行线程持有了管程，其他任何线程都无法再获取

每个对象天生都带着一个“**对象监视器**”Monitor，每一个被锁住的对象都会和 Monitor 关联起来

在 HotSpot 虚拟机中，monitor 采用 `ObjectMonitor` 实现
`ObjectMonitor.java` --> `ObjectMonitor.cpp` --> `objectMonitor.hpp`
其主要数据结构如下（位于 HotSpot 虚拟机源码 `objectMonitor.hpp` 文件，C++实现的）

![](assets/Pasted%20image%2020231127220305.png)

指针指向 monitor 对象（也称为管程或监视器）的起始地址。每个对象都在在着一个 monitor 与之关联，当一个 monitor 被某个线程持有后，它便处于锁定状态。

![](assets/Pasted%20image%2020231129223403.png)

## 公平锁和非公平锁

公平锁：是指多个线程按照==申请锁的顺序==来获取锁，这里类似于排队买票，先来的人先买，后来的人再队尾排着，这是公平的
- `Lock lock = new ReentrantLock(true)`---表示公平锁，先来先得。

非公平锁：是指多个线程获取锁的顺序并不是按照申请的顺序，有可能后申请的线程比先申请的线程优先获取锁，在高并发环境下，有可能造成优先级反转或者饥饿的状态（某个线程一直得不到锁）
- `Lock lock = new ReentrantLock(false)` ---表示非公平锁，后来的也可能先获得锁，==默认为非公平锁==。

买票案例演示公平和非公平锁

```java
public class SaleTicketDemo {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        new Thread(() -> {
            for (int i = 0; i < 50; i++) {
                ticket.sale();
            }
        }, "a").start();
        new Thread(() -> {
            for (int i = 0; i < 50; i++) {
                ticket.sale();
            }
        }, "b").start();
        new Thread(() -> {
            for (int i = 0; i < 50; i++) {
                ticket.sale();
            }
        }, "c").start();
    }
}

class Ticket {
    private int num = 50;
    Lock lock = new ReentrantLock(true); // 公平锁

    public void sale() {
        lock.lock();
        try {
            if (num > 0) {
                System.out.println(
                        String.format("%s 抢到了第 %s 张票", Thread.currentThread().getName(), num--));
            }
        } finally {
            lock.unlock();
        }
    }
}
```

> 面试题：为什么会有公平锁和非公平锁的设计？为什么默认非公平

- 恢复挂起的线程到真正锁的获取还是有时间差的，从开发人员来看这个时间微乎其微，但是从 CPU 的角度来看，这个时间差存在的还是很明显的。
	- 所以非公平锁能更充分的利用 CPU 的时间片，==尽量减少 CPU 空闲状态时间==。
- 使用多线程很重要的考量点是线程切换的开销，当采用非公平锁时，当 1 个线程请求锁获取同步状态，然后释放同步状态，所以==刚释放锁的线程在此刻再次获取同步状态的概率就变得非常大，所以就减少了线程的开销。==


## 自旋锁 SpinLock

见[CAS与自旋锁，借鉴CAS思想](#CAS与自旋锁，借鉴CAS思想)

# LockSupport与线程中断
## 线程中断机制

> 阿里金服面试题：
> 1. 如何中断一个运行中的线程？
> 2. 如何停止一个运行中的线程？

一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止
所以`Thread.stop`, `Thread.suspend`, `Thread.resume` 都已经被废弃了。

在Java中没有办法立即停止一条线程，然而==停止线程尤为重要，如取消一个耗时操作==。
Java提供了**中断标识协商机制**。没有给中断增加任何语法，中断的过程完全需要程序员自己实现。

若要中断一个线程
- 需要手动调用该线程的`interrupt`方法，该方法也仅仅是将线程对象的中断标识设成true；
	- 每个线程对象都有一个中断标识位，用于表示线程是否被中断：true表示中断，false表示未中断
- 接着需要==自己写代码不断地检测当前线程的标识位==，如果为true，表示别的线程请求这条线程中断，此时究竟该做什么需要你自己写代码实现。
- 可以在别的线程中调用，也可以在自己的线程中调用

### Thread 中断机制三大方法

1）`public void interrupt()`：将线程对象的中断标识设成true

2）`public static boolean interrupted()` （见后续）

3）`public boolean isInterrupted()`：检查中断标识位，判断当前线程是否被中断

### 如何中断运行中线程？

1）volatile变量

```java
public class InterruptDemo {
    static volatile boolean isStop = false;

    public static void main(String[] args) {
        new Thread(() -> {
            while (true) {
                if (isStop) {
                    System.out.println(Thread.currentThread().getName() + "\t is stop");
                    break;
                }
                System.out.println("t1---hello");
            }
        }, "t1").start();

        try {
            TimeUnit.MILLISECONDS.sleep(20);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        new Thread(() -> {
            isStop = true;
        }, "t2").start();
    }
}
```

2）通过AtomicBoolean

```java
public class InterruptDemo {
    static AtomicBoolean atomicBoolean = new AtomicBoolean(false);

    public static void main(String[] args) {
        new Thread(() -> {
            while (true) {
                if (atomicBoolean.get()) {
                    System.out.println(Thread.currentThread().getName() + "\t is stop");
                    break;
                }
                System.out.println("t1---hello");
            }
        }, "t1").start();

        try {
            TimeUnit.MILLISECONDS.sleep(20);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        new Thread(() -> {
            atomicBoolean.set(true);
        }, "t2").start();
    }
}
```

3）通过Thread类自带的中断api实例方法实现

在需要中断的线程中不断监听中断状态，一旦发生中断，就执行相应的中断处理业务逻辑
前两种思想是一样的

```java
public class InterruptDemo {  
    public static void main(String[] args) {  
        Thread t1 = new Thread(() -> {  
            while (true) {  
                if (Thread.currentThread().isInterrupted()) {  
                    System.out.println(Thread.currentThread().getName() + "\t is stop");  
                    break;                }  
                System.out.println("t1---hello");  
            }  
        }, "t1");  
        t1.start();  
        System.out.println("t1的默认中断标识位："+t1.isInterrupted());  
  
        try {  
            TimeUnit.MILLISECONDS.sleep(10);  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
  
        t1.interrupt();  
    }  
}
```

### 当前线程的中断标识为true，是不是线程就立刻停止?

具体来说，当对一个线程，调用 `interrupt()` 时:
- 如果线程处于**正常活动状态**
	- 将该线程的中断标志设置为 true，仅此而已。该线程将继续正常运行，不受影响。
	- 所以， `interrupt()` 并==不能真正的中断线程，需要被调用的线程自己进行配合才行==。
- 如果线程处于**被阻塞状态**（sleep, wait, join 等状态）
	- 那么==它的中断状态将被清除==
	- 线程将立即退出被阻塞状态，并抛出一个`InterruptedException` 异常。
	- 所以，在==catch块中需要再次给中断表示为设置为true==
- 中断**不活动**的线程
	- 不会产生任何影响。即调用 `interrupt()`以后还是中断标志位还是false

---
中断sleep线程导致无限循环案例：

会无限输出 `t1---hello`，t2--->t1发出中断协商，t1处于阻塞状态，中断状态将被清除。

```java
public class InterruptDemo {  
    public static void main(String[] args) {  
        new Thread(() -> {  
            while (true) {  
                if (Thread.currentThread().isInterrupted()) {  
                    System.out.println(Thread.currentThread().getName() + "\t is stop");  
                    break;  
                }  
                try {  
                    Thread.sleep(200);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
                System.out.println("t1---hello");  
            }  
        }, "t1").start();  

        try {  
            TimeUnit.SECONDS.sleep(1);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
  
        new Thread(t1::interrupt,"t2").start();  
    }  
}
```

### 静态方法 Thread.interrupted()

`public static boolean interrupted()
1. 返回当前线程的中断状态，==测试当前线程是否已被中断==
2. 将当前线程的中断状态==清零并重新设为false==，清除线程的中断状态

```java
public class InterruptDemo4 {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
        System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
        System.out.println("---1");
        Thread.currentThread().interrupt();
        System.out.println("---2");
        System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
        System.out.println(Thread.currentThread().getName() + "\t" + Thread.interrupted());
        /**
         * main	false
         * main	false
         * ---1
         * ---2
         * main	true
         * main	false
         */
    }
}
```

---
和 `public boolean isInterrupted()` 关系

- 两者调用的都是同一个native方法

```java
public static boolean interrupted() {  
    return currentThread().isInterrupted(true);  
}  
  
public boolean isInterrupted() {  
    return isInterrupted(false);  
}

private native boolean isInterrupted(boolean ClearInterrupted);
```

方法的注释也清晰的表达了“中断状态将会根据==传入的Clearlnterrupted参数值确定是否重置==”
- **静态方法interrupted**会清除中断状态（传入的参数Clearlnterrupted为true）
- **实例方法islnterrupted**则不会（传入的参数Clearlnterrupted为false）

## 线程等待唤醒机制

### 三种让线程等待和唤醒的方式

1）Object 中的 `wait()` 方法让线程等待，使用 Object 中的 `notify()` 方法唤醒线程

- `wait()` 和 `notify()` 必须要在 **同步块/方法** 里，且成对出现
	- 否则会抛出 `IllegalMonitorStateException` 异常
- 顺序：先 `wait()` 后 `notify()` 

```java
public class Demo {  
    public static void main(String[] args) throws InterruptedException {  
        Object lock = new Object();  
  
        new Thread(() -> {  
            synchronized (lock) {  
                try {  
                    lock.wait();  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
            System.out.println(Thread.currentThread().getName() + "\t" + "被唤醒");  
        }, "t1").start();  
  
        Thread.sleep(1000);  
  
        new Thread(() -> {  
            synchronized (lock) {  
                lock.notify();  
                System.out.println(Thread.currentThread().getName() + "\t" + "发出通知");  
            }  
        }, "t2").start();  
    }  
}
```

--- 
2）JUC 包中 Condition 的 `await()` 让线程等待，`signal()` 唤醒线程

```java
public class Demo2 {
    public static void main(String[] args) throws InterruptedException {
        Lock lock = new ReentrantLock();
        Condition condition = lock.newCondition();

        new Thread(() -> {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "\t" + "come in");
                // ----- 等待
                condition.await();
                System.out.println(Thread.currentThread().getName() + "\t" + "被唤醒");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }, "t1").start();

        Thread.sleep(1000);

        new Thread(() -> {
            lock.lock();
            try {
                // ----- 唤醒
                condition.signal();
                System.out.println(Thread.currentThread().getName() + "\t" + "发出通知");
            }finally {
                lock.unlock();
            }
        }, "t2").start();
    }
}

```

注意：
- `await()`和`signal()`要在`lock()`和`unlock()`对里
	- 否则会抛出 `IllegalMonitorStateException` 异常
- 先 `await()` 后 `signal()`             

--- 
3）LockSupport 类

弥补了上两种方法的痛点。

### LockSupport

> java.util.concurrent.locks.LockSupport。用于**创建**锁和其他同步类的**基本线程阻塞原语**
> 
> LockSuppot是一个线程阻寒工具类，所有的方法都是**静态方法**，可以让线程在任意位置阻塞，阻寒之后也有对应的唤醒方法。归根结底，==LockSupport调用的Unsafe中的native代码==。
> 
> LockSupport使用了一种名为Permit（凭证）的概念来做到阻塞和唤醒线程的功能
> - 每个线程都有一个permit
> - 但与Semaphore不同的是，Permit最多只有一个

主要方法：
- `public static void park()` 阻塞当前线程
	- 如果有permit，则会直接消耗掉这个凭证然后正常退出
	- 如果没有permit，就必须阻塞等待凭证可用
		- 默认是没有的，所以第一次调用`park()`后当前线程就会阻塞，直到别的线程发放permit，才会被唤醒
- `public static void park(Object blocker)` 阻塞传入的具体线程
- `public static void unpark(Thread thread)` 唤醒线程
	- 增加一个permit，最多一个，累加无效

```java
public class Demo3 {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
	        // 先唤醒后等待也可以
            // try {
            //     Thread.sleep(2000);
            // } catch (InterruptedException e) {
            //     e.printStackTrace();
            // }
            System.out.println(Thread.currentThread().getName() + "\t" + "come in");
            // ----- 等待
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + "\t" + "被唤醒");
        }, "t1");
        t1.start();

        Thread.sleep(1000);

        new Thread(() -> {
            // ----- 唤醒
            LockSupport.unpark(t1);
            System.out.println(Thread.currentThread().getName() + "\t" + "发出通知");
        }, "t2").start();
    }
}
```

特点：
- 无锁块要求
- 之前错误的先唤醒后等待，也支持
- `park()` 和 `unpark()` 要成双成对

> 面试题一：为什么可以突破`wait/notify`原有调用顺序

因为unpark获得了一个凭证，之后调用park()会消费凭证。

> 面试题二：为什么唤醒两次后阻塞两次，但最终结果还会阻塞线程？

因为凭证最多一个，两次unpark()只会增加一个凭证，而调用两次park()需要消费两个凭证。

# Java 内存模型之 JMM

## JMM 引入和介绍

> 从大厂面试开始
> - 你知道什么是 Java 内存模型 JMM 吗？
> - JMM 和 volatile 他们两个之间的关系？
> - JMM 有哪些特征或者它的三大特征是什么？
> - 为什么要有 JMM，它为什么出现？作用和功能是什么？
> - happens-before 先行并发原则你有了解过吗？

### CPU 缓存模型

计算机硬件存储体系

![|700](assets/Pasted%20image%2020240106181702.png)

![](assets/Pasted%20image%2020240205210423.png)

CPU的运行并不是直接操作内存，而是先把内存里边的数据读到缓存。内存的读和写操作的时候就会造成不一致的问题。

![|400](assets/Pasted%20image%2020240106181855.png)



## JMM 规范下，多线程先行发生原则之 happens-before

在JMM中如果一个操作**执行的结果**需要对另一个操作 **可见** 或者 **代码重排序（结果一致）**，那么这两个操作之间必须存在happens-before（先行发生）原则逻辑上的先后关系。

> 案例

写后读：
- x=5，线程A执行
- y=x，线程B执行

y是否等于5呢？如果线程A的操作 (x=5) happens-before (先行发生) 线程B的操作 (y=x) ，那么可以确定线程B执行后 y=5 一定成立。

如果他们不存在happens-before原则，那么y=5 不一定成立。

这就是happens-before原则的威力。==包含可见性和有序性的约束==。

> 先行发生原则说明

如果Java内存模型中所有的有序性都仅靠volatile和synchronized来完成，那么有很多操作都将会变得非常啰嗦，但是我们在编写Java并发代码的时候并没有察觉到这一点。

==我们没有时时、处处、次次，添加volatile和synchrnized来完成程序==，这是因为Java语言中JMM原则下有一个“先行发生”(Happens-Before)的原则限制和规矩，给你立好了规矩！

这个原则非常重要。它是判断数据是否存在竞争，线程是否安全的非常有用的手段。依赖这个原则，我们可以通过几条简单规则一揽子解决并发环境下两个操作之间是否可能存在冲突的所有问题，而不需要陷入Java内存模型苦涩难懂的底层编译原理之中。

> happens-before总原则

如果一个操作happens-before另一个操作，那么==第一个操作的执行结果将对第二个操作可见==，而且第一个操作的执行顺序排在第二个操作之前。

两个操作之间存在happens-before关系，并不意味着一定要按照happens-before原则制定的顺序来执行如果重排序之后的执行结果与按照happens-before关系来执行的==结果一致，那么这种重排序并不非法==。

> happens-before之8条

1. 次序规则：==一个线程内==，按照代码顺序，写在前面的操作**先行发生于**写在后面的操作
	1. （通俗）前一个操作的结果可以被后续的操作获取
2. 锁定规则：一个unLock操作**先行发生于**后面对同一个锁的lock操作
	1. 后面 指时间上的先后
3. volatile变量规则：对一个volatile变量的写操作先行发生于后面对这个变量的读操作
	1. 前面的写对后面的读是可见的，这里的“后面”同样是指==时间上的先后==
4. 传递规则：如果操作A**先行发生于**操作B，而操作B又**先行发生于**操作C，则可以得出操作A**先行发生于**操作C
5. 线程启动规则：Thread对象的`start()`方法**先行发生于**此线程的每一个动作
6. 线程中断规则：对线程`interrupt()`方法的调用**先行发生于**被中断线程的代码检测到中断事件的发生
	1. 可以通过`Thread.interrupted()`检测到是否发生中断
	2. 要先调用`interrupt()`方法设置过中断标志位，才能检测到中断发送
7. 线程终止规则：线程中的所有操作都**优先发生于**对此线程的终止检测
	1. 可以通过`isAlive()`等手段检测线程是否已经终止执行。
8. 对象终结规则：一个对象的初始化完成（构造函数执行结束）**先行发生于**它的`finalize()`方法的开始
	1. 对象没有完成初始化之前，是不能调用`finalized()`方法的

> happens-before 小总结

在Java语言里面，Happens-before的语义本质上是一种**可见性**

 A happens-before B ，意味着A发生过的事情对B而言是可见的，==无论A事件和B事件是否发生在同一线程里==
 
JVM的设计分为两部分：
- 一部分是面向我们程序员提供的，也就是happens-before规则，它通俗易懂的向我们程序员阐述了一个强内存模型，==我们只要理解happens-before规则，就可以编写并发安全的程序了==
- 另一部分是针对JVM实现的，为了尽可能少的对编译器和处理器做约束从而提升性能，JMM在不影响程序执行结果的前提下对其不做要求，即允许优化重排序，==我们只要关注前者就好了==，也就是理解happens-before规则即可，其他繁杂的内容由JMM规范结合操作系统给我们搞定，我们只写好代码即可。

> 案例说明

```java
private int value =0;
public int getValue(){
    return value;
}
public int setValue(){
    return ++value;
}
```

问题描述：假设存在线程A和B，线程A先（时间上的先后）调用了setValue()方法，然后线程B调用了同一个对象的getValue()方法，那么线程B收到的返回值是什么？

答案：不一定

分析happens-before规则（规则5，6，7，8可以忽略，和代码无关）
1. 由于两个方法由不同线程调用，==不满足一个线程的条件==，不满足程序次序规则
2. 两个方法==都没有用锁==，不满足锁定规则
3. 变量==没有使用volatile修饰==，所以不满足volatile变量规则
4. 传递规则肯定不满足

综上：==无法通过happens-before原则推导出 线程A happens-before 线程B==，虽然可以确定时间上线程A优于线程B，但就是无法确定线程B获得的结果是什么，所以==这段代码不是线程安全的==

注意：如果两个操作的执行次序无法从happens-before原则推导出来，那么就==不能保证他们的有序性==，虚拟机可以==随意对他们进行重排序==

--- 
如何修复？

1）把getter/setter方法都定义为synchronized方法--->不好，重量锁，并发性下降
```java
private int value =0;

public synchronized int getValue(){
    return value;
}

public synchronized int setValue(){
    return ++value;
}
```

2）把==value定义为volatile变量==，由于setter() 对value的修改不依赖value的原值，满足volatile关键字使用场景
```java
/**
* 利用volatile保证读取操作的可见性，
* 利用synchronized保证符合操作的原子性结合使用锁和volatile变量来减少同步的开销
*/
private volatile int value =0;

public int getValue(){
    return value;
}

public synchronized int setValue(){
    return ++value;
}
```

# CAS

## CAS 引入

> 原子类

Java.util.concurrent.atomic

> 没有CAS之前

多线程环境中不使用原子类保证线程安全i++（基本数据类型）
```java
class Test {
	private volatile int count = 0;
	
	// 若要线程安全执行执行count++，需要加锁
	public synchronized void increment() {
		 count++;
	}

	public int getCount() {
		 return count;
	}
}
```

> 使用CAS之后

多线程环境中使用原子类保证线程安全i++（基本数据类型）--->类似于乐观锁
```java
class Test2 {
	private AtomicInteger count = new AtomicInteger();

	public void increment() {
		 count.incrementAndGet();
	}
	
    // 使用AtomicInteger之后，不需要加锁，也可以实现线程安全。
	public int getCount() {
		 return count.get();
	}
}
```

## CAS 是什么？

> 概念说明

CAS (compare and swap)，比较并交换，实现并发算法时常用到的一种技术，用于保证**共享变量的原子性更新**。

它包含三个操作数：内存位置、预期原值与更新值。

执行CAS操作的时候，将**内存位置的值**与**预期原值**进行比较：
- 如果相匹配，那么处理器会自动将该位置更新为新值
- 如果不匹配，处理器不做任何操作或重来，这种重来重试的行为称为**自旋**

多个线程同时执行CAS操作==只有一个会成功==。

> 案例演示

```java
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(5);
        /**
         * true     2022
         * false	2022
         */
        System.out.println(atomicInteger.compareAndSet(5, 2022) + "\t" + atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 2023) + "\t" + atomicInteger.get());
    }
}
```

> 源码查看

AtomicInteger
```java
public final boolean compareAndSet(int expect, int update) {  
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
}
```

Unsafe
```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

上面三个方法都是类似的，主要对4个参数做一下说明。
1. var1: 表示要操作的对象
2. var2: 表示要操作对象中属性地址的偏移量
3. var4: 表示需要修改数据的期望的值
4. var5/var6: 表示需要修改为的新值

> 硬件级别保证

**非阻塞！**

CAS是JDK提供的**非阻塞原子性操作**，它通过硬件保证了比较-更新的原子性。

它是非阻塞的且自身具有原子性，也就是说这玩意效率更高且通过硬件保证，说明这玩意更可靠。

CAS是一条CPU的原子指令 (**cmpxchg指令**)，不会造成所谓的数据不一致问题，**Unsafe** 提供的CAS方法 (如compareAndSwapXXX) 底层实现即为CPU指令cmpxchg。

执行cmpxchg指令的时候，会判断当前系统是否为多核系统，如果是就给总线加锁，只有一个线程会对总线加锁成功，加锁成功之后会执行cas操作，也就是说==CAS的原子性实际上是CPU实现独占的，比起用synchronized重量级锁， 这里的排他时间要短很多， 所以在多线程情况下性能会比较好。==

## CAS底层原理？谈谈你对Unsafe的理解

### Unsafe

> 概念说明

Unsafe 类是 **CAS 的核心类**

由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe 类中的==所有方法都是native修饰的==，都能直接调用操作系统底层资源执行相应任务。

Unsafe 类存在于 sun.misc 包中，其内部方法操作可以像C的指针一样直接操作内存，因此==Java 中 CAS 操作的执行依赖于 Unsafe 类的方法==。

> AtomicInteger

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

	......
}
```

`valueOffset`：表示该变量值在内存中的偏移地址，因为Unsafe就是根据内存偏移地址获取数据的

`volatile`：变量value用volatile修饰，保证了多线程之间的内存可见性。

> 我们知道 i++ 是线程不安全的，那  `atomiclnteger.getAndIncrement()`？

AtomicInteger类主要利用==CAS+volatile和native方法==来保证原子操作，从而避免synchronized的高开销，执行效率大为提升

Atomiclnteger类
```java
public final int getAndIncrement() {  
    return unsafe.getAndAddInt(this, valueOffset, 1);  
}
```

Unsafe类
```java
public final int getAndAddInt(Object var1, long var2, int var4) {  
	// var4=1
    int var5;  
    do {  
		// 获得当前的最新值，期望值
        var5 = this.getIntVolatile(var1, var2);  
        // 自旋：期望值和当前内存值不一致就一直重试
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));  

    return var5;
}
```

**CAS并发原语**==体现在Java语言中==就是Unsafe类中的各个方法。调用Unsafe类中的CAS方法，==JVM会帮我们实现出CAS汇编指令==。这是一种完全依赖于硬件的功能，通过它实现了原子操作。

再次强调，由于CAS是一种**系统原语**，原语属于操作系统用语范畴，是由若干条指令组成的，用于完成某个功能的一个过程，并且==原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成所谓的数据不一致问题。==

个人理解：`compareAndSwapInt()` 这个方法就是原语级别的执行，保证了自旋的原子性

> 源码分析

假设线程A和线程B两个线程同时执行`getAndAddint()`操作(分别跑在不同CPU上) :
1. Atomiclnteger里面的value原始值为3，即主内存中Atomicinteger的value为3，根据JMM模型，线程A和线程B各自持有份值为3的value的副本分别到各自的工作内存。
2. 线程A通过`getlntVolatile(var1,var2)`拿到value值3，这时线程A被挂起
3. 线程B也通过`getlntVolatile(var1, var2)`方法获取到value值3，此时刚好线程B没有被挂起并执行`compareAndSwapInt()`比较内存值也为3，成功修改内存值为4，线程B打完收工，一切OK。
4. 这时线程A恢复，执行`compareAndSwaplnt()`比较，发现自己手里的值数字3和主内存的值数字4不一致，说明该值已经被其它线程抢先一步修改过了，那A线程本次修改失败，只能重新读取重新来一遍了。
5. 线程A重新获取value值，因为变量value被volatile修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行`compareAndSwaplnt()`进行比较替换，直到成功。

### 底层汇编

[74_CAS之Unsafe类底层汇编源码分析_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1ar4y1x727/?p=74&spm_id_from=pageDriver&vd_source=2c36db3ac89c0a3fdac39c4e8a1068fa)

你只需要记住：CAS是靠硬件实现的从而在硬件层面提升效率，最底层还是交给硬件来保证原子性和可见性

实现方式是基于硬件平台的汇编指令，在intel的CPU中(X86机器上)，使用的是**汇编指令cmpxchg指令**。

核心思想就是：比较要更新变量的值V和预期值E (compare)，相等才会将V的值设为新值N  (swap) 如果不相等自旋再来.

## 原子引用

Atomiclnteger 原子整型，可否有其他原子类型？例如：AtomicUser、AtomicOrder

java.util.concurrent.atomic.AtomicReference

> AtomicReference案例

```java
public class AtomicReferenceDemo {
    public static void main(String[] args) {
        AtomicReference<User> userAtomicReference = new AtomicReference<>();
        User user1 = new User("user1", 22);
        User user2 = new User("user2", 33);
        User user3 = new User("user3", 33);

        userAtomicReference.set(user1);

        /**
         * true	    User(username=user2, age=33)
         * false	User(username=user2, age=33)
         */
        System.out.println(userAtomicReference.compareAndSet(user1, user2) + "\t" +
                userAtomicReference.get().toString());
        System.out.println(userAtomicReference.compareAndSet(user1, user3) + "\t" +
                userAtomicReference.get().toString());
    }
}
```

## CAS与自旋锁，借鉴CAS思想

> 概念说明

CAS是实现自旋锁的基础，CAS利用CPU指令保证了操作的原子性，以达到锁的效果

自旋锁，字面意思自己旋转。是指尝试获取锁的线程==不会立即阻塞==，而是采用==循环的方式去尝试获取锁==，当线程发现锁被占用时，会不断循环判断锁的状态，直到获取。

这样的**好处**是减少线程上下文切换的消耗，**缺点**是循环会消耗CPU

**By Boer：非阻塞式，减少线程上下文切换的消耗**

> 实现自旋锁

```java
public class SpinLockDemo {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void lock() {
        Thread thread = Thread.currentThread();
        System.out.println(Thread.currentThread().getName() + "\t" + "--- come in ");
        while (!atomicReference.compareAndSet(null, thread)) ;
        System.out.println(Thread.currentThread().getName() + "\t" + "--- lock ");
    }

    public void unlock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(Thread.currentThread().getName() + "\t" + "--- unlock ");
    }

    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();

        new Thread(() -> {
            spinLockDemo.lock();
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            spinLockDemo.unlock();
        }, "A").start();

        new Thread(() -> {
            spinLockDemo.lock();
            spinLockDemo.unlock();
        }, "B").start();

        /**
         * A	--- come in
         * B	--- come in
         * A	--- lock
         * A	--- unlock
         * B	--- lock
         * B	--- unlock
         */
    }
}
```

## CAS 缺点

> 循环时间长开销很大

`getAndAddInt()`方法执行时有个 do while，如果CAS失败会一直进行尝试。

如果CAS长时间一直不成功，就会给CPU带来很大的开销！

> 会导致 “ABA问题”

CAS算法实现一个重要前提需要取出内存中某时刻的数据并在当下时刻比较并替换，那么在这个时间差类会导致数据的变化。

比如说
- 一个线程1从内存位置V中取出A，
- 这时候另一个线程2也从内存中取出A，
- 并且线程2进行了一些操作将值变成了B
- 然后线程2又将V位置的数据变成A，
- 这时候线程1进行CAS操作发现内存中仍然是A，预期OK，然后线程1操作成功。

尽管线程1的CAS操作成功，但是==不代表这个过程就是没有问题的。==

## 版本号时间戳原子引用

java.util.concurrent.atomic.AtomicStampedReference

用于解决ABA问题

```java
public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
```

> 单线程——AtomicStampedReference带戳记流水的简单演示

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
class Book {
    private int id;
    private String bookName;
}

public class AtomicStampedReferenceDemo {
    public static void main(String[] args) {
        Book javaBook = new Book(1, "javaBook");
        AtomicStampedReference<Book> atomicStampedReference = new AtomicStampedReference<>(javaBook, 1);
        System.out.println(atomicStampedReference.getReference() + "\t" + atomicStampedReference.getStamp());

        Book mysqlBook = new Book(2, "mysqlBook");
        boolean b;
        b = atomicStampedReference.compareAndSet(javaBook, mysqlBook, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        System.out.println(b + "\t" + atomicStampedReference.getReference() + "\t" + atomicStampedReference.getStamp());

        b = atomicStampedReference.compareAndSet(mysqlBook, javaBook, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        System.out.println(b + "\t" + atomicStampedReference.getReference() + "\t" + atomicStampedReference.getStamp());
        /**
         * Book(id=1, bookName=javaBook)	1
         * true	Book(id=2, bookName=mysqlBook)	2
         * true	Book(id=1, bookName=javaBook)	3
         */
    }
}
```

> 多线程情况下演示AtomicStampedReference解决ABA问题

By Boer. 先拿到版本号再做事
```java
public class ABADemo {
    static AtomicInteger atomicInteger = new AtomicInteger(100);
    static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(100, 1);

    public static void main(String[] args) {
//        abaHappen();//true	2023
        /**
         * t3	首次版本号: 1
         * t4	首次版本号: 1
         * t3	2次版本号: 2
         * t3	3次版本号: 3
         * false	100	3
         */
        abaNoHappen();
    }

    private static void abaNoHappen() {
        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t" + "首次版本号: " + stamp);

            try {
                TimeUnit.MILLISECONDS.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            atomicStampedReference.compareAndSet(100, 101,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t" + "2次版本号: " + atomicStampedReference.getStamp());

            atomicStampedReference.compareAndSet(101, 100,
                    atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
            System.out.println(Thread.currentThread().getName() + "\t" + "3次版本号: " + atomicStampedReference.getStamp());
        }, "t3").start();

        new Thread(() -> {
            int stamp = atomicStampedReference.getStamp();
            System.out.println(Thread.currentThread().getName() + "\t" + "首次版本号: " + stamp);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            boolean b = atomicStampedReference.compareAndSet(100, 200, stamp, stamp + 1);
            System.out.println(b + "\t" + atomicStampedReference.getReference() + "\t" + atomicStampedReference.getStamp());
        }, "t4").start();
    }

    private static void abaHappen() {
        new Thread(() -> {
            atomicInteger.compareAndSet(100, 101);
            try {
                TimeUnit.MILLISECONDS.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            atomicInteger.compareAndSet(101, 100);
        }, "t1").start();


        new Thread(() -> {
            try {
                TimeUnit.MILLISECONDS.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(atomicInteger.compareAndSet(100, 2023) + "\t" + atomicInteger.get());//true	2023
        }, "t2").start();
    }
}
```

# 原子操作类

> java.util.concurrent.atomic

上半场
1. AtomicBoolean
2. AtomicInteger
3. AtomicIntegerArray
4. AtomicIntegerFieldUpdater
5. AtomicLong
6. AtomicLongArray
7. AtomicLongFieldUpdater
8. AtomicMarkableReference
9. AtomicReference
10. AtomicReferenceArray
11. AtomicReferenceFieldUpdater
12. AtomicStampedReference

下半场
1. DoubleAccumulator
2. DoubleAdder 
3. LongAccumulator
4. LongAdder

> 参考阿里巴巴开发手册，编程规约——并发处理

【参考】**volatile** 解决多线程内存不可见问题对于==一写多读==，是可以解决变量同步问题，==但是如果多写，同样无法解决线程安全问题== （By Boer. 原子类来解决）

说明：如果是 count++操作，使用如下类实现：

```java
AtomicInteger count = new Atomiclnteger();
count.addAndGet(1):
```

如果是JDK8，==推荐使用 LongAdder 对象，比 AtomicLong 性能更好== (减少乐观锁的重试次数)

## 数组类型原子类

- `AtomicIntegerArray`：整型数组原子类
- `AtomicLongrArray`：长整型数组原子类
- `AtomicReferenceArray`：引用类型数组原子类

> 常用 API

```java
public final int get(int i) //获取 index=i 位置元素的值
public final int getAndSet(int i, int newValue) //返回 index=i 位置的当前的值，并将其设置为新值：newValue
public final int getAndIncrement(int i) //获取 index=i 位置元素的值，并让该位置的元素自增
public final int getAndDecrement(int i) //获取 index=i 位置元素的值，并让该位置的元素自减
public final int getAndAdd(int i, int delta) //获取 index=i 位置元素的值，并加上预期的值
boolean compareAndSet(int i, int expect, int update) //如果输入的数值等于预期值，则以原子方式将 index=i 位置的元素值设置为输入值（update）
public final void lazySet(int i, int newValue) //最终 将index=i 位置的元素设置为newValue,使用 lazySet 设置之后可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```

> 案例

```java
public class AtomicIntegerArrayDemo {
    public static void main(String[] args) {
//        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[]{1, 2, 3, 4, 5});
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[5]);
        for (int i = 0; i < atomicIntegerArray.length(); i++) {
            System.out.println(atomicIntegerArray.get(i));
        }
        System.out.println();

        int tempInt = 0;
        // 更新 位置 i 的值并返回旧值
        tempInt = atomicIntegerArray.getAndSet(0, 1122);
        System.out.println(tempInt + "\t" + atomicIntegerArray.get(0)); // 0	1122
        // 位置 i 的值自增 并返回旧值
        tempInt = atomicIntegerArray.getAndIncrement(0);
        System.out.println(tempInt + "\t" + atomicIntegerArray.get(0)); // 1122	1123
    }
}
```

## 引用类型原子类

- `AtomicReference`：引用类型原子类。[原子引用](#原子引用)
- `AtomicStampedReference`：原子更新带有版本号的引用类型。该类将整数值与引用关联起来，可用于解决原子的更新数据和数据的版本号。[版本号时间戳原子引用](#版本号时间戳原子引用)
	- 可以解决使用 CAS 进行原子更新时可能出现的 ABA 问题。
	- 解决修改过几次
- `AtomicMarkableReference`：原子更新带有标记的引用类型。
	- 解决是否修改过
		- 它的定义就是==将标记戳简化为 true/false==
		- 类似于一次性筷子

## 对象的属性修改原子类

- `AtomicIntegerFieldUpdater`：原子更新对象中int类型字段的值
- `AtomicLongFieldUpdater`：原子更新对象中Long类型字段的值
- `AtomicReferenceFieldUpdater`：原子更新对象中引用类型字段的值

> 使用目的

以一种线程安全的方式操作非线程安全对象内的某些字段

> 使用要求

- 更新的对象属性==必须使用public volatile修饰符==
- 因为对象的属性修改类型原子类都是**抽象类**，所以每次使用都必须使用静态方法`newUpdater()`==创建一个更新器，并且需要设置想要更新的类和属性==

> AtomicIntegerFieldUpdater 案例

```java
class BankAccount {
    public volatile int money = 0;

    AtomicIntegerFieldUpdater<BankAccount> atomicIntegerFieldUpdater = AtomicIntegerFieldUpdater.newUpdater(BankAccount.class, "money");

    public void transferMoney(BankAccount bankAccount) {
        atomicIntegerFieldUpdater.getAndIncrement(bankAccount);
    }
}

/**
 * @author Boer
 */
public class AtomicIntegerUpdaterDemo {
    static int SIZE = 10;

    public static void main(String[] args) throws InterruptedException {
        BankAccount bankAccount = new BankAccount();
        CountDownLatch countDownLatch = new CountDownLatch(SIZE);
        for (int i = 0; i < SIZE; i++) {
            new Thread(() -> {
                try {
                    for (int j = 0; j < 1000; j++) {
                        bankAccount.transferMoney(bankAccount);
                    }
                } finally {
                    countDownLatch.countDown();
                }
            }).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + '\t' + bankAccount.money);
    }
}
```

> AtomicReferenceFieldUpdater 案例

需求：多线程并发调用一个类的初始化方法，如果未被初始化过，将执行初始化工作。要求只能被初始化一次，只有一个线程操作成功
```java
class MyVar { // 资源类
    public volatile Boolean isInit = Boolean.FALSE;
    AtomicReferenceFieldUpdater<MyVar, Boolean> referenceFieldUpdater =
            AtomicReferenceFieldUpdater.newUpdater(MyVar.class, Boolean.class, "isInit");

    public void init(MyVar myVar) {
        if (referenceFieldUpdater.compareAndSet(myVar, Boolean.FALSE, Boolean.TRUE)) {
            System.out.println(Thread.currentThread().getName() + "\t" + "--------------start init ,need 2 seconds");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "\t" + "--------------over init");
        } else {
            System.out.println(Thread.currentThread().getName() + "\t" + "--------------已经有线程进行初始化工作了。。。。。");
        }
    }
}

/**
 * @author Boer
 */
public class AtomicReferenceFieldUpdaterDemo {
    public static void main(String[] args) {
        MyVar myVar = new MyVar();
        for (int i = 1; i <= 5; i++) {
            new Thread(() -> {
                myVar.init(myVar);
            }, "Thread" + i).start();
        }
    }
}
/**
 * Thread1	--------------start init ,need 2 seconds
 * Thread5	--------------已经有线程进行初始化工作了。。。。。
 * Thread2	--------------已经有线程进行初始化工作了。。。。。
 * Thread4	--------------已经有线程进行初始化工作了。。。。。
 * Thread3	--------------已经有线程进行初始化工作了。。。。。
 * Thread1	--------------over init
 */
```

## 原子操作增强类原理深度解析

- `DoubleAccumulator`：一个或多个变量，它们一起保持运行double使用所提供的功能更新值
- `DoubleAdder`：一个或多个变量一起保持初始为零double总和
- `LongAccumulator`：一个或多个变量，一起保持使用提供的功能更新运行的值long ，提供了自定义的函数操作
- `LongAdder`：一个或多个变量一起维持初始为零long总和（重点），只能用来计算加法，且从0开始计算

> 阿里面试题

1. 热点商品点赞计算器，点赞数加加统计，不要求实时精确
2. 一个很大的list，里面都是int类型，如何实现加加，思路？

### LongAdder 和 LongAccumulator

- LongAdder只能用来计算加法，且从零开始计算
- LongAccumulator提供了自定义的函数操作

> 入门 API 调用案例

```java
public class LongAdderAPIDemo {
    public static void main(String[] args) {
        LongAdder longAdder = new LongAdder();

        longAdder.increment();
        longAdder.increment();
        System.out.println(longAdder.sum()); //2
        longAdder.add(2L);
        System.out.println(longAdder.sum()); //4

        LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x * y, 5);
        longAccumulator.accumulate(2);
        System.out.println(longAccumulator.get()); //10
        longAccumulator.accumulate(3);
        System.out.println(longAccumulator.get()); //30
    }
}

```

> 性能优势——点赞计数器案例

【阿里巴巴Java开发手册】如果是JDK8，==推荐使用 LongAdder 对象，比 AtomicLong 性能更好== (减少乐观锁的重试次数)

【需求】50个线程，每个线程100w点赞，总点赞数出来  
```java
/**  
 * 需求：50个线程，每个线程100w点赞，总点赞数出来  
 */  
class ClickNumber {  
    int number = 0;  
  
    public synchronized void clickBySynchronized() {  
        number++;  
    }  
  
    AtomicLong atomicLong = new AtomicLong(0);  
  
    public void clickByAtomicLong() {  
        atomicLong.getAndIncrement();  
    }  
  
    LongAdder longAdder = new LongAdder();  
  
    public void clickByLongAdder() {  
        longAdder.increment();  
    }  
  
    LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x + y, 0);  
  
    public void clickByLongAccumulator() {  
        longAccumulator.accumulate(1);  
    }  
}  
  
/**  
 * @author Boer  
 */public class AccumulatorCompareDemo {  
    public static final int NUMBER_OF_LIKES = 1000000; // 每个线程点赞次数  
    public static final int THREAD_NUMBER = 50;  
  
    public static void costTime(ClickNumber clickNumber, Runnable runnable, String clickByName) throws InterruptedException {  
        Long startTime = System.currentTimeMillis();  
        CountDownLatch countDownLatch = new CountDownLatch(THREAD_NUMBER);  
        for (int i = 1; i <= THREAD_NUMBER; i++) {  
            new Thread(() -> {  
                try {  
                    for (int j = 1; j <= NUMBER_OF_LIKES; j++) {  
                        runnable.run();  
                    }  
                } finally {  
                    countDownLatch.countDown();  
                }  
            }, String.valueOf(i)).start();  
        }  
        countDownLatch.await();  
        Long endTime = System.currentTimeMillis();  
        System.out.println("------costTime: " + (endTime - startTime) + " 毫秒\t" + clickByName + ": " + clickNumber.number);  
    }  
  
    public static void main(String[] args) throws InterruptedException {  
        ClickNumber clickNumber = new ClickNumber();  
  
        costTime(clickNumber, clickNumber::clickBySynchronized, "clickBySynchronized");  
        costTime(clickNumber, clickNumber::clickByAtomicLong, "clickByAtomicLong");  
        costTime(clickNumber, clickNumber::clickByLongAdder, "clickByLongAdder");  
        costTime(clickNumber, clickNumber::clickByLongAccumulator, "clickByLongAccumulator");  
    }  
}  
/**  
 * ------costTime: 3249 毫秒  clickBySynchronized: 50000000  
 * ------costTime: 826 毫秒   clickByAtomicLong: 50000000  
 * ------costTime: 55 毫秒    clickByLongAdder: 50000000  
 * ------costTime: 36 毫秒    clickByLongAccumulator: 50000000  
 */
```

### longAdder 源码分析

跳过


# ThreadLocal

## ThreadLocal 简介

> 面试题 

- ThreadLocal中ThreadLocalMap的数据结构和关系？
- ThreadLocal的key是弱引用，这是为什么？
- ThreadLocal内存泄漏问题你知道吗？
- ThreadLocal中最后为什么要加remove方法？

> 是什么？

ThreadLocal提供**线程局部变量**。

这些变量与正常的变量不同，因为每一个线程在访问ThreadLocal实例的时候（通过其get或set方法）==都有自己的、独立初始化的变量副本==。

ThreadLocal实例通常是类中的私有静态字段，使用它的目的是希望==将状态（例如，用户ID或事物ID）与线程关联起来==。

> 能干嘛？

主要解决了让每个线程绑定自己的值，通过使用get()和set()方法，获取默认值 或 将其改为当前线程所存的副本的值，从而==避免了线程安全问题==。

比如8锁案例中，资源类是使用同一部手机，多个线程抢夺同一部手机，假如人手一份不是天下太平？

## ThreadLocal 的使用

> API

![](assets/Pasted%20image%2020240111160943.png)

> 阿里Java开发手册

【强制】SimpleDateFormat 是**线程不安全**的类，==一般不要定义为 static 变量==，如果定义为 static，必须加锁，或者使用 DateUtils 工具类。

正例：注意线程安全，使用 DateUtils。亦推荐如下处理：
```java
private static final ThreadLocal<DateFormat> dateStyle = new ThreadLocal<DateFormat>() {
	@Override
	protected DateFormat initialValue() {
		return new SimpleDateFormat("yyyy-MM-dd");
	}
};
```

说明：如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar，DateTimeFormatter 代替 SimpleDateFormat，官方给出的解释：simple beautiful strong immutable **thread-safe**。

>initialValue()

返回此线程局部变量的当前线程的“**初始值**”。此方法将在线程第一次使用 get() 方法访问变量时调用，除非该线程先前调用了set(T)方法，在这种情况下，该线程将不会调用initialValue方法。通常，每个线程最多调用此方法一次，但如果随后调用remove() 和 get()，则可能会再次调用该方法。

此实现仅返回null；==如果程序员希望线程局部变量具有null以外的初始值，则必须子类化ThreadLocal，并覆盖此方法==。通常，将使用**匿名内部类**。

```java
protected T initialValue() {  
    return null;  
}
```

>withInitial()

创建线程局部变量。 变量的初始值是通过调用Supplier的 get 方法来确定的。

`withInitial()` 是 Since 1.8，推荐用于初始化线程局部变量
```java
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {  
    return new SuppliedThreadLocal<>(supplier);  
}
```

> 卖房案例

卖房子——房产中介销售都有自己的销售额指标
```java
class House {
    int saleCount = 0;

    public synchronized void saleHouse() {
        saleCount++;
    }

    // 匿名内部类实现initialValue()--->初始化线程局部变量
//    ThreadLocal<Integer> saleVolume = new ThreadLocal<Integer>() {
//        @Override
//        protected Integer initialValue() {
//            return 0;
//        }
//    };

    // 静态方法 withInitial()--->初始化线程局部变量
    ThreadLocal<Integer> saleVolume = ThreadLocal.withInitial(() -> 0);

    public void saleVolumeByThreadLocal() {
        saleVolume.set(1 + saleVolume.get());
    }
}

/**
 * @author Boer
 */
public class ThreadLocalDemo {
    final static int SALESMAN_NUM = 5;

    public static void main(String[] args) throws InterruptedException {
        House house = new House();
        for (int i = 1; i <= SALESMAN_NUM; i++) {
            new Thread(() -> {
                int size = new Random().nextInt(5) + 1;
                try {
                    for (int j = 1; j <= size; j++) {
                        house.saleHouse();
                        house.saleVolumeByThreadLocal();
                    }
                    System.out.println(Thread.currentThread().getName() + " 号销售卖出：" + house.saleVolume.get());
                } 
                finally {
                    house.saleVolume.remove();
                }
            }, String.valueOf(i)).start();
        }
        TimeUnit.MILLISECONDS.sleep(300);
        System.out.println(Thread.currentThread().getName() + "\t" + "共计卖出多少套： " + house.saleCount);
    }
}
/**
 * 1 号销售卖出：1
 * 3 号销售卖出：1
 * 5 号销售卖出：2
 * 2 号销售卖出：2
 * 4 号销售卖出：3
 * main	共计卖出多少套： 9
 */
```

>remove() 阿里Java开发手册

【强制】必须回收自定义的 ThreadLocal 变量记录的当前线程的值，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成内存泄露等问题。

尽量在代码中使用 **try-finally** 块进行回收。

正例：
```java
objectThreadLocal.set(userInfo);

try {
	// ...
} finally {
	objectThreadLocal.remove();
}
```

---
remove() 线程池案例
```java
class MyData {
    ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

    public void add() {
        threadLocal.set(threadLocal.get() + 1);
    }
}

public class ThreadLocalRemoveDemo {
    public static void main(String[] args) {
        MyData myData = new MyData();

        ExecutorService threadPool = Executors.newFixedThreadPool(3);

        try {
            for (int i = 0; i < 10; i++) {
                threadPool.submit(() -> {
                    try {
                        Integer boforeInt = myData.threadLocal.get();
                        myData.add();
                        Integer afterInt = myData.threadLocal.get();
                        System.out.println(String.format("线程%s ---> beforeInt: %s, afterInt: %s",
                                Thread.currentThread().getName(), boforeInt, afterInt));
                    } finally {
                        myData.threadLocal.remove();
                    }
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}
/**
 * 无 remove()：
 *      线程pool-1-thread-3 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-2 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-1 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-2 ---> beforeInt: 1, afterInt: 2
 *      线程pool-1-thread-1 ---> beforeInt: 1, afterInt: 2
 *      线程pool-1-thread-3 ---> beforeInt: 1, afterInt: 2
 *      线程pool-1-thread-2 ---> beforeInt: 2, afterInt: 3
 *      线程pool-1-thread-3 ---> beforeInt: 2, afterInt: 3
 *      线程pool-1-thread-2 ---> beforeInt: 3, afterInt: 4
 *      线程pool-1-thread-1 ---> beforeInt: 2, afterInt: 3
 *
 * 有 remove()：
 *      线程pool-1-thread-3 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-2 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-1 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-3 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-1 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-2 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-3 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-1 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-2 ---> beforeInt: 0, afterInt: 1
 *      线程pool-1-thread-3 ---> beforeInt: 0, afterInt: 1
 */
```

## ThreadLocal 源码解读

> Thread、ThreadLocal、ThreadLocalMap关系

ThreadLocal本身并不存储值(ThreadLocal是一个壳子)，它只是自己作为一个key来让线程从ThreadLocalMap获取value.正因为这个原理，所以ThreadLocal能够实现“数据隔离”，获取当前线程的局部变量值，不受其他线程影响~

![](assets/Pasted%20image%2020240115132318.png)

每个Thread线程内部都有一个ThreadLocalMap
```java 
//Thread.java
ThreadLocal.ThreadLocalMap threadLocals = null;
```

ThreadLocalMap实际上就是一个以ThreadLocal实例为**key**，任意对象为**value**的Entry对象。当我们为ThreadLocal变量赋值，实际上就是以当前【ThreadLocal实例为key，值为value】的 Entry 往这个ThreadLocalMap中存放

```java
// ThreadLocal.java

// 静态内部类
static class ThreadLocalMap {  
	static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
	    }
	}

	ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {  
	    table = new Entry[INITIAL_CAPACITY];  
	    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);  
	    table[i] = new Entry(firstKey, firstValue);  
	    size = 1;  
	    setThreshold(INITIAL_CAPACITY);  
	}

	private void set(ThreadLocal<?> key, Object value){
		...
	}
}

public T get() {
    Thread t = Thread.currentThread();  
    // 获取当前线程的 threadLocalMaps
    ThreadLocalMap map = getMap(t);
    if (map != null) {  
		// 根据当前ThreadLocal对象取出value
        ThreadLocalMap.Entry e = map.getEntry(this);  
        if (e != null) {  
            @SuppressWarnings("unchecked")  
            T result = (T)e.value;  
            return result;
        }
    }
    // 设置初始值
    return setInitialValue();  
}

public void set(T value) {  
    Thread t = Thread.currentThread();  
    ThreadLocalMap map = getMap(t);  
    if (map != null)  
        map.set(this, value);  
    else  
        createMap(t, value);  
}

ThreadLocalMap getMap(Thread t) {  
    return t.threadLocals;  
}
```

## ThreadLocal 内存泄露问题

###  强软弱虚引用

![](assets/Pasted%20image%2020240115144221.png)

Java 技术允许使用 `finalize() `方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。

> 强引用

对于强引用的对象，==就算是出现了OOM也不会对该对象进行回收，死都不收==。

强引用是我们==最常见的普通对象引用==，只要还有强引用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰这种对象。在Java 中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。

当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾回收机制回收的，即使该对象以后永远都不会被用到，JVM也不会回收。因此强引用是造成Java内存泄漏的主要原因之一。

对于一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者==显式地将相应（强）引用赋值为 null==，一般认为就是可以被垃圾收集的了 (当然具体回收时机还是要看垃圾收集策略)。

案例：
```java
class MyObject {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("----- invoke finalize method!");
    }
}

public class ReferenceDemo {
    public static void main(String[] args) throws InterruptedException {
        MyObject myObject = new MyObject();
        System.out.println("gc before: " + myObject);
        myObject = null;
        System.gc();
        Thread.sleep(1000);
        System.out.println("gc after: " + myObject);
        /**
         * gc before: com.boer.tl.MyObject@2503dbd3
         * ----- invoke finalize method!
         * gc after: null
         */
    }
}

```

> 软引用

软引用是一种相对强引用弱化了一些的引用，需要用 `java.lang.ref.SoftReference` 类来实现，可以让对象豁免一些垃圾收集。

对于只有软引用的对象来说，
- 当系统内存充足时它不会被回收，
- 当系统内存不足时它会被回收。

软引用通常用在对内存敏感的程序中，比如高速缓存就有用到软引用，==内存够用的时候就保留，不够用就回收！==

案例：配置虚拟机参数 `-Xms10m -Xmx10m` 模拟内存不够的情况
```java
class MyObject {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("----- invoke finalize method!");
    }
}

public class ReferenceDemo {
    public static void main(String[] args) throws InterruptedException {
        SoftReference<MyObject> softReference = new SoftReference<MyObject>(new MyObject());
        System.out.println("gc before: " + softReference);

        System.gc();
        Thread.sleep(1000);
        System.out.println("gc after 内存够用: " + softReference.get());

        try {
            byte[] bytes = new byte[20 * 1024 * 1024]; // 20MB对象，让内存溢出
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("gc after 内存不够: " + softReference.get());
        }
        /**
         * gc before: java.lang.ref.SoftReference@2503dbd3
         * gc after 内存够用: com.boer.tl.MyObject@4b67cf4d
         * gc after 内存不够: null
         * ----- invoke finalize method!
         * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
         * 	at com.boer.tl.ReferenceDemo.main(ReferenceDemo.java:28)
         */
    }
}
```

> 弱引用

弱引用需要用`java.lang.ref.WeakReference`类来实现，它比软引用的生存期更短，

对于只有弱引用的对象来说，只要垃圾回收机制一运行，==不管JVM的内存空间是否足够，都会回收该对象占用的内存==。

案例：
```java
class MyObject {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("----- invoke finalize method!");
    }
}

public class ReferenceDemo {
    public static void main(String[] args) throws InterruptedException {
        WeakReference<MyObject> weakReference = new WeakReference<>(new MyObject());
        System.out.println("gc before 内存够用: " + weakReference.get());

        System.gc();
        Thread.sleep(1000);
        System.out.println("gc after 内存够用: " + weakReference.get());
    }
}
```

> 软引用和弱引用的使用场景

假如有一个应用需要读取大量的本地图片: 
- 如果每次读取图片都从硬盘读取则会严重影响性
- 如果一次性全部加载到内存中又可能造成内存溢出。

使用软引用可以解决这个问题。

设计思路是：用一个HashMap来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，从而有效地避免了OOM的问题。

```java
Map<String, SoftReference<Bitmap>> imageCache = new HashMap<String, SoftReference<Bitmap>>();
```

> 虚引用

虚引用需要`java.lang.ref.PhantomReference`类来实现，顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。

如果一个对象仅持有虚引用，那么它就和没有任何引用一样，==在任何时候都可能被垃圾回收器回收。

==它不能单独使用也不能通过它访问对象，虚引用必须和引用队列(ReferenceQueue)联合使用。==

PhantomReference 的 get 方法总是返回 null，因此无法访问对应的引用对象。

虚引用的主要作用是==跟踪对象被垃圾回收的状态==。 仅仅是提供了一种确保对象被 finalize以后，做某些事情的**通知机制**。

换句话说，设置虚引用关联对象的**唯一目的**，就是在这个对象被收集器回收的时候收到一个系统通知或者后续添加进一步的处理，用来实现比finalize机制更灵活的回收操作

案例：
```java
public class ReferenceDemo {
    public static void main(String[] args) {
        ReferenceQueue<MyObject> referenceQueue = new ReferenceQueue<>();
        PhantomReference<MyObject> phantomReference = new PhantomReference<>(new MyObject(), referenceQueue);
//        System.out.println("phantomReference.get(): " + phantomReference.get());

        List<byte[]> list = new ArrayList<>();

        // 设置10MB内存
        new Thread(() -> {
            while (true) {
                list.add(new byte[1024 * 1024]); //1MB
                try {
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("phantomReference.get(): " + phantomReference.get() + "\t list add ok!");
            }
        }, "t1").start();

        new Thread(() -> {
            while (true) {
                Reference<? extends MyObject> reference = referenceQueue.poll();
                if (reference != null) {
                    System.out.println("----- 有虚对象回收加入了队列");
                    break;
                }
            }
        }, "t2").start();
    }
```


### 内存泄露的原因

> 内存泄漏

不再会被使用的对象或者变量占用的==内存不能被回收==，就是内存泄漏

> 原因？

ThreadLocalMap使用 `WeakReference<ThreadLocal<?>>` 将ThreadLocal对象变成一个**弱引用**的对象

![](assets/Pasted%20image%2020240115171701.png)

ThreadLocal本身并不存储值(ThreadLocal是一个壳子)，它只是自己作为一个key来让线程从ThreadLocalMap获取value.正因为这个原理，所以ThreadLocal能够实现“数据隔离”，获取当前线程的局部变量值，不受其他线程影响~

> 为什么要用弱引用？不用如何？

![](assets/Pasted%20image%2020240115173225.png)

当方法执行完毕后，栈帧销毁，强引用t1也就没有了，但此时线程的ThreadLocalMap里某个entry的Key引用还指向这个对象，若这个Key是强引用，就会导致Key指向的ThreadLocal对象即V指向的对象不能被gc回收，造成内存泄露

若这个引用时弱引用就大概率会减少内存泄漏的问题（当然，还得考虑key为null这个坑），使用弱引用就可以使ThreadLocal对象在方法执行完毕后顺利被回收且entry的key引用指向为null

【来自弹幕】thread 和 threadlocalmap 绑定，threadlocal 只是个工具，不能和 threadlocalmap 绑定，所以是弱引用

> 弱引用就万事大吉了吗？

ThreadLocalMap使用ThreadLocal的弱引用作为Key，如果一个ThreadLocal没有被外部强引用，那么系统gc时势必会被回收他，这样一来，ThreadLocalMap中就会出现==Key为null的Entry==。

如果当前线程迟迟不结束的话（好比正在使用线程池），这些 key 为 null 的 Entry 的 value 就会一直存在一条强引用链（当然，如果当前 thread 运行结束，threadLocal，threadLocalMap.Entry 没有引用链可达，在垃圾回收的时候都会被系统进行回收）。

我们调用 get, set 或 remove 方法时，会尝试==删除 key 为 null 的 entry==，可以释放 value 对象所占用的内存。
- 通过 expungeStaleEntry，cleanSomeSlots，replaceStaleEntry 这三个方法清理掉 key 为 null 的脏 entry。

弱引用不能 100% 保证内存不泄露。==我们要在不使用某个 ThreadLocal 对象后，手动调用 remove 方法来删除它==，尤其是在线程池中。
- 不仅仅是内存泄露的问题，
- 因为线程池中的线程是重复使用的，意味着这个线程的 ThreadLocalMap 对象也是重复使用的，如果我们不手动调用 remove 方法，那么后面的线程就有可能获取到上个线程遗留下来的 value 值，造成 bug。

## 最佳实践

- ThreadLocal一定要初始化 `ThreadLocal.withInitial()` ，避免空指针异常。
	- 没有初始值直接 get ()，默认 value 是 null
- 建议把 ThreadLocal 修饰为 static
- 用完记得手动 remove

> 阿里Java开发手册

【参考】==ThreadLocal 对象使用 static 修饰==，ThreadLocal 无法解决共享对象的更新问题。

说明：这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象（只要是这个线程内定义的）都可以操控这个变量。

# Java 对象内存布局和对象头

