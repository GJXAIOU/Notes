# 特别放送（一）\| 经典的Kafka学习资料有哪些？

作者: 胡夕

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/35/72/35aac5760f00427d869d4f1798f3bf72.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/89/0e/8959b8e7cb714bba239f4a60ac1ba60e.mp3" type="audio/mpeg"></audio>

你好，我是胡夕。我们的课程已经更新一段时间了，你每节课都按时学了吗？如果你一直跟着学习的话，相信你一定会有很多收获的。

当然了，我也知道，在学习源码的过程中，除了有进步的快乐，可能还会有一些痛苦，毕竟，源码并不是那么容易就可以掌握的。

如果你在学习的过程中遇到任何问题，都可以给我留言，我会尽快回复你，帮助你解决问题。如果你发现自己被一些不好的情绪包围了，除了要努力坚持以外，我建议你学着从这种情绪中暂时跳脱出来，让自己转换到一些轻松的话题上。今天，我要讲的特别放送的内容就非常让人放松，因为我会给你分享一些 Kafka 的学习资料。

实际上，在更新的这段时间里，经常有同学问我：“老师，我想更多地了解下 Kafka，你能给我推荐一些相关的学习资料吗？”今天，我就借着这个特别放送的环节，专门为你搜罗了各种 Kafka 学习资料，包括书籍、视频、博客等一切影音像资料，我还把它们做成了清单，一起分享给你。

这些资料的深浅程度不一样，有的偏重于基础理论，有的深入底层架构，有的侧重于实际案例，有的是分享最佳实践。

如果你期望得到实际使用上的指导，那么可以重点关注下我提到的社区维护的文档以及各类 Kafka 实战书籍。如果你对 Kafka 源码的学习兴趣更加浓厚，那么，这节课里的各类大神级的博客以及视频资料是你需要重点关注的。因为他们经常会直接给出源码级的分析，学习这类资料既能开拓我们的思路与视野，也能加深我们对源码知识的理解，可以说是具有多重好处。

<!-- [[[read_end]]] -->

总之，我建议你基于自身的学习目标与兴趣，有针对性地利用这些资料。

我把这份清单大体分为英文资料和中文资料两大部分，我先给出收集到的所有英文资料清单。

## 英文资料

