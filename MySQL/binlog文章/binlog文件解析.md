**目录**

**1、** **什么是****binlog****？**

**2、** **如何开启****binlog****？**

**3、** **什么是二进制格式的文件？**

**4、** **binlog****的三种模式？**

**5、** **binlog****文件里面写入的是啥玩意？**

**6、** **常见的****event****事件？**

**7、** **解析****binlog****后的每个事务的第一个****timestamp****。**

**8、** **MySQL****是如何写入****binlog****的？**

**9、** **相关问题？**

## 一、什么是binlog？

答：1.binlog 是一个二进制格式的文件，用于记录用户对数据库更新的 SQL 语句信息，例如更改数据库表和更改内容的 SQL 语句都会记录到 binlog 里，但是对库表等内容的查询不会记录。binlog 是 mysql 本身提供的一种逻辑日志，和具体的存储引擎无关，但是不同的存储引擎对 binlog 写入的模式有要求。

2、作用：当有数据写入到数据库时，会同时把更新的 SQL 语句写入到对应的 binlog 文件中，主要作用是用于数据库的主从复制及数据的增量恢复。比如使用 mysqldump 或者 xtrabackup 进行一次全量备份后，运行一段时间数据库出现故障，那么就可以通过 binlog 来进行数据恢复了。

## 二、如何开启binlog？

show variables like ‘log_bin’，可以看到有没有开启 binlog，而且可以在配置文件中配置 binlog 文件的大小，保留天数等等。

**问题：** binlog 文件的大小是一样的吗？

答：正常情况下，线上 binlog 文件的大小默认都是 1G，但是当写入最后一个事务的时候这个事务数据量比较大，则 binlog 会记录这一完整的事务导致 binlog 文件大于 1G。

## 三、什么是二进制格式的文件？

计算机文件基本上分为二种：二进制文件和 ASCII（utf-8 等字符编码）文件（也称纯文本文件），图形文件及文字处理程序等计算机程序都属于二进制文件。这些文件含有特殊的格式及计算机代码。ASCII 则是可以用任何文字处理程序阅读的简单文本文件。

使用二进制文件的好处：

- 第一是二进制文件比较节约空间，这两者储存字符型数据时并没有差别。但是在储存数字，特别是实型数字（小数）时，二进制更节省空间，比如储存数据：3.1415927，文本文件需要 9 个字节，分别储存：3 . 1 4 1 5 9 2 7 这 9 个 ASCII 值，而二进制文件只需要 4 个字节（DB 0F 49 40）（eg：golang 的 float32 类型只占 4 个字节）。

- 第二个原因是，内存中参加计算的数据都是用二进制无格式储存起来的，因此，使用二进制储存到文件就更快捷。如果储存为文本文件，则需要一个转换的过程。在数据量很大的时候，两者就会有明显的速度差别了。

## 四、记录 binlog 的三种模式？

### （一）STATEMENT（SBR，statement-based replication）模式：

记录每条会修改数据的 sql 语句；为了这些语句能在 slave 上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在 slave 得到和在 master 端执行时候相同的结果。

存在问题：

- 像一些特定函数功能，在 MySQL 主从复制时，slave 要与 master 上保持一致会有很多相关问题，如 sql 中包含 now(), last_insert_id()、UUID()等函数时。

- 在执行 insert … select …语句时会产生大量行锁。

### （二）ROW(RBR，row-based replication)模式：

不记录相关的 sql 语句，仅保存哪条记录被修改了，也就是说日志中会记录成每行数据被修改的形式。会详细记录每行数据的每个字段对应的值内容。

两个参数：（mysql 5.6.2 之后引入）

- binlog_rows_query_log_events：

    在 row 模式下，开启该参数，将把 sql 语句打印到 binlog 日志里面。默认是 0(off)；虽然将语句放入了 binlog，但不会执行这个 sql，就相当于注释一样。但对于 dba 来说,在查看 binlog 的时候,很有用处。

- binlog_row_image：

    默认为 full，在 binlog 为 row 格式下，full 模式将记录 update 前后所有字段的值；minimal 时只记录更改字段的值和 where 字段的值；noblob 时记录除了 blob 和 text 的所有字段的值。如果 update 的 blob 或 text 字段,也只记录该字段更改后的值,更改前的不记录。这样做的目的是控制 binlog 写入的量。

缺点是开启 row 格式后，生成 binlog 文件的内容太多，比如 alter 表时，所有修改的记录都会被写进 binlog 文件中。

### （三）MIXED(MBR)混合模式：

对于绝大部分操作，都使用 STATEMENT 来进行 binlog 的记录，只有以下操作使用 ROW 来实现：

- 表的存储引擎为 NDB；
- 使用了 uuid() 等不确定函数；
- 使用了 insert delay 语句；
- 2 个及以上包含 AUTO_INCREMENT 字段的表被更新时；
- 使用了临时表（比如 insert … select 语句）。

注：在 ROW 格式下，使用 update 语句时如果对行数据没有影响是不会记录 binlog 中的，但是在 MIXED 和 STATEMENT 模式下还是会将语句记录 binlog 中的（update test set name=’666’ where name=’666’，即使 where 条件没有匹配的行）。

## 五、binlog文件里面写入的是啥玩意？

- binlog 是由一个一个 event 组成，event 是 binlog 的最小组成单元。

