# FrameDay04_3 Hibernate

**上节内容**
* 实体类编写规则
* hibernate主键生成策略
* 实体类操作
  * crud操作
  * 实体类对象状态
* hibernate的一级缓存
* hibernate的事务操作
  * 事务代码规则写法
* hibernate其他的api（查询）

**今天内容**

- 列表功能实现
- 表与表之间关系回顾
  - 一对多（客户和联系人）
  - 多对多（用户和角色）
- hibernate一对多操作
  - 一对多映射配置
  - 一对多级联保存
  - 一对多级联删除
  - inverse属性
- hibernate多对多操作
  - 多对多映射配置
  - 多对多级联保存（重点）
  - 多对多级联删除
  - 维护第三张表


## 一、表与表之间关系回顾（重点）
Hibernate 框架实现了 ORM 的思想， 将关系数据库中表的数据映射成对象， 使开发人员把对数据的操作转化为对对象的操作， Hibernate 的关联关系映射主要包括多表的映射配置、数据的增加、删除等。

- 一对多
  - 例如分类和商品关系，一个分类里面有多个商品，一个商品只能属于一个分类
  
- 客户和联系人是一对多关系
客户：与公司有业务往来，例如百度、新浪；
  联系人：公司里面的员工，百度里面有很多员工，联系员工；
    - 客户是一，联系人是多
    - 一个客户里面有多个联系人，一个联系人只能属于一个客户
  
- **一对多建表：通过外键建立关系**：在多的一方创建外键指向一的一方的主键；
  
    ![一对多建表示例](FrameDay04_3%20Hibernate.resource/%E4%B8%80%E5%AF%B9%E5%A4%9A%E5%BB%BA%E8%A1%A8%E7%A4%BA%E4%BE%8B.png)
  
- 多对多
  - 例如订单和商品关系，一个订单里面有多个商品，一个商品属于多个订单
  - 用户和角色多对多关系
    - 用户： 小王、小马、小宋
    - 角色：总经理、秘书、司机、保安
  比如小王 可以 是总经理，可以是司机
  比如小宋 可以是司机，可以是秘书，可以保安
  比如小马 可以是 秘书，可以是总经理
    - 一个用户里面可以有多个角色，一个角色里面可以有多个用户
  - **多对多建表：创建第三张表维护关系**：中间表中至少两个字段作为外键分别指向多对多双方的主键；

![多对多建表示例](FrameDay04_3%20Hibernate.resource/%E5%A4%9A%E5%AF%B9%E5%A4%9A%E5%BB%BA%E8%A1%A8%E7%A4%BA%E4%BE%8B.png)

- 一对一
  - 例如班级与班长之间的关系，一个班级只能有一个班长，一个班长只能在一个班级任职；
  - **一对一建表：**两种方式：
    - 唯一的外键对应：即假设一对一是任意一方为多，然后在多的一方创建外键指向一的一方的主键，然后将外键设置为唯一；
    - 主键对应：一方的主键作为另一方的主键；

**对应于 Hibernate 中就是使用 Java 对象关系进行描述：**

![Hibernate中表之间关系](FrameDay04_3%20Hibernate.resource/Hibernate%E4%B8%AD%E8%A1%A8%E4%B9%8B%E9%97%B4%E5%85%B3%E7%B3%BB.png)



**下面用到的一个文件：HibernateUtil.java**
```java
public class HibernateUtil {
    static Configuration configuration = null;
    static SessionFactory sessionFactory = null;

    static {
        // 加载核心配置文件
        configuration = new Configuration();
        configuration.configure();
        sessionFactory = configuration.buildSessionFactory();

    }

    /**
     * 提供 session 与本地线程绑定的方法
     * @return 与本地线程绑定之后的 Session
     */
    public static Session getSessionObject() {
        return sessionFactory.getCurrentSession();
    }

    /**
     * 提供返回 sessionFactory 方法
     * @return sessionFactory 对象
     */
    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}
```

