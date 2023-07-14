# 第10讲 \| UDP协议：因性善而简单，难免碰到“城会玩”

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/88/da/88ba89d6830218e0fa2489c23076bcda.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/60/83/6046c29969d44eeb559bf99d0a366983.mp3" type="audio/mpeg"></audio>

讲完了IP层以后，接下来我们开始讲传输层。传输层里比较重要的两个协议，一个是TCP，一个是UDP。对于不从事底层开发的人员来讲，或者对于开发应用的人来讲，最常用的就是这两个协议。由于面试的时候，这两个协议经常会被放在一起问，因而我在讲的时候，也会结合着来讲。

## TCP和UDP有哪些区别？

一般面试的时候我问这两个协议的区别，大部分人会回答，TCP是面向连接的，UDP是面向无连接的。

什么叫面向连接，什么叫无连接呢？在互通之前，面向连接的协议会先建立连接。例如，TCP会三次握手，而UDP不会。为什么要建立连接呢？你TCP三次握手，我UDP也可以发三个包玩玩，有什么区别吗？

**所谓的建立连接，是为了在客户端和服务端维护连接，而建立一定的数据结构来维护双方交互的状态，用这样的数据结构来保证所谓的面向连接的特性。**

例如，**TCP提供可靠交付**。通过TCP连接传输的数据，无差错、不丢失、不重复、并且按序到达。我们都知道IP包是没有任何可靠性保证的，一旦发出去，就像西天取经，走丢了、被妖怪吃了，都只能随它去。但是TCP号称能做到那个连接维护的程序做的事情，这个下两节我会详细描述。而**UDP继承了IP包的特性，不保证不丢失，不保证按顺序到达。**

再如，**TCP是面向字节流的**。发送的时候发的是一个流，没头没尾。IP包可不是一个流，而是一个个的IP包。之所以变成了流，这也是TCP自己的状态维护做的事情。而**UDP继承了IP的特性，基于数据报的，一个一个地发，一个一个地收。**

还有**TCP是可以有拥塞控制的**。它意识到包丢弃了或者网络的环境不好了，就会根据情况调整自己的行为，看看是不是发快了，要不要发慢点。**UDP就不会，应用让我发，我就发，管它洪水滔天。**

因而**TCP其实是一个有状态服务**，通俗地讲就是有脑子的，里面精确地记着发送了没有，接收到没有，发送到哪个了，应该接收哪个了，错一点儿都不行。而**UDP则是无状态服务**。通俗地说是没脑子的，天真无邪的，发出去就发出去了。

我们可以这样比喻，如果MAC层定义了本地局域网的传输行为，IP层定义了整个网络端到端的传输行为，这两层基本定义了这样的基因：网络传输是以包为单位的，二层叫帧，网络层叫包，传输层叫段。我们笼统地称为包。包单独传输，自行选路，在不同的设备封装解封装，不保证到达。基于这个基因，生下来的孩子UDP完全继承了这些特性，几乎没有自己的思想。

## UDP包头是什么样的？

我们来看一下UDP包头。

前面章节我已经讲过包的传输过程，这里不再赘述。当我发送的UDP包到达目标机器后，发现MAC地址匹配，于是就取下来，将剩下的包传给处理IP层的代码。把IP头取下来，发现目标IP匹配，接下来呢？这里面的数据包是给谁呢？

发送的时候，我知道我发的是一个UDP的包，收到的那台机器咋知道的呢？所以在IP头里面有个8位协议，这里会存放，数据里面到底是TCP还是UDP，当然这里是UDP。于是，如果我们知道UDP头的格式，就能从数据里面，将它解析出来。解析出来以后呢？数据给谁处理呢？

处理完传输层的事情，内核的事情基本就干完了，里面的数据应该交给应用程序自己去处理，可是一台机器上跑着这么多的应用程序，应该给谁呢？

无论应用程序写的使用TCP传数据，还是UDP传数据，都要监听一个端口。正是这个端口，用来区分应用程序，要不说端口不能冲突呢。两个应用监听一个端口，到时候包给谁呀？所以，按理说，无论是TCP还是UDP包头里面应该有端口号，根据端口号，将数据交给相应的应用程序。

