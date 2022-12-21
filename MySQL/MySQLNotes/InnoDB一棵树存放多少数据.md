## 一、文件结构

我们知道，InnoDB 引擎是支持事务的，所以表里的数据肯定都是存储在磁盘上的。如果在 test 数据库下创建两个表：t1 和 t2，那么在相应的数据目录下就会发现两个文件。

```bash
[root@localhost test]# ls
db.opt  t1.frm  t1.ibd  t2.frm  t2.ibd
[root@localhost test]# pwd
/var/lib/mysql/test
```

其中，frm 文件是表结构信息，ibd 文件是表中的数据。

表结构信息包含 MySQL 表的元数据（例如表定义）的文件，比如表名、表有多少列、列的数据类型啥的，不重要，我们先不管；

ibd 文件存储的是表中的数据，比如数据行和索引。这个文件比较重要，它是今天我们的重点研究对象。

我们说，MySQL 表里的数据都是存放在磁盘上的。**在磁盘上最小单元是扇区，每个扇区可以存放 512 个字节的数据；操作系统中最小单元是块（block），最小单位是 4kb。**

在Windows系统中，我们可以通过`fsutil fsinfo ntfsinfo c:`来查看。

```c
C:\Windows\system32>fsutil fsinfo ntfsinfo c:
NTFS 卷序列号:             0x78f40b2cf40aec66
NTFS 版本:                 3.1
LFS 版本:                  2.0
扇区数量:                  0x000000001bcb6fff
簇总数:                    0x0000000003796dff
可用簇:                    0x0000000000a63a03
保留总数:                  0x00000000000017c3
每个扇区字节数:            512
每个物理扇区字节数:        4096
每个簇字节数:              4096
每个 FileRecord 段字节数:  1024
每个 FileRecord 段簇数:    0
```

在Linux系统上，可以通过以下两个命令查看，这取决于文件系统的格式。

```shell
xfs_growfs /dev/mapper/centos-root | grep bsize
tune2fs -l /dev/mapper/centos-root | grep Block
```

MySQL 的 InnoDB 存储引擎它也是有最小存储单位的，叫做页（Page），默认大小是 16kb。

我们新创建一个表 t3，里面任何数据都没有，我们来看它的ibd文件。

```sql
[root@localhost test]# ll
总用量 18579600
-rw-r-----. 1 mysql mysql          67 11月 30 20:59 db.opt
-rw-r-----. 1 mysql mysql       12756 12月  7 21:10 t1.frm
-rw-r-----. 1 mysql mysql 13077839872 12月  7 21:37 t1.ibd
-rw-r-----. 1 mysql mysql        8608 12月  7 21:43 t2.frm
-rw-r-----. 1 mysql mysql  5947523072 12月  7 21:52 t2.ibd
-rw-r-----. 1 mysql mysql       12756 12月  8 21:02 t3.frm
-rw-r-----. 1 mysql mysql       98304 12月  8 21:02 t3.ibd
```

不仅是 t3，我们看到，任何表的 ibd 文件大小，它永远是 16k 的整数倍。理解这个事非常重要，MySQL 从磁盘加载数据是按照页来读取的，即便你查询一条数据，它也会读取一页 16k 的数据出来。

## 二、聚簇索引

**数据库表中的数据都是存储在页里的**，那么这一个页可以存放多少条记录呢？

这取决于一行记录的大小是多少，假如一行数据大小是 1k，那么理论上一页就可以放 16 条数据。

当然，查询数据的时候，MySQL 也不能把所有的页都遍历一遍，所以就有了索引，InnoDB 存储引擎用 B+ 树的方式来构建索引。

聚簇索引就是按照每张表的主键构造一颗 B+ 树，叶子节点存放的是整行记录数据，**在非叶子节点上存放的是键值以及指向数据页的指针，同时每个数据页之间都通过一个双向链表来进行链接。**

![img](InnoDB%E4%B8%80%E6%A3%B5%E6%A0%91%E5%AD%98%E6%94%BE%E5%A4%9A%E5%B0%91%E6%95%B0%E6%8D%AE.resource/e3af8906b5ed4b3086d95ae5fb76f281tplv-k3u1fbpfcp-zoom-in-crop-mark4536000.png)

如上图所示，就是一颗聚簇索引树的大致结构。它先将数据记录按照主键排序，放在不同的页中，**下面一行是数据页。上面的非叶子节点，存放主键值和一个指向页的指针。**

