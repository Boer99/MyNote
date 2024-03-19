
> 参考：黑马 JUC 教程 2022
> 
> 并发相关Java包
> 
> - `java.util.concurrent`
> - `java.util.concurrent.atomic`
> - `java.util.concurrent.locks`

# ---------- 进程与线程
## 并行与并发

单核cpu下，线程实际还是**串行执行**的。操作系统中有一个组件叫做任务调度器，将 cpu 的时间片（windows 下时间片最小约为 15 毫秒）分给不同的程序使用，只是由于 cpu 在线程间（时间片很短）的切换非常快，人类感觉是同时运行的 。总结为一句话就是：**微观串行，宏观并行** 。


一般会将这种线程轮流使用 CPU 的做法称为并发，**concurrent**

| **CPU** | **时间片 1** | **时间片 2** | **时间片 3** | **时间片 4** |
| ------- | --------- | --------- | --------- | --------- |
| core    | 线程 1      | 线程 2      | 线程 3      | 线程 4      |

多核 cpu下，每个 核（core） 都可以调度运行线程，这时候线程可以是并行的。

| **CPU** | **时间片 1** | **时间片 2** | **时间片 3** | **时间片 4** |
| --- | --- | --- | --- | --- |
| core1 | 线程 1 | 线程 2 | 线程 3 | 线程 4 |
| core2 | 线程 4 | 线程 4 | 线程 2 | 线程 2 |

引用 Rob Pike 的一段描述： 
- 并发（concurrent）是同一时间应对（dealing with）多件事情的能力 。
- 并行（parallel）是同一时间动手做（doing）多件事情的能力。

## 同步与异步

从**方法调用**的角度来说，如果

- 需要**等待结果返回**，才能继续运行就是“同步”
- 不需要等待结果返回，就能继续运行就是“异步”

> 注意：同步在多线程中还有另外一层意思，是让多个线程步调一致

多线程可以让方法执行变为异步的（即不要巴巴干等着）

> - 比如说读取磁盘文件时，假设读取操作花费了 5 秒钟，如果没有线程调度机制，这 5 秒 cpu 什么都做不了，其它代码都得暂停...
> - 比如在项目中，视频文件需要转换格式等操作比较费时，这时开一个新线程处理视频转换，避免阻塞主线程
> - tomcat 的异步 servlet 也是类似的目的，让用户线程处理耗时较长的操作，避免阻塞 tomcat 的工作线程
> - ui 程序中，开线程进行其他操作，避免阻塞 ui 线程

- **单核 cpu** 下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu ，**不至于一个线程总占用 cpu，别的线程没法干活**
- **多核 cpu** 可以**并行跑多个线程**，但能否提高程序运行效率还是要分情况的 
   - 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任务都能拆分（参考后文的【阿姆达尔定律】）
   - 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
- IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO】，这时相当于线程虽然不用 cpu，但需要一 直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO】和【异步 IO】优化。

# ---------- Java 线程

## 创建和运行线程

### 方式一：Thread

方法一：继承 Thread，重写 run()

```java
@Slf4j
public class Demo {
    public static void main(String[] args) {
        // 创建线程对象
        Thread t = new Thread() {
            // 匿名内部类
            @Override
            public void run() {
                // 要执行的任务
                log.debug("running");
            }
        };
        t.setName("t1");
        // 启动线程
        t.start();

        log.debug("running");

        //00:04:20.036 [t1] DEBUG com.boer.java线程.Demo - running
        //00:04:20.036 [main] DEBUG com.boer.java线程.Demo - running
    }
}
```

###  方式二：Runnable 配合 Thread

把【线程】和【任务】（要执行的代码）分开
- Thread 代表线程
- Runnable 可运行的任务（线程要执行的代码）

```java
@Slf4j
public class Demo {
    public static void main(String[] args) {
        Runnable runnable = new Runnable() { // 匿名内部类
            @Override
            public void run() {
                // 要执行的任务
                log.debug("running");
            }
        };
        // 创建线程对象
        Thread t = new Thread(runnable);
        // 启动线程
        t.start();

        // 00:19:04.422 [Thread-0] DEBUG com.boer.java线程.Demo - running
    }
}
```

lambda 精简代码

```java
Runnable task2 = () -> log.debug("hello");
```

### Thread 与 Runnable 的关系

分析 Thread 的源码，理清它与 Runnable 的关系

- Runnable 是函数式接口

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

- Thread 源码：如果 target 不为空，就调用 target 的 Runnable 方法

```java
//Thread源码（部分） 
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;
    
    public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        //...
        this.target = target;
       //...
    }
    
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
}
```

两种方式对比：
- 方法 1 是把线程和任务合并在了一起，方法 2 是把线程和任务分开了 
- 用 Runnable 更容易**与线程池等高级 API 配合** 
- 用 Runnable 让任务类**脱离了 Thread 继承体系，更灵活**

> Java 里组合优于继承，Runnable 和 Thread 就是组合关系

### 方式三：FutureTask 配合 Thread

FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
	// 创建任务对象
	FutureTask<Integer> task = new FutureTask<>(new Callable<Integer>() {
		@Override
		public Integer call() throws Exception {
			log.debug("running...");
			Thread.sleep(2000);
			return 100;
		}
	});
	
	// 传入任务对象，启动线程
	Thread t1 = new Thread(task, "t1");
	t1.start();
	
	// 主线同步阻塞，同步等待task执行完毕
	log.debug("{}",task.get());
}

```

lamba 简化

```java
FutureTask<Integer> task = new FutureTask<>(() -> {
	return 100;
});
```

---
Callable 接口 对比 Runnable 接口，`call()` 方法
- 有返回值
- 方法上抛出了异常

```java
package java.util.concurrent;

@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

### 多个线程同时运行

- 交替执行
- 谁先谁后，不由我们控制

## 查看进程线程的方法

Windows
- 任务管理器可以查看进程和线程数，也可以用来杀死进程
- tasklist 查看进程 
   - `tasklist` | `findstr` (查找关键字)

