# 腾讯工程师带你深入解析 MySQL binlog

> 原文：https://zhuanlan.zhihu.com/p/33504555

[腾讯云开发者](https://www.zhihu.com/org/teng-xun-yun-ji-zhu-she-qu)

[](https://www.zhihu.com/question/48509984)

人工智能话题下的优秀答主

190 人赞同了该文章

欢迎大家前往[云+社区](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer)，获取更多腾讯海量技术实践干货哦~

> 作者：[腾讯云数据库内核团队](https://link.zhihu.com/?target=https%3A//cloud.tencent.com/developer/user/1002332/activities)
> 原文标题：【腾讯云CDB】深入解析MySQL binlog

## 1.概述

binlog 是 Mysql sever 层维护的一种二进制日志，与 innodb 引擎中的 redo/undo log 是完全不同的日志；其主要是用来记录对 mysql 数据更新或潜在发生更新的 SQL 语句，并以"事务"的形式保存在磁盘中；

作用主要有：

- 复制：MySQL Replication 在 Master 端开启 binlog，Master 把它的二进制日志传递给 slaves 并回放来达到 master-slave 数据一致的目的
- 数据恢复：通过 mysqlbinlog 工具恢复数据
- 增量备份

## 2.binlog管理

- 开启 binlogmy.cnf 配置中设置：log_bin="存放 binlog 路径目录"

```text
binlog信息查询binlog开启后，可以在配置文件中查看其位置信息，也可以在myslq命令行中查看：
show variables like '%log_bin%';
+---------------------------------+-------------------------------------+
| Variable_name                   | Value                               |
+---------------------------------+-------------------------------------+
| log_bin                         | ON                                  |
| log_bin_basename                | /var/lib/mysql/3306/mysql-bin       |
| log_bin_index                   | /var/lib/mysql/3306/mysql-bin.index |
| log_bin_trust_function_creators | OFF                                 |
| log_bin_use_v1_row_events       | OFF                                 |
| sql_log_bin                     | ON                                  |
+---------------------------------+-------------------------------------+
binlog文件开启binlog后，会在数据目录（默认）生产host-bin.n（具体binlog信息）文件及host-bin.index索引文件（记录binlog文件列表）。当binlog日志写满(binlog大小max_binlog_size，默认1G),或者数据库重启才会生产新文件，但是也可通过手工进行切换让其重新生成新的文件（flush logs）；另外，如果正使用大的事务，由于一个事务不能横跨两个文件，因此也可能在binlog文件未满的情况下刷新文件
mysql> show binary logs; //查看binlog文件列表,
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       177 |
| mysql-bin.000002 |       177 |
| mysql-bin.000003 |  10343266 |
| mysql-bin.000004 |  10485660 |
| mysql-bin.000005 |     53177 |
| mysql-bin.000006 |      2177 |
| mysql-bin.000007 |      1383 |
+------------------+-----------+
```



```text
查看binlog的状态：show master status可查看当前二进制日志文件的状态信息，显示正在写入的二进制文件，及当前position
 mysql> show master status;
 +------------------+----------+--------------+------------------+-------------------+
 | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
 +------------------+----------+--------------+------------------+-------------------+
 | mysql-bin.000007 |      120 |              |                  |                   |
 +------------------+----------+--------------+------------------+-------------------+
```



- reset master 清空 binlog 日志文件

## 3.binlog内容

默认情况下 binlog 日志是二进制格式，无法直接查看。可使用两种方式进行查看：

```text
   a. mysqlbinlog: /usr/bin/mysqlbinlog  mysql-bin.000007
        - mysqlbinlog是mysql官方提供的一个binlog查看工具，
        - 也可使用–read-from-remote-server从远程服务器读取二进制日志，
        - 还可使用--start-position --stop-position、--start-time= --stop-time精确解析binlog日志
        
        截取位置1190-1352 binlog如下：
        ***************************************************************************************
        # at 1190   //事件的起点
        #171223 21:56:26 server id 123  end_log_pos 1190 CRC32 0xf75c94a7 	Intvar
        SET INSERT_ID=2/*!*/;
        #171223 21:56:26 server id 123  end_log_pos 1352 CRC32 0xefa42fea 	Query	thread_id=4	exec_time=0	error_code=0
        SET TIMESTAMP=1514123786/*!*/;              //开始事务的时间起点 (每个at即为一个event)
        insert into tb_person  set name="name__2", address="beijing", sex="man", other="nothing"  //sql语句
        /*!*/;
        # at 1352
        #171223 21:56:26 server id 123  end_log_pos 1383 CRC32 0x72c565d3 	Xid = 5 //执行时间，及位置戳，Xid:事件指示提交的XA事务
        ***************************************************************************************
    
    b.直命令行解析
        SHOW BINLOG EVENTS
            [IN 'log_name'] //要查询的binlog文件名
            [FROM pos]  
            [LIMIT [offset,] row_count]  
       
        1190-135如下：mysql> show binlog events in 'mysql-bin.000007' from 1190 limit 2\G
        *************************** 13. row ***************************
           Log_name: mysql-bin.000007
                Pos: 1190
         Event_type: Query  //事件类型
          Server_id: 123
        End_log_pos: 1352   //结束pose点，下个事件的起点
               Info: use `test`; insert into tb_person  set name="name__2", address="beijing", sex="man", other="nothing"
        *************************** 14. row ***************************
           Log_name: mysql-bin.000007
                Pos: 1352
         Event_type: Xid
          Server_id: 123
        End_log_pos: 1383
               Info: COMMIT /* xid=51 */
```

## 4.binlog格式

Mysql binlog 日志有 ROW,Statement,MiXED 三种格式；可通过 my.cnf 配置文件及 ==set global binlog_format='ROW/STATEMENT/MIXED'== 进行修改，命令行 ==show variables like 'binlog_format'== 命令查看 binglog 格式；。

- Row level: 仅保存记录被修改细节，不记录 sql 语句上下文相关信息优点：能非常清晰的记录下每行数据的修改细节，不需要记录上下文相关信息，因此不会发生某些特定情况下的 procedure、function、及 trigger 的调用触发无法被正确复制的问题，任何情况都可以被复制，且能加快从库重放日志的效率，保证从库数据的一致性
    缺点:由于所有的执行的语句在日志中都将以每行记录的修改细节来记录，因此，可能会产生大量的日志内容，干扰内容也较多；比如一条 update 语句，如修改多条记录，则 binlog 中每一条修改都会有记录，这样造成 binlog 日志量会很大，特别是当执行 alter table 之类的语句的时候，由于表结构修改，每条记录都发生改变，那么该表每一条记录都会记录到日志中，实际等于重建了表。
    tip: - row 模式生成的 sql 编码需要解码，不能用常规的办法去生成，需要加上相应的参数(--base64-output=decode-rows -v)才能显示出 sql 语句; - 新版本 binlog 默认为 ROW level，且 5.6 新增了一个参数：binlog_row_image；把 binlog_row_image 设置为 minimal 以后，binlog 记录的就只是影响的列，大大减少了日志内容
- Statement level: 每一条会修改数据的 sql 都会记录在 binlog 中优点：只需要记录执行语句的细节和上下文环境，避免了记录每一行的变化，在一些修改记录较多的情况下相比 ROW level 能大大减少 binlog 日志量，节约 IO，提高性能；还可以用于实时的还原；同时主从版本可以不一样，从服务器版本可以比主服务器版本高
    缺点：为了保证 sql 语句能在 slave 上正确执行，必须记录上下文信息，以保证所有语句能在 slave 得到和在 master 端执行时候相同的结果；另外，主从复制时，存在部分函数（如 sleep）及存储过程在 slave 上会出现与 master 结果不一致的情况，而相比 Row level 记录每一行的变化细节，绝不会发生这种不一致的情况
- Mixedlevel level: 以上两种 level 的混合使用经过前面的对比，可以发现 ROW level 和 statement level 各有优势，如能根据 sql 语句取舍可能会有更好地性能和效果；Mixed level 便是以上两种 leve 的结合。不过，新版本的 MySQL 对 row level 模式也被做了优化，并不是所有的修改都会以 row level 来记录，像遇到表结构变更的时候就会以 statement 模式来记录，如果 sql 语句确实就是 update 或者 delete 等修改数据的语句，那么还是会记录所有行的变更；因此，现在一般使用 row level 即可。
- 选取规则如果是采用 INSERT，UPDATE，DELETE 直接操作表的情况，则日志格式根据 binlog_format 的设定而记录
    如果是采用 GRANT，REVOKE，SET PASSWORD 等管理语句来做的话，那么无论如何都采用 statement 模式记录

## 5.复制

复制是 mysql 最重要的功能之一，mysql 集群的高可用、负载均衡和读写分离都是基于复制来实现的；从 5.6 开始复制有两种实现方式，基于 binlog 和基于 GTID（全局事务标示符）；本文接下来将介绍基于 binlog 的一主一从复制；其复制的基本过程如下：

```text
   a.Master将数据改变记录到二进制日志(binary log)中
    b.Slave上面的IO进程连接上Master，并请求从指定日志文件的指定位置（或者从最开始的日志）之后的日志内容
    c.Master接收到来自Slave的IO进程的请求后，负责复制的IO进程会根据请求信息读取日志指定位置之后的日志信息，返回给Slave的IO进程。
        返回信息中除了日志所包含的信息之外，还包括本次返回的信息已经到Master端的bin-log文件的名称以及bin-log的位置
    d.Slave的IO进程接收到信息后，将接收到的日志内容依次添加到Slave端的relay-log文件的最末端，并将读取到的Master端的 bin-log的
        文件名和位置记录到master-info文件中，以便在下一次读取的时候能够清楚的告诉Master从某个bin-log的哪个位置开始往后的日志内容
    e.Slave的Sql进程检测到relay-log中新增加了内容后，会马上解析relay-log的内容成为在Master端真实执行时候的那些可执行的内容，并在自身执行
```

接下来使用实例演示基于 binlog 的主从复制：

```text
   a.配置master
        主要包括设置复制账号，并授予REPLICATION SLAVE权限，具体信息会存储在于master.info文件中，及开启binlog；
        mysql> CREATE USER 'test'@'%' IDENTIFIED BY '123456';
        mysql> GRANT REPLICATION SLAVE ON *.* TO 'test'@'%';
        mysql> show variables like "log_bin";
            +---------------+-------+
            | Variable_name | Value |
            +---------------+-------+
            | log_bin       | ON    |
            +---------------+-------+
        查看master当前binlogmysql状态：mysql> show master status;
            +------------------+----------+--------------+------------------+-------------------+
            | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
            +------------------+----------+--------------+------------------+-------------------+
            | mysql-bin.000003 |      120 |              |                  |                   |
            +------------------+----------+--------------+------------------+-------------------+
        建表插入数据：
            CREATE TABLE `tb_person` (
        	   `id` int(11) NOT NULL AUTO_INCREMENT,
               `name` varchar(36) NOT NULL,                           
               `address` varchar(36) NOT NULL DEFAULT '',    
               `sex` varchar(12) NOT NULL DEFAULT 'Man' ,
        	   `other` varchar(256) NOT NULL ,
               PRIMARY KEY (`id`)
             ) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8;
             
        	 insert into tb_person  set name="name1", address="beijing", sex="man", other="nothing";
        	 insert into tb_person  set name="name2", address="beijing", sex="man", other="nothing";
        	 insert into tb_person  set name="name3", address="beijing", sex="man", other="nothing";
        	 insert into tb_person  set name="name4", address="beijing", sex="man", other="nothing";
    b.配置slave
        Slave的配置类似master，需额外设置relay_log参数，slave没有必要开启二进制日志，如果slave为其它slave的master，须设置bin_log
    c.连接master
        mysql> CHANGE MASTER TO
           MASTER_HOST='10.108.111.14',
           MASTER_USER='test',
           MASTER_PASSWORD='123456',
           MASTER_LOG_FILE='mysql-bin.000003',
           MASTER_LOG_POS=120;
    d.show slave status;
        mysql> show slave status\G
        *************************** 1. row ***************************
                       Slave_IO_State:   ---------------------------- slave io状态，表示还未启动
                          Master_Host: 10.108.111.14  
                          Master_User: test  
                          Master_Port: 20126  
                        Connect_Retry: 60   ------------------------- master宕机或连接丢失从服务器线程重新尝试连接主服务器之前睡眠时间
                      Master_Log_File: mysql-bin.000003  ------------ 当前读取master binlog文件
                  Read_Master_Log_Pos: 120  ------------------------- slave读取master binlog文件位置
                       Relay_Log_File: relay-bin.000001  ------------ 回放binlog
                        Relay_Log_Pos: 4   -------------------------- 回放relay log位置
                Relay_Master_Log_File: mysql-bin.000003  ------------ 回放log对应maser binlog文件
                     Slave_IO_Running: No
                    Slave_SQL_Running: No
                  Exec_Master_Log_Pos: 0  --------------------------- 相对于master从库的sql线程执行到的位置
                Seconds_Behind_Master: NULL
        Slave_IO_State, Slave_IO_Running, 和Slave_SQL_Running为NO说明slave还没有开始复制过程。
    e.启动复制
        start slave
    f.再次观察slave状态
        mysql> show slave status\G
        *************************** 1. row ***************************
                       Slave_IO_State: Waiting for master to send event -- 等待master新的event
                          Master_Host: 10.108.111.14
                          Master_User: test
                          Master_Port: 20126
                        Connect_Retry: 60
                      Master_Log_File: mysql-bin.000003
                  Read_Master_Log_Pos: 3469  ---------------------------- 3469  等于Exec_Master_Log_Pos，已完成回放
                       Relay_Log_File: relay-bin.000002                    ||
                        Relay_Log_Pos: 1423                                ||
                Relay_Master_Log_File: mysql-bin.000003                    ||
                     Slave_IO_Running: Yes                                 ||
                    Slave_SQL_Running: Yes                                 ||
                  Exec_Master_Log_Pos: 3469  -----------------------------3469  等于slave读取master binlog位置，已完成回放
                Seconds_Behind_Master: 0
        可看到slave的I/O和SQL线程都已经开始运行，而且Seconds_Behind_Master=0。Relay_Log_Pos增加，意味着一些事件被获取并执行了。
        
        最后看下如何正确判断SLAVE的延迟情况，判定slave是否追上master的binlog：
        1、首先看 Relay_Master_Log_File 和 Maser_Log_File 是否有差异；
        2、如果Relay_Master_Log_File 和 Master_Log_File 是一样的话，再来看Exec_Master_Log_Pos 和 Read_Master_Log_Pos 的差异，对比SQL线程比IO线程慢了多少个binlog事件；
        3、如果Relay_Master_Log_File 和 Master_Log_File 不一样，那说明延迟可能较大，需要从MASTER上取得binlog status，判断当前的binlog和MASTER上的差距；
        4、如果以上都不能发现问题，可使用pt_heartbeat工具来监控主备复制的延迟。
        
    g.查询slave数据，主从一致
        mysql> select * from tb_person;
            +----+-------+---------+-----+---------+
            | id | name  | address | sex | other   |
            +----+-------+---------+-----+---------+
            |  5 | name4 | beijing | man | nothing |
            |  6 | name2 | beijing | man | nothing |
            |  7 | name1 | beijing | man | nothing |
            |  8 | name3 | beijing | man | nothing |
            +----+-------+---------+-----+---------+
关于mysql复制的内容还有很多，比如不同的同步方式、复制格式情况下有什么区别，有什么特点，应该在什么情况下使用....这里不再一一介绍。
```

## 6.恢复

```text
   恢复是binlog的两大主要作用之一，接下来通过实例演示如何利用binlog恢复数据：
    
    a.首先，看下当前binlog位置
        mysql> show master status;
        +------------------+----------+--------------+------------------+-------------------+
        | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
        +------------------+----------+--------------+------------------+-------------------+
        | mysql-bin.000008 |     1847 |              |                  |                   |
        +------------------+----------+--------------+------------------+-------------------+
    b.向表tb_person中插入两条记录：
        insert into tb_person  set name="person_1", address="beijing", sex="man", other="test-1";
        insert into tb_person  set name="person_2", address="beijing", sex="man", other="test-2";
    c.记录当前binlog位置：
        mysql> show master status;
        +------------------+----------+--------------+------------------+-------------------+
        | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
        +------------------+----------+--------------+------------------+-------------------+
        | mysql-bin.000008 |     2585 |              |                  |                   |
        +------------------+----------+--------------+------------------+-------------------+
    d.查询数据 
        mysql> select *  from tb_person where name ="person_2" or name="person_1";
        +----+----------+---------+-----+--------+
        | id | name     | address | sex | other  |
        +----+----------+---------+-----+--------+
        |  6 | person_1 | beijing | man | test-1 |
        |  7 | person_2 | beijing | man | test-2 |
        +----+----------+---------+-----+--------+
    e.删除一条: delete from tb_person where name ="person_2";
        mysql> select *  from tb_person where name ="person_2" or name="person_1";
        +----+----------+---------+-----+--------+
        | id | name     | address | sex | other  |
        +----+----------+---------+-----+--------+
        |  6 | person_1 | beijing | man | test-1 |
        +----+----------+---------+-----+--------+
    f. binlog恢复（指定pos点恢复/部分恢复）
        mysqlbinlog   --start-position=1847  --stop-position=2585  mysql-bin.000008  > test.sql
        mysql> source /var/lib/mysql/3306/test.sql
    d.数据恢复完成 
        mysql> select *  from tb_person where name ="person_2" or name="person_1";
        +----+----------+---------+-----+--------+
        | id | name     | address | sex | other  |
        +----+----------+---------+-----+--------+
        |  6 | person_1 | beijing | man | test-1 |
        |  7 | person_2 | beijing | man | test-2 |
        +----+----------+---------+-----+--------+
    e.总结
        恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已
```

## 7.总结

本文简要介绍 binlog 原理及其在恢复、复制中的使用方法；

## 8.参考

- *[https://dev.mysql.com/doc/internals/en/binary-log-versions.html](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/internals/en/binary-log-versions.html)*
- *[http://www.php.cn/mysql-tutorials-361643.html](https://link.zhihu.com/?target=http%3A//www.php.cn/mysql-tutorials-361643.html)*
- *[https://www.jianshu.com/p/c16686b35807](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/c16686b35807)*
- *[https://www.cnblogs.com/jackluo/p/3336585.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/jackluo/p/3336585.html)*
- *[http://www.cnblogs.com/hustcat/archive/2009/12/19/1627525.html](https://link.zhihu.com/?target=http%3A//www.cnblogs.com/hustcat/archive/2009/12/19/1627525.html)*

## 相关阅读

[腾讯工程师教你玩转 RocksDB](https://link.zhihu.com/?target=https%3A//link.juejin.im/%3Ftarget%3Dhttps%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1032768)

[虚拟机备份和恢复的六大最佳实践](https://link.zhihu.com/?target=https%3A//link.juejin.im/%3Ftarget%3Dhttps%3A%2F%2Fcloud.tencent.com%2Fdeveloper%2Farticle%2F1014749)