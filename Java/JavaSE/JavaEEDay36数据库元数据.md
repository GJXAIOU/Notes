# JavaEEDay36数据库元数据
@toc

## 一、数据库元数据
一般使用 JDBC 处理数据库的接口主要有三个，即：Connection、PreparedStatement、ResultSet
同时，对于这三个接口，还可以获取不同类型的元数据，**通过这些元数据类获得一些数据库的信息。**

　　元数据(MetaData)，即定义数据的数据。打个比方，就好像我们要想搜索一首歌(歌本身是数据)，而我们可以通过歌名，作者，专辑等信息来搜索，那么这些歌名，作者，专辑等等就是这首歌的元数据。因此数据库的元数据就是一些注明数据库信息的数据。

① 由Connection对象的getMetaData()方法获取的是DatabaseMetaData对象。主要封装了是对数据库本身的一些整体综合信息，例如数据库的产品名称，数据库的版本号，数据库的URL，是否支持事务等等。
② 由PreparedStatement对象的getParameterMetaData ()方法获取的是ParameterMetaData对象。
③由ResultSet对象的getMetaData()方法获取的是ResultSetMetaData对象。

### （一）由Connection对象的getMetaData()方法获取的是DatabaseMetaData对象；

```java
package metadata;

import org.junit.jupiter.api.Test;

import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.SQLException;

/**
 * @author GJXAIOU
 * @create 2019-08-02-16:14
 */
public class getMetadata {
    /**
     * 由Connection对象的getMetaData()方法获取DatabaseMetaData对象
     * @throws SQLException
     */
    @Test
    public void getMetadataByConnection() throws SQLException {
        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.获取数据库元数据
        DatabaseMetaData metaData = connection.getMetaData();

        // 3.具体的方法
        //获取数据库产品名称
        String productName = metaData.getDatabaseProductName();
        //获取数据库版本号
        String productVersion = metaData.getDatabaseProductVersion();
        //获取数据库用户名
        String userName = metaData.getUserName();
        //获取数据库连接URL
        String userUrl = metaData.getURL();
        //获取数据库驱动
        String driverName = metaData.getDriverName();
        //获取数据库驱动版本号
        String driverVersion = metaData.getDriverVersion();
        //查看数据库是否允许读操作
        boolean isReadOnly = metaData.isReadOnly();
        //查看数据库是否支持事务操作
        boolean supportsTransactions = metaData.supportsTransactions();

        System.out.println("productName : " + productName + "\n"+ "productVersion : " + productVersion + "\n"+"userName : " + userName +"\n"+
                "userUrl : " + userUrl +"\n"+ "driverName : " + driverName + "\n"+"driverVersion : " + driverVersion + "\n"+
                "isReadOnly : " + isReadOnly + "\n"+ "supportsTransactions : " + supportsTransactions);
    }
}

```
程序运行结果：
```
productName : MySQL
productVersion : 5.7.25-log
userName : root@localhost
userUrl : jdbc:mysql://localhost:3306/day34jdbc?serverTimezone = GMT%2B8
driverName : MySQL Connector/J
driverVersion : mysql-connector-java-8.0.14 (Revision: 36534fa273b4d7824a8668ca685465cf8eaeadd9)
isReadOnly : false
supportsTransactions : true
```

###  (二)由PreparedStatement对象的getParameterMetaData ()方法获取的是ParameterMetaData对象。
**该对象主要是针对PreparedStatement对象和其预编译的SQL命令语句提供一些信息**，比如像”delete from  person where id = ?”这样的预编译SQL语句，ParameterMetaData能提供占位符参数的个数，获取指定位置占位符的SQL类型等等。

以下有一些关于ParameterMetaData的常用方法：
getParameterCount：获取预编译SQL语句中占位符参数的个数

在我看来，**ParameterMetaData对象能用的只有获取参数个数的getParameterCount()方法**。

