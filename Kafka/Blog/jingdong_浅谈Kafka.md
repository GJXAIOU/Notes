# 浅谈kafka

## **导读**

当今大数据时代，高吞吐、高可靠成为了分布式系统中重要的指标。而Apache Kafka作为一个高性能、分布式、可扩展的消息队列系统，被越来越多的企业和开发者所关注和使用。 本文将介绍Kafka的基本概念，包括Kafka的架构、消息的存储和处理方式、Kafka的应用场景等，帮助读者快速了解Kafka的特点和优势。同时探讨Kafka的一些高级特性，如Kafka的配置、文件存储机制、分区 等，帮助读者更好地使用Kafka构建分布式系统和应用。

## 01 入门

在今年的敏捷团队建设中，我通过Suite执行器实现了一键自动化单元测试。Juint除了Suite执行器还有哪些执行器呢？由此我的Runner探索之旅开始了！

### 1.什么是kafka？

apache Kafka is a distributed streaming platform. What exactly dose that mean?

Apache Kafka 是消息引擎系统，也是一个分布式流处理平台（Distributed Streaming Platform）

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110688.png)

图1. kafka新特性



### **2. kafka全景图**

**![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110837.png)**

图2. kakfa基本脉络

### **3. kafka版本演进**

**![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110937.png)**

图3. kafka里程碑

### **4. kafka选型**

Apache Kafka：也称社区版 Kafka。优势在于迭代速度快，社区响应度高，使用它可以有更高的把控度；缺陷在于仅提供基础核心组件，缺失一些高级的特性。（如果仅仅需要一个消息引擎系统亦或是简单的流处理应用场景，同时需要对系统有较大把控度，那么推荐使用 Apache Kafka）；

Confluent Kafka ：Confluent 公司提供的 Kafka。优势在于集成了很多高级特性且由 Kafka 原班人马打造，质量上有保证；缺陷在于相关文档资料不全，普及率较低，没有太多可供参考的范例。（如果需要用到 Kafka 的一些高级特性，那么推荐使用 Confluent Kafka）；

CDH/HDP Kafka：大数据云公司提供的 Kafka，内嵌 Apache Kafka。优势在于操作简单，节省运维成本；缺陷在于把控度低，演进速度较慢。（如果需要快速地搭建消息引擎系统，或者需要搭建的是多框架构成的数据平台且 Kafka 只是其中一个组件，那么推荐使用这些大数据云公司提供的 Kafka）。



### **5. kafka基本概念**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110492.png)

图4. kafka基本概念



### **6. kafka基本结构**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110631.png)

图5. kafka集群概览



### **7. kafka集群结构**

**![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110456.png)**

图6. kafka集群结构



### **8. kafka应用场景**

![image-20230618174443807](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/image-20230618174443807.png)

图7. kafka应用场景



### **9. kafka队列模式**

**（1）点对点模式：**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110779.png)

图8. kafka队列模式-点对点

**（2）发布/订阅模式：**

**![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110399.png)**

图9. kafka队列模式-点对点

### **10. kafka构成角色**

#### **（1）broker：**

消息格式: 主题 - 分区 - 消息 、主题下的每条消息只会保存在某一个分区中，而不会在多个分区中被保存多份。

这样设计的原因是：不使用多topic做负载均衡，意义在于对业务屏蔽该逻辑。业务只需要对topic进行发送，指定负载均衡策略即可 同时 topic分区是实现负载均衡以及高吞吐量的关键。



Topic的创建流程如下：

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110920.png)

图10. kafka创建topic流程

#### **（2）Producer：**

发送消息流程

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174527467.png)

﻿图11. kafka发送消息流程

#### **（3）Consumer：**

Kafka消费者对象订阅主题并接收Kafka的消息，然后验证消息并保存结果。Kafka消费者是消费者组的一部分。一个消费者组里的消费者订阅的是同一个主题，每个消费者接收主题一部分分区的消息。消费者组的设计是对消费者进行的一个横向伸缩，用于解决消费者消费数据的速度跟不上生产者生产数据的速度的问题，通过增加消费者，让它们分担负载，分别处理部分分区的消息。

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174555215.png)

