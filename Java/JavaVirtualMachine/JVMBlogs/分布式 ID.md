带你了解「美团、百度和滴滴」的分布式 ID 生成系统

博客地址：https://blog.csdn.net/qq_35246620/article/details/109099078?spm=1001.2014.3001.5501

CG国斌 2020-10-16 21:15:50  734  收藏 5
文章标签： Leaf uid-generator Tinyid ID生成器 开源项目
版权

史上最简单的 MySQL 教程
超..超...超......简单的 MySQL 数据库教程。
CG国斌
¥9.90
订阅博主
文章目录
美团
背景
常见方法介绍
UUID
类snowflake方案
数据库生成
Leaf 方案实现
Leaf-segment 数据库方案
双 buffer 优化
Leaf 高可用容灾
Leaf-snowflake 方案
弱依赖 ZooKeeper
解决时钟问题
Leaf 现状
百度
snowflake
DefaultUidGenerator
delta seconds
worker id
sequence
小结
CachedUidGenerator
RingBuffer Of Flag
RingBuffer Of UID
worker id
初始化
取值
小结
滴滴
简介
性能与可用性
性能
可用性
Tinyid 的特性
推荐使用方式
Tinyid 的原理
ID 生成系统要点
Tinyid 的实现原理
DB 号段算法描述
号段生成方案的简单架构
简单架构的问题
优化办法
美团
背景
在复杂分布式系统中，往往需要对大量的数据和消息进行唯一标识。如在美团点评的金融、支付、餐饮、酒店、猫眼电影等产品的系统中，数据日渐增长，对数据分库分表后需要有一个唯一 ID 来标识一条数据或消息，数据库的自增 ID 显然不能满足需求；特别一点的如订单、骑手、优惠券也都需要有唯一 ID 做标识。此时一个能够生成全局唯一 ID 的系统是非常必要的。概括下来，那业务系统对 ID 号的要求有哪些呢？

全局唯一性：不能出现重复的 ID 号，既然是唯一标识，这是最基本的要求。
趋势递增：在 MySQL InnoDB 引擎中使用的是聚集索引，由于多数 RDBMS 使用 B-tree 的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键保证写入性能。
单调递增：保证下一个 ID 一定大于上一个 ID，例如事务版本号、IM 增量消息、排序等特殊需求。
信息安全：如果 ID 是连续的，恶意用户的扒取工作就非常容易做了，直接按照顺序下载指定 URL 即可；如果是订单号就更危险了，竞对可以直接知道我们一天的单量。所以在一些应用场景下，会需要 ID 无规则、不规则。
上述 1、2、3 对应三类不同的场景，3 和 4 需求还是互斥的，无法使用同一个方案满足。

同时除了对 ID 号码自身的要求，业务还对 ID 号生成系统的可用性要求极高，想象一下，如果 ID 生成系统瘫痪，整个美团点评支付、优惠券发券、骑手派单等关键动作都无法执行，这就会带来一场灾难。

由此总结下一个 ID 生成系统应该做到如下几点：

平均延迟和 TP999 延迟都要尽可能低；
可用性 5 个 9；
高 QPS。
常见方法介绍
UUID
UUID（Universally Unique Identifier）的标准型式包含 32 个 16 进制数字，以连字号分为五段，形式为8-4-4-4-12的 36 个字符，示例：550e8400-e29b-41d4-a716-446655440000，到目前为止业界一共有 5 种方式生成 UUID，详情见 IETF 发布的 UUID 规范 A Universally Unique IDentifier (UUID) URN Namespace。

优点：
性能非常高：本地生成，没有网络消耗。
缺点：
不易于存储：UUID 太长，16 字节 128 位，通常以 36 长度的字符串表示，很多场景不适用。
信息不安全：基于 MAC 地址生成 UUID 的算法可能会造成 MAC 地址泄露，这个漏洞曾被用于寻找梅丽莎病毒的制作者位置。
ID 作为主键时在特定的环境会存在一些问题，比如做 DB 主键的场景下，UUID 就非常不适用：
① MySQL 官方有明确的建议主键要尽量越短越好，36 个字符长度的 UUID 不符合要求。
② 对 MySQL 索引不利：如果作为数据库主键，在 InnoDB 引擎下，UUID 的无序性可能会引起数据位置频繁变动，严重影响性能。
类snowflake方案
这种方案大致来说是一种以划分命名空间（UUID 也算，由于比较常见，所以单独分析）来生成 ID 的一种算法，这种方案把 64-bit 分别划分成多段，分开来标示机器、时间等，比如在 snowflake 中的 64-bit 分别表示如下图（图片来自网络）所示：



41-bit 的时间可以表示(1L<<41)/(1000L*3600*24*365)=69年的时间，10-bit 机器可以分别表示 1024 台机器。如果我们对 IDC 划分有需求，还可以将 10-bit 分 5-bit 给 IDC，分 5-bit 给工作机器。这样就可以表示 32 个 IDC，每个 IDC 下可以有 32 台机器，可以根据自身需求定义。12 个自增序列号可以表示2^12个 ID，理论上 snowflake 方案的 QPS 约为 409.6w/s，这种分配方式可以保证在任何一个 IDC 的任何一台机器在任意毫秒内生成的 ID 都是不同的。

