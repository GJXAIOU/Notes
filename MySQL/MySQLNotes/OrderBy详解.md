# order by 详解

## 一、使用示例

假设用一张员工表，表结构如下：

```mysql
CREATE TABLE `staff` (
`id` BIGINT ( 11 ) AUTO_INCREMENT COMMENT '主键id',
`id_card` VARCHAR ( 20 ) NOT NULL COMMENT '身份证号码',
`name` VARCHAR ( 64 ) NOT NULL COMMENT '姓名',
`age` INT ( 4 ) NOT NULL COMMENT '年龄',
`city` VARCHAR ( 64 ) NOT NULL COMMENT '城市',
PRIMARY KEY ( `id`),
INDEX idx_city ( `city` )
) ENGINE = INNODB COMMENT '员工表';
```

表数据如下：

![图片](OrderBy详解.resource/640.png)

我们现在有这么一个需求：**查询前 10 个，来自深圳员工的姓名、年龄、城市，并且按照年龄小到大排序**。对应的 SQL 语句：

```mysql
select name,age,city from staff where city = '深圳' order by age limit 10;
```

这条语句的逻辑很清楚，但是它的**底层执行流程**是怎样的呢？

## 二、order by 工作原理

### （一）explain 执行计划

用 **Explain**关键字查看一下执行计划

![图片](OrderBy详解.resource/640-16493456044543.png)

- 执行计划的 **key** 字段，表示使用到索引 `idx_city`；
- Extra 字段中的 `Using index condition` 表示索引条件，`Using filesort` 表示用到排序

因此这条 SQL 使用到了索引，并且也用到排序。那么它是**怎么排序**的呢？

### （二）全字段排序

MySQL 会给每个查询线程分配一块小内存用于排序的，称为 `sort_buffer`。什么时候把字段放进去排序呢，其实是通过`idx_city` 索引找到对应的数据，才把数据放进去。

先回顾下索引是怎么找到匹配的数据的，现在先把索引树画出来吧，`idx_city` 索引树如下：

![图片](OrderBy详解.resource/640-16493458643226.png)

`idx_city` 索引树，叶子节点存储的是**主键 id**。还有一棵 id 主键聚族索引树如下：

![图片](OrderBy详解.resource/640-16493459808679.png)

查询语句找到匹配数据过程：先通过 `idx_city` 索引树，找到对应的主键 id，然后再通过拿到的主键 id，搜索 id 主键索引树，找到对应的行数据。

加上 order by 之后，整体的执行流程就是：

1. MySQL 为对应的线程初始化 `sort_buffer`，放入需要查询的 `name`、`age`、`city` 字段；
2. 从索引树 `idx_city`， 找到第一个满足 `city="深圳"` 条件的主键 id，也就是图中的 id=9；
3. 到主键 id 索引树拿到 id=9 的这一行数据， 取 `name`、`age`、`city` 三个字段的值，存到 `sort_buffer`；
4. 从索引树 `idx_city` 拿到下一个记录的主键 id，即图中的 id=13；
5. 重复步骤 3、4 直到 `city` 的值不等于深圳为止；
6. 前面 5 步已经查找到了所有 `city` 为深圳的数据，在 sort_buffer 中，将所有数据根据 age 进行排序；
7. 按照排序结果取前 10 行返回给客户端。

执行示意图如下：

![图片](OrderBy详解.resource/640-164934645030312.png)

将查询所需的字段全部读取到 `sort_buffer` 中，就是**全字段排序**。把查询的所有字段都放到 `sort_buffer`，而 `sort_buffer` 是一块内存来的，如果数据量太大，`sort_buffer` 放不下怎么办呢？

### 磁盘临时文件辅助排序

实际上，`sort_buffer` 的大小是由一个参数控制的：`sort_buffer_size`**。如果要排序的数据小于 `sort_buffer_size`，排序在 `sort_buffer` 内存中完成，如果要排序的数据大于 `sort_buffer_size`，则借助磁盘文件来进行排序**。

如何确定是否使用了磁盘文件来进行排序呢？可以使用以下这几个命令

```mysql
## 打开optimizer_trace，开启统计
set optimizer_trace = "enabled=on";
## 执行SQL语句
select name,age,city from staff where city = '深圳' order by age limit 10;
## 查询输出的统计信息
select * from information_schema.optimizer_trace 
```

可以从结果的 `number_of_tmp_files` 中看出，是否使用了临时文件。

![图片](OrderBy详解.resource/640-164934660791515.png)

`number_of_tmp_files` 表示使用来排序的磁盘临时文件数。如果 `number_of_tmp_files` 大于 0，则表示使用了磁盘文件来进行排序。

使用了磁盘临时文件，整个排序过程如下：

1. 从主键 id 索引树，拿到需要的数据，并放到 `sort_buffer` 内存块中。当 `sort_buffer` 快要满时，就对 `sort_buffer` 中的数据排序，排完后，把数据临时放到磁盘一个小文件中。
2. 继续回到主键 id 索引树取数据，继续放到 `sort_buffer` 内存中，排序后，也把这些数据写入到磁盘临时小文件中。
3. 继续循环，直到取出所有满足条件的数据。最后把磁盘的临时排好序的小文件，合并成一个有序的大文件。

