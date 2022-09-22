# 12丨视图在SQL中的作用是什么，它是怎样工作的？

我们之前对 SQL 中的数据表查询进行了讲解，今天我们来看下如何对视图进行查询。**视图，也就是我们今天要讲的虚拟表，本身是不具有数据的**，它是 SQL 中的一个重要概念。从下面这张图中，你能看到，虚拟表的创建连接了一个或多个数据表，不同的查询应用都可以建立在虚拟表之上。

![image-20220921001257936](12%E4%B8%A8%E8%A7%86%E5%9B%BE%E5%9C%A8SQL%E4%B8%AD%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E4%BB%80%E4%B9%88%EF%BC%8C%E5%AE%83%E6%98%AF%E6%80%8E%E6%A0%B7%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9F.resource/image-20220921001257936.png)

视图一方面可以帮我们使用表的一部分而不是所有的表，另一方面也可以针对不同的用户制定不同的查询视图。比如，针对一个公司的销售人员，我们只想给他看部分数据，而某些特殊的数据，比如采购的价格，则不会提供给他。

刚才讲的只是视图的一个使用场景，实际上视图还有很多作用，今天我们就一起学习下。今天的文章里，你将重点掌握以下的内容：

1. 什么是视图？如何创建、更新和删除视图？
2. 如何使用视图来简化我们的 SQL 操作？
3. 视图和临时表的区别是什么，它们各自有什么优缺点？

## 如何创建，更新和删除视图

视图作为一张虚拟表，帮我们封装了底层与数据表的接口。它相当于是一张表或多张表的数据结果集。视图的这一特点，可以帮我们简化复杂的 SQL 查询，比如在编写视图后，我们就可以直接重用它，而不需要考虑视图中包含的基础查询的细节。同样，我们也可以根据需要更改数据格式，返回与底层数据表格式不同的数据。

通常情况下，小型项目的数据库可以不使用视图，但是在大型项目中，以及数据表比较复杂的情况下，视图的价值就凸显出来了，它可以帮助我们把经常查询的结果集放到虚拟表中，提升使用效率。理解和使用起来都非常方便。

### 创建视图：CREATE VIEW

那么该如何创建视图呢？创建视图的语法是：

```
CREATE VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition
```

实际上就是我们在 SQL 查询语句的基础上封装了视图 VIEW，这样就会基于 SQL 语句的结果集形成一张虚拟表。其中 view_name 为视图名称，column1、column2 代表列名，condition 代表查询过滤条件。

我们以 NBA 球员数据表为例。我们想要查询比 NBA 球员平均身高高的球员都有哪些，显示他们的球员 ID 和身高。假设我们给这个视图起个名字 player_above_avg_height，那么创建视图可以写成：

```
CREATE VIEW player_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player)
```

视图查询结果（18 条记录）：

```mysql
+-----------+--------+
| player_id | height |
+-----------+--------+
|     10003 |   2.11 |
...
|     10037 |   2.08 |
+-----------+--------+
18 rows in set (0.00 sec)
```

当视图创建之后，它就相当于一个虚拟表，可以直接使用：

```
SELECT * FROM player_above_avg_height
```

运行结果和上面一样。

### 嵌套视图

当我们创建好一张视图之后，还可以在它的基础上继续创建视图，比如我们想在虚拟表 player_above_avg_height 的基础上，找到比这个表中的球员平均身高高的球员，作为新的视图 player_above_above_avg_height，那么可以写成：

```
CREATE VIEW player_above_above_avg_height AS
SELECT player_id, height
FROM player
WHERE height > (SELECT AVG(height) from player_above_avg_height)
```

视图查询结果（11 条记录）：

```mysql
mysql> SELECT * FROM  player_above_above_avg_height;
+-----------+--------+
| player_id | height |
+-----------+--------+
|     10003 |   2.11 |
|     10004 |   2.16 |
|     10009 |   2.11 |
|     10010 |   2.08 |
|     10011 |   2.08 |
|     10015 |   2.11 |
|     10023 |   2.11 |
|     10024 |   2.11 |
|     10032 |   2.08 |
|     10033 |   2.08 |
|     10037 |   2.08 |
+-----------+--------+
11 rows in set (0.04 sec)
```

你能看到这个视图的数据记录数为 11 个，比之前的记录少了 7 个。

### 修改视图：ALTER VIEW

修改视图的语法是：

```
ALTER VIEW view_name AS
SELECT column1, column2
FROM table
WHERE condition
```

你能看出来它的语法和创建视图一样，只是对原有视图的更新。比如我们想更新视图 player_above_avg_height，增加一个 player_name 字段，可以写成：

```
ALTER VIEW player_above_avg_height AS
SELECT player_id, player_name, height
FROM player
WHERE height > (SELECT AVG(height) from player)
```

这样的话，下次再对视图进行查询的时候，视图结果就进行了更新。

```
SELECT * FROM player_above_avg_height
```

运行结果（18 条记录）：

```mysql
```

