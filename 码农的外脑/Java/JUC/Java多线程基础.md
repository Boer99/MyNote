## 线程简介
### Process与Thread
**程序**：说起进程，就不得不说下程序。程序是指令和数据的有序集合，其本身没有任何运行的含义，是一个静态的概念。<br />**进程**：而进程则是执行程序的一次执行过程，它是一个动态的概念。是**系统资源分配的单位**<br />**线程**：通常在一个进程中可以包含若干个线程，当然一个进程中至少有一个线程，不然没有存在的意义。线程是**CPU调度和执行的单位**
> 注意：很多多线程是模拟出来的，真正的多线程是指有多个 cpu，即多核，如服务器。如果是模拟出来的多线程，即在一个 cpu 的情况下，在同一个时间点，cpu 只能执行一个代码，因为切换的很快，所以就有同时执行的错觉。

### 核心概念

- 线程就是独立的执行路径
- 在程序运行时，即使没有自己创建线程，后台也会有多个线程，如主线程，gc 线程
- `**main()**`**称之为主线程，为系统的入口，用于执行整个程序**
- 在一个进程中，如果开辟了多个线程，线程的运行由调度器安排调度，调度器是与操作系统紧密相关的，先后顺序是不能人为的干预的。对同一份资源操作时，会存在资源抢夺的问题，需要加入并发控制
- 线程会带来额外的开销，如 cpu 调度时间，并发控制开销。
- 每个线程在自己的工作内存交互，内存控制不当会造成数据不一致

## 线程创建

### 继承 Thread 类

1. 自定义线程类**继承Thread类**
2. **重写run()方法**，编写线程执行体
3. 创建自定义线程类对象，调用`start()`**方法启动线程**

线程开启不一定立即执行，由 CPU 调度执行
```java
/**
  * 每次输出结果都不一样
  */
public class TestThread1 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            System.out.println("TestThread1---" + i);
        }
    }

    public static void main(String[] args) {
        //main线程，主线程

        //创建一个线程对象
        TestThread1 testThread1 = new TestThread1();
        //调用 start 方法开启线程
        testThread1.start();

        for (int i = 0; i < 2000; i++) {
            System.out.println("main---" + i);
        }
    }
}
```
### 实现 Runnable 接口

1. 定义 MyRunnable 类实现 Runnable 接口
2. 重写 `run()` 方法，编写线程执行体
3. 创建 Thread 对象，调用 start() 方法启动线程

```java
public class TestThread3 implements Runnable {

    @Override
    public void run() {
        for (int i = 0; i < 200; i++) {
            System.out.println("TestThread3---" + i);
        }
    }

    public static void main(String[] args) {
        // 创建 Runnable接口实现类对象
        TestThread3 testThread3 = new TestThread3();
        // 创建Thread对象
        Thread thread = new Thread(testThread3);
        // 启动线程
        thread.start();

        for (int i = 0; i < 2000; i++) {
            System.out.println("main---" + i);
        }
    }
}
```
### 两种方式的比较
继承 Thread 类

- 子类继承 Thread 类具备多线程能力
- 启动线程：子类对象. start()

实现 Runnable 接口

- 实现接口 Runnable 具有多线程能力
- 启动线程：传入目标对象+Thread对象.start()

开发中优先选择**实现Runnable接口**的方式<br />原因：

- 实现的方式没有**类的单继承性的局限性**
- 实现的方式更适合来处理**多个线程有共享数据**的情况。
> 没有更好，只有更适合的业务场景
> 如果实现 Runnable 接口不能保证同一个对象被多个线程使用，不如继承 Thread 类，见“死锁案例”


### 初识并发问题
案例：模拟抢票
```java
public class TestThread4 implements Runnable {

    private int ticketNums = 10;

    @Override
    public void run() {
        while (true) {
            if (ticketNums <= 0)
                break;
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "--->拿到了第" + ticketNums-- + "张票");
        }
    }

    public static void main(String[] args) {
        TestThread4 testThread4 = new TestThread4();
        new Thread(testThread4, "Gala").start();
        new Thread(testThread4, "XiaoHu").start();
        new Thread(testThread4, "Ming").start();
    }
}
```
发现问题：多个线程操作同一个资源的情况下，线程不安全，数据紊乱。
### 静态代理
真实对象和代理对象都要实现同一个接口。代理对象要代理真实角色<br />好处：

- 代理对象可以做很多真实对象做不了的事情
- 真实对象专注做自己的事情

