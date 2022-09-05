# 第34讲 \| 基于JSON的RESTful接口协议：我不关心过程，请给我结果

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/05/e6/05c1710c966ae46cbd41ec1b9ed909e6.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/7a/cb/7a7e5b3b70e60c69b8a533659260dacb.mp3" type="audio/mpeg"></audio>

上一节我们讲了基于XML的SOAP协议，SOAP的S是啥意思来着？是Simple，但是好像一点儿都不简单啊！

你会发现，对于SOAP来讲，无论XML中调用的是什么函数，多是通过HTTP的POST方法发送的。但是咱们原来学HTTP的时候，我们知道HTTP除了POST，还有PUT、DELETE、GET等方法，这些也可以代表一个个动作，而且基本满足增、删、查、改的需求，比如增是POST，删是DELETE，查是GET，改是PUT。

## 传输协议问题

对于SOAP来讲，比如我创建一个订单，用POST，在XML里面写明动作是CreateOrder；删除一个订单，还是用POST，在XML里面写明了动作是DeleteOrder。其实创建订单完全可以使用POST动作，然后在XML里面放一个订单<!-- [[[read_end]]] -->

的信息就可以了，而删除用DELETE动作，然后在XML里面放一个订单的ID就可以了。

于是上面的那个SOAP就变成下面这个简单的模样。

```
POST /purchaseOrder HTTP/1.1
Host: www.geektime.com
Content-Type: application/xml; charset=utf-8
Content-Length: nnn

<?xml version="1.0"?>
 <order>
     <date>2018-07-01</date>
      <className>趣谈网络协议</className>
       <Author>刘超</Author>
       <price>68</price>
  </order>
```

而且XML的格式也可以改成另外一种简单的文本化的对象表示格式JSON。

```
POST /purchaseOrder HTTP/1.1
Host: www.geektime.com
Content-Type: application/json; charset=utf-8
Content-Length: nnn

{
 "order": {
  "date": "2018-07-01",
  "className": "趣谈网络协议",
  "Author": "刘超",
  "price": "68"
 }
}
```

经常写Web应用的应该已经发现，这就是RESTful格式的API的样子。

## 协议约定问题

然而RESTful可不仅仅是指API，而是一种架构风格，全称Representational State Transfer，表述性状态转移，来自一篇重要的论文《架构风格与基于网络的软件架构设计》（Architectural Styles and the Design of Network-based Software Architectures）。

这篇文章从深层次，更加抽象地论证了一个互联网应用应该有的设计要点，而这些设计要点，成为后来我们能看到的所有高并发应用设计都必须要考虑的问题，再加上REST API比较简单直接，所以后来几乎成为互联网应用的标准接口。

因此，和SOAP不一样，REST不是一种严格规定的标准，它其实是一种设计风格。如果按这种风格进行设计，RESTful接口和SOAP接口都能做到，只不过后面的架构是REST倡导的，而SOAP相对比较关注前面的接口。

而且由于能够通过WSDL生成客户端的Stub，因而SOAP常常被用于类似传统的RPC方式，也即调用远端和调用本地是一样的。

然而本地调用和远程跨网络调用毕竟不一样，这里的不一样还不仅仅是因为有网络而导致的客户端和服务端的分离，从而带来的网络性能问题。更重要的问题是，客户端和服务端谁来维护状态。所谓的状态就是对某个数据当前处理到什么程度了。

这里举几个例子，例如，我浏览到哪个目录了，我看到第几页了，我要买个东西，需要扣减一下库存，这些都是状态。本地调用其实没有人纠结这个问题，因为数据都在本地，谁处理都一样，而且一边处理了，另一边马上就能看到。

当有了RPC之后，我们本来期望对上层透明，就像上一节说的“远在天边，尽在眼前”。于是使用RPC的时候，对于状态的问题也没有太多的考虑。

就像NFS一样，客户端会告诉服务端，我要进入哪个目录，服务端必须要为某个客户端维护一个状态，就是当前这个客户端浏览到哪个目录了。例如，客户端输入cd hello，服务端要在某个地方记住，上次浏览到/root/liuchao了，因而客户的这次输入，应该给它显示/root/liuchao/hello下面的文件列表。而如果有另一个客户端，同样输入cd hello，服务端也在某个地方记住，上次浏览到/var/lib，因而要给客户显示的是/var/lib/hello。

