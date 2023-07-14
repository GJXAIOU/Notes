# JavaEEDay34数据库

@toc

# 笔记总结：2019-8-1
首先是用户管理，即新建用户并授予权限以及删除用户；
然后介绍事务；
接着介绍数据库数据导入和导出的方法；
最后介绍使用 Java 连接数据库；
- 第一个代码是连接数据库的方式一；
- 第二个代码是连接数据库和释放资源的方式二：
- 下面的代码是基于第二种连接数据库的方式进行增删改查操作；
最最后介绍了一种防止 SQL 注入的方式：PreparedStatement


## 一、DCL 操作：
一般是项目经理或者 DBA 进行管理；
即管理用户：创建用户，给予用户操作数据的权限；


## 二、用户管理：

- 创建一个用户，用户名为 zhangsan 所在数据库的主机地址为 localhost，初始化密码：12345:
`create user "zhangsan"@"localhost" identified by "12345";`
- 授予新用户权限，上面的 localhost 也可以是一个 IP 地址；
`grant all on hello.* to "zhangsan" @"localhost";` //将 hello 数据库赋值权限给 zhangsan;
- 刷新权限：`flush privileges;` 至此完成新用户管理；
- 取消授权：`revoke all on hello.* from "zhangsan" @ "localhost";`
- 删除用户：`drop user "zhangsan" @ "localhost";`


## 三、 DTL 事务处理
当出现操作错误的时候，给予回退的机会；

- 以银行转账为例，首先创建数据表：
```sql
create table bank(
  userID tinyint not null primary key auto_increment,
  money double(15,2),
  name varchar(30)
);
```
- 插入部分数据，用于测试：
```sql
insert into bank(money,name) values(66,"zhangsan");
insert into bank(money,name) values(11,"lisi");
```
- 开启事务：一共两种方式：开始事务是将这个表加载在内存中进行处理，没有改变原来结构
  - 方式一：`start transaction;`
  - 方式二：`set autocommit = 0;`
```sql
start transaction;
```
- 然后开始进行操作：
```sql
update bank set money = money - 55 where userID = 1;
update bank set money = money + 55 where userID = 2;
```
- 操作完成之后，两种确认方式：
  - 回滚，即操作中出现了异常，可以回滚到开始事务之前；命令为：`rollback;`
  - 提交，即确认操作无误，提交开始事务之后的所有操作；命令为：`commit;`


## 四、数据库导入导出
使用命令行导入导出数据库，必须是没有登录数据库的状态下；


### （一）导出数据库：
导出的是一个.sql 文件，里面包含了所有的数据库操作信息，包括创建数据库和插入数据；
**在未登录数据库的状态下操作以下语句：**
`mysqldump -uroot -p hello > hello.sql;` 将 hello 数据库导出保存为 hello.sql 文件；
**注意：** 导出文件的保存位置为当前操作的命令的文件目录下，可以通过先修改操作命令的目录进行保存位置的修改；

### （二）导入数据库：
**前提：** 必须在数据库中创建一个数据库用于接收导入的数据库,**然后退出数据库，操作以下语句：**
`mysql -uroot -p receiveHello < hello.sql;` 将 hello.sql 数据库导入到 receiveHello 数据库中；


## 五、Java 连接数据库方式  
JDBC：Java Database Connectivity 操作数据库的规范；

JDBC 主要通过接口实现，组成 JDBC 中两个包：java.sql 和 javax.sql，以上两个包是 JavaSE 中包含的，但是需要导入 JDBC 的实现类才可以使用，该实现类是由第三方数据库提供商完成；

JDBC 主要的接口和类：
- Driver 接口：连接数据库的驱动 API；
- DriverManager 类：驱动管理类，负责驱动的注册（加载），获取数据库连接；
- Statement 接口：负责 SQL 语句的执行；
  - PreparedStatement 接口：负责 SQL 语句的预处理；
- ResultSet 接口：处理查询数据库的结果集；


### （一）通过 JDBC 连接 MySQL

