# 18 \| PartitionStateMachine：分区状态转换如何实现？

作者: 胡夕

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/cd/bc/cd026ad8019edaa79e3c149fcbbfe7bc.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/66/ac/66bbea807c71235125288f2faf46beac.mp3" type="audio/mpeg"></audio>

你好，我是胡夕。今天我们进入到分区状态机（PartitionStateMachine）源码的学习。

PartitionStateMachine 负责管理 Kafka 分区状态的转换，和 ReplicaStateMachine 是一脉相承的。从代码结构、实现功能和设计原理来看，二者都极为相似。上节课我们已经学过了 ReplicaStateMachine，相信你在学习这节课的 PartitionStateMachine 时，会轻松很多。

在面试的时候，很多面试官都非常喜欢问 Leader 选举的策略。学完了今天的课程之后，你不但能够说出 4 种 Leader 选举的场景，还能总结出它们的共性。对于面试来说，绝对是个加分项！

话不多说，我们就正式开始吧。

## PartitionStateMachine简介

PartitionStateMachine.scala 文件位于 controller 包下，代码结构不复杂，可以看下这张思维导图：

![](<https://static001.geekbang.org/resource/image/b3/5e/b3f286586b39b9910e756c3539a1125e.jpg?wh=2250*1467>)

代码总共有 5 大部分。

- **PartitionStateMachine**：分区状态机抽象类。它定义了诸如 startup、shutdown 这样的公共方法，同时也给出了处理分区状态转换入口方法 handleStateChanges 的签名。
- **ZkPartitionStateMachine**：PartitionStateMachine 唯一的继承子类。它实现了分区状态机的主体逻辑功能。和 ZkReplicaStateMachine 类似，ZkPartitionStateMachine 重写了父类的 handleStateChanges 方法，并配以私有的 doHandleStateChanges 方法，共同实现分区状态转换的操作。
- **PartitionState接口及其实现对象**：定义 4 类分区状态，分别是 NewPartition、OnlinePartition、OfflinePartition 和 NonExistentPartition。除此之外，还定义了它们之间的流转关系。
- **PartitionLeaderElectionStrategy接口及其实现对象**：定义 4 类分区 Leader 选举策略。你可以认为它们是发生 Leader 选举的 4 种场景。
- **PartitionLeaderElectionAlgorithms**：分区 Leader 选举的算法实现。既然定义了 4 类选举策略，就一定有相应的实现代码，PartitionLeaderElectionAlgorithms 就提供了这 4 类选举策略的实现代码。

<!-- -->

<!-- [[[read_end]]] -->

## 类定义与字段

PartitionStateMachine 和 ReplicaStateMachine 非常类似，我们先看下面这两段代码：

```
// PartitionStateMachine抽象类定义
abstract class PartitionStateMachine(
  controllerContext: ControllerContext) extends Logging {
  ......
}
// ZkPartitionStateMachine继承子类定义
class ZkPartitionStateMachine(config: KafkaConfig,
	stateChangeLogger: StateChangeLogger,
	controllerContext: ControllerContext,
	zkClient: KafkaZkClient,
	controllerBrokerRequestBatch: ControllerBrokerRequestBatch) extends PartitionStateMachine(controllerContext) {
  // Controller所在Broker的Id
  private val controllerId = config.brokerId
  ......
}
```

从代码中，可以发现，它们的类定义一模一样，尤其是 ZkPartitionStateMachine 和 ZKReplicaStateMachine，它们接收的字段列表都是相同的。此刻，你应该可以体会到它们要做的处理逻辑，其实也是差不多的。

同理，ZkPartitionStateMachine 实例的创建和启动时机也跟 ZkReplicaStateMachine 的完全相同，即：每个 Broker 进程启动时，会在创建 KafkaController 对象的过程中，生成 ZkPartitionStateMachine 实例，而只有 Controller 组件所在的 Broker，才会启动分区状态机。

下面这段代码展示了 ZkPartitionStateMachine 实例创建和启动的位置：

```
class KafkaController(......) {
  ......
  // 在KafkaController对象创建过程中，生成ZkPartitionStateMachine实例
  val partitionStateMachine: PartitionStateMachine = 
    new ZkPartitionStateMachine(config, stateChangeLogger, 
      controllerContext, zkClient, 
      new ControllerBrokerRequestBatch(config, 
      controllerChannelManager, eventManager, controllerContext, 
      stateChangeLogger))
	......
    private def onControllerFailover(): Unit = {
	......
	replicaStateMachine.startup() // 启动副本状态机
    partitionStateMachine.startup() // 启动分区状态机
    ......
    }
}
```

有句话我要再强调一遍：**每个Broker启动时，都会创建对应的分区状态机和副本状态机实例，但只有Controller所在的Broker才会启动它们**。如果 Controller 变更到其他 Broker，老 Controller 所在的 Broker 要调用这些状态机的 shutdown 方法关闭它们，新 Controller 所在的 Broker 调用状态机的 startup 方法启动它们。

## 分区状态

既然 ZkPartitionStateMachine 是管理分区状态转换的，那么，我们至少要知道分区都有哪些状态，以及 Kafka 规定的转换规则是什么。这就是 PartitionState 接口及其实现对象做的事情。和 ReplicaState 类似，PartitionState 定义了分区的状态空间以及流转规则。

我以 OnlinePartition 状态为例，说明下代码是如何实现流转的：

```
sealed trait PartitionState {
  def state: Byte // 状态序号，无实际用途
  def validPreviousStates: Set[PartitionState] // 合法前置状态集合
}

case object OnlinePartition extends PartitionState {
  val state: Byte = 1
  val validPreviousStates: Set[PartitionState] = Set(NewPartition, OnlinePartition, OfflinePartition)
}
```

如代码所示，每个 PartitionState 都定义了名为 validPreviousStates 的集合，也就是每个状态对应的合法前置状态集。

对于 OnlinePartition 而言，它的合法前置状态集包括 NewPartition、OnlinePartition 和 OfflinePartition。在 Kafka 中，从合法状态集以外的状态向目标状态进行转换，将被视为非法操作。

目前，Kafka 为分区定义了 4 类状态。

- NewPartition：分区被创建后被设置成这个状态，表明它是一个全新的分区对象。处于这个状态的分区，被 Kafka 认为是“未初始化”，因此，不能选举 Leader。
- OnlinePartition：分区正式提供服务时所处的状态。
- OfflinePartition：分区下线后所处的状态。
- NonExistentPartition：分区被删除，并且从分区状态机移除后所处的状态。

<!-- -->

下图展示了完整的分区状态转换规则：

![](<https://static001.geekbang.org/resource/image/5d/b7/5dc219ace96f3a493f42ca658410f2b7.jpg?wh=2975*1408>)

图中的双向箭头连接的两个状态可以彼此进行转换，如 OnlinePartition 和 OfflinePartition。Kafka 允许一个分区从 OnlinePartition 切换到 OfflinePartition，反之亦然。

另外，OnlinePartition 和 OfflinePartition 都有一根箭头指向自己，这表明 OnlinePartition 切换到 OnlinePartition 的操作是允许的。**当分区Leader选举发生的时候，就可能出现这种情况。接下来，我们就聊聊分区Leader选举那些事儿**。

## 分区Leader选举的场景及方法

刚刚我们说了两个状态机的相同点，接下来，我们要学习的分区 Leader 选举，可以说是 PartitionStateMachine 特有的功能了。

每个分区都必须选举出 Leader 才能正常提供服务，因此，对于分区而言，Leader 副本是非常重要的角色。既然这样，我们就必须要了解 Leader 选举什么流程，以及在代码中是如何实现的。我们重点学习下选举策略以及具体的实现方法代码。

### PartitionLeaderElectionStrategy

先明确下分区 Leader 选举的含义，其实很简单，就是为 Kafka 主题的某个分区推选 Leader 副本。

那么，Kafka 定义了哪几种推选策略，或者说，在什么情况下需要执行 Leader 选举呢？

这就是 PartitionLeaderElectionStrategy 接口要做的事情，请看下面的代码：

```
// 分区Leader选举策略接口
sealed trait PartitionLeaderElectionStrategy
// 离线分区Leader选举策略
final case class OfflinePartitionLeaderElectionStrategy(
  allowUnclean: Boolean) extends PartitionLeaderElectionStrategy
// 分区副本重分配Leader选举策略  
final case object ReassignPartitionLeaderElectionStrategy 
  extends PartitionLeaderElectionStrategy
// 分区Preferred副本Leader选举策略
final case object PreferredReplicaPartitionLeaderElectionStrategy 
  extends PartitionLeaderElectionStrategy
// Broker Controlled关闭时Leader选举策略
final case object ControlledShutdownPartitionLeaderElectionStrategy 
  extends PartitionLeaderElectionStrategy
```

当前，分区 Leader 选举有 4 类场景。

- OfflinePartitionLeaderElectionStrategy：因为 Leader 副本下线而引发的分区 Leader 选举。
- ReassignPartitionLeaderElectionStrategy：因为执行分区副本重分配操作而引发的分区 Leader 选举。
- PreferredReplicaPartitionLeaderElectionStrategy：因为执行 Preferred 副本 Leader 选举而引发的分区 Leader 选举。
- ControlledShutdownPartitionLeaderElectionStrategy：因为正常关闭 Broker 而引发的分区 Leader 选举。

<!-- -->

### PartitionLeaderElectionAlgorithms

针对这 4 类场景，分区状态机的 PartitionLeaderElectionAlgorithms 对象定义了 4 个方法，分别负责为每种场景选举 Leader 副本，这 4 种方法是：

- offlinePartitionLeaderElection；
- reassignPartitionLeaderElection；
- preferredReplicaPartitionLeaderElection；
- controlledShutdownPartitionLeaderElection。

<!-- -->

offlinePartitionLeaderElection 方法的逻辑是这 4 个方法中最复杂的，我们就先从它开始学起。

```
def offlinePartitionLeaderElection(assignment: Seq[Int], 
  isr: Seq[Int], liveReplicas: Set[Int], 
  uncleanLeaderElectionEnabled: Boolean, controllerContext: ControllerContext): Option[Int] = {
  // 从当前分区副本列表中寻找首个处于存活状态的ISR副本
  assignment.find(id => liveReplicas.contains(id) && isr.contains(id)).orElse {
    // 如果找不到满足条件的副本，查看是否允许Unclean Leader选举
    // 即Broker端参数unclean.leader.election.enable是否等于true
    if (uncleanLeaderElectionEnabled) {
      // 选择当前副本列表中的第一个存活副本作为Leader
      val leaderOpt = assignment.find(liveReplicas.contains)
      if (leaderOpt.isDefined)
        controllerContext.stats.uncleanLeaderElectionRate.mark()
      leaderOpt
    } else {
      None // 如果不允许Unclean Leader选举，则返回None表示无法选举Leader
    }
  }
}
```

我再画一张流程图，帮助你理解它的代码逻辑：

![](<https://static001.geekbang.org/resource/image/28/7c/2856c2a7c6dd1818dbdf458c4e409d7c.jpg?wh=3152*3129>)

这个方法总共接收 5 个参数。除了你已经很熟悉的 ControllerContext 类，其他 4 个非常值得我们花一些时间去探究下。

**1\.assignments**

这是分区的副本列表。该列表有个专属的名称，叫 Assigned Replicas，简称 AR。当我们创建主题之后，使用 kafka-topics 脚本查看主题时，应该可以看到名为 Replicas 的一列数据。这列数据显示的，就是主题下每个分区的 AR。assignments 参数类型是 Seq[Int]。**这揭示了一个重要的事实：AR是有顺序的，而且不一定和ISR的顺序相同！**

**2\.isr**

ISR 在 Kafka 中很有名气，它保存了分区所有与 Leader 副本保持同步的副本列表。注意，Leader 副本自己也在 ISR 中。另外，作为 Seq[Int]类型的变量，isr 自身也是有顺序的。

**3\.liveReplicas**

从名字可以推断出，它保存了该分区下所有处于存活状态的副本。怎么判断副本是否存活呢？可以根据 Controller 元数据缓存中的数据来判定。简单来说，所有在运行中的 Broker 上的副本，都被认为是存活的。

**4\.uncleanLeaderElectionEnabled**

在默认配置下，只要不是由 AdminClient 发起的 Leader 选举，这个参数的值一般是 false，即 Kafka 不允许执行 Unclean Leader 选举。所谓的 Unclean Leader 选举，是指在 ISR 列表为空的情况下，Kafka 选择一个非 ISR 副本作为新的 Leader。由于存在丢失数据的风险，目前，社区已经通过把 Broker 端参数 unclean.leader.election.enable 的默认值设置为 false 的方式，禁止 Unclean Leader 选举了。

值得一提的是，社区于 2.4.0.0 版本正式支持在 AdminClient 端为给定分区选举 Leader。目前的设计是，如果 Leader 选举是由 AdminClient 端触发的，那就默认开启 Unclean Leader 选举。不过，在学习 offlinePartitionLeaderElection 方法时，你可以认为 uncleanLeaderElectionEnabled=false，这并不会影响你对该方法的理解。

了解了这几个参数的含义，我们就可以研究具体的流程了。

代码首先会顺序搜索 AR 列表，并把第一个同时满足以下两个条件的副本作为新的 Leader 返回：

- 该副本是存活状态，即副本所在的 Broker 依然在运行中；
- 该副本在 ISR 列表中。

<!-- -->

倘若无法找到这样的副本，代码会检查是否开启了 Unclean Leader 选举：如果开启了，则降低标准，只要满足上面第一个条件即可；如果未开启，则本次 Leader 选举失败，没有新 Leader 被选出。

其他 3 个方法要简单得多，我们直接看代码：

```
def reassignPartitionLeaderElection(reassignment: Seq[Int], isr: Seq[Int], liveReplicas: Set[Int]): Option[Int] = {
  reassignment.find(id => liveReplicas.contains(id) && isr.contains(id))
}

def preferredReplicaPartitionLeaderElection(assignment: Seq[Int], isr: Seq[Int], liveReplicas: Set[Int]): Option[Int] = {
  assignment.headOption.filter(id => liveReplicas.contains(id) && isr.contains(id))
}

def controlledShutdownPartitionLeaderElection(assignment: Seq[Int], isr: Seq[Int], liveReplicas: Set[Int], shuttingDownBrokers: Set[Int]): Option[Int] = {
  assignment.find(id => liveReplicas.contains(id) && isr.contains(id) && !shuttingDownBrokers.contains(id))
}
```

可以看到，它们的逻辑几乎是相同的，大概的原理都是从 AR，或给定的副本列表中寻找存活状态的 ISR 副本。

讲到这里，你应该已经知道 Kafka 为分区选举 Leader 的大体思路了。**基本上就是，找出AR列表（或给定副本列表）中首个处于存活状态，且在ISR列表的副本，将其作为新Leader。**

## 处理分区状态转换的方法

掌握了刚刚的这些知识之后，现在，我们正式来看 PartitionStateMachine 的工作原理。

### handleStateChanges

前面我提到过，handleStateChanges 是入口方法，所以我们先看它的方法签名：

```
def handleStateChanges(
  partitions: Seq[TopicPartition],
  targetState: PartitionState, 
  leaderElectionStrategy: Option[PartitionLeaderElectionStrategy]): 
  Map[TopicPartition, Either[Throwable, LeaderAndIsr]]
```

如果用一句话概括 handleStateChanges 的作用，应该这样说：**handleStateChanges把partitions的状态设置为targetState，同时，还可能需要用leaderElectionStrategy策略为partitions选举新的Leader，最终将partitions的Leader信息返回。**

其中，partitions 是待执行状态变更的目标分区列表，targetState 是目标状态，leaderElectionStrategy 是一个可选项，如果传入了，就表示要执行 Leader 选举。

下面是 handleStateChanges 方法的完整代码，我以注释的方式给出了主要的功能说明：

```
override def handleStateChanges(
    partitions: Seq[TopicPartition],
    targetState: PartitionState,
    partitionLeaderElectionStrategyOpt: Option[PartitionLeaderElectionStrategy]
  ): Map[TopicPartition, Either[Throwable, LeaderAndIsr]] = {
    if (partitions.nonEmpty) {
      try {
        // 清空Controller待发送请求集合，准备本次请求发送
        controllerBrokerRequestBatch.newBatch()
        // 调用doHandleStateChanges方法执行真正的状态变更逻辑
        val result = doHandleStateChanges(
          partitions,
          targetState,
          partitionLeaderElectionStrategyOpt
        )
        // Controller给相关Broker发送请求通知状态变化
        controllerBrokerRequestBatch.sendRequestsToBrokers(
          controllerContext.epoch)
        // 返回状态变更处理结果
        result
      } catch {
        // 如果Controller易主，则记录错误日志，然后重新抛出异常
        // 上层代码会捕获该异常并执行maybeResign方法执行卸任逻辑
        case e: ControllerMovedException =>
          error(s"Controller moved to another broker when moving some partitions to $targetState state", e)
          throw e
        // 如果是其他异常，记录错误日志，封装错误返回
        case e: Throwable =>
          error(s"Error while moving some partitions to $targetState state", e)
          partitions.iterator.map(_ -> Left(e)).toMap
      }
    } else { // 如果partitions为空，什么都不用做
      Map.empty
    }
  }
```

整个方法就两步：第 1 步是，调用 doHandleStateChanges 方法执行分区状态转换；第 2 步是，Controller 给相关 Broker 发送请求，告知它们这些分区的状态变更。至于哪些 Broker 属于相关 Broker，以及给 Broker 发送哪些请求，实际上是在第 1 步中被确认的。

当然，这个方法的重点，就是第 1 步中调用的 doHandleStateChanges 方法。

### doHandleStateChanges

先来看这个方法的实现：

```
private def doHandleStateChanges(
    partitions: Seq[TopicPartition],
    targetState: PartitionState,
    partitionLeaderElectionStrategyOpt: Option[PartitionLeaderElectionStrategy]
  ): Map[TopicPartition, Either[Throwable, LeaderAndIsr]] = {
    val stateChangeLog = stateChangeLogger.withControllerEpoch(controllerContext.epoch)
    val traceEnabled = stateChangeLog.isTraceEnabled
    // 初始化新分区的状态为NonExistentPartition
    partitions.foreach(partition => controllerContext.putPartitionStateIfNotExists(partition, NonExistentPartition))
    // 找出要执行非法状态转换的分区，记录错误日志
    val (validPartitions, invalidPartitions) = controllerContext.checkValidPartitionStateChange(partitions, targetState)
    invalidPartitions.foreach(partition => logInvalidTransition(partition, targetState))
    // 根据targetState进入到不同的case分支
    targetState match {
    	......
    }
}
```

这个方法首先会做状态初始化的工作，具体来说就是，在方法调用时，不在元数据缓存中的所有分区的状态，会被初始化为 NonExistentPartition。

接着，检查哪些分区执行的状态转换不合法，并为这些分区记录相应的错误日志。

之后，代码携合法状态转换的分区列表进入到 case 分支。由于分区状态只有 4 个，因此，它的 case 分支代码远比 ReplicaStateMachine 中的简单，而且，只有 OnlinePartition 这一路的分支逻辑相对复杂，其他 3 路仅仅是将分区状态设置成目标状态而已，

所以，我们来深入研究下目标状态是 OnlinePartition 的分支：

```
case OnlinePartition =>
  // 获取未初始化分区列表，也就是NewPartition状态下的所有分区
  val uninitializedPartitions = validPartitions.filter(
    partition => partitionState(partition) == NewPartition)
  // 获取具备Leader选举资格的分区列表
  // 只能为OnlinePartition和OfflinePartition状态的分区选举Leader 
  val partitionsToElectLeader = validPartitions.filter(
    partition => partitionState(partition) == OfflinePartition ||
     partitionState(partition) == OnlinePartition)
  // 初始化NewPartition状态分区，在ZooKeeper中写入Leader和ISR数据
  if (uninitializedPartitions.nonEmpty) {
    val successfulInitializations = initializeLeaderAndIsrForPartitions(uninitializedPartitions)
    successfulInitializations.foreach { partition =>
      stateChangeLog.info(s"Changed partition $partition from ${partitionState(partition)} to $targetState with state " +
        s"${controllerContext.partitionLeadershipInfo(partition).leaderAndIsr}")
      controllerContext.putPartitionState(partition, OnlinePartition)
    }
  }
  // 为具备Leader选举资格的分区推选Leader
  if (partitionsToElectLeader.nonEmpty) {
    val electionResults = electLeaderForPartitions(
      partitionsToElectLeader,
      partitionLeaderElectionStrategyOpt.getOrElse(
        throw new IllegalArgumentException("Election strategy is a required field when the target state is OnlinePartition")
      )
    )
    electionResults.foreach {
      case (partition, Right(leaderAndIsr)) =>
        stateChangeLog.info(
          s"Changed partition $partition from ${partitionState(partition)} to $targetState with state $leaderAndIsr"
        )
        // 将成功选举Leader后的分区设置成OnlinePartition状态
        controllerContext.putPartitionState(
          partition, OnlinePartition)
      case (_, Left(_)) => // 如果选举失败，忽略之
    }
    // 返回Leader选举结果
    electionResults
  } else {
    Map.empty
  }
```

虽然代码有点长，但总的步骤就两步。

**第1步**是为 NewPartition 状态的分区做初始化操作，也就是在 ZooKeeper 中，创建并写入分区节点数据。节点的位置是`/brokers/topics/&lt;topic&gt;/partitions/&lt;partition&gt;`，每个节点都要包含分区的 Leader 和 ISR 等数据。而**Leader和ISR的确定规则是：选择存活副本列表的第一个副本作为Leader；选择存活副本列表作为ISR**。至于具体的代码，可以看下 initializeLeaderAndIsrForPartitions 方法代码片段的倒数第 5 行：

```
private def initializeLeaderAndIsrForPartitions(partitions: Seq[TopicPartition]): Seq[TopicPartition] = {
	......
    // 获取每个分区的副本列表
    val replicasPerPartition = partitions.map(partition => partition -> controllerContext.partitionReplicaAssignment(partition))
    // 获取每个分区的所有存活副本
    val liveReplicasPerPartition = replicasPerPartition.map { case (partition, replicas) =>
        val liveReplicasForPartition = replicas.filter(replica => controllerContext.isReplicaOnline(replica, partition))
        partition -> liveReplicasForPartition
    }
    // 按照有无存活副本对分区进行分组
    // 分为两组：有存活副本的分区；无任何存活副本的分区
    val (partitionsWithoutLiveReplicas, partitionsWithLiveReplicas) = liveReplicasPerPartition.partition { case (_, liveReplicas) => liveReplicas.isEmpty }
    ......
    // 为"有存活副本的分区"确定Leader和ISR
    // Leader确认依据：存活副本列表的首个副本被认定为Leader
    // ISR确认依据：存活副本列表被认定为ISR
    val leaderIsrAndControllerEpochs = partitionsWithLiveReplicas.map { case (partition, liveReplicas) =>
      val leaderAndIsr = LeaderAndIsr(liveReplicas.head, liveReplicas.toList)
      ......
    }.toMap
    ......
}
```

**第2步**是为具备 Leader 选举资格的分区推选 Leader，代码调用 electLeaderForPartitions 方法实现。这个方法会不断尝试为多个分区选举 Leader，直到所有分区都成功选出 Leader。

选举 Leader 的核心代码位于 doElectLeaderForPartitions 方法中，该方法主要有 3 步。

代码很长，我先画一张图来展示它的主要步骤，然后再分步骤给你解释每一步的代码，以免你直接陷入冗长的源码行里面去。

![](<https://static001.geekbang.org/resource/image/ee/ad/ee017b731c54ee6667e3759c1a6b66ad.jpg?wh=2428*5780>)

看着好像图也很长，别着急，我们来一步步拆解下。

就像前面说的，这个方法大体分为 3 步。第 1 步是从 ZooKeeper 中获取给定分区的 Leader、ISR 信息，并将结果封装进名为 validLeaderAndIsrs 的容器中，代码如下：

```
// doElectLeaderForPartitions方法的第1部分
val getDataResponses = try {
  // 批量获取ZooKeeper中给定分区的znode节点数据
  zkClient.getTopicPartitionStatesRaw(partitions)
} catch {
  case e: Exception =>
    return (partitions.iterator.map(_ -> Left(e)).toMap, Seq.empty)
}
// 构建两个容器，分别保存可选举Leader分区列表和选举失败分区列表
val failedElections = mutable.Map.empty[TopicPartition, Either[Exception, LeaderAndIsr]]
val validLeaderAndIsrs = mutable.Buffer.empty[(TopicPartition, LeaderAndIsr)]
// 遍历每个分区的znode节点数据
getDataResponses.foreach { getDataResponse =>
  val partition = getDataResponse.ctx.get.asInstanceOf[TopicPartition]
  val currState = partitionState(partition)
  // 如果成功拿到znode节点数据
  if (getDataResponse.resultCode == Code.OK) {
    TopicPartitionStateZNode.decode(getDataResponse.data, getDataResponse.stat) match {
      // 节点数据中含Leader和ISR信息
      case Some(leaderIsrAndControllerEpoch) =>
        // 如果节点数据的Controller Epoch值大于当前Controller Epoch值
        if (leaderIsrAndControllerEpoch.controllerEpoch > controllerContext.epoch) {
          val failMsg = s"Aborted leader election for partition $partition since the LeaderAndIsr path was " +
            s"already written by another controller. This probably means that the current controller $controllerId went through " +
            s"a soft failure and another controller was elected with epoch ${leaderIsrAndControllerEpoch.controllerEpoch}."
          // 将该分区加入到选举失败分区列表
          failedElections.put(partition, Left(new StateChangeFailedException(failMsg)))
        } else {
          // 将该分区加入到可选举Leader分区列表 
          validLeaderAndIsrs += partition -> leaderIsrAndControllerEpoch.leaderAndIsr
        }
      // 如果节点数据不含Leader和ISR信息
      case None =>
        val exception = new StateChangeFailedException(s"LeaderAndIsr information doesn't exist for partition $partition in $currState state")
        // 将该分区加入到选举失败分区列表
        failedElections.put(partition, Left(exception))
    }
  // 如果没有拿到znode节点数据，则将该分区加入到选举失败分区列表
  } else if (getDataResponse.resultCode == Code.NONODE) {
    val exception = new StateChangeFailedException(s"LeaderAndIsr information doesn't exist for partition $partition in $currState state")
    failedElections.put(partition, Left(exception))
  } else {
    failedElections.put(partition, Left(getDataResponse.resultException.get))
  }
}

if (validLeaderAndIsrs.isEmpty) {
  return (failedElections.toMap, Seq.empty)
}
```

首先，代码会批量读取 ZooKeeper 中给定分区的所有 Znode 数据。之后，会构建两个容器，分别保存可选举 Leader 分区列表和选举失败分区列表。接着，开始遍历每个分区的 Znode 节点数据，如果成功拿到 Znode 节点数据，节点数据包含 Leader 和 ISR 信息且节点数据的 Controller Epoch 值小于当前 Controller Epoch 值，那么，就将该分区加入到可选举 Leader 分区列表。倘若发现 Zookeeper 中保存的 Controller Epoch 值大于当前 Epoch 值，说明该分区已经被一个更新的 Controller 选举过 Leader 了，此时必须终止本次 Leader 选举，并将该分区放置到选举失败分区列表中。

遍历完这些分区之后，代码要看下 validLeaderAndIsrs 容器中是否包含可选举 Leader 的分区。如果一个满足选举 Leader 的分区都没有，方法直接返回。至此，doElectLeaderForPartitions 方法的第一大步完成。

下面，我们看下该方法的第 2 部分代码：

```
// doElectLeaderForPartitions方法的第2部分
// 开始选举Leader，并根据有无Leader将分区进行分区
val (partitionsWithoutLeaders, partitionsWithLeaders) = 
  partitionLeaderElectionStrategy match {
  case OfflinePartitionLeaderElectionStrategy(allowUnclean) =>
    val partitionsWithUncleanLeaderElectionState = collectUncleanLeaderElectionState(
      validLeaderAndIsrs,
      allowUnclean
    )
    // 为OffinePartition分区选举Leader
    leaderForOffline(controllerContext, partitionsWithUncleanLeaderElectionState).partition(_.leaderAndIsr.isEmpty)
  case ReassignPartitionLeaderElectionStrategy =>
    // 为副本重分配的分区选举Leader
    leaderForReassign(controllerContext, validLeaderAndIsrs).partition(_.leaderAndIsr.isEmpty)
  case PreferredReplicaPartitionLeaderElectionStrategy =>
    // 为分区执行Preferred副本Leader选举
    leaderForPreferredReplica(controllerContext, validLeaderAndIsrs).partition(_.leaderAndIsr.isEmpty)
  case ControlledShutdownPartitionLeaderElectionStrategy =>
    // 为因Broker正常关闭而受影响的分区选举Leader
    leaderForControlledShutdown(controllerContext, validLeaderAndIsrs).partition(_.leaderAndIsr.isEmpty)
}
```

这一步是根据给定的 PartitionLeaderElectionStrategy，调用 PartitionLeaderElectionAlgorithms 的不同方法执行 Leader 选举，同时，区分出成功选举 Leader 和未选出 Leader 的分区。

前面说过了，这 4 种不同的策略定义了 4 个专属的方法来进行 Leader 选举。其实，如果你打开这些方法的源码，就会发现它们大同小异。基本上，选择 Leader 的规则，就是选择副本集合中首个存活且处于 ISR 中的副本作为 Leader。

现在，我们再来看这个方法的最后一部分代码，这一步主要是更新 ZooKeeper 节点数据，以及 Controller 端元数据缓存信息。

```
// doElectLeaderForPartitions方法的第3部分
// 将所有选举失败的分区全部加入到Leader选举失败分区列表
partitionsWithoutLeaders.foreach { electionResult =>
  val partition = electionResult.topicPartition
  val failMsg = s"Failed to elect leader for partition $partition under strategy $partitionLeaderElectionStrategy"
  failedElections.put(partition, Left(new StateChangeFailedException(failMsg)))
}
val recipientsPerPartition = partitionsWithLeaders.map(result => result.topicPartition -> result.liveReplicas).toMap
val adjustedLeaderAndIsrs = partitionsWithLeaders.map(result => result.topicPartition -> result.leaderAndIsr.get).toMap
// 使用新选举的Leader和ISR信息更新ZooKeeper上分区的znode节点数据
val UpdateLeaderAndIsrResult(finishedUpdates, updatesToRetry) = zkClient.updateLeaderAndIsr(
  adjustedLeaderAndIsrs, controllerContext.epoch, controllerContext.epochZkVersion)
// 对于ZooKeeper znode节点数据更新成功的分区，封装对应的Leader和ISR信息
// 构建LeaderAndIsr请求，并将该请求加入到Controller待发送请求集合
// 等待后续统一发送
finishedUpdates.foreach { case (partition, result) =>
  result.foreach { leaderAndIsr =>
    val replicaAssignment = controllerContext.partitionFullReplicaAssignment(partition)
    val leaderIsrAndControllerEpoch = LeaderIsrAndControllerEpoch(leaderAndIsr, controllerContext.epoch)
    controllerContext.partitionLeadershipInfo.put(partition, leaderIsrAndControllerEpoch)
    controllerBrokerRequestBatch.addLeaderAndIsrRequestForBrokers(recipientsPerPartition(partition), partition,
      leaderIsrAndControllerEpoch, replicaAssignment, isNew = false)
  }
}
// 返回选举结果，包括成功选举并更新ZooKeeper节点的分区、选举失败分区以及
// ZooKeeper节点更新失败的分区
(finishedUpdates ++ failedElections, updatesToRetry)
```

首先，将上一步中所有选举失败的分区，全部加入到 Leader 选举失败分区列表。

然后，使用新选举的 Leader 和 ISR 信息，更新 ZooKeeper 上分区的 Znode 节点数据。对于 ZooKeeper Znode 节点数据更新成功的那些分区，源码会封装对应的 Leader 和 ISR 信息，构建 LeaderAndIsr 请求，并将该请求加入到 Controller 待发送请求集合，等待后续统一发送。

最后，方法返回选举结果，包括成功选举并更新 ZooKeeper 节点的分区列表、选举失败分区列表，以及 ZooKeeper 节点更新失败的分区列表。

这会儿，你还记得 handleStateChanges 方法的第 2 步是 Controller 给相关的 Broker 发送请求吗？那么，到底要给哪些 Broker 发送哪些请求呢？其实就是在上面这步完成的，即这行语句：

```
controllerBrokerRequestBatch.addLeaderAndIsrRequestForBrokers(
  recipientsPerPartition(partition), partition,
  leaderIsrAndControllerEpoch, replicaAssignment, isNew = false)
```

## 总结

今天，我们重点学习了 PartitionStateMachine.scala 文件的源码，主要是研究了 Kafka 分区状态机的构造原理和工作机制。

学到这里，我们再来回答开篇面试官的问题，应该就不是什么难事了。现在我们知道了，Kafka 目前提供 4 种 Leader 选举策略，分别是分区下线后的 Leader 选举、分区执行副本重分配时的 Leader 选举、分区执行 Preferred 副本 Leader 选举，以及 Broker 下线时的分区 Leader 选举。

这 4 类选举策略在选择 Leader 这件事情上有着类似的逻辑，那就是，它们几乎都是选择当前副本有序集合中的、首个处于 ISR 集合中的存活副本作为新的 Leader。当然，个别选举策略可能会有细小的差别，你可以结合我们今天学到的源码，课下再深入地研究一下每一类策略的源码。

我们来回顾下这节课的重点。

- PartitionStateMachine 是 Kafka Controller 端定义的分区状态机，负责定义、维护和管理合法的分区状态转换。
- 每个 Broker 启动时都会实例化一个分区状态机对象，但只有 Controller 所在的 Broker 才会启动它。
- Kafka 分区有 4 类状态，分别是 NewPartition、OnlinePartition、OfflinePartition 和 NonExistentPartition。其中 OnlinPartition 是分区正常工作时的状态。NewPartition 是未初始化状态，处于该状态下的分区尚不具备选举 Leader 的资格。
- Leader 选举有 4 类场景，分别是 Offline、Reassign、Preferrer Leader Election 和 ControlledShutdown。每类场景都对应于一种特定的 Leader 选举策略。
- handleStateChanges 方法是主要的入口方法，下面调用 doHandleStateChanges 私有方法实现实际的 Leader 选举功能。

<!-- -->

下个模块，我们将来到 Kafka 延迟操作代码的世界。在那里，你能了解 Kafka 是如何实现一个延迟请求的处理的。另外，一个 O(N)时间复杂度的时间轮算法也等候在那里，到时候我们一起研究下它！

## 课后讨论

源码中有个 triggerOnlineStateChangeForPartitions 方法，请你分析下，它是做什么用的，以及它何时被调用？

欢迎你在留言区畅所欲言，跟我交流讨论，也欢迎你把今天的内容分享给你的朋友。