- taskkill 杀死进程 
   - `taskkill` `/F` (彻底杀死）`/PID` (进程 PID)

Linux
- `ps -fe` 查看所有进程
- `ps -fT -p`  查看某个进程（PID）的所有线程
- `kill` 杀死进程 top 按大写 H 切换是否显示线程
- `top -H -p`  查看某个进程（PID）的所有线程

Java
- `jps` 命令查看所有 Java 进程
- `jstack` 查看某个 Java 进程（PID）的所有线程状态
- `jconsole` 来查看某个 Java 进程中线程的运行情况（图形界面）

## Java 线程运行原理

### 栈与栈帧

> Java Virtual Machine Stacks （Java 虚拟机栈）

JVM 中由堆、栈、方法区所组成，每个线程启动后，虚拟机就会为其分配一块栈内存。
- 每个栈由多个**栈帧**（Frame）组成，对应着每次方法调用时所占用的内存
- 每个线程**只能有一个活动栈帧**，对应着当前**正在执行的那个方法**

### 线程上下文切换

> Thread Context Switch

因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

- 线程的 cpu 时间片用完
- 垃圾回收
- 有更高优先级的线程需要运行

线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法

当 Context Switch 发生时，需要由操作系统保存当前线程的状态，并恢复另一个线程的状态，Java 中对应的概念就是*程序计数器（Program Counter Register）*，它的作用是记住下一条 jvm 指令的执行地址，是**线程私有**的

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
- **Context Switch 频繁发生会影响性能**

## 常用方法

### Thread 类 API

| 方法                                    | 说明                                          |
| ------------------------------------- | ------------------------------------------- |
| public void setName(String name)      | 给当前线程取名字                                    |
| public void getName()                 | 获取当前线程的名字 线程存在默认名称：子线程是 Thread-索引，主线程是 main |
| public static Thread currentThread()  | 获取当前线程对象，代码在哪个线程中执行                         |
| public final native boolean isAlive() | 线程是否存活（还没有运行完毕）                             |

### start() & run()

```java
public synchronized void start()
```

- 功能：启动一个**新线程**，运行 run 方法中的代码
- 注意点：
	- 线程状态：运行 `-->` **`RUNNABLE`**
		- 里面代码**不一定立刻运行**（CPU 的时间片还没分给它）
	- 每个线程对象只能调用一次
		- 调用多次会出现 `IllegalThreadStateException`

```java
public void run()
```

- 功能：新线程**启动后**会调用的方法
- 注意点：
	- 如果在构造 Thread 对象时传递了 Runnable 参数，则线程启动后会**调用 Runnable 中的 run 方法**，否则默认不执行任何操作
	- 可以创建 Thread 的**子类对象，重写 `run()`，来覆盖默认行为**

两者对比
- 直接调用 run 是在主线程中执行了 run，**没有启动新的线程**
- 使用 start 是启动新的线程，通过新的线程间接执行 run 中的代码

### sleep() & yield()

#### sleep

```java
public static native void sleep(long millis) throws InterruptedException;
```

- **static**
- 功能：让当前执行的线程休眠 n 毫秒，休眠时让出 cpu 的时间片给其它线程
- 注意点：
	- 当前线程状态：运行 `-->` **`Timed Waiting` （阻塞）**
	- 其它线程可以使用 `interrupt()` 方法**打断正在睡眠的线程**，这时 sleep 方法会抛出 `InterruptedException`
	- 睡眠结束后的线程未必会立刻得到执行
	- **更推荐用 `TimeUnit` 的 `sleep()`**，来获得更好的可读性
		- 还是调用的 sleep()，做了单位的换算优化

观察线程状态

```java
public static void main(String[] args) throws InterruptedException {
	Thread t1 = new Thread("t1") {
		@Override
		public void run() {
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				throw new RuntimeException(e);
			}
		}
	};

	t1.start();
	// t1 还没执行，RUNNABLE状态
	log.debug("t1 state: {}", t1.getState());
	// 23:25:00.483 [main] DEBUG com.boer.java线程.SleepDemo - t1 state: RUNNABLE

	Thread.sleep(500);

	log.debug("t1 state: {}", t1.getState());
	// 23:25:01.014 [main] DEBUG com.boer.java线程.SleepDemo - t1 state: TIMED_WAITING
}
```

打断正在睡眠的线程

```java
public static void main(String[] args) throws InterruptedException {  
    Thread t1 = new Thread("t1") {  
        @Override  
        public void run() {  
            try {  
                Thread.sleep(2000);  
            } catch (InterruptedException e) {  
                log.debug("wake up...");  
            }  
            log.debug("t1 state: {}", Thread.currentThread().getState());  
        }  
    };  
  
    t1.start();  
  
    Thread.sleep(500);  
  
    log.debug("interrupt");  
    t1.interrupt();  
  
    //23:41:59.658 [main] DEBUG com.boer.java线程.SleepDemo2 - interrupt  
    //23:41:59.667 [t1] DEBUG com.boer.java线程.SleepDemo2 - wake up...  
    //23:41:59.667 [t1] DEBUG com.boer.java线程.SleepDemo2 - t1 state: RUNNABLE  
}
```

#### yield    

```java
public static native void yield();
```

- **static**
- 功能：提示线程调度器让出当前线程对 CPU 的使用
- 注意：
	- 当前线程状态： `Running` `-->` **`Runnable`** （就绪），然后调度执行**其它**线程
	- 具体的实现依赖于操作系统的任务调度器

---
两者区别：
- Runnable 状态下线程还是有可能再次被再次调度（yield）
- yield 没有等待时间

#### 案例——防止CPU占用100%

在没有利用 cpu 来计算时，**不要让 `while(true)` 空转浪费 cpu**，这时可以使用 yield 或 sleep 来让出 cpu 的使用权给其他程序

```java
public static void main(String[] args) throws InterruptedException {  
    new Thread(() -> {  
        while (true) {  
            try {  
                Thread.sleep(50);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }).start();  
}
```

- 可以用 wait 或 条件变量 达到类似的效果。
	- 不同的是，后两种都需要加锁，并且需要相应的唤醒操作，一般适用于要进行**同步的场景**
- sleep 适用于**无需锁同步**的场景

### 线程优先级

> Java 提供一个线程调度器来监控程序中启动后进入**就绪**状态的所有线程

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器**可以忽略**它
- cpu **忙**，优先级高的线程会获得更多的时间片
- cpu 闲，优先级几乎没作用

线程的优先级用数字表示，范围从 1~10，Thread 定义了的线程优先级如下：

```java
public final static int MIN_PRIORITY = 1;

/**
 * The default priority that is assigned to a thread.
 */
public final static int NORM_PRIORITY = 5;

/**
 * The maximum priority that a thread can have.
 */
public final static int MAX_PRIORITY = 10;
```

设置、获取线程的优先级

```java
public final void setPriority(int newPriority)

public final int getPriority()
```

```java
public class PriorityDemo {
    public static void main(String[] args) {
        Runnable task1 = () -> {
            int count = 0;
            for (; ; ) {
                System.out.println("----->1 " + count++);
            }
        };

        Runnable task2 = () -> {
            int count = 0;
            for (; ; ) {
                System.out.println("     ----->2 " + count++);
            }
        };

        Thread t1 = new Thread(task1, "t1");
        Thread t2 = new Thread(task2, "t2");

        t1.setPriority(Thread.MIN_PRIORITY);
        t2.setPriority(Thread.MAX_PRIORITY);
        t1.start();
        t2.start();
    }
}
```

### join()

> 线程同步的概念： [同步与异步](#同步与异步)

> 当一个线程需要等待另一个线程的结果，sleep() 不知道另一个线程还要执行多久 ×
> 
> 下面的案例要获取到 r=10 怎么办？用 join()
> 
> ```java
> public class JoinDemo {
>     static int r = 0;
> 
>     public static void main(String[] args) {
>         log.debug("开始");
>         Thread t1 = new Thread(() -> {
>             log.debug("开始");
>             try {
>                 Thread.sleep(1000);
>             } catch (InterruptedException e) {
>                 throw new RuntimeException(e);
>             }
>             log.debug("结束");
>             r = 10;
>         });
>         t1.start();
>         // 解决方案：t1.join()
>         log.debug("结果为:{}", r);
>         log.debug("结束");
> 
>         //01:04:16.056 [main] DEBUG com.boer.java线程.JoinDemo - 开始
>         //01:04:16.095 [Thread-0] DEBUG com.boer.java线程.JoinDemo - 开始
>         //01:04:16.095 [main] DEBUG com.boer.java线程.JoinDemo - 结果为:0
>         //01:04:16.096 [main] DEBUG com.boer.java线程.JoinDemo - 结束
>         //01:04:17.104 [Thread-0] DEBUG com.boer.java线程.JoinDemo - 结束
>     }
> }
> ```

```java
public final void join() throws InterruptedException
```

- 功能：调用此方法的线程被阻塞，仅当该方法完成以后，才能继续运行
- 线程状态：运行 `-->` **`WAITING`**

```java
public final synchronized void join(long millis) 
	throws InterruptedException

public final synchronized void join(long millis, int nanos)
    throws InterruptedException
```

- 功能：有时间限制的等待

> join 可以想成合并线程

等待多个结果

```java
@Slf4j
public class JoinDemo2 {
    static int r1 = 0;
    static int r2 = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            r1 = 10;
        }, "t1");
        Thread t2 = new Thread(() -> {
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            r1 = 10;
        }, "t2");

        long start = System.currentTimeMillis();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        long end = System.currentTimeMillis();
        log.debug("r1:{} r2:{} cost{}", r1, r2, end - start);
        // 19:25:04.314 [main] DEBUG com.boer.java线程.JoinDemo2 - r1:10 r2:0 cost2002
    }
}
```

> 分析：t1、t2 异步执行，主线程先等待 t1 执行完，再等待 t2 执行完，获取两个结果，总共耗时 2s

### interrupt()

```java
public void interrupt()
```

- 功能：打断线程
- 注意：
	- 打断线程正在 sleep，wait，join ：
		- 被打断的线程会 **抛出** `InterruptedException`
		- **清除** 打断标记
	- 打断 running 线程：设置 打断标记
	- 打断 park 的线程：设置 打断标记

```java
public boolean isInterrupted()
```

- 功能：判断线程是否被打断

```java
public static boolean interrupted()
```

- 功能：判断当前线程是否被打断，并**清除**打断标记

---
> 【案例】打断 sleep 中的线程

```java
public static void main(String[] args) {  
    Thread t1 = new Thread(() -> {  
        sleep(1000);  
    }, "t1");  
  
    t1.start();  
    sleep(500);  
    t1.interrupt();  
    log.debug(" 打断状态: {}", t1.isInterrupted());  
    // 20:55:02.645 [main] DEBUG com.boer.java线程.InterruptedDemo -  打断状态: false  
}
```

### 不推荐的方法

这些方法已过时，容易**破坏同步代码块**，造成线程死锁

- `stop()`：停止线程运行
- `suspend()`：挂起（暂停）线程运行
- `resume()`：恢复线程运行

## 主线程与守护线程

> 默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。

守护线程：只要其它非守护线程运行结束，即使守护线程的代码**没有执行完，也会强制结束**
- 例如：**垃圾回收器线程**

> Tomcat 中的 Acceptor 和 Poller 线程都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等待它们处理完当前请求

```java
public static void main(String[] args) {  
    log.debug("开始运行...");  
  
    Thread t1 = new Thread(() -> {  
        log.debug("开始运行...");  
        sleep(10);  
        log.debug("运行结束...");  
    }, "daemon");  
  
    // 设置该线程为守护线程  
    t1.setDaemon(true);  
    t1.start();  
  
    sleep(1);  
    log.debug("运行结束...");  
    //21:50:29.889 [main] DEBUG com.boer.java线程.DaemonDemo - 开始运行...  
    //21:50:29.930 [daemon] DEBUG com.boer.java线程.DaemonDemo - 开始运行...  
    //21:50:30.936 [main] DEBUG com.boer.java线程.DaemonDemo - 运行结束...  
}
```

## 线程状态

### 五种状态

从 操作系统 层面来描述
- 【初始状态】仅是在**语言层面**创建了线程对象，还未与操作系统线程关联
- 【可运行状态】（就绪状态）指该线程已经被创建（与操作系统线程关联），可以由 CPU 调度执行
- 【运行状态】指获取了 CPU 时间片运行中的状态
	- 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换
- 【阻塞状态】
	- 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际**不会用到 CPU**，会导致线程上下文切换，进入【阻塞状态】
	- 等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】
	- 与【可运行状态】的区别是，对【阻塞状态】的线程来说**只要它们一直不唤醒，调度器就一直不会考虑调度它们**
- 【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态

![|500](assets/Pasted%20image%2020240303215817.png)

### 六种状态

从 Java API 层面来描述，根据 `Thread.State` 枚举类，分为六种状态

```java
public enum State {
	NEW,
	RUNNABLE,
	BLOCKED,
	WAITING,
	TIMED_WAITING,
	TERMINATED;
}
```

- `NEW`：线程刚被创建，但是还**没有调用 start() 方法**
- `RUNNABLE`：当调用了 start() 方法之后
	- 注意，Java API 层面的 `RUNNABLE` 状态**涵盖了 操作系统 层面**的【可运行状态】、【运行状态】和【阻塞状态】（由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为是可运行）
- `BLOCKED` ， `WAITING` ， `TIMED_WAITING` 都是 Java API 层面**对【阻塞状态】的细分**
- `TERMINATED` 当线程代码运行结束

> #todo `BLOCKED` ， `WAITING` ， `TIMED_WAITING` 详细介绍见后续章节

![|500](assets/Pasted%20image%2020240303220915.png)

> 6 种状态演示

```java
public static void main(String[] args) throws IOException {  
    Thread t1 = new Thread("t1") {  
        @Override  
        public void run() {  
            log.debug("running...");  
        }  
    }; // NEW  
  
    Thread t2 = new Thread("t2") {  
        @Override  
        public void run() {  
            while(true) { // RUNNABLE  
  
            }  
        }  
    };  
    t2.start();  
  
    Thread t3 = new Thread("t3") {  
        @Override  
        public void run() {  
            log.debug("running...");  
        }  
    };  
    t3.start(); // TERMINATED  
  
    Thread t4 = new Thread("t4") {  
        @Override  
        public void run() {  
            synchronized (TestState.class) {  
                try {  
                    Thread.sleep(1000000); // TIMED_WAITING  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
    };  
    t4.start();  
  
    Thread t5 = new Thread("t5") {  
        @Override  
        public void run() {  
            try {  
                t2.join(); // WAITING  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    };  
    t5.start();  
  
    Thread t6 = new Thread("t6") {  
        @Override  
        public void run() {  
            // t4先拿到了锁，t6拿不到了  
            synchronized (TestState.class) { // blocked  
                try {  
                    Thread.sleep(1000000);  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                }  
            }  
        }  
    };  
    t6.start();  
  
    try {  
        Thread.sleep(500);  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
    log.debug("t1 state {}", t1.getState());  
    log.debug("t2 state {}", t2.getState());  
    log.debug("t3 state {}", t3.getState());  
    log.debug("t4 state {}", t4.getState());  
    log.debug("t5 state {}", t5.getState());  
    log.debug("t6 state {}", t6.getState());  
    //23:08:34.039 c.TestState [t3] - running...  
	//23:08:34.551 c.TestState [main] - t1 state NEW  
	//23:08:34.552 c.TestState [main] - t2 state RUNNABLE  
	//23:08:34.552 c.TestState [main] - t3 state TERMINATED  
	//23:08:34.552 c.TestState [main] - t4 state TIMED_WAITING  
	//23:08:34.552 c.TestState [main] - t5 state WAITING  
	//23:08:34.552 c.TestState [main] - t6 state BLOCKED
}
```

# ---------- 共享模型_管程

## 共享带来的问题

### ++案例

两个线程对初始值为 0 的静态变量一个做自增，一个做自减，各做 5000 次，结果是 0 吗？

```java
@Slf4j
public class Demo1 {
    static int counter = 0;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                counter++;
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                counter--;
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("counter:{}", counter);
        // 23:45:31.175 [main] DEBUG cn.itcast.n4.Demo1 - counter:-611
    }
}
```

Java 中对静态变量的自增，自减并不是原子操作

`i++` 和 `i--` 的字节码指令：

```java
getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
iadd // 自增
putstatic i // 将修改后的值存入静态变量i

getstatic i // 获取静态变量i的值
iconst_1 // 准备常量1
isub // 自减
putstatic i // 将修改后的值存入静态变量i
```

Java 的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换

![|500](assets/Pasted%20image%2020240303234728.png)

出现负数的情况

![|500](assets/Pasted%20image%2020240303234824.png)

### 临界区 Critical Section

> 一个程序运行多个线程本身是没有问题的，问题出在多个线程访问共享资源
> 
> - 多个线程读共享资源其实也没有问题，
> - 在多个线程对共享资源读写操作时发生**指令交错**，就会出现问题

临界区：一段代码块内如果存在**对共享资源的多线程读写操作**，称这段代码块为临界区  

```java
static int counter = 0;

static void increment() 
// 临界区
{ 
 counter++;
}

static void decrement() 
// 临界区
{ 
 counter--;
}
```

### 竞态条件 Race Condition

多个线程在临界区内执行，由于**代码的执行序列不同**而导致**结果无法预测**，称之为发生了“竞态条件”

## synchronized

> 为了**避免临界区的竞态条件发生**，有多种手段可以达到目的。
> 
> - **阻塞式**的解决方案：synchronized，Lock
> - **非阻塞式**的解决方案：原子变量

synchronized：即俗称的【对象锁】
- 采用**互斥**的方式让同一时刻至多只有一个线程能持有【对象锁】，其它线程再想获取这个【对象锁】时就会**阻塞**住。
	- 用对象锁保证了临界区内代码的**原子性**
- 这样就能保证拥有锁的线程可以**安全地执行临界区内的代码**，不用担心线程上下文切换

> 注意：虽然 java 中互斥和同步都可以采用 synchronized 关键字来完成，但它们还是有区别的：
> - 互斥是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区代码
> - 同步是由于线程执行的先后、顺序不同、需要一个线程等待其它线程运行到某个点

### 同步块

```java
synchronized(对象) // 线程1， 线程2(blocked)
{
	临界区
}
```

> 对“++案例”修改

```java
@Slf4j  
public class CounterDemo {  
    static int counter = 0;  
    static final Object room = new Object();  
  
    public static void main(String[] args) throws InterruptedException {  
        Thread t1 = new Thread(() -> {  
            for (int i = 0; i < 5000; i++) {  
                synchronized (room) {  
                    counter++;  
                }  
            }  
        }, "t1");  
  
        Thread t2 = new Thread(() -> {  
            for (int i = 0; i < 5000; i++) {  
                synchronized (room) {  
                    counter--;  
                }  
            }  
        }, "t2");  
  
        t1.start();  
        t2.start();  
        t1.join();  
        t2.join();  
        log.debug("{}", counter);  
    }  
}
```

![|500](assets/Pasted%20image%2020240304003916.png)

> 思考下面的问题：
> - 如果把 `synchronized(obj)` 放在 for 循环的外面，如何理解？
> 	- 保证临界区的安全执行，但没意义，等于线程间串行执行
> - 如果 t1 `synchronized(obj1)` 而 t2 `synchronized(obj2)` 会怎样运作？
> 	- 无法保证临界区的安全执行，要对同一个对象加锁
> - 如果 t1 `synchronized(obj)` 而 t2 没有加会怎么样？如何理解？
> 	- 无法保证临界区的安全执行，每个线程都要加锁

### 同步方法

1）普通同步方法
- 锁的是 this 对象

```java
class Test{
	public synchronized void test() {
	
	}
}

// 等价于

class Test{
	public void test() {
		synchronized(this) {
		
		}
	}
}
```

2）静态同步方法
- 锁的是 当前类的 class 对象

```java
class Test{
	public synchronized static void test() {
	}
}

// 等价于

class Test{
	public static void test() {
		synchronized(Test.class) {
		}
	}
}
```

### 线程 8 锁

其实就是考察 synchronized 锁住的是哪个对象？

> #todo 

## 共享资源类

把需要保护的**共享变量放入一个类**

```java
@Slf4j
public class Test17 {
    public static void main(String[] args) throws InterruptedException {
        Room room = new Room();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                room.increment();
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                room.decrement();
            }
        }, "t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("{}", room.getCounter());
    }
}

// 资源类
class Room {
    private int counter = 0;

    public synchronized void increment() {
        counter++;
    }

    public synchronized void decrement() {
        counter--;
    }

    public synchronized int getCounter() {
        return counter;
    }
}
```

## 变量的线程安全分析

“成员变量”和“静态变量”是否线程安全？
- 没有被共享，则线程安全
- 被**共享**了，
	- 只有读操作，则线程安全
	- 有**读写**操作，则这段代码是临界区，需要考虑线程安全

“局部变量”是否线程安全？
- 本身是线程安全的（存在栈上，共享不了）
- 但其**引用的对象则未必**（堆上就可能被共享），要看该对象是否**逃离方法的作用范围**
	- 未逃离：线程安全
	- **逃离**：需要考虑线程安全

> #Boer 总结：先看变量是否被 多线程共享，再看是否 读写

### 非引用类型局部变量

> 演示：非引用类型局部变量，肯定线程安全

```java
public static void test1() {
	int i = 10;
	i++;
}
```

分析：每个线程调用 `test1()` 方法时，局部变量 i 会在**每个线程的栈帧**内存中被创建多份，因此**不存在共享**

![|400](assets/Pasted%20image%2020240305004636.png)

### 引用类型成员变量

> 演示：**共享的**引用类型成员变量，多线程对其读写，线程不安全

```java
class ThreadUnsafe {
	// 共享的成语变量
    ArrayList<String> list = new ArrayList<>();

    public void method1(int loopNumber) {
        for (int i = 0; i < loopNumber; i++) {
            method2();
            method3();
        }
    }

    private void method2() {
        list.add("1");
    }

    private void method3() {
        list.remove(0);
    }

	static final int THREAD_NUMBER = 2;
    static final int LOOP_NUMBER = 200;

    public static void main(String[] args) {
        ThreadSafeSubClass test = new ThreadSafeSubClass();
        for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(() -> {
                test.method1(LOOP_NUMBER);
            }, "Thread" + (i + 1)).start();
        }
    }
}
```

分析：如果线程 2 还未 add，线程 1 remove 就会报错。

![|500](assets/Pasted%20image%2020240305005034.png)

### 引用类型局部变量

> 先演示个**不共享**的

```java
class ThreadSafe {
    public final void method1(int loopNumber) {
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < loopNumber; i++) {
            method2(list);
            method3(list);
        }
    }

    private void method2(ArrayList<String> list) {
        list.add("1");
    }

    private void method3(ArrayList<String> list) {
        System.out.println(1);
        list.remove(0);
    }

	static final int THREAD_NUMBER = 2;
    static final int LOOP_NUMBER = 200;

    public static void main(String[] args) {
        ThreadSafeSubClass test = new ThreadSafeSubClass();
        for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(() -> {
                test.method1(LOOP_NUMBER);
            }, "Thread" + (i + 1)).start();
        }
    }
}
```

没有共享：
- list 是局部变量，每个线程调用时会创建其不同实例
- method2、method3 的参数与 method1 中引用同一个对象

---
> 演示**共享的**

把 method2 和 method3 修改为 public
- 情况 1：有其它线程调用 method2 和 method3。
	- 线程安全
- 情况 2：在 情况 1 的基础上，为 ThreadSafe 类添加子类，重写 method2 或 method3 方法中，开启新线程操作 list，出现了其他线程共享 list
	- 线程**不安全**

> 情况 2 演示：
> 
> 从这个例子可以看出 private 或 final 提供【安全】的意义所在，请体会开闭原则中的【闭】

```java
class ThreadSafeSubClass extends ThreadSafe {
    @Override
    public void method3(ArrayList<String> list) {
        new Thread(() -> {
            list.remove(0);
        }).start();
    }
    
	static final int THREAD_NUMBER = 2;
    static final int LOOP_NUMBER = 200;

    public static void main(String[] args) {
        ThreadSafeSubClass test = new ThreadSafeSubClass();
        for (int i = 0; i < THREAD_NUMBER; i++) {
            new Thread(() -> {
                test.method1(LOOP_NUMBER);
            }, "Thread" + (i + 1)).start();
        }
    }
}
```

报错：

```java
Exception in thread "Thread-3816" java.lang.IndexOutOfBoundsException: Index: 0, Size: 0
```

> `THREAD_NUMBER` 调大一些更容易报错。
> 
> #todo 报错原因分析：目前认为是 add、remove 操作都不是原子的。

## 常见线程安全类

- `String`
- 包装类，例如 `Integer`
- `StringBuffer`
- `Random`
- `Vector`：线程安全的 List 实现
- `Hashtable`：线程安全的 Map 实现
- `java.util.concurrent` 包下的类

此处线程安全：多个线程调用它们**同一个实例**的某个方法时，是线程安全的

> 理解为如下代码：

```java
Hashtable table = new Hashtable();

new Thread(()->{
	table.put("key", "value1");
}).start();

new Thread(()->{
	table.put("key", "value2");
}).start();
```

### 线程安全类方法的组合

这些类的每个方法是**原子的**，但它们多个方法的**组合不是原子的**

> #Bo 加锁的时候要把读写操作都包含进来

```java
Hashtable table = new Hashtable();
// 线程1，线程2
if( table.get("key") == null) {
	table.put("key", value);
}
```

![|500](assets/Pasted%20image%2020240306005256.png)

### 不可变类线程安全性

> （摘自 Java 工具宝典）不可变类，简单来说就是其实例无法被修改

String、Integer 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是**线程安全**的

String 有 `replace()`、`substring()` 等方法【可以】改变值，这些方法怎么保证线程安全？

### 实例分析

> #todo 

### 购票问题

> 超卖问题

```java
@Slf4j()
public class ExerciseSell {
    public static void main(String[] args) throws InterruptedException {
        // 模拟多人买票
        TicketWindow window = new TicketWindow(1000);

        // 所有线程的集合
        List<Thread> threadList = new ArrayList<>();
        // 卖出的票数统计，Vector是线程安全类
        List<Integer> amountList = new Vector<>();
        for (int i = 0; i < 2000; i++) {
            Thread thread = new Thread(() -> {
                // 买票
                int amount = window.sell(random(5));
                // int amount = window.safeSell(random(5));

                // 统计买票数
                amountList.add(amount);
            });
            threadList.add(thread);
            thread.start();
        }

        for (Thread thread : threadList) {
            thread.join();
        }

        // 统计卖出的票数和剩余票数
        log.debug("余票：{}",window.getCount());
        log.debug("卖出的票数：{}", amountList.stream().mapToInt(i-> i).sum());
    }

    // Random 为线程安全
    static Random random = new Random();

    // 随机 1~5
    public static int random(int amount) {
        return random.nextInt(amount) + 1;
    }
}

// 售票窗口
class TicketWindow {
    private int count;

    public TicketWindow(int count) {
        this.count = count;
    }

    // 获取余票数量
    public int getCount() {
        return count;
    }

	// 售票
    public int sell(int amount)   {
        if (this.count >= amount) {
            // 模拟卖票延时
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            this.count -= amount;
            return amount;
        } else {
            return 0;
        }
    }
}

23:21:19.727 [main] DEBUG c.ExerciseSell - 余票：-74
23:21:19.741 [main] DEBUG c.ExerciseSell - 卖出的票数：1103
```

> 这里要用线程安全的 Vector 记录购票数

资源类 TicketWindow 唯一，其中 sell 方法是临界区，多线程共享读写 count 变量

解决方法：sell 修改为同步方法

```java
// 售票
public syncronized int sell(int amount)   {
	if (this.count >= amount) {
		// 模拟卖票延时
		try {
			Thread.sleep(1);
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		this.count -= amount;
		return amount;
	} else {
		return 0;
	}
}
```

### 转账问题

```java
@Slf4j()
public class ExerciseTransfer {
    public static void main(String[] args) throws InterruptedException {
        Account a = new Account(1000);
        Account b = new Account(1000);

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                a.transfer(b, randomAmount());
            }
        }, "t1");
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                b.transfer(a, randomAmount());
            }
        }, "t2");
        t1.start();
        t2.start();
        t1.join();
        t2.join();

        // 查看转账2000次后的总金额
        log.debug("total:{}", (a.getMoney() + b.getMoney()));
    }

    // Random 为线程安全
    static Random random = new Random();

    // 随机 1~100
    public static int randomAmount() {
        return random.nextInt(100) + 1;
    }
}

// 账户
@Data
class Account {
    // 余额
    private int money;

    public Account(int money) {
        this.money = money;
    }

    // 转账
    public void transfer(Account target, int amount) {
        if (this.money >= amount) {
            this.setMoney(this.getMoney() - amount);
            target.setMoney(target.getMoney() + amount);
        }
    }
}

结果：钱少了或者多了
23:56:16.014 [main] DEBUG c.ExerciseTransfer - total:4776
```

`transfer()` 是临界区，账户 a、b 的 money 都是共享变量

1）将 `transfer()` 修改为同步方法
- 不可行，将 a 作为锁住的对象，`b.transfer()` 还是可以给 a 转账

```java
// 转账
public synchronized void transfer(Account target, int amount) {
	if (this.money >= amount) {
		this.setMoney(this.getMoney() - amount);
		target.setMoney(target.getMoney() + amount);
	}
}
```

2）用类锁
- 可行，只有一个线程能转账
- 这么锁效率不好。还有别的账户的话，a 给 b 转，c 就不能给 d 转了 

```java
// 转账
public void safeTransfer(Account target, int amount) {
	synchronized (Account.class) {
		if (this.money >= amount) {
			this.setMoney(this.getMoney() - amount);
			target.setMoney(target.getMoney() + amount);
		}
	}
}
```

## Monitor

### Java 对象头

> 以 32 位虚拟机为例

普通对象

```
|--------------------------------------------------------------|
|                   Object Header (64 bits)                    |
|------------------------------------|-------------------------|
|           Mark Word (32 bits)      |   Klass Word (32 bits)  |
|------------------------------------|-------------------------|
```

数组对象

![|600](assets/Pasted%20image%2020240307005129.png)

其中 Mark Word 结构为

1）32 bit

![|500](assets/Pasted%20image%2020240307005142.png)

2）64 bit

![|600](assets/Pasted%20image%2020240307005219.png)

### Monitor

> Monitor 被翻译为监视器或管程

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置指向 Monitor 对象的指针

![](assets/Pasted%20image%2020240307135941.png)

工作流程：
- 刚开始 Monitor 中 Owner 为 null
- 当 Thread-2 执行 `synchronized(obj)` 就会将 Monitor 的所有者 `Owner` 置为 Thread-2
	- Monitor 中只能有一个 Owner
- 在 Thread-2 上锁的过程中，如果 其他线程 也来执行 `synchronized(obj)`，就会进入 EntryList
	- 状态：`BLOCKED`
- Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争是**非公平**的
- WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件**不满足**进入 `WAITING` 状态的线程

注意：
- synchronized 必须是进入**同一个**对象的 monitor 才有上述的效果
- 不加 synchronized 的对象不会关联监视器，不遵从以上规则

## synchronized 进阶

### 原理

见八股

### 锁升级

#todo 

## wait() & notify()

### 原理

【**Owner 线程**】发现条件不满足，调用 `wait()`，进入 WaitSet，变为 `[WAITING]` 状态

- `[BLOCKED]` 和 `[WAITING]` 的线程都处于【阻塞】状态，**不占用** CPU 时间片
- `[BLOCKED]` 线程会在 【Owner 线程】**释放锁**时唤醒
- `[WAITING]` 线程会在 【Owner 线程】调用 **`notify()` 或 `notifyAll()`** 时唤醒，
	- 唤醒后并不意味者立刻获得锁，仍需进入 EntryList **重新竞争**

> Owner 线程说明已经 syncronized

![](assets/Pasted%20image%2020240307135941.png)

### API

> 以下都是线程之间进行协作（通信）的手段

前提：必须**获得此对象的锁**（Owner 线程），才能调用这几个方法

都属于 **Object** 对象的方法：

```java
public final void wait() throws InterruptedException

public final native void wait(long timeout) throws InterruptedException

public final native void notify();

public final native void notifyAll();
```

- `wait()` 让进入 object 监视器的线程（Owner 线程）到 waitSet 等待
	- `wait(long n)` **有时限**的等待, 到 n 毫秒后结束等待，或是被 notify
- `notify()` 在 object 上正在 waitSet 等待的线程中**挑一个唤醒**
- `notifyAll()` 让 object 上正在 waitSet 等待的线程**全部唤醒**

> 演示：`wait(long n)` 和 `notify()`

```java
@Slf4j(topic = "c.TestWaitNotify")
public class TestWaitNotify {
    final static Object obj = new Object();

    public static void main(String[] args) {
        Runnable task = () -> {
            synchronized (obj) {
                log.debug("执行....");
                try {
                    obj.wait(1000); // 让线程在obj上一直等待下去
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.debug("其它代码....");
            }
        };

        new Thread(task, "t1").start();
        new Thread(task, "t2").start();

        // 主线程两秒后执行
        sleep(2);
        log.debug("唤醒 obj 上其它线程");
        synchronized (obj) {
//            obj.notify(); // 唤醒obj上一个线程
            obj.notifyAll(); // 唤醒obj上所有等待线程
        }
    }
}
```

### sleep(long n) 和 wait(long n) 的区别

见八股

### 正确姿势

```java
synchronized(lock) {
	while(条件不成立) {
		lock.wait();
	}
	// 干活
}

//另一个线程
synchronized(lock) {
	lock.notifyAll();
}
```

### join()原理

join()通过 wait() 实现，体现的是【保护性暂停】模式

> 【保护性暂停】模式：[超时版 GuardedObject 实现](#超时版%20GuardedObject%20实现)

```java
t1.join();

// 等价于下面的代码

synchronized (t1) {
	// 调用者线程进入 t1 的 waitSet 等待, 直到 t1 运行结束
	while (t1.isAlive()) {
		t1.wait(0);
	}
}
```

> join()的源码

```java
public final void join() throws InterruptedException {  
    join(0);  
}

public final synchronized void join(long millis)  
throws InterruptedException {  
    long base = System.currentTimeMillis();  
    long now = 0;  

    if (millis < 0) {  
        throw new IllegalArgumentException("timeout value is negative");  
    }  

    if (millis == 0) {  
	    // 要等待的线程存活就无限等待
        while (isAlive()) {  
            wait(0);  
        }  
    } else {  
        while (isAlive()) {
            long delay = millis - now;  
            if (delay <= 0) {  
                break;  
            }  
            wait(delay);  
            now = System.currentTimeMillis() - base;  
        }  
    }  
}
```

## park() & unpark()

它们是 LockSupport 类中的方法

```java
// 暂停当前线程
LockSupport.park(); 
// 恢复某个线程的运行
LockSupport.unpark(暂停线程对象)
```

特点（与 wait、notify 相比）：
- 不必配合 Object Monitor 一起使用 
- **以线程为单位**来【阻塞】和【唤醒】线程
	- notify、notifyall 只能随机或全部唤醒等待线程
- 可以先 unpark，再 park
	- 不能先 notify 再 wait
- #Bo unpark 是要指定线程的，而 notify、notifyAll 不需要

> 演示：先 park 后 unpark

```java
public static void main(){
	Thread t1 = new Thread(() -> {
		log.debug("start...");
		sleep(1);
		log.debug("park...");
		LockSupport.park();
		log.debug("resume...");
	},"t1");
	t1.start();
	sleep(2);
	
	log.debug("unpark...");
	LockSupport.unpark(t1);
}

18:42:52.585 c.TestParkUnpark [t1] - start... 
18:42:53.589 c.TestParkUnpark [t1] - park... 
18:42:54.583 c.TestParkUnpark [main] - unpark... 
18:42:54.583 c.TestParkUnpark [t1] - resume...
```

> 演示：先 unpark 后 park，park后直接继续运行

```java
public static void main(){
	Thread t1 = new Thread(() -> {
		log.debug("start...");
		sleep(2);
		log.debug("park...");
		LockSupport.park();
		log.debug("resume...");
	},"t1");
	t1.start();
	sleep(1);
	
	log.debug("unpark...");
	LockSupport.unpark(t1);
}

18:43:50.765 c.TestParkUnpark [t1] - start... 
18:43:51.764 c.TestParkUnpark [main] - unpark... 
18:43:52.769 c.TestParkUnpark [t1] - park... 
18:43:52.769 c.TestParkUnpark [t1] - resume...
```

### 原理

#todo 

## 线程状态转换

![|500](assets/Pasted%20image%2020240303220915.png)

> t：t线程
> curT：当前线程

1）NEW --> RUNNABLE
- `t.start()` 

---
RUNNABLE <---> WAITING

2）t 用 `synchronized(obj)` 获取了对象锁之后

- `obj.wait()`：【RUNNABLE --> WAITING】
- `obj.notify()，obj.notifyAll()，t.interrupt()`
	- 竞争锁成功：t 从 【WAITING --> RUNNABLE】
	- 竞争锁失败：t 从 【WAITING --> BLOCKED】

3）

- curT 调用 `t.join()`：curT 从 【RUNNABLE --> WAITING】
	- 注意是 curT 在 t 线程对象的监视器上等待
- t 运行结束 或 调用了 curT 的 `interrupt()` 时，curT 从 【WAITING --> RUNNABLE】

4）

- curT 调用 `LockSupport.park()`：curT从 【RUNNABLE --> WAITING】
- `LockSupport.unpark(t)` 或 `t.interrupt()`：curT 从【WAITING --> RUNNABLE】

---
RUNNABLE <---> TIMED_WAITING

5）t 用 `synchronized(obj)` 获取了对象锁后

- t 调用 `obj.wait(long n)`：t 从 【RUNNABLE --> TIMED_WAITING】
- t 等待时间超过，或调用 `obj.notify()，obj.notifyAll()，t.interrupt()` 时
	- 竞争锁成功，t 从 【TIMED_WAITING --> RUNNABLE】
	- 竞争锁失败，t 从 【TIMED_WAITING --> BLOCKED】

6）

- curT 调用 `t.join(long n)`：curT 从 【RUNNABLE --> TIMED_WAITING】
	- 注意是 curT 在 t 线程对象的监视器上等待
- curT 等待时间超过，或 t 运行结束，或调用了 curT 的 `interrupt()`：curT 从【TIMED_WAITING --> RUNNABLE】

7）

- curT 调用 `Thread.sleep(long n)`：
	- curT 从【RUNNABLE --> TIMED_WAITING】
- curT 等待时间超过：
	- curT 从【TIMED_WAITING --> RUNNABLE】

8）

- curT 调用 `LockSupport.parkNanos(long nanos)` 或 `LockSupport.parkUntil(long millis)`：
	- curT 从【RUNNABLE --> TIMED_WAITING】
- `LockSupport.unpark(t)` 或 `t.interrupt()`，或是等待超时：
	- t 从【TIMED_WAITING--> RUNNABLE】

---
RUNNABLE <---> BLOCKED

9）

- t 用 `synchronized(obj)` 获取了对象锁时，如果竞争失败，
	- t 从【RUNNABLE --> BLOCKED】
- owner 线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED 的线程重新竞争，如果其中 t 竞争成功，
	- t 从 【BLOCKED --> RUNNABLE】
	- 其它失败的线程仍然 BLOCKED

---
RUNNABLE <--> TERMINATED

10）当前线程所有代码运行完毕，进入 TERMINATED

## 线程活跃性

> 活跃性：由于某些外部原因导致线程一直执行不完，一直处于活跃状态。
> 
> 死锁、活锁、饥饿 是活跃性的三种情况

### 多把锁

> 场景：
> 
> - 一间大屋子有两个功能：睡觉、学习，**互不相干**。
> - 现在小南要学习，小女要睡觉，但如果只用一间屋子（**一个对象锁**）的话，那么并发度很低
> - 解决方法是准备多个房间（**多个对象锁**）

将锁的粒度细分

- 好处，是可以增强并发度
- 坏处，如果**一个**线程需要**同时**获得**多把锁**，就容易发生**死锁**

```java
@Slf4j(topic = "c.BigRoom")
class BigRoom {
    private final Object studyRoom = new Object();
    private final Object bedRoom = new Object();

    public void sleep() {
        synchronized (bedRoom) {
            log.debug("sleeping 2 小时");
            Sleeper.sleep(2);
        }
    }

    public void study() {
        synchronized (studyRoom) {
            log.debug("study 1 小时");
            Sleeper.sleep(1);
        }
    }

	public static void main(String[] args) {
        BigRoom bigRoom = new BigRoom();
        
        new Thread(() -> {
            bigRoom.study();
        },"小南").start();
        
        new Thread(() -> {
            bigRoom.sleep();
        },"小女").start();
    }
}
```

### 死锁

> 见八股

### 哲学家就餐

#todo 

### 活锁

活锁出现在两个线程**互相改变对方的结束条件**，最后谁也无法结束

```java
public class TestLiveLock {
    static volatile int count = 10;
    static final Object lock = new Object();
    
    public static void main(String[] args) {
        new Thread(() -> {
            // 期望减到 0 退出循环
            while (count > 0) {
                sleep(0.2);
                count--;
                log.debug("count: {}", count);
            }
        }, "t1").start();
        
        new Thread(() -> {
            // 期望超过 20 退出循环
            while (count < 20) {
                sleep(0.2);
                count++;
                log.debug("count: {}", count);
            }
        }, "t2").start();
    }
}
```

### 饥饿

很多教程中把饥饿定义为，一个线程由于优先级太低，始终得不到 CPU 调度执行，也不能够结束，饥饿的情况不易演示，讲读写锁时会涉及饥饿问题

下面我讲一下我遇到的一个线程饥饿的例子，先来看看使用顺序加锁的方式解决之前的死锁问题

#todo 

## ReentrantLock

特点：
- 与 synchronized 一样，都支持**可重入**
- 相对于 synchronized 它具备如下特点
	- 等待可中断
	- 可设置超时时间
	- 可设置为公平锁
	- 支持多个条件变量

基本语法：

```java
Lock lock = new ReentrantLock();

// 获取锁
lock.lock();
try {
	// 临界区
} finally {
	// 释放锁
	lock.unlock();
}
```

- 加锁次数和释放锁次数要一致

### 可重入

> 概念见八股

```java
public class ReEntryLockDemo {
    public static void main(String[] args) {
        // reEntryM1();
        reEntryM2();
    }

    // 显示可重入锁
    private static void reEntryM2() {
        final Lock lock = new ReentrantLock();
        new Thread(() -> {
            lock.lock();
            try {
                System.out.println("---外层调用");
                lock.lock();
                try {
                    System.out.println("---内层调用");
                } finally {
                    lock.unlock();
                }
            } finally {
                lock.unlock();
            }
        }).start();
    }

    // 隐式可重入锁
    private static void reEntryM1() {
        final Object o = new Object();
        new Thread(() -> {
            synchronized (o) {
                System.out.println("---外层调用");
                synchronized (o) {
                    System.out.println("---中层调用");
                    synchronized (o) {
                        System.out.println("---内层调用");
                    }
                }
            }
        }).start();
    }
}
```

### 等待可中断

通过 `lock.lockInterruptibly()` 中断**等待锁**的线程

- 也就是说正在等待的线程可以选择放弃等待，改为处理其他事情

> #Boer 正在 wait 等的线程不是在“等待锁”，而是缺少条件或者主动进入睡眠

> 演示：main 先获得了锁，t1 等待锁，main 打断了 t1 等待锁

```java
public static void main() {
	ReentrantLock lock = new ReentrantLock();

	Thread t1 = new Thread(() -> {
		log.debug("启动...");
		lock.lock();
		try {
			log.debug("获得了锁");
		} finally {
			lock.unlock();
		}
	}, "t1");


	lock.lock();
	log.debug("获得了锁");
	t1.start();
	try {
		sleep(1);
		t1.interrupt();
		log.debug("执行打断");
		sleep(1);
	} finally {
		log.debug("释放了锁");
		lock.unlock();
	}
}

17:29:08.167 [main] DEBUG c.TestInterrupt - 获得了锁
17:29:08.176 [t1] DEBUG c.TestInterrupt - 启动...
17:29:09.189 [main] DEBUG c.TestInterrupt - 执行打断
17:29:10.191 [main] DEBUG c.TestInterrupt - 释放了锁
17:29:10.191 [t1] DEBUG c.TestInterrupt - 获得了锁
```

> 如果是不可中断模式 `lock.lock()`，使用了 `t1.interrupt()` 也不会让 t1 等待中断

### 锁超时

```java
public boolean tryLock();

public boolean tryLock(long timeout, TimeUnit unit)  
        throws InterruptedException
```

> 演示：t1 获取锁等待 1s 后失败，返回

```java
public static void main() {  
    ReentrantLock lock = new ReentrantLock(); 
     
    Thread t1 = new Thread(() -> {  
        log.debug("启动...");  
        try {  
            if (!lock.tryLock(1, TimeUnit.SECONDS)) {  
                log.debug("获取等待 1s 后失败，返回");  
                return;  
            }  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        try {  
            log.debug("获得了锁");  
        } finally {  
            lock.unlock();  
        }  
    }, "t1");  
  
    lock.lock();  
    log.debug("获得了锁");  
    t1.start();  
    try {  
        sleep(2);  
    } finally {  
        lock.unlock();  
    }  
}
```

---
#todo 解决哲学家就餐问题

### 公平锁

> 概念见八股

> 演示：非公平，后面的线程有机会插入到前面的线程之前运行

```java
public static void main(String[] args) throws InterruptedException {
	// 公平
	// ReentrantLock lock = new ReentrantLock(true);
	// 非公平
	ReentrantLock lock = new ReentrantLock(false);

	lock.lock();

	for (int i = 0; i < 20; i++) {
		new Thread(() -> {
			lock.lock();
			try {
				System.out.println(Thread.currentThread().getName() + " running...");
			} finally {
				lock.unlock();
			}
		}, "t" + i).start();
	}
	// 都卡着抢不到

	// 1s 之后去争抢锁
	Thread.sleep(1000);
	for (int i = 20; i < 25; i++) {
		new Thread(() -> {
			lock.lock();
			try {
				System.out.println(Thread.currentThread().getName() + " running...");
			} finally {
				lock.unlock();
			}
		}, "t" + i + " 强行插入").start();
	}

	lock.unlock();
	// 开始抢了
}

公平：完全按照申请锁的顺序
t0 running...
t1 running...
t2 running...
t3 running...
t4 running...
t5 running...
t6 running...
t7 running...
t8 running...
t9 running...
t10 running...
t11 running...
t12 running...
t13 running...
t14 running...
t15 running...
t16 running...
t18 running...
t17 running...
t19 running...
t20 强行插入 running...
t22 强行插入 running...
t21 强行插入 running...
t23 强行插入 running...
t24 强行插入 running...
```

### 条件变量

> synchronized 中也有条件变量，就是我们讲原理时那个 waitSet 休息室，当条件不满足时进入 waitSet 等待

ReentrantLock 支持**多个**条件变量的
- synchronized 是那些不满足条件的线程都在**一间休息室**等消息
- ReentrantLock 支持**多间休息室**（专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒）

使用要点：

- await 前需要**先获得锁**，否则会抛出 IllegalMonitorStateException
- await 执行后，会**释放锁**，进入 conditionObject 等待
- await 的线程被唤醒（或打断、或超时）**重新竞争** lock 锁
- 竞争 lock 锁成功后，从 await 后继续执行

```java
@Slf4j()
public class TestCondition {
    static ReentrantLock lock = new ReentrantLock();
    static Condition waitCigaretteQueue = lock.newCondition();
    static Condition waitbreakfastQueue = lock.newCondition();
    static volatile boolean hasCigrette = false;
    static volatile boolean hasBreakfast = false;

    public static void main(String[] args) {
        new Thread(() -> {
            try {
                lock.lock();
                while (!hasCigrette) {
                    try {
                        waitCigaretteQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.debug("等到了它的烟");
            } finally {
                lock.unlock();
            }
        }).start();

        new Thread(() -> {
            try {
                lock.lock();
                while (!hasBreakfast) {
                    try {
                        waitbreakfastQueue.await();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                log.debug("等到了它的早餐");
            } finally {
                lock.unlock();
            }
        }).start();

        sleep(1);
        sendBreakfast();
        sleep(1);
        sendCigarette();
    }

    private static void sendCigarette() {
        lock.lock();
        try {
            log.debug("送烟来了");
            hasCigrette = true;
            waitCigaretteQueue.signal();
        } finally {
            lock.unlock();
        }
    }

    private static void sendBreakfast() {
        lock.lock();
        try {
            log.debug("送早餐来了");
            hasBreakfast = true;
            waitbreakfastQueue.signal();
        } finally {
            lock.unlock();
        }
    }
}
```

# ---------- 共享模型_内存

JMM 即 Java Memory Model，它定义了**主存、工作内存抽象概念**，底层对应着 CPU 寄存器、缓存、硬件内存、CPU 指令优化等。

JMM 体现在以下几个方面

- 原子性 - 保证指令不会受到线程上下文切换的影响
- 可见性 - 保证指令不会受 cpu 缓存的影响
- 有序性 - 保证指令不会受 cpu 指令并行优化的影响

## 主内存与工作内存

#《深入理解Java虚拟机》 

Java 内存模型的主要目的是定义程序中各种变量的访问规则，即关注在虚拟机中把变量值存储到内存和从内存中取出变量值这样的底层细节

> 此处的变量（Variables）与 Java 编程中所说的变量有所区别，它包括了实例字段、静态字段和构成数组对象的元素，但是**不包括 局部变量 与 方法参数**，因为后者是**线程私有的，不会被共享，自然就不会存在竞争问题**

- Java 内存模型规定了所有的变量都存储在主内存（Main Memory）中
- 每条线程还有自己的工作内存，
	- 保存了被该线程使用的**变量的主内存副本**，
	- 线程对变量的所有操作（读取、赋值等）都**必须在工作内存中进行**，而不能直接读写主内存中的数据
- 不同的线程之间也无法直接访问对方工作内存中的变量，**线程间变量值的传递均需要通过主内存**来完成

> 此处的主内存与介绍物理硬件时提到的主内存名字一样，两者也可以类比，但**物理上它仅是虚拟机内存的一部分**

![](assets/Pasted%20image%2020240205224953.png)

> 这里所讲的主内存、工作内存与 Java 内存区域中的 Java 堆、栈、方法区等并不是同一个层次的对内存的划分，这两者基本上是没有任何关系的。如果两者一定要勉强对应起来，那么从变量、主内存、工作内存的定义来看，
> - 主内存主要对应于 Java “堆”中的对象实例数据部分[4]，
> - 而工作内存则对应于“虚拟机栈”中的部分区域。
> 
> 从更基础的层次上说，
> - 主内存直接对应于物理硬件的内存，
> - 而为了获取更好的运行速度，虚拟机（或者是硬件、操作系统本身的优化措施）可能会让工作内存优先存储于寄存器和高速缓存中，因为程序运行时主要访问的是工作内存。

> #JavaGuide 
> 
> - *主内存*：所有线程创建的实例对象都存放在主内存中，不管该实例对象是成员变量，还是局部变量，类信息、常量、静态变量都是放在主内存中。为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中。
> - *本地内存*：每个线程都有一个私有的本地内存，本地内存存储了该线程以**读 / 写共享变量的副本**。
> 	- 每个线程只能操作自己本地内存中的变量，无法直接访问其他线程的本地内存。
> 	- **如果线程间需要通信，必须通过主内存来进行**。
> 	- 本地内存是 JMM **抽象**出来的一个概念，并不真实存在，它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化。

### ~~内存间交互操作~~

#《深入理解Java虚拟机》 

关于主内存与工作内存之间具体的交互协议，即一个变量如何从主内存拷贝到工作内存、如何从工作内存同步回主内存这一类的实现细节，Java 内存模型中定义了 8 种操作来完成。

Java 虚拟机实现时必须保证下面提及的每一种操作都是**原子的、不可再分**的（对于 double 和 long 类型的变量来说， load、store、read 和 write 操作在某些平台上允许有例外）
- **锁定（lock）**: 作用于主内存中的变量，将他标记为一个线程独享变量。
- **解锁（unlock）**: 作用于主内存中的变量，解除变量的锁定状态，被解除锁定状态的变量才能被其他线程锁定。
- **read（读取）**：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的 load 动作使用。
- **load(载入)**：把 read 操作从主内存中得到的变量值放入工作内存的变量的副本中。
- **use(使用)**：把工作内存中的一个变量的值传给执行引擎，每当虚拟机遇到一个使用到变量的指令时都会使用该指令。
- **assign（赋值）**：作用于工作内存的变量，它把一个从执行引擎接收到的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
- **store（存储）**：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的 write 操作使用。
- **write（写入）**：作用于主内存的变量，它把 store 操作从工作内存中得到的变量的值放入主内存的变量中。

> 如果要把一个变量从主内存拷贝到工作内存，那就要按顺序执行 read 和 load 操作，如果要把变量从工作内存同步回主内存，就要按顺序执行 store 和 write 操作。注意，Java 内存模型只要求上述两个操作必须按顺序执行，但不要求是连续执行。也就是说 read 与 load 之间、store 与 write 之间是可插入其他指令的，如对主内存中的变量 a、b进行访问时，一种可能出现的顺序是 read a、read b、load b、load a。

除此之外，Java 内存模型还规定了在执行上述 8 种基本操作时必须满足如下规则：
- 不允许 read 和 load、store 和 write 操作之一单独出现，
	- 即不允许一个变量从主内存读取了但工作内存不接受，或者工作内存发起回写了但主内存不接受的情况出现。
- 不允许一个线程丢弃它最近的 assign 操作，
	- 即变量在工作内存中改变了之后必须把该变化同步回主内存。
- 不允许一个线程无原因地（没有发生过任何 assign 操作）把数据从线程的工作内存同步回主内存中。
- 一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load 或 assign）的变量，
	- 换句话说就是对一个变量实施 use、store 操作之前，必须先执行 assign 和 load 操作。
- 一个变量在同一个时刻只允许一条线程对其进行 lock 操作，但 lock 操作可以被同一条线程重复执行多次，多次执行 lock 后，只有执行相同次数的 unlock 操作，变量才会被解锁。
- 如果对一个变量执行 lock 操作，那将会**清空工作内存中此变量的值**，在执行引擎使用个变量前，需要重新执行 load 或 assign 操作以初始化变量的值。
- 如果一个变量事先没有被 lock 操作锁定，那就不允许对它执行 unlock 操作，也不允许去 unlock 一个被其他线程锁定的变量。
- 对一个变量执行 unlock 操作之前，必须先把此变量同步回主内存中（执行 store、write 操作）。


## 原子性

1）原子性：指一个操作是不可打断的，即多线程环境下，操作不能被其他线程干扰

> #《深入理解Java虚拟机》 
> 
> 由 Java 内存模型来直接保证的原子性变量操作包括 read、load、assign、use、store 和 write 这六个，我们大致可以认为，**基本数据类型的访问、读写都是具备原子性的**（例外就是 long 和 double 的非原子性协定，读者只要知道这件事情就可以了，无须太过在意这些几乎不会发生的例外情况）
> 
> 如果应用场景需要一个更大范围的原子性保证（经常会遇到），Java 内存模型还提供了 lock 和 unlock 操作来满足这种需求，尽管虚拟机未把 lock 和 unlock 操作直接开放给用户使用，但是却提供了更高层次的**字节码指令 monitorenter 和 monitorexit 来隐式地使用这两个操作**。这两个字节码指令反映到 Java 代码中就是同步块——**synchronized 关键字**，因此在 synchronized 块之间的操作也具备原子性。

## 可见性

定义：当一个线程修改了某一个共享变量的值，其他线程是否能够**立即**知道该变更。

> #Bo 在不使用 volatile 关键字的情况下，有哪些情况会导致线程的工作内存失效，然后必须重新去读取主存的共享变量？
> 
> - 线程中释放锁时
> - 线程切换时
> - CPU有空闲时间时（比如线程休眠时）
> 
> 参考：[多线程中主存与线程工作空间同步数据的时机_线程在读取一个静态变量后,什么时候再次同步主内存的数据-CSDN博客](https://blog.csdn.net/Hellowenpan/article/details/103202898)

### 退不出的循环

main 线程对 run 变量的修改对于 t 线程**不可见**，导致了 t 线程无法停止

```java
 static boolean run = true;

public static void main(String[] args) throws InterruptedException {
	Thread t = new Thread(() -> {
		while (true) {
            // System.out.println(run);
			if (!run) {
				break;
			}
		}
	});
	t.start();

	sleep(1);
	run = false; // 线程t不会如预想的停下来
}
```

> #todo while 里面加入 sout，线程就停止了
> - 第一种说法（不靠谱）：`println()` 方法内有同步块，但是 run 变量没在同步块里面啊？
> - 第二种说法：IO 操作会阻塞线程 t，上下文切换回来后，t 会重新读取主存的值

分析：

- 初始状态，t 线程刚开始从主内存读取了 run 的值到工作内存。
- 因为 t 线程要频繁从主内存中读取 run 的值，**JIT 编译器**会将 run 的值缓存至自己工作内存中的高速缓存中，减少对主存中 run 的访问，提高效率
- 1 秒之后，main 线程修改了 run 的值，并同步至主存，而 t 是从自己工作内存中的高速缓存中读取这个变量的值，结果永远是旧值

> #Bo 不是一开始就缓存，执行到一定次数后缓存，所以主线程要 `sleep(1)`，不然 t 线程会停下来

![|500](assets/Pasted%20image%2020240309225817.png)

### 解决办法

> #《深入理解Java虚拟机》 
> 
> - volatile 保证了多线程操作时变量的可见性
> 
> - synchronized 的可见性是由“**对一个变量执行 unlock 操作之前，必须先把此变量同步回主内存中（执行 store、write 操作）**”这条规则获得的
> 
> - final 关键字的可见性是指：被 final 修饰的字段**在构造器中一旦被初始化完成**，并且构造器没有把“this”的引用传递出去（this 引用逃逸是一件很危险的事情，其他线程有可能通过这个引用访问到“初始化了一半”的对象），那么**在其他线程中就能看见 final 字段的值**

## 有序性

JVM 会在不影响正确性的前提下，可以调整语句的执行顺序

> #《深入理解Java虚拟机》 
> 
> Java 程序中**天然的有序性**：如果在本线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有的操作都是无序的。
> 
> - 前半句是指“线程内似表现为串行的语义”，
> - 后半句是指“**指令重排序**”现象和“**工作内存与主内存同步延迟**”现象。


在某些情况下要**禁止**指令重排序
- 单线程环境里能够确保程序最终执行结果和代码顺序执行的结果一致。处理器在进行重排序时==必须要考虑==指令之间的**数据依赖性**。
- 多线程环境中线程交替执行，由于**编译器优化重排**的存在，两个线程中使用的变量能否保证一致性是==无法确定的==，结果无法预测。

> #JavaGuide 
> 
> 编译器和处理器的“指令重排序”的处理方式不一样。
> - 对于编译器，通过禁止特定类型的编译器重排序的方式来禁止重排序。
> - 对于处理器，通过插入内存屏障（Memory Barrier，或有时叫做内存栅栏，Memory Fence）的方式来禁止特定类型的处理器重排序。指令并行重排 和 内存系统重排 都属于是处理器级别的指令重排序。

### 指令级并行原理

#todo 

### 指令重排序

指令的重排序：Java 规范规定 JVM 线程内部维持顺序化语义，即只要 **程序的最终结果 与 它顺序化执行的结果相等**，那么 指令的执行顺序 可以与 代码顺序 不一致

> Java 源码到最终执行：源代码-->编译器优化的重排-->指令并行的重排-->内存系统的重排-->最终执行的指令

优缺点：

- JVM 能根据处理器特性（CPU 多级缓存系统、多核处理器等）适当的对机器指令进行重排序，使机器指令能更符合 CPU 的执行特性，最大限度的**发挥机器性能**。
- 但是，指令重排可以**保证串行语义一致**，但**不保证多线程间的语义也一致**

> #JavaGuide 
> 
> 常见的指令重排序有下面 2 种情况：
> - *编译器优化重排*：编译器（包括 JVM、JIT 编译器等）在**不改变单线程程序语义**的前提下，重新安排语句的执行顺序。
> - *指令并行重排*：现代处理器采用了指令级并行技术(Instruction-Level Parallelism，ILP)来将多条指令重叠执行。如果**不存在数据依赖性**，处理器可以改变语句对应机器指令的执行顺序。

### 诡异的结果

结果可能是 0：线程2 执行 `ready = true`，切换到线程 1，进入 if 分支，相加为 0，再切回线程 2 执行 num = 2。这种现象叫做指令重排

> 线程切换到 1，会重新从贮存中读取 ready

```java
int num=0;
boolean ready = false;

// 线程1 执行此方法
public void actor1(I_Result r) {
	if(ready) {
		r.r1 = num + num;
	} else {
		r.r1 = 1;
	}
}

// 线程2 执行此方法
public void actor2(I_Result r) { 
	num = 2;
	ready = true; 
}
```



> #todo 并发工具复现错误

### 解决方法

volatile 修饰的变量，可以禁用指令重排

> #《深入理解Java虚拟机》 
> 
> Java 语言提供了 volatile 和 synchronized 两个关键字来保证**线程之间操作的有序性**，
> 
> - volatile 关键字本身就包含了**禁止指令重排序**的语义，
> - synchronized 由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，这个规则决定了**持有同一个锁的两个同步块只能串行地进入**
> - final

> #todo synchronized无法禁用同步块内部的指令重排

## happens-before

happens-before 规定了对共享变量的写操作对其它线程的读操作可见，它是**可见性与有序性**的一套规则总结

抛开以下 happens-before 规则，JMM 并不能保证一个线程对共享变量的写，对于其它线程对该共享变量的读可见

- 线程解锁 m 之前对变量的写，对于接下来对 m 加锁的其它线程对该变量的读可见

```java
static int x;
static Object m = new Object();

new Thread(()->{
	synchronized(m) {
		x = 10;
	}
},"t1").start();

new Thread(()->{
	synchronized(m) {
		System.out.println(x);
	}
},"t2").start();
```

- 线程对 volatile 变量的写，对接下来其它线程对该变量的读可见

```java
volatile static int x;

new Thread(()->{
	x = 10;
},"t1").start();

new Thread(()->{
	System.out.println(x);
},"t2").start();
```

- 线程 start 前对变量的写，对该线程开始后对该变量的读可见

```java
static int x;
x = 10;

new Thread(()->{
	System.out.println(x);
},"t2").start();
```

- 线程结束前对变量的写，对其它线程得知它结束后的读可见（比如其它线程调用 t1.isAlive() 或 t1.join()等待它结束）

```java
static int x;

Thread t1 = new Thread(()->{
	x = 10;
},"t1");
t1.start();

t1.join();
System.out.println(x);
```

- 线程 t1 打断 t2（interrupt）前对变量的写，对于其他线程得知 t2 被打断后对变量的读可见（通过 t2.interrupted 或 t2.isInterrupted）

```java
static int x;

public static void main(String[] args) {
	Thread t2 = new Thread(()->{
		while(true) {
			if(Thread.currentThread().isInterrupted()) {
				System.out.println(x);
				break;
			}
		}
	},"t2");
	t2.start();
	
	new Thread(()->{
		sleep(1);
		x = 10;
		t2.interrupt();
	},"t1").start();
	
	while(!t2.isInterrupted()) {
		Thread.yield();
	}
	
	System.out.println(x);
}
```

- 对变量默认值（0，false，null）的写，对其它线程对该变量的读可见

- 具有传递性，如果 x hb-> y 并且 y hb-> z 那么有 x hb-> z ，配合 volatile 的防指令重排，有下面的例子

```java
volatile static int x;
static int y;

new Thread(()->{ 
	y = 10;
	x = 20;
},"t1").start();

new Thread(()->{
	// x=20 对 t2 可见, 同时 y=10 也对 t2 可见
	System.out.println(x); 
},"t2").start();
```






## volatile

volatile（易变关键字）：

- 可以用来修饰**成员变量**和**静态成员变量**，
- 可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存

被 volatile 修饰的变量特点：

1. 可见性
2. 有序性：有排序要求，有时需要禁重排
3. 不保证原子性

内存语义：
- 写 volatile 变量时，线程对应的本地内存中的**共享变量值立即刷新回主内存中**
- 读 volatile 变量时，线程对应的本地内存设置为无效，**重新回到主内存中读取最新共享变量的值**

### 原理

> 代码参考[诡异的结果](#诡异的结果)

volatile 的底层实现原理是【内存屏障】（Memory Barrier/Memory Fence）

- 对 volatile 变量的写指令**后**会加入写屏障
- 对 volatile 变量的读指令**前**会加入读屏障

#### 保证可见性

- 写屏障（sfence）保证在该屏障**之前**的，对共享变量的改动，都同步到主存当中
- 读屏障（lfence）保证在该屏障**之后**，对共享变量的读取，加载的是主存中最新数据

```java
boolean volatile ready;

// Thread1
public void actor2(I_Result r) {
	num = 2;
	ready = true; // ready 是 volatile 赋值带写屏障
	// 写屏障
}

// Thread2
public void actor1(I_Result r) {
	// 读屏障
	// ready 是 volatile 读取值带读屏障
	if(ready) {
		r.r1 = num + num;
	} else {
		r.r1 = 1;
	}
}
```

![|600](assets/Pasted%20image%2020240310125710.png)

#### 保证有序性

- 写屏障会确保指令重排序时，不会将写屏障**之前**的代码排在写屏障之后
- 读屏障会确保指令重排序时，不会将读屏障**之后**的代码排在读屏障之前

![|600](assets/Pasted%20image%2020240310130758.png)

**不能解决指令交错**：

- 写屏障仅仅是保证之后的读能够读到最新的结果，但不能保证读跑到它前面去
- 而有序性的保证也只是保证了**本线程内**相关代码不被重排序

![|500](assets/Pasted%20image%2020240310132745.png)

#### 不保证原子性

> #《深入理解Java虚拟机》 
> 
> Java 里面的运算操作符并非原子操作，这导致 volatile 变量的运算在并发下一样是不安全的

> 无原子性演示：多线程环境下 `inc++` 指令交错

```java
public class VolatileSeeDemo2 {
    public volatile static int inc;

    public static void addPlusPlus() {
        inc++;
    }

    public static void main(String[] args) throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    addPlusPlus();
                }
            }).start();
        }

        Thread.sleep(3000);
        log.debug("inc: {}", inc);
        // 13:40:18.597 [main] DEBUG cn.itcast.n5.VolatileSeeDemo2 - inc: 7758
    }
}
```

`inc++` 其实是一个复合操作，包括三步：

1. 读取 inc 的值。
2. 对 inc 加 1。
3. 将 inc 的值写回内存

> 即便这三个操作可见性和有序性得到了保证，但是指令交错会导致无效的写操作

volatile 是无法保证这三个操作是具有原子性的，可能导致两个线程分别对 inc 进行了一次自增操作后，inc 实际上只增加了 1

改进方式：

```java
//synchronized
public synchronized void increase() {
    inc++;
}

//AtomicInteger
public AtomicInteger inc = new AtomicInteger();

public void increase() {
    inc.getAndIncrement();
}

//ReentrantLock
Lock lock = new ReentrantLock();
public void increase() {
    lock.lock();
    try {
        inc++;
    } finally {
        lock.unlock();
    }
}
```

### double-checked locking

以著名的 double-checked locking 单例模式为例，它的实现特点：

- 懒惰实例化
- 首次使用 `getInstance()` 才使用 synchronized 加锁，后续使用时无需加锁
- `INSTANCE == null` 是在同步块之外，INSTANCE 用 **volatile** 修饰保证有序性

```java
public final class Singleton {
    private Singleton() {}

    private static volatile Singleton INSTANCE = null;

    public static Singleton getInstance() {
        // 实例没创建，才会进入内部的 synchronized代码块
        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                // 也许有其它线程已经创建实例，所以再判断一次
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

---
`getInstance()` 的字节码如下

```java
0: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
// Singleton不为null，跳转37
3: ifnonnull 37 
// 获得类对象Singleton.class
6: ldc #3 // class cn/itcast/n5/Singleton 
8: dup
9: astore_0
10: monitorenter
11: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
14: ifnonnull 27
17: new #3 // class cn/itcast/n5/Singleton
20: dup
21: invokespecial #4 // Method "<init>":()V
24: putstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
27: aload_0
28: monitorexit
29: goto 37
32: astore_1
33: aload_0
34: monitorexit
35: aload_1
36: athrow
37: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
40: areturn
```

> - 17 表示创建对象，将对象引用入栈 // new Singleton
> - 20 表示复制一份对象引用 // 引用地址
> - 21 表示利用一个对象引用，调用构造方法
> - 24 表示利用一个对象引用，赋值给 static INSTANCE

也许 jvm 会优化为：先执行 24，再执行 21

- t1 执行了 24，INSTANCE 变量中已经有地址值
- t1 还未执行 21，即没调用构造方法
- t2 执行 0，发现 `INSTANCE!=null`，t2 将拿到一个**未初始化完毕的单例**

![|700](assets/Pasted%20image%2020240310150852.png)

对 INSTANCE 使用 volatile 修饰即可，可以禁用指令重排
- 24 带写屏障，24 前的 21 就不可以排到 24 后面

> 字节码上看不出来 volatile 指令的效果

```java
-------------------------------------> 加入对 INSTANCE 变量的读屏障
0: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
3: ifnonnull 37
6: ldc #3 // class cn/itcast/n5/Singleton
8: dup
9: astore_0
10: monitorenter -----------------------> 保证原子性、可见性
11: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
14: ifnonnull 27
17: new #3 // class cn/itcast/n5/Singleton
20: dup
21: invokespecial #4 // Method "<init>":()V
24: putstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
-------------------------------------> 加入对 INSTANCE 变量的写屏障
27: aload_0
28: monitorexit ------------------------> 保证原子性、可见性
29: goto 37
32: astore_1
33: aload_0
34: monitorexit
35: aload_1
36: athrow
37: getstatic #2 // Field INSTANCE:Lcn/itcast/n5/Singleton;
40: areturn
```

![|700](assets/Pasted%20image%2020240310153225.png)

### ~~内存屏障~~

> 内存屏障是什么？

内存屏障（也称内存栅栏，屏障指令等）是一类**同步屏障指令**，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作，避免代码重排序。

内存屏障其实就是一种**JVM指令**，Java内存模型的重排规则会要求==Java编译器在生成JVM指令时插入特定的内存屏障指令==，通过这些内存屏障指令，volatile实现了Java内存模型中的可见性和有序性（禁重排），但==volatile无法保证原子性==
- 内存屏障**之前**的所有**写操作**都要回写到主内存
- 内存屏障**之后**的所有**读操作**都能获得内存屏障之前的所有写操作的最新结果（可见性）

~~写屏障（Store Memory Barrier）：~~
- ~~告诉处理器在写屏障之前将所有存储在缓存(store buffers)中的数据同步到主内存，~~
- ~~也就是说当看到**Store屏障指令**，就必须把该指令之前的所有写入指令执行完毕才能继续往下执行~~

~~读屏障（Load Memory Barrier）：~~
- ~~处理器在读屏障之后的读操作，都在读屏障之后执行。~~
- ~~也就是说在**Load屏障指令**之后就能够保证后面的读取数据指令一定能够读取到最新的数据。 |~~

重排序时，==不允许把内存屏障之后的指令重排序到内存屏障之前==。

一句话：对一个volatile变量的写，先行发生于任意后续对这个volatile变量的读，也叫写后读

> 内存屏障分类

粗分两种：
- 读屏障（Load Barrier）：在==读指令之前插入读屏障==，让工作内存或CPU高速缓存 当中的缓存数据失效，重新回到主内存中获取最新数据。
- 写屏障（Store Barrier）：在==写指令之后插入写屏障==，强制把缓冲区的数据刷回到主内存中。

> 内存屏障禁重排--->保证有序性

- 重排序有可能影响程序的执行和实现，因此，我们有时候希望告诉JVM别自动重排序，我这里不需要重排序，一切听我的。
- 对于编译器的重排序，JMM会根据重排序的规则，禁止特定类型的编译器重排序
- 对于处理器的重排序，Java编译器在生成指令序列的适当位置，插入内存屏障指令，来禁止特定类型的处理器排序。

> happens-before之volatile变量规则

| 第一个操作 | 第二个操作：普通读写 | 第二个操作：volatile读 | 第二个操作：volatile写 |
| ---- | ---- | ---- | ---- |
| 普通读写 | 可以重排 | 可以重排 | 不可以重排 |
| volatile读 | 不可以重排 | 不可以重排 | 不可以重排 |
| volatile写 | 可以重排 | 不可以重排 | 不可以重排 |

- 当第一个操作为volatile读时，不论第二个操作是什么，都不能重排序，这个操作保证了volatile读之后的操作不会被重排到volatile读之前。
- 当第一个操作为volatile写，第二个操作为volatile读时，不能重排
- 当第二个操作为volatile写时，不论第一个操作是什么，都不能重排序
	- 这个操作保证了volatile写之前的操作不会被重排到volatile写之后

> JMM就将内存屏障插入策略分为4种规则

| **屏障类型** | **指令示例** | **说明** |
| ---- | ---- | ---- |
| LoadLoad | Load1;LoadLoad;Load2 | 保证Load1的读取操作在Load2及后续读取操作之前执行 |
| StoreStore | Store1;StoreStore;Store2 | 在store2及其后的写操作执行前，保证Store1的写操作已经刷新到主内存 |
| LoadStore | Load1;LoadStore;Store2 | 在Store2及其后的写操作执行前，保证Load1的读操作已经结束 |
| StoreLoad | Store1;StoreLoad;Load2 | 保证Store1的写操作已经刷新到主内存后，Load2及其后的读操作才能执行 |

读屏障：在每个volatile**读**操作的**后面**插入一个
- LoadLoad屏障：禁止处理器把上面的volatile读与下面的普通读重排序。
- LoadStore屏障：禁止处理器把上面的volatile读与下面的普通写重排序。

![](assets/Pasted%20image%2020240108174829.png)

写屏障：在每个volatile**写**操作的
- **前面**插入StoreStore屏障：保证在volatile写之前，其前面的所有普通写操作都已经刷新到主内存中
- **后面**插入StoreLoad屏障：避免volatile写与后面可能有的volatile读/写操作重排序

![](assets/Pasted%20image%2020240108175107.png)

### 如何正确使用 volatile ？

> 单一赋值可以，但是含复合运算赋值不可以（i++之类的）

- volatile int a = 10;
- volatile boolean flag = true;

> 状态标志，判断业务是否结束

```java
public class VolatileSeeDemo {

	// 状态标志
    static volatile boolean flag = true;

    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t-------come in");
            while (flag) {
				// 业务
            }
            System.out.println(Thread.currentThread().getName() + "\t-------flag被设置为false，程序停止");
        }, "t1").start();

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 更新标志位
        flag = false;
    }
}
```

> 开销较低的读，写锁策略

当读远多于写，结合使用内部锁和 volatile 变量来==减少同步的开销==
- 利用 volatile 保证读取操作的可见性
- 用 synchronized 保证复合操作的原子性

```java
private volatile int value =0;

public int getValue(){
    return value;
}

public synchronized int setValue(){
    return value++;
}
```

### 字节码层面理解 volatile

凭什么我们Java写了一个volatile关键字，系统底层加入内存屏障？两者的关系如何勾搭？

![](assets/Pasted%20image%2020240109140623.png)

## 习题

#todo balking 模式习题

#todo 线程安全单例习题

# ---------- 共享模型_无锁

## CAS

### 引出

要保证 withdraw() 取款方法的线程安全

```java
interface Account {
    // 获取余额
    Integer getBalance();

    // 取款
    void withdraw(Integer amount);

    /**
     * 方法内会启动 1000 个线程，每个线程做 -10 元 的操作
     * 如果初始余额为 10000 那么正确的结果应当是 0
     */
    static void demo(Account account) {
        List<Thread> ts = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(10);
            }));
        }
        long start = System.nanoTime();
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        long end = System.nanoTime();
        System.out.println(account.getBalance()
                + " cost: " + (end-start)/1000_000 + " ms");
    }
}

