# 第5讲 \| 从物理层到MAC层：如何在宿舍里自己组网玩联机游戏？

<audio><source src="https://static001.geekbang.org/resource/audio/c0/4e/c01745571018a2a0dd62c5fa269d644e.mp3" type="audio/mpeg"></audio>

上一节，我们见证了IP地址的诞生，或者说是整个操作系统的诞生。一旦机器有了IP，就可以在网络的环境里和其他的机器展开沟通了。

故事就从我的大学宿舍开始讲起吧。作为一个八零后，我要暴露年龄了。

我们宿舍四个人，大一的时候学校不让上网，不给开通网络。但是，宿舍有一个人比较有钱，率先买了一台电脑。那买了电脑干什么呢？

首先，有单机游戏可以打，比如说《拳皇》。两个人用一个键盘，照样打得火热。后来有第二个人买了电脑，那两台电脑能不能连接起来呢？你会说，当然能啊，买个路由器不就行了。

现在一台家用路由器非常便宜，一百多块的事情。那时候路由器绝对是奢侈品。一直到大四，我们宿舍都没有买路由器。可能是因为那时候技术没有现在这么发达，导致我对网络技术的认知是逐渐深入的，而且每一层都是实实在在接触到的。

## 第一层（物理层）

使用路由器，是在第三层上。我们先从第一层物理层开始说。

物理层能折腾啥？现在的同学可能想不到，我们当时去学校配电脑的地方买网线，卖网线的师傅都会问，你的网线是要电脑连电脑啊，还是电脑连网口啊？

我们要的是电脑连电脑。这种方式就是一根网线，有两个头。一头插在一台电脑的网卡上，另一头插在另一台电脑的网卡上。但是在当时，普通的网线这样是通不了的，所以水晶头要做交叉线，用的就是所谓的**1－3**、**2－6交叉接法**。

<!-- [[[read_end]]] -->

水晶头的第1、2和第3、6脚，它们分别起着收、发信号的作用。将一端的1号和3号线、2号和6号线互换一下位置，就能够在物理层实现一端发送的信号，另一端能收到。

当然电脑连电脑，除了网线要交叉，还需要配置这两台电脑的IP地址、子网掩码和默认网关。这三个概念上一节详细描述过了。要想两台电脑能够通信，这三项必须配置成为一个网络，可以一个是192.168.0.1/24，另一个是192.168.0.2/24，否则是不通的。

这里我想问你一个问题，两台电脑之间的网络包，包含MAC层吗？当然包含，要完整。IP层要封装了MAC层才能将包放入物理层。

到此为止，两台电脑已经构成了一个最小的**局域网**，也即**LAN。**可以玩联机局域网游戏啦！

等到第三个哥们也买了一台电脑，怎么把三台电脑连在一起呢？

先别说交换机，当时交换机也贵。有一个叫做**Hub**的东西，也就是**集线器**。这种设备有多个口，可以将宿舍里的多台电脑连接起来。但是，和交换机不同，集线器没有大脑，它完全在物理层工作。它会将自己收到的每一个字节，都复制到其他端口上去。这是第一层物理层联通的方案。

## 第二层（数据链路层）

你可能已经发现问题了。Hub采取的是广播的模式，如果每一台电脑发出的包，宿舍的每个电脑都能收到，那就麻烦了。这就需要解决几个问题：

1. 这个包是发给谁的？谁应该接收？
2. 大家都在发，会不会产生混乱？有没有谁先发、谁后发的规则？
3. 如果发送的时候出现了错误，怎么办？

<!-- -->

这几个问题，都是第二层，数据链路层，也即MAC层要解决的问题。**MAC**的全称是**Medium Access Control**，即**媒体访问控制。**控制什么呢？其实就是控制在往媒体上发数据的时候，谁先发、谁后发的问题。防止发生混乱。这解决的是第二个问题。这个问题中的规则，学名叫**多路访问**。有很多算法可以解决这个问题。就像车管所管束马路上跑的车，能想的办法都想过了。

比如接下来这三种方式：

- 方式一：分多个车道。每个车一个车道，你走你的，我走我的。这在计算机网络里叫作**信道划分；**

- 方式二：今天单号出行，明天双号出行，轮着来。这在计算机网络里叫作**轮流协议；**

- 方式三：不管三七二十一，有事儿先出门，发现特堵，就回去。错过高峰再出。我们叫作**随机接入协议。**著名的以太网，用的就是这个方式。


<!-- -->

解决了第二个问题，就是解决了媒体接入控制的问题，MAC的问题也就解决好了。这和MAC地址没什么关系。

接下来要解决第一个问题：发给谁，谁接收？这里用到一个物理地址，叫作**链路层地址。**但是因为第二层主要解决媒体接入控制的问题，所以它常被称为**MAC地址**。

解决第一个问题就牵扯到第二层的网络包**格式**。对于以太网，第二层的最开始，就是目标的MAC地址和源的MAC地址。

