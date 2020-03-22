---
tags: 
- web
style:  summer
---


## Web页面请求的历程



## **场景与网络环境**

一名学生Bob启动他的计算机，然后将其用一根以太网电缆连接到学校的以太网交换机，交换机又与学校的路由器相连，然后通过浏览器访问Google主页。学校的路由器与一个ISP连接，本例中ISP为comcast.net本例中comcast.net为学校提供了DNS服务，所以**DNS服务器驻留在Comcast网络中**而不是学校网络中。

![](https://mmbiz.qpic.cn/mmbiz_png/88IXhNkSic6fY2WYCZclia2F8dHGamhTtubPtaDOvDtn8mt3oVCOfldtp7KibmDXHTjJ17oXghibxhhkR4Hibt5LFag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

接下来我们分析一下Bob的计算机如何下载一个Web页面（www.google.com主页）

![](https://mmbiz.qpic.cn/mmbiz_png/88IXhNkSic6fY2WYCZclia2F8dHGamhTtu8yqOcwvcOia70UMgHsNV3kTiaMhx4lG1hegOufL7ygHw5jct1VcCvnibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 一、**准备：DHCP、UDP、IP和以太网**

Bob的计算机目前还没有IP地址，如何动态获取IP地址？运行动态主机配置协议（Dynamic Host Configuration，DHCP），以从本地的DHCP服务器获取一个IP地址以及其他信息。

- DHCP服务器发现

在UDP分组中向端口67发送一个DHCP发现报文（DHCP discover message）。但这个UDP数据报发送给谁呢？此时主机不知道它所连接网络的IP地址，更不用说用于该网路的DHCP服务器地址了。在这种情况下，DHCP客户生成包含DHCP发现报文的IP数据报，**目的地址为广播IP地址255.255.255.255，源地址为0.0.0.0。该IP数据报传递给链路层，链路层然后将该帧广播到所有该子网连接的子网。**

- DHCP服务器提供

DHCP服务器收到一个DHCP发现报文时，用一个DHCP提供报文（DHCP offer message）向客户作出响应，仍然使用**IP广播地址255.255.255.255**。 为什么这个服务器必须采用广播？（也因为目前的客户端还没有IP地址）在子网中可能有几个DHCP服务器，该客户也许会发现它所处于能在几个提供者之间进行选择的优越位置。
每台服务器提供的**报文包含有收到的发现报文的事务ID、向客户推荐的IP地址（yiaddr，yourinternet address）、网络掩码以及IP地址租用期（address lease time），即IP地址有效时间**。

**事务ID作用**：客户端生成一个随机数事务ID，并记录下来，然后把它插入到xid字段。客户端发送DHCP DISCOVER广播报文。服务器从DHCP DISCOVER消息解析得到xid值，把xid值插入到DHCP OFFER消息的xid字段，发送DHCP OFFER报文到请求客户端。如果DHCP OFFER消息中的xid值与最近发送DHCP DISCOVER消息中的xid值不同，那么客户端必须忽略这个DHCP OFFER。接收到的任何DHCPACK必须悄悄地丢弃。


- DHCP请求

客户主机从一个或多个服务器提供中选择一个，并向选中的服务器提供报文用DHCP请求报文（DHCP requestmessage）进行响应，回显配置参数。

- DHCP ACK

服务器用DHCP ACK报文（DHCP ACK message）对DHCP请求报文进行响应，证实所要求的参数。

Bob计算机上的操作系统生成一个DHCP请求报文，并将这个报文放入 **目的地端口67（DHCP服务器）和源端口68（DHCP客户)** 的UDP报文段。该UDP报文段则被放置在一个具有广播IP目的地地址（255.255.255.255）和源IP地址0.0.0.0的IP数据报中，因为此时Bob的计算机还不具有一个IP地址。

该IP数据报则被放置在以太网帧中，以太网帧的目的MAC地址FF:FF:FF:FF:FF:FF，使该帧将广播到与交换机连接的所有设备，源MAC地址为Bob计算机的网卡MAC地址00:16:D3:23:68:8A。

协议层	|数据
---|---
应用层|	DHCP请求报文
传输层	|UDP  源端口68   目的端口67
网络层 |	源IP 0.0.0.0    目的IP 255.255.255.255
链路层 |	源MAC 00:16:D3:23:68:8A    目的MAC FF:FF:FF:FF:FF:FF


**包含DHCP请求的广播以太网帧是第一个由Bob计算机发送到以太网交换机的帧**，该帧中包含DHCP请求。该交换机在所有的出端口广播帧，包括连接到路由器的端口。同时交换机会在交换机表（switch table）中新添加一条记录，内容包括MAC地址，通往该MAC地址的交换机接口，该记录放置在表中的时间。

| 地址 | 接口 | 时间 |
| --- | --- | --- |
| 00-16-D3-23-68-8A | 1 | 9:30 |

路由器在它的接口（该接口MAC地址00:22:6B:45:1F:1B）接收到该广播以太网帧，该帧包含DHCP请求，并且从该以太网帧中抽取出IP数据报。该数据报的广播IP目的地址指示了这个IP数据报应当由在该结点的高层协议处理，因此该数据报的载荷（一个UDP报文段）被分解向上到达UDP，从此UDP报文段中抽取出DCHP请求报文。

我们假设运行在路由器中的DHCP服务器能够以CIDR(无类别域间路由选择)块68.85.2.0/24分配IP地址，所以Bob计算机地址在Comcast地址块中，然后分配地址68.85.2.101给Bob计算机。DHCP服务器生成一个DHCP ACK报文，包含内容IP地址DNS服务器IP地址（68.87.71.226）默认网关路由器（第一跳路由）IP地址 68.85.2.1和子网块（网络掩码） 68.85.2.0/24。
该DHCP报文被放入一个UDP报文段中,UDP报文段被放入一个IP数据报中,IP数据报再被放入一个以太网帧中.一个以太网帧的源MAC地址是路由器连接到归属网络时接口的MAC地址(00:22:6B:45:1F:1B),目的MAC地址是Bob计算机的MAC地址(00:16:D3:23:68:8A).

| DHCP ACK报文 | 值 |
| --- | --- |
| IP地址 | 68.85.2.101 |
| DNS服务器IP地址 | 68.87.71.226 |
| 默认网关路由器（第一跳路由）IP地址 | 68.85.2.1 |
| 子网块（网络掩码） | 68.85.2.0/24 |

包含DHCP ACK报文的以太网帧由路由器发送给交换机。因为交换机是自学习的，并且先前从Bob计算机收到（包含DHCP请求的）以太网帧，所以该交换机从交换机表中查询到通往Bob计算机MAC地址00:22:6B:45:1F:1B的相应接口。

Bob计算机接收到包含DHCP ACk的以太网帧，从该以太网帧中抽取IP数据报，从IP数据报中抽取UDP报文段，从UDP报文段中抽取DHCP ACK报文。Bob的计算机DHCP客户则记录下它的IP地址和它的DNS服务器的IP地址。它还在其IP转发表中安装默认网关的地址。

如果Bob计算机想向除子网68.85.2.0/24之外的目的地址发送数据报，则先要经过默认网关。

- 如果没有找到DHCP服务器，则客户机会从保留的B类网段169.254.0.0中挑选一个IP地址作为自己的IP地址，子网为16。然后通过客户机将挑选的IP地址通过ARP广播，确定自己挑选的IP地址是否已经被网络上的其他设备所采用，如果已经被使用，则重新挑选，最多获取10次，直到配置成功。之后每隔5分钟尝试与DHCP服务器进行通信，一旦取得通信则放弃自动配置的IP地址，使用服务器分配的IP地址和其他配置信息

![分界]($resource/%E5%88%86%E7%95%8C.png)

**仍在准备：DNS和ARP**

当Bob将www.google.com的URL键入其Web浏览器时，他开启了一长串事件，这导致Google主页最终显示在其Web浏览器上。Bob的Web浏览器通过生成一个TCP套接字开始了该过程，套接字用于向www.google.com发送HTTP请求。
为了生成套接字，①Bob的计算机机需要知道www.google.com的IP地址。这就需要DNS协议提供这种域名到IP地址的转换服务。Bob计算机上的操作系统生成一个DNS查询报文，将字符串www.google.com放入DNS报文的问题段中。该DNS报文则放置在一个具有**53号（DNS服务器**）目的端口的UDP报文段中。该UDP报文段则被放入具有IP目的地址68.87.71.226（在DHCP ACK返回的DNS服务器地址）和源地址68.85.2.101的IP数据报中。
②Bob的计算机则将包含DNS请求报文的数据报放入一个以太网帧中。该帧将发送到（在链路层寻址）Bob学校网络中的网关路由器。目前仅从DHCP ACK报文知道学校网关路由器的IP地址65.85.2.1，但不知道网关路由器的MAC地址。此时就需要ARP协议提供IP地址到MAC地址的转换服务。

ARP

Bob计算机生成一个具有目的IP地址68.85.2.1（默认网关）的ARP查询报文，将该ARP报文放置在一个具有广播目的地址FF:FF:FF:FF:FF:FF的以太网帧中，并向交换机发送该以太网帧，交换机将该帧交付给所有连接的设备，包括网关路由器。


ARP只为同一个子网上的主机和路由器接口解析IP地址。DNS为在因特网中任何的主机解析为IP地址。

域	|值
---|---
硬件类型	|1表示以太网
协议类型	|发送者所提供/请求的高级协议地址类型 ;;     0x0800代表IP协议
发送者IP地址	|68.85.2.101
发送者MAC地址|	00:16:D3:23:68:8A
目的IP地址|	68.85.2.1
目的MAC地址|	FF:FF:FF:FF:FF:FF

网关路由器在通往学校网络的接口上接收到包含该ARP查询报文的帧，发现在ARP报文中的目标地址68.85.2.1与其接口IP地址相同，网关路由器此时将准备一个ARP回答，指示IP地址68.85.2.1对应的MAC地址为00:22:6B:45:1F:1B。它将ARP回答放在一个以太网帧中，其目的地址为00:16:D3:23:68:8A（Bob的计算机），并向交换机发送该帧，再由交换机将帧交付给Bob的计算机。Bob计算机收到包含ARP回到报文的帧，并从ARP回答报文中抽取网关路由器的MAC地址（00:22:6B:45:1F:1B）。并在本地ARP表中创建一条新的记录。
| IP地址 | MAC地址 | TTL(Time to Live) |
| --- | --- | --- |
| 68.85.2.1 | 00-22-6B-45-1F-1B | 09:30:00 |

现在，Bob的计算机能够使包含DNS查询报文的以太网帧寻址到网关路由器的MAC地址了。

DNS

网关路由器接收该帧并抽取包含DNS查询的IP数据报。路由器查找该数据报的目的地址（68.87.71.226），并根据其转发表决定该数据报应道发送到Comcast网络中最左边的路由器。

Comcast最左边的路由器接到该帧，抽取IP数据报，检查该数据报的目的地址（68.87.71.226），并根据其转发表确定接口，经过该接口朝着DNS服务器转发数据报，而转发表已根据Comcast的域内协议（如RIP、OSPF或IS-IS）以及因特网的域间协议BGP所填写。

最终包含DNS查询的IP数据报到达了DNS服务器。DNS服务器抽取出DNS查询报文，在它的DNS数据库中查找域名www.google.com，找到包含对应www.google.com的IP地址（64.233.169.105）的DNS源记录。该DNS服务器形成了一个包含这种主机名到IP地址映射的DNS回答报文，将该DNS回答报文放入UDP报文段中，该报文段放入寻址到Bob计算机的IP数据报中。该数据报将通过Comcast网络反向转发到学校的路由器，并从这里经过以太网交换机到Bob计算机。Bob计算机从DNS报文抽取出服务器www.google.com的IP地址。经过大量的工作后，Bob的计算机终于可以与www.google.com服务器通信了。

![](https://mmbiz.qpic.cn/mmbiz_png/88IXhNkSic6fY2WYCZclia2F8dHGamhTtu8yqOcwvcOia70UMgHsNV3kTiaMhx4lG1hegOufL7ygHw5jct1VcCvnibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**Web客户——服务器交互：TCP和HTTP**

Bob计算机有了www.google.com的IP地址，它就能够生成TCP套接字，该套接字将用于向www.google.com发送HTTP GET报文。当Bob生成TCP套接字时，在Bob计算机中的TCP必须首先与www.google.com中的TCP执行三次握手。Bob计算机首先生成一个具有目的端口80（针对HTTP）的TCP SYN报文段，将该TCP报文段放置在具有目的IP地址64.233.169.105（www.google.com）的IP数据报中，将该数据报放置在MAC地址为00:22:6B:45:1F:1B（网关路由器）的帧中，并向交换机发送该帧。

在学校网络、Comcast网络和Google网络中的路由器朝着www.google.com转发包含TCP SYN的数据报，使用每台路由器中的转发表。

最终，包含TCP SYN的数据报到达www.google.com。Google服务器从数据报抽取出TCP SYN报文并分解到与端口80相联系的套接字。对于Google HTTP服务器和Bob计算机之间的TCP连接生成一个连接套接字。产生一个TCP SYNACK报文段（SYNACK segment），将其放入一个向Bob计算机寻址的数据报中。包含TCP SYNACK报文段的数据报经过Google、Comcast和学校网络，最终到达Bob计算机的以太网卡。Bob的Web浏览器生成HTTP GET报文。HTTP GET报文则写入套接字，其中GET报文成为一个TCP报文段的载荷，该TCP报文段则被放进一个数据报中，并交付到www.google.com。
在www.google.com的HTTP服务器从TCP套接字读取HTTP GET报文，生成一个HTTP响应报文，将请求的Web页面内容放入HTTP响应体重，并将报文发送进TCP套接字中。包含HTTP响应报文的数据报经过Google、Comcast和学校网络转发，到达Bob计算机。Bob的Web浏览器从套接字读取HTTP响应，从HTTP响应体中抽取Web网页的HTML，最终显示出Web网页。