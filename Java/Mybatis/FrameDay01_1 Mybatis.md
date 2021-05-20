---

---
# FrameDay01_1 MyBatis

[TOC]

## 一、MVC开发模式

作用：视图和逻辑分离 

- M（Model，模型）：实体类和业务和 dao

- V（view 视图）：现在一般使用 .js

- C: Controller 控制器，现在使用 servlet

- **项目实现步骤**：

  设计数据库 -> 再写实体类：用于封装数据 -> 持久层 DAO -> 业务逻辑 -> 控制器 -> 视图

  <img src="FrameDay01_1%20Mybatis.resource/MVC%E5%BC%80%E5%8F%91%E6%A8%A1%E5%BC%8F.png" alt="MVC开发模式" style="zoom:50%;" />

## 二、框架概述

- 框架：软件的半成品，为解决问题制定的一套约束，在提供功能基础上进行扩充。
- **框架中一些不能被封装的代码(变量)，需要使用框架者新建一个 xml 文件，在文件中添加变量内容**。
  - **需要建立特定位置和特定名称的配置文件**。
  - **需要使用 xml 解析技术和反射技术**。
  - 是 MySQL Mapper Framework for Java

- MyBatis 是数据访问层（DAO 层）框架，底层是对 JDBC 的封装。在使用 MyBatis 时不需要编写实现类，只需要写执行的 SQL 语句。


## 三、MyBatis 使用

### （一）环境搭建

- 步骤一：导入 jar

    ```java
    // 基本核心包
    mybatis.jar  // MyBatis 的核心包
    mysql-connector-java.jar   // MySQL 驱动包
    cglib.jar   // 动态代理包    
    asm.jar     // cglib 依赖包
    javassist-GA.jar   // cglib 依赖包（负责字节码解析的包）
        
    // 日志包    
    commons-logging.jar   // 日志包
    log4j.jar  // 日志包
    log4j-api.jar // 日志包
    log4j-core.jar // 日志包
    slf4j-api.jar // 日志包
    slf4j-log4j.jar // 日志包
    ```

- 步骤二：**在 src 下新建全局配置文件**(编写 JDBC 四个变量)
  - 没有名称和地址要求，示例名称为： mybatis.xml
  - **在全局配置文件中引入 DTD 或 schema**
  
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!--注意这里是 configuration ，下面为 mybatis-3-config -->
  <!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    
  <configuration>
    <!-- default 为引用 environment 的 id，即当前所使用的环境 -->
    <environments default="default">
    	<!-- 声明可以使用的环境 -->
    	<environment id="default">
    		<!-- 使用原生 JDBC 事务 -->
    		<transactionManager type="JDBC"></transactionManager>
    		<dataSource type="POOLED">
        		  <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        	          <property name="url" value="jdbc:mysql://localhost:3306/数据库名"/>
        		  <property name="username" value="root"/>
        	          <property name="password" value="GJXAIOU"/>
    		</dataSource>
    	</environment>
    </environments>
    
    <mappers>
        <!-- 配置实体类 Mapper 文件，一般使用 package -->
    	<mapper resource="com/gjxaiou/mapper/FlowerMapper.xml"/>
    </mappers>
  </configuration>
  ```
  
  **注：** mybatis.xml 中的 `<mappers>`中配置的 resource 是整个项目中的所有其他 `实体类Mapper.xml` 文件；
  
  mappers 标签下有许多 mapper 标签，每一个 mapper 标签中配置的都是一个独立的映射配置文件的路径，配置方式有以下几种。
  
  - 第一种：使用**相对路径**进行配置。
  
      ```xml
      <mappers>
          <mapper resource="org/mybatis/mappers/UserMapper.xml"/>
          <mapper resource="org/mybatis/mappers/ProductMapper.xml"/>
      </mappers>
      ```
  
  - 第二种：使用**绝对路径**进行配置。
  
      ```xml
      <mappers>
          <mapper url="file:///var/mappers/UserMapper.xml"/>
          <mapper url="file:///var/mappers/ProductMapper.xml"/>
      </mappers>
      ```
  
  - 第三种：使用**接口信息**进行配置。
  
      ```xml
      <mappers>
          <mapper class="org.mybatis.mappers.UserMapper"/>
          <mapper class="org.mybatis.mappers.ProductMapper"/>
      </mappers>
      ```
  
  - 第四种：使用接口所在包进行配置。
  
      ```xml
      <mappers>
          <package name="org.mybatis.mappers"/>
      </mappers>
      ```
  
- 步骤三：新建以 mapper 结尾的包，在包下新建：`实体类名+Mapper.xml`
  - 文件作用：编写需要执行的 SQL 命令
  - **把 xml 文件理解成实现类**；
   FlowerMapper.xml 文件内容为：【**注意抬头中的信息不同，将上面抬头中的 config 全部换为 mapper**】

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  <!-- namespace:理解成实现类的全路径(包名+类名)，用于绑定 Dao 接口（即面向接口编程），使用 namespace 之后就不用写实现类，业务逻辑会直接通过这个绑定寻找到对应点的 SQL 语句进行对应的数据处理 -->
  <mapper namespace="a.b" >
  	<!-- 如果方法返回值是list，在resultType中写List的泛型，因为mybatis
  		对jdbc封装，一行一行读取数据-->
  	<!--id:表示方法名; parameterType:定义参数类型;resultType:返回值类型 -->
  	<select id="selAll" resultType="com.gjxaiou.pojo.Flower">
  		select * from flower
  	</select>
  </mapper>
  ```

