---
title: 深入浅出Mybatis系列二-配置简介（mybatis源码篇）
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4031613.html)

上篇文章《深入浅出Mybatis系列（一）---Mybatis入门》， 写了一个Demo简单体现了一下Mybatis的流程。本次，将简单介绍一下Mybatis的配置文件：

上次例子中，我们以 SqlSessionFactoryBuilder 去创建 SqlSessionFactory,  那么，我们就先从SqlSessionFactoryBuilder入手， 咱们先看看源码是怎么实现的：

# SqlSessionFactoryBuilder源码
```
public class SqlSessionFactoryBuilder {

  /**
  * Reader读取mybatis配置文件，传入构造方法
  * 除了Reader外，其实还有对应的inputStream作为参数的构造方法，
  * 这也体现了mybatis配置的灵活性
  */
  public SqlSessionFactory build(Reader reader) {
    return build(reader, null, null);
  }

  public SqlSessionFactory build(Reader reader, String environment) {
    return build(reader, environment, null);
  }
  
  //mybatis配置文件 + properties, 此时mybatis配置文件中可以不配置properties，也能使用${}形式
  public SqlSessionFactory build(Reader reader, Properties properties) {
    return build(reader, null, properties);
  }
  
  //通过XMLConfigBuilder解析mybatis配置，然后创建SqlSessionFactory对象
  public SqlSessionFactory build(Reader reader
		, String environment
		, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      //下面看看这个方法的源码
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }

  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }

}
```
通过源码，我们可以看到SqlSessionFactoryBuilder 通过XMLConfigBuilder 去解析我们传入的mybatis的配置文件， 下面就接着看看 XMLConfigBuilder 部分源码：
 
# XMLConfigBuilder源码
```
/**
 * mybatis 配置文件解析
 */
public class XMLConfigBuilder extends BaseBuilder {
  public XMLConfigBuilder(InputStream inputStream,String environment,Properties props){
    this(new XPathParser(inputStream, true, props
			, new XMLMapperEntityResolver()), environment, props);
  }

  private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
  }
  
  //外部调用此方法对mybatis配置文件进行解析
  public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    //从根节点configuration
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }

  //此方法就是解析configuration节点下的子节点
  //由此也可看出，我们在configuration下面能配置的节点为以下10个节点
  private void parseConfiguration(XNode root) {
    try {
　　　　//issue #117 read properties first
      propertiesElement(root.evalNode("properties")); 
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      settingsElement(root.evalNode("settings"));

　　　　// read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments")); 
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " 
			+ e, e);
    }
  }
}
```
通过以上源码，我们就能看出，在mybatis的配置文件中：

1. configuration节点为根节点。

2. 在configuration节点之下，我们可以配置10个子节点， 分别为：properties、typeAliases、plugins、objectFactory、objectWrapperFactory、settings、environments、databaseIdProvider、typeHandlers、mappers。

本篇文章就先只介绍这些内容，接下来的文章将依次分析解析这个10个节点中比较重要的几个节点的源码，看看在解析这些节点的时候，到底做了些什么。


---
