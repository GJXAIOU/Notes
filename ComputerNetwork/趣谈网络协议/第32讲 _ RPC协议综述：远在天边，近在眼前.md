# 第32讲 \| RPC协议综述：远在天边，近在眼前

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/48/9c/487fd48164a2cd9e9bc09931e4b4399c.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/97/2d/97718e118ab62bfaa6283e753ee9392d.mp3" type="audio/mpeg"></audio>

前面我们讲了容器网络如何实现跨主机互通，以及微服务之间的相互调用。

![](<https://static001.geekbang.org/resource/image/a0/0d/a0763d50fc4e8dcec37ae25a2f6cc60d.jpeg?wh=1582*1080>)

网络是打通了，那服务之间的互相调用，该怎么实现呢？你可能说，咱不是学过Socket吗。服务之间分调用方和被调用方，我们就建立一个TCP或者UDP的连接，不就可以通信了？

![](<https://static001.geekbang.org/resource/image/87/ea/87c8ae36ae1b42653565008fc47aceea.jpg?wh=1626*2172>)

你仔细想一下，这事儿没这么简单。我们就拿最简单的场景，客户端调用一个加法函数，将两个整数加起来，返回它们的和。

如果放在本地调用，那是简单的不能再简单了，只要稍微学过一种编程语言，三下五除二就搞定了。但是一旦变成了远程调用，门槛一下子就上去了。

首先你要会Socket编程，至少先要把咱们这门网络协议课学一下，然后再看N本砖头厚的Socket程序设计的书，学会咱们学过的几种Socket程序设计的模型。这就使得本来大学毕业就能干的一项工作，变成了一件五年工作经验都不一定干好的工作，而且，搞定了Socket程序设计，才是万里长征的第一步。后面还有很多问题呢！

## 如何解决这五个问题？

### 问题一：如何规定远程调用的语法？

客户端如何告诉服务端，我是一个加法，而另一个是乘法。我是用字符串“add”传给你，还是传给你一个整数，比如1表示加法，2表示乘法？服务端该如何告诉客户端，我的这个加法，目前只能加整数，不能加小数，不能加字符串；而另一个加法“add1”，它能实现小数和整数的混合加法。那返回值是什么？正确的时候返回什么，错误的时候又返回什么？

<!-- [[[read_end]]] -->

### 问题二：如果传递参数？

我是先传两个整数，后传一个操作符“add”，还是先传操作符，再传两个整数？是不是像咱们数据结构里一样，如果都是UDP，想要实现一个逆波兰表达式，放在一个报文里面还好，如果是TCP，是一个流，在这个流里面，如何将两次调用进行分界？什么时候是头，什么时候是尾？把这次的参数和上次的参数混了起来，TCP一端发送出去的数据，另外一端不一定能一下子全部读取出来。所以，怎么才算读完呢？

### 问题三：如何表示数据？

在这个简单的例子中，传递的就是一个固定长度的int值，这种情况还好，如果是变长的类型，是一个结构体，甚至是一个类，应该怎么办呢？如果是int，不同的平台上长度也不同，该怎么办呢？

在网络上传输超过一个Byte的类型，还有大端Big Endian和小端Little Endian的问题。

假设我们要在32位四个Byte的一个空间存放整数1，很显然只要一个Byte放1，其他三个Byte放0就可以了。那问题是，最后一个Byte放1呢，还是第一个Byte放1呢？或者说1作为最低位，应该是放在32位的最后一个位置呢，还是放在第一个位置呢？

最低位放在最后一个位置，叫作Little Endian，最低位放在第一个位置，叫作Big Endian。TCP/IP协议栈是按照Big Endian来设计的，而X86机器多按照Little Endian来设计的，因而发出去的时候需要做一个转换。

### 问题四：如何知道一个服务端都实现了哪些远程调用？从哪个端口可以访问这个远程调用？

假设服务端实现了多个远程调用，每个可能实现在不同的进程中，监听的端口也不一样，而且由于服务端都是自己实现的，不可能使用一个大家都公认的端口，而且有可能多个进程部署在一台机器上，大家需要抢占端口，为了防止冲突，往往使用随机端口，那客户端如何找到这些监听的端口呢？

### 问题五：发生了错误、重传、丢包、性能等问题怎么办？

本地调用没有这个问题，但是一旦到网络上，这些问题都需要处理，因为网络是不可靠的，虽然在同一个连接中，我们还可通过TCP协议保证丢包、重传的问题，但是如果服务器崩溃了又重启，当前连接断开了，TCP就保证不了了，需要应用自己进行重新调用，重新传输会不会同样的操作做两遍，远程调用性能会不会受影响呢？

## 协议约定问题

看到这么多问题，你是不是想起了我[第一节](<https://time.geekbang.org/column/article/7581>)讲过的这张图。

![](<https://static001.geekbang.org/resource/image/7b/33/7be56272a7e738b6cfe5bcbf658c3933.jpg?wh=2643*380>)

本地调用函数里有很多问题，比如词法分析、语法分析、语义分析等等，这些编译器本来都能帮你做了。但是在远程调用中，这些问题你都需要重新操心。

很多公司的解决方法是，弄一个核心通信组，里面都是Socket编程的大牛，实现一个统一的库，让其他业务组的人来调用，业务的人不需要知道中间传输的细节。通信双方的语法、语义、格式、端口、错误处理等，都需要调用方和被调用方开会协商，双方达成一致。一旦有一方改变，要及时通知对方，否则通信就会有问题。

可是不是每一个公司都有这种大牛团队，往往只有大公司才配得起，那有没有已经实现好的框架可以使用呢？

当然有。一个大牛Bruce Jay Nelson写了一篇论文[Implementing Remote Procedure Calls](<http://www.cs.cmu.edu/~dga/15-712/F07/papers/birrell842.pdf>)，定义了RPC的调用标准。后面所有RPC框架，都是按照这个标准模式来的。

![](<https://static001.geekbang.org/resource/image/29/db/2933bbd1ee6471b6de3824bb86f6d0db.jpg?wh=1999*707>)

当客户端的应用想发起一个远程调用时，它实际是通过本地调用本地调用方的Stub。它负责将调用的接口、方法和参数，通过约定的协议规范进行编码，并通过本地的RPCRuntime进行传输，将调用网络包发送到服务器。

服务器端的RPCRuntime收到请求后，交给提供方Stub进行解码，然后调用服务端的方法，服务端执行方法，返回结果，提供方Stub将返回结果编码后，发送给客户端，客户端的RPCRuntime收到结果，发给调用方Stub解码得到结果，返回给客户端。

这里面分了三个层次，对于用户层和服务端，都像是本地调用一样，专注于业务逻辑的处理就可以了。对于Stub层，处理双方约定好的语法、语义、封装、解封装。对于RPCRuntime，主要处理高性能的传输，以及网络的错误和异常。

最早的RPC的一种实现方式称为Sun RPC或ONC RPC。Sun公司是第一个提供商业化RPC库和 RPC编译器的公司。这个RPC框架是在NFS协议中使用的。

NFS（Network File System）就是网络文件系统。要使NFS成功运行，要启动两个服务端，一个是mountd，用来挂载文件路径；一个是nfsd，用来读写文件。NFS可以在本地mount一个远程的目录到本地的一个目录，从而本地的用户在这个目录里面写入、读出任何文件的时候，其实操作的是远程另一台机器上的文件。

操作远程和远程调用的思路是一样的，就像操作本地一样。所以NFS协议就是基于RPC实现的。当然无论是什么RPC，底层都是Socket编程。

![](<https://static001.geekbang.org/resource/image/2a/eb/2a0fd84c2d3dced623511e2a5226d0eb.jpg?wh=2366*1704>)

XDR（External Data Representation，外部数据表示法）是一个标准的数据压缩格式，可以表示基本的数据类型，也可以表示结构体。

这里是几种基本的数据类型。

![](<https://static001.geekbang.org/resource/image/4a/af/4a649954fea1cee22fcfa8bdb34c03af.jpg?wh=3514*1668>)

在RPC的调用过程中，所有的数据类型都要封装成类似的格式。而且RPC的调用和结果返回，也有严格的格式。

- XID唯一标识一对请求和回复。请求为0，回复为1。

- RPC有版本号，两端要匹配RPC协议的版本号。如果不匹配，就会返回Deny，原因就是RPC\_MISMATCH。

- 程序有编号。如果服务端找不到这个程序，就会返回PROG\_UNAVAIL。

- 程序有版本号。如果程序的版本号不匹配，就会返回PROG\_MISMATCH。

- 一个程序可以有多个方法，方法也有编号，如果找不到方法，就会返回PROC\_UNAVAIL。

- 调用需要认证鉴权，如果不通过，则Deny。

- 最后是参数列表，如果参数无法解析，则返回GABAGE\_ARGS。


<!-- -->

![](<https://static001.geekbang.org/resource/image/c7/65/c724675527afdbd43964bdf24684fa65.jpg?wh=3777*1506>)

为了可以成功调用RPC，在客户端和服务端实现RPC的时候，首先要定义一个双方都认可的程序、版本、方法、参数等。

![](<https://static001.geekbang.org/resource/image/5c/58/5c3ebb31ac4415d7895247bf8758fa58.jpg?wh=3849*1878>)

如果还是上面的加法，则双方约定为一个协议定义文件，同理如果是NFS、mount和读写，也会有类似的定义。

有了协议定义文件，ONC RPC会提供一个工具，根据这个文件生成客户端和服务器端的Stub程序。

![](<https://static001.geekbang.org/resource/image/27/b9/27dc1ccd0481408055c87e0e5d8b02b9.jpg?wh=4035*2151>)

最下层的是XDR文件，用于编码和解码参数。这个文件是客户端和服务端共享的，因为只有双方一致才能成功通信。

在客户端，会调用clnt\_create创建一个连接，然后调用add\_1，这是一个Stub函数，感觉是在调用本地一样。其实是这个函数发起了一个RPC调用，通过调用clnt\_call来调用ONC RPC的类库，来真正发送请求。调用的过程非常复杂，一会儿我详细说这个。

当然服务端也有一个Stub程序，监听客户端的请求，当调用到达的时候，判断如果是add，则调用真正的服务端逻辑，也即将两个数加起来。

服务端将结果返回服务端的Stub，这个Stub程序发送结果给客户端，客户端的Stub程序正在等待结果，当结果到达客户端Stub，就将结果返回给客户端的应用程序，从而完成整个调用过程。

有了这个RPC的框架，前面五个问题中的前三个“如何规定远程调用的语法？”“如何传递参数？”以及“如何表示数据？”基本解决了，这三个问题我们统称为**协议约定问题**。

## 传输问题

但是错误、重传、丢包、性能等问题还没有解决，这些问题我们统称为**传输问题**。这个就不用Stub操心了，而是由ONC RPC的类库来实现。这是大牛们实现的，我们只要调用就可以了。

![](<https://static001.geekbang.org/resource/image/33/e4/33e1afe4a79e81096e09b850424930e4.jpg?wh=3862*1882>)

在这个类库中，为了解决传输问题，对于每一个客户端，都会创建一个传输管理层，而每一次RPC调用，都会是一个任务，在传输管理层，你可以看到熟悉的队列机制、拥塞窗口机制等。

由于在网络传输的时候，经常需要等待，因而同步的方式往往效率比较低，因而也就有Socket的异步模型。为了能够异步处理，对于远程调用的处理，往往是通过状态机来实现的。只有当满足某个状态的时候，才进行下一步，如果不满足状态，不是在那里等，而是将资源留出来，用来处理其他的RPC调用。

![](<https://static001.geekbang.org/resource/image/02/f5/0258775aac1126735504c9a6399745f5.jpg?wh=4045*2276>)

从这个图可以看出，这个状态转换图还是很复杂的。

首先，进入起始状态，查看RPC的传输层队列中有没有空闲的位置，可以处理新的RPC任务。如果没有，说明太忙了，或直接结束或重试。如果申请成功，就可以分配内存，获取服务的端口号，然后连接服务器。

连接的过程要有一段时间，因而要等待连接的结果，会有连接失败，或直接结束或重试。如果连接成功，则开始发送RPC请求，然后等待获取RPC结果，这个过程也需要一定的时间；如果发送出错，可以重新发送；如果连接断了，可以重新连接；如果超时，可以重新传输；如果获取到结果，就可以解码，正常结束。

这里处理了连接失败、重试、发送失败、超时、重试等场景。不是大牛真写不出来，因而实现一个RPC的框架，其实很有难度。

## 服务发现问题

传输问题解决了，我们还遗留一个问题，就是问题四“如何找到RPC服务端的那个随机端口”。这个问题我们称为服务发现问题。在ONC RPC中，服务发现是通过portmapper实现的。

![](<https://static001.geekbang.org/resource/image/2a/7c/2aff190d1f878749d2a5bd73228ca37c.jpg?wh=2382*2074>)

portmapper会启动在一个众所周知的端口上，RPC程序由于是用户自己写的，会监听在一个随机端口上，但是RPC程序启动的时候，会向portmapper注册。客户端要访问RPC服务端这个程序的时候，首先查询portmapper，获取RPC服务端程序的随机端口，然后向这个随机端口建立连接，开始RPC调用。从图中可以看出，mount命令的RPC调用，就是这样实现的。

## 小结

好了，这一节就到这里，我们来总结一下。

- 远程调用看起来用Socket编程就可以了，其实是很复杂的，要解决协议约定问题、传输问题和服务发现问题。

- 大牛Bruce Jay Nelson的论文、早期ONC RPC框架，以及NFS的实现，给出了解决这三大问题的示范性实现，也即协议约定要公用协议描述文件，并通过这个文件生成Stub程序；RPC的传输一般需要一个状态机，需要另外一个进程专门做服务发现。


<!-- -->

最后，给你留两个思考题。

1. 在这篇文章中，mount的过程是通过系统调用，最终调用到RPC层。一旦mount完毕之后，客户端就像写入本地文件一样写入NFS了，这个过程是如何触发RPC层的呢？

2. ONC RPC是早期的RPC框架，你觉得它有哪些问题呢？


<!-- -->

我们的专栏更新到第32讲，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送**学习奖励礼券**和我整理的**独家网络协议知识图谱**。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(38)

- ![img](https://static001.geekbang.org/account/avatar/00/10/a8/1b/ced1d171.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  空档滑行

  1.rpc调用是在进行读写操作时，调用的操作系统的读写接口，nfs对接口做了实现，实现的代码里封装了rpc 2.需要调用双方有接口描述文件，有文件就需要双方要做信息交换，所以客户端和服务端不是完全透明的

  2018-07-30

  **1

  **37

- ![img](%E7%AC%AC32%E8%AE%B2%20_%20RPC%E5%8D%8F%E8%AE%AE%E7%BB%BC%E8%BF%B0%EF%BC%9A%E8%BF%9C%E5%9C%A8%E5%A4%A9%E8%BE%B9%EF%BC%8C%E8%BF%91%E5%9C%A8%E7%9C%BC%E5%89%8D.resource/resize,m_fill,h_34,w_34-1662308227296-10047.jpeg)

  吴军旗^_^

  越往后人的留言越少， 看来成为大牛的路上会越来越孤单

  作者回复: 坚持到最后，你就是大牛

  2019-07-10

  **2

  **21

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/62/cd7d8b3b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  叹息无门

  1.应用在读写文件时，会创建文件描述符，NFS Client会将文件描述符的操作代理成RPC请求。 2.XDR有严格的格式限制，两端必须完全匹配，无法支持灵活数据格式的传递。

  2018-07-30

  **

  **19

- ![img](%E7%AC%AC32%E8%AE%B2%20_%20RPC%E5%8D%8F%E8%AE%AE%E7%BB%BC%E8%BF%B0%EF%BC%9A%E8%BF%9C%E5%9C%A8%E5%A4%A9%E8%BE%B9%EF%BC%8C%E8%BF%91%E5%9C%A8%E7%9C%BC%E5%89%8D.resource/resize,m_fill,h_34,w_34-1662308227297-10049.jpeg)

  mgxian

  1.nfs挂载的时候指定了文件系统类型 当应用对文件进行read write等操作时 会调用系统底层的vfs文件系统相关函数， nfs 实现了 vfs规定的 接口函数，调用vfs相关函数时 vfs其实会调用nfs的实现 实现访问远程文件系统 2.不支持多语言 

  2018-07-30

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/11/a2/12/c429f550.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  何重阳

  能不能收我为徒😂

  2018-08-03

  **1

  **5

- ![img](%E7%AC%AC32%E8%AE%B2%20_%20RPC%E5%8D%8F%E8%AE%AE%E7%BB%BC%E8%BF%B0%EF%BC%9A%E8%BF%9C%E5%9C%A8%E5%A4%A9%E8%BE%B9%EF%BC%8C%E8%BF%91%E5%9C%A8%E7%9C%BC%E5%89%8D.resource/resize,m_fill,h_34,w_34-1662308227298-10051.jpeg)

  忆水寒

  刘老师，我们目前的分布式系统采用以下方式。我们实现了一种中间件，每个进程（客户端）要与其他进程通信，就要到中间件注册（注册了自己进程的一个ID，任务名称，还有一个消息队列），然后将消息用google的protobuf封装进行传输（因为这种序列化的效率高）。在其他进程中接收到消息，会解析消息id，然后根据定义好的格式去取内容。这样也算RPC调用吧？

  作者回复: 是的

  2018-07-30

  **

  **5

- ![img](%E7%AC%AC32%E8%AE%B2%20_%20RPC%E5%8D%8F%E8%AE%AE%E7%BB%BC%E8%BF%B0%EF%BC%9A%E8%BF%9C%E5%9C%A8%E5%A4%A9%E8%BE%B9%EF%BC%8C%E8%BF%91%E5%9C%A8%E7%9C%BC%E5%89%8D.resource/resize,m_fill,h_34,w_34-1662308227298-10052.jpeg)

  朽木自雕

  刘老师，我都认真的看了就是有的看不太懂，但是我真的好期待您的那个知识图谱，我觉得这个有助于对知识的加深理解，因为我认为这种图谱被我所喜欢的原因是它属于空间的结构，我自己这么认为的。

  2018-07-30

  **

  **4

- ![img](%E7%AC%AC32%E8%AE%B2%20_%20RPC%E5%8D%8F%E8%AE%AE%E7%BB%BC%E8%BF%B0%EF%BC%9A%E8%BF%9C%E5%9C%A8%E5%A4%A9%E8%BE%B9%EF%BC%8C%E8%BF%91%E5%9C%A8%E7%9C%BC%E5%89%8D.resource/resize,m_fill,h_34,w_34-1662308227298-10053.jpeg)

  小谢同学

  作为一名不会coding的从业者，想问刘老师几个基础问题，首先是文中提到的onc rpc框架，就是包含了stub（编解码）+传输（类库）+服务发现的一套东西？那么目前主流的rpc框架，比如dubbo也是实现了这些功能的集大成者？另外一个问题就是，thrift 和protobuf 我理解只是实现了rpc编解码环节的工作，也就是所说的序列化与反序列化，对么？

  作者回复: 是的，rpc的几个时代，就是这样演进过来的

  2018-07-31

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/16/72/85/c337e9a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  老兵

  \1. mounted NFS会创建RPC的client端调用的映射，当NFS有写入时，就根据这个映射将结果发给client的stub。 2. 每次添加新的RPC方法，会需要重新部署（启动）client和server。

  2020-06-27

  **

  **2

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/pTD8nS0SsORKiaRD3wB0NK9Bpd0wFnPWtYLPfBRBhvZ68iaJErMlM2NNSeEibwQfY7GReILSIYZXfT9o8iaicibcyw3g/132)

  雷刚

  \1. nfs 和 hdfs 一样都是分布式文件系统。当往往 nfs 目录中读写时，就会和这个目录对应的远程服务器建立 RPC 通道，我们看这是像往本地文件读写，实际是通过 RPC 往远程服务器读写。 2. 现代的 RPC 框架越来越像一个生态（如 Dubbo），不仅仅是一个 RPC 框架，更是一整套微服务的解决方案。如分布式配置（Nacos）、服务注册与发现（Nacos）、服务调用、负载均衡、限流与熔断、网关、分布式消息中间件等。

  2020-04-10

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/f6/32/358f9411.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  梦想启动的蜗牛

  最低位放在最后不是大端模式嘛？

  2018-08-09

  **1

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/ce/a8c8b5e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jason

  这篇我看懂了哈哈。工作中一涉及到rpc，我简直是thrift的铁杆粉丝，Google的protobuf也不错，但其中的原理的我并没深究。通过这篇，我学到了rpc的架构原理，赞。至于nfs，其实工作中也有用过，但仅仅是用而已，没有深究其中的奥妙，期待超哥下篇的解答。

  2018-07-31

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/b8/31c7e110.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LVM_23

  终于来了些看得懂和日常已经使用的知识了，手动狗头

  2020-11-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/a5/98/a65ff31a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  djfhchdh

  问题2：如果约定的协议内容发生变化，比如，增加参数、增加方法等，都需要重新生成stub程序，并重启rpc的客户端和服务端，比较麻烦

  2020-08-25

  **

  **2

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLvdWoCic6ItzibF8ia8vrUTRuyj6AT3tg5f4QicIK0jTIFheJ6274ZkibuRLFP1NXG3jibv5TiaSKNoJpLw/132)

  Geek_37984c

  老师GABAGE_ARGS 是写错了吗 GARBAGE_ARGS?

  作者回复: 是的

  2019-08-24

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/78/ac/e5e6e7f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古夜

  看完知道一点zookeeper这个注册中心为啥要注册了，懵懵懂懂的也算是知道一点了😂

  2019-04-25

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/1a/e5/6899701e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  favorlm

  rpc，现在用框架已经简单了很多

  2018-07-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/59/dc9bbb21.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Join

  RPC看得比较爽了，学习了

  2021-10-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/b2/91/cf0de36e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  saltedfish

  问题二：如果传递参数？应该是如何传递参数吧？

  2021-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/3d/c5/f43fa619.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  🍀柠檬鱼也是鱼

  RPC runtime是指什么呢，网上没有查到诶

  2020-11-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a5/98/a65ff31a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  djfhchdh

  问题1：NFS是一个文件系统，在mount成功之后，对于这个文件系统的读写，都由内核中NFS的读写操作接口实现，在这些接口中封装了对nfsd的rpc调用。

  2020-08-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/85/87/727142bc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  MacBao

  老师您好，Webservice和 WCF也算是一种RPC吗？

  2020-04-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  两个程序的通信需要Socket进程来保证，这个通信过程中又需要协议的达成、服务发现等知识来保证正常运行。

  2020-03-26

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/N0NACGUr8dNAbN6BdiagPHBaB0EnyDsI9zWpwJteqTY38apOEnTOA7JkBAQnzYKJBgxu3Q8YMUILwLAB6camn4w/132)

  Swing

  em，安卓的IPC机制。。。Binder机制，里面说道的stub，以及 自动化生成的aidl接口。。 还有 里面的ServiceManager。。。 看来都是借鉴这里的 实现思路

  2020-03-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/7a/32/27a8572a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  渣渣

  第一个问题：mount完毕后，通过；portmapper发现服务，从而调用。第二个问题：使用较为复杂吧，需要客户端和服务器有较多实现。我理解在分布式系统中，机器和机器之间相互连接，每一台机器既是客户端也是服务器吧，都要实现RPC，老师是这样吗

  2020-02-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d1/29/1b1234ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  DFighting

  rpc原来和网络是这样联系在了一起的，虽然没自己研究过rpc框架，但是很多的方法都是通用的，比如服务注册、状态机等，虽然只是入门课程，但是广度和深度依然不小(里面好多的实现细节)，这可比读单纯的理论书有价值多了!

  2020-01-02

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  zKerry

  哦...原来是有示范性标准的啊

  作者回复: 是的

  2019-09-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/85/0a/e564e572.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  N_H

  老师，您好，文章里面，“RPC程序是用户自己写的，会监听在一个随机端口上”。我们公司用的grpc服务，都会自己指明一个端口号

  作者回复: 当然可以指定。我这里的RPC还比较老。

  2019-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/49/3b/d214038f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  java_zhao

  想问一下 我们平时的程序不都是指定端口的吗 为什么还会随机端口 用portmapper呢？

  作者回复: portmapper作为一个服务发现中心，不同端口的都可以注册到他这里，例如有个8081，再有个8082，都行。

  2019-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a0/57/3a729755.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灯盖

  强大

  2019-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/d9/02/c4e2d7e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Geek_Huahui

  刘老师，能讲解一下现在用的比较多的JSON-RPC吗

  作者回复: restful会讲的

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/10/d5/b527fdcb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  迭代升级

  看了这个大端小端有一些疑惑，百度百科https://m.baidu.com/sf_bk/item/%E5%A4%A7%E5%B0%8F%E7%AB%AF%E6%A8%A1%E5%BC%8F/6750542?fr=aladdin&ms=1&rid=8203658230675846926 上说:大端模式，是指数据的高字节保存在内存的低地址中，而数据的低字节保存在内存的高地址中，这样的存储模式有点儿类似于把数据当作字符串顺序处理：地址由小向大增加，而数据从高位往低位放；这和我们的阅读习惯一致。 小端模式，是指数据的高字节保存在内存的高地址中，而数据的低字节保存在内存的低地址中，这种存储模式将地址的高低和数据位权有效地结合起来，高地址部分权值高，低地址部分权值低。 而文章上说:最低位放在最后一个位置，叫做小端 最低位放在第一个位置，叫做大端 感觉有点矛盾呀，是我的理解出错了吗？

  作者回复: 不矛盾呀，不是一个意思吗

  2019-06-12

  **3

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c9/20/e4f1b17c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zj

  插问一下，http协议中有类似XID这样的标识别唯一请求id吗

  作者回复: header里面可以自己加

  2019-06-03

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  我在使用以太坊的geth客户端的时候, 里面和geth通讯用的也叫rpc协议, 但是他其实是一个json格式的数据传输过程, 感觉和本文讲的rpc协议有很大区别.

  作者回复: json是rpc的一种

  2019-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/85/ed/905b052f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  超超

  答问题一：可以通过类似Linux的inotify机制，监听文件变化，触发回调，把文件的变化通过RPC传输写入服务端。 答问题二：服务是固定的，不够灵活，当出现新增加服务时，无法在线升级服务。

  2019-04-11

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epia1tnD3nqibxHxLowyyTjAcRLKb6Z2UDuxaTLTKKm00MMdB5lpXEOicxkBUJMiapEMq4wFEz7rR29vA/132)

  Geek_549e33

  在咋那么像以前的EJB

  2018-12-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9d/bc/7b2a5339.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  土豆柠檬

  老师，rpc和rmi有什么区别？

  2018-08-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c2/e0/7188aa0a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  blackpiglet

  1 应该是函数指针吧，将 nfs的RPC 函数，作为回调函数，传递给系统调用。 2 缺点一是太复杂了，开发和调用都难度很高；二是无法做到语言无关。

  2018-08-09
