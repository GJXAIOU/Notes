# JavaEEDay35数据库
@toc

## preparedstatement 接口使用
提供一个 POJO 文件：Person.java
然后在 PersonDao.java 中实现增删改查方法
**与 Day34 代码不同点：** 通过在方法中传入 Person 对象，实现 参数中方式的是 get 方法得到;
最后来使用 PersonView.java 实现页面；
- 首先对应数据库建立一个实体类：Person.java
	 * 实体类：一般情况会与数据库中表内的数据类型一致
	 * 建议:成员变量的名字要和数据库里面的字段名一致
	 * 建议:使用基本数据类型的包装类
```java
package jdbc.preparedStatement;

/**
 * @author GJXAIOU
 * @create 2019-08-01-20:09
 */
public class Person {
    private Integer id;
    private String name;
    private String gender;
    private Integer score;
    private String home;
    private String hobby;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public Integer getScore() {
        return score;
    }

    public void setScore(Integer score) {
        this.score = score;
    }

    public String getHome() {
        return home;
    }

    public void setHome(String home) {
        this.home = home;
    }

    public String getHobby() {
        return hobby;
    }

    public void setHobby(String hobby) {
        this.hobby = hobby;
    }

    @Override
    public String toString() {
        return "Person{" +
                "id=" + this.getId()+
                ", name='" + this.getName() + '\'' +
                ", gender='" + this.getGender() + '\'' +
                ", score=" + this.getScore() +
                ", home='" + this.getHome() + '\'' +
                ", hobby='" + this.getHobby() + '\'' +
                '}';
    }
}

```

- 然后完成对应的具体实现类 PersonDao.java
```java
package jdbc.preparedStatement;


import org.junit.jupiter.api.Test;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**通过使用PreparedStatement实现增删改查
 * @author GJXAIOU
 * @create 2019-08-01-21:23
 */
public class PersonDao {
    /**
     * 输入一个Person类对象，保存数据到数据库中
     * @param person
     * @return int类型，返回值大于0表示添加成功，等于0表示添加数据失败
     */
    public int addTest(Person person) {
        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备预处理SQL语句
        String sql = "insert into person(name, gender, score, home, hobby) values(?, ?, ?, ?, ?)";

        PreparedStatement preparedStatement = null;
        try {
            // 3.使用preparedStatement处理SQL语句
            preparedStatement = connection.prepareStatement(sql);

            //插入数据
            preparedStatement.setString(1,person.getName());
            preparedStatement.setString(2,person.getGender());
            preparedStatement.setInt(3, person.getScore());
            preparedStatement.setString(4,person.getHome());
            preparedStatement.setString(5, person.getHobby());

            //指向SQL语句，返回的int 类型为影响的行数
            return preparedStatement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            JdbcUtil.closeConnection(connection, preparedStatement);
        }

        return 0;

    }

    /**
     * 根据id删除数据库中数据
     * @param id
     * @return int类型，返回值大于0表示删除成功，返回0表示删除数据失败
     */
    public int deleteTest(int id){
        // 1.建立数据库连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备预处理SQL语句
        String sql = "delete from person where id = ?";

        // 3.使用preparedStatement处理SQL语句
        PreparedStatement preparedStatement = null;
        try {
            preparedStatement = connection.prepareStatement(sql);

            // 4.输入参数
            preparedStatement.setInt(1, id);

            return preparedStatement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnection(connection, preparedStatement);
        }

        return 0;
    }

    /**
     * 修改Person表中数据信息
     * @param person 传入的person类对象
     * @return int类型，返回值大于0表示修改成功，返回0表示修改数据失败
     */
    public int updateTest(Person person){
        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备预处理的SQL语句
        String sql = "update person set name = ? , gender = ? where id = ?";

        // 3.使用preparedstatement处理SQL语句
        PreparedStatement preparedStatement = null;
        try {
            preparedStatement = connection.prepareStatement(sql);
            // 4.输入参数
            preparedStatement.setString(1, person.getName());
            preparedStatement.setString(2, person.getGender());
            preparedStatement.setInt(3, person.getId());

            return preparedStatement.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnection(connection, preparedStatement);
        }

        return 0;
    }

    /**
     * 查询数据库person中所有的信息，返回一个list集合
     * @return 返回保存person类对象的list集合
     */
    public List<Person> selectTest() {
        ResultSet set = null;
        PreparedStatement preparedStatement = null;
        List<Person> list = new ArrayList<Person>();

        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备SQL语句
        String sql = "select * from person";

        // 3.使用preparedstatement执行SQL语句
        try {
            preparedStatement = connection.prepareStatement(sql);

            // 4.接收结果集
            set = preparedStatement.executeQuery();

            // 5.获取查找结果
            while (set.next()){
                Person person = new Person();
                person.setId(set.getInt("id"));
                person.setName(set.getString("name"));
                person.setGender(set.getString("gender"));
                person.setScore(set.getInt("score"));
                person.setHome(set.getString("home"));
                person.setHobby(set.getString("hobby"));

                list.add(person);
            }
            return list;
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnectionWithResult(connection, preparedStatement, set);
        }
        return null;
    }

    /**
     * 根据ID，查询数据库person中的对应信息，返回一个person类对象
     * @param id 要查询的id
     * @return 返回一个person类对象，如果没有找到，返回null
     */

    public Person selectByIdTest(int id){
        ResultSet set = null;
        PreparedStatement preparedStatement = null;
        Connection connection = null;
        Person person = new Person();


        // 1.连接数据库
        connection = JdbcUtil.getConnection();

        // 2.准备SQL语句
        String sql = "select * from person where id = ?";

        // 3.使用Preparedstatement指向SQL语句
        try {
            preparedStatement = connection.prepareStatement(sql);

            // 4.传入参数：
            preparedStatement.setInt(1,id);

            set = preparedStatement.executeQuery();

            if (set.next()){
                person.setId(set.getInt("id"));
                person.setName(set.getString("name"));
                person.setGender(set.getString("gender"));
                person.setScore(set.getInt("score"));
                person.setHome(set.getString("home"));
                person.setHobby(set.getString("hobby"));
            }
            return person;
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnectionWithResult(connection, preparedStatement, set);
        }
        return null;
    }

}

```

