# 第11讲 \| TCP协议（上）：因性恶而复杂，先恶后善反轻松

上一节，我们讲的 UDP，基本上包括了传输层所必须的端口字段。它就像我们小时候一样简单，相信“网之初，性本善，不丢包，不乱序”。

后来呢，我们都慢慢长大，了解了社会的残酷，变得复杂而成熟，就像TCP协议一样。它之所以这么复杂，那是因为它秉承的是“性恶论”。它天然认为网络环境是恶劣的，丢包、乱序、重传，拥塞都是常有的事情，一言不合就可能送达不了，因而要从算法层面来保证可靠性。

## 一、TCP包头格式

TCP 头的格式如图所示，它比 UDP 复杂得多。

![](<%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/642947c94d6682a042ad981bfba39fbf.jpg>)

首先，源端口号和目标端口号是不可少的，这一点和 UDP 是一样的。如果没有这两个端口号。数据就不知道应该发给哪个应用。

接下来是包的序号。为什么要给包编号呢？当然是为了**解决乱序的问题**。不编好号怎么确认哪个应该先来，哪个应该后到呢。编号是为了解决乱序问题。

还应该有的就是确认序号。发出去的包应该有确认，要不然我怎么知道对方有没有收到呢？如果没有收到就应该重新发送，直到送达。**确认序号可以解决不丢包的问题**。

TCP 是靠谱的协议，但是这不能说明它面临的网络环境好。从 IP 层面来讲，如果网络状况的确那么差，是没有任何可靠性保证的，而作为 IP 的上一层 TCP 也无能为力，唯一能做的就是更加努力，不断重传，通过各种算法保证。也就是说，对于 TCP 来讲，IP 层你丢不丢包，我管不着，但是我在我的层面上，会努力保证可靠性。

这有点像如果你在北京，和客户约十点见面，那么你应该清楚堵车是常态，你干预不了，也控制不了，你唯一能做的就是早走。打车不行就改乘地铁，尽力不失约。

接下来有一些状态位。例如 SYN 是发起一个连接，ACK 是回复，RST 是重新连接，FIN 是结束连接等。TCP 是面向连接的，因而双方要维护连接的状态，这些带状态位的包的发送，会引起双方的状态变更。

还有一个重要的就是窗口大小。TCP 要做流量控制，通信双方各声明一个窗口，标识自己当前能够的处理能力，别发送的太快，撑死我，也别发的太慢，饿死我。

作为老司机，做事情要有分寸，待人要把握尺度，既能适当提出自己的要求，又不强人所难。除了做流量控制以外，TCP 还会做拥塞控制，对于真正的通路堵车不堵车，它无能为力，唯一能做的就是控制自己，也即控制发送的速度。不能改变世界，就改变自己嘛。

通过对 TCP 头的解析，我们知道要掌握 TCP 协议，重点应该关注以下几个问题：

- 顺序问题 ，稳重不乱；

- 丢包问题，承诺靠谱；

- 连接维护，有始有终；

- 流量控制，把握分寸；

- 拥塞控制，知进知退。

## 二、TCP的三次握手

所有的问题，首先都要先建立一个连接，所以我们先来看**连接维护**问题。

TCP的连接建立，我们常常称为三次握手。

A：您好，我是A。

B：您好A，我是B。

A：您好B。

我们也常称为“请求->应答->应答之应答”的三个回合。这个看起来简单，其实里面还是有很多的学问，很多的细节。

首先，为什么要三次，而不是两次？按说两个人打招呼，一来一回就可以了啊？为了可靠，为什么不是四次？

我们还是假设这个通路是非常不可靠的，A 要发起一个连接，当发了第一个请求杳无音信的时候，会有很多的可能性，比如第一个请求包丢了，再如没有丢，但是绕了弯路，超时了，还有 B没有响应，不想和我连接。

A 不能确认结果，于是再发，再发。终于，有一个请求包到了 B，但是请求包到了 B 的这个事情，目前 A 还是不知道的，A 还有可能再发。

B 收到了请求包，就知道了 A 的存在，并且知道 A 要和它建立连接。如果B不乐意建立连接，则A会重试一阵后放弃，连接建立失败，没有问题；如果B是乐意建立连接的，则会发送应答包给A。

当然对于B来说，这个应答包也是一入网络深似海，不知道能不能到达A。这个时候B自然不能认为连接是建立好了，因为应答包仍然会丢，会绕弯路，或者A已经挂了都有可能。

而且这个时候B还能碰到一个诡异的现象就是，A和B原来建立了连接，做了简单通信后，结束了连接。还记得吗？A建立连接的时候，请求包重复发了几次，有的请求包绕了一大圈又回来了，B会认为这也是一个正常的的请求的话，因此建立了连接，可以想象，这个连接不会进行下去，也没有个终结的时候，纯属单相思了。因而两次握手肯定不行。

B 发送的应答可能会发送多次，但是只要一次到达A，A就认为连接已经建立了，因为对于A来讲，他的消息有去有回。A会给B发送应答之应答，而B也在等这个消息，才能确认连接的建立，只有等到了这个消息，对于B来讲，才算它的消息有去有回。

当然A发给B的应答之应答也会丢，也会绕路，甚至B挂了。按理来说，还应该有个应答之应答之应答，这样下去就没底了。所以四次握手是可以的，四十次都可以，关键四百次也不能保证就真的可靠了。只要双方的消息都有去有回，就基本可以了。

好在大部分情况下，A和B建立了连接之后，A会马上发送数据的，一旦A发送数据，则很多问题都得到了解决。例如A发给B的应答丢了，当A后续发送的数据到达的时候，B可以认为这个连接已经建立，或者B压根就挂了，A发送的数据，会报错，说B不可达，A就知道B出事情了。

当然你可以说A比较坏，就是不发数据，建立连接后空着。我们在程序设计的时候，可以要求开启keepalive机制，即使没有真实的数据包，也有探活包。

另外，你作为服务端B的程序设计者，对于A这种长时间不发包的客户端，可以主动关闭，从而空出资源来给其他客户端使用。

三次握手除了双方建立连接外，主要还是为了沟通一件事情，就是**TCP包的序号的问题**。

A要告诉B，我这面发起的包的序号起始是从哪个号开始的，B同样也要告诉A，B发起的包的序号起始是从哪个号开始的。为什么序号不能都从1开始呢？因为这样往往会出现冲突。

例如，A连上B之后，发送了1、2、3三个包，但是发送3的时候，中间丢了，或者绕路了，于是重新发送，后来A掉线了，重新连上B后，序号又从1开始，然后发送2，但是压根没想发送3，但是上次绕路的那个3又回来了，发给了B，B自然认为，这就是下一个包，于是发生了错误。

因而，每个连接都要有不同的序号。这个序号的起始序号是随着时间变化的，可以看成一个32位的计数器，每4微秒加一，如果计算一下，如果到重复，需要4个多小时，那个绕路的包早就死翘翘了，因为我们都知道IP包头里面有个TTL，也即生存时间。

好了，双方终于建立了信任，建立了连接。前面也说过，为了维护这个连接，双方都要维护一个状态机，在连接建立的过程中，双方的状态变化时序图就像这样。

![](<%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/c067fe62f49e8152368c7be9d91adc08.jpg>)

一开始，客户端和服务端都处于CLOSED状态。先是服务端主动监听某个端口，处于LISTEN状态。然后客户端主动发起连接SYN，之后处于SYN-SENT状态。服务端收到发起的连接，返回SYN，并且ACK客户端的SYN，之后处于SYN-RCVD状态。客户端收到服务端发送的SYN和ACK之后，发送ACK的ACK，之后处于ESTABLISHED状态，因为它一发一收成功了。服务端收到ACK的ACK之后，处于ESTABLISHED状态，因为它也一发一收了。

## 三、TCP四次挥手

好了，说完了连接，接下来说一说“拜拜”，好说好散。这常被称为四次挥手。

A：B啊，我不想玩了。

B：哦，你不想玩了啊，我知道了。

这个时候，还只是A不想玩了，也即A不会再发送数据，但是B能不能在ACK的时候，直接关闭呢？当然不可以了，很有可能A是发完了最后的数据就准备不玩了，但是B还没做完自己的事情，还是可以发送数据的，所以称为半关闭的状态。

这个时候A可以选择不再接收数据了，也可以选择最后再接收一段数据，等待B也主动关闭。

B：A啊，好吧，我也不玩了，拜拜。

A：好的，拜拜。

这样整个连接就关闭了。但是这个过程有没有异常情况呢？当然有，上面是和平分手的场面。

A开始说“不玩了”，B说“知道了”，这个回合，是没什么问题的，因为在此之前，双方还处于合作的状态，如果A说“不玩了”，没有收到回复，则A会重新发送“不玩了”。但是这个回合结束之后，就有可能出现异常情况了，因为已经有一方率先撕破脸。

一种情况是，A说完“不玩了”之后，直接跑路，是会有问题的，因为B还没有发起结束，而如果A跑路，B就算发起结束，也得不到回答，B就不知道该怎么办了。另一种情况是，A说完“不玩了”，B直接跑路，也是有问题的，因为A不知道B是还有事情要处理，还是过一会儿会发送结束。

那怎么解决这些问题呢？TCP协议专门设计了几个状态来处理这些问题。我们来看断开连接的时候的**状态时序图**。

