---
title: 深入浅出设计模式系列--单例模式
tags: 设计模式
categories: 设计模式
keywords: 设计模式 Java
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

# 前言
**深入浅出设计模式系列，尽量采用通俗易懂、循序渐进的方式，让大家真正理解设计模式的精髓！**

先了解一下单例模式定义：确保一个类只有一个自行实例化的实例，并提供一个全局访问点，向整个系统提供这个实例。

单例模式的本质：控制实例的数目。

# 单例模式的主要问题

我们经常使用的单例模式主要涉及的问题有：

- 如何保证一个类只有一个实例？
- 如何保证单例模式在多线程环境下的线程安全？
- 如何防止反射对单例模式的破坏？
- 如何保证单例模式在序列化与反序列化过程中实例的唯一性？

下面针对以上的问题进行深入研究！

# 单例模式的实现

单例模式常见的有饿汉式、懒汉式、双重检验锁、静态内部类方式、枚举方式，这里主要讨论多线程环境下的单例模式。

## 饿汉式
这种饿汉式单例是比较常见的，我们看看它是怎么实现的！

```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 饿汉式单例模式
 */
public class Singleton {

    private static Singleton instance = new Singleton();

    private Singleton (){}

    public static Singleton getInstance() {
        return instance;
    }
}
```

饿汉式单例本身是线程安全的，但它采用空间换取时间的方式，当类加载时马上就实例化Singleton对象，不管使用者用不用，后续每次调用 **getInstance()** 方法的时候，就不需要判断它是否实例化，从而节约了时间。但有些情况下需要懒加载实例化对象，针对这种情形，于是有了懒汉式的单例模式。


## 懒汉式

我们看一下懒汉式单例模式的实现。

```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 懒汉式单例模式
 */
public class Singleton {

    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

```

这就是我们常见的懒汉式单例模式！大家发现没？这种懒汉式单例模式在多线程环境下，存在线程安全的问题！那它是怎么引发多线程安全的问题的呢？

为便于大家理解单例模式在多线程环境下容易出现的问题，下面直接采用图片的方式，请看图片：
![pattern-singleton-thread.png](/uploads/pattern-singleton-thread.png)

从上图可知多线程环境下懒汉式单例模式的问题了吧！

针对上面单例模式存在多线程安全的缺陷，有人说这还不容易解决，直接在 **getInstance** 方法上加上Java关键词 **synchronized**， 即。
```
    public synchronized static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
```

可是你可别忘了，在Java中每个语句都是执行都需要时间的，加上 **synchronized** 关键词需要底层执行更多的语句，并且每次都需要通过 **getInstance()** 方法获取实例，效率非常底，更别说在多线程高并发下的执行情况了，因此必须对此加以改进。

## 双重检验锁

针对上面懒汉式单例的性能问题，我通过修改得到双重检验锁单例。
```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 双重检验锁单例模式
 */
public class Singleton {

    private static Singleton instance;

    private Singleton (){}

    public static Singleton getInstance() {
        if (instance == null) { // 双重检测机制
            synchronized (Singleton.class) { // 同步锁
                if (instance == null) { // 双重检测机制
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
上面双重检验锁单例的写法，是不是比懒汉式单例效率高，因为每次需要通过 **getInstance** 方法获取实例时，只在第一次实例化 **instance** 加同步锁，后续多线程进入该方法后，直接进入外层 **if (instance == null)** 判断语句，得知instance实例不为空，就直接返回instance实例了。

虽然上面的双重校验锁机制的单例增加了一定的安全性，提高了效率，但是这个双重检验锁单例也有缺陷，因为它没有考虑到 JVM 编译器的指令重排。

**1、什么是指令重排**
比如 java 中简单的一句 instance = new Singleton()，会被编译器编译成如下 JVM 指令。
```
memory = allocate();    //1：分配对象的内存空间 
ctorInstance(memory);  //2：初始化对象 
instance = memory;     //3：设置instance指向刚分配的内存地址
```
但是这些指令顺序并非一成不变，有可能会经过 JVM 和 CPU 的优化，指令重排成下面的顺序。
```
memory = allocate();    //1：分配对象的内存空间 
instance = memory;      //3：设置instance指向刚分配的内存地址 
ctorInstance(memory);  //2：初始化对象
```

**2、影响**

高并发情况下，线程 A 执行 **instance = new Singleton();** 完上面的1、3步骤时，准备走2，即 instance 对象还未完成初始化，但此时 **instance** 已经不再指向 null 。

此时线程 B 抢占到CPU资源，执行 if(instance == null) 时，返回的结果会是 false，

从而返回一个没有初始化完成的instance对象。

**3、解决方法**
如何去解决这个问题呢？很简单，可以利用关键字 volatile 来修饰 instance 对象，如下优化：

**双重检验锁改进**

```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 单例模式使用双重检验锁方式实现，优点：延迟初始化、性能优化、线程安全
 */
