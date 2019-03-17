---
title: 深入浅出Mybatis系列三-配置详解之properties与environments（mybatis源码篇）
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4036796.html)

上篇文章《深入浅出Mybatis系列（二）---配置简介（mybatis源码篇）》我们通过对mybatis源码的简单分析，可看出，在mybatis配置文件中，在configuration根节点下面，可配置properties、typeAliases、plugins、objectFactory、objectWrapperFactory、settings、environments、databaseIdProvider、typeHandlers、mappers这些节点。那么本次，就会先介绍properties节点和environments节点。

为了让大家能够更好地阅读mybatis源码，我先简单的给大家示例一下properties的使用方法。

# properties节点
```
<configuration>
　<!-- 方法一：从外部指定properties配置文件, 除了使用resource属性指定外，还可通过url属性指定url  
  　　<properties resource="dbConfig.properties"></properties> 
  -->
  <!-- 方法二： 直接配置为xml -->
  <properties>
      <property name="driver" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
      <property name="username" value="root"/>
      <property name="password" value="root"/>
  </properties>
```
那么，我要是 两种方法都同时用了，那么哪种方法优先？
当以上两种方法都xml配置优先， 外部指定properties配置其次。至于为什么，接下来的源码分析会提到，请留意一下。

再看一下envirements元素节点的使用方法吧.

# envirements节点
```
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
    
    <!-- 我再指定一个environment -->
    <environment id="test">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <!-- 与上面的url不一样 -->
        <property name="url" value="jdbc:mysql://localhost:3306/demo"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
    
  </environments>
```
environments元素节点可以配置多个environment子节点， 怎么理解呢？ 

假如我们系统的开发环境和正式环境所用的数据库不一样（这是肯定的）， 那么可以设置两个environment, 两个id分别对应开发环境（dev）和正式环境（final），那么通过配置environments的default属性就能选择对应的environment了， 例如，我将environments的deault属性的值配置为dev, 那么就会选择dev的environment。 至于这个是怎么实现的， 下面源码就会讲。

好啦，上面简单给大家介绍了一下properties 和 environments 的配置， 接下来就正式开始看源码了：

上次我们说过mybatis 是通过XMLConfigBuilder这个类在解析mybatis配置文件的，那么本次就接着看看XMLConfigBuilder对于properties和environments的解析。

