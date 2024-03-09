# Java8新特性介绍
Java 8 (又称为 JDK 8或JDK1.8) 是 Java 语言开发的一个主要版本。 Java 8 是oracle公司于2014年3月发布，可以看成是自Java 5 以来最具革命性的版本。Java 8为Java语言、编译器、类库、开发工具与JVM带来了大量新特性。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1692621575623-44374dab-27fb-40ec-9362-06f425f6d33d.png#averageHue=%23fafafa&clientId=u2d106317-defa-4&from=paste&height=620&id=u2b4982c4&originHeight=869&originWidth=690&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=53349&status=done&style=none&taskId=ua986633d-2db0-48bd-a8f5-19dee41c375&title=&width=492.2684020996094)

-  速度更快 
-  代码更少(增加了新的语法：**Lambda** **表达式**) 
-  强大的 **Stream API** 
-  便于并行 
   - **并行流**就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。相比较串行的流，并行的流可以很大程度上提高程序的执行效率。
   - Java 8 中将并行进行了优化，我们可以很容易的对数据进行并行操作。Stream API 可以声明性地通过 `parallel()` 与 `sequential()` 在并行流与顺序流之间进行切换。
-  最大化减少空指针异常：`Optional` 
-  `Nashorn`引擎，允许**在JVM上运行JS应用 **
   - 发音“nass-horn”，是德国二战时一个坦克的命名
   - `javascript`运行在jvm已经不是新鲜事了，Rhino早在jdk6的时候已经存在。现在替代Rhino，官方的解释是Rhino相比其他JavaScript引擎（比如google的V8）实在太慢了，改造Rhino还不如重写。所以Nashorn的性能也是其一个亮点。
   - Nashorn 项目在 JDK 9 中得到改进；在JDK11 中`Deprecated`，后续JDK15版本中`remove`。在JDK11中取以代之的是GraalVM。（GraalVM是一个运行时平台，它支持Java和其他基于Java字节码的语言，但也支持其他语言，如JavaScript，Ruby，Python或LLVM。性能是Nashorn的2倍以上。）
# Lambda表达式
## 冗余的匿名内部类
当需要启动一个线程去完成任务时，通常会通过`java.lang.Runnable`接口来定义任务内容，并使用`java.lang.Thread`类来启动该线程。代码如下：
```java
public class UseFunctionalProgramming {
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("多线程任务执行！");
            }
        }).start(); // 启动线程
    }
}
```
本着“一切皆对象”的思想，这种做法是无可厚非的：首先创建一个`Runnable`接口的匿名内部类对象来指定任务内容，再将其交给一个线程来启动。
**代码分析：**
对于`Runnable`的匿名内部类用法，可以分析出几点内容：

- `Thread`类需要`Runnable`接口作为参数，其中的抽象`run`方法是用来指定线程任务内容的核心；
- 为了指定`run`的方法体，**不得不**需要`Runnable`接口的实现类；
- 为了省去定义一个`RunnableImpl`实现类的麻烦，**不得不**使用匿名内部类；
- 必须覆盖重写抽象`run`方法，所以方法名称、方法参数、方法返回值**不得不**再写一遍，且不能写错；
- 而实际上，**似乎只有方法体才是关键所在**。
## Lambda 及其使用举例
Lambda 是一个**匿名函数**，我们可以把 Lambda 表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。使用它可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使Java的语言表达能力得到了提升。

- 从匿名类到 Lambda 的转换举例1

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1692623595964-0fac8d20-3101-4478-abc2-0ea3036bbe7d.png#averageHue=%23fdf6f6&clientId=u1447eb48-9e69-4&from=paste&height=247&id=u0fe7718e&originHeight=487&originWidth=872&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=83195&status=done&style=none&taskId=u95b83c16-49e9-48ea-aaaa-9eb0e3d9aba&title=&width=442.9206617398133)

- 从匿名类到 Lambda 的转换举例2

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1692623610978-7189cbe4-29e5-4d4b-9ce2-c2507dc4a736.png#averageHue=%23faf4f3&clientId=u1447eb48-9e69-4&from=paste&height=252&id=uf4113bed&originHeight=497&originWidth=965&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=150546&status=done&style=none&taskId=u6e0ed9b6-c03d-4606-92ba-7bac5a4700f&title=&width=490.1587598382108)
## 语法
Lambda 表达式：在Java 8 语言中引入的一种新的语法元素和操作符。这个操作符为 “`->`” ， 该操作符被称为 `Lambda 操作符`或`箭头操作符`。它将 Lambda 分为两个部分：