这种方式的优缺点是：

优点：
毫秒数在高位，自增序列在低位，整个 ID 都是趋势递增的。
不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成 ID 的性能也是非常高的。
可以根据自身业务特性分配 bit 位，非常灵活。
缺点：
强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态。
MongoDB 的「ObjectID」可以算作是和 snowflake 类似方法，通过时间+机器码+pid+inc共 12 个字节，通过4+3+2+3的方式最终标识成一个 24 长度的十六进制字符。

数据库生成
以 MySQL 举例，利用给字段设置auto_increment_increment和auto_increment_offset来保证 ID 自增，每次业务使用下列 SQL 读写 MySQL 得到 ID 号。

begin;
REPLACE INTO Tickets64 (stub) VALUES ('a');
SELECT LAST_INSERT_ID();
commit;
1
2
3
4


这种方案的优缺点如下：

优点：
非常简单，利用现有数据库系统的功能实现，成本小，有 DBA 专业维护。
ID 号单调自增，可以实现一些对 ID 有特殊要求的业务。
缺点：
强依赖 DB，当 DB 异常时整个系统不可用，属于致命问题。配置主从复制可以尽可能的增加可用性，但是数据一致性在特殊情况下难以保证。主从切换时的不一致可能会导致重复发号。
ID 发号性能瓶颈限制在单台 MySQL 的读写性能。
对于 MySQL 性能问题，可用如下方案解决：在分布式系统中我们可以多部署几台机器，每台机器设置不同的初始值，且步长和机器数相等。比如有两台机器。设置步长 step 为 2，TicketServer1的初始值为1（1，3，5，7，9，11 …）、TicketServer2的初始值为2（2，4，6，8，10 …）。这是 Flickr 团队在 2010 年撰文介绍的一种主键生成策略（Ticket Servers: Distributed Unique Primary Keys on the Cheap）。如下所示，为了实现上述方案分别设置两台机器对应的参数，TicketServer1从 1 开始发号，TicketServer2从 2 开始发号，两台机器每次发号之后都递增 2。

TicketServer1:
auto-increment-increment = 2
auto-increment-offset = 1

TicketServer2:
auto-increment-increment = 2
auto-increment-offset = 2
1
2
3
4
5
6
7
假设我们要部署 N 台机器，步长需设置为 N，每台的初始值依次为0, 1, 2… N-1那么整个架构就变成了如下图所示：



这种架构貌似能够满足性能的需求，但有以下几个缺点：

系统水平扩展比较困难，比如定义好了步长和机器台数之后，如果要添加机器该怎么做？假设现在只有一台机器发号是1, 2, 3, 4, 5（步长是 1），这个时候需要扩容机器一台。可以这样做：把第二台机器的初始值设置得比第一台超过很多，比如 14（假设在扩容时间之内第一台不可能发到 14），同时设置步长为 2，那么这台机器下发的号码都是 14 以后的偶数。然后摘掉第一台，把 ID 值保留为奇数，比如 7，然后修改第一台的步长为 2。让它符合我们定义的号段标准，对于这个例子来说就是让第一台以后只能产生奇数。扩容方案看起来复杂吗？貌似还好，现在想象一下如果我们线上有 100 台机器，这个时候要扩容该怎么做？简直是噩梦。所以系统水平扩展方案复杂难以实现。
ID 没有了单调递增的特性，只能趋势递增，这个缺点对于一般业务需求不是很重要，可以容忍。
数据库压力还是很大，每次获取 ID 都得读写一次数据库，只能靠堆机器来提高性能。
Leaf 方案实现
Leaf 这个名字是来自德国哲学家、数学家莱布尼茨的一句话：There are no two identical leaves in the world，即“世界上没有两片相同的树叶”。

综合对比上述几种方案，每种方案都不完全符合我们的要求。所以 Leaf 分别在上述第二种和第三种方案上做了相应的优化，实现了 Leaf-segment 和 Leaf-snowflake 方案。

Leaf-segment 数据库方案
第一种 Leaf-segment 方案，在使用数据库的方案上，做了如下改变：

原方案每次获取 ID 都得读写一次数据库，造成数据库压力大。改为利用proxy server批量获取，每次获取一个segment（step 决定大小）号段的值。用完之后再去数据库获取新的号段，可以大大的减轻数据库的压力。
各个业务不同的发号需求用biz_tag字段来区分，每个biz-tag的 ID 获取相互隔离，互不影响。如果以后有性能需求需要对数据库扩容，不需要上述描述的复杂的扩容操作，只需要对biz_tag分库分表就行。
数据库表设计如下：

