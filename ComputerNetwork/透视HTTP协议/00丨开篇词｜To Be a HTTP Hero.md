# 开篇词｜To Be a HTTP Hero

你好，我是罗剑锋（Chrono），一名埋头于前线，辛勤「耕耘」了十余载的资深「码农」。

工作的这十多年来，我开发过智能 IC 卡，也倒腾过商用密码机；做过政务项目，也做过商务搜索；写过网游核心引擎，也写过 CDN 存储系统；在 Windows 上用 C/C++ 做客户端，在 AIX、Linux 上用 Java、PHP 写后台服务……现在则是专注于「魔改」Nginx，深度定制实现网络协议的分析与检测。

当极客时间的编辑联系我，要我写 HTTP 专栏的时候，我的第一反应是：「HTTP 协议好简单的，有这个必要吗？」

你可能也会有同样的想法：「HTTP 不就是请求 / 响应、GET/POST、Header/Body 吗？网络上的资料一抓一大把，有什么问题搜一下就是了。」

不瞒你说，我当时就是这么想的，在之前的工作中也是一直这么做的，而且一直「感觉良好」，觉得 HTTP 就是这个样子，没有什么特别的地方，没有什么值得讲的。

但在编辑的一再坚持下，我「勉为其难」接下了这个任务。然后做了一个小范围的「调查」，问一些周围的同事，各个领域的都有，比如产品、开发、运维、测试、前端、后端、手机端……想看看他们有什么意见。

出乎我的意料，他们无一例外都对这个「HTTP 专栏」有很强烈的需求，想好好「补补课」，系统地学习了解 HTTP，这其中甚至还包括有七、八年（甚至更多）工作经验的老手。，

这不禁让我陷入了思考，为什么如此「简单」的协议却还有这么多的人想要学呢？

我想，一个原因可能是 HTTP 协议「**太常见**」了。就像现实中的水和空气一样，如此重要却又如此普遍，普遍到我们几乎忽视了它的存在。真的很像那句俗语所说：「鱼总是最后看见水的」，但水对鱼的生存却又是至关重要。

我认真回忆了一下这些年的工作经历，这才发现 HTTP 只是表面上显得简单，而底层的运行机制、工作原理绝不简单，可以说是非常地复杂。只是我们平常总是「KPI 优先」，网上抓到一个解决方法用过就完事了，没有去深究里面的要点和细节。

下面的几个场景，都是我周围同事的实际感受，你是否也在工作中遇到过这样的困惑呢？你能把它们都解释清楚吗？

- 用 Nginx 搭建 Web 服务器，照着网上的文章配好了，但里面那么多的指令，什么 keepalive、rewrite、proxy_pass 都是怎么回事？为什么要这么配置？
- 用 Python 写爬虫，URI、URL「傻傻分不清」，有时里面还会加一些奇怪的字符，怎么处理才好？
- 都说 HTTP 缓存很有用，可以大幅度提升系统性能，可它是怎么做到的？又应该用在何时何地？
- HTTP 和 HTTPS 是什么关系？还经常听说有 SSL/TLS/SNI/OCSP/ALPN……这么多稀奇古怪的缩写，头都大了，实在是搞不懂。

其实这些问题也并不是什么新问题，把关键字粘贴进搜索栏，再点一下按钮，搜索引擎马上就能找出几十万个相关的页面。但看完第一页的前几个链接后，通常还是有种「懵懵懂懂」「似懂非懂」的感觉，觉得说的对，又不全对，和自己的思路总是不够「Match」。

不过大多数情况下你可能都没有时间细想，优先目标是把手头的工作「对付过去」。长此以来，你对 HTTP 的认识也可能仅限于这样的「知其然，而不知其所以然」，实际情况就是 HTTP 天天用，时时用，但想认真、系统地学习一下，梳理出自己的知识体系，经常会发现无从下手。

我把这种 HTTP 学习的现状归纳为三点：**正式资料「少」、网上资料「杂」、权威资料「难」**。

第一个，**正式资料「少」**。

上购书网站，搜个 Python、Java，搜个 MySQL、Node.js，能出一大堆。但搜 HTTP，实在是少得可怜，那么几本，一只手的手指头就可以数得过来，和语言类、数据库类、框架类图书真是形成了鲜明的对比。

现有的 HTTP 相关图书我都看过，怎么说呢，它们都有一个特点，「广撒网，捕小鱼」，都是知识点，可未免太「照本宣科」了，==理论有余实践不足==，看完了还是不知道怎么去用。