![](<https://static001.geekbang.org/resource/image/bf/13/bf1254f85d527c77cc4088a35ac11d13.jpg?wh=1693*1534>)

断开的时候，我们可以看到，当A说“不玩了”，就进入FIN\_WAIT\_1的状态，B收到“A不玩”的消息后，发送知道了，就进入CLOSE\_WAIT的状态。

A收到“B说知道了”，就进入FIN\_WAIT\_2的状态，如果这个时候B直接跑路，则A将永远在这个状态。TCP协议里面并没有对这个状态的处理，但是Linux有，可以调整tcp\_fin\_timeout这个参数，设置一个超时时间。

如果B没有跑路，发送了“B也不玩了”的请求到达A时，A发送“知道B也不玩了”的ACK后，从FIN\_WAIT\_2状态结束，按说A可以跑路了，但是最后的这个ACK万一B收不到呢？则B会重新发一个“B不玩了”，这个时候A已经跑路了的话，B就再也收不到ACK了，因而TCP协议要求A最后等待一段时间TIME\_WAIT，这个时间要足够长，长到如果B没收到ACK的话，“B说不玩了”会重发的，A会重新发一个ACK并且足够时间到达B。

A直接跑路还有一个问题是，A的端口就直接空出来了，但是B不知道，B原来发过的很多包很可能还在路上，如果A的端口被一个新的应用占用了，这个新的应用会收到上个连接中B发过来的包，虽然序列号是重新生成的，但是这里要上一个双保险，防止产生混乱，因而也需要等足够长的时间，等到原来B发送的所有的包都死翘翘，再空出端口来。

等待的时间设为2MSL，**MSL**是**Maximum Segment Lifetime**，**报文最大生存时间**，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。因为TCP报文基于是IP协议的，而IP头中有一个TTL域，是IP数据报可以经过的最大路由数，每经过一个处理他的路由器此值就减1，当此值为0则数据报将被丢弃，同时发送ICMP报文通知源主机。协议规定MSL为2分钟，实际应用中常用的是30秒，1分钟和2分钟等。

还有一个异常情况就是，B超过了2MSL的时间，依然没有收到它发的FIN的ACK，怎么办呢？按照TCP的原理，B当然还会重发FIN，这个时候A再收到这个包之后，A就表示，我已经在这里等了这么长时间了，已经仁至义尽了，之后的我就都不认了，于是就直接发送RST，B就知道A早就跑了。

## 五、TCP状态机

将连接建立和连接断开的两个时序状态图综合起来，就是这个著名的TCP的状态机。学习的时候比较建议将这个状态机和时序状态机对照着看，不然容易晕。

![](<https://static001.geekbang.org/resource/image/fd/2a/fd45f9ad6ed575ea6bfdaafeb3bfb62a.jpg?wh=2447*2684>)

在这个图中，加黑加粗的部分，是上面说到的主要流程，其中阿拉伯数字的序号，是连接过程中的顺序，而大写中文数字的序号，是连接断开过程中的顺序。加粗的实线是客户端A的状态变迁，加粗的虚线是服务端B的状态变迁。

## 小结

好了，这一节就到这里了，我来做一个总结：

- TCP包头很复杂，但是主要关注五个问题，顺序问题，丢包问题，连接维护，流量控制，拥塞控制；

- 连接的建立是经过三次握手，断开的时候四次挥手，一定要掌握的我画的那个状态图。

最后，给你留两个思考题。

1. TCP的连接有这么多的状态，你知道如何在系统中查看某个连接的状态吗？

2. 这一节仅仅讲了连接维护问题，其实为了维护连接的状态，还有其他的数据结构来处理其他的四个问题，那你知道是什么吗？

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(181)

- 老师您好，昨天阿里云出故障，恢复后，我们调用阿里云服务的时后出现了调用出异常  connection reset。netstat看了下这个ip发现都是timewait，链接不多，但是始终无法连接放对方的服务。按照今天的内容，难道是我的程序关闭主动关闭链接后没有发出最后的ack吗？之前都没有问题，很不解

  2018-06-28

  **6

  **51

- 

  进阶的码农

  置顶

  状态机图里的不加粗虚线看不懂什么意思 麻烦老师点拨下

  作者回复: 其他非主流过程

  2018-06-11

  **

  **18

- Jerry银银

  看了大家的留言，很有感触，对于同一个技术点，技术的原理是一样的，可是每个人都有自己的理解，和自己独有的知识关联，因为每个人过往所积累的知识体系有所不同。 我对TCP，甚至整个网络知识，喜欢用「信息论」中的一些理论去推演它们。我觉得在现在和未来的时代，「信息论」是一把利器。 信息论中，有个很重要的思想：要想消除信息的不确定性，就得引入信息。将这个思想应用到TCP中，很容易理解TCP的三次握手和四次挥手的必要性：它们的存在以及复杂度，就是为了消除不确定性，这里我们叫「不可靠性」吧。拿三次握手举例： 为了描述方便，将通信的两端用字母A和B替代。A要往B发数据，A要确定两件事： 1. B在“那儿”，并且能接受数据 —— B确实存在，并且是个“活人”，能听得见 2. B能回应  —— B能发数据，能说话 为了消除这两个不确定性，所以必须有前两次握手，即A发送了数据，B收到了，并且能回应——“ACK” 同样的，对于B来说，它也要消除以上两个不确定性，通过前两次握手，B知道了A能说，但是不能确定A能听，这就是第三次握手的必要性。 当然你可能会问，增加第四次握手有没有必要？从信息论的角度来说，已经不需要了，因为它的增加也无法再提高「确定性」

  2019-05-17

  **22

  **269

- 

  TaoLR

  旧书不厌百回读，熟读深思子自知啊。 再一次认识下TCP这位老司机，这可能是我读过的最好懂的讲TCP的文章了。同时有了一个很爽的触点，我发现但凡复杂点儿的东西，状态数据都复杂很多。还有一个，也是最重要的，我从TCP中再一次认识到了一个做人的道理，像孔子说的：“不怨天，不尤人”。人与人相处，主要是“我，你，我和你的关系”这三个处理对象，“你”这个我管不了，“我”的成长亦需要时间，我想到的是“我和你的关系”，这个状态的维护，如果能“无尤”，即不抱怨。有时候自己做多点儿，更靠谱点，那人和人的这个连接不是会更靠谱么？这可能是我从TCP这儿学到的最棒的东西了。 感谢老师的讲解，让我有了新的想法和收获。

  作者回复: 赞

  2018-09-23

  **7

  **121

- 

  krugle

  流量控制和拥塞控制什么区别

  作者回复: 一个是对另一端的，一个是针对网络的，即流量控制是照顾通信对象 拥塞控制是照顾通信环境

  

- 如果是建立链接了，数据传输过程链接断了，客户端和服务器端各自会是什么状态？ 或者我可以这样理解么，所谓的链接根本是不存在的，双方握手之后，数据传输还是跟udp一样，只是tcp在维护顺序、流量之类的控制

  作者回复: 是的，连接就是两端的状态维护，中间过程没有所谓的连接，一旦传输失败，一端收到消息，才知道状态的变化

  

- 

  为什么要三次握手 三次握手的目的是建立可靠的通信信道，说到通讯，简单来说就是数据的发送与接收，而三次握手最主要的目的就是双方确认自己与对方的发送与接收是正常的。 第一次握手：Client 什么都不能确认；Server 确认了对方发送正常 第二次握手：Client 确认了：自己发送、接收正常，对方发送、接收正常；Server 确认了：自己接收正常，对方发送正常 第三次握手：Client 确认了：自己发送、接收正常，对方发送、接收正常；Server 确认了：自己发送、接收正常，对方发送接收正常 所以三次握手就能确认双发收发功能都正常，缺一不可。

  2019-03-27

  **1

  **62

- ![img](https://static001.geekbang.org/account/avatar/00/14/45/f5/b3d7febf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Adolph

  为什么要四次挥手 任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接。 举个例子：A 和 B 打电话，通话即将结束后，A 说“我没啥要说的了”，B回答“我知道了”，但是 B 可能还会有要说的话，A 不能要求 B 跟着自己的节奏结束通话，于是 B 可能又巴拉巴拉说了一通，最后 B 说“我说完了”，A 回答“知道了”，这样通话才算结束。

  作者回复: 是的

  2019-03-27

  **5

  **51

- 

  麋鹿在泛舟

  多谢分享，精彩。扫除了我之前很多的疑问。tcp连接的断开比建立复杂一些，本质上是因为资源的申请（初始化）本身就比资源的释放简单，以c++为例，构造函数初始化对象很简单，而析构函数则要考虑所有资源安全有序的释放，tcp断连时序中除了断开这一重要动作，另外重要的潜台词是“我要断开连接了 你感觉把收尾工作做了”

  作者回复: 谢谢

  2018-06-11

  **2

  **42

- 

  麋鹿在泛舟

  评论区不能互评，如下两个问题是我的看法，不对请指出 多谢  我们做一个基于tcp的“物联网”应用（中国移动网络），如上面所说tcp层面已经会自动重传数据了，业务层面上还有必要再重传吗？如果是的话，业务需要多久重传一次？ --- TCP的重传是网络层面和粒度的，业务层面需要看具体业务，比如发送失败可能对端在重启，一个重启时间是1min，那就没有必要每秒都发送检测啊. 1、‘序号的起始序号随时间变化，...重复需要4个多小时’，老师这个重复时间怎么计算出来的呢？每4ms加1，如果有两个TCP链接都在这个4ms内建立，是不是就是相同的起始序列号呢。 答:序号的随时间变化，主要是为了区分同一个链接发送序号混淆的问题，两个链接的话，端口或者IP肯定都不一样了.2、报文最大生存时间（MSL）和IP协议的路由条数（TTL）什么关系呢，报文当前耗时怎么计算？TCP层有存储相应时间？ 答:都和报文生存有关，前者是时间维度的概念，后者是经过路由跳数，不是时间单位.

  作者回复: 赞

- 

  张亚

  可以用 netstat 或者 lsof 命令 grep 一下 establish listen close_wait 等这些查看

  2018-06-11

  **

  **30

- ![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,m_fill,h_34,w_34-1662307433379-5903.jpeg)

  李小四

  网络_11 读完今天呢内容后，有一个强烈的感受：技术的细节非常生动。 之前对于TCP的感知就是简单的“三次握手”，“四次挥手”，觉得自己掌握了精髓，但随便一个问题就懵了，比如， - 客户端什么时候建立连接？ > 根据以前的认知，会以为是“三次握手”后，双方同时建立连接。很显然是做不到的，客户端不知道“应答的应答”有没有到达，以及什么时候到达。。。 - 客户端什么时候断开连接？ > 不仔细思考的话，就会说“四次挥手”之后喽，但事实上，客户端发出最后的应答(第四次“挥手”)后，永远无法知道有没有到达。于是有了2MSL的等待，在不确定的网络中，把问题最大程度地解决。 TCP的状态机，以及很多的设计细节，都是为了解决不稳定的网络问题，让我们看到了在无法改变不稳定的底层网络时，人类的智慧是如果建立一个基本可靠稳定的网络的。

  作者回复: 赞

  2019-08-07

  **3

  **26

- ![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,m_fill,h_34,w_34-1662307433379-5904.jpeg)

  eason2017

  老师好，我看书中说是计数器每4微妙就加1的。

  作者回复: 赞

  2018-07-04

  **

  **23

- ![img](https://static001.geekbang.org/account/avatar/00/16/4d/62/bfbf8ee3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  走过全世界。

  三次握手连接就是： A：你瞅啥 B：瞅你咋地 A：再瞅一个试试 然后就可以开始亲密友好互动了😏

  作者回复: 哈哈哈

  2019-06-03

  **4

  **20

- ![img](https://static001.geekbang.org/account/avatar/00/11/54/04/c143ae3c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  korey

  敲黑板，这是面试重点

  2018-07-24

  **2

  **18

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/7d/370b0d4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  西部世界

  我也纠结了4ms加一重复一次的时间是4个多小时的问题。 具体如下: 4ms是4微秒，等于100万分之一秒，32位无符号数的最大值是max=4294967296(注意区分有符号整数最大值) 公式如下4/1000000*max/60/60=4.772小时。

  作者回复: 赞

  2018-11-27

  **2

  **18

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/c5/3467cf94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  正是那朵玫瑰

  老师有几个疑问： 1、当三次握手建立连接后，每次数据交互都还会ack吗？比如建立连接后，客户端发送数据包给服务器端，服务器成功收到数据包后会发送ack给客户端么？ 2、如果建立连接后，客户端和服务器端没有任何数据交互，双方也不主动关闭连接，理论上这个连接会一直存在么？ 3、2基础上，如果连接一直会在，双方又没有任何数据交互，若一方突然跑路了，另一方怎么知道对方已经不在了呢？在java scoket编程中，我开发客户端与服务器端代码，双方建立连接后，不发送任何数据，当我强制关闭一端时，另一端会收到一个强制关闭异常，这是如何知道对方已经强制关闭了呢？

  作者回复: 是的，每次都ack。可以有keepalive，如果不交互，两边都不像释放，那就数据结构一直占用内存，对网络没啥影响。必须是发送数据的时候，才知道跑路的事情。你所谓的强制关是怎么关？有可能还是发送了fin的

  2018-06-12

  **4

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7d/d8/d7c77764.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  HunterYuan

  老师回答，@正是那朵玫瑰的第一个问题，感觉稍微有点问题，我理解，不是每次数据交互的会发ack，有些ack是可以合并的，用于减少数据包量，通过wirshark抓包，也证实了。理由是，因为ack号是表示已收到的数据量，也就是说，它是告诉发送方目前已接收的数据的最后一个位置在哪里，因此当需要连续发送ack时，只要发送最后一个ack号就可以了，中间的可以全部省略。同样的机制还有ack号通知和窗口更新通知合并。这是我的理解，若有问题，希望老师多多指教。谢谢

  2018-07-14

  **1

  **15

- ![img](https://static001.geekbang.org/account/avatar/00/11/11/4f/be6a03b6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  babos

  四次挥手是不是可以做成三次，把第二步和第三步的ACK和FIN一起发 ——————— 因为b可能还有一些事情没做完，需要做完了才能结束

  2018-07-02

  **4

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/a6/723854ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  姜戈

  Tcp三次握手设计被恶意利用 ，造就了ddos。

  作者回复: 是有连接攻击的

  2018-06-11

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/11/22/e8/8e154de6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  圆嗝嗝

  四次握手为什么需要等待2MSL？ A向B发送ACK时，可能会发生两种情况： (1) A发送的ACK包没有到达B: 那么B会重发FIN包给A，A收到FIN包后再次向B发送ACK (2) A发送的ACK包到达B：那么B会关闭连接，A与B断开连接。 对于第一种情况，A向B发送ACK包在网络中存活的时间是1MSL；B因为没有收到A发过来的ACK包，所以B再次向Ａ发送FIN包，FIN包在网络中存活的时间是1MSL。在网络没有故障(重发的FIN被丢弃或错误转发)，那么A一定能在2MSL之内收到该FIN，因此A只需要等待2MSL。

  2019-04-24

  **1

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/12/f1/15/8fcf8038.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  William![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  感谢老师耐心回答，另外RFC文档上，microsecond是微秒哈，毫秒是milisecond😁

  作者回复: 哦，谢谢指正

  2018-11-23

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/12/f1/15/8fcf8038.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  William![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  我认为每4微妙，seq自增1这个说法不太靠谱，可能是以讹传讹，计算出来是4.7小时左右一轮回。随着网卡速率加快，我觉得这个数据会变小。可以通过wireshark抓包实验计算一下。 http://www.cnblogs.com/chenboo/archive/2011/12/19/2293327.html

  作者回复:   To avoid confusion we must prevent segments from one incarnation of a  connection from being used while the same sequence numbers may still  be present in the network from an earlier incarnation.  We want to  assure this, even if a TCP crashes and loses all knowledge of the  sequence numbers it has been using.  When new connections are created,  an initial sequence number (ISN) generator is employed which selects a  new 32 bit ISN.  The generator is bound to a (possibly fictitious) 32  bit clock whose low order bit is incremented roughly every 4  microseconds.  Thus, the ISN cycles approximately every 4.55 hours.  Since we assume that segments will stay in the network no more than  the Maximum Segment Lifetime (MSL) and that the MSL is less than 4.55  hours we can reasonably assume that ISN's will be unique.

  2018-11-21

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  老师再问个问题，TCP保证有序（后续到的包会等待之前的包），流量控制，拥塞机制，导致网络出现抖动时，延迟性就高，响应慢，，为什么要在应用层写重发机制，，毕竟TCP保证有序性，意义不大啊😄

  作者回复: 应用层重试是解决应用层的错误，假设你调用一个进程，但是没调用成功，挂了，重试是给另一个进程发

  2018-06-11

  **2

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/1d/37/76/6c85bc5a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  水到渠成

  “……RST 是重新连接，……”“……A 就表示，我已经在这里等了这么长时间了，已经仁至义尽了，之后的我就都不认了，于是就直接发送 RST，B 就知道 A 早就跑了。”感觉这里不太理解，为什么早就跑路了却要发一个RST,那不是又要建立连接了，但是我已经不需要建立连接了呀！难道我要在连接上然后专门过去说一下再见？

  作者回复: 这是个特殊处理流程，rst的一般意思是重新连接，但是收到包的时候，可以根据上下文进行处理。这有点像咱们约定返回值，到了边界条件，某个返回值就代表特殊意义。

  2020-06-03

  **2

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/12/87/e7/043f9dda.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  .

  之前看过《UNIX网络编程 卷I》，看这几节简直不要太舒服，跟吃豆腐脑一样。

  作者回复: 原来吃豆腐脑如此舒服呀，请问豆腐脑应该是甜的还是咸的？

  2019-09-03

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ce/ce/761f285c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  潇洒

  四次挥手为什么不能做成三次，是因为释放资源时间较长吗，释放资源长那就等释放完资源再发确认不行吗，反正A第一次接收到确认前后状态都一样。 对于这点困惑了好久

  2018-11-21

  **1

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/13/49/3c/5d54c510.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  静静聆听

  说一下四次握手关闭的理解，A发送完后，不能直接跑路，因为tcp要保证可靠性，发送完后不能保证B一定会受到，所以A说不玩了，要等待B的相应，以便后续出现问题重新发送，B接收到A说不玩了，也不能直接关闭，只能给A发送一个ack，因为B接受到数据后很可能还要进行处理，等处理完后B在给A发送FIN和ACK，然后等待A的响应，A收到后关闭，发送给BACK，然后B接受到才能关闭，之所以要四次握手关闭，其实建立在可靠性传输的条件下

  2018-11-14

  **

  **5

- ![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,m_fill,h_34,w_34-1662307433379-5901.jpeg)

  小田

  掌握 TCP 协议，重点应该关注以下几个问题： 顺序问题 ，稳重不乱； 丢包问题，承诺靠谱； 连接维护，有始有终； 流量控制，把握分寸； 拥塞控制，知进知退。

  2018-06-12

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/5a/b273689e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  洪宇舟

  刘老师，最后的状态机中，细虚线表示什么呢？

  2018-06-11

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/27/0b/12dee5ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  进化论

  keepalive机制老师能深入讲解下吗

  2018-06-11

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/5d/16/9a57266b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘宝明

  老师可以再教程里加一点小实验吗

  2018-06-11

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/1f/ff/0a/1a8734c4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Cooper

  还有一个异常情况就是，B 超过了 2MSL 的时间，依然没有收到它发的 FIN 的 ACK，怎么办呢？按照 TCP 的原理，B 当然还会重发 FIN，这个时候 A 再收到这个包之后，A 就表示，我已经在这里等了这么长时间了，已经仁至义尽了，之后的我就都不认了，于是就直接发送 RST，B 就知道 A 早就跑了。 在这里超过B超过2MSL，那A不是已经跑路了吗，还怎么会收到包呢？

  2020-08-14

  **1

  **3

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  涤生

  tcp三次握手 第一次握手是客户端送送，第二次握手服务端能确认自己接收正常，客户端发送和接收正常，第三次握手服务端确认自己接收和发送都正常，所以缺一不可

  2020-02-04

  **

  **3

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/TTPicMEZ5s1zKiaKYecIljicRV9dibITPM0958W3VuNHXlTQic2Dj6XYGibF7dqrG3JWr0LMx7hY9zXGzuvTDNGw8KAA/132)

  阳光梦

  四次挥手为何有fin+ack？为何不只是fin？

  作者回复: 已经建立的连接，在没有完全关闭之前，要按既定的流程来

  2019-08-16

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/13/5c/cb/65b38e27.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wang-possible

  老师您好，如果一个端口是如何接收多个客户端的请求

  作者回复: 多个socket连接

  2019-07-24

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/b3/798a4bb2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  帽子丨影

  Java服务器，close_wait特别多，怎么定位到是那一块的代码有问题呢

  作者回复: 是有socket没有close吧

  2019-03-25

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/f1/15/8fcf8038.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  William![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  TCP包头格式一节，图后第三段第四句话有语病，“解决不丢包问题”，应该是”解决丢包问题“。另外，关于序号的解读有误，序号并非每4ms自增1，如果是，时间4个多小时也对不上，查了《TCP/IP》详解，其数字和传输的包大小以及SYN状态位有关，在 SYN flag 置 1 时，此为当前连接的初始序列号（Initial Sequence Number, ISN），数据的第一个字节序号为此 ISN + 1；在 SYN flag 置 0 时，为当前连接报文段的累计数据包字节数。

  作者回复: 刚看了一下rfc793 page 26，是四毫秒增一呀

  2018-11-21

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/96/f9/728063a2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  龐校長

  小孩快乐很简单 成人简单很快乐

  2018-08-14

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/2c/cd/f6f4dfb8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Df

  TCP是可靠连接，建立在解决“不可靠”的网络基础上。无论建立连接 还是 关闭连接，每一方 都要经历 “我准备好了” & “对方确认了”， 才能算完成。 所以每一方 都要有一个 发送（SYN/FIN）& 收到ACK，才算完成。 所以理论上，连接还是断开连接，都要 四次（1 A发 - 2 B确认 - 3 B发 - 4 A确认） 但是连接时，2、3可以合并，所以就变成了三次握手。 断开时，2、3不可以合并，所以还是四次挥手。 至于断开 2、3为什么不能合并，@Adolph 打电话的栗子其实很贴切。 第一次 FIN：A完成自己工作了 可以关闭了，然后告诉B “我完成了”。 第二次 ACK：但是B可能还没完活，所以B只能先回复A“收到了”。 第三次 FIN： 然后B接着处理自己的事，直到工作完成了，B才能发出可以关闭的报文 “我也完成了”。 第四次 ACK：最后A回复“收到了”

  2020-12-11

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/20/9d/4a/09a5041e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  TinyCalf

  老师你好，读到这段有个疑问，希望解答一下： “一旦 A 发送数据，则很多问题都得到了解决。例如 A 发给 B 的应答丢了，当 A 后续发送的数据到达的时候，B 可以认为这个连接已经建立” 那么也就是说，A根本不需要发送最后的ACK，而是直接发送数据，也可以正常建立连接，那岂不是两次握手也可以了？

  2020-09-21

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  老师 文中提到的icmp报文  是那台 将ttl减为0上的那台路由器上发出的吗？

  作者回复: 是的

  2019-09-02

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f6f02b

  因而，每个连接都要有不同的序号。这个序号的起始序号是随着时间变化的，可以看成一个 32 位的计数器，每 4ms 加一，这里有个疑问就是，这个序号只跟时间有关跟，上一个序号无关是吗，比如上一个序号是 X ,那么如果过来 8ms 再发下一个包，那么这个包的 tcp 序号就是 x+2 ,而不是 x+1 是吗？

  作者回复: 只有起始序号是随着时间变的。 发送的序号是不断加一的

  2019-04-19

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/a6/43/cb6ab349.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Spring

  RST是重新连接，为什么A在2MSL后收到B的FIN包会发RST呢？难道此时的A已经变成了另一个应用？

  作者回复: 是的，早就结束了。

  2019-04-03

  **

  **2

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIHq3CYG3iaEwGv7FVYZGXaGYbGHc1VNmog1ByZNtTjYJdmdATQI64icd7P1hmS4uib2wbn1cicKYkjPw/132)

  yeoman

  “The generator is bound to a (possibly fictitious) 32 bit clock whose low order bit is incremented roughly every 4 microseconds.” 上面是从RFC739 p26复制的。 4 microseconds = 4微秒 = 4μs 所以是4μs自增1

  作者回复: 对的，微妙

  2019-02-14

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/f4/b4492888.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Talent

  老师你好，连接和断开的状态时序图中，小写的ack和seq是什么，值是什么，有什么用？谢谢老师

  2018-06-28

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  我是大神郑

  老师，您好。MPLS与传统IP交换的区别能帮我解答一下吗？谢谢

  2018-06-12

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  还是我老师，，TCP重传，底层会处理去重后传给上层？   那会不会把缓冲区撑死啊，毕竟要过滤对比去重，再传给上层。。那一个包rece缓存区生命周期是？

  作者回复: 是的，去重后给上层，这就是分层的意义，下面的算法对上面透明。不会撑死，同样的包只会缓冲一个，正是因为没有收到才重发

  2018-06-11

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/66/34/0508d9e4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  u

  MSL有2分钟吗？怎么可能等这么长的时间才断开连接？

  作者回复: 文章里写了，不一定是这个时间。应用层断开，你会发现系统里面有很多timewait

  2018-06-11

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/6c/d4/85ef1463.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  路漫漫

  看完透视HTTP协议，再回来看这个TCP就很有感觉了，多看几次，总是很有收获

  2021-11-21

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/83/af/1cb42cd3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  马以

  三次握手就像是确认恋爱关系 女：你爱我吗？ 男：我爱你，你爱我吗？ 女：我也爱你！

  2021-09-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/ae/8b/43ce01ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ezekiel

  为什么是三次握手，我觉得还有一个原因是。A发报文B收到，但是B的返回报文A不一定能收到。A和B的报文都有去有回，才说明A和B是互通的。

  作者回复: 是的，双方都有去有回

  2019-07-12

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/88/cd/2c3808ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  Yangjing

  连接成功后，如果一方断电 TCP 的连接变化还是 4 次挥手吗？

  作者回复: 还有half close

  2019-06-22

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  关于序号确认号的一个说法, 我觉得说得非常好, 可以帮助大家彻底理解问题. 序列号为当前端成功发送的数据位数，确认号为当前端成功接收的数据位数，SYN标志位和FIN标志位也要占1位

  2019-05-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/b9/af/f59b4c7c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  深海极光

  老师请问一下，我们都说tcp链接是四元组，从ip协议层有5元组的说法，是加上协议类型，那是不是说明同一个端口既能接收tcp的包也能接收udp的包呢？

  作者回复: tcp监听了，udp就不能用了。

  2019-03-07

  **4

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erTlRJ6skf7iawAeqNfIT1PPgjD7swUdRIRkX1iczjj97GNrxnsnn3QuOhkVbCLgFYAm7sMZficNTSbA/132)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  senekis

  老师讲的太好了，之前我也一直没搞懂为什么tcp需要三次握手，老师一语点醒梦中人，因为握手两端都要保证握手信息有来有回一次，所以是三次握手。 感觉终于有多学会一个知识点，很开心！

  2019-03-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/df/1e/cea897e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  传说中的成大大

  有一个疑问 如果客户端处于time_wait状态，超过了设定的最大报文生存时间，是否就会释放掉端口了? 置顶那个阿里云问题 一直处于time_wait状态难道是因为设置的等待时间太久了？

  作者回复: 是的，会释放。你这个是不是短连接太多，创建删除连接太频繁，所以看着一直有

  2019-03-04

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEL5xnfuicbtRz4F87AAjZX6oCEjMtYiaIu4iaQichQmy0vEBA6Sumic1RDvUCeuBEqj6iatnt2kENbKYmuw/132)

  dexter

  老师，我现在一直没搞明白这个udp 和tcp包头中里面的“数据”是什么数据，是应用层封装的数据吗

  作者回复: 是的

  2019-03-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/55/f7/af3feb56.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jike_chanpin_shishabi

  老师，客户端请求服务端的时候如果客户端断开连接，timewait不应该是在客户端机器上么，为什么服务器上有好多timewait

  2019-02-17

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/94/2c22bd4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  克里斯

  这个时候，还只是 A 不想玩了，也即 A 不会再发送数据，但是 B 能不能在 ACK 的时候，直接关闭呢？当然不可以了，很有可能 A 是发完了最后的数据就准备不玩了，但是 B 还没做完自己的事情，还是可以发送数据的，所以称为半关闭。 老师能举例说明一下B不能用直接关闭的场景吗？

  作者回复: B可以直接关闭，就会主动发FIN

  2019-01-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/94/2c22bd4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  克里斯

  网络通信是在应用程序自己的进程，还是在tcp进程？

  作者回复: 没有tcp进程的说法，tcp在内核里面

  2019-01-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得：tcp的连接 比 udp复杂，因为需要考虑很多问题，比如网络、丢包、阻塞、乱序 等许多问题，tcp 提倡 网络传输的性恶论 比较考虑实际问题，它应用成人考虑现实问题的思维，udp 性善论，小孩子 天真无邪的只关注自己，忽视现实的环境，场景不一样 应该使用不同的协议。 tcp 协议 考虑 稳重不乱的顺序问题、承诺靠谱的丢包问题、有始有终的连接维护、把握分寸的流量控制、知进退的硬塞控制。 TCP包头 包括：源端口号 和目的端口号、序号、确定序号、窗口大小 、校验码、紧急指针、选项 和数据。 IP层为第三次，TCP为传输层 为第四层，tcp的使命是 不改变网络世界，只改变自己。 tcp三次握手，四次挥手 里面包括各种状态 以及相应的处理情况，比如空连接，可以使用 keepalive机制验证，判断为空连接 是否释放这次的连接资源。 三次挥手 主要是 请求--应答--应答的应答，比如老师 所说的 A：你好，我是A， B：你好A，我是B，A：你好B，不知道 为什么三次握手，二次可能不保险，四次以上看似稳妥，其实 不能保证 一直在连接，所以只有 相互确认反应，才能保证 是否一直处于连接状态，所以三次 是相对的 效率高 且有保障的 连接次数。 四次挥手 比三次握手 复杂一点，握手 只需要 建立连接，挥手 得考虑 相互的状态，事情做好了没，我要下线了，收到请回复，有一个 把事情收尾 和 最后相互 告别的过程，所以 处理的事情多一点。 在握手和挥手时，还需要考虑时间问题，不能一直占用资源，所以得考虑时效性，所以MSC和TTC很有必要，有同学解读区别：MSC是报文最大生存时间，它为数据维度，TTC 为IP协议的路由条数，它也为路由跳转的数目。 tcp的传输过程包括 SYN发生数据、ACT回复、RST重连接、Fin结束连接。 回答老师的问题：1.查看连接的状态 可以使用（1）返回的信息 （2）日志系统 查看 有不对 请指正。

  2019-01-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  老师，在看一些资料里面，提及TCP报文头时，保留字只有3位，TCP flag里面多了ECN的识别号，这个跟拥塞控制处理有关，这种情况常用吗？

  2019-01-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8f/ad/6e3e9e15.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  产品助理

  netstat查看我们服务器上信息，（服务器跑着Ngx + PHP)，按理说应该是客户端（用户浏览器等）链接我们服务器，服务器相当于图中的B。 但是也有很多timeout，这个是指我们的业务在系统输出（相当于php进程处理完用户的请求业务逻辑，并退出）后，我们的服务器先发送了 FIN吗？

  2018-11-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/45/33/59a7be1e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  蔡波

  这个序号的起始序号是随着时间变化的，可以看成一个 32 位的计数器，每 4ms 加一，如果计算一下，如果到重复，需要 4 个多小时 ………………………………………… 以上这段时间貌似不对，4,294,967,296/900000（每小时数值变化的次数）≈4772小时

  2018-10-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/15/5c/aa3f8306.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  潇是潇洒的洒

  我想问下，为什么TCP四次挥手的图中，Server端恢复的FIN请求中，ack=p+1而不是ack=p+2

  作者回复: 因为从来没有包p+1传过来，只能ack传过来的包的下一个

  2018-09-28

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/21/38/548f36ef.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ylck

  能讲一讲 bbr 是怎么做到加速流量的吗？

  2018-08-23

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8a/5f/c60d0ffe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  硅谷居士

  顺序问题，seq。丢包问题，重传机制。流量控制，虚拟发送窗口。拥塞控制，到达拥塞点之前指数增长，到达后线性增长。

  2018-07-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4e/94/0b22b6a2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Luke

  A 建立连接的时候，请求包重复发了几次，有的请求包绕了一大圈又回来了，B 会认为这也是一个正常的的请求的话，因此建立了连接，可以想象，这个连接不会进行下去，也没有个终结的时候，纯属单相思了。因而两次握手肯定不行。tcp链接不是有序的吗？难道建立连接的时候发送包是无序的，连接建立好了后，才是有序的？

  作者回复: 发包都是无序的，是接收端的缓存来保证顺序

  2018-06-12

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/e2/1fad12eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张洪阆

  我们做一个基于tcp的“物联网”应用（中国移动网络），如上面所说tcp层面已经会自动重传数据了，业务层面上还有必要再重传吗？如果是的话，业务需要多久重传一次？

  2018-06-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  还是我老师，，TCP重传，底层会处理去重后传给上层？   那会不会把缓冲区撑死啊，毕竟要过滤对比去重，再传给上层。。那一个包rece缓存区生命周期是？

  作者回复: 不对比，有个序列号，同一个序列号只接收一次

  2018-06-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/bc/88a905a5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赵强强

  如果客户端主动关闭连接，当服务器处于CLOSE_WAIT状态时，教科书（谢希仁）说仍然可以给客户端发送数据，请问发送的数据还需要客户端确认吗？应用层一般都是通过Socket进行编程，客户端通过调用close方法关闭连接，关闭后已经无法在处理输入和输出，那CLOSE_WAIT阶段传送的数据仅仅是放置到了TCP接收缓存中吧，应用层应该已经无法感知了。 请老师帮忙解答一下，谢谢。

  作者回复: 我记得谢老师说的是半关闭的时候，closewait已经不行了吧，我再去查一下

  2018-06-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/bc/88a905a5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赵强强

  1、‘序号的起始序号随时间变化，...重复需要4个多小时’，老师这个重复时间怎么计算出来的呢？每4ms加1，如果有两个TCP链接都在这个4ms内建立，是不是就是相同的起始序列号呢。 2、报文最大生存时间（MSL）和IP协议的路由条数（TTL）什么关系呢，报文当前耗时怎么计算？TCP层有存储相应时间？ 请老师帮忙解答一下，谢谢😬 

  2018-06-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/29/f7/49/791d0f5e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  UDP少年

  在不光能学到技术知识，如果你喜欢看评论的话，还能从网友哪里学到做人的道理，总之在这里氛围到位了。

  2022-08-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/10/bb/f1061601.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Demon.Lee![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  四次挥手流程多，任何一个中间状态都可能出现异常，老师看我下面的理解对吗，谢谢。 - A 发 FIN 报文之前，B 已经跑路了（假设停电宕机，下同），A 不知道：  A 停在 FIN_WAIT_1 状态，A 会不断重发 FIN 报文，直到超时（至于超时后的处理逻辑依赖代码是怎么写的） - A 发 FIN 报文后，B 没来得及回复 ACK 就跑路了：   同上 - A 发 FIN 报文后，B 回复 ACK 后，B 跑路：  A 停在 FIN_WAIT_2 状态，TCP 规范未对该状态进行处理，但是 Linux 通过 `/proc/sys/net/ipv4/tcp_fin_timeout` 来控制 FIN 包的最长等待时间，过了这个时间，就强制关闭。 tcp_fin_timeout (integer; default: 60; since Linux 2.2)：This specifies how many seconds to wait for a final FIN  packet before the socket is forcibly closed.  This is strictly a violation of the TCP specification, but required to prevent denial-of-service attacks.  In Linux 2.2, the default value was 180. - B 开始发送 FIN 包，B 的状态由 CLOSE_WAIT 变成 LAST_ACK，A 收到后，从 FIN_WAIT_2 变成 TIME_WAIT，如果此时 A 跑路：   B 不断重发 FIN 报文，直到超时

  2022-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/ea/10661bdc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinsu

  TCP 状态机 是必须要记忆的知识点吗？

  2022-04-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/7b/47/96dad3ff.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  Hunter_Dark

  老师讲的都是正常状态吧？学着学着脑海里的问题越来越多，第一个四次挥手也是客户端触发的吗？服务端可以触发close吗？keeplived引入？

  2022-02-10

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/XktNguBOU1CyXt2QoNtY6TAHnfA1QEqXZmdn7whTZggeKSiaibnbRwnHR1T1XqPDQrVJDbUicFw4bXBzlNby71a1g/132)

  夜色下云淡风轻

  对四次挥手TIME_WAIT有点迷惑，百度了一篇文章是这样说法，如果没有TIME_WAIT，在B的FIN没收到A的ACK的情况下，B此时会重发FIN包，但是由于A这边连接已经不存在了，A表示不理解没有连接何来断开？直接返回一个RST，然后B就会收到错误(常见如:connect reset by peer)

  2022-01-12

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/34/cf/0a316b48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  蝴蝶

  比如第一次握手时客户端该发SYN信号，那这个报文里面的响应序列号是0吗？是根据状态位来判断吗？

  2021-12-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0c/46/dfe32cf4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多选参数

  这些非主流的过程是啥意思？为什么会有 A、B 同时发送 SYN 的时候？

  2021-09-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/7c/91/0d8e2472.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  子非鱼

  netstat：net statistic可以查看socket连接的状态，常用netstat -anp

  2021-09-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/34/df/64e3d533.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  leslie

  再次重修看到的东西又不同了，突然发现原来知识是相通的；之前刘老师说网络和计算机组成是互通的，可是做为一个偏系统与硬件和DB领域的从业者其实网络这块一直学的很烂，昨天一个网络狠人说各种问题都是找之间的关系，今早再次学习发现许多数据系统中的事情原来同样在网络中的通的。

  2021-09-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/ec/4d/1551ed5f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  前端小猪

  老师，我想问一下，一个tcp连接中，服务端可以主动关闭连接吗？如果能，如果客户端还有发送数据呢，是否会等数据发完才会完全断开？

  2021-07-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/fb/85/f3fd1724.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  PHP菜鸟

  比喻的成分有点多了,其实没必要弄这么多比喻,

  2021-06-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/a4/1b/0d7b9ee1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  了凡

  老师，A，B是两个服务器，各自为对方的客户端和服务器，如果A和B同时发起连接请求，并且同时到达，是对应各自产生有两个socket处理自己的连接和响应还是怎么处理

  2021-06-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/a4/1b/0d7b9ee1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  了凡

  老师，问一下，第三次握手，SYN标记为0，是为了防止歧义吗，还是什么原因

  2021-05-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/51/b0/d32c895d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  熊能

  老哥，稳

  2021-04-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/12/dc/d09d1afb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  沭沭沭

  结束请求为什么是发送端先发起的，服务端如果发送完了就发起结束请求可以吗

  2021-03-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/12/dc/d09d1afb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  沭沭沭

  前边看的挺明白的，最后一张图把我给整懵逼了

  2021-03-16

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLRLdXHaicsXMB27zm6YCD4uAMwSh8l3xMTdYhyN8KIuaGcgoURlKsolrjibichMgiact5VoK0QTPp7Aw/132)

  undefined

  两次握手问题：假设 A 发了 2 个建立连接请求，然后与 B 简单沟通后关闭，这时第二个建立连接请求到了，B 以为 A 又想建立连接，那么 B 就会建立一个错误的链接 两次挥手问题：B 如果未把数据完全发送完，下一个使用 A 的端口将会接收到一些无效数据，可能会影响数据 三次挥手问题：A 如果未收到 B 的 Close 请求，这个时候如果 B 已经关闭，那么 A 永远都不会关闭

  2021-01-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fd/67/66ba04fd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  鹏

  这是个特殊处理流程，rst的一般意思是重新连接，但是收到包的时候，可以根据上下文进行处理。这有点像咱们约定返回值，到了边界条件，某个返回值就代表特殊意义>> 这一段我着实没有理解. 老师交流一下我的理解是否有问题. A并不知道B是否接到自己的ACK，A是这么想的： 1）如果B没有收到自己的ACK，会超时重传FiN那么A再次接到重传的FIN，会再次发送ACK 2）如果B收到自己的ACK，也不会再发任何消息，包括ACK无论是1还是2，A都需要等待，要取这两种情况等待时间的最大值，以应对最坏的情况发生， 这个最坏情况是：去向ACK消息最大存活时间（MSL) + 来向FIN消息的最大存活时间(MSL)。这恰恰就是2MSL( Maximum Segment Life)。 如果A的等待超过了2MSL确实仁尽义至, 自然就去执行ReSet, 进入CloseState, 那么B呢，一直超时等待不到A的Ack, 数次尝试之后，收不到A的Ack, 默认A已经关闭，B也执行Reset, 进入CloseState，至此双方都关闭.

  2020-12-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/70/9f/741cd6a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Henry

  提个问题。如果服务端最后一直发送FIN会怎样？

  2020-11-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/b1/17/4e533c1b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王

  关于tcp关闭时的四次挥手我还不太明白，希望大家可以答疑解惑一下。 如果是客户端主动关闭连接的话，客户端就进入fin1状态了；等服务器收到客户端的fin后，会给客户端发送ack；客户端收到服务器的ack后进入fin2状态。 我想问一下服务器这里给客户端发送ack时为什么不能带着fin一起发送呢。按照我的理解，客户端发送完fin后，就不会再发送数据了，对应到程序里面是不是就是close（socket描述符）呢？难道客户端在（客户端close后服务器close前）还可以继续接受服务器发送的数据吗？而且服务器在收到客户端的fin后它也知道人家客户端要关闭了，服务器还给人家发数据是不是有些过分呢。还是说服务器在接受到客户端的fin后有可能还有客户端在close前发送给它的数据还没有接受完，所以服务器不能直接发送fin。这个问题我hin费解呀

  2020-11-29

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/34/d3/81aacf46.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Lougar

  老师你好，请教一下，用wireshark抓的tcp包，数据包里的ack id一直不变，可能是啥原因哈

  2020-11-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e1/93/365ba71d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈德伟

  1，netstat -antp 或ss 

  2020-10-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6b/3d/ae41c2b3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  data

  哈哈每读一遍都有新的收获

  2020-10-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/bf/cd6bfc22.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  自然

  java'应用在服务器上需要调用别的应用的http接口，产生了大量的time_wait，这个应该如何正确和优雅的解决？

  2020-09-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/47/00/3202bdf0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  piboye

  timewait状态最主要作用还是为了保障网络中的包已经消失了，这里有个问题想请教老师。 nat网络中，客户端主动关闭，nat上网络设备上会有timewait不？nat为了避免seq回绕会做seq的转换不？ 还有收sync包发现seq回绕，会回复reset包不？

  2020-09-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/9d/4a/09a5041e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  TinyCalf

  “客户端收到服务端发送的 SYN 和 ACK 之后，发送 ACK 的 ACK，之后处于 ESTABLISHED 状态，因为它一发一收成功了。服务端收到 ACK 的 ACK 之后，处于 ESTABLISHED 状态，因为它也一发一收了。” 我觉得这里“ACK的ACK”的表述有点问题。为了避免之前谈到的无限回应的问题，所有ACK包本身都是严格不需要回应的，第三次握手的ACK其实时第二次握手SYN的回应，而不是ACK的回应

  2020-09-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/65/20/65dd107f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  大兔叽

  老师，提一个小问题哈； 「这个序号的起始序号是随着时间变化的，可以看成一个 32 位的计数器，每 4ms 加一，如果计算一下，如果到重复，需要 4 个多小时，那个绕路的包早就死翘翘了，因为我们都知道 IP 包头里面有个 TTL，也即生存时间。」 这里面说的4ms其实应该是写错了，应该是μs，否则算出来的值会是4千多小时

  2020-08-19

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_65ee97

  关于为什么需要3次握手，我见过最精辟的解释就是 A->B  B->A 这个过程 对A来说它可以确认AB之间的收+发是正常的 B->A  A->B 这个过程 对B来说它可以确认BA之间的收+发是正常的 两端互相都确认收发正常就可以建立连接了。

  2020-08-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/4b/a276c1d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  儘管如此世界依然美麗

  老师，我突然有一个问题，就是TCP连接建立的时候它们会进行TCP包序号确认的沟通，这里的序号和TCP首部的序号是一回事嘛？ 希望可以解答一下

  2020-07-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/7f/13/3f5a8b96.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李阳

  感觉三次握手和三体的发现好像，不要回答，不要回答，不要回答。

  2020-06-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/a2/5e/3871ff79.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  迷途书童

  就这一篇来说，作者讲的不够彻底。 为什么TCP要三次握手？因为要保证发送方/接受方知道自己和对方有收发的能力

  2020-06-27

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/cc/de/e28c01e1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  剑八

  tcp主要是解决网络的可靠连接通信问题，是一个协议。 主要解决通信过程中： 1.顺序问题 2.丢包问题 3.连接问题 4.流量控制 5.拥塞控制 tcp建立连接是3次握手: 客户端状态：close->syn_send->connected 服务端状态: close->listener->sync_recieve->connected tcp关闭连接是4次握手: 客户端状态：connected->fin_wait1->fin_wait2->time_wait->closed 服务端状态:  connected->close_wait->last_ack->closed time_wait的状态主要是解决两个问题： 1.如果最后服务端说我也要关闭了的消息，客户端没收到 客户端保挂这个time_wait多等的时间可以让服务端重发【我也要关闭】的消息并能接收到这个消息 2.再有一个如果客户端没有这个time_wait的时间的话，直接在这个端口进行新的连接处理的话，就会接收到上个连接的服务器一些延迟的包到达， 这里主要是双保险，保证在新进程复用这个端口前，前面链接的包都已经挂了

  2020-06-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c7/0c/8e7f1a85.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Tintin

  TCP建立连接的时候，图片中表示的是服务器需要设置SYN标志，是不是还需要设置ACK标志呢？

  作者回复: 回复设置ack

  2020-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/41/87/46d7e1c2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  Better me

  TCP面向连接，UDP无连接 正因为连接的特性，才使得TCP能够处理大量的网络异常情况 比如乱序 通过序列号来实现 比如丢包重传，通过确认序列号实现 比如流量控制和拥塞控制，通过流量窗口和一定的降速传的算法来实现 TCP的面向连接需要客户服务两端通过一定的数据结构维护之间的连接状态，连接是很抽象的(根本不存在)，其表现形式就是两端各自维护的状态 UDP的无连接特性一般适用于请求资源小，网络情况较好的内网以及传输时延敏感的应用 三次握手需确认双方各自接受发送信息的能力 四次挥手需双方各自主动断开才能真正的断连(可能藕断丝连，即客户端已完成发送FIN和接受ACK，但是服务端此时还有数据需要处理，所以需另外一次单独发起FIN断连的交互)，所以需要四次

  2020-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/ac/83/3a613ec3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  左左4143

  对于握手需要三次，挥手需要四次，我可不可以这样理解：连接只需要一方主动发起，另一方回复，之前的一方再次确认就可以。而断开需要双方都发起，同时另一方都回复才可以，所以断开要比连接多一步

  2020-05-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/2a/3d/3642672b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  上弦月

  TCP4次挥手状态时序图，服务端在发送FIN的时候，序号seq=q，写成了sep=q。不过我相信大家都能理解这seq是sequence序号的意思。

  2020-05-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  第三次握手，ack=y+1，这个时候的seq还是x+1吗？

  2020-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  TCP中的顺序号与确认号，是客户端与服务端公用的吗？比如seq是客户端与服务端共同使用的吗？

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  关于三次握手，个人觉得在画图模型的时候，应该把客户端与服务端对应的信号状态与顺序号、确认号分开表示。不要逗使用同一个标示。比如ACK表示回复，ack=x+1，这样只是大小写区分而已，容易把人弄混淆反而不清晰。状态是状态，数据是数据。前人怎么总结是前人的理解，后人应该根据自己的理解有所改变，而不是一味的沿用前人的理解或总结，勉强解释或说明。不知道我是否说明白了，望老师参考一下。

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  seq表示sequence number（顺序号），ack表示acknowledge number（确认号）。即对应TCP数据包中的序号和确认号，是吗？

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  三次握手中的seq是啥意思？这个是一个变量吗？它与syn和ack的关系是什么？起初seq=x，然后又变成seq=y？这个在实际握手的时候是什么样的机制呢？难道是一个bit位吗？

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/56/06/ea49b29d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小洛

  请教老师问题，为什么需要三次握手，我看留言说是为了确认双方的收发能力是正常，但是我看文章老师说了一个诡异现象，就是重复的请求A到达B，然后认为是一个正常的请求，建立了链接，开辟了资源等待A的应答，所以第三次握手是不是为了资源复位？这才是三次握手的根本原因

  2020-03-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8f/4d/65fb45e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  学个球

  对于四次挥手断开连接的描述 “A 直接跑路还有一个问题是，A 的端口就直接空出来了，B 原来发过的很多包很可能还在路上”。 有一个疑问是，当 B 向 A 发 FIN 的时候，没有确定自己要发送的包均收到了来自 A 的 ACK 吗？意思就是，B 已经确定了没有包还在路上的情况，所以 B 才会向 A 去发送 FIN。

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/a5/b8/17129075.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  benamazing

  解释得挺好的，比网上的要易懂好多

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/4b/75/64ddeffe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  minions

  求问题二答案

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  TCP的报文比较复杂，其中很多字段都有其作用。从保持顺序、报文重发、连接保持和流控等等作用，通过报文中的序列号、状态位和窗口大小等来达到目的。就如自身一样，相应的五官和六腑都有其作用，各司其职，这一个系统运行就正常。

  2020-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/74/57/7b828263.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张sir

  关于timewait为什么是2MSL还是不理解，感觉没有因果关系

  2020-03-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/71/48/44df7f4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  凯

  1.接收端怎么通过确认序号，判断包的完整性 2. 三次握手请求，A 向 B 发送syn请求， 到底有没有到底B，这个是全靠B响应的ack？ 如果是这样，三次握手就好理解了， A和B打电话， A： 喂		B听到了，说明A的话筒、B的听筒是没有问题， B： 诶，谁呀    B回答，测试自己的话筒，A的听筒是不是有问题， A： 我是xxx	A收到，说明A听筒是好的，B的话筒是好的。 	B收到，他就知道自己的话筒是没有问题。

  2020-03-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/54/cf/fddcf843.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  芋头

  老师，请问下，三次握手的时候，当客户端发情 SYN的时候 seq=x，而服务端返回ack的时候，为啥ack是x+1而不是x呢

  2020-02-24

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTI4akcIyIOXB2OqibTe7FF90hwsBicxkjdicUNTMorGeIictdr3OoMxhc20yznmZWwAvQVThKPFWgOyMw/132)

  Chuan

  老师，请问下，如果是服务器B主动关闭，也是四次挥手吗？即： B：我不想玩了。 A：知道了。 A：我没有要发送的了。 B：知道了。 请问是这样吗？还是只是两次： B：我不想玩了。 A：知道了。 我感觉应该还是四次，保证通信双方平等性？万一A还有话没说完？但是如果A还有话没说完，即使说了，B还会接着处理吗？因为它已经不想工作了！

  2020-02-17

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/kC4X5YKgOj5yGibYGBbVfBtf2tvKiaFhY5lk0bdZ1O104flDHBpjdMzUFHjqgl44sHXrzmNsLaz6Sqx1iaLAy6TDA/132)

  鱼鱼

  redis如果没有配置连接空闲超时时间，经常就会出现服务端有大量连接，但是这些连接的对端，也就是客户端，都已经不存在了。就会出现新的客户端连接redis报无法连接的错误。

  2020-02-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/d2/f7/fabad335.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天边的一只鱼

  老实您好，send()函数是把数据传送到发送缓冲区，具体什么时候发送是由TCP控制的， TCP是什么时候发送的？ 是等发送缓冲区写满还是间隔一段时间发送一次？

  2019-12-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f4/b1/4a43e5e1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  梨桃桃

  tcp的三次握手类似异地两个人确认恋爱关系，男方写信给女方说我喜欢你，之后心里在嘀咕女方喜不喜欢我，女方收到信后回信给男方说我也喜欢你。女方看到后心里兴奋，感觉这关系成了，于是写信给男方说我也喜欢你。但是心里也嘀咕，不知道男方收到我的信了没有。最后男方收到女方信后男方释疑了。男方又回信给女方说我知道你也喜欢我了，女方收到信后，也释疑了。于是，他们就彼此认为关系确立了

  2019-11-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/a0/69/e42348a8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李奇锋

  为什么图二最后一条连线内容是？ ACK, seq=x+1, ack=y+1  不应该是？ ACK, ack=y+1 

  2019-10-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/0c/30/bb4bfe9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  lyonger

  老师，请问下，线上服务端出现很多close_wait状态的链接，有办法强制让服务端释放吗？除了重启进程。谢谢啦

  2019-10-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/ce/e8/12cb8e99.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  小松松

  请问一下，当TCP开始第一次握手的时候 seq一定是0吗？

  2019-10-23

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/e2/b1/d00399c0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  心有林夕

  老师，看谢希仁计算机网络中，说到关闭连接时，等待2MSL就是防止“已失效的连接请求报文段”出现在本连接中。那么上面您举例的二次握手的诡异现象，关闭连接后又收到了饶了一圈的SYN请求，这个说法是不是不成立呢？

  2019-10-19

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  vip3

  老师讲得很棒，开头的故事很引人入胜。 向您请教个问题。 我想让pad和PC之上的两个应用全双工通信，想到两种办法。 一是和服务器建立websocket连接，让其作为通讯的中转，但是这种方法网络不好的情况下容易挂掉； 为了解决办法一的问题，想了第二个方法，在PC上启动一个websocket server，让pad应用与PC应用内启的这个server建立连接，实现局域网内部全双工通信，但这有个限制，必须要求pad和PC连到同一局域网下。但是因为PC可能连接着网线，也可能在用无线网络，且无线网络不止一个，我检测不出来PC和pad连接的是否是同一子网。 所以想问问老师，有没有什么办法能解决上述两个办法的问题？

  2019-10-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/fd/91/65ff3154.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  _stuView

  拥塞控制和流量控制有什么区别呢

  2019-10-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/54/b2/5ea0b709.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Danpier

  老师，有个疑问，客户端第4次挥手如果没到服务端，服务端重新发送 FIN 请求，客户端再次接收到 FIN 请求会重新刷新等待时间吗？ 还有，既然客户端等待超时关闭连接，服务端之后的 FIN 请求会接收到 RST 回应，那客户端第4次挥手后直接关闭连接，服务端收到 RST 不就能避免尬等（LAST-ASK）状态了吗？那客户端等待的意义也就只剩防止新应用和旧的数据包混淆了。

  2019-09-07

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/6a8fRQFxX5VXOpRKyYibsemKwDMexMxkzZOBquPo6T4HOcYicBiaTcqibDoTIhZSjVjF3nKXTEGDYOGPt2xqqwiawjg/132)

  咩咩咩

  还有一个异常情况就是，B 超过了 2MSL 的时间，依然没有收到它发的 FIN 的 ACK，怎么办呢？按照 TCP 的原理，B 当然还会重发 FIN，这个时候 A 再收到这个包之后，A 就表示，我已经在这里等了这么长时间了，已经仁至义尽了，之后的我就都不认了，于是就直接发送 RST，B 就知道 A 早就跑了。 请问这是什么意思?A发送RST,但是这不能保证B收到了RST了啊

  作者回复: 不用保证，RST要是丢了，A还可以再发，再被RST

  2019-08-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/4e/0f/c43745e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  hola

  老师，linux里面的这个变量net.ipv4.tcp_fin_timeout 到底指代的是什么意义呢。 tcp详解第一卷里面， 说time_wait的等待时间由这个变量记录 然后说 FIN_WAIT_2状态的等待时间也由这个变量记录 然后有些博客说，其实time_wait状态的超时时间，并没有读取这个变量，而是由宏定义的。 有点懵逼

  作者回复: tcp_fin_timeout这个参数决定了它保持在FIN-WAIT-2状态的时间

  2019-07-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/9c/35/50133278.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  chris

  这个序号的起始序号是随着时间变化的，可以看成一个 32 位的计数器，每 4ms 加一，如果计算一下，如果到重复，需要 4 个多小时，那个绕路的包早就死翘翘了，因为我们都知道 IP 包头里面有个 TTL，也即生存时间。 请问一下是4毫秒还是4微秒？

  作者回复: 微秒

  2019-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ae/8b/43ce01ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ezekiel

  断开的时候，那个状态图。在什么情况下，有用？

  作者回复: 当然有用呀。很多连接断了没有释放，占用系统资源，都是要看这个状态图的

  2019-07-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/92/8d/ab469ad5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黄强

  对3次握手，我的理解是第1,2次握手是确认A到B是完整发送、接收能力的，第2,3次握手是确认B到A也是有完整的发送、接收能力了，那么我们可以开始建立连接了

  2019-06-14

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  起风了001

  感觉很多细节没有说到. 比如序号和确认号分别是什么, 服务端给客户端发送数据, 分别对应的这两个号是什么值;这样只简单说确认号就是用来确认的, 这等于没说呀.

  2019-05-20

  **

  **