## 二、Hibernate的一对多操作（重点）

### （一）一对多映射配置（重点）

以客户和联系人为例：客户是一，联系人是多

- 第一步：创建两个实体类，客户和联系人
```Customer_java
@Setter
@Getter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Customer {
	private Integer cid;
	private String custName;
	private String custLevel;
	private String custSource;
	private String custPhone;
	private String custMobile;
	/**
	 * 在客户实体类里面表示多个联系人，一个客户有多个联系人
	 * hibernate要求使用集合表示多的数据，使用set集合
	 */
	private Set<LinkMan> setLinkMan = new HashSet<LinkMan>();
}
```

```LinkMan_java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class LinkMan {
    private Integer lkmId;
    private String lkmName;
    private String lkmGender;
    private String lkmPhone;
    /**
     *  在联系人实体类里面表示所属客户,一个联系人只能属于一个客户
     */
    private Customer customer;
}
```

- 第二步 让两个实体类之间互相表示
  - 在客户实体类里面表示多个联系人：一个客户里面有多个联系人
需要在 Customer 实体类中添加`private Set<LinkMan> setLinkMan = new HashSet<LinkMan>();` 以及对应的 get、set 方法，完整的代码见上面；

  - 在联系人实体类里面表示所属客户：一个联系人只能属于一个客户
在 LinkMan 实体类中添加 `private Customer customer;` 以及对应的 get、set 方法，完整的代码见上面；

- 第三步：配置映射关系
一般一个实体类对应一个映射文件
**首先把映射最基本配置完成，然后在映射文件中，配置一对多关系**
  - 在客户映射文件中，表示所有联系人
```Customer_hbm_xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping >
    <class name="com.gjxaiou.pojo.Customer" table="customer">
        <id name="cid" column="cid">
            <generator class="native"></generator>
        </id>

        <property name="custName" column="custname"></property>
        <property name="custLevel" column="custlevel"></property>
        <property name="custSource" column="custsource"></property>
        <property name="custPhone" column="custphone"></property>
        <property name="custMobile" column="custmobile"></property>

        <!-- 在客户映射文件中，使用 set 标签表示所有联系人
			set标签里面有name属性：属性值写在客户实体类里面表示联系人的set集合名称（多的一方集合的属性名称）
			inverse属性默认值：false不放弃关系维护，true表示放弃关系维护
		-->
        <set name="setLinkMan" inverse="true">
            <!-- 一对多建表，需要外键建立关系
				hibernate机制：双向维护外键，在一和多那一方都配置外键
				column属性值：外键名称（多的一方外键名称）
			 -->
            <key column="clid"></key>
            <!-- 客户所有的联系人，class里面写联系人实体类全路径（多的一方的类全路径） -->
            <one-to-many class="com.gjxaiou.pojo.LinkMan"></one-to-many>
        </set>
    </class>
</hibernate-mapping>
```

  - 在联系人映射文件中，表示所属客户
```LinkMan_hbm_java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping >
    <class name="com.gjxaiou.pojo.LinkMan" table="linkman">
        <id name="lkmId" column="lkmid">
            <generator class="native"></generator>
        </id>

        <property name="lkmName" column="lkmname"></property>
        <property name="lkmGender" column="lkmgender"></property>
        <property name="lkmPhone" column="lkmphone"></property>

        <!-- 表示联系人所属客户
	        name属性：因为在联系人实体类使用customer对象表示，写customer名称
	        class属性：customer全路径
	        column属性：外键名称
        -->
        <many-to-one name="customer" class="com.gjxaiou.pojo.Customer"></many-to-one>
    </class>
</hibernate-mapping>
```

- 第四步 创建核心配置文件，把映射文件引入到核心配置文件中
```hibernate_cfg_xml
<!-- 第三部分： 把映射文件放到核心配置文件中 必须的-->
<mapping resource="Customer.hbm.xml"/> 
<mapping resource="LinkMan.hbm.xml"/>
```