+-------------+--------------+------+-----+-------------------+-----------------------------+
| Field       | Type         | Null | Key | Default           | Extra                       |
+-------------+--------------+------+-----+-------------------+-----------------------------+
| biz_tag     | varchar(128) | NO   | PRI |                   |                             |
| max_id      | bigint(20)   | NO   |     | 1                 |                             |
| step        | int(11)      | NO   |     | NULL              |                             |
| desc        | varchar(256) | YES  |     | NULL              |                             |
| update_time | timestamp    | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+-------------+--------------+------+-----+-------------------+-----------------------------+
1
2
3
4
5
6
7
8
9
重要字段说明：biz_tag用来区分业务，max_id表示该biz_tag目前所被分配的 ID 号段的最大值，step表示每次分配的号段长度。原来获取 ID 每次都需要写数据库，现在只需要把step设置得足够大，比如 1000。那么只有当 1000 个号被消耗完了之后才会去重新读写一次数据库。读写数据库的频率从 1 减小到了1/step，大致架构如下图所示：



test_tag在第一台 Leaf 机器上是1~1000的号段，当这个号段用完时，会去加载另一个长度为step=1000的号段，假设另外两台号段都没有更新，这个时候第一台机器新加载的号段就应该是3001~4000。同时数据库对应的biz_tag这条数据的max_id会从 3000 被更新成 4000，更新号段的 SQL 语句如下：

Begin
UPDATE table SET max_id=max_id+step WHERE biz_tag=xxx
SELECT tag, max_id, step FROM table WHERE biz_tag=xxx
Commit
1
2
3
4
这种模式有以下优缺点：

优点：

Leaf 服务可以很方便的线性扩展，性能完全能够支撑大多数业务场景。
ID 号码是趋势递增的 8byte 的 64 位数字，满足上述数据库存储的主键要求。
容灾性高：Leaf 服务内部有号段缓存，即使 DB 宕机，短时间内 Leaf 仍能正常对外提供服务。
可以自定义max_id的大小，非常方便业务从原有的 ID 方式上迁移过来。
缺点：

ID 号码不够随机，能够泄露发号数量的信息，不太安全。
TP999 数据波动大，当号段使用完之后还是会 hang 在更新数据库的 I/O 上，TP999 数据会出现偶尔的尖刺。
DB 宕机会造成整个系统不可用。
双 buffer 优化
对于第二个缺点，Leaf-segment 做了一些优化，简单的说就是：

Leaf 取号段的时机是在号段消耗完的时候进行的，也就意味着号段临界点的 ID 下发时间取决于下一次从 DB 取回号段的时间，并且在这期间进来的请求也会因为 DB 号段没有取回来，导致线程阻塞。如果请求 DB 的网络和 DB 的性能稳定，这种情况对系统的影响是不大的，但是假如取 DB 的时候网络发生抖动，或者 DB 发生慢查询就会导致整个系统的响应时间变慢。
为此，我们希望 DB 取号段的过程能够做到无阻塞，不需要在 DB 取号段的时候阻塞请求线程，即当号段消费到某个点时就异步的把下一个号段加载到内存中。而不需要等到号段用尽的时候才去更新号段。这样做就可以很大程度上的降低系统的 TP999 指标。详细实现如下图所示：



采用双 buffer 的方式，Leaf 服务内部有两个号段缓存区segment。当前号段已下发 10% 时，如果下一个号段未更新，则另启一个更新线程去更新下一个号段。当前号段全部下发完后，如果下个号段准备好了则切换到下个号段为当前segment接着下发，循环往复。

每个biz-tag都有消费速度监控，通常推荐segment长度设置为服务高峰期发号 QPS 的 600 倍（10分钟），这样即使 DB 宕机，Leaf 仍能持续发号 10 ~ 20 分钟不受影响。

每次请求来临时都会判断下个号段的状态，从而更新此号段，所以偶尔的网络抖动不会影响下个号段的更新。

Leaf 高可用容灾
对于第三点“DB 可用性”问题，我们目前采用一主两从的方式，同时分机房部署，Master和Slave之间采用半同步方式同步数据。同时使用公司 Atlas 数据库中间件（已开源，改名为DBProxy）做主从切换。当然这种方案在一些情况会退化成异步模式，甚至在非常极端情况下仍然会造成数据不一致的情况，但是出现的概率非常小。如果你的系统要保证 100% 的数据强一致，可以选择使用“类 Paxos 算法”实现的强一致 MySQL 方案，如 MySQL 5.7 前段时间刚刚 GA 的 MySQL Group Replication。但是运维成本和精力都会相应的增加，根据实际情况选型即可。



同时 Leaf 服务分 IDC 部署，内部的服务化框架是“MTthrift RPC”。服务调用的时候，根据负载均衡算法会优先调用同机房的 Leaf 服务。在该 IDC 内 Leaf 服务不可用的时候才会选择其他机房的 Leaf 服务。同时服务治理平台 OCTO 还提供了针对服务的过载保护、一键截流、动态流量分配等对服务的保护措施。

