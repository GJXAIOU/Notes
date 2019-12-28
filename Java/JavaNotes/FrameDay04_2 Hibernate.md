# FrameDay04_2 Hibernate


**上节内容**
- web内容回顾
- hibernate概述
  - orm思想
- hibernate入门案例
- hibernate的配置文件
- hibernate的核心api


**今天内容**
- 实体类编写规则
- hibernate主键生成策略
  - native
  - uuid
- 实体类操作
  - crud操作
  - 实体类对象状态
- hibernate的一级缓存
- hibernate的事务操作
  - 事务代码规范写法
- hibernate其他的api（查询）
  - Query
  - Criteria
  - SQLQuery

## 一、实体类编写规则

实体类同时称为持久化类，数据持久化就是将内存中的数据永久存储到关系型数据库中；
持久化类：值一个 Java 类与数据库建立了映射关系，则该类称为持久化类；


- 实体类里面属性私有的
- 持久化类要提供无参数的构造方法：**因为 Hibernate 底层需要使用 反射 生成类的实例；**
- 私有属性使用公开的set和get方法进行操作
- 要求实体类有属性作为唯一值（一般使用id值），作为与表中的主键相对应；
因为 Hibernate 中通过唯一的标识 OID 区分内存中是否是同一个持久化类（Java 中是通过地址进行区分是都是同一个对象的，关系型数据库表中是通过主键区分是否为同一条记录的），Hibernate 不允许在内存中存在两个 唯一值相同的持久化对象；
- 持久化类尽量不要使用 final 进行修饰；
因为 Hibernate 中有延迟加载的机制，该机制会产生代理机制，Hibernate 产生代理对象使用的是字节码的增强技术完成的，本质上就会说产生了当前类的一个子类对象实现的，如果使用 final 修饰，就不能产生子类，就没有代理对象，从而延迟加载策略就会失效；
- 实体类属性建议不使用基本数据类型，使用基本数据类型对应的包装类
  为什么使用包装类：
  比如 表示学生的分数，假如 int score;
  - 比如学生得了0分 ，int score = 0;
  - 如果表示学生没有参加考试，int score = 0;不能准确表示学生是否参加考试
  解决：使用包装类可以了， Integer score = 0，表示学生得了0分，
  表示学生没有参加考试，Integer score = null;

## 二、Hibernate 主键生成策略

因为持久类中需要有一个唯一标识 OID 与表的主键建立映射关系，这个主键尽量使用程序生成，Hibernate 提供了相应的主键生成方式；

**主键类型：**
- 自然主键：将具有业务含义的字段作为主键；字段作为主键的前提是该字段满足主键条件：不能为 null，且不能重复，不能修改；
- 代理主键：将不具备业务含义的字段作为主键，一般使用整数类型；

- hibernate要求实体类里面有一个属性作为唯一值，其对应表主键，主键可以不同生成策略；
-  hibernate主键生成策略有很多的值；示例为：`<generator class = "native"></generator>`
 -  在class属性里面有很多值：常用的是 native 和 uuid 两个；
  - native：根据使用的数据库帮助选择哪个值（因为有些值不是所有数据库都支持）；
  - uuid：之前 web 阶段写代码生成 uuid 值，hibernate 会帮我们生成 uuid 值；

Hibernate 中提供的主键生成策略

| 值 | 描述 |
|---|---|
|native | 根据底层数据库对自动生成表示符的能力来选择过 identity 、sequence、hilo三种生成器中的一种，适合跨数据库平台开发。适用于代理主键。|
|uuid | Hibernate 采用 128 位的 UUID 算法来生成标识符。 该算法能够在网络环境中生成唯一的字符串标识符，其 UUID 被编码为一个长度为32 位的十六进制字符串。 这种策略并不流行， 因为字符串类型的主键比整数类型的主键占用更多的数据库空间。 适用于代理主键。|
|  increment | 用于 long、 short、或 int 类型， 由 Hibernate 自动以递增的方式生成唯一标识符，每次增量为 1。只有当没有其它进程向同一张表中插入数据时才可以使用，不能在集群环境下使用。 适用于代理主键。|
|identity| 采用底层数据库本身提供的主键生成标识符，条件是数据库支持自动增长数据类型。 在 DB2、 MySQL、 MS SQL Server、 Sybase 和 HypersomcSQL 数据库中可以使用该生成器，该生成器要求在数据库中 把主键定义成为自增长类型。 适用于代理主键。|
|sequence | Hibernate 根据底层数据库序列生成标识符。 条件是数据库支持序列。 适用于代理主键。|
| assigned | 由 Java程序负责生成标识符，如果不指定过元素的 generator 属性，则默认使用该主键生成策略。 适用于自然主键。|