﻿﻿图12. kafka生产者写相同topic不同分区

#### **（4）Consumer Group：****它是kafka提供的具有可扩展且可容错的消费者机制。**

**特性：**

- Consumer Group下可以有一个或多个 Consumer实例；
- 在一个Katka集群中，Group ID标识唯一的一个Consumer Group;
- Consumer Group 下所有实例订阅的主题的单个分区，只能分配给组内的 某个Consumer实例消费。

**Consumer Group 两大模型：**

- 如果所有实例都属于同一个Group，那么它实现的是消息队列模型；
- 如果所有实例分别属于不同的GrouD，那么它实现的就是发布/订阅模型。

### **11. Kafka工作流程：**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110748.png)

﻿图13. kafka工作流程

### **12. Kafka常用命令：**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110582.png)

﻿图14. kafka常用命令

## **02** **进阶**

### 2.1 Kafka 的文件存储机制

**1. log**

![image-20230618174730426](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/image-20230618174730426.png)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)图15. log在磁盘的写入方式

**2. 分片/索引**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110621.png)

图16. log文件在磁盘的存储方式

**3. index/log**

**![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110956.png)**

图17. log索引在磁盘的存储方式

### **2.2  kafka如何支持百万QPS？**

**1. 顺序读写：生产者写入数据和消费者读取数据都是顺序读写的。**

**![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110685.png)**

图18. 不同环境下的写入对比

#### **2. Batch Data（数据批量处理）：**

当消费者（consumer）需要消费数据时，首先想到的是消费者需要一条，kafka发送一条，消费者再要一条kafka再发送一条。但实际上 Kafka 不是这样做的，Kafka 耍小聪明了。Kafka 把所有的消息都存放在一个一个的文件中，当消费者需要数据的时候 Kafka 直接把文件发送给消费者。比如说100万条消息放在一个文件中可能是10M的数据量，如果消费者和Kafka之间网络良好，10MB大概1秒就能发送完，既100万TPS，Kafka每秒处理了10万条消息。

**3. MMAP（内存映射文件）：**

MMAP也就是内存映射文件，在64位操作系统中一般可以表示 20G 的数据文件，它的工作原理是直接利用操作系统的 Page 来实现文件到物理内存的直接映射，完成映射之后对物理内存的操作会被同步到硬盘上。

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110556.png)

﻿﻿图19. mmap的调用流程

通过MMAP技术进程可以像读写硬盘一样读写内存（逻辑内存），不必关心内存的大小，因为有虚拟内存兜底。这种方式可以获取很大的I/O提升，省去了用户空间到内核空间复制的开销。也有一个很明显的缺陷，写到MMAP中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用 flush 的时候才把数据真正的写到硬盘。

#### **4. Zero Copy（零拷贝）：**

如果不使用零拷贝技术，消费者（consumer）从Kafka消费数据，Kafka从磁盘读数据然后发送到网络上去，数据一共发生了四次传输的过程。其中两次是 DMA 的传输，另外两次，则是通过 CPU 控制的传输。

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174843187.png)

﻿﻿图20. 数据的4次传输

第一次传输：从硬盘上将数据读到操作系统内核的缓冲区里，这个传输是通过 DMA 搬运的；

第二次传输：从内核缓冲区里面的数据复制到分配的内存里面，这个传输是通过 CPU 搬运的；

第三次传输：从分配的内存里面再写到操作系统的 Socket 的缓冲区里面去，这个传输是由 CPU 搬运的；

第四次传输：从 Socket 的缓冲区里面写到网卡的缓冲区里面去，这个传输是通过 DMA 搬运的。

实际上在kafka中只进行了两次数据传输，如下图：

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110747.png)

﻿﻿图21. 数据的2次传输

第一次传输：通过 DMA从硬盘直接读到操作系统内核的读缓冲区里面；

第二次传输：根据 Socket 的描述符信息直接从读缓冲区里面写入到网卡的缓冲区里面。