![](<https://static001.geekbang.org/resource/image/80/41/8072e4885b0cbc6cb5384ea84d487e41.jpg?wh=2443*601>)

接下来是**类型**，大部分的类型是IP数据包，然后IP里面包含TCP、UDP，以及HTTP等，这都是里层封装的事情。

有了这个目标MAC地址，数据包在链路上广播，MAC的网卡才能发现，这个包是给它的。MAC的网卡把包收进来，然后打开IP包，发现IP地址也是自己的，再打开TCP包，发现端口是自己，也就是80，而nginx就是监听80。

于是将请求提交给nginx，nginx返回一个网页。然后将网页需要发回请求的机器。然后层层封装，最后到MAC层。因为来的时候有源MAC地址，返回的时候，源MAC就变成了目标MAC，再返给请求的机器。

对于以太网，第二层的最后面是**CRC**，也就是**循环冗余检测**。通过XOR异或的算法，来计算整个包是否在发送的过程中出现了错误，主要解决第三个问题。

这里还有一个没有解决的问题，当源机器知道目标机器的时候，可以将目标地址放入包里面，如果不知道呢？一个广播的网络里面接入了N台机器，我怎么知道每个MAC地址是谁呢？这就是**ARP协议**，也就是已知IP地址，求MAC地址的协议。

![](<https://static001.geekbang.org/resource/image/56/37/561324a275460a4abbc15e73a476e037.jpg?wh=2323*1951>)

在一个局域网里面，当知道了IP地址，不知道MAC怎么办呢？靠“吼”。

![](<https://static001.geekbang.org/resource/image/48/ad/485b5902066131de547acbcf3579c4ad.jpg?wh=2623*2203>)

广而告之，发送一个广播包，谁是这个IP谁来回答。具体询问和回答的报文就像下面这样：

![](<https://static001.geekbang.org/resource/image/1f/9b/1f7cfe6046c5df606cfbb6bb6c7f899b.jpg?wh=2078*694>)

为了避免每次都用ARP请求，机器本地也会进行ARP缓存。当然机器会不断地上线下线，IP也可能会变，所以ARP的MAC地址缓存过一段时间就会过期。

## 局域网

好了，至此我们宿舍四个电脑就组成了一个局域网。用Hub连接起来，就可以玩局域网版的《魔兽争霸》了。

![](<https://static001.geekbang.org/resource/image/33/ac/33d180e376439ca10e3f126eb2e36bac.jpg?wh=390*590>)

打开游戏，进入“局域网选项”，选择一张地图，点击“创建游戏”，就可以进入这张地图的房间中。等同一个局域网里的其他小伙伴加入后，游戏就可以开始了。

这种组网的方法，对一个宿舍来说没有问题，但是一旦机器数目增多，问题就出现了。因为Hub是广播的，不管某个接口是否需要，所有的Bit都会被发送出去，然后让主机来判断是不是需要。这种方式路上的车少就没问题，车一多，产生冲突的概率就提高了。而且把不需要的包转发过去，纯属浪费。看来Hub这种不管三七二十一都转发的设备是不行了，需要点儿智能的。因为每个口都只连接一台电脑，这台电脑又不怎么换IP和MAC地址，只要记住这台电脑的MAC地址，如果目标MAC地址不是这台电脑的，这个口就不用转发了。

谁能知道目标MAC地址是否就是连接某个口的电脑的MAC地址呢？这就需要一个能把MAC头拿下来，检查一下目标MAC地址，然后根据策略转发的设备，按第二节课中讲过的，这个设备显然是个二层设备，我们称为**交换机**。

交换机怎么知道每个口的电脑的MAC地址呢？这需要交换机会学习。

一台MAC1电脑将一个包发送给另一台MAC2电脑，当这个包到达交换机的时候，一开始交换机也不知道MAC2的电脑在哪个口，所以没办法，它只能将包转发给除了来的那个口之外的其他所有的口。但是，这个时候，交换机会干一件非常聪明的事情，就是交换机会记住，MAC1是来自一个明确的口。以后有包的目的地址是MAC1的，直接发送到这个口就可以了。

当交换机作为一个关卡一样，过了一段时间之后，就有了整个网络的一个结构了，这个时候，基本上不用广播了，全部可以准确转发。当然，每个机器的IP地址会变，所在的口也会变，因而交换机上的学习的结果，我们称为**转发表**，是有一个过期时间的。

有了交换机，一般来说，你接个几十台、上百台机器打游戏，应该没啥问题。你可以组个战队了。能上网了，就可以玩网游了。

## 小结

好了，今天的内容差不多了，我们来总结一下，有三个重点需要你记住：

第一，MAC层是用来解决多路访问的堵车问题的；

第二，ARP是通过吼的方式来寻找目标MAC地址的，吼完之后记住一段时间，这个叫作缓存；

第三，交换机是有MAC地址学习能力的，学完了它就知道谁在哪儿了，不用广播了。

最后，给你留两个思考题吧。

1. 在二层中我们讲了ARP协议，即已知IP地址求MAC；还有一种RARP协议，即已知MAC求IP的，你知道它可以用来干什么吗？
2. 如果一个局域网里面有多个交换机，ARP广播的模式会出现什么问题呢？

<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(159)

- ![img](https://static001.geekbang.org/account/avatar/00/10/d6/42/bafec9b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  盖

  ARP广播时，交换机会将一个端口收到的包转发到其它所有的端口上。 比如数据包经过交换机A到达交换机B，交换机B又将包复制为多份广播出去。 如果整个局域网存在一个环路，使得数据包又重新回到了最开始的交换机A，这个包又会被A再次复制多份广播出去。 如此循环，数据包会不停得转发，而且越来越多，最终占满带宽，或者使解析协议的硬件过载，行成广播风暴。

  作者回复: 赞

  2018-05-29

  **11

  **696

- ![img](https://static001.geekbang.org/account/avatar/00/10/d6/42/bafec9b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  盖

  之前有无盘工作站，即没有硬盘的机器，无法持久化ip地址到本地，但有网卡，所以可以用RARP协议来获取IP地址。RARP可以用于局域网管理员想指定机器IP（与机器绑定，不可变），又不想每台机器去设置静态IP的情况，可以在RARP服务器上配置MAC和IP对应的ARP表，不过获取每台机器的MAC地址，好像也挺麻烦的。这个协议现在应该用得不多了吧，都用BOOTP或者DHCP了。

  作者回复: 对的，赞

  2018-05-29

  **4

  **301

- ![img](https://static001.geekbang.org/account/avatar/00/11/6a/06/66831563.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阡陌

  不得不说，看留言也能学到很多东西

  作者回复: 高手还是很多的

  2018-05-30

  **3

  **201

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/8f/466f880d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没心没肺

  Hub： 1.一个广播域，一个冲突域。 2.传输数据的过程中易产生冲突，带宽利用率不高 Switch： 1.在划分vlan的前提下可以实现多个广播域，每个接口都是一个单独的冲突域 2.通过自我学习的方法可以构建出CAM表，并基于CAM进行转发数据。 3.支持生成树算法。可以构建出物理有环，逻辑无环的网络，网络冗余和数据传输效率都甩Hub好几条街。SW是目前组网的基本设备之一。

  作者回复: 赞

  2018-05-28

  **5

  **150

- ![img](https://static001.geekbang.org/account/avatar/00/11/47/90/6828af58.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  偷代码的bug农

  一直坚持看到第五讲，我理解能力太差了，感觉还是一头雾水唉……

  2018-05-29

  **16

  **115

- ![img](https://static001.geekbang.org/account/avatar/00/11/69/df/71563d52.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  戴劼 DAI JIE🤪

  当年上课学习记住了交叉线和直连线的区别，工作后有一次两台机器对拷，发现网卡能自适应直连线，懵逼了。

  作者回复: 是的，现在自适应了

  2018-06-07

  **

  **68

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/00/51f09059.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  hujunr

  当时用的交换机，把一条网线的2端同时接到交换机了，结果所有电脑都连不上网了，这是为什么？

  作者回复: arp广播塞满了

  2018-07-30

  **8

  **58

- ![img](https://static001.geekbang.org/account/avatar/00/16/d2/f7/fabad335.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天边的一只鱼

  看了前几章，个人理解下访问外网ip的流程，不知道对不对，  我现在在公司的内网想要访问一个北京的外网ip， 首先把我自己的ip地址，mac地址，端口，外网的ip地址，端口，在内网吼一下，被公司网关收到，判断下这个ip是不是内网的， 不是的话，添加上公司自己的mac地址，然后往更上一层吼一下(某个区域电信的网关)，然后这个区域的电信网管判断下ip是不是我这一片的,再试再加上自己的mac地址，再层层往上吼，一直找到这个ip为止。   不知道这么裂解对不对，刘老师。

  作者回复: 对的

  2019-04-12

  **15

  **46

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/45/cdfe6842.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Z3

  当年玩魔兽经常出现他建房我看不见，我建房他能看见之类的问题。 这些可能是应用层的问题吗？

  作者回复: 这个，场景不在了，很难分析

  2018-05-30

  **4

  **34

- ![img](https://static001.geekbang.org/account/avatar/00/12/6a/03/cb597311.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  远心

  用网线直接连接两台计算机的方式，如何知道另一台计算机的 MAC  地址？使用 ARP 协议吗？也就是说其实每一台计算机都安装着 ARP Client/Server 吗？

  作者回复: 是arp，内核里面就有这部分逻辑

  2018-09-15

  **

  **31

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/ea/10661bdc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinsu

  是不是可以理解成交换机是具备学习功能的hub

  2018-08-25

  **

  **25

- ![img](https://static001.geekbang.org/account/avatar/00/12/e9/29/629d9bb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天王

  网络分层第一层物理层，第二层数据链路层。物理层提供设备，即数据传输的通路，将多台设备连接起来。设备如网线，接线头，hub等。数据链路层，即mac层，medium access controller，媒体访问控制，需要解决几个问题1谁先发谁后发，2发给谁，谁接收，3 发送出现了错误，怎么办。1 .1信道划分，各走各的道 1.2 轮流 1.3 ， 随机接入协议，错开高峰，不行就等。2.1 mac地址即数据链路层地址，第二层数据包格式，依次是mac地址，目标mac，源mac，数据类型比如ip数据包，数据包在链路层上广播，目标mac的机器会找到，去掉mac包，看下ip是自己的，则认为是发给自己的，再打开tcp包，发现端口是80，会交给nginx，让nginx处理。第二层最后面是CRC，循环冗余检测，来计算发送过程中是否发生错误。现在有ip，没mac，需要将mac放入数据包。需要ARP协议，通过ip找mac，1 查找本地ARP表，有的直接返回，2 广播ARP请求，靠吼，3等着应答。hub是广播的，什么都会发出去，让主机判断，需不需要。交换机比较智能，他有记忆功能，一开始不知道哪个mac对应哪台电脑，一开始会都广播，后来有应答之后，会记住哪台电脑，以后就直接发到那台电脑上。 - [ ] 

  2019-03-05

  **

  **24

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/14/06eff9a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jerry银银

  文章开头说到：一旦机器有了 IP，就可以在网络的环境里和其他的机器展开沟通了。 我有个疑问：机器没有ip，就不能和其它机器通信了吗？

  作者回复: 不能呀，至少不能通过TCP/IP协议栈进行通信，其他通信方式不在这门课讨论之列

  2019-05-15

  **

  **17

- ![img](https://static001.geekbang.org/account/avatar/00/10/fc/75/af67e5cc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑猫紧张

  ARP协议是在局域网中的，那么在公网中二层如何获得目标mac地址？

  作者回复: 公网也是一个个的局域网组成的呀

  2018-12-01

  **4

  **19

- ![img](https://static001.geekbang.org/account/avatar/00/17/85/0a/e564e572.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  N_H

  老师，根据你前面的几个讲解，我理解到，机器A知道机器B的ip无法准确进行通信，因为ip可以在局域网内进行分配。但是知道mac地址肯定是能进行通信的，因为mac地址是唯一的，无论这个机器到哪里去了，我都能通过mac地址找到这个机器，既然这样，那为什么还需要ip地址？ 这是我看了几期课程以来一直的疑问。

  作者回复: mac是局域网的定位，ip是跨网络的定位

  2019-06-28

  **7

  **17

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8d/a6/22c37c91.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  楊_宵夜

  连续看了好几篇！真的是大道至简！逻辑无比清晰！细节的深度刚刚好！赞！以太网也是个很有趣的东西，记得在知乎上看到过，最离奇的Bug都有哪些。其中一个关于以太网的是说，某个大学因为建筑施工，挖掘机的声波频率影响了光纤电缆中的信号传输，导致信号最多只能传输520多公里哈哈哈，也就是说520公里以外的服务器都访问不到了。 这里细节肯定记错了，抛个砖，有兴趣的各位可以在知乎搜搜😂😂

  作者回复: 谢谢啦

  2018-06-08

  **

  **18

- ![img](https://static001.geekbang.org/account/avatar/00/12/df/d7/ee3da208.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Yuki

  非科班出身，理解很困难。但每天坚持看一篇，多看看一定能慢慢理解～

  作者回复: 可以几天看一篇的，慢慢消化，不图快

  2019-06-13

  **

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/11/3c/fb/ac114c4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  高磊

  局域网玩的是魔兽争霸，不是魔兽世界

  作者回复: 哈哈

  2019-04-11

  **3

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/52/19553613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘培培

  ARAP 协议：https://tools.ietf.org/html/rfc903 因为太难实现，被后来的 BOOTP 协议：https://tools.ietf.org/html/rfc951 和 DHCP 协议：https://tools.ietf.org/html/rfc2131  替代。

  作者回复: 牛

  2019-08-26

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/11/e9/86/d34800a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  heyman

  "这里还有一个没有解决的问题，当源机器知道目标机器的时候，可以将目标地址放入包里面，如果不知道呢？一个广播的网络里面接入了 N 台机器，我怎么知道每个 MAC 地址是谁呢？这就是ARP 协议，也就是已知 IP 地址，求 MAC 地址的协议。" 请问一下，机器a是怎么知道机器b的ip？

  作者回复: 是人知道的。从a机器上，访问b机器上的网站，不得输入一个b机器的地址呀。就是访问163，也有个地址呀，是人知道地址，告诉机器的

  2019-05-23

  **5

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ed/4d/1d1a1a00.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  magict4

  两台电脑直连的情况下，谁是网关呢？电脑可以充当网关吗？

  作者回复: 不用配置网关就能通

  2018-06-18

  **9

  **11

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  天天向⬆️

  老师，一直没明白你说的mac层是什么？是物理层吗？还是数据链路层？

  作者回复: 链路层，后面也解释了尴尬的叫法，但是约定俗成

  2018-07-12

  **

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/d7/454a1b90.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  leon

  文中提到ARP缓存和交换机转发表都有过期时间，这个时间设置多长如何取呢？当arp缓存或转发表里的内容不对时有办法知道吗？

  作者回复: 内容不对就连不上，报icmp，会刷新

  2019-05-06

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fa/33/0eb31e67.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Amorfati

  \1. rarp 除了通过mac地址去向dhcp之类服务要一个IP，其他没有想到有什么用，按照我的理解，如果非直联同一交换机，一个设备去找另一个设备，知道其mac地址而不知其ip，说不通 2. 多个交换机，若pc1所连交换机为A，pc2,pc3所连交换机为B，AB直连，pc1找pc2的时候A会把直连B的端口缓存为pc2的mac地址，在过期时间内，pc1都没有办法找到pc3 这个就是我的理解，如有误还请老师指正

  2018-05-28

  **2

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/14/17/c8/ca17d37e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夕夕熊

  我已经搞糊涂了 “IP 层要封装了 MAC 层才能将包放入物理层”到底是MAC层封装IP层  还是IP层封装MAC层 。在前几节我接触的应该是网络包经过到达MAC层之后 已经包含了IP层  也就是说MAC层会封装IP层的吧。

  作者回复: MAC在外，IP在内，语言有歧义了

  2019-02-20

  **2

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/52/bb/225e70a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  hunterlodge

  交换机学习后如果把两个网口的线互换会出现什么情况呢？交换机能识别出错误更新配置吗？

  作者回复: 会有超时时间，也会重新学习

  2018-06-15

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/11/10/4f/ac326acf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  绿豆先生

  ip更换时，如果没到过期时间，交换机如何感知呢？

  作者回复: 会短暂无法连接，交换机会刷新

  2019-06-16

  **

  **6

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epyb06M55D5wXsDrsObicnNJtph90rsuKug7KYOJWleyGJrltDA2PAkp3jRAzIxFfUbnA96ic5TibLTQ/132)

  MrGuo

  老师  mac缓存这些有个有效时间 如果有效时间还没到  但是局域网有其他设备改变了ip mac，这时缓存的mac不就不对了吗

  作者回复: 是的，这个时候会发送不成功，会刷新

  2018-11-23

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/12/7b/57/a9b04544.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  QQ怪

  原来集线器和交换机是有区别的啊！之前一直以为是一个东西！

  作者回复: 有的

  2019-03-10

  **2

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/0f/df/38/3552c2ae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  匿名用户

  为了避免每次都用 ARP 请求，机器本地也会进行 ARP 缓存。当然机器会不断地上线下线，IP 也可能会变，所以 ARP 的 MAC 地址缓存过一段时间就会过期。这就是路由表存在的意义？这里应该提一下啊…

  2018-06-03

  **1

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/52/19553613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘培培

  \1. RARP 用来主动获取 IP 地址，不过现在都用 DHCP 了

  作者回复: 对的

  2019-08-26

  **

  **3

- ![img](https://wx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKhEe47WkwPmOR7ggcsTWyGZlB4lsK2iclspeb1D0OQ3IibHEEPpdo9a4nuyXbrPC2MWGFIz3yCJmyg/132)

  Geek_77971a

  请问arp协议既然是二层协议，也就是ip与mac的对应关系，但是好像用不到交换机，arp协议的处理流程是在主机进行的，也就是说用hub将主机连接起来就可以实现arp查询mac地址。

  2019-03-19

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ec/81/2127e215.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  梦醒十分

  看到现在了，感觉还是没进入状态，或许这种知识不好说清楚吧！一头雾水！

  作者回复: 多看几遍就好啦

  2019-03-03

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d8/0a/9c734a9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  潇xiao

  老师写的挺好的，我正在准备考研(还有不到一个月)现在回过头来看思路更清晰了，最主要的是通俗易懂，不像书本晦涩难懂。而且回看觉得学到了新的东西，很赞

  2018-11-26

  **

  **3

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/4wiaKDwz4YI8B68hLd7qSa6SrIOqkBo67Y7gcIFevGHgaAKzbK5PoXcIJJhrmkzKyWvWOkYcMs66iav5EVbHZ9ag/132)

  Geek__ILoveStudyAndWork

  关于文中说将网线一头里面的的将1和3线，2和6线换位置 一开始我是比较好奇为啥要这样换，我觉得换1，3就好了，为啥2，4也要换，后面搜了资料发现其实1，2一般情况下是绞在一起的 代表收，3和6是绞在一起的，代表发，要换就一起换的 后面又发现其实换完之后刚好是一个新的接口标准 还有好多细节，感兴趣的可以看一个总结，里面包括了水晶头里面的线的顺序已经直通线和交叉线是怎么交互的 https://blog.csdn.net/lingfy1234/article/details/121310755

  2021-11-14

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/53/d4/2ed767ea.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wmg

  老师，lvs的dr模式，rs向客户端的回报会不会刷新交换机里vip对应的mac地址，这样会不会导致网络出错，有什么解决办法吗？

  作者回复: 单臂模式的话，应该是两台交换机了

  2019-08-13

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/d5/8c/1a6cc556.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Y

  作者说ip层封装了mac层才能被物理层接受，这个不对吧，封包是从上而下，解包是从下而上

  作者回复: 意思是发出去的时候，ip层外面一定要带mac层，才能发送到物理层，才能被目标机器的物理层接受

  2019-07-19

  **3

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/ea/10661bdc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinsu

  如果一个局域网里面有多个交换机，ARP 广播的模式会出现环路问题吧

  作者回复: 会的

  2019-04-29

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/21/02/fff4409b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  采菇凉的小🍄

  能说一下XOR异或具体操作什么吗？以及如何通过操作进行检验的

  2018-10-26

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/ba/f4/0dda3069.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小白

  看课好几遍，终于明白啦

  2018-07-20

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/90/167a13fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  徐良红

  rarp现网似乎很少见，理论上应该是根据mac去查询对应的ip。 一个局域网如果没有划分vlan，广播报文会传播到整个网内所有主机。 请老师指正

  2018-05-28

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/ec/2e/49d13bd2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  SMTCode

  RARP：知道IP查询MAC 第二个问题会产生网络风暴吗？arp攻击？

  2018-05-28

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/d5/77/d9c1f012.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云追月

  讲故事是好的，帮助理解。但是讲故事不是重点啊，故事背后的体系化知识讲解才是重点啊。课程看了这么久，总感觉是在给具有网络知识背景的人讲故事，没有背景的人，读起来很费劲，面对时不时冒出来专业名词，很困惑

  2020-06-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/1d/56/3c/574475ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  redtax

  请问刘老师： 如果一个交换机的A端口，连接了一个HUB，HUB的其他口又连了多台设备。那么HUB一端的每一个设备在往外发送数据包时，MAC地址会统一都记录在交换机的这一个端口上吗？有数量限制吗？还是会轮询替换呢？

  2020-05-21

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  交换机为 第二层 数据链接层，在演化的过程中 开始是hub的作为接受和发送通道。

  2018-12-14

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/91/14/946c2aa6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  young

  感觉场景可以描述的更完整些。比如Arp场景的之一是用户手动操作访问指定的ip地址，接下来才是文中描述的过程

  2018-12-14

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  wrzgeek

  网络小白，终于知道交换机和集线器的区别了，每天进步一点点，很开心

  2018-09-16

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c0/71/c83d8b15.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  一个坏人

  老师好，请问arp"吼"的原理是？也就是广播的原理。

  2018-08-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/58/97/8e14e7d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  楚人

  老师讲得好，留言也很赞👍，永久保存，常拿出来看看！获益匪浅！

  作者回复: 是的，高手很多的

  2018-06-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5b/b9/d07de7c6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FLOSS

  交换机只有刚才介绍的这一种策略吗？交换机为啥不会主动询问每个口的MAC地址？

  作者回复: 请看下集，多个交换机的场景

  2018-05-28

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/18/4b/d7/f46c6dfd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  William Ning

  老师同学好～ 搜索了下，但是并没有很好解释清楚，“为什么不同网络设备之间的连接是使用直通线而不是交叉线，我觉得任何设备之间的连接都使用交叉线才合理啊？”的问题？ 尽管现在已经做到网卡自适应，不用再区分直通线和交叉线，但是还是有建议，相同设备使用交叉线， 而且路由器和主机，应该算作相同设备，因为网卡只有两种。 还是很困惑～

  2022-05-26

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKWHqcx2zIN5RE913nEQWNZPyxadYVyDicUqRFCy59icxOznbrDvwGvymYpU3tk0s8mHaTYujSYAw4A/132)

  Geek_4932c3

  广播风暴

  2022-05-14

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/HibtYWgbgTKZPTkqEGvB0pIyFfrhiaKrEkWTgPa9EYFsH7VV2an6oXPCvLzqOb1KfsNN8flQuRUWo0WntI5M1iapw/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  tony

  服务端可以根据ip获取mac地址吗？

  2022-03-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/ea/5d/ccb4c205.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  绘世浮夸 つ

  老师，我有个问题，两个电脑通过网线互联了，电脑本身的网络从哪里来啊

  2022-02-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b2/e9/f1541957.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wgl

  时隔多年，我又回来复读了。当时一头雾水，现在大雾逐渐消散。嘿嘿嘿嘿嘿嘿～

  2022-01-09

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/wObTxbxOFRbVjTuicn081m0ISlpRBVyvR7Y58xEyyUXdbjTODIKInmEJmqkU2Iia5BQSM56nFUwkL4iaaribaVcm3w/132)

  Geek小张

  如果听过子网掩码计算出不是属于同一子网内，这个时候需要发送到网关，虽然我们配置了网关ip，但是实际上还没有知道网关的mac地址，是不是网关也得发送arp包询问下mac地址才能发送到网关

  2021-11-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/0c/0f/93d1c8eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mickey

  如果一个局域网里面有多个交换机，ARP 广播的模式会出现什么问题呢？ ==》形成环路会产生广播风暴，使MAC表不稳定，MAC频繁复制。

  2021-11-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/0c/0f/93d1c8eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mickey

  信道划分就是SDM 空分多路复用 轮流协议就是TDM 时分多路复用

  2021-11-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/0c/0f/93d1c8eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mickey

  信道划分就是FDM频分多路复用 轮流协议就是TDM时分多路复用

  2021-11-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/96/36/37b9e314.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Emmcd

  1）网线水晶头接法只能实现局域网中两台主机间的通信。 2）多台主机间通信需要借助集线器（Hub），但 Hub 会将端口收到的数据全部转发到其他端口上去，由主机数据链路层判断是否是发给自己的。 3）交换机通过学习可以将数据从特定的端口转发给目的主机。 4）交换机学习时需要在局域网内进行泛洪，为了减少泛洪流量，又实现了 VLAN，即虚拟局域网，限制了广播域的范围。交换机接收到广播数据帧时，只会将这个数据帧在相同 VLAN 的端口上进行广播。也就是说，同属一个 VLAN 的主机通过数据链路层直接通信，而不同 VLAN 的主机只能通过 IP 路由功能才能实现通信。

  2021-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/27/e1/f4/be25debf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  幸福里

  arp是通过ip获取mac地址的,假如我发送到国外是包 我第一个获取的mac地址是网关mac地址,然后到下一个网关获取下下一个网关mac地址 可是我获取到最后的网关地址时,才能获取到最终的我要访问的ip对应的那个mac地址,中途我要访问的那个IP地址给别的mac使用了,这怎么办,那就不是我一开始要访问的IP地址对应mac设备了

  2021-06-03

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/9f/c0/86febfff.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Master

  数据链路层，二层，主要是用来防止包的发送链路出现混乱，比如a发给b，c却收到，交换机是二层设备，主要也是辅助进行精准转发

  2021-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ec/b0/4e22819f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  syz

  ARP缓存中有ip和mac对应关系吧，再有一个交换机来做这个缓存？还有别的必要吗

  2021-05-27

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/17/fa/05f2cbc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ZZZZZQ7

  关于ARP协议缓存 "当然机器会不断地上线下线，IP 也可能会变，所以 ARP 的 MAC 地址缓存过一段时间就会过期。" 这里缓存过期和IP变更应该不是同步的，那么是不是会存在以下场景: 1. 机器A向192.168.0.100发送数据包，获取到机器B的MAC地址并缓存 2. 机器B下线，归还IP 3. 机器B，C都上线，B分配到了新IP(192.168.0.101)，C则分配到了原来B的IP(192.168.0.100) 假设至此第一步的缓存仍未过期，此时机器A向192.168.0.100发出的数据包应该是带着机器C的IP以及机器B的MAC地址，那么数据包会被先发送到机器B，机器B发现IP地址不是自己的，则通过ARP重新在局域网内获取到机器C的MAC地址，然后将数据包转发给机器C 我的理解正确么？

  2021-05-20

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/8d/fa/93e48f3b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ℡ __ 柚稚

  老师，您好，交换机的转发表，当有一个ip要找mac是先去转发表去找吗，没有的话就arp，那ip地址是会变的，ip变了就会导致转发表和真实不一致，是会拿到错误的mac地址吗，这个转发表缓存过期是怎么个逻辑

  2021-04-27

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/ee/ae/855b7e6e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Gabriel

  1：物理层的含义？ 物理层是计算机网路OSI中最低的一层，也是最基本的一层。简单的说，网络的物理层确保原始的数据可以在各种物理媒体上传输。物流层规定：为传输数据、需要物理链路与设备的创建、维持、拆除，并具有机械的、电子的、功能的、规范的特性。 大体意思就是说，用什么传输，怎么传输？ 就像我们小时候用线，在两个纸杯上做两个听筒。 物理层就是：两个纸杯、传输媒介就是线。 MAC层是什么？ 哈哈，刚刚开始我以为MAC是笔记本的MAC,原来不是。 MAC的英文名，media access control，媒体介入控制层，位于数据链路层下层子层。 它定义了数据帧怎样在介质上进行传输，数据链路层分为上层LLC（逻辑链路控制），下层的MAC（介质访问控制），MAC主要负责控制与链接物理层的物理介质。在发送数据的时候，MAC协议事先判断是否可以发送数据，如果可以发送将给数据加上一些控制信息，最终将数据以及控制信息以及规定的格式发送到物理层，MAC协议首先判断输入信息并是否发生传输错误，如果没有错误，则去掉控制信息发送至LLC层。 https://baike.baidu.com/item/MAC/329671

  2021-04-21

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  张玮吾

  有个疑惑，因为ARP类型的数据包，发送方应该是无法带有目标MAC，那么按照最开始说的网络包格式里需要封装的目标的MAC地址是不是会有问题

  2021-04-16

  **1

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLq1w2nAUjZRWqJBWauiciatglgQoseE9GTia7xiag4F5Npicxybe2yRUS6Xwql5Iic7TmseiaYmMM8bvnqg/132)

  Geek_9d7d91

  数据链路层的类型2bytes能封装那么多东西? 里面具体是什么东西?

  2021-04-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/01/95/fd09e8a8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  布拉姆

  老师，交换机的学习成果叫做转发表。转发表存储的，究竟是MAC-网口的映射？还是MAC-IP地址的映射？因为上文提到“每个机器的 IP 地址会变，所在的口也会变”

  2021-04-03

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c7/05/19c5c255.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  微末凡尘

  总结： 第一层：物理层。主要是指存在的东西，比如两台电脑完连接局域网，其中的网线属于物理层。 第二层：数据链路层。这一层主要是来解决以下几个问题的： 1) 这个包是发给谁的？谁应该接收？ 2) 大家都在发，会不会产生混乱？有没有谁先发、谁后发的规则？ 3) 如果发送的时候出现了错误，怎么办？ MAC 的全称为媒体访问控制，就是用来控制谁先发谁后发的问题，即解决第二个问题。这里涉及几个方式： 方式一：分多个车道。每个车一个车道，你走你的，我走我的。这在计算机网络里叫作信道划分； 方式二：今天单号出行，明天双号出行，轮着来。这在计算机网络里叫作轮流协议； 方式三：不管三七二十一，有事儿先出门，发现特堵，就回去。错过高峰再出。我们叫作随机接入协议。著名的以太网，用的就是这个方式。 这一层会用到一个物理层地址，即链路层地址，也即 MAC 地址。来解决发给谁，谁接受的问题。 根据网络包格式，如下图所示，最后是 CRC 也就是循环冗余检测。通过 XOR 异或的算法，来计算整个包是否在发送的过程中出现了错误，主要解决第三个问题。 交换机是有 MAC 地址学习能力的，学完了它就知道谁在哪儿了，不用广播了。

  2021-03-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/39/67/743128f7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  书木子谢明

  “有了这个目标 MAC 地址，数据包在链路上广播，MAC 的网卡才能发现，这个包是给它的。MAC 的网卡把包收进来，然后打开 IP 包，发现 IP 地址也是自己的”，请教：为什么检验了mac地址，还需要再检验ip 地址呢？

  2021-02-21

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/6d/910b2445.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fastkdm

  有了交换机的转发表，还需要ARP缓存吗

  2021-01-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/15/ee/f461168f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jay

  将一端的 1 号和 3 号线、2 号和 6 号线互换一下位置，就能够在物理层实现一端发送的信号，另一端能收到。（为什么呢？）

  2021-01-27

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/nM9cJvTbKIYdkj7kTF1MffLjJwPr78icUulumxM065HK7QargYDkxxBy9bwxMjc8NuD966mc0jVanAiaKVfVmtMA/132)

  pdw

  已知 IP 地址求 MAC前提条件是在一个局域网内，如果不是在一个局域网内咋办？

  2021-01-12

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/a0/aa6d4ecd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张潇赟

  局域网内出现多个交换机的情况应该是应为机器太多，当一个交换机接口数量满足不了组网需要。这个时候就需要两个或者多个交换机对接。这种情况下对某一个交换机的转发表从数据结构上来看应该和单个交换机没有区别。只是某个接口上记录的mac地址是另一个交换机的。当A交换机下面连的主机a，需要给B交换机的主机b发送数据的时候，第一次通过arp获取的mac地址其实就是B交换机的mac

  2020-12-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9c/82/27527c00.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  window0006

  网线直接连接的两台电脑，网关要要设置为其中一个的IP吗？可以不可写成同网段的其他任意IP？同网段内直接通过广播的方式通信,如果直接连接两个电脑,这时候没有交换机来识别包目的ip是广播地址,这个广播是会让网关发送的吗?

  2020-11-11

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKNDKOCoZvCqoYVM1t97Q77QPLmRBGvOLYzFsh8073RicycoIuwGrIsCXpAFEyVBOxcyE3Ih1mr6Vw/132)

  Geek_bbbda3

  为啥不让二层设备去做arp的事情呢？有没有这样的二层设备

  2020-11-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/41/ba/ae028565.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  YqY

  第一层 物理层 网线 第二层 数据链路层 1.通过链路层地址解决发送和接收对象谁是谁的问题 2.通过信道划分、轮流协议、随机接入协议解决谁先发后发问题 3.通过crc--循环冗余检测解决发送过程中出现的错误 4.如果源机器不知道目标机器的地址的时候，通过arp协议求得。 交换机 交换机比集线器聪明，有转发表，不需要广播，可以准确转发。

  2020-09-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/6c/a8/1922a0f5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郑祖煌

  DHCP是一个没有ip的主机动态的从DHCP Server获取一个动态的可能以后会变化的IP地址。而RARP则是一个没有ip的主机从RARP Server获取一个ip地址，而这个IP地址是直接通过  （）MAC地址->IP地址）  配置的服务器配置上的，除非配置变化否则他的IP地址是不变的。

  2020-08-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/f5/9d/104bb8ea.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek2014

  这个课程，还是需要一定的网络知识

  2020-08-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/1c/d6/57e811df.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  花生糯米团

  太牛了，简直培养了对网络协议的兴趣

  2020-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/cc/52/8be45005.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  叶敏

  把前面四讲看了好几遍，不懂的概念问别人加搜索，理解之后感觉整个都通了。后面看起来也变得容易理解了

  2020-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d6/39/6b45878d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  意无尽

  总结： 第一层：物理层。主要是指存在的东西，比如两台电脑完连接局域网，其中的网线属于物理层。 第二层：数据链路层。这一层主要是来解决以下几个问题的： 1) 这个包是发给谁的？谁应该接收？ 2) 大家都在发，会不会产生混乱？有没有谁先发、谁后发的规则？ 3) 如果发送的时候出现了错误，怎么办？ MAC 的全称为媒体访问控制，就是用来控制谁先发谁后发的问题，即解决第二个问题。这里涉及几个方式： 方式一：分多个车道。每个车一个车道，你走你的，我走我的。这在计算机网络里叫作信道划分； 方式二：今天单号出行，明天双号出行，轮着来。这在计算机网络里叫作轮流协议； 方式三：不管三七二十一，有事儿先出门，发现特堵，就回去。错过高峰再出。我们叫作随机接入协议。著名的以太网，用的就是这个方式。 这一层会用到一个物理层地址，即链路层地址，也即 MAC 地址。来解决发给谁，谁接受的问题。 根据网络包格式，如下图所示，最后是 CRC 也就是循环冗余检测。通过 XOR 异或的算法，来计算整个包是否在发送的过程中出现了错误，主要解决第三个问题。 交换机是有 MAC 地址学习能力的，学完了它就知道谁在哪儿了，不用广播了。

  2020-05-03

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  朱奇

  后悔没早来😁

  2020-04-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  个人觉着,对于第二个问题,可能会出现循环往复的进行发送的问题

  2020-04-21

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eov38ZkwCyNoBdr5drgX0cp2eOGCv7ibkhUIqCvcnFk8FyUIS6K4gHXIXh0fu7TB67jaictdDlic4OwQ/132)

  珠闪闪

  刘老师，我在网上看到水晶头的1、2脚用来发送数据，3，6脚用来接收数据。跟您在文章中写的相反，请问哪个是对的？

  2020-04-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/f9/e0/2cdd1bc9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Gray13

  做了那么多年IT网络一直很烂 不得不说 这个文章真的写得非常有趣味又很好懂 非常喜欢 哈哈

  2020-04-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/68/a8/1fa41264.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  马什么梅

  不知道老师现在还回不回答,数据链路层中的mac地址有两个,一个是目的,一个是源,发送方式有三种,广播,单播,组播. 不知道组播这里的目的地址是怎么填写并且发送的呢

  2020-04-10

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  够细

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/94/0a/7f7c9b25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  宋健

  很有意思！

  2020-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  带着问题去探求真相，让自己更加专注于学习的过程。从MAC层和MAC地址的作用到HUP、交换机的演进，作者的解释能更加深入明白。感谢

  2020-03-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f2/13/3ee5a9b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chenzesam

  有趣哈哈

  2020-03-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_5b0e47

  找到导致大学上不了网，ARP风暴的根本原因了，赞

  2020-03-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/13/e7/6e75012c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谭倩倩_巴扎黑

  那帧头帧尾是干啥的？

  2020-03-12

  **4

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e6/1c/9d3744ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  小李讲源码

  第一个思考题： 通过 IP 找目标地址，就是我们先找到某个小区（IP 地址），然后再联系这个人（MAC 地址） 而通过 MAC 地址，找 IP 地址，有点类似与卫星定位了，确认某个人现在所在的具体位置。 第二个思考题 如果两个交换机构成一个循环的网络，交换机就无法确定 MAC 地址到底是在哪里。

  2020-03-12

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEIaTvOKvUt4WnuSjkBp0tjd6O6vvVyw5fcib3UgZibE8tz2ICbTfkwbzs8MHNMJjV6W2mLjywLsvBibg/132)

  火力全开

  DHCP的四次交互是不是可以减少为三次交互呢？

  2020-03-04

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/34/bb/b28428ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钦

  “分层”可将庞大而复杂的问题，转化为若干较小的局部问题，而这些较小的局部问题就比较易于研究和处理。——《计算机网络》谢希仁

  2020-02-23

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJKj3GbvevFibxwJibTqm16NaE8MXibwDUlnt5tt73KF9WS2uypha2m1Myxic6Q47Zaj2DZOwia3AgicO7Q/132)

  饭

  老师！还有一个问题，有朋友提到，通过集线器局域网，机器A想对机器C发数据包，于是在局域网广播数据包，比如机器B收到广播的数据包，发现不是给自己的，会转发出去。机器D也收到了数据包，也发现不是给自己的，也转发出去。这样一来B又收到D转发的，D收B转发的。继续发现不是自己的，接着又转发，不是死循环了。。这一层MAC层会有验证重复数据包的机制吗

  2020-01-19

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJKj3GbvevFibxwJibTqm16NaE8MXibwDUlnt5tt73KF9WS2uypha2m1Myxic6Q47Zaj2DZOwia3AgicO7Q/132)

  饭

  老师，请教一下，当网关收到数据包，首先检查IP和MAC，发现不是自己的情况下，向上级网络网关转发时，源MAC地址会改成网关自己的吗？如果源MAC改的话，最初发起数据包的源机器MAC那不是被覆盖了？

  2020-01-19

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/20/b9/8e444fed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  SoulEdge

  多台交换机广播风暴呀

  2019-12-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/18/49/b1d864e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Hinimix

  老师，局域网如何只通过mac地址交流呢，不配置三层任何东西，按上一章理解是不能交流对吗?

  2019-12-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/4a/df/a3ee01da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sailor

  1 物理层  集线器 2 数据链路层 Mac ARP广播寻址，随机接入解决多路复用，下一层类型 IP 交换机 Mac地址表 vlan，广播域，广播风暴，生成树

  2019-12-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/a1/95/3234523a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  XXX

  老师您好，上面arp报文的类型是0806，老师不小心打成了8086

  2019-11-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/86/06/72b01bb7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  美美

  广播是ip tcp通信的基石呢，想知道mac靠吼，想知道ip也靠后吼。广播包里面的ip地址和mac地址是约定俗成的吗

  2019-11-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/10/b7974690.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BD

  mac地址是用来在局域网精确查找某台计算机的，解决了每次都需要广播造成网络杜塞的问题 ARP协议是靠吼的方式获取网络中的mac地址的 交换机有mac学习的能力 可以通过缓存的mac直接找到目标机器转发数据，但是缓存表有过期时间

  2019-11-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/69/9a/6eb47718.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BabyT1ger

  我想请教下，mac地址用于同一网段里的通信，ip地址用于跨网段通信，那为什么判断数据包是否发送给本机时，是先判断mac地址是否为本机的，而不是判断ip地址先再判断mac地址

  2019-11-15

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/d4/44/0ec958f4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Eleven

  如果一个局域网里面有多个交换机，ARP 广播的模式会出现什么问题呢？这个问题，我觉得因为多引入了一台交换机，那么交换机和交换机之间的通讯就会带来复杂性

  2019-11-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/d4/44/0ec958f4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Eleven

  在二层中我们讲了 ARP 协议，即已知 IP 地址求 MAC；还有一种 RARP 协议，即已知 MAC 求 IP 的，你知道它可以用来干什么吗？虽然我不知道能干嘛，但是能猜出是用在知道MAC地址求iP的场景，网络小白>o<

  2019-11-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/64/9b/0b578b08.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  J.Smile

  我在使用集线器组网后，使用飞秋发消息，还是会以广播形式经过集线器吧？

  2019-11-14

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTL17rDiannrcyT9gOY8udNhhicXsgxLF1bffdeLuqmOKsaicfg3dricE6NbZl9wGw2IP9p6Rz6wzhzdLA/132)

  行者周

  交换机在学习到目标地址的mac地址之前，也是在进行广播吧。只不过一旦学习到了就不再广播。而hub是即使有了本地缓存，也会在一定时间缓存失效后继续广播。这样理解对吗？

  2019-11-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/38/19/c8d72c61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  木白

  老师你这个分层是用的osi参考模型的分层吧，不是tcp/ip的四层模型吧

  2019-11-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7f/ca/ea85bfdd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  helloworld

  老师，如果直接将两台电脑用网线连接来通讯的话，直接通过MAC地址就行了吧？无需再配置IP地址。老师能解释下嘛，感谢

  2019-10-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/55/cb/1efe460a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  渴望做梦

  老师，IP地址后面的/24是什么意思呢

  2019-10-24

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/4c/f5f95f9b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阳光不锈

  冲突域：信号在介质上传输会产生冲突的区域。 集线器所有端口在同一个冲突域，共享网络带宽 交换机的每个端口在同一个冲突域，独享端口带宽

  2019-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/c9/f9/39492855.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿阳

  老师，你好。数据链路层分为两层，LLC和MAC层，请问能增加关于LLC层的相关知识的讲解吗？

  2019-09-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/52/19553613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘培培

  \2. IP 可能会冲突。

  2019-08-26

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/RVgoywqdiaoTDV66lVJwED3jtAf1OuAal6K1QeWLoeqNbbQPicRz6XhJU1vShUJ50eHLGvXc22ZVX9hnG5ZNKARw/132)

  叶叶

  是不是应该先梳理MAC层的体系结构，再以例子展开介绍比较好？

  作者回复: 这个不用，一般平时也接触不到MAC层，了解原理就可以了

  2019-07-15

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/EhQCxJRge5ehF3j5WuhzTKKEzZA8qOkByOAEVlEj8vtGOy7j3y4q4pnckcnUol6QQckdBQiaDDmOnBmTcZSiaQ1Q/132)

  lilyhuang

  二成交换机基于MAC转发，三层交换机基于IP转发，但是二层可以划vlan也能实现三层交换机的功能，三层交换机也可以只基于MAC转发，那二层交换机和三层交换机各自的优势和意义在哪儿呢？希望老师能解答一下

  作者回复: 能跨vlan的交换机就不是二层交换机，可以叫三层交换机。当然三层交换机功能更丰富啦。但是这里的二层交换机你可以理解成为学术上机械的定义。

  2019-07-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/08/92f42622.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  尔冬橙

  老师，文中有时候提到了端口，比如8台机器连接在同一个交换机或者集线器上，他们监听的端口号可能一致么？

  作者回复: 这里的端口不是端口号，是交换机的port，那个屋里的口

  2019-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/37/d0/26975fba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  aof

  多路访问的第三种方式很像乐观锁的工作原理啊

  作者回复: 融会贯通

  2019-07-10

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIy5ULaodUwsLoPuk1wd22hqXsaBbibNEqXM0kgrCTYDGKYQkZICYEyH9wMj4hyUicuQwHdDuOKRj0g/132)

  辉煌码农

  还是没明白物理层到时是干嘛的啊

  作者回复: Ethernet协议

  2019-06-30

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/24/18/8edc6b97.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Roger

  “这里还有一个没有解决的问题，当源机器知道目标机器的时候，可以将目标地址放入包里面，如果不知道呢？一个广播的网络里面接入了了 N 台机器，我怎么知道每个 MAC 地址是谁呢？这就是ARP 协议，也就是已知 IP 地址，求 MAC 地址的协议。”这里是不是讲反了，这里应该用的是RARP协议，根据MAC地址，找IP地址吧？

  作者回复: 想象一下真实的场景，怎么可能不知道对端的IP还去连接他。

  2019-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/94/3b/d883b5c0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kangeru

  物理层：局域网、集线器，采用广播的方式。 数据链路层：MAC的全称是Medium Access Control，即媒体访问控制。多路访问。我怎么知道每个 MAC 地址是谁呢？这就是ARP 协议。 ARP通过吼的方式知道目标地址。

  2019-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/2a/7c/e0086894.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王酌

  看到这里，真的是把以前学校没想明白的问题都捋顺了

  作者回复: 赞

  2019-05-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/67/09/b907de5c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  今天不卖烤地瓜

  精选留言里也是干货满满啊

  2019-04-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a0/57/3a729755.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灯盖

  写的好，毕竟是一线人员，都是干料

  2019-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/45/f5/b3d7febf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Adolph

  路由器的四个LAN口是不是可以当作一个集线器，一台电脑通过网线连接在LAN1上，另一台电脑也通过网线连接在LAN2上，这两台电脑是不是可以互相访问了。

  作者回复: 是的，但是不是集线器

  2019-04-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/93/4a/de82f373.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  AICC

  没有过这样的操作，理解上有点不确定，文中提到的水晶头的第 1、2 和第 3、6 脚，它们分别起着收、发信号的作用。将一端的1号线和3号线、2号和6号线互换一下位置，就能够在物理层实现一端发送的信号，另一端能收到 也就是一条线的两端本来是都能收和发，现在改造后就是一端全是发，另一端全是收？ 按这个操作换好后，一端是1，2，1，2号线起着收信号，另一端就是3，6，3，6号线起着发信号

  作者回复: 现在不用这样切换了

  2019-03-13

  **2

  **

