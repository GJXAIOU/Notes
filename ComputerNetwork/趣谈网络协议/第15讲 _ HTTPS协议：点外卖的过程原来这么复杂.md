# 第15讲 \| HTTPS协议：点外卖的过程原来这么复杂

作者: 刘超

完成时间:

总结时间:



<audio><source src="https://static001.geekbang.org/resource/audio/f3/8d/f32cb9ab428dd51e209d6c1ce260ae8d.mp3" type="audio/mpeg"></audio>

用HTTP协议，看个新闻还没有问题，但是换到更加严肃的场景中，就存在很多的安全风险。例如，你要下单做一次支付，如果还是使用普通的HTTP协议，那你很可能会被黑客盯上。

你发送一个请求，说我要点个外卖，但是这个网络包被截获了，于是在服务器回复你之前，黑客先假装自己就是外卖网站，然后给你回复一个假的消息说：“好啊好啊，来来来，银行卡号、密码拿来。”如果这时候你真把银行卡密码发给它，那你就真的上套了。

那怎么解决这个问题呢？当然一般的思路就是**加密**。加密分为两种方式一种是**对称加密**，一种是**非对称加密**。

在对称加密算法中，加密和解密使用的密钥是相同的。也就是说，加密和解密使用的是同一个密钥。因此，对称加密算法要保证安全性的话，密钥要做好保密。只能让使用的人知道，不能对外公开。

在非对称加密算法中，加密使用的密钥和解密使用的密钥是不相同的。一把是作为公开的公钥，另一把是作为谁都不能给的私钥。公钥加密的信息，只有私钥才能解密。私钥加密的信息，只有公钥才能解密。

因为对称加密算法相比非对称加密算法来说，效率要高得多，性能也好，所以交互的场景下多用对称加密。

## 对称加密

假设你和外卖网站约定了一个密钥，你发送请求的时候用这个密钥进行加密，外卖网站用同样的密钥进行解密。这样就算中间的黑客截获了你的请求，但是它没有密钥，还是破解不了。

<!-- [[[read_end]]] -->

这看起来很完美，但是中间有个问题，你们两个怎么来约定这个密钥呢？如果这个密钥在互联网上传输，也是很有可能让黑客截获的。黑客一旦截获这个秘钥，它可以佯作不知，静静地等着你们两个交互。这时候你们之间互通的任何消息，它都能截获并且查看，就等你把银行卡账号和密码发出来。

我们在谍战剧里面经常看到这样的场景，就是特工破译的密码会有个密码本，截获无线电台，通过密码本就能将原文破解出来。怎么把密码本给对方呢？只能通过**线下传输**。

比如，你和外卖网站偷偷约定时间地点，它给你一个纸条，上面写着你们两个的密钥，然后说以后就用这个密钥在互联网上定外卖了。当然你们接头的时候，也会先约定一个口号，什么“天王盖地虎”之类的，口号对上了，才能把纸条给它。但是，“天王盖地虎”同样也是对称加密密钥，同样存在如何把“天王盖地虎”约定成口号的问题。而且在谍战剧中一对一接头可能还可以，在互联网应用中，客户太多，这样是不行的。

## 非对称加密

所以，只要是对称加密，就会永远在这个死循环里出不来，这个时候，就需要非对称加密介入进来。

非对称加密的私钥放在外卖网站这里，不会在互联网上传输，这样就能保证这个密钥的私密性。但是，对应私钥的公钥，是可以在互联网上随意传播的，只要外卖网站把这个公钥给你，你们就可以愉快地互通了。

比如说你用公钥加密，说“我要定外卖”，黑客在中间就算截获了这个报文，因为它没有私钥也是解不开的，所以这个报文可以顺利到达外卖网站，外卖网站用私钥把这个报文解出来，然后回复，“那给我银行卡和支付密码吧”。

先别太乐观，这里还是有问题的。回复的这句话，是外卖网站拿私钥加密的，互联网上人人都可以把它打开，当然包括黑客。那外卖网站可以拿公钥加密吗？当然不能，因为它自己的私钥只有它自己知道，谁也解不开。

另外，这个过程还有一个问题，黑客也可以模拟发送“我要定外卖”这个过程的，因为它也有外卖网站的公钥。

为了解决这个问题，看来一对公钥私钥是不够的，客户端也需要有自己的公钥和私钥，并且客户端要把自己的公钥，给外卖网站。

