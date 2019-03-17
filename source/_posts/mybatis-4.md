---
title: 深入浅出Mybatis系列四-配置详解之typeAliases别名（mybatis源码篇）
tags: Mybatis
categories: Mybatis
keywords: Mybatis,Source code
comments: false
---
注：本文转载自[南轲梦](https://www.cnblogs.com/dongying/p/4037678.html)

上篇文章《深入浅出Mybatis系列（三）---配置详解之properties与environments（mybatis源码篇）》 介绍了properties与environments， 本篇继续讲剩下的配置节点之一：typeAliases。 typeAliases节点主要用来设置别名，其实这是挺好用的一个功能， 通过配置别名，我们不用再指定完整的包名，并且还能取别名。

例如： 我们在使用 com.demo.entity. UserEntity 的时候，我们可以直接配置一个别名user, 这样以后在配置文件中要使用到com.demo.entity. UserEntity的时候，直接使用User即可。

就以上例为例，我们来实现一下，看看typeAliases的配置方法:

# typeAliases配置
```
<configuration>
    <typeAliases>
      <!--
      通过package, 可以直接指定package的名字， mybatis会自动扫描你指定包下面的javabean,
      并且默认设置一个别名，默认的名字为： javabean 的首字母小写的非限定类名来作为它的别名。
      也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user) 
      <package name="com.dy.entity"/>
       -->
      <typeAlias alias="UserEntity" type="com.dy.entity.User"/>
  </typeAliases>
  
  ......
  
</configuration>
```
再写一段测试代码，看看有没生效：（我只写一段伪代码）
```
Configuration con = sqlSessionFactory.getConfiguration();
Map<String, Class<?>> typeMap = con.getTypeAliasRegistry().getTypeAliases();
for(Entry<String, Class<?>> entry: typeMap.entrySet()) {
    System.out.println(entry.getKey() + " ================> " 
	+ entry.getValue().getSimpleName());
}
```
上面给大家简单介绍了typeAliases的用法， 接下来就看看Mybatis中的源码了。

老规矩，先从对xml的解析讲起：

# typeAliases源码
```
/**
 * 解析typeAliases节点
 */
private void typeAliasesElement(XNode parent) {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        //如果子节点是package, 那么就获取package节点的name属性， mybatis会扫描指定的package
        if ("package".equals(child.getName())) {
          String typeAliasPackage = child.getStringAttribute("name");
          /**
          * TypeAliasRegistry 负责管理别名， 这儿就是通过TypeAliasRegistry 进行别名注册，
          * 下面就会看看TypeAliasRegistry源码
          */
          configuration.getTypeAliasRegistry().registerAliases(typeAliasPackage);
        } else {
          //如果子节点是typeAlias节点，那么就获取alias属性和type的属性值
          String alias = child.getStringAttribute("alias");
          String type = child.getStringAttribute("type");
          try {
            Class<?> clazz = Resources.classForName(type);
            if (alias == null) {
              typeAliasRegistry.registerAlias(clazz);
            } else {
              typeAliasRegistry.registerAlias(alias, clazz);
            }
          } catch (ClassNotFoundException e) {
            throw new BuilderException("Error registering typeAlias for '" 
		+ alias + "'. Cause: " + e, e);
          }
        }
      }
    }
  }
```
重要的源码在这儿!

# TypeAliasRegistry源码
```
public class TypeAliasRegistry {
  
  /**
   * 这就是核心所在啊， 原来别名就仅仅通过一个HashMap来实现， 
   * key为别名， value就是别名对应的类型（class对象）
   */
  private final Map<String, Class<?>> TYPE_ALIASES = new HashMap<String, Class<?>>();

  /**
   * 以下就是mybatis默认为我们注册的别名
   */
  public TypeAliasRegistry() {
    registerAlias("string", String.class);

    registerAlias("byte", Byte.class);
    registerAlias("long", Long.class);
    registerAlias("short", Short.class);
    registerAlias("int", Integer.class);
    registerAlias("integer", Integer.class);
    registerAlias("double", Double.class);
    registerAlias("float", Float.class);
    registerAlias("boolean", Boolean.class);

    registerAlias("byte[]", Byte[].class);
    registerAlias("long[]", Long[].class);
    registerAlias("short[]", Short[].class);
    registerAlias("int[]", Integer[].class);
    registerAlias("integer[]", Integer[].class);
    registerAlias("double[]", Double[].class);
    registerAlias("float[]", Float[].class);
    registerAlias("boolean[]", Boolean[].class);

    registerAlias("_byte", byte.class);
    registerAlias("_long", long.class);
    registerAlias("_short", short.class);
    registerAlias("_int", int.class);
    registerAlias("_integer", int.class);
    registerAlias("_double", double.class);
    registerAlias("_float", float.class);
    registerAlias("_boolean", boolean.class);

    registerAlias("_byte[]", byte[].class);
    registerAlias("_long[]", long[].class);
    registerAlias("_short[]", short[].class);
    registerAlias("_int[]", int[].class);
    registerAlias("_integer[]", int[].class);
    registerAlias("_double[]", double[].class);
    registerAlias("_float[]", float[].class);
    registerAlias("_boolean[]", boolean[].class);

    registerAlias("date", Date.class);
    registerAlias("decimal", BigDecimal.class);
    registerAlias("bigdecimal", BigDecimal.class);
    registerAlias("biginteger", BigInteger.class);
    registerAlias("object", Object.class);

    registerAlias("date[]", Date[].class);
    registerAlias("decimal[]", BigDecimal[].class);
    registerAlias("bigdecimal[]", BigDecimal[].class);
    registerAlias("biginteger[]", BigInteger[].class);
    registerAlias("object[]", Object[].class);

    registerAlias("map", Map.class);
    registerAlias("hashmap", HashMap.class);
    registerAlias("list", List.class);
    registerAlias("arraylist", ArrayList.class);
    registerAlias("collection", Collection.class);
    registerAlias("iterator", Iterator.class);

    registerAlias("ResultSet", ResultSet.class);
  }

  
  /**
   * 处理别名， 直接从保存有别名的hashMap中取出即可
   */
  @SuppressWarnings("unchecked")
  public <T> Class<T> resolveAlias(String string) {
    try {
      if (string == null) return null;
      String key = string.toLowerCase(Locale.ENGLISH); // issue #748
      Class<T> value;
      if (TYPE_ALIASES.containsKey(key)) {
        value = (Class<T>) TYPE_ALIASES.get(key);
      } else {
        value = (Class<T>) Resources.classForName(string);
      }
      return value;
    } catch (ClassNotFoundException e) {
      throw new TypeException("Could not resolve type alias '" 
		+ string + "'.  Cause: " + e, e);
    }
  }
  
  /**
   * 配置文件中配置为package的时候， 会调用此方法，根据配置的报名去扫描javabean ，
   * 然后自动注册别名，默认会使用 Bean 的首字母小写的非限定类名来作为它的别名
   * 也可在javabean 加上注解@Alias 来自定义别名， 例如： @Alias(user)
   */
  public void registerAliases(String packageName){
    registerAliases(packageName, Object.class);
  }

  public void registerAliases(String packageName, Class<?> superType){
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<Class<?>>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for(Class<?> type : typeSet){
      // Ignore inner classes and interfaces (including package-info.java)
      // Skip also inner classes. See issue #6
      if (!type.isAnonymousClass() 
		&& !type.isInterface() 
		&& !type.isMemberClass()) {
        registerAlias(type);
      }
    }
  }

  public void registerAlias(Class<?> type) {
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
      alias = aliasAnnotation.value();
    } 
    registerAlias(alias, type);
  }

  /**
  * 这就是注册别名的本质方法， 其实就是向保存别名的hashMap新增值而已，
  * 呵呵，别名的实现太简单了，对吧
  */
  public void registerAlias(String alias, Class<?> value) {
    if (alias == null) throw new TypeException("The parameter alias cannot be null");
    String key = alias.toLowerCase(Locale.ENGLISH); // issue #748
    if (TYPE_ALIASES.containsKey(key) 
		&& TYPE_ALIASES.get(key) != null 
		&& !TYPE_ALIASES.get(key).equals(value)) {
      throw new TypeException("The alias '" + alias 
	  + "' is already mapped to the value '" 
	  + TYPE_ALIASES.get(key).getName() + "'.");
    }
    TYPE_ALIASES.put(key, value);
  }

  public void registerAlias(String alias, String value) {
    try {
      registerAlias(alias, Resources.classForName(value));
    } catch (ClassNotFoundException e) {
      throw new TypeException("Error registering type alias "
		+alias+" for "+value+". Cause: " + e, e);
    }
  }
  
  /**
   * 获取保存别名的HashMap, Configuration对象持有对TypeAliasRegistry的引用，
   * 因此，如果需要，我们可以通过Configuration对象获取
   */
  public Map<String, Class<?>> getTypeAliases() {
    return Collections.unmodifiableMap(TYPE_ALIASES);
  }

}
```
由源码可见，设置别名的原理就这么简单，Mybatis默认给我们设置了不少别名，在上面代码中都可以见到。

好啦，本篇内容就是这么简单，到此为止。 下篇将继续讲解还没讲完的配置节点。


---
