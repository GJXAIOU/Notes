# 10 | 模块答疑：我们能从Hadoop学到什么？

专栏的模块一已经更新完毕，按照计划，今天是我们答疑的时间。首先要感谢订阅专栏的同学给我留言，每条留言我都看过了，有些留言对我的启发也很大，希望同学们可以多多跟我互动。我在每个模块都设置了一个答疑的主题，想跟你聊聊我在学习这个模块时的心得体会。另外，我也会贴出一些同学的疑问，跟你聊聊我的想法。

今天的主题是：我们能从 Hadoop 学到什么？

最近几年，我跟很多创业者交流，发现创业最艰难的地方，莫过于创业项目难以实现商业价值。很多时候技术实现了、产品做好了，然后千辛万苦做运营，各种补贴、各种宣传，但是用户就是不买账，活跃差、留存低。

很多时候，我们不是不够努力，可是如果方向错了，再多努力似乎也没有用。阿里有句话说的是“方向对了，路就不怕远”，雷军也说过“不要用你战术上的勤奋，掩盖你战略上的懒惰”。这两句话都是说，要找好方向、找准机会，不要为了努力而努力，要为了目标和价值而努力。而王兴则更加直言不讳：“很多人为了放弃思考，什么事情都干得出来”。

说了那么多，我们再回过来看看 Hadoop 的成长历程。从 2004 年 Google 发表论文，到 2008 年 Hadoop 成为 Apache 的开源项目，历时 4 年。当时世界上那么多搜索引擎公司似乎都对这件事熟视无睹，Yahoo、百度、搜狐（是的，搜狐曾经是一家搜索引擎公司），都任由这个机会流失。只有 Doug Cutting 把握住机会，做出了 Hadoop，开创了大数据行业，甚至引领了一个时代。

所以，**我们能从 Hadoop 中学到的第一个经验就是识别机会、把握机会**。有的时候，你不需要多么天才的思考力，也不需要超越众人去预见未来，你只需要当机会到来的时候，能够敏感地意识到机会，全力以赴付出你的才智和努力，就可以脱颖而出了。

结合大数据来说，**虽然大数据技术已经成熟，但是和各种应用场景的结合正方兴未艾，如果你能看到大数据和你所在领域结合的机会，也许你就找到了一次出人头地的机会。**

另一方面，你看一下 Hadoop 几个主要产品的架构设计，就会发现它们都有相似性，都是一主多从的架构方案。HDFS，一个 NameNode，多个 DataNode；MapReduce 1，一个 JobTracker，多个 TaskTracker；Yarn，一个 ResourceManager，多个 NodeManager。

事实上，很多大数据产品都是这样的架构方案：Storm，一个 Nimbus，多个 Supervisor；Spark，一个 Master，多个 Slave。

大数据因为要对数据和计算任务进行统一管理，所以和互联网在线应用不同，需要一个全局管理者。而在线应用因为每个用户请求都是独立的，而且为了高性能和便于集群伸缩，会尽量避免有全局管理者。

所以**我们从 Hadoop 中可以学到大数据领域的一个架构模式，也就是集中管理，分布存储与计算**。我在思考题里提出如何利用个人设备构建一个存储共享的应用，很多同学也都提到了类似的架构方案。

最后我想说，使用 Hadoop，要先了解 Hadoop、学习 Hadoop、掌握 Hadoop，要做工具的主人，而不是工具的奴隶，不能每天被工具的各种问题牵着走。最终的目标是要超越 Hadoop，打造适合自己业务场景的大数据解决方案。

正好提到了每期文章后留给你的思考题，在这里也分享一下**我是如何设置思考题的**。

关于思考题，你会发现，我留的思考题很多都是和当期内容没有直接关联的，甚至和大数据无关的。它们或是相似的问题场景，或是有类似的解决思路，或是引申的一些场景。

其实我是希望你在学习大数据的时候，不要仅局限在大数据技术这个领域，能够用更开阔的视野和角度去看待大数据、去理解大数据。这样一方面可以更好地学习大数据技术本身，另一方面也可以把以前的知识都融会贯通起来。

