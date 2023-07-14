---

---

## 前言

Binlog 作为 MySQL 中鼎鼎大名的一部分，是一个很熟悉的陌生人。正好最近在工作中遇到处理 binlog 的场景，便正好总结回顾一下这个知识点。本文将从以下几个部分详述：

- binlog 是什么？
- binlog 可以做什么？起什么作用？
- binlog 物理存储形式？
- binlog 处理工具？

本文使用 MySQL 版本为：5.7.18

## 一、binlog 概念

binlog 即二进制日志文件，是 MySQL 的其中一种日志文件，MySQL 的日志文件还包括：错误日志文件、慢日志文件、查询日志文件。

binlog 中记录了对数据库执行更改的所有操作，这里的关键词是「更改」，有两个含义：

- 像 select/show 这种操作是不会记录到 binlog 中；

- 部分操作即使没有导致数据库变化（语句返回为 changed:0）也可能写入 binlog；

    ==》 todo 哪些命令会记录 binlog



### （二）开启 binlog

默认情况下 binlog 是关闭的，我们可以通过参数 `log_bin` 开启;

```sql
show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
1 row in set (0.00 sec)

mysql> SET global log_bin = ON;
ERROR 1238 (HY000): Variable 'log_bin' is a read only variable
```

因为系统变量 `log_bin` 为静态参数，不能动态修改，所以不能通过上述方式修改；

正确修改方式： 将 my.cnf 中的 [mysqld] 下面添加 `log_bin=[DIR\[filename]`，其中 DIR 指定二进制文件的存储路径，fileName 指定二进制文件的文件名。

my.cnf 位于 MySQL 安装目录下，其中 MySQL 的按照目录可以通过以下命令查询：

```mysql
mysql> select @@basedir;
+--------------------------------------------+
| @@basedir                                  |
+--------------------------------------------+
| /usr/local/mysql-5.7.18-macos10.12-x86_64/ |
+--------------------------------------------+
1 row in set (0.00 sec)
```

如果上述命令还是找不到 my.cnf 文件，可以通过下面命令获取 MySQL 的配置文件的默认加载顺序

```shell
mysql --help I grep my.cnf
## 在该命令输出结果中，存在以下信息
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
```

上述是加载顺序，如果有重复参数，后加载的值覆盖先加载的，但是我们通常在 `/etc/my.cnf` 中配置即可；

```shell
## 方式一
log_bin=mysql-bin
## 配置 server-id
server-id=1

## 方式二：5.7 版本使用该方式
#开启binlog日志
log_bin=ON
#binlog日志的基本文件名
log_bin_basename=mysql-bin
#binlog文件的索引文件，管理所有binlog文件
log_bin_index=mysql-bin.index
#配置serverid
server-id=1
```

然后重启服务器之后即可：

```sql
mysql> show variables like '%log_bin%';
+---------------------------------+---------------------------------------+
| Variable_name                   | Value                                 |
+---------------------------------+---------------------------------------+
| log_bin                         | ON                                    |
| log_bin_basename                | /usr/local/mysql/data/mysql-bin       |
| log_bin_index                   | /usr/local/mysql/data/mysql-bin.index |
| log_bin_trust_function_creators | OFF                                   |
| log_bin_use_v1_row_events       | OFF                                   |
| sql_log_bin                     | ON                                    |
+---------------------------------+---------------------------------------+
6 rows in set (0.00 sec)
```

默认的 binlog 格式为：row

```sql
mysql> show variables like 'binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.00 sec)
```



## 二、binlog 作用

> 即为什么会有 binlog，binlog 可以做什么？







## 三、binlog 直观展示

> 即 binlog 以什么形式保存，保存位置在哪里，怎么查看，文件内部格式是？
>
> 每个语句以什么形式保存，比如 insert 一条语句对应的 binlog 是？

首先查看 binlog 的物理存储形式：

