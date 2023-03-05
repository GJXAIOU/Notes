# 16 \| 高性能NoSQL

关系数据库经过几十年的发展后已经非常成熟，强大的SQL功能和ACID的属性，使得关系数据库广泛应用于各式各样的系统中，但这并不意味着关系数据库是完美的，关系数据库存在如下缺点。

- 关系数据库存储的是行记录，无法存储数据结构

以微博的关注关系为例，“我关注的人”是一个用户ID列表，使用关系数据库存储只能将列表拆成多行，然后再查询出来组装，无法直接存储一个列表。

- 关系数据库的schema扩展很不方便

关系数据库的表结构schema是强约束，操作不存在的列会报错，业务变化时扩充列也比较麻烦，需要执行DDL（data definition language，如CREATE、ALTER、DROP等）语句修改，而且修改时可能会长时间锁表（例如，MySQL可能将表锁住1个小时）。

- 关系数据库在大数据场景下I/O较高

如果对一些大量数据的表进行统计之类的运算，关系数据库的I/O会很高，因为即使只针对其中某一列进行运算，关系数据库也会将整行数据从存储设备读入内存。

- 关系数据库的全文搜索功能比较弱

关系数据库的全文搜索只能使用like进行整表扫描匹配，性能非常低，在互联网这种搜索复杂的场景下无法满足业务要求。

针对上述问题，分别诞生了不同的NoSQL解决方案，这些方案与关系数据库相比，在某些应用场景下表现更好。但世上没有免费的午餐，NoSQL方案带来的优势，本质上是牺牲ACID中的某个或者某几个特性，**因此我们不能盲目地迷信NoSQL是银弹，而应该将NoSQL作为SQL的一个有力补充**，NoSQL != No SQL，而是NoSQL = Not Only SQL。

常见的NoSQL方案分为4类。

- K-V存储：解决关系数据库无法存储数据结构的问题，以Redis为代表。

- 文档数据库：解决关系数据库强schema约束的问题，以MongoDB为代表。

- 列式数据库：解决关系数据库大数据场景下的I/O问题，以HBase为代表。

- 全文搜索引擎：解决关系数据库的全文搜索性能问题，以Elasticsearch为代表。

今天，我来介绍一下各种高性能NoSQL方案的典型特征和应用场景。

## K-V存储

K-V存储的全称是Key-Value存储，其中Key是数据的标识，和关系数据库中的主键含义一样，Value就是具体的数据。

Redis是K-V存储的典型代表，它是一款开源（基于BSD许可）的高性能K-V缓存和存储系统。Redis的Value是具体的数据结构，包括string、hash、list、set、sorted set、bitmap和hyperloglog，所以常常被称为数据结构服务器。

