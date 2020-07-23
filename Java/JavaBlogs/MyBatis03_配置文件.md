# Mybatis的配置文件入门介绍

[原文地址](https://blog.csdn.net/andamajing/article/details/71405243)

从前面的几篇文章，我们看到了，如何简单的使用Mybatis。从这篇文章开始，我们将从其核心配置文件入手，对Mybatis支持的核心配置文件进行简单详细的描述。

从下面这段代码是我们在使用mybatis前的配置初始化过程，我们通过阅读其源码来逐步了解内部实现原理。

```
// Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互
	private static SqlSessionFactory getSessionFactory() {
		SqlSessionFactory sessionFactory = null;
		String resource = "configuration.xml";
		try {
			sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource));
		} catch (IOException e) {
			e.printStackTrace();
		}
		return sessionFactory;
	}
```

 我们进入到SqlSessionFactoryBuilder类里面，查看其源码：

```
/**
 *    Copyright 2009-2016 the original author or authors.
 */
package org.apache.ibatis.session;
 
import java.io.IOException;
import java.io.InputStream;
import java.io.Reader;
import java.util.Properties;
 
import org.apache.ibatis.builder.xml.XMLConfigBuilder;
import org.apache.ibatis.exceptions.ExceptionFactory;
import org.apache.ibatis.executor.ErrorContext;
import org.apache.ibatis.session.defaults.DefaultSqlSessionFactory;
 
/**
 * Builds {@link SqlSession} instances.
 *
 * @author Clinton Begin
 */
public class SqlSessionFactoryBuilder {
 
  public SqlSessionFactory build(Reader reader) {
    return build(reader, null, null);
  }
 
  public SqlSessionFactory build(Reader reader, String environment) {
    return build(reader, environment, null);
  }
 
  public SqlSessionFactory build(Reader reader, Properties properties) {
    return build(reader, null, properties);
  }
 
  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
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
 
  public SqlSessionFactory build(InputStream inputStream) {
    return build(inputStream, null, null);
  }
 
  public SqlSessionFactory build(InputStream inputStream, String environment) {
    return build(inputStream, environment, null);
  }
 
  public SqlSessionFactory build(InputStream inputStream, Properties properties) {
    return build(inputStream, null, properties);
  }
 
  public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        inputStream.close();
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

 在这个类中，支持多种构造SqlSessionFactory的方法。可以只传入mybatis配置文件，也可以同时传入properties配置文件替代mybatis配置文件中的<properties>元素标签，另外也支持传入环境参数envirmont参数。

我们跟随着源码继续往下看：

```
 public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
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
```

**这里创建了一个XMLConfigBuilder类实例，通过他来对mybatis配置文件（一个xml配置文件）进行解析**。解析的代码入口如下所示：

```
public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
 
  private void parseConfiguration(XNode root) {
    try {
      Properties settings = settingsAsPropertiess(root.evalNode("settings"));
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      loadCustomVfs(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

 从这里看出，**配置文件是以configuration为根节点，在根节点之下有多个子节点，它们分别为：settings、properties、typeAliases、plugins、objectFactory、objectWrapperFactory、environments、databaseIdProvider、typeHandlers、mappers**。