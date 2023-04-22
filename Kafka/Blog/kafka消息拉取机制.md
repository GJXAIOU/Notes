# Kafka 消息拉取机制

在详细介绍 Kafka 拉取之前，我们再来回顾一下消息拉取的整体流程：
<img src="kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/消息拉取整体流程.png" alt="在这里插入图片描述" style="zoom: 33%;" />
在消费者加入到消费组后，消费者 Leader 会根据当前在线消费者个数与分区的数量进行队列负载，每一个消费者获得一部分分区，接下来就是要从 Broker 服务端将数据拉取下来，提交给消费端进行消费，对应流程中的 pollForFetches 方法。

要正确写出优秀的 Kafka 端消费代码，详细了解其拉取模型是非常重要的一步。

## 1、消息拉取详解

#### 1.1 消费端拉取流程详解

消息拉取的实现入口为：KafkaConsumer 的 pollForFetches，接下来我们将详细剖析其流程，探讨 kafka 消息拉取模型，其实现如下所示：
![在这里插入图片描述](kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/源码一.png)
整个消息拉取的核心步骤如下：

- 获取本次拉取的超时时间，会取自用户设置的超时时间与一个心跳包的间隔之中的最小值。
- 从**拉取缓存区**中解析已异步拉取的消息。
- **向Broker发送拉取请求，该请求是一个异步请求**。
- 通过 ConsumerNetworkClient 触发底层 NIO 通信。
- 再次尝试从缓存区中解析已拉起的消息。

#### 1.1 Fetcher 的sendFetches详解

经过队列负载算法分配到部分分区后，消费者接下来需要向 Broker 发送消息拉起请求，具体由 sendFetches 方法实现。

![在这里插入图片描述](kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/源码二.jpg)

Step1:通过调用 preparefetchRequest，构建请求对象，其实现的核心要点如下：

- 构建一个请求列表，这里采用了 Build 设计模式，最终生成的请求对象：Node 为 Key,FetchSessionHandler.FetchRequestData 为 Value 的请求，我觉得这里有必要看一下 FetchRequestData 的数据结构：
  <img src="kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/构建者模式.png" alt="在这里插入图片描述" style="zoom: 50%;" />
  其中 ParitionData 汇总包含了本次消息拉取的开始位点。
- 通过 fetchablePartitions 方法获取本次可拉取的队列，其核心实现要点如下：
  - 从队列负载结果中获取可拉取的分区信息，主要的判断标准：**未被暂停与有正确位点信息**。
  - nextInLineRecords？
  - 去除掉拉取缓存区中的存在队列信息(completedFetches)，即**如果缓存区中的数据未被消费端消费则不会继续拉取新的内容**。
- 获取待拉取分区所在的 leader 信息，如果未找到，本次拉取将忽略该分区，但是会设置需要更新 topic 路由信息,在下次拉取之前会从 Broker 拉取最新的路由信息。
- **如果客户端与待拉取消息的broker节点有待发送的网络请求**(见代码@4)，则本次拉取任务将**不会再发起新的拉取请求**，待已有的请求处理完毕后才会拉取新的消息。
- 拉取消息时需要指定拉取消息偏移量，来自队列负载算法时指定，主要消费组的最新消费位点。

![在这里插入图片描述](kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/源码三.png)
Step2:按 Node 依次构建请求节点，并通过 client 的 send 方法将请求异步发送，当收到请求结果后会调用对应的事件监听器，这里主要的是一次拉取最大的字节数 50M。

> 值得注意的是在Kafka中调用client的send方法并不会真正触发网络请求，而是将请求放到发送缓冲区中，Client的poll方法才会真正触发底层网络请求。

Step3:当客户端收到服务端请求后会将原始结果放入到 completedFetches 中，等待客户端从中解析。

> 本篇文章暂时不关注服务端对fetch请求的处理，等到详细剖析了Kafka的存储相关细节后再回过来看Fetch请求的响应。

#### 1.2 Fetcher的fetchedRecords方法详解