class AccountUnsafe implements Account {

    private Integer balance;

    public AccountUnsafe(Integer balance) {
        this.balance = balance;
    }

    @Override
    public Integer getBalance() {
        synchronized (this) {
            return this.balance;
        }
    }

    @Override
    public void withdraw(Integer amount) {
        synchronized (this) {
            this.balance -= amount;
        }
    }

	public static void main(String[] args) {
        Account account = new AccountUnsafe(10000);
        Account.demo(account);
        //330 cost: 306 ms
    }
}
```

原有实现并不是线程安全的，无锁解决，AtomicInteger

```java
class AccountCas implements Account {
    private AtomicInteger balance;

    public AccountCas(int balance) {
        this.balance = new AtomicInteger(balance);
    }

    @Override
    public Integer getBalance() {
        return balance.get();
    }

    @Override
    public void withdraw(Integer amount) {
        while(true) {
            // 获取余额的最新值
            int prev = balance.get();
            // 要修改的余额
            int next = prev - amount;
            // 真正修改
            if(balance.compareAndSet(prev, next)) {
                break;
            }
        }
        // 可以简化为下面的方法
        // balance.getAndAdd(-1 * amount);
    }
}
```

### 介绍

> AtomicInteger 的解决方法，内部并没有用锁来保护共享变量的线程安全，

其中的关键是 `compareAndSet()`，它的简称就是 CAS （也有 Compare And Swap 的说法），用于实现**乐观锁**

CAS 是一个**原子操作**

> 原子操作 即最小不可拆分的操作，也就是说操作一旦开始，就不能被打断，直到操作完成。
> 
> CAS底层依赖一条原子指令 lock cmpxchg 指令（X86 架构），在单核 CPU 和**多核** CPU 下都能够保证【比较-交换】的原子性。
> 
> - 在多核状态下，某个核执行到带 lock 的指令时，CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断，保证了多个线程对内存操作的准确性，是原子的

CAS 涉及到三个操作数：

- **V**：要更新的变量值(Var)
- **E**：预期值(Expected)
- **N**：拟写入的新值(New)

思想：用 E 和 V 比较，相等用 N 更新 V。如果不等，说明已经有其它线程更新了 V

![|500](assets/Pasted%20image%2020240310174629.png)

### CAS 与 volatile

> 获取共享变量时，为了保证该变量的可见性，需要使用 volatile 修饰。
> 
> 它可以用来修饰成员变量和静态成员变量，他可以避免线程从自己的工作缓存中查找变量的值，必须到主存中获取它的值，线程操作 volatile 变量都是直接操作主存。即一个线程对 volatile 变量的修改，对另一个线程可见。

volatile 仅仅保证了共享变量的可见性，让其它线程能够看到最新值，但**不能解决【指令交错】** 问题（不能保证原子性）

CAS 必须**借助 volatile** 才能读取到共享变量的最新值来实现【比较并交换】的效果

### 无锁效率高的原因

无锁情况下，即使重试失败，**线程始终在高速运行，没有停歇**，而 synchronized 会让线程在没有获得锁的时候，发生上下文切换，进入阻塞。

> 打个比喻线程就好像高速跑道上的赛车，高速运行时，速度超快，一旦发生上下文切换，就好比赛车要减速、熄火，等被唤醒又得重新打火、启动、加速... 恢复到高速运行，代价比较大

但无锁情况下，因为线程要保持运行，需要额外 CPU 的支持，CPU 在这里就好比高速跑道，**没有额外的跑道**，线程想高速运行也无从谈起，虽然不会进入阻塞，但由于没有分到时间片，仍然会进入可运行状态，**还是会导致上下文切换**。

> #Bo 空间换时间

### CAS 特点

> - CAS 是基于**乐观锁**的思想：最乐观的估计，不怕别的线程来修改共享变量，就算改了也没关系，我吃亏点再重试呗
> - synchronized 是基于**悲观锁**的思想：最悲观的估计，得防着其它线程来修改共享变量，我上了锁你们都别想改，我改完了解开锁，你们才有机会

结合 CAS 和 volatile 可以

- 实现无锁并发、无阻塞并发
	- 没有使用 synchronized，线程不会陷入阻塞，这是效率提升的因素之一
- 适用场景：**线程数少、多核 CPU**
- 但如果竞争激烈，可以想到重试必然频繁发生，反而效率会受影响

## 基本类型原子类

J.U.C 并发包提供了：

- AtomicBoolean
- AtomicInteger
- AtomicLong

> 以 AtomicInteger 为例

```java
AtomicInteger i = new AtomicInteger(0);

