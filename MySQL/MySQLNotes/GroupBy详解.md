# group by详解

`group by` 一般用于**分组统计**，它表达的逻辑就是`根据一定的规则，进行分组`。

## 一、使用案例

假设用一张员工表，表结构如下：

```javascript
CREATE TABLE `staff` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `id_card` varchar(20) NOT NULL COMMENT '身份证号码',
  `name` varchar(64) NOT NULL COMMENT '姓名',
  `age` int(4) NOT NULL COMMENT '年龄',
  `city` varchar(64) NOT NULL COMMENT '城市',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8 COMMENT='员工表';
```

表存量的数据如下：

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125816951.png)

我们现在有这么一个需求：**统计每个城市的员工数量**。对应的 SQL 语句就可以这么写：

```MySQL
SELECT city ,COUNT(*) AS num FROM staff GROUP BY city;
```

执行结果如下，下面将详述其执行流程：

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817013.png)

## 二、group by 原理分析

### （一）explain 分析

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125816976.png)

- Extra 这个字段的`Using temporary`表示在执行分组的时候使用了**临时表**
- Extra 这个字段的`Using filesort`表示使用了**排序**

`group by` 怎么就使用到`临时表和排序`了呢？我们来看下这个SQL的执行流程

### （二）group by 的简单执行流程

该 SQL的执行流程大致如下：

- 创建内存临时表，表里有两个字段 `city` 和 `num`；

- 全表扫描 `staff` 的记录，依次取出 `city = 'X'` 的记录。

    - 判断**临时表**中是否有为 `city='X'` 的行，没有就插入一个记录` (X,1)`;

    - 如果临时表中有 `city='X'` 的行，就将 x 这一行的 num 值加 1；

- 遍历完成后，再根据字段 `city` 做**排序**，得到结果集返回给客户端。

这个流程的执行图如下：

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620.png)

临时表的排序就是把需要排序的字段，放到 sort buffer，排完就返回。排序分**全字段排序**和**rowid排序**

- 如果是`全字段排序`，需要查询返回的字段，都放入`sort buffer`，根据**排序字段**排完，直接返回
- 如果是`rowid排序`，只是需要排序的字段放入`sort buffer`，然后多一次**回表**操作，再返回。

怎么确定走的是全字段排序还是rowid 排序排序呢？由一个[数据库](https://cloud.tencent.com/solution/database?from=10680)参数控制的，`max_length_for_sort_data`，详见 [**看一遍就理解：order by详解**](https://mp.weixin.qq.com/s?__biz=Mzg3NzU5NTIwNg==&mid=2247490571&idx=1&sn=e8638573ec8d720fd25da5b2b0d90ed2&chksm=cf21c322f8564a34461acd9811730d14d12075cf5c7438a3a11433725b9ce463fcb78e7916a1&token=574771970&lang=zh_CN&scene=21#wechat_redirect)

## 三、where 和 having 的区别

- group by + where 的执行流程
- group by + having 的执行流程
- 同时有where、group by 、having的执行顺序

### （一）group by + where 的执行流程

如果 SQL 加了**where 条件**之后，并且 where 条件列加了索引，执行流程详解如下：

```mysql
-- 添加 WHERE 条件
SELECT city ,COUNT(*) AS num FROM staff WHERE age> 30 GROUP BY city;
-- 添加索引
alter table staff add index idx_age (age);
```

expain 分析结果：

```mysql
explain select city ,count(*) as num from staff where age> 30 group by city;
```

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125816958.png)

从 explain 执行计划结果，可以发现查询条件命中了`idx_age`的索引，并且使用了`临时表和排序`

> **Using index condition**:表示索引下推优化，根据索引尽可能的过滤数据,然后再返回给服务器层根据where其他条件进行过滤。这里单个索引为什么会出现索引下推呢？explain出现并不代表一定是使用了索引下推，只是代表可以使用，但是不一定用了。

执行流程如下：