计算机知识更新迭代非常快速，如果你只是什么技术新就学什么，或者什么热门学什么，就会处于一种永远在学习，永远都学不完的境地。前一阵子有个闹得沸沸扬扬的事件，有个程序员到 GitHub 上给一个国外的开源软件提了个 Issue“不要再更新了，老子学不动了”，就是一个典型例子。

如果这些知识点对于你而言都是孤立的，新知识真的就是新的知识，你无法触类旁通，无法利用过往的知识体系去快速理解这些新知识，进而掌握这些新知识。你不但学得累，就算学完了，忘得也快。

所以不要纠结在仅仅学习一些新的技术和知识点上了，构建起你的知识和思维体系，不管任何新技术出现，都能够快速容纳到你的知识和思维体系里面。这样你非但不会惧怕新技术、新知识，反而会更加渴望，因为你需要这些新知识让你的知识和思维体系更加完善。

关于学习新知识我有一点心得体会想与你分享。我在学习新知识的时候会遵循一个**5-20-2 法则**，用 5 分钟的时间了解这个新知识的特点、应用场景、要解决的问题；用 20 分钟理解它的主要设计原理、核心思想和思路；再花 2 个小时看关键的设计细节，尝试使用或者做一个 demo。

如果 5 分钟不能搞懂它要解决的问题，我就会放弃；20 分钟没有理解它的设计思路，我也会放弃；2 个小时还上不了手，我也会放一放。你相信我，一种真正有价值的好技术，你这次放弃了，它过一阵子还会换一种方式继续出现在你面前。这个时候，你再尝试用 5-20-2 法则去学习它，也许就会能理解了。我学 Hadoop 实际上就是经历了好几次这样的过程，才终于入门。而有些技术，当时我放弃了，它们再也没有出现在我面前，后来它们被历史淘汰了，我也没有浪费自己的时间。

还有的时候，你学一样新技术却苦苦不能入门，可能仅仅就是因为你看的文章、书籍本身写的糟糕，或者作者写法跟你的思维方式不对路而已，并不代表这个技术有多难，更不代表你的能力有问题，如果换个方式、换个时间、换篇文章重新再看，可能就豁然开朗了。

接下来我们看一下同学们的具体问题。

![image-20230416190854444](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416190854444.png)

我的判断是是大数据与业务的结合，在每个垂直领域、每个垂直领域的细分领域，将大数据和业务场景结合起来，利用大数据发现新的业务增长点。

对技术人员而言，其实挑战更高了，一方面要掌握大数据的知识，这正是专栏想要输出的；另一方面，要掌握业务知识，甚至得成为业务领域的专家，能发现业务中可以和大数据结合的点，利用大数据和业务结合构建起技术驱动业务的增长点，这需要你在业务中有敏锐的观察力和领悟力。

![image-20230416190908831](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416190908831.png)

实时计算的结果一般通过两种方式输出，一个是写入到数据库里，和离线计算的结果组成全量数据供业务使用；一个是通过 Kafka 之类的实时队列给业务，比如你提到的监控展示。关于大数据怎么和业务结合，我会在专栏第四模块与你讨论，请继续关注。

![image-20230416190918024](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416190918024.png)

事实上并不会慢，影响文件读写速度的是磁盘的速度。同样的数据量、同样类型的磁盘，HDFS 可以将数据分布在更多的服务器和磁盘上，肯定比单机上的的几块磁盘速度更快。

HDFS 常用的使用方式是结合 MapReduce 或者 Spark 这样的大数据计算框架进行计算，这些计算框架会在集群中启动很多的分布式计算进程同时对 HDFS 上的数据进行读写操作，数据读写的速度是非常快的，甚至能支撑起 Impala 这样的准实时计算引擎。

![image-20230416190929590](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416190929590.png)

我在专栏第 8 期留了一个思考题，我看了大家的留言，发现很多同学可能没有意识到互联网处理一个请求的复杂性，这里我来谈谈我的理解。

