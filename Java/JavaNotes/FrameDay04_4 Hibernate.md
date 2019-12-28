# FrameDay04_4 Hibernate

**上节内容**
-  表与表之间关系回顾
  - 一对多（客户和联系人）
  - 多对多（用户和角色）

- Hibernate一对多操作
  - 一对多映射配置
  - 一对多级联保存
  - 一对多级联删除
  - inverse属性

- Hibernate多对多操作
  - 多对多映射配置
  - 多对多级联保存（重点）
  - 多对多级联删除（了解）
  - 维护第三张表

**今天内容**

**Hibernate 的查询方式**

 - 对象导航查询

- hql查询
（1）查询所有
（2）条件查询
（3）排序查询
（4）分页查询
（5）投影查询
（6）聚集函数使用

- qbc查询
（1）查询所有
（2）条件查询
（3）排序查询
（4）分页查询
（5）统计查询
（6）离线查询

- hql多表查询
（1）mysql多表查询回顾
（2）hql多表查询
内连接、迫切内连接、左外连接、迫切左外连接、右外连接

 Hibernate的检索策略
（1）概念
- Hibernate分成 ：立即和延迟查询
- 延迟查询分成：类级别和关联级别延迟
（2）具体操作



## 一、Hibernate 查询方式

- 对象导航查询
示例：根据 id 查询某个客户，再查询这个客户里面所有的联系人；

- OID 查询
示例：根据 id 查询某一条记录，返回对象；

- HQL 查询
使用 Query 对象，写 hql 语句实现查询；

- QBC 查询
使用 Criteria 对象，不需要写 SQL 语句，操作类和属性；

- 本地 sql 查询
使用 SQLQuery 对象，使用普通 sql 实现查询；

### （一）对象导航查询
对象图导航检索方式是根据已经加载的对象，导航到它的关联对象，它利用类与类之间的关系来检索对象；
查询某个客户里面所有联系人过程，就是使用对象导航实现；（即根据联系人对象自动导航找到联系人所属的客户对象）。
```java
// 首先查询 cid = 1 的客户，然后查询该客户所有联系人
Customer customer = session.get(Customer.class, 1);
// 查询该客户所有联系人，直接调用 Customer 类中 LinkMan 属性的 get方法即可；
Set<LinkMan> linkman = customer.getSetLinkMan();
System.out.println(linkman.size());
```


### （二）OID查询
指利用 Session 的 get() 和 load() 方法加载某条记录对应的对象；
根据 id 查询符合的记录，就是调用 session 里面的 get 方法实现；
`Customer customer = session.get(Customer.class, 2);`


### （三）HQL 查询【推荐、用的最多】

- hql：Hibernate query language，是 Hibernate 提供一种面向对象的查询语言，hql 语言和普通 sql 很相似，区别：普通 sql 操作数据库表和字段，hql 操作实体类和属性；
- 功能：
  - 在查询语句中设定各种查询条件。
  - 支待投影查询， 即仅检索出对象的部分属性。
  - 支待分页查询。
  - 支待分组查询， 允许使用 group by和having关键字。
  - 提供内置聚集函数， 如s um()、 min()和 max()。
  - 能够调用用户定义的SQL函数。
  - 支持子查询， 即嵌套查询。
  - 支待动态绑定参数。

- 常用的hql语句
  - 查询所有： `from 实体类名称`
  - 条件查询： `from 实体类名称 where 属性名称=?`
  - 排序查询： `from 实体类名称 order by 实体类属性名称 asc/desc`

- **使用 hql 查询操作时候，使用 Query 对象**
  - 步骤一：创建 Query 对象，写 hql 语句
  - 步骤二：调用 query 对象里面的方法得到结果

#### 1.  查询所有
查询所有的时候通常省略 `select`关键字；
示例：查询所有客户记录
```java
Query query = session.createQuery("from Customer");
List<Customer> list = query.list();
```

#### 2. 条件查询

- hql条件查询语句写法：
一般查询： `from  实体类名称 where 实体类属性名称=? and实体类属性名称=?`
模糊查询： `from  实体类名称 where 实体类属性名称 like ?`

一般查询：
```java
// 下面 hql 对应于 SQL：select * from Customer c where c.cid = ? and c.custName = ?
Query query = session.createQuery("from Customer c where c.cid = ? and c.custName = ?");
// 参数一分别为 ？处值，从 0 开始；参数二为具体赋予的参数值
query.setParameter(0, 1);
query.setParameter(1,"百度");
// 调用方法获得结果
List<Customer> list = query.list();
```

模糊查询：
```java
Query query = session.createQuery("from Customer c where c.custName like ?");
query.setParameter(0, "%浪%");
List<Customer> list = query.list();
```

#### 3. 排序查询
-  hql排序语句写法
`from 实体类名称 order by 实体类属性名称 asc/desc`

```java
Query query = session.createQuery("from Customer order by cid desc");
List<Customer> list = query.list();
```