- 演示生成策略值 uuid
 - 使用 uuid 生成策略，实体类 id 属性类型 必须 **字符串类型**：`private String uid;`
 - 配置部分写出uuid值：`<generator class="uuid"></generator>`


## 三、实体类操作
## 对实体类crud操作
**下面使用的是 native 方式**
### （一）添加操作
- 调用 session 里面的 save 方法实现
```java
User user = new User();
user.setUsername("GJXAIOU");
user.setPassword("GJXAIOU");
user.setAddress("江苏");
// 使用 session 的方法实现添加
session.save(user);
```

- 根据id查询【这里使用是原来的 native 方式】
调用 session 里面的 get 方法实现
```java
// 根据 id 查询，调用 session 中 get 方法实现
   // 参数一：实体类的 class；参数二：要查询的 id 值
User user = session.get(User.class, 1);
```
结果：
```sql
Hibernate: 
    select
        user0_.uid as uid1_0_0_,
        user0_.username as username2_0_0_,
        user0_.password as password3_0_0_,
        user0_.address as address4_0_0_ 
    from
        user user0_ 
    where
        user0_.uid=?
User(uid=1, username=GJXAIOU, password=GJXAIOU, address=江苏)
```

### （二）修改操作

- 首先查询，然后再修改值
（1）根据id查询，返回对象
```java
// 目的：修改 uid=1 记录的 username 值
  // 首先更加 id 查询
User user = session.get(User.class, 1);
  // 然后向返回的 user 对象里面设置修改只有的值
user.setUsername("gjxaiou");
  // 最后调用 session 的方法 update 修改：执行过程：在 user 对象里面找到 uid 值，更加 uid 进行修改
session.update(user);
```
 结果：
```sql
Hibernate: 
    select
        user0_.uid as uid1_0_0_,
        user0_.username as username2_0_0_,
        user0_.password as password3_0_0_,
        user0_.address as address4_0_0_ 
    from
        user user0_ 
    where
        user0_.uid=?
Hibernate: 
    update
        user 
    set
        username=?,
        password=?,
        address=? 
    where
        uid=?
```


### （三）删除操作
先查询再删除
- 调用 session 里面 delete 方法实现
```java
// 删除操作：第一种：根据 id 查询对象
User user = session.get(User.class, 2);
session.delete(user);

// 第二种方法
User user = new User();
user.setUid(2);
session.delete(user);
```
测试结果：
```sql
Hibernate: 
    select
        user0_.uid as uid1_0_0_,
        user0_.username as username2_0_0_,
        user0_.password as password3_0_0_,
        user0_.address as address4_0_0_ 
    from
        user user0_ 
    where
        user0_.uid=?
Hibernate: 
    delete 
    from
        user 
    where
        uid=?
```


## 四、实体类对象状态（概念）

实体类状态有三种：瞬时态、持久态、托管态（游离态）；
- 瞬时态：对象里面没有 id 值，对象与 session 没有关联
瞬时态也称为临时态或者自由态，它的实例是由 new 命令创建和开辟内存空间的对象，**不存在持久化标识 OID（相当于主键值），并且尚未与 Hibernate Session 关联**，在数据库中也没有记录，失去引用之后会被 JVM 回收；瞬时态对象在内存中是孤立存在的，和数据库中数据没有任何关联，仅仅是一个信息携带的载体；
```java
User user = new User();
user.setUsername("zhangsan");
user.setPassword("123");
user.setAddress("jiangsu");
```