Leaf-snowflake 方案
Leaf-segment 方案可以生成趋势递增的 ID，同时 ID 号是可计算的，不适用于订单 ID 生成场景，比如竞对在两天中午 12 点分别下单，通过订单 ID 号相减就能大致计算出公司一天的订单量，这个是不能忍受的。面对这一问题，我们提供了 Leaf-snowflake 方案。



Leaf-snowflake 方案完全沿用 snowflake 方案的 bit 位设计，即是1+41+10+12的方式组装 ID 号。对于workerID的分配，当服务集群数量较小的情况下，完全可以手动配置。Leaf 服务规模较大，动手配置成本太高。所以使用 Zookeeper 持久顺序节点的特性自动对 snowflake 节点配置wokerID。Leaf-snowflake 是按照下面几个步骤启动的：

启动 Leaf-snowflake 服务，连接 Zookeeper，在leaf_forever父节点下检查自己是否已经注册过（是否有该顺序子节点）。
如果有注册过直接取回自己的workerID（ZK 顺序节点生成的int类型 ID 号），启动服务。
如果没有注册过，就在该父节点下面创建一个持久顺序节点，创建成功后取回顺序号当做自己的workerID号，启动服务。


弱依赖 ZooKeeper
除了每次会去 ZK 拿数据以外，也会在本机文件系统上缓存一个workerID文件。当 ZooKeeper 出现问题，恰好机器出现问题需要重启时，能保证服务能够正常启动。这样做到了对三方组件的弱依赖，一定程度上提高了 SLA。

解决时钟问题
因为这种方案依赖时间，如果机器的时钟发生了回拨，那么就会有可能生成重复的 ID 号，需要解决时钟回退的问题。



参见上图整个启动流程图，服务启动时首先检查自己是否写过 ZooKeeper leaf_forever节点：

若写过，则用自身系统时间与leaf_forever/${self}节点记录时间做比较，若小于leaf_forever/${self}时间则认为机器时间发生了大步长回拨，服务启动失败并报警。
若未写过，证明是新服务节点，直接创建持久节点leaf_forever/${self}并写入自身系统时间，接下来综合对比其余 Leaf 节点的系统时间来判断自身系统时间是否准确，具体做法是取leaf_temporary下的所有临时节点（所有运行中的 Leaf-snowflake 节点）的服务IP:Port，然后通过 RPC 请求得到所有节点的系统时间，计算sum(time)/nodeSize。
若abs(系统时间-sum(time)/nodeSize) < 阈值，认为当前系统时间准确，正常启动服务，同时写临时节点leaf_temporary/${self}维持租约。
否则认为本机系统时间发生大步长偏移，启动失败并报警。
每隔一段时间（3s）上报自身系统时间写入leaf_forever/${self}。
由于强依赖时钟，对时间的要求比较敏感，在机器工作时 NTP 同步也会造成秒级别的回退，建议可以直接关闭 NTP 同步。要么在时钟回拨的时候直接不提供服务直接返回ERROR_CODE，等时钟追上即可。或者做一层重试，然后上报报警系统，更或者是发现有时钟回拨之后自动摘除本身节点并报警，如下：

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
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
从上线情况来看，在 2017 年闰秒出现那一次出现过部分机器回拨，由于 Leaf-snowflake 的策略保证，成功避免了对业务造成的影响。

Leaf 现状
Leaf 在美团点评公司内部服务包含金融、支付交易、餐饮、外卖、酒店旅游、猫眼电影等众多业务线。目前 Leaf 的性能在 4C8G 的机器上 QPS 能压测到近 5w/s，TP999 1ms，已经能够满足大部分的业务的需求。

百度
UidGenerator 是百度开源的 Java 语言实现，基于 Snowflake 算法的唯一 ID 生成器。而且，它非常适合虚拟环境，比如：Docker。另外，它通过消费未来时间克服了雪花算法的并发限制。UidGenerator 提前生成 ID 并缓存在RingBuffer中。 检测结果显示，单个实例的 QPS 能超过 6000,000。

依赖环境：

JDK8+
MySQL（用于分配workerId）
snowflake
由下图可知，雪花算法的几个核心组成部分：

1 位sign标识位
41 位时间戳
10 位workId（数据中心+工作机器，可以其他组成方式）
12 位自增序列

但是百度对这些组成部分稍微调整了一下：


由上图可知，UidGenerator 的时间部分只有 28 位，这就意味着 UidGenerator 默认只能承受 8.5 年（2^28-1/86400/365）。当然，根据你业务的需求，UidGenerator 可以适当调整delta seconds、worker node id和sequence占用位数。

接下来分析百度 UidGenerator 的实现。需要说明的是 UidGenerator 有两种方式提供：DefaultUidGenerator和CachedUidGenerator。我们先分析比较容易理解的DefaultUidGenerator。

DefaultUidGenerator
delta seconds
这个值是指当前时间与epoch时间的时间差，且单位为秒。epoch时间就是指集成 UidGenerator 生成分布式 ID 服务第一次上线的时间，可配置，也一定要根据你的上线时间进行配置，因为默认的epoch时间可是2016-09-20，不配置的话，会浪费好几年的可用时间。