可以看到同一份数据的传输次数从四次变成了两次，并且没有通过 CPU 来进行数据搬运，所有的数据都是通过 DMA 来进行传输的。没有在内存层面去复制（Copy）数据，这个方法称之为零拷贝（Zero-Copy）。

无论传输数据量的大小，传输同样的数据使用了零拷贝能够缩短 65%的时间，大幅度提升了机器传输数据的吞吐量，这也是Kafka能够支持百万TPS的一个重要原因。

### **2.3 压缩**

**1.** 特性：节省网络传输带宽以及 Kafka Broker 端的磁盘占用。

#### 2. 生产者配置

```
compression.typeProperties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("acks", "all");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
// 开启GZIP压缩
props.put("compression.type", "gzip");
Producerproducer = new KafkaProducer<>(props)
```

**3. broker开启压缩：**

Broker 端也有一个参数叫 compression.type 默认值为none，这意味着发送的消息是未压缩的。否则，您指定支持的类型：gzip、snappy、lz4或zstd。Producer 端压缩、Broker 端保持、Consumer 端解压缩。

#### **4. broker何时压缩：**

**情况一：**

Broker 端指定了和 Producer 端不同的压缩算法。（风险：可能会发生预料之外的压缩 / 解压缩操作，表现为 Broker 端 CPU 使用率飙升）

想象一个对话：

Producer 说：“我要使用 GZIP 进行压缩。”

Broker 说：“不要，我这边接收的消息必须使用配置的 lz4 进行压缩。”

**情况二：**

Broker 端发生了消息格式转换 （风险：涉及额外压缩/解压缩，且 Kafka 丧失 Zero Copy 特性）

Kafka 共有两大类消息格式，社区分别称之为 V1 版本和 V2 版本。

为了兼容老版本的格式，Broker 端会对新版本消息执行向老版本格式的转换。这个过程中会涉及消息的解压缩和重新压缩。

#### **5. 消息何时解压缩：**

Consumer：收到到压缩过的消息会解压缩还原成之前的消息。

broker：收到producer的消息 压缩算法和自己的不一致/兼容新老版本的消息格式。

#### **6. 压缩算法对比：**

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110581.png)

﻿﻿图22. 不同压缩算法的速度对比

### **2.4** **Exactly-Once（ACK应答机制）**

**1. At Least Once**

最少发送一次，Ack级别为-1，保证数据不丢失。

**2. At Most Once**

最多发送一次，Ack级别为1，保证数据不重复。

**3. 幂等性**

保证producer发送的数据在broker只持久化一条。

**4. Exactly Once（0.11版本）**

At Least Once + 幂等性 = Exactly Once

要启用幂等性，只需要将Producer的参数中 enable.idompotence设置为 true即可。 Kafka的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。

### 2.5 **producer如何获取metadata**

1. 在创建KafkaProducer实例时 第一步：生产者应用会在后台创建并启动一个名为Sender的线程。
2. 该Sender线程开始运行时，首先会创建与Broker的连接。第二步：此时不知道要连接哪个Broker，kafka会通过METADATA请求获取集群的元数据，连接所有的Broker。
3. Producer 通过 metadata.max.age.ms定期更新元数据，在连接多个broker的情况下，producer的InFlightsRequests中维护着每个broker的等待回复消息的队列，等待数量越少说明broker处理速度越快，负载越小，就会发到哪个broker上。

### **2.6** kafka真的会丢消息吗

#### kafka最优配置

**1. Producer：**

如果是Java客户端 建议使用 producer.send(msg, callback) ，callback（回调）它能准确地告知消息是否真的提交成功了。

设置 acks = all。acks 是 Producer 的参数，如果设置成 all，需要所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。

设置 retries 为一个较大的值。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。

#### **2. Consumer：**

消息消费完成再提交。Consumer 端有个参数 enable.auto.commit，最好把它设置成 false，并采用手动提交位移的方式。

#### **3. broker ：**

设置 unclean.leader.election.enable = false。它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。

设置 replication.factor >= 3,目前防止消息丢失的主要机制就是冗余。

设置 min.insync.replicas > 1,控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。确保 replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。

### **2.7** kafka Replica![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110734.png)

图23. kafka副本机制

