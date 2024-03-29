# Kafka在美团数据平台的实践

> 原文：https://tech.meituan.com/2022/08/04/the-practice-of-kafka-in-the-meituan-data-platform.html

Kafka 在美团数据平台承担着统一的数据缓存和分发的角色，随着数据量的增长，集群规模的扩大，Kafka 面临的挑战也愈发严峻。本文分享了美团 Kafka 面临的实际挑战，以及美团针对性的一些优化工作，希望能给从事相关开发工作的同学带来帮助或启发。

- 1. 现状和挑战

  - 1.1 现状
  - 1.2 挑战


- 2. 读写延迟优化

  - 2.1 概览

  - 2.2 应用层
  - 2.3 系统层
  - 2.4 混合层-SSD 新缓存架构

- 3. 大规模集群管理优化

  - 3.1 隔离策略
  - 3.2 全链路监控
  - 3.3 服务生命周期管理
  - 3.4 TOR 容灾

- 4 未来展望

## 1. 现状和挑战

### 1.1 现状

Kafka 是一个开源的流处理平台，我们首先了解一下 Kafka 在美团数据平台的现状。

<img src="meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846697-2.jpeg" alt="图片" style="zoom:50%;" />

图 1-1 Kafka 在美团数据平台的现状

如图 1-1 所示，蓝色部分描述了 **Kafka 在数据平台定位为流存储层。主要的职责是做数据的缓存和分发**，它会将收集到的日志分发到不同的数据系统里，这些日志来源于系统日志、客户端日志以及业务数据库。下游的数据消费系统包括通过 ODS 入仓提供离线计算使用、直接供实时计算使用、通过 DataLink 同步到日志中心，以及做 OLAP 分析使用。

Kafka 在美团的集群规模总体机器数已经超过了 15000+ 台，单集群的最大机器数也已经到了 2000+ 台。在数据规模上，天级消息量已经超过了 30+P，天级消息量峰值也达到了 4+亿/秒。不过随着集群规模的增大，数据量的增长，Kafka 面临的挑战也愈发严峻，下面讲一下具体的挑战都有哪些。

### 1.2 挑战

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-3.jpeg)

图 1-2 Kafka 在美团数据平台面临的挑战

如图 1-2 所示，具体的挑战可以概括为两部分：

**第一部分是慢节点影响读写**，这里慢节点参考了 HDFS 的一个概念，具体定义指的是读写延迟 TP99 大于 300ms 的 Broker。造成慢节点的原因有三个：

- 集群负载不均衡会导致局部热点，就是整个集群的磁盘空间很充裕或者 ioutil 很低，但部分磁盘即将写满或者 ioutil 打满。

- PageCache 容量，比如说，80GB 的 PageCache 在 170MB/s 的写入量下仅能缓存 8 分钟的数据量。那么如果消费的数据是 8 分钟前的数据，就有可能触发慢速的磁盘访问。

- Consumer 客户端的线程模型缺陷会导致端到端延时指标失真。例如当 Consumer 消费的多个分区处于同一 Broker 时，TP90 可能小于 100ms，但是当多个分区处于不同 Broker 时，TP90 可能会大于 1000ms。

**第二部分是大规模集群管理的复杂性**，具体表现有 4 类问题：

- 不同 Topic 之间会相互影响，个别 Topic 的流量突增，或者个别消费者的回溯读会影响整体集群的稳定性。
- Kafka 原生的 Broker 粒度指标不够健全，导致问题定位和根因分析困难。
- 故障感知不及时，处理成本较高。
- Rack 级别的故障会造成部分分区不可用。

## 2. 读写延迟优化

接下来我们先介绍一下针对读写延迟问题，美团数据平台做了哪些优化。首先从宏观层面，我们将受影响因素分为应用层和系统层，然后详细介绍应用层和系统层存在的问题，并给出对应的解决方案，包括流水线加速、Fetcher 隔离、迁移取消和 Cgroup 资源隔离等，下面具体介绍各种优化方案的实现。

### 2.1 概览

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-4.jpeg)

图 2-1 Kafka 读写延迟优化概览

图 2-1 是针对读写延迟碰到的问题以及对应优化方案的概览图。我们把受影响的因素分为应用层和系统层。

**应用层**主要包括 3 类问题：

1）Broker 端负载不均衡，例如磁盘使用率不均衡、ioutil 不均衡等问题。个别磁盘负载升高影响整个 Broker 的请求受到影响。