// 获取并自增（i = 0, 结果 i = 1, 返回 0），类似于 i++
System.out.println(i.getAndIncrement());

// 自增并获取（i = 1, 结果 i = 2, 返回 2），类似于 ++i
System.out.println(i.incrementAndGet());

// 自减并获取（i = 2, 结果 i = 1, 返回 1），类似于 --i
System.out.println(i.decrementAndGet());

// 获取并自减（i = 1, 结果 i = 0, 返回 1），类似于 i--
System.out.println(i.getAndDecrement());

// 获取并加值（i = 0, 结果 i = 5, 返回 0）
System.out.println(i.getAndAdd(5));

// 加值并获取（i = 5, 结果 i = 0, 返回 0）
System.out.println(i.addAndGet(-5));

// 获取并更新（i = 0, p 为 i 的当前值, 结果 i = -2, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.getAndUpdate(p -> p - 2));

// 更新并获取（i = -2, p 为 i 的当前值, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.updateAndGet(p -> p + 2));

// 获取并计算（i = 0, p 为 i 的当前值, x 为参数1, 结果 i = 10, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
// getAndUpdate 如果在 lambda 中引用了外部的局部变量，要保证该局部变量是 final 的
// getAndAccumulate 可以通过 参数1 来引用外部的局部变量，但因为其不在 lambda 中因此不必是 final
System.out.println(i.getAndAccumulate(10, (p, x) -> p + x));

