# 导读 \| 构建Kafka工程和源码阅读环境、Scala语言热身

作者: 胡夕

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/61/d0/617e765c15083ee788e654dfe829f7d0.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/f5/8e/f5dd9821615c4c3dd4e62391b7a94a8e.mp3" type="audio/mpeg"></audio>

你好，我是胡夕。

从今天开始，我们就要正式走入 Kafka 源码的世界了。既然咱们这个课程是教你阅读 Kafka 源码的，那么，你首先就得掌握如何在自己的电脑上搭建 Kafka 的源码环境，甚至是知道怎么对它们进行调试。在这节课，我展示了很多实操步骤，建议你都跟着操作一遍，否则很难会有特别深刻的认识。

话不多说，现在，我们就先来搭建源码环境吧。

## 环境准备

在阅读 Kafka 源码之前，我们要先做一些必要的准备工作。这涉及到一些工具软件的安装，比如 Java、Gradle、Scala、IDE、Git，等等。

如果你是在 Linux 或 Mac 系统下搭建环境，你需要安装 Java、IDE 和 Git；如果你使用的是 Windows，那么你需要全部安装它们。

咱们这个课程统一使用下面的版本进行源码讲解。

- Oracle Java 8：我们使用的是 Oracle 的 JDK 及 Hotspot JVM。如果你青睐于其他厂商或开源的 Java 版本（比如 OpenJDK），你可以选择安装不同厂商的 JVM 版本。
- Gradle 6.3：我在这门课里带你阅读的 Kafka 源码是社区的 Trunk 分支。Trunk 分支目前演进到了 2.5 版本，已经支持 Gradle 6.x 版本。你最好安装 Gradle 6.3 或更高版本。
- Scala 2.13：社区 Trunk 分支编译当前支持两个 Scala 版本，分别是 2.12 和 2.13。默认使用 2.13 进行编译，因此我推荐你安装 Scala 2.13 版本。
- IDEA + Scala 插件：这门课使用 IDEA 作为 IDE 来阅读和配置源码。我对 Eclipse 充满敬意，只是我个人比较习惯使用 IDEA。另外，你需要为 IDEA 安装 Scala 插件，这样可以方便你阅读 Scala 源码。
- Git：安装 Git 主要是为了管理 Kafka 源码版本。如果你要成为一名社区代码贡献者，Git 管理工具是必不可少的。

<!-- -->

<!-- [[[read_end]]] -->

## 构建Kafka工程

等你准备好以上这些之后，我们就可以来构建 Kafka 工程了。

首先，我们下载 Kafka 源代码。方法很简单，找一个干净的源码路径，然后执行下列命令去下载社区的 Trunk 代码即可：

```
$ git clone https://github.com/apache/kafka.git
```