2）Broker 的数据迁移存在效率问题和资源竞争问题。具体来讲，包括以下 3 个层面：

- 迁移只能按批次串行提交，每个批次可能存在少量分区迁移缓慢，无法提交下个批次，导致迁移效率受影响。
- 迁移一般在夜间执行，如果迁移拖到了午高峰还未完成，可能会显著影响读写请求。
- 迁移请求和实时拉取存在共用 Fetcher 线程的问题导致分区迁移请求可能会影响实时消费请求。

3）Consumer 端单线程模型存在缺陷导致运维指标失真，并且单 Consumer 消费的分区数不受限制，消费能力不足就无法跟上实时最新的数据，当消费的分区数增多时可能会引起回溯读。

**系统层**也主要包括 3 类问题：

1）PageCache 污染。Kafka 利用内核层提供的 ZeroCopy 技术提升性能，但是内核层无法区分实时读写请求和回溯读请求，导致磁盘读可能污染 PageCache，影响实时读写。

2）HDD 在随机读写负载下性能差。HDD 对于顺序读写友好，但是面对混合负载场景下的随机读写，性能显著下降。

3）CPU 和内存等系统资源在混部场景下的资源竞争问题。在美团大数据平台，为了提高资源的利用率，IO 密集型的服务（比如 Kafka）会和 CPU 密集型的服务（比如实时计算作业）混布，混布存在资源竞争，影响读写延迟。

以上提到的问题，我们采取了针对性的策略。比如应用层的磁盘均衡、迁移流水线加速、支持迁移取消和 Consumer 异步化等。系统层的 Raid 卡加速、Cgroup 隔离优化等。此外，针对 HDD 随机读写性能不足的问题，我们还设计并实现了基于 SSD 的缓存架构。

### 2.2 应用层

**① 磁盘均衡**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-5.jpeg)图 2-2 Kafka 应用层磁盘均衡

磁盘热点导致两个问题：

- 实时读写延迟变高，比如说 TP99 请求处理时间超过 300ms 可能会导致实时作业发生消费延迟问题，数据收集拥堵问题等。
- 集群整体利用率不足，虽然集群容量非常充裕，但是部分磁盘已经写满，这个时候甚至会导致某些分区停止服务。

针对这两个问题，我们采用了基于空闲磁盘优先的分区迁移计划，整个计划分为 3 步，由组件 Rebalancer 统筹管理：

1. 生成迁移计划。Rebalancer 通过目标磁盘使用率和当前磁盘使用率（通过 Kafka Monitor 上报）持续生成具体的分区迁移计划。
2. 提交迁移计划。Rebalancer 向 Zookeeper 的 Reassign 节点提交刚才生成的迁移计划，Kafka 的 Controller 收到这个 Reassign 事件之后会向整个 Kafka Broker 集群提交 Reassign 事件。
3. 检查迁移计划。Kafka Broker 负责具体执行数据迁移任务，Rebalancer 负责检查任务进展。

如图 2-2 所示，每块 Disk 持有 3 个分区是一个相对均衡的状态，如果部分 Disk 持有 4 个分区，比如 Broker1-Disk1 和 Broker4-Disk4；部分 Disk 持有 2 个分区，比如 Broker2-Disk2，Broker3-Disk3，Reblanacer 就会将 Broker1-Disk1 和 Broker4-Disk4 上多余的分区分别迁移到 Broker2-Disk2 和 Broker3-Disk3，最终尽可能地保证整体磁盘利用率均衡。

**② 迁移优化**

虽然基于空闲磁盘优先的分区迁移实现了磁盘均衡，但是迁移本身仍然存在效率问题和资源竞争问题。接下来，我们会详细描述我们采取的针对性策略。

1. 采取流水线加速策略优化迁移缓慢引起的迁移效率问题。
2. 支持迁移取消解决长尾分区迁移缓慢引起的读写请求受影响问题。
3. 采取 Fetcher 隔离缓解数据迁移请求和实时读写请求共用 Fetcher 线程的问题。

**优化一，流水线加速**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-6.jpeg)图 2-3 流水线加速

如图 2-3 所示，箭头以上原生 Kafka 版本只支持按批提交，比如说一批提交了四个分区，当 TP4 这个分区一直卡着无法完成的时候，后续所有分区都无法继续进行。采用流水线加速之后，即使 TP4 这个分区还没有完成，可以继续提交新的分区。在相同的时间内，原有的方案受阻于 TP4 没有完成，后续所有分区都没办法完成，在新的方案中，TP4 分区已经迁移到 TP11 分区了。图中虚线代表了一个无序的时间窗口，主要用于控制并发，目的是为了和原有的按组提交的个数保持一致，避免过多的迁移影响读写请求服务。