worker id
接下来说一下 UidGenerator 是如何被worker id赋值的，搭建 UidGenerator 的话，需要创建一个表：

DROP DATABASE IF EXISTS `xxxx`;
CREATE DATABASE `xxxx` ;
use `xxxx`;
DROP TABLE IF EXISTS WORKER_NODE;
CREATE TABLE WORKER_NODE
(
ID BIGINT NOT NULL AUTO_INCREMENT COMMENT 'auto increment id',
HOST_NAME VARCHAR(64) NOT NULL COMMENT 'host name',
PORT VARCHAR(64) NOT NULL COMMENT 'port',
TYPE INT NOT NULL COMMENT 'node type: ACTUAL or CONTAINER',
LAUNCH_DATE DATE NOT NULL COMMENT 'launch date',
MODIFIED TIMESTAMP NOT NULL COMMENT 'modified time',
CREATED TIMESTAMP NOT NULL COMMENT 'created time',
PRIMARY KEY(ID)
)
 COMMENT='DB WorkerID Assigner for UID Generator',ENGINE = INNODB;
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
UidGenerator 会在集成用它生成分布式 ID 的实例启动的时候，往这个表中插入一行数据，得到的 ID 值就是准备赋给workerId的值。由于workerId默认 22 位，那么，集成 UidGenerator 生成分布式 ID 的所有实例重启次数是不允许超过 4194303 次（即2^22-1），否则会抛出异常。

这段逻辑的核心代码来自
DisposableWorkerIdAssigner.java中，当然，你也可以实现WorkerIdAssigner.java接口，自定义生成workerId。

sequence
核心代码如下，几个实现的关键点：

synchronized保证线程安全
如果时间有任何的回拨，那么直接抛出异常
如果当前时间和上一次是同一秒时间，那么sequence自增。如果同一秒内自增值超过2^13-1，那么就会自旋等待下一秒（getNextSecond）
如果是新的一秒，那么sequence重新从 0 开始
    protected synchronized long nextId() {
        long currentSecond = getCurrentSecond();

        // Clock moved backwards, refuse to generate uid
        if (currentSecond < lastSecond) {
            long refusedSeconds = lastSecond - currentSecond;
            throw new UidGenerateException("Clock moved backwards. Refusing for %d seconds", refusedSeconds);
        }
    
        // At the same second, increase sequence
        if (currentSecond == lastSecond) {
            sequence = (sequence + 1) & bitsAllocator.getMaxSequence();
            // Exceed the max sequence, we wait the next second to generate uid
            if (sequence == 0) {
                currentSecond = getNextSecond(lastSecond);
            }
    
        // At the different second, sequence restart from zero
        } else {
            sequence = 0L;
        }
    
        lastSecond = currentSecond;
    
        // Allocate bits for UID
        return bitsAllocator.allocate(currentSecond - epochSeconds, workerId, sequence);
    }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
小结
通过DefaultUidGenerator的实现可知，它对时钟回拨的处理比较简单粗暴。另外如果使用 UidGenerator 的DefaultUidGenerator方式生成分布式 ID，一定要根据你的业务的情况和特点，调整各个字段占用的位数：

        <property name="timeBits" value="29"/>
        <property name="workerBits" value="21"/>
        <property name="seqBits" value="13"/>
        <property name="epochStr" value="2016-09-20"/>
1
2
3
4
CachedUidGenerator
CachedUidGenerator是 UidGenerator 的重要改进实现。它的核心利用了RingBuffer，如下图所示，它本质上是一个数组，数组中每个项被称为slot。UidGenerator 设计了两个RingBuffer，一个保存唯一 ID，一个保存flag。RingBuffer的尺寸是2^n，n必须是正整数：



RingBuffer Of Flag
其中，保存flag这个RingBuffer的每个slot的值都是 0 或者 1，0 是CANPUTFLAG的标志位，1 是CANTAKEFLAG的标识位。每个slot的状态要么是CANPUT，要么是CANTAKE。以某个slot的值为例，初始值为 0，即CANPUT。接下来会初始化填满这个RingBuffer，这时候这个slot的值就是 1，即CANTAKE。等获取分布式 ID 时取到这个slot的值后，这个slot的值又变为 0，以此类推。

RingBuffer Of UID
保存唯一 ID 的RingBuffer有两个指针，Tail指针和Cursor指针。

Tail指针表示最后一个生成的唯一 ID。如果这个指针对上了Cursor指针，意味着RingBuffer已经满了。这时候，不允许再继续生成 ID 了。用户可以通过属性rejectedPutBufferHandler指定处理这种情况的策略。

Cursor指针表示最后一个已经给消费的唯一 ID。如果Cursor指针追上了Tail指针，意味着RingBuffer已经空了。这时候，不允许再继续获取 ID 了。用户可以通过属性rejectedTakeBufferHandler指定处理这种异常情况的策略。

另外，如果你想增强RingBuffer提升它的吞吐能力，那么需要配置一个更大的boostPower值：

		<!-- 默认:3， 原bufferSize=8192, 扩容后bufferSize= 8192 << 3 = 65536 -->
		<property name="boostPower" value="3"></property>
