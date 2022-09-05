# 第28讲 \| 云中网络的隔离GRE、VXLAN：虽然住一个小区，也要保护隐私

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/02/3a/02311815d4d429a8a055e66737e3833a.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/05/c1/05d27405cee757dcf8a8f13bab4f68c1.mp3" type="audio/mpeg"></audio>

对于云平台中的隔离问题，前面咱们用的策略一直都是VLAN，但是我们也说过这种策略的问题，VLAN只有12位，共4096个。当时设计的时候，看起来是够了，但是现在绝对不够用，怎么办呢？

**一种方式是修改这个协议**。这种方法往往不可行，因为当这个协议形成一定标准后，千千万万设备上跑的程序都要按这个规则来。现在说改就放，谁去挨个儿告诉这些程序呢？很显然，这是一项不可能的工程。

**另一种方式就是扩展**，在原来包的格式的基础上扩展出一个头，里面包含足够用于区分租户的ID，外层的包的格式尽量和传统的一样，依然兼容原来的格式。一旦遇到需要区分用户的地方，我们就用这个特殊的程序，来处理这个特殊的包的格式。

这个概念很像咱们[第22讲](<https://time.geekbang.org/column/article/10386>)讲过的**隧道理论**，还记得自驾游通过摆渡轮到海南岛的那个故事吗？在那一节，我们说过，扩展的包头主要是用于加密的，而我们现在需要的包头是要能够区分用户的。

底层的物理网络设备组成的网络我们称为**Underlay网络**，而用于虚拟机和云中的这些技术组成的网络称为**Overlay网络**，**这是一种基于物理网络的虚拟化网络实现**。这一节我们重点讲两个Overlay的网络技术。

## GRE

第一个技术是**GRE**，全称Generic Routing Encapsulation，它是一种IP-over-IP的隧道技术。它将IP包封装在GRE包里，外面加上IP头，在隧道的一端封装数据包，并在通路上进行传输，到另外一端的时候解封装。你可以认为Tunnel是一个虚拟的、点对点的连接。

<!-- [[[read_end]]] -->

![](<https://static001.geekbang.org/resource/image/d4/7d/d45e189d89b2273690yyc8yy131ffe7d.jpg?wh=1309*1050>)

从这个图中可以看到，在GRE头中，前32位是一定会有的，后面的都是可选的。在前4位标识位里面，有标识后面到底有没有可选项？这里面有个很重要的key字段，是一个32位的字段，里面存放的往往就是用于区分用户的Tunnel ID。32位，够任何云平台喝一壶的了！

下面的格式类型专门用于网络虚拟化的GRE包头格式，称为**NVGRE**，也给网络ID号24位，也完全够用了。

除此之外，GRE还需要有一个地方来封装和解封装GRE的包，这个地方往往是路由器或者有路由功能的Linux机器。

使用GRE隧道，传输的过程就像下面这张图。这里面有两个网段、两个路由器，中间要通过GRE隧道进行通信。当隧道建立之后，会多出两个Tunnel端口，用于封包、解封包。

![](<https://static001.geekbang.org/resource/image/76/ce/76298ce51d349bc9805fbf317312e4ce.jpg?wh=2220*899>)

1. 主机A在左边的网络，IP地址为192.168.1.102，它想要访问主机B，主机B在右边的网络，IP地址为192.168.2.115。于是发送一个包，源地址为192.168.1.102，目标地址为192.168.2.115。因为要跨网段访问，于是根据默认的default路由表规则，要发给默认的网关192.168.1.1，也即左边的路由器。

2. 根据路由表，从左边的路由器，去192.168.2.0/24这个网段，应该走一条GRE的隧道，从隧道一端的网卡Tunnel0进入隧道。

3. 在Tunnel隧道的端点进行包的封装，在内部的IP头之外加上GRE头。对于NVGRE来讲，是在MAC头之外加上GRE头，然后加上外部的IP地址，也即路由器的外网IP地址。源IP地址为172.17.10.10，目标IP地址为172.16.11.10，然后从E1的物理网卡发送到公共网络里。

4. 在公共网络里面，沿着路由器一跳一跳地走，全部都按照外部的公网IP地址进行。

5. 当网络包到达对端路由器的时候，也要到达对端的Tunnel0，然后开始解封装，将外层的IP头取下来，然后根据里面的网络包，根据路由表，从E3口转发出去到达服务器B。


<!-- -->

从GRE的原理可以看出，GRE通过隧道的方式，很好地解决了VLAN ID不足的问题。但是，GRE技术本身还是存在一些不足之处。

首先是**Tunnel的数量问题**。GRE是一种点对点隧道，如果有三个网络，就需要在每两个网络之间建立一个隧道。如果网络数目增多，这样隧道的数目会呈指数性增长。

![](<https://static001.geekbang.org/resource/image/00/fe/006cc8a4bf7a13fea0f456905c263afe.jpg?wh=711*507>)

其次，**GRE不支持组播**，因此一个网络中的一个虚机发出一个广播帧后，GRE会将其广播到所有与该节点有隧道连接的节点。

另外一个问题是目前还是**有很多防火墙和三层网络设备无法解析GRE**，因此它们无法对GRE封装包做合适地过滤和负载均衡。

## VXLAN

第二种Overlay的技术称为VXLAN。和三层外面再套三层的GRE不同，VXLAN则是从二层外面就套了一个VXLAN的头，这里面包含的VXLAN ID为24位，也够用了。在VXLAN头外面还封装了UDP、IP，以及外层的MAC头。

![](<https://static001.geekbang.org/resource/image/87/9f/87b61f26a10972df1f56d461ef97009f.jpg?wh=1771*666>)

VXLAN作为扩展性协议，也需要一个地方对VXLAN的包进行封装和解封装，实现这个功能的点称为**VTEP**（VXLAN Tunnel Endpoint）。

VTEP相当于虚拟机网络的管家。每台物理机上都可以有一个VTEP。每个虚拟机启动的时候，都需要向这个VTEP管家注册，每个VTEP都知道自己上面注册了多少个虚拟机。当虚拟机要跨VTEP进行通信的时候，需要通过VTEP代理进行，由VTEP进行包的封装和解封装。

和GRE端到端的隧道不同，VXLAN不是点对点的，而是支持通过组播的来定位目标机器的，而非一定是这一端发出，另一端接收。

当一个VTEP启动的时候，它们都需要通过IGMP协议。加入一个组播组，就像加入一个邮件列表，或者加入一个微信群一样，所有发到这个邮件列表里面的邮件，或者发送到微信群里面的消息，大家都能收到。而当每个物理机上的虚拟机启动之后，VTEP就知道，有一个新的VM上线了，它归我管。

![](<https://static001.geekbang.org/resource/image/88/4f/88a0fa7f6a46e6a8980279c73c14604f.jpg?wh=1760*1718>)

如图，虚拟机1、2、3属于云中同一个用户的虚拟机，因而需要分配相同的VXLAN ID=101。在云的界面上，就可以知道它们的IP地址，于是可以在虚拟机1上ping虚拟机2。

虚拟机1发现，它不知道虚拟机2的MAC地址，因而包没办法发出去，于是要发送ARP广播。

![](<https://static001.geekbang.org/resource/image/8b/b7/8bea66c28395e3a9b77e845803bffbb7.jpg?wh=2590*1719>)

ARP请求到达VTEP1的时候，VTEP1知道，我这里有一台虚拟机，要访问一台不归我管的虚拟机，需要知道MAC地址，可是我不知道啊，这该咋办呢？

VTEP1想，我不是加入了一个微信群么？可以在里面@all 一下，问问虚拟机2归谁管。于是VTEP1将ARP请求封装在VXLAN里面，组播出去。

当然在群里面，VTEP2和VTEP3都收到了消息，因而都会解开VXLAN包看，里面是一个ARP。

VTEP3在本地广播了半天，没人回，都说虚拟机2不归自己管。

VTEP2在本地广播，虚拟机2回了，说虚拟机2归我管，MAC地址是这个。通过这次通信，VTEP2也学到了，虚拟机1归VTEP1管，以后要找虚拟机1，去找VTEP1就可以了。

![](<https://static001.geekbang.org/resource/image/f5/10/f5cb3442dd8705a7ac5e46c896b02210.jpg?wh=2776*1617>)

VTEP2将ARP的回复封装在VXLAN里面，这次不用组播了，直接发回给VTEP1。

VTEP1解开VXLAN的包，发现是ARP的回复，于是发给虚拟机1。通过这次通信，VTEP1也学到了，虚拟机2归VTEP2管，以后找虚拟机2，去找VTEP2就可以了。

虚拟机1的ARP得到了回复，知道了虚拟机2的MAC地址，于是就可以发送包了。

![](<https://static001.geekbang.org/resource/image/42/ed/42b4393b5c5772e1c35bcf1a95586eed.jpg?wh=3052*1723>)

虚拟机1发给虚拟机2的包到达VTEP1，它当然记得刚才学的东西，要找虚拟机2，就去VTEP2，于是将包封装在VXLAN里面，外层加上VTEP1和VTEP2的IP地址，发送出去。

网络包到达VTEP2之后，VTEP2解开VXLAN封装，将包转发给虚拟机2。

虚拟机2回复的包，到达VTEP2的时候，它当然也记得刚才学的东西，要找虚拟机1，就去VTEP1，于是将包封装在VXLAN里面，外层加上VTEP1和VTEP2的IP地址，也发送出去。

网络包到达VTEP1之后，VTEP1解开VXLAN封装，将包转发给虚拟机1。

![](<https://static001.geekbang.org/resource/image/8d/95/8d72af40bbc0833f4b66fb1f7c040895.jpg?wh=3012*1712>)

有了GRE和VXLAN技术，我们就可以解决云计算中VLAN的限制了。那如何将这个技术融入云平台呢？

还记得将你宿舍里面的情况，所有东西都搬到一台物理机上那个故事吗？

![](<https://static001.geekbang.org/resource/image/96/0e/96de854bf0480133ef6081a912f0c20e.jpg?wh=2299*1671>)

虚拟机是你的电脑，路由器和DHCP Server相当于家用路由器或者寝室长的电脑，外网网口访问互联网，所有的电脑都通过内网网口连接到一个交换机br0上，虚拟机要想访问互联网，需要通过br0连到路由器上，然后通过路由器将请求NAT后转发到公网。

接下来的事情就惨了，你们宿舍闹矛盾了，你们要分成三个宿舍住，对应上面的图，你们寝室长，也即路由器单独在一台物理机上，其他的室友也即VM分别在两台物理机上。这下把一个完整的br0一刀三断，每个宿舍都是单独的一段。

![](<https://static001.geekbang.org/resource/image/0a/a8/0a867f63aa4874b985beb292896308a8.jpg?wh=2619*2110>)

可是只有你的寝室长有公网口可以上网，于是你偷偷在三个宿舍中间打了一个隧道，用网线通过隧道将三个宿舍的两个br0连接起来，让其他室友的电脑和你寝室长的电脑，看起来还是连到同一个br0上，其实中间是通过你隧道中的网线做了转发。

为什么要多一个br1这个虚拟交换机呢？主要通过br1这一层将虚拟机之间的互联和物理机机之间的互联分成两层来设计，中间隧道可以有各种挖法，GRE、VXLAN都可以。

使用了OpenvSwitch之后，br0可以使用OpenvSwitch的Tunnel功能和Flow功能。

OpenvSwitch支持三类隧道：GRE、VXLAN、IPsec\_GRE。在使用OpenvSwitch的时候，虚拟交换机就相当于GRE和VXLAN封装的端点。

我们模拟创建一个如下的网络拓扑结构，来看隧道应该如何工作。

![](<https://static001.geekbang.org/resource/image/52/62/52770d81e0fcd97ec1ece04686c5c962.jpg?wh=3281*1566>)

三台物理机，每台上都有两台虚拟机，分别属于两个不同的用户，因而VLAN tag都得打地不一样，这样才不能相互通信。但是不同物理机上的相同用户，是可以通过隧道相互通信的，因而通过GRE隧道可以连接到一起。

接下来，所有的Flow Table规则都设置在br1上，每个br1都有三个网卡，其中网卡1是对内的，网卡2和3是对外的。

下面我们具体来看Flow Table的设计。

![](<https://static001.geekbang.org/resource/image/55/41/55f5213ef580e3d081e467eb2e61d341.jpg?wh=4060*2864>)

1\.Table 0是所有流量的入口，所有进入br1的流量，分为两种流量，一个是进入物理机的流量，一个是从物理机发出的流量。

从port 1进来的，都是发出去的流量，全部由Table 1处理。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 in_port=1 actions=resubmit(,1)"
```

从port 2、3进来的，都是进入物理机的流量，全部由Table 3处理。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 in_port=2 actions=resubmit(,3)"
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 in_port=3 actions=resubmit(,3)"
```

如果都没匹配上，就默认丢弃。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=0 actions=drop"
```

2\.Table 1用于处理所有出去的网络包，分为两种情况，一种是单播，一种是多播。

对于单播，由Table 20处理。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 table=1 dl_dst=00:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,20)"
```

对于多播，由Table 21处理。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 table=1 dl_dst=01:00:00:00:00:00/01:00:00:00:00:00 actions=resubmit(,21)"
```

3\.Table 2是紧接着Table1的，如果既不是单播，也不是多播，就默认丢弃。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=0 table=2 actions=drop"
```

4\.Table 3用于处理所有进来的网络包，需要将隧道Tunnel ID转换为VLAN ID。

如果匹配不上Tunnel ID，就默认丢弃。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=0 table=3 actions=drop"
```

如果匹配上了Tunnel ID，就转换为相应的VLAN ID，然后跳到Table 10。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 table=3 tun_id=0x1 actions=mod_vlan_vid:1,resubmit(,10)"
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 table=3 tun_id=0x2 actions=mod_vlan_vid:2,resubmit(,10)"
```

5\.对于进来的包，Table 10会进行MAC地址学习。这是一个二层交换机应该做的事情，学习完了之后，再从port 1发出去。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1 table=10  actions=learn(table=20,priority=1,hard_timeout=300,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1"
```

Table 10是用来学习MAC地址的，学习的结果放在Table 20里面。Table20被称为MAC learning table。

NXM\_OF\_VLAN\_TCI是VLAN tag。在MAC learning table中，每一个entry都仅仅是针对某一个VLAN来说的，不同VLAN的learning table是分开的。在学习结果的entry中，会标出这个entry是针对哪个VLAN的。

NXM\_OF\_ETH\_DST[]=NXM\_OF\_ETH\_SRC[]表示，当前包里面的MAC Source Address会被放在学习结果的entry里的dl\_dst里。这是因为每个交换机都是通过进入的网络包来学习的。某个MAC从某个port进来，交换机就应该记住，以后发往这个MAC的包都要从这个port出去，因而源MAC地址就被放在了目标MAC地址里面，因为这是为了发送才这么做的。

load:0->NXM\_OF\_VLAN\_TCI[]是说，在Table20中，将包从物理机发送出去的时候，VLAN tag设为0，所以学习完了之后，Table 20中会有actions=strip\_vlan。

load:NXM\_NX\_TUN\_ID[]->NXM\_NX\_TUN\_ID[]的意思是，在Table 20中，将包从物理机发出去的时候，设置Tunnel ID，进来的时候是多少，发送的时候就是多少，所以学习完了之后，Table 20中会有set\_tunnel。

output:NXM\_OF\_IN\_PORT[]是发送给哪个port。例如是从port 2进来的，那学习完了之后，Table 20中会有output:2。

![](<https://static001.geekbang.org/resource/image/4d/dd/4dc0fe34819ee02a53a97c89811747dd.jpg?wh=1014*334>)

所以如图所示，通过左边的MAC地址学习规则，学习到的结果就像右边的一样，这个结果会被放在Table 20里面。

6\.Table 20是MAC Address Learning Table。如果不为空，就按照规则处理；如果为空，就说明没有进行过MAC地址学习，只好进行广播了，因而要交给Table 21处理。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=0 table=20 actions=resubmit(,21)"
```

7\.Table 21用于处理多播的包。

如果匹配不上VLAN ID，就默认丢弃。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=0 table=21 actions=drop"
```

如果匹配上了VLAN ID，就将VLAN ID转换为Tunnel ID，从两个网卡port 2和port 3都发出去，进行多播。

```
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1table=21dl_vlan=1 actions=strip_vlan,set_tunnel:0x1,output:2,output:3"
ovs-ofctl add-flow br1 "hard_timeout=0 idle_timeout=0 priority=1table=21dl_vlan=2 actions=strip_vlan,set_tunnel:0x2,output:2,output:3"
```

## 小结

好了，这一节就到这里了，我们来总结一下。

- 要对不同用户的网络进行隔离，解决VLAN数目有限的问题，需要通过Overlay的方式，常用的有GRE和VXLAN。

- GRE是一种点对点的隧道模式，VXLAN支持组播的隧道模式，它们都要在某个Tunnel Endpoint进行封装和解封装，来实现跨物理机的互通。

- OpenvSwitch可以作为Tunnel Endpoint，通过设置流表的规则，将虚拟机网络和物理机网络进行隔离、转换。


<!-- -->

最后，给你留两个思考题。

1. 虽然VXLAN可以支持组播，但是如果虚拟机数目比较多，在Overlay网络里面，广播风暴问题依然会很严重，你能想到什么办法解决这个问题吗？

2. 基于虚拟机的云比较复杂，而且虚拟机里面的网卡，到物理网络转换层次比较多，有一种比虚拟机更加轻量级的云的模式，你知道是什么吗？


<!-- -->

我们的专栏更新到第28讲，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送<span class="orange">学习奖励礼券</span>

和我整理的<span class="orange">独家网络协议知识图谱</span>

。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(47)

- ![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,m_fill,h_34,w_34.jpeg)

  Jason

  一定是我基础差，最近几篇感觉有点难，不能系统的理解，准备复读！

  2018-07-20

  **

  **56

- ![img](https://static001.geekbang.org/account/avatar/00/11/da/dd/a8b93a4a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晨晨ada

  ARP 抑制是一种减缓ARP广播风暴的方案。

  2018-07-21

  **

  **13

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f6f02b

  是不是大的逻辑来说还是传统的7层网络模型，但是有了 OpenvSwitch 之后，就可以对数据包做任意的更改，所以在 IP 层跟 TCP 直间加一个 VXLAN 层数据包头，这里面有足够多的VXLAN ID (24位)，让 OpenvSwitch 去解决 VLAN ID 不足问题，知道这个 Tunnel ID 跟 VXLAN ID 有对应关系，但是不知道有 VXLAN ID 为什么还要 Tunnel ID。不知道理解的对不对

  作者回复: tunnel id是ovs的概念，vxlan id是vxlan协议的要求

  2019-04-25

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/11/68/c3/8e1a8dbf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Plantegg

  flow table图箭头缺了是、否。另外这里面vlan ID也不是一定需要的吧，感觉图有点问题

  2018-07-23

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8f/8b/9080f1fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  猫头鹰波波

  建议老师多讲点WHY，再讲WHAT，这样比较好理解一些，一上来就讲细节有点难理解

  2020-02-03

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/10/5e/2b/df3983e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  朱显杰

  问题1:可以采用underlay网络，比如calico。问题二：容器云

  2018-07-20

  **

  **6

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIvMlvSXsYgJibIyDO78gPacZR1qukEOJrpfHAJmyGVtWPO3XMqVA9dImHhGJm2icp6lDuBw1GrNDbA/132)

  赤脚小子

  问题1 广播风暴的主要原因就是环路了，传统网络采用stp，针对overlay ，通过路由协议ecmp实现

  2018-07-20

  **

  **4

- ![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,m_fill,h_34,w_34-1662308137189-9626.jpeg)

  勤劳的小胖子-libo

  第一个跟vpc虚拟专用网络有关吗？ 第二个是容器云吧，会讲k8s吗

  2018-07-22

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/77/cb933f6c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  hill

  overlay一般对广播报文进行抑制和代理，来将广播限制在接入设备下面

  2018-07-21

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/1b/e4/d5e021f9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  crysSuenny

  自打老师直播提到、录音时间长会嘴瓢，我就老赶脚在录音中听到嘴瓢....不然累的时候就别录了？～好心疼噢..～

  2018-07-21

  **

  **3

- ![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,m_fill,h_34,w_34-1662308137190-9629.jpeg)![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,w_14.png)

  千岁寒

  问题一:  广播抑制，典型的方案是ARP代理和ARP代答。 问题二：容器云，Docker和k8s。

  2020-02-01

  **

  **2

- ![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,m_fill,h_34,w_34-1662308137190-9630.jpeg)

  偏偏喜欢你

  虽然干货满满，基础太差，得多读几遍了

  2018-10-01

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/46/2e/1017900c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  牛逼中…

  “OpenvSwitch 支持三类隧道：GRE、VXLAN、IPsec_GRE。”，所以文章中的例子是使用GRE隧道了？那没明白为什么有vlan id和tunnel id的转换？或者说这里为什么出现vlan id？

  2020-03-21

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  最后这个 Flow Table  太绕了，这篇粗糙点过了先。

  2019-12-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/11/b2/dd0606b2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  水先生

  老师，请问GRE的隧道两端的端口，是怎么实现封包和解包的呢？

  2019-09-26

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/80/ac/37afe559.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小毅(Eric)

  太难了，太难了，要多用多读啊

  作者回复: 到这里难一些了，加油

  2019-08-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fc/8e/ff99e2b2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  于琳琳

  有点听不懂了(￣∀￣)

  2018-07-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/16/4d1e5cc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mgxian

  思考题2   容器云

  2018-07-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/47/00/3202bdf0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,w_14.png)

  piboye

  gre和vxlan各用于什么场景？

  2022-08-23

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJcwXucibksEYRSYg6icjibzGa7efcMrCsGec2UwibjTd57icqDz0zzkEEOM2pXVju60dibzcnQKPfRkN9g/132)![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,w_14.png)

  Geek_93970d

  "网络包到达 VTEP1 之后，VTEP1 解开 VXLAN 封装，将包转发给虚拟机 1。" 图里面为啥是 ICMP ？

  2022-08-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/23/e8/9f445339.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  章潘

  Neutron的架构精髓就这样通俗的讲清楚了，拜膜大佬

  2022-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/24/3f9f7c70.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zixuan

  可读性有点差

  2022-03-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/67/0f/3cb10900.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜鸟

  使用GRE通道通信时、两端私网IP如何知道对端的公网IP和私网IP？

  2022-01-18

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  牛学真

  容器网络重头戏来了

  2021-07-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/6d/910b2445.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fastkdm

  从openvswitch开始就读不懂了

  2021-02-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/37/2b/b32f1d66.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ball

  这几篇有点难，要找机会实践一下

  2021-02-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e2/5a/9b058e14.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Wonder

  有个问题，没搞明白， OVS自身应该会根据进来的报文学习mac table吧？为什么还要加一个table 专门学习mac address呢？

  2021-02-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/15/86/cd97bf7e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  戴宇

  GRE看起来像是工作在三层协议，是不是要有和前面的VLAN已经不同，VLAN是二层网络协议，而且GRE是不是要特殊的路由器能认识这个头来能实现。如果返回去看数据中心那一节。GRE Endpoint应该在什么位置，数据中心里面说核心交换机在二层，是通过一个可以通过MAC地址进行路由的协议交互。那么 GRE Endpoint 是不是还在放在核心交换机上面？

  2020-10-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_6d1d93

  我是刚接触到这个部分，越听越难，有讲议吗？

  2020-09-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/5d/ff/b58dd422.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  星星⭐

  overly网络和openvswith有关系吗？

  2020-09-18

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTI4akcIyIOXB2OqibTe7FF90hwsBicxkjdicUNTMorGeIictdr3OoMxhc20yznmZWwAvQVThKPFWgOyMw/132)

  Chuan

  哇。。真的是膜拜老师了。。三年前大学搞SDN的时候，好多时候都没太理解VXLAN这些东西怎么搞、为什么这么搞，老师这个云计算网络章节，真的是完全解惑了，感谢感谢，再下去多消化下：）

  2020-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/26/fa3bb8e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ⊙▽⊙

  老师请教一个问题，openstack通过vxlan实现了网络隔离，那如果租户间有通信的需求，有什么办法可以让租户直接通信吗？而不是通过浮动ip

  2020-05-22

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eqy7GudyFZicjyYw9LuPAK3IUH6zzzUpJTpAjVkkbpmNcZ5GkvW8ibPsqVsgpP8iajXtxUvVTIjxibkAQ/132)

  xpxdx

  老师，工作中遇到问题了。 请教一下，云中隔离时，VPC、微服务，能否讲解一下，不是很懂。尤其横向隔离，当前和未来解决方案如何？谢谢老师！

  2020-05-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/de/3c/b1fe1f52.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,w_14.png)

  wsjx

  vlan知道，vxlan以前没听说过

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/36/0e/a41a4cdb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一个假的程序员

  刘超老师，你好，请问一下，GRE的隧道的数目多了会有什么影响呢

  2020-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c9/ce/cc85c6c3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,w_14.png)

  阿恒

  听不懂了，需要多听几遍了

  2020-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e4/a1/178387da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  25ma

  现在往后的每节基本至少听两遍，而且还是对着文稿一起，总算有些理解，还是得加强

  2020-03-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  这一节讲的overlay网络：GRE和VXLAN。GRE是建立端到端的隧道，缺点是随着用户网络的增多，相应隧道的数量也是指数级增长，同时不能传递组播，在运行的环境也有可能某一些设备不支持GRE；VXLAN是在乘客协议的外面加上UDP和承载协议，扩展了数据包，利于在数据中心运行。

  2020-03-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/db/c2/5c493433.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  William孟祥龙

  这一讲是从事云网络工作强相关并且重要的章节。

  2020-02-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/13/66/bfa42cb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Hello World 工程师

  你们有没有感觉，网络要学要记的东西太多了，不像编程，重的是逻辑。

  2019-09-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  zKerry

  对ip包的各种魔改，服了

  2019-09-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e5/69/80441a0a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  老蒙

  思考题：1是基于evpn的vxlan，2是docker

  2019-05-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/94/c9/374a69f2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张爽

  请问老师，有没有什么软件可以将专栏的这些知识实现一遍，记得Cisco公司有开发过一种做网络拓扑的工具，平时开发过程时很少接触到一些协议，光看理论没法更好的理解，看得云里雾里

  作者回复: 实现一遍，太难了

  2019-04-10

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/21/20/1299e137.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC28%E8%AE%B2%20_%20%E4%BA%91%E4%B8%AD%E7%BD%91%E7%BB%9C%E7%9A%84%E9%9A%94%E7%A6%BBGRE%E3%80%81VXLAN%EF%BC%9A%E8%99%BD%E7%84%B6%E4%BD%8F%E4%B8%80%E4%B8%AA%E5%B0%8F%E5%8C%BA%EF%BC%8C%E4%B9%9F%E8%A6%81%E4%BF%9D%E6%8A%A4%E9%9A%90%E7%A7%81.resource/resize,w_14.png)

  秋天

  已经不能联系起来啦，请作者指点从哪普及一下基础

  作者回复: 其实是有联系的，多读几遍前面的

  2018-09-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/f7/c4/bd7dd30a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小文

  我已蒙圈了，云中网网络之后都不懂了，是我基础太差吗

  2018-08-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/6f/6051e0f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Summer___J

  听了两遍还是有点懵懵懂懂……

  2018-08-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/28/1e307312.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鲍勃

  太难了，太难了，要多用多读啊