- 需要一：确定数据库的 URL；
  - 例如：`jdbc:mysql://localhost:3306/hello` 协议：子协议：//IP：端口号/数据库名？参数
    - 协议：JDBC 总协议；
    - 子协议：目前使用的是连接 MySQL 数据库的协议；
    - IP：是数据库服务器的 IP 地址，localhost 表示本机的 IP 地址；
    - 端口号：3306 MySQL 默认的端口号，可以修改；
    - 数据库名：目前连接的操作的数据库是哪一个；
    - 参数：通常为：`useUnicode = true ` `characterEncoding = utf-8`;

- 需要二：连接数据库需要用户名和密码：


==具体的连接操作方式：==
- 方式一：直接在代码中写入要操作的数据库以及用户名和密码信息，但是代码不可复用；
  - 步一：通过 `Class.forName(“com.mysql.jdbc.cj.Driver”);` 注册驱动;
  - 步二：准备 URL：`jdbc:mysql://数据库IP:3306/数据库名 ? serverTimezone = GMT%2B8`;
  - 步三：通过 DriverManager 连接对象：即调用：`DriverManager.getConnection(url, 用户名， 密码)`；
  - 步四：关闭数据库：`连接对象.close()`;

- 方式二：使用.properties 文件
  - 步一：建立.properties 文件，内容为：
```
driver = com.mysql.jdbc.cj.Driver
user = 用户名
password = 密码
url = jdbc:mysql://IP:3306/数据库名 ? serverTimezone = GMT%2B8
```
  - 步二：建立 JdbcUtil.java 类，里面包含数据库连接和关闭的方式
      -  准备工作：
        - 步一：读取配置文件，即.properties 文件：`Properties p = new Properties();` 并将其加入到内存之中：`InputStream is = new FileInputStream(文件路径)`
        - 步二：利用 Properties 里面的 load 方法：`p.load(is)`
        - 步三：通过 properties 类对象获取想要的数据：`p.getProperty("url");等等`
        - 步四：加载类文件：`Class.forName(driver)`
        - 步五：关闭资源：在 finally 里面加入：`is.close();`
      - 进行连接：
        - 步一：建立新的连接方法：`getConnection(),返回值为Connection类型`
        - 步二：建立连接：`DriveManager.getConnection(url, user, password)`
        - 步三：返回建立的连接：`return connection;`
      - 关闭连接：
        - 类一：没有结果集的关闭（针对：增、删、改操作）
          - 步一：新建的关闭方法中参数为 Connection 和 Statement 类型；
          - 步二：逐个关闭资源：先关闭 statement，后关闭 Connection；
        - 类二：含有结果集的关闭（针对：查操作）
          - 步一：方法中的参数为：Connection、Statement、ResultSet 类型；
          - 步二：逐个关闭资源：顺序为：Statement、Connection、ResultSet

- 具体实现类：（以创建数据表和查询数据表为例）
  - 创建表格：同删除、修改操作，仅仅是 SQL 语句不同
    - 步一：通过已经创建好的工具类创建数据库连接对象：`Connection connection = jdbcUtil.getConnection()`
    - 步二：获取 Statement（SQL 语句运输者）：`Statement statement = connection.createStatement();`
    - 步三：准备 SQL 语句，语句的最后不需要分号：`String sql = "";`
    - 步四： 通过 statement 执行 SQL 语句：`statement.executeUpdate(sql);`
    - 步五：关闭资源：`JdbcUtil.closeConnection(conn.., sta);`
  - 查询信息：即查操作
    - 步一：连接数据库，同上；
    - 步二：获取 Statement；
    - 步三：准备 SQL 语句；
    - 步四：通过 statement 执行 SQL 语句：`ResultSet set = statement.executeQuery(sql);`
    - 步五：关闭资源：`JdbcUtil.closeConnectionWithResult(con.., state.., set)`
    
 下面代码中： `Class.forName("com.mysql.jdbc.Driver");`
  这句话首先会加载com.mysql.jdbc.Driver类文件到内存当中，而在这个类文件中有一下这段代码
   static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
 这个代码块是一个静态代码块，会在类文件加载到内存时，直接运行
 而在这个静态代码块中，完成了以下这些事情：
 	1. 创建的MySQL连接的Java程序的JDBC.Driver对象
 	2. 将这个创建的Driver对象，注册到java.sql.DriverManager里面

 这样做到好处：
 	简化代码的逻辑，提高效率
