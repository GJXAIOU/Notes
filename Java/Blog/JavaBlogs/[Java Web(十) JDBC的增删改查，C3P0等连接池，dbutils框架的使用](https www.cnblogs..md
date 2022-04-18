# [Java Web(十) JDBC的增删改查，C3P0等连接池，dbutils框架的使用](https://www.cnblogs.com/whgk/p/6442768.html)

前面做了一个非常垃圾的小 demo，真的无法直面它，菜的抠脚啊，真的菜，好好努力把。菜鸡。

--WZY

一、JDBC 是什么？

Java Data Base Connectivity，java 数据库连接，在需要存储一些数据，或者拿到一些数据的时候，就需要往数据库里存取数据，那么 java 如何连接数据库呢？需要哪些步骤？

1、注册驱动

什么是驱动？

驱动就是 JDBC 实现类，通俗点讲，就是能够连接到数据库功能的东西就是驱动，由于市面上有很多数据库，Oracle、MySql 等等，所以 java 就有一个连接数据库的实现规范接口，定义一系列的连接数据库接口(java.sql.Driver 接口)，但是不提供实现，而每个数据库厂家来提供这些接口的具体实现，这样一来，不管使用的是什么数据库，我们开发者写的代码都是相同的，就不必因为数据库的不同，而写法不同，唯一的不同就是数据库驱动不一样，使用 mysql，那么就必须使用 mysql 的驱动，使用 Oracle 就必须使用 oracle 的驱动实现类。　看下面 mysql 连接数据的原理图，看看驱动是在哪里，起什么作用。就明白了什么是驱动了。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226115733632-971333249.png)　

DriverManager，一个工具类，是用于操作管理 JDBC 实现类的，

原始写法：DriverManager.register(new Driver());　　//因为使用的是 MySql，所以在导包时就需要导入 com.mysql.jdbc.Driver

现在写法：Class.forName("com.mysql.jdbc.Driver");　　//不用导包，会执行 com.mysql.jdbc.Driver 类中的静态代码块，其静态代码块的内容为

static {
try {                      

java.sql.DriverManager.registerDriver(new Driver());

} catch (SQLException E) {
throw new RuntimeException("Can't register driver!");
}
}　　

会发现第二种加载驱动的方法的底层其实就是第一种加载驱动。为什么要这样呢？原因很简单，　第一种是硬编程，直接将数据库驱动给写死了，无法扩展，如果使用第一种，那么连接的数据库只能是 mysql，因为导包导的是 mysql 的驱动包，如果换成 Oracle，就会报错，需要在代码中将 Oracle 的驱动包导入，这样很麻烦，而第二种写法就不一样了，第二种是使用的字符串方法注册驱动的，我们只需要将该字符串提取到一个配置文件中，以后想换成 oracle 数据库，只需要将该字符串换成 oracle 驱动的类全名即可，而不需要到代码中去修改什么东西。　

2、获得连接

使用 DriverManage 来获得连接，因为 DriverManager 是驱动实现类的管理者

Connection conn = DriverManager.getConnection(url,user,password);

url:确定数据库服务器的位置，端口号，数据库名

jdbc:mysql://localhost:3306/**db**　

user:登录名称，默认 root

password:密码，默认 root　　　

这里只是说 mysql，别的数据库，url 格式就不同了。

MySQL　　　　jdbc:mysql://localhost:3306/**db　　　　**默认端口是 3306，粗体为连接时使用的数据库名

Oracle　　　　 jdbc:oracle:thin:@localhost:1521:**db**　　默认端口号 1521

DB2　　　　　 jdbc:db2://localhost:6789/**db**　　　　　　默认端口号 6789

SQLServer　　jdbc:microsoft:sqlserver://localhost:1433;databaseName=**db**　　默认端口号 1433

SQLServer 2005　　jdbc:sqlserver://localhost:1433;databaseName=**db**　　默认端口号 1433

3、获取执行 sql 语句对象，PraparedStament 对象