**TPS:** 借助磁盘临时小文件排序，实际上使用的是**归并排序**算法。

问题：既然 `sort_buffer` 放不下，就需要用到临时磁盘文件，这会影响排序效率。那为什么还要把排序不相关的字段（name，city）放到 `sort_buffer` 中呢？为什么不能只放排序相关的 age 字段。

### rowid 排序

rowid 排序就是，只把查询 SQL **需要用于排序的字段和主键 id**，放到 `sort_buffer` 中。那怎么确定走的是全字段排序还是 rowid 排序排序呢？

通过 `max_length_for_sort_data` 参数来区分，它表示 MySQL 用于排序行数据的长度的一个参数，如果单行的长度超过这个值，MySQL 就认为单行太大，就换 rowid 排序。我们可以通过命令看下这个参数取值。

```mysql
show variables like 'max_length_for_sort_data';
```

`max_length_for_sort_data` 默认值是 1024。因为本文示例中 name,age,city 长度=64+4+64 =132 < 1024，所以走的是全字段排序。我们来改下这个参数，改小一点，

```mysql
## 修改排序数据最大单行长度为32
set max_length_for_sort_data = 32;
## 执行查询SQL
select name,age,city from staff where city = '深圳' order by age limit 10;
```

==问题：如果该值小于要排序的字段呢，即如果小于 age 长度呢？==

使用 rowid 排序的话，整个 SQL 执行流程如下：

1. MySQL 为对应的线程初始化 `sort_buffer`，放入需要排序的 age 字段，以及主键 id；
2. 从索引树 `idx_city`， 找到第一个满足 `city='深圳’` 条件的主键 id，也就是图中的 id=9；
3. 到主键 id 索引树拿到 `id=9` 的这一行数据， 取 `age` 和主键 `id` 的值，存到 `sort_buffer`；
4. 从索引树 `idx_city` 拿到下一个记录的主键 id，即图中的 id=13；
5. 重复步骤 3、4 直到 `city` 的值不等于深圳为止；
6. 前面 5 步已经查找到了所有 city 为深圳的数据，在 `sort_buffer` 中，将所有数据根据 age 进行排序；
7. 遍历排序结果，取前 10 行，并按照 id 的值回到原表中，取出 `city`、`name` 和 `age` 三个字段返回给客户端。

执行示意图如下：

![图片](OrderBy详解.resource/640-164934756941721.png)

对比一下**全字段排序**的流程，rowid 排序多了一次**回表**。

> 回表：拿到主键再回到主键索引查询的过程，就叫做回表

我们通过 `optimizer_trace`，可以看到是否使用了 rowid 排序的：

```mysql
## 打开optimizer_trace，开启统计
set optimizer_trace = "enabled=on";
## 执行SQL语句
select name,age,city from staff where city = '深圳' order by age limit 10;
## 查询输出的统计信息
select * from information_schema.optimizer_trace 
```

![图片](OrderBy详解.resource/640-164934720785718.png)

### 全字段排序与 rowid 排序对比

- 全字段排序：`sort_buffer` 内存不够的话，就需要用到磁盘临时文件，造成**磁盘访问**。
- rowid 排序：`sort_buffer` 可以放更多数据，但是需要再回到原表去取数据，比全字段排序多一次**回表**。

一般情况下，对于 InnoDB 存储引擎，会优先使**用全字段**排序。可以发现 `max_length_for_sort_data` 参数设置为 1024，这个数比较大的。一般情况下，排序字段不会超过这个值，也就是都会走**全字段**排序。

## 三、order by 的一些优化思路

我们如何优化 order by 语句呢？

- 因为数据是无序的，所以就需要排序。如果数据本身是有序的，那就不用排了。而索引数据本身是有序的，我们通过建立**联合索引**，优化 order by 语句。
- 我们还可以通过调整 `max_length_for_sort_data` 等参数优化；

### 联合索引优化

再回顾下示例 SQL 的查询计划

```mysql
explain select name,age,city from staff where city = '深圳' order by age limit 10;
```

![图片](OrderBy详解.resource/640-164934774265024.png)

我们给查询条件`city`和排序字段`age`，加个联合索引 `idx_city_age`。再去查看执行计划

```mysql
alter table staff add  index idx_city_age(city,age);
explain select name,age,city from staff where city = '深圳' order by age limit 10;
```

![图片](OrderBy详解.resource/640-164934781244727.png)

可以发现，加上 `idx_city_age` 联合索引，就不需要 `Using filesort` 排序了。为什么呢？因为索引本身是有序的，我们可以看下 `idx_city_age` 联合索引示意图，如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/PoF8jo1Pmpwlj1dDtegxsHWfNCSQev65hh5Pu2cIJrytFx1YZicLJcr2NKhSvZV4lJFAocibRBAyLtFlXGq0s9Yw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

整个 SQL 执行流程为：