测试： 只要写一个 test 开启事务即可
```java
Hibernate: 
    
    create table customer (
       cid integer not null auto_increment,
        custname varchar(255),
        custlevel varchar(255),
        custsource varchar(255),
        custphone varchar(255),
        custmobile varchar(255),
        primary key (cid)
    ) engine=MyISAM
Hibernate: 
    
    create table linkman (
       lkmid integer not null auto_increment,
        lkmname varchar(255),
        lkmgender varchar(255),
        lkmphone varchar(255),
        customer integer,
        clid integer,
        primary key (lkmid)
    ) engine=MyISAM
Hibernate: 
    
    alter table linkman 
       add constraint FKjyu2v2fsw52yp276bhmm47jtk 
       foreign key (customer) 
       references customer (cid)
Hibernate: 
    
    alter table linkman 
       add constraint FKa41307x5ixhiogw7flgf68uag 
       foreign key (clid) 
       references customer (cid)
```



### （二）一对多级联操作
级联操作是指：当主控方执行保存、更新或者删除操作时候，其关联对象（被控方）也执行相同的操作。
级联的方向性：在保存一的一方级联多的一方和在保存多的一方级联一的一方；
- 级联操作
  - 级联保存
      添加一个客户，为这个客户添加多个联系人
  - 级联删除
      删除某一个客户，这个客户里面的所有的联系人也删除

#### 1. 一对多级联保存
 需求：添加客户，为这个客户添加两个联系人
因为需要保存客户，因此客户是主控方，则在客户的配置文件中配置关系映射；
```Customer_hbm_xml
<!-- 在客户映射文件中，使用 set 标签表示所有联系人  set标签里面有name属性：属性值写在客户实体类里面表示联系人的set集合名称（多的一方集合的属性名称）  inverse属性默认值：false不放弃关系维护，true表示放弃关系维护 -->
  <set name="setLinkMan" inverse="true">
  <!-- 一对多建表，需要外键建立关系  hibernate机制：双向维护外键，在一和多那一方都配置外键  column属性值：外键名称（多的一方外键名称）
  -->
  <key column="clid"></key>
  <!-- 客户所有的联系人，class里面写联系人实体类全路径（多的一方的类全路径） -->
  <one-to-many class="com.gjxaiou.pojo.LinkMan"></one-to-many>
 </set>
```

- **复杂写法**：
使用方法
级联方向性：保存用户同时级联用户的联系人
```HibernateOneToMany_java
public class TableTest {
    @Test
    public void createOneToMany(){
        Session session = HibernateUtil.getSessionObject();
        Transaction transaction = session.beginTransaction();

        // 创建一个客户
        Customer customer = new Customer();
        customer.setCustName("Alibaba");
        customer.setCustLevel("100");
        customer.setCustMobile("110");

        // 创建两个联系人
        LinkMan linkMan1 = new LinkMan();
        linkMan1.setLkmName("GJXAIOU");
        LinkMan linkMan2 = new LinkMan();
        linkMan2.setLkmName("gjxaiou");
        linkMan2.setLkmGender("man");

        // 建立关系
        customer.getSetLinkMan().add(linkMan1);
        customer.getSetLinkMan().add(linkMan2);
        linkMan1.setCustomer(customer);
        linkMan2.setCustomer(customer);

        session.save(customer);
        session.save(linkMan1);
        session.save(linkMan2);

        transaction.commit();
    }
```
运行结果：
表一 customer 中的 cid 和表二 linkman 中的 clid 相对应；
|cid | custname | custlevel | custsource | custphone | custmobie |
|---|---|---|---|---|---|
|1|Alibaba|100| |  |110 |


|lkmid | lkmname |lkmgender|lkmphone|customer |clid |
|---|---|---|---|---|---|
|1|GJXAIOU| | |1| |
|2 |gjxaiou | man |  | 1 | |

