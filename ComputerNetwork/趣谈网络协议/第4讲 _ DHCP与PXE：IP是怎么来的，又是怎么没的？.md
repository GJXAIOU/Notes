# 第4讲 \| DHCP与PXE：IP是怎么来的，又是怎么没的？

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/4e/93/4e4b432837b091c064bc3ef23a4b0093.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/8f/35/8fcc98e770f39dc43af82f717ffd4f35.mp3" type="audio/mpeg"></audio>

上一节，我们讲了IP的一些基本概念。如果需要和其他机器通讯，我们就需要一个通讯地址，我们需要给网卡配置这么一个地址。

## 如何配置IP地址？

那如何配置呢？如果有相关的知识和积累，你可以用命令行自己配置一个地址。可以使用ifconfig，也可以使用ip addr。设置好了以后，用这两个命令，将网卡up一下，就可以开始工作了。

**使用net-tools：**

```
$ sudo ifconfig eth1 10.0.0.1/24
$ sudo ifconfig eth1 up
```

**使用iproute2：**

```
$ sudo ip addr add 10.0.0.1/24 dev eth1
$ sudo ip link set up eth1
```

你可能会问了，自己配置这个自由度太大了吧，我是不是配置什么都可以？如果配置一个和谁都不搭边的地址呢？例如，旁边的机器都是192.168.1.x，我非得配置一个16.158.23.6，会出现什么现象呢？

不会出现任何现象，就是包发不出去呗。为什么发不出去呢？我来举例说明。

192\.168.1.6就在你这台机器的旁边，甚至是在同一个交换机上，而你把机器的地址设为了16.158.23.6。在这台机器上，你企图去ping192.168.1.6，你觉得只要将包发出去，同一个交换机的另一台机器马上就能收到，对不对？

可是Linux系统不是这样的，它没你想的那么智能。你用肉眼看到那台机器就在旁边，它则需要根据自己的逻辑进行处理。

<!-- [[[read_end]]] -->

还记得我们在第二节说过的原则吗？**只要是在网络上跑的包，都是完整的，可以有下层没上层，绝对不可能有上层没下层。**

所以，你看着它有自己的源IP地址16.158.23.6，也有目标IP地址192.168.1.6，但是包发不出去，这是因为MAC层还没填。

自己的MAC地址自己知道，这个容易。但是目标MAC填什么呢？是不是填192.168.1.6这台机器的MAC地址呢？

当然不是。Linux首先会判断，要去的这个地址和我是一个网段的吗，或者和我的一个网卡是同一网段的吗？只有是一个网段的，它才会发送ARP请求，获取MAC地址。如果发现不是呢？

**Linux默认的逻辑是，如果这是一个跨网段的调用，它便不会直接将包发送到网络上，而是企图将包发送到网关。**

如果你配置了网关的话，Linux会获取网关的MAC地址，然后将包发出去。对于192.168.1.6这台机器来讲，虽然路过它家门的这个包，目标IP是它，但是无奈MAC地址不是它的，所以它的网卡是不会把包收进去的。

如果没有配置网关呢？那包压根就发不出去。

如果将网关配置为192.168.1.6呢？不可能，Linux不会让你配置成功的，因为网关要和当前的网络至少一个网卡是同一个网段的，怎么可能16.158.23.6的网关是192.168.1.6呢？

所以，当你需要手动配置一台机器的网络IP时，一定要好好问问你的网络管理员。如果在机房里面，要去网络管理员那里申请，让他给你分配一段正确的IP地址。当然，真正配置的时候，一定不是直接用命令配置的，而是放在一个配置文件里面。**不同系统的配置文件格式不同，但是无非就是CIDR、子网掩码、广播地址和网关地址**。

## 动态主机配置协议（DHCP）

原来配置IP有这么多门道儿啊。你可能会问了，配置了IP之后一般不能变的，配置一个服务端的机器还可以，但是如果是客户端的机器呢？我抱着一台笔记本电脑在公司里走来走去，或者白天来晚上走，每次使用都要配置IP地址，那可怎么办？还有人事、行政等非技术人员，如果公司所有的电脑都需要IT人员配置，肯定忙不过来啊。

因此，我们需要有一个自动配置的协议，也就是**动态主机配置协议（Dynamic Host Configuration Protocol）**，简称**DHCP**。

有了这个协议，网络管理员就轻松多了。他只需要配置一段共享的IP地址。每一台新接入的机器都通过DHCP协议，来这个共享的IP地址里申请，然后自动配置好就可以了。等人走了，或者用完了，还回去，这样其他的机器也能用。

所以说，**如果是数据中心里面的服务器，IP一旦配置好，基本不会变，这就相当于买房自己装修。DHCP的方式就相当于租房。你不用装修，都是帮你配置好的。你暂时用一下，用完退租就可以了。**

## 解析DHCP的工作方式

当一台机器新加入一个网络的时候，肯定一脸懵，啥情况都不知道，只知道自己的MAC地址。怎么办？先吼一句，我来啦，有人吗？这时候的沟通基本靠“吼”。这一步，我们称为**DHCP Discover。**

新来的机器使用IP地址0.0.0.0发送了一个广播包，目的IP地址为255.255.255.255。广播包封装了UDP，UDP封装了BOOTP。其实DHCP是BOOTP的增强版，但是如果你去抓包的话，很可能看到的名称还是BOOTP协议。

在这个广播包里面，新人大声喊：我是新来的（Boot request），我的MAC地址是这个，我还没有IP，谁能给租给我个IP地址！

格式就像这样：

