---
title: Java:如何优雅地使用close()?
tags: Java
categories: Java
keywords: Java
comments: false
---

**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

# 前言

本文尽量采用通俗易懂、循序渐进的方式，让大家真正优雅地使用close()方法！

# 问题场景

平时我们使用资源后一般都会关闭资源，即close()方法，但这个步骤重复性很高，还面临上述执行顺序不明的风险，而且很多人还是不能正确合理地关闭资源。

我们来看看close()是怎么错误地关闭资源的？

# 错误的close()

先来看看如下的错误关闭资源方式：

```
package com.chloneda.jutils.test;

import java.sql.*;

/**
 * @author chloneda
 * @description: close()方法测试
 * 错误的close()
 */
public class CloseTest {
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            //1.加载驱动程序
            Class.forName("com.mysql.jdbc.Driver");
            //2.获得数据库链接
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/common", "root", "123456");
            //3.通过数据库的连接操作数据库，实现增删改查
            st = conn.createStatement();
            rs = st.executeQuery("select * from new_table");
            //4.处理数据库的返回结果
            while (rs.next()) {
                System.out.println(rs.getString("id") + " " + rs.getString("name"));
            }
            //5.关闭资源
            rs.close();
            st.close();
            conn.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
上面代码的资源关闭写在了try代码块中，一旦close方法调用之前(比如**3步骤**)就抛出异常，那么关闭资源的代码就永远不会得到执行。

如果我们把关闭资源的代码放在finally中行不行呢？
```
    try {
        //1.加载驱动程序
        Class.forName("com.mysql.jdbc.Driver");
        //2.获得数据库链接
        conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/common", "root", "123456");
        //3.通过数据库的连接操作数据库，实现增删改查
        st = conn.createStatement();
        rs = st.executeQuery("select * from new_table");
        //4.处理数据库的返回结果
        while (rs.next()) {
            System.out.println(rs.getString("id") + " " + rs.getString("name"));
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        //5.关闭资源
        try {
            rs.close();
            st.close();
            conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
答案是不行！如果在 **2步骤** 的try中conn获得数据库链接抛出异常，那么conn仍然为null，此时进入finally代码块中，执行close()就报空指针异常了，关闭资源没有意义！因此，我们需要在close()之前判断一下conn等是否为空，只有不为空的时候才需要close。


# 常见的close()

针对上述场景，得到常见的使用close()方式如下：
```
/**
 * @author chloneda
 * @description: close()方法测试
 * 常见的close()
 */
public class CloseTest {
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            //1.加载驱动程序
            Class.forName("com.mysql.jdbc.Driver");
            //2.获得数据库链接
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/common", "root", "123456");
            //3.通过数据库的连接操作数据库，实现增删改查
            st = conn.createStatement();
            rs = st.executeQuery("select * from new_table");
            //4.处理数据库的返回结果
            while (rs.next()) {
                System.out.println(rs.getString("id") + " " + rs.getString("name"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源
            if (null != rs) {
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (null != st) {
                try {
                    st.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (null != conn) {
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }

        }
    }
}

```
这是常见的close()！但是finally代码块的代码重复性太高了，这还只是三个资源的关闭，如果有很多个资源需要在finally中关闭，那不是需要编写很多不优雅的代码？其实，关闭资源是没啥逻辑的代码，我们需要精简代码，减少代码重复性，优雅地编程！


# 使用AutoCloseable接口

自从Java7以后，我们可以使用 **AutoCloseable接口** (Closeable接口也可以)来优雅的关闭资源了 看看修改例子：
```
/**
 * @author chloneda
 * @description: close()方法测试
 * 使用AutoCloseable接口
 */
public class CloseTest {
    public static void main(String[] args) {
        Connection conn = null;
        Statement st = null;
        ResultSet rs = null;
        try {
            //1.加载驱动程序
            Class.forName("com.mysql.jdbc.Driver");
            //2.获得数据库链接
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/common", "root", "123456");
            //3.通过数据库的连接操作数据库，实现增删改查
            st = conn.createStatement();
            rs = st.executeQuery("select * from new_table");
            //4.处理数据库的返回结果
            while (rs.next()) {
                System.out.println(rs.getString("id") + " " + rs.getString("name"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //5.关闭资源，调用自定义的close()方法
            close(rs);
            close(st);
            close(conn);
        }
    }

    public static void close(AutoCloseable closeable) {
        if (closeable != null) {
            try {
                closeable.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
上面的finally代码块的代码量是不是减少了许多，就单纯地调用的静态的 **close(AutoCloseable closeable)** 方法。为什么可以这样呢？

其实Connection、Statement、ResultSet三个接口都继承了**AutoCloseable接口**。所以只要涉及到资源的关闭，继承了AutoCloseable接口，实现了close()方法，我们都可以调用 **close(AutoCloseable closeable)** 方法进行资源关闭。

此外，java IO流的很多类都实现了 **Closeable接口**，而**Closeable接口**又继承自 **AutoCloseable接口**，也可以调用上面的 **close(AutoCloseable closeable)** 方法进行资源关闭。是不是一语惊醒梦中人啊？


# 使用try-with-resources

其实Java7以后，还有一种关闭资源的方式，也就是 **try-with-resources**，这种方式也是我们推荐的！很优雅！

我们来看看它是怎么优雅地关闭资源的！
```
/**
 * @author chloneda
 * @description: close()方法测试
 * 使用try-with-resources
 */
public class CloseTest {
    public static void main(String[] args) throws ClassNotFoundException {
        //1.加载驱动程序
        Class.forName("com.mysql.jdbc.Driver");
        try (//2.获得数据库链接
             Connection conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/common", "root", "123456");
             //3.通过数据库的连接操作数据库，实现增删改查
             Statement st = conn.createStatement();
             ResultSet rs = st.executeQuery("select * from new_table")
        ) {
            //4.处理数据库的返回结果
            while (rs.next()) {
                System.out.println(rs.getString("id") + " " + rs.getString("name"));
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
这种方式就省略了finally，不必重复编写关闭资源的代码了！而且资源也得到了关闭！怎么验证这个问题？可以查看底下的 **实际应用** 章节！

其实**try-with-resources**关闭资源的操作，本质上是继承了**java.lang.AutoCloseable接口**，实现了close方法，所以使用**try-with-resources**能关闭资源。很神奇吧！

# 实际应用

这个章节就是验证使用**try-with-resources**可以关闭资源的问题的！

上面我们说了使用**try-with-resources**关闭资源，只要是继承了**java.lang.AutoCloseable接口** 实现close()方法就可以使用！

我们自定义一个资源类，来看看实际应用吧！
```
package com.chloneda.jutils.test;

/**
 * @author chloneda
 * @description: 资源类, 实现AutoCloseable接口.
 */
public class Resources implements AutoCloseable {
    public void useResource() {
        System.out.println("useResource:{} 正在使用资源!");
    }

    @Override
    public void close() {
        System.out.println("close:{} 自动关闭资源!");
    }
}

/**
 * @description: 使用try-with-resources自动关闭资源测试.
 */
class AutoClosableTest {
    public static void main(String[] args) {
        /** 使用try-with-resource，自动关闭资源 */
        try (
                Resources resource = new Resources()
        ) {
            resource.useResource();
        } catch (Exception e) {
            e.getMessage();
        } finally {
            System.out.println("Finally!");
        }
    }
}
```
结果输出。
```
useResource:{} 正在使用资源!
close:{} 自动关闭资源!
Finally!
```
看到运行结果了吗？Resources类实现AutoCloseable接口，实现了close()方法，**try-with-resources** 就会自动关闭资源！

一旦**Resources**类没有继承**java.lang.AutoCloseable接口**，没有实现close()方法，AutoClosableTest类的try模块就在编译期报错，提示信息如下。
```
Incompatible types.
Required: java.lang.AutoCloseable
Found: com.chloneda.jutils.test.Resources
```

最后，需要说明的是**try-with-resources**就是一个**JVM语法糖**！关于JVM语法糖可以查查相关资料，看看Java中有哪些有趣的语法糖！

# 尾语

《Effective Java》在第三版中也推荐使用try-with-resources语句替代try-finally语句。

所以在处理必须关闭的资源时，使用try-with-resources语句替代try-finally语句。生成的代码更简洁，更清晰，并且生成的异常更有用。 try-with-resources语句在编写必须关闭资源的代码时会更容易，也不会出错，而使用try-finally语句实际上是不可能的。

如此，推荐大家使用**try-with-resources**优雅地关闭资源！



---