**从上面代码可以看出是双向维护关系**
不能仅仅配置一个（就是 save 只配置一个），如果只配置了一方，就是持久态对象关联了一个瞬时态对象，瞬时对象异常；
如果真的仅仅想保留一方，就使用下面 Hibernate 提供的级联操作；
**级联操作**： 当主控发执行保存、更新或者删除操作时，其关联对象（被控方）也执行相同的操作；



- **上面对应的简化写法**
  - 一般根据客户添加联系人
第一步 在客户映射文件中进行配置
`<set name="setLinkMan" cascade="save-update">`
  - 在客户映射文件里面set标签进行配置


第二步 创建客户和联系人对象，只需要把联系人放到客户里面就可以了，最终只需要保存客户就可以了
```java
//演示一对多级联保存
	@Test
	public void testAddDemo2() {
		SessionFactory sessionFactory = null;
		Session session = null;
		Transaction tx = null;
		try {
			//得到sessionFactory
			sessionFactory = HibernateUtils.getSessionFactory();
			//得到session
			session = sessionFactory.openSession();
			//开启事务
			tx = session.beginTransaction();
			// 添加一个客户，为这个客户添加一个联系人
			//1 创建客户和联系人对象
			Customer customer = new Customer();
			customer.setCustName("百度");
			customer.setCustLevel("普通客户");
			customer.setCustSource("网络");
			customer.setCustPhone("110");
			customer.setCustMobile("999");
		
			LinkMan linkman = new LinkMan();
			linkman.setLkm_name("小宏");
			linkman.setLkm_gender("男");
			linkman.setLkm_phone("911");
			//2 把联系人放到客户里面
			customer.getSetLinkMan().add(linkman);
			//3 保存客户
			session.save(customer);
			
			//提交事务
			tx.commit();
		}catch(Exception e) {
			tx.rollback();
		}finally {
			session.close();
			//sessionFactory不需要关闭
			sessionFactory.close();
		}
	}
```



#### 2. 一对多级联删除
 需求：删除某个客户，把客户里面所有的联系人删除

**具体实现**
- 第一步： 在客户映射文件set标签，进行配置
（1）使用属性 cascade 属性值 delete（下面因为是在上面的基础上进行的配置，中间使用，隔开就行）
 `<set name = "setLinkMan" cascade = "save-update,delete">`
 **如果不配置 cascade = “delete”，则会先将联系人的外键置为  null，然后删除客户；**
- 第二步 在代码中直接删除客户（下面这段代码替换前面的自定义代码）
（1）根据id查询对象，调用session里面delete方法删除
```java
Customer customer = session.get(Customer.class,3);
session.delete(customer);
```

**执行过程：**
（1）根据id查询客户

![根据 id 查询客户](FrameDay04_3%20Hibernate.resource/%E6%A0%B9%E6%8D%AE%20id%20%E6%9F%A5%E8%AF%A2%E5%AE%A2%E6%88%B7.png)
（2）根据外键id值查询联系人

![根据外键id 查询联系人](FrameDay04_3%20Hibernate.resource/%E6%A0%B9%E6%8D%AE%E5%A4%96%E9%94%AEid%20%E6%9F%A5%E8%AF%A2%E8%81%94%E7%B3%BB%E4%BA%BA.png)
（3）把联系人外键设置为null

![设置联系人外键为null](FrameDay04_3%20Hibernate.resource/%E8%AE%BE%E7%BD%AE%E8%81%94%E7%B3%BB%E4%BA%BA%E5%A4%96%E9%94%AE%E4%B8%BAnull.png)
（4）删除联系人和客户

![删除联系人和客户](FrameDay04_3%20Hibernate.resource/%E5%88%A0%E9%99%A4%E8%81%94%E7%B3%BB%E4%BA%BA%E5%92%8C%E5%AE%A2%E6%88%B7.png)


### （三）一对多修改操作（inverse属性）