**优化二，迁移取消**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-7.jpeg)图 2-4-1 迁移问题

如图 2-4-1 所示，箭头左侧描述了因为迁移影响的三种线上类型。第一种是因为迁移会触发最旧读，同步大量的数据，在这个过程中会首先将数据回刷到 PageCache 上引起 PageCache 污染，导致某个实时读的分区发生 Cache Miss，触发磁盘度进而影响读写请求；第二种是当存在某些异常节点导致迁移 Hang 住时，部分运维操作无法执行，比如流量上涨触发的 Topic 自动扩分区。因为在 Kafka 迁移过程中这类运维操作被禁止执行。第三种和第二种类似，它的主要问题是当目标节点 Crash，Topic 扩分区也无法完成，用户可能一直忍受读写请求受影响。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-8.jpeg)图 2-4-2 迁移取消

针对上面提到的 3 种问题，我们支持了迁移取消功能。管理员可以调用迁移取消命令，中断正在迁移的分区，针对第一种场景，PageCache 就不会被污染，实时读得以保证；在第二、三种场景中，因为迁移取消，扩分区得以完成。迁移取消会删除未完成迁移的分区，删除可能会导致磁盘 IO 出现瓶颈影响读写，因此我们通过支持平滑删除避免大量删除引起的性能问题。

**优化三，Fetcher隔离**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846698-9.jpeg)图 2-5 Fetcher 隔离

如图 2-5，绿色代表实时读，红色代表延时读。当某一个 Follower 的实时读和延时读共享同一个 Fetcher 时，延时读会影响实时读。因为每一次延时读的数据量是显著大于实时读的，而且延时读容易触发磁盘读，可能数据已经不在 PageCache 中了，显著地拖慢了 Fetcher 的拉取效率。

针对这种问题，我们实施的策略叫 Fetcher 隔离。也就是说所有 ISR 的 Follower 共享 Fetcher，所有非 ISR 的 Follower 共享 Fetcher，这样就能保证所有 ISR 中的实时读不会被非 ISR 的回溯读所影响。

**③ Consumer异步化**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-10.jpeg)

图 2-6 Kafka-Broker 分阶段延时统计模型

在讲述 Consumer 异步化前，需要解释下图 2-6 展示的 Kafka-Broker 分阶段延时统计模型。Kafka-Broker 端是一个典型的事件驱动架构，各组件通过队列通信。请求在不同组件流转时，会依次记录时间戳，最终就可以统计出请求在不同阶段的执行耗时。

具体来说，当一个 Kafka 的 Producer 或 Consumer 请求进入到 Kafka-Broker 时，Processor 组件将请求写入 RequestQueue，RequestHandler 从 RequestQueue 拉取请求进行处理，在 RequestQueue 中的等待时间是 RequestQueueTime，RequestHandler 具体的执行时间是 LocalTime。当 RequestHandler 执行完毕后会将请求传递给 DelayedPurgatory 组件中，该组件是一个延时队列。

当触发某一个延时条件完成了以后会把请求写到 ResponseQueue 中，在 DelayedPurgatory 队列持续的时间为 RemoteTime，Processor 会不断的从 ResponseQueue 中将数据拉取出来发往客户端，标红的 ResponseTime 是可能会被客户端影响的，因为如果客户端接收能力不足，那么 ResponseTime 就会一直持续增加。从 Kafka-Broker 的视角，每一次请求总的耗时时 RequestTotalTime，包含了刚才所有流程分阶段计时总和。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-11.jpeg)

图 2-7 Consumer 异步化

ResponseTime 持续增加的主要问题是因为 Kafka 原生 Consumer 基于 NIO 的单线程模型存在缺陷。如图 2-7 所示，在 Phase1，User 首先发起 Poll 请求，Kafka-Client 会同时向 Broker1、Broker2 和 Broker3 发送请求，Broker1 的数据先就绪时，Kafka Client 将数据写入 CompleteQueue，并立即返回，而不是继续拉取 Broker2 和 Broker3 的数据。后续的 Poll 请求会直接从 CompleteQueue 中读取数据，然后直接返回，直到 CompleteQueue 被清空。在 CompleteQueue 被清空之前，即使 Broker2 和 Broker3 的端的数据已经就绪，也不会得到及时拉取。如图中 Phase2，因为单线程模型存在缺陷导致 WaitFetch 这部分时长变大，导致 Kafka-Broker 的 RespnseTime 延时指标不断升高，带来的问题是无法对服务端的处理瓶颈进行精准的监控与细分。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-12.jpeg)