- binlog 文件头部固定以 4 个字节开头，这四个字节称为 BINLOG_MAGIC(fe 62 69 6e) 魔数，当使用 `mysqldump` 命令解析 binlog 文件时，来识别该文件属于 binlog 文件。

- 每个 binlog 文件以一个 `FORMAT_DESCRIPTION_EVENT` 类型的 event 开始。以一个 Rotate 类型的 event 结束（但也有特殊情况，当数据库出现宕机的情况，重新启动数据库会生成一个新的 binlog 文件，但是宕机之前的最新 binlog 文件中，不是以 ROTATE_EVENT 结束的）

    在 `FORMAT_DESCRIPTION_EVENT` 和 `ROTATE_EVENT` 之间是各种不同 event，每个 event 代表 Master 上不同的操作。

- 查看 binlog 的命令和 event 的命令以及 binlog 记录位点：

    ```sql
    show binlog events in ‘mysql-bin.000111’(from position); 
    show master status; 
    show binary logs; 
    ```

## 六、常见的event事件

### （一）binlog事件

由 `<公有事件头>+[私有事件头post header]+[事件体event body]` 三部分组成。所有的 binlog 事件都包含公有事件头，另外两个部分是根据事件类型可选。

公有事件头内容（事件头根据 binlog 版本不同，分为 13 字节和 19 字节。mysql5+ 版本都是 binlog v4 版本，都是 19 个字节。）：

![image-20230409215706181](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409215706181-1048628.png)                        

- `timestamp (4)` -- seconds since unix epoch：第一个字段是事件生成的时间戳，占 4 个字节。

- `event_type(1)` -- see Binlog Event Type：第二个字段是事件类型，占 1 个字节。

- `server_id (4)` -- server-id of the originating mysql-server. Used to filter out events in circular replication：第三个字段是源 mysql 主机的 server id，占 4 个字节，用于过滤循环复制中的事件。

- `event_size (4)` -- size of the event (header, post-header, body)：第四个字段是事件大小，占 4 个字节，包括事件头、事件体的总大小。

- `log_pos (4)` -- position of the next event：第五个字段是位置，占 4 个字节，表示下一个事件的位置。

- `flags (2)` -- see Binlog Event Flag：最后一个字段标志状态，占 2 个字节。https://dev.mysql.com/doc/internals/en/binlog-event-flag.html

### （二）binlog 版本问题

binlog 文件格式有以下几种：

- v1：用于 3.23 版本；
- v2：版本只在 4.0.x 版本中使用，目前已经不再支持了。

- v3：用于 4.0.2 到 4.1 版本；

- v4：用于 5.0 及以上版本；

### （三）mysql5.7 关闭 gtid 之后的 binlog event 类型：

ddl create table：Anonymous_Gtid event --> Query event --> XID event。

insert/update/delete：Anonymous_Gtid event --> Query event--> Table_map event --> Write_rows event（Delete_rows 或者 Update_rows）-> XID event

注：**binlog 事件的产生是以语句为单位而不是以数据行为单位，即一条语句会产生一个事件而不是每操作一行数据会有一个事件**。

![image-20230409220624883](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409220624883.png)

## 七、binlog事件介绍

### （一）FORMAT_DESCRIPTION_EVENT

A format description event is the first event of a binlog for binlog-version 4. It describes how the other events are layed out. 

这个事件定义了 binlog header 的格式，而且记录了一些元数据信息。

 ![image-20230409221037177](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409221037177.png)

- `binlog-version (2)` ：binlog 文件版本号，2 个字节。

- `mysql-server version (50)`：创建 binlog 的 MySQL 服务器版本，50 个字节。
- `create_timestamp (4)`：binlog 创建时的时间戳，4 个字节。

- `event_header_length (1)`：binlog header 占用的字节数 19，1 个字节。

- `event type header length(39)`：这里的是每个事件的 post header 的长度的字节数组，每一个事件一个字节。39 个事件就有 39 个字节。这个数组长度不是固定的(每个版本包含的事件数很可能是不同的)。

- `CRC32(4)`: CRC32 校验位，4 个字节。

### （二）Rows_query event

当 `binlog_rows_query_log_events` 这个参数开启时，会将执行的源 sql 语句存入这个事件中，但是在主从复制的过程中，从库并不会去执行这个事件。

### （三）Query event

The query event is used to send text querys right the binlog. 查询事件用于向 binlog 发送文本查询。

![image-20230409221859575](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409221859575.png) 

#### 1、触发条件

QUERY_EVENT 类型的事件通常在以下几种情况下使用：

- 事务开始时，执行的 BEGIN 操作。

- STATEMENT 格式中的 DML 操作。

- ROW 格式中的 DDL 操作（如 create table）。

![image-20230409222233011](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409222233011.png)

#### 2.固定数据部分(私有事件头)

- `thread_id`：4 个字节，存储了不同连接或会话的线程 ID。

- `execution time`：4 个字节，查询从开始执行到记录到 binlog 所花时间，单位 s。

- `schema length`：1 个字节，schema 字符长度 6。即库名的长度。

- `error code`：2 个字节，错误码。

- `status-var length`：2 个字节，23 00，status-var 的长度，为 35

#### 3.可变数据部分（事件体）--------