**Thread类也实现了Runnable接口**
```java
public class StaticProxy {
    public static void main(String[] args) {
        You you = new You(); //你要结婚
        WeddingCompany weddingCompany = new WeddingCompany(you);
        weddingCompany.HappyMarry();
    }
}

interface Marry{
    void HappyMarry();
}

//真实角色，你去结婚
class You implements Marry{

    @Override
    public void HappyMarry() {
        System.out.println("秦老师要结婚了，超开心");
    }
}

//代理角色，帮助你结婚
class WeddingCompany implements Marry{

    //代理谁-->真实目标角色
    private Marry target;

    public WeddingCompany(Marry target) {
        this.target = target;
    }

    @Override
    public void HappyMarry() {
        before();
        this.target.HappyMarry();//这就是真实对象
        after();
    }

    private void after() {
        System.out.println("结婚之后，付尾款，很痛苦");
    }

    private void before() {
        System.out.println("结婚之前，很幸福");
    }
}
```
## 线程的五大状态
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1700101884865-acbb55d9-1f8d-47eb-bf0b-08d9b3f01dcd.png#averageHue=%23f9f7f4&clientId=ub9577af6-303a-4&from=paste&height=173&id=udd90fbfb&originHeight=346&originWidth=1120&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=163294&status=done&style=none&taskId=u5f6db8ed-34d3-41cc-990d-40d8671cd13&title=&width=560)<br />Java线程的五大状态<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1700102089142-fead9c8d-29b5-4643-992f-242a11181242.png#averageHue=%23f3f3f3&clientId=ub9577af6-303a-4&from=paste&height=344&id=u0b2a3e61&originHeight=688&originWidth=1408&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=317951&status=done&style=none&taskId=u2bc5b83f-c1d7-4ba2-8386-46cd7458fb8&title=&width=704)<br />操作线程的方法<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1700102237491-504f48ba-8716-4f8b-819a-d88ee7c688d3.png#averageHue=%23a6b8d6&clientId=ub9577af6-303a-4&from=paste&height=267&id=ufb6043d0&originHeight=533&originWidth=1167&originalType=binary&ratio=1.5&rotation=0&showTitle=false&size=298821&status=done&style=none&taskId=ue06f9ba6-3b31-49fa-b067-329775b5864&title=&width=584)
### 停止线程（标志位）
不推荐使用JDK提供的 `stop()`、`destroy()`方法【已废弃】<br />推荐线程自己停止下来，建议使用一个标志位进行终止变量，当flag=false，则终止线程运行。
```java
public class TestStop implements Runnable {

    //1.设置一个标志位
    private boolean flag = true;

    @Override
    public void run() {
        int i = 0;
        while (flag) {
            System.out.println("run ... Thread" + i++);
        }
    }

    //2.设置一个公开的方法停止线程，转换标志位
    public void stop() {
        this.flag = false;
    }

    public static void main(String[] args) {
        TestStop testStop = new TestStop();
        new Thread(testStop).start();
        for (int i = 0; i < 1000; i++) {
            System.out.println("main" + i);
            if (i == 900) {
                //3.调用stop方法切换标志位,让线程停止
                testStop.stop();
                System.out.println("线程该停止了");
            }
        }
    }
}
```
### 线程休眠（sleep）
`sleep (时间)`：指定当前线程阻塞的毫秒数，可用于模拟网络延时，放大问题的发生性<br /> `public static native void sleep(long millis)`

- sleep 存在异常 `InterruptedException`
- sleep 时间达到后线程进入就绪状态
- sleep 可以模拟网络延时，倒计时等
- **每一个对象都有一个锁，sleep 不会释放锁**

抢票案例：
```java
public class TestSleep implements Runnable {
    private int ticketNums = 10;

    @Override
    public void run() {
        while (true) {
            if (ticketNums <= 0) break;
            //模拟延时
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "---> 拿到了第" + ticketNums-- + "张票");
        }
    }

    public static void main(String[] args) {
        TestSleep ticket = new TestSleep();
        new Thread(ticket, "小明").start();//name：线程的名字
        new Thread(ticket, "张三").start();
        new Thread(ticket, "黄牛").start();
    }
}
```
案例：打印系统当前时间**（秒级）**
```java
public class TestSleep2 {
     public static void main(String[] args) throws InterruptedException {
         //打印系统当前时间
         Date startTime = new Date();
         while (true) {
             Thread.sleep(1000);
             System.out.println(new SimpleDateFormat("HH:mm:ss").format(startTime));
             startTime = new Date(System.currentTimeMillis());//更新当前时间
         }
     }
 }
```
### 线程礼让（yield）
`public static native void yield();`

