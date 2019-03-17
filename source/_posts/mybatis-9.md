---
title: 深入浅出Mybatis系列九-强大的动态SQL
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4092662.html)

上篇文章《深入浅出Mybatis系列（八）---mapper映射文件配置之select、resultMap》简单介绍了mybatis的查询，至此，CRUD都已讲完。本文将介绍mybatis强大的动态SQL。

那么，问题来了： 什么是动态SQL? 动态SQL有什么作用？

传统的使用JDBC的方法，相信大家在组合复杂的的SQL语句的时候，需要去拼接，稍不注意哪怕少了个空格，都会导致错误。Mybatis的动态SQL功能正是为了解决这种问题， 其通过 if, choose, when, otherwise, trim, where, set, foreach标签，可组合成非常灵活的SQL语句，从而提高开发人员的效率。下面就去感受Mybatis动态SQL的魅力吧：

# mybatis中if判断
作为程序猿，谁不懂if　!在mybatis中也能用 if 啦：
```
<select id="findUserById" resultType="user">
           select * from user where 
           <if test="id != null">
               id=#{id}
           </if>
            and deleteFlag=0;
</select>
```

上面例子： 如果传入的id 不为空， 那么才会SQL才拼接id = #{id}。 这个相信大家看一样就能明白，不多说。

细心的人会发现一个问题：“你这不对啊！ 要是你传入的id为null,  那么你这最终的SQL语句不就成了 select * from user where and deleteFlag=0,  这语句有问题！”

是啊，这时候，mybatis的 where 标签就该隆重登场啦：

# mybatis中where
咱们通过where改造一下上面的例子：
```
<select id="findUserById" resultType="user">
           select * from user 
           <where>
               <if test="id != null">
                   id=#{id}
               </if>
               and deleteFlag=0;
           </where>
 </select>
```

有些人就要问了： “你这都是些什么玩意儿！ 跟上面的相比， 不就是多了个where标签嘛！ 那这个还会不会出现  select * from user where and deleteFlag=0 ？”

的确，从表面上来看，就是多了个where标签而已， 不过实质上， mybatis是对它做了处理，当它遇到AND或者OR这些，它知道怎么处理。其实我们可以通过 trim 标签去自定义这种处理规则。

# trim:我的地盘，我做主
上面的where标签，其实用trim 可以表示如下：
```
<trim prefix="WHERE" prefixOverrides="AND |OR ">
  ... 
</trim>
```
它的意思就是： 当WHERE后紧随AND或则OR的时候，就去除AND或者OR。 除了WHERE以外， 其实还有一个比较经典的实现，那就是SET。

# set: 信我，不出错
```
<update id="updateUser" parameterType="com.dy.entity.User">
           update user set 
           <if test="name != null">
               name = #{name},
           </if> 
           <if test="password != null">
               password = #{password},
           </if> 
           <if test="age != null">
               age = #{age}
           </if> 
           <where>
               <if test="id != null">
                   id = #{id}
               </if>
               and deleteFlag = 0;
           </where>
</update>
```
问题又来了： “如果我只有name不为null,  那么这SQL不就成了 update set name = #{name}, where ........ ?  你那name后面那逗号会导致出错啊！”

是的，这时候，就可以用mybatis为我们提供的set 标签了。下面是通过set标签改造后：
```
<update id="updateUser" parameterType="com.dy.entity.User">
           update user
        <set>
          <if test="name != null">name = #{name},</if> 
             <if test="password != null">password = #{password},</if> 
             <if test="age != null">age = #{age},</if> 
        </set>
        <where>
             <if test="id != null">
                   id = #{id}
             </if>
             	   and deleteFlag = 0;
        </where>
</update>
```

这个用trim 可表示为：
```
<trim prefix="SET" suffixOverrides=",">
  ...
</trim>
```

# foreach:  你有for, 我有foreach！
java中有for, 可通过for循环， 同样， mybatis中有foreach, 可通过它实现循环，循环的对象当然主要是java容器和数组。
```
<select id="selectPostIn" resultType="domain.blog.Post">
	SELECT *
	  FROM POST P
	  WHERE ID in
	<foreach item="item" index="index" collection="list"
	      open="(" separator="," close=")">
		#{item}
	</foreach>
</select>
```
将一个 List 实例或者数组作为参数对象传给 MyBatis，当这么做的时候，MyBatis 会自动将它包装在一个 Map 中并以名称为键。List 实例将会以“list”作为键，而数组实例的键将是“array”。同样， 当循环的对象为map的时候，index其实就是map的key。

# choose: 我选择了你，你选择了我
Java中有switch,  mybatis有choose
```
<select id="findActiveBlogLike" resultType="Blog">
	SELECT * FROM BLOG WHERE state = ‘ACTIVE’
	<choose>
	    <when test="title != null">
	        AND title like #{title}
	    </when>
	    <when test="author != null and author.name != null">
	        AND author_name like #{author.name}
	    </when>
	    <otherwise>
	        AND featured = 1
	    </otherwise>
	</choose>
</select>
```

以上例子中： 当title和author都不为null的时候， 那么选择二选一（前者优先）， 如果都为null, 那么就选择 otherwise中的， 如果tilte和author只有一个不为null, 那么就选择不为null的那个。

纵观mybatis的动态SQL， 强大而简单， 相信大家简单看一下就能使用了。

好啦，本次就写到这！下篇文章将结合mybatis的源码分析一次sql语句执行的整个过程。


---
