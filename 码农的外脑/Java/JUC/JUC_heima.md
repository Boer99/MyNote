# ---------- 进程与线程
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

- await 前需要**先获得锁**
- await 执行后，会**释放锁**，进入 conditionObject 等待
- await 的线程被唤醒（或打断、或超时）取**重新竞争** lock 锁
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

JMM 即 Java Memory Model，它定义了主存、工作内存抽象概念，底层对应着 CPU 寄存器、缓存、硬件内存、CPU 指令优化等。

JMM 体现在以下几个方面

- 原子性 - 保证指令不会受到线程上下文切换的影响
- 可见性 - 保证指令不会受 cpu 缓存的影响
- 有序性 - 保证指令不会受 cpu 指令并行优化的影响

## 可见性

### 退不出的循环

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

> #todo while 里面不能有输出，sout 里面有同步代码块



# ---------- 模式

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

## 同步模式_保护性暂停

### 定义

即 Guarded Suspension，用在 一个线程 **等待** 另一个线程 的【**执行结果**】

注意：【结果等待着】和【结果生产者】**一一对应**

> 因为要等待另一方的结果，因此归类到同步模式

要点：
- 让他们关联同一个 **GuardedObject**，其中 response 存放传递的结果
- 如果有结果**不断**从一个线程到另一个线程，可以使用**消息队列**（见生产者/消费者）

JDK 中，**join** 的实现、**Future** 的实现，采用的就是此模式

![](assets/Pasted%20image%2020240307195230.png)

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

## 同步模式之顺序控制

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