```shell
mysql> show variables like 'datadir';
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| datadir       | /usr/local/mysql/data/ |
+---------------+------------------------+
1 row in set (0.00 sec)

mysql> exit;
Bye
gaojixu@ZBMAC-67b947857 mysql % sudo ls -lh /usr/local/mysql/data 
total 246328
-rw-r-----    1 _mysql  _mysql   2.6K  4  8 18:32 ZBMAC-67b947857.err
-rw-r-----    1 _mysql  _mysql    56B  4  8 17:07 auto.cnf
-rw-r-----    1 _mysql  _mysql   303B  4  8 18:27 ib_buffer_pool
-rw-r-----    1 _mysql  _mysql    48M  4  8 18:35 ib_logfile0
-rw-r-----    1 _mysql  _mysql    48M  4  8 17:07 ib_logfile1
-rw-r-----    1 _mysql  _mysql    12M  4  8 18:35 ibdata1
-rw-r-----    1 _mysql  _mysql    12M  4  8 18:35 ibtmp1
drwxr-x---   77 _mysql  _mysql   2.4K  4  8 17:07 mysql
-rw-r-----    1 _mysql  _mysql   154B  4  8 18:35 mysql-bin.000001
-rw-r-----    1 _mysql  _mysql    19B  4  8 18:35 mysql-bin.index
-rw-r-----    1 _mysql  _mysql   238K  4  8 18:42 mysqld.local.err
-rw-r-----    1 _mysql  _mysql     5B  4  8 18:35 mysqld.local.pid
drwxr-x---   90 _mysql  _mysql   2.8K  4  8 17:07 performance_schema
drwxr-x---  108 _mysql  _mysql   3.4K  4  8 17:07 sys
```

因为我们在 my.cnf 中指定了 binlog 的文件名，所以目前的 binlog 为：mysql-bin.000001，对应的索引文件为：mysql-bin.index。

我们在数据库中创建一张表：

```sql
mysql> create database test;
Query OK, 1 row affected (0.02 sec)

mysql> use test;
Database changed

mysql> CREATE TABLE `t` (
    ->   `pk` int(11) NOT NULL,
    ->   `a` int(11) DEFAULT NULL,
    ->   `b` int(11) DEFAULT NULL,
    ->   `insert_time` datetime NOT NULL,
    ->   `update_time` datetime NOT NULL,   
    ->   PRIMARY KEY (`pk`),
    ->   KEY `a` (`a`)
    -> ) ENGINE=InnoDB;
Query OK, 0 rows affected (0.05 sec)
```

目前 binlog 中信息如下：