这样，客户端给外卖网站发送的时候，用外卖网站的公钥加密。而外卖网站给客户端发送消息的时候，使用客户端的公钥。这样就算有黑客企图模拟客户端获取一些信息，或者半路截获回复信息，但是由于它没有私钥，这些信息它还是打不开。

## 数字证书

不对称加密也会有同样的问题，如何将不对称加密的公钥给对方呢？一种是放在一个公网的地址上，让对方下载；另一种就是在建立连接的时候，传给对方。

这两种方法有相同的问题，那就是，作为一个普通网民，你怎么鉴别别人给你的公钥是对的。会不会有人冒充外卖网站，发给你一个它的公钥。接下来，你和它所有的互通，看起来都是没有任何问题的。毕竟每个人都可以创建自己的公钥和私钥。

例如，我自己搭建了一个网站cliu8site，可以通过这个命令先创建私钥。

```
openssl genrsa -out cliu8siteprivate.key 1024
```

然后，再根据这个私钥，创建对应的公钥。

```
openssl rsa -in cliu8siteprivate.key -pubout -outcliu8sitepublic.pem
```

这个时候就需要权威部门的介入了，就像每个人都可以打印自己的简历，说自己是谁，但是有公安局盖章的，就只有户口本，这个才能证明你是你。这个由权威部门颁发的称为**证书**（**Certificate**）。

证书里面有什么呢？当然应该有**公钥**，这是最重要的；还有证书的**所有者**，就像户口本上有你的姓名和身份证号，说明这个户口本是你的；另外还有证书的**发布机构**和证书的**有效期**，这个有点像身份证上的机构是哪个区公安局，有效期到多少年。

这个证书是怎么生成的呢？会不会有人假冒权威机构颁发证书呢？就像有假身份证、假户口本一样。生成证书需要发起一个证书请求，然后将这个请求发给一个权威机构去认证，这个权威机构我们称为**CA**（ **Certificate Authority**）。

证书请求可以通过这个命令生成。

```
openssl req -key cliu8siteprivate.key -new -out cliu8sitecertificate.req
```

将这个请求发给权威机构，权威机构会给这个证书卡一个章，我们称为**签名算法。**问题又来了，那怎么签名才能保证是真的权威机构签名的呢？当然只有用只掌握在权威机构手里的东西签名了才行，这就是CA的私钥。

签名算法大概是这样工作的：一般是对信息做一个Hash计算，得到一个Hash值，这个过程是不可逆的，也就是说无法通过Hash值得出原来的信息内容。在把信息发送出去时，把这个Hash值加密后，作为一个签名和信息一起发出去。

权威机构给证书签名的命令是这样的。

```
openssl x509 -req -in cliu8sitecertificate.req -CA cacertificate.pem -CAkey caprivate.key -out cliu8sitecertificate.pem
```

这个命令会返回Signature ok，而cliu8sitecertificate.pem就是签过名的证书。CA用自己的私钥给外卖网站的公钥签名，就相当于给外卖网站背书，形成了外卖网站的证书。

我们来查看这个证书的内容。

```
openssl x509 -in cliu8sitecertificate.pem -noout -text
```

这里面有个Issuer，也即证书是谁颁发的；Subject，就是证书颁发给谁；Validity是证书期限；Public-key是公钥内容；Signature Algorithm是签名算法。

这下好了，你不会从外卖网站上得到一个公钥，而是会得到一个证书，这个证书有个发布机构CA，你只要得到这个发布机构CA的公钥，去解密外卖网站证书的签名，如果解密成功了，Hash也对的上，就说明这个外卖网站的公钥没有啥问题。

你有没有发现，又有新问题了。要想验证证书，需要CA的公钥，问题是，你怎么确定CA的公钥就是对的呢？

所以，CA的公钥也需要更牛的CA给它签名，然后形成CA的证书。要想知道某个CA的证书是否可靠，要看CA的上级证书的公钥，能不能解开这个CA的签名。就像你不相信区公安局，可以打电话问市公安局，让市公安局确认区公安局的合法性。这样层层上去，直到全球皆知的几个著名大CA，称为**root CA**，做最后的背书。通过这种**层层授信背书**的方式，从而保证了非对称加密模式的正常运转。

除此之外，还有一种证书，称为**Self-Signed Certificate**，就是自己给自己签名。这个给人一种“我就是我，你爱信不信”的感觉。这里我就不多说了。

## HTTPS的工作模式