- 礼让线程，让当前正在执行的线程暂停，但不阻塞
- 将线程从运行状态转为就绪状态
- 让cpu重新调度，**礼让不一定成功！**看CPU心情
```java
public class TestYield {
    public static void main(String[] args) {
        MyYield myYield = new MyYield();
        new Thread(myYield, "小明").start();
        new Thread(myYield, "小英").start();
    }
}

class MyYield implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程开始执行");
        Thread.yield();
        System.out.println(Thread.currentThread().getName() + "线程结束执行");
    }
}
```
### 线程强制执行（join）
`public final void join() throws InterruptedException`

- Join 合并线程，待此线程执行完成后，再执行其他线程，其他线程阻塞
- 可以想象成插队
```java
public class TestJoin implements Runnable{
    @Override
    public void run() {
        for (int i = 0; i < 500; i++) {
            System.out.println("线程vip来了" + i);
        }
    }

    public static void main(String[] args) throws InterruptedException {
        // 启动我们的线程
        Thread vipThread = new Thread(new TestJoin());
        vipThread.start();
        // 主线程
        for (int i = 0; i < 500; i++) {
            if (i == 200) {
                vipThread.join(); // 插队，该线程全部执行完再轮到主线程
            }
            System.out.println("main" + i);
        }
    }
}
```
### 线程状态观测（getState）
线程状态`Thread.State`，Thread类中的枚举类`public enum State{}`<br />获取当前线程的状态`public State getState();`<br />线程可以处于以下状态之一：

- `NEW`尚未启动的线程处于此状态。
- `RUNNABLE`在 Java 虚拟机中执行的线程处于此状态。
- `BLOCKED`被阻塞等待监视器锁定的线程处于此状态。
- `WAITING`正在等待另一个线程执行特定动作的线程处于此状态。
- `TIMED_ WAITING`正在等待另一个线程执行动作达到指定等待时间的线程处于此状态。
- `TERMINATED`已退出的线程处于此状态。

一个线程可以在给定时间点处于一个状态。这些状态是**不反映任何操作系统线程状态的虚拟机状态**。<br />观察测试线程的状态：
```java
public class TestState {
    public static void main(String[] args) throws InterruptedException {
        // Runnable接口是函数式接口，因此可以用lambda表达式
        Thread thread = new Thread(() -> {
            for (int i = 0; i < 5; i++) { // 循环5次，每次睡1s
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            System.out.println("********");
        });
        
        // 未启动
        Thread.State state = thread.getState();
        System.out.println(state); // NEW

        // 启动后
        thread.start();
        state = thread.getState();
        System.out.println(state); // RUNNABLE

        while (state != Thread.State.TERMINATED) { // 只要线程不终止，就一直输出状态
            Thread.sleep(100); // 主线程睡眠
            state = thread.getState();
            System.out.println(state); // TIMED_WAITING
        }
    }
}
```
### 线程优先级（PRIORITY）
Java提供一个线程调度器来监控程序中启动后进入就绪状态的所有线程，线程调度器按照优先级决定应该调度哪个线程来执行。<br />线程的优先级用数字表示，范围从1~10

- `Thread.MIN_PRIORITY= 1;`
- `Thread.MAX_PRIORITY =10;`
- `Thread.NORM_PRIORITY = 5;`

使用以下方式改变或获取优先级
```java
public final void setPriority(int newPriority)
public final int getPriority()
```
优先级低只是意味着获得调度的概率低，并不是优先级低就不会被调用了，这都是看CPU的调度<br />优先级的设定建议在`start()`调度前

测试线程的优先级
```java
public class TestPriority {
    public static void main(String[] args) {
        // 主线程默认优先级
        Thread currentThread = Thread.currentThread();
        System.out.println(currentThread.getName() + "--->" + currentThread.getPriority()); // 5

        MyPriority myPriority = new MyPriority();
        Thread t1 = new Thread(myPriority, "t1");
        Thread t2 = new Thread(myPriority, "t2");
        Thread t4 = new Thread(myPriority, "t4");

        //先设置优先级，再启动
        t1.start(); // 5

        t2.setPriority(1);
        t2.start(); // 1

        t4.setPriority(Thread.MAX_PRIORITY);
        t4.start(); // 10
    }
}

class MyPriority implements Runnable {
    @Override
    public void run() {
        Thread currentThread = Thread.currentThread();
        System.out.println(currentThread.getName() + "--->" + currentThread.getPriority());
    }
}
```
### 守护(daemon)线程
线程分为**用户线程**和**守护线程**

- 用户线程：虚拟机必须确保用户线程执行完毕
- 守护线程：虚拟机不用等待守护线程执行完毕（如后台记录操作日志，监控内存，垃圾回收等待…）

默认创建的是用户线程