注意：ParameterMetaData许多方法MySQL并不友好支持，比如像获取指定参数的SQL类型的getParameterType方法，必须要将URL修改为“jdbc:mysql://localhost:3306/day34jdbc?serverTimezone = GMT%2B8 & generateSimpleParameterMetadata=true”才行，否则 idea 会抛出 SQLException。但是像getParameterType等等与其他的方法也没多好用，因为如下面的例子，这些方法好像只会将所有的参数认为是字符串(VARCHAR)类型。
```java
package metadata;

import org.junit.jupiter.api.Test;

import java.sql.*;

/**
 * @author GJXAIOU
 * @create 2019-08-02-16:14
 */
public class getMetadata {
    /**
     *  由PreparedStatement对象的getParameterMetaData ()方法获取的是ParameterMetaData对象
     * @throws SQLException
     */
    @Test
    public void getMetadataByPreparedStatement() throws SQLException {
        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备SQL语句
        String sql = "delete from  person where id = ?";

        // 3.建立preparedStatement运输SQL语句
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setInt(1, 2);

        // 4.获取元数据
        ParameterMetaData parameterMetaData = preparedStatement.getParameterMetaData();

        // 5.常见的方法：
        //获取参数个数
        int paramCount = parameterMetaData.getParameterCount();
        //以字符串形式获取指定参数的SQL类型，这里有问题
        String paramTypeName = parameterMetaData.getParameterTypeName(1);
        //返回指定参数的SQL类型，以java.sql.Types类的字段表示，这里有问题
        int paramType = parameterMetaData.getParameterType(1);
        //返回指定参数类型的Java完全限定名称，这里有问题
        String paramClassName = parameterMetaData.getParameterClassName(1);
        //返回指定参数的模，，这里有问题
        int paramMode = parameterMetaData.getParameterMode(1);
        //返回指定参数的列大小，这里有问题
        int precision = parameterMetaData.getPrecision(1);
        //返回指定参数的小数点右边的位数，这里有问题
        int scale = parameterMetaData.getScale(1);

        System.out.println("paramCount : " + paramCount + "\n" + "paramTypeName : " + paramTypeName + "\n" +
                "paramType : " + paramType + "\n" + "paramClassName : " + paramClassName + "\n" +
                "paramMode : " + paramMode + "\n" + "precision : " + precision + "\n" + "scale : " + scale);
    }
}

```
程序运行结果：
下面程序中输出结果中只有第一个占位符参数的个数是正确的，其他的都是有问题的；
```java
paramCount : 1
paramTypeName : VARCHAR
paramType : 12
paramClassName : java.lang.String
paramMode : 1
precision : 0
scale : 0
```
因为我们的SQL语句为"delete from  person where id = ?"，而我们所有利用ParameterMetaData查询的信息除了参数个数以外，都是查询第一个参数的信息，也就是“id”列，而这个“id”列我们创建时是int整型的，但是利用ParameterMetaData的查询结果都是显示为字符串类型，因此我对ParameterMetaData的功能产生了怀疑。

　　因此在以后使用参数元数据ParameterMetaData尽量只要使用其getParamterCount()方法获取参数个数，对于该对象其他方法请慎用。


### (三)由ResultSet对象的getMetaData()方法获取的是ResultSetMetaData对象。
```java
package metadata;

import org.junit.jupiter.api.Test;

import java.sql.*;

/**
 * @author GJXAIOU
 * @create 2019-08-02-16:14
 */
public class getMetadata {
    /**
     * 由ResultSet对象的getMetaData()方法获取的是ResultSetMetaData对象
     * @throws SQLException
     */
    @Test
    public void getMetadataByRestltSet() throws SQLException {
        // 1.建立连接
        Connection connection = JdbcUtil.getConnection();

        // 2.准备SQL语句
        String sql = "select * from person";

        // 3.使用preparedStatement运输SQL语句
        PreparedStatement preparedStatement = connection.prepareStatement(sql);

        // 4.指向SQL语句，返回结果集
        ResultSet set = preparedStatement.executeQuery();

        // 5.获取元数据
        ResultSetMetaData resultMetaData = set.getMetaData();

        //获取结果集的列数
        int columnCount = resultMetaData.getColumnCount();
        //获取指定列的名称
        String columnName = resultMetaData.getColumnName(1);
        //获取指定列的SQL类型对应于java.sql.Types类的字段
        int columnType = resultMetaData.getColumnType(1);
        //获取指定列的SQL类型
        String columnTypeName = resultMetaData.getColumnTypeName(1);
        //获取指定列SQL类型对应于Java的类型
        String className = resultMetaData.getColumnClassName(1);
        //获取指定列所在的表的名称
        String tableName = resultMetaData.getTableName(1);

        System.out.println("columnCount : " + columnCount + "\n" + "columnName : " + columnName + "\n" +
                "columnType : " + columnType + "\n" + "columnTypeName :" + columnTypeName + "\n" +
                "className : " + className + "\n" + "tableName : " + tableName);

    }
}

```
程序运行结果：
```java
columnCount : 6
columnName : id
columnType : -6
columnTypeName :TINYINT // 第一项的数据类型
className : java.lang.Integer
tableName : person
```

