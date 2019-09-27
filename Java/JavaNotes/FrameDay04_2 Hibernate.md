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

- 实体类里面属性私有的
- 私有属性使用公开的set和get方法进行操作
- 要求实体类有属性作为唯一值（一般使用id值）
- 实体类属性建议不使用基本数据类型，使用基本数据类型对应的包装类
  为什么使用包装类：
  比如 表示学生的分数，假如 int score;
  - 比如学生得了0分 ，int score = 0;
  - 如果表示学生没有参加考试，int score = 0;不能准确表示学生是否参加考试
  解决：使用包装类可以了， Integer score = 0，表示学生得了0分，
  表示学生没有参加考试，Integer score = null;

## 二、Hibernate 主键生成策略

- hibernate要求实体类里面有一个属性作为唯一值，其对应表主键，主键可以不同生成策略；
-  hibernate主键生成策略有很多的值；示例为：`<generator class = "native"></generator>`
 -  在class属性里面有很多值：常用的是 native 和 uuid 两个；
  - native：根据使用的数据库帮助选择哪个值（因为有些值不是所有数据库都支持）；
  - uuid：之前 web 阶段写代码生成 uuid 值，hibernate 会帮我们生成 uuid 值；

| 值 | 描述
|---|---
|native | 根据底层数据库对自动生成表示符的能力来选择过 identity 、sequence、hilo三种生成器中的一种，适合跨数据库平台开发。适用于代理主键。 
|uuid | Hibernate 采用 128 位的 UUID 算法来生成标识符。 该算法能够在网络环境中生成唯一的字符串标识符，其 UUID 被编码为一个长度为32 位的十六进制字符串。 这种策略并不流行， 因为字符串类型的主键比整数类型的主键占用更多的数据库空间。 适用于代理主键。   

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
user.setUsername("张三");
user.setPassword("123");
user.setAddress("南京");
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
结果图：
![添加操作]($resource/%E6%B7%BB%E5%8A%A0%E6%93%8D%E4%BD%9C.png)

### （二）修改操作

- 首先查询，然后再修改值
（1）根据id查询，返回对象
```java
// 目的：修改 uid=2 记录的 username 值
  // 首先更加 id 查询
User user = session.get(User.class, 2);
  // 然后向返回的 user 对象里面设置修改只有的值
user.setUsername("李四");
  // 最后调用 session 的方法 update 修改：执行过程：在 user 对象里面找到 uid 值，更加 uid 进行修改
session.update(user);
```
 结果图：
![修改操作]($resource/%E4%BF%AE%E6%94%B9%E6%93%8D%E4%BD%9C.png)
 

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
 

## 四、实体类对象状态（概念）

实体类状态有三种：瞬时态、持久态、托管态；
- 瞬时态：对象里面没有 id 值，对象与 session 没有关联
```java
User user = new User();
user.setUsername("zhangsan");
user.setPassword("123");
user.setAddress("jiangsu");
```

- 持久态：对象里面有 id 值，对象与 session 关联
`User user = session.get(User.class, 1);` 

- 托管态：对象有 id 值，对象与 session 没有关联
```java
User user = new User();
user.setUid(3);
```
 

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

### （二）Hibernate缓存
hibernate 框架中提供很多优化方式，hibernate 的缓存就是一个优化方式；

- hibernate缓存特点：
第一类 hibernate的一级缓存
（1）hibernate的一级缓存默认打开的
（2）hibernate的一级缓存使用范围，是session范围，从session创建到session关闭范围
（3）hibernate的一级缓存中，存储数据必须 持久态数据

第二类 hibernate的二级缓存
（1）目前已经不使用了，替代技术 redis
（2）二级缓存默认不是打开的，需要配置
（3）二级缓存使用范围，是sessionFactory范围