```sql
mysql> show binlog events in 'mysql-bin.000001';
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                                                                                                                                                                                                                           |
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc    |       100 |         123 | Server ver: 5.7.18-log, Binlog ver: 4                                                                                                                                                                                                          |
| mysql-bin.000001 | 123 | Previous_gtids |       100 |         154 |                                                                                                                                                                                                                                                |
| mysql-bin.000001 | 154 | Anonymous_Gtid |       100 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 219 | Query          |       100 |         313 | create database test                                                                                                                                                                                                                           |
| mysql-bin.000001 | 313 | Anonymous_Gtid |       100 |         378 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 378 | Query          |       100 |         678 | use `test`; CREATE TABLE `t` (
  `pk` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  `insert_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,   
  PRIMARY KEY (`pk`),
  KEY `a` (`a`)
) ENGINE=InnoDB |
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
6 rows in set (0.01 sec)
```

然后插入一些数据

```sql
mysql> INSERT INTO t VALUES(1,1,1,now(),now()),(2,2,2,now(),now()),(3,3,3,now(),now());
Query OK, 3 rows affected (0.03 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

此时 binlog 信息相比上述增加如下信息：

```sql
mysql> show binlog events in 'mysql-bin.000001';
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                                                                                                                                                                                                                           |
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
--说明：前面相同信息省略
| mysql-bin.000001 | 678 | Anonymous_Gtid |       100 |         743 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 743 | Query          |       100 |         823 | BEGIN                                                                                                                                                                                                                                          |
| mysql-bin.000001 | 823 | Table_map      |       100 |         873 | table_id: 219 (test.t)                                                                                                                                                                                                                         |
| mysql-bin.000001 | 873 | Write_rows     |       100 |         977 | table_id: 219 flags: STMT_END_F                                                                                                                                                                                                                |
| mysql-bin.000001 | 977 | Xid            |       100 |        1008 | COMMIT /* xid=23 */                                                                                                                                                                                                                            |
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
11 rows in set (0.00 sec)
```

然后具体的 SQL 语句需要通过 mysqlbinlog 工具查询，刚刚事务是从  Pos 为 678 开始的；

> 如果 /mysql/data 目录咩有权限，使用  sudo chmod -R a+rwx /usr/local/mysql/data

```shell
gaojixu@ZBMAC-67b947857 mysql % mysqlbinlog -vv /usr/local/mysql/data/mysql-bin.000001 --start-position=678;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#230408 18:35:55 server id 100  end_log_pos 123 CRC32 0xc922bd62 	Start: binlog v 4, server v 5.7.18-log created 230408 18:35:55 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
i0MxZA9kAAAAdwAAAHsAAAABAAQANS43LjE4LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACLQzFkEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AWK9Isk=
'/*!*/;
# at 678
#230408 18:55:24 server id 100  end_log_pos 743 CRC32 0xda72c163 	Anonymous_GTID	last_committed=2	sequence_number=3
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 743
#230408 18:55:24 server id 100  end_log_pos 823 CRC32 0xe1ae7dd7 	Query	thread_id=409	exec_time=0	error_code=0
SET TIMESTAMP=1680951324/*!*/;
SET @@session.pseudo_thread_id=409/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.time_zone='SYSTEM'/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 823
#230408 18:55:24 server id 100  end_log_pos 873 CRC32 0x91f3a149 	Table_map: `test`.`t` mapped to number 219
# at 873
#230408 18:55:24 server id 100  end_log_pos 977 CRC32 0x5a9a91f3 	Write_rows: table id 219 flags: STMT_END_F

BINLOG '
HEgxZBNkAAAAMgAAAGkDAAAAANsAAAAAAAEABHRlc3QAAXQABQMDAxISAgAABkmh85E=
HEgxZB5kAAAAaAAAANEDAAAAANsAAAAAAAEAAgAF/+ABAAAAAQAAAAEAAACZr9Et2Jmv0S3Y4AIA
AAACAAAAAgAAAJmv0S3Yma/RLdjgAwAAAAMAAAADAAAAma/RLdiZr9Et2PORmlo=
'/*!*/;
### INSERT INTO `test`.`t`
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */
###   @3=1 /* INT meta=0 nullable=1 is_null=0 */
###   @4='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @5='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
### INSERT INTO `test`.`t`
### SET
###   @1=2 /* INT meta=0 nullable=0 is_null=0 */
###   @2=2 /* INT meta=0 nullable=1 is_null=0 */
###   @3=2 /* INT meta=0 nullable=1 is_null=0 */
###   @4='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @5='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
### INSERT INTO `test`.`t`
### SET
###   @1=3 /* INT meta=0 nullable=0 is_null=0 */
###   @2=3 /* INT meta=0 nullable=1 is_null=0 */
###   @3=3 /* INT meta=0 nullable=1 is_null=0 */
###   @4='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @5='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
# at 977
#230408 18:55:24 server id 100  end_log_pos 1008 CRC32 0xeff639f9 	Xid = 23
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;

```



### update 语句

执行以下语句：

```sql
mysql> update  t set a = 11,update_time = now() where pk = 1;
```

然后 binlog 为：

```sql
mysql> show binlog events in 'mysql-bin.000001';
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                                                                                                                                                                                                                           |
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| mysql-bin.000001 |   4 | Format_desc    |       100 |         123 | Server ver: 5.7.18-log, Binlog ver: 4                                                                                                                                                                                                          |
| mysql-bin.000001 | 123 | Previous_gtids |       100 |         154 |                                                                                                                                                                                                                                                |
| mysql-bin.000001 | 154 | Anonymous_Gtid |       100 |         219 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 219 | Query          |       100 |         313 | create database test                                                                                                                                                                                                                           |
| mysql-bin.000001 | 313 | Anonymous_Gtid |       100 |         378 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 378 | Query          |       100 |         678 | use `test`; CREATE TABLE `t` (
  `pk` int(11) NOT NULL,
  `a` int(11) DEFAULT NULL,
  `b` int(11) DEFAULT NULL,
  `insert_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,   
  PRIMARY KEY (`pk`),
  KEY `a` (`a`)
) ENGINE=InnoDB |
| mysql-bin.000001 | 678 | Anonymous_Gtid |       100 |         743 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 743 | Query          |       100 |         823 | BEGIN                                                                                                                                                                                                                                          |
| mysql-bin.000001 | 823 | Table_map      |       100 |         873 | table_id: 219 (test.t)                                                                                                                                                                                                                         |
| mysql-bin.000001 | 873 | Write_rows     |       100 |         977 | table_id: 219 flags: STMT_END_F                                                                                                                                                                                                                |
| mysql-bin.000001 | 977 | Xid            |       100 |        1008 | COMMIT /* xid=23 */                                                                                                                                                                                                                            |
+------------------+-----+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
11 rows in set (0.00 sec)