# XMLConfigBuilder源码
```
public class XMLConfigBuilder extends BaseBuilder {

    private boolean parsed;
    //xml解析器
    private XPathParser parser;
    private String environment;
  
    /**
    * 上次说到这个方法是在解析mybatis配置文件中能配置的元素节点
    * 今天首先要看的就是properties节点和environments节点
    */
    private void parseConfiguration(XNode root) {
        try {
          //解析properties元素
　　　　　　//issue #117 read properties first
          propertiesElement(root.evalNode("properties")); 
          typeAliasesElement(root.evalNode("typeAliases"));
          pluginElement(root.evalNode("plugins"));
          objectFactoryElement(root.evalNode("objectFactory"));
          objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
          settingsElement(root.evalNode("settings"));

          //解析environments元素
　　　　　　// read it after objectFactory and objectWrapperFactory issue #631
          environmentsElement(root.evalNode("environments")); 
          databaseIdProviderElement(root.evalNode("databaseIdProvider"));
          typeHandlerElement(root.evalNode("typeHandlers"));
          mapperElement(root.evalNode("mappers"));
        } catch (Exception e) {
          throw new BuilderException("Error parsing SQL Mapper Configuration.Cause: "
			　+　e,　e);
        }
    }
  
    
    //下面就看看解析properties的具体方法
    private void propertiesElement(XNode context) throws Exception {
        if (context != null) {
          /**
          * 将子节点的 name 以及value属性set进properties对象
          * 这儿可以注意一下顺序，xml配置优先， 外部指定properties配置其次
          */
          Properties defaults = context.getChildrenAsProperties();
          //获取properties节点上 resource属性的值
          String resource = context.getStringAttribute("resource");
          //获取properties节点上 url属性的值, resource和url不能同时配置
          String url = context.getStringAttribute("url");
          if (resource != null && url != null) {
            throw new BuilderException("The properties element cannot specify 
		both a URL and a resource based property file reference. 
		Please specify one or the other.");
          }
          //把解析出的properties文件set进Properties对象
          if (resource != null) {
            defaults.putAll(Resources.getResourceAsProperties(resource));
          } else if (url != null) {
            defaults.putAll(Resources.getUrlAsProperties(url));
          }

          /**
          * 将configuration对象中已配置的Properties属性与刚刚解析的融合
          * configuration这个对象会装载所解析mybatis配置文件的所有节点元素，
          * 以后也会频频提到这个对象，既然configuration对象用有一系列的get/set方法,
          * 那是否就标志着我们可以使用java代码直接配置？ 
          * 答案是肯定的， 不过使用配置文件进行配置，优势不言而喻
          */
          Properties vars = configuration.getVariables();
          if (vars != null) {
            defaults.putAll(vars);
          }
          //把装有解析配置propertis对象set进解析器， 因为后面可能会用到
          parser.setVariables(defaults);
          //set进configuration对象
          configuration.setVariables(defaults);
        }
    }
    
    //下面再看看解析enviroments元素节点的方法
    private void environmentsElement(XNode context) throws Exception {
        if (context != null) {
            if (environment == null) {
                //解析environments节点的default属性的值
                //例如: <environments default="development">
                environment = context.getStringAttribute("default");
            }
            //递归解析environments子节点
            for (XNode child : context.getChildren()) {
                /**
                * <environment id="development">, 只有enviroment节点有id属性，
                * 那么这个属性有何作用？environments节点下可以拥有多个environment子节点
                * 类似于这样： 
                *          <environments default="development">
                *               <environment id="development">...</environment>
                *               <environment id="test">...</environment>
                *          </environments>
                * 意思就是我们可以对应多个环境，比如开发环境，测试环境等，
                * 由environments的default属性,去选择对应的enviroment
                */
                String id = child.getStringAttribute("id");
                //isSpecial就是根据由environments的default属性去选择对应的enviroment
                if (isSpecifiedEnvironment(id)) {
                    /**
                    * 事务，mybatis有两种：JDBC 和 MANAGED, 配置为JDBC则直接使用JDBC的事务，
		    * 配置为MANAGED则是将事务托管给容器
                    */
                    TransactionFactory txFactory 
                        =transactionManagerElement(child.evalNode("transactionManager"));
                    /**
                    * enviroment节点下面就是dataSource节点了，解析dataSource节点
                    * 下面会贴出解析dataSource的具体方法
                    */
                    DataSourceFactory dsFactory 
			= dataSourceElement(child.evalNode("dataSource"));
                    DataSource dataSource = dsFactory.getDataSource();
                    Environment.Builder environmentBuilder=new Environment.Builder(id)
                          .transactionFactory(txFactory)
                          .dataSource(dataSource);
                    //老规矩，会将dataSource设置进configuration对象
                    configuration.setEnvironment(environmentBuilder.build());
                }
            }
        }
    }
    
    //下面看看dataSource的解析方法
    private DataSourceFactory dataSourceElement(XNode context) throws Exception {
        if (context != null) {
            //dataSource的连接池
            String type = context.getStringAttribute("type");
            //子节点 name, value属性set进一个properties对象
            Properties props = context.getChildrenAsProperties();
            //创建dataSourceFactory
            DataSourceFactory factory 
		= (DataSourceFactory) resolveClass(type).newInstance();
            factory.setProperties(props);
            return factory;
        }
        throw new BuilderException("Environment declaration 
		requires a DataSourceFactory.");
    } 
}
```
通过以上对mybatis源码的解读，相信大家对mybatis的配置又有了一个深入的认识。

还有一个问题， 上面我们看到，在配置dataSource的时候使用了 ${driver} 这种表达式， 这种形式是怎么解析的？其实，是通过PropertyParser这个类解析：

# PropertyParser源码
```
/**
 * 这个类解析${}这种形式的表达式
 */
public class PropertyParser {

  public static String parse(String string, Properties variables) {
    VariableTokenHandler handler = new VariableTokenHandler(variables);
    GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
    return parser.parse(string);
  }

  private static class VariableTokenHandler implements TokenHandler {
    private Properties variables;

    public VariableTokenHandler(Properties variables) {
      this.variables = variables;
    }

    public String handleToken(String content) {
      if (variables != null && variables.containsKey(content)) {
        return variables.getProperty(content);
      }
      return "${" + content + "}";
    }
  }
}
```
好啦，以上就是对于properties 和 environments元素节点的分析，比较重要的都在对于源码的注释中标出。本次文章到此结束，接下来的文章会继续分析其他节点的配置。


---