- 步骤四：测试结果(只有在单独使用 mybatis 时使用，最后 ssm 整合时下面代码不需要编写)

    ```java
    import com.gjxaiou.pojo.Flower;
    
    public class Test {
    	public static void main(String[] args) throws IOException {
    		InputStream is = Resources.getResourceAsStream("mybatis.xml");
    		// 使用工厂设计模式
    		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
    		// 生产 SqlSession
    		SqlSession session=factory.openSession();
    		
    		List<Flower> list = session.selectList("a.b.selAll");
    		for (Flower flower : list) {
    			System.out.println(flower.toString());
    		}
    		
    		session.close();
    	}
    }
    ```

### （二）环境搭建详解

**全局配置文件中内容**
- `<transactionManager/>` type 属性可取值如下：
  - JDBC：事务管理使用 JDBC 原生事务管理方式；
  - MANAGED：把事务管理转交给其他容器，相当于设置原生 JDBC 事务`setAutoMapping(false)`；
- `<dataSouce/>`type 属性可取值如下：
  - POOLED：使用数据库连接池；
  - UNPOOLED：不使用数据库连接池，和直接使用 JDBC 一样；
  - JNDI：是 Java 命名目录接口技术；

## 四、数据库连接池

- 在内存中开辟一块空间，存放多个数据库连接对象；

- JDBC Tomcat Pool，直接由 tomcat 产生数据库连接池； 

- 数据库连接池中有很多连接，他们可能处于 Active 或者 Idle 等状态；
  - 活跃状态（active）：当前连接对象被应用程序使用中
  - 空闲状态（Idle）：等待应用程序使用
  
