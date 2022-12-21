1、常用数据类型映射表：

| MySQL            | JDBCType                 | JavaType                       | 备注                                                         |
| :--------------- | :----------------------- | :----------------------------- | :----------------------------------------------------------- |
| char             | CHAR                     | String                         | 定长字符,无符号[0,255]，有符号[-128,127]                     |
| varchar          | VARCHAR                  | String                         | 变长字符串,整数[0,255]                                       |
| tinyint          | TINYINT                  | Byte                           | 1字节                                                        |
| tinyint unsigned | TINYINT                  | Byte                           |                                                              |
| smallint         | SMALLINT                 | short                          | 2字节,无符号[0,65535]，有符号[-32768,32767]                  |
| int              | INTEGER                  | Integer                        | 4字节,无符号[0,2^32-1]，有符号[-2^31,2^31-1]                 |
| int unsigned     | integer                  | integer                        |                                                              |
| float            | FLOAT                    | float                          | 4字节,Float(M,D) 单精度浮点数。这里的D是精度，如果D<=24则为默认的FLOAT，如果D>24则会自动被转换为DOUBLE型。 |
| bigint           | BIGINT                   | Long                           | 8字节,无符号[0,2^64-1]，有符号[-2^63 ,2^63 -1]               |
| bigint unsigned  | bigint                   | Long                           |                                                              |
| double           | DOUBLE                   | double                         | 8字节,double(M,D)                                            |
| double unsigned  | double                   | double                         |                                                              |
| bit              | BOOLEAN(Bit，这里有争议) | boolean（integer，这里有争议） | 布尔类型                                                     |
| BLOB             | LONGVARBINARY            | byte[]                         |                                                              |
| decimal          | decimal                  | BigDecimal                     |                                                              |
| decimal unsigned | decimal                  | BigDecimal                     |                                                              |
| JSON             | LONGVARCHAR              | String                         |                                                              |
| TEXT             | LONGVARCHAR              | String                         |                                                              |
| LONGBLOB         | LONGVARBINARY            | byte[]                         |                                                              |
| LONGTEXT         | LONGVARCHAR              | String                         |                                                              |
| TINYText         | VARCHAR                  | String                         |                                                              |



2、日期时间和大对象映射表。

| MySQL     | JDBCType    | JavaType            | 备注                     |
| :-------- | :---------- | :------------------ | :----------------------- |
| date      | DATE        | util.Date、sql.Date | YYYY-MM-DD               |
| time      | TIME        | util.Date、sql.Date | HH:MM:SS                 |
| timestamp | TIMESTAMP   | util.Date、sql.Date | YYYY-MM-DD HH:MM:SS      |
| text      | VARCHAR     | String              | 文本类型【2^16-1字节】   |
| longtext  | LONGVARCHAR | String              | 长文本类型【2^32-1字节】 |