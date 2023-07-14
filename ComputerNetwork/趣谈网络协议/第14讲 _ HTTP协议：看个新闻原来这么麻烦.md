# 第14讲 \| HTTP协议：看个新闻原来这么麻烦

<audio><source src="https://static001.geekbang.org/resource/audio/4f/34/4f71470a46d7f82c603d9d27b2756034.mp3" type="audio/mpeg"></audio>

前面讲述完**传输层**，接下来开始讲**应用层**的协议。从哪里开始讲呢，就从咱们最常用的HTTP协议开始。

HTTP协议，几乎是每个人上网用的第一个协议，同时也是很容易被人忽略的协议。

既然说看新闻，咱们就先登录 [http://www.163.com](<http://www.163.com>) 。

[http://www.163.com](<http://www.163.com>) 是个URL，叫作**统一资源定位符**。之所以叫统一，是因为它是有格式的。HTTP称为协议，www.163.com是一个域名，表示互联网上的一个位置。有的URL会有更详细的位置标识，例如 [http://www.163.com/index.html](<http://www.163.com/index.html>) 。正是因为这个东西是统一的，所以当你把这样一个字符串输入到浏览器的框里的时候，浏览器才知道如何进行统一处理。

## HTTP请求的准备

浏览器会将www.163.com这个域名发送给DNS服务器，让它解析为IP地址。有关DNS的过程，其实非常复杂，这个在后面专门介绍DNS的时候，我会详细描述，这里我们先不管，反正它会被解析成为IP地址。那接下来是发送HTTP请求吗？

不是的，HTTP是基于TCP协议的，当然是要先建立TCP连接了，怎么建立呢？还记得第11节讲过的三次握手吗？

目前使用的HTTP协议大部分都是1.1。在1.1的协议里面，默认是开启了Keep-Alive的，这样建立的TCP连接，就可以在多次请求中复用。

<!-- [[[read_end]]] -->

学习了TCP之后，你应该知道，TCP的三次握手和四次挥手，还是挺费劲的。如果好不容易建立了连接，然后就做了一点儿事情就结束了，有点儿浪费人力和物力。

## HTTP请求的构建

建立了连接以后，浏览器就要发送HTTP的请求。

请求的格式就像这样。

![](<https://static001.geekbang.org/resource/image/85/c1/85ebb0396cbaa45ce00b505229e523c1.jpeg?wh=1920*1080>)

HTTP的报文大概分为三大部分。第一部分是**请求行**，第二部分是请求的**首部**，第三部分才是请求的**正文实体**。

### 第一部分：请求行

在请求行中，URL就是 [http://www.163.com](<http://www.163.com>) ，版本为HTTP 1.1。这里要说一下的，就是方法。方法有几种类型。

对于访问网页来讲，最常用的类型就是**GET**。顾名思义，GET就是去服务器获取一些资源。对于访问网页来讲，要获取的资源往往是一个页面。其实也有很多其他的格式，比如说返回一个JSON字符串，到底要返回什么，是由服务器端的实现决定的。

例如，在云计算中，如果我们的服务器端要提供一个基于HTTP协议的API，获取所有云主机的列表，这就会使用GET方法得到，返回的可能是一个JSON字符串。字符串里面是一个列表，列表里面是一项的云主机的信息。

另外一种类型叫做**POST**。它需要主动告诉服务端一些信息，而非获取。要告诉服务端什么呢？一般会放在正文里面。正文可以有各种各样的格式。常见的格式也是JSON。

例如，我们下一节要讲的支付场景，客户端就需要把“我是谁？我要支付多少？我要买啥？”告诉服务器，这就需要通过POST方法。

再如，在云计算里，如果我们的服务器端，要提供一个基于HTTP协议的创建云主机的API，也会用到POST方法。这个时候往往需要将“我要创建多大的云主机？多少CPU多少内存？多大硬盘？”这些信息放在JSON字符串里面，通过POST的方法告诉服务器端。

还有一种类型叫**PUT**，就是向指定资源位置上传最新内容。但是，HTTP的服务器往往是不允许上传文件的，所以PUT和POST就都变成了要传给服务器东西的方法。

在实际使用过程中，这两者还会有稍许的区别。POST往往是用来创建一个资源的，而PUT往往是用来修改一个资源的。

例如，云主机已经创建好了，我想对这个云主机打一个标签，说明这个云主机是生产环境的，另外一个云主机是测试环境的。那怎么修改这个标签呢？往往就是用PUT方法。

再有一种常见的就是**DELETE**。这个顾名思义就是用来删除资源的。例如，我们要删除一个云主机，就会调用DELETE方法。

### 第二部分：首部字段

请求行下面就是我们的首部字段。首部是key value，通过冒号分隔。这里面，往往保存了一些非常重要的字段。

例如，**Accept-Charset**，表示**客户端可以接受的字符集**。防止传过来的是另外的字符集，从而导致出现乱码。

再如，**Content-Type**是指**正文的格式**。例如，我们进行POST的请求，如果正文是JSON，那么我们就应该将这个值设置为JSON。

这里需要重点说一下的就是**缓存**。为啥要使用缓存呢？那是因为一个非常大的页面有很多东西。

例如，我浏览一个商品的详情，里面有这个商品的价格、库存、展示图片、使用手册等等。商品的展示图片会保持较长时间不变，而库存会根据用户购买的情况经常改变。如果图片非常大，而库存数非常小，如果我们每次要更新数据的时候都要刷新整个页面，对于服务器的压力就会很大。

对于这种高并发场景下的系统，在真正的业务逻辑之前，都需要有个接入层，将这些静态资源的请求拦在最外面。

这个架构的图就像这样。

![](<https://static001.geekbang.org/resource/image/ca/1d/caec3ba1086557cbf694c621e7e01e1d.jpeg?wh=1254*1080>)

其中DNS、CDN我在后面的章节会讲。和这一节关系比较大的就是Nginx这一层，它如何处理HTTP协议呢？对于静态资源，有Vanish缓存层。当缓存过期的时候，才会访问真正的Tomcat应用集群。

在HTTP头里面，**Cache-control**是用来**控制缓存**的。当客户端发送的请求中包含max-age指令时，如果判定缓存层中，资源的缓存时间数值比指定时间的数值小，那么客户端可以接受缓存的资源；当指定max-age值为0，那么缓存层通常需要将请求转发给应用集群。

另外，**If-Modified-Since**也是一个关于缓存的。也就是说，如果服务器的资源在某个时间之后更新了，那么客户端就应该下载最新的资源；如果没有更新，服务端会返回“304 Not Modified”的响应，那客户端就不用下载了，也会节省带宽。

到此为止，我们仅仅是拼凑起了HTTP请求的报文格式，接下来，浏览器会把它交给下一层传输层。怎么交给传输层呢？其实也无非是用Socket这些东西，只不过用的浏览器里，这些程序不需要你自己写，有人已经帮你写好了。

## HTTP请求的发送

HTTP协议是基于TCP协议的，所以它使用面向连接的方式发送请求，通过stream二进制流的方式传给对方。当然，到了TCP层，它会把二进制流变成一个个报文段发送给服务器。

在发送给每个报文段的时候，都需要对方有一个回应ACK，来保证报文可靠地到达了对方。如果没有回应，那么TCP这一层会进行重新传输，直到可以到达。同一个包有可能被传了好多次，但是HTTP这一层不需要知道这一点，因为是TCP这一层在埋头苦干。

TCP层发送每一个报文的时候，都需要加上自己的地址（即源地址）和它想要去的地方（即目标地址），将这两个信息放到IP头里面，交给IP层进行传输。

IP层需要查看目标地址和自己是否是在同一个局域网。如果是，就发送ARP协议来请求这个目标地址对应的MAC地址，然后将源MAC和目标MAC放入MAC头，发送出去即可；如果不在同一个局域网，就需要发送到网关，还要需要发送ARP协议，来获取网关的MAC地址，然后将源MAC和网关MAC放入MAC头，发送出去。

网关收到包发现MAC符合，取出目标IP地址，根据路由协议找到下一跳的路由器，获取下一跳路由器的MAC地址，将包发给下一跳路由器。

这样路由器一跳一跳终于到达目标的局域网。这个时候，最后一跳的路由器能够发现，目标地址就在自己的某一个出口的局域网上。于是，在这个局域网上发送ARP，获得这个目标地址的MAC地址，将包发出去。

目标的机器发现MAC地址符合，就将包收起来；发现IP地址符合，根据IP头中协议项，知道自己上一层是TCP协议，于是解析TCP的头，里面有序列号，需要看一看这个序列包是不是我要的，如果是就放入缓存中然后返回一个ACK，如果不是就丢弃。

TCP头里面还有端口号，HTTP的服务器正在监听这个端口号。于是，目标机器自然知道是HTTP服务器这个进程想要这个包，于是将包发给HTTP服务器。HTTP服务器的进程看到，原来这个请求是要访问一个网页，于是就把这个网页发给客户端。

## HTTP返回的构建

HTTP的返回报文也是有一定格式的。这也是基于HTTP 1.1的。

![](<https://static001.geekbang.org/resource/image/6b/63/6bc37ddcb4e7a61ca3275790820f2263.jpeg?wh=1761*937>)

状态码会反映HTTP请求的结果。“200”意味着大吉大利；而我们最不想见的，就是“404”，也就是“服务端无法响应这个请求”。然后，短语会大概说一下原因。

接下来是返回首部的**key value**。

这里面，**Retry-After**表示，告诉客户端应该在多长时间以后再次尝试一下。“503错误”是说“服务暂时不再和这个值配合使用”。

在返回的头部里面也会有**Content-Type**，表示返回的是HTML，还是JSON。

构造好了返回的HTTP报文，接下来就是把这个报文发送出去。还是交给Socket去发送，还是交给TCP层，让TCP层将返回的HTML，也分成一个个小的段，并且保证每个段都可靠到达。

这些段加上TCP头后会交给IP层，然后把刚才的发送过程反向走一遍。虽然两次不一定走相同的路径，但是逻辑过程是一样的，一直到达客户端。

客户端发现MAC地址符合、IP地址符合，于是就会交给TCP层。根据序列号看是不是自己要的报文段，如果是，则会根据TCP头中的端口号，发给相应的进程。这个进程就是浏览器，浏览器作为客户端也在监听某个端口。

当浏览器拿到了HTTP的报文。发现返回“200”，一切正常，于是就从正文中将HTML拿出来。HTML是一个标准的网页格式。浏览器只要根据这个格式，展示出一个绚丽多彩的网页。

这就是一个正常的HTTP请求和返回的完整过程。

## HTTP 2.0

当然HTTP协议也在不断的进化过程中，在HTTP1.1基础上便有了HTTP 2.0。

HTTP 1.1在应用层以纯文本的形式进行通信。每次通信都要带完整的HTTP的头，而且不考虑pipeline模式的话，每次的过程总是像上面描述的那样一去一回。这样在实时性、并发性上都存在问题。

为了解决这些问题，HTTP 2.0会对HTTP的头进行一定的压缩，将原来每次都要携带的大量key value在两端建立一个索引表，对相同的头只发送索引表中的索引。

另外，HTTP 2.0协议将一个TCP的连接中，切分成多个流，每个流都有自己的ID，而且流可以是客户端发往服务端，也可以是服务端发往客户端。它其实只是一个虚拟的通道。流是有优先级的。

HTTP 2.0还将所有的传输信息分割为更小的消息和帧，并对它们采用二进制格式编码。常见的帧有**Header帧**，用于传输Header内容，并且会开启一个新的流。再就是**Data帧**，用来传输正文实体。多个Data帧属于同一个流。

通过这两种机制，HTTP 2.0的客户端可以将多个请求分到不同的流中，然后将请求内容拆成帧，进行二进制传输。这些帧可以打散乱序发送， 然后根据每个帧首部的流标识符重新组装，并且可以根据优先级，决定优先处理哪个流的数据。

我们来举一个例子。

假设我们的一个页面要发送三个独立的请求，一个获取css，一个获取js，一个获取图片jpg。如果使用HTTP 1.1就是串行的，但是如果使用HTTP 2.0，就可以在一个连接里，客户端和服务端都可以同时发送多个请求或回应，而且不用按照顺序一对一对应。

![](<https://static001.geekbang.org/resource/image/9a/1a/9a54f97931377dyy2fde0de93f4ecf1a.jpeg?wh=1401*1080>)

HTTP 2.0其实是将三个请求变成三个流，将数据分成帧，乱序发送到一个TCP连接中。

![](<https://static001.geekbang.org/resource/image/3d/d3/3da001fac5701949b94e51caaee887d3.jpeg?wh=1920*651>)

HTTP 2.0成功解决了HTTP 1.1的队首阻塞问题，同时，也不需要通过HTTP 1.x的pipeline机制用多条TCP连接来实现并行请求与响应；减少了TCP连接数对服务器性能的影响，同时将页面的多个数据css、js、 jpg等通过一个数据链接进行传输，能够加快页面组件的传输速度。

## QUIC协议的“城会玩”

HTTP 2.0虽然大大增加了并发性，但还是有问题的。因为HTTP 2.0也是基于TCP协议的，TCP协议在处理包时是有严格顺序的。

当其中一个数据包遇到问题，TCP连接需要等待这个包完成重传之后才能继续进行。虽然HTTP 2.0通过多个stream，使得逻辑上一个TCP连接上的并行内容，进行多路数据的传输，然而这中间并没有关联的数据。一前一后，前面stream 2的帧没有收到，后面stream 1的帧也会因此阻塞。

于是，就又到了从TCP切换到UDP，进行“城会玩”的时候了。这就是Google的QUIC协议，接下来我们来看它是如何“城会玩”的。

### 机制一：自定义连接机制

我们都知道，一条TCP连接是由四元组标识的，分别是源 IP、源端口、目的 IP、目的端口。一旦一个元素发生变化时，就需要断开重连，重新连接。在移动互联情况下，当手机信号不稳定或者在WIFI和 移动网络切换时，都会导致重连，从而进行再次的三次握手，导致一定的时延。

这在TCP是没有办法的，但是基于UDP，就可以在QUIC自己的逻辑里面维护连接的机制，不再以四元组标识，而是以一个64位的随机数作为ID来标识，而且UDP是无连接的，所以当IP或者端口变化的时候，只要ID不变，就不需要重新建立连接。

### 机制二：自定义重传机制

前面我们讲过，TCP为了保证可靠性，通过使用**序号**和**应答**机制，来解决顺序问题和丢包问题。

任何一个序号的包发过去，都要在一定的时间内得到应答，否则一旦超时，就会重发这个序号的包。那怎么样才算超时呢？还记得我们提过的**自适应重传算法**吗？这个超时是通过**采样往返时间RTT**不断调整的。

其实，在TCP里面超时的采样存在不准确的问题。例如，发送一个包，序号为100，发现没有返回，于是再发送一个100，过一阵返回一个ACK101。这个时候客户端知道这个包肯定收到了，但是往返时间是多少呢？是ACK到达的时间减去后一个100发送的时间，还是减去前一个100发送的时间呢？事实是，第一种算法把时间算短了，第二种算法把时间算长了。

QUIC也有个序列号，是递增的。任何一个序列号的包只发送一次，下次就要加一了。例如，发送一个包，序号是100，发现没有返回；再次发送的时候，序号就是101了；如果返回的ACK 100，就是对第一个包的响应。如果返回ACK 101就是对第二个包的响应，RTT计算相对准确。

但是这里有一个问题，就是怎么知道包100和包101发送的是同样的内容呢？QUIC定义了一个offset概念。QUIC既然是面向连接的，也就像TCP一样，是一个数据流，发送的数据在这个数据流里面有个偏移量offset，可以通过offset查看数据发送到了哪里，这样只要这个offset的包没有来，就要重发；如果来了，按照offset拼接，还是能够拼成一个流。

![](<https://static001.geekbang.org/resource/image/80/2c/805aa4261yyb30a2a0e5a2f06ce5162c.jpeg?wh=1682*1080>)

### 机制三：无阻塞的多路复用

有了自定义的连接和重传机制，我们就可以解决上面HTTP 2.0的多路复用问题。

同HTTP 2.0一样，同一条QUIC连接上可以创建多个stream，来发送多个 HTTP 请求。但是，QUIC是基于UDP的，一个连接上的多个stream之间没有依赖。这样，假如stream2丢了一个UDP包，后面跟着stream3的一个UDP包，虽然stream2的那个包需要重传，但是stream3的包无需等待，就可以发给用户。

### 机制四：自定义流量控制

TCP的流量控制是通过**滑动窗口协议**。QUIC的流量控制也是通过window\_update，来告诉对端它可以接受的字节数。但是QUIC的窗口是适应自己的多路复用机制的，不但在一个连接上控制窗口，还在一个连接中的每个stream控制窗口。

还记得吗？在TCP协议中，接收端的窗口的起始点是下一个要接收并且ACK的包，即便后来的包都到了，放在缓存里面，窗口也不能右移，因为TCP的ACK机制是基于序列号的累计应答，一旦ACK了一个序列号，就说明前面的都到了，所以只要前面的没到，后面的到了也不能ACK，就会导致后面的到了，也有可能超时重传，浪费带宽。

QUIC的ACK是基于offset的，每个offset的包来了，进了缓存，就可以应答，应答后就不会重发，中间的空档会等待到来或者重发即可，而窗口的起始位置为当前收到的最大offset，从这个offset到当前的stream所能容纳的最大缓存，是真正的窗口大小。显然，这样更加准确。

![](<https://static001.geekbang.org/resource/image/a6/22/a66563b46906e7708cc69a02d43afb22.jpg?wh=795*422>)

另外，还有整个连接的窗口，需要对于所有的stream的窗口做一个统计。

## 小结

好了，今天就讲到这里，我们来总结一下：

- <span class="orange">HTTP协议虽然很常用，也很复杂，重点记住GET、POST、 PUT、DELETE这几个方法，以及重要的首部字段；</span>

- <span class="orange">HTTP 2.0通过头压缩、分帧、二进制编码、多路复用等技术提升性能；</span>

- <span class="orange">QUIC协议通过基于UDP自定义的类似TCP的连接、重试、多路复用、流量控制技术，进一步提升性能。</span>


<!-- -->

接下来，给你留两个思考题吧。

1. QUIC是一个精巧的协议，所以它肯定不止今天我提到的四种机制，你知道它还有哪些吗？

2. 这一节主要讲了如何基于HTTP浏览网页，如果要传输比较敏感的银行卡信息，该怎么办呢？


<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(108)

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34.jpeg)

  我那么圆

  http1.0的队首阻塞 对于同一个tcp连接，所有的http1.0请求放入队列中，只有前一个请求的响应收到了，然后才能发送下一个请求。 可见，http1.0的队首组塞发生在客户端。 3 http1.1的队首阻塞 对于同一个tcp连接，http1.1允许一次发送多个http1.1请求，也就是说，不必等前一个响应收到，就可以发送下一个请求，这样就解决了http1.0的客户端的队首阻塞。但是，http1.1规定，服务器端的响应的发送要根据请求被接收的顺序排队，也就是说，先接收到的请求的响应也要先发送。这样造成的问题是，如果最先收到的请求的处理时间长的话，响应生成也慢，就会阻塞已经生成了的响应的发送。也会造成队首阻塞。 可见，http1.1的队首阻塞发生在服务器端。 4 http2是怎样解决队首阻塞的 http2无论在客户端还是在服务器端都不需要排队，在同一个tcp连接上，有多个stream，由各个stream发送和接收http请求，各个steam相互独立，互不阻塞。 只要tcp没有人在用那么就可以发送已经生成的requst或者reponse的数据，在两端都不用等，从而彻底解决了http协议层面的队首阻塞问题。

  2019-01-29

  **11

  **138

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577423-6843.jpeg)

  月饼

  既然quic这么牛逼了干嘛还要tcp？

  2018-06-20

  **18

  **93

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  起风了001

  以前一直不是很确定Keep-Alive的作用, 今天结合tcp的知识, 终于是彻底搞清楚了. 其实就是浏览器访问服务端之后, 一个http请求的底层是tcp连接, tcp连接要经过三次握手之后,开始传输数据, 而且因为http设置了keep-alive,所以单次http请求完成之后这条tcp连接并不会断开, 而是可以让下一次http请求直接使用.当然keep-alive肯定也有timeout, 超时关闭.

  作者回复: 赞

  2019-05-21

  **6

  **59

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577423-6844.jpeg)

  鲍勃

  我是做底层的，传输层还是基本能hold住，现在有点扛不住了😂

  2018-06-18

  **13

  **41

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577423-6845.jpeg)

  云学

  http2.0的多路复用和4g协议的多harq并发类似，quick的关键改进是把Ack的含义给提纯净了，表达的含义是收到了包，而不是tcp的＂期望包＂

  2018-06-19

  **1

  **32

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577424-6846.jpeg)

  偷代码的bug农

  一窍不通，云里雾里

  2018-07-13

  **2

  **25

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577424-6847.jpeg)

  墨萧

  每次http都要经过TCP的三次握手四次挥手吗

  作者回复: keepalive就不用

  2018-06-18

  **2

  **23

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577424-6848.jpeg)

  Untitled

  QUIC说是基于UDP，无连接的，但是老师又说到是面向连接的，看的晕乎乎的。。

  作者回复: 讲tcp的时候讲了，所谓的连接其实是两边的状态，状态如果不在udp层维护，就可以在应用层维护

  2018-09-17

  **

  **23

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577425-6849.jpeg)

  skye

  “一前一后，前面 stream 2 的帧没有收到，后面 stream 1 的帧也会因此阻塞。”这个和队首阻塞的区别是啥，不太明白？

  作者回复: 一个是http层的，一个是tcp层的

  2018-06-20

  **3

  **15

- ![img](https://static001.geekbang.org/account/avatar/00/13/0f/c0/e6151cce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  花仙子

  UDP不用保持连接状态，不用建立更多socket，是不是就说服务端只能凭借客户端的源端口号来判定是客户端哪个应用发送的，是吗？

  作者回复: 是的

  2019-02-11

  **2

  **11

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577425-6851.jpeg)

  柯察金

  怎么说呢，感觉听了跟看书效果一样的，比较晦涩。因为平时接触比较多的就是 tcp http ，结果听了感觉对实际开发好像帮助不大，因为都是一个个知识点的感觉，像准备考试。希望能结合实际应用场景，讲解。

  作者回复: 太深入就不适合听了，所以定位还是入门

  2019-01-25

  **3

  **11

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577425-6852.jpeg)

  Summer___J

  老师，目前QUIC协议的典型应用场景是什么？这个协议听上去比TCP智能，随着技术演进、这个协议已经融合到TCP里面了还是仍然是一个独立的协议？

  2018-06-28

  **6

  **11

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577426-6853.jpeg)

  李小四

  网络_14: 开始看到文章有点懵： HTTP/2解决了HTTP/1.1队首阻塞问题.但是HTTP2是基于TCP的，TCP严格要求包的顺序，这不是矛盾吗？ 然后研究了一下，发现是不同层的队首阻塞： 1. 应用层的队首阻塞(HTTP/1.1-based head of line blocking): HTTP/1.1可以使用多个TCP连接，但对连接数依然有限制，一次请求要等到连接中其他请求完成后才能开始(Pipeline机制也没能解决好这个问题)，所以没有空闲连接的时候请求被阻塞，这是应用层的阻塞。 HTTP/2底层使用了一个TCP连接，上层虚拟了stream，HTTP请求跟stream打交道，无需等待前面的请求完成，这确实解决了应用层的队首阻塞问题。 2. 传输层的队首阻塞(TCP-based head of line blocking): 正如文中所述，TCP的应答是严格有序的，如果前面的包没到，即使后面的到了也不能应答，这样可能会导致后面的包被重传，窗口被“阻塞”在队首，这就是传输层的队首阻塞。 不管是HTTP/1.1还是HTTP/2，在传输层都是基于TCP，那么TCP层的队首阻塞问题都是存在的(只能由HTTP/3(based on QUIC)来解决了)，另外，HTTP/2用了单个TCP连接，那么在丢包率严重的场景，表现可能比HTTP/1.1更差。

  2020-10-22

  **

  **8

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577426-6854.jpeg)

  传说中的风一样

  cache control部分讲错了，max–age不是这么用的

  作者回复: 这里忘了说一下客户端的cache-control机制了，一个是客户端是否本地缓存过期。这里重点强调的类似varnish缓存服务器的行为

  2019-04-23

  **2

  **8

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577426-6855.jpeg)

  caohuan

  本篇所得：1.应用层 http1.0和2.0的使用，url通过DNS转化为IP地址，HTTP是基于TCP协议，TCP需要三次握手 进行连接，四次挥手 断开连接，而QUIC基于 UDP连接，不需要三次握手，节省资源，提高了性能和效率; 2.http 包括 请求行 、首部、正文实体，请求行 包含 方法、SP、URL、版本，首部 包含 首部字段名、SP、字段值、cr/if; 3.http 很复杂，它基于TCP协议，它包含 get获取资源，put 传输 修改的信息给服务器，post发送 新建的资源 给服务器，delete 删除资源; 4.http1.0为串联计算，http2.0可以并联计算，http2.0可以通过 头压缩 分帧、二进制编码、多路复用 来提示性能。 5.QUIC协议基于UDP自定义的类型，TCP的连接、重试、多路复用、流量控制 提升性能，对应四个机制，分别为 1）自定义连接机制 2）自定义重传机制 3）无阻塞多路复用 4）自定义流量控制。 回答老师的问题：第一个问题为提高性能，听老师后面的解答，第二个问题：输送敏感数据 比如银行信息 可以用 加密技术，公钥和私钥 保证安全性。

  2019-03-04

  **

  **8

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577427-6856.jpeg)

  意无尽

  哇，竟然坚持到这里了（虽然一半都还不到），虽然前面也有很多不懂。基本上从第二章开时候每一节都会花费一两个小时去理解，但是花费确实值啊，让我一个网络小白慢慢了解了网络的各个方面，感觉像是打开了另一个奇妙的世界！相当赞，后期还要刷第二遍！学完这个必须继续购买趣谈Linux操作系统！感谢刘超老师！

  作者回复: 谢谢

  2020-05-30

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/72/67/aa52812a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  stark

  有个地方不是很明白，就是里面说的流数据，比如，我在实际的应用里怎么查看下这些数据什么，比如像top这样的，怎么查看呢？

  作者回复: tcpdump，其实dump出来没有所谓的流，都是感觉上的流，还不是一个个的网络包

  2019-07-11

  **

  **4

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577427-6858.jpeg)

  heliang

  http2.0 并行传输同一个请求不同stream的时候，如果“”前面 stream 2 的帧没有收到，后面 stream1也会阻塞"，是阻塞在tcp重组上吗

  作者回复: 是的

  2019-04-12

  **

  **4

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577428-6859.jpeg)

  大灰狼

  quic推动了tls1.3，实现了tcp的序列号，流控，拥塞控制，重传，实现了http2的多路复用，流控等 参考infoq文章。 在实际进行中间人的过程中，quic是一个很头疼的协议。

  2018-09-09

  **

  **4

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577428-6860.jpeg)

  张张张 💭

  那个keepalive不是很理解，如果是这种模式，什么时候连接断开呢，怎么判断的，比如我访问淘宝首页，有很多http请求，是每个请求独立维护一个keepalive吗

  2018-06-29

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/de/bf/4df4224d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  shuifa![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  URI 属于 URL 更高层次的抽象，一种字符串文本标准。 就是说，URI 属于父类，而 URL 属于 URI 的子类。URL 是 URI 的一个子集。 二者的区别在于，URI 表示请求服务器的路径，定义这么一个资源。而 URL 同时说明要如何访问这个资源（http://）。

  2018-12-25

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/60/94/602b5746.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  雾满杨溪

  http和socket之间有什么关系？

  2018-06-20

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/4a/8a/c1069412.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  makermade

  服务器某个http应用存在大量fin_wait2怎么解决

  2018-06-19

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/58/97/8e14e7d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  楚人

  第13和14节有些难度，需要多看多听几遍

  2018-06-18

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/1d/12/13/e103a6e3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  扩散性百万咸面包

  原文说TCP层把IP信息装进IP头。。 正常的不应该是把TCP头装进去吗，然后传给IP层

  2020-04-04

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/15/04/bb/5e5c37c1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Angus

  我觉得很奇妙的事情是，看起来这么复杂的过程，竟然只要几十ms就可以做完，再想想我打完这行字得多少秒，它得做多少事情。

  2020-01-15

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/f9/90/f90903e5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  菜菜

  老师，为什么我看http1.x的pipeline是一条TCP连接，而您写的是多条

  2019-09-11

  **

  **2

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eqyicZYyW7ahaXgXUD8ZAS8x0t8jx5rYLhwbUCJiawRepKIZfsLdkxdQ9XQMo99c1UDibmNVfFnAqwPg/132)

  程序水果宝

  TCP协议为了防止发送过多的ack消耗大量带宽，采用累计确认机制。而谷歌的QUIC 的 ACK 是基于 offset 的，每个 offset 的包来了，进了缓存，就可以应答。每个offset包都应答，岂不是会大量消耗带宽？

  2019-08-30

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/5f/e8/94365887.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ieo

  有两个问题，麻烦老师回答下：1:一个页面请求了同一域名下的两个地址，如果没有用keep-alive，三次握手会进行两次吗？如果有了这个设置，就会进行一次吗？ 2:一个请求发到服务器后，服务器给客户端返回内容时，还要和客户端三次握手吗？如果还需要握手的话，为啥不能用客户端和服务器建立的tcp连接呢？

  作者回复: 没有keep-alive，每次都要握手，建立tcp连接。有了keep-alive就可以复用tcp连接了

  2019-05-05

  **3

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/a5/40/ad00a484.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  \- -

  “当其中一个数据包遇到问题，TCP 连接需要等待这个包完成重传之后才能继续进行。虽然 HTTP 2.0 通过多个 stream，使得逻辑上一个 TCP 连接上的并行内容，进行多路数据的传输，然而这中间并没有关联的数据。一前一后，前面 stream 2 的帧没有收到，后面 stream 1 的帧也会因此阻塞。” 请问这一段怎么理解？感觉和前面所述“http2.0的乱序传输支持”是不是有些矛盾？

  作者回复: 两层，一个是http层，一个是tcp层

  2019-02-05

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/b6/97/93e82345.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陆培尔

  QUIC协议自定义了连接机制、重传机制和滑动窗口协议，感觉这些都是原来tcp干的活，那为啥QUIC是属于应用层协议而不是传输层呢？应该把QUIC协议加入内核的网络栈而不是做在chrome这样的用户态应用程序里面吧

  作者回复: 基于UDP

  2019-01-16

  **4

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/cf/82/a3ea8076.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  tim

  有个问题： 文中指出tcp采样不准确。 序列号可以发送一致的。 但是之前讲的序列号是根据时间来增长的，除非你过去非常长的时间，不然是不可能重复的。 这个问题不知是我理解序列号的增长策略有问题还是文中作者的推断有问题🤨

  作者回复: 是同一个序列号发送两次，也就是说同一个包发送两次

  2018-10-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/72/fb/7e9ccf63.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  飞鹅普斯

  获取MAC地址时，局域网内的arp 广播跟发给网关的arp广播的广播内容有什么差异吗？

  2018-07-04

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a6/c1/d4e6147e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  一箭中的

  敏感信息可以使用https，记得http2天然具备https特性，所以https和http2都可以解决敏感信息问题。

  2018-06-19

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/0d/ad/3787a71a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古轩。

  敏感信息可以通过对报文RSA非对称加密来保证安全性能。

  2019-04-04

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/60/57/83f3a377.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  LiBloom

  请问：http协议有这么多 0.9、1、1.1、2、3 ，浏览器是怎么选择使用哪个协议跟服务器通信的呢？我看1.1开始有了协议协商，那1.1之前浏览器是有一个默认的协议吗？

  作者回复: 浏览器要兼容的，协议里面有协议号

  2019-03-06

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/8c/5e/eeaada1d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王鹏飞

  问个问题：http 请求头 的content type和 enctype有什么区别和联系

  2019-02-09

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/db/26/54f2c164.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  靠人品去赢

  @哪位不知道keepalive在哪哪位同学，开发者模式打开header那块，你会看到文章讲的几个属性。

  2019-01-18

  **

  **1