#### 4. 分页查询
- mysql 实现分页
使用关键字 limit 实现：`select * from t_customer limit 0,3;`

- 在 hql 中实现分页
在 hql 操作中，在语句里面不能写 limit，Hibernate 的 Query 对象封装两个方法实现分页操作

```java
Query query = session.createQuery("from Customer");
// 设置分页，分别设置开始位置和每页记录数
query.setFirstResult(0);
query.setMaxResults(3);
List<Customer> list = query.list();
```


#### 5.  投影查询
投影查询：查询不是所有字段值，而是部分字段的值

- 投影查询hql语句写法：
`select 实体类属性名称1, 实体类属性名称2...  from 实体类名称`
  - select 后面不能写 * ，不支持的

**注意这里的返回值可以是列值的类型，也可以是 Object；**
```java
Query query = session.createQuery("select custName from Customer");
List<Object> list = query.list();
for(Object object : list){
    System.out.println(object);
}
```


#### 6. 聚集函数使用

- 常用的聚集函数
count、sum、avg、max、min

- hql 聚集函数语句写法
示例：查询表记录数：`select count(*) from 实体类名称`
```java
Query query = session.createQuery("select count(*) from Customer");
// 使用 query 中的方法，直接返回对象形式，这里的 obj，本质上的 long 类型；
Object object = query.uniqueResult();
// 需要先转换为 Long，然后再转换为 int，否则会报错；
Long longObject = (Long)object;
int count = longObject.intValue();
```



### （四）QBC查询
QBC（Query BY Criteria）查询主要由 Criteria 接口、Criterion 接口以及 Expression 类组成；其中 Criteria 接口是一个查询接口，由 session 创建，Criterion 是 Criteria 的查询条件，在 Criteria 总提供了 add（Criterion criterion）方法来添加查询条件。
- 使用 hql 查询需要写 hql 语句实现，但是使用 qbc 时候，不需要写语句了，使用方法实现 p；
- 使用 qbc 时候，操作实体类和属性；
- 使用qbc，使用 Criteria 对象实现；

#### 1. 查询所有
- 创建Criteria对象
- 调用方法得到结果
```java
Criteria criteria = session.createCriteria(Customer.class);
List<Customer> list = criteria.list();
```

#### 2. 条件查询
-  没有语句，使用封装的方法实现
```java
Criteria criteria = session.createCriteria(Customer.class);
// 使用 Criteria 对象中的方法设置条件值，首先使用 add 方法表示设置条件值，然后在 add 方法中使用类的方法实现条件设置，类似于： cid = ?
criteria.add(Restrictions.eq("cid",1));
criteria.add(Restrictions.eq("custName", "百度"));
/**
* 上面方式可以分开书写
* // 设定查询条件
* Criterion criterion = Restrictions.eq("cid",1);
* // 添加查询条件
* criteria.add(criterion); 
*/
List<Customer> list = criteria.list(); 
```
对应的如果使用模糊查询：`criteria.add(Restrictions.eq("custName", "%百%"));`

Restrictions 类提供的方法

方法名 | 说明
---|---
Restrictions.eq | 等于
Restrictions.allEq | 使用 Map, 使用 key/value 进行多个等于的比较
Restrictions.gt | 大于＞
Restrictions.ge | 大于等于＞＝
Restrictions.It | 小于
Restrictions.le | 小于等于＜＝
Restrictions.between | 对应 SQL 的 between 子句
Restrictions.like | 对应 SQL 的 like 子句
Restrictions.in | 对应 SQL 的 IN 子句
Restrictions.and | and 关系
Restrictions.or | or 关系
Restrictions.sqlRestnction | SQL 限定查询

#### 3. 排序查询
 分为 asc 和 desc 两种排序方式
```java
Criteria criteria = session.createCriteria(Customer.class);
// 设定对哪个属性进行排序，同时设定排序规则
criteria.addOrder(Order.asc("cid"));
criteria.addOrder(Order.desc("cid"));
```

#### 4. 分页查询
```java
Criteria criteria = session.createCriteria(Customer.class);
// 设置分页数据，包括设置开始位置和每页显示记录数
criteria.setFirstRestlt(0);
criteria.setMaxResults(3);
```
开始位置计算公式： （当前页-1）*每页记录数

#### 5. 统计查询
 这里以计算表中行数为例：
```java
Criteria criteria = session.createCriteria(Customer.class);
criteria.setProjection(Projections.rowCount());
Object obj = criteria.uniqueResult();
Long longObj = (Long)obj;
int cout = longObj.intValue();
```

#### 6. 离线查询
离线条件查询：可以脱离 Session 来使用的一种条件查询对象，默认先有 Session 后生成 Criteria 对象，然而 DetachedCriteria 对象可以在其它层对条件进行封装；

- servlet 调用 service，service 调用 dao
  - 在 dao 里面对数据库 crud 操作
  - 在 dao 里面使用 Hibernate 框架，使用 Hibernate 框架时候，调用 session 里面的方法实现功能；
