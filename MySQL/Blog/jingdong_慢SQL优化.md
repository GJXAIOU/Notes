# 一文带你搞懂如何优化慢SQL

> 原文：https://www.toutiao.com/article/7215757910859612724/

## 一、前言

最近通过SGM监控发现有两个SQL的执行时间占该任务总执行时间的90%，通过对该 SQL 进行分析和优化的过程中，又重新对 SQL 语句的执行顺序和 SQL 语句的执行计划进行了系统性的学习，整理的相关学习和总结如下；

## 二、SQL语句执行顺序

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/sql执行.png)



要想优化慢 SQL 语句首先需要了解 SQL 语句的执行顺序，SQL语 句中的各关键词执行顺序如下：

- 首先执行 `from`、`join` 来确定表之间的连接关系，得到初步的数据。

- 然后利用 `where` 关键字后面的条件对符合条件的语句进行筛选。

**from&join&where**：用于确定要查询的表的范围，涉及到哪些表。

选择**一**张表，然后用**join**连接：

```
from table1 join table2 on table1.id=table2.id
```

选择**多**张表，用**where**做关联条件：

```
from table1,table2 where table1.id=table2.id
```

最终会得到满足关联条件的两张表的数据，不加关联条件会出现笛卡尔积。

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/笛卡尔积.png)

- 然后利用 group by 对数据进行分组。

按照SQL语句中的分组条件对数据进行分组，但是不会筛选数据。

下面用按照 id 的**奇偶**进行分组：

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/分组.png)

- 然后分组后的数据分别执行**having**中的普通筛选或者聚合函数筛选。

**having&where**

`having` 中可以是普通条件的筛选，也能是聚合函数，而 `where` 中只能是普通函数；一般情况下，有 `having` 可以不写 `where`，把 `where` 的筛选放在 `having` 里，SQL语句看上去更丝滑。

使用**where**再**group by** ： 先把不满足 where 条件的数据删除，再去分组。

使用**group by** 在**having**：先分组再删除不满足 `having` 条件的数据。（该两种几乎没有区别）

比如举例如下：100/2=50，此时我们把100拆分(10+10+10+10+10…)/2=5+5+5+…+5=50,只要筛选条件没变，即便是分组了也得满足筛选条件，所以**where**后**group by** 和**group by**再**having**是不影响结果的！

不同的是，**having**语法支持聚合函数,其实**having**的意思就是针对每组的条件进行筛选。我们之前看到了普通的筛选条件是不影响的，但是**having**还支持聚合函数，这是**where**无法实现的。

当前的数据分组情况

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/当前分组.png)﻿﻿

执行**having**的筛选条件，可以使用聚合函数。筛选掉工资小于各组平均工资的 `having salary>avg(salary)`：

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/聚合.png)

然后再根据我们要的数据进行 `select`，普通字段查询或者聚合函数查询，如果是聚合函数，**select**的查询结果会增加一条字段。

分组结束之后，我们再执行**select**语句，因为聚合函数是依赖于分组的，聚合函数会单独新增一个查询出来的字段，这里我们两个 id 重复了，我们就保留一个id，重复字段名需要指向来自哪张表，否则会出现唯一性问题。最后按照用户名去重。

```
select employee.id,distinct name,salary, avg(salary)
```



![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/having.png)

将各组**having**之后的数据再合并数据。

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/合并.jpeg)

- 然后将查询到的数据结果利用 `distinct` 关键字去重。

- 然后合并各个分组的查询结果，按照 `order by` 的条件进行排序。

比如这里按照 id 排序。如果此时有 `limit` 那么查询到相应的我们需要的记录数时，就不继续往下查了。

- 最后使用 `limit` 做分页。

记住 `limit` 是最后查询的，为什么呢？假如我们要查询薪资最低的三个数据，如果在排序之前就截取到 3 个数据。实际上查询出来的不是最低的三个数据而是前三个数据了，记住这一点。