图 2-8 引入异步拉取线程

针对这个问题，我们的改进是引入异步拉取线程。异步拉取线程会及时地拉取就绪的数据，避免服务端延时指标受影响，而且原生 Kafka 并没有限制同时拉取的分区数，我们在这里做了限速，避免 GC 和 OOM 的发生。异步线程在后台持续不断地拉取数据并放到 CompleteQueue 中。

### 2.3 系统层

**① Raid卡加速**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-13.jpeg)

图 2-9 Raid 卡加速

HDD 存在随机写性能不足的问题，表现为延时升高，吞吐降低。针对这个问题我们引入了 Raid 卡加速。Raid 卡自带缓存，与 PageCache 类似，在 Raid 这一层会把数据 Merge 成更大的 Block 写入 Disk，更加充分利用顺序写 HDD 的带宽，借助 Raid 卡保证了随机写性能。

**② Cgroup隔离优化**

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-14.jpeg)

图 2-10 Cgroup 隔离

为了提高资源利用率，美团数据平台将 IO 密集型应用和 CPU 密集型应用混合部署。IO 密集型应用在这里指的就是 Kafka，CPU 密集型应用在这里指的是 Flink 和 Storm。但是原有的隔离策略存在两个问题：首先是物理核本身会存在资源竞争，在同一个物理核下，共享的 L1Cache 和 L2Cache 都存在竞争，当实时平台 CPU 飙升时会导致 Kafka 读写延时受到影响；其次，Kafka 的 HT 跨 NUMA，增加内存访问耗时，如图 2-10 所示，跨 NUMA 节点是通过 QPI 去做远程访问，而这个远程访问的耗时是 40ns。

针对这两个问题，我们改进了隔离策略，针对物理核的资源竞争，我们新的混布策略保证 Kafka 独占物理核，也就是说在新的隔离策略中，不存在同一个物理核被 Kafka 和 Flink 同时使用；然后是保证 Kafka 的所有超线程处于同一侧的 NUMA，避免 Kafka 跨 NUMA 带来的访问延时。通过新的隔离策略，Kafka 的读写延时不再受 Flink CPU 飙升的影响。

### 2.4 混合层-SSD新缓存架构

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-15.jpeg)

图 2-11 Page 污染引起的性能问题

**背景和挑战**

Kafka 利用操作系统提供的 ZeroCopy 技术处理数据读取请求，PageCache 容量充裕时数据直接从 PageCache 拷贝到网卡，有效降低了读取延时。但是实际上，PageCache 的容量往往是不足的，因为它不会超过一个机器的内存。容量不足时，ZeroCopy 就会触发磁盘读，磁盘读不仅显著变慢，还会污染 PageCache 影响其他读写。

如图 2-11 中左半部分所示，当一个延迟消费者去拉取数据时，发现 PageCache 中没有它想要的数据，这个时候就会触发磁盘读。磁盘读后会将数据回写到 PageCache，导致 PageCache 污染，延迟消费者消费延迟变慢的同时也会导致另一个实时消费受影响。因为对于实时消费而言，它一直读的是最新的数据，最新的数据按正常来说时不应该触发磁盘读的。

**选型和决策**

针对这个问题，我们这边在做方案选型时提供了两种方案：

**方案一**，读磁盘时不回写 PageCache，比如使用 DirectIO，不过 Java 并不支持；

**方案二**，在内存和 HDD 之间引入中间层，比如 SSD。众所周知，SSD 和 HDD 相比具备良好的随机读写能力，非常适合我们的使用场景。针对 SSD 的方案我们也有两种选型：

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846699-16.png)

**方案一**，可以基于操作系统的内核实现，这种方案 SSD 与 HDD 存储空间按照固定大小分块，并且 SSD 与 HDD 建立映射关系，同时会基于数据局部性原理，Cache Miss 后数据会按 LRU 和 LFU 替换 SSD 中部分数据，业界典型方案包括 OpenCAS 和 FlashCache。其优势是数据路由对应用层透明，对应用代码改动量小，并且社区活跃可用性好；但是问题在于局部性原理并不满足 Kafka 的读写特性，而且缓存空间污染问题并未得到根本解决，因为它会根据 LRU 和 LFU 去替换 SSD 中的部分数据。

