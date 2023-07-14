> 原文：https://www.cnblogs.com/martinzhang/p/3454358.html

binlog 基本认识
    MySQL 的二进制日志可以说是 MySQL 最重要的日志了，它记录了所有的 DDL 和 DML(除了数据查询语句)语句，以事件形式记录，还包含语句所执行的消耗的时间，MySQL 的二进制日志是事务安全型的。

    一般来说开启二进制日志大概会有1%的性能损耗(参见MySQL官方中文手册 5.1.24版)。二进制有两个最重要的使用场景: 
    其一：MySQL Replication在Master端开启binlog，Mster把它的二进制日志传递给slaves来达到master-slave数据一致的目的。 
    其二：自然就是数据恢复了，通过使用mysqlbinlog工具来使恢复数据。
    
    二进制日志包括两类文件：二进制日志索引文件（文件名后缀为.index）用于记录所有的二进制文件，二进制日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML(除了数据查询语句)语句事件。 


一、开启 binlog 日志：
    vi 编辑打开 mysql 配置文件
    # vi /usr/local/mysql/etc/my.cnf
    在[mysqld] 区块
    设置/添加 log-bin=mysql-bin  确认是打开状态(值 mysql-bin 是日志的基本名或前缀名)；

    重启mysqld服务使配置生效
    # pkill mysqld
    # /usr/local/mysql/bin/mysqld_safe --user=mysql &


二、也可登录 mysql 服务器，通过 mysql 的变量配置表，查看二进制日志是否已开启 单词：variable[ˈvɛriəbəl] 变量

    登录服务器
    # /usr/local/mysql/bin/mysql -uroot -p123456
    
    mysql> show variables like 'log_%'; 
    +----------------------------------------+---------------------------------------+
    | Variable_name                          | Value                                 |
    +----------------------------------------+---------------------------------------+
    | log_bin                                | ON                                    | ------> ON表示已经开启binlog日志
    | log_bin_basename                       | /usr/local/mysql/data/mysql-bin       |
    | log_bin_index                          | /usr/local/mysql/data/mysql-bin.index |
    | log_bin_trust_function_creators        | OFF                                   |
    | log_bin_use_v1_row_events              | OFF                                   |
    | log_error                              | /usr/local/mysql/data/martin.err      |
    | log_output                             | FILE                                  |
    | log_queries_not_using_indexes          | OFF                                   |
    | log_slave_updates                      | OFF                                   |
    | log_slow_admin_statements              | OFF                                   |
    | log_slow_slave_statements              | OFF                                   |
    | log_throttle_queries_not_using_indexes | 0                                     |
    | log_warnings                           | 1                                     |
    +----------------------------------------+---------------------------------------+

三、常用 binlog 日志操作命令
    1.查看所有 binlog 日志列表
      mysql> show master logs;

    2.查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值
      mysql> show master status;
    
    3.刷新log日志，自此刻开始产生一个新编号的binlog日志文件
      mysql> flush logs;
      注：每当mysqld服务重启时，会自动执行此命令，刷新binlog日志；在mysqldump备份数据时加 -F 选项也会刷新binlog日志；
    
    4.重置(清空)所有binlog日志
      mysql> reset master;