```java
// 首先创建对象
DetachedCriteria detachedCriteria = DetachedCriteria.forClass(Customer.class);
// 最终执行的时候需要 session
Criteria criteria detachedCriteria.getExecutableCriteria(session);
List<Customer> list = criteria.list();
```


### （五）HQL 多表查询

**MySQL 中的多表查询分类与语句**：

- 内连接（查的是两个表关联的数据，没有关联的不会）
 `select * from t_customer c, t_linkman l where c.cid = l.cid`；或者使用 inner join 表示为：`select * from t_customer c inner join t_linkman l on c.cid = l.cid;`

-  左外连接（连接左边的表都会查询出来，右边如果没有关联查询不出）
 `select * from t_customer c left outer join t_linkman l on c.cid = l.cid;`

- 右外连接
 `select * from t_customer c right outer join t_linkman l on c.cid = l.cid;`

**HQL实现多表查询**

* 内连接
* 左外连接
* 右外连接
* 迫切内连接
* 迫切左外连接

#### 1. HQL内连接

- 内连接查询hql语句写法：以客户和联系人为例
`from  Customer  c  inner  join  c.setLinkMan`

```java
Query query = session.createQuery("from Customer c inner join c.setLinkMan");
List list = query.list();
```
返回值为 list，list 里面每部分是数组形式，类型为 Object；

- 演示迫切内连接
  - 迫切内连接和内连接底层实现一样的
  - 区别：使用内连接返回list中每部分是数组，迫切内连接返回list每部分是对象
  - hql语句写法： `from  Customer  c  inner  join  fetch  c.setLinkMan`，其他与上面相同；


#### 2. HQL左外连接

- 左外连接hql语句：`from  Customer  c  left  outer  join  c.setLinkMan`
- 迫切左外连接：`from  Customer  c  left  outer  join  fetch  c.setLinkMan`
- 左外连接返回 list 中每部分是数组，迫切左外连接返回 list 每部分是对象
- 右外连接 hql 语句：`from  Customer  c  right  outer  join  c.setLinkMan`


## 二、Hibernate检索策略

### （一）检索策略的概念

Hibernate检索策略分为两类：
- 立即查询：根据id查询，调用get方法，一调用get方法马上发送语句查询数据库
```java
// 调用 get 方法马上会发送 SQL 语句查询数据库
Customer customer = session.get(Customer.class, 1);
```

- 延迟查询：根据id查询，还有load方法，调用load方法不会马上发送语句查询数据，只有得到对象里面的值时候才会发送语句查询数据库
```java
Customer customer = session.load(Customer.class, 2);
// 这里不会发送 SQL 语句给数据库
System.out.println(customer.getCid());
// 当查询 custName 的时候才会将语句发送给数据库
System.out.println(customer.getCustName());
```

- 延迟查询分成两类：
  - 类级别延迟：根据id查询返回实体类对象，调用load方法不会马上发送语句
  - 关联级别延迟：
查询某个客户，再查询这个客户的所有联系人，查询客户的所有联系人的过程是否需要延迟，这个过程称为关联级别延迟

```java
Customer customer = session.get(Customer.class, 1);
// 这里通过查询到的客户得到所有联系人的 set 集合并没有发送 SQL 语句；
Set<LinkMan> linkman = customer.getSetLinkMan();
// 发送 SQL 语句
System.out.println(linkman.size());
```


### （二）关联级别延迟操作

在映射文件中进行配置实现
（1）根据客户得到所有的联系人，在客户映射文件中配置

- 在set标签上使用属性
  - fetch：值select（默认）
  - lazy：值
    - true：延迟（默认）
    - false：不延迟
    - extra：极其延迟
   `<fetch="select" lazy = "true">`
   `<fetch="select" lazy = "false">`

 - 当 lazy 值为 true 之后， 调用get之后，发送两条sql语句

![lazy 值为 true](FrameDay04_4%20Hibernate.resource/lazy%20%E5%80%BC%E4%B8%BA%20true.png)

- 当值为 extra 时候， 极其懒惰，要什么值给什么值，最终执行的 SQL 语句为：
```sql
select count(lkm_id) from t_linkman where clid = ?
```

 

### （三）批量抓取

查询所有的客户，返回list集合，遍历list集合，得到每个客户，得到每个客户的所有联系人
（1）上面操作代码，发送多条sql语句
```java
Criteria criteria = session.createCriteria(Customer.class);
List<Customer> list = criteria.list();
// 得到每个客户里面所有联系人
for(Customer customer:list){
  System.out.println(customer.getCid() + ":" + customer.getCustName());
  // 每个客户里面所有联系人
  Set<LinkMan> setLinkMan = customer.getSetLinkMan();
  for(LinkMan linkMan : setLinkMan){
      System.out.println(linkMan.getLkm_id() + " : " + linkMan.getLkm_name());
  }
}
```


2 在客户的映射文件中，set 标签配置
（1）batch-size值，值越大发送语句越少
 `<set name = "setLinkMan" batch-size = "10">`



