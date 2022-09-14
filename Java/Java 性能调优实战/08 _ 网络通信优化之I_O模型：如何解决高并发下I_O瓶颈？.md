# 08 \| 网络通信优化之I/O模型：如何解决高并发下I/O瓶颈？

提到Java I/O，相信你一定不陌生。你可能使用I/O操作读写文件，也可能使用它实现Socket的信息传输…这些都是我们在系统中最常遇到的和I/O有关的操作。

我们都知道，I/O的速度要比内存速度慢，尤其是在现在这个大数据时代背景下，I/O的性能问题更是尤为突出，I/O读写已经成为很多应用场景下的系统性能瓶颈，不容我们忽视。

今天，我们就来深入了解下Java I/O在高并发、大数据业务场景下暴露出的性能问题，从源头入手，学习优化方法。

## 什么是I/O

I/O是机器获取和交换信息的主要渠道，而流是完成I/O操作的主要方式。

在计算机中，流是一种信息的转换。流是有序的，因此相对于某一机器或者应用程序而言，我们通常把机器或者应用程序接收外界的信息称为输入流（InputStream），从机器或者应用程序向外输出的信息称为输出流（OutputStream），合称为输入/输出流（I/O Streams）。

机器间或程序间在进行信息交换或者数据交换时，总是先将对象或数据转换为某种形式的流，再通过流的传输，到达指定机器或程序后，再将流转换为对象数据。因此，流就可以被看作是一种数据的载体，通过它可以实现数据交换和传输。

Java的I/O操作类在包java.io下，其中InputStream、OutputStream以及Reader、Writer类是I/O包中的4个基本类，它们分别处理字节流和字符流。如下图所示：

