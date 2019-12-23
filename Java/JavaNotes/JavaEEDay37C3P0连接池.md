# JavaEEDay37 C3P0连接池


## 四、连接池

==不再使用 JDBC 连接数据库，采用连接池的方式==
- 问题：发现在程序中，不断的有连接数据库的操作，但是也同时存在，每一次连接之后操作结束，立马就会关闭 ，因为涉及到数据库的打开，关闭，这里非常影响软件的运行效率。
- 解决方案：把数据库连接对象，放到一个池子里	
- 连接池功能如下：
1.初始化连接的个数，最大连接数，当前连接数，池子用集合来表示 ，一般使用LinkedList ；因为增删多，基本上没有查找
2.构造方法：创建初始化连接
3.创建连接的方法
4.获取连接的方法：
---> 判断：池子中有没有可用的连接
  --> 有，直接拿走
  --> 没有	判断是否达到了最大连接数，
    -->到达则抛出异常
    -->没有达到，创建新的连接
5.释放连接： 是将正在使用的数据库连接对象，放回池子内

```java
package connectionpool;

import metadata.JdbcUtil;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.LinkedList;

/**
 * @author GJXAIOU
 * @create 2019-08-02-20:37
 */
public class ConnectionPoolPractice {
    /*
    初始化连接数目，这里默认为3
     */
    private int initCount = 3;
    /*
    连接池最大连接数目，这里定义为6
     */
    private final int maxCount = 6;
    /*
     记录当前的连接数目
     */
    private int currentCount = 0;

    // 新建连接池，使用linkedlist操作
    private  LinkedList<Connection> connectionPool = new LinkedList<Connection>();

    // 1.构造方法，按照指定初始化连接个数并且创建新的连接
    public ConnectionPoolPractice(){
        for (int i = 0; i < initCount; i++) {
            // 类内创建连接的方式
            Connection connection = createConnection();
            connectionPool.addLast(connection);
            currentCount++;
        }
    }

    public int getCurrentCount(){
        return currentCount;
    }

    // 2.创建一个新的连接
    private Connection createConnection(){
        // 2.1加载驱动
       final Connection connection = JdbcUtil.getConnection();

        Connection proxy = (Connection)Proxy.newProxyInstance(connection.getClass().getClassLoader(), new Class[]{Connection.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // 定义一个返回值
                Object result = null;
                // 获取要执行方法的方法名
                String methodName = method.getName();
                // 这里只是限制了close的操作，其他的方法按照原来的操作进行
                // 修改关闭close执行的任务
                if ("close".equals(methodName)) {
                    System.out.println("执行close方法");
                    // 放回连接池内
                    connectionPool.addLast(connection);
                    System.out.println("数据库连接已经放回到连接池中");
                }else{
                    result = method.invoke(connection, args);
                }
                return result;
            }
        });
        return proxy;

    }

    // 3.从连接池中获取连接的方法
    public Connection getConnection(){
        // 3.1判断池子中有没有连接
        if (connectionPool.size() > 0){
            return connectionPool.removeFirst();
        }

        // 3.2如果没有连接，判断当前连接数是否达到最大值限制
        if (currentCount < maxCount){
            currentCount++;
            return createConnection();
        }
        throw new RuntimeException("当前连接数已经达到最大值");
    }

    // 4.释放连接
    public void realeaseConnection(Connection connection){
        // 4.1如果池子中读取的数目小于初始化连接，放入池子
        if(connectionPool.size() < initCount){
            connectionPool.addLast(connection);
        }else {
            currentCount--;
            try {
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ConnectionPoolPractice connectionPoolPractice = new ConnectionPoolPractice();
        System.out.println("当前连接数目为：" + connectionPoolPractice.getCurrentCount());

        Connection connection = connectionPoolPractice.getConnection();
        Connection connection1 = connectionPoolPractice.getConnection();
        Connection connection2 = connectionPoolPractice.getConnection();
        System.out.println(connection);
        System.out.println(connection1);
        System.out.println(connection2);

        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        Connection connection3 = connectionPoolPractice.getConnection();
        System.out.println(connection3);

    }

}


```


## 使用 C3P0 连接池
一共两种连接方式：
方式一：硬编码方式
方式二：使用 XML 文件