```java
package jdbc.connection;

import org.junit.jupiter.api.Test;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

/**
 * @author GJXAIOU
 * @create 2019-08-01-14:14
 */
public class JdbcConnection {
    @Test
    public void connection()  {
        // 1.注册驱动 JDBC 连接MySQL
        try {
         Class.forName("com.mysql.cj.jdbc.Driver");

        // 2.准备url，是JDBC所要连接MySQL数据库的URL，后面参数为设置时区
        String url = "jdbc:mysql://localhost:3306/day34jdbc?serverTimezone = GMT%2B8" ;

        // 3.通过DriverManager连接对象
        Connection con = null;
        con = DriverManager.getConnection(url, "root", "GJXAIOU");
        System.out.println(con);

        // 4.关闭数据库连接，释放资源
        con.close();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

}

```
程序运行结果：`com.mysql.cj.jdbc.ConnectionImpl@15761df8`

- 方式二：将数据库的具体信息写入：.properties 文件中，然后使用 Java 代码读取文件中信息，更改数据库信息只要在.properties 文件中修改就行；
db.properties
```properties
driver = com.mysql.cj.jdbc.Driver 
url = jdbc:mysql://localhost:3306/day34jdbc?serverTimezone = GMT%2B8 
user = root 
password = GJXAIOU
```

这里 URL 后面对应的重要参数见下面：不同参数之间用 `&`进行分割
| 参数名称 | 参数说明 | 缺省值 | 最低版本要求 |
|---|---|---|---|
| user | 数据库用户名（用于连接数据库） | | 所有版本 |
| password | 用户密码（用于连接数据库） | | 所有版本 |
| useUnicode | 是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为true | false | 1.1g |
| characterEncoding | 当useUnicode设置为true时，指定字符编码。比如可设置为gb2312或gbk | false | 1.1g |
| autoReconnect | 当数据库连接异常中断时，是否自动重新连接？ | false | 1.1 |
| autoReconnectForPools | 是否使用针对数据库连接池的重连策略 | false | 3.1.3 |
| failOverReadOnly | 自动重连成功后，连接是否设置为只读？ | true | 3.0.12 |
| maxReconnects | autoReconnect设置为true时，重试连接的次数 | 3 | 1.1 |
| initialTimeout | autoReconnect设置为true时，两次重连之间的时间间隔，单位：秒 | 2 | 1.1 |
| connectTimeout | 和数据库服务器建立socket连接时的超时，单位：毫秒。 0表示永不超时，适用于JDK 1.4及更高版本 | 0 | 3.0.1 |
| socketTimeout | socket操作（读写）超时，单位：毫秒。 0表示永不超时 | 0 | 3.0.1 |





JdbcUtil.java  数据库连接自定义工具类

- 自定义JDBC工具类 
1. 加载驱动
2. 获取连接对象
3. 关闭连接 
 	
 把连接数据库需要的信息，都保存在一个文件中，这个文件是一个properties文件
