# FrameDay04_1 Hibernate

## Hibernate 整体规划

Hibernate入门和基本操作
Hibernate概念和api使用
Hibernate配置一对多和多对多
Hibernate查询操作


## 一、WEB 内容回顾

### （一）JavaEE 三层结构

- web层：struts2、SpringMVC 框架
- service层：spring 框架
- dao层：Hibernate、MyBatis 框架
（1）对数据库进行 CRUD 操作

![JavaEE三层架构](FrameDay04_1%20Hibernate.resource/JavaEE%E4%B8%89%E5%B1%82%E6%9E%B6%E6%9E%84.png)

## 二、Hibernate 概述

### （一）Hibernate框架概念（重点）

- hibernate 框架应用在 javaee 三层结构中 dao 层框架
- 在 dao 层里面做对数据库 CUED 操作，使用 hibernate 实现 CUED 操作，hibernate 底层代码就是 jdbc，hibernate 对 jdbc 进行封装，使用 hibernate 好处，不需要写复杂 jdbc 代码了，不需要写 sql 语句实现；
- hibernate是开源的轻量级的框架；
- hibernate版本：目前一般使用 Hibernate5.x；

### （二）什么是 ORM 思想（重点）

- hibernate 使用 orm 思想对数据库进行 crud 操作，完成数据持久化；是一个全自动 ORM 框架
- **ORM：Object Relational Mapping，对象关系映射**，利用描述对象和数据库表之间映射的元数据，自动把 Java 应用程序中对象持久化到关系型数据库的表中；
![ORM原理](FrameDay04_1%20Hibernate.resource/ORM%E5%8E%9F%E7%90%86.png)

文字描述：
  - 让实体类和数据库表进行一一对应关系
    - 让实体类首先和数据库表对应；
    - 让实体类属性 和 表里面字段对应；
  - 不需要直接操作数据库表，而只需要操作表对应实体类对象即可；
  - 通过配置 xml 文件达到两者的映射；

## 三、Hibernate 入门

### （一）搭建 Hibernate 环境（重点）

- 第一步导入Hibernate的jar包
除了需要 Hibernate 的核心包之外，因为使用 hibernate 时候，有日志信息输出，hibernate 本身没有日志输出的 jar 包，导入其他日志的 jar 包以及不要忘记还有 mysql 驱动的jar包。
```jar
// required 目录下的核心包
antlr.jar
dom4j.jar
geronimo-jta.jar
hibernate-commons-annotation.jar
hibernate-core.jar
hibernate-jpa.jar
hibernate-entitymanager.jar
jandex.jar
javassist.jar
jboss-logging.jar
// 数据库驱动包
mysql-connector-java.jar
// 日志记录包
log4j.jar
slf4j-api.jar
slf4j-log4j.jar
```

- 第二步创建实体类
```User_java
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class User {
    // Hibernate 要求实体类中至少有一个属性的唯一的；
    private Integer uid;
    private String username;
    private String password;
    private String address;
}
```
使用 hibernate 时候，不需要自己手动创建表，hibernate 会帮助创建表；

- 第三步  配置实体类和数据库表一一对应关系（映射关系）
使用配置文件实现映射关系

  - （1）创建xml格式的配置文件
    - 映射配置文件名称和位置没有固定要求
    - 建议：在实体类所在包里面创建：`实体类名称.hbm.xml`

  - （2）配置是 xml 格式，在配置文件中首先引入 xml 约束；
    - 在 hibernate 里面引入的约束是 dtd 约束；
dtd 的名称为：`hibernate-mapping-3.0.dtd`
对应文件的配置内容为：
```User_hbm_xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
```

   - （3）配置映射关系
```User_hbm_xml
<hibernate-mapping>
    <!--配置类和数据库中表的对应关系 -->
        <!--class 标签中：name 为实体类的全路径，table 为数据库表的名称-->
    <class name="com.gjxaiou.pojo.User" table="user">
        <!--配置实体类 id 值和表的 id 值对应，Hibernate 要求实体类中有一个属性作为唯一值，同时表中有一个字段作为唯一值-->
            <!--id 标签中：name 为实体类中的 id 属性名称，column 为生成表的字段名称，length：字段长度-->
        <id name="uid" column="uid">
            <!--设置数据库表中增长策略，native 表示生成表 id 值就是主键且自动增长-->
            <generator class="native"></generator>
        </id>
        <!--配置其他属性和表字段进行对应，name 表示实体类属性名称，column 表示生成表字段名称-->
        <property name="username" column="username"></property>
        <property name="password" column="password"></property>
        <property name="address" column="address"></property>
    </class>
</hibernate-mapping>
```

