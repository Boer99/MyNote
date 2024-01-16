# 类和对象

## 内部类

一个类的内部又完整地嵌套了另一个类结构，是类的第五大成员（属性、方法、构造器、代码块、内部类）

最大的特点是可以==直接访问私有属性==，并且可以==体现类与类之间的包含关系==

> 内部类的分类

定义在外部类局部位置上（比如方法内）：
1. 局部内部类（有类名）
2. 匿名内部类（无类名，重点在这里）

定义在外部类的成员位置上：
1. 成员内部类（无static修饰）
2. 静态内部类（有static修饰）

### 局部内部类

定义在外部类的局部位置（方法/代码块）

可以直接访问外部类的所有成员

不能添加访问修饰符，因为它的地位就是一个局部变量（局部变量是不能用修饰符的），可以使用final修饰（局部变量可以使用final）

作用域：仅仅在定义它的方法或代码块中

外部类访问局部内部类的成员的访问方式：**创建对象再访问**（必须在作用域内）

外部类和内部类重名，遵循就近原则。访问外部类的成员：外部类名.this.成员名（四种内部类都是这样）

本质是一个类

```
public class Test {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.m1();
        System.out.println("outer的hashCode=" + outer); //同内部类输出的相同
    }
}

class Outer {
    private int n1 = 100;

    private void m2() {}

    public void m1() {
        final class Inner {
            private int n1 = 800;

            public void f1() {
              	//重名
                System.out.println("n1=" + n1); //800
                System.out.println(Outer.this.n1); //100
                //Outer.this表示当前对象
                System.out.println("Outer.this hashCode=" + Outer.this);
                m2();
            }
        }
        Inner inner = new Inner(); //作用域内
        inner.f1();
    }
}
```

### 匿名内部类

定义位置、访问成员、作用域、可添加修饰符 同局部内部类一样

特点：

- 需求：类只使用一次，用完就没有了
- 没有名字
- 本质是一个类，同时也是一个**对象**
- 相当于继承了外部类或者实现了接口

匿名内部类的理解：**将继承\实现，方法重写，创建对象，放在了一步进行**

```
public class Test {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.method();
        //tiger对象的运行类型是：class demo7.d6.Outer$1
        //匿名内部类重写了cry()
        //接收到name=jack
        //father对象的运行类型是：class demo7.d6.Outer$2
        //匿名内部类重写了run()
        //狗吃骨头
    }
}

class Outer {
  
    private int n1 = 10;

    public void method() {
        //基于接口的匿名内部类
        //tiger的编译类型是IA，运行类型是匿名内部类Outer$2
        //jdk底层创建匿名内部类，马上就创建了实例，并把地址传给tiger
        IA tiger = new IA() {
            @Override
            public void cry() {
                System.out.println("匿名内部类重写了cry()");
            }
        };
        System.out.println("tiger对象的运行类型是：" + tiger.getClass());
        tiger.cry();

        //基于普通类的匿名内部类
        Father father = new Father("jack") {
            @Override
            public void run() {
                System.out.println("匿名内部类重写了run()");
            }
        };
        System.out.println("father对象的运行类型是：" + father.getClass()); //Outer$1，没有大括号就是father
        father.run();

        //基于抽象类的匿名内部类
        Animal dog = new Animal() {
            @Override
            void eat() {
                System.out.println("狗吃骨头");
            }
        };
        dog.eat();
    }
  
}
```

最佳实践：当作实参直接传递，简洁高效

```
public class Test2 {
    public static void main(String[] args) {
        //直接当作参数传递
        f1(new IL() {
            @Override
            public void show() {
                System.out.println("这是一幅名画");
            }
        });
    }

    public static void f1(IL il) {
        il.show();
    }
}

interface IL {
    void show();
}
```

### 成员内部类

定义在外部类的成员位置

可以添加任意访问修饰符（地位等同于一个成员）

作用域：同外部类其他成员一样，为整个类体

外部类访问内部类：创建对象，再访问

外部其他类访问内部类的两种方式

```
public class Test {
    public static void main(String[] args) {
        Outer outer = new Outer();
        outer.t1();
        //外部其他类访问成员内部类
        //第一种 只是一种语法，不必纠结
        //Outer.Inner inner = outer.new Inner();
        //inner.say();
        new Outer().new Inner().say();
        //第二种 在外部类中编写一个方法，返回Inner对象
        outer.getInnerInstance().say();
    }
}

class Outer {
    private int n1 = 10;
    public String name = "张三";

    class Inner {
        public void say() {
            System.out.println("n1=" + n1 + " name=" + name);
        }
    }

    public Inner getInnerInstance() {
        return new Inner();
    }

    public void t1() {
        //使用成员内部类
        Inner inner = new Inner();
        inner.say();
    }
}
```

### 静态内部类

定义位置、访问修饰符、作用域

用static修饰

可直接访问外部类的所有静态成员，不能直接访问非静态成员

```
public class Test {
    public static void main(String[] args) {
        //外部其他类访问静态内部类
        //方式一 静态的，可以直接通过类名访问
        Outer.Inner inner = new Outer.Inner();
        inner.say();
        //方式二
        new Outer().getInnerInstance().say();
        //方式三 创建静态方法访问
        Outer.getInnerInstance_().say();
    }
}

class Outer {
    private static String name = "张三";

    static class Inner {
        private static String name = "张三";

        public void say() {
            //重名
            System.out.println("name=" + Outer.name);
            System.out.println("name=" + name);
        }
    }

    public Inner getInnerInstance() {
        return new Inner();
    }

    public static Inner getInnerInstance_() {
        return new Inner();
    }
}
```