1\.[Apache Kafka官方网站](<https://kafka.apache.org/documentation/>)

我不知道你有没有认真地读过官网上面的文字，这里面的所有内容都是出自 Kafka Committer 之手，文字言简意赅，而且内容翔实丰富。我推荐你重点研读一下其中的**Configuration篇**、**Operations篇**以及**Security篇**，特别是 Configuration 中的参数部分。熟练掌握这些关键的参数配置，是进阶学习 Kafka 的必要条件。

2\.Confluent 公司自己维护的[官方文档](<http://docs.confluent.io/current/>)

Confluent 公司是 Kafka 初创团队创建的商业化公司，主要是为了提供基于 Kafka 的商业化解决方案。我们经常把他们提供的产品称为 Confluent Kafka。就我个人的感觉而言，这个公司的官网质量要比社区版（即 Apache Kafka 官网）上乘，特别是关于 Security 和 Kafka Streams 两部分的介绍，明显高出社区版一筹。因此，我推荐你重点学习 Confluent 官网上关于[Security配置](<https://docs.confluent.io/current/security/index.html>)和[Kafka Streams组件](<https://docs.confluent.io/current/streams/index.html>)的文档。

3\.Kafka 的[Jira列表](<https://issues.apache.org/jira/issues/?filter=-4&jql=project%20%3D%20KAFKA%20ORDER%20BY%20created%20DESC>)，也就是我们俗称的 Bug 列表

你可以在这个页面上搜索自己在实际环境中碰到的 Kafka 异常名称，然后结合自己的 Kafka 版本来看，这样的话，你通常能判断出该异常是不是由已知 Bug 造成的，从而避免浪费更多的时间去定位问题原因。另外，你还可以通过认领 Jira 的方式来为社区贡献代码。后面我会单独再用一节课的时间，给你具体介绍一下为社区贡献代码的完整流程。

4\.Kafka 的[KIP网站](<https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Improvement+Proposals>)

KIP 的完整名称是 Kafka Improvement Proposals，即 Kafka 的新功能提案。在这里你能够了解到 Kafka 社区关于新功能的所有提案与相关讨论。有意思的是，有些 KIP 的讨论要比 KIP 本身精彩得多。针对各种新功能，全球开发者在这里审思明辨，彼此讨论，有时协同互利，有时针锋相对，实在是酣畅淋漓。KIP 的另一大魅力则在于它非常民主——**任何人都能申请新功能提案，将自己的想法付诸实践**。

5\.StackOverflow 的[Kafka专区](<https://stackoverflow.com/questions/tagged/apache-kafka?sort=newest&pageSize=15>)

大名鼎鼎的 StackOverflow 网站我就不过多介绍了。这上面的 Kafka 问题五花八门，而且难度参差不齐，不过你通常都能找到你想要的答案。同时，如果你对 Kafka 非常了解，不妨尝试回答一些问题，不仅能赚取一些积分，还能和全球的使用者一起交流，实在是一举两得。

6\.[Confluent博客](<https://www.confluent.io/blog/>)

这里面的文章含金量特别高，你一定要仔细研读下。举个简单的例子，我已经被太多人问过这样的问题了：“一个 Kafka 集群到底能支撑多少分区？”其实我们都知道这种问题是没有标准答案的，你需要了解的是原理！碰巧，Confluent 博客上就有一篇这样的原理性解释[文章](<https://www.confluent.jp/blog/apache-kafka-supports-200k-partitions-per-cluster/>)，是 Kafka 创始人饶军写的，你不妨看一看。

7\.Kafka 社区非公开的各种资料，这包括社区维护的[Confluence文档](<https://cwiki.apache.org/confluence/display/KAFKA/Index>)和 Google Docs

你几乎能在 Confluence Space 中找到所有的 Kafka 设计文档，其中关于 Controller 和新版 Clients 设计的文章非常值得一读；而 Google Docs 主要是两篇：一篇是 Kafka 事务的[详细设计文档](<https://docs.google.com/document/d/11Jqy_GjUGtdXJK94XGsEIK7CP1SnQGdp2eF0wSw9ra8/edit>)，另一篇是[Controller重设计文档](<https://docs.google.com/document/d/1rLDmzDOGQQeSiMANP0rC2RYp_L7nUGHzFD9MQISgXYM/edit>)。这两篇是我目前所见过的最详细的 Kafka 设计文档。国内的很多 Kafka 书籍和博客多是援引这两篇文章，甚至是直接翻译的，足见它们的价值非凡。

8\.Kafka 社区的[Twitter首页](<https://twitter.com/apachekafka>)和 Confluent 的[Twitter首页](<https://twitter.com/confluentinc>)

你可能会说，Twitter 算哪门子学习资料啊？但实际上，很多时候，你就是能够在这上面搜罗到有价值的 Kafka 文章，特别是 Confluent 的 Twitter，它会定期推送关于 Kafka 最佳实践方面的内容。每次看到这类文章， 我都有一种意外淘到宝藏的感觉。我给你举个例子，Kafka Twitter 在 2019 年 10 月 12 日转载了一篇名为[*Exploit Apache Kafka’s Message Format to Save Storage and Bandwidth*](<https://medium.com/swlh/exploit-apache-kafkas-message-format-to-save-storage-and-bandwidth-7e0c533edf26>) 的文章，这里面的内容水准很高，读起来非常过瘾，我建议你好好读一读。

9\.YouTube 上的[Kafka视频](<https://www.youtube.com/results?search_query=apache+kafka&sp=EgIIAw%253D%253D>)

这些视频内容主要包括 Kafka 原理的讲解、业界牛人分享等。有的时候，你会发现，我们对文字类资料的掌握程度远不如看视频来得深入。如果你的英语基础还不错的话，我推荐你重点关注下 YouTube 上的这些分享视频。

好了，上面这九个资料就是我总结的 Kafka 英文学习资料。总体上看，这些资料都不要求太高的英文基础。即使是像 YouTube 上的英文视频，也是支持实时翻译的，你不用担心出现无法理解内容的情况。

接下来，我来给出中文资料清单。

## 中文资料

首先，我给出我认为比较好的五本 Apache Kafka 书籍。

1\.[《Kafka权威指南》](<https://book.douban.com/subject/27665114/>)

这本书是“Kafka Definitive Guide”的中译本。实际上，这本书的中英两个版本我都看过，应该说中文版翻译得很棒，你直接看中译本就行了。这本书虽然很薄，但它包含的内容几乎囊括了 Kafka 的方方面面，而且这本书由 Committer 执笔，质量上无可挑剔。

2\.[《Kafka技术内幕》](<https://book.douban.com/subject/27179953/>)

这本书出版后一跃成为市面上能见到的 Kafka 最好书籍。这本书当得起“技术内幕”这四个字，里面很多 Kafka 原理的讲解清晰而深入，我自己读起来觉得收获很大。

3\.[《深入理解Kafka：核心设计与实践原理》](<https://book.douban.com/subject/30437872/>)

我与这本书的作者相识，他同时精通 Kafka 和 RabbitMQ，可以说是消息中间件领域内的大家。这本书成书于 2019 年，是目前市面上最新的一本 Kafka 书籍。我推荐你买来读一下。

4\.[《Kafka Streams实战》](<https://book.douban.com/subject/33425155/>)

这本书是“Kafka Streams in action”的中译本，由 Kafka Committer 撰写而成。该书是我见到的**最深入讲解Kafka Streams的书籍**。如果你想学习基于 Kafka Streams 的实时流处理，那么这本书是不能不看的。

5\.[《Apache Kafka实战》](<https://book.douban.com/subject/30221096/>)

我这本书是 2018 年出版的，和之前那些面向 Kafka 设计原理的国内佳作不同的是，该书以讨论 Kafka 实际应用为主。我在里面分享了我这几年参与 Kafka 社区以及在使用 Kafka 的过程中积累的各种“江湖杂技”。如果你以使用为主，那么我推荐你看下这本书。

书籍的推荐告一段落，下面，我再介绍三个网站给你。

第一个是 OrcHome。据我所知，OrcHome 是国人自己维护的一个 Kafka 教程网站。这个网站最具特色的是它有个[Kafka问答区](<https://www.orchome.com/kafka/issues>)，你可以在这上面提问，会有人专门负责解答你提出的问题。

第二个是我本人的[博客](<https://www.cnblogs.com/huxi2b/>)。这个博客里大多是关于 Kafka 或者是其他大数据技术的原创文章。另外，我也会定期分享转载一些国内外的优秀博客。

第三个是知乎的[Kafka专区](<https://www.zhihu.com/topic/20012159/newest>)。和 StackOverflow 不同的是，这个专区上的问题多以理论探讨为主。通常大家对于这类问题的解答还是很踊跃的，我也经常在这里回复问题。如果你对 Kafka 的某些原理想要做深入的了解，不妨在知乎的这个专区上提出你的问题，我相信很快就会有人回复的。

## 总结

好了，上面的这些内容就是我总结的 Kafka 学习资料清单，希望它们对你是有帮助的。我把它们整理在了一张表格里，你可以重点看下。

![](<https://static001.geekbang.org/resource/image/4d/b9/4d773e45c4a3f86c5d9e86bb4a7ac7b9.jpg?wh=2083*3672>)

另外，我强烈建议你把社区官网和 Confluent 官网文档仔仔细细地读几遍，我保证你一定会有很大的收获，毕竟，相比于清单上的其他项，官网文档是最最权威的第一手资料。

## 课后讨论

最后，我也请你分享一下你自己的 Kafka 学习书单、网站、影音像资料以及好的学习方法。

欢迎你在留言区畅所欲言，跟我交流讨论，也欢迎你把今天的内容分享给你的朋友。

