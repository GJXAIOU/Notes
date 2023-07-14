# 10 \| 网络通信优化之通信协议：如何优化RPC网络通信？

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/60/40/60dcf923eeffb956645d2eef6f778340.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/5b/00/5b68aa25cbd602da866668e9272e6b00.mp3" type="audio/mpeg"></audio>

你好，我是刘超。今天我将带你了解下服务间的网络通信优化。

上一讲中，我提到了微服务框架，其中SpringCloud和Dubbo的使用最为广泛，行业内也一直存在着对两者的比较，很多技术人会为这两个框架哪个更好而争辩。

我记得我们部门在搭建微服务框架时，也在技术选型上纠结良久，还曾一度有过激烈的讨论。当前SpringCloud炙手可热，具备完整的微服务生态，得到了很多同事的票选，但我们最终的选择却是Dubbo，这是为什么呢？

## RPC通信是大型服务框架的核心

我们经常讨论微服务，首要应该了解的就是微服务的核心到底是什么，这样我们在做技术选型时，才能更准确地把握需求。

就我个人理解，<span class="orange">我认为微服务的核心是远程通信和服务治理。</span>

远程通信提供了服务之间通信的桥梁，服务治理则提供了服务的后勤保障。所以，我们在做技术选型时，更多要考虑的是这两个核心的需求。

我们知道服务的拆分增加了通信的成本，特别是在一些抢购或者促销的业务场景中，如果服务之间存在方法调用，比如，抢购成功之后需要调用订单系统、支付系统、券包系统等，这种远程通信就很容易成为系统的瓶颈。所以，在满足一定的服务治理需求的前提下，对远程通信的性能需求就是技术选型的主要影响因素。

<!-- [[[read_end]]] -->

目前，很多微服务框架中的服务通信是基于RPC通信实现的，在没有进行组件扩展的前提下，SpringCloud是基于Feign组件实现的RPC通信（基于Http+Json序列化实现），Dubbo是基于SPI扩展了很多RPC通信框架，包括RMI、Dubbo、Hessian等RPC通信框架（默认是Dubbo+Hessian序列化）。不同的业务场景下，RPC通信的选择和优化标准也不同。

例如，开头我提到的我们部门在选择微服务框架时，选择了Dubbo。当时的选择标准就是RPC通信可以支持抢购类的高并发，在这个业务场景中，请求的特点是瞬时高峰、请求量大和传入、传出参数数据包较小。而Dubbo中的Dubbo协议就很好地支持了这个请求。

**以下是基于Dubbo:2.6.4版本进行的简单的性能测试。**分别测试Dubbo+Protobuf序列化以及Http+Json序列化的通信性能（这里主要模拟单一TCP长连接+Protobuf序列化和短连接的Http+Json序列化的性能对比）。为了验证在数据量不同的情况下二者的性能表现，我分别准备了小对象和大对象的性能压测，通过这样的方式我们也可以间接地了解下二者在RPC通信方面的水平。