测试守护线程：
```java
public class TestDaemon {
    public static void main(String[] args) {
        Thread godThread = new Thread(new God());
        godThread.setDaemon(true); // 默认是false表示是用户线程，正常的线程都是用户线程...
        godThread.start(); // 上帝 守护线程启动
        new Thread(new You1()).start();// 你 用户线程启动
    }
}

// 上帝 守护线程
class God implements Runnable {
    @Override
    public void run() {
        while (true) { // 无限循环
            System.out.println("上帝保佑着你！");
        }
    }
}

// 你 用户线程
class You1 implements Runnable {
    @Override
    public void run() {
        for (int i = 0; i < 36500; i++) {
            System.out.println("你活了第" + i + "天");
        }
        System.out.println("翘辫子啦！");
    }
}
```

## 线程同步
多个线程操作同一个资源
> 现实生活中,我们会遇到 ”同一个资源,多个人都想使用” 的问题，比如,食堂排队打饭，每个人都想吃饭，最天然的解决办法就是排队，一个个来。

处理多线程问题时，多个线程访问同一个对象，并且某些线程还想修改这个对象。这时候我们就需要线程同步，线程同步其实就是一种等待机制，多个需要同时访问此对象的线程进入这个 **对象的等待池** 形成队列，等待前面线程使用完毕，下一个线程再使用<br />队列和锁：

- 排队上厕所，每个进入厕所的人都会锁上门，上完厕所后再释放锁，让后面的人进去
- 线程同步的形成条件：队列+锁
### 三个线程不安全案例
#### 1、不安全的买票
```java
/**
  * 不安全的买票
  * 线程不安全，有人抢到同一张票，或者0和负数票
  */
public class UnSafeBuyTicket {
    public static void main(String[] args) {
        BuyTicket station = new BuyTicket();
        new Thread(station, "熊大").start();
        new Thread(station, "熊二").start();
        new Thread(station, "黄牛").start();
    }
}

class BuyTicket implements Runnable {
    private int ticketNums = 10; // 剩余票数
    private boolean flag = true; // 线程停止标志位

    @Override
    public void run() {
        // 买票
        while (flag) {
            try {
                buy();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void buy() throws InterruptedException {
        // 判断是否有票
        if (ticketNums <= 0) {
            flag = false;
            return;
        }
        // 模拟延时
        Thread.sleep(100);
        // 买票
        System.out.println(Thread.currentThread().getName() + "抢到了" + ticketNums--);
    }
}
```
#### 2、不安全的取钱
```java
/**
  * 不安全的取钱
  * 两个人去银行取钱
  */
public class UnsafeBank {
    public static void main(String[] args) {
        //账户
        Account account = new Account(100, "结婚基金");

        Drawing you = new Drawing(account, 50, "你");
        Drawing girlFriend = new Drawing(account, 100, "girlFriend");

        you.start();
        girlFriend.start();
    }
}

//账户
class Account {
    int money;//余额
    String name;//卡名

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }
}

//银行：模拟取款
class Drawing extends Thread {
    Account account; //账户
    int drawingMoney; //取了多少钱
    int nowMoney; //现在手里有多少钱

    public Drawing(Account account, int drawingMoney, String name) {
        super(name); //线程名
        this.account = account;
        this.drawingMoney = drawingMoney;
    }

    //取钱
    @Override
    public void run() {
        //判断有没有钱
        if (account.money - drawingMoney < 0) {
            System.out.println("余额不足，" + Thread.currentThread().getName() + "取不了！");
            return;
        }
        //模拟延时
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //卡内余额=余额-要取的钱数
        account.money = account.money - drawingMoney;
        //你手里的钱
        nowMoney = nowMoney + drawingMoney;

        System.out.println(account.name + "余额为：" + account.money);
        //Thread.currentThread().getName()=this.getName()
        System.out.println(this.getName() + "手里的钱为：" + nowMoney);
    }
}
```
#### 3、线程不安全的集合
结果不到10000个，原因是`new Thread()`新创建的线程可能并不会立刻执行，这会导致可能出现多个线程在同一个下标位置进行 add 操作，就覆盖了
```java
public class UnSafeList {
    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10000; i++) {
            new Thread(() -> {
                list.add(Thread.currentThread().getName());
            }).start();
        }
        // 线程睡眠得越久，下面的结果越准确，因为线程都执行完了，但仍可能出现覆盖现象
        Thread.sleep(3000);
        System.out.println(list.size()); // 不到10000
    }
}
```

### 同步方法及同步块 synchronized

由于同一进程的多个线程共享同一块存储空间，在带来方便的同时，也带来了访问冲突问题，为了保证数据在方法中被访问时的正确性，在访问时加入锁机制 `synchronized`，当一个线程获得对象的排它锁，独占资源，其他线程必须等待，使用后释放锁即可<br />存在以下问题：

