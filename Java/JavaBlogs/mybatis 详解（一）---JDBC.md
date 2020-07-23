# mybatis 详解（一）---JDBC



## 1、什么是MyBatis?

　　MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。

　　iBATIS一词来源于“internet”和“abatis”的组合，是**一个基于Java的持久层框架**。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）。

　　MyBatis 是支持普通 SQL查询，存储过程和高级映射的优秀持久层框架。MyBatis 消除了几乎所有的JDBC代码和参数的手工设置以及结果集的检索。**MyBatis 使用简单的 XML或注解用于配置和原始映射，将接口和 Java 的POJOs（Plain Ordinary Java Objects，普通的 Java对象）映射成数据库中的记录**。

## 2、为什么会有 MyBatis?

　　通过上面的介绍，我们知道 MyBatis 是来和数据库打交道。那么在这之前，我们是使用 JDBC 来对数据库进行增删改查等一系列操作的，而我们之所以会放弃使用 JDBC，转而使用 MyBatis 框架，这是为什么呢？或者说使用 MyBatis 对比 JDBC 有什么好处？

下面我们通过一段 JDBC 对 Person 表的操作来具体看看。

 person 表为：
```person_java
public class Person {
    private Long pid;
    private String pname;
    // 省略 set、get 方法
}
```

JDBC 查询操作：
```CRUDDao_java
package com.ys.dao;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
 
import javax.swing.DebugGraphics;
 
import com.ys.bean.Person;
 
public class CRUDDao {
    //MySQL数据库驱动
    public static String driverClass = "com.mysql.jdbc.Driver";
    //MySQL用户名
    public static String userName = "root";
    //MySQL密码
    public static String passWord = "root";
    //MySQL URL
    public static String url = "jdbc:mysql://localhost:3306/test";
    //定义数据库连接
    public static Connection conn = null;
    //定义声明数据库语句,使用 预编译声明 PreparedStatement提高数据库执行性能
    public static PreparedStatement ps = null;
    //定义返回结果集
    public static ResultSet rs = null;
    
    /**
     * 查询 person 表信息
     * @return：返回 person 的 list 集合
     */
    public static List<Person> readPerson(){
        List<Person> list = new ArrayList<>();
        try {
            // 加载数据库驱动
            Class.forName(driverClass);
            // 获取数据库连接
            conn = DriverManager.getConnection(url, userName, passWord);
            // 定义 sql 语句, ? 表示占位符
            String sql = "select * from person where pname=?";
            // 获取预编译处理的 statement
            ps = conn.prepareStatement(sql);
            // 设置 sql 语句中的参数，第一个为 sql 语句中的参数的?(从1开始)，第二个为设置的参数值
            ps.setString(1, "qzy");
            //向数据库发出 sql 语句查询，并返回结果集
            rs = ps.executeQuery();
            while (rs.next()) {
                Person p = new Person();
                p.setPid(rs.getLong(1));
                p.setPname(rs.getString(2));
                list.add(p);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally{
            //关闭数据库连接
            if(rs!=null){
                try {
                    rs.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(ps!=null){
                try {
                    ps.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(conn!=null){
                try {
                    conn.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
         
        return list;
    }
 
    public static void main(String[] args) {
        System.out.println(CRUDDao.readPerson());
    }
}
```

### 3、分析

通过上面的例子我们可以分析如下几点：

①、问题一：数据库连接，使用时就创建，使用完毕就关闭，这样会对数据库进行频繁的获取连接和关闭连接，造成数据库资源浪费，影响数据库性能。
设想解决：使用数据库连接池管理数据库连接

②、问题二：将 sql 语句硬编码到程序中，如果sql语句修改了，那么需要重新编译 Java 代码，不利于系统维护
设想解决：将 sql 语句配置到 xml 文件中，即使 sql 语句变化了，我们也不需要对 Java 代码进行修改，重新编译

③、问题三：在 PreparedStatement 中设置参数，对占位符设置值都是硬编码在Java代码中，不利于系统维护
设想解决：将 sql 语句以及占位符和参数都配置到 xml 文件中

④、问题四：从 resultset 中遍历结果集时，对表的字段存在硬编码，不利于系统维护
设想解决：将查询的结果集自动映射为 Java 对象

⑤、问题五：重复性代码特别多，频繁的 try-catch
设想解决：将其整合到一个 try-catch 代码块中

⑥、问题六：缓存做的很差，如果存在数据量很大的情况下，这种方式性能特别低
设想解决：集成缓存框架去操作数据库

⑦、问题七：sql 的移植性不好，如果换个数据库，那么sql 语句可能要重写
设想解决：在 JDBC 和 数据库之间插入第三方框架，用第三方去生成 sql 语句，屏蔽数据库的差异

既然直接使用 JDBC 操作数据库有那么多的缺点，那么我们如何去解决呢？请看下面 mybatis 框架的入门实例介绍。