- 最后实现界面的 PersonView.java 
```java
package jdbc.preparedStatement;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

/**
 * @author GJXAIOU
 * @create 2019-08-02-9:39
 */
public class PersonView {
    public static void main(String[] args) {
        PersonDao personDao = new PersonDao();
        Scanner scanner = new Scanner(System.in);

        while (true) {
            System.out.println("$$$$$$$$$$$$$$$$$$$$$$");
            System.out.println("1. 添加数据");
            System.out.println("2. 删除数据");
            System.out.println("3. 更新数据");
            System.out.println("4. 查询所有数据");
            System.out.println("5. 查看指定数据");
            System.out.println("6. 退出");
            System.out.println("$$$$$$$$$$$$$$$$$$$$$$");

            int choose = scanner.nextInt();
            switch (choose) {
                case 1:
                    System.out.println("请输入姓名");
                    String name = scanner.next();
                    System.out.println("请输入性别");
                    String gender = scanner.next();
                    System.out.println("请输入分数");
                    int score = scanner.nextInt();
                    System.out.println("请输入家乡：江苏、上海、杭州");
                    String home = scanner.next();
                    System.out.println("请输入爱好（多选）：游泳、打球、跑步");
                    String hobby = scanner.next();

                    Person person = new Person();
                    person.setName(name);
                    person.setGender(gender);
                    person.setScore(score);
                    person.setHome(home);
                    person.setHobby(hobby);

                    personDao.addTest(person);
                    break;
                case 2:
                    System.out.println("请输入要删除的ID号");
                    int idDelete = scanner.nextInt();
                    personDao.deleteTest(idDelete);
                    break;
                case 3:
                    Person person1 = new Person();
                    System.out.println("请输入要修改人员的ID号：");
                    int idUpdate = scanner.nextInt();
                    person1.setId(idUpdate);
                    System.out.println("请输入修改后的姓名：");
                    String nameUpdate = scanner.next();
                    person1.setName(nameUpdate);
                    System.out.println("请输入修改后的性别：");
                    String genderUpdate = scanner.next();
                    person1.setGender(genderUpdate);
                    personDao.updateTest(person1);
                    break;
                case 4:
                    System.out.println("person数据表中所有数据为：");
                    for (Person person2 : personDao.selectTest()) {
                        System.out.println(person2.toString());
                    }
                    break;
                case 5:
                    System.out.println("请输入要查询人员的ID号：");
                    int idSelect = scanner.nextInt();
                    System.out.println(personDao.selectByIdTest(idSelect).toString());

                    break;
                case 6:
                    System.out.println("退出程序");
                    System.exit(0);
                    break;

                default:
                    break;
            }
        }

    }
}

```