**方案二**，基于 Kafka 的应用层去实现，具体就是 Kafka 的数据按照时间维度存储在不同设备上，对于近实时数据直接放在 SSD 上，针对较为久远的数据直接放在 HDD 上，然后 Leader 直接根据 Offset 从对应设备读取数据。这种方案的优势是它的缓存策略充分考虑了 Kafka 的读写特性，确保近实时的数据消费请求全部落在 SSD 上，保证这部分请求处理的低延迟，同时从 HDD 读取的数据不回刷到 SSD 防止缓存污染，同时由于每个日志段都有唯一明确的状态，因此每次请求目的明确，不存在因 Cache Miss 带来的额外性能开销。同时劣势也很明显，需要在 Server 端代码上进行改进，涉及的开发以及测试的工作量较大。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-17.jpeg)

图 2-13 KafkaSSD 新缓存架构

**具体实现**

下面来介绍一下 SSD 新缓存架构的具体实现。

1. 首先新的缓存架构会将 Log 内的多个 Segment 按时间维度存储在不同的存储设备上，如图 2-14 中的红圈 1，新缓存架构数据会有三种典型状态，一种叫 Only Cache，指的是数据刚写进 SSD，还未同步到 HDD 上；第 2 个是 Cached，指数据既同步到了 HDD 也有一部分缓存在 SSD 上；第三种类型叫 WithoutCache，指的是同步到了 HDD 但是 SSD 中已经没有缓存了。
2. 然后后台异步线程持续地将 SSD 数据同步到 HDD 上。
3. 随着 SSD 的持续写入，当存储空间达到阈值后，会按时间顺序删除距当前时间最久的数据，因为 SSD 的数据空间有限。
4. 副本可根据可用性要求灵活开启是否写入 SSD。
5. 从 HDD 读取的数据是不会回刷到 SSD 上的，防止缓存污染。



![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-18.jpeg)

图 2-14 SSD 新缓存架构细节优化

**细节优化**

介绍了具体实现之后，再来看一下细节优化。

1. 首先是关于日志段同步，就是刚才说到的 Segment，只同步 Inactive 的日志段，Inactive 指的是现在并没有在写的日志段，低成本解决数据一致性问题。
2. 其次是做同步限速优化，在 SSD 向 HDD 同步时是需要限速的，同时保护了两种设备，不会影响其他 IO 请求的处理。

## 3. 大规模集群管理优化

### 3.1 隔离策略

美团大数据平台的 Kafka 服务于多个业务，这些业务的 Topic 混布在一起的话，很有可能造成不同业务的不同 Topic 之间相互影响。此外，如果 Controller 节点同时承担数据读写请求，当负载明显变高时，Controller 可能无法及时控制类请求，例如元数据变更请求，最终可能会造成整个集群发生故障。

针对这些相互影响的问题，我们从业务、角色和优先级三个维度来做隔离优化。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-19.jpeg)

图 3-1 隔离优化

- 第一点是业务隔离，如图 3-1 所示，每一个大的业务会有一个独立的 Kafka 集群，比如外卖、到店、优选。
- 第二点是分角色隔离，这里 Kafka 的 Broker 和 Controller 以及它们依赖的组件 Zookeeper 是部署在不同机器上的，避免之间相互影响。
- 第三点是分优先级，有的业务 Topic 可用性等级特别高，那么我们就可以给它划分到 VIP 集群，给它更多的资源冗余去保证其可用性。

### 3.2 全链路监控

随着集群规模增长，集群管理碰到了一系列问题，主要包括两方面：

- Broker 端延时指标无法及时反应用户问题。

- - 随着请求量的增长，Kafka 当前提供的 Broker 端粒度的 TP99 甚至 TP999 延时指标都可能无法反应长尾延时。
  - Broker 端的延时指标不是端到端指标，可能无法反应用户的真实问题。

- 故障感知和处理不及时。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-20.jpeg)

图 3-2 全链路监控

针对这两个问题，我们采取的策略是全链路监控。全链路监控收集和监控 Kafka 核心组件的指标和日志。全链路监控架构如图 3-2 所示。当某一个客户端读写请求变慢时，我们通过全链路监控可以快速定位到具体慢在哪个环节，全链路指标监控如图 3-3 所示。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-21.jpeg)