当我们通过主键来查询的时候，比如 `id=6` 的条件，就是通过这颗 B+ 树来查找数据的过程。它先找到根页面（page offset=3），然后通过二分查找，定位到 `id=6` 的数据在指针为 5 的页上。然后进一步的去 page offset=5 的页面上加载数据。

在这里，我们需要理解两件事：

上图中 B+ 树的根节点（page offset=3），是固定不会变化的。只要表创建了聚簇索引，它的`根节点`页号就被记录到某个地方了。还有一点，B+ 树索引本身并不能直接找到具体的一条记录，只能知道该记录在哪个页上，数据库会把页载入到内存，再通过二分查找定位到具体的记录。

现在我们知道了 InnoDB 存储引擎最小存储单元是页，在 B+ 树索引结构里，页可以放一行一行的数据（叶子节点），也可以放主键+指针（非叶子节点）。

上面已经说过，假如一行数据大小是 1k，那么理论上一页就可以放 16 条数据。那一页可以放多少主键+指针呢？

假如我们的主键 id 为 bigint 类型，长度为 8 字节，而指针大小在 InnoDB 源码中设置为 6 字节。这样算下来就是 16384 / 14 = 1170，就是说一个页上可以存放 1170 个指针。

一个指针指向一个存放记录的页，一个页里可以放 16 条数据，那么一颗高度为 2 的 B+ 树就可以存放 1170 * 16=18720 条数据。同理，高度为 3 的 B+ 树，就可以存放 1170 * 1170 * 16 = 21902400 条记录。

理论上就是这样，在 InnoDB 存储引擎中，B+ 树的高度一般为 2-4 层，就可以满足千万级数据的存储。查找数据的时候，一次页的查找代表一次 IO，那我们通过主键索引查询的时候，其实最多只需要 2-4 次 IO 就可以了。

那么，实际上到底是不是这样呢？我们接着往下看。

## 三、页的类型

在开始验证之前，我们不仅需要了解页，还需要知道，在 InnoDB 引擎中，页并不是只有一种。常见的页类型有：

- 数据页，B-tree Node；
- undo页，undo Log Page；
- 系统页，System Page；
- 事务数据页，Transaction system Page；
- 插入缓冲位图页，Insert Buffer Bitmap；
- 插入缓冲空闲列表页，Insert Buffer Free List；
- 未压缩的二进制大对象页，Uncompressed BLOB Page；

在这里我们重点来看 B-tree Node，我们的索引和数据就放在这种页上。既然有不同的页类型，我们怎么知道当前的页属于什么页呢？

那么我们就需要大概了解下数据页的结构，数据页由七个部分组成，每个部分都有不同的含义。

- File Header，文件头，固定38字节；
- Page Header，页头，固定56字节；
- Infimum + supremum，固定26字节；
- User Records，用户记录，即行记录，大小不固定；
- Free Space，空闲空间，大小不固定；
- Page Directort，页目录，大小不固定；
- File Trailer，文件结尾信息，固定8字节。

其中，File Header 用来记录页的一些头信息，共占用 38 个字节。在这个头信息里，我们可以获取到该页在表空间里的偏移值和这个数据页的类型。

接下来是 Page Header，它记录的是数据页的状态信息，共占用 56 个字节。在这一部分，我们可以获取到两个重要的信息：该页中记录的数量和当前页在索引树的层级，其中 0x00 代表叶子节点，比如聚簇索引中的叶子节点放的就是整行数据，它总是在第 0 层。

## 四、验证

前面我们已经说过，ibd 文件就是表数据文件。这个文件会随着数据库表里数据的增长而增长，不过它始终会是 16k 的整数倍。里面就是一个个的页，那我们就可以一个一个页的来解析，通过文件头可以判断它是什么页，找到 B-tree Node，就可以看到里面的 Page Level，它的值 +1，就代表了当前 B+ 树的高度。

我们现在就来重新创建一个表，为了使这个表中的数据一行大小为 1k，我们设置几个 char(255) 的字段即可。

```sql
CREATE TABLE `t5` (
  `id` bigint(8) NOT NULL,
  `c1` char(245) NOT NULL DEFAULT '1',
  `c2` char(255) NOT NULL DEFAULT '1',
  `c3` char(255) NOT NULL DEFAULT '1',
  `c4` char(255) NOT NULL DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

然后笔者写了一个存储过程，用来批量插入数据用的，为了加快批量插入的速度，笔者还修改了 `innodb_flush_log_at_trx_commit=0`，切记生产环境可不要这样玩。

```sql
BEGIN
    DECLARE i int DEFAULT 0;
    select ifnull(max(id),0) into i from t5;
    set i = i+1;
    WHILE i <= 100000 DO
        insert into t5(id)value(i);
        set i = i+1;
    END WHILE;