// 计算并获取（i = 10, p 为 i 的当前值, x 为参数1, 结果 i = 0, 返回 0）
// 其中函数中的操作能保证原子，但函数需要无副作用
System.out.println(i.accumulateAndGet(-10, (p, x) -> p + x));
```

getAndUpdate 的源码

- IntUnaryOperator 是一个函数式接口
- 通过 `compareAndSet()` 更新 value

```java
public final int getAndUpdate(IntUnaryOperator updateFunction) {  
    int prev, next;  
    do {  
        prev = get();  
        next = updateFunction.applyAsInt(prev);  
    } while (!compareAndSet(prev, next));  
    return prev;  
}
```

getAndAccumulate 的源码

- IntBinaryOperator 也是一个函数式接口，抽象方法比 IntUnaryOperator 中的多一个参数 x

```java
public final int getAndAccumulate(int x,  
                                  IntBinaryOperator accumulatorFunction) {  
    int prev, next;
    do {  
        prev = get();  
        next = accumulatorFunction.applyAsInt(prev, x);  
    } while (!compareAndSet(prev, next));  
    return prev;  
}
```

## 原子引用

J.U.C 并发包提供了：

- `AtomicReference<V>`
- `AtomicMarkableReference<V>`
- `AtomicStampedReferenc<V>`

作用：自定义其他原子类型

### AtomicReference

CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5 开始，提供了AtomicReference 类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作

```java
@Slf4j  
public class AtomicReferenceDemo {  
    public static void main(String[] args) {  
        User user1 = new User("user1", 22);  
        User user2 = new User("user2", 33);  
        User user3 = new User("user3", 33);  
        AtomicReference<User> userAtomicReference = new AtomicReference<>(user1);  
  
        log.debug("{}, {}",  
                userAtomicReference.compareAndSet(user1, user2),  
                userAtomicReference.get().toString());  
        //true, User(name=user2, age=33)  
  
        log.debug("{}, {}",  
                userAtomicReference.compareAndSet(user1, user3),  
                userAtomicReference.get().toString());  
        //false, User(name=user2, age=33)  
    }  
}  
  