- 创建内存临时表，表里有两个字段 `city` 和 `num`；

- 扫描索引树 `idx_age`，找到大于年龄大于 30 的主键 ID

- 通过主键ID，回表找到 `city = 'X'`

    - 判断**临时表**中是否有为 `city='X'` 的行，没有就插入一个记录 (X,1);

    - 如果临时表中有 `city='X'` 的行，就将 x 这一行的 num 值加 1；

- 继续重复 2,3 步骤，找到所有满足条件的数据，

- 最后根据字段 `city` 做**排序**，得到结果集返回给客户端。

### （二）group by + having 的执行

如果你要查询每个城市的员工数量，获取到员工数量不低于 3 的城市，having 可以很好解决你的问题，SQL 如下：

```mysql
select city ,count(*) as num from staff  group by city having num >= 3;
```

查询结果如下：

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817041.png)

`having` 称为分组过滤条件，它对返回的结果集操作。

### （三）同时有where、group by 、having 的执行顺序

```mysql
select city ,count(*) as num from staff  where age> 19 group by city having num >= 3;
```

上述 SQL 同时含有 `where、group by、having`子句，执行顺序为：

- 执行 `where` 子句查找符合年龄大于 19 的员工数据

- `group by` 子句对员工数据，根据城市分组。

- 对 `group by` 子句形成的城市组，运行聚集函数计算每一组的员工数量值；

- 最后用 `having` 子句选出员工数量大于等于3的城市组。

### （四）where + having 区别总结

- `having`子句用于**分组后筛选**，where 子句用于**行**条件筛选
- `having`一般都是配合`group by` 和聚合函数一起出现如(`count()`,`sum()`,`avg()`,`max()`,`min()`)
- `where` 条件子句中不能使用聚集函数，而 `having` 子句就可以。
- `having`只能用在 group by 之后，where 执行在 group by之 前

## 四、使用 group by 注意的问题

使用 group by 主要有这几点需要注意：

- `group by`一定要配合聚合函数一起使用嘛？
- `group by`的字段一定要出现在select中嘛
- `group by`导致的慢SQL问题

###  （一）group by 一定要配合聚合函数使用嘛？

group by 就是**分组统计**的意思，一般情况都是配合聚合函数如 `count()`,`sum(),avg(),max(),min())`一起使用。如果没有配合聚合函数使用，在 MySQL 5.7 版本可以使用，不会报错，但是返回的是分组的第一行数据。

例如下列 SQL 结果为：

```mysql
select city,id_card,age from staff group by  city;
```

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125816989.png)

通过对比可知，返回的就是每个分组的第一条数据

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817380.png)

因此 group by 还是配合聚合函数使用的，除非一些特殊场景，比如你想**去重**，当然去重用`distinct`也是可以的。

### （二）group by 后面跟的字段是否一定要出现在 select 中。

不一定，比如以下SQL：

```mysql
select max(age)  from staff group by city;
```

执行结果如下：

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817312.png)

分组字段 `city` 不在 select  后面，并不会报错。当然，这个可能跟**不同的数据库，不同的版本**有关吧。

## （三）group by 导致的慢 SQL 问题

group by 使用不当，很容易就会产生慢SQL 问题。因为它既用到**临时表**，又默认用到**排序**。有时候还可能用到**磁盘临时表**。

- 如果执行过程中，会发现内存临时表大小到达了**上限**（控制这个上限的参数就是 `tmp_table_size`），会把**内存临时表转成磁盘临时表**。
- 如果数据量很大，很可能这个查询需要的磁盘临时表，就会占用大量的磁盘空间。

## 五、group by 的一些优化方案

- 优化方向1：既然默认会排序，不给它排是不是就行。
- 优化方向2：既然临时表是影响 group by 性能的因素，是不是可以不用临时表

