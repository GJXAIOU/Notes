# 第19讲 \| HttpDNS：网络世界的地址簿也会指错路

作者: 刘超

完成时间:

总结时间:



<audio><source src="https://static001.geekbang.org/resource/audio/c5/8e/c5878b08f79096cdad80d9f9e656818e.mp3" type="audio/mpeg"></audio>

上一节我们知道了DNS的两项功能，第一是根据名称查到具体的地址，另外一个是可以针对多个地址做负载均衡，而且可以在多个地址中选择一个距离你近的地方访问。

然而有时候这个地址簿也经常给你指错路，明明距离你500米就有个吃饭的地方，非要把你推荐到5公里外。为什么会出现这样的情况呢？

还记得吗？当我们发出请求解析DNS的时候，首先，会先连接到运营商本地的DNS服务器，由这个服务器帮我们去整棵DNS树上进行解析，然后将解析的结果返回给客户端。但是本地的DNS服务器，作为一个本地导游，往往有自己的“小心思”。

## 传统DNS存在哪些问题？

### 1\.域名缓存问题

它可以在本地做一个缓存，也就是说，不是每一个请求，它都会去访问权威DNS服务器，而是访问过一次就把结果缓存到自己本地，当其他人来问的时候，直接就返回这个缓存数据。

这就相当于导游去过一个饭店，自己脑子记住了地址，当有一个游客问的时候，他就凭记忆回答了，不用再去查地址簿。这样经常存在的一个问题是，人家那个饭店明明都已经搬了，结果作为导游，他并没有刷新这个缓存，结果你辛辛苦苦到了这个地点，发现饭店已经变成了服装店，你是不是会非常失望？

另外，有的运营商会把一些静态页面，缓存到本运营商的服务器内，这样用户请求的时候，就不用跨运营商进行访问，这样既加快了速度，也减少了运营商之间流量计算的成本。在域名解析的时候，不会将用户导向真正的网站，而是指向这个缓存的服务器。

<!-- [[[read_end]]] -->

很多情况下是看不出问题的，但是当页面更新，用户会访问到老的页面，问题就出来了。例如，你听说一个餐馆推出了一个新菜，你想去尝一下。结果导游告诉你，在这里吃也是一样的。有的游客会觉得没问题，但是对于想尝试新菜的人来说，如果导游说带你去，但其实并没有吃到新菜，你是不是也会非常失望呢？

再就是本地的缓存，往往使得全局负载均衡失败，因为上次进行缓存的时候，缓存中的地址不一定是这次访问离客户最近的地方，如果把这个地址返回给客户，那肯定就会绕远路。

就像上一次客户要吃西湖醋鱼的事，导游知道西湖边有一家，因为当时游客就在西湖边，可是，下一次客户在灵隐寺，想吃西湖醋鱼的时候，导游还指向西湖边的那一家，那这就绕得太远了。

