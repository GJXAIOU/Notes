# [Kafka学习笔记](https://my.oschina.net/jallenkwong/blog/4449224)

https://www.bilibili.com/video/BV1a4411B7V9?from=search&seid=9190821010667516360

[本文用到的源码](https://gitee.com/jallenkwong/LearnKafka)

**Kafka学习资料** 链接：https://pan.baidu.com/s/1oHYCvHZ4Uanll1Bj3v-3Hw 提取码：5afq

## 一、Kafka入门

[Kafka官网主页](http://kafka.apache.org/) 和[Kafka官方文档](http://kafka.apache.org/documentation/)

### （一）定义

Kafka 是一个**分布式**的基于**发布/订阅模式**的**消息队列**（Message Queue），主要应用于大数据实时处理领域。

### （二）消息队列

- 传统消息队列的应用场景

![img](Kafka入门使用.resource/01.png)

使用消息队列的好处

- 解耦（类似 Spring 的 IOC）

    允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

- 可恢复性

    系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

- 缓冲

    有助于控制和优化数据流经过系统的速度， 解决生产消息和消费消息的处理速度不一致的情况（更多是生产大于消费）。

- 灵活性 & 峰值处理能力（削峰）

    在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

- 异步通信

    很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。

### （三）消费模式

消息队列的两种模式

#### 点对点模式

一对一，消费者需要**主动拉取**数据，消息收到后消息清除。

消息生产者生产消息发送到 Queue 中，然后消息消费者从 Queue 中取出并且消费消息。消息被消费以后， Queue  中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue 支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。

![img](Kafka入门使用.resource/02.png)

#### 发布/订阅模式

一对多，消费者消费数据之后不会清除消息

消息生产者（发布）将消息发布到 topic 中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到 topic 的消息会被所有订阅者消费。

这类分为两种：一种是消费者主动拉取消息（如 kafka），另一种是队列主动推送给消费者的。

![img](Kafka入门使用.resource/03.png)

### （四）基础架构

![img](Kafka入门使用.resource/04.png)

- Producer ： 消息生产者，就是向 Kafka broker 发消息的客户端；

- Consumer： 消息消费者，向 Kafka broker 取消息的客户端；

- Consumer Group （CG）： 消费者组，由多个 consumer 组成。 **消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响**。 所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。

- Broker：一台 Kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多个 topic。

- Topic： 话题，可以理解为一个队列， 生产者和消费者面向的都是一个 topic；

- Partition： 为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列；

- Replica： 副本（Replication），为保证集群中的某个节点发生故障时， 该节点上的 partition 数据不丢失，且 Kafka 仍然能够继续工作， Kafka 提供了副本机制，一个 topic 的每个分区都有若干个副本，一个 leader 和若干个 follower。

- Leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是 leader。

- Follower： 每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据的同步。 **leader 发生故障时，某个 Follower 会成为新的 leader。**

某一个分区只能被某个消费者组里面的某一个消费者消费。

## 二、安装&启动&关闭

以下基于 Windows 进行，Linux 版本的差别不大，运行脚本文件后缀从`bat`改为`sh`，配置路径改用 Unix 风格的；

[官网安装教程](http://kafka.apache.org/0110/documentation/#quickstart)

- 步骤一：下载安装包

    注意：下载的时候不要下载名称带 src 的，这种的需要编译一下才行，可以直接下载：https://downloads.apache.org/kafka/3.0.0/

    Kafka 主目录文件位置为：`D:\Apache\Kafka\kafka-3.0.0`【下面使用  KAFKA_HOME 表示】

- 步骤二：启动 server

    Kafka 用到 ZooKeeper 功能，所以要预先运行 ZooKeeper，但是最新版本的 Kafka 已经内置 Zookeeper；这里使用内置的 Zookeeper；

    - 修改 `config\zookeeper.properties`，修改增加以下内容：

        ```properties
        ## 该内置 Zookeeper 和单独下载的 Zookeeper 配置路径一直，便于查找
        dataDir=D:\Apache\Zookeeper\Data
        dataLogDir=D:\Apache\Zookeeper\Logs
        
        # the directory where the snapshot is stored.
        dataDir=lZookeeper
        dataLogDir=/home/GJXAIOU/kafka_2.13-3.0.0/Data/Zookeeper
        ```

    - 修改 `config\server.properties`，修改一下内容：

        ```properties
        log.dir=D:\Apache\Kafka\Data
        log.dirs=D:\Apache\Kafka\Data
        ```

    - 启动 cmd，工作目录切换到`%KAFKA_HOME%`，执行命令行：

        ```shell
        ## 先启动 Zookeeper
        start bin\windows\zookeeper-server-start.bat config\zookeeper.properties
        start bin\windows\kafka-server-start.bat config\server.properties
        ```

- 可写一脚本，一键启动

- 关闭服务，`bin\windows\kafka-server-stop.bat`和`bin\windows\zookeeper-server-stop.bat`。

------

TODO:**一个问题**，通过`kafka-server-stop.bat`或右上角关闭按钮来关闭 Kafka 服务后，马上下次再启动 Kafka，抛出异常，说某文件被占用，需清空`log.dirs`目录下文件，才能重启 Kafka。

```
[2020-07-21 21:43:26,755] ERROR There was an error in one of the threads during logs loading: java.nio.file.FileSystemException: C:\Kafka\data\kafka-logs-0\my-replicated-topic-0\00000000000000000000.timeindex: 另一个程序正在使用此文件，进程无法访问。
 (kafka.log.LogManager)
...
```

参阅网络，这可能是在 windows 下的一个 Bug，没有更好的解决方案，暂时写个 py 脚本用来对 kafka 的 log 文件进行删除。下次启动 kafka，先运行这个删除脚本吧。

**好消息**，当你成功启动 kafka，然后在对应的命令行窗口用`Ctrl + C`结束 Kakfa，下次不用清理 kafka 日 志，也能正常启动。

如果一直提示找不到 `java.nio.file.FileSystemException` 则删除按照盘符下面的 `temp\kafka-logs` 文件夹；

如果还有有问题，则先关闭 kafka 然后关闭 zookeeper，在启动即可；

如果提示 `Exception in thread "main" joptsimple.UnrecognizedOptionException: zookeeper is not a recognized option` 则先

#### Step 3: Create a topic

- 用单一 partition 和单一 replica 创建一个名为`test`的 topic：

```bat
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
bin\windows\kafka-topics.bat --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```

- 查看已创建的 topic，也就刚才创建的名为`test`的 topic：

```bat
bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

或者，你可配置你的 broker 去自动创建未曾发布过的 topic，代替手动创建 topic

#### Step 4: Send some messages

运行 producer，然后输入几行文本，发至服务器：

```bat
bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test
>hello, kafka.
>what a nice day!
>to be or not to be. that' s a question.
```

请勿关闭窗口，下面步骤需要用到

#### Step 5: Start a consumer

运行 consumer，将[Step 4](https://my.oschina.net/jallenkwong/blog/4449224#)中输入的几行句子，标准输出。

```bat
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
hello, kafka.
what a nice day!
to be or not to be. that' s a question.
```

若你另启 cmd，执行命令行`bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning`来运行 consumer，然后在[Step 4](https://my.oschina.net/jallenkwong/blog/4449224#)中 producer 窗口输入一行句子，如`I must admit, I can't help but feel a twinge of envy.`，两个 consumer 也会同时输出`I must admit, I can't help but feel a twinge of envy.`。

#### Step 6: Setting up a multi-broker cluster

目前为止，我们仅作为一个单一 broker，这不好玩。让我们弄个有三个节点的集群来玩玩。

- 首先，在`%KAFKA%\config\server.properties`的基础上创建两个副本`server-1.properties`和`server-2.properties`。

```bat
copy config\server.properties config\server-1.properties
copy config\server.properties config\server-2.properties
```

- 打开副本，编辑如下属性

```properties
#config/server-1.properties:
broker.id=1
listeners=PLAINTEXT://127.0.0.1:9093
log.dir=C:\\Kafka\\data\\kafka-logs-1
 
#config/server-2.properties:
broker.id=2
listeners=PLAINTEXT://127.0.0.1:9094
log.dir=C:\\Kafka\\data\\kafka-logs-2
```

这个`broker.id`属性是集群中每个节点的唯一永久的名称。

我们必须重写端口和日志目录，只是因为我们在同一台机器上运行它们，并且我们希望阻止 brokers 试图在同一个端口上注册或覆盖彼此的数据。

- 我们已经启动了 Zookeeper 和我们的单个节点，所以我们只需要启动两个新节点：

```bat
start bin\windows\kafka-server-start.bat config\server-1.properties
start bin\windows\kafka-server-start.bat config\server-2.properties
```

- 创建一个 replication-factor 为 3 的 topic:

```bat
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

- OK，现在我们有了一个集群，但是我们怎么知道哪个 broker 在做什么呢？那就运行`describe topics`命令：

```bat
bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181 --topic my-replicated-topic

Topic:my-replicated-topic       PartitionCount:1        ReplicationFactor:3
Configs:
		Topic: my-replicated-topic      Partition: 0    Leader: 0       
		Replicas: 0,1,2        Isr: 0,1,2
```

- 以下是输出的说明。第一行给出所有 Partition 的摘要，每一行提供有关一个 Partition 的信息。因为这个 Topic 只有一个 Partition，所以只有一行。
    - "leader"是负责给定 Partition 的所有读写的节点。每个节点都可能成为 Partition 随机选择的 leader。
    - "replicas"是复制此 Partition 日志的节点列表，无论它们是 leader 还是当前处于存活状态。
    - "isr"是一组 "in-sync" replicas。这是 replicas 列表的一个子集，它当前处于存活状态，并补充 leader。

注意，**在我的示例中，node 0是Topic唯一Partition的leader**。（下面操作需要用到）

- 让我们为我们的新 Topic 发布一些信息：

```bat
bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic my-replicated-topic

>There's sadness in your eyes, I don't want to say goodbye to you.
>Love is a big illusion, I should try to forget, but there's something left in m
y head.
>
```

- 让我们接收刚刚发布的信息吧！

```bat
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic

There's sadness in your eyes, I don't want to say goodbye to you.
Love is a big illusion, I should try to forget, but there's something left in my head.
```

- 让我们测试一下容错性，由上文可知，Broker 0 身为 leader，因此，让我们干掉它吧：
    - 先找出 Broker 0 的进程 pid。
    - 杀掉 Broker 0 的进程。

```bat
wmic process where "caption='java.exe' and commandline like '%server.properties%'" get processid,caption
Caption   ProcessId
java.exe  7528

taskkill /pid 7528 /f
成功: 已终止 PID 为 7528 的进程。
```

- 原 leader 已被替换成它的 flowers 中的其中一个，并且 node 0 不在 in-sync replica 集合当中。

```bat
bin\windows\kafka-topics.bat --describe --zookeeper localhost:2181 --topic my-replicated-topic

Topic:my-replicated-topic       PartitionCount:1        ReplicationFactor:3
Configs:
		Topic: my-replicated-topic      Partition: 0    Leader: 1       Replicas: 0,1,2 Isr: 1,2
```

- 尽管原 leader 已逝，当原来消息依然可以接收。（注意，参数`--bootstrap-server localhost:9093`，而不是`--bootstrap-server localhost:9092`）

```bat
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9093 --from-beginning --topic my-replicated-topic

There's sadness in your eyes, I don't want to say goodbye to you.
Love is a big illusion, I should try to forget, but there's something left in my head.
I don't forget the way your kissing, the feeling 's so strong which is lasting for so long.
```

### server.properties一瞥

```properties
#broker 的全局唯一编号，不能重复
broker.id=0
#删除 topic 功能使能
delete.topic.enable=true
#处理网络请求的线程数量
num.network.threads=3
#用来处理磁盘 IO 的现成数量
num.io.threads=8
#发送套接字的缓冲区大小
socket.send.buffer.bytes=102400
#接收套接字的缓冲区大小
socket.receive.buffer.bytes=102400
#请求套接字的缓冲区大小
socket.request.max.bytes=104857600
#kafka 运行日志存放的路径
log.dirs=/opt/module/kafka/logs
#topic 在当前 broker 上的分区个数
num.partitions=1
#用来恢复和清理 data 下数据的线程数量
num.recovery.threads.per.data.dir=1
#segment 文件保留的最长时间，超时将被删除
log.retention.hours=168
#配置连接 Zookeeper 集群地址
zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181
```

## 07.Kafka入门_命令行操作Topic增删查

### 查看当前服务器中的所有 topic

```bat
bin\windows\kafka-topics.bat --list --zookeeper localhost:2181
```

### 创建 topic

```
bin\windows\kafka-topics.bat --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

选项说明：

- --topic 定义 topic 名
- --replication-factor 定义副本数
- --partitions 定义分区数

> 为了实现扩展性，一个非常大的 topic 可以分布到多个 broker（即服务器）上，一个 topic 可以分为多个 partition，每个 partition 是一个有序的队列；
>
> a broker = a kafka server a broker can contain N topic a topic can contain N partition a broker can contain a part of a topic (a broker can contain M(N>M) partition)

### 删除 topic

```bat
bin\windows\kafka-topics.bat --zookeeper localhost:2181 --delete --topic my-replicated-topic
```

需要 server.properties 中设置 `delete.topic.enable=true` 否则只是标记删除。

### 查看某个 Topic 的详情

```bat
bin\windows\kafka-topics.bat --zookeeper localhost:2181 --describe --topic first
```

### 修改分区数

```bat
bin\windows\kafka-topics.bat --zookeeper localhost:2181 --alter --topic first --partitions 6
```

## 08.Kafka入门_命令行控制台生产者消费者测试

### 发送消息

```bat
bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test

>hello, kafka.
>what a nice day!
>to be or not to be. that' s a question.
```

### 消费消息

```bat
bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning

hello, kafka.
what a nice day!
to be or not to be. that' s a question.
```

- --from-beginning： 会把主题中以往所有的数据都读取出来。

## 09.Kafka入门_数据日志分离

略

## 10.Kafka入门_回顾

略

## 11.Kafka高级_工作流程

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/05.png)

Kafka 中消息是以 topic 进行分类的， producer 生产消息，consumer 消费消息，都是面向 topic 的。(从命令行操作看出)

```bat
bin\windows\kafka-console-producer.bat --broker-list localhost:9092 --topic test

bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning
```

topic 是逻辑上的概念，而 partition 是物理上的概念，每个 partition 对应于一个 log 文件，该 log 文件中存储的就是 producer 生产的数据。（topic = N partition，partition = log）

Producer 生产的数据会被不断追加到该 log 文件末端，且每条数据都有自己的 offset。 consumer 组中的每个 consumer， 都会实时记录自己消费到了哪个 offset，以便出错恢复时，从上次的位置继续消费。（producer -> log with offset -> consumer(s)）

## 12.Kafka高级_文件存储

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/06.png)

由于生产者生产的消息会不断追加到 log 文件末尾， 为防止 log 文件过大导致数据定位效率低下， Kafka 采取了**分片**和**索引**机制，将每个 partition 分为多个 segment。

每个 segment 对应两个文件——“.index”文件和“.log”文件。 这些文件位于一个文件夹下， 该文件夹的命名规则为： topic 名称+分区序号。例如， first 这个 topic 有三个分区，则其对应的文件夹为 first-0,first-1,first-2。

```
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```

index 和 log 文件以当前 segment 的第一条消息的 offset 命名。下图为 index 文件和 log 文件的结构示意图。

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/07.png)

**“.index”文件存储大量的索引信息，“.log”文件存储大量的数据**，索引文件中的元数据指向对应数据文件中 message 的物理偏移地址。

> segment 英 [ˈseɡmənt , seɡˈment] 美 [ˈseɡmənt , seɡˈment]
> n.部分;份;片;段;(柑橘、柠檬等的)瓣;弓形;圆缺 v.分割;划分

## 13.Kafka高级_生产者分区策略

### 分区的原因

1. **方便在集群中扩展**，每个 Partition 可以通过调整以适应它所在的机器，而一个 topic 又可以有多个 Partition 组成，因此整个集群就可以适应适合的数据了；
2. **可以提高并发**，因为可以以 Partition 为单位读写了。（联想到 ConcurrentHashMap 在高并发环境下读写效率比 HashTable 的高效）

### 分区的原则

我们需要将 producer 发送的数据封装成一个 `ProducerRecord` 对象。

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/08.png)

1. 指明 partition 的情况下，直接将指明的值直接作为 partiton 值；
2. 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值；
3. 既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。

## 14.Kafka高级_生产者ISR

为保证 producer 发送的数据，能可靠的发送到指定的 topic， topic 的每个 partition 收到 producer 发送的数据后，都需要向 producer 发送 ack（acknowledgement 确认收到），如果 producer 收到 ack， 就会进行下一轮的发送，否则重新发送数据。

> acknowledgement 英 [əkˈnɒlɪdʒmənt] 美 [əkˈnɑːlɪdʒmənt]
> n.(对事实、现实、存在的)承认;感谢;谢礼;**收件复函**

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/09.png)

**何时发送ack？**

确保有 follower 与 leader 同步完成，leader 再发送 ack，这样才能保证 leader 挂掉之后，能在 follower 中选举出新的 leader。

------

**多少个follower同步完成之后发送ack？**

1. 半数以上的 follower 同步完成，即可发送 ack 继续发送重新发送
2. 全部的 follower 同步完成，才可以发送 ack

### 副本数据同步策略

| 序号 | 方案                          | 优点                                                         | 缺点                                                         |
| ---- | ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | 半数以上完成同步， 就发送 ack | 延迟低                                                       | 选举新的 leader 时，容忍 n 台节点的故障，需要 2n+1 个副本。（如果集群有2n+1台机器，选举leader的时候至少需要半数以上即n+1台机器投票，那么能容忍的故障，最多就是n台机器发生故障）容错率：1/2 |
| 2    | 全部完成同步，才发送ack       | 选举新的 leader 时， 容忍 n 台节点的故障，需要 n+1 个副本（如果集群有n+1台机器，选举leader的时候只要有一个副本就可以了）容错率：1 | 延迟高                                                       |

Kafka 选择了第二种方案，原因如下：

1. 同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1 个副本，而 Kafka 的每个分区都有大量的数据， 第一种方案会造成大量数据的冗余。
2. 虽然第二种方案的网络延迟会比较高，但网络延迟对 Kafka 的影响较小。

### ISR

采用第二种方案之后，设想以下情景： leader 收到数据，所有 follower 都开始同步数据，但有一个 follower，因为某种故障，迟迟不能与 leader 进行同步，那 leader 就要一直等下去，直到它完成同步，才能发送 ack。这个问题怎么解决呢？

Leader 维护了一个动态的 **in-sync replica set** (ISR)，意为和 leader 保持同步的 follower 集合。当 ISR 中的 follower 完成数据的同步之后，就会给 leader 发送 ack。如果 follower 长时间未向 leader 同步数据，则该 follower 将被踢出 ISR，该时间阈值由`replica.lag.time.max.ms`参数设定。 Leader 发生故障之后，就会从 ISR 中选举新的 leader。

> **replica.lag.time.max.ms**
>
> **DESCRIPTION**: If a follower hasn't sent any fetch requests or hasn't consumed up to the leaders log end offset for at least this time, the leader will remove the follower from isr
>
> **TYPE**: long
>
> **DEFAULT**: 10000
>
> [Source](http://kafka.apache.org/0110/documentation/#brokerconfigs)

## 15.Kafka高级_生产者ACk机制

对于某些不太重要的数据，对数据的可靠性要求不是很高，能够容忍数据的少量丢失，所以没必要等 ISR 中的 follower 全部接收成功。

所以 Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡，选择以下的配置。

**acks 参数配置**：

- 0： producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟， broker 一接收到还没有写入磁盘就已经返回，当 broker 故障时有可能**丢失数据**；
- 1： producer 等待 broker 的 ack， partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会**丢失数据**；

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/10.png)

- -1（all） ： producer 等待 broker 的 ack， partition 的 leader 和 ISR 的 follower 全部落盘成功后才返回 ack。但是如果在 follower 同步完成后， broker 发送 ack 之前， leader 发生故障，那么会造成**数据重复**。

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/11.png)

**助记：返ACK前，0无落盘，1一落盘，-1全落盘，（落盘：消息存到本地）**

> **acks**
>
> **DESCRIPTION**:
>
> The number of acknowledgments the producer requires the leader to have received before considering a request complete. This controls the durability of records that are sent. The following settings are allowed:
>
> - `acks=0` If set to zero then the producer will not wait for any acknowledgment from the server at all. The record will be immediately added to the socket buffer and considered sent. No guarantee can be made that the server has received the record in this case, and the retries configuration will not take effect (as the client won't generally know of any failures). The offset given back for each record will always be set to -1.
> - `acks=1` This will mean the leader will write the record to its local log but will respond without awaiting full acknowledgement from all followers. In this case should the leader fail immediately after acknowledging the record but before the followers have replicated it then the record will be lost.
> - `acks=all` This means the leader will wait for the full set of in-sync replicas to acknowledge the record. This guarantees that the record will not be lost as long as at least one in-sync replica remains alive. This is the strongest available guarantee. This is equivalent to the acks=-1 setting.
>
> **TYPE**:string
>
> **DEFAULT**:1
>
> **VALID VALUES**:[all, -1, 0, 1]
>
> [Source](http://kafka.apache.org/0110/documentation/#producerconfigs)

## 16.Kafka高级_数据一致性问题

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/12.png)

- LEO：（Log End Offset）每个副本的最后一个 offset
- HW：（High Watermark）高水位，指的是消费者能见到的最大的 offset， ISR 队列中最小的 LEO

### follower 故障和 leader 故障

- **follower 故障**：follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后， follower 会读取本地磁盘记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重新加入 ISR 了。
- **leader 故障**：leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性， 其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader 同步数据。

注意： 这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。

## 17.Kafka高级_ExactlyOnce

将服务器的 ACK 级别设置为-1（all），可以保证 Producer 到 Server 之间不会丢失数据，即 **At Least Once** 语义。

相对的，将服务器 ACK 级别设置为 0，可以保证生产者每条消息只会被发送一次，即 **At Most Once** 语义。

At Least Once 可以保证数据不丢失，但是不能保证数据不重复；相对的， At Most Once 可以保证数据不重复，但是不能保证数据不丢失。 但是，对于一些非常重要的信息，比如说**交易数据**，下游数据消费者要求数据既不重复也不丢失，即 **Exactly Once** 语义。

> - At least once—Messages are **never lost** but may be redelivered.
> - At most once—Messages **may be lost** but are never redelivered.
> - Exactly once—this is what people actually want, each message is delivered once and only once.
>
> [Source](http://kafka.apache.org/0110/documentation/#semantics)

在 0.11 版本以前的 Kafka，对此是无能为力的，只能保证数据不丢失，再在下游消费者对数据做全局去重。对于多个下游应用的情况，每个都需要单独做全局去重，这就对性能造成了很大影响。

0.11 版本的 Kafka，引入了一项重大特性：**幂等性**。**所谓的幂等性就是指 Producer 不论向 Server 发送多少次重复数据， Server 端都只会持久化一条**。幂等性结合 At Least Once 语义，就构成了 Kafka 的 Exactly Once 语义。即：

```
At Least Once + 幂等性 = Exactly Once
```

要启用幂等性，只需要将 Producer 的参数中 `enable.idempotence` 设置为 true 即可。 Kafka 的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的 Producer 在初始化的时候会被分配一个 PID，发往同一 Partition 的消息会附带 Sequence Number。而 Broker 端会对`<PID, Partition, SeqNumber>`做缓存，当具有相同主键的消息提交时， Broker 只会持久化一条。

但是 PID 重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨分区跨会话的 Exactly Once。

> **enable.idempotence**
>
> DESCRIPTION:When set to 'true', the producer will ensure that exactly one copy of each message is written in the stream. If 'false', producer retries due to broker failures, etc., may write duplicates of the retried message in the stream. This is set to 'false' by default. Note that enabling idempotence requires `max.in.flight.requests.per.connection` to be set to 1 and `retries` cannot be zero. Additionally acks must be set to 'all'. If these values are left at their defaults, we will override the default to be suitable. If the values are set to something incompatible with the idempotent producer, a ConfigException will be thrown.
>
> TYPE:boolean
>
> DEFAULT:false
>
> [Source](http://kafka.apache.org/0110/documentation/#producerconfigs)

## 18.Kafka高级_生产者总结

略

## 19.Kafka高级_消费者分区分配策略

### 消费方式

**consumer 采用 pull（拉） 模式从 broker 中读取数据**。

**push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的**。它的目标是尽可能以最快速度传递消息，但是这样很容易造成 consumer 来不及处理消息，典型的表现就是拒绝服务以及网络拥塞。而 pull 模式则可以根据 consumer 的消费能力以适当的速率消费消息。

**pull 模式不足之处**是，如果 kafka 没有数据，消费者可能会陷入循环中， 一直返回空数据。 针对这一点， Kafka 的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有数据可供消费， consumer 会等待一段时间之后再返回，这段时长即为 timeout。

[Push vs. pull](http://kafka.apache.org/0110/documentation/#design_pull)

### 分区分配策略

一个 consumer group 中有多个 consumer，一个 topic 有多个 partition，所以必然会涉及到 partition 的分配问题，即确定那个 partition 由哪个 consumer 来消费。

Kafka 有两种分配策略：

- round-robin 循环
- range

> **partition.assignment.strategy**
>
> Select between the "range" or "roundrobin" strategy for assigning分配 partitions to consumer streams.
>
> The **round-robin** partition assignor lays out规划 all the available partitions and all the available consumer threads. It then proceeds to do接着做 a round-robin assignment from partition to consumer thread. If the subscriptions订阅 of all consumer instances are identical完全同样的, then the partitions will be uniformly 均匀地distributed. (i.e.也就是说, the partition ownership counts will be within a delta of exactly one across all consumer threads.) Round-robin assignment is permitted only if:
>
> 1. Every topic has the same number of streams within a consumer instance
> 2. The set of subscribed topics is identical for every consumer instance within the group.
>
> **Range** partitioning works on a per-**topic** basis. For each topic, we lay out the available partitions in numeric order and the consumer threads in lexicographic词典式的 order. We then divide the number of partitions by the total number of consumer streams (threads) to determine the number of partitions to assign to each consumer. If it does not evenly divide, then the first few consumers will have one extra partition.
>
> **DEFAULT**:range
>
> [Source](http://kafka.apache.org/0110/documentation/#oldconsumerconfigs)

------

[Kafka再平衡机制详解](https://zhuanlan.zhihu.com/p/86718818)

#### Round Robin

关于 Roudn Robin 重分配策略，其主要采用的是一种轮询的方式分配所有的分区，该策略主要实现的步骤如下。这里我们首先假设有三个 topic：t0、t1 和 t2，这三个 topic 拥有的分区数分别为 1、2 和 3，那么总共有六个分区，这六个分区分别为：t0-0、t1-0、t1-1、t2-0、t2-1 和 t2-2。这里假设我们有三个 consumer：C0、C1 和 C2，它们订阅情况为：C0 订阅 t0，C1 订阅 t0 和 t1，C2 订阅 t0、t1 和 t2。那么这些分区的分配步骤如下：

- 首先将所有的 partition 和 consumer 按照字典序进行排序，所谓的字典序，就是按照其名称的字符串顺序，那么上面的六个分区和三个 consumer 排序之后分别为：

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/16.png)

- 然后依次以按顺序轮询的方式将这六个分区分配给三个 consumer，如果当前 consumer 没有订阅当前分区所在的 topic，则轮询的判断下一个 consumer：
- 尝试将 t0-0 分配给 C0，由于 C0 订阅了 t0，因而可以分配成功；
- 尝试将 t1-0 分配给 C1，由于 C1 订阅了 t1，因而可以分配成功；
- 尝试将 t1-1 分配给 C2，由于 C2 订阅了 t1，因而可以分配成功；
- 尝试将 t2-0 分配给 C0，由于 C0 没有订阅 t2，因而会轮询下一个 consumer；
- 尝试将 t2-0 分配给 C1，由于 C1 没有订阅 t2，因而会轮询下一个 consumer；
- 尝试将 t2-0 分配给 C2，由于 C2 订阅了 t2，因而可以分配成功；
- 同理由于 t2-1 和 t2-2 所在的 topic 都没有被 C0 和 C1 所订阅，因而都不会分配成功，最终都会分配给 C2。
- 按照上述的步骤将所有的分区都分配完毕之后，最终分区的订阅情况如下：

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/17.png)

从上面的步骤分析可以看出，轮询的策略就是简单的将所有的 partition 和 consumer 按照字典序进行排序之后，然后依次将 partition 分配给各个 consumer，如果当前的 consumer 没有订阅当前的 partition，那么就会轮询下一个 consumer，直至最终将所有的分区都分配完毕。但是从上面的分配结果可以看出，轮询的方式会导致每个 consumer 所承载的分区数量不一致，从而导致各个 consumer 压力不均一。

#### Range

所谓的 Range 重分配策略，就是首先会计算各个 consumer 将会承载的分区数量，然后将指定数量的分区分配给该 consumer。这里我们假设有两个 consumer：C0 和 C1，两个 topic：t0 和 t1，这两个 topic 分别都有三个分区，那么总共的分区有六个：t0-0、t0-1、t0-2、t1-0、t1-1 和 t1-2。那么 Range 分配策略将会按照如下步骤进行分区的分配：

- 需要注意的是，Range 策略是按照 topic 依次进行分配的，比如我们以 t0 进行讲解，其首先会获取 t0 的所有分区：t0-0、t0-1 和 t0-2，以及所有订阅了该 topic 的 consumer：C0 和 C1，并且会将这些分区和 consumer 按照字典序进行排序；
- 然后按照平均分配的方式计算每个 consumer 会得到多少个分区，如果没有除尽，则会将多出来的分区依次计算到前面几个 consumer。比如这里是三个分区和两个 consumer，那么每个 consumer 至少会得到 1 个分区，而 3 除以 2 后还余 1，那么就会将多余的部分依次算到前面几个 consumer，也就是这里的 1 会分配给第一个 consumer，总结来说，那么 C0 将会从第 0 个分区开始，分配 2 个分区，而 C1 将会从第 2 个分区开始，分配 1 个分区；
- 同理，按照上面的步骤依次进行后面的 topic 的分配。
- 最终上面六个分区的分配情况如下：

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/18.png)

可以看到，如果按照`Range`分区方式进行分配，其本质上是依次遍历每个 topic，然后将这些 topic 的分区按照其所订阅的 consumer 数量进行平均的范围分配。这种方式从计算原理上就会导致排序在前面的 consumer 分配到更多的分区，从而导致各个 consumer 的压力不均衡。

TODO:我的问题：topic 分多个 partition，有些 custom 根据上述策略，分到 topic 的部分 partition，难道不是要全部 partition 吗？是不是还要按照相同策略多分配多一次？

## 20.Kafka高级_消费者offset的存储

由于 consumer 在消费过程中可能会出现断电宕机等故障， consumer 恢复后，需要从故障前的位置的继续消费，所以 **consumer 需要实时记录自己消费到了哪个 offset**，以便故障恢复后继续消费。

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/13.png)

**Kafka 0.9 版本之前， consumer 默认将 offset 保存在 Zookeeper 中，从 0.9 版本开始，consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为__consumer_offsets**。

1. 修改配置文件 consumer.properties，`exclude.internal.topics=false`。
2. 读取 offset
    - 0.11.0.0 之前版本 - `bin/kafka-console-consumer.sh --topic __consumer_offsets --zookeeper hadoop102:2181 --formatter "kafka.coordinator.GroupMetadataManager\$OffsetsMessageFormatter" --consumer.config config/consumer.properties --from-beginning`
    - 0.11.0.0 及之后版本 - `bin/kafka-console-consumer.sh --topic __consumer_offsets --zookeeper hadoop102:2181 --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --consumer.config config/consumer.properties --from-beginning`

TODO:上机实验

## 21.Kafka高级_消费者组案例

### 需求

测试同一个消费者组中的消费者， **同一时刻只能有一个**消费者消费。

### 操作步骤

1.修改`%KAFKA_HOME\config\consumer.properties%`文件中的`group.id`属性。

```properties
group.id=shan-kou-zu
```

2.打开两个 cmd，分别启动两个消费者。（以`%KAFKA_HOME\config\consumer.properties%`作配置参数）

```bat
bin\windows\kafka-console-consumer.bat --zookeeper 127.0.0.1:2181 --topic test --consumer.config config\consumer.properties
```

3.再打开一个 cmd，启动一个生产者。

```bat
bin\windows\kafka-console-producer.bat --broker-list 127.0.0.1:9092 --topic test
```

4.在生产者窗口输入消息，观察两个消费者窗口。**会发现两个消费者窗口中，只有一个才会弹出消息**。

## 22.Kafka高级_高效读写&ZK作用

### 顺序写磁盘

Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端，为顺序写。 官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这与磁盘的机械机构有关，顺序写之所以快，是因为其**省去了大量磁头寻址的时间**。

### 零复制技术

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/14.png)

- NIC network interface controller 网络接口控制器

### Zookeeper 在 Kafka 中的作用

Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所有 topic 的分区副本分配和 leader 选举等工作。[Reference](http://kafka.apache.org/0110/documentation/#design_replicamanagment)

Controller 的管理工作都是依赖于 Zookeeper 的。

以下为 partition 的 leader 选举过程：

![Leader选举流程](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/15.png)

## 23.Kafka高级_Ranger分区再分析

略

## 24.Kafka高级_事务

Kafka 从 0.11 版本开始引入了事务支持。事务可以保证 Kafka 在 Exactly Once 语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。

### Producer 事务

为了实现跨分区跨会话的事务，需要引入一个全局唯一的 Transaction ID，并将 Producer 获得的 PID 和 Transaction ID 绑定。这样当 Producer 重启后就可以通过正在进行的 TransactionID 获得原来的 PID。

为了管理 Transaction， Kafka 引入了一个新的组件 Transaction Coordinator。 Producer 就是通过和 Transaction Coordinator 交互获得 Transaction ID 对应的任务状态。 Transaction Coordinator 还负责将事务所有写入 Kafka 的一个内部 Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。

### Consumer 事务

上述事务机制主要是从 Producer 方面考虑，对于 Consumer 而言，事务的保证就会相对较弱，尤其时无法保证 Commit 的信息被精确消费。这是由于 Consumer 可以通过 offset 访问任意信息，而且不同的 Segment File 生命周期不同，同一事务的消息可能会出现重启后被删除的情况。

## 25.Kafka高级_API生产者流程

Kafka 的 Producer 发送消息采用的是异步发送的方式。在消息发送的过程中，涉及到了两个线程——main 线程和 Sender 线程，以及一个线程共享变量——RecordAccumulator。 main 线程将消息发送给 RecordAccumulator， Sender 线程不断从 RecordAccumulator 中拉取消息发送到 Kafka broker。

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/19.png)

相关参数：

- **batch.size**： 只有数据积累到 batch.size 之后， sender 才会发送数据。
- **linger.ms**： 如果数据迟迟未达到 batch.size， sender 等待 linger.time 之后就会发送数据。

## 26.Kafka高级_异步发送API普通生产者

### 导入依赖

[pom.xml](https://my.oschina.net/jallenkwong/blog/pom.xml)

```xml
<dependency>
	<groupId>org.apache.kafka</groupId>
	<artifactId>kafka-clients</artifactId>
	<version>0.11.0.0</version>
</dependency>
```

### 编写代码

需要用到的类：

- KafkaProducer：需要创建一个生产者对象，用来发送数据
- ProducerConfig：获取所需的一系列配置参数
- ProducerRecord：每条数据都要封装成一个 ProducerRecord 对象

[CustomProducer.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/producer/CustomProducer.java)

```java
import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class CustomProducer {

	public static void main(String[] args) {
		Properties props = new Properties();
		// kafka 集群， broker-list
		
		props.put("bootstrap.servers", "127.0.0.1:9092");
		
		//可用ProducerConfig.ACKS_CONFIG 代替 "acks"
		//props.put(ProducerConfig.ACKS_CONFIG, "all");
		props.put("acks", "all");
		// 重试次数
		props.put("retries", 1);
		// 批次大小
		props.put("batch.size", 16384);
		// 等待时间
		props.put("linger.ms", 1);
		// RecordAccumulator 缓冲区大小
		props.put("buffer.memory", 33554432);
		props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		Producer<String, String> producer = new KafkaProducer<>(props);
		for (int i = 0; i < 100; i++) {
			producer.send(new ProducerRecord<String, String>("test", "test-" + Integer.toString(i),
					"test-" + Integer.toString(i)));
		}
		producer.close();
	}

}
```

## 27.Kafka高级_回顾

略

## 28.Kafka案例_异步发送API带回调函数的生产者

回调函数会在 producer 收到 ack 时调用，为异步调用， 该方法有两个参数，分别是 RecordMetadata 和 Exception，如果 Exception 为 null，说明消息发送成功，如果 Exception 不为 null，说明消息发送失败。

**注意**：消息发送失败会自动重试，不需要我们在回调函数中手动重试。

[CallBackProducer.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/producer/CallBackProducer.java)

```java
import java.util.Properties;

import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

public class CallBackProducer {
	public static void main(String[] args) {
		Properties props = new Properties();
		props.put("bootstrap.servers", "127.0.0.1:9092");//kafka 集群， broker-list
		props.put("acks", "all");
		props.put("retries", 1);//重试次数
		props.put("batch.size", 16384);//批次大小
		props.put("linger.ms", 1);//等待时间
		props.put("buffer.memory", 33554432);//RecordAccumulator 缓冲区大小
		props.put("key.serializer",
		"org.apache.kafka.common.serialization.StringSerializer");
		props.put("value.serializer",
		"org.apache.kafka.common.serialization.StringSerializer");
		
		Producer<String, String> producer = new KafkaProducer<>(props);
		for (int i = 0; i < 100; i++) {
			producer.send(new ProducerRecord<String, String>("test",
				"test" + Integer.toString(i)), new Callback() {
			
				//回调函数， 该方法会在 Producer 收到 ack 时调用，为异步调用
				@Override
				public void onCompletion(RecordMetadata metadata, Exception exception) {
					if (exception == null) {
						System.out.println(metadata.partition() + " - " + metadata.offset());
					} else {
						exception.printStackTrace();
					}
				}
			});
		}
		
		producer.close();
	}
}
```

## 29.Kafka案例_API生产者分区策略测试

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/20.png)

ProducerRecord 类有许多构造函数，其中一个参数 partition 可指定分区

TODO:根据策略阐述，亲自设计分区策略测试，range 和 round-robin 间的区别

## 30.Kafka案例_API带自定义分区器的生成者

[MyPartitioner.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/producer/MyPartitioner.java)

```java
import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;

public class MyPartitioner implements Partitioner {

	@Override
	public void configure(Map<String, ?> configs) {
		// TODO Auto-generated method stub

	}

	@Override
	public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public void close() {
		// TODO Auto-generated method stub

	}

}
```

具体内容填写可参考默认分区器`org.apache.kafka.clients.producer.internals.DefaultPartitioner`

然后 Producer 配置中注册使用

```java
Properties props = new Properties();
...
props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, MyPartitioner.class);
...
Producer<String, String> producer = new KafkaProducer<>(props);
```

## 31.Kafka案例_API同步发送生成者

同步发送的意思就是，一条消息发送之后，会阻塞当前线程， 直至返回 ack。

由于 send 方法返回的是一个 Future 对象，根据 Futrue 对象的特点，我们也可以实现同步发送的效果，只需在调用 Future 对象的 get 方发即可。

[SyncProducer.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/producer/SyncProducer.java)

```java
Producer<String, String> producer = new KafkaProducer<>(props);
for (int i = 0; i < 100; i++) {
	producer.send(new ProducerRecord<String, String>("test",  "test - 1"), new Callback() {
		@Override
		public void onCompletion(RecordMetadata metadata, Exception exception) {
			...
		}
	}).get();//<----------------------
}
```

## 32.Kafka案例_API简单消费者

- **KafkaConsumer**： 需要创建一个消费者对象，用来消费数据
- **ConsumerConfig**： 获取所需的一系列配置参数
- **ConsuemrRecord**： 每条数据都要封装成一个 ConsumerRecord 对象

**为了使我们能够专注于自己的业务逻辑， Kafka 提供了自动提交 offset 的功能**。

自动提交 offset 的相关参数：

- **enable.auto.commit**：是否开启自动提交 offset 功能
- **auto.commit.interval.ms**：自动提交 offset 的时间间隔

[CustomConsumer.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/consumer/CustomConsumer.java)

```
import java.util.Arrays;
import java.util.Properties;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

public class CustomConsumer {
	public static void main(String[] args) {
		
		Properties props = new Properties();
		
		props.put("bootstrap.servers", "127.0.0.1:9092");
		props.put("group.id", "abc");
		props.put("enable.auto.commit", "true");
		props.put("auto.commit.interval.ms", "1000");
		props.put("key.deserializer",
				"org.apache.kafka.common.serialization.StringDeserializer");
		props.put("value.deserializer",
				"org.apache.kafka.common.serialization.StringDeserializer");
		
		KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
		
		consumer.subscribe(Arrays.asList("test"));
		
		while (true) {
			ConsumerRecords<String, String> records = consumer.poll(100);
			for (ConsumerRecord<String, String> record : records) {
				System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
			}
		}
	}
}
```

## 33.Kafka案例_API消费者重置offset

Consumer 消费数据时的可靠性是很容易保证的，因为数据在 Kafka 中是持久化的，故不用担心数据丢失问题。

由于 consumer 在消费过程中可能会出现断电宕机等故障， consumer 恢复后，需要从故障前的位置的继续消费，所以** consumer 需要实时记录自己消费到了哪个 offset，以便故障恢复后继续消费**。

**所以 offset 的维护是 Consumer 消费数据是必须考虑的问题**。

```java
public static final String AUTO_OFFSET_RESET_CONFIG = "auto.offset.reset";
Properties props = new Properties();
...
props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
props.put("group.id", "abcd");//组id需另设，否则看不出上面一句的配置效果
...
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
```

从结果看，`props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");`与命令行中`bin\windows\kafka-console-consumer.bat --bootstrap-server localhost:9092 --topic test --from-beginning`的`--from-beginning`拥有相同的作用。

------

> **--from-beginning**
>
> If the consumer does not already have an established offset to consume from, start with the earliest message present in the log rather than the latest message.
>
> From [kafka-console-sonsumer.bat](https://my.oschina.net/jallenkwong/blog/4449224)

------

> **auto.offset.reset**
>
> What to do when there is no initial offset in Kafka or if the current offset does not exist any more on the server (e.g. because that data has been deleted):
>
> - **earliest**: automatically reset the offset to the earliest offset
> - **latest**: automatically reset the offset to the latest offset
> - **none**: throw exception to the consumer if no previous offset is found for the consumer's group
> - **anything else**: throw exception to the consumer.
>
> **TYPE**:string
>
> **DEFAULT**:latest
>
> **VALID VALUES**:[latest, earliest, none]
>
> [Source](http://kafka.apache.org/0110/documentation/#newconsumerconfigs)

## 34.Kafka案例_消费者保存offset读取问题

```java
props.put("enable.auto.commit", "true");
```

> **enable.auto.commit**
>
> If true the consumer's offset will be periodically committed in the background.
>
> **TYPE**:boolean
>
> **DEFAULT**:true
>
> [Source](http://kafka.apache.org/0110/documentation/#newconsumerconfigs)

PS.我将 Offset 提交类比成数据库事务的提交。

## 35.Kafka案例_API消费者手动提交offset

虽然自动提交 offset 十分便利，但由于其是基于时间提交的， 开发人员难以把握 offset 提交的时机。因此 **Kafka 还提供了手动提交 offset 的 API**。

手动提交 offset 的方法有两种：

1. commitSync（同步提交）
2. commitAsync（异步提交）

两者的**相同点**是，都会将本次 poll 的一批数据最高的偏移量提交；

**不同点**是，commitSync 阻塞当前线程，一直到提交成功，并且会自动失败重试（由不可控因素导致，也会出现提交失败）；而 commitAsync 则没有失败重试机制，故有可能提交失败。

### 同步提交offset

由于同步提交 offset 有失败重试机制，故更加可靠，以下为同步提交 offset 的示例。

[SyncCommitOffset.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/consumer/SyncCommitOffset.java)

```java
public class SyncCommitOffset {
	public static void main(String[] args) {
		Properties props = new Properties();
		...
		//<-----------------
		//关闭自动提交 offset
		props.put("enable.auto.commit", "false");
		...
		KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
		consumer.subscribe(Arrays.asList("first"));//消费者订阅主题
		while (true) {
			//消费者拉取数据
			ConsumerRecords<String, String> records =
			consumer.poll(100);
			for (ConsumerRecord<String, String> record : records) {
				System.out.printf("offset = %d, key = %s, value= %s%n", record.offset(), record.key(), record.value());
			}
			//<---------------------------------------
			//同步提交，当前线程会阻塞直到 offset 提交成功
			consumer.commitSync();
		}
	}
}
```

### 异步提交offset

虽然同步提交 offset 更可靠一些，但是由于其会阻塞当前线程，直到提交成功。因此吞吐量会收到很大的影响。因此更多的情况下，会选用异步提交 offset 的方式。

[AsyncCommitOffset.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/consumer/AsyncCommitOffset.java)

```java
public class AsyncCommitOffset {
	public static void main(String[] args) {
		Properties props = new Properties();
		...
		//<--------------------------------------
		//关闭自动提交
		props.put("enable.auto.commit", "false");
		...
		KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
		consumer.subscribe(Arrays.asList("first"));// 消费者订阅主题
		while (true) {
			ConsumerRecords<String, String> records = consumer.poll(100);// 消费者拉取数据
			for (ConsumerRecord<String, String> record : records) {
				System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
			}
			//<----------------------------------------------
			// 异步提交
			consumer.commitAsync(new OffsetCommitCallback() {
				@Override
				public void onComplete(Map<TopicPartition, OffsetAndMetadata> offsets, Exception exception) {
					if (exception != null) {
						System.err.println("Commit failed for" + offsets);
					}
				}
			});
		}
	}
}
```

### 数据漏消费和重复消费分析

无论是同步提交还是异步提交 offset，都有可能会造成数据的漏消费或者重复消费。先提交 offset 后消费，有可能造成数据的漏消费；而先消费后提交 offset，有可能会造成数据的重复消费。

### 自定义存储 offset

Kafka 0.9 版本之前， offset 存储在 zookeeper， 0.9 版本及之后，默认将 offset 存储在 Kafka 的一个内置的 topic 中。除此之外， Kafka 还可以选择自定义存储 offset。

offset 的维护是相当繁琐的， 因为需要考虑到消费者的 Rebalace。

**当有新的消费者加入消费者组、 已有的消费者推出消费者组或者所订阅的主题的分区发生变化，就会触发到分区的重新分配，重新分配的过程叫做 Rebalance**。

消费者发生 Rebalance 之后，每个消费者消费的分区就会发生变化。**因此消费者要首先获取到自己被重新分配到的分区，并且定位到每个分区最近提交的 offset 位置继续消费**。

要实现自定义存储 offset，需要借助 `ConsumerRebalanceListener`， 以下为示例代码，其中提交和获取 offset 的方法，需要根据所选的 offset 存储系统自行实现。(可将 offset 存入 MySQL 数据库)

[CustomSaveOffset.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/consumer/CustomSaveOffset.java)

```java
public class CustomSaveOffset {
	private static Map<TopicPartition, Long> currentOffset = new HashMap<>();

	public static void main(String[] args) {
		// 创建配置信息
		Properties props = new Properties();
		...
		//<--------------------------------------
		// 关闭自动提交 offset
		props.put("enable.auto.commit", "false");
		...
		// 创建一个消费者
		KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
		// 消费者订阅主题
		consumer.subscribe(Arrays.asList("first"), 
			//<-------------------------------------
			new ConsumerRebalanceListener() {
			// 该方法会在 Rebalance 之前调用
			@Override
			public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
				commitOffset(currentOffset);
			}

			// 该方法会在 Rebalance 之后调用
			@Override
			public void onPartitionsAssigned(Collection<TopicPartition> partitions) {

				currentOffset.clear();
				for (TopicPartition partition : partitions) {
					consumer.seek(partition, getOffset(partition));// 定位到最近提交的 offset 位置继续消费
				}
			}
		});
		
		while (true) {
			ConsumerRecords<String, String> records = consumer.poll(100);// 消费者拉取数据
			for (ConsumerRecord<String, String> record : records) {
				System.out.printf("offset = %d, key = %s, value = %s%n", record.offset(), record.key(), record.value());
				currentOffset.put(new TopicPartition(record.topic(), record.partition()), record.offset());
			}
			commitOffset(currentOffset);// 异步提交
		}
	}

	// 获取某分区的最新 offset
	private static long getOffset(TopicPartition partition) {
		return 0;
	}

	// 提交该消费者所有分区的 offset
	private static void commitOffset(Map<TopicPartition, Long> currentOffset) {
	}
}
```

## 36.Kafka案例_API自定义拦截器（需求分析）

### 拦截器原理

Producer 拦截器(interceptor)是在 Kafka 0.10 版本被引入的，主要用于实现 clients 端的定制化控制逻辑。

对于 producer 而言， interceptor 使得用户在消息发送前以及 producer 回调逻辑前有机会对消息做一些定制化需求，比如`修改消息`等。同时， producer 允许用户指定多个 interceptor 按序作用于同一条消息从而形成一个拦截链(interceptor chain)。 Intercetpor 的实现接口是`org.apache.kafka.clients.producer.ProducerInterceptor`，其定义的方法包括：

- `configure(configs)`：获取配置信息和初始化数据时调用。
- `onSend(ProducerRecord)`：该方法封装进 KafkaProducer.send 方法中，即它运行在用户主线程中。 Producer 确保**在消息被序列化以及计算分区前**调用该方法。 用户可以在该方法中对消息做任何操作，但最好保证不要修改消息所属的 topic 和分区， 否则会影响目标分区的计算。
- `onAcknowledgement(RecordMetadata, Exception)`：**该方法会在消息从 RecordAccumulator 成功发送到 Kafka Broker 之后，或者在发送过程中失败时调用**。 并且通常都是在 producer 回调逻辑触发之前。 onAcknowledgement 运行在 producer 的 IO 线程中，因此不要在该方法中放入很重的逻辑，否则会拖慢 producer 的消息发送效率。
- `close()`：关闭 interceptor，主要用于执行一些**资源清理**工作

如前所述， interceptor 可能被运行在多个线程中，因此在具体实现时用户需要自行确保线程安全。另外**倘若指定了多个 interceptor，则 producer 将按照指定顺序调用它们**，并仅仅是捕获每个 interceptor 可能抛出的异常记录到错误日志中而非在向上传递。这在使用过程中要特别留意。

## 37.Kafka案例_API自定义拦截器（代码实现）

### 需求

实现一个简单的双 interceptor 组成的拦截链。

- 第一个 interceptor 会在消息发送前将时间戳信息加到消息 value 的最前部；
- 第二个 interceptor 会在消息发送后更新成功发送消息数或失败发送消息数。

### 案例实操

#### 增加时间戳拦截器

[TimeInterceptor.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/interceptor/TimeInterceptor.java)

```java
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

public class TimeInterceptor implements ProducerInterceptor<String, String> {
	@Override
	public void configure(Map<String, ?> configs) {
	}

	@Override
	public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
		// 创建一个新的 record，把时间戳写入消息体的最前部
		return new ProducerRecord(record.topic(), record.partition(), record.timestamp(), record.key(),
				"TimeInterceptor: " + System.currentTimeMillis() + "," + record.value().toString());
	}

	@Override
	public void close() {
	}

	@Override
	public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
		// TODO Auto-generated method stub
		
	}
}
```

#### 增加时间戳拦截器

统计发送消息成功和发送失败消息数，并在 producer 关闭时打印这两个计数器

[CounterInterceptor.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/interceptor/CounterInterceptor.java)

```java
import java.util.Map;

import org.apache.kafka.clients.producer.ProducerInterceptor;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.clients.producer.RecordMetadata;

public class CounterInterceptor implements ProducerInterceptor<String, String>{

	private int errorCounter = 0;
	private int successCounter = 0;
	
	@Override
	public void configure(Map<String, ?> configs) {
		// TODO Auto-generated method stub
		
	}

	@Override
	public ProducerRecord<String, String> onSend(ProducerRecord<String, String> record) {
		return record;
	}

	@Override
	public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
		// 统计成功和失败的次数
		if (exception == null) {
			successCounter++;
		} else {
			errorCounter++;
		}
	}

	@Override
	public void close() {
		// 保存结果
		System.out.println("Successful sent: " + successCounter);
		System.out.println("Failed sent: " + errorCounter);
		
	}

}
```

#### producer 主程序

[InterceptorProducer.java](https://gitee.com/jallenkwong/LearnKafka/blob/master/src/main/java/com/lun/kafka/producer/InterceptorProducer.java)

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;

public class InterceptorProducer {
	public static void main(String[] args) {
		// 1 设置配置信息
		Properties props = new Properties();
		props.put("bootstrap.servers", "127.0.0.1:9092");
		props.put("acks", "all");
		props.put("retries", 3);
		props.put("batch.size", 16384);
		props.put("linger.ms", 1);
		props.put("buffer.memory", 33554432);
		props.put("key.serializer",
				"org.apache.kafka.common.serialization.StringSerializer");
		props.put("value.serializer",
				"org.apache.kafka.common.serialization.StringSerializer");
		//<--------------------------------------------
		// 2 构建拦截链
		List<String> interceptors = new ArrayList<>();
		interceptors.add("com.lun.kafka.interceptor.TimeInterceptor");
		interceptors.add("com.lun.kafka.interceptor.CounterInterceptor");
		
		props.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, interceptors);
		
		String topic = "test";
		Producer<String, String> producer = new KafkaProducer<>(props);
		// 3 发送消息
		for (int i = 0; i < 10; i++) {
			ProducerRecord<String, String> record = new ProducerRecord<>(topic, "message" + i);
			producer.send(record);
		}
		
		// 4 一定要关闭 producer，这样才会调用 interceptor 的 close 方法
		producer.close();
		
	}
}
```

## 38.Kafka案例_API自定义拦截器（案例测试）

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/21.png)

## 39.Kafka案例_监控Eagle的安装

- [Kafka Eagle官方主页](https://www.kafka-eagle.org/)
- [Kafka Eagle下载页面](http://download.kafka-eagle.org/)
- [Kafka Eagle官方文档](https://www.kafka-eagle.org/articles/docs/documentation.html)

### 什么是Kafka Eagle

> Kafka Eagle is open source visualization and management software. It allows you to query, visualize, alert on, and explore your metrics no matter where they are stored. In plain English, it provides you with tools to turn your kafka cluster data into beautiful graphs and visualizations.
>
> Kafka Eagle是开源可视化和管理软件。它允许您查询、可视化、提醒和探索您的指标，无论它们存储在哪里。简单地说，它为您提供了将kafka集群数据转换为漂亮的图形和可视化的工具。
>
> [Source](https://www.kafka-eagle.org/articles/docs/introduce/what-is-kafka-eagle.html)

一个运行在 Tomcat 的 Web 应用。

### 安装与运行

#### 在Linux上安装与运行

略

#### 在Windows上安装与运行

- 安装 JDK，设置环境变量时，**路径最好不要带空格**，否则，后序运行 ke.bat 抛异常。若路径必须带有空格，可以通过小技巧，让 ke.bat 成功运行。这个技巧是：若你的`JAVA_HOME`的变量值为`C:\Program Files\Java\jdk1.8.0_161`，则将其改成`C:\progra~1\Java\jdk1.8.0_161`。
- 到[Kafka Eagle下载页面](http://download.kafka-eagle.org/)下载安装包。也可在网盘下载，链接在[本文首部](https://my.oschina.net/jallenkwong/blog/4449224#)
- 解压安装包，然后设置环境变量`KE_HOME`，其值如`C:\Kafka\kafka-eagle-web-1.3.7`。若想打开 cmd 输入命令`ke.bat`运行 Kafka Eagle，在`PATH`环境变量的值头添加`%KE_HOME%\bin;`
- 修改配置文件`%KE_HOME%\conf\system-config.properties`
- (可选)Kafka Server 的 JVM 调参，用文本编辑器打开`%KAFKA_HOME%\bin\windows\kafka-server-start.bat`，其中的`set KAFKA_HEAP_OPTS=-Xmx1G -Xms1G`改为`set KAFKA_HEAP_OPTS=-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70`

```properties
######################################
# multi zookeeper&kafka cluster list
######################################<---设置ZooKeeper的IP地址
kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=localhost:2181

######################################
# zk client thread limit
######################################
kafka.zk.limit.size=25

######################################
# kafka eagle webui port
######################################
kafka.eagle.webui.port=8048

######################################
# kafka offset storage
######################################
cluster1.kafka.eagle.offset.storage=kafka
#cluster2.kafka.eagle.offset.storage=zk

######################################
# enable kafka metrics
######################################<---metrics.charts从false改成true
kafka.eagle.metrics.charts=true
kafka.eagle.sql.fix.error=false

######################################
# kafka sql topic records max
######################################
kafka.eagle.sql.topic.records.max=5000

######################################
# alarm email configure
######################################
kafka.eagle.mail.enable=false
kafka.eagle.mail.sa=alert_sa@163.com
kafka.eagle.mail.username=alert_sa@163.com
kafka.eagle.mail.password=mqslimczkdqabbbh
kafka.eagle.mail.server.host=smtp.163.com
kafka.eagle.mail.server.port=25

######################################
# alarm im configure
######################################
#kafka.eagle.im.dingding.enable=true
#kafka.eagle.im.dingding.url=https://oapi.dingtalk.com/robot/send?access_token=

#kafka.eagle.im.wechat.enable=true
#kafka.eagle.im.wechat.token=https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=xxx&corpsecret=xxx
#kafka.eagle.im.wechat.url=https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=
#kafka.eagle.im.wechat.touser=
#kafka.eagle.im.wechat.toparty=
#kafka.eagle.im.wechat.totag=
#kafka.eagle.im.wechat.agentid=

######################################
# delete kafka topic token
######################################
kafka.eagle.topic.token=keadmin

######################################
# kafka sasl authenticate
######################################
cluster1.kafka.eagle.sasl.enable=false
cluster1.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
cluster1.kafka.eagle.sasl.mechanism=PLAIN
cluster1.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="kafka-eagle";

#cluster2 在此没有用到，将其注释掉
#cluster2.kafka.eagle.sasl.enable=false
#cluster2.kafka.eagle.sasl.protocol=SASL_PLAINTEXT
#cluster2.kafka.eagle.sasl.mechanism=PLAIN
#cluster2.kafka.eagle.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="kafka-eagle";

######################################
# kafka jdbc driver address
######################################
kafka.eagle.driver=org.sqlite.JDBC
#将url设置在本地
kafka.eagle.url=jdbc:sqlite:/C:/Kafka/kafka-eagle-web-1.3.7/db/ke.db
#进入系统需要用到的账号与密码
kafka.eagle.username=root
kafka.eagle.password=123456
```

- 运行
    - 运行 ZooKeeper
    - 运行 Kafka 集群，另外运行 kafka server 前，需设置 JMX_PORT，否则 Kafka Eagle 后台提示连接失败。执行命令行`set JMX_PORT=9999 & start bin\windows\kafka-server-start.bat config\server.properties`设置 JMX_PORT 且运行 Kafkaserver。在单节点开启 Kafka 集群，小心端口号冲突。
    - 点击`%KE_HOME%\bin\ke.bat`，运行 Kafka Eagle。
    - 打开浏览器，在地址栏输入`http://localhost:8048/ke/`，然后在登录页面，输入在配置文件`%KE_HOME%\conf\system-config.properties`设置的账号与密码。
    - 登录成功，便可进入 Kafka Eagle

![img](https://gitee.com/jallenkwong/LearnKafka/raw/master/image/22.png)

## 40.Kafka案例_监控Eagle的使用

[QUICK START](https://www.kafka-eagle.org/articles/docs/quickstart/dashboard.html)

## 41.Kafka案例_Kafka之与Flume对接

> Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量**日志采集、聚合和传输的系统**，Flume支持在日志系统中定制各类数据发送方，用于收集数据；同时，Flume提供对数据进行简单处理，并写到各种数据接受方（可定制）的能力。
>
> [Source](https://baike.baidu.com/item/flume/6250098)

TODO:学习 Flume 时，再补充

## 42.Kafk之与Flume对接（数据分类）

TODO:学习 Flume 时，再补充

## 43.Kafka之Kafka面试题

1. Kafka 中的 ISR(InSyncRepli)、 OSR(OutSyncRepli)、 AR(AllRepli)代表什么？
2. Kafka 中的 HW、 LEO 等分别代表什么？
3. Kafka 中是怎么体现消息顺序性的？
4. Kafka 中的分区器、序列化器、拦截器是否了解？它们之间的处理顺序是什么？
5. Kafka 生产者客户端的整体结构是什么样子的？使用了几个线程来处理？分别是什么？
6. “消费组中的消费者个数如果超过 topic 的分区，那么就会有消费者消费不到数据”这句 话是否正确？
7. 消费者提交消费位移时提交的是当前消费到的最新消息的 offset 还是 offset+1？
8. 有哪些情形会造成重复消费？
9. 那些情景会造成消息漏消费？
10. 当你使用 kafka-topics.sh 创建（删除）了一个 topic 之后， Kafka 背后会执行什么逻辑？
    - 会在 zookeeper 中的/brokers/topics 节点下创建一个新的 topic 节点，如：/brokers/topics/first
    - 触发 Controller 的监听程序
    - kafka Controller 负责 topic 的创建工作，并更新 metadata cache
11. topic 的分区数可不可以增加？如果可以怎么增加？如果不可以，那又是为什么？
12. topic 的分区数可不可以减少？如果可以怎么减少？如果不可以，那又是为什么？
13. Kafka 有内部的 topic 吗？如果有是什么？有什么所用？
14. Kafka 分区分配的概念？
15. 简述 Kafka 的日志目录结构？
16. 如果我指定了一个 offset， Kafka Controller 怎么查找到对应的消息？
17. 聊一聊 Kafka Controller 的作用？
18. Kafka 中有那些地方需要选举？这些地方的选举策略又有哪些？
19. 失效副本是指什么？有那些应对措施？
20. Kafka 的哪些设计让它有如此高的性能？

[Kafka常见面试题(附个人解读答案+持续更新)](https://blog.csdn.net/C_Xiang_Falcon/article/details/100917145)

## 快捷启动Kafka的Python脚本代码

### 启动单个Kafka

[StartKafka.py3](https://my.oschina.net/jallenkwong/blog/StartKafka.py3)

```python
# -*- coding: utf-8 -*
import os
import time
import subprocess

startZooKeeperServerCmd = 'start bin\windows\zookeeper-server-start.bat config\zookeeper.properties'

startKafkaServerCmd = 'set JMX_PORT=9999 & start bin\windows\kafka-server-start.bat config\server.properties'

print('Starting ZooKeeper Server...')
subprocess.Popen(startZooKeeperServerCmd, shell=True)
time.sleep(10)

# 启动10s后, 轮询 2181端口是否启用
print('Polling...')
startKafkaFalg = False

#每次轮询间隔5秒
interval = 5
count = 6

while count > 0:
    tmpFile = os.popen('netstat -na','r')
    breakWhileFlag = False
    for line in tmpFile.readlines():
        if line.startswith('  TCP    0.0.0.0:2181'):
            breakWhileFlag = True
            break
    print("Not yet.")
    if breakWhileFlag:
        print("It's Ok.")
        startKafkaFalg = True
        break
    else:
        count -= 1
        time.sleep(interval)

if startKafkaFalg:
    time.sleep(interval)
    print("Starting the Kafka .")
    subprocess.Popen(startKafkaServerCmd, shell=True)
else:
    print("Something wrong ...")
    input()#raw_input()
```

### 启动三个Kafka

[StartKafkaCluster.py3](https://my.oschina.net/jallenkwong/blog/StartKafkaCluster.py3)

```python
# -*- coding: utf-8 -*
import os, time, subprocess

startZooKeeperServerCmd = 'start bin\windows\zookeeper-server-start.bat config\zookeeper.properties'

startKafkaServerCmd = 'set JMX_PORT=%d & start bin\windows\kafka-server-start.bat config\%s'

print('Starting ZooKeeper Server...')
subprocess.Popen(startZooKeeperServerCmd, shell=True)
time.sleep(10)

zooKeeperPortNumber = 2181
kafkaPortNumber = 9092
kafkaPortNumber2 = 9093
kafkaPortNumber3 = 9094

kafkaJmxPortNumber = 9997
kafkaJmxPortNumber2 = 9998
kafkaJmxPortNumber3 = 9999

def polling(portNumber, interval = 5, count = 10):
    while count > 0:
        tmpFile = os.popen('netstat -na','r')
        portNumberStr = str(portNumber)
        print("Polling the port: " + portNumberStr)
        for line in tmpFile.readlines():
            if line.startswith('  TCP    0.0.0.0:' + portNumberStr) or line.startswith('  TCP    127.0.0.1:' + portNumberStr):
                return True
        print("Not yet. " + str(portNumber))
        count -= 1
        time.sleep(interval)
    print("Polling the port: " + portNumberStr + " unsuccessfully.")
    return False

if polling(zooKeeperPortNumber):
    print("Starting the Kafka cluster...")
    subprocess.Popen(startKafkaServerCmd % (kafkaJmxPortNumber, 'server.properties'), shell=True)

    if polling(kafkaPortNumber):
        subprocess.Popen(startKafkaServerCmd % (kafkaJmxPortNumber2, 'server-1.properties'), shell=True)

    if polling(kafkaPortNumber2):
        subprocess.Popen(startKafkaServerCmd % (kafkaJmxPortNumber3, 'server-2.properties'), shell=True)
else:
    print("Something wrong ...")
    input()#raw_input()
```

[java](https://www.oschina.net/p/java)[eagle](https://www.oschina.net/p/eagle)[apache](https://www.oschina.net/p/apache+http+server)[zookeeper](https://www.oschina.net/p/zookeeper)[gitee](https://www.oschina.net/p/gitosc)s