END
```

`innodbPageInfo.jar` 是笔者用 Java 代码写的一个工具类，用来输出 ibd 文件中，页的信息。

```
-path 后面是文件的路径，-v 是否显示页的详情信息，0是 1否。
```

上面我们创建了 t5 这张表，一条数据还没有的情况下，我们看一下这个 ibd 文件的信息。

```sql
[root@localhost innodbInfo]# java -jar innodbPageInfo.jar -path /var/lib/mysql/test/t5.ibd -v 0
page offset 00000000,page type <File Space Header>
page offset 00000001,page type <Insert Buffer Bitmap>
page offset 00000002,page type <File Segment inode>
page offset 00000003,page type <B-tree Node>,page level <0000>
page offset 00000000,page type <Freshly Allocated Page>
page offset 00000000,page type <Freshly Allocated Page>
数据页总记录数:0
Total number of page: 6
Insert Buffer Bitmap: 1
File Segment inode: 1
B-tree Node: 1
File Space Header: 1
Freshly Allocated Page: 2
[root@localhost innodbInfo]# 
```

t5 表现在没有任何数据，它的 ibd 文件大小是 98304，也就是说一共有 6 个页。其中第四个页（page offset 3）是数据页，page level 等于 0，代表该页为叶子节点。因为目前还没有数据，可以认为 B+ 树的索引只有 1 层。

我们接着插入 10 条数据，这个 page level 还是为 0，B+ 树的高度还是 1，这是因为一个页大约能存放 16 条大小为 1k 的数据。

```typescript
page offset 00000003,page type <B-tree Node>,page level <0000>
数据页总记录数:10
Total number of page: 6
```

当我们插入 15 条数据的时候，一个页就放不下了，原本为新分配的页（Freshly Allocated Page）就会变成数据页，原来的根页面（page offset=3）就会升级成存储目录项的页。offset 04 和 05 就变成了叶子节点的数据页，所以现在整个 B+ 树的高度为 2。

```typescript
page offset 00000003,page type <B-tree Node>,page level <0001>
page offset 00000004,page type <B-tree Node>,page level <0000>
page offset 00000005,page type <B-tree Node>,page level <0000>
数据页总记录数:15
Total number of page: 6
```

继续插入 10000 条数据，我们再来看一下 B+ 树高的情况。当然现在信息比较多了，我们把输出结果写到文件里。

```
java -jar innodbPageInfo.jar -path /var/lib/mysql/test/t5.ibd -v 0 > t5.txt
```

截取部分结果如下：

```typescript
[root@localhost innodbInfo]# vim t5.txt 
page offset 00000003,page type <B-tree Node>,page level <0001>
page offset 00000004,page type <B-tree Node>,page level <0000>
page offset 00000005,page type <B-tree Node>,page level <0000>
page offset 00000000,page type <Freshly Allocated Page>
数据页总记录数:10000
Total number of page: 1216
B-tree Node: 716
```

可以看到，1 万条 1k 大小的记录，一共用了 716 个数据页，根页面显示的树高还是 2 层。

前面我们计算过，2 层的 B+ 树理论上可以存放 18000 条左右，笔者测试大约 13000 条数据左右，B+ 树就会成为 3 层了。

```yaml
page offset 00000003,page type <B-tree Node>,page level <0002>
数据页总记录数:13000
Total number of page: 1472
B-tree Node: 933
```

原因也不难理解，因为每个页不可能只放数据本身。

首先每个页都有一些固定的格式，比如文件头部、页面头部、文件尾部这些，我们的数据放在`用户记录`这部分里的；

其次，用户记录也不只放数据行，每个数据行还有一些其他标记，比如是否删除、最小记录、记录数、在堆中的位置信息、记录的类型、下一条记录的相对位置等等；

另外，MySQL 参考手册中也有说到，InnoDB 会保留页的1/16空闲，以便将来插入或者更新索引使用，如果主键 id 不是顺序插入的，那可能还不是1/16，会占用更多的空闲空间。

总之，我们理解一个页不会全放数据就行了。所以，实测跟理论上不一致也是完全正常的，因为上面的理论没有排除这些项。

接着来，我们再插入 1000 万条数据，现在 ibd 文件已经达到 11GB。

```less
page offset 00000003,page type <B-tree Node>,page level <0002>
数据页总记录数:10000000
Total number of page: 725760
B-tree Node: 715059
```

我们看到，1千万条数据，数据页已经有 71 万个，B+ 树的高度还是3层，这也就是说几万条数据和一千万条数据的查询效率基本上是一样的。 比如我们现在根据主键 ID 查询一条数据，`select * from t5 where id = 6548215;` ，查询时间显示用了 0.010 秒。

什么时候会到 4 层呢？大概在 1300 万左右，B+ 树就会增加树高到 4 层。

什么时候会到 5 层呢？笔者没测试出来，因为插入到 5000 万条数据的时候，ibd 数据文件大小已经 55G了，虚拟机已经空间不足了。。

```less
page offset 00000003,page type <B-tree Node>,page level <0003>
数据页总记录数:50000000
B-tree Node: 3575286
```

即便是5000万条数据，我们通过主键ID查询，查询时间也是毫秒级的。

理论上要达到十亿 - 百亿行数据，树高才能到 5 层。如果有小伙伴用这种方法，测试出来 5 层高的数据，欢迎在评论区留言，让我看看。

另外，朋友们有没有意识到一个问题？其实**影响 B+ 树树高的因素，不仅是数据行，还有主键ID的长度**。我们上面的测试中，ID 的类型是bigint(8)，在其他字段长度均不变的情况下，我们把 ID 的类型改为int(4)，相同的树高就会容纳更多的数据，因为它单个页能承载的指针数变多了。

```sql
CREATE TABLE `t6` (
  `id` int(4) NOT NULL,
  `c1` char(245) NOT NULL DEFAULT '1',
  `c2` char(255) NOT NULL DEFAULT '1',
  `c3` char(255) NOT NULL DEFAULT '1',
  `c4` char(255) NOT NULL DEFAULT '1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

针对t6这张表，我们插入16000条数据，然后输出一下页面信息。

```less
page offset 00000003,page type <B-tree Node>,page level <0001>
数据页总记录数:16000
B-tree Node: 1145
```

我们来看，如果按照主键ID类型bigint(8)来测试，13000条数据的时候，树高就已经是3了，现在改为int(4)，16000条数据，树高依然还是2层。尽管数据页（B-tree Node）数量还是那么多，变化并不大，但是它不影响树高。

ok，看到这里，相信朋友们对开头提出的问题已经有自己的答案了，如果你也跟着试一遍，理解可能会更加深入。

看到这，还有道经典的面试题：为什么MySQL的索引要使用B+树而不是其它树形结构?比如B树？

简单来说，其中有一个原因就是B+树的高度比较稳定，因为它的非叶子节点不会保存数据，只保存键值和指针的情况下，一个页能承载大量的数据。你想啊，B树它的非叶子节点也会保存数据的，同样的一行数据大小是1kb，那么它一页最多也只能保存16个指针，在大量数据的情况下，树高就会速度膨胀，导致IO次数就会很多，查询就会变得很慢。

## 源码地址

本文的`innodbPageInfo.jar`代码是笔者参考 `MySQL技术内幕（InnoDB存储引擎）`一书中的工具包，书里作者是用Python写的，所以笔者在这里用Java重新实现了一遍。

Java版本的源码我放在GitHub上了：[github.com/taoxun/inno…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ftaoxun%2FinnodbPageInfo)

已经打完包的Jar版本，也可以下载：[pan.baidu.com/s/1IZVJRNUk…](https://link.juejin.cn/?target=https%3A%2F%2Fpan.baidu.com%2Fs%2F1IZVJRNUk_bPESp5zoQwOvA) 提取码:5rnz。

朋友们可以拿这个工具看一看，自己认为较大的表，它的B+树索引到底有几层？

**参考资料：**

姜承尧：《MySQL技术内幕:InnoDB存储引擎》

天涯泪小武：[tianyalei.blog.csdn.net/article/det…](https://link.juejin.cn/?target=https%3A%2F%2Ftianyalei.blog.csdn.net%2Farticle%2Fdetails%2F100015840)

飘扬的红领巾：[www.cnblogs.com/leefreeman/…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.cnblogs.com%2Fleefreeman%2Fp%2F8315844.html)

MySQL官方参考手册：[dev.mysql.com/doc/refman/…](https://link.juejin.cn/?target=https%3A%2F%2Fdev.mysql.com%2Fdoc%2Frefman%2F5.7%2Fen%2F)