# 33 \| 传统的可扩展架构模式：分层架构和SOA

作者: 李运华

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/34/ae/34af6d7174d170074fa6dff032e97aae.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/dc/25/dc83615607b96cabb69a6d0689b3ec25.mp3" type="audio/mpeg"></audio>

相比于高性能、高可用架构模式在最近几十年的迅猛发展来说，可扩展架构模式的发展可以说是步履蹒跚，最近几年火热的微服务模式算是可扩展模式发展历史中为数不多的亮点，但这也导致了现在谈可扩展的时候必谈微服务，甚至微服务架构都成了架构设计的银弹，高性能也用微服务、高可用也用微服务，很多时候这样的架构设计看起来高大上，实际上是大炮打蚊子，违背了架构设计的“合适原则”和“简单原则”。

为了帮助你在实践中更好的进行可扩展架构设计，我将分别介绍几种可扩展架构模式，指出每种架构模式的关键点和优缺点。今天我来介绍传统的可扩展模式，包括<span class="orange">分层架构和SOA</span>

，后面还会介绍微服务架构。

## 分层架构

分层架构是很常见的架构模式，它也叫N层架构，通常情况下，N至少是2层。例如，C/S架构、B/S架构。常见的是3层架构（例如，MVC、MVP架构）、4层架构，5层架构的比较少见，一般是比较复杂的系统才会达到或者超过5层，比如操作系统内核架构。

按照分层架构进行设计时，根据不同的划分维度和对象，可以得到多种不同的分层架构。

1. C/S架构、B/S架构

<!-- -->

划分的对象是整个业务系统，划分的维度是用户交互，即将和用户交互的部分独立为一层，支撑用户交互的后台作为另外一层。例如，下面是C/S架构结构图。

<!-- [[[read_end]]] -->