- 一个线程持有锁会导致其他所有需要此锁的线程挂起
- 在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题
- 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能问题

由于我们可以通过 private 关键字来保证数据对象只能被方法访问，所以我们只需要针对方法提出一套机制，就是 `**synchronized** `关键字，它包括两种用法：`**synchronized**`**方法**和`**synchronized**`**块**
#### 同步方法
`public synchronized void method(int args) {}`<br />`synchronized`方法控制对 “对象” 的访问

- 每个对象对应一把锁，每个`synchronized`方法都必须获得调用该方法的对象的锁才能执行，否则线程会阻塞
- 方法一旦执行就独占该锁，直到该方法返回才释放锁，后面被阻塞的线程才能获得这个锁，继续执行

**缺陷：**若将一个大的方法申明为 `synchronized` 将会影响效率<br />`**synchronized**`**默认锁的是this**

修改之前的不安全买票案例，给buy()方法加上`synchronized`
```java
public class UnSafeBuyTicket {
    public static void main(String[] args) {
        BuyTicket station = new BuyTicket();
        new Thread(station, "熊大").start();
        new Thread(station, "熊二").start();
        new Thread(station, "黄牛").start();
    }
}

class BuyTicket implements Runnable {
    private int ticketNums = 10; // 票数
    private boolean flag = true; // 外部停止方式

    @Override
    public void run() {
        // 买票
        while (flag) {
            try {
                buy();
                // 模拟延时，买到以后停一停，不然票容易被一个人抢光
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    // synchronized 同步方法，锁的是 this
    private synchronized void buy() throws InterruptedException {
        // 判断是否有票
        if (ticketNums <= 0) {
            flag = false;
            return;
        }
        // 模拟延时
        Thread.sleep(100);
        // 买票
        System.out.println(Thread.currentThread().getName() + "抢到了" + ticketNums--);
    }
}
```
> 不安全的取钱案例中，给run()方法加synchronized是没用的，`while (flag)`无限执行，一个人就抢完了
> 解决方法：同步块


#### 同步块
`synchronized (Obj ){}`<br />Obj 称之为**同步监视器**

- Obj 可以是**任何对象**，但是推荐使用共享资源作为同步监视器
- 同步方法中无需指定同步监视器，因为同步方法的同步监视器就是 this ，就是这个对象本身，或者是class[反射中讲解]

同步监视器的执行过程

1. 第一个线程访问，锁定同步监视器，执行其中代码。
2. 第二个线程访问，发现同步监视器被锁定，无法访问。
3. 第一个线程访问完毕，解锁同步监视器
4. 第二个线程访问，发现同步监视器没有锁，然后锁定并访问

修改不安全的取钱案例：<br />在 Drawing 类的 run() 方法中加上 synchronized 修饰，发现还是线程不安全的，因为 synchronized 默认锁的是 this，即you和girlfriend两个不同的对象<br />产生线程不安全的原因是两个线程同时操作了共享资源 account 对象（you和girlfriend取的都是account的钱），因此使用**同步块锁住共享资源 Account 对象**
```java
public class UnsafeBank {
    public static void main(String[] args) {
        // 账户
        Account account = new Account(500, "结婚基金");

        Drawing you = new Drawing(account, 50, "你");
        Drawing girlFriend = new Drawing(account, 100, "girlFriend");

        you.start();
        girlFriend.start();
    }
}

/**
 * 账户
 */
class Account {
    int money; // 余额
    String name; // 卡名

    public Account(int money, String name) {
        this.money = money;
        this.name = name;
    }
}

/**
 * 银行：模拟取款
 */
class Drawing extends Thread {
    Account account;  // 账户
    int drawingMoney;  // 一次取多少钱
    int nowMoney;  // 现在手里有多少钱

    boolean flag = true;

    public Drawing(Account account, int drawingMoney, String name) {
        super(name);  // 线程名
        this.account = account;
        this.drawingMoney = drawingMoney;
    }

    // 取钱
    @Override
    public void run() {
        while (flag) {
            // 将 account 作为同步监视器
            synchronized (account) {
                // 判断account有没有钱
                if (account.money - drawingMoney < 0) {
                    System.out.println(String.format("余额：%s，已不足，%s取不了！", account.money, Thread.currentThread().getName()));
                    flag = false;
                    return;
                }
                // 模拟延时
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 卡内余额 = 余额 - 要取的钱数
                account.money = account.money - drawingMoney;
                // 你手里的钱
                nowMoney = nowMoney + drawingMoney;

                System.out.println(String.format("=====%s余额为：%s=====", account.name, account.money));
                System.out.println(this.getName() + "手里的钱为：" + nowMoney);
            }
        }
    }
}
```