- 方式一：使用硬编码方式的代码
```java
package connectionpool;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import java.beans.PropertyVetoException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * @author GJXAIOU
 * @create 2019-08-02-21:42
 */
public class C3P0ConnectionPool {
    /**
     * 使用硬连接的方式
     * @throws PropertyVetoException
     * @throws SQLException
     */
    public void HardcodeStyleTest() throws PropertyVetoException, SQLException {
        // 1.创建连接池的核心工具类
        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();

        // 2.设置连接数据库所需的参数
        comboPooledDataSource.setDriverClass("com.mysql.cj.jdbc.Driver");
        comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/day34jdbc?serverTimezone = GMT%2B8 ");
        comboPooledDataSource.setUser("root");
        comboPooledDataSource.setPassword("GJXAIOU");

        // 3.设置C3P0连接池的属性:初始化连接数、最大连接数、等待时间
        comboPooledDataSource.setInitialPoolSize(3);
        comboPooledDataSource.setMaxPoolSize(6);
        comboPooledDataSource.setMaxIdleTime(1000);

        // 4.从连接池中获取数据库连接对象
        Connection connection = comboPooledDataSource.getConnection();

        // 5.准备preparedStatement执行SQL语句
        connection.prepareStatement("delete from person where id = 3").executeUpdate();
    }
}


```

- 方式二：使用 XML 文件的形式
```java
package connectionpool;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import java.beans.PropertyVetoException;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * @author GJXAIOU
 * @create 2019-08-02-21:42
 */
public class C3P0ConnectionPool {
    /**
     * 通过配置XML文件，创建C3P0连接池
     * @throws SQLException
     */
    public void XmlStyleTest() throws SQLException {
        // 1.创建C3P0核心类，会自动的加载s目录下的c3p0-config.xml文件
        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();

        // 2.创建连接
        Connection connection = comboPooledDataSource.getConnection();

        // 3.准备preparedStatement以及SQL语句
        PreparedStatement preparedStatement = null;
        String sql = "insert into person(name, gender, score, home, hobby) value(?, ?, ?, ?, ?)";

        for (int i = 0; i < 10; i++) {
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setString(1,"钱十");
            preparedStatement.setString(2,"男");
            preparedStatement.setInt(3, 98);
            preparedStatement.setString(4, "江苏");
            preparedStatement.setString(5, "游泳");
            preparedStatement.executeUpdate();
        }
        preparedStatement.close();
        connection.close();
    }
}



```
针对第二种方式，一般使用 XML 配置文件，也可以使用.properties 文件，这里使用 xml 文件，将其放置在上面代码同一包下即可；命名为：c3p0-config.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <default-config>
        <property name="jdbcUrl">jdbc:mysql://localhost:3306/day34jdbc?serverTimezone = GMT%2B8 </property>
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
        <property name="user">root</property>
        <property name="password">GJXAIOU</property>
        <property name="initialPoolSize">3</property>
        <property name="maxPoolSize">6</property>
        <property name="maxIdleTime">1000</property>
    </default-config>
</c3p0-config>

```







-- MySQL增强
create database day38;
use day38;

# 一、 数据约束
## （一） 默认值
```sql
create table student(
	id int, 
	name varchar(20),
	country varchar(20) default '中国' -- 默认值
);
```
-- 没有设置默认值的字段，在没有赋值的情况下都是NULL
insert into student(id, name) values(1, "孬蛋");
insert into student(id, name, country) values(2, "金豆", "China");
insert into student(id) values(3);


## （二）非空
```sql
create table student(
	id int, 
	name varchar(20),
	gender varchar(2) not null -- 非空
);
```
- 如果存在非空字段，必须赋值,下面会报错；
`insert into student(id, name) values(1, "孬蛋");`
ERROR 1364 (HY000): Field 'gender' doesn't have a default value

- 必须给非空字段赋值,下面语句正确；
`insert into student(id, name, gender) values(1, "狗蛋", "男");`

- 非空字段用在什么地方？
用户名，密码，邮箱，手机

##  （三）唯一
```sql
create table student(
	id int UNIQUE, -- 唯一
	name varchar(20)
);
```
insert into student(id, name) values(1, "孬蛋");
insert into student(id, name) values(1, "狗剩");
`ERROR 1062(23000):Duplicate entry '1' for key 'id'`
-- 唯一值不能重复



## (四)主键 (非空+唯一)
```sql
create table student(
	id int primary key,
	name varchar(20)
);
```
insert into student(id, name) values(1, "狗蛋");
insert into student(id, name) values(2, "狗剩");
insert into student(name) values('辣鸡');
-- ERROR 1364 (HY000): Field 'id' doesn't have a default value
-- ID值不能为空，必须有数据

insert into student(id, name) values(1, "炸鸡");
-- ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
-- ID值是唯一索引，不能为空，并且不能重复



## (五)自增长
```sql
CREATE TABLE student(
	-- 自增长 ZEROFILL 零填充，从0开始
	id INT ZEROFILL PRIMARY KEY AUTO_INCREMENT, 
	name VARCHAR(20)
);

