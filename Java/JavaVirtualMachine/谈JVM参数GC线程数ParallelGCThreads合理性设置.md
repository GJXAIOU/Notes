# 谈JVM参数GC线程数ParallelGCThreads合理性设置

# 1. ParallelGCThreads参数含义

在讲这个参数之前，先谈谈JVM垃圾回收(GC)算法的两个优化标的：吞吐量和停顿时长。JVM会使用特定的GC收集线程，当GC开始的时候，GC线程会和业务线程抢占CPU时间，吞吐量定义为CPU用于业务线程的时间与CPU总消耗时间的比值。为了承接更大的流量，吞吐量越大越好。

为了安全的垃圾回收，在GC或者GC某个阶段，所有业务线程都会被暂停，也就是STW（Stop The World)，STW持续时间就是停顿时长，停顿时长影响响应速度，因此越小越好。

这两个优化目标是有冲突的，在一定范围内，参与GC的线程数越多，停顿时长越小，但吞吐量也越小。生产实践中，需要根据业务特点设置一个合理的GC线程数，取得吞吐量和停顿时长的平衡。

目前广泛使用的GC算法，包括PS MarkSweep/PS Scavenge, ConcurrentMarkSweep/ParNew, G1等，都可以通过ParallelGCThreads参数来指定JVM在并行GC时参与垃圾收集的线程数。该值设置过小，GC暂停时间变长影响RT，设置过大则影响吞吐量，从而导致CPU过高。

# 2. ParallelGCThreads参数设置

GC并发线程数可以通过JVM启动参数: -XX:ParallelGCThreads=<N>来指定。在未明确指定的情况下，JVM会根据逻辑核数ncpus，采用以下公式来计算默认值：

◦当ncpus小于8时，ParallelGCThreads = ncpus

◦否则 ParallelGCThreads = 8 + (ncpus - 8 ) ( 5/8 ) 

一般来说，在无特殊要求下，ParallelGCThreads参数使用默认值就可以了。**但是在JRE版本1.8.0_131之前，JVM无法感知Docker的CPU限制，会使用宿主机的逻辑核数计算默认值。**比如部署在128核物理机上的容器，JVM中默认ParallelGCThreads为83，远超过了容器的核数。过多的GC线程数抢占了业务线程的CPU时间，加上线程切换的开销，较大的降低了吞吐量。因此JRE 1.8.0_131之前的版本，未明确指定ParallelGCThreads会有较大的风险。

# 3. ParallelGCThreads参数实验

创建 8C12G 容器，宿主机是128C。模拟线上真实流量，采用相同QPS，观察及对比JVM YoungGC，JVM CPU，容器CPU等监控数据。场景如下：

◦场景1: JVM ParallelGCThreads 默认值，QPS = 420，持续5分钟，CPU恒定在70%

◦场景2: JVM ParallelGCThreads=8，QPS = 420，持续5分钟，CPU恒定在65%

◦场景3: JVM ParallelGCThreads 默认值，QPS瞬时发压到420，**前1min CPU持续100%**

◦场景4: JVM ParallelGCThreads=8，QPS瞬时发压到420，**前2s CPU持续100%，后面回落**

﻿

从监控数据来看，各场景下CPU差距较明显，特别是场景3和场景4的对比。场景3由于GC线程过多，CPU持续100%时长达1分钟。可以得出以下两个结论：

1.修改 ParallelGCThreads = 8后，同等QPS情况下，CPU会降低5%左右

2.修改 ParallelGCThreads = 8后，**瞬间发压且CPU打满情况下，CPU恢复较快**

1.

﻿

![img](%E8%B0%88JVM%E5%8F%82%E6%95%B0GC%E7%BA%BF%E7%A8%8B%E6%95%B0ParallelGCThreads%E5%90%88%E7%90%86%E6%80%A7%E8%AE%BE%E7%BD%AE.resource/link-20230516202033071.png)

﻿

﻿

![img](%E8%B0%88JVM%E5%8F%82%E6%95%B0GC%E7%BA%BF%E7%A8%8B%E6%95%B0ParallelGCThreads%E5%90%88%E7%90%86%E6%80%A7%E8%AE%BE%E7%BD%AE.resource/link-20230516202033155.png)

﻿

﻿

图1: 容器CPU对比图：场景3(上)和场景4(下)

﻿

![img](%E8%B0%88JVM%E5%8F%82%E6%95%B0GC%E7%BA%BF%E7%A8%8B%E6%95%B0ParallelGCThreads%E5%90%88%E7%90%86%E6%80%A7%E8%AE%BE%E7%BD%AE.resource/link-20230516202033080.png)

﻿﻿



﻿

![img](%E8%B0%88JVM%E5%8F%82%E6%95%B0GC%E7%BA%BF%E7%A8%8B%E6%95%B0ParallelGCThreads%E5%90%88%E7%90%86%E6%80%A7%E8%AE%BE%E7%BD%AE.resource/link-20230516202033068.png)

﻿﻿



图2: JVM Young GC对比图：场景3(上)和场景4(下)

# 4. ParallelGCThreads修改建议

 ParallelGCThreads配置存在风险的应用，修改方式为以下两种方案（任选一种）：

◦升级JRE版本到**1.8.0_131以上**，推荐1.8.0_192

◦在JVM启动参数明确指定 -XX:ParallelGCThreads=<N>，N为下表的推荐值：

|  容器核数  |  2   |  4   |  8   |  16  |  32   |  64   |
| :--------: | :--: | :--: | :--: | :--: | :---: | :---: |
|   推荐值   |  2   |  4   |  8   |  13  |  23   |  43   |
| 建议上下界 | 1~2  | 2~4  | 4~8  | 8~16 | 16~32 | 32~64 |

## 回复

- 建议其实最好是升级JDK8的高版本。因为本文的配置只能解决GC的问题，其实Java很多库都有使用下面的语句获取CPU数量以创建对应的线程数量，在JDK8的低版本，下面的语句都会获取到有问题的CPU数量，也可能导致线程数量有问题。因此还是建议直接在镜像中升级高版本的JDK8最好

```
Runtime.getRuntime().availableProcessors();
```

﻿回复：1.8.131以上可以的。低的话会获取物理机的核数。

张运鹏

设置完ParallelGCThreads后，YGC的时间也会有一定程度的下降