## 批处理操作

一般针对于批量插入操作；BatchPractice.java
```java
package jdbc.batching;

import jdbc.preparedStatement.JdbcUtil;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

/**
 * @author GJXAIOU
 * @create 2019-08-02-15:15
 */
public class BatchPratice {
    public static void main(String[] args) {
        batch();
    }

    public static  void batch() {
        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备是SQL语句
        String sql = "insert into person(name, gender, score, home, hobby) value(?, ?, ?, ?, ?)";

        // 3.使用preparedstatement运输SQL语句
        PreparedStatement preparedStatement = null;
        try {
             preparedStatement = connection.prepareStatement(sql);

             int flag = 0;
             for (int i = 0; i < 10; i++) {
                String name = "张三" + i + "号";
                String gender = "男";
                int score =  87 + i;
                String home = "江苏";
                String hobby = "游泳";

                 preparedStatement.setString(1, name);
                 preparedStatement.setString(2, gender);
                 preparedStatement.setInt(3, score);
                 preparedStatement.setString(4, home);
                 preparedStatement.setString(5, hobby);

                 // 4.添加批处理
                 preparedStatement.addBatch();
                 flag++;
                 // 设置每5条批处理一次
                 if (flag % 5 == 0){
                     // 执行保存到批处理里面的SQL语句
                     preparedStatement.executeBatch();
                     // 执行保存在批处理里面的SQL语句之后，清空批处理的缓冲区
                     preparedStatement.clearBatch();
                     flag = 0;
                 }
             }

             // 处理批处理中剩余的SQL语句，即剩余不够5个的倍数
            if (flag > 0){
                preparedStatement.executeBatch();
                preparedStatement.clearBatch();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnection(connection, preparedStatement);
        }
    }
}

```


## 保存文本数据到数据库
内容使用 blob 格式
```java
package c_blob;

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

import utils.JDBCUtil;

public class Demo {
	public static void main(String[] args) throws Exception {
		//writeTest();
		readTest();
	}
	
	/**
	 * 把文件数据，保存到数据库
	 * @throws Exception 
	 */
	public static void writeTest() throws Exception {
		Connection conn = JDBCUtil.getConnection();
		String sql = "insert into testblob(value) values(?)";
		PreparedStatement statement = conn.prepareStatement(sql);
		
		//使用缓冲字节流读取数据
		BufferedInputStream bs = new BufferedInputStream(
				new FileInputStream(new File("./res/《The Story of the Stone》.txt")));
		
		//设置字节流
		statement.setBinaryStream(1, bs);
		//statement.setCharacterStream(int parameterIndex, Reader reader);
		
		statement.executeUpdate();
		
		JDBCUtil.close(conn, statement);
		bs.close();
	}
	
	
	//将刚才放入数据库是的数据拿出
	public static void readTest() throws Exception {
		Connection conn = JDBCUtil.getConnection();
		String sql = "select value from testblob where id = 1";
		PreparedStatement statement = conn.prepareStatement(sql);
		BufferedInputStream bs = null;
		BufferedOutputStream bos = null;
		
		ResultSet set = statement.executeQuery();
		
		if (set.next()) {  
			//采用字符流进行读取
			InputStream in = set.getBinaryStream(1);
			bs = new BufferedInputStream(in);
			bos = new BufferedOutputStream(
					new FileOutputStream(new File("C:\\javaEE1707\\Day36\\res\\1.txt")));
			
			int length = 0;
			byte[] buf = new byte[8 * 1024];
			while ((length = bs.read(buf)) != -1) {
				bos.write(buf, 0, length);
			}
		}
		JDBCUtil.close(conn, statement, set);
		bos.close();
		bs.close();
	}
}


```