在漫长的等待之后，你的路径上会新增一个名为 kafka 的子目录，它就是 Kafka 项目的根目录。如果在你的环境中，上面这条命令无法执行的话，你可以在浏览器中输入[https://codeload.github.com/apache/kafka/zip/trunk](<https://codeload.github.com/apache/kafka/zip/trunk>)下载源码 ZIP 包并解压，只是这样你就失去了 Git 管理，你要手动链接到远程仓库，具体方法可以参考这篇[Git文档](<https://help.github.com/articles/fork-a-repo/>)。

下载完成后，你要进入工程所在的路径下，也就是进入到名为 kafka 的路径下，然后执行相应的命令来构建 Kafka 工程。

如果你是在 Mac 或 Linux 平台上搭建环境，那么直接运行下列命令构建即可：

```
$ ./gradlew build
```

该命令首先下载 Gradle Wrapper 所需的 jar 文件，然后对 Kafka 工程进行构建。需要注意的是，在执行这条命令时，你很可能会遇到下面的这个异常：

```
Failed to connect to raw.githubusercontent.com port 443: Connection refused
```

如果碰到了这个异常，你也不用惊慌，你可以去这个[官网链接](<https://raw.githubusercontent.com/gradle/gradle/v6.3.0/gradle/wrapper/gradle-wrapper.jar>)或者是我提供的[链接](<https://pan.baidu.com/s/1tuVHunoTwHfbtoqMvoTNoQ>)（提取码：ntvd）直接下载 Wrapper 所需的 Jar 包，手动把这个 Jar 文件拷贝到 kafka 路径下的 gradle/wrapper 子目录下，然后重新执行 gradlew build 命令去构建工程。

我想提醒你的是，官网链接包含的版本号是 v6.3.0，但是该版本后续可能会变化，因此，你最好先打开 gradlew 文件，去看一下社区使用的是哪个版本的 Gradle。**一旦你发现版本不再是v6.3.0了，那就不要再使用我提供的链接了。这个时候，你需要直接去官网下载对应版本的Jar包**。

举个例子，假设 gradlew 文件中使用的 Gradle 版本变更为 v6.4.0，那么你需要把官网链接 URL 中的版本号修改为 v6.4.0，然后去下载这个版本的 Wrapper Jar 包。

如果你是在 Windows 平台上构建，那你就不能使用 Gradle Wrapper 了，因为 Kafka 没有提供 Windows 平台上可运行的 Wrapper Bat 文件。这个时候，你只能使用你自己的环境中自行安装的 Gradle。具体命令是：

```
kafka> gradle.bat build
```

无论是 gradle.bat build 命令，还是 gradlew build 命令，首次运行时都要花费相当长的时间去下载必要的 Jar 包，你要耐心地等待。

下面，我用一张图给你展示下 Kafka 工程的各个目录以及文件：

![](<https://static001.geekbang.org/resource/image/a2/f7/a2ef664cd8d5494f55919643df1305f7.png?wh=940*770>)

这里我再简单介绍一些主要的组件路径。

- **bin目录**：保存 Kafka 工具行脚本，我们熟知的 kafka-server-start 和 kafka-console-producer 等脚本都存放在这里。
- **clients目录**：保存 Kafka 客户端代码，比如生产者和消费者的代码都在该目录下。
- **config目录**：保存 Kafka 的配置文件，其中比较重要的配置文件是 server.properties。
- **connect目录**：保存 Connect 组件的源代码。我在开篇词里提到过，Kafka Connect 组件是用来实现 Kafka 与外部系统之间的实时数据传输的。
- **core目录**：保存 Broker 端代码。Kafka 服务器端代码全部保存在该目录下。
- **streams目录**：保存 Streams 组件的源代码。Kafka Streams 是实现 Kafka 实时流处理的组件。

<!-- -->

其他的目录要么不太重要，要么和配置相关，这里我就不展开讲了。

除了上面的 gradlew build 命令之外，我再介绍一些常用的构建命令，帮助你调试 Kafka 工程。

我们先看一下测试相关的命令。Kafka 源代码分为 4 大部分：Broker 端代码、Clients 端代码、Connect 端代码和 Streams 端代码。如果你想要测试这 4 个部分的代码，可以分别运行以下 4 条命令：

```
$ ./gradlew core:test
$ ./gradlew clients:test
$ ./gradlew connect:[submodule]:test
$ ./gradlew streams:test
```

你可能注意到了，在这 4 条命令中，Connect 组件的测试方法不太一样。这是因为 Connect 工程下细分了多个子模块，比如 api、runtime 等，所以，你需要显式地指定要测试的子模块名，才能进行测试。

如果你要单独对某一个具体的测试用例进行测试，比如单独测试 Broker 端 core 包的 LogTest 类，可以用下面的命令：

```
$ ./gradlew core:test --tests kafka.log.LogTest
```

另外，如果你要构建整个 Kafka 工程并打包出一个可运行的二进制环境，就需要运行下面的命令：

```
$ ./gradlew clean releaseTarGz
```

成功运行后，core、clients 和 streams 目录下就会分别生成对应的二进制发布包，它们分别是：

- **kafka-2.12-2.5.0-SNAPSHOT.tgz**。它是 Kafka 的 Broker 端发布包，把该文件解压之后就是标准的 Kafka 运行环境。该文件位于 core 路径的/build/distributions 目录。
- **kafka-clients-2.5.0-SNAPSHOT.jar**。该 Jar 包是 Clients 端代码编译打包之后的二进制发布包。该文件位于 clients 目录下的/build/libs 目录。
- **kafka-streams-2.5.0-SNAPSHOT.jar**。该 Jar 包是 Streams 端代码编译打包之后的二进制发布包。该文件位于 streams 目录下的/build/libs 目录。

<!-- -->

## 搭建源码阅读环境

刚刚我介绍了如何使用 Gradle 工具来构建 Kafka 项目工程，现在我来带你看一下如何利用 IDEA 搭建 Kafka 源码阅读环境。实际上，整个过程非常简单。我们打开 IDEA，点击“文件”，随后点击“打开”，选择上一步中的 Kafka 文件路径即可。

项目工程被导入之后，IDEA 会对项目进行自动构建，等构建完成之后，你可以找到 core 目录源码下的 Kafka.scala 文件。打开它，然后右键点击 Kafka，你应该就能看到这样的输出结果了：

![](<https://static001.geekbang.org/resource/image/ce/d2/ce0a63e7627c641da471b48a62860ad2.png?wh=933*372>)

这就是无参执行 Kafka 主文件的运行结果。通过这段输出，我们能够学会启动 Broker 所必需的参数，即指定 server.properties 文件的地址。这也是启动 Kafka Broker 的标准命令。

在开篇词中我也说了，这个课程会聚焦于讲解 Kafka Broker 端源代码。因此，在正式学习这部分源码之前，我先给你简单介绍一下 Broker 端源码的组织架构。下图展示了 Kafka core 包的代码架构：

![](<https://static001.geekbang.org/resource/image/df/b2/dfdd73cc95ecc5390ebeb73c324437b2.png?wh=940*844>)

我来给你解释几个比较关键的代码包。

- controller 包：保存了 Kafka 控制器（Controller）代码，而控制器组件是 Kafka 的核心组件，后面我们会针对这个包的代码进行详细分析。
- coordinator 包：保存了**消费者端的GroupCoordinator代码**和**用于事务的TransactionCoordinator代码**。对 coordinator 包进行分析，特别是对消费者端的 GroupCoordinator 代码进行分析，是我们弄明白 Broker 端协调者组件设计原理的关键。
- log 包：保存了 Kafka 最核心的日志结构代码，包括日志、日志段、索引文件等，后面会有详细介绍。另外，该包下还封装了 Log Compaction 的实现机制，是非常重要的源码包。
- network 包：封装了 Kafka 服务器端网络层的代码，特别是 SocketServer.scala 这个文件，是 Kafka 实现 Reactor 模式的具体操作类，非常值得一读。
- server 包：顾名思义，它是 Kafka 的服务器端主代码，里面的类非常多，很多关键的 Kafka 组件都存放在这里，比如后面要讲到的状态机、Purgatory 延时机制等。

<!-- -->

在后续的课程中，我会挑选 Kafka 最主要的代码类进行详细分析，帮助你深入了解 Kafka Broker 端重要组件的实现原理。

另外，虽然这门课不会涵盖测试用例的代码分析，但在我看来，**弄懂测试用例是帮助你快速了解Kafka组件的最有效的捷径之一**。如果时间允许的话，我建议你多读一读 Kafka 各个组件下的测试用例，它们通常都位于代码包的 src/test 目录下。拿 Kafka 日志源码 Log 来说，它对应的 LogTest.scala 测试文件就写得非常完备，里面多达几十个测试用例，涵盖了 Log 的方方面面，你一定要读一下。

## Scala 语言热身

因为 Broker 端的源码完全是基于 Scala 的，所以在开始阅读这部分源码之前，我还想花一点时间快速介绍一下 Scala 语言的语法特点。我先拿几个真实的 Kafka 源码片段来帮你热热身。

先来看第一个：

```
def sizeInBytes(segments: Iterable[LogSegment]): Long =
    segments.map(_.size.toLong).sum
```

这是一个典型的 Scala 方法，方法名是 sizeInBytes。它接收一组 LogSegment 对象，返回一个长整型。LogSegment 对象就是我们后面要谈到的日志段。你在 Kafka 分区目录下看到的每一个.log 文件本质上就是一个 LogSegment。从名字上来看，这个方法计算的是这组 LogSegment 的总字节数。具体方法是遍历每个输入 LogSegment，调用其 size 方法并将其累加求和之后返回。

再来看一个：

```
val firstOffset: Option[Long] = ......

def numMessages: Long = {
    firstOffset match {
      case Some(firstOffsetVal) if (firstOffsetVal >= 0 && lastOffset >= 0) => (lastOffset - firstOffsetVal + 1)
      case _ => 0
    }
  }
```

该方法是 LogAppendInfo 对象的一个方法，统计的是 Broker 端一次性批量写入的消息数。这里你需要重点关注 **match** 和 **case** 这两个关键字，你可以近似地认为它们等同于 Java 中的 switch，但它们的功能要强大得多。该方法统计写入消息数的逻辑是：如果 firstOffsetVal 和 lastOffset 值都大于 0，则写入消息数等于两者的差值+1；如果不存在 firstOffsetVal，则无法统计写入消息数，简单返回 0 即可。

倘若对你而言，弄懂上面这两段代码有些吃力，我建议你去快速地学习一下 Scala 语言。重点学什么呢？我建议你重点学习下 Scala 中对于**集合的遍历语法**，以及**基于match的模式匹配用法**。

另外，由于 Scala 具有的函数式编程风格，你至少**要理解Java中Lambda表达式的含义**，这会在很大程度上帮你扫清阅读障碍。

相反地，如果上面的代码对你来讲很容易理解，那么，读懂 Broker 端 80%的源码应该没有什么问题。你可能还会关心，剩下的那晦涩难懂的 20%源码怎么办呢？其实没关系，你可以等慢慢精通了 Scala 语言之后再进行阅读，它们不会对你熟练掌握核心源码造成影响的。另外，后面涉及到比较难的 Scala 语法特性时，我还会再具体给你解释的，所以，还是那句话，你完全不用担心语言的问题！

## 总结

今天是我们开启 Kafka 源码分析的“热身课”，我给出了构建 Kafka 工程以及搭建 Kafka 源码阅读环境的具体方法。我建议你对照上面的内容完整地走一遍流程，亲身体会一下 Kafka 工程的构建与源码工程的导入。毕竟，这些都是后面阅读具体 Kafka 代码的前提条件。

最后我想再强调一下，阅读任何一个大型项目的源码都不是一件容易的事情，我希望你在任何时候都不要轻言放弃。很多时候，碰到读不懂的代码你就多读几遍，也许稍后就会有醍醐灌顶的感觉。

## 课后讨论

熟悉 Kafka 的话，你一定听说过 kafka-console-producer.sh 脚本。我前面提到过，该脚本位于工程的 bin 目录下，你能找到它对应的 Java 类是哪个文件吗？这个搜索过程能够给你一些寻找 Kafka 所需类文件的好思路，你不妨去试试看。

欢迎你在留言区畅所欲言，跟我交流讨论，也欢迎你把文章分享给你的朋友。

