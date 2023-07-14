## **MySQL5.5直接执行锁表操作**

**MySQL5.5不支持online ddl，只支持\**fast index creation特性，所有对表结构更改的操作都会锁表影响读写，只有辅助索引的创建和删除只阻塞写不阻塞读\****

## **MySQL5.6直接执行锁表操作（红色）**

| 类型         | Operation                                          | In Place | Rebuilds Table | Permits  Concurrent DML | Only Modifies  Metadata |
| ------------ | -------------------------------------------------- | -------- | -------------- | ----------------------- | ----------------------- |
| 索引操作     | Creating or adding a secondary index               | Yes      | No             | Yes                     | No                      |
| 索引操作     | Dropping an index                                  | Yes      | No             | Yes                     | Yes                     |
| **索引操作** | Adding a FULLTEXT index                            | Yes*     | No*            | No                      | No                      |
| 索引操作     | Changing the index type                            | Yes      | No             | Yes                     | Yes                     |
| 主键操作     | Adding a primary key                               | Yes*     | Yes*           | Yes                     | No                      |
| **主键操作** | Dropping a primary key                             | No       | Yes            | No                      | No                      |
| 主键操作     | Dropping a primary key and adding  another         | Yes      | Yes            | Yes                     | No                      |
| 列操作       | Adding a column                                    | Yes      | Yes            | Yes*                    | No                      |
| 列操作       | Dropping a column                                  | Yes      | Yes            | Yes                     | No                      |
| 列操作       | Renaming a column                                  | Yes      | No             | Yes*                    | Yes                     |
| 列操作       | Reordering columns                                 | Yes      | Yes            | Yes                     | No                      |
| 列操作       | Setting a column default value                     | Yes      | No             | Yes                     | Yes                     |
| **列操作**   | Changing the column data type                      | No       | Yes            | No                      | No                      |
| 列操作       | Dropping the column default value                  | Yes      | No             | Yes                     | Yes                     |
| 列操作       | Changing the auto-increment value                  | Yes      | No             | Yes                     | No*                     |
| 列操作       | Making a column NULL                               | Yes      | Yes*           | Yes                     | No                      |
| 列操作       | Making a column NOT NULL                           | Yes*     | Yes*           | Yes                     | No                      |
| 列操作       | Modifying the definition of  an ENUM or SET column | Yes      | No             | Yes                     | Yes                     |
| 外键操作     | Adding a foreign key constraint                    | Yes*     | No             | Yes                     | Yes                     |
| 外键操作     | Dropping a foreign key constraint                  | Yes      | No             | Yes                     | Yes                     |
| 表操作       | Changing the ROW_FORMAT                            | Yes      | Yes            | Yes                     | No                      |
| 表操作       | Changing the KEY_BLOCK_SIZE                        | Yes      | Yes            | Yes                     | No                      |
| 表操作       | Setting persistent table statistics                | Yes      | No             | Yes                     | Yes                     |
| **表操作**   | Specifying a character set                         | Yes      | Yes*           | No                      | No                      |
| **表操作**   | Converting a character set                         | No       | Yes            | No                      | No                      |
| 表操作       | Optimizing a table                                 | Yes*     | Yes            | Yes                     | No                      |
| 表操作       | Rebuilding with the FORCEoption                    | Yes*     | Yes            | Yes                     | No                      |
| 表操作       | Performing a null rebuild                          | Yes*     | Yes            | Yes                     | No                      |
| 表操作       | Renaming a table                                   | Yes      | No             | Yes                     | Yes                     |

## **MySQL5.7直接执行锁表操作（红色）**

