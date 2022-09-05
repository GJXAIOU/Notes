# 第13讲 \| 套接字Socket：Talk is cheap, show me the code

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/a9/c6/a9b6972262137a72fdaa0d06916ac0c6.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/75/6f/75b47bb7d89a675b096c50ed84d2da6f.mp3" type="audio/mpeg"></audio>

前面讲完了TCP和UDP协议，还没有上手过，这一节咱们讲讲基于TCP和UDP协议的Socket编程。

在讲TCP和UDP协议的时候，我们分客户端和服务端，在写程序的时候，我们也同样这样分。

Socket这个名字很有意思，可以作插口或者插槽讲。虽然我们是写软件程序，但是你可以想象为弄一根网线，一头插在客户端，一头插在服务端，然后进行通信。所以在通信之前，双方都要建立一个Socket。

在建立Socket的时候，应该设置什么参数呢？Socket编程进行的是端到端的通信，往往意识不到中间经过多少局域网，多少路由器，因而能够设置的参数，也只能是端到端协议之上网络层和传输层的。

在网络层，Socket函数需要指定到底是IPv4还是IPv6，分别对应设置为AF\_INET和AF\_INET6。另外，还要指定到底是TCP还是UDP。还记得咱们前面讲过的，TCP协议是基于数据流的，所以设置为SOCK\_STREAM，而UDP是基于数据报的，因而设置为SOCK\_DGRAM。

## 基于TCP协议的Socket程序函数调用过程

两端创建了Socket之后，接下来的过程中，TCP和UDP稍有不同，我们先来看TCP。

TCP的服务端要先监听一个端口，一般是先调用bind函数，给这个Socket赋予一个IP地址和端口。为什么需要端口呢？要知道，你写的是一个应用程序，当一个网络包来的时候，内核要通过TCP头里面的这个端口，来找到你这个应用程序，把包给你。为什么要IP地址呢？有时候，一台机器会有多个网卡，也就会有多个IP地址，你可以选择监听所有的网卡，也可以选择监听一个网卡，这样，只有发给这个网卡的包，才会给你。

<!-- [[[read_end]]] -->

当服务端有了IP和端口号，就可以调用listen函数进行监听。在TCP的状态图里面，有一个listen状态，当调用这个函数之后，服务端就进入了这个状态，这个时候客户端就可以发起连接了。

在内核中，为每个Socket维护两个队列。一个是已经建立了连接的队列，这时候连接三次握手已经完毕，处于established状态；一个是还没有完全建立连接的队列，这个时候三次握手还没完成，处于syn\_rcvd的状态。

接下来，服务端调用accept函数，拿出一个已经完成的连接进行处理。如果还没有完成，就要等着。

在服务端等待的时候，客户端可以通过connect函数发起连接。先在参数中指明要连接的IP地址和端口号，然后开始发起三次握手。内核会给客户端分配一个临时的端口。一旦握手成功，服务端的accept就会返回另一个Socket。

这是一个经常考的知识点，就是监听的Socket和真正用来传数据的Socket是两个，一个叫作**监听Socket**，一个叫作**已连接Socket**。

连接建立成功之后，双方开始通过read和write函数来读写数据，就像往一个文件流里面写东西一样。

这个图就是基于TCP协议的Socket程序函数调用过程。

