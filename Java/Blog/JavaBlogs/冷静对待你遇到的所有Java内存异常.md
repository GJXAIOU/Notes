# 冷静对待你遇到的所有Java内存异常
[原文地址](https://zazalu.space/2019/09/17/java-memory-error-solution-Theoretically/)

## 被人说烂的Java内存模型

Java 内存模型的相关资料在网上实在是太多了,不管是过时的还是不过时的,网络上充斥的学习资料,比如各类研究 Java 内存模型的博文,也随着 Java 的发展,渐渐失去了其内容的准确性.

要在那么多网络资料中找到对 Java 内存模型最新最全的说法,估计最好的方式只有翻阅 Oracle 的文档了!(字体大小太不舒服了!)

我最近也不停的查阅和总结了不少网上的资料,不过由于类似资料实在太多,所以不打算重复的说明这个被人说烂的 Java 内存模型

## 从各种OOM异常出发来零距离的理解Java内存模型

对于大脑来说, 大脑更喜欢问题, 而不是从陈述开始.

本文会从平时工作中可能会遇到的 OOM 异常出发,来一步步的深入理解我们所熟知的 Java 内存模型,从而哪怕可以更加理解一点这些方面的编程思想和设计精髓, 也是一个不小的进步

## java.lang.StackOverFlowError

### 这个Stack是什么鬼东西

Stack 是个栈, 是一种数据结构, 会占用一块内存空间

### Java在哪些地方会使用Stack来存储数据

1.  最常见的就是虚拟机栈, 它是专门为 java Method 执行服务的一块内存, 每个方法调用都会往这个栈中压入一个栈帧(stackFrame), 由于方法可以互调,迭代,所以使用栈模型来服务 Java Method 是很适合的一种数据结构模型

2.  别忘了还有一个本地方法栈, 它是专门为 java 的底层 native 方法执行服务的一块内存. 然而由于 native 方法都是术语 jdk 内部的测试稳定的程序,所以作为应用 java 开发人员的我们,一般是不可能遇到这个层面抛出的这个异常,同时我也几乎可以判断这种方法是不会直接抛出 java.lang.StackOverFlowError 异常的,所以我们可以缩小我们的关注范围,把抛出这个异常的原因全部指向于虚拟机栈即可

### 这种异常是如何发生的?

我们知道每调用一次 Java Method,就会往虚拟机栈中压入一个栈帧,在方法结束之前都不会出栈. 所以可以直接推理出在一个 java 线程运行过程中,如果同时调用的方法过多(比如递归的调用一个方法),就会出现这个异常

事实上,除了恶性递归或者虚拟机栈可用内存过小的情况下, 也很难触发这种异常, 所以一般来说遇到这种异常几乎是可以直接断定程序中存在恶性递归导致的.

这类问题在实际开发中遇到的并不多, 反而是在做一些算法问题的时候, 由于自己的疏忽从而引发不可预知的恶性递归

### 一个简单的Demo复现这种异常

```java
public class Main {
    public static void main(String[] args) {
        Main.main(null);
    }
}
```


上述代码就会报 StackOverFlowError, 因为 main 方法会被不停的循环执行, 直到超出虚拟机栈能够承受的大小

### 相关JVM参数

-Xss, 正常取值 128K~256K, 如果仍然不够可以进行加大, 这个选项对性能影响比较大，需要严格的测试哦

## java.lang.OutOfMemoryError: Java heap space

这个异常表示, Java 程序运行过程中遭遇了内存超限问题, 根本原因是 Java 的堆(Heap)内存超限

### Java常用的内存空间对应计算机硬件是哪些组件?

1.  寄存器(比如每个 Java 线程独享程序计数器(Program Counter Register))
2.  RAM(也就是我们常说的内存,java 中的虚拟机栈,堆内存都用的这块)

### 什么是Java的堆内存(Heap)

这就涉及了 Java 的运行时内存模型了~

我就简单来说下吧~

一个 JVM 进程运行后, 会有一个主线程去运行我们写的 Java 程序, 那么每一个这种线程都拥有两大块内存空间

*   线程共享内存空间
    *   堆(Heap, 所有 java 的对象实例和数组,jdk8 后还存放了字符串常量池和类静态变量)
    *   方法区(存放类元数据,符号引用,静态常量,jdk8 后 HotSpot 将其从永久代移动到了 Metaspace)
*   线程独享内存空间
    *   虚拟机栈(为 Java 方法提供的一块内存空间,内部有栈帧组成)
    *   本地方法栈(为 Java 的 native 方法)
    *   程序计数器(PC 寄存器,记录执行行号)

所以 Java 的堆内存就是 JVM 中设定的一块专门存储所有 java 的对象实例和数组,jdk8 后甚至包括字符串常量池和类静态变量的内存区域

### 这种异常是如何发生的?

如果是 1.7 以前, Java 堆溢出的问题根源是简单的, 就是运行时存在的对象实例和数组太多了!

但是在 1.8 后, 由于还存放了字符串常量, 所以出现异常还有一种可能就是 interned Strings 过多导致的哦!

### 最小复现Demo

执行前最好先修改下 JVM 参数,防止等待时间过长
JVM 参数:
-Xms20m
-Xmx20m
-XX:MetaspaceSize=10m
-XX:MaxMetaspaceSize=10m
-XX:-UseGCOverheadLimit

JVM 参数说明: 限制堆大小 20M,方便快速报错! 由于我用的是 jdk8,所以限制了元空间的大小为 10m,说实话在这个情况下没啥用哈哈哈哈哈哈哈(就是觉得加上去舒服才加的,不信我说的你可以自己 google)!最后一个参数-XX:-UseGCOverheadLimit 这个有必要加一下. 因为我的 demo 程序属于那种恶意的程序,所以一次 GC 几乎没办法清理任何对象实例,因为他们都在被占用着! 所以必须使用这个参数来防止 GC 检测出我的这种恶意程序,从而正常的提示堆溢出的错误而不是 GC Overhead limit exceeded 错误(这个错误会在后面细讲)

1.  普通的对象实例爆掉堆内存
```java
public static void main(String[] args) {
        List<Object> list = new ArrayList();
        int i = 0;
        while(true){
            list.add(new Object());
        }
    }
```


1.  interned Strings 过多爆掉堆内存(有待考证此代码的准确性,请不要盲目相信,要有自己的想法)
```java
public static void main(String[] args) {
        List<String> list = new ArrayList();
        int i = 0;
        while(true){
            list.add(String.valueOf(i++).intern());
        }
    }
```

代码说明: 这串代码会每次生成一个新的 interned String, 也就是数字递增对应的 String 表示, 所以最终爆掉内存, 证明了是 interned Strings 爆掉了内存, 相同的代码在 jkd1.7 以前是不会报堆内存溢出的, 请注意

### 相关JVM参数

-Xms : 初始堆大小
-Xmx : 最大堆大小

### 如何处理?

查看 jvm 快照,分析占用内存大的对象是哪些, 然后定位到代码位置, 最后进行优化

我一般使用 visualVM 来查看这类问题

## java.lang.OutOfMemoryError: GC Overhead limit exceeded

这个异常表示您的 Java 程序在运行的时候, 98%的时间都在执行 GC 回收, 但是每次回收只回收不到 2%的空间!

换句话说,其实这个异常往往是抛出 java.lang.OutOfMemoryError: Java heap space 异常的前兆! 因为 Java 程序每次都 GC 回收只能回收一点点内存空间,而你的程序却仍然在不停的产生新的对象实例, 这无疑导致了两种可能结果:

1.  不停的进行 GC
2.  直接超出的堆内存大小

这个问题还有一些细节需要我们去掌握,我们先从下面的例子来看吧

### 最小复现Demo

```java
public static void main(String args[]) throws Exception {
        Map map = System.getProperties();
        Random r = new Random();
        while (true) {
            map.put(r.nextInt(), "value");
        }
    }
```

代码说明: 这段代码不停的往 map 中加入新的 key-value,导致 map 大小不断变大! 当到达堆内存顶点的时候,GC 发生, 但是清理完毕后,JVM 发现清理前后的堆内存大小改变很小,不到 2%; 这时候程序继续运行,继续往 map 中加数据!GC 又发生了!又只清理不到 2%! 如此不停的循环, 最后 JVM 得出了一个判断! 你的 Java 程序在占用 CPU 进行运算的时间里,98%的时间都特么的在垃圾回收,而每次 GC 居然只能回收堆内存的 2%空间, 这肯定是代码存在问题,于是抛出了这个异常. 如果这个时候,你断定不是自己的代码问题, 使用 JVM 参数-XX:-UseGCOverheadLimit 来关闭这种检查! 然后你就会发现你的程序抛出了堆溢出异常! 为什么呢? 因为堆内存不断的被占满,最终导致最后一次加入新的 int 的时候, 堆内存空间直接不足了!

### 这个异常一般如何处理

和堆溢出的解决方式一致

### 相关JVM参数

-XX:-UseGCOverheadLimit

## java.lang.OutOfMemoryError: Permgen space (jdk8已经不会出现此异常,请注意)

只存在于 jdk1.8 以前的 java 程序中! 这个异常表示,永久代大小不够!

### 什么是Permgen

是 HotSpot 在 jdk1.8 以前存在的一个区域,用于实现方法区

### 什么时候会产生这个错误以及如何解决

由于是实现方法区的地方, 所以肯定是类元信息或者常量（jdk1.7 后部分常量已经挪到堆中），静态常量和 JIT 即时编译器编译后的代码等数据太多导致大小不够

乍一看也许你会头晕! 不过没关系, 根据我两年的开发经验, 我碰到过的唯一一次 Permgen space 问题是因为 SpringIoC 容器一口气加载了过多的 Bean 导致的!

所以正常来说, 直接扩大这个区域的大小即可!

比如使用如下 JVM 参数扩大:
-XX:MaxNewSize=xxxm -XX:MaxPermSize=xxxm

### 最小复现Demo

运行要求: jdk 版本 <= 1.6
```java
import javassist.ClassPool;

public class MicroGenerator {
  public static void main(String[] args) throws Exception {
    for (int i = 0; i < 100_000_000; i++) {
      generate("eu.plumbr.demo.Generated" + i);
    }
  }

  public static Class generate(String name) throws Exception {
    ClassPool pool = ClassPool.getDefault();
    return pool.makeClass(name).toClass();
  }
}
```


借助了 javassist 来不停的加载新的 class,直至爆掉永久代区域

### 相关JVM参数

-XX:PermSize=xxxm
-XX:MaxPermSize=xxxm

## java.lang.OutOfMemoryError: Metaspace (since jdk8 才有可能抛出的错误)

这个异常表示: Metaspace 的空间不足导致 OOM 异常发生

### 什么是Metaspace

有些不太专注 JVM 知识的小伙伴可能对 Metaspace 是陌生的, 因为这玩意是 jdk8 开始才正式登场的一块内存区域. 它专门用于替代原来的永久代, 且存在于本地内存中, 所以它的最大内存理论就是你电脑的最大内存. 和永久代不一样的是, 它可以进行自我扩容, 直到达到规定的 MaxMetaspaceSize 或者到达本机的最大可用内存为止.

Metaspace 接替了永久代的任务, 方法区的内容全部转移到此处(除了字符串常量池被挪到了堆中)

不过相比于永久代, Metaspace 进行 GC 的时候, 稍微改变了一点规则, Metaspace 中类元数据是否需要回收是根据类加载器死活来来决定的, 这不同于永久代的, 只要类引用消失就会被回收. 这种规则会产生一些问题:

1.  [https://blog.csdn.net/xyghehehehe/article/details/78820135#commentsedit](https://blog.csdn.net/xyghehehehe/article/details/78820135#commentsedit)
2.  [https://zhuanlan.zhihu.com/p/25634935](https://zhuanlan.zhihu.com/p/25634935)

所以在 jdk8 后使用反射,动态代理等会生成 class 对象的方法, 一定要小心 MetaSpace 是否会对其进行回收, 如果不会, 则需要进行相应的优化处理

### 为什么要移除永久代

1.  方法区大小难以设定，容易发生内存溢出。永久代会存放 Class 的相关信息，一般这些信息在编译期间就能确定大小。但是如果是在一些需要动态生成大量 Class 的应用中，如：Spring 的动态代理、大量的 JSP 页面或动态生成 JSP 页面等，由于方法区的大小在一开始就要分配好，因此就能难确定大小，容易出现内存溢出

2.  GC 复杂且效率低。方法区存储了类的元数据信息和各种常量，它的内存回收目标理应当是对这些类型的卸载和常量的回收。但由于这些数据被类的实例引用，卸载条件变得复杂且严格，回收不当会导致堆中的类实例失去元数据信息和常量信息。因此，回收方法区内存不是一件简单高效的事情。

3.  促进 HotSpot JVM 与 JRockit VM 的融合。JRockit 没有方法区，移除永久代可以促进 HotSpot JVM 与 JRockit VM 的融合。

### 最小复现Demo

```java
/**
 -XX:MetaspaceSize=8m
 -XX:MaxMetaspaceSize=8m
 */
public class MetaSpaceOOMTest {

    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                    return proxy.invokeSuper(obj, args);
                }
            });
            //无限创建动态代理，生成Class对象
            enhancer.create();
        }
    }

    static class OOMObject {

    }
}
```


### 如何解决这类异常

1.  增大 MetaSpace 的最大空间大小
2.  类似检查永久代异常一样的处理方式, 检查 dump 文件, 查看哪些类加载存在异常

### 相关JVM参数

-XX:MetaspaceSize=8m
-XX:MaxMetaspaceSize=8m

## java.lang.OutOfMemoryError: Unable to create new native thread

这个异常表示,JVM 无法再创建新的线程了!JVM 能够创建的线程数是有限制的,

### 复现demo
```java
public class TestNativeOutOfMemoryError {  
  
    public static void main(String[] args) {  
  
        for (int i = 0;; i++) {  
            System.out.println("i = " + i);  
            new Thread(new HoldThread()).start();  
        }  
    }  
  
}  
  
class HoldThread extends Thread {  
    CountDownLatch cdl = new CountDownLatch(1);  
  
    public HoldThread() {  
        this.setDaemon(true);  
    }  
  
    public void run() {  
        try {  
            cdl.await();  
        } catch (InterruptedException e) {  
        }  
    }  
}
```



### 解决方案

1.  去用线程池!

2.  检查代码是否存在 bug 在不停的生成新线程!

3.  如果确实需要那么多线程,那就修改 OS 和 JVM 的参数设置,并且加大你的硬件内存容量!

## java.lang.OutOfMemoryError: request size bytes for reason

如果你看到了这个异常, 说明你的 OS 内存不够用了, JVM 想本地操作系统申请内存被拒绝, 导致 JVM 进程无法继续运行! 发生这个问题的原因一般是你的 Java 程序需要的内存容量超过了操作系统可提供给 JVM 的最大内存容量, 连 swap 内存都没了

## java.lang.OutOfMemoryError: Requested array size exceeds VM

当你正准备创建一个超过虚拟机允许的大小的数组时，这条错误就会出现在你眼前!

## 尾

本文对 java 常见的 OOM 异常做了总结说明,同时对于涉及的 Java 内存模型进行了说明,希望可以在日后遇到类似问题的时候可以沉着冷静,不慌不忙的来排查问题

参考:
[https://medium.com/platform-engineer/understanding-java-memory-model-1d0863f6d973](https://medium.com/platform-engineer/understanding-java-memory-model-1d0863f6d973)
[https://juejin.im/post/5ca02d046fb9a05e6a086cb7](https://juejin.im/post/5ca02d046fb9a05e6a086cb7)
[https://zhuanlan.zhihu.com/p/25634935](https://zhuanlan.zhihu.com/p/25634935)
[https://www.zhihu.com/question/39990490/answer/369690291](https://www.zhihu.com/question/39990490/answer/369690291)