假如 SQL 语句执行顺序是先做 `limit` 再执行 `order by`，执行结果为3500,5500,7000了（正确SQL执行的最低工资的是3500,5500,5500）。

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/分页.png)

**SQL查询时需要遵循的两个顺序：**

1、关键字的顺序是不能颠倒的。

```
SELECT ... FROM ... WHERE ... GROUP BY ... HAVING ... ORDER BY ... LIMIT
```

2、select语句的执行顺序（在MySQL和Oracle中，select执行顺序基本相同）。

```
FROM > WHERE > GROUP BY > HAVING > SELECT的字段 > DISTINCT > ORDER BY > LIMIT
```

以 SQL 语句举例，那么该语句的关键字顺序和执行顺序如下：

```
SELECT DISTINCT player_id, player_name, count(*) as num #顺序5
FROM player JOIN team ON player.team_id = team.team_id #顺序1
WHERE height > 1.80 #顺序2
GROUP BY player.team_id #顺序3
HAVING num > 2 #顺序4
ORDER BY num DESC #顺序6
LIMIT 2 #顺序7
```

## 三、SQL执行计划

可以参考：https://juejin.cn/post/6844904149864169486

### 为什么要学习 SQL 的执行计划？

因为一个 sql 的执行计划可以告诉我们很多关于如何优化 sql 的信息 。通过一个 sql 计划，如何访问表中的数据 （是使用全表扫描还是索引查找？）一个表中可能存在多个不同的索引，表中的类型是什么、是否子查询、关联查询等…

### 如何获取 SQL 的执行计划？

在 SQL 语句前加上 `explain` 关键词皆可以得到相应的执行计划。其中：在 MySQL8.0 中是支持对 select/delete/insert/replace/update 语句来分析执行计划，而 MySQL5.6 前只支持对 select 语句分析执行计划。 `replace` 语句是跟 `instert` 语句非常类似，只是插入的数据和表中存在的数据（存在主键或者唯一索引）**冲突**的时候`replace` 语句会把原来的数据替换新插入的数据，表中不存在唯一的索引或主键，则直接插入新的数据。

### 如何分析SQL语句的执行计划？

**下面对SQL语句执行计划中的各个字段的含义进行介绍并举例说明。**

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/执行计划.png)

#### 1.id 列

**id 标识查询执行的顺序，当 id 相同时，由上到下分析执行，当 id 不同时，由大到小分析执行。**

id 列中的值只有两种情况，一组数字（说明查询的 SQL 语句对数据对象的操作顺序）或者NULL（代表数据由另外两个查询的 union 操作后所产生的结果集）。

```sql
EXPLAIN
SELECT course_id,class_name,level_name,title,study_cnt
FROM imc_course a
JOIN imc_class b on b.class_id=a.class_id
JOIN imc_level c on c.level_id =a.level_id
WHERE study_cnt > 3000
```

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/分析执行计划.png)

返回 3 行结果，并且 ID 值是一样的。由上往下读取 sql 的执行计划，第一行是 table c表作为驱动表 ，等于是以 C 表为基础来进行循环嵌套的一个关联查询。 （`4*100*1 =400` 总共扫描 400 行等到数据）

#### 2.select_type 列

| **值**             | **含义**                                                     |
| ------------------ | ------------------------------------------------------------ |
| SIMPLE             | 不包含子查询或者 UNION 操作的查询（简单查询）                |
| PRIMARY            | 查询中如果包含任何子查询，那么最外层的查询则被标记为PRIMARY  |
| SUBQUERY           | select 列表中的子查询                                        |
| DEPENDENT SUBQUERY | 依赖外部结果的子查询                                         |
| UNION              | union 操作的第二个或者之后的查询值为 union                   |
| DEPENDENT UNION    | 当 union 作为子查询时，第二或是第二个后的查询的值为select_type |
| UNION RESULT       | union 产生的结果集                                           |
| DERIVED            | 出现在 from 子句中的子查询（派生表）                         |

