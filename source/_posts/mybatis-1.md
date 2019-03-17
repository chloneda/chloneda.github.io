---
title: 深入浅出Mybatis系列一-Mybatis入门
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4031382.html)

最近两年 springmvc + mybatis 的在这种搭配还是蛮火的，楼主我呢，也从来没真正去接触过mybatis, 趁近日得闲， 就去学习一下mybatis吧。 本次拟根据自己的学习进度，做一次关于mybatis 的一系列教程， 记录自己的学习历程， 同时也给还没接触过mybatis的朋友探一次道。本系列教程拟 由浅（使用）入深（分析mybatis源码实现），故可能需要好长几天才能更新完。好啦，下面就开始本次的mybatis 学习之旅啦， 本次为第一篇教程， 就先简单地写个demo, 一起来认识一下mybatis吧。　　

为了方便，我使用了maven，至于maven怎么使用，我就不做介绍了。没用过maven的，也不影响阅读。

# Mybatis环境搭建

新建web项目， 添加依赖包：mybatis包、数据库驱动包(我使用的是mysql)、日志包(我使用的是log4j)， 由于我的是maven项目， 那么添加依赖包就简单了，直接在pom.xml添加依赖即可。

**pom.xml**

```
<dependencies>
      <!-- 添加junit -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    
    <!-- 添加log4j -->
    <dependency>
        <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
        <version>1.2.16</version>
    </dependency>
    
    <!-- 添加mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
        <version>3.2.6</version>
    </dependency>
    
    <!-- 添加mysql驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.12</version>
    </dependency>
    
  </dependencies>
```

# 配置mybatis
在classpath建立一个用于配置log4j的配置文件log4j.properties, 再建立一个用于配置Mybatis的配置文件configuration.xml（文件可随便命名）。log4j的配置，我就不多说，这儿主要说一下configuration.xml:

**configuration.xml**
```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

  <!-- 指定properties配置文件， 我这里面配置的是数据库相关 -->
  <properties resource="dbConfig.properties"></properties>
  
  <!-- 指定Mybatis使用log4j -->
  <settings>
     <setting name="logImpl" value="LOG4J"/>
  </settings>
      
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
          <!--
          如果上面没有指定数据库配置的properties文件，那么此处可以这样直接配置 
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
         -->
         
         <!-- 上面指定了数据库配置文件， 配置文件里面也是对应的这四个属性 -->
         <property name="driver" value="${driver}"/>
         <property name="url" value="${url}"/>
         <property name="username" value="${username}"/>
         <property name="password" value="${password}"/>
         
      </dataSource>
    </environment>
  </environments>
  
  <!-- 映射文件，mybatis精髓， 后面才会细讲 -->
  <mappers>
    <mapper resource="com/dy/dao/userDao-mapping.xml"/>
  </mappers>
  
</configuration>
```

# 开始写Demo
首先，在mysql数据库test1建立一张表user。

先编写一个实体类User: User类用于与User表相对应。

**User**
```
public class User {

    private int id;
    private String name;
    private String password;
    private int age;
    private int deleteFlag;

    //setter和getter方法省略...
}
```
再编写一个UserDao 接口：

**UserDao**
```
public interface UserDao {
    public void insert(User user);
    public User findUserById (int userId);
    public List<User> findAllUsers();
}
```
再编写一个userDao-mapping.xml （可随便命名）:

**userDao-mapping.xml**
```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 
<mapper namespace="com.dy.dao.UserDao">
   <select id="findUserById" resultType="com.dy.entity.User" > 
      select * from user where id = #{id}
   </select>
</mapper>
```
userDao-mapping.xml相当于是UserDao的实现， 同时也将User实体类与数据表User成功关联起来。

# 测试代码Demo
**UserDaoTest**
```
public class UserDaoTest {
    @Test
    public void findUserById() {
        SqlSession sqlSession = getSessionFactory().openSession();  
        UserDao userMapper = sqlSession.getMapper(UserDao.class);  
        User user = userMapper.findUserById(2);  
        Assert.assertNotNull("没找到数据", user);
    }
    
    //Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互
    private static SqlSessionFactory getSessionFactory() {  
        SqlSessionFactory sessionFactory = null;  
        String resource = "configuration.xml";  
        try {  
            sessionFactory = new SqlSessionFactoryBuilder().build(Resources  
                    .getResourceAsReader(resource));
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        return sessionFactory;  
    }  
}
```
好啦，这样一个简单的mybatis 的demo就能成功运行啦。通过这个demo, 应该你就也能初步看出mybatis的运行机制，如果不清楚，也没关系。从下一篇文章开始，才开始正式讲解mybatis。


---
