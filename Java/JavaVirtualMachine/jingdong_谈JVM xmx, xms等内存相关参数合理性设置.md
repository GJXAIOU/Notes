# 谈JVM xmx, xms等内存相关参数合理性设置

[上一篇文章](http://xingyun.jd.com/shendeng/article/detail/7217?jdme_router=jdme%3A%2F%2Fweb%2F202206081297%3Furl%3Dhttps%3A%2F%2Fshendengh5.jd.com%2FarticleDetail%3Fid%3D7217)说到JVM垃圾回收算法的两个优化标的：吞吐量和停顿时长，并提到这两个优化目标是有冲突的。那么有没有可能提高吞吐量而不影响停顿时长，甚至缩短停顿时长呢？答案是有可能的，提高内存占用（Memory Footprint）就有可能同时优化这两个标的，这篇文章就来聊聊内存相关内容。

内存占用一般指应用运行需要的所有内存，包括堆内内存（On-heap Memory)和堆外内存（Off-heap Memory）

## 一、堆内内存

堆内内存是分配给JVM的部分内存，用来存放所有Java Class对象实例和数组，JVM GC操作的就是这部分内容。我们先来回顾一下堆内内存的模型：



![img](jingdong_%E8%B0%88JVM%20xmx,%20xms%E7%AD%89%E5%86%85%E5%AD%98%E7%9B%B8%E5%85%B3%E5%8F%82%E6%95%B0%E5%90%88%E7%90%86%E6%80%A7%E8%AE%BE%E7%BD%AE.resource/link.png)

﻿﻿图1. 堆内内存

堆内内存包括年轻代（浅绿色），老年代（浅蓝色），在 JDK7 或者更老的版本，图中右边还有个永久代（永久代在逻辑上位于 JVM 的堆区，但又被称为非堆内存，在 JDK8 中被元空间取代）。JVM 有动态调整内存策略，通过-Xms，-Xmx 指定堆内内存动态调整的上下限。 在 JVM 初始化时实际只分配部分内存，可通过 `-XX:InitialHeapSize` 指定初始堆内存大小，未被分配的空间为图中 virtual 部分。年轻代和老年代在每次 GC 的时候都有可能调整大小，以保证存活对象占用百分比在特定阈值范围内，直到达到 Xms 指定的下限或 Xms 指定的上限。（阈值范围通过 `-XX:MinHeapFreeRatio`, `XX:MaxHeapFreeRatio` 指定，默认值分别为40， 70）。

GC 调优中还有个的重要参数是老年代和年轻代的比例，通过 `-XX:NewRatio` 设定，与此相关的还有 `-XX:MaxNewSize` 和 `-XX:NewSize`，分别设定年轻代大小的上下限，-Xmn则直接指定年轻代的大小。

### （一）参数默认值

- `-Xmx`: Xmx 的默认值比较复杂，官方文档上有时候写的是1GB，但实际值跟JRE版本、JVM 模式（client, server）和系统（平台类型，32位，64位）等都有关。经过查阅源码和实验，确定在生产环境下（server模式，64位Centos，JRE 8），Xmx 的默认值可以采用以下规则计算：
    - 容器内存小于等于2G：默认值为容器内存的1/2，最小16MB， 最大512MB。
    - 容器内存大于2G：默认值为容器内存的1/4, 最大可到达32G。

- `-Xms`: 默认值为容器内存的1/64, 最小8MB，如果明确指定了Xmx并且小于容器内存1/64, Xms默认值为Xmx指定的值。

- `-NewRatio`: 默认2，即年轻代和年老代的比例为1:2， 年轻代大小为堆内内存的1/3。

**NOTE：在JRE版本1.8.0_131之前，JVM无法感知Docker的资源限制，Xmx, Xms未明确指定时，会使用宿主机的内存计算默认值。**

### （二）最佳实践

由于每次Eden区满就会触发YGC，而每次YGC的时候，晋升到老年代的对象大小超过老年代剩余空间的时候，就会触发FGC。所以基本来说，GC频率和堆内内存大小是成反比的，也就是说堆内内存越大，吞吐量越大。

如果Xmx设置过小，不仅浪费了容器资源，在大流量下会频繁GC，导致一系列问题，包括吞吐量降低，响应变长，CPU升高，java.lang.OutOfMemoryError异常等。当然Xmx也不建议设置过大，否则会导致进程hang住或者使用容器Swap。所以合理设置Xmx非常重要，特别是对于1.8.0_131之前的版本，一定要明确指定Xmx。**推荐设置为容器内存的50%，不能超过容器内存的80%。**

JVM的动态内存策略不太适合服务使用，因为每次GC需要计算Heap是否需要伸缩，内存抖动需要向系统申请或释放内存，特别是在服务重启的预热阶段，内存抖动会比较频繁。另外，容器中如果有其他进程还在消费内存，JVM内存抖动时可能申请内存失败，导致OOM。**因此建议服务模式下，将Xms设置Xmx一样的值。**

