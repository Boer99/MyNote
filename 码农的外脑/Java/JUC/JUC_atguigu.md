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
	- 认为自己在使用数据的时候==一定有别的线程来修改数据==，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改，**synchronized**和**Lock**的实现类都是悲观锁
	- 特点：
		- 适合==写操作多==的场景，先加锁可以保证写操作时数据正确，显示的锁定之后再操作同步资源
		- 狼性锁  
- 乐观锁： 
	- 认为自己在使用数据的时候==不会有别的线程修改数据或资源==，不会添加锁
	- Java中使用无锁编程来实现，只是在更新的时候去判断，之前有没有别的线程更新了这个数据
		- 如果这个数据没有被更新，当前线程将自己修改的数据成功写入
		- 如果已经被其他线程更新，则根据不同的实现方式执行不同的操作，比如：放弃修改、重试抢锁等等。
	- 实现方式：
		- 版本号机制 **Version**
		- 最常采用的是 **CAS** (Compare-and-Swap，比较并替换)算法
			- Java原子类中的递增操作就通过CAS自旋实现的。
	- 特点
		- 适合==读操作多==的场景，不加锁的特性能够使其读操作的性能大幅提升
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

### 三种让线程等待和唤醒的方式

1）Object 中的`wait()`方法让线程等待，使用 Object 中的`notify()`方法唤醒线程

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

注意：
- `wait()`和`notify()`必须要在同步块或者方法里，且成对出现
	- 否则会抛出`IllegalMonitorStateException`异常
- 先`wait()`后`notify()`              

--- 
2）JUC 包中 Condition 的`await()`让线程等待，`signal()`唤醒线程

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

# Java内存模型之JMM

> 从大厂面试开始

- 你知道什么是Java内存模型JMM吗？
- JMM和volatile他们两个之间的关系？
- JMM有哪些特征或者它的三大特征是什么？
- 为什么要有JMM，它为什么出现？作用和功能是什么？
- happens-before先行并发原则你有了解过吗？

> 计算机硬件存储体系

![|700](assets/Pasted%20image%2020240106181702.png)

CPU的运行并不是直接操作内存，而是先把内存里边的数据读到缓存。内存的读和写操作的时候就会造成不一致的问题。
![|400](assets/Pasted%20image%2020240106181855.png)

> JMM是什么？

JVM规范中试图定义一种Java内存模型（Java Memory Model，简称JMM）

本身是一种抽象的概念并不真实存在它仅仅述的是一组约定或规范，通过这组规范定义了程序中(尤其是多线程)各个变量的读写访问方式，并决定一个线程对共享变量的写入何时以及如何变成对另一个线程可见。

原则：JMM的关键技术点都是围绕多线程的原子性、可见性和有序性展开的

> 为什么需要JMM？/ JMM能干嘛？

- 实现线程和主内存之间的抽象关系。
- 来屏蔽掉各种硬件和操作系统的内存访问差异，实现让Java在各种平台下都能达到一致的内存访问效果。

## JMM规范下，三大特性

可见性、原子性和有序性

> 可见性

定义：当一个线程修改了某一个共享变量的值，其他线程是否能够**立即**知道该变更。（线程修改完后写回主内存，然后通知其他线程）

JMM规定了所有的变量都存储在主内存中。

系统主内存**共享变量**数据修改被写入的时机是不确定的，多线程并发下很可能出现"脏读"，

所以==每个线程都有自己的工作内存==，保存了该线程用到的变量的**主内存副本拷贝**，线程对变量的所有操作（读取、赋值等 ）都==必需在线程自己的工作内存中进行==，而不能够直接读写主内存中的变量。

不同线程之间也无法直接访问对方工作内存中的变量，==线程间变量值的传递均需要通过主内存来完成==。

![|700](assets/Pasted%20image%2020240107140209.png)

线程脏读举例
- 主内存有变量x，初始值为0
- 线程 A 要将x 加 1，先将 x=0 拷贝到自己的私有内存中，然后更新 x 的值
- 线程 A 将更新后的 x 值回刷到主内存的时间是不固定的
- 刚好在线程 A 没有回刷 x 到主内存时（线程执行时间到，CPU调度其他线程了），线程 B 同样从主内存中读取 ，此时为 0，和线程 A 一样的操作，==最后期盼的 x=2 就会变成 x=1==

