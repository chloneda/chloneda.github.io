---
title: 深入浅出Mybatis系列七-mapper映射文件配置之insert、update、delete
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4048828.html)

上篇文章《深入浅出Mybatis系列（六）---objectFactory、plugins、mappers简介与配置》简单地给mybatis的配置画上了一个句号。那么从本篇文章开始，将会介绍mapper映射文件的配置， 这是mybatis的核心之一，一定要学好。在mapper文件中，以mapper作为根节点，其下面可以配置的元素节点有： select, insert, update, delete, cache, cache-ref, resultMap, sql 。

本篇文章将简单介绍 insert, update, delete 的配置及使用，以后会对mybatis的源码进行深入讲解。

相信，看到insert, update, delete， 我们就知道其作用了，顾名思义嘛，myabtis 作为持久层框架，必须要对CRUD啊。

好啦，咱们就先来看看 insert, update, delete 怎么配置， 能配置哪些元素吧：

相关元素配置
```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<!-- mapper 为根元素节点， 一个namespace对应一个dao -->
<mapper namespace="com.dy.dao.UserDao">

    <insert
      <!-- 
        1. id （必须配置）
        id是命名空间中的唯一标识符，可被用来代表这条语句。 
        一个命名空间（namespace） 对应一个dao接口, 
        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致 
　　　　-->
      
      id="insertUser"
      
      <!-- 
         2. parameterType （可选配置, 默认为mybatis自动选择处理）
         将要传入语句的参数的完全限定类名或别名， 如果不配置，
　　　　　 mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
         parameterType 主要指定参数类型，可以是int, short, long, 
　　　　　 string等类型，也可以是复杂类型（如对象）
　　　　-->
      
      parameterType="com.demo.User"
      
      <!-- 
         3. flushCache （可选配置，默认配置为true）
         将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，
         默认值：true（对应插入、更新和删除语句） 
      -->
      
      flushCache="true"
      
      <!-- 
         4. statementType （可选配置，默认配置为PREPARED）
         STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，
         PreparedStatement 或 CallableStatement，默认值：PREPARED。 
      -->
      
      statementType="PREPARED"
      
      <!-- 
         5. keyProperty (可选配置， 默认为unset)
        （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 
         getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，
         默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 
      -->
      
      keyProperty=""
      
      <!-- 
         6. keyColumn     (可选配置)
        （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库
        （像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。
         如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 
      -->
      
      keyColumn=""
      
      <!-- 
         7. useGeneratedKeys (可选配置， 默认为false)
        （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 
         getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 
         这样的关系数据库管理系统的自动递增字段），默认值：false。  
      -->
      
      useGeneratedKeys="false"
      
      <!-- 
         8. timeout  (可选配置， 默认为unset, 依赖驱动)
         这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 
      -->
      timeout="20">

    <update
      id="updateUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">

    <delete
      id="deleteUser"
      parameterType="com.demo.User"
      flushCache="true"
      statementType="PREPARED"
      timeout="20">
</mapper>
```
以上就是一个模板配置， 哪些是必要配置，哪些是根据自己实际需求，看一眼就知道了。

下面， 还是用第一篇文章《深入浅出Mybatis系列（一）---Mybatis入门》里面的demo来示例吧：

# User类
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
# UserDao类
```
public interface UserDao {
    public void insertUser (User user);
    public void updateUser (User user);
    public void deleteUser (User user);
}
```
# userDao.xml
```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

<mapper namespace="com.dy.dao.UserDao">
   
   <!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
           insert into user(id, name, password, age, deleteFlag) 
               values(#{id}
			, #{name}
			, #{password}
			, #{age}
			, #{deleteFlag})
   </insert>
   
   <!-- 对应userDao中的updateUser方法 -->
   <update id="updateUser" parameterType="com.dy.entity.User">
           update user set name = #{name}
		, password = #{password}
		, age = #{age}
		, deleteFlag = #{deleteFlag}
               where id = #{id};
   </update>
    
   <!-- 对应userDao中的deleteUser 方法 --> 
   <delete id="deleteUser" parameterType="com.dy.entity.User">
           delete from user where id = #{id};
   </delete>
</mapper>
```
这样，一个简单的映射关系就建立了。仔细观察上面parameterType,  "com.dy.entity.User"，包名要是再长点呢，每次都这样写，写得蛋疼了。别忘了之前讲的 typeAliases（别名）， 那么这个地方，用上别名，岂不是技能跟蛋疼的长长的包名说拜拜了。好啦，咱们配上别名，在哪儿配？ 当然是在mybatis 的全局配置文件（我这儿名字是mybatis-conf.xml）， 不要认为是在mapper的配置文件里面配置哈。

# mybatis-conf.xml
```
<typeAliases>
      <!--
      通过package, 可以直接指定package的名字， mybatis会自动扫描你指定包下面的javabean,
      并且默认设置一个别名，默认的名字为： javabean 的首字母小写的非限定类名来作为它的别名。
      也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user) 
      <package name="com.dy.entity"/>
       -->
      <typeAlias alias="user" type="com.dy.entity.User"/>
  </typeAliases>
```
这样，一个别名就取好了，咱们可以把上面的 com.dy.entity.User 都直接改为user 了。 这多方便呀！

我这儿数据库用的是mysql, 我把user表的主键id 设置了自动增长， 以上代码运行正常， 那么问题来了（当然，我不是要问学挖掘机哪家强），我要是换成oracle数据库怎么办？ oracle 可是不支持id自增长啊？ 怎么办？请看下面：
```
<!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
           <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id 
           
        <selectKey resultType="int" order="BEFORE" keyProperty="id">
              select seq_user_id.nextval as id from dual
        </selectKey>
        
        -->   
           insert into user(id, name, password, age, deleteFlag) 
               values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
   </insert>
```
同理，如果我们在使用mysql的时候，想在数据插入后返回插入的id, 我们也可以使用 selectKey 这个元素：
```
<!-- 对应userDao中的insertUser方法，  -->
   <insert id="insertUser" parameterType="com.dy.entity.User">
        <!-- 
	oracle等不支持id自增长的，可根据其id生成策略，先获取id 
        <selectKey resultType="int" order="BEFORE" keyProperty="id">
              select seq_user_id.nextval as id from dual
        </selectKey>
        --> 
        
        <!-- mysql插入数据后，获取id -->
        <selectKey keyProperty="id" resultType="int" order="AFTER" >
               SELECT LAST_INSERT_ID() as id
        </selectKey>
          
        insert into user(id, name, password, age, deleteFlag) 
                values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})
   </insert>
```
这儿，我们就简单提一下 <selectKey> 这个元素节点吧:
```
<selectKey
        <!-- 
	selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，
	也可以是逗号分隔的属性名称列表。 
	-->
        keyProperty="id"

        <!-- 
	结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。
	MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，
	则可以使用一个包含期望属性的 Object 或一个 Map。 
	-->
        resultType="int"

        <!-- 
	这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，
	设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，
	然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 
	-->
        order="BEFORE"

        <!-- 
	与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，
	分别代表 PreparedStatement 和 CallableStatement 类型。 
	-->
        statementType="PREPARED">
```

好啦，本篇文章主要介绍了insert, update, delete的配置及用法。 下篇文章将介绍复杂的 select相关的配置及用法， 待这些都讲完后，会先根据源码分析一下mybatis的整个运行流程，然后再深入mybatis的用法。


---
