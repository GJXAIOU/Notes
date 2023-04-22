# 12 \| ControllerChannelManager：Controller如何管理请求发送？

作者: 胡夕

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/9a/ef/9a28662042adcd4f9d0cd324974d3aef.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/27/03/2788d6cb2b61d63a9dd239be95b61d03.mp3" type="audio/mpeg"></audio>

你好，我是胡夕。上节课，我们深入研究了 ControllerContext.scala 源码文件，掌握了 Kafka 集群定义的重要元数据。今天，我们来学习下 Controller 是如何给其他 Broker 发送请求的。

掌握了这部分实现原理，你就能更好地了解 Controller 究竟是如何与集群 Broker 进行交互，从而实现管理集群元数据的功能的。而且，阅读这部分源码，还能帮你定位和解决线上问题。我先跟你分享一个真实的案例。

当时还是在 Kafka 0.10.0.1 时代，我们突然发现，在线上环境中，很多元数据变更无法在集群的所有 Broker 上同步了。具体表现为，创建了主题后，有些 Broker 依然无法感知到。

我的第一感觉是 Controller 出现了问题，但又苦于无从排查和验证。后来，我想到，会不会是 Controller 端请求队列中积压的请求太多造成的呢？因为当时 Controller 所在的 Broker 本身承载着非常重的业务，这是非常有可能的原因。

在看了相关代码后，我们就在相应的源码中新加了一个监控指标，用于实时监控 Controller 的请求队列长度。当更新到生产环境后，我们很轻松地定位了问题。果然，由于 Controller 所在的 Broker 自身负载过大，导致 Controller 端的请求积压，从而造成了元数据更新的滞后。精准定位了问题之后，解决起来就很容易了。后来，社区于 0.11 版本正式引入了相关的监控指标。

<!-- [[[read_end]]] -->

你看，阅读源码，除了可以学习优秀开发人员编写的代码之外，我们还能根据自身的实际情况做定制化方案，实现一些非开箱即用的功能。

## Controller发送请求类型

下面，我们就正式进入到 Controller 请求发送管理部分的学习。你可能会问：“Controller 也会给 Broker 发送请求吗？”当然！**Controller会给集群中的所有Broker（包括它自己所在的Broker）机器发送网络请求**。发送请求的目的，是让 Broker 执行相应的指令。我用一张图，来展示下 Controller 都会发送哪些请求，如下所示：