- **实现 JDBC tomcat Pool 的步骤**：
  - 在 web 项目的 META-INF 中存放 context.xml，在 context.xml 编写数据库连接池相关属性
  - 把项目发布到 tomcat 中，则数据库连接池产生了
  
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <Context>
  	<Resource
  		driverClassName="com.mysql.jdbc.Driver"
  		url="jdbc:mysql://localhost:3306/lianxi"
  		username="root"
  		password="gjxaiou"
  		maxActive="50"
  		maxIdle="20"
  		name="test"
  		auth="Container"
  		maxWait="10000"
  		type="javax.sql.DataSource"
  	/>
  </Context>
  ```
  
 - 可以在 Java 中使用 JNDI 获取数据库连接池中对象

  - Context:上下文接口.context.xml 文件对象类型；代码见下面

  - 当关闭连接对象时，把连接对象归还给数据库连接池，把状态改变成 Idle

```java
@WebServlet("/pool")
public class DemoServlet extends HttpServlet {
	@Override
	protected void service(HttpServletRequest req, HttpServletResponse res) throws ServletException, IOException {
  	  try {
  		Context cxt = new InitialContext();
  		DataSource ds = (DataSource) cxt.lookup("java:comp/env/test");
  		Connection conn = ds.getConnection();
  		PreparedStatement ps = conn.prepareStatement("select * from flower");
  		ResultSet rs = ps.executeQuery();
  		res.setContentType("text/html;charset=utf-8");
  		PrintWriter out = res.getWriter();
  		while(rs.next()){
  			out.print(rs.getInt(1)+"&nbsp;&nbsp;&nbsp;&nbsp;"+rs.getString(2)+"<br/>");
  		}
  		out.flush();
  		out.close();
  		rs.close();
  	} catch (NamingException e) {
  		e.printStackTrace();
  	} catch (SQLException e) {
  		e.printStackTrace();
  	}
  }
}
```


## 五、三种查询方式

- `selectList()`  返回值为 `List<resultType  属性控制>`
适用于查询结果都需要遍历的需求

- `selectOne()` 返回值 `Object`
适用于返回结果只是变量或一行数据时

- `selectMap()` 返回值 `Map<key，resultType  属性控制>`
  适用于需要在查询结果中通过某列的值取到这行数据的需求
```java
public class Test {
	public static void main(String[] args) throws IOException {
        /** MyBatis 默认不加载配置文件，因此需要先加载配置文件，返回整个配置文件的流对象；
        * 在数据访问层处理异常和在控制器中处理异常，一般在 service 中只抛出异常；
        */
		InputStream is = Resources.getResourceAsStream("mybatis.xml");
		// 使用工厂设计模式
		// 前面是工厂  实例化工厂对象时使用的是构建者设计模式   它的名称标志:后面有Builder 
		// 构建者设计模式意义: 简化对象实例化过程
		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
		// 生产 SqlSession, 整个 sqlsession 就是 MyBatis 中 API 封装的对象，增删改查都在里面
		SqlSession session=factory.openSession();
		
		// selectList
		List<Flower> list = session.selectList("a.b.selAll");
		for (Flower flower : list) {
			System.out.println(flower.toString());
		}
		
		// selectOne
		int count = session.selectOne("a.b.selById");
		System.out.println(count);
		
		// selectMap
		// 把数据库中哪个列的值当作 map 的 key
		Map<Object, Object> map = session.selectMap("a.b.c", "name123");
		System.out.println(map);
		
		session.close();
	}
}
```

对应的 实体类 Mapper.xml 文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
  
<mapper namespace="a.b" >
	<select id="selAll" resultType="com.gjxaiou.pojo.Flower">
		select id,name name123,price,production from flower
	</select>
	
	// 这里的 int 相当于 Ingeter
	<select id="selById" resultType="int">
		select count(*) from flower
	</select>
	
	<select id="c" resultType="com.gjxaiou.pojo.Flower">
		select id,name name123,price,production from flower
	</select>
	
</mapper>
```



## 六、Log4J
### （一）基本配置

- 导入 log4j-xxx.jar
- 在 src 下新建 log4j.properties(路径和名称都不允许改变)

```java
// log4j 五大输出级别：fatal > error > warn > info > debug 只有大于配置的级别的日志才输出
// 遇到什么级别才输出以及输出位置：调试信息输出， 输出到控制台，输出到 log 文件
log4j.rootCategory=DEBUG, CONSOLE, LOGFILE
// 向控制台输出相关的配置
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout 
// 控制台日志输出格式    
log4j.appender.CONSOLE.layout.ConversionPattern=%C %d{YYYY-MM-dd hh:mm:ss}%m  %n

// 向日志文件输出的相关的配置
log4j.appender.LOGFILE=org.apache.log4j.FileAppender 
// 日志文件位置和名称    
log4j.appender.LOGFILE.File=E:/my.log 
// true 表示往后追加，false 表示清空再追加    
log4j.appender.LOGFILE.Append=true log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout 
// 日志文件输出格式    
log4j.appender.LOGFILE.layout.ConversionPattern=%C	%m %L %n
```

### （二） 基本使用

- 使用方法一：

    ```java
    // 省略 Test 类的内容
    Logger  logger = Logger.getLogger(Test.class);
    // 这里以 debug 为例，具体的方法看上面输出级别调用不同的方法
    logger.debug("这是调试信息"); 
    ```