这个思考题其实并不简单，考察的是一个典型的互联网应用，比如淘宝的架构是怎样的。简化描述下，这个过程是：

首先，一个请求从 Web 或者移动 App 上发起，请求的 URL 是用域名标识的，比如 taobao.com 这样，而 HTTP 网络通信需要得到 IP 地址才能建立连接，所以先要进行域名解析，访问域名解析服务器 DNS，得到域名的 IP 地址。

得到的这个 IP 地址其实也不是淘宝的服务器的 IP 地址，而是 CDN 服务器的 IP 地址，CDN 服务器提供距离用户最近的静态资源缓存服务，比如图片、JS、CSS 这些。如果 CDN 有请求需要的资源就直接返回，如果没有，再把请求转发给真正的淘宝数据中心服务器。

请求到达数据中心后，首先处理请求的是负载均衡服务器，它会把这个请求分发给下面的某台具体服务器处理。

这台服具体的服务器通常是反向代理服务器，这里同样缓存着大量的静态资源，淘宝也会把一些通常是动态资源的数据，比如我们购物时经常访问的商品详情页，把整个页面缓存在这里，如果请求的数据在反向代理服务器，就返回；如果没有，请求将发给下一级的负载均衡服务器。

这一级的负载均衡服务器负责应用服务器的负载均衡，将请求分发给下面某个具体应用服务器处理，淘宝是用 Java 开发的，也就是分发被某个 Java Web 容器处理。事实上，淘宝在 Java Web 容器之前，还前置了一台 Nginx 服务器，做一些前置处理。

应用服务器根据请求，调用后面的微服务进行逻辑处理。如果是一个写操作请求，比如下单请求，应用服务器和微服务之间通过消息队列进行异步操作，避免对后面的数据库造成太大的负载压力。

微服务如果在处理过程中需要读取数据，会去缓存服务器查找，如果没有找到，就去数据库查找，或者 NoSQL 数据库，甚至用搜索引擎查找，得到数据后，进行相关计算，将结果返回给应用服务器。

应用服务器将结果包装成前端需要的格式后继续返回，经过前面的访问通道，最后到达用户发起请求的地方，完成一次互联网请求的旅程。如果用架构图表示的话，就是下面的样子。

![image-20230416190951692](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416190951692.png)

这张图来自我写的《大型网站技术架构：核心原理与案例分析》一书，对互联网实时业务处理感兴趣的同学，欢迎阅读这本书。大数据的数据来源最主要的就是网站数据，了解网站架构对学习大数据、用好大数据也很有帮助。

最后，我在今天的文章里贴了陈晨、虎虎、您的好友 William、lyshrine、不求、Panmax、wmg、西贝木土的留言，我认为是比较精彩很有深度的，也把它们分享给你，希望其他同学的思考也能对你有所启发，也欢迎你给我留言与我一起讨论。


![image-20230416191015366](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191015366.png)

![image-20230416191033158](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191033158.png)

![image-20230416191052129](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191052129.png)

![image-20230416191105683](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191105683.png)

![image-20230416191132250](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191132250.png)

![image-20230416191143521](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191143521.png)

![image-20230416191157966](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191157966.png)

![image-20230416191224550](10_%E6%A8%A1%E5%9D%97%E7%AD%94%E7%96%91%EF%BC%9A%E6%88%91%E4%BB%AC%E8%83%BD%E4%BB%8EHadoop%E5%AD%A6%E5%88%B0%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20230416191224550.png)



## 精选留言(24)

- 

  带你去旅行 置顶

  2018-11-20

  **4

  蜗牛，您好，那是一段 spark 代码，你可以尝试把每一步的结果打印出来，就能明白每一个算子的用途了。

- 

  三木子

  2018-11-20

  **20

  老师好。感谢你提供的方法论，我想谈下我学大数据失败心得，hadoop 我也断断续续看过几次文档，写过 wordCount,但是就是没有进步，我总结了下原因是我在工作中没有应用到 hadoop，没有实际场景体会，体会不到它的价值。学 spark 也是这样，这里就有个问题想请问老师，向我这样在工作没用到大数据可以在什么地方找些实际案列学习提高呢，后期也想转型搞找大数据。谢谢！

  展开**

  作者回复: 坚持看完专栏，也许会有收获。