- 第四步创建 Hibernate 的核心配置文件
   - 核心配置文件格式 xml，但是**核心配置文件名称和位置固定的**
      - 位置：必须 src 下面；
      - 名称：必须 hibernate.cfg.xml；
   - 引入dtd约束
```hibernate_cfg_xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
```

   - hibernate操作过程中，只会加载核心配置文件，其他配置文件不会加载，整配置文件可以化为三个部分：
第一部分：配置数据库信息必须的
第二部分：配置Hibernate信息可选的
第三部分：把映射文件放到核心配置文件中
```hibernate_cfg_xml
<hibernate-configuration>
    <session-factory>
        
<!--第一部分：配置数据库信息-->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/lianxi</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">GJXAIOU</property>

<!--第二部分：配置 Hibernate 信息-->
            <!--在控制台输出底层对应的 SQL 语句-->
        <property name="hibernate.show_sql">true</property>
            <!--按照格式输出 SQL 语句-->
        <property name="hibernate.format_sql">true</property>
            <!--设置让 Hibernate 自动创建表，update：表示如果已经有表则更新，如果没有则创建，none 表示不用 Hibernate 自动创建表；create：表示每次都创建一个新表，create-drop :每次都创建一个新表，执行程序结束后删除该表；validate：只会使用原来的表，对映射关系进行校验-->
        <property name="hibernate.hbm2ddl.auto">update</property>
            <!--配置数据库方言，即让 Hibernate 根据不同的数据库自动解析为对应的语句-->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
        
<!--第三部分：把映射文件加载到核心配置文件中-->
        <mapping resource="User.hbm.xml"></mapping>

    </session-factory>
</hibernate-configuration>
```


### （二）实现添加操作

* 第一步  加载 hibernate 核心配置文件
* 第二步  创建 SessionFactory 对象
* 第三步  使用 SessionFactory 创建 session 对象
* 第四步  开启事务
* 第五步  写具体逻辑 crud 操作
* 第六步  提交事务
* 第七步  关闭资源

```AddTest_java
public class AddTest {
    public static void main(String[] args) {
        // 步一：加载 Hibernate 配置文件
            // 首先会在 src 下找到名称为：hibernate.cfg.xml，然后在 hibernate 里面封装对象
        Configuration configuration = new Configuration();
        configuration.configure();

        // 步二：创建 SessionFactory 对象
            // 读取 hibernate 核心配置文件内容，创建 sessionFactory，同时会根据映射关系在配置的数据库中创建表
        SessionFactory sessionFactory = configuration.buildSessionFactory();

        // 步三：使用 SessionFactory 创建 session 对象（session 类似于连接）
        Session session = sessionFactory.openSession();

        // 步四：开启事务
        Transaction transaction = session.beginTransaction();

        // 步五：写具体的 CURD 操作
            // 添加功能
        User user = new User();
        user.setUsername("GJXAIOU");
        user.setPassword("GJXAIOU");
        user.setAddress("江苏");
            // 调用 session 的方法实现添加
        session.save(user);
        
        // 步六：提交事务
        transaction.commit();
        
        // 步七：关闭资源
        session.close();
        sessionFactory.close();
    }
}

```
程序结果：
```sql
Hibernate: 
    
    create table user (
       uid integer not null auto_increment,
        username varchar(255),
        password varchar(255),
        address varchar(255),
        primary key (uid)
    ) engine=MyISAM
Hibernate: 
    insert 
    into
        user
        (username, password, address) 
    values
        (?, ?, ?)
```
数据库中生成了表，同时数据库表中是否有记录；

首先创建Configuration类的实例， 并通过它来读取并解析配置文件hibemate.cfg.xml。 然后创建SessionFactory 读取解析映射文件信息， 并将Configuration 对象中的所有配置信息拷贝到SessionFactory内存中。 接下来， 打开Session, 让SessionFactory提供连接， 并开启一个事务， 之后创建对象， 向对象中添加数据， 通过session.save()方法完成向数据库中保存数据的操作。 最后提交事务， 并关闭资源。


## 四、Hibernate配置文件详解

### （一）Hibernate 映射配置文件（重点）

- 映射配置文件名称和位置没有固定要求
- 映射配置文件中，**标签name属性值写实体类相关内容**
  - class标签name属性值为实体类全路径
  - id标签和property标签name属性值 为实体类属性名称