- 使用方法二：
  在 catch 中 ： `logger.error(e.getMessage());` 即可，注意在 log4j 配置文件中加上相关的表达式；

- pattern 中常用几个表达式（区分大小写，多个共同使用时中间添加空格）
  - %C 输出包名+类名；
  - %d{YYYY-MM-dd  HH:mm:ss} 时间；
  - %L 行号
  - %m 信息
  - %n 换行


## 七、`<settings>`标签

- 在 mybatis 全局配置文件中通过 `<settings>` 标签控制 MyBatis 全局开关，**该标签必须位于所有配置的最前面**。

- 在 mybatis.xml 中开启 log4j 命令如下：
  -  必须保证有 log4j.jar
  -  在 src 下有 log4j.properties
  
  ```xml
  <settings>
      <setting  name="logImpl"  value="LOG4J"/>
  </settings>
  ```
  
- log4j 中可以输出指定内容的日志(控制某个局部内容的日志级别) ：直接在 log4j.properties 中设置；
    -  命名级别(包级别): `<mapper>namespace 属性中除了最后一个类名`
      例如 `namespace=”com.gjxaiou.mapper.PeopleMapper”` 其中包级别为 `com.gjxaiou.mapper` ，需要在 `log4j.propeties` 中做两件事情  
      - 先在总体级别调成 Error，这样可以不输出无用信息
      -  在设置某个指定位置级别为 DEBUG
      
      ```properties
      ## -----log4j.properties--------
      log4j.rootCategory=ERROR
      log4j.logger.com.gjxaiou.mapper=DEBUG   ## 只有这个包输出是 Debug 级别，其它都是 Error 级别
      ```
      
      - 类级别

        namespace 属性值 ，相当于namespace 类名

      - 方法级别

        使用 namespace 属性值+标签 id 属性值

## 八、 parameterType 属性（Mybatis 中参数设置）

**传递多个参数时候，可以使用对象或者 map，和第二部分的多参数实现方法**；

- 在 XXXMapper.xml  中`<select>` `<delete>`等标签的 `parameterType` 可以控制参数类型（例如可以传入 select 语句的参数，控制输入参数的类型）

- **SqlSession 的 `selectList()` 和 `selectOne()` 的第二个参数和 `selectMap()` 的第三个参数都表示方法的参数** 。

  ```java
  // 示例代码
  People  p  =  session.selectOne("a.b.selById",1);
  System.out.println(p);
  ```

  - 在 实例名Mapper.xml  中可以通过`#{}`获取参数
    - `parameterType`  控制参数类型

    - `#{}`  获取参数内容
      - 使用索引，从 0 开始 `#{0}`表示第一个参数（尽量不使用这个方法），或者使用`#{param1}`，表示第一个参数
      - 如果**只有一个参数(其参数应该是基本数据类型或 String)**，mybatis 对 `#{}` 里面内容没有要求只要写内容即可。
      - 如果参数是对象`#{属性名}`
      -  如果参数是 map， 写成`#{key}` ：当需要传递多个参数时候，目前只能使用 map 或者对象
      
      ```xml
      <select id="selById"
        resultType="com.gjxaiou.pojo.People"  parameterType="int">
        select * from people where id=#{0}
      </select>
      ```

- `#{}` 和 `${}` 的区别

  - `#{}` 获取参数的内容，支持索引获取，或者使用 param1 获取指定位置参数，并且 SQL 使用 `?` 占位符
  - `${}` 字符串拼接，不使用?，默认找`${内容}`内容的 get/set 方法，如果写数字，就是一个数字

  ```xml
  <!------PeopleMapper.xml----->
  <select id="test" resultType="com.gjxaiou.pojo.People" parameterType="com.gjxaiou.pojo.People"> 
    select * from people where id =  ${id}
  </select>
  ```

  ```java
  // --------Test.java--------
  People peo =new People();
  peo.setId(1);
  People p = session.selectOne("a.b.selById",people);
  ```

- 如果在 XML 文件中出现`<`、`>`  或者 `""` 等特殊字符时可以使用 XML 文件转义标签(XML 自身的)，格式为：`<![CDATA[ 内 容 ]]>`

