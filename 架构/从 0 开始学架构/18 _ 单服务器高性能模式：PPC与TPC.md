# 18 \| 单服务器高性能模式：PPC与TPC

作者: 李运华

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/ea/46/eaa4e3bb9a54638d2b873dec75cf7346.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/b3/50/b38f5619ecd3bcb6c0fe8c49d557f350.mp3" type="audio/mpeg"></audio>

高性能是每个程序员的追求，无论我们是做一个系统还是写一行代码，都希望能够达到高性能的效果，而高性能又是最复杂的一环，磁盘、操作系统、CPU、内存、缓存、网络、编程语言、架构等，每个都有可能影响系统达到高性能，一行不恰当的debug日志，就可能将服务器的性能从TPS 30000降低到8000；一个tcp\_nodelay参数，就可能将响应时间从2毫秒延长到40毫秒。因此，要做到高性能计算是一件很复杂很有挑战的事情，软件系统开发过程中的不同阶段都关系着高性能最终是否能够实现。

站在架构师的角度，当然需要特别关注高性能架构的设计。高性能架构设计主要集中在两方面：

- 尽量提升单服务器的性能，将单服务器的性能发挥到极致。

- 如果单服务器无法支撑性能，设计服务器集群方案。


<!-- -->

除了以上两点，最终系统能否实现高性能，还和具体的实现及编码相关。但架构设计是高性能的基础，如果架构设计没有做到高性能，则后面的具体实现和编码能提升的空间是有限的。形象地说，架构设计决定了系统性能的上限，实现细节决定了系统性能的下限。

<!-- [[[read_end]]] -->

单服务器高性能的关键之一就是**服务器采取的并发模型**，并发模型有如下两个关键设计点：

- 服务器如何管理连接。

- 服务器如何处理请求。


<!-- -->

以上两个设计点最终都和操作系统的I/O模型及进程模型相关。

- I/O模型：阻塞、非阻塞、同步、异步。

- 进程模型：单进程、多进程、多线程。


<!-- -->

在下面详细介绍并发模型时会用到上面这些基础的知识点，所以我建议你先检测一下对这些基础知识的掌握情况，更多内容你可以参考《UNIX网络编程》三卷本。今天，我们先来看看<span class="orange">单服务器高性能模式：PPC与TPC</span>

。

## PPC

PPC是Process Per Connection的缩写，其含义是指每次有新的连接就新建一个进程去专门处理这个连接的请求，这是传统的UNIX网络服务器所采用的模型。基本的流程图是：

