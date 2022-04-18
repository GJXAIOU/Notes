## Mybatis架构与原理


### MyBatis功能架构设计

![](https://mmbiz.qpic.cn/mmbiz/eQPyBffYbueR2DuLeqaeaJE6qIDcYN5TOuFfBibjs1JNBnPQbLiaz3V9hsgEaaYvcV2mjhZVTWJSDFBhHvKZ9wFw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**功能架构讲解：**

我们把 Mybatis 的功能架构分为三层：

*   API 接口层：提供给外部使用的接口 API，开发人员通过这些本地 API 来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。

*   数据处理层：负责具体的 SQL 查找、SQL 解析、SQL 执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。

*   基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

### 框架架构

![](https://mmbiz.qpic.cn/mmbiz/eQPyBffYbueR2DuLeqaeaJE6qIDcYN5TcsAp6rKT4zsiadrvrPX2Micu4SNXZEw26uZUoSe2G1eaV20Q1XSTvMXg/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**框架架构讲解：**

这张图从上往下看。MyBatis 的初始化，会从 mybatis-config.xml 配置文件，解析构造成 Configuration 这个类，就是图中的红框。

1.  加载配置：配置来源于两个地方，一处是配置文件，一处是 Java 代码的注解，将 SQL 的配置信息加载成为一个个 MappedStatement 对象（包括了传入参数映射配置、执行的 SQL 语句、结果映射配置），存储在内存中。

2.  SQL 解析：当 API 接口层接收到调用请求时，会接收到传入 SQL 的 ID 和传入对象（可以是 Map、JavaBean 或者基本数据类型），Mybatis 会根据 SQL 的 ID 找到对应的 MappedStatement，然后根据传入参数对象对 MappedStatement 进行解析，解析后可以得到最终要执行的 SQL 语句和参数。

3.  SQL 执行：将最终得到的 SQL 和参数拿到数据库进行执行，得到操作数据库的结果。

4.  结果映射：将操作数据库的结果按照映射的配置进行转换，可以转换成 HashMap、JavaBean 或者基本数据类型，并将最终结果返回。

### MyBatis核心类

**1、SqlSessionFactoryBuilder**

每一个 MyBatis 的应用程序的入口是 SqlSessionFactoryBuilder。

它的作用是通过 XML 配置文件创建 Configuration 对象（当然也可以在程序中自行创建），然后通过 build 方法创建 SqlSessionFactory 对象。没有必要每次访问 Mybatis 就创建一次 SqlSessionFactoryBuilder，通常的做法是创建一个全局的对象就可以了。

示例程序如下：

```
private static SqlSessionFactoryBuilder sqlSessionFactoryBuilder;private static SqlSessionFactory sqlSessionFactory;private static void init() throws IOException {    String resource = "mybatis-config.xml";    Reader reader = Resources.getResourceAsReader(resource);    sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();    sqlSessionFactory = sqlSessionFactoryBuilder.build(reader);}
```

org.apache.ibatis.session.Configuration 是 mybatis 初始化的核心。

mybatis-config.xml 中的配置，最后会解析 xml 成 Configuration 这个类。

SqlSessionFactoryBuilder 根据传入的数据流(XML)生成 Configuration 对象，然后根据 Configuration 对象创建默认的 SqlSessionFactory 实例。

**2、SqlSessionFactory对象由SqlSessionFactoryBuilder创建：**

它的主要功能是创建 SqlSession 对象，和 SqlSessionFactoryBuilder 对象一样，没有必要每次访问 Mybatis 就创建一次 SqlSessionFactory，通常的做法是创建一个全局的对象就可以了。SqlSessionFactory 对象一个必要的属性是 Configuration 对象，它是保存 Mybatis 全局配置的一个配置对象，通常由 SqlSessionFactoryBuilder 从 XML 配置文件创建。

这里给出一个简单的示例：

```
<?xml version="1.0" encoding="UTF-8" ?><!DOCTYPE configuration PUBLIC    "-//mybatis.org//DTD Config 3.0//EN"   "http://mybatis.org/dtd/mybatis-3-config.dtd"><configuration>   <!-- 配置别名 -->   <typeAliases>       <typeAlias type="org.iMybatis.abc.dao.UserDao" alias="UserDao" />       <typeAlias type="org.iMybatis.abc.dto.UserDto" alias="UserDto" />   </typeAliases>   <!-- 配置环境变量 -->   <environments default="development">       <environment id="development">           <transactionManager type="JDBC" />           <dataSource type="POOLED">               <property name="driver" value="com.mysql.jdbc.Driver" />               <property name="url" value="jdbc:mysql://127.0.0.1:3306/iMybatis?characterEncoding=GBK" />               <property name="username" value="iMybatis" />               <property name="password" value="iMybatis" />           </dataSource>       </environment>   </environments>   <!-- 配置mappers -->   <mappers>       <mapper resource="org/iMybatis/abc/dao/UserDao.xml" />   </mappers></configuration>
```

**3、SqlSession**

SqlSession 对象的主要功能是完成一次数据库的访问和结果的映射，它类似于数据库的 session 概念，由于不是线程安全的，所以 SqlSession 对象的作用域需限制方法内。

SqlSession 的默认实现类是 DefaultSqlSession，它有两个必须配置的属性：Configuration 和 Executor。Configuration 前文已经描述这里不再多说。SqlSession 对数据库的操作都是通过 Executor 来完成的。

SqlSession ：默认创建 DefaultSqlSession 并且开启一级缓存，创建执行器 、赋值。
SqlSession 有一个重要的方法 getMapper，顾名思义，这个方式是用来获取 Mapper 对象的。什么是 Mapper 对象？

根据 Mybatis 的官方手册，应用程序除了要初始并启动 Mybatis 之外，还需要定义一些接口，接口里定义访问数据库的方法，存放接口的包路径下需要放置同名的 XML 配置文件。

SqlSession 的 getMapper 方法是联系应用程序和 Mybatis 纽带，应用程序访问 getMapper 时，Mybatis 会根据传入的接口类型和对应的 XML 配置文件生成一个代理对象，这个代理对象就叫 Mapper 对象。应用程序获得 Mapper 对象后，就应该通过这个 Mapper 对象来访问 Mybatis 的 SqlSession 对象，这样就达到里插入到 Mybatis 流程的目的。

```
SqlSession session= sqlSessionFactory.openSession();  UserDao userDao = session.getMapper(UserDao.class);  UserDto user = new UserDto();  user.setUsername("iMybatis");  List<UserDto> users = userDao.queryUsers(user);  public interface UserDao {    public List<UserDto> queryUsers(UserDto user) throws Exception;}<?xml version="1.0" encoding="UTF-8" ?>  <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  <mapper namespace="org.iMybatis.abc.dao.UserDao">      <select id="queryUsers" parameterType="UserDto" resultType="UserDto"          useCache="false">          <![CDATA[         select * from t_user t where t.username = #{username}         ]]>      </select>  </mapper>
```

**4、Executor**

Executor 对象在创建 Configuration 对象的时候创建，并且缓存在 Configuration 对象里。Executor 对象的主要功能是调用 StatementHandler 访问数据库，并将查询结果存入缓存中（如果配置了缓存的话）。

**5、StatementHandler**

StatementHandler 是真正访问数据库的地方，并调用 ResultSetHandler 处理查询结果。

**6、ResultSetHandler**

处理查询结果。

### MyBatis成员层次&职责

![](https://mmbiz.qpic.cn/mmbiz/eQPyBffYbueR2DuLeqaeaJE6qIDcYN5TpvjaWCJsHGlrR1Ca74YxB2iasrEPibBpBLyBhJb4Omyod5P4ibCcNceqA/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1.  SqlSession 作为 MyBatis 工作的主要顶层 API，表示和数据库交互的会话，完成必要数据库增删改查功能

2.  Executor MyBatis 执行器，是 MyBatis 调度的核心，负责 SQL 语句的生成和查询缓存的维护

3.  StatementHandler 封装了 JDBC Statement 操作，负责对 JDBCstatement 的操作，如设置参数、将 Statement 结果集转换成 List 集合。

4.  ParameterHandler 负责对用户传递的参数转换成 JDBC Statement 所需要的参数

5.  ResultSetHandler *负责将 JDBC 返回的 ResultSet 结果集对象转换成 List 类型的集合；

6.  TypeHandler 负责 java 数据类型和 jdbc 数据类型之间的映射和转换

7.  MappedStatement MappedStatement 维护了一条

    节点的封
8.  SqlSource 负责根据用户传递的 parameterObject，动态地生成 SQL 语句，将信息封装到 BoundSql 对象中，并返回

9.  BoundSql 表示动态生成的 SQL 语句以及相应的参数信息

10.  Configuration MyBatis 所有的配置信息都维持在 Configuration 对象之中