- 

  老男孩

  2018-11-20

  **8

  老师，我还有一个问题。有点不好意思问……大数据的学习是否还要学习掌握一些高深的算法？您也说了 hadoop 只是一个工具，具体的业务场景是不是还要用对应的算法模型去挖掘出有价值的数据？我的一个同事昨天略带藐视语气对我说，只有研究生学历的才能研究大数据。数学不行的，还是老老实实写你的业务代码。想到自己是学渣，尽无言以对，当场懵逼了。

  展开**

  作者回复: 并不是，后面专栏会讲算法。有可能你跟我一样，数学不好，是因为不明白数学有什么用，学了算法，明白了用途，说不定数学也好了。

- 

  风之翼

  2019-01-08

  **4

  我认为，学习一个新的东西，首先要弄清楚三件事：这是什么东西（干什么的）？为什么需要它（怎么来的）？它是如何运作的？
  随着互联网信息产业的发展，网络上时刻产生的数据以及沉淀的历史数据量规模呈爆炸式增长，而计算机硬件诸如 CPU、内存等性能的增长速度远远跟不上数据的增长速度，因此传统的单机处理程序已经无法满足数据的处理需求。分布式处理系统应运而生，这是大数据系统的前身。
  大数据系统主要处理大规模（一般指 PB 级别的数据量）的动态和存量数据，通过大量数据的读取和分析，能够从中找出人们从未关注到的甚至没有想到的 事物之间的关联关系，并以此为人们日常生活以及各种生产活动提供必需的决策支持。
  对于大数据系统的运作原理，我想从一个设计者的角度来思考：
  要清晰的知道，不论大数据系统功能多么强大，性能多么 NB，体系如何庞杂，它本质上依然和传统软件的运作模型是一样的：I-P-O，没错，就是输入-计算-输出。以 MapReduce 为例，输入就是各 taskStracker 在本地各自读取数据分片；而计算过程有 2 大的步骤：1 是 map 进程阶段将原始数据进行初步合并计算，并将得出的结果发给 reduce 进程，我把这个过程称为预处理，预处理后的数据量会降低到网络可以承受的地步；2 是 reduce 收到 map 传来的预处理数据，并进行最终合并计算；输出部分:reduce 进程将最终计算的结果保存到 HDFS（本地数据块），并由 HDFS 将所有 reduce 保存的数据块合并成一个 HDFS 文件。你看，这个过程是不是就是一个 IPO 过程？只不过每个具体过程的执行者以及数量发生了改变。
  OK，弄清楚了基本运作模型，接下来就是考虑：针对大规模数据分散存储（先不考虑实时数据哈）在大量服务器上的数据存储背景，如何设计一款软件，能够高效读取、处理这些数据，并有效输出呢？
  首先是数据读写。现在大家都清楚，在数据规模达到 PB 级别的情况下，如果使用集中读取，集中处理的方式，单机的硬件和网络根本承载不了。所以最好的办法就是让数据所在的服务器自行读取本地数据并进行计算，然后将每个服务器计算结果汇总后在写入本地，当然所有服务器写入本地的数据最终又会汇总成为一个可以被识别的输出文件。那么如何让每台服务都知道自己应该读哪个数据，输出时又该如何写入呢，写入之后又如何能够合并成一个可以识别的输出文件呢？分布式文件系统就是一个很好的解决方案。（这就是 HDFS 的由来）
  其次是数据计算。前面说过，我们要让数据所在的服务器自行读取文件在本地的数据块，并进行计算。首先是本地读取完数据块后，执行的初步计算并得出结果；然而问题来了，在所有本地服务计算完成后，他们的计算结果中一般都存在维度重叠的问题（即服务器 1 计算结果中有 A B C 三个维度统计数据，而服务器 2 中有 A C D E4 个维度的数据，此时不能直接将各服务器的结果写入输出文件），因此还必须将这些计算结果进一步合并，以保证每个维度的 key 是唯一的。因此计算过程应该有两部分任务组成：一是本地服务器计算统计后得出初步结果（也叫中间数据）；二是对这些中间数据进行最终合并计算。Hadoop 的大部分计算框架基本都是这两部走的，只是具体执行方式不太一样罢了（比如 spark 会把中间数据放在内存中而不是 HDFS 从而提高运行速度等）
  嘿嘿，弄清楚了设计原理，大家在回去看看李老师的课程，比如 HDFS，MapReduce 之类的，是不是感觉容易理解多了呢
  本人今年刚转大数据，李老师的课是我学习的第一门大数据课程，刚到第十章，今后每隔几章我都会写一篇心得和大家分享，也请老师多多指教哈

  展开**

  作者回复: 👍🏻

