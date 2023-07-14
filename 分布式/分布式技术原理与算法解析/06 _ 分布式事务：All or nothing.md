# 06 \| 分布式事务：All or nothing

作者: 聂鹏程

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/f0/cc/f07225825571263cb0c7379b03e989cc.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/b0/9a/b0dfd7a26b72bd08e039ef34b9468a9a.mp3" type="audio/mpeg"></audio>

你好，我是聂鹏程。今天，我来继续带你打卡分布式核心技术。

对于网上购物的每一笔订单来说，电商平台一般都会有两个核心步骤：一是订单业务采取下订单操作，二是库存业务采取减库存操作。

通常，这两个业务会运行在不同的机器上，甚至是运行在不同区域的机器上。针对同一笔订单，当且仅当订单操作和减库存操作一致时，才能保证交易的正确性。也就是说一笔订单，只有这两个操作都完成，才能算做处理成功，否则处理失败，充分体现了“All or nothing”的思想。

在分布式领域中，这个问题就是分布式事务问题。那么今天，我们就一起打卡分布式事务吧。

## 什么是分布式事务？

在介绍分布式事务之前，我们首先来看一下什么是事务。

事务（Transaction）提供一种机制，将包含一系列操作的工作序列纳入到一个不可分割的执行单元。只有所有操作均被正确执行才能提交事务；任意一个操作失败都会导致整个事务回滚（Rollback）到之前状态，即所有操作均被取消。简单来说，事务提供了一种机制，使得工作要么全部都不做，要么完全被执行，即all or nothing。

通常情况下，我们所说的事务指的都是本地事务，也就是在单机上的事务。而事务具备四大基本特征ACID，具体含义如下。

<!-- [[[read_end]]] -->

- **A：原子性（Atomicity**），即事务最终的状态只有两种，全部执行成功和全部不执行，不会停留在中间某个环节。若处理事务的任何一项操作不成功，就会导致整个事务失败。一旦操作失败，所有操作都会被取消（即回滚），使得事务仿佛没有被执行过一样。就好比买一件商品，购买成功时，则给商家付了钱，商品到手；购买失败时，则商品在商家手中，消费者的钱也没花出去。
- **C：一致性（Consistency）**，是指事务操作前和操作后，数据满足完整性约束，数据库保持一致性状态。比如，用户A和用户B在银行分别有800元和600元，总共1400元，用户A给用户B转账200元，分为两个步骤，从A的账户扣除200元和对B的账户增加200元。一致性就是要求上述步骤操作后，最后的结果是用户A还有600元，用户B有800元，总共1400元，而不会出现用户A扣除了200元，但用户B未增加的情况（该情况，用户A和B均为600元，总共1200元）。
- **I：隔离性（Isolation）**，是指当系统内有多个事务并发执行时，多个事务同时使用相同的数据时，不会相互干扰，每个事务都有一个完整的数据空间，对其他并发事务是隔离的。也就是说，消费者购买商品这个事务，是不影响其他消费者购买的。
- **D：持久性（Durability）**，也被称为永久性，是指一个事务被执行后，那么它对数据库所做的更新就永久地保存下来了。即使发生系统崩溃或宕机等故障，重新启动数据库系统后，只要数据库能够重新被访问，那么一定能够将其恢复到事务完成时的状态。就像消费者在网站上的购买记录，即使换了手机，也依然可以查到。

<!-- -->

只有在数据操作请求满足上述四个特性的条件下，存储系统才能保证处于正确的工作状态。因此，无论是在传统的集中式存储系统还是在分布式存储系统中，任何数据操作请求都必须满足 ACID 特性。

**分布式事务，就是在分布式系统中运行的事务，由多个本地事务组合而成。**在分布式场景下，对事务的处理操作可能来自不同的机器，甚至是来自不同的操作系统。文章开头提到的电商处理订单问题，就是典型的分布式事务。

分布式事务由多个事务组成，因此基本满足ACID，其中的C是强一致性，也就是所有操作均执行成功，才提交最终结果，以保证数据一致性或完整性。但随着分布式系统规模不断扩大，复杂度急剧上升，达成强一致性所需时间周期较长，限定了复杂业务的处理。为了适应复杂业务，出现了BASE理论，该理论的一个关键点就是采用最终一致性代替强一致性。我会在“知识扩展”模块与你详细展开BASE理论这部分内容。

介绍完什么是事务和分布式事务，以及它们的基本特征后，就进入“怎么做”的阶段啦。所以接下来，我们就看看如何实现分布式事务吧。

## 如何实现分布式事务？

实际上，分布式事务主要是解决在分布式环境下，组合事务的一致性问题。实现分布式事务有以下3种基本方法：

- 基于XA协议的二阶段提交协议方法；
- 三阶段提交协议方法；
- 基于消息的最终一致性方法。

<!-- -->

其中，基于XA协议的二阶段提交协议方法和三阶段提交协议方法，采用了强一致性，遵从ACID。基于消息的最终一致性方法，采用了最终一致性，遵从BASE理论。下面，我将带你一起学习这三种方法。

### 基于XA协议的二阶段提交方法

XA是一个分布式事务协议，规定了事务管理器和资源管理器接口。因此，XA协议包括事务管理器和本地资源管理器两个部分。