1. 从索引 `idx_city_age` 找到满足 `city='深圳’` 的主键 id
2. 到主键 id 索引取出整行，拿到 `name`、`city`、`age` 三个字段的值，作为结果集的一部分直接返回
3. 从索引 `idx_city_age` 取下一个记录主键 id
4. 重复步骤 2、3，直到查到第 10 条记录，或者是不满足 `city='深圳’` 条件时循环结束。

流程示意图如下：

![图片](OrderBy详解.resource/640-164934798036530.png)

从示意图看来，还是有一次**回表**操作。针对本次示例，有没有更高效的方案呢？有的，可以使用**覆盖索引**：

> 覆盖索引：在查询的数据列里面，不需要回表去查，直接从索引列就能取到想要的结果。换句话说，你 SQL 用到的索引列数据，覆盖了查询结果的列，就算上覆盖索引了。

我们给 `city`，`name`，`age` 组成一个联合索引，即可用到了覆盖索引，这时候 SQL 执行时，连回表操作都可以省去。

### 调整参数优化

我们还可以通过调整参数，去优化 order b y 的执行。比如可以调整 `sort_buffer_size` 的值。因为 `sort_buffer` 值太小，数据量大的话，会借助磁盘临时文件排序。如果 MySQL 服务器配置高的话，可以使用稍微调整大点。

我们还可以调整 `max_length_for_sort_data` 的值，这个值太小的话，order by 会走 rowid 排序，会回表，降低查询性能。所以 `max_length_for_sort_data` 可以适当大一点。

当然，很多时候，这些 MySQL 参数值，我们直接采用默认值就可以了。

## 四、使用 order by  的一些注意点

### 没有 where 条件，order by 字段需要加索引吗

日常开发过程中，我们可能会遇到没有 where 条件的 order by，那么，这时候 order by 后面的字段是否需要加索引呢。如有这么一个 SQL，create_time 是否需要加索引：

```mysql
select * from A order by create_time;
```

**无条件查询的话，即使 `create_time`上有索引，也不会使用到**。因为 MySQL 优化器认为走普通二级索引，再去回表成本比全表扫描排序更高。所以选择走全表扫描,然后根据全字段排序或者 rowid 排序来进行。

如果查询 SQL 修改一下：

```mysql
lselect * from A order by create_time limit m;
```

无条件查询，如果 m 值较小，是可以走索引的。因为 MySQL 优化器认为，根据索引有序性去回表查数据，然后得到 m 条数据，就可以终止循环，那么成本比全表扫描小，则选择走二级索引。

### 分页 limit 过大时，会导致大量排序怎么办?

假设 SQL 如下：

```mysql
select * from A order by a limit 100000,10
```

- 可以记录上一页最后的 id，下一页查询时，查询条件带上 id，如：where id > 上一页最后 id limit 10。
- 也可以在业务允许的情况下，限制页数。

### 索引存储顺序与 order by 不一致，如何优化？

假设有联合索引 `idx_age_name`, 我们需求修改为这样：**查询前 10 个员工的姓名、年龄，并且按照年龄小到大排序，如果年龄相同，则按姓名降序排**。对应的 SQL 语句就可以这么写：

```mysql
select name,age from staff  order by age ,name desc limit 10;
```

我们看下执行计划，发现使用到 Using filesort。

![图片](OrderBy详解.resource/640-164934849969933.png)

这是因为，`idx_age_name` 索引树中，age 从小到大排序，如果 `age` 相同，再按 `name` 从小到大排序。而 order by 中，是按 age 从小到大排序，如果 age 相同，再按 name 从大到小排序。即索引存储顺序与 order by 不一致。

我们怎么优化呢？如果 MySQL 是 8.0 版本，支持 `Descending Indexes`，可以这样修改索引：

```mysql
CREATE TABLE `staff` (
  `id` bigint(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `id_card` varchar(20) NOT NULL COMMENT '身份证号码',
  `name` varchar(64) NOT NULL COMMENT '姓名',
  `age` int(4) NOT NULL COMMENT '年龄',
  `city` varchar(64) NOT NULL COMMENT '城市',
  PRIMARY KEY (`id`),
  KEY `idx_age_name` (`age`,`name` desc) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=15 DEFAULT CHARSET=utf8 COMMENT='员工表';
```

### 使用了 in 条件多个属性时，SQL 执行是否有排序过程

如果我们有联合索引 `idx_city_name`，执行这个 SQL 的话，是不会走排序过程的，如下：

```mysql
select * from staff where city in ('深圳') order by age limit 10;
```

![图片](OrderBy详解.resource/640-164934875350136.png)

但是，如果使用 in 条件，并且有多个条件时，就会有排序过程。

```
 explain select * from staff where city in ('深圳','上海') order by age limit 10;
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/PoF8jo1Pmpwlj1dDtegxsHWfNCSQev65mBZzGBAC5xu0PZoPndgHwrV9ELiaibMg07wiaHMc1C9U3suZkOLGniaKnQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

这是因为:in 有两个条件，在满足深圳时，age 是排好序的，但是把满足上海的 age 也加进来，就不能保证满足所有的 age 都是排好序的。因此需要 Using filesort。