mysql> select * from t;
+----+------+------+---------------------+---------------------+
| pk | a    | b    | insert_time         | update_time         |
+----+------+------+---------------------+---------------------+
|  1 |    1 |    1 | 2023-04-08 18:55:24 | 2023-04-08 18:55:24 |
|  2 |    2 |    2 | 2023-04-08 18:55:24 | 2023-04-08 18:55:24 |
|  3 |    3 |    3 | 2023-04-08 18:55:24 | 2023-04-08 18:55:24 |
+----+------+------+---------------------+---------------------+
3 rows in set (0.00 sec)

mysql> update table t set a = 11,update_time = now() where pk = 1;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'table t set a = 11,update_time = now() where pk = 1' at line 1
mysql> update  t set a = 11,update_time = now() where pk = 1;
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select * from t;
+----+------+------+---------------------+---------------------+
| pk | a    | b    | insert_time         | update_time         |
+----+------+------+---------------------+---------------------+
|  1 |   11 |    1 | 2023-04-08 18:55:24 | 2023-04-08 19:19:06 |
|  2 |    2 |    2 | 2023-04-08 18:55:24 | 2023-04-08 18:55:24 |
|  3 |    3 |    3 | 2023-04-08 18:55:24 | 2023-04-08 18:55:24 |
+----+------+------+---------------------+---------------------+
3 rows in set (0.00 sec)

mysql> show binlog events in 'mysql-bin.000001';
+------------------+------+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Log_name         | Pos  | Event_type     | Server_id | End_log_pos | Info                                                                                                                                                                                                                                           |
+------------------+------+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
### 省略上面已有信息

| mysql-bin.000001 | 1008 | Anonymous_Gtid |       100 |        1073 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                                                                                           |
| mysql-bin.000001 | 1073 | Query          |       100 |        1153 | BEGIN                                                                                                                                                                                                                                          |
| mysql-bin.000001 | 1153 | Table_map      |       100 |        1203 | table_id: 219 (test.t)                                                                                                                                                                                                                         |
| mysql-bin.000001 | 1203 | Update_rows    |       100 |        1285 | table_id: 219 flags: STMT_END_F                                                                                                                                                                                                                |
| mysql-bin.000001 | 1285 | Xid            |       100 |        1316 | COMMIT /* xid=36 */                                                                                                                                                                                                                            |
+------------------+------+----------------+-----------+-------------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

然后

```sql
gaojixu@ZBMAC-67b947857 Desktop % mysqlbinlog -vv /usr/local/mysql/data/mysql-bin.000001 --start-position=1008;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#230408 18:35:55 server id 100  end_log_pos 123 CRC32 0xc922bd62 	Start: binlog v 4, server v 5.7.18-log created 230408 18:35:55 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
i0MxZA9kAAAAdwAAAHsAAAABAAQANS43LjE4LWxvZwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAACLQzFkEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
AWK9Isk=
'/*!*/;
# at 1008
#230408 19:19:06 server id 100  end_log_pos 1073 CRC32 0x8756c76d 	Anonymous_GTID	last_committed=3	sequence_number=4
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
# at 1073
#230408 19:19:06 server id 100  end_log_pos 1153 CRC32 0x7432771e 	Query	thread_id=1247exec_time=0	error_code=0
SET TIMESTAMP=1680952746/*!*/;
SET @@session.pseudo_thread_id=1247/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
SET @@session.time_zone='SYSTEM'/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;
BEGIN
/*!*/;
# at 1153
#230408 19:19:06 server id 100  end_log_pos 1203 CRC32 0xfa4b3e61 	Table_map: `test`.`t` mapped to number 219
# at 1203
#230408 19:19:06 server id 100  end_log_pos 1285 CRC32 0xe449b014 	Update_rows: table id 219 flags: STMT_END_F

BINLOG '
qk0xZBNkAAAAMgAAALMEAAAAANsAAAAAAAEABHRlc3QAAXQABQMDAxISAgAABmE+S/o=
qk0xZB9kAAAAUgAAAAUFAAAAANsAAAAAAAEAAgAF///gAQAAAAEAAAABAAAAma/RLdiZr9Et2OAB
AAAACwAAAAEAAACZr9Et2Jmv0TTGFLBJ5A==
'/*!*/;
### UPDATE `test`.`t`
### WHERE
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=1 /* INT meta=0 nullable=1 is_null=0 */
###   @3=1 /* INT meta=0 nullable=1 is_null=0 */
###   @4='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @5='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
### SET
###   @1=1 /* INT meta=0 nullable=0 is_null=0 */
###   @2=11 /* INT meta=0 nullable=1 is_null=0 */
###   @3=1 /* INT meta=0 nullable=1 is_null=0 */
###   @4='2023-04-08 18:55:24' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
###   @5='2023-04-08 19:19:06' /* DATETIME(0) meta=0 nullable=0 is_null=0 */
# at 1285
#230408 19:19:06 server id 100  end_log_pos 1316 CRC32 0xf4fed9c4 	Xid = 36
COMMIT/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```