- ![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,m_fill,h_34,w_34-1662307433378-5898.jpeg)

  灯盖

  赞赞赞

  2019-05-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/ea/10661bdc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinsu

  netstat -anpt 可以在centos系统中查看某个进程的连接的状态

  2019-04-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/24/18/8edc6b97.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Roger

  拥塞控制:就是使用前面讲过的三种方式吗?信道划分、轮流协议、随机接入协议。

  作者回复: 不是的，不是的，不同层次了，这是TCP层

  2019-04-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/90/e5/45064235.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  林间野猪狂奔

  以前只知道有“三次握手”、“四次挥手”，知道三次握手的时候的几个状态，但是不清楚原来四次挥手也有这么多的状态时序，长知识了

  2019-04-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ac/ed/f379c523.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  献礼

  ISN不从1开始，主要是为了安全吧

  作者回复: 是的

  2019-04-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0d/ad/3787a71a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古轩。

  tcp状态机细的虚线啥子意思吗？

  作者回复: 非主流场景

  2019-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/62/d7/5bfdecf2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chen泽林¹⁵⁶²⁶¹⁸⁷³³⁹

  你好， 我用idea的自带数据库管理工具控制台， 执行查询， 隔4分钟左右， 再执行一次， 会卡顿，  用 wireshark抓包结果每次卡顿都发了几次[TCP Retransmission] 。 请问是什么原因造成， 有什么解决方案

  2019-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/3b/67/46c45977.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  知君

  想问一下，在三次握手中，客户端收到服务端的SYN和ACK然后回复ACK,为什么说这个ACK是回复服务端的ACK的ACK而不是回复SYN的呢？我的想法是SYN请求连接，ACK回复SYN 不知道有什么问题，ack不是也等于y+1而不是x+2么

  作者回复: 客户端收到的不是服务端的SYN和ACK，而是收到服务端的SYN的ACK.

  2019-02-26

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/94/2c22bd4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  克里斯

  寝室里面，一个网端口连接一个网卡，如果只有一个端口就只能连接一个电脑，这种情况下如何让多个电脑能连上呢？

  作者回复: 一台电脑买两个网卡，当路由器，或者直接买个家用路由器

  2019-01-30

  **

  **