1
2
CachedUidGenerator的理论讲完后，接下来就是它具体是如何实现的了，我们首先看它的声明，它是实现了DefaultUidGenerator，所以，它事实上就是对DefaultUidGenerator的增强：

public class CachedUidGenerator extends DefaultUidGenerator implements DisposableBean {
    private static final Logger LOGGER = LoggerFactory.getLogger(CachedUidGenerator.class);
    private static final int DEFAULT_BOOST_POWER = 3;
    ....
}
1
2
3
4
5
worker id
CachedUidGenerator的workerId实现继承自它的父类DefaultUidGenerator，即实例启动时往表WORKER_NODE插入数据后得到的自增 ID 值。

接下来深入解读CachedUidGenerator的核心操作，即对RingBuffer的操作，包括初始化、取分布式唯一 ID、填充分布式唯一 ID等。

初始化
CachedUidGenerator在初始化时除了给workerId赋值，还会初始化RingBuffer。这个过程主要工作有：

根据boostPower的值确定RingBuffer的size
构造RingBuffer，默认paddingFactor为50。这个值的意思是当RingBuffer中剩余可用 ID 数量少于 50% 的时候，就会触发一个异步线程往RingBuffer中填充新的唯一 ID（调用BufferPaddingExecutor中的paddingBuffer()方法，这个线程中会有一个标志位running控制并发问题），直到填满为止
判断是否配置了属性scheduleInterval，这是另外一种RingBuffer填充机制, 在Schedule线程中, 周期性检查填充。默认：不配置，即不使用Schedule线程；如需使用，请指定Schedule线程时间间隔，单位为秒
初始化put操作拒绝策略，对应属性rejectedPutBufferHandler。即当RingBuffer已满, 无法继续填充时的操作策略。默认无需指定，将丢弃put操作，仅日志记录； 如有特殊需求，请实现RejectedPutBufferHandler接口（支持Lambda表达式）
初始化take操作拒绝策略，对应属性rejectedTakeBufferHandler。即当环已空，无法继续获取时的操作策略。默认无需指定，将记录日志，并抛出UidGenerateException异常；如有特殊需求，请实现RejectedTakeBufferHandler接口
初始化填满RingBuffer中所有slot（即塞满唯一 ID，这一步和第 2 步骤一样都是调用BufferPaddingExecutor中的paddingBuffer()方法）
开启buffer补丁线程（前提是配置了属性scheduleInterval），原理就是利用ScheduledExecutorService的scheduleWithFixedDelay()方法
说明：第二步的异步线程实现非常重要，也是 UidGenerator 解决时钟回拨的关键：在满足填充新的唯一 ID 条件时，通过时间值递增得到新的时间值（
lastSecond.incrementAndGet()），而不是System.currentTimeMillis()这种方式，而lastSecond是AtomicLong类型，所以能保证线程安全问题。

取值
RingBuffer初始化有值后，接下来的取值就简单了。不过，由于分布式 ID 都保存在RingBuffer中，取值过程中就会有一些逻辑判断：

如果剩余可用 ID 百分比低于paddingFactor参数指定值，就会异步生成若干个 ID 集合，直到将RingBuffer填满。
如果获取值的位置对上了tail指针，就会执行take操作的拒绝策略。
获取slot中的分布式 ID。
将这个slot的标志位置为CANPUTFLAG。
小结
通过上面对 UidGenerator 的分析可知，CachedUidGenerator方式主要通过采取如下一些措施和方案规避了时钟回拨问题和增强唯一性：

自增列：UidGenerator的workerId在实例每次重启时初始化，且就是数据库的自增 ID，从而完美的实现每个实例获取到的workerId不会有任何冲突。
RingBuffer：UidGenerator 不再在每次取 ID 时都实时计算分布式 ID，而是利用RingBuffer数据结构预先生成若干个分布式 ID 并保存。
时间递增：传统的雪花算法实现都是通过System.currentTimeMillis()来获取时间并与上一次时间进行比较，这样的实现严重依赖服务器的时间。而 UidGenerator 的时间类型是AtomicLong，且通过incrementAndGet()方法获取下一次的时间，从而脱离了对服务器时间的依赖，也就不会有时钟回拨的问题。
实际上，这种做法也有一个小问题，即分布式 ID 中的时间信息可能并不是这个 ID 真正产生的时间点，例如：获取的某分布式 ID 的值为3200169789968523265，它的反解析结果为{"timestamp":"2019-05-02 23:26:39","workerId":"21","sequence":"1"}，但是这个 ID 可能并不是在2019-05-02 23:26:39这个时间产生的。

滴滴
简介
Tinyid 是用 Java 开发的一款分布式 ID 生成系统，基于数据库号段算法实现，关于这个算法可以参考美团 Leaf 或者 Tinyid 原理介绍。Tinyid 扩展了 leaf-segment 算法，支持了多 DB（master），同时提供了java-client（sdk）使 ID 生成本地化，获得了更好的性能与可用性。Tinyid 在滴滴客服部门使用，均通过tinyid-client方式接入，每天生成亿级别的 ID。Tinyid 的架构如下图所示：