通过 Connection 对象获取 Statement 或者 PraparedStament 对象(使用它)处理 sql

Statement

Statement st = conn.createStatement();　　//获取 sql 语句执行对象

st.excuteUpdate(sql);　　//执行增删改语句

st.excuteQuery(sql);　　//执行查询语句　　　　　　

sql 语句必须是完整的。

PraparedStatment

sql 语句可以不是完整的，可以将参数用?替代，然后在预编译后加入未知参数

PraparedStatment ps = conn.prapareStatement(sql);　　//获取 sql 语句执行对象 praparedStatment

赋值

ps.setInt(Index,value);　　ps.setString(index,value);　　//可以设置很多中类型，index 从 1 开始，代表 sql 语句中的第几个未知参数，

ps.excuteUpdate();　　//执行增删改语句

ps.excuteQuery(sql);　　//执行查询语句

这两个的区别，常使用的是 PraparedStatment 对象，因为它可以预编译，效率高，可以设置参数等等优点

4、获得结果集对象

int count = ps.excuteUpdate();　　　//执行增删改的 sql 语句时，返回一个 int 类型的整数，代表数据库表影响的行数，

Result result = ps.excuteQuery();　　//执行查询 sql 语句时，返回一个结果集对象，该对象装着所有查询到的数据信息，一行一行的存储数据库表信息。

5、处理结果

对查询到的 Result 结果进行处理，拿到所有数据，并封装成对象。

while(rs.next()){

获取行数据的第一种方式

rs.getString(index);//index 代表第几列，从 1 开始

获取行数据的第二中方式

rs.getString(string);　　//string：代表字段名称。

}

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226130549757-272947954.png)

总结：java 的 JDBC 就分为 5 步，4 个属性

属性：driver、url、user、password

五步：

注册驱动、获取连接、获取执行 sql 语句对象、获取结果集对象、处理结果。

二、JDBC 的 CURD 操作

创建（Create）、更新（Update）、读取（Retrieve）和删除（Delete）操作

查询所有(读取 Retrieve)

