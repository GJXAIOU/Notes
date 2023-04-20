# MySQL哪些操作会记录至Binlog文件？操作影响行数为0会写Binlog？

## 一、前言

> 之前对什么情况会写Binlog文件的理解是，当执行DDL或DML操作成功且返回的影响行数大于0时，会将操作记录至Binlog，后面发现这样描述不太准确，需限定在ROW模式下。

------

> **先解释两个基本数语**
>
> - `DDL(Data Definition Language)`，数据定义语言，用于创建、修改和删除库或表的结构，即表示库或表的结构变更。例如`CREATE, ALTER, DROP...`
> - `DML(Data Manipulation Language)`，数据操作语言，用于插入、修改、删除和查询表的数据。例如`INSERT, UPDATE, DELETE, SELECT(本文说的DML不包括SELECT等查询语句)...`

## 二、BINLOG文件

> 即二进制文件，记录了数据库的所有变更，并以二进制的形式保存在磁盘上。

### （一）主要用途

- 主从复制，MySQL使用主从架构后，从库和主库建立连接后，主库会将Binlog发送至从库，而从库通过解析回放执行Binglog文件的内容，达到主从一致的效果。所有的数据订阅、同步和迁移的工具也是基于这一原理来实现的。
- 数据恢复，Binlog详细记录了数据库的所有变化，例如当某一时刻某些数据因为一些错误的操作而导致数据错误覆盖，可以使用Mysqlbinlog工具将数据恢复至操作之前的某一时刻。

### （二）记录的模式(基于MySQL8.0)

- Statement，Binlog存储的是原始SQL语句。
    - 优点：不需要记录每一行的变化，和`Row`相比减少了Binlog日志量，节约了IO，提高了性能。
    - 缺点：由于记录的只是执行语句，某些情况下在从库上回放会导致主从不一致，例如SQL中包含`UUID(),USER()`等函数。
- Row，Binlog存储的是每一行所有字段变化前后的值。
    - 优点：由于记录了每行每个字段变更前后的值，不会出现`Statement`模式带来的问题。
    - 缺点：一条SQL可能导致100行产生变化，那么Binlog就会详细记录这100行每个字段变化前后的值，`Statement`模式只需要记录这一条SQL，所以和`Statement`模式相比**可能会产生大量的日志内容**。
- `Mixed`，混合模式，会对于每一条SQL进行分析，当使用`Statement`模式记录会导致问题时使用`Row`模式记录，例如SQL中包含`UUID(),USER()`等函数会使用`Row`模式。

## 三、哪些情况会写Binlog文件？操作没产生表数据变化会写Binlog吗？

### （一）结论(基于MySQL8.0)

- `Statement`模式下，会记录所有执行成功DDL和DML操作，包括`UPDATE 和 DELETE`操作影响行数为0的情况。
- `Row`模式下，会记录所有执行成功且返回的影响行数大于0的DDL和DML操作，当`UPDATE 和 DELETE`操作的影响行数为0时不会记录操作至Binlog。
- Mixed 模式下，当一条SQL分析到应该使用 Statement 模式写Binlog时，按照 Statement 规则，当一条SQL分析到应该使用 Row 模式写Binlog时，按照 Row 规则。
    - 一条UPDATE中含有USER()函数，且执行成功但操作返回的影响行数为0，由于SQL中含有USER()函数，会按照`Row`规则，故这不会记录至Binlog。
    - 一条SQL`UPDATE SET NAME = 'kd' WHERE ID = 1`，且ID为1的数据不存在，尽管执行成功且返回的影响行数为0，但这一条SQL会使用`Statement`规则，所以这个操作依然会记录至Binlog。

### （二）准备环境

**查看版本和服务状态**

```sql
mysql> SELECT VERSION()\G
*************************** 1. row ***************************
VERSION(): 8.0.27
1 row in set (0.00 sec)

mysql> SHOW MASTER STATUS\G
*************************** 1. row ***************************
File: binlog.000003
         Position: 5423
     Binlog_Do_DB: 
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
复制代码
```

**准备库表**

```sql
mysql> CREATE DATABASE KD;
Query OK, 1 row affected (0.01 sec)

mysql> USE KD
Database changed

mysql> CREATE TABLE `TEST`(`id` int, `name` varchar(100), PRIMARY KEY(`id`));
Query OK, 0 rows affected (0.03 sec)
复制代码
```

##### 验证`Statement`

```sql
mysql> SET BINLOG_FORMAT='STATEMENT';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'BINLOG_FORMAT'\G
*************************** 1. row ***************************
Variable_name: binlog_format
        Value: STATEMENT
1 row in set (0.01 sec)
```

**操作步骤**

- 插入：`INSERT INTO TEST VALUES(1,'1');`【记录至Binlog】
- 修改：`UPDATE TEST SET NAME ='kd' WHERE ID=1;`【记录至Binlog】
- 修改：`再次执行上一步的SQL -〉执行成功但操作的影响行数为0`【记录至Binlog】

