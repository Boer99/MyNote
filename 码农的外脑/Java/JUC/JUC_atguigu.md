**参考资料**
尚硅谷2022版JUC并发编程（对标阿里P6-P7）
讲师：周阳
[https://www.bilibili.com/video/BV1ar4y1x727?p=4&share_source=copy_web&vd_source=eb6ade7f410b79ba275b327b6011d56e](https://www.bilibili.com/video/BV1ar4y1x727?p=4&share_source=copy_web&vd_source=eb6ade7f410b79ba275b327b6011d56e)

# 线程基础知识复习

### 并发相关Java包

- `java.util.concurrent`
- `java.util.concurrent.atomic`
- `java.util.concurrent.locks`

### 并发与并行

并发：
是在**同一实体**上的多个事件,
是在同一台处理器上“同时”处理多个任务，
同一时刻，其实是只有一个事件在发生。

并行：
是在**不同实体**上的多个事件，
是在多台处理器上同时处理多个任务，
同一时刻，大家都真的在做事情，你做你的，我做我的

### 进程、线程

进程：系统中运行的一个应用程序就是一个进程，每一个进程都有它自己的内存空间和系统资源。

线程：也被称为轻量级进程，在同一个进程内基本会有1一个或多个线程，是大多数操作系统进行调度的基本单元。

### 管程

Monitor（监视器），也就是我们平时说的**锁**

Monitor其实是一种**同步机制**，他的义务是保证（同一时间）只有一个线程可以访问被保护的数据和代码。

JVM中同步是基于**进入和退出监视器对象**（Monitor，管程对象）来实现的，**每个对象实例都会有一个Monitor对象**

Monitor对象会和Java对象**一同创建并销毁**，它底层是由C++语言来实现的。

> JVM第三版：
> 
> 方法级的同步是隐式的，无须通过字节码指令来控制，它实现在方法调用和返
> 操作之中。虚拟机可以从方法常量池中的方法表结构中的ACC SYNCHRONIZED访问标志得知一个方法是否被声明为同步方法。当方法调用时，调用指令将会检查方法的ACC SYNCHRONIZED访问标志是否被设置，如果设置了，==执行线程就要求先成功持有管程，然后才能执行方法，最后当方法完成 (无论是正常完成还是非正常完成）时释放管程。在方法执行期间，执行线程持有了管程，其他任何线程都无法再获取到同一个管程==。如果一个同步方法执行期间抛出了异常，并且在方法内部无法处理此异常，那这个同步方法所持有的管程将在异常抛到同步方法边界之外时自动释放。

--
### 线程的状态

进入 java.lang.Thread 类，找到内部类 State
- NEW（新建）
- RUNNABLE（准备就绪）
- BLOCKED（阻塞）
- WAITING（等待-不见不散）
- TIMED_WAITING（等待-过时不候）
- TERMINATED（终结）

### wait和sleep

区别：
- sleep是Thread的静态方法；wait是Object的方法，任何对象实例都能调用。
- sleep不会释放锁，它也不需要占用锁；wait会释放锁，但调用它的前提是当前线程占有锁（即代码要在synchronized中）
- 它们都可以被interrupt方法中断

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

# CompletableFuture

## Future接口

Future是Java5新加的一个接口，定义了==操作异步任务执行的一些方法==

```java
package java.util.concurrent;
public interface Future<V> {
	boolean cancel(boolean mayInterruptIfRunning);  

    boolean isCancelled();  
  
    boolean isDone();  
  
    V get() throws InterruptedException, ExecutionException;  
  
    V get(long timeout, TimeUnit unit)  
        throws InterruptedException, ExecutionException, TimeoutException;  
}
```

- 如获取异步任务的执行结果、取消任务的执行、判断任务是否被取消、判断任务执行是否完毕等。（异步：可以被叫停，可以被取消）
- 一句话：Future接口可以为主线程开一个分支任务，专门为主线程处理耗时和费力的复杂业务。

> 比如主线程让一个子线程去执行任务，子线程可能比较耗时，启动子线程开始执行任务后，主线程就去做其他事情了，过了一会才去获取子任务的执行结果。
> 
> 例如：老师在上课，但是口渴，于是让班长这个线程去买水，自己可以继续上课，实现了异步任务。

异步多线程任务执行且有返回结果，三个特点：
1. 多线程
2. 有返回
3. 异步任务

## FutureTask
Future接口的实现类

继承关系：
![](assets/Pasted%20image%2020231120205347.png)

在源码可以看到
- FutureTask实现了`RunnableFuture`接口
- FutureTask不支持空参构造，仅支持构造传入Runnable和Callable（有返回值、可抛出异常）
```java

package java.util.concurrent;

public class FutureTask<V> implements RunnableFuture<V> {
	public FutureTask(Runnable runnable, V result) {  
	    this.callable = Executors.callable(runnable, result);  
	    this.state = NEW;       // ensure visibility of callable  
	}
	
	public FutureTask(Callable<V> callable) {  
	    if (callable == null)  
	        throw new NullPointerException();  
	    this.callable = callable;  
	    this.state = NEW;       // ensure visibility of callable  
	}
}
```

FutureTask案例：

多线程（一个主线程，一个mythread），有返回（返回了"hello callable"），异步
```java
public class CompetableFutureDemo {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        FutureTask<String> futureTask = new FutureTask<>(new MyThread());  
        Thread t1 = new Thread(futureTask, "t1");  
        t1.start();  
        System.out.println(futureTask.get());   
        //-----come in call  
        //hello Callable  
    }  
}

class MyThread implements Callable {  
    @Override  
    public Object call() throws Exception {  
        System.out.println("-----come in call");  
        return "hello Callable";  
    }  
}  
```

## Future优缺点

优点：==future+线程池 异步多线程任务配合，能显著提高程序的执行效率==

优点演示：
- 方案一，3个任务1个main线程处理，大概1101ms
- 方案二，3个任务开启多个异步任务线程处理，利用线程池（假如每次new一个Thread，太浪费资源，会有GC这些工作），大概848ms
```java
public class FutureThreadPoolDemo {  
    public static void main(String[] args) throws InterruptedException {  
        // m1();  
        // ---cost time: 1134        // main---end  
        m2();  
        // ---cost time: 336  
        // main---end    }  
  
    /**  
     * 三个任务，只有一个main线程处理  
     */  
    private static void m1() throws InterruptedException {  
        long startTime = System.currentTimeMillis();  
  
        TimeUnit.MILLISECONDS.sleep(500);  
        TimeUnit.MILLISECONDS.sleep(300);  
        TimeUnit.MILLISECONDS.sleep(300);  
  
        long endTime = System.currentTimeMillis();  
        System.out.println("---cost time: " + (endTime - startTime));  
        System.out.println(Thread.currentThread().getName() + "---end");  
    }  
  
    /**  
     * 三个任务，main+两个异步线程处理  
     */  
    private static void m2() throws InterruptedException {  
        ExecutorService threadPool = Executors.newFixedThreadPool(3);  
  
        long startTime = System.currentTimeMillis();  
  
        // 异步1  
        FutureTask<String> futureTask1 = new FutureTask<>(() -> {  
            TimeUnit.MILLISECONDS.sleep(500);  
            return "task1 over";  
        });  
        // 异步2  
        FutureTask<String> futureTask2 = new FutureTask<>(() -> {  
            TimeUnit.MILLISECONDS.sleep(300);  
            return "task1 over";  
        });  
        threadPool.submit(futureTask1);  
        threadPool.submit(futureTask2);  
        // 3  
        TimeUnit.MILLISECONDS.sleep(300);  
  
        long endTime = System.currentTimeMillis();  
        System.out.println("---cost time: " + (endTime - startTime));  
        System.out.println(Thread.currentThread().getName() + "---end");  
    }  
}
```

缺点：
- `get()`阻塞
	- 一旦调用get()方法，不管是否计算完成，都会导致阻塞（所以一般get方法放到最后）
	- 假如我不愿意等待很长时间，我希望过时不候，可以自动离开。
- `isDone()`轮询
	- 轮询的方式会耗费无谓的CPU资源，而且也不见得能及时地得到计算结果
	- 如果想要异步获取结果，通常都会以轮询的方式去获取结果尽量不要阻塞

==Future对于结果的获取不是很友好，只能通过阻塞或轮询的方式得到任务的结果==

缺点演示：
```java
public class FutureAPIDemo {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        // m1();  
        m2();  
    }  
  
    /**  
     * get()阻塞  
     */  
    public static void m1() throws ExecutionException, InterruptedException {  
        FutureTask<String> futureTask = new FutureTask<>(() -> {  
            System.out.println(Thread.currentThread().getName() + "---come in");  
            TimeUnit.SECONDS.sleep(5);  
            return "task over";  
        });  
        new Thread(futureTask, "t1").start();  
  
        System.out.println(futureTask.get()); // 不见不散，非要等到结果才会离开，容易程序阻塞  
  
        System.out.println(Thread.currentThread().getName() + "---忙其他任务了");  
    }  
  
    /**  
     * isDone()轮训  
     */  
    public static void m2() throws ExecutionException, InterruptedException {  
        FutureTask<String> futureTask = new FutureTask<>(() -> {  
            System.out.println(Thread.currentThread().getName() + "---come in");  
            TimeUnit.SECONDS.sleep(5);  
            return "task over";  
        });  
        new Thread(futureTask, "t1").start();  
  
        while (true){  
            if (futureTask.isDone()){  
                System.out.println(futureTask.get());  
                break;            }else {  
                TimeUnit.MILLISECONDS.sleep(500);  
                System.out.println("正在处理中，不要再催了");  
            }  
        }  
  
        System.out.println(Thread.currentThread().getName() + "---忙其他任务了");  
    }  
}
```

完成一些复杂的任务：
- 回调通知
	- 应对Future的完成时间，完成了可以告诉我
	- 通过轮询的方式去判断任务是否完成这样非常占CPU并且代码也不优雅
- 创建异步任务：Future+线程池组合
- 多个任务前后依赖可以组合处理
	- 想将多个异步任务的计算结果组合起来，后一个异步任务的计算结果需要前一个异步任务的值
	- 将两个或多个异步计算合成一个异步计算，这几个异步计算互相独立，同时后面这个又依赖前一个处理的结果
- 对计算速度选最快
	- 当Future集合中某个任务最快结束时，返回结果，返回第一名处理结果。
	结论：
- 使用Future之前提供的那点API就囊中羞涩，处理起来不够优雅，这时候还是让CompletableFuture以声明式的方式优雅的处理这些需求
- Future能干的，CompletableFuture都能干

## CompletableFuture

对于简单的业务场景使用Future完全OK

**为什么会出现？**
- get() 阻寒的方式和异步编程的设计理念相违背
- isDone() 轮询的方式会耗费无谓的CPU资源。
- 对于真正的异步处理我们希望是可以通过传入回调函数，在Future结束时自动调用该回调函数，这样，我们就不用等待结果。

因此，JDK8设计出`CompletableFuture`。`CompletableFuture`提供了一种观察者模式类似的机制，可以让任务执行完成后通知监听的一方。

**类架构说明**
- 接口`CompletionStage`
	- 代表异步计算过程中的某一个阶段，一个阶段完成以后可能会触发另外一个阶段。
	- 一个阶段的执行可能是被单个阶段的完成触发，也可能是由多个阶段一起触发
- 类`CompletableFuture`
	- 提供了非常强大的Future的扩展功能，可以帮助我们简化异步编程的复杂性，并且提供了函数式编程的能力，可以通过回调的方式处理计算结果，也提供了转换和组合CompletableFuture的方法
	- 它可能代表一个明确完成的Future，也可能代表一个完成阶段（`CompletionStage`），它支持在计算完成以后触发一些函数或执行某些动作

![](assets/Pasted%20image%2020231122133337.png)

### 核心的四个静态方法

runAsync无返回值
- `public static CompletableFuture<Void> runAsync(Runnable runnable)`
- `public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)`
supplyAsync有返回值
- `public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)`
- `public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)`

Executor参数解释
- 没有指定`Executor`的方法，直接使用默认的`ForkJoinPool.commonPool()`作为它的线程池执行异步代码。
- 如果指定线程池，则使用我们自定义的或者特别指定的线程池执行异步代码

runAsync无返回值演示：
```java
public class CompetableFutureBuildDemo {  
    public static void main(String[] args) throws ExecutionException, InterruptedException {  
        m1();  
        // ForkJoinPool.commonPool-worker-25  
        // null  
        // m2();        // pool-1-thread-1        // null    }  
  
    /**  
     * 不带线程池参数  
     * @throws ExecutionException  
     * @throws InterruptedException  
     */    private static void m1() throws ExecutionException, InterruptedException {  
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {  
            System.out.println(Thread.currentThread().getName());  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
        });  
  
        System.out.println(completableFuture.get());  
    }  
  
    /**  
     * 带线程池参数  
     * @throws ExecutionException  
     * @throws InterruptedException  
     */    private static void m2() throws ExecutionException, InterruptedException {  
        ExecutorService threadPool = Executors.newFixedThreadPool(3);  
  
        CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {  
            System.out.println(Thread.currentThread().getName());  
            try {  
                TimeUnit.SECONDS.sleep(1);  
            } catch (InterruptedException e) {  
                throw new RuntimeException(e);  
            }  
        }, threadPool);  
  
        System.out.println(completableFuture.get());  
    }  
}
```

### 通用异步编程
CompletableFuture减少阻塞和轮询，可以传入回调对象，当异步任务完成或者发生异常时，自动调用回调对象的回调方法。
```java
public class CompletableFutureUseDemo {
    public static void main(String[] args) throws InterruptedException {
        /**
         * 无异常结果：
         *      main线程先去忙其他任务
         *      ---1s后出结果：2
         *      ---计算完成，更新系统：2
         *
         * 有异常情况：
         *      ForkJoinPool.commonPool-worker-25---come in
         *      main线程先去忙其他任务
         *      ---1s后出结果：5
         *      异常情况：java.lang.ArithmeticException: / by zero	java.lang.ArithmeticException: / by zero
         */
        m1();
    }

    private static void m1() throws InterruptedException {
        CompletableFuture.supplyAsync(() -> {
            System.out.println(Thread.currentThread().getName() + "---come in");
            int result = ThreadLocalRandom.current().nextInt(10);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            System.out.println("---1s后出结果：" + result);
            if (result > 4) {
                int i = 10 / 0;
            }
            return result;
        }).whenComplete((v, e) -> {
            if (e == null) {
                System.out.println("---计算完成，更新系统：" + v);
            }
        }).exceptionally(e -> {
            // e.printStackTrace();
            System.out.println("异常情况：" + e.getCause() + "\t" + e.getMessage());
            return null;
        });
        System.out.println(Thread.currentThread().getName() + "线程先去忙其他任务");

        // 主线程不要立刻结束，否则completableFuture默认使用的线程池会立即关闭，执行不到whenComplete()
        TimeUnit.SECONDS.sleep(3);
    }
```

优点：
- 异步任务**结束**时，会==自动回调==某个对象的方法
- 主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行
- 异步任务**出错**时，会==自动回调==某个对象的方法。

### 常用方法

1）获得结果和触发计算
- 获取结果
	- `public T get()` 不见不散
	- `public T get(long timeout,TimeUnit unit)`：过时不候，超时抛出`TimeoutException`
	- `public T join()`：和get一样的作用，只是==不需要抛出异常==
	- `public T getNow(T valuelfAbsent)` 计算完成就返回正常值，否则==返回备胎值==（传入的参数），立即获取结果不阻塞
- 主动触发计算
	- `public boolean complete(T value)` 是否==打断get方法==立即返回括号值

```java
public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
	CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			throw new RuntimeException(e);
		}
		return "abc";
	});

//        System.out.println(completableFuture.get(2L,TimeUnit.SECONDS));
//        System.out.println(completableFuture.join());
//        System.out.println(completableFuture.getNow("xxx"));
	/**
	 * true	complete
	 */
	System.out.println(completableFuture.complete("complete")+"\t"+completableFuture.join());
}
```

2）对计算结果进行处理
- `thenApply()` 计算结果存在==依赖==关系，这两个线程==串行化==
	- 由于存在==依赖==关系（当前步错，不走下一步），当前步骤有异常的话就叫停
- `handle()` 计算结果存在依赖关系，这两个线程串行化
	- 有异常也可以往下走一步
```java
public class CompletableFutureApiDemo2 {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
//        m1(threadPool);
        m2(threadPool);

        /**
         * 串行化，thenApply，无异常：
         *      main，main---主线程去忙别的了
         *      pool-1-thread-1 步骤1
         *      pool-1-thread-1 步骤2
         *      pool-1-thread-1 步骤3
         *      ---计算结果：6
         * 有异常直接停
         *      main，main---主线程去忙别的了
         *      pool-1-thread-1 步骤1
         *      java.lang.ArithmeticException: / by zero
         */
        /**
         * 串行化，thenApply，有异常也往下走：
         *      main，main---主线程去忙别的了
         *      pool-1-thread-1 步骤1
         *      pool-1-thread-1 步骤3
         */
        System.out.println(Thread.currentThread().getName() + "，main---主线程去忙别的了");
        threadPool.shutdown();
    }

    /**
     * thenApply测试
     */
    private static void m1(ExecutorService threadPool) {
        CompletableFuture.supplyAsync(() -> {
            timeSleep();
            System.out.println(Thread.currentThread().getName() + " 步骤1");
            return 1;
        }, threadPool).thenApply(f -> {
            // 制造异常
            int i = 10 / 0;
            timeSleep();
            System.out.println(Thread.currentThread().getName() + " 步骤2");
            return f + 2;
        }).thenApply(f -> {
            timeSleep();
            System.out.println(Thread.currentThread().getName() + " 步骤3");
            return f + 3;
        }).whenComplete((v, e) -> {
            if (e == null) {
                System.out.println("---计算结果：" + v);
            }
        }).exceptionally(e -> {
            System.out.println(e.getMessage());
            ;
            return null;
        });
    }

    /**
     * handle()测试
     */
    private static void m2(ExecutorService threadPool) {
        CompletableFuture.supplyAsync(() -> {
            timeSleep();
            System.out.println(Thread.currentThread().getName() + " 步骤1");
            return 1;
        }, threadPool).handle((f, e) -> {
            // 制造异常
            int i = 10 / 0;
            timeSleep();
            System.out.println(Thread.currentThread().getName() + " 步骤2");
            return f + 2;
        }).handle((f, e) -> {
            timeSleep();
            System.out.println(Thread.currentThread().getName() + " 步骤3");
            return f + 3;
        }).whenComplete((v, e) -> {
            if (e == null) {
                System.out.println("---计算结果：" + v);
            }
        }).exceptionally(e -> {
//            System.out.println(e.getMessage());
            return null;
        });
    }

    private static void timeSleep() {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

3）对计算结果进行消费
- `thenAccept()`
	- 接受任务的处理结果，并==消费处理==，==无返回结果==
	- 对比补充
		- `thenRun(Runnable runnable)`：任务A执行完执行B，并且不需要A的结果
		- `thenAccept(Consumer action)`：任务A执行完执行B，==B需要A的结果==，但是任务B==没有返回值==
		- `thenApply(Function fn)`：任务A执行完执行B，==B需要A的结果==，同时任务==B有返回值==
```java
public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
	System.out.println(CompletableFuture.supplyAsync(() -> "resultA").thenRun((() -> {
		System.out.println("thenRun");
	})).join()); // 无返回结果 null

	System.out.println(CompletableFuture.supplyAsync(() -> "resultA").thenAccept(s -> {
		System.out.println(s);
	}).join());  // 无返回结果 null

	System.out.println(CompletableFuture.supplyAsync(() -> "resultA").thenApply(s -> {
		return s + " resultB";
	}).join());  // 有返回结果 resultA resultB
}
```

4）对计算速度进行选用

`applyToEither()`：谁快用谁

```java
public <U> CompletableFuture<U> applyToEither(  
    CompletionStage<? extends T> other, Function<? super T, U> fn)
```

```java
public class CompletableFutureFastDemo {
    public static void main(String[] args) throws InterruptedException {
        CompletableFuture<String> playA = CompletableFuture.supplyAsync(() -> {
            System.out.println("A come in");
            timeSleep(1, TimeUnit.SECONDS);
            return "playA";
        });

        CompletableFuture<String> playB = CompletableFuture.supplyAsync(() -> {
            System.out.println("B come in");
            timeSleep(2, TimeUnit.SECONDS);
            return "playB";
        });

        CompletableFuture<String> result = playA.applyToEither(playB, f -> f + " is winner");
        System.out.println(Thread.currentThread().getName() + "\t" + result.join());
    }

    private static void timeSleep(long num, TimeUnit timeUnit) {
        try {
            timeUnit.sleep(num);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

5）对计算结果进行合并

`thenCombine`
- 两个CompletableStage任务都完成后，最终能把两个任务的结果一起交给thenCombine来处理
- 先完成的先等着，等待其他分支任务

```java
public class CompletableFutureCombineDemo {
    public static void main(String[] args) throws InterruptedException {
        CompletableFuture<Integer> playA = CompletableFuture.supplyAsync(() -> {
            System.out.println("A come in");
            timeSleep(1, TimeUnit.SECONDS);
            return 10;
        });

        CompletableFuture<Integer> playB = CompletableFuture.supplyAsync(() -> {
            System.out.println("B come in");
            timeSleep(2, TimeUnit.SECONDS);
            return 20;
        });

        CompletableFuture<Integer> result = playA.thenCombine(playB, (x, y) -> {
            System.out.println("两个结果开始合并");
            return x + y;
        });
        System.out.println(result.join());
    }

    private static void timeSleep(long num, TimeUnit timeUnit) {
        try {
            timeUnit.sleep(num);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

### 线程池运行选择

` CompletableFuture`和线程池说明
- 如果没有传入自定义线程池，都用默认线程池`ForkJoinPool`
- 传入一个线程池，如果你执行第一个任务时，传入了一个自定义线程池
	- 调用`thenRun`方法执行第二个任务时，则第二个任务和第一个任务时共用同一个线程池
	- 调用`thenRunAsync`执行第二个任务时，则第一个任务使用的是你自定义的线程池，第二个任务使用的是`ForkJoin`线程池
- 可能是线程处理太快，系统优化切换原则， 直接使用main线程处理（指定了线程池不一定用，很少会出现）
- 备注：`thenAccept`和`thenAcceptAsync`，`thenApply`和`thenApplyAsync`等，之间的区别同理。
```java
public class CompletableFutureWithThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(5);
        m1(threadPool);
        /**
         * 1	pool-1-thread-1
         * 2	main
         * 3	main
         */
//        m2(threadPool);
    }

    /**
     * 执行第一个任务时，传入了一个自定义线程池
     */
    private static void m1(ExecutorService threadPool) {
        try {
            CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
//                        timeSleep();
                        System.out.println("任务1\t" + Thread.currentThread().getName());
                        return "abcd";
                    }, threadPool)
                    /**
                     * 第二个任务和第一个任务时共用同一个线程池
                     */
//                    .thenRun(() -> {
//                        timeSleep();
//                        System.out.println("2\t" + Thread.currentThread().getName());
//                    })
                    /**
                     * 第一个任务使用的是你自定义的线程池，第二个任务使用的是`ForkJoin`线程池
                     */
                    .thenRunAsync(() -> {
                        timeSleep();
                        System.out.println("任务2\t" + Thread.currentThread().getName());
                    })
                    .thenRun(() -> {
                        timeSleep();
                        System.out.println("任务3\t" + Thread.currentThread().getName());
                    });
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }

    /**
     * 线程处理太快，系统优化切换原则， 直接使用main线程处理
     */
    private static void m2(ExecutorService threadPool) {
        try {
            CompletableFuture<Void> completableFuture = CompletableFuture.supplyAsync(() -> {
                        System.out.println("1\t" + Thread.currentThread().getName());
                        return "abcd";
                    }, threadPool)
                    .thenRun(() -> {
                        timeSleep();
                        System.out.println("2\t" + Thread.currentThread().getName());
                    })
                    .thenRun(() -> {
                        timeSleep();
                        System.out.println("3\t" + Thread.currentThread().getName());
                    });
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }

    private static void timeSleep() {
        try {
            TimeUnit.MILLISECONDS.sleep(10);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```


## 案例精讲-从电商网站的比价需求展开

**需求分析：**
1. 需求说明：
	1. 同一款产品，同时搜索出同款产品在各大电商平台的售价
	2. 同一款产品，同时搜索出本产品在同一个电商平台下，各个入驻卖家售价是多少
2. 输出返回：
	1. 出来结果希望是同款产品的在不同地方的价格清单列表，返回一个`List<String>`
3. 解决方案，对比同一个产品在各个平台上的价格，要求获得一个清单列表
	1. step by step，按部就班，查完淘宝查京东，查完京东查天猫....
	2. all in，万箭齐发，一口气多线程异步任务同时查询

```java
public class CompletableFutureMallDemo {  
    static List<NetMall> list = Arrays.asList(  
            new NetMall("jd"),  
            new NetMall("taobao"),  
            new NetMall("dangdang"));  
  
    /**  
     * step by step     */    public static List<String> getPrice(List<NetMall> list, String productName) {  
        //《Mysql》 in jd price is 88.05        return list.stream()  
                .map(netMall -> String.format(  
                        "《" + productName + "》" + "in %s price is %.2f",  
                        netMall.getNetMallName(), netMall.calcPrice(productName)))  
                .collect(Collectors.toList());  
    }  
  
    /**  
     * all in     * 把list里面的内容映射给CompletableFuture()  
     * List<NetMall> ----->List<CompletableFuture<String>>------> List<string>  
     */  
    public static List<String> getPriceByCompletableFuture(List<NetMall> list, String productName) {  
        return list.stream()  
                .map(netMall ->  
                        CompletableFuture.supplyAsync(() ->  
                                String.format(  
                                        "《" + productName + "》" + "in %s price is %.2f",  
                                        netMall.getNetMallName(),  
                                        netMall.calcPrice(productName)))) // Stream<CompletableFuture<String>>  
                .collect(Collectors.toList()) //List<CompletableFuture<String>>  
                .stream() // Stream<String>  
                .map(s -> s.join())  
                .collect(Collectors.toList()); // List<String>  
    }  
  
    public static void main(String[] args) {  
        /**  
         * 采用step by step方式查询  
         * 《mysql》in jd price is 110.11  
         * 《mysql》in taobao price is 109.32  
         * 《mysql》in dangdang price is 109.24  
         * ------costTime: 3094 毫秒  
         */  
        long StartTime = System.currentTimeMillis();  
        List<String> list1 = getPrice(list, "mysql");  
        for (String element : list1) {  
            System.out.println(element);  
        }  
        long endTime = System.currentTimeMillis();  
        System.out.println("------costTime: " + (endTime - StartTime) + " 毫秒");  
  
        /**  
         * 采用 all in三个异步线程方式查询  
         * 《mysql》in jd price is 109.71  
         * 《mysql》in taobao price is 110.69  
         * 《mysql》in dangdang price is 109.28  
         * ------costTime: 1009 毫秒  
         */  
        long StartTime2 = System.currentTimeMillis();  
        List<String> list2 = getPriceByCompletableFuture(list, "mysql");  
        for (String element : list2) {  
            System.out.println(element);  
        }  
        long endTime2 = System.currentTimeMillis();  
        System.out.println("------costTime: " + (endTime2 - StartTime2) + " 毫秒");  
  
    }  
}  
  
@AllArgsConstructor  
@NoArgsConstructor  
@Data  
class NetMall {  
    private String netMallName;  
  
    public double calcPrice(String productName) {  
        try {  
            TimeUnit.SECONDS.sleep(1);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
  
        return ThreadLocalRandom.current().nextDouble() * 2 + productName.charAt(0);  
    }  
}
```

# 说说Java"锁"事

## 从轻松的乐观锁和悲观锁开讲

- 悲观锁： 
	- 认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改，synchronized和Lock的实现类都是悲观锁
	- 特点：
		- 适合写操作多的场景，先加锁可以保证写操作时数据正确，显示的锁定之后再操作同步资源
		- 狼性锁  
- 乐观锁： 
	- 认为自己在使用数据的时候不会有别的线程修改数据或资源，不会添加锁
	- Java中使用无锁编程来实现，只是在更新的时候去判断，之前有没有别的线程更新了这个数据
		- 如果这个数据没有被更新，当前线程将自己修改的数据成功写入
		- 如果已经被其他线程更新，则根据不同的实现方式执行不同的操作，比如：放弃修改、重试抢锁等等。
	- 实现方式：
		- 版本号机制Version
		- 最常采用的是CAS(Compare-and-Swap，比较并替换)算法
			- Java原子类中的递增操作就通过CAS自旋实现的。
	- 特点
		- 适合读操作多的场景，不加锁的特性能够使其读操作的性能大幅提升
		- 乐观锁则直接去操作同步资源，是一种无锁算法，得之我幸不得我命，再努力就是
		- 佛系锁

## 通过8种情况演示锁运行案例，看看锁到底是什么

```java
现象描述：
 * 1 标准访问ab两个线程，请问先打印邮件还是短信？ --------先邮件，后短信  共用一个对象锁
 * 2. sendEmail钟加入暂停3秒钟，请问先打印邮件还是短信？---------先邮件，后短信  共用一个对象锁
 * 3. 添加一个普通的hello方法，请问先打印普通方法还是邮件？ --------先hello，再邮件
 * 4. 有两部手机，请问先打印邮件还是短信？ ----先短信后邮件  资源没有争抢，不是同一个对象锁
 * 5. 有两个静态同步方法，一步手机， 请问先打印邮件还是短信？---------先邮件后短信  共用一个类锁
 * 6. 有两个静态同步方法，两部手机， 请问先打印邮件还是短信？ ----------先邮件后短信 共用一个类锁
 * 7. 有一个静态同步方法 一个普通同步方法，请问先打印邮件还是短信？ ---------先短信后邮件   一个用类锁一个用对象锁
 * 8. 有一个静态同步方法，一个普通同步方法，两部手机，请问先打印邮件还是短信？ -------先短信后邮件 一个类锁一个对象锁
 */

public class Lock8Demo {
    public static void main(String[] args) {
        Phone phone = new Phone();
//        Phone phone2 = new Phone();
        new Thread(() -> {
            phone.sendEmail();
        }).start();

        // 保证线程的启动顺序
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        new Thread(() -> {
            phone.sendSMS();
        }).start();
    }
}

class Phone {
    public synchronized void sendEmail() {
//        try {
//            TimeUnit.SECONDS.sleep(2);
//        } catch (InterruptedException e) {
//            throw new RuntimeException(e);
//        }
        System.out.println("---sendEmail");
    }

    public synchronized void sendSMS() {
        System.out.println("---sendSMS");
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

## 字节码角度分析synchronized

> 使用Java编译器（`javac`命令），将Java源代码编译成字节码。字节码是一种中间代码，不是直接在特定硬件上运行的机器代码，而是在Java虚拟机（JVM）上执行的代码。
> `javac HelloWorld.java`
> 上述命令将生成`HelloWorld.class`文件，其中包含了编译后的字节码。
> `javap HelloWorld.class` 反编译

1）synchronized同步代码块

实现使用的是`monitorenter`和`monitorexit`指令
- 一般情况是1个enter对应2个exit
	- 一个正常退出、一个异常退出
	- 方法里面抛了异常了，那就只有一个exit
```
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

调用指令将会检查方法的`ACC SYNCHRONIZED`访问标志是否被设置
如果设置了，执行线程会将先持有monitor锁，然后再执行方法，最后在方法完成（无论是正常完成还是非正常完成）时释放 monitor

```
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

```
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

## 反编译synchronized锁的是什么？

> 面试题：为什么任何一个对象都可以成为一个锁？

> 管程（英语:Monitors，也称为监视器）是一种程序结构，结构内的多个子程序(对象或模块)形成的多个工作线程互斥访问共享资源。
> 
> 这些共享资源一般是硬件设备或一群变量。对共享变量能够进行的所有操作集中在一个模块中。(把信号量及其操作原语“封装”在一个对象内部）管程实现了在一个时间点，最多只有一个线程在执行管程的某个子程序。管程提供了一种机制，管程可以看做一个软件模块，它是将共享的变量和对于这些共享变量的操作封装起来，形成一个具有一定接口的功能模块，进程可以调用管程来实现进程级别的并发控制。

> Java虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步结构都是使用管程（Monitor，更常见的是直接将它称为“锁”）来实现的。
> 
> 方法级的同步是隐式的，无须通过字节码指令来控制，它实现在方法调用和返回操作之中。虚拟机可以从方法常量池中的方法表结构中的 `ACC SYNCHRONIZED` 访问标志得知一个方法是否被声明为同步方法。当方法调用时，调用指令将会检查方法的 `ACC SYNCHRONIZED` 访问标志是否被设置，如果设置了，执行线程就要求先成功持有管程，然后才能执行方法，最后当方法完成 (无论是正常完成还是非正常完成)时释放管程。在方法执行期间，执行线程持有了管程，其他任何线程都无法再获取

每个对象天生都带着一个对象监视器，每一个被锁住的对象都会和Monitor关联起来

在HotSpot虚拟机中，monitor采用`ObjectMonitor`实现
`ObjectMonitor.java` --> `ObjectMonitor.cpp` --> `objectMonitor.hpp`
其主要数据结构如下（位于HotSpot虚拟机源码`objectMonitor.hpp`文件，C++实现的）

![](assets/Pasted%20image%2020231127220305.png)

指针指向monitor对象（也称为管程或监视器）的起始地址。每个对象都在在着一个monitor与之关联，当一个 monitor 被某个线程持有后，它便处于锁定状态。

![](assets/Pasted%20image%2020231129223403.png)

## 公平锁和非公平锁

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

- 公平锁：是指多个线程按照==申请锁的顺序==来获取锁，这里类似于排队买票，先来的人先买，后来的人再队尾排着，这是公平的
	- `Lock lock = new ReentrantLock(true)`---表示公平锁，先来先得。
- 非公平锁：是指多个线程获取锁的顺序并不是按照申请的顺序，有可能后申请的线程比先申请的线程优先获取锁，在高并发环境下，有可能造成优先级反转或者饥饿的状态（某个线程一直得不到锁）
	- `Lock lock = new ReentrantLock(false)`---表示非公平锁，后来的也可能先获得锁，==默认为非公平锁==。

> 面试题：为什么会有公平锁和非公平锁的设计？为什么默认非公平

- 恢复挂起的线程到真正锁的获取还是有时间差的，从开发人员来看这个时间微乎其微，但是从CPU的角度来看，这个时间差存在的还是很明显的。
	- 所以非公平锁能更充分的利用CPU 的时间片，==尽量减少 CPU 空闲状态时间==。
- 使用多线程很重要的考量点是线程切换的开销，当采用非公平锁时，当1个线程请求锁获取同步状态，然后释放同步状态，所以==刚释放锁的线程在此刻再次获取同步状态的概率就变得非常大，所以就减少了线程的开销。==

## 可重入锁（递归锁）

是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提。锁对象得是同一个对象），不会因为之前已经获取过还没释放而阻塞。

如果是1个有 synchronized 修饰的递归调用方法，程序第2次进入被自己阻塞了岂不是天大的笑话，出现了作茧自缚。所以Java中ReentrantLock和synchronized都是可重入锁，可重入锁的一个优点是可一定程度避免死锁。

种类
- 隐式锁（synchronized关键字使用的锁）默认是可重入锁
	- 实现原理：
		- ==每个锁对象拥有一个锁计数器和一个指向持有该锁的线程的指针==。
		- 当执行`monitorenter`时，如果目标锁对象的**计数器为零**，那么说明它没有被其他线程所持有，Java虚拟机会将该锁对象的持有线程设置为当前线程，并且将其计数器加1。
		- 在目标锁对象的**计数器不为零**的情况下，如果锁对象的持有线程是当前线程，那么 Java 虚拟机可以将其计数器加1，否则需要等待直至持有线程释放该锁。
		- 当执行`monitorexit`时，Java虚拟机则需将锁对象的计数器减1。计数器为零代表锁已被释放
- 显示锁（Lock）也有ReentrantLock这样的可重入锁
	- 加锁几次就要解锁几次

```java
public class ReEntryLockDemo {
    public static void main(String[] args) {
        // reEntryM1();
        reEntryM2();
    }

    /**
     * 显示可重入锁
     */
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
                // lock.unlock();
            }
        }).start();

        // 加锁次数和释放锁次数不一致，第二个线程始终无法获取到锁，导致一直在等待。

        new Thread(() -> {
            lock.lock();
            try {
                System.out.println("---外层调用");
            } finally {
                lock.unlock();
            }
        }).start();
    }

    /**
     * 隐式可重入锁
     */
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


## 死锁及排查

死锁是指两个或两个以上的线程在执行过程中，因争夺资源而造成的一种==互相等待==的现象,若无外力干涉那它们都将无法推进下去，如果系统资源充足，进程的资源请求都能够得到满足，死锁出现的可能性就很低，否则就会因争夺有限的资源而陷入死锁。

> 一句话：吃着碗里的，看着锅里的

产生死锁主要原因
- 系统资源不足
- 进程运行推进的顺序不合适
- 资源分配不当

###  请写一个死锁代码案例

```java
public class ThreadLockDemo {
    public static void main(String[] args) {
        Object oA = new Object();
        Object oB = new Object();

        new Thread(() -> {
            synchronized (oA) {
                System.out.println(Thread.currentThread().getName() + "持有A锁，希望获得B锁");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                synchronized (oB) {
                    System.out.println(Thread.currentThread().getName() + "成功获得B锁");
                }
            }
        },"A").start();

        new Thread(() -> {
            synchronized (oB) {
                System.out.println(Thread.currentThread().getName() + "持有B锁，希望获得A锁");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                synchronized (oA) {
                    System.out.println(Thread.currentThread().getName() + "成功获得A锁");
                }
            }
        },"B").start();
    }
}
```

### 如何排查死锁

1）命令方式

`jps -l`查出进程号
```bash
> jps -l 
16624 org.jetbrains.jps.cmdline.Launcher
24608 jdk.jcmd/sun.tools.jps.Jps
2752 com.boer.lock.ThreadLockDemo
5408
14644 org.jetbrains.idea.maven.server.RemoteMavenServer36
```

`jstack 进程号` 打印栈信息
```bash
> jstack 2752
"B":
        at com.boer.lock.ThreadLockDemo.lambda$main$1(ThreadLockDemo.java:38)
        - waiting to lock <0x000000076cb8a1d8> (a java.lang.Object)
        - locked <0x000000076cb8a1e8> (a java.lang.Object)
        at com.boer.lock.ThreadLockDemo$$Lambda$2/1989780873.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"A":
        at com.boer.lock.ThreadLockDemo.lambda$main$0(ThreadLockDemo.java:24)
        - waiting to lock <0x000000076cb8a1e8> (a java.lang.Object)
        - locked <0x000000076cb8a1d8> (a java.lang.Object)
        at com.boer.lock.ThreadLockDemo$$Lambda$1/2074407503.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
```

2）图形化界面

jconsole

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

### LockSupport是什么？

class LockSupport

创建锁和其他同步类的基本**线程阻塞原语**

LockSupport中的
- park()：阻塞线程
- unpark()：解除阻塞线程

### 三种让线程等待和唤醒的方式

1. Object 中的`wait()`方法让线程等待，使用 Object 中的`notify()`方法唤醒线程
2. JUC 包中 Condition 的`await()`方法让线程等待，使用`signal()`方法唤醒线程
3. **LockSupport 类**

方式一：

```java
public class Demo {  
    public static void main(String[] args) throws InterruptedException {  
        Object lock = new Object();  
  
        new Thread(() -> {  
            synchronized (lock) {  
                try {  
                    lock.wait();  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
            System.out.println(Thread.currentThread().getName() + "\t" + "被唤醒");  
        }, "t1").start();  
  
        Thread.sleep(3000);  
  
        new Thread(() -> {  
            synchronized (lock) {  
                lock.notify();  
            }  
        }, "t2").start();  
    }  
}
```