而且这些书的「岁数」都很大，依据的都是 20 年前的 RFC2616，很多内容都不合时宜，而新标准 7230 已经更新了很多关键的细节。

第二个，**网上资料「杂」**。

正式的图书少，而且过时，那就求助于网络社区吧。现在的博客、论坛、搜索引擎非常发达，网上有很多 HTTP 协议相关的文章，也都是网友的实践经验分享，「干货」很多，很能解决实际问题。

但网上文章的特点是细小、零碎，通常只「钉」在一个很小的知识点上，而且由于帖子长度的限制，无法深入展开论述，很多都是「浅尝辄止」，通常都止步在「How」层次，很少能说到「Why」，能说透的更是寥寥无几。

网文还有一个难以避免的「毛病」，就是「良莠不齐」。同一个主题可能会有好几种不同的说法，有的还会互相矛盾、以讹传讹。这种情况是最麻烦的，你必须花大力气去鉴别真假，不小心就会被「带到沟里」。

可想而知，这种「东一榔头西一棒子」的学习方式，用「碎片」拼凑出来的 HTTP 知识体系是非常不完善的，会有各种漏洞，遇到问题时基本派不上用场，还得再去找其他的「碎片」。

第三个，**权威资料「难」**。

图书少，网文杂，我们还有一个终极的学习资料，那就是 RFC 文档。

RFC 是互联网工程组（IETF）发布的官方文件，是对 HTTP 最权威的定义和解释。但它也是最难懂的，全英文看着费劲，理解起来更是难上加难，文档之间还会互相关联引用，「劝退率」极高。

这三个问题就像是「三座大山」，阻碍了像你这样的很多有心人去学习、了解 HTTP 协议。

那么，怎么才能更好地学习 HTTP 呢？

我为这个专栏定了一个基调：「要有广度，但更要有深度」。目标是成为含金量最高的 HTTP 学习资料，新手可以由浅入深、系统学习，老手可以温故知新、查缺补漏，让你花最少的时间，用最少的精力，掌握最多、最全面、最系统的知识。

由于 HTTP 应用得非常广泛，几乎涉及到所有的领域，所以我会在广度上从 HTTP 尽量向外扩展，不只讲协议本身，与它相关的 TCP/IP、DNS、SSL/TLS、Web Server 等都会讲到，而且会把它们打通串联在一起，形成知识链，让你知道它们之间是怎么联系、怎么运行的。

专栏文章的深度上我也是下足了功夫，全部基于最新的 RFC 标准文档，再结合我自己多年的实践体会，力求讲清讲透，能让你看了以后有豁然开朗的感觉。

比如分析 HTTPS，我会用 Wireshark 从建立 TCP 连接时就开始抓包，从二进制最底层来分析里面的 Record、Cipher Suite、Extension，讲 ECDHE、AES、SHA384，再画出详细的流程图，做到「一览无余」。

陆游有诗：「**纸上得来终觉浅，绝知此事要躬行**」。学习网络协议最重要的就是实践，在专栏里我还会教你用 Nginx 搭建一个「麻雀虽小，五脏俱全」的实验环境，让你与 HTTP 零距离接触。

它有一个最大的优点：自身就是一个完整的网络环境，即使不联网也能够在里面收发 HTTP 消息。

我还精心设计了配套的测试用例，最小化应用场景，排除干扰因素，你可以在里面任意测试 HTTP 的各种特性，再配合 Wireshark 抓包，就能够理论结合实践，更好地掌握 HTTP 的知识。

每一讲的末尾，我也会留几个思考题，你可以把它当作是求职时的面试官问题，尽量认真思考后再回答，这样能够把专栏的学习由「被动地听」，转变为「主动地学」，实现「学以致用」。

当然了，你和我的「兴趣点」不可能完全一样，我在讲课时也难免「顾此失彼」「挂一漏万」，希望你积极留言，我会视情况做些调整，或者用答疑的形式补充没讲到的内容。

今年是万维网和 HTTP 诞生 30 周年，也是 HTTP/1.1 诞生 20 周年，套用莎翁《哈姆雷特》里的名句，让我们在接下来的三个月里一起努力。

「To Be a HTTP Hero！」

## 精选留言(105)