以List数据结构为例，Redis提供了下面这些典型的操作（更多请参考链接：[http://redis.cn/commands.html#list](<http://redis.cn/commands.html#list>)）：

- LPOP key从队列的左边出队一个元素。

- LINDEX key index获取一个元素，通过其索引列表。

- LLEN key获得队列（List）的长度。

- RPOP key从队列的右边出队一个元素。

以上这些功能，如果用关系数据库来实现，就会变得很复杂。例如，LPOP操作是移除并返回 key对应的list的第一个元素。如果用关系数据库来存储，为了达到同样目的，需要进行下面的操作：

- 每条数据除了数据编号（例如，行ID），还要有位置编号，否则没有办法判断哪条数据是第一条。注意这里不能用行ID作为位置编号，因为我们会往列表头部插入数据。

- 查询出第一条数据。

- 删除第一条数据。

- 更新从第二条开始的所有数据的位置编号。

可以看出关系数据库的实现很麻烦，而且需要进行多次SQL操作，性能很低。

Redis的缺点主要体现在并不支持完整的ACID事务，Redis虽然提供事务功能，但Redis的事务和关系数据库的事务不可同日而语，Redis的事务只能保证隔离性和一致性（I和C），无法保证原子性和持久性（A和D）。

虽然Redis并没有严格遵循ACID原则，但实际上大部分业务也不需要严格遵循ACID原则。以上面的微博关注操作为例，即使系统没有将A加入B的粉丝列表，其实业务影响也非常小，因此我们在设计方案时，需要根据业务特性和要求来确定是否可以用Redis，而不能因为Redis不遵循ACID原则就直接放弃。

## 文档数据库

为了解决关系数据库schema带来的问题，文档数据库应运而生。文档数据库最大的特点就是no-schema，可以存储和读取任意的数据。目前绝大部分文档数据库存储的数据格式是JSON（或者BSON），因为JSON数据是自描述的，无须在使用前定义字段，读取一个JSON中不存在的字段也不会导致SQL那样的语法错误。

文档数据库的no-schema特性，给业务开发带来了几个明显的优势。

1\.新增字段简单

业务上增加新的字段，无须再像关系数据库一样要先执行DDL语句修改表结构，程序代码直接读写即可。

2\.历史数据不会出错

对于历史数据，即使没有新增的字段，也不会导致错误，只会返回空值，此时代码进行兼容处理即可。

3\.可以很容易存储复杂数据

JSON是一种强大的描述语言，能够描述复杂的数据结构。例如，我们设计一个用户管理系统，用户的信息有ID、姓名、性别、爱好、邮箱、地址、学历信息。其中爱好是列表（因为可以有多个爱好）；地址是一个结构，包括省市区楼盘地址；学历包括学校、专业、入学毕业年份信息等。如果我们用关系数据库来存储，需要设计多张表，包括基本信息（列：ID、姓名、性别、邮箱）、爱好（列：ID、爱好）、地址（列：省、市、区、详细地址）、学历（列：入学时间、毕业时间、学校名称、专业），而使用文档数据库，一个JSON就可以全部描述。

```
{                    
    "id": 10000, 
    "name": "James", 
    "sex": "male", 
    "hobbies": [  
        "football", 
        "playing", 
        "singing"
    ], 
    "email": "user@google.com", 
    "address": {  
        "province": "GuangDong", 
        "city": "GuangZhou", 
        "district": "Tianhe", 
        "detail": "PingYun Road 163"
    }, 
    "education": [  
        {  
            "begin": "2000-09-01", 
            "end": "2004-07-01", 
            "school": "UESTC", 
            "major": "Computer Science & Technology"
        }, 
        {  
            "begin": "2004-09-01", 
            "end": "2007-07-01", 
            "school": "SCUT", 
            "major": "Computer Science & Technology"
        }
    ]
 }
```

通过这个样例我们看到，使用JSON来描述数据，比使用关系型数据库表来描述数据方便和容易得多，而且更加容易理解。

文档数据库的这个特点，特别适合电商和游戏这类的业务场景。以电商为例，不同商品的属性差异很大。例如，冰箱的属性和笔记本电脑的属性差异非常大，如下图所示。

<img src="16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/81c57d42e269521ba4b671cac345066e.jpg" style="zoom: 50%;" /><img src="16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/83614dfae6106ae3d08yy5a8b3bda5e7.jpg" style="zoom:50%;" />

即使是同类商品也有不同的属性。例如，LCD和LED显示器，两者有不同的参数指标。这种业务场景如果使用关系数据库来存储数据，就会很麻烦，而使用文档数据库，会简单、方便许多，扩展新的属性也更加容易。

文档数据库no-schema的特性带来的这些优势也是有代价的，最主要的代价就是不支持事务。例如，使用MongoDB来存储商品库存，系统创建订单的时候首先需要减扣库存，然后再创建订单。这是一个事务操作，用关系数据库来实现就很简单，但如果用MongoDB来实现，就无法做到事务性。异常情况下可能出现库存被扣减了，但订单没有创建的情况。因此某些对事务要求严格的业务场景是不能使用文档数据库的。

文档数据库另外一个缺点就是无法实现关系数据库的join操作。例如，我们有一个用户信息表和一个订单表，订单表中有买家用户id。如果要查询“购买了苹果笔记本用户中的女性用户”，用关系数据库来实现，一个简单的join操作就搞定了；而用文档数据库是无法进行join查询的，需要查两次：一次查询订单表中购买了苹果笔记本的用户，然后再查询这些用户哪些是女性用户。

## 列式数据库

顾名思义，列式数据库就是按照列来存储数据的数据库，与之对应的传统关系数据库被称为“行式数据库”，因为关系数据库是按照行来存储数据的。

关系数据库按照行式来存储数据，主要有以下几个优势：

- 业务同时读取多个列时效率高，因为这些列都是按行存储在一起的，一次磁盘操作就能够把一行数据中的各个列都读取到内存中。

- 能够一次性完成对一行中的多个列的写操作，保证了针对行数据写操作的原子性和一致性；否则如果采用列存储，可能会出现某次写操作，有的列成功了，有的列失败了，导致数据不一致。

我们可以看到，行式存储的优势是在特定的业务场景下才能体现，如果不存在这样的业务场景，那么行式存储的优势也将不复存在，甚至成为劣势，典型的场景就是海量数据进行统计。例如，计算某个城市体重超重的人员数据，实际上只需要读取每个人的体重这一列并进行统计即可，而行式存储即使最终只使用一列，也会将所有行数据都读取出来。如果单行用户信息有1KB，其中体重只有4个字节，行式存储还是会将整行1KB数据全部读取到内存中，这是明显的浪费。而如果采用列式存储，每个用户只需要读取4字节的体重数据即可，I/O将大大减少。

除了节省I/O，列式存储还具备更高的存储压缩比，能够节省更多的存储空间。普通的行式数据库一般压缩率在3:1到5:1左右，而列式数据库的压缩率一般在8:1到30:1左右，因为单个列的数据相似度相比行来说更高，能够达到更高的压缩率。

同样，如果场景发生变化，列式存储的优势又会变成劣势。典型的场景是需要频繁地更新多个列。因为列式存储将不同列存储在磁盘上不连续的空间，导致更新多个列时磁盘是随机写操作；而行式存储时同一行多个列都存储在连续的空间，一次磁盘写操作就可以完成，列式存储的随机写效率要远远低于行式存储的写效率。此外，列式存储高压缩率在更新场景下也会成为劣势，因为更新时需要将存储数据解压后更新，然后再压缩，最后写入磁盘。

基于上述列式存储的优缺点，一般将列式存储应用在离线的大数据分析和统计场景中，因为这种场景主要是针对部分列单列进行操作，且数据写入后就无须再更新删除。

## 全文搜索引擎

传统的关系型数据库通过索引来达到快速查询的目的，但是在全文搜索的业务场景下，索引也无能为力，主要体现在：

- 全文搜索的条件可以随意排列组合，如果通过索引来满足，则索引的数量会非常多。

- 全文搜索的模糊匹配方式，索引无法满足，只能用like查询，而like查询是整表扫描，效率非常低。

我举一个具体的例子来看看关系型数据库为何无法满足全文搜索的要求。假设我们做一个婚恋网站，其主要目的是帮助程序员找朋友，但模式与传统婚恋网站不同，是“程序员发布自己的信息，用户来搜索程序员”。程序员的信息表设计如下：

<img src="16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/d93121cecabc2182edb68bebfc467f39.jpg" style="zoom:33%;" />

我们来看一下这个简单业务的搜索场景：

- 美女1：听说PHP是世界上最好的语言，那么PHP的程序员肯定是钱最多的，而且我妈一定要我找一个上海的。

美女1的搜索条件是“性别 + PHP + 上海”，其中“PHP”要用模糊匹配查询“语言”列，“上海”要查询“地点”列，如果用索引支撑，则需要建立“地点”这个索引。

- 美女2：我好崇拜这些技术哥哥啊，要是能找一个鹅厂技术哥哥陪我旅游就更好了。

美女2的搜索条件是“性别 + 鹅厂 + 旅游”，其中“旅游”要用模糊匹配查询“爱好”列，“鹅厂”需要查询“单位”列，如果要用索引支撑，则需要建立“单位”索引。

- 美女3：我是一个“女程序员”，想在北京找一个猫厂的Java技术专家。

美女3的搜索条件是“性别 + 猫厂 + 北京 + Java + 技术专家”，其中“猫厂 + 北京”可以通过索引来查询，但“Java”“技术专家”都只能通过模糊匹配来查询。

- 帅哥4：程序员妹子有没有漂亮的呢？试试看看。

帅哥4的搜索条件是“性别 + 美丽 + 美女”，只能通过模糊匹配搜索“自我介绍”列。

以上只是简单举个例子，实际上搜索条件是无法列举完全的，各种排列组合非常多，通过这个简单的样例我们就可以看出关系数据库在支撑全文搜索时的不足。

1\.全文搜索基本原理

全文搜索引擎的技术原理被称为“倒排索引”（Inverted index），也常被称为反向索引、置入档案或反向档案，是一种索引方法，其基本原理是建立单词到文档的索引。之所以被称为“倒排”索引，是和“正排“索引相对的，“正排索引”的基本原理是建立文档到单词的索引。我们通过一个简单的样例来说明这两种索引的差异。

假设我们有一个技术文章的网站，里面收集了各种技术文章，用户可以在网站浏览或者搜索文章。

正排索引示例：

<img src="https://static001.geekbang.org/resource/image/5f/87/5fe73007957ecfcca009fd81f673df87.jpg?wh=3708*1624" title="注：文章内容仅为示范，文章内 [br] 实际上存储的是几千字的内容" style="zoom:33%;" />

正排索引适用于根据文档名称来查询文档内容。例如，用户在网站上单击了“面向对象葵花宝典是什么”，网站根据文章标题查询文章的内容展示给用户。

倒排索引示例：

<img src="https://static001.geekbang.org/resource/image/ea/f6/ea5dc300ec9c556dc13790b69f4d60f6.jpg?wh=3692*1601" title="注：表格仅为示范，不是完整的倒排索引表格，[br] 实际上的倒排索引有成千上万行，因为每个单词就是一个索引" style="zoom:33%;" />

倒排索引适用于根据关键词来查询文档内容。例如，用户只是想看“设计”相关的文章，网站需要将文章内容中包含“设计”一词的文章都搜索出来展示给用户。

2\.全文搜索的使用方式

全文搜索引擎的索引对象是单词和文档，而关系数据库的索引对象是键和行，两者的术语差异很大，不能简单地等同起来。因此，为了让全文搜索引擎支持关系型数据的全文搜索，需要做一些转换操作，即将关系型数据转换为文档数据。

目前常用的转换方式是将关系型数据按照对象的形式转换为JSON文档，然后将JSON文档输入全文搜索引擎进行索引。我同样以程序员的基本信息表为例，看看如何转换。

将前面样例中的程序员表格转换为JSON文档，可以得到3个程序员信息相关的文档，我以程序员1为例：

```
{
  "id": 1,
  "姓名": "多隆",
  "性别": "男",
  "地点": "北京",
  "单位": "猫厂",
  "爱好": "写代码，旅游，马拉松",
  "语言": "Java、C++、PHP",
  "自我介绍": "技术专家，简单，为人热情"
 }
```

全文搜索引擎能够基于JSON文档建立全文索引，然后快速进行全文搜索。以Elasticsearch为例，其索引基本原理如下：

> Elastcisearch是分布式的文档存储方式。它能存储和检索复杂的数据结构——序列化成为JSON文档——以实时的方式。

> 在Elasticsearch中，每个字段的所有数据都是默认被索引的。即每个字段都有为了快速检索设置的专用倒排索引。而且，不像其他多数的数据库，它能在相同的查询中使用所有倒排索引，并以惊人的速度返回结果。

（[https://www.elastic.co/guide/cn/elasticsearch/guide/current/data-in-data-out.html](<https://www.elastic.co/guide/cn/elasticsearch/guide/current/data-in-data-out.html>)）

## 小结

今天我为你讲了为了弥补关系型数据库缺陷而产生的NoSQL技术，希望对你有所帮助。

这就是今天的全部内容，留一道思考题给你吧，因为NoSQL的方案功能都很强大，有人认为NoSQL = No SQL，架构设计的时候无需再使用关系数据库，对此你怎么看？

欢迎你把答案写到留言区，和我一起讨论。相信经过深度思考的回答，也会让你对知识的理解更加深刻。（编辑乱入：精彩的留言有机会获得丰厚福利哦！）

## 精选留言(102)

- 关于NoSQL，看过一张图，挺形象：“1970，We have no SQL”->“1980，Know SQL”->“2000，NoSQL”->“2005，Not only SQL”->“2015，No，SQL”。目前，一些新型数据库，同时具备了NoSQL的扩展性和关系型数据库的很多特性。       关系型和NoSQL数据库的选型。考虑几个指标，数据量、并发量、实时性、一致性要求、读写分布和类型、安全性、运维性等。根据这些指标，软件系统可分成几类。       1.管理型系统，如运营类系统，首选关系型。       2.大流量系统，如电商单品页的某个服务，后台选关系型，前台选内存型。       3.日志型系统，原始数据选列式，日志搜索选倒排索引。       4.搜索型系统，指站内搜索，非通用搜索，如商品搜索，后台选关系型，前台选倒排索引。       5.事务型系统，如库存、交易、记账，选关系型+缓存+一致性协议，或新型关系数据库。       6.离线计算，如大量数据分析，首选列式，关系型也可以。       7.实时计算，如实时监控，可以选时序数据库，或列式数据库。

  作者回复: 求原图😃

  2018-06-04

  **22

  **304

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091062516.jpeg)

  Snway

  No SQL并非银弹，如ACID方面就无法跟关系型数据库相比，实际运用中，需要根据业务场景来分析，比较好的做法是，No SQL+关系型数据库结合使用，取长补短。如我们之前的做法是将商品/订单/库存等相关基本信息放在关系型数据库中(如MySQL，业务操作上支持事务，保证逻辑正确性)，缓存可以用Redis(减少DB压力)，搜索可以用Elasticsearch(提升搜索性能，可通过定时任务定期将DB中的信息刷到ES中)。

  2018-06-03

  **2

  **44

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072517.jpeg)

  约书亚

  看过一些资料，RDB当年（比我还大好多）也是在数据库大战的血雨腥风中一条血路杀出来的。现在回魂的document db当前也是RDB对手之一。如果真有这么多问题，怎么还能脱颖而出，主宰软件开发领域这么久。只能说现互联网和物联网的兴起使得应用场景向海量数据，线下上分析，读多写也多这些情况偏移了，这些场景对ACID的要求很低。 但现在大多数应用的也许还是直接面对用户，要求数据有一定可靠性，数据总量和并发量并没那么高。RDB技术成熟，人才易得，声明式SQL语法让开发者忽略底层实现，关系型的模型也符合人的思考模式。而且现在大多数一流的RDB都集成了基本的文档存储和索引，空间存储，全文检索，数据分析等功能。在没达到一定规模和深度前，完全可以用个RDB来做MVP，甚至搞定中小型也许场景。 从码农的角度看，我还是更崇拜关系型数据库，因为其底层实现里包罗了算法，系统，网络，分布式，数学统计学各种绝世武功。 前几年在NoSQL炒起来没多久，NewSQL的概念又被提出了。现在各路牛人都投入到D RDB的研发中，成型的也有不少。虽然不太可能完全取代现在的各种NoSQL，但也许能收复不少失地。 历史就是个循环，天下大势分久必合合久必分...

  作者回复: RDB的功能是最强大的

  2018-06-02

  **

  **37

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072518.jpeg)

  姜戈

  华仔，我们公司刚起步的时候有2个应用，数据库用的mysql,  日志也存储在mysql,   用户增长很快，没有专职架构师，现在准备重构以适应业务增长，考虑jhipster on kubernetes ,  昨天同事之间交流了一下方案: gateway + registry + k8s traefik ingress, 通过k8s 实现hipster技术栈的高可用，所有微服务容器化， 第一阶段准备拆成5个service,  在gateway做路由规则和鉴权，日志部分用mongodb,  mysql暂时不分库，订单rocketMQ队列化,  这样合理吗？有什么更好的建议?

  作者回复: 我感觉你们一下子引入这么多对你们来说是新的技术，风险有点大。

  2018-06-02

  **3

  **27

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072519.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  公号-技术夜未眠![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  需求驱动架构，无论选用RDB/NoSQL/DRDB，一定是以需求为导向，最终的数据存储方案也必然是各种权衡的设计妥协。 没有银弹，因为不同的应用需求对数据的要求也各不相同。当前我们不可能找到一个存储方案能满足所有的应用需求，但是我们可以通过认识到各种存储方案的优点与缺点（陷阱），在实际应用中，根据应用场景的不同要求（比如对可用性、一致性、易用性、支持事务、响应延迟、伸缩性等），按照上述特性要求的优先级排序，找到最合适的方案并“混合搭配”使用各种数据存储方案。 多种方案搭配混用必然会增加应用的复杂性与增加运维成本，但同时也带来了系统更多的灵活性。 举例：传统/互联网金融领域目前就不可能离开RDB。

  2018-06-03

  **

  **23

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072520.jpeg)

  彬哥

  我认为，行式数据库读取的时候，用sql可以指定关心的列，无需读取所有。 所以，这一段是不是笔者笔误？求赐教

  作者回复: 数据库返回你需要的列给你，但是数据库将数据从磁盘读入内存的时候，是整行读取的，事实上是读取行所在的整页数据

  2018-06-28

  **6

  **21

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072521.jpeg)

  孙振超

  本章最大的收获是了解了nosql数据库的四种类型以及特点、使用场景和关系型数据库的区别，比如redis中的list结构的强大之处、列式存储为什么压缩比会高很多，这也是作者比较强的地方。作为存储数据的系统，肯定有共同的地方，但既然能单独拎出来又说明有不同之处，掌握了相同之处可以快速理解系统能力，了解了不同之处可以明白其使用范围，并且对自己以后的架构设计方案选择提供基础。 自己之前总觉得要掌握一个系统要从看源代码开始，但后来发现这种方式太过耗时，效率过低，就开始转变为了解这个系统主要的能力是什么，然后根据能力去进行拆解，看那些是自己已经掌握了解的，那些是陌生的，对于陌生的部分了解其实现思路是怎么样的，对于实现思路的主要细节看下代码了解，这样有针对性的看效率得到了比较大的提升。 对于本篇的问题，在文章中其实已经说明白了，关系型数据库是不可替代的，比如事务能力、强管控、经常性读取多列的场景都需要关系型数据库。

  作者回复: 这种方法叫做“比较学习法”，对比学习，效果最明显

  2018-06-18

  **3

  **20

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072522.jpeg)

  9527

  老师您好，问个题外话😂 如今各种新技术层出不穷，像那种底层的大块头之类的书籍，比如深入理解计算机系统，编译原理这样的，还有必要深入学习吗？ 像深入理解计算机系里面的反汇编分析在实际工作中确是是用不到，更别提开发个编译器了 面对这些看似不错的书籍，但又觉得里面的内容无法运用到实际中，有些迷茫，望老师指点

  作者回复: 请到聊聊架构公众号搜索《当我们聊技术实力的时候，我们到底在聊什么》，我写了一篇文章来回答你这个问题

  2018-06-05

  **3

  **16

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072523.jpeg)

  铃兰Neko

  认真的看到现在，有两点疑问。 1.es和lucene这种，一般当做搜索引擎，也划在nosql里面合适吗? 2.能否给一些常见nosql性能和数据量的数据， 网上搜不大到，唯一知道的都是从ppt里面扣的 另外，nosql那个图原图应该在reddit上，地址是 http://i.imgur.com/lkG9Vm8.jpg

  作者回复: 1. 只要是为了弥补关系数据库的缺陷的方案，都可算nosql 2. 量级一般都是上万，但和测试硬件测试用例相关，如果要用，最好自己测试

  2018-06-04

  **

  **16

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072524.jpeg)

  小喵喵

  跨库操作如何做事物呢？看到这期了还是没有答案吗？

  作者回复: 跨库操作不建议事物，效率低，容易出错，用最终一致性

  2018-06-03

  **3

  **13

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072525.jpeg)

  明翼

  文档数据库和es有何区别，es我看也是对scheme不敏感的啊

  作者回复: 文档数据库定位于存储和访问，es定位于搜索，但目前差别并不是很大，因为系统边界的扩充，就像mongodb也要支持ACID，mysql也要支持文档存储

  2018-06-14

  **

  **11

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072526.jpeg)

  空档滑行

  NoSQL出现的原因是针对解决某一特定问题，这个问题用关系型数据库无法很好的解决。所以就注定了NO SQL不会是个大而全的东西，虽然这几年特性不断增加，使得部分业务场景可以完全不依赖关系数据库，但是关系数据库仍然是最容易理解，适用场景最广泛的数据库。就像OLAP无法完全代替离线计算，HTAP无法完全代替OLAP+OLTP一样，NOSQL是很好的补充，不是为了代替

  2018-06-02

  **

  **10

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072527.jpeg)

  func

  关系数据库就是行式存储，从硬盘读到内存的时候，是整行读取，实际上还是整页读取的。不明白这个整页指的是？这个整页指的是  包括很多行数据吗？这个整页的数据都是我需要的吗？

  作者回复: 为了存储高效，文件存储在磁盘上是分页存储的，每页包含很多行，不一定都是你要的，更多信息可以参考innodb得文档

  2018-09-29

  **

  **9

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/132.jpeg)

  monkay

  有个商品搜索的需求，怎么评估elasticsearch和solr哪个更合适？

  作者回复: 功能上其实都可以，关键看实时性和性能，如果都满足，按照简单原则，你熟悉哪个就用哪个

  2018-06-02

  **2

  **8

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072528.jpeg)

  轩辕十四

  mongo4 开始支持事务了吧

  作者回复: 我不敢用，我相信MySQL😊

  2018-06-27

  **

  **6

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072529.jpeg)

  zj

  提一个问题，数据本来是存在mysql的，然后将数据同步到es中，中年如何保证一致性

  作者回复: 更新mysql成功后就更新es，再加一个定时校验

  2018-06-06

  **

  **6

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072524.jpeg)

  小喵喵

  跨库如何用最终一致性呢？老师能加微信吗

  作者回复: 土的方式就是日志加后台定时对账，牛逼的就是分布式一致性算法

  2018-06-04

  **

  **6

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072530.jpeg)

  幻影霸者

  mongodb4.0不是支持acid事务吗？

  作者回复: 官方宣称会实现事务，但需要看最终实现，实现事务不是完全没有代价的，要么性能降低，要么灵活性降低

  2018-06-03

  **

  **6

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082531.jpeg)

  J.Smile

  要点总结： ● 文档数据库的缺点 ○ 不支持事务 ○ 不支持join ● 倒排索引与正排索引 ○ “正排索引”的基本原理是建立文档到单词的索引 ○ ”倒排索引“适用于根据关键词来查询文档内容。 ✓ 心得 ● 记得一次面试中被问到了关于条件组合搜索的问题，当时回答的一塌糊涂，现在才明白可以使用全文搜索引擎比如es这种方案来代替。而当时自己的思维局限在了关系型数据库。 ● 通过学习了解到作为一个技术人，不一定动不动就去看源码，那样很耗时耗力，结果未必就是好的。但一定要知道哪些技术解决那些问题。这样当做架构的概要设计与技术选型的时候有利于纵览全局，快速确定目标方案。而具体的细节问题可以在详细设计阶段掌握。

  作者回复: 倒排索引就是为了解决关系数据库在组合条件搜索时无能为力的问题。目前比较流行的是ElasticSearch和MongoDB，两者的文档都写的很好，直接看官方文档就可以了。

  2020-12-10

  **2

  **5

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082532.jpeg)

  allen.huang

  华哥，我想问下，我之前在做电商时，将商品全部存到redis中以hash结构存储，同时商品下面有个库存也存在其下面，库存在做减操作时，我用hincrby来进行，并发小的时候问题不大，库存不会超。 并发大了以后会不会数据库一样超了？

  作者回复: redis本身不存在并发访问，因为是单进程的，但你的业务代码很可能会有问题，例如先get然后判断后再hincrby就会出现并发问题

  2018-12-27

  **3

  **5

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082533.jpeg)

  Otto Yen

  你好，我有找到一张 history of nosql图档，分享给您。 https://www.reddit.com/r/ProgrammerHumor/comments/2mk8sb/history_of_nosql/

  2018-06-11

  **

  **5

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082534.jpeg)

  ncicheng

  对于需要存储大量非文本格式的文档（如doc），项目中一般采取blob，但是搜索文档内容不容易，如果换成NoSQL来存储是不是比现在好？谢谢~

  作者回复: 赶紧换es，谁用谁知道😂

  2018-06-05

  **

  **5

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082535.jpeg)

  三月沙@wecatch

  Nosql 数据库好用，但是用好不容易，接触过一些团队在 MongoDb 中存储一些结构变动频繁的数据，结果导致每次结构发生变化时，之前的数据都不做结构变更，由于业务复杂，代表量大，很多地方没有做好兼容性，而且还用的动态语言，导致线上莫名就会就会出 null和 keyError 错误。从个人使用角度来讲，虽然数据库本身 schema 没有强约束，不代表开发者可以滥用这一能力，最佳实践是应该是终保持新老数据结构的一致性，保证数据在代码中的稳定性。

  作者回复: 滥用还不如约束，滥用到最后面就是混乱，纠偏的代价很高

  2018-06-04

  **

  **5

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082536.jpeg)

  caison

  redis的操作应该是原子性的吧，因为redis的单线程的

  作者回复: 是的

  2018-07-25

  **

  **4

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082537.jpeg)

  ZYCHD(子玉)

  保险行业用的关系型数据库多些，No SQL少些。保险行业一个重要的要求是数据不能丢失！

  作者回复: 主要还是保险行业技术不够开放😄，保险行业数据和支付宝相比一下就知道了

  2018-06-18

  **2

  **4

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082538.jpeg)

  Andy

  对于这个观点，个人觉得，对方并没有完全吃透关系型数据库与非关系型数据之间的区别和联系，为什么会产生非关系型数据，那是为了弥补关系型数据库的缺点而诞生的，基于大数据时代背景而出现的，一个用于解决缺点的东西，并不能说明把优点就覆盖了，这个世界没有觉得最优的解决方案。关系型数据的存在，就像作者说的，基于其自身ACID特性和强大的SQL，这是非关系型数据库无法替代的，故NoSql != No Sql而是Not Only Sql

  作者回复: 正解

  2020-09-10

  **

  **3

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082539.jpeg)

  shihpeng

  曾经面试过一个在新加坡金融创业公司的小老弟，让他介绍一下之前做的系统：主要数据存储是MongoDB，因为他们数据量比较大 (2亿条)，要用大数据存储；架构图中MongoDB旁边一个PostgreSQL，因为MongoDB没法join，所以用了PostgreSQL；下面有一个ES，因为MongoDB/PSQL搜索不方便；远一点的地方有一个ArangoDB，说是因为他们需要事务处理.... 我问他，你知道你的这些需求一台MySQL可以全搞定吗？他回：MySQL不是很慢吗...

  作者回复: 哈哈，2亿数据就敢号称是大数据，也是对大数据的“大”有误解了。 这个架构确实过度设计复杂了，而且金融数据大部分都是用关系数据库来管理，因为数据之间的关系比较复杂，他们却反其道而行之。 而且MongoDB和ES搭配使用这是很少见的，都是文档存储系统，其实整个方案MySQL基本能搞定，最多加个ES做全文搜索。

  2021-01-08

  **2

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082540.jpeg)

  谭方敏

  K-V型 redis部分支持事务(I-Isolation,C-Consistency),不支持(A-Atomicity,D-Durability).面向数据结构。 文档型 mongodb支持json格式，no-schema,基本上啥都可以塞进去，不支持事务. 列式 hbase, 支持读取单列，压缩效率高，反复读列效率低， 全文 elasticsearch, 倒排索引，解决关系型数据库中like性能低下的问题 选择哪种数据库，需要结合业务属性，而不是拍脑袋的，每种数据库都有自己的优劣势。 如果在要求强事务的业务中，那么关系型数据库还是不二之选。

  2020-03-02

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082541.jpeg)

  (╯‵□′)╯︵┻━┻

  思考：这个说法是自相矛盾的。假如NoSQL的强大是因为一系列适配专有的业务场景的NoSQL种类，那么就会产生一种适应传统SQL业务场景的NoSQL，也就是SQL本身。

  作者回复: 角度刁钻👍👍

  2019-08-03

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082542.jpeg)

  冯宇

  所以postgresql功能性和性能就平衡的非常好，原生支持全文检索，不必因为简单的关键字查询就必须引入es增加架构复杂性

  作者回复: es其它功能也很牛逼呀，pg本质上是数据库，会约束其实现方式和功能

  2018-11-21

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082543.jpeg)

  孙晓明

  前两天看消息说MySql发布新版本，支持NoSql，这是不是也是数据库发现的趋势，李老师对这个怎么看？

  作者回复: 数据库厂商为了提升自己吸引力做的融合方案，例如MongoDB也宣称要支持ACID，但我还是相信专业的系统做专业的事情，融合方案不是好的选择

  2018-06-21

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091072524.jpeg)

  小喵喵

  而用文档数据库是无法进行 join 查询的，需要查两次：一次查询订单表中购买了苹果笔记本的用户，然后再查询这些用户哪些是女性用户。 具体如何查呢？ 把一串的 ID拼接起来，然后用户ID in () and sex ＝女吗，如果这样的话，数据多性能很差的。期待老师其他更好的方案。

  作者回复: 我真没有太好的方案😂要么查两次，要么冗余数据

  2018-06-02

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082544.png)

  Chris

  根据实际需求来决定需要使用NoSQL或关系型数据库，或者结合使用，取长补短。实际的场景中，一般都是两者相互结合:例如我司，关系型数据库为主，因为其提供了强大的SQL功能以及ACID事务属性；同时，也是用了Redis，其主要作用是缓存，缓解关系型数据库的访问压力。一个字，怎么使用或者结合使用，都是需要根据实际的生产需求来决定，而不是拍着屁股决定NoSQL就真的等于No SQL嘿嘿嘿。

  2018-06-02

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082545.jpeg)

  王磊

  个人理解，关系型数据库也有列式存储，如Greenplum, Redshift, 所以这个不是NoSQL和关系数据库的区别吧。

  2018-06-02

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082546.jpeg)

  yankerzhu

  记得有文章介绍workday，系统中用到关系型数据库和非关系型数据库，但是关系型数据库中只有三个表，用来存储元数据，主要在系统启动初始化时用到，业务数据全部在非关系型数据库中，如果有人用过这样架构的，请分享经验！

  2018-06-02

  **

  **2

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082547.jpeg)

  Little_Wang

  mongo支持事务和join

  作者回复: 支持的粒度和力度和数据库的事务相比还是有差距的

  2021-08-05

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091082548.jpeg)

  joinboo

  华哥，hbase作为列式数据库的代表，在实时写和读的性能都不错，对于一些数据一致性要求不那么高的在线场景我们就是直接用hbase, 目前用着也挺香。所以我想问下，在你之前的项目中使用hbase提供在线服务的场景多吗，比如有哪些场景？

  作者回复: HBase定位就是在线服务，而不是离线大数据存储平台，我知道的用HBase在线服务的，有用作K-V的，例如移动设备的token、定位信息等存储，也有用HBase做时序数据存储的，因为HBase是sorted map，key是排序的，好像滴滴用来存储司机的轨迹，你可以查查

  2021-04-21

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092549.jpeg)

  枫

  点到即止，理论居多，作用不大

  作者回复: 每个系统都可以写一篇专栏了，如果你是想深入学习每个系统，可以订阅其它专栏，例如Redis、ES、MongoDB等都有专栏了。

  2021-04-10

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_9b9530

  文档数据库的这个特点，特别适合电商和游戏这类的业务场景。 我想知道，作者是否真实的在电商场景下使用文档数据库来对商品的属性进行存储。如果数据量不大的情况下，这么对商品属性进行设计应该没问题，海量数据下，我觉得这些属性可能就会乱用。关系型数据库的严谨结构，应该更适合去维护一个稳定结构的产品。

  作者回复: 我在游戏交易的业务中用过ES，你说的关系数据库严谨结构是对的，但是在电商和游戏业务里面，数据结构变化太频繁了，恰好不符合关系数据库的场景。 至于乱用，你用关系数据库一样可能会乱用的。

  2020-11-30

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092550.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  escray![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/d57fddfa5b2c2ce3071e92304da8af62.png)

  如果遇到关系型数据就用 MySQL；需要 K-V 存储的时候，就采用 Redis；需要文档数据库的时候，就用 MongoDB；需要列式数据库的话，就采用 HBase；需要全文搜索的时候，用 Elasticsearch 或者 Solr。 谈笑风生……但是我的问题是，除了 MySQL，对其他几个都不熟，学习之路漫漫。 K-V 存储的 Redis 和文档数据库都不支持事务，如果需要的话，只能使用代码实现。列式存储虽然没有明说，但是我觉的在事务支持上应该也没什么优势，与全文搜索数据库类似，都是更重视读的效率。 NoSQL = Not Only SQL 可想而知，其实有很多领域还是无法离开关系型数据库的，比如涉及到账目流转的金融系统，还有电商平台的订单系统，大部分的用户管理系统…… NoSQL 可能会成为关系型数据库的好的补充方案，在我的感觉里面，现实社会映射到技术领域，应该还是关系型数据为主，也相对简单一些。总不可能，未来所有的系统都变成人工智能、深度学习……的黑盒子。

  作者回复: 大牛不是那么容易当的������

  2020-09-08

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  jhansin

  老师，假设有个场景是统计功能，并且有日期筛选，选择哪种数据库更合适呢

  作者回复: 我们以前用infobrightt来做统计数据库

  2020-08-19

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092551.jpeg)

  不能扮演天使

  老师，讲hbase和mysql区别时说hbase是解决列式IO问题的，如果mysql有id,name,addr三个字段，name有索引，sql只查询id和name，难道还是会把包含age字段的整行数据读进bufferpool吗？不可能吧(ﾉﾟ0ﾟ)ﾉ~

  作者回复: 是这样的，你去看看mysql技术内幕之类的书

  2020-07-11

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092552.jpeg)

  陈先生（Ken）

  老师，mongodb最新版本加入了事务管理，现在的mongodb能否处理传统银行相关业务对于事务强需求的场景

  作者回复: 事务强需求请相信数据库，事务的实现复杂度在于很多细节，mongodb肯定不成熟

  2020-06-15

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_e4c318

  hbase并不是列式数据库，严格的说hbase式列族式，和类似于sap hana这样的纯列式数据库相比是存在很大区别的，什么时候hbase才是真正的列式数据库，极端情况，一个列就是一个column family才算是真正标准意义上的列式数据库。

  作者回复: 列式数据库可以是概念上的泛称😃sap的东西在互联网用的不多

  2020-03-29

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092553.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  Ryoma

  看到很多人说的 NoSQL 4大分类跟老师的不太一样，比如包含 图数据库，老师觉得呢？

  作者回复: 没有绝对的，不过我理解图数据库可以算一种新的存储系统

  2019-05-18

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092554.jpeg)

  行者

  NoSQL在我看来更多的算是对关系数据库的一种补充，解决关系数据库无法解决的一些问题；回到老师的问题，对于需要事务保证的业务，关系数据库还是很有必要的。

  作者回复: 是的，关系数据库目前不可替代

  2019-03-02

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  请叫我红领巾

  redis不支持原子性？

  作者回复: redis的事务不支持原子性

  2018-09-28

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092555.jpeg)

  文竹

  NoSql=No Sql，我刚开始接触Java行业时就是处于这个状态，因为名字缺人令人误解。 文中介绍的4种NoSql代表，都有各自的应用场景。目前关系型数据库仍主要被使用，其他Nosql只能作为补充。

  作者回复: 目前的主流方式就是你说的，Not Only SQL

  2018-08-19

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  autopenguin

  现在PostgreSQL、MySQL等数据库已经开始支持json类型的数据了，那是否意味着文档型数据库不再有相应的缺点了呢？

  作者回复: 文档型数据库ACID特性不好做，简单来说，现在不敢用文档型数据来存余额和库存吧😄

  2018-08-12

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/132-16617956091092556.jpeg)

  乘风

  看了很多大牛的评价，很精髓，我也说点自己的小理解，虽然都是存储系统，但各系统的特性也决定了适用的场景，熟悉最多的还是rd，他的特点保证我们的数据有序可靠，但在一定程度上又降低了性能，但为什么有些技术不是我们出的，归根结底还是没有场景，没有量级，其实大部分还是没区别的，sql>=nosql，敢想才有突破

  作者回复: 是的，业务场景决定技术架构和方案，BAT的方案很牛，但放到小公司，小公司可能就一堆问题

  2018-08-09

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092557.jpeg)

  bubble

  跨库操作不建议事物，效率低，容易出错，用最终一致性，如何保证最终一致性

  作者回复: 最常用的就是定时校验，记录很多关键日志？，或者用zk这种分布式协调系统

  2018-07-19

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092558.jpeg)

  追寻云的痕迹

  期待未来讲讲NewSQL

  作者回复: 目前不是很成熟，建议不要大规模用，例如mongodb号称要支持ACID

  2018-07-02

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092559.jpeg)

  YMF_WX1981

  \1. nosql有的资料也称内存数据库，耗硬件资源，内存速度快但没磁盘稳定，全部nosql不脱妥 2. 对于请求数据不是立马显示的数据放sql数据库，妥妥的，sql数据库有存在必要性

  作者回复: nosql不适合称为内存数据库，redis ，mongodb, hbase都是磁盘存储

  2018-06-14

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092560.jpeg)

  Joker

  我们可以看到，行式存储的优势是在特定的业务场景下才能体现，如果不存在这样的业务场景，那么行式存储的优势也将不复存在，甚至成为劣势，典型的场景就是海量数据进行统计。例如，计算某个城市体重超重的人员数据，实际上只需要读取每个人的体重这一列并进行统计即可，而行式存储即使最终只使用一列，也会将所有行数据都读取出来。如果单行用户信息有 1KB，其中体重只有 4 个字节，行式存储还是会将整行 1KB 数据全部读取到内存中，这是明显的浪费。而如果采用列式存储，每个用户只需要读取 4 字节的体重数据即可，I/O 将大大减少。  这一段描述感觉有点问题 行式 是否该改成 列式

  作者回复: 没毛病，老铁，关系数据库就是行式存储，从硬盘读到内存的时候，是整行读取，实际上还是整页读取的

  2018-06-12

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092561.jpeg)

  W_T

  不能这么说，这两者是互补关系而不是竞争关系，他们各有所长，在特定的方向能发挥自己的优势，在都不是面面俱到的。 我们更多的选择是两种类型的数据库都用，比如mysql+redis已经成为我们公司系统的标配

  2018-06-05

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092562.jpeg)

  @漆~心endless

  NoSQL是为了解决SQL无法解决的问题。或者说是NoSQL弥补了SQL的不足，以达到系统的完整性。

  2018-06-05

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092563.jpeg)

  不记年

  华哥，我有个想法：将同一份数据，用不同类型的数据库存储。根据业务需求，选择最合适的数据库进行操作，是不是性能更高些呢？(我是比较小白的，如果问题问的太蠢还请见谅哈)

  作者回复: 性能会高，但是保证数据在多个存储中的一致性是比较复杂的

  2018-06-02

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092564.jpeg)

  沧海一粟

  我们目前的项目，除了用了redis外，都是用的关系型数据库，如果要做全文搜索，是在创建或更新记录时，同时将全量数据插入或更新到es，请问老师有什么备选方案？

  作者回复: es就够了

  2018-06-02

  **

  **1

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091102565.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  归零

  这篇文章非常赞，高屋建瓴的介绍关系型数据的问题和对应的NoSQL解决方案，对于技术选型很有帮助

  作者回复: 谢谢 ：）

  2022-08-24

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091102566.jpeg)

  Allan![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/d57fddfa5b2c2ce3071e92304da8af62.png)

  如果nosql能够做到事务性，我觉得是可以替代关系型数据库，但是目前是做不到的。银行这种业务稍微一个计算不准，那就要祭天了。

  作者回复: 想做到事务性就实现不了nosql的高性能 ：）

  2022-08-15

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/132-16617956091102567.png)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  Geek_06d12d

  Redis 事物的隔离性也无法保证把，只是把多个命令一起提交但是还是按序执行存在读到不一致的数据

  作者回复: 和数据库的一致性不能比

  2022-08-10

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091102568.jpeg)

  Ted

  NO SQL 只能作为 关系型数据库的补充。 因为NO SQL的数据存储方案，都有其特定的应用场景，且不支持ACID.

  作者回复: 正解

  2021-08-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f2/f8/16aecb83.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Robert Zhang

  es现在除了搜索，大部分是当文档数据库用了，第四类应该是图数据库，geo之类的

  作者回复: 文档数据库为啥不用mongodb，mongodb定位就是文档数据库哦

  2021-08-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/1e/5a/d362f613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ZYZ

  关系型数据库查询某个字段，也会将整行字段存入数据库？？？select xx与select *没区别了？

  作者回复: 自己搜索一下“行式存储”和“列式存储”

  2021-07-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/4f/67/6a7a7534.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  雨飞Vee

  TIDB怎么样？解决分布式ACID问题的。

  作者回复: 目测现在比较火，但我还没详细研究过

  2021-04-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/66/9b/59776420.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  百威

  之前看到过一个nosql分类，前三个一样，第四个是图数据库，不知道es怎么归类的

  作者回复: ES是倒排索引全文搜索的，是为了解决SQL的like低效问题，肯定不是图数据库

  2021-04-12

  **2

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091102573.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  DFighting

  读完这篇文章最大的收益是NoSQL=Not only SQL，SQL语言已经存在了这么多年，设计之初就无法预料到如当下般这么多是数据，所以在某些场景下会不合适。但这是存储相关架构演进的必然，但是在需要ACID特性的场景下，他的价值就是很重要的。换个角度来看，NoSQL的几种实现方式都是为了解决某种应用场景(或者某些情况)下的产物，牺牲了ACID的某个或多个，至于架构设计之初选择啥，得依照合适原则来挑选。

  2021-03-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/2f/5a/4ed3465d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜心1986

  使用 nosql要经常做统计，但是需求又会变化。这种情况是否还是适合 hbase？ 如果是规则的关系数据，只是为了提高查询速度，这种场景用什么比较合适？ es还合适么？

  作者回复: Hbase不是很适合统计，它的定位是有序map，统计需求变化的时候，HBase也支撑不了。 一般来说，将数据复制为两份，一份用于在线服务，一份用于统计分析，两份数据可以用不同的存储系统，例如可以用Clickhouse来做统计。 规则的关系数据直接用关系数据库，设计好索引和查询语句，是最合理的方式

  2021-02-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ae/4a/a71a889b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  迷宫中的将军

  多模数据库也许是个解，比如微软的CosmosDB

  作者回复: 大概看了一下CosmosDB的资料，初步来看是内部用了不同的存储引擎，对外提供了不同形式的API，你在使用的时候，还是要自己判断该用什么格式的API，例如是用MongoDB的API还是Cassandra的API

  2021-01-25

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/f8/9e/f396fc0c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bazinga

  大神我很自动，当初在没有nosql数据库的时候，阿里是这么做到物品条件搜索的（你在文章说了，各个商品属性差异很大关系型数据库建表困难）

  作者回复: 这部分没有了解过，猜测只能建很多查询表或者专门用于查询的字段

  2020-07-10

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fa/ec/af22c941.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Vaccae

  nosql不只是sql

  2020-03-25

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091102578.jpeg)

  LJK

  想请问一下MongoDB不支持事务吗？我看到说4.0版本后已经支持事务了，虽然我不用MongoDB，但是还是想请教一下是不是自己理解错了 官方文档链接：https://docs.mongodb.com/manual/core/transactions/

  作者回复: 我写文章的时候还不支持，现在支持了

  2019-11-28

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091102579.jpeg)

  钱

  用人之计，在于用人所长避人所短。 使用工具的原则也是同样的道理，不光数据库，什么工具都一样。那如果工具类似哪？不同的工具总归有其独特之处，否则也不会单归某一类了，那就看他这个独特之处适合什么场景吧！现在的问题是对自己业务使用场景的判断啦！ 数据库，广义来讲凡是能存储数据的都算是吧！计算机界的数据库，感觉全部是在围绕怎么查来做文章，或者说是主要围绕怎么查询的快这个事情来做文章，查询的需求千变万化，大部分业务场景也是读多写少，所以关系型数据库使用索引加快查询速度；kv数据库使用内存加快查询速度；es使用倒排索引加快查询速度；列式数据库使用将列单存来加快查询速度，不同的存储结构决定了以什么方式查询更加的快捷，他们都有各自的市场，不过关系型数据库还是主流，毕竟他的有些特性目前还没有出现完全替代的牛逼产品。

  2019-08-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b3/e2/b90bf7c4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Pakhay

  当我们聊技术实力的时候，我们到底在聊什么

  作者回复: 这是我的一篇文章标题😂

  2019-07-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/d8/06/43bbbf2d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ycj

  Redis只能保证原子性跟隔离性无法保证一致性和持久性

  2019-06-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ee/d2/7024431c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  探索无止境

  老师您好，是否可以分享下支付宝在实际应用中，是否有选择以上讲的4款NoSQL?分别在什么场景下使用？

  作者回复: 我不太了解支付宝的细节，可以上网搜搜

  2019-04-22

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/132-16617956091112583.jpeg)

  gkb111

  数据库这方面，nosql需要进一步加强学习

  2019-01-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/78/92/ae2c1b12.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  D

  这里对于mongodb不能使用join的描述有问题，mongodb在3.2以后提供了lookup来实现join操作

  作者回复: 我写的时候还没有😀

  2018-12-18

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091112585.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  Geek_fb3db2

  nosql的实现方案有很多，但是每种都会有不足和优点，同样的关系型数据库也是为了解决某种特定需求而诞生的，最重要的是能够满足ACID特性，这是nosql无法比拟的，虽然nosql也可以通过其他方式来满足同样要求，但是往往工作量是巨大的，我们从来不指望一种实现方案能解决所有的问题

  2018-12-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/1a/9a/7b246eb1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大可可

  "能够一次性完成对一行中的多个列的写操作，保证了针对行数据写操作的原子性和一致性；否则如果采用列存储，可能会出现某次写操作，有的列成功了，有的列失败了，导致数据不一致。" 列式数据库的优点里这块是不是写错了

  作者回复: 原文中是说这是“行式存储”的优点

  2018-11-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/29/2a/9079f152.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谢真

  感谢，数据库这块今天算是比较系统的有个了解了，以前只知道有这些，却并不知各自优缺点

  作者回复: 其实都是环环相扣，自成体系的

  2018-10-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/cd/ff/b7690816.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云里雾花

  nosql应该如博主所说是no only sql，主要是根据自己的业务场景选择不同的储存方案来解决关系型数据库不能很好解决的问题，关系型数据库也有比较好的，比如事务支持上，各有利弊，关键是场景和取舍，以上算是博主文章总结，受教了。

  作者回复: 总结思考很好👍

  2018-10-19

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092555.jpeg)

  文竹

  No Sql是Not only sql，作为架构设计的一个补充。一个业务场景可以选择出适合的Sql技术引擎。

  2018-08-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/8a/b7/65d4b7ad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  YangXinjie

  所谓倒排索引其实就是传统索引，只是特定场景下的查询顺序不同罢了。那个词语索引到文档号的例子，把文档号看成rowid就是普通索引了

  作者回复: 应该说都是索引，不是传统索引

  2018-07-12

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092558.jpeg)

  追寻云的痕迹

  那TiDB呢？

  2018-07-03

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091112590.jpeg)

  sensitivemix

  mongodb 也要支持事务了

  作者回复: 世界大势，分久必合，合久必分😂 不过我还是相信专业系统做专业事情，大一统不好

  2018-06-15

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091112591.jpeg)![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,w_14.png)

  王维

  我们现在公司做的是在线考试系统，从数据存储来看，关系型数据库已经能完全满足我们的要求。我觉得凡事都有它存在的道理，比方NoSql,就是牺牲ACID的部分特性来换取时间或者空间上的一些性能上的。到底是用关心关系型数据库还是No Sql，还是根据业务来吧。

  2018-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ca/61/b3f00e6f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  byte

  SQL和NoSQL各有利弊，各有适用场景。如订单-事务场景：SQL  ，缓存，简单关系场景（1对1）适合用NoSQL。

  2018-06-05

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091092562.jpeg)

  @漆~心endless

  NoSQL是为了解决SQL无法解决的问题。或者说是NoSQL弥补了SQL的不足，以达到系统的完整性。

  2018-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c5/9e/d6fce09c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  暴走的🐌

  老师，hbase以key查询时，是如何高效取的key对应的所有列数据的，还是说存储时每一列都会加强key

  2018-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/11/ea921534.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  itperson

  关系型数据库还是有自身的优势 所以并不能抛弃而应该合理共用

  2018-06-04

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091112595.jpeg)

  亚伦碎语

  场景不同用不同的东西，做数据的统计分析，用用nosql试试

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f5/c2/6f895099.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  cooruo

  讲得很好啊，我今年刚换到互联网做后台开发，正需要这方面知识。

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/76/60/9452d5ea.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小王

  筷子和叉子的区别，相辅相成

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f1/8c/9c2d8791.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_3577d4

  搜索集群，需要merge搜索结果还要排序分页，有什么比较好的解决方案吗

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ac/a1/43d83698.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云学

  受益匪浅，请问物联网领域的开源数据库有哪些

  作者回复: 抱歉，不懂物联网😊😊

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ea/33/37f261a3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  John

  为了应对不同的应用场景，可以采用不同的搜索方式，并不是说有了no sql 就绝对不用sql,只是看哪一种能满足性能的需要。

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/de/17/75e2b624.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  feifei

  因为 NoSQL 的方案功能都很强大，有人认为 NoSQL = No SQL，架构设计的时候无需再使用关系数据库，对此你怎么看？ 我觉得这个不能一概而论，对于某些场景下，确实可以实现，no sql，比如全文搜索。 在很多的业务中，还是需要关系数据库，它的一致性事务，sql简单，在小数据量下的优势很明显。

  2018-06-02

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/resize,m_fill,h_34,w_34-16617956091122602.jpeg)

  李志博

  强事务，或者查询简单，并且不需要考虑扩容的场景还是直接mysql 吧

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a6/90/5295fce8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  昵称

  不同种类的数据库能否提供下相应的数据库代表

  2018-06-02

  **

  **

- ![img](16%E9%AB%98%E6%80%A7%E8%83%BDNoSQL.resource/132-16617956091122604.jpeg)

  xiaojiit

  受益匪浅，拓宽思路，任何业务不可能只用一类数据库解决，取其精华，去其糟粕，合适的人做合适的事，用合适的技术处理合适的需求，这是个人的看法，欢迎大家拍砖

  作者回复: 没法拍砖，就是这样的😃

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/87/0a045ddf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Carter

  我觉得大型项目也才有必要用关系型数据库，mongodb的查询效率可以满足多次查询的场景，事物问题确实不好解决

  2018-06-02