> 原子性

定义：指一个操作是不可打断的，即多线程环境下，操作不能被其他线程干扰

加锁

> 有序性（指令重排、禁用）

定义：
- 对于一个线程的执行代码而言，我们总是习惯性认为代码的执行总是从上到下，有序执行。但为了==提升性能==，编译器和处理器通常会对指令序列进重新排序。
- Java规范规定JVM线程内部维持顺序化语义，即只要程序的==最终结果与它顺序化执行的结果相等==，那么==指令的执行顺序可以与代码顺序不一致==，此过程叫**指令的重排序**

优缺点：
- JVM能根据处理器特性（CPU多级缓存系统、多核处理器等）适当的对机器指令进行重排序，使机器指令能更符合CPU的执行特性，最大限度的发挥机器性能。
- 但是，指令重排可以==保证串行语义==一致，但==不保证多线程间的语义==也一致（即可能产生"脏读"）
	- 简单说，两行以上不相干的代码在执行的时候有可能先执行的不是第一条，不见得是从上到下顺序执行，执行顺序会被优化。

源码到最终执行：源代码-->编译器优化的重排-->指令并行的重排-->内存系统的重排-->最终执行的指令

在某些情况下要**禁止**指令重排序
- 单线程环境里能够确保程序最终执行结果和代码顺序执行的结果一致。处理器在进行重排序时==必须要考虑==指令之间的**数据依赖性**。
- 多线程环境中线程交替执行，由于**编译器优化重排**的存在，两个线程中使用的变量能否保证一致性是==无法确定的==，结果无法预测。

## JMM规范下，多线程对变量的读写过程

我们定义的所有共享变量都储存在**物理主内存**中

每个线程都有自己独立的工作内存，里面保存该线程使用到的变量的副本（主内存中该变量的一份拷贝）。线程对共享变量所有的操作都必须先在线程自己的工作内存中进行后写回主内存，不能直接从主内存中读写（不能越级）

不同线程之间也无法直接访问其他线程的工作内存中的变量，==线程间变量值的传递需要通过主内存来进行==（同级不能相互访问）

## JMM规范下，多线程先行发生原则之happens-before

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

# volatile与JMM

## 被volatile修饰的变量有两大特点

1. 可见性
2. 有序性：有排序要求，有时需要==禁重排==

内存语义：
- 当**写**一个volatile变量时，JMM会把该线程对应的本地内存中的==共享变量值立即刷新回主内存中==
- 当**读**一个volatile变量时，JMM会把该线程对应的==本地内存设置为无效==，重新==回到主内存中读取==最新共享变量的值
- 所以volatile的写内存语义是直接刷新到主内存中，读的内存语义是直接从主内存中读取

volatile凭什么可以保证可见性和有序性？**内存屏障Memory Barrier**

## 内存屏障（面试重点）

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

## volatile的特性

### 保证可见性

保证不同线程对某个变量完成操作后结果及时可见，即该共享变量一旦改变所有线程立即可见

> 代码案例

```java
public class VolatileSeeDemo {
    /**
     * t1	-------come in
     * main	 修改完成
     */
//    static boolean flag = true;

    /**
     * t1	-------come in
     * main	 修改完成
     * t1	-------flag被设置为false，程序停止
     */
    static volatile boolean flag = true;

    public static void main(String[] args) {
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + "\t-------come in");
            while (flag) {

            }
            System.out.println(Thread.currentThread().getName() + "\t-------flag被设置为false，程序停止");
        }, "t1").start();

        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        // 更新flag值
        flag = false;
        System.out.println(Thread.currentThread().getName() + "\t 修改完成");
    }
}
```

- 不加volatile，没有可见性，程序==无法停止==
- 加了volatile，保证可见性，程序可以停止

> 线程 t1 中为何看不到被主线程 main修改为 false 的 flag 的值？

问题可能:
1. 主线程修改了flag之后没有将其刷新到主内存，所以t1线程看不到。
2. 主线程将flag刷新到了主内存，但是t1一直读取的是自己工作内存中flag的值，没有去主内存中更新获取fag最新的值。

