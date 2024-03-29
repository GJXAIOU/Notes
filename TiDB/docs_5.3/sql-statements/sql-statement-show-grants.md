---
title: SHOW GRANTS
summary: TiDB 数据库中 SHOW GRANTS 的使用概况。
---

# SHOW GRANTS

`SHOW GRANTS` 语句用于显示与用户关联的权限列表。与在 MySQL 中一样，`USAGE` 权限表示登录 TiDB 的能力。

## 语法图

**ShowGrantsStmt:**

![ShowGrantsStmt](/media/sqlgram/ShowGrantsStmt.png)

**Username:**

![Username](/media/sqlgram/Username.png)

**UsingRoles:**

![UsingRoles](/media/sqlgram/UsingRoles.png)

**RolenameList:**

![RolenameList](/media/sqlgram/RolenameList.png)

**Rolename:**

![Rolename](/media/sqlgram/Rolename.png)

## 示例

{{< copyable "sql" >}}

```sql
SHOW GRANTS;
```

```
+-------------------------------------------+
| Grants for User                           |
+-------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' |
+-------------------------------------------+
1 row in set (0.00 sec)
```

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR 'u1';
```

```
ERROR 1141 (42000): There is no such grant defined for user 'u1' on host '%'
```

{{< copyable "sql" >}}

```sql
CREATE USER u1;
```

```
Query OK, 1 row affected (0.04 sec)
```

{{< copyable "sql" >}}

```sql
GRANT SELECT ON test.* TO u1;
```

```
Query OK, 0 rows affected (0.04 sec)
```

{{< copyable "sql" >}}

```sql
SHOW GRANTS FOR u1;
```

```
+------------------------------------+
| Grants for u1@%                    |
+------------------------------------+
| GRANT USAGE ON *.* TO 'u1'@'%'     |
| GRANT Select ON test.* TO 'u1'@'%' |
+------------------------------------+
2 rows in set (0.00 sec)
```

## MySQL 兼容性

`SHOW GRANTS` 语句与 MySQL 完全兼容。如发现任何兼容性差异，请在 GitHub 上提交 [issue](https://github.com/pingcap/tidb/issues/new/choose)。

## 另请参阅

* [SHOW CREATE USER](/sql-statements/sql-statement-show-create-user.md)
* [GRANT](/sql-statements/sql-statement-grant-privileges.md)