![](<https://static001.geekbang.org/resource/image/ac/55/acae5ee24fd6e6c41edb193a88e32f55.jpg?wh=3062*1768> "图片来自网络")

2. MVC架构、MVP架构

<!-- -->

划分的对象是单个业务子系统，划分的维度是职责，将不同的职责划分到独立层，但各层的依赖关系比较灵活。例如，MVC架构中各层之间是两两交互的：

![](<https://static001.geekbang.org/resource/image/36/50/3602b5bb371ebyy597c2d6b68f83d150.jpg?wh=3038*1570>)

3. 逻辑分层架构

<!-- -->

划分的对象可以是单个业务子系统，也可以是整个业务系统，划分的维度也是职责。虽然都是基于职责划分，但逻辑分层架构和MVC架构、MVP架构的不同点在于，逻辑分层架构中的层是自顶向下依赖的。典型的有操作系统内核架构、TCP/IP架构。例如，下面是Android操作系统架构图。

![](<https://static001.geekbang.org/resource/image/15/3a/15712073102390bede89e54ce6f2d13a.jpg?wh=3800*3700>)

典型的J2EE系统架构也是逻辑分层架构，架构图如下：

![](<https://static001.geekbang.org/resource/image/32/cb/3296df5420c88834493a6239d8dc62cb.jpg?wh=3148*2543>)

针对整个业务系统进行逻辑分层的架构图如下：

![](<https://static001.geekbang.org/resource/image/df/e4/dfd57e17b23823926d427db57620a0e4.jpg?wh=2497*1938>)

无论采取何种分层维度，分层架构设计最核心的一点就是**需要保证各层之间的差异足够清晰，边界足够明显，让人看到架构图后就能看懂整个架构**，这也是分层不能分太多层的原因。否则如果两个层的差异不明显，就会出现程序员小明认为某个功能应该放在A层，而程序员老王却认为同样的功能应该放在B层，这样会导致分层混乱。如果这样的架构进入实际开发落地，则A层和B层就会乱成一锅粥，也就失去了分层的意义。

分层架构之所以能够较好地支撑系统扩展，本质在于**隔离关注点**（separation of concerns），即每个层中的组件只会处理本层的逻辑。比如说，展示层只需要处理展示逻辑，业务层中只需要处理业务逻辑，这样我们在扩展某层时，其他层是不受影响的，通过这种方式可以支撑系统在某层上快速扩展。例如，Linux内核如果要增加一个新的文件系统，则只需要修改文件存储层即可，其他内核层无须变动。

当然，并不是简单地分层就一定能够实现隔离关注点从而支撑快速扩展，分层时要保证层与层之间的依赖是稳定的，才能真正支撑快速扩展。例如，Linux内核为了支撑不同的文件系统格式，抽象了VFS文件系统接口，架构图如下：

![](<https://static001.geekbang.org/resource/image/f3/77/f3df4b47a3fbb3e1975a5f9ae61c1477.jpg?wh=3037*2496>)

如果没有VFS，只是简单地将ext2、ext3、reiser等文件系统划为“文件系统层”，那么这个分层是达不到支撑可扩展的目的的。因为增加一个新的文件系统后，所有基于文件系统的功能都要适配新的文件系统接口；而有了VFS后，只需要VFS适配新的文件系统接口，其他基于文件系统的功能是依赖VFS的，不会受到影响。

对于操作系统这类复杂的系统，接口本身也可以成为独立的一层。例如，我们把VFS独立为一层是完全可以的。而对于一个简单的业务系统，接口可能就是Java语言上的几个interface定义，这种情况下如果独立为一层，看起来可能就比较重了。例如，经典的J2EE分层架构中，Presentation Layer和Business Layer之间如果硬要拆分一个独立的接口层，则显得有点多余了。

分层结构的另外一个特点就是层层传递，也就是说一旦分层确定，整个业务流程是按照层进行依次传递的，不能在层之间进行跳跃。最简单的C/S结构，用户必须先使用C层，然后C层再传递到S层，用户是不能直接访问S层的。传统的J2EE 4层架构，收到请求后，必须按照下面的方式传递请求：

![](<https://static001.geekbang.org/resource/image/b2/21/b2d9b4b5a978dd0f7ef4d4502dec5821.jpg?wh=3773*2762>)

分层结构的这种约束，好处在于强制将分层依赖限定为两两依赖，降低了整体系统复杂度。例如，Business Layer被Presentation Layer依赖，自己只依赖Persistence Layer。但分层结构的代价就是冗余，也就是说，不管这个业务有多么简单，每层都必须要参与处理，甚至可能每层都写了一个简单的包装函数。我以用户管理系统最简单的一个功能“查看头像”为例。查看头像功能的实现很简单，只是显示一张图片而已，但按照分层分册架构来实现，每层都要写一个简单的函数。比如：

Presentation Layer：

```
package layer;
	&nbsp;
	/**
	&nbsp;* Created by Liyh on 2017/9/18.
	&nbsp;*/
	public class AvatarView {
	&nbsp;&nbsp;&nbsp;public void displayAvatar(int userId){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String url = AvatarBizz.getAvatarUrl(userId);
	&nbsp;
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//此处省略渲染代码
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return;
	&nbsp;&nbsp;&nbsp;}
	}
```

Business Layer：

```
package layer;
	&nbsp;
	/**
	&nbsp;* Created by Liyh on 2017/9/18.
	&nbsp;*/
	public class AvatarBizz {
	&nbsp;&nbsp;&nbsp;public static String getAvatarUrl(int userId){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return AvatarDao.getAvatarUrl(userId);
	&nbsp;&nbsp;&nbsp;}
	}
```

Persistence Layer：

```
package layer;
	&nbsp;
	/**
	&nbsp;* Created by Liyh on 2017/9/18.
	&nbsp;*/
	public class AvatarDao {
	&nbsp;&nbsp;&nbsp;public static String getAvatarUrl(int userId) {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//此处省略具体实现代码，正常情况下可以从MySQL数据库中通过userId查询头像URL即可
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return "http://avatar.csdn.net/B/8/3/1_yah99_wolf.jpg";
	&nbsp;&nbsp;&nbsp;}
	}
```

可以看出Business Layer的AvatarBizz类的getAvatarUrl方法和Persistence Layer的AvatarDao类的getAvatarUrl方法，名称和参数都一模一样。

既然如此，我们是否应该自由选择是否绕过分层的约束呢？例如，“查看头像”的示例中，直接让AvatarView类访问AvatarDao类，不就可以减少AvatarBizz的冗余实现了吗？

答案是不建议这样做，分层架构的优势就体现在通过分层强制约束两两依赖，一旦自由选择绕过分层，时间一长，架构就会变得混乱。例如，Presentation Layer直接访问Persistence Layer，Business Layer直接访问Database Layer，这样做就失去了分层架构的意义，也导致后续扩展时无法控制受影响范围，牵一发动全身，无法支持快速扩展。除此以外，虽然分层架构的实现在某些场景下看起来有些啰嗦和冗余，但复杂度却很低。例如，样例中AvatarBizz的getAvatarUrl方法，实现起来很简单，不会增加太多工作量。

分层架构另外一个典型的缺点就是性能，因为每一次业务请求都需要穿越所有的架构分层，有一些事情是多余的，多少都会有一些性能的浪费。当然，这里所谓的性能缺点只是理论上的分析，实际上分层带来的性能损失，如果放到20世纪80年代，可能很明显；但到了现在，硬件和网络的性能有了质的飞越，其实分层模式理论上的这点性能损失，在实际应用中，绝大部分场景下都可以忽略不计。

## SOA

SOA的全称是Service Oriented Architecture，中文翻译为“面向服务的架构”，诞生于上世纪90年代，1996年Gartner的两位分析师Roy W. Schulte和Yefim V. Natis发表了第一个SOA的报告。

2005年，Gartner预言：到了2008年，SOA将成为80%的开发项目的基础（[https://www.safaribooksonline.com/library/view/soa-in-practice/9780596529550/ch01s04.html](<https://www.safaribooksonline.com/library/view/soa-in-practice/9780596529550/ch01s04.html>)）。历史证明这个预言并不十分靠谱，SOA虽然在很多企业成功推广，但没有达到占有绝对优势的地步。SOA更多是在传统企业（例如，制造业、金融业等）落地和推广，在互联网行业并没有大规模地实践和推广。互联网行业推行SOA最早的应该是亚马逊，得益于杰弗·贝索斯的远见卓识，亚马逊内部的系统都以服务的方式构造，间接地促使了后来的亚马逊云计算技术的出现。

SOA出现 的背景是企业内部的IT系统重复建设且效率低下，主要体现在：

- 企业各部门有独立的IT系统，比如人力资源系统、财务系统、销售系统，这些系统可能都涉及人员管理，各IT系统都需要重复开发人员管理的功能。例如，某个员工离职后，需要分别到上述三个系统中删除员工的权限。

- 各个独立的IT系统可能采购于不同的供应商，实现技术不同，企业自己也不太可能基于这些系统进行重构。

- 随着业务的发展，复杂度越来越高，更多的流程和业务需要多个IT系统合作完成。由于各个独立的IT系统没有标准的实现方式（例如，人力资源系统用Java开发，对外提供RPC；而财务系统用C#开发，对外提供SOAP协议），每次开发新的流程和业务，都需要协调大量的IT系统，同时定制开发，效率很低。


<!-- -->

为了应对传统IT系统存在的问题，SOA提出了3个关键概念。

1. 服务

<!-- -->

所有业务功能都是一项服务，服务就意味着要对外提供开放的能力，当其他系统需要使用这项功能时，无须定制化开发。

服务可大可小，可简单也可复杂。例如，人力资源管理可以是一项服务，包括人员基本信息管理、请假管理、组织结构管理等功能；而人员基本信息管理也可以作为一项独立的服务，组织结构管理也可以作为一项独立的服务。到底是划分为粗粒度的服务，还是划分为细粒度的服务，需要根据企业的实际情况进行判断。

2. ESB

<!-- -->

ESB的全称是Enterprise Service Bus，中文翻译为“企业服务总线”。从名字就可以看出，ESB参考了计算机总线的概念。计算机中的总线将各个不同的设备连接在一起，ESB将企业中各个不同的服务连接在一起。因为各个独立的服务是异构的，如果没有统一的标准，则各个异构系统对外提供的接口是各式各样的。SOA使用ESB来屏蔽异构系统对外提供各种不同的接口方式，以此来达到服务间高效的互联互通。

3. 松耦合

<!-- -->

松耦合的目的是减少各个服务间的依赖和互相影响。因为采用SOA架构后，各个服务是相互独立运行的，甚至都不清楚某个服务到底有多少对其他服务的依赖。如果做不到松耦合，某个服务一升级，依赖它的其他服务全部故障，这样肯定是无法满足业务需求的。

但实际上真正做到松耦合并没有那么容易，要做到完全后向兼容，是一项复杂的任务。

典型的SOA架构样例如下：

![](<https://static001.geekbang.org/resource/image/40/40/40f125000a4f9fda00d612a2b3171740.jpg?wh=2961*2242>)

SOA架构是比较高层级的架构设计理念，一般情况下我们可以说某个企业采用了SOA的架构来构建IT系统，但不会说某个独立的系统采用了SOA架构。例如，某企业采用SOA架构，将系统分为“人力资源管理服务”“考勤服务”“财务服务”，但人力资源管理服务本身通常不会再按照SOA的架构拆分更多服务，也不会再使用独立的一套ESB，因为这些系统本身可能就是采购的，ESB本身也是采购的，如果人力资源系统本身重构为多个子服务，再部署独立的ESB系统，成本很高，也没有什么收益。

SOA解决了传统IT系统重复建设和扩展效率低的问题，但其本身也引入了更多的复杂性。SOA最广为人诟病的就是ESB，ESB需要实现与各种系统间的协议转换、数据转换、透明的动态路由等功能。例如，下图中ESB将JSON转换为Java（摘自《Microservices vs. Service-Oriented Architecture》）。

![](<https://static001.geekbang.org/resource/image/fa/84/fa9e178249d261b1e29850f3c9dc2684.jpg?wh=3368*1583>)

下图中ESB将REST协议转换为RMI和AMQP两个不同的协议：

![](<https://static001.geekbang.org/resource/image/b0/c8/b04b7a89e8e859d84c70e55aa05372c8.jpg?wh=3332*1435>)

ESB虽然功能强大，但现实中的协议有很多种，如JMS、WS、HTTP、RPC等，数据格式也有很多种，如XML、JSON、二进制、HTML等。ESB要完成这么多协议和数据格式的互相转换，工作量和复杂度都很大，而且这种转换是需要耗费大量计算性能的，当ESB承载的消息太多时，ESB本身会成为整个系统的性能瓶颈。

当然，SOA的ESB设计也是无奈之举。回想一下SOA的提出背景就可以发现，企业在应用SOA时，各种异构的IT系统都已经存在很多年了，完全重写或者按照统一标准进行改造的成本是非常大的，只能通过ESB方式去适配已经存在的各种异构系统。

## 小结

今天我为你讲了传统的可扩展架构模式，包括分层架构和SOA架构，希望对你有所帮助。

这就是今天的全部内容，留一道思考题给你吧，为什么互联网企业很少采用SOA架构？

欢迎你把答案写到留言区，和我一起讨论。相信经过深度思考的回答，也会让你对知识的理解更加深刻。（编辑乱入：精彩的留言有机会获得丰厚福利哦！）

## 精选留言(62)

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34.jpeg)

  Lee

  SOA是把多个系统整合，而微服务是把单个系统拆开来，方向正好相反

  作者回复: 言简意赅👍

  2018-07-19

  **2

  **201

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881710.jpeg)

  辉辉

  soa是集成的思想，是解决服务孤岛打通链条，是无奈之举。esb集中化的管理带来了性能不佳，厚重等问题。也无法快速扩展。不适合互联网的业务特点

  作者回复: 赞同👍

  2018-07-18

  **

  **72

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881711.jpeg)

  铃兰Neko

  尝试说下个人浅见: 为什么互联网不用SOA? 1. 互联网企业, 通常比较年轻, 没有那么多异构系统, 技术是公司的关键; 如果有整合或者服务化的需求, 公司有人也有钱专门搞这个; 拆到重做/重构 很平常; 相反的, 传统企业, 举个例子:  某传统炼钢国企 : 有多个遗留.net系统,有几个实习生做的java系统, 有基于数据库procedure的系统; 有各种已经倒闭了的第三方企业的系统 等等; 企业领导不会有精力和想法全部推倒重来, 只会花钱请第三方 , 成本越低越好;  这个时候就需要ESB这种总线 2. 传统企业IT追求的是"需求灵活,变更快", 而互联网企业追求性能, 传统soa 性能不佳 传统的esb ,说实话, 使用webservice 以及soap这种基于xml的技术; wsdl 这东西是真的难用, 难学难用难维护 ; 结构冗杂; 3. soa 这个东西很多时候只是一个概念, 而不是实践 个人觉得, 现在的微服务 , 更像是 soa 思想的一个落地 (相比esb)	

  作者回复: 分析的很好，微服务和SOA的关系后面会讲

  2018-07-12

  **2

  **43

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132.png)

  赤脚小子

  回答问题:文中也说了，soa是特定历史条件下的产物，为了适配各种异构的it系统，而有如此多系统的自然是变化减少且稳定的传统企业。互联网企业的特点就是小，新，快。没有历史包袱，变化快，大部分是从单体演进到分布式，技术栈一脉相承或者在分布式之前已经从php,ruby等改造到java等了。而到了分布式之后，面对不断的耦合，系统复杂度的陡增，这时一个soa的特例微服务出现了。 实际上soa的思想还在，只不过实现的方式不一样了。

  作者回复: 关于soa和微服务的关系，我会特别讲述

  2018-07-12

  **

  **15

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881712.jpeg)![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,w_14.png)

  xxx

  一直不明白SOA和微服务的具体区别，知道作者讲到了ESB 的功能，原来就是适配各种协议，顿时明白了!SOA是为了适配老系统。

  作者回复: 是的，所以SOA不适合创新型的互联网企业，比较适合传统大企业

  2019-10-25

  **

  **13

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881713.jpeg)

  孙振超

  在传统企业从原先的手工作业转为采用IT系统作业的过程中，大多是采用向外采购的方式逐步实现的，在这个过程中不同部门采购系统的实现语言、通信协议并不完全相同，但为提升运行效率又要能够做到企业内部信息互通、相互协作，这是soa诞生的背景。 而互联网企业是新创的企业，没有这么多的历史包袱，同时出于快速迭代的要求，有时会自建所需的系统，即使是对外采购，也会选择和已有系统对接方便的系统，从根本上避免了相关问题，因而soa在互联网公司中使用不多。

  作者回复: 赞同👍

  2018-09-09

  **

  **10

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881714.jpeg)

  hello

  soa解决的是资源的重复利用，它的拆分粒度比较大，比如财务系统跟oa系统的员工模块。1.互联网企业有几种情况，1.初创公司，这种公司一般会有试错的过程，需要技术快速实现业务落地，这种情况下使用SOA不适合快速敏捷迭代开发。2.对于成熟的互联网业务来说，需要解决的是是高并发，高性能和高存储等一系列问题，对于这类企业来说，使用SOA拆分不能解决太多问题，还得做更加细粒度的拆分。

  作者回复: 分析到位👍

  2018-08-21

  **

  **10

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881715.jpeg)

  7侠

  ​    SOA更像一种架构理念，不够具体。在传统企业的IT系统中落地为ESB，主要是为了集成异构系统。因为传统企业特别是大型企业的历史长，在其发展过程中自己开发或采购了不少异构系统。    而互联网企业历史都短(腾讯98年，阿里99年，百度2000年)，很少有遗留异构系统(像阿里的系统绝大部分应该都是Java开发的吧？)。像阿里这种互联网大型企业的痛点是随着业务越来越多，整个系统成了个巨无霸(可能是数以千记的模块数)，模块之间的调用像蜘蛛网，极大降低了开发、测试、部署、运维的效率，所以把庞大的业务逻辑层又切分成了业务更独立的应用层和公共功能模块组成服务层。接下来一是要提供应用层与服务层之间、服务层内部服务之间的高效通信机制，二是要对大量的服务进行治理，于是分布式服务框架出现了(阿里就出了Dubbo和HSF两个服务框架？)。感觉在大型互联网企业，SOA实际是落地为分布式服务框架，它更像是微服务架构的一个雏形，服务框架提供的功能实际也是微服务架构里必不可少的功能。

  作者回复: 确实也有人将SOA理解为一个思想，微服务理解为SOA的具体实现

  2019-05-04

  **

  **7

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  yason li

  其实，个人理解的传统SOA和ESB在互联网企业之所以不怎么使用主要原因就是中心化的ESB架构会逐渐成为性能、可用性、可扩展性的瓶颈。但是SOA的思想本身是没有什么问题的。互联网企业中用的微服务甚至最近很火的Service Mesh都可以看成是SOA、ESB的变形。比如Service Mesh也可以看成是一个去中心化的ESB。

  作者回复: 这也是一种理解方式吧，微服务基础设施做完，确实感觉是做了一个ESB

  2018-07-18

  **

  **4

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881716.jpeg)

  欧星星

  1.没有历史包袱 2.SOA架构太重，ESB容易成瓶颈

  作者回复: 言简意赅😄

  2018-07-12

  **

  **4

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881717.jpeg)

  tim

  soa是不是可以分成两部分： 1.服务化的思想，这个就是SOA 。 2.带有ESB 的实践，这个是针对当时问题的一种解决方案。 如果是这样其实微服务就是SOA的另一种实践。互联网公司其实是用了SOA的，不过esb和服务划分粒度已经不适合他们的场景了。 先接触了分布式和微服务，对SOA不是很了解，感觉都是分而治之的思想，粒度渐细，自动运维什么的附属产物

  作者回复: 看SOA和微服务的对比

  2019-06-13

  **

  **3

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141881718.jpeg)

  何磊

  soa主要是为了解决历史遗留问题，引入的esb。而互联网企业都年轻，没有历史包袱，另外互联网企业大部分都需要高性能，而esb可能成为瓶颈。 最后想咨询老师一个问题：在分层结构中。同层能不能互相调用？比如：下单时，需要用户信息，此时应该调用同层用户模块来完成，还是如何处理呢？

  作者回复: 当然可以互相调用，特别是业务逻辑，如果说同层不能互相调用，这代码没法写

  2018-08-10

  **

  **3

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141881719.jpeg)

  gen_jin

  我觉得有三点～互联网企业1.没有老系统包袱，2.钱多 3.需求变化快 2c性能及并发量要求高，三高(高性能 可用 扩展)，传统soa(esb)无法满足。

  作者回复: 有的人说是互联网企业钱少，买不起ESB

  2018-07-12

  **

  **3

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891720.jpeg)

  小橙橙

  老师，有个疑问并不是很理解，互联网企业多数是采用微服务架构，那微服务不属于面向服务SOA架构的范畴吗？

  作者回复: 微服务章节我会讲

  2018-07-12

  **

  **3

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891721.jpeg)

  cqc

  我觉得主要有以下几个原因： 1.互联网公司大多比较年轻，没有已有系统的历史包袱，所以不用考虑各种兼容的问题； 2.互联网公司大多业务量都比较大，对于性能要求比较高，所以不会优先考虑SOA

  作者回复: 其实大部分互联网公司开始的时候业务量真不大😄

  2018-07-12

  **

  **2

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891722.jpeg)![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,w_14.png)

  oddrock

  一、互联网企业的it系统大多数都是自己的it部门开发的，不存在太多异构问题，可以有较好的内部服务规范，因此不需要soa和esb去做异构系统的屏蔽和互通 二、esb的总线型架构会导致一定程度上的性能问题，因此互联网企业一般采用分布式服务架构 三、esb实质是用空间和时间换取it系统架构的有序性，esb本身有采购或研发费用，esb部署也要服务器，这些成本对互联网企业都是不必要的

  作者回复: 一个字：穷，四个字：技术牛逼😄😄

  2018-07-12

  **

  **2

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891723.jpeg)

  narry

  互联网行业很少采用soa，感觉有两点原因:1）soa主要是解决异构系统之间的集成，传统企业有大量的异构系统，而互联网属于新兴行业，不存在大量的异构系统需要集成，2）esb存在性能的瓶颈和不易扩展的问题，无法应对互联网这种业务会快速增长场景

  作者回复: 赞同

  2018-07-12

  **

  **2

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891724.jpeg)

  悟空

  13年刚开始学习ESB的时候一知半解，还是要多做总结。

  2019-12-11

  **

  **1

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891725.jpeg)![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,w_14.png)

  梁中华

  05,06年服务化刚开始流行的时候，我就找了很多SOA的书，很多书上都讲到ESB，我记得当时还有个叫mule的很流行的总线，当时就觉得有点奇怪，这个东东明显很重，有点鸡肋，不要这个东西，也完全跑的很好。原来是个历史的产物。

  2019-05-10

  **

  **1

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891726.jpeg)

  雨幕下的稻田

  第一章说MVC是开发规范，本章节说MVC是分层架构，不太明白，这两MVC说的不是一个东西吗，还是出发点不同

  作者回复: 按照MVC开发规范的系统就是MVC架构，这个是逻辑上的架构

  2019-02-15

  **

  **1

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891727.jpeg)

  华烬

  请教一下，像dubbo这种，可以做到服务注册，服务发现，统一协议，服务之间都是各自调用，这种算不算soa？

  2018-10-08

  **

  **1

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891728.jpeg)![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,w_14.png)

  呆

  SOA厚重，用于将各个系统拼接起来，与互联网的快速迭代理念冲突

  2022-08-02

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891729.jpeg)

  Rainbow福才

  SOA架构用来整合传统企业已有子系统，微服务架构用来拆分负责业务系统。

  作者回复: 正解

  2022-03-01

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891730.jpeg)![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,w_14.png)

  沐雪

  原因有3： 1、SOA出现的背景是 各种异构系统的不同协议之间的融合；而互联网行业根据就没有这个需求。 2、SOA服务的粒度不好控制，但是肯定是微服务粗。 3、SOA实现比较复杂，比较重。

  作者回复: 正解

  2022-02-07

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891731.jpeg)

  DDL007

  企业经营本质上是业务运营。在技术对业务的贡献度上，传统企业的认识远没有互联网企业认识深刻，即便有认识，投入上也远远不够。互联网企业则是既有认识，又愿意投入。所以，EBS对传统企业是在将就，互联网企业则是想办法彻底解决，后者显然是不将就的，也就不会引入EBS。

  作者回复: ESB也不是将就，而是2B环境造成的，传统企业没有自己实力强大的研发团队，事实上他们也不需要自己养一个强大的研发团队，而是将系统开发工作外包，外包就会涉及不同的团队不同的技术栈，因此ESB才有用武之地。

  2021-12-09

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891732.jpeg)

  小超人

  互联网企业没有传统企业这么多的历史包袱， SOA 主要是解决现有的多个系统的协调问题

  作者回复: 是的，所以不是SOA不好，每个技术都有适应场景

  2021-10-18

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141891733.jpeg)

  Drake敏

  SOA更像是一套完整的系统，如果一出问题就会服务全挂。微服务的话就算注册挂了登陆服务还是ok的

  作者回复: 本质就是服务的粒度不同

  2021-09-22

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141901734.jpeg)

  ZHANGPING![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/8ac15yy848f71919ef784a1c9065e495.png)

  互联网本身就是技术和业务对半的性质，有技术问题，就解决即使问题。而传统行业：IT只是一个解决问题的工具，并不擅长技术。

  作者回复: 非常正确

  2021-07-17

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141901735.jpeg)

  Geek_626167

  首先各个服务之间可能会有很多共同的功能模块，如果按服务划分，可能存在重复开发的问题；其次，ESB层建设耗费大量资源但收益是很低的。 互联网服务往往单个服务是瓶颈，适合逐个优化扩展，如果建设ESB这种集中层，反而会拖垮整个系统，如果基于服务做拆分更适合用微服务架构。

  作者回复: 不是这么简单的，后面的文章会告诉你，微服务的基础设施现在已经比ESB还复杂了

  2021-07-16

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141901736.jpeg)

  蓝诚

  SOA是整合已有系统服务，微服务是拆分或重构复杂系统服务

  作者回复: 一针见血 ：）

  2021-06-02

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141901737.jpeg)

  prader26

  今天我来介绍传统的可扩展模式，包括分层架构和 SOA。 1 C/S 架构、B/S 架构 2 MVC 架构、MVP 架构 3 逻辑分层架构 SOA 提出了 3 个关键概念。1 服务 2 ESB  3 松耦合

  2021-04-18

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141901738.jpeg)

  TorresTu

  互联网的特点就是海量，传统的SOA使用的ESB会成为瓶颈，所以才会有中台这样的理念，虽然互联网不承认自己是SOA，但我认为其实也是一种SOA，只是实现的方式更贴近互联网思维，不是传统的SOA

  作者回复: 我在微服务章节解释了SOA和微服务，本质都是服务拆分，但具体的实现方式不同。我理解你说的的“也是一种SOA”其实就是指“服务拆分”这部分

  2021-03-19

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  袁军

  说到底各种架构模式的产生最终是为了提高程序员的生产力，能满足业务需要，易学易用的就会更流行。SOA主要是规范与理论过于严谨和复杂，正所谓物极必反，才有微服务的出现。

  作者回复: SOA主要是ESB太重了，为了兼容各种异构系统不得以为之。

  2021-02-20

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141901739.jpeg)

  小神david

  反过来想下，为什么传统企业更愿意采用 SOA 架构，理解下可能原因是：传统企业的系统确实大部分是独立的、分散的，整合到一起就需要服务化改造；而另一方面，传统企业的系统又没有那么复杂、更新频率也并不高，不会涉及到非常庞大的协议转换。回到互联网企业来看，互联网企业的系统发展比较快、更新迭代速率高，想要实现一套ESB并且不断更新实在太难了... 另一方面，互联网企业技术人员储备相对比较丰富，可以充分发挥在每个领域、每个系统中的服务化开发。总体感觉，尽管互联网企业没有大规模采用SOA架构，但是它的架构设计理念已经被采用和渗透，只是实现的方式更加“互联网”化。

  作者回复: 你的分析思路很不错，里面有个细节我纠正一下：传统企业的服务是“很复杂”而不是“没有那么复杂”，但是更新频率确实不高

  2020-12-30

  **2

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_7389a6

  ESB太重了，一般的互联网企业无福消受，性价比低

  作者回复: 其实现在微服务全套搞下来，不见得比ESB简单了，ESB的核心问题是所有东西都在一套系统里面😃

  2020-03-30

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141911740.jpeg)

  Kevin

  分层和SOA放到一起讲，二者不对立吧？分层的思想我想在SOA的服务，甚至微服务的设计时也应该遵循的思想吧？

  作者回复: 不对立，但关注点也不一样

  2020-03-19

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911741.jpeg)

  谭方敏

  分层架构，好处：结构清晰，坏处：浪费，不能快层调用。 soa架构，好处：减少重复，坏处：esb带来复杂度。 为什么不用soa？soa性价比不高，减少重复建设带来的收益率低于soa引入esb带入的复杂度，是鸡肋。

  2020-03-10

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911742.jpeg)

  J.Smile

  感觉ESB就像一个集中式的网关一样…

  作者回复: 是的，不单是网关，是整个微服务基础组件都在里面

  2020-02-23

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911743.jpeg)

  钱

  呵呵，感觉内容挺丰富，不过学完后我又不清楚自己究竟学习到了什么？可能需要休息，看的多了，用力过猛。 目前公司要求开发要实现组件化、服务化、可自由组合、可灵活插拔，有点晕，这是微服务化还是SOA？感觉它们是一体多面，是站在不同角度的描述？

  作者回复: 多看，或者实践后再来看

  2019-09-01

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911744.jpeg)

  公告-SRE运维实践

  我们的api网关用了esb来进行内部协议的转换，启动时间太长了

  作者回复: ESB太重

  2019-08-22

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141911745.jpeg)

  gkb111

  分层架构，两层cs三层，mvc，有逻辑分层，降低系统整体复杂性，相邻模块关联， soa架构

  2019-05-06

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911746.jpeg)

  日光倾城

  互联网企业比较新，没有什么历史包袱系统

  2019-03-29

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911747.jpeg)

  蓝焰

  mvc是框架还是架构？

  作者回复: 看来你需要复习第一篇 01|架构到底是什么😊

  2018-11-01

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911748.gif)

  包子

  SOA的目的更多是整合现有系统，兼容。看了后面章节，主要从细粒度，传输协议，自动化，应用场景方面和微服务有较大的区别

  2018-10-11

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911749.jpeg)

  文竹

  互联网企业业务的特点：变化快，用户量大，业务复杂。 传统企业业务的特点：几乎没有什么变化，用户基数小，一般使用的系统都存在一些年了，各个系统的开放接口差异大。 从性能上看，由于互联网用户量大，使用ESB容易出现性能瓶颈，让服务独立调用其他服务是一种很好的做法（去中心化）。

  2018-08-25

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911750.jpeg)

  吕韬

  小团队 0到1 喊一嗓子就完事了

  作者回复: 小团队坐在一起，都不用喊，确认过眼神就可以了😄😄

  2018-07-26

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911750.jpeg)

  吕韬

  总监桌子一拍 统一json格式 esb就没有什么鸟用了

  作者回复: 还有传输协议呢？统一为HTTP么？😀

  2018-07-19

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141911751.jpeg)

  鹏飞天下

  我对soa一直是不太清楚，其中ESB是不是可以理解为我们用的dubbo，soa可不可以理解为把项目进行粗力度的拆分，通过dubbo或者http协议向外提供服务

  作者回复: dubbo不是ESB，dubbo是统一的协议，ESB需要兼容和适配很多协议。 soa主要不是为了拆分，而是将已经存在的异构系统整合起来

  2018-07-17

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921752.jpeg)

  云学

  之前还以为SOA在互联网很流行呢，没想到根本不用啊

  作者回复: 用得少，都用微服务了

  2018-07-17

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921753.jpeg)

  卫江

  不知道大家为什么总是提到esb的问题，从文章中可以看到esb的出现也是无奈之举，互联网完全可以做到统一，不需要esb，而且soa只是在功能分解上面的发展，而按照功能分解实现是很早就开始的，所以，不明白为什么互联网公司soa比较少。

  作者回复: 虽然是无奈，但确实太重量级了，互联网公司soa用的少，但微服务做到最后，复杂度和ESB其实也差不多了

  2018-07-16

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921754.jpeg)

  小喵喵

  esb是硬件吗？是需要购买吗？

  作者回复: 中间件系统，IBM, ORACLE, Microsoft都有

  2018-07-15

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921755.jpeg)

  张玮(大圣)

  想想人们解决问题是矛盾的，想通过总线屏蔽异构系统的繁琐复杂无趣的工作，又想把这些隔离层工作做的小，做得轻，😄，合久必分，分久必合 互联网也有异构系统，只是相对少些吧，我觉得互联网公司不怎么采用的原因是开发成本，效率，以及后续的扩展性这些方面权衡的，说白了，现在甚至很多公司没有权衡，全家桶甩起来，100个创业公司能活下来几个，牛掰起来再做，不晚的哈

  作者回复: 如果说开发效率，传统企业也类似哦

  2018-07-13

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921756.jpeg)

  Skysper![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/bba6d75e3ea300b336b4a1e33896a665.png)

  也有疑惑，SOA与微服务要怎么做严格的区分？互联网应用开放性高 自研应用可控性高 不依赖ESB这类的设施 可通过其他协议或通信机制进行服务实现 部分功能可适当冗余开发

  作者回复: 微服务部分会讲

  2018-07-13

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921757.jpeg)

  空档滑行

  互联网企业发展的时间不长，没有太多历史的it系统需要对接维护，大部分系统在采购时可能就已经考虑了接口的问题 互联网业务和产品迭代更新快，相应的业务系统被淘汰的周期很短，搭建soa架构太大，不利于小业务的迭代和试错

  作者回复: 微服务完整做下来架构也不小

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921758.jpeg)

  张立春

  也有相同疑问：现在的互联网系统中流行的微服务难道不也是面向服务的一种架构？SOA和互联网架构有什么本质区别？

  作者回复: 微服务章节会讲

  2018-07-12

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Xg huang

  我个人觉得这个跟互联网公司的组织架构和业务特点有关，以我们公司为例，一个项目，第一个版本，从开发到上线就两周，真正的开发时间只有一周，如果按soa的套路来，时间至少多一倍。业务通常都不愿意花这个时间来等待soa带来的收益。另外就是互联公司所开发出来的产品它们之间的重合度不高，比如说将小视频业务soa后，并不会给直播业务带来什么收益

  作者回复: 那这样的话微服务其实也没必要呢😄

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921759.jpeg)

  肉松

  1.互联网业务场景注重高性能高可用，SOA的ESB性能达不到且存在单点问题；2.为了性能，互联网公司会使用一些私有协议，ESB不太好支持或者支持后性能损失也已经非常严重了；3.互联网很多业务系统生命周期短，不是那么稳定，这类系统采用SOA架构，效率低和维护成本过高；

  作者回复: ESB在协议适配方面那是无敌的哦😄

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/132-16618728141921760.jpeg)

  XNorth

  互联网研发的系统，大多是成套完整的。存在需要相互关联性的子系统不多，如果关联性较强会在开始设计时就集成到一起。最后就是减少因soa分层导致的维护扩展成本。

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921761.jpeg)

  feifei

  Soa架构，简单分层就3层，应用，esb,服务， 这种架构适合企业本身已经存在很多的系统。 优点 1，现有系统可重用性最大 2，支持扩展 缺点 1，需要增加成本，即ESB，需要专门的转化层 2，性能损失，多增加了一层网络调用，时延增加 3，扩展系统需要扩展2层，ESB和服务 SOA在互联网中优势不明显，反尔代价更高，而且互联网应用大多都是自研项目，可以统一化对外提供接口格式

  作者回复: 赞同

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921762.jpeg)

  何国平

  soa架构要有很强控制力的技术部门才行。小公司追求快速开发，根本不管这事，而随着业务的增加，很多功能冗余也造成整合困难。

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921763.jpeg)

  甲韭

  可能是ESB 很贵，而且还不单独销售

  作者回复: 互联网企业刚开始都穷么😂😂

  2018-07-12

  **

  **

- ![img](33%20_%20%E4%BC%A0%E7%BB%9F%E7%9A%84%E5%8F%AF%E6%89%A9%E5%B1%95%E6%9E%B6%E6%9E%84%E6%A8%A1%E5%BC%8F%EF%BC%9A%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84%E5%92%8CSOA.resource/resize,m_fill,h_34,w_34-16618728141921764.jpeg)

  郭涛

  SOA架构中ESB容易成为性能瓶颈，而互联网服务相比企业it系统会有更高的并发。而且互联网企业相比传统企业，异构it系统的历史包袱也没那么重。

  作者回复: 赞同

  2018-07-12
