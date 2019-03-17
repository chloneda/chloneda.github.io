---
title: 深入浅出Mybatis系列六-objectFactory、plugins、mappers简介与配置
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4046488.html)

上篇文章《深入浅出Mybatis系列（五）---TypeHandler简介及配置（mybatis源码篇）》简单看了一下TypeHandler， 本次将结束对于mybatis的配置文件的学习， 本次涉及到剩下没提及到的几个节点的配置：objectFactory、databaseIdProvider、plugins、mappers。

那么，接下来，就简单介绍一下这几个配置的作用吧：

1、objectFactory是干什么的？ 需要配置吗？

MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。默认情况下，我们不需要配置，mybatis会调用默认实现的objectFactory。 除非我们要自定义ObjectFactory的实现， 那么我们才需要去手动配置。

那么怎么自定义实现ObjectFactory？ 怎么配置呢？

自定义ObjectFactory只需要去继承DefaultObjectFactory（是ObjectFactory接口的实现类），并重写其方法即可。具体的，本处不多说，后面再具体讲解。

写好了ObjectFactory, 仅需做如下配置： 
```
<configuration>
    ......
    <objectFactory type="org.mybatis.example.ExampleObjectFactory">
        <property name="someProperty" value="100"/>
    </objectFactory>
    ......
  </configuration
```

plugin有何作用？ 需要配置吗？

plugins 是一个可选配置。mybatis中的plugin其实就是个interceptor， 它可以拦截Executor 、ParameterHandler 、ResultSetHandler 、StatementHandler 的部分方法，处理我们自己的逻辑。Executor就是真正执行sql语句的东西， ParameterHandler 是处理我们传入参数的，还记得前面讲TypeHandler的时候提到过，mybatis默认帮我们实现了不少的typeHandler, 当我们不显示配置typeHandler的时候，mybatis会根据参数类型自动选择合适的typeHandler执行，其实就是ParameterHandler 在选择。ResultSetHandler 就是处理返回结果的。

怎么自定义plugin? 怎么配置？

要自定义一个plugin, 需要去实现Interceptor接口， 这儿不细说， 后面实战部分会详细讲解。定义好之后，配置如下：
```
<configuration>
    ......
    <plugins>
      <plugin interceptor="org.mybatis.example.ExamplePlugin">
        <property name="someProperty" value="100"/>
      </plugin>
    </plugins>
    ......
  </configuration>
```

mappers, 这下引出mybatis的核心之一了，mappers作用 ? 需要配置吗？

mappers 节点下，配置我们的mapper映射文件， 所谓的mapper映射文件，就是让mybatis 用来建立数据表和javabean映射的一个桥梁。在我们实际开发中，通常一个mapper文件对应一个dao接口， 这个mapper可以看做是dao的实现。所以,mappers必须配置。

那么怎么配置呢？
```
<configuration>
    ......
    <mappers>
	     <!-- 第一种方式：通过resource指定 -->
	     <mapper resource="com/dy/dao/userDao.xml"/>
	    
	     <!-- 
		第二种方式， 通过class指定接口，进而将接口与对应的xml文件形成映射关系
		不过，使用这种方式必须保证 接口与mapper文件同名(不区分大小写)， 
		我这儿接口是UserDao,那么意味着mapper文件为UserDao.xml 
		<mapper class="com.dy.dao.UserDao"/>
	     -->
	      
	     <!-- 
		第三种方式，直接指定包，自动扫描，与方法二同理 
		<package name="com.dy.dao"/>
	     -->

	     <!-- 
		第四种方式：通过url指定mapper文件位置
		<mapper url="file://........"/>
	     -->
  </mappers>
    ......
  </configuration>
```

本篇仅作简单介绍， 更高级的使用以及其实现原理，会在后面的实战部分进行详细讲解。

这几个节点的解析源码，与之前提到的那些节点的解析类似，源码需要的可以从这里看看。

# 相关节点源码
```
/**
 * objectFactory 节点解析
 */
private void objectFactoryElement(XNode context) throws Exception {
    if (context != null) {
      //读取type属性的值， 接下来进行实例化ObjectFactory, 并set进 configuration
      //到此，简单讲一下configuration这个对象，其实它里面主要保存的都是mybatis的配置
      String type = context.getStringAttribute("type");
      //读取propertie的值， 根据需要可以配置， mybatis默认实现的objectFactory没有使用properties
      Properties properties = context.getChildrenAsProperties();
      
      ObjectFactory factory = (ObjectFactory) resolveClass(type).newInstance();
      factory.setProperties(properties);
      configuration.setObjectFactory(factory);
    }
 }
 
  
  /**
   * plugins 节点解析
   */
  private void pluginElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        String interceptor = child.getStringAttribute("interceptor");
        Properties properties = child.getChildrenAsProperties();
        /**
        * 由此可见，我们在定义一个interceptor的时候，需要去实现Interceptor, 
        * 这儿先不具体讲，以后会详细讲解
        */
        Interceptor interceptorInstance 
		= (Interceptor) resolveClass(interceptor).newInstance();
        interceptorInstance.setProperties(properties);
        configuration.addInterceptor(interceptorInstance);
      }
    }
  }
  
  /**
   * mappers 节点解析
   * 这是mybatis的核心之一，这儿先简单介绍，在接下来的文章会对它进行分析
   */
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          //如果mappers节点的子节点是package, 那么就扫描package下的文件, 注入进configuration
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          //resource, url, class 三选一
          
          if (resource != null && url == null && mapperClass == null) {
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            //mapper映射文件都是通过XMLMapperBuilder解析
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream
						, configuration
						, resource
						, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream
						, configuration
						, url
						, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url
			, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

本次就简单的到此结束， 从下篇文章开始，将会开始接触到mybatis的核心部分，不过首先还是要讲mapper文件的配置及使用， 并通过源码进一步深入核心。


---