- `status var`:35 个字节，即以 KV 对的形式保存起来的一些了由 SET 命令设置的上下文信息。eg：

    ```sql
    SET @@session.pseudo_thread_id=7322991/*!*/;
    
    SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
    
    SET @@session.sql_mode=0/*!*/;
    
    SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
    
    /*!\C utf8 *//*!*/;
    
    SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=33/*!*/;
    
    SET @@session.lc_time_names=0/*!*/;
    
    SET @@session.collation_database=DEFAULT/*!*/;
    ```

下面详细说一下这几个参数的含义：

- 会话系统变量 `pseudo_thread_id` 用于标记当前会话的 mysql 连接 ID。

- `SET @@session.foreign_key_checks=1` 开启外键约束，=0 是取消外键约束。

- `sql_auto_is_null：sql_auto_is_null` 参数默认值为 OFF 关闭 0，如果启用了此变量，则在成功插入自动生成的 AUTO_INCREMENT 值的语句之后 ，可以通过发出以下形式的语句来找到该值：`SELECT * FROM tbl_name WHERE auto_col IS NULL`。

- `@@session.unique_checks=1`，这个参数是用于检查主键和唯一键是否重复的参数.

- `@@session.autocommit=1`，MySQL 从自动提交（autocommit）模式运行，这种模式会在每条语句执行完毕后把它作出的修改立刻提交给数据库并使之永久化。

- `@@session.sql_mode=1073741824`，通过设置 sql mode, 可以完成不同严格程度的数据校验，有效地保障数据准备性。例如：我在创建一个表时，该表中有一个字段为 name，给 name 设置的字段类型时 char(10)，如果我在插入数据的时候，其中 name 这个字段对应的有一条数据的长度超过了 10，例如'1234567890abc'，超过了设定的字段长度 10，那么不会报错，并且取前十个字符存上，也就是说你这个数据被存为了'1234567890',而'abc'就没有了。

- `auto_increment_offset` 表示自增长字段从那个数开始，他的取值范围是 1 .. 65535，`auto_increment_increment` 表示自增长字段每次递增的量，其默认值是 1，取值范围是 1 .. 65535。

- `SET @@session.lc_time_name=0`，日期的星期和月份名称，默认是英文的，`set lc_time_names='zh_CN'`，则可以设置为中文的。

- `SET @@session.character_set_client=45`，id 为 45 的字符集为 utf8mb4 字符集，33 为 utf8.客户端发送到 server 的字符串使用的字符集，server 会按照该变量值来解析客户端发来的语句。

    参考：http://mysql.taobao.org/monthly/2018/01/07/

    https://blog.csdn.net/shilukun/article/details/85230599

- `@@session.collation_connection=45`，45 对应 utf8mb4_general_ci，字符集校对规则：是在字符集内用于字符比较和排序的一套规则，比如有的规则区分大小写，有的则无视。

    - 两个不同的字符集不能有相同的校对规则；
    - 每个字符集有一个默认校对规则；
    - 存在校对规则命名约定：以其相关的字符集名开始，中间包括一个语言名，并且以`_ci`（大小写不敏感）、`_cs`（大小写敏感）或 `_bin`（二进制形式，也是区分大小写）结束。

    character_set_connection/collation_connection：没有指定字符集的常量字符串使用时的字符集。

    mysql 的 collation 作用于 3 个层面，并且下级遵循上级的配置分为

    数据库的 collation、表的 collation、字段的 collation

    原则：修改数据库的 collation，对修改后新建的表才会生效，已存在表不生效，修改表的 collation，对修改后新建的字段才会生效，已存在字段不生效，修改字段的 collation，对修改后新写入生效，同时对已存在的也生效。

    ```sql
    ALTER DATABASE `basename` CHARACTER SET utf8COLLATE utf8_bin;
    
    ALTER TABLE `basename`.`tablename` COLLATE=utf8_bin;
    
    ALTER TABLE `tablename` MODIFY COLUMN `name` varchar(8) CHARACTER SET utf8 COLLATE utf8_bin;
    ```

    同时也可以在 my.cnf 中修改 `collation_server = utf8_bin` (重启生效)。（该参数不支持动态修改。）

    系统使用 utf8 字符集，若使用 utf8_bin 校对规则执行 SQL 查询时区分大小写，使用 utf8_general_ci 不区分大小写(默认的 utf8 字符集对应的校对规则是 utf8_general_ci)。

    校对规则是在字符集内用于比较字符的一套规则,可以控制 select 查询时 where 条件大小写是否敏感的规则.如字段 col 在表中的值为 'abc','ABC','AbC' 在不同的校对规则下,where col='ABC'会有不同的结果。

    每个列都应该有一个校对，如果没有显示指定，MySQL 就使用属于该字符集的默认校对。如果指定了一个字符集和一个校对，字符集应该放在前面。