- ![img](https://static001.geekbang.org/account/avatar/00/14/78/ac/e5e6e7f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  古夜

  冲你这个发型我订了

  作者回复: 服就一个字。

  2019-05-29

  **5

  **304

- ![img](https://static001.geekbang.org/account/avatar/00/11/fd/1c/8245a8d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  imxintian

  我也是看这个发型，直接就来了。

  作者回复: 羡慕你的秀发，笑。

  2019-05-30

  **

  **64

- ![img](00%E4%B8%A8%E5%BC%80%E7%AF%87%E8%AF%8D%EF%BD%9CTo%20Be%20a%20HTTP%20Hero.resource/132.jpeg)

  鱼向北游

  看发型我就知道老师是高手 买买买

  2019-06-04

  **1

  **55

- ![img](https://static001.geekbang.org/account/avatar/00/16/ec/cb/6ee732c9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

   Sapph

  HTTP权威指南买了好长时间，我就没看完过，太难搞了，希望能跟着Chrono老师学好HTTP💪 

  作者回复: 一起努力，这本书的内容比较陈旧，专栏里会紧跟最新的标准文档。

  2019-05-29

  **

  **26

- ![img](https://static001.geekbang.org/account/avatar/00/16/a5/98/a65ff31a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  djfhchdh

  罗老师胖了，头发更少了~~~

  作者回复: 目前正在跑步减肥，头发是没办法了。

  2019-06-01

  **

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/13/71/ca/87688ff9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iGeneral

  发型来分析是整个极客第二厉害的最强王者，订阅。（http上课打瞌睡，然后自己就看了一本http图解，膜拜大佬，抓住痛点）

  作者回复: 不用着急，很快就成琦玉了，笑。

  2019-05-30

  **

  **15

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  极客酱

  老师是从硬搞到软的啊，软硬结合的人最有技术含量，就冲这一点，我买了。

  作者回复: 做的比较杂，见笑了。

  2019-05-30

  **2

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/0f/94/db/4e658ce8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  继业(Adrian)

  光看介绍，就让人不禁的搓搓手

  2019-05-29

  **

  **13

- ![img](https://static001.geekbang.org/account/avatar/00/16/75/89/74ee014c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_Corey

  想好好学习一下 曾经被面试官打击的要死 现在准备学好再打击回去

  作者回复: 进击的程序员！

  2019-05-30

  **

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/4f/6fb51ff1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  一步![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  RFC文档的根本看不下去啊，都是文字，还那么长，老师可以讲讲有关RFC文档的东西，怎么阅读，怎么去查找某个标准对应的哪一个RFC文档，还有RFC的文档名怎么定义的

  作者回复: 破冰篇最后一讲会有GitHub项目，列出http相关的rfc，方便你查阅。

  2019-05-29

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/17/52/eb/eec719f3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  开水

  冲着这发型，必须得喊一句，老师！

  2019-06-04

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/12/8b/7d/bfa7b4ad.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黄

  是时候还罗老师送我的书钱了！

  作者回复: 好熟悉的头像，哈哈。

  2019-05-29

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/16/e8/c9/59bcd490.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  听水的湖

  之前看趣谈网络协议的时候，里面关于HTTP就两三篇，而这个专栏看来是把HTTP讲细讲透了，上线就更了一篇正文，读起来挺有意思的，不枯燥，种草了。

  作者回复: 期待与你共同学习进步。

  2019-05-29

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/16/55/86/ca7c94ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  羁绊12221

  深知读者痛点，赞一个！很期待后续的课程

  作者回复: thanks。

  2019-05-31

  **

  **6

- ![img](00%E4%B8%A8%E5%BC%80%E7%AF%87%E8%AF%8D%EF%BD%9CTo%20Be%20a%20HTTP%20Hero.resource/132-166222341290412.jpeg)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  名

  一、专栏开篇 需求：HTTP协议太常见，各职业角色都有系统想真正搞懂的需要 现状：权威、正式资料少且晦涩难懂且个别信息过时；网上资料太碎没有体系的东西 学习：理论与实践结合实操起来并形象的类比理解学习

  作者回复: 欢迎新同学。

  2019-06-12

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/10/75/93/8135f895.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bywuu

  我就是在极客时间上评论“看发型我就知道，这个是高手！”的人。。。不订对不起自己的判断

  作者回复: 感谢支持。

  2019-06-01

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/11/3c/8a/900ca88a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  test

  老师我正在学rtsp这个视频流应用层协议，和http类似，用ffmpeg接受，有什么方法可以拦截两个软件之间的流量呢，可以把接收方软件的地址搞成类似实验环境里的127.0.0.1吗？ 有时vs开发一些软件，也会发http给远程设备，这种情况怎么简化为127.0.0.1这种实验环境呢？

  作者回复:  1.用wireshark可以捕获任何网络中的数据包。 2.可以在本机上搭一个类似的环境，然后客户端配置成127.0.0.1，但需要结合你的实际情况。

  2020-01-06

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/28/71/0a6f52c4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不会凉的黄花菜

  已经学完了该专栏，很喜欢老师的讲课风格，有趣易懂不枯燥。老师把知识用最通俗最容易理解的方式讲了出来，引人入胜，受益匪浅。期待老师，出下一个新专栏。

  作者回复: 承蒙夸奖，不胜感激。

  2019-09-06

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/30/8a/b5ca7286.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  业余草

  楼主和1楼都优秀，老司机开车，稳！

  2019-05-30

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/13/16/c8/980776fc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  走马

  hERO!

  作者回复: Wanna be a 最强 hero——

  2019-05-30

  **

  **4

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKObibUvAjbt2hG3Sb9uFAnLurM6aQDvppQOia7f7QCPk50W8KCc24PaXEm9YVxEOND1PDpp24NUloA/132)

  Kauh

  掉头发是对技术最大的尊重

  作者回复: 这个我不能同意，最好是不掉头发也能尊重技术，笑。

  2020-03-22

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/17/df/84/f7b759d9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  没有梦想的咸🐠

  迈向网络安全学习的第一步

  作者回复: 安全篇会有大量干货，敬请期待。

  2019-06-05

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/59/37/c6eec6a3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  止于至善

  罗老师的课程中会介绍一些由浅入深/循序渐进的好书籍吗? 您说好多经典书的内容都有些过时了, 很想您能给小白的我推荐一系列关于HTTP(包括网络当然最好了)方面的好的书籍吗? 这样再配合罗老师的专栏把HTTP和网络系统的学习一下, 打好基础~

  作者回复: HTTP以及协议相关的书太少了，没有什么特别推荐的，如果要求不高的话电商上网站上随便挑一本就是了。

  2019-05-30

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/1d/3a/cdf9c55f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  未来已来

  老师看着很像包贝尔啊，支持一个

  作者回复: 我觉得像琦玉多一点。

  2019-05-30

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/13/e4/a1/178387da.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  25ma

  http，tcp/ip 协议的书买了4本，没怎么看，希望通过老师和同学一起坚持下去，提升自己的知识面，非常需要这个专栏!

  作者回复: 边看书边学专栏，效果肯定会更好，课上课下一起努力吧，不过要注意书里的知识点很多都已经过时了，不要被误导。

  2019-05-30

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/d3/ce/8ed72840.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多多洛

  从跟帖回答所有的思考题开始

  作者回复: 学习态度值得点赞！

  2019-05-29

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/17/5b/4a/7308decb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  东边日出西边雨🌧

  这是一门必学课程

  作者回复: thanks。

  2019-07-05

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/17/ec/e6/b671944c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_Tester

  棒棒哒~第一次买极客时间里的课程 想了解这些点❤丰富自己的知识面 测试可能会用到~

  作者回复: 无论是开发、测试还是运维，http都是必须要掌握的技能，多努力吧。

  2019-06-08

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/37/77/10997526.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  水果哥

  曾经在面试中遇到让用TCP实现http的题目，瞬间跪倒在地。终于能系统的学下HTTP啦！谢谢老师!

  作者回复: 不用客气，一起努力吧。

  2019-06-05

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/c7/47/30d4b61e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不是云不飘

  还真是天天用，但也就仅限于基础的使用，称此时机系统了解下。

  作者回复: go on。

  2019-06-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/d6/53/21da9e2b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  磊爷

  发展动力:信息多样化，传输速度需求，信息安全需要。 我理解的HTTP: 1.保持信息的完整性，在传输过程中不丢失数据。 2.传输速度快，在各层间传输时间少，传输节点不发生错误。 3.稳定，不因为中间节点宕机而使原始数据被更改。

  作者回复: 理解的不完全对。 1、2其实是下层tcp的功能，http只是借用了它的能力。 3也是一样，tcp/ip的路由能力可以不丢包，自动路由选择路线。 至于保证数据完整、不更改，应该用到https。

  2019-05-31

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/c8/8627f5c1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  右耳朵猫咪

  我的兴趣是当hero

  作者回复: 的确，我们都是趣味使然的hero。

  2019-05-31

  **

  **2

- ![img](https://wx.qlogo.cn/mmopen/vi_32/DYAIOgq83eo0ef1iaEbdegjIPUibORqB0EcSjdfd05HianjRcCfwWMRaibtFP8rH0icxqzl3AqaiaJnSzOd79FmbPfbw/132)

  kennyeric

  tcp协议是面向传输的协议 其设计初衷是解决传输的安全性和可靠性 而在应用层有许多公共需求 如获取 修改 删除各种类型资源 如果没有http协议 各种浏览器都需要自己去实现tcp之上的私有协议 更大的问题是网站的服务提供商要去兼容各个终端的协议 听起来就是无法完成的任务 势必影响互联网在全球的普及 所以我个人认为 http的出现出用户因素以外 也是服务提供商不得不去做的事情

  作者回复: 双向促进，螺旋上升，原因有很多，大家都说的很好。

  2019-05-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/7e/bb/019c18fc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  徐云天

  http权威指南说get请求没有请求体，当时我就感觉奇怪，为啥postman可以发请求体呢？

  作者回复: 任何报文都可以有请求体，get当然也可以带，老书有错误也可以理解，关键是不能把错误一直带着不纠正。

  2019-05-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/fa/2c/9a0c45e6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  微凉

  1.需求倒逼技术发展。 2.http就是互联网上数据传输交互的一种标准。

  作者回复: 说的没错，但还要进一步深化，是什么样的标准，都规定了什么。

  2019-05-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/d0/46/1a9229b3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  NEVER SETTLE

  一直在学习老师的C++和Nginx书籍，本身在公司里用到的技术栈就是 Nginx C++ ，一直对这个方面挺感兴趣的。一直有个想法，想把Nginx用C++封装下，搞一个自己的框架玩。对老师说的“魔改”Nginx很有兴趣，期待老师出一个现代C++的专栏，我之前已经反映很多次，极客时间没有C++相关的专栏，我觉得老师是最合适的人选。

  作者回复: 在GitHub上有c++ nginx开发，https://github.com/chronolaw/ngx_cpp_dev。c++等以后有机会吧。

  2019-05-29

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/9d/af/6fb341fa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  SamGo

  之前买了《HTTP权威指南》和《TCP/IP详解 卷1：协议》，确实很难啃，希望这个专栏有不同的体验

  2019-05-29

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_3e1530

  老师，ssl层是在tcp层之上的，为什么会先进行tcp握手，后进行ssl握手，是因为ssl的握手是基于tcp进行的么？还是因为ssl握手比tcp握手复杂，会先tcp握手来进行网络环境的确认？既然先进行tcp握手，为啥ssl层要放到tcp的上层？

  作者回复: 这个从协议栈来看是很自然的事情了。 ssl在tcp之上，所有数据必须由tcp来承载发送，所以必须先建立tcp连接，否则后面的ssl协议就无从谈起了。 这个与协议的复杂度无关，因为tcp是传输层，而ssl是会话/表示层，所以必须要依赖于tcp。

  2021-07-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  和老师刚开始的感觉差不多，HTTP协议好像就是挺简单的呀！ 先说一下自己的认识，学完对照一下从老师哪里获取了多少新的东西。 1：所有浏览器在使用，主要用于传输超文本，超文本是啥玩意？比文本牛逼一点的东西，比如：音频、视频、动画、图片、超链接等，就是构成网页的那些玩意。 2：她比较简单，发出一个请求，然后得到一个响应，响应的数据支持许多的格式。协议大概分成两部分，协议头和协议体，协议头负责说明所发送消息的格式信息以及使用协议的各种特性，协议体用于传输真正的信息。响应回来的信息包括一个状态码，标识响应的状态，有一堆状态码，不同的状态表示不同的意思，告诉请求方是失败还是成功还是其他。 3：她在网络协议的应用层，如果不管她的依赖层都是则处理信息的，她本身就是这么简单。现在有许多的RPC服务也使用这个协议来实现，由于靠上，所以，性能稍差，除此之外她有许多优点，最典型的就是简单易用也易学。 我只所以学习这个专栏，就是我目前的小组许多RPC服务都是使用她来完成的，希望对她有更多的了解，想知道她的性能到底如何？如果慢慢在哪里？怎么优化？传输数据安全性到底如何？如果不安全，为什么？怎么传输才安全？另外，这个协议怎么能很方便的限流降级嘛？再者就是希望出现各种异常，比如：IO异常、超时异常、链接不够异常什么的都能清楚的知道为啥？以及该如何快速的定位到问题和解决问题？别的不说，如果性能优化方面能学到一些能用于实际项目的优化技巧就值啦！ 看老师发型，就知道水平了得，期待后面的分享，也期待自己预期的问题都能在此专栏里学到解决思路。

  作者回复: 码字辛苦了，多分享经验不仅能够提高自己，也能够帮助别人。 http虽然简单，但它却能够衍生出非常复杂的技术和应用，就像元素周期表，只有100多种元素，但却组合出了世界上所有的东西

  2020-03-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/4e/6c/71020c59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王麒

  访问www.chrono.com都自动跳转到https://park.zunmi.cn/ 。这是被劫持了吗？求解救。

  作者回复: 检查一下hosts文件，清理一下浏览器的缓存，让浏览器重新解析域名。

  2020-03-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/5f/fb/af061ca7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  qpzm7903

  战斗力指标：发量

  2019-09-21

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/10/20/103c6d99.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  盛夏

  作为一个前端，天天跟http打交道，但对http却没有深入了解过，专门搜索了一下没想到真的有http专栏。希望能跟着老师学习能够加深对http的理解，知道http中相关的知识，做到知其然且知其所以然～

  作者回复: 坚持就是胜利，努力就有收获。

  2019-07-27

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/35/25/bab760a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  好好学习

  这是一个发型驱动的专栏

  2019-05-31

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b5/61/9802a552.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ican_只会0到9

  重新系统学习http，希望是一段充实的时光

  作者回复: 敬请期待，笑。

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/17/c9/43/b1d31b73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  feifei

  好好学习

  2019-05-29

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKnoSoric6IJjI9icQdhaL3IKRwbeic4IoLYAFricOzm0LnGbALtY6VQCYZ1AOiaux2foHok3OpRY94oxw/132)

  红红股海

  来补课

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/7b/57/a9b04544.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  QQ怪

  来打基础了，来补课

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/15/5c/2d/226a3631.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  MZC

  只是看过一些描述  大概知道一丢丢概念性的东西  希望跟老师收获多多

  作者回复: 一起努力，贵在坚持。

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/b7/cb/18f12eae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不靠谱～

  一直想深入学习http，但是一直停留在get post put delete上。希望跟着老师努力学习。

  作者回复: 叫老师夸张了，“学业有先后，术业有专攻”而已。

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/15/54/f3/2216f04f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我叫不知道

  搬起我的小板凳～

  作者回复: 要瓜子吗，笑。

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/22/00/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  lanco

  一起学习，透过问题看本质

  2019-05-29

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/8d/0e/5e97bbef.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  半橙汁

  一刷~20220701~

  作者回复: welcome

  2022-07-01

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  李文杰

  coming.

  作者回复: welcome

  2022-03-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/ac/6e/c13d131c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  她微笑的脸y

  开始深入学习http！知其然知其所以然！

  作者回复: welcome！

  2022-03-30

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/EwOocztTDRQHo2bXlCqvRWTlOiaCZAibMJF0BXfcFXwpALDWHJjLamDn9KXl5yt7icS7fiaibMTBeECUlyu7jibIkwEA/132)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  BlackTango

  20220226

  2022-02-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/6a/14/aeda3c7e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  浮生

  非科班的不懂网络这块，这节课能有帮助吗

  作者回复: 建议先试看几节课，如果能有收获再考虑。

  2022-01-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/32/0e/5024c2dd.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  星空

  希望后面可以看到用wireshark来分析nginx反向代理的抓包内容

  作者回复: Nginx反向代理用的也是http协议，和client-server通信没有本质的不同，主要就是多了一些头字段。

  2022-01-09

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83epuH2q0TG9wSxpmQJrNvUPCseiaJTZ2AQvBOxptXBG46qTSw40H90oqhNgPXicmXrhRjjE5CzsDYCMw/132)

  Geek_779a81

  老师，我最近在做一个文件分片上传的需求，每次执行任务时，总有几个分片上传的特别慢(其他的速度正常)，但也能正常传过去，这种我应该怎么排查问题？

  作者回复: 我觉得可能先要看客户端与服务器是怎么通信的，分片采用的是哪种方式，才好定解决的方案，现在还不太清楚。 像这样的问题，可能还是先从抓包上来看比较好。

  2021-12-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/36/07/c9ba247b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  桃之翼

  老mac装不上OpenResty就很苦恼

  作者回复: 用虚拟机或者docker吧，实机的确有很多麻烦，我现在都用虚拟机。

  2021-11-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/7f/d1/15a39351.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小门神

  哈哈，有时候看起来简单，其实深入本质就挺难的，http就是这种感觉呀。跟着老师好好学习http，深入本质，研究底层机制，每天加油干，好好学习！

  作者回复: keep going.

  2021-11-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2a/3e/71/98af2a83.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  HILLIEX

  来的有点晚

  作者回复: welcome

  2021-11-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/40/d2/36071a79.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  冯四

  网上查了好多关于HTTP的相关资料，有些简直就是误导人

  作者回复: 也不能完全这么说，认知有差异也是可以谅解的。

  2021-03-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/6d/89/14031273.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Direction

  http真的是看似简单，用起来细节巨多，要注意的地方也很多，优化更是大有学问，正好来查缺补漏，支持老师一个。

  作者回复: 感谢支持！

  2021-03-02

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIgmWmb9ia7ekgfoCWicvQicE4S9yPuUdd1MB8oVCBex8jGntqZK4yiaMgEAOk11Y6ictbC28uV2dUib2FA/132)

  孤狼

  看发型就知道是个高手，默默的订一波

  作者回复: many thanks 

  2021-03-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/16/ec/dd915863.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  不差钱的锅仔

  学起来

  作者回复: welcome!

  2021-02-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/59/8d/0e7e4def.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  沐(╯3╰)小璃

  前端小白来报道，（看到发型订阅的，看来不止我一个）

  作者回复: welcome!

  2021-02-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7c/c8/8627f5c1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  右耳朵猫咪

  网络模型七层协议看书看得头疼，很难理解。

  作者回复: 很多书上讲的比较理论，离实际工作较远，所以学起来费劲，建议还是结合实际工作来学，也不要求完全掌握，一次get一两个点，时间长了就能够烂熟于心了。

  2020-12-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6e/9e/c38e1a94.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  罗基

  搭配   系统性能调优必知必会 的网络部分， 会理解的更深。

  作者回复: 知识就是要多交叉学习，形成网状结构，才会融会贯通。

  2020-10-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/04/a6/18c4f73c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Airsaid

  老师您好，我看 RFC 文档上最新的还有 HTTP2.0，看日期是 2015 年，对于这块您为什么没有提到呢？

  作者回复: 后面都会讲，HTTP/2还有HTTP/3，刚开始的第一讲不要着急，笑。

  2020-10-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ac/96/46b13896.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  williamcai

  老师一看就是高人，直接订了

  作者回复: 欢迎加入学习http的大家庭。

  2020-09-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/a4/27/15e75982.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小袁

  一直在写业务逻辑层，想学一下http，刚好看到琦玉老师同款，我也想变强。

  作者回复: 现在http的应用范围很广，无论做什么，学它都是很有必要的。

  2020-09-13

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/LalJD9ruYQI5zVM1GOCe4PjunIbbeeMiacFHC4TAj0DBVeialKt3vRCLs9dxn1vYXvfp8pgcyaeEQkh1nde1JoBQ/132)

  jun

  原来发型这么重要，苦于一头乌黑不得要领

  作者回复: 哎，最好是即掌握了知识还能保持发量，我这是无奈，笑。

  2020-08-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/ce/b2/1f914527.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  海盗船长

  看完一遍了，打个卡，二刷走起。

  作者回复: very nice。

  2020-05-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/49/94/d5e5c959.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  忘了叫什么

  声音好喜欢，冲声音买了

  作者回复: 支持大感谢！

  2020-04-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/df/22/547f4aed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_ce3731

  最近正在学习网络通信的基础知识，看到直接订阅了

  作者回复: welcome，my friend。

  2020-04-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/f3/d0/aa7bac6f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  喵小喵

  看了简介，已经迫不及待想要开始了

  作者回复: 欢迎留言打卡。

  2020-03-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/cf/2c/8aeb9b64.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  波塞冬

  跟随大佬爬三座大山。

  作者回复: keep studying。

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/cf/2c/8aeb9b64.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  波塞冬

  第一句我居然听成我是龙卷风，😓

  作者回复: 汗……我原来以为普通话还挺标准的……

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e6/3f/b9ca7b14.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Geek_f544a9

  看到发型，我就梭哈，买！

  作者回复: 不要学我用脑过度，要适当休息，多出去运动。

  2020-03-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/60/05/ff1d4225.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kfzx-yuanjh

  作者文笔看着舒服，我也订了

  作者回复: Many thanks.

  2020-03-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/ad/c6/9397d3e4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Q

  看到作者这个发型，再看看以前买的作者写的nginx和boost库扉页的头发

  作者回复: 汗，岁月不饶人啊。

  2020-02-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/a1/69/0ddda908.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  满怀

  打卡~

  作者回复: 努力，加油。

  2020-01-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/8d/0e/5e97bbef.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  半橙汁

  好多熟悉却不知道全程和作用的名词= =、 mark一下，大学里没学好的，终究要在工作中补回来~ 出来混总是要还的~o(╥﹏╥)o

  作者回复: 学习从来不晚，实践中学效果更好。

  2020-01-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/d3/3e/0f3514c7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  秦时明月

  我和他们不一样，我不是看发型来的，哈哈

  作者回复: 难得的认真学习态度，笑。

  2019-11-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/21/3f/7e3e9dc4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  song218888

  好多都说到点子上了，不服不行啊，喜欢这个课程

  作者回复: 欢迎新同学。

  2019-10-16

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/19/29/4a8214b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Bug Killer

  http一直都是懵懵懂懂的状态，正好看到了这个课程，好好的跟着学一下

  作者回复: 坚持到最后，肯定会有收获。

  2019-09-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/46/4d/161f3779.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  ls

  打卡

  2019-09-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/cd/77/b2ab5d44.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  👻 小二

  讲到痛点了， 勾起了学习欲

  作者回复: keep going。

  2019-08-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/58/83/cd70f710.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  stone

  HTTP一直是短板，努力努力再努力～

  作者回复: keep going。

  2019-07-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f3/a0/a693e561.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  魔曦

  向高P低头哈😃

  2019-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/96/3c/05da15f5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  挡不住我

  罗老师，试听了您的课程，感觉讲的挺好的，作为一个运维，现在公司面临的现状是，公司的nginx配置杂乱无章，很多配置都是百度上找到，很多参数都不知道什么意思，希望我从头到尾理一下nginx，自己的水平也是会基本的配置，各种原理，http协议，wirashark抓包，都是模棱两可，现在要详细掌握这一部分，显然一个http协议是不够的，还有很多内容要学习了解，感觉越学越乱，是否可以给一个思路呢，现在是一脸懵逼的状态，好多东西呀，感觉自己掉到坑里面了😅😅

  作者回复: 极客上有一个nginx的教程，我觉得挺好的，你可以看看是否合适。

  2019-06-25

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/24/c6100ac6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  C家族铁粉

  Ok，那我就先学第三版了

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/24/c6100ac6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  C家族铁粉

  下午好，问一下大佬的《C++11/14高级编程：Boost程序库探秘（第3版）》一书第四版大概啥时候上线呢？

  作者回复: 暂时还没有想好第四版该修订哪些内容，目前c++11还没有太普及，而c++20又快出了，想看哪些内容可以提出来。

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/02/21/57f45071.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  余额不足

  自身对网络工程，网络编程，及软件编程很感兴趣的小白，励志要做一个全栈工程师的我，来听课了😁

  2019-06-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/6c/d8/68fec932.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  花花young

  开启学习http 模式,小白打卡

  2019-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e5/df/0c8e3fdc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小胖狗

  冲着这发型背后的深厚积累。😂😂😂

  作者回复: 大家不要再关心这个问题了……笑。

  2019-06-06

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJ4DXgQJF1uekMz5mrswQNEPwKICFjpKJ3vZ4JTZMV10ebIwicYt7T1pib7uKYErcQDIYfbflngDkIw/132)

  空中的鱼

  资深老马头真的是经不住时间的打磨

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/01/ac/2b552e26.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Chenhm

  哈哈哈，给口岸的老前辈点个赞！ 

  作者回复: 他乡遇故知，哈哈。

  2019-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5f/83/bb728e53.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Douglas

  再出发， 加油

  2019-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9d/9a/4cf0e500.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  芒果

  签到

  2019-05-31

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  远东通信-应用软件

  好东西，一定要好好学一学

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/c1/bf/494d0895.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  、时光偷走当初คิดถึง

  订了订了，坐等更新

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/81/6b/9a118acf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  西子无盐

  视频，还是音频

  2019-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c7/5f/2028aae5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  搏未来

  瓜子带板凳

  2019-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/4f/a7/f7124e0f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李坤

  GEEKXZ4C8

  2019-05-29

  **

  **