public class Singleton {

    private volatile static Singleton instance;

    private Singleton (){}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

volatile 修饰符在此处的作用就是阻止变量访问前后的指令重排，从而保证了指令的执行顺序。

即指令的执行顺序是严格按照上文的 1、2、3 步骤来执行，从而保证对象不会出现中间态。

但是上面的双重检验锁改进版单例模式也有问题，因为它无法解决反射对单例模式的破坏性。我将在静态内部类单例模式中加以阐述。

## 静态内部类

在了解静态内部类单例模式之前，让我们先了解一下静态内部类的两个知识。

- 静态内部类加载一个类时，其内部类不会同时被加载。
- 一个类被加载，当且仅当其某个静态成员如静态域、构造器、静态方法等被调用时才会被加载。 

我们先看一个静态内部类的测试，以验证上面这两个观点。
```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 静态内部类测试类
 */
public class OuterClassTest {

    private OuterClassTest() {}

    static {
        System.out.println("1、我是外部类静态模块...");
    }

    // 静态内部类
    private static final class StaticInnerTest {
        private static OuterClassTest oct = new OuterClassTest();

        static {
            System.out.println("2、我是静态内部类的静态模块... " + oct);
        }

        static void staticInnerMethod() {
            System.out.println("3、静态内部类方法模块... " + oct);
        }
    }

    public static void main(String[] args) {
        OuterClassTest oct = new OuterClassTest(); // 此刻内部类不会被加载
        System.out.println("===========分割线===========");
        OuterClassTest.StaticInnerTest.staticInnerMethod(); // 调用内部类的静态方法
    }
}
```

输出如下。
```
1、我是外部类静态模块...
=========分割线=========
2、我是静态内部类的静态模块... com.chloneda.jutils.test.OuterClassTest@b1bc7ed
3、静态内部类的方法模块... com.chloneda.jutils.test.OuterClassTest@b1bc7ed
```

从运行结果来看，验证是正确的！

由于静态内部类的特性，只有在其被第一次引用的时候才会被加载，所以可以保证其线程安全性。

由此可得出静态内部类单例模式的写法。

**静态内部类**
```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description:
 *  单例模式使用静态内部类方式实现，优点：实现代码简洁、延迟初始化、线程安全
 */
public class Singleton {

    private static final class SingletonHolder {
        private static Singleton INSTANCE = new Singleton();
    }

    private Singleton(){}

    public static Singleton getInstance(){
        return SingletonHolder.INSTANCE;
    }
}
```

这种写法的单例，外部无法访问静态内部类 SingletonHolder，只有当调用 Singleton.getInstance() 方法的时候，才能得到单例对象 INSTANCE。

而且静态内部类单例的 **getInstance()** 方法中没有使用 **synchronized** 关键字，提高了执行效率，同时兼顾了懒汉模式的内存优化（使用时才初始化，节约空间，达到懒加载的目的）以及饿汉模式的安全性。

但这种单例也有问题！这种方式需要两个类去做到这一点，也就是说，虽然懒加载静态内部类的对象，但其 外部类及内部静态类的 Class 对象还是会被创建，同时也无法防止反射对单例的破坏性（**很多单例的写法都有这个通病**），从而无法保证对象的唯一性。

我们通过以下测试类测试反射对静态内部类的破坏性。

```
/**
 * @Created by chloneda
 * @Description: 反射破坏静态内部类单例模式的测试类
 */
public class SingletonReflectTest {
    public static void main(String[] args) {
        //创建第一个实例
        Singleton instance1 = Singleton.getInstance();

        //通过反射创建第二个实例
        Singleton instance2 = null;
        try {
            Class<Singleton> clazz = Singleton.class;
            Constructor<Singleton> cons = clazz.getDeclaredConstructor();
            cons.setAccessible(true);
            instance2 = cons.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }

        //检查两个实例的hash值
        System.out.println("Instance1 hashCode: " + instance1.hashCode());
        System.out.println("Instance2 hashCode: " + instance2.hashCode());
    }
}
```

输出结果如下。
```
Instance1 hashCode: 186370029
Instance2 hashCode: 2094548358
```

从输出结果可以看出，通过反射获取构造函数，并调用 **setAccessible(true)** 就可以调用私有的构造函数，从而得到Instance1和Instance2两个不同的对象。

**静态内部类改进**

如何防止这种反射对单例的破坏呢？我们可以通过修改构造器，让它在被要求创建第二个实例的时候抛出异常。

静态内部类修改如下。
```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 防止反射破坏静态内部类单例模式
 */
public class Singleton {

    private static boolean initialized = false;

    private static final class SingletonHolder {
        private static Singleton INSTANCE = new Singleton();
    }

    private Singleton() {
        synchronized (Singleton.class) {
            if (initialized == false) {
                initialized = !initialized;
            } else {
                throw new RuntimeException("单例模式禁止二次创建，防止反射！");
            }
        }
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
    
}
```

我们还用一个 **SingletonReflectTest** 测试类测试一下，输出结果如下。
```
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.chloneda.jutils.test.SingletonReflectTest.main(Singleton.java:46)
Caused by: java.lang.RuntimeException: 单例模式禁止二次创建，防止反射！
	at com.chloneda.jutils.test.Singleton.<init>(Singleton.java:24)
	... 5 more
Instance1 hashCode: 1053782781
Exception in thread "main" java.lang.NullPointerException
	at com.chloneda.jutils.test.SingletonReflectTest.main(Singleton.java:53)
```

所以我们通过修改构造器防止反射对单例的破坏性。

但是这种方式的单例也存在问题！什么问题呢？即序列化和反序列化之后无法继续保持单例（**很多单例的写法也有这个通病**）。

我们让上面防止反射破坏静态内部类的单例实现 **Serializable** 接口。
```
public class Singleton implements Serializable 
```
并通过以下测试类进行序列化和反序列化测试。
```
/**
 * @Created by chloneda
 * @Description: 序列化破坏静态内部类单例模式的测试类
 */
pubic class SingletonSerializableTest {
    public static void main(String[] args) {
        try {
            Singleton instance1 = Singleton.getInstance();
            ObjectOutput out = null;

            out = new ObjectOutputStream(new FileOutputStream("Singleton.ser"));
            out.writeObject(instance1);
            out.close();

            //从文件中反序列化一个Singleton对象
            ObjectInput in = new ObjectInputStream(new FileInputStream("Singleton.ser"));
            Singleton instance2 = (Singleton) in.readObject();
            in.close();

            System.out.println("instance1 hashCode: " + instance1.hashCode());
            System.out.println("instance2 hashCode: " + instance2.hashCode());

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

输出结果如下。
```
instance1 hashCode: 240650537
instance2 hashCode: 1566502717
```
从结果可以看出，很明显不是同一个单例对象！

那如何解决这个问题呢？

**静态内部类再改进**

我们可以实现 readResolve() 方法，它代替了从流中读取对象，确保了在序列化和反序列化的过程中没人可以创建新的实例。

可以得到改进版的静态内部类单例，可以有效防止序列化及反射的破坏！
```
package com.chloneda.jutils.test;

import java.io.*;

/**
 * @Created by chloneda
 * @Description: 可以防止序列化及反射破坏的静态内部类单例模式
 */
public class Singleton implements Serializable {