修改不安全的集合案例，将list对象做为同步监视器
```java
public class UnSafeList {
    public static void main(String[] args) throws InterruptedException {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 10000; i++) {
            new Thread(() -> {
                // 对 list 上锁
                synchronized (list) {
                    list.add(Thread.currentThread().getName());
                }
            }).start();
        }
        Thread.sleep(3000); // 这里需要延迟一下主线程，因为new了1w个线程还没执行完
        System.out.println(list.size()); // 10000
    }
}
```

### 死锁
多个线程各自占有一些共享资源，并且互相等待其他线程占有的资源才能运行，而导致**两个或者多个线程都在等待对方释放资源，都停止执行**的情形。

**某一个同步块同时拥有两个以上对象的锁**时，就可能会发生"死锁"的问题。

> 互相持有对方的锁对象


死锁案例：
```java
/**
 * 死锁:多个线程互相抱着对方需要的资源，然后形成僵持
 */
public class DeadLock {
    public static void main(String[] args) {
        Makeup g1 = new Makeup(0, "静香");
        Makeup g2 = new Makeup(1, "冰冰");
        g1.start();
        g2.start();
    }
}

// 口红
class Lipstick {}

// 镜子
class Mirror {}

// 化妆线程
class Makeup extends Thread {
    // 需要的资源只有一份，用 static 保证只有一份
    static Lipstick lipstick = new Lipstick();
    static Mirror mirror = new Mirror();

    int choice; // 选择
    String girlName; // 使用化妆品的人

    public Makeup(int choice, String girlName) {
        this.choice = choice;
        this.girlName = girlName;
    }

    @Override
    public void run() {
        //化妆
        try {
            makeup();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // 化妆，互相持有对方的锁，就是需要拿到对方的资源
    private void makeup() throws InterruptedException {
        if (choice == 0) {
            // 获得口红的锁
            synchronized (lipstick) {
                System.out.println(this.girlName + "获得口红的锁");
                Thread.sleep(1000);
                // 一秒后想获得镜子的锁
                synchronized (mirror) {
                    System.out.println(this.girlName + "获得镜子的锁");
                }
            }
        } else {
            // 获得镜子的锁
            synchronized (mirror) {
                System.out.println(this.girlName + "获得镜子的锁");
                Thread.sleep(2000);
                // 两秒后想获得口红的锁
                synchronized (lipstick) {
                    System.out.println(this.girlName + "获得口红的锁");
                }
            }
        }
    }

}
```

**死锁避免方法**<br />产生死锁的四个必要条件:

- 互斥条件：一个资源每次只能被一个进程使用。
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件：进程已获得的资源，在末使用完之前，不能强行剥夺。
- 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

上面列出了死锁的四个必要条件，我们只要想办法破其中的**任意一个或多个**条件就可以避免死锁发生

### Lock锁
从 JDK 5.0 开始，Java 提供了更强大的线程同步机制——通过显式定义同步锁对象来实现同步。同步锁使用Lock对象充当

- `java.util.concurrent.locks.Lock` 接口是控制多个线程对共享资源进行访问的工具。锁提供了对共享资源的独占访问，**每次只能有一个线程对 Lock 对象加锁**，线程开始访问共享资源之前应先获得 Lock 对象
- `ReentrantLock`类实现了`Lock`，它拥有与`synchronized`相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是`ReentrantLock`，可以**显式加锁、释放锁**
> `ReentrantLock`：可重入锁