![](<https://static001.geekbang.org/resource/image/8b/eb/8b5e670e22c459e859293db1bed643eb.jpg?wh=1383*965>)

### 2\.域名转发问题

缓存问题还是说本地域名解析服务，还是会去权威DNS服务器中查找，只不过不是每次都要查找。可以说这还是大导游、大中介。还有一些小导游、小中介，有了请求之后，直接转发给其他运营商去做解析，自己只是外包了出去。

这样的问题是，如果是A运营商的客户，访问自己运营商的DNS服务器，如果A运营商去权威DNS服务器查询的话，权威DNS服务器知道你是A运营商的，就返回给一个部署在A运营商的网站地址，这样针对相同运营商的访问，速度就会快很多。

但是A运营商偷懒，将解析的请求转发给B运营商，B运营商去权威DNS服务器查询的话，权威服务器会误认为，你是B运营商的，那就返回给你一个在B运营商的网站地址吧，结果客户的每次访问都要跨运营商，速度就会很慢。

![](<https://static001.geekbang.org/resource/image/cf/f1/cfde4b3bc5c0aabaaa81f3a26cd99cf1.jpg?wh=1645*1055>)

### 3\.出口NAT问题

前面讲述网关的时候，我们知道，出口的时候，很多机房都会配置**NAT**，也即**网络地址转换**，使得从这个网关出去的包，都换成新的IP地址，当然请求返回的时候，在这个网关，再将IP地址转换回去，所以对于访问来说是没有任何问题。

但是一旦做了网络地址的转换，权威的DNS服务器，就没办法通过这个地址，来判断客户到底是来自哪个运营商，而且极有可能因为转换过后的地址，误判运营商，导致跨运营商的访问。

### 4\.域名更新问题

本地DNS服务器是由不同地区、不同运营商独立部署的。对域名解析缓存的处理上，实现策略也有区别，有的会偷懒，忽略域名解析结果的TTL时间限制，在权威DNS服务器解析变更的时候，解析结果在全网生效的周期非常漫长。但是有的时候，在DNS的切换中，场景对生效时间要求比较高。

例如双机房部署的时候，跨机房的负载均衡和容灾多使用DNS来做。当一个机房出问题之后，需要修改权威DNS，将域名指向新的IP地址，但是如果更新太慢，那很多用户都会出现访问异常。

这就像，有的导游比较勤快、敬业，时时刻刻关注酒店、餐馆、交通的变化，问他的时候，往往会得到最新情况。有的导游懒一些，8年前背的导游词就没换过，问他的时候，指的路往往就是错的。

### 5\.解析延迟问题

从上一节的DNS查询过程来看，DNS的查询过程需要递归遍历多个DNS服务器，才能获得最终的解析结果，这会带来一定的时延，甚至会解析超时。

## HttpDNS的工作模式

既然DNS解析中有这么多问题，那怎么办呢？难不成退回到直接用IP地址？这样显然不合适，所以就有了** HttpDNS**。

HttpDNS其实就是，不走传统的DNS解析，而是自己搭建基于HTTP协议的DNS服务器集群，分布在多个地点和多个运营商。当客户端需要DNS解析的时候，直接通过HTTP协议进行请求这个服务器集群，得到就近的地址。

这就相当于每家基于HTTP协议，自己实现自己的域名解析，自己做一个自己的地址簿，而不使用统一的地址簿。但是默认的域名解析都是走DNS的，因而使用HttpDNS需要绕过默认的DNS路径，就不能使用默认的客户端。使用HttpDNS的，往往是手机应用，需要在手机端嵌入支持HttpDNS的客户端SDK。

通过自己的HttpDNS服务器和自己的SDK，实现了从依赖本地导游，到自己上网查询做旅游攻略，进行自由行，爱怎么玩怎么玩。这样就能够避免依赖导游，而导游又不专业，你还不能把他怎么样的尴尬。

下面我来解析一下** HttpDNS的工作模式**。

在客户端的SDK里动态请求服务端，获取HttpDNS服务器的IP列表，缓存到本地。随着不断地解析域名，SDK也会在本地缓存DNS域名解析的结果。

当手机应用要访问一个地址的时候，首先看是否有本地的缓存，如果有就直接返回。这个缓存和本地DNS的缓存不一样的是，这个是手机应用自己做的，而非整个运营商统一做的。如何更新、何时更新，手机应用的客户端可以和服务器协调来做这件事情。

如果本地没有，就需要请求HttpDNS的服务器，在本地HttpDNS服务器的IP列表中，选择一个发出HTTP的请求，会返回一个要访问的网站的IP列表。

请求的方式是这样的。

```
curl http://106.2.xxx.xxx/d?dn=c.m.163.com
{"dns":[{"host":"c.m.163.com","ips":["223.252.199.12"],"ttl":300,"http2":0}],"client":{"ip":"106.2.81.50","line":269692944}}
```

手机客户端自然知道手机在哪个运营商、哪个地址。由于是直接的HTTP通信，HttpDNS服务器能够准确知道这些信息，因而可以做精准的全局负载均衡。

![](<https://static001.geekbang.org/resource/image/aa/75/aa45cf8a07b563ccea376f712b2e8975.jpg?wh=1581*1261>)

当然，当所有这些都不工作的时候，可以切换到传统的LocalDNS来解析，慢也比访问不到好。那HttpDNS是如何解决上面的问题的呢？

其实归结起来就是两大问题。一是解析速度和更新速度的平衡问题，二是智能调度的问题，对应的解决方案是HttpDNS的缓存设计和调度设计。

### HttpDNS的缓存设计

解析DNS过程复杂，通信次数多，对解析速度造成很大影响。为了加快解析，因而有了缓存，但是这又会产生缓存更新速度不及时的问题。最要命的是，这两个方面都掌握在别人手中，也即本地DNS服务器手中，它不会为你定制，你作为客户端干着急没办法。

而HttpDNS就是将解析速度和更新速度全部掌控在自己手中。一方面，解析的过程，不需要本地DNS服务递归的调用一大圈，一个HTTP的请求直接搞定，要实时更新的时候，马上就能起作用；另一方面为了提高解析速度，本地也有缓存，缓存是在客户端SDK维护的，过期时间、更新时间，都可以自己控制。

HttpDNS的缓存设计策略也是咱们做应用架构中常用的缓存设计模式，也即分为客户端、缓存、数据源三层。

- 对于应用架构来讲，就是应用、缓存、数据库。常见的是Tomcat、Redis、MySQL。

- 对于HttpDNS来讲，就是手机客户端、DNS缓存、HttpDNS服务器。


<!-- -->

![](<https://static001.geekbang.org/resource/image/9e/56/9e0141a2939e7c194689e990859ed456.jpg?wh=871*868>)

只要是缓存模式，就存在缓存的过期、更新、不一致的问题，解决思路也是很像的。

例如DNS缓存在内存中，也可以持久化到存储上，从而APP重启之后，能够尽快从存储中加载上次累积的经常访问的网站的解析结果，就不需要每次都全部解析一遍，再变成缓存。这有点像Redis是基于内存的缓存，但是同样提供持久化的能力，使得重启或者主备切换的时候，数据不会完全丢失。

SDK中的缓存会严格按照缓存过期时间，如果缓存没有命中，或者已经过期，而且客户端不允许使用过期的记录，则会发起一次解析，保障记录是更新的。

解析可以**同步进行**，也就是直接调用HttpDNS的接口，返回最新的记录，更新缓存；也可以**异步进行**，添加一个解析任务到后台，由后台任务调用HttpDNS的接口。

**同步更新**的**优点**是实时性好，缺点是如果有多个请求都发现过期的时候，同时会请求HttpDNS多次，其实是一种浪费。

同步更新的方式对应到应用架构中缓存的**Cache-Aside机制**，也即先读缓存，不命中读数据库，同时将结果写入缓存。

![null](<https://static001.geekbang.org/resource/image/34/b4/346a1bf30fb56ef918d708f422dc3bb4.jpg?wh=664*801>)**异步更新**的**优点**是，可以将多个请求都发现过期的情况，合并为一个对于HttpDNS的请求任务，只执行一次，减少HttpDNS的压力。同时可以在即将过期的时候，就创建一个任务进行预加载，防止过期之后再刷新，称为**预加载**。

它的**缺点**是当前请求拿到过期数据的时候，如果客户端允许使用过期数据，需要冒一次风险。如果过期的数据还能请求，就没问题；如果不能请求，则失败一次，等下次缓存更新后，再请求方能成功。

![](<https://static001.geekbang.org/resource/image/df/98/df65059e9b66c32bbbca6febf6ecb298.jpg?wh=472*826>)

异步更新的机制对应到应用架构中缓存的**Refresh-Ahead机制**，即业务仅仅访问缓存，当过期的时候定期刷新。在著名的应用缓存Guava Cache中，有个RefreshAfterWrite机制，对于并发情况下，多个缓存访问不命中从而引发并发回源的情况，可以采取只有一个请求回源的模式。在应用架构的缓存中，也常常用**数据预热**或者**预加载**的机制。

![](<https://static001.geekbang.org/resource/image/16/28/161a14f41b3547739982ec551aa26928.jpg?wh=626*770>)

### HttpDNS的调度设计

由于客户端嵌入了SDK，因而就不会因为本地DNS的各种缓存、转发、NAT，让权威DNS服务器误会客户端所在的位置和运营商，而可以拿到第一手资料。

在**客户端**，可以知道手机是哪个国家、哪个运营商、哪个省，甚至哪个市，HttpDNS服务端可以根据这些信息，选择最佳的服务节点访问。

如果有多个节点，还会考虑错误率、请求时间、服务器压力、网络状况等，进行综合选择，而非仅仅考虑地理位置。当有一个节点宕机或者性能下降的时候，可以尽快进行切换。

要做到这一点，需要客户端使用HttpDNS返回的IP访问业务应用。客户端的SDK会收集网络请求数据，如错误率、请求时间等网络请求质量数据，并发送到统计后台，进行分析、聚合，以此查看不同的IP的服务质量。

在**服务端**，应用可以通过调用HttpDNS的管理接口，配置不同服务质量的优先级、权重。HttpDNS会根据这些策略综合地理位置和线路状况算出一个排序，优先访问当前那些优质的、时延低的IP地址。

HttpDNS通过智能调度之后返回的结果，也会缓存在客户端。为了不让缓存使得调度失真，客户端可以根据不同的移动网络运营商WIFI的SSID来分维度缓存。不同的运营商或者WIFI解析出来的结果会不同。

![](<https://static001.geekbang.org/resource/image/3b/ae/3b368yy7a4f2319bd6491bc4e66e55ae.jpg?wh=1937*1634>)

## 小结

好了，这节就到这里了，我们来总结一下，你需要记住这两个重点：

- 传统的DNS有很多问题，例如解析慢、更新不及时。因为缓存、转发、NAT问题导致客户端误会自己所在的位置和运营商，从而影响流量的调度。

- HttpDNS通过客户端SDK和服务端，通过HTTP直接调用解析DNS的方式，绕过了传统DNS的这些缺点，实现了智能的调度。


<!-- -->

最后，给你留两个思考题。

1. 使用HttpDNS，需要向HttpDNS服务器请求解析域名，可是客户端怎么知道HttpDNS服务器的地址或者域名呢？

2. HttpDNS的智能调度，主要是让客户端选择最近的服务器，而有另一种机制，使得资源分发到离客户端更近的位置，从而加快客户端的访问，你知道是什么技术吗？


<!-- -->

我们的专栏更新过半了，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从已发布的文章中选出一批认真留言的同学，赠送学习奖励礼券和我整理的独家网络协议知识图谱。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(80)

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/bc/88a905a5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赵强强

  老师能不能写个专题介绍一下国内几大运营商网络状况和互联关系😬 

  2018-06-29

  **4

  **126

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/16/4d1e5cc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mgxian

  1.httpdns服务器的地址一般不变 可以使用dns的方式获取httpdns服务器的ip地址 也可以直接把httpdns服务器的ip地址写死在客户端中。 2.让用户获取离用户最近的资源 一般是静态资源 这使用的是cdn技术

  2018-06-29

  **1

  **87

- ![img](https://static001.geekbang.org/account/avatar/00/0f/80/fb/ffa57759.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  xyz

  \1. HTTPDNS 服务器的地址或者域名（和网站域名类似）一般不变，所以可以写死在SDK里； 2..为方便用户近源访问，使用CDN技术（content distribution network，内容分发网络）。  一点题外话：作者的每一期讲解都有好好学习，讲的都挺通俗易懂的，不过如果能加入一些相关的经典的参考链接，应该会更好，授人以鱼的同时也授人以渔。

  2018-06-29

  **

  **41

- ![img](https://static001.geekbang.org/account/avatar/00/12/4b/b2/7f600f9e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  silence

  之前在软件公司干活，有个大客户，有很多站点，但是他们原先的系统不支持调度。希望我们给他出个方案，做到智能调度。当时我出的方案是给他们搞一个类似 HTTPDNS 的系统。根据运营商、地理位置、站点负载进行调度。一次返回多个服务列表。让客户端进行缓存，定期更新。今天看了这个文章，在回想起当时我给他们做的方案，当时只做了服务器上的调度，客户端因为无法控制，当时没考虑进行改造，添加反馈。看了文章，看到还是有可以再优化的点的。

  2018-11-19

  **

  **26

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIu1n1DhUGGKS1obrPQicEHlhC7A5zuu6fCQVR4PNx06LNEgfITQqic9fRGYicI0TkFMQUfDAnR7YVLA/132)

  quanbove

  HTTPNDS 其实就是，不走传统的 DNS 解析，而是自己搭建基于 HTTP 协议的 DNS 服务器集群，分布在多个地点和多个运营商。 发现一个笔误的地方，HTTPDNS，而不是HTTPNDS，我是又多无聊[捂脸]

  作者回复: 谢谢指正

  2018-09-26

  **

  **19

- ![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,m_fill,h_34,w_34-1662307745804-8235.jpeg)

  upstream

  3.出口nat问题这段个人认为说的不准确。客户端的dns服务器指向一般都是一个运营商的localdns，不考虑edns的情况下，权威服务器拿到的是localdns地址。 如果递归dns支持edns，权威服务器能看到客户端出口nat地址。 只有像长城宽带这类流氓二道贩子，会把用户出口nat地址全国飘

  2018-06-29

  **

  **15

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/4f/6fb51ff1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  一步![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  我想问的是:HttpDns服务器的域名地址簿，也是通过递归根域名，顶级域名，权威域名服务器获取的吗？

  2018-08-03

  **3

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/11/47/8b/a5dfb558.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  摩诃不思议

  非计算机专业同学，学的比较慢，老师能送我一份网络协议知识图谱吗？

  2018-07-28

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6a/a8/ee414a25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  公告-SRE运维实践

  这个文章看起来感觉httpdns能解决传统的dns问题，但是前提是有自己的客户端，这种要对业务进行全部改造。那么如果就是浏览器访问，有没有什么方案，在不使用cdn的情况下

  作者回复: httpdns需要自己的客户端

  2018-08-21

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/11/60/c7/a147b71b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Fisher

  最近几篇文章都很赞啊，这一篇讲清楚了为什么跨运营商速度会慢的原因，之前知道这个问题但是不知道答案，其次httpdns这个新技术第一次听说，赞一个

  2018-06-29

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/10/07/8c/0d886dcc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Shopee内推码：NTAGxyl

  HTTPDNS能支持HTTPS 不

  作者回复: 可以的

  2019-06-16

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/12/a0/a7/db7a7c50.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  送普选

  咨询一下httpdns如何获取权威dns上的IP地址？每次有客户端httpdns请求来了就查一下，还是每1秒定期轮询后缓存？谢谢

  2019-06-14

  **1

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/16/b4/94/2796de72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的人

  非计算机专业同学，老师能送我一份网络协议知识图谱吗？

  作者回复: 买了课程的，最后一节都有

  2019-06-05

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/1d/8e/0a546871.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  凡凡

  1、可以是通过域名解析，也可以是直接配置所有httpdns服务器节点。阿里云采用的是配置ip列表的方式。2、cdn内容分发网络

  2018-07-03

  **1

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/15/53/0c/b4907516.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Marco

  那httpdns不需要考虑负载均衡么?

  作者回复: 考虑，httpdns里面也可以配置负载均衡策略的

  2019-03-19

  **2

  **3

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  哈哈哈

  请问老师：文中的在不同运营商或地区部署httpdns服务，根据客户端位置返回最优应用地址，httpdns服务端部署方式是如何的？如何达到不同运营商或地区效果？httpdns适用场景只有哪些呢？谢谢

  作者回复: 一般是多个数据中心分布式部署，适用于手机客户端的情况

  2019-02-14

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/94/2c22bd4e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  克里斯

  权威DNS服务器的ip地址不会变吗？

  作者回复: 很少变

  2019-02-08

  **

  **3

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/yy4cUibeUfPHPkXXZQnQwjXY7m5rXY5ib6a7pC1vkupj1icibF305N4pJSdqw0fO1ibvyfKCQ7HWggLhwiaNbbRPBsKg/132)

  桃子妈妈

  老师好，很想了解HTTPDNS服务端仍然会使用递归的方式依次通过根域名、顶级域名、权威域名去递归查询吗？

  作者回复: 不会的，直接返回了

  2019-01-31

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/b6/43/c23a38b2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  whats your name

  老师，有一个疑惑点：HTTPDNS的域名/ip地址簿是从哪里来的呢？

  作者回复: 管理员配置，反正httpdns服务器也是你自己搭建的，如果你用公有云，他也让你配置的

  2019-04-13

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/2b/84/07f0c0d6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  supermouse

  想问一下老师，HTTPDNS服务器集群是谁来搭建的？

  作者回复: 你可以自己搭建，也可以去公有云买

  2019-04-11

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/9e/c3/c321541c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Nicole

  1.写死在SDK中，支持DNS over HTTPS方案 2.CDN内容分发服务

  2018-08-03

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/6d/910b2445.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fastkdm

  为什么A运营商会偷懒，将请求转发给B运营商，有什么历史案例吗

  2021-02-09

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/9f/d6/f66133b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吴贤龙

  老师，要实现httpDNS，除了手机端，PC端怎么办呢？

  2020-04-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/37/56/11068390.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  倡印

  问个问题。移动端使用流量的时候使用HTTPDNS这好理解，但是移动端也可以接入家庭网络Wi-Fi家庭网络又是传统的DNS解析，这种情况下移动端还是会使用HTTPDNS来解析吗？？？？？？

  2020-02-07

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/4f/a5/71358d7b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  J.M.Liu

  老师，那httpdns服务器自己的地址簿，他怎么更新呢？

  作者回复: 会提供配置界面的

  2019-08-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/0e/e9/98b6ea61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员大天地

  第一次听说HTTPDNS，没有听懂。HTTPDNS服务器是如何部署的，服务器是如何知道域名的对应ip，难度它不用递归向权威DNS服务器查询吗？

  作者回复: 不用递归，其实就是普通的http服务器了

  2019-05-04

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/38/2d/9c971119.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  若丶相依

  HTTPDNS 服务器的地址或者域名由传统DNS做第一次的解析，获取到HTTPDNS服务器地址后由HTTPDNS重新解析自己的地址为最优节点 ?

  2018-12-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/24/e1cb609e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  破晓

  1，写死域名 2，cdn

  2018-07-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ac/a1/43d83698.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云学

  httpdns和反向代理有什么区别呢？

  作者回复: 没有关系吧

  2018-07-02

  **2

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/de/17/75e2b624.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  feifei

  使用 HTTPDNS，需要向 HTTPDNS 服务器请求解析域名，可是客户端怎么知道 HTTPDNS 服务器的地址或者域名呢？ 我的理解是服务端会对外公布一个基于ip的服务，客户端固定从这个服务获取信息 HTTPDNS 的智能调度，主要是让客户端选择最近的服务器，而有另一种机制，使得资源分发到离客户端更近的位置，从而加快客户端的访问，你知道是什么技术吗？ 比如一个跨地域的应用系统，存在北京，上海2个机房 可使用全局负载均衡来实现 可配置使用用户的ip来进行划分，比如某地址段访问北京机房，另外的访问上海机房 也可以感知用户的访问时间来进行负载均衡 需要按照业务特点来选择算法

  2018-06-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/44/f8/6fdb50ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  肖飞码字

  HTTPDNS这种只适用于手机app，对于电脑这种是没办法的吧？

  2018-06-29

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  free

  配置ip cdn

  2018-06-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/f8/8a/07cdeaf9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jason

  域名选择方面，我改选择.cn的还是.com的？.cn的域名访问速度是不是比.com快？

  2018-06-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/23/6c/82ba5e1f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LIKE

  题外话集成第三方的HttpDNS一般是要收费的，老师好，关于httpDNS 在手机切换网络时（4g切换到wifi）一般是怎么处理的，会重新请求更新ip吗？

  2022-05-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/26/76/a6/aac72751.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  半点茶凉

  httpdns很像微服务中注册中心的理论基础

  2022-04-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/29/53/92/21c78176.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小黄鸭

  不太明白，那HTTPDNS服务器里面的域名和ip映射关系，是从哪里来的？

  2022-02-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/34/cf/0a316b48.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  蝴蝶

  \1. 内置好地址如8.8.8.8 2.CDN

  2021-10-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/3d/a0/acf6b165.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  奋斗

  如果在阿里买一个域名，怎么同步到httpDNS

  2021-05-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/24/d2/0f/8f14c85b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  红薯板栗

  传统dns，由于缓存，转发，nat，导致不可控，httpdns自己搭建类似dns服务，可以自由在调度，缓存，更新上控制。

  2021-04-15

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83er6OV33jHia3U9LYlZEx2HrpsELeh3KMlqFiaKpSAaaZeBttXRAVvDXUgcufpqJ60bJWGYGNpT7752w/132)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  dog_brother

  第一次接触HTTP DNS的概念，觉得跟服务发现要解决的问题很类似

  2021-03-25

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eoYBX95GZxEdKz9LQJVwohUiaRxNge5WpHRbeOC2tGc2rsdpfYKCTKdQicBn8MvSrlZTX7HY2jS3YFA/132)

  青冈

  老师，iOS客户端有时做网络请求会有未能找到主机名的问题。这个就是dns导致的么？

  2021-03-11

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/ef/66/dc3f4555.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  赛飞

  听着想是只有在手机端安装SDK才能使用httpDNS，这个在PC端可以用吗？

  2020-11-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/9a/f1/fac9924a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  IMER

  传统 DNS 存在哪些问题 只有 1 2 4 5，没有 3？

  2020-09-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/6c/a8/1922a0f5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郑祖煌

  应该HttpDNS服务器是自己安装和使用的，所以可以自己把HttpDNS的地址写到配置中去。

  2020-08-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  奥佛铎

  1不经常变化，所以可以作为一种配置文件写在客户端中 2 CDN

  2020-08-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/3c/84/608f679b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  连边

  HttpDNS域名配置hosts

  2020-07-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  起风了001

  httpdns的ip地址列表是真的可以直接写死在客户端的, 比特币的原始节点服务器的ip就是写死的代码里的, 人家还是玩得好好的.

  作者回复: 不懂比特币。。。

  2020-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  1.对于HttpDNS的获取,应该是类似着CA权威认证服务器一样,是直接写死在客户端里面的,方便直接调用 2.对于所使用的的技术,应该是CDN

  2020-05-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/58/c7/a65f5080.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Lucky Guy

  老师如果我的app现在已经使用了CDN做加速， 如果我再使用httpdns做解析的话，我需要在httpdns里面配置cname的解析记录。 httpdns也支持cname解析么？

  2020-04-16

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/tKvmZ3Vs4t6RZ3X7cAliaW47Zatxhn1aV5PcCYT9NZ9k9WWqRrEBGHicGtRWvsG6yQqHnaWw6cGNSbicNLjZebcHA/132)

  柳十三

  1.客户端请求，内部做了个拦截，拦截后转发到httpdns 2.使用cdn

  2020-04-09

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  pc端如果也使用httpdns，就必须安装客户端了吧？

  2020-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  平常接触到DNS，考虑的都是通过域名去解析出IP进行访问，没去思考传统DNS的一些缺点：解析慢、缓存不更新等问题。同时对于手机端去访问网站也是没有去关注的，文中提到的httpDNS来快速进行解析和访问的智能调度，这些优点也是传统DNS的进一步解决方案。不断拜读文章，也是不断丰富自己的知识面，当然其中知识点的很多细节，需要自己在接下来的岁月不断打磨和完善。

  2020-03-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e4/a1/178387da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  25ma

  问题2，应该是大名鼎鼎的cdn网络资源分发

  2020-03-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/81/00/1df7bb5d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yvanbu

  请教一个问题：如果需要使用 HTTPDNS 服务，除了使用 阿里云 和 腾讯云 提供好的服务之外，可以自己搭建HTTPDNS 服务吗，成本大不大，需要不同的国家不同运营商搭建吗

  2020-03-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/ff/89/32e3f682.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  肖小强

  DNS协议发出的解析请求也是基于http的吗？

  2020-02-26

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  涤生

  第一个问题，把httpdns域名写死在sdk中，那么这个域名服务器全国就只有这一台吗？假设httpdns的域名为A，移动用户在B省、C省，B和C都通过A来获取域名？

  2020-02-07

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  涤生

  httpdns它也是向本地域名服务器请求结果的，如果本地域名服务器没有，也得向上一级域名服务器请求呀！为嘛这个说可以直接搞定？

  2020-02-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/07/d2/d7a200d5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小鸟淫太

  我们公司做的私有DNS感觉和这个类似，但是只是配置一个IP地址对应一个域名，没这么复杂。。

  2020-01-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ef/f1/8b06801a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谁都别拦着我

  刘超老师，最近对云计算感兴趣，有没有入门科普类的书推荐一下

  2019-12-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/d1/29/1b1234ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  DFighting

  1、httpdns服务器一般不需要太多的变化，所以可以考虑定死为某个IP;如果需要考虑容灾和负载均衡，可以将httpdns注册到zk中，以一种服务的方式提供调度和多活 2、cdn

  2019-12-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/99/af/d29273e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  饭粒

  “同步更新的优点是实时性好，缺点是如果有多个请求都发现过期的时候，同时会请求 HTTPDNS 多次，其实是一种浪费。” 老师，请问下，这里的多个请求都发现过期是什么意思？感觉做了 HTTPDNS 的应该是特定的应用，需要通过 HTTPDNS 解析的域名也不会太多，而且一般手机客户端一次发起的请求也不会太多吧。

  2019-12-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/48/ad/8be724da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈泽坛

  httpdns自己的dns服务器  成本好高吧？

  作者回复: 是的，但是公有云有这种服务的

  2019-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/e9/29/629d9bb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天王

  HTTPDNS，DNS是网络地址的地址簿，给一个域名，找到对应服务器的ip地址，也可以用DNS在多台服务器之间做负载均衡。1DNS也有一些问题，1 .1域名缓存问题，DNS解析的过程，首先运营商去本地DNS缓存服务器去看有没有，没有去DNS树去一级级的查找，如果缓存的地址或者资源文件变了，访问会出错 或者是过期的文件，另外DNS缓存缓存的地址，会使就近原则的负载均衡失效。1.2域名转发问题 很多网络运营商会把请求导到其他运营商，DNS服务器会认为是转发之后的运营商，给的ip也是转发以后运营商的ip，跨运营商会比较慢 1.3 ip地址转换问题 很多机房出口ip会进行转换成新ip，DNS服务器会识别出错误的运营商网络，跨网络请求会比较慢1.4 域名更新问题    域名解析策略，不同的地区，不同的运营商都有差异，但有时候需要新解析立刻生效，比如容错的双机房的一个机房出故障，需要全部解析到第二个机房。1.5 解析延迟问题。 2 2.1 HTTPDNS解析是自己搭建DNS解析服务器集群，分布在不同的地区和不同的运营商，域名走自己的解析，不走运营商的，绕过传统的DNS解析，需要客户端安装支持HTTPDNS的sdk。2.2 HTTPDNS的缓存策略 首先去访问HTTPDNS服务器，获取到ip列表，缓存在手机客户端本地，请求的时候，如果发现本地有，直接从本地获取，没有去httpdns服务器获取，因为手机客户端肯定知道在哪个地区和哪个运营商，选择最优的，更新缓存策略可以自己制定。

  2019-06-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/9f/64/a0a0904d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  cheetah

  1、域名写死 2、CDN HTTPDNS的自己的数据也是递归获取到的吗，从根域名到顶级域名再到权威域名

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b4/94/2796de72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的人

  1    httpdns服务器的IP地址是不变的，可 从DNS服务器查到 。 2    CDN(content  distribution  network   内容分发网络)

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/78/ac/e5e6e7f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古夜

  有人知道网页劫持是咋回事吗

  作者回复: 好多劫持的方法，dns劫持是一种方式

  2019-04-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/85/ed/905b052f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  超超

  答问题1：在客户端SDK中配置HTTPSDNS的域名

  2019-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  域名缓存问题的那段的配图有点问题，游客的位置放在灵隐寺，更好吧？

  2019-02-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/76/23/31e5e984.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  空知

  分维度那个,不能直接用IP,因为IP可能是私网的,不一定准确,还是得用WIFI的SSID来区分

  2018-12-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c0/0c/c978bd7c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  半城

  通过httpdns建立一个本地地址簿，问题是这个地址簿的内容是怎么获取到的呢？

  2018-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6a/e9/d76511d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Yesterday

  如何解决 httpdns 访问 https 的问题？因为 https 会做 hostname 检验，而使用 httpdns 之后是直接访问的服务器 ip, 导致检验失败。 一些客户端，比如 apache http client 可以重写域名检验部分的代码，但是很多库不会保留修改的接口。grpc 的我就没找到。

  2018-07-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/98/ef/bf392cc0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  shanshan

  苹果手机APP有时候会提示 “找不到主机名称”（a server with the specified hostname could not be found） 这个是DNS Pollution 导致的吗， 网上有人说 “这是苹果手机的问题， DNS过期了，它不能及时更新，系统又不使用过期的，所以就不能联系上服务器了 我觉得iPhone可能为了省流量或者是电，所以是有个间隔”统一更新DNS缓存，而我们域名默认的过期时间小于“统一更新时间 ，这时候问题就发生了” 但是这个DNS缓存不是在运营商那里吗，那导致这个问题的原因什么,希望老师能指点一下。

  2018-07-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a9/90/0c5ed3d9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  颇忒妥

  这么看来httpdns 和spring cloud 的client side load balance 有异曲同工之妙

  2018-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/e8/1b/9d1c6077.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  曹源

  1，可以在sdk中写死，或者用传统dns解析

  2018-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/23/5b/983408b9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  悟空聊架构

  第二个问题：CDN

  2018-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/20/36/b3e2f1d5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  wykkx

  Httpdns的地址一般不变，所以可以配置在客户端。第二个问题说的是cdn技术

  2018-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4e/fc/a10213cf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  aha

  sdk可以内置IP或者域名 cdn

  2018-06-29

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  khooh

  第二个是CDN，内容分发网络

  2018-06-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/41/b3/3ba14f9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张磊

  问题一: 客户端如果要通过httpdns域名解析，应该通过编码的方式，指向httpdns服务器。或使用三方的httpdns的sdk。使用提供的api请求得到所访问域名的ip地址。缓存在客户端。

  2018-06-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d4/c2/910d231e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC19%E8%AE%B2%20_%20HttpDNS%EF%BC%9A%E7%BD%91%E7%BB%9C%E4%B8%96%E7%95%8C%E7%9A%84%E5%9C%B0%E5%9D%80%E7%B0%BF%E4%B9%9F%E4%BC%9A%E6%8C%87%E9%94%99%E8%B7%AF.resource/resize,w_14.png)

  oddrock

  请教老师几个问题： 1、为什么客户端要根据不同的移动网络运营商 WIFI 的 SSID 来分维度缓存，这个根据客户端发起请求的IP地址不就可以区分了吗？ 2、DNS是基于什么协议的？TCP吗？

  2018-06-29