图 3-3 全链路指标监控

图 3-4 是一个根据全链路指标定位请求瓶颈的示例，可以看出服务端 RemoteTime 占比最高，这说明耗时主要花费在数据复制。日志和指标的解析服务可以自动实时感知故障和慢节点，大部分的故障（内存、磁盘、Raid 卡以及网卡等）和慢节点都已经支持自动化处理，还有一类故障是计划外的故障，比如分区多个副本挂掉导致的不可用，迁移 Hang 住以及非预期的错误日志等，需要人工介入处理。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-22.jpeg)

图 3-4 全链路监控指标示例

### 3.3 服务生命周期管理

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-23.jpeg)

图 3-5 服务生命周期管理

美团线上 Kafka 的服务器规模在万级别，随着服务规模的增长，我们对服务和机器本身的管理，也在不断迭代。我们的自动化运维系统能够处理大部分的机器故障和服务慢节点，但对于机器和服务本身的管理是割裂的，导致存在两类问题：

1. 状态语义存在歧义，无法真实反映系统状态，往往需要借助日志和指标去找到真实系统是否健康或者异常。
2. 状态不全面，异常 Case 需人工介入处理，误操作风险极大。

为了解决这两类问题，我们引入了生命周期管理机制，确保能够真实反映系统状态。生命周期管理指的是从服务开始运行到机器报废停止服务的全流程管理，并且做到了服务状态和机器状态联动，无需人工同步变更。而且新的生命周期管理机制的状态变更由特定的自动化运维触发，禁止人工变更。

### 3.4 TOR容灾

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846700-24.jpeg)

图 3-6 TOR 容灾挑战

我们从工程实现的角度，归纳总结了当前主流图神经网络模型的基本范式，实现一套通用框架，以期涵盖多种 GNN 模型。以下按照图的类型（同质图、异质图和动态图）分别讨论。

![图片](meituan_Kafka%E5%9C%A8%E7%BE%8E%E5%9B%A2%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0%E7%9A%84%E5%AE%9E%E8%B7%B5.resource/640-1672584846701-25.jpeg)

图 3-7 TOR 容灾

TOR 容灾保证同一个分区的不同副本不在同一个 Rack 下，如图 3-7 所示，即使 Rack1 整个发生故障，也能保证所有分区可用。

## 4 未来展望

过去一段时间，我们围绕降低服务端的读写延迟做了大量的优化，但是在服务高可用方面，依然有一些工作需要完成。未来一段时间，我们会将重心放在提升鲁棒性和通过各种粒度的隔离机制缩小故障域。比如，让客户端主动对一些故障节点进行避让，在服务端通过多队列的方式隔离异常请求，支持服务端热下盘，网络层主动反压与限流等等。

另外，随着美团实时计算业务整体的发展，实时计算引擎（典型如 Flink）和流存储引擎（典型如 Kafka）混合部署的模式越来越难以满足业务的需求。因此，我们需要在保持当前成本不变的情况下对 Kafka 进行独立部署。这就意味着需要用更少的机器（在我们的业务模式下，用原来 1/4 的机器）来承载不变的业务流量。如何在保障服务稳定的情况下，用更少的机器扛起业务请求，也是我们面临的挑战之一。

最后，随着云原生趋势的来临，我们也在探索流存储服务的上云之路。

## 5 作者简介

海源、仕禄、肖恩、鸿洛、启帆、胡荣、李杰等，均来自美团数据科学与平台部。



**也许你还想看**

 **|** [基于SSD的Kafka应用层缓存架构设计与实现](http://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651756933&idx=1&sn=a23c294fe1873d6b2c50730e47eda608&chksm=bd1240c88a65c9de720b8568bf7cf90a365c1df45732a36493eb58cc1ff8cf8461cb4829f102&scene=21#wechat_redirect)

 **|** [美团技术团队博客：Kafka文件存储机制那些事](http://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=203083582&idx=1&sn=701022c664d42b54b55d43e6bc46056b&chksm=2f06167318719f65e90a8154a48e875f474c56e47b66aa6607f27c9ba6779ae062195720e6f6&scene=21#wechat_redirect)

 **|** [美团酒旅数据治理实践](http://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651761795&idx=1&sn=49a812ee8b9bf1cae5bd088c53772007&chksm=bd1275ce8a65fcd8cb960cf727125c8462022c39e50a314bc79a60737a75909b26791c37536e&scene=21#wechat_redirect)