不光NFS，如果浏览翻页，我们经常要实现函数next()，在一个列表中取下一页，但是这就需要服务端记住，客户端A上次浏览到20～30页了，那它调用next()，应该显示30～40页，而客户端B上次浏览到100～110页了，调用next()应该显示110～120页。

上面的例子都是在RPC场景下，由服务端来维护状态，很多SOAP接口设计的时候，也常常按这种模式。这种模式原来没有问题，是因为客户端和服务端之间的比例没有失衡。因为一般不会同时有太多的客户端同时连上来，所以NFS还能把每个客户端的状态都记住。

公司内部使用的ERP系统，如果使用SOAP的方式实现，并且服务端为每个登录的用户维护浏览到报表那一页的状态，由于一个公司内部的人也不会太多，把ERP放在一个强大的物理机上，也能记得过来。

但是互联网场景下，客户端和服务端就彻底失衡了。你可以想象“双十一”，多少人同时来购物，作为服务端，它能记得过来吗？当然不可能，只好多个服务端同时提供服务，大家分担一下。但是这就存在一个问题，服务端怎么把自己记住的客户端状态告诉另一个服务端呢？或者说，你让我给你分担工作，你也要把工作的前因后果给我说清楚啊！

那服务端索性就要想了，既然这么多客户端，那大家就分分工吧。服务端就只记录资源的状态，例如文件的状态，报表的状态，库存的状态，而客户端自己维护自己的状态。比如，你访问到哪个目录了啊，报表的哪一页了啊，等等。

这样对于API也有影响，也就是说，当客户端维护了自己的状态，就不能这样调用服务端了。例如客户端说，我想访问当前目录下的hello路径。服务端说，我怎么知道你的当前路径。所以客户端要先看看自己当前路径是/root/liuchao，然后告诉服务端说，我想访问/root/liuchao/hello路径。

再比如，客户端说我想访问下一页，服务端说，我怎么知道你当前访问到哪一页了。所以客户端要先看看自己访问到了100～110页，然后告诉服务器说，我想访问110～120页。

这就是服务端的无状态化。这样服务端就可以横向扩展了，一百个人一起服务，不用交接，每个人都能处理。

所谓的无状态，其实是服务端维护资源的状态，客户端维护会话的状态。对于服务端来讲，只有资源的状态改变了，客户端才调用POST、PUT、DELETE方法来找我；如果资源的状态没变，只是客户端的状态变了，就不用告诉我了，对于我来说都是统一的GET。

虽然这只改进了GET，但是已经带来了很大的进步。因为对于互联网应用，大多数是读多写少的。而且只要服务端的资源状态不变，就给了我们缓存的可能。例如可以将状态缓存到接入层，甚至缓存到CDN的边缘节点，这都是资源状态不变的好处。

按照这种思路，对于API的设计，就慢慢变成了以资源为核心，而非以过程为核心。也就是说，客户端只要告诉服务端你想让资源状态最终变成什么样就可以了，而不用告诉我过程，不用告诉我动作。

还是文件目录的例子。客户端应该访问哪个绝对路径，而非一个动作，我就要进入某个路径。再如，库存的调用，应该查看当前的库存数目，然后减去购买的数量，得到结果的库存数。这个时候应该设置为目标库存数（但是当前库存数要匹配），而非告知减去多少库存。

这种API的设计需要实现幂等，因为网络不稳定，就会经常出错，因而需要重试，但是一旦重试，就会存在幂等的问题，也就是同一个调用，多次调用的结果应该一样，不能一次支付调用，因为调用三次变成了支付三次。不能进入cd a，做了三次，就变成了cd a/a/a。也不能扣减库存，调用了三次，就扣减三次库存。

当然按照这种设计模式，无论RESTful API还是SOAP API都可以将架构实现成无状态的，面向资源的、幂等的、横向扩展的、可缓存的。

但是SOAP的XML正文中，是可以放任何动作的。例如XML里面可以写< ADD >，< MINUS >等。这就方便使用SOAP的人，将大量的动作放在API里面。

RESTful没这么复杂，也没给客户提供这么多的可能性，正文里的JSON基本描述的就是资源的状态，没办法描述动作，而且能够出发的动作只有CRUD，也即POST、GET、PUT、DELETE，也就是对于状态的改变。