![](<https://static001.geekbang.org/resource/image/90/81/90b4d41ee38e891031705d987d5d8481.jpg?wh=1405*1141>)

如果一个网络管理员在网络里面配置了**DHCP Server**的话，他就相当于这些IP的管理员。他立刻能知道来了一个“新人”。这个时候，我们可以体会MAC地址唯一的重要性了。当一台机器带着自己的MAC地址加入一个网络的时候，MAC是它唯一的身份，如果连这个都重复了，就没办法配置了。

只有MAC唯一，IP管理员才能知道这是一个新人，需要租给它一个IP地址，这个过程我们称为**DHCP Offer**。同时，DHCP Server为此客户保留为它提供的IP地址，从而不会为其他DHCP客户分配此IP地址。

DHCP Offer的格式就像这样，里面有给新人分配的地址。

![](<https://static001.geekbang.org/resource/image/a5/6b/a52c8c87b925b52059febe9dfcd6be6b.jpg?wh=1405*1141>)

DHCP Server仍然使用广播地址作为目的地址，因为，此时请求分配IP的新人还没有自己的IP。DHCP Server回复说，我分配了一个可用的IP给你，你看如何？除此之外，服务器还发送了子网掩码、网关和IP地址租用期等信息。

新来的机器很开心，它的“吼”得到了回复，并且有人愿意租给它一个IP地址了，这意味着它可以在网络上立足了。当然更令人开心的是，如果有多个DHCP Server，这台新机器会收到多个IP地址，简直受宠若惊。

它会选择其中一个DHCP Offer，一般是最先到达的那个，并且会向网络发送一个DHCP Request广播数据包，包中包含客户端的MAC地址、接受的租约中的IP地址、提供此租约的DHCP服务器地址等，并告诉所有DHCP Server它将接受哪一台服务器提供的IP地址，告诉其他DHCP服务器，谢谢你们的接纳，并请求撤销它们提供的IP地址，以便提供给下一个IP租用请求者。

![](<https://static001.geekbang.org/resource/image/cd/fa/cdbcaad24e1a4d24dd724e38f6f043fa.jpg?wh=1405*1141>)

此时，由于还没有得到DHCP Server的最后确认，客户端仍然使用0.0.0.0为源IP地址、255.255.255.255为目标地址进行广播。在BOOTP里面，接受某个DHCP Server的分配的IP。

当DHCP Server接收到客户机的DHCP request之后，会广播返回给客户机一个DHCP ACK消息包，表明已经接受客户机的选择，并将这一IP地址的合法租用信息和其他的配置信息都放入该广播包，发给客户机，欢迎它加入网络大家庭。

![](<https://static001.geekbang.org/resource/image/cc/a9/cca8b0baa4749bb359e453b1b482e1a9.jpg?wh=1405*1141>)

最终租约达成的时候，还是需要广播一下，让大家都知道。

## IP地址的收回和续租

既然是租房子，就是有租期的。租期到了，管理员就要将IP收回。

如果不用的话，收回就收回了。就像你租房子一样，如果还要续租的话，不能到了时间再续租，而是要提前一段时间给房东说。DHCP也是这样。

客户机会在租期过去50%的时候，直接向为其提供IP地址的DHCP Server发送DHCP request消息包。客户机接收到该服务器回应的DHCP ACK消息包，会根据包中所提供的新的租期以及其他已经更新的TCP/IP参数，更新自己的配置。这样，IP租用更新就完成了。

好了，一切看起来完美。DHCP协议大部分人都知道，但是其实里面隐藏着一个细节，很多人可能不会去注意。接下来，我就讲一个有意思的事情：网络管理员不仅能自动分配IP地址，还能帮你自动安装操作系统！

## 预启动执行环境（PXE）

普通的笔记本电脑，一般不会有这种需求。因为你拿到电脑时，就已经有操作系统了，即便你自己重装操作系统，也不是很麻烦的事情。但是，在数据中心里就不一样了。数据中心里面的管理员可能一下子就拿到几百台空的机器，一个个安装操作系统，会累死的。

所以管理员希望的不仅仅是自动分配IP地址，还要自动安装系统。装好系统之后自动分配IP地址，直接启动就能用了，这样当然最好了！

这事儿其实仔细一想，还是挺有难度的。安装操作系统，应该有个光盘吧。数据中心里不能用光盘吧，想了一个办法就是，可以将光盘里面要安装的操作系统放在一个服务器上，让客户端去下载。但是客户端放在哪里呢？它怎么知道去哪个服务器上下载呢？客户端总得安装在一个操作系统上呀，可是这个客户端本来就是用来安装操作系统的呀？

其实，这个过程和操作系统启动的过程有点儿像。首先，启动BIOS。这是一个特别小的小系统，只能干特别小的一件事情。其实就是读取硬盘的MBR启动扇区，将GRUB启动起来；然后将权力交给GRUB，GRUB加载内核、加载作为根文件系统的initramfs文件；然后将权力交给内核；最后内核启动，初始化整个操作系统。

那我们安装操作系统的过程，只能插在BIOS启动之后了。因为没安装系统之前，连启动扇区都没有。因而这个过程叫做**预启动执行环境（Pre-boot Execution Environment）**，简称**PXE。**

PXE协议分为客户端和服务器端，由于还没有操作系统，只能先把客户端放在BIOS里面。当计算机启动时，BIOS把PXE客户端调入内存里面，就可以连接到服务端做一些操作了。

首先，PXE客户端自己也需要有个IP地址。因为PXE的客户端启动起来，就可以发送一个DHCP的请求，让DHCP Server给它分配一个地址。PXE客户端有了自己的地址，那它怎么知道PXE服务器在哪里呢？对于其他的协议，都好办，要有人告诉他。例如，告诉浏览器要访问的IP地址，或者在配置中告诉它；例如，微服务之间的相互调用。

但是PXE客户端启动的时候，啥都没有。好在DHCP Server除了分配IP地址以外，还可以做一些其他的事情。这里有一个DHCP Server的一个样例配置：

```
ddns-update-style interim;
ignore client-updates;
allow booting;
allow bootp;
subnet 192.168.1.0 netmask 255.255.255.0
{
option routers 192.168.1.1;
option subnet-mask 255.255.255.0;
option time-offset -18000;
default-lease-time 21600;
max-lease-time 43200;
range dynamic-bootp 192.168.1.240 192.168.1.250;
filename "pxelinux.0";
next-server 192.168.1.180;
}
```

按照上面的原理，默认的DHCP Server是需要配置的，无非是我们配置IP的时候所需要的IP地址段、子网掩码、网关地址、租期等。如果想使用PXE，则需要配置next-server，指向PXE服务器的地址，另外要配置初始启动文件filename。

这样PXE客户端启动之后，发送DHCP请求之后，除了能得到一个IP地址，还可以知道PXE服务器在哪里，也可以知道如何从PXE服务器上下载某个文件，去初始化操作系统。

## 解析PXE的工作过程

接下来我们来详细看一下PXE的工作过程。

首先，启动PXE客户端。第一步是通过DHCP协议告诉DHCP Server，我刚来，一穷二白，啥都没有。DHCP Server便租给它一个IP地址，同时也给它PXE服务器的地址、启动文件pxelinux.0。

其次，PXE客户端知道要去PXE服务器下载这个文件后，就可以初始化机器。于是便开始下载，下载的时候使用的是TFTP协议。所以PXE服务器上，往往还需要有一个TFTP服务器。PXE客户端向TFTP服务器请求下载这个文件，TFTP服务器说好啊，于是就将这个文件传给它。

然后，PXE客户端收到这个文件后，就开始执行这个文件。这个文件会指示PXE客户端，向TFTP服务器请求计算机的配置信息pxelinux.cfg。TFTP服务器会给PXE客户端一个配置文件，里面会说内核在哪里、initramfs在哪里。PXE客户端会请求这些文件。

最后，启动Linux内核。一旦启动了操作系统，以后就啥都好办了。

![](<https://static001.geekbang.org/resource/image/bb/8e/bbc2b660bba0ad00b5d1179db158498e.jpg?wh=2083*3001>)

## 小结

好了，这一节就到这里了。我来总结一下今天的内容：

- DHCP协议主要是用来给客户租用IP地址，和房产中介很像，要商谈、签约、续租，广播还不能“抢单”；

- DHCP协议能给客户推荐“装修队”PXE，能够安装操作系统，这个在云计算领域大有用处。


<!-- -->

最后，学完了这一节，给你留两个思考题吧。

1. PXE协议可以用来安装操作系统，但是如果每次重启都安装操作系统，就会很麻烦。你知道如何使得第一次安装操作系统，后面就正常启动吗？
2. 现在上网很简单了，买个家用路由器，连上WIFI，给DHCP分配一个IP地址，就可以上网了。那你是否用过更原始的方法自己组过简单的网呢？说来听听。

<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(249)

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/19/965c845c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  袁沛

  20年前大学宿舍里绕了好多同轴电缆的10M以太网，上BBS用IP，玩星际争霸用IPX。那时候没有DHCP，每栋楼有个哥们负责分配IP。

  作者回复: 赞

  2018-05-25

  **9

  **304

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epmicgRSX8C0bABLic1MA6ssqOtqAu0vqkWgshsAfOhWXDnmmkfVicPBlJs6j6DRVuZx61Ldia7617YXg/132)

  xcodeproj

  新机器进来申请分配ip地址，老师你说在dhcp request的时候这台机器以0.0.0.0为源地址发出请求，那如果有多台机器同时申请呢？DHCP server如何分辨啊

  2018-05-25

  **9

  **145

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/c6/4a7b2517.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Will王志翔(大象)

  以问答写笔记： 1. 正确配置IP?  CIDR、子网掩码、广播地址和网关地址。  2. 在跨网段调用中，是如何获取目标IP的mac地址的？  从源IP网关获取所在网关mac, 然后又替换为目标IP所在网段网关的mac, 最后是目标IP的mac地址  3. 手动配置麻烦，怎么办？  DHCP！Dynamic Host Configuration Protocol！ DHCP, 让你配置IP，如同自动房产中介。  4. 如果新来的，房子是空的(没有操作系统)，怎么办？  PXE， Pre-boot Execution Environment. "装修队"PXE，帮你安装操作系统。

  作者回复: 赞

  2018-07-07

  **

  **126

- ![img](https://static001.geekbang.org/account/avatar/00/10/36/9e/8f031100.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ERIC

  刘老师你好，文章关于DHCP可能是有两处错误。DHCP Offer 和 DHCP ACK都不是广播包，而是直接发到客户机的网卡上的。这是wiki上的链接： https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#DHCP_offer https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol#DHCP_acknowledgement 另外我自己也抓了包验证，https://baixiang.oss-cn-shenzhen.aliyuncs.com/dhcp/dhcp.png。

  作者回复: 这个在答疑环节讲过啦

  2019-03-01

  **13

  **85

- ![img](https://static001.geekbang.org/account/avatar/00/11/67/95/5bd8911f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinwang527

  现在一般电脑的网卡几乎都支持PXE启动， PXE client 就在网卡的 ROM 中，当计算机引导时，BIOS 把 PXE client 调入内存执行。 安装完成后，将提示重新引导计算机。这个时候，在重新引导的过程中将BIOS修改回从硬盘启动就可以了。

  2018-06-28

  **2

  **60

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ed/4d/1d1a1a00.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  magict4

  请问在 Offer 和 ACK 阶段，为什么 DHCP Server 给新机器的数据包中，MAC 头里用的是广播地址（ff:ff:ff:ff:ff:ff）而不是新机器的 MAC 地址？

  2018-06-16

  **6

  **60

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/FMwyx76xm95LgNQKtepBbNVMz011ibAjM42N2PicvqU9tib9n43AURiaq6CKCqEoGo9iahsNNsTSiaqANMmfCbK0kZhQ/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  机器人

  那么跨网段调用中，是如何获取目标IP 的mac地址的？根据讲解推理应该是从源IP网关获取所在网关 mac,然后又替换为目标IP所在网段网关的mac,最后是目标IP的mac地址，不知对否

  作者回复: 是的

  2018-05-25

  **

  **50

- ![img](https://static001.geekbang.org/account/avatar/00/0f/97/39/60d6a10d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天涯囧侠

  在一个有dhcp的网络里，如果我手动配置了一个IP，dhcp Server会知道这个信息，并不再分配这个IP吗？会的话具体是怎样交互的呢？

  作者回复: 有可能冲突的，所以办公网里面一般禁止配置静态ip

  2018-05-25

  **

  **45

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f8/ba/14e05601.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  约书亚

  跨网段的通信，一般都是ip包头的目标地址是最终目标地址，但2层包头的目标地址总是下一个网关的，是么？

  作者回复: 是的

  2018-05-25

  **6

  **34

- ![img](https://static001.geekbang.org/account/avatar/00/10/d0/f1/432e0476.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  X

  进入BIOS设置页面，有一项PXE Boot to LAN，若设置为Enabled则表示计算机从网络启动，从PXE服务端下载配置文件和操作系统内核进行启动；若设置为Disabled则表示从本地启动，启动动BIOS后，会去寻找启动扇区，如果没有安装操作系统，就会找不到启动扇区，这个时候就启动不起来。

  作者回复: 是的，还有一种服务端的配置

  2018-05-27

  **

  **33

- ![img](https://static001.geekbang.org/account/avatar/00/11/4c/fd/8022b3a2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  penghuster

  请教一下，pxe客户端请求的IP，是否最终会直接用于系统

  作者回复: 不会的，系统起来后配置ip是他自己的事情

  2018-06-01

  **

  **31

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7d/95/dd73022c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我是曾经那个少年

  看了虽然懂了，但是对于一个做软件开发的，不知道怎么去实战！

  作者回复: 最后会有一个实验管理的搭建，一台机器足以

  2018-05-25

  **

  **30

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/8f/466f880d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没心没肺

  DHCP Request究竟使用广播还是单播取决于DHCP Offer包中Broadcast位的设置值。该位置1则使用广播发送，置0则使用单播发送。

  2018-05-25

  **

  **28

- ![img](https://static001.geekbang.org/account/avatar/00/10/54/cf/fddcf843.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  芋头

  要是以前大学老师能够讲得如此精彩，易懂，大学就不会白学了

  作者回复: 谢谢

  2018-06-01

  **

  **27

- ![img](https://static001.geekbang.org/account/avatar/00/16/f8/bd/16545faf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈浩佳

  我分享一个我最近遇到的问题: 最近我们的设备增加了dhcp自动分配地址的功能。我把几台设备连到同个路由器上，但是发现每台设备最后分配到的ip都是一样的，我登录了路由器里面查看，显示的设备列表确实是ip都是一致的，mac地址是不一致的。。。。所以就觉得有点奇怪。不过这里要说明的是，设备的mac地址是我们自己程序里面设置的，网卡不带mac地址的----最后查看代码发现，我们设备代码是先启动了dhcp客户端，后面再设置了mac地址，这里就有问题了，所以，我把它倒过来，先设置mac地址，再启动dhcp客户端，这样就解决问题了。。。由于原先启动dhcp的时候还未设置mac地址，所以默认的mac地址都是一致的，所以获取的ip都是一致的。但是，这里也说明一个问题，路由器列表上的mac地址不一定就是分配ip时的mac地址，如果分配到ip后再去修改mac地址，也是会同步到路由器上的，但是不会重新分配ip。

  作者回复: 赞，活学活用

  2020-06-07

  **2

  **21

- ![img](https://static001.geekbang.org/account/avatar/00/10/3f/97/8d7a6460.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  卡卡

  pxe要去tftp下载初始文件，那么pxe自己是不是也需要一个tftp客户端？

  作者回复: 是的，不过tftp很轻量

  2018-05-25

  **

  **21

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  呵呵

  pxe客户端是放在哪里的？

  作者回复: bios

  2018-05-25

  **

  **17

- ![img](https://static001.geekbang.org/account/avatar/00/11/14/b5/d5cb9fad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  zhy

  之前一直不清楚dhcp是干嘛的→_→终于明白了

  2018-05-25

  **

  **15

- ![img](https://static001.geekbang.org/account/avatar/00/0f/84/63/062d0f28.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Tristen陈涛

  文章开头，源 IP 地址 16.158.23.6，目标 IP 地址 192.168.1.6 发包的问题，还是不太懂 此种情况下的结果是： 网关收到 16.158.23.6 包后直接拒绝了还是有别的处理？

  2018-05-29

  **1

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/12/4f/94/05044c31.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  踢车牛

  老师，你好，上面你提到网关至少和当前一个网络的一个网卡在同一个网段内，这么说网关上可以配置多个网卡， 假如网关上有两个网卡，其中一个是192.168.1.6，另一个是 16.158.23.X,这样包可以发出去么？

  作者回复: 可以的，网关上有路由表

  2018-09-05

  **3

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/10/f6/41/62ea275d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  geduo4612

  PXE client 只能把整个操作系统放在内存里面里面？因为此时还木有文件系统啥的吧？DHCP，ARP，IP/TCP协议都是BIOS自带的？

  2018-06-03

  **1

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/11/a5/cd/3aff5d57.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alery

  “如果你配置了网关的话，Linux 会获取网关的 MAC 地址，然后将包发出去。对于 192.168.1.6 这台机器来讲，虽然路过它家门的这个包，目标 IP 是它，但是无奈 MAC 地址不是它的，所以它的网卡是不会把包收进去的。” 刘老师，网络包到达网关，根据第一章网关应该也是会通过ARP协议大吼一声谁的ip地址是192.168.1.6，当192.168.1.6这台主机发现在叫自己就会响应网关他的mac地址，这样不就有获得192.168.1.6主机的mac地址了吗？

  作者回复: 因为不在一个网段，所以吼都不会去吼

  2018-06-16

  **5

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fe/8f/466f880d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没心没肺

  当年联机打C&C，眼看快要败了，偷偷拧掉终结器……嘿嘿……

  作者回复: 是的是的，就是这样

  2018-05-25

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/ce/a8c8b5e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jason

  大道至简。牛

  作者回复: 谢谢

  2018-05-25

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/16/af/73/946c4638.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  🤘🤘🤘

  Clients requesting renewal of an existing lease may communicate directly via UDP unicast, since the client already has an established IP address at that point. Additionally, there is a BROADCAST flag (1 bit in 2 byte flags field, where all other bits are reserved and so are set to 0) the client can use to indicate in which way (broadcast or unicast) it can receive the DHCPOFFER: 0x8000 for broadcast, 0x0000 for unicast.[4] Usually, the DHCPOFFER is sent through unicast. For those hosts which cannot accept unicast packets before IP addresses are configured, this flag can be used to work around this issue.  维基上说的是两种选择(广播和单播 通常为单播)

  作者回复: 赞，后面答疑环节说各种问题了

  2019-04-05

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/12/10/4d/f548bc68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Apollo

  可以更改 boot 方式或者优先级，避免下次再从 pxe启动

  2018-07-22

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/7e/25/f1098783.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  GeekNEO

  在Cobbler里面，或者其他开源的自动化安装系统中，其实是可以设置一个参数进行仅一次安装，后面就正常加载硬盘启动。应该是改变启动顺序。不直接调用PXE启动。

  2018-06-13

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5b/b9/d07de7c6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FLOSS

  交换机，路由器，集线器这些东西的区别是什么？DHCP是不是只有在路由器中有？ARP协议是PC对PC的还是，PC给路由器，然后路由器再给另外一台PC，路由器有自己的MAC地址吗？

  作者回复: 网口都有mac地址的，三个的区别就是上边跑的程序不同，前面讲过二层，三层设备的大概区别。

  2018-05-27

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/16/88/fe/c18a85fe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  随风

  pxe在bios后面启动，那时候还没有操作系统，也就没有文件系统，那下载的文件存放在何处呢？小白求解！

  作者回复: 用光盘安装的时候，也没有文件系统，那文件放在哪里呢，不还是安装在硬盘上了。一样的过程

  2019-04-11

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/bb/85/191eea69.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  搬铁少年ai

  想问一下老师，为什么要用tftp而不是ftp，我记得ap更新系统也是tftp

  作者回复: 轻量级

  2018-10-02

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ca/61/b3f00e6f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  byte

  客户端和DHCP server的识别在MAC和IP层都是广播地址，他们通过BTOOP的协议内容来识别是否为自己的应答？

  作者回复: 是的

  2018-05-30

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/44/35/3b8372c5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chinhu ko

  为什么我的网卡标识一个是 enp5s0f1 ,一个是 lo ,一个是 wlp4s0 ,没有你提到的 eth0 和 eth1 ？

  作者回复: 网卡名字你可以自己改

  2018-05-26

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/57/0f/1f229bf5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Void_seT

  我印象中10.0.0.1/24是路由器地址，这是把这台机器配成路由器了么？配成10.0.0.2/24也没问题吧？

  作者回复: 没问题，这只是个配置的例子

  2018-05-25

  **

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Subhuti

  在这个广播包里面，新人大声喊：我是新来的（Boot request），我的 MAC 地址是这个，我还没有 IP，谁能给租给我个 IP 地址！ 请问，目标端口67，这个是如何获得的呢？

  作者回复: 这个是约定好的，所有实现DHCP的程序都按这个端口来

  2020-01-09

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  吼是同一个网段 是arp协议在发挥作用 是在找mac地址相同的机器 是在二层设备里发生的事情

  作者回复: 是的

  2019-08-27

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/13/0f/70/759b1567.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张飞online

  在上面是，先判断目的地址是不是在同一网段，然后确定是否发arp广播，还是先发广播，发现没有应答，然后发网关

  作者回复: 先用cidr计算

  2018-10-06

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/08/d2/71987d0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  夜禹

  老师，跨网段请求如果在外面没找到目标MAC，就不会回子网找了吗

  作者回复: 不会的，都是在一个子网才找mac

  2018-09-14

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/ba/f4/0dda3069.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小白

  太赞了👍，做移动端开发几年了，因为非计算机专业，好多计算机知识都不清楚，刘超老师讲的真的很透彻，多看几遍终于明白了

  2018-07-20

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/18/de/45875491.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  蔺波

  Dhcp server的回复包可以是广播包也可以是单播包，这涉及到dhcp relay.并不一定是广播包

  2018-06-30

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  为什么ACK阶段，目标mac还是广播？是要通知全网段？

  2018-06-27

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/96/32/3e759846.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  it

  老师，看的云里雾里，越来越不懂了。没有什么基础，可以推荐基本比较基础的书嘛

  2018-06-23

  **1

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ab/69/5f1f0d1c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  支离书

  学习了，pxe客户端获取到的操作系统最后安装到硬盘上了吗？如何安装的？pxe写启动扇区吗？

  作者回复: 是的

  2018-06-22

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/52/bb/225e70a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  hunterlodge

  请问网段是如何定义的呢？同一个子网掩码吗得到的子网相同就是一个网段吗？

  作者回复: 是的

  2018-06-14

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/54/fa/1e4230e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  喵大人

  装完系统后，把dhcp服务器里的next-server删掉就行了吧，就可以正常启动，而不是继续安装

  2018-06-01

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/0e/7c64101d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  $(CF_HB)

  最近做过一些测试，测试内容是: 在局域网内伪造了一个DHCP Server，发现新接入的电脑时，主动广播Offer告知新电脑使用被分配出去的IP地址，测试发现如果那个已经被分配的IP那台电脑是dhcp获取形式，新电脑很容易就把这个IP抢夺过来使用了！(两边都会提示ip冲突)，如果它是手动配置的IP，就没有办法抢夺过来使用，但是新电脑时不时有一瞬间可以使用，另外一个电脑有丢包情况！！想问问老师，DHCP或系统是否有这个机制，当出现IP冲突时，主动发起新的IP申请，更换IP以解决冲突问题？这个机制或这类现象如何解释呢？

  2018-05-29

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/a6/723854ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  姜戈

  Pxe dhcp是工程师的基本素养😁

  作者回复: 是的是的

  2018-05-25

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/64/ff/0a04e025.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  时文杰

  写的真好！！

  作者回复: 谢谢

  2018-05-25

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/49/24/895e9b13.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  难道和如果

  两个电脑直接连一起算么？

  2018-05-25

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/16/85/e3/64571c5b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  痛并快乐着

  老师我用的ubuntu16的虚拟机  每次都不能进行自动获取DHCP的ip地址 每次都要执行/sbin/dhclient 然后才可以获得ip地址 才可以ping通 请问这是什么原因导致的？

  2019-10-21

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/17/85/0a/e564e572.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  N_H

  网关是硬件吗？

  作者回复: 软硬件都可以

  2019-06-27

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/cf/24/6bff9615.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  瓶七七

  一个网络中加入一个新机器，这个新机器会以 IP 地址 0.0.0.0 向整个局域网发一个广播包， DHCP server 收到后会给这个机器分配 IP ，也是以广播地址的方式。可能会有多个 DHCP server，新机器会选择第一个到达的包，然后再发一个广播包告诉大家我选了谁的，之后 DHCP server 会广播一个确认包。 我好像明白了。

  作者回复: 赞

  2019-06-17

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7d/a5/930c9103.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Feng

  pxe安装的系统client端可以选择吗，如果server端配置多个镜像，还是说只会有一个标准？

  作者回复: 可以的，可以配置多个操作系统的Kickstart文件，可以配置pxelinux.cfg多个操作系统的启动菜单

  2019-06-03

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/06/4f/14cc4b53.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不专注的linuxer

  网络帧必须有下层，可以没有上层。那么问题来了，我跨网段访问中，只知道目标的IP地址，不知道目标Mac地址，那么我发的第一个包的目标Mac地址是不是当前源IP所在网关的Mac地址？然后网关再根据目标IP，把网络帧发送给目标IP所在的网关的，然后网关再发送给目标IP

  作者回复: 是的

  2019-05-07

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/0d/b8/6363ca03.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  桐爷哥

  请问老师，如果只有一个网卡，例子中源ip为16.158.23.6，且有网关地址是192.168.1.x网段的，目的ip是192.168.1.6。 那能成功把包发送到网关吗？ 如果可以，到达网关后网关能成功发送到目的ip的主机吗？

  作者回复: 16.158.23.6不能把网关设成192.168.1.x

  2019-04-10

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/93/4a/de82f373.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  AICC

  只要是在网络上跑的包，都是完整的，可以有下层没上层，绝对不可能有上层没下层，这个还是不很理解，能举个简单的小例子？这里的有上层没下层是不是指的是我在应用层不可能凭空得到一个包，并且这个包不是经过我的mac层逐层传递上来的？

  作者回复: 不可能没有底层就发出去。比如有人会问TCP三次握手要带mac地址吗，当然是带的

  2019-03-13

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/ef/f1/8b06801a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谁都别拦着我

  “对于 192.168.1.6 这台机器来讲，虽然路过它家门的这个包，目标 IP 是它，但是无奈 MAC 地址不是它的” 老师，我这一段不太理解，这里的 MAC 地址因为是当前网络网关的MAC，不是192.168.1.6这台机器的所以不匹配是嘛？ 如果是这样的话，在真实跨网段的调用，这里的 MAC 不也是当前网络网关的 MAC 吗？和真实目的IP的机器的 MAC 还是不匹配，不一样会出问题吗？

  作者回复: 是的。网关的mac是匹配的，只不过ip不匹配，所以做转发

  2018-11-25

  **3

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/ff/c6/b9a849c0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  魂断506

  老师，您在文中有提到，当一个新机器加入dhcp网络的时候，它为什么能加入网络呢。。这里任何机器都能加入网络吗？

  作者回复: 连上网线，在一个子网里面，dhcp就能管了

  2018-09-19

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/a0/30/5a7247eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  eden

  我见过的最有趣的dhcp协议讲解👍

  2018-09-10

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/44/1b/fa287ed5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  半桶水

  请问，有什么方式可以批量开启或者关闭pxe功能。

  2018-06-29

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6a/d5/73c75eb3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夜行观星

  赞，对微服务请求地址的视角，与开发角度完全不同，我认为大佬的视角很棒

  作者回复: 谢谢

  2018-05-25

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/29/b7/67/e2d1ae25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  R.M

  我从wireshark抓包的结果看dhcp discover是广播，源0.0.0.0，目的255.255.255.255没错，但服务器发回来的dhcp offer是单播的，如源192.168.1.254，目的192.168.1.151，当然这里面还包括租期、掩码等信息没错，但这里讲的是这时候客户端还没ip所以服务器发广播？所以老师这里dhcp offer是应该是单播吧？可惜不能发图片，不然抓包结果一看就很明了

  2021-09-28

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/16/58/0ef55c6a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek-Pax

  个人觉得老师举得那个网关例子有点小毛病  源ip和目的ip若是在同一个网关的集线器下面 那么网关IP只要配置成源IP或是目的IP同一网段就都满足"网关要和当前的网络至少一个网卡是同一个网段"这个条件,所以"如果将网关配置为 192.168.1.6 呢？不可能，Linux 不会让你配置成功的"这句是错误的,是可以成功的,只是此时源IP由于和网关不是同一网段无法通过arp获得网关MAC地址导致发包失败.实际生活中一个局域网内的所有网卡都应该和网关同一网段.

  2021-07-28

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/1d/83/c1/6c40636e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ming

  1.首先在BIOS中设置启动次序，一般设置硬盘--DVD--PXE。安装好之后，下次就可以从硬盘启动了。有的服务器支持，启动的时候手动选择启动项目。可以通过手动方式临时设置下即可。 2.配置过双路由桥接方式。

  2021-04-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/8d/84/508767e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ZXX

  目标ip和自己同一网段，使用arp获取目标mac 不同网段，发给网关 Pxe 启动第一次听说，通过dhcp服务器过去tftp的pxe服务器的地址，并进行下一步安装

  作者回复: 赞

  2020-06-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  相关的内容，比较深奥。不是专业领域研究，感觉还是有些难度的。一直都在表面使用这些东西，还从来很少去深入研究其底层机制原理。虽然，大部分内容不是很理解。但还是有收获的。

  2020-04-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/37/56/11068390.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  倡印

  若果这样会不会有恶意的人使用相同的广播给dhcp server，或者劫持信息让请求机设定ip错误无法上网。会不会有这么无聊的攻击

  2020-01-21

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/96/74/ef636095.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Dom

  DHCP的RFC协议文档 https://tools.ietf.org/html/rfc2132

  2019-10-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/15/e4/e6/5795b1aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨晓维

  老师，您好，我有以下几个问题，1.请问DHCP Server的配置在哪里配置？什么文件下面吗？2. 对于配置文件，只有next-server，filename，那么这个filename也没有路径，怎么能知道是哪一个呢？这个next-server是如何得到的？ 

  作者回复: 你可以自己搭建一个DHCP Server的。有个默认路径的，配置PXE Server和DHCP Server的时候。

  2019-07-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/17/05/80/9fdb4947.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  🐷

  我想请问一个问题，就是当一个数据包来到网卡前，发现目的MAC就是我，则把它收进来，到网络层，发现目的ip不是我，会怎么办，把这个包丢掉吗？我收到数据包，一层一层向上分析，是否不合适就会丢掉。比如说boot request在子网内广播，非DHCP Server在收到后最终都会丢掉该包吧？

  作者回复: 是的，mac不符合就丢掉了

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/30/ef/0949b631.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  区展翔

  dhcp用广播的方式的话，dhcp offer怎么确认让谁接收

  作者回复: 大家都能收到

  2019-05-26

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/1c/bd/4cdbc941.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  勤

  以前还在上小学的时候都是用的集线器组建局域网，那时候还经常用网上邻居

  作者回复: 小学。。。牛

  2019-05-08

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Geek_e5f033

  做网线时，网线中间几根线对调一下，直接插在另一台机器上，就可以实现双机互连啦

  作者回复: 是的

  2019-05-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/af/73/946c4638.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  🤘🤘🤘

  就像在群里你想要本书  你问谁有  很多人说我有  然后你选了其中一个人说我决定要他的书    这就是为什么mac头中要用广播mac   你不广播怎么让众多的dhcpserver知道？ 用0.0.0.0   dhcp server怎么区分？局域网络当然靠新机子的mac地址呀。

  作者回复: 赞

  2019-04-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/12/cddd766c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大树

  老师你好，如果电脑上有多个网卡，而我ping的地址和这些网卡都不再一个网段内，是否会依次和网卡上的ip进行比较，如果是同一个网段则发送arp，不是就等到最后网卡比较完后，发送到默认网关呢？

  作者回复: 都不在一个网段内，有个default默认网关的

  2019-03-11

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  涟漪852

  协议学起来跟做梦一样😊

  2018-12-11

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/f9/90/f90903e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜菜

  老师您好，能不能具体讲讲数据中心是怎样一种场景呢

  作者回复: 有数据中心一节

  2018-11-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/57/6e/b6795c44.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夜空中最亮的星

  讲的特别好

  2018-11-13

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Walker

  前面讲随便配置一个非同网段的ip，没法通信。那里就指明了是Linux发送和配置规则。那么，在pxe协议中，既然没有安装好操作系统，那么，和TFTP server通信时，网络协议的实现代码在哪里来的

  2018-06-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/56/4e/9291fac0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Jay

  很精彩的文章，DHCP类比租房很贴切，这样记忆很深刻，有个疑问就是这个BIOS就跟自己的电脑主机似的是出场自带的吧，还有PXE客户端是管理员一个个安装在服务器的BIOS里面的吗（这样也挺麻烦吧）还不太了解服务器的启动过程

  2018-05-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/1b/e4/d5e021f9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  crysSuenny

  好cute~

  作者回复: 谢谢，尽量通俗易懂哈

  2018-05-26

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/2b/89/50/aee9fdab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  小杰

  1、引导文件下载到本地即可 2、配置局域网ip、广播地址、子网掩码

  2022-09-02

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/tKvmZ3Vs4t6RZ3X7cAliaW47Zatxhn1aV5PcCYT9NZ9k9WWqRrEBGHicGtRWvsG6yQqHnaWw6cGNSbicNLjZebcHA/132)

  柳十三

  第一个问题答案，猜测是检查启动扇区，如果没有启动扇区，就用pex协议去安装系统;如果有，就启动系统

  2022-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/26/67/c3d90f46.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  may_huang

  如果你配置了网关的话，Linux 会获取网关的 MAC 地址，然后将包发出去。对于 192.168.1.6 这台机器来讲，虽然路过它家门的这个包，目标 IP 是它，但是无奈 MAC 地址不是它的，所以它的网卡是不会把包收进去的。 这段话不是很明白，16.158.23.6 的网关和它是在同一个网段，数据包发送给网关之后怎么走呢？

  2022-05-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/10/bb/f1061601.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Demon.Lee![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  如果是数据中心里面的服务器，IP 一旦配置好，基本不会变，这就相当于买房自己装修。DHCP 的方式就相当于租房。你不用装修，都是帮你配置好的。你暂时用一下，用完退租就可以了。 ----------------- 请教老师，“数据中心里面的服务器，IP 一旦配置好，基本不会变”，数据中心成千上万台服务器，一般用什么方式配置会保证 ip 不会变呢（即使机器重启）？

  2022-01-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0e/ca/a96616b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  月甜

  1、如何配置IP？ 1）手动配置 配置自由度虽然很大，但是配错了，包发不出去。 需要联系网络管理员，确认给你分配一个可用IP，通过配置文件的方式配置IP。 不同系统的配置文件格式不同，但是无非就是 CIDR、子网掩码、广播地址和网关地址。  2）自动配置 DHCP 动态主机配置协议（Dynamic Host Configuration Protocol） 网络管理员只需要配置一段共享的 IP 地址。每一台新接入的机器都通过 DHCP 协议，来这个共享的 IP 地址里申请，然后自动配置，用完了，还回去，其他的机器也能用。相当于租房，随时可租可退。 一般数据中心内的服务器设置的静态IP 个人电脑一般通过DHCP动态分配IP

  2022-01-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/0c/0f/93d1c8eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mickey

  老师这里讲得有问题，DHCP Offer是单播发给客户端，而不是广播。

  2021-11-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/54/8c/a3b98f6c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  denofiend

  在家里，链接的是同一个路由器。我两个手机，ip 地址手动改为一样的，为啥还能上网？ DHCP不允许ip 地址重复，到底哪里出问题了呢？

  2021-11-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/d8/bc/a20dc219.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  A漩

  请问DHCP发生在什么时候，在没有安装操作系统的时候，就能用DHCP分配IP吗？ 按照PXE这个流程来看，是先以DHCP客户端的身份申请到IP，然后再安装操作系统！

  2021-10-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/7c/91/0d8e2472.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  子非鱼

  补充一下哈，平时家里的路由器也充当了DHCP server。

  2021-09-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7e/1f/b1d458a9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iamjohnnyzhuang

  家里接了家用路由器，手机、电脑都可以通过wifi上网。想知道，手机电脑被DHCP分配到的IP是哪里来的呢？ 应该不是路由器作为DHCP Server 分配的吧？

  2021-09-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/29/17/a4/737689d4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  温淼

  “客户机会在租期过去 50% 的时候，直接向为其提供 IP 地址的 DHCP Server 发送 DHCP request 消息包。客户机接收到该服务器回应的 DHCP ACK 消息包，会根据包中所提供的新的租期以及其他已经更新的 TCP/IP 参数，更新自己的配置。” 这里的更新的IP参数是DHCP server随机分配的吗？是有可能跟之前的不一样吗

  2021-08-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/34/29/820852bb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  -m

  网络小白，请教一下同学们。假设家里有电脑、wifi路由器，光猫这三个设备。 现在电脑浏览器需要访问百度，按照课程的说法： 1. 数据包直接到网关，网关再出去，最终到达百度服务器 问题： 是不是应该先经过wifi路由器，再到光猫，光猫作为一个统一网关出口呢？ 虚心请教

  2021-07-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/24/d3/a9/2b84cc97.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Linux C·Core Api

  目前不太懂pex 不影响后面的学习吧？

  2021-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/3c/c86e3052.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  猛仔

  (Linux 不会让你配置成功的，因为网关要和当前的网络至少一个网卡是同一个网段的，怎么可能 16.158.23.6 的网关是 192.168.1.6 呢？)想问网关和网卡是什么关系

  2021-06-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/58/bb/c78c6482.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李小隆_

  不懂就问：如果不但将自己IP地址改为 16.158.23.6 ，而且还将网关地址改为  16.158.23.254 。那可以 ping 通 192.168.1.6 吗？

  2021-05-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/41/37/b89f3d67.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我在睡觉

  问一个问题，dhcp server回复给申请IP地址的主机的IP offer包的包头的Mac地址是一个广播地址，那么如果多个主机同时注册，大家都收到了相同的IP offer包要如何处理呢？

  2021-04-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/96/11671504.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  蔡箐菁

  DHCP RFC 文档 里似乎没说租期过了百分之五十就要续租？

  2021-04-28

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eqt9faiaaWA5B7piazRicHDaiaREic8y7bH2UQQgxbMRWycPjb3G56WXjRQwM45kiaica5XSJpZd8pMc7EtA/132)

  Geek_96b5b8

  老师，有个问题我看了好多资料都没搞清楚。比如我们公司的所有电脑的IP都处在同一个子网网段，因为用子网掩码划分了子网给不同的电脑主机的。那我想知道，公网IP是不是也是同样的道理，公司被分配的那个公网IP是不是也是某个子网网段的其中一个？

  2021-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/01/95/fd09e8a8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  布拉姆

  “DHCP Server 便租给它一个 IP 地址，同时也给它 PXE 服务器的地址、启动文件 pxelinux.0。” 对于pxelinux.0, 这句话下面的图画的是DHCP server

  2021-04-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/33/ab/d8ba4242.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  哒哒哒

  作者讲的太好了，以前虽然想过但没想明白的问题，这里都得到了解决。

  2021-03-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c7/05/19c5c255.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  微末凡尘

  以问答写笔记： 1. 正确配置IP? CIDR、子网掩码、广播地址和网关地址。 2. 在跨网段调用中，是如何获取目标IP的mac地址的？ 从源IP网关获取所在网关mac, 然后又替换为目标IP所在网段网关的mac, 最后是目标IP的mac地址 3. 手动配置麻烦，怎么办？ DHCP！Dynamic Host Configuration Protocol！ DHCP, 让你配置IP，如同自动房产中介。 4. 如果新来的，房子是空的(没有操作系统)，怎么办？ PXE， Pre-boot Execution Environment. "装修队"PXE，帮你安装操作系统。

  2021-03-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/96/3a/e06f8367.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  胡小涵

  为什么我们公司的架构师就只知道让我们这些开发人员去用手装，真特么水！

  2021-02-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/c9/fe/874b172b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  benxiong

  老师，我们线上阿里云的一个服务器今天出现访问故障，经排查发现域名解析成 127.0.0.1，不知道这是什么原因导致的。请老师帮助！阿里云域名提供商还没有回复。谢谢老师

  2021-01-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/5d/29/8d83861d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  effeywang

  老师您好 有个疑问？就是本机和附近机器不是一个网段那个例子，当本机把报文发送给网关后，为什么直接填的是网关的MAC地址呢？正常向外部访问不也是要通过网关吗？

  2021-01-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/76/87/c5b6ef6b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  袁满

  之前没有听过PXE这个概念，看了之后随即想到它在云计算领域的重要性。没想到老师后面就提到了它在云计算领域会大有用处。不过不知道老师后面会不会讲到具体是哪些用处。

  2020-11-19

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJUHvicvia3fpBUA1aB5rmlDVAtq8qs4luk6uXCUO14oEJ0JnunWZ5H9iaiauSKriaIibY6HQllDSFZsLsQ/132)

  Geek_33348f

  为什么DHCP两次广播的时候，第一次使用0.0.0.0，第二次使用192.168.1.2

  2020-10-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/d6/aa/ffa0bd0e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  SolskGaer

  pxe这个协议还是第一回听说

  2020-10-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ce/f4/5bfc786a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Vilochen.

  DHCP Offer中的报文格式图，DHCP Server的IP地址不该是0.0.0.0吧，这个时候，服务器提供报文，服务器应该是知道自己的ip是多少的

  2020-09-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  PXE平常在工作中会有接触，只知道配置，工作原理确实自己没有去深究，惭愧

  2020-09-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/6c/a8/1922a0f5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郑祖煌

  1.重启之后就会在BIOS系统里面记录要读取扇区的位置信息，这样当他启动的时候就不去再去连DHCP服务器请求信息，而是直接去读对应的扇区然后启动。 2.没有。

  2020-08-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/fc/68/1371d3fc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iron

  老师，图片中的DHCP Server 的IP 为什么由 0.0.0.0 变为 192.168.1.2 呢

  2020-08-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/39/d8/9727b45c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mångata

  双方通信的本质是收发包呗。协议的职责在于如何协调通信双方收发的内容，完成预期通信目的。协议的设计会基于下层的硬件支持和软件支持，如广播地址，对某特定的网络包在网络内进行广播，等等。

  2020-06-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/de/bf524817.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  慌张而黑糖

  dhcp offer的的格式中，为什么dhcp server 的ip是0.0.0.0

  2020-05-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d6/39/6b45878d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  意无尽

  这一篇看了两三遍才慢慢看懂，没基础的太难了。 总结： 1、如何配置 IP 地址？这就会涉及到动态主机配置协议(DHCP)，其主要工作方式如下： 1) DHCP Discover. 新来一台机器需要配置 IP 的时候，靠“吼”一声来通知 DHCP Server。 2) DHCP Offer. DHCP Server 租给新来的机器 IP 的过程。并且，DHCP Server 还要为此客户保留一份 IP ，以避免跟别人重复。 2、IP 地址的收回和续租 客户机会在租期过去 50% 的时候，直接向为其提供 IP 地址的 DHCP Server 发送 DHCP request 消息包。客户机接收到该服务器回应的 DHCP ACK 消息包，会根据包中所提供的新的租期以及其他已经更新的 TCP/IP 参数，更新自己的配置。这样，IP 租用更新就完成了。 3、预启动执行环境（PXE） 1) 首先，启动 BIOS。读取硬盘的 MBR 启动扇区，将 GRUB 启动起来；然后将权力交给 GRUB，GRUB 加载内核、加载作为根文件系统的 initramfs 文件；然后将权力交给内核；最后内核启动，初始化整个操作系统。 4、解析 PXE 的工作过程 1) 首先，启动 PXE 客户端 2) 其次，PXE 客户端知道要去 PXE 服务器下载这个文件后，就可以初始化机器。 3) 然后，PXE 客户端收到这个文件后，就开始执行这个文件。 4) 最后，启动 Linux 内核。

  2020-05-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/df/1e/cea897e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  传说中的成大大

  我有个小疑问哈！在dhcp回复新人的ip地址时候，只有通过端口号才能确定回复给谁？如果网络中还存在着另一个程序也是同样的端口呢？

  2020-05-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/ce/b2/1f914527.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  海盗船长

  老师你好，我用抓包工具抓了下DHCP，惊奇地发现在DHCP Offer和DHCP ACK阶段，目的地址不是255.255.255.255 而是我的域名（我的电脑加了公司的域了）。请问如何解释呀？

  2020-04-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  DHCP之所以在Offer的时候进行广播,是因为在网段中,可能有多个机器在以0.0.0.0来寻求IP

  2020-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/4b/a276c1d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  儘管如此世界依然美麗

  请问，网络号和网段不是需要相同吗，为什么在DHCP的DHCP Discover过程中，是使用0.0.0.0来作为新机器的初始IP地址来发送到255.255.255.255的？

  2020-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/68/a8/1fa41264.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  马什么梅

  专科学的网络安全,用锐捷的防火墙做过dhcp

  2020-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/a7/84/b9b81d8d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大包

  TFTP Server为啥不一次给完文件，承载不了吗？

  2020-04-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/6a/01/d9cb531d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  这得从我捡到一个鼠标垫开始说起

  牛，pxe协议

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/63/7a/e91c3771.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  WulalaOlala

  有没有好用的mac抓包的工具集可以推荐的？

  2020-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/de/bf524817.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  慌张而黑糖

  对于dhcp协议的一点问题，在我获取到ip地址以后，没过多长时间，我断开了，然后再接入这个网络下，我发现我的机器少了dhcp discover这一步，是因为我的ip地址的租期还没到吗。

  2020-03-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/de/bf524817.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  慌张而黑糖

  我是小白，问个比较愚蠢的问题，就是两个网段不同的ip地址是怎么通信的？是通过一层一层的代理吗？还是什么办法，希望能有人帮忙解答一下，越详细越好，谢谢啦

  2020-03-25

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  PXE，RARP，这两个是之前一直没找到应用场景的协议，学习了！

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/94/0a/7f7c9b25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  宋健

  很不错！

  2020-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  这节是我影响最大的是，在学习网络协议的时候一定要记得某一些原则，不能乱套用和乱按寨扎营，就说会原点了：要有自己的知识体系和框架，这样你在学习东西的时候，就知道放在哪里，同时你在思考一件事情时，你根据你的体系和框架来思考和判断。回到具体知识点，拆解一个网络协议，一步一个脚印，扎扎实实地学会、学透。

  2020-03-20

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEIaTvOKvUt4WnuSjkBp0tjd6O6vvVyw5fcib3UgZibE8tz2ICbTfkwbzs8MHNMJjV6W2mLjywLsvBibg/132)

  火力全开

  由于DHCP服务器响应的时候都是广播， 如果有两台机器同时发起BOOTP request， 按照文中描述的过程显然会产生并发的问题啊？

  2020-03-04

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/45/b4/afa76165.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  华子91

  DHCP Offer 部分IP头源DHCP IP地址写错了吧，写成0.0.0.0了。

  编辑回复: 收到 我们看一下

  2020-03-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/23/96/e1f20e9f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  树袋熊

  如果将网关配置成了16.158.23.0，然后恰好16.158.23.0是一个网关的话，但是他们不在一个物理局域网里面。这样会发生什么？

  作者回复: 不能设置为0，从1到254

  2020-01-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/bf/0b/173b87bb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  慕城暖阳

  刘超老师，您好，我有个关于pxe协议传输过程的问题，就是在数据中心自动安装linux操作系统的过程中，explinux0,explinux.cfp,tptp下载的文件，还有内核和initramf,都属于linux操作系统的组成部分吗？还有就是如果exp协议存在，就不需要GRUB加载内存和根文件系统initramfs了

  2019-12-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/6c/0f/7d242cc2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吃水不用钱

  最终租约达成的时候，还是需要广播一下，让大家都知道。 请问下广播的是什么信息？通过什么协议广播？

  2019-12-31

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  kissingers

  老师，请教个问题：linux 一接口被选中作为发包端口，该端口又有多个ipv6地址，那么选哪个作为源地址呢？接口的第一个地址？谢谢

  2019-11-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/10/b7974690.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BD

  Dhcp负责动态分配ip地址给新人 负责回收ip地址 配合pxe协议可以安装操作系统

  2019-11-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/35/18/f8ddebb6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  汐微浅

  既然16.158.23.6 可以通过跨网关最后发送到 192.168.1.6，为什么前面又说没有什么现象，就是包发不过去呢？这不是前后矛盾嘛 

  2019-11-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/5c/02/e7af1750.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  teddytyy

  对于开始 16.158.23.6发向192.168.1.6的例子有点不解，既然linux发现两个ip不在同一网端，发向网关，那网关为什么不能发现目标地址，再把包发回给目标地址呢？

  2019-11-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/1a/b5/4c92decd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  弦乐

  测试环境中经常有同事在搭建云环境时，习惯将控制节点服务器设置为dhcp服务器，计算节点通过pxe安装云环境，搭建完环境一般不会关闭dhcp服务，这时其他人搭建另一个云环境也会起一个控制节点，且pxe的vlan与上一个pxe的vlan相同，大部分时候这时通过pxe发现计算节点就失败了，一直没搞清楚为什么？

  2019-11-06

  **

  **

- ![img](https://wx.qlogo.cn/mmopen/vi_32/ox4nIqvFdt77xSW2rm5QJcYPj0r10up6etUXohw47B50Giby7wexBksrpcSu3n9n7iaqGFCGt2STMkJlJgKaTzvg/132)

  鱿鱼先森

  请问当有多个新人服务器同时发起ip申请请求，只有一个DHCP服务器的情况下，新人服务器如何判断该回复ip是属于自己的？靠端口吗

  2019-10-15

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a5/98/a65ff31a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  djfhchdh

  1、第一次用PXE协议安装操作系统后， 修改下bios，从本地硬盘启动就可以了

  2019-10-07

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  随缘

  BIOS启动项中可以配置启动顺序，安装完系统后可以配置优先从硬盘加载系统

  2019-10-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/04/c5/89aa711b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  熏墨染

  对于没有一点网络基础的我来说，真的看不懂，网关 网段这些基本概念都不清楚，是不是应该先去把计算机网络书本过一遍。。。

  作者回复: 网上搜一下就可以哈

  2019-09-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/87/e7/043f9dda.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  .

  2019年度感动中国程序员老师

  作者回复: 谢谢颁发此奖

  2019-08-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/41/87/46d7e1c2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Better me

  老师我想问下，pxe执行过程中的客户端都是指数据中心的空机器吗？ “把客户端放在 BIOS 里面，当计算机启动时，BIOS 把 PXE 客户端调入内存里面，就可以连接到服务端做一些操作了”这里的BIOS系统是服务端计算机启动的BIOS吗？然后BIOS又是如何把PXE客户端装入内存中的呢？这几点不是很懂，希望老师抽空解答下

  作者回复: 是的，要做一个lan里面。是计算机启动的BIOS。本身就自带的 pxe客户端，你个人电脑里面也有的，从网络启动就是。

  2019-08-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/e4/e6/5795b1aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨晓维

  老师，您好，在图中，对于initrams来说，前面已经将它嵌入在配置文件里面，而PEX也已经进行读取，后面还需要单独再给一次吗?看图的意思是这样的，不太理解这两次的initramfs有什么区别或者说是，这个步骤是为了什么？

  作者回复: 前面是配置，后面是真的initramfs

  2019-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/e4/e6/5795b1aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨晓维

  老师，您好，我有以下几个问题，1.请问DHCP Server的配置在哪里配置？什么文件下面吗？2. 对于配置文件，只有next-server，filename，那么这个filename也没有路径，怎么能知道是哪一个呢？这个next-server是如何得到的？ 

  作者回复: 网上可以搜索一下DHCP Server和PXE Server的搭建，还挺简单的，就像咱们搭建一个网站，也是有个默认路径的

  2019-07-22

  **

  **

- ![img](https://wx.qlogo.cn/mmopen/vi_32/JtEfIwx1OuZiaxnhjTR6UmUy6Y0EibgFfcEicWO6goVWDNrYVL2ibcicoJlcKP5MaMNMYLMSicqIwwib6oYANIX7OES3g/132)

  pxiou

  打卡第一天

  2019-07-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/3b/85/7f26586d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  周维勇

  应该是通过mac区分吧

  2019-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/67/4b/6a91f14b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿拉索

  我记得以前装备搭建的pxe服务器，不是配置pxelinux.cfg，而是直接配置mac地址的，及一个default文件

  作者回复: 看看格式是不是一样

  2019-06-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7d/a5/930c9103.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Feng

  pxe预装的系统client是可以选择的吧？如果server端有多个镜像

  2019-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/30/ef/0949b631.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  区展翔

  作者你好，我自己用抓包软件测试过，发现dhcp在offer和ack的时候目的地址都是用的dhcp服务器分配的地址比如192.168.1.41，而不是广播地址255.255.255.255，还有目的的物理地址是我的客户端的物理地址不是那个广播用的ff

  作者回复: 是的，这个在答疑里面回复了，根据客户端的不同而不同

  2019-05-28

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eovceicRfqpejeicblRZCWxXTaj1QDTJIzjMicialUqCwpxnflcUib74OcbXTLWyb0lvFdFERiafQg5cOAg/132)

  Spencer

  有连个问题：1. 当计算机启动时，BIOS 把 PXE 客户端调入内存里面。PXE客户端是存在哪里的呢？BIOS要从哪里把PXE客户端调入内存？ 2.从PXETFTP服务器下载的操作系统文件是放在哪里呢？硬盘吗？

  作者回复: BIOS中开启或关闭（PXE）网卡启动方法，就在BIOS里面。 是的，硬盘上，会安装的，和用CDROM安装是一样的

  2019-05-18

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epKQRu38KaYHy1E7TjJ16FOrEIIhHXhvo2tibWicsmTEbnMHuxniapHX3hnI86ZegTMttQibqKO75wQew/132)

  Geek_c749ce

  可是我用wireshack抓包，断开wifi再连上，之抓到了request和ack的包，这是为什么？

  作者回复: 看看是什么包，从哪里发到哪里的

  2019-05-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/c1/93031a2a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Aaaaaaaaaaayou

  跟eric同样的疑问，认为Dhcp服务在回应的时候应该是用目标机器的mac，因为如果同时有多个机器在请求ip就不知道回应是发给谁的了

  作者回复: 也可以回复给特定的mac的，答疑环节详细讲了

  2019-05-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/24/d2/a5e272ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夜空咏叹调

  DHCP主要是给新进来的MAC地址分配IP地址，且它可能会在多个DHCP serves里得到回应，然后一般会取第一个作为自己的ip地址。

  2019-04-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9b/16/1dd3e6fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  曲冠儒

  很有收获

  2019-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9b/16/1dd3e6fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  曲冠儒

  很好

  2019-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fb/2b/44cd01e9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BillWu

  讲得好

  2019-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/d2/f5/b2c95027.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  waldow

  客户端怎么知道dhcp服务器地址的？

  作者回复: 广播

  2019-04-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/24/18/8edc6b97.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Roger

  有一个问题: 为什么说是配置客户端的MAC地址？，前面提到说是自己的MAC地址已经知道，服务的MAC地址不知道，这是不是有点冲突，而且根据第一节提到的，知道了源IP地址和目标IP地址，如果源IP地址和目标IP地址不在一个网段，需要通过网关，MAC层会返回网关的MAC地址，这个地方有点不是很理解，希望老师和同学有知道的可以帮忙解答下。

  作者回复: 这是启动的时候，原来说的是发送的时候

  2019-04-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a0/57/3a729755.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灯盖

  讲的好，感谢

  2019-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/81/e6/6cafed37.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  旅途

  队列？

  作者回复: 是的

  2019-04-05

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/86QEF74Mhc6ECbBBMr62hVz0ezOicI2Kbv8QBA7qR7KepeoDib9W6KLxxMPuQ24JGusvjC03NNr8uj8GyK0DxKiaw/132)

  HerofH

  请问一下，这里新主机加入网段后，会发送一个广播报文，我想知道新主机是如何知道目的端口的呢？

  作者回复: 固定端口呀

  2019-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/3d/e5/a2176e73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  愤怒的一口

  新机器进来，申请分配ip地址有多个dhcp service接受到dhcp request，dhcp service发出多个dhcp offer机器这会会收到多个返回的ip分配，但是会选择先收到的，并然后告诉其他的dhcp service已经收到ip分配。 同理在多台机器进来申请分配ip地址时，当dhcp service接收到一台都就会将一个ip地址分配出去,并ip会被锁定不会再次分配出去。

  2019-03-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/3e/c7/e1f4f2f4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  臻

  有的时候网络断网，拔掉路由器上的一根网线（Lan口上的），网络就通了，是DHCP分配IP有异常吗？

  作者回复: 这个要具体看了，通过这个描述很难定位

  2019-03-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/47/82/faf79264.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我的video

  讲的非常通俗易懂

  2019-03-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/5b/72/4f8a4297.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  阿May的海绵宝宝

  客户机发送请求给server，有明确的ip和mac。但是server发回的包用的广播mac和ip。所有客户机都会收到这个包吗？客户机如何区分这个包是给我的？

  作者回复: 是的，所有的都会

  2019-03-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/27/40/ae886719.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  geekYang

  数据中心里面的管理员可能一下子就拿到几百台空的机器，需要一个个安装PXE客户端和里面的tftp客户端吗？

  作者回复: 不需要，bios里面有的

  2019-02-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/00/6e/11362a1e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  感动超人

  @magict4 是为了让同网络的机器都知道有新人加入

  2019-02-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/51/01/9f10cb3f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ellis

  老师好，有两个疑问望能解答下 1.当前请求的IP判断不在同个网段的情况下，会向该网段的网关发出请求，那该网段的网关怎么寻找目标网段的网关 2.寻找目标Mac地址的过程中，是找到网关的Mac就先返回，再找到目标IP的Mac再返回。还是直接就找到网关再找到目标地址IP的Mac直接返回 谢谢

  作者回复: 1. 路由协议 2.找到网关mac就返回，先发给网关，然后一跳一跳的找到下一个网关

  2019-02-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c7/11/89ba9915.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  永远的草莓地

  老师，dhcp server 在哪里跑的？路由器里面吗？

  作者回复: 路由器，服务器都行，咱们家里的路由器就会默认带一个

  2019-01-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  老师讲的太好了！

  2019-01-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  老师，我们爱您！ 谢谢你！

  2019-01-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/3a/4f/92df5916.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小小

  老师讲的真好👍

  2019-01-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得1.客户端 都拥有自己的mac地址，接入网络 ，路由 或者 网关 会自由分配ip地址，之后 就可以与其他客户端 、服务器 通讯了。 2.DHCP可以自动分配 IP地址，所以 IP地址 既可以 手动分配，也可以自动分配。 老师的第一个问题：pxe预安装问题，可以在里面加个判断标准 比如 flag，开始flag=false，安装过一次flag=true，不会再次按照，就可以了。 第二个问题：玩过 在没有 外网（没有以太网） 的条件下，1.用路由  组成局域网 打游戏2.把打印机加入 网络，用wifi 的打印机控制端 打印文件。

  2018-12-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5b/e0/d27145c8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  discoverer-tab

  学到了，打卡😜ོ

  2018-12-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/7e/25/f1098783.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  GeekNEO

  对于PXE来说，如果用Cobbler批量安装，可以在服务端使用pxe_just_one配置1，就是为了防止误重装。其他的批量安装服务，如: FAI,Spacewalk等很少使用。不是特别清楚，不过我觉得原理都是一样，就是开启一次pxe安装。

  2018-12-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/8e/10/10092bb1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Luke

  BIOS里面有pxe开关，也有叫做lan boot，他们可以配置pxe是否启用。 但是，我有点疑惑，第一次的时候必须是enabled的，后续可以远程disable这个设置吗？还是说pxe协议里有这方面内容可以保证安装成功后自动禁用pxe enabled设置。 原始的连接只试过两台笔记本对联，还有就是网络共享。高中用win98看到网络邻居里老师电脑里的内容感觉还是很神奇的😁。

  2018-12-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/39/61/dd78b0ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  梦见山

  bios中同时有 tftp client和pxe client？

  2018-12-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e5/4f/731ef2c1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  geektime_zpf

  “数据中心管理员一下子拿到几百台空的机器，一个个安装操作系统，会累死的。” 遇到过这种场景，这个过程实现自动化还是可以的，记得之前在华某公司做过类似的自动配置过程。需要提前准备好DHCP Server，TFTP  Server，配置文件*.cfg*（一本以ip地址等标志性的规则命名），mac地址，python脚本（用于自动实现整个自动化过程），网络线缆若干，网络配置命令若干，管理员只需要给几百台空机器接通电源重启时，脚本可以通过先分配IP地址，再去服务器上取操作系统、自动安装操作系统，最后加载配置，几百台空机器即可正常运转。当时还不太懂用户为什么有这样的需求，现在看了老师的文章，恍然醒悟。期待接下来的文章。

  2018-11-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/2d/1f/8d53c785.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  沃野伏地

  申请IP地址是怎么发出的？是怎么广播出去的？

  2018-11-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ef/f1/8b06801a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谁都别拦着我

  “如果将网关配置为 192.168.1.6 呢？不可能，Linux 不会让你配置成功的，因为网关要和当前的网络至少一个网卡是同一个网段的，怎么可能 16.158.23.6 的网关是 192.168.1.6 呢？“ 这样的话再配置 ip 为16.158.23.6时，同时需要配置的网关 IP的对应 MAC 地址，是怎么获取到的呢？

  作者回复: mac是不能配置的，是通过arp得到的

  2018-11-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/48/d4/755f058b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bboot9001

  老师 你好 问个问题 linux 下想监控网络的变化 怎么解决？比如网卡关闭打开 ip地址变化等

  作者回复: 只要监控系统重启就行了，为啥要监控ip变化呢？

  2018-11-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f7/38/1b74f53d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Nero.t.Kang

  为什么本机的IP一定要和网关在同一个网域内呢?

  作者回复: 协议就是规定嘛

  2018-11-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/e5/e5/0395607e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小太阳

  老师问你个网络连接问题，wan miniport(pppoe)跟以太网连接有什么区别？wan miniport是网卡吗谢谢老师

  2018-11-16

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIY4XKvs9LvsXbZwRNUic4wDpiaeyibZibaJLeq2yOTZK4nHwIjlkiawtEiaC7lOpfrJXgPfwYKFHB304kg/132)

  Jamin

  埃罗芒阿老师 安装完系统之后，机器需要关闭pxe功能的吧，不然每次都会走一遍流程？关了应该就正常启动了吧 2018-05-25 作者回复 是的，可以关闭网络启动方式，也可以在服务端修改 刘老师说的服务端修改具体是指修改哪个配置文件的参数？ 是指 pxelinux.0 引导文件对应的配置文件 pxelinux.cfg？ 这个 pxelinux.cfg 配置文件是定义引导菜单的相关选项参数吧？ 如何避免某些将 PXE 启动作为第一启动项的服务器重启后误重装系统？ 比如设置：如果在 30 秒内没做任何选择则默认从本地磁盘启动 相关的几个配置参数为： default local timeout 30 label local  menu label Boot from ^local drive  localboot 0xffff 但上面这种配置修改后达不到自动化安装的目的，还是得在引导菜单中手动选择从网络启动安装吧？ 关于自动化安装操作系统，可以使用 Cobbler。 Cobbler 中有一个参数 “pxe_just_once” 可以避免重复安装。 pxe_just_once 预防由于服务器设置从网络引导，导致循环安装，激活此设置，机器会告诉 Cobbler 安装也完成。Cobbler 会将对象的 netboot 标志改为 false，这会强制服务器从本地引导。

  作者回复: 见答疑环节，有详细描述

  2018-11-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/1b/4d/2cc44d9a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  刘忽悠

  同问在 Offer 和 ACK 阶段，为什么 DHCP Server 给新机器的数据包中，MAC 头里用的是广播地址（ff:ff:ff:ff:ff:ff）而不是新机器的 MAC 地址？客户端倒是可以理解，因为开始不知道DHCP server的地址，或者是要给多个DHCP server返回信息，所以用广播可以理解，不太明白，既然DHCP server已经收到了客户端请求，那么它应该知道这台主机的MAC地址，为什么在回复的时候这里面还填写的是ff.ff.ff.ff.ff.ff,而不是主机的MAC地址呢？

  作者回复: 见答疑环节

  2018-11-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/30/ec/10ea524c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大关

  讲的真好！

  2018-11-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ce/c6/958212b5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sugar

  老师我想问下，dhcp既然是租约，在分配ip时候是否由某一方声明租约有效期呢，比如在公司我的笔记本连入wifi后刚用dhcp分配一个ip，然后我突然断电 计算机来不及发出任何请求 端着笔记本我就跑出wifi覆盖范围，那么ip何时能被dhcp回收？

  2018-10-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e1/a1/17ddf150.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大龙

  不要把网卡设为第一启动项，把硬盘设为第一启动项，从本地启动就可以避免重装了

  2018-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/97/06/ad491e82.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  楼兰

  苹果Mac电脑可以在线安装MacOS系统，是不是类似PXE的原理，但是普通PC笔记本也有BIOS，为啥不能在线安装Windows呢，是微软禁止了麽

  作者回复: 能啊，好多网吧就是的，需要一个pxeserver

  2018-10-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/ec/2a/b11d5ad8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  曾经瘦过

  跨网段发送包  是   源 IP 地址 填写所在网管mac 地址 发送到网关，然后 到达网关后，网关获取目标IP 所在网关的MAC地址 替换MAC 地址 发送到目标IP 网关，然后在 目标网关 获取目标IP的MAC地址 再发过去吗

  2018-10-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/df/f517304b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  橙子

  想问一下超哥BIOS中的PXE是什么时候放进去的呢？

  作者回复: 刚买的笔记本，就有从网络启动

  2018-10-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/38/3a/102afd91.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  你好梦梦

  为什么DHCP Offer发出的报文中的目的mac地址是ffffffffffff呢，此时虽然不知道新来主机的ip，可是他的mac应该是知道的。

  作者回复: 见答疑环节，回答了这个问题

  2018-10-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/38/3a/102afd91.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  你好梦梦

  为什么DHCP Offer发出的报文中的目的mac地址是ffffffffffff呢，此时虽然不知道新来主机的ip，可是他的mac应该是知道的。

  作者回复: 见答疑环节

  2018-10-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/cb/1d/fb630819.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_f47e84

  老师您好，我有个问题想请教下，就是您提的第二个问题，如果是家用路由器的话，在路由器上有个dhcp服务器的设置，然后计算机里边配置ip地址的时候有一个 自动获取ip地址，这两个dhcp是什么关系呀？假如把路由器的dhcp关掉，计算机里边那个好想还是能够自动获取到ip地址，这是怎么回事呀？ 环境(大学宿舍小局域网，电脑win10，家用路由器)，希望老师看到了可以解答一下。😁

  作者回复: 路由器上的服务端，计算机上是客户端

  2018-09-23

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJJJdibLfaPjOqvgdRSn11JhwdqaI2KiaUdkeA9cwQ5btpnFX00V0rpFXlksKfBtaArprs7DN2ybFug/132)

  逍遥夏末

  老师好，有个疑惑的地方，16.158.23.6把ping包发给16.158.23.6对应的网关，网关转发给192.168.1.6所在的网关，这样包就发送到了192.168.1.6所在子网，就可以ARP获得目标mac？ 请问这个解释问题在哪里呢？

  作者回复: 多两个网关，就可以了

  2018-09-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/e1/fa0435cf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  斩风

  讲的真不错，之前买的刚开始看，一下子看了 4 讲了，停不下来，每一讲都有收获，深入浅出，希望再出一些其他方面课程

  2018-09-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/b8/fb19aa6a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Marnie

  简单点儿说： DHCP 不仅负责给客户端分配IP，还负责给新机器装系统。

  2018-09-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/ca/3f/cec5b84a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  回家放羊 skr~

  服务器批量安装第一次pxe引导系统 安装完后重启操作会自动读取硬盘里的系统，我的理解是磁盘的优先级高 刚开始硬盘是空的 扫描发现是空会跳过而选次优先级可以是pxe或者其他介质……

  2018-09-15

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  DHCP 请求哪些设备会转播哪些不会转播，是什么东西控制是否转呢？比如机器连交换机1，交换机1连交换机2，交换机2连路由器1，交换机1和2会转播所有交换机接口，而路由器1不转播，是这样吗？因为如果路由器也转播的话，岂不是会广播到公网去了？

  作者回复: dhcp只会局限于一个子网

  2018-09-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4b/2f/186918b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  C J J

  大学时没学懂网络协议这门课程。毕业一年多了，从事java开发两年多，到目前还是云里雾里，开始补上这块知识。讲得很好，需要看几遍、花时间消化。

  2018-09-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/71/ab/b19a1ba2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BUG君

  mac是下一个网关的，那最终进程的mac放在吗？？ip里面也有mac?最终进程的mac包含在ip里面吗？

  作者回复: 不需要最终mac的，只需要最终ip的

  2018-08-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/ea/10661bdc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevinsu

  讲的很精彩！受教了

  2018-08-25

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Pr8laRQY3skrzzgen37ZIt4HQvtaThAcqvyK8eAzc9DRiak803q5HS7gCnXFxpx6CWibqT1Sic0h1TLMmVNUpJRibA/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  nico

  dhcp协议分配ip后，回复的包中的mac地址为什么不是申请机器的mac地址？

  作者回复: 可以设置到底是单播，还是广播

  2018-08-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/07/05/db76a933.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jay

  老师，您好！由于还没有得到 DHCP Server 的最后确认，客户端仍然使用 0.0.0.0 为源 IP 地址、255.255.255.255 为目标地址进行广播。如果多个客户端同时进来要求分配ip,服务端怎么区分不同客户端是哪个客户端需要确认呢？

  作者回复: mac

  2018-08-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/14/ed/a7584bde.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Iris

  最近在负责做项目里云上的网关设置，真是云里雾里的，从老师的课里恶补中

  2018-08-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/23/25/fe87906a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  anzaikk

  小白一个，听了两三遍，才理解的更好一点，要静下心来弄懂每个协议之间的关系(≧∇≦)/

  2018-08-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/12/0d/7b696603.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大溪

  能否介绍一下bootp协议？

  2018-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/54/53/d5d0ec20.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Lilian

  没篇文章几乎都在两遍以上，每次看都觉得很精彩，虽然有些地方不懂，自己知识面比较薄，但是每次看都会有不一样的收获，赞，感谢老师

  作者回复: 谢谢

  2018-07-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/7c/91/0d8e2472.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  子非鱼

  又巩固了一遍网络知识，这个价格很良心了

  2018-07-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/4d/3e/5e0cac1b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  雨

  这节看得真爽

  2018-07-01

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Right now

  关于ARP响应有3个疑问： 1.回复ARP的机器 就是 ARP请求包里面的目标IP的机器吗？还是说只有ARP缓存中有对应IP地址的机器都可以回复? 2.ARP收到ARP响应之后，是否需要广播一个包，说我已经拿到目标IP机器的Mac地址呢？ 3.ARP收到ARP响应，会不会同时收到很多个ARP响应包，这个时候要怎么处理?

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/01/1b/24db90c7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Cola

  通过是否有IP来判别有无操作系统吗？

  2018-06-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/27/d5/0fd21753.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一粟

  PXE客户端和PXEServer分别在局域网里的两个子网里还能实现系统安装的功能么？

  2018-06-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/27/d5/0fd21753.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一粟

  PXE客户端和PXEServer分别在局域网里的两个子网里还能实现，系统安装的功能么?

  2018-06-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/7b/9e/37d69ff0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  balancer

  老师，发送dhcp 请求的时候，由于没有ip 都是0.0.0.0 ，那么dhcp 服务器怎么将数据发给dhcp 客户端的呢，是通过mac 地址识别么？

  作者回复: 是的

  2018-06-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/12/cddd766c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大树

  linux系统上静态路由的设置是否也需要，系统上有ip地址和目标地址或网络，在同一网段呢？谢谢

  2018-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/12/cddd766c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大树

  linux系统上静态路由的设置是否也需要，系统上有ip地址和目标地址或网络，在同一网段呢？谢谢

  2018-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/12/cddd766c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大树

  刘超，老师好，您提到：如果将网关配置为 192.168.1.6 呢Linux 不会让你配置成功的，因为网关要和当前的网络至少一个网卡是同一个网段的。我是否可以理解为这个是网关配置的限制，如果linux上配置静态路由的话，是否就可以配置成功了呢？谢谢

  作者回复: 你可以试一下，应该会报错

  2018-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ff/ce/73ee54bf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Linux云计算网络

  第一个问题想的是配完第一次之后，在dhcp的配置文件中把next-server地址去掉，这样就不会执行pxe的请求流程，不知是否正确

  2018-06-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/61/62/72296b09.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小雨

  这节课应该结合下第二节的分层，毕竟子网掩码和ip是一块儿配置的。

  2018-06-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/61/62/72296b09.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小雨

  所使用ip可以不和网关一个网段吧，企业级路由器可以支持多个网段，当然只有一个路由地址，且同一层级。

  2018-06-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8d/a6/22c37c91.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  楊_宵夜

  寥寥几句，尽显深厚的功力

  2018-06-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a7/ec/db6dd54c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Star_Trek_

  老师好，有个疑惑的地方，16.158.23.6把ping包发给网关，网关转发给192.168.1.6，所以ping包应该能到达目的吧？

  2018-06-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/8a/98/2be9d17b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  破晓^_^

  刘老师你好，我想问一下，平常电脑采用 dhcp 方式租用 IP 时，租约到50%时进行续约，为什么往往续约得到的 IP 和之前相同，这是 dhcp 的租用策略吗？谢谢

  作者回复: 既然是续约，当然不搬家了啊

  2018-06-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/24/3f9f7c70.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zixuan

  前面的章节提到网关也是ARP吼出来的，那是不是不配网关也可以呢？

  作者回复: 不配网关的ip，吼的时候吼什么呢？

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9d/34/177bf988.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kaka.Song

  下一个dhcp的环境中，经常出现   一个ip分给了A，过一段时间这个ip又分给了B.但是A的地址还是原先分配的地址，A还是dhcp状态，出这个问题会是什么原因？

  2018-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/be/8c/a5a3788f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大灰狼

  dhcp怎么解决内网地址的信任问题

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c8/75/654bd899.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  海狸

  讲的很形象，愿意看下去，我是名程序员对这些网络协议传输的东西一直稀里糊涂，看了您的文章解惑了

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/c8/75cd38b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  宣宣

  服务器IP是怎么固定不变的呢？是手动分配了之后，给办公区dhcp的网段不包括服务器IP吗？

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/c8/75cd38b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  宣宣

  Pxe server 就是 tftp server 吗？

  作者回复: 不是，那个ip地址的机器上要搭建一个tftp服务器

  2018-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/3c/b7091379.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  还能再肝的biubiubiu

  老师讲的挺好的，提一个建议，能不能把一些术语的英文也给出来。。比如CIDR这样的，翻译内容可能会有点偏差

  作者回复: 好的，这个以后加上原英文

  2018-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/24/3f9f7c70.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zixuan

  Pxe客户端在没有os网络栈和驱动的条件下竟然能做成这么多事

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/86/e3/28d1330a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fsj

  DHCP OFFER mac头有误吧，应该是新人的mac地址，不应该是广播地址

  作者回复: 其实广播，单播都是可以的

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8e/c7/3ad139d6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黑翼天佑

  最终租约达成的时候，还是需要广播一下，让大家都知道。 这里是谁去广播呢？是客户机还是网关？

  2018-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/60/bc/c6ba84bd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ant

  动态IP更新时间是多少？也就是IP租用时间

  2018-05-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/46/9c/c0226990.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jk.鴻

  那能不能通过NAT模式来配置不同网段的ip？在本机中通过在vmnet8来配置跟本机不同网段的ip和网关，然后在虚拟机中就能设置vmnet中的网段嘛？两个不同网段能ping的通嘛

  2018-05-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/61/5d/908ecfa5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lh

  刘哥，同一个网段内获取目标主机的mac地址用ARP，里面包含了目标主机的IP，目标主机发现是自己的IP就回复自己的mac地址。那么跨网段呢，目标主机所在的网段内所有机器，它们的IP都是相同的，那么用IP没法找到目标主机了吧？如果这样，那是通过什么找到目标主机呢？端口号吗

  作者回复: 跨网段要到网关

  2018-05-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/09/de/55cc6c2f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大树

  看了无数介绍DHCP和PXE的，还是这文章透彻。刘哥威武。

  作者回复: 谢谢

  2018-05-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f5/c2/6f895099.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  cooruo

  讲的不错，以前做过相关工作，没有搞明白。

  作者回复: 谢谢

  2018-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/80/bd6419bb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  化雨

  试着回答下第一题。安装完操作系统后将这一信息(是否安装)写入bios.以后重启时pxe首先从bios读取是否已安装系统，如果已经安装，直接向dhcp请求分配ip而无需请求启动文件。

  2018-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5a/73/d8db600f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  F.Y.

  装好系统后再重新配置DHCP server?

  作者回复: 服务端可以修改，但不是改dhcpserver

  2018-05-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4d/dc/87809ad2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  埃罗芒阿老师

  安装完系统之后，机器需要关闭pxe功能的吧，不然每次都会走一遍流程？关了应该就正常启动了吧

  作者回复: 是的，可以关闭网络启动方式，也可以在服务端修改

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8c/45/e3e2dfbf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  glennut

  讲的很清楚！ 虽然我自己搭了PXE给项目组里用了很久了，但一直没了解到这里面的逻辑和内容竟然还有这么多。😀

  作者回复: 谢谢

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/df/97d1dfc5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈亮

  老师，请问PEX的协议数据解析部分是自己实现的吗？还是用了内核的协议解析？

  作者回复: 那个时候还没有内核

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/df/97d1dfc5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈亮

  老师，请问PXE客户端是否是自己实现的协议解析体系，类似于内核的协议栈解析方式？还是直接把内核体系拿过来了？谢谢。

  作者回复: pxe是在BIOS里的，至于是不是和内核一样，这个需要研究一下了

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/33/92/99530cee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灰飞灰猪不会灰飞.烟灭

  啥意思啊，就是说指定IP地址，在不同网段是没办法通讯的 对吧？老师

  作者回复: 要过路由器就可以

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5f/71/3997eb14.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  扬帆起航

  我在机房可以把其他服务器第一块盘作为启动盘，硬盘上面没有操作系统直接去找dhcp，安装完成之后，重启应该直接从硬盘启动了。

  作者回复: 怎么拿其他服务器的盘做第一块盘呢？盘怎么映射过来呢？启动的时候如何知道那台服务器的地址呢？

  2018-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ba/01/5ce8ce0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Leoorz

  这一讲很赞👍   了解了很多东西

  2018-05-25