下面是一些关于这个架构图的说明:

nextId和getNextSegmentId是tinyid-server对外提供的两个http接口
nextId是获取下一个 ID，当调用nextId时，会传入bizType，每个bizType的 ID 数据是隔离的，生成 ID 会使用该bizType类型生成的IdGenerator
getNextSegmentId是获取下一个可用号段，tinyid-client会通过此接口来获取可用号段
IdGenerator是 ID 生成的接口
IdGeneratorFactory是生产具体IdGenerator的工厂，每个bizType生成一个IdGenerator实例。通过工厂，我们可以随时在 DB 中新增bizType，而不用重启服务
IdGeneratorFactory实际上有两个子类IdGeneratorFactoryServer和IdGeneratorFactoryClient，区别在于getNextSegmentId的不同，一个是DbGet,一个是HttpGet
CachedIdGenerator则是具体的 ID 生成器对象，持有currentSegmentId和nextSegmentId对象，负责nextId的核心流程。nextId最终通过AtomicLong.andAndGet(delta)方法产生
性能与可用性
性能
http 方式访问，性能取决于http server的能力，网络传输速度
java-client方式，ID 为本地生成，号段长度（step）越长，QPS 越大，如果将号段设置足够大，则 QPS 可达 1000w+
可用性
依赖 DB，当 DB 不可用时，因为server有缓存，所以还可以使用一段时间，如果配置了多个 DB，则只要有 1 个 DB 存活，则服务可用
使用tiny-client，只要server有一台存活，则理论上可用，server全挂，因为client有缓存，也可以继续使用一段时间
Tinyid 的特性
全局唯一的long型 ID
趋势递增的 ID，即不保证下一个 ID 一定比上一个大
非连续性
提供http和java-client方式接入
支持批量获取ID
支持生成1, 3, 5, 7, 9…序列的 ID
支持多个 DB 的配置，无单点
适用场景：只关心 ID 是数字，趋势递增的系统，可以容忍 ID 不连续，有浪费的场景

不适用场景：类似订单 ID 的业务，因为生成的 ID大部分是连续的，容易被扫库、或者测算出订单量

推荐使用方式
tinyid-server推荐部署到多个机房的多台机器
多机房部署可用性更高，http方式访问需使用方考虑延迟问题
推荐使用tinyid-client来获取 ID，好处如下:
ID 为本地生成（调用AtomicLong.addAndGet方法），性能大大增加
client对server访问变的低频，减轻了server的压力
因为低频，即便client使用方和server不在一个机房，也无须担心延迟
即便所有server挂掉，因为client预加载了号段，依然可以继续使用一段时间
注意：使用tinyid-client方式，如果client机器较多频繁重启，可能会浪费较多的 ID，这时可以考虑使用http方式
推荐 DB 配置两个或更多:
DB 配置多个时，只要有 1 个 DB 存活，则服务可用
多 DB 配置，如配置了两个 DB，则每次新增业务需在两个 DB 中都写入相关数据
Tinyid 的原理
ID 生成系统要点
在简单系统中，我们常常使用 DB 的 ID 自增方式来标识和保存数据，随着系统的复杂，数据的增多，分库分表成为了常见的方案，DB 自增已无法满足要求。这时候全局唯一的 ID 生成系统就派上了用场。当然这只是 ID 生成其中的一种应用场景。那么 DB 生成系统有哪些要求呢？

全局唯一的 ID：无论怎样都不能重复，这是最基本的要求了
高性能：基础服务尽可能耗时少，如果能够本地生成最好
高可用：虽说很难实现 100% 的可用性，但是也要无限接近于 100% 的可用性
简单易用：能够拿来即用，接入方便，同时在系统设计和实现上要尽可能的简单
Tinyid 的实现原理
我们先来看一下最常见的 ID 生成方式，DB 的auto_increment，相信大家都非常熟悉，我也见过一些同学在实战中使用这种方案来获取一个 ID，这个方案的优点是简单，缺点是每次只能向 DB 获取一个 ID，性能比较差，对 DB 访问比较频繁，DB 的压力会比较大。那么是不是可以对这种方案优化一下呢，可否一次向 DB 获取一批 ID 呢？答案当然是可以的。

一批 ID，我们可以看成是一个 ID 范围，例如(1000,2000]，这个 1000 到 2000 也可以称为一个"号段"，我们一次向 DB 申请一个号段，加载到内存中，然后采用自增的方式来生成 ID，这个号段用完后，再次向 DB 申请一个新的号段，这样对 DB 的压力就减轻了很多，同时内存中直接生成 ID，性能则提高了很多。那么保存 DB 号段的表该怎设计呢？