    private static boolean initialized = false;

    private static final class SingletonHolder {
        private static Singleton INSTANCE = new Singleton();
    }

    private Singleton() {
        synchronized (Singleton.class) {
            if (initialized == false) {
                initialized = !initialized;
            } else {
                throw new RuntimeException("单例模式禁止二次创建，防止反射！");
            }
        }
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private Object readResolve() {
        return getInstance();
    }
}
```

我们再用上面的 **SingletonSerializableTest** 测试类测试一下结果。

输出结果如下。
```
instance1 hashCode: 240650537
instance2 hashCode: 240650537
```
此时就说明，单例在序列化和反序列化时的对象是一致的了。

其实上面饿汉式、懒汉式、双重校验锁及静态内部类单例所出现的问题，都可以通过枚举型单例进行解决，这也是《Effective Java》中推荐的写法。

## 枚举型


我们知道java中的枚举类是在JDK5中才有的，因此枚举型单例对大部分人来说是比较陌生的，先举一个例子吧！

枚举型单例实现很简单。
```
public enum Singleton {
    INSTANCE;
}
```
就三行，很简单吧。那么为了大家更方便理解，我再用以下例子来剖析一下枚举型单例吧！

```
package com.chloneda.jutils.test;

/**
 * @Created by chloneda
 * @Description: 单例类
 */
public class Singleton{}

/**
 * 枚举型单例
 */
public enum SingletonEnum {
    INSTANCE;
    private Singleton singleton = null;