- mysqlm 字符集转换过程：

    MySQL 服务响应客户端操作的字符的字符集和客户端信息处理过程:

    客户端发出的 SQL 语句，所使用的字符集由系统变量 character_set_client 来指定

    MySQL 服务端接收语句后，会用 character_set_connection 和 collation_connection 两个系统变量中的设置，并且会将客户端发送的语句字符集由 character_set_client 转到 character_set_connection(除非用户执行语句时，已经对字符列明确指定了字符集）。对于语句中指定的字符串比较或排序，还需要应用 collation_connection 中指定的校对规则处理，而对于语句中指定的列的比较则无关 collation_connection 的设置了，因为对象的列表拥有自己的校对规则，他们拥有更高的优先级。

    MySQL 服务执行完语句后，会按照 character_set_result 系统变量设置的字符集返回结果集（或错误信息）到客户端。

    可以用语句：`show global variables like 'character_set_\%'`;查看这些系统变量设置。

- `@@session.collation_server=45`：用途：当你创建数据库，且没有指定字符集、字符序（校验规则）时，server 字符集、server 字符序就会作为该数据库的默认字符集、排序规则。

    如何指定：MySQL 服务启动时，可通过命令行参数指定。也可以通过配置文件的变量指定。

    server 默认字符集、校验规则：在 MySQL 编译的时候，通过编译参数指定。

    character_set_server、collation_server 分别对应 server 字符集、server 校验规则。

- `SET @@session.collation_database=DAFAULT` 当前数据库的默认校对。每次用 USE 语句来“跳转”到另一个数据库的时候，这个变量的值就会改变。如果没有当前数据库，这个变量的值就是 collation_server 变量的值。

#### 4.其它参数

- `schema`：6 字节，表示选择的数据库。

- `00`：默认的 1 个字节

- `sql文本`：33 个字节，即：query 的文本格式，里面存储的是 BEGIN（dml）或原生的 SQL（ddl）等。

### （四）Table_map event

用于从 MySQL 5.1.5 开始的基于行的二进制日志记录。每个 ROW_EVENT 之前都有一个 TABLE_MAP_EVENT，用于描述表的内部 ID 和结构定义。

#### 1.触发条件

ROW 格式中每个 ROW_EVENT 之前。

#### 2.整体结构

![image-20230409224048207](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409224048207.png)

 

- 固定数据部分：

    - `table id`：6 bytes  //6 个字节存储 table id

    - `reserve`：2 bytes：Reserved for future use //2 个字节保留未来使用

- 可变数据部分：

    - `db name len`：1 byte. The length of the database name.  //数据库名长度：1 字节。
    - `db name`：The database name (null-terminated).  //数据库名：可变长度

    - `table name len(1)`:01，表名占用 1 个字节。
    - `table name`：表名，可变长度。

    - `00`

    - `column count(1)`：列的个数，一个字节。
    - `column type`：//列类型数组，每一列 1 个字节。每个版本的数据库的数据类型不一定是相同的，所以在解析 binlog 的时候一定要注意版本的兼容性。比如 5.7 有地理位置这一类型，在 5.6 就不存在，所以用 5.6 的库去同步 5.7 的库就会造成错误。
    - `column metadata len（1）`：The length of the metadata block. //元数据块的长度。
    - `column metadata`：The metadata block; see log_event.h for contents and format. //元数据块 元数据块村的是类型的精度之类的数据，比如时间戳的精度，float 的精度等等。
    - `null bitmap`：//位字段，指示每个列是否可以为空，每个列一位。如果表有 N 列，需要：INT((N+7)/8) 字节。
    - `crc32（4）`：4 个字节，校验值。

#### 3.Table_map event 的作用：

- 检测 slave 上的表的字段类型是否和 master 上的表的字段类型兼容，如果不兼容，slave 会上报错误。

- 将数据转换为 slave 上表的字段类型，但仅限于字段类型兼容的情况。

- 为什么一个 update 在 ROW 模式下需要分解成两个 event：一个 Table_map，一个 Update_rows。我们想象一下，一个 update 如果更新了 10000 条数据，（大 rows 会拆分成小 rows）那么对应的表结构信息是否需要记录 10000 次?其实是对同一个表的操作，所以这里 binlog 只是记录了一个 Table_map 用于记录表结构相关信息,而后面的 Update_rows 记录了更新数据的行信息。他们之间是通过 table_id 来联系的。

#### 4.Table_map event 中的 table_id 的作用详解：

- 根据 table id 可以找到缓存中对用的表结构信息。

- table_id 和数据库表并非是固定的对应关系，table id 在 mysql 实例生命周期内是递增的。

- 如果缓存中已经存在该表，则 table id 不变，而如果 table cache 中不存在时，则该值根据上一次操作的 table id 自增 1；此外如果 table_definition_cache 缓存表中默认存放 n 个表定义，如果超出了该范围，则将最久未用的表定义置换出 table cache。使用 flush tables;会清空所有缓存的中表信息。

- table_map_event 和 rows_event 通过 table_id 进行关联，slave 通过先解析 table_map_event 并存于内存中，而后 rows_event 通过 table_id 找到对应的表信息。

#### 5.在一个事务中，一个insert对应一个table map event。而一个事务中，

### （五）WRITE_ROWS_EVENT，UPDATE_ROWS_EVENT，DELETE_ROWS_EVENT：

`SELECT @@GLOBAL.BINLOG_ROW_EVENT_MAX_SIZE`

rows event 是从库需要执行的具体语句。而且 row event 不记录具体的表结构信息。会记录每行数据的每个字段更改前后的值。

一条语句可能会导致多个 rows event，比如 delete from t limit 10000;如果一条 sql 语句只更改了一行数据，则一个 event 就一行数据。

![image-20230409224853168](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409224853168.png)

#### 1.版本信息：

- MySQL 5.1.0 to 5.1.15

    DELETE_ROWS_EVENTv0；UPDATE_ROWS_EVENTv0；WRITE_ROWS_EVENTv0

- MySQL 5.1.15 to 5.6.x

    DELETE_ROWS_EVENTv1；UPDATE_ROWS_EVENTv1；WRITE_ROWS_EVENTv1

- MySQL 5.6.x

    DELETE_ROWS_EVENTv2；UPDATE_ROWS_EVENTv2；WRITE_ROWS_EVENTv2

#### 2.私有事件头

- table ID：table id 与 table map event 中的 table id 对应。

- flag：2 个字节，可以包含以下信息。该事件是否是语句的最后一个事件，是否需要进行外键约束检查，针对 innodb 的二级索引是否需要进行唯一性检查，

该事件是否包含了完整一行数据也就是说覆盖了所有列。

。。。

#### 3.参数BINLOG_ROW_EVENT_MAX_SIZE

- 这个参数是 binlog 每个 event 的字节上限，当超过这个值的时候，大 event 会拆分成小的 event，比如说：event max size 大小为 8k，更新 10000 条数据，生产约 181K 的 binlog 日志，MySQL 将这些日志按照每个最大 8K 的拆分成 23 个 binlog rows event，每个 binlog event 大概包含 400-450 个行记录的更新日志。

- 那么同一个事务这么多 event 怎么知道是否是最后一个？

rows event 的私有事件的 post header 中会记录一个 flag 标志，来判断是否是最后一个 rows event，在 binlog 文件中的解读为：flags: STMT_END_F。

#### 4.还有另一个情况也会产生多个rows event那就是当一个dml造成多个表的变更，即使每个表只变更了一行也会产生n个row event。

### （六）PREVIOUS_GTIDS_LOG_EVENT

它记录了上一个 binlog 文件结束时执行到的 gtid 集合，包括已经删除的 BINLOG。

### （七）GTID EVENT

#### 1.gtid 简介：

从 MySQL 5.6.5 开始新增了一种基于 GTID 的复制方式。**通过 GTID 保证了每个在主库上提交的事务在集群中有一个唯一的 ID**。这种方式强化了数据库的主备一致性，故障恢复以及容错能力。

- GTID (Global Transaction ID)是全局事务 ID，当在主库上提交事务或者被从库应用时，可以定位和追踪每一个事务。它是全局唯一和单调递增的。

- 不支持非事务引擎。

- GTID 复制与普通复制最大的区别就是 不需要指定二进制文件名和位置。

#### 2.gtid 事件内容：

- gtid_flags，占用 1 个字节。记录 binlog 格式，如果 gtid_flags 值为 1，表示 binlog 中可能有以 statement 方式记录的 binlog，如果为 0 表示，binlog 中只有以 row 格式记录的 binlog。

- sid，server uuid，占用 16 字节，比如：b0d850c2-dbd0-11e9-90c3-080027b8bded，一共 16 字节。

- gno，占用 8 字节，比如：b0d850c2-dbd0-11e9-90c3-080027b8bded:1，数字 1 就是 gno，占用 8 个字节。

- lt_type，占用 1 字节，打开逻辑并行复制时，lt_type 值为 2。

- last_committed，占用 8 字节，lt_type 为 2 时才会有该值。

- sequence_number，占用 8 字节，lt_type 为 2 时才会有该值。

- 最后 4 字节，是这个 binlog event 的 crc32 检验值。

- ANONYMOUS_GTID_LOG_EVENT：在 ANONYMOUS GTID event 中除了 last_committed，sequence_number 其他字节为 0X00，这是为了提供组提交的信息。

#### 3.部分说明

- 在 gtid event 中，last_committed=2 组提交标识、同一组提交的事务具备相同的 last_committed 值，可以在从库并行重放，以减少同步延迟。

- sequence_number=3，事务对应的顺序号，该值单调递增，同时也标识了同一组提交事务的顺序，在从库设置 slave_preserve_commit_order=1 时，依赖该值模拟主库的提交顺序，在从库提交。以达到数据落盘过程完全一致。

在每台 MySQL 服务器上都是从 1 开始自增长的序列，一个数值对应一个事务。

#### 4.binlog如何保证同一个gtid的gno的有序性？

对于打开了 gtid_mode 的实例，每个事务起始位置都会有一个 gtid event，其内容输出格式为 UUID:gn，gno 是一个整型数。

gno 生成于事务提交时写 binlog 的时候。注意这里不是生成 binlog，而是将 binlog 写入磁盘的时候。因此实现上确保了同一个 UUID 下 gno 的有序性。

### （八）XID EVENT

当事务提交时，不论是 statement 还是 row 格式的 binlog 都会添加一个 XID_EVENT 作为事务的结束。该事件记录了该事务的 ID。在 mysql 进行崩溃恢复时根据 binlog 中提交的情况来决定是否提交存储引擎中 prepared 状态的事务。

- 表示支持内部 XA 的存储引擎上的事务提交。正常的事务是通过 QUERY_EVENT 来发送一个 BEGIN 语句并且通过 XID_EVENT 来发送一个 COMMIT 语句（如果事务回滚则发送的是 ROLLBACK）实现。

- XA 事务：XA 事务存在于 binlog 和 innodb 存储引擎之间，在事务提交的时候，binlog 日志和重做日志的提交是原子操作，即二进制日志和重做日志必须是同时写入磁盘的。若二进制先写了，而在写入 innodb 存储引擎时发生了宕机，那么 slave 可能会收到 master 并未提交的事务，而造成了主从不一致。为了解决这个问题，MySQL 数据库在 binlog 与 innodb 存储引擎之间采用 XA 事务。当事务提交时，innodb 存储引擎会先做一个 prepare 操作，将事务的 XID 写入，接着进行二进制日志的写入，最后在写入 redo log。redo log 与 bin log 通过事务 Xid 进行关联。其中，二进制日志（逻辑日志）只在事务提交完成后进行一次性写入，而 innodb 存储引擎的 redo log（物理日志）在事务进行中就不断被写入。

- 当 binlog 格式为 row，且事务中更新的是事务引擎时，每个事务的结束位置都有 Xid，Xid 的类型为整型。

    - MySQL 中每个语句都会被分配一个全局递增的 query_id(重启会被重置)，每个事务的 Xid 来源于事务第一个语句的 query_id。

        mysql query_id 记录在 INFORMATION_SCHEMA.PROFILING 这个表中，用于分析语句占用的资源情况。Query_ID 表示从连接上数据库到现在执行的 SQL 语句序号，select 也会记录。

    - xid 的无序性：

        ```sql
        考虑一个简单的操作顺序：
        
        session 1: begin; select; update;
        
        session 2: begin; select; update; insert; commit;（xid2）
        
        session 1: insert; commit;（xid1）
        
        显然Xid1 > Xid2，但因为事务2会先于事务1记录写binlog，因此在这个binlog中，会出现Xid不是有序的情况。
        ```

## 七、解析binlog后的每个事务的第一个TIMESTAMP：

1、戳的有序性可能是被误用最多的。在 mysqlbinlog 这个工具的输出结果中，每个事务起始有会输出一个 SET TIMESTAMP=n。这个值取自第一个更新事件（update）的时间。上一节的例子中，timestamp2>timestamp1,但因为事务 2 会先于事务 1 记录写 binlog，因此在这个 binlog 中，会出现 TIMESTAMP 不是有序的情况。

2、每个事务的多个 event 的时间戳是不同的，其中 gtid event 事件和 xid 事件的时间戳是事务提交时的时间戳，而 query evnet， rows event 的时间戳是事务生成时的时间戳。因此拿 xid 的时间戳是不准确的。

3、一个 binlog 文件中的 Xid 和 TIMESTAMP 无法保证有序性。在无特殊操作的情况下，相同的 UUID 可以保证 gno 的有序性。

## 八、MySQL是如何写binlog的？

参考：https://www.bookstack.cn/read/aliyun-rds-core/11ba3d7ee1553c0f.md

### （一）对于单个事务的两阶段提交过程

MySQL 采用了如下的过程实现内部 XA 的两阶段提交：

- Prepare 阶段：InnoDB 将回滚段设置为 prepare 状态；将 redolog 写文件并刷盘；

- Commit 阶段：Binlog 写入文件；binlog 刷盘；InnoDB commit；

- innodb redo log 这个 commit 和 prepare 状态它自身是不会记录的，而是由 XA 事务来进行标记的。
- 两阶段提交保证了事务在多个引擎和 binlog 之间的原子性，以 binlog 写入成功作为事务提交的标志；

单个事务的崩溃恢复：

在崩溃恢复中，是以 binlog 中的 xid 和 redolog 中的 xid 进行比较，xid 在 binlog 里存在则提交，不存在则回滚。

- 在 prepare 阶段崩溃，即已经写入 redolog，在写入 binlog 之前崩溃，则会回滚；

- 在 commit 阶段，当没有成功写入 binlog 时崩溃，也会回滚；

- 如果已经写入 binlog，在写入 InnoDB commit 标志时崩溃，则重新写入 commit 标志，完成提交。

### （二）关于组提交问题

MySQL 的内部 XA 机制保证了单个事务在 binlog 和 InnoDB 之间的原子性，接下来我们需要考虑，在多个事务并发执行的情况下，怎么保证在 binlog 和 redolog 中的顺序一致？为什么需要保证二进制日志的写入顺序和 InnoDB 层事务提交顺序一致性呢？

（事务按照 T1、T2、T3 顺序开始执行，将二进制日志（按照 T1、T2、T3 顺序）写入日志文件系统缓冲，调用 fsync()进行一次 group commit 将日志文件永久写入磁盘，但是存储引擎提交的顺序为 T2、T3、T1。当 T2、T3 提交事务之后，若通过在线物理备份[xtrabackup]进行数据库恢复来建立复制时，因为在 InnoDB 存储引擎层会检测事务 T3 在上下两层都完成了事务提交，不需要在进行恢复了，此时**主备**数据不一致，这里为什么会造成数据不一致呢？因为 T1、T2、T3 在 binlog 中已经落盘提交并且已经同步到从库了，但是事务 T1 在主库还没有提交，所以使用 xtra 备份的时候就只会备份到 T3，而 T1 是不会进行备份的，这样就造成了**主备**数据不一致了。）

### （三）两个写磁盘的参数

innodb_flush_log_at_trx_commit：控制 redo log 刷盘策略。

- innodb_flush_log_at_trx_commit=0：每秒一次将 Log Buffer 中数据写入到 Log File 中，并且 Flush 到磁盘。事务提交不会主动触发写磁盘操作。

    当 innodb_flush_log_at_trx_commit=0 时，最近一秒的事务日志存在 MySQL 的 Log Buffer 中，无论是 MySQL 实例停止还是 MySQL 服务器宕机，都会导致最近一秒的事务日志丢失。

- innodb_flush_log_at_trx_commit=1：每次事务提交时将 Log Buffer 数据写入到 Log File 中，并且 Flush 到磁盘。

- innodb_flush_log_at_trx_commit=2：每次事务提交时将 Log Buffer 数据写入到 Log File 中，但不立即 Flush 到磁盘，MySQL 会每秒一次刷新到磁盘。由于进程调度问题，每条一次操作不能保证每一秒都执行一次。

    当 innodb_flush_log_at_trx_commit=1 时，最近一秒的事务日志存在操作系统的文件缓存中，MySQL 实例停止不会导致事务日志丢失，但 MySQL 服务器宕机会导致最近一秒事务日志丢失。上述的一秒一次刷新，取决于参数 innodb_flush_log_at_timeout 默认值为 1，DDL 或其他 InnoDB 内部操作并不受参数 innodb_flush_log_at_trx_commit 的限制。

sync_binlog：控制 binlog 落盘操作。

- sync_binlog=0：每次事务提交后，将 Binlog Cache 中的数据写入到 Binlog 文件，但不立即刷新到磁盘。由文件系统(file system)决定何时刷新到磁盘中。

    当 sync_binlog=0(默认设置)时，不会执行强制刷盘指令，性能最好同时风险最高。

- sync_binlog=N：每 N 次事务提交后，将 Binlog Cache 中的数据写入到 Binlog 文件,调用 fdatasync()函数将数据刷新到磁盘中。

    当 sync_binlog=N 时，当 MySQL 服务器宕机后，会导致最近 N 个事务的 BINLOG 丢失。

### （四）binlog的写入过程

首先，事务语句执行期间，binlog event 写入到 binlog buffer（内存）；其次，如果 binlog buffer 超过 binlog_cache_size 设定的大小后，就会将事件放到临时文件中（disk），这个是落盘的；最后，当 commit 的时候，将日志写入到 binlog file 中。

### （五）版本改进

- 在 MySQL 5.6 版本之前，使用 prepare_commit_mutex 对整个 2PC 过程进行加锁，只有当上一个事务 commit 后释放锁，下个事务才可以进行 prepare 操作，这样完全串行化的执行保证了顺序一致。

    存在的问题是，prepare_commit_mutex 的锁机制会严重影响高并发时的性能，在每个事务执行过程中，都会调用多次刷盘操作。

- 为了提高并发性能，肯定要细化锁粒度。MySQL 5.6 引入了 binlog 的组提交（group commit）功能，prepare 阶段不变，只针对 commit 阶段，将 commit 阶段拆分为三个过程：
    - flush stage：多个线程按进入的顺序将 binlog 从 cache 写入文件（不刷盘）；
    - sync stage：对 binlog 文件做 fsync 操作（多个线程的 binlog 合并一次刷盘）；
    - commit stage：各个线程按顺序做 InnoDB commit 操作。

- MySQL 5.6 的组提交逻辑中，每个事务各自做 prepare 并写 redo log，只有到了 commit 阶段才进入组提交，因此每个事务的 redolog sync 操作成为性能瓶颈。

    在 5.7 版本中，修改了组提交的 flush 阶段，在 prepare 阶段不再让线程各自执行 flush redolog 操作，而是推迟到组提交的 flush 阶段，flush stage 修改成如下逻辑：

    - 收集组提交队列，得到 leader 线程，其余 follower 线程进入阻塞；
    - leader 调用 ha_flush_logs 做一次 redo write/sync，即，一次将所有线程的 redolog 刷盘；
    - 将队列中 thd 的所有 binlog cache 写到 binlog 文件中。

    这个优化是将 redolog 的刷盘延迟到了 binlog group commit 的 flush stage 之中，sync binlog 之前。通过延迟写 redolog 的方式，为 redolog 做了一次组写入，这样 binlog 和 redolog 都进行了优化。

## 九、MySQL的crash-safe？

### （一）什么是crash-safe？

答：crash-safe replication 是指当 master/slave 任何一个节点发生宕机等意外情况下，服务器重启后 master/slave 的数据依然能够保证一致性。

https://tech.youzan.com/shen-ru-qian-chu-mysql-crash-safe/

## 十、gtid executed 和 gtid purged？

1、

## 十一、replace into产生的binlog问题？

### 场景1：有一个唯一索引和主键。主键和唯一索引都冲突

表 1：

CREATE TABLE `test` (

 `id` bigint(20) NOT NULL AUTO_INCREMENT,

 `name` varchar(250) DEFAULT NULL,

 `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

 PRIMARY KEY (`id`),

 UNIQUE KEY `name` (`name`)

) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8 COMMENT='test'

两条数据：

![image-20230409230752512](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409230752512.png) 

 

1、执行：replace into test(name) values ('mk1');

对应的操作：这条 sql 与唯一索引 mk1 冲突，实际 mysql 底层采用的是 update 的方式：

```sql
BEGIN

/*!*/;

\# at 898

\#210410 21:53:55 server id 1 end_log_pos 949 CRC32 0x75f759dd    Table_map: `web`.`test` mapped to number 334

\# at 949

\#210410 21:53:55 server id 1 end_log_pos 1023 CRC32 0xc59becf3   Update_rows: table id 334 flags: STMT_END_F

\### UPDATE `web`.`test`

\### WHERE

\###  @1=9 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='mk1' /* VARSTRING(750) meta=750 nullable=1 is_null=0 */

\###  @3='2021-04-10 21:52:31' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\### SET

\###  @1=11 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='mk1' /* VARSTRING(750) meta=750 nullable=1 is_null=0 */

\###  @3='2021-04-10 21:53:55' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\# at 1023

```

2、当前数据：

 ![image-20230409230857289](binlog%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90.resource/image-20230409230857289.png)

执行 sql：replace into test (id, name) values (12, 'mk1');

分析：主键冲突和唯一索引冲突。mysql 底层处理：先删除 id=12 的这条数据，然后更新唯一索引为 mk1 的这条数据。binlog 数据如下：

```sql
\#210410 21:58:20 server id 1 end_log_pos 1549 CRC32 0x47d929ca   Table_map: `web`.`test` mapped to number 334

\# at 1549

\#210410 21:58:20 server id 1 end_log_pos 1603 CRC32 0x0b9bb956  Delete_rows: table id 334

\# at 1603

\#210410 21:58:20 server id 1 end_log_pos 1677 CRC32 0x3dbbf24d   Update_rows: table id 334 flags: STMT_END_F

\### DELETE FROM `web`.`test`

\### WHERE

\###  @1=12 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='mk2' /* VARSTRING(750) meta=750 nullable=1 is_null=0 */

\###  @3='2021-04-10 21:56:29' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\### UPDATE `web`.`test`

\### WHERE

\###  @1=11 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='mk1' /* VARSTRING(750) meta=750 nullable=1 is_null=0 */

\###  @3='2021-04-10 21:53:55' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\### SET

\###  @1=12 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='mk1' /* VARSTRING(750) meta=750 nullable=1 is_null=0 */

\###  @3='2021-04-10 21:58:20' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\# at 1677

\#210410 21:58:20 server id 1 end_log_pos 1708 CRC32 0xcb5baf2b   Xid = 38

COMMIT/*!*/;
```



### 场景2：两个唯一索引：主键和唯一索引都冲突**

表 2：

CREATE TABLE `test6` (

 `id` bigint(20) NOT NULL AUTO_INCREMENT,

 `update_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',

 `num1` bigint(20) NOT NULL,

 `num2` bigint(20) NOT NULL,

 PRIMARY KEY (`id`),

 UNIQUE KEY `num` (`num1`,`num2`),

 UNIQUE KEY `num1` (`num1`),

 UNIQUE KEY `num2` (`num2`)

) ENGINE=InnoDB AUTO_INCREMENT=31 DEFAULT CHARSET=utf8 COMMENT='test6'

历史数据

 

执行 sql：replace into test6 (id, num1, num2) values (25,100, 5);

对应的 binlog 日志：

BEGIN

/*!*/;

\# at 3508

\#210410 22:31:58 server id 1 end_log_pos 3559 CRC32 0x454a31c5   Table_map: `web`.`test6` mapped to number 343

\# at 3559

\#210410 22:31:58 server id 1 end_log_pos 3654 CRC32 0x31e51ba0   Delete_rows: table id 343

\# at 3654

\#210410 22:31:58 server id 1 end_log_pos 3750 CRC32 0x69a8def1   Update_rows: table id 343 flags: STMT_END_F

\### DELETE FROM `web`.`test6`

\### WHERE

\###  @1=25 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='2021-02-09 17:40:32' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\###  @3=6 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @4=6 /* LONGINT meta=0 nullable=0 is_null=0 */

\### DELETE FROM `web`.`test6`

\### WHERE

\###  @1=20 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='2021-02-09 09:22:33' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\###  @3=100 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @4=100 /* LONGINT meta=0 nullable=0 is_null=0 */

\### UPDATE `web`.`test6`

\### WHERE

\###  @1=27 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='2021-02-10 21:53:48' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\###  @3=5 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @4=5 /* LONGINT meta=0 nullable=0 is_null=0 */

\### SET

\###  @1=25 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @2='2021-04-10 22:31:58' /* DATETIME(0) meta=0 nullable=1 is_null=0 */

\###  @3=100 /* LONGINT meta=0 nullable=0 is_null=0 */

\###  @4=5 /* LONGINT meta=0 nullable=0 is_null=0 */

\# at 3750

\#210410 22:31:58 server id 1 end_log_pos 3781 CRC32 0x3bddb3ff   Xid = 64

COMMIT/*!*/;

 

整个删除的顺序是按照列的顺序来的，即会更新最后一个唯一索引所在的行，前面索引冲突的行都会被删除。

**场景****3****：只有主键冲突，唯一索引不冲突：**

delete+insert。

**场景****3****：只有唯一索引冲突，主键不冲突：**

delete+update

**问题：**

1、为什么一个事务中，每执行一个 insert，都会生成一个 table map event，而不是共用一个 table map event？

事实上 mysql 是一个事务中的一条 sql 产生一个 table map event，而当执行 replace into 的时候，虽然底层可能是 delete+update 的形式，但是 binlog 中对应的还是一个 table map event。

2、解析 binlog 的时候 binlog 的 positon 如果使用 uint32 来解析，可能会超，因为 uint32 来解析 position 只能解析 4G 的文件，当 binlog 文件大于 4G 的时候，就不能用 uint32 来解析，应该用 uint64。

 

3、如果在 binlog dump 的时候，起始位点如果是 table map event 之后的位点，则 dump 位点的时候会报错。因为找不到 table id。

4、mysql 主从架构，半同步复制，异步复制是由从库决定的还是主库决定的，还是两者结合呢？

 

 

 

 