- id标签和property标签，column属性可以省略的
  - 不写值和name属性值一样的（建议书写）
-  property标签type属性，设置生成表字段的类型，一般hibernate会自动对应类型

### （二）Hibernate核心配置文件

- 写配置文件位置要求
```hibernate_cfg_xml
<hibernate-configuration>
    <session-factory>
        <!--书写配置文件-->
    </session-factory>
</hibernate-configuration>
```

- 配置三部分要求
  - 数据库部分必须的
  - hibernate 部分可选的
  - 映射文件必须的
- 核心配置文件名称和位置固定的
  - 位置：src 下面
  - 名称：hibernate.cfg.xml


## 五、Hibernate 核心 API

### （一）Configuration
```java
// 作用：到src下面找到名称 hibernate.cfg.xml 配置文件，创建对象，把配置文件放到对象里面（加载核心配置文件）；
Configuration configuration = new Configuration();
configuration.configure();
```
下面是 configure()方法 部分代码 L243：

```java
	public Configuration configure() throws HibernateException {
		return configure( StandardServiceRegistryBuilder.DEFAULT_CFG_RESOURCE_NAME );
	}
// 其中： 	public static final String DEFAULT_CFG_RESOURCE_NAME = "hibernate.cfg.xml";
```

因此上面作用就是默认是在 src 目录下寻找 hibernate.cfg.xml 配置文件，当然配置文件路径可以自定义，只需要在使用：`configuration.configure("xml 文件位置");` 即可；路径的根目录为 src，也就是说如果放置在 src/config 包中，则路径为：`/config/hibernate.cfg.xml`

可以只配置文件路径原理如下：

```java
// 这里取自 Configuration.java 的 L257
public Configuration configure(String resource) throws HibernateException {
    standardServiceRegistryBuilder.configure( resource );
    properties.putAll( standardServiceRegistryBuilder.getSettings() );
    return this;
}
```

比较这里和上面的代码就可以发现，这里的 `configure()`方法进行了重载，如果有参数就会调用含参的方法，该方法本质上调用了`localConfigXmlResource()`方法，该方法代码如下下：

```java
public StandardServiceRegistryBuilder configure(String resourceName) {
    return configure( configLoader.loadConfigXmlResource( resourceName ) );
}
```

`localConfigXmlResource()`代码如下

```java
public LoadedConfig loadConfigXmlResource(String cfgXmlResourceName) {
    // 将 xml 文件变成输入流
		final InputStream stream = bootstrapServiceRegistry.getService( ClassLoaderService.class ).locateResourceStream( cfgXmlResourceName );
    // 如果没有内容，报错找不到该文件
		if ( stream == null ) {
			throw new ConfigurationException( "Could not locate cfg.xml resource [" + cfgXmlResourceName + "]" );
		}

		try {
            // unmarshal() 方法将传入的流和资源类型，该方通过 FraxFactory 创建一个createXMLEventReader 读取 xml 文件
			final JaxbCfgHibernateConfiguration jaxbCfg = jaxbProcessorHolder.getValue().unmarshal(
					stream,
					new Origin( SourceType.RESOURCE, cfgXmlResourceName )
			);

			return LoadedConfig.consume( jaxbCfg );
		}
		finally {
			try {
				stream.close();
			}
			catch (IOException e) {
				log.debug( "Unable to close cfg.xml resource stream", e );
			}
		}
	}
```

- 可以用于加载映射文件
因为 Hibernate 的核心配置文件可以使用上面的 xml 形式，也可以使用. properties 文件形式，其中使用键值对的形式存放信息，默认文件名：hibernate.properties；使用的时候，如果使用 properties 作为核心配置文件，其他的基本配置可以使用键值对，但是其中加载映射文件部分没法读取，这里就可以使用 configuration 进行手动加载映射文件。、
`configuration.addResource("com/gjxaiou/domain/Customer.hbm.xml");`

### （二）SessionFactory（重点）

SessionFactory接口负责Hibernate的初始化和建立Session对象。 它在Hibernate中起到 个缓冲区作用，Hibernate可以将自动生成的SQL语句、映射数据以及某些可重复利用的的数据放在这个缓冲区中。 同时它还保存了对数据库配置的所有映射关系， 维护了当前的二级缓存。

- 特点：
  - 线程安全：它的同一个实例能够提供给多个线程共享；
  - 重量级：不能随便的创建和销毁它的实例

