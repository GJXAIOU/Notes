# 第18讲 \| DNS协议：网络世界的地址簿

作者: 刘超

完成时间:

总结时间:



<audio><source src="https://static001.geekbang.org/resource/audio/c5/34/c56a67b3e6e791b6677d9d327b3fd834.mp3" type="audio/mpeg"></audio>

前面我们讲了平时常见的看新闻、支付、直播、下载等场景，现在网站的数目非常多，常用的网站就有二三十个，如果全部用IP地址进行访问，恐怕很难记住。于是，就需要一个地址簿，根据名称，就可以查看具体的地址。

例如，我要去西湖边的“外婆家”，这就是名称，然后通过地址簿，查看到底是哪条路多少号。

## DNS服务器

在网络世界，也是这样的。你肯定记得住网站的名称，但是很难记住网站的IP地址，因而也需要一个地址簿，就是**DNS服务器**。

由此可见，DNS在日常生活中多么重要。每个人上网，都需要访问它，但是同时，这对它来讲也是非常大的挑战。一旦它出了故障，整个互联网都将瘫痪。另外，上网的人分布在全世界各地，如果大家都去同一个地方访问某一台服务器，时延将会非常大。因而，**DNS服务器，一定要设置成高可用、高并发和分布式的**。

于是，就有了这样**树状的层次结构**。