NewRatio建议在2~3之间，最优选择取决于对象的生命周期分布。一般先确定老年代的空间（足够放下所有live data，并适当增加10%~20%)，其余是年轻代，年轻代大小一定要小于老年代。

另外，以上建议都是基于一个容器部署一个JVM实例的使用情况。有个别需求，需要在一个容器内启用多个JVM，或者包含其他语言的，研发需要按业务需求在推荐值范围内分配JVM的Xmx。

## 二、堆外内存

和堆内内存对应的就是堆外内存。堆外内存包括很多部分，比如Code Cache， Memory Pool，Stack Memory，Direct Byte Buffers, Metaspace等等，其中我们需要重点关注的是Direct Byte Buffers和Metaspace。

### （一）Direct Byte Buffers

Direct Byte Buffers 是系统原生内存，不位于 JVM 里，狭义上的堆外内存就是指的Direct Byte Buffers。为什么要使用系统原生内存呢? 为了更高效的进行 Socket I/O 或文件读写等内核态资源操作，会使用 JNI（Java 原生接口），此时操作的内存需要是连续和确定的。而 Heap 中的内存不能保证连续，且 GC 也可能导致对象随时移动。因此涉及 Output 操作时，不直接使用 Heap 上的数据，需要先从 Heap 上拷贝到原生内存，Input操作则相反。因此为了避免多余的拷贝，提高 I/O 效率，不少第三方包和框架使用 Direct Byte Buffers，比 Netty。

Direct Byte Buffers 虽然有上述优点，但使用起来也有一定风险。常见的 Direct Byte Buffers 使用方法是用 `java.nio.DirectByteBuffer` 的 `unsafe.allocateMemory` 方法来创建，DirectByteBuffer 对象只保存了系统分配的原生内存的大小和启始位置，这些原生内存的释放需要等到 DirectByteBuffer 对象被回收。有些特殊的情况下（比如 JVM 一直没有 FGC，设置 `-XX:+DisableExplicitGC` 禁用了System.gc），这部分对象会持续增加，直到堆外内存达到 `-XX:MaxDirectMemorySize` 指定的大小或者耗尽所有的系统内存。

MaxDirectMemorySize 不明确指定的时候，默认值为 0，在代码中实际为`Runtime.getRuntime().maxMemory()`，略小于 -Xmx 指定的值（堆内内存的最大值减去一个 Survivor 区大小）。此默认值有点过大，MaxDirectMemorySize 未设置或设置过大，有可能发生堆外内存泄露，导致进程被系统 Kill。

由于存在一定风险，建议在启动参数里明确指定 `-XX:MaxDirectMemorySize` 的值，并满足下面规则：

- **Xmx * 110% + MaxDirectMemorySize + 系统预留内存 <= 容器内存** 

- Xmx * 110% 中额外的10%是留给其他堆外内存的，是个保守估计，个别业务运行时线程较多，需自行判断，上式中左侧还需加上Xss * 线程数

- 系统预留内存512M到1G，视容器规格而定

- I/O较多的业务适当提高MaxDirectMemorySize比例

### （二）Metaspace

Metaspace（元空间）是JDK8关于方法区新的实现，取代之前的永久代，用来保存类、方法、数据结构等运行时信息和元信息的。很多研发在老版本时可能遇到过java.lang.OutOfMemoryError: PermGen Space，这说明永久代的空间不够用了，可以通过-XX:PermSize，-XX:MaxPermSize来指定永久代的初始大小和最大大小。Metaspace取代永久代，位置由JVM内存变成系统原生内存，也取消默认的最大空间限制。与此有关的参数主要有下面两个：

- -XX:MaxMetaspaceSize 指定元空间的最大空间，默认为容器剩余的所有空间

- -XX:MetaspaceSize 指定元空间首次扩充的大小，默认为20.8M

由于MaxMetaspaceSize未指定时，默认无上限，所以需要特别关注内存泄露的问题，如果程序动态的创建了很多类，或出现过java.lang.OutOfMemoryError:Metaspace，建议明确指定-XX:MaxMetaspaceSize。另外Metaspace实际分配的大小是随着需要逐步扩大的，每次扩大需要一次FGC，-XX:MetaspaceSize默认的值比较小，需要频繁GC扩充到需要的大小。通过下面的日志可以看到Metaspace引起的FGC：

[Full GC (Metadata GC Threshold) ...]

为减少预热影响，可以将-XX:MetaspaceSize，-XX:MaxMetaspaceSize指定成相同的值。另外不少应用由JDK7升级到了JDK8，但是启动参数中仍有-XX:PermSize，-XX:MaxPermSize，**这些参数是不生效的**，建议修改成-XX:MetaspaceSize，-XX:MaxMetaspaceSize。

## 三、应用健康度检查规则