- 

  小辉辉

  2018-11-22

  **3

  非常赞同老师说的要构建自己的知识体系，当然还有思维体系。刚刚入行做程序员那时候，自己也想着平时要去积累一些知识，买过书和视频。但是每次都是碰到一个东西，觉得这东西我会，但是一但动手了，好像觉得自己啥都不会，看完的书和视频也是看完了就忘了。最后是努力的把自己感动的一塌糊涂，完了东西又没学进去，中间有段时间也是很迷茫的，想学点什么，又不想去学。从刚入行到现在差不多也有 6 年了，经过慢慢的积累，有了自己的学习方式和思考方式，也在慢慢的构建自己的知识和思维体系，来学老师的这个专栏也希望自己的知识和思维更进一步。

  展开**

- 

  程序员小灰

  2018-11-20

  **3

  计算机知识更新迭代非常快速，如果你只是什么技术新就学什么，或者什么热门学什么，就会处于一种永远在学习，永远都学不完的境地。

  这句话很真实，感觉现在技术迭代太快了，并且门槛也不高，IT 行业靠经验值、阅历的工作也越来越少。实际工作中，做一颗螺丝钉，工作之余不敢懈怠，要努力学习`造航母`的知识。

  展开**

- 

  we

  2019-04-18

  **2

  不要纠结在仅仅学习一些新的技术和知识点上了，构建起你的知识和思维体系，不管任何新技术出现，都能够快速容纳到你的知识和思维体系里面。这样你非但不会惧怕新技术、新知识，反而会更加渴望，因为你需要这些新知识让你的知识和思维体系更加完善。

- 

  风之翼

  2019-01-08

  **2

  补充一点忘了说了，就是调度的问题（YARN）。前面说过，要让数据所在的服务器自行读取本地数据并进行计算。那么这里有几个问题：1、由谁来确定任务需要的数据文件分布在哪些服务器上呢？2、如果数据块所在的服务器正忙于其他事物，无暇顾及新分配的任务该怎么办呢？3、任务程序如何进入数据块所在服务器并自动执行的呢？于是就得引入任务调度模块，该模块必须具备以下功能：1、能够实时查询数据文件分布在哪些服务器上；2、能够实时监控各个服务器的运行状态，以便分配任务；3、如果发现数据块所在的服务器比较繁忙，就得用完善的调度算法，把任务分配给其他空闲服务器，并主导目标数据块的传输；4、当目标服务器获得分配的任务后，能够自动加载并执行该任务程序；最后，当所有合并计算完成后，还能够将各服务器写入的数据块合并成一个可读文件。在 Hadoop1 时期，这些功能由 MapReduce 兼任，后来的版本中被单独剥离出来，并加强了一些功能，便形成了 yarn（比如 yarn 使用容器作为服务器资源的最小单位，每个容器分配一定的 cpu，内存资源，每个节点服务器根据其硬件配置，启动若干容器，这些容器的运行状态被资源管理器实时掌握）。

  展开**

- 

  hua168

  2018-11-20

  **2

  想不到大神是这么负责的人，不仅告诉我们技术，更重要是告诉我们思想！每条留言都认真看，佩服！
  看完了 hadoop 给我最大提示是:使用就是硬道理，机会来了你站在风口上，成功了！

  作者回复: 加油