```java
package jdbc.connection;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.sql.*;
import java.util.Properties;

/**实现数据库连接和资源关闭操作
 * @author GJXAIOU
 * @create 2019-08-01-15:16
 */
public class JdbcUtil {
   private static String url = null;
   private static String user = null;
   private static String password = null;
   private static String driver = null;
   private static InputStream inputStream = null;

   // 这里利用静态代码块的特征，在类文件加载到内存的时候，
   // 就会执行在静态代码块中的代码
    static {
       // 1.读取配置文件信息，即读取properties文件
       Properties properties = new Properties();
       // 如果一个properties文件加载到内存中，需要借助IO流；
       try {
           inputStream = new FileInputStream(
                   "E:\\Program\\Java\\Project\\VideoClass\\JavaEEDay34" +
                           "\\src\\jdbc\\connection\\db.properties");
       } catch (FileNotFoundException e) {
           e.printStackTrace();
       }

       // 2.利用properties里面的load方法
       try {
           properties.load(inputStream);
       } catch (IOException e) {
           e.printStackTrace();
       }

       // 3.可以通过properties类对象，获取想要的数据
       url = properties.getProperty("url");
       user = properties.getProperty("user");
       password = properties.getProperty("password");
       driver = properties.getProperty("driver");

       // 4.加载类文件
       try {
           Class.forName(driver);
       } catch (ClassNotFoundException e) {
           e.printStackTrace();
           System.out.println("驱动加载失败");
       }finally {
           // 关闭文件连接
           if (inputStream != null){
               try {
                   inputStream.close();
               } catch (IOException e) {
                   e.printStackTrace();
               }
           }
       }
   }


    /**
     * 获取数据库连接对象
     * @return Connection对象
     */
   public static Connection getConnection(){
       Connection connection = null;
       try {
           connection = DriverManager.getConnection(url, user, password);
       } catch (SQLException e) {
           e.printStackTrace();
       }

       return  connection;
   }

    /**
     * 关闭数据库连接，释放statement
     * @param connection 数据库连接对象
     * @param statement statement对象
     */
    public static void closeConnection(Connection connection, Statement statement){

        try {
            if (statement != null){
                statement.close();
            }

            if (connection != null){
            connection.close();
            }
        }catch (SQLException e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        }
    }

    /**
     * 关闭带有结果集的查询语句资源
     * @param connection 数据库连接对象
     * @param statement statement 对象
     * @param set 结果集
     */
    public static void closeConnectionWithResult(Connection connection, Statement statement, ResultSet set){
        if (statement != null){
            try {
                statement.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (connection != null){
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (set != null){
            try {
                set.close();
            } catch (SQLException e) {
                e.printStackTrace();
                throw new RuntimeException(e);
            }
        }
    }


}


```

jdbcRealizationTest.java
 实现常见的增删改查操作：
