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
从方法调用的角度来说，如果

- 需要等待结果返回，才能继续运行就是**同步**
- **不需要等待结果返回**，就能继续运行就是**异步**

注意：**同步在多线程中还有另外一层意思，是让多个线程步调一致**

多线程可以让方法执行变为异步的（即不要巴巴干等着）、比如说读取磁盘文件时，假设读取操作花费了 5 秒钟，如果没有线程调度机制，这 5 秒 cpu 什么都做不了，其它代码都得暂停...
- 比如在项目中，视频文件需要转换格式等操作比较费时，这时开一个新线程处理视频转换，避免阻塞主线程
- tomcat 的异步 servlet 也是类似的目的，让用户线程处理耗时较长的操作，避免阻塞 tomcat 的工作线程
- ui 程序中，开线程进行其他操作，避免阻塞 ui 线程

1. **单核 cpu** 下，多线程不能实际提高程序运行效率，只是为了能够在不同的任务之间切换，不同线程轮流使用 cpu ，**不至于一个线程总占用 cpu，别的线程没法干活**
2. **多核 cpu** 可以并行跑多个线程，但能否提高程序运行效率还是要分情况的 
   1. 有些任务，经过精心设计，将任务拆分，并行执行，当然可以提高程序的运行效率。但不是所有计算任务都能拆分（参考后文的【阿姆达尔定律】）
   2. 也不是所有任务都需要拆分，任务的目的如果不同，谈拆分和效率没啥意义
3. IO 操作不占用 cpu，只是我们一般拷贝文件使用的是【阻塞 IO】，这时相当于线程虽然不用 cpu，但需要一 直等待 IO 结束，没能充分利用线程。所以才有后面的【非阻塞 IO】和【异步 IO】优化。

# ---------- Java 线程

## 创建和运行线程

### 方式一：Thread

方法一：继承 Thread

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

*Thread Context Switch*

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

start()
- 功能：启动一个**新线程**，在新的线程运行 run 方法中的代码
- 注意点：
	- start 方法只是让线程进入**就绪**，里面代码不一定立刻运行（CPU 的时间片还没分给它）。
	- 每个线程对象的 start 方法**只能调用一次**，如果调用了多次会出现 `IllegalThreadStateException`

run()
- 功能：新线程启动后会调用的方法
- 注意点：
	- 如果在构造 Thread 对象时传递了 Runnable 参数，则线程启动后会**调用 Runnable 中的 run 方法**，否则默认不执行任何操作。
	- 可以创建 Thread 的**子类对象，来覆盖默认行为**

两者对比
- 直接调用 run 是在主线程中执行了 run，**没有启动新的线程**
- 使用 start 是**启动新的线程**，通过新的线程间接执行 run 中的代码

### sleep() 与 yield()

`sleep()`
- **static**
- 功能：让当前执行的线程休眠 n 毫秒，休眠时让出 cpu 的时间片给其它线程
- 注意点：
	- 调用 sleep 会让当前线程从 `Running` 进入 **`Timed Waiting` 状态（阻塞）**
	- 其它线程可以使用 `interrupt()` 方法**打断正在睡眠的线程**，这时 sleep 方法会抛出 `InterruptedException`
	- 睡眠结束后的线程未必会立刻得到执行
	- **建议用 `TimeUnit` 的 `sleep()`** 代替 Thread 的 `sleep()` 来获得更好的可读性

`yield()`
- **static**
- 功能：提示线程调度器让出当前线程对 CPU 的使用
- 注意：
	- 调用 yield 会让当前线程从 `Running` 进入 **`Runnable` 状态（就绪）**，然后调度执行其它线程
	- 具体的实现依赖于操作系统的任务调度器

