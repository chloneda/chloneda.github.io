title: Java基础系列-代码块详解
type: "Java"
categories: Java
comments: false
date: 2019-11-24 18:31:55
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)


# 前言
**Java系列，尽量采用通俗易懂、循序渐进的方式，让大家真正理解Java基础知识！**

# 代码块分类

在Java中，使用{}括起来的代码被称为代码块（Code block），根据其位置和声明的不同，可以分为：
- 局部代码块。
- 构造代码块。
- 同步代码块。
- 静态代码块。


Java代码块的核心问题是：
- 代码块初始化是在什么时候？
- 代码块执行顺序是怎样的？
- 代码块在**继承**时，执行顺序是怎样的？

下面将对各种代码块就核心问题展开叙述！



# 局部代码块

在方法中出现，可以限定变量生命周期，及早释放，提高内存利用率。示例代码：
```
package com.chloneda.jutils.test;
/**
 * @author chloneda
 * @description: 局部代码块测试
 */
public class CodeBlockTest {
    public static void main(String[] args) {
        //局部代码块
        {
            int number = 100;
            System.out.println(number);
        }
        //找不到number变量，原因呢？？？
        //System. out.println(number);
    }
}
```

执行结果：
```
100
```


# 构造代码块

在类中方法外出现，每次调用构造方法都会执行，并且在构造方法前执行。示例代码：
```
package com.chloneda.jutils.test;

/**
 * @author chloneda
 * @description: 构造代码块类
 */
class CodeBlock {
    //构造代码块，在方法外出现
    {
        int number1 = 10;
        System.out.println("number1: " + number1);
    }

    //构造方法
    public CodeBlock() {
        System.out.println("这是构造方法");
    }

    //在构造代码块在构造方法前后出现，但构造代码块先于构造方法执行
    {
        int number2 = 100;
        System.out.println("number2: " + number2);
    }
}

//构造代码块测试类
public class CodeBlockTest {
    public static void main(String[] args) {
        // 创建对象
        CodeBlock codeBlock = new CodeBlock();
        // 注意：构造代码块通过构造方法自动调用
    }
}
```

执行结果：
```
number1: 10
number2: 100
这是构造方法
```

由上面的执行结果可知：
- 构造代码块依赖于构造方法，而且优先于构造函数执行，即实例对象建立，才会运行构造代码块，类不能调用构造代码块的。

**Tip**: 
**构造代码块与构造函数的区别**：构造代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，但无论建立哪个对象，都会先执行相同的构造代码块。

也就是说，构造代码块中定义的是不同对象共性的初始化内容。

# 同步代码块

同步代码块指的是被Java中Synchronized关键词修饰的代码块，在Java中，Synchronized关键词不仅仅可以用来修饰代码块，与此同时也可以用来修饰方法，是一种线程同步机制，被Synchronized关键词修饰的代码块会被加上内置锁。

需要说明的是Synchronized同步代码块是一种高开销的操作，因此我们应该尽量减少被同步的内容，在很多场景，我们没有必要去同步整个方法，而只需要同步部分代码即可，也就是使用同步代码块（JDK源码中有很多应用）。

同步代码块示例如下。
```
package com.chloneda.jutils.test;

/**
 * @author chloneda
 * @description: 同步代码块测试
 */
public class CodeBlock implements Runnable {
    @Override
    public void run() {
        synchronized (CodeBlock.class) {
            System.out.print("同步代码块!");
        }
    }

    public static void main(String[] args) {
        CodeBlock a = new CodeBlock();
        CodeBlock b = new CodeBlock();
        new Thread(a).start();
        new Thread(b).start();
    }
}

```

此外，静态代码是属于类而不是属于对象的，因此使用Synchronized来修饰静态方法和静态对象的时候，类下的所有对象都会被锁定。


# 静态代码块

在类中方法外出现，并加上static修饰，常用于给类进行初始化，在加载的时候就执行，并且静态代码块执行一次。

## 代码块执行顺序

首先我们验证一下代码块执行顺序，示例代码如下。
```
package com.chloneda.jutils.test;

/**
 * @author chloneda
 * @description: 验证代码块执行顺序
 */
class StatisCodeBlock {
    //静态代码块，在方法外出现
    static {
        int number1 = 20;
        System.out.println("1、静态代码块变量： " + number1);
    }

    //构造代码块，在方法外出现
    {
        int number2 = 100;
        System.out.println("2、构造代码块变量： " + number2);
    }

    public StatisCodeBlock() {
        System.out.println("这是构造方法 StatisCodeBlock()");
    }

    static {
        int number3 = 200;
        System.out.println("3、静态代码块变量： " + number3);
    }

    //在构造代码块在构造方法前后，但构造代码块先于构造方法执行
    {
        int number4 = 1000;
        System.out.println("4、构造代码块变量： " + number4);
    }
}

//静态代码块测试类
public class CodeBlockTest {
    public static void main(String[] args) {
        // 创建对象
        StatisCodeBlock codeBlock = new StatisCodeBlock();
        // 注意：构造代码块通过构造方法自动调用
        System.out.println("======我是分割线======");
        StatisCodeBlock codeBlock2 = new StatisCodeBlock();
    }
}
```