@Data  
class User {  
    private String name;  
    private int age;  
  
    public User(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
}
```


> 演示：多线程，每个线程对 BigDecimal 类型的 balance 减 10，初值为 10000 减为 0

```java
public class DecimalAccountCas {
    private AtomicReference<BigDecimal> balance;

    public DecimalAccountCas(BigDecimal balance) {
        this.balance = new AtomicReference<>(balance);
    }

    public BigDecimal getBalance() {
        return balance.get();
    }

    public void withdraw(BigDecimal amount) {
        while(true) {
            BigDecimal prev = balance.get();
            BigDecimal next = prev.subtract(amount);
            if (balance.compareAndSet(prev, next)) {
                break;
            }
        }
    }

    public static void main(String[] args) {
        DecimalAccountCas account = new DecimalAccountCas(new BigDecimal(10000));

        List<Thread> ts = new ArrayList<>();
        for (int i = 0; i < 1000; i++) {
            ts.add(new Thread(() -> {
                account.withdraw(BigDecimal.TEN);
            }));
        }
        ts.forEach(Thread::start);
        ts.forEach(t -> {
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        System.out.println(account.getBalance()); //0
    }
}
```

---
> #Bo BigDecimal 不是不可变的，线程安全的？

不使用原子引用尝试一下

```java
public class DecimalAccountCas2 {
    BigDecimal balance;

    public DecimalAccountCas2(BigDecimal balance) {
        this.balance = balance;
    }

    public BigDecimal getBalance() {
        return balance;
    }

    public void withdraw(BigDecimal amount) {
        balance = balance.subtract(amount);
    }
}
```

结果是>0，为什么呢？原子操作的组合？

```java
balance = balance.subtract(amount);

// 等同于

BigDecimal newBalance = balance.subtract(amount);  
balance = newBalance;
```

### ABA

```java
static AtomicReference<String> ref = new AtomicReference<>("A");
  
public static void main(String[] args) throws InterruptedException {  
	String prev = ref.get();
	
	other();
	
	sleep(1);
	// 尝试改为 C
	log.debug("change A->C {}", ref.compareAndSet(prev, "C"));
}  
  
private static void other() {  
    new Thread(() -> {
		log.debug("change A->B {}", ref.compareAndSet(ref.get(), "B"));
	}, "t1").start();

	sleep(0.5);

	new Thread(() -> {
		log.debug("change B->A {}", ref.compareAndSet(ref.get(), "A"));
	}, "t2").start();
}

11:29:52.379 c.Test36 [t1] - change A->B true 
11:29:52.879 c.Test36 [t2] - change B->A true 
11:29:53.880 c.Test36 [main] - change A->C true
```

主线程仅能判断出共享变量的值与最初值 A 是否相同，不能感知到这种**从 A 改为 B 又 改回 A** 的情况

如果主线程希望：只要有其它线程【动过了】共享变量，那么自己的 cas 就算失败，这时，仅比较值是不够的，需要**再加一个版本号**

### AtomicStampedReference

JDK 1.5 以后的 `AtomicStampedReference` 类就是用来解决 ABA 问题的，其中的 `compareAndSet()` 方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

AtomicStampedReference 可以给原子引用加上**版本号**，追踪原子引用整个的变化过程，知道引用变量中途被**更改了几次**

```java
public class AtomicStampedReference<V> {
	private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

	private volatile Pair<V> pair;

    public AtomicStampedReference(V initialRef, int initialStamp) {
        pair = Pair.of(initialRef, initialStamp);
    }

	public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            (
            // 新的引用和stamp 和 current 都一样，不用改了
            (newReference == current.reference &&
              newStamp == current.stamp) 
              ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
}
```

> ABA 问题解决

```java
static AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

public static void main(String[] args) throws InterruptedException {
	String prev = ref.getReference();
	// 获取版本号
	int stamp = ref.getStamp();
	log.debug("版本 {}", stamp);
	
	// 中间有其它线程干扰，发生了 ABA 现象
	other();
	sleep(1);
	
	// 尝试改为 C
	log.debug("change A->C {}", 
		ref.compareAndSet(prev, "C", stamp, stamp + 1));
}

private static void other() {
	new Thread(() -> {
		log.debug("change A->B {}", 
			ref.compareAndSet(ref.getReference(), "B", ref.getStamp(), ref.getStamp() + 1));
		log.debug("更新版本为 {}", ref.getStamp());
	}, "t1").start();
	
	sleep(0.5);
	
	new Thread(() -> {
		log.debug("change B->A {}", ref.compareAndSet(ref.getReference(), "A", ref.getStamp(), ref.getStamp() + 1));
		log.debug("更新版本为 {}", ref.getStamp());
	}, "t2").start();
}

15:41:34.894 c.Test36 [main] - 版本 0 
15:41:34.956 c.Test36 [t1] - change A->B true 
15:41:34.956 c.Test36 [t1] - 更新版本为 1 
15:41:35.457 c.Test36 [t2] - change B->A true 
15:41:35.457 c.Test36 [t2] - 更新版本为 2 
15:41:36.457 c.Test36 [main] - change A->C false
```

### AtomicMarkableReference

> #todo 是否解决aba？
> 
> - 针对是否更改过，肯定是解决了

有时候，并不关心引用变量更改了几次，只是单纯的关心**是否更改过**，所以就有了AtomicMarkableReference

![|200](assets/Pasted%20image%2020240311114302.png)

```java
public class Test38 {
    public static void main(String[] args) throws InterruptedException {
        GarbageBag bag = new GarbageBag("装满了垃圾");
        // 参数2 mark 可以看作一个标记，表示垃圾袋满了
        AtomicMarkableReference<GarbageBag> ref = new AtomicMarkableReference<>(bag, true);

        log.debug("start...");
        GarbageBag prev = ref.getReference();
        log.debug(prev.toString());

        new Thread(() -> {
            log.debug("start...");
            bag.setDesc("空垃圾袋");
            ref.compareAndSet(bag, bag, true, false);
            log.debug(bag.toString());
        },"保洁阿姨").start();

        sleep(1);
        log.debug("想换一只新垃圾袋？");
        boolean success = ref.compareAndSet(prev, new GarbageBag("空垃圾袋"), true, false);
        log.debug("换了么？" + success);
        log.debug(ref.getReference().toString());
    }
}

class GarbageBag {
    String desc;

    public GarbageBag(String desc) {
        this.desc = desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    @Override
    public String toString() {
        return super.toString() + " " + desc;
    }
}
```

## 原子数组

- AtomicIntegerArray
- AtomicLongArray
- AtomicReferenceArray

#todo 

## 字段更新器

- AtomicReferenceFieldUpdater // 域 字段
- AtomicIntegerFieldUpdater
- AtomicLongFieldUpdater

#todo 

## 原子累加器

#todo 

## Unsafe

#todo 

# ---------- 共享模型_不可变

## 日期转换问题

SimpleDateFormat 不是线程安全的

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

for (int i = 0; i < 10; i++) {
	 new Thread(() -> {
		 try {
			 log.debug("{}", sdf.parse("1951-04-21"));
		 } catch (Exception e) {
			 log.error("{}", e);
		 }
	 }).start();
}

java.lang.NumberFormatException
```

同步锁解决问题，但带来的是性能上的损失

```java
new Thread(() -> {
	 synchronized (sdf) {
		 try {
			 log.debug("{}", sdf.parse("1951-04-21"));
		 } catch (Exception e) {
			 log.error("{}", e);
		 }
	 }
 }).start();
```

如果一个对象在**不能够修改其内部状态（属性）**，那么它就是线程安全的，因为不存在并发修改，例如在 Java 8 后，提供了一个新的日期格式化类

```java
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy-MM-dd");

for (int i = 0; i < 10; i++) {
	new Thread(() -> {
		LocalDate date = dtf.parse("2018-10-01", LocalDate::from);
		log.debug("{}", date);
	}).start();
}
```

## 不可变设计

以 String 类为例

> #JavaGuide `String` 不可变原因：
> 
> - 保存字符串的数组被 `final` 修饰且为私有的，并且 `String` 类**没有提供/暴露修改这个字符串的方法**。（相当于只读）
> - `String` 类被 `final` 修饰导致其不能被继承，进而避免了子类破坏 `String` 不可变。

### final 的使用

- 所有属性用 final 修饰
- 类用 final 修饰，防止子类无意间破坏不可变性

```java
public final class String  
    implements java.io.Serializable, Comparable<String>, CharSequence {  
    /** The value is used for character storage. */  
    private final char value[];  
  
    /** Cache the hash code for the string */  
    private int hash; // Default to 0
}
```

### 保护性拷贝

使用字符串时，也有一些跟修改相关的方法啊，比如 substring 等。

substring 内部是调用 String 的构造方法创建了一个新字符串

```java
public String substring(int beginIndex) {
	if (beginIndex < 0) {
		throw new StringIndexOutOfBoundsException(beginIndex);
	}
	int subLen = value.length - beginIndex;
	if (subLen < 0) {
		throw new StringIndexOutOfBoundsException(subLen);
	}
	return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
}
```

构造新字符串对象时，会生成新的 `char[] value`，对内容进行复制。这种通过创建副本对象来避免共享的手段称之为【**保护性拷贝**（defensive copy）】

```java
public String(char value[], int offset, int count) {  
    if (offset < 0) {  
        throw new StringIndexOutOfBoundsException(offset);  
    }  
    if (count <= 0) {  
        if (count < 0) {  
            throw new StringIndexOutOfBoundsException(count);  
        }  
        if (offset <= value.length) {  
            this.value = "".value;  
            return;  
        }  
    }  
    // Note: offset or count might be near -1>>>1.  
    if (offset > value.length - count) {  
        throw new StringIndexOutOfBoundsException(offset + count);  
    }  
    this.value = Arrays.copyOfRange(value, offset, offset+count);  
}
```

考虑到不可变类需要频繁创建对象，一般都会关联一种设计模式【享元模式】

## final 原理

### 设置 final 变量的原理

final 变量的赋值也会通过 putfield 指令来完成，同样在这条指令之后也会加入**写屏障**，保证在其它线程读到它的值时不会出现为 0 的情况

> #《深入理解Java虚拟机》 
> 
> final 关键字的**可见性**是指：被final 修饰的字段在**构造器中一旦被初始化完成**，并且构造器没有把“this”的引用传递出去（this 引用逃逸是一件很危险的事情，其他线程有可能通过这个引用访问到“初始化了一半”的对象），那么**在其他线程中就能看见 final 字段的值** 

```java
public class TestFinal {
	final int a = 20;
}

0: aload_0
1: invokespecial #1 // Method java/lang/Object."<init>":()V
4: aload_0
5: bipush 20
7: putfield #2 // Field a:I
 <-- 写屏障
10: return
```

### 获取 final 变量的原理

#todo 

## 无状态

在 web 阶段学习时，设计 Servlet 时为了保证其线程安全，都会有这样的建议，不要为 Servlet 设置成员变量

这种没有**任何成员变量的类**是线程安全的

# ---------- 共享模型_线程池

## 自定义线程池

![|600](assets/Pasted%20image%2020240312131843.png)

> #todo 自己实现

## ThreadPoolExecutor

![|500](assets/Pasted%20image%2020240312145505.png)

### 线程池状态

ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量

![|600](assets/Pasted%20image%2020240312145604.png)

> 从数字上比较（第一位是符号位），TERMINATED > TIDYING > STOP > SHUTDOWN > RUNNING

线程池状态与线程个数存储在一个**原子变量** ctl 中，这样就可以用**一次 cas 原子操作**进行赋值

```java
// c 为旧值， ctlOf 返回结果为新值
ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))));

// rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们
private static int ctlOf(int rs, int wc) { return rs | wc; }
```

### 构造方法

```java
public ThreadPoolExecutor(
	int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue,
	ThreadFactory threadFactory,
	RejectedExecutionHandler handler
)
```

- *corePoolSize*：核心线程数目 (最多保留的线程数)
- *maximumPoolSize*：最大线程数目
- *keepAliveTime*：生存时间（针对救急线程）
- *unit*：时间单位（针对救急线程）
- *workQueue*：阻塞队列
- *threadFactory*：线程工厂 - 可以为线程创建时起个好名字
- *handler*：拒绝策略

根据这个构造方法，JDK **Executors** 类中提供了众多工厂方法来**创建各种用途的线程池**

工作流程：

- 线程池一开始没有线程，会创建新线程执行任务
- 当线程数达到 corePoolSize，新任务放入 workQueue 排队，直到有空闲线程
	- 如果 workQueue 是**有界**队列，那么任务超过了队列大小时，会创建 `maximumPoolSize - corePoolSize` 数目的线程来**救急**
- 如果线程数到达 maximumPoolSize 仍然有新任务，会执行**拒绝策略**。
- 当高峰过去后，超过 corePoolSize 的救急线程如果一段时间没有任务做，需要结束节省资源，这个时间由 keepAliveTime 和 unit 来控制

### 拒绝策略

```java
public interface RejectedExecutionHandler {  
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);  
}
```

jdk 提供了 4 种：

- *AbortPolicy*（默认）让调用者抛出 RejectedExecutionException
- *CallerRunsPolicy* 让调用者运行任务
- *DiscardPolicy* 放弃本次任务
- *DiscardOldestPolicy* 放弃队列中最早的任务，本任务取而代之

![](assets/Pasted%20image%2020240312153205.png)

> 其它著名框架也提供的拒绝策略实现：
> 
> - *Dubbo* 的实现，在抛出 RejectedExecutionException 异常之前会记录日志，并 dump 线程栈信息，方便定位问题
> - *Netty* 的实现，是创建一个新线程来执行任务
> - *ActiveMQ* 的实现，带超时等待（60s）尝试放入队列，类似我们之前自定义的拒绝策略
> - *PinPoint* 的实现，它使用了一个拒绝策略链，会逐一尝试策略链中每种拒绝策略

### newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>(),  
                                  threadFactory);  
}
```

特点：

- 核心线程数 `==` 最大线程数（没有救急线程被创建），因此也无需超时时间
- 阻塞队列是无界的，可以放任意数量的任务

适用场景：适用于任务量已知，相对耗时的任务

```java
ExecutorService pool = Executors.newFixedThreadPool(2, new ThreadFactory() {  
    private AtomicInteger t = new AtomicInteger(1);  
  
    @Override  
    public Thread newThread(Runnable r) {  
	    // 为线程取名
        return new Thread(r, "mypool_t" + t.getAndIncrement());  
    }  
});  
  
pool.execute(() -> {  
    log.debug("1");  
});  
  
pool.execute(() -> {  
    log.debug("2");  
});  
  
pool.execute(() -> {  
    log.debug("3");  
});
```

### newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
	return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
								  60L, TimeUnit.SECONDS,
								  new SynchronousQueue<Runnable>());
}
```

特点：
- 核心线程数是 0，最大线程数是 Integer.MAX_VALUE，救急线程的空闲生存时间是 60s，
	- 全部都是救急线程（60s 后可以回收）
	- 救急线程可以无限创建
- 队列采用了 SynchronousQueue，实现特点是，它**没有容量**，没有线程来取是放不进去的（一手交钱、一手交货）

> **线程数**会根据任务量不断增长，**没有上限**，当任务执行完毕，空闲 1 分钟后释放线程。

使用场景：适合任务数比较**密集**，但每个任务**执行时间较短**的情况

> 演示：SynchronousQueue

```java
public static void main(String[] args) throws InterruptedException {  
    SynchronousQueue<Integer> integers = new SynchronousQueue<>();  
  
    new Thread(() -> {  
        try {  
            log.debug("putting {} ", 1);  
            integers.put(1);  
            log.debug("{} putted...", 1);  
  
            log.debug("putting...{} ", 2);  
            integers.put(2);  
            log.debug("{} putted...", 2);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    },"t1").start();  
  
    sleep(1);  
  
    new Thread(() -> {  
        try {  
            log.debug("taking {}", 1);  
            integers.take();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    },"t2").start();  
  
    sleep(1);  
  
    new Thread(() -> {  
        try {  
            log.debug("taking {}", 2);  
            integers.take();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    },"t3").start();  
}

11:48:15.500 c.TestSynchronousQueue [t1] - putting 1 
11:48:16.500 c.TestSynchronousQueue [t2] - taking 1 
11:48:16.500 c.TestSynchronousQueue [t1] - 1 putted... 
11:48:16.500 c.TestSynchronousQueue [t1] - putting...2 
11:48:17.502 c.TestSynchronousQueue [t3] - taking 2 
11:48:17.503 c.TestSynchronousQueue [t1] - 2 putted...
```

### newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {  
    return new FinalizableDelegatedExecutorService  
        (new ThreadPoolExecutor(1, 1,  
                                0L, TimeUnit.MILLISECONDS,  
                                new LinkedBlockingQueue<Runnable>()));  
}
```

特点：

- 线程数固定为 1，且不能修改
	- FinalizableDelegatedExecutorService 应用的是**装饰器模式**，只对外暴露了 ExecutorService 接口，因此不能调用 ThreadPoolExecutor 中特有的方法
- 任务数多于 1 时，会放入**无界**队列排队
- 任务执行完毕，这唯一的线程也不会被释放
- 如果任务执行失败而终止，线程池还会新建一个线程，保证池的正常工作

适用场景：希望多个任务排队执行

> `Executors.newFixedThreadPool(1)` 初始时为1，以后还可以修改
> 
> - 对外暴露的是 ThreadPoolExecutor 对象，可以**强转**后调用 setCorePoolSize 等方法进行修改

### 提交任务

```java
// 执行任务
void execute(Runnable command);

// 提交任务 task，用返回值 Future 获得任务执行结果
<T> Future<T> submit(Callable<T> task);

// 提交 tasks 中所有任务
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
 throws InterruptedException;
 
// 提交 tasks 中所有任务，带超时时间
<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
 long timeout, TimeUnit unit)
 throws InterruptedException;
 
// 提交 tasks 中所有任务，哪个任务先成功执行完毕，返回此任务执行结果，其它任务取消
<T> T invokeAny(Collection<? extends Callable<T>> tasks)
 throws InterruptedException, ExecutionException;
```

> 无超时 invokeAll 演示

```java
private static void main(String[] args) throws InterruptedException {  
    ExecutorService pool = Executors.newFixedThreadPool(3);  
  
    List<Future<String>> futures = pool.invokeAll(Arrays.asList(  
            () -> {  
                log.debug("1");  
                Thread.sleep(1000);  
                return "1";  
            },  
            () -> {  
                log.debug("2");  
                Thread.sleep(2000);  
                return "2";  
            },  
            () -> {  
                log.debug("3");  
                Thread.sleep(3000);  
                return "3";  
            }  
    ));  
  
    futures.forEach(  
            f -> {  
                try {  
                    log.debug("{}", f.get());  
                } catch (InterruptedException | ExecutionException e) {  
                    e.printStackTrace();  
                }  
            }  
    );  
}
```

invokeAny 只有一个返回结果

```java
private static void myTest2() throws InterruptedException, ExecutionException {  
    ExecutorService pool = Executors.newFixedThreadPool(3);  
  
    String res = pool.invokeAny(Arrays.asList(  
            () -> {  
                log.debug("1");  
                Thread.sleep(1000);  
                return "1";  
            },  
            () -> {  
                log.debug("2");  
                Thread.sleep(2000);  
                return "2";  
            },  
            () -> {  
                log.debug("3");  
                Thread.sleep(3000);  
                return "3";  
            }  
    ));  
  
    log.debug("{}", res);  
}

11:20:12.540 [pool-2-thread-1] DEBUG c.TestSubmit - 1
11:20:12.540 [pool-2-thread-2] DEBUG c.TestSubmit - 2
11:20:12.540 [pool-2-thread-3] DEBUG c.TestSubmit - 3
11:20:13.556 [main] DEBUG c.TestSubmit - 1
```

### 关闭线程池

```java
public interface ExecutorService extends Executor {
	void shutdown();
	