DB 号段算法描述
id	start_id	end_id
1	1000	2000
如上表，我们很容易想到的是 DB 直接存储一个范围(start_id,end_id]，当这批 DB 使用完毕后，我们做一次update操作，update start_id=2000(end_id), end_id=3000(end_id+1000)，update成功了，则说明获取到了下一个 ID 范围。

仔细想想，实际上start_id并没有起什么作用，新的号段总是(end_id,end_id+1000]。所以这里我们更改一下，DB 设计应该是这样的：

id	biz_type	max_id	step	version
1	1000	2000	1000	0
这里我们增加了biz_type，这个代表业务类型，不同的业务的id隔离
max_id则是上面的end_id了，代表当前最大的可用id
step代表号段的长度，可以根据每个业务的 QPS 来设置一个合理的长度
version是一个乐观锁，每次更新都加上version，能够保证并发更新的正确性
那么我们可以通过如下几个步骤来获取一个可用的号段，

查询当前的max_id信息：select id, biz_type, max_id, step, version from tiny_id_info where biz_type='test'
计算新的max_id：new_max_id = max_id + step
更新 DB 中的max_id：update tiny_id_info set max_id=#{new_max_id} , verison=version+1 where id=#{id} and max_id=#{max_id} and version=#{version}
如果更新成功，则可用号段获取成功，新的可用号段为(max_id, new_max_id]
如果更新失败，则号段可能被其他线程获取，回到步骤 1，进行重试
号段生成方案的简单架构
如上我们已经完成了号段生成逻辑，那么我们的 ID 生成服务架构可能是这样的：



ID 生成系统向外提供http服务，请求经过我们的负载均衡router，到达其中一台tinyid-server，从事先加载好的号段中获取一个 ID，如果号段还没有加载，或者已经用完，则向 ID 再申请一个新的可用号段，多台server之间因为号段生成算法的原子性，而保证每台server上的可用号段不重，从而使 ID 生成不重。

可以看到如果tinyid-server如果重启了，那么号段就作废了，会浪费一部分 ID；同时 ID 也不会连续；每次请求可能会打到不同的机器上，ID 也不是单调递增的，而是趋势递增的，不过这对于大部分业务都是可接受的。

简单架构的问题
到此一个简单的 ID 生成系统就完成了，那么是否还存在问题呢？回想一下我们最开始的id生成系统要求，高性能、高可用、简单易用，在上面这套架构里，至少还存在以下问题:

当 ID 用完时需要访问 DB 加载新的号段，DB 更新也可能存在version冲突，此时 ID 生成耗时明显增加
DB 是一个单点，虽然 DB 可以建设主从等高可用架构，但始终是一个单点
使用http方式获取一个 ID，存在网络开销，性能和可用性都不太好
优化办法
双号段缓存

对于号段用完需要访问 DB，我们很容易想到在号段用到一定程度的时候，就去异步加载下一个号段，保证内存中始终有可用号段，则可避免性能波动。

增加多 DB 支持

DB 只有一个master时，如果 DB 不可用（down 掉或者主从延迟比较大），则获取号段不可用。实际上我们可以支持多个 DB，比如 2 个 DB，A 和 B，我们获取号段可以随机从其中一台上获取。那么如果 A，B 都获取到了同一号段，我们怎么保证生成的 ID 不重呢？Tinyid 是这么做的，让 A 只生成偶数 ID，B 只生产奇数 ID，对应的 DB 设计增加了两个字段，如下所示

id	biz_type	max_id	step	delta	remainder	version
1	1000	2000	1000	2	0	0
其中，delta代表 ID 每次的增量，remainder代表余数，例如可以将 A，B 都delta都设置 2，remainder分别设置为 0 和 1，则 A 的号段只生成偶数号段，B 是奇数号段。通过delta和remainder两个字段我们可以根据使用方的需求灵活设计 DB 个数，同时也可以为使用方提供只生产类似奇数的 ID 序列。

增加 tinyid-client

使用http获取一个 ID，存在网络开销，是否可以本地生成 ID？为此我们提供了tinyid-client，我们可以向tinyid-server发送请求来获取可用号段，之后在本地构建双号段、ID 生成，如此 ID 生成则变成纯本地操作，性能大大提升，因为本地有双号段缓存，则可以容忍tinyid-server一段时间的 down 掉，可用性也有了比较大的提升。

Tinyid 最终架构

最终我们的架构可能是这样的：



Tinyid 提供http和tinyid-client两种方式接入
tinyid-server内部缓存两个号段
号段基于 DB 生成，具有原子性
DB 支持多个
tinyid-server内置easy-router选择 DB
原文链接：

Leaf——美团点评分布式ID生成系统
百度开源的分布式唯一ID生成器UidGenerator，解决了时钟回拨问题
太赞了！滴滴开源了一套分布式ID的生成系统…
开源项目地址：

美团 Leaf 在 GitHub 上的开源项目仓库
百度 uid-generator 在 GitHub 上的开源项目仓库
滴滴 Tinyid 在 GitHub 上的开源项目仓库
------------------------------------------------
版权声明：本文为CSDN博主「CG国斌」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/qq_35246620/article/details/109099078