![](<https://static001.geekbang.org/resource/image/dc/54/dc950f3a5ff15253e101fac90c192f54.jpg?wh=1438*321>)

![](<https://static001.geekbang.org/resource/image/20/b1/20814a2a87057fdc03af699454f703b1.jpg?wh=1429*312>)

这个测试是我之前的积累，基于测试环境比较复杂，这里我就直接给出结果了，如果你感兴趣的话，可以留言和我讨论。

通过以上测试结果可以发现：**无论从响应时间还是吞吐量上来看，单一TCP长连接+Protobuf序列化实现的RPC通信框架都有着非常明显的优势。**

在高并发场景下，我们选择后端服务框架或者中间件部门自行设计服务框架时，RPC通信是重点优化的对象。

其实，目前成熟的RPC通信框架非常多，如果你们公司没有自己的中间件团队，也可以基于开源的RPC通信框架做扩展。在正式进行优化之前，我们不妨简单回顾下RPC。

## 什么是RPC通信

一提到RPC，你是否还想到MVC、SOA这些概念呢？如果你没有经历过这些架构的演变，这些概念就很容易混淆。**你可以通过下面这张图来了解下这些架构的演变史。**

![](<https://static001.geekbang.org/resource/image/e4/a5/e43a8f81d76927948a73a9977643daa5.jpg?wh=2560*576>)

无论是微服务、SOA、还是RPC架构，它们都是分布式服务架构，都需要实现服务之间的互相通信，我们通常把这种通信统称为RPC通信。

RPC（Remote Process Call），即远程服务调用，是通过网络请求远程计算机程序服务的通信技术。RPC框架封装好了底层网络通信、序列化等技术，我们只需要在项目中引入各个服务的接口包，就可以实现在代码中调用RPC服务同调用本地方法一样。正因为这种方便、透明的远程调用，RPC被广泛应用于当下企业级以及互联网项目中，是实现分布式系统的核心。

RMI（Remote Method Invocation）是JDK中最先实现了RPC通信的框架之一，RMI的实现对建立分布式Java应用程序至关重要，是Java体系非常重要的底层技术，很多开源的RPC通信框架也是基于RMI实现原理设计出来的，包括Dubbo框架中也接入了RMI框架。接下来我们就一起了解下RMI的实现原理，看看它存在哪些性能瓶颈有待优化。

## RMI：JDK自带的RPC通信框架

目前RMI已经很成熟地应用在了EJB以及Spring框架中，是纯Java网络分布式应用系统的核心解决方案。RMI实现了一台虚拟机应用对远程方法的调用可以同对本地方法的调用一样，RMI帮我们封装好了其中关于远程通信的内容。

### RMI的实现原理

RMI远程代理对象是RMI中最核心的组件，除了对象本身所在的虚拟机，其它虚拟机也可以调用此对象的方法。而且这些虚拟机可以不在同一个主机上，通过远程代理对象，远程应用可以用网络协议与服务进行通信。

我们可以通过一张图来详细地了解下整个RMI的通信过程：

![](<https://static001.geekbang.org/resource/image/11/4f/1113e44dd62591ce68961e017c11ed4f.jpg?wh=2070*1106>)

### RMI在高并发场景下的性能瓶颈

- Java默认序列化

<!-- -->

RMI的序列化采用的是Java默认的序列化方式，我在09讲中详细地介绍过Java序列化，我们深知它的性能并不是很好，而且其它语言框架也暂时不支持Java序列化。

- TCP短连接

<!-- -->

由于RMI是基于TCP短连接实现，在高并发情况下，大量请求会带来大量连接的创建和销毁，这对于系统来说无疑是非常消耗性能的。

- 阻塞式网络I/O

<!-- -->

在08讲中，我提到了网络通信存在I/O瓶颈，如果在Socket编程中使用传统的I/O模型，在高并发场景下基于短连接实现的网络通信就很容易产生I/O阻塞，性能将会大打折扣。

## 一个高并发场景下的RPC通信优化路径

SpringCloud的RPC通信和RMI通信的性能瓶颈就非常相似。SpringCloud是基于Http通信协议（短连接）和Json序列化实现的，在高并发场景下并没有优势。 那么，在瞬时高并发的场景下，我们又该如何去优化一个RPC通信呢？

RPC通信包括了建立通信、实现报文、传输协议以及传输数据编解码等操作，接下来我们就从每一层的优化出发，逐步实现整体的性能优化。

### 1\.选择合适的通信协议

要实现不同机器间的网络通信，我们先要了解计算机系统网络通信的基本原理。网络通信是两台设备之间实现数据流交换的过程，是基于网络传输协议和传输数据的编解码来实现的。其中网络传输协议有TCP、UDP协议，这两个协议都是基于Socket编程接口之上，为某类应用场景而扩展出的传输协议。通过以下两张图，我们可以大概了解到基于TCP和UDP协议实现的Socket网络通信是怎样的一个流程。

![](<https://static001.geekbang.org/resource/image/2c/0b/2c7c373963196a30e9d4fdc524a92d0b.jpg?wh=1924*1394>)

基于TCP协议实现的Socket通信是有连接的，而传输数据是要通过三次握手来实现数据传输的可靠性，且传输数据是没有边界的，采用的是字节流模式。

基于UDP协议实现的Socket通信，客户端不需要建立连接，只需要创建一个套接字发送数据报给服务端，这样就不能保证数据报一定会达到服务端，所以在传输数据方面，基于UDP协议实现的Socket通信具有不可靠性。UDP发送的数据采用的是数据报模式，每个UDP的数据报都有一个长度，该长度将与数据一起发送到服务端。

通过对比，我们可以得出优化方法：<span class="orange">为了保证数据传输的可靠性，通常情况下我们会采用TCP协议。</span>

如果在局域网且对数据传输的可靠性没有要求的情况下，我们也可以考虑使用UDP协议，毕竟这种协议的效率要比TCP协议高。

### 2\.使用单一长连接

如果是基于TCP协议实现Socket通信，我们还能做哪些优化呢？

服务之间的通信不同于客户端与服务端之间的通信。客户端与服务端由于客户端数量多，基于短连接实现请求可以避免长时间地占用连接，导致系统资源浪费。

但服务之间的通信，连接的消费端不会像客户端那么多，但消费端向服务端请求的数量却一样多，我们基于长连接实现，就可以省去大量的TCP建立和关闭连接的操作，从而减少系统的性能消耗，节省时间。

### 3\.优化Socket通信

建立两台机器的网络通信，我们一般使用Java的Socket编程实现一个TCP连接。传统的Socket通信主要存在I/O阻塞、线程模型缺陷以及内存拷贝等问题。我们可以使用比较成熟的通信框架，比如Netty。Netty4对Socket通信编程做了很多方面的优化，具体见下方。

**实现非阻塞I/O：**在08讲中，我们提到了多路复用器Selector实现了非阻塞I/O通信。

**高效的Reactor线程模型：**Netty使用了主从Reactor多线程模型，服务端接收客户端请求连接是用了一个主线程，这个主线程用于客户端的连接请求操作，一旦连接建立成功，将会监听I/O事件，监听到事件后会创建一个链路请求。

链路请求将会注册到负责I/O操作的I/O工作线程上，由I/O工作线程负责后续的I/O操作。利用这种线程模型，可以解决在高负载、高并发的情况下，由于单个NIO线程无法监听海量客户端和满足大量I/O操作造成的问题。

**串行设计：**服务端在接收消息之后，存在着编码、解码、读取和发送等链路操作。如果这些操作都是基于并行去实现，无疑会导致严重的锁竞争，进而导致系统的性能下降。为了提升性能，Netty采用了串行无锁化完成链路操作，Netty提供了Pipeline实现链路的各个操作在运行期间不进行线程切换。

**零拷贝：**在08讲中，我们提到了一个数据从内存发送到网络中，存在着两次拷贝动作，先是从用户空间拷贝到内核空间，再是从内核空间拷贝到网络I/O中。而NIO提供的ByteBuffer可以使用Direct Buffers模式，直接开辟一个非堆物理内存，不需要进行字节缓冲区的二次拷贝，可以直接将数据写入到内核空间。

**除了以上这些优化，我们还可以针对套接字编程提供的一些TCP参数配置项，提高网络吞吐量，Netty可以基于ChannelOption来设置这些参数。**

**TCP\_NODELAY：**TCP\_NODELAY选项是用来控制是否开启Nagle算法。Nagle算法通过缓存的方式将小的数据包组成一个大的数据包，从而避免大量的小数据包发送阻塞网络，提高网络传输的效率。我们可以关闭该算法，优化对于时延敏感的应用场景。

**SO\_RCVBUF和SO\_SNDBUF：**可以根据场景调整套接字发送缓冲区和接收缓冲区的大小。

**SO\_BACKLOG：**backlog参数指定了客户端连接请求缓冲队列的大小。服务端处理客户端连接请求是按顺序处理的，所以同一时间只能处理一个客户端连接，当有多个客户端进来的时候，服务端就会将不能处理的客户端连接请求放在队列中等待处理。

**SO\_KEEPALIVE：**当设置该选项以后，连接会检查长时间没有发送数据的客户端的连接状态，检测到客户端断开连接后，服务端将回收该连接。我们可以将该时间设置得短一些，来提高回收连接的效率。

### 4\.量身定做报文格式

接下来就是实现报文，我们需要设计一套报文，用于描述具体的校验、操作、传输数据等内容。为了提高传输的效率，我们可以根据自己的业务和架构来考虑设计，尽量实现报体小、满足功能、易解析等特性。我们可以参考下面的数据格式：

![](<https://static001.geekbang.org/resource/image/6d/c1/6dc21193a6ffbf94a7dd8e5a0d2302c1.jpg?wh=1512*204>)<br>

![](<https://static001.geekbang.org/resource/image/f3/ae/f3bb46ed6ece4a8a9bcc3d9e9df84cae.jpg?wh=1191*718>)

### 5\.编码、解码

在09讲中，我们分析过序列化编码和解码的过程，对于实现一个好的网络通信协议来说，兼容优秀的序列化框架是非常重要的。如果只是单纯的数据对象传输，我们可以选择性能相对较好的Protobuf序列化，有利于提高网络通信的性能。

### 6\.调整Linux的TCP参数设置选项

如果RPC是基于TCP短连接实现的，我们可以通过修改Linux TCP配置项来优化网络通信。开始TCP配置项的优化之前，我们先来了解下建立TCP连接的三次握手和关闭TCP连接的四次握手，这样有助后面内容的理解。

- 三次握手

<!-- -->

![](<https://static001.geekbang.org/resource/image/32/de/32381d3314bd982544f69e4d3faba1de.jpg?wh=1218*800>)

- 四次握手

<!-- -->

![](<https://static001.geekbang.org/resource/image/df/91/df9f4e3f3598a7e160c899f552a59391.jpg?wh=964*666>)

我们可以通过sysctl -a \| grep net.xxx命令运行查看Linux系统默认的的TCP参数设置，如果需要修改某项配置，可以通过编辑 vim/etc/sysctl.conf，加入需要修改的配置项， 并通过sysctl -p命令运行生效修改后的配置项设置。通常我们会通过修改以下几个配置项来提高网络吞吐量和降低延时。

![](<https://static001.geekbang.org/resource/image/9e/bc/9eb01fe017b267367b11170a864bd0bc.jpg?wh=1503*1340>)

以上就是我们从不同层次对RPC优化的详解，除了最后的Linux系统中TCP的配置项设置调优，其它的调优更多是从代码编程优化的角度出发，最终实现了一套RPC通信框架的优化路径。

弄懂了这些，你就可以根据自己的业务场景去做技术选型了，还能很好地解决过程中出现的一些性能问题。

## 总结

在现在的分布式系统中，特别是系统走向微服务化的今天，服务间的通信就显得尤为频繁，掌握服务间的通信原理和通信协议优化，是你的一项的必备技能。

在一些并发场景比较多的系统中，我更偏向使用Dubbo实现的这一套RPC通信协议。Dubbo协议是建立的单一长连接通信，网络I/O为NIO非阻塞读写操作，更兼容了Kryo、FST、Protobuf等性能出众的序列化框架，在高并发、小对象传输的业务场景中非常实用。

在企业级系统中，业务往往要比普通的互联网产品复杂，服务与服务之间可能不仅仅是数据传输，还有图片以及文件的传输，所以RPC的通信协议设计考虑更多是功能性需求，在性能方面不追求极致。其它通信框架在功能性、生态以及易用、易入门等方面更具有优势。

## 思考题

目前实现Java RPC通信的框架有很多，实现RPC通信的协议也有很多，除了Dubbo协议以外，你还使用过其它RPC通信协议吗？<span class="orange">通过这讲的学习，你能对比谈谈各自的优缺点了吗？</span>

期待在留言区看到你的见解。也欢迎你点击“请朋友读”，把今天的内容分享给身边的朋友，邀请他一起学习。

## 精选留言(28)

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34.jpeg)

  夏天39度

  老师，能说一下Netty是如何实现串行无锁化完成链路操作吗，怎么做到无锁化的线程切换

  作者回复: Netty中，分为Reactor主线程和Reactor从线程，主线程主要用来监听连接事件，从线程主要用来处理监听I/O事件，以及处理读写I/O操作。 一般有其他的业务操作，我们可以在handler中创建线程池来处理。但为了减少上下文切换，我们可以在ChannelPipeline上注册handler事件，例如解码过程。一般Reactor从线程监听到读操作，会立即调用ChannelPipeline的fireChannelRead方法完成读操作，在调用完读操作之后，会检查是否有其他handler，如果有，则直接调用，不会创建新的线程。 这种方式的好处是，不用创建新的线程，在从线程中串行化完成所有的I/O以及业务操作，减少上下文切换；坏处是，给从线程带来了一定的阻塞。

  2019-06-11

  **2

  **24

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1811.jpeg)

  WL

  请教老师两个问题: 1. 在RMI的实现原理示意图中客户端的存根和服务端的骨架这两个概念是啥意思, 我感觉不太理解. 2. 在TCP的四次挥手中, 客户端最后的TIME_WAIT状态是不是就是CLOSE的状态, 如果不是那TIME_WAIT状态是在啥时候转换成CLOSE状态的.

  作者回复: 我先回答第二个问题。 是的，TIME_WAIT状态就是主动断开方的最后状态了。主动断开连接方之所以是TIME_WAIT状态，是担心被断开方没有收到最后的ACK，这个TIME_WAIT时间内核默认设置是2MSL(报文最大生存时间)，被断开方如果超时没有收到ACK，将重新发送FIN，主动断开方收到之后又会重新发送ACK通知，重置TIME_WAIT时间。 正常情况下，当主动断开方的TIME_WAIT状态到达了定时时间后，内核就会关闭该连接。 第一个问题，这块文章中没有过多的介绍，我在这里再叙述下:  Stub是client端的远程对象的代理，负责将远程对象上的方法调用转发到实际远程对象实现所在的服务器，我们的程序要通过远程调用，底层一定是套接字的字节传输，要一个对象序列化成为字节码，传输到服务器或者客户端的对端之后，再把该对象反序列化成为对应的对象，Stub承担着底层序列化、数据组装以及协议封装等工作。 Skeleton则是server端的服务对象的代理，负责将接收解析远程调用分派到实际远程对象实现调用。Stub与Skeleton的关系以及操作是对应的关系，只有实现了java.rmi.Remote接口的类或者继承了java.rmi.Remote接口的接口，才能作为Stub与Skeleton之间通信的远程对象，Stub与Skeleton之间的通信使用Java远程消息交换协议JRMP（Java Remote Messaging Protocol）进行通信，JRMP是专为Java的远程对象制定的协议。Stub和Skeleton之间协作完成客户端与服务器之间的方法调用时的通信。

  2019-06-11

  **

  **18

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1812.jpeg)

  Stalary

  老师，如果业务架构已经选择了SpringCloud，该如何优化远程调用呢，目前使用Feign，底层配置了HttpClient，发现qps一直上不去，暂时是对频繁的请求做了本地cache，但是需要订阅更新事件进行刷新

  作者回复: 可以尝试扩展其他RPC框架，例如有同学提到的Google的grpc框架，也是基于Netty通信框架实现，基于protobuf实现的序列化。

  2019-06-11

  **3

  **13

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  n88

  写http是短连接不太严谨

  作者回复: 这里纠正下，http1.0版本默认是短链接，而在http1.0以后默认是保持连接的，但只是一个单向的长连接，默认情况下保持60s

  2019-10-25

  **2

  **9

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1813.jpeg)

  尔冬橙

  这个Dubbo阿里可以吹很多年啊

  2019-09-14

  **1

  **5

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1814.jpeg)

  行则将至

  老师。我看到留言中有同学提到http和tcp的对比。http不是建立在tcp的基础上吗？http和tcp的关系应该怎么定义呢？

  作者回复: http是基于tcp实现的协议。如果做过Socket编程通信，你会发现两个端之间如果要实现接口通信，除了传输我们需要的请求参数和返回参数之外，我们还需要给通信定义一个协议头，单纯的tcp通信是没有协议头的，而http则是在tcp基础上定义了自己的消息头和序列化方式。http通信协议是一种短连接，也就是说通信完成之后会断开连接。 简而言之，http是tcp的一个上层封装协议，http是基于tcp实现的。

  2019-08-13

  **2

  **5

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1815.jpeg)

  晓晨同学

  有个问题请教一下老师 1.一直不清楚通信协议和序列化协议的区别是什么，两者都是制定报文协议然后传输，感觉序列化协议更具体到业务属性

  作者回复: 通信协议是指我们传输信息的协议，包括头协议和包体，头协议中可能包含传输的id、包体大小、序列化方式等等信息，序列化则表示我们传输的包体的载体是什么样的格式，例如是将对象转成json格式还是转成xml格式，再转成二进制进行传输。

  2019-08-07

  **

  **3

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1816.jpeg)

  nightmare

  能不能讲一下netty的串行无锁化

  作者回复: Netty中，分为Reactor主线程和Reactor从线程，主线程主要用来监听连接事件，从线程主要用来处理监听I/O事件，以及处理读写I/O操作。 一般有其他的业务操作，我们可以在handler中创建线程池来处理。但为了减少上下文切换，我们可以在ChannelPipeline上注册handler事件，例如解码过程。一般Reactor从线程监听到读操作，会立即调用ChannelPipeline的fireChannelRead方法完成读操作，在调用完读操作之后，会检查是否有其他handler，如果有，则直接调用，不会创建新的线程。 这种方式的好处是，不用创建新的线程，在从线程中串行化完成所有的I/O以及业务操作，减少上下文切换；坏处是，给从线程带来了一定的阻塞。

  2019-06-13

  **

  **3

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1817.jpeg)

  Y丶z

  老师好，我想问下已经在线上跑的服务，序列化方式是hessian，如果直接换成Protobuf，那么consumer会报错吗？如果报错的话，如何避免这种情况发生呢？

  作者回复: 服务端和消费端重启，会走protobuf序列化

  2019-06-11

  **

  **4

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1818.jpeg)

  张德

  还知道JAVA和Python系统之间互相调用的thrift

  作者回复: thrift框架也很优秀

  2019-07-07

  **

  **2

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1819.jpeg)

  晓杰

  请问老师，对于大文件的传输，用哪种协议比较好

  作者回复: 建议使用hessian协议

  2019-06-11

  **

  **3

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1820.jpeg)

  假装自己不胖

  对于网络编程比较迷茫，请问有没有小白一些的读物或博客推荐一下

  作者回复: 这块知识点比较多，建议可以看一些基础书籍，例如Unix网络编程、TCP/IP网络编程，再看看netty实战，就可以进阶Java网络编程了。

  2019-06-11

  **

  **2

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1821.jpeg)

  Sdylan

  Spring Clound 的Feigin更多是路由功能，将注解拼接成地址，Spring Cloud主要是整体完备，负载均衡、熔断啥的。

  2019-10-15

  **

  **1

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1822.jpeg)

  钱

  课后思考及问题 1：网上常见有关TCP和HTTP的问题，比如： TCP连接的建立详细过程？TCP的连接断开过程？ 三次握手建立连接，四次握手断开连接，感觉有些简单啦！如果面试时问到这个问题，老师建议该怎么回答？ 另外，还有问一次HTTP请求经过了几次TCP连接，这个如果面试时遇到了，老师又建议该怎么好好的回答？

  作者回复: 熟悉连接时为什么是三次握手，断开时为什么需要四次握手，以及粘包拆包的问题、解决方案。我理解的一次HTTP请求只有一次TCP连接。

  2019-09-07

  **2

  **1

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1823.jpeg)

  电光火石

  \1. 老师线上有用过grpc吗，看文档说好像现在还不是特别的稳定？ 2. 文中的性能测试，http是否有打开keep alive？走tcp无疑更快，我只想知道用http会慢多少，因为毕竟http更简单。有看过其他的benchmark，在打开keep alive的情况下，性能也还行，不知道老师这个测试是否打开？另外，从测试结果上看，当单次请求数据量很大的时候，http比tcp好像查不了多少是吗？ 谢谢！

  作者回复: grpc目前很多公司在用，在Github中有源码阅读https://github.com/grpc。 对的，没有打开keep alive，如果开启效果会好一些，减少了网络连接。

  2019-06-11

  **

  **1

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/132.png)

  ZeWe

  RPC 远程服务调用，是一种通过网络通信调用远程服务的技术。RPC框架底层封装了网络通信，序列化等技术，提供接口包。本地调用接口可以无感透明的访问远程服务，是分布式系统的核心。 如何优化RPC框架，可从网络通信，报文序列化两方面考虑。 网络通信 1）选择合适的通信协议 TCP, UDP等 2）TCP协议下 使用长链接，减少连接资源消耗 3）高效的IO模型 如Netty NIO, 非阻塞io，Reactor线程模型，零拷贝等 4) OS层 TCP参数设置  file-max, keepalive-time等 序列化 1） 简单高效的报文设计 2) 高效序列化框架 编码解码

  2021-09-07

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1824.jpeg)

  惘 闻

  老师netty的reactor监听链接事件的主线程,可以认为是IO多路复用里的多路复用器selector吗?

  2020-12-28

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1825.jpeg)

  放下

  老师你上面Feign的压测结果都是基于HTTP1.0的短链接吗？如果是HTTP1.1压测效果会不会更好一些

  作者回复: 会的，现在都默认使用http1.1版本了，默认是在超时时间内为长连接。

  2020-05-23

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/132-1662425669284-1826.jpeg)

  greekw

  老师，能讲下netty的rector 线程模型的实例，以及串行无锁化设计的原理

  作者回复: 加餐篇中讲到了

  2019-10-20

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1827.jpeg)

  lizhibo

  老师 我想问的 Dubbo怎么更换成Protobuf 系列化协议？是要扩展Dubbo Serialization 接口吗？另外 Kryo 和 Protobuf 哪个性能高点啊

  作者回复: 是的，目前2.7版本已经加入了Protobuf序列化。Kryo和Protobuf 性能非常接近，Kryo序列化后的空间要比Protobuf小一些，但Protobuf序列化与反序列化的时间要优于Kryo。

  2019-09-28

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1813.jpeg)

  尔冬橙

  老师，您说http是短链接，http1·1的keep alive 不是长链接么。或者用http2·0也算是优化吧？

  作者回复: 可手动设置keep alive，保持连接

  2019-09-14

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669283-1819.jpeg)

  晓杰

  今天面试，我说springcloud的feign是rpc框架，然后他说不是，是伪rpc框架，请问老师springcloud的feign是rpc框架吗

  作者回复: feign是通过http实现接口请求，与传统的RPC框架有一定的区别，但也是一种RPC实现。

  2019-08-07

  **2

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1828.jpeg)

  Joker

  老师厉害，层层递进啊。。。虽然可能我们目前没有用到，但是却是个非常好的地图啊。

  2019-07-09

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1829.jpeg)

  -W.LI-

  老师好!文中提到了高效的 Reactor 线程模型。有适合新手的资料链接么?还有个pr啥的能一起给我个么谢谢了，文中讲的看不懂。

  作者回复: 下一讲中会讲到Reactor、Proactor线程模型。

  2019-06-11

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1830.jpeg)

  拒绝

  RPC要怎么学，项目中也用到了dubbo，可跟老师说的完全不一样，不是添加其他服务接口的依赖调用方法，而是发送http请求调用其他服务的接口。使我们用错了吗？ 有个疑问依赖其他的服务的接口调用方法，接口都没有实现怎么调用？

  作者回复: 请问你们用的什么协议？dubbo是消费服务类型，服务会注册接口到注册中心，消费端通过拉取注册中心的注册服务接口，与服务端通信。所以一般都是通过接口服务实现消息通信。

  2019-06-11

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1831.jpeg)

  Liam

  能否讲讲不同情况下，tcp各个调优参数的值应该怎么设和通常设置为多少

  2019-06-11

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1832.jpeg)

  黑崽

  断开连接是四次挥手吧

  作者回复: 是的，文中提到了。

  2019-06-11

  **

  **

- ![img](10%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E9%80%9A%E4%BF%A1%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96RPC%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425669284-1833.jpeg)

  进阶的码农

  我们是用spring-cloud 继承google的grpc

  作者回复: 优秀