- 持久态：对象里面有 id 值，对象与 session 关联
持久态对象存在持久化标识 OID，同时加入了 Session 缓存中，并且相关联的 Session 没有关闭，在数据库中有对应的记录，每条记录只对应唯一的持久化对象，**持久化对象是在事务还未提交前变成持久态的**。
`User user = session.get(User.class, 1);` 

- 托管态：对象有 id 值，对象与 session 没有关联
同时称为离线态或者游离态，当某个持久化状态的实例与 Session 的关联被关闭的时候就变成了托管态。**托管态对象存在持久化标识 OID，并且仍然与数据库中的数据存在关联，只是失去了与当前 Session 的关联**，托管状态对象发生改变的时候 Hibernate 不能检测到。
```java
User user = new User();
user.setUid(3);
```

**测试三种状态：**

```java
@Test
public void stateTest(){
    Configuration configuration = new Configuration();
    configuration.configure();

    SessionFactory sessionFactory = configuration.buildSessionFactory();
    Session session = sessionFactory.openSession();

    Transaction transaction = session.beginTransaction();
    
    User user = new User();
    // 瞬时态对象：没有持久化标识 OID，没有被 session 管理
      // 这里的 user 对象由 new 关键字创建，还没有与 Session 关联 
    
    user.setUsername("persistent");
    Serializable id = session.save(user);
    // 这里是持久态对象：有持久化标识 OID，被 session 管理
      // 这里执行 save() 方法之后，user 对象就被纳入了 Session 的管理范围，该对象就变成了持久化对象，同时 Session 的事务还没有被提交
   
    transaction.commit();

    session.close();
    sessionFactory.close();
    System.out.println(user);
    // 这里是托管态对象：有持久化标识 OID，没有被 session 管理
       // 这里执行 commit() 并且关闭 Session之后，customer 对象与 Session 的关联被关闭，这时候 customer 对象变成了托管态
}
```

**Hibernate 持久化对象三种状态之间的转换**

![持久化对象三种状态之间转换](FrameDay04_2%20Hibernate.resource/%E6%8C%81%E4%B9%85%E5%8C%96%E5%AF%B9%E8%B1%A1%E4%B8%89%E7%A7%8D%E7%8A%B6%E6%80%81%E4%B9%8B%E9%97%B4%E8%BD%AC%E6%8D%A2.png)


- 演示操作实体类对象的方法
saveOrUpdate方法：实现添加、实现修改
```java
User user = new User();
user.setUsername("jack");
user.setPassword("123");
user.setAdress("shanghai");
// 实体类对象状态是瞬时态，做添加操作
session.saveOrUpdate(user);

User user = new User();
user.setUsername("tom");
user.setPassword("124");
user.setAdress("beijing");
// 实体类对象状态是托管态，做修改操作
session.saveOrUpdate(user);

User user = session.get(User.class, 7);
user.setUsername("lilei");
// 实体类对象状态是持久态，做修改操作
session.saveOrUpdate(user);
```


## 五、Hibernate 的一级缓存

### （一）缓存的含义
数据存到数据库里面，数据库本身是文件系统，使用流方式操作文件效率不是很高。
（1）把数据存到内存里面，不需要使用流方式，可以直接读取内存中数据
（2）把数据放到内存中，提供读取效率

### （二）Hibernate 缓存
hibernate 框架中提供很多优化方式，hibernate 的缓存就是一个优化方式；
**Hibernate 的一级缓存就是指：Session 缓存**，Session 缓存是一个 内存空间，用来存放相互管理的 Java 对象；
在使用Hibernate查询对象的时候， 首先会使用对象屈性的 OID值在Hibernate的一级缓存中进行查找，如果找到匹配OID值的对象，就直接将该对象从一级缓存中取出使用，不再查询数据库，否则会去数据库中进行查找；同时当从数据库中查询到所需数据时候，该数据信息也会放置到一级缓存中。
在Session 接口的实现中包含**一系列的Java集合， 这些Java集合构成了Session 缓存**。只要Session 实例没有结束生命周期，存放在它缓存中的对象也不会结束生命周期。固一级缓存也被称为是Session基本的缓存。


