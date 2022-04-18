# DTD 简介

*   [DTD 教程](https://www.w3school.com.cn/dtd/index.asp "DTD 教程")
*   [DTD 构建模块](https://www.w3school.com.cn/dtd/dtd_building.asp "DTD - XML 构建模块")

**文档类型定义（DTD）可定义合法的XML文档构建模块。它使用一系列合法的元素来定义文档的结构。**

**DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。**

## 内部的 DOCTYPE 声明

假如 DTD 被包含在您的 XML 源文件中，它应当通过下面的语法包装在一个 DOCTYPE 声明中：

`<!DOCTYPE 根元素 [元素声明]>`

带有 DTD 的 XML 文档实例（请在 IE5 以及更高的版本打开，并选择查看源代码）：

```xml
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>
```

[在您的浏览器中打开此 XML 文件，并选择“查看源代码”命令](https://www.w3school.com.cn/dtd/note_in_dtd.xml)。

### 以上 DTD 解释如下：

_!DOCTYPE note_ (第二行)定义此文档是 _note_ 类型的文档。

_!ELEMENT note_ (第三行)定义 _note_ 元素有四个元素："to、from、heading,、body"

_!ELEMENT to_ (第四行)定义 _to_ 元素为 "#PCDATA" 类型

_!ELEMENT from_ (第五行)定义 _from_ 元素为 "#PCDATA" 类型

_!ELEMENT heading_ (第六行)定义 _heading_ 元素为 "#PCDATA" 类型

_!ELEMENT body_ (第七行)定义 _body_ 元素为 "#PCDATA" 类型

## 外部文档声明

假如 DTD 位于 XML 源文件的外部，那么它应通过下面的语法被封装在一个 DOCTYPE 定义中：

<!DOCTYPE 根元素 SYSTEM "文件名">


这个 XML 文档和上面的 XML 文档相同，但是拥有一个外部的 DTD: （[在 IE5 中打开](https://www.w3school.com.cn/dtd/note_ex_dtd.xml)，并选择“查看源代码”命令。）

<?xml version="1.0"?>

<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note> 


这是包含 DTD 的 "note.dtd" 文件：

<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>

## 为什么使用 DTD？

通过 DTD，您的每一个 XML 文件均可携带一个有关其自身格式的描述。

通过 DTD，独立的团体可一致地使用某个标准的 DTD 来交换数据。

而您的应用程序也可使用某个标准的 DTD 来验证从外部接收到的数据。

您还可以使用 DTD 来验证您自身的数据。







# DTD和DOCTYPE的作用

[原文链接](https://blog.csdn.net/caomiao2006/article/details/9223421)
一直以来写网页，不论用 Adobe（以前是 Macromedia）的 DW，还是 Editplus 自动生成的初始网页，头部都会加上类似的一句话：

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0Transitional//EN""http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 而一直以来本人对他的态度也是忽略的，以为这个东西没什么作用。今天知道他原来在HTML中是很有作用的。


**DTD声明是什么意思呢？**

DTD 意为 Document Type Definition（文档类型定义），先撇开 DTD 文件的具体内容不谈。我们看到 HTML 文档中的 DTD 声明始终以!DOCTYPE 开头，空一格后跟着文档根元素的名称（网页也就是 HTML）。如果是内部 DTD，则再空一格出现[]，在中括号中是文档类型定义的内容。而对于外部 DTD，则又分为私有 DTD 与公共 DTD，私有 DTD 使用 SYSTEM 表示，接着是外部 DTD 的 URL。而公共 DTD 则使用 PUBLIC，接着是 DTD 公共名称，接着是 DTD 的 URL。下面是一些示例。

公共 DTD，DTD 名称格式为“注册//组织//类型 标签//语言”,“注册”指示组织是否由国际标准化组织(ISO)注册，+表示是，-表示不是。“组织”即组织名称，如：W3C；“类型”一般是 DTD，“标签”是指定公开文本描述，即对所引用的公开文本的唯一描述性名称，后面可附带版本号。最后“语言”是 DTD 语言的 ISO 639 语言标识符，如：EN 表示英文，ZH 表示中文。

本文开头的 DTD 也就是 XHTML 1.0 Transitional 的 DTD。以!DOCTYPE 开始，html 是文档根元素名称，PUBLIC 表示是公共 DTD，后面是 DTD 名称，以-开头表示是非 ISO 组织，组织名称是 W3C，文档类型是 XHTML，版本号 1.0 过渡类型，EN 表示 DTD 语言是英语，最后是 DTD 的 URL。

注意：虽然 DTD 的文件 URL 可以使用相对 URL 也可以使用绝对 URL，但推荐标准是使用绝对 URL。另一方面，对于公共 DTD，如果解释器能够识别其名称，则不去查看 URL 上的 DTD 文件。

**下面来说说DTD的作用。**

<P align="center">这是一个居中段落</P>

在 XHTML 中，标记是区分大小写的，上面的代码毫无意义。可在 HTML 中它是一个居中段落。浏览器是怎样处理这种情况呢？难道浏览器认为你写的是 HTML，然后把它作为一个一个居中段落显示？如是你写的是 XHTML 呢，它将是一段不可显示的代码！浏览器是怎样知道你用的是什么标记语言然后正确对待这段代码呢？

这就是 DTD 的工作了。一个 DTD 应该放在每一个文档的第一行。这样正确地放置，你的 DTD 才能告诉浏览器的用的是什么标记语言。在通常情况下，如果你编写的是正确代码，并拥有一个合适的 DTD，浏览器将会根据 W3C 的标准显示你的代码。

如果说你没有使用 DTD，你将很难预测浏览器是怎样显示你的代码，仅仅在同一浏览器就有不同的显示效果。尽管你的网页做得非常飘亮，要是没有使用 DTD，你的努力也是白费的。因此，一个 DTD 是必不可少的。

**XHTML**较为规范，他的 DTD 分为几种：Strick、Transitional 和 Frameset

XHTML1.0 Strict DTD（严格的文档类定义）：要求严格的 DTD，你不能使用表现标识和属性，和 CSS 一同使用。完整代码如下：

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">


XHTML1.0 Transitional DTD（过渡的文档类定义）：要求非常宽松的 DTD，它允许你继续使用 HTML4.01 的标识(但是要符合 xhtml 的写法)。完整代码如下：

<!DOCTYPE html  PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">


XHTML1.0 Frameset DTD（框架集文档类定义）:专门针对框架页面设计使用的 DTD，如果你的页面中包含有框架，需要采用这种 DTD。完整代码如下：    

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">


**HTML**的语法就非常宽松了，他的 DTD 也分为一样的三种。

HTML 4.01 Strict DTD （严格的文档类定义）不能包含已过时的元素（或属性）和框架元素。对于使用了这类 DTD 的文档，使用如下文档声明：

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">


HTML 4.01 Transitional DTD（过渡的文档类定义）能包含已过时的元素和属性但不能包含框架元素。对于使用了这类 DTD 的文档，使用如下文档声明：    

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"  "http://www.w3.org/TR/html4/loose.dtd">


HTML 4.01 Frameset DTD（框架集文档类定义）。能包含已过时的元素和框架元素。对于使用了这类 DTD 的文档，使用如下文档声明：

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN"  "http://www.w3.org/TR/html4/frameset.dtd">


更早的 HTML 版本：HTML 3.2   

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">  


更早的 HTML 版本：HTML 2.0    

<!DOCTYPE html PUBLIC "-//IETF//DTD HTML 2.0//EN">


**我们选择什么样的DOCTYPE**

理想情况当然是严格的 DTD，但对于我们大多数刚接触 web 标准的设计师来说，过渡的 DTD 是目前理想选择。因为这种 DTD 还允许我们使用表现层的标识、元素和属性，也比较容易通过 W3C 的代码校验。

上面说的“表现层的标识、属性”是指那些纯粹用来控制表现的 tag，例如用于排版的表格（<table>标签）、换行（<br>标签）和背景颜色标识等。在 XHTML 中标识是用来表示结构的，而不是用来实现表现形式，我们过渡的目的是最终实现数据和表现相分离。

[http://blog.sina.com.cn/s/blog_44dd2a630100pc6y.html](http://blog.sina.com.cn/s/blog_44dd2a630100pc6y.html)



# 从一个简单的示例，我们开始进入Mybatis的世界！

[原文链接](https://blog.csdn.net/andamajing/article/details/71202481)


在这篇文章中，我们通过一个简单的 Java 示例来说明如何使用 Mybatis，不必追究细枝末节的东西，只是看看如何去使用而已。

首选，我假定大家用过 maven，因为我这里建立的是 Maven 项目，因为觉得用 Maven 引用 jar 包太方便了（发明这个东西的人太有才了）。

接下来我们**需要在pom文件中添加我们需要的jar包**，包含以下几个方面：

（1）mysql 的驱动；
（2）mybaits 的 jar 包；
（3）单元测试框架类；
（4）日志框架类；

具体 pom 文件内容如下所示：

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

 引入 pom 文件后，经过 install 我们就能看到我们得到了我们想要的各种 jar 包了。

![](https://img-blog.csdn.net/20170505144109701?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

一切准备就绪，接下来我们开始来真正的写业务部分了。**对于mybatis来说，有一个很重要的配置文件，那就是configuration.xml**，这个文件是我们使用 mybatis 的核心，后面也将围绕着这个配置文件来说明 mybatis 的各种功能。

我们先来看下这个配置文件 configuration.xml：

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

| id   | name     | password | age  | deleteFlag |
| ---- | -------- | -------- | ---- | ---------- |
| 1    | zhangsan | admin    | 11   | 0          |

我们根据这个数据库表结构，来创建一个相应的 POJO 实体类 User，如下所示:

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

我们在 UserDao 接口所在文件目录创建一个 UserDaoMapper.xml 的配置文件，配置如下：

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

 是不是比之前繁琐的 Java 代码好多了，反正我这么觉得。至此，一个入门的简单说明到此为止，后面我们陆续学习 Mybatis 的一些常用功能。接下来一篇文章会简单说明下 mybatis 如何实现增删改查基本功能。



# 通过Mybatis实现数据的增删改查（CRUD）操作

[原文链接](https://blog.csdn.net/andamajing/article/details/71216067)

在上一篇文章中，我们应该算是简单的进入了 MyBatis 的世界，在这篇文章中，我们从简单的增删改查说起，毕竟对于数据库操作来说，这几种操作是肯定逃不掉的。

在这篇文章中，我们不在对所有需要的东西全部列举出来，而是在上一篇文章的基础上进行修改。

首先，我们需要修改的就是接口类，提供增删改查的方法，如下所示：

```
package com.majing.learning.mybatis.dao;
import com.majing.learning.mybatis.entity.User;
 
public interface UserDao {
	/**
	 * 查询
	 */
	User findUserById (int userId);
	/**
	 * 增加
	 */
	void addUser(User user);
	/**
	 * 删除
	 */
	void deleteUser(int userId);
	/**
	 * 更新
	 */
	void updateUser(User user);
}
```

 紧接着，当然我们需要修改 UserDaoMapper.xml 文件，毕竟我们**要为每个方法提供对应的sql实现**。下面给出调整后的配置文件：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">

<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
	<select id="findUserById" resultType="com.majing.learning.mybatis.entity.User">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="com.majing.learning.mybatis.entity.User" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="com.majing.learning.mybatis.entity.User">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

 至此，我们所需要修改的东西应该就没了，下面我们来写测试用例，先插入一条记录，然后更新这条记录，然后再查询之前的记录，最后删除这条记录。测试代码如下：

```
package com.majing.learning.mybatis;
  
public class UserDaoTest {
	
	@Test
	public void findUserById(){
		SqlSession sqlSession = getSessionFactory().openSession();  
		UserDao userMapper = sqlSession.getMapper(UserDao.class);  
		
		//增加一条新记录
		User user = new User();
		user.setName("majing");
		user.setPassword("19880101");
		user.setAge(29);
		userMapper.addUser(user);
		
		//更新该记录
		user.setName("new_majing");
		userMapper.updateUser(user);
		
		
		//查询该记录
		int id = user.getId();
		user = null;
		user = userMapper.findUserById(id);  
		System.out.println("更新后记录为："+user);
		
		//删除记录
		System.out.println("尝试删除该记录...");
		userMapper.deleteUser(id);
		
		user = userMapper.findUserById(id);  
		if(user==null){
			System.out.println("该记录已删除！");
		}else{
			System.out.println("该记录未被成功删除！");
		}
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

 如果我们执行这个单元测试类，会得到如下的输出，一切都是这么正常，So easy：

![](https://img-blog.csdn.net/20170505172105717?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

但是
如果单步调试的话，我们会发现虽然每一步都按照我们的意愿进行了操作（增删改查），但是目标数据库中却始终一条记录都没有！！！数据根本没有插入到数据库中。

其实我们在获取数据库连接的时候采用了 MyBatis 的默认配置，即不会自动提交。所以这里我们需要设置一下，在获取连接时设置 sql 语句是自动提交的。如下所示：

```
SqlSession sqlSession = getSessionFactory().openSession(true);  
```

 这下就一切正常了，不信的可以试试。这也告诉我们一个道理，其实看别人写的东西都很简单，但是其实往往看到的和自己动手操作的往往自己得到的并不一样，只有自己动手写过才能更清晰的知道其中的知识，也掌握的更牢固。这也是我为什么学什么都自己尝试一遍的原因。



# Mybatis的配置文件入门介绍

[原文地址](https://blog.csdn.net/andamajing/article/details/71405243)

从前面的几篇文章，我们看到了，如何简单的使用 Mybatis。从这篇文章开始，我们将从其核心配置文件入手，对 Mybatis 支持的核心配置文件进行简单详细的描述。

从下面这段代码是我们在使用 mybatis 前的配置初始化过程，我们通过阅读其源码来逐步了解内部实现原理。

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

 我们进入到 SqlSessionFactoryBuilder 类里面，查看其源码：

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

 在这个类中，支持多种构造 SqlSessionFactory 的方法。可以只传入 mybatis 配置文件，也可以同时传入 properties 配置文件替代 mybatis 配置文件中的<properties>元素标签，另外也支持传入环境参数 envirmont 参数。

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



# Mybatis配置之属性配置元素详述

[原文链接](https://blog.csdn.net/andamajing/article/details/71441028)

紧接着上篇博客 Mybatis 的配置文件入门介绍，我们开始对 mybatis 核心配置文件中的各个元素进行详细的说明，在这篇文章中，我们首先来看下<properties>元素，这个元素从上篇文章中可以看到是最先被解析的，设置的属性值将会被其他元素所使用。

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

（2）通过<properties>元素的 resource 属性或者 url 属性进行配置；
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

 而<properties>标签元素的 resource 属性设置的 mysql.properties（相对于根目录的路径）内容如下所示：

```properties
driver = com.mysql.jdbc.Driver
url = jdbc:mysql://localhost:3306/mybatis?characterEncoding = utf-8
username = root
password = GJXAIOU
```

**使用配置文件的方式，可以使得一次配置在多个地方重复使用**

（3）通过在初始化的时候，以代码的方式传入 Properties 类实例；

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

 从上可以看出，在创建 SqlSessionFactory 的时候，人为写代码传入了一套属性配置。

上面三种方式都可以实现相同的功能，那就是给 mybatis 初始化的时候设置一系列的属性值以供使用。但是这三者又有什么区别呢？

通过查看源码，一个直观的感觉就是**这三种配置是有优先级关系的且不同方式配置的配置项是可以并存的，优先级次序如下：第三种方式>第二种方式>第一种方式**。即如果三种方式都配置了同一个配置项，那么优先级高的配置方式的配置值生效。这主要还是因为 mybats 的源码解析过程导致的。下面我们看下具体的解析逻辑：

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

 从代码看，<properties>元素的 resource 属性和 url 属性是不能同时设置的，否则会报异常。同时，解析的时候是先解析的<property>标签元素，而后从 resource 或者 url 指定的配置文件开始读取配置，如果之前有了相同的配置项则进行覆盖，如果没有则进行添加。在这之后，开始判断是否有第三种方式的属性配置，如果有，则将相关配置添加到之前的属性集合中，如果存在同名的配置也进行覆盖。这样的逻辑也是导致为什么会有优先级的直接原因。

# Mybatis配置之别名配置元素详述

[原文链接](https://blog.csdn.net/andamajing/article/details/71503263)

在前面的文章 Mybatis 配置之<properties>属性配置元素详述，我们讲述了<properties>标签元素的配置和使用方法。在这篇文章中，我们来说说**<typeAliases>标签元素，这个元素主要是用于对类型进行别名控制**，具体什么意思呢？我们下面用一个示例说明，看了之后我相信你就会明白了。

这里我们贴出之前的 UserDao 对应的 mapper 文件，如下所示：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="com.majing.learning.mybatis.entity.User">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="com.majing.learning.mybatis.entity.User" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="com.majing.learning.mybatis.entity.User">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

 从这个配置文件中，我们可以看到<select>、<insert>和<update>三个标签元素的 resultType 都是 User 对象，需要设置这个 User 对象的类全限定名，即 packname.classname。

我们发现一个问题，那就是这个类名，我们需要写多次，如果要改这个类名的话，我们需要在多个地方进行修改。很明显，这样配置的话很容易造成修改上的遗漏，同时也书写上也比较麻烦。因此，**MyBatis为我们提供了一个简单方便的配置方法，那就是使用<typeAliases>标签元素，给实体类设置一个别名**。

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
	
	<typeAliases>
 		<typeAlias alias="User" type="com.majing.learning.mybatis.entity.User"/> 
	</typeAliases>
      
	<!---省略其他配置---->
 
</configuration>
```

如上所示，我们**在原来的mybatis配置文件中增加了<typeAliases>标签，并将com.majing.learning.mybatis.entity.User这个实体类重命名为User**，然后我们在 mapper 配置文件中就可以如下使用了。

备注：这里需要注意的是，**typeAliases配置需要放置在settings之后**，否则会出异常！！！ 

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="User">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="User" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="User">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

 这样即使实体类名修改了，所需要修改的地方也只有一处，便于集中管理。

也许你会有疑问，如果实体类比较多怎么办？还不是要配置很多实体类和别名，NO,NO,NO！下面跟大家说说另一种配置方法。

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
	
	<typeAliases>
		<package name="com.majing.learning.mybatis.entity"/>
	</typeAliases>
 
	<!---省略其他配置---->
 
</configuration>
```

 在这里，**我们不再使用<typeAliases>标签下<typeAliase>，而是使用<package>标签，表示扫描该包名下的所有类（除了接口和匿名内部类），如果类名上有注解，则使用注解指定的名称作为别名，如果没有则使用类名首字母小写作为别名，如`com.majing.learning.mybatis.entity.User`这个类如果没有设置@Alias注解，则此时会被关联到user这个别名上**。

因此，按照上面的配置，我们还需要将实体类做一下调整，如下两种方式所示：

（1）给实体类添加@Alias 注解 

```
package com.majing.learning.mybatis.entity;
 
import org.apache.ibatis.type.Alias;
 
@Alias(value="User")
public class User {
// 具体类的内容
	
}
```

 （2）实体类不加注解的情况下，修改 mapper 文件中引用的类型别名，改为小写，如下所示：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="user">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="user" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="user">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

最后想说，mybatis 为我们已经实现了很多别名，已经为许多常见的 Java 类型内建了相应的类型别名。它们都是大小写不敏感的，需要注意的是由基本类型名称重复导致的特殊处理。

| 别名       | 映射的类型 |
| ---------- | ---------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

  至此，关于别名的全部使用方法这里便介绍完成了。

# Mybatis配置之配置元素详述

[原文链接](https://blog.csdn.net/andamajing/article/details/71616712)

在这篇文章中，我们接着前文继续往下看其他的配置元素，今天的主角就是我们的**<environments>元素，该元素用于对我们需要访问的数据库配置进行设置**，我们先来看一下配置：

```
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
```

 从上面看，我们知道<environments>下面可以配置多个<environment>元素节点，而**每个<environment>节点我们可以配置两个东西，一个是事务管理器配置<transactionManager>，另一个是数据源配置<dataSource>**。

我们先从源码开始看起，看看这块是怎么解析的，然后再具体看里面都要配置什么哪些参数。

还是从解析的入口开始看起：

![](https://img-blog.csdn.net/20170511123828382?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

进入方法内部：

![](https://img-blog.csdn.net/20170511124034429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

从代码看，就是**首先获取<environments>标签元素的default属性，这个属性作用就是指定当前情况下使用哪个数据库配置，也就是使用哪个<environment>节点的配置，default的值就是配置的<environment>标签元素的id值**。

正如上面代码中 isSpecifiedEnvironment(id)方法一样，在遍历所有<environment>的时候一次判断相应的 id 是否是 default 设置的值，如果是，则使用当前<environment>元素进行数据库连接的初始化。

isSpecifiedEnvironment 方法如下所示：

![](https://img-blog.csdn.net/20170511124525058?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

紧接着，下面的代码就是用设置的事务管理器和数据源构造相应的对象了。

```
TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
DataSource dataSource = dsFactory.getDataSource();
Environment.Builder environmentBuilder = new Environment.Builder(id)
              .transactionFactory(txFactory)
              .dataSource(dataSource);
configuration.setEnvironment(environmentBuilder.build());
```


## 事务管理器

 我们首先从事务管理器的解析开始，进入到 transactionManagerElement()方法内：

```
private TransactionFactory transactionManagerElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      Properties props = context.getChildrenAsProperties();
      TransactionFactory factory = (TransactionFactory) resolveClass(type).newInstance();
      factory.setProperties(props);
      return factory;
    }
    throw new BuilderException("Environment declaration requires a TransactionFactory.");
  }
```

 我们可以看到，这里其实是**根据<transactionManager>这个元素的type属性来找相应的事务管理器的**。

在**Mybatis 里面支持两种配置：JDBC 和 MANAGED**。这里根据 type 的设置值来返回相应的事务管理器。我们看下，在 Mybatis 的 Configuration 类中，已经将这两种配置及对应的事务管理器做了某种关联，如下所示：

```
public Configuration() {
    typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
    typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
 
    typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
    typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
    typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
 
    typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
    typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
    typeAliasRegistry.registerAlias("LRU", LruCache.class);
    typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
    typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
 
    typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);
 
    typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
    typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);
 
    typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
    typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
    typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
    typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
    typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
    typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
    typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);
 
    typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
    typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);
 
    languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
    languageRegistry.register(RawLanguageDriver.class);
  }
```

 这两种事务管理器的区别：

JDBC：这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。

MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如:

```
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```

 备注：如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器， 因为 **Spring 模块会使用自带的管理器来覆盖前面的配置**。


## 数据源配置

说完了事务管理器，紧接着，我们来看看数据源配置。

**dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源**。Mybatis 支持三种内建的数据源类型，分别是 UNPOOLED、POOLED 和 JNDI，即我们在配置<dataSource>元素的 type 属性时，我们可以直接支持设置这三个值。

下面分别对这三种类型做一个简单的说明：

- （1）UNPOOLED
    这个数据源的实现只是每次被请求时打开和关闭连接。虽然一点慢，它对在及时可用连接方面没有性能要求的简单应用程序是一个很好的选择。 不同的数据库在这方面表现也是不一样的，所以对某些数据库来说使用连接池并不重要，这个配置也是理想的。
    - UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：
        * driver ： 这是 JDBC 驱动的 Java 类的完全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
        * url ：这是数据库的 JDBC URL 地址。
        * username ： 登录数据库的用户名。
        * password ：登录数据库的密码。
        * defaultTransactionIsolationLevel ： 默认的连接事务隔离级别。 

作为可选项，你也可以传递属性给数据库驱动。要这样做，属性的前缀为“driver.”，例如：`driver.encoding=UTF8`

这将通过 DriverManager.getConnection(url,driverProperties)方法传递值为 UTF8 的 encoding 属性给数据库驱动。

- （2）POOLED  
    这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。

    - 除了上述提到 UNPOOLED 下的属性外，会有更多属性用来配置 POOLED 的数据源： 

        * poolMaximumActiveConnections ： 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10 
        * poolMaximumIdleConnections ：任意时间可能存在的空闲连接数。 
        * poolMaximumCheckoutTime ：在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒） 
        * poolTimeToWait ：这是一个底层设置，如果获取连接花费的相当长的时间，它会给连接池打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒）。 
        * poolPingQuery ： 发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。 
        * poolPingEnabled ： 是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值：false。

        -  poolPingConnectionsNotUsedFor ： 配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。 

- （3）JNDI
    这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。

    - 这种数据源配置只需要两个属性：
        - `initial_context` ： 这个属性用来在 InitialContext 中寻找上下文（即，`initialContext.lookup(initial_context)）`。这是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。
        - `data_source` ： 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给初始上下文。比如：`env.encoding=UTF8`
这就会在初始上下文（InitialContext）实例化时往它的构造方法传递值为 UTF8 的 encoding 属性。

至此，关于<environments>元素的相关配置使用便介绍完毕了。



# Mybatis中的数据源与连接池详解

[原文链接](https://blog.csdn.net/andamajing/article/details/71715846)

在前面的文章 Mybatis 配置之 **<environments> 配置元素详述中我们已经知道里面可以配置两个元素，一个是数据源及连接池的配置，一个是事务管理器的配置**。在上篇文章中我们只是简单的描述了一下，从这篇文章开始，我们将分两篇博文，分别对这两个问题进行详细说明。

这篇文章我们先来了解一下数据源及连接池的配置。

（1）Mybatis 中支持的数据源  

在上篇文章中，我们知道 Mybatis 中支持三种形式数据源的配置，分别为：UNPOOLED、POOLED 和 JNDI，如下红色区域所示： 

![](https://img-blog.csdn.net/20170512124019425?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

在 Mybatis 内部定义了一个接口 DataSourceFactory，而支持的三种形式都需要实现这个接口。DataSourceFactory 接口定义如下：

```
package org.apache.ibatis.datasource; import java.util.Properties;import javax.sql.DataSource; /** * @author Clinton Begin */public interface DataSourceFactory {   void setProperties(Properties props);   DataSource getDataSource(); }
```

 与 UNPOOLED、POOLED 和 JNDI 相对应的，在 mybatis 内部定义实现了 DataSourceFactory 接口的三个类，分别为 UnpooledDataSourceFactory、PooledDataSourceFactory 和 JndiDataSourceFactory。

具体结构如下所示：

![](https://img-blog.csdn.net/20170512125841268?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

与这些数据源工厂类相对应的也定义了相应的数据源对象，其中 UnpooledDataSourceFactory 和 PooledDataSourceFactory 工厂返回的分别是 UnpooledDataSource 和 PooledDataSource，而 JndiDataSourceFactory 则是根据配置返回相应的数据源。  

![](https://img-blog.csdn.net/20170512125100546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

（2）mybatis 中数据源的创建过程

首先从配置文件开始看起：

```
<!-- 数据库连接池 -->			<dataSource type="POOLED">				<property name="driver" value="${driver}" />				<property name="url" value="${url}" />				<property name="username" value="${username}" />				<property name="password" value="${password}" />			</dataSource>
```

（a）在 mybatis 初始化的时候，在解析到<dataSource>节点时，会根据相应的 type 类型设置来创建相应的数据源工厂类实例，如下所示：

```
DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
```

```
private DataSourceFactory dataSourceElement(XNode context) throws Exception {    if (context != null) {      String type = context.getStringAttribute("type");      Properties props = context.getChildrenAsProperties();      DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();      factory.setProperties(props);      return factory;    }    throw new BuilderException("Environment declaration requires a DataSourceFactory.");  }
```

在上面代码里，根据 type 类型去寻找相应的数据源工厂类并实例化一个。具体每一个配置对应什么类，在 Configuration 类中已经进行了声明，如下所示： 

```
typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
```

（b）之后，从数据源工厂类实例中通过 getDataSource()方法获取一个 DataSource 对象；

（c）MyBatis 创建了 DataSource 实例后，会将其放到 Configuration 对象内的 Environment 对象中， 供以后使用。如下所示：

```
DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));          DataSource dataSource = dsFactory.getDataSource();          Environment.Builder environmentBuilder = new Environment.Builder(id)              .transactionFactory(txFactory)              .dataSource(dataSource);          configuration.setEnvironment(environmentBuilder.build());
```

 （3）数据源 DataSource 对象什么时候创建数据库连接

当我们需要创建 SqlSession 对象并需要执行 SQL 语句时，这时候 MyBatis 才会去调用 dataSource 对象来创建 java.sql.Connection 对象。也就是说，java.sql.Connection 对象的创建一直延迟到执行 SQL 语句的时候。  

```
public void testFindUserById(){		SqlSession sqlSession = getSessionFactory().openSession(true);  		UserDao userMapper = sqlSession.getMapper(UserDao.class);   		User user = userMapper.findUserById(10);  		System.out.println("记录为："+user);	}
```

 对于上面这段代码，我们通过调试会发现，在前两句的时候其实是没有创建数据库连接的，而是在执行 userMapper.findUserById()方法的时候才触发了数据库连接的创建。

（4）非池化的数据源 UnpooledDataSource 

我们先直接从代码入手：

```
@Override  public Connection getConnection() throws SQLException {    return doGetConnection(username, password);  }
```

```
private Connection doGetConnection(String username, String password) throws SQLException {    Properties props = new Properties();    if (driverProperties != null) {      props.putAll(driverProperties);    }    if (username != null) {      props.setProperty("user", username);    }    if (password != null) {      props.setProperty("password", password);    }    return doGetConnection(props);  }
```

```
private Connection doGetConnection(Properties properties) throws SQLException {    initializeDriver();    Connection connection = DriverManager.getConnection(url, properties);    configureConnection(connection);    return connection;  }
```

 从上面的代码可以知道 UnpooledDataSource 创建数据库连接的主要流程，具体时序图如下所示：

（a）调用 initializeDriver()方法进行驱动的初始化；

判断 driver 驱动是否已经加载到内存中，如果还没有加载，则会动态地加载 driver 类，并实例化一个 Driver 对象，使用 DriverManager.registerDriver()方法将其注册到内存中，以供后续使用。 

（b）调用 DriverManager.getConnection()获取数据库连接；

（c）对数据库连接进行一些设置，并返回数据库连接 Connection; 

设置数据库连接是否自动提交，设置事务级别等。

![](https://img-blog.csdn.net/20170512164355919?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

有人可能会有疑问，这里的 username 和 password 是什么传递给数据源的呢？

这个问题其实上面已经提到过了，在 mybatis 初始化的时候，就已经解析了<dataSource>元素，并将其下相关的<property>配置作为数据源的配置初始化进去了。也就是下面这段逻辑：

```
Properties props = context.getChildrenAsProperties();      DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();      factory.setProperties(props);
```

 至此，对于 UnpooledDataSource 数据源算是有比较清楚的了解了。下面我们看看带连接池的 PooledDataSource

（5）带连接池的 PooledDataSource 

为什么要使用带连接池的数据源呢，最根本的原因还是因为每次创建连接开销比较大，频繁的创建和关闭数据库连接将会严重的影响性能。因此，常用的做法是维护一个数据库连接池，每次使用完之后并不是直接关闭数据库连接，再后面如果需要创建数据库连接的时候直接拿之前释放的数据库连接使用，避免频繁创建和关闭数据库连接造成的开销。

在 mybatis 中，定义了一个数据库连接池状态的类 PoolState，在这个类里，除维护了数据源实例，还维护着数据库连接。数据库连接被分成了两种状态类型并存放在两个列表中：idleConnections 和 activeConnections。

idleConnections:

空闲(idle)状态 PooledConnection 对象被放置到此集合中，表示当前闲置的没有被使用的 PooledConnection 集合，调用 PooledDataSource 的 getConnection()方法时，会优先从此集合中取 PooledConnection 对象。当用完一个 java.sql.Connection 对象时，MyBatis 会将其包裹成 PooledConnection 对象放到此集合中。

activeConnections:活动(active)状态的 PooledConnection 对象被放置到名为 activeConnections 的 ArrayList 中，表示当前正在被使用的 PooledConnection 集合，调用 PooledDataSource 的 getConnection()方法时，会优先从 idleConnections 集合中取 PooledConnection 对象,如果没有，则看此集合是否已满，如果未满，PooledDataSource 会创建出一个 PooledConnection，添加到此集合中，并返回。

![](https://img-blog.csdn.net/20170514134254024?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

下面我们看看怎么从连接池中获取一个数据库连接，还是从 PooledDataSource 类开始看起。

```
@Override  public Connection getConnection() throws SQLException {    return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();  }   @Override  public Connection getConnection(String username, String password) throws SQLException {    return popConnection(username, password).getProxyConnection();  }
```

这里都是调用了 popConnection()方法，然后返回其代理对象。

```
private PooledConnection popConnection(String username, String password) throws SQLException {    boolean countedWait = false;    PooledConnection conn = null;    long t = System.currentTimeMillis();    int localBadConnectionCount = 0;     while (conn == null) {      synchronized (state) {        if (!state.idleConnections.isEmpty()) {          // Pool has available connection          conn = state.idleConnections.remove(0);          if (log.isDebugEnabled()) {            log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");          }        } else {          // Pool does not have available connection          if (state.activeConnections.size() < poolMaximumActiveConnections) {            // Can create new connection            conn = new PooledConnection(dataSource.getConnection(), this);            if (log.isDebugEnabled()) {              log.debug("Created connection " + conn.getRealHashCode() + ".");            }          } else {            // Cannot create new connection            PooledConnection oldestActiveConnection = state.activeConnections.get(0);            long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();            if (longestCheckoutTime > poolMaximumCheckoutTime) {              // Can claim overdue connection              state.claimedOverdueConnectionCount++;              state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;              state.accumulatedCheckoutTime += longestCheckoutTime;              state.activeConnections.remove(oldestActiveConnection);              if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {                try {                  oldestActiveConnection.getRealConnection().rollback();                } catch (SQLException e) {                  log.debug("Bad connection. Could not roll back");                }                }              conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);              oldestActiveConnection.invalidate();              if (log.isDebugEnabled()) {                log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");              }            } else {              // Must wait              try {                if (!countedWait) {                  state.hadToWaitCount++;                  countedWait = true;                }                if (log.isDebugEnabled()) {                  log.debug("Waiting as long as " + poolTimeToWait + " milliseconds for connection.");                }                long wt = System.currentTimeMillis();                state.wait(poolTimeToWait);                state.accumulatedWaitTime += System.currentTimeMillis() - wt;              } catch (InterruptedException e) {                break;              }            }          }        }        if (conn != null) {          if (conn.isValid()) {            if (!conn.getRealConnection().getAutoCommit()) {              conn.getRealConnection().rollback();            }            conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));            conn.setCheckoutTimestamp(System.currentTimeMillis());            conn.setLastUsedTimestamp(System.currentTimeMillis());            state.activeConnections.add(conn);            state.requestCount++;            state.accumulatedRequestTime += System.currentTimeMillis() - t;          } else {            if (log.isDebugEnabled()) {              log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");            }            state.badConnectionCount++;            localBadConnectionCount++;            conn = null;            if (localBadConnectionCount > (poolMaximumIdleConnections + 3)) {              if (log.isDebugEnabled()) {                log.debug("PooledDataSource: Could not get a good connection to the database.");              }              throw new SQLException("PooledDataSource: Could not get a good connection to the database.");            }          }        }      }     }     if (conn == null) {      if (log.isDebugEnabled()) {        log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");      }      throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");    }     return conn;  }
```

我们看下上面的方法都做了什么：

1.  先看是否有空闲(idle)状态下的 PooledConnection 对象，如果有，就直接返回一个可用的 PooledConnection 对象；否则进行第 2 步。
    2\. 查看活动状态的 PooledConnection 池 activeConnections 是否已满；如果没有满，则创建一个新的 PooledConnection 对象，然后放到 activeConnections 池中，然后返回此 PooledConnection 对象；否则进行第三步；
2.  看最先进入 activeConnections 池中的 PooledConnection 对象是否已经过期：如果已经过期，从 activeConnections 池中移除此对象，然后创建一个新的 PooledConnection 对象，添加到 activeConnections 中，然后将此对象返回；否则进行第 4 步；
3.  线程等待，循环 2 步。

流程图如下所示：

![](https://img-blog.csdn.net/20170514140700209?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

当我们拿到数据库连接 PooledConnection 后，我们在使用完之后一般来说就要关闭这个数据库连接，但是，对于池化来说，我们关闭了一个数据库连接并不是真正意义上想关闭这个连接，而是想把它放回到数据库连接池中。

怎么实现呢？mybatis 中使用了代理模式有效的解决了该问题。就是返回给外部使用的数据库连接其实是一个代理对象（通过调用 getProxyConnection()返回的对象）。这个代理对象时在真实数据库连接创建的时候被创建的，如下所示：

```
public PooledConnection(Connection connection, PooledDataSource dataSource) {    this.hashCode = connection.hashCode();    this.realConnection = connection;    this.dataSource = dataSource;    this.createdTimestamp = System.currentTimeMillis();    this.lastUsedTimestamp = System.currentTimeMillis();    this.valid = true;    this.proxyConnection = (Connection) Proxy.newProxyInstance(Connection.class.getClassLoader(), IFACES, this);  }
```

而在调用这个代理对象的各个方法的时候，都是通过反射的方式，从 invoke()方法进入，我们来看看：

```
 @Override  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {    String methodName = method.getName();    if (CLOSE.hashCode() == methodName.hashCode() && CLOSE.equals(methodName)) {      dataSource.pushConnection(this);      return null;    } else {      try {        if (!Object.class.equals(method.getDeclaringClass())) {          // issue #579 toString() should never fail          // throw an SQLException instead of a Runtime          checkConnection();        }        return method.invoke(realConnection, args);      } catch (Throwable t) {        throw ExceptionUtil.unwrapThrowable(t);      }    }  }
```

```
private static final String CLOSE = "close";
```

我们可以看到，这里做了一个特殊处理，那就是判断调用的方法名是否是 close()方法，如果是的话，就调用数据源对象的 pushConnection()方法将数据库连接放回到连接池中，如下所示：

```
protected void pushConnection(PooledConnection conn) throws SQLException {     synchronized (state) {      state.activeConnections.remove(conn);      if (conn.isValid()) {        if (state.idleConnections.size() < poolMaximumIdleConnections && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {          state.accumulatedCheckoutTime += conn.getCheckoutTime();          if (!conn.getRealConnection().getAutoCommit()) {            conn.getRealConnection().rollback();          }          PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);          state.idleConnections.add(newConn);          newConn.setCreatedTimestamp(conn.getCreatedTimestamp());          newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());          conn.invalidate();          if (log.isDebugEnabled()) {            log.debug("Returned connection " + newConn.getRealHashCode() + " to pool.");          }          state.notifyAll();        } else {          state.accumulatedCheckoutTime += conn.getCheckoutTime();          if (!conn.getRealConnection().getAutoCommit()) {            conn.getRealConnection().rollback();          }          conn.getRealConnection().close();          if (log.isDebugEnabled()) {            log.debug("Closed connection " + conn.getRealHashCode() + ".");          }          conn.invalidate();        }      } else {        if (log.isDebugEnabled()) {          log.debug("A bad connection (" + conn.getRealHashCode() + ") attempted to return to the pool, discarding connection.");        }        state.badConnectionCount++;      }    }  }
```

简单的说下上面这个方法的逻辑：

1\. 首先将当前数据库连接从活动数据库连接集合 activeConnections 中移除；

2\. 判断当前数据库连接是否有效，如果无效，则跳转到第 4 步；如果有效，则继续下面的判断；

3\. 判断当前 idleConnections 集合中的闲置数据库连接数量是否没超过设置的阈值且是当前数据库连接池的创建出来的链接，如果是，则将该数据库连接放回到 idleConnections 集合中并且通知在此据库连接池上等待的请求对象线程，如果不是，则将数据库连接关闭；

4\. 将连接池中的坏数据库连接数+1，并返回；

（6）JNDI 类型的数据源工厂 JndiDataSourceFactory

对于 JNDI 类型的数据源的获取比较简单，mybatis 中定义了一个 JndiDataSourceFactory 类用来创建通过 JNDI 形式创建的数据源。这个类源码如下： 

```
public class JndiDataSourceFactory implements DataSourceFactory {
 
  public static final String INITIAL_CONTEXT = "initial_context";
  public static final String DATA_SOURCE = "data_source";
  public static final String ENV_PREFIX = "env.";
 
  private DataSource dataSource;
 
  @Override
  public void setProperties(Properties properties) {
    try {
      InitialContext initCtx;
      Properties env = getEnvProperties(properties);
      if (env == null) {
        initCtx = new InitialContext();
      } else {
        initCtx = new InitialContext(env);
      }
 
      if (properties.containsKey(INITIAL_CONTEXT)
          && properties.containsKey(DATA_SOURCE)) {
        Context ctx = (Context) initCtx.lookup(properties.getProperty(INITIAL_CONTEXT));
        dataSource = (DataSource) ctx.lookup(properties.getProperty(DATA_SOURCE));
      } else if (properties.containsKey(DATA_SOURCE)) {
        dataSource = (DataSource) initCtx.lookup(properties.getProperty(DATA_SOURCE));
      }
 
    } catch (NamingException e) {
      throw new DataSourceException("There was an error configuring JndiDataSourceTransactionPool. Cause: " + e, e);
    }
  }
 
  @Override
  public DataSource getDataSource() {
    return dataSource;
  }
 
  private static Properties getEnvProperties(Properties allProps) {
    final String PREFIX = ENV_PREFIX;
    Properties contextProperties = null;
    for (Entry<Object, Object> entry : allProps.entrySet()) {
      String key = (String) entry.getKey();
      String value = (String) entry.getValue();
      if (key.startsWith(PREFIX)) {
        if (contextProperties == null) {
          contextProperties = new Properties();
        }
        contextProperties.put(key.substring(PREFIX.length()), value);
      }
    }
    return contextProperties;
  }
 
}
```

因为这块没看明白，对 JNDI 不是太熟悉，所以这块就不解释了，回头对这块了解了之后再行补充说明。如果有懂的朋友，也可以留言说明一下，谢谢。



# Mybatis中的事务管理器详述

[原文链接](https://blog.csdn.net/andamajing/article/details/72026693)

在上篇文章<[Mybatis中的数据源与连接池详解 ](http://blog.csdn.net/majinggogogo/article/details/71715846)>中，我们结合源码对 mybatis 中的数据源和连接池进行了比较详细的说明。在这篇文章中，我们讲讲相关的另外一个主题——事务管理器。

在前面的文章中，我们知道 mybatis 支持两种事务类型，分别为 JdbcTransaction 和 ManagedTransaction。接下来，我们从 mybatis 的 xml 配置文件入手，讲解事务管理器工厂的创建，然后讲述事务的创建和使用，最后分析这两种事务的实现和两者的区别。

我们先看看配置文件中相关的配置： 

![](https://img-blog.csdn.net/20170514164939290?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

Mybatis 定义了一个事务类型接口 Transaction，JdbcTransaction 和 ManagedTransaction 两种事务类型都实现了 Transaction 接口。我们看看 Transaction 这个接口的定义：

```
public interface Transaction {   /**   * Retrieve inner database connection   * @return DataBase connection   * @throws SQLException   */  Connection getConnection() throws SQLException;   /**   * Commit inner database connection.   * @throws SQLException   */  void commit() throws SQLException;   /**   * Rollback inner database connection.   * @throws SQLException   */  void rollback() throws SQLException;   /**   * Close inner database connection.   * @throws SQLException   */  void close() throws SQLException;   /**   * Get transaction timeout if set   * @throws SQLException   */  Integer getTimeout() throws SQLException;  }
```

 在事务接口中，定义了若干方法，如下结构所示：

![](https://img-blog.csdn.net/20170514165238385?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

事务的继承关系如下：

![](https://img-blog.csdn.net/20170514165311260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

JdbcTransaction 和 ManagedTransaction 的区别如下：

JdbcTransaction：利用 java.sql.Connection 对象完成对事务的提交（commit()）、回滚（rollback()）、关闭（close()）等；

ManagedTransaction：MyBatis 自身不会去实现事务管理，而是让程序的容器来实现对事务的管理；

那么在 mybatis 中又是怎么使用事务管理器的呢？首先需要根据 xml 中的配置确定需要创建什么样的事务管理器，然后从事务管理器中获取相应的事务。

在 mybatis 初始化的时候，在解析<transactionManager>节点的时候，根据设置的 type 类型去初始化相应的事务管理器，解析源码如下所示：

```
 /**    * 解析<transactionManager>节点，创建对应的TransactionFactory    * @param context    * @return    * @throws Exception    */  private TransactionFactory transactionManagerElement(XNode context) throws Exception {    if (context != null) {      String type = context.getStringAttribute("type");      Properties props = context.getChildrenAsProperties();      /*           在Configuration初始化的时候，会通过以下语句，给JDBC和MANAGED对应的工厂类           typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);           typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);           下述的resolveClass(type).newInstance()会创建对应的工厂实例      */      TransactionFactory factory = (TransactionFactory) resolveClass(type).newInstance();      factory.setProperties(props);      return factory;    }    throw new BuilderException("Environment declaration requires a TransactionFactory.");  }  
```

 从代码可以看出来，如果 type 配置成 JDBC，则创建一个 JdbcTransactionFactory 实例，如果 type 配置成 MANAGED，则会创建一个 ManagedTransactionFactory 实例。这两个事务管理器类型都实现了 mybatis 定义的 TransactionFactory 接口。

![](https://img-blog.csdn.net/20170514170734398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

事务管理器工厂接口的定义如下所示：

```
public interface TransactionFactory {   /**   * Sets transaction factory custom properties.   * @param props   */  void setProperties(Properties props);   /**   * Creates a {@link Transaction} out of an existing connection.   * @param conn Existing database connection   * @return Transaction   * @since 3.1.0   */  Transaction newTransaction(Connection conn);    /**   * Creates a {@link Transaction} out of a datasource.   * @param dataSource DataSource to take the connection from   * @param level Desired isolation level   * @param autoCommit Desired autocommit   * @return Transaction   * @since 3.1.0   */  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit); }
```

 从接口定义看，不管是 JdbcTransactionFactory，还是 ManagedTransactionFactory，都需要自行实现事务 Transaction 的创建工作。我们从源码上看看这两个类都是怎么定义实现的。

JdbcTransactionFactory 定义如下： 

```
public class JdbcTransactionFactory implements TransactionFactory {   @Override  public void setProperties(Properties props) {  }   @Override  public Transaction newTransaction(Connection conn) {    return new JdbcTransaction(conn);  }   @Override  public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {    return new JdbcTransaction(ds, level, autoCommit);  }}
```

 ManagedTransactionFactory 定义如下：

```
public class ManagedTransactionFactory implements TransactionFactory {   private boolean closeConnection = true;   @Override  public void setProperties(Properties props) {    if (props != null) {      String closeConnectionProperty = props.getProperty("closeConnection");      if (closeConnectionProperty != null) {        closeConnection = Boolean.valueOf(closeConnectionProperty);      }    }  }   @Override  public Transaction newTransaction(Connection conn) {    return new ManagedTransaction(conn, closeConnection);  }   @Override  public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {    // Silently ignores autocommit and isolation level, as managed transactions are entirely    // controlled by an external manager.  It's silently ignored so that    // code remains portable between managed and unmanaged configurations.    return new ManagedTransaction(ds, level, closeConnection);  }}
```

 从源码看，JdbcTransactionFactory 会创建 JDBC 类型的事务 JdbcTransaction，而 ManagedTransactionFactory 则会创建 ManagedTransaction。

接下来，我们再来看看这两种类型的事务类型。

先从 JdbcTransaction 看起吧，这种类型的事务就是使用 Java 自带的 Connection 来实现事务的管理。connection 对象的获取被延迟到调用 getConnection()方法。如果 autocommit 设置为 on，开启状态的话，它会忽略 commit 和 rollback。直观地讲，就是 JdbcTransaction 是使用的 java.sql.Connection 上的 commit 和 rollback 功能，JdbcTransaction 只是相当于对 java.sql.Connection 事务处理进行了一次包装（wrapper），Transaction 的事务管理都是通过 java.sql.Connection 实现的。

JdbcTransaction 的代码实现如下：

```
public class JdbcTransaction implements Transaction {   private static final Log log = LogFactory.getLog(JdbcTransaction.class);   protected Connection connection;  protected DataSource dataSource;  protected TransactionIsolationLevel level;  protected boolean autoCommmit;   public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {    dataSource = ds;    level = desiredLevel;    autoCommmit = desiredAutoCommit;  }   public JdbcTransaction(Connection connection) {    this.connection = connection;  }   @Override  public Connection getConnection() throws SQLException {    if (connection == null) {      openConnection();    }    return connection;  }   @Override  public void commit() throws SQLException {    if (connection != null && !connection.getAutoCommit()) {      if (log.isDebugEnabled()) {        log.debug("Committing JDBC Connection [" + connection + "]");      }      connection.commit();    }  }   @Override  public void rollback() throws SQLException {    if (connection != null && !connection.getAutoCommit()) {      if (log.isDebugEnabled()) {        log.debug("Rolling back JDBC Connection [" + connection + "]");      }      connection.rollback();    }  }   @Override  public void close() throws SQLException {    if (connection != null) {      resetAutoCommit();      if (log.isDebugEnabled()) {        log.debug("Closing JDBC Connection [" + connection + "]");      }      connection.close();    }  }   protected void setDesiredAutoCommit(boolean desiredAutoCommit) {    try {      if (connection.getAutoCommit() != desiredAutoCommit) {        if (log.isDebugEnabled()) {          log.debug("Setting autocommit to " + desiredAutoCommit + " on JDBC Connection [" + connection + "]");        }        connection.setAutoCommit(desiredAutoCommit);      }    } catch (SQLException e) {      // Only a very poorly implemented driver would fail here,      // and there's not much we can do about that.      throw new TransactionException("Error configuring AutoCommit.  "          + "Your driver may not support getAutoCommit() or setAutoCommit(). "          + "Requested setting: " + desiredAutoCommit + ".  Cause: " + e, e);    }  }   protected void resetAutoCommit() {    try {      if (!connection.getAutoCommit()) {        // MyBatis does not call commit/rollback on a connection if just selects were performed.        // Some databases start transactions with select statements        // and they mandate a commit/rollback before closing the connection.        // A workaround is setting the autocommit to true before closing the connection.        // Sybase throws an exception here.        if (log.isDebugEnabled()) {          log.debug("Resetting autocommit to true on JDBC Connection [" + connection + "]");        }        connection.setAutoCommit(true);      }    } catch (SQLException e) {      if (log.isDebugEnabled()) {        log.debug("Error resetting autocommit to true "          + "before closing the connection.  Cause: " + e);      }    }  }   protected void openConnection() throws SQLException {    if (log.isDebugEnabled()) {      log.debug("Opening JDBC Connection");    }    connection = dataSource.getConnection();    if (level != null) {      connection.setTransactionIsolation(level.getLevel());    }    setDesiredAutoCommit(autoCommmit);  }   @Override  public Integer getTimeout() throws SQLException {    return null;  }  }
```

 最后我们再来看看 ManagedTransaction 对象，这个对象因为是将事务的管理交给容器去控制，所以，这里的 ManagedTransaction 是没有做任何控制的。我们先来看看源码：

```
public class ManagedTransaction implements Transaction {   private static final Log log = LogFactory.getLog(ManagedTransaction.class);   private DataSource dataSource;  private TransactionIsolationLevel level;  private Connection connection;  private boolean closeConnection;   public ManagedTransaction(Connection connection, boolean closeConnection) {    this.connection = connection;    this.closeConnection = closeConnection;  }   public ManagedTransaction(DataSource ds, TransactionIsolationLevel level, boolean closeConnection) {    this.dataSource = ds;    this.level = level;    this.closeConnection = closeConnection;  }   @Override  public Connection getConnection() throws SQLException {    if (this.connection == null) {      openConnection();    }    return this.connection;  }   @Override  public void commit() throws SQLException {    // Does nothing  }   @Override  public void rollback() throws SQLException {    // Does nothing  }   @Override  public void close() throws SQLException {    if (this.closeConnection && this.connection != null) {      if (log.isDebugEnabled()) {        log.debug("Closing JDBC Connection [" + this.connection + "]");      }      this.connection.close();    }  }   protected void openConnection() throws SQLException {    if (log.isDebugEnabled()) {      log.debug("Opening JDBC Connection");    }    this.connection = this.dataSource.getConnection();    if (this.level != null) {      this.connection.setTransactionIsolation(this.level.getLevel());    }  }   @Override  public Integer getTimeout() throws SQLException {    return null;  } }
```

 从源码我们可以看到，ManagedTransaction 的 commit 和 rollback 方法是没有做任何事情的，它将事务交由了更上层的容易来进行控制和实现。至此，关于事务管理器我们描述的已经差不多了，如果需要深究可以自己再去研究研究。



# Mybatis配置之元素详述

2017 年 05 月 14 日 21:10:54 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 4866 更多

分类专栏： [Mybatis](https://blog.csdn.net/andamajing/article/category/6902012) [Mybatis应用及原理探析](https://blog.csdn.net/andamajing/article/category/6902014) [深入了解Mybatis使用及实现原理](https://blog.csdn.net/andamajing/article/category/9268791)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72063702](https://blog.csdn.net/andamajing/article/details/72063702) 

在<[Mybatis配置之<environments>配置元素详述](http://blog.csdn.net/majinggogogo/article/details/71616712)>我们对<environments>元素配置进行了说明，而后通过两篇文章<[Mybatis中的数据源与连接池详解](http://blog.csdn.net/majinggogogo/article/details/71715846)><[Mybatis中的事务管理器详述](http://blog.csdn.net/majinggogogo/article/details/72026693)>分别对数据源和事务管理器进行了详述，从这篇文章起，我们继续来讲述 mybatis 配置文件中的其他元素配置项。今天，我们就来说说<typeHandlers>这个元素,看看是怎么使用的。

我们都知道，mybatis 为我们隐藏了很多操作数据库的代码，如在预处理语句 PreparedStatement 中设置一个参数，或是在执行完 SQL 语句后从结果集中取出数据。而这两个过程都需要合适的数据类型处理器来帮我们对数据进行正确的类型转换，在 mybatis 中又是谁帮我们在做这些事情呢？

那就是<typeHandlers>元素了！

我们还是从配置文件开始看起，来看看 typeHandlers 怎么配置。

```
<typeHandlers>      <!--           当配置package的时候，mybatis会去配置的package扫描TypeHandler          <package name="com.dy.demo"/>       -->            <!-- handler属性直接配置我们要指定的TypeHandler -->      <typeHandler handler=""/>            <!-- javaType 配置java类型，例如String, 如果配上javaType, 那么指定的typeHandler就只作用于指定的类型 -->      <typeHandler javaType="" handler=""/>            <!-- jdbcType 配置数据库基本数据类型，例如varchar, 如果配上jdbcType, 那么指定的typeHandler就只作用于指定的类型  -->      <typeHandler jdbcType="" handler=""/>            <!-- 也可两者都配置 -->      <typeHandler javaType="" jdbcType="" handler=""/>        </typeHandlers>
```

 结合上面的配置，我们再看看 mybatis 源码中是怎么进行解析的。

```
private void typeHandlerElement(XNode parent) throws Exception {    if (parent != null) {      for (XNode child : parent.getChildren()) {        //子节点为package时，获取其name属性的值，然后自动扫描package下的自定义typeHandler        if ("package".equals(child.getName())) {          String typeHandlerPackage = child.getStringAttribute("name");          typeHandlerRegistry.register(typeHandlerPackage);        } else {          //子节点为typeHandler时， 可以指定javaType属性， 也可以指定jdbcType, 也可两者都指定          //javaType 是指定java类型          //jdbcType 是指定jdbc类型（数据库类型： 如varchar）          String javaTypeName = child.getStringAttribute("javaType");          String jdbcTypeName = child.getStringAttribute("jdbcType");          //handler就是我们配置的typeHandler          String handlerTypeName = child.getStringAttribute("handler");          //resolveClass方法就是我们上篇文章所讲的TypeAliasRegistry里面处理别名的方法          Class<?> javaTypeClass = resolveClass(javaTypeName);          //JdbcType是一个枚举类型，resolveJdbcType方法是在获取枚举类型的值          JdbcType jdbcType = resolveJdbcType(jdbcTypeName);          Class<?> typeHandlerClass = resolveClass(handlerTypeName);          //注册typeHandler, typeHandler通过TypeHandlerRegistry这个类管理          if (javaTypeClass != null) {            if (jdbcType == null) {              typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);            } else {              typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);            }          } else {            typeHandlerRegistry.register(typeHandlerClass);          }        }      }    }}
```

 从源码看，大的类型上支持两种方式，一种是包名的方式，另一种是只配置单个类型转换器。在解析的过程中，将配置的这些类型转换器都注册到 typeHandlerRegistry 中。其实，在 mybatis 中已经为我们事先注册好了绝大多数的类型转换器，可以满足我们绝大多数的使用场景。

```
private final Map<JdbcType, TypeHandler<?>> JDBC_TYPE_HANDLER_MAP = new EnumMap<JdbcType, TypeHandler<?>>(JdbcType.class);  private final Map<Type, Map<JdbcType, TypeHandler<?>>> TYPE_HANDLER_MAP = new HashMap<Type, Map<JdbcType, TypeHandler<?>>>();  private final TypeHandler<Object> UNKNOWN_TYPE_HANDLER = new UnknownTypeHandler(this);  private final Map<Class<?>, TypeHandler<?>> ALL_TYPE_HANDLERS_MAP = new HashMap<Class<?>, TypeHandler<?>>();   public TypeHandlerRegistry() {    register(Boolean.class, new BooleanTypeHandler());    register(boolean.class, new BooleanTypeHandler());    register(JdbcType.BOOLEAN, new BooleanTypeHandler());    register(JdbcType.BIT, new BooleanTypeHandler());     register(Byte.class, new ByteTypeHandler());    register(byte.class, new ByteTypeHandler());    register(JdbcType.TINYINT, new ByteTypeHandler());     register(Short.class, new ShortTypeHandler());    register(short.class, new ShortTypeHandler());    register(JdbcType.SMALLINT, new ShortTypeHandler());     register(Integer.class, new IntegerTypeHandler());    register(int.class, new IntegerTypeHandler());    register(JdbcType.INTEGER, new IntegerTypeHandler());     register(Long.class, new LongTypeHandler());    register(long.class, new LongTypeHandler());     register(Float.class, new FloatTypeHandler());    register(float.class, new FloatTypeHandler());    register(JdbcType.FLOAT, new FloatTypeHandler());     register(Double.class, new DoubleTypeHandler());    register(double.class, new DoubleTypeHandler());    register(JdbcType.DOUBLE, new DoubleTypeHandler());     register(Reader.class, new ClobReaderTypeHandler());    register(String.class, new StringTypeHandler());    register(String.class, JdbcType.CHAR, new StringTypeHandler());    register(String.class, JdbcType.CLOB, new ClobTypeHandler());    register(String.class, JdbcType.VARCHAR, new StringTypeHandler());    register(String.class, JdbcType.LONGVARCHAR, new ClobTypeHandler());    register(String.class, JdbcType.NVARCHAR, new NStringTypeHandler());    register(String.class, JdbcType.NCHAR, new NStringTypeHandler());    register(String.class, JdbcType.NCLOB, new NClobTypeHandler());    register(JdbcType.CHAR, new StringTypeHandler());    register(JdbcType.VARCHAR, new StringTypeHandler());    register(JdbcType.CLOB, new ClobTypeHandler());    register(JdbcType.LONGVARCHAR, new ClobTypeHandler());    register(JdbcType.NVARCHAR, new NStringTypeHandler());    register(JdbcType.NCHAR, new NStringTypeHandler());    register(JdbcType.NCLOB, new NClobTypeHandler());     register(Object.class, JdbcType.ARRAY, new ArrayTypeHandler());    register(JdbcType.ARRAY, new ArrayTypeHandler());     register(BigInteger.class, new BigIntegerTypeHandler());    register(JdbcType.BIGINT, new LongTypeHandler());     register(BigDecimal.class, new BigDecimalTypeHandler());    register(JdbcType.REAL, new BigDecimalTypeHandler());    register(JdbcType.DECIMAL, new BigDecimalTypeHandler());    register(JdbcType.NUMERIC, new BigDecimalTypeHandler());     register(InputStream.class, new BlobInputStreamTypeHandler());    register(Byte[].class, new ByteObjectArrayTypeHandler());    register(Byte[].class, JdbcType.BLOB, new BlobByteObjectArrayTypeHandler());    register(Byte[].class, JdbcType.LONGVARBINARY, new BlobByteObjectArrayTypeHandler());    register(byte[].class, new ByteArrayTypeHandler());    register(byte[].class, JdbcType.BLOB, new BlobTypeHandler());    register(byte[].class, JdbcType.LONGVARBINARY, new BlobTypeHandler());    register(JdbcType.LONGVARBINARY, new BlobTypeHandler());    register(JdbcType.BLOB, new BlobTypeHandler());     register(Object.class, UNKNOWN_TYPE_HANDLER);    register(Object.class, JdbcType.OTHER, UNKNOWN_TYPE_HANDLER);    register(JdbcType.OTHER, UNKNOWN_TYPE_HANDLER);     register(Date.class, new DateTypeHandler());    register(Date.class, JdbcType.DATE, new DateOnlyTypeHandler());    register(Date.class, JdbcType.TIME, new TimeOnlyTypeHandler());    register(JdbcType.TIMESTAMP, new DateTypeHandler());    register(JdbcType.DATE, new DateOnlyTypeHandler());    register(JdbcType.TIME, new TimeOnlyTypeHandler());     register(java.sql.Date.class, new SqlDateTypeHandler());    register(java.sql.Time.class, new SqlTimeTypeHandler());    register(java.sql.Timestamp.class, new SqlTimestampTypeHandler());     // mybatis-typehandlers-jsr310    try {      // since 1.0.0      register("java.time.Instant", "org.apache.ibatis.type.InstantTypeHandler");      register("java.time.LocalDateTime", "org.apache.ibatis.type.LocalDateTimeTypeHandler");      register("java.time.LocalDate", "org.apache.ibatis.type.LocalDateTypeHandler");      register("java.time.LocalTime", "org.apache.ibatis.type.LocalTimeTypeHandler");      register("java.time.OffsetDateTime", "org.apache.ibatis.type.OffsetDateTimeTypeHandler");      register("java.time.OffsetTime", "org.apache.ibatis.type.OffsetTimeTypeHandler");      register("java.time.ZonedDateTime", "org.apache.ibatis.type.ZonedDateTimeTypeHandler");      // since 1.0.1      register("java.time.Month", "org.apache.ibatis.type.MonthTypeHandler");      register("java.time.Year", "org.apache.ibatis.type.YearTypeHandler");     } catch (ClassNotFoundException e) {      // no JSR-310 handlers    }     // issue #273    register(Character.class, new CharacterTypeHandler());    register(char.class, new CharacterTypeHandler());  }
```

 我们可以发现这些注册了的类型转换器都继承自 BaseTypeHandler，如果现有的类型注册器不能满足需求，那么就需要我们根据需要自己进行定制，而定制的类型转换器都必须继承 BaseTypeHandler。我们先看看这个基类的源码实现：

```
public abstract class BaseTypeHandler<T> extends TypeReference<T> implements TypeHandler<T> {   protected Configuration configuration;   public void setConfiguration(Configuration c) {    this.configuration = c;  }   @Override  public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {    if (parameter == null) {      if (jdbcType == null) {        throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");      }      try {        ps.setNull(i, jdbcType.TYPE_CODE);      } catch (SQLException e) {        throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . " +                "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. " +                "Cause: " + e, e);      }    } else {      try {        setNonNullParameter(ps, i, parameter, jdbcType);      } catch (Exception e) {        throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . " +                "Try setting a different JdbcType for this parameter or a different configuration property. " +                "Cause: " + e, e);      }    }  }   @Override  public T getResult(ResultSet rs, String columnName) throws SQLException {    T result;    try {      result = getNullableResult(rs, columnName);    } catch (Exception e) {      throw new ResultMapException("Error attempting to get column '" + columnName + "' from result set.  Cause: " + e, e);    }    if (rs.wasNull()) {      return null;    } else {      return result;    }  }   @Override  public T getResult(ResultSet rs, int columnIndex) throws SQLException {    T result;    try {      result = getNullableResult(rs, columnIndex);    } catch (Exception e) {      throw new ResultMapException("Error attempting to get column #" + columnIndex+ " from result set.  Cause: " + e, e);    }    if (rs.wasNull()) {      return null;    } else {      return result;    }  }   @Override  public T getResult(CallableStatement cs, int columnIndex) throws SQLException {    T result;    try {      result = getNullableResult(cs, columnIndex);    } catch (Exception e) {      throw new ResultMapException("Error attempting to get column #" + columnIndex+ " from callable statement.  Cause: " + e, e);    }    if (cs.wasNull()) {      return null;    } else {      return result;    }  }   public abstract void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;   public abstract T getNullableResult(ResultSet rs, String columnName) throws SQLException;   public abstract T getNullableResult(ResultSet rs, int columnIndex) throws SQLException;   public abstract T getNullableResult(CallableStatement cs, int columnIndex) throws SQLException; }
```

 从上面的源码可以看出来，核心的代码已经实现了，如怎么给预处理语句设置参数，怎么从查询结果集中获取数据。在基类中还有四个抽象方法，具体的类型转换器只需要实现这四个方法就可以了。如下面我们定义一个类型转换器用于对 VARCHAR 类型数据进行处理。类型转换器定义如下：

```
@MappedJdbcTypes(JdbcType.VARCHAR)  //此处如果不用注解指定jdbcType, 那么，就可以在配置文件中通过"jdbcType"属性指定， 同理， javaType 也可通过 @MappedTypes指定public class ExampleTypeHandler extends BaseTypeHandler<String> {   @Override  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {    ps.setString(i, parameter);  }   @Override  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {    return rs.getString(columnName);  }   @Override  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {    return rs.getString(columnIndex);  }   @Override  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {    return cs.getString(columnIndex);  }}
```

 然后我们只需要在配置文件中配置一下就可以了，如下所示：

```
<typeHandlers>      <!-- 由于自定义的TypeHandler在定义时已经通过注解指定了jdbcType, 所以此处不用再配置jdbcType -->      <typeHandler handler="ExampleTypeHandler"/>  </typeHandlers>
```

 备注说明：可以在类型转换器定义的时候通过@MappedJdbcTypes 指定 jdbcType, 通过 @MappedTypes 指定 javaType, 如果没有使用注解指定，那么我们就需要在配置文件中配置。

至此，关于<typeHandlers>元素的配置知识便描述的差不多了，感兴趣的朋友可以自己研究研究。



# Mybatis配置之元素详述

2017 年 05 月 15 日 07:23:53 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 2200 更多

分类专栏： [Mybatis](https://blog.csdn.net/andamajing/article/category/6902012) [Mybatis应用及原理探析](https://blog.csdn.net/andamajing/article/category/6902014) [深入了解Mybatis使用及实现原理](https://blog.csdn.net/andamajing/article/category/9268791)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72085580](https://blog.csdn.net/andamajing/article/details/72085580) 

在上篇文章中，我们对<typeHandlers>配置及背后的源码进行了比较详细的说明，今天，我们来对下一个元素<objectFactory>进行详细说明。

这个元素，大家在使用 mybatis 的时候设置吗？我是从来没有设置过啊。使用 mybatis 为我们已经写好的默认实现已经能够满足绝大多数的场景需求。那么这个元素又是干什么的呢？

官方文档上是这么说的：

**MyBatis 每次创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认构造方法，要么在参数映射存在的时候通过参数构造方法来实例化。 如果想覆盖对象工厂的默认行为，则可以通过创建自己的对象工厂来实现。**

我理解的意思就是说，这个类其实是为了在对查询结果集中获取数据被封装成所希望的 Java 实体类型时用到的，使用这个工厂类通过反射的方式来进行实例对象的创建。所有的工厂类都必须实现 ObjectFactory 接口。我们来看看这个接口的定义：

```
public interface ObjectFactory {   /**   * Sets configuration properties.   * @param properties configuration properties   */  void setProperties(Properties properties);   /**   * Creates a new object with default constructor.    * @param type Object type   * @return   */  <T> T create(Class<T> type);   /**   * Creates a new object with the specified constructor and params.   * @param type Object type   * @param constructorArgTypes Constructor argument types   * @param constructorArgs Constructor argument values   * @return   */  <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs);    /**   * Returns true if this object can have a set of other objects.   * It's main purpose is to support non-java.util.Collection objects like Scala collections.   *    * @param type Object type   * @return whether it is a collection or not   * @since 3.1.0   */  <T> boolean isCollection(Class<T> type); }
```

 从这个接口定义可以看出，它包含了两种通过反射机制构造实体类对象的方法，一种是通过无参构造函数，一种是通过带参数的构造函数。另外，为了使的这个工厂类能接收设置的附带属性，还提供了 setProperties()方法。mybatis 为我们实现了一个默认实现，那就是 DefaultObjectFactory，我们来看看默认是怎么实现的。

```
public class DefaultObjectFactory implements ObjectFactory, Serializable {   private static final long serialVersionUID = -8855120656740914948L;   @Override  public <T> T create(Class<T> type) {    return create(type, null, null);  }   @SuppressWarnings("unchecked")  @Override  public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {    Class<?> classToCreate = resolveInterface(type);    // we know types are assignable    return (T) instantiateClass(classToCreate, constructorArgTypes, constructorArgs);  }   @Override  public void setProperties(Properties properties) {    // no props for default  }   <T> T instantiateClass(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {    try {      Constructor<T> constructor;      if (constructorArgTypes == null || constructorArgs == null) {        constructor = type.getDeclaredConstructor();        if (!constructor.isAccessible()) {          constructor.setAccessible(true);        }        return constructor.newInstance();      }      constructor = type.getDeclaredConstructor(constructorArgTypes.toArray(new Class[constructorArgTypes.size()]));      if (!constructor.isAccessible()) {        constructor.setAccessible(true);      }      return constructor.newInstance(constructorArgs.toArray(new Object[constructorArgs.size()]));    } catch (Exception e) {      StringBuilder argTypes = new StringBuilder();      if (constructorArgTypes != null && !constructorArgTypes.isEmpty()) {        for (Class<?> argType : constructorArgTypes) {          argTypes.append(argType.getSimpleName());          argTypes.append(",");        }        argTypes.deleteCharAt(argTypes.length() - 1); // remove trailing ,      }      StringBuilder argValues = new StringBuilder();      if (constructorArgs != null && !constructorArgs.isEmpty()) {        for (Object argValue : constructorArgs) {          argValues.append(String.valueOf(argValue));          argValues.append(",");        }        argValues.deleteCharAt(argValues.length() - 1); // remove trailing ,      }      throw new ReflectionException("Error instantiating " + type + " with invalid types (" + argTypes + ") or values (" + argValues + "). Cause: " + e, e);    }  }   protected Class<?> resolveInterface(Class<?> type) {    Class<?> classToCreate;    if (type == List.class || type == Collection.class || type == Iterable.class) {      classToCreate = ArrayList.class;    } else if (type == Map.class) {      classToCreate = HashMap.class;    } else if (type == SortedSet.class) { // issue #510 Collections Support      classToCreate = TreeSet.class;    } else if (type == Set.class) {      classToCreate = HashSet.class;    } else {      classToCreate = type;    }    return classToCreate;  }   @Override  public <T> boolean isCollection(Class<T> type) {    return Collection.class.isAssignableFrom(type);  } }
```

 从上面的源码来看，创建实例对象最终都是通过 instantiateClass()方法能实现的，在这个方法中，获取实体类的无参构造函数或者带参构造函数，然后采用反射的机制来实例化实体类对象出来。 

如果我们想定义一个自定义的对象工厂类，我们可以实现 ObjectFactory 这个接口，但是这样我们就需要自己去实现一些在 DefaultObjectFactory 已经实现好了的东西，因此如果想自定义一个，可以继承这个 DefaultObjectFactory 类，这样可以使得实现起来更为简单。

下面我们来举个示例，我们假设需要自定义一个对象工厂类 ExampleObjectFactory，在要创建我之前定义的 User 对象时会给属性字段 author 赋值为我的名字，接下来我们看看怎么实现。

首先我们定义实现这个对象工厂类：

```
package com.majing.learning.mybatis.reflect.objectfactory; import org.apache.ibatis.reflection.factory.DefaultObjectFactory; import com.majing.learning.mybatis.entity.User; public class ExampleObjectFactory extends DefaultObjectFactory{ 	private static final long serialVersionUID = 3608715667301891724L; 	@Override	public <T> T create(Class<T> type) {		T result = super.create(type);		if(type.equals(User.class)){			((User)result).setAuthor("马靖");		}		return result;	} }
```

 在这个工厂类中，我们对 User 类中无惨构造函数构造出来的对象做了个特殊处理，加了个我的名字。接下来，我们在配置文件中配置这个类为我们想要的对象工厂类。

```
<objectFactory type="com.majing.learning.mybatis.reflect.objectfactory.ExampleObjectFactory"></objectFactory>
```

 然后我们执行测试代码，查询一条用户记录出来，发现返回的对象中的确设置了我的名字，如下所示：

![](https://img-blog.csdn.net/20170515071916680?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

至此，关于 mybatis 中的对象工厂相关的知识点便讲解的差不多了，下一篇文章我们再来看看<Mappers>的配置和使用，敬请期待。



# Mybatis配置之元素详述

2017 年 05 月 15 日 10:12:06 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 9651

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72085696](https://blog.csdn.net/andamajing/article/details/72085696) 

在前面的若干篇文章中，我们已经对 mybatis 中主要的配置元素做了讲述，还剩下一个比较重要的元素，那就是<mappers>元素。

这个元素是干嘛用的呢？<mappers>用来在 mybatis 初始化的时候，告诉 mybatis 需要引入哪些 Mapper 映射文件。那什么又是 Mapper 映射文件呢？它是 Java 实体类与数据库对象之间的桥梁。在实际的使用过程中，一般一个 Mapper 文件对应一个数据库操作 Dao 接口。

在 mybatis 中，mappers 必须配置！那么怎么配置呢？我们先从源码上看下这个节点是怎么解析的。

```
private void mapperElement(XNode parent) throws Exception {    if (parent != null) {      for (XNode child : parent.getChildren()) {        if ("package".equals(child.getName())) {          String mapperPackage = child.getStringAttribute("name");          configuration.addMappers(mapperPackage);        } else {          String resource = child.getStringAttribute("resource");          String url = child.getStringAttribute("url");          String mapperClass = child.getStringAttribute("class");          if (resource != null && url == null && mapperClass == null) {            ErrorContext.instance().resource(resource);            InputStream inputStream = Resources.getResourceAsStream(resource);            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());            mapperParser.parse();          } else if (resource == null && url != null && mapperClass == null) {            ErrorContext.instance().resource(url);            InputStream inputStream = Resources.getUrlAsStream(url);            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());            mapperParser.parse();          } else if (resource == null && url == null && mapperClass != null) {            Class<?> mapperInterface = Resources.classForName(mapperClass);            configuration.addMapper(mapperInterface);          } else {            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");          }        }      }    }  }
```

 上面的代码便是 Mybatis 中<mappers>节点的解析入口了，我们来简单看下。

从代码看，我们知道分为两大类，一种是配置<package>子元素，另一种是配置<mapper>元素（从http://mybatis.org/dtd/mybatis-3-config.dtd文件限制的）。如下截图所示，

![](https://img-blog.csdn.net/20170515090624015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这两种方式都是怎么配置呢？<package>配置很简单，只需要配置一个 name 属性用于设置包名。<mapper>的配置则比较多样化，支持三种属性方式设置，分别为 resource、url 和 class 三个属性。具体的配置如下所示：

```
<configuration>    ......    <mappers>      <!-- 第一种方式：通过resource指定 -->    <mapper resource="com/dy/dao/userDao.xml"/>         <!-- 第二种方式， 通过class指定接口，进而将接口与对应的xml文件形成映射关系             不过，使用这种方式必须保证 接口与mapper文件同名(不区分大小写)，              我这儿接口是UserDao,那么意味着mapper文件为UserDao.xml      <mapper class="com.dy.dao.UserDao"/>      -->            <!-- 第三种方式，直接指定包，自动扫描，与方法二同理       <package name="com.dy.dao"/>      -->      <!-- 第四种方式：通过url指定mapper文件位置      <mapper url="file://........"/>       -->  </mappers>    ......  </configuration>
```

 关于<mappers>元素的配置讲解大概就这么多，其实核心的不在这里怎么配置，而是每个 mapper 映射文件才是重点。从下篇文章开始，我们将对 mapper 映射文件包含的元素进行详细说明，敬请期待。



# Mybatis中Mapper映射文件详解

2017 年 05 月 15 日 15:23:15 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 48408 更多

分类专栏： [Mybatis](https://blog.csdn.net/andamajing/article/category/6902012) [Mybatis应用及原理探析](https://blog.csdn.net/andamajing/article/category/6902014) [深入了解Mybatis使用及实现原理](https://blog.csdn.net/andamajing/article/category/9268791)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72123185](https://blog.csdn.net/andamajing/article/details/72123185) 

紧接上文所述，在这篇文章中我将对 Mapper 映射文件进行详细的说明。

Mapper 映射文件是一个 xml 格式文件，必须遵循相应的 dtd 文件规范，如 ibatis-3-mapper.dtd。我们先大体上看看支持哪些配置？如下所示，从 Eclipse 里截了个屏：

![](https://img-blog.csdn.net/20170515103333891?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

从上图可以看出，映射文件是以<mapper>作为根节点，在根节点中支持 9 个元素，分别为 insert、update、delete、select（增删改查）;cache、cache-ref、resultMap、parameterMap、sql。

下文中，我们将首先对增删改进行描述，然后对查进行详细说明，最后对其他五个元素进行简单说明。

（1）insert、update、delete

我们先从配置文件看起：

```
<?xml version="1.0" encoding="UTF-8" ?>   <!DOCTYPE mapper   PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">  <!-- mapper 为根元素节点， 一个namespace对应一个dao --><mapper namespace="com.dy.dao.UserDao">     <insert      <!-- 1. id （必须配置）        id是命名空间中的唯一标识符，可被用来代表这条语句。         一个命名空间（namespace） 对应一个dao接口,         这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致 -->            id="addUser"            <!-- 2\. parameterType （可选配置, 默认为mybatis自动选择处理）        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->            parameterType="user"            <!-- 3\. flushCache （可选配置，默认配置为true）        将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句） -->            flushCache="true"            <!-- 4\. statementType （可选配置，默认配置为PREPARED）        STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 -->            statementType="PREPARED"            <!-- 5\. keyProperty (可选配置， 默认为unset)        （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->            keyProperty=""            <!-- 6\. keyColumn     (可选配置)        （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->            keyColumn=""            <!-- 7\. useGeneratedKeys (可选配置， 默认为false)        （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。  -->            useGeneratedKeys="false"            <!-- 8\. timeout  (可选配置， 默认为unset, 依赖驱动)        这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 -->      timeout="20">     <update      id="updateUser"      parameterType="user"      flushCache="true"      statementType="PREPARED"      timeout="20">     <delete      id="deleteUser"      parameterType="user"      flushCache="true"      statementType="PREPARED"      timeout="20"></mapper>
```

 上面给出了一个比较全面的配置说明，但是在实际使用过程中并不需要都进行配置，可根据自己的需要删除部分配置项。

在这里，我列举出我自己的配置文件，精简之后是这样的：

```
<?xml version="1.0" encoding="UTF-8" ?>   <!DOCTYPE mapper   PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"><mapper namespace="com.majing.learning.mybatis.dao.UserDao">		<insert id="addUser" parameterType="user" useGeneratedKeys="true" keyProperty="id">		insert into user(name,password,age) values(#{name},#{password},#{age})	</insert>		<delete id="deleteUser" parameterType="int">		delete from user where id = #{id}	</delete>		<update id="updateUser" parameterType="user" >		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}	</update>		 </mapper>
```

 这里的 parameterType 设置成 user 是因为如果不设置的情况下，会自动将类名首字母小写后的名称，原来的类名为 User。不过，建议大家还是使用 typeAlias 进行配置吧。唯一需要说明的就是<insert>元素里面的 useGeneratedKeys 和 keyProperties 属性，这两个属性是用来获取数据库中的主键的。

在数据库里面经常性的会给数据库表设置一个自增长的列作为主键，如果我们操作数据库后希望能够获取这个主键该怎么弄呢？正如上面所述，如果是支持自增长的数据库，如 mysql 数据库，那么只需要设置 useGeneratedKeys 和 keyProperties 属性便可以了，但是对于不支持自增长的数据库（如 oracle）该怎么办呢？

mybatis 里面在<insert>元素下面提供了<selectKey>子元素用于帮助解决这个问题。来看下配置：

```
<selectKey        <!-- selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->        keyProperty="id"        <!-- 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 -->        resultType="int"        <!-- 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 -->        order="BEFORE"        <!-- 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 -->        statementType="PREPARED"></selectKey>
```

 针对不能使用自增长特性的数据库，可以使用下面的配置来实现相同的功能：

```
<insert id="insertUser" parameterType="com.dy.entity.User">           <!-- oracle等不支持id自增长的，可根据其id生成策略，先获取id -->                   <selectKey resultType="int" order="BEFORE" keyProperty="id">              select seq_user_id.nextval as id from dual        </selectKey>             insert into user(id, name, password, age, deleteFlag)                values(#{id}, #{name}, #{password}, #{age}, #{deleteFlag})   </insert>
```

 讲完了 insert、update、delete，接下来我们看看用的比较多的 select。

（2）select、resultType、resultMap 

我们先来看看 select 元素都有哪些配置可以设置：

```
<select        <!--  1. id （必须配置）        id是命名空间中的唯一标识符，可被用来代表这条语句。         一个命名空间（namespace） 对应一个dao接口,         这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致  -->          id="findUserById"          <!-- 2\. parameterType （可选配置, 默认为mybatis自动选择处理）        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->     parameterType="int"          <!-- 3\. resultType (resultType 与 resultMap 二选一配置)         resultType用以指定返回类型，指定的类型可以是基本类型，可以是java容器，也可以是javabean -->     resultType="User"          <!-- 4\. resultMap (resultType 与 resultMap 二选一配置)         resultMap用于引用我们通过 resultMap标签定义的映射类型，这也是mybatis组件高级复杂映射的关键 -->     resultMap="userResultMap"          <!-- 5\. flushCache (可选配置)         将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false -->     flushCache="false"          <!-- 6\. useCache (可选配置)         将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true -->     useCache="true"          <!-- 7\. timeout (可选配置)          这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）-->     timeout="10000"          <!-- 8\. fetchSize (可选配置)          这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动)-->     fetchSize="256"          <!-- 9\. statementType (可选配置)          STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED-->     statementType="PREPARED"          <!-- 10\. resultSetType (可选配置)          FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）-->     resultSetType="FORWARD_ONLY">
```

 我们还是从具体的示例来看看，

```
<select id="findUserById" resultType="User">	select * from user where id = #{id}</select>
```

 这里我们根据用户 id 去查询这个用户的信息，resultType=User 是一个别名，如果我们接触到的是这种一对一的问题，那么可以简单的定义一个实体，这个实体代表数据库表中的一条记录即可。但是如果我们遇到一对多的问题呢，就拿这里的查询用户信息来说，如果每个用户有各种兴趣，需要维护每个用户的兴趣信息，那么我们可能就存在下面的数据表结构：

```
CREATE TABLE `user` (  `id` int(11) NOT NULL AUTO_INCREMENT,  `name` varchar(255) NOT NULL,  `password` varchar(255) NOT NULL,  `age` int(3) DEFAULT NULL,  PRIMARY KEY (`id`)) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8; CREATE TABLE `userinterests` (  `userid` int(11) DEFAULT NULL,  `interestid` int(11) DEFAULT NULL) ENGINE=InnoDB DEFAULT CHARSET=utf8; CREATE TABLE `interests` (  `interestid` int(11) NOT NULL,  `interestname` varchar(255) DEFAULT NULL,  PRIMARY KEY (`interestid`)) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

 其中 user 表用于记录用户信息，interests 表用于维护所有的兴趣标签，而 userinterests 用于维护每个用户的兴趣情况。

这时候，如果我们需要根据用户 id 去查询用户的信息和兴趣信息该怎么做呢？这时候我们就要用到<select>中的另一个重要属性了，那就是 resultMap。

在配置查询的返回结果时，resultType 和 resultMap 是二选一的操作。对于比较复杂的查询结果，一般都会设置成 resultMap。

resultMap 该怎么配置呢？又支持哪些配置呢？我们看看下面：

```
<resultMap type="" id="">            <!-- id, 唯一性，注意啦，这个id用于标示这个javabean对象的唯一性， 不一定会是数据库的主键（不要把它理解为数据库对应表的主键）             property属性对应javabean的属性名，column对应数据库表的列名            （这样，当javabean的属性与数据库对应表的列名不一致的时候，就能通过指定这个保持正常映射了）        -->        <id property="" column=""/>                <!-- result与id相比， 对应普通属性 -->            <result property="" column=""/>                <!--             constructor对应javabean中的构造方法         -->        <constructor>            <!-- idArg 对应构造方法中的id参数 -->            <idArg column=""/>            <!-- arg 对应构造方法中的普通参数 -->            <arg column=""/>        </constructor>                <!--             collection，对应javabean中容器类型, 是实现一对多的关键             property 为javabean中容器对应字段名            column 为体现在数据库中列名            ofType 就是指定javabean中容器指定的类型        -->        <collection property="" column="" ofType=""></collection>                <!--             association 为关联关系，是实现N对一的关键。            property 为javabean中容器对应字段名            column 为体现在数据库中列名            javaType 指定关联的类型         -->        <association property="" column="" javaType=""></association></resultMap>
```

 根据上面的说明，我们来看看之前的问题怎么解决？ 

先截图给出三个表的数据情况：

![](https://img-blog.csdn.net/20170515150252540?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](https://img-blog.csdn.net/20170515150303337?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![](https://img-blog.csdn.net/20170515150314685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

接着，我们来定义兴趣类，并改写之前的用户实体类

```
package com.majing.learning.mybatis.entity; public class Interests {	private int id;	private String name;	public int getId() {		return id;	}	public void setId(int id) {		this.id = id;	}	public String getName() {		return name;	}	public void setName(String name) {		this.name = name;	}	@Override	public String toString() {		return "Interests [id=" + id + ", name=" + name + "]";	}	}
```

```
package com.majing.learning.mybatis.entity; import java.util.List; public class User2 {	private int id;	private String name;	private String password;	private int age;	private List<Interests> interests;	private String author;		public List<Interests> getInterests() {		return interests;	}	public void setInterests(List<Interests> interests) {		this.interests = interests;	}	public String getAuthor() {		return author;	}	public void setAuthor(String author) {		this.author = author;	}	public int getId() {		return id;	}	public void setId(int id) {		this.id = id;	}	public String getName() {		return name;	}	public void setName(String name) {		this.name = name;	}	public String getPassword() {		return password;	}	public void setPassword(String password) {		this.password = password;	}	public int getAge() {		return age;	}	public void setAge(int age) {		this.age = age;	}	@Override	public String toString() {		return "User2 [id=" + id + ", name=" + name + ", password=" + password + ", age=" + age + ", interests=" + interests + ", author=" + author + "]";	}	}
```

 紧接着，我们改写 Dao 接口：

```
package com.majing.learning.mybatis.dao; import com.majing.learning.mybatis.entity.User2; public interface UserDao2 {	/**	 * 查询	 * @param userId	 * @return	 */	User2 findUserById (int userId);}
```

 然后，我们给相关实体类创建一下别名：

```
<typeAliases>		<typeAlias alias="UserWithInterests" type="com.majing.learning.mybatis.entity.User2"/>		<typeAlias alias="Interests" type="com.majing.learning.mybatis.entity.Interests"/></typeAliases>
```

然后再创建一个对应跟 UserDao2 对应的 Mapper 映射文件：

```
<?xml version="1.0" encoding="UTF-8" ?>   <!DOCTYPE mapper   PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  "http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"><mapper namespace="com.majing.learning.mybatis.dao.UserDao2"> 	<resultMap type="UserWithInterests" id="UserWithInterestsMap">		<id property="id" column="id"/>		<result property="name" column="name"/>		<result property="password" column="password"/>		<result property="age" column="age"/>		<collection property="interests" column="interestname" ofType="Interests">			<id property="id" column="interestid"/>			<result property="name" column="interestname"/>		</collection>	</resultMap> 	<select id="findUserById" resultMap="UserWithInterestsMap">		select a.*,c.* from user as a right join userinterests as b on a.id = b.userid right join interests as c on b.interestid = c.interestid where id = #{id}	</select> </mapper>
```

 最后将这个映射文件添加到<mappers>元素配置下：

```
<mappers>		<mapper resource="com\majing\learning\mybatis\dao\UserDaoMapper.xml" />		<mapper resource="com\majing\learning\mybatis\dao\UserDao2Mapper.xml" /></mappers>
```

 下面我们来写个测试代码来测试一下是否可以正常运行：

```
public class UserDaoTest3 {	@Test	public void testFindUserById(){		SqlSession sqlSession = getSessionFactory().openSession(true);  		UserDao2 userMapper = sqlSession.getMapper(UserDao2.class);   		User2 user = userMapper.findUserById(10);  		System.out.println("记录为："+user); 	} 	// Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互	private static SqlSessionFactory getSessionFactory() {		SqlSessionFactory sessionFactory = null;		String resource = "configuration.xml";		try {			sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource));		} catch (IOException e) {			e.printStackTrace();		}		return sessionFactory;	}}
```

 运行代码后，我们可以看到成功啦！

![](https://img-blog.csdn.net/20170515151054341?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

（3）sql  

sql 元素的意义，在于我们可以定义一串 SQL 语句，其他的语句可以通过引用它而达到复用它的目的。因为平时用到的也不多，这里就不多介绍了，感兴趣的朋友可以自行查阅资料了解一下。

（4）cache、cache-ref

关于缓存的这块逻辑还没有研究过，后面专门写篇文章针对缓存进行针对性的说明，敬请关注。

至此，关于 mapper 映射文件的相关配置和使用讲解的便差不多了，希望对大家有所帮助。



# Mybatis中SQL语句执行过程详解

2017 年 05 月 15 日 20:37:24 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 4222 文章标签： [数据库](https://so.csdn.net/so/search/s.do?q=%E6%95%B0%E6%8D%AE%E5%BA%93&t=blog)[mybatis](https://so.csdn.net/so/search/s.do?q=mybatis&t=blog)[架构](https://so.csdn.net/so/search/s.do?q=%E6%9E%B6%E6%9E%84&t=blog)[源码](https://so.csdn.net/so/search/s.do?q=%E6%BA%90%E7%A0%81&t=blog) 更多

分类专栏： [Mybatis](https://blog.csdn.net/andamajing/article/category/6902012) [Mybatis应用及原理探析](https://blog.csdn.net/andamajing/article/category/6902014) [深入了解Mybatis使用及实现原理](https://blog.csdn.net/andamajing/article/category/9268791)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72179560](https://blog.csdn.net/andamajing/article/details/72179560) 

前面的十来篇文章我们对 Mybatis 中的配置和使用已经进行了比较详细的说明，想了解的朋友可以查看一下我专栏中的其他文章。 

但是你对整个 SQL 语句操作的流程了解吗？如果你还不是很了解，那么可以继续往下看，如果你已经了解了，那么可以跳过啦![大笑](http://static.blog.csdn.net/xheditor/xheditor_emot/default/laugh.gif)（因为一大推的源码估计要看的你头晕啊！！！)

所有语句的执行都是通过 SqlSession 对象来操作的，SqlSession 是由 SqlSessionFactory 类生成的。 

首先根据配置文件来创建一个 SqlSessionFactory，然后调用 openSession 来获取一个 SqlSession。我们从时序图来看看可能会更加清晰：

![](https://img-blog.csdn.net/20170515173800117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

（1）生成 SqlSessionFactory 对象（默认实现是 DefaultSqlSessionFactory）的过程

```
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {    try {      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);      return build(parser.parse());    } catch (Exception e) {      throw ExceptionFactory.wrapException("Error building SqlSession.", e);    } finally {      ErrorContext.instance().reset();      try {        reader.close();      } catch (IOException e) {        // Intentionally ignore. Prefer previous error.      }    }  }
```

```
// Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互	private static SqlSessionFactory getSessionFactory() {		SqlSessionFactory sessionFactory = null;		String resource = "configuration.xml";		try {			sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource));		} catch (IOException e) {			e.printStackTrace();		}		return sessionFactory;	}
```

（2）获取 SqlSession 对象

通过调用 DefaultSqlSessionFactory 的 openSession()方法来获取 SqlSession 对象。

```
@Override  public SqlSession openSession() {    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);  }
```

```
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {    Transaction tx = null;    try {      final Environment environment = configuration.getEnvironment();      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);      final Executor executor = configuration.newExecutor(tx, execType);      return new DefaultSqlSession(configuration, executor, autoCommit);    } catch (Exception e) {      closeTransaction(tx); // may have fetched a connection so lets call close()      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);    } finally {      ErrorContext.instance().reset();    }  }
```

 通过代码可以看出，最终返回的是一个 DefaultSqlSession 实例对象。接下来就是根据这个 DefaultSqlSession 来获取对应的 Mapper 对象。

（3）获取 MapperProxy 对象

```
UserDao userMapper = sqlSession.getMapper(UserDao.class);  
```

 如上所示，通过 SqlSession 对象调用 getMapper()方法来获取相应的 Dao 接口实现。我们通过代码跟踪一直往下看：

DefaultSqlSession 类中： 

```
@Override  public <T> T getMapper(Class<T> type) {    return configuration.<T>getMapper(type, this);  }
```

Configuration 类中： 

```
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {    return mapperRegistry.getMapper(type, sqlSession);  }
```

 MapperRegistry 类中：

```
 @SuppressWarnings("unchecked")  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);    if (mapperProxyFactory == null) {      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");    }    try {      return mapperProxyFactory.newInstance(sqlSession);    } catch (Exception e) {      throw new BindingException("Error getting mapper instance. Cause: " + e, e);    }  }
```

 在 MapperRegistry 类中维护着一个 Map，这个 Map 中存储着每个 Mapper 类型和其对应的代理对象工厂类，如下定义所示：

```
private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<Class<?>, MapperProxyFactory<?>>();
```

 在 mybatis 初始化的过程中就根据配置文件<mappers>元素的配置，将相关的映射文件给加载到了内存，同时保存到了这个 knownMappers 中。这里，在调用 getMapper()的时候，就会从这个 knownMappers 中寻找该 Dao 接口，如果没有找到，就直接抛出异常，说明没有在配置文件中配置说明，如果获取到了，那么就拿出其对应的代理对象工厂类出来，并从工厂类中通过 newInstance()方法来获取一个代理对象。

MapperProxyFactory 类中： 

```
@SuppressWarnings("unchecked")  protected T newInstance(MapperProxy<T> mapperProxy) {    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);  }   public T newInstance(SqlSession sqlSession) {    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);    return newInstance(mapperProxy);  }
```

 从代码可以看出来，其实我们调用 sqlSession.getMapper(UserDao.class)方法的时候,返回的是一个和 UserDao 接口对应的 MapperProxy 代理对象。如下定义所示，MapperProxy 类是一个实现了 InvocationHandler 的代理类：

```
public class MapperProxy<T> implements InvocationHandler, Serializable
```

上面的代码，如果整理成时序图，如下所示：

![](https://img-blog.csdn.net/20170515184616400?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

拿到了 Dao 接口的代理对象后，我们应该就可以进行具体的增删改查了，我们继续往下看。

（4）Executor 对象

当拿到了 UserDao 对象（其实是 MapperProxy 代理对象）后，我们调用 Dao 接口中定义的方法，如下所示：

```
User user = userMapper.findUserById(10);  
```

 这时候便调用了 MapperProxy 对象的 invoke 方法了；

```
@Override  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {    if (Object.class.equals(method.getDeclaringClass())) {      try {        return method.invoke(this, args);      } catch (Throwable t) {        throw ExceptionUtil.unwrapThrowable(t);      }    }    final MapperMethod mapperMethod = cachedMapperMethod(method);    return mapperMethod.execute(sqlSession, args);  }
```

 MapperMethod 类中：

```
public Object execute(SqlSession sqlSession, Object[] args) {    Object result;    switch (command.getType()) {      case INSERT: {    	Object param = method.convertArgsToSqlCommandParam(args);        result = rowCountResult(sqlSession.insert(command.getName(), param));        break;      }      case UPDATE: {        Object param = method.convertArgsToSqlCommandParam(args);        result = rowCountResult(sqlSession.update(command.getName(), param));        break;      }      case DELETE: {        Object param = method.convertArgsToSqlCommandParam(args);        result = rowCountResult(sqlSession.delete(command.getName(), param));        break;      }      case SELECT:        if (method.returnsVoid() && method.hasResultHandler()) {          executeWithResultHandler(sqlSession, args);          result = null;        } else if (method.returnsMany()) {          result = executeForMany(sqlSession, args);        } else if (method.returnsMap()) {          result = executeForMap(sqlSession, args);        } else if (method.returnsCursor()) {          result = executeForCursor(sqlSession, args);        } else {          Object param = method.convertArgsToSqlCommandParam(args);          result = sqlSession.selectOne(command.getName(), param);        }        break;      case FLUSH:        result = sqlSession.flushStatements();        break;      default:        throw new BindingException("Unknown execution method for: " + command.getName());    }    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {      throw new BindingException("Mapper method '" + command.getName()           + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");    }    return result;  }
```

 从上面这个方法实现上可以看出，已经根据执行方法(CRUD)进行了不同的处理，我们简单看一个方法 executeForMany，代码如下所示：

```
private <E> Object executeForMany(SqlSession sqlSession, Object[] args) {    List<E> result;    Object param = method.convertArgsToSqlCommandParam(args);    if (method.hasRowBounds()) {      RowBounds rowBounds = method.extractRowBounds(args);      result = sqlSession.<E>selectList(command.getName(), param, rowBounds);    } else {      result = sqlSession.<E>selectList(command.getName(), param);    }    // issue #510 Collections & arrays support    if (!method.getReturnType().isAssignableFrom(result.getClass())) {      if (method.getReturnType().isArray()) {        return convertToArray(result);      } else {        return convertToDeclaredCollection(sqlSession.getConfiguration(), result);      }    }    return result;  }
```

 通过观察这些代码，发现最终的实现都是通过 sqlSession 对象来进行操作的。我们继续往里看，看看 selectList 方法：

```
@Override  public <E> List<E> selectList(String statement) {    return this.selectList(statement, null);  }   @Override  public <E> List<E> selectList(String statement, Object parameter) {    return this.selectList(statement, parameter, RowBounds.DEFAULT);  }   @Override  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {    try {      MappedStatement ms = configuration.getMappedStatement(statement);      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);    } catch (Exception e) {      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);    } finally {      ErrorContext.instance().reset();    }  }
```

 可以看到，内部是把查询操作委托给了一个 Executor 对象（即 executor.query()），Executor 是一个接口，mybatis 为其实现了一个抽象基类 BaseExecutor，我们跟踪上面的代码中的 query 方法继续往里看：

BaseExecutor 类中： 

```
@Override  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {    BoundSql boundSql = ms.getBoundSql(parameter);    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);    return query(ms, parameter, rowBounds, resultHandler, key, boundSql); }   @SuppressWarnings("unchecked")  @Override  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());    if (closed) {      throw new ExecutorException("Executor was closed.");    }    if (queryStack == 0 && ms.isFlushCacheRequired()) {      clearLocalCache();    }    List<E> list;    try {      queryStack++;      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;      if (list != null) {        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);      } else {        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);      }    } finally {      queryStack--;    }    if (queryStack == 0) {      for (DeferredLoad deferredLoad : deferredLoads) {        deferredLoad.load();      }      // issue #601      deferredLoads.clear();      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {        // issue #482        clearLocalCache();      }    }    return list;  }
```

 在上面的方法中，我们看到当 list==null 的时候会调用 queryFromDatabase()方法，这个方法如下：

```
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {    List<E> list;    localCache.putObject(key, EXECUTION_PLACEHOLDER);    try {      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);    } finally {      localCache.removeObject(key);    }    localCache.putObject(key, list);    if (ms.getStatementType() == StatementType.CALLABLE) {      localOutputParameterCache.putObject(key, parameter);    }    return list;  }
```

 然后会调用 doQuery()方法，BaseExecutor 中的 doQuery 方法定义成了抽象方法，由具体的继承类进行个性化的实现。这里，我们拿 mybatis 中默认使用的 SimpleExecutor 来看看：

SimpleExecutor 类中：

```
 @Override  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {    Statement stmt = null;    try {      Configuration configuration = ms.getConfiguration();      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);      stmt = prepareStatement(handler, ms.getStatementLog());      return handler.<E>query(stmt, resultHandler);    } finally {      closeStatement(stmt);    }  }
```

 从这个方法可以看到，首先根据调用 Configuration 类的 newStatementHandler 方法来获取一个 sql 操作对象：

Configuration 类中：

```
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);    return statementHandler;  }
```

RoutingStatementHandler 类中： 

```
public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {     switch (ms.getStatementType()) {      case STATEMENT:        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);        break;      case PREPARED:        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);        break;      case CALLABLE:        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);        break;      default:        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());    }   }
```

 可以看到，这里根据配置来创建 Statement、PreparedStatement 或者 CallableStatement 三者之中的一个。然后调用相应的方法，如 query（）方法。以 SimpleStatementHandler 为例，我们看看具体的 sql 操作：

```
 @Override  public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {    String sql = boundSql.getSql();    statement.execute(sql);    return resultSetHandler.<E>handleResultSets(statement);  }
```

 看到这里，我们终于看到了黎明的曙光，因为这里已经看到 Jdbc 中的数据库操作代码了，即 statement.execute(sql)。在查询完之后使用 resultSetHandler 来进行查询结果集的处理。

上面的代码的整体流程图大概如下所示：

![](https://img-blog.csdn.net/20170515203114001?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

至此，一次完整的 sql 解析和处理过程便讲解完毕了，感兴趣的可以自己对着源码看看。