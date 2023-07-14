# [ZooKeeper学习笔记](https://my.oschina.net/jallenkwong/blog/4405741)

[教学视频](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fwww.bilibili.com%2Fvideo%2FBV1PW411r7iP)

[源码](https://gitee.com/jallenkwong/LearnZooKeeper)

| -                                                            | -                                                            | -                                                            |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [01.课程介绍](https://my.oschina.net/jallenkwong/blog/4405741#h2_1) | [02.概述](https://my.oschina.net/jallenkwong/blog/4405741#h2_2) | [03.特点](https://my.oschina.net/jallenkwong/blog/4405741#h2_3) |
| [04.数据结构](https://my.oschina.net/jallenkwong/blog/4405741#h2_4) | [05.应用场景](https://my.oschina.net/jallenkwong/blog/4405741#h2_5) | [06.下载地址](https://my.oschina.net/jallenkwong/blog/4405741#h2_11) |
| [07.本地模式安装](https://my.oschina.net/jallenkwong/blog/4405741#h2_12) | [08.配置参数解读](https://my.oschina.net/jallenkwong/blog/4405741#h2_18) | [09.选举机制](https://my.oschina.net/jallenkwong/blog/4405741#h2_19) |
| [10.节点类型](https://my.oschina.net/jallenkwong/blog/4405741#h2_20) | [11.分布式安装](https://my.oschina.net/jallenkwong/blog/4405741#h2_21) | [12.Shell命令操作](https://my.oschina.net/jallenkwong/blog/4405741#h2_28) |
| [13.Stat结构体](https://my.oschina.net/jallenkwong/blog/4405741#h2_30) | [14.监听器原理](https://my.oschina.net/jallenkwong/blog/4405741#h2_31) | [15.写数据流程](https://my.oschina.net/jallenkwong/blog/4405741#h2_32) |
| [16.创建ZooKeeper客户端](https://my.oschina.net/jallenkwong/blog/4405741#h2_33) | [17.创建一个节点](https://my.oschina.net/jallenkwong/blog/4405741#h2_36) | [18.获取子节点并监听节点变化](https://my.oschina.net/jallenkwong/blog/4405741#h2_37) |
| [19.判断节点是否存在](https://my.oschina.net/jallenkwong/blog/4405741#h2_38) | [20.服务器节点动态上下线案例分析](https://my.oschina.net/jallenkwong/blog/4405741#h2_39) | [21.服务器节点动态上下线案例注册代码](https://my.oschina.net/jallenkwong/blog/4405741#h2_42) |
| [22.服务器节点动态上下线案例全部代码实现](https://my.oschina.net/jallenkwong/blog/4405741#h2_45) | [23.企业面试真题](https://my.oschina.net/jallenkwong/blog/4405741#h2_46) | -                                                            |

## 01.课程介绍

[ZooKeeper官网](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fzookeeper.apache.org%2F)

[ZooKeeper 3.4 Documentation](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fzookeeper.apache.org%2Fdoc%2Fr3.4.14%2Findex.html)

## 02.概述

Zookeeper是一个开源的分布式的，为分布式应用提供协调服务的 Apache 项目。

> ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them, which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.
>
> ZooKeeper是一个集中服务，用于维护配置信息、命名、提供分布式同步和提供组服务。所有这些类型的服务都以某种形式被分布式应用程序使用。每次实现它们时，都要进行大量的工作来修复不可避免的bug和竞争条件。由于实现这类服务的困难，应用程序最初通常会忽略它们，这使得它们在发生变化时变得脆弱，并且难以管理。即使操作正确，在部署应用程序时，这些服务的不同实现也会导致管理复杂性。

Zookeeper从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，Zookeeper就将负责通知已经在Zookeeper上注册的那些观察者做出相应的反应。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/01.png)

1. 服务端启动时去注册信息（创建都是临时节点）
2. 获取到当前在线服务器列表，并且注册监听
3. 服务器节点下线
4. 服务器节点上下线事件通知
5. 重新再去获取服务器列表，并注册监听

一言蔽之：**ZooKeeper = 文件系统 + 通知机制**

## 03.特点

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/02.png)

1. Zookeeper：一个领导者（Leader，多个跟随者（Follower组成的集群。
2. 集群中只要有半数以上节点存活，Zookeeper集群就能正常服务。
3. 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一致的。
4. 更新请求顺序进行，来自同一个Client的更新请求按其发送顺序依次执行。
5. 数据更新原子性，一次数据更新要么成功，要么失败。
6. 实时性，在一定时间范围内，Client能读到最新数据。

## 04.数据结构

ZooKeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一棵树， 每个节点称做一个ZNode。每一个ZNode默认能够存储1MB的数据， 每个ZNode都可以通过其路径唯一标识。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/03.png)

## 05.应用场景

提供的服务包括：

1. 统一命名服务
2. 统一配置管理
3. 统一集群管理
4. 服务器节点动态上下线
5. 软负载均衡
6. ...

### 统一命名服务

在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。

例如：IP不容易记住，而域名容易记住。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/04.png)

### 统一配置管理

1. 分布式环境下，配置文件同步非常常见
    1. 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群
    2. 对配置文件修改后，希望能够快速同步到各个节点上
2. 配置管理可交由ZooKeeper实现
    1. 可将配置信息写入ZooKeeper上的一个Znode
    2. 各个客户端服务器监听这个Znode
    3. 一旦Znode中的数据被修改， ZooKeeper将通知各个客户端服务器

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/05.png)

### 统一集群管理

1. 分布式环境中，实时掌握每个节点的状态是必要的
    1. 可根据节点实时状态做出一些调整
2. ZooKeeper可以实现实时监控节点状态变化
    1. 可将节点信息写入ZooKeeper上的一个ZNode
    2. 监听这个ZNode可获取它的实时状态变化

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/06.png)

### 服务器动态上下线

客户端能实时洞察到服务器上下线的变化。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/01.png)

### 软负载均衡

在Zookeeper中记录每台服务器的访问数， 让访问数最少的服务器去处理最新的客户端请求。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/07.png)

## 06.下载地址

在[下载地址](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fzookeeper.apache.org%2Freleases.html)下载zookeeper-3.4.14。

## 07.本地模式安装

### Linux下安装

略

### Windows下安装

#### 安装前准备

解压后的文件路径：`D:\Apache\Zookeeper\Zookeeper-3.7.0`

#### 配置修改

- 在`D:\Apache\Zookeeper\`路径下创建新文件夹`Data`和 `Logs`
- 在`D:\Apache\Zookeeper\Zookeeper-3.7.0\conf`中的`zoo-sample.conf`更名为`zoo.cfg`
- 用文本编辑器打开`zoo.cfg`，将`dataDir=/tmp/zookeeper`改成`D:\\Apache\\Zookeeper\\Data`

#### 操作 Zookeeper

- 运行`D:\Apache\Zookeeper\Zookeeper-3.7.0\bin\zkServer.cmd`
- 运行`D:\Apache\Zookeeper\Zookeeper-3.7.0\bin\zkCli.cmd`，若命令行窗口含有`Welcome to ZooKeeper!`，表示安装成功，输入`quit`退出`zkCli.cmd`。

## 08.配置参数解读

Zookeeper中的配置文件zoo.cfg中参数含义解读如下：

1. tickTime=2000： **通信心跳数， Zookeeper 服务器与客户端心跳时间，单位毫秒**<br> Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。<br> 它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。 (session的最小超时时间是2*tickTime)
2. initLimit=10： **LF 初始通信时限**<br> 集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。
3. syncLimit=5： **LF 同步通信时限**<br> 集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime， Leader认为Follwer死掉，从服务器列表中删除Follwer。
4. dataDir：**数据文件目录+数据持久化路径**<br> 主要用于保存 Zookeeper 中的数据。
5. clientPort =2181：**客户端连接端口**<br> 监听客户端连接的端口。

## 09.选举机制

**面试重点**

1. 半数机制：集群中半数以上机器存活，集群可用。所以 Zookeeper 适合安装奇数台服务器。例如，5台服务器有3台存活，集群可用，而只有2台存活，集群不可用。
2. Zookeeper 虽然在配置文件中并没有指定 Master 和 Slave。 但是， Zookeeper 工作时，是有一个节点为 Leader，其他则为 Follower， Leader 是通过**内部的选举机制**临时产生的。
3. 以一个简单的例子来说明整个选举的过程。

假设有五台服务器组成的 Zookeeper 集群，它们的 id 从 1-5，同时它们都是最新启动的，也就是没有历史数据，在存放数据量这一点上，都是一样的。假设这些服务器依序启动，来看看会发生什么，如下图所示。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/08.png)

1. 服务器 1 启动， 发起一次选举。服务器 1 投自己一票。此时服务器 1 票数一票，不够半数以上（3 票），选举无法完成，服务器 1 状态保持为 LOOKING；
2. 服务器 2 启动， 再发起一次选举。服务器 1 和 2 分别投自己一票并交换选票信息：此时服务器 1 发现服务器 2 的 ID 比自己目前投票推举的（服务器 1）大，更改选票为推举服务器 2。此时服务器 1 票数 0 票，服务器 2 票数 2 票，没有半数以上结果，选举无法完成，服务器 1， 2 状态保持 LOOKING
3. 服务器 3 启动， 发起一次选举。此时服务器 1 和 2 都会更改选票为服务器 3。此次投票结果：服务器 1 为 0 票，服务器 2 为 0 票，服务器 3 为 3 票。此时服务器 3 的票数已经超过半数，服务器 3 当选 Leader。服务器 1， 2 更改状态为 FOLLOWING，服务器 3 更改状态为 LEADING；
4. 服务器 4 启动， 发起一次选举。此时服务器 1， 2， 3 已经不是 LOOKING 状态，不会更改选票信息。交换选票信息结果：服务器 3 为 3 票，服务器 4 为 1 票。此时服务器 4服从多数，更改选票信息为服务器 3，并更改状态为 FOLLOWING；
5. 服务器 5 启动，同 4 一样当小弟。

## 10.节点类型

- 持久（Persistent）：客户端和服务器端断开连接后， 创建的节点不删除
- 短暂（Ephemeral）：客户端和服务器端断开连接后， 创建的节点自己删除

> ephemeral 英 [ɪˈfemərəl] 美 [ɪˈfemərəl]
> adj. 短暂的;瞬息的

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/09.png)

1. **持久化目录节点** 客户端与Zookeeper断开连接后，该节点依旧存在
2. **持久化顺序编号目录节点** 客户端与Zookeeper断开连接后， 该节点依旧存在， 只是Zookeeper给该节点名称进行顺序编号
3. **临时目录节点** 客户端与Zookeeper断开连接后， 该节点被删除
4. **临时顺序编号目录节点** 客户端与Zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号。

说明：创建znode时设置顺序标识，znode名称后会附加一个值，顺序号是一个单调递增的计数器，由父节点维护

注意：在分布式系统中，顺序号可以被用于为所有的事件进行全局排序， 这样客户端可以通过顺序号推断事件的顺序

## 11.分布式安装

### 在Linux上分布安装

略

### 在Windows上分布安装

在单机上实现伪集群。

#### 创建配置文件

- 在`C:\ZooKeeper\zookeeper-3.4.14\conf\`新建文件夹`cluster`
- 在`C:\ZooKeeper\zookeeper-3.4.14\conf\cluster`创建新文件，命名为`zoo1.cfg`，将下面内容复制到`zoo1.cfg`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=C:\\ZooKeeper\\data\\cluster\\1
dataLogDir=C:\\ZooKeeper\\log\\cluster\\1
clientPort=2182
server.1=127.0.0.1:2887:3887   
server.2=127.0.0.1:2888:3888   
server.3=127.0.0.1:2889:3889
```

- 在`%ZOOKEEPER_HOME%\conf\cluster`创建新文件，命名为`zoo2.cfg`，将下面内容复制到`zoo2.cfg`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=C:\\ZooKeeper\\data\\cluster\\2
dataLogDir=C:\\ZooKeeper\\log\\cluster\\2
clientPort=2183
server.1=127.0.0.1:2887:3887   
server.2=127.0.0.1:2888:3888   
server.3=127.0.0.1:2889:3889
```

- 在`%ZOOKEEPER_HOME%\conf\cluster`创建新文件，命名为`zoo3.cfg`，将下面内容复制到`zoo3.cfg`

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=C:\\ZooKeeper\\data\\cluster\\3
dataLogDir=C:\\ZooKeeper\\log\\cluster\\3
clientPort=2184
server.1=127.0.0.1:2887:3887   
server.2=127.0.0.1:2888:3888   
server.3=127.0.0.1:2889:3889
```

------

**配置参数解读**

```
server.A=B:C:D
```

- A 是一个数字，表示这个是第几号服务器；<br>集群模式下配置一个文件 myid， 这个文件在 dataDir 目录下，这个文件里面有一个数据就是 A 的值， Zookeeper 启动时读取此文件，拿到里面的数据与 zoo.cfg 里面的配置信息比 较从而判断到底是哪个 server。
- B 是这个服务器的IP地址；
- C 是这个服务器 Follower 与集群中的 Leader 服务器交换信息的端口；
- D 是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader，而这个端口就是用来执行选举时服务器相互通信的端口。

#### 创建新文件夹和新文件

- 在`C:\\ZooKeeper\\data\\cluster\\`分别创建名为`1`、`2`、`3`新文件夹，然后在这三个新文件夹分别创建名为`myid`的文件
- 在`C:\\ZooKeeper\\data\\cluster\\1\\myid`文件写入`1`
- 在`C:\\ZooKeeper\\data\\cluster\\2\\myid`文件写入`2`
- 在`C:\\ZooKeeper\\data\\cluster\\3\\myid`文件写入`2`
- 在`C:\\ZooKeeper\\log\\cluster\\`分别创建名为`1`、`2`、`3`新文件夹

#### 修改zkServer.cmd副本

- 将`C:\ZooKeeper\zookeeper-3.4.14\bin\zkServer.cmd`复制成3份在同一级目录中，分别命名为`zkServer1.cmd`、`zkServer2.cmd`、`zkServer3.cmd`
- 用文本编辑器打开`zkServer1.cmd`，在`set ZOOMAIN=org.apache.zookeeper.server.quorum.QuorumPeerMain`下一行添加`set ZOOCFG=..\conf\cluster\zoo1.cfg`
- 用文本编辑器打开`zkServer2.cmd`，在`set ZOOMAIN=org.apache.zookeeper.server.quorum.QuorumPeerMain`下一行添加`set ZOOCFG=..\conf\cluster\zoo2.cfg`
- 用文本编辑器打开`zkServer3.cmd`，在`set ZOOMAIN=org.apache.zookeeper.server.quorum.QuorumPeerMain`下一行添加`set ZOOCFG=..\conf\cluster\zoo3.cfg`

#### 运行zkServerX.cmd

- 在`C:\ZooKeeper\zookeeper-3.4.14\bin`分别运行`zkServer1.cmd`、`zkServer2.cmd`、`zkServer3.cmd`（依次启动的时刻有错误信息，因为你启动server1 的时候 2 和 3 没找到，但是后面都启动了，就没问题了）
- 打开cmd，分别执行`cd C:\ZooKeeper\zookeeper-3.4.14\bin`和`zkCli.cmd -server 127.0.0.1:2182`，启动客户端对其中一个服务端进行访问。
- 另外，在cmd中运行`jps`，可见4个Java程序在运行：

```
C:\ZooKeeper\zookeeper-3.4.14\bin>jps
6064 QuorumPeerMain
6688 Jps
1108 QuorumPeerMain
5884 QuorumPeerMain
```

三个`QuorumPeerMain`就是刚刚启动的三个Server。

## 12.Shell命令操作

| 命令基本语法     | 功能描述                                               |
| ---------------- | ------------------------------------------------------ |
| help             | 显示所有操作命令                                       |
| ls path [watch]  | 使用 ls 命令来查看当前 znode 中所包含的内容            |
| ls2 path [watch] | 查看当前节点数据并能看到更新次数等数据                 |
| create           | 普通创建<br>-s 含有序列<br>-e 临时（重启或者超时消失） |
| get path [watch] | 获得节点的值                                           |
| set              | 设置节点的具体值                                       |
| stat             | 查看节点状态                                           |
| delete           | 删除节点                                               |
| rmr              | 递归删除节点                                           |

### 命令实战

1． 启动客户端

```
zkCli.cmd -server 127.0.0.1:2182
```

------

2．显示所有操作命令

```
help
```

------

3． 查看当前 znode 中所包含的内容

```
ls /
[zookeeper]
```

------

4． 查看当前节点详细数据

```
ls2 /
[zookeeper]
cZxid = 0x0
ctime = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

------

5． 分别创建 2 个普通节点

```
create /sanguo "jinlian"
Created /sanguo

create /sanguo/shuguo "liubei"
Created /sanguo/shuguo
```

------

6．获得节点的值

```
get /sanguo
jinlian
cZxid = 0x300000004
ctime = Sat Jul 18 13:07:51 CST 2020
mZxid = 0x300000004
mtime = Sat Jul 18 13:07:51 CST 2020
pZxid = 0x300000005
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1

get /sanguo/shuguo
liubei
cZxid = 0x300000005
ctime = Sat Jul 18 13:09:21 CST 2020
mZxid = 0x300000005
mtime = Sat Jul 18 13:09:21 CST 2020
pZxid = 0x300000005
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
[zk: 127.0.0.1:2182(CONNECTED) 8]
```

------

7． 创建短暂节点

```
create -e /sanguo/wuguo "zhouyu"
Created /sanguo/wuguo
```

a. 在当前客户端是能查看到的

```
ls /sanguo
[wuguo, shuguo]
```

b. 退出当前客户端然后再重启客户端

```
quit

zkCli.cmd -server 127.0.0.1:2182
```

c. 再次查看根目录下短暂节点已经删除

```
ls /sanguo
[shuguo]
```

------

8． 创建带序号的节点

a. 先创建一个普通的根节点`/sanguo/weiguo`

```
create /sanguo/weiguo "caocao"
Created /sanguo/weiguo
```

b. 创建带序号的节点

```
[zk: 127.0.0.1:2182(CONNECTED) 2] create -s /sanguo/weiguo/xiaoqiao "meinv"
Created /sanguo/weiguo/xiaoqiao0000000000
[zk: 127.0.0.1:2182(CONNECTED) 3] create -s /sanguo/weiguo/daqiao "meinv"
Created /sanguo/weiguo/daqiao0000000001
[zk: 127.0.0.1:2182(CONNECTED) 4] create -s /sanguo/weiguo/sunshangxiang "meinv"
Created /sanguo/weiguo/sunshangxiang0000000002
```

如果原来没有序号节点，序号从 0 开始依次递增。 如果原节点下已有 2 个节点，则再排序时从 2 开始，以此类推。

------

9． 修改节点数据值

```
set /sanguo/weiguo "simayi"
```

------

10． 节点的值变化监听

a. 在 127.0.0.1:2182 上注册监听/sanguo 节点数据变化

```
[zk: 127.0.0.1:2182(CONNECTED) 7] get /sanguo watch
jinlian
cZxid = 0x300000004
ctime = Sat Jul 18 13:07:51 CST 2020
mZxid = 0x300000004
mtime = Sat Jul 18 13:07:51 CST 2020
pZxid = 0x300000009
cversion = 4
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 2
```

b. 在 127.0.0.1:2183 上修改/sanguo 节点的数据

```
[zk: 127.0.0.1:2183(CONNECTED) 1] set /sanguo "tongyi"
cZxid = 0x300000004
ctime = Sat Jul 18 13:07:51 CST 2020
mZxid = 0x30000000f
mtime = Sat Jul 18 13:46:52 CST 2020
pZxid = 0x300000009
cversion = 4
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 2
[zk: 127.0.0.1:2183(CONNECTED) 2]
```

c. 观察 127.0.0.1:2182 收到数据变化的监听

```
WATCHER::

WatchedEvent state:SyncConnected type:NodeDataChanged path:/sanguo
```

------

11． 节点的子节点变化监听（路径变化）

a. 在 127.0.0.1:2182 上注册监听/sanguo 节点的子节点变化

```
ls /sanguo watch
[shuguo, weiguo]
```

b. 在 127.0.0.1:2183 上修改/sanguo 节点上创建子节点

```
create /sanguo/jin "simayi"
Created /sanguo/jin
```

c. 观察 127.0.0.1:2182 收到子节点变化的监听

```
WATCHER::

WatchedEvent state:SyncConnected type:NodeChildrenChanged path:/sanguo
```

------

12． 删除节点

```
delete /sanguo/jin

get /sanguo/jin
Node does not exist: /sanguo/jin
```

------

13．递归删除节点

```
rmr /sanguo/shuguo

get /sanguo/shuguo
Node does not exist: /sanguo/shuguo
```

------

14．查看节点状态

```
stat /sanguo
cZxid = 0x300000004
ctime = Sat Jul 18 13:07:51 CST 2020
mZxid = 0x30000000f
mtime = Sat Jul 18 13:46:52 CST 2020
pZxid = 0x300000012
cversion = 7
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 1
```

## 13.Stat结构体

1. czxid - 创建节点的事务 zxid <br>每次修改 ZooKeeper 状态都会收到一个 zxid 形式的时间戳，也就是 ZooKeeper 事务 ID。事务 ID 是 ZooKeeper 中所有修改总的次序。每个修改都有唯一的 zxid，如果 zxid1 小于 zxid2，那么 zxid1 在 zxid2 之前发生。
2. ctime - znode 被创建的毫秒数(从 1970 年开始)
3. mzxid - znode 最后更新的事务 zxid
4. mtime - znode 最后修改的毫秒数(从 1970 年开始)
5. pZxid-znode 最后更新的子节点 zxid
6. cversion - znode 子节点变化号， znode 子节点修改次数
7. dataversion - znode 数据变化号
8. aclVersion - znode 访问控制列表的变化号
9. ephemeralOwner- 如果是临时节点，这个是 znode 拥有者的 session id。如果不是临时节点则是 0。
10. dataLength- znode 的数据长度
11. numChildren - znode 子节点数量

## 14.监听器原理

**面试重点**

监听原理详解：

1. 首先要有一个main()线程
2. 在main线程中创建Zookeeper客户端， 这时就会创建两个线程， 一个负责网络连接通信（ connet ）， 一个负责监听（ listener ）。
3. 通过connect线程将注册的监听事件发送给Zookeeper。
4. 在Zookeeper的注册监听器列表中将注册的监听事件添加到列表中。
5. Zookeeper监听到有数据或路径变化， 就会将这个消息发送给listener线程。
6. listener线程内部调用了process()方法。

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/10.png)

常见的监听:

1. 监听节点数据的变化 `get path [watch]`
2. 监听子节点增减的变化 `ls path [watch]`

## 15.写数据流程

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/11.png)

1. Client 向 ZooKeeper 的Server1 上写数据，发送一个写请求。
2. 如果Server1不是Leader，那么Server1 会把接受到的请求进一步转发给Leader，因为每个ZooKeeper的Server里面有一个是Leader。这个Leader 会将写请求广播给各个Server， 比如Server1和Server2，各个**Server会将该写请求加入待写队列**，并向Leader发送成功信息。
3. 当Leader收到半数以上 Server 的成功信息， 说明该写操作可以执行。Leader会向各个Server 发送提交信息，各个Server收到信息后会**落实队列里的写请求， 此时写成功**。
4. Server1会进一步通知 Client 数据写成功了，这时就认为整个写操作成功

## 16.创建ZooKeeper客户端

### 在 Eclipse 环境搭建

- 创建一个 Maven 工程
- 为 [pom.xml](https://my.oschina.net/jallenkwong/blog/pom.xml) 添加关键依赖

```xml
<dependencies>
	<dependency>
		<groupId>org.apache.logging.log4j</groupId>
		<artifactId>log4j-core</artifactId>
		<version>2.8.2</version>
	</dependency>
	
	<dependency>
		<groupId>org.apache.zookeeper</groupId>
		<artifactId>zookeeper</artifactId>
		<version>3.4.10</version>
	</dependency>
</dependencies>
```

- 需要在项目的 `src/main/resources` 目录下，新建一个文件，命名为[log4j.properties](https://my.oschina.net/jallenkwong/blog/src/main/resources/log4j.properties)，在文件中填入如下内容：

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

### 创建 ZooKeeper 客户端

[ZooKeeperTest源码](https://my.oschina.net/jallenkwong/blog/src/test/java/com/lun/ZooKeeperTest.java)

```java
public class ZooKeeperTest {
	private static String connectString = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	// @Test
	@Before
	public void init() throws Exception {

		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			// 收到事件通知后的回调函数（用户的业务逻辑）
			public void process(WatchedEvent event) {
				System.out.println(event.getType() + "--" + event.getPath());

				// 再次启动监听
				try {
					List<String> children = zkClient.getChildren("/", true);
					for (String child : children) {
						System.out.println(child);
					}
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}
}
```

## 17.创建一个节点

```java
// 创建子节点
@Test
public void create() throws Exception {
	// 参数 1：要创建的节点的路径； 参数 2：节点数据 ； 参数 3：节点权限 ；参数 4：节点的类型
	String nodeCreated = zkClient.create("/root", "root".getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);

	System.out.println(nodeCreated);
}
```

## 18.获取子节点并监听节点变化

```java
// 获取子节点
@Test
public void getChildren() throws Exception {
	List<String> children = zkClient.getChildren("/", true);
	for (String child : children) {
		System.out.println(child);
	}

	// 延时阻塞
	Thread.sleep(Long.MAX_VALUE);
}
```

## 19.判断节点是否存在

```java
// 判断 znode 是否存在
@Test
public void exist() throws Exception {
	Stat stat = zkClient.exists("/eclipse", false);
	System.out.println(stat == null ? "not exist" : "exist");
}
```

## 20.服务器节点动态上下线案例分析

### 需求

某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知到主节点服务器的上下线。

### 需求分析

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/01.png)

## 21.服务器节点动态上下线案例注册代码

### 先在集群上创建/servers节点

```
[zk: 127.0.0.1:2182(CONNECTED) 6] create /servers "servers"
Created /servers
```

### 服务器端向 Zookeeper 注册代码

[DistributeServer源码](https://my.oschina.net/jallenkwong/blog/src/main/java/com/lun/DistributeServer.java)

```java
public class DistributeServer {
	private static String connectString = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	private String parentNode = "/servers";

	// 创建到 zk 的客户端连接
	public void getConnect() throws IOException {
		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
			}
		});
	}

	// 注册服务器
	public void registServer(String hostname) throws Exception {
		String create = zkClient.create(parentNode + "/server", hostname.getBytes(), Ids.OPEN_ACL_UNSAFE,
				CreateMode.EPHEMERAL_SEQUENTIAL);
		System.out.println(hostname + " is online " + create);
	}

	// 业务功能
	public void business(String hostname) throws Exception {
		System.out.println(hostname + " is working ...");
		Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {
		// 1 获取 zk 连接
		DistributeServer server = new DistributeServer();
		server.getConnect();
		// 2 利用 zk 连接注册服务器信息
		server.registServer(args[0]);
		// 3 启动业务功能
		server.business(args[0]);
	}

}
```

## 22.服务器节点动态上下线案例全部代码实现

[DistributeClient源码](https://my.oschina.net/jallenkwong/blog/src/main/java/com/lun/DistributeClient.java)

```java
public class DistributeClient {
	private static String connectString = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
	private static int sessionTimeout = 2000;
	private ZooKeeper zkClient = null;

	private String parentNode = "/servers";

	// 创建到 zkClient 的客户端连接
	public void getConnect() throws IOException {
		zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
			public void process(WatchedEvent event) {
				// 再次启动监听
				try {
					getServerList();
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		});
	}

	// 获取服务器列表信息
	public void getServerList() throws Exception {
		// 1 获取服务器子节点信息，并且对父节点进行监听
		List<String> children = zkClient.getChildren(parentNode, true);
		// 2 存储服务器信息列表
		ArrayList<String> servers = new ArrayList<>();
		// 3 遍历所有节点，获取节点中的主机名称信息
		for (String child : children) {
			byte[] data = zkClient.getData(parentNode + "/" + child, false, null);
			servers.add(new String(data));
		}
		// 4 打印服务器列表信息
		System.out.println(servers);
	}

	// 业务功能
	public void business() throws Exception {
		System.out.println("client is working ...");
		Thread.sleep(Long.MAX_VALUE);
	}

	public static void main(String[] args) throws Exception {
		// 1 获取 zk 连接
		DistributeClient client = new DistributeClient();
		client.getConnect();
		// 2 获取 servers 的子节点信息，从中获取服务器信息列表
		client.getServerList();
		// 3 业务进程启动
		client.business();
	}

}
```

## 23.企业面试真题

**请简述 ZooKeeper 的选举机制**

半数机制，myid最大的为Leader

------

**ZooKeeper 的监听原理是什么？**

![img](https://gitee.com/jallenkwong/LearnZooKeeper/raw/master/image/10.png)

------

**ZooKeeper 的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？**

1. 部署方式单机模式、集群模式
2. 角色： Leader 和 Follower
3. 集群最少需要机器数： 3

------

**ZooKeeper 的常用命令**

CRUD：

1. C
    - create
2. R
    - ls
    - ls2
    - get
    - stat
3. U
    - set
4. D
    - delete
    - rmr
5. help

[gitee](https://www.oschina.net/p/gitosc)[log4j](https://my.oschina.net/jallenkwong?q=log4j)[zkclient](https://www.oschina.net/p/zkclient)[java](https://www.oschina.net/p/java)[apache](https://www.oschina.net/p/apache+http+server)

© 著作权归作者所有

举报

打赏

1 赞

6 收藏

分享

### 作者的其它热门文章

[Kafka学习笔记](https://my.oschina.net/jallenkwong/blog/4449224)

[【清华大学】《逻辑学概论》笔记](https://my.oschina.net/jallenkwong/blog/4544103)

[Redis学习笔记](https://my.oschina.net/jallenkwong/blog/4411044)

[《Java8实战》笔记（10）：用Optional取代null](https://my.oschina.net/jallenkwong/blog/4503910)