例如：查询学习人数大于 3000, 合并课程是 MySQL 的记录。

```
EXPLAIN
SELECT 
course_id,class_name,level_name,title,study_cnt
FROM imc_course a
join imc_class b on b.class_id =a.class_id
join imc_level c on c.level_id = a.level_id
WHERE study_cnt > 3000

union

SELECT course_id,class_name,level_name,title,study_cnt
FROM imc_course a
join imc_class b on b.class_id = a.class_id
join imc_level c on c.level_id = a.level_id
WHERE class_name ='MySQL'
```

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/结果集.png)

分析数据表：先看 id 等于 2

`id=2` 则是查询 mysql 课程的 sql 信息，分别是 b,a,c 3个表，是 union 操作，select_type 是 UNION。

id=1 为是查询学习人数 3000人 的 sql 信息，是 primary 操作的结果集，分别是c,a,b 3 个表，select_type 为 PRIMARY。

最后一行是 NULL, select_type 是 UNION RESULT 代表是 2 个 sql 组合的结果集。

#### 3.table 列

指明是该SQL语句从哪个表中获取数据

| **值**                       | **含义**                                                  |
| ---------------------------- | --------------------------------------------------------- |
| `<table name>`               | 展示数据库表名（如果表取了别名显示别名）                  |
| `<unionM, N>`                | 由 ID 为 M、N 查询 union 产生的结果集                     |
| `<dirived N> / <subquery N>` | 由 ID 为 N 的查询产生的结果（通常也是一个子查询的临时表） |

```
EXPLAIN
SELECT
course id,class name,level name,title,study cnt
FROM imc course a
join imc class b on b.class id =a.class id
join imc level c on c.level id = a.level id
WHERE study cnt > 3000

union

SELECT course id,class name,level name,title,study _cnt
FROM imc course a
join imc class b on b.class id = a.class id
join imc level c on c.level id = a.level id
WHERE class name ='MySOL'
```

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/双结果集.png)

#### 4.type 列

注意: **在 MySQL 中不一定是使用 JOIN 才算是关联查询，实际上 MySQL 会认为每一个查询都是连接查询，就算是查询一个表，对 MySQL 来说也是关联查询。**

`type` 的取值是体现了 MySQL 访问数据的一种方式。`type` 列的值**按照性能高到低排列** system > const > eq_ref > ref > ref_or_null > index_merge > range > index > ALL

| **值**      | **含义**                                                     |
| ----------- | ------------------------------------------------------------ |
| system      | const 连接类型的特例，当查询的表只有一行时使用               |
| const       | **表中有且只有一个匹配的行时使用，如对主键或唯一索引的查询**，这是效率最高的连接方式 |
| eq_ref      | 唯一索引或主键查询，对应每个索引建，表中只有一条记录与之匹配【A表扫描每一行 B 表只有一行匹配满足】 |
| ref         | 非主键非唯一索引等值扫描，例如普通非唯一索引                 |
| ref_or_null | 类似于 ref 类型的查询，但是附加了对 NULL 值列的查询          |
| index_merge | 表示使用了索引合并优化方法                                   |
| range       | 索引范围扫描，常见于 between、>、< 这样的查询条件            |
| index       | FULL index Scan 全索引扫描，同 ALL 的区别是，遍历的是索引树  |
| ALL         | FULL TABLE Scan 全表扫描，效率最差的连接方式                 |

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/type列.png)

- 如果 `where class_name like “MySQL%”`，type 类型为？

    虽然 class_name 加了索引 ，但是使用 where 的 like% 右统配，所以会走索引范围扫描。

    ```sql
    EXPLAIN
    SELECT
    course id,class name,level name,title,study_cnt
    FROM imc course a
    join imc class b on b.class id= a.class id
    join imc level c on c.level id = a.level id
    WHERE class namelike'MySQL%'
    ```

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/type类型.png)﻿﻿