- ![img](https://wx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJnfEgoyUEPw8dvMVz42K1nt1YIoOt5dIH8onNE1db4N4ViaOFjv8uPxkguCJ414IyYILqpkeRAzfA/132)

  Geek_ab2b44

  大赞。这节课学习很多，清楚了交换机和集线器的区别，以前学校学的只是死记硬背，没有理解。 交换机会学习，在mac层工作，维护mac表。 集线器无脑，只是简单的复制转发，好比广播。 获取目标主机的mac地址通过Arp协议，即“吼”，维护ARP缓存表。

  2019-02-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/5e/7c/94af3f5e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  条

  老师，arp广播塞满了是什么原因，是异常状况吗?

  作者回复: 试试看，啥都发不出去了，tcpdump的时候，N多arp在跑，永远不结束。

  2019-02-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/94/2c22bd4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  克里斯

  主机发送一个255.255.255.255数据包，如何做到在局域网中广播给所有人？他把数据包发给交换机，交换机将发现没有mac地址，将此包广播给其他主机。

  2019-01-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/a5/40/ad00a484.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  \- -

  维基百科和百度百科中显示MAC全称为media access control，虽说单词意思一样，最好纠正下

  2019-01-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/97/16/2d19c0e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  林海锋

  如果我假冒了ARP的回复包后续相关的包会发给假冒者吗？（如果假冒者回复的比实际的机器快）

  2019-01-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得 1.hub 集线器，会把包 复制到每台连接的电脑上，不需要包的电脑运动会发生，占用通讯资源 造成了 浪费 2.交换机 具有记忆功能，可以定期缓存 IP与Mac的数据 3.ip封装了tcp、udp、http等数据 4.IP已知 可以使用 ARP协议 用ip获得mac地址，包里有目标mac地址 最好，没有 可以运用ARP协议 通过 吼 获得 目标mac地址 5.通讯的第一个问题 是，谁发送、发给谁、谁接受 6.信息发送 有三种方式 1）信道划分 每个信道一个路径 互不影响 2）轮流协作 比如单双号 发送 3）随机接入协议，先发送 交通堵塞，避开 或者 错开高峰。 回答老师的问题一：ARP协议，已知IP可获得 Mac地址，RARP已知Mac 可获得 IP地址，管理员 可以用Mac定位 定位IP终端设备 问题二：一个局域网有多个交换机 ，广播摸索 回循环 从A-B-C-...-A，这样消耗网络资源，直至资源资源枯竭。 有个问题 请教老师：ip地址 有没有 封装 底层的mac地址，如果封装了，就不需要ARP协议，听过IP获得Mac，如果ip没有Mac地址，上层没有封装下一层协议 好像不太对，怎么思考这个问题的。

  2018-12-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/69/18/c3bd616c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  coder吕

  老师,有个问题希望可以帮我解惑 当CRC校验出错时，链路层接下来会如何处理？

  2018-12-13

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0e/e9/98b6ea61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员大天地

  终于知道了交换机和集线器的工作原理了，那么问题来了，路由器和交换机有什么区别呢？

  2018-12-10

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fc/75/af67e5cc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑猫紧张

  小白提问，ARP算通信吗，如果算那么每次通信都需要目标MAC吧 不理解这个鸡生蛋 蛋生鸡的问题

  2018-12-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e2/63/039f3896.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿吉

  交换机内部有个Mac地址表，记录连接每个端口网卡的对应的Mac地址。 目地Mac地址会到Mac地址表中找相应的端口，找到的话就控制交换电路，给目地和源之间建立一跳独立的信道。 找不到则会向发送端口外其他所有端口广播，目地机如果连在端口上会检测到该条广播并回应，交换机此时会更新路由表，并给目地和源之间建立一跳独立的信道。 貌似一段时间内，凡事未和交换通信的Mac地址都会被从Mac表中清除。

  2018-11-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/cf/96/251c0cee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  xindoo![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  我想问下老师，tcpdump是怎么实现的？它是把不属于本机的流量广播流量拿了出来吗？ 如果是这样，在一个交换机下的几台服务器如何tcpdump抓包？

  2018-10-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/2d/30/b840bd9e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  交叉路口

  如果arp 缓存错误的映射关系，那包不是发送失败？失败后发送方又进行广播再很新本地arp ？

  2018-10-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/2d/18/918eaecf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  后端进阶

  继续昨天那个问题提问，既然目标ip是不变的，为啥还要拿到目标Mac地址再进行传输呢？历史原因还是？

  2018-09-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/60/36/1848c2b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  dovefi

  记得在《大话存储3》里面也讲到了网络设备的进化历程，今天又讲了遍hub，温故知新了

  2018-09-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b3/3b/1104b8e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  XinghaiVictorStarseaSingHoy星...

  讲得很好，深入浅出。对于很多年前已经学过，现在几乎忘光的我，再看这文章，感觉滋滋有味，非常愉快的复习经验。

  2018-09-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9e/d0/efa32d10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张晶鹏

  lan不是通过交换机组成的嘛

  作者回复: 是的

  2018-08-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/71/f0/be872719.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  炎炎

  亲自读，真棒，更有助于理解，讲得也很生动，很值！

  2018-08-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c5/06/6b1ae431.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yao

  基本上每讲都是听三遍看三遍，才能确保一字不漏和理解

  2018-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/48/ab/2e7f2c50.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小冋王月月鸟💡

  感觉少了点细节，ARP查询某个机器的MAC的包也有目标MAC的，这个MAC应该是啥？不会是全0或者全F吧。实践过程中经常会被这些细节搞晕。

  2018-07-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/78/58/84f38396.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张君华

  数据链路层上的“数据包”叫帧是不是更合适一点？

  2018-07-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/24/3f9f7c70.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zixuan

  RARP可以用来更新转发表吧

  2018-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/9a/e3/0a094b50.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Wales

  先猜下，这样一个方案是否可行： 首先，物理上所有人连到同一个局域网里（比如连同一个WIFI）； 然后，通过设置，让每个人的游戏客户端都采用A端口发送数据，用端口B接收数据； 再次，每个参与者在各自的客户端里添加IP列表，该列表包含所有玩家的IP； 最后，游戏进行中，游戏客户端不断地往IP列表里的各个设备进行组播就行了； 改进：IP玩家列表还可添加端口号一项，用于应对游戏客户端从操作系统那里获得的端口号不是A和B的情况。

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/40/f8/8a4aae74.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小龙问路

  以车道来类比信道划分、轮询、随机访问，很形象~

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/df/38/3552c2ae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  匿名用户

  没有交换机的情况下，arp缓存靠集线器来做？

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/ea/813e195d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Nick

  刘老师，客户端在nginx服务器的时候，这个过程中应该经过很多网关，也是通过ARP协议来获取服务器端的MAC地址吗？获取到MAC之后再发数据包？这个过程具体是怎么用样的？如果经过的网关很多，这个访问过程会不会很耗时？

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/19/ee/e395a35e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小先生

  以后大大后期讲讲三层交换机。

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ba/01/5ce8ce0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Leoorz

  高手都在民间，看留言学习中

  作者回复: 是的

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/14/e1/ee5705a2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Zend

  请问一台机器IP:192.168.1.51  MAC:255.255.255.0另一台机器IP:192.168.200.148 MAC:255.255.254.0这两台机器不通，但我在一台机器上部署一个Nginx然后在另一台机器上通过浏览器能访问，这是为什么？

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/06/ad/fe79da1d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  维维

  请教老师，什么场景下会知道对方ip而不知道对方的mac呢？ 如果新接入局域网的pc，是不是在获取到ip地址后，主动发起一个arp，学习局域网的其他设备呢

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8a/de/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  凌传洲

  这是写给零基础的人看的吗？感觉这几篇还是看了还是很蒙蔽啊？我需要一些预备知识吗？

  作者回复: 其实不需要的，比如看ip地址，这个初学者是不是也会呀，例如家里两台电脑组网，其实回家弄两个笔记本电脑就可以试一下了

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/51/a8/14a7c3a0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  EchoRep

  rarp类似dhcp设备获取ip地址