```java
/**
 * 测试Lock锁
 */
public class TestLock {
    public static void main(String[] args) {
        TestLock2 testLock2 = new TestLock2();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
        new Thread(testLock2).start();
    }
}

class TestLock2 implements Runnable {
    int ticketNum = 1000;

    // 定义Lock锁
    private final ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            // 需要保证线程安全的代码，放在 try 里
            try {
                lock.lock(); // 加锁
                if (ticketNum > 0) {
                    System.out.println(String.format("%s 拿到了第%s张票", Thread.currentThread().getName(), ticketNum--));
                } else {
                    break;
                }
            } finally {
                lock.unlock(); // 解锁
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

### synchronized 与 Lock 的对比

- Lock是**显式锁**（手动开启和关闭锁，别忘记关闭锁) 。synchronized是**隐式锁**，出了作用域自动释放
- Lock**只有代码块锁**，synchronized有**代码块锁和方法锁**
- 使用Lock锁，JVM将花费较少的时间来调度线程，**性能更好**。并且具有更好的扩展性（提供更多的子类）

优先使用顺序：Lock > 同步代码块（已经进入了方法体，分配了相应资源）> 同步方法（在方法体之外）

## 线程通信

应用场景：生产者和消费者问题

- 假设仓库中只能存放一件产品，生产者将生产出来的产品放入仓库，消费者将仓库中产品取走消费。
- 如果仓库中没有产品，则生产者将产品放入仓库，否则停止生产并等待，直到仓库中的产品被消费者取走为止。
- 如果仓库中放有产品，则消费者可以将产品取走消费，否则停止消费并等待，直到仓库中再次放入产品为止。

分析：这是一个线程同步问题，生产者和消费者共享同一个资源，并且生产者和消费者之间相互依赖、互为条件

- 对于生产者，没有生产产品之前，要**通知消费者等待**，而生产了产品之后，又需要马上**通知消费者消费**
- 对于消费者，在消费之后，要**通知生产者已经结束消费**，需要生产新的产品以供消费
- 在生产者消费者问题中，仅有 synchronized 是不够的
   - synchronized 可阻止并发更新同一个共享资源，实现了同步
   - **synchronized 不能用来实现不同线程之间的消息传递(通信)**

**Java提供了几个方法解决线程之间的通信问题**

1. `wait()`：使当前线程进入阻塞状态，一直等待，并**释放同步监视器。**
   1. `wait(long timeout)`：指定等待的毫秒数
2. `notify()`：唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个
3. `notifyAll()`：唤醒同一个对象上所有被wait的线程，优先级别高的优先调度

注意：

- 均是 **Object类 **的方法
- **只能在同步方法或者同步代码块中使用**
   - 否则会抛出异常`llIegalMonitorStateException`
- 三个方法的**调用者必须是**同步代码块或同步方法中的**同步监视器**。
   - 否则，会出现`IllegalMonitorStateException`

### 管程法
并发协作模型 “生产者/消费者模式”—>管程法

- 生产者：负责生产数据的模块(可能是方法，对象，线程，进程);
- 消费者：负责处理数据的模块(可能是方法，对象，线程，进程);
- **缓冲区：**消费者不能直接使用生产者的数据，他们之间有个"缓冲区"

生产者将生产好的数据放入缓冲区，消费者从缓冲区拿出数据

```java
// 测试:生产者消费者模型-->利用缓冲区解决:管程法
// 生产者，消费者，产品，缓冲区
public class TestPc {
    public static void main(String[] args) {
        SynContainer container = new SynContainer();
        new Productor(container).start();
        new Consumer(container).start();
    }
}

//生产者
class Productor extends Thread {
    SynContainer container; //缓冲区

    public Productor(SynContainer container) {
        this.container = container;
    }

    //生产
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            container.push(new Chicken(i));
            System.out.println("生产了第" + i + "只鸡");
        }
    }
}

//消费者
class Consumer extends Thread {
    SynContainer container;

    public Consumer(SynContainer container) {
        this.container = container;
    }

    //消费
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了第" + container.pop().id + "只鸡");
        }
    }
}

//产品
class Chicken {
    int id;

    public Chicken(int id) {
        this.id = id;
    }
}

//缓冲区
class SynContainer {
    //需要一个容器大小
    Chicken[] chickens = new Chicken[10];
    //容器计数器
    int count = 0;

    //生产者放入产品
    public synchronized void push(Chicken chicken) {
        //如果容器满了，就需要等待消费者消费
        if (count == chickens.length) {
            //通知消费者消费，等待生产
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果没有满，我们就需要丢入产品
        chickens[count] = chicken;
        count++;
        //可以通知消费者消费
        this.notifyAll();
    }

    //消费者消费产品
    public synchronized Chicken pop() {
        //判断能否消费
        if (count == 0) {
            //等待生产者生产
            try {
                this.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        //如果可以消费
        count--;
        Chicken chicken = chickens[count];

        //吃完了，通知生产者生产
        this.notifyAll();

        return chicken;
    }
}
```
### 信号灯法
```java
/**
 * 生产者消费者问题: 信号灯法,标志位解决
 */
public class TestPc2 {
    public static void main(String[] args) {
        Tv tv = new Tv();
        new Player(tv).start();
        new Watcher(tv).start();
    }
}

//生产者--->演员
class Player extends Thread {
    Tv tv;

    public Player(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if (i % 2 == 0) {
                this.tv.play("快乐大本营播放中");
            } else {
                this.tv.play("广告ing...");
            }
        }
    }
}

//消费者--->观众
class Watcher extends Thread {
    Tv tv;

    public Watcher(Tv tv) {
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            this.tv.watch();
        }
    }
}

// 产品--->节目
class Tv {
    // 演员表演,观众等待  flag = true

