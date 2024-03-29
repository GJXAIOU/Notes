# 开篇词 | Java程序员如何快速成长？

如果你和我一样选择了 Java Web 开发这个方向，并且正在学习和提高的路上，你一定思考过这个问题：

**我怎样才能成长为一名高级程序员或者架构师？**

对于这个问题，每个人的答案都可能都不太一样，我先来讲讲我的经历。十年前我在实习的时候是做嵌入式系统开发，用的开发语言是 C 和 C++。出于我个人的兴趣爱好，当时我想转 Java，在学了一段时间的 Java 后，发现 Java 上手还是挺快的，API 比较齐全，而且也不需要自己来管理内存，感觉比 C 语言高级。毕业后我也顺利地找到了一个 Java 开发的工作，入职后我的工作主要是实现一些小模块，很多时候通过代码的复制粘贴，再稍微改改就能完成功能，这样的状态大概持续了一年。

在这个过程中，虽然我对 Java 语法更加熟悉了，也“背”过一些设计模式，用过一些 Web 框架，但是我很少有机会将一些 Java 的高级特性运用到实际项目中，因此我对它们的理解也是模糊的。那时候如果让我独立设计一个系统，我会感到非常茫然，不知道从哪里下手；对于 Web 框架，我也只是知道这样用是可以的，不知道它背后的原理是什么。并且在我脑子里也没有一张 Java Web 开发的全景图，比如我并不知道浏览器的请求是怎么跟 Spring 中的代码联系起来的。

后来我分析发现，我的知识体系在广度和深度上都有问题。为了突破这个瓶颈，我当时就想，为什么不站在巨人的肩膀上学习一些优秀的开源系统，看看大牛们是如何思考这些问题的呢。

于是我注意到了像 Tomcat 和 Jetty 这样的 Web 容器，觉得它们很神奇，只需要把 Web 应用打成 WAR 包放到它的目录下，启动起来就能通过浏览器来访问了，我非常好奇 Web 容器是如何工作的。此外 Tomcat 的设计非常经典，并且运用了方方面面的 Java 技术，而这些正好是我欠缺的，于是我决定选择 Tomcat 来深入研究。

**学习了 Tomcat 的原理之后，我发现 Servlet 技术是 Web 开发的原点，几乎所有的 Java Web 框架（比如 Spring）都是基于 Servlet 的封装，Spring 应用本身就是一个 Servlet，而 Tomcat 和 Jetty 这样的 Web 容器，负责加载和运行 Servlet**。你可以通过下面这张图来理解 Tomcat 和 Jetty 在 Web 开发中的位置。

![image-20220814200448175](%E5%BC%80%E7%AF%87%E8%AF%8D%20%20Java%E7%A8%8B%E5%BA%8F%E5%91%98%E5%A6%82%E4%BD%95%E5%BF%AB%E9%80%9F%E6%88%90%E9%95%BF%EF%BC%9F.resource/image-20220814200448175.png)

随着学习的深入，我还发现 Tomcat 和 Jetty 中用到不少 Java 高级技术，比如 Java 多线程并发编程、Socket 网络编程以及反射等等。之前我仅仅只是了解这些技术，为了面试也背过一些题，但是总感觉“知道”和“会用”之间存在一道鸿沟。**通过对 Tomcat 和 Jetty 源码的学习，我学会了在什么样的场景下去用这些技术，这一点至关重要。**

**还有就是系统设计能力，Tomcat 和 Jetty 作为工业级的中间件，它们的设计非常优秀，比如面向接口编程、组件化、骨架抽象类、一键式启停、对象池技术以及各种设计模式，比如模板方法、观察者模式、责任链模式等，之后我也开始模仿它们并把这些设计思想运用到实际的工作中。**

在理解了 Web 容器以及 JVM 的工作原理后，我开始解决线上的疑难杂症，并且尝试对线上的 Tomcat 进行调优。性能的提升也是实实在在的成果，我也因此得到了同事们的认可。

总之在这个过程中，我逐渐建立起了自己的知识体系，也开始独立设计一个系统，独立解决技术难题，也就是说我渐渐具备了**独当一面**的能力，而这正是高级程序员或者架构师的特质。

概括一下，独当一面的能力，离不开**技术的广度和深度**。

**技术的广度体现在你的知识是成体系的，从前端到后端、从应用层面到操作系统、从软件到硬件、从开发、测试、部署到运维**…有些领域虽然你不需要挖得很深，但是你必须知道这其中的“门道”。

**而技术的深度体现在对于某种技术，你不仅知道怎么用，还知道这项技术如何产生的、它背后的原理是什么，以及它为什么被设计成这样，甚至你还得知道如何去改进它。**