## 九、MyBatis 中实现 MySQL 分页

不允许在关键字前后进行数学运算，需要在代码中计算完成后传递到 mapper.xml  中；

- Java 中代码为：

    ```java
    // 显示几个
    int pageSize = 2;
    // 第几页
    int pageNumber = 2;
    // 如果希望传递多个参数，可以使用对象或 map
    Map<String,Object> map = new HashMap<>();
    map.put("pageSize", pageSize);
    map.put("pageStart", pageSize*(pageNumber-1));
    List<People> p = session.selectList("a.b.page",map);
    ```

- mapper.xml 中代码为：

    ```xml
    <select id="page" resultType="com.gjxaiou.pojo.People" parameterType="map">
        select * from people limit #{pageStart},#{pageSize}
    </select>
    ```

## 十、typeAliases 别名

**别名配置必须在 `<environments>`标签的前面**，分为系统内置别名、给某个类的别名、给某个包下所有类的别名。

- 系统内置别名：就是把类型全小写（见文档 ）

- 给某个类起别名

    ```xml
    <!--在 MyBatis 配置文件 mybatis.xml 中使用 alias="XXX" 进行自定义-->
    <typeAliases>
        <typeAlias type="com.gjxaiou.pojo.People" alias="peo"/>
    </typeAliases>
    
    
    <!--在各个 mapper.xml 中可以引用，这里就是使用 peo 引用 People 类-->
    <select id="page" resultType="peo" parameterType="map">
        select * from people limit #{pageStart},#{pageSize}
    </select>
    ```

- **直接给某个包下所有类起别名，别名为类名，区分大小写**

  ```xml
  <!--首先在 mybatis.xml 中进行配置-->
  <typeAliases>
      <package name="com.gjxaiou.pojo" />
  </typeAliases>
      
  <!--在各个 mapper.xml 中进行引用-->    
  <select id="page" resultType="People" parameterType="map">
      select * from people limit #{pageStart},#{pageSize}
  </select>    
  ```

## 十一、MyBatis 实现新增/修改/删除

- **在 mybatis 中默认是关闭了 JDBC 的自动提交功能**
  -  每一个 SqlSession 默认都是不自动提交事务
  -  可以使用 `session.commit()` 提交事务，也可以使用 `openSession(true);` 自动提交底层为：`.setAutoCommit(true);`
- MyBatis 底层是对 JDBC 的封装

  JDBC 中 `executeUpdate()` 执行新增，删除，修改 SQL，方法的返回值 int，表示受影响的行数。**因此 MyBatis 中 `<insert> <delete> <update>` 标签没有 resultType 属性，认为返回值都是 int**。

- 在 openSession() 时 Mybatis 会创建 SqlSession 时同时创建一个 Transaction(事务对象)，同时 autoCommit 都为 false
- 如果出现异常，应该使用 `session.rollback()` 回滚事务.
### （一）实现新增的步骤

- 在 mapper.xml  中提供 `<insert>` 标签，标签没有返回值类型
- 通过 `session.insert()` 调用新增方法
mapper.xml 值为；

```xml
<insert id="ins" parameterType="People">
insert into people values(default,#{name},#{age})
</insert>
```

```java
int index1 = session.insert("a.b.ins", p);
if(index1>0){
   System.out.println("成功");
}else{
  System.out.println("失败");
}
```


### （二）实现修改的步骤

- 在 mapper.xml 中提供 `<update>` 标签
```java
<update id="upd" parameterType="People">
  update people set name = #{name} where id = #{id}
</update>
```
- 编写代码
```java
People peo = new People();
peo.setId(3);
peo.setName("王五");
int index = session.update("a.b.upd", peo);
if(index>0){
System.out.println("成功");
}else{
System.out.println("失败");
}
session.commit();
```


###  （三）实现删除步骤

- 在 mapper.xml  提供`<delete>` 标签
```xml
<delete id="del" parameterType="int">
    delete from people where id = #{0}
</delete>
```

- 编写代码
```java
 int  del  =  session.delete("a.b.del",3);

if(del>0){
   System.out.println("成功");
}else{
   System.out.println("失败");
}

session.commit();
```