----

## 二、通用DAO层

==即将上面的 Dao 层再次的精简，只用两个方法代替原来的四个方法；==
*  1. 数据更新(增，删，改) 
*  2. 数据查询(查)
```java
package metadata;

import org.apache.commons.beanutils.BeanUtils;

import java.lang.reflect.InvocationTargetException;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

/**提供增删改查数据库的通用Dao层
 * @author GJXAIOU
 * @create 2019-08-02-19:31
 */
public class BaseDao {

    /**
     * 增删改的通用方法
     * @param sql 要执行的SQL语句，可以是：insert、delete、update
     * @param paramValue 参数数组，用来填充SQL语句中的占位符参数，若无参数，请传入null
     */
    public void updateCurrent(String sql, Object[] paramValue){
        // 1.连接数据库
        Connection connection = JdbcUtil.getConnection();

        // 2.获取PreparedStatement
        PreparedStatement preparedStatement = null;
        try {
            preparedStatement = connection.prepareStatement(sql);

            // 3.得到SQL语句中参数元数据个数（即占位符？的个数）
            int parameterCount = preparedStatement.getParameterMetaData().getParameterCount();

            // 4.利用参数元数据给SQL语句的占位符需要的参数赋值
            if (paramValue != null && paramValue.length > 0) {
                // 通过循环给SQL语句完整赋值
                for (int i = 0; i < parameterCount; i++) {
                    // 因为参数（占位符）计数是从1开始，因此需要+1
                    preparedStatement.setObject(i + 1, paramValue[i]);
                }
            }

            // 5.执行
            preparedStatement.executeUpdate();
            } catch (SQLException e) {
            e.printStackTrace();
        }finally {
            JdbcUtil.closeConnection(connection, preparedStatement);
        }
    }

    /**
     * 查询的通用方法（使用了泛型和反射）
     * @param sql 查询需要的SQL语句
     * @param paramValue 查询需要的参数，若没有这设置为null
     * @param tClass list集合中保存的数据类型
     * @param <T> list集合，返回一个带有指定数据类型的list集合
     * @return
     */
    public <T> List<T> inquiryCurrent(String sql, Object[] paramValue, Class<T> tClass){
        // 1.要返回的数据集合
        List<T> list = new ArrayList<>();

        // 2.确定list集合中要保存的对象
        T t = null;

        // 3.连接数据库
        Connection connection = JdbcUtil.getConnection();

        // 4.获取到PreparedStatement
        PreparedStatement preparedStatement = null;
        try {
            preparedStatement = connection.prepareStatement(sql);

            // 5.得到SQL语句中参数元数据个数（即占位符？的个数）
            int parameterCount = preparedStatement.getParameterMetaData().getParameterCount();

            // 6.利用参数元数据给SQL语句的占位符需要的参数赋值
            if (paramValue != null && paramValue.length > 0) {
                for (int i = 0; i < parameterCount; i++) {
                    preparedStatement.setObject(i + 1, paramValue[i]);
                }
            }

            // 7.执行查询操作，返回ResultSet
            ResultSet set = preparedStatement.executeQuery();

            // 8.获取结果集元数据
            ResultSetMetaData metaDataResultSet = set.getMetaData();
            // 获取数据库列的数量
            int columnCount = metaDataResultSet.getColumnCount();

            // 9.遍历ResultSet数据集
            while (set.next()){
                // 创建要保存的对象
                t = tClass.newInstance();

                    // 10.遍历数据行的每一列，得到每一列的名字，再获取到数据，保存到T对象中
                    for (int i = 0; i < columnCount; i++) {
                        // 首先获取每一列的名字
                        String columnName = metaDataResultSet.getColumnName(i + 1);
                        // 获取每一列的数据
                        Object value = set.getObject(columnName);
                        // 利用Beanutils给T对象赋值
                        BeanUtils.setProperty(t, columnName, value);
                    }
                // 将创建好的T对象，放入list对象中
                list.add(t);
                }
            } catch (IllegalAccessException ex) {
            ex.printStackTrace();
        } catch (InstantiationException ex) {
            ex.printStackTrace();
        } catch (SQLException ex) {
            ex.printStackTrace();
        } catch (InvocationTargetException ex) {
            ex.printStackTrace();
        }
        return null;
    }

}

```

