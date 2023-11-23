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
# Java线程
## 创建和运行线程
### Thread和Runnable
方法一：继承Thread
```java
// 创建线程对象
Thread t = new Thread() { // 匿名内部类
    @Override
    public void run() {
        // 要执行的任务
    }
};
// 启动线程
t.start();
```
方法二：实现Runnable接口
```java
Runnable runnable = new Runnable() { // 匿名内部类
    @Override
    public void run(){
        // 要执行的任务
    }
};
// 创建线程对象
Thread t = new Thread( runnable );
// 启动线程
t.start(); 
```

**Thread 与 Runnable 的关系**
分析 Thread 的源码，理清它与 Runnable 的关系
1) Runnable源码：函数式接口
```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
2) Thread源码：如果target不为空，就调用target的Runnable方法
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
- 方法1 是把线程和任务合并在了一起，方法2 是把线程和任务分开了 
- 用 Runnable 更容易与线程池等高级API 配合 
- 用 Runnable 让任务类脱离了 Thread 继承体系，更灵活

### FutureTask 配合 Thread
Callable接口：对比Runnable接口，call()方法有返回值且方法上抛出了异常
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
FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况
```java
@Slf4j(topic = "c.TestFutureTask")
public class TestFutureTask {
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
}
```
## 查看进程线程的方法

- 任务管理器可以查看进程和线程数，也可以用来杀死进程
- tasklist 查看进程 
   - `tasklist` | `findstr` (查找关键字)

- taskkill 杀死进程 
   - `taskkill` `/F`(彻底杀死）`/PID`(进程PID)

Linux
- `ps -fe` 查看所有进程
- `ps -fT -p`  查看某个进程（PID）的所有线程
- `kill` 杀死进程 top 按大写 H 切换是否显示线程
- `top -H -p`  查看某个进程（PID）的所有线程

Java
- `jps` 命令查看所有 Java 进程
- `jstack`查看某个 Java 进程（PID）的所有线程状态
- `jconsole` 来查看某个 Java 进程中线程的运行情况（图形界面）