但是人的精力是有限的，广度和深度该如何权衡呢？我建议找准一个点先突破深度，而 Tomcat 和 Jetty 就是非常好的选择。但同时它们也是比较复杂的，具体应该怎么学呢？我想通过这个专栏，来分享一些我的经验。

首先我们要学习一些基础知识，比如操作系统、计算机网络、Java 语言，面向对象设计、HTTP 协议以及 Servlet 规范等。

接下来我们会学习 Tomcat 的 Jetty 的总体架构，并从全貌逐步深入到各个组件。在这个过程中，我会重点关注组件的工作原理和设计思路，比如这个组件为什么设计成这样，设计者们当时是怎么考虑这个问题的。然后通过源码的剖析，加深你的理解。**更重要的是，帮你学会在真实的场景下如何运用 Java 技术**。

同时我还会通过 Jetty 与 Tomcat 的对比，比较它们各自的设计特点，让你对选型有更深的理解。并且通过思考和总结，帮你从中提炼一些通用的设计原则，以及实现高性能高并发的思路。

在深入了解 Tomcat 和 Jetty 的工作原理之后，我会从实战出发，带你看看如何监控 Tomcat 的性能，以及怎么从内存、线程池和 I/O 三个方面进行调优，同时我也还会分析和解决一些你在实际工作中可能会碰到的棘手问题。

在这个过程中，我还会介绍 Tomcat 和 Jetty 支持的 Servlet 新技术，比如 WebSocket 和异步 Servlet 等，我会重点分析这些新技术是从何而来，以及 Tomcat 和 Jetty 是如何支持的。这些都是 Web 技术的最新动向，你可以在自己的工作中根据需要选用这些新技术。

总之，弄懂了 Tomcat 和 Jetty，Java Web 开发对你来说就已经毫无“秘密”可言。并且你能体会到大神们是如何设计 Tomcat 和 Jetty 的，体会他们如何思考问题、如何写代码。比如怎样设计服务端程序的 I/O 和线程模型、怎样写高性能高并发程序、Spring 的 IoC 容器为什么设计成这个样子、设计一个中间件或者框架有哪些套路等…这些都能快速增加你的经验值。

成长的道路没有捷径，不仅需要上进心和耐心，还要保持对知识的好奇心。如果你也想在技术和视野上有所突破，拥有独当一面的能力，从 Tomcat 和 Jetty 入手是一个非常好的选择，我也邀请你与我一起探究 Tomcat 和 Jetty 的设计精髓，一起收获经验、享受成长。

最后，如果你正在 Java Web 开发这条路上向着架构师的方向狂奔，欢迎你给我留言，讲讲你所付出的努力、遇到了哪些问题，或者写写你对这个专栏的期待，期待与你交流。

## 1716143665 拼课微信(48)

- 学习的时候感觉一直卡在一个level，上不去了
      不知道老师有没有过相似经历
     还有就是老师当年是怎么平衡工作和学习的 如果工作都是增删改查，没有高并发，没有各种新技术，老师会怎么去做

  作者回复: 你说的这个情况很典型，大多数技术人都可能会经历这个瓶颈期，可能你做的项目比较简单或者技术比较陈旧，得不到锻炼的机会，但每天又比较忙，这个时候要勇于打破舒适区，挤出时间来学习一些新东西，学什么呢？五花八门的技术太多了，但是呢这些技术都不开计算机基础，基础扎实了，学习这些新技术才更有效率。但是基础知识也很多，操作系统、算法、网络....学久了容易枯燥，你会怀疑这些知识到底有没有用，难以坚持。这个时候可以读读一些经典的，优秀的源代码，比如源码中用到了高并发技术、用到了Java的各种高级玩法、通用的设计思想，在这个过程中，你会发现自己在基础上还有哪些薄弱点，再查漏补缺，建立起知识体系。但最终要落实你的职业生涯上来，比如这个时候你基础扎实了，深度和广度都有了一定的积累，你可以选择跳槽，也可以换个项目组，因为最终你还是需要通过有挑战、有技术深度的项目来锤炼自己，才能让你的简历更好看..
  
- 老师终于等到你 ， 好早就在期待 tomcat课程了 。

  老师 ， 在我原来的理解 ， spring是一种 servlet , 而 tomcat&jetty 是servlet 容器 ，就是负责给类似spring这种servlet提供一个环境去运行的 。老师我这么理解对么 ？

  但是有了 servlet环境还不够 ，还要有个web环境 ， 这时候tomcat自己可以作为一个独立的web容器 。 早期也可以兼容Apache web容器 。 但是具体 web容器和servlet容器他们俩的分工界限在哪里我也一直不清楚 ， 一名刚毕业不久的程序员 ，努力跟着老师的脚步去学习 😁

  展开**

  作者回复: 橙子你好，你前面的理解是对的。Apache是一个HTTP服务器，而Tomcat或者Jetty是一个HTTP服务器+Servlet容器。HTTP服务器与Servlet容器的功能界限是：你可以把HTTP服务器想象成前台的接待，负责网络通信和解析请求，而Servlet容器是业务部门，负责处理业务请求。