```java
package jdbc.connection;

import org.junit.jupiter.api.Test;

import java.math.BigDecimal;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;


/**使用JdbcUtil工具类创建并操作数据表
 * @author GJXAIOU
 * @create 2019-08-01-16:07
 */
public class JdbcRealizationTest {

    /**
     * 通过JDBC创建数据表
     */
    @Test
    public void createTable()  {
        // 1.通过已经封装好的JdbcUtil工具类，获取数据库的连接对象
        Connection connection = JdbcUtil.getConnection();

        try {
            // 2.获取Statement，即SQL语句的运输者，作用是将SQL语句运输到MySQL中，让MySQL运行
            Statement statement = connection.createStatement();

            // 3.准备好SQL语句，语句最后不用分号
            String sql = "create table person(" +
                    "  id   tinyint  not null  primary key  auto_increment," +
                    "  name  char(5) not null," +
                    "  gender  char(1)," +
                    "  score  decimal(4, 2)," +
                    "  home  enum(\"江苏\", \"上海\", \"杭州\", \"苏州\")," +
                    "  hobby  set(\"游泳\", \"打球\", \"跑步\"))";

            // 4.通过statement执行SQL语句
            int count = statement.executeUpdate(sql);

            // 5.查看创建的结果
            System.out.println("影响的行数" + count);

        } catch (SQLException e) {
            e.printStackTrace();
        }

    }

    /**
     * 使用statement执行DML语句-- insert into
     */
    @Test
    public void testInsert(){
        // 1.建立数据库连接
        Connection connection = JdbcUtil.getConnection();

        Statement statement = null;
        try {
            // 2.获取到Statement
            statement = connection.createStatement();

            // 3.准备SQL语句
            String sql1 = "insert into person(name, gender, score, home, hobby) values(\"张三\", \"男\", 98.23, 2, 3)";
            String sql2 = "insert into person(name, gender, score, home, hobby) values(\"李四五\", \"女\",99.00,\"江苏\", \"游泳,打球\")";

            // 4.通过statement执行SQL语句
            int count1 = statement.executeUpdate(sql1);
            int count2 = statement.executeUpdate(sql2);

            // 5.打印影响的行数
            System.out.println("改变的行数：" + (count1 + count2));
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            // 6.关闭所有资源:connection 是连接数据库的资源，statement是引用程序到MySQL之间的SQL语句运输者，都是资源
            JdbcUtil.closeConnection(connection, statement);
        }
    }

    @Test
    /**
     * 使用statement删除数据库中一条数据
     */
    public void testDelete() {
        // 1.建立数据库连接
        Connection connection = JdbcUtil.getConnection();

        Statement statement = null;
        try {
            // 2.获取到statement
            statement = connection.createStatement();

            // 3.准备SQL语句
            String sql = "delete from person where id = 1";

            // 4.使用statement执行SQL语句
            int count = statement.executeUpdate(sql);

            // 5.输出影响的行数
            System.out.println("改变的行数：" + count);
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            // 6.关闭所有资源
            JdbcUtil.closeConnection(connection, statement);
        }
    }

    /**
     * 使用statement修改数据库中一条数据
     */
    public void testUpdate(){
        // 1.建立数据库连接
        Connection connection = JdbcUtil.getConnection();

        Statement statement = null;
        try {
            // 2.获取到statement
            statement = connection.createStatement();

            // 3.准备SQL语句
            String sql = "update person set name = '李四' where id = 2";

            // 4.使用statement执行SQL语句
            int count = statement.executeUpdate(sql);

            // 5.输出影响的行数
            System.out.println("改变的行数" + count);
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            // 6.关闭所有的资源
            JdbcUtil.closeConnection(connection, statement);
        }
    }

    /**
     * 使用statement查询数据库中数据并返回结果集set
     */
    @Test
    public void testSelect(){
        // 1.连接数据库
        Connection connection = JdbcUtil.getConnection();

        Statement statement = null;
        // 查询语句返回的结果集对象
        ResultSet set = null;
        try {
            // 2.获取到statement
            statement = connection.createStatement();

            // 3.准备SQL语句
            String sql = "select * from person";

            // 4.通过statement执行SQL语句, 获得查询结果集
            set = statement.executeQuery(sql);

            // 5.输出影响的行数
            while(set.next()){
                int id = set.getInt("id");
                String name = set.getString("name");
                String gender = set.getString("gender");
                BigDecimal score = set.getBigDecimal("score");
                String home = set.getString("home");
                String hobby = set.getString("hobby");

                System.out.println("id :" + id + " name :" + name + " gender :" + gender +
                        " socore :" + score + " home :" + home + " hobby :" + hobby);


            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnectionWithResult(connection, statement, set);
        }
    }


}

```
第一个 Test 结果：`影响的行数0`
第二个 Test 结果：`改变的行数：2`
第三个 Test 结果：`改变的行数：1`
第四个 Test 结果：`id :2 name :李四五 gender :女 socore :99.00 home :江苏 hobby :游泳,打球`


**上面的 set.next()方法使用说明：** 该 next()方法属于 RestltSet 中
```
boolean next()
      throws [SQLException](../../java/sql/SQLException.html "class in java.sql")

将光标从当前位置向前移动一行。 `ResultSet`光标最初位于第一行之前; 第一次调用方法`next`使第一行成为当前行; 第二个调用使第二行成为当前行，依此类推。

当调用`next`方法返回`false`时，光标位于最后一行之后。 任何调用需要当前行的`ResultSet`方法将导致抛出`SQLException` 。 如果结果集类型为`TYPE_FORWARD_ONLY` ，这是他们指定的JDBC驱动程序实现是否会返回供应商`false`或抛出一个`SQLException`上的后续调用`next` 。

如果当前行的输入流已打开，则对方法`next`的调用将隐式关闭它。 当读取新行时， `ResultSet`对象的警告链将被清除。

结果

`true`如果新的当前行有效; `false`如果没有更多的行

异常

`[SQLException](../../java/sql/SQLException.html "class in java.sql")` - 如果发生数据库访问错误，或者在关闭的结果集上调用此方法



```

