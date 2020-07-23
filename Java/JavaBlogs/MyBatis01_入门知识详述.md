# 从一个简单的示例，我们开始进入Mybatis的世界！

[原文链接](https://blog.csdn.net/andamajing/article/details/71202481)


在这篇文章中，我们通过一个简单的Java示例来说明如何使用Mybatis，不必追究细枝末节的东西，只是看看如何去使用而已。

首选，我假定大家用过maven，因为我这里建立的是Maven项目，因为觉得用Maven引用jar包太方便了（发明这个东西的人太有才了）。

接下来我们**需要在pom文件中添加我们需要的jar包**，包含以下几个方面：

（1）mysql的驱动；
（2）mybaits的jar包；
（3）单元测试框架类；
（4）日志框架类；

具体pom文件内容如下所示：

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
 
	<groupId>com.majing.learning</groupId>
	<artifactId>mybatis</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>
 
	<name>mybatis</name>
	<url>http://maven.apache.org</url>
 
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>
 
	<dependencies>
		<!-- 添加mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.4.1</version>
		</dependency>
 
		<!-- 添加mysql驱动 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.40</version>
		</dependency>
 
		<!-- 添加junit -->
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.9</version>
			<scope>test</scope>
		</dependency>
 
		<!-- 添加log4j -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>1.2.16</version>
		</dependency>
	</dependencies>
 
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.0</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```

 引入pom文件后，经过install我们就能看到我们得到了我们想要的各种jar包了。

![](https://img-blog.csdn.net/20170505144109701?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

一切准备就绪，接下来我们开始来真正的写业务部分了。**对于mybatis来说，有一个很重要的配置文件，那就是configuration.xml**，这个文件是我们使用mybatis的核心，后面也将围绕着这个配置文件来说明mybatis的各种功能。

我们先来看下这个配置文件configuration.xml：
```
<?xml version="1.0" encoding="UTF-8"?>
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

 我们先不管配置文件中的细节配置，从配置上我们可以直观的看出我们**对日志、数据库连接地址和驱动都做了相关的配置**。和日志相关的还需要有一个配置文件，用于控制日志往什么地方刷，这个配置文件就是`log4j.properties`，具体配置格式如下：

```
// 首先只有Debug信息会输出，然后输出位置分别为 mybatis 和 stdout(控制台输出)
log4j.rootLogger=DEBUG,mybatis,stdout 
 
log4j.appender.mybatis=org.apache.log4j.DailyRollingFileAppender
log4j.appender.mybatis.layout=org.apache.log4j.PatternLayout
log4j.appender.mybatis.DatePattern='.'yyyy-MM-dd-HH
log4j.appender.mybatis.layout.ConversionPattern=%m%n
log4j.appender.mybatis.file=D:/data/logs/mybatis/mybatis.log
log4j.appender.mybatis.encoding=UTF-8
 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %p [%c] %m%n 
```

 至此，该有的配置文件我们都已经准备好了。

接下来开始写业务代码。这篇文章是一个示例，所以我们就按照最简单的来。我们模拟从数据库里读取一个用户。

我们先来看下数据库（表名为： user）中的数据：
 id | name | password | age | deleteFlag
 ---|---|---|---|---|
1 |zhangsan|admin   |11|0|

我们根据这个数据库表结构，来创建一个相应的POJO实体类User，如下所示:

```
package com.majing.learning.mybatis.entity;
 
public class User {
	private int id;
	private String name;
	private String password;
	private int age;
	private int deleteFlag;
	// 省略以上所有的 get/set 方法
}
```

 接着，我们创建一个接口类，**这个接口类定义我们对数据库表的操作行为**，如下所示，我们定义一个查询方法：

```
package com.majing.learning.mybatis.dao;
import com.majing.learning.mybatis.entity.User;
 
public interface UserDao {
	User findUserById (int userId);
}
```

 有了这些够了吗？NO!

我们还缺少一个很重要的东西，想想就知道，其实我们并**没有告诉mybatis该怎么去查询，也就是这个方法对应什么样的执行语句，我们还需要mapper文件**！！！

我们在UserDao接口所在文件目录创建一个UserDaoMapper.xml的配置文件，配置如下：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="com.majing.learning.mybatis.entity.User">
		select * from user where id = #{id}
	</select>
 
</mapper>
```

 **配置文件中通过id指定了方法和sql语句的对应关系，并且将返回结果指定为User实体**。

至此，一个简单的示例所需要的东西都出来了，我们写个简单的测试类来看下怎么查询。

```
package com.majing.learning.mybatis;
// 省略各种导包
 
public class UserDaoTest {
	
	@Test
	public void findUserById(){
		SqlSession sqlSession = getSessionFactory().openSession();  
		UserDao userMapper = sqlSession.getMapper(UserDao.class);  
		User user = userMapper.findUserById(1);  
		System.out.println(user);
		Assert.assertNotNull("没找到数据", user);
	}
 
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
}
```

 从测试用例代码中，我们看到我们查询一个用户其实只用到了三条语句，也就是下面：

```
SqlSession sqlSession = getSessionFactory().openSession();  
UserDao userMapper = sqlSession.getMapper(UserDao.class);  		
User user = userMapper.findUserById(1);  
```

 是不是比之前繁琐的Java代码好多了，反正我这么觉得。至此，一个入门的简单说明到此为止，后面我们陆续学习Mybatis的一些常用功能。接下来一篇文章会简单说明下mybatis如何实现增删改查基本功能。