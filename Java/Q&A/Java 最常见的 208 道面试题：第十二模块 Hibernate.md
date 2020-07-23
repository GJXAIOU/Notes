---
style: summer
tags: [面试]
---
# Java 最常见的 208 道面试题：第十二模块 Hibernate

@toc

## 113 . 为什么要使用 hibernate？

*   对JDBC访问数据库的代码做了封装，大大简化了数据访问层繁琐的重复性代码。

*   Hibernate是一个基于JDBC的主流持久化框架，是一个优秀的ORM实现。他很大程度的简化DAO层的编码工作

*   hibernate使用Java反射机制，而不是字节码增强程序来实现透明性。

*   hibernate的性能非常好，因为它是个轻量级框架。映射的灵活性很出色。它支持各种关系数据库，从一对一到多对多的各种复杂关系。

## 114 . 什么是 ORM 框架？

对象-关系映射（Object-Relational Mapping，简称ORM），面向对象的开发方法是当今企业级应用开发环境中的主流开发方法，关系数据库是企业级应用环境中永久存放数据的主流数据存储系统。对象和关系数据是业务实体的两种表现形式，业务实体在内存中表现为对象，在数据库中表现为关系数据。内存中的对象之间存在关联和继承关系，而在数据库中，关系数据无法直接表达多对多关联和继承关系。因此，对象-关系映射(ORM)系统一般以中间件的形式存在，主要实现程序对象到关系数据库数据的映射。

## 115 . hibernate 中如何在控制台查看打印的 sql 语句？

参考：Java- > Blog ->  Hibernate在控制台打印sql语句以及参数

## 116 . hibernate 有几种查询方式？

- hql查询
- sql查询
- 条件查询

```
hql查询，sql查询，条件查询

HQL:  Hibernate Query Language. 面向对象的写法:
Query query = session.createQuery("from Customer where name = ?");
query.setParameter(0, "张三");
Query.list();


QBC:  Query By Criteria.(条件查询)
Criteria criteria = session.createCriteria(Customer.class);
criteria.add(Restrictions.eq("name", "花姐"));
List<Customer> list = criteria.list();

SQL:
SQLQuery query = session.createSQLQuery("select * from customer");
List<Object[]> list = query.list();

SQLQuery query = session.createSQLQuery("select * from customer");
query.addEntity(Customer.class);
List<Customer> list = query.list();

```

- Hql：具体分类
1、 属性查询 2、 参数查询、命名参数查询 3、 关联查询 4、 分页查询 5、 统计函数

- HQL和SQL的区别
HQL是面向对象查询操作的，SQL是结构化查询语言 是面向数据库表结构的

## 117 . hibernate 实体类可以被定义为 final 吗？

可以将Hibernate的实体类定义为final类，但这种做法并不好。因为Hibernate会使用代理模式在延迟关联的情况下提高性能，如果你把实体类定义成final类之后，因为 Java不允许对final类进行扩展，所以Hibernate就无法再使用代理了，如此一来就限制了使用可以提升性能的手段。不过，如果你的持久化类实现了一个接口而且在该接口中声明了所有定义于实体类中的所有public的方法轮到话，你就能够避免出现前面所说的不利后果。

## 118 . 在 hibernate 中使用 Integer 和 int 做映射有什么区别？

在Hibernate中，如果将OID定义为Integer类型，那么Hibernate就可以根据其值是否为null而判断一个对象是否是临时的，如果将OID定义为了int类型，还需要在hbm映射文件中设置其unsaved-value属性为0。

## 119 . hibernate 是如何工作的？

**hibernate工作原理**：
1.  通过Configuration config = new Configuration().configure(); //读取并解析hibernate.cfg.xml配置文件
2.  由hibernate.cfg.xml中的<mapping resource="com/xx/User.hbm.xml"/>读取并解析映射信息
3.  通过SessionFactory sf = config.buildSessionFactory();//创建SessionFactory
4.  Session session = sf.openSession();//打开Sesssion
5.  Transaction tx = session.beginTransaction();//创建并启动事务Transation
6.  persistent operate操作数据，持久化操作
7.  tx.commit();//提交事务
8.  关闭Session
9.  关闭SesstionFactory

## 120 . get()和 load()的区别？

*   load() 没有使用对象的其他属性的时候，没有SQL  延迟加载
*   get() 没有使用对象的其他属性的时候，也生成了SQL  立即加载

## 121 . 说一下 hibernate 的缓存机制？

Hibernate中的缓存分为一级缓存和二级缓存。

一级缓存就是  Session 级别的缓存，在事务范围内有效是,内置的不能被卸载。二级缓存是 SesionFactory级别的缓存，从应用启动到应用结束有效。是可选的，默认没有二级缓存，需要手动开启。保存数据库后，缓存在内存中保存一份，如果更新了数据库就要同步更新。

什么样的数据适合存放到第二级缓存中？
*   很少被修改的数据   帖子的最后回复时间
*   经常被查询的数据   电商的地点
*   不是很重要的数据，允许出现偶尔并发的数据
*   不会被并发访问的数据
*   常量数据

扩展：hibernate的二级缓存默认是不支持分布式缓存的。使用  memcahe,redis等中央缓存来代替二级缓存。

## 122 . hibernate 对象有哪些状态？

hibernate里对象有三种状态：
1.  Transient（瞬时）：对象刚new出来，还没设id，设了其他值。
2.  Persistent（持久）：调用了save()、saveOrUpdate()，就变成Persistent，有id。
3.  Detached（脱管）：当session  close()完之后，变成Detached。

![](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaIOvqXXzXPicacuC197UIKZfM3BiapgtPw2oOJz7kDWMdTKicbTCcLsunQtIDoRwWDBdZTUckm8WZ8kHg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 123 . 在 hibernate 中 getCurrentSession 和 openSession 的区别是什么？

openSession 从字面上可以看得出来，是打开一个新的session对象，而且每次使用都是打开一个新的session，假如连续使用多次，则获得的session不是同一个对象，并且使用完需要调用close方法关闭session。

getCurrentSession ，从字面上可以看得出来，是获取当前上下文一个session对象，当第一次使用此方法时，会自动产生一个session对象，并且连续使用多次时，得到的session都是同一个对象，这就是与openSession的区别之一，简单而言，getCurrentSession 就是：如果有已经使用的，用旧的，如果没有，建新的。

注意：在实际开发中，往往使用getCurrentSession多，因为一般是处理同一个事务（即是使用一个数据库的情况），所以在一般情况下比较少使用openSession或者说openSession是比较老旧的一套接口了。

## 124 . hibernate 实体类必须要有无参构造函数吗？为什么？

必须，因为hibernate框架会调用这个默认构造方法来构造实例对象，即Class类的newInstance方法，这个方法就是通过调用默认构造方法来创建实例对象的。

另外再提醒一点，如果你没有提供任何构造方法，虚拟机会自动提供默认构造方法（无参构造器），但是如果你提供了其他有参数的构造方法的话，虚拟机就不再为你提供默认构造方法，这时必须手动把无参构造器写在代码里，否则new Xxxx()是会报错的，所以默认的构造方法不是必须的，只在有多个构造方法时才是必须的，这里“必须”指的是“必须手动写出来”。