### （二）上面总结：JDBC 核心 API
- Driver接口：
  - connect(url, propertie);
      - url: JDBC连接数据库(目前为 MySQL)URL
            标准格式为：`jdbc:mysql://localhost:3306/javaee1707?useSSL=true`
       - propertie：
            连接数据库的属性，主要包含的是数据库的用户名和密码
            
- DriverManager类：是驱动管理类，用于管理【加载/注册】过的驱动程序
    - registerDriver(driver); 注册驱动程序
    - Connection    getConnection(url, user, password);返回值是获取一个数据库的连接对象，需要的参数是存在JDBC协议的URL， 数据库用户名 和 密码

- Connection接口：
    -  Statement   createStament(); 创建一个Statement的实现类对象（因为 Statement 是接口）
    - PreparedStatement   preparedStatement(String sql); 获取到一个PreparedStatement SQL语句预处理对象
    -  CallableStatmenet    preparedCall(String sql); 了解

- Statement接口：
   - int    executeUpdate(String sql); 执行给定的SQL语句，通常用来执行DDL，DML，返回影响数据的行数
   - ResultSet    executeQuery(String sql); 执行给定的SQL语句 DQL 查询语句，返回数据结果集

- PreparedStatement接口：
    - int    executeUpdate(); 执行预处理的SQL语句，通常用来执行DDL，DML，返回影响数据的行数
    - ResultSet    executeQuery(); 执行预处理的SQL语句 DQL 查询语句，返回数据结果集
    预处理有利于防止 SQL注入

- ResultSet接口： 查询语句的数据结果集：
    - boolean    next(); 得到当前数据行，并且光标指向下一个数据行，如果没有数据行，返回false
    - getXXX(String "字段名"); 获取指定数据类型的字段数据


使用 PreparedStatement 防止 SQL 注入代码：
```java
package jdbc.connection;

import org.junit.jupiter.api.Test;

import java.sql.*;

/**使用preparedStatement防止SQL注入，因为在获取preparedStatement时候，已经对SQL语句进行了预处理
 * @author GJXAIOU
 * @create 2019-08-01-19:19
 */
public class PreparedStatementTest {
    @Test
    public void loginInTest(){
        // 1.建立连接
        Connection connection = null;
        connection= JdbcUtil.getConnection();

        // 2.准备预处理SQL语句，其中？为占位符，且顺序从1开始
        String sql = "select * from person where id = ? and name = ?";

        // 3.获取preparedStatement对象
        PreparedStatement preparedStatement = null;
        try {
            preparedStatement = connection.prepareStatement(sql);
        } catch (SQLException e) {
            e.printStackTrace();
        }

        // 4.准备参数
        try {
            preparedStatement.setInt(1,2);
            preparedStatement.setString(2,"李四五");
        } catch (SQLException e) {
            e.printStackTrace();
        }

        // 5.准备接收查询结果
        ResultSet set = null;
        try {
            set = preparedStatement.executeQuery();
            System.out.println(set);
        } catch (SQLException e) {
            e.printStackTrace();
        }

        // 6.判断输入的参数在数据表中有没有
        try {
            if (set.next()){
                System.out.println("登录成功");
            }else{
                System.out.println("登录失败");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnectionWithResult(connection, preparedStatement, set);
        }
    }
}

```
程序输出的结果：
`com.mysql.cj.jdbc.result.ResultSetImpl@23529fee`
`登录成功`
此时对应的数据库中数据为：
```sql
mysql> select * from person;
+----+-----------+--------+-------+--------+---------------+
| id | name      | gender | score | home   | hobby         |
+----+-----------+--------+-------+--------+---------------+
|  2 | 李四五    | 女     | 99.00 | 江苏   | 游泳,打球     |
+----+-----------+--------+-------+--------+---------------+
```





















