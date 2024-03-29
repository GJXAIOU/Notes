---
title: USE
summary: TiDB 数据库中 USE 的使用概况。
---

# USE

`USE` 语句可为用户会话选择当前数据库。

## 语法图

**UseStmt:**

![UseStmt](/media/sqlgram/UseStmt.png)

**DBName:**

![DBName](/media/sqlgram/DBName.png)

## 示例

{{< copyable "sql" >}}

```sql
USE mysql;
```

```
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
```

{{< copyable "sql" >}}

```sql
SHOW TABLES;
```

```
+-------------------------+
| Tables_in_mysql         |
+-------------------------+
| GLOBAL_VARIABLES        |
| bind_info               |
| columns_priv            |
| db                      |
| default_roles           |
| expr_pushdown_blacklist |
| gc_delete_range         |
| gc_delete_range_done    |
| global_priv             |
| help_topic              |
| opt_rule_blacklist      |
| role_edges              |
| stats_buckets           |
| stats_feedback          |
| stats_histograms        |
| stats_meta              |
| stats_top_n             |
| tables_priv             |
| tidb                    |
| user                    |
+-------------------------+
20 rows in set (0.01 sec)
```

{{< copyable "sql" >}}

```sql
CREATE DATABASE newtest;
```

```
Query OK, 0 rows affected (0.10 sec)
```

{{< copyable "sql" >}}

```sql
USE newtest;
```

```
Database changed
```

{{< copyable "sql" >}}

```sql
SHOW TABLES;
```

```
Empty set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
CREATE TABLE t1 (a int);
```

```
Query OK, 0 rows affected (0.10 sec)
```

{{< copyable "sql" >}}

```sql
SHOW TABLES;
```

```
+-------------------+
| Tables_in_newtest |
+-------------------+
| t1                |
+-------------------+
1 row in set (0.00 sec)
```

## MySQL 兼容性

`USE` 语句与 MySQL 完全兼容。如发现任何兼容性差异，请在 GitHub 上提交 [issue](https://github.com/pingcap/tidb/issues/new/choose)。

## 另请参阅

* [CREATE DATABASE](/sql-statements/sql-statement-create-database.md)
* [SHOW TABLES](/sql-statements/sql-statement-show-tables.md)