    private SingletonEnum() {
        singleton = new Singleton();
    }

    public Singleton getInstance() {
        return singleton;
    }
}
```
上面这个枚举型单例，通过反编译你就会得到以下代码。
```
public final class SingletonEnum extends java.lang.Enum<SingletonEnum> {
    public static final SingletonEnum INSTANCE;
    public static SingletonEnum[] values();
    public static SingletonEnum valueOf(String);
    public Singleton getSingleton();
    static {};
}
```
上面代码经过处理（只是去掉包名，便于阅读）！

可以发现枚举型单例相关属性都被 **static** 关键词修饰，仔细一看还是 **final** 修饰的一个普通类，只不过继承自 **java.lang.Enum** 枚举类而已。也就是说这个枚举型单例是经过编译器处理的，这是JDK5提供的语法糖（Syntactic Sugar），所谓语法糖是指在计算机语言中添加的某种语法，这种语法对语言的功能并没有影响，只是在编译器上做了手脚，可以更方便我们程序员使用。

枚举型单例天生就是线程安全的，这是因为 **JVM类加载机制** 的缘故，这里就不展开论述了，可以参考一下相关资料！

### 枚举型单例反射问题
我们先了解一下枚举型单例为什么可以防止反射的问题？

其实在JDK5中，Java虚拟机对枚举类做了特殊处理，即 **JVM** 会阻止反射获取枚举类的私有构造方法。

我们用这个枚举型单例作为例子。
```
public enum Singleton {
    INSTANCE;

    Singleton() {
    }
}
```

继续使用上文的反射代码 **SingletonReflectTest** 来进行测试，先把SingletonReflectTest类的创建第一个实例的语句改成这样。

```
    //创建第一个实例
    Singleton instance1 = Singleton.INSTANCE;
```

运行后输出结果如下。
```
java.lang.NoSuchMethodException: com.chloneda.jutils.test.Singleton.<init>()
	at java.lang.Class.getConstructor0(Class.java:3082)
	at java.lang.Class.getDeclaredConstructor(Class.java:2178)
	at com.chloneda.jutils.test.SingletonReflectTest.main(SingletonReflectTest.java:18)
Instance1 hashCode: 1929600551
Exception in thread "main" java.lang.NullPointerException
	at com.chloneda.jutils.test.SingletonReflectTest.main(SingletonReflectTest.java:27)
```
直接报异常，也就是说枚举型单例可以完美解决反射的问题。

### 枚举型单例反序列化问题
下面再深入了解一下为什么枚举会满足反序列化的问题

Java规范中规定，每一个枚举类型极其定义的枚举变量在JVM中都是唯一的，因此在枚举类型的序列化和反序列化上，Java做了特殊的规定。

在序列化的时候Java仅仅是将枚举对象的name属性输出到结果中，反序列化的时候则是通过 java.lang.Enum 的 valueOf() 方法来根据名字查找枚举对象。

以上面的**public enum SingletonEnum** 枚举为例，序列化的时候只将 INSTANCE 这个名称输出，反序列化的时候再通过这个名称，查找对于的枚举类型，因此反序列化后的实例也会和之前被序列化的对象实例相同。

我们通过测试类测试一下，我们首先让枚举型单例 **SingletonEnum** 实现 **Serializable** 接口。
```
public enum SingletonEnum implements Serializable
```

再使用序列化测试类 **SingletonSerializableTest** 进行测试，输出结果如下。

```
instance1 hashCode: 1674896058
instance2 hashCode: 1674896058
```
也就是说枚举型单例可以解决反序列化带来的问题。

综上所述，枚举型单例可以有效解决线程安全、反射、反序列化带来的问题！

然而你别得意，枚举型单例也有问题，就是它无法进行懒加载，因为枚举型单例的实例对象是在**静态代码块**即static块进行初始化的，是不是一语惊醒梦中人啊！

## 中场休息
说了这么多单例模式，知道了各个单例模式利弊，所以当我们使用时，我们要根据实际情况做出取舍，因为我们不可能实现一个单例可以满足所有情况。

下面让我们来看看单例模式的实际应用场景吧！

# 单例模式的实际应用

## 生活中的单例

我们计算机上有很多场景应用到单例模式，可能经常看到，但我们并没有认识到，比如以下例子。

- Windows的任务管理器（Task Manager）就是很典型的单例模式。你试过能打开两个windows的任务管理器吗？

- windows的回收站也是典型的单例应用。在整个系统运行过程中，回收站一直维护着仅有的一个实例。 

- 网站的计数器，一般也是采用单例模式实现，否则难以同步。 

- 应用程序的日志应用，一般都何用单例模式实现，这一般是由于共享的日志文件一直处于打开状态，因为只能有一个实例去操作，否则内容不好追加。
 
- Web应用的配置对象的读取，一般也应用单例模式，这个是由于配置文件是共享的资源。 

- 多线程的线程池的设计一般也是采用单例模式，这是由于线程池要方便对池中的线程进行控制。

当然还有很多单例模式的应用，希望大家可以发现哦！

## JDK中的单例

### java.lang.Runtime

Runtime类封装了Java运行时的环境。每一个java程序实际上都是启动了一个JVM进程，那么每个JVM进程都是对应这一个Runtime实例，此实例是由JVM为其实例化的。每个 Java 应用程序都有一个 Runtime 类实例，使应用程序能够与其运行的环境相连接。

由于Java是单进程的，所以，在一个JVM中，Runtime的实例应该只有一个，所以应该使用单例来实现。

我们来看看 **java.lang.Runtime** 的单例模式实现。
```
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }

