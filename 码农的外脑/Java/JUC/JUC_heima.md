# 并发编程
## 并行与并发

单核cpu下，线程实际还是**串行执行**的。操作系统中有一个组件叫做任务调度器，将 cpu 的时间片（windows 下时间片最小约为 15 毫秒）分给不同的程序使用，只是由于 cpu 在线程间（时间片很短）的切换非常快，人类感觉是同时运行的 。总结为一句话就是：**微观串行，宏观并行** 。


一般会将这种线程轮流使用 CPU 的做法称为并发，**concurrent**

| **CPU** | **时间片 1** | **时间片 2** | **时间片 3** | **时间片 4** |
| --- | --- | --- | --- | --- |
| core | 线程 1 | 线程 2 | 线程 3 | 线程 4 |

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

1. **单核 cpu** 下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu ，**不至于一个线程总占用 cpu，别的线程没法干活**
2. **多核 cpu** 可以并行跑多个线程，但能否提高程序运行效率还是要分情况的 
   1. 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任务都能拆分（参考后文的【阿姆达尔定律】）
   2. 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
3. IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO】，这时相当于线程虽然不用 cpu，但需要一 直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO】和【异步 IO】优化。

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

*Java Virtual Machine Stacks （Java 虚拟机栈）*

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

### Thead 类 API

| 方法                                          | 说明                                                          |
| ------------------------------------------- | ----------------------------------------------------------- |
| public void start()                         | 启动一个新线程，Java虚拟机调用此线程的 run 方法                                |
| public void run()                           | 线程启动后调用该方法                                                  |
| public void setName(String name)            | 给当前线程取名字                                                    |
| public void getName()                       | 获取当前线程的名字 线程存在默认名称：子线程是 Thread-索引，主线程是 main                 |
| public static Thread currentThread()        | 获取当前线程对象，代码在哪个线程中执行                                         |
| public static void sleep(long time)         | 让当前线程休眠多少毫秒再继续执行 **Thread.sleep(0)** : 让操作系统立刻重新进行一次 CPU 竞争 |
| public static native void yield()           | 提示线程调度器让出当前线程对 CPU 的使用                                      |
| public final int getPriority()              | 返回此线程的优先级                                                   |
| public final void setPriority(int priority) | 更改此线程的优先级，常用 1 5 10                                         |
| public void interrupt()                     | 中断这个线程，异常处理机制                                               |
| public static boolean interrupted()         | 判断当前线程是否被打断，清除打断标记                                          |
| public boolean isInterrupted()              | 判断当前线程是否被打断，不清除打断标记                                         |
| public final void join()                    | 等待这个线程结束                                                    |
| public final void join(long millis)         | 等待这个线程死亡 millis 毫秒，0 意味着永远等待                                |
| public final native boolean isAlive()       | 线程是否存活（还没有运行完毕）                                             |
| public final void setDaemon(boolean on)     | 将此线程标记为守护线程或用户线程                                            |

### start() 与 run()

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

### sleep() 与 yield()

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

设置线程的优先级

```java
public final void setPriority(int newPriority){}
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

- 功能：等待线程运行结束
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

#### interrupt()

```java
public void interrupt()
```

- 功能：打断线程
- 注意：
	- 打断线程正在 sleep，wait，join ：被打断的线程会抛出 `InterruptedException`，并**清除** 打断标记
	- 打断 running 线程：设置 打断标记
	- 打断park 的线程：设置 打断标记

```java
public boolean isInterrupted()
```

- 功能：判断线程是否被打断

```java
public static boolean interrupted()
```

- 功能：判断当前线程是否被打断
- 注意：会**清除**打断标记

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

#### 两阶段终止模式

![|500](assets/Pasted%20image%2020240303202847.png)

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

## 不推荐的方法

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

## 五种状态

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

## 六种状态

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

## syncronized

为了**避免临界区的竞态条件发生**，有多种手段可以达到目的。

- **阻塞式**的解决方案：synchronized，Lock
- **非阻塞式**的解决方案：原子变量

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

### 面向对象改进

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


## 变量的线程安全分析

“成员变量”和“静态变量”是否线程安全？
- 没有被共享，则线程安全
- 被**共享**了，
	- 只有读操作，则线程安全
	- 有**读写**操作，则这段代码是临界区，需要考虑线程安全

“局部变量”是否线程安全？
- 本身是线程安全的（存在栈上）
- 但其**引用的对象则未必**（堆上就可能被共享），要看该对象是否**逃离方法的作用范围**
	- 未逃离：线程安全
	- **逃离**：需要考虑线程安全

> #Boer 逃离方法的作用范围，通俗点，就是看堆上的对象**会不会被其他线程共享**

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

### 共享的引用类型成员变量

> 演示：共享的引用类型成员变量，多线程对其读写，线程不安全

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

### 共享的引用类型局部变量

> 先演示个不共享的

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
> 演示共享的

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

```java
Hashtable table = new Hashtable();
// 线程1，线程2
if( table.get("key") == null) {
 table.put("key", value);
}
```

![|500](assets/Pasted%20image%2020240306005256.png)

### 不可变类线程安全性

String、Integer 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的

有同学或许有疑问，String 有 replace，substring 等方法【可以】改变值啊，那么这些方法又是如何保证线程安
全的呢？






