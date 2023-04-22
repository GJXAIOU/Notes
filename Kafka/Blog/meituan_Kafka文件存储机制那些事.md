# Kafka文件存储机制那些事

## Kafka是什么

Kafka 最初由 Linkedin 公司开发，是一个分区、多副本、多订阅者、且基于 zookeeper 协调的分布式日志系统(也可以当做 MQ 系统)，常用于 web/nginx 日志、访问日志，消息服务等，Linkedin 于 2010 年贡献给了 Apache 基金会并成为顶级开源项目。

## 1.前言

一个商业化消息队列文件存储机制设计，是衡量其技术水平的关键指标之一。
下面将从 Kafka 文件存储机制和物理结构角度，分析 Kafka 如何实现高效文件存储，及实际应用效果。

## 2.Kafka文件存储机制

Kafka 部分名词解释如下：

- Broker：消息中间件处理结点，一个 Kafka 节点就是一个 broker，多个 broker 可以组成一个 Kafka 集群。
- Topic：一类消息，例如 page view 日志、click 日志等都可以以 topic 的形式存在，Kafka 集群能够同时负责多个 topic 的分发。
- Partition：topic 物理上的分组，一个 topic 可以分为多个 partition，每 个 partition 是一个有序的队列。
- Segment：partition 物理上由多个 segment 组成，下面 2.2 和 2.3 有详细说明。

分析过程分为以下 4 个步骤：

1. topic 中 partition 存储分布
2. partiton 中文件存储方式
3. partiton 中 segment 文件存储结构
4. 在 partition 中如何通过 offset 查找 message

通过上述 4 过程详细分析，我们就可以清楚认识到 kafka 文件存储机制的奥秘。

## 2.1 topic 中 partition 存储分布

假设实验环境中 Kafka 集群只有一个 broker，xxx/message-folder 为数据文件存储根目录，在 Kafka broker 中 server.properties 文件配置(参数`log.dirs=xxx/message-folder`)，例如创建 2 个 topic 名称分别为 report_push、launch_info, partitions 数量都为 partitions=4
存储路径和目录规则为：
xxx/message-folder

```
|--report_push-0
  |--report_push-1
  |--report_push-2
  |--report_push-3
  |--launch_info-0
  |--launch_info-1
  |--launch_info-2
  |--launch_info-3
```

在 Kafka 文件存储中，同一个 topic 下有多个不同 partition，每个 partition 为一个目录，partiton 命名规则为 topic 名称+有序序号，第一个 partiton 序号从 0 开始，序号最大值为 partitions 数量减 1。
如果是多 broker 分布情况，请参考 kafka 集群 partition 分布原理分析

## **2.2 partiton中文件存储方式**

下面示意图形象说明了 partition 中文件存储方式:
![图片](meituan_Kafka%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E6%9C%BA%E5%88%B6%E9%82%A3%E4%BA%9B%E4%BA%8B.resource/640.png)

图 1

- 每个 partion(目录)相当于一个巨型文件被平均分配到多个大小相等 segment(段)数据文件中。但每个段 segment file 消息数量不一定相等，这种特性方便 old segment file 快速被删除。
- 每个 partiton 只需要支持顺序读写就行了，segment 文件生命周期由服务端配置参数决定。

这样做的好处是快速删除无用文件，有效提高磁盘利用率。

## **2.3 partiton中segment文件存储结构**

读者从 2.2 节了解到 Kafka 文件系统 partition 存储方式，本节深入分析 partion 中 segment file 组成和物理结构。

- **segment file组成**：由 2 大部分组成，分别为 index file 和 data file，此 2 个文件一一对应，成对出现，后缀".index"和“.log”分别表示为 segment 索引文件、数据文件.
- **segment文件命名规则**：partion 全局的第一个 segment 从 0 开始，后续每个 segment 文件名为上一个全局 partion 的最大 offset(偏移 message 数)。数值最大为 64 位 long 大小，19 位数字字符长度，没有数字用 0 填充。

下面文件列表是笔者在 Kafka broker 上做的一个实验，创建一个 topicXXX 包含 1 partition，设置每个 segment 大小为 500MB,并启动 producer 向 Kafka broker 写入大量数据,如下图 2 所示 segment 文件列表形象说明了上述 2 个规则：
![图片](meituan_Kafka%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E6%9C%BA%E5%88%B6%E9%82%A3%E4%BA%9B%E4%BA%8B.resource/640-1678721441367-61.jpeg)

图 2

以上述图 2 中一对 segment file 文件为例，说明 segment 中 index<—->data file 对应关系物理结构如下：
![图片](meituan_Kafka%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E6%9C%BA%E5%88%B6%E9%82%A3%E4%BA%9B%E4%BA%8B.resource/640-1678721441367-62.jpeg)