![](<https://static001.geekbang.org/resource/image/3e/f7/3e8b0a34f003db5d67d5adafe8781ef7.jpg?wh=1403*1254>)

当前，Controller 只会向 Broker 发送三类请求，分别是 LeaderAndIsrRequest、StopReplicaRequest 和 UpdateMetadataRequest。注意，这里我使用的是“当前”！我只是说，目前仅有这三类，不代表以后不会有变化。事实上，我几乎可以肯定，以后能发送的 RPC 协议种类一定会变化的。因此，你需要掌握请求发送的原理。毕竟，所有请求发送都是通过相同的机制完成的。

还记得我在[第8节课](<https://time.geekbang.org/column/article/232134>)提到的控制类请求吗？没错，这三类请求就是典型的控制类请求。我来解释下它们的作用。

- LeaderAndIsrRequest：最主要的功能是，告诉 Broker 相关主题各个分区的 Leader 副本位于哪台 Broker 上、ISR 中的副本都在哪些 Broker 上。在我看来，**它应该被赋予最高的优先级，毕竟，它有令数据类请求直接失效的本领**。试想一下，如果这个请求中的 Leader 副本变更了，之前发往老的 Leader 的 PRODUCE 请求是不是全部失效了？因此，我认为它是非常重要的控制类请求。
- StopReplicaRequest：告知指定 Broker 停止它上面的副本对象，该请求甚至还能删除副本底层的日志数据。这个请求主要的使用场景，是**分区副本迁移**和**删除主题**。在这两个场景下，都要涉及停掉 Broker 上的副本操作。
- UpdateMetadataRequest：顾名思义，该请求会更新 Broker 上的元数据缓存。集群上的所有元数据变更，都首先发生在 Controller 端，然后再经由这个请求广播给集群上的所有 Broker。在我刚刚分享的案例中，正是因为这个请求被处理得不及时，才导致集群 Broker 无法获取到最新的元数据信息。

<!-- -->

现在，社区越来越倾向于**将重要的数据结构源代码从服务器端的core工程移动到客户端的clients工程中**。这三类请求 Java 类的定义就封装在 clients 中，它们的抽象基类是 AbstractControlRequest 类，这个类定义了这三类请求的公共字段。

我用代码展示下这三类请求及其抽象父类的定义，以便让你对 Controller 发送的请求类型有个基本的认识。这些类位于 clients 工程下的 src/main/java/org/apache/kafka/common/requests 路径下。

先来看 AbstractControlRequest 类的主要代码：

```
public abstract class AbstractControlRequest extends AbstractRequest {
    public static final long UNKNOWN_BROKER_EPOCH = -1L;
    public static abstract class Builder<T extends AbstractRequest> extends AbstractRequest.Builder<T> {
        protected final int controllerId;
        protected final int controllerEpoch;
        protected final long brokerEpoch;
        ......
}
```

区别于其他的数据类请求，抽象类请求必然包含 3 个字段。

- **controllerId**：Controller 所在的 Broker ID。
- **controllerEpoch**：Controller 的版本信息。
- **brokerEpoch**：目标 Broker 的 Epoch。

<!-- -->

后面这两个 Epoch 字段用于隔离 Zombie Controller 和 Zombie Broker，以保证集群的一致性。

在同一源码路径下，你能找到 LeaderAndIsrRequest、StopReplicaRequest 和 UpdateMetadataRequest 的定义，如下所示：

```
public class LeaderAndIsrRequest extends AbstractControlRequest { ...... }
public class StopReplicaRequest extends AbstractControlRequest { ...... }
public class UpdateMetadataRequest extends AbstractControlRequest { ...... }
```

## RequestSendThread

说完了 Controller 发送什么请求，接下来我们说说怎么发。

Kafka 源码非常喜欢生产者-消费者模式。该模式的好处在于，**解耦生产者和消费者逻辑，分离两者的集中性交互**。学完了“请求处理”模块，现在，你一定很赞同这个说法吧。还记得 Broker 端的 SocketServer 组件吗？它就在内部定义了一个线程共享的请求队列：它下面的 Processor 线程扮演 Producer，而 KafkaRequestHandler 线程扮演 Consumer。

对于 Controller 而言，源码同样使用了这个模式：它依然是一个线程安全的阻塞队列，Controller 事件处理线程（第 13 节课会详细说它）负责向这个队列写入待发送的请求，而一个名为 RequestSendThread 的线程负责执行真正的请求发送。如下图所示：

![](<https://static001.geekbang.org/resource/image/82/21/825d084eb1517daace5532d1c93b0321.jpg?wh=2186*1232>)

Controller 会为集群中的每个 Broker 都创建一个对应的 RequestSendThread 线程。Broker 上的这个线程，持续地从阻塞队列中获取待发送的请求。

那么，Controller 往阻塞队列上放什么数据呢？这其实是由源码中的 QueueItem 类定义的。代码如下：

```
case class QueueItem(apiKey: ApiKeys, request: AbstractControlRequest.Builder[_ <: AbstractControlRequest], callback: AbstractResponse => Unit, enqueueTimeMs: Long)
```

每个 QueueItem 的核心字段都是**AbstractControlRequest.Builder对象**。你基本上可以认为，它就是阻塞队列上 AbstractControlRequest 类型。

需要注意的是这里的“<:”符号，它在 Scala 中表示**上边界**的意思，即字段 request 必须是 AbstractControlRequest 的子类，也就是上面说到的那三类请求。

这也就是说，每个 QueueItem 实际保存的都是那三类请求中的其中一类。如果使用一个 BlockingQueue 对象来保存这些 QueueItem，那么，代码就实现了一个请求阻塞队列。这就是 RequestSendThread 类做的事情。

接下来，我们就来学习下 RequestSendThread 类的定义。我给一些主要的字段添加了注释。

```
class RequestSendThread(val controllerId: Int, // Controller所在Broker的Id
    val controllerContext: ControllerContext, // Controller元数据信息
    val queue: BlockingQueue[QueueItem], // 请求阻塞队列
    val networkClient: NetworkClient, // 用于执行发送的网络I/O类
    val brokerNode: Node, // 目标Broker节点
    val config: KafkaConfig, // Kafka配置信息
    val time: Time, 
    val requestRateAndQueueTimeMetrics: Timer,
    val stateChangeLogger: StateChangeLogger,
    name: String) extends ShutdownableThread(name = name) {
    ......
}
```

其实，RequestSendThread 最重要的是它的**doWork方法**，也就是执行线程逻辑的方法：

```
override def doWork(): Unit = {
    def backoff(): Unit = pause(100, TimeUnit.MILLISECONDS)
    val QueueItem(apiKey, requestBuilder, callback, enqueueTimeMs) = queue.take() // 以阻塞的方式从阻塞队列中取出请求
    requestRateAndQueueTimeMetrics.update(time.milliseconds() - enqueueTimeMs, TimeUnit.MILLISECONDS) // 更新统计信息
    var clientResponse: ClientResponse = null
    try {
      var isSendSuccessful = false
      while (isRunning && !isSendSuccessful) {
        try {
          // 如果没有创建与目标Broker的TCP连接，或连接暂时不可用
          if (!brokerReady()) {
            isSendSuccessful = false
            backoff() // 等待重试
          }
          else {
            val clientRequest = networkClient.newClientRequest(brokerNode.idString, requestBuilder,
              time.milliseconds(), true)
            // 发送请求，等待接收Response
            clientResponse = NetworkClientUtils.sendAndReceive(networkClient, clientRequest, time)
            isSendSuccessful = true
          }
        } catch {
          case e: Throwable =>
            warn(s"Controller $controllerId epoch ${controllerContext.epoch} fails to send request $requestBuilder " +
              s"to broker $brokerNode. Reconnecting to broker.", e)
            // 如果出现异常，关闭与对应Broker的连接
            networkClient.close(brokerNode.idString)
            isSendSuccessful = false
            backoff()
        }
      }
      // 如果接收到了Response
      if (clientResponse != null) {
        val requestHeader = clientResponse.requestHeader
        val api = requestHeader.apiKey
        // 此Response的请求类型必须是LeaderAndIsrRequest、StopReplicaRequest或UpdateMetadataRequest中的一种
        if (api != ApiKeys.LEADER_AND_ISR && api != ApiKeys.STOP_REPLICA && api != ApiKeys.UPDATE_METADATA)
          throw new KafkaException(s"Unexpected apiKey received: $apiKey")
        val response = clientResponse.responseBody
        stateChangeLogger.withControllerEpoch(controllerContext.epoch)
          .trace(s"Received response " +
          s"${response.toString(requestHeader.apiVersion)} for request $api with correlation id " +
          s"${requestHeader.correlationId} sent to broker $brokerNode")

        if (callback != null) {
          callback(response) // 处理回调
        }
      }
    } catch {
      case e: Throwable =>
        error(s"Controller $controllerId fails to send a request to broker $brokerNode", e)
        networkClient.close(brokerNode.idString)
    }
  }
```

我用一张图来说明 doWork 的执行逻辑：

![](<https://static001.geekbang.org/resource/image/86/19/869727e22f882509a149d1065a8a1719.jpg?wh=1799*2675>)

总体上来看，doWork 的逻辑很直观。它的主要作用是从阻塞队列中取出待发送的请求，然后把它发送出去，之后等待 Response 的返回。在等待 Response 的过程中，线程将一直处于阻塞状态。当接收到 Response 之后，调用 callback 执行请求处理完成后的回调逻辑。

需要注意的是，RequestSendThread 线程对请求发送的处理方式与 Broker 处理请求不太一样。它调用的 sendAndReceive 方法在发送完请求之后，会原地进入阻塞状态，等待 Response 返回。只有接收到 Response，并执行完回调逻辑之后，该线程才能从阻塞队列中取出下一个待发送请求进行处理。

## ControllerChannelManager

了解了 RequestSendThread 线程的源码之后，我们进入到 ControllerChannelManager 类的学习。

这个类和 RequestSendThread 是合作共赢的关系。在我看来，它有两大类任务。

- 管理 Controller 与集群 Broker 之间的连接，并为每个 Broker 创建 RequestSendThread 线程实例；
- 将要发送的请求放入到指定 Broker 的阻塞队列中，等待该 Broker 专属的 RequestSendThread 线程进行处理。

<!-- -->

由此可见，它们是紧密相连的。

ControllerChannelManager 类最重要的数据结构是 brokerStateInfo，它是在下面这行代码中定义的：

```
protected val brokerStateInfo = new HashMap[Int, ControllerBrokerStateInfo]
```

这是一个 HashMap 类型，Key 是 Integer 类型，其实就是集群中 Broker 的 ID 信息，而 Value 是一个 ControllerBrokerStateInfo。

你可能不太清楚 ControllerBrokerStateInfo 类是什么，我先解释一下。它本质上是一个 POJO 类，仅仅是承载若干数据结构的容器，如下所示：

```
case class ControllerBrokerStateInfo(networkClient: NetworkClient,
    brokerNode: Node,
    messageQueue: BlockingQueue[QueueItem],
    requestSendThread: RequestSendThread,
    queueSizeGauge: Gauge[Int],
    requestRateAndTimeMetrics: Timer,
reconfigurableChannelBuilder: Option[Reconfigurable])
```

它有三个非常关键的字段。

- **brokerNode**：目标 Broker 节点对象，里面封装了目标 Broker 的连接信息，比如主机名、端口号等。
- **messageQueue**：请求消息阻塞队列。你可以发现，Controller 为每个目标 Broker 都创建了一个消息队列。
- **requestSendThread**：Controller 使用这个线程给目标 Broker 发送请求。

<!-- -->

你一定要记住这三个字段，因为它们是实现 Controller 发送请求的关键因素。

为什么呢？我们思考一下，如果 Controller 要给 Broker 发送请求，肯定需要解决三个问题：发给谁？发什么？怎么发？“发给谁”就是由 brokerNode 决定的；messageQueue 里面保存了要发送的请求，因而解决了“发什么”的问题；最后的“怎么发”就是依赖 requestSendThread 变量实现的。

好了，我们现在回到 ControllerChannelManager。它定义了 5 个 public 方法，我来一一介绍下。

- **startup方法**：Controller 组件在启动时，会调用 ControllerChannelManager 的 startup 方法。该方法会从元数据信息中找到集群的 Broker 列表，然后依次为它们调用 addBroker 方法，把它们加到 brokerStateInfo 变量中，最后再依次启动 brokerStateInfo 中的 RequestSendThread 线程。
- **shutdown方法**：关闭所有 RequestSendThread 线程，并清空必要的资源。
- **sendRequest方法**：从名字看，就是发送请求，实际上就是把请求对象提交到请求队列。
- **addBroker方法**：添加目标 Broker 到 brokerStateInfo 数据结构中，并创建必要的配套资源，如请求队列、RequestSendThread 线程对象等。最后，RequestSendThread 启动线程。
- **removeBroker方法**：从 brokerStateInfo 移除目标 Broker 的相关数据。

<!-- -->

这里面大部分的方法逻辑都很简单，从方法名字就可以看得出来。我重点说一下**addBroker**，以及**底层相关的私有方法addNewBroker和startRequestSendThread方法**。

毕竟，addBroker 是最重要的逻辑。每当集群中扩容了新的 Broker 时，Controller 就会调用这个方法为新 Broker 增加新的 RequestSendThread 线程。

我们先来看 addBroker：

```
def addBroker(broker: Broker): Unit = {
    brokerLock synchronized {
      // 如果该Broker是新Broker的话
      if (!brokerStateInfo.contains(broker.id)) {
        // 将新Broker加入到Controller管理，并创建对应的RequestSendThread线程
        addNewBroker(broker) 
        // 启动RequestSendThread线程
        startRequestSendThread(broker.id)
      }
    }
  }
```

整个代码段被 brokerLock 保护起来了。还记得 brokerStateInfo 的定义吗？它仅仅是一个 HashMap 对象，因为不是线程安全的，所以任何访问该变量的地方，都需要锁的保护。

这段代码的逻辑是，判断目标 Broker 的序号，是否已经保存在 brokerStateInfo 中。如果是，就说明这个 Broker 之前已经添加过了，就没必要再次添加了；否则，addBroker 方法会对目前的 Broker 执行两个操作：

1. 把该 Broker 节点添加到 brokerStateInfo 中；
2. 启动与该 Broker 对应的 RequestSendThread 线程。

<!-- -->

这两步分别是由 addNewBroker 和 startRequestSendThread 方法实现的。

addNewBroker 方法的逻辑比较复杂，我用注释的方式给出主要步骤：

```
private def addNewBroker(broker: Broker): Unit = {
  // 为该Broker构造请求阻塞队列
  val messageQueue = new LinkedBlockingQueue[QueueItem]
  debug(s"Controller ${config.brokerId} trying to connect to broker ${broker.id}")
  val controllerToBrokerListenerName = config.controlPlaneListenerName.getOrElse(config.interBrokerListenerName)
  val controllerToBrokerSecurityProtocol = config.controlPlaneSecurityProtocol.getOrElse(config.interBrokerSecurityProtocol)
  // 获取待连接Broker节点对象信息
  val brokerNode = broker.node(controllerToBrokerListenerName)
  val logContext = new LogContext(s"[Controller id=${config.brokerId}, targetBrokerId=${brokerNode.idString}] ")
  val (networkClient, reconfigurableChannelBuilder) = {
    val channelBuilder = ChannelBuilders.clientChannelBuilder(
      controllerToBrokerSecurityProtocol,
      JaasContext.Type.SERVER,
      config,
      controllerToBrokerListenerName,
      config.saslMechanismInterBrokerProtocol,
      time,
      config.saslInterBrokerHandshakeRequestEnable,
      logContext
    )
    val reconfigurableChannelBuilder = channelBuilder match {
      case reconfigurable: Reconfigurable =>
        config.addReconfigurable(reconfigurable)
        Some(reconfigurable)
      case _ => None
    }
    // 创建NIO Selector实例用于网络数据传输
    val selector = new Selector(
      NetworkReceive.UNLIMITED,
      Selector.NO_IDLE_TIMEOUT_MS,
      metrics,
      time,
      "controller-channel",
      Map("broker-id" -> brokerNode.idString).asJava,
      false,
      channelBuilder,
      logContext
    )
    // 创建NetworkClient实例
    // NetworkClient类是Kafka clients工程封装的顶层网络客户端API
    // 提供了丰富的方法实现网络层IO数据传输
    val networkClient = new NetworkClient(
      selector,
      new ManualMetadataUpdater(Seq(brokerNode).asJava),
      config.brokerId.toString,
      1,
      0,
      0,
      Selectable.USE_DEFAULT_BUFFER_SIZE,
      Selectable.USE_DEFAULT_BUFFER_SIZE,
      config.requestTimeoutMs,
      ClientDnsLookup.DEFAULT,
      time,
      false,
      new ApiVersions,
      logContext
    )
    (networkClient, reconfigurableChannelBuilder)
  }
  // 为这个RequestSendThread线程设置线程名称
  val threadName = threadNamePrefix match {
    case None => s"Controller-${config.brokerId}-to-broker-${broker.id}-send-thread"
    case Some(name) => s"$name:Controller-${config.brokerId}-to-broker-${broker.id}-send-thread"
  }
  // 构造请求处理速率监控指标
  val requestRateAndQueueTimeMetrics = newTimer(
    RequestRateAndQueueTimeMetricName, TimeUnit.MILLISECONDS, TimeUnit.SECONDS, brokerMetricTags(broker.id)
  )
  // 创建RequestSendThread实例
  val requestThread = new RequestSendThread(config.brokerId, controllerContext, messageQueue, networkClient,
    brokerNode, config, time, requestRateAndQueueTimeMetrics, stateChangeLogger, threadName)
  requestThread.setDaemon(false)

  val queueSizeGauge = newGauge(QueueSizeMetricName, () => messageQueue.size, brokerMetricTags(broker.id))
  // 创建该Broker专属的ControllerBrokerStateInfo实例
  // 并将其加入到brokerStateInfo统一管理
  brokerStateInfo.put(broker.id, ControllerBrokerStateInfo(networkClient, brokerNode, messageQueue,
    requestThread, queueSizeGauge, requestRateAndQueueTimeMetrics, reconfigurableChannelBuilder))
}
```

为了方便你理解，我还画了一张流程图形象说明它的执行流程：

![](<https://static001.geekbang.org/resource/image/4f/22/4f34c319f9480c16ac12dee78d5ba322.jpg?wh=4470*2201>)

addNewBroker 的关键在于，**要为目标Broker创建一系列的配套资源**，比如，NetworkClient 用于网络 I/O 操作、messageQueue 用于阻塞队列、requestThread 用于发送请求，等等。

至于 startRequestSendThread 方法，就简单得多了，只有几行代码而已。

```
protected def startRequestSendThread(brokerId: Int): Unit = {
  // 获取指定Broker的专属RequestSendThread实例
  val requestThread = brokerStateInfo(brokerId).requestSendThread
  if (requestThread.getState == Thread.State.NEW)
    // 启动线程
    requestThread.start()
}
```

它首先根据给定的 Broker 序号信息，从 brokerStateInfo 中找出对应的 ControllerBrokerStateInfo 对象。有了这个对象，也就有了为该目标 Broker 服务的所有配套资源。下一步就是从 ControllerBrokerStateInfo 中拿出 RequestSendThread 对象，再启动它就好了。

## 总结

今天，我结合 ControllerChannelManager.scala 文件，重点分析了 Controller 向 Broker 发送请求机制的实现原理。

Controller 主要通过 ControllerChannelManager 类来实现与其他 Broker 之间的请求发送。其中，ControllerChannelManager 类中定义的 RequestSendThread 是主要的线程实现类，用于实际发送请求给集群 Broker。除了 RequestSendThread 之外，ControllerChannelManager 还定义了相应的管理方法，如添加 Broker、移除 Broker 等。通过这些管理方法，Controller 在集群扩缩容时能够快速地响应到这些变化，完成对应 Broker 连接的创建与销毁。

我们来回顾下这节课的重点。

- Controller 端请求：Controller 发送三类请求给 Broker，分别是 LeaderAndIsrRequest、StopReplicaRequest 和 UpdateMetadataRequest。
- RequestSendThread：该线程负责将请求发送给集群中的相关或所有 Broker。
- 请求阻塞队列+RequestSendThread：Controller 会为集群上所有 Broker 创建对应的请求阻塞队列和 RequestSendThread 线程。

<!-- -->

其实，今天讲的所有东西都只是这节课的第二张图中“消费者”的部分，我们并没有详细了解请求是怎么被放到请求队列中的。接下来，我们就会针对这个问题，深入地去探讨 Controller 单线程的事件处理器是如何实现的。

![](<https://static001.geekbang.org/resource/image/00/b9/00fce28d26a94389f2bb5e957b650bb9.jpg?wh=2308*1068>)

## 课后讨论

你觉得，为每个 Broker 都创建一个 RequestSendThread 的方案有什么优缺点？

欢迎你在留言区写下你的思考和答案，跟我交流讨论，也欢迎你把今天的内容分享给你的朋友。