![](<https://static001.geekbang.org/resource/image/2c/84/2c9a109f3be308dea901004a5a3b4c84.jpg?wh=2183*1103>)

当我们看到UDP包头的时候，发现的确有端口号，有源端口号和目标端口号。因为是两端通信嘛，这很好理解。但是你还会发现，UDP除了端口号，再没有其他的了。和下两节要讲的TCP头比起来，这个简直简单得一塌糊涂啊！

## UDP的三大特点

UDP就像小孩子一样，有以下这些特点：

<!-- [[[read_end]]] -->

第一，**沟通简单**，不需要一肚子花花肠子（大量的数据结构、处理逻辑、包头字段）。前提是它相信网络世界是美好的，秉承性善论，相信网络通路默认就是很容易送达的，不容易被丢弃的。

第二，**轻信他人**。它不会建立连接，虽然有端口号，但是监听在这个地方，谁都可以传给他数据，他也可以传给任何人数据，甚至可以同时传给多个人数据。

第三，**愣头青，做事不懂权变**。不知道什么时候该坚持，什么时候该退让。它不会根据网络的情况进行发包的拥塞控制，无论网络丢包丢成啥样了，它该怎么发还怎么发。

## UDP的三大使用场景

基于UDP这种“小孩子”的特点，我们可以考虑在以下的场景中使用。

第一，**需要资源少，在网络情况比较好的内网，或者对于丢包不敏感的应用**。这很好理解，就像如果你是领导，你会让你们组刚毕业的小朋友去做一些没有那么难的项目，打一些没有那么难的客户，或者做一些失败了也能忍受的实验性项目。

我们在第四节讲的DHCP就是基于UDP协议的。一般的获取IP地址都是内网请求，而且一次获取不到IP又没事，过一会儿还有机会。我们讲过PXE可以在启动的时候自动安装操作系统，操作系统镜像的下载使用的TFTP，这个也是基于UDP协议的。在还没有操作系统的时候，客户端拥有的资源很少，不适合维护一个复杂的状态机，而且因为是内网，一般也没啥问题。

第二，**不需要一对一沟通，建立连接，而是可以广播的应用**。咱们小时候人都很简单，大家在班级里面，谁成绩好，谁写作好，应该表扬谁惩罚谁，谁得几个小红花都是当着全班的面讲的，公平公正公开。长大了人心复杂了，薪水、奖金要背靠背，和员工一对一沟通。

UDP的不面向连接的功能，可以使得可以承载广播或者多播的协议。DHCP就是一种广播的形式，就是基于UDP协议的，而广播包的格式前面说过了。

对于多播，我们在讲IP地址的时候，讲过一个D类地址，也即组播地址，使用这个地址，可以将包组播给一批机器。当一台机器上的某个进程想监听某个组播地址的时候，需要发送IGMP包，所在网络的路由器就能收到这个包，知道有个机器上有个进程在监听这个组播地址。当路由器收到这个组播地址的时候，会将包转发给这台机器，这样就实现了跨路由器的组播。

在后面云中网络部分，有一个协议VXLAN，也是需要用到组播，也是基于UDP协议的。

第三，**需要处理速度快，时延低，可以容忍少数丢包，但是要求即便网络拥塞，也毫不退缩，一往无前的时候**。记得曾国藩建立湘军的时候，专门招出生牛犊不怕虎的新兵，而不用那些“老油条”的八旗兵，就是因为八旗兵经历的事情多，遇到敌军不敢舍死忘生。

同理，UDP简单、处理速度快，不像TCP那样，操这么多的心，各种重传啊，保证顺序啊，前面的不收到，后面的没法处理啊。不然等这些事情做完了，时延早就上去了。而TCP在网络不好出现丢包的时候，拥塞控制策略会主动的退缩，降低发送速度，这就相当于本来环境就差，还自断臂膀，用户本来就卡，这下更卡了。

当前很多应用都是要求低时延的，它们可不想用TCP如此复杂的机制，而是想根据自己的场景，实现自己的可靠和连接保证。例如，如果应用自己觉得，有的包丢了就丢了，没必要重传了，就可以算了，有的比较重要，则应用自己重传，而不依赖于TCP。有的前面的包没到，后面的包到了，那就先给客户展示后面的嘛，干嘛非得等到齐了呢？如果网络不好，丢了包，那不能退缩啊，要尽快传啊，速度不能降下来啊，要挤占带宽，抢在客户失去耐心之前到达。

由于UDP十分简单，基本啥都没做，也就给了应用“城会玩”的机会。就像在和平年代，每个人应该有独立的思考和行为，应该可靠并且礼让；但是如果在战争年代，往往不太需要过于独立的思考，而需要士兵简单服从命令就可以了。

曾国藩说哪支部队需要诱敌牺牲，也就牺牲了，相当于包丢了就丢了。两军狭路相逢的时候，曾国藩说上，没有带宽也要上，这才给了曾国藩运筹帷幄，城会玩的机会。同理如果你实现的应用需要有自己的连接策略，可靠保证，时延要求，使用UDP，然后再应用层实现这些是再好不过了。

## 基于UDP的“城会玩”的五个例子

我列举几种“城会玩”的例子。

### “城会玩”一：网页或者APP的访问

原来访问网页和手机APP都是基于HTTP协议的。HTTP协议是基于TCP的，建立连接都需要多次交互，对于时延比较大的目前主流的移动互联网来讲，建立一次连接需要的时间会比较长，然而既然是移动中，TCP可能还会断了重连，也是很耗时的。而且目前的HTTP协议，往往采取多个数据通道共享一个连接的情况，这样本来为了加快传输速度，但是TCP的严格顺序策略使得哪怕共享通道，前一个不来，后一个和前一个即便没关系，也要等着，时延也会加大。

而**QUIC**（全称**Quick UDP Internet Connections**，**快速UDP互联网连接**）是Google提出的一种基于UDP改进的通信协议，其目的是降低网络通信的延迟，提供更好的用户互动体验。

QUIC在应用层上，会自己实现快速连接建立、减少重传时延，自适应拥塞控制，是应用层“城会玩”的代表。这一节主要是讲UDP，QUIC我们放到应用层去讲。

### “城会玩”二：流媒体的协议

现在直播比较火，直播协议多使用RTMP，这个协议我们后面的章节也会讲，而这个RTMP协议也是基于TCP的。TCP的严格顺序传输要保证前一个收到了，下一个才能确认，如果前一个收不到，下一个就算包已经收到了，在缓存里面，也需要等着。对于直播来讲，这显然是不合适的，因为老的视频帧丢了其实也就丢了，就算再传过来用户也不在意了，他们要看新的了，如果老是没来就等着，卡顿了，新的也看不了，那就会丢失客户，所以直播，实时性比较比较重要，宁可丢包，也不要卡顿的。

另外，对于丢包，其实对于视频播放来讲，有的包可以丢，有的包不能丢，因为视频的连续帧里面，有的帧重要，有的不重要，如果必须要丢包，隔几个帧丢一个，其实看视频的人不会感知，但是如果连续丢帧，就会感知了，因而在网络不好的情况下，应用希望选择性的丢帧。

还有就是当网络不好的时候，TCP协议会主动降低发送速度，这对本来当时就卡的看视频来讲是要命的，应该应用层马上重传，而不是主动让步。因而，很多直播应用，都基于UDP实现了自己的视频传输协议。

### “城会玩”三：实时游戏

游戏有一个特点，就是实时性比较高。快一秒你干掉别人，慢一秒你被别人爆头，所以很多职业玩家会买非常专业的鼠标和键盘，争分夺秒。

因而，实时游戏中客户端和服务端要建立长连接，来保证实时传输。但是游戏玩家很多，服务器却不多。由于维护TCP连接需要在内核维护一些数据结构，因而一台机器能够支撑的TCP连接数目是有限的，然后UDP由于是没有连接的，在异步IO机制引入之前，常常是应对海量客户端连接的策略。

另外还是TCP的强顺序问题，对战的游戏，对网络的要求很简单，玩家通过客户端发送给服务器鼠标和键盘行走的位置，服务器会处理每个用户发送过来的所有场景，处理完再返回给客户端，客户端解析响应，渲染最新的场景展示给玩家。

如果出现一个数据包丢失，所有事情都需要停下来等待这个数据包重发。客户端会出现等待接收数据，然而玩家并不关心过期的数据，激战中卡1秒，等能动了都已经死了。

游戏对实时要求较为严格的情况下，采用自定义的可靠UDP协议，自定义重传策略，能够把丢包产生的延迟降到最低，尽量减少网络问题对游戏性造成的影响。

### “城会玩”四：IoT物联网

一方面，物联网领域终端资源少，很可能只是个内存非常小的嵌入式系统，而维护TCP协议代价太大；另一方面，物联网对实时性要求也很高，而TCP还是因为上面的那些原因导致时延大。Google旗下的Nest建立Thread Group，推出了物联网通信协议Thread，就是基于UDP协议的。

### “城会玩”五：移动通信领域

在4G网络里，移动流量上网的数据面对的协议GTP-U是基于UDP的。因为移动网络协议比较复杂，而GTP协议本身就包含复杂的手机上线下线的通信协议。如果基于TCP，TCP的机制就显得非常多余，这部分协议我会在后面的章节单独讲解。

## 小结

好了，这节就到这里了，我们来总结一下：

- 如果将TCP比作成熟的社会人，UDP则是头脑简单的小朋友。TCP复杂，UDP简单；TCP维护连接，UDP谁都相信；TCP会坚持知进退；UDP愣头青一个，勇往直前；

- UDP虽然简单，但它有简单的用法。它可以用在环境简单、需要多播、应用层自己控制传输的地方。例如DHCP、VXLAN、QUIC等。


<!-- -->

最后，给你留两个思考题吧。

1. 都说TCP是面向连接的，在计算机看来，怎么样才算一个连接呢？

2. 你知道TCP的连接是如何建立，又是如何关闭的吗？


<!-- -->

欢迎你留言和讨论。趣谈网络协议，我们下期见！

## 精选留言(96)

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/c8/342a063f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iiwoai

  每次最后的问题，应该在第二次讲课的时候给答案说出来啊

  2018-06-23

  **4

  **283

- ![img](https://static001.geekbang.org/account/avatar/00/10/fa/ab/0d39e745.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李小四

  网络_10 # 作业 - 连接：在自己监听的端口接收到连接的请求，然后经过“三次握手”，维护一定的数据结构和对方的信息，确认了该信息：我发的内容对方会接收，对方发的内容我也会接收，直到连接断开。 - 断开：经过“四次挥手”确保双方都知道且同意对方断开连接，然后在remove为对方维护的数据结构和信息，对方之后发送的包也不会接收，直到 再次连接。 我看到有的同学说，TCP是建立了一座桥，我认为这个比喻不恰当，TCP更好的比喻是在码头上增加了记录人员，核查人员和督导人员，至于IP层和数据链路层，它没有任何改造。

  作者回复: 这个比喻太好了，对的TCP不是桥，是在码头上增加了记录人员，核查人员和督导人员

  2019-08-07

  **4

  **228

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8e/c4/8d1150f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Richie

  TCP/UDP建立连接的本质就是在客户端和服务端各自维护一定的数据结构（一种状态机），来记录和维护这个“连接”的状态 。并不是真的会在这两个端之间有一条类似“网络专线”这么一个东西（在学网络协议之前脑海里是这么想象的）。 在IP层，网络情况该不稳定还是不稳定，数据传输走的是什么路径上层是控制不了的，TCP能做的只能是做更多判断，更多重试，更多拥塞控制之类的东西。

  作者回复: 理解的太对了

  2019-06-30

  **5

  **141

- ![img](https://static001.geekbang.org/account/avatar/00/11/40/88/b36caddb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  仁者

  我们可以把发送端和接收端比作河流的两端，把传输的数据包比作运送的石料。TCP 是先搭桥（即建立连接）再一车一车地运（即面向数据流），确保可以顺利到达河对岸，当遇到桥上运输车辆较多时可以自行控制快慢（即拥堵控制）；UDP 则是靠手一个一个地扔（即无连接、基于数据报），不管货物能否顺利到达河对岸，也不关心扔的快慢频率。

  2019-02-21

  **

  **69

- ![img](%E7%AC%AC10%E8%AE%B2%20_%20UDP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E5%9B%A0%E6%80%A7%E5%96%84%E8%80%8C%E7%AE%80%E5%8D%95%EF%BC%8C%E9%9A%BE%E5%85%8D%E7%A2%B0%E5%88%B0%E2%80%9C%E5%9F%8E%E4%BC%9A%E7%8E%A9%E2%80%9D.resource/resize,m_fill,h_34,w_34-1662307381001-5631.jpeg)

  赵强强

  第一个问题:TCP连接是通过三次握手建立连接，四次挥手释放连接，这里的连接是指彼此可以感知到对方的存在，计算机两端表现为socket,有对应的接受缓存和发送缓存，有相应的拥塞控制策略

  2018-06-08

  **

  **29

- ![img](https://static001.geekbang.org/account/avatar/00/10/2e/ac/7f056518.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Mr.Peng

  对包分析的工具及书籍分享一些！

  2018-06-10

  **1

  **27

- ![img](https://static001.geekbang.org/account/avatar/00/12/a4/01/22729368.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Havid

  包最终都是通过链路层、物量层这样子一个一个出去的，所以连接只是一种逻辑术语，并不存在像管道那样子的东西，连接在这里相当于双方一种约定，双方按说好的规矩维护状态。

  2018-09-14

  **

  **25

- ![img](https://static001.geekbang.org/account/avatar/00/11/80/a0/26a8d76b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Chok Wah

  可以详细讲讲UDP常见小技巧，面试常问。 比如UDP发过去怎么确认收到； 基于UDP封装的协议一般会做哪些措施。

  2018-06-12

  **1

  **19

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7d/03/4e71c307.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  蓝色理想

  似懂非懂，看来还是没懂╯□╰…如果有代码演示看看效果就很好了👍

  2018-06-08

  **3

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/10/6f/a7/0bc9616f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  耿老的竹林

  UDP协议看起来简单，恰恰提供给了巨大的扩展空间。

  2018-09-22

  **1

  **13

- ![img](https://static001.geekbang.org/account/avatar/00/0f/48/bf/a9c9b1a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Yang

  连接:3次握手之后，咱俩之间确认自己发的东西对方都能收到，然后咱俩各自维护这种状态！连接就建立了……

  2018-06-08

  **

  **13

- ![img](https://static001.geekbang.org/account/avatar/00/11/4d/dc/87809ad2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  埃罗芒阿老师

  1.两端各自记录对方的IP端口序列号连接状态等，并且维护连接状态 2.三次握手四次挥手

  2018-06-09

  **

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  以前觉得UDT应用不广，听了课之后觉得UDT的应用很多，但实际怎么查看应用用的是TCP还是UDP 呢？

  2018-07-03

  **1

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/11/50/1c/26dc1927.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  次郎

  能不能讲解一些打洞之类的知识？

  2018-07-02

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/62/0b/13ee0753.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小q

  上面说可以推荐本分接析包的书籍，是什么书呀

  2018-06-15

  **1

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/0f/88/cd/2c3808ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Yangjing

  后面可以讲一下实际的分析不？比如用工具 wireshark 对包进行分析讲解，自己能看懂一部分简单的

  作者回复: 因为是音频课程，所以不太适合对包进行分析讲解，但是可以推荐本书，有很多书已经非常好了

  2018-06-08

  **8

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/13/11/3e/925aa996.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  HelloBug

  老师好，如何理解HTTP协议的多数据通道共享一个连接？

  作者回复: 一个tcp连接

  2018-11-26

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/12/43/6a/3ea0c553.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Eric

  深入浅出，少见的课程，很好！

  2018-08-08

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/22/fd/56c4fb54.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Skye

  请问老师，TCP自面向字节流，和UDP的数据包具体形式是怎么样的。面向字节流之后下层还怎么封装呀

  2018-06-25

  **1

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/66/34/0508d9e4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  u

  听一遍不够，要多听几遍

  2018-06-09

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/13/f4/14/1d235e1c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  A-李永军

  刘老师，udp传输怎样能避免乱序呢？

  作者回复: udp应该一个包是一个，每个都是完整的，不用排序。如果非得要排，就需要应用层自己来排序了

  2018-11-18

  **2

  **3

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  horsechestnut

  计算机会在内核中维护一个数据结构，以及状态机，去保证tcp是连接的

  2018-08-07

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/66/82/c2acd57e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  蔡呆呆

  异步IO是怎么维护TCP连接的呢，不需要每个连接都在内核中维护数据结构吗？

  2018-08-03

  **

  **3

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  robin

  连接是一种状态，建立连接是维持一种状态，维持状态通过一定的数据结构来完成

  2018-06-12

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/1b/ac/41ec8c80.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  孙悟空

  请教下，这些网络知识如何实践呢

  2018-06-09

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ac/a1/43d83698.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云学

  作者还懂4G协议呀，厉害

  2018-06-08

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/a6/723854ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  姜戈

  技术中穿插小故事，大师风范，我等叹服^_^

  2018-06-08

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/7b/2a/7d8b5943.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LH

  前面说tcp是基于流的传输，无头无尾。后面说UDP的头和tcp不一样，我被搞晕流

  作者回复: UDP的头和TCP的头的意思是网络包的头。这里面说的无头无尾的意思是无始无终，除非显式的关闭，可以一直传输。所以两个“头”的意思不是一个意思

  2020-01-10

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/bd/62/283e24ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  雨后的夜

  "而这个 RTMP 协议也是基于 TCP 的"  -- 这里是不是搞错了，是UDP吧？

  2019-11-08

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/18/6c/d8/e8eda334.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Yang

  我想问一下，如果一个公司开发应用，他们可以自己选择用UDP 还是TCP吗？ 

  作者回复: 没有特殊要求，就tcp吧

  2019-09-08

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/17/85/0a/e564e572.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  N_H

  客户端和服务端建立tcp连接时，为了验证连接是否还在，客户端和服务端之间应该会不停地发送一些确认的信息，保证客户端和服务端之间的连接还在。（推测的）

  作者回复: keepalive

  2019-06-29

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/16/d7/39/6698b6a9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Hector

  其实TCP来源于人类的交流，航海的双方通信交流的一个要义我们术语叫double-check.双方建立通信连接和结束会话一模一样，还对重要信息进行复述。

  2019-05-05

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/db/26/54f2c164.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  靠人品去赢

  关于协议是有状态的还是无状态的，比喻很通俗易懂，想问一下除了TCP，还有那些协议是有状态的？，毕竟基于TCP的HTTP都是无状态的，想扩展一下思路。

  2019-01-09

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8a/5f/c60d0ffe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  硅谷居士

  \1. 五元组，协议、本机ip、本机端口、远程主机ip、远程主机端口 2. 三次握手，四次挥手(也可能是三次挥手)

  2018-07-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8d/8c/8e3d356c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  野山门

  分析的很好。作为程序员经常和TCP和UDP打交道，能行底层原理和优缺点的角度了解，是这个专栏的核心价值。

  2018-06-21

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/87/dd/4f53f95d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  进阶的码农

  赞 每天早上都要看下是否有更新 期待

  2018-06-08

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/77/70/466368e1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杰森莫玛

  结合现实生活、历史来讲解非常有意思，理解的也更快.不是死板的知识点，感谢老师的分享

  2019-09-23

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/57/91/3a082914.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  葡萄有点酸

  问题1：三次握手，即双向都有发送和接收，这样就建立起了一个连接； 问题2：四次挥手，双向都表示不发送也不接受，这样就断开了一个连接。

  2019-09-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/e9/29/629d9bb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天王

  UDP和TCP都是基于传输层的协议，UDP和TCP协议的区别是，UDP是无连接的协议，UDP的特性是不保证不丢包，不保证顺序到达，TCP是基于有连接的协议，有连接是说在通讯之前，先建立一个连接，数据在这个连接里面传输，在本地维护一套数据结构来维护和记录状态和传输包的情况。TCP传输的是字节流，UDP传输的是包，构成有源端口号和目标端口号，剩下的就是数据包。UDP适用的场景，1 比较简单不复杂，网络比较好的内网环境2 不需要一对一建立连接处理，广播的模式3对丢包不敏感，没有重传和等待，丢了还继续重传。

  2019-03-28

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/a3/87/c415e370.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  滢

  UDP应该也有错误检测的吧，老师是否补充下UDP检验和^_^

  作者回复: 有校验，没有重传

  2019-03-12

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  跨路由器的组播，是否意味着可以在内网发组播到路由器连接运营商的端口

  2018-09-17

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/07/1c/a255e6ac.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  SimonKoh

  老师讲的好棒！ 我觉得要充分吸收老师所讲的东西还是需要有一定的网络基础，推荐b站韩立刚的计算机网络

  2018-08-28

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  天天向⬆️

  我基础比较差，一时不了解帧，包，流的区别，老师可以通俗的解释一下吗？

  2018-07-16

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/8f/466f880d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没心没肺

  UDP也有春天

  2018-06-09

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/1f/2f/e35d6a1d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  I.am DZX

  三次握手 四次挥手

  2018-06-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/2b/e0/ca/adfaa551.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  孙新![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  求问老师和各位大佬，有没有什么稍微深入一点的udp相关的网络或者网络编程书籍推荐的。有时候碰到的困惑问题真的比较难解决，也没有人问。身边都同是c++开发，不太懂深入的网络方面的问题。

  2022-01-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/0c/0f/93d1c8eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mickey

  TCP 三次握手，四次分手。

  2021-11-18

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKib3vNM6TPT1umvR3TictnLurJPKuQq4iblH5upgBB3kHL9hoN3Pgh3MaR2rjz6fWgMiaDpicd8R5wsAQ/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  陈阳

  文中：“再如，TCP 是面向字节流的。发送的时候发的是一个流，没头没尾。IP 包可不是一个流，而是一个个的 IP 包。之所以变成了流，这也是 TCP 自己的状态维护做的事情。而 UDP 继承了 IP 的特性，基于数据报的，一个一个地发，一个一个地收。” 这一段怎么理解？ 怎么定义字节流和数据包， 我认为的数据包应该也是二进制字节数组啊

  2021-08-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_5baa01

  \1. 都说 TCP 是面向连接的，在计算机看来，怎么样才算一个连接呢？ 有目标的链接信息，比如端口，然后目标的心跳正常 2。你知道 TCP 的连接是如何建立，又是如何关闭的吗？ 3次握手，4次挥手

  2021-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/55/37/57aeb6af.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没有軒的筝

  1：先说说传输层，传输层是提供应用进程间的逻辑通信，而这逻辑通信有主要的两种方式或者协议，TCP和UDP；应用程序在使用TCP协议传输之前，必须先建立连接，就像打电话一样，先拨通号确定对方已经接通，然后进行通话，最后挂断通话；有人会问，TCP连接到底能解决什么问题，不先连接行不行?首先第一要解决的问题是客户端和服务端要通信，双方必须都得知道对方已经存在，要不然你传也白传，还浪费资源，第二协商一些参数（这个大家可以下面自行了解），第三对资源的分配，比如TCP缓存大小；最后说一下网络层和传输层，网络层和传输层都是提供逻辑通信，网络层是为主机之间提供逻辑通信，运输层是为应用进程之间提供端到端的逻辑通信

  2021-04-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/f8/2b/339660f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Wangyf

  有一个问题，既然 UDP 是无连接的，那么第一次发送 UDP 的时候，目的端口号是怎么获取的？ 看网上说，服务器里的服务使用的都是 1024 以内的数字，那这也说得通。那如果假设服务器上运行的是自定义的服务，没有使用 1024 以内的端口，那么该如何让源主机知道，在监听哪个端口？人肉通知吗？

  2021-03-28

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTI0eGJygV4lh6PJuotKrz1jsZcOdNiaHnUC3y5A2O3yudUQLkzOE8758icDoXlvgpytQ50ibSIc9nJmg/132)

  余巍

  本质上网络上包数据该丢还是丢，udp和tcp都是算法上的实现策略，我理解udp和tcp就像两个极端，很多时候我们更需要中庸一些的策略（算法实现）

  2021-03-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/97/a8/779cdd2f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  你听得到

  statsd 也是使用 UDP 协议的

  2020-12-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/3f/cf/854ae8fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  🍓准准

  有个问题 如果按照tcp强顺序执行的话，像kafka这种有ack参数的，ack=0接许顺序不一致，所以kafka是基于udp的吗

  2020-12-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/18/65/35361f02.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  潇潇雨歇

  1、客户端与服务端建立通信，双方都能请求应答 2、三次握手和四次挥手

  2020-11-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e1/93/365ba71d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈德伟

  1，在计算机看来，TCP连接就是维护了一个占位符或则socket结构数据符号，标记一下状态 2，tcp三次握手建立连接、四次挥手释放连接

  2020-10-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  传输层一直都是自己的痛点，对于UDP知识点觉得简单，却没有和TCP联系在一起进行对比学习，从而学透。

  2020-09-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/1d/64/52a5863b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  大土豆

  有一个问题，为什么udp的包头需要长度字段，ip包不是已经有长度了吗？基于ip的icmp包也没有长度字段。

  2020-09-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/6c/a8/1922a0f5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郑祖煌

  1.简历连接，就是在收发数据之前已经确保通路之间是没有问题的了，通过三次握手已经确定了。同时确定后的连接，收发双方都会维护了相关的信息和状态。 2.通过三次握手建立连接 ，通过四次挥手来断开连接。

  2020-08-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/e9/4e/22c73926.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天然

  老师您好，最近碰到个问题。linux服务器用tcpdump能抓取到发过来的syslog日志，但是程序读取不到，想问下您可能是什么原因呢？用root用户，日志端口是11001

  2020-07-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c2/cc/ca22bb7c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  蓝士钦

  文章中说到移动通讯领域4G的上网数据是营运商自定义的UDP，那么我们移动设备打开浏览器访问网络的时候，难道不是TCP吗？

  2020-07-12

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLnYfSUc8hJ3oLfa39qkNiaXNibs3VyAbgT7ZXasZXp89fRL7YBakIZdNNEE7kClOjN2KpBUuGpacfQ/132)

  wanj

  问题1：tcp的客户端和服务端都会维护一个状态机，当两端通过握手协议，状态机的都太都为established时，即视为建立了连接。

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/e4/73/74dce191.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鼠里鼠气

  对于"TCP是面向字节流"的理解：在网络环境不好的情况下，假设应用层一次性给了传输层很多的数据，这些数据被放在传输层的缓存中。在传输数据的时候，TCP会根据窗口大小等条件，将数据进行分组(分包)，这些分组向下传递，层层封装，最终成为网络上传送的数据包。而我们不应该认为 应用层给的数据会被一次性的传输走，所以这个"流"，我认为可以理解成一个大桶里装满水，分几次倒出来。(前提，你认为一次性倒出来不好。)   不知道理解得对不对?还望大神指点！！

  2020-06-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/1d/de/62bfa83f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  aoe

  之前运维说“UDP风暴”很可怕，所以内网禁止了UDP，真正的互联网也会出现禁止UDP的网关，导致消息发布出去吗？

  2020-05-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  TCP的建立连接,是通过了三次握手来让建立连接的双方彼此感知到了,这就是建立连接,而关闭连接则是四次挥手,正式下达分手通知书

  2020-04-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/14/29/48ad4b9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  看不见的城市

  接收乱序的数据包后，会怎样，如果包都丢失，有序无序有区别吗

  2020-04-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  通过这样的对比，对于TCP与UDP的理解更深刻了。同时，对于UDP的使用也了解到了更多。

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/7d/75/c812597b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yuliz

  。。。小孩子的特性不是愣头青。。。。

  2020-04-01

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  性善，尽力而为，这两个褒义词，值得献给IT人！

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  这节讲到传输层：UDP和TCP。UDP简单、时延小，在如今追求更快的时代有很多应用都是基于UDP的；TCP可靠，对于追求稳的应用是基于TCP的。不同的一些都有各自的优缺点，就如人一样，找到长处优点，基于这些优势发挥出你的魅力，这也是很美妙的事情。

  2020-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/2e/74/88c613e0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  扶幽

  怎样才算是一个连接呢？

  2019-10-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/cf/bd/4fa01a1c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wd2010

  老师，请问这些网络知识如何实践呢

  2019-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/73/90/9118f46d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chenhz

  "但是 TCP 的严格顺序策略使得哪怕共享通道，前一个不来，后一个和前一个即便没关系，也要等着，时延也会加大。" HTTP 1.1 数据包请求响应是异步的，同一个站点同时发起请求，不需要排队的吧？

  2019-09-26

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/40/10/b6bf3c3c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  纯洁的憎恶

  当前不确认，后续不传输，看来是TCP影响实时响应的罪魁祸首啊。

  2019-09-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/ab/58/ae036dfc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  月成FUN

  计算机看来，两端维护相同的数据结构就相当于建立连接了吧

  2019-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/41/87/46d7e1c2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Better me

  1、TCP中所谓的连接在计算机看来就是客户端服务端各自的一发一收成功了，两端各自维护自己内核中的数据结构(记录对方IP端口序列号等可以说是一种状态机)，通过这样感知到对方的存在，从而建立起连接 2、三次握手，四次挥手

  2019-08-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/6e/04/94677062.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  嘎子

  哈哈哈！太赞啦！写的好好笑！

  作者回复: 谢谢

  2019-07-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b4/94/2796de72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的人

  TCP协议通过三次握手建立连接，四次挥手关闭连接

  2019-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/62/49/6332c99b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  man1s

  为什么我感觉从上层往底层讲会好一些

  作者回复: 自顶往下方法，哈哈。 有本书是这样讲的，但是难度太大

  2019-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/bc/59/c0725015.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  彭旭锐

  传输层的“地址”是端口，传输层底层的网络层的地址是IP地址，因此在传输层的区分一个连接需要5个信息：源IP地址、目标IP地址、协议号（UDP/TCP等）、源端口号、目标端口号

  2019-04-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b5/98/ffaf2aca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ronnyz

  第二问：TCP的三次握手和四次挥手

  2019-04-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c1/86/afd6e862.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王浩

  UDP不需要链接传输，速度快，容易丢包，像个纯真的小孩相信整个网络是完美无瑕的。 TCP需要建立链接，每个建立链接需要经过三次握手来建立连接，同时是以流的方式传输数据，并且保证无丢包，不重复，有顺序。

  2019-04-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得 1.TCP是面向连接的，有状态，有自己规则的协议，它以 字节流 的形式传输 数据。 2.UDP是面向无连接，无状态，它继承IP的特性， 网络层 传输数据是基于包的形式，传输在 数据链路层 叫帧，网络层 叫 包，传输层 叫段，笼统的称为 包，MAC地址匹配，IP头里面，有8个协议，UDP的端口号 包括 源端口号 和目标端口号。 UDP的特性 1）沟通简单 2）轻信他人 3）愣头青，不懂权变。 UDP的使用场景 1）需要资源少，内网的模式，可允许丢包 ，DHCP就是UDP的协议 2）可以广播 3）快速且低延迟。 UDP的实际运用场景 1）服从命令 比如Google的QUIC 快速UDP互联网连接  基于UDP的协议 2）流媒体的协议，比如直播 可以丢帧，不能卡顿 3）实时游戏，只在乎 实时数据处理 4）IOT 物联网 因为终端资源少，终端只能囊括 一个很小的芯片 5）移动通讯领域4G的GTP-U是基于UDP协议的。 回答老师的问题，问题1的个人解答：TCP是听过 三次握手 四次挥手，且在交互反馈的时候 有反馈命令，符合 则正常连接，反之 连接异常。 问题2的个人解答：TCP通过握手连接，挥手就是关闭连接，期待老师下一讲的解答。

  2019-01-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/59/1d/c89abcd8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  四喜

  这一课很生动简单

  2019-01-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/59/94/b94b5995.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  自然

  解决了心中很多困惑。谢谢老师。

  2018-12-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ce/ce/761f285c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  潇洒

  问题一：建立连接的本质就是两端都有分配内存空间用于两者的交互 问题二：三次握手，四次挥手 这样理解对吗

  2018-11-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/52/46/3d4f32f7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  TaoLR

  老师讲得真好👨‍🏫

  2018-09-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b9/5d/9e6e7a67.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  潇洒哥er

  老师，你越写越调皮了😝

  2018-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fc/34/c733b116.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  何磊

  第一个问题：所谓链接就是服务端与客户端二者的socket互相保持对方的信息，直到数据发送结束。 第二个问题：断开者主动调用close，让客户端ack后，等应用程序把数据处理完后，客户端再向服务端发送一个close操作，服务器响应一个ack，整个关闭结束。

  2018-09-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9e/2b/221af43f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  番薯佬

  推荐的书呢

  2018-08-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/88/cd/2c3808ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Yangjing

  建立连接在代码层面就是通过 Socket 建立描述符，然后调用 connect 连接。 描述符=socket(xx, xx)  connect()

  2018-08-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/a5/cd/3aff5d57.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alery

  老师，您推荐对包分析的书是什么书呢？

  2018-07-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6c/2f/6cf39bf6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  you~胖子

  希望老师能对基础差的推荐一些书，不然云里雾里的，谢谢老师

  2018-07-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/4f/6fb51ff1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  一步![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  我知道QQ是基于UDP，那 消息应该是基于应用层进行控制的，重发啊什么的

  2018-07-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/3d/54bbc1df.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Jaime

  1 对应与TCP的实现，TCP是维持了一个状态机，那么对于计算机来看，应该是当处于连接状态的时候就是一条链接。 2 TCP是靠三次握手保证了连接成功，四次握手实现了可靠的网络关闭。

  2018-07-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/3b/18/45397ce9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  海波

  推荐的书呢。

  2018-06-20