-------
==下面没看，不知道讲啥==

## 三、针对 BeanUtil 包的基本使用

### JavaBean 定义
//JavaBean ：所有 成员变量私有化，并且对外提供一个set和get方法，这种类叫做JavaBean
也称之为 POJO 类：私有化+ setter 和 getter 方法以及 tostring 方法；
```java
package b_beanutils;

public class User {
	private String name;
	private int age;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	@Override
	public String toString() {
		return "User [name=" + name + ", age=" + age + "]";
	}
}

```

### BeanUtils 工具类方法使用示例
- BeanUtils 是一个工具类，里面的方法都是静态修饰的，可以通过类名直接调用
使用时需要导入：commons-beanutils-1.8.3.jar 和 commons-logging-1.1.3.jar 日志支持包

- 工具类中主要方法：
  - 方法一：setProperty(Object bean, String name, Object value);
    - Object bean 是按照JavaBean规定定义的类，这个类要求所有的成员变量私有化，并且对外提供对应的setter和getter方法
    - String name 要赋值的属性名，这里是借助于反射的思想获取到对应setter方法
    - Object value 要赋值给属性的数据
  	
  - 方法二：getProperty(Object bean, String name);
    - Object bean 是按照JavaBean规定定义的类，这个类要求所有的成员变量私有化，并且对外提供对应的setter和getter方法
    - String name 要获取的属性的名字，这里实际调用大方法是对应的getter方法
  - 方法三：copyProperties(Object dest, Object orig)
    - Object dest 是要被赋值的类对象
    - Object orig 复制的源数据

```java
package b_beanutils;

import org.apache.commons.beanutils.BeanUtils;

import java.lang.reflect.InvocationTargetException;

public class Demo {
	public static void main(String[] args) {
		test();
	}

	public static void test() {
		//最原始的方法：
		User u = new User();
		u.setName("张三");


		//利用BeanUtils里面的方法来给对象属性赋值
		try {

			Class cls = Class.forName("b_beanutils.User");
			Object obj = cls.newInstance(); //可以直接调用编译器给我们提供的该类的无参构造方法

			BeanUtils.setProperty(obj, "name", "李四");
			
			//这里调用的是User里面的getName方法，这个方法，有可能不存在，所有报异常NoSuchMethodException
			System.out.println(BeanUtils.getProperty(obj, "name"));
			
			//复制两个对象之间的属性
			BeanUtils.copyProperties(obj, u);
			System.out.println(obj);
		} catch (IllegalAccessException | InvocationTargetException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			// TODO: handle exception
		} catch (InstantiationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (NoSuchMethodException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
}	

```



### 具体是示例
student.java:是 JavaBean 类
studentDao.java

student.java
```java
package c_jdbc;

public class Student {
	private Integer stuId;
	private String stuName;
	private String stuSex;
	private Integer stuAge;
	private Integer stuScore;
	public Integer getStuId() {
		return stuId;
	}
	public void setStuId(Integer stuId) {
		this.stuId = stuId;
	}
	public String getStuName() {
		return stuName;
	}
	public void setStuName(String stuName) {
		this.stuName = stuName;
	}
	public String getStuSex() {
		return stuSex;
	}
	public void setStuSex(String stuSex) {
		this.stuSex = stuSex;
	}
	public Integer getStuAge() {
		return stuAge;
	}
	public void setStuAge(Integer stuAge) {
		this.stuAge = stuAge;
	}
	public Integer getStuScore() {
		return stuScore;
	}
	public void setStuScore(Integer stuScore) {
		this.stuScore = stuScore;
	}
	@Override
	public String toString() {
		return "Student [stuId=" + stuId + ", stuName=" + stuName + ", stuSex=" + stuSex + ", stuAge=" + stuAge
				+ ", stuScore=" + stuScore + "]";
	}	
}

```

