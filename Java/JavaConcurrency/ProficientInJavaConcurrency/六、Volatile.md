# Volatile

[TOC]

volatile 可以说是 Java 虚拟机提供的最轻量级的同步机制了

## volatile 关键字的作用以及缺点

==volatile 不能保证原子性，为什么能实现原子操作==

- 实现 long/double 类型变量的原子操作

    因为 JVM 中的 long 和 double 都是占 64 位，针对现在的 32 位和 64 位机器，其可能分成低 32 位和高 32 位两个步骤进行分别填充赋值。导致其非原子性。

    ![image-20210320151618677](Volatile.resource/image-20210320151618677.png)

    正确使用示例：

    ```java
    // 使用 volatile 或者 atomicLong/Integer 保证原子性更新数据。
    volatile double a = 1.0;
    AtomicLong b = new AtomicLong(1);
    ```

    关于为什么没有 AtomicFloat 和 AtomicDouble，可以查看[stackoverflow](https://stackoverflow.com/questions/5505460/java-is-there-no-atomicfloat-or-atomicdouble)。

- 防止指令重排序

- 实现变量的可见性

    当使用 volatile 修饰变量时，应用就不会从寄存器中获取该变量的值，而是从内存（高速缓存）中获取。

    > 因为程序在读取一个变量值的时候不会直接从内存中进行读取，而是从 CPU 中的寄存器中读取。当使用 volatile 修饰一个变量的时候，编译器就不会将该变量放置到寄存器中进行存储。对变量的访问和修改都需要访问内存。

### volatile 的缺点

为了实现原子性操作那么每次取数据的时候不会在寄存器上取而是在主内存或者高速缓存上取这将带来性能的损失。

###  使用方式

```java
volatile int a = b + 2;    // 错误示例，无法保证对 a 的原子性。这里的赋值包含两个步骤：首先实现 b + 2，然后将其结果赋值给了 a。因为第一个线程获取 b 进行了 + 2，第二个线程也获取了 b，则 b 的值都不一样。【b + 2，其实是两步，先读取 b 然后进行 + 2】因为这里是两条指令进行一个操作。

volatile int a = a++;     // 同样需要两条指令来完成相应的操作

volatile int count = 1;    // 正确示例

volatile boolean flag = false; // 正确示例

volatile Date date = new Date( );// 这个如果是多线程的话也可能出现问题，因为 new Date() 首先需要在堆上创建一个 Date 对象的存储并生成一些数据，然后返回一个引用给左侧的 date。加上 volatile 仅仅能保证将引用值赋值给 date 是一个原子操作。因为通常以上操作都是在一个方法中，只能被一个线程执行，所以没有问题。
```

如果要实现volatile写操作的原子性，那么在等号右侧的赋值变量中就不能出现被多线程所共享的变量，哪怕这个变量也是个volatile也不可以。

## 实现原理

防止指令重排序和实现变量的可见性都是通过内存屏障（Memory Barrier）进行实现的。

JIT 编译器将源代码生成字节码的过程中可能会进行指令的重排序以提升执行性能。在单线程问题下没有任何问题。Volatile 可以防止指令重排序，代码示例如下：

### volatile 修饰写入操作

```java
int a = 1;
String s = " hello" ;

内存屏障(Release Barrier，释放屏障) // 这是 JVM 底层自己加的，类似 monitorenter 和 monitorexist
    
volatile boolean v = false; // 这是一个写操作，看到 volatile 会在前后自动加上屏障

写入操作内存屏障(store Barrier，存储屏障)
```

- Release Barrier 实现两点作用
    - **防止下面的 volatile 与上面的所有操作的指令重排序**；
    - 让内存屏障前的所有读/写操作都能立刻发布到其它所有的线程中，使得其它线程可以看到修改结果。（为了防止重排序就把上面的代码给执行掉），保证在 volatile 的修改之前，前面所有的读/写操作都已经被提交了，当前线程在执行 volatile 写操作时候，其它线程都可以看到 volatile 前面的所有的修改，这样就保证了其他的读线程对于当前写线程在写入 volatile 变量之前对共享变量的所有更新变量的看到顺序与源代码顺序一致，即防止指令重排序。

- Store Barrier∶重要作用是刷新处理器缓存，结果是可以确保该存储屏障之前一切（包括 volatile 修饰和非 volatile 修饰）的操作所生成的结果对于其他处理器来说都立刻可见。

### volatile 修饰读操作

```java
内存屏障(Load Barrier，加载屏障)
     
volatile boolean v1 = v;

内存屏障(Acquire Barrier，获取屏障)

int a = 1;
String s = "hello" ;
```

- Load Barrier：可以刷新处理器缓存，同步其他处理器对该 volatile 变量的修改结果到自己的缓存中。

- Acquire Barrier：可以防止上面的 volatile 读取操作与下面的所有操作语句的指令重排序。

 **总结**：

对于 volatile 关键字变量的读写操作，本质上都是通过内存屏障来执行的。内存屏障兼具了防止指令重排序和实现变量内存的可见性的能力。

- 对于读取操作来说，volatile 可以确保该操作与其后续的所有读写操作都不会进行指令重排序。

- 对于修改操作来说，volatile 可以确保该操作与其上面的所有读写操作都不会进行指令重排序。

volatile 只能修饰原生类型，不能修饰非原生类型，如 ArrayList(对于非原生类型，如果是赋值将具备原子性的操作，但是创建这个对象什么的不具备原子性)

 

## volatile与锁的异同

**相似点**：Volatile 和锁都可以确保变量的内存可见性和防止指令重排序。

以 synchroned 修饰代码块为例，其防止指令重排序也是使用内存屏障实现，在两个指令的前后的内存屏障如下所示：

```java
monitorenter

内存屏障(Acquire Barrier, 获取屏障)

。。。。。

内存屏障(Release Barrier,释放屏障)

monitorexit
```

**不同点**：

- volatile 可以保证其修饰变量写操作的原子性，但是不具备锁的排他性（互斥性）。

    原子性可以理解为就是一条 CPU 指令，不能在分割了。而排他性是指像 synchronized 那样同一时刻只能有一个线程操作，而 volatile 修饰的变量可能同时有多个线程对其进行写操作。

- 使用锁可能会导致线程的上下文切换(内核态与用户态之间的切换)，但使用 volatile 则可以保证一直在用户态。



## Java 内存模型(Java Memory Model, JMM) 

JMM 主要规定以下问题，但是最终实现需要各个 JVM 来自定义实现。下面提到的变量主要是成员变量或者静态变量，因为局部变量不存在这些问题。

- 变量的原子性问题

- 变量的可见性问题

- 变量修改的时序性问题。

## happen-before 重要规则

happen-before 定义在 JMM 规划中，在多核的场景下，某个处理器对某个变量的修改操作**最终**是可以被其它处理器所知晓的，但是没确定什么时候能获取到。不**保证实时一致性，只能保证最终一致性。**

happen-before 原则一方面具有传递性，同时除了第一个在单个线程内部，其它均是在多个线程之间。

- 顺序执行规则(限定在单个线程上的) ：该线程的每个动作都 happen-before 它的后面的动作。

    > 指令重排序和 happen-before 是不矛盾的，只要不违反 happen-before 原则是允许指令重排序的，比如一个线程执行 a/b/c 三条语句，则 a  happen-before b happen before c，但是并不表示 a 一定会在 b 之前执行，因为如果  a  和 b 之间操作的语句没有任何关系的话，JIT 编译器是允许他们进行指令重排序的。但是如果有有先后关系则不会进行指令重排序。

- 隐式锁(monitor) 规则：针对同一把锁 unlock happen- before lock, 之前的线程对于同步代码块的所有执行结果对于后续获取同一把锁的线程来说都是可见的。

- volatile读写规则：对于一个volatile变量的写操作一定会happen-before后续对该变量的读操作。

- 多线程的启动规则：Thread对象的start方法happen-before该线程run方法中的任何一个动作，包括在其中启动的任何子线程。 （可以在一个线程中启动另一个线程使其作为当前线程的子线程使用，即可以在父线程的 run 方法中启动另一个线程，则子线程在执行其 run 方法前可以看到父线程在执行 start 方法前的所有操作结果）保证父线程所作的一切对子线程都是可见的。

- 多线程的终正规则：一个线程启动了一个子线程，并且调用了子线程的join方法则该线程会等待其子线程结束，那么当子线程结束后，父线程的接下来的所有操作都可以看到子线程run方法中的执行结果。

- 线程的中断规则：可以调用 interrupt 方法来中断线程，这个调用 happen-before 对该线程中断的检查(isInterrupted) 。









## 语义一：可见性

前面介绍Java内存模型的时候，我们说过可见性是指当一个线程修改了共享变量的值，其它线程能立即感知到这种变化。

关于Java内存模型的讲解请参考【[死磕 java同步系列之JMM（Java Memory Model）](https://mp.weixin.qq.com/s/jownTN--npu3o8B4c3sbeA)】。

而普通变量无法做到立即感知这一点，变量的值在线程之间的传递均需要通过主内存来完成，比如，线程A修改了一个普通变量的值，然后向主内存回写，另外一条线程B只有在线程A的回写完成之后再从主内存中读取变量的值，才能够读取到新变量的值，也就是新变量才能对线程B可见。

在这期间可能会出现不一致的情况，比如：

（1）线程A并不是修改完成后立即回写；

![volatile](六、Volatile.resource/volatile1.png)

（线路A修改了变量x的值为5，但是还没有回写，线程B从主内存读取到的还旧值0）

（2）线程B还在用着自己工作内存中的值，而并不是立即从主内存读取值；

![volatile](六、Volatile.resource/volatile2.png)

（线程A回写了变量x的值为5到主内存中，但是线程B还没有读取主内存的值，依旧在使用旧值0在进行运算）

基于以上两种情况，所以，普通变量都无法做到立即感知这一点。

但是，volatile变量可以做到立即感知这一点，也就是volatile可以保证可见性。

java内存模型规定，volatile变量的每次修改都必须立即回写到主内存中，volatile变量的每次使用都必须从主内存刷新最新的值。

![volatile](六、Volatile.resource/volatile3.png)

volatile的可见性可以通过下面的示例体现：

```java
public class VolatileTest {
    // public static int finished = 0;
    public static volatile int finished = 0;

    private static void checkFinished() {
        while (finished == 0) {
            // do nothing
        }
        System.out.println("finished");
    }

    private static void finish() {
        finished = 1;
    }

    public static void main(String[] args) throws InterruptedException {
        // 起一个线程检测是否结束
        new Thread(() -> checkFinished()).start();

        Thread.sleep(100);

        // 主线程将finished标志置为1
        finish();

        System.out.println("main finished");

    }
}
```

在上面的代码中，针对finished变量，使用volatile修饰时这个程序可以正常结束，不使用volatile修饰时这个程序永远不会结束。

因为不使用volatile修饰时，checkFinished()所在的线程每次都是读取的它自己工作内存中的变量的值，这个值一直为0，所以一直都不会跳出while循环。

使用volatile修饰时，checkFinished()所在的线程每次都是从主内存中加载最新的值，当finished被主线程修改为1的时候，它会立即感知到，进而会跳出while循环。

## 语义二：禁止重排序

前面介绍Java内存模型的时候，我们说过Java中的有序性可以概括为一句话：如果在本线程中观察，所有的操作都是有序的；如果在另一个线程中观察，所有的操作都是无序的。

前半句是指线程内表现为串行的语义，后半句是指“指令重排序”现象和“工作内存和主内存同步延迟”现象。

关于Java内存模型的讲解请参考【[死磕 java同步系列之JMM（Java Memory Model）](https://mp.weixin.qq.com/s/jownTN--npu3o8B4c3sbeA)】。

普通变量仅仅会保证在该方法的执行过程中所有依赖赋值结果的地方都能获得正确的结果，而不能保证变量赋值操作的顺序与程序代码中的执行顺序一致，因为一个线程的方法执行过程中无法感知到这点，这就是“线程内表现为串行的语义”。

比如，下面的代码：

```java
// 两个操作在一个线程
int i = 0;
int j = 1;
```

上面两句话没有依赖关系，JVM在执行的时候为了充分利用CPU的处理能力，可能会先执行`int j = 1;`这句，也就是重排序了，但是在线程内是无法感知的。

看似没有什么影响，但是如果是在多线程环境下呢？

我们再看一个例子：

```java
public class VolatileTest3 {
    private static Config config = null;
    private static volatile boolean initialized = false;

    public static void main(String[] args) {
        // 线程1负责初始化配置信息
        new Thread(() -> {
            config = new Config();
            config.name = "config";
            initialized = true;
        }).start();

        // 线程2检测到配置初始化完成后使用配置信息
        new Thread(() -> {
            while (!initialized) {
                LockSupport.parkNanos(TimeUnit.MILLISECONDS.toNanos(100));
            }

            // do sth with config
            String name = config.name;
        }).start();
    }
}

class Config {
    String name;
}
```

这个例子很简单，线程1负责初始化配置，线程2检测到配置初始化完毕，使用配置来干一些事。

在这个例子中，如果initialized不使用volatile来修饰，可能就会出现重排序，比如在初始化配置之前把initialized的值设置为了true，这样线程2读取到这个值为true了，就去使用配置了，这时候可能就会出现错误。

（此处这个例子只是用于说明重排序，实际运行时很难出现。）

通过这个例子，彤哥相信大家对“如果在本线程内观察，所有操作都是有序的；在另一个线程观察，所有操作都是无序的”有了更深刻的理解。

所以，重排序是站在另一个线程的视角的，因为在本线程中，是无法感知到重排序的影响的。

而volatile变量是禁止重排序的，它能保证程序实际运行是按代码顺序执行的。

## 实现：内存屏障

上面讲了volatile可以保证可见性和禁止重排序，那么它是怎么实现的呢？

答案就是，内存屏障。

内存屏障有两个作用：

（1）阻止屏障两侧的指令重排序；

（2）强制把写缓冲区/高速缓存中的数据回写到主内存，让缓存中相应的数据失效；

关于“内存屏障”的知识点，各路大神的观点也不完全一致，所以这里彤哥也就不展开讲述了，感兴趣的可以看看下面的文章：

（注意，公众号不允许外发链接，所以只能辛苦复制链接到浏览器中阅读了，而且还可能需要科学上网）

（1） Doug Lea的《The JSR-133 Cookbook for Compiler Writers》

http://g.oswego.edu/dl/jmm/cookbook.html

Doug Lea 就是java并发包的作者，大牛！
    
（2）Martin Thompson的《Memory Barriers/Fences》

https://mechanical-sympathy.blogspot.com/2011/07/memory-barriersfences.html

Martin Thompson 专注于把性能提升到极致，专注于从硬件层面思考问题，比如如何避免伪共享等，大牛！

它的博客地址就是上面这个地址，里面有很多底层的知识，有兴趣的可以去看看。

（3）Dennis Byrne的《Memory Barriers and JVM Concurrency》

https://www.infoq.com/articles/memory_barriers_jvm_concurrency

这是InfoQ英文站上面的一篇文章，我觉得写的挺好的，基本上综合了上面的两种观点，并从汇编层面分析了内存屏障的实现。

目前国内市面上的关于内存屏障的讲解基本不会超过这三篇文章，包括相关书籍中的介绍。

我们还是来看一个例子来理解内存屏障的影响：

```java
public class VolatileTest4 {
    // a不使用volatile修饰
    public static long a = 0;
    // 消除缓存行的影响
    public static long p1, p2, p3, p4, p5, p6, p7;
    // b使用volatile修饰
    public static volatile long b = 0;
    // 消除缓存行的影响
    public static long q1, q2, q3, q4, q5, q6, q7;
    // c不使用volatile修饰
    public static long c = 0;

    public static void main(String[] args) throws InterruptedException {
        new Thread(()->{
            while (a == 0) {
                long x = b;
            }
            System.out.println("a=" + a);
        }).start();

        new Thread(()->{
            while (c == 0) {
                long x = b;
            }
            System.out.println("c=" + c);
        }).start();

        Thread.sleep(100);

        a = 1;
        b = 1;
        c = 1;
    }
}
```

这段代码中，a和c不使用volatile修饰，b使用volatile修饰，而且我们在a/b、b/c之间各加入7个long字段消除伪共享的影响。

关于伪共享的相关知识，可以查看彤哥之前写的文章【[杂谈 什么是伪共享（false sharing）？](https://mp.weixin.qq.com/s/rd13SOSxhLA6TT13N9ni8Q)】。

在a和c的两个线程的while循环中我们获取一下b，你猜怎样？如果把`long x = b;`这行去掉呢？运行试试吧。

彤哥这里直接说结论了：volatile变量的影响范围不仅仅只包含它自己，它会对其上下的变量值的读写都有影响。

## 缺陷

上面我们介绍了volatile关键字的两大语义，那么，volatile关键字是不是就是万能的了呢？

当然不是，忘了我们内存模型那章说的一致性包括的三大特性了么？

一致性主要包含三大特性：原子性、可见性、有序性。

volatile关键字可以保证可见性和有序性，那么volatile能保证原子性么？

请看下面的例子：

```java
public class VolatileTest5 {
    public static volatile int counter = 0;

    public static void increment() {
        counter++;
    }

    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(100);
        IntStream.range(0, 100).forEach(i->
                new Thread(()-> {
                    IntStream.range(0, 1000).forEach(j->increment());
                    countDownLatch.countDown();
                }).start());

        countDownLatch.await();

        System.out.println(counter);
    }
}
```

这段代码中，我们起了100个线程分别对counter自增1000次，一共应该是增加了100000，但是实际运行结果却永远不会达到100000。

让我们来看看increment()方法的字节码（IDEA下载相关插件可以查看）：

```java
0 getstatic #2 <com/coolcoding/code/synchronize/VolatileTest5.counter>
3 iconst_1
4 iadd
5 putstatic #2 <com/coolcoding/code/synchronize/VolatileTest5.counter>
8 return
```

可以看到counter++被分解成了四条指令：

（1）getstatic，获取counter当前的值并入栈

（2）iconst_1，入栈int类型的值1

（3）iadd，将栈顶的两个值相加

（4）putstatic，将相加的结果写回到counter中

由于counter是volatile修饰的，所以getstatic会从主内存刷新最新的值，putstatic也会把修改的值立即同步到主内存。

但是中间的两步iconst_1和iadd在执行的过程中，可能counter的值已经被修改了，这时并没有重新读取主内存中的最新值，所以volatile在counter++这个场景中并不能保证其原子性。

volatile关键字只能保证可见性和有序性，不能保证原子性，要解决原子性的问题，还是只能通过加锁或使用原子类的方式解决。

进而，我们得出volatile关键字使用的场景：

（1）运算的结果并不依赖于变量的当前值，或者能够确保只有单一的线程修改变量的值；

（2）变量不需要与其他状态变量共同参与不变约束。

说白了，就是volatile本身不保证原子性，那就要增加其它的约束条件来使其所在的场景本身就是原子的。

比如：

```java
private volatile int a = 0;

// 线程A
a = 1;

// 线程B
if (a == 1) {
    // do sth
}
```

`a = 1;`这个赋值操作本身就是原子的，所以可以使用volatile来修饰。

## 总结

volatile 关键字可以保证可见性、有序性，但不能保证原子性。

- volatile 关键字的底层主要是通过内存屏障来实现的；

- **volatile 关键字的使用场景必须是场景本身就是原子的；**