- hibernate 缓存特点：
**第一类：hibernate 的一级缓存**
- hibernate 的一级缓存默认打开的；
- hibernate 的一级缓存使用范围，是 session 范围，从 session 创建到 session 关闭范围；
- hibernate 的一级缓存中，存储数据必须是持久态数据；
- 当应用程序调用Session接口的save()、update() 、saveOrUpdate时， 如果Session缓存中没有相应的对象，Hibernate就会自动的把从数据库中查询到的相应对象信息加入到一级缓存中去。
- 当调用Session接口的load()、get()方法， 以及Query接口的list()、iterator()方法时， 会判断缓存中是否存在该对象， 有则返回， 不会查询数据库， 如果缓存中没有要查询对象， 再去数据库中查询对应对象， 并添加到一级缓存中。
- 当调用Session的close()方法时，Session缓存会被清空。

第二类 hibernate的二级缓存
（1）目前已经不使用了，替代技术 redis
（2）二级缓存默认不是打开的，需要配置
（3）二级缓存使用范围，是 sessionFactory 范围

### （三）验证一级缓存存在
- 验证方式
（1）首先根据uid=6 查询，返回对象
（2）其次再根据uid=6 查询，返回对象，看两者返回对象是否相同；
```java
// 执行第一个 get 方法是否查询数据库，是否发送 SQL 语句
User user1 = session.get(User.class, 2); 
System.out.println(user1);

// 执行第二个 get 方法是否查询数据库，是否发送 SQL 语句
User user2 = session.get(User.class, 2);
System.out.println(user2);

System.out.println(user1.equals(user2));
```
程序结果：
```sql
Hibernate: 
    select
        user0_.uid as uid1_0_0_,
        user0_.username as username2_0_0_,
        user0_.password as password3_0_0_,
        user0_.address as address4_0_0_ 
    from
        user user0_ 
    where
        user0_.uid=?
User(uid=2, username=GJXAIOU, password=GJXAIOU, address=江苏)
User(uid=2, username=GJXAIOU, password=GJXAIOU, address=江苏)
true

```

从执行结果中可以看出（可以使用 debug 看出）：（注：如果没有记录，使用 equals 会报 NPE）
第一步执行get方法之后，发送sql语句查询数据库
第二个执行get方法之后，没有发送sql语句，查询一级缓存内容

### （四）Hibernate一级缓存执行过程