### （三）验证一级缓存存在
- 验证方式
（1）首先根据uid=6 查询，返回对象
（2）其次再根据uid=6 查询，返回对象，看两者返回对象是否相同；
```java
// 执行第一个 get 方法是否查询数据库，是否发送 SQL 语句
User user1 = session.get(User.class, 6);
System.out.println(user1);

// 执行第二个 get 方法是否查询数据库，是否发送 SQL 语句
User user2 = session.get(User.class, 6);
System.out.println(user2);
```
![一级缓存]($resource/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98.png)
从执行结果中可以看出：
第一步执行get方法之后，发送sql语句查询数据库
第二个执行get方法之后，没有发送sql语句，查询一级缓存内容

### （四）Hibernate一级缓存执行过程
 
![一级缓存执行过程]($resource/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

### （五）Hibernate一级缓存特性

持久态会自动更新数据库，即就相等于不执行 update也可以，但是会到来一系列的问题；
```java
User user = session.get(User.class, 7);
user.setUsername("hanmeimei");
// session.update(user);
```

-  执行过程（了解）
 
![一级缓存具体执行过程]($resource/%E4%B8%80%E7%BA%A7%E7%BC%93%E5%AD%98%E5%85%B7%E4%BD%93%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

## 六、Hibernate事务操作

### （一）事务相关概念

- 不考虑隔离性产生问题
（1）脏读
（2）不可重复读
（3）虚读

- 设置事务隔离级别
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
@Test
	public void testTx() {
		SessionFactory sessionFactory = null;
		Session session = null;
		Transaction tx = null;
		try {
			sessionFactory = HibernateUtils.getSessionFactory();
			session = sessionFactory.openSession();
			//开启事务
			tx = session.beginTransaction();
			
			//添加
			User user = new User();
			user.setUsername("小马");
			user.setPassword("250");
			user.setAddress("美国");
			
			session.save(user);
			
			int i = 10/0;
			//提交事务
			tx.commit();
		}catch(Exception e) {
			e.printStackTrace();
			//回滚事务
			tx.rollback();
		}finally {
			//关闭操作
			session.close();
			sessionFactory.close();
		}
	}
```

### （三）Hibernate绑定session
用于保证单线程

- session类似于jdbc的connection，之前web阶段学过 ThreadLocal

- Hibernate 可以实现与本地线程绑定session

- 获取与本地线程session 步骤：
  - 在hibernate核心配置文件中配置：`<property name="hibernate.current_session_context_class">thread</property>`
  - 调用sessionFactory里面的方法得到
```java
// 提供返回与本地线程绑定的 session 方法
public static Session getSessionObject(){
    return sessionFactory.getCurrentSession();
}
```

   - 然后实现session时候使用上面方法构建即可；
- 获取与本地线程绑定session时候，如果关闭session会报错，不需要手动关闭了
 

## 七、Hibernate 的 api 使用（下面三个都是查询的）

### （一）Query对象

- 使用query对象，不需要写sql语句，但是写hql语句：hql：hibernate query language，hibernate提供查询语言，这个hql语句和普通sql语句很相似
- hql和sql语句区别：
  - 使用sql操作表和表字段
  - 使用hql操作实体类和属性

- 查询所有hql语句：`from 实体类名称`

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

 

### Criteria对象
1 使用这个对象查询操作，但是使用这个对象时候，不需要写语句，直接调用方法实现

2 实现过程
（1）创建criteria对象
（2）调用对象里面的方法得到结果
 

### SQLQuery对象
1 使用hibernate时候，调用底层sql实现

2 实现过程
（1）创建对象
（2）调用对象的方法得到结果
 
返回list集合每部分是数组
 

返回list中每部分是对象形式
 
 

## 完成任务
1 查询表所有记录，把记录显示页面中
（1）servlet里面调用service，service调用dao
（2）在dao里面使用hibernate实现操作
（3）在页面中显示所有数据
- 在servlet里面把list集合放到域对象
- 在jsp中使用el表达式+foreach标签获取

2 复习知识
（1）一对多和建表
（2）多对多和建表