insert into student(name) values("张三");
insert into student(name) values("李四");
insert into student(name) values("王五");
insert into student(name) values("马六");
```



##  （六）外键约束
下面表的设计中：每一个人都是JavaEE教学部，这里每一个数据行中，都有JavaEE教学部,导致了数据的冗余,是否可以把部门做成一张表
```sql
create table employee(
	id int primary key, -- 员工ID
	empName varchar(20), -- 员工名
	deptName varchar(20) -- 部门名
);

insert into employee values(1, "老刘", "JavaEE教学部");
insert into employee values(2, "我党", "JavaEE教学部");
insert into employee values(3, "大飞", "JavaEE教学部");
insert into employee values(4, "瑞哥", "JavaEE教学部");
insert into employee values(5, "赋赋", "JavaEE教学部");
```

-- 设计一个独立的部门表
```sql
create table dept(
	id int primary key,
	deptName varchar(20)
);
```

-- 设计一个新的员工表，带有部门ID
```sql
create table employee(
	id int primary key,
	empName varchar(20),
	deptID int, -- 用部门的ID来表示当前员工的部门是哪一个
	-- 建立一个外键约束
	CONSTRAINT emp_dept_fk foreign key(deptID) references dept(id) on update cascade on delete cascade -- 级联修改
	--emp_dept_fk :外键名称     deptID:外键   dept(id):连接的参考字段
);

insert into dept(id, deptName) values(1, "JavaEE教学部");
insert into dept(id, deptName) values(2, "PHP教学部");
insert into dept(id, deptName) values(3, "iOS教学部");


insert into employee values(1, "张三", 1);
insert into employee values(2, "李四", 2);
insert into employee values(3, "王五", 3);
insert into employee values(4, "赵六", 1);
```
`insert into employee values(5, "喜峰", 4);`
-- 存在问题，因为在部门中并没有部门ID为4的部门，这里数据无法添加
-- ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`day38`.`employee`, CONSTRAINT `emp_dept_fk` FOREIGN KEY (`deptID`) REFERE
 NCES `dept` (`id`) ON DELETE CASCADE ON UPDATE CASCADE)


-- 级联修改因为后面的这句话： on update cascade on delete cascade
- 如果修改部门，这里会随之修改主表里面的数据
`update dept set id=4 where id=3;`
- 删除部门表 同时会帮助我们删除员工信息
`delete from dept where id=2;`
- 修改员工表信息
`update employee set deptID=2 where id=4;`


- 如果没有级联修改
  - 1) 当存在外键约束，添加数据的顺序：先添加主表，在添加副表
  - 2) 当存在外键约束，修改数据的顺序：先修改副表，在修改主表
  - 3) 当存在外键约束，删除数据的顺序：先删除副表，在删除主表

## 二、关联查询

### 交叉查询
`select empName,deptName from employee, dept;`
 这个结果是有问题的 笛卡尔乘积 存在重复数据，不推荐使用

-- 需求：查询员工及其所在部门，显示员工姓名和部门名称
将上面的表格进行重新新建；

### 多表查询
- 规则：
  - 1. 确定查询那些表格 2. 确定要查询的字段 3. 表和表之间的关系
#### 内连接查询。
只有满足条件的结果才会展示(使用最多的多表查询)
`select empName,deptName from employee, dept where employee.deptID = dept.id;`
select empName,deptName  -- 要查询的字段
from employee, dept   -- 要查询的表格
where employee.deptID = dept.id; -- 表和表之间的关系

- inner join 内连接的另一种语法
`select empName, deptName from employee  inner join dept  on employee.deptID = dept.id;`
select empName, deptName
from employee   -- 主表
inner join dept -- 连接的是哪一张表
on employee.deptID = dept.id; -- 表示条件 

-  或者 使用别名
`select e.empName, d.deptName from employee e inner join dept d on e.deptID = d.id;`

### 左右外连接
- 需求，查看每一个部门的员工
-- 预期结果
	--	JavaEE 张三
	-- 	JavaEE 赵六
--  iOS  王五
--  PHP  李四

- 左[外]连接查询：使用左边表中的数据来匹配右边表的数据，如果符合条件，展示数据
-- 如果没有符合条件的连接数据，显示null
`select d.deptName, e.empName from dept d left outer join employee e on d.id = e.deptID;`

-- 右[外]连接查询：使用右边表中的数据来匹配左边表的数据，如果符合条件，展示数据
-- 如果没有符合条件的连接数据，显示null
`select d.deptName, e.empName from employee e right outer join dept d on d.id = e.deptID;`

###  自连接查询
-- 修改员工表结构，添加上司
```sql
alter table employee add bossId int;

update employee set bossId=1 where id=2;
update employee set bossId=2 where id=3;
update employee set bossId=3 where id=4;
```

-- 预期结果
	-- 张三  null
	-- 李四  张三
	-- 王五  李四
	-- 赵六  王五
`select e.empName, b.empName from employee e left outer join employee b on e.bossId = b.id;`