四、查看某个 binlog 日志内容，常用有两种方式：

    1.使用mysqlbinlog自带查看命令法：
      注: binlog是二进制文件，普通文件查看器cat more vi等都无法打开，必须使用自带的 mysqlbinlog 命令查看
          binlog日志与数据库文件在同目录中(我的环境配置安装是选择在/usr/local/mysql/data中)
      在MySQL5.5以下版本使用mysqlbinlog命令时如果报错，就加上 “--no-defaults”选项
    
      # /usr/local/mysql/bin/mysqlbinlog /usr/local/mysql/data/mysql-bin.000013
        下面截取一个片段分析：
    
         ...............................................................................
         # at 552
         #131128 17:50:46 server id 1  end_log_pos 665   Query   thread_id=11    exec_time=0     error_code=0 ---->执行时间:17:50:46；pos点:665
         SET TIMESTAMP=1385632246/*!*/;
         update zyyshop.stu set name='李四' where id=4              ---->执行的SQL
         /*!*/;
         # at 665
         #131128 17:50:46 server id 1  end_log_pos 692   Xid = 1454 ---->执行时间:17:50:46；pos点:692 
         ...............................................................................
    
         注: server id 1     数据库主机的服务号；
             end_log_pos 665 pos点
             thread_id=11    线程号


    2.上面这种办法读取出binlog日志的全文内容较多，不容易分辨查看pos点信息，这里介绍一种更为方便的查询命令：
    
      mysql> show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];
    
             选项解析：
               IN 'log_name'   指定要查询的binlog文件名(不指定就是第一个binlog文件)
               FROM pos        指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
               LIMIT [offset,] 偏移量(不指定就是0)
               row_count       查询总条数(不指定就是所有行)
    
             截取部分查询结果：
             *************************** 20. row ***************************
                Log_name: mysql-bin.000021  ----------------------------------------------> 查询的binlog日志文件名
                     Pos: 11197 ----------------------------------------------------------> pos起始点:
              Event_type: Query ----------------------------------------------------------> 事件类型：Query
               Server_id: 1 --------------------------------------------------------------> 标识是由哪台服务器执行的
             End_log_pos: 11308 ----------------------------------------------------------> pos结束点:11308(即：下行的pos起始点)
                    Info: use `zyyshop`; INSERT INTO `team2` VALUES (0,345,'asdf8er5') ---> 执行的sql语句
             *************************** 21. row ***************************
                Log_name: mysql-bin.000021
                     Pos: 11308 ----------------------------------------------------------> pos起始点:11308(即：上行的pos结束点)
              Event_type: Query
               Server_id: 1
             End_log_pos: 11417
                    Info: use `zyyshop`; /*!40000 ALTER TABLE `team2` ENABLE KEYS */
             *************************** 22. row ***************************
                Log_name: mysql-bin.000021
                     Pos: 11417
              Event_type: Query
               Server_id: 1
             End_log_pos: 11510
                    Info: use `zyyshop`; DROP TABLE IF EXISTS `type`
    
      这条语句可以将指定的binlog日志文件，分成有效事件行的方式返回，并可使用limit指定pos点的起始偏移，查询条数；
      
      A.查询第一个(最早)的binlog日志：
        mysql> show binlog events\G; 
    
      B.指定查询 mysql-bin.000021 这个文件：
        mysql> show binlog events in 'mysql-bin.000021'\G;
    
      C.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起：
        mysql> show binlog events in 'mysql-bin.000021' from 8224\G;
    
      D.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，查询10条
        mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 10\G;
    
      E.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，偏移2行，查询10条
        mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 2,10\G;


