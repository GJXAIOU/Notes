# 第3讲 \| ifconfig：最熟悉又陌生的命令行

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/0e/a8/0ec9478a039081489d07cd19a00cd2a8.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/2b/da/2be5885efb35636c8351bf2058aba2da.mp3" type="audio/mpeg"></audio>

上一节结尾给你留的一个思考题是，你知道怎么查看IP地址吗？

当面试听到这个问题的时候，面试者常常会觉得走错了房间。我面试的是技术岗位啊，怎么问这么简单的问题？

的确，即便没有专业学过计算机的人，只要倒腾过电脑，重装过系统，大多也会知道这个问题的答案：在Windows上是ipconfig，在Linux上是ifconfig。

那你知道在Linux上还有什么其他命令可以查看IP地址吗？答案是ip addr。如果回答不上来这个问题，那你可能没怎么用过Linux。

那你知道ifconfig和ip addr的区别吗？这是一个有关net-tools和iproute2的“历史”故事，你刚来到第三节，暂时不用了解这么细，但这也是一个常考的知识点。

想象一下，你登录进入一个被裁剪过的非常小的Linux系统中，发现既没有ifconfig命令，也没有ip addr命令，你是不是感觉这个系统压根儿没法用？这个时候，你可以自行安装net-tools和iproute2这两个工具。当然，大多数时候这两个命令是系统自带的。

安装好后，我们来运行一下ip addr。不出意外，应该会输出下面的内容。

```
root@test:~# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff
    inet 10.100.122.2/24 brd 10.100.122.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fec7:7975/64 scope link 
       valid_lft forever preferred_lft forever
```

这个命令显示了这台机器上所有的网卡。大部分的网卡都会有一个IP地址，当然，这不是必须的。在后面的分享中，我们会遇到没有IP地址的情况。

<!-- [[[read_end]]] -->

<span class="orange">IP地址是一个网卡在网络世界的通讯地址，相当于我们现实世界的门牌号码。</span>

既然是门牌号码，不能大家都一样，不然就会起冲突。比方说，假如大家都叫六单元1001号，那快递就找不到地方了。所以，有时候咱们的电脑弹出网络地址冲突，出现上不去网的情况，多半是IP地址冲突了。

如上输出的结果，10.100.122.2就是一个IP地址。这个地址被点分隔为四个部分，每个部分8个bit，所以IP地址总共是32位。这样产生的IP地址的数量很快就不够用了。因为当时设计IP地址的时候，哪知道今天会有这么多的计算机啊！因为不够用，于是就有了IPv6，也就是上面输出结果里面inet6 fe80::f816:3eff:fec7:7975/64。这个有128位，现在看来是够了，但是未来的事情谁知道呢？

本来32位的IP地址就不够，还被分成了5类。现在想想，当时分配地址的时候，真是太奢侈了。