| 类型           | Operation                                                   | In Place | Rebuilds Table | Permits  Concurrent DML | Only Modifies  Metadata |
| -------------- | ----------------------------------------------------------- | -------- | -------------- | ----------------------- | ----------------------- |
| 索引操作       | Creating or adding a secondary index                        | Yes      | No             | Yes                     | No                      |
| 索引操作       | Dropping an index                                           | Yes      | No             | Yes                     | Yes                     |
| 索引操作       | Renaming an index                                           | Yes      | No             | Yes                     | Yes                     |
| **索引操作**   | Adding a FULLTEXT index                                     | Yes*     | No*            | No                      | No                      |
| **索引操作**   | Adding a SPATIAL index                                      | Yes      | No             | No                      | No                      |
| 索引操作       | Changing the index type                                     | Yes      | No             | Yes                     | Yes                     |
| 主键操作       | Adding a primary key                                        | Yes*     | Yes*           | Yes                     | No                      |
| **主键操作**   | Dropping a primary key                                      | No       | Yes            | No                      | No                      |
| 主键操作       | Dropping a primary key and adding  another                  | Yes      | Yes            | Yes                     | No                      |
| 列操作         | Adding a column                                             | Yes      | Yes            | Yes*                    | No                      |
| 列操作         | Dropping a column                                           | Yes      | Yes            | Yes                     | No                      |
| 列操作         | Renaming a column                                           | Yes      | No             | Yes*                    | Yes                     |
| 列操作         | Reordering columns                                          | Yes      | Yes            | Yes                     | No                      |
| 列操作         | Setting a column default value                              | Yes      | No             | Yes                     | Yes                     |
| **列操作**     | Changing the column data type                               | No       | Yes            | No                      | No                      |
| 列操作         | Extending VARCHARcolumn size                                | Yes      | No             | Yes                     | Yes                     |
| 列操作         | Dropping the column default value                           | Yes      | No             | Yes                     | Yes                     |
| 列操作         | Changing the auto-increment value                           | Yes      | No             | Yes                     | No*                     |
| 列操作         | Making a column NULL                                        | Yes      | Yes*           | Yes                     | No                      |
| 列操作         | Making a column NOT NULL                                    | Yes*     | Yes*           | Yes                     | No                      |
| 列操作         | Modifying the definition of  an ENUM or SET column          | Yes      | No             | Yes                     | Yes                     |
| **生成列操作** | Adding a STORED column                                      | No       | Yes            | No                      | No                      |
| **生成列操作** | Modifying STORED column order                               | No       | Yes            | No                      | No                      |
| 生成列操作     | Dropping a STOREDcolumn                                     | Yes      | Yes            | Yes                     | No                      |
| 生成列操作     | Adding a VIRTUAL column                                     | Yes      | No             | Yes                     | Yes                     |
| 生成列操作     | Modifying VIRTUALcolumn order                               | No       | Yes            | No                      | No                      |
| 生成列操作     | Dropping a VIRTUALcolumn                                    | Yes      | No             | Yes                     | Yes                     |
| 外键操作       | Adding a foreign key constraint                             | Yes*     | No             | Yes                     | Yes                     |
| 外键操作       | Dropping a foreign key constraint                           | Yes      | No             | Yes                     | Yes                     |
| 表操作         | Changing the ROW_FORMAT                                     | Yes      | Yes            | Yes                     | No                      |
| 表操作         | Changing the KEY_BLOCK_SIZE                                 | Yes      | Yes            | Yes                     | No                      |
| 表操作         | Setting persistent table statistics                         | Yes      | No             | Yes                     | Yes                     |
| **表操作**     | Specifying a character set                                  | Yes      | Yes*           | No                      | No                      |
| **表操作**     | Converting a character set                                  | No       | Yes*           | No                      | No                      |
| 表操作         | Optimizing a table                                          | Yes*     | Yes            | Yes                     | No                      |
| 表操作         | Rebuilding with the FORCEoption                             | Yes*     | Yes            | Yes                     | No                      |
| 表操作         | Performing a null rebuild                                   | Yes*     | Yes            | Yes                     | No                      |
| 表操作         | Renaming a table                                            | Yes      | No             | Yes                     | Yes                     |
| **表空间操作** | Enabling or disabling file-per-table  tablespace encryption | No       | Yes            | No                      | No                      |