**XA实现分布式事务的原理，就类似于我在**[**第3讲**](<https://time.geekbang.org/column/article/141772>)**中与你介绍的集中式算法**：事务管理器相当于协调者，负责各个本地资源的提交和回滚；而资源管理器就是分布式事务的参与者，通常由数据库实现，比如Oracle、DB2等商业数据库都实现了XA接口。

基于 XA协议的二阶段提交方法中，二阶段提交协议（Two-phase Commit Protocol，2PC），用于保证分布式系统中事务提交时的数据一致性，是XA在全局事务中用于协调多个资源的机制。

那么，**两阶段提交协议如何保证分布在不同节点上的分布式事务的一致性呢**？为了保证它们的一致性，我们需要引入一个协调者来管理所有的节点，并确保这些节点正确提交操作结果，若提交失败则放弃事务。接下来，我们看看两阶段提交协议的具体过程。

两阶段提交协议的执行过程，分为投票（Voting）和提交（Commit）两个阶段。

首先，我们看一下**第一阶段投票**：在这一阶段，协调者（Coordinator，即事务管理器）会向事务的参与者（Cohort，即本地资源管理器）发起执行操作的CanCommit请求，并等待参与者的响应。参与者接收到请求后，会执行请求中的事务操作，将操作信息记录到事务日志中但不提交（即不会修改数据库中的数据），待参与者执行成功，则向协调者发送“Yes”消息，表示同意操作；若不成功，则发送“No”消息，表示终止操作。

当所有的参与者都返回了操作结果（Yes或No消息）后，**系统进入了第二阶段提交阶段**（也可以称为，执行阶段）。在提交阶段，协调者会根据所有参与者返回的信息向参与者发送DoCommit（提交）或DoAbort（取消）指令。具体规则如下：

- 若协调者从参与者那里收到的都是“Yes”消息，则向参与者发送“DoCommit”消息。参与者收到“DoCommit”消息后，完成剩余的操作（比如修改数据库中的数据）并释放资源（整个事务过程中占用的资源），然后向协调者返回“HaveCommitted”消息；
- 若协调者从参与者收到的消息中包含“No”消息，则向所有参与者发送“DoAbort”消息。此时投票阶段发送“Yes”消息的参与者，则会根据之前执行操作时的事务日志对操作进行回滚，就好像没有执行过请求操作一样，然后所有参与者会向协调者发送“HaveCommitted”消息；
- 协调者接收到来自所有参与者的“HaveCommitted”消息后，就意味着整个事务结束了。

<!-- -->

接下来，**我以用户A要在网上下单购买100件T恤为例，重点与你介绍下单操作和减库存操作这两个操作**，帮助你加深对二阶段提交协议的理解。

第一阶段：订单系统中将与用户A有关的订单数据库锁住，准备好增加一条关于用户A购买100件T恤的信息，并将同意消息“Yes”回复给协调者。而库存系统由于T恤库存不足，出货失败，因此向协调者回复了一个终止消息“No”。

![](<https://static001.geekbang.org/resource/image/8a/6a/8a880c358c5f1a1fe9c8cc8179d6b56a.png?wh=471*444>)

第二阶段：由于库存系统操作不成功，因此，协调者就会向订单系统和库存系统发送“DoAbort”消息。订单系统接收到“DoAbort”消息后，将系统内的数据退回到没有用户A购买100件T恤的版本，并释放锁住的数据库资源。订单系统和库存系统完成操作后，向协调者发送“HaveCommitted”消息，表示完成了事务的撤销操作。

至此，用户A购买100件T恤这一事务已经结束，用户A购买失败。

![](<https://static001.geekbang.org/resource/image/bd/5c/bd73d10eb000ee554a448d169344f95c.png?wh=552*495>)

由上述流程可以看出，**二阶段提交的算法思路可以概括为**：协调者向参与者下发请求事务操作，参与者接收到请求后，进行相关操作并将操作结果通知协调者，协调者根据所有参与者的反馈结果决定各参与者是要提交操作还是撤销操作。

虽然基于XA的二阶段提交算法尽量保证了数据的强一致性，而且实现成本低，但依然有些不足。主要有以下三个问题：

- **同步阻塞问题**：二阶段提交算法在执行过程中，所有参与节点都是事务阻塞型的。也就是说，当本地资源管理器占有临界资源时，其他资源管理器如果要访问同一临界资源，会处于阻塞状态。因此，基于XA的二阶段提交协议不支持高并发场景。
- **单点故障问题：**该算法类似于集中式算法，一旦事务管理器发生故障，整个系统都处于停滞状态。尤其是在提交阶段，一旦事务管理器发生故障，资源管理器会由于等待管理器的消息，而一直锁定事务资源，导致整个系统被阻塞。
- **数据不一致问题：**在提交阶段，当协调者向所有参与者发送“DoCommit”请求时，如果发生了局部网络异常，或者在发送提交请求的过程中协调者发生了故障，就会导致只有一部分参与者接收到了提交请求并执行提交操作，但其他未接到提交请求的那部分参与者则无法执行事务提交。于是整个分布式系统便出现了数据不一致的问题。

<!-- -->

### 三阶段提交方法

三阶段提交协议（Three-phase Commit Protocol，3PC），是对二阶段提交（2PC）的改进。为了更好地处理两阶段提交的同步阻塞和数据不一致问题，**三阶段提交引入了超时机制和准备阶段**。

- 与2PC只是在协调者引入超时机制不同，3PC同时在协调者和参与者中引入了超时机制。如果协调者或参与者在规定的时间内没有接收到来自其他节点的响应，就会根据当前的状态选择提交或者终止整个事务，从而减少了整个集群的阻塞时间，在一定程度上减少或减弱了2PC中出现的同步阻塞问题。
- 在第一阶段和第二阶段中间引入了一个准备阶段，或者说把2PC的投票阶段一分为二，也就是在提交阶段之前，加入了一个预提交阶段。在预提交阶段尽可能排除一些不一致的情况，保证在最后提交之前各参与节点的状态是一致的。

<!-- -->

三阶段提交协议就有CanCommit、PreCommit、DoCommit三个阶段，下面我们来看一下这个三个阶段。

**第一，CanCommit阶段。**

协调者向参与者发送请求操作（CanCommit请求），询问参与者是否可以执行事务提交操作，然后等待参与者的响应；参与者收到CanCommit请求之后，回复Yes，表示可以顺利执行事务；否则回复No。

3PC的CanCommit阶段与2PC的Voting阶段相比：

- 类似之处在于：协调者均需要向参与者发送请求操作（CanCommit请求），询问参与者是否可以执行事务提交操作，然后等待参与者的响应。参与者收到CanCommit请求之后，回复Yes，表示可以顺利执行事务；否则回复No。
- 不同之处在于，在2PC中，在投票阶段，若参与者可以执行事务，会将操作信息记录到事务日志中但不提交，并返回结果给协调者。但在3PC中，在CanCommit阶段，参与者仅会判断是否可以顺利执行事务，并返回结果。而操作信息记录到事务日志但不提交的操作由第二阶段预提交阶段执行。

<!-- -->

CanCommit阶段不同节点之间的事务请求成功和失败的流程，如下所示。

![](<https://static001.geekbang.org/resource/image/56/7c/56fe63b378ed63d24e318af22419bb7c.png?wh=752*458>)

当协调者接收到所有参与者回复的消息后，进入预提交阶段（PreCommit阶段）。

**第二，PreCommit阶段。**

协调者根据参与者的回复情况，来决定是否可以进行PreCommit操作（预提交阶段）。

- 如果所有参与者回复的都是“Yes”，那么协调者就会执行事务的预执行：

- 协调者向参与者发送PreCommit请求，进入预提交阶段。

- 参与者接收到PreCommit请求后执行事务操作，并将Undo和Redo信息记录到事务日志中。

- 如果参与者成功执行了事务操作，则返回ACK响应，同时开始等待最终指令。

- 假如任何一个参与者向协调者发送了“No”消息，或者等待超时之后，协调者都没有收到参与者的响应，就执行中断事务的操作：

- 协调者向所有参与者发送“Abort”消息。

- 参与者收到“Abort”消息之后，或超时后仍未收到协调者的消息，执行事务的中断操作。


<!-- -->

预提交阶段，不同节点上事务执行成功和失败的流程，如下所示。

![](<https://static001.geekbang.org/resource/image/4e/45/4edc9b1a1248825d62db5b9107b09045.png?wh=754*458>)

预提交阶段保证了在最后提交阶段（DoCmmit阶段）之前所有参与者的状态是一致的。

**第三，DoCommit阶段。**

DoCmmit阶段进行真正的事务提交，根据PreCommit阶段协调者发送的消息，进入执行提交阶段或事务中断阶段。

- **执行提交阶段：**

- 若协调者接收到所有参与者发送的Ack响应，则向所有参与者发送DoCommit消息，开始执行阶段。

- 参与者接收到DoCommit消息之后，正式提交事务。完成事务提交之后，释放所有锁住的资源，并向协调者发送Ack响应。

- 协调者接收到所有参与者的Ack响应之后，完成事务。

- **事务中断阶段：**

- 协调者向所有参与者发送Abort请求。

- 参与者接收到Abort消息之后，利用其在PreCommit阶段记录的Undo信息执行事务的回滚操作，释放所有锁住的资源，并向协调者发送Ack消息。

- 协调者接收到参与者反馈的Ack消息之后，执行事务的中断，并结束事务。


<!-- -->

执行阶段不同节点上事务执行成功和失败(事务中断)的流程，如下所示。

![](<https://static001.geekbang.org/resource/image/7e/0f/7e332feee4fb6b6fd67689b66cc2610f.png?wh=797*487>)

3PC协议在协调者和参与者均引入了超时机制。即当参与者在预提交阶段向协调者发送 Ack消息后，如果长时间没有得到协调者的响应，在默认情况下，参与者会自动将超时的事务进行提交，从而减少整个集群的阻塞时间，在一定程度上减少或减弱了2PC中出现的同步阻塞问题。

但三阶段提交仍然存在数据不一致的情况，比如在PreCommit阶段，部分参与者已经接受到ACK消息进入执行阶段，但部分参与者与协调者网络不通，导致接收不到ACK消息，此时接收到ACK消息的参与者会执行任务，未接收到ACK消息且网络不通的参与者无法执行任务，最终导致数据不一致。

### 基于分布式消息的最终一致性方案

2PC和3PC核心思想均是以集中式的方式实现分布式事务，这两种方法都存在两个共同的缺点，一是，同步执行，性能差；二是，数据不一致问题。为了解决这两个问题，通过分布式消息来确保事务最终一致性的方案便出现了。

在eBay的分布式系统架构中，架构师解决一致性问题的核心思想就是：将需要分布式处理的事务通过消息或者日志的方式异步执行，消息或日志可以存到本地文件、数据库或消息队列中，再通过业务规则进行失败重试。这个案例，就是使用**基于分布式消息的最终一致性方案**解决了分布式事务的问题。

基于分布式消息的最终一致性方案的事务处理，引入了一个消息中间件（在本案例中，我们采用Message Queue，MQ，消息队列），用于在多个应用之间进行消息传递。实际使用中，阿里就是采用RocketMQ 机制来支持消息事务。

基于消息中间件协商多个节点分布式事务执行操作的示意图，如下所示。

![](<https://static001.geekbang.org/resource/image/9c/30/9c48c611124574c64806f45f62f8b130.png?wh=771*591>)

仍然以网上购物为例。假设用户A在某电商平台下了一个订单，需要支付50元，发现自己的账户余额共150元，就使用余额支付，支付成功之后，订单状态修改为支付成功，然后通知仓库发货。

在该事件中，涉及到了订单系统、支付系统、仓库系统，这三个系统是相互独立的应用，通过远程服务进行调用。

![](<https://static001.geekbang.org/resource/image/f6/45/f687a6a05dac8e974a4dac04e1ce1a45.png?wh=901*543>)

根据基于分布式消息的最终一致性方案，用户A通过终端手机首先在订单系统上操作，通过消息队列完成整个购物流程。然后整个购物的流程如下所示。

![](<https://static001.geekbang.org/resource/image/d9/a4/d9b2d32660e49a4ea613871337b570a4.png?wh=919*502>)

1. 订单系统把订单消息发给消息中间件，消息状态标记为“待确认”。
2. 消息中间件收到消息后，进行消息持久化操作，即在消息存储系统中新增一条状态为“待发送”的消息。
3. 消息中间件返回消息持久化结果（成功/失败），订单系统根据返回结果判断如何进行业务操作。失败，放弃订单，结束（必要时向上层返回失败结果）；成功，则创建订单。
4. 订单操作完成后，把操作结果（成功/失败）发送给消息中间件。
5. 消息中间件收到业务操作结果后，根据结果进行处理：失败，删除消息存储中的消息，结束；成功，则更新消息存储中的消息状态为“待发送（可发送）”，并执行消息投递。
6. 如果消息状态为“可发送”，则MQ会将消息发送给支付系统，表示已经创建好订单，需要对订单进行支付。支付系统也按照上述方式进行订单支付操作。
7. 订单系统支付完成后，会将支付消息返回给消息中间件，中间件将消息传送给订单系统。若支付失败，则订单操作失败，订单系统回滚到上一个状态，MQ中相关消息将被删除；若支付成功，则订单系统再调用库存系统，进行出货操作，操作流程与支付系统类似。

<!-- -->

在上述过程中，可能会产生如下异常情况，其对应的解决方案为：

1. 订单消息未成功存储到MQ中，则订单系统不执行任何操作，数据保持一致；
2. MQ成功将消息发送给支付系统（或仓库系统），但是支付系统（或仓库系统）操作成功的ACK消息回传失败（由于通信方面的原因），导致订单系统与支付系统（或仓库系统）数据不一致，此时MQ会确认各系统的操作结果，删除相关消息，支付系统（或仓库系统）操作回滚，使得各系统数据保持一致；
3. MQ成功将消息发送给支付系统（或仓库系统），但是支付系统（或仓库系统）操作成功的ACK消息回传成功，订单系统操作后的最终结果（成功或失败）未能成功发送给MQ，此时各系统数据可能不一致，MQ也需确认各系统的操作结果，若数据一致，则更新消息；若不一致，则回滚操作、删除消息。

<!-- -->

基于分布式消息的最终一致性方案采用消息传递机制，并使用异步通信的方式，避免了通信阻塞，从而增加系统的吞吐量。同时，这种方案还可以屏蔽不同系统的协议规范，使其可以直接交互。

在不需要请求立即返回结果的场景下， 这些特性就带来了明显的通信优势，并且通过引入消息中间件，实现了消息生成方（如上述的订单系统）本地事务和消息发送的原子性，采用最终一致性的方式，只需保证数据最终一致即可，一定程度上解决了二阶段和三阶段方法要保证强一致性而在某些情况导致的数据不一致问题。

可以看出，分布式事务中，当且仅当所有的事务均成功时整个流程才成功。所以，**分布式事务的一致性是实现分布式事务的关键问题，目前来看还没有一种很简单、完美的方案可以应对所有场景。**

### 三种实现方式对比

现在，为了方便你理解并记忆这三种方法，我总结了一张表格，从算法一致性、执行方式、性能等角度进行了对比：

![](<https://static001.geekbang.org/resource/image/9c/2b/9c789a486aa8df6d9d12182b953a862b.jpg?wh=3826*2049>)

## 知识扩展：刚性事务与柔性事务

在讨论事务的时候，我们经常会提到刚性事务与柔性事务，但却很难区分这两种事务。所以，今天的知识扩展内容，我就来和你说说什么是刚性事务、柔性事务，以及两者之间有何区别？

- 刚性事务，遵循ACID原则，具有强一致性。比如，数据库事务。
- 柔性事务，其实就是根据不同的业务场景使用不同的方法实现最终一致性，也就是说我们可以根据业务的特性做部分取舍，容忍一定时间内的数据不一致。

<!-- -->

总结来讲，与刚性事务不同，柔性事务允许一定时间内，数据不一致，但要求最终一致。而柔性事务的最终一致性，遵循的是BASE理论。

那，**什么是BASE理论**呢？

eBay 公司的工程师 Dan Pritchett曾提出了一种分布式存储系统的设计模式——BASE理论。 BASE理论包括基本可用（Basically Available）、柔性状态（Soft State）和最终一致性（Eventual Consistency）。

- 基本可用：分布式系统出现故障的时候，允许损失一部分功能的可用性，保证核心功能可用。比如，某些电商618大促的时候，会对一些非核心链路的功能进行降级处理。
- 柔性状态：在柔性事务中，允许系统存在中间状态，且这个中间状态不会影响系统整体可用性。比如，数据库读写分离，写库同步到读库（主库同步到从库）会有一个延时，其实就是一种柔性状态。
- 最终一致性：事务在操作过程中可能会由于同步延迟等问题导致不一致，但最终状态下，所有数据都是一致的。

<!-- -->

BASE理论为了支持大型分布式系统，通过牺牲强一致性，保证最终一致性，来获得高可用性，是对ACID原则的弱化。ACID 与 BASE 是对一致性和可用性的权衡所产生的不同结果，但二者都保证了数据的持久性。ACID 选择了强一致性而放弃了系统的可用性。与ACID原则不同的是，BASE理论保证了系统的可用性，允许数据在一段时间内可以不一致，最终达到一致状态即可，也即牺牲了部分的数据一致性，选择了最终一致性。

具体到今天的三种分布式事务实现方式，二阶段提交、三阶段提交方法，遵循的是ACID原则，而消息最终一致性方案遵循的就是BASE理论。

## 总结

我从事务的ACID特性出发，介绍了分布式事务的概念、特征，以及如何实现分布式事务。在关于如何实现分布式的部分，我以网购为例，与你介绍了常见的三种实现方式，即基于XA协议的二阶段提交方法，三阶段方法以及基于分布式消息的最终一致性方法。

二阶段和三阶段方法是维护强一致性的算法，它们针对刚性事务，实现的是事务的ACID特性。而基于分布式消息的最终一致性方案更适用于大规模分布式系统，它维护的是事务的最终一致性，遵循的是BASE理论，因此适用于柔性事务。

在分布式系统的设计与实现中，分布式事务是不可或缺的一部分。可以说，没有实现分布式事务的分布式系统，不是一个完整的分布式系统。分布式事务的实现过程看似复杂，但将方法分解剖析后，你就会发现分布式事务的实现是有章可循的。

我将实现分布式事务常用的三个算法整理为了一张思维导图，以帮助你加深理解与记忆。

![](<https://static001.geekbang.org/resource/image/3d/13/3dabbddf3eab0297c2d154245ccb3c13.png?wh=795*777>)

## 思考题

你觉得分布式互斥与分布式事务之间的关系是什么呢？

我是聂鹏程，感谢你的收听，欢迎你在评论区给我留言分享你的观点，也欢迎你把这篇文章分享给更多的朋友一起阅读。我们下期再会！

## 精选留言(80)

- ![img](06%20_%20%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%EF%BC%9AAll%20or%20nothing.resource/resize,m_fill,h_34,w_34.jpeg)

  约书亚

  疑问不少 1. 2pc和3pc的第一步到底是不是“类似”？从本文中看，2pc corhort 收到CanCommit已经开始执行事务但不提交，3pc则写着在PreCommit阶段开始预提交。文中说二者第一步“类似”，但其实是非常不类似的吧？ 2. 我看到的资料里，3pc出现的目的，并不是文中说的为了解决那两个问题，因为这两个问题的解决方案在2pc中也可以引入。同步阻塞问题和本地事务隔离性相关。数据不一致在两者也都会出现。3pc多了一步，这些就能避免了么？ 3pc很多机制在这里没提到，这些才是真正对比2pc的改进。比如只要coordinator收到大多数ack，第三阶段即进入commit状态。本质上3pc并不能像共识算法那样保证一致性，而是比起2pc，增加了在一些未知状态下，“状态可能是成功”的判断依据。 3. 分布式消息中间件实现事务，重点是回查，这点没提啊。

  作者回复: 从你的提问中，可以看出你很爱思考。首先，非常抱歉，因为要忙每周的上线稿子定稿和后续新章节，所以没能及时回复，后面我会注意这个问题，尽量及时回复，希望理解。下面看一下这三个问题： 1. 2pc和3pc的第一步在文中的类似是指，均是通过协调者询问参与者是否可以正常执行事务提交操作，参与者也都会给协调者回复。在2pc中，如果所有参与者都返回结果后，会进入第二阶段的提交阶段，也可以说是执行阶段，根据第一阶段的投票结果，进行提交或取消。在3pc中，进入真正的提交阶段前，会有一个预提交阶段，这个预提交阶段不会做真正的提交，会将相关信息记录到事物日志中，当所有参与者都返回YES后，才会进入真正的提交阶段或执行阶段。 2. 3pc通过在协调者和参与者均引入了超时机制（2pc只是在协调者引入了超时）减少了整个集群的阻塞时间，在一定程度上减少或减弱了2pc中出现的同步阻塞问题；3pc中引入了预提交阶段，相对于2pc来讲，相当于增加了一个缓冲，保证了在最后提交阶段之前各参与节点的状态是一致的。 3. 不知道你说的回查是不是指的“失败重试”？如果是的话，这个和业务设计有关系，在介绍ebay系统中有提到，但本文主要是针对基于分布式消息的最终一致性方案的原理进行介绍，所以没有展开介绍。

  2019-10-04

  **17

  **46

- ![img](https://static001.geekbang.org/account/avatar/00/10/62/1e/8054e6db.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  樂文💤

  不太明白基于消息队列的方法如何解决数据不一致的问题 如果现在我有四个功能模块 前三个都成功了 按照文中所示协调者已经将前三个模块数据作出修改 但此时如果第四个模块数据更新失败的话 前三个模块如何做到回滚 因为前三个模块都没有锁定数据库资源

  2019-10-16

  **7

  **25

- ![img](https://static001.geekbang.org/account/avatar/00/14/b0/b0/30b29949.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  愚人

  基于消息的分布式事务，所给的例子中，如果支付失败或者出货，如何触发“订单系统”回滚？此外，这里的订单，支付和仓库三个节点更像是流水线，而不是事务的概念

  2019-11-16

  **

  **20

- ![img](https://static001.geekbang.org/account/avatar/00/14/30/2c/3feb407d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  倾听

  2pc和3pc都会存在数据不一致问题，为什么属于强一致性方案？

  2019-11-22

  **1

  **12

- ![img](https://static001.geekbang.org/account/avatar/00/13/33/9f/8dbd9558.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  逆流的鱼

  三阶段为什么就不阻塞了？没明白

  作者回复: 不阻塞的主要原因是在三阶段中引入了超时机制，有了超时机制协调者不用死等其它节点，其它节点也无需死等协调者，从而不会造成堵塞。

  2019-10-10

  **3

  **12

- ![img](06%20_%20%E5%88%86%E5%B8%83%E5%BC%8F%E4%BA%8B%E5%8A%A1%EF%BC%9AAll%20or%20nothing.resource/resize,m_fill,h_34,w_34-16622229096361048.jpeg)

  忆水寒

  分布式互斥是访问某一个共享资源防止产生不一致现象。分布式事务，是保证一组命令的完整执行或完全不执行。过程来看，都是保证数据一致性的手段。但是两者又不一样，互斥不可以回退（除非发起反向新操作），事务可以回退。

  2019-10-04

  **2

  **9

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_a1d0be

  内容较浅适合框架学习，细节还需要找更多的资料。异常处理基本没有，这些才是一个协议的精华。

  2021-01-03

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6b/f8/b4da7936.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大魔王汪汪

  可靠消息这种方式必须采用mq吗？使用db是不是也可以，看起来只是一个事务状态的存储和管理，是多个两阶段提交的组合啊！

  作者回复: 你这个问题提的很好！没有说必须采用MQ，但是MQ天生就是为了高效信息交互而生，所以我在这里是以MQ进行讲解的。使用DB自然是可以的，不过考虑到DB为了保证各种约束而产生的开销，性能上肯定会打一定的折扣。

  2019-10-06

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/14/20/02/df2bfda9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  公共号

  分布式消息中间件咋解决分布式事务回滚呢，觉得只是搞了个中间件把同步调用改成异步了，记录了状态。

  2019-10-12

  **1

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/14/4b/4b/97926cba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Luciano李鑫

  总结了很久，总结出来我的问题是，文中提到的两阶段提交或者三阶段提交应该算是协议层的抽象，想知道具体实现这些协议的项目中协调者和参与者分别是哪些，和他们的关系。

  2019-10-12

  **1

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/16/ed/89/86340059.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  licong

  三阶段也有一个协调者，为什么就不会有单点故障了？

  作者回复: 不是绝对地没有单点故障问题了，是一定程度上减少了单点故障带来的问题。三阶段协议中，参与者也引入了超时机制，如果长时间没有得到协调者的响应，在默认情况下，参与者会自动将超时的事务进行提交，不会像两阶段提交那样被阻塞住。

  2019-10-09

  **2

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek3340

  三阶段提交，没有拿买衣服的例子去讲每一步做了什么，只讲过程。就理解不了了。没有讲明白。下面大家问的问题，答的又不彻底。建议这篇重新讲下。用一个例子，贯穿全文。

  2020-09-09

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/17/06/7e/735968e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  西门吹牛

  对于2PC： 只要第一阶段确定了的，第二阶段一定要按照第一阶段的确定结果执行提交或者回滚，也就是说，第一阶段，如果参与者都回复YES，那么第二阶段，所有参与者必须全部提交成功，并且只能成功不能失败，要保证只能成功，就必须要有重试机制，一次不行，俩次，这个重试时间越长就会导致资源占用越久，也就说同步阻塞问题。 至于老师说的数据不一致问题，个人感觉不太准确，2PC就是保证一致性的事物协议，如果第二阶段，发送断网，或者节点故障，那么在网络恢复后，或者节点恢复后，可以根据持久化的日志，继续执行第二阶段的提交，直到成功。典型的例子：mysql日志提交就是二阶段提交，binlog和redo log，就是靠2pc解决一致性的。

  2020-08-18

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  留言区卧虎藏龙，老师互动量不足呀！ 2PC/3PC/BASE理论之前也学习过也许是先入为主的缘故，这节我也觉得老师讲的不太好😁 正如其他同学的疑问一样3PC的协调者，如果是个集群，单点问题是能解决，否则应该还存在？不阻塞确实说不通？

  2020-02-14

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/39/fa/a7edbc72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  安排

  上节说到分布式共识算法是保障系统满足不同程度一致性的核心算法，那这里的2pc,3pc可以认为是一种分布式共识算法吗？ 还有paxos,2pc,3pc等等，这些一致性算法都用在哪些场景呢？最主要的是paxos的应用场景有哪些？

  2019-12-11

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/c0/6c/29be1864.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  随心而至

  老师能不能放一些参考链接放出来，或者一些引申的链接

  2019-10-05

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fd/66/6d4c2e9b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  乔巴

  基于分布式消息的最终一致性方案。介绍里的5.6.7完全没有理解。 如果下单成功，那么下单的消息也应该发出去。后续的支付仓储和下单没有半毛钱关系了。 假设 下单，支付，仓储履约 要放到一个事务，支付的结果是怎么通过同一个mq再告诉下单呢？如果不是同一个消息 我觉得事务型消息只是保证了消息发送与本地事务的原子性。下单的本地事务提交成功，通知mq把事务消息变为可发送，到此事务性消息的事务也就结束了。然后下游业务方消费消息就好了。

  2021-01-25

  **2

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  voyager

  感觉讲的2PC、3PC和MQ是两个维度的事情，2PC、3PC是为了解决分布式事务。MQ是一个同步转异步的问题。且不论MQ带来的延迟影响，光是MQ如何实现实务，如何回滚操作也没有说清楚，那么如何使用MQ做到分布式一致呢

  2020-12-31

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/1b/87/37/b071398c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  等风来🎧

  你好，应该是3PC把2PC的投票阶段再次一分为二，不是把提交阶段一分为二了吧

  作者回复: 可以这么理解，也可以理解为3PC是在2PC的第一阶段和第二阶段之间引入了一个预提交阶段。

  2020-06-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/15/61/b2/f36c1d40.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  破发者

  老师好： 3pc在 DoCommit 阶段，当参与者向协调者发送 Ack 消息后，整个事务不就结束了吗？为什么文章里还说如果长时间没有得到协调者的响的话参与者会自动将超时的事务进行提交。

  作者回复: 3PC协议在协调者和参与者均引入了超时机制。即当参与者在预提交阶段向协调者发送 Ack消息后，如果长时间没有得到协调者的响应，在默认情况下，参与者会自动将超时的事务进行提交，从而减少整个集群的阻塞时间，在一定程度上减少或减弱了2PC中出现的同步阻塞问题。

  2020-05-28

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/8d/c5/898b13b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  亢（知行合一的路上）

  老师对比了分布式事务的三种实现方式，分布式消息牺牲掉强一致性，增大了并发度和性能，在妥协中取得更大的收益。 老师的这种学习方法值得借鉴，对比每种方式的优劣，才能真正理解👍

  2020-03-10

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/3c/74/580d5bbb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lucas

  最后的总结表格中  3pc和2pc都有单点问题和资源锁定问题吧  为什么表格中写的是无呢

  2020-03-06

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/19/f2/f5/46c23dd0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  leechanx

  3PC的第一步为什么感觉上没什么用。。。

  2020-01-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/09/42/1f762b72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Hurt

  老师  基于 XA 协议的二阶段提交方法，三阶段方法以及基于分布式消息的最终一致性方法  都有哪些成熟的系统 能给举几个例子吗 想了解一下

  2020-01-07

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/08/1f/b24a561d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  ～风铃～

  两阶段提交或者三阶段提交的应用场景一般只在数据库之间吧，而且数据库要能支持相关的协议。 比如记录回滚日志，如果不是数据库本身🈶️实现，其他资源要实现很复杂

  2019-11-11

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_xiaoer

  \1. "在 DoCommit 阶段，当参与者向协调者发送 Ack 消息后，如果长时间没有得到协调者的响应，在默认情况下，参与者会自动将超时的事务进行提交，不会像两阶段提交那样被阻塞住。" 在"DoCommit"阶段，当参与者向协调者发送 Ack 消息后，协调者需要响应吗？ 2. "为了解决两阶段提交的同步阻塞和数据不一致问题，三阶段提交引入了超时机制和准备阶段" "2PC 和 3PC 这两种方法，有两个共同的缺点，***二是，没有解决数据不一致的问题。" 所以3PC有没有解决数据一致性问题？ 3. 说实话，感觉很多东西都没讲清楚，有点失望 4. 期待讲的细致一点，把问题讲清楚，而不是把概念仍一下就完了

  2019-11-07

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/3c/cf/fa5c5123.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿星

  基于分布式消息实现最终一致性方案的事物回滚流程是否可以补充一下？

  2019-10-14

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/4b/4b/97926cba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Luciano李鑫

  请问老师，以mysql为例 1.他的事务实现是2PC还是3PC? 2.所谓的协调者是应用程序吗？

  2019-10-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/a5/98/a65ff31a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  djfhchdh

  个人感觉应该是分布式事务在并发时会用到分布式互斥，比如基于XA协议，在多个分布式事务并发时，哪个会占用事务管理器资源，这时就会用到分布式互斥;基于分布式消息，消息中间件是不用互斥占用的，但是与消息中间件交互的其他资源管理模块（比如订单模块，支付模块等）在多个事务并发的情况下，就需要有互斥机制了

  2019-10-04

  **

  **1

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKmyjUJe2FxeyL5VMuJlpCFeJKy4SYpicbpCgyPSqbiafPlhibQT2fLWJzqV1ANSDiaSMDVTJVGyAnIow/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  wangshanhe

  2PC方式，下面有一个疑问点： 在这一阶段，首先是，协调者（Coordinator，即事务管理器，一般为应用端）会向事务的参与者（Cohort，即本地资源管理器，一般为数据库端）发起执行操作的 CanCommit 请求，并等待参与者的响应。参与者接收到请求后，会执行请求中的事务操作，将操作信息记录到事务日志中但不提交（即不会修改数据库中的数据），待参与者执行成功，则向协调者发送“Yes”消息，表示同意操作；若不成功，则发送“No”消息，表示终止操作。 问题：事务管理器，一般为应用端，那多个应用服务应用，像订单服务和库存服务，他们是去中心的，怎么去平衡一个中心节点来统一处理所有节点的执行逻辑呢，这个是怎么实现的啊？

  2022-08-07

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erp3Is6ckmsGqtTP8t7xTQkrLUcOcFstAkE565FmYFibawfVIXTtu1vXtXibpHZK5dBL2F42ZpM0iamA/132)

  InfoQ_7b1f2555555f

  老师，下面这段描述是不是有问题？ACK不是协调者接收的吗？ “但三阶段提交仍然存在数据不一致的情况，比如在 PreCommit 阶段，部分参与者已经接受到 ACK 消息进入执行阶段，但部分参与者与协调者网络不通，导致接收不到 ACK 消息，此时接收到 ACK 消息的参与者会执行任务，未接收到 ACK 消息且网络不通的参与者无法执行任务，最终导致数据不一致。”

  2022-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/10/7d/567d53d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  暖冬

  基于最终一致性的分布式方案中，是怎么做回滚的，如果订单创建成功，调用支付系统支付也成功，说明数据中已经把用户的钱扣除了，也就是说数据库事务已经提交了。此时如果调用仓库系统出货失败，怎么对支付系统中已经支付的钱进行回滚

  2022-05-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/1e/d7/7d28a531.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  飞机翅膀上

  这章可以很简单也可以很深入，但说的既不适合新手也不适合老手。

  2022-04-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/03/c5/600fd645.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  tianbingJ

  TCC、Saga、Percolator也该介绍介绍，起码比XA用的多

  2022-03-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/38/8a/4dd15bd9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  牛不才

  确实有点看不懂这篇写的，不考虑重新讲解下这个章节吗？核心读者问到的问题，作为授业者是否应该给予解答一下。不要老回答那些大家都懂的东西！

  2021-11-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/24/c2/ee/f0aeca63.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_d46017

  不管是2pc还是3pc,协调者最终都需要接收参与者commit的确认ack，此时ack的丢失协调者会执行默认的操作，如果和实际应执行的操作不一致，那么会出现不一致的情况。不知道这样理解有问题吗？

  2021-04-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/f5/9d/104bb8ea.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek2014

  3pc到底为啥有一致性问题，这两段没看懂： 3PC 协议在协调者和参与者均引入了超时机制。即当参与者在预提交阶段向协调者发送 Ack 消息后，如果长时间没有得到协调者的响应，在默认情况下，参与者会自动将超时的事务进行提交，从而减少整个集群的阻塞时间，在一定程度上减少或减弱了 2PC 中出现的同步阻塞问题。但三阶段提交仍然存在数据不一致的情况，比如在 PreCommit 阶段，部分参与者已经接受到 ACK 消息进入执行阶段，但部分参与者与协调者网络不通，导致接收不到 ACK 消息，此时接收到 ACK 消息的参与者会执行任务，未接收到 ACK 消息且网络不通的参与者无法执行任务，最终导致数据不一致。 第一段不是说超时会自动提交吗，怎么会有一致性问题？第二段更是看不懂，参与者不是发送ack的吗？哪来的接收ack消息？到底是哪个阶段会导致一致性有问题，能不能给小白我讲述清楚呢？我是真看不懂。第二阶段协调者没收到所有ack怎么办？这块也没写清楚。协调者接收到所有参与者发送的 Ack 响应，则向所有参与者发送 DoCommit 消息，这不是得收到所有ack才向参与者发送执行命令吗？怎么会收到部分ack就执行了呢？大佬，基本功要扎实啊，真心没看懂

  2021-04-08

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  15130223197

  分布式互斥和分布式事务，它们都是为了解决分布式中数据一致性的问题，但是应用的场景不一样，分布式互斥通常就是指分布式锁，它用于解决多进程间的共享资源操作的一致性问题，具体场景有多实例间只有一个定时任务执行或者只有一个文件写操作执行等；但是分布式事务指的是原子性操作被拆分到不同服务，一次操作要保证这些服务最终要同时成功失败。核心区别就是多实例和多服务的区别。

  2021-02-12

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Zzz

  问个问题，mpp数据库的事务处理是不是使用了二阶段提交或者三阶段提交

  2020-10-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/66/77/194ba21d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lzh

  在纸上记录对比了一下文中的2PC和3PC，就我个人理解： 2PC，阶段一：执行事务（锁资源），返回结果YES或NO。阶段二：收到DoCommit，提交事务（释放资源）。若协调者挂了，参与者收不到DoCommit也没超时机制，则一直不释放资源了，即同步阻塞+单点故障的问题。 3PC，阶段一，询问是否可执行，参与者检查各种状态然后返回YES或NO（相当于对协调者做出一种承诺）。阶段二，PreCommit，执行事务操作（锁资源），返回ACK。阶段三，参与者收到DoCommit，则提交事务（释放资源），若收不到DoCommit，则timeout后自动提交（为什么能大胆的提交？因为之前已向协调者承诺过YES了，并且也收到PreCommit了---说明其他节点也是承诺YES的），这表示只要有PreCommit，则资源一定会被释放，即便协调者挂了，也不会像2PC那样所有节点的资源都被锁住。 所以3PC的阶段一还是有存在的意义的，至少能做出YES的承诺，后面timeout自动提交事务时也是有足够理由的。 但若3PC的某些参与者连PreCommit都没收到？那这些节点也不会执行事务，也不会锁住资源，这就出现了数据不一致的问题 个人拙见，只是看完文章觉得这么理解是最顺的

  2020-09-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_224f63

  基于消息的分布式事务，没有看明白如何保证事务，老师能不能展开讲一下

  2020-09-06

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/c2/6b/149d9ab1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李树增

  三阶段提交相关疑问，希望能得到老师的解答，不胜感激！ Q1:对于CanCommit阶段，协调者如果超时还没有收到参与者的消息，后续是否发送DoAbort消息。 Q2:对于PreCommit阶段，如果协调者发送了DoAbort消息，但是参与者超时还没有收到协调者的消息，是否是当成DoAbort处理。 Q3:对于DoCommit阶段，如果参与者提交事务失败会如何处理，看文章中似乎没有提及。

  2020-08-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/68/73/3cda533e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  乔良qiaoliang

  有个叫saga 的动词，和2pc，3pc 是一类功能么

  2020-07-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  丽丽

  在众多MQ中，选择哪款MQ用来实现事务的最终一致性比较靠谱？支持消息状态的变更和消息系统回查事务状态。

  2020-06-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6c/ea/ce9854a5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  坤

  非核心链路功能的降级处理，这里的降级具体指的什么？可以解释一下吗？

  2020-06-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/71/4c/2cefec07.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  静水流深

  Saga模式是属于本节中的哪种方法呢？

  2020-06-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/47/00/3202bdf0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  piboye

  我觉得老师可以分两步来讲，先讲正常流程，让大家了解个大概过程。再细分讲各种异常情况。目前发现是异常情况讲的太少了，之前的bully算法就完全没讲异常的处理，网上搜索讲es的实现才明白

  2020-05-06

  **

  **

- ![img](https://wx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKfhxOIicKh6QauLw53mMd7XpjoqZwqngb2InjbjofuhoEXY1UEpvgmS0kCw3GmCBoxqj6TJzlLADw/132)

  Geek_1aa247

  分布式互斥指的是在分布式系统里，排他性的资源访问方式，它可以通过增加’协调者‘解决； 分布式事务中为了保证一致性，也增加了’协调者‘，我认为在这一点上他们两个有很强的相似性。 不知道理解的对不对，请老师指正。

  2020-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/7a/93/c9302518.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  高志强

  发现个问题，二阶段提交当协调者发送doAbort，参与者回复的为什么不是haveAbort 而是haveCommit ，这有点迷糊了。

  2020-04-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/15/9c9ca35c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sipom

  二阶段、三阶段、tic

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5c/e6/1a823214.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鸭先知

  分布式互斥和分布式事物都是为了数据一致性，二者侧重点不同，分布式互斥侧重资源的使用，分布式事物侧重步骤的协作

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0e/39/174741d1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  特种流氓

  老师 在分布式领域是不是不存在一个真正支持事务操作的数据库

  2020-03-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/f1/a4/79c36b87.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  流云

  分布式互斥是多个节点要读写一个共享的临界资源。 分布式事务如本文所言，是在分布式环境的多个本地事务组成的一个满足ACID特性的事务，分布式事务本身可能会需要多个节点采用分布式互斥的算法来访问共享资源。

  2020-03-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/b6/f8/e5f7ebb7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  冰雨寒

  老师，能简单介绍一下saga吗？

  2020-02-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/25/00/3afbab43.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  88591

  老师 三阶段提交对如下情况是怎么处理的（只考虑协调者挂了的情况，不考虑参与者挂了，并且一阶段完成的情况下）。 2.1、 协调者未发出preCommit 挂了 ，参与者 超时未收到 doCommit/doAbort ,回滚事务。 2.2、协调者发出部分preCommit 挂了 ，参与者 超时未收到 doCommit/doAbort ,回滚事务。 2.3、协调者发全部 preCommit 挂了 ，参与者 超时未收到 doCommit/doAbort ,?。  3.1、协调者收到了 所有 preCommit 的ACK ,协调者未发出doCommit/doAbort 挂了，参与者超时未收到 doCommit/doAbort ,?。 3.2、协调者收到了 所有 preCommit 的ACK ,协调者发出部分doCommit/doAbort 挂了，参与者超时未收到 doCommit/doAbort ,?。 3.3、协调者收到了 所有 preCommit 的ACK ,协调者发出全部doCommit/doAbort 挂了，参与者超时未收到 doCommit/doAbort ,提交/回滚事务。

  2020-01-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5d/52/458a3b68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  农村闲散劳动力

  2PC和3PC的目标都是维护强一致性，但由于网络分区的存在，实际上是无法完全实现强一致性的，因此采用消息队列的方式，索性不要强一致性了，最终一致性就好

  2020-01-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5d/52/458a3b68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  农村闲散劳动力

  “三阶段提交协议（Three-phase commit protocol，3PC），是对二阶段提交（2PC）的改进。为了解决两阶段提交的同步阻塞和数据不一致问题” 这一句应该是“为了解决两阶段提交的同步阻塞和单点故障问题“吧，在网络隔离情况下，数据不一致问题并没有解决

  2020-01-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5d/52/458a3b68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  农村闲散劳动力

  新增加的预提交阶段，就是为了让参与者知道投票阶段的结果： 1. 如果投票阶段后参与者没能顺利进入下一个阶段，参与者可知协调者可能宕机，会放弃提交或选出新的协调者 2. 如果投票阶段后参与者进入了预提交阶段，参与者可知其他参与者也在打算commit。此时即便协调者宕机，导致在超时结束后参与者还未收到协调者的提交命令，参与者也会默认提交

  2020-01-07

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/9f/4a/f5b8c6b3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Walker Jiang

  点个赞，写的不错

  2019-12-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4e/90/a8d19e7b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张理查

  分布式技术原理与算法#Day6 看完了分布式互斥及其实现分布式锁，我们来看一下分布式事务。 昨天说的那个多办公室公用一个厕所的场景，如厕完成包括了走出自己的办公室，进入厕所所在办公室，走入厕所锁门，如厕，走出厕所回自己办公室这几个步骤，这是一个如厕闭环。任何一个环节产生了意外，都不会如厕成功。可见分布式事务是由多个本地事务构成的。 事务的特征，是用过数据库都听说过的ACID,那什么是ACID呢，A原子性是说要么成功，要么失败回滚。C一致性是说的状态一致，比如转账前后一定仍然还能对得上账。I隔离性说的是事务与事务没有干扰。D持久性谈的是即使宕机也能保持事务一致。 这里的C指的是强一致性，也就是所有操作成功才能提交。这个对分布式就不太友好，胖子瘦子同时上厕所就是有可能胖子瘦子都 通过但是有一个人无法通畅，这就出现了BASE，保证最终一致。 这里介绍了三种分布式事务方式。： 一是XA协议的二阶段提交。分离了事务与资源。其中事务管理器管理着所有本地资源的提交与回滚，资源管理器就是事务参与者，通常由数据库厂商来实现XA。叫号员负责管理下发各个屋的事务和收集事务结果，决定是否提交。但存在同步阻塞(比如由于某节点没发送给协调者而产生阻塞)、单点故障(协调者)、数据不一致(还是某进程阻塞所导致的) 二是三阶段提交，增加了超时机制消灭阻塞，增加了准备阶段，通过预提交消灭不一致问题。 三是基于消息，好处在于不锁定资源，其实就是将事务异步了。 分布式事务的一致性是实现分布式事务的关键问题，目前来看还没有一种很简单、完美的方案可以应对所有场景。

  2019-12-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4e/90/a8d19e7b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张理查

  二是三阶段提交，增加了超时机制消灭阻塞，增加了准备阶段，通过预提交消灭不一致问题。 三是基于消息，好处在于不锁定资源，其实就是将事务异步了。 分布式事务的一致性是实现分布式事务的关键问题，目前来看还没有一种很简单、完美的方案可以应对所有场景。

  2019-12-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/23/f8/24fcccea.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  💢 星星💢

  我觉得互斥是保证事务一致性的前提，如果二个参与者都不互斥，怎么保证数据的一致性呢，肯定要有个排队等待的阶段。不然会出现数据混乱。。另外想听听老师的对这个问题的看法。

  2019-11-16

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKsicnqXeDnGYk55iaR1oBU02lo7EA4J12Mv5Hc2VMSwpok7zgTwzXK7HAHUqibzS7aYM4BJPcHfePvQ/132)

  白杨付

  如果2pc 参与者也引入超时机制呢？那么这种情况下，3pc的优势是什么？

  2019-10-29

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/20/b7/bdb3bcf0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Eternal

  我有一个疑问:3pc的第二个阶段，预提交的时候，将undo，redo日志记录，这个是怎么做到的，本地事务只能提交和回滚两个操作，只提交一半这个操作是业务系统自己通过数据库存undo,redo操作来模拟的吗？也就是最终提交的时候才是真的提交数据库本地事务。

  2019-10-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/d0/7f/1ad28cd3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王博

  三阶段提交一定能保证数据一致吗，假设三台机器A,B,C.PreCommit时A挂了，此时BC应该用undo log回滚，假设B在收到协调者回滚消息之前挂了会怎么样

  2019-10-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/72/03/da1fcc81.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  overland

  如果开发这个2pc或者3pc，是单独部署吗，和业务系统怎么对接呢，rpc接口吗，也要保障他的高可用吧？

  2019-10-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/71/4c/2cefec07.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  静水流深

  谢谢老师分享，解决了我困惑已久的问题！

  2019-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fb/a8/97eb38ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fierys

  现在我们项目计划使用Seata,正在研究和对比

  2019-10-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c7/ba/4c449be2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zhaozp

  打卡文章学习： 1、分布式事务就是分布式系统中运行的事务，由多个本地事务组合而成，需要保证事务的的基本特征ACID。 2、实现分布式事务的3种基本方法：2pc，3pc和基于消息的最终一致性方法。 3、2pc分为投票和提交两个阶段，3pc引入了超时机制和准备阶段。

  2019-10-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/82/07/5940b400.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  SaimSaim

  文中说基于XA协议的二阶段提交一开始协调者发送CanCommit给参与者,参与者看执行事务操作成不成功返回Yes or No 三阶段提交说PreCommit阶段才开始执行事务操作,那三阶段协调者发送CanCommit给参与者的时候,参与者根据什么返回Yes or No???

  2019-10-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c6/73/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  疯琴

  请问老师一个问题：两阶段提交中，第一阶段订单系统只是准备好增加一条关于用户 A 购买 100 件 T 恤的信息并锁库，但是没有实际的数据操作，那么在第二阶段不是直接解锁就好了么，何来的数据退回呢？

  作者回复: 因为在第一阶段的日志系统这些会记录信息的，只是没有提交修改具体的数据，当失败的时候，日志系统记录的相关信息也需要进行处理，回退到之前版本的

  2019-10-12

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/3b/47/f6c772a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jackey

  想起了没学分布式事务之前被数据一致性支配的恐惧…另外老师可以推荐一些比较优秀的分布式事务解决方案吗

  2019-10-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/bc/25/1c92a90c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tt

  1、老师，因为不能再直接恢复您的留言了，所以接着上次的留言再请教一下。 我上次留言中所说的三个节点最终是属于三个不同法律实体的，我也不知道这个算不算是分布式系统，但想到区块链中的节点也是属于互相没有关系的实体，似乎觉得应该可以算是分布式系统。 2、突然又觉得我说的那个三节点的系统所做的事其实也可以理解成为一个达成分布式共识的过程。就是参与业务的三方其实是就一个工作流的状态达成一个共识，比如一笔融资完全符合各方的要求。 进一步，是不是分布式共识是一个超大规模节点实现分布式事物的手段，达成共识且被各方记录下来就算分布式事物完成了，即分布式共识+分布式持久化=分布式事物，此时分布式锁的角色由工作量证明之类的算法代替？

  2019-10-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/2f/ed/7d7825c4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Kean

  干货满满

  2019-10-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5b/0d/597cfa28.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  田奇

  我想知道，这里描述的任意一个通讯过程，比如canCommit挂了，tcp断开了，重置了或者超时了等等，那么这个2pc，3pc以及分布式消息如何处理的，第一种是重试并设置超时，超时后就认为通讯失败，那么整个事务失败，但是会不会出现中间状态，因为一次事务涉及n多次tcp通讯调用，中间某次tcp挂了，但是以前的步骤可能已经执行或提交了，此时如何保证一致性？老师能不能描述更细节的，因为这里只是把2pc的大致过程描述了，但总感觉有问题

  2019-10-08

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f8/49/e292f240.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  boglond

  CAP理论老师没有讲一讲。

  作者回复: CAP会在后面第五站分布式数据存储中进行讲解

  2019-10-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/bc/25/1c92a90c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tt

  老师，分布式事物，可以用于工作流系统么？ 比如工作流上总体有3个节点，A,B,C，每个节点的操作均分为经办和复合操作，当作业顺序经历A→B→C后，一个事务才算完成。 中间任意一个环节都可能发生回退，如C认为条件不成立而将作业退回给B。 最后，每个环节经历的事件可能都很长，已天为度量单位。 我想请教一下老师，这样的系统还算是分布式系统么？如果不是，那该如何借鉴分布式事物的原理进行设计呢？

  作者回复: 你说的工作流系统中的作业顺序可以设计为分布式事务的。分布式事务，就是在分布式系统中运行的事务，由多个本地事务组合而成。重点在于你的工作流系统是否运行在分布式环境中，该系统本身是否为分布式系统。

  2019-10-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/44/a7/171c1e86.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  啦啦啦

  第一阶段：订单系统中将与用户 A 有关的订单数据库锁住，准备好增加一条关于用户 A 购买 100 件 T 恤的信息，并将同意消息“Yes”回复给协调者。老师这个地方是没有进行插入操作的吧？应该只是锁住了相关数据的吧

  2019-10-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/17/27/ec30d30a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jxin

  1.事务是实现原子性操作的手段，互斥是保障一个线程唯一持有一个竞量的手段。 2.事务的实现需要互斥，互斥的存在不一定是为了实现事务。

  2019-10-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/1e/f9/6d61c58d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geekki

  分布式事务在访问临界资源时会用到分布式互斥算法？

  作者回复: 存在多个进程同时需要访问临界资源的情况下，会用到分布式互斥算法。

  2019-10-04

  **

  **