向服务端发送拉取请求异步返回后会将结果返回到一个 completedFetches 中，也可以理解为**接收缓存区**,接下来将从缓存区中将结果解析并返回给消费者消费。从接收缓存区中解析数据的具体实现见 Fetcher 的 fetchedRecords 方法。
![在这里插入图片描述](kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/源码四.png)

核心实现要点如下：

- 首先说明一下 nextInLineRecords 的含义，接下来的 fetchedRecords 方法将从这里获取值，该参数主要是因为引入了 maxPollRecords(默认为 500)，一次拉取的消息条数，一次 Fetch 操作一次每一个分区最多返回 50M 数据，可能包含的消息条数大于 maxPollRecords。

  如果 nextInLineRecords 为空或者所有内容已被拉取，则从 completedFetch 中解析。

- 从 completedFetch 中解析解析成 nextInlineRecords。

- 从 nextInlineRecords 中继续解析数据。

关于将 CompletedFetch 中解析成 PartitionRecords 以及从 PartitionRecords 提取数据成 Map< TopicPartition, List< ConsumerRecord< K, V>>>的最终供应用程序消费的数据结构，代码实现非常简单，这里就不再介绍。

**有关服务端响应SEND_FETCH的相关分析，将在详细分析Kafka存储相关机制时再介绍**。在深入存储细节时，从消息拉取，消息写入为切入点是一个非常不错的选择。

## 2、消息消费端模型

**阅读源码**是手段而不是目的，通过阅读源码，我们应该总结提炼一下 Kafka 消息拉取模型(特点)，以便更好的指导实践。

首先再强调一下消费端的三个重要参数：

- fetch.max.bytes

  客户端单个 Fetch 请求一次拉取的最大字节数，默认为**50M**，根据上面的源码分析得知，**Kafka会按Broker节点为维度进行拉取**， 即按照队列负载算法分配在同一个 Broker 上的多个队列进行聚合，同时尽量保证各个分区的拉取平衡，通过 max.partition.fetch.bytes 参数设置。

- max.partition.fetch.bytes
  一次 fetch 拉取单个队列最大拉取字节数量，默认为 1M。

- max.poll.records
  调用一次 KafkaConsumer 的 poll 方法，返回的消息条数，默认为 500 条。

> 实践思考：fetch.max.bytes默认是max.partition.fetch.bytes的50倍，也就是默认情况一下，一个消费者一个Node节点上至少需要分配到50个队列，才能尽量**满额**拉取。**但50个分区(队列)可以来源于这个消费组订阅的所有的topic**。

#### 2.1Kafka消费线程拉取线程模型

KafkaConsumer 并不是线程安全的，即 KafkaConsumer 的所有方法调用必须在同一个线程中，但消息拉取却是是并发的，线程模型说明如下图所示：

![在这里插入图片描述](kafka%E6%B6%88%E6%81%AF%E6%8B%89%E5%8F%96%E6%9C%BA%E5%88%B6.resource/线程拉取模型.png)

其核心设计理念是 KafkaConsumer 在调用 poll 方法时，如果**本地缓存区中(completedFeches)**存在未拉取的消息，则直接从本地缓存区中拉取消息，否则会调用 client#send 方法进行异步多线程并行发送拉取请求，发往不同的 broker 节点的请求是并发执行，执行完毕后，**再将结果放入到poll方法所在线程中的缓存区，实现多个线程的协同**。

#### 2.2 poll方法返回给消费端线程特点

pol l 方法会从缓存区中依次获取一个 CompletedFetch 对象，一次只从 CompletedFetch 对象中获取 500 条消息，一个 CompletedFetch 对象包含一个分区的数据，默认最大的消息体大小为 1M，可通过 max.partition.fetch.bytes 改变默认值。

如果一个分区中消息超过 500 条，则 KafkaConsumer 的 poll 方法将只会返回 1 个分区的数据，这样在**顺序消费**时基于单分区的顺序性保证时**如果采取与RocketMQl类似的机制，对分区加锁**，则其并发度非常低，因为此时顺序消费的并发度取决于这 500 条消息**包含的分区个数**。

**Kafka顺序消费最佳实践**： 单分区中消息可以并发执行，但要保证同一个 key 的消息必须串行执行。因为在实践应用场景中，通常只需要同一个业务实体的不同消息顺序执行。