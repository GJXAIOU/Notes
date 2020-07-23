# Mybatis配置之属性配置元素详述
[原文链接](https://blog.csdn.net/andamajing/article/details/71441028)

紧接着上篇博客 Mybatis 的配置文件入门介绍，我们开始对mybatis核心配置文件中的各个元素进行详细的说明，在这篇文章中，我们首先来看下<properties>元素，这个元素从上篇文章中可以看到是最先被解析的，设置的属性值将会被其他元素所使用。

我们先将**之前的配置文件**在这里拷贝一份，以便对比观察，如下所示：
```
<!DOCTYPE configuration    
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"    
    "http://mybatis.org/dtd/mybatis-3-config.dtd">  

<configuration>  
  
    <settings>  
        <setting name="logImpl" value="LOG4J" />  
    </settings>  
  
    <!-- 和spring整合后 environments配置将废除 -->  
    <environments default="development">  
        <environment id="development">  
            <!-- 使用jdbc事务管理 -->  
            <transactionManager type="JDBC" />  
            <!-- 数据库连接池 -->  
            <dataSource type="POOLED">  
                <property name="driver" value="com.mysql.jdbc.Driver" />  
                <property name="url"  
                    value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8" />  
                <property name="username" value="root" />  
                <property name="password" value="root" />  
            </dataSource>  
        </environment>  
    </environments>  
  
    <mappers>  
        <mapper resource="com\majing\learning\mybatis\dao\UserDaoMapper.xml" />  
    </mappers>  
  
</configuration>
```

属性值有三种方式书写，接下来我们一个一个的看。

（1）**通过<properties>元素里面配置<property>元素**；
之前的配置文件中<dataSource>元素中设置了数据库的驱动、连接字符串还有账号密码等信息，但是我们这里不想这么弄，通过设置<property>来进行设置，如下所示：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
	<properties>
		<property name="username" value="root"/>
		<property name="password" value="root"/>
		<property name="driver" value="com.mysql.jdbc.Driver"/>
		<property name="url" value="jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8"/>
	</properties>
 
	<settings>
		<setting name="logImpl" value="LOG4J" />
	</settings>
 
	<!-- 和spring整合后 environments配置将废除 -->
	<environments default="development">
		<environment id="development">
			<!-- 使用jdbc事务管理 -->
			<transactionManager type="JDBC" />
			<!-- 数据库连接池 -->
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
	</environments>
 
	<mappers>
		<mapper resource="com\majing\learning\mybatis\dao\UserDaoMapper.xml" />
	</mappers>
 
</configuration>
```

 这样，我们就在需要配置的地方统一到了<properties>元素中，便于统一管理。

（2）通过<properties>元素的resource属性或者url属性进行配置；
**相当于将数据库配置存在单独的 .properties 文件中**，然后在这里直接调用；
这里我们不用<property>标签元素进行设置，而是**使用属性配置文件的方式**。如下所示：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
	<properties resource="mysql.properties">
	</properties>
 
	<settings>
		<setting name="logImpl" value="LOG4J" />
	</settings>
 
	<!-- 和spring整合后 environments配置将废除 -->
	<environments default="development">
		<environment id="development">
			<!-- 使用jdbc事务管理 -->
			<transactionManager type="JDBC" />
			<!-- 数据库连接池 -->
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
	</environments>
 
	<mappers>
		<mapper resource="com\majing\learning\mybatis\dao\UserDaoMapper.xml" />
	</mappers>
 
</configuration>
```

 而<properties>标签元素的resource属性设置的mysql.properties（相对于根目录的路径）内容如下所示：
```properties
driver = com.mysql.jdbc.Driver
url = jdbc:mysql://localhost:3306/mybatis?characterEncoding = utf-8
username = root
password = GJXAIOU
```
**使用配置文件的方式，可以使得一次配置在多个地方重复使用**

（3）通过在初始化的时候，以代码的方式传入Properties类实例；

具体如下所示：

```
package com.majing.learning.mybatis;
 

public class UserDaoTest1 extends TestCase{
	
	@Test
	public void testFindUserById(){
		SqlSession sqlSession = getSessionFactory().openSession(true);  
		UserDao userMapper = sqlSession.getMapper(UserDao.class);  
 
		User user = userMapper.findUserById(10);  
		System.out.println("记录为："+user);
	}
 
	// Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互
	private static SqlSessionFactory getSessionFactory() {
		SqlSessionFactory sessionFactory = null;
		String resource = "configuration.xml";
		try {
			sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource), buildInitProperties());
		} catch (IOException e) {
			e.printStackTrace();
		}
		return sessionFactory;
	}
	
	private static Properties buildInitProperties(){
		Properties properties = new Properties();
		properties.put("driver", "com.mysql.jdbc.Driver");
		properties.put("url", "jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf-8");
		properties.put("username", "root");
		properties.put("password", "root");
		return properties;
	}
}
```

 从上可以看出，在创建SqlSessionFactory的时候，人为写代码传入了一套属性配置。

上面三种方式都可以实现相同的功能，那就是给mybatis初始化的时候设置一系列的属性值以供使用。但是这三者又有什么区别呢？

通过查看源码，一个直观的感觉就是**这三种配置是有优先级关系的且不同方式配置的配置项是可以并存的，优先级次序如下：第三种方式>第二种方式>第一种方式**。即如果三种方式都配置了同一个配置项，那么优先级高的配置方式的配置值生效。这主要还是因为mybats的源码解析过程导致的。下面我们看下具体的解析逻辑：

```
 private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
      Properties defaults = context.getChildrenAsProperties();
      String resource = context.getStringAttribute("resource");
      String url = context.getStringAttribute("url");
      if (resource != null && url != null) {
        throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
      }
      if (resource != null) {
        defaults.putAll(Resources.getResourceAsProperties(resource));
      } else if (url != null) {
        defaults.putAll(Resources.getUrlAsProperties(url));
      }
      Properties vars = configuration.getVariables();
      if (vars != null) {
        defaults.putAll(vars);
      }
      parser.setVariables(defaults);
      configuration.setVariables(defaults);
    }
  }
```

 从代码看，<properties>元素的resource属性和url属性是不能同时设置的，否则会报异常。同时，解析的时候是先解析的<property>标签元素，而后从resource或者url指定的配置文件开始读取配置，如果之前有了相同的配置项则进行覆盖，如果没有则进行添加。在这之后，开始判断是否有第三种方式的属性配置，如果有，则将相关配置添加到之前的属性集合中，如果存在同名的配置也进行覆盖。这样的逻辑也是导致为什么会有优先级的直接原因。