- ![img](https://time.geekbang.org/0)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  lingyou

  问题1.请问下打开tcp_tw_reuse参数有什么顾虑点嘛，为什么系统没有默认打开？ 问题2.请问什么情况下一个链接的状态在进入fin_wait_2后，迅速的就结束，看不到进入time-wait状态？ 问题3.请问有什么工具能记录到一个4元组的所有状态迁移，ss之类的刷新频率限制了有的状态看不到

  2018-12-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/1f/ef/72769407.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  •̅_•̅

  好在大部分情况下，A 和 B 建立了连接之后，A 会马上发送数据的，一旦 A 发送数据，则很多问题都得到了解决。例如 A 发给 B 的应答丢了，当 A 后续发送的数据到达的时候，B 可以认为这个连接已经建立，或者 B 压根就挂了，A 发送的数据，会报错，说 B 不可达，A 就知道 B 出事情了。 这里“假如A发给B的应答丢了”，如果第三次握手的ack的话，那表述就有问题了，在这时候，A认为连接已建立，而B认为连接还未建立，这个时候A给B发送数据，B会响应rst，而不是认为连接已建立

  2018-12-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b9/f0/cf8e01e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ada

  老师，您好，麻烦图里面可以标注下哪个是A，哪个是B吗？刚开始看有点晕，往回看了几次才懂。还有请教老师，A指的是客户端，B指的是服务端吗？TCP的链接和断开，都是客户端发起的吗？

  作者回复: 发起的是客户端，关闭双方都可以

  2018-12-05

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/ABjAPveWxOuBs3ibbCaBicX7OSibic3prycYG9vOicGHMEv8Vws5o3epykBSFHkbysnaKeMqQaJufINNUncGhmAEomg/132)

  雪人

  老师您好，大概理解的意思是？tcp只不过是在通信的时候维护了一个时间，当一端主动发送消息之后，经过几次重发，一段时间之内，如果一直接收不到另一端的回复，是不是就认为已经断开连接了

  2018-12-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/c1/93031a2a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Aaaaaaaaaaayou

  四次挥手最后那一下如果每次重传b都收不到，然后a到了时间还是会closed呀，那b怎么办呢

  2018-11-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/37/55/9a7781d7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的大象

  请教一下，为什么我用socket实现机器间的tcp通信，A端和B端的初始序号都为0呢？

  作者回复: 抓包抓的？

  2018-10-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c0/95/6f0aad03.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  loveluckystar

  有一次系统出现问题，server端出现了大量的close_wait状态的连接，这是为什么呢？如果根据状态机来看，按理说CLOSE_WAIT只会是一个中间状态，为什么会有大量这个状态的连接呢？求解。

  2018-10-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b4/1d/7d8115ec.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王雅光

  老师，您好，有个问题，无法从传输链接中读取或写入数据，你的主机终止了一个链接！像这种情况发生的原因是什么！谢谢！

  2018-09-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/84/b0/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  小雨青青

  不是太理解TCP基于字节流，tcp报文不是也有报头吗？

  作者回复: 对的，但是写程序的意识不到，一直往里写就可以了

  2018-09-21

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eo6TmyGF3wMIRLx3lPWOlBWusQCxyianFvZvWeW6hYCABLqEow3p7tGc6XgnqUPVvf6Cbj2KUYQIiag/132)

  孙健波

  CLOSE_WAIT 说明这一端还有数据要发没发完，另一端就算调用了CLOSE方法，也照样可以READ

  2018-09-18

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Tom

  状态机图FIN_WAIT1直接到TIME_WAIT转换动作是什么

  2018-09-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5a/31/14f00b1c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  燃

  tcp所能做的流控是很有限的，特别是在当下大数据场景，它的上限就是网卡缓冲。所以，在应用层还需要做更宏观更有效的流控

  2018-08-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/01/87/c45e7044.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  崔朝普🐳📡

  a三次握手和b建立连接后，和第一个正式数据包发送间隔多长时间

  2018-07-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ba/f4/0dda3069.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小白

  老师，四次挥手不是很理解，比如FIN_WAIT_1 FIN_WAIT_2。这俩具体是在干嘛？等待期吗？

  2018-07-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/06/81/28418795.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  衣申人

  老师您好。请问两个问题， tcp有重传机制，那么如果网络不可达，会一直重传何时终止不传了？ 客户端和服务器建立连接后，如果没用keeplive也没有在应用级别检测连接活性，一方宕机，对于另一方，连接会一直保持直到永远？？

  2018-07-14

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83ep8ibEQqN1Slfh9Vg0YJcXcico7NKfl9evCeMpNKZCVE2KWtz3f404LejIsGFFzcubpW5WvhC9ibYevQ/132)

  子榕

  老师你好，请问服务器主动断开是什么流程？我现在的client有大量的close_wait。 还有个问题，哪些状态是不适合长时间占用的，一般怎么解决，比如有大量的time_wait合适吗？会影响新连接使用端口吗，也就是新链接会主动把time_wait端口抢占回来吗？同理close_wait怎么处理？

  2018-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8a/5f/c60d0ffe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  硅谷居士

  wireshark 抓包，是有三次挥手端断开连接的，供参考

  2018-07-08

  **

  **