![](<https://static001.geekbang.org/resource/image/53/ba/53b17d63a31c6b551d3a039a2568daba.jpg?wh=3249*2950>)

- 父进程接受连接（图中accept）。

- 父进程“fork”子进程（图中fork）。

- 子进程处理连接的读写请求（图中子进程read、业务处理、write）。

- 子进程关闭连接（图中子进程中的close）。


<!-- -->

注意，图中有一个小细节，父进程“fork”子进程后，直接调用了close，看起来好像是关闭了连接，其实只是将连接的文件描述符引用计数减一，真正的关闭连接是等子进程也调用close后，连接对应的文件描述符引用计数变为0后，操作系统才会真正关闭连接，更多细节请参考《UNIX网络编程：卷一》。

PPC模式实现简单，比较适合服务器的连接数没那么多的情况，例如数据库服务器。对于普通的业务服务器，在互联网兴起之前，由于服务器的访问量和并发量并没有那么大，这种模式其实运作得也挺好，世界上第一个web服务器CERN httpd就采用了这种模式（具体你可以参考[https://en.wikipedia.org/wiki/CERN\_httpd](<https://en.wikipedia.org/wiki/CERN_httpd>)）。互联网兴起后，服务器的并发和访问量从几十剧增到成千上万，这种模式的弊端就凸显出来了，主要体现在这几个方面：

- fork代价高：站在操作系统的角度，创建一个进程的代价是很高的，需要分配很多内核资源，需要将内存映像从父进程复制到子进程。即使现在的操作系统在复制内存映像时用到了Copy on Write（写时复制）技术，总体来说创建进程的代价还是很大的。

- 父子进程通信复杂：父进程“fork”子进程时，文件描述符可以通过内存映像复制从父进程传到子进程，但“fork”完成后，父子进程通信就比较麻烦了，需要采用IPC（Interprocess Communication）之类的进程通信方案。例如，子进程需要在close之前告诉父进程自己处理了多少个请求以支撑父进程进行全局的统计，那么子进程和父进程必须采用IPC方案来传递信息。

- 支持的并发连接数量有限：如果每个连接存活时间比较长，而且新的连接又源源不断的进来，则进程数量会越来越多，操作系统进程调度和切换的频率也越来越高，系统的压力也会越来越大。因此，一般情况下，PPC方案能处理的并发连接数量最大也就几百。


<!-- -->

## prefork

PPC模式中，当连接进来时才fork新进程来处理连接请求，由于fork进程代价高，用户访问时可能感觉比较慢，prefork模式的出现就是为了解决这个问题。

顾名思义，prefork就是提前创建进程（pre-fork）。系统在启动的时候就预先创建好进程，然后才开始接受用户的请求，当有新的连接进来的时候，就可以省去fork进程的操作，让用户访问更快、体验更好。prefork的基本示意图是：

![](<https://static001.geekbang.org/resource/image/3c/2f/3c931b04d3372ebcebe4f2c2cf59d42f.jpg?wh=3219*2430>)

prefork的实现关键就是多个子进程都accept同一个socket，当有新的连接进入时，操作系统保证只有一个进程能最后accept成功。但这里也存在一个小小的问题：“惊群”现象，就是指虽然只有一个子进程能accept成功，但所有阻塞在accept上的子进程都会被唤醒，这样就导致了不必要的进程调度和上下文切换了。幸运的是，操作系统可以解决这个问题，例如Linux 2.6版本后内核已经解决了accept惊群问题。

prefork模式和PPC一样，还是存在父子进程通信复杂、支持的并发连接数量有限的问题，因此目前实际应用也不多。Apache服务器提供了MPM prefork模式，推荐在需要可靠性或者与旧软件兼容的站点时采用这种模式，默认情况下最大支持256个并发连接。

## TPC

TPC是Thread Per Connection的缩写，其含义是指每次有新的连接就新建一个线程去专门处理这个连接的请求。与进程相比，线程更轻量级，创建线程的消耗比进程要少得多；同时多线程是共享进程内存空间的，线程通信相比进程通信更简单。因此，TPC实际上是解决或者弱化了PPC fork代价高的问题和父子进程通信复杂的问题。

TPC的基本流程是：

![](<https://static001.geekbang.org/resource/image/25/e7/25b3910c8c5fb0055e184c5c186eece7.jpg?wh=4464*3285>)

- 父进程接受连接（图中accept）。

- 父进程创建子线程（图中pthread）。

- 子线程处理连接的读写请求（图中子线程read、业务处理、write）。

- 子线程关闭连接（图中子线程中的close）。


<!-- -->

注意，和PPC相比，主进程不用“close”连接了。原因是在于子线程是共享主进程的进程空间的，连接的文件描述符并没有被复制，因此只需要一次close即可。

TPC虽然解决了fork代价高和进程通信复杂的问题，但是也引入了新的问题，具体表现在：

- 创建线程虽然比创建进程代价低，但并不是没有代价，高并发时（例如每秒上万连接）还是有性能问题。

- 无须进程间通信，但是线程间的互斥和共享又引入了复杂度，可能一不小心就导致了死锁问题。

- 多线程会出现互相影响的情况，某个线程出现异常时，可能导致整个进程退出（例如内存越界）。


<!-- -->

除了引入了新的问题，TPC还是存在CPU线程调度和切换代价的问题。因此，TPC方案本质上和PPC方案基本类似，在并发几百连接的场景下，反而更多地是采用PPC的方案，因为PPC方案不会有死锁的风险，也不会多进程互相影响，稳定性更高。

## prethread

TPC模式中，当连接进来时才创建新的线程来处理连接请求，虽然创建线程比创建进程要更加轻量级，但还是有一定的代价，而prethread模式就是为了解决这个问题。

和prefork类似，prethread模式会预先创建线程，然后才开始接受用户的请求，当有新的连接进来的时候，就可以省去创建线程的操作，让用户感觉更快、体验更好。

由于多线程之间数据共享和通信比较方便，因此实际上prethread的实现方式相比prefork要灵活一些，常见的实现方式有下面几种：

- 主进程accept，然后将连接交给某个线程处理。

- 子线程都尝试去accept，最终只有一个线程accept成功，方案的基本示意图如下：


<!-- -->

![](<https://static001.geekbang.org/resource/image/11/4d/115308f686fe0bb1c93ec4b1728eda4d.jpg?wh=4527*3150>)

Apache服务器的MPM worker模式本质上就是一种prethread方案，但稍微做了改进。Apache服务器会首先创建多个进程，每个进程里面再创建多个线程，这样做主要是为了考虑稳定性，即：即使某个子进程里面的某个线程异常导致整个子进程退出，还会有其他子进程继续提供服务，不会导致整个服务器全部挂掉。

prethread理论上可以比prefork支持更多的并发连接，Apache服务器MPM worker模式默认支持16 × 25 = 400 个并发处理线程。

## 小结

今天我为你讲了传统的单服务器高性能模式PPC与TPC，希望对你有所帮助。

这就是今天的全部内容，留一道思考题给你吧，什么样的系统比较适合本期所讲的高性能模式？原因是什么？

欢迎你把答案写到留言区，和我一起讨论。相信经过深度思考的回答，也会让你对知识的理解更加深刻。（编辑乱入：精彩的留言有机会获得丰厚福利哦！）

## 精选留言(83)

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34.jpeg)

  鹅米豆发

  ​       不同并发模式的选择，还要考察三个指标，分别是响应时间（RT），并发数（Concurrency），吞吐量（TPS）。三者关系，吞吐量=并发数/平均响应时间。不同类型的系统，对这三个指标的要求不一样。       三高系统，比如秒杀、即时通信，不能使用。       三低系统，比如ToB系统，运营类、管理类系统，一般可以使用。       高吞吐系统，如果是内存计算为主的，一般可以使用，如果是网络IO为主的，一般不能使用。

  作者回复: 赞，分析到位

  2018-06-07

  **4

  **179

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653134.jpeg)

  Regular

  我怎么觉得，凡是高并发请求的系统都适合本节讲的高性能模式？！

  作者回复: 高并发需要根据两个条件划分：连接数量，请求数量。 1. 海量连接（成千上万）海量请求：例如抢购，双十一等 2. 常量连接（几十上百）海量请求：例如中间件 3. 海量连接常量请求：例如门户网站 4. 常量连接常量请求：例如内部运营系统，管理系统 你再尝试分析一下看看😃

  2018-06-07

  **6

  **95

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653135.jpeg)

  W_T

  老师在文章和留言里已经透露答案了。 首先，PPC和TPC能够支持的最大连接数差不多，都是几百个，所以我觉得他们适用的场景也是差不多的。 接着再从连接数和请求数来划分，这两种方式明显不支持高连接数的场景，所以答案就是： 1. 常量连接海量请求。比如数据库，redis，kafka等等 2. 常量连接常量请求。比如企业内部网址

  作者回复: 回答正确😃😃

  2018-06-08

  **2

  **81

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653136.jpeg)

  JackLei

  看到这篇文章，这个专栏的价值远远远远远远远远远远远远远远远远大于专栏购买的价格。

  作者回复: 这篇这么值呀😄其实我觉得前面的更值，架构本质，架构设计目的，架构设计原则，架构设计流程……都是就此一家，别无分店😄

  2018-06-13

  **

  **37

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653137.jpeg)

  正是那朵玫瑰

  根据华仔回复留言的提示，分析下 1. 海量连接（成千上万）海量请求：例如抢购，双十一等 这样的系统，我觉得这讲所说的模式都不适应，面对海量的连接至少要使用IO复用模型或者异步IO模型，针对海量的请求，无论使用多进程处理还是多线程，单机都是无法支撑的，应该集群了吧。 2. 常量连接（几十上百）海量请求：例如中间件 常量连接，我觉得这讲的模式应该可以适用，如使用TPC的preyhtead模型，启动几十上百的线程去处理连接，应该问题不大吧，但是老师举的列子是中间件是这类系统，我就有点疑问了，是不是中间件系统都可以是阻塞IO模型来实现，比如activemq既支持BIO也支持NIO，但是NIO只是解决了能处理更多的连接，而真正每个请求的处理快慢还得看后面的业务的处理；而阿里的rocketmq也是使用netty这样的NIO框架实现的。但在面对常量连接的场景下，NIO并没有优势啊。 3. 海量连接常量请求：例如门户网站 这种系统我觉得非常适合使用netty这样的NIO框架来实现，IO复用模型可以处理海量的连接，而每个连接的请求数据量会很小，处理会很长快，如华仔说的门户网站，只要简单返回页面即可。 4. 常量连接常量请求：例如内部运营系统，管理系统 这种系统，本讲的模式就很适合了。 水平有限，望华仔指点下。

  作者回复: 写的很好，你的疑问补充如下： 1. 常量连接模式下NIO除了复杂一点外，也没有缺点，因此也可以用。 2. 海量连接情况下，单机也要支持很多连接，不然集群成本太高

  2018-06-08

  **3

  **35

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653138.jpeg)

  peison

  请教一个比较小白的问题…为什么说门户网站是海量连接常量请求的情况？海量连接下为什么会常量请求，一直想不通

  作者回复: 海量连接：连接的用户很多 常量请求：每个用户请求数量不多，大部分都是看完一篇文章再去点击另外的文章

  2018-07-24

  **5

  **27

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663139.jpeg)

  孙晓明

  李老师，看完文章后查了一下bio和nio，还有一种aio，看的不是太明白，能麻烦您解答一下，并且它与nio的差别在哪里？

  作者回复: bio：阻塞io，PPC和TPC属于这种 NIO：多路复用io，reactor就是基于这种技术 aio：异步io，Proactor就是基于这种技术

  2018-06-22

  **4

  **17

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663140.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  公号-技术夜未眠![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  BIO：一个线程处理一个请求。缺点：并发量高时，线程数较多，浪费资源。Tomcat7或以下，在Linux系统中默认使用这种方式。可以适用于小到中规模的客户端并发数场景，无法胜任大规模并发业务。如果编程控制不善，可能造成系统资源耗尽。 NIO：利用多路复用IO技术，可以通过少量的线程处理大量的请求。Tomcat8在Linux系统中默认使用这种方式。Tomcat7必须修改Connector配置来启动。 NIO最适用于“高并发”的业务场景，所谓高并发一般是指1ms内至少同时有成百上千个连接请求准备就绪，其他情况下NIO技术发挥不出它明显的优势。

  2018-06-07

  **

  **17

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663141.jpeg)

  小文同学

  PPC和TPC对那些吞吐量比较大，长连接且连接数不多的系统应该比较适用。两种模式的特点都比较重，每个连接都能占有较多计算资源，一些内部系统，如日志系统用于实时监控的估计可以采用。这类型的系统一般连接数不多，吞吐量比较大，不求服务数量，求服务质量。不知道这样的假设符不符合？

  作者回复: 回答正确

  2018-06-07

  **

  **14

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663142.jpeg)

  LakNeumann

  大神， ……纯新手感觉这里读起来已经有点吃力了 ～  好难啊

  作者回复: 这是操作系统基础，可以看看《UNIX网络编程 卷一》

  2018-06-23

  **

  **13

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663143.jpeg)

  胖胖的程序猿

  \1. 海量连接（成千上万）海量请求：例如抢购，双十一等 2. 常量连接（几十上百）海量请求：例如中间件 3. 海量连接常量请求：例如门户网站 4. 常量连接常量请求：例如内部运营系统，管理系统 这个不理解，连接和请求有什么区别

  作者回复: 一个连接就是TCP连接，一个连接每秒可以发一个请求，也可以发几千个请求

  2019-04-01

  **4

  **12

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663144.png)

  无聊夫斯基

  我无法想到ppc比tpc更适合的场景

  作者回复: tpc异常时整个服务器就挂了，而ppc不会，所以ppc适合数据库，中间件这类

  2018-08-22

  **5

  **11

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663145.jpeg)

  云学

  希望再多写几篇讲解单机性能优化，比如线程模型，数据库，网络等，猜测下一篇讲IO复用了吧

  作者回复: 具体的技术细节点好多，专栏聚焦架构。 一些常见的细节点如下： java：推荐看disruptor的设计论文，包括false sharing, 并发无锁，ring buffer等； 网络：tcp_nodelay，NIO； 内存：内存池，对象池，数据结构 存储：磁盘尾部追加，LSM；

  2018-06-07

  **

  **11

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663146.jpeg)

  孙振超

  本章中提到的几个概念，比如阻塞、非阻塞、同步、异步以及主要的两种方式ppc和tpc，以前都是记住了，今天通过这篇文章是理解了而不再是记住了。 ppc和tpc都是有一个进程来处理链接，收到一个请求就新创建一个进程或线程来处理，在处理完成之前调用方是处于阻塞状态，这种机制决定了单机容量不会太大。 但在文章中提到数据库一般是ppc模式，但数据库通常是链接数少请求量大的场景，为什么会采用这种模式呢？reactor模式为什么不适合数据库？还望老师解惑，多谢！

  作者回复: 数据库链接数少请求量大，所以单线程或者单进程io轮询性能也高，因为一直都有数据处理，不会浪费时间阻塞等待，但数据库的引擎可以是多线程或者多进程，也就是说一条链接的请求交给引擎后，引擎可以是多线程来处理。 reactor适应于连接数很大但活动连接并没有那么多的场景，例如互联网web服务器，reactor功能上也可以用于数据库，只是关系数据库都是历史悠久，久经考验，没有必要把原来的模式改为reactor

  2018-07-09

  **

  **10

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663147.jpeg)

  james

  互联网高并发的系统全部适应啊，现在C10K已经不是问题了，有些可优化度较高的系统（例如读多写少的系统）C100k的性能都很常见了

  作者回复: 互联网用ppc和tpc的不多呢，尤其是海量连接的场景

  2018-06-07

  **

  **9

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748663148.jpeg)

  海军上校

  如何系统的学习liunx～ 包括网络～文件系统～内存管理～进程管理～然后串起来形成面呢😂

  作者回复: 系统学习最好的方式就是看书，推荐《Unix环境高级编程》《Linux系统编程》

  2018-08-08

  **2

  **8

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/132.png)

  Hanson

  如果针对自内部系统，是否使用长链接性能损耗较低，毕竟频繁建链拆链性能损耗还是不小的，尤其是TLS的情况下

  作者回复: 内部系统长连接多，例如各种中间件，数据库

  2018-11-24

  **

  **7

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673149.jpeg)

  盘尼西林

  怎么理解常量连接 海量请求。 如果请求多的话，连接还会是常量么？

  作者回复: 常量连接海量请求：100个连接，每个连接每秒发10000个请求，例如数据库 海量连接常量请求：100000个连接，每个连接每秒发1个请求，例如nginx

  2020-03-24

  **

  **5

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673150.jpeg)

  文竹

  PPC适合对稳定性要求高，但并发量不大的场景，对于互联网的场景不适合。 TPC支持的并发量大，适合互联网产品场景。对于支持稳定性，需要创建冗余进程。

  作者回复: TPC支持的并发量也不大呢

  2018-08-19

  **

  **5

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673151.jpeg)

  星火燎原

  像您说的这种多进程多线程模式好似更加稳定，但是tomcat为什么采用单进程多线程模型呢？

  作者回复: tomcat是java语言开发的，java用线程是最好的

  2018-06-07

  **

  **5

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673152.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  DFighting

  看大家的评论，对aio、bio和nio讨论的比较多，我也来凑个数： 请求到达服务器到最终处理完毕可以分为3个过程，接受请求+系统处理+结果返回，其中需要切换线程写作的是请求接受后，在系统处理这段时间，连接线程如何做： bio:阻塞调用等待系统处理完毕 nio:不阻塞调用，不断查询结果，其实从结果上来看依然是阻塞等待的一种，只不过是主动点 io复用：一个连接同时处理多个io请求，本质上还是阻塞，不过由多个线程同时阻塞等待转为一个线程阻塞，等待多个io调用结果 aio:连接请求和处理请求完全异步，连接交给系统处理以后，我就可以去处理下一个连接，等待系统处理结果完毕以后，发送消息或者信号异步发送结果返回 其实从本质以上来看，阻塞和非阻塞这两种性能差异感觉不大，多路复用虽然也是阻塞但是它复用了阻塞线程，提高了性能。异步io才是真正实现了连接线程和系统处理线程的并发执行。 以上是我自己的了解，如有不正确的地方，欢迎指正和探讨

  作者回复: 异步io性能相比io复用没有明显优势，但操作系统设计比较复杂，目前来看：Linux上主流的是epoll io复用，windows上是IOCP的异步io

  2021-03-08

  **2

  **4

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673153.jpeg)

  咬尾巴的蛇

  很感谢分享，我想问下 就是我开一个tomcat，支持tcp请求，然后什么都不做处理，同时有几百个请求是后面连接就进不来了吗，我是新手才开始架构这块，希望能帮忙解释下，非常感谢

  作者回复: tomcat应该不是ppc的模式，如果你找一个ppc的服务器，连接满了后新的连接进不来，但旧的连接发送请求是没问题的

  2018-08-15

  **2

  **4

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/132-16617957748673154.jpeg)

  大光头

  高吞吐和高稳定是互斥的，如果要高吞吐就要采用prethread模式，如果要高稳定就需要高preprocess，如果要综合，线程进程同时

  作者回复: 这就是apache的其中一个模式，线程进程结合

  2018-06-09

  **

  **4

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673155.jpeg)

  琴扬枫

  老师好，请教个问题，文中提到的几百连接是在什么样的软硬件环境参数下面呢？谢谢

  作者回复: 常见的服务器配置就可以，16核16g以上

  2018-06-07

  **

  **4

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673156.jpeg)

  钱

  课前思考及问题 1：啥是PPC？它怎么提高性能的？ 2：啥是TPC？它怎么提高性能的？ 3：还存在XPC吗？ 课后思考及问题 1：今天讲的知识，感觉是自己的知识盲点，不是很了解。 2：文中核心观点 2-1：高性能架构设计主要集中在两方面： 一是，尽量提升单服务器的性能，将单服务器的性能发挥到极致。 二是，如果单服务器无法支撑性能，设计服务器集群方案。 2-2：架构设计决定了系统性能的上限，实现细节决定了系统性能的下限。 2-3：服务器高性能的关键之一就是服务器采取的并发模型，并发模型有如下两个关键设计点： 服务器如何管理连接，和I/O模型相关，I/O 模型：阻塞、非阻塞、同步、异步。 服务器如何处理请求，和进程模型相关，进程模型：单进程、多进程、多线程。 详情需参考《UNIX网络编程》《NUIX环境高级编程》《LINUX系统编程》 3：PPC 是 Process Per Connection 的缩写，其含义是指每次有新的连接就新建一个进程去专门处理这个连接的请求，这是传统的 UNIX 网络服务器所采用的模型。 这里没太明白老师讲的是网络通信的通用I/O模型，还是指UNIX特有的，感觉没讲全，也没讲出它怎么实现单服务器的高性能的？ 4：TPC 是 Thread Per Connection 的缩写，其含义是指每次有新的连接就新建一个线程去专门处理这个连接的请求。与进程相比，线程更轻量级，创建线程的消耗比进程要少得多；同时多线程是共享进程内存空间的，线程通信相比进程通信更简单。 5：这块知识差的太多了，感觉听完老师的讲解，发现自己对于进程、线程、子进程、子线程、怎么阻塞的、怎么唤醒的、怎么分配内存空间的、怎么被CPU执行的、网络链接怎么建立的、进程怎么切换的、线程怎么切换的、一个请求怎么发送怎么处理的怎么返回的、长链接怎么保持的这些东西的具体细节都不太清楚。惭愧啊！知耻而后勇，我一定把这些补上来！

  作者回复: 加油

  2019-08-28

  **

  **3

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673157.jpeg)

  互联网老辛

  单台数据库服务器可以支撑多少并发？单台nginx 可以支撑多少呢

  作者回复: nginx反向代理可以达到3万以上，具体需要根据业务和场景测试

  2018-06-09

  **2

  **3

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673158.jpeg)

  熊猫酒仙

  单方close和双方close应该是全双工与半双工的区别，连接仍然还存在的。 ppc/tpc比较适合系统中担当路由器或容器角色的部分，像nginx，甚至像mongodb集群也有router。 感觉一些部署容器的线程池，应该属于tpc范畴！

  作者回复: close是减少文件描述符引用计数，当socket的文件描述符引用计数为0时，操作系统底层会执行连接关闭操作，全双工和半双工是shutdown操作。 路由器恰恰不适合ppc/tpc，nginx也不是这种模式

  2018-06-08

  **2

  **3

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673159.jpeg)

  ranger2

  游戏服务器还是比较注重于性能，一般都会特意挑选一个适合的线程模型：既要保证对单用户请求的顺序处理，还要保证内存可见性。 在游戏服务器中，如果一个线程只用于处理一个连接的所有请求，那就是太浪费资源了

  2018-06-07

  **

  **3

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673160.jpeg)

  feifei

  PPC和TPC,这2种模式，无论进程还是线程都受cpu的限制，进程和线程的切换都是有代价的，所以像小型的系统合适，所以很多传统行业选择的是这2种

  作者回复: 是的，连接数不多

  2018-06-07

  **

  **3

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673161.png)

  Liang

  进程和线程就像公司和部门。 PPC需要进程之间通信，效率会比较低。 TPC进程调用线程就像公司指挥自己的部门，资源是共享的，效率比进程之间的通信要高。但是有死锁风险，就像公司不同部门之间争夺资源有时会导致某些项目无法推进。

  作者回复: 这个比喻很形象 ：）

  2021-03-26

  **2

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673162.jpeg)

  undefined

  请教下，PPC和NIO在同一台服务器上是否可以混合使用？redis用单线程处理IO请求时使用多路复用技术,日志持久化时某些操作会fork子进程来实现异步。关于redis单机高性能的设计跟今天的课程接合起来，是否可以简述一下?

  作者回复: PPC和NIO不会混用，PPC是指Process Per Connection，不是说用了多进程就是PPC，而是说用进程来处理连接才是PPC。 所以redis用fork来创建持久化进程不是PPC。

  2020-12-30

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683163.jpeg)

  落叶飞逝的恋

  几百连接的这种性能只适合内部管理系统之类。

  作者回复: 中间件也可以，例如数据库，缓存系统等

  2020-10-16

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683164.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  escray![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/d57fddfa5b2c2ce3071e92304da8af62.png)

  之前没有接触过相关的内容，比较偏底层，虽然能看懂但是没有经验也没有太多想法，感觉 PPC 和 TPC 都是在想尽办法去榨取单机的性能，我觉的这篇文章最有价值的部分在于留言和老师的回复。 复述一下 @正是那朵玫瑰 同学的回答 海量连接海量请求，例如抢购、双十一、秒杀，不适合单机高可用，应考虑集群 常量连接海量请求，例如中间件，这个似乎可以 海量连接常量请求，例如门户网站，这个不合适 常量连接常量请求，例如内部运营系统，管理系统，这个可以 还有就是在后面提到 PPC 和 TPC 实际上都属于 BIO（阻塞IO），而专栏后面要讲到的 Reactor 属于（NIO），Proactor 属于（AIO）。 以前对于 Java IO 这里并不是特别的清晰，看到这里感觉理解更深入了一些，知识点关联起来了。 昨天看到老师集中回复了我的一些留言，感谢华仔，特别是老师在专栏已经结束的情况下，仍然能够回复，对我来说算是意外的惊喜。准备接着参加架构师训练营的机会，把专栏重读一遍（第一次学习的时候基本没怎么留言）。

  作者回复: 别这样说啊，文章也是有价值的好吧������ 架构师训练营是李智慧老师的

  2020-09-14

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683165.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  技术修行者![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  在考虑高性能架构时，主要集中在两方面： 1. 尽量提升单服务器的性能，将单服务器的性能发挥到极致。 2. 如果单服务器无法支撑性能，设计服务器集群方案。 另外，编码实现也会影响性能结果，架构设计决定了系统性能的上限，实现细节决定了系统性能的下限。 对于单服务器来说，我们一般采用并发方式来提高性能，需要考虑： 1. 服务器如何管理连接。 2. 服务器如何处理请求。 常见的并发方式有： 1. PPC， Process Per Conection，每个连接都使用一个独立的进程来处理。优点是稳定，当个请求崩溃并不会引起服务器崩溃，缺点是1) 创建进程成本大，2)进程间通信成本大。 2. TPC，Thread Per Connection，每个请求使用一个独立的线程进行处理。优点是性能要比PPC方式好，缺点是1) 线程间资源竞争可能会引发死锁，需要考虑线程同步，2) 单个线程崩溃可能会引起整个服务器进程崩溃。 从IO角度来看，PPC和TPC都属于BIO模式，在大规模互联网应用中不常见。 从连接和请求的角度来看，PPC和TPC适合常量连接的场景，对于请求来说，常量请求和海量请求都可支持。

  2020-01-05

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_88604f

  TPC中的说法:和 PPC 相比，主进程不用“close”连接了。原因是在于子线程是共享主进程的进程空间的，连接的文件描述符并没有被复制，因此只需要一次 close 即可。这里的意思是每个子线程自己创建连接？

  作者回复: 只有一个线程会创建连接，就是accept所在的主线程

  2019-08-20

  **2

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683166.jpeg)

  xiao皮孩。。

  Chrome浏览器多进程

  2019-03-29

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683167.jpeg)

  馒头

  理论上可以理解，如果有实际场景分析配合一起讲解就完美啦

  作者回复: 可以看看apache服务器不同运行模式

  2018-11-28

  **2

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683168.jpeg)

  ZH

  老师说的一行不恰当的 debug 日志，就可能将服务器的性能从 TPS 30000 降低到 8000； 记日志不应该用同步的方式记录数据库或者文件，而是应该发消息的方式吗？

  作者回复: 不是的，日志还是同步记录简单些，这个例子是说单服务器高性能细节很多

  2018-09-24

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683169.jpeg)

  blacknccccc

  ppc和tpc模式一般如何根据硬件配置计算可支持的并发数呢

  作者回复: 一般按照CPU核数*2来计算

  2018-07-23

  **2

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683170.jpeg)

  Geek_59a17f

  架构设计决定了系统性能的上限，实现细节决定了系统性能的下限

  2018-06-14

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683171.jpeg)

  衣申人

  有个疑问: 看ppc和tpc模式，最后都是服务器write之后就close，意思是这两种模式都不支持长连接？

  作者回复: 都支持长连接，循环read-处理-write，图中write后可以继续read

  2018-06-10

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683172.jpeg)

  东

  著名的邮件服务器postfix采用的是多进程模式，应该是为了稳定性，单个进程崩溃对整个系统影响很小。更难能可贵的是性能也非常好，当然这个得益于全异步模式，让客户端感觉响应很快，后面放到queue里，由另外一组进程来响应。

  作者回复: 我觉得postfix应该是c/c++写的😃写c/c++的喜欢用多进城，java用多线程

  2018-06-09

  **

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683173.jpeg)

  Hook

  老师，我使用 Netty 开发了一个 Java 网络应用程序，是一个长连接的场景。我使用 CentOS、14G 内存、2 个CPU，经过测试发现最高能支持 3 万个并发长连接。 在这 3 万个长连接场景下，CPU利用率大概在 65%左右，而内存剩余不到 2 个G。 这个结果，与网上很多文章介绍的，轻轻松松就能支持百万长连接例子相差很大啊。 结合老师的经验，我这个结果能打几分呢？能给一些优化的方向吗？

  作者回复: 轻松百万是假的吧？这个结果已经是正常结果了

  2018-06-07

  **2

  **2

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683174.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  katejc

  老师，我看了一些留言，有疑问：一个请求不就会 进行一次连接吗，那么海量的请求不就是 会带来海量的连接吗；

  作者回复: 连接分为：长连接 和 短连接，长连接可以一个连接发送大量的请求。

  2022-05-29

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683175.jpeg)

  达叔灬

  prethread  感觉有点像线程池。

  作者回复: 就是线程池

  2021-11-26

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748683176.jpeg)

  谭方敏

  现在传统业务架构中并发不止是几百了，所以纯粹的ppc，tpc没法适用了，不过在一些部分场景下可以使用，比如中间件和数据库访问上。基于tpc仍旧开销大的情况，要有海量连接，一般会采用线程池外加连接池的模式来达到优化的效果，每个新加入的连接不再是独占一个线程。他们会共享固定数量的线程。这也是目前比较经典的reactor，随着对减少死锁和临界资源呼声越来越大，又出现了actor模型。 目前erlang就是继续这种模式，java，go对这种模式都支持的非常好。

  2020-03-03

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693177.jpeg)

  shinee_x_X

  想问下老师，Java的lwp进程是这里所说的线程再进一步创建的进程吗

  作者回复: 一般说Linux的LWP进程，JAVA没有LWP进程的概念，JAVA在LINUX上的线程其实是LWP进程

  2019-10-17

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693178.jpeg)

  天外来客

  主要考察连接数，因单机的fd 是有限的，可根据并发连接数考虑是否部署集群！

  作者回复: fd不是关键，关键是进程或者线程数量太多，所占资源和上下文切换很耗费性能

  2018-06-20

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693179.jpeg)

  贾智文

  老师想问两个问题。 您说互联网模式下不适合ppc和tpc，那么互联网模式下一版用什么呢？ 还有前面留言说海量链接可以用nio的多路复用来减少连接的需求数量，我理解多路复用是点对点之间而言的，门户网站这种海量是点很多，点对点之间连接不多，这种多路复用也没有太大效果吧？

  作者回复: 互联网用nio，reactor，门户网站单条连接也是点对点

  2018-06-14

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653136.jpeg)

  JackLei

  看到了这一篇，文章的价值远远远远远远远远远远大于这个专栏的购买价格

  2018-06-13

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693180.jpeg)

  辉

  Prethread为啥不用线程池

  作者回复: 具体实现可以做成线程池，但线程池涉及到不同连接复用线程时的资源共享问题，处理会复杂一些；如果一个线程只给一个连接用，连接关闭就结束线程，新连接用新线程，实现起来很简单

  2018-06-12

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693181.jpeg)

  肖一林

  对于客户端不多的情况下，可以长连接。对于客户端海量的情况下，只能短连接，还要控制瞬时连接数？反正都要适应ppc和tpc，对吧

  作者回复: Reactor，Proactor都可以处理海量长连接

  2018-06-11

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693182.jpeg)

  ◎⌒◎乖虸※

  计算密集型系统应该会比较适合

  2018-06-07

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693183.jpeg)

  空档滑行

  PPC我觉得适合连接数相对固定，长链接，比如系统内服务调用，或者消息推送。TPC适合频繁的创建和销毁链接，比如web服务。不知道这么理解对不对

  作者回复: PPC的基本正确，TPC不对，web服务一般用后面讲的reactor模式

  2018-06-07

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693184.jpeg)

  ghosttwo

  大佬可以分析下线程池模型和单线程异步模型在高并发下的优势和劣势吗？感觉现在很流行这种异步的请求方式，比如reactor和spring5的webflux

  作者回复: reactor会在后面讲

  2018-06-07

  **

  **1

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693185.jpeg)

  Allan![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/d57fddfa5b2c2ce3071e92304da8af62.png)

  目前跑大数据的离线任务是不是适合这个tpc的模式，一下子会开启好多的线程然后提高任务执行的速度。而且每次多个人提交多个离线任务，每个离线任务都是一个进程也就是ppc模式。不知道我说的是否正确。

  作者回复: java都是多线程，例如spark这种平台，调度的都是线程

  2022-08-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  Geek_039a5c

  我想问Netty 算是哪种？ preThread 模式吗？

  作者回复: Netty是多路复用，Reactor模式，后面会讲

  2022-03-02

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693186.jpeg)

  易飞

  常量链接的系统把，海量链接数会造成系统压力太大

  作者回复: 正解

  2022-02-03

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693187.jpeg)

  siwei

  同一页面发起多次ajax请求是一个连接还是多个连接？

  作者回复: ajax发的是HTTP请求，所以你要看你具体的HTTP协议版本是1.0还是1.1还是HTTP/2

  2021-11-04

  **3

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  石燕

  这种模式应该适用于连接数不大（差不多几百，多进程+多线程，最多几千的情况），想知道怎么实现海量连接和海量请求

  作者回复: 继续学习后面的内容

  2021-09-19

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693188.jpeg)

  Erebus![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/8ac15yy848f71919ef784a1c9065e495.png)

  老师讲的很棒，评论区的伙伴们也总结、拓展的很棒。

  作者回复: 评论区卧虎藏龙

  2021-03-30

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693189.jpeg)

  张小方

  TPC模式那里是不是写错了？既然是同一个进程，图的左边是不是主线程，右边是子线程？还是说父进程里面有个一个线程（主线程接受连接），另外fork出子进程，子进程中有子线程？感觉应该是前者，即一个进程下，主线程负责接受连接，然后将连接交给同一个进程的其他线程（子线程）。

  作者回复: 主进程创建子线程，这个说法没错哦，创建线程不需要先fork子进程，再来pthread_create线程

  2021-03-24

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693190.jpeg)

  艺超(鲁鸣)

  再看一遍，有个疑问，tpc右侧是不应该是线程吗？看图上是进程呢

  作者回复: 感谢，是图中文字错误，确实是子线程，我让编辑更新一下。

  2021-03-15

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673152.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  DFighting

  这种架构的问题在于连接数不能多，上限也就几百个，但是这不代表就不是高并发了，因为一次连接可以处理多次请求或者连接池这种复用连接。也就是这个架构适用情况是不需要频繁多次建立连接的情况。比如存储，缓存。换句话来说，如果用这种架构不维持那种长连接或者复用，一次连接处理一个请求，那这种性能就算不上高了吧

  作者回复: PPC和TPC适合常量连接，海量请求，每个请求都要创建一次连接的话，性能高不了。

  2021-03-08

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693191.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  piboye

  这两种性能都不高啊，协程方式

  2020-08-26

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748693192.jpeg)

  牺牲

  感觉TPC跟PPC相比，浑身优点。那为什么我们不直接选择更灵活、更快的TPC？还要做两个选项呢

  作者回复: 文中已经写了TPC的几个缺点，这几个缺点在某些场景下影响比较大

  2020-05-22

  **2

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_88604f

  prethread 理论上可以比 prefork 支持更多的并发连接，这里有几个方面的原因呢？

  作者回复: 文中有详细讲呀

  2019-08-20

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703193.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  无名之辈

  想问老师，您认为想成为一个合格的架构师，需要有什么潜质呢

  作者回复: 后面有谈架构师成长的

  2019-01-25

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/132-16617957748703194.jpeg)

  gkb111

  海量连接海量请求，可以使用多进程多线程模型ppc+tpc

  作者回复: PPC不适合海量连接

  2019-01-25

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703195.jpeg)

  日光倾城

  能支持的最大并发量都不高，后台管理系统？

  2018-10-07

  **1

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748673150.jpeg)

  文竹

  对于PPC或TPC，由于accept在单一进程中，故支持的QPS有限。另外两种，由于accept和IO操作在同一个进程或线程中，支持的QPS也有限。

  2018-08-09

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703196.jpeg)

  Geek_b04b12

  像Php这种编程语言，我觉得这种高性能模式很实用，特别是在那种了电商，之类的，包括秒杀系统，

  作者回复: 好像nginx的php-fpm是线程池，prethread模式

  2018-07-13

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703197.jpeg)

  追寻云的痕迹

  gunicorn 用了pre-fork http://docs.gunicorn.org/en/stable/design.html

  作者回复: 学习了👍

  2018-07-02

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703198.jpeg)

  Mk

  老师能都讲一下对于业务架构人员 DDD TDD 淘宝业务架构是如何选型 

  2018-06-26

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748653134.jpeg)

  Regular

  谢谢老师的回复。 本文的模式适合的是海量请求的系统，因为得创建新的进程线程。不知道这样理解对不对[捂脸]

  作者回复: 可以，数据库就可以用

  2018-06-08

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703199.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  小龙

  老师，架构是在软件开发的概要设计阶段吗？

  作者回复: 不是，架构设计在概要设计之前，先有架构设计，后有方案设计

  2018-06-08

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703200.jpeg)

  小江

  常量连接常量请求适合，常量连接海量请求（基于内存快速响应）也适合。

  2018-06-08

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703201.jpeg)

  靈域

  tpc应该就是常说的bio模式吧？每个请求进来就创建一个线程，用完就销毁！prethread感觉就是线程池，先创建好一定数量的线程，然后用这些线程去处理请求，用完了在放回池子！

  2018-06-07

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703202.jpeg)![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,w_14.png)

  SMTCode

  多核服务器。

  2018-06-07

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703203.jpeg)

  何磊

  专栏看到今天，我想请教老师另外一个问题：什么样的服务器才算高性能呢？多少qps？多少tps？ 日常工作中我们如何去对自己的系统进行高性能评估？就用ab测试查看结果吗？

  作者回复: 不能简单以一个数字来衡量，和具体的业务相关，redis能够达到单机tps 5万，普通业务服务器不可能这么高，几千已经很高了

  2018-06-07

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703204.jpeg)

  卡莫拉内西

  20年前的系统可能适合，哈哈，有些无厘头了，从最大并发支持来看，也就适合一些初创公司的网站吧😄

  2018-06-07

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/resize,m_fill,h_34,w_34-16617957748703204.jpeg)

  卡莫拉内西

  20年前的系统比较适合，哈哈，这样回答是不是有些无厘头

  2018-06-07

  **

  **

- ![img](18%20_%20%E5%8D%95%E6%9C%8D%E5%8A%A1%E5%99%A8%E9%AB%98%E6%80%A7%E8%83%BD%E6%A8%A1%E5%BC%8F%EF%BC%9APPC%E4%B8%8ETPC.resource/132-16617957748703205.png)

  性能

  应用服务器好像使用prethread更多一些，因为服务器内部多线程之间经常需要共享数据，比如连接池资源，参数等。prefork多子进程还得复制多份内存，多份内存数据同步也比较麻烦

  作者回复: 看怎么设计了，不过总体来说大部分技术同学更熟悉线程

  2018-06-07

  **

  **