![](<https://static001.geekbang.org/resource/image/fa/5e/fa9a00346b8a83a84f4c00de948a5b5e.jpg?wh=2323*1333>)

在网络地址中，至少在当时设计的时候，对于A、B、 C类主要分两部分，前面一部分是网络号，后面一部分是主机号。这很好理解，大家都是六单元1001号，我是小区A的六单元1001号，而你是小区B的六单元1001号。

下面这个表格，详细地展示了A、B、C三类地址所能包含的主机的数量。在后文中，我也会多次借助这个表格来讲解。

![](<https://static001.geekbang.org/resource/image/6d/4c/6de9d39a68520c8e3b75aa919fadb24c.jpg?wh=2323*673>)

这里面有个尴尬的事情，就是C类地址能包含的最大主机数量实在太少了，只有254个。当时设计的时候恐怕没想到，现在估计一个网吧都不够用吧。而B类地址能包含的最大主机数量又太多了。6万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。

## 无类型域间选路（CIDR）

于是有了一个折中的方式叫作**无类型域间选路**，简称**CIDR**。这种方式打破了原来设计的几类地址的做法，将32位的IP地址一分为二，前面是**网络号**，后面是**主机号**。从哪里分呢？你如果注意观察的话可以看到，10.100.122.2/24，这个IP地址中有一个斜杠，斜杠后面有个数字24。这种地址表示形式，就是CIDR。后面24的意思是，32位中，前24位是网络号，后8位是主机号。

伴随着CIDR存在的，一个是**广播地址**，10.100.122.255。如果发送这个地址，所有10.100.122网络里面的机器都可以收到。另一个是**子网掩码**，255.255.255.0。

将子网掩码和IP地址进行AND计算。前面三个255，转成二进制都是1。1和任何数值取AND，都是原来数值，因而前三个数不变，为10.100.122。后面一个0，转换成二进制是0，0和任何数值取AND，都是0，因而最后一个数变为0，合起来就是10.100.122.0。这就是**网络号**。**将子网掩码和IP地址按位计算AND，就可得到网络号。**

## 公有IP地址和私有IP地址

在日常的工作中，几乎不用划分A类、B类或者C类，所以时间长了，很多人就忘记了这个分类，而只记得CIDR。但是有一点还是要注意的，就是公有IP地址和私有IP地址。

![](<https://static001.geekbang.org/resource/image/df/a9/df90239efec6e35880b9abe55089ffa9.jpg?wh=2359*736>)

我们继续看上面的表格。表格最右列是私有IP地址段。平时我们看到的数据中心里，办公室、家里或学校的IP地址，一般都是私有IP地址段。因为这些地址允许组织内部的IT人员自己管理、自己分配，而且可以重复。因此，你学校的某个私有IP地址段和我学校的可以是一样的。

这就像每个小区有自己的楼编号和门牌号，你们小区可以叫6栋，我们小区也叫6栋，没有任何问题。但是一旦出了小区，就需要使用公有IP地址。就像人民路888号，是国家统一分配的，不能两个小区都叫人民路888号。

公有IP地址有个组织统一分配，你需要去买。如果你搭建一个网站，给你学校的人使用，让你们学校的IT人员给你一个IP地址就行。但是假如你要做一个类似网易163这样的网站，就需要有公有IP地址，这样全世界的人才能访问。

表格中的192.168.0.x是最常用的私有IP地址。你家里有Wi-Fi，对应就会有一个IP地址。一般你家里地上网设备不会超过256个，所以/24基本就够了。有时候我们也能见到/16的CIDR，这两种是最常见的，也是最容易理解的。

不需要将十进制转换为二进制32位，就能明显看出192.168.0是网络号，后面是主机号。而整个网络里面的第一个地址192.168.0.1，往往就是你这个私有网络的出口地址。例如，你家里的电脑连接Wi-Fi，Wi-Fi路由器的地址就是192.168.0.1，而192.168.0.255就是广播地址。一旦发送这个地址，整个192.168.0网络里面的所有机器都能收到。

但是也不总都是这样的情况。因此，其他情况往往就会很难理解，还容易出错。

## 举例：一个容易“犯错”的CIDR

我们来看16.158.165.91/22这个CIDR。求一下这个网络的第一个地址、子网掩码和广播地址。

你要是上来就写16.158.165.1，那就大错特错了。

/22不是8的整数倍，不好办，只能先变成二进制来看。16.158的部分不会动，它占了前16位。中间的165，变为二进制为‭10100101‬。除了前面的16位，还剩6位。所以，这8位中前6位是网络号，16.158.<101001>，而<01>.91是机器号。

第一个地址是16.158.<101001><00>.1，即16.158.164.1。子网掩码是255.255.<111111><00>.0，即255.255.252.0。广播地址为16.158.<101001><11>.255，即16.158.167.255。

这五类地址中，还有一类D类是**组播地址**。使用这一类地址，属于某个组的机器都能收到。这有点类似在公司里面大家都加入了一个邮件组。发送邮件，加入这个组的都能收到。组播地址在后面讲述VXLAN协议的时候会提到。

讲了这么多，才讲了上面的输出结果中很小的一部分，是不是觉得原来并没有真的理解ip addr呢？我们接着来分析。

在IP地址的后面有个scope，对于eth0这张网卡来讲，是global，说明这张网卡是可以对外的，可以接收来自各个地方的包。对于lo来讲，是host，说明这张网卡仅仅可以供本机相互通信。

lo全称是**loopback**，又称**环回接口**，往往会被分配到127.0.0.1这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现。

## MAC地址

在IP地址的上一行是link/ether fa:16:3e:c7:79:75 brd ff:ff:ff:ff:ff:ff，这个被称为**MAC地址**，是一个网卡的物理地址，用十六进制，6个byte表示。

MAC地址是一个很容易让人“误解”的地址。因为MAC地址号称全局唯一，不会有两个网卡有相同的MAC地址，而且网卡自生产出来，就带着这个地址。很多人看到这里就会想，既然这样，整个互联网的通信，全部用MAC地址好了，只要知道了对方的MAC地址，就可以把信息传过去。

这样当然是不行的。 **一个网络包要从一个地方传到另一个地方，除了要有确定的地址，还需要有定位功能。** 而有门牌号码属性的IP地址，才是有远程定位功能的。

例如，你去杭州市网商路599号B楼6层找刘超，你在路上问路，可能被问的人不知道B楼是哪个，但是可以给你指网商路怎么去。但是如果你问一个人，你知道这个身份证号的人在哪里吗？可想而知，没有人知道。

<span class="orange">MAC地址更像是身份证，是一个唯一的标识。</span>

它的唯一性设计是为了组网的时候，不同的网卡放在一个网络里面的时候，可以不用担心冲突。从硬件角度，保证不同的网卡有不同的标识。

MAC地址是有一定定位功能的，只不过范围非常有限。你可以根据IP地址，找到杭州市网商路599号B楼6层，但是依然找不到我，你就可以靠吼了，大声喊身份证XXXX的是哪位？我听到了，我就会站起来说，是我啊。但是如果你在上海，到处喊身份证XXXX的是哪位，我不在现场，当然不会回答，因为我在杭州不在上海。

所以，MAC地址的通信范围比较小，局限在一个子网里面。例如，从192.168.0.2/24访问192.168.0.3/24是可以用MAC地址的。一旦跨子网，即从192.168.0.2/24到192.168.1.2/24，MAC地址就不行了，需要IP地址起作用了。

## 网络设备的状态标识

解析完了MAC地址，我们再来看 <BROADCAST,MULTICAST,UP,LOWER\_UP>是干什么的？这个叫做**net\_device flags**，**网络设备的状态标识**。

UP表示网卡处于启动的状态；BROADCAST表示这个网卡有广播地址，可以发送广播包；MULTICAST表示网卡可以发送多播包；LOWER\_UP表示L1是启动的，也即网线插着呢。MTU1500是指什么意思呢？是哪一层的概念呢？最大传输单元MTU为1500，这是以太网的默认值。

上一节，我们讲过网络包是层层封装的。MTU是二层MAC层的概念。MAC层有MAC的头，以太网规定正文部分不允许超过1500个字节。正文里面有IP的头、TCP的头、HTTP的头。如果放不下，就需要分片来传输。

qdisc pfifo\_fast是什么意思呢？qdisc全称是**queueing discipline**，中文叫**排队规则**。内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的qdisc（排队规则）把数据包加入队列。

最简单的qdisc是pfifo，它不对进入的数据包做任何的处理，数据包采用先入先出的方式通过队列。pfifo\_fast稍微复杂一些，它的队列包括三个波段（band）。在每个波段里面，使用先进先出规则。

三个波段（band）的优先级也不相同。band 0的优先级最高，band 2的最低。如果band 0里面有数据包，系统就不会处理band 1里面的数据包，band 1和band 2之间也是一样。

数据包是按照服务类型（**Type of Service，TOS**）被分配到三个波段（band）里面的。TOS是IP头里面的一个字段，代表了当前的包是高优先级的，还是低优先级的。

队列是个好东西，后面我们讲云计算中的网络的时候，会有很多用户共享一个网络出口的情况，这个时候如何排队，每个队列有多粗，队列处理速度应该怎么提升，我都会详细为你讲解。

## 小结

怎么样，看起来很简单的一个命令，里面学问很大吧？通过这一节，希望你能记住以下的知识点，后面都能用得上：

- IP是地址，有定位功能；MAC是身份证，无定位功能；

- CIDR可以用来判断是不是本地人；

- IP分公有的IP和私有的IP。后面的章节中我会谈到“出国门”，就与这个有关。


<!-- -->

最后，给你留两个思考题。

1. 你知道net-tools和iproute2的“历史”故事吗？
2. 这一节讲的是如何查看IP地址，那你知道IP地址是怎么来的吗？

<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(310)

- ![img](https://static001.geekbang.org/account/avatar/00/10/c5/4d/ac0f0371.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  猿来是你

  置顶

  能讲的详细些吗？非网络科班出身，理解不透彻！不要一带而过！

  作者回复: 第三讲主要通过ip addr命令对于网络相关概念有一个总体的介绍，深入了其中一部分，如果您觉得其他部门讲的粗略还不能理解透彻，到这一节可以先忽略，应该不影响。当时设计讲网络的时候，其实就有个难点，相互关联性太强，二层会依赖四层，四层也会依赖二层，如果每一点都深挖的另一个问题就是一下子深入进去，让初学者晕了。所以我想用的方式是从平时接触到的东西开始逐层深入，如果文中说这里不详述的部分，其实是对当前知识点的理解尚不构成阻碍，等构成阻碍了，就会讲清楚。

  2018-05-27

  **19

  **230

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  船新版本

  置顶

  cidr那块将IP和子网掩码都转成二进制列出来对比的话会比较直观很多，第一遍看到这块的时候有点懵

  作者回复: 这段纠结了好久，完全二进制的话，音频就没法读了。现在这样😊好像读起来也有点别扭。是要照顾上班路上只听音频的朋友

  2018-05-26

  **10

  **73

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/c6/4a7b2517.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Will王志翔(大象)

  第三讲笔记  # 面试考点：  1.  ip addr → 不知道基本没有用Linux 2.  ifconfig 和 ip addr 的区别吗？ 3.  CIDR 4.  共有IP和私有IP 5.  MAC地址 6.  网络设备的状态标识 # 知识点：  ## 核心：  1. IP设计时犯的错误？  低估了未来网络的发展，32位地址不够用。于是有了现在IPv6（128位） 分类错误。分成了5类。C类太少，B类太多。C类254个，网络都不够；D类6万多，给企业都太多。  2. 那后来者如何弥补IP设计者犯的错误呢？  CIDR，无类型域间选路。 打破原来几类地址设计的做法，将32位IP地址一分二，前者网络号，后者主机号。 如何分呢？ 栗子：10.100.122.2/24 24 = 前24位是网络号，那么后8位就是主机号。 那如何用？ 如发送行信息给 10.100.122.255 所有以 10.100.122... 开头的机器都能收到。 于是有了两个概念： 广播地址：10.100.122.255 子网掩码：255.255.255.0 -> AND 得到网络号。  3. 每一个城市都有人民广场，IP设计是如何解决的？  公有IP地址和私有IP地址。 搭建世界人民都可以访问的网站，需要共有IP地址 搭建只有学校同学使用饿的网站，只要私有IP地址 例子1: Wi-Fi 192.168.0.x 是最常用的私有 IP 地址 192.168.0 是网络号 192.168.0.1，往往就是你这个私有网络的出口地址 192.168.0.255 就是广播地址。一旦发送这个地址，整个 192.168.0 网络里面的所有机器都能收到。 例子2: 16.158.165.91/22 4. 如何理解MAC地址？  如果说IP是地址，有定位功能。那Mac就是身份证，唯一识别。  ## 琐碎：  5. 讲了ABC，那是D类是什么？  D 类是组播地址。使用这一类地址，属于某个组的机器都能收到。这有点类似在公司里面大家都加入了一个邮件组。发送邮件，加入这个组的都能收到。组播地址在后面讲述 VXLAN 协议的时候会提到。  6. IP地址scope是什么意思？  对于 eth0 这张网卡来讲，是 global，说明这张网卡是可以对外的，可以接收来自各个地方的包。对于 lo 来讲，是 host，说明这张网卡仅仅可以供本机相互通信。  7. 那lo是什么意思？  lo 全称是loopback，又称环回接口，往往会被分配到 127.0.0.1 这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现。  8. < BROADCAST,MULTICAST,UP,LOWER_UP > 是干什么的？  net_device flags，网络设备的状态标识。 UP 表示网卡处于启动的状态； BROADCAST 表示这个网卡有广播地址，可以发送广播包； MULTICAST 表示网卡可以发送多播包； LOWER_UP 表示 L1 是启动的，也即网线插着呢。  9. MTU1500 是指什么意思呢？是哪一层的概念？  最大传输单元 MTU 为 1500，这是以太网的默认值。 MTU 是二层 MAC 层的概念。MAC 层有 MAC 的头，以太网规定连 MAC 头带正文合起来，不允许超过 1500 个字节。  10. qdisc pfifo_fast 是什么意思呢？  排队规则。规定数据包如何进出的。有pfifo, pfifo_fast.  

  2018-07-07

  **9

  **503

- ![img](https://static001.geekbang.org/account/avatar/00/10/d6/42/bafec9b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  盖

  net-tools起源于BSD，自2001年起，Linux社区已经对其停止维护，而iproute2旨在取代net-tools，并提供了一些新功能。一些Linux发行版已经停止支持net-tools，只支持iproute2。 net-tools通过procfs(/proc)和ioctl系统调用去访问和改变内核网络配置，而iproute2则通过netlink套接字接口与内核通讯。 net-tools中工具的名字比较杂乱，而iproute2则相对整齐和直观，基本是ip命令加后面的子命令。 虽然取代意图很明显，但是这么多年过去了，net-tool依然还在被广泛使用，最好还是两套命令都掌握吧。

  作者回复: 太赞了

  2018-05-23

  **5

  **468

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a6/6b/12d87713.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jealone

  MTU 大小是不包含二层头部和尾部的，MTU 1500表示二层MAC帧大小不超过1518. MAC 头14 字节，尾4字节。可以抓包验证

  作者回复: 赞

  2018-05-29

  **7

  **259

- ![img](https://static001.geekbang.org/account/avatar/00/10/4e/38/3faa8377.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  登高

  mac是身份证，ip是地址 透彻

  2018-05-23

  **

  **92

- ![img](https://static001.geekbang.org/account/avatar/00/10/e9/e8/31df61df.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  军秋

  现在很多工具都可以更改本机的MAC地址，也就是网络上存在很多MAC地址被更改成一样的，然而并没有出现通讯异常或者混乱这是为什么？ 这是一个别人的留言，老师回答了会出问题，但没回答为什么？ MAC在一个局域网内冲突才会影响网络通讯，局域网外是通过IP定位，所以不同局域网的网络设备MAC一样是不会有通讯问题的。

  2018-05-24

  **2

  **75

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  来生树

  看了3篇，精彩阿，这个课程定价，严重定低了。应该299起嘛

  作者回复: 哈哈，谢谢

  2018-05-23

  **3

  **55

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eqoz1fVUGgXYRheScrGHctMxicaxIYnjeoZpI7Ga4rxvia9ykkQ4berKb5oeZjfyo3iaVCxicEejmtibxQ/132)

  Bill

  补充内核恐慌的老梗： 不知道有没有内核恐慌的水友(̿▀̿̿Ĺ̯̿̿▀̿ ̿)̄ 1、1.1.1.1 不是测试用的，原来一直没分配，现在被用来做一个DNS了，宣传是比谷歌等公司的dns服务 更保护用户隐私。 2、IP地址255.255.255.255，代表有限广播，它的目标是网络中的所有主机。 3、IP地址0.0.0.0，通常代表未知的源主机。当主机采用DHCP动态获取IP地址而无法获得合法IP地址时，会用IP地址0.0.0.0来表示源主机IP地址未知。 4、NID不能以数字127开头。NID 127被保留给内部回送函数,作为本机循环测试使用。 例如，使用命令ping 127.0.0.1测试TCP/IP协议栈是否正确安装。在路由器中，同样支持循环测试地址的使用。

  2018-07-08

  **

  **53

- ![img](https://static001.geekbang.org/account/avatar/00/11/4e/f7/618a8e52.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  周磊

  大学学的计算机网络课程关于ip地址要比这详细多，但刘老师讲的更为生动，联系生活中的场景做比喻，读后印象深刻。 建议读起来困难的同学先了解下二进制以及与十进制转换，再就是找相关的资料补充一下。 很多东西第一遍读不懂没关系，无论你不理解或忘记多少，当你在另一个地方再次看到这些东西时，你便会有种亲切感，以前模糊的地方会在这次变得清晰一些。经过多次的接触同一个知识点，你会越来越清楚直到透彻。

  作者回复: 赞，毕竟大学的课时比较多

  2018-05-31

  **2

  **46

- ![img](https://static001.geekbang.org/account/avatar/00/12/7a/0a/0ce5c232.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吕

  请教一下：我阿里云的多台机器，有172.16.2.145 和172.16.3.28这两个是怎么联通的？

  作者回复: 如果/16，就是一个网段的。如果是/24，则中间会有路由器

  2019-05-16

  **3

  **35

- ![img](https://static001.geekbang.org/account/avatar/00/0f/87/dd/4f53f95d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  进阶的码农

  A B C 类别表里A类数据有问题 应该是1:0:0:1-126:255:255:254 建议检查以下B 和C类

  作者回复: (⊙o⊙)哇，好严谨。A类IP的地址第一个字段范围是0~127，但是由于全0和全1的地址用作特殊用途，实际可指派的第一个字段范围是1~126。所以仔细搜了一下，如果较真的考试题的说法是，A类地址范围和A类有效地址范围。

  2018-06-01

  **

  **31

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4b/0f/d747ed96.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Steve

  推荐两本书籍：《图解 TCP/IP》、《wireshark 数据包分析实战》

  作者回复: 赞

  2019-03-30

  **

  **27

- ![img](https://static001.geekbang.org/account/avatar/00/11/88/9c/cbc463e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  仰望星空

  解题思路 1、首先求子网掩码，子网掩码掩盖网络号，暴露主机号，用1表示掩盖，用0表示暴露。22表示网络号有22位，那么主机号有10位，因此子网掩码的二进制的值为： 11111111.11111111.11111100.00000000，转换为十进制：255.255.252.0 2、再求网络号，将子网掩码和 IP 地址按位计算 AND，就可得到网络号， 16.158.165.91的二进制： ‭00010000.‬10011110.‭10100101‬.‭01011011‬ AND 11111111.11111111.11111100.00000000 = 00010000.10011110.10100100.00000000 = 16.158.164.0， 那么第一个网络号（主机号的最后一位是1）为： 00010000.10011110.10100100.00000001=16.158.164.1， 最后一个网路号也即广播地址（主机号的所有位是1）为： 00010000.10011110.10100111.11111111=16.158.167.255

  2019-12-18

  **6

  **22

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/1a/11b56711.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  秋生

  刘老师，您好，您举例说的那个容易犯错的CIDR问题里，为什么第一个地址和子网掩码都是补上00，而广播地址是补上11；本人是个小白，希望能得到您的解答，谢谢

  2018-05-23

  **5

  **22

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/c6/4a7b2517.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Will王志翔(大象)

  第三讲笔记  # 面试考点：  1.  ip addr → 不知道基本没有用Linux 2.  ifconfig 和 ip addr 的区别吗？ 3.  CIDR 4.  共有IP和私有IP 5.  MAC地址 6.  网络设备的状态标识 # 知识点：  ## 核心：  1. IP设计时犯的错误？  低估了未来网络的发展，32位地址不够用。于是有了现在IPv6（128位） 分类错误。分成了5类。C类太少，B类太多。C类254个，网络都不够；D类6万多，给企业都太多。  2. 那后来者如何弥补IP设计者犯的错误呢？  CIDR，无类型域间选路。 打破原来几类地址设计的做法，将32位IP地址一分二，前者网络号，后者主机号。 如何分呢？ 栗子：10.100.122.2/24 24 = 前24位是网络号，那么后8位就是主机号。 那如何用？ 如发送行信息给 10.100.122.255 所有以 10.100.122... 开头的机器都能收到。 于是有了两个概念： 广播地址：10.100.122.255 子网掩码：255.255.255.0 -> AND 得到网络号。  3. 每一个城市都有人民广场，IP设计是如何解决的？  公有IP地址和私有IP地址。 搭建世界人民都可以访问的网站，需要共有IP地址 搭建只有学校同学使用饿的网站，只要私有IP地址 例子1: Wi-Fi 192.168.0.x 是最常用的私有 IP 地址 192.168.0 是网络号 192.168.0.1，往往就是你这个私有网络的出口地址 192.168.0.255 就是广播地址。一旦发送这个地址，整个 192.168.0 网络里面的所有机器都能收到。 例子2: 16.158.165.91/22 4. 如何理解MAC地址？  如果说IP是地址，有定位功能。那Mac就是身份证，唯一识别。  ## 琐碎：  5. 讲了ABC，那是D类是什么？  D 类是组播地址。使用这一类地址，属于某个组的机器都能收到。这有点类似在公司里面大家都加入了一个邮件组。发送邮件，加入这个组的都能收到。组播地址在后面讲述 VXLAN 协议的时候会提到。  6. IP地址scope是什么意思？  对于 eth0 这张网卡来讲，是 global，说明这张网卡是可以对外的，可以接收来自各个地方的包。对于 lo 来讲，是 host，说明这张网卡仅仅可以供本机相互通信。  7. 那lo是什么意思？  lo 全称是loopback，又称环回接口，往往会被分配到 127.0.0.1 这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现。  8. < BROADCAST,MULTICAST,UP,LOWER_UP > 是干什么的？  net_device flags，网络设备的状态标识。 UP 表示网卡处于启动的状态； BROADCAST 表示这个网卡有广播地址，可以发送广播包； MULTICAST 表示网卡可以发送多播包； LOWER_UP 表示 L1 是启动的，也即网线插着呢。  9. MTU1500 是指什么意思呢？是哪一层的概念？  最大传输单元 MTU 为 1500，这是以太网的默认值。 MTU 是二层 MAC 层的概念。MAC 层有 MAC 的头，以太网规定连 MAC 头带正文合起来，不允许超过 1500 个字节。  10. qdisc pfifo_fast 是什么意思呢？  排队规则。规定数据包如何进出的。有pfifo, pfifo_fast.  

  2018-07-07

  **

  **17

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  来生树

  采精，唯一不足，这个课程定价严重定低了，哈哈

  2018-05-23

  **2

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5b/b9/d07de7c6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FLOSS

  现在很多工具都可以更改本机的MAC地址，也就是网络上存在很多MAC地址被更改成一样的，然而并没有出现通讯异常或者混乱这是为什么？

  作者回复: 会啊，如果你创建虚拟机，复制的时候，没有原则重新生成mac，你就发现你连不上了

  2018-05-24

  **

  **15

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5b/b9/d07de7c6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FLOSS

  还是没理解，我和同事的电脑就是相同的MAC地址，我们再各自的家里上网都是正常的。

  作者回复: 哈哈，不在一个局域网，当然没问题了

  2018-05-25

  **4

  **14

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Norman

  老师可以给推荐一本网络协议相关的书吗？我是小白，之前没有系统学习过网络协议，想好好看一下

  2018-05-23

  **

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/0f/50/34/fefb125f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  笨笨熊

  @Norman，wireshark网络分析就这么简单 这本书不错，推荐给你！

  作者回复: 谢谢

  2018-05-24

  **2

  **13

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  hellsplum

  （CIDR 可以用来判断是不是本地人）这句话，哪位大神帮我讲解下是什么意思呢~~~

  作者回复: CIDR是192.168.23.101/24，如果对方的IP是192.168.23.9，就是本地人，如果是192.168.24.9就不是

  2019-01-16

  **3

  **12

- ![img](https://static001.geekbang.org/account/avatar/00/12/cd/1c/98b32e26.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  雷大鸡

  其实全部使用 MAC 地址也是可以通讯的，但是那样需要维护一个巨大无比的路由表，或者要求生产出来的网卡按照约定给与 MAC 地址并分配到特定的地区使用，形成类似 IP 的寻址机制，但显然那样耗时耗力，不如 IP 机制简洁。

  作者回复: 是的，这就是把定位功能给了mac

  2018-10-12

  **

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/68/bef5f7d3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  metalmac.kyle

  浅显易懂，言简意赅，网络的书看了很多加上实践和作者的讲解豁然开朗，重温复习很有效，如果能再搭配一些相关书籍章节的参考和深入更赞了👍

  作者回复: 谢谢，看来我要出一个推荐书列表了

  2018-05-27

  **

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/13/11/3e/925aa996.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  HelloBug

  老师，你好~MAC地址是网络生产商写在网卡上的，相当于是写死的。但是iproute2里有命令可以修改网卡的MAC地址，是不是只有虚拟机上的系统的网卡可以修改，而物理网卡的MAC地址是不能修改的？

  作者回复: 物理网卡可以改，但改的是系统里面的，不是硬件里面的

  2018-12-02

  **

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f8/ba/14e05601.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  约书亚

  我就是来纯感谢的，天天ip addr，好多内容也不清楚干什么的。

  作者回复: 谢谢

  2018-05-23

  **

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/12/e0/99/5d603697.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  MJ

  再补充一个在linux系统下查看ip的命令：netstat -ie

  2018-11-26

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/74/34409399.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Chain

  课程几十块钱，感觉我捡便宜了😂️😂️😂️

  作者回复: 谢谢

  2018-05-25

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/16/4c/a8/2dbe1e90.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Build

  Ip用于网络寻址，mac在一个局域网内唯一

  作者回复: 是的

  2019-04-13

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/67/5633ffc3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  未成年

  老师有个问题，如果包在路途当中，指向的ip突然断了，然后又随机随机分配了另一个ip，而原有的ip又被重新分配到了新的一台机器上，此时路途中的包，按ip招到了新的机器，但是mac对不上号了，就等于我在路上呢那家人家搬家了住了个新的进来，走过去一看不是，那这个包应该如何处理

  2018-09-26

  **1

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8a/5f/c60d0ffe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  硅谷居士

  MTU1500是指 mac 层的有效载荷长度还是mac 头加上有效载荷长度呢？看一些资料都说是 mac 的有效载荷。

  2018-05-23

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/17/8c/14/86736fb7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晴枫慕竹

  有个问题，就是文中说10.100.122.2/24，网络号是24位，后8位是主机号，按这个来讲它应该属于C类地址，但是从那个表格来看这个IP看他是10开头的，应该又属于A类，这样是否矛盾，还是我哪里理解错了，谢谢

  作者回复: 无类间，就是不分类了，CIDR

  2019-05-25

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ae/e0/3e636955.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李博越

  能讲讲子网掩码跟网关之间，具体的关系是怎样的？

  作者回复: 一个子网里面中的一台机器是默认出口，就是网关

  2018-09-19

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/67/95/5bd8911f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinwang527

  net-tools起源于BSD的TCP/IP工具箱，后来成为老版本Linux内核中配置网络功能的工具。但自2001年起，Linux社区已经对其停止维护。一些Linux发行版则已经完全抛弃了net-tools，只支持iproute2。iproute2的出现旨在从功能上取代net-tools。net-tools通过procfs(/proc)和ioctl系统调用去访问和改变内核网络配置，而iproute2则通过netlink套接字接口与内核通讯。抛开性能而言，iproute2的用户接口比net-tools显得更加直观，命名更加规范。更重要的是，到目前为止，iproute2仍处在持续开发中。 

  2018-06-26

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/4c/09/6de8b2f6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张晶晶

  11分钟的时候说ip地址像身份证，结束的时候说mac地址像身份证，是不是有一个说错了？

  作者回复: 应该是读错了，文本为准

  2018-06-01

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/8a/4b/9863a07b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  timgise

  第一个地址是 16.158.<101001><00> 这里00是为了补齐8bit吧

  2018-05-23

  **1

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/94/bd/3b61c1c1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  origin

  \1. A,B,C网络分类法不实用 2. CIDR无类型域间选路(网络号+主机号) 3. IP地址 & 子网掩码 = 网络号 4. loopback: 127.0.0.1 5. IP是地址，可以远程定位; MAC是身份证，只能局域网定位

  2018-11-28

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/50/4d/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  richey

  大师，最后总结说CIDR 可以用来判断是不是本地人，我看了好几遍没发现文中有说啊，指的是根据公有ip或私有ip判断？望解答。

  作者回复: 私有ip,也就是本机能看到的ip

  2018-08-23

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/bd/e3c73a66.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mouri

  如果把IP为192.168.1.2的主机和IP为192.168.2.2的主机连到不带Vlan的传统交换机，是不是也可以相互通信？

  作者回复: 没写cidr如果24就不能，16就能

  2018-05-25

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/5e/c8/d5a1c6fd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  许波

  容易犯错的CIDR部分想了好久才理解

  2018-05-23

  **1

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/17/01/38/5daf2cfb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吴军旗^_^

  为什么A、B、C类的私有地址是以10、172、192开头的， 没有太明白

  作者回复: 就是分类而已，没有原因

  2019-05-06

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/82/4d/ed67c248.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鹅的天牙(Dain) 🐜

  我们来看 16.158.165.91/22 这个 CIDR。求一下这个网络的第一个地址、子网掩码和广播地址。 你要是上来就写 16.158.165.1，那就大错特错了。 /22 不是 8 的整数倍，不好办，只能先变成二进制来看。16.158 的部分不会动，它占了前 16 位。中间的 165，变为二进制为‭10100101‬。除了前面的 16 位，还剩 6 位。所以，这 8 位中前 6 位是网络号，16.158.<101001>，而 <01>.91 是机器号。 第一个地址是 16.158.<101001><00>.1，即 16.158.164.1。子网掩码是 255.255.<111111><00>.0，即 255.255.252.0。广播地址为 16.158.<101001><11>.255，即 16.158.167.255 不明白这是怎么算的。 怎么得到16.158.164.1的？

  2018-12-12

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/f9/90/f90903e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜菜

  老师好！想请问32位的IP地址不够，为什么48位的MAC地址就足够呢，毕竟它也是全球独一无二的啊。特别是现在IoT设备数量急剧增加，网卡数量应该也在急剧增加，所以。。。

  作者回复: 一个局域网不重复就行，概率很小

  2018-11-29

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/26/12445259.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  翁秀玲

  以前在学校上过计算机网络课程，前2讲主要是宏观介绍，先让读者有个大的网络轮廓，收获不大，反而有点能懂。这讲不错噢。继续保持。

  作者回复: 谢谢

  2018-05-26

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/48/73/cefaaf71.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  alex

  能讲讲广播，组播的实际应用场景吗，有没有具体案例

  作者回复: 组播在讲vxlan的时候会讲到。

  2018-05-23

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/e9/86/d34800a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  heyman

  CIDR的私有IP地址范围是什么？CIDR和A、B、C类的关系是什么？怎么决定一个网络是用哪种类型？网络设备怎么知道？希望老师可以解答一下～

  2022-06-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/18/c5/2e/0326eefc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Larry

  如果公网ip分配的A类，内网多台机器，只能用A类的私有ip？

  2020-03-18

  **1

  **1

- ![img](https://wx.qlogo.cn/mmopen/vi_32/HfnrQlGW842BmSeVG7iciazV7U2CaYbODPbQ144qG1pSgHjhCW3OziaoqKJISfoZlEBsWsuVZtkLdCvuNUoibnsUfw/132)

  Geek_hander

  这个关于子网掩码的解释似乎更直观。“通过计算机的子网掩码判断两台计算机是否属于同一网段的方法是，将计算机十进制的IP地址和子网掩码转换为二进制的形式，然后进行二进制“与”(AND)计算（全1则得1，不全1则得0），如果得出的结果是相同的，那么这两台计算机就属于同一网段。”

  2020-03-16

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/15/f7/744720a2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  DriveMan_邱佳源

  这节主要理解cidr和由ip addr命令符打印出来的内容，理解网络号/主机号，如何根据cidr求出子网掩码/广播地址，ip addr打印出来的内容分为两部分：lo / eth0 ，lo为本地连接【了解即可】，eth0为网络连接【主要理解 ：mac地址/公网地址/ipv6/网络设备标识状态/传输网络包字节大小/队列规则】

  2019-11-26

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/17/31/7a/f8b0a282.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Januarius

  我觉得讲的挺好的，第一遍好多没有看懂，多看几遍整理下笔记，就很透彻了，想要听一遍就什么都搞明白了，不大现实

  作者回复: 对的，一遍不行两遍，加油

  2019-08-10

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/97/78/97934c1e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LZY

  老师你好，有一个疑问，若PC0的本机IP地址为192.168.0.2。那么192.168.0.2/24表示的含义，是表示一个网段（192.168.0.0~192.168.0.255），该网段中有一个IP地址是192.168.0.2；还是说表示一个IP地址192.168.0.2，该IP地址属于网段192.168.0.0~192.168.0.255？也就是说这种写法表示的到底是一个网段还是一个具体IP地址？

  作者回复: 是子网掩码的算法。

  2019-07-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/18/3c/2a/446d7a1f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ruter

  CentOS精简版就只有ip addr

  作者回复: ip addr是新的

  2019-07-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/a9/79/16c46704.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_Zu

  私有IP地址的范围是如何确定的？

  作者回复: 规定的

  2019-04-06

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/c6/ac/60d1fd42.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  抽离の❤️

  老师，你好！为什么第一个地址拼接00，子网掩码拼接00，广播地址拼接11？

  作者回复: 这是约定

  2019-01-10

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKV9oQgKw3jv0IdVJicOQ8Z8UBDibOJqP021cibsb6XGibskqicY27VMbuUTibq7avMiaoCER6UBzQacVuIg/132)

  超级用户

  MAC地址有可能耗尽吗？

  作者回复: 没问题，只要局域网冲突概率低就可以了，参考uuid

  2018-12-03

  **

  **1

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/ojzLmQukvUQ9HenEnpm1YXqvnzAezZBQlJRf2BBpe6RvmbRFECVYUXbH3XhSRrfMuDk4w63ia7PVo8Ge6kibficeg/132)

  HoKwan

  老师，我有一处理解的不是很清晰，IP的ABC分类法基本弃用了，换成了CIDR，但是ABC分类法中的私有IP段继续沿用了下来（因为这样划分出来的子网基本满足私人的需求），请问是这样理解么？

  作者回复: 是的

  2018-11-24

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/22/69/09f7a8a2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Don Wang

  超神，跟着 openstack 博客过来的

  2018-11-23

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e9/e9/31b68852.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天宇

  写的确实很有趣味，而且还透彻，划算！

  2018-09-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/b5/39affaad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Hkesd

  一个A类的公网ip会有一个B类的私网IP吗？？为什么我的阿里云上会出现这种情况？？

  作者回复: 经典网络有两个网卡，vpc会有一个临时外网ip

  2018-09-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/b5/39affaad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Hkesd

  老师，为什么c类地址的网络号不能是5位呢？这样的话主机数不就够了吗？

  作者回复: 当时设计的时候比较死板。如果能够随便自己定位数，其实就是cidr了

  2018-09-18

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/69/21/7db9faf1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  简迷离

  对于广播地址192.168.0.255，一旦发送这个地址，整个 192.168.0 网络里面的所有机器都能收到。这里的“一旦发送这个地址”怎么理解，是不是向广播地址发送消息，广播地址所在的整个网路的机器都能收到的意思？

  作者回复: 是的

  2018-09-12

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  楼上的抓包验证，是怎么做的，求指导一下

  2018-09-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/ca/45/ded13010.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Radon

  C类地址的主机数为什么254个，不是256个？什么是网络地址和广播地址，其意义是什么？组播地址是做什么用的，能否介绍一下组播原理？

  2018-07-09

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/67/95/5bd8911f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinwang527

  关于IP地址的由来：IP地址可以说是冷战的产物，最早是由美国国防部为了满足战争的需要而建立的实验性网络——阿帕网（ARPAnet，Internet的前身）所使用的，当时出于实验目的，人们没有想到IP协议会得到广泛的应用，会发展的如此迅猛，由于设计的不合理，IP地址出现分配的问题，IP地址资源变得紧张。

  2018-06-26

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/a5/cd/3aff5d57.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alery

  “qdisc 全称是queueing discipline，中文叫排队规则。内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的 qdisc（排队规则）把数据包加入队列。” 刘老师，上面一段话提到的网络接口是指网卡吗？

  作者回复: 是的，也有可能是虚拟网卡

  2018-06-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/df/38/3552c2ae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  匿名用户

  深度好文，接地气。当时大学的时候计算机网络才80不到，要是听你这课，90稳了。

  2018-06-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/88/b4/72f61627.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  老黑

  老师讲的很好，通俗易懂。 仔细看完后，我有个问题。就是网络号，ip地址，子网掩码和广播地址的先后关系是是么呢？是先有网络号，后有ip地址和子网掩码呢，还是先有ip地址子网掩码后有网络号呢？广播地址就是该网络段的最后一个ip地址吗？ 再次感谢老师的课。

  2018-05-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/18/cb/edb5a0a0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小橙橙

  我没有系统的学习过网络协议，课程整体上听得比较吃力，老师可以推荐一本相关书籍，辅助学习吗？

  2018-05-30

  **

  **1

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q3auHgzwzM4BtLmq9ia6sWuwQMibrmdDGWkOzng6kUxick42mLEXnhojTQFF9blxPysafKJnwzswaSenJkZJGlWYQ/132)

  smart

  请问下老师，IP 地址中 D 类多播地址跟子网中的网络地址有什么区别呢？

  2018-05-28

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/32/08/17fc5b22.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  语鬼

  好文！可能很多人了解上半部分，但网络设备状态标示都不是很关注。涨知识了

  作者回复: 谢谢

  2018-05-24

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/2b/89/50/aee9fdab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  小杰

  第一个问题，我看大家都回答了，我搜索了ip地址的由来，居然是冷战时期搞出来的，但是没想到居然那么受欢迎😂。

  2022-09-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/e1/d2/42ad2c87.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  今夜秋风和

  A 类地址比如127.0.0.2,127.0.0.3 这些地址在什么情况下才会被用到？

  2022-08-20

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIlk0ZTU5Jn4Pl4Z5MB71c0RR0iayBb3wpwveBkg0G2sqm2ZiaxWc1YiblT3RhOMAK2pQEy5QNvuR3ibw/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  安权

  表格中的 192.168.0.x 是最常用的私有 IP 地址。你家里有 Wi-Fi，对应就会有一个 IP 地址。一般你家里地上网设备不会超过 256 个，所以 /24 基本就够了。有时候我们也能见到 /16 的 CIDR，这两种是最常见的，也是最容易理解的。 上文中256是不是应该是254，第一可用是xx.xx.xx.1 最后一个可用是xx.xx.xx.254，应该是254个设备？

  2022-07-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/55/5c/643c0e33.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  ShiPF

  ip地址是逻辑层寻址。mac是物理层寻址。arp逻辑层到物理层转换。这么理解有问题不？

  2022-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/bf/ef/6f6a0c1f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  风中之心

  讲错了吧，1和任何数取and都是1和0和任何数取and等于0他俩矛盾啊

  2022-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/84/2c/1b0926b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Even

  C类地址用IP地址前24位表示网络ID，用IP地址后8位表示主机ID。C类地址用来表示网络ID的前三位必须以110开始，其他22位可以是任 意值，当其他22位全为0是网络ID最小，IP地址的第一个字节为192；当其他22位全为1时网络ID最大，第一个字节数最大，即为223。

  2022-06-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/84/2c/1b0926b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Even

  B类地址用IP地址前16位表示网络ID，用IP地址后16位表示主机ID。B类地址用来表示网络ID的前两位必须以10开始，其他14位可以是任 意值，当其他14位全为0是网络ID最小，即为128；当其他14位全为1时网络ID最大，第一个字节数最大，即为191。B类IP地址第一个字节的有效 范围为128－191，共16384个B类网络；

  2022-06-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/84/2c/1b0926b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Even

  A类地址用IP地址前8位表示网络ID，用IP地址后24位表示主机ID。A类地址用来表示网络ID的第一位必须以0开始，其他7位可以是任意值， 当其他7位全为0是网络ID最小，即为0；当其他7位全为1时网络ID最大，即为127。网络ID不能为0，它有特殊的用途，用来表示所有网段，所以网络 ID最小为1；网络ID也不能为127；127用来作为网络回路测试用。所以A类网络网络ID的有效范围是1-126共126个网络

  2022-06-09

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Geek_0386d2

  」

  2022-05-31

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIn0FpbcH9JA3e3bEibYFjicPVKibJuIicx1LQOkJtbn6QK4Rdsrib3Kcqb54iaK7P5f0f4cYibfkXIoThzA/132)

  Dong.sir

  文中很多地方提到一个“大吼一声“，这个大吼是发送的一个广播，本网络的所有设备都能收到这个信息么，然后开始解析处理，如果其中一个设备是一个特殊的，比如他什么数据都想要，那他能否对所有的数据进行接收，实现监听的功能，如果这个设备还有点不守规矩，返回一些错误的那内容，这种是否就实现了破坏

  2022-05-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/11/ed/722a1d06.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

   大头

  请教一下老师和各位同学，有点没搞明白私有 IP，假设有这样一个 IP：192.168.1.1/24，这个局域网下要分配私有 IP 是不是只能利用后 8 位主机号进行分配，即可以分配的私有 IP 为 192.168.1.2 ~ 192.168.1.254（剔除私网出口IP和广播IP）

  2022-01-01

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek2103

  不明白。既然IP是寻址功能，为什么IP不放在最下层呢，解析判断IP是不是自己这个局域网，如果是再由mac地址定位具体的计算机。而是mac地址在最下层，每次都判断mac地址，多此一举。

  2021-12-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_1e5cce

  这里觉得应该将一下子网掩码的计算方法 可以CIDR直接推出子网掩码 

  2021-11-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/97/79/71aba0c4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黄维一

  只能说讲的太好了， 平时都只关注什么ip 地址之类的。对ip addr的其它信息也不了解，现在是知道了更多了

  2021-09-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5e/61/985f3eb7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  songyy

  有点困惑 -- 文章的标题是 ifconfig，但是文章内容讲的是 ip addr

  2021-08-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_5e926f

  我觉得可能就类似初学者学编程一样，第一次读遇见不懂的不用担心，硬着头皮往下读，所有内容过了一遍后再看第二遍，可能就会恍然大悟

  2021-08-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Binary

  推荐一本书《网络是怎样连接的》，很通俗，看完再来看专栏很舒服

  2021-08-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/29/53/92/21c78176.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小黄鸭

  不懂，私有地址的范围是怎么来的，既然是192.168.0.0-192.168.255.255，这应该是255*254个主机数吧？

  2021-07-24

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  小甲鱼

  我想问，我请求的时候，我知道目标地址的ip，但是，我是如何知道目标的mac地址的？通过ip找，如果找到了一个内网里，我怎么知道我要找哪个mac地址呢

  2021-07-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/01/c5/b48d25da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  cake

  老师 请问下一个困扰我很久的问题  我Linux 经常过一段时间 我 ip addr ip就只剩127.0.0.1了,导致外面机器连接不了里面的服务,网上的办法我也找遍了,每次都以重装为结束,不知道是什么导致的这个原因, 我今天连不上了,记起来以前有个快照,我就恢复快照,又能连上了。  

  2021-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/f5/a2/8a470344.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Run_dream

  ABC类网络分类还有意义吗 有实际应用场景吗

  2021-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/1d/3e/e2c6e678.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一途

  对入门小白来说，真的很好理解。例子特别的容易明白。就是在一些新名词出现的时候，希望老师能提一下在本节里这个名词不需要掌握或者深究。哈哈哈哈不然挺打击学习积极性的。

  2021-05-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/ee/ae/855b7e6e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Gabriel

  怎么查看计算机上的ip地址呢？windows ipconfig lunix ifcofig ipaddr 老师将的真的不错，比很多教学书籍好很多。 ip地址和mac地址 ip地址就像分配的一具体地址。比如是哪条路，这条路附近是什么， mac地址就像房地产分配的地址一样。 mac通讯的长度是有限的，就像在一个小区，通信没有问题。 然后将了通信规则，通信规则其实也像通往小区的门口，进门需要刷卡，人多了需要排序。 进门时候，刷卡有可能有一个，或者多个，就像传输速率一样。 书读百遍其义自见，我听了两遍，看了两遍。一遍比一遍可能理解的不一样，会有自己感悟。

  2021-04-20

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTICmZyq1WgQh5TjMETS502trkgurxCYEVWTnf982nDGk5jSWtxvcuKAHytvDVyVk7m7EaTU9hF3ibA/132)

  Geek_a27fb4

  建议把ifconfig的结果也放出来

  2021-03-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/4c/12/76fc82b6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Co🥥chO

  私有IP和公有IP这里有不理解的，IP adder显示的里面lo是只能局域网里访问那么应该是私有IP，但是底下的IP又不在私有段里，global网卡的IP是公有但是又在私有段里，整懵了

  2021-03-29

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/ia5NFICdEuzaQ8Vib0depvkB6UmxPBFib51aClSJYfCIa7tn2nXauddwxDvbxYuQ9UeRGVICLfTtDJysnDJ5EfQcg/132)

  Geek_8866d4

  ip地址是怎么实现区域定位的呢，还有路由器是怎么实现跨网段的呢？

  2021-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/23/ee/5b/42e178ad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  念雅

  容易出错的这个例子看半天没懂。

  2021-03-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  201201837

  我们学校网络的192.168.1.85访问其他学校的192.168.1.85，这个封装数据包时应该怎么封装呢。起始ip和目的ip是一样的吗？

  2021-03-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/06/a2/350c4af0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  知易

  以下来自，仰望星空，同学的评论。 解题思路 1、首先求子网掩码，子网掩码掩盖网络号，暴露主机号，用1表示掩盖，用0表示暴露。22表示网络号有22位，那么主机号有10位，因此子网掩码的二进制的值为： 11111111.11111111.11111100.00000000，转换为十进制：255.255.252.0 2、再求网络号，将子网掩码和 IP 地址按位计算 AND，就可得到网络号， 16.158.165.91的二进制： ‭00010000.‬10011110.‭10100101‬.‭01011011‬ AND 11111111.11111111.11111100.00000000 = 00010000.10011110.10100100.00000000 = 16.158.164.0， 那么第一个网络号（主机号的最后一位是1）为： 00010000.10011110.10100100.00000001=16.158.164.1， 最后一个网路号也即广播地址（主机号的所有位是1）为： 00010000.10011110.10100111.11111111=16.158.167.255

  2021-03-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c7/05/19c5c255.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  微末凡尘

  第三讲笔记  # 面试考点： 1. ip addr → 不知道基本没有用Linux 2. ifconfig 和 ip addr 的区别吗？ 3. CIDR 4. 共有IP和私有IP 5. MAC地址 6. 网络设备的状态标识 # 知识点： ## 核心： 1. IP设计时犯的错误？ 低估了未来网络的发展，32位地址不够用。于是有了现在IPv6（128位） 分类错误。分成了5类。C类太少，B类太多。C类254个，网络都不够；D类6万多，给企业都太多。 2. 那后来者如何弥补IP设计者犯的错误呢？ CIDR，无类型域间选路。 打破原来几类地址设计的做法，将32位IP地址一分二，前者网络号，后者主机号。 如何分呢？ 栗子：10.100.122.2/24 24 = 前24位是网络号，那么后8位就是主机号。 那如何用？ 如发送行信息给 10.100.122.255 所有以 10.100.122... 开头的机器都能收到。 于是有了两个概念： 广播地址：10.100.122.255 子网掩码：255.255.255.0 -> AND 得到网络号。 3. 每一个城市都有人民广场，IP设计是如何解决的？ 公有IP地址和私有IP地址。 搭建世界人民都可以访问的网站，需要共有IP地址 搭建只有学校同学使用饿的网站，只要私有IP地址 例子1: Wi-Fi 192.168.0.x 是最常用的私有 IP 地址 192.168.0 是网络号 192.168.0.1，往往就是你这个私有网络的出口地址 192.168.0.255 就是广播地址。一旦发送这个地址，整个 192.168.0 网络里面的所有机器都能收到。 例子2: 16.158.165.91/22 4. 如何理解MAC地址？ 如果说IP是地址，有定位功能。那Mac就是身份证，唯一识别。 ## 琐碎： 5. 讲了ABC，那是D类是什么？ D 类是组播地址。使用这一类地址，属于某个组的机器都能收到。这有点类似在公司里面大家都加入了一个邮件组。发送邮件，加入这个组的都能收到。组播地址在后面讲述 VXLAN 协议的时候会提到。 6. IP地址scope是什么意思？ 对于 eth0 这张网卡来讲，是 global，说明这张网卡是可以对外的，可以接收来自各个地方的包。对于 lo 来讲，是 host，说明这张网卡仅仅可以供本机相互通信。 7. 那lo是什么意思？ lo 全称是loopback，又称环回接口，往往会被分配到 127.0.0.1 这个地址。这个地址用于本机通信，经过内核处理后直接返回，不会在任何网络中出现。 8. < BROADCAST,MULTICAST,UP,LOWER_UP > 是干什么的？ net_device flags，网络设备的状态标识。 UP 表示网卡处于启动的状态； BROADCAST 表示这个网卡有广播地址，可以发送广播包； MULTICAST 表示网卡可以发送多播包； LOWER_UP 表示 L1 是启动的，也即网线插着呢。 9. MTU1500 是指什么意思呢？是哪一层的概念？ 最大传输单元 MTU 为 1500，这是以太网的默认值。 MTU 是二层 MAC 层的概念。MAC 层有 MAC 的头，以太网规定连 MAC 头带正文合起来，不允许超过 1500 个字节。 10. qdisc pfifo_fast 是什么意思呢？ 排队规则。规定数据包如何进出的。有pfifo, pfifo_fast.

  2021-03-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6a/7b/a303078f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  雷夏

  老师，苹果电脑和windows电脑的访问网络都是一样的吗？为什么同一无线网络，windows可以ping 通服务器ip,而苹果电脑系统不可以呢？

  2020-12-22

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_69fd35

  原为：6 万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。 公司用B类地址闲置的地址怎么算浪费呢，后期有机器加进去就可以了啊，另外一个公司B 完全可以用B类地址啊，两个公司肯定是两个不同的局域网，也不会造成冲突啊， 这个地方没明白，希望能解释下

  2020-12-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/3a/4c/b6200773.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一步

  网关：A default gateway is the node in a computer network using the internet protocol suite that serves as the forwarding host (router) to other networks when no other route specification matches the destination IP address of a packet. 掩码:For IPv4, a network may also be characterized by its subnet mask or netmask, which is the bitmask that when applied by a bitwise AND operation to any IP address in the network, yields the routing prefix。 A media access control address (MAC address) is a unique identifier assigned to a network interface controller  Classless Inter-Domain Routing (CIDR /ˈsaɪdər, ˈsɪ-/) is a method for allocating IP addresses and for IP routing.  Its goal was to slow the growth of routing tables on routers across the Internet, and to help slow the rapid exhaustion of IPv4 addresses.

  2020-11-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/f9/d3/2cb7516e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ？

  开始看到c类地址有点困惑，为何范围是192.168.0.0到192.168.255.255，这不有254×254个地址吗。后面想起来这个一个c类地址是前24位是网络号，后面才是主机号，所以是只能放254台主机。同理可以用cidr中。就是我们常说的网断，比如我们用16位的网络号，那么就有254×254的机器是在同一个网断，这个时候不用路由也能够在一个局域网通信。

  2020-11-18

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/oltLEqTrmHm2aJP99BK6tHu5h7hp4aj08wR5Wt6H31iadFduDAVvjYKmhQ2nvGbLV3lkVdiat2GRasgWXoJeTibUg/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  杨

  16.158.165.91/20 第一个地址是 16.158.<1010><0000>.1，即 16.158.160.1。子网掩码是 255.255.<1111><0000>.0，即 255.255.240.0。广播地址为 16.158.<1010><1111>.255，即 16.158.175.255  是这样算的吧

  2020-10-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f3/b7/d7377f0c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Vincent

  IPv4一共32位，分4组数字，每组8位； 通过ip addr获取到的IPv6一共有6组数字，每组数中有4个数字，每个数字都是十六进制，这样算下来每组一共有16位，如果要满足128位就必须有8组数字，为何ip addr获取到的只有6组呢！

  2020-09-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  一个简单命令解读包含了：IP地址分类、CIDR、MTU等基础重要的知识点，真的是先把书读厚，再把书读薄，赞。

  2020-09-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/62/40/faf9a818.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吃茫茫

  二刷清楚了许多✌🏻️

  2020-09-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/f4/aa/4bdc097a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  伊辰

  🧐学习还是要主动些的。什么AND计算，按位与逻辑与以及IP组成啥的都可以百度下，花十分钟摸索下，了解文中这些网络号怎么得出的很有必要

  2020-08-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/57/d8/5816eb6b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  啦啦啦

  赞，解决了之前压测的一个问题

  2020-08-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/d6/01/2448b4a9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  py

  网络号干啥的？内网IP？

  2020-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/de/bf524817.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  慌张而黑糖

  我看书上说cidr能够缓解ipv4地址的压力，我在想ip地址就这么多，即使再划分也不会比原来多呀，这里不太理解，请老师和同学们指点一二

  2020-06-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/39/d8/9727b45c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mångata

  思考题1 Net-tools和iproute2的历史故事（该思考题答案参考了博客https://www.cnblogs.com/leisurelylicht/p/nettools-he-iproute2-de-li-shi-gu-shi.html） net-tools起源于BSD，自2001年起，Linux社区已经对其停止维护，而iproute2旨在取代net-tools，并提供了一些新功能。一些Linux发行版已经停止支持net-tools，只支持iproute2。这应该是Ubuntu18不识别ifconfig的原因。 net-tools通过procfs(/proc)和ioctl系统调用去访问和改变内核网络配置，而iproute2则通过netlink套接字接口与内核通讯。 net-tools中工具的名字比较杂乱，而iproute2则相对整齐和直观，基本是ip命令加后面的子命令。 虽然取代意图很明显，但是这么多年过去了，net-tools依然还在被广泛使用，最好还是两套命令都掌握吧。  思考题2 IP地址是怎么来的？ 公有IP由特定的机构分配。不是很理解这个思考题要问什么。

  2020-06-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9c/62/f625b2bb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  酸葡萄

  老师你好,"IP 是地址，有定位功能；MAC 是身份证，无定位功能."   如果mac地址可以和我们身份证号一样,固定区位采用固定的标识,应该也是可以取代IP的吧?

  2020-05-17

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epEuxNkyqFG5Gpicu8XALibeicQLOlicL64tKw40J4zTWOvhAJV6CJu2GVzJG0UucIaSKzia5hzxicyKQaQ/132)

  张舒

  如果发送数据内容大于1500字节，到二层时，二层自动拆分数据吗？

  2020-05-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/47/02/84e7237f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Lucky Box

  hostname -I 也能查ip

  2020-05-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/39/d2/845c0e39.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  送过快递的码农

  就是 我如果访问一个局域网里面的私有ip 网关是怎么知道 这是私有的还是公有的 公有的ip 有个111.111.111.111  私有的也是111.111.111.111 网关怎么知道的

  2020-05-09

  **1

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  朝闻道

  老师，flags=4163是什么意思呢？

  2020-05-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d6/39/6b45878d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  意无尽

  总结： 1) 查看 IP 地址的方式： Windows 上在命令行输入 ipconfig，Linux 上有两种，一种是 ifconfig，另一种是 ip addr。ip addr 命令能显示了这台机器上所有的网卡。 2) IP 地址是一个网卡在网络世界的通讯地址，相当于我们现实世界的门牌号码。而 32 位的 IP 地址现在已经无法满足我们的需求，而且还被分成了 ABCDE 五类，导致更加不够用了。于是就有了无类型域间选路(CIDR)。将 32 位的 IP 地址一分为二，前面是网络号，后面是主机号。 3) 公有 IP 和私有 IP。私有 IP 用于比如学校，办公室或者数据中心等场景，因为这些允许组织内部人员进行管理和分配。而公有 IP 一般是由相关组织进行统一分配。 4) MAC地址。是一个网卡的物理地址。一个网络包要从一个地方传到另一个地方，除了要有确定的地址，还需要有定位功能。而 MAC 地址只能定位大致的范围，只有结合 IP 地址，才能准确定位。

  2020-04-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f6/28/baa9521c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  绝影

  私有ip范围是约定的吗，还是计算出来的

  2020-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  第一个字段是16.158.<10100100>.1,子网掩码是取AND,就是255.255.<111111100>.0,而广播地址是这个网段中最大的值,就是 16.158.<10100111>.255 去了前面的6位网段不变,后面的都取最大的,所以出现这个广播地址

  2020-04-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  熊佳利

  可以具体讲讲排队规则qdisc的类型吗？设备查到的是mq，不知道是什么意思？

  2020-04-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/00/9a/4d76df15.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王艺霖

  \1. A,B,C,D类和CIDR的关系仅仅是产生先后吗？现在用的是CIDR？ 2. 从中间那张图可以看出来，如果知道一台电脑的私有IP范围，是否就可以确定这台电脑所属于的网络的类型了？

  2020-04-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/19/b9/c3700576.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  以梦为马

  继续坚持

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/25/24/97dd57bc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  😊

  嗯 我想问一下  A类的网络号不是占8位,B类占16位,C类24位吗?前面 0 ,1 占的位置是什么意思呀?还有就是关于这个无类型域间选路 能在通俗点吗解释一下吗?麻烦老师了

  2020-03-24

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  小区、门牌号，这个形容网络号和主机号不错，之前也想过类似的例子，没这个形象，赞！

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/94/0a/7f7c9b25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  宋健

  收获很大

  2020-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  从IP地址出发见了与之相关的掩码，掩码可以区分IP地址的网络位和主机位（文中的的例子也是利于理解的）；再到IP地址的分类延续到目前网络世界地址分配的问题，谈到了CIDR这个东西的作用：打破了之前划分IP地址的原则，与之相关的概念有广播地址和子网掩码，也动手计算一下网段、第一个地址、广播地址和子网掩码（其中1和0的AND运算要清楚）。比喻不是显摆，是拉近知识之间的熟悉度：IP地址有定位功能（如你地址），MAC地址是你身份的唯一标识（如身份证）。

  2020-03-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f2/13/3ee5a9b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chenzesam

  学到更底层都是泪

  2020-03-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/42/fc/c89243d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  侯代烨

  mac类似于人的身份证，无定位功能 ip类似于门牌号，地址，有定位的功能

  2020-03-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/58/64/b715d45a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  何柄融

  没cidr时也是有网络号和主机号，有时也是这样，没看出区别，本人小白，想求问到底有了之后产生了什么变化

  2020-03-12

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  ricepot100

  作者能否讲讲历史上的多种网络体系，为什么tcp/ip最终赢得了广泛的胜利，像atm之类的就没落了

  2020-03-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/12/70/10faf04b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Lywane

  valid_lft forever preferred_lft forever这是啥意思

  2020-03-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/f1/aa/c29def94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  草裡菌

  怎么记得CIDR是为了解决骨干路由路由表过长的技术。它通过聚合ip前缀＋最长前缀匹配实现。比如有三个地址： 194.12.0.0/21  194.12.8.0/22  194.12.16.0/20 通过CIDR就可以聚合成194.12.0.0/19。

  2020-03-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/28/ca/47333d8b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lpqoang

  不知道老师还能不能看到了，我有个疑问。 我在局域网中有两个ip地址例如 192.168.1.2和192.168.1.3这两个地址，一个配16位掩码一个配24位掩码，也能通信，是为什么呢？

  2020-02-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/92/ce/9d24cb2c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小学一年级

  我们已经有了广播地址，那还要用到D类组播地址吗? 文中写到 发送广播地址同个网络段里都能收到，那这不是跟组播能达到的是同一个效果吗？ 不太明白 为什么计算机会分五类，依据是什么？

  2020-02-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/c1/b0/b52d9ade.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  苏彧

  老师，1和and值都是原来的值，这句话没怎么理解

  2020-01-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/c1/b0/b52d9ade.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  苏彧

  老师，这个ipv6地址怎样知道是128位呢

  2020-01-14

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Subhuti

  能说说交换机和路由器在整个网络传输中起到的不同左右吗？

  作者回复: 后面会详细讲

  2020-01-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/88/9c/cbc463e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  仰望星空

  第一个地址是 16.158.<101001><00>.1，即 16.158.164.1。子网掩码是 255.255.<111111><00>.0，即 255.255.252.0。广播地址为 16.158.<101001><11>.255，即 16.158.167.255，广播地址不应该是16.158.164.255吗？

  2019-12-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/24/5d/65e61dcb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  听雨

  为什么B类网络从192开始，C类网络从223开始，而不是其他数字呢，有什么讲究吗

  2019-12-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  ip addr 输出分析太赞了，以前就知道看 ip，mac。课程生动易理解，赞👍

  2019-11-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ce/53/d6a1f585.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  光羽隼

  CIDR这块有点不太好理解

  2019-11-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/7f/91/962eba1a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  唐朝首都

  分片传输，如何保证完整性呢？

  2019-11-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/96/3d/99c58108.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  董懂

  快把谢希仁那本教材看完了，回来再看这些才明白……

  2019-11-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/10/b7974690.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BD

  子网掩码是主机号全是0 ，想知道网络号就用子网掩码与ip地址取与运算 。ip地址分类 A b C D E分类不常用，但每个ip分类都有私有ip.和公有ip之分。

  2019-11-17

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLmWgscKlnjXiaBugNJ2ozMmZibAEKichZv7OfGwQX9voDicVy2qnKtlm5kWQAKZ414vFohR8FV5N9ZhA/132)

  菜鸡

  MTU到底包不包含mac的部分。我看到了另外一个版本讲以太网最大的mtu就是1500（它是不包含二层头部的）

  2019-11-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/53/49/5b8c0831.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  myron

  看完就忘怎么办

  2019-11-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/df/f517304b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  橙子

  超哥,16.158.165.91/22这个IP第一个数字16转换成2进制不是10000么?怎么占到8位的呢?前三位是0么?

  2019-11-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/55/cb/1efe460a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  渴望做梦

  老师，那阿里云主机的ip应该是公有ip吧，但是如果每申请一台云主机就分配一个ip不是很快就分配完了吗

  2019-10-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/28/31b0cf2f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑色毛衣

  MAC 也是有定位作用的，局域网内

  2019-10-20

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/28/31b0cf2f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑色毛衣

  MAC层 为什么是二层呢？应该是1 层 吧，毕竟最底层

  2019-10-20

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKU0MC8lDhUl8Z7kGVFRMUxK9iaBeKDSnhJ5mgD1lCzvXibjlDDicjeHeBfMyEttibREvG9BUpOeNHQbg/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Geek_4c94d2

  在虚拟机中，可以自己修改网卡。那如果虚拟机连上局域网，而这个虚拟机的mac地址，和一台主机mac地址一样的话，那会出现什么情况呢？另外，一些收费的软件的授权，是绑定mac地址的。那在虚拟机里面，将mac地址改成软件授权的mac地址，那是不是虚拟机就可以使用该软件了？

  2019-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/ab/58/ae036dfc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  月成FUN

  一直不太懂私有地址的含义。 以我现在的理解，私有地址就是每一类地址里预留出来的地址，不作为公有地址，只用作局域网内地址分配等

  2019-09-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/df/07/7bf65329.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  艾草

  作者大大： 在举CIDR的例子的时候，16.158.<101001><00>.1，我的疑问是第一个地址不应该是 16.158.<00><101001>.1吗？求出17-22位的二进制是<101001>,那么应该在左边补0吧。

  2019-09-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_fe2ac0

  老师，不是and运算后前面地址不变吗， 然后两个地址and后地址相同就是同一个子网

  2019-09-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  老师 我这个是tcp的三次握手吗？ 00:07:23.355608 IP 192.168.31.10.49481 > 182.61.62.11.https: Flags [S], seq 3348731978, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 1063881671 ecr 0,sackOK,eol], length 0 00:07:23.356301 IP 192.168.31.10.49482 > 180.88.56.35.https: Flags [S], seq 257197554, win 65535, options [mss 1460,nop,wscale 6,nop,nop,TS val 1063881671 ecr 0,sackOK,eol], length 0 00:07:23.363264 IP 182.61.62.11.https > 192.168.31.10.49481: Flags [S.], seq 1960980723, ack 3348731979, win 8192, options [mss 1440,nop,wscale 5,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,nop,sackOK,eol], length 0 00:07:23.363300 IP 192.168.31.10.49481 > 182.61.62.11.https: Flags [.], ack 1, win 4096, length 0 00:07:23.363823 IP 192.168.31.10.49481 > 182.61.62.11.https: Flags [P.], seq 1:518, ack 1, win 4096, length 517 00:07:23.365168 IP 180.88.56.35.https > 192.168.31.10.49482: Flags [S.], seq 3911074934, ack 257197555, win 65535, options [mss 1340,nop,nop,sackOK,nop,wscale 6], length 0 00:07:23.365197 IP 192.168.31.10.49482 > 180.88.56.35.https: Flags [.], ack 1, win 4096, length 0 00:07:23.365879 IP 192.168.31.10.49482 > 180.88.56.35.https: Flags [P.], seq 1:648, ack 1, win 4096, length 647 00:07:23.376412 IP 180.88.56.35.https > 192.168.31.10.49482: Flags [.], ack 648, win 1116, length 0 00:07:23.376417 IP 180.88.56.35.https > 192.168.31.10.49482: Flags [P.], seq 1:257, ack 648, win 1116, length 256 00:07:23.376460 IP 192.168.31.10.49482 > 180.88.56.35.https: Flags [.], ack 257, win 4092, length 0 00:07:23.376971 IP 192.168.31.10.49482 > 180.88.56.35.https: Flags [P.], seq 648:728, ack 257, win 4096, length 80 00:07:23.377637 IP 192.168.31.10.49482 > 180.88.56.35.https: Flags [P.], seq 728:1502, ack 257, win 4096, length 774

  作者回复: 没有看到syn呢

  2019-08-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  老师 您帮我看看 我这电脑是不是中病毒了 我执行了tcpdump (输完密码之后) 打印如下，恳请您帮我看看是不是有问题，因为我看到xiaoqiang.domain 了  23:57:46.357606 ARP, Request who-has 192.168.31.238 tell xiaoqiang, length 28 23:57:46.460821 ARP, Request who-has 192.168.31.243 tell xiaoqiang, length 28 23:57:46.461736 ARP, Request who-has 192.168.31.244 tell xiaoqiang, length 28 23:57:46.562417 IP 192.168.31.10.64727 > xiaoqiang.domain: 38115+ PTR? 245.31.168.192.in-addr.arpa. (45) 23:57:46.563287 ARP, Request who-has 192.168.31.247 tell xiaoqiang, length 28 23:57:46.564082 ARP, Request who-has 192.168.31.248 tell xiaoqiang, length 28 23:57:46.570005 IP xiaoqiang.domain > 192.168.31.10.64727: 38115 NXDomain* 0/0/0 (45) 23:57:46.571320 IP 192.168.31.10.57115 > xiaoqiang.domain: 39053+ PTR? 246.31.168.192.in-addr.arpa. (45) 23:57:46.577152 IP xiaoqiang.domain > 192.168.31.10.57115: 39053 NXDomain* 0/0/0 (45) 23:57:46.578406 IP 192.168.31.10.65285 > xiaoqiang.domain: 52337+ PTR? 247.31.168.192.in-addr.arpa. (45) 23:57:46.582173 IP xiaoqiang.domain > 192.168.31.10.65285: 52337 NXDomain* 0/0/0 (45) 23:57:46.583468 IP 192.168.31.10.52100 > xiaoqiang.domain: 32901+ PTR? 248.31.168.192.in-addr.arpa. (45) 23:57:46.588109 IP xiaoqiang.domain > 192.168.31.10.52100: 32901 NXDomain* 0/0/0 (45) 23:57:46.663520 ARP, Request who-has 192.168.31.249 tell xiaoqiang, length 28 23:57:46.664429 IP 192.168.31.10.65068 > xiaoqiang.domain: 62137+ PTR? 249.31.168.192.in-addr.arpa. (45) 23:57:46.664567 ARP, Request who-has 192.168.31.250 tell xiaoqiang, length 28 23:57:46.667569 IP xiaoqiang.domain > 192.168.31.10.65068: 62137 NXDomain* 0/0/0 (45) 23:57:46.668657 IP 192.168.31.10.49947 > xiaoqiang.domain: 36274+ PTR? 250.31.168.192.in-addr.arpa. (45) 23:57:46.673474 IP xiaoqiang.domain > 192.168.31.10.49947: 36274 NXDomain* 0/0/0 (45) 23:57:46.971475 ARP, Request who-has 192.168.31.251 tell xiaoqiang, length 28 23:57:46.971762 ARP, Request who-has 192.168.31.252 tell xiaoqiang, length 28 太长了 我删了一部分

  作者回复: 这个看不出来呀

  2019-08-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/dc/65/3da02c30.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  once

  有个问题想问下老师，ip这个东西是谁去分配的，比如我用的运营商是电信，我的ip是电信给我的么？，运营商有分配公网ip的能力么？那我的CIRD信息是怎么查询的，我的这个ip有多少个主机数，因为用ifconfig查询到的都是私有网络的ip地址，真正的公网的地址在百度上用ip工具才能看到，再往上看，电信公司拥有的ip又是怎么申请到的呢？

  作者回复: 你手机的ip地址是你家用路由器分给你的，你家用路由器的地址是电信分给他的。运营商有分配公网IP的能力。真正的公网肯定是被配置在某个网卡上了。

  2019-08-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/04/a6/18c4f73c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Airsaid

  您好，请问这么多网卡厂商，是如何保证 Mac 地址是全球唯一的呢？

  作者回复: 类似UUID，不用绝对唯一

  2019-08-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/20/9a/3b1c65fd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  八百

  😂老师为啥cidr的中文名叫那个啊，不理解，感觉记不住

  作者回复: 打破了分类。

  2019-08-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/95/68/41546e8a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Leo

  打卡

  2019-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/e4/e6/5795b1aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨晓维

  A类地址不是没有讲吗？

  作者回复: 后面基本都是cidr了。但是分类又不能不讲

  2019-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/48/64/e0b94df2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  草莓&#47;mg

  ifconfig ip addr的区别还是没讲吧

  作者回复: 没有讲，留成课后练习题了，在后面答疑的部分有答案 你知道 net-tools 和 iproute2 的区别吗

  2019-07-19

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIicibkDiceIZd7R3y57L7WIVFpVWU6ebp40G3ZNLFtlUWb7d4O5BpyxKQSd81FXo4rQFbQoUWu35jJg/132)

  zhuxuxu

  https://blog.csdn.net/m0_37323771/article/details/81228854

  作者回复: 额，盗版满天飞

  2019-07-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/f6/d274a39c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ChengQian

  老师,请问这句话怎么理解:“CIDR 可以用来判断是不是本地人”?是不是说同一个子网下的即为本地?

  作者回复: 是的

  2019-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/f6/d274a39c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ChengQian

  老师,请问这里说的四层、二层是基于OSI标准的七层模型,还是TCP/IP的四层模型?

  作者回复: 都一样。TCP/IP的四层我们也不会说一二三四层，而是会说一二三四七层

  2019-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/83/75/75bad843.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  第二姑娘

  每一章都是宝藏，第一次读云里雾里，后面了解了ip再回来看就明白了很多，来回看了三四遍，第二章的串讲也很精彩，从linux课程过来的，那个课程还只看了七八章，哈哈哈哈(尴尬)

  作者回复: 越看越精彩

  2019-06-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/54/85/081804f7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  逍遥

  CIDR可以用来判断是不是本地人？这句话该怎么理解呢？

  作者回复: 根据/来计算是不是相同的网络号，相同则是本地

  2019-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/c9/f9/39492855.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿阳

  以前上学时和工作中，使劲啃了多本经典的计算机网络教材，总感觉理解的不深。看到这一节，我真的是拍案叫绝，用形象的比喻手法，加上一个看似简单的ifconfig命令，将ip和mac知识巧妙的串起来。理论和实际联系的真是好。值了值了！

  作者回复: 谢谢夸奖

  2019-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b4/94/2796de72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的人

  可以先看完这门课，再去读下TCP/IP卷一和卷二的书，这2本书比较厚，而且有难度。不是入门级读物。作者讲的真好，比看书好理解多了，赞👍。ps  课程的定价真的物超所值

  作者回复: 谢谢

  2019-05-29

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eo2SjCeylLv0P3Glle5277kA4b8cAuxr1NrC0njPKEqzSpB8IEicHB29GicFFwG1qiaxs4hxRiaBmoibVw/132)

  阳仔

  怎么抓包看

  作者回复: tcpdump

  2019-05-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/39/95/a72ef023.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  木子

  老师，那个CIDR的计算能详细说明一下吗？什么是子网掩码？什么是广播地址以及相关的计算方式？看的懵懵懂懂的🙁

  作者回复: 可以多读几遍哈

  2019-05-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/f7/e5/ec543f3b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张宗伟

  大学学过 计算机网络，期末考试时，感觉复习的也不错，现在快毕业了也忘了。有一个痛点就是知识不能灵活运用！

  作者回复: 多用就能灵活运用了

  2019-05-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/7a/0a/0ce5c232.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吕

  这个问题我明白了，二进制的算法，我当时想的太死板了

  作者回复: 是的

  2019-05-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/7a/0a/0ce5c232.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吕

  子网掩码和 IP 地址按位计算 AND,这个AND计算是指的什么计算？按位与 & 还是按位或 | 还是按位异或，你说的1与任何and都是1，这是一种什么位运算呢？

  作者回复: &就是AND

  2019-05-14

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0f/d7/31d07471.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  牛年榴莲

  从来没用过ip addr。我感觉我是个渣渣。

  作者回复: 不是的，不是的，很容易上手的

  2019-05-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c0/6c/29be1864.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  随心而至

  所以主机号位数组最后一个为1的是出口IP地址，主机号位数组都为1的是广播地址

  作者回复: 是的

  2019-04-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/bc/59/c0725015.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  彭旭锐

  一个子网需要分配至少一下IP地址： 1、 一个网路目标地址（子网中最低IP地址） 2、一个网关地址（子网中次高IP地址） 3、一个广播地址（子网中最高IP地址） 4、n个主机地址 5、预留（或浪费）的IP地址（子网中次低到次高间的IP地址分配给主机地址之外的部分） 对于，16.158.165.91/22 这个 CIDR，16.158.164.0就是目标地址啦，所以第一个（主机）地址就是16.158.164.1

  2019-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/24/d2/a5e272ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夜空咏叹调

  ip地址分公有和私有，公有对外，私有对内。公有是唯一的，但私有可能重复，这和文件夹取名类似。然后子网掩码这块还是有问题，后面需要再看看。

  2019-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/54/b2/5ea0b709.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Danpier

  老师，IP分类图里的E类IP是不是画错了？ E类范围为： 240.0.0.1-255.255.255.254 应该是1111××××.××××××××.××××××××.×××××××× 如果前缀是11110，那E类的范围不就成了：240.0.0.1-247.255.255.255，这样不对吧？ 还有网络号也包括前缀吧？图里小括号那样标位数，容易让人理解错。

  作者回复: tcp ip 详解里面写的就是11110呀

  2019-04-18

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/ec/1b/650e3dbe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zws

  我自己是计算机网络的专科生，学校学的也差不多忘光了，看了前三章，感觉非常不错。 

  作者回复: 加油看完

  2019-04-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fa/25/9f7bbf63.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  秋风

  讲的很清晰啊，我觉得我可以尝试实现一个简单的dhcp服务器试试

  作者回复: 可以的

  2019-04-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/88/fe/c18a85fe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  随风

  小白请教，ip地址范围怎么确定的？都是硬性规定的吗？比如B类是128.0.0.0--191.255.255.255, 私有ip范围是176.16.0.0--172.31.255.255, 这些是怎么计算出来的？

  作者回复: 规定的

  2019-04-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a0/57/3a729755.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灯盖

  写的好，这篇是第二遍看了

  作者回复: 赞，加油

  2019-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/60/c9/8c9b0636.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不识南风

  讲的很好，很容易理解！

  2019-04-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/5a/8a/a0ed5f5c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ch

  老师，您好，我在linux服务器上输入ip addr看到的qdisc的值是mq，这个是什么意思？

  作者回复: 你啥系统？

  2019-04-06

  **2

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/nFjwzhDmSTHovqZWpGGicmUiahMvfagbx9gFMuaBTaTNicZ7Rv6BwUXia2Du2biaOLCg8lmgIblM3iaLUibG0yF9yeE5Q/132)

  六日

  网卡驱动去丢弃它

  2019-04-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a8/2c/805b5ef6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  松

  网络号和主机号有什么用？一个ip不就代表一台主机的地址吗？

  作者回复: 是的，但是如何判断两个主机是不是在同一个局域网里面呢

  2019-04-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/21/41/97ce3356.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  苏小小

  子网掩码，网络位都是1，主机位都是0；广播地址，网络位不变，主机位都是；

  2019-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/89/52/2b9ff286.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  

  所以谁能告诉我ip addr 和ifconfig的区别😂老师说了ip addr的功能，并没有比较二者区别啊

  作者回复: 这方面资料很多的

  2019-04-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/74/88/f36f2612.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  啊呀呦

    trim

  2019-04-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fb/b2/b1c23254.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  唐宋元明清

  建议把计算机网络原理读一遍再来看这个会好多

  作者回复: 可是计算机原理讲的也是这些呀

  2019-04-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/10/5d/9fc4ae25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  K

  题目是ifconfig讲的是ip addr这是不是有点问题呀

  作者回复: 标题党

  2019-04-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/22/0d/a0d84610.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小伟

  我是一个java程序员，因为半路出家，一直对网络知识这块不是很懂，只能说一脸懵逼进来，一脸懵逼出去，这个是不是不太适合零网络基础的人学习？

  作者回复: 上手操作一下就会好

  2019-03-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/22/b5/d841e5fc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  翁思塞代

  子网划分可以单独出一节，毕竟是科班出身必须会的基本功

  作者回复: 是的

  2019-03-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/e9/29/629d9bb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天王

  ip地址有CIDR，将32位的ip地址一分为二，前面是网络号，后面是主机号。最后8位是255，是广播地址。子网掩码255.255.255.0。分公有地址和私有地址，私有地址可以局域网分配，公有地址需要到特定地申请。有定位功能，mac地址像身份证，在同一个子网内有定位功能。

  2019-03-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/1c/6f/3ea2a599.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  嘉木

  老师您好，请问我的主机上 qdisc 是mq，这个表示什么呢？

  作者回复: 应该是硬件网卡有多个传输队列，内核可以识别

  2019-02-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/b4/f6/735673f7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  W.jyao

  就是说必须ip地址和mac都存在，才能收到包，缺一不可？有ip地址只是定位哪一个子网？然后发一个类似广播包，然后对应mac地址相同的就会收到包解包，其余的机器就会忽略？我是想问，如果同一个子网，为啥只知道ip就不行，ip不能定位出哪个机器吗？

  作者回复: 是的，都得有。因为mac不符合就丢掉了

  2019-02-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c4/f9/fa5ee4fb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  星际之门

  内容很好，对网络的认知更深了，还有一点不足，定价真的太低。

  2019-02-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/5b/72/4f8a4297.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿May的海绵宝宝

  文章举例中，把IP比作定位用的门牌号，网卡的mac地址比作人的身份证号， 1.这个IP和网卡mac地址是1对多的关系？ 2.这个IP指的是路由的IP,还是每个主机上网卡的IP?

  作者回复: 一个网卡上是可以有多个IP的。但是主要还是理解定位功能。IP都是网卡上，就算是在路由器上，路由器上也有网卡

  2019-01-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  这一节读完了，老师写的真好！ 看来还是CIDR 更灵活啊！ 您写的文章，至少还要看两遍！

  2019-01-15

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTK8UYbedASKg0kicuQFQpnRuq71eKhw2Lwycaaxhnora0ibMucGNwQh4WFxJFhkWSsmjnPvTBzajWwA/132)

  克里斯

  每类中的私有地址中的范围都是规定好的吗

  作者回复: 是的

  2019-01-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/59/1d/c89abcd8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  四喜

  Ipv6有2^128数，这个是不可想象的。绝对够了

  2018-12-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/21/42/a8dee95f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  like

  私有IP范围为啥是那样划分的，也是规定的？

  2018-12-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0d/4d/f692ba8b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wmmmeng

  请问：我想通过tcp协议给不在同一网段的另外一台电脑发送数据，这个流程是怎么样的？

  2018-12-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/1c/4b/7ccd2499.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  F.

  很难懂…

  2018-12-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/eb/e4/dabff318.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  邵峰

  老师，有个疑问一直折磨我，有可能两台电脑的Mac地址一样吗？就算一样不在同一个局域网也没关系是吗？可是同一局域网的两名同学恰好买了两台Mac地址一样的电脑，难道只有一台可以用吗？

  2018-12-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b9/3b/7224f3b8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  janey

  感觉好像听懂了，希望留个细挖的问题，验证一下

  2018-12-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/aa/72/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  孙文正

  16是A类网络，165的前6位不是网络号。 这里想举的例子是C类网络的IP结构。 希望说明修改一下

  2018-12-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/83/61/b32787f6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  rjb

  传统地址分类,无类型域间选路即CIDR,公有私有IP到底什么关系？现实生活中现在是怎么用的。感觉好迷糊。

  2018-12-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所获：1.IP地址 是门牌号，可以定位，Mac地址为身份证 2.IP地址分公有IP地址和私有IP地址，公有IP地址对应以太网，私有IP地址对应局域网 3.子网掩码 AND IP地址 的结果为 网络号 4.ip 最后一位 等于1 如 192.168.0.1为出口，ip最后一位 等于255，如192.168.0.255为广播 ，通知所有局域网的设备。 回答老师第二个的问题：IP的分配 分 以太网 和局域网，以太网 是 国家 或者组织 统一分配，局域网 内的IP可以自动获取 或者 手动设置。

  2018-12-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/9a/062f83c8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑羽

  MAC地址的🌰举得太生动了👍

  2018-12-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/69/18/c3bd616c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  coder吕

  老师讲课思路非常清晰，刷新了自己好多知识盲区.计算机比较抽象，能结合现实生活的例子来讲课，太厉害了吧！！！点赞

  2018-12-06

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  涟漪852

  好复杂～(￣▽￣～)~

  2018-12-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/9a/cc/d4d52609.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  run

  mtu的概念好像说的不太准，mtu应该是mac层能承载的最大载荷吧，应该是不包括mac头长度的，即应该是ip包的最大长度

  2018-12-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/11/3e/925aa996.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  HelloBug

  老师，你好~通过route命令查看我自己CentOS7系统（虚拟机上）的路由表，Destination那一列显示地址是0.0.0.0， 但是在TCP-IP详解卷1的3.6特殊情况的IP地址里说全为0的IP地址只能做远端，不可能作为目的端。这种情况该怎么理解呢？

  作者回复: 这个是默认路由

  2018-12-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0e/e9/98b6ea61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员大天地

  可以，讲得还是很清楚的啊。

  2018-11-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c8/f0/a37daf86.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  👹

  为什么192.168.0.2/24 到 192.168.1.2/24算两个子网

  作者回复: 计算cidr

  2018-11-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/90/43/e9345631.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  学无止境

  总算是将ip地址和MAC地址区分开了，ip地址用来定位，就像要找一个人你在哪个城市呀，哪个小区呀。MAC地址是确确实实唯一的就像身份证，找到小区后(同一个网络中)，就可以问身份证为这个的人在哪(即可以通过MAC地址找到特定目标了)!一个人的位置可能会变就像一个局域网中某个设备分配的ip可能是会变的，但一个人的身份证不会变，就像MAC地址一样！很清晰！

  2018-11-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e2/63/039f3896.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿吉

  私有地址出网管会转成共有地址最好简单讲一下，这个困惑直到网管这张才消除

  2018-11-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/37/1b/82310e20.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  拿笔小星

  太棒了，终于知道子网掩码是干嘛的了！还有原来abcd已经不常用了！看子网掩码和IP地址AND计算，对照自己阿里云服务器网卡，计算，一开始总是算的不对，看到后面的例子，终于知道怎么算了！真的很棒，但愿学完后能对网络这方便的弱项不要太弱！

  2018-11-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/36/45/23cf8dcd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没想好。。。

  这节讲了IP的一些基本概念，IP地址分为私有的和共有的。在一个局域网内IP不能重复（比如公司里，学校，家里）。IP地址是一个网卡在网络世界的通讯地址，类似于门牌号。MAC地址类似于身份证，在出厂时跟随网卡确定，是唯一的。

  2018-11-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/4b/21/d17f295c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  棋寂

  赞~~

  2018-10-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ce/c6/958212b5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sugar

  想问一下 是否可以理解为 net-tools是我们在cli中常用的ifconfig所依赖的内核组件，而iproute2则是ip addr依赖的？

  2018-10-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/4b/61/762ec306.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我们的时光__

  老师，这个广播地址怎么理解呢，有哪些应用场景？

  作者回复: 往后看，后面有讲

  2018-10-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/49/96/59f6a194.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  、姜尸可乐

  @秋生 1. 子网掩码与IP地址进行位与运算，得处网络地址 2. 网络地址 | (~子网掩码)，得出广播地址 |：位或运算； ~：按位取反 子网掩码最后两位是00 取反就是11 .所以,最后是11  百度的广播地址计算

  2018-10-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d5/85/9da08fd3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  aaaVege💫

  网络基础不好的人听着很一般。

  2018-10-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d4/19/b108dc28.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alan

  基础知识，讲得真好，当初学网络没有碰到这么好的讲解！没听懂的人可以再多听一遍，基础打好是内力。

  2018-10-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/bb/85/191eea69.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  搬铁少年ai

  dhcp真的很复杂

  2018-10-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ce/67/0948d3b0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Imperfect

  老师讲的很棒，建议那些遇到问题看不懂的小白可以去百度，由点及面，这样学的东西会更多。

  2018-09-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ae/e0/3e636955.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李博越

  CIDR的例子中，第一个地址是16.158.<101001><00>.1，即 16.158.164.1，那主机号的范围是16.158.164.1~16.158.164.256吗？

  作者回复: 不是的，也要按子网掩码来算

  2018-09-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/10/b7974690.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BD

  Dubbo的广播注册用的就是d类ip地址吧

  2018-09-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/10/b7974690.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BD

  A类地址网络号最大126为什么子网掩码可以是255

  2018-09-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/cb/1d/fb630819.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_f47e84

  写的真好。 以后，学习上遇到困难了，就来这里学习老师写的网络知识。看完之后觉得很开心。

  2018-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c2/14/bfd5e94e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  简

  这么低的价钱能买这么好的资料，我觉得中国的科技有希望了

  作者回复: 这个拔的太高了，哈哈

  2018-09-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/94/db/4e658ce8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  继业(Adrian)

  由于没法回复自己的留言，这里新写一条。 上面我问到如何确定ifconfig显示出来的多个网卡中，哪个是真正被用来发网络包的。目前自己看来只能是抓包的方法，看包里的信息。但是如果我们写程序的配置的话，怎么来抓包呢。下面一段go的代码用来在程序中抓包。获取自己的本机ip。 func localIP() (string,error) {   conn, err := net.Dial("udp", "8.8.8.8:80")   if err != nil {      log.Error("Get local ip error.",err)      return "",err   }   defer conn.Close()   localAddr := conn.LocalAddr().(*net.UDPAddr)   localIP := localAddr.IP.String()   log.Info("Local IP : ",localIP)   return localIP,nil }

  2018-09-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/94/db/4e658ce8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  继业(Adrian)

  ifconfig之后会列出所有本机的ip地址和对应的mac信息，那么我怎么能知道，我正常访问网络用的是哪个网卡呢，查了资料说，一般把eth0当作。

  作者回复: 可以抓包看一下

  2018-09-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/49/2a/83fe5388.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  C1zel

  老师最近抓 arp 包发现一个问题不太明白。 192.168.1.10() 与 192.168.1.11 使用 hub 连接， 使用 192.168.1.10 ping 192.168.1.11 (wireshark capture) 0.000000	96:7c:d1:5b:ba:39	Broadcast	ARP	42	Who has 192.168.1.11? Tell 192.168.1.10 0.000351	92:ca:c9:e3:42:7f	96:7c:d1:5b:ba:39	ARP	42	192.168.1.11 is at 92:ca:c9:e3:42:7f 这个地方是广播去找 192.168.1.11 的 mac 地址。 一段时间之后，就有下面的这种包。 这样的 arp 包怎么理解？ 5.003390	92:ca:c9:e3:42:7f	96:7c:d1:5b:ba:39	ARP	42	Who has 192.168.1.10? Tell 192.168.1.11 5.003584	96:7c:d1:5b:ba:39	92:ca:c9:e3:42:7f	ARP	42	192.168.1.10 is at 96:7c:d1:5b:ba:39

  作者回复: 这是回复的时候，也是需要arp一下

  2018-09-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/a2/1c/c8fc9374.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mr.周

  所以现在cidr已经取代了原来网络地址划分的做法了是吗？

  作者回复: 是的

  2018-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ff/ce/73ee54bf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Linux云计算网络

  问题很有意思，但可惜不知道，我还是查一查，看看留言吧

  2018-09-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5e/13/c822e53c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鹏鹏鹏先森

  老师，那个根据cidr求网络的第一个地址，子网掩码和广播地址没看懂啊，能补充一下么

  2018-09-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/44/3f/1cc74a36.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  姬建政

  请问一张图片里面私有IP地址画了一个红色的方块，里面的c类私有IP地址是不是写错了，那个192.168.0.0到192.168.255.255

  2018-08-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/71/ea/4ab8d452.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  4fun

  讲得很透彻，获益匪浅。

  2018-08-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5c/b5/0737c1f2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kuzan

  老师这个课程定价太便宜了，精品，通俗易懂！

  2018-08-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/47/5e/59cd7b94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Terry

  非常生动风趣

  2018-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/07/8a369805.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  豆子

  L1启动 这个是什么意思？

  2018-08-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/51/3a/95168093.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一只智障

  豁然开朗

  2018-08-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/0e/c77ad9b1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  eason2017

  老师的文章好，仔细读文章，不断的咀嚼每一段话！老师提的问题也很好，很有智慧，学习很多，更新了很多知识库，👍

  2018-07-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9e/ee/68f6493d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Chuck

  刘老师真是用心了。

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/c0/bc/c49e1eaa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  静静张

  先把老师讲的听一遍，再去买本书系统的看，有需要的话再来听一遍。 一口吃不成胖子，并且现在是为工作做储备，看工作需要掌握到什么程度。

  2018-07-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5b/66/ad35bc68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  党

  讲解的非常好 关键点都讲到了

  2018-07-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/d7/cf/3c30f1d7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  比如刘恩

  本章作者依然讲的通俗易懂，看见很多朋友觉得不够深入，我觉得点到为止得刚刚好。通俗易懂和鞭辟入里往往如鱼与熊掌。前者需要的是历尽千帆大道化简，所以我认为想要以此作入门之路的人最好辅以课本。毕竟试图以小无相功贯通少林七十二绝技的最后难免要走火入魔。 另外有点拙见 我认为 ip协议具备定位寻址功能的原因是路由表以及网络层的各种协议。 mac在网桥内（链路层？）同样具备一定的广播寻址功能。 我一直觉得，很多时候我们选用某种协议或者规则并不是因为它的绝对合理性，而是结合某个特定时期的发展程度和可持续成本而做出得选择。

  作者回复: 后期有点过于深入了，好像

  2018-07-03

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eoYBX95GZxEdKz9LQJVwohUiaRxNge5WpHRbeOC2tGc2rsdpfYKCTKdQicBn8MvSrlZTX7HY2jS3YFA/132)

  青冈

  老师你好，请问下当不同的客户端发送请求，计算机根据什么标识来确认这个网络包应该给safair浏览器而不是给谷歌浏览器呢？大家都监听的同一个端口号？

  作者回复: 不同的端口号，随机的

  2018-06-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ba/f4/0dda3069.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小白

  老师，后面的私有ip 地址和公有ip 地址没有看懂

  2018-06-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8a/ce/f5e42fbd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Kun3375

  太棒了，这种枯燥的知识居然能这么清晰简单的讲明白，还挺有趣…真是这价钱便宜我们可😄

  2018-06-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b4/9a/17309b19.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Moorez

  讲的挺好的，生动形象

  2018-06-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/1d/39/1be8b56c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  snzhaoch

  C类地址主机号只有8位，因此最大主机数254个，但是私有ip范围为什么是192.168.0.0-192.168.255.255呢，这不是65535个么？

  2018-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/a5/cd/3aff5d57.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alery

  “而 B 类地址能包含的最大主机数量又太多了。6 万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。” 刘老师，上面这段话有点不理解，同一个网络的ip不一定要全部分给同一家企业，而且企业里面一般都是局域网，剩余的那些ip不就可以分给其他企业的吗？不知道我这种理解对不对。还望老师指点一下？？

  作者回复: 这是私网ip，不同局域网可以同样的ip段，但是都浪费

  2018-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/a5/cd/3aff5d57.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alery

  “qdisc 全称是queueing discipline，中文叫排队规则。内核如果需要通过某个网络接口发送数据包，它都需要按照为这个接口配置的 qdisc（排队规则）把数据包加入队列。” 刘老师，这里提到的网络接口是指网卡吗？

  2018-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4f/ec/039fb885.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨(Yang)

  最后的波段和TOF讲得有点仓促了，毕业十几年了，看了两遍才回忆起来

  2018-06-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/9e/474e43d9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Bernard

  A B C 类和 CIDR 是如何做到共存的呢

  2018-06-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/9e/474e43d9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Bernard

  A B C 分类 和 CIDR 是如何做到共存的？

  作者回复: cidr就是无类，也就是打破了这些类

  2018-06-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/6f/6051e0f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Summer___J

  听了这节突然觉得大学白上了……虽然专业是网络工程，但学校里没有把网络知识讲得那么深。第一次知道ifconfig的回显里面有那么多学问，特别是bond、TOS，这听起来都是新鲜玩意儿。还记得刚工作时虚拟机咋弄都出不来IP地址，急坏了，后来同事过来执行了dhclient，噔～有了！当时觉得好神奇～

  2018-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/07/f5/33d5cf22.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  痴痴

  一个公有ip最多能映射多少个私有ip？？

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a1/af/db1c6a2c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  我❤️北极星

  这是我看过的把网络讲的最通俗易懂的课程了，感谢！！

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/49/ba/23c9246a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mαnajay

  网络号和主机号 记住了！ Mac地址虽然是唯一，但是不能进行长距离的网络通信，只可以局域网定位。 

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/01/b8709bb9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大师兄

  感谢回答，我上午问了2个问题我没有描述清楚，比如“第一个地址是 16.158.<101001><00>.1，即 16.158.164.1”这句话，其实我2个问题就是不清楚这个ip地址的数字是多少进制，我查了查这种常见的地址格式是十进制的，现在知道怎么转换的格式了

  作者回复: 赞

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/60/c7/a147b71b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Fisher

  对于 Mac 地址相当于身份证的类比没什么问题，但是说为什么不用有了 Mac 还用 IP地址是因为 IP 有定位功能，而 Mac 没有这个解释有点不明白 IP 地址的定位功能也是通过 DNS 来实现的，也不是说就直接有的，原理也是通过记录每一个 IP 分配给了谁从而可以定位的，那么这个问题的本质其实是为什么当时 DNS 不直接记录 MAC 地址，而要记录 IP 地址，希望老师能够解惑

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/01/b8709bb9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大师兄

  第一个地址是 16.158.<101001><00>.1，即 16.158.164.1 这个算法能不能说一下

  作者回复: 就是除了网络地址，其他都为零，让后最后加一

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/01/b8709bb9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大师兄

  中间的 165，变为二进制为‭10100101 这是多少进制转的？

  作者回复: 转成二进制

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/47/ec/85b1a9cb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Aries Amen

  耐着性子读了三遍，总算看懂了一点

  2018-05-29

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Gkedlmv1ibTmUcALf69BCDhg5j0CGtqxaHHtpnbWc6yF8Wld7ILmgoScJtk4QiblfqoUJl9jyNhjgib9aEBGlIh8Q/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  执__生

  那个，能不能顺便安利给小白一些linux的资料，要不然很快就跟不上了......

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/59/7f/1c505317.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  killua

  为什么说ip有定位功能呢？

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/3c/b7091379.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  还能再肝的biubiubiu

  要是能更新的再快点就好啦。。

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/50/12/e81db565.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  执笔书生

  ip addr 让我见识了，天天都敲，从来没有详细理解过！感谢老师

  2018-05-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/95/96/0020bd67.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夏洛克的救赎

  公网IP地址不是唯一的吗？  通过IP只能定位到组网？结合mac地址才能确定一台机器？没理解透。 这对我们编程有何帮助？ 自己把命令看了下，一下子去研究原理也不合适吧

  2018-05-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/74/1d9980a7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  saxon

  一只在找cidr 根据檐马快速计算方法

  2018-05-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/09/de/55cc6c2f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大树

  头一次知道pfifo_fast还有bond机制，类似于kinesis stream 的shard, 可以对数据进行区别处理。MAC是ID,比喻很赞。

  2018-05-27

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/ajNVdqHZLLDcRhwfibjWzUHSPpicvWYRayXwomAuxwmXSiaO9AO2v7uz1ymcSLQ2xnEvsBZ11bJib0lZOXk1Ediaibdg/132)

  hshjsjs

  “MAC 地址更像是身份证，是一个唯一的标识。” 老师你把它说成了 IP。

  作者回复: 可能是口误了，自己录还是很紧张

  2018-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4a/ce/89ae650f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sonny

  mtu 1500如果是mac头加报文的长度，那么如果http请求方法是post的话，报文body中含有内容，那么这个内容不是没有限制的吗？那岂不是到了二层的mac这块会和mtu 1500这个限制冲突吗？

  2018-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/66/34/0508d9e4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  u

  cidr是根据目标ip地址和本地子网掩码来判断是不是本地人？

  作者回复: 是的

  2018-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/eb/a6/7dfa974a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  阿泰

  讲的太好了，通俗易懂，以前学的忘了的又重新回来了，温故而知新

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/74/34409399.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Chain

  课程几十块钱，感觉我捡便宜了

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ea/d7/9506fe35.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  pllsxyc

  写的太棒了，不夸张的说，我以前就这么理解IP地址的，看到老师和我一样的想法，好高兴啊

  作者回复: 赞

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c8/9b/f014e9e9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Yang

  昨天刚好自己配置了虚拟机上centos7的ip，用到了ifconfig和ip addr，今天又看到这篇文章，从实践到理论贯通的感觉真棒👍

  作者回复: 谢谢

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/50/82/eb5713bb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  牛牛01

  不错，不错，老师后期有https相关吗？那块不太懂。

  作者回复: 有的

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ba/f5/359015e4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  isunsw

  是谁深夜扣我门，是谁良辰知我心。这玩意竟然能告诉我从我的心到你的心，绕一绕，拐个弯就到了。你信吗？ 

  2018-05-25

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eqy7GudyFZicjyYw9LuPAK3IUH6zzzUpJTpAjVkkbpmNcZ5GkvW8ibPsqVsgpP8iajXtxUvVTIjxibkAQ/132)

  xpxdx

  老师，请教一个问题：MAC物理地址和数据链路层的MAC子层是一样的吗？是不是对应关系的。（印象中数据链路层好像有MAC层和LLC层）

  作者回复: 是的，学术中分的比较开，应用中一般就说二层

  2018-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/d8/c90d4702.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  jameskaron

  真的讲解得十分清晰

  作者回复: 谢谢

  2018-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/f1/4c0b8411.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  回不去的fly

  前面基本不关注，只想看传输层

  作者回复: 别啊，其实前面很重要的

  2018-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/40/23/5fda1246.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  霖霖先

  还是没有基础，看的懵懵懂懂的

  作者回复: 看来还是不够通俗

  2018-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5b/52/fea5ec99.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  俊飞

  坐等讲解云计算里面涉及到的网络

  作者回复: 好的，里面的概念就比较多了

  2018-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/76/a7/374e86a7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  欢乐的小马驹

  接着我上个问题。在百度的搜索框输入ip，就能查到本机的ip公网地址(国内的，联通)。在谷歌浏览器输入ip能得到另外一个本机的ip公网地址（可能是我设置vpn 地址是美国的）。还有ip addr和ifconfig为什么看不到外网地址？

  作者回复: 那是在出口网关nat了，你ip addr的地址是私网地址吧

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/3f/97/8d7a6460.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  卡卡

  当年发明ip地址时，为什么要做分类，分成abc等类，这种分类运用场景在哪？现在看来很多余！

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d9/db/66d5b3f4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Leo

  ip用于定位，mac是身份证，定好位置再找到正确的人

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/76/a7/374e86a7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  欢乐的小马驹

  老师好，我在百度搜索ip和在谷歌搜索ip为什么不一样？ip addr 和 ifconfig打印出来的ip都没有前边那两个外网ip。后来用curl ifconfig.me搜出来的ip跟百度搜出来的ip一样。我已经晕了～怎么解释？

  作者回复: 百度搜索ip的意思是？

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/56/20/669aef03.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小酒

  期待实战

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/90/167a13fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  徐良红

  深入浅出，通俗易懂，非常感谢

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/43/bf/9a982bc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  子悠

  以前查看ip真的只是看一下ip，原来里面东西这么复杂，现在明白多了。看到下面的讲解，还往上翻了好几遍看里面的参数。

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/42/9a/183455ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  先天专治不服

  假设我在一个内网中，对外发送了一个数据包，如果对方回应我了，首先应该是可以找到我所使用的公网的出口地址，然后通过mac在局域网内找到我这个设备。老师，请问我这样的理解是不是正确的？还有，有什么工具可以做到？

  2018-05-23

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKWHqcx2zIN5Q11ecfJiaubwHBrN04wyibic7xCyEfiaZqgQ3Mx9cxe5xvu6ryRllO0B8u6NBEWKeGW6g/132)

  f_066

  ip addr查看时最后一句valid_lft forever preferred_lft forever这个是什么意思？

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/eb/f6/ec7971f4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  固态U盘

  机械专业的小白路过。老师讲得通俗易懂，自己以前要计算子CIDR相关的问题时，有些蒙圈。看了老师的讲解，终于解惑了。 另外，一个ifconfig或ip命令下面还隐藏了这么多的知识点，qdisc不是很清楚具体做什么的，老师后面会有讲解吗？用它可以做什么吗？ 期待老师后面更精彩的内容！

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/49/2a/83fe5388.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  C1zel

  mac是身份证，ip用于定位的解释很棒 分配ip，一般有静态手动分配ip，还有动态的dhcp服务分配ip

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/44/58/a5436e61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Define

  看完这章感觉有点说的不太明白 IP地址在终端上是存在哪里的 使用的时候都有一个程序来计算 对应的掩码网络号等吗 

  作者回复: 会通过cidr来判断是不是本局域网的

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/58/ca/05045b95.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  少

  很准时👍👍

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/8f/466f880d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没心没肺

  可以再详细说说VLSM

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/a6/723854ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  姜戈

  二层还有arp  rarp协议

  作者回复: 后面会讲

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/55/f5/8d1588bd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  xfun

  有没有这样的可能呢，比如，IP直接定位到了目标，或者不需要内网IP了

  2018-05-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/ce/a8c8b5e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jason

  讲的真好。

  2018-05-23

  **

  **