## 四、处理 binlog 工具

> 即 binlog 工具，这里可以使用 https://github.com/alibaba/canal

binlog 通常用于 MySQL 的主从同步，同样可以用于数据库的镜像或者数据库实时备份。而 binlog 作为二进制文件，直接解析较为困难，因此通常使用工具，这里以 alibaba 的 canal 为例；

> 相关文章：https://www.cnblogs.com/throwable/p/12483983.html
>
> https://cloud.tencent.com/developer/article/1700339、
>
> https://juejin.cn/post/6993860875551522847

### 环境搭建与使用

因为 canal 本质是将自身作为 slave 接受 master 的 binlog，所以需要两个步骤：

- mysql binlog 必须开始，同时格式为 ROW 格式；

    ```sql
    [mysqld]
    log-bin=mysql-bin # 开启 binlog
    binlog-format=ROW # 选择 ROW 模式
    server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
    ```

- 需要授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限；

    ```sql
    CREATE USER canal IDENTIFIED BY 'canal';  
    GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
    -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
    FLUSH PRIVILEGES;
    ```

### 环境部署

下载：https://github.com/alibaba/canal/releases  选择 deployer 版本

然后将下载的压缩包解压之后，修改其中的 conf/example/instance.properties  

```properties
## mysql serverId
canal.instance.mysql.slaveId = 1234
#position info，需要改成自己的数据库信息
canal.instance.master.address = 127.0.0.1:3306 
canal.instance.master.journal.name = 
canal.instance.master.position = 
canal.instance.master.timestamp = 
#canal.instance.standby.address = 
#canal.instance.standby.journal.name =
#canal.instance.standby.position = 
#canal.instance.standby.timestamp = 
#username/password，需要改成自己的数据库信息
canal.instance.dbUsername = canal  
canal.instance.dbPassword = canal
canal.instance.defaultDatabaseName =
# canal.instance.connectionCharset 代表数据库的编码方式对应到 java 中的编码类型，比如 UTF-8，GBK , ISO-8859-1
canal.instance.connectionCharset = UTF-8
#table regex
canal.instance.filter.regex = .\*\\\\..\*
```

注意：如果系统是 1 个 cpu，需要将 canal.instance.parser.parallel 设置为 false

- 启动

    ```shell
    sh bin/startup.sh
    ```

- 查看 server 日志

    ```
    vi logs/canal/canal.log
    ```

    ```
    2013-02-05 22:45:27.967 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## start the canal server.
    2013-02-05 22:45:28.113 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[10.1.29.120:11111]
    2013-02-05 22:45:28.210 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## the canal server is running now ......
    ```

- 查看 instance 的日志

    ```
    vi logs/example/example.log
    ```

    ```
    2013-02-05 22:50:45.636 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [canal.properties]
    2013-02-05 22:50:45.641 [main] INFO  c.a.o.c.i.spring.support.PropertyPlaceholderConfigurer - Loading properties file from class path resource [example/instance.properties]
    2013-02-05 22:50:45.803 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start CannalInstance for 1-example 
    2013-02-05 22:50:45.810 [main] INFO  c.a.otter.canal.instance.spring.CanalInstanceWithSpring - start successful....
    ```

- 关闭

    ```
    sh bin/stop.sh
    ```



#### 使用方式

- canal 的 docker 模式快速启动，参考：[Docker QuickStart](https://github.com/alibaba/canal/wiki/Docker-QuickStart)
- canal 链接 Aliyun RDS for MySQL，参考：[Aliyun RDS QuickStart](https://github.com/alibaba/canal/wiki/aliyun-RDS-QuickStart)
- canal 消息投递给 kafka/RocketMQ，参考：[Canal-Kafka-RocketMQ-QuickStart](https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart)



### 客户端