- 老师，你好。很幸运能够学习您的课程。我也是一个web开发者，平常工作也是cv模式。感觉自己好菜，尝试着去看Tomcat和Servlet的书籍和源码，总是感觉浮光掠影。这个怎么办才好？

  还有一些问题，希望老师能够解答一下，感激不尽:
  1.现在的各种中间价、框架都是非常多，对想进阶而言。老师建议阅读和深入学习那些框架和中间件
  2.阅读源码时不时会绕进去，不是很有全局性，所以请教一下阅读源码时候的一些技巧。

  作者回复: 1，我觉得可以从Tomcat/Jetty开始，因为它们跟Web开发紧密相关，一举两得，既学了技术，又弄懂了Web的原理~
  2.是的，看源码很容易迷失在细节里无法自拔：），所以要抓住主线，分析源码之前看看它的主要功能有哪些，比如对于Tomcat、Jetty来说，主线就是启停、请求处理过程和类加载。 另外还是需要把源码跑起来，打断点调试。



- 可以结合深入剖析tomcat这本书，基于tomcat4，5每一章节由浅入深怎么开发一个web容器。

- 一直想深入到tomcat这只喵的底层：包括但不仅限于tomcat的底层数据结构和算法、设计模式、源码、socket编程、并发编程等等等等。以前有尝试过好几次细究tomcat，但总之还是没坚持下来。今天终于等到你。希望和大家一起学习成长。

  在此立下flag：
保持这份学习的热情，定下学透tomcat的目标，坚持不落队读本专栏的每一篇，不懂就问，最后产出-调优公司某一套基于tomcat部署的环境。
  
  展开**
  
  作者回复: 我会循序渐进，逐步深入，你有这样的热情，我相信肯定跟的上的，加油 👍

- 老师好!阅读源码时没法看到全貌，网上找的资料大多没有调理。英语又不好都是连懵带猜的看，往往花了半天可以只看了一个类。进度很慢很容易坚持不下去。想听听老师的起步阶段是怎么走过来了。

  展开**

  作者回复: 主要是抓主线、理清主要功能，再加断点调试，**初期不要太在意细节，后期再细细品味设计和编码。**

- 老师后面能不能也顺带讲讲undertow，之前做过测试，undertow的性能很快，想了解一下他们的不同，谢谢啦

  作者回复: 之前没安排这块内容，只能通过加餐~

  

- 老师，首先谢谢你的分享，自己的收益很大。
  总结下自己目前的问题：
  1.知道知识点，但是不知道如何应用。
  2.自己的系统设计能力不行。
  3.知识没有体系化。
  从老师出得到的解决方案： 站在巨人的肩膀上学习一些优秀的开源系统。感受下大牛是如何思考问题的。
  还想请教老师问题，什么是知识体系化，老师在学习的过程中如何使自己的知识体系化的？

  展开**

  作者回复: 直观一点说，首先你需要有扎实的基础，包括java语言，操作系统，计算机网络，设计模式和数据结构算法。

  然后就是前端后端的各种技术框架中间件：比如JS，Ajax，HTML，AngularJS，React，NodeJS，Nginx，Tomcat/Jetty，Spring，SpringMVC，Spring Boot，Spring Cloud，Mybatis，Mycat，Mysql，Redis，Kafka，mongoDB，ES…… 你不需要精通每种技术，但是你大概知道它们是做什么的，知道按照什么套路把它们组装成一个后台系统，以及信息以一种什么形式在它们中间流转。

- 老师您好，有个问题没太明白，为什么servlet可以在tomcat中运行，自己写main方法，怎样让servlet运行？

  作者回复: Servlet没有main方法，只有一个“service”接口，Tomcat负责调用这个接口。

  程序员一般不会直接调用Servlet的方法，如果想试一下，可以new一个Servlet，然后直接调它的service方法，但是这个方法有两个参数：HttpServletRequest和HTTPServletResponse，这两个参数也是Tomcat准备好的，如果你想直接调Servlet的service方法，你得构造这两个参数。

- 那这里的spring就是特指springMvc吧？

  展开**

  作者回复: 对的

- spring并不一定非要运行在web项目啊，为什么说spring也是个servlet呢？想不明白

  作者回复: SpringMVC实现了Servlet接口