- ![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,m_fill,h_34,w_34-1662307433379-5904.jpeg)

  eason2017

  老师，您好。我看书中说的是计数器的数值每4微妙加1的。

  作者回复: 赞

  2018-07-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e9/91/4219d305.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC11%E8%AE%B2%20_%20TCP%E5%8D%8F%E8%AE%AE%EF%BC%88%E4%B8%8A%EF%BC%89%EF%BC%9A%E5%9B%A0%E6%80%A7%E6%81%B6%E8%80%8C%E5%A4%8D%E6%9D%82%EF%BC%8C%E5%85%88%E6%81%B6%E5%90%8E%E5%96%84%E5%8F%8D%E8%BD%BB%E6%9D%BE.resource/resize,w_14.png)

  初学者

  对初始序列号为什么不能一直从1开始还有一些疑问，从安全角度考虑我能理解。但从client重连角度，如果client的localIP和localPort和原来不同，相当于是新的链接，和原来的网络包应该没关联的，从1开始是不应该有问题的。如果重连复用了本地IP和Port，就算初始序列号是随机且随时间递增的，假如开始发了1 2 3 4 5五个包，重连后初始序列号不可能是从2或3或4开始吗，如果可能，应该还是有问题的吧，期待老师的回答。

  2018-06-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4d/a0/af1f53f0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  A_吖我去

  四次挥手是不是可以做成三次，把第二步和第三步的ACK和FIN一起发

  作者回复: 里面说了原因了

  2018-06-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f6/32/358f9411.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  梦想启动的蜗牛

  挺不错的👍🏻

  2018-06-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/58/33/9f68bcde.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  贺文杰

  还有一个异常情况就是，B 超过了 2MSL 的时间，依然没有收到它发的 FIN 的 ACK，怎么办呢？... 老师，这段话中A由于经过2msl时间没有收到B发出的FIN，应该认为B已经收到了最后发的ACK应答了，A应该已经关闭了连接，再收到FIN为什么还会有rst应答呢？

  作者回复: 如果已经有了新的在监听

  2018-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/58/e8/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  飓风

  “还有一个异常情况就是，B 超过了 2MSL 的时间，依然没有收到它发的 FIN 的 ACK，怎么办呢？按照 TCP 的原理，B 当然还会重发 FIN，这个时候 A 再收到这个包之后，A 就表示，我已经在这里等了这么长时间了，已经仁至义尽了，之后的我就都不认了，于是就直接发送 RST，B 就知道 A 早就跑了。”这段中如果超过2MSL,B继续重发FIN,此时A收到但是不认这个包却又发送一个RST上面提到的重新连接？这样岂不是又要和B建立连接了？B又是从中怎么知道A早就跑路了呢？这里有点迷惑了！

  作者回复: 如果有了新的在监听

  2018-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/e2/1fad12eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张洪阆

  tcp有校验了，应用层的校验是否可以免了

  2018-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/87/0a045ddf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Carter

  超哥好，啥时候能详解一下网关、路由器以及防火墙吗

  2018-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/e2/1fad12eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张洪阆

  还有物联网应用中，通过gsm上网。tcp协议有自动心跳包，可是业务层面也必须发心跳，不发就会被网络踢掉。为什么业务也要发心跳呢？和tcp心跳有什么区别？

  2018-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/77/59/8bb1f879.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  涛声依旧

  实际过程中，一端clostwait比较多，另一端timewait比较多，这个在图中不太明显，那这个具体是哪端的问题？

  2018-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咖啡猫口里的咖啡猫🐱

  TCP的重传机制，导致业余需要去重，这种解决思路

  作者回复: tcp重传不需要业务去重的

  2018-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/2a/37/130d3a7e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Kobe Bryant 24

  丢包问题，连接维护，流量控制，拥塞控制，这几个问题会详细讲解嘛？

  作者回复: 会，不还有tcp下的吗

  2018-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/2f/25/d2162fce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  龚极客

  关于双保险那里，A可以直接起一个进程替换原来的端口，并没有等待2msl时间啊。

  作者回复: 是的，reuse

  2018-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/06/5fe5900a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  24es鱼

  沙发

  2018-06-11