findAll()

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1     @Test
2     public void findAll() throws Exception{ 3         //1 注册驱动
4         Class.forName("com.mysql.jdbc.Driver");
5         //2 获得连接
6         Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
7         //3 语句执行者，sql 语句
8         Statement st = conn.praparedStatement("select * from t_user");
9         //4 执行查询语句
10         ResultSet rs = st.executeQuery(); 11         //5 处理数据 12         // * 如果查询多个使用，使用 while 循环进行所有数据获取 13         // * 技巧：如果查询结果最多 1 条，使用  if(rs.next()) {  查询到了 } else {  没有数据 }
14         while(rs.next()){ 15             int id = rs.getInt(1); 16             String username = rs.getString(2); 17             String password =rs.getString(3); 18             System.out.print(id + ", "); 19             System.out.print(username + ", "); 20 System.out.println(password); 21 } 22         //6 释放资源
23 rs.close(); 24 st.close(); 25 conn.close(); 26         
27     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

save(),增加操作(创建 Create)

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1     @Test
2     public void save() throws Exception{ 3         //1 注册驱动
4         Class.forName("com.mysql.jdbc.Driver");
5         //2 获得连接
6         Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
7         //3 语句执行者
8         Statement st = conn.praparedStatement("insert into t_user(username,password) values(?,?)");
9                 //3.1 赋值
10                 st.setString(1,"xiaoming"); 11                 st.setString(2,"123"); 12         //4 执行 DML 语句
13         int r = st.executeUpdate(); 14         
15         //5 处理数据
16 System.out.println(r); 17         
18         //6 释放资源 19         //rs.close();
20 st.close(); 21 conn.close(); 22     }        

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

update(),更新

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1     @Test
2     public void update() throws Exception{ 3         //1 注册驱动
4         Class.forName("com.mysql.jdbc.Driver");
5         //2 获得连接
6         Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
7         //3 语句执行者
8         Statement st = conn.praparedStatement("update t_user set username = ？ where id = ？ ");
9                 //3.1 赋值参数
10                 st.setString(1,"xiaoye"); 11                 st.setInt(2,2); 12         //4 执行 DML 语句
13         int r = st.executeUpdate(); 14         
15         //5 处理数据
16 System.out.println(r); 17         
18         //6 释放资源 19         //rs.close();
20 st.close(); 21 conn.close(); 22 }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

delete(),删除

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1     @Test
2     public void delete() throws Exception{ 3         //1 注册驱动
4         Class.forName("com.mysql.jdbc.Driver");
5         //2 获得连接
6         Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test", "root", "root");
7         //3 语句执行者
8         Statement st = conn.praparedStatement("delete from t_user where id = ?");
9                 //3.1 赋值参数
10                 st.setInt(1,2); 11         //4 执行 DML 语句
12         int r = st.executeUpdate(); 13         
14         //5 处理数据
15 System.out.println(r); 16         
17         //6 释放资源 18         //rs.close();
19 st.close(); 20 conn.close(); 21 }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

上面重复代码过多，所以使用一个获得连接的工具类，来帮我们获得连接，并且把四个属性提取出来，放在配置文件中

使用 jdbcInfo.properties(放在 src 下面即可)保存四个属性。以方便修改

jdbcInfo.properties

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

1 driver = com.mysql.jdbc.Driver 2 url = jdbc:mysql://localhost:3306/myums
3 user = root 4 password =root

写一个工具类，注册驱动，提供连接，就不必每次都重复写注册驱动，连接代码了

JdbcUtils.java

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1 public class JdbcUtils { 2     
3     private static String url; 4     private static String user; 5     private static String password; 6     static{
7         try { 8             
9             // 将具体参数存放配置文件中， xml，properties（key=value） 10             // 1 加载 properties 文件 src （类路径） -->  WEB-INF/classes 11             // 方式 1： 使用类加载 ClassLoader 的方式加载资源
12             InputStream is = JdbcUtils.class.getClassLoader().getResourceAsStream("jdbcInfo.properties"); 13             // 方式 2：使用 Class 对象加载，必须添加/，表示 src 14             // InputStream is = JdbcUtils.class.getResourceAsStream("/jdbcInfo.properties"); 15             // * 如果不使用/表示，从当前类所在的包下加载资源 16             //InputStream is = JdbcUtils.class.getResourceAsStream("/com/itheima/d_utils/jdbcInfo2.properties"); 17             // 2 解析
18             Properties props = new Properties(); 19 props.load(is); 20             
21             // 3 获得配置文件中数据
22             String driver = props.getProperty("driver"); 23             url = props.getProperty("url"); 24             user = props.getProperty("user"); 25             password = props.getProperty("password");; 26             
27             // 4 注册驱动
28 Class.forName(driver); 29         } catch (Exception e) { 30             throw new RuntimeException(e); 31 } 32 } 33     
34     
35     /**
36 * 获得连接 37 * @return
38      */
39     public static Connection getConnection(){ 40         try { 41             
42             Connection conn = DriverManager.getConnection(url, user, password); 43             return conn; //获得连接
44         } catch (Exception e) { 45             //将编译时异常 转换 运行时 ， 以后开发中 运行时异常使用比较多的。 
46 } 47 } 48     
49     /**
50 * 释放资源 51 * @param conn 52 * @param st 53 * @param rs 54      */
55     public static void closeResource(Connection conn,Statement st,ResultSet rs){ 56         try { 57             if (rs != null) { 58 rs.close(); 59 } 60         } catch (Exception e) { 61             throw new RuntimeException(e); 62         } finally{ 63             try { 64                 if (st != null) { 65 st.close(); 66 } 67             } catch (Exception e) { 68                 throw new RuntimeException(e); 69             } finally{ 70                 try { 71                     if (conn != null) { 72 conn.close(); 73 } 74                 } catch (Exception e) { 75                     throw new RuntimeException(e); 76 } 77 } 78 } 79     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

模版代码

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1     //模板代码
2     public void demo01(){ 3         // 0 提供变量
4         Connection conn = null;
5         Statement st = null;
6         ResultSet rs = null;
7         
8         try { 9             //1 获得连接
10             conn = JdbcUtils.getConnection(); 11             
12             
13             //2 获得语句执行者 14             //3 执行 sql 语句 15             //4 处理结果
16             
17         } catch (Exception e) { 18             throw new RuntimeException(e); 19         } finally{ 20             //end 释放资源
21 JdbcUtils.closeResource(conn, st, rs); 22 } 23     }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

三、连接池

在上面，我们在进行 CRUD 时，一直重复性的写一些代码，比如最开始的注册驱动，获取连接代码，一直重复写，通过编写一个获取连接的工具类后，解决了这个问题，但是又会出现新的问题，每进行一次操作，就会获取一个连接，用完之后，就销毁，就这样一直新建连接，销毁连接，新建，销毁，连接 Connection 创建与销毁 比较耗时的。所以应该要想办法解决这个问题。

连接池就是为了解决这个问题而出现的一个方法，为了提高性能，开发连接池，连接池中一直保持有 n 个连接，供调用者使用，调用者用完返还给连接池，继续给别的调用者使用，比如连接池中一开始就有 10 个连接，当有 5 个用户拿走了 5 个连接后，池中还剩 5 个，当第 6 个用户在去池中拿连接而前面 5 个连接还没归还时，连接池就会新建一个连接给第六个用户，让池中一直能够保存最少 5 个连接，而当这样新建了很多连接后，用户归还连接回来时，会比原先连接池中的 10 个连接更多，连接池就会设置一个池中最大空闲的连接数，如果超过了这个数，就会将超过的连接给释放掉，连接池就是这样工作的。

现在介绍几款连接池，DBCP、C3P0、tomcat 内置连接池(JNDI)(这个不讲)

DBCP 连接池，

两种方式获得连接，使用配置文件，不使用配置文件

1、不使用配置文件，自己手动设置参数

导包

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226142403491-1963442485.png)

核心类 BasicDataSource，通过 new 出 BasicDataSource 对象，设置参数 然后获得连接　　　　　　　　　　　　

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1         //创建核心类
2         BasicDataSource bds = new BasicDataSource(); 3         //配置 4 个基本参数
4         bds.setDriverClassName("com.mysql.jdbc.Driver");
5         bds.setUrl("jdbc:mysql:///myums");
6         bds.setUsername("root");
7         bds.setPassword("root");
8         
9         //管理连接配置
10         bds.setMaxActive(50);    //最大活动数
11         bds.setMaxIdle(20);    //最大空闲数
12         bds.setMinIdle(5);    //最小空闲数
13         bds.setInitialSize(10);//初始化个数 14         
15         //获取连接
16         try { 17             Connection conn = bds.getConnection(); 18 System.out.println(conn); 19             
20         } catch (SQLException e) { 21             throw new RuntimeException(e); 22         }

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

2、使用配置文件，参数写入配置文件中即可，也就是通过配置文件来配置驱动、用户名、密码、等信息

导包

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226142403491-1963442485.png)

导入配置文件 dbcpconfig.properties

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1 #连接设置
2 driverClassName=com.mysql.jdbc.Driver
3 url=jdbc:mysql://localhost:3306/test
4 username=root
5 password=root
6 
7 #<!-- 初始化连接 -->
8 initialSize=10
9 
10 #最大连接数量 11 maxActive=50
12 
13 #<!-- 最大空闲连接 -->
14 maxIdle=20
15 
16 #<!-- 最小空闲连接 -->
17 minIdle=5
18 
19 #<!-- 超时等待时间以毫秒为单位 6000毫秒/1000等于60秒 -->
20 maxWait=60000
21 
22 
23 #JDBC 驱动建立连接时附带的连接属性属性的格式必须为这样：[属性名=property;] 24 #注意："user" 与 "password" 两个属性会被明确地传递，因此这里不需要包含他们。 25 connectionProperties=useUnicode=true;characterEncoding=gbk 26 
27 #指定由连接池所创建的连接的自动提交（auto-commit）状态。 28 defaultAutoCommit=true
29 
30 #driver default 指定由连接池所创建的连接的只读（read-only）状态。 31 #如果没有设置该值，则“setReadOnly”方法将不被调用。（某些驱动并不支持只读模式，如：Informix） 32 defaultReadOnly=
33 
34 #driver default 指定由连接池所创建的连接的事务级别（TransactionIsolation）。 35 #可用值为下列之一：（详情可见 javadoc。）NONE,READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE 36 defaultTransactionIsolation=READ_UNCOMMITTED

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

获取连接

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1         //通过类加载器获取指定配置文件的输入流，Dbcp1 是一个类名，
2         InputStream is = Dbcp1.class.getClassLoader().getResourceAsStream("dbcpconfig.properties"); 3         Properties properties = new Properties(); 4 properties.load(is); 5         //加载配置文件，获得配置信息
6         DataSource ds = BasicDataSourceFactory.createDataSource(properties); 7         Connection conn = ds.getConnection(); 8         System.out.println(conn);

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

C3P0 连接池

导包

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226144938210-699906856.png)

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226145010913-385534310.png)