我们可以知道，非对称加密在性能上不如对称加密，那是否能将两者结合起来呢？例如，公钥私钥主要用于传输对称加密的秘钥，而真正的双方大数据量的通信都是通过对称加密进行的。

当然是可以的。这就是HTTPS协议的总体思路。

![](<https://static001.geekbang.org/resource/image/df/b4/df1685dd308cef1db97e91493f911ab4.jpg?wh=2285*4076>)

当你登录一个外卖网站的时候，由于是HTTPS，客户端会发送Client Hello消息到服务器，以明文传输TLS版本信息、加密套件候选列表、压缩算法候选列表等信息。另外，还会有一个随机数，在协商对称密钥的时候使用。

这就类似在说：“您好，我想定外卖，但你要保密我吃的是什么。这是我的加密套路，再给你个随机数，你留着。”

然后，外卖网站返回Server Hello消息, 告诉客户端，服务器选择使用的协议版本、加密套件、压缩算法等，还有一个随机数，用于后续的密钥协商。

这就类似在说：“您好，保密没问题，你的加密套路还挺多，咱们就按套路2来吧，我这里也有个随机数，你也留着。”

然后，外卖网站会给你一个服务器端的证书，然后说：“Server Hello Done，我这里就这些信息了。”

你当然不相信这个证书，于是你从自己信任的CA仓库中，拿CA的证书里面的公钥去解密外卖网站的证书。如果能够成功，则说明外卖网站是可信的。这个过程中，你可能会不断往上追溯CA、CA的CA、CA的CA的CA，反正直到一个授信的CA，就可以了。

证书验证完毕之后，觉得这个外卖网站可信，于是客户端计算产生随机数字Pre-master，发送Client Key Exchange，用证书中的公钥加密，再发送给服务器，服务器可以通过私钥解密出来。

到目前为止，无论是客户端还是服务器，都有了三个随机数，分别是：自己的、对端的，以及刚生成的Pre-Master随机数。通过这三个随机数，可以在客户端和服务器产生相同的对称密钥。

有了对称密钥，客户端就可以说：“Change Cipher Spec，咱们以后都采用协商的通信密钥和加密算法进行加密通信了。”

然后发送一个Encrypted Handshake Message，将已经商定好的参数等，采用协商密钥进行加密，发送给服务器用于数据与握手验证。

同样，服务器也可以发送Change Cipher Spec，说：“没问题，咱们以后都采用协商的通信密钥和加密算法进行加密通信了”，并且也发送Encrypted Handshake Message的消息试试。当双方握手结束之后，就可以通过对称密钥进行加密传输了。

这个过程除了加密解密之外，其他的过程和HTTP是一样的，过程也非常复杂。

上面的过程只包含了HTTPS的单向认证，也即客户端验证服务端的证书，是大部分的场景，也可以在更加严格安全要求的情况下，启用双向认证，双方互相验证证书。

## 重放与篡改

其实，这里还有一些没有解决的问题，例如重放和篡改的问题。

没错，有了加密和解密，黑客截获了包也打不开了，但是它可以发送N次。这个往往通过Timestamp和Nonce随机数联合起来，然后做一个不可逆的签名来保证。

Nonce随机数保证唯一，或者Timestamp和Nonce合起来保证唯一，同样的，请求只接受一次，于是服务器多次收到相同的Timestamp和Nonce，则视为无效即可。

如果有人想篡改Timestamp和Nonce，还有签名保证不可篡改性，如果改了用签名算法解出来，就对不上了，可以丢弃了。

## 小结

好了，这一节就到这里了，我们来总结一下。

- <span class="orange">加密分对称加密和非对称加密。对称加密效率高，但是解决不了密钥传输问题；非对称加密可以解决这个问题，但是效率不高。</span>

- <span class="orange"> 非对称加密需要通过证书和权威机构来验证公钥的合法性。</span>

- <span class="orange">HTTPS是综合了对称加密和非对称加密算法的HTTP协议。既保证传输安全，也保证传输效率。</span>


<!-- -->

最后，给你留两个思考题：

1. HTTPS协议比较复杂，沟通过程太繁复，这样会导致效率问题，那你知道有哪些手段可以解决这些问题吗？

2. HTTP和HTTPS协议的正文部分传输个JSON什么的还好，如果播放视频，就有问题了，那这个时候，应该使用什么协议呢？


<!-- -->

欢迎你留言和我讨论。趣谈网络协议，我们下期见！

## 精选留言(108)

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34.jpeg)

  我那么圆

  http1.0的队首阻塞 对于同一个tcp连接，所有的http1.0请求放入队列中，只有前一个请求的响应收到了，然后才能发送下一个请求。 可见，http1.0的队首组塞发生在客户端。 3 http1.1的队首阻塞 对于同一个tcp连接，http1.1允许一次发送多个http1.1请求，也就是说，不必等前一个响应收到，就可以发送下一个请求，这样就解决了http1.0的客户端的队首阻塞。但是，http1.1规定，服务器端的响应的发送要根据请求被接收的顺序排队，也就是说，先接收到的请求的响应也要先发送。这样造成的问题是，如果最先收到的请求的处理时间长的话，响应生成也慢，就会阻塞已经生成了的响应的发送。也会造成队首阻塞。 可见，http1.1的队首阻塞发生在服务器端。 4 http2是怎样解决队首阻塞的 http2无论在客户端还是在服务器端都不需要排队，在同一个tcp连接上，有多个stream，由各个stream发送和接收http请求，各个steam相互独立，互不阻塞。 只要tcp没有人在用那么就可以发送已经生成的requst或者reponse的数据，在两端都不用等，从而彻底解决了http协议层面的队首阻塞问题。

  2019-01-29

  **11

  **138

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7f/27/03007a5e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

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

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/28/1e307312.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  鲍勃

  我是做底层的，传输层还是基本能hold住，现在有点扛不住了😂

  2018-06-18

  **13

  **41

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615552-7135.jpeg)

  云学

  http2.0的多路复用和4g协议的多harq并发类似，quick的关键改进是把Ack的含义给提纯净了，表达的含义是收到了包，而不是tcp的＂期望包＂

  2018-06-19

  **1

  **32