目标：让lucy联系人所属客户不是百度，而是阿里
```java
//1 根据id查询lucy联系人，根据id查询百度的客户
Customer baidu = session.get(Customer.class, 1);
LinkMan lucy = session.get(LinkMan.class, 2);
//2 设置持久态对象值
//把联系人放到客户里面
baidu.getSetLinkMan().add(lucy);
//把客户放到联系人里面
lucy.setCustomer(baidu);
```


-  inverse属性
  - 因为hibernate双向维护外键，在客户和联系人里面都需要维护外键，修改客户时候修改一次外键，修改联系人时候也修改一次外键，造成效率问题

![inverse 属性](FrameDay04_3%20Hibernate.resource/inverse%20%E5%B1%9E%E6%80%A7.png)
   - 解决方式：让其中的“一”方不维护外键
        - 一对多里面，让其中“一”方放弃外键维护
        - 一个国家有总统，国家有很多人，总统不能认识国家所有人，国家所有人可以认识总统
  - 具体实现：
在放弃关系维护映射文件中（这里在 Custumer.hbn.xml中），进行配置，在set标签上使用inverse属性
```Custumer_hbm_xml
<!-- 在客户映射文件中，使用 set 标签表示所有联系人
	set标签里面有name属性：属性值写在客户实体类里面表示联系人的set集合名称
	inverse属性默认值：false不放弃关系维护，true表示放弃关系维护
-->
<set name="setLinkMan" inverse="true">
```

**区分 cascade 和 inverse**
- cascade 强调是操作一个对象的时候，是否操作其关联对象；
- inverse 强调的是外键的维护权；

## 三、Hibernate多对多操作

### （一）多对多映射配置
以用户和角色为例演示

- 第一步 创建实体类，用户和角色
```Role_java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Role {
  	private Integer role_id;
  	private String role_name;
  	private String role_memo;
	/**
	 *  一个角色拥有多个用户
	 */
	private Set<User> setUser = new HashSet<User>();
}
```

```User_java
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class User {
  	private Integer user_id;
  	private String user_name;
  	private String user_password;
	/**
	 * 一个用户可以有多个角色
	 */
  	private Set<Role> setRole = new HashSet<Role>();
}
```

- 第二步 让两个实体类之间互相表示
（1）一个用户里面表示所有角色，使用set集合
`private Set<Role> setRole = new HashSet<Role>();` 
（2）一个角色有多个用户，使用set集合
`private Set<User> setUser = new HashSet<User>();` 

第三步 配置映射关系
（1）基本配置
（2）配置多对多关系
- 在用户里面表示所有角色，使用set标签
```User
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="cn.itcast.manytomany.User" table="t_user">
		<id name="user_id" column="user_id">
			<generator class="native"></generator>
		</id>
		<property name="user_name" column="user_name"></property>
		<property name="user_password" column="user_password"></property>
		<!-- 在用户里面表示所有角色，使用set标签 
			name属性：角色set集合名称
			table属性：第三张表名称
		-->
		<set name="setRole" table="user_role" cascade="save-update,delete">
			<!-- key标签里面配置
				配置当前映射文件在第三张表外键名称
			 -->
			<key column="userid"></key>
			<!-- class：角色实体类全路径
			     column：角色在第三张表外键名称
			 -->
			<many-to-many class="cn.itcast.manytomany.Role" column="roleid"></many-to-many>
		</set>
	</class>
</hibernate-mapping>
```


- 在角色里面表示所有用户，使用set标签
```Role_hbm_xml
<hibernate-mapping>
	<class name="cn.itcast.manytomany.Role" table="t_role">
		<id name="role_id" column="role_id">
			<generator class="native"></generator>
		</id>
		<property name="role_name" column="role_name"></property>
		<property name="role_memo" column="role_memo"></property>
		
		<!-- 在角色里面表示所有用户，使用set标签 -->
		<set name="setUser" table="user_role">
			<!-- 角色在第三张表外键 -->
			<key column="roleid"></key>
			<many-to-many class="cn.itcast.manytomany.User" column="userid"></many-to-many>
		</set>
	</class>
</hibernate-mapping>
```