## 获取自增长值
```java
package d_getincrement;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import utils.JDBCUtil;

public class Demo {
	public static void main(String[] args) throws SQLException {
		//getCount();
		getAutoIncrementValue();
	}
	
	public static void getAutoIncrementValue() throws SQLException {
		Connection conn = JDBCUtil.getConnection();
		String sql = "insert into person(name, age) values('吴京', 40)";
		
		/*
		 * 想要在插入数据的同时获取到当前插入数据的ID号
		 * 实际意义：秒杀活动第几名，第几个注册用户
		 */
		//获取PreparedStatement添加一个参数 Statement.RETURN_GENERATED_KEYS
		PreparedStatement statement = conn.prepareStatement(sql,
				Statement.RETURN_GENERATED_KEYS);
		
		statement.executeUpdate();
		
		//从已经设置过参数的PreparedStatement里面获取到自增长值结果集ResultSet类型
		ResultSet generatedKeys = statement.getGeneratedKeys();
		
		if (generatedKeys.next()) {
			int id = generatedKeys.getInt(1);
			System.out.println("id = " + id);
		}
		
		JDBCUtil.close(conn, statement, generatedKeys);
	}
	
	//计算数据库中数量
	public static void getCount() throws SQLException {
		Connection conn = JDBCUtil.getConnection();
		String sql = "select count(*) c from person";
		PreparedStatement statement = conn.prepareStatement(sql);
		
		ResultSet set = statement.executeQuery();
		
		if (set.next()) {
			//System.out.println(set.getInt("count(*)"));
			System.out.println(set.getInt("c"));
		}
		
		JDBCUtil.close(conn, statement, set);
	}
}

```

## 事务处理
- 方式一：关闭自动提交

```java
package e_transaction;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Savepoint;

//Savepoint
import utils.JDBCUtil;

public class Demo1 {
	public static void main(String[] args) {
		account();
	}

	/**
	 * 最基本在Java中使用数据库的回滚机制
	 */
	public static void account() {
		Connection conn = JDBCUtil.getConnection();
		PreparedStatement statement1 = null;
		PreparedStatement statement2 = null;

		try {
			//关闭自动提交，这就是开启事务，设置回滚点
			conn.setAutoCommit(false);
			String sql1 = "update bank set money = money - 1000000000 where userID=1";
			statement1 = conn.prepareStatement(sql1);

			String sql2 = "update bank set money = money + 1000000000 where userID=2";
			statement2 = conn.prepareStatement(sql2);

			statement1.executeUpdate();
			statement2.executeUpdate();

			//如果这里没有确定提交SQL语句，也就是没有commit，数据库里面的数据不会发生改变
			conn.commit();
		} catch (SQLException e) {
			//如果在执行某一个SQL语句时发生了异常，那么一般情况下，这里事务之后的SQL语句，全部要回滚到事件之前
			e.printStackTrace();
			try {
				//回滚到事务之前
				conn.rollback();
			} catch (SQLException e1) {
				e1.printStackTrace();
			}
		} finally {
			try {
				statement1.close();
				statement2.close();
				//关闭数据库连接要放到最后面，确定在数据库操作中产生的资源全部释放掉之后，再断开数据库连接
				conn.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}
```

- 方式二：设置还原点
**注意 commit 位置**，如果还原点之后代码没有问题，就应该将还原点之前操作和之后的操作都提交；如果有问题，应该将还原点之前的代码进行提交。

```java
package e_transaction;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Savepoint;

//Savepoint
import utils.JDBCUtil;

public class Demo1 {
	public static void main(String[] args) {
		useSavepoint();
	}


	public static void useSavepoint() {
		Connection conn = JDBCUtil.getConnection();
		PreparedStatement statement1 = null;
		PreparedStatement statement2 = null;
		//还原点！！！
		Savepoint savepoint = null;

		try {
			//下面这里没有还原点
			conn.setAutoCommit(false);
			String sql1 = "update bank set money = money - 1000000000 where userID=1";
			statement1 = conn.prepareStatement(sql1);

			String sql2 = "update bank set money = money + 1000000000 where userID=2";
			statement2 = conn.prepareStatement(sql2);

			statement1.executeUpdate();
			statement2.executeUpdate();
			statement1.close();
			statement2.close();

			
			//下面有还原点
			//设置还原点
			savepoint = conn.setSavepoint();

			sql1 = "update bank set name = '马云' where userID=1";
			statement1 = conn.prepareStatement(sql1);

			sql2 = "update bank set name = '匿名君' where userID=2";
			statement2 = conn.prepareStatement(sql2);

			statement1.executeUpdate();
			statement2.executeUpdate();
		} catch (SQLException e) {
			e.printStackTrace();

			try {
				//回滚到还原点，相当于取消设置还原点之后的代码执行
				conn.rollback(savepoint);
			} catch (SQLException e1) {
				// TODO Auto-generated catch block
				e1.printStackTrace();
			}
		} finally {

			try {
				conn.commit();
				
				statement1.close();
				statement2.close();
				conn.close();
			} catch (SQLException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}

```










