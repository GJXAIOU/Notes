# 第30讲 \| 容器网络之Flannel：每人一亩三分地

作者: 刘超

完成时间:

总结时间:



<audio><source src="https://static001.geekbang.org/resource/audio/43/d9/43537d3108145697419700fac4ad6ad9.mp3" type="audio/mpeg"></audio>

上一节我们讲了容器网络的模型，以及如何通过NAT的方式与物理网络进行互通。

每一台物理机上面安装好了Docker以后，都会默认分配一个172.17.0.0/16的网段。一台机器上新创建的第一个容器，一般都会给172.17.0.2这个地址，当然一台机器这样玩玩倒也没啥问题。但是容器里面是要部署应用的，就像上一节讲过的一样，它既然是集装箱，里面就需要装载货物。

如果这个应用是比较传统的单体应用，自己就一个进程，所有的代码逻辑都在这个进程里面，上面的模式没有任何问题，只要通过NAT就能访问进来。

但是因为无法解决快速迭代和高并发的问题，单体应用越来越跟不上时代发展的需要了。

你可以回想一下，无论是各种网络直播平台，还是共享单车，是不是都是很短时间内就要积累大量用户，否则就会错过风口。所以应用需要在很短的时间内快速迭代，不断调整，满足用户体验；还要在很短的时间内，具有支撑高并发请求的能力。

单体应用作为个人英雄主义的时代已经过去了。如果所有的代码都在一个工程里面，开发的时候必然存在大量冲突，上线的时候，需要开大会进行协调，一个月上线一次就很不错了。而且所有的流量都让一个进程扛，怎么也扛不住啊！

没办法，一个字：拆！拆开了，每个子模块独自变化，减少相互影响。拆开了，原来一个进程扛流量，现在多个进程一起扛。所以，微服务就是从个人英雄主义，变成集团军作战。

<!-- [[[read_end]]] -->

容器作为集装箱，可以保证应用在不同的环境中快速迁移，提高迭代的效率。但是如果要形成容器集团军，还需要一个集团军作战的调度平台，这就是Kubernetes。它可以灵活地将一个容器调度到任何一台机器上，并且当某个应用扛不住的时候，只要在Kubernetes上修改容器的副本数，一个应用马上就能变八个，而且都能提供服务。

然而集团军作战有个重要的问题，就是通信。这里面包含两个问题，第一个是集团军的A部队如何实时地知道B部队的位置变化，第二个是两个部队之间如何相互通信。

第一个问题位置变化，往往是通过一个称为注册中心的地方统一管理的，这个是应用自己做的。当一个应用启动的时候，将自己所在环境的IP地址和端口，注册到注册中心指挥部，这样其他的应用请求它的时候，到指挥部问一下它在哪里就好了。当某个应用发生了变化，例如一台机器挂了，容器要迁移到另一台机器，这个时候IP改变了，应用会重新注册，则其他的应用请求它的时候，还是能够从指挥部得到最新的位置。