    // 观众观看,演员等待 flag = false
    String voice; // 表演的节目
    boolean flag = true;

    // 表演
    public synchronized void play(String voice) {
        if (!flag) {
            try {
                this.wait(); // 有观众观看，阻塞
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("演员表演了：" + voice);
        // 通知观众观看
        this.notifyAll();
        this.voice = voice; // 更新表演的节目
        this.flag = !this.flag;
    }

    // 观看
    public synchronized void watch() {
        if (flag) {
            try {
                this.wait(); // 演员表演，阻塞
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println("观众观看了：" + voice);
        // 通知演员表演
        this.notifyAll();
        this.flag = !this.flag;
    }
}
```

### 面试题：`sleep()` 和 `wait()` 的异同？

相同点：一旦执行方法，都可以使得当前的线程进入**阻塞状态**<br />不同点：

- 声明的位置： 
   - Thread类中声明sleep() 
   - Object类中声明wait()
- 调用的要求： 
   - sleep()可以在任何需要的场景下调用。
   - wait()必须使用在同步代码块或同步方法中（调用它的前提是当前线程占有锁）
- 是否释放同步监视器： 
   - 如果两个方法都使用在同步代码块或同步方法中 
      - sleep()不会释放锁
      - wait()会释放锁

## JDK5新增两种线程创建方式

### 实现 Callable 接口

- 创建 `Callable` 的实现类，实现 `call` 方法
- 创建 `Callable` 对象 object1
- 创建 `FutureTask` 的对象 object2，参数为 object1
- 创建 `Thread` 对象，参数为 object2，并调用 start()
- `futureTask.get()` 获取 Callable 中 call 方法的返回值

```java
// 1.创建一个实现Callable的实现类
class NumThread implements Callable {
    // 2.实现call方法，将此线程需要执行的操作声明在call()中
    @Override
    public Object call() throws Exception {
        int sum = 0;
        for (int i = 1; i <= 100; i++) {
            if (i % 2 == 0) {
                System.out.println(i);
                sum += i;
            }
        }
        return sum;
    }
}

public class ThreadNew {
    public static void main(String[] args) {
        // 3.创建Callable接口实现类的对象
        NumThread numThread = new NumThread();
        // 4.将此Callable接口实现类的对象作为传递到FutureTask构造器中，创建FutureTask的对象
        FutureTask futureTask = new FutureTask(numThread);
        // 5.将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()
        new Thread(futureTask).start();

        try {
            // 6.获取Callable中call方法的返回值
            // get()返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。
            Object sum = futureTask.get();
            System.out.println("总和为：" + sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

}
```

实现 `Callable` 接口的方式创建多线程 比 实现 `Runnable` 接口创建多线程方式强大？

1. `call()` 可以有返回值。
2. `call()` 可以抛出异常，被外面的操作捕获，获取异常的信息
3. `Callable` 支持泛型

### 线程池

背景：经常创建和销毁、使用量特别大的资源，比如并发情况下的线程，对性能影响很大。

思路：**提前创建**好多个线程，放入线程池中，**使用时直接获取**，使用完放回池中。可以避免频繁创建销毁、实现重复利用。类似生活中的公共交通工具。

好处:
- 提高响应速度（减少了创建新线程的时间)
- 降低资源消耗（重复利用线程池中线程，不需要每次都创建)
- 便于线程管理

线程池参数：
   - `corePoolSize`：核心池的大小
   - `maximumPoolSize`：最大线程数
   - `keepAliveTime`：线程没有任务时最多保持多长时间后会终止

JDK 5.0 起提供了线程池相关 API: `ExecutorService` 和 `Executor`
- `ExecutorService`：真正的线程池接口。常见子类 `ThreadPoolExecutor`
   - （1）`void execute(Runnable command)`︰执行任务/命令，没有返回值，**一般用来执行 Runnable**
   - （2）`<T> Future<T> submit(Callable<T> task)`：执行任务，有返回值，**一般用来执行 Callable**
   - （3）`void shutdown()`︰关闭连接池
- `Executors`：工具类、线程池的工厂类，用于创建并返回不同类型的线程池

```java
//测试线程池
public class TestPool {
    public static void main(String[] args) {
        // 1.创建服务，创建线程池
        ExecutorService service = Executors.newFixedThreadPool(10); // 10是线程池大小

        service.execute(new MyThread1());
        service.execute(new MyThread1());
        service.execute(new MyThread1());
        service.execute(new MyThread1());

        // 2.关闭连接
        service.shutdown();
    }
}

class MyThread1 implements Runnable{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
    }
}
```
<br /> 