执行 `group by` 语句为什么需要临时表呢？`group by` 的语义逻辑，就是统计不同的值出现的个数。如果这个**这些值一开始就是有序的**，我们是不是直接往下扫描统计就好了，就不用**临时表来记录并统计结果**啦?

- group by 后面的字段加索引
- order by null 不用排序
- 尽量只使用内存临时表
- 使用 SQL_BIG_RESULT

### （一）group by 后面的字段加索引

如何保证`group by`后面的字段数值一开始就是有序的呢？当然就是**加索引**啦。

我们回到一下这个SQL

```mysql
select city ,count(*) as num from staff where age= 19 group by city;
```

它的执行计划

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817004.png)

如果我们给它加个联合索引`idx_age_city（age,city）`

```mysql
alter table staff add index idx_age_city(age,city);
```

再去看执行计划，发现既不用排序，也不需要临时表啦。

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125816911.png)

**加合适的索引**是优化`group by`最简单有效的优化方式。

### （二） order by null 不用排序

并不是所有场景都适合加索引的，如果碰上不适合创建索引的场景，我们如何优化呢？

> 如果你的需求并不需要对结果集进行排序，可以使用 `order by null`。

```mysql
select city ,count(*) as num from staff group by city order by null
```

执行计划如下，已经没有`filesort`啦

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817739.png)

### （三）尽量只使用内存临时表

如果`group by`需要统计的数据不多，我们可以尽量只使用**内存临时表**；因为如果 group by 的过程因为内存临时表放不下数据，从而用到磁盘临时表的话，是比较耗时的。因此可以适当调大`tmp_table_size`参数，来避免用到**磁盘临时表**。

### **5.4 使用SQL_BIG_RESULT优化**

如果数据量实在太大怎么办呢？总不能无限调大`tmp_table_size`吧？但也不能眼睁睁看着数据先放到内存临时表，**随着数据插入**发现到达上限，再转成磁盘临时表吧？这样就有点不智能啦。

因此，如果预估数据量比较大，我们使用`SQL_BIG_RESULT` 这个提示直接用磁盘临时表。MySQl优化器发现，磁盘临时表是B+树存储，存储效率不如数组来得高。因此会直接用数组来存

示例SQl如下：

```javascript
select SQL_BIG_RESULT city ,count(*) as num from staff group by city;
```

执行计划的`Extra`字段可以看到，执行没有再使用临时表，而是只有排序

![img](GroupBy%E8%AF%A6%E8%A7%A3.resource/1620-20220407125817618.png)

执行流程如下：

1. 初始化 sort_buffer，放入city字段；
2. 扫描表staff，依次取出city的值,存入 sort_buffer 中；
3. 扫描完成后，对 sort_buffer的city字段做排序
4. 排序完成后，就得到了一个有序数组。
5. 根据有序数组，统计每个值出现的次数。

## **6. 一个生产慢SQL如何优化** 

最近遇到个生产慢SQL，跟group by相关的，给大家看下怎么优化哈。

表结构如下：

```javascript
CREATE TABLE `staff` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `id_card` varchar(20) NOT NULL COMMENT '身份证号码',
  `name` varchar(64) NOT NULL COMMENT '姓名',
  `status` varchar(64) NOT NULL COMMENT 'Y-已激活 I-初始化 D-已删除 R-审核中',
  `age` int(4) NOT NULL COMMENT '年龄',
  `city` varchar(64) NOT NULL COMMENT '城市',
  `enterprise_no` varchar(64) NOT NULL COMMENT '企业号',
  `legal_cert_no` varchar(64) NOT NULL COMMENT '法人号码',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8 COMMENT='员工表';
```

查询的SQL是这样的：

```javascript
select * from t1 where status = #{status} group by #{legal_cert_no}
```

我们先不去探讨这个SQL的=是否合理。如果就是这么个SQL，你会怎么优化呢？有想法的小伙伴可以留言讨论哈，也可以加我微信加群探讨。如果你觉得文章那里写得不对，也可以提出来哈，一起进步，加油呀