![](<https://static001.geekbang.org/resource/image/64/7d/64f1e83054e2997f1fd96b221fb0da7d.jpg?wh=1006*420>)

回顾我的经历，我记得在初次阅读Java I/O流文档的时候，我有过这样一个疑问，在这里也分享给你，那就是：“**不管是文件读写还是网络发送接收，信息的最小存储单元都是字节，那为什么I/O流操作要分为字节流操作和字符流操作呢？**”

我们知道字符到字节必须经过转码，这个过程非常耗时，如果我们不知道编码类型就很容易出现乱码问题。所以I/O流提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。下面我们就分别了解下“字节流”和“字符流”。

### 1\.字节流

InputStream/OutputStream是字节流的抽象类，这两个抽象类又派生出了若干子类，不同的子类分别处理不同的操作类型。如果是文件的读写操作，就使用FileInputStream/FileOutputStream；如果是数组的读写操作，就使用ByteArrayInputStream/ByteArrayOutputStream；如果是普通字符串的读写操作，就使用BufferedInputStream/BufferedOutputStream。具体内容如下图所示：

![](<https://static001.geekbang.org/resource/image/12/8f/12bbf6e62c7c29ae82bf90fead72b98f.jpg?wh=1474*1086>)

### 2\.字符流

Reader/Writer是字符流的抽象类，这两个抽象类也派生出了若干子类，不同的子类分别处理不同的操作类型，具体内容如下图所示：

![](<https://static001.geekbang.org/resource/image/24/9f/24592c6f90300f7bab86ec4141dd7e9f.jpg?wh=1126*742>)

## 传统I/O的性能问题

我们知道，I/O操作分为磁盘I/O操作和网络I/O操作。前者是从磁盘中读取数据源输入到内存中，之后将读取的信息持久化输出在物理磁盘上；后者是从网络中读取信息输入到内存，最终将信息输出到网络中。但不管是磁盘I/O还是网络I/O，在传统I/O中都存在严重的性能问题。

### 1\.多次内存复制

在传统I/O中，我们可以通过InputStream从源数据中读取数据流输入到缓冲区里，通过OutputStream将数据输出到外部设备（包括磁盘、网络）。你可以先看下输入操作在操作系统中的具体流程，如下图所示：

![](<https://static001.geekbang.org/resource/image/4c/c2/4c4af15b08d3b11de3fe603a70dc6ac2.jpg?wh=1458*686>)

- JVM会发出read()系统调用，并通过read系统调用向内核发起读请求；
- 内核向硬件发送读指令，并等待读就绪；
- 内核把将要读取的数据复制到指向的内核缓存中；
- 操作系统内核将数据复制到用户空间缓冲区，然后read系统调用返回。

在这个过程中，数据先从外部设备复制到内核空间，再从内核空间复制到用户空间，这就发生了两次内存复制操作。这种操作会导致不必要的数据拷贝和上下文切换，从而降低I/O的性能。

### 2\.阻塞

在传统I/O中，InputStream的read()是一个while循环操作，它会一直等待数据读取，直到数据就绪才会返回。**这就意味着如果没有数据就绪，这个读取操作将会一直被挂起，用户线程将会处于阻塞状态。**

在少量连接请求的情况下，使用这种方式没有问题，响应速度也很高。但在发生大量连接请求时，就需要创建大量监听线程，这时如果线程没有数据就绪就会被挂起，然后进入阻塞状态。一旦发生线程阻塞，这些线程将会不断地抢夺CPU资源，从而导致大量的CPU上下文切换，增加系统的性能开销。

## 如何优化I/O操作

面对以上两个性能问题，不仅编程语言对此做了优化，各个操作系统也进一步优化了I/O。JDK1.4发布了java.nio包（new I/O的缩写），NIO的发布优化了内存复制以及阻塞导致的严重性能问题。JDK1.7又发布了NIO2，提出了从操作系统层面实现的异步I/O。下面我们就来了解下具体的优化实现。

### 1\.使用缓冲区优化读写流操作

在传统I/O中，提供了基于流的I/O实现，即InputStream和OutputStream，这种基于流的实现以字节为单位处理数据。

NIO与传统 I/O 不同，它是基于块（Block）的，它以块为基本单位处理数据。在NIO中，最为重要的两个组件是缓冲区（Buffer）和通道（Channel）。Buffer是一块连续的内存块，是 NIO 读写数据的中转地。Channel表示缓冲数据的源头或者目的地，它用于读取缓冲或者写入数据，是访问缓冲的接口。

传统I/O和NIO的最大区别就是传统I/O是面向流，NIO是面向Buffer。Buffer可以将文件一次性读入内存再做后续处理，而传统的方式是边读文件边处理数据。虽然传统I/O后面也使用了缓冲块，例如BufferedInputStream，但仍然不能和NIO相媲美。使用NIO替代传统I/O操作，可以提升系统的整体性能，效果立竿见影。

### 2\. 使用DirectBuffer减少内存复制

NIO的Buffer除了做了缓冲块优化之外，还提供了一个可以直接访问物理内存的类DirectBuffer。普通的Buffer分配的是JVM堆内存，而DirectBuffer是直接分配物理内存(非堆内存)。

我们知道数据要输出到外部设备，必须先从用户空间复制到内核空间，再复制到输出设备，而在Java中，在用户空间中又存在一个拷贝，那就是从Java堆内存中拷贝到临时的直接内存中，通过临时的直接内存拷贝到内存空间中去。此时的直接内存和堆内存都是属于用户空间。

![](<https://static001.geekbang.org/resource/image/39/c2/399d715ed2f687e22ec9ca2a65bd88c2.jpg?wh=1652*824>)

你肯定会在想，为什么Java需要通过一个临时的非堆内存来复制数据呢？如果单纯使用Java堆内存进行数据拷贝，当拷贝的数据量比较大的情况下，Java堆的GC压力会比较大，而使用非堆内存可以减低GC的压力。

DirectBuffer则是直接将步骤简化为数据直接保存到非堆内存，从而减少了一次数据拷贝。以下是JDK源码中IOUtil.java类中的write方法：

```
if (src instanceof DirectBuffer)
            return writeFromNativeBuffer(fd, src, position, nd);

        // Substitute a native buffer
        int pos = src.position();
        int lim = src.limit();
        assert (pos <= lim);
        int rem = (pos <= lim ? lim - pos : 0); 
        ByteBuffer bb = Util.getTemporaryDirectBuffer(rem);
        try {
            bb.put(src);
            bb.flip();
        // ...............
```

这里拓展一点，由于DirectBuffer申请的是非JVM的物理内存，所以创建和销毁的代价很高。DirectBuffer申请的内存并不是直接由JVM负责垃圾回收，但在DirectBuffer包装类被回收时，会通过Java Reference机制来释放该内存块。

DirectBuffer只优化了用户空间内部的拷贝，而之前我们是说优化用户空间和内核空间的拷贝，那Java的NIO中是否能做到减少用户空间和内核空间的拷贝优化呢？

答案是可以的，DirectBuffer是通过unsafe.allocateMemory(size)方法分配内存，也就是基于本地类Unsafe类调用native方法进行内存分配的。而在NIO中，还存在另外一个Buffer类：MappedByteBuffer，跟DirectBuffer不同的是，MappedByteBuffer是通过本地类调用mmap进行文件内存映射的，map()系统调用方法会直接将文件从硬盘拷贝到用户空间，只进行一次数据拷贝，从而减少了传统的read()方法从硬盘拷贝到内核空间这一步。

### 3\.避免阻塞，优化I/O操作

NIO很多人也称之为Non-block I/O，即非阻塞I/O，因为这样叫，更能体现它的特点。为什么这么说呢？

传统的I/O即使使用了缓冲块，依然存在阻塞问题。由于线程池线程数量有限，一旦发生大量并发请求，超过最大数量的线程就只能等待，直到线程池中有空闲的线程可以被复用。而对Socket的输入流进行读取时，读取流会一直阻塞，直到发生以下三种情况的任意一种才会解除阻塞：

- 有数据可读；
- 连接释放；
- 空指针或I/O异常。

阻塞问题，就是传统I/O最大的弊端。NIO发布后，通道和多路复用器这两个基本组件实现了NIO的非阻塞，下面我们就一起来了解下这两个组件的优化原理。

**通道（Channel）**

前面我们讨论过，传统I/O的数据读取和写入是从用户空间到内核空间来回复制，而内核空间的数据是通过操作系统层面的I/O接口从磁盘读取或写入。

最开始，在应用程序调用操作系统I/O接口时，是由CPU完成分配，这种方式最大的问题是“发生大量I/O请求时，非常消耗CPU“；之后，操作系统引入了DMA（直接存储器存储），内核空间与磁盘之间的存取完全由DMA负责，但这种方式依然需要向CPU申请权限，且需要借助DMA总线来完成数据的复制操作，如果DMA总线过多，就会造成总线冲突。

通道的出现解决了以上问题，Channel有自己的处理器，可以完成内核空间和磁盘之间的I/O操作。在NIO中，我们读取和写入数据都要通过Channel，由于Channel是双向的，所以读、写可以同时进行。

**多路复用器（Selector）**

Selector是Java NIO编程的基础。用于检查一个或多个NIO Channel的状态是否处于可读、可写。

Selector是基于事件驱动实现的，我们可以在Selector中注册accpet、read监听事件，Selector会不断轮询注册在其上的Channel，如果某个Channel上面发生监听事件，这个Channel就处于就绪状态，然后进行I/O操作。

一个线程使用一个Selector，通过轮询的方式，可以监听多个Channel上的事件。我们可以在注册Channel时设置该通道为非阻塞，当Channel上没有I/O操作时，该线程就不会一直等待了，而是会不断轮询所有Channel，从而避免发生阻塞。

目前操作系统的I/O多路复用机制都使用了epoll，相比传统的select机制，epoll没有最大连接句柄1024的限制。所以Selector在理论上可以轮询成千上万的客户端。

**下面我用一个生活化的场景来举例，**看完你就更清楚Channel和Selector在非阻塞I/O中承担什么角色，发挥什么作用了。

我们可以把监听多个I/O连接请求比作一个火车站的进站口。以前检票只能让搭乘就近一趟发车的旅客提前进站，而且只有一个检票员，这时如果有其他车次的旅客要进站，就只能在站口排队。这就相当于最早没有实现线程池的I/O操作。

后来火车站升级了，多了几个检票入口，允许不同车次的旅客从各自对应的检票入口进站。这就相当于用多线程创建了多个监听线程，同时监听各个客户端的I/O请求。

最后火车站进行了升级改造，可以容纳更多旅客了，每个车次载客更多了，而且车次也安排合理，乘客不再扎堆排队，可以从一个大的统一的检票口进站了，这一个检票口可以同时检票多个车次。这个大的检票口就相当于Selector，车次就相当于Channel，旅客就相当于I/O流。

## 总结

Java的传统I/O开始是基于InputStream和OutputStream两个操作流实现的，这种流操作是以字节为单位，如果在高并发、大数据场景中，很容易导致阻塞，因此这种操作的性能是非常差的。还有，输出数据从用户空间复制到内核空间，再复制到输出设备，这样的操作会增加系统的性能开销。

传统I/O后来使用了Buffer优化了“阻塞”这个性能问题，以缓冲块作为最小单位，但相比整体性能来说依然不尽人意。

于是NIO发布，它是基于缓冲块为单位的流操作，在Buffer的基础上，新增了两个组件“管道和多路复用器”，实现了非阻塞I/O，NIO适用于发生大量I/O连接请求的场景，这三个组件共同提升了I/O的整体性能。

你可以在[Github](<https://github.com/nickliuchao/io>)上通过几个简单的例子来实践下传统IO、NIO。

## 思考题

在JDK1.7版本中，Java发布了NIO的升级包NIO2，也就是AIO。AIO实现了真正意义上的异步I/O，它是直接将I/O操作交给操作系统进行异步处理。这也是对I/O操作的一种优化，那<span class="orange">为什么现在很多容器的通信框架都还是使用NIO呢？</span>

期待在留言区看到你的见解。也欢迎你点击“请朋友读”，把今天的内容分享给身边的朋友，邀请他一起学习。



## 精选留言(55)

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34.jpeg)

  ZOU志伟

  老师，为什么要字符流还是没懂

  作者回复: 我们通常在通信时，使用的是字节流FileInputStream来实现数据的传输，你会发现，我们在读取read()和写入write()的时候都是先将字符转成字节码再进行写入操作，同样读取也是类似。如果是中文，在gbk中一般一个中文占用两个字节，如果通过字节流的方式只读取一个字节，是无法转编码为一个中文汉字。 而字符流就是为了解决这种问题，如果用字符流去读取，字符流会根据默认编码一次性的读取一个字符，即若若是gbk编码就会一次读取2个字节。因此字符流是根据字符所占字节大小而决定读取多少字节的。这就是字符流和字节流的本质不同。

  2019-06-19

  **4

  **68

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132.jpeg)

  张学磊

  在Linux中，AIO并未真正使用操作系统所提供的异步I/O，它仍然使用poll或epoll，并将API封装为异步I/O的样子，但是其本质仍然是同步非阻塞I/O，加上第三方产品的出现，Java网络编程明显落后，所以没有成为主流

  作者回复: 对的，异步I/O模型在Linux内核中没有实现

  2019-06-06

  **

  **48

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1584.jpeg)

  Only now

  老师能不能讲讲DMA和Channel的区别, DMA需要占用总线, 那么Channel是如何跳过总线向内存传输数据的?

  作者回复: 一个设备接口试图通过总线直接向外部设备(磁盘)传送数据时，它会先向CPU发送DMA请求信号。外部设备(磁盘)通过DMA的一种专门接口电路――DMA控制器（DMAC），向CPU提出接管总线控制权的总线请求，CPU收到该信号后，在当前的总线周期结束后，会按DMA信号的优先级和提出DMA请求的先后顺序响应DMA信号。CPU对某个设备接口响应DMA请求时，会让出总线控制权。于是在DMA控制器的管理下，磁盘和存储器直接进行数据交换，而不需CPU干预。数据传送完毕后，设备接口会向CPU发送DMA结束信号，交还总线控制权。 而通道则是在DMA的基础上增加了能执行有限通道指令的I/O控制器，代替CPU管理控制外设。通道有自己的指令系统，是一个协处理器，他实质是一台能够执行有限的输入输出指令，并且有专门通讯传输的通道总线完成控制。

  2019-06-06

  **3

  **46

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1585.jpeg)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  皮皮

  老师，个人觉得本期的内容讲的稍微浅了一点，关于IO的几种常见模型可以配图讲一下的，另外就是linux下的select，poll，epoll的对比应用场景。最重要的目前用的最多的IO多路复用可以深入讲一下的。

  作者回复: 你好，这篇I/O性能优化主要是普及NIO对I/O的性能优化。I/O这块的知识点很多，包括IO模型、事件处理模型以及操作系统层的事件驱动，如果都压缩到一讲，由于字数有限，很难讲完整。对于一些童鞋来说，也不好理解。  我将会在后面的一讲中，补充大家提到的一些内容。谢谢你的建议。

  2019-06-06

  **

  **27

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1586.jpeg)

  小兵

  很多知识linux 网络 I/O模型底层原理，零拷贝技术等深入讲一下，毕竟学Java性能调优的学员都是有几年工作经验的， 希望老师后面能专门针对这次io 出个补充，这一讲比较不够深入。

  作者回复: 这一讲中提到了DirectBuffer，也就是零拷贝的实现。谢谢你的建议，后面我会补充下几种网络I/O模型的底层原理。

  2019-06-07

  **

  **13

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1587.jpeg)

  -W.LI-

  老师好!隔壁的李好双老师说一次普通IO需要要进过六次拷贝。 网卡->内核->临时本地内存->堆内存->临时本地内存->内核->网卡。 directbfuffer下 网卡->内核->本地内存->内核->网卡 ARP下C直接调用 文件->内核->网卡。 李老师说的对么? 本地内存和堆内存都是在用户空间的是么?

  作者回复: 李老师说的对的

  2019-06-20

  **6

  **11

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630561-1588.jpeg)

  td901105

  在少量连接请求的情况下，使用这种方式没有问题，响应速度也很高。但在发生大量连接请求时，就需要创建大量监听线程，这时如果线程没有数据就绪就会被挂起，然后进入阻塞状态。一旦发生线程阻塞，这些线程将会不断地抢夺 CPU 资源，从而导致大量的 CPU 上下文切换，增加系统的性能开销。 后面一句一旦发生线程阻塞，这些线程会不断的抢夺CPU资源，从而导致大量的CPU进行上下文切换，增加系统开销，这一句不是太明白，能解释一下吗？阻塞线程不是不会占用CPU资源吗？

  作者回复: 阻塞线程在阻塞状态是不会占用CPU资源的，但是会被唤醒争夺CPU资源。操作系统将CPU轮流分配给线程任务，当线程数量越多的时候，当某个线程在规定的时间片运行完之后，会被其他线程抢夺CPU资源，此时会导致上下文切换。抢夺越激烈，上下文切换就越频繁。

  2019-12-04

  **3

  **10

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1589.jpeg)

  钱

  课后思考及问题 1本文核心观点 1-1：JAVA IO操作为啥分为字节流和字符流？我们知道字符到字节必须经过转码，这个过程非常耗时，如果我们不知道编码类型就很容易出现乱码问题。所以 I/O 流提供了一个直接操作字符的接口，方便我们平时对字符进行流操作。 有个疑问，字符流虽然在不知道其编码类型的情况下可以操作，不过一旦需要进行网络传输了，还是必须要知道的？ 1-2：一次读取数据的操作流程，如下 用户空间—请求数据—内核空间—请求数据—数据存储设备—响应数据—内核空间—响应数据—用户空间，应用程序从用户空间的数据缓存区中读取数据，数据以字节流的形式流转。这种方式数据经过了两次复制过程，比较耗性能。 1-3：传统IO耗性能的地方主要有两点，一是数据从从存储设备到应用程序中经历了两次复制，二是数据的处理方式是阻塞式的 1-4：NIO快就快在解决了传统IO的两个耗性能的问题，一是减少复制次数，二是数据处理线程不再阻塞，为此增加了缓存区+通道+多路复用选择器+操作系统数据缓存区 对比来看，感觉老师讲的有点凌乱，IO网络通信模型，我在很多的课程上都学过，这个几乎讲到高性能这一块是必讲的，极客时间里有好几个专栏里也都有讲，大概原理还行，不过体系和细致成度一般，可能是篇幅有限吧! 我见过最通俗易懂的讲解就是netty权威指南的李林峰，用了好几章来讲解这一块的内容。 他从IO的历史演进来讲，一个个IO通信模型是怎么来的？前一个有什么问题？后一个基本是为了解决前一个的问题而来的，以及具体是怎么解决的？ 磁盘或网络IO由于其内部结构决定和内存、各级缓存、CUP的速度有巨大的鸿沟，写操作系统的大神们和JDK的大神都清楚，所以，他们也都在绞尽脑汁来通过其他方式来尽量的解决这些问题。 希望他们的脑汁没没白绞，真心能明白他们绞尽脑汁后都产生了什么牛逼的方案。 非阻塞、零拷贝、多路复用选择器、Reactor、Preactor、DMA、epoll、通道这些概念有些理解啦有些还没，不过性能优化的原则没变还是那一套。

  作者回复: 总结的很好，这章后面需要再优化

  2019-09-07

  **2

  **8

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1590.jpeg)

  仙道

  有两个地方理解不了，请老师指点一下。 1.传统io请求数据没有的话就会挂起进入阻塞状态，既然进入了阻塞状态文中为什么还会说会继续强悍cpu。 2，传统io对文件边读边处理，NIO一次性将文件读到缓冲区，就这样的话问什么说NIO要快？我觉得单是读取的时间花费的是一样的

  作者回复: 1、阻塞会引起上下文切换，文中强调的是上下文切换； 2、如果能读到则是一样的。在没有bytebuff缓存的情况下，一旦读取数据的SO_RCVBUF满了，将会通知对端TCP协议中的窗口关闭（滑动窗口），将影响TCP发送端，这也就影响到了整个TCP通信的速度。而有了bytebuff，我们可以先将读取的数据缓存在bytebuff中，提高TCP的通信能力。

  2019-08-16

  **

  **8

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1591.jpeg)

  Haies

  讲的很精炼，不过有三个问题没明白 1.图2的directbuffer是不是临时直接内存，和文中提到的DirectBuffer应该不是一回事吧。 2.Chanel有自己的处理器，这个何解？ 3.传统I/O使用buffer后，是不是处理单位也变成块了，怎么可以优化阻塞的问题呢，不太明白？

  作者回复: 1、是通一个directbuffer，是一个临时堆外内存； 2、就是DMA处理器； 3、内存处理比磁盘处理的IO好很多。

  2019-11-24

  **2

  **5

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1592.jpeg)

  昨夜的柠檬

  文中说，多线程阻塞时，会占用大量CPU资源。线程阻塞应该会让出CPU吧？

  作者回复: 没有找到相关描述，文中是这样描述的：一旦发生线程阻塞，这些线程将会不断地抢夺 CPU 资源，从而导致大量的 CPU 上下文切换，增加系统的性能开销。 阻塞线程在阻塞状态是不会占用CPU资源的，但是会被唤醒争夺CPU资源。操作系统将CPU轮流分配给线程任务，当线程数量越多的时候，当某个线程在规定的时间片运行完之后，会被其他线程抢夺CPU资源，此时会导致上下文切换。抢夺越激烈，上下文切换就越频繁。

  2020-03-08

  **2

  **4

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1593.jpeg)

  张三丰

  如果单纯使用 Java 堆内存进行数据拷贝，当拷贝的数据量比较大的情况下，Java 堆的 GC 压力会比较大，而使用非堆内存可以减低 GC 的压力。   为何GC压力会比较大呢？只能说是没法回收导致内存泄漏吧。

  作者回复: 这里说的GC，指的是Minor GC，这里不会导致内存泄漏，会被回收的

  2020-04-07

  **

  **2

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630561-1594.png)

  天星之主

  “DirectBuffe直接将步骤简化为从内核空间复制到外部设备，减少了数据拷贝”，direct buffer申请的非堆内存，只是不受JVM管控，不应该还是用户态的内存吗

  作者回复: direct buffer是用户态内存，已更新这一小节

  2019-11-07

  **3

  **2

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1595.jpeg)

  al-byte

  我们可以在注册 Channel 时设置该通道为非阻塞，当 Channel 上没有 I/O 操作时，该线程就不会一直等待了，而是会不断轮询所有 Channel，从而避免发生阻塞。 如果一个Channel上I/O耗时很长是不是后续的Channel就被阻塞了？

  作者回复: 是的。如果I/O操作时间比较长，我们可以创建新的一个线程来执行I/O操作，避免阻塞Reactor从线程。

  2019-06-17

  **

  **2

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1596.jpeg)

  小荷才露尖尖角

  谢谢老师！ 我觉得从基础讲起再深入挺好的 有逻辑与层次感， 一上来就是好高深的内容 就会让一半道行没够的同学放弃治疗了。

  2019-06-16

  **

  **2

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1597.jpeg)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  Better me

  老师在评论中的下面这段话 阻塞线程在阻塞状态是不会占用CPU资源的，但是会被唤醒争夺CPU资源。操作系统将CPU轮流分配给线程任务，当线程数量越多的时候，当某个线程在规定的时间片运行完之后，会被其他线程抢夺CPU资源，此时会导致上下文切换。抢夺越激烈，上下文切换就越频繁。 这里抢夺越激烈，上下文切换越频繁有点不太理解，如果只是单次抢夺操作，最后不都只有一个线程抢夺到CPU从而进行一次上下文切换吗？还是这里是以抢夺资源的这整段时间维度去衡量理解的

  作者回复: 抢夺资源的整段时间维度去衡量，非自愿的上下文切换会降低线程的CPU使用效率，使得大量时间消耗在寄存器和计数器的存取之间。最直观的是查看非自愿上下文切换次数以及系统的负载。

  2020-05-24

  **

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1598.jpeg)

  xaviers

  老师，有个疑问。 Buffer 可以将文件一次性读入内存再做后续处理，而传统的方式是边读文件边处理数据。 按说边读文件边处理应该更快呀

  作者回复: 来回读写，由用户态到内核态的切换耗时，减少用户态和内核态的切换，性能更佳

  2020-04-25

  **2

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1599.jpeg)

  袁林

  根据 ‘使用 DirectBuffer 减少内存复制’ 这段描述 DirectBuffer 申请的应该是内核的内存，这是如何实现的？unix使用的是哪些系统调用？ 还请解答

  作者回复: DirectBuffer 是直接内存，这里的减少复制是减少JVM内存到直接内存的复制

  2019-08-07

  **

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1600.jpeg)

  码农Kevin亮

  请问老师，我理解传统bio之所以慢是因为它要等待数据从磁盘到内核空间再到用户空间，由于经过两次复制数据所以慢。 我没太理解nio是快在哪？只有用户空间直接到磁盘一次的复制？具体是怎么实现的呢？

  作者回复: 减少了一次中间拷贝。 Java 堆内存中拷贝到临时的直接内存中，通过临时的直接内存拷贝到内核空间中去，而NIO中的DirectBuffer是直接内存，我们无需再在JVM内存中创建对象，再拷贝到直接内存中去了，而是在直接内存中创建对象，减少了一次拷贝。

  2019-08-05

  **

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630562-1601.jpeg)

  知行合一

  思考题：是因为会很耗费cpu吗？

  作者回复: 答案已经有同学给出了，异步I/O没有在Linux内核中实现

  2019-06-10

  **

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1602.jpeg)

  vvsuperman

  建议加写例子，比如tomcat用的io造成阻塞之类，实例分析等

  编辑回复: 感谢这位同学的建议，老师会在11讲中集中补充有关IO的一些实战内容。

  2019-06-06

  **

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1587.jpeg)

  -W.LI-

  老师好!能说下哪些操作需要在用户态下完成么?正常的代码运行用户态就可以了是吗? 1.创建selector 2.创建serverSockekChannel 3.OP_ACCEPT事件注册到selector中 4.监听到OP_ACCEPT事件 5.创建channel 6.OP_READ事件注册到selector中 7.监听到READ事件 8.从channel中读数据 读的时候需要先切换到内核模式，复制在内核空间，然后从内核空间复制到用户空间。 9.处理数据  10.write:用户模式切换到内核模式，从用户空间复制到内核空间，再从内核空间发送到IO网络上。 1-7步里面有哪些操作需要在内核模式下执行的么? 第8和10我是不是理解错了? DMA啥时候起作用啊? JVM的内存属于用户空间是吧，directBuffer直接申请的物理内存，是属于特殊的用户空间么。内核模式直接往那里写。kafka的0拷贝和directbuffer一个意思么?╯﹏╰都不知道

  作者回复: 是的，directBuffer也是属于用户空间，directbuffer的零拷贝指的是通过allocate分配直接内存空间，减少堆内存与直接内存的复制，NIO中还存在一个MappedByteBuffer类，该类通过调用mmap系统方法通过内存映射的方式拷贝数据，减少用户空间到内核空间的拷贝，而kafka的拷贝也是基于内存映射的方式实现的零拷贝。

  2019-06-06

  **

  **1

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1603.jpeg)

  薇薇

  老师能不能多给点代码

  2022-08-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  Carla

  directbytebuffer继承了mappedbytebuffer，意思是directbytebuffer同时优化了jvm堆的内存拷贝和系统空间的拷贝

  2022-07-21

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1604.jpeg)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  Bumblebee

  一、字节流与的区别？ ①  字节流本身也是基于字节流的一层包装； ②  字符流的目的是给流制定编码（因为不同字符集编码单个字符所占的字节是不一样的）； 二、InputStream/OutputStream是字节输入输出流的顶层父类； 三、Reader/Writer是字符流的顶层父类； 四、nio高性能的三大利器    ①  Buffer（使用缓存优化读写流操作，Buffer 可以将文件一次性读入内存再做后续处理，而传统的方式是边读文件边处理数据）；     ②  Channel（Channel是双向的可以同时进行数据的读写）；     ③  Selector（Selector 是基于事件驱动实现的，我们可以在 Selector 中注册 accpet、read 监听事件，Selector 会不断轮询注册在其上的 Channel，如果某个 Channel 上面发生监听事件，这个 Channel 就处于就绪状态，然后进行 I/O 操作，一个线程使用一个 Selector，通过轮询的方式，可以监听多个 Channel 上的事件。我们可以在注册 Channel 时设置该通道为非阻塞，当 Channel 上没有 I/O 操作时，该线程就不会一直等待了，而是会不断轮询所有 Channel，从而避免发生阻塞）；

  2022-05-16

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630562-1605.jpeg)

  qwer

  老师 java NIO 中的channel 和 硬件层面的IO 通道有没有关系呢？如果有关系 操作系统是如何切换DMA控制方式和IO 通道的

  2022-04-12

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1606.jpeg)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  余生

  普通的 Buffer 分配的是 JVM 堆内存，而 DirectBuffer 是直接分配物理内存 (非堆内存)。------分配物理内存是指什么? 物理内存是缺页中断产生时才分配的?

  2021-07-11

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1606.jpeg)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  余生

  虽然传统 I/O 后面也使用了缓冲块，例如 BufferedInputStream，但仍然不能和 NIO 相媲美。使用 NIO 替代传统 I/O 操作，可以提升系统的整体性能，效果立竿见影。------然后呢?

  2021-07-11

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1607.jpeg)

  猪大强

  老师你好， DirectBuffer 则是直接将步骤简化为数据直接保存到非堆内存，从而减少了一次数据拷贝。以下是 JDK 源码中 IOUtil.java 类中的 write 方法： 这句话不对呢，拷贝次数没有减少呢，只是一个是堆外内存，一个是堆内存，怎么就减少了一次拷贝呢

  2021-01-29

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1608.jpeg)

  惘 闻

  我们可以在注册 Channel 时设置该通道为非阻塞，当 Channel 上没有 I/O 操作时，该线程就不会一直等待了，而是会不断轮询所有 Channel，从而避免发生阻塞。 老师的这一句话中避免线程阻塞使用轮循一直保持线程的可运行状态,这样会提升效率吗?线程又不工作,而且还导致其他线程无法获得资源工作.那不就是浪费资源吗?怎么还会提升效率呢?

  2020-12-28

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_9d0e04

  mmap直接从磁盘拷贝到用户空间，不使用pagecache吗？？如果是这样的话，那pagecache的预读功能，写合并功能都用不到了，性能是不是会很差？

  2020-11-25

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1609.jpeg)![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,w_14.png)

  与路同飞

  使用AIO是基于异步非阻塞IO，但是异步非阻塞IO在linux上比没有完全实现。又因为大多数服务器都是linux，故很多框架仍然使用NIO

  2020-09-22

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630562-1610.jpeg)

  苏籍

  为什么 Java 需要通过一个临时的非堆内存来复制数据呢？如果单纯使用 Java 堆内存进行数据拷贝，当拷贝的数据量比较大的情况下，Java 堆的 GC 压力会比较大，而使用非堆内存可以减低 GC 的压力。 ________ 为啥单纯使用 Java 堆内存进行数据拷贝 会对GC造成压力，拷贝的数据量不是一样的吗

  2020-07-26

  **1

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f37119

  这句话啥意思，挂起了还怎么争抢cpu，只有活动队列才会争抢吧，挂起的不会占用cpu. 这时如果线程没有数据就绪就会被挂起，然后进入阻塞状态。一旦发生线程阻塞，这些线程将会不断地抢夺 CPU 资源，从而导致大量的 CPU 上下文切换，增加系统的性能开销。

  2020-06-04

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1598.jpeg)

  xaviers

  老师，通道channel需要硬件的支持吗？channel和DAM之间的区别是啥？

  2020-04-25

  **1

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630561-1593.jpeg)

  张三丰

  传统的 I/O 即使使用了缓冲块，依然存在阻塞问题。由于线程池线程数量有限，一旦发生大量并发请求，超过最大数量的线程就只能等待  奇怪了 传统的IO和线程池有什么关系？？

  作者回复: 传统IO中，也有一个线程模型，可以进入这一节中的加餐篇详细阅读下

  2020-04-07

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1611.jpeg)

  10年以后

  io

  2020-03-29

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630563-1612.jpeg)

  天天向上

  Buffer 可以将文件一次性读入内存再做后续处理，而传统的方式是边读文件边处理数据。 按说边读文件边处理应该更快呀。

  2019-12-25

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630563-1613.jpeg)

  亚洲舞王.尼古拉斯赵四

  看具体情况的吧，通信框架并不是类似于向用户提供的接口，同时会有大量的访问，使用一个selector完全可以够用的情况下，创建多个线程去执行反而消耗资源

  2019-10-24

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630563-1614.jpeg)

  greekw

  老师，NIO模型里面的多路复用的线程机制能否讲下？还有零拷贝是怎么实现的？

  作者回复: 加餐篇中讲到了

  2019-10-20

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630563-1615.jpeg)

  Sdylan

  2019.10.14 打卡： 本文讲解太抽象了，最好集合例子来讲。

  2019-10-14

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630563-1616.png)

  没有小名的曲儿

  老师，我想请教一个问题，对于向服务器上传多张图片，服务器接收到后进行压缩或者合成pdf等类似的io操作，会占用很大的资源，我试着用线程池来规避数量可服务器还是承受不住这种压力，对于这种耗时比较长的操作您有什么建议吗

  作者回复: 尝试看看是否可分片并行处理。

  2019-08-19

  **4

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630563-1617.jpeg)

  Geek_94aebf

  Java后台需求要返回几百上千条数据，数据是实时变化的，每次返回数据有几百k到几m不等，非常耗费网络带宽，现在单机带宽30m，该从哪些方面优化?

  作者回复: 是否可以改用分页查询呢？或者将重要信息先传输到页面，次要的通过单次点击查看。

  2019-08-13

  **2

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630563-1618.jpeg)

  Knight²º¹⁸

  Java的IO模型和操作系统的关系是啥？

  作者回复: 可以转到11讲，有详细答案

  2019-08-11

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630563-1619.jpeg)

  L.

  老师，您github上 NIO 的例子是不是没写完整呢，客户端好像收不到服务端的的回复消息（现在时间为：）,谢谢～

  2019-08-09

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630562-1599.jpeg)

  袁林

  根据 通道（Channel）一节的描述， channel貌似越过了硬件（DMA），请问顶层是如何实现的？

  2019-08-08

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630564-1620.jpeg)

  晓晨同学

  请问老师，传统io中的buffer和nio中的buffer同样都在用户内存缓冲区，那么有什么区别呢

  作者回复: 两者的buffer并没有什么区别

  2019-08-06

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630564-1621.jpeg)

  尔冬橙

  老师，可以讲一下，直接缓冲区、内存映射地址、NIO零拷贝之间的关系么，我总觉得他们间好像很像，但是理不清

  2019-08-03

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630564-1622.jpeg)

  飞鱼

  老师，请教个问题，如果是长连接下，高并发应该怎么样设计方案更合理呢，谢谢

  作者回复: 使用异步代替同步，提高系统的并发性能

  2019-07-02

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/132-1662425630564-1623.jpeg)

  Geek_37bdff

  老师，您好，能通俗解释一下什么是同步阻塞，异步阻塞，同步非阻塞，异步非阻塞这些概念不，还有就是nio是属于同步非阻塞还是异步非阻塞，为什么

  编辑回复: 同学你好！周四即将更新的11讲答疑课堂就能解决你的问题。到时如有疑问，可以继续给老师留言。

  2019-06-12

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630564-1624.jpeg)

  z.l

  从文章的描述我猜测DirectBuffer属于内核空间的内存，但java作为用户进城是如何操作内核空间内存的呢？

  作者回复: 文章已修正。不是的，DirectBuffer是直接内存，也属于用户空间。

  2019-06-09

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_801517

  老师，我也觉得今天的内容浅了一点，nio的多路复用也可以深入讲下或者netty的实现，epoll这些也是，还有其他io模型也可以对比下

  编辑回复: 收到～老师会集中大家的留言，在11讲答疑课堂中做出补充讲解。感谢你的建议！

  2019-06-09

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630564-1625.jpeg)

  晓杰

  老师，channel只是解决了内核空间和磁盘之前的io操作问题，那用户空间和内核空间之间的来回复制是不是依然是一个耗时的操作

  作者回复: 这讲中提到了零拷贝，用DirectBuffer减少内存复制，也就是避免了用户空间与内核空间来回复制。

  2019-06-07

  **

  **

- ![img](08%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8BI_O%E6%A8%A1%E5%9E%8B%EF%BC%9A%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3%E9%AB%98%E5%B9%B6%E5%8F%91%E4%B8%8BI_O%E7%93%B6%E9%A2%88%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425630564-1626.jpeg)

  圣西罗

  使用webflux的过程中最大的不方便是不支持threadlocal,导致像创建人修改人id的赋值需要明传参数

  2019-06-06

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  胖妞

  git上的测试案例有吗？想很多通过时间具体对比一下！总感觉讲的有点抽象和概念了，脑子里没有形成一个具体的形象！希望能给几个小demo看一下！麻烦了！

  作者回复: git上有源码，分别是io和nio的简单实现的demo。如果需要通过简单的代码测试比对两者的性能，可以自己尝试一下，有疑问可以再问老师。

  2019-06-06

  **

  **
