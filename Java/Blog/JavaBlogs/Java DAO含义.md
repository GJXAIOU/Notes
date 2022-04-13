# DAO-数据访问对象(Data Access Object) 模式


**业务对象只应该关注业务逻辑，不应该关心数据存取的细节**。数据访问对象必须实现特定的持久化策略（如，基于JＤＢＣ或Ｈｉｂｅｒｎａｔｅ的持久化逻辑）， 这样就抽出来了**DAO层，作为数据源层**，而之上的Domain Model层与之通讯而已，如果将那些实现了数据访问操作的所有细节都放入高层Domain model(领域模型)的话，系统的结构一定层次上来说就变得有些混乱。低级别的数据访问逻辑与高级别的业务逻辑分离，**用一个DAO接口隐藏持久化操作的细节，这样使用的最终目的就是让业务对象无需知道底层的持久化技术知识**，这是标准 J2EE 设计模式之一。

## 一、 DAO模式使用环境

- 具体类图如下。

![200901281233114873707]($resource/200901281233114873707.jpg)

- 参与者和职责

1)    BusinessObject(业务对象)
代表数据客户端。正是该对象需要访问数据源以获取和存储数据。

2)  DataAccessObject(数据访问对象)
是该模式的主要对象。DataAccessObject抽取该BusinessObject的低层数据访问实现，以保证对数据源的透明访问。BusinessObject也可以把数据加载和存储操作委托给DataAccessObject。

3)    DataSource(数据源)
代表数据源实现。数据源可以是各RDBMSR数据库，OODBMS,XML文件等等。

4)   ValueObject(值对象)
代表用做数据携带着的值对象。DataAccessObject可以使用值对象来把数据返回给客户端。
DataAccessObject也许会接受来自于客户端的数据，其中这些用于更新数据源的数据存放于值对象中来传递。

## 二、 数据访问对象的工厂策略

通过调整抽象工厂和工厂方法模式，DAO模式可以达到很高的灵活度。具体类图如下。
![200901281233114917993]($resource/200901281233114917993.jpg)

**DAO层使应用程序更加容易地迁移到一个不同的数据库实现**。业务对象不了解低层数据实现。因而，该迁移只涉及对DAO层的变化。更进一步说，如果使用工厂策略，则有可能为每一个低层存储实现提供一个具体工厂实现。在这种情况下，迁移到不同的迁移实现意味着给应用程序提供一个新的工厂实现。

同时，抽象DAO工厂可以指定需要创建的实例DAO，并交由不同的具体DAO工厂去创建。

## 三、 DAO代码实例（工程代码见附件）

一个典型的DAO组成：DAO工厂类，DAO接口，实现DAO接口的具体类(每个 DAO 实例负责一个主要域对象或实体)，VO（Value Object）。

![200901281233114969611]($resource/200901281233114969611.jpg)

**VO为Student类，将所有关于持久化的操作归入StudentDAO类中**。

Student.java
```java
public class Student {
  private String id;
  private String name;
  private String cardId;
  private int age;

  getter/setter()。。。

}
```

StudentDAO.java
```java
public interface StudentDAO {
  public boolean insertStudent(Student student);
  public boolean deleteStudent(int id);
  public Student findStudent(int id);
}
```

抽象DAO工厂DAOFactory指定了可能的具体DAO工厂，并指定需要创建的具体DAO。

DAOFactory.java
```java
public abstract class DAOFactory {
    // List of DAO types supported by the factory
    public static final int SQLSERVER = 1;
    public static final int MYSQL = 2;
    // There will be a method for each DAO that can be
    // created. The concrete factories will have to
    // implement these methods.
    public abstract StudentDAO getStudentDAO();
 
    public static DAOFactory getDAOFactory(int whichFactory) {
       switch (whichFactory) {
       case SQLSERVER:
           return new SqlServerDAOFactory();
       case MYSQL:
           return new MySqlDAOFactory();
       default:
           return null;
       }
    }
}
```

这里提供两个具体DAO工厂，SqlServerDAOFactory和MySqlDAOFactory。提供它们来得到具体的DAO实现。

SqlServerDAOFactory.java

```java
public class SqlServerDAOFactory extends DAOFactory{
    public static final String DRIVER = "com.microsoft.sqlserver.jdbc.SQLServerDriver";
    public static final String DBURL = "jdbc:sqlserver://localhost:1025; DatabaseName=tmp";
    private static String userName = "sa";
    private static String userPwd = "root";
 
    public static Connection createConnection() {
       Connection dbConn = null;
       try {
           Class.forName(DRIVER);
           dbConn = DriverManager.getConnection(DBURL, userName, userPwd);
       } catch (ClassNotFoundException e) {
           e.printStackTrace();
       } catch (SQLException e) {
           e.printStackTrace();
       }
       return dbConn;
    }
 
    public StudentDAO getStudentDAO() {
       return new SqlServerStudentDAO(createConnection());
    }
 
    。。。。。。
}
```

MySqlDAOFactory.java略

这里提供一个缺省的StudentDAO实现StudentDAODefaultImpl，它依据特定的Connection，来实现数据库相关操作。

StudentDAODefaultImpl.java
```java
public abstract class StudentDAODefaultImpl implements StudentDAO {
    private Connection dbConn;
 
    public StudentDAODefaultImpl(Connection dbConn) {
       this.dbConn = dbConn;
    }
 
    public boolean deleteStudent(int id) {
       Statement stmt;
       try {
           stmt = dbConn.createStatement();
           String sql = "DELETE FROM student_table WHERE id = '" + id + "'";
           int delete = stmt.executeUpdate(sql);
           if (delete == 1)
              return true;
       } catch (SQLException e) {
           e.printStackTrace();
       }
       return false;
    }
 
    public Student findStudent(int id) {。。。}
    public boolean insertStudent(Student stu) {。。。}
}
```
两个特定的DAO类分别从两个具体DAO工厂，SqlServerDAOFactory和MySqlDAOFactory中得到连接对象。

SqlServerStudentDAO.java
```java
public class SqlServerStudentDAO extends StudentDAODefaultImpl {
    private Connection dbConn = SqlServerDAOFactory.createConnection();
   
    public SqlServerStudentDAO(Connection dbConn) {
       super(dbConn);
    }
 
    public Connection getDbConn() {
       return dbConn;
    }
}
```

MySqlStudentDAO.java略

测试类Test.java
```java
public class Test {
    public static void main(String[] args) {
       Student student = new Student("1", "zj", "0901", 27);
       DAOFactory mysqlDAOFactory = DAOFactory.getDAOFactory(DAOFactory.MYSQL);
       StudentDAO studentDAO = mysqlDAOFactory.getStudentDAO();
       studentDAO.insertStudent(student);
    }
}
```