所以，从接口角度，就让你死了这条心。当然也有很多技巧的方法，在使用RESTful API的情况下，依然提供基于动作的有状态请求，这属于反模式了。

## 服务发现问题

对于RESTful API来讲，我们已经解决了传输协议的问题——基于HTTP，协议约定问题——基于JSON，最后要解决的是服务发现问题。

有个著名的基于RESTful API的跨系统调用框架叫Spring Cloud。在Spring Cloud中有一个组件叫 Eureka。传说，阿基米德在洗澡时发现浮力原理，高兴得来不及穿上裤子，跑到街上大喊：“Eureka（我找到了）！”所以Eureka是用来实现注册中心的，负责维护注册的服务列表。

服务分服务提供方，它向Eureka做服务注册、续约和下线等操作，注册的主要数据包括服务名、机器IP、端口号、域名等等。

另外一方是服务消费方，向Eureka获取服务提供方的注册信息。为了实现负载均衡和容错，服务提供方可以注册多个。

当消费方要调用服务的时候，会从注册中心读出多个服务来，那怎么调用呢？当然是RESTful方式了。

Spring Cloud提供一个RestTemplate工具，用于将请求对象转换为JSON，并发起Rest调用，RestTemplate的调用也是分POST、PUT、GET、 DELETE的，当结果返回的时候，根据返回的JSON解析成对象。

通过这样封装，调用起来也很方便。

## 小结

好了，这一节就到这里了，我们来总结一下。

- SOAP过于复杂，而且设计是面向动作的，因而往往因为架构问题导致并发量上不去。

- RESTful不仅仅是一个API，而且是一种架构模式，主要面向资源，提供无状态服务，有利于横向扩展应对高并发。


<!-- -->

最后，给你留两个思考题：

1. 在讨论RESTful模型的时候，举了一个库存的例子，但是这种方法有很大问题，那你知道为什么要这样设计吗？

2. 基于文本的RPC虽然解决了二进制的问题，但是它本身也有问题，你能举出一些例子吗？


<!-- -->

我们的专栏更新到第34讲，不知你掌握得如何？每节课后我留的思考题，你都有没有认真思考，并在留言区写下答案呢？我会从**已发布的文章中选出一批认真留言的同学**，赠送<span class="orange">学习奖励礼券</span>

和我整理的<span class="orange">独家网络协议知识图谱</span>

。

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(30)

- ![img](%E7%AC%AC34%E8%AE%B2%20_%20%E5%9F%BA%E4%BA%8EJSON%E7%9A%84RESTful%E6%8E%A5%E5%8F%A3%E5%8D%8F%E8%AE%AE%EF%BC%9A%E6%88%91%E4%B8%8D%E5%85%B3%E5%BF%83%E8%BF%87%E7%A8%8B%EF%BC%8C%E8%AF%B7%E7%BB%99%E6%88%91%E7%BB%93%E6%9E%9C.resource/resize,m_fill,h_34,w_34.jpeg)

  feifei

  在讨论 RESTful 模型的时候，举了一个库存的例子，但是这种方法有很大问题，那你知道为什么要这样设计吗？ 此方法的问题在于，不是解决问题，而是将数据状态进行了转移，将状态交给存储，这样业务将可以无状态化运行，这种设计可以很好的解决扩展的问题，因为无状态，可以进行负载均衡！使用集群化来解决单机的问题。 基于文本的 RPC 虽然解决了二进制的问题，但是它本身也有问题，你能举出一些例子吗？ 1，效率问题，程序与文本之间转换效率低，因而不适合内部大数据交换，因为文本利用阅读，对外采用较好 2，相比于二进制rpc,传输需要的带宽更大，二进制的rpc因为可以使用专用的客户短和服务器代码，可以更好的压缩数据，以提供更大的吞吐量

  2018-08-03

  **

  **48

- ![img](%E7%AC%AC34%E8%AE%B2%20_%20%E5%9F%BA%E4%BA%8EJSON%E7%9A%84RESTful%E6%8E%A5%E5%8F%A3%E5%8D%8F%E8%AE%AE%EF%BC%9A%E6%88%91%E4%B8%8D%E5%85%B3%E5%BF%83%E8%BF%87%E7%A8%8B%EF%BC%8C%E8%AF%B7%E7%BB%99%E6%88%91%E7%BB%93%E6%9E%9C.resource/resize,m_fill,h_34,w_34-1662308266040-10213.png)

  扬～

  文本传输最终都会转化为二进制流啊，为什么文本要比二进制rpc占用带宽？

  作者回复: 数字2如果用int传输用几个bit，如果是字符串呢？

  2018-11-10

  **2

  **15