- 正常使用方式：
`SessionFactory sessionFactory = configuration.buildSessionFactory();`
使用 configuration 对象创建 sessionFactory 对象
创建 sessionfactory 过程中做的事情：根据核心配置文件中，其中有数据库配置，有映射文件部分，到数据库里面根据映射关系把表创建（如果是自动创建，需要配置下面一行）
`<property name="hibernate.hbm2ddl.auto">update</property>`

- 但是创建 sessionFactory 过程中，这个过程特别耗资源的，因此在 hibernate 操作中，建议一个项目一般创建一个 sessionFactory 对象【只有当应用中存在多个数据源时候，才为每一个数据源建立一个 SessionFactory 实例】；实现方式：使用工具类创建，即通过静态代码块实现，因为静态代码块在类加载的时候执行，而且仅仅执行一次；
```java
public class HibernateUtil {
     static Configuration configuration = null;
     static SessionFactory sessionFactory = null;
     static{
        // 加载核心配置文件
        configuration = new Configuration();
        configuration.configure();
        sessionFactory = configuration.buildSessionFactory();
        
    }
     public static SessionFactory getSessionFactory(){
         return sessionFactory;
     }
}
```
使用的时候，直接调换原来一大块代码，直接使用 ：`SessionFactory sessionfactory = sessionFactoryUtil.getSessionFactory();`

同时 SessionFactory 内部维护了一个连接池，当我们需要第三方连接池如：C3P0。需要手动配置；
- 需要的 jar 包
  - c3p0.jar
  - hibernate-c3p0.jar
  - mchange-commons-java.jar

```xml
＜!--配置C3PO连接池--＞
<property name= "connection.provider_c1ass"> org.hibernate.connection.C3POConnectionProvider </property> 
<!--在连接池中可用的数据库连接的最少数目--＞
<property name= "c3p0.min_size ">5</property>
<!--在连接池中所有数据库连接的最大数目 －－＞ 
<property name= "c3p0.max_size ">20</property> 
<!--设定数据库连接的过期时间，以秒为单位，如果连接池中的某个数据库连接处于空闲状态的时间超过了timeout时间，就会从连接池中消除--＞
<property name= "c3p0. timeout">120</property>
<!--每3000秒检查所有连接池中的空闲连接以秒为单位--＞
<property name= "c3p0. idle_ test_period ">3000</property> 

```




### （三）Session（重点）
Session 接口负责执行被持久化对象的 CRUD 操作，Session 对象是**非线程安全**的，多个并发线程同时操作一个 Session 实例的时候，可能导致 Session 数据存取混乱，应该避免多线程共享同一个 Session 实例，因为其是轻量级的，因此实例的创建和销毁不需要消耗太多的资源，同时它拥有 Hibernate 的一级缓存（该缓存主要用于存放当前工作单元加载的对象）。 这里的 session 仅仅指的是 Hibernate 的 session， HttpSession 对象称为：用户 session；

Session是应用程序与数据库之间交互操作的 个单线程对象，是Hibernate运作的中心， 它的主要功能是为持久化对象提供创建、 读取和删除的能力，所有待久化对象必须在session的管理下才可以进行待久化操作。

通过 SessionFactory 获取 Session 实例有两种方式，一种是 openSession()，另一种是 getCurrentSession();，
```java
Session session = SessionFactory.openSession();
Session session = SessionFactory.getCurrentSession();
```
**两者的区别：**
- 使用 openSession 方法获取 Session 实例时候，SessionFactory 直接创建一个新的 Session 实例，并且在使用完成之后需要调用 close 方法进行手动关闭；
- 使用 getCurrentSession 方法创建的 Session 实例会被绑定到当前线程中，它在提交或者回滚操作时候会自动关闭；

- session 类似于 jdbc 中 connection
- 调用 session 里面不同的方法实现 crud 操作
  * 添加 save 方法
  * 修改 update 方法
  * 删除 delete 方法
  * 根据 id 查询 get 方法
- session 对象是单线程对象
  - session 对象不能共用，只能自己使用

### （四）Transaction
该接口用于管理实务，对底层的事务接口进行了分装，Transaction 接口的事务对象是通过 Session 对象开启的，开启方法见下面：
- 事务对象：
开启事务：`Transaction transaction = session.beginTransaction();`
- 事务操作方法：事务提交和事务回滚
`transaction.commit();` 和`transaction.rollback();`
- 事务四个特性
原子性、一致性、隔离性、持久性