- ![img](https://static001.geekbang.org/account/avatar/00/11/47/90/6828af58.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  偷代码的bug农

  一窍不通，云里雾里

  2018-07-13

  **2

  **25

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c0/25/348b4d76.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  墨萧

  每次http都要经过TCP的三次握手四次挥手吗

  作者回复: keepalive就不用

  2018-06-18

  **2

  **23

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  QUIC说是基于UDP，无连接的，但是老师又说到是面向连接的，看的晕乎乎的。。

  作者回复: 讲tcp的时候讲了，所谓的连接其实是两边的状态，状态如果不在udp层维护，就可以在应用层维护

  2018-09-17

  **

  **23

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615552-7139.jpeg)

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

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615553-7141.jpeg)

  柯察金

  怎么说呢，感觉听了跟看书效果一样的，比较晦涩。因为平时接触比较多的就是 tcp http ，结果听了感觉对实际开发好像帮助不大，因为都是一个个知识点的感觉，像准备考试。希望能结合实际应用场景，讲解。

  作者回复: 太深入就不适合听了，所以定位还是入门

  2019-01-25

  **3

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/11/53/6f/6051e0f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Summer___J

  老师，目前QUIC协议的典型应用场景是什么？这个协议听上去比TCP智能，随着技术演进、这个协议已经融合到TCP里面了还是仍然是一个独立的协议？

  2018-06-28

  **6

  **11

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615553-7143.jpeg)

  李小四

  网络_14: 开始看到文章有点懵： HTTP/2解决了HTTP/1.1队首阻塞问题.但是HTTP2是基于TCP的，TCP严格要求包的顺序，这不是矛盾吗？ 然后研究了一下，发现是不同层的队首阻塞： 1. 应用层的队首阻塞(HTTP/1.1-based head of line blocking): HTTP/1.1可以使用多个TCP连接，但对连接数依然有限制，一次请求要等到连接中其他请求完成后才能开始(Pipeline机制也没能解决好这个问题)，所以没有空闲连接的时候请求被阻塞，这是应用层的阻塞。 HTTP/2底层使用了一个TCP连接，上层虚拟了stream，HTTP请求跟stream打交道，无需等待前面的请求完成，这确实解决了应用层的队首阻塞问题。 2. 传输层的队首阻塞(TCP-based head of line blocking): 正如文中所述，TCP的应答是严格有序的，如果前面的包没到，即使后面的到了也不能应答，这样可能会导致后面的包被重传，窗口被“阻塞”在队首，这就是传输层的队首阻塞。 不管是HTTP/1.1还是HTTP/2，在传输层都是基于TCP，那么TCP层的队首阻塞问题都是存在的(只能由HTTP/3(based on QUIC)来解决了)，另外，HTTP/2用了单个TCP连接，那么在丢包率严重的场景，表现可能比HTTP/1.1更差。

  2020-10-22

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/11/a7/92/d4ce2462.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  传说中的风一样

  cache control部分讲错了，max–age不是这么用的

  作者回复: 这里忘了说一下客户端的cache-control机制了，一个是客户端是否本地缓存过期。这里重点强调的类似varnish缓存服务器的行为

  2019-04-23

  **2

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/ec/af6d0b10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  caohuan

  本篇所得：1.应用层 http1.0和2.0的使用，url通过DNS转化为IP地址，HTTP是基于TCP协议，TCP需要三次握手 进行连接，四次挥手 断开连接，而QUIC基于 UDP连接，不需要三次握手，节省资源，提高了性能和效率; 2.http 包括 请求行 、首部、正文实体，请求行 包含 方法、SP、URL、版本，首部 包含 首部字段名、SP、字段值、cr/if; 3.http 很复杂，它基于TCP协议，它包含 get获取资源，put 传输 修改的信息给服务器，post发送 新建的资源 给服务器，delete 删除资源; 4.http1.0为串联计算，http2.0可以并联计算，http2.0可以通过 头压缩 分帧、二进制编码、多路复用 来提示性能。 5.QUIC协议基于UDP自定义的类型，TCP的连接、重试、多路复用、流量控制 提升性能，对应四个机制，分别为 1）自定义连接机制 2）自定义重传机制 3）无阻塞多路复用 4）自定义流量控制。 回答老师的问题：第一个问题为提高性能，听老师后面的解答，第二个问题：输送敏感数据 比如银行信息 可以用 加密技术，公钥和私钥 保证安全性。

  2019-03-04

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/12/d6/39/6b45878d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

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