- ![img](%E7%AC%AC34%E8%AE%B2%20_%20%E5%9F%BA%E4%BA%8EJSON%E7%9A%84RESTful%E6%8E%A5%E5%8F%A3%E5%8D%8F%E8%AE%AE%EF%BC%9A%E6%88%91%E4%B8%8D%E5%85%B3%E5%BF%83%E8%BF%87%E7%A8%8B%EF%BC%8C%E8%AF%B7%E7%BB%99%E6%88%91%E7%BB%93%E6%9E%9C.resource/resize,m_fill,h_34,w_34-1662308266040-10214.jpeg)

  kuzan

  crud的语义离业务有点远，客户端往往不想关心crud，客户端关注的语义是业务，比如审批、下单，添加好友。感觉用http这几个method做语义就把服务变成了dao，一个贫血服务

  2018-08-08

  **1

  **14

- ![img](%E7%AC%AC34%E8%AE%B2%20_%20%E5%9F%BA%E4%BA%8EJSON%E7%9A%84RESTful%E6%8E%A5%E5%8F%A3%E5%8D%8F%E8%AE%AE%EF%BC%9A%E6%88%91%E4%B8%8D%E5%85%B3%E5%BF%83%E8%BF%87%E7%A8%8B%EF%BC%8C%E8%AF%B7%E7%BB%99%E6%88%91%E7%BB%93%E6%9E%9C.resource/resize,m_fill,h_34,w_34-1662308266040-10215.jpeg)

  fsj

  没有restful api之前的json api 是什么样子的？感觉对于客户端同学，不了解之前是什么样子，很难体会到restful api的有点。

  2018-08-08

  **1

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/11/23/5b/983408b9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  悟空聊架构

  题目1: 库存的问题是：存在并发，导致库存可能为负值。 用Restful来进行无状态的访问，库存量即状态由业务层来解决。 题目2: 用文本来进行RPC的请求和响应，占用的字节数大。

  2018-08-06

  **

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/10/5e/2b/df3983e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  朱显杰

  能不能谈谈dubbo和springcloud在服务发现方面的优缺点？

  2018-08-03

  **

  **7

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  文本传输最终都会转化为二进制流啊，为什么文本要比二进制rpc占用带宽？  1 2018-11-10 作者回复: 数字2如果用int传输用几个bit，如果是字符串呢？ 这个有点疑问, 用int传输需要32或者64位; 字符串的话, 看编码, 如果是utf8编码的话, 还是1个字节8bit即可表示.....

  作者回复: int当然不会用32位编码呀

  2019-05-24

  **3

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c2/e0/7188aa0a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  blackpiglet

  第一题，我的理解是，资源最终还是有状态的，所以 rest 方式，是把状态转移到了数据库和客户端。那么数据库的抗压能力和稳定性就非常重要了，这也是为什么最近有这么多内存数据库和键值数据库的原因。另外客户端有状态也会造成很多麻烦，毕竟这是不受开发人员控制的，如果逻辑没有切割清楚，升级会非常痛苦。 第二题，能想到的主要是性能的损耗，毕竟传输的内容更多了，单位带宽下能传输的信息总量会有明显下降。

  2018-08-07

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/10/1d/8e/0a546871.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  凡凡

  第一个问题有点模糊，感觉这个设计没有问题，任何传输方式，一旦经过网络，都会发生很多可能性，必须做幂等处理。如果指的是服务端无状态的话，原因就是提升扩展性，应对C端用户的大规模和并发。 第二个问题，主要在于序列化和传输。序列化方面由于有格式就不如二进制紧凑，传输的数据量相对来说要大。另一个是二进制可以自定义规范，或者编码方案，传输路径上，数据被截获，也不容易解析。

  2018-08-03

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/ed/f1/e31585a8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  jonhey

  看到这里，强烈建议老刘基于此教程，丰富一下内容，写本书，估计能成为网络、云计算、网络编程、微服务领域集大成的经典教材

  作者回复: 不敢不敢

  2019-02-18

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/13/32/fc/5d901185.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  vic

  楼上那位，登陆动作可以看作是对session的CRUD操作

  2018-12-04

  **

  **4

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_f6f02b

  RESTful 模型如何实现幂等，这个表示没有想通，就是好比之前SOAP，你告诉我减库存，我就执行减，减后还剩多少，是否成功返回，但是如果是客户端直接减去库存，然后告诉我说，将库存设置成这么多，服务端只要告诉我是否成功，并发了怎么解决，多个人同时请求，如果库存为10，5个请求都是减1，如果前面失败，但是后面成功，前面分别说将库存设置成9、8、7、6，都失败了，最后一个说设置成5成功了，感觉会有问题。

  作者回复: 你这种减是有问题的，并发怎么办，后面5个只有一个成功是对的，其他还可以重试。

  2019-04-25

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f3/1f/05f97f0f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  葱本

  问个问题，购买这个动作，是告诉服务端减一，难受没有办法做到只告诉服务端结果吧

  作者回复: 购买可不止减库存，服务端可以有个状态机的，要不然重复购买怎么办。

  2019-04-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/85/7c/03a268fe.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  leohuachao

  我觉得RESTFul架构流行，也得益于前端框架的丰富吧，要不然维护客户端会话也够难实现

  作者回复: 是的，简单

  2018-09-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/4a/8a/c1069412.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  makermade

  讲得很好

  2018-09-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/24/df/645f8087.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Yayu

  JSON-RESTful 算是一种协议吗？把它理解成一种规范会更好吧？

  作者回复: 没必要纠结吧

  2018-08-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/2e/27/b4/df65c0f7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  | ~浑蛋~![img](https://static001.geekbang.org/resource/image/8a/95/8ac15yy848f71919ef784a1c9065e495.png)

  基于文本的 RPC怎么做文件传输，base64编码吗

  2022-07-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/2c/5e/7b/0e1eb97a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  luojielu

  存在并冲突的问题，比如两个都是出库的操作，两个操作都在未减库存前的时间点查询了库存，然后都返回了服务端减去库存后的数量，正确的结果应该是分开执行

  2022-03-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/90/a0/162f298a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Kanon°

  soap是指传输xml格式的接口，restful是传输json格式的接口，是这样定义的吗？；如果使用http请求传输xml中包了个json，这种叫什么？还是没有理清到底什么是soap接口，什么是restful接口，有没有严格的规范定义

  2021-09-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/43/e1/b7be5560.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sam

  表述性状态转移就是由前端转移到后端吗

  2020-07-02

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/72/85/c337e9a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  老兵

  \1. 库存的例子restful存在的问题，在于并发的请求，可能会导致库存变成负数 2. 基于文本的rpc问题传输量大，且存在序列化和反序列化的操作（伴随着大量的计算）

  2020-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/21/b1/87d742bf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  在下科南有何贵干

  那我在手机app上看的B站的视频进度，为什么可以同步到PC的网页上呢？它在B站服务器上还是无状态化吗？

  2020-03-27

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  在应用层SOAP和RESTful中做资源的调度，有自己的系统运作的模式。

  2020-03-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5f/81/1c614f4a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  stg609

  想问下，关于库存问题的最终答案在课程哪一讲中有公布吗？

  2019-10-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c0/6c/29be1864.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  随心而至

  底子好，学起什么来都快

  作者回复: 是的

  2019-05-25

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  我们平时设计的api, 比如购买东西, 是传的数量, 而不是传库存剩余多少呀. 传数量应该是主流才是, 所以传数量不是RESTful模式吗?

  作者回复: 对外接口是，内部实现不是

  2019-05-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/65/03/973b24ec.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  谢晋

  1、库存的扣减，需要考虑幂等性 ，商品的库存也是一种资源，但是好像没有删除库存一说，只有修改和查询。 2、基于文本的RPC是指的SOAP吗？

  2019-05-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ac/a1/43d83698.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  云学

  这篇把无状态服务讲透彻了，太棒了

  2018-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/15/4b/bd086599.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  波

  高并发下  尽管接口支持了幂等性  库存资源修改时需要支持cas操作

  2018-08-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/16/4d1e5cc1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mgxian

  2.字段冗余 有时候soap只需要一次调用能完成的功能 restful可能需要多次api调用才能完成 不是所有动作都可以转换为资源 比如 login 

  2018-08-03

  **

  **

