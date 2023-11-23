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

<br>

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
}

```

优点：
- 异步任务**结束**时，会==自动回调==某个对象的方法;
- 主线程设置好回调后，不再关心异步任务的执行，异步任务之间可以顺序执行
- 异步任务**出错**时，会==自动回调==某个对象的方法;

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
- `thenApply()` 计算结果存在依赖关系，这两个线程串行化
	- 由于存在依赖关系（当前步错，不走下一步），当前步骤有异常的话就叫停
- `handle()` 计算结果存在依赖关系，这两个线程串行化
	- 有异常也可以往下走一步

3）对计算结果进行消费

4）对计算速度进行选用

5）对计算结果进行合并

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


# Lock接口

### synchronized关键字回顾

synchronized是Java 中的关键字，是一种同步锁。它修饰的对象有以下几种:

1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是 { } 括起来的代码，作用的对象是调用这个代码块的对象;
    
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象
    
3. 修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是==这个类的所有对象==
    
4. 修饰一个类，其作用的范围是synchronized后面括号括起来的部分，作用的对象是==这个类的所有对象==。
    

> 虽然可以使用synchronized来定义方法，但synchronized并不属于方法定义的一部分，因此，synchronized 关键字不能被继承。如果在父类中的某个方法使用了synchronized关键字，而在子类中覆盖了这个方法，在子类中的这个方法默认情况下并不是同步的，而必须显式地在子类的这个方法中加上synchronized 关键字才可以。当然，还可以在子类方法中调用父类中相应的方法，这样虽然子类中的方法不是同步的，但子类调用了父类的同步方法，因此，子类的方法也就相当于同步了。

使用synchronized实现卖票例子

> 多线程的编程步骤：
> 
> 第一：创建一个资源类，在资源类创建属性和操作方法
> 
> 第二：创建多线程，调用资源类里面的操作方法

 // 资源类  
 class Ticket {  
     private int rest = 30;  
 ​  
     public synchronized void saleTicket() {  
         if (rest > 0)  
             System.out.println(Thread.currentThread().getName() + "卖出一张篇，还剩" + --rest + "张");  
     }  
 }  
 ​  
 public class SaleTicketTest {  
     public static void main(String[] args) {  
         Ticket ticket = new Ticket();  
         Runnable r = new Runnable() {  
             @Override  
             public void run() {  
                 for (int i = 0; i < 1000; i++) {  
                     ticket.saleTicket();  
                 }  
             }  
         };  
         // 创建多个线程  
         new Thread(r, "A").start();  
         new Thread(r, "B").start();  
         new Thread(r, "C").start();  
     }  
 }

### 什么是Lock接口？

Lock锁实现提供了比使用同步方法和语句可以获得的更广泛的锁操作。它们允许更灵活的结构，可能具有非常不同的属性，并且可能支持多个关联的条件对象。Lock提供了比 synchronized更多的功能。

Lock 与的Synchronized区别

- Lock==不是Java语言内置的==，synchronized是Java语言的关键字，因此是内置特性。Lock是一个类，通过这个类可以实现同步访问;.
    
- Lock和synchronized有一点非常大的不同，采用==synchronized不需要用户去手动释放锁==，当synchronized方法或者 synchronized代码块执行完之后，系统会自动让线程释放对锁的占用; 而==Lock则必须要用户去手动释放锁==，如果没有主动释放锁，就有可能导致出现死锁现象。,