上面配置中，两个 table 的值相同，用户配置中的 `<key column="userid">`和角色中的`<many-to-many column="roleid">`，反之角色配置中的：`<key column="roleid">`和用户中的 `<many-to-many column="userid">`

- 第四步 在核心配置文件中引入映射文件
```hibernate_cfg_xml
<mapping resource="com/gjxaiou/manytomany/User.hbm.xml"/>
<mapping resource="com/gjxaiou/manytomany/Role.hbm.xml"/>
```

测试：
![测试结果](FrameDay04_3%20Hibernate.resource/%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C.png)


### （二） 多对多级联保存
根据用户保存角色

- 第一步 在用户配置文件中 set 标签进行配置，cascade 值 save-update（User中）
`<set name="setRole" table="user_role" cascade="save-update">` 

- 第二步 写代码实现
创建用户和角色对象，把角色放到用户里面，最终保存用户就可以了
```HibernateManyToMany_java
public class HibernateManytoMany {	
	//演示多对多修级联保存
	@Test
	public void testSave() {
		SessionFactory sessionFactory = null;
		Session session = null;
		Transaction tx = null;
		try {
			//得到sessionFactory
			sessionFactory = HibernateUtils.getSessionFactory();
			//得到session
			session = sessionFactory.openSession();
			//开启事务
			tx = session.beginTransaction();
			
			//添加两个用户，为每个用户添加两个角色
			//1 创建对象
			User user1 = new User();
			user1.setUser_name("lucy");
			user1.setUser_password("123");
			
			User user2 = new User();
			user2.setUser_name("mary");
			user2.setUser_password("456");
			
			Role r1 = new Role();
			r1.setRole_name("总经理");
			r1.setRole_memo("总经理");
			
			Role r2 = new Role();
			r2.setRole_name("秘书");
			r2.setRole_memo("秘书");
			
			Role r3 = new Role();
			r3.setRole_name("保安");
			r3.setRole_memo("保安");
			
			//2 建立关系，把角色放到用户里面
			// user1 -- r1/r2
			user1.getSetRole().add(r1);
			user1.getSetRole().add(r2);
			
			// user2 -- r2/r3
			user2.getSetRole().add(r2);
			user2.getSetRole().add(r3);
			
			//3 保存用户
			session.save(user1);
			session.save(user2);
			
			//提交事务
			tx.commit();

		}catch(Exception e) {
			tx.rollback();
		}finally {
			session.close();
			//sessionFactory不需要关闭
			sessionFactory.close();
		}
	}
}
```
测试结果：
| userid | roleid|
|---|---|
|1|1|
|1|2|
|2|2|
|2|3|



### （三）多对多级联删除（了解）
第一步 在set标签进行配置，cascade值delete
`<set cascade="save-update,delete">`
第二步 删除用户
```java
User user = session.get(User.class, 1);
session.delete(user);
```

 

### （四）维护第三张表关系

- 用户和角色多对多关系，维护关系通过第三张表维护

- 让某个用户有某个角色
  - 第一步 根据id查询用户和角色
  - 第二步 把角色放到用户里面
    - 把角色对象放到用户使用set集合

- 让某个用户没有某个角色
第一步 根据id查询用户和角色
```HibernateManyToMany_java
// 让某个用户有某个角色
//让lucy有经纪人角色
//1 查询lucy和经纪人
User lucy = session.get(User.class, 1);
Role role = session.get(Role.class, 1);

//2 把角色放到用户的set集合里面
lucy.getSetRole().add(role);
```


第二步 从用户里面把角色去掉
（1）从set集合里面把角色移除
```HibernateManyToMany_java
// 让某个用户没有有某个角色
User user = session.get(User.class, 2);
Role role = session.get(Role.class, 3);

//2 从用户里面把角色去掉
user.getSetRole().remove(role);
```