studengDao.java
```java
package c_jdbc;

import java.util.List;

import utils.BaseDao;

public class StudentDao extends BaseDao {
	/**
	 * 删除
	 */
	public void deleteById(int id) {
		String sql = "delete from stuInfo where stuId=?";
		Object[] paramsValue = {id};
		
		super.update(sql, paramsValue);
	}
	
	/**
	 * 插入
	 */
	public void addStudent(Student stu) {
		String sql = "insert into stuInfo(stuId, stuName, stuAge, stuSex, stuScore)"
				+ " values(?,?,?,?,?)";
		Object[] paramsValue = {stu.getStuId(), stu.getStuName(), 
				stu.getStuAge(), stu.getStuSex(), stu.getStuScore()};
		
		super.update(sql, paramsValue);
	}
	
	/**
	 * 修改
	 */
	public void updateStudent(Student stu) {
		String sql = "update stuInfo set stuName=?, stuAge=?, stuSex=?, stuScore=? where stuId=?";
		Object[] paramsValue = {stu.getStuName(), 
				stu.getStuAge(), stu.getStuSex(), stu.getStuScore(), stu.getStuId()};
		
		super.update(sql, paramsValue);
	}
	
	/**
	 * 查询所有
	 */
	public List<Student> getAll() {
		String sql = "select * from stuInfo";
		List<Student> list = super.query(sql, null, Student.class);
		return list;
	}
	
	/**
	 * 获取指定ID的学生
	 */
	public Student findByID(int id) {
		String sql = "select * from stuInfo where stuId=?";
		Object[] paramsValue = {id};
		List<Student> list = super.query(sql, paramsValue, Student.class);
		
		return (list != null && list.size() > 0) ? list.get(0) : null;
	}
}

```
test.java
```java
package c_jdbc;

import java.util.List;

public class Test {
	public static void main(String[] args) {
		StudentDao dao = new StudentDao();
		
		Student stu = new Student();
		stu.setStuName("测试");
		stu.setStuSex("男");
		stu.setStuAge(10);
		stu.setStuScore(100);
		stu.setStuId(16);
//		dao.addStudent(stu);
		
		stu.setStuName("天王盖地虎");
		//dao.updateStudent(stu);
		
		//dao.deleteById(16);
		
		List<Student> list = dao.getAll();
		
		for (Student student : list) {
			System.out.println(student);
		}
		
		Student stu1 = dao.findByID(5);
		System.out.println(stu1);
	}
}

```


## dbUtils 使用
girl.java 
Demo1.java  :常用的方法 查询  增加删除修改

girl.java
```java
package d_dbutils;

public class Girl {
	private Integer girlID;
	private String girlName;
	private Integer girlAge;
	
	public Girl() {}
	
	public Girl(Integer girlID, String girlName, Integer girlAge) {
		super();
		this.girlID = girlID;
		this.girlName = girlName;
		this.girlAge = girlAge;
	}

	public String getGirlName() {
		return girlName;
	}
	public void setGirlName(String girlName) {
		this.girlName = girlName;
	}
	public Integer getGirlAge() {
		return girlAge;
	}
	public void setGirlAge(Integer girlAge) {
		this.girlAge = girlAge;
	}
	public Integer getGirlID() {
		return girlID;
	}
	public void setGirlID(Integer girlID) {
		this.girlID = girlID;
	}
	@Override
	public String toString() {
		return "Girl [girlID=" + girlID + ", girlName=" + girlName + ", girlAge=" + girlAge + "]";
	}
	
	
}

```

