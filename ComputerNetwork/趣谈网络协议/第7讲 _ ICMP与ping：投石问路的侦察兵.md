# 第7讲 \| ICMP与ping：投石问路的侦察兵

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/9c/9c/9ccec7f6265d778a9973afa25db7ee9c.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/24/ad/2486b041bd60d50f2c45a19ce827e8ad.mp3" type="audio/mpeg"></audio>

无论是在宿舍，还是在办公室，或者运维一个数据中心，我们常常会遇到网络不通的问题。那台机器明明就在那里，你甚至都可以通过机器的终端连上去看。它看着好好的，可是就是连不上去，究竟是哪里出了问题呢？

## ICMP协议的格式

一般情况下，你会想到ping一下。那你知道ping是如何工作的吗？

ping是基于ICMP协议工作的。**ICMP**全称**Internet Control Message Protocol**，就是**互联网控制报文协议**。这里面的关键词是“控制”，那具体是怎么控制的呢？

网络包在异常复杂的网络环境中传输时，常常会遇到各种各样的问题。当遇到问题的时候，总不能“死个不明不白”，要传出消息来，报告情况，这样才可以调整传输策略。这就相当于我们经常看到的电视剧里，古代行军的时候，为将为帅者需要通过侦察兵、哨探或传令兵等人肉的方式来掌握情况，控制整个战局。

ICMP报文是封装在IP包里面的。因为传输指令的时候，肯定需要源地址和目标地址。它本身非常简单。因为作为侦查兵，要轻装上阵，不能携带大量的包袱。