```sql
mysql> INSERT INTO `TEST` VALUES(1,'1');
Query OK, 1 row affected (0.01 sec)

mysql> UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=1;
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=1;
Query OK, 0 rows affected (0.01 sec)
Rows matched: 1  Changed: 0  Warnings: 0

SHOW BINLOG EVENTS IN 'binlog.000003';
复制代码
```

**尽管第三条SQL操作的影响行数0，但是依然记录至了Binlog**

![image.png](xitujuejing_MySQL%E5%93%AA%E4%BA%9B%E6%93%8D%E4%BD%9C%E4%BC%9A%E8%AE%B0%E5%BD%95%E8%87%B3Binlog%E6%96%87%E4%BB%B6%EF%BC%9F%E6%93%8D%E4%BD%9C%E5%BD%B1%E5%93%8D%E8%A1%8C%E6%95%B0%E4%B8%BA0%E4%BC%9A%E5%86%99Binlog.resource/46c218dbc5c44da59394ca2b6dda9ad1~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

##### 验证`Row`

```sql
mysql> SET BINLOG_FORMAT='ROW';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'BINLOG_FORMAT'\G
*************************** 1. row ***************************
Variable_name: binlog_format
        Value: ROW
1 row in set (0.00 sec)
复制代码
```

**操作步骤**

- 插入：`INSERT INTO  TEST  VALUES(3,'3');`【记录至Binlog】
- 修改：`UPDATE TEST SET NAME ='kd' WHERE ID=3;`【记录至Binlog】
- 修改：`再次执行上一步的SQL -〉执行成功但操作的影响行数为0`【没有记录至Binlog】

```sql
mysql> INSERT INTO `TEST` VALUES(3,'3');
Query OK, 1 row affected (0.01 sec)

mysql> UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=3;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=3;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0

SHOW BINLOG EVENTS IN 'binlog.000003';
```

- 对于`Row`模式来说执行成功且返回的影响行数大于0才会记录Binlog，故第三条SQL没有记录Binlog。
- `Row`模式下需使用Mysqlbinlog工具解析Binlog才能看到详细每行的变化。![image.png](xitujuejing_MySQL%E5%93%AA%E4%BA%9B%E6%93%8D%E4%BD%9C%E4%BC%9A%E8%AE%B0%E5%BD%95%E8%87%B3Binlog%E6%96%87%E4%BB%B6%EF%BC%9F%E6%93%8D%E4%BD%9C%E5%BD%B1%E5%93%8D%E8%A1%8C%E6%95%B0%E4%B8%BA0%E4%BC%9A%E5%86%99Binlog.resource/0dbf487b9c834d6fb9b136c0b6ce9d33~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

`Row`模式下Binlog详情

```shell
// 通过mysqlbinlog工具解析Binlog文件，因为Binlog是二进制文件直接打开会收获乱码
root# mysqlbinlog --base64-output=decode-rows -v binlog.000003
复制代码
```

**INSERT INTO `TEST` VALUES(3,'3');的Binlog详情**

```sql
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 6730
#220220  8:28:26 server id 1  end_log_pos 6803 CRC32 0x90515f14 Query thread_id=10 exec_time=0 error_code=0
SET TIMESTAMP=1645345706/*!*/;
BEGIN
/*!*/;
# at 6803
#220220  8:28:26 server id 1  end_log_pos 6859 CRC32 0xe8817f41 Table_map: `KD`.`TEST` mapped to number 97
# at 6859
#220220  8:28:26 server id 1  end_log_pos 6902 CRC32 0x94a36e8f Write_rows: table id 97 flags: STMT_END_F
### INSERT INTO `KD`.`TEST`
### SET
###   @1=3
###   @2='3'
# at 6902
#220220  8:28:26 server id 1  end_log_pos 6933 CRC32 0xfc598006 Xid = 137
COMMIT/*!*/;
复制代码
```

**UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=3;的Binlog详情**

```sql
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 7012
#220220  8:28:34 server id 1  end_log_pos 7094 CRC32 0xf9220b3c Query thread_id=10 exec_time=0 error_code=0
SET TIMESTAMP=1645345714/*!*/;
BEGIN
/*!*/;
# at 7094
#220220  8:28:34 server id 1  end_log_pos 7150 CRC32 0x7f47ccbc Table_map: `KD`.`TEST` mapped to number 97
# at 7150
#220220  8:28:34 server id 1  end_log_pos 7203 CRC32 0xbddcd2d7 Update_rows: table id 97 flags: STMT_END_F
### UPDATE `KD`.`TEST`
### WHERE
###   @1=3
###   @2='3'
### SET
###   @1=3
###   @2='kd'
# at 7203
#220220  8:28:34 server id 1  end_log_pos 7234 CRC32 0xd1959d3f Xid = 138
COMMIT/*!*/;
复制代码
```

##### 验证`MIXED`

```sql
mysql> SET BINLOG_FORMAT='MIXED';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'BINLOG_FORMAT'\G
*************************** 1. row ***************************
Variable_name: binlog_format
        Value: MIXED
1 row in set (0.01 sec)
复制代码
```

**操作步骤**

- 插入：`INSERT INTO `TEST` VALUES(4,'4');`【使用的是`Statement`模式，记录至Binlog】
- 修改：`UPDATE `TEST`SET`NAME`='kd' WHERE `ID`=4;`【使用的是`Statement`模式，记录至Binlog】
- 修改：`再次执行上一步的SQL -〉执行成功但操作的影响行数为0`【使用的是`Statement`模式，记录至Binlog】
- 修改：`UPDATE `TEST`SET`NAME`=USER() WHERE `ID`=4;`【使用的是`Row`模式，记录至Binlog】
- 修改：`再次执行上一步的SQL -〉执行成功但操作的影响行数为0`【使用的是`Row`模式，没有记录至Binlog】

```sql
mysql> INSERT INTO `TEST` VALUES(4,'4');
Query OK, 1 row affected (0.01 sec)

mysql> UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=4;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> UPDATE `TEST` SET `NAME`='kd' WHERE `ID`=4;
Query OK, 0 rows affected (0.01 sec)
Rows matched: 1  Changed: 0  Warnings: 0

mysql> UPDATE `TEST` SET `NAME`=USER() WHERE `ID`=4;
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> UPDATE `TEST` SET `NAME`=USER() WHERE `ID`=4;
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0

SHOW BINLOG EVENTS IN 'binlog.000003';
复制代码
```

- `Mixed`模式下，当一条SQL分析到应该使用`Statement`模式写Binlog时，按照`Statement`规则，当一条SQL分析到应该使用`Row`模式写Binlog时，按照`Row`规则。
- 一二三条使用SQL使用的是`Statement`，四五条SQL使用的是`Row`。
- 由于第五条SQL使用的是`Row`，且执行成功但操作的影响行数为0，故不记录Binlog。![image.png](xitujuejing_MySQL%E5%93%AA%E4%BA%9B%E6%93%8D%E4%BD%9C%E4%BC%9A%E8%AE%B0%E5%BD%95%E8%87%B3Binlog%E6%96%87%E4%BB%B6%EF%BC%9F%E6%93%8D%E4%BD%9C%E5%BD%B1%E5%93%8D%E8%A1%8C%E6%95%B0%E4%B8%BA0%E4%BC%9A%E5%86%99Binlog.resource/c58fab439f814a81bb4adb79ef94cd7b~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

**第四条UPDATE `TEST` SET `NAME`=USER() WHERE `ID`=4;的Binlog详情**

```sql
root# mysqlbinlog --base64-output=decode-rows -v binlog.000003

SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 8248
#220220  8:52:12 server id 1  end_log_pos 8330 CRC32 0xfd4b0743 Query thread_id=10 exec_time=0 error_code=0
SET TIMESTAMP=1645347132/*!*/;
BEGIN
/*!*/;
# at 8330
#220220  8:52:12 server id 1  end_log_pos 8386 CRC32 0xf9725844 Table_map: `KD`.`TEST` mapped to number 97
# at 8386
#220220  8:52:12 server id 1  end_log_pos 8452 CRC32 0x9049fa91 Update_rows: table id 97 flags: STMT_END_F
### UPDATE `KD`.`TEST`
### WHERE
###   @1=4
###   @2='kd'
### SET
###   @1=4
###   @2='root@localhost'
# at 8452
#220220  8:52:12 server id 1  end_log_pos 8483 CRC32 0xc51d1b7a Xid = 147
COMMIT/*!*/;
复制代码
```

### 总结

- 由于篇幅有限，没有把DELETE的测试放在文章里面，但DELETE和UPDATE是一致的。
- `Statement`模式下，会记录所有执行成功DDL和DML操作，包括`UPDATE和DELETE`操作的影响行数为0的情况。
- `Row`模式下，会记录所有执行成功且返回的影响行数大于0的DDL和DML操作，当`UPDATE和DELETE`操作的影响行数为0时不会记录操作至Binlog。
- Mixed 模式下，当一条SQL应该使用 Statement 模式写Binlog时，按照 Statement 规则，当一条SQL应该使用 Row 模式写Binlog时，按照Row 规则。
    - 一条UPDATE中含有USER()函数，且执行成功但操作返回的影响行数为0，由于SQL中含有USER()函数，会按照`Row`规则，故这不会记录至Binlog。
    - 一条SQL`UPDATE SET NAME = 'kd' WHERE ID = 1`，且ID为1的数据不存在，尽管执行成功且返回的影响行数为0，但这一条SQL会使用`Statement`规则，所以这个操作依然会记录至Binlog。