图 3

上述图 3 中索引文件存储大量元数据，数据文件存储大量消息，索引文件中元数据指向对应数据文件中 message 的物理偏移地址。
其中以索引文件中元数据 3,497 为例，依次在数据文件中表示第 3 个 message(在全局 partiton 表示第 368772 个 message)、以及该消息的物理偏移地址为 497。

从上述图 3 了解到 segment data file 由许多 message 组成，下面详细说明 message 物理结构如下：
![图片](meituan_Kafka%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E6%9C%BA%E5%88%B6%E9%82%A3%E4%BA%9B%E4%BA%8B.resource/640-1678721441367-63.jpeg)

图 4

### 参数说明：

| 关键字              | 解释说明                                                     |
| ------------------- | ------------------------------------------------------------ |
| 8 byte offset       | 在parition(分区)内的每条消息都有一个有序的id号，这个id号被称为偏移(offset),它可以唯一确定每条消息在parition(分区)内的位置。即offset表示partiion的第多少message |
| 4 byte message size | message大小                                                  |
| 4 byte CRC32        | 用crc32校验message                                           |
| 1 byte “magic"      | 表示本次发布Kafka服务程序协议版本号                          |
| 1 byte “attributes" | 表示为独立版本、或标识压缩类型、或编码类型。                 |
| 4 byte key length   | 表示key的长度,当key为-1时，K byte key字段不填                |
| K byte key          | 可选                                                         |
| value bytes payload | 表示实际消息数据。                                           |

## **2.4 在partition中如何通过offset查找message**

例如读取 offset=368776 的 message，需要通过下面 2 个步骤查找。

1. 查找 segment file
   上述图 2 为例，其中 00000000000000000000.index 表示最开始的文件，起始偏移量(offset)为 0.第二个文件 00000000000000368769.index 的消息量起始偏移量为 368770 = 368769 + 1.同样，第三个文件 00000000000000737337.index 的起始偏移量为 737338=737337 + 1，其他后续文件依次类推，以起始偏移量命名并排序这些文件，只要根据 offset **二分查找**文件列表，就可以快速定位到具体文件。
   当 offset=368776 时定位到 00000000000000368769.index|log
2. 通过 segment file 查找 message
   通过第一步定位到 segment file，当 offset=368776 时，依次定位到 00000000000000368769.index 的元数据物理位置和 00000000000000368769.log 的物理偏移地址，然后再通过 00000000000000368769.log 顺序查找直到 offset=368776 为止。

从上述图 3 可知这样做的优点，segment index file 采取稀疏索引存储方式，它减少索引文件大小，通过 mmap 可以直接内存操作，稀疏索引为数据文件的每个对应 message 设置一个元数据指针,它比稠密索引节省了更多的存储空间，但查找起来需要消耗更多的时间。

## **3. Kafka文件存储机制–实际运行效果**

实验环境：

- Kafka 集群：由 2 台虚拟机组成
- cpu：4 核
- 物理内存：8GB
- 网卡：千兆网卡
- jvm heap: 4GB
- 详细 Kafka 服务端配置及其优化请参考：kafka server.properties 配置详解

![图片](meituan_Kafka%E6%96%87%E4%BB%B6%E5%AD%98%E5%82%A8%E6%9C%BA%E5%88%B6%E9%82%A3%E4%BA%9B%E4%BA%8B.resource/640-1678721441368-64.png)

图 5

从上述图 5 可以看出，Kafka 运行时很少有大量读磁盘的操作，主要是定期批量写磁盘操作，因此操作磁盘很高效。这跟 Kafka 文件存储中读写 message 的设计是息息相关的。Kafka 中读写 message 有如下特点:

写 message

- 消息从 java 堆转入 page cache(即物理内存)。
- 由异步线程刷盘,消息从 page cache 刷入磁盘。

读 message

- 消息直接从 page cache 转入 socket 发送出去。
- 当从 page cache 没有找到相应数据时，此时会产生磁盘 IO,从磁
  盘 Load 消息到 page cache,然后直接从 socket 发出去

## **4.总结**

Kafka 高效文件存储设计特点

- Kafka 把 topic 中一个 parition 大文件分成多个小文件段，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。
- 通过索引信息可以快速定位 message 和确定 response 的最大大小。
- 通过 index 元数据全部映射到 memory，可以避免 segment file 的 IO 磁盘操作。
- 通过索引文件稀疏存储，可以大幅降低 index 文件元数据占用空间大小。

## **参考**

1.Linux Page Cache 机制
2.Kafka 官方文档