- 左侧：指定了 Lambda 表达式需要的参数列表
- 右侧：指定了 Lambda 体，是**抽象方法**的实现逻辑，也即 Lambda 表达式要执行的功能。
> 接口中的方法默认是抽象的、无需使用abstract修饰
> 
> **在语法格式三** Lambda 表达式中的参数类型都是由编译器推断得出的。Lambda 表达式中无需指定类型，程序依然可以编译，这是因为 javac 根据程序的上下文，在后台推断出了参数的类型。Lambda 表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“**类型推断**”。
> ![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1692628827107-0d9dd522-baa1-4c91-a26b-bda6b523875e.png#averageHue=%23f7f4f2&clientId=u1447eb48-9e69-4&from=paste&height=93&id=uc4045c86&originHeight=145&originWidth=519&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=33007&status=done&style=none&taskId=u28a2622c-0e40-4e4e-8ac0-e0901265ccb&title=&width=334.6190490722656)
> 


**语法格式一：**无参，无返回值
```java
@Test
public void test1(){
    //未使用Lambda表达式
    Runnable r1 = new Runnable() {
        @Override
        public void run() {
            System.out.println("我爱北京天安门");
        }
    };
    r1.run();

    System.out.println("***********************");

    //使用Lambda表达式
    Runnable r2 = () -> {
        System.out.println("我爱北京故宫");
    };
    r2.run();
}
```
**语法格式二：**Lambda 需要一个参数，但是没有返回值。
```java
@Test
public void test2(){
    //未使用Lambda表达式
    Consumer<String> con = new Consumer<String>() {
        @Override
        public void accept(String s) {
            System.out.println(s);
        }
    };
    con.accept("谎言和誓言的区别是什么？");

    System.out.println("*******************");

    //使用Lambda表达式
    Consumer<String> con1 = (String s) -> {
        System.out.println(s);
    };
    con1.accept("一个是听得人当真了，一个是说的人当真了");
}
```
**语法格式三：**数据类型可以省略，因为可由编译器推断得出，称为“**类型推断**”
```java
@Test
public void test3(){
    //语法格式三使用前
    Consumer<String> con1 = (String s) -> {
        System.out.println(s);
    };
    con1.accept("一个是听得人当真了，一个是说的人当真了");

    System.out.println("*******************");
    //语法格式三使用后
    Consumer<String> con2 = (s) -> {
        System.out.println(s);
    };
    con2.accept("一个是听得人当真了，一个是说的人当真了");

}
```
 `Consumer<String>`中通过泛型已经给方法`void accept(T t)`中的`T`设置为`String`类型
```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);

    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}

```
**语法格式四：**Lambda 若只需要一个参数时，参数的小括号可以省略
```java
@Test
public void test4(){
    //语法格式四使用前
    Consumer<String> con1 = (s) -> {
        System.out.println(s);
    };
    con1.accept("一个是听得人当真了，一个是说的人当真了");

    System.out.println("*******************");
    //语法格式四使用后
    Consumer<String> con2 = s -> {
        System.out.println(s);
    };
    con2.accept("一个是听得人当真了，一个是说的人当真了");


}
```
**语法格式五：**Lambda 需要两个或以上的参数，多条执行语句，并且可以有返回值
```java
@Test
public void test5() {
    //语法格式五使用前
    Comparator<Integer> com1 = new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            System.out.println(o1);
            System.out.println(o2);
            return o1.compareTo(o2);
        }
    };
    System.out.println(com1.compare(12, 21));

    System.out.println("*****************************");

    //语法格式五使用后
    Comparator<Integer> com2 = (o1, o2) -> {
        System.out.println(o1);
        System.out.println(o2);
        return o1.compareTo(o2);
    };
    System.out.println(com2.compare(12, 6));
}
```
**语法格式六：**当 Lambda 体只有一条语句时，return 与大括号若有，都可以省略（**要省都省**，不然报错）
```java
@Test
public void test6(){
    //语法格式六使用前
    Comparator<Integer> com1 = (o1,o2) -> {
        return o1.compareTo(o2);
    };

    System.out.println(com1.compare(12,6));

    System.out.println("*****************************");
    //语法格式六使用后
    Comparator<Integer> com2 = (o1,o2) -> o1.compareTo(o2);

    System.out.println(com2.compare(12,21));
}

@Test
public void test7(){
    //语法格式六使用前
    Consumer<String> con1 = s -> {
        System.out.println(s);
    };
    con1.accept("一个是听得人当真了，一个是说的人当真了");

    System.out.println("*****************************");
    //语法格式六使用后
    Consumer<String> con2 = s -> System.out.println(s);

    con2.accept("一个是听得人当真了，一个是说的人当真了");
}
```
## Lambda表达式的本质
一方面，lambda表达式作为**接口的实现类的对象**。--->"万事万物皆对象

- 接口只有一个抽象

另一方面，lambda表达式是一个**匿名函数**。
# 函数式(Functional)接口
## 什么是函数式接口

- 只包含`一个抽象方法`（`Single Abstract Method`，简称`SAM`）的接口，称为函数式接口。当然该接口可以包含其他非抽象方法。
- 你可以通过 `Lambda`表达式来创建该接口的对象。（若`Lambda`表达式抛出一个受检异常(即：非运行时异常)，那么该异常需要在目标接口的抽象方法上进行声明）。
- 我们可以在一个接口上使用 `**@FunctionalInterface**` 注解，这样做可以**检查它是否是一个函数式接口**。同时 javadoc 也会包含一条声明，说明这个接口是一个函数式接口。
- 在`java.util.function`包下定义了Java 8 的丰富的函数式接口
## 如何理解函数式接口

- Java从诞生日起就是一直倡导“一切皆对象”，在Java里面面向对象(OOP)编程是一切。但是随着python、scala等语言的兴起和新技术的挑战，Java不得不做出调整以便支持更加广泛的技术要求，即Java不但可以支持OOP还可以支持OOF（面向函数编程） 
   - Java8引入了Lambda表达式之后，Java也开始支持函数式编程。
   - Lambda表达式不是Java最早使用的。目前C++，C#，Python，Scala等均支持Lambda表达式。
- 面向对象的思想： 
   - 做一件事情，找一个能解决这个事情的对象，调用对象的方法，完成事情。
- 函数式编程思想： 
   - 只要能获取到结果，谁去做的，怎么做的都不重要，重视的是结果，不重视过程。
- 在函数式编程语言当中，函数被当做一等公民对待。在将函数作为一等公民的编程语言中，Lambda表达式的类型是函数。但是在Java8中，有所不同。在Java8中，Lambda表达式是对象，而不是函数，它们必须依附于一类特别的对象类型——函数式接口。
- 简单的说，在Java8中，Lambda表达式就是一个函数式接口的实例。这就是Lambda表达式和函数式接口的关系。也就是说，只要一个对象是函数式接口的实例，那么该对象就可以用Lambda表达式来表示。
## Java 内置函数式接口
### 之前的函数式接口
之前学过的接口，有些就是函数式接口，比如：

- java.lang.Runnable 
   - public void run()
- java.lang.Iterable 
   - public Iterator iterate()
- java.lang.Comparable 
   - public int compareTo(T t)
- java.util.Comparator 
   - public int compare(T t1, T t2)
### 四大核心函数式接口
| 函数式接口 | 称谓 | 参数类型 | 用途 |
| --- | --- | --- | --- |
| `Consumer<T>` | 消费型接口 | T | 对类型为T的对象应用操作，
包含方法：  `void accept(T t)` |
| `Supplier<T>` | 供给型接口 | 无 | 返回类型为T的对象，
包含方法：`T get()` |
| `Function<T, R>` | 函数型接口 | T | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。
包含方法：`R apply(T t)` |
| `Predicate<T>` | 判断型接口 | T | 确定类型为T的对象是否满足某约束，并返回 boolean 值。
包含方法：`boolean test(T t)` |

### 其它接口
**类型1：消费型接口**
消费型接口的抽象方法特点：有形参，但是返回值类型是void

| 接口名 | 抽象方法 | 描述 |
| --- | --- | --- |
| BiConsumer<T,U> | void accept(T t, U u) | 接收两个对象用于完成功能 |
| DoubleConsumer | void accept(double value) | 接收一个double值 |
| IntConsumer | void accept(int value) | 接收一个int值 |
| LongConsumer | void accept(long value) | 接收一个long值 |
| ObjDoubleConsumer | void accept(T t, double value) | 接收一个对象和一个double值 |
| ObjIntConsumer | void accept(T t, int value) | 接收一个对象和一个int值 |
| ObjLongConsumer | void accept(T t, long value) | 接收一个对象和一个long值 |

**类型2：供给型接口**
这类接口的抽象方法特点：无参，但是有返回值

| 接口名 | 抽象方法 | 描述 |
| --- | --- | --- |
| BooleanSupplier | boolean getAsBoolean() | 返回一个boolean值 |
| DoubleSupplier | double getAsDouble() | 返回一个double值 |
| IntSupplier | int getAsInt() | 返回一个int值 |
| LongSupplier | long getAsLong() | 返回一个long值 |

**类型3：函数型接口**
这类接口的抽象方法特点：既有参数又有返回值

| 接口名 | 抽象方法 | 描述 |
| --- | --- | --- |
| UnaryOperator | T apply(T t) | 接收一个T类型对象，返回一个T类型对象结果 |
| DoubleFunction | R apply(double value) | 接收一个double值，返回一个R类型对象 |
| IntFunction | R apply(int value) | 接收一个int值，返回一个R类型对象 |
| LongFunction | R apply(long value) | 接收一个long值，返回一个R类型对象 |
| ToDoubleFunction | double applyAsDouble(T value) | 接收一个T类型对象，返回一个double |
| ToIntFunction | int applyAsInt(T value) | 接收一个T类型对象，返回一个int |
| ToLongFunction | long applyAsLong(T value) | 接收一个T类型对象，返回一个long |
| DoubleToIntFunction | int applyAsInt(double value) | 接收一个double值，返回一个int结果 |
| DoubleToLongFunction | long applyAsLong(double value) | 接收一个double值，返回一个long结果 |
| IntToDoubleFunction | double applyAsDouble(int value) | 接收一个int值，返回一个double结果 |
| IntToLongFunction | long applyAsLong(int value) | 接收一个int值，返回一个long结果 |
| LongToDoubleFunction | double applyAsDouble(long value) | 接收一个long值，返回一个double结果 |
| LongToIntFunction | int applyAsInt(long value) | 接收一个long值，返回一个int结果 |
| DoubleUnaryOperator | double applyAsDouble(double operand) | 接收一个double值，返回一个double |
| IntUnaryOperator | int applyAsInt(int operand) | 接收一个int值，返回一个int结果 |
| LongUnaryOperator | long applyAsLong(long operand) | 接收一个long值，返回一个long结果 |
| BiFunction<T,U,R> | R apply(T t, U u) | 接收一个T类型和一个U类型对象，返回一个R类型对象结果 |
| BinaryOperator | T apply(T t, T u) | 接收两个T类型对象，返回一个T类型对象结果 |
| ToDoubleBiFunction<T,U> | double applyAsDouble(T t, U u) | 接收一个T类型和一个U类型对象，返回一个double |
| ToIntBiFunction<T,U> | int applyAsInt(T t, U u) | 接收一个T类型和一个U类型对象，返回一个int |
| ToLongBiFunction<T,U> | long applyAsLong(T t, U u) | 接收一个T类型和一个U类型对象，返回一个long |
| DoubleBinaryOperator | double applyAsDouble(double left, double right) | 接收两个double值，返回一个double结果 |
| IntBinaryOperator | int applyAsInt(int left, int right) | 接收两个int值，返回一个int结果 |
| LongBinaryOperator | long applyAsLong(long left, long right) | 接收两个long值，返回一个long结果 |

**类型4：判断型接口**
这类接口的抽象方法特点：有参，但是返回值类型是boolean结果。

| 接口名 | 抽象方法 | 描述 |
| --- | --- | --- |
| BiPredicate<T,U> | boolean test(T t, U u) | 接收两个对象 |
| DoublePredicate | boolean test(double value) | 接收一个double值 |
| IntPredicate | boolean test(int value) | 接收一个int值 |
| LongPredicate | boolean test(long value) | 接收一个long值 |

# 方法引用与构造器引用
Lambda表达式是可以简化函数式接口的变量或形参赋值的语法。而方法引用和构造器引用是为了**简化Lambda表达式的**。
## 方法引用
当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！
方法引用可以看做是Lambda表达式深层次的表达。换句话说，方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是**Lambda表达式的一个语法糖**。
> 语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·约翰·兰达（Peter J. Landin）发明的一个术语，指计算机语言中添加的某种语法，这种语法`对语言的功能并没有影响，但是更方便程序员使用`。通常来说使用语法糖能够增加程序的可读性，从而减少程序代码出错的机会。

### 方法引用格式

-  格式：使用**方法引用操作符 **`**::**` 将类(或对象) 与 方法名分隔开来。 
-  如下三种主要使用情况： 
   - 情况1：`对象 :: 实例方法名`
   - 情况2：`类 :: 静态方法名`
   - 情况3：`类 :: 实例方法名`
### 方法引用使用
**使用前提：**Lambda体**只有一句语句**，并且是通过**调用一个对象的/类现有的方法**来完成的

---

**针对情况1：**函数式接口中的抽象方法a在被重写时使用了某一个对象的方法b。

- 如果方法a的形参列表、返回值类型与方法b的形参列表、返回值类型**都相同（或一致）**，则我们可以使用方法b实现对方法a的重写、替换。
- 注意：b是非静态方法，需要对象调用
```java
/**
 * 情况一：对象::实例方法
 */
// Consumer中的void accept(T t)
// PrintStream中的void println(T t)
@Test
public void test1() {
    Consumer<String> con1 = str -> System.out.println(str);
    con1.accept("北京");

    System.out.println("*******************");

//        Consumer<String> con2 = System.out::println;
    PrintStream ps = System.out;
    Consumer<String> con2 = ps::println;
    con2.accept("beijing");
}

// Supplier中的T get()
// Employee中的String getName()
@Test
public void test2() {
    Employee emp = new Employee(1001, "Tom", 23, 5600);

    Supplier<String> sup1 = () -> emp.getName();
    System.out.println(sup1.get());

    System.out.println("*******************");

    Supplier<String> sup2 = emp::getName;
    System.out.println(sup2.get());

}
```

---

**针对情况2：**函数式接口中的抽象方法a在被重写时使用了某一个类的静态方法b。

- 如果方法a的形参列表、返回值类型与方法b的形参列表、返回值类型**都相同（或一致）**，则我们可以使用方法b实现对方法a的重写、替换。
```java
/**
     * 情况二：类::静态方法
     */
    // Comparator中的int compare(T t1,T t2)
    // Integer中的int compare(T t1,T t2)
    @Test
    public void test3() {
        Comparator<Integer> com1 = (t1, t2) -> Integer.compare(t1, t2);
        System.out.println(com1.compare(12, 21));

        System.out.println("*******************");

        Comparator<Integer> com2 = Integer::compare;
        System.out.println(com2.compare(12, 3));
    }

    // Function中的R apply(T t)
    // Math中的Long round(Double d)
    @Test
    public void test4() {
        // 匿名内部类
        Function<Double, Long> func = new Function<Double, Long>() {
            @Override
            public Long apply(Double d) {
                return Math.round(d);
            }
        };
        System.out.println("*******************");

        // lambda
        Function<Double, Long> func1 = d -> Math.round(d);
        System.out.println(func1.apply(12.3));
        System.out.println("*******************");

        // 对象引用
        Function<Double, Long> func2 = Math::round;
        System.out.println(func2.apply(12.6));
    }
```

---

**针对情况3：**函数式接口中的抽象方法a在被重写时使用了某一个对象的方法b。

- 如果方法a的返回值类型与方法b的返回值类型相同，同时方法a的形参列表中有n个参数，方法b的形参列表有n-1个参数，且方法a的第1个参数作为方法b的调用者，且方法a的后n-1参数与方法b的n-1参数匹配（类型相同或满足多态场景也可以）
> 例如：`t->System.out.println(t)`、`() -> Math.random()`都是无参

```java
/**
 * 情况三：类 :: 实例方法  (有难度)
 */
// Comparator中的int comapre(T t1,T t2)
// String中的int t1.compareTo(t2)
@Test
public void test5() {
    // lambda
    Comparator<String> com1 = (s1, s2) -> s1.compareTo(s2);
    System.out.println(com1.compare("abc", "abd"));
    System.out.println("*******************");

    // 对象引用
    Comparator<String> com2 = String::compareTo;
    System.out.println(com2.compare("abd", "abm"));
}

// BiPredicate中的boolean test(T t1, T t2);
// String中的boolean t1.equals(t2)
@Test
public void test6() {
    // lambda
    BiPredicate<String, String> pre1 = (s1, s2) -> s1.equals(s2);
    System.out.println(pre1.test("abc", "abc"));
    System.out.println("*******************");

    // 对象引用
    BiPredicate<String, String> pre2 = String::equals;
    System.out.println(pre2.test("abc", "abd"));
}

// Function中的R apply(T t)
// Employee中的String getName();
@Test
public void test7() {
    Employee employee = new Employee(1001, "Jerry", 23, 6000);
    Function<Employee, String> func1 = e -> e.getName();
    System.out.println(func1.apply(employee));

    System.out.println("*******************");
    Function<Employee, String> func2 = Employee::getName;
    System.out.println(func2.apply(employee));
}
```
## 构造器引用
当Lambda表达式是创建一个对象，并且满足Lambda表达式形参，正好是给创建这个对象的构造器的实参列表，就可以使用构造器引用。
格式：`类名::new`
```java
// Supplier：T get()
// Employee的空参构造器：Employee()
@Test
public void test1() {
    // 匿名
    Supplier<Employee> sup = new Supplier<Employee>() {
        @Override
        public Employee get() {
            return new Employee();
        }
    };

    // lambda
    Supplier<Employee> sup1 = () -> new Employee();
    System.out.println(sup1.get());

    // 构造器引用
    Supplier<Employee> sup2 = Employee::new;
    System.out.println(sup2.get());
}

// Function：R apply(T t)
@Test
public void test2() {
    // 匿名
    Function<Integer, Employee> func = new Function<Integer, Employee>() {
        @Override
        public Employee apply(Integer id) {
            return new Employee(id);
        }
    };
    System.out.println(func.apply(1000));

    // lambda
    Function<Integer, Employee> func1 = id -> new Employee(id);
    Employee employee = func1.apply(1001);
    System.out.println(employee);

    // 构造器引用
    Function<Integer, Employee> func2 = Employee::new;
    Employee employee1 = func2.apply(1002);
    System.out.println(employee1);
}

// BiFunction中的R apply(T t,U u)
@Test
public void test3() {
    BiFunction<Integer, String, Employee> func = new BiFunction<Integer, String, Employee>() {
        @Override
        public Employee apply(Integer id, String name) {
            return new Employee(id, name);
        }
    };

    BiFunction<Integer, String, Employee> func1 = (id, name) -> new Employee(id, name);
    System.out.println(func1.apply(1001, "Tom"));

    BiFunction<Integer, String, Employee> func2 = Employee::new;
    System.out.println(func2.apply(1002, "Tom"));
}
```
## 数组构造引用
当Lambda表达式是创建一个数组对象，并且满足Lambda表达式形参，正好是给创建这个数组对象的长度，就可以数组构造引用。
格式：`数组类型名::new`
```java
@Test
public void test4() {
    // 匿名
    Function<Integer, String[]> func = new Function<Integer, String[]>() {
        @Override
        public String[] apply(Integer length) {
            return new String[length];
        }
    };
    System.out.println(Arrays.toString(func.apply(5)));

    // lambda
    Function<Integer, String[]> func1 = length -> new String[length];
    String[] arr1 = func1.apply(5);
    System.out.println(Arrays.toString(arr1));

    // 数组构造引用
    Function<Integer, String[]> func2 = String[]::new;
    String[] arr2 = func2.apply(10);
    System.out.println(Arrays.toString(arr2));
}
```
# 强大的Stream API
## 介绍
### 说明

- Java8中有两大最为重要的改变。第一个是 Lambda 表达式；另外一个则是 Stream API。
- `**Stream API ( java.util.stream) **`把真正的函数式编程风格引入到Java中。这是目前为止对Java类库`最好的补充`，因为Stream API可以极大提供Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。
- Stream 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。 **使用Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。**也可以使用 Stream API 来并行执行操作。简言之，Stream API 提供了一种高效且易于使用的处理数据的方式。
### 为什么要使用Stream API
实际开发中，项目中多数数据源都来自于MySQL、Oracle等。但现在数据源可以更多了，有MongDB，Radis等，而这些NoSQL的数据就需要Java层面去处理。
### 什么是Stream
Stream 是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。
Stream 和 Collection 集合的区别：**Collection 是一种静态的内存数据结构，讲的是数据，而 Stream 是有关计算的，讲的是计算。**前者是主要面向内存，存储在内存中，后者主要是面向 CPU，通过 CPU 实现计算。
注意：

1. Stream 自己不会存储元素。
2. Stream 不会改变源对象。相反，他们会返回一个持有结果的新Stream。
3. Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。即一旦执行终止操作，就执行中间操作链，并产生结果。
4. Stream一旦执行了终止操作，就不能再调用其它中间操作或终止操作了。
## Stream的操作三个步骤

**1- 创建 Stream**
一个数据源（如：集合、数组），获取一个流

**2- 中间操作**
每次处理都会返回一个持有结果的新Stream，即中间操作的方法返回值仍然是Stream类型的对象。因此中间操作可以是个`操作链`，可对数据源的数据进行n次处理，但是**在终结操作前，并不会真正执行**。

**3- 终止操作(终端操作)**

终止操作的方法返回值类型就不再是Stream了，因此一旦执行终止操作，就结束整个Stream操作了。**一旦执行终止操作，就执行中间操作链，最终产生结果并结束Stream**。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1692706054785-6a255e59-bfcd-487a-a4d1-a245c06c96fc.png#averageHue=%23f1f1f1&clientId=ue5fb61ee-2ed7-4&from=paste&height=190&id=u21100106&originHeight=375&originWidth=1333&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=32110&status=done&style=none&taskId=ufe0b0d87-e6c1-4e64-befb-a71459aef50&title=&width=677.0794060770311)

### 创建Stream实例

#### 通过集合

Java8 中 **Collection 接口** 被扩展，提供了两个获取流的方法：

-  `default Stream stream()` : 返回一个顺序流 
-  `default Stream parallelStream()` : 返回一个并行流 

```java
@Test
public void test01() {
    List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

    // default Stream<E> stream()：返回一个顺序流
    Stream<Integer> stream = list.stream();
    // default Stream<E> parallelStream()：返回一个并行流
    Stream<Integer> stream1 = list.parallelStream();
}
```

#### 通过数组

Java8 中 **Arrays** 的 静态方法 `stream()` 可以获取数组流：

- `public static Stream stream(T[] array)`: 返回一个流
- `public static IntStream stream(int[] array)`
- `public static LongStream stream(long[] array)`
- `public static DoubleStream stream(double[] array)`

```java
@Test
public void test02() {
    String[] arr = {"hello", "world"};
    Stream<String> stream = Arrays.stream(arr);

    int[] arr1 = {1, 2, 3, 4, 5};
    IntStream stream1 = Arrays.stream(arr1);
}
```

---

**方式三：通过Stream的of()**
可以调用 **Stream类 **静态方法 `of()`, 通过显示值创建一个流。它可以接收任意数量的参数。

- `public static Stream of(T... values)` : 返回一个流
```java
@Test
public void test04() {
    Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
    Stream<String> stream1 = Stream.of("AA", "BB", "CC");
}
```

---

**方式四：创建无限流(了解)**
可以使用静态方法 `Stream.iterate()` 和 `Stream.generate()`, 创建无限流。

-  迭代 `public static Stream iterate(final T seed, final UnaryOperator f) `
-  生成 `public static Stream generate(Supplier s) `
```java
// 方式四：创建无限流
@Test
public void test05() {
	// 迭代
	// public static<T> Stream<T> iterate(final T seed, final
	// UnaryOperator<T> f)
	Stream<Integer> stream = Stream.iterate(0, x -> x + 2);
	stream.limit(10).forEach(System.out::println);

	// 生成
	// public static<T> Stream<T> generate(Supplier<T> s)
	Stream<Double> stream1 = Stream.generate(Math::random);
	stream1.limit(10).forEach(System.out::println);
}
```

### 一系列中间操作

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性全部处理，称为“惰性求值”。

#### 筛选与切片

| **方法** | **描述** |
| --- | --- |
| `filter(Predicatep)` | 接收  Lambda ， 从流中排除某些元素 |
| `distinct()` | 筛选，通过流所生成元素的  hashCode() 和 equals() 去除重复元素 |
| `limit(long maxSize)` | 截断流，使其元素不超过给定数量 |
| `skip(long n)` | 跳过元素，返回一个扔掉了前  n 个元素的流。

若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补 |

```java
@Test
public void test01() {
    List<Employee> list = EmployeeData.getEmployees();

    // filter —— 从流中排除某些元素
    // 薪资大于7000
    System.out.println("********** filter **********");
    list.stream()
            .filter(emp -> emp.getSalary() > 7000)
            .forEach(System.out::println);

    // limit(n) —— 截断流，元素不超过给定数量
    System.out.println("********** limit **********");
    list.stream()
            .limit(2)
            .forEach(System.out::println);

    // skip(n) —— 跳过前n个元素
    System.out.println("********** skip **********");
    list.stream()
            .skip(5)
            .forEach(System.out::println);

    // distinct() —— 去重，通过hashCode()和equals()去除重复元素
    list.add(new Employee(1009, "马斯克", 40, 12500.32));
    list.add(new Employee(1009, "马斯克", 40, 12500.32));
    System.out.println("********** distinct **********");
    list.stream()
            .distinct()
            .forEach(System.out::println);
}
```

#### 映射

| **方法** | **描述** |
| --- | --- |
| `map(Function f)` | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。 |
| `mapToDouble(ToDoubleFunction f)` | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的 DoubleStream。 |
| `mapToInt(ToIntFunction f)` | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的  IntStream。 |
| `mapToLong(ToLongFunction f)` | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的  LongStream。 |
| `flatMap(Function f)` | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |

```java
@Test
public void test02() {
    List<Employee> list = EmployeeData.getEmployees();

    // 姓名长度大于3的员工的姓名
    list.stream()
            .filter(emp -> emp.getName().length() > 3)
//                .map(Employee::getName)
            .map(emp -> emp.getName())
            .forEach(System.out::println);
}
```

#### 排序

| **方法** | **描述** |
| --- | --- |
| `sorted()` | 产生一个新流，其中按自然顺序排序 |
| `sorted(Comparatorcom)` | 产生一个新流，其中按比较器顺序排序 |

```java
@Test
public void test03() {
    List<Employee> list = EmployeeData.getEmployees();
    Integer[] arr = new Integer[]{1, 2, 3, 4, 5, 6, 2, 2, 3, 3, 4, 4, 5};
    String[] arr2 = {"hello", "world", "java"};

    // ********** sorted() —— 自然排序 **********
    //Arrays.stream(arr).sorted().forEach(System.out::println);
    //Arrays.stream(arr2).sorted().forEach(System.out::println);
    
    // 因为Employee没有实现Comparable接口，所以报错！
    //list.stream().sorted().forEach(System.out::println);

    // ********** sorted(Comparator com) —— 定制排序 **********
    System.out.println("********** 年龄排序 **********");
    list.stream()
            .sorted((e1, e2) -> e1.getAge() - e2.getAge())
            .forEach(System.out::println);

    System.out.println("********** 针对于字符串从大到小排序 **********");
    Arrays.stream(arr2)
            .sorted((s1, s2) -> -s1.compareTo(s2))
            .forEach(System.out::println);

    System.out.println("********** 针对于字符串从大到小排序 **********");
    Arrays.stream(arr2)
            .sorted(String::compareTo)
            .forEach(System.out::println);
}
```

### 终止操作

-  终止操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void
-  流进行了终止操作后，不能再次使用

#### 匹配与查找

| **方法** | **描述** |
| --- | --- |
| `allMatch(Predicate p)` | 检查是否匹配所有元素 |
| `anyMatch(Predicate p)  ` | 检查是否至少匹配一个元素 |
| `noneMatch(Predicatep)` | 检查是否没有匹配所有元素 |
| `findFirst()` | 返回第一个元素 |
| `findAny()` | 返回当前流中的任意元素 |
| `count()` | 返回流中元素总数 |
| `max(Comparator c)` | 返回流中最大值 |
| `min(Comparator c)` | 返回流中最小值 |
| `forEach(Consumer c)` | 内部迭代(使用  Collection  接口需要用户去做迭代，称为外部迭代。

相反，Stream  API 使用内部迭代——它帮你把迭代做了

```java
@Test
public void test01() {
    List<Employee> list = EmployeeData.getEmployees();
    // 是否所有员工年龄都>18
    System.out.println("********** allMatch **********");
    System.out.println(list.stream()
            .allMatch(emp -> emp.getAge() > 18)
    );

    // 是否存在年龄>18的员工
    System.out.println("********** anyMatch **********");
    System.out.println(list.stream()
            .anyMatch(emp -> emp.getAge() > 18)
    );

    // 返回第一个元素
    System.out.println("********** findFirst **********");
    System.out.println(list.stream()
            .findFirst()
            .get()
    );

    // 返回元素总个数
    System.out.println("********** count **********");
    System.out.println(list.stream().count());

    // 返回最高工资
    // 方式1
    System.out.println("********** max1 **********");
    System.out.println(list.stream()
            .max((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
            .get()
            .getSalary()
    );
    // 方式2
    System.out.println("********** max2 **********");
    System.out.println(list.stream()
            .map(emp -> emp.getSalary())
            .max(Double::compare)
            .get()
    );

    // 内部迭代
    System.out.println("********** foreach **********");
    list.stream().forEach(System.out::println);

    // 针对于集合，jdk8中增加了一个遍历的方法
    System.out.println("********** list.forEach **********");
    list.forEach(System.out::println);
    // 针对于List来说，遍历的方式: 1 使用Iterator 2 增for 3 一for 4 forEach()
}
```

#### 归约

| **方法** | **描述** |
| --- | --- |
| `reduce(T identity, BinaryOperator b)` | 可以将流中元素反复结合起来，得到一个值。返回  T |
| `reduce(BinaryOperator b)` | 可以将流中元素反复结合起来，得到一个值。返回 Optional |

备注：map 和 reduce 的连接通常称为 map-reduce 模式，因 Google 用它来进行网络搜索而出名。

```java
 @Test
    public void test02() {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        List<Employee> list1 = EmployeeData.getEmployees();

        // reduce(T identity, BinaryOperator b)
        // 练习1：计算1-10的自然数
        System.out.println(list.stream()
                .reduce(0, (x1, x2) -> x1 + x2)
        ); //55
        System.out.println(list.stream()
                .reduce(10, (x1, x2) -> x1 + x2)
        ); //65
        System.out.println(list.stream()
                .reduce(0, (x1, x2) -> Integer.sum(x1, x2))
        ); //55
        System.out.println(list.stream()
                .reduce(0, Integer::sum)
        ); //55

        // 练习2：计算所有员工工资总和
        System.out.println(list1.stream()
                .map(emp -> emp.getSalary())
                .reduce(Double::sum)
                .get()
        );
    }
```

#### 收集

| **方   法**              | **描   述**          |
| ---------------------- | ------------------ |
| `collect(Collector c)` | **将流转换为其他形式（集合）**。 |
接收一个Collector接口的实现，
用于给Stream中元素做汇总的方法 |

Collector 接口中方法的实现决定了如何对流执行收集的操作(如收集到 List、Set、Map)。
另外， Collectors 实用类提供了很多静态方法，可以方便地创建常见收集器实例，具体方法与实例如下表：

| **方法**                | **返回类型**                                | **作用**                                            |
| --------------------- | --------------------------------------- | ------------------------------------------------- |
| **toList**            | `Collector<T, ?, List>`                 | 把流中元素收集到List                                      |
| **toSet**             | `Collector<T, ?, Set>`                  | 把流中元素收集到Set                                       |
| **toCollection**      | `Collector<T, ?, C>`                    | 把流中元素收集到创建的集合                                     |
| **counting**          | `Collector<T, ?, Long>`                 | 计算流中元素的个数                                         |
| **summingInt**        | `Collector<T, ?, Integer>`              | 对流中元素的整数属性求和                                      |
| **averagingInt**      | `Collector<T, ?, Double>`               | 计算流中元素Integer属性的平均值                               |
| **summarizingInt**    | `Collector<T, ?, IntSummaryStatistics>` | 收集流中Integer属性的统计值。如：平均值                           |
| **joining**           | `Collector<CharSequence, ?, String>`    | 连接流中每个字符串                                         |
| **maxBy**             | `Collector<T, ?, Optional>`             | 根据比较器选择最大值                                        |
| **minBy**             | `Collector<T, ?, Optional>`             | 根据比较器选择最小值                                        |
| **reducing**          | `Collector<T, ?, Optional>`             | 从一个作为累加器的初始值开始，利用BinaryOperator与流中元素逐个结合，从而归约成单个值 |
| **collectingAndThen** | `Collector<T,A,RR>`                     | 包裹另一个收集器，对其结果转换函数                                 |
| **groupingBy**        | `Collector<T, ?, Map<K, List>>`         | 根据某属性值对流分组，属性为K，结果为V                              |

```java
@Test
public void test03() {
    List<Employee> list = EmployeeData.getEmployees();
    // collect(Collector c)

    // 练习1: 查找工资大于6000的员工，结果返回为一个List或Set
    List<Employee> list1 = list.stream()
            .filter(employee -> employee.getSalary() > 6000)
            .collect(Collectors.toList());
    list.forEach(System.out::println);

    System.out.println();

    // 练习2: 按照员工的年龄进行排序，返回到一个新的List中
    List<Employee> list2 = list.stream()
            .sorted((e1, e2) -> e1.getAge() - e2.getAge())
            .collect(Collectors.toList());
    list2.forEach(System.out::println);
}
```