﻿[泰山应用健康度](http://taishan.jd.com/appHealth)现在已支持扫描JVM相关风险，在应用TAB的JVM配置检测项下。主要包括以下检测：

|     检测指标      | 风险等级 |                           巡检规则                           |
| :---------------: | :------: | :----------------------------------------------------------: |
|      JVM版本      |   中危   |                     版本不低于1.8.0_191                      |
|    JVM GC方法     |   中危   |                      所有分组GC方法一致                      |
|        Xmx        |   高危   |           明确指定，并且在容器内存的50%~80%范围内            |
|        Xms        |   中危   |                明确指定，并且等于Xmx指定的值                 |
|     堆外内存      |   中危   |       明确指定，并且 堆内*1.1+堆外+系统预留<=容器内存        |
| ParallelGCThreads |   高危   |        ParallelGCThreads在容器CPU核数的50%~100%范围内        |
|   ConcGCThreads   |   低危   | ConcGCThreads在ParallelGCThreads的20%~50%范围内（限CMS，G1） |
|  CICompilerCount  |   低危   | 指定CICompilerCount在推荐值50%~150%内（限1.8<JRE<1.8.0_131)  |

﻿[上一篇文章](http://xingyun.jd.com/shendeng/article/detail/7217?jdme_router=jdme%3A%2F%2Fweb%2F202206081297%3Furl%3Dhttps%3A%2F%2Fshendengh5.jd.com%2FarticleDetail%3Fid%3D7217)已经说了ParallelGCThreads，这里再补充一下新支持的两个检测，ConcGCThreads，CICompilerCount。

ConcGCThreads一般称为并发标记线程数，为了减少GC的STW的时间，CMS和G1都有并发标记的过程，此时业务线程仍在工作，只是并发标记是CPU密集型任务，业务的吞吐量会下降，RT会变长。ConcGCThreads的默认值不同GC策略略有不同，CMS下是(ParallelGCThreads + 3) / 4 向下取整，G1下是ParallelGCThreads / 4 四舍五入。一般来说采用默认值就可以了，但是还是由于在JRE版本1.8.0_131之前，JVM无法感知Docker的资源限制的问题，ConcGCThreads的默认值会比较大（20左右），对业务会有影响。

CICompilerCount是JIT进行热点编译的线程数，和并发标记线程数一样，热点编译也是CPU密集型任务，默认值为2。在CICompilerCountPerCPU开启的时候（JDK7默认关闭，JDK8默认开启），手动指定CICompilerCount是不会生效的，JVM会使用系统CPU核数进行计算。所以当使用JRE8并且版本小于1.8.0_131，采用默认参数时，CICompilerCount会在20左右，对业务性能影响较大，特别是启动阶段。建议升级Java版本，特殊情况要使用老版本Java 8，请加上-XX:CICompilerCount=[n], 同时不能指定-XX:+CICompilerCountPerCPU ，下表给出了生产环境下常见规格的推荐值。

|          容器CPU核数          |  1   |  2   |  4   |  8   |  16  |
| :---------------------------: | :--: | :--: | :--: | :--: | :--: |
| CICompilerCount手动指定推荐值 |  2   |  2   |  3   |  3   |  8   |

﻿

##  四、修改建议

1） 再次建议升级JRE版本到1.8.0_191及以上； 2） 建议在Shell脚本中，Export JAVA_OPTS环境变量, 至少包含以下几项(**方括号中的值根据文中推荐选取**)：

```
-server -Xms[8192m] -Xmx[8192m] -XX:MaxDirectMemorySize=[4096m]
```

如果特殊原因要使用1.8.0_131以下版本， 则同时需要加上以下参数(**方括号中的值根据文中推荐选取**)：

```
-XX:ParallelGCThreads=[8] -XX:ConcGCThreads=[2] -XX:CICompilerCount=[2]
```

**下面的项建议测试后使用，需自行确定具体大小**（特别是使用JRE8但仍配置-XX:PermSize，-XX:MaxPermSize的应用）：

```
-XX:MaxMetaspaceSize=[256]m -XX:MetaspaceSize=[256]m
```

  环境变量设置如下例子：

```
export JAVA_OPTS="-Djava.library.path=/usr/local/lib -server -Xms4096m -Xmx4096m -XX:MaxMetaspaceSize=512m -XX:MetaspaceSize=512m -XX:MaxDirectMemorySize=2048m -XX:ParallelGCThreads=8 -XX:ConcGCThreads=2 -XX:CICompilerCount=2 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/export/Logs -XX:+UseG1GC [other_options...] -jar jarfile [args...]"
```

另外，如果应用未接入UMP或PFinder， JAVA_OPTS中尽量不要用Shell函数或者变量，否则健康度有可能会提示解析失败。

**NOTE: Java options 的使用应该按照下面的顺序：**

◦执行类： java [-options] class [args...]

◦执行包：java [-options] -jar jarfile [args...] 或 java -jar [-options] jarfile [args...]

即options要放到执行对象之前，部分应用使用了以下顺序：

​    java -jar jarfile [-options] [args...] 或者 java -jar jarfile [args...] [-options]

这些Java options都不会生效。