使用volatile修饰共享变量，就可以达到上面的效果，被volatile修改的变量有以下特点:
1. 线程中读取的时候，每次读取都会去主内存中读取共享变量最新的值，然后将其复制到工作内存
2. 线程中修改了工作内存中变量的副本，修改之后会立即刷新到主内存

> volatile 变量读写过程（了解）

Java内存模型中定义的8种每个线程自己的工作内存与主物理内存之间的原子操作。
- read: 作主内存，将store传用于主内存，将变量的值从主内存传输到工作内存，主内存到工作内存
- load: 作用于工作内存，将read从主内存传输的变量值放入工作内存变量副本中，即数据加载
- use: 作用于工作内存，将工作内存变量副本的值传递给执行引擎，每当JVM遇到需要该变量的字节码指令时会执行该操作
- assign: 作用于工作内存，将从执行引接收到的值赋值给工作内存变量，每当JVM遇到一个给变量赋值字节码指令时会执行该操作
- store: 作用于工作内存，将赋值完毕的工作变量的值写回给主内存
- write: 作用于输过来的变量值赋值给主内存中的变量

由于上述6条只能保证单条指令的原子性，针对多条指令的组合性原子保证，没有大面积加锁，所以，JVM提供了另外两个原子指令
- lock: 作用于主内存，将一个变量标记为一个线程独占的状态，只是写时候加锁，就只是锁了写变量的过程。
- unlock: 作用于主内存，把一个处于锁定状态的变量释放，然后才能被其他线程占用

![](assets/Pasted%20image%2020240108204011.png)

### 没有原子性

volatile变量的复合操作不具有原子性，例如number++

> 代码案例

```java
class MyNumber {  
    volatile int number;  
  
    public void addPlusPlus() {  
        number++;  
    }  
}  
  
public class VolatileSeeDemo2 {  
    public static void main(String[] args) throws InterruptedException {  
        MyNumber myNumber = new MyNumber();  
  
        for (int i = 0; i < 10; i++) {  
            new Thread(() -> {  
                for (int j = 0; j < 1000; j++) {  
                    myNumber.addPlusPlus();  
                }  
            }).start();  
        }  
  
        Thread.sleep(2000);  
  
        System.out.println(myNumber.number);  
    }  
}
```

res：不到10000

> 为什么没有保证原子性？

![](assets/Pasted%20image%2020240108212147.png)

对于volatile变量具备可见性，JVM==只是保证==从主内存加载到线程工作内存的值是最新的，也仅是数据加载时是最新的。

但是多线程环境下，"数据计算"和"数据赋值"操作可能多次出现。数据在加载之后，若主内存volatile修饰变量发生修改，线程工作内存中的操作将会作废，去读主内存最新值，操作出现写丢失问题。即各线程私有内存和主内存公共内存中变量不同步，进而导致数据不一致。

由此可见volatile解决的是变量读时的可见性问题，但无法保证原子性，对于==多线程修改主内存共享变量的场景必须使用加锁同步==。比如给`addPlusPlus()`加上 synchronized

> 结论

volatile变量不适合参与到依赖当前值的运算，如  `i=i+1` `i++` 之类的

通常volatile用做保存某个状态的 boolean值 或者 int值。

《深入理解Java虚拟机》提到：由于volatile变量只能保证可见性，在不符合以下两条规则的运算场景中，我们仍然要通过加锁（使用synchronized、java.util.concurrent中的锁或原子类）来保证原子性。
- 运算结果并==不依赖变量的当前值==，或者能够确保==只有单一的线程修改==变量的值
- 变量不需要与其他的状态变量==共同参与==不变约束。

### 指令禁重排

volatile 的底层实现是通过内存屏障 [内存屏障（面试重点）](#内存屏障（面试重点）)

## 如何正确使用 volatile ？

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
- 利用volatile保证读取操作的可见性
- 用synchronized保证复合操作的原子性

```java
private volatile int value =0;

public int getValue(){
    return value;
}

public synchronized int setValue(){
    return value++;
}
```

> DCL (doube-checked-locking 双重检查锁定) 双端锁的发布