五、恢复 binlog 日志实验(zyyshop 是数据库)
    1.假设现在是凌晨 4:00，我的计划任务开始执行一次完整的数据库备份：

      将zyyshop数据库备份到 /root/BAK.zyyshop.sql 文件中：
      # /usr/local/mysql/bin/mysqldump -uroot -p123456 -lF --log-error=/root/myDump.err -B zyyshop > /root/BAK.zyyshop.sql
        ......
    
        大约过了若干分钟，备份完成了，我不用担心数据丢失了，因为我有备份了，嘎嘎~~~
    
      由于我使用了-F选项，当备份工作刚开始时系统会刷新log日志，产生新的binlog日志来记录备份之后的数据库“增删改”操作，查看一下：
      mysql> show master status;
      +------------------+----------+--------------+------------------+
      | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
      +------------------+----------+--------------+------------------+
      | mysql-bin.000023 |      120 |              |                  |
      +------------------+----------+--------------+------------------+
      也就是说， mysql-bin.000023 是用来记录4:00之后对数据库的所有“增删改”操作。


    2.早9:00上班了，业务的需求会对数据库进行各种“增删改”操作~~~~~~~
      @ 比如：创建一个学生表并插入、修改了数据等等：
        CREATE TABLE IF NOT EXISTS `tt` (
          `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
          `name` varchar(16) NOT NULL,
          `sex` enum('m','w') NOT NULL DEFAULT 'm',
          `age` tinyint(3) unsigned NOT NULL,
          `classid` char(6) DEFAULT NULL,
          PRIMARY KEY (`id`)
         ) ENGINE=InnoDB DEFAULT CHARSET=utf8;


      导入实验数据
      mysql> insert into zyyshop.tt(`name`,`sex`,`age`,`classid`) values('yiyi','w',20,'cls1'),('xiaoer','m',22,'cls3'),('zhangsan','w',21,'cls5'),('lisi','m',20,'cls4'),('wangwu','w',26,'cls6');


      查看数据
      mysql> select * from zyyshop.tt;
      +----+----------+-----+-----+---------+
      | id | name     | sex | age | classid |
      +----+----------+-----+-----+---------+
      |  1 | yiyi     | w   |  20 | cls1    |
      |  2 | xiaoer   | m   |  22 | cls3    |
      |  3 | zhangsan | w   |  21 | cls5    |
      |  4 | lisi     | m   |  20 | cls4    |
      |  5 | wangwu   | w   |  26 | cls6    |
      +----+----------+-----+-----+---------+


      中午时分又执行了修改数据操作
      mysql> update zyyshop.tt set name='李四' where id=4;
      mysql> update zyyshop.tt set name='小二' where id=2;
    
      修改后的结果：
      mysql> select * from zyyshop.tt;
      +----+----------+-----+-----+---------+
      | id | name     | sex | age | classid |
      +----+----------+-----+-----+---------+
      |  1 | yiyi     | w   |  20 | cls1    |
      |  2 | 小二     | m   |  22 | cls3    |
      |  3 | zhangsan | w   |  21 | cls5    |
      |  4 | 李四     | m   |  20 | cls4    |
      |  5 | wangwu   | w   |  26 | cls6    |
      +----+----------+-----+-----+---------+


      假设此时是下午18:00，莫名地执行了一条悲催的SQL语句，整个数据库都没了：
      mysql> drop database zyyshop;


    3.此刻杯具了，别慌！先仔细查看最后一个binlog日志，并记录下关键的pos点，到底是哪个pos点的操作导致了数据库的破坏(通常在最后几步)；
    
      备份一下最后一个binlog日志文件：
      # ll /usr/local/mysql/data | grep mysql-bin
      # cp -v /usr/local/mysql/data/mysql-bin.000023 /root/
    
      此时执行一次刷新日志索引操作，重新开始新的binlog日志记录文件，理论说 mysql-bin.000023 这个文件不会再有后续写入了(便于我们分析原因及查找pos点)，以后所有数据库操作都会写入到下一个日志文件；
      mysql> flush logs;
      mysql> show master status;


    4.读取binlog日志，分析问题
      方式一：使用mysqlbinlog读取binlog日志：
        # /usr/local/mysql/bin/mysqlbinlog  /usr/local/mysql/data/mysql-bin.000023
    
      方式二：登录服务器，并查看(推荐)：
        mysql> show binlog events in 'mysql-bin.000023';
        
        以下为末尾片段：
        +------------------+------+------------+-----------+-------------+------------------------------------------------------------+
        | Log_name         | Pos  | Event_type | Server_id | End_log_pos | Info                                                       |
        +------------------+------+------------+-----------+-------------+------------------------------------------------------------+
        | mysql-bin.000023 |  922 | Xid        |         1 |         953 | COMMIT /* xid=3820 */                                      |
        | mysql-bin.000023 |  953 | Query      |         1 |        1038 | BEGIN                                                      |
        | mysql-bin.000023 | 1038 | Query      |         1 |        1164 | use `zyyshop`; update zyyshop.tt set name='李四' where id=4|
        | mysql-bin.000023 | 1164 | Xid        |         1 |        1195 | COMMIT /* xid=3822 */                                      |
        | mysql-bin.000023 | 1195 | Query      |         1 |        1280 | BEGIN                                                      |
        | mysql-bin.000023 | 1280 | Query      |         1 |        1406 | use `zyyshop`; update zyyshop.tt set name='小二' where id=2|
        | mysql-bin.000023 | 1406 | Xid        |         1 |        1437 | COMMIT /* xid=3823 */                                      |
        | mysql-bin.000023 | 1437 | Query      |         1 |        1538 | drop database zyyshop                                      |
        +------------------+------+------------+-----------+-------------+------------------------------------------------------------+
    
        通过分析，造成数据库破坏的pos点区间是介于 1437--1538 之间，只要恢复到1437前就可。


    5.现在把凌晨备份的数据恢复：
      
      # /usr/local/mysql/bin/mysql -uroot -p123456 -v < /root/BAK.zyyshop.sql;
    
      注: 至此截至当日凌晨(4:00)前的备份数据都恢复了。
          但今天一整天(4:00--18:00)的数据肿么办呢？就得从前文提到的 mysql-bin.000023 新日志做文章了......


    6.从binlog日志恢复数据
      
      恢复语法格式：
      # mysqlbinlog mysql-bin.0000xx | mysql -u用户名 -p密码 数据库名
    
        常用选项：
          --start-position=953                   起始pos点
          --stop-position=1437                   结束pos点
          --start-datetime="2013-11-29 13:18:54" 起始时间点
          --stop-datetime="2013-11-29 13:21:53"  结束时间点
          --database=zyyshop                     指定只恢复zyyshop数据库(一台主机上往往有多个数据库，只限本地log日志)
            
        不常用选项：    
          -u --user=name              Connect to the remote server as username.连接到远程主机的用户名
          -p --password[=name]        Password to connect to remote server.连接到远程主机的密码
          -h --host=name              Get the binlog from server.从远程主机上获取binlog日志
          --read-from-remote-server   Read binary logs from a MySQL server.从某个MySQL服务器上读取binlog日志
    
      小结：实际是将读出的binlog日志内容，通过管道符传递给mysql命令。这些命令、文件尽量写成绝对路径；
    
      A.完全恢复(本例不靠谱，因为最后那条 drop database zyyshop 也在日志里，必须想办法把这条破坏语句排除掉，做部分恢复)
        # /usr/local/mysql/bin/mysqlbinlog  /usr/local/mysql/data/mysql-bin.000021 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop 
    
      B.指定pos结束点恢复(部分恢复)：
        @ --stop-position=953 pos结束点
        注：此pos结束点介于“导入实验数据”与更新“name='李四'”之间，这样可以恢复到更改“name='李四'”之前的“导入测试数据”
        # /usr/local/mysql/bin/mysqlbinlog --stop-position=953 --database=zyyshop /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop
      
        在另一终端登录查看结果(成功恢复了)：
        mysql> select * from zyyshop.tt;
        +----+----------+-----+-----+---------+
        | id | name     | sex | age | classid |
        +----+----------+-----+-----+---------+
        |  1 | yiyi     | w   |  20 | cls1    |
        |  2 | xiaoer   | m   |  22 | cls3    |
        |  3 | zhangsan | w   |  21 | cls5    |
        |  4 | lisi     | m   |  20 | cls4    |
        |  5 | wangwu   | w   |  26 | cls6    |
        +----+----------+-----+-----+---------+
    
      C.指定pso点区间恢复(部分恢复)：
        更新 name='李四' 这条数据，日志区间是Pos[1038] --> End_log_pos[1164]，按事务区间是：Pos[953] --> End_log_pos[1195]；
    
        更新 name='小二' 这条数据，日志区间是Pos[1280] --> End_log_pos[1406]，按事务区间是：Pos[1195] --> End_log_pos[1437]；
    
        c1.单独恢复 name='李四' 这步操作，可这样：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=1038 --stop-position=1164 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop
    
           也可以按事务区间单独恢复，如下：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=953 --stop-position=1195 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop


        c2.单独恢复 name='小二' 这步操作，可这样：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=1280 --stop-position=1406 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop
    
           也可以按事务区间单独恢复，如下：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=1195 --stop-position=1437 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop


        c3.将 name='李四'、name='小二' 多步操作一起恢复，需要按事务区间，可这样：
           # /usr/local/mysql/bin/mysqlbinlog --start-position=953 --stop-position=1437 --database=zyyshop  /usr/local/mysql/data/mysql-bin.000023 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop


      D.在另一终端登录查看目前结果(两名称也恢复了)：
        mysql> select * from zyyshop.tt;
        +----+----------+-----+-----+---------+
        | id | name     | sex | age | classid |
        +----+----------+-----+-----+---------+
        |  1 | yiyi     | w   |  20 | cls1    |
        |  2 | 小二     | m   |  22 | cls3    |
        |  3 | zhangsan | w   |  21 | cls5    |
        |  4 | 李四     | m   |  20 | cls4    |
        |  5 | wangwu   | w   |  26 | cls6    |
        +----+----------+-----+-----+---------+
    
      E.也可指定时间区间恢复(部分恢复)：除了用pos点的办法进行恢复，也可以通过指定时间区间进行恢复，按时间恢复需要用mysqlbinlog命令读取binlog日志内容，找时间节点。
        比如，我把刚恢复的tt表删除掉，再用时间区间点恢复
        mysql> drop table tt;
    
        @ --start-datetime="2013-11-29 13:18:54"  起始时间点
        @ --stop-datetime="2013-11-29 13:21:53"   结束时间点
    
        # /usr/local/mysql/bin/mysqlbinlog --start-datetime="2013-11-29 13:18:54" --stop-datetime="2013-11-29 13:21:53" --database=zyyshop /usr/local/mysql/data/mysql-bin.000021 | /usr/local/mysql/bin/mysql -uroot -p123456 -v zyyshop
    
      总结：所谓恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已。