![](<https://static001.geekbang.org/resource/image/a0/0d/a0763d50fc4e8dcec37ae25a2f6cc60d.jpeg?wh=1582*1080>)

接下来是如何相互通信的问题。NAT这种模式，在多个主机的场景下，是存在很大问题的。在物理机A上的应用A看到的IP地址是容器A的，是172.17.0.2，在物理机B上的应用B看到的IP地址是容器B的，不巧也是172.17.0.2，当它们都注册到注册中心的时候，注册中心就是这个图里这样子。

![](<https://static001.geekbang.org/resource/image/e2/dd/e20596506dd34122e302a7cfc8bb85dd.jpg?wh=1920*847>)

这个时候，应用A要访问应用B，当应用A从注册中心将应用B的IP地址读出来的时候，就彻底困惑了，这不是自己访问自己吗？

怎么解决这个问题呢？一种办法是不去注册容器内的IP地址，而是注册所在物理机的IP地址，端口也要是物理机上映射的端口。

![](<https://static001.geekbang.org/resource/image/8f/18/8fabf1de2a7d346856a032dbf2417b18.jpg?wh=2478*1268>)

这样存在的问题是，应用是在容器里面的，它怎么知道物理机上的IP地址和端口呢？这明明是运维人员配置的，除非应用配合，读取容器平台的接口获得这个IP和端口。一方面，大部分分布式框架都是容器诞生之前就有了，它们不会适配这种场景；另一方面，让容器内的应用意识到容器外的环境，本来就是非常不好的设计。

说好的集装箱，说好的随意迁移呢？难道要让集装箱内的货物意识到自己传的信息？而且本来Tomcat都是监听8080端口的，结果到了物理机上，就不能大家都用这个端口了，否则端口就冲突了，因而就需要随机分配端口，于是在注册中心就出现了各种各样奇怪的端口。无论是注册中心，还是调用方都会觉得很奇怪，而且不是默认的端口，很多情况下也容易出错。

Kubernetes作为集团军作战管理平台，提出指导意见，说网络模型要变平，但是没说怎么实现。于是业界就涌现了大量的方案，Flannel就是其中之一。

对于IP冲突的问题，如果每一个物理机都是网段172.17.0.0/16，肯定会冲突啊，但是这个网段实在太大了，一台物理机上根本启动不了这么多的容器，所以能不能每台物理机在这个大网段里面，抠出一个小的网段，每个物理机网段都不同，自己看好自己的一亩三分地，谁也不和谁冲突。

例如物理机A是网段172.17.8.0/24，物理机B是网段172.17.9.0/24，这样两台机器上启动的容器IP肯定不一样，而且就看IP地址，我们就一下子识别出，这个容器是本机的，还是远程的，如果是远程的，也能从网段一下子就识别出它归哪台物理机管，太方便了。

接下来的问题，就是**物理机A上的容器如何访问到物理机B上的容器呢？**

你是不是想到了熟悉的场景？虚拟机也需要跨物理机互通，往往通过Overlay的方式，容器是不是也可以这样做呢？

**这里我要说Flannel使用UDP实现Overlay网络的方案。**

![](<https://static001.geekbang.org/resource/image/07/71/07217a9ee64e1970ac04de9080505871.jpeg?wh=1920*817>)

在物理机A上的容器A里面，能看到的容器的IP地址是172.17.8.2/24，里面设置了默认的路由规则default via 172.17.8.1 dev eth0。

如果容器A要访问172.17.9.2，就会发往这个默认的网关172.17.8.1。172.17.8.1就是物理机上面docker0网桥的IP地址，这台物理机上的所有容器都是连接到这个网桥的。

在物理机上面，查看路由策略，会有这样一条172.17.0.0/24 via 172.17.0.0 dev flannel.1，也就是说发往172.17.9.2的网络包会被转发到flannel.1这个网卡。

这个网卡是怎么出来的呢？在每台物理机上，都会跑一个flanneld进程，这个进程打开一个/dev/net/tun字符设备的时候，就出现了这个网卡。

你有没有想起qemu-kvm，打开这个字符设备的时候，物理机上也会出现一个网卡，所有发到这个网卡上的网络包会被qemu-kvm接收进来，变成二进制串。只不过接下来qemu-kvm会模拟一个虚拟机里面的网卡，将二进制的串变成网络包，发给虚拟机里面的网卡。但是flanneld不用这样做，所有发到flannel.1这个网卡的包都会被flanneld进程读进去，接下来flanneld要对网络包进行处理。

物理机A上的flanneld会将网络包封装在UDP包里面，然后外层加上物理机A和物理机B的IP地址，发送给物理机B上的flanneld。

为什么是UDP呢？因为不想在flanneld之间建立两两连接，而UDP没有连接的概念，任何一台机器都能发给另一台。

物理机B上的flanneld收到包之后，解开UDP的包，将里面的网络包拿出来，从物理机B的flannel.1网卡发出去。

在物理机B上，有路由规则172.17.9.0/24 dev docker0 proto kernel scope link src 172.17.9.1。

将包发给docker0，docker0将包转给容器B。通信成功。

上面的过程连通性没有问题，但是由于全部在用户态，所以性能差了一些。

跨物理机的连通性问题，在虚拟机那里有成熟的方案，就是VXLAN，那**能不能Flannel也用VXLAN呢**？

当然可以了。如果使用VXLAN，就不需要打开一个TUN设备了，而是要建立一个VXLAN的VTEP。如何建立呢？可以通过netlink通知内核建立一个VTEP的网卡flannel.1。在我们讲OpenvSwitch的时候提过，netlink是一种用户态和内核态通信的机制。

当网络包从物理机A上的容器A发送给物理机B上的容器B，在容器A里面通过默认路由到达物理机A上的docker0网卡，然后根据路由规则，在物理机A上，将包转发给flannel.1。这个时候flannel.1就是一个VXLAN的VTEP了，它将网络包进行封装。

内部的MAC地址这样写：源为物理机A的flannel.1的MAC地址，目标为物理机B的flannel.1的MAC地址，在外面加上VXLAN的头。

外层的IP地址这样写：源为物理机A的IP地址，目标为物理机B的IP地址，外面加上物理机的MAC地址。

这样就能通过VXLAN将包转发到另一台机器，从物理机B的flannel.1上解包，变成内部的网络包，通过物理机B上的路由转发到docker0，然后转发到容器B里面。通信成功。

![](<https://static001.geekbang.org/resource/image/01/79/01f86f6049eef051d48e2e235fa43d79.jpeg?wh=1920*835>)

## 小结

好了，今天的内容就到这里，我来总结一下。

- 基于NAT的容器网络模型在微服务架构下有两个问题，一个是IP重叠，一个是端口冲突，需要通过Overlay网络的机制保持跨节点的连通性。

- Flannel是跨节点容器网络方案之一，它提供的Overlay方案主要有两种方式，一种是UDP在用户态封装，一种是VXLAN在内核态封装，而VXLAN的性能更好一些。


<!-- -->

最后，给你留两个问题：

1. 通过Flannel的网络模型可以实现容器与容器直接跨主机的互相访问，那你知道如果容器内部访问外部的服务应该怎么融合到这个网络模型中吗？

2. 基于Overlay的网络毕竟做了一次网络虚拟化，有没有更加高性能的方案呢？


<!-- -->

我们的专栏更新到第30讲，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送<span class="orange">学习奖励礼券</span>

和我整理的<span class="orange">独家网络协议知识图谱</span>

。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(26)

- ![img](%E7%AC%AC30%E8%AE%B2%20_%20%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C%E4%B9%8BFlannel%EF%BC%9A%E6%AF%8F%E4%BA%BA%E4%B8%80%E4%BA%A9%E4%B8%89%E5%88%86%E5%9C%B0.resource/resize,m_fill,h_34,w_34.jpeg)

  小宇宙

  flannel的backend除了UDP和vxlan还有一种模式就是host-gw，通过主机路由的方式，将请求发送到容器外部的应用，但是有个约束就是宿主机要和其他物理机在同一个vlan或者局域网中，这种模式不需要封包和解包，因此更加高效。

  作者回复: 对的

  2018-08-27

  **5

  **39

- ![img](https://static001.geekbang.org/account/avatar/00/10/d6/60/f21b2164.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  jacy

  回答自己提的udp丢包的问题，希望老师帮忙看看是否正确。 flannel实际上是将docker出来的包再加udp封装，以支持二层网络在三层网络中传输。udp确实可能丢包，丢包是发生在flannel层上，丢包后内层的docker（如果被封装的是tcp）无ack,源docker的虚拟网卡还会按tcp协议进行重发。无非是按flannel原理多来几次。

  作者回复: udp会丢的

  2019-03-14

  **

  **16

- ![img](%E7%AC%AC30%E8%AE%B2%20_%20%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C%E4%B9%8BFlannel%EF%BC%9A%E6%AF%8F%E4%BA%BA%E4%B8%80%E4%BA%A9%E4%B8%89%E5%88%86%E5%9C%B0.resource/resize,m_fill,h_34,w_34-1662308186318-9854.jpeg)

  天王

  1 快速迭代，单体应用不能满足快速迭代和高并发的要求，所以需要拆成微服务，容器作为集装箱，可以保证应用在不同的环境快速迁移，还需要一个容器的调度平台，可以将容器快速的调度到任意服务器，这个调度平台就是k8s。2 微服务之间存在服务调用的问题，就像集团军作战，需要解决各个部队位置和部队之间通讯的问题，2.1 位置问题用注册中心，但是可能会有ip端口冲突，Flannel是为了解决这种问题的技术，给每个物理机分配一小段网络段，每个物理机的容器只使用属于自己的网络段，2.2 部队之间通讯 容器网络互相访问，Flannel使用UDP实现Overlay网络，每台物理机上都跑一个flannelid进程，打开dev/net/tun设备的时候，就会有这个网卡，所有发到flannel.1的网络包，都会被flannelid进程截获，会讲网络包封装进udp包，发到b的flannel.1，b的flannelid收到网络包以后，解开，由flannel.1发出去，通过dock0给到容器b。通讯比较慢，Flannel使用VXLAN技术，建立一个VXLAN的VTEP，通过netlink通知内核建立一个VTEP的网卡flannel.1，A物理机上的flannel.1就是vxlan的vtep，将网络包封装，通过vxlan将包转到另一台服务器上，b的flannel.1接收到，解包，变成内部的网络包，通过物理机上的路由转发到docker0，然后转发到容器B里面，通信成功。

  2019-07-30

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a0/6e/85512d27.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘工的一号马由

  1默认路由发到容器网络的网关 2underlay网络

  2018-07-26

  **

  **6

- ![img](%E7%AC%AC30%E8%AE%B2%20_%20%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C%E4%B9%8BFlannel%EF%BC%9A%E6%AF%8F%E4%BA%BA%E4%B8%80%E4%BA%A9%E4%B8%89%E5%88%86%E5%9C%B0.resource/resize,m_fill,h_34,w_34-1662308186319-9856.jpeg)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  悟空聊架构

  “例如物理机 A 是网段 172.17.8.0/24，物理机 B 是网段 172.17.9.0/24”，这里应该是指物理机A给docker分配的网段吧？老师，这个容易造成误解哦……

  2018-08-03

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/10/83/ae/c082bb25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大星星

  想问下老师，后一种使用VTEP，为什么flannel.id里面内部源地址配置的是flannel.id的mac地址，而不是使用第一种打开dev/net/tun时候源地址写的是容器A的源地址。 为什么两种情况下不一样了，谢谢

  作者回复: 这是flannel的一个问题，会造成很多困扰

  2019-02-22

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/a1/3a/9e48ce31.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小宇

  老师好，我这里有一个疑问，第28讲的VXLAN，虚拟机通信时包的结构， VTEP的设备ip为什么在VXLAN头之外，我的理解VTEP设备的ip应该和flannel的VXLAN模式一样，在VXLAN头里面。

  作者回复: vxlan的格式都是一样的

  2019-01-23

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  \#  tcpdump -i eth0 dst 192.168.1.7 -w  dst.pcap 抓了下 vxlan 类型的 flannel 容器间的包，确实是 udp 的封包，但是没有看到 vxlan 的信息，不知道这要怎么看？ Frame 3: 201 bytes on wire (1608 bits), 201 bytes captured (1608 bits) Ethernet II, Src: fa:16:3e:08:a8:46 (fa:16:3e:08:a8:46), Dst: fa:16:3e:5a:de:91 (fa:16:3e:5a:de:91) Internet Protocol Version 4, Src: 192.168.1.4, Dst: 192.168.1.7 User Datagram Protocol, Src Port: 54228, Dst Port: 8472 Data (159 bytes)

  2019-12-07

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/3a/e1/b6b311cb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ╯梦深处゛

  对于 IP 冲突的问题，如果每一个物理机都是网段 172.17.0.0/16，肯定会冲突啊，但是这个网段实在太大了，一台物理机上根本启动不了这么多的容器，所以能不能每台物理机在这个大网段里面，抠出一个小的网段，每个物理机网段都不同，自己看好自己的一亩三分地，谁也不和谁冲突。 ----------------------------------------------------------------------------------------------------- 老师，不同节点之间不一定都是不同网段的，相反很多时候，一个K8S集群的所有节点都是在一个网段，请问在这种场景下，如果避免这种冲突的问题呢？

  作者回复: 一个K8S集群的所有节点都是在一个大网段里面，但是不同的主机分了小网段。

  2019-04-23

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/16/85/ed/905b052f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  超超

  回答问题1：是不是在dockerX网卡上做NAT？

  作者回复: 是的

  2019-04-08

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/d6/60/f21b2164.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  jacy

  udp丢包能接受吗

  2019-03-13

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/26/fa3bb8e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ⊙▽⊙

  老师我想问下，如何知道容器宿主机的地址，这样才可以在对数据包进行二次封装的时候把目的地址填写进去

  2018-12-29

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/10/78/29bd3f1e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王子瑞Aliloke有事电联

  老刘讲的太好了！！！这个专栏超值，开眼界了，而且讲的通俗易懂-虽然还有很多知识我没有懂，但开眼界了。

  2019-03-14

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/47/00/3202bdf0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  piboye

  vxlan要通过组播来实现广播，k8s有完整的集群信息，flannel是不是可以不用组播也能实现arp的解析？

  2022-08-18

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/rurTzy9obgda82kG3FTrszfzuIRQH2Mljc36u9KZLnOcJEtjY1NqdEROjpkLZia8Lu97OKhoIIicHu4xoiclHpOAA/132)

  Geek_536b07

  容器网络主流应该是k8s的网络实现，涉及到namespace 如果做pod内容器网络共享，如何通过svc访问pod，sts如何固定dns，没有结合k8s单独把网络插件拎出来讲，就很懵逼

  2022-08-11

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/rurTzy9obgda82kG3FTrszfzuIRQH2Mljc36u9KZLnOcJEtjY1NqdEROjpkLZia8Lu97OKhoIIicHu4xoiclHpOAA/132)

  Geek_536b07

  容器网络，讲的的太模糊了，容器集群注册的一般是podip，一般都会遇到跨集群访问的问题，这才是开发人员要去解决的重点

  2022-08-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/23/e8/9f445339.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  章潘

  ip，mac，vlan，vxlan等等都可以理解为对网络数据或者资源的标记。从这个角度出发，容器中的IP或端口冲突问题，是因为在同一个域用了相同的标签。所以要解决冲突问题，方式就太多了。选择不同的网段是方式之一。

  2022-06-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2a/f9/73/01eafd3c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  singularity of space time

  老师您好 在原文的表述中“物理机 B 上的 flanneld 收到包之后，解开 UDP 的包，将里面的网络包拿出来，从物理机 B 的 flannel.1 网卡发出去” 这里写“发出去”感觉并不恰当，它应该是写入/dev/net/tun设备，然后被flannel.1网卡接收到，再进入宿主机协议栈，然后经过路由发往docker0网卡，从docker0网卡中出去进入网关，再得到容器内部

  2022-02-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/45/83/93d389ba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我是谁

  老师好，有个问题。 28讲的时候，vxlan报文的外层ip和外层mac地址都是vtep设备的，但是这讲提到的flannel的vxlan方案中，外层ip和外层mac地址都是宿主机的，vtep设备的mac地址反而放在里内层，这是一个问题。另一个问题是内层mac地址不应该是虚拟机的吗？

  2021-03-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  在容器真实运用在实际环境中时，必须考虑通信的问题，运用flannel网络方案来解决问题，一个是UDP的用户态方案，一个是VXLAN内核态的方案，VXLAN方案在实际的使用中更常见。

  2020-03-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  VXLAN 模式的 VTEP 相较于 /dev/net/tun 字符设备网卡，性能改善是在 VTEP 通信比字符设备网卡通信少了内核态用户态间的数据拷贝？

  2019-12-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/bd/94/d4499319.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  allwmh

  flannel怎么做容器的IP保持呢？

  作者回复: 不保持

  2019-04-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e1/1a/20977779.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  峻铭

  如物理机A坏了，docker要迁移到C上去，物理机B内容器中的应用如何和C中的通信

  作者回复: 会重新配置

  2018-10-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8e/5a/2ed0cdec.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  donson

  flannel网络的地址段是10.242.0.0/16，默认每个Node上的子网划分是10.242.x.0/24，请问这个子网划分在哪里可以配置，比如我想划分成/21

  2018-09-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/84/ae/2b8192e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  selboo

  物理机A和物理机B是以access模式连接到交换机。此时flannel.1 在加上 VXLAN 头可以通信吗？？？求老师解答。

  2018-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/16/4d1e5cc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mgxian

  1.容器访问外面还是应该用nat 2.使用bgp协议的其他实现 calico kube-router

  2018-07-25

  **

  **