本质就是一个只能追加写消息的提交日志。根据 Kafka 副本机制的定义，同一个分区下的所有副本保存有相同的消息序列，这些副本分散保存在不同的 Broker 上，从而能够对抗部分 Broker 宕机带来的数据不可用。

#### **1. 三个特性：**

第一，在 Kafka 中，副本分成两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本。

第二，Kafka 的副本机制比其他分布式系统要更严格一些。在 Kafka 中，追随者副本是不对外提供服务的。这就是说，任何一个追随者副本都不能响应消费者和生产者的读写请求。所有的请求都必须由领导者副本来处理，或者说，所有的读写请求都必须发往领导者副本所在的 Broker，由该 Broker 负责处理。追随者副本不处理客户端请求，它唯一的任务就是从领导者副本异步拉取消息，并写入到自己的提交日志中，从而实现与领导者副本的同步。

第三，当领导者副本挂掉了，或者说领导者副本所在的 Broker 宕机时，Kafka 依托于监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。老 Leader 副本重启回来后，只能作为追随者副本加入到集群中。

#### **2. 意义：**

#### **方便实现“Read-your-writes”**

（1）当使用生产者API向Kafka成功写入消息后，马上使用消息者API去读取刚才生产的消息。

（2）如果允许追随者副本对外提供服务，由于副本同步是异步的，就可能因为数据同步时间差，从而使客户端看不到最新写入的消息。

**方便实现单调读（Monotonic Reads）**

（1）单调读：对于一个消费者用户而言，在多处消息消息时，他不会看到某条消息一会存在，一会不存在。

（2）如果允许追随者副本提供读服务，由于消息是异步的，则多个追随者副本的状态可能不一致。若客户端每次命中的副本不同，就可能出现一条消息一会看到，一会看不到。

### **2.8** ISR（In-Sync Replica Set）LEO&HW 机制

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618175228972.png)

图24. afka的ISR机制

**HW(High Watermark)是所有副本中最小的LEO。**

比如：一个分区有3个副本，一个leader，2个follower。producer向leader写了10条消息，follower1从leader处拷贝了5条消息，follower2从leader处拷贝了3条消息，那么leader副本的LEO就是10，HW=3；follower1副本的LEO就是5

HW作用：保证消费数据的一致性和副本数据的一致性 通过HW机制。leader处的HW要等所有follower LEO都越过了才会前移

ISR：所有与leader副本保持一定程度同步的副本（包括leader副本在内）组成ISR（In-Sync Replicas）

**1. Follower故障：**

当follower挂掉之后，会被踢出ISR；

当follower恢复后，会读取本地磁盘记录的HW，然后截掉HW之后的部分，从HW开始从leader继续同步数据，当该follower的LEO大于等于该partition的HW的时候，就是它追上leader的时候，会被重新加入到ISR中。

#### **2. Leader故障：**

当leader故障之后，会从follower中选出新的leader，为保证多个副本之间的数据一致性，其余的follower会将各自HW之后的部分截掉（新leader如果没有那部分数据 follower就会截掉造成数据丢失），重新从leader开始同步数据，但是只能保证副本之间的数据一致性，并不能保证数据不重复或丢失。

### **2.9** Consumer分区分配策略

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618175306529.png)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)图25. 消费者分区策略

**自定义分区策略：**

需要显式地配置生产者端的参数partitioner.class。这个参数该怎么设定呢？

方法很简单，在编写生产者程序时，可以编写一个具体的类实现org.apache.kafka.clients.producer.Partitioner接口。这个接口也很简单，只定义了两个方法：partition()和close()，通常只需要实现最重要的 partition 方法。

```
int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster){
List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
//随机
//return ThreadLocalRandom.current().nextInt(partitions.size());
//按消息键保序策略
//return Math.abs(key.hashCode()) % partitions.size();
//指定条件
return partitions.stream().filter(Predicate(指定条件))).map(PartitionInfo::partition).findAny().get();
}
```

### 2.10 kafka中一个不为人知的topic

#### **consumer_offsets：**