![一级缓存执行过程](FrameDay04_2%20Hibernate.resource/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

### （五）Hibernate一级缓存特性

持久态会自动更新数据库，即就相等于不执行 update也可以，但是会到来一系列的问题；**Hibernate 的持久态对象能够自动更新数据库，依赖的就是一级缓存**
```java
User user = session.get(User.class, 7);
user.setUsername("hanmeimei");
// session.update(user);
```

-  执行过程（了解）

![一级缓存具体执行过程](FrameDay04_2%20Hibernate.resource/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E5%85%B7%E4%BD%93%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

## 六、Hibernate事务操作

### （一）事务相关概念
事务就是由一条或者多条操作数据库的 SQL 语句组成的一个不可分割的工作单元；只有事务中所有操作都正常完成，整个事务才会提交到数据库，否则就会回滚。


**事务四大特性 ACID：**
- 原子性(Atomic) : 表示将事务中所做的操作捆绑成一个不可分割的单元， 即对事务所进行的数据修改等操作， 要么全部执行， 要么全都不执行。
- 一致性(Consistency) : 表示事务完成时， 必须使所有的数据都保持一致状态。
- 隔离性Clsolation) : 指一个事务的执行不能被其它事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的， 并发执行的各个事务之间不能互相干扰。
- 持久性(Durability) : 持久性也称永久性(permanence) , 指一个事务一旦提交， 它对数据库中数据的改变就应该是永久性的。提交后的其他操作或故障不会对其有任何影响。

**不考虑隔离性，在并发是会产生问题**
（1）脏读：一个事务读取到另一个事务未提交的数据。
（2）不可重复读：一个事务读到了另一个事务已经提交的update 的数据，导致在同一个事务中的多次查询结果不一致。
（3）幻读：一个事务读到了另一个事务已经提交的insert 的数据，导致在同一个事务中的多次查询结果不一致。

- 设置事务隔离级别
  - 读未提交(Read Uncomm让ted, 1 级）： 一个事务在执行过程中， 既可以访问其他事务未提交的新插入的数据， 又可以访问未提交的修改数据。如果一个事务已经开始写数据， 则另外一个事务则不允许同时进行写操作， 但允许其他事务读此行数据。此隔离级别可防止丢失更新。
  - 已提交读(Read Committed, 2 级）： 一个事务在执行过程中， 既可以访问其他事务成功提交的新插入的数据， 又可以访问成功修改的数据。读取数据的事务允许其他事务继续访问该行数据，但是未提交的写事务将会禁止其他事务访问该行。此隔离级别可有效防止脏读。
  - 可重复读(Repeatable Read, 4 级）： 一个事务在执行过程中， 可以访问其他事务成功提交的新插入的数据， 但不可以访问成功修改的数据。读取数据的事务将会禁止写事务（但允许读事务）， 写事务则禁止任何其他事务。此隔离级别可有效的防止不可重复读和脏读。
  - 序列化／串行化(Serializable, 8 级）： 提供严格的事务隔离。它要求事务序列化执行， 事务只能一个接着一个地执行， 但不能并发执行。此隔离级别可有效的防止脏读、不可重复读和幻读。



| 隔离级别 | 含义|
|:--|---|
|READ UNCOMMITTED |允许你读取还未提交的改变了的数据。可能导致脏、幻、不可重复读|
|READ COMMITTED |允许在并发事务已经提交后读取。 可防止脏读，但幻读和不可重复读仍可发生|
|REPEATABLE_READ |对相同字段的多次读取是一致的，除非数据被事务本身改变。 可防止 脏、不可重复读，但幻读仍可能发生。|
|SERIALIZABLE |完全服从ACID的隔离级别，确保不发生脏、 幻、不可重复读。这在所有的隔离级别中是最慢的，它是典型的通过 完全锁定在事务中 涉及的数据表来完成的。|
事务的隔离级别，是由数据库提供的，并不是所有数据库都支持四种隔离级别
• MySQL : `READ_UNCOMMITTED` 、`READ_COMMITTED` 、`REPEATABLE_READ` 、`SERI ALIZABLE` (默认REPEATABLE—READ)
• Oracle: `READ_UNCOMMITTED`、`READ_COMMITTED`、`SERIALIZABLE`（默认READ_COMMITTED )
在使用数据库时候，隔离级别越高，安全性越高，性能越低。
实际开发中，不会选择最高或者最低隔离级别，选择READ_COMMITTED ( oracle 默认） 、
REPEATABLE_READ (mysql默认）

mysql默认隔离级别 repeatable read

### （二）Hibernate事务代码规范写法
代码结构
```java
try {
  开启事务
  提交事务
}catch() {
  回滚事务
}finally {
  关闭
}
```
以前代码修改之后的标准示例：
```java
public class TransactionTest {
    @Test
    public void StandardTransaction(){
        SessionFactory sessionFactory = null;
        Transaction transaction = null;
        Session session = null;

        try {
            sessionFactory = HibernateUtil.getSessionFactory();
            session = sessionFactory.openSession();
            transaction = session.beginTransaction();

            User user = new User();
            user.setUsername("GJXaiou");
            user.setPassword("GJXaiou");
            user.setAddress("鼓楼");

            session.save(user);

            // int i = 10/0;

            // 提交事务
            transaction.commit();
        }catch (Exception e) {
            e.printStackTrace();
            // 事务回滚
            transaction.rollback();
        }finally {
            // 关闭操作
            session.close();
            sessionFactory.close();
        }
    }
}
```

**设置事务隔离级别**
可以在 hibernate.cfg.xml 的 `<session-factory>`标签中设置：`<property name = "hibernate.connection.isolation">4</property>`，中间的值可以自己设定为：1,2,4,8；

**事务控制应该在 Service 层实现，不应该在 DAO 层实现**，可以在 Service 中调用多个 DAO 实现一个业务逻辑操作，

![在Service层添加事务](FrameDay04_2%20Hibernate.resource/%E5%9C%A8Service%E5%B1%82%E6%B7%BB%E5%8A%A0%E4%BA%8B%E5%8A%A1.png)

**保证 Service 和多个 DAO 使用的 Session 是同一个的方法：**
- 向下传递：可以在业务层获取 Session，并将 Session 作为参数传递给 DAO；
- 绑定到当前线程：使用 ThreadLocal 将业务层获取到的 Session 绑定到当前线程中，然后在 DAO 中获取 Session 的时候，都从该线程中获取；

### （三）Hibernate绑定session
用于保证单线程

- session类似于jdbc的connection，之前web阶段学过 ThreadLocal

- Hibernate 可以实现与本地线程绑定session

- 获取与本地线程session 步骤：
  - **在hibernate核心配置文件中配置**：`<property name="hibernate.current_session_context_class">thread</property>`
其实一共是有三个属性值，对应 Hibernate 提供的三种管理 Session 对象的方法：
    - thead：Session 对象的生命周期和本地线程绑定；
    - jta：Session 对象的生命周期与 JTA 事务绑定；
    - managed：Hibernate 委托程序来管理 Session 对象的生命周期；

  - 调用sessionFactory里面的方法得到
```java
// 提供返回与本地线程绑定的 session 方法
public static Session getSessionObject(){
    return sessionFactory.getCurrentSession();
}
```

   - 然后实现session时候使用上面方法构建即可；
- 获取与本地线程绑定session时候，如果关闭session会报错，**不需要手动关闭了**


## 七、Hibernate 的 api 使用（下面三个都是查询的）

### （一）Query对象
**Query 代表面向对象的一个 Hibernate 查询操作**；
- 使用query对象，不需要写sql语句，但是写hql语句：hql：hibernate query language，hibernate提供查询语言，这个hql语句和普通sql语句很相似
- hql和sql语句区别：
  - 使用sql操作表和表字段
  - 使用hql操作实体类和属性

- 查询所有hql语句：`from 实体类名称`
也可以用于查询所有、条件查询、分页查询等；

- Query对象使用
（1）创建Query对象
（2）调用query对象里面的方法得到结果
```java
// 首先创建 Query 对象，然后在方法中写 hql 语句
Query query = session.createQuery("from User");

// 调用 query 对象里面的方法得到结果
List<User> list = query.list();
for(User user : list){
    System.out.println(user);
}
```

### （二）Criteria对象
**Criteria 是一个完全面向对象，可拓展的条件查询 API；**
Criteria 查询又称为：QBC（Query By Criteria）查询；
使用这个对象查询操作，但是使用这个对象时候，不需要写语句，直接调用方法实现

- 实现过程
（1）创建criteria对象
（2）调用对象里面的方法得到结果
```java
// 首先创建 criteria 对象，方法中的参数是实体类 class
Criteria criteria = session.createCriteria(User.class);
// 调用方法得到结果
List<User> list = criteria.list();
```

### （三）SQLQuery对象

- 使用 hibernate 时候，调用底层 sql 实现
- 实现过程
（1）创建对象
（2）调用对象的方法得到结果
```java
// 首先创建 SQLQuery 对象，参数是普通的 SQL 语句
SQLQuery sqlQuery = session.createSQLQuery("select * from user");
// 调用 sqlQuery 里面的方法，返回 list 集合，默认里面每部分是数组结构
List<Object[]> list = sqlQuery.list();
for(Object[] objects:list){
  System.out.println(Arrays.toString(objects));
}
```
 上面代码中返回 list 集合每部分是数组
 通过修改使得下面代码中返回 list 中每部分是对象形式
```java
// 首先创建 SQLQuery 对象，参数是普通的 SQL 语句
SQLQuery sqlQuery = session.createSQLQuery("select * from user");
// 返回的 list 中每部分都是对象形式
sqlQuery.addEntity(User.class);
// 调用 sqlQuery 里面的方法，返回 list 集合
List<User> list = sqlQuery.list();
```