- 如果 `where class_name like “%MySQL%”`，type类型为？

    虽然 `class_name` 加了索引 ，但是使用 `where` 的 %like% 左右统配，所以会走全索引扫描，如果不加索引的话，左右统配会走全表扫描。

    > 这里说明不一定对，例如针对 adowner 使用 %xxx%，如果 select * 则走 all，如果 select adowner 则走 index;
    
    ```sql
    EXPLAIN
    SELECT
    course id,class name,level name,title,study_cnt
    FROM imc course a
    join imc class b on b.class id= a.class id
    join imc level c on c.level id = a.level id
    WHERE class_name like'%MySQL%'
    ```

![img](jingdong_%E6%85%A2SQL%E4%BC%98%E5%8C%96.resource/type类型2.png)

#### 5.possible_key、key 列

`possible_keys` 说明表可能用到了哪些索引，而 `key` 是指实际上使用到的索引。基于查询列和过滤条件进行判断。查询出来都会被列出来，但是不一定会是使用到。

如果在表中没有可用的索引，那么 key 列展示 NULL，possible_keys 是 NULL，这说明查询到覆盖索引。

#### 6.key_len 列

实际用的索引使用的字节数。

注意，在联合索引中，如果有 3 列，那么总字节是长度是 100 个字节的话，那么 key_len 值数据可能少于 100 字节，比如 30 个字节，这就说明了**查询中并没有使用联合索引的所有列。而只是利用到某一些列或者 2 列**。

**key_len 的长度是由表中的定义的字段长度来计算的，并不是存储的实际长度**，所以满足数据最短的实际字段存储，因为会直接影响到生成执行计划的生成 。

#### 7.ref 列

指出那些列或常量被用于索引查找

#### 8.rows 列

该列有两个含义：

- 根据统计信息预估的扫描行数。

- 另一方面是关联查询内嵌的次数，每获取匹配一个值都要对目标表查询，所以循环次数越多性能越差。

**因为扫描行数的值是预估的，所以并不准确。**

#### 9.filtered 列

**表示返回结果的行数占需读取行数的百分比。**

filtered 列跟 rows 列是有关联的，是返回预估符合条件的数据集，再去取的行的百分比。也是预估的值。**数值越高查询性能越好。**

#### 10.Extra 列

包括了不适合在其他列中所显示的额外信息。

| **值**                       | **含义**                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| Distinct                     | 优化distinct操作，在找到第一匹配的元组后即停止找同样值得动作 |
| Not exists                   | 使用not exisits来优化查询                                    |
| Using filesort               | 使用文件来进行排序，通常会出现在 order by 或group by 查询中  |
| Using index                  | 使用了覆盖索引进行查询【查询所需要的信息用所用来获取，不需要对表进行访问】 |
| Using temporary              | MySQL 需要使用临时表来处理，常见于排序、子查询和分组查询     |
| Using where                  | 需要在MySQL服务器层使用where条件来过滤数据                   |
| select tables optimized away | 直接通过索引来获取数据，不用访问表                           |

## 四、SQL索引失效

- 最左前缀原则：要求建立索引的一个列都不能缺失，否则会出现索引失效。
- 索引列上的计算，函数、类型转换（列类型是字符串在条件中需要使用引号，否则不走索引）、均会导致索引失效。
- 索引列中使用is not null会导致索引列失效。
- 索引列中使用like查询的前以%开头会导致索引列失效。
- 索引列用 or 连接时会导致索引失效。

## 五、实际优化慢SQL中遇到问题

下面是在慢SQL优化过程中所遇到的一些问题。

•**MySQL查询到的数据排序是稳定的么？**

•**force_index的使用方式？**

•**为什么有时候order by id会导致索引失效？**

•**........未完整理中......**



## 六、总结

通过本次对慢SQL的优化的需求进而发现有关SQL语句执行顺序、执行计划、索引失效场景、底层SQL语句执行原理相关知识还存在盲区，得益于此次需求的开发，有深入的对相关知识进行学习和总结。接下来会对SQL底层是如何执行SQL语句