    private Runtime() {}
}
```
看见没，这里使用的单例模式是饿汉式单例模式，也就是说，当类第一次被classloader加载的时候，实例就被创建出来了，当调用者每次调用的时候，就不需要再判断这个实例是否已经初始化，典型的空间换时间方案。

这里使用的是饿汉式单例模式，无疑是非常合适的！

### java.awt.Toolkit
Toolkit是GUI中的类，与RunTime不同的是Toolkit采用的是懒汉式单例模式，因为它们并不需要事先创建好，只要在第一次真正用到的时候再创建就可以了。

我们来看看Toolkit类的代码。
```
public abstract class Toolkit {

    private static Toolkit toolkit;
    
    public static synchronized Toolkit getDefaultToolkit() {
        if (toolkit == null) {
            java.security.AccessController.doPrivileged(
                    new java.security.PrivilegedAction<Void>() {
                public Void run() {
                    Class<?> cls = null;
                    String nm = System.getProperty("awt.toolkit");
                    try {
                        cls = Class.forName(nm);
                    } catch (ClassNotFoundException e) {
                        ClassLoader cl = ClassLoader.getSystemClassLoader();
                        if (cl != null) {
                            try {
                                cls = cl.loadClass(nm);
                            } catch (final ClassNotFoundException ignored) {
                                throw new AWTError("Toolkit not found: " + nm);
                            }
                        }
                    }
                    try {
                        if (cls != null) {
                            toolkit = (Toolkit)cls.newInstance();
                            if (GraphicsEnvironment.isHeadless()) {
                                toolkit = new HeadlessToolkit(toolkit);
                            }
                        }
                    } catch (final InstantiationException ignored) {
                        throw new AWTError("Could not instantiate Toolkit: " + nm);
                    } catch (final IllegalAccessException ignored) {
                        throw new AWTError("Could not access Toolkit: " + nm);
                    }
                    return null;
                }
            });
            loadAssistiveTechnologies();
        }
        return toolkit;
    }
    
}

```

观察上面的代码你会发现Toolkit是一个抽象类，本身就无法实例化，而是通过反射机制加载类并创建新的实例。

懒汉式单例模式，并不会第一时间创建实例，提高了JVM的启动速度，典型的时间换空间方案，同时也体现了延迟加载的思想。

此外，需要注意的是懒汉式单例模式的线程安全问题，关于网上也有很多版本，都各有优势，大家适当取舍吧！

## 框架中的单例

- Mybatis中的单例模式：如ErrorContext和LogFactory。
- Spring框架中的单例模式：采用单例注册表的方式进行实现中的单例模式，如AbstractBeanFactory抽象类的getBeans()方法。


# 小结
这篇文章有点长，不过总算把单例模式说清楚了，哎，我的脑细胞！