Demo1.java
```java
package d_dbutils;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.ResultSetHandler;
import org.apache.commons.dbutils.handlers.ArrayHandler;
import org.apache.commons.dbutils.handlers.ArrayListHandler;
import org.apache.commons.dbutils.handlers.BeanHandler;
import org.apache.commons.dbutils.handlers.BeanListHandler;
import org.apache.commons.dbutils.handlers.MapHandler;
import org.apache.commons.dbutils.handlers.MapListHandler;

import utils.JDBCUtil;

public class Demo1 {
	public static void main(String[] args) {
		//query1();
		//query2();
		//query3();
		//query4();
		//query5();
		//query6();
		query7();
	}
	
	/**
	 * 获取单个girl对象
	 * 方法一：使用ResultSetHandler
	 */
	public static void query1() {
		Connection conn = JDBCUtil.getConnection();
		
		//操作数据的核心类
		QueryRunner qr = new QueryRunner();
		String sql = "select * from girl where girlID = 2";
		
		try {
			Girl girl = qr.query(conn, sql, new ResultSetHandler<Girl>() {

				@Override
				public Girl handle(ResultSet arg0) throws SQLException {
					Girl g = null;
					if (arg0.next()) {
						g = new Girl(arg0.getInt("girlID"), arg0.getString("girlName"), 
								arg0.getInt("girlAge"));
					}	
					return g;
				}
				
			});
			System.out.println(girl);
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			JDBCUtil.close(conn, null);
		}
	}
	
	/**
	 * 获取单个girl对象
	 * 方法二：BeanHandler  更加简单
	 */
	public static void query2() {
		Connection conn = JDBCUtil.getConnection();
		QueryRunner qr = new QueryRunner();
		String sql = "select * from girl where girlID=2";
		
		try {
			Girl girl = qr.query(conn, sql, new BeanHandler<>(Girl.class));
			System.out.println(girl);
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			JDBCUtil.close(conn, null);
		}
	}
	
	/**
	 * 获取List集合Girl数据
	 * BeanListHandler
	 */
	public static void query3() {
		Connection conn = JDBCUtil.getConnection();
		QueryRunner qr = new QueryRunner();
		String sql = "select * from girl";
		
		try {
			List<Girl> list = qr.query(conn, sql, new BeanListHandler<>(Girl.class));
			
			for (Girl girl : list) {
				System.out.println(girl);
			}
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			JDBCUtil.close(conn, null);
		}
		
	}

	/**
	 * 获取一个Gril对象
	 * 使用的方法，
	 * QueryRunner query(Connection conn, String sql, ResultSetHandler rs, Object... params)
	 */
	public static void query4() {
		Connection conn = JDBCUtil.getConnection();
		QueryRunner qr = new QueryRunner();
		
		String sql = "select * from girl where girlID=?";
		
		try {
			Girl girl = qr.query(conn, sql, new BeanHandler<>(Girl.class), 2);
			System.out.println(girl);
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			JDBCUtil.close(conn, null);
		}		
	}
	
	/**
	 * 将数据库查询到的 数据 保存的到一个Object类型的数组中
	 */
	public static void query5() {
		Connection conn = JDBCUtil.getConnection();
		QueryRunner qr = new QueryRunner();
		
		String sql = "select * from girl where girlID=2";
		
		try {
			Object[] arr = qr.query(conn, sql, new ArrayHandler());
			System.out.println(Arrays.toString(arr));
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
	/**
	 * 将数据库查询到的 所有数据 ，每一个数据行做成一个数组，把所有的数组放入到一个List集合中
	 */
	public static void query6() {
		Connection conn = JDBCUtil.getConnection();
		QueryRunner qr = new QueryRunner();
		
		String sql = "select * from girl";
		
		try {
			List<Object[]> list = qr.query(conn, sql, new ArrayListHandler());
			
			for (Object[] objects : list) {
				System.out.println(Arrays.toString(objects));
			}
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

	/**
	 * 拿到所有数据，同时含有字段名
	 */
	public static void query7() {
		Connection conn = JDBCUtil.getConnection();
		QueryRunner qr = new QueryRunner();
		
		String sql = "select * from girl";
		try {
			List<Map<String, Object>> map = qr.query(conn, sql, new MapListHandler());
			for (Map<String, Object> map2 : map) {
				Set<Entry<String, Object>> entrySet = map2.entrySet();
				for (Entry<String, Object> entry : entrySet) {
					System.out.println(entry);
				}
			}
			
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}

    	/**
	 * 这里可以用到所有的增，删，该方法
	 * @throws SQLException
	 */
	@Test
	public void testUpdate() throws SQLException {
		String sql = "delete from girl where girlID=2";
		Connection conn = JDBCUtil.getConnection();
		
		QueryRunner qr = new QueryRunner();
		
		qr.update(conn, sql);
		
		JDBCUtil.close(conn, null);
		
		//可以使用DbUitls里面的方法
		//DbUtils.close(conn);
	}
	
	/**
	 * 批处理，减少访问数据库的次数，降低数据库的压力
	 */
	@Test
	public void testBatch() {
		String sql = "insert into girl(girlName, girlAge) values(?,?)";
		
		Connection conn = JDBCUtil.getConnection();
		
		QueryRunner qr = new QueryRunner();
		
		try {
			qr.batch(conn, sql, new Object[][] {{"刘亦菲", 20}, {"小龙女", 18}});
		} catch (SQLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

```