从配置信息中获取  配置文件必须为 xml

c3p0-config.xml

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1 <c3p0-config>
2     <!-- 默认配置，如果没有指定则使用这个配置 -->
3     <default-config>
4         <property name="driverClass">com.mysql.jdbc.Driver</property>
5         <property name="jdbcUrl">jdbc:mysql://localhost:3306/myums</property>
6         <property name="user">root</property>
7         <property name="password">root</property>
8     
9         <property name="checkoutTimeout">30000</property>
10         <property name="idleConnectionTestPeriod">30</property>
11         <property name="initialPoolSize">10</property>
12         <property name="maxIdleTime">30</property>
13         <property name="maxPoolSize">100</property>
14         <property name="minPoolSize">10</property>
15         <property name="maxStatements">200</property>
16         <user-overrides user="test-user">
17             <property name="maxPoolSize">10</property>
18             <property name="minPoolSize">1</property>
19             <property name="maxStatements">0</property>
20         </user-overrides>
21     </default-config> 
22     <!-- 命名的配置 -->
23     <named-config name="jxpx">
24         <property name="driverClass">com.mysql.jdbc.Driver</property>
25         <property name="jdbcUrl">jdbc:mysql://localhost:3306/myums</property>
26         <property name="user">root</property>
27         <property name="password">root</property>
28     <!-- 如果池中数据连接不够时一次增长多少个 -->
29         <property name="acquireIncrement">5</property>
30         <property name="initialPoolSize">20</property>
31         <property name="minPoolSize">10</property>
32         <property name="maxPoolSize">40</property>
33         <property name="maxStatements">0</property>
34         <property name="maxStatementsPerConnection">5</property>
35     </named-config>
36 </c3p0-config> 

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

