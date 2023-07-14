# Leaf——美团点评分布式ID生成系统

原文：https://tech.meituan.com/2017/04/21/mt-leaf.html

Github：https://github.com/Meituan-Dianping/Leaf/blob/master/README_CN.md

## 一、背景

在复杂分布式系统中，往往需要对大量的数据和消息进行唯一标识。如在美团点评的金融、支付、餐饮、酒店、猫眼电影等产品的系统中，数据日渐增长，对数据分库分表后需要有一个唯一 ID 来标识一条数据或消息，数据库的自增 ID 显然不能满足需求；特别一点的如订单、骑手、优惠券也都需要有唯一 ID 做标识。此时一个能够生成全局唯一 ID 的系统是非常必要的。概括下来，那业务系统对 ID 号的要求有哪些呢？

- 全局唯一性：不能出现重复的 ID 号，既然是唯一标识，这是最基本的要求。

- 趋势递增：在 MySQL InnoDB 引擎中使用的是聚集索引，由于多数 RDBMS 使用 B-tree 的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键保证写入性能。

- 单调递增：保证下一个 ID 一定大于上一个 ID，例如事务版本号、IM 增量消息、排序等特殊需求。

- 信息安全：如果 ID 是连续的，恶意用户的扒取工作就非常容易做了，直接按照顺序下载指定 URL 即可；如果是订单号就更危险了，竞对可以直接知道我们一天的单量。所以在一些应用场景下，会需要 ID 无规则、不规则。

上述 123 对应三类不同的场景，3 和 4 需求还是互斥的，无法使用同一个方案满足。

同时除了对 ID 号码自身的要求，业务还对 ID 号生成系统的可用性要求极高，想象一下，如果 ID 生成系统瘫痪，整个美团点评支付、优惠券发券、骑手派单等关键动作都无法执行，这就会带来一场灾难。

由此总结下一个 ID 生成系统应该做到如下几点：

- 平均延迟和 TP999 延迟都要尽可能低；

- 可用性 5 个 9；

- 高 QPS。

## 二、常见方法介绍

### （一）UUID

UUID(Universally Unique Identifier)的标准型式包含 32 个（不包括 