执行结果：
```
1、静态代码块变量： 20
3、静态代码块变量： 200
2、构造代码块变量： 100
4、构造代码块变量： 1000
这是构造方法 StatisCodeBlock()
======我是分割线======
2、构造代码块变量： 100
4、构造代码块变量： 1000
这是构造方法 StatisCodeBlock()
```

由上面的执行结果可知：
- 静态代码块：在类加载JVM时初始化，且只被执行一次；常用来执行类属性的初始化；静态块优先于各种代码块以及构造函数；此外静态代码块不能访问普通变量。

- 构造代码块：每次调用构造方法，构造代码块都执行一次；构造代码块优先于构造函数执行；同时构造代码块的运行依赖于构造函数。

##  继承中代码块执行顺序

下面再我们验证一下继承中代码块执行顺序，示例代码如下。

```
package com.chloneda.jutils.test;

/**
 * @author chloneda
 * @description: 验证继承中代码块执行顺序
 */
class Parent {
    //静态代码块，在方法外出现
    static {
        int number1 = 20;
        System.out.println("1、父类静态代码块变量： " + number1);
    }

    //构造代码块，在方法外出现
    {
        int number2 = 100;
        System.out.println("2、父类构造代码块变量： " + number2);
    }

    public Parent() {
        System.out.println("父类构造方法 Parent()");
    }

    static {
        int number3 = 200;
        System.out.println("3、父类静态代码块变量： " + number3);
    }

    //在构造代码块在构造方法前后，但构造代码块先于构造方法执行
    {
        int number4 = 1000;
        System.out.println("4、父类构造代码块变量： " + number4);
    }
}

class Child extends Parent {
    //静态代码块，在方法外出现
    static {
        int number1 = 2001;
        System.out.println("11、子类静态代码块变量： " + number1);
    }

    //构造代码块，在方法外出现
    {
        int number2 = 10001;
        System.out.println("22、子类构造代码块变量： " + number2);
    }

    public Child() {
        System.out.println("子类构造方法 Child()");
    }

    static {
        int number3 = 2002;
        System.out.println("33、子类静态代码块变量： " + number3);
    }

    //在构造代码块在构造方法前后，但构造代码块先于构造方法执行
    {
        int number4 = 100002;
        System.out.println("44、子类构造代码块变量： " + number4);
    }
}

public class CodeBlockTest {
    public static void main(String[] args) {
        // 创建对象
        Child child = new Child();
        // 注意：构造代码块通过构造方法自动调用
    }
}
```

执行结果如下。
```
1、父类静态代码块变量： 20
3、父类静态代码块变量： 200
11、子类静态代码块变量： 2001
33、子类静态代码块变量： 2002

2、父类构造代码块变量： 100
4、父类构造代码块变量： 1000
父类构造方法 Parent()

22、子类构造代码块变量： 10001
44、子类构造代码块变量： 100002
子类构造方法 Child()
```

由此可见**继承中代码块执行顺序**：父类静态块 ==> 子类静态块 ==> 父类代码块 ==> 父类构造器 ==> 子类代码块 ==> 子类构造器 。


# 小结

通过上面的例子讲解，现在可以解答代码块的核心问题了！

- 静态代码块：在类加载JVM时初始化，且只被执行一次；常用来执行类属性的初始化；静态块优先于各种代码块以及构造函数；此外静态代码块不能访问普通变量。

- 构造代码块：每次调用构造方法，构造代码块都执行一次；构造代码块优先于构造函数执行；同时构造代码块的运行依赖于构造函数。

**代码块初始化时机**：构造代码块在实例对象创建时进行初始化；静态代码块在类加载时进行初始化。

**代码块执行顺序**：静态代码块 ==> main()方法 ==> 构造代码块 ==> 构造方法 ==> 局部代码块 。

**继承中代码块执行顺序**：父类静态块 ==> 子类静态块 ==> 父类代码块 ==> 父类构造器 ==> 子类代码块 ==> 子类构造器 。


这就是java代码块的全部内容，大家理解了吗？