- ![img](%E7%AC%AC14%E8%AE%B2%20_%20HTTP%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%9C%8B%E4%B8%AA%E6%96%B0%E9%97%BB%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E9%BA%BB%E7%83%A6.resource/resize,m_fill,h_34,w_34-1662307577424-6848.jpeg)

  Untitled

  老师，您好，有个疑惑点：HTTP2.0 3个请求一定要创建3个流吗？每个流里面都要传输一次header帧？理解是要的，提到数据报文里面的请求行要确定方法和URL，3个请求的URL不同，方法类型也可能有差别，因此要分开3个header帧，但请问3个header帧可以放到同一个流里面吗？还是程序需要通过流来确定属于哪个请求的数据？

  2018-12-31

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/cb/f8/f4adadcb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陈天境

  刘超，你好，我们最近踩到一个Tomcat8.5的坑，就是HTTP响应本来应该是 HTTP/1.1 200 OK 但是Tomcat8.5 默认只会返回 HTTP/1.1 200 也是就是OK没掉了，好奇参考了HTTP协议的rfc文档，按照官方描述，是表示OK其实不是必须的意思吗？ https://tools.ietf.org/html/rfc7230#section-3.1.2 The first line of a response message is the status-line, consisting   of the protocol version, a space (SP), the status code, another   space, a possibly empty textual phrase describing the status code,   and ending with CRLF.

  2018-09-22

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/99/3873b659.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Hulk

  你好，Accept-Charset是针对字符的编码？如果请求的是文件，那二进制的编码是？

  作者回复: 不一定都需要这个头，看客户端和服务器怎么解析，怎么用

  2018-06-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b4/21/6d31d152.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  日天君

  QUIC是面向连接的，建立连接应该要双方约定id，这个过程是怎么做的？

  2018-06-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/a1/bb/b46e59b1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  evilKitsch

  新人报道😍

  2018-06-18

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/d1/f1/ffa93b4b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Wade

  请求各种： sp等英文 分别是什么含义？

  2022-06-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/27/af/29/3c27174a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  浪客剑心

  这里补充下HTTP1.1 pipeline模式的概念： 客户端 可以“流水线化”请求，即，在同一个TCP 连接上，发送多个请求而无需等待每个响应，服务端 必须按照与收到请求的相同顺序来向这些请求发送响应

  2022-06-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/02/d4/1e0bb504.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Peter

  在 1.1 的协议里面，默认是开启了 Keep-Alive 的 ---> 所以是通过Keep-Alive 来建立三次握手的？

  2022-05-29

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erXRaa98A3zjLDkOibUJV1254aQ4EYFTbSLJuEvD0nXicMNA8pLoxOfHf5kPTbGLXNicg8CPFH3Tn0mA/132)

  Geek_115bc8

  这个是大致把http总结了一遍。

  2022-05-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2b/00/fb/85b07045.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  肥柴

  终于看到不懵逼的地方了 

  2022-05-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/fe/2d/e23fc6ee.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  深水蓝

  早期google的chrome浏览器明显比其他浏览器的速度都快，是不是就是因为QUIC的加持呢？

  2022-01-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/65/b7/058276dc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  i_chase

  QUIC优点： 1 通过uuid维护连接状态，ip/port改变不需要重新连接 2 超时时间RTT采样准确（序号每次+1，offset不变） 3 基于UDP,多个stream不会互相影响阻塞 4 滑动窗口更加准确（非按序到达也进行确认），每个stream维护自己的窗口大小

  2022-01-03

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/QtSJo6NntUDXe45TLYaTic8WclQ2lFkVQxFGJMQLYtiabMQchSfFebLglFo8rcKiaHEMQOXia4mMOQaE8X1e3F9HqQ/132)

  Geek_873fe4

  传输层看得晕晕的，反倒是看到应用层能看得懂了。大概是因为我之前学了极客的另一门http协议课程吧

  2021-11-22

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLEIsgI4ub1VOKWtVOfouAzSqx8Yt8ibQEsAnwNJsJHmuJzzpQqG79HullvYwpic8hgiclgON2GwXSjw/132)

  cv0cv0

  如何查看一个网站是用http 2.0和quic传输的？

  2021-11-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/f5/0b/73628618.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  兔嘟嘟

  wiki和官网都说QUIC是传输层协议

  2021-08-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/fe/b4/295338e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Allan

  网络知识小白，感觉只是听过表面的这些名词，有的还没听过，原理更是不知道，听的也是云里雾里。。。

  2021-07-28

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/s4dusuWr2EbMAUklHUSLmFRlmysHJjt7xBBZVrQGMzmTT5Fc4y0hiaRR2svTy5UIZYclGVjmoDj7V7EG8JUsO9A/132)

  光悔

  两个重要问题被一笔带过了，麻烦能详细讲一下吗，第一个是http请求报文中请求行中的url到底是干什么的？一个是keepAlive的用途。

  2021-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/02/7b/eae23749.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  平头辉辉

  不知道怎么验证你说的对或不对，反正没听懂，怀疑你在乱说

  2021-05-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/df/e6/bd1b3c0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Jesse

  quic协议一点不懂，看不懂了。

  2021-05-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/99/95/1e332315.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_2014ce

  整体内容上还是有深度，有料的。不过感觉很多关键的点没有讲明白。整体听下来就有很多疑惑的点。

  2021-05-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/22/c1/8f/06db18d6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钟文娟

  TCP 的 ACK 机制是基于序列号的累计应答，一旦 ACK 了一个系列号，就说明前面的都到了，所以只要前面的没到，后面的到了也不能 ACK，这样好处可以合并ack包，但是quic每个接收包都要回ack包，缺点是不是增加了更多的ack包传输

  2021-03-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b2/e0/d856f5a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鱼

  详细的QUIC文档建议参考这篇文章，老师提到的部分只是告诉你QUIC是什么。《技术扫盲：新一代基于UDP的低延时网络传输层协议——QUIC详解》 链接:http://www.52im.net/thread-1309-1-1.html QUIC官方地址：http://www.chromium.org/quic

  2021-01-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/78/af/f5e68fd6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  生物第一

  问一个问题~ 如果应用层数据太大，那协议栈的分包是在哪一层或者哪几层？

  2020-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/69/81/01c2bde8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kid

  get 请求和post 请求 有什么区别呢？  网上大把大把的文献 弄得我有点乱  想听老师正统的回答 或者 给个 url 我去 get 下

  2020-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b7/b6/17103195.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Elliot

  机制四：自定义流量控制。 对于窗口的概念能详细解释下吗？

  2020-06-11

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/LWicoUend7QOH6pyXGJyJicAzm5T4TUD8TaicSCHVPJp7sbIicpeArcicZiaMGcQ7uUDWjGZYgVnUqNGFFDVe9h0EV4w/132)

  Geek_f7658e

  刘老师，您好！请问steam是什么？一个在tcp上的应用层程序？

  作者回复: 是一个逻辑的概念，应用层的逻辑概念

  2020-06-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/62/b3/edf0f4d9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  indeyo

  这里面，Retry-After 表示，告诉客户端应该在多长时间以后再次尝试一下。“503 错误”是说“服务暂时不再和这个值配合使用”。 这里不太懂，意思是如果是503，Retry-After暂时无效的意思吗？ 查了一下RFC文档，对503的描述如下： 6.6.4.  503 Service Unavailable    The 503 (Service Unavailable) status code indicates that the server   is currently unable to handle the request due to a temporary overload   or scheduled maintenance, which will likely be alleviated after some   delay.  The server MAY send a Retry-After header field   (Section 7.1.3) to suggest an appropriate amount of time for the   client to wait before retrying the request.       Note: The existence of the 503 status code does not imply that a      server has to use it when becoming overloaded.  Some servers might      simply refuse the connection. 理解一下，503是一个服务暂时不可用/负载的状态，可以用Retry-After来告诉用户多久以后重试。 看上去，像是支持大家在503的时候使用Retry-After。 所以这算是实际情况与理论存在一定差距的情况么?

  2020-05-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_057464

  请问tcp怎么开启多个stream，平时用的时候就一个输出流，一个输入流

  2020-05-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/94/0a/7f7c9b25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  宋健

  问题：如果要传输比较敏感的银行卡信息，该怎么办呢？ 回答：HTTPS可以保证信息安全吧？采用信息加密技术保证信息的安全性。

  2020-05-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/01/4b/a276c1d0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  儘管如此世界依然美麗

  我想请问一下：在上述所说的，Header帧会开启一个新的流进行传输，传输正文实体的是另一个流，那么TCP的另一端如何得知Data数据所对应的Header呢？通过流的ID吗？如果是的话，那么这个流ID是存放在哪里的？

  2020-05-11

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/YtfU7SyicGVWNZePKkoiaXt0nDAgD06TEZEsZyeJlhEUaUGpiaqKwXVNOBAplOoGB118SOJysrGjcEFVKaBUIBQOg/132)

  doraemonext

  “TCP 层发送每一个报文的时候，都需要加上自己的地址（即源地址）和它想要去的地方（即目标地址），将这两个信息放到 IP 头里面，交给 IP 层进行传输。” 老师有个问题想咨询下，加源地址和目标地址应该是 IP 网络层要处理的事情，还是就是在 TCP 层面完成了之后再交给的网络层？

  2020-05-05

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/ACAwW2gejNjQJnKzTQb3GXQibKbWSyRboxWgPU8UFAPicmwHbEHpAMyiaoWy6PSgYrmVtIXqZZhKJc0GDib8J6dycw/132)

  TheSunnyMan

  之前抓包分析过quic协议，用Chrome访问他家的视频网站是走的quic协议。还研究使用过v2ray，里面也支持quic传输。 不过针对单向的传输设备，比如关闸貌似这个协议不行。HTTPS over UDP

  2020-05-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/d9/ff/b23018a6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Heaven

  QUIC是基于UDP的,那么是否就不需要三次握手了呢?而是直接分配一个随机ID了,是这样吗?

  2020-04-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5d/11/40b47496.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李海涛

  刘老师，你好！HTTP 1.1 在应用层以纯文本的形式进行通信，但网页上图片或视频，在浏览器里面查看Response Header里面看到的HTTP 1.1/200 OK。请问这个怎么理解？谢谢

  2020-04-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/3f/a9127a73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  到了这一章节，原来不止我一个人觉得有些难度。是真的越到底层，越陌生呀。基础中蕴含着智慧！哈哈哈哈

  2020-04-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/6a/01/d9cb531d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  这得从我捡到一个鼠标垫开始说起

  放了好久继续开始看

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/b8/83/7c1ed918.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  如歌

  老师，再来一个趣谈 的课程吧，感谢老师，趣谈linux，让我入了个门，作为一个不从事底层开发的人员，趣谈去掉了我很多的疑惑

  2020-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/39/f9/b946719a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  King-ZJ

  对于应用层，比如说HTTP协议真的是了解甚少或者说是文盲，虽然平常生活中天天用，却对于其怎么去运作是那么无知。HTTP协议的发展和演进，是这个协议功能的完善，而为了让用户体验更好的服务，QUIC的出现很好地解决了丢包、重传、延时等问题。

  2020-03-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f0bd6b

  TCP详解刚结束，http又出发，真是刺激啊

  2020-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f2/13/3ee5a9b4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  chenzesam

  前端需要多学学

  2020-03-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/62/1e/ad721e61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  flow

  HTTP 2.0 虽然大大增加了并发性，但还是有问题的。因为 HTTP 2.0 也是基于 TCP 协议的，TCP 协议在处理包时是有严格顺序的。当其中一个数据包遇到问题，TCP 连接需要等待这个包完成重传之后才能继续进行。虽然 HTTP 2.0 通过多个 stream，使得逻辑上一个 TCP 连接上的并行内容，进行多路数据的传输，然而这中间并没有关联的数据。一前一后，前面 stream 2 的帧没有收到，后面 stream 1 的帧也会因此阻塞。-- 老师, 这里的意思是不是说, http2.0其实只是应用层的改变, 在tcp层还是跟http1.1一样的? 所以假如中间的一个数据包丢了, 由于tcp的累计应答机制, 实际上还是会有阻塞?

  2019-11-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/55/cb/1efe460a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  渴望做梦

  但是，HTTP 的服务器往往是不允许上传文件的 老师，你这句话是什么意思，服务器不是可以通过接口上传文件吗？

  2019-11-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/fd/91/65ff3154.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  _stuView

  假如 stream2 丢了一个 UDP 包，后面跟着 stream3 的一个 UDP 包，虽然 stream2 的那个包需要重传，但是 stream3 的包无需等待，就可以发给用户。 这一点有点奇怪，既然是使用UDP协议，那怎么还有重传机制呢？

  2019-11-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/57/91/3a082914.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  葡萄有点酸

  我发现了一个规律 像失败重连 丢包重试 包括keepalive断开 都会有个策略 一般是按照某种算法生成一个超时时长 超过该时长就采取一定的措施

  2019-09-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/75/16/4dd77397.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  a

  关于post和put方法和我理解的完全不一样.老师说的put是修改数据,post是创建数据.但我一直理解的是post是修改局部数据,put是创建和修改幂等数据,难道我一直都理解错了?

  作者回复: 都是约定呀。

  2019-08-08

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/BnzAUnic9oAGQQ7BkK3CujP109RAESq4WicDwbUIia1BQhMf8LsVHu3e6YKGaAHicbxnCw8sicUeic2V978ff74t1ReA/132)

  Geek_98b975

  http 2.0有点像sctp呢

  2019-07-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c1/0e/2b987d54.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  蜉蝣

  老师，如你所说：“而 QUIC 以一个 64 位的随机数作为 ID 来标识，即使 IP 或者端口发生变化，只要 ID 不变，就不需要重新建立连接。” 那我是不是可以认为，QUIC 的安全性比 TCP 低。

  作者回复: TCP的安全你指啥

  2019-06-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4c/0c/ada45f25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  憎爱不关心

  看完这一篇 不敢看新闻了。。。

  作者回复: 完了，等看完后面的，不敢下小电影了

  2019-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/b4/94/2796de72.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  追风筝的人

  如果要传输银行卡等敏感信息  使用TCP+SSL协议吗？

  作者回复: 当然

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/e9/29/629d9bb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天王

  1 http协议是处于应用层的协议，一般的url是统一资源定位符，http代表了http协议，后面的是域名，dns服务器解析成ip，最终请求的ip地址。2 http协议的请求报文，由三部分组成，请求行，首部，和请求体三部分组成，请求行包括get,post方法，url，版本等。首部是键值对，中间用冒号分隔，Accept_charset，content_type，Cache_controll，当max-age比指定的缓存小，从缓存取，否则转发到应用层。3 http请求发送，http基于tcp协议的，是面向连接的方式发送请求，发送的是二进制流，tcp将二进制流转化成一个个的报文发送给服务器。tcp发送之前会把源ip和目标ip放进ip头里面，交给ip层进行传输，目标服务器收到之后会发一个ack，交给相应的应用进行处理，应用的进程收到请求，进行处理。4 http返回由三部分组成，状态行，首部和响应体，状态行包括版本，状态码，首部是键值对，包括Content-Type，构造好了返回的http报文，交给socket去发送，交给tcp层，tcp将返回的报文分成一个个的段，保证每个段都发出去，客户端接收到，发现是200，则进行处理。5 http2.0和1.1的区别，5.1 http1.1每次都是完整的请求头发送，http2.0相同的请求头会创建索引，发送只发送索引 5.2 http2.0将一个tcp连接切成多个流，每个流有一个流id，流是有优先级的，http还将传输消息切割为更小的消息和帧，采用二进制格式编码，有Head帧，会开启新的流，有Data帧，用来传输正文实体，多个Data帧属于同一个流。http2.0的客户端可以将多个请求拆成多个不同的流中，将请求内容拆成帧，有id和优先级，服务端接收之后按照帧首部的流标识进行组装，优先级高的先处理。5.3http2.0解决了阻塞问题，占用了一个tcp连接，对服务器资源占用不多，和http1.1相比，在一个链接里进行传输，响应比较快。5.4QUIC协议，tcp连接存在问题，虽然实现了多路传输，因为保证顺序，一个帧传输失败，要进行重试，组装的时候还是要等待，QUIC协议又切回了UDP，5.41自定义连接机制，一个tcp连接是以四元组标识的，源ip,端口号，目标ip和端口号，一旦一个元素变化，手机信号不好或者切换网络就要进行重连，产生时延，udp以64位的随机数作为标识，ip和端口发生变化不影响5.42QUIC发送包的id是自增的，如果判断重试的包和第一次发送失败的包是同一个包，发送的数据在流里有个偏移量offset，通过offset判断包发送到哪里了，重试的包来了按照offset拼接流，5.43 多路传输无阻塞5.4.3自定义流量控制tcp是通过滑动窗口协议，udp通过windows_update，来告诉客户端可以接收的字节数，窗口适应多路复用机制，在连接和stream上控制窗口

  2019-06-04

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_f6f02b

  HTTP 2.0 虽然大大增加了并发性，但还是有问题的。因为 HTTP 2.0 也是基于 TCP 协议的，TCP 协议在处理包时是有严格顺序的。当其中一个数据包遇到问题，TCP 连接需要等待这个包完成重传之后才能继续进行。这里指的严格顺序是否是因为受到接受方同时最多接受字节限制导致要顺序，也就是说在接受方限制字符内是无须顺序的，可以先发的后接收，后发的先接收，是吗？

  作者回复: 严格顺序是TCP协议要求顺序的。但是同一个连续的buffer填满发往应用层的时候，这里面可以有多个通道的

  2019-04-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/fc/24/122142cd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bearsmall

  打卡

  2019-03-10

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJGOSxM1GIHX9Y2JIe7vGQ87rK8xpo5F03KmiaGyXeKnozZsicHeSZrbSlzUVhTOdDlXCkTrcYNIVJg/132)

  ferry

  我感觉QUIC是TCP和UDP长处的结合体，但是既然TCP协议下的数据也是流的形式，为什么TCP协议不采用offset的方式来标记数据，而要采用序号来标记呢？

  作者回复: 当时设计的时候，没想到有这些问题

  2019-02-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/c0/2a/86ca523d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  shaohsiung

  keepalive和pipeline有什么区别？

  作者回复: 一个是探活，keepalive。pipeline是发送请求的时候，不用一来一回，可以连着发

  2019-02-18

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/6f/63/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  扬～

  每天两篇，每天都有问题要问。 IP层可以通过头部的ID，长度和偏移量来重组数据报，那么TCP如何根据哪些字段来重组一个应用数据包从而交给应用层呢。

  2018-11-02

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/3IrblACSCxr7ianvicQXRexScIaZ1zXVQYc1eLUlia6WkhPNKPzMoIJRgfVHe1BHskfTx8E9FCmicYGCeZic6HrGbRA/132)

  我爱探索

  quic tcp_bbr详解增加个补充

  2018-10-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5b/66/ad35bc68.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  党

  只可惜quic没有普及呢 http2.0也没有普及呢 作为了web开发者 这节看的没啥难度

  2018-09-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/19/a6/4529cdfa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  功夫熊猫

  quic目前应用的场景多吗，主要是哪些地方会用到

  2018-09-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b3/df/0fabb233.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  闪客sun

  老师，就是浏览器构建完http请求，封装进tcp头里这一步是浏览器完成的，那再把这个tcp数据封装在网络层里，这一步是谁完成的？是底层操作系统么？如果我想从头到尾完全自己封装一段数据直接从网口发出去，我需要怎么做呀？谢谢

  2018-08-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/65/70/7e137498.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FOCUS

  1、不了解 QUIC； 2、银行卡敏感信息，可以在客户端js加密传送，还可以对表单做一个token。

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/65/70/7e137498.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FOCUS

  \#QUIC 的流量控制也是通过 window_update，来告诉对端它可以接受的字节数。# 对端是什么来的？

  2018-07-24

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/65/70/7e137498.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FOCUS

  \#例如，发送一个包，序号为 100，发现没有返回，于是再发送一个 100，过一阵返回一个 ACK101。# 这里包的序号不应该是 101 吗？不然，怎么返回了 ACK101？

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/65/70/7e137498.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FOCUS

  \#HTTP 2.0 成功解决了 HTTP 1.1 的队首阻塞问题#，请问“队首阻塞”是否为笔误？不是队列？

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/65/70/7e137498.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FOCUS

  【另外，HTTP 2.0 协议将一个 TCP 的连接中，切分成多个流，每个流都有自己的 ID，而且流可以是客户端发往服务端，也可以是服务端发往客户端。】 “切分多个流”—这是指将一个tcp链接切分为流形式？而不是一个 socket 方式？

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/65/70/7e137498.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  FOCUS

  统一资源定位符 不是 URI吗？ URI 与 URL 两个名词有什么区别吗？

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4a/8a/c1069412.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  makermade

  超哥。服务器存在大量time_wait的连接，是网络问题，还是web应用问题？

  作者回复: 多为应用问题

  2018-06-29

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  唐唐

  我听了3遍了，还得在看几遍，才行，想问下，在哪里可以看是否有keepalive，是不是可以理解为一个网站请求了一个地址，请求该网站的其它地址不用再次三次握手了，keepalive是对该网站域名的标识吗？

  2018-06-20

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/80/b5/f59d92f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Cloud

  HTTPS吧😀

  2018-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/52/e0/ef42c4ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓.光

  敏感信息那就HTTPS吧

  2018-06-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/1b/ac/41ec8c80.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  孙悟空

  问题2 https

  2018-06-18