`-` 共计 32 个，） 16 进制数字，以连字号分为五段，形式为 8-4-4-4-12 的 36 个字符，示例：`550e8400-e29b-41d4-a716-446655440000`，到目前为止业界一共有 5 种方式生成 UUID，详情见 IETF 发布的 UUID 规范 [A Universally Unique IDentifier (UUID) URN Namespace](http://www.ietf.org/rfc/rfc4122.txt)。

优点：

- 性能非常高：本地生成，没有网络消耗。

缺点：

- 不易于存储：UUID 太长，16 字节 128 位，通常以 36 长度（包括 `-`）的字符串表示，很多场景不适用。

- 信息不安全：基于 MAC 地址生成 UUID 的算法可能会造成 MAC 地址泄露，这个漏洞曾被用于寻找梅丽莎病毒的制作者位置。

- ID 作为主键时在特定的环境会存在一些问题，比如做 DB 主键的场景下，UUID 就非常不适用：

    - MySQL 官方有明确的建议主键要尽量越短越好[4]，36 个字符长度的 UUID 不符合要求。

    > All indexes other than the clustered index are known as secondary indexes. In InnoDB, each record in a secondary index contains the primary key columns for the row, as well as the columns specified for the secondary index. InnoDB uses this primary key value to search for the row in the clustered index.*** If the primary key is long, the secondary indexes use more space, so it is advantageous to have a short primary key***.

    - 对 MySQL 索引不利：如果作为数据库主键，在 InnoDB 引擎下，UUID 的无序性可能会引起数据位置频繁变动，严重影响性能。

### （二）类snowflake方案

这种方案大致来说是一种以划分命名空间（UUID 也算，由于比较常见，所以单独分析）来生成 ID 的一种算法，这种方案把 64-bit 分别划分成多段，分开来标示机器、时间等，比如在 snowflake 中的 64-bit 分别表示如下图（图片来自网络）所示：

![image-20230216221313364](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221313364.png)

41-bit 的时间可以表示（1L<<41）/(1000L*3600*24*365)=69 年的时间，10-bit 机器可以分别表示 1024 台机器。如果我们对 IDC 划分有需求，还可以将 10-bit 分 5-bit 给 IDC，分 5-bit 给工作机器。这样就可以表示 32 个 IDC，每个 IDC 下可以有 32 台机器，可以根据自身需求定义。12 个自增序列号可以表示 2^12 个ID，理论上 snowflake 方案的 QPS 约为 409.6w/s，这种分配方式可以保证在任何一个 IDC 的任何一台机器在任意毫秒内生成的 ID 都是不同的。

这种方式的优缺点是：

优点：

- 毫秒数在高位，自增序列在低位，整个 ID 都是趋势递增的。
- 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成 ID 的性能也是非常高的。
- 可以根据自身业务特性分配 bit 位，非常灵活。

缺点：

- **强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态**。

#### 应用举例Mongdb objectID

[MongoDB官方文档 ObjectID](https://docs.mongodb.com/manual/reference/method/ObjectId/#description)可以算作是和 snowflake 类似方法，通过“时间+机器码+pid+inc”共12个字节，通过4+3+2+3的方式最终标识成一个24长度的十六进制字符。

### （三）数据库生成

以 MySQL 举例，利用给字段设置 `auto_increment_increment` 和 `auto_increment_offset` 来保证 ID 自增，每次业务使用下列 SQL 读写 MySQL 得到 ID 号。

```sql
begin;
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();
commit;
```

![image-20230216221327771](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221327771.png)

这种方案的优缺点如下：

优点：

- 非常简单，利用现有数据库系统的功能实现，成本小，有 DBA 专业维护。
- ID 号单调自增，可以实现一些对 ID 有特殊要求的业务。

缺点：

- 强依赖 DB，当 DB 异常时整个系统不可用，属于致命问题。配置主从复制可以尽可能的增加可用性，但是数据一致性在特殊情况下难以保证。主从切换时的不一致可能会导致重复发号。
- **ID 发号性能瓶颈限制在单台 MySQL 的读写性能**。

对于 MySQL 性能问题，可用如下方案解决：在分布式系统中我们可以多部署几台机器，**每台机器设置不同的初始值，且步长和机器数相等**。比如有两台机器。设置步长 step 为 2，TicketServer1 的初始值为 1（1，3，5，7，9，11…）、TicketServer2 的初始值为 2（2，4，6，8，10…）。这是 Flickr 团队在 2010 年撰文介绍的一种主键生成策略（[Ticket Servers: Distributed Unique Primary Keys on the Cheap ](http://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)）。如下所示，为了实现上述方案分别设置两台机器对应的参数，TicketServer1 从 1 开始发号，TicketServer2 从 2 开始发号，两台机器每次发号之后都递增 2。

```sql
TicketServer1:
auto-increment-increment = 2
auto-increment-offset = 1

TicketServer2:
auto-increment-increment = 2
auto-increment-offset = 2
```

假设我们要部署 N 台机器，步长需设置为 N，每台的初始值依次为 0,1,2…N-1 那么整个架构就变成了如下图所示：

![image-20230216221341815](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221341815.png)

这种架构貌似能够满足性能的需求，但有以下几个缺点：

- 系统水平扩展比较困难，比如定义好了步长和机器台数之后，如果要添加机器该怎么做？假设现在只有一台机器发号是 1,2,3,4,5（步长是1），这个时候需要扩容机器一台。可以这样做：把第二台机器的初始值设置得比第一台超过很多，比如 14（假设在扩容时间之内第一台不可能发到14），同时设置步长为2，那么这台机器下发的号码都是14以后的偶数。然后摘掉第一台，把ID值保留为奇数，比如7，然后修改第一台的步长为2。让它符合我们定义的号段标准，对于这个例子来说就是让第一台以后只能产生奇数。扩容方案看起来复杂吗？貌似还好，现在想象一下如果我们线上有100台机器，这个时候要扩容该怎么做？简直是噩梦。所以系统水平扩展方案复杂难以实现。
- **ID 没有了单调递增的特性，只能趋势递增，这个缺点对于一般业务需求不是很重要，可以容忍。**
- **数据库压力还是很大，每次获取 ID 都得读写一次数据库，只能靠堆机器来提高性能**。

## 三、Leaf方案实现

Leaf 这个名字是来自德国哲学家、数学家莱布尼茨的一句话： >There are no two identical leaves in the world > “世界上没有两片相同的树叶”

综合对比上述几种方案，每种方案都不完全符合我们的要求。所以 **Leaf 分别在上述第二种和第三种方案上做了相应的优化，实现了 Leaf-segment 和 Leaf-snowflake 方案**。

### （一）Leaf-segment数据库方案

第一种 Leaf-segment 方案，在使用数据库的方案上，做了如下改变： 

- 原方案每次获取 ID 都得读写一次数据库，造成数据库压力大。改为利用 proxy server 批量获取，每次获取一个 segment(step 决定大小)号段的值。用完之后再去数据库获取新的号段，可以大大的减轻数据库的压力。
- 各个业务不同的发号需求用 biz_tag 字段来区分，每个 biz-tag 的 ID 获取相互隔离，互不影响。如果以后有性能需求需要对数据库扩容，不需要上述描述的复杂的扩容操作，只需要对 biz_tag 分库分表就行。

数据库表设计如下：

```sql
+-------------+--------------+------+-----+-------------------+-----------------------------+
| Field       | Type         | Null | Key | Default           | Extra                       |
+-------------+--------------+------+-----+-------------------+-----------------------------+
| biz_tag     | varchar(128) | NO   | PRI |                   |                             |
| max_id      | bigint(20)   | NO   |     | 1                 |                             |
| step        | int(11)      | NO   |     | NULL              |                             |
| desc        | varchar(256) | YES  |     | NULL              |                             |
| update_time | timestamp    | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------------+--------------+------+-----+-------------------+-----------------------------+
```

重要字段说明：biz_tag 用来区分业务，max_id 表示该 biz_tag 目前所被分配的 ID 号段的最大值，step 表示每次分配的号段长度。原来获取 ID 每次都需要写数据库，现在只需要把 step 设置得足够大，比如 1000。那么只有当 1000 个号被消耗完了之后才会去重新读写一次数据库。读写数据库的频率从 1 减小到了 1/step，大致架构如下图所示：

![image-20230216221400255](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221400255.png)

test_tag 在第一台 Leaf 机器上是 `1~1000` 的号段，当这个号段用完时，会去加载另一个长度为 step=1000 的号段，假设另外两台号段都没有更新，这个时候第一台机器新加载的号段就应该是3001~4000。同时数据库对应的 biz_tag 这条数据的 max_id 会从 3000 被更新成 4000，更新号段的SQL语句如下：

```sql
Begin
UPDATE table SET max_id=max_id+step WHERE biz_tag=xxx
SELECT tag, max_id, step FROM table WHERE biz_tag=xxx
Commit
```

这种模式有以下优缺点：

优点：

- Leaf 服务可以很方便的线性扩展，性能完全能够支撑大多数业务场景。
- ID 号码是**趋势递增**的 8byte 的 64 位数字，满足上述数据库存储的主键要求。
- 容灾性高：Leaf 服务内部有号段缓存，即使 DB 宕机，短时间内 Leaf 仍能正常对外提供服务。
- 可以自定义 max_id 的大小，非常方便业务从原有的 ID 方式上迁移过来。

缺点：

- ID 号码不够随机，能够泄露发号数量的信息，不太安全。
- TP999 数据波动大，当号段使用完之后还是会hang在更新数据库的I/O上，tg999数据会出现偶尔的尖刺。
- DB 宕机会造成整个系统不可用。

### 双buffer优化 

对于第二个缺点，Leaf-segment 做了一些优化，简单的说就是：

Leaf 取号段的时机是在号段消耗完的时候进行的，也就意味着号段临界点的ID下发时间取决于下一次从DB取回号段的时间，并且在这期间进来的请求也会因为DB号段没有取回来，导致线程阻塞。如果请求DB的网络和DB的性能稳定，这种情况对系统的影响是不大的，但是假如取DB的时候网络发生抖动，或者DB发生慢查询就会导致整个系统的响应时间变慢。

为此，我们希望DB取号段的过程能够做到无阻塞，不需要在DB取号段的时候阻塞请求线程，即当号段消费到某个点时就异步的把下一个号段加载到内存中。而不需要等到号段用尽的时候才去更新号段。这样做就可以很大程度上的降低系统的TP999指标。详细实现如下图所示：

![image-20230216221412645](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221412645.png)

采用双buffer的方式，Leaf服务内部有两个号段缓存区segment。当前号段已下发10%时，如果下一个号段未更新，则另启一个更新线程去更新下一个号段。当前号段全部下发完后，如果下个号段准备好了则切换到下个号段为当前segment接着下发，循环往复。

- 每个biz-tag都有消费速度监控，通常推荐segment长度设置为服务高峰期发号QPS的600倍（10分钟），这样即使DB宕机，Leaf仍能持续发号10-20分钟不受影响。
- 每次请求来临时都会判断下个号段的状态，从而更新此号段，所以偶尔的网络抖动不会影响下个号段的更新。

### Leaf高可用容灾 

对于第三点“DB可用性”问题，我们目前采用一主两从的方式，同时分机房部署，Master和Slave之间采用**半同步方式[5]**同步数据。同时使用公司Atlas数据库中间件（已开源，改名为[DBProxy](https://github.com/Meituan-Dianping/DBProxy)）做主从切换。当然这种方案在一些情况会退化成异步模式，甚至在**非常极端**情况下仍然会造成数据不一致的情况，但是出现的概率非常小。如果你的系统要保证100%的数据强一致，可以选择使用“类Paxos算法”实现的强一致MySQL方案，如MySQL 5.7前段时间刚刚GA的[MySQL Group Replication](https://dev.mysql.com/doc/refman/5.7/en/group-replication.html)。但是运维成本和精力都会相应的增加，根据实际情况选型即可。

![image-20230216221421697](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221421697.png)

同时Leaf服务分IDC部署，内部的服务化框架是“MTthrift RPC”。服务调用的时候，根据负载均衡算法会优先调用同机房的Leaf服务。在该IDC内Leaf服务不可用的时候才会选择其他机房的Leaf服务。同时服务治理平台OCTO还提供了针对服务的过载保护、一键截流、动态流量分配等对服务的保护措施。

## 四、Leaf-snowflake 方案

Leaf-segment 方案可以生成趋势递增的 ID，同时 ID 号是可计算的，不适用于订单 ID 生成场景，比如竞对在两天中午 12 点分别下单，通过订单 id 号相减就能大致计算出公司一天的订单量，这个是不能忍受的。面对这一问题，我们提供了 Leaf-snowflake 方案。

![image-20230216221430078](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221430078.png)

Leaf-snowflake 方案完全沿用 snowflake 方案的 bit 位设计，即是“1+41+10+12”的方式组装 ID 号。对于 workerID 的分配，当服务集群数量较小的情况下，完全可以手动配置。Leaf 服务规模较大，动手配置成本太高。所以使用 Zookeeper 持久顺序节点的特性自动对 snowflake 节点配置 wokerID。Leaf-snowflake 是按照下面几个步骤启动的：

1. 启动 Leaf-snowflake 服务，连接 Zookeeper，在 leaf_forever 父节点下检查自己是否已经注册过（是否有该顺序子节点）。
2. 如果有注册过直接取回自己的 workerID（zk 顺序节点生成的 int 类型 ID 号），启动服务。
3. 如果没有注册过，就在该父节点下面创建一个持久顺序节点，创建成功后取回顺序号当做自己的 workerID 号，启动服务。

![image-20230216221438133](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221438133.png)

### 弱依赖ZooKeeper

除了每次会去 ZK 拿数据以外，也会在本机文件系统上缓存一个 workerID 文件。当 ZooKeeper 出现问题，恰好机器出现问题需要重启时，能保证服务能够正常启动。这样做到了对三方组件的弱依赖。一定程度上提高了 SLA。

### 解决时钟问题

因为这种方案依赖时间，如果机器的时钟发生了回拨，那么就会有可能生成重复的 ID 号，需要解决时钟回退的问题。

![image-20230216221448143](meituan_Leaf%E2%80%94%E2%80%94%E7%BE%8E%E5%9B%A2%E7%82%B9%E8%AF%84%E5%88%86%E5%B8%83%E5%BC%8FID%E7%94%9F%E6%88%90%E7%B3%BB%E7%BB%9F.resource/image-20230216221448143.png)

参见上图整个启动流程图，服务启动时首先检查自己是否写过ZooKeeper leaf_forever节点：

1. 若写过，则用自身系统时间与 `leaf_forever/${self}` 节点记录时间做比较，若小于 `leaf_forever/${self}` 时间则认为机器时间发生了大步长回拨，服务启动失败并报警。
2. 若未写过，证明是新服务节点，直接创建持久节点 `leaf_forever/${self}` 并写入自身系统时间，接下来综合对比其余 Leaf 节点的系统时间来判断自身系统时间是否准确，具体做法是取`leaf_temporary` 下的所有临时节点(所有运行中的 Leaf-snowflake 节点)的服务IP：Port，然后通过RPC请求得到所有节点的系统时间，计算 sum(time)/nodeSize。
3. 若abs( 系统时间-sum(time)/nodeSize ) < 阈值，认为当前系统时间准确，正常启动服务，同时写临时节点 `leaf_temporary/${self}` 维持租约。
4. 否则认为本机系统时间发生大步长偏移，启动失败并报警。
5. 每隔一段时间(3s)上报自身系统时间写入 `leaf_forever/${self}`。

由于强依赖时钟，对时间的要求比较敏感，在机器工作时 NTP 同步也会造成秒级别的回退，建议可以直接关闭 NTP 同步。要么在时钟回拨的时候直接不提供服务直接返回 ERROR_CODE，等时钟追上即可。**或者做一层重试，然后上报报警系统，更或者是发现有时钟回拨之后自动摘除本身节点并报警**，如下：

```java
 //发生了回拨，此刻时间小于上次发号时间
 if (timestamp < lastTimestamp) {
  			  
            long offset = lastTimestamp - timestamp;
            if (offset <= 5) {
                try {
                	//时间偏差大小小于5ms，则等待两倍时间
                    wait(offset << 1);//wait
                    timestamp = timeGen();
                    if (timestamp < lastTimestamp) {
                       //还是小于，抛异常并上报
                        throwClockBackwardsEx(timestamp);
                      }    
                } catch (InterruptedException e) {  
                   throw  e;
                }
            } else {
                //throw
                throwClockBackwardsEx(timestamp);
            }
        }
 //分配ID       
        
```

**从上线情况来看，在 2017 年闰秒出现那一次出现过部分机器回拨，由于 Leaf-snowflake 的策略保证，成功避免了对业务造成的影响。**

## Leaf现状

Leaf在美团点评公司内部服务包含金融、支付交易、餐饮、外卖、酒店旅游、猫眼电影等众多业务线。目前 Leaf 的性能在 4C8G 的机器上 QPS 能压测到近 5万/s，TP999 1ms，已经能够满足大部分的业务的需求。每天提供亿数量级的调用量，作为公司内部公共的基础技术设施，必须保证高 SLA 和高性能的服务，我们目前还仅仅达到了及格线，还有很多提高的空间。

## 作者简介

照东，美团点评基础架构团队成员，主要参与美团[大型分布式链路跟踪系统Mtrace](https://tech.meituan.com/2016/10/14/mt-mtrace.html)和美团点评分布式 ID 生成系统 Leaf 的开发工作。曾就职于阿里巴巴，2016年7月加入美团。

**招聘**：如果你对大规模分布式环境下的服务治理、分布式会话链追踪等系统感兴趣，诚挚欢迎投递简历至：zhangjinlu#meituan.com。

## 参考资料

1. 施瓦茨. 高性能MySQL[M]. 电子工业出版社, 2010:162-171.
2. [维基百科：UUID](https://zh.wikipedia.org/wiki/通用唯一识别码).
3. [snowflake](https://github.com/twitter/snowflake).
4. [MySQL: Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html).
5. [半同步复制 Semisynchronous Replication](https://dev.mysql.com/doc/refman/5.5/en/replication-semisync.html).