![](<https://static001.geekbang.org/resource/image/89/4d/890ff98fde625c6a60fb71yy22d8184d.jpg?wh=1512*984>)\- 根DNS服务器 ：返回顶级域DNS服务器的IP地址

- 顶级域DNS服务器：返回权威DNS服务器的IP地址

- 权威DNS服务器 ：返回相应主机的IP地址


<!-- -->

## DNS解析流程

为了提高DNS的解析性能，很多网络都会就近部署DNS缓存服务器。于是，就有了以下的DNS解析流程。

1. 电脑客户端会发出一个DNS请求，问www.163.com的IP是啥啊，并发给本地域名服务器 (本地DNS)。那本地域名服务器 (本地DNS) 是什么呢？如果是通过DHCP配置，本地DNS由你的网络服务商（ISP），如电信、移动等自动分配，它通常就在你网络服务商的某个机房。

2. 本地DNS收到来自客户端的请求。你可以想象这台服务器上缓存了一张域名与之对应IP地址的大表格。如果能找到 www.163.com，它就直接返回IP地址。如果没有，本地DNS会去问它的根域名服务器：“老大，能告诉我www.163.com的IP地址吗？”根域名服务器是最高层次的，全球共有13套。它不直接用于域名解析，但能指明一条道路。

3. 根DNS收到来自本地DNS的请求，发现后缀是 .com，说：“哦，www.163.com啊，这个域名是由.com区域管理，我给你它的顶级域名服务器的地址，你去问问它吧。”

4. 本地DNS转向问顶级域名服务器：“老二，你能告诉我www.163.com的IP地址吗？”顶级域名服务器就是大名鼎鼎的比如 .com、.net、 .org这些一级域名，它负责管理二级域名，比如 163.com，所以它能提供一条更清晰的方向。

5. 顶级域名服务器说：“我给你负责 www.163.com 区域的权威DNS服务器的地址，你去问它应该能问到。”

6. 本地DNS转向问权威DNS服务器：“您好，www.163.com 对应的IP是啥呀？”163.com的权威DNS服务器，它是域名解析结果的原出处。为啥叫权威呢？就是我的域名我做主。

7. 权威DNS服务器查询后将对应的IP地址X.X.X.X告诉本地DNS。

8. 本地DNS再将IP地址返回客户端，客户端和目标建立连接。


<!-- -->

<!-- [[[read_end]]] -->

至此，我们完成了DNS的解析过程。现在总结一下，整个过程我画成了一个图。

![](<https://static001.geekbang.org/resource/image/71/e8/718e3a1a1a7927302b6a0f836409e8e8.jpg?wh=1456*1212>)

## 负载均衡

站在客户端角度，这是一次**DNS递归查询过程。**因为本地DNS全权为它效劳，它只要坐等结果即可。在这个过程中，DNS除了可以通过名称映射为IP地址，它还可以做另外一件事，就是**负载均衡**。

还是以访问“外婆家”为例，还是我们开头的“外婆家”，但是，它可能有很多地址，因为它在杭州可以有很多家。所以，如果一个人想去吃“外婆家”，他可以就近找一家店，而不用大家都去同一家，这就是负载均衡。

DNS首先可以做**内部负载均衡**。

例如，一个应用要访问数据库，在这个应用里面应该配置这个数据库的IP地址，还是应该配置这个数据库的域名呢？显然应该配置域名，因为一旦这个数据库，因为某种原因，换到了另外一台机器上，而如果有多个应用都配置了这台数据库的话，一换IP地址，就需要将这些应用全部修改一遍。但是如果配置了域名，则只要在DNS服务器里，将域名映射为新的IP地址，这个工作就完成了，大大简化了运维。

在这个基础上，我们可以再进一步。例如，某个应用要访问另外一个应用，如果配置另外一个应用的IP地址，那么这个访问就是一对一的。但是当被访问的应用撑不住的时候，我们其实可以部署多个。但是，访问它的应用，如何在多个之间进行负载均衡？只要配置成为域名就可以了。在域名解析的时候，我们只要配置策略，这次返回第一个IP，下次返回第二个IP，就可以实现负载均衡了。

另外一个更加重要的是，DNS还可以做**全局负载均衡**。

为了保证我们的应用高可用，往往会部署在多个机房，每个地方都会有自己的IP地址。当用户访问某个域名的时候，这个IP地址可以轮询访问多个数据中心。如果一个数据中心因为某种原因挂了，只要在DNS服务器里面，将这个数据中心对应的IP地址删除，就可以实现一定的高可用。

另外，我们肯定希望北京的用户访问北京的数据中心，上海的用户访问上海的数据中心，这样，客户体验就会非常好，访问速度就会超快。这就是全局负载均衡的概念。

## 示例：DNS访问数据中心中对象存储上的静态资源

我们通过DNS访问数据中心中对象存储上的静态资源为例，看一看整个过程。

假设全国有多个数据中心，托管在多个运营商，每个数据中心三个可用区（Available Zone）。对象存储通过跨可用区部署，实现高可用性。在每个数据中心中，都至少部署两个内部负载均衡器，内部负载均衡器后面对接多个对象存储的前置服务器（Proxy-server）。

![](<https://static001.geekbang.org/resource/image/0b/b6/0b241afef775a1c942c5728364b302b6.jpg?wh=2200*2262>)

1. 当一个客户端要访问object.yourcompany.com的时候，需要将域名转换为IP地址进行访问，所以它要请求本地DNS解析器。

2. 本地DNS解析器先查看看本地的缓存是否有这个记录。如果有则直接使用，因为上面的过程太复杂了，如果每次都要递归解析，就太麻烦了。

3. 如果本地无缓存，则需要请求本地的DNS服务器。

4. 本地的DNS服务器一般部署在你的数据中心或者你所在的运营商的网络中，本地DNS服务器也需要看本地是否有缓存，如果有则返回，因为它也不想把上面的递归过程再走一遍。

5. 至 7. 如果本地没有，本地DNS才需要递归地从根DNS服务器，查到.com的顶级域名服务器，最终查到 yourcompany.com 的权威DNS服务器，给本地DNS服务器，权威DNS服务器按说会返回真实要访问的IP地址。


<!-- -->

对于不需要做全局负载均衡的简单应用来讲，yourcompany.com的权威DNS服务器可以直接将 object.yourcompany.com这个域名解析为一个或者多个IP地址，然后客户端可以通过多个IP地址，进行简单的轮询，实现简单的负载均衡。

但是对于复杂的应用，尤其是跨地域跨运营商的大型应用，则需要更加复杂的全局负载均衡机制，因而需要专门的设备或者服务器来做这件事情，这就是**全局负载均衡器**（**GSLB**，**Global Server Load Balance**）。

在yourcompany.com的DNS服务器中，一般是通过配置CNAME的方式，给 object.yourcompany.com起一个别名，例如 object.vip.yourcomany.com，然后告诉本地DNS服务器，让它请求GSLB解析这个域名，GSLB就可以在解析这个域名的过程中，通过自己的策略实现负载均衡。

图中画了两层的GSLB，是因为分运营商和地域。我们希望不同运营商的客户，可以访问相同运营商机房中的资源，这样不跨运营商访问，有利于提高吞吐量，减少时延。

1. 第一层GSLB，通过查看请求它的本地DNS服务器所在的运营商，就知道用户所在的运营商。假设是移动，通过CNAME的方式，通过另一个别名 object.yd.yourcompany.com，告诉本地DNS服务器去请求第二层的GSLB。

2. 第二层GSLB，通过查看请求它的本地DNS服务器所在的地址，就知道用户所在的地理位置，然后将距离用户位置比较近的Region里面，六个**内部负载均衡**（**SLB**，S**erver Load Balancer**）的地址，返回给本地DNS服务器。

3. 本地DNS服务器将结果返回给本地DNS解析器。

4. 本地DNS解析器将结果缓存后，返回给客户端。

5. 客户端开始访问属于相同运营商的距离较近的Region 1中的对象存储，当然客户端得到了六个IP地址，它可以通过负载均衡的方式，随机或者轮询选择一个可用区进行访问。对象存储一般会有三个备份，从而可以实现对存储读写的负载均衡。


<!-- -->

## 小结

好了，这节内容就到这里了，我们来总结一下：

- DNS是网络世界的地址簿，可以通过域名查地址，因为域名服务器是按照树状结构组织的，因而域名查找是使用递归的方法，并通过缓存的方式增强性能；

- 在域名和IP的映射过程中，给了应用基于域名做负载均衡的机会，可以是简单的负载均衡，也可以根据地址和运营商做全局的负载均衡。


<!-- -->

最后，给你留两个思考题：

1. 全局负载均衡为什么要分地址和运营商呢？

2. 全局负载均衡使用过程中，常常遇到失灵的情况，你知道具体有哪些情况吗？对应应该怎么来解决呢？


<!-- -->

我们的专栏更新过半了，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送\*\*<span class="orange">学习奖励礼券</span>

**和我整理的**<span class="orange">独家网络协议知识图谱</span>

\*\*。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(136)

- ![img](%E7%AC%AC18%E8%AE%B2%20_%20DNS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF.resource/resize,m_fill,h_34,w_34.jpeg)

  iiwoai

  图谱应该都给赠送才对啊

  2018-06-27

  **3

  **303

- ![img](https://static001.geekbang.org/account/avatar/00/11/21/0f/b61dc67e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ^_^

  老师，独家网络协议知识图谱，这么好的东西，应该阳光普照，哈哈

  2018-06-27

  **1

  **125

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6a/a8/ee414a25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  公告-SRE运维实践

  分地址和运营商主要是为了返回最优的ip，也就是离用户最近的ip，提高用户访问的速度，分运营商也是返回最快的一条路径。gslb失灵一般是因为一个ns请求gslb的时候，看不到用户真实的ip，从而不一定是最优的，而且这个返回的结果可能给一个用户或者一万个用户，可以通过流量监测来缓解。

  作者回复: 对的

  2018-08-19

  **6

  **91

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/fa/6fb4e123.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  coldfire

  从真名到别名怎么完成均衡负载，感觉没讲清楚，对于小白来说这就是一笔带过留了个影响

  2018-06-27

  **3

  **40

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f2/46/09c457eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Garwen

  刘超老师，根域名服务器的访问不还是无法逃离高并发访问的场景吗，虽说本地的DNS已经可以解决80%请求的地址解析，但还是会有极其庞大数量的访问需要去找根域名服务器的吧，这里的根域名服务器的高并发场景不比12306小吧，是如何解决的呢？

  作者回复: 一方面，dns要求有缓存。另一方面，根服务器没有12306业务逻辑这么复杂，什么各种高铁段的计算啊，而且涉及事务，两个人不能同一个位置，涉及支付。根服务器基本只需要维护下一级别的就可以了，而下一个级别的相对稳定，而且都是静态的。所以支撑过双十一我们就知道，凡是静态的，我有一万种方式可以搞定，就怕落数据库

  2019-05-05

  **3

  **31

- ![img](https://static001.geekbang.org/account/avatar/00/11/5a/06/ca4cd46e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  知识改变命运

  感谢刘老师的讲解！这几次课的理解程度好多了！话说不是不愿意回答思考题，而是小白无力回答~最后，同求知识图谱~多谢多谢

  2018-06-27

  **

  **21

- ![img](https://static001.geekbang.org/account/avatar/00/0f/43/c1/576c01a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  upstream

  问题1，分成两级还是在于国内网络跨运营商访问慢 问题2，如果用户是上海电信，用户本地dns不是用的电信dns服务器，负载均衡调度会不准确。针对app的可以httpdns 针对网站的除非能支持edns协议才比较好。 另外当用户使用阿里云这类anycast 地址时，不清楚gslb调度策略如何的

  2018-06-27

  **

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/0f/43/c1/576c01a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  upstream

  虽然有了域名，但对于大多数人而言，能记住的就那么一两个网站地址。于是，打开baidu.com 搜索想要打开的网站，也就不奇怪了。打开baidu.com 搜索谷歌，再去访问Google 也就成为很多人的默认行为。说白了，也还是记不住域名。 Baidu.com 搜索qq邮箱的大有人在

  2018-06-27

  **3

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/12/67/61/ea54229a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  jim

  浏览器，路由器也有缓存吧？

  作者回复: 是的

  2018-11-28

  **2

  **13

- ![img](https://static001.geekbang.org/account/avatar/00/16/88/18/a88cdaf5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  alexgzh

  以前只知道DNS是方便記憶。今天學到了DNS其實是解耦應用和被訪問應用的IP地址，還有局部負載均衡和全局負載均衡的功能, 學到了！！

  作者回复: 对的，负载均衡

  2019-07-20

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/14/78/ac/e5e6e7f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古夜

  这一课看懂了，还联想到了ping百度域名有时候返回ip不一样的问题，感谢

  作者回复: 对的

  2019-04-19

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/11/e2/58/8c8897c8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨春鹏

  当访问网站的时候，是先去访问host文件还是，本地DNS缓存？

  作者回复: host

  2018-09-12

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/18/51/d2/dddffa12.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  胸怀宇宙瓜皮李

  确实不是很理解为什么需要两次的全局负载均衡？为什么不在第一次的负载均衡中，直接获取DNS服务器所在地址和DNS服务提供商？

  作者回复: 不是所有的人都要两层

  2019-07-14

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/10/27/d5/0fd21753.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一粟

  要解析本地DNS缓存，需要启动DNS Sclient服务，此服务不开启hosts配置会失效。

  2018-07-15

  **1

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7f/0c/392ce255.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  极客时间

  引用原文中的话： “在 yourcompany.com 的 DNS 服务器中，一般是通过配置 CNAME 的方式，给 object.yourcompany.com 起一个别名，例如 object.vip.yourcomany.com，然后告诉本地 DNS 服务器，让它请求 GSLB 解析这个域名，GSLB 就可以在解析这个域名的过程中，通过自己的策略实 现负载均衡” 这里有一句没明白，告诉本地服务器，让object.vip.yourcomany.com域名去GSLB解析，这个应该如何告诉呢

  作者回复: 返回给他就可以了。

  2019-06-17

  **6

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/9e/80/b87f8b49.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LongXiaJun

  😃😃😃现在一周三天都在期待专栏更新~

  2018-06-27

  **

  **5

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  起风了001

  好像设置GSLB是通过添加NS记录实现的, 不是CNAME呀.

  作者回复: 通过添加NS也是可以的

  2019-05-22

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/12/ec/2a/b11d5ad8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  曾经瘦过

  报名太晚了 不知道是否还可以获得独家网络知识图谱和学习奖励券

  作者回复: 图谱最后一篇里面有啊

  2018-10-18

  **3

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/27/f5/6e/55ec1ed1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  C++大雄

  文中有一处不严谨，DNS分为迭代查询和递归查询，图三中5-9的查询应该是迭代，3是递归

  2021-08-07

  **

  **3

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/pTD8nS0SsORKiaRD3wB0NK9Bpd0wFnPWtYLPfBRBhvZ68iaJErMlM2NNSeEibwQfY7GReILSIYZXfT9o8iaicibcyw3g/132)

  雷刚

  ISO 七层协议中每一层都有自己的寻址方式：应用层用过域名（主机名）寻址，网络层通过 IP 寻址，链路层通过MAC 寻址。这样每一层都是独立的，但不同层之间交流时就需要进行地址转换： 1. DNS 协议：将域名转换为 IP 地址。 2. ARP 协议：将 IP 转换为 MAC 地址。

  2020-04-10

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/42/52/19553613.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘培培

  \1. 不同地区和运营商之间延迟不同，所以优先选择同运营商、同地区服务 2. GLSB 挂掉了，客户端地址、运营商变更了，缓存未及时更新。

  作者回复: 赞

  2019-08-27

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得：1.DNS区域名称系统 也为域名系统，它为网络世界的地址薄，可以通过域名查地址，因为域名服务是按树状结构组织，域名查询是使用递归方法查询，缓存可以增强反应速度，提供性能 ;2.域名通过DNS查询IP，DNS给予应用基于域名做负载均衡，可以是简单的负载均衡也可以是基于地址和运营商做全局负载均衡。 回答老师的问题1.全局负载均衡 分地址，为返回数据提供最快的速度，分运营商 这有跨网络，寻找路线 不是最短或者最优的，所以 就近 在同一个运营商 较快。2.全局负载均衡 失灵，DNS没有及时更新，所以域名与IP对应不上，得不到反馈，不能正常访问，有不对之处 请指正。

  2019-02-23

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/0f/84/ae/2b8192e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  selboo

  在递归的时候。返回的授权ns如果链接不上。dns服务器怎么处理的？ 比如dns服务器收到 www.163.com 解析请求 递归返回ns记录为 ns1.163.com 和ns2.163.com 但此时 ns1.163.com 网络不通。 此时dns服务器是一直等待？还是重试几次返回失败？还是其他？

  2018-07-09

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/40/ba/2c8af305.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_zz

  dns 解析过程7步骤里有个错字，是权威，而不是权限

  2018-10-26

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fc/34/c733b116.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  何磊

  我对dns解析后，把所有IP都返回给客户端，由客户端负责轮训达到负载均衡表示不解。这里不应该是dns服务器返回一个随机IP，然后客户端直接访问即可吗？

  2018-09-26

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/20/5d/69170b96.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  灰灰

  写得太好了，刘超老师，你还有其他技术专栏吗？或者有其他推荐的技术专栏吗？

  2018-06-27

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  今天刚看到的。 在过去 30 年里，美国利用先发优势，在根服务器治理体系中占据主导地位。在 IPv4（互联网协议第四版）体系内，全球被限制为总共 13 台根服务器，唯一主根部署在美国，其余 12 台辅根服务器有 9 台在美国，2 台在欧洲，1 台在日本——我们都是互联网的客人。

  2019-12-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/08/92f42622.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  尔冬橙

  为什么本地dns不直接访问顶级dns

  作者回复: 仅仅顶级的放不下，压力也太大

  2019-08-12

  **2

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d6/53/f5b4d2f0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  亮哥

  “在 yourcompany.com 的 DNS 服务器中，一般是通过配置 CNAME 的方式，给 object.yourcompany.com 起一个别”    请问一下老师这里说的DNS服务器是指域名解析里的配置吗？像阿里云买的域名里的：云解析DNS/域名解析/解析设置，记录类型选择CNAME，记录值是：object.vip.yourcomany.com，另外这个域名是第一层 GSLB的域名吗？

  作者回复: 对的，就是云解析里面的

  2019-08-07

  **

  **1

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/wYPdGBibR1FJWWMzFzYBy4tNTd22rMgtlYTgR7wuicFLQjiaozuRWM2VSMFxPYjVdkLJLGvMav2icH3icgz07hmDsDw/132)

  dayspring

  老师，有一个问题请教一下本地dns，是怎么知道本地dns server的ip地址的？同理本地dns server是怎么知道根dns server的ip地址的？他们之间的访问有负载均衡吗？

  作者回复: 就像我们本地配置dns一样的。有负载均衡的

  2019-05-13

  **2

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/5f/e8/94365887.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ieo

  刘老师，dns服务器请求权威dns后，怎么请求到全局负载均衡服务器的，这块没怎么看懂，是权威dns告诉本地dns的难道是全局负载均衡的ip吗？

  作者回复: 是的

  2019-05-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/1f/3f/c9f7e73c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  风中拾遗

  老师，这个域名绑定的ip应该是公网的ip吧？不然会发生冲突，但如果是公网的ip，那个数就那么多，总有一天会不够用的。

  作者回复: 是的，所以有ipv6呀

  2019-03-25

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d3/b4/bb883fc8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  pana@

  本地DNS服务器在请求object.vip.yourcompany.com这个域名时，是不是也要走一遍dns解析流程才能到gslb?gslb还能解析域名？应该只能根据域名通过某些策略返回IP吧?

  作者回复: 通过别名到glsb，是可以解析域名的

  2019-01-17

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/b8/fb19aa6a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Marnie

  DNS域名和IP是一对多的关系，一个域名，查找到多个IP地址，根据一定策略选择不同IP，从而实现负载均衡。高可用也与多IP有关，当一个IP有问题时，将其从DNS地址簿移除，本机可以访问其他IP.

  2018-11-26

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e8/d9/76efdd8d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wlh2119

  刘老师的每一篇文章都认真的学习了，对上层HTTP到底层协议都有了更深的理解与认识，建立一个粗略而完整的协议框架。还要继续深入学习。谢谢。

  2018-07-04

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/17/e3/37a68985.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tony

  本地dns解析器是在客户端本机上面吗？

  2018-07-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/20/36/b3e2f1d5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wykkx

  全局负载均衡使用过程中，常常遇到失灵的情况，你知道具体有哪些情况吗？对应应该怎么来解决呢？ 全局负载均衡失灵的时间，可以分情况来应对， 1，全局负载均衡器因为流量过大，而导致的失灵， 此情况，是因为流量已经超过了当前机器的极限所导致的，针对此只能通过扩容来解决。 2，全局负载均衡器因为机器故障了，导致的失灵。 此情况的发生，说明机器存在负载均衡器有单点问题，须通过增加备机，或者更为可靠的集群来解决。 3，全局负载均衡器因为网络故障，导致的失灵。 此情况，案例莫过于支付宝的光纤被挖掘机挖断，此问题可通过接入更多的线路来解决，  我能想到的只有这些，还请超哥来指证，谢谢

  2018-07-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/bc/13de31c5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  羽毛

  “我们希望不同运营商的客户，可以访问相同运营商机房中的资源”为什么不同运营商访问相同的运营商机房中的资源

  2018-06-27

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/28/cb6d8768.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jobs

  1.防止单个地区或运营商的流量过爆而挂了。 2.服务器故障，网络故障如中间路由器故障，SLB设备故障，断电故障等。Solution:返回多重A记录？

  2018-06-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/29/f7/49/791d0f5e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  UDP少年

  知识有限，回答不出精彩的内容，但学习的心态还是积极向上的，希望老师能给大家都分享下知识图谱

  2022-08-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/e8/e2/2bcaef68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王瑞强

  rpc服务就近访问也是这个原理么

  2022-08-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/27/af/29/3c27174a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  浪客剑心

  请教，本地DNS解析器和本地DNS服务器的区别是什么？本地DNS解析器的“位置”是在本地主机上吗？

  2022-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/fe/88/9ad320e3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Geek_ea3932

  关于根 DNS服务器，是不是每个国家自己都维护一个的呢？

  2022-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b9/3b/7224f3b8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  janey

  刘老师，通过这一课对dns的理解加深了负载均衡的部分。但是有个问题不太理解： 示例：DNS 访问数据中心中对象存储上的静态资源 这里看到有2个“本地DNS服务器”，理解的比较模糊，模糊的点如下： 1.这2个“本地DNS服务器”是一种类主从的关系还是地位等同的2个服务器？客户端在访问的时候按什么机制访问这2台服务器？是一定要在2台都查找过而不会只查一台，查询确认没有了才做后面的5，6，...这些步骤吗？ 2.这2台“本地DNS服务器”是部署在哪里的呢？客户端的内网？客户端同主机还是啥？ 可能在使用中没遇到过这样的场景，比较迷糊 希望能得到解答，感谢~

  2022-02-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7d/56/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张涛

  有两个小细节不太准确，1. “每个数据中心三个可用区”这里，明白老师的意思，但说法不准确，可用区往上一层是Region而非数据中心；2.对象存储本身就是Region级别的概念，不需要再跨可用区部署了

  2022-02-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0c/46/dfe32cf4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多选参数

  引用原文“在 yourcompany.com 的 DNS 服务器中，一般是通过配置 CNAME 的方式，给 object.yourcompany.com 起一个别名，例如 object.vip.yourcomany.com，然后告诉本地 DNS 服务器，让它请求 GSLB 解析这个域名，GSLB 就可以在解析这个域名的过程中，通过自己的策略实现负载均衡。” 那么这里“让它请求 GSLB 解析这个域名”中的这个域名到底是 object.vip.yourcomany.com 这个域名还是，yourcomany.com 这个域名。

  2021-10-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/0c/46/dfe32cf4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多选参数

  引用原文“在 yourcompany.com 的 DNS 服务器中，一般是通过配置 CNAME 的方式，给 object.yourcompany.com 起一个别名，例如 object.vip.yourcomany.com，然后告诉本地 DNS 服务器，让它请求 GSLB 解析这个域名，GSLB 就可以在解析这个域名的过程中，通过自己的策略实现负载均衡。”

  2021-10-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/34/cf/0a316b48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  蝴蝶

  地址相近但是运营商可能不同＞所以需要两层负载均衡

  2021-10-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7e/1f/b1d458a9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iamjohnnyzhuang

  1、全局负载均衡为什么要分地址和运营商呢？ 给出链路最优的边缘节点。最典型的应用就是在 cdn 场景上， 比如我们在上海和新疆同时获取一个静态图片。 服务器可能只部署在上海，如果没走全局负载均衡都来我们这请求的话，新疆的由于地域问题（运营商可能也不一样）效果会很差。 然后我们会考虑接入 cdn 厂商， 在我们的权威服务器这里 cname 到 cdn 厂商给我们的域名， cdn 厂商做全局负载均衡，按照调度算法（通常就是 运营商+空间距离）匹配最优节点。  2、全局负载均衡使用过程中，常常遇到失灵的情况，你知道具体有哪些情况吗？对应应该怎么来解决呢？ 这也算传统 cdn 的硬伤了， 因为这一切都依赖于本地 dns。 如果用户自己配置了 dns 地址，并且配置的 dns 服务器，区域、运营商都不一致，那么全局负载均衡则无法给出最优的调度。 有一种解决就是使用 httpdns

  2021-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/8f/13/50454a94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程惠玲

  基于别名记录的负载均衡方，一个域名指向多个其他域名，CNAME肯定去全局负载均衡器获取吗，负载均衡策略在哪配置呢？

  2021-05-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/8d/fa/93e48f3b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ℡ __ 柚稚

  dns本地都有缓存，那如果域名映射ip变化啦，怎么解决缓存一致性

  2021-05-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/8d/d2/8a6be8d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员班吉

  第一个问题，假设不分地址和运营商的话，那么很有肯能北京的用户访问到了上海的IP，联通的用户被当做移动的用户，这样会导致用户访问了离自己比较远的资源。分地址和运营商的好处是，可以让用户访问到最近的资源，有更好的用户体验。 第二个问题，负载均衡失灵可能是是因为用户在上海访问了一个网页，然后立马飞到北京，此时如果用户再去访问之前的网站有可能使用的还是在上海访问时的dns本地缓存。

  2020-12-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/06/e4/9375269a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  依法治国

  我觉得GSLB最大的一个问题是:原本的目的是实现数据中心的容灾切换，但是DNS默认有缓存机制，所以一个数据中心的某个AZ挂了，用户还是会根据缓存去访问一个挂掉的服务

  2020-11-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/fa/e2/178bc954.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赵小通

  老师本地DNS服务器是指运营商的吗？

  2020-10-29

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/C1YjuX47p9whXQVvR7n39xQ0UHYaj1hx6QAd703ulKhuoJhg4ZHLU2qkcQdBPxNjtlztrKQYWibywV8pibmoVpag/132)

  learner

  感觉举的DNS可以做内部负载均衡和全局负载均衡没有什么区别啊，都是轮询策略，都是部署多个，有什么区别吗？

  2020-10-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/35/90/fdb1421e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我相信指数爆炸

  需要补充 httpDNS

  2020-09-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/38/2d/f3c6493e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  罗樱罂

  谢谢刘老师的课程。我想提一个小小的建议（这个应该是私信吧？）要是可以每个专有名词都标注英文名字就更好了。比如根dns（root dns）， 顶级dns （top level dns TLD）之类的。谢谢老师，老师辛苦了！

  2020-08-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/75/00/618b20da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  徐海浪

  1 首先是就近访问，其次是避免跨运营商。 2 DNS解析有时效问题。

  作者回复: 是的

  2020-06-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/4a/34/1faac99b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夕林语

  思考题： 1.分地址和运营商是为了让用户有更快的访问速度和更好的体验 2.浏览器的缓存或者本地dns的缓存

  作者回复: 是的

  2020-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/be/05/55b72986.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  shaukin

  从权威域名服务器返回给本地DNS服务器不是返回IP地址了吗，那怎么还要去返回GLSB实现负载均衡呢

  作者回复: 返回ip地址是常规场景，还可以转到glsb

  2020-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/4b/a276c1d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  儘管如此世界依然美麗

  问题一：分运营商可以让相同运营商的用户访问的相同运营商机房中所对应的资源，减少时延 分地址可以让用户就近访问资源，同样也可以减少时延

  2020-05-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  1.必然是分层的概念,首先利用不同的运营商进行了筛选和优化,然后利用地区,进行了最后的优化,提高到了不能再提高的部分 2.像是之前说的NAT网关,会在传递的时候进行变化,这就导致了可能在最后到达了GSLB的时候,运营商解析错误了

  2020-05-07

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/tKvmZ3Vs4t6RZ3X7cAliaW47Zatxhn1aV5PcCYT9NZ9k9WWqRrEBGHicGtRWvsG6yQqHnaWw6cGNSbicNLjZebcHA/132)

  柳十三

  全局负载均衡分地址可能是就近原则，最近访问最快；分运营商可能是每个运营商速度不一样 全局负载均衡失灵猜测可能解析完换ip,解决思路冲原来ip跳转过去

  2020-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  在平常的生活中上网，差不多都是属于域名去访问一些网站来获取所需的资源。而DNS就是互联网世界的地址簿，大大提升了大家上网的体验和便捷度，同时考虑到高可用和低延时等问题，DNS通过递归查询和缓存来提升查询的速度，另外也通过内部负载均衡和全局负载均衡来达到一样的目的。

  2020-03-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e4/a1/178387da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  25ma

  老师，这个dns访问根服务器的时候是如何进行连接的，采用的协议是什么？

  2020-03-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/37/56/11068390.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  倡印

  如果本地dns没有地址，就要访问根dns感觉好麻烦

  2020-02-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/11/78/4f0cd172.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  妥协

  在域名解析的时候，我们只要配置策略，这次返回第一个 IP，下次返回第二个 IP，就可以实现负载均衡了。 如上所述，客户端第一次请求到IP后，会缓存本地，不会再第二次请求了。这个负载均衡策略是指不同客户端访问，返回不同的地址吗？ 有讲到dns返回一个域名的多个地址，一个客户端自己轮训多个地址。 这两种方案都可以吗？

  2019-11-06

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIjUDIRQ0gRiciax3Wo78c5rVjuWDiaw4ibcCiby8xiaMXJh5ibjU5242vfCGOK4ehibe1IKyxex2A4IX4XSA/132)

  追风者

  当一个权限域名服务器还不能直接给出最后的查询结果时，就会告诉发出查询的DNS客户，下一步应当找哪一个权限域名服务器。 老师，应该如何理解这句话？一次DNS解析中，本地服务器会向权限域名服务器发送多次请求？我在书上看到说查询www.abc.xyz.com本地DNS会发送4次查询请求。困扰已久！

  作者回复: com->xyz->abc->www

  2019-08-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/54/26/d3997877.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Ronnie

  如果GSLB 是基于DNS protocol的话，那么它基于CNAME 配置的话，我感觉有点不对，是不是应该基于NS记录来配置的。 CNAME实现的GSLB 是现在很多cloud provider 提供的基于4/7层协议的。

  2019-07-07

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eoRyUPicEMqGsbsMicHPuvwM8nibfgK8Yt0AibAGUmnic7rLF4zUZ4dBj4ialYz54fOD6sURKwuJIWBNjhg/132)

  咸鱼与果汁

  请问DNS域名IP印象变更，对应dns服务器是如何做数据同步的？

  2019-06-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/12/f9/7e6e3ac6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_04e22a

  DNS解析流程：本地缓存—本地DNS服务器—根服务器

  2019-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/e9/29/629d9bb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天王

  1 DNS协议，相当于地址簿，将域名转换成ip的协议，2 DNS服务器包括DNS缓存服务器和DNS服务器，DNS服务器包括根DNS服务器，顶级域名服务器，权威DNS服务器，树状结构 3 客户端请求的过程 客户端请求域名，3.1 访问www.163.com首先去DNS缓存服务器找，如果有，一般在网络服务商的某个机房，返回ip地址，没有继续往后找 3.2 去根域名服务器找，返回顶级域名的ip .com的ip，3.3 去顶级域名服务器找，返回权威域名服务器的ip 3.4 权威域名服务器返回www.163.com 的ip 4 DNS做负载均衡 给客户端都是域名，如果加减服务器，修改域名和ip的映射关系，客户端地址不需要修改，负载均衡能分流，轮询机制，将不同的请求依次轮询转发到不同的ip上。这是内部负载均衡。全局负载均衡，多个数据中心，一个数据中心挂了，剔除掉，另外各个地区访问各地的数据中心，提高访问速度。5 全局负载均衡器 GSLB全称 全局负载均衡器，一个数据中心可以设置多个可用区，可用区后面可以设置2个内部负载均衡器，后面接存储对象的前置服务器，GSLB一般是通过在权威DNS服务器配置一个别名，该别名去向GSLB请求，实现负载均衡，GSLB分2层，运营商和地域，不跨运营商访问可以提高吞吐量，减小时延 ，客户端请求的流程5.1 第一层CGLB通过请求查看本地dns服务器所在的运营商，确定运营商， 通过cname找到别名，去找第2层的GSLB 5.2第二层的CGLB通过请求的本地DNS服务器找到用户的地理位置，找到距离用户比较近的数据中心，数据中心内部通过内部的负载均衡器，返回内部的多个负责均衡器的地址 5.3 本地DSN服务器拿到地址之后，返回到本地DNS解析器，本地DNS解析器缓存之后，返回给客户端 5.4 客户端拿到6个ip之后，随机或者轮询进行访问 。

  2019-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b4/94/2796de72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的人

  缓存DNS缓存->根DNS服务器-->顶级域DNS服务器（.com/.cn.net 等）-->权威DNS服务器

  2019-06-05

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  陈耀

  关于客户端的dns的查询有部分是错误的，浏览器到本地dns（甚至运营商dns）使用的是递归查询，而向根域服务器、顶级域服务器、权威域服务器 使用的是迭代查询。

  作者回复: 你说的对，但是和我说的一样呀

  2019-04-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b1/23/5df1f341.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  且听疯吟

  2.全局负载策略失败的原因： 1.负载均衡策略失效，比如某个时间段或者某个地区发生了DNS网络攻击直接会导致负载均衡策略失效。 2.区域的问题会造成实际的负载均衡出现问题，举个例子，比如淘宝双11活动，不发达城市的区域实际上访问量较小，服务器基本上轻松无压力，但是发达地区由于人员访问连接过大，但是负载均衡仍然是优先指定本地的服务器，这时会造成发达地区的服务器负载压力非常大，但是人员稀少的区域服务器时间上还有非常大的服务空间。实际的负载均衡策略这时应该需要按需进行调整。

  2019-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b1/23/5df1f341.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  且听疯吟

  1、实际使用过程中不同的运营商的连接速度 与 带宽的问题需要考虑负载均衡。比如实际使用过程中运营商A 租用了100M的带宽，运营商B租用了200M的带宽，运营商C租用了300M的带宽，这时的负载均衡按照1:2:3的比例来均衡，再次考虑到不同运营商之间出入口速率的问题，比如客户端要访问的地址经过查看和比对发现该地址的服务器属于运营商A的，则这时需要尽量将该报文送往运营商A的出口，防止跨运营商访问。

  2019-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ad/d1/62376eff.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Zoe

  老师，如果遇到dns网络劫持了怎么解决。

  2019-03-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/d0/f1/432e0476.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  X

  xxxx could not be resolved (110: Operation timed out) 我在一个项目中要使用http协议去请求其他N个域名，每次请求都会有DNS解析这一步。刚开始时，DNS解析正常，没有超时，但是过一会，DNS解析超时越来越多。单独拿出服务中解析超时的域名，使用linux命令dig，全部正常，ping，也全部正常。我的nginx.conf配置中，resolver_timeout 60s（默认30s）,resolver 8.8.8.8。请问是哪里的问题？我需要怎么去排查？

  作者回复: 是不是还配置了别的dns server，那个先超时了。

  2019-03-01

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/9a/69/d86bafe1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  挖下水道的

  DNS污染的问题应该讲一讲

  2019-02-28

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/04/71/0b949a4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  何用

  为何根域名服务器只有13套？

  2019-02-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6d/1c/10d5c280.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  哈哈

  有个问题想问下，如果权威域名服务器没有分多个运营商的话，从权威域名服务器dns解析到gslb过程的不是会很慢么？ 还是说从根开始每个都是gslb这种模式，或者还是权威域名服务器都部署在主干网所以很快

  作者回复: 不是，只有权威之下才是gslb。

  2019-02-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/f9/90/f90903e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜菜

  老师，通过负载均衡，客户端拿到假如说6和ip，通过轮询的话，如果是tcp连接的话，每次都要新建一个tcp连接吗？这样会不会又增加负担……

  作者回复: 如果要做负载均衡，就需要六个tcp

  2019-02-09

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/13/46/f678d54f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  &#47; _在路上。

  老师能不能讲一下子域未授权的问题，例如在政府网中总部和省级的关系，省级二级域上配了一个独有的子域转发，按理在省内请求该子域应由省级域直接相应请求，但是为何响应会返回为空，此场景的dns请求过程是怎样的？

  2019-01-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/52/fe/df362d52.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不纯有机物

  老师您好，我在网上搜索的结果是dns解析顺序先是本地dns缓存>hosts文件>dns服务器，但是我看您在评论里说的是hosts文件在本地dns缓存之前

  2018-12-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/76/23/31e5e984.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  空知

  打卡,滴滴,看一半了！

  2018-12-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/88/c2/c8159ba7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yifan.L

  老师，最近在接触负载均衡，能否分享一些关于负载均衡的知识点。

  2018-12-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5d/a2/c6fb8fb8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  狗蛋儿

  您好 两层GLSB的例子中 第四步“本地 DNS 解析器将结果缓存后，返回给客户端。” 本地DNS解析器会把6个IP都返回给客户端？还有下次访问同样域名时候，假设客户端缓存失败，本地DNS解析器缓存，也是返回6个IP吗？

  2018-12-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/e0/99/5d603697.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  MJ

  本地DNS服务器访问GLSB的时候，如何做到把原域名改为别名来访问呢？

  作者回复: 在dns服务器上可以配置别名

  2018-11-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/7d/370b0d4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  西部世界

  分运营商是因为，跨运营商访问速度相当慢，分地区当然是选择就近机房下的运营商响应更快。

  2018-11-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/6f/63/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  扬～

  疑问，为什么要把6个proxy的IP都发给客户端呢?这样不会侵入代码吗，这不应该是服务端的责任吗，不需要对客户端透明吗?

  作者回复: 可以透明，比如客户端是一个普通应用，也可以在客户端做，例如对象存储的sdk，需要做一些自己的决策

  2018-11-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/68/5d/1ccee378.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  茫农

  感觉域名和ip有点像ip和mac，

  2018-10-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bf/97/14626661.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lewis

  分运营商，不同的运营商切换需要的链路更长。假设用户在移动，切换到联通的过程:首先数据包到达移动的服务器节点，再由移动节点转发到联通节点？这个理解对吗？

  2018-10-24

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Tom

  本地DNS不是本地局域网的吧？本机io配置中的DNS就是本地局域网吧？

  作者回复: 不是本地局域网

  2018-10-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/bb/85/191eea69.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  搬铁少年ai

  一般来讲，公司内部组网的客户端并不会配置dns服务器的地址，因为网络变化可能导致ip地址的变动，即使有dhcp分配dns服务器地址。取而代之的是，在中间设备，比如接入路由器，防火墙等设备上配置dns代理，而客户端的dns server地址指向dns代理的地址，由dns代理负责转发dns请求报文到服务器，同时，dns代理可以在本地维护缓存表，提高查询效率，减少发到服务器的查询报文，降低服务器压力。另外，还可以在网络设备上配置dns欺骗，用于用户认证的重定向等

  2018-10-13

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJJJdibLfaPjOqvgdRSn11JhwdqaI2KiaUdkeA9cwQ5btpnFX00V0rpFXlksKfBtaArprs7DN2ybFug/132)

  逍遥夏末

  刘老师，dns服务器是由运营商维护吗，也就是服务器所有者维护，GSLB是由我们需要访问的网址服务器维护吗，比如访问163.com，是不是网易给运营商一个规则，说来访问我的服务器都给我指定到我的GSLB服务器上来，我自己做全局负载均衡，我来返回用户最优访问的ip。请刘老师指正。

  作者回复: 使用别名

  2018-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/56/aa/6c60f057.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  江龍

  负载均衡方式有好多种，可以专门写一个专题文章，比如nginx单向代理，dns方式，在数据链路层做，在ip层做都可以

  2018-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/e2/58/8c8897c8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  杨春鹏

  当访问网站的时候，是先去访问host文件还是，本地DNS缓存？

  2018-09-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/5b/2a/a6628985.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Windshell

  刘老师能不能再仔细讲下DNS类型，例如A，CNAME等

  2018-08-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/56/93/fb3a6e19.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黄加菜

  理解的还不够透彻，还需多看几遍。

  2018-08-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/40/05/749bafae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  凯子哥

  讲的挺好，思路清晰

  2018-08-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/54/cf/fddcf843.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  芋头

  老师，有一个问题不明白，IP是缓存在DNS上的，那域名对应的IP改变了，岂不是访问不到了？

  2018-08-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/71/96/66ad1cd2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  飞

  本地配置一个假的，难怪也解析呢，原来先从自己本地缓存开始解析

  2018-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/74/3d/54bbc1df.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Jaime

  问题一，分区域，分运营商是因为要有更好的用户体验，减少响应时间和增加吞吐量。 问题二，对全局负载均衡还不是很理解，等理解了在回来回答

  2018-07-25

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/IM2JvRB8ibNWib6gc3Wr2QPPX1DxwVerxbvdJHRQ1a4n3WQfoQdfqjda9ib8ylbWP4IX63thsj0ApxLcePGHeiaEIw/132)

  snowqiang

  服务端的IP更换了，或者部署新机器了应该会有影响

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/24/e1cb609e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  破晓

  问题1，跨网络运营商访问慢。 问题2，网络运营商有自己的dns缓存服务器，导致全局失效。

  2018-07-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4e/08/d497c158.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  coderliang

  请教一个问题，个人网站，大陆和国外如何做全局负载均衡。国外有一台服务器，国内有一台服务器。能不能给指一个方案，或者方向？感谢！

  2018-07-12

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  米兰的小铁匠

  负载均衡分地区好理解，就近访问，速度肯定快； 分运营商可能是兼容性问题，以及不同运营商在不同地区的带宽，资源不一样

  2018-07-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/45/be/494010aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  zcpromising

  老师，dns解析为啥没讲迭代解析呢

  作者回复: 讲了呀

  2018-07-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/80/b5/f59d92f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Cloud

  运营商之间切换，会有网络抖动 机器故障，网络故障，负载过高

  2018-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/17/e3/37a68985.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tony

  请问本地解析器是在本机，还是在运营商服务器↑？谢谢

  2018-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/1b/ac/41ec8c80.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  孙悟空

  \1. 为了低时延以及减少跨运营商的带宽和费用 2.后端ip变化，后端故障

  2018-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/4b/cd/185e5378.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  泊浮目

  问题1我想是因为不同的运营商通信成本会上去吧，就像不同的地区一样，要多走几次路由。 问题2暂时能想到的问题是因为流量过大导致机器宕机，解决方案是加机器做高可用集群或者使用限流算法。

  2018-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/20/36/b3e2f1d5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wykkx

  全局负载均衡使用过程中，常常遇到失灵的情况，你知道具体有哪些情况吗？对应应该怎么来解决呢？ 全局负载均衡失灵的时间，可以分情况来应对， 1，全局负载均衡器因为流量过大，而导致的失灵， 此情况，是因为流量已经超过了当前机器的极限所导致的，针对此只能通过扩容来解决。 2，全局负载均衡器因为机器故障了，导致的失灵。 此情况的发生，说明机器存在负载均衡器有单点问题，须通过增加备机，或者更为可靠的集群来解决。 3，全局负载均衡器因为网络故障，导致的失灵。 此情况，案例莫过于支付宝的光纤被挖掘机挖断，此问题可通过接入更多的线路来解决，  我能想到的只有这些，还请超哥来指证，谢谢

  2018-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/20/36/b3e2f1d5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wykkx

  第一问题是为了让用户就近访问和使用和自己运营商一样的线路访问，提升用户体验。 第二个问题，有几个可能原因一是dns没有配置健康检查机制，后端服务ip已经挂了或者更换了。二是受到了类似ddos攻击，超过了dns服务的解析能力。 对于第一类问题，需要增加dns健康检查。对于第二类需要增加dns的安全模块。 求老师的知识图谱，多谢！

  2018-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/90/167a13fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  徐良红

  第二种情况是不是用户设置了通用dns例如谷歌的，这样就无从判断是哪个运营商来的，失灵了

  2018-06-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/43/90/167a13fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  徐良红

  问题1的原因主要是运营商之间互联互通不畅

  2018-06-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/49/e2/1fad12eb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张洪阆

  1.按地址和运营商划分考虑的是距离和带宽 2.不清楚

  2018-06-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/de/17/75e2b624.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  feifei

  全局负载均衡为什么要分地址和运营商呢？  全局负载均衡GSLB，实现在广域网(包括互联网)上不同地域的服务器间的流量调配，保证使用最佳的服务器服务离自己最近的客户，从而确保访问质量。 分地址，我觉得是为了保证高可用，防止单点故障。 分运营商，这个我觉得可能就是中国特色了，用一句最调侃的话，世界上最远的距离，莫过于你的电信，我在联通。  全局负载均衡使用过程中，常常遇到失灵的情况，你知道具体有哪些情况吗？对应应该怎么来解决呢？ 全局负载均衡失灵的时间，可以分情况来应对， 1，全局负载均衡器因为流量过大，而导致的失灵， 此情况，是因为流量已经超过了当前机器的极限所导致的，针对此只能通过扩容来解决。 2，全局负载均衡器因为机器故障了，导致的失灵。 此情况的发生，说明机器存在负载均衡器有单点问题，须通过增加备机，或者更为可靠的集群来解决。 3，全局负载均衡器因为网络故障，导致的失灵。 此情况，案例莫过于支付宝的光纤被挖掘机挖断，此问题可通过接入更多的线路来解决，  我能想到的只有这些，还请超哥来指证，谢谢

  2018-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/1d/8e/0a546871.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  凡凡

  1.分地址和运营商都是为了就近服务，较少中间网络延时和不确定，不稳定性，提升访问性能。2.失灵情况猜测是不是由于ip到区域的映射不是总那么准确，或者说精度太低，一般只能定位到省级？

  2018-06-28

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIWSTcia43udFKpGW0YT4zDJpO7LqkCicmTIgiaFcfszibGDkjfSSwqQ7pIMQl1Mhxc8PsdLS1UrnsrRA/132)

  方圆佰李

  一个网站下的每个页面都有地址，这些地址都有对应IP吗？

  2018-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/66/b9/673f42c0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李梦游

  负载均衡失灵的问题可能是IP地址失效或更换之类的吧

  2018-06-28

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  唐唐

  刘老师，文中的本地DNS的缓存，是指用户所在的网络服务器地址上的host文件吧，用户自己的host配置了某些域名，ip对应关系的话，会直接用吗？

  2018-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5e/4c/f87a9b60.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赵艳飞

  小白一枚，只能听懂面上的东西～希望可以多学点东西

  2018-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5d/16/9a57266b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘宝明

  老师，讲一下公司内网的请求数据包是怎么发到公网的，响应数据包又是怎么返回到局域网的

  2018-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/43/79/18073134.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  test

  1.跨运营商访问慢

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/27/0b/12dee5ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  进化论

  分运营商，是不是解决跨运营商访问慢的问题啊！ 第二个是因为有缓存？

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/39/486faadf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  浪子恒心

  还是没太想明白为什么分了两层全局负载，一层负载不能全解决吗？全局负载一般在用户配置了公共DNS服务时会失效吧，通过HTTPDNS能解决，但移动端需要配合做改造，难度比较大。

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ef/a2/6ea5bb9e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  LEON

  超哥，目前我国DNS SEC可以使用了吗？最早听说我们运营商不支持。

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ef/a2/6ea5bb9e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  LEON

  超哥，请教个问题，GSLB在判断站点故障后切换A记录到可用站点(GSLB上的A记录配置的TTL30秒)。但是部分用户并没有切换成功，是因为GSLB提供的A记录的TTL让部分运营商的local dns 改成了12个小时，导致用户本地TTL缓存长，必须清缓存换local dns。用户端的local dns配置不可控制。这种情况下除了不使用HTTP DNS改造外，有什么更好的办法？如果使用EDNS不知道目前支持情况怎么样？谢谢

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/2f/25/d2162fce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  龚极客

  问题1，因为机房的网络对应的运营商速度不同 问题2，如果某一个节点不可达，需要弹性伸缩，数据中心master 和slave的延迟也是个问题。

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/07/f2/156a220f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  叮当

  同求知识图谱阳光普照

  2018-06-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  free

  1 跨运营商慢的问题 2 gslb自身的高可用

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/b6/74/c63449b1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  问题究竟系边度

  分地址和运营商应该是使用适用的网络提高访问效率。 这种分配方式只是直接就近分配。并没有根据对应链路质量进行监测和调整。 是否应该部署两个gslb预防单点故障和网络攻击？

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e6/d9/4cc39ed1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Kail

  1.CDN? 2.DNS 劫持？HTTPDNS

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a6/c1/d4e6147e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  一箭中的

  思考题2:当域名对应的ip地址更换了，那么dns缓存内容就失效了，此时会导致无法访问到该网址。所以dns缓存都会超时时间，超时后缓存就会失效。

  2018-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/ce/a8c8b5e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jason

  每篇都能学到新知识。谢超哥。

  2018-06-27

  **

  **