- ![img](https://static001.geekbang.org/account/avatar/00/12/5f/dc/d16e0923.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  heliang

  http2.0 并行传输同一个请求不同stream的时候，如果“”前面 stream 2 的帧没有收到，后面 stream1也会阻塞"，是阻塞在tcp重组上吗

  作者回复: 是的

  2019-04-12

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/be/8c/a5a3788f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大灰狼

  quic推动了tls1.3，实现了tcp的序列号，流控，拥塞控制，重传，实现了http2的多路复用，流控等 参考infoq文章。 在实际进行中间人的过程中，quic是一个很头疼的协议。

  2018-09-09

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/49/86/554cc50e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张张张 💭

  那个keepalive不是很理解，如果是这种模式，什么时候连接断开呢，怎么判断的，比如我访问淘宝首页，有很多http请求，是每个请求独立维护一个keepalive吗

  2018-06-29

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/de/bf/4df4224d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  shuifa![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  URI 属于 URL 更高层次的抽象，一种字符串文本标准。 就是说，URI 属于父类，而 URL 属于 URI 的子类。URL 是 URI 的一个子集。 二者的区别在于，URI 表示请求服务器的路径，定义这么一个资源。而 URL 同时说明要如何访问这个资源（http://）。

  2018-12-25

  **

  **3

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7152.jpeg)

  雾满杨溪

  http和socket之间有什么关系？

  2018-06-20

  **2

  **3

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7153.jpeg)

  makermade

  服务器某个http应用存在大量fin_wait2怎么解决

  2018-06-19

  **2

  **3

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7154.jpeg)

  楚人

  第13和14节有些难度，需要多看多听几遍

  2018-06-18

  **

  **3

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7155.jpeg)

  扩散性百万咸面包

  原文说TCP层把IP信息装进IP头。。 正常的不应该是把TCP头装进去吗，然后传给IP层

  2020-04-04

  **1

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7156.jpeg)

  Angus

  我觉得很奇妙的事情是，看起来这么复杂的过程，竟然只要几十ms就可以做完，再想想我打完这行字得多少秒，它得做多少事情。

  2020-01-15

  **

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7157.jpeg)

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

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7158.jpeg)

  ieo

  有两个问题，麻烦老师回答下：1:一个页面请求了同一域名下的两个地址，如果没有用keep-alive，三次握手会进行两次吗？如果有了这个设置，就会进行一次吗？ 2:一个请求发到服务器后，服务器给客户端返回内容时，还要和客户端三次握手吗？如果还需要握手的话，为啥不能用客户端和服务器建立的tcp连接呢？

  作者回复: 没有keep-alive，每次都要握手，建立tcp连接。有了keep-alive就可以复用tcp连接了

  2019-05-05

  **3

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7159.jpeg)

  \- -

  “当其中一个数据包遇到问题，TCP 连接需要等待这个包完成重传之后才能继续进行。虽然 HTTP 2.0 通过多个 stream，使得逻辑上一个 TCP 连接上的并行内容，进行多路数据的传输，然而这中间并没有关联的数据。一前一后，前面 stream 2 的帧没有收到，后面 stream 1 的帧也会因此阻塞。” 请问这一段怎么理解？感觉和前面所述“http2.0的乱序传输支持”是不是有些矛盾？

  作者回复: 两层，一个是http层，一个是tcp层

  2019-02-05

  **

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7160.jpeg)

  陆培尔

  QUIC协议自定义了连接机制、重传机制和滑动窗口协议，感觉这些都是原来tcp干的活，那为啥QUIC是属于应用层协议而不是传输层呢？应该把QUIC协议加入内核的网络栈而不是做在chrome这样的用户态应用程序里面吧

  作者回复: 基于UDP

  2019-01-16

  **4

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7161.jpeg)

  tim

  有个问题： 文中指出tcp采样不准确。 序列号可以发送一致的。 但是之前讲的序列号是根据时间来增长的，除非你过去非常长的时间，不然是不可能重复的。 这个问题不知是我理解序列号的增长策略有问题还是文中作者的推断有问题🤨

  作者回复: 是同一个序列号发送两次，也就是说同一个包发送两次

  2018-10-30

  **

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7162.jpeg)

  飞鹅普斯

  获取MAC地址时，局域网内的arp 广播跟发给网关的arp广播的广播内容有什么差异吗？

  2018-07-04

  **2

  **2

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7163.jpeg)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7165.jpeg)

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

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7167.jpeg)

  靠人品去赢

  @哪位不知道keepalive在哪哪位同学，开发者模式打开header那块，你会看到文章讲的几个属性。

  2019-01-18

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/dc/68/006ba72c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Untitled

  老师，您好，有个疑惑点：HTTP2.0 3个请求一定要创建3个流吗？每个流里面都要传输一次header帧？理解是要的，提到数据报文里面的请求行要确定方法和URL，3个请求的URL不同，方法类型也可能有差别，因此要分开3个header帧，但请问3个header帧可以放到同一个流里面吗？还是程序需要通过流来确定属于哪个请求的数据？

  2018-12-31

  **1

  **1

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7168.jpeg)

  陈天境

  刘超，你好，我们最近踩到一个Tomcat8.5的坑，就是HTTP响应本来应该是 HTTP/1.1 200 OK 但是Tomcat8.5 默认只会返回 HTTP/1.1 200 也是就是OK没掉了，好奇参考了HTTP协议的rfc文档，按照官方描述，是表示OK其实不是必须的意思吗？ https://tools.ietf.org/html/rfc7230#section-3.1.2 The first line of a response message is the status-line, consisting   of the protocol version, a space (SP), the status code, another   space, a possibly empty textual phrase describing the status code,   and ending with CRLF.

  2018-09-22

  **1

  **1

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7169.jpeg)

  Hulk

  你好，Accept-Charset是针对字符的编码？如果请求的是文件，那二进制的编码是？

  作者回复: 不一定都需要这个头，看客户端和服务器怎么解析，怎么用

  2018-06-29

  **

  **1

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615555-7170.jpeg)

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

- ![img](https://static001.geekbang.org/account/avatar/00/2b/00/fb/85b07045.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](https://static001.geekbang.org/account/avatar/00/1b/65/b7/058276dc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](https://static001.geekbang.org/account/avatar/00/14/df/e6/bd1b3c0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

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

- ![img](https://static001.geekbang.org/account/avatar/00/13/6a/01/d9cb531d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](https://static001.geekbang.org/account/avatar/00/13/62/1e/ad721e61.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b3/df/0fabb233.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,w_14.png)

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

- ![img](%E7%AC%AC15%E8%AE%B2%20_%20HTTPS%E5%8D%8F%E8%AE%AE%EF%BC%9A%E7%82%B9%E5%A4%96%E5%8D%96%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8E%9F%E6%9D%A5%E8%BF%99%E4%B9%88%E5%A4%8D%E6%9D%82.resource/resize,m_fill,h_34,w_34-1662307615554-7153.jpeg)

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
