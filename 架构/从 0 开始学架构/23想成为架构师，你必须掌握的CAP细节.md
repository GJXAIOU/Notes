# 23 \| 想成为架构师，你必须掌握的CAP细节

理论的优点在于清晰简洁、易于理解，但缺点就是高度抽象化，省略了很多细节，导致在将理论应用到实践时，由于各种复杂情况，可能出现误解和偏差，CAP 理论也不例外。如果我们没有意识到这些关键的细节点，那么在实践中应用 CAP 理论时，就可能发现方案很难落地。

而且当谈到数据一致性时，CAP、ACID、BASE 难免会被我们拿出来讨论，原因在于这三者都是和数据一致性相关的理论，如果不仔细理解三者之间的差别，则可能会陷入一头雾水的状态，不知道应该用哪个才好。

今天，我来讲讲<span class="orange">CAP 的具体细节，简单对比一下 ACID、BASE 几个概念的关键区别点。</span>

## CAP关键细节点

埃里克·布鲁尔（Eric Brewer）在《CAP 理论十二年回顾：“规则”变了》（[http://www.infoq.com/cn/articles/cap-twelve-years-later-how-the-rules-have-changed](<http://www.infoq.com/cn/articles/cap-twelve-years-later-how-the-rules-have-changed>)）一文中详细地阐述了理解和应用 CAP 的一些细节点，可能是由于作者写作风格的原因，对于一些非常关键的细节点一句话就带过了，这里我特别提炼出来重点阐述。

- CAP 关注的粒度是**数据**，而不是整个系统。

原文就只有一句话：

> C与A之间的取舍可以在同一系统内以非常细小的粒度反复发生，而每一次的决策可能因为具体的操作，乃至因为牵涉到特定的数据或用户而有所不同。

但这句话是理解和应用 CAP 理论非常关键的一点。CAP 理论的定义和解释中，用的都是 system、node 这类系统级的概念，这就给很多人造成了很大的误导，认为我们在进行架构设计时，整个系统要么选择 CP，要么选择 AP。但在实际设计过程中，每个系统不可能只处理一种数据，而是包含多种类型的数据，有的数据必须选择 CP，有的数据必须选择 AP。而如果我们做设计时，从整个系统的角度去选择 CP 还是 AP，就会发现顾此失彼，无论怎么做都是有问题的。

以一个最简单的用户管理系统为例，用户管理系统包含用户账号数据（用户 ID、密码）、用户信息数据（昵称、兴趣、爱好、性别、自我介绍等）。通常情况下，用户账号数据会选择 CP，而用户信息数据会选择 AP，如果限定整个系统为 CP，则不符合用户信息数据的应用场景；如果限定整个系统为 AP，则又不符合用户账号数据的应用场景。

所以在 CAP 理论落地实践时，我们需要将系统内的数据按照不同的应用场景和要求进行分类，每类数据选择不同的策略（CP 还是 AP），而不是直接限定整个系统所有数据都是同一策略。

- CAP 是忽略网络延迟的。

这是一个非常隐含的假设，布鲁尔在定义一致性时，并没有将延迟考虑进去。也就是说，当事务提交时，数据能够瞬间复制到所有节点。但实际情况下，从节点 A 复制数据到节点 B，总是需要花费一定时间的。如果是相同机房，耗费时间可能是几毫秒；如果是跨地域的机房，例如北京机房同步到广州机房，耗费的时间就可能是几十毫秒。这就意味着，CAP 理论中的 C 在实践中是不可能完美实现的，在数据复制的过程中，节点 A 和节点 B 的数据并不一致。

不要小看了这几毫秒或者几十毫秒的不一致，对于某些严苛的业务场景，例如和金钱相关的用户余额，或者和抢购相关的商品库存，技术上是无法做到分布式场景下完美的一致性的。而业务上必须要求一致性，因此单个用户的余额、单个商品的库存，理论上要求选择 CP 而实际上 CP 都做不到，只能选择 CA。也就是说，只能单点写入，其他节点做备份，无法做到分布式情况下多点写入。

需要注意的是，这并不意味着这类系统无法应用分布式架构，只是说“单个用户余额、单个商品库存”无法做分布式，但系统整体还是可以应用分布式架构的。例如，下面的架构图是常见的将用户分区的分布式架构。

<img src="23%E6%83%B3%E6%88%90%E4%B8%BA%E6%9E%B6%E6%9E%84%E5%B8%88%EF%BC%8C%E4%BD%A0%E5%BF%85%E9%A1%BB%E6%8E%8C%E6%8F%A1%E7%9A%84CAP%E7%BB%86%E8%8A%82.resource/66476fd7ffd5d6f80f4f9ba0938d0443-1677896806437-1.png" style="zoom:50%;" />

我们可以将用户 id 为 0 \~ 100 的数据存储在 Node 1，将用户 id 为 101 \~ 200 的数据存储在 Node 2，Client 根据用户 id 来决定访问哪个 Node。对于单个用户来说，读写操作都只能在某个节点上进行；对所有用户来说，有一部分用户的读写操作在 Node 1 上，有一部分用户的读写操作在 Node 2 上。

这样的设计有一个很明显的问题就是某个节点故障时，这个节点上的用户就无法进行读写操作了，但站在整体上来看，这种设计可以降低节点故障时受影响的用户的数量和范围，毕竟只影响 20%的用户肯定要比影响所有用户要好。这也是为什么挖掘机挖断光缆后，支付宝只有一部分用户会出现业务异常，而不是所有用户业务异常的原因。

- 正常运行情况下，不存在 CP 和 AP 的选择，可以同时满足 CA。

CAP 理论告诉我们分布式系统只能选择 CP 或者 AP，但其实这里的前提是系统发生了“分区”现象。如果系统没有发生分区现象，也就是说 P 不存在的时候（节点间的网络连接一切正常），我们没有必要放弃 C 或者 A，应该 C 和 A 都可以保证，这就要求架构设计的时候**既要考虑分区发生时选择CP还是AP，也要考虑分区没有发生时如何保证CA**。

同样以用户管理系统为例，即使是实现 CA，不同的数据实现方式也可能不一样：用户账号数据可以采用“消息队列”的方式来实现 CA，因为消息队列可以比较好地控制实时性，但实现起来就复杂一些；而用户信息数据可以采用“数据库同步”的方式来实现 CA，因为数据库的方式虽然在某些场景下可能延迟较高，但使用起来简单。

- 放弃并不等于什么都不做，需要为分区恢复后做准备。

CAP 理论告诉我们三者只能取两个，需要“牺牲”（sacrificed）另外一个，这里的“牺牲”是有一定误导作用的，因为“牺牲”让很多人理解成什么都不做。实际上，CAP 理论的“牺牲”只是说在分区过程中我们无法保证 C 或者 A，但并不意味着什么都不做。因为在系统整个运行周期中，大部分时间都是正常的，发生分区现象的时间并不长。例如，99.99%可用性（俗称 4 个 9）的系统，一年运行下来，不可用的时间只有 50 分钟；99.999%（俗称 5 个 9）可用性的系统，一年运行下来，不可用的时间只有 5 分钟。分区期间放弃 C 或者 A，并不意味着永远放弃 C 和 A，我们可以在分区期间进行一些操作，从而让分区故障解决后，系统能够重新达到 CA 的状态。

最典型的就是在分区期间记录一些日志，当分区故障解决后，系统根据日志进行数据恢复，使得重新达到 CA 状态。同样以用户管理系统为例，对于用户账号数据，假设我们选择了 CP，则分区发生后，节点 1 可以继续注册新用户，节点 2 无法注册新用户（这里就是不符合 A 的原因，因为节点 2 收到注册请求后会返回 error），此时节点 1 可以将新注册但未同步到节点 2 的用户记录到日志中。当分区恢复后，节点 1 读取日志中的记录，同步给节点 2，当同步完成后，节点 1 和节点 2 就达到了同时满足 CA 的状态。

而对于用户信息数据，假设我们选择了 AP，则分区发生后，节点 1 和节点 2 都可以修改用户信息，但两边可能修改不一样。例如，用户在节点 1 中将爱好改为“旅游、美食、跑步”，然后用户在节点 2 中将爱好改为“美食、游戏”，节点 1 和节点 2 都记录了未同步的爱好数据，当分区恢复后，系统按照某个规则来合并数据。例如，按照“最后修改优先规则”将用户爱好修改为“美食、游戏”，按照“字数最多优先规则”则将用户爱好修改为“旅游，美食、跑步”，也可以完全将数据冲突报告出来，由人工来选择具体应该采用哪一条。

## ACID

ACID 是数据库管理系统为了保证事务的正确性而提出来的一个理论，ACID 包含四个约束，下面我来解释一下。

1\.Atomicity（原子性）

一个事务中的所有操作，要么全部完成，要么全部不完成，不会在中间某个环节结束。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样。

2\.Consistency（一致性）

在事务开始之前和事务结束以后，数据库的完整性没有被破坏。

3\.Isolation（隔离性）

数据库允许多个并发事务同时对数据进行读写和修改的能力。隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

4\.Durability（持久性）

事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

可以看到，ACID 中的 A（Atomicity）和 CAP 中的 A（Availability）意义完全不同，而 ACID 中的 C 和 CAP 中的 C 名称虽然都是一致性，但含义也完全不一样。ACID 中的 C 是指数据库的数据完整性，而 CAP 中的 C 是指分布式节点中的数据一致性。再结合 ACID 的应用场景是数据库事务，CAP 关注的是分布式系统数据读写这个差异点来看，其实 CAP 和 ACID 的对比就类似关公战秦琼，虽然关公和秦琼都是武将，但其实没有太多可比性。

## BASE

BASE 是指基本可用（Basically Available）、软状态（ Soft State）、最终一致性（ Eventual Consistency），核心思想是即使无法做到强一致性（CAP 的一致性就是强一致性），但应用可以采用适合的方式达到最终一致性。

1\.基本可用（Basically Available）

分布式系统在出现故障时，允许损失部分可用性，即保证核心可用。

这里的关键词是“**部分**”和“**核心**”，具体选择哪些作为可以损失的业务，哪些是必须保证的业务，是一项有挑战的工作。例如，对于一个用户管理系统来说，“登录”是核心功能，而“注册”可以算作非核心功能。因为未注册的用户本来就还没有使用系统的业务，注册不了最多就是流失一部分用户，而且这部分用户数量较少。如果用户已经注册但无法登录，那就意味用户无法使用系统。例如，充了钱的游戏不能玩了、云存储不能用了……这些会对用户造成较大损失，而且登录用户数量远远大于新注册用户，影响范围更大。

2\.软状态（Soft State）

允许系统存在中间状态，而该中间状态不会影响系统整体可用性。这里的中间状态就是 CAP 理论中的数据不一致。

3\.最终一致性（Eventual Consistency）

系统中的所有数据副本经过一定时间后，最终能够达到一致的状态。

这里的关键词是“一定时间” 和 “最终”，“一定时间”和数据的特性是强关联的，不同的数据能够容忍的不一致时间是不同的。举一个微博系统的例子，用户账号数据最好能在 1 分钟内就达到一致状态，因为用户在 A 节点注册或者登录后，1 分钟内不太可能立刻切换到另外一个节点，但 10 分钟后可能就重新登录到另外一个节点了；而用户发布的最新微博，可以容忍 30 分钟内达到一致状态，因为对于用户来说，看不到某个明星发布的最新微博，用户是无感知的，会认为明星没有发布微博。“最终”的含义就是不管多长时间，最终还是要达到一致性的状态。

BASE 理论本质上是对 CAP 的延伸和补充，更具体地说，**是对CAP中AP方案的一个补充**。前面在剖析 CAP 理论时，提到了其实和 BASE 相关的两点：

- CAP 理论是忽略延时的，而实际应用中延时是无法避免的。

这一点就意味着完美的 CP 场景是不存在的，即使是几毫秒的数据复制延迟，在这几毫秒时间间隔内，系统是不符合 CP 要求的。因此 CAP 中的 CP 方案，实际上也是实现了最终一致性，只是“一定时间”是指几毫秒而已。

- AP 方案中牺牲一致性只是指分区期间，而不是永远放弃一致性。

这一点其实就是 BASE 理论延伸的地方，分区期间牺牲一致性，但分区故障恢复后，系统应该达到最终一致性。

综合上面的分析，ACID 是数据库事务完整性的理论，CAP 是分布式系统设计理论，BASE 是 CAP 理论中 AP 方案的延伸。

## 小结

今天我为你讲了深入理解 CAP 理论所需要特别关注的细节点，以及 ACID 和 BASE 两个相似的术语，这些技术细节在架构设计中非常关键，希望对你有所帮助。

这就是今天的全部内容，留一道思考题给你吧，假如你来设计电商网站的高可用系统，按照 CAP 理论的要求，你会如何设计？

欢迎你把答案写到留言区，和我一起讨论。相信经过深度思考的回答，也会让你对知识的理解更加深刻。（编辑乱入：精彩的留言有机会获得丰厚福利哦！）

## 精选留言(68)

- ![img](23%E6%83%B3%E6%88%90%E4%B8%BA%E6%9E%B6%E6%9E%84%E5%B8%88%EF%BC%8C%E4%BD%A0%E5%BF%85%E9%A1%BB%E6%8E%8C%E6%8F%A1%E7%9A%84CAP%E7%BB%86%E8%8A%82.resource/resize,m_fill,h_34,w_34.jpeg)

  luop

  结合这两期对 CAP 的学习和思考，谈下个人的理解。 设计分布式系统的两大初衷：横向扩展（scalability）和高可用性（availability）。“横向扩展”是为了解决单点瓶颈问题，进而保证高并发量下的「可用性」；“高可用性”是为了解决单点故障（SPOF）问题，进而保证部分节点故障时的「可用性」。由此可以看出，分布式系统的核心诉求就是「可用性」。这个「可用性」正是 CAP 中的 A：用户访问系统时，可以在合理的时间内得到合理的响应。 为了保证「可用性」，一个分布式系统通常由多个节点组成。这些节点各自维护一份数据，但是不管用户访问到哪个节点，原则上都应该读取到相同的数据。为了达到这个效果，一个节点收到写入请求更新自己的数据后，必须将数据同步到其他节点，以保证各个节点的数据「一致性」。这个「一致性」正是 CAP 中的 C：用户访问系统时，可以读取到最近写入的数据。 需要注意的是：CAP 并没有考虑数据同步的耗时，所以现实中的分布式系统，理论上无法保证任何时刻的绝对「一致性」；不同业务系统对上述耗时的敏感度不同。 分布式系统中，节点之间的数据同步是基于网络的。由于网络本身固有的不可靠属性，极端情况下会出现网络不可用的情况，进而将网络两端的节点孤立开来，这就是所谓的“网络分区”现象。“网络分区”理论上是无法避免的，虽然实际发生的概率较低、时长较短。没有发生“网络分区”时，系统可以做到同时保证「一致性」和「可用性」。 发生“网络分区”时，系统中多个节点的数据一定是不一致的，但是可以选择对用户表现出「一致性」，代价是牺牲「可用性」：将未能同步得到新数据的部分节点置为“不可用状态”，访问到这些节点的用户显然感知到系统是不可用的。发生“网络分区”时，系统也可以选择「可用性」，此时系统中各个节点都是可用的，只是返回给用户的数据是不一致的。这里的选择，就是 CAP 中的 P。 分布式系统理论上一定会存在 P，所以理论上只能做到 CP 或 AP。如果套用 CAP 中离散的 C/A/P 的概念，理论上没有 P 的只可能是单点（子）系统，所以理论上可以做到 CA。但是单点（子）系统并不是分布式系统，所以其实并不在 CAP 理论的描述范围内。

  作者回复: 写的非常好👍👍

  2018-08-24

  **24

  **392

- ![img](23%E6%83%B3%E6%88%90%E4%B8%BA%E6%9E%B6%E6%9E%84%E5%B8%88%EF%BC%8C%E4%BD%A0%E5%BF%85%E9%A1%BB%E6%8E%8C%E6%8F%A1%E7%9A%84CAP%E7%BB%86%E8%8A%82.resource/resize,m_fill,h_34,w_34-1661871622415257.jpeg)

  空档滑行

  一个电商网站核心模块有会员，订单，商品，支付，促销管理等。 对于会员模块，包括登录，个人设置，个人订单，购物车，收藏夹等，这些模块保证 AP，数据短时间不一致不影响使用。 订单模块的下单付款扣减库存操作是整个系统的核心，我觉得 CA 都需要保证，在极端情况下牺牲 P 是可以的。 商品模块的商品上下架和库存管理保证 CP,搜索功能因为本身就不是实时性非常高的模块，所以保证 AP 就可以了。 促销是短时间的数据不一致，结果就是优惠信息看不到，但是已有的优惠要保证可用，而且优惠可以提前预计算，所以可以保证 AP 现在大部分的电商网站对于支付这一块是独立的系统，或者使用第三方的支付宝，微信。其实 CAP 是由第三方来保证的，支付系统是一个对 CAP 要求极高的系统，C 是必须要保证的，AP 中 A 相对更重要，不能因为分区，导致所有人都不能支付

  作者回复: 分析思路很清晰

  2018-06-20

  **4

  **90

- ![img](23%E6%83%B3%E6%88%90%E4%B8%BA%E6%9E%B6%E6%9E%84%E5%B8%88%EF%BC%8C%E4%BD%A0%E5%BF%85%E9%A1%BB%E6%8E%8C%E6%8F%A1%E7%9A%84CAP%E7%BB%86%E8%8A%82.resource/132.jpeg)

  乘风

  老师 你上面说 redis 集群是互联且数据共享，但按官网上说 redis 集群后，每个主节点的数据是不一致的，也不存在共享数据，现在对 cap 中指的分布式系统(互联且数据共享)还是不太明白，谢谢老师

  作者回复: 这里的共享是指同一份数据在多个节点都有，但并不一定在所有节点都有，简单理解有数据复制的系统就是 CAP 应用的场景。 一份数据在多个节点有但不是所有节点都有，这是非对称集群；例如 ES 所有数据在所有节点都有，这是对称集群，例如 zookeeper

  2018-11-30

  **4

  **26

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  刘毅

  CAP 理解： 任何一个正常运行的分布式系统，起源于 CA 状态，中间（发生分区时）可能经过 CP 和 AP 状态，最后回到 CA 状态。 所以一个分布式系统，需要考虑实现三点： 1.正常运行时的 CA 状态。 2.发生分区时转变为 CP 或 AP 状态。 3.分区解决时变会 CA 状态。

  作者回复: 总结的非常好，第三点可以优化为“分区解决时如何恢复为 CA 状态”

  2020-06-16

  **

  **24

- ![img](https://static001.geekbang.org/account/avatar/00/10/d2/f4/c0ec0ea9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  飞

  独家解读啊，赞

  2018-06-19

  **

  **21

- ![img](https://static001.geekbang.org/account/avatar/00/14/dd/cb/23b114a7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  xiao 皮孩。。

  理解 CAP 理论的最简单方式是想象两个节点分处分区两侧。允许至少一个节点更新状态会导致数据不一致，即丧失了 C 性质。如果为了保证数据一致性，将分区一侧的节点设置为不可用，那么又丧失了 A 性质。除非两个节点可以互相通信，才能既保证 C 又保证 A，这又会导致丧失 P 性质。

  作者回复: 是的👍👍

  2019-04-02

  **7

  **17

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/c5/3467cf94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  正是那朵玫瑰

  老师有个疑问，像电商这样的系统，有订单系统和库存系统，创建订单会调用库存系统，这两个系统是互联的，但是并没有数据共享，但超时的情况下，会造成订单数据和库存数据状态不一致，这种算不算 cap 讨论范畴呢？

  作者回复: 不算，CAP 的应用范围已经明确了：互联，共享数据。 这种情况下的不一致靠对账，人工修复等方式解决

  2018-08-25

  **

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/10/d5/3f/80bf4841.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  文竹

  电商网站核心功能有用户、产品、订单、支付、库存，相应的数据有用户、产品、订单、支付、库存。 对于用户数据，选择 CP。因为用户注册后，可能几分钟后重新登录，所以需要满足一致性；在网络出现分区时，因为需要满足一致性而暂时不能提供写服务，所以无法满足可用性；对于分区容错性，只要能返回一个合理的响应就能满足，这一点能很好满足。 对于产品数据，无需满足一致性，所以选择 AP。 对于订单数据，业务需要满足一致性，所以选择 CP。 对于支付数据，业务需要满足一致性，所以选择 CP。 对于库存数据，业务需要满足一致性，所以选择 CP。

  作者回复: 思路很清晰👍👍

  2018-08-21

  **

  **13

- ![img](https://static001.geekbang.org/account/avatar/00/13/e5/39/951f89c8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  信信

  PACELC 定理了解一下？ 在理论计算机科学中，PACELC 定理是 CAP 定理的扩展。它指出，在分布式计算机系统中，在网络分区（P）的情况下，必须在可用性（A）和一致性（C）之间进行选择（根据 c a p 定理），但是（E），即使系统在没有分区的情况下正常运行，也必须在延迟（L）和一致性（C）之间进行选择。 PACELC 定理首先由耶鲁大学的 Daniel J. Abadi 在 2010 年的一篇博文中描述，他后来在 2012 年的一篇论文中正式确定。PACELC 的目的是声称他的论点“忽略复制系统的一致性/延迟权衡是一个主要的疏忽[在 CAP 中]，因为它在系统运行期间始终存在，而 CAP 仅在可以说是罕见的网络分区情况下才相关。

  作者回复: 多谢👍👍

  2020-03-20

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/c9/da/22328e47.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zhouwm

  CAP 理论延时问题：因为延时无法避免意味着完美 CP 场景不存在。—— 这个说法有问题，一致性是从外部使用者的角度去看的（黑盒），只要在回复应答前保障数据已经同步到其他节点，后续请求能得到新数据就是一致的，etcd 就是完美 cp，zk 其实也能算完美 cp。延时问题是系统设计的时候要考虑的点

  作者回复: 不会的，zk 是顺序一致性，保证不了完美 cp，raft 为了可理解而简化了异常处理，某些场景下会丢失数据

  2018-07-24

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/de/17/75e2b624.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  feifei

  电商网站要做高可用架构，就必须先确定业务模块，我没有过电商的经验，就说说我的理解，电商主要有商品，用户，交易，这 3 快核心 商品，商户发布某商品或者修改商品，用户查看商品，不需要非常强的一致性，即可以一部分用户先看到，可以使用最终一致性，满足可用性和容错性，可以使用发布队列，进行发布 用户，有普通的用户，商户这 2 种用户，用户模块的功能都是注册和登录，需要保证一致性，不能出现用户注册成功了，却不能登录。为了将宕机影响控制在最小，以用户进行分布，针对单节点可以使用数据库的主从，从节点分担读压力 交易，由于交易系统有强一致性要求，用户的交易要只能成功或者失败，有强事务的处理，考虑交易量大的问题，也按照用户进行分布，单节点采用数据库的集群，数据的多分写入 这是我的一个初步设计，还请老师指点，谢谢

  作者回复: 大概思路就是这样，按照业务分析

  2018-06-20

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/c9/da/22328e47.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zhouwm

  cap 的 c 指的是线性一致性还是顺序一致性？如果把 zk 改成读写请求都通过 master 的话，是否就算完美 CP 了？因为延时对外不可见。raft 在什么情况下丢数据，我理解的丢数据指的是给了请求者正确应答后，写入的数据没保存好，理论上不应该出现的（如果是磁盘缓存导致的，有可能），k8s 之类的用 etcd 也是基于 raft 的，如果会丢数据都无法保证的话，坑有点大，应该不会有人用吧

  作者回复: CAP 的 C 是严格意义上的绝对一致性，因为不考虑复制延时。 zk 全部走 master 就不符合 CAP 的 CP 定义了，因为 CAP 是要求各个节点都可以提供读写操作，而不是只做备份复制 Raft 论文中对于 leader 切换时可能覆盖日志给了一个详细的案例，这个案例不常见，发生概率非常小，而且只覆盖一条数据

  2018-07-31

  **3

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a7/d3/af0c1e4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我的名字叫浩仔丶

  老师，请教下 P 是什么时机才会触发分区呢？

  作者回复: 断网，网线断也可以，网卡坏也可以，交换机故障也可以

  2018-06-28

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/f2/ca989d6f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Leon Wong

  我理解 CA 和 CP 不是系统的整体设计，而是具体业务点的设计权衡，一种 tradeoff ，表现形式是分布式系统在检测到网络分区发生的时候，是倾向于一致性 C 或是可用性 A。而 CA 对应到的场景是不考虑分区容错的场景，数据有可能在整个分布式系统里只存一份(如果引入多副本相当于引入分区问题)，那么就会有单点故障的风险，只能通过将数据分散存储在不同服务器的做法来降低风险的影响面。 可见每一个设计都有优势，也有弊端，我们需要根据具体场景去做 tradeoff……

  作者回复: 架构设计就是判断和选择😀

  2018-06-21

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/fc/e5/605f423f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  肖一林

  在开源的分布式系统中，通常可以让用户配置，选择是 CP 还是 AP，比如 kafka，每种消息都可以选择配置。这就是 CAP 理论对数据粒度的控制。老师归纳的非常好。

  作者回复: 掌握了理论，看具体各种系统实现就比较容易理解了

  2018-06-20

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/05/e5/aa579968.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王磊

  系统架构中考虑 p 和不考虑 p，对外表现是什么?例如对于考虑 p 的系统，当 p 发生时，它还是 functon 的?发生分区意味着节点间不再联通，数据不再共享，要么为了 c 回复错误，此时丢了 a; 要么为了 a，但可能会返回过期数据，丢了 c。所以考虑了 p 比没考虑 p 的系统到底带来额外什么好处?

  作者回复: 考虑 P 的系统在分区场景下还能提供部分业务

  2018-06-19

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/13/50/2b/2344cdaa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  第一装甲集群司令克莱斯特

  CAP 追求强一致性是丰满的梦想，BASE 最终一致性才是骨感的现实！

  作者回复: 工程实现和理论模型有巨大的鸿沟

  2020-10-30

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/20/2d/dfa5bec8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Han

  您提到 CAP 是要求各个节点都可以提供读写操作，但现在大部分的分布式中间件实现基本都是一主多从架构，只有主节点才会接收写请求，主从节点都能接收读请求。 而且很多一致性算法都是设定只有主节点才接收写请求，然后把数据同步到从节点。 请问您怎么看？

  作者回复: 这样做是通过限制主才能写来避免数据一致性问题，但是复杂度转移到如何选举主上去了，这就是大家普遍认为选举比数据一致性要简单一些😁

  2020-07-18

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/36/80/a7167a3a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  姜华梁 taric

  老师，互联与共享数据，怎么理解，可以举一个实例嘛

  作者回复: memcache 集群不互联不共享数据，redis 集群互联且共享数据

  2018-11-19

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/d4/d2/96dbfa5a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云辉

  对 CAP 和 Base 了解更清楚了，原来 Base 是说出现 P 的情况下一种合适的解决方案

  2018-06-19

  **1

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/56/43/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿巍-豆夫

  老师，我一直很困惑，在 cp 的架构中，如果一旦发生分区，怎么保证 c 呢？ 连个节点网络都不可达了，怎么可能保证一致性呢？ ap 我能理解为很好实现。

  作者回复: 直接不允许写，或者分区节点不提供服务，参考 Paxos 或者 Zookeeper

  2018-09-19

  **2

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_88604f

  ​        电商架构从场景上看主要涉及到查询和下单，对查询操作来说可以满足 BASE 理论:正常情况下用户查询到某个商品有剩余，但是查询的时候却提示商品已售完，这也是可以接受的（即软状态）；当后端服务出现异常时可以通过降级等措施优先保证 A（即基本可用）；当用户实际下单的时候还会涉及到扣款和扣库存，通常采用 TCC 或 MQ 异步确保型等分布式事务技术来保证 C，异常情况下人工介入保证 E。          从业务特点上看电商业务流量大、时延低、可靠性要求高。要支持大流量后端必须是多实例，通过 ID 或 HASH 的方式将用户分配到不同实例上。每个实例对接独立的 DB。DB 可以选择单实例以保证 C，这样当实例故障后该分区的用户会业务不可用，由于只影响少量用户只要做到快速恢复即可，如果还要考虑数据迁移那就太复杂了。；也可以选择主备以保证 A（当选择 A 时主备可能发生脑裂，即 P）。        从数据上看分为用户数据、商品数据、订单数据，每种数据进一步细分为关键数据和非关键数据，关键数据要满足 C，非关键数据可以就考虑满足 A。这里的商品数据需要特别考虑，因为必须每个分区共享，但是由于同步的时间差是没有办法做到真正的 C 的。

  作者回复: 分析到位

  2019-09-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  电商系统，首先回想一下，我觉得涉及的东西相当庞杂，如果以是否对用户可见来划分，对用户可见的部分有账户管理、商品搜索管理、订单管理等，主要涉及用户登录，商品搜索列表页，商品详情页，商品结算页，商品支付页，订单列表页，订单详情页，售后商品页等。用户不可见的主要是商品的采购管理，商品的仓储管理，订单的物流管理，订单的退换货，订单的备件管理等，当然，还有秒杀、夺宝岛、时效等。根据 CAP 理论 P 不可避免，我们只能根据业务情况去权衡是选择 C 还是 A，我觉得只要和钱不直接挂钩的为了系统的高可用性都会选择 A，保证系统的可用性，数据一致性采用最终一致性，而和钱直接挂钩比如：支付，我觉得就应该选择 C 啦！即使提示支付失败牺牲点用户的体验，也不要造成用户的误会。 不过，这些情况发生的概率比较小，可能我们的系统相对用户感知度小，经过几次大促都没有出现什么特别大的问题。如果有问题，我们也会保 A 的，系统都不可用了，那比出现可弥补的错误要严重的多。

  作者回复: 分析思路很好

  2019-08-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/10/98/bba354ae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  丶丶丶丶

  意思就是 CAP 就是一个“保大保小”的选择题。需要规定好不同情况下遇到这个选择题时的答案，这样在遇到这种问题时不管是系统自动处理还是人工干预都有法可依

  2019-05-17

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/04/2a/462d01db.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  金蝉子

  CAP 细节： 1，CAP 关注的粒度是数据，而不是系统，系统中往往需要根据模块业务特性来选择是 AP 还是 CP 2，CAP 是忽略网络延迟的，在存在延迟的情况下，没有绝对的 CP，都是某种意义上的最终一致性 3，在没有分区出现的情况下，是可以同时满足 CA 的，并不是说在分区出现的情况下，可以牺牲掉 C 或者 A，而是要在分区恢复的时候能够自愈 4，BASE 理论是对 AP 的延伸，达到最终一致性

  2019-05-16

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/f1/97/e3bbbb14.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  胖胖的程序猿

  怎么理解共享数据，假如 A,B 两个不同服务相连，并且公用同一个数据库表，这个是否算共享，是否算 CAP 范围。看上篇是要节点间发生数据复制情况才在 cap 范围，那一般是中间件系统才会有节点数据复制，web 应用一般都不会有数据复制吧

  作者回复: 你说的这个不算，分布式节点通过复制来共享数据才算

  2019-04-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/d9/a2/afbc447c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  海军上校

  选择 ap 后，节点 2 如何感知到分区存在，然后返回错误的～不太懂

  作者回复: 心跳也可以，别人通知它也可以，取决于不同的架构方式，你看看高可用集群的架构模式

  2018-07-25

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/d6/60/f21b2164.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  jacy

  只从电商网站的核心业务分析，不考虑登录注册等业务，商品信息显示可细分为关键商品信息（如价格，库存）和非关键信息（商品介绍，用户评论），可按以前提到的分表分库的方法进行信息切割，对关键信息采用 ca,非关键信息采用 ap,最终达到 base 即可。

  作者回复: 思路正确，区分不同数据

  2018-06-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/f5/4fb5284b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_59a17f

  综合上面的分析，ACID 是数据库事务完整性的理论，CAP 是分布式系统设计理论，BASE 是 CAP 理论中 AP 方案的延伸。

  2018-06-25

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/ce/2b/a47a0936.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  何国平

  解析到位，分布式系统无法保持完全一致性，只能保持最终一致性

  2018-06-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ac/a1/43d83698.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云学

  能够结合实践讲解理论才是真本事，特喜欢这种文章

  2018-06-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/2e/27/b4/df65c0f7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  | ~浑蛋~![img](https://static001.geekbang.org/resource/image/d5/62/d57fddfa5b2c2ce3071e92304da8af62.png)

  数据库的数据完整性是啥意思

  2022-06-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/22/e1/6b/74a8b7d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Hugh

  既然 CAP 是忽略网络延迟的，无法做到完美的一致性，那为什么说 CAP 的一致性是指强一致性呢？

  作者回复: 理论的角度来说是强一致性的

  2022-05-12

  **2

  **

- ![img](23%E6%83%B3%E6%88%90%E4%B8%BA%E6%9E%B6%E6%9E%84%E5%B8%88%EF%BC%8C%E4%BD%A0%E5%BF%85%E9%A1%BB%E6%8E%8C%E6%8F%A1%E7%9A%84CAP%E7%BB%86%E8%8A%82.resource/132-1661871622417285.jpeg)

  Geek_06b952

  现在大厂好像用 raft 协议比较多。能简单介绍一下 raft 吗？

  作者回复: 简化版的 Paxos，另外，raft 官网本身就有很好的介绍资料: https://raft.github.io/ 

  2022-04-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/4b/d7/f46c6dfd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  William Ning

  看了文章和评论以及回复，感觉大佬很多～ Keep Learning～

  作者回复: 评论卧虎藏龙

  2022-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/0b/21/f1aea35b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  let_me_go

  CAP：在网络发生分区时，我们只能在 CP 和 AP 之间选择，但是之后要进行补偿来同时满足 CA，在正常情况下，一个分布式系统能同时具备 C 和 A。在网络发生分区时，选择 CP，由于各节点之间的数据同步也需要花费时间，不可能做到时时刻刻各个节点之间的数据都保持一致，完美的 CP 场景实际是不存在的。理论和实践是有差距的，需要取舍，如果就是要强一致性的，那么可以写操作限制在单台机器上，不做分布式写，读可以分布式读。选择 AP，仅仅是在发生网络分区这样的故障的时候放弃了一致性 C，分区故障恢复后，系统应该要达到最终一致性。 ACID：Atomicity(原子性)，Consistency(一致性)，Isolation(隔离性)，Durability(持久性)。ACID 主要都是针对事务而言。原子性，事务具有原子性，整个执行过程不会被打断，和并发编程中的原子性类似。一致性，要求数据具有一致性，事务要么成功，要么失败，事务执行完成后不存在第三种状态，数据也是，要么是事务执行成功后的状态，要么是执行前的状态。隔离性，事务之间是隔离的，引申出来就有数据库的隔离级别。读未提交，读已提交，可重复读，串行化。持久性，事务一旦成功，数据就一定能持久化，不会因为宕机等原因重启数据库导致丢数据，引申出来就有 mysql 的 bin log ，redo log 等机制。 BASE：BASE 是指基本可用(Basically Available)，软状态(Soft State)，最终一致性(Eventual Consistency)，可以看作是 CAP 的衍生，发生网络分区 P 时，选择 A，但是当系统恢复以后，我们最终还是需要达到 C。发生网络分区 P 时，选择 C，实践很难达到强一致性，通常是保证最终一致性即可。

  2022-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/26/eb/d7/90391376.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  if...else...![img](https://static001.geekbang.org/resource/image/d5/62/d57fddfa5b2c2ce3071e92304da8af62.png)

  厉害了，老师

  2022-03-17

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKHxicHW07jz5vB9I8QAonrDXrcFmOS9CtqufVexs0wY1YxH7picctcTMOiaibgVvwkQX3UcicqqUXWTYQ/132)

  Geek_aq

  干货满满 核心思想讲解的很真切！

  作者回复: 谢谢 ：）

  2022-02-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/23/18/4284361f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  易飞

  理论理解了，真正去落地还是有难度的

  作者回复: 难在“取舍”二字

  2022-01-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/04/fe/60db4684.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜波

  zk 全部走 master 就不符合 CAP 的 CP 定义了，因为 CAP 是要求各个节点都可以提供读写操作，而不是只做备份复制 ==== 老师，如 mysql 主从这样的架构，写走主库，读走从库，应该属与 CAP 中什么场景呢？

  作者回复: 你先自己做出判断，如果自己没法做出判断那就说明疑惑在哪里，不要直接问一个答案

  2021-09-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/04/fe/60db4684.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜波

  zk 全部走 master 就不符合 CAP 的 CP 定义了，因为 CAP 是要求各个节点都可以提供读写操作，而不是只做备份复制

  作者回复: CAP 没有要求各个节点都可以读写呀，再去看看 CP 和 AP 的具体含义

  2021-09-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/18/75/68487e89.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  川川

  老师我想问下，你这里说的 C 是绝对的一致性无法实现，是面向客户端来说只要能够读到最新写入的数据即可。但是我们在读的情况下使用 Qurom 算法，是否可以让这个 C 得到保证

  作者回复: 如果能够保证 Quorum 多数存活，那么使用 Paxos 一类的算法是可以保证 C 的，但是 CAP 说的是指整个系统在任何时刻，如果 Paxos 系统不足 Quorum 数量的节点存在的话，就无法提供服务了

  2021-06-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/9f/c0/86febfff.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Master

  电商网站，我觉得可以分为两个大块：商品，订单。 商品，包括消息展示，活动促销等等，这块理论上可以简单的用 AP 理论来进行保证，因为可能商品消息更新没那么敏感，只要用户看到商品即可，商品具体的内容变更不敏感，但是这里有几个属性比较敏感，促销活动信息（618，双十一）以及对应的价格等关键数据，这里我认为有必要 CP，要不就都展示旧的，要不就索性都不展示，具体理论很简单，避免引起歧义纠纷。总结不是单纯的 AP，也要考虑 CP 订单，这个体系就比较关键了，从简单到复杂来说，下单后订单信息，这里我认为要 AP，因为用户下单后，订单就必须要按照预期合理响应，说白了和银行存钱一样。其次针对那种下单是秒杀，抢购的，这里要走 CP，理论最终秒杀结果都是异步计算，最终用户即使当时操作的时候是可以秒，但是结论没有秒中都是可以接收，目前也是日常实际比较用户可接受的范畴。

  作者回复: 分析的很详细，点赞

  2021-05-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_b6e6bc

  老师  通过这节理解到的是 cap 讨论的是数据的中心，那么对于 cap 的讨论我们关注的中心应该是存储系统，像数据库、redis 这种，不是应用服务这类，

  作者回复: 应用服务也可以用 CAP 去衡量，异地多活章节就会讲如何基于 CAP 来设计异地多活方案

  2021-05-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/b7/3e/7d93176e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  麦麦

  CP:一致性+分区容错性，适用写一致性场景 AP：可用性+分区容错性，适用读及时性场景

  2021-04-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/b7/3e/7d93176e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  麦麦

  P 要求分布式和数据同步，C 要求数据完全一致，A 要求返回及时 CAP 关注的粒度是数据，而不是整个系统。 CAP 是针对于局部数据的，而不是全局系统的。 以一个最简单的用户管理系统为例，用户管理系统包含用户账号数据（用户 ID、密码）、用户信息数据（昵称、兴趣、爱好、性别、自我介绍等）。通常情况下，用户账号数据会选择 CP，而用户信息数据会选择 AP，如果限定整个系统为 CP，则不符合用户信息数据的应用场景；如果限定整个系统为 AP，则又不符合用户账号数据的应用场景。 CP:一致性+分区容错性，适用写一致性场景 AP：可用性+分区容错性，适用读及时性场景

  2021-04-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d1/29/1b1234ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  DFighting

  读到这里算是慢慢有点理解 CAP 了，首先，网络环境下，只要系统节点间互联+数据共享，那么 P 就是很常见的一个场景，那么余下的就是如何选择 CA 了，以这个电商平台为例，我们可以分为用户、商品、订单和支付数据，其中每个数据又可能存在增删改查几种情况，忽略后者，就这几种数据本身来看， 用户数据，只要最终数据一致，问题都不大，所以应该是 AP 商品数据，涉及到库存的扣减等直接的数据操作，用户感知其实不大，所以得优先保障 CP 支付数据，这个数据很敏感，绝对不可以做，也不能无法支付，所以可以考虑单点 CA 订单数据，这部分我不太理解订单是啥的抽象，感觉和商品数据关联性比较大，只要最终订单提交成功，我觉得都可以，所以应该是 CP 看到这里，感觉直接面对用户的数据抽象 A>C,后台的重要数据 C>A，涉及到钱的肯定是 CA 都要

  作者回复: 订单数据就是你的购买记录，主要是订单有状态，状态值要保证幂等和最终正确，因此订单用 AP 也可以。

  2021-04-10

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/21/78/732a2e33.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  M1racle

  关于 cp 记录日志有个问题？当节点 2 不可用的时候，节点 1 记录日志，那么这时候算是注册成功还是什么呢？如果注册成功，那么很多逻辑都需要调整，比如登录，权限验证什么的。如果注册未成功，那么记录日志的用意是什么呢？

  作者回复: 当然是注册成功了，原文有详细描述日志作用

  2020-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/13/ae/fe9f93f0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  DaWang

  ACID 是数据库事务完整性的理论，CAP 是分布式系统设计理论，BASE 是 CAP 理论中 AP 方案的延伸

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/1b/70/547042ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谭方敏

  ACID 是数据库事务完整性的理论（A-原子性，要么执行完毕，要么不执行，不至于有啥中间状态，如果有错误状态了，则会执行回滚操作；C-一致性，在事务操作发生前后，数据库完整性没变；I-隔离性，多事务操作读写的时候相对独立；D-永久性，事务操作后，数据就不会丢失，永久保存了） CAP 是分布式系统设计理论， BASE 是 CAP 理论中 AP 方案的延伸（BA-基本可用，在故障情况下，可以允许部分功能出问题，而核心功能可用；S-软状态，也就是中间状态，数据不一致性；E-最终一致性，系统副本经过一段时间后，最终能达成一致性状态） cap 理论在落地的时候，需要考虑一些场景： 1）cap 关心的是数据并非系统或者节点。并且需要考虑系统在不同场景下的数据情况，不然会完成系统内部 cap 跟系统 cap 相冲突。 2）cap 忽略网络延迟。在数据复制过程中，网络延迟是非常大问题，这样决定数据的一致性状态。 3）正因为数据场景复杂化，所以不存在必然的只能考虑 ap 和 cp，而不能选 ca。如果不能分区的话，那么 ca 也是一个可选项。 4）放弃了并不意味着不需要恢复。cap 理论只能代表某个状态，之后还可以状态的恢复，比如用 cp，丢失了 a，到之后可能就找回 a，变 ap 了。

  2020-03-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/23/a2/e2df1b88.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mr.wrong

  李老师，你文章说没有完美的 cp 理论，实际上还是会有延迟。比如同步复制的情况，节点 a 的修改得等节点 b 数据同步之后，才给客户端响应呢？这样不算有延迟吧

  作者回复: 实际上这个延迟对业务有影响的

  2020-02-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8f/5b/2f16ca95.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  悟空

  内容很赞 学到了东西

  2019-12-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/2d/ca/02b0e397.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fomy

  必须满足 CP 的功能：扣减库存，充值余额 必须满足 AP 的功能：下订单，收藏，生成物流信息，加入购物车

  2019-11-24

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  鲁班

  P 为“分区容错” 理解起来有歧义，一般会认为是出现分区的时候系统也是可用的因为容错了，这就和 C 可用性又有些重合，如果 CAP 简单说成 在网络分区时不能同时达成一致性和可用性，这样很好理解

  作者回复: C 不是可用，是强一致性

  2019-10-18

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/4d/bc/e26f63d9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  炎黄伙哥

  以前也没少搜 CAP 的解释，但这次学习之后确实理解了不少，尤其是分区这个概念，以前最不理解的就是这个，然而现在看来，CAP 的讨论的基础恰恰就是在分区发生的情况下，这点不理解又怎么可能理解整个概念

  作者回复: 是的，这点是关键

  2019-10-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/47/a0/f12115b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Sam.张朝

  指数据库中数据在逻辑上的一致性、正确性、有效性和相容

  2019-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/12/f8/888a9b9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  随风

  电商系统：对于用户、订单、支付这些核心关键数据要求保证实时一致性，因此，需要满足 CP。如果某节点故障，那么只能提示用户系统异常，而不能返回一个旧数据。而对于商品、库存、地址信息等这些数据可以采用 AP、即时某台机器故障、或者机器间复制数据延迟，导致用户看不到部分商品并不会有大问题，系统可以继续提供服务。

  作者回复: 库存是强一致性的，而且是最难的

  2019-08-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/8f/b1/7b697ed4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓晨同学

  我理解 cap 理论更适用于中间件和数据库层面，像电商系统非要说 cap 的话就牵扯到数据库集群采用哪种思想，比如订单类的采用分库分表保证 ca，商品或者用户不重要的信息可以采用读写分离保证 ap，老师您觉得呢

  作者回复: CAP 是关于单个数据的，不是整个系统的

  2019-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/69/187b9968.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  南山![img](https://static001.geekbang.org/resource/image/bb/65/bba6d75e3ea300b336b4a1e33896a665.png)

  cap 需要各个节点都有一份数据，并且每个节点都允许对数据进行读写，任意一节点的数据发生变更，数据都会同步到其他节点，那么这样是不是有多少个节点，就有多少份完全一样的数据呢(正常情况下)？这样的场景有没有实际的例子或者开源并且知名的系统或者服务呢？

  作者回复: 我没有研究过所有的开源系统，不敢说一定没有，但真有的话，性能和复制是很难做到很好的，要保证一致性性能会低，不保证一致性数据会乱

  2019-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9b/a4/ddb31c5b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小狮子辛巴

  只有我一个看不懂吗？

  作者回复: 说出你的问题，我们一起探讨

  2018-11-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/36/2c/8bd4be3a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小喵喵

  ACID 中的 C 和 I 我看了好多资料，还是不太理解，请老师能举例说明一下，谢谢。

  作者回复: 找本数据库原理看，没法一两句话说明白的，另外，写 2 个事务尝试一下

  2018-10-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/22/e0/8e58a7e1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小思绪

  那 zk 属于 cp 还是 ap 呢？当大部分节点可用时，zk 依然能对外提供服务，对于在大部分范围内的节点来说，可用性和一致性都得到了保证，客户端了解到这部分节点，依然能正常 function。

  作者回复: CP，原因你自己再对照专栏内容分析一下

  2018-09-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5b/30/9dcf35ff.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  森林

  有一些点有不赞同，1.cap 定义里就说明了，在合理时间内能提供服务，合理时间就是延时。2.cap 通常针对副本，base 通常针对整个大型事务的最终一致，也就是多条记录。

  作者回复: CAP 针对副本这句话有点歧义啊，具体还是通过副本来实现高可用，然后需要保证副本一致性，这里的副本就是数据副本，可以是文件，可以是关系数据库，base 也不是只限于大型事务的，数据复制和一致性都可以

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f0/21/104b9565.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小飞哥 ‍超級會員

  cap 和 base 都是指数据。我是做前台的。我们电商划分前台是中台原子化接口的组装业务。并没有事务

  2018-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fd/fd/1e3d14ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  王宁

  对于核心的数据如账户数据采用 CP 架构，通过了一致性 Hash 的算法将用户数据分散的在Ｎ个主节点之上. 对于商品描述类的数据采取 AP 架构，因为数据的无状态保持数据的可用性。 秒杀可以采取内存存储方案等实现,对于秒杀的结果采用 CP 架构,但是秒杀的记录等可以采取 AP 架构. 总结来说对于强一致性要求的如用户帐户采取 CP 架构，对于无状态或者状态不是强要求的采取 AP，逐渐过渡吧。 象购买记录、活动等场景依据场景选择合适的 CP 架构和延迟时间。 高可用还可以在系统负荷过高时采取前端限流，和后端采取降级和熔断等方案。 另外： 用户账号数据可以采用“消息队列”的方式来实现，这里的消息队列是怎么理解的。

  作者回复: A 机房创建一个账户后，通过消息队列告诉 AB 机房相关信息，B 机房拿到消息后也创建这个账户

  2018-07-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fa/fb/ef99d6ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tiger

  电商网站，包含交易，支付，库存，用户系统，交易和库存都应该是 CA 系统，要么是成功，要么是失败。 而支付满足最终一致性即可，正常情况下，用户支付完成一定时间后才需要发货处理，并且可通过退款等方式解决问题所以可以设计成 AP 系统。

  2018-06-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/d0/29/271cf37c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lhc

  请问网络在什么情况下会出现分区容错？

  作者回复: 断网最常见，断网的原因就很多了

  2018-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ee/d2/7024431c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  探索无止境

  “通常情况下，用户账号数据会选择 CP，而用户信息数据会选择 AP"，这个选择的依据是什么？

  作者回复: 业务对一致性的要求不一样