	List<Runnable> shutdownNow();
}
```

```java
public class ThreadPoolExecutor extends AbstractExecutorService {
	public void shutdown() {
		final ReentrantLock mainLock = this.mainLock;
		mainLock.lock();
		try {
			checkShutdownAccess();
			// 修改线程池状态
			advanceRunState(SHUTDOWN);
			// 仅会打断空闲线程
			interruptIdleWorkers();
			onShutdown(); // 扩展点 ScheduledThreadPoolExecutor
		} finally {
			mainLock.unlock();
		}
		// 尝试终结(没有运行的线程可以立刻终结，如果还有运行的线程也不会打断)
		tryTerminate();
	}

	public List<Runnable> shutdownNow() {
		List<Runnable> tasks;
		final ReentrantLock mainLock = this.mainLock;
		mainLock.lock();
		try {
			checkShutdownAccess();
			// 修改线程池状态
			advanceRunState(STOP);
			// 打断所有线程
			interruptWorkers();
			// 获取队列中剩余任务
			tasks = drainQueue();
		} finally {
			mainLock.unlock();
		}
		// 尝试终结
		tryTerminate();
		return tasks;
	}
}
```

调用 `shutdown()`，线程池状态变为 SHUTDOWN

- 不会接收新任务
- 但**已提交任务会执行完**
- 此方法**不会阻塞**调用线程的执行

调用 `shutdownNow()`，线程池状态变为 STOP

- 不会接收新任务
- 会将队列中的任务返回
- 并用 interrupt 的方式**中断正在执行的任务**

其他方法：

```java
// 不在 RUNNING 状态的线程池，此方法就返回 true
boolean isShutdown();

// 线程池状态是否是 TERMINATED
boolean isTerminated();

// 调用 shutdown 后，由于调用线程并不会等待所有任务运行结束，因此如果它想在线程池 TERMINATED 后做些事情，可以利用此方法等待
boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
```

> shutdownNow()演示

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {  
	ExecutorService pool = Executors.newFixedThreadPool(2);  

	Future<Integer> result1 = pool.submit(() -> {  
		log.debug("task 1 running...");  
		Thread.sleep(1000);  
		log.debug("task 1 finish...");  
		return 1;  
	});  

	Future<Integer> result2 = pool.submit(() -> {  
		log.debug("task 2 running...");  
		Thread.sleep(1000);  
		log.debug("task 2 finish...");  
		return 2;  
	});  

	Future<Integer> result3 = pool.submit(() -> {  
		log.debug("task 3 running...");  
		Thread.sleep(1000);  
		log.debug("task 3 finish...");  
		return 3;  
	});  

	log.debug("shutdown");  
//        pool.shutdown();  
//        pool.awaitTermination(3, TimeUnit.SECONDS);  
	List<Runnable> runnables = pool.shutdownNow();  
	log.debug("other.... {}" , runnables);  
}
```

### newScheduledThreadPool

#### Timer

在『任务调度线程池』功能加入之前，可以使用 java.util.Timer 来实现定时功能，

Timer 
- 优点：简单易用，所有任务都是由同一个线程来调度，因此所有任务都是**串行执行**的，同一时间只能有一个任务在执行
- 缺点：前一个任务的延迟或异常都将会影响到之后的任务。

```java
private static void method1() {  
    Timer timer = new Timer();  
      
    TimerTask task1 = new TimerTask() {  
        @Override  
        public void run() {  
            log.debug("task 1");  
            sleep(2);  
        }  
    };  
      
    TimerTask task2 = new TimerTask() {  
        @Override  
        public void run() {  
            log.debug("task 2");  
        }  
    };  
  
    log.debug("start...");  
    timer.schedule(task1, 1000);  
    timer.schedule(task2, 1000);  
}

20:46:09.444 c.TestTimer [main] - start... 
20:46:10.447 c.TestTimer [Timer-0] - task 1 
20:46:12.448 c.TestTimer [Timer-0] - task 2
```

#### 延迟

线程数固定，任务数多于线程数时，会放入**无界**队列排队。任务执行完毕，这些线程也不会被释放。用来执行延迟或反复执行的任务

使用 ScheduledExecutorService 改写

- 可以并行执行
- 前一个任务的延迟或异常**不会**影响到之后的任务

```java
public ScheduledFuture<?> schedule(Runnable command,  
                                   long delay, TimeUnit unit);
```

```java
private static void main(String[] args){
	// 并行
	// ScheduledExecutorService pool = Executors.newScheduledThreadPool(2);
	// 串行
	ScheduledExecutorService pool = Executors.newScheduledThreadPool(1); 
	 
	pool.schedule(() -> {  
	    log.debug("task1");  
	    int i = 1 / 0;  
	}, 1, TimeUnit.SECONDS);  
	  
	pool.schedule(() -> {  
	    log.debug("task2");  
	}, 1, TimeUnit.SECONDS);
}

16:56:54.757 [pool-1-thread-1] DEBUG c.TestTimer - task1
16:56:54.757 [pool-1-thread-2] DEBUG c.TestTimer - task2
```

#### 定时

```java
// 从上一个任务的开始时间计算间隔 period
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,  
                                              long initialDelay,  
                                              long period,  
                                              TimeUnit unit);

// 从上一个任务的结束时间开始计算间隔 delay
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,  
                                                 long initialDelay,  
                                                 long delay,  
                                                 TimeUnit unit);
```

> scheduleAtFixedRate 演示
> - 任务执行时间 > 间隔时间，间隔被『撑』到了 2s

```java
private static void method3() {  
    ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);  
    log.debug("start...");  
    pool.scheduleAtFixedRate(() -> {  
        log.debug("running...");  
        sleep(2);
    }, 1, 1, TimeUnit.SECONDS);  
}

17:20:40.430 [main] DEBUG c.TestTimer - start...
17:20:41.481 [pool-1-thread-1] DEBUG c.TestTimer - running...
17:20:43.489 [pool-1-thread-1] DEBUG c.TestTimer - running...
17:20:45.491 [pool-1-thread-1] DEBUG c.TestTimer - running...
```

> scheduleWithFixedDelay 演示
> - 上一个任务结束 <-> 延时 <-> 下一个任务开始

```java
private static void method4() {  
    ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);  
    log.debug("start...");  
    pool.scheduleWithFixedDelay(() -> {  
        log.debug("running...");  
        sleep(2);  
    }, 1, 1, TimeUnit.SECONDS);  
}

17:26:33.127 [main] DEBUG c.TestTimer - start...
17:26:34.186 [pool-1-thread-1] DEBUG c.TestTimer - running...
17:26:37.203 [pool-1-thread-1] DEBUG c.TestTimer - running...
17:26:40.219 [pool-1-thread-1] DEBUG c.TestTimer - running...
```

#### 应用_定时任务

#todo

如何让每周四 18:00:00 定时执行任务？

```java
public static void main(String[] args) {  
    //  获取当前时间  
    LocalDateTime now = LocalDateTime.now();  
    System.out.println(now);  
    // 获取 周四18:00:00  
    LocalDateTime time = now.withHour(18).withMinute(0).withSecond(0).withNano(0).with(DayOfWeek.THURSDAY);  
    // 如果 当前时间 > 本周周四，必须找到下周周四  
    if (now.compareTo(time) > 0) {  
        time = time.plusWeeks(1);  
    }  
    System.out.println(time);  
    // initailDelay 代表当前时间和周四的时间差  
    // period 一周的间隔时间  
    long initailDelay = Duration.between(now, time).toMillis();  
    long period = 1000 * 60 * 60 * 24 * 7;  
  
    ScheduledExecutorService pool = Executors.newScheduledThreadPool(1);  
    pool.scheduleAtFixedRate(() -> {  
        System.out.println("running...");  
    }, initailDelay, period, TimeUnit.MILLISECONDS);  
}
```

### 处理执行任务异常

1）try-catch 主动捉异常

```java
ExecutorService pool = Executors.newFixedThreadPool(1);

pool.submit(() -> {
	try {
		log.debug("task1");
		int i = 1 / 0;
	} catch (Exception e) {
		log.error("error:", e);
	}
});
```

2）使用 Future

```java
ExecutorService pool = Executors.newFixedThreadPool(1);

Future<Boolean> f = pool.submit(() -> {
	log.debug("task1");
	int i = 1 / 0;
	return true;
});

log.debug("result:{}", f.get());
```

### Tomcat 线程池

#todo 

## Fork/Join

#todo

# ---------- 共享模型_工具_J.U.C

## AQS

*AbstractQueuedSynchronizer*，翻译 抽象队列同步器，是 阻塞式锁 和 相关的同步器工具 的框架

- 抽象类，为**构建锁和同步器**提供了一些通用功能的实现
- 在 `java.util.concurrent.locks` 包下面

![|250](assets/Pasted%20image%2020240314114025.png)

特点：

- 用 state 属性来表示**资源的状态**
	- 独占模式：是只有一个线程能够访问资源，
	- 共享模式：可以允许多个线程访问资源
- 子类需要定义如何维护这个状态，从而控制 获取锁和释放锁
	- *getState*：获取 state 状态
	- *setState*：设置 state 状态
	- *compareAndSetState*：cas 机制设置 state 状态
- 提供了基于 FIFO（先进先出）的等待队列（类似于 Monitor 的 EntryList）
- 支持**多条件变量**，实现等待、唤醒机制，（类似于 Monitor 的 WaitSet）

> Monitor 是 c++实现，AQS 是纯 Java 实现

AQS 使用了模板方法模式，自定义同步器时需要重写下面几个 AQS 提供的钩子方法

> 钩子方法：一种被声明在**抽象类**中的方法，一般使用 `protected` 关键字修饰，它可以是空方法（由子类实现），也可以是默认实现的方法。模板设计模式通过钩子方法控制固定步骤的实现。

```java
//独占方式。尝试获取资源，成功则返回true，失败则返回false。
protected boolean tryAcquire(int)

//独占方式。尝试释放资源，成功则返回true，失败则返回false。
protected boolean tryRelease(int)

//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
protected int tryAcquireShared(int)

//共享方式。尝试释放资源，成功则返回true，失败则返回false。
protected boolean tryReleaseShared(int)

//该线程是否正在独占资源。只有用到condition才需要去实现它。
protected boolean isHeldExclusively()
```

- 默认实现都是抛出 UnsupportedOperationException
- 除了上面的钩子方法外，AQS 类中的**其他方法都是 final**，无法被子类重写

### 不可重入锁实现

#todo 


## ReentrantLock 原理

![|500](assets/Pasted%20image%2020240314162557.png)

### 非公平锁原理

默认是非公平锁实现

```java
public ReentrantLock() {
	// NonfairSync 继承自 AQS
	sync = new NonfairSync();
}

 static final class NonfairSync extends Sync {
	private static final long serialVersionUID = 7316153563782823691L;

	/**
	 * Performs lock.  Try immediate barge, backing up to normal
	 * acquire on failure.
	 */
	final void lock() {
		if (compareAndSetState(0, 1))
			setExclusiveOwnerThread(Thread.currentThread());
		else
			acquire(1);
	}

	protected final boolean tryAcquire(int acquires) {
		return nonfairTryAcquire(acquires);
	}
}
```

无竞争时，当前独占线程是 Thread-0

![|500](assets/Pasted%20image%2020240314163224.png)

第一个竞争出现时，

![|500](assets/Pasted%20image%2020240314164351.png)

- Thread-1 `compareAndSetState(0, 1)` 失败

```java
public final void acquire(int arg) {  
    if (!tryAcquire(arg) &&  
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))  
        selfInterrupt();  
}
```

## Semaphore

#todo 


## CountdownLatch

> 翻译：倒计时锁

用来进行线程同步协作，等待所有线程完成倒计时

其中构造参数用来初始化等待计数值，await() 用来等待计数归零，countDown() 用来让计数减一




## CyclicBarrier














# ---------- 模式

## 同步模式_保护性暂停

即 Guarded Suspension，用在 一个线程 **等待** 另一个线程 的【**执行结果**】

注意：【结果等待着】和【结果生产者】**一一对应**

> 因为要等待另一方的结果，因此归类到同步模式

要点：
- 让他们关联同一个 **GuardedObject**，其中 response 存放传递的结果
- 如果有结果**不断**从一个线程到另一个线程，可以使用**消息队列**（见生产者/消费者）

JDK 中，**join** 的实现、**Future** 的实现，采用的就是此模式

![|500](assets/Pasted%20image%2020240307195230.png)

### GuardedObject 实现

```java
class GuardedObject {
	// 存放需要传递的结果
    private Object response;

    private final Object lock = new Object();

    public Object get() {
        synchronized (lock) {
            // 条件不满足则等待
            while (response == null) {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            return response;
        }
    }

    public void complete(Object response) {
        synchronized (lock) {
            // 条件满足，通知等待线程
            this.response = response;
            lock.notifyAll();
        }
    }
}
```

> 演示：子线程执行下载，主线程阻塞等待

```java
public static void main(String[] args) {
	GuardedObject guardedObject = new GuardedObject();

	// 开启一个线程去下载资源，下载完毕后通知主线程
	new Thread(() -> {
		try {
			List<String> response = download();
			log.debug("download complete...");
			guardedObject.complete(response);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}).start();

	log.debug("waiting...");
	Object response = guardedObject.get();
	log.debug("get response: [{}] lines", ((List<String>) response).size());
}

21:13:17.285 [main] DEBUG c.TestGuardedObject - waiting...
21:13:18.255 [Thread-0] DEBUG c.TestGuardedObject - download complete...
21:13:18.255 [main] DEBUG c.TestGuardedObject - get response: [3] lines
```

### 超时版 GuardedObject

**虚假唤醒**：在等待时间内，等待的线程被唤醒了，但是没有结果
- 等待结果的线程应该继续等待，并且要减掉已经等待的时间

```java
/**
 * 添加超时处理
 */
@Slf4j(topic = "c.GuardedObjectV2")
class GuardedObjectV2 {

    private Object response;
    private final Object lock = new Object();

    public Object get(long millis) {
        synchronized (lock) {
            // 1) 记录最初时间
            long last = System.currentTimeMillis();
            // 2) 已经经历的时间
            long timePassed = 0;
            while (response == null) {
                // 4) 假设 millis 是 1000，结果在 400 时唤醒了，那么还有 600 要等
                long waitTime = millis - timePassed;
                log.debug("waitTime: {}", waitTime);
                if (waitTime <= 0) {
                    log.debug("break...");
                    break;
                }
                try {
                    lock.wait(waitTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 3) 如果提前被唤醒，这时已经经历的时间假设为 400
                timePassed = System.currentTimeMillis() - last;
                log.debug("timePassed: {}, object is null {}", timePassed, response == null);
            }
            return response;
        }
    }

    public void complete(Object response) {
        synchronized (lock) {
            this.response = response;
            log.debug("notify...");
            lock.notifyAll();
        }
    }
}
```

> 演示：超时版，虚假唤醒

```java
public static void main(String[] args) {
	GuardedObjectV2 v2 = new GuardedObjectV2();
	new Thread(() -> {
		// 模拟虚假唤醒
		sleep(1);
		v2.complete(null);
		// 唤醒
		sleep(1);
		v2.complete(Arrays.asList("a", "b", "c"));
	}).start();

	Object response = v2.get(2500);
	if (response != null) {
		log.debug("get response: [{}] lines", ((List<String>) response).size());
	} else {
		log.debug("can't get response");
	}
}
```

### 多任务版 GuardedObject

图中 Futures 就好比居民楼一层的信箱（每个信箱有房间编号），左侧的 t0，t2，t4 就好比等待邮件的居民，右侧的 t1，t3，t5 就好比邮递员

![](assets/Pasted%20image%2020240308114135.png)

如果需要在多个类之间使用 GuardedObject 对象，作为参数传递不是很方便，因此设计一个用来解耦的**中间类**，这样不仅能够解耦【结果等待者】和【结果生产者】，还能够同时支持**多个任务**的管理

---
> postman：结果产生者。people：结果等待者

新增 id 标识 Guarded Object

```java
// 增加超时效果
class GuardedObject {

    // 标识 Guarded Object
    private int id;

    public GuardedObject(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    ......
}
```

中间解耦类

```java
class Mailboxes {
    private static Map<Integer, GuardedObject> boxes = new Hashtable<>();

    private static int id = 1;
    // 产生唯一 id
    private static synchronized int generateId() {
        return id++;
    }

    public static GuardedObject getGuardedObject(int id) {
        return boxes.remove(id);
    }

    public static GuardedObject createGuardedObject() {
        GuardedObject go = new GuardedObject(generateId());
        boxes.put(go.getId(), go);
        return go;
    }

    public static Set<Integer> getIds() {
        return boxes.keySet();
    }
}
```

业务相关类

```java
@Slf4j(topic = "c.People")
class People extends Thread{
    @Override
    public void run() {
        // 收信
        GuardedObject guardedObject = Mailboxes.createGuardedObject();
        log.debug("开始收信 id:{}", guardedObject.getId());
        Object mail = guardedObject.get(5000);
        log.debug("收到信 id:{}, 内容:{}", guardedObject.getId(), mail);
    }
}

@Slf4j(topic = "c.Postman")
class Postman extends Thread {
    private int id;
    private String mail;

    public Postman(int id, String mail) {
        this.id = id;
        this.mail = mail;
    }

    @Override
    public void run() {
        GuardedObject guardedObject = Mailboxes.getGuardedObject(id);
        log.debug("送信 id:{}, 内容:{}", id, mail);
        guardedObject.complete(mail);
    }
}
```

测试

```java
public static void main(String[] args) throws InterruptedException {
	for (int i = 0; i < 3; i++) {
		new People().start();
	}
	Sleeper.sleep(1);
	for (Integer id : Mailboxes.getIds()) {
		new Postman(id, "内容" + id).start();
	}
}

10:35:05.689 c.People [Thread-1] - 开始收信 id:3
10:35:05.689 c.People [Thread-2] - 开始收信 id:1
10:35:05.689 c.People [Thread-0] - 开始收信 id:2
10:35:06.688 c.Postman [Thread-4] - 送信 id:2, 内容:内容2
10:35:06.688 c.Postman [Thread-5] - 送信 id:1, 内容:内容1
10:35:06.688 c.People [Thread-0] - 收到信 id:2, 内容:内容2
10:35:06.688 c.People [Thread-2] - 收到信 id:1, 内容:内容1
10:35:06.688 c.Postman [Thread-3] - 送信 id:3, 内容:内容3
10:35:06.689 c.People [Thread-1] - 收到信 id:3, 内容:内容3
```

## 同步模式_Balking

Balking （犹豫）模式用在一个线程发现另一个线程或本线程已经做了某一件相同的事，那么本线程就无需再做了，直接结束返回

#todo

## 同步模式_顺序控制

#todo

## 异步模式_生产者/消费者

### 定义

> 与前面的保护性暂停中的 GuardObject 不同，**不需要**产生结果和消费结果的线程一一对应
> 
> JDK 中各种阻塞队列，采用的就是这种模式

思路：
- 【生产者】仅负责产生结果数据，不关心数据该如何处理
- 【消费者】专心处理结果数据
- 【消费队列】可以用来平衡生产和消费的线程资源
	- 有容量限制，满时不会再加入数据，空时不会再消耗数据

![](assets/Pasted%20image%2020240308131915.png)

### 实现

消息

```java
@Data
final class Message {
    private int id;
    private Object value;

    public Message(int id, Object value) {
        this.id = id;
        this.value = value;
    }
}
```

消息队列

```java
@Slf4j(topic = "c.MessageQueue")
class MessageQueue {
    // 消息的队列集合
    private LinkedList<Message> list = new LinkedList<>();
    // 队列容量
    private int capcity;

    public MessageQueue(int capcity) {
        this.capcity = capcity;
    }

    // 获取消息
    public Message take() {
        synchronized (list) {
            // 检查队列是否为空
            while (list.isEmpty()) {
                try {
                    log.debug("队列为空, 消费者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 从队列头部获取消息并返回
            Message message = list.removeFirst();
            log.debug("已消费消息 {}", message);
            list.notifyAll();
            return message;
        }
    }
    
    // 存入消息
    public void put(Message message) {
        synchronized (list) {
            // 检查对象是否已满
            while (list.size() == capcity) {
                try {
                    log.debug("队列已满, 生产者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 将消息加入队列尾部
            list.addLast(message);
            log.debug("已生产消息 {}", message);
            list.notifyAll();
        }
    }
}
```

测试

```java
public static void main(String[] args) {  
    MessageQueue queue = new MessageQueue(2);  
  
    for (int i = 0; i < 3; i++) {  
        int id = i;  
        new Thread(() -> {  
            queue.put(new Message(id, "值" + id));  
        }, "生产者" + i).start();  
    }  
  
    new Thread(() -> {  
        while (true) {  
            sleep(1);  
            Message message = queue.take();  
        }  
    }, "消费者").start();  
}

14:05:13.256 [生产者0] DEBUG c.MessageQueue - 已生产消息 Message(id=0, value=值0)
14:05:13.265 [生产者1] DEBUG c.MessageQueue - 已生产消息 Message(id=1, value=值1)
14:05:13.265 [生产者2] DEBUG c.MessageQueue - 队列已满, 生产者线程等待
14:05:14.243 [消费者] DEBUG c.MessageQueue - 已消费消息 Message(id=0, value=值0)
14:05:14.243 [生产者2] DEBUG c.MessageQueue - 已生产消息 Message(id=2, value=值2)
14:05:15.244 [消费者] DEBUG c.MessageQueue - 已消费消息 Message(id=1, value=值1)
14:05:16.257 [消费者] DEBUG c.MessageQueue - 已消费消息 Message(id=2, value=值2)
14:05:17.271 [消费者] DEBUG c.MessageQueue - 队列为空, 消费者线程等待
```

## 异步模式_工作线程

让有限的工作线程（Worker Thread）来轮流异步处理无限多的任务。也可以将其归类为分工模式，它的典型实现就是**线程池**，也体现了经典设计模式中的**享元模式**。

> 例如，海底捞的服务员（线程），轮流处理每位客人的点餐（任务），如果为每位客人都配一名专属的服务员，那么成本就太高了（对比另一种多线程设计模式：Thread-Per-Message）

注意，不同任务类型应该使用不同的线程池，这样能够避免饥饿，并能提升效率

> 例如，如果一个餐馆的工人既要招呼客人（任务类型A），又要到后厨做菜（任务类型B）显然效率不咋地，分成服务员（线程池A）与厨师（线程池B）更为合理，当然你能想到更细致的分工

### 饥饿

**固定**大小线程池会有饥饿现象

> 线程池中线程不足导致的饥饿现象
> 
> - 两个工人是同一个线程池中的两个线程
> - 他们要做的事情是：为客人点餐和到后厨做菜，这是两个阶段的工作
> 	- 客人点餐：必须先点完餐，等菜做好，上菜，在此期间处理点餐的工人必须等待
> 	- 后厨做菜：没啥说的，做就是了
> - 比如工人A 处理了点餐任务，接下来它要等着 工人B 把菜做好，然后上菜，他俩也配合的蛮好
> - 但现在同时来了两个客人，这个时候工人A 和工人B **都去处理点餐了，这时没人做饭了**，饥饿

```java
public static void test1(){  
    ExecutorService pool = Executors.newFixedThreadPool(2);  
  
    pool.execute(() -> {  
        log.debug("处理点餐...");  
        Future<String> f = pool.submit(() -> {  
            log.debug("做菜");  
            return cooking();  
        });  
        try {  
            log.debug("上菜: {}", f.get());  
        } catch (InterruptedException | ExecutionException e) {  
            e.printStackTrace();  
        }  
    });  
  
    pool.execute(() -> {  
        log.debug("处理点餐...");  
        Future<String> f = pool.submit(() -> {  
            log.debug("做菜");  
            return cooking();  
        });  
        try {  
            log.debug("上菜: {}", f.get());  
        } catch (InterruptedException | ExecutionException e) {  
            e.printStackTrace();  
        }  
    });  
  
    // 15:25:34.237 [pool-1-thread-1] DEBUG c.TestDeadLock - 处理点餐...  
    // 15:25:34.237 [pool-1-thread-2] DEBUG c.TestDeadLock - 处理点餐...  
}
```

增加线程池的大小，不是根本解决方案

```java
ExecutorService pool = Executors.newFixedThreadPool(3);  

16:23:46.999 [pool-1-thread-1] DEBUG c.TestDeadLock - 处理点餐...
16:23:46.999 [pool-1-thread-2] DEBUG c.TestDeadLock - 处理点餐...
16:23:47.009 [pool-1-thread-3] DEBUG c.TestDeadLock - 做菜
16:23:47.009 [pool-1-thread-3] DEBUG c.TestDeadLock - 做菜
16:23:47.009 [pool-1-thread-2] DEBUG c.TestDeadLock - 上菜: 宫保鸡丁
16:23:47.009 [pool-1-thread-1] DEBUG c.TestDeadLock - 上菜: 宫保鸡丁
```

不同的任务类型，采用不同的线程池

```java
public static void test2(){  
    ExecutorService waiterPool = Executors.newFixedThreadPool(1);  
    ExecutorService cookPool = Executors.newFixedThreadPool(1);  
  
    waiterPool.execute(() -> {  
        log.debug("处理点餐...");  
        Future<String> f = cookPool.submit(() -> {  
            log.debug("做菜");  
            return cooking();  
        });  
        try {  
            log.debug("上菜: {}", f.get());  
        } catch (InterruptedException | ExecutionException e) {  
            e.printStackTrace();  
        }  
    });  
  
    waiterPool.execute(() -> {  
        log.debug("处理点餐...");  
        Future<String> f = cookPool.submit(() -> {  
            log.debug("做菜");  
            return cooking();  
        });  
        try {  
            log.debug("上菜: {}", f.get());  
        } catch (InterruptedException | ExecutionException e) {  
            e.printStackTrace();  
        }  
    });  
}

16:32:37.009 [pool-1-thread-1] DEBUG c.TestDeadLock - 处理点餐...
16:32:37.018 [pool-2-thread-1] DEBUG c.TestDeadLock - 做菜
16:32:37.018 [pool-1-thread-1] DEBUG c.TestDeadLock - 上菜: 宫保鸡丁
16:32:37.019 [pool-1-thread-1] DEBUG c.TestDeadLock - 处理点餐...
16:32:37.019 [pool-2-thread-1] DEBUG c.TestDeadLock - 做菜
16:32:37.019 [pool-1-thread-1] DEBUG c.TestDeadLock - 上菜: 烤鸡翅
```

### 创建多少线程合适？

- 过小会导致程序不能充分地利用系统资源、容易导致饥饿
- 过大会导致更多的线程上下文切换，占用更多内存

1）CPU 密集型运算

`cpu 核数 + 1` 能够实现最优的 CPU 利用率

- +1 是保证当线程由于页缺失故障（操作系统）或其它原因导致暂停时，额外的这个线程就能顶上去，保证 CPU 时钟周期不被浪费

2） I/O 密集型运算

CPU 不总是处于繁忙状态，例如，当你执行业务计算时，这时候会使用 CPU 资源，但当你执行 I/O 操作时、远程RPC 调用时，包括进行数据库操作时，这时候 CPU 就闲下来了，你可以利用多线程提高它的利用率。

公式：`线程数 = 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间`

> 例如 4 核 CPU 计算时间是 50% ，其它等待时间是 50%，期望 cpu 被 100% 利用，套用公式 `4 * 100% * 100% / 50% = 8`
> 
> 例如 4 核 CPU 计算时间是 10% ，其它等待时间是 90%，期望 cpu 被 100% 利用，套用公式 `4 * 100% * 100% / 10% = 40`

## 终止模式_两阶段终止

Two Phase Termination，在一个线程 T1 中“优雅”终止线程 T2
- 【优雅】指的是给 T2 一个**料理后事**的机会

### 错误思路

使用线程对象的 stop() 方法停止线程
- 会真正杀死线程，如果这时线程**锁住了共享资源**，那么它被杀死后再也没有机会释放锁，其它线程将永远无法获取锁

使用 `System.exit(int)` 方法停止线程
- 目的仅是停止一个线程，但这种做法会让**整个程序都停止**

### isInterrupted()实现

![|500](assets/Pasted%20image%2020240303202847.png)

思想：
- 把代码包在 while 中，想要结束线程就 break

```java
@Slf4j
public class TwoPhaseTermination {
    private Thread monitor;

    public void start() {
        monitor = new Thread(() -> {
            while (true) {
                Thread current = Thread.currentThread();
                if (current.isInterrupted()){
                    log.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.debug("执行监控记录");
                } catch (InterruptedException e) {
                    log.debug("sleep被打断");
                    // 重新设置打断标记
                    current.interrupt();
                }
            }
        });
        monitor.start();
    }

    public void stop(){
        monitor.interrupt();
    }

    public static void main(String[] args) throws InterruptedException {
        TwoPhaseTermination tpt = new TwoPhaseTermination();
        tpt.start();

        Thread.sleep(3500);
        tpt.stop();
    }
}
```

### volatile 停止标记实现

通过标志位 stop 控制线程中断，不需要重新设置打断标记

`stop()` 中设置了 `stop = true`，t 不会立即中断，如果睡眠时间过长不想等，就 `t.interrupt()`

> 标志位不加 volatile，t 也会中断，IO 阻塞导致线程切换、线程 sleep 导致 CPU 空闲，都会导致重新从主存读取值，保险起见还是加上

```java
class TPTVolatile {
    private Thread t;
    private volatile boolean stop = false;

    public void start() {
        t = new Thread(() -> {
            while (true) {
                Thread current = Thread.currentThread();
                if (stop) {
                    log.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.debug("将结果保存");
                } catch (InterruptedException e) {
                    log.debug("sleep被打断");
                }
            }
        }, "监控线程");
        t.start();
    }

    public void stop() {
        stop = true;
        // t.interrupt();
    }
}
```

## 享元模式

> Java 设计模式，归类 Structual patterns

Flyweight pattern. 当需要**重用数量有限的同一类对象**时

体现：包装类、String 串池、BigDecimal BigInteger

### 实现

一个线上商城应用，QPS 达到数千，如果每次都重新创建和关闭数据库连接，性能会受到极大影响。 

这时**预先创建好一批连接，放入连接池**。一次请求到达后，从连接池**获取连接**，使用完毕后再**还回连接池**，这样既节约了连接的创建和关闭时间，也实现了连接的重用，不至于让庞大的连接数压垮数据库。


```java
public class Test3 {
    public static void main(String[] args) {
        Pool pool = new Pool(2);
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                Connection conn = pool.borrow();
                try {
                    Thread.sleep(new Random().nextInt(1000));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                pool.free(conn);
            }).start();
        }
    }
}

@Slf4j(topic = "c.Pool")
class Pool {
    // 1. 连接池大小
    private final int poolSize;

    // 2. 连接对象数组
    private Connection[] connections;

    // 3. 连接状态数组 0 表示空闲， 1 表示繁忙
    private AtomicIntegerArray states;

    // 4. 构造方法初始化
    public Pool(int poolSize) {
        this.poolSize = poolSize;
        this.connections = new Connection[poolSize];
        this.states = new AtomicIntegerArray(new int[poolSize]);
        for (int i = 0; i < poolSize; i++) {
            connections[i] = new MockConnection("连接" + (i+1));
        }
    }

    // 5. 借连接
    public Connection borrow() {
        while(true) {
            for (int i = 0; i < poolSize; i++) {
                // 获取空闲连接
                if(states.get(i) == 0) {
                    if (states.compareAndSet(i, 0, 1)) {
                        log.debug("borrow {}", connections[i]);
                        return connections[i];
                    }
                }
            }
            // 如果没有空闲连接，当前线程进入等待
            synchronized (this) {
                try {
                    log.debug("wait...");
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    // 6. 归还连接
    public void free(Connection conn) {
        for (int i = 0; i < poolSize; i++) {
            if (connections[i] == conn) {
                states.set(i, 0);
                synchronized (this) {
                    log.debug("free {}", conn);
                    this.notifyAll();
                }
                break;
            }
        }
    }
}

class MockConnection implements Connection {

    private String name;

    public MockConnection(String name) {
        this.name = name;
    }
}
```

以上实现没有考虑：

- 连接的动态增长与收缩
- 连接保活（可用性检测）
- 等待超时处理
- 分布式 hash

对于关系型数据库，有比较成熟的连接池实现，例如c3p0, druid等 

对于更通用的对象池，可以考虑使用 apache commons pool，例如 redis 连接池可以参考 jedis 中关于连接池的实现





