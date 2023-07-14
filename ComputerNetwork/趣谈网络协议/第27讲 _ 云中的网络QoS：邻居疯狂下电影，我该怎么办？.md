# 第27讲 \| 云中的网络QoS：邻居疯狂下电影，我该怎么办？

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/2d/58/2d5bd57702ed4d573e064fe58e5eb858.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/00/26/006be4d2d2e2e70b6063bdd40c1dbc26.mp3" type="audio/mpeg"></audio>

在小区里面，是不是经常有住户不自觉就霸占公共通道，如果你找他理论，他的话就像一个相声《楼道曲》说的一样：“公用公用，你用我用，大家都用，我为什么不能用？”。

除此之外，你租房子的时候，有没有碰到这样的情况：本来合租共享WiFi，一个人狂下小电影，从而你网都上不去，是不是很懊恼？

在云平台上，也有这种现象，好在有一种流量控制的技术，可以实现**QoS**（Quality of Service），从而保障大多数用户的服务质量。

对于控制一台机器的网络的QoS，分两个方向，一个是入方向，一个是出方向。

![](<https://static001.geekbang.org/resource/image/74/11/747b0d537fd1705171ffcca3faf96211.jpg?wh=1539*646>)

其实我们能控制的只有出方向，通过Shaping，将出的流量控制成自己想要的模样。而进入的方向是无法控制的，只能通过Policy将包丢弃。

## 控制网络的QoS有哪些方式？

在Linux下，可以通过TC控制网络的QoS，主要就是通过队列的方式。

### 无类别排队规则

第一大类称为**无类别排队规则**（Classless Queuing Disciplines）。还记得我们讲[ip addr](<https://time.geekbang.org/column/article/7772>)的时候讲过的**pfifo\_fast**，这是一种不把网络包分类的技术。

![](<https://static001.geekbang.org/resource/image/e3/6c/e391b4b79580a7d66afe4307ff3f6f6c.jpg?wh=2037*1175>)

pfifo\_fast分为三个先入先出的队列，称为三个Band。根据网络包里面TOS，看这个包到底应该进入哪个队列。TOS总共四位，每一位表示的意思不同，总共十六种类型。

<!-- [[[read_end]]] -->

通过命令行tc qdisc show dev eth0，可以输出结果priomap，也是十六个数字。在0到2之间，和TOS的十六种类型对应起来，表示不同的TOS对应的不同的队列。其中Band 0优先级最高，发送完毕后才轮到Band 1发送，最后才是Band 2。

另外一种无类别队列规则叫作**随机公平队列**（Stochastic Fair Queuing）。

![](<https://static001.geekbang.org/resource/image/b6/99/b6ec2e4e20ddee7d6952b7fa4586ba99.jpg?wh=2177*1182>)

会建立很多的FIFO的队列，TCP Session会计算hash值，通过hash值分配到某个队列。在队列的另一端，网络包会通过轮询策略从各个队列中取出发送。这样不会有一个Session占据所有的流量。

当然如果两个Session的hash是一样的，会共享一个队列，也有可能互相影响。hash函数会经常改变，从而session不会总是相互影响。

还有一种无类别队列规则称为**令牌桶规则**（TBF，Token Bucket Filte）。

![](<https://static001.geekbang.org/resource/image/14/9b/145c6f8593bf7603eae79246b9d6859b.jpg?wh=1894*1100>)

所有的网络包排成队列进行发送，但不是到了队头就能发送，而是需要拿到令牌才能发送。

令牌根据设定的速度生成，所以即便队列很长，也是按照一定的速度进行发送的。

当没有包在队列中的时候，令牌还是以既定的速度生成，但是不是无限累积的，而是放满了桶为止。设置桶的大小为了避免下面的情况：当长时间没有网络包发送的时候，积累了大量的令牌，突然来了大量的网络包，每个都能得到令牌，造成瞬间流量大增。

### 基于类别的队列规则

另外一大类是**基于类别的队列规则**（Classful Queuing Disciplines），其中典型的为**分层令牌桶规则**（**HTB**， Hierarchical Token Bucket）。

HTB往往是一棵树，接下来我举个具体的例子，通过TC如何构建一棵HTB树来带你理解。

![](<https://static001.geekbang.org/resource/image/e6/f5/e6de57bf00f2fe8865ec3548bf8c67f5.jpg?wh=2042*1852>)

使用TC可以为某个网卡eth0创建一个HTB的队列规则，需要付给它一个句柄为（1:）。

这是整棵树的根节点，接下来会有分支。例如图中有三个分支，句柄分别为（:10）、（:11）、（:12）。最后的参数default 12，表示默认发送给1:12，也即发送给第三个分支。

```
tc qdisc add dev eth0 root handle 1: htb default 12
```

对于这个网卡，需要规定发送的速度。一般有两个速度可以配置，一个是**rate**，表示一般情况下的速度；一个是**ceil**，表示最高情况下的速度。对于根节点来讲，这两个速度是一样的，于是创建一个root class，速度为（rate=100kbps，ceil=100kbps）。

```
tc class add dev eth0 parent 1: classid 1:1 htb rate 100kbps ceil 100kbps
```

接下来要创建分支，也即创建几个子class。每个子class统一有两个速度。三个分支分别为（rate=30kbps，ceil=100kbps）、（rate=10kbps，ceil=100kbps）、（rate=60kbps，ceil=100kbps）。

```
tc class add dev eth0 parent 1:1 classid 1:10 htb rate 30kbps ceil 100kbps
tc class add dev eth0 parent 1:1 classid 1:11 htb rate 10kbps ceil 100kbps
tc class add dev eth0 parent 1:1 classid 1:12 htb rate 60kbps ceil 100kbps
```

你会发现三个rate加起来，是整个网卡允许的最大速度。

HTB有个很好的特性，同一个root class下的子类可以相互借流量，如果不直接在队列规则下面创建一个root class，而是直接创建三个class，它们之间是不能相互借流量的。借流量的策略，可以使得当前不使用这个分支的流量的时候，可以借给另一个分支，从而不浪费带宽，使带宽发挥最大的作用。

最后，创建叶子队列规则，分别为**fifo**和**sfq**。

```
tc qdisc add dev eth0 parent 1:10 handle 20: pfifo limit 5
tc qdisc add dev eth0 parent 1:11 handle 30: pfifo limit 5
tc qdisc add dev eth0 parent 1:12 handle 40: sfq perturb 10
```

基于这个队列规则，我们还可以通过TC设定发送规则：从1.2.3.4来的，发送给port 80的包，从第一个分支1:10走；其他从1.2.3.4发送来的包从第二个分支1:11走；其他的走默认分支。

```
tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 1.2.3.4 match ip dport 80 0xffff flowid 1:10
tc filter add dev eth0 protocol ip parent 1:0 prio 1 u32 match ip src 1.2.3.4 flowid 1:11
```

## 如何控制QoS？

我们讲过，使用OpenvSwitch将云中的网卡连通在一起，那如何控制QoS呢？

就像我们上面说的一样，OpenvSwitch支持两种：

- 对于进入的流量，可以设置策略Ingress policy；

<!-- -->

```
ovs-vsctl set Interface tap0 ingress_policing_rate=100000
ovs-vsctl set Interface tap0 ingress_policing_burst=10000
```

- 对于发出的流量，可以设置QoS规则Egress shaping，支持HTB。

<!-- -->

我们构建一个拓扑图，来看看OpenvSwitch的QoS是如何工作的。

![](<https://static001.geekbang.org/resource/image/3b/ee/3b0de72bc937e108519a473067f607ee.jpg?wh=1945*1985>)

首先，在port上可以创建QoS规则，一个QoS规则可以有多个队列Queue。

![](<https://static001.geekbang.org/resource/image/e6/84/e65435bde65a255a085f10d02d2ff184.jpg?wh=2184*1918>)

```
ovs-vsctl set port first_br qos=@newqos -- --id=@newqos create qos type=linux-htb other-config:max-rate=10000000 queues=0=@q0,1=@q1,2=@q2 -- --id=@q0 create queue other-config:min-rate=3000000 other-config:max-rate=10000000 -- --id=@q1 create queue other-config:min-rate=1000000 other-config:max-rate=10000000 -- --id=@q2 create queue other-config:min-rate=6000000 other-config:max-rate=10000000
```

上面的命令创建了一个QoS规则，对应三个Queue。min-rate就是上面的rate，max-rate就是上面的ceil。通过交换机的网络包，要通过流表规则，匹配后进入不同的队列。然后我们就可以添加流表规则Flow(first\_br是br0上的port 5)。

```
ovs-ofctl add-flow br0 "in_port=6 nw_src=192.168.100.100 actions=enqueue:5:0"
ovs-ofctl add-flow br0 "in_port=7 nw_src=192.168.100.101 actions=enqueue:5:1"
ovs-ofctl add-flow br0 "in_port=8 nw_src=192.168.100.102 actions=enqueue:5:2"
```

接下来，我们单独测试从192.168.100.100，192.168.100.101，192.168.100.102到192.168.100.103的带宽的时候，每个都是能够打满带宽的。

如果三个一起测试，一起狂发网络包，会发现是按照3:1:6的比例进行的，正是根据配置的队列的带宽比例分配的。

如果192.168.100.100和192.168.100.101一起测试，发现带宽占用比例为3:1，但是占满了总的流量，也即没有发包的192.168.100.102有60%的带宽被借用了。

如果192.168.100.100和192.168.100.102一起测试，发现带宽占用比例为1:2。如果192.168.100.101和192.168.100.102一起测试，发现带宽占用比例为1:6。

## 小结

好了，这一节就讲到这里了，我们来总结一下。

- 云中的流量控制主要通过队列进行的，队列分为两大类：无类别队列规则和基于类别的队列规则。

- 在云中网络Openvswitch中，主要使用的是分层令牌桶规则（HTB），将总的带宽在一棵树上按照配置的比例进行分配，并且在一个分支不用的时候，可以借给另外的分支，从而增强带宽利用率。


<!-- -->

最后，给你留两个思考题。

1. 这一节中提到，入口流量其实没有办法控制，出口流量是可以很好控制的，你能想出一个控制云中的虚拟机的入口流量的方式吗？

2. 安全性和流量控制大概解决了，但是不同用户在物理网络的隔离还是没有解决，你知道怎么解决吗？


<!-- -->

我们的专栏更新到第27讲，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送<span class="orange">学习奖励礼券</span>

和我整理的<span class="orange">独家网络协议知识图谱</span>

。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(26)

- ![img](%E7%AC%AC27%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9CQoS%EF%BC%9A%E9%82%BB%E5%B1%85%E7%96%AF%E7%8B%82%E4%B8%8B%E7%94%B5%E5%BD%B1%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.resource/resize,m_fill,h_34,w_34.jpeg)

  Hurt

  云里雾里 不是科班 感觉要补的东西太多了

  2018-07-18

  **5

  **36

- ![img](https://static001.geekbang.org/account/avatar/00/10/2f/c5/aaacb98f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yungoo

  通过ingress qdisc策略将入口流量重定向到虚拟网卡ifb，然后对ifb的egress进行出口限速，从而变通实现入口流控。

  2018-07-18

  **1

  **20

- ![img](https://static001.geekbang.org/account/avatar/00/11/52/e0/ef42c4ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓.光

  越来越发现云中的网络控制跟本地原理一致~

  作者回复: 对啊，所以原理是通的

  2018-07-18

  **

  **16

- ![img](%E7%AC%AC27%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9CQoS%EF%BC%9A%E9%82%BB%E5%B1%85%E7%96%AF%E7%8B%82%E4%B8%8B%E7%94%B5%E5%BD%B1%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662308110604-9557.jpeg)

  zcpromising

  前面15讲以前的内容，在学校是可以接触到的，后面每讲的内容，在学校是体会不到的，每天听老师您的课程，感觉就像发现了新大陆一样，惊喜万分。要是学校老师能够按照您这样的方式讲，那该多好。

  2018-07-18

  **

  **10

- ![img](%E7%AC%AC27%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9CQoS%EF%BC%9A%E9%82%BB%E5%B1%85%E7%96%AF%E7%8B%82%E4%B8%8B%E7%94%B5%E5%BD%B1%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662308110606-9558.jpeg)

  超超

  答问题1：是否可以在虚拟机的前一级控制出口流量？前一级的出口流量得到控制，那么虚拟机的入口流量也就得到了控制。

  作者回复: 是的，可以联动

  2019-04-07

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/c8/6b/0f3876ef.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iron_man

  虚拟机的流量都通过openv switch控制，机器数量多了，openvswitch会不会成为一个瓶颈

  2018-08-08

  **

  **6

- ![img](%E7%AC%AC27%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9CQoS%EF%BC%9A%E9%82%BB%E5%B1%85%E7%96%AF%E7%8B%82%E4%B8%8B%E7%94%B5%E5%BD%B1%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662308110608-9560.jpeg)

  Fisher

  这篇文章控制的出口流量是控制网卡层面的，那么像标题里面的，如果是在局域网中别人疯狂下载，这种控制网速的是在哪个层面控制的，路由器本身控制速度的原理又是什么呢，只是控制转发速度吗

  2018-07-21

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/11/66/34/0508d9e4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  u

  超哥，有个问题想问下：你平时代码写的多吗？

  作者回复: 现在写的少了

  2018-08-02

  **

  **5

- ![img](%E7%AC%AC27%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9CQoS%EF%BC%9A%E9%82%BB%E5%B1%85%E7%96%AF%E7%8B%82%E4%B8%8B%E7%94%B5%E5%BD%B1%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662308110608-9562.jpeg)

  Gabriel

  嗯 什么就硬着头皮也要看 我现在就是

  2021-03-12

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/eb/48/c7aad9d6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  HIK

  请问如何实现疯狂发包？

  作者回复: 性能测试软件都可以

  2018-07-18

  **3

  **2

- ![img](%E7%AC%AC27%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%9A%84%E7%BD%91%E7%BB%9CQoS%EF%BC%9A%E9%82%BB%E5%B1%85%E7%96%AF%E7%8B%82%E4%B8%8B%E7%94%B5%E5%BD%B1%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662308110609-9564.jpeg)

  King-ZJ

  QOS服务质量在业务运行中做一个按需的配置，更好调节网络的运行流畅度，带给客户更好的体验。

  2020-03-25

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Cib5umA0W17N9pichI08pnrXAExdbyh7AVzH4nEhD6KN3FXuELk4LJJuqUPPD7xmIy9nq5Hjbgnzic7sVZG5BKiaUQ/132)

  被过去推开

  很多网关都提供了基于令牌桶模式的限流，比如spring cloud gateway

  2019-10-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e4/8b/8a0a6c86.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  haha

  几种队列的控制策略，放到哪合适的场景适用，借鉴与启发，原理都是如此。

  2019-01-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/1e/9a/09/af250bf8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  熊@熊

  HTB的子队列会打满么？如果满了，新流量是丢弃么？还是会调整窗口？

  2022-03-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/92/36/6f4d8528.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夜⊙▽⊙

  老师，文中提到的类别该如何理解划分？

  2021-12-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/f9/12/0e6620cd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  三景页

  从刘超老师的 刘超的通俗云计算博客 追到极客时间来了

  2021-04-24

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/24/da/b3/35859560.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ciao🌚

  老师，看网上有的资料说，实际上实现Qos都使用了IP头部TOS field的6个bit的DSCP的概念，而不是那4个bit，因为application都不支持。是这样吗？

  2021-01-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/48/33/4663928e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  W҈T҈H҈

  Qos流量判断可以做网络黑洞吗？来抵御流量攻击？

  2020-10-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/25/4b/4cbd001e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  佳俊

  ovs-ofctl add-flow br0 "in_port=6 nw_src=192.168.100.100 actions=enqueue:5:0" ovs-ofctl add-flow br0 "in_port=7 nw_src=192.168.100.101 actions=enqueue:5:1" ovs-ofctl add-flow br0 "in_port=8 nw_src=192.168.100.102 actions=enqueue:5:2" 1. 这几个命令里面的in_port=6,7,8怎么和上面的queue对应起来的？

  作者回复: 不对应，对应的是enqueue

  2020-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e4/a1/178387da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  25ma

  感觉非科班需要花更多时间来消化这些知识，喜欢超哥的课程

  2020-03-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c8/6b/0f3876ef.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iron_man

  所有的流量都通过openv

  2018-08-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/6f/6051e0f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Summer___J

  3:1:6的例子、假如一开始是2个节点疯狂发包占满带宽，假如是第一个和第二个一开始在发，带宽利用占比是3:1。一段时间后，第三个节点再开始疯狂发包。这种情况，当第三个上来以后，这三个节点在带宽占用上会动态地回到3:1:6吗？

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/eb/82/4b56fa5f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  rtoday

  ovs-vsctl set port first_br qos=@newqos  -- --id=@newqos create qos type=linux-htb other-config:max-rate=10,000,000 queues=0=@q0,1=@q1,2=@q2 -- --id=@q0 create queue other-config:min-rate=3,000,000 other-config:max-rate=10,000,000  -- --id=@q1 create queue other-config:min-rate=1,000,000 other-config:max-rate=10,000,000  -- --id=@q2 create queue other-config:min-rate=6,000,000 other-config:max-rate=10,000,000 这是倒数第二个指令 我刻意排版一下，并且把数字使用三位一个撇节 1. 语法问题 为何写成 -- --id=@newqos 我可以只写下面这样吗 --id=@newqos 双横线然后后面不加上option，请问有什么特殊用意吗？ 2.语意问题 是否是我吹毛求疵了 好象每个数字都少一个0 还是我的认知有误呢

  2018-07-19

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  网络已断开

  看完似懂非懂，心里痒痒的

  2018-07-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/eb/82/4b56fa5f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  rtoday

  第一题 这篇讲的是Client如何控制出口流量 那Client的入口流量 也按照一样的原理，只是由Server端，或是数据中心端的人，来控制他们的出口流量即可。这应该是有没有权限的问题。 第二题，应该是下期的主题，等下期出刊后，再来回顾本题，可能体会的比较完整。

  2018-07-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/87/b5/dd0353f4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  三水

  问题2: 可以使用Linux Network Namespace进行隔离，cgroup 就进行资源调配和统计，云计算多租户资源使用，如基于docker的云计算服务

  2018-07-18