从配置文件中看，需要注意一个地方，一个是 default-config，一个是 name-config，两者都区别在于创建核心类对象时，如果将 name-config 作为参数传进去，那么将会调用 name-config 下的配置信息，否则将调用 default-config 下的配置信息，

两种方式使用 c3p0，加参数，使用 named-config 的配置信息，不加参数，自动加载配置信息，加载的是 default-config 中的信息

获得连接，使用核心类

![](https://images.cnblogs.com/OutliningIndicators/ExpandedBlockStart.gif)

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

1         //1 c3p0...jar 将自动加载配置文件。规定：WEB-INF/classes (src)  c3p0-config.xml,也就是将配置文件放在 src 下就会自动加载。 2         //ComboPooledDataSource dataSource = new ComboPooledDataSource(); //自动从配置文件 <default-config>
3         ComboPooledDataSource dataSource = new ComboPooledDataSource(); //手动指定配置文件 <named-config name="jxpx">
4         Connection conn = dataSource.getConnection(); 5         System.out.println(conn);

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

四、dbutils 框架的使用

DBUtil 是一个框架，用于简化 JDBC 开发，   像之前有连接池来优化获取连接操作，而 DBUtils 用来操作 sql 语句、将获取的数据封装到我们想要的结果，也就不需要在像之前用 statement、预处理对象、ResultSet 这些东西来处理 sql 语句了， DBUtils 全部帮帮我们做好了，只需要两句代码就可以解决问题。

1、导包

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226153448866-149956461.png)