- 

  落叶飞逝的...

  2018-11-20

  **2

  学习大数据不仅仅局限于新的技术，而且需要利用原有的知识体系进行融会贯通这种很重要！

- 

  yzwall

  2018-11-22

  **1

  这篇文章字字珠玑，技术思考比技术讲解更重要。

  展开**

- 

  纯洁的憎恶

  2018-11-20

  **1

  大数据架构中大量出现了一主多从的形式，这似乎与去中心化、自底向上的自组织体系还有较大距离。这会是大数据的潜力所在么？还是说万事万物都不能绝对，无论去中心化还是中心化，都要达到某种平衡，才能有效运转？

  展开**

  作者回复: 去中心，自组织的成本会更高；中心可靠高效的情况下，有中心效率更高。参考区块链，也可以参考现实世界。

- 

  hua168

  2018-11-20

  **1

  看到 hadoop 我就想起了 openstack，现在云计算公司那么多，还有必要学 openstack 吗？看到 HDFS 我就想起了对象存储，现在开源的对象存储有哪些呀？可以用在生产环境的…最后想到学习大数据是不是要学一门编程语言？我看很多大数据的教程都是用 Java，python 用于爬虫，机器学习，AI 方面比较多。有一小部分人说 java 会慢慢被淘汰，建议不要学，未来几十年应该不会被淘汰吧？

  展开**

  作者回复: 要学编程，你思考很多，很好，但是还是要自己动手，再能真正明白。

- 

  🐱您的好...

  2018-11-20

  **1

  目前我感触最深的话就是雷军那句：“不要用战术上的努力掩饰战略上的懒惰。”，其实人都是太“聪明”了，用各种各样的手段回避思考和思考带来的不确定性。

  展开**

- 

  老男孩

  2018-11-20

  **1

  很受启发。第一模块不仅学到大数据的知识，还让我重新理解了以前的一些错误观点。期待下文，快快发啊！这样一天看一点真不过瘾。

  作者回复: 谢谢

- 

  小苏饼

  2018-11-20

  **1

  hdfs 不是一个 active 一个 standny 的 namenode 吗保持 HA 的高可用，这算一个 namenode 还是两个 namenode 啊 😂为什么文中说一个 namenode，yarn 也是用两个呀😳

  展开**

  作者回复: 逻辑上是一个，当我们讨论系统流程逻辑的时候，就说是一个。当我们讨论 ha 的时候，就是两个。
  这也是一种抽象。

- 

  冉

  2019-04-19

  **

  这才是真正有价值的专栏，不光学到技术更能提高技术视野，作者很用心，希望能多出专栏👍

  作者回复: 谢谢鼓励~

- 

  saxon

  2019-03-10

  **

  读完这篇对我很有启发，构建自己的知识体系，并不断积累完善。

  展开**

- 

  Skye

  2019-02-27

  **

  买老师的专栏真的值了。以前学东西经常学了就忘，就是看待知识的角度不对，没有理解其核心。应该构建自己的知识体系，不被技术牵着走。

- 

  杰之 7

  2019-02-11

  **

  通过前第一章的复习，回顾了大数据 Hadoop 产品，它们的发展是这个时代必然的产物。今天我想不管是哪个细分的领域，只要我们有自己的一点观察力，也许都可以用大数据产品诞生的思维来做市场，并结合大数据技术本身来突出重围。

  在大数据产品 HDFS，MR，Yarn 中，都使用了一主多从的架构方式。这种方式也被其他的大数据产品所使用，成为了大数据架构的主要方式。

  对于通过学习大数据技术背后的原理，我们应该达到的目标是能投过表象看本质，新的技术会一直持续下去，但没有对技术本质的把握，永远也是被动的学习及低效的努力。

  过了技术这一个门槛，后面的路途会更宽敞明亮，思路也应该会更宽阔，这时我们做技术的主人，做时代的幸运儿。

  展开**