![](<https://static001.geekbang.org/resource/image/20/e2/201589bb205c5b00ad42e0081aa46fe2.jpg?wh=3043*1243>)

ICMP报文有很多的类型，不同的类型有不同的代码。**最常用的类型是主动请求为8，主动请求的应答为0**。

## 查询报文类型

我们经常在电视剧里听到这样的话：主帅说，来人哪！前方战事如何，快去派人打探，一有情况，立即通报！

<!-- [[[read_end]]] -->

这种是主帅发起的，主动查看敌情，对应ICMP的**查询报文类型**。例如，常用的**ping就是查询报文，是一种主动请求，并且获得主动应答的ICMP协议。**所以，ping发的包也是符合ICMP协议格式的，只不过它在后面增加了自己的格式。

对ping的主动请求，进行网络抓包，称为**ICMP ECHO REQUEST。**同理主动请求的回复，称为**ICMP ECHO REPLY**。比起原生的ICMP，这里面多了两个字段，一个是**标识符**。这个很好理解，你派出去两队侦查兵，一队是侦查战况的，一队是去查找水源的，要有个标识才能区分。另一个是**序号**，你派出去的侦查兵，都要编个号。如果派出去10个，回来10个，就说明前方战况不错；如果派出去10个，回来2个，说明情况可能不妙。

在选项数据中，ping还会存放发送请求的时间值，来计算往返时间，说明路程的长短。

## 差错报文类型

当然也有另外一种方式，就是差错报文。

主帅骑马走着走着，突然来了一匹快马，上面的小兵气喘吁吁的：报告主公，不好啦！张将军遭遇埋伏，全军覆没啦！这种是异常情况发起的，来报告发生了不好的事情，对应ICMP的**差错报文类型**。

我举几个ICMP差错报文的例子：**终点不可达为3，源抑制为4，超时为11，重定向为5**。这些都是什么意思呢？我给你具体解释一下。

**第一种是终点不可达**。小兵：报告主公，您让把粮草送到张将军那里，结果没有送到。

如果你是主公，你肯定会问，为啥送不到？具体的原因在代码中表示就是，网络不可达代码为0，主机不可达代码为1，协议不可达代码为2，端口不可达代码为3，需要进行分片但设置了不分片位代码为4。

具体的场景就像这样：

- 网络不可达：主公，找不到地方呀？
- 主机不可达：主公，找到地方没这个人呀？
- 协议不可达：主公，找到地方，找到人，口号没对上，人家天王盖地虎，我说12345！
- 端口不可达：主公，找到地方，找到人，对了口号，事儿没对上，我去送粮草，人家说他们在等救兵。
- 需要进行分片但设置了不分片位：主公，走到一半，山路狭窄，想换小车，但是您的将令，严禁换小车，就没办法送到了。

<!-- -->

**第二种是源站抑制**，也就是让源站放慢发送速度。小兵：报告主公，您粮草送的太多了吃不完。

**第三种是时间超时**，也就是超过网络包的生存时间还是没到。小兵：报告主公，送粮草的人，自己把粮草吃完了，还没找到地方，已经饿死啦。

**第四种是路由重定向**，也就是让下次发给另一个路由器。小兵：报告主公，上次送粮草的人本来只要走一站地铁，非得从五环绕，下次别这样了啊。

差错报文的结构相对复杂一些。除了前面还是IP，ICMP的前8字节不变，后面则跟上出错的那个IP包的IP头和IP正文的前8个字节。

而且这类侦查兵特别恪尽职守，不但自己返回来报信，还把一部分遗物也带回来。

- 侦察兵：报告主公，张将军已经战死沙场，这是张将军的印信和佩剑。
- 主公：神马？张将军是怎么死的（可以查看ICMP的前8字节）？没错，这是张将军的剑，是他的剑（IP数据包的头及正文前8字节）。

<!-- -->

## ping：查询报文类型的使用

接下来，我们重点来看ping的发送和接收过程。

![](<https://static001.geekbang.org/resource/image/57/21/57a77fb89bc4a5653842276c70c0d621.jpg?wh=4463*2786>)

假定主机A的IP地址是192.168.1.1，主机B的IP地址是192.168.1.2，它们都在同一个子网。那当你在主机A上运行“ping 192.168.1.2”后，会发生什么呢?

ping命令执行的时候，源主机首先会构建一个ICMP请求数据包，ICMP数据包内包含多个字段。最重要的是两个，第一个是**类型字段**，对于请求数据包而言该字段为 8；另外一个是**顺序号**，主要用于区分连续ping的时候发出的多个数据包。每发出一个请求数据包，顺序号会自动加1。为了能够计算往返时间RTT，它会在报文的数据部分插入发送时间。

然后，由ICMP协议将这个数据包连同地址192.168.1.2一起交给IP层。IP层将以192.168.1.2作为目的地址，本机IP地址作为源地址，加上一些其他控制信息，构建一个IP数据包。

接下来，需要加入MAC头。如果在本节ARP映射表中查找出IP地址192.168.1.2所对应的MAC地址，则可以直接使用；如果没有，则需要发送ARP协议查询MAC地址，获得MAC地址后，由数据链路层构建一个数据帧，目的地址是IP层传过来的MAC地址，源地址则是本机的MAC地址；还要附加上一些控制信息，依据以太网的介质访问规则，将它们传送出去。

主机B收到这个数据帧后，先检查它的目的MAC地址，并和本机的MAC地址对比，如符合，则接收，否则就丢弃。接收后检查该数据帧，将IP数据包从帧中提取出来，交给本机的IP层。同样，IP层检查后，将有用的信息提取后交给ICMP协议。

主机B会构建一个 ICMP 应答包，应答数据包的类型字段为 0，顺序号为接收到的请求数据包中的顺序号，然后再发送出去给主机A。

在规定的时候间内，源主机如果没有接到 ICMP 的应答包，则说明目标主机不可达；如果接收到了 ICMP 应答包，则说明目标主机可达。此时，源主机会检查，用当前时刻减去该数据包最初从源主机上发出的时刻，就是 ICMP 数据包的时间延迟。

当然这只是最简单的，同一个局域网里面的情况。如果跨网段的话，还会涉及网关的转发、路由器的转发等等。但是对于ICMP的头来讲，是没什么影响的。会影响的是根据目标IP地址，选择路由的下一跳，还有每经过一个路由器到达一个新的局域网，需要换MAC头里面的MAC地址。这个过程后面几节会详细描述，这里暂时不多说。

如果在自己的可控范围之内，当遇到网络不通的问题的时候，除了直接ping目标的IP地址之外，还应该有一个清晰的网络拓扑图。并且从理论上来讲，应该要清楚地知道一个网络包从源地址到目标地址都需要经过哪些设备，然后逐个ping中间的这些设备或者机器。如果可能的话，在这些关键点，通过tcpdump -i eth0 icmp，查看包有没有到达某个点，回复的包到达了哪个点，可以更加容易推断出错的位置。

经常会遇到一个问题，如果不在我们的控制范围内，很多中间设备都是禁止ping的，但是ping不通不代表网络不通。这个时候就要使用telnet，通过其他协议来测试网络是否通，这个就不在本篇的讲述范围了。

说了这么多，你应该可以看出ping这个程序是使用了ICMP里面的ECHO REQUEST和ECHO REPLY类型的。

## Traceroute：差错报文类型的使用

那其他的类型呢？是不是只有真正遇到错误的时候，才能收到呢？那也不是，有一个程序Traceroute，是个“大骗子”。它会使用ICMP的规则，故意制造一些能够产生错误的场景。

所以，**Traceroute的第一个作用就是故意设置特殊的TTL，来追踪去往目的地时沿途经过的路由器**。Traceroute的参数指向某个目的IP地址，它会发送一个UDP的数据包。将TTL设置成1，也就是说一旦遇到一个路由器或者一个关卡，就表示它“牺牲”了。

如果中间的路由器不止一个，当然碰到第一个就“牺牲”。于是，返回一个ICMP包，也就是网络差错包，类型是时间超时。那大军前行就带一顿饭，试一试走多远会被饿死，然后找个哨探回来报告，那我就知道大军只带一顿饭能走多远了。

接下来，将TTL设置为2。第一关过了，第二关就“牺牲”了，那我就知道第二关有多远。如此反复，直到到达目的主机。这样，Traceroute就拿到了所有的路由器IP。当然，有的路由器压根不会回这个ICMP。这也是Traceroute一个公网的地址，看不到中间路由的原因。

怎么知道UDP有没有到达目的主机呢？Traceroute程序会发送一份UDP数据报给目的主机，但它会选择一个不可能的值作为UDP端口号（大于30000）。当该数据报到达时，将使目的主机的 UDP模块产生一份“端口不可达”错误ICMP报文。如果数据报没有到达，则可能是超时。

这就相当于故意派人去西天如来那里去请一本《道德经》，结果人家信佛不信道，消息就会被打出来。被打的消息传回来，你就知道西天是能够到达的。为什么不去取《心经》呢？因为UDP是无连接的。也就是说这人一派出去，你就得不到任何音信。你无法区别到底是半路走丢了，还是真的信佛遁入空门了，只有让人家打出来，你才会得到消息。

**Traceroute还有一个作用是故意设置不分片，从而确定路径的MTU。**要做的工作首先是发送分组，并设置“不分片”标志。发送的第一个分组的长度正好与出口MTU相等。如果中间遇到窄的关口会被卡住，会发送ICMP网络差错包，类型为“需要进行分片但设置了不分片位”。其实，这是人家故意的好吧，每次收到ICMP“不能分片”差错时就减小分组的长度，直到到达目标主机。

## 小结

好了，这一节内容差不多了，我来总结一下：

- ICMP相当于网络世界的侦察兵。我讲了两种类型的ICMP报文，一种是主动探查的查询报文，一种异常报告的差错报文；
- ping使用查询报文，Traceroute使用差错报文。

<!-- -->

最后，给你留两个思考题吧。

1. 当发送的报文出问题的时候，会发送一个ICMP的差错报文来报告错误，但是如果ICMP的差错报文也出问题了呢？
2. 这一节只说了一个局域网互相ping的情况。如果跨路由器、跨网关的过程会是什么样的呢？

<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(156)

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34.jpeg)

  ^_^

  一点个人感觉:老师的比喻有点太用力了，不如简单通俗的讲解原理. 另外错别字和语句通顺程度是会影响理解的，前几期有时也有这个感觉。 专业水平老师还是很牛的，学到很多知识。

  2018-06-03

  **15

  **305

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131997-4483.jpeg)

  Marnie

  TTL的作用没有讲清楚，特此记录一下。 TTL是网络包里的一个值，它告诉路由器包在网络中太长时间是否需要被丢弃。TTL最初的设想是，设置超时时间，超过此范围则被丢弃。每个路由器要将TTL减一，TTL通常表示被丢弃前经过的路由器的个数。当TTL变为0时，该路由器丢弃该包，并发送一个ICMP包给最初的发送者。 tranceroute差错报文会使用

  2018-09-29

  **7

  **166

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131997-4484.jpeg)

  knull

  许多人问：tracerouter发udp，为啥出错回icmp？ 正常情况下，协议栈能正常走到udp，当然正常返回udp。 但是，你主机不可达，是ip层的（还没到udp）。ip层，当然只知道回icmp。报文分片错误也是同理。

  作者回复: 是的

  2018-09-12

  **5

  **135

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4485.jpeg)

  简

  老师我觉得跟多人说你比喻太抽象，我觉得他们底子太差，需要恶补。我觉得你比喻真好！我原来把一本网络基础书看完了，所以一说就懂，你这么说我觉得我六脉打通了

  2018-09-16

  **11

  **130

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4486.jpeg)

  盖

  好像不发ICMP差错报文的一种情况就是ICMP 的差错报文出错，其它还有目的地址为广播时或者源地址不唯一时也不发差错报文。 前面评论问udp为什么返回ICMP报文的，ICMP一般认为属于网络层的，和IP同一层，是管理和控制IP的一种协议，而UDP和TCP是传输层，所以UDP出错可以返回ICMP差错报文

  作者回复: 赞

  2018-06-01

  **5

  **109

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4487.jpeg)

   臣馟飞扬

  百度一下traceroute的原理也很简单啊，看了比喻反而更乱了，希望不是为了比喻而比喻： Traceroute程序的设计是利用ICMP及IP header的TTL（Time To Live）栏位（field）。首先，traceroute送出一个TTL是1的IP datagram（其实，每次送出的为3个40字节的包，包括源地址，目的地址和包发出的时间标签）到目的地，当路径上的第一个路由器（router）收到这个datagram时，它将TTL减1。此时，TTL变为0了，所以该路由器会将此datagram丢掉，并送回一个「ICMP time exceeded」消息（包括发IP包的源地址，IP包的所有内容及路由器的IP地址），traceroute 收到这个消息后，便知道这个路由器存在于这个路径上，接着traceroute 再送出另一个TTL是2 的datagram，发现第2 个路由器...... traceroute 每次将送出的datagram的TTL 加1来发现另一个路由器，这个重复的动作一直持续到某个datagram 抵达目的地。当datagram到达目的地后，该主机并不会送回ICMP time exceeded消息，因为它已是目的地了，那么traceroute如何得知目的地到达了呢？ Traceroute在送出UDP datagrams到目的地时，它所选择送达的port number 是一个一般应用程序都不会用的号码（30000 以上），所以当此UDP datagram 到达目的地后该主机会送回一个「ICMP port unreachable」的消息，而当traceroute 收到这个消息时，便知道目的地已经到达了。所以traceroute 在Server端也是没有所谓的Daemon 程式。 Traceroute提取发 ICMP TTL到期消息设备的IP地址并作域名解析。每次 ，Traceroute都打印出一系列数据,包括所经过的路由设备的域名及 IP地址,三个包每次来回所花时间。

  2019-12-10

  **3

  **65

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4488.jpeg)

  秦俊山

  非计算机专业出身的，听得一脸懵逼。

  2018-06-02

  **13

  **65

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4489.png)

  手撕油条

  越听越想听，一周三更听不爽

  作者回复: 谢谢

  2018-06-01

  **

  **57

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4490.jpeg)

  张张张 💭

  看了这么多，我觉得可以先说原理性描述，然后再比喻，这样可以对比着理解，这样应该更好些

  2018-06-27

  **

  **41

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4491.jpeg)

  番茄尼玛

  同觉得比喻有些用力过猛了，希望老师能在比喻之后再用技术语言解释一下。比如这篇文章中，终点不可达的几个场景，我就没想通网络不可达和主机不可达有什么区别，还有协议不可达和端口不可达有什么区别

  2018-06-15

  **4

  **39

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4492.jpeg)

  Leon📷

  老师，最好能够抓包截图给大家看，有图有真相，可惜留言不能发图片，不然我就把ping 和traceroute的包细节发出来给大家看看，哈哈

  2018-10-26

  **2

  **33

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4493.jpeg)

  川云

  我觉得比喻很好，方便理解，聪明的人才愿意打比方

  2018-12-02

  **

  **23

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4494.jpeg)

  大坏狐狸

  1.对于携带ICMP差错报文的数据报，不再产生ICMP差错报文。   如果主机A发送了一个ICMP的数据报文给主机B，数据在传输过程中经过其中一个路由器出现错误，由于该路由器已经接收到一个ICMP数据报文，所以不会再产生一个ICMP差错报文。 2.对于分片的数据报，如果不是第一个分片，则不产生ICMP差错报文   对于主机A发送了一个分片的数据，如果路由设备或主机接收到的分片数据不是第一个分片数据，不会产生ICMP差错报文。 3.对于具有多播地址的数据报，不产生ICMP差错报文   如果一个ip地址是一个广播地址的话，不会产生ICMP差错报文。 4.对于具有特殊地址如（127.0.0.0或0.0.0.0）的数据报，不产生ICMP差错报文

  2019-04-02

  **1

  **18

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4495.jpeg)

  YXsong

  没基础的小白听的云里雾里呀，好难🤯

  2018-07-22

  **

  **17

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4496.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  weineel![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  老师您好，以我的经验使用tcp/udp协议可以用scoket api编程，但是文中提到ping程序使用了ip协议，那ping程序的实现是不是用到了其他网络编程接口？ 换句话说，每一层的协议都有相应的api可以调用吗？有的话又是什么呢？ 但又感觉我们写的程序一般是应用层用socket就行了…

  作者回复: 不是每一层都有接口的，tcp ip协议栈在内核里面，内核的最外层是系统调用，所以你看起来就只能调用socket了

  2018-06-02

  **2

  **17

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131997-4483.jpeg)

  Marnie

  差错报文就是故意找茬，制造错误让别人打回去。

  2018-09-29

  **

  **16

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132000-4497.jpeg)

  hunterlodge

  差错报文中的端口不可达是指什么端口呢？为什么ICMP协议会有端口概念呢？

  2018-06-16

  **1

  **15

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132000-4498.jpeg)

  xfan

  如果网络不可达，是谁回复的差错报文呢，是网关吗

  作者回复: 网络不可达也是到达了某个地方，发现走不下去了，到哪里哪里返回，一般是某个路由器

  2019-04-09

  **2

  **14

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131997-4483.jpeg)

  Marnie

  感觉查询报文就像编程中的正常流程，差错报文相当于测试程序。

  2018-06-04

  **

  **14

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132000-4499.jpeg)

  Trust me ҉҉҉҉҉҉҉❀

  tracerouter发送的包是什么程序给的响应？  是目标主机的traceroute程序？还是ICMP

  作者回复: 没有服务端，全部在制造错误

  2018-06-01

  **

  **11

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132001-4500.jpeg)

  西门吹牛

  ICMP是第三层网络层的协议 计算几网络教材是这么描述的：为了更有效转发ip数据报和提高转发的机会，ICMP允许主机或者路由器报告差错情况和提供有关异常情况的报告，用于传输出错报告控制信息； 简单的道理放到教材中就晦涩难懂了。 其实可以理解为，ICMP报文可以在主机和路由器之间传输，但传输的不是用户数据，而是网络通不通、主机是否可达、路由是否可用等网络本身的消息，所以可以比作侦察兵，用来侦察在ip层报文传输过程中遇到的异常情况，并能汇报信息，有了异常信息，就可以控制异常，达到更有效的传输效率；控制这种报文，用的是差错报文类型； 另外一种是查询报文类行，可以理解为用来测试俩个主机间的连接是否有效，报文多久能到达； 总之，通过ICMP协议，可以对ip层报文的传输做到把控

  作者回复: 是的

  2020-06-10

  **

  **8

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132001-4501.jpeg)

  橙子

  听了三遍，看了五遍，我决定先查一下这些缩写啥意思😳

  2018-06-25

  **

  **8

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132001-4502.jpeg)

  一一

  我是通过Windows命令行路由追踪新浪网的，一共跳了8次。注：Linux平台命令traceroute，与Windows有所区别。 C:\>tracert www.sina.com.cn 通过最多 30 个跃点跟踪 到 spool.grid.sinaedge.com [112.13.174.126] 的路由:   1     1 ms    <1 毫秒    1 ms  192.168.0.1  2     5 ms     4 ms     4 ms  112.17.101.254  3     5 ms     *        *     111.0.78.165  4     *        *        6 ms  211.138.119.229  5     7 ms     6 ms     6 ms  218.205.58.186  6     8 ms     8 ms     7 ms  218.205.86.30  7     *        *        *     请求超时。  8     5 ms     5 ms     5 ms  112.13.174.126 跟踪完成。

  2019-12-24

  **

  **7

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132001-4503.jpeg)

  caohuan

  本篇所得：ICMP为侦察兵，全程为 Internet control message protocol 互联网控制报文协议，它在网络出现问题的时候 返回（回传）错误包，ICMP包括1.主动探查的 查询报文 2.异常报告的差错报文，查询报文 一般用 ping，差错报文 用Traceroute 请求，差错报文 包括1）终点不可达 2）源站拟制 3）时间超时 4）路由重向。 回答老师的问题 1.如果ICMP差错报文出错，怎么处理，觉得 可以1）多发几次 看回来的错误是否一样，不一样 可能就是ICMP差错报文有异常 2）改变发生的内容 或者 减少发送的内容，查看 差错报文 是否一样。 2.跨网关、跨路由，怎么处理ping主动查询 ，1）老师 在本篇专栏 提到 在进入 新的路由时 会换Mac头里面的Mac地址2）为防止 中间设备 禁止 ping，可用 telnet协议 查找网络问题。 太懂网络，所以才来学习，有不对之处，期待老师的指正。

  2018-12-24

  **

  **6

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132.png)

  anthann

  Traceroute发送的是udp包，为什么也能收到icmp响应呢？

  作者回复: 所以是侦查兵啊，错误了就会发

  2018-06-01

  **

  **6

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132001-4504.jpeg)

  约书亚

  差错包也出问题是不是就拉到了，看上层是不是定时重发的协议了？

  作者回复: 是的，差错包不会在发差错包

  2018-06-01

  **

  **6

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132001-4505.jpeg)

  多襄丸

  [小白评论] 看评论说 icmp也属于网络层啊？ 我还以为icmp属于应用层呢

  作者回复: icmp是网络层，ping是应用

  2019-08-29

  **3

  **5

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  啊路

  ICMP报文图片最后一个字段是相应还是响应，相应怎么理解

  作者回复: 不好意思，写错了，响应

  2018-06-01

  **

  **5

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4506.jpeg)

  饭粒

  Linux 下 man 了下 traceroute。 traceroute 默认是发送的 UDP 探测包，当然也有选项 -I 发送 ICMP 探测包或 -T 发送TCP 探测包，都跟据返回的差错报文 ICMP TIME_EXCEEDED 响应来判断是否可以达到目标地址，如果最终可达应该就没有 TIME_EXCEEDED 响应了，就不用增加 TTL 重发探测包。 测试使用 traceroute www.baidu.com 一直不可达目标服务器（应该是服务器不提供 UDP 服务？）。使用 traceroute -I www.baidu.com，traceroute -T www.baidu.com 经过 12 跳后就达到了目标服务器。

  2019-11-30

  **1

  **4

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4507.jpeg)

  n0thing

  买了很久，最近才静下心来学习，有个疑问:在客户端ping直观上是一个应用程序，理解应该位于应用层，使用icmp协议属于网络层，那ping的过程经过传输层时干了些啥？还是说ping的过程就没传输层啥事？

  作者回复: 没有传输层，可以有下层没上层

  2019-04-25

  **

  **4

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132.png)

  anthann

  ptraceroute用了udp包，为什么也能收到返回的icmp包呢？

  作者回复: 出错了就能了

  2018-06-01

  **

  **4

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4508.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  一个工匠

  ping属于应用层协议，封包后直接传递到ICMP网络层，在这个过程中，是没有传输层事情的。那么端口号是如何确定的呢？没有端口号这个数据包能够发出去吗？

  作者回复: 是的，没有端口号的事情

  2019-08-03

  **3

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4509.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  Null

  补充说明下：ICMP是（Internet Control Message Protocol）Internet控制报文协议。它是TCP/IP协议簇的一个子协议，用于在IP主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。

  2019-03-15

  **

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4510.jpeg)

  loveluckystar

  ICMP在发送时需要通过IP层进行封装然后发出去，那ICMP为啥算是网络层的协议，而不是像TCP一样算是传输层的协议呢？

  2018-10-06

  **2

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4511.jpeg)

  正直D令狐勇

  icmp的差错报文，最终会反馈在socket的api返回的错误代码中吧

  作者回复: 会的

  2018-10-03

  **

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132002-4512.jpeg)

  我爱探索

  不能下载成文本真是最大的遗憾

  2018-09-10

  **

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132000-4497.jpeg)

  hunterlodge

  如果不在我们的控制范围内，很多中间设备都是禁止 ping 的，但是 ping 不通不代表网络不通 请问这种禁止是如何做到的呢？禁止ICMP协议吗？禁止的目的是出于安全吗？如果是的话，不禁会有什么样的安全风险呢？

  2018-06-16

  **

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132002-4513.jpeg)

  wangql

  @大唐江山 不通的原因极可能是桥接的网卡设备对应有问题 默认是auto，但是笔记本一般都有两个或以上网卡，桥接对应出错了就不通了。可以在虚拟网络编辑选项里手工调整

  2018-06-08

  **

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4514.jpeg)

  大唐江山

  穿插一个话题:我在虚拟机上配置桥接，系统是ubuntu，同一网段设置好了，完成后ubuntu能上网，但是ping宿主机不能通ping通。360也卸载了还是不成功。望大佬在后面稍微点拨一下小生，不胜感激，若是之后再添加这么一个内容那就再好不过了。

  作者回复: windows防火墙？还有有时候虚拟化软件建的那个桥也会出问题，可以尝试重新建那个桥

  2018-06-01

  **

  **3

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4515.jpeg)

  Q罗

  比喻太好了，虽然原来这块啥都不懂，听起来也好轻松。赞赞赞！

  2020-09-09

  **

  **2

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4516.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  熊能

  张将军应该是张飞、他要哭了

  2018-11-06

  **

  **2

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4517.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  项峥

  粮草送的太多了吃不完笑尿

  2018-06-09

  **

  **2

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4518.jpeg)

  Smallfly

  老师你好，从 IP 地址到获得 Mac 地址是通过 APR 协议广播，那知道 Mac 地址后发送数据包到目的地也是通过广播的形式发出去，Mac 匹配的设备给予回应吗？如果这样那岂不是只要发数据设备都要广播，设备很多不会造成干扰吗？

  作者回复: 那不是，知道了mac就不是广播了

  2018-06-01

  **

  **2

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4519.jpeg)

  fsj

  抢个沙发

  作者回复: 沙发好

  2018-06-01

  **

  **2

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4520.jpeg)

  Direction

  ping的sequence number（BE）最大为0xFFFF，超过这个以后ping会怎么样？自动结束吗？

  2020-01-30

  **

  **1

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4521.jpeg)

  雨后的夜

  我觉得作者的比喻挺恰当的，有些同学觉得比喻反而更费解可能是因为每个人对比喻的理解方式不一样，个人觉得不需要纠结比喻本身。看看这个课程的标题是“趣谈”，能把枯燥的技术知识讲的生动有趣，这已经是非常好了！

  2019-11-06

  **

  **1

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4522.jpeg)

  侯清风

  看的很过瘾

  作者回复: 过瘾就多看

  2019-07-11

  **

  **1

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132004-4523.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  HelloBug

  老师，你好~如果是ping同一个子网内的一台主机，是哪个主机应答不可达呢？如果是跨网段呢？最后一跳的路由器来应答网络不可达吗？那其他的不可达呢？

  作者回复: 不是最后一跳，是到了某个路由器，发现在路由表中找不到匹配的项了，就是网络不可达

  2018-12-06

  **

  **1

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132004-4524.jpeg)

  天宇

  讲的虽然通俗，但是还是需要基础，非专业出身，网络方面的知识学了不少，这么一看更加贯通了

  2018-09-30

  **

  **1

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132004-4525.jpeg)

  9527

  刘老师，想问一下，这些协议的格式需要掌握吗

  作者回复: 需要知道里面重要的字段，但不需要精确的记住，毕竟可以查的嘛，关键是原理

  2018-06-02

  **

  **1

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132005-4526.jpeg)

  Vikee

  我觉得这篇内容的比喻还是很生动的。

  2022-08-31

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132005-4527.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  piboye

  icmp遇到nat设备后，是怎么保障包可以回来的？

  2022-07-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132005-4528.png)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  水泥中的鱼

  以前白学了，哈哈，重新学一遍

  2022-07-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  先生

  可以进行死亡之ping进行攻击吧？

  2022-06-09

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132005-4529.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  暮雨

  其实学习这这几个协议相关的，最好就是安装个wireshark，用下几个命令ping、traceroute（windows上是TRACERT.exe，linux上没有可以安装下sudo apt install traceroute），抓包看下，再结合老师的讲解，理解起来会更具体

  2022-03-25

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132005-4530.jpeg)

  阳树

  ICMP有专门的应用情景吗

  2022-02-11

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132005-4531.jpeg)

  一步

  比喻很有趣啊，而且一开始设计这些的人灵感也是来源于生活啊。搞不好真是行军打仗，通讯最开始也是用于军事的吧

  2021-12-19

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132005-4532.jpeg)

  Stephen

  ping的过程： 主机a想要ping主机b。他会先封装一个。icmp的8类型的报文。然后加上目的ip地址和自己的ip地址。然后加上自己的mac地址和目的的mac地址。mac地址的获取，如果是arp映射表中，能够拿到则直接去拿。否则会以通过arp协议来获取目的地址的mac，封装的包发送给主机b。主机b底层收到后会逐层解包到达的上层，然后自己封装一个类型为0的报文作为应答，然后按照类似的方式把这个包传递给主机a。

  2021-11-12

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132005-4533.jpeg)

  听说少年

  我还是喜欢直接听原理 我自己去理解比喻.... 老师的这比喻好难受啊啊啊啊啊啊

  2021-11-10

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132005-4534.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  御舟

  老师的比喻很好不过是有点用力了

  2021-10-30

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132006-4535.jpeg)

  xd

  traceroute公网ip,中间有的路由器不会返回,那么源怎么设置后续的TTL? 不是正好被不返回icmp的路由器消耗完了TTL么

  2021-10-24

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132006-4536.jpeg)

  蓝笨笨咱们走

  这门课我认为很好，当然我还是认为这个课适合有基础的。对于有基础的同学，这个课会在关键点点拨你，让你的真气更上一级。此外，能够学习老司机的理解套路和思维模式，这很酷。

  2021-10-21

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132006-4537.jpeg)

  菽绣

  差错报文出错那发送方也会收到一个差错报文吧

  2021-07-02

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_5baa01

  MTU 是什么？

  2021-06-26

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4538.jpeg)

  蔡箐菁

  可以边看文章边查阅文档，https://tools.ietf.org/html/rfc792#ref-1

  2021-04-30

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4539.jpeg)

  Gabriel

  这节我主要学到几个知识点 1：什么事ICMP？ 答：ICMP（internet control message protocol）internet控制报文协议。它是TCP/IP协议簇的一个子协议，用于在IP主机、路由器之间传递控制消息。控制消息是指网路不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然不传输用户数据，但是对用用户数据的传递起着重要作用。 2：如果利用ICMP协议实现网路功能？ a：ping 目标ip或者域名（不带端口），用来测试网络可达性 b：tracert，用来显示到达目的主机的路径 原来，有时候接口报错，504，或是请求时间太长。刚刚开始都不知道怎么排错，首先去看代码，然后自己在本地请求接口，然后就是说代码没有问题呀，怎么会出现这个问题呢？ 其实，在服务器端的报错，不要一开始就检查就想自己代码的问题。既然是服务端的报错，那么就先去排查，网路是否通，如果网络不同，请求不到答，这样去解决网络的问题。 如果网路是通的，这时候去看自己的nnginx进程是否正常，查看access日志和error日志。 如果nginx没有问题，那么就去看php是否异常，查看php进程，在查询phperror， 最后再去看框架自带的错误日志。 学习这门课程，让我懂得该怎么去更好排查错误。

  2021-04-25

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4540.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  念雅

  有点不明白，ICMP是网络层，IP也是网络层，为什么要提出来一个IP层呢？为什么不说成网络层

  2021-03-26

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4541.jpeg)

  V万能的小黑V

  讲得很好，清晰易懂

  2021-03-24

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4542.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  Wangyf

  看了别的同学的留言，差错报文出错了，就不发差错报文了，那.....法什么报文呢？是什么都不返回，等着发出 ICMP 报文的主机超时吗？

  2021-03-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4543.jpeg)

  Bigonepiece

  课程中的图是啥软件画的？windows系统有没有画图软件推荐，谢谢老师。

  2021-03-08

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4544.jpeg)

  fastkdm

  话锋一转就到了网络层，讲完了我都不知道是在讲网络层，查了ICMP协议才知道属于网络层，那网络层能总结一下是干啥的吗

  2021-01-28

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132007-4545.png)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  gerald

  比喻的地方全部跳过后，真正讲解技术的文字没几行，根本没解释清楚

  2020-11-05

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132007-4545.png)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  gerald

  太刻意了，简单的一个协议讲的冗余复杂，为了有趣而有趣

  2020-11-05

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4546.jpeg)

  陈德伟

  1，traceroute默认是 UDP，很多网络设备都禁止udp，tcp得加-I选项 2，traceroute需要root权限 3，tracepath不需要root权限，但好像只能udp 4，mtr也能做侦测

  2020-10-20

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132007-4547.jpeg)

  Bob Ji

  刘老师，您好 ping应该是工作在应用层吧，那它直接使用网络层的ICMP协议，岂不是没有经过传输层？ 始终牢记一个原则：只要是在网络上跑的包，都是完整的。可以有下层没上层，绝对不可能有上层没下层。 那是有矛盾了吗

  2020-10-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132008-4548.jpeg)

  King-ZJ

  这一节对ICMP的差错报文主动使用来探测网络情况有了一个新的认知，赞👍

  2020-09-11

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132008-4549.jpeg)

  正平

  这比喻有意思

  2020-08-11

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132008-4550.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  Siri

  有一个疑问，既然到达目标地址后是先检查mac地址再检查ip地址 但是mac地址不已经是唯一的了么，mac地址一致的情况下为什么还要检查IP地址

  作者回复: mac不是全局唯一，ip和mac也不是一一绑定，有时候响应arp，但是我的ip不是这个，而且中转一下，可以去看lvs的dr模式

  2020-06-09

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4551.jpeg)

  Mabel.Chen

  弱弱地问一下老师，老师讲的这些都是自己学习和经验的积累，那么要想学到这些知识，我想单单是学习老师的课程也还是远远不够的，如果可以的话，希望老师能指导一下还有哪些必备学习资源。

  2020-05-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4552.jpeg)

  吃瓜胖子😂

  对端服务器禁ping，如何确认到对端服务器的网络质量/是否有丢包?

  2020-04-24

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4553.jpeg)

  Heaven

  对于TTL,并非常见的超时时间,而是每次到了一个中转站,都会进行减一,是这样吗

  2020-04-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  学习了，差错报文，就是告诉发送方前方有异常

  2020-03-22

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4554.jpeg)

  宋健

  想请问老师，windows下的tracer的icmp数据类型跟ping是一样的吗？我尝试linux下tracerout百度，结果每一跳都是星星；而在Windows下tracert百度，发现能追踪到路由，是不是他们的icmp的数据类型不同？如果不同，tracert又是什么类型呢？

  2020-03-22

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  ricepot100

  ICMP 报文是封装在 IP 包里面的。这句话怎么理解？是指icmp是ip上层的协议吗？为什么没有mac头？

  2020-03-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132009-4555.jpeg)

  Geek_f7658e

  第一次看的模糊，第二次看懂了，老师的文章是总领知识的，有些细节需要初学者打磨下

  2020-03-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4556.jpeg)

  子杨

  这里面想了解一下，traceroute 发 UDP 包是干嘛的？ping 为什么不需要 UDP 呢？

  2020-02-24

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4557.jpeg)

  YU

  建议先看一遍图解tcp/ip之后，再来看这些内容。图解top/ip不厚，讲的也还不错。

  2020-01-14

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4558.jpeg)

  麻花

  有一点不明白， “它会选择一个不可能的值作为 UDP 端口号（大于 30000）” 这里大于30000也不是不可能的端口号，只能说是一个不常用的端口号吧

  2020-01-01

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_wongk

  感觉有点吃力的可以先找一本网络书看看，补补基础再来看。老师的水平很高，基础越扎实收获越多。

  2019-12-29

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4559.jpeg)

  暮雨

  traceroute的每次路由路径会一样么

  2019-12-27

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132009-4560.jpeg)

  teddytyy

  时间超时的比喻笑死我了

  2019-11-13

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4561.jpeg)

  宋不肥

  非科班，控制出身，网络方面0基础。研一上课，导研二终于有时间来看了，每一节看的都很辛苦，第一遍都听得头晕，但看了两三遍就慢慢能看懂了，到第四遍左右就会发现老师的行文逻辑确实很清晰nice，对应着在看计算机的那本教材，本科选修的时候看不懂，现在对应的部分，对着书和老师的课看就感觉理解的还挺透彻的（自我感觉），确实有用。感谢老师

  2019-10-27

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4562.jpeg)

  阿阳

  仔细反复的看到这一节，真心佩服老师对于网络知识讲解的安排顺序。网络知识关联性太强，稍微安排顺序有些问题就可能讲不通。看到这种讲解顺序，真是用心良苦，理论联系实际，打通了我的很多困惑的点。赞！

  2019-09-19

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4563.jpeg)

  胡永

  老师的比喻挺好的，如果配合上终端的操作图示就更好了

  作者回复: 主要讲原理，适配音频

  2019-09-04

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132010-4564.png)

  叶叶

  比较适合对计算机网络有所了解的人食用。

  作者回复: 仔细品读，其实还好啦

  2019-07-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4565.jpeg)

  尔冬橙

  老师，ping那个图右半部分不是从下往上？按理不应该先经过MAC层再到IP层再ICMP响应回来？

  作者回复: 图中画的也是这样的

  2019-07-11

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4566.jpeg)

  天然

  老师讲的蛮好，以前觉得网络太枯燥，听您的越听越想听

  作者回复: 想听就多听，加油

  2019-07-09

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4567.jpeg)

  若星

  有种还在练习扎马步，老师在讲武功秘籍的赶脚😭

  2019-06-23

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4568.jpeg)

  FF

  一环扣一环的

  2019-06-21

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132010-4569.jpeg)

  Aaaaaaaaaaayou

  Traceroute 的参数指向某个目的 IP 地址，它会发送一个 UDP 的数据包。  这里为什么要发一个udp的包？不应该是icmp的包吗？

  作者回复: 错了才发icmp

  2019-05-28

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132011-4570.jpeg)

  Leedom

  觉得老师讲的挺好的，在校上这门课就一直期末背书，背知识点，而现在貌似更清晰的来理解，希望秋招也能拿网易offer，offer++

  作者回复: 加油，offer多多

  2019-05-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132011-4571.jpeg)

  Carlos

  老师，这篇文章读了好几遍，总算基本搞清楚ping和traceroute的原理了，但是我还是不知道这俩命令的具体应用场景是什么？我现在理解ping就是看目标主机是否可达，并且了解下延时是多少。traceroute是看下从客户端到达目标主机经过了哪些路由，以及所用的时间。还是没搞清楚什么情况下要使用它。

  作者回复: 发现通路不通的时候，需要通过他们来探路

  2019-05-11

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132011-4572.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  Geek_f8cd05

  老师讲的诙谐幽默，通俗易懂，👍

  2019-05-05

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132011-4573.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  xiaoq

  windows 下 使用 tracert，抓包发现其发出的并不是 udp， 而是 icmp 。

  2019-05-02

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132011-4574.jpeg)

  Geek_199a85

  通过tracerote 来跟踪 包经过的路由器的时候，为什么中间又几个是 timeout，但是最后包还是能送到目的IP 通过最多 30 个跃点跟踪到 xx.xx.xx.xx  的路由   1     2 ms     2 ms     2 ms  10.134.28.1  2     *        *        *     请求超时。  3     *        *        *     请求超时。  4    38 ms    38 ms    38 ms  xx.xx.xx.xx 类似这样

  2019-04-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132011-4575.jpeg)

  Nyng

  比喻太精彩了太栩栩如生了！一看到这些比喻我就精神一振会心一笑，同时感觉知识也融会贯通了，真像上面有人说的打通了任督二脉。

  2019-04-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132011-4576.jpeg)

  灯盖

  一半一半，一半懂一半不懂，还得再来一遍

  作者回复: 那就再来一遍

  2019-04-12

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132011-4577.jpeg)

  allwmh

  1.tracerouter收到的icmp的包都是路由器返回的吗？ 2.如果是tcp发送的一个包到某个不存在的ip地址的时候，是否也是能收到路由器发回的icmp包呢？ 3.如果2能收到，意思是不是说icmp的一部分能力其实也能理解为ip层的一种网络处理方式呢？

  作者回复: 是的。icmp是网络层的

  2019-04-03

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4578.jpeg)

  少盐

  Icmp是协议吗，作为侦察兵属于层级，traceroute 故意发送错误信息，如果能收到回复说明道路是通的

  作者回复: 是的

  2019-03-31

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4579.jpeg)

  愤怒的一口

  看到这里觉得老师写的确实是好啊！！！前面都讲解的非常到位。但是这一章的Traceroute  这块 有点蒙，不知道后面还没有讲解，先继续再看两章再回来看一下。

  作者回复: 后面icmp不再讲了。可以试一下

  2019-03-30

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4580.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  唐宋元明清

  听不懂或者感觉太困难的请把大学的计算机网络原理仔细看一遍

  作者回复: 好，赞

  2019-03-27

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4581.png)

  小mango.Melody

  老师这个段子手属性太强了

  2019-03-26

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4582.jpeg)

  且听疯吟

  ping的过程如果跨路由或者网关的的话就会比较复杂。

  作者回复: 对的

  2019-03-25

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131999-4494.jpeg)

  大坏狐狸

  一看到西游记，我就乐。为什么我买这个课程，就是因为看到刘老师的，趣谈网络协议的一篇文章，讲的就是大乘佛法。哈哈，来取取经。

  作者回复: 哈哈，大乘佛法

  2019-03-22

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132012-4583.png)

  桂城老托尼

  听老师讲课，需要对比着书上的原理动手实践

  2019-03-21

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4584.jpeg)

  蜉蝣

  以为：与其说老师比喻太用力，不如说自己的抽象能力、知识提炼能力太差。这就像是语文课上学了一篇大师的文章，语文老师说，你们来把文章讲了什么总结一下，结果下面人一个个目瞪口呆。——请多从自身找原因。

  作者回复: 谢谢

  2019-03-19

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4585.jpeg)

  云计算小菜鸟

  初学者问一个问题:每种种类的icmp差错报文都是什么设备返回的？比如如果是网络不可达类型的，具体是哪一个设备返回的这个错误报文呢？

  作者回复: 中间设备，找不到下一跳，就返回

  2019-03-12

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/132-1662307132012-4586.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  senekis

  老师比喻的很好，以前没有理解好的命令和协议有了更进一步的认识，以前都是死记硬背的，帮助很大，谢谢老师

  2019-03-05

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132012-4587.jpeg)

  卡萨布兰卡

  mac层只会获取mac头，不会去获取ip头的吧，ip头是要ip层去获取的吧

  作者回复: 是的

  2019-02-16

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4588.jpeg)

  ...

  老师 traceroute如果是主机不可达会返回icmp吗

  作者回复: 是的

  2019-01-13

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4589.jpeg)

  herongwei

  ICMP 差错报文的例子，老师举的例子真是太生动形象了，科班出身的人真是看的得劲啊，哈哈哈

  2019-01-09

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4590.jpeg)

  古夜

  首先我觉得老师的必须没问题，但那是针对有底子的人来说。对于没接触过网络协议的人靠比喻对他们没有一个清晰的认识。老师能推荐一本协议的书吗，能和课程配套看最好，这样很多问题就迎刃而解了。

  作者回复: 网络协议，自顶向下方法

  2019-01-08

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4591.jpeg)

  young

  哪些不同情况下的差错报文都是谁回的呢？

  2018-12-14

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4592.jpeg)

  艾子晴👟💓（Veronica Chen...

  同样非计算机专业 我觉得比喻得很好 听不懂的人 自己查查 或者再听一听 或者留言具体的问题呗

  2018-12-07

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4593.jpeg)

  Untitled

  老师，有个概念比较吃力。ICMP属于几层协议呢，如果属于网络层协议，其又怎么能故意设置端口不可达、协议不可达等差错报文呢？，属于哪一层协议，是通过什么规则来判断的？

  作者回复: 不用纠结层，实践中用不到，如果考试，教材说哪层就是哪层

  2018-12-02

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132013-4594.jpeg)

  joy

  老师，第一个图中的请求与“相应”应该是响应

  2018-11-27

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132014-4595.jpeg)

  Ada

  老师讲的很清晰，比喻特别适合无基础的，自己用心听，不懂的百度再问老师，可以理解的80%，还是学习到很多。就是我的语音听一会儿就自动停了，过了一会儿就又开始自己播放了，就是这么设置的，还是有问题呀

  2018-11-23

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132015-4596.jpeg)

  周曙光爱学习

  光看或者光听肯定是有些枯燥且不易懂的，用wireshark抓个包对照文字看的话那么就一目了然了

  2018-11-21

  **1

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132015-4597.png)

  扬～

  ICMP协议不经过传输层，那如何知道端口不可达呢？   traceroute的通过不断增加TTL来获知到目的地经过的路由器信息，但是每个数据包的路径都是根据当时的网络环境动态确定的路由路径，所以TTL为5走的路径可能与TTL为3走的不一样。那么我们最终得到的不同TTL走到的最远节点，但是这些节点连接起来可能不能到达最终的主机啊，所以得到这样的意义何在呢

  作者回复: 发送的是udp

  2018-11-08

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132016-4598.jpeg)

  小羊

  tracerouter 设置不分片，但是mtu的check在mac层，mac层已经报错了，为什么还会返回icmp（ip）层

  2018-10-26

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132016-4599.jpeg)

  搬铁少年ai

  我到觉得比喻很好，听完会心一笑。当然，我对基本的力量知识已经有所掌握理解，这可能是一个原因

  2018-10-11

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132016-4600.jpeg)

  小先生

  ICMP一般认为属于网络层的，和IP同一层，是管理和控制IP的一种协议，而UDP和TCP是传输层，所以UDP出错可以返回ICMP差错报文

  2018-09-28

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4601.jpeg)

  dovefi

  ICMP协议本身有太多种代码了，每一种代码一个编号，所以很容易混乱，即使过了一遍《tcp/ip协议》卷一原理清晰，但是对于记住代码还是很费劲

  2018-09-26

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4602.jpeg)

  马若飞

  张将军真可怜

  2018-09-20

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4603.jpeg)![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,w_14.png)

  王佳

  老师，你好。我想问一下为什么对具有多播地址的数据报，都不发送ICMP差错报告报文 对具有特殊地址（例如127.0.0.0或0.0.0.0）的数据报，都不发送ICMP差错报告报文。 等等这些不发送ICMP差错报文的情况，为什么不发送ICMP差错报文

  2018-09-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4485.jpeg)

  简

  老师这些我都知道，但你一讲我就感觉学的知识活了

  2018-09-16

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307131998-4485.jpeg)

  简

  作者你知道你要成就多少给技术高手不？简直爱死你了

  2018-09-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  朝闻道

  有些ip地址mtr不通，ping可以通，请问这是为什么？

  2018-09-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  ICMP报文类型有多种，最常用的是查询报文类型是主动请求的，还有一种差错报文类型不用主动请求的；Traceroute发udp包的过程并没有主动发送ICMP请求报文，是路上碰到的路由器觉得有问题返回的ICMP差错报文。是这么理解吗？

  作者回复: 是的

  2018-09-13

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4604.jpeg)

  土拨鼠

  听了都说好

  作者回复: 谁用谁知道，哈哈

  2018-09-04

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4605.jpeg)

  冰封飞飞

  1.icmp报文有校验和，如果出错，校验失败会丢包 2.icmp也是有ip头的，在公网上面转发和正常ip报文应该一样

  2018-08-20

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4606.jpeg)

  Cola

  纠正一下对上一条留言，icmp应该没有纠错机制的，出错了只能等待超时。

  2018-08-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132018-4606.jpeg)

  Cola

  对于问题二，ping的过程应该是这样的 1. 查看这个ip是不是在同一网段 2.发现不在同一网段，找到设置到本机网关的ip 3.通过arp协议拿到网关mac地址并发包到网关 4.网关解包，发现目的ip是在a口出，转发该包 5后续流程相同 对于问题一，差错报文自己出错是不是有自己的重试机制，如果多次重试无效就超时？

  2018-08-15

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4607.jpeg)

  慢熊胖胖跑![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  ping包的顺序字段有些不理解，我ping一次不是就一个包吗？那里来的顺序？

  2018-08-08

  **1

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4608.jpeg)

  fantast

  我想问，发送一个icmp请求，无论出错还是请求成功，是由谁回应呢，即谁发送查询结果或查询报错报文呢？

  2018-08-01

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4609.jpeg)

  SimonKoh

  太赞了！ 这个比喻能感觉到老师下了好大的功夫，谢谢老师😁

  2018-07-31

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4610.jpeg)

  杨领well

  以这个专栏为提纲，在看《TCPIP详解卷一》相关内容前，先对大概内容有个全局的了解。这是我现在的学习方式。

  2018-07-18

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4611.jpeg)

  张君华

  文中描述TTL的递增获取网络设备信息，TTL和跳数有什么区别？

  2018-07-04

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4612.jpeg)

  -Nao剧-

  老师你好～有两个问题请教下： 1、icmp协议是属于哪一层的协议？ 2、之前有说过一个网络包必须是完整的，那icmp的包是否也有应用层传输层的数据呢？

  作者回复: 可以有下层没上层，不可以有上层没下层

  2018-06-22

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132019-4613.jpeg)

  超

  Transroute 发送udp包，设置ttl为1  udp头里没有头ttl，具体怎么发送跟设置呢？

  作者回复: ip里面有的

  2018-06-20

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132020-4614.jpeg)

  helloworld

  “tracerouter发送的包是什么程序给的响应？ 是目标主机的traceroute程序？还是ICMP”，我觉得应该是目标主机的内核机制响应的，这么理解对不对

  2018-06-05

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132020-4615.jpeg)

  Cloud

  云计算新人期待后续课程😁

  2018-06-04

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132003-4518.jpeg)

  Smallfly

  前面有一篇文章写道“你就可以靠吼了，大声喊身份证 XXXX 的是哪位？我听到了，我就会站起来说，是我啊。” 这里把 Mac 地址比喻成身份证的例子，不是在广播么……

  作者回复: 是的

  2018-06-02

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132020-4616.jpeg)

  katsueiki

  icmp是三层协议吧，请问icmp和普通的ip协议有什么区别？

  2018-06-02

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132004-4525.jpeg)

  9527

  刘老师，想问一下，这些协议的格式需要都掌握吗？

  2018-06-02

  **

  **

- ![img](%E7%AC%AC7%E8%AE%B2%20_%20ICMP%E4%B8%8Eping%EF%BC%9A%E6%8A%95%E7%9F%B3%E9%97%AE%E8%B7%AF%E7%9A%84%E4%BE%A6%E5%AF%9F%E5%85%B5.resource/resize,m_fill,h_34,w_34-1662307132020-4617.jpeg)

  天空白云

  请教下老师，apm 方面，首包时间精准定义是什么？有人说是网络请求发完算起，到第一个返回到数据包。 另外有说是dns，建立连接也算起来了。有点迷糊了。老师辛苦了！

  2018-06-01

  **

  **