老版本的Kafka会把位移信息保存在Zk中 ，但zk不适用于高频的写操作，这令zk集群性能严重下降，在新版本中将位移数据作为一条条普通的Kafka消息，提交至内部主题（_consumer_offsets）中保存，实现高持久性和高频写操作。

**位移主题每条消息内容格式：Group ID，主题名，分区号**

当Kafka集群中的第一个Consumer程序启动时，Kafka会自动创建位移主题。也可以手动创建 分区数依赖于Broker端的offsets.topic.num.partitions的取值，默认为50 副本数依赖于Broker端的offsets.topic.replication.factor的取值，默认为3。

#### **思考：**

只要 Consumer 一直启动着，它就会无限期地向位移主题写入消息，就算没有新消息进来 也会通过定时任务重复写相同位移 最终撑爆磁盘？

Kafka 提供了专门的后台线程定期地巡检待 Compact 的主题，看看是否存在满足条件的可删除数据，这个后台线程叫 Log Cleaner，对相同的key只保留最新的一条消息。

### **2.11** **Consumer Group Rebalance**

#### **1. 术语简介：**

Rebalance ：就是让一个 Consumer Group 下所有的 Consumer 实例就如何消费订阅主题的所有分区达成共识的过程。

Coordinator：它专门为 Consumer Group 服务，负责为 Group 执行 Rebalance 以及提供位移管理和组成员管理等。

Consumer 端应用程序在提交位移时，其实是向 Coordinator 所在的 Broker 提交位移。同样地，当 Consumer 应用启动时，也是向 Coordinator 所在的 Broker 发送各种请求，然后由 Coordinator 负责执行消费者组的注册、成员管理记录等元数据管理操作。

如何确定Coordinator位置 ：partitionId=Math.abs(groupId.hashCode() % offsetsTopicPartitionCount) 比如（abs(627841412 % 50)=12 Coordinator就在 partitionId=12的Leader 副本所在的 Broker）。

#### **2. Rebalance的危害：**

Rebalance 影响 Consumer 端 TPS 这期间不会工作。

Rebalance 很慢 Consumer越多 Rebalance时间越长。

Rebalance 效率不高 需要所有成员参与。

#### **3. 触发 Rebalance场景：**

组成员数量发生变化。

订阅主题数量发生变化。

订阅主题的分区数发生变化。

#### **4. 如何避免 Rebalance：**

#### 设置 session.timeout.ms = 15s （session连接时间 默认10）

设置 heartbeat.interval.ms = 2s（心跳时间）

max.poll.interval.ms （取决你一批消息处理时长 默认5分钟）

要保证 Consumer 实例在被判定为“dead”之前，能够发送至少 3 轮的心跳请求，即 session.timeout.ms >= 3 * heartbeat.interval.ms。

### **2.12** **kafka拦截器**

Kafka 拦截器分为生产者拦截器和消费者拦截器，可以应用于包括客户端监控、端到端系统性能检测、消息审计等多种功能在内的场景。

例：生产者Interceptor

![图片](jingdong_%E6%B5%85%E8%B0%88Kafka.resource/640-20230618174110852.png)图26. kafka拦截器部分代码

 

## **总结**

理解，首先 MCube 会依据模板缓存状态判断是否需要网络获取最新模板，当获取到模板后进行模板加载，加载阶段会将产物转换为视图树的结构，转换完成后将通过表达式引擎解析表达式并取得正确的值，通过事件解析引擎解析用户自定义事件并完成事件的绑定，完成解析赋值以及事件绑定后进行视图的渲染，最终将目标页面展示到屏幕。

总的来说，Kafka作为一款高性能、可靠、可扩展的分布式消息队列系统，在使用Kafka时，可以根据实际需求和场景进行配置和调优，以达到更好的性能和可靠性。同时，Kafka的生态系统也在不断壮大，例如Kafka Connect和Kafka Streams等技术可以帮助开发者更好地集成和处理数据。因此，建议读者深入学习和掌握Kafka的相关技术和工具，以便更好地应用Kafka构建分布式系统和应用。

*部分图文资料地址 ：*

*-- 极客时间 Kafka 核心技术与实战*

*--google*