![](<https://static001.geekbang.org/resource/image/87/ea/87c8ae36ae1b42653565008fc47aceea.jpg?wh=1626*2172>)

说TCP的Socket就是一个文件流，是非常准确的。因为，Socket在Linux中就是以文件的形式存在的。除此之外，还存在文件描述符。写入和读出，也是通过文件描述符。

在内核中，Socket是一个文件，那对应就有文件描述符。每一个进程都有一个数据结构task\_struct，里面指向一个文件描述符数组，来列出这个进程打开的所有文件的文件描述符。文件描述符是一个整数，是这个数组的下标。

这个数组中的内容是一个指针，指向内核中所有打开的文件的列表。既然是一个文件，就会有一个inode，只不过Socket对应的inode不像真正的文件系统一样，保存在硬盘上的，而是在内存中的。在这个inode中，指向了Socket在内核中的Socket结构。

在这个结构里面，主要的是两个队列，一个是**发送队列**，一个是**接收队列**。在这两个队列里面保存的是一个缓存sk\_buff。这个缓存里面能够看到完整的包的结构。看到这个，是不是能和前面讲过的收发包的场景联系起来了？

整个数据结构我也画了一张图。<br>

![](<https://static001.geekbang.org/resource/image/60/13/604f4cb37576990b3f836cb5d7527b13.jpg?wh=3646*2998>)

## 基于UDP协议的Socket程序函数调用过程

对于UDP来讲，过程有些不一样。UDP是没有连接的，所以不需要三次握手，也就不需要调用listen和connect，但是，UDP的交互仍然需要IP和端口号，因而也需要bind。UDP是没有维护连接状态的，因而不需要每对连接建立一组Socket，而是只要有一个Socket，就能够和多个客户端通信。也正是因为没有连接状态，每次通信的时候，都调用sendto和recvfrom，都可以传入IP地址和端口。

这个图的内容就是基于UDP协议的Socket程序函数调用过程。<br>

![](<https://static001.geekbang.org/resource/image/6b/31/6bbe12c264f5e76a81523eb8787f3931.jpg?wh=1245*1261>)

## 服务器如何接更多的项目？

会了这几个基本的Socket函数之后，你就可以轻松地写一个网络交互的程序了。就像上面的过程一样，在建立连接后，进行一个while循环。客户端发了收，服务端收了发。

当然这只是万里长征的第一步，因为如果使用这种方法，基本上只能一对一沟通。如果你是一个服务器，同时只能服务一个客户，肯定是不行的。这就相当于老板成立一个公司，只有自己一个人，自己亲自上来服务客户，只能干完了一家再干下一家，这样赚不来多少钱。

那作为老板你就要想了，我最多能接多少项目呢？当然是越多越好。

我们先来算一下理论值，也就是**最大连接数**，系统会用一个四元组来标识一个TCP连接。

```
{本机IP, 本机端口, 对端IP, 对端端口}
```

服务器通常固定在某个本地端口上监听，等待客户端的连接请求。因此，服务端端TCP连接四元组中只有对端IP, 也就是客户端的IP和对端的端口，也即客户端的端口是可变的，因此，最大TCP连接数=客户端IP数×客户端端口数。对IPv4，客户端的IP数最多为2的32次方，客户端的端口数最多为2的16次方，也就是服务端单机最大TCP连接数，约为2的48次方。

当然，服务端最大并发TCP连接数远不能达到理论上限。首先主要是**文件描述符限制**，按照上面的原理，Socket都是文件，所以首先要通过ulimit配置文件描述符的数目；另一个限制是**内存**，按上面的数据结构，每个TCP连接都要占用一定内存，操作系统是有限的。

所以，作为老板，在资源有限的情况下，要想接更多的项目，就需要降低每个项目消耗的资源数目。

### 方式一：将项目外包给其他公司（多进程方式）

这就相当于你是一个代理，在那里监听来的请求。一旦建立了一个连接，就会有一个已连接Socket，这时候你可以创建一个子进程，然后将基于已连接Socket的交互交给这个新的子进程来做。就像来了一个新的项目，但是项目不一定是你自己做，可以再注册一家子公司，招点人，然后把项目转包给这家子公司做，以后对接就交给这家子公司了，你又可以去接新的项目了。

这里有一个问题是，如何创建子公司，并如何将项目移交给子公司呢？

在Linux下，创建子进程使用fork函数。通过名字可以看出，这是在父进程的基础上完全拷贝一个子进程。在Linux内核中，会复制文件描述符的列表，也会复制内存空间，还会复制一条记录当前执行到了哪一行程序的进程。显然，复制的时候在调用fork，复制完毕之后，父进程和子进程都会记录当前刚刚执行完fork。这两个进程刚复制完的时候，几乎一模一样，只是根据fork的返回值来区分到底是父进程，还是子进程。如果返回值是0，则是子进程；如果返回值是其他的整数，就是父进程。

进程复制过程我画在这里。<br>

![](<https://static001.geekbang.org/resource/image/18/d0/18070c00ff5d0082yy1fbc32b84e73d0.jpg?wh=4842*3559>)

因为复制了文件描述符列表，而文件描述符都是指向整个内核统一的打开文件列表的，因而父进程刚才因为accept创建的已连接Socket也是一个文件描述符，同样也会被子进程获得。

接下来，子进程就可以通过这个已连接Socket和客户端进行互通了，当通信完毕之后，就可以退出进程，那父进程如何知道子进程干完了项目，要退出呢？还记得fork返回的时候，如果是整数就是父进程吗？这个整数就是子进程的ID，父进程可以通过这个ID查看子进程是否完成项目，是否需要退出。

### 方式二：将项目转包给独立的项目组（多线程方式）

上面这种方式你应该也能发现问题，如果每次接一个项目，都申请一个新公司，然后干完了，就注销掉这个公司，实在是太麻烦了。毕竟一个新公司要有新公司的资产，有新的办公家具，每次都买了再卖，不划算。

于是你应该想到了，我们可以使用**线程**。相比于进程来讲，这样要轻量级的多。如果创建进程相当于成立新公司，购买新办公家具，而创建线程，就相当于在同一个公司成立项目组。一个项目做完了，那这个项目组就可以解散，组成另外的项目组，办公家具可以共用。

在Linux下，通过pthread\_create创建一个线程，也是调用do\_fork。不同的是，虽然新的线程在task列表会新创建一项，但是很多资源，例如文件描述符列表、进程空间，还是共享的，只不过多了一个引用而已。

![](<https://static001.geekbang.org/resource/image/a3/64/a36537201678e08ac83e5410562d5f64.jpg?wh=4628*3401>)

新的线程也可以通过已连接Socket处理请求，从而达到并发处理的目的。

上面基于进程或者线程模型的，其实还是有问题的。新到来一个TCP连接，就需要分配一个进程或者线程。一台机器无法创建很多进程或者线程。有个**C10K**，它的意思是一台机器要维护1万个连接，就要创建1万个进程或者线程，那么操作系统是无法承受的。如果维持1亿用户在线需要10万台服务器，成本也太高了。

其实C10K问题就是，你接项目接的太多了，如果每个项目都成立单独的项目组，就要招聘10万人，你肯定养不起，那怎么办呢？

### 方式三：一个项目组支撑多个项目（IO多路复用，一个线程维护多个Socket）

当然，一个项目组可以看多个项目了。这个时候，每个项目组都应该有个项目进度墙，将自己组看的项目列在那里，然后每天通过项目墙看每个项目的进度，一旦某个项目有了进展，就派人去盯一下。

由于Socket是文件描述符，因而某个线程盯的所有的Socket，都放在一个文件描述符集合fd\_set中，这就是**项目进度墙**，然后调用select函数来监听文件描述符集合是否有变化。一旦有变化，就会依次查看每个文件描述符。那些发生变化的文件描述符在fd\_set对应的位都设为1，表示Socket可读或者可写，从而可以进行读写操作，然后再调用select，接着盯着下一轮的变化。

### 方式四：一个项目组支撑多个项目（IO多路复用，从“派人盯着”到“有事通知”）

上面select函数还是有问题的，因为每次Socket所在的文件描述符集合中有Socket发生变化的时候，都需要通过轮询的方式，也就是需要将全部项目都过一遍的方式来查看进度，这大大影响了一个项目组能够支撑的最大的项目数量。因而使用select，能够同时盯的项目数量由FD\_SETSIZE限制。

如果改成事件通知的方式，情况就会好很多，项目组不需要通过轮询挨个盯着这些项目，而是当项目进度发生变化的时候，主动通知项目组，然后项目组再根据项目进展情况做相应的操作。

能完成这件事情的函数叫epoll，它在内核中的实现不是通过轮询的方式，而是通过注册callback函数的方式，当某个文件描述符发送变化的时候，就会主动通知。<br>

![](<https://static001.geekbang.org/resource/image/d6/b1/d6efc5c5ee8e48dae0323de380dcf6b1.jpg?wh=4410*2335>)

如图所示，假设进程打开了Socket m, n, x等多个文件描述符，现在需要通过epoll来监听是否这些Socket都有事件发生。其中epoll\_create创建一个epoll对象，也是一个文件，也对应一个文件描述符，同样也对应着打开文件列表中的一项。在这项里面有一个红黑树，在红黑树里，要保存这个epoll要监听的所有Socket。

当epoll\_ctl添加一个Socket的时候，其实是加入这个红黑树，同时红黑树里面的节点指向一个结构，将这个结构挂在被监听的Socket的事件列表中。当一个Socket来了一个事件的时候，可以从这个列表中得到epoll对象，并调用call back通知它。

这种通知方式使得监听的Socket数据增加的时候，效率不会大幅度降低，能够同时监听的Socket的数目也非常的多了。上限就为系统定义的、进程打开的最大文件描述符个数。因而，**epoll被称为解决C10K问题的利器**。

## 小结

好了，这一节就到这里了，我们来总结一下：

- 你需要记住TCP和UDP的Socket的编程中，客户端和服务端都需要调用哪些函数；

- 写一个能够支撑大量连接的高并发的服务端不容易，需要多进程、多线程，而epoll机制能解决C10K问题。


<!-- -->

最后，给你留两个思考题：

1. epoll是Linux上的函数，那你知道Windows上对应的机制是什么吗？如果想实现一个跨平台的程序，你知道应该怎么办吗？

2. 自己写Socket还是挺复杂的，写个HTTP的应用可能简单一些。那你知道HTTP的工作机制吗？


<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(97)

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34.jpeg)

  进阶的码农

  置顶

  AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)。 我根据图中例子计算 14-((5-1)-0) 算出来是10 ，括号里边的-1是减的什么，为啥和图例算出来的结果不一样，还是我计算的有问题，麻烦详细说一下 谢谢

  作者回复: 这里我写的的确有问题，nextbyteexpected其实是6，就是目前接收到五，下一个期望的是六，这样就对了

  2018-06-13

  **6

  **46

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507933-6588.jpeg)

  小文同学

  1 设备缓存会导致延时？ 假如经过设备的包都不需要进入缓存，那么得到的速度是最快的。进入缓存且等待，等待的时间就是额外的延时。BBR就是为了避免这些问题： 充分利用带宽；降低buffer占用率。 2 降低发送packet的速度，为何反而提速了？ 标准TCP拥塞算法是遇到丢包的数据时快速下降发送速度，因为算法假设丢包都是因为过程设备缓存满了。快速下降后重新慢启动，整个过程对于带宽来说是浪费的。通过packet速度-时间的图来看，从积分上看，BBR充分利用带宽时发送效率才是最高的。可以说BBR比标准TCP拥塞算法更正确地处理了数据丢包。对于网络上有一定丢包率的公网，BBR会更加智慧一点。 回顾网络发展过程，带宽的是极大地改进的，而最小延迟会受限与介质传播速度，不会明显减少。BBR可以说是应运而生。 3 BBR如何解决延时？ S1：慢启动开始时，以前期的延迟时间为延迟最小值Tmin。然后监控延迟值是否达到Tmin的n倍，达到这个阀值后，判断带宽已经消耗尽且使用了一定的缓存，进入排空阶段。 S2：指数降低发送速率，直至延迟不再降低。这个过程的原理同S1 S3：协议进入稳定运行状态。交替探测带宽和延迟，且大多数时间下都处于带宽探测阶段。 深夜读了BBR的论文和网上大牛的讲解得出的小结，分享给大家，过程比较匆忙，不足之处也望老师能指出指正。

  2018-06-14

  **5

  **213

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507933-6589.jpeg)

  大唐江山

  作者辛苦啊，这一章读起来开始有点累了😂

  2018-06-13

  **1

  **82

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507933-6590.jpeg)

  刘培培

  BBR 论文原文：https://queue.acm.org/detail.cfm?id=3022184

  作者回复: 就应该这样，一言不合就读论文，是最好的方式

  2019-08-27

  **4

  **57

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  华林

  这么说起来，我们经常发现下载速度是慢慢的增加到顶峰，然后就在那个值的附近徘徊，原因就是tcp的流量控制喽？

  2018-06-13

  **9

  **33

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507933-6591.jpeg)

  食用淡水鱼

  快速重传那一段有问题。接收端接收6，7，8，9，10时漏掉了7，不是连续发送3个6ack，而是收到6，发送6的确认ack，收到8，9，10各发送一个6的重复ack，发送端检测到3个重复ack时（加上确认ack，总共有4个ack），进入重传机制。包括下面的拥塞控制，也是类似的逻辑

  2019-01-21

  **3

  **24

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6592.jpeg)

  谛听

  不太清楚累积应答，比如接收端收到了包1、2、3、4，它的应答应该是5吗？也就是说中间的包就不用应答了吗？

  作者回复: 是的

  2018-09-22

  **4

  **20

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6593.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  Null

  BBR 不填满缓存还是不填缓存？不填缓存那么缓存干啥用，如果填了了，即使不满，但是不是还有延迟。。。

  作者回复: 填的少，延迟就少了，当然做不到完全避免，毕竟缓存是路径上每一个设备自己的事情。缓存是每个设备自己的设计选择，BBR算法是两端的算法。 就像买火车票，我建议网上购买，身份证刷进去，这样速度快，但是对于火车站还是要设置人工窗口，因为不是每个人都会选择速度快的方式的。

  2019-03-27

  **

  **17

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  卜

  好奇什么级别的程序开发需要了解怎么细，开发了好多网络程序都没用到，重传这些都是应用层在做

  2018-06-24

  **6

  **12

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6594.jpeg)

  秋去冬来

  快速重传那块6.8.9   7丢了为什么会发送3个冗余的6的3个ack

  作者回复: 六的ack里面强调下一个是七

  2018-06-22

  **

  **11

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6595.jpeg)

  MoFanDon

  5的ACK丢包了，出发发送端重发。重发过去后，接收端发现接收过了，丢弃。……如果接收端丢弃5后，没有继续发送ACK,那这样不是发送端永远也也没法接受到5的ACK？

  2018-07-24

  **3

  **10

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6596.png)

  扬～

  2个问题： 1. TCP可靠的连接会不会影响到业务层，比如超时重传导致了服务端函数调用2次，那岂不是业务都要考虑幂等性了，我很懵逼，果然是懂得越多越白痴。 2. 拥塞控制的窗口跟流量控制的窗口一回事吗，还是流量控制的窗口的出来后就会进入拥塞控制窗口？

  作者回复: 不会，重传的包不会交给应用层。是一个窗口

  2018-11-01

  **9

  **9

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6597.jpeg)

  咖啡猫口里的咖啡猫🐱

  老师，TCP协议栈，保证包一定到吗，，哪几种情况下会丢失，，，能不能总结下

  作者回复: 不能保证，只是尽力重试，再重试

  2018-06-13

  **2

  **9

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6598.jpeg)

  茫农

  有一个值 ssthresh 为 65535 个字节，，这个是什么意思？

  作者回复: slow start threshold

  2018-10-23

  **2

  **8

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6599.jpeg)

  行者

  问题1，想到BBR可以根据ACK时间来判断，比如同一时刻发送了A、B、C三个包，A、B两个包10ms收到ACK，而C包20ms后收到ACK，那么就认为网络拥堵或被中间设备缓存，降低发送速度。 问题2，TCP优点在于准确到达，可靠性高，但是速度慢；UDP优点在于简单，但是不确认可达；像后端接口一般使用TCP协议，因为客户端和服务器之间会有多次交互，且请求数据要确认可达；但是如果是直播的话使用UDP会更好，管你网络咋样，反正我赶紧发，让用户等急了就不好了。 刘老师，有一个问题，接口有时候会受到2次相同时间相同内容的请求，但是客户端仅仅调用了接口一次，会是什么原因导致这个问题呢？TCP重复发送包导致的嘛？

  2018-06-13

  **1

  **7

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6600.jpeg)

  Magic

  祝刘超老师教师节快乐，专栏很棒，受益良多

  作者回复: 谢谢

  2019-09-10

  **

  **7

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6601.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  Yangjing

  对于发送端，为什么会保存着“已发送并已确认”的数据呢？已确认的不是已经没用了吗？

  作者回复: 是的，可以回收了，这里是分一下类

  2019-06-22

  **

  **4

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6602.jpeg)

  黑猫紧张

  内核是将从网络接收的tcp数据 都接收完成再一次发给应用层呢 还是在tcp接收的过程中就已经开始发给应用层了 求回复

  作者回复: 接收的过程中就发给应用层了

  2019-05-29

  **2

  **4

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6603.jpeg)

  caohuan

  本篇所得： 1.数据在不同客户端传输 需要经过网关，在TCP传输层，用有不同任务的编号JIRA ID 标注 那些 包 已发送，哪些包 有反馈，灵活的做到 4个分类（1）发送 完成 且有反馈 （2）发送完成 没有反馈（3）将发送（4）不会发送，这可能是没有 带宽和空间，也可以知道 那些包 的ack丢了、数据丢了，这个可以使用 滑动窗口 的缓存机制 来处理，然后 通过调节 滑动窗口的大小 灵活应对 顺序问题 和丢包 问题 以及流量控制，最后还可以 灵活给TCP 指派 任务量，多少可以完成，多少可以待处理，多少不能被处理。 2.拥塞控制 处理 网络带宽塞满，防止 包丢失，超时重传 带来的 低延时 和 带宽利用率低的情况，所以 采用 TCP BBR拥塞算法 提高带宽利用率 和 低延时，并且不占缓存 的 高效方法。 回答老师的问题，第一个问题1.TCP的BBR很牛，如何达到最优点，老师 提到过 （1）高效率利用 带宽，让带宽 处于 满贯状态 （2）不占用缓存空间（3）低延时， 一旦空间被 占满，会流入 缓冲空间时，马上降速，降低传输速度，一直到有空余空间 再慢启动 提高 带宽利用率。 第二个问题：1.UDP和TCP应用在不同的应用场景下， UDP性善论，网络 一片和谐，有任务只管发送 和 接受，不太理会 网络环境 和 资源，不会主动调整速度;TCP 性恶论，世界很黑暗，得制定规则，如果 网络资源好，多传输数据，不好的环境，降低 传输速度，还得注重 顺序问题、丢包问题、控制情况、带宽流量等现实情况。 2.TCP的坑;TCP需要 考虑 带宽资源、阻塞、延时等各种问题，里面设计很多算法和数据结构，都没完全捋清 里面的原理，还不知道 坑在哪里，期待继续听 刘超老师的课程，都遇问题，然后解决问题 ，来填补TCP的坑。

  2019-01-16

  **

  **4

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6604.jpeg)

  产品助理

  问题： 1、如下公式的 -1  到底是为什么？ 	AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)    图例中的LastByteRead是0还是1？NextByteExpected是6还是5？MaxRcvBuffer是14吗？ 2、如果按照上述公式，那下面又是为了什么？ 	NextByteExpected 加 AdvertisedWindow 就是第二部分和第三部分的分界线，其实也就是 LastByteRead 加上 MaxRcvBuffer。 	按照第一条的公式，NextByteExpected + AdvertisedWindow = NextByteExpected + （MaxRcvBuffer-((NextByteExpected-1)-LastByteRead)) = MaxRcvBuffer + 1 + LastByteRead 	应该有个+1啊。。 多谢老师！	

  作者回复: 这里我写的的确有问题，nextbyteexpected其实是6，就是目前接收到五，下一个期望的是六，这样就对了 2018-06-13

  2018-11-22

  **2

  **4

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507935-6605.png)

  旭风

  在传统算法和快速算法的对比描述中，对于快速算法中提到 cwnd减半为cwnd/2，sshthresh=cwnd ，后面cwnd=sshthresh+3，转折点是20变为13，这里的cwnd为10吗？ 2018-06-15  作者回复 cwnd先是为10，后面变为13 继上一个问题，传统算法，cwnd 从1,2,4,8,16指数增长，这时cwnd为16，收发是不是各占8个？然后由16变成17，满8个确认时cwnd＋1，就是17，那么增加到20这里，cwnd为20，这时产生拥塞，如果是传统的话，立马降下来，如果是快速，那么减半，也就是10，等待3个确认包＋3，，后续每一个确认cwnd就＋1吗？还是？这样子理解吗？

  作者回复: 是的

  2018-06-19

  **

  **4

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507936-6606.jpeg)

  雷刚

  \1. rwnd：滑动窗口。上一讲提到的 TCP 报文中的 windown 字段，表示接收方将能接收的数据大小告知发送发方。 2. cwnd：拥塞窗口。TCP 发送方维护的一个变量，表示发送方一次要发送多少字节的数据。 总而言之，rwnd 用来控制能发送多少数据，cwnd 控制发送速度。这就和我们吃饭一样，虽然能吃两碗饭，但你不能一口就全部吃完，还是要一口口吃。

  2020-04-09

  **

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  这一章感觉比前面好一些, 比喻比较形象.也挺详细. 谢谢, 就是有一点音频和文字不完全一样, 有时候边看变听有点跟不上

  作者回复: 音频更能表达我的思想，哈哈哈

  2019-05-21

  **

  **3

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6607.jpeg)

  古夜

  最后一个图是画的最清楚的了🤷‍♂️

  2019-02-23

  **

  **3

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6608.jpeg)

  拿笔小星

  怎么办，越到后面越看不懂了，没有网络基础，老湿有什么建议吗？

  2018-12-08

  **2

  **3

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6609.jpeg)

  lyz

  jira太真实了

  2018-07-24

  **

  **3

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507933-6590.jpeg)

  刘培培

  https://medium.com/google-cloud/tcp-bbr-magic-dust-for-network-performance-57a5f1ccf437

  2019-08-27

  **

  **2

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6610.jpeg)

  李小四

  网络_12 看了一些资料，还是没有最终弄明白，只知道了Cubic是基于丢包的，BBR是基于测量的。 关于UDP与TCP的程序，它们的选型以及“坑”都与特点密切相关。 - UDP特点是快，不可靠。所以在需要无法容忍高时延的场景要选择它，当然，这个时候，一些必要校验和重发逻辑就留给了应用层，这里应该是最大的“坑”。 - TCP特点是可靠，慢。在我开发过的程序中，“坑”比较多地存在于长连接的keepalive阶段，需要在资源消耗与稳定性之间取得平衡。

  作者回复: 加油，多看几遍

  2019-08-07

  **

  **2

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6611.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  Abirdcfly

  问个问题。最刚开始ssthres 是65535字节.然后一个mss是1460字节。那么指数增长应该是到44才变慢啊。为啥是16呢？

  作者回复: 举个例子而已，重点是那个曲线，而非数值

  2019-06-13

  **

  **2

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507936-6612.png)

  Geek_hustnw

  （1）文章提到下面这段话：    每收到一个确认后，cwnd 增加 1/cwnd，我们接着上面我们接着上面的过程来，一次发送八个，当八个确认到来的时候，每... （2）然后快速重传算法里面提到拥塞时：    cwnd 减半为 cwnd/2，然后 sshthresh =... 问题：快速重传拥塞处理里面，为什么遇到3个包返回的时候，是“sshthresh + 3”，而不是“sshthresh + 3/cwnd”

  作者回复: 先慢启动一把

  2019-04-14

  **

  **2

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6613.jpeg)

  陈栋

  第三部分和第四部分分界线写错了，应该是lastByteSent+advertisedWindow

  作者回复: 没错的，advertisewindow是那个黑框，不是接下来能发的部分

  2018-06-19

  **

  **2

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507935-6605.png)

  旭风

  在传统算法和快速算法的对比描述中，对于快速算法中提到 cwnd减半为cwnd/2，sshthresh=cwnd ，后面cwnd=sshthresh+3，转折点是20变为13，这里的cwnd为10吗？

  作者回复: cwnd先是为10，后面变为13

  2018-06-15

  **

  **2

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6614.jpeg)

  罗辑思维

  接受方的缓存数据分为三部分：接受并确认，等待接受未确认，不能接受。 有个不理解的地方 如果对应发送方的几部分，接受方不是应该有个「接受未确认」。

  2020-03-08

  **2

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507936-6615.jpeg)

  lakeslove

  解释一下ssthresh： 慢启动阈值「ssthresh」的概念，如果「cwnd」小于「ssthresh」，那么表示在慢启动阶段；如果「cwnd」大于「ssthresh」，那么表示在拥塞避免阶段，此时「cwnd」不再像慢启动阶段那样呈指数级整整，而是趋向于线性增长，以期避免网络拥塞，此阶段有多种算法实现。

  2020-03-03

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507937-6616.jpeg)

  我D

  发送端和接收端的角色是固定的吗？一直都是单向的？还有，能讲一下思考题吗，自己找的不一定对啊，又不是老师可以修改作业，那不是白学了

  2020-02-06

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507937-6617.jpeg)

  尔冬橙

  为什么填缓存还会加大时延，有缓存不是更快了么

  作者回复: 这是指网络设备的缓存

  2019-07-30

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507937-6618.jpeg)

  zjw

  所以我每次用uplay这种服务器很烂的应用下游戏时速度有时候会一下子掉下来，然后又慢慢回上去。是因为tcp在传输过程中丢包了，然后自动降低了传输速度了吗

  作者回复: 很可能是的

  2019-07-14

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507937-6619.png)

  InfoQ_0ef441e6756e

  请问老师，比如发送方 发送了 【1，2，3，4】，接收方回了3的ack，这种情况窗口怎么动啊？   

  作者回复: 累积ack，前三个都算结束了

  2019-07-07

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507938-6620.jpeg)

  Jerry银银

  请教老师一个概念性的问题：packet   chunk   segment  block   这几个词在“网络”文章或者书籍中经常初出现，感觉它们很相似，又感觉它们又有些细微的区别，但是具体的也说不清！

  作者回复: 不同层次的叫法不一样

  2019-05-17

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507938-6621.png)

  wildmask

  如果接收方发现7丢了，为什么不能直接要求重发7，发送3个ack 有必要吗？

  作者回复: 3个ack就表示要求重发7呀，要不然咋要求。

  2019-05-07

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507938-6622.jpeg)

  Ronnyz

  接收端是采用累计确认机制的吗

  作者回复: 是的

  2019-04-17

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507938-6623.jpeg)

  德道

  当三个包返回的时候，cwnd = sshthresh + 3，当三个包返回是什么意思，为什么要加3

  作者回复: 每返回一个加一

  2019-04-09

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6593.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  Null

  而且，老师行文过程，哪些是现象，问题，算法解决方案，目前最优，或者广泛采用的是哪个没有清晰的脉络，很容易看完一遍一头雾水。

  作者回复: 哪有最优，都是根据各自的场景做选择而已。

  2019-03-27

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507938-6624.jpeg)

  赵加兴

  不是很理解第三部分和第四部分的分界线，老师能说明下么

  作者回复: 超过这个不能发送了

  2019-01-14

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6625.jpeg)

  (-.-)

  "快速重传，接收方发现 6、8、9 都已经接收了，就是 7 没来，发送三个6的ack，要求下一个是7.客户端收到 3 个，就会发现 7 的确又丢了，不等超时.." 这个不应该是服务端收到三个6的ack包吗?

  作者回复: 客户端收到6的ack，就说明7不在，因为6的ack里面要求下一个是7

  2019-01-07

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6626.jpeg)

  yinhua_upc

  "6、7 是等待接收的；8、9 是已经接收，但是没有 ACK 的" 从接收端的图看6-14都是等待接收未确认，为什么8、9是已经接收？

  作者回复: 假设，所以画了实线

  2018-11-09

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6627.jpeg)

  GL

  网络层分片和传输层分片是怎么进行的？

  2018-06-22

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6628.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  初学者

  fack 应该不是收到一个大于期望的包发送三个ack，收到一个包只会回复这个ack吧

  2018-06-14

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6629.jpeg)

  正是那朵玫瑰

  为大牛点赞，有点复杂，先慢慢消化下；先问个问题，tcp建立连接后，每发送一个数据包都要像三次握手一样来回三次么，发送数据-发送ack-回复ack，才算数据发送成功？

  2018-06-13

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6630.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  人什么人

  老师, 慢启动门限到底是sshthresh还是sshresh

  2022-07-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  GMA029

  接收端的数据结构不清晰。语音说6和7是虚线，但是看图片还是实线。这对初学者很容易造成困扰

  2022-06-26

  **

  **1

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6631.jpeg)

  LIKE

  还请各位大哥大姐们帮忙解答，问题： 1.接收端的LastByteRead 在哪个位置？ 2.6、7、8、9 肯定都发了，但是 8、9 已经到了，但是 6、7 没到，出现了乱序，缓存着但是没办法 ACK。   从哪里可以看到 6和7没有到？

  2022-05-05

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507939-6632.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  赌神很低调

  请问下老师，如何写好高性能的网络程序，尽量占用带宽，比如大数据传输，想尽量把带宽打满。如万兆网卡的两个节点传数据，最大带宽可以达到1GB/s,但我们写的程序测出吞吐量最大约为350MB/s，请问可以通过哪些手段来提高吞吐量，逼近带宽呢？

  2022-04-27

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507940-6633.png)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  张可夫斯基

  数学不好，这个点+点=范围？

  2022-03-02

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507940-6634.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  i_chase

  文章里提到的也是教科书里的拥塞控制算法，是Reno算法。Linux默认的应该是cubic拥塞控制算法，三次函数，初始增速较快。BBR是基于带宽时延积计算判断网络拥塞情况的，reno和cubic是根据丢包判断网络是否拥堵。文章里也说了，丢包无法准确反映出网络拥塞（管子漏水，中间设备缓冲区的存在）

  2021-12-31

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507940-6635.jpeg)

  多选参数

  “cwnd 减半为 cwnd/2，然后 sshthresh = cwnd，当三个包返回的时候，cwnd = sshthresh + 3，也就是没有一夜回到解放前，而是还在比较高的值，呈线性增长。” +3 是怎么来？

  2021-10-28

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507940-6636.jpeg)

  大年

  强烈建议作者重写下这节，既然有错误为什么不修改呢？我们作为消费者是花钱买的课想要更准确的知识 输出，这里不是论坛的免费分享

  2021-10-24

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507940-6637.jpeg)

  dbant

  参考一起看：https://queue.acm.org/detail.cfm?id=3022184

  2021-03-29

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507941-6638.jpeg)

  fastkdm

  老师，我看网上对于低能窗口综合症，提到了Nagle算法和延迟ACK机制，能讲一下两者的关系吗，就比如说网络中默认启用的机制是什么，什么情况下会启用

  2021-02-03

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  K菌无惨

  感觉把滑动窗口改成接收窗口就比较好理解了

  2021-01-27

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507941-6639.jpeg)

  团

  老师好，对发送速度和窗口大小的算法关系还不是十分清楚，文中说通过控制窗口大小来控制发送速度，具体是怎么一个算法？比如窗口是14时，对应的是每秒发送多少个包或者多少字节？窗口大小为8时又是对应每秒发送多少包或多少字节。比如文中拥塞控制时的举例，是每秒发送一个包，窗口大小和带宽容量有关，那这样看来窗口大小和‘每秒发送一个包’这个速度没有什么直接联系？文中有一处讲如果发送方发的过猛，把四个包都发送了，这个’猛’是怎么控制的？

  2020-09-29

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507942-6640.jpeg)

  lixf

  老师，这节课逻辑很清楚啊，点个赞

  2020-08-27

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507942-6641.jpeg)

  啦啦啦

  哈哈，正好之前系统查redis慢，sar发现redis有丢包的情况，明天研究研究

  2020-08-12

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507942-6642.jpeg)

  连边

  开始吃力，记录一下。

  2020-07-09

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507943-6643.jpeg)

  ai

  拥塞控制和流量有什么区别, 流量控制窗口不是已经控制了发送者频率了吗?

  2020-07-06

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507943-6644.jpeg)

  小侠

  图非常形象

  作者回复: 谢谢

  2020-06-01

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507943-6645.jpeg)

  谭倩倩_巴扎黑

  不是说累计确认累计应答么，为什么我看到的例子都是发送一个包返回一个ack?不是累计确认？

  2020-05-08

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507943-6646.jpeg)

  aoe

  看完很激动，滑动窗口是限流神器啊

  2020-05-02

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507943-6647.jpeg)

  KK

  这一节听得的确懵啦。接触了一些未曾接触过的概念。看来越往深海，越是寒冷，哈哈哈哈

  2020-04-03

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507943-6648.jpeg)

  Larry

  流量控制的对象是发送端和接收端，目的是使得接收方处理的过来。 拥塞控制的对象是带宽，目的是让找出传输量最优。

  2020-03-24

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507944-6649.jpeg)

  King-ZJ

  TCP中的顺序、丢包和拥塞问题，都涉及到一个窗口大小的事情。文中以工作中上司安排下属任务为例来说明（一个是发送端、一个是接收端），一是上司安排任务时，为了提升效率，以ID号标识一件事情（以防乱序），二是下属领会到上司传达的任务后，要有响应，别石沉大海让领导干着急（解决丢包），三是领导通过和下属长期相处过程中，知道下属的能力和潜力，在安排任务时要量力而为，别拔苗助长。回到TCP的知识体系中，其中很多细节是需要去琢磨的，比喻是为了让读者更靠近指示器，而不是忽略知识。

  2020-03-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  这个tcp协议讲的，牛！服了！

  2020-03-22

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507944-6649.jpeg)

  King-ZJ

  看到这个TCP的包丢失和包重传，其中涉及的窗口大小还是有一些晕的，但是不会放过它的，就如何曾别人放过你一样，加油钱。多画画、多看看、多想想，就多收获。

  2020-03-21

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507944-6650.jpeg)

  Chuan

  “当三个包返回的时候，cwnd = sshthresh + 3，当三个包返回是什么意思，为什么要加3” 老师，请问下： 1. 这里的三是有什么特指吗，为什么是3不是2？ 2. 另外您回复说这三个包返回一个加一，那是不是是从10到11、12，再到13的。 3. 最后，这三个包应该就是指之前发过的包吧？还是特定的什么包？

  2020-02-19

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507944-6651.jpeg)

  奔跑的码仔

  下面是对于滑动窗口和拥塞窗口的一点个人的理解： 1.滑动窗口是关于TCP发送和接收端来说的，用于控制顺序问题、丢包问题以及流量控制等问题，这些问题也都是端-端的问题。 2.拥塞窗口是关于TCP发送和接收端之间传输链路容量的问题。

  2019-12-19

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507944-6652.jpeg)

  饭粒

  这节确实对照音频看要顺畅，好理解些。

  2019-12-01

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507944-6653.jpeg)

  雨后的夜

  这一课有点烧脑啊

  2019-11-11

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507944-6654.jpeg)

  sky

  cwnd为20时产生拥塞，是快速重传的话减半，也就是cwnd和ssthresh都是10。但为什么“等待3个确认包＋3”？cwnd到达门限后，不是应该每个确认包 + 1/cwnd么？

  2019-11-10

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507945-6655.jpeg)

  tongmin_tsai

  老师，如果发送方发了1 2 3 4 5，接收方目前接收了1 2 4 5，3丢了，那么接收方会把1 2发给应用层吗？那如果这样的话，4 5怎么办？存储在哪里？如果很多包都丢了中间的，那所有接收到的包都存储，存储不会爆掉吗？

  2019-10-02

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507945-6656.jpeg)

  Mr_Tree

  前几章看起来还有点轻松，这一张看起来有点难懂，晦涩，看着看着就想睡觉，不知道为什么，老师能给些能上手的练习吗？纯理论容易反馈

  2019-09-19

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507945-6657.png)

  火力全开

  目的端口通常我们都比较清楚，想请教一下源端口是怎么来的？

  作者回复: 随机分配

  2019-09-07

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507945-6658.jpeg)

  吴林峰 🏀

  AdvertisedWindow=MaxRcvBuffer-((NextByteExpected-1)-LastByteRead) 这个计算感觉有问题啊，不应该是：AdvertisedWindow=MaxRcvBuffer-(NextByteExpected-LastByteRead+1) 么？

  作者回复: AdvertisedWindow=MaxRcvBuffer-[(NextByteExpected-1)-LastByteRead]这个是对的。NextByteExpected - 1是第一部分和第二部分的分界线

  2019-07-24

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507945-6659.jpeg)

  杨帆

  老师，能结合最近爆出的TCP SACK漏洞详细讲解一下SACK吗

  2019-06-20

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507945-6660.jpeg)

  半清醒

  感觉老师的课，确实是要科班出身读起来要轻松许多！

  2019-05-03

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507946-6661.jpeg)

  程序员大天地

  老师这个拥塞控制用水管倒水比喻相当好，很容易理解，👍

  2019-04-17

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/132-1662307507946-6662.jpeg)

  dexter

  这章还是 tcp握手建立链接过程吧，不包括具体数据的传输。

  作者回复: 是数据的传输呀

  2019-02-25

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507946-6663.jpeg)

  媛

  接受方的缓存数据分为三部分：接受并确认，等待接受未确认，不能接受。那后面的例子，接受方8、9收到了没有ask，是啥意思，8、9属于上面哪一部分数据呢？上面三部分并没有接受未确认数据类型啊？

  作者回复: 第二部分。因为没有ack，客户端就会认为没有发成功，还会再发的。

  2019-02-23

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507946-6664.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  克里斯

  talk is cheap,show me the code.这些内容教材上也讲了，能不能看看内核代码如何实现流量控制，拥塞控制呢？

  作者回复: 内核是个大事情，不是一两句话说的清楚的。可以参考相关书籍

  2019-01-30

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507946-6665.jpeg)

  刘士涛

  请教下像tcpcopy这类请求复制工具的原理是啥，怎么处理丢包问题？

  2019-01-29

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507946-6666.jpeg)

  佳楠

  不仅学会了网络知识，还学会了工作方式，真正的触类旁通吧！

  2019-01-08

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507947-6667.jpeg)

  航仔

  第三部分的和第四部分的分界线应该是lastByteAcked + lastByteSent吧，3 + 9 = 12，如果是 lastByteAcked + AdvertisedWindow的话就是 3 + 12 = 15，如果老师说的没有问题，麻烦老师解释下

  作者回复: advertise window指向的是黑框，是9

  2018-11-01

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507947-6668.jpeg)

  太子长琴

  发送和接受的图为什么不能对应起来呢？

  2018-09-07

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507947-6669.jpeg)![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,w_14.png)

  秋天

  滑动窗口移动计算没明白

  2018-08-07

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507947-6670.jpeg)

  Roy

  真正深入浅出，这专栏物超所值

  2018-08-03

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507934-6595.jpeg)

  MoFanDon

  顺序问题和丢包问题中的例子，接收端6789，从图上看，都是等待接收未确认，是如何判断6789乱序的呢？

  2018-07-24

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6597.jpeg)

  咖啡猫口里的咖啡猫🐱

  老师能不能问个深得的问题，mqtt为什么有的嵌入TCP协议栈，，TCP的重传与mqtt  qos保证，重复了qos1或者2应用层重复，这个感觉不对劲啊，冲突啊

  2018-06-13

  **

  **

- ![img](%E7%AC%AC13%E8%AE%B2%20_%20%E5%A5%97%E6%8E%A5%E5%AD%97Socket%EF%BC%9ATalk%20is%20cheap,%20show%20me%20the%20code.resource/resize,m_fill,h_34,w_34-1662307507935-6597.jpeg)

  咖啡猫口里的咖啡猫🐱

  根据ack划掉，不是根据ttl划掉，要是极端情况，，client发送端重传包出去，这时接到server ack，清除，，，server端收到包给上层，发送ack清除buffer区，这时重发包在ttl内到达，不是会重复吗

  2018-06-13
