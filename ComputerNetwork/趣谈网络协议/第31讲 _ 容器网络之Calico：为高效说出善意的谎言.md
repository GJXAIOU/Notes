# 第31讲 \| 容器网络之Calico：为高效说出善意的谎言

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/ec/7a/ec13f12833f302b5bdaca5bdbbb2907a.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/1c/98/1ca02d9aac71d9173ec414fd7ec34e98.mp3" type="audio/mpeg"></audio>

上一节我们讲了Flannel如何解决容器跨主机互通的问题，这个解决方式其实和虚拟机的网络互通模式是差不多的，都是通过隧道。但是Flannel有一个非常好的模式，就是给不同的物理机设置不同网段，这一点和虚拟机的Overlay的模式完全不一样。

在虚拟机的场景下，整个网段在所有的物理机之间都是可以“飘来飘去”的。网段不同，就给了我们做路由策略的可能。

## Calico网络模型的设计思路

我们看图中的两台物理机。它们的物理网卡是同一个二层网络里面的。由于两台物理机的容器网段不同，我们完全可以将两台物理机配置成为路由器，并按照容器的网段配置路由表。

![](<https://static001.geekbang.org/resource/image/19/37/1957b75dd689127c4621b5460c356137.jpg?wh=2250*1165>)

例如，在物理机A中，我们可以这样配置：要想访问网段172.17.9.0/24，下一跳是192.168.100.101，也即到物理机B上去。

这样在容器A中访问容器B，当包到达物理机A的时候，就能够匹配到这条路由规则，并将包发给下一跳的路由器，也即发给物理机B。在物理机B上也有路由规则，要访问172.17.9.0/24，从docker0的网卡进去即可。

当容器B返回结果的时候，在物理机B上，可以做类似的配置：要想访问网段172.17.8.0/24，下一跳是192.168.100.100，也即到物理机A上去。

<!-- [[[read_end]]] -->

当包到达物理机B的时候，能够匹配到这条路由规则，将包发给下一跳的路由器，也即发给物理机A。在物理机A上也有路由规则，要访问172.17.8.0/24，从docker0的网卡进去即可。

这就是**Calico网络的大概思路**，**即不走Overlay网络，不引入另外的网络性能损耗，而是将转发全部用三层网络的路由转发来实现**，只不过具体的实现和上面的过程稍有区别。

首先，如果全部走三层的路由规则，没必要每台机器都用一个docker0，从而浪费了一个IP地址，而是可以直接用路由转发到veth pair在物理机这一端的网卡。同样，在容器内，路由规则也可以这样设定：把容器外面的veth pair网卡算作默认网关，下一跳就是外面的物理机。

于是，整个拓扑结构就变成了这个图中的样子。

![](<https://static001.geekbang.org/resource/image/f4/58/f4fab81e3f981827577aa7790b78dc58.jpg?wh=2552*1558>)

## Calico网络的转发细节

我们来看其中的一些细节。

容器A1的IP地址为172.17.8.2/32，这里注意，不是/24，而是/32，将容器A1作为一个单点的局域网了。

容器A1里面的默认路由，Calico配置得比较有技巧。

```
default via 169.254.1.1 dev eth0 
169.254.1.1 dev eth0 scope link
```

这个IP地址169.254.1.1是默认的网关，但是整个拓扑图中没有一张网卡是这个地址。那如何到达这个地址呢？

前面我们讲网关的原理的时候说过，当一台机器要访问网关的时候，首先会通过ARP获得网关的MAC地址，然后将目标MAC变为网关的MAC，而网关的IP地址不会在任何网络包头里面出现，也就是说，没有人在乎这个地址具体是什么，只要能找到对应的MAC，响应ARP就可以了。

ARP本地有缓存，通过ip neigh命令可以查看。

```
169.254.1.1 dev eth0 lladdr ee:ee:ee:ee:ee:ee STALE
```

这个MAC地址是Calico硬塞进去的，但是没有关系，它能响应ARP，于是发出的包的目标MAC就是这个MAC地址。

在物理机A上查看所有网卡的MAC地址的时候，我们会发现veth1就是这个MAC地址。所以容器A1里发出的网络包，第一跳就是这个veth1这个网卡，也就到达了物理机A这个路由器。

在物理机A上有三条路由规则，分别是去两个本机的容器的路由，以及去172.17.9.0/24，下一跳为物理机B。

```
172.17.8.2 dev veth1 scope link 
172.17.8.3 dev veth2 scope link 
172.17.9.0/24 via 192.168.100.101 dev eth0 proto bird onlink
```

同理，物理机B上也有三条路由规则，分别是去两个本机的容器的路由，以及去172.17.8.0/24，下一跳为物理机A。

```
172.17.9.2 dev veth1 scope link 
172.17.9.3 dev veth2 scope link 
172.17.8.0/24 via 192.168.100.100 dev eth0 proto bird onlink
```

如果你觉得这些规则过于复杂，我将刚才的拓扑图转换为这个更加容易理解的图。

![](<https://static001.geekbang.org/resource/image/5f/47/5f23071c1e1b17cc46f1cb8955084247.jpg?wh=3070*1131>)

在这里，物理机化身为路由器，通过路由器上的路由规则，将包转发到目的地。在这个过程中，没有隧道封装解封装，仅仅是单纯的路由转发，性能会好很多。但是，这种模式也有很多问题。

## Calico的架构

### 路由配置组件Felix

如果只有两台机器，每台机器只有两个容器，而且保持不变。我手动配置一下，倒也没啥问题。但是如果容器不断地创建、删除，节点不断地加入、退出，情况就会变得非常复杂。

![](<https://static001.geekbang.org/resource/image/f2/31/f29027cca71f3dfbba8c2f1a35c29331.jpg?wh=1895*876>)

就像图中，有三台物理机，两两之间都需要配置路由，每台物理机上对外的路由就有两条。如果有六台物理机，则每台物理机上对外的路由就有五条。新加入一个节点，需要通知每一台物理机添加一条路由。

这还是在物理机之间，一台物理机上，每创建一个容器，也需要多配置一条指向这个容器的路由。如此复杂，肯定不能手动配置，需要每台物理机上有一个agent，当创建和删除容器的时候，自动做这件事情。这个agent在Calico中称为Felix。

### 路由广播组件BGP Speaker

当Felix配置了路由之后，接下来的问题就是，如何将路由信息，也即将“如何到达我这个节点，访问我这个节点上的容器”这些信息，广播出去。

能想起来吗？这其实就是路由协议啊！路由协议就是将“我能到哪里，如何能到我”的信息广播给全网传出去，从而客户端可以一跳一跳地访问目标地址的。路由协议有很多种，Calico使用的是BGP协议。

在Calico中，每个Node上运行一个软件BIRD，作为BGP的客户端，或者叫作BGP Speaker，将“如何到达我这个Node，访问我这个Node上的容器”的路由信息广播出去。所有Node上的BGP Speaker 都互相建立连接，就形成了全互连的情况，这样每当路由有所变化的时候，所有节点就都能够收到了。

### 安全策略组件

Calico中还实现了灵活配置网络策略Network Policy，可以灵活配置两个容器通或者不通。这个怎么实现呢？

![](<https://static001.geekbang.org/resource/image/dd/f2/ddae28956780cc3e45fde76ae96701f2.jpg?wh=3516*1434>)

虚拟机中的安全组，是用iptables实现的。Calico中也是用iptables实现的。这个图里的内容是iptables在内核处理网络包的过程中可以嵌入的处理点。Calico也是在这些点上设置相应的规则。

![](<https://static001.geekbang.org/resource/image/8f/3f/8f2be6638615fc5501c460d1206bff3f.jpg?wh=3977*1818>)

当网络包进入物理机上的时候，进入PREOUTING规则，这里面有一个规则是cali-fip-dnat，这是实现浮动IP（Floating IP）的场景，主要将外网的IP地址dnat作为容器内的IP地址。在虚拟机场景下，路由器的网络namespace里面有一个外网网卡上，也设置过这样一个DNAT规则。

接下来可以根据路由判断，是到本地的，还是要转发出去的。

如果是本地的，走INPUT规则，里面有个规则是cali-wl-to-host，wl的意思是workload，也即容器，也即这是用来判断从容器发到物理机的网络包是否符合规则的。这里面内嵌一个规则cali-from-wl-dispatch，也是匹配从容器来的包。如果有两个容器，则会有两个容器网卡，这里面内嵌有详细的规则“cali-fw-cali网卡1”和“cali-fw-cali网卡2”，fw就是from workload，也就是匹配从容器1来的网络包和从容器2来的网络包。

如果是转发出去的，走FORWARD规则，里面有个规则cali-FORWARD。这里面分两种情况，一种是从容器里面发出来，转发到外面的；另一种是从外面发进来，转发到容器里面的。

第一种情况匹配的规则仍然是cali-from-wl-dispatch，也即from workload。第二种情况匹配的规则是cali-to-wl-dispatch，也即to workload。如果有两个容器，则会有两个容器网卡，在这里面内嵌有详细的规则“cali-tw-cali网卡1”和“cali-tw-cali网卡2”，tw就是to workload，也就是匹配发往容器1的网络包和发送到容器2的网络包。

接下来是匹配OUTPUT规则，里面有cali-OUTPUT。接下来是POSTROUTING规则，里面有一个规则是cali-fip-snat，也即发出去的时候，将容器网络IP转换为浮动IP地址。在虚拟机场景下，路由器的网络namespace里面有一个外网网卡上，也设置过这样一个SNAT规则。

至此为止，Calico的所有组件基本凑齐。来看看我汇总的图。

![](<https://static001.geekbang.org/resource/image/71/07/71f22fd9e8336c7e10c8ff7bd276af07.jpg?wh=3098*3650>)

## 全连接复杂性与规模问题

这里面还存在问题，就是BGP全连接的复杂性问题。

你看刚才的例子里只有六个节点，BGP的互连已经如此复杂，如果节点数据再多，这种全互连的模式肯定不行，到时候都成蜘蛛网了。于是多出了一个组件BGP Route Reflector，它也是用BIRD实现的。有了它，BGP Speaker就不用全互连了，而是都直连它，它负责将全网的路由信息广播出去。

可是问题来了，规模大了，大家都连它，它受得了吗？这个BGP Router Reflector会不会成为瓶颈呢？

所以，肯定不能让一个BGP Router Reflector管理所有的路由分发，而是应该有多个BGP Router Reflector，每个BGP Router Reflector管一部分。

多大算一部分呢？咱们讲述数据中心的时候，说服务器都是放在机架上的，每个机架上最顶端有个TOR交换机。那将机架上的机器连在一起，这样一个机架是不是可以作为一个单元，让一个BGP Router Reflector来管理呢？如果要跨机架，如何进行通信呢？这就需要BGP Router Reflector也直接进行路由交换。它们之间的交换和一个机架之间的交换有什么关系吗？

有没有觉得在这个场景下，一个机架就像一个数据中心，可以把它设置为一个AS，而BGP Router Reflector有点儿像数据中心的边界路由器。在一个AS内部，也即服务器和BGP Router Reflector之间使用的是数据中心内部的路由协议iBGP，BGP Router Reflector之间使用的是数据中心之间的路由协议eBGP。

![](<https://static001.geekbang.org/resource/image/47/a0/474cb05d5536f11d75baeb6332d788a0.jpg?wh=3918*2408>)

这个图中，一个机架上有多台机器，每台机器上面启动多个容器，每台机器上都有可以到达这些容器的路由。每台机器上都启动一个BGP Speaker，然后将这些路由规则上报到这个Rack上接入交换机的BGP Route Reflector，将这些路由通过iBGP协议告知到接入交换机的三层路由功能。

在接入交换机之间也建立BGP连接，相互告知路由，因而一个Rack里面的路由可以告知另一个Rack。有多个核心或者汇聚交换机将接入交换机连接起来，如果核心和汇聚起二层互通的作用，则接入和接入之间之间交换路由即可。如果核心和汇聚交换机起三层路由的作用，则路由需要通过核心或者汇聚交换机进行告知。

## 跨网段访问问题

上面的Calico模式还有一个问题，就是跨网段问题，这里的跨网段是指物理机跨网段。

前面我们说的那些逻辑成立的条件，是我们假设物理机可以作为路由器进行使用。例如物理机A要告诉物理机B，你要访问172.17.8.0/24，下一跳是我192.168.100.100；同理，物理机B要告诉物理机A，你要访问172.17.9.0/24，下一跳是我192.168.100.101。

之所以能够这样，是因为物理机A和物理机B是同一个网段的，是连接在同一个交换机上的。那如果物理机A和物理机B不是在同一个网段呢？

![](<https://static001.geekbang.org/resource/image/58/89/58bb1d0965c383b1eaac06946998f089.jpg?wh=2773*1756>)<br>

![](<https://static001.geekbang.org/resource/image/1b/37/1b4514ed6d0e952a9d14f55yy36c0937.jpg?wh=3047*1018>)

例如，物理机A的网段是192.168.100.100/24，物理机B的网段是192.168.200.101/24，这样两台机器就不能通过二层交换机连接起来了，需要在中间放一台路由器，做一次路由转发，才能跨网段访问。

本来物理机A要告诉物理机B，你要访问172.17.8.0/24，下一跳是我192.168.100.100的，但是中间多了一台路由器，下一跳不是我了，而是中间的这台路由器了，这台路由器的再下一跳，才是我。这样之前的逻辑就不成立了。

我们看刚才那张图的下半部分。物理机B上的容器要访问物理机A上的容器，第一跳就是物理机B，IP为192.168.200.101，第二跳是中间的物理路由器右面的网口，IP为192.168.200.1，第三跳才是物理机A，IP为192.168.100.100。

这是咱们通过拓扑图看到的，关键问题是，在系统中物理机A如何告诉物理机B，怎么让它才能到我这里？物理机A根本不可能知道从物理机B出来之后的下一跳是谁，况且现在只是中间隔着一个路由器这种简单的情况，如果隔着多个路由器呢？谁能把这一串的路径告诉物理机B呢？

我们能想到的第一种方式是，让中间所有的路由器都来适配Calico。本来它们互相告知路由，只互相告知物理机的，现在还要告知容器的网段。这在大部分情况下，是不可能的。

第二种方式，还是在物理机A和物理机B之间打一个隧道，这个隧道有两个端点，在端点上进行封装，将容器的IP作为乘客协议放在隧道里面，而物理主机的IP放在外面作为承载协议。这样不管外层的IP通过传统的物理网络，走多少跳到达目标物理机，从隧道两端看起来，物理机A的下一跳就是物理机B，这样前面的逻辑才能成立。

这就是Calico的**IPIP模式**。使用了IPIP模式之后，在物理机A上，我们能看到这样的路由表：

```
172.17.8.2 dev veth1 scope link 
172.17.8.3 dev veth2 scope link 
172.17.9.0/24 via 192.168.200.101 dev tun0 proto bird onlink
```

这和原来模式的区别在于，下一跳不再是同一个网段的物理机B了，IP为192.168.200.101，并且不是从eth0跳，而是建立一个隧道的端点tun0，从这里才是下一跳。

如果我们在容器A1里面的172.17.8.2，去ping容器B1里面的172.17.9.2，首先会到物理机A。在物理机A上根据上面的规则，会转发给tun0，并在这里对包做封装：

- 内层源IP为172.17.8.2；

- 内层目标IP为172.17.9.2；

- 外层源IP为192.168.100.100；

- 外层目标IP为192.168.200.101。


<!-- -->

将这个包从eth0发出去，在物理网络上会使用外层的IP进行路由，最终到达物理机B。在物理机B上，tun0会解封装，将内层的源IP和目标IP拿出来，转发给相应的容器。

## 小结

好了，这一节就到这里，我们来总结一下。

- Calico推荐使用物理机作为路由器的模式，这种模式没有虚拟化开销，性能比较高。

- Calico的主要组件包括路由、iptables的配置组件Felix、路由广播组件BGP Speaker，以及大规模场景下的BGP Route Reflector。

- 为解决跨网段的问题，Calico还有一种IPIP模式，也即通过打隧道的方式，从隧道端点来看，将本来不是邻居的两台机器，变成相邻的机器。


<!-- -->

最后，给你留两个思考题：

1. 将Calico部署在公有云上的时候，经常会选择使用IPIP模式，你知道这是为什么吗？

2. 容器是用来部署微服务的，微服务之间的通信，除了网络要互通，还需要高效地传输信息，例如下单的商品、价格、数量、支付的钱等等，这些要通过什么样的协议呢？


<!-- -->

我们的专栏更新到第31讲，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送<span class="orange">学习奖励礼券</span>

和我整理的<span class="orange">独家网络协议知识图谱</span>

。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(45)

- ![img](%E7%AC%AC31%E8%AE%B2%20_%20%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C%E4%B9%8BCalico%EF%BC%9A%E4%B8%BA%E9%AB%98%E6%95%88%E8%AF%B4%E5%87%BA%E5%96%84%E6%84%8F%E7%9A%84%E8%B0%8E%E8%A8%80.resource/resize,m_fill,h_34,w_34.jpeg)

  sunlight001

  工作中接触不到，现在完全看不明白的举个手！

  2018-07-27

  **3

  **220

- ![img](%E7%AC%AC31%E8%AE%B2%20_%20%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C%E4%B9%8BCalico%EF%BC%9A%E4%B8%BA%E9%AB%98%E6%95%88%E8%AF%B4%E5%87%BA%E5%96%84%E6%84%8F%E7%9A%84%E8%B0%8E%E8%A8%80.resource/resize,m_fill,h_34,w_34-1662308202804-9922.jpeg)

  固态U盘

  感觉老师的这个课程标价有点低

  2018-07-27

  **2

  **43

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c1/60/fc3689d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小谢同学

  请问老师，在IPIP模式下，原本的性能优势是否又回退到了overlay类似呢？

  作者回复: 是的

  2019-01-24

  **4

  **21

- ![img](https://static001.geekbang.org/account/avatar/00/0f/44/f2/e7158fa0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张稀虹

  提个建议 老师能不能在下一期文章出来的时候在前一期文章中更新问题的答案，感觉比较深的话题讨论区的讨论就比较少了

  2018-07-28

  **

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/16/4d1e5cc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mgxian

  1.因为公有云中的虚拟机之间 不直接通过交换机互通 中间有路由 使用VPC有可能可以实现 2.微服务数据交换现在有两种主流方式 http 和 rpc

  2018-07-27

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/65/25/c6de04bc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  斜月浮云![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  现在提问还来得及吗？问下ipip模式中，隧道是点到点的？那么如果服务部署到两个或多个局域网，每个局域网有n台机器，那么为了保证互相跨网互通，是否需求全部点对点打隧道？是否资源损毁太大了？怎么优化？

  作者回复: 使用路由，汇聚到物理路由器，物理路由器之间打通

  2018-10-14

  **2

  **7

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  kissingers

  我们能想到的第一种方式是，让中间所有的路由器都来适配 Calico。本来它们互相告知路由，只互相告知物理机的，现在还要告知容器的网段。 这种情况为什么不行?  BGP也能交换容器网段和路由信息吧

  作者回复: 对啊，中间的都适配calico，但是公司网管肯定不干

  2019-04-17

  **2

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/12/b4/00/bfc101ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Tendrun

  老师您好，不太明白为什么当物理机A、B跨网断中间有多个路由器时不可用。如果路由器都支持BGP的话，且A、B之间可通。那么把A、B配置成bgp对端，应该就可以分发容器的路由，然后跨节点的容器就可以通信了吧

  作者回复: A-router-B，这样的话，B肯定会告诉A，如果想访问某个容器，router是下一跳，但问题是包到了router，router是物理的呀，他又不知道容器的那个段，他往哪里转发呀。除非router也配置了bgp

  2019-04-20

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/0e/7c64101d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  $(CF_HB)

  不是很明白，回去公司打开个wireshark分析一下网络。分析一下路由协议。

  2018-10-31

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/13/58bcac86.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  silenceper

  不支持bgp协议，是不是就用不了calico?

  2018-09-13

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/7b/9e/37d69ff0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  balancer

  老师，网络数据包到达网卡的时候，内核怎么定位这个数据包属于哪个进程的那个socketfd？

  2018-07-29

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8b/64/d8bf2f6f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  旻言

  如果能在路由器上自动配置去各个容器网络的路由规则，是不是垮网段访问的问题就不成立，就不会有IPIP这种overlay带来的开销呢？

  2021-01-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/ef/66/dc3f4555.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赛飞

  作为一名前端工程师，表示听的有点懵了😂

  2020-11-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/17/5d/ff/b58dd422.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  星星⭐

  刘老师，是tun0还是tunl0?

  2020-09-21

  **1

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_042531

  公有云使用ipip这样的overlay 技术：1.实现大二层网络，构建突破vlan 4K的限制，2.容器跨3层迁移

  2019-09-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/5b/52/fea5ec99.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  俊飞

  听了老师讲解明白多了，我们的k8s1.13.0集群里面用到的就是擦calico网络，当时还在纳闷k8s怎么不用flannel网络了，现在明白了，calico网络性能要比flannel高很多，采用ip-in-ip的方式进行隧道通信，安全性也提高不少。

  作者回复: 后来flannel也支持calico的所有模式了

  2019-01-06

  **3

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c2/e0/7188aa0a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  blackpiglet

  \1. 公有云上跨网段的现象很常见，而且如果涉及到跨数据中心，中间会经过很多路由设备，只能通过隧道方式打通连接。 2. 微服务间的信息交互主要是 RPC 和 RESTful。

  2018-08-01

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJcwXucibksEYRSYg6icjibzGa7efcMrCsGec2UwibjTd57icqDz0zzkEEOM2pXVju60dibzcnQKPfRkN9g/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Geek_93970d

  和 vxlan 有何区别？vxlan 用 udp，IPIP 呢？ vxlan 的乘客协议是 二层协议，承载协议是 UDP，通常用于解决具有相同网段但是分布在不同物理机上的 docker 之间的通信（当然如果docker网段不同也是可以的，比如 Flannel）。 IPIP 的乘客协议和承载协议都是 IP，即用 IP 封装 IP，用于解决不同物理机上不同网段的 docker 之间的通信，因为是不同网段，所以必然需要通过路由转发，通过两种方式可以实现：要么用物理机当路由器使，要么通过 IPIP 隧道，前一种方式需要限制物理机必须同网段，所以才产生后一种方式。

  2022-08-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/23/e8/9f445339.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  章潘

  虚拟机网络场景与容器场景其中一个比较大的区别是虚拟机采用了绝对的网络隔离技术，如vlan，vxlan等，所以虚拟机网段重叠很普遍。

  2022-06-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/83/fb/621adceb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  linker

  思考题1  有的公有云使用ARP代答，所以获取到的对端虚拟机的Mac地址不是真实的Mac，不能把虚拟机作为一个路由器使用，使用要连接隧道

  2021-11-23

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erCibehm9W3tbhKic1RnbTvPVCgWDmludx9YQ97BneVRhyegkr13R6vrFPYol4IYEF98s07MicgOtS0g/132)

  hao

  刘老师，如果用阿里云，需要考虑这些吗？

  2021-09-27

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/6cyOoRd2dROgiblAJkW6RLhUyH1wwU0NNibIIuV930eQ9TiaNT41K61kBSVkvYoDYg7mJtuEoCQY1awBmV0WW6BFg/132)

  大方方

  非网络专业从业者，实话实说从第三模块的第五讲开始就已经完全out了，现在都是硬着头皮去尝试理解大概的思路，力争刷过第一遍以后二刷细嚼。

  2021-08-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/6d/910b2445.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fastkdm

  想问下，Calico的IPIP模式，如果每两个物理机都要建立隧道的话，会导致隧道过多成网状结构吗

  2021-02-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/15/86/cd97bf7e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  戴宇

  我觉得第二个图是不是画得不太准确，和后面那个全连接的图也对不上，从物理机A经过路由规则到172.17.9.0/24不能直接连到物理机B上面吧。这样看起来这个网络包发出去都没有经过物理机A自己的网卡。这里是性能损耗是不是指从docker0到eth0出去做SNAT的性能损耗？

  2020-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/15/86/cd97bf7e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  戴宇

  有疑问，这个calico还是不是overlay 网络， 我查了https://docs.projectcalico.org/networking/vxlan-ipip，上面还是说这个calico ipip是一个overlay网络，我也觉得是，因为跨网段要建立隧道，这个隧道就很像GRE。

  2020-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/3c/84/608f679b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  连边

  我觉得，要去部署一下微服务环境，然后回过头来看这些文章，才会有感触。

  2020-08-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/db/0b/6aa6f653.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  侯聪

  IPIP模式下的包封装是直接把容器A发出来的payload作为应用层内容吗？  这样的话在物理路由器上不需要支持VXLAN或者Geneve等overlay封装协议了

  2020-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/73/90/9118f46d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chenhz

  问题1:公有云分配到的 ip 可能不是同一网段的，通过IPIP模式的隧道模式可以实现跨网段的物理机里的容器互联互通； 问题2:微服务之间通信协议主要为TCP和HTTP，序列化和反序列化主流方案为为 protobuf 和 JSON，通过注册中心服务发现的机制，通过服务名访问。

  2020-07-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/72/85/c337e9a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  老兵

  \1. Calico部署到公网使用IPIP模式，主要因为容器或者应用之间可能分布着很多路由，两个应用，或者两个容器无法知道从物理机出来的下一跳是不是目标物理机，所以使用IPIP模式，在机器之间启动一个tunnel。 2. 微服务间的传输主要是 RPC 和 RESTful

  2020-06-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  jackwu

  请问 ipip 模式丢包问题有同学遇到过吗？

  2020-05-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/83/af/1cb42cd3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  马以

  "将这个包从 eth0 发出去，在物理网络上会使用外层的 IP 进行路由，最终到达物理机 B。在物理机 B 上，tun0 会解封装，将内层的源 IP 和目标 IP 拿出来，转发给相应的容器。” 最后一段这里，包是从eth0发出去走传统的路由，还是走tun0通道，直接到达物理机B，这里是笔误还是什么？求解答～

  2020-04-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  容器跨主机跨网段访问，Calico应用的组合知识点来解决这些通信问题，不管是iptables的还是BGP中，最终组合起来形成解决方案。

  2020-03-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/60/d9/829ac53b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fangxuan

  老师，calico中的bird是不是有限制，如果bgp通告过来的路由下一跳不是bgp邻居的ip，就会把路由标记为不可达？

  2019-11-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/0c/30/bb4bfe9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  lyonger

  越到后面越抽象了，要实践老师讲的知识，估计需要从事云计算或容器相关岗位，有业务场景才能理解更深刻。

  2019-11-12

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e2/d8/f0562ede.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  manatee

  想请教一下老师，这个讲的网络都是容器与容器之间的网络，那容器与虚拟机，虚拟机与容器之间的网络是怎么样的呢

  2019-10-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0e/e9/98b6ea61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员大天地

  没接触过，看来只有等后面用到才能看懂

  作者回复: 很容易自己搭建一个的

  2019-08-29

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/87/64/3882d90d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yandongxiao

  超哥给的信息量真的是太大了，越品越精彩。

  作者回复: 谢谢

  2019-08-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a0/57/3a729755.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灯盖

  强大

  2019-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/6e/8b/60095195.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  功夫

  容器 网络越来越抽象类

  作者回复: 对的，但是原理还是那么多

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/85/ed/905b052f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  超超

  回答问题一：ipip模式由于是走隧道协议，安全性比较好！ 回答问题二：RPC

  2019-04-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c7/40/66a203cd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  江南皮革厂研发中心保安队长

  服务之间的通信我们自己http RESTful 风格的比较多 

  作者回复: 现在用restful的多了

  2019-03-18

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/47/62/25a4aeae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  丁丁

  觉得很高深，学会了会提升好多啊

  2019-03-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/28/d3/85b6534d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Dangdang

  弱弱的问一句，l2，l3，几层是怎么分的？？？

  2019-02-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b6/11/e8506a04.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小宇宙

  1 公有云厂商不一定实现大二层网络，容器的宿主机也不一定放在同一个vlan或者网段中，因此要建隧道了，还有一点，建隧道以后还可以实现容器自由迁移，保持容器的mac地址和ip地址不变。

  2018-08-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/92/01/c723d180.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饼子

  1 公有云上面会夸机房使用 2 微服务通信，基于tcp的rpc 调用，

  2018-08-21

  **

  **