2、核心类　　QueryRunner

方式一，没有事务　　

new QueryRunner(dataSource);//将连接池传进去，因为不用管理事务，所以它将自动帮我们维护连接

增删改：update(sql,params...) 执行 DML sql 语句，并设置实际参数(可变参数，任意个参数，取决于有多少问号) 这里也就是用预处理了。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226153813163-1988539040.png)

其中 JdbcUtils 是一个工具类，获取 c3p0 的数据源

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226154047070-1029064350.png)　

查询：query(sql,handler,params...) 执行 DDL sql：查询语句，handler：将我们查询到的数据封装到想要的结果。  params：设置实际参数，可变。

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226154305366-1072825968.png)

处理类：BeanListHandler，还有别的很多处理类

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226154501226-30995565.png)

BeanListHandler：将查询每一条数据封装到指定 JavaBean，在将 JavaBean 封装到 List 集合中 最后返回集合     new List<User,User,...> 

使用：BeanListHandler<User>(User.class)

BeanHandler:　　将查询的一条数据封装到指定 JavaBean，并返回 javabean 实例

使用：BeanHandler<User>(User.class)　

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226154730366-810928320.png)　

ScalarHandler：处理一行一列结果集，也就是一个单元格，单个数据(不是一条数据)，(聚合)函数

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226154939804-1412977116.png)　

ArrayHandler：将查询一条记录所有数据封装到数组中， Object arr[] ={1,"jack","1234"}   

使用：new ArrayHandler()

ArrayListHandler 将查询的所有记录每条记录分别封装到数组中，在将数组封装到 list 集合中，最后返回集合  new List() list.add(arr);

ColumnListHandler 将执行列封装到 list 集合中，返回 list 集合 List　list= {"jack","rose","tom"}

KeyedHandler 将每一条记录封装到 Map<String,Object>A 中，在将 mapA 封装到 mapB 中，mapB.value 就是 mapA  mapB.key 就是指定的 key

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226155347476-1359270423.png)

MapHandler 将一条记录封装到 map 并返回 map  {id=2,username=jack,password=1234}

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226155443632-360131979.png)

MapListHandler 将每条记录都分别封装到 Map 中，然后将 Map 添加到 List 集合中，最后返回 list 集合    list<map，map>　　

方式二、使用事务，必须手动管理连接，且程序进行维护

构造方法：new QueryRunner() 这里不用参数，因为连接将手动获取

增删改：update(conn，sql，params...)　

查询：query(conn，sql，handler，params...）

跟没有事务差不多，多了个 conn　　

删除：

不使用 dbutils 来处理事务

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226155809663-1852832161.png)

使用 dbutils 框架中的工具类 DbUtils 来处理事务

![](https://images2015.cnblogs.com/blog/874710/201702/874710-20170226160017054-593090179.png)　　

五、总结

一篇很基础的对 JDBC 操作的文章，一步步从最基础最原生的 JDBC 代码讲起，一步步优化，优化连接，使用连接池，优化操作代码，使用第三方框架 dbutils 来操作。最终两句代码就搞定了对数据库的增删改查操作，其中要了解 dbutils 和连接池是如何实现的话，需要一些设计模式的知识，比如在 dbutils 中使用的策略模式等等，我感觉我暂时还不用去了解，还没到那种深度，等后面厉害了，再回过头来慢慢理解其中的精华。现在基本上会用就行了。其中所有用到的开发 jar 包，和配置文件我都会放在下面的链接中。