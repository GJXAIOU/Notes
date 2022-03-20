# JVM 垃圾回收

[TOC]

## 一、垃圾收集区域

### （一）概述

- 考虑哪些内存需要回收、什么时候回收以及如何回收；

- 首先**程序计数器、虚拟机栈、本地方法栈都是随线程而生/灭**，大体上都是在编译期可知的（运行期会由  JIT 编译器进行优化），因此它们在方法或者线程结束之后对应的内存空间就回收了。

- 针对 Java 堆和方法区，因为一个接口中的各个实现类需要的内存可能各不相同，一个方法的各个分支需要的内存也不一样，只能在程序运行期才能知道会创建哪些对象和回收都是动态的，需要关注这部分内存变化。

- **一般使用 new 语句创建对象的时候消耗 12 个字节，其中引用在栈上占 4 个字节，空对象在堆中占 8 个字节**。如果该语句所在方法执行结束之后，对应栈中的变量会马上进行回收，但是堆中的对象要等到 GC 来回收。

### （二）方法区

- **Java 虛拟机规范表示可以不要求虚拟机在这区实现 GC，该区 GC 的「性价比」比较低**。因为在堆中，尤其是在新生代，常规应用进行 1 次 GC 一般可以回收 70%~95% 的空间，而方法区的 GC 效率远小于此。
- 当前的商业 JVM 都有实现方法区的 GC ，主要回收两部分内容：**废弃常量与无用类**。
- ==方法区的类回收需要同时满足如下三个条件==：（可以回收但是不一定回收）
    - 该类所有的实例都已经被 GC，即 JVM 中不存在该 Class 的任何实例；
    - 加载该类的类加载器已经被 GC（因为类加载器和该类加载器加载的 Class 对象之间是双向引用的）；
    - 该类对应的 `java.lang.Class` 对象没有在任何地方被引用，且不能在任何地方通过反射访问该类的方法；
- 在大量使用反射、动态代理、CGLib 等字节码框架、动态生成 JSP 以及 OSGi 这类频繁自定义类加载器的场景都需要 JVM 具备类卸载的支持以保证方法区不会溢出。

## 二、垃圾判断

### （一）垃圾判断的算法

- 引用计数算法（Reference Counting）

    - 给对象添加一个引用计数器，当有一个地方引用它则计数器 +1，当引用失效的时候计数器 -1，任何时刻计数器为 0 的对象就是不可能再被使用；

    - **引用计数算法无法解决对象循环引用的问题**。==问题：循环引用能不能解决==

      如下面代码中两个对象处理互相引用对方，再无任何引用

      ```java
      package com.gjxaiou.gc.algorithm;
      
      import org.junit.Test;
      
      public class ReferenceCountingGC {
          public Object instance = null;
          private static final int memory = 1024 * 1024;
          /**
           * 该成员属性作用为：占用内存，以便能在 GC 日志中看清楚是否被回收过
           */
          private byte[] bigSize = new byte[2 * memory];
      
          @Test
          public  void testGC() {
              ReferenceCountingGC objA = new ReferenceCountingGC();
              ReferenceCountingGC objB = new ReferenceCountingGC();
              objA.instance = objB;
              objB.instance = objA;
              objA = null;
              objB = null;
      
              // 直接进行 GC
              System.gc();
          }
      }
      
      ```
    
    分析：`testGC()` 方法的前四行执行之后，`objA` 对象被 `objA` 和 `objB.instance` 引用着，`objB` 也类似；执行`objA = null` 和 `objB = null` 之后，`objA` 对象的 `objA` 引用失效，但是 `objB.instance` 引用仍然存在，因此如果采用单纯的引用计数法，`objA` 并不会被回收，除非在执行 `objB = null` 时，遍历 `objB` 对象的属性，将里面的引用全部置为无效。

- 根搜索算法( GC Roots Tracing )【可达性】

    - **在实际的生产语言中（Java、 C#等）都是使用根搜索算法判定对象是否存活**。

    - 算法基本思路就是通过一系列的称为 GC Roots 的点作为起始点进行向下搜索，当一个对象到 GC Roots 没有任何引用链（Reference Chain）相连，则证明此对象是不可用的。下图中 object5/6/7 之间虽然互相有引用，但是它们到 GC Roots 是不可达的，因此会被判定为是可回收对象。

### （二）可作为GC Roots的对象

- 虚拟机栈（栈帧中的本地变量表）中引用的对象。
- 方法区中类静态属性引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象

==问题：为什么这些可以作为 GC Root 点，同时能够输出所有的 GC root 点==

<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191212212156641.png" alt="image-20191212212156641" style="zoom:50%;" />

-----------

## 三、引用

**引用分为强引用（Strong Reference）、 软引用（Soft Reference）、 弱引用（Weak Reference）、 虚引用（Phantom Reference）**4 种，其引用强度依次逐渐减弱。

- 强引用就是指在程序代码之中普遍存在的，类似 `Object obj = new Object()` 这类的引用，只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象。
- 软引用是用来描述一些还有用但并非必需的对象。 **对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。 如果这次回收还没有足够的内存，才会抛出内存溢出异常**。 在 JDK 1.2 之后，提供了 `SoftReference` 类来实现软引用。
- 弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，**被弱引用关联的对象只能生存到下一次垃圾收集发生之前**。 当垃圾收集器工作时，**无论当前内存是否足够，都会回收掉只被弱引用关联的对象**。 在JDK 1.2 之后，提供了 `WeakReference` 类来实现弱引用。
- 虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。**为一个对象设置虚引用关联的唯一目的就是希望能在这个对象被收集器回收时收到一个系统通知**。在 JDK 1.2 之后，提供了 `PhantomReference` 类来实现虚引用。


示例：

```java
MyObject ref = new MyObject();
SoftReference softRef = new SoftReference(ref);
```

**一旦 SoftReference 保存了对一个 Java 对象的软引用后，在垃圾线程对这个 Java 对象回收前，SoftReference 类所提供的 get() 方法返回 Java 对象的强引用**。另外，一旦垃圾线程回收该 Java 对象之后，get() 方法将返回 null。在 Java 集合中有一种特殊的 Map 类型：WeakHashMap， 在这种 Map 中存放了键对象的弱引用，当一个键对象被垃圾回收，那么相应的值对象的引用会从 Map 中删除。WeakHashMap 能够节约存储空间，可用来缓存那些非必须存在的数据。

## 四、对象回收过程

**即使在可达性分析算法中不可达的对象，也并非是「非死不可」的**，这时候它们暂时处于「缓刑」阶段，要真正宣告一个对象死亡，至少要经历两次标记过程：

- 如果对象在进行可达性分析后发现没有与 GC Roots 相连接的引用链，那它将会被第一次标记并且进行一次筛选，**筛选的条件是**此对象是否有必要执行 finalize() 方法。当对象没有覆盖 finalize() 方法，或者 finalize() 方法已经被虚拟机调用过，虚拟机将这两种情况都视为「没有必要执行」。**如果这个对象被判定为有必要执行 finalize() 方法，那么这个对象将会放置在一个叫做 F-Queue 的队列之中，并在稍后由一个由虚拟机自动建立的、低优先级的 Finalizer 线程去执行它**。**这里所谓的「执行」是指虚拟机会触发这个方法，但并不承诺会等待它运行结束**，这样做的原因是，如果一个对象在 finalize() 方法中执行缓慢或者发生了死循环，将很可能会导致 F-Queue 队列中其他对象永久处于等待，甚至导致整个内存回收系统崩溃。

- finalize() 方法是对象逃脱死亡命运的最后一次机会，稍后 GC 将对 F-Queue 中的对象进行第二次小规模的标记，如果对象要在 finalize() 中成功拯救自己——**只要重新与引用链上的任何一个对象建立关联即可，譬如把自己（this 关键字）赋值给某个类变量或者对象的成员变量**，那在第二次标记时它将被移除出「即将回收」的集合；如果对象这时候还没有逃脱,那基本上它就真的被回收了。

代码示例：

```java
package com.gjxaiou.gc;

/**
 * 此代码演示两点：
 * 1.对象可以在 GC 时自我救赎。
 * 2.这种自我救赎的机会只有一次，因为finalize()方法最多只会被调用一次。
 */
public class FinalizeEscapeGC {
    public static FinalizeEscapeGC saveMe = null;

    public void isLive() {
        System.out.println("我还活着！");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("执行finalize()方法中……");
        // 完成自我救赎
        saveMe = this;
    }

    public static void main(String[] args) throws InterruptedException {
        saveMe = new FinalizeEscapeGC();

        // 对象第一次拯救自己
        saveMe = null;
        System.gc();

        // 因为finalize方法优先级比较低，所以暂停进行等待
        Thread.sleep(5000);

        if (saveMe == null) {
            System.out.println("我已经死亡！");
        } else {
            saveMe.isLive();
        }

        // 对象第二次自我救赎，失败
        saveMe = null;
        System.gc();
        Thread.sleep(5000);

        if (saveMe == null) {
            System.out.println("我已经死亡！");
        } else {
            saveMe.isLive();
        }
    }
}
```

执行结果：

```java
执行finalize()方法中……
我还活着！
我已经死亡！
```

从结果可以看出 saveMe 对象的 finalize() 方法确实被 GC 收集器触发过，但是在被收集前逃脱了；

同时程序中两段相同的代码执行结果一次逃脱一次失败，因为任何一个对象的 finalize() 方法都只会被系统自动调用一次，如果对象面临下一次回收，它的 finalize() 方法不会被再次执行，因此第二段代码中自救失败。

![finalize 执行过程](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/finalize%20%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.jpg)

需要特别说明的是，**建议尽量避免使用这种方法来拯救对象**，因为它不是 C/C++ 中的析构函数，而是 Java 刚诞生时为了使 C/C++ 程序员更容易接受它所做出的一个妥协。它的运行代价高昂，不确定性大，无法保证各个对象的调用顺序。有些教材中描述它适合做“关闭外部资源”之类的工作“，这完全是对这个方法用途的一种自我安慰。finalize() 能做的所有工作，使用 try-finally 或者其他方式都可以做得更好、更及时，所以建议大家完全可以忘掉 Java 语言中有这个方法的存在。

## 五、JVM 常见的 GC 算法

- 标记-清除算法（Mark Sweep）
- 标记-整理算法（Mark-Compact）
- 复制算法（Copying）
- 分代算法（Generational）


**新生代使用复制算法，老年代一般采用标记-清除算法或者标记-整理算法**；

### （一）标记一清除算法（Mark-Sweep）

- 算法分为「标记」和「清除」两个阶段， 首先标记出所有需要回收的对象，然后回收所有需要回收的对象；

- 缺点：

    - 效率问题，标记和清理两个过程效率都不高，需要扫描所有对象，因此堆越大，GC 越慢； 
    - 空间问题， ==**标记清理之后会产生大量不连续的内存碎片，空间碎片太多可能会导致后续使用中无法找到足够的连续内存来分配给对象而提前触发另一次的垃圾收集动作**==；GC 次数越多，碎片越为严重。
    
    <img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/1574823017674.gif" alt="1574823017674" style="zoom:50%;" />

    上图中，左侧是运行时虚拟机栈（可以作为 GC Roots），箭头表示引用，绿色就是不能被回收的。

### （二）标记一整理算法（Mark-Compact）

- 标记过程仍然一样，但后续步骤不是进行直接清理，而是令所有存活的对象一端移动，然后直接清理掉这端边界以外的内存。

- 优点：**没有内存碎片**；

- 缺点：比标记清理耗费更多的时间进行整理；

  <img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191212212328669.png" alt="image-20191212212328669" style="zoom:50%;" />

### （三）复制收集算法

> 用于解决上面的效率问题

- 将可用内存划分为两块，每次只使用其中的一块，当一半区内存用完了，仅将还存活的对象复制到另外一块上面，然后就把原来整块内存空间一次性清理掉；这样使得每次内存回收都是对整个半区的回收，内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存就可以了，实现简单，运行高效。只是这种算法的代价是将内存缩小为原来的一半，代价高昂；
- **现在的商业虚拟机中都使用该种收集算法来回收新生代**；
- 算法优化（减少浪费的空间）：
  - 将内存分为一块较大的 eden 空间和 2 块较少且相同的 survivor 空间，每次使用 eden 和其中一块 survivor，当回收时将 eden 和 survivor 还存活的对象一次性拷贝到另外一块 survivor 空间上，然后清理掉 eden 和用过的 survivor；
  - Oracle HotSpot 虚拟机默认 eden 和 survivor 的大小比例是 8:1，即每次只有 10% 的内存是「浪费」的；
- 复制收集算法在对象存活率高的时候，效率有所下降；
- 如果不想浪费 50% 的空间（而采用上面优化方法），就需要有额外的空间进行分配担保用于应付半区内存中所有对象都 100% 存活的极端情况，**所以在老年代一般不能直接选用这种算法**；

<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/1574824343266.gif" alt="1574824343266" style="zoom:50%;" />

- 优点：
  - 只需要扫描存活的对象，效率更高；
  - 不会产生碎片；
  - **复制算法非常适合生命周期比较短的对象，因为每次 GC 总能回收大部分的对象，复制的开销比较小**；
  - 根据 IBM 的专项研究，98% 的 Java 对象只会存活 1 个 GC 周期，对这些对象很适合用复制算法。而且不用 1: 1 的划分工作区和复制区的空间；
- 缺点：
  - 需要浪费额外的内存作为复制区。
  - 如果回收之后存活的对象大于 10%，即 Survivor 区域放置不下的时候，需要依赖其他内存（这里是指老年代）进行分配担保（Handle Promotion）【**因此如果另外一块 survivor 空间没有足够空间存放上一次新生代收集下来的存活对象，这些对象将直接通过分配担保机制进入老年代**】。
  - 对象存活率较高时候复制操作较多，同时需要额外的空间分配担保，因此**老年代不适用该算法**。

### （四）分代收集（Generational Collecting）算法

- 当前商业虚拟机的垃圾收集都是采用「分代收集」( Generational Collecting）算法，根据对象不同的存活周期将内存划分为几块。

- **一般是把 Java 堆分作新生代和老年代**，这样就可以根据各个年代的特点采用最适当的收集算法，譬如新生代每次 GC 都有大批对象死去，只有少量存活，则选用复制算法，只需要付出少量存活对象的复制成本，即完成收集。同时老年代中对象存活率较高，没有额外空间对其进行担保，必须使用「标记-清理」 或者 「标记-整理」进行回收。

- HotSpot JVM 6 中共划分为三个代:
  
  年轻代（Young Generation）、老年代（Old Generation）、永久代（Permanent Generation）

<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/drrrr.png" alt="Hotspot JVM 6中共划分为三个代" style="zoom: 50%;" />

- 年轻代（新生代）
    - 新生成的对象都放在新生代，年轻代用复制算法进行 GC （理论上年轻代对象的生命周期非常短，所以适合复制算法）。
    - 年轻代分三个区：一个 Eden 区，两个 Survivor 区（可以通过参数设置 Survivor 个数）。对象在 Eden 区中生成。当 Eden 区满时，还存活的对象将被复制到一个 Survivor 区，当这个 Survivor 区满时，此区的存活对象将被复制到另外一个 Survivor 区，当第二个 Survivor 区也满了的时候，从第一个 Survivor 区复制过来的并且此时还存活的对象，将被复制到老年代。2 个 Survivor 是完全对称，轮流替换；
    - Eden 和 2 个 Survivor 的缺省比例是 8:1:1，即 10% 的空间会被浪费。可以根据 GC 日志的信息调整大小的比例；
- 老年代
    - 存放了经过一次或多次 GC 还存活的对象；
    - 一般采用 Mark-Sweep 或者 Mark-Compact 算法进行 GC；
    - 有多种垃圾收集器可以选择。每种垃圾收集器可以看作一个 GC 算法的具体实现。可以根据具体应用的需求选用合适的垃圾收集器（例如追求吞吐量或者最短的响应时间）。
- ~~永久代~~
    - 并不属于堆（Heap），但是 GC 也会涉及到这个区域；
    - 存放了每个 Class 的结构信息， 包括常量池、字段描述、方法描述。与垃圾收集要收集的 Java 对象关系不大；

## 六、内存分配与回收

### （一）内存分配和回收方案

- 堆上分配： 大多数情况在 eden 上分配，偶尔会直接在 old 上分配，细节取决于不同 GC 的实现；如果启动了本地线程分配缓冲，将按线程优先在 TLAB 上分配。
- **栈上分配： 原子类型的局部变量**；
- GC 要做的是将那些死亡的对象所占用的内存回收掉
    - HotSpot 认为没有引用的对象是死亡的；
    - HotSpot 将引用分为四种: Strong、 Soft、Weak、Phantom， Strong 即默认通过 `Object o = new Object()` 这种方式赋值的引用 ，Soft、Weak、 Phantom 这三种则都是继承 Reference；
- 在 Full GC 时会对 Reference 类型的引用进行特殊处理
    - Soft：内存不够时一定会被 GC、长期不用也会被 GC
    - Weak：一定会被 GC， 当被标记为 dead, 会在 Reference Queue 中通知
    - Phantom：本来就没引用，当从 JVM 堆中释放时会通知

### （二）GC 回收的时机

在分代模型（新生代和老年代）的基础上，GC 从时机上分为两种：Scavenge GC 和 Full GC
- Scavenge GC (Minor GC)
    -  触发时机：新对象生成时，Eden 空间满了
    - 理论上 Eden 区大多数对象会在 Scavenge GC 回收，复制算法的执行效率会很高，Scavenge GC 时间比较短。
- Full GC 
    - 对整个 JVM 进行整理，包括 Young、Old 和 Perm
    - 主要的触发时机
        -  Old 满了
        - Perm 满了
        - 执行 `system.gc()`
    - 效率很低，尽量减少 Full GC。

## 七、HotSpot 的算法实现

### （一）枚举根节点

- 当 Java 执行系统停顿（保证分析过程中不会出现对象引用关系的变更）下来之后，并不需要一个不漏的检查完所有执行上下文和全局的引用位置（这两者通常作为  GC Roots 的节点），虚拟机应当有办法直接得知哪些地方存放着对象引用。在 HotSpot 的实现中，是使用一组称为 OopMap （OOP：Ordinary Object Pointer 普通对象指针）的数据结构来达到该目的。在类加载完成之后，HotSpot 就把对象内什么偏移量上面是什么类型的数据计算出来了，在 JIT 编译过程中，也会在特定位置记录栈和寄存器中哪些位置是引用。所以 GC 扫描时候就可以得知。

- **CMS 收集器在枚举根节点时候也必须停顿**。

### （二）安全点

在 OopMap 的协助下，HotSpot 可以快速且准确的完成 GC Roots 枚举，但一个很现实的问题随之而来：可能导致引用关系变化，或者说 OopMap 内容变化的指令非常多，如果为每一条指令都生成对应的 OopMap，那将会需要大量的额外空间，这样 GC 的空间成本将会更高。

**实际上，HotSpot 并没有为每条指令都生成 OopMap，而只是在「特定位置」记录了这些信息，这些位置称为「安全点（Safepoint）」，即程序执行时并非在所有地方都能停顿下来开始 GC，只有在达到安全点时才能暂停。**

Safepoint 的选定既不能太少以致于让 GC 等待时间太长，也不能过于频繁以致于过分增大运行时的负荷。所以，安全点的选定基本上是以程序「是否具有让程序长时间执行的特征」为标准进行选定的。因为每条指令执行的时间非常短暂，程序不太可能因为指令流长度太长这个原因而过长时间运行，「长时间执行」的最明显特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等等，所以具有这些功能的指令才会产生 Safepoint。

对于安全点，另一个需要考虑的问题是如何在 GC 发生时让所有线程（这里不包括执行 JNI 调用的线程）都「跑」到最近的安全点上再停顿下来。这里有两种方案可供选择：抢先式中断和主动式中断。

- **抢先式中断**：不需要线程的执行代码主动去配合，在 GC 发生时，首先把所有线程（应用线程）全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它「跑」到安全点上。但是**现在几乎没有**虚拟机实现采用抢先式中断来暂停线程从而响应 GC 事件。

- **主动式中断**：当 GC 需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的，另外再加上创建对象需要分配内存的地方。

### （三）安全区域

使用安全点似乎已经完美地解决了如何进入 GC 的问题，但是实际情况却并不一定。安全点机制保证了程序执行时，在不太长的时间内就会遇到可进入 GC 的 Safepoint。**但是在程序不执行的时候就无法做到这一点，比如线程在休眠或阻塞状态。对于这种情况，就需要安全区域（Safe Region）来解决**。 

安全区域是指在一段代码片段之中，引用关系不会发生变化。在这个区域中的任意地方开始 GC 都是安全的。我们也可以把 Safe Region 看做是被扩展了的 Safepoint。

**在线程执行到 Safe Region 中的代码时，首先标识自己已经进入了 Safe Region，那样，当在这段时间里 JVM 要发起 GC 时，就不用管标识自己为 Safe Region 状态的线程了**。在线程要离开 SafeRegion 时，它要检查系统是否已经完成了根节点枚举（或者是整个 GC 过程），如果完成了，那线程就继续执行，否则它就必须等待直到收到可以安全离开 Safe Region 的信号为止。

## 七、垃圾回收器（Garbage Collector）

Java 7/8中默认使用收集器组合为：Parallel Scavenge（新生代）+ Parallel Old（老年代），要使用 CMS 或者 G1，请手动打开。

收集算法是内存回收的方法论，垃圾收集器就是内存回收的具体实现。

- 分代模型: GC 的宏观愿景;
- 垃圾回收器: GC 的具体实现，HotSpot JVM 提供多种垃圾回收器，我们需要根据具体应用的需要采用不同的回收器，没有万能的垃圾回收器，每种垃圾回收器都有自己的适用场景

**垃圾收集器的「并行」和「并发」**

- 并行（Parallel）：指多个收集器的线程同时工作，但是用户线程处于等待状态；
- 并发（Concurrent）：指收集器在工作的同时，可以允许用户线程工作。并发不代表解决了GC 停顿的问题，在关键的步骤还是要停顿。比如在收集器标记垃圾的时候。但在清除垃圾的时候，用户线程可以和 GC 线程并发执行。同时用户线程和垃圾收集线程同时执行，并不代表两者一定是并行，可能是交替执行，用户程序在继续执行，垃圾收集程序运行在另一个 CPU 之上。

![常见垃圾回收器](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/%E5%B8%B8%E8%A7%81%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8.png)

### （一）Serial 收集器

- **单线程收集器**，收集会暂停所有工作线程（Stop The World，STW），使用**复制收集算法**，虚拟机运行在 Client 模式时的默认新生代收集器；（因为该模式下虚拟机管理的内存小，并且该收集器没有线程交互，接收机效率高，整体的停顿时间可接受）

- **新生代和老年代都可以使用**
    - **在新生代，采用复制算法**;
    - **在老年代，采用标记-整理算法**，因为是单线程 GC，没有多线程切换的额外开销，简单实用，是 HotSpot Client 模式默认的收集器。


![Serial收集器](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/serial.png)

### （二）ParNew 收集器

- ParNew 收集器就是 Serial 的多线程版本，除了使用多个收集线程外，其余行为包括算法、STW、对象分配规则、回收策略等都与 Serial 收集器一样。
- 是虚拟机运行**在 Server 模式的默认新生代收集器**（因为除了 Serial ，只有 ParNew 可以和 CMS 一起工作），在单 CPU 的环境中，ParNew 收集器并不会比 Serial 收集器有更好的效果；
- 使用复制算法（因为针对新生代），只有在多 CPU 的环境下，效率才会比 Serial 收集器高；
- 可以通过 `-XX:ParallelGCThreads` 来控制 GC 线程数的多少，需要结合具体 CPU 的个数；
- 可以通过 `-XX:+UseConcMarkSweepGC` 或者**`-XX:+UseParNewGC` 来指定其为新生代收集器**；

![ParNew收集器](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/parnew.png)

### （三）Parallel Scavenge 收集器

- 适合需要与用户交互的程序，具有较高的响应速度，适合在后台运算而不需要太多交互的任务。

- 是一个**应用于新生代、使用复制算法的并行多线程收集器**，但它的对象分配规则与回收策略都与 ParNew 收集器有所不同，它是**以吞吐量最大化**（即 GC 时间占总运行时间最小）为目标的收集器实现，它允许较长时间的 STW 换取总吞吐量最大化；

    > 吞吐量 = 运行用户代码时间 / （运行用户代码时间 + 垃圾收集时间）；

- 参数：`-XX:MaxGCPauseMillis` 控制最大垃圾收集停顿时间，数值为大于 0 的毫秒数，收集器会尽可能保证内存回收时间不超过设定值。值不能太小，**GC 停顿时间缩短是以牺牲吞吐量和新生代空间换取的**，新生代越小，会导致垃圾回收更加频繁，停顿时间下降但是吞吐量也下降。 

- 参数：`-XX:GCTimeRatio` 直接设置吞吐量（是个百分比）大小；值为 0-100，默认值为 99，即表示允许最大 1 / (1 + 99) 的垃圾收集时间。

- 参数：`-XX:+UseAdaptiveSizePolicy` 为开关参数，打开后无需设定新生代大小、Eden 和 Survivor 比例等等，虚拟机会根据系统运行情况自动调节，即 **GC 自适应的调节策略（GC Ergonomics）**。

### （四）Serial Old 收集器

- Serial Old 是单线程收集器，使用标记- 整理算法，是老年代的收集器；
- 同样主要用于 Client 模式下的虚拟机使用；
- Server 模式下：**作为 CMS 收集器的后备预案，在并发收集发生 Concurrent  Mode Failure 时候使用**。

### （五）Parallel Old 收集器

- 是 Parallel Scavenge 收集器的老年代版本，使用多线程和标记 - 整理算法，**吞吐量优先收集器**。

- 从 JDK 1.6 开始提供，在此之前，新生代使用了 PS 收集器的话，老年代只能使用 Serial Old 收集器（无法充分利用服务器的多 CPU 处理能力）整体效果不好，因为 PS 无法和 CMS 收集器配合工作；

    ![image-20191212220755481](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191212220755481.png)

### （六）CMS（Concurrent Mark Sweep）收集器

- CMS 是一种以**最短停顿时间为目标**的收集器，使用 CMS 并不能达到 GC 效率最高（总体 GC 时间最小），但它能尽可能降低 GC 时服务的停顿时间，**CMS 收集器使用的是标记一清除算法**；是 HotSpot 中真正意义上的第一款并发垃圾收集器。
- 特点：
    - 追求最短停顿时间，非常适合 Web 应用；
    - **只针对老年区**，一般结合 ParNew 使用；
    - GC 线程和用户线程并发工作（尽量并发 ），因此只有在多 CPU 环境下才有意义；
    - 使用 `-XX:+UseConcMarkSweepGC` 打开；
- CMS 收集器的缺点
    - CMS 以牺牲 CPU 资源的代价来减少用户线程的停顿。当 CPU 个数少于 4 的时候，有可能对吞吐量影响非常大；
    - CMS 在并发清理的过程中，用户线程还在跑。这时候需要预留一部分空间给用户线程；
    - CMS 用标记清除算法会带来碎片问题，碎片过多的时候会容易频繁触发 Full GC；

![CMS收集器](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/cms.png)

##  八、常见 Java 内存泄露（OOM）的经典原因

- 对象定义在错误的范围
- 异常（Exception）处理不当
- 集合数据管理不当

代码示例一：对象定义在错误范围

```java
// 方式一：如果 Foo 实例对象的生命周期较长，会导致临时性内存泄露（这里的 names 变量其实就是临时作用）
Class Foo{
    // names 变量定义在类中，即使没有其他地方使用该变量，但是因为 Foo 实例存在，所以该变量一直存在
	private String[] names;
    public void doIt(int length){
    	if(names == null || names.length < lenth){
        	names = new String[length];
        	populate(names);
            print(names);
        }
    }
}

// 修改方式二： JVM 喜欢生命周期短的对象，更加的高效
class Foo{
	public void doIt(int length){
    // 将 names 从成员变量变成局部变量，当 doIt 方法执行完成之后里面的局部变量都会被回收，所以不论 Foo 这个实例存活多长时间，都不会影响 names 被回收
    	String[] names = new String[length];
        populate(names);
        print(names);
    }
}
```

代码示例二：异常处理不当

```java
// 方式一：如果 doSomeStuff() 中抛出异常，则 rs.close() 和 conn.close() 不会被调用，导致内存泄露和 DB 连接泄露
Connection conn = DriverManager.getConnection(url, name, passwd);

try{
	String sql = "do a query sql";
    PreparedStatement stmt = conn.prepareStatement(sql);
    ResultSet rs = stmt.executeQuery();
    while (rs.next()){
    	doSomeStuff();
    }
    rs.close();
    conn.close();
}catch(Exception e){

}


// 方式二：修改如下，将资源关闭操作放在 finally 语句中
Connection conn = null; 
ResultSet rs = null;

try{
	String sql = "do a query sql";
    stmt = conn.prepareStatement(sql);
    ResultSet rs = stmt.executeQuery();
    while (rs.next()){
    	doSomeStuff();
    }
}catch(Exception e){

} finally {
	if (rs != null){
		rs.close();
	}
    if(stmt != null){
  		stmt.close();
  	}  
	conn.close();
}
```

代码示例三：数据集合管理不当

- 当我们使用基于数组的数据结构（如 ArrayList，HashMap 的时候），尽量较少 resize 操作，因为一旦重新指定大小或者扩容则必定带来复制操作，成本较高；
    - 比如在创建 ArrayList 时候尽量估计 Size，在创建的时候就将 size 估算好；
    - 减少 resize 可以避免没有必要的数组拷贝、GC 碎片等问题；
- 如果一个 List 只需要进行顺序访问，不需要随机访问，则使用 Linkedlist 代替 ArrayList，因为 Linkedlist 本质上链表，不需要 resize，但是只适用于顺序操作；

## 九、对象分配和回收示例验证代码

测试是在 Client 模式虚拟机进行，默认未指定收集器组合情况下是使用 Serial / Serial Old 收集器（ParNew / Serial Old 收集器组合的规则类似）来验证内存分配和回收策略。

### （一）对象优先在 Eden 分配

VM Options： ==verbose  和 gcdetail 区别？==

-  `-verbose:gc` ：会输出详细的垃圾回收的日志
-  `-Xms20M`：设置虚拟机启动时候堆初始大小为 20 M
- `-Xmx20M`：设置虚拟机中堆最大值为 20 M
- `-Xmn10M`：设置堆中新生代大小为 10 M
- `-XX:+PrintGCDetails`：打印出 GC 详细信息
- `-XX:SurvivorRatio=8`：表示 Eden 空间和 survivor 空间占比为 8:1

```java
package com.gjxaiou.gc;

/**
 * @Author GJXAIOU
 * @Date 2019/12/13 20:50
 */
public class MyTest1 {
    public static void main(String[] args) {
        int size = 1024 * 1024;
        // 这种情况下只有 GC，如果数组大小都是 3 * size，则还会包括 Full GC
        byte[] myAlloc1 = new byte[2 * size];
        byte[] myAlloc2 = new byte[2 * size];
        byte[] myAlloc3 = new byte[3 * size];
        System.out.println("hello world");
    }
}
```

输出结果：

```java
// (触发 GC 的原因)[新生代使用 Parallel Scavenge 收集器：垃圾回收之前新生代存活对象占用的空间->垃圾回收之后新生代存活对象占用的空间（新生代总的空间容量，因为这里包括 Eden 和 survivor 区域，survivor 包括 FromSurvivor 和 toSurvivor，两者只有一个可以被使用）] 执行 GC 之前总的堆中存活对象占空间的大小，包括新生代和老年代 -> GC 之后堆中活着占空间大小【因为前面对象还活着，所以变化不大】（总的堆中可用容量），执行 GC 花费时间
[GC (Allocation Failure) [PSYoungGen: 5751K->824K(9216K)] 5751K->4928K(19456K), 0.0018545 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
hello world
Heap
 PSYoungGen      total 9216K, used 4219K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 41% used [0x00000000ff600000,0x00000000ff950ce0,0x00000000ffe00000)
  from space 1024K, 80% used [0x00000000ffe00000,0x00000000ffece030,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
// GC 时候发现前面对象太大无法放入 Survivor 空间（Survivor 大小为 1 M），所以只能通过分配担保机制提前转移到老年代中。                             
 ParOldGen       total 10240K, used 4104K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 40% used [0x00000000fec00000,0x00000000ff002020,0x00000000ff600000)
 Metaspace       used 3135K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 342K, capacity 388K, committed 512K, reserved 1048576K
```

上面运算结果计算比较：

`PSYoungGen: 5751K->824K(9216K)] 5751K->4928K(19456K)`：5751 - 824 = 4927K，表示执行完 GC 之后，新生代释放的空间（包括真正释放的空间和晋升到老年代的空间）， 5751 - 4928 = 823k，表示执行完 GC 之后，总的堆空间释放的容量（真正释放的空间），所以 4927 - 823 = 4104k ，表示从新生代晋升到老年代的空间，正好和 ：`ParOldGen    total 10240K, used 4104K` 符合。

输出结果二：将创建数组大小均改为： 3 * size 之后会产生 Full GC

```java
[GC (Allocation Failure) [PSYoungGen: 7963K->824K(9216K)] 7963K->6976K(19456K), 0.0026002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
// Full GC 会对老年代和元空间进行回收
[Full GC (Ergonomics) [PSYoungGen: 824K->0K(9216K)] [ParOldGen: 6152K->6759K(10240K)] 6976K->6759K(19456K), [Metaspace: 3132K->3132K(1056768K)], 0.0051304 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
hello world
Heap
 PSYoungGen      total 9216K, used 3396K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 41% used [0x00000000ff600000,0x00000000ff9512a0,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 // Par：Parallel Old（老年代垃圾收集器）
 ParOldGen       total 10240K, used 6759K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 66% used [0x00000000fec00000,0x00000000ff299e18,0x00000000ff600000)
 Metaspace       used 3151K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 343K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```



#### 大对象直接进入老年代

**新生代和老年代**

打印默认的 JVM 参数 `java -XX:+PrintCommandLineFlags -version`，对应的控制台输出结果为：

```java
-XX:InitialHeapSize=266067584 -XX:MaxHeapSize=4257081344 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

其中`-XX:+UseParallelGC` 表示默认对新生代使用 `Parallel Scavenge` ，对老年代使用 `Parallel Old`垃圾收集器；

- 测试一：

    `-verbose:gc -Xms20M  -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:PretenureSizeThreshold=4194304 -XX:+UseSerialGC`

    其中 `-XX:PretenureSizeThreshold=4194304` 表示当我们创建对象的字节大于 `PretenureSizeThreshold` 的数值（单位：字节），对象将不会在新生代分配而是直接进入老年代（避免 Eden 区和两个 Survivor 去之间发生大量的内存复制）；**该参数需要和串行垃圾收集器配合使用**，因此在上面参数中同时制定了使用 Serial 垃圾收集器。 **该参数只对 Serial 和 ParNew 收集器有用**。

    ```java
    package com.gjxaiou.gc;
    
    /**
     * @Author GJXAIOU
     * @Date 2019/12/15 10:27
     */
    public class MyTest2 {
        public static void main(String[] args) {
            int size = 1024 * 1024;
            byte[] bytes = new byte[5 * size];
        }
    }
    ```

    从下面结果中：` tenured generation   total 10240K, used 5120K` 可以看出是直接在老年代进行了分配；

    ```java
    Heap
     def new generation   total 9216K, used 1983K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
      eden space 8192K,  24% used [0x00000000fec00000, 0x00000000fedefd20, 0x00000000ff400000)
      from space 1024K,   0% used [0x00000000ff400000, 0x00000000ff400000, 0x00000000ff500000)
      to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
     tenured generation   total 10240K, used 5120K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
       the space 10240K,  50% used [0x00000000ff600000, 0x00000000ffb00010, 0x00000000ffb00200, 0x0000000100000000)
     Metaspace       used 3149K, capacity 4496K, committed 4864K, reserved 1056768K
      class space    used 343K, capacity 388K, committed 512K, reserved 1048576K
    ```

- 测试二：去掉上面程序中 VM Options 中的 `-XX:+UseSerialGC`，同时将字节数组空间改为 `8 * size`，结果如下：

    ```java
    Heap
     PSYoungGen      total 9216K, used 1983K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
      eden space 8192K, 24% used [0x00000000ff600000,0x00000000ff7efd20,0x00000000ffe00000)
      from space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
      to   space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
     ParOldGen       total 10240K, used 8192K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
      object space 10240K, 80% used [0x00000000fec00000,0x00000000ff400010,0x00000000ff600000)
     Metaspace       used 3202K, capacity 4496K, committed 4864K, reserved 1056768K
      class space    used 346K, capacity 388K, committed 512K, reserved 1048576K
    ```

    因为 Eden 空间的大小为 8 * size，但是因为新创建的对象大小为 8 * size，因此 Eden 空间容纳不了，因此直接进入老年代（对象是不可能拆分放入两个代的）

- 测试三：同上，但是将空间大小改为 10 * size

    ```java
    [GC (Allocation Failure) [PSYoungGen: 1819K->808K(9216K)] 1819K->816K(19456K), 0.0006639 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 808K->808K(9216K)] 816K->816K(19456K), 0.0005789 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [Full GC (Allocation Failure) [PSYoungGen: 808K->0K(9216K)] [ParOldGen: 8K->612K(10240K)] 816K->612K(19456K), [Metaspace: 3116K->3116K(1056768K)], 0.0037378 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [GC (Allocation Failure) [PSYoungGen: 0K->0K(9216K)] 612K->612K(19456K), 0.0002203 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    [Full GC (Allocation Failure) [PSYoungGen: 0K->0K(9216K)] [ParOldGen: 612K->594K(10240K)] 612K->594K(19456K), [Metaspace: 3116K->3116K(1056768K)], 0.0041631 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
    Heap
     PSYoungGen      total 9216K, used 410K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
      eden space 8192K, 5% used [0x00000000ff600000,0x00000000ff666800,0x00000000ffe00000)
      from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
      to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
     ParOldGen       total 10240K, used 594K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
      object space 10240K, 5% used [0x00000000fec00000,0x00000000fec94b58,0x00000000ff600000)
     Metaspace       used 3200K, capacity 4496K, committed 4864K, reserved 1056768K
      class space    used 347K, capacity 388K, committed 512K, reserved 1048576K
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
    	at com.gjxaiou.gc.MyTest2.main(MyTest2.java:10)
    ```

- 测试四：恢复原来参数 `-XX:+UseSerialGC`，代码更改如下：

    ```java
    package com.gjxaiou.gc;
    
    /**
     * @Author GJXAIOU
     * @Date 2019/12/15 10:27
     */
    public class MyTest2 {
        public static void main(String[] args) {
            int size = 1024 * 1024;
            byte[] bytes = new byte[5 * size];
            try {
                Thread.sleep(1000000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    ```
    
程序执行过程中使用 JVisualVM 观察堆空间状况：
    
![image-20191215104352640](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191215104352640.png)

程序输出结果和对应的监控图片为：

```java
// 该 GC 是因为当启动一个检测工具（这里为 JVisualVM），会对原有的进程进行一次 Touch 动作，会创建一些对象从而造成内存空间不够从而会进行 GC（Minor GC），
[GC (Allocation Failure) [DefNew: 8192K->1024K(9216K), 0.0218174 secs] 13312K->6779K(19456K), 0.0218597 secs] [Times: user=0.01 sys=0.00, real=0.02 secs] 
[GC (Allocation Failure) [DefNew: 9216K->494K(9216K), 0.1351672 secs] 14971K->7272K(19456K), 0.1351873 secs] [Times: user=0.00 sys=0.00, real=0.13 secs] 
[Full GC (System.gc()) [Tenured: 6777K->7059K(10240K), 0.0075978 secs] 13027K->7059K(19456K), [Metaspace: 9186K->9186K(1058816K)], 0.3133278 secs] [Times: user=0.01 sys=0.00, real=0.31 secs] 
```

![image-20191215104636489](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191215104636489.png)

因为默认情况下只有创建对象的时候才会可能出现垃圾回收的操作，但是调用 `System.gc()`，会告诉 JVM 需要进行垃圾回收，JVM 会自行决定什么时候进行垃圾回收，同时可能在没有创建对象情况下执行垃圾回收；

同时可以使用 jmc 查看运行结果，可以看出 Eden 空间大小变化情况；

`jps -l` 查看当前进程对应的进程编号

`jcmd 进程号 VM.flags` 查看运行参数

#### 新生代到老年代晋升

`-verbose:gc -Xms20M  -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5   -XX:+PrintTenuringDistribution`

- 其中：`-XX:MaxTenuringThreshold=5` ：在可以自动调节对象晋升（Promote）到老年代阈值的 GC 中，设置该阈值的最大值；默认情况下新生代中对象经过一次 GC 对应的年龄就 + 1，这里当年龄  >5 的时候该对象就晋升到老年代。这只是一个最大值，但是可能没有到达该阈值 JVM 也会将其晋升到老年代。

    并且该参数的默认值为：15，其中在 CMS 中默认值为：6，在 G1 中默认值为：15，因为在 JVM 中该数值由 4 个 bit 来标识，所以最大值为 1111，即为 15；

- 经历过多次 GC 之后，新生代中存活的对象会在 From Survivor 和 To Survivor 之间来回存放，而前提是这两个空间有足够的的大小来存放这些数据，在 GC 算法中会计算每个对象年龄的大小，如果到达某个年龄后发现该年龄的对象总大小已经大于 Survivor（其中一个 Survivor） 空间的 50%，这个时候就需要调整阈值，不能在继续等到默认的 15 次 GC 之后才完成晋升，因为会导致 Survivor 空间不足，所有需要调整阈值，让这些存活的对象尽快完成晋升来释放 Survivor 空间。

示例代码：

```java
package com.gjxaiou.gc;

/**
 * @Author GJXAIOU
 * @Date 2019/12/15 12:43
 */
public class MyTest3 {
    public static void main(String[] args) {
        int size = 1024 * 1024;
        byte[] myAlloc1 = new byte[2 * size];
        byte[] myAlloc2 = new byte[2 * size];
        byte[] myAlloc3 = new byte[2 * size];
        byte[] myAlloc4 = new byte[2 * size];
        System.out.println("hello world");
    }
}
```

结果显示：

```java
-XX:InitialHeapSize=20971520 -XX:InitialTenuringThreshold=5 -XX:MaxHeapSize=20971520 -XX:MaxNewSize=10485760 -XX:MaxTenuringThreshold=5 -XX:NewSize=10485760 -XX:+PrintCommandLineFlags -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintTenuringDistribution -XX:SurvivorRatio=8 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
[GC (Allocation Failure) 
 // new threshold 5 是动态计算的阈值，该值 <= 后面设置的最大值 5
 // 所需 Survivor 空间为 1048576/1024/1024 = 1M,和设置的一样
Desired survivor size 1048576 bytes, new threshold 5 (max 5)
[PSYoungGen: 7799K->808K(9216K)] 7799K->6960K(19456K), 0.0028644 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 808K->0K(9216K)] [ParOldGen: 6152K->6754K(10240K)] 6960K->6754K(19456K), [Metaspace: 3104K->3104K(1056768K)], 0.0049277 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
hello world
Heap
 PSYoungGen      total 9216K, used 2372K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
  eden space 8192K, 28% used [0x00000000ff600000,0x00000000ff851200,0x00000000ffe00000)
  from space 1024K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x00000000fff00000)
  to   space 1024K, 0% used [0x00000000fff00000,0x00000000fff00000,0x0000000100000000)
 ParOldGen       total 10240K, used 6754K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  object space 10240K, 65% used [0x00000000fec00000,0x00000000ff298bd0,0x00000000ff600000)
 Metaspace       used 3126K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 338K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

#### 动态阈值设置原理

默认情况下：Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或者等于该年龄的对象就可以直接进入老年代，无须等到 MaxTenuringThreshold 中要求的年龄。

综合测试代码

`-verbose:gc -Xmx200M -Xmn50M -XX:TargetSurvivorRatio=60 -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:MaxTenuringThreshold=3 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC`

- `-XX:TargetSurvivorRatio=60`表示当一个 Survivor 空间中存活的对象占据了 60% 的空间，就会重新计算晋升的阈值（不在使用配置或者默认的阈值）

```java
package com.gjxaiou.gc;

/**
 * @Author GJXAIOU
 * @Date 2019/12/15 13:27
 */
public class MyTest4 {
    public static void main(String[] args) throws InterruptedException {
        // 下面两个字节数组在 main() 方法中，不会被GC
        byte[] byte1 = new byte[512 * 1024];
        byte[] byte2 = new byte[512 * 1024];

        myGC();
        Thread.sleep(1000);
        System.out.println("----111111111------");
        myGC();
        Thread.sleep(1000);
        System.out.println("----22222222------");
        myGC();
        Thread.sleep(1000);
        System.out.println("----333333333------");
        myGC();
        Thread.sleep(1000);
        System.out.println("----444444444------");

        byte[] byte3 = new byte[1024 * 1024];
        byte[] byte4 = new byte[1024 * 1024];
        byte[] byte5 = new byte[1024 * 1024];
        myGC();
        Thread.sleep(1000);
        System.out.println("----555555555------");
        myGC();
        Thread.sleep(1000);
        System.out.println("----666666666------");

        System.out.println("hello world");

    }

    // 方法中定义的变量当方法执行完成之后生命周期就结束了，下次垃圾回收时候就可以回收了
    private static void myGC() {
        for (int i = 0; i < 40; i++) {
            byte[] byteArray = new byte[1024 * 1024];
        }
    }
}

```

程序运行结果为：

```java
2019-12-15T14:18:18.013+0800: [GC (Allocation Failure) 2019-12-15T14:18:18.022+0800: [ParNew
Desired survivor size 3145728 bytes, new threshold 3 (max 3)
- age   1:    1712592 bytes,    1712592 total
: 40346K->1706K(46080K), 0.0091928 secs] 40346K->1706K(199680K), 0.0186267 secs] [Times: user=0.00 sys=0.00, real=0.02 secs] 
----111111111------
2019-12-15T14:18:19.032+0800: [GC (Allocation Failure) 2019-12-15T14:18:19.032+0800: [ParNew
// 3145728（3M），因为默认 8:1:1，即 Survivor 空间为 5M，对应的 60% 即为 3M；                                                                         
Desired survivor size 3145728 bytes, new threshold 3 (max 3)
- age   1:     342632 bytes,     342632 total
- age   2:    1762376 bytes,    2105008 total
: 41847K->2413K(46080K), 0.0007192 secs] 41847K->2413K(199680K), 0.0007452 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----22222222------
2019-12-15T14:18:20.034+0800: [GC (Allocation Failure) 2019-12-15T14:18:20.034+0800: [ParNew
Desired survivor size 3145728 bytes, new threshold 3 (max 3)
- age   1:         80 bytes,         80 total
- age   2:     342096 bytes,     342176 total
- age   3:    1761160 bytes,    2103336 total
: 42927K->2424K(46080K), 0.0006154 secs] 42927K->2424K(199680K), 0.0006435 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
----333333333------
2019-12-15T14:18:21.037+0800: [GC (Allocation Failure) 2019-12-15T14:18:21.037+0800: [ParNew
                                                                                      // 上面 age = 3 的这里垃圾回收之后变成 4 晋升为老年代了
Desired survivor size 3145728 bytes, new threshold 3 (max 3)
- age   1:         80 bytes,         80 total
- age   2:         80 bytes,        160 total
- age   3:     341992 bytes,     342152 total
: 43144K->1050K(46080K), 0.0017927 secs] 43144K->2738K(199680K), 0.0018190 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----444444444------
2019-12-15T14:18:22.040+0800: [GC (Allocation Failure) 2019-12-15T14:18:22.040+0800: [ParNew
                                                                                      // 这里阈值变成了 1，因为对象空间超过 Survivor 空间的 60% 即为 3M，重新计算了阈值，计算公式为取当前年龄和 MaxThreshold 的最小值，因为新创建数组，当前年龄为 1，所以最终为 1；
Desired survivor size 3145728 bytes, new threshold 1 (max 3)
- age   1:    3145856 bytes,    3145856 total
- age   2:         80 bytes,    3145936 total
- age   3:         80 bytes,    3146016 total
: 41777K->3128K(46080K), 0.0009780 secs] 43465K->5151K(199680K), 0.0010024 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----555555555------
2019-12-15T14:18:23.042+0800: [GC (Allocation Failure) 2019-12-15T14:18:23.042+0800: [ParNew
                                                                                      // 上面的 age 为 1,2,3 的经过一次 GC 之后全部晋升到老年代了，下面是新加入新生代的对象
Desired survivor size 3145728 bytes, new threshold 3 (max 3)
- age   1:         80 bytes,         80 total
: 43859K->14K(46080K), 0.0011804 secs] 45882K->5109K(199680K), 0.0012060 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----666666666------
hello world
Heap
 par new generation   total 46080K, used 18015K [0x00000000f3800000, 0x00000000f6a00000, 0x00000000f6a00000)
  eden space 40960K,  43% used [0x00000000f3800000, 0x00000000f49946a8, 0x00000000f6000000)
  from space 5120K,   0% used [0x00000000f6000000, 0x00000000f6003840, 0x00000000f6500000)
  to   space 5120K,   0% used [0x00000000f6500000, 0x00000000f6500000, 0x00000000f6a00000)
 concurrent mark-sweep generation total 153600K, used 5095K [0x00000000f6a00000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 3735K, capacity 4536K, committed 4864K, reserved 1056768K
  class space    used 410K, capacity 428K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

#### 空间分配担保

在发生 Minor GC 之前，虚拟机会检查**老年代最大可用的连续空间**是否大于**新生代所有对象的总空间**，

- 如果大于，则此次 **Minor GC 是安全的**（因为新生代是复制算法，如果 Survivor 中空间不够存放存活对象，会直接晋升到老年代）。==全部晋升还是部分晋升？==
- 如果小于，则虚拟机会查看 **HandlePromotionFailure** 设置值是否允许担保失败。
    如果 HandlePromotionFailure=true，那么会继续检查老年代最大可用连续空间是否大于**历次晋升到老年代的对象的平均大小**，如果大于，则尝试进行一次 Minor GC，但这次 Minor GC 依然是有风险的；如果小于或者 HandlePromotionFailure=false，则改为进行一次 Full GC。

上面提到了 Minor GC 依然会有风险，是因为新生代采用**复制收集算法**，假如大量对象在 Minor GC 后仍然存活（最极端情况为内存回收后新生代中所有对象均存活），而 Survivor 空间是比较小的，这时就需要老年代进行分配担保，把 Survivor 无法容纳的对象放到老年代。**老年代要进行空间分配担保，前提是老年代得有足够空间来容纳这些对象**，但一共有多少对象在内存回收后存活下来是不可预知的，**因此只好取之前每次垃圾回收后晋升到老年代的对象大小的平均值作为参考**。使用这个平均值与老年代剩余空间进行比较，来决定是否进行 Full GC 来让老年代腾出更多空间。

取平均值仍然是一种**概率性的事件**，如果某次 Minor GC 后存活对象陡增，远高于平均值的话，必然导致担保失败，如果出现了分配担保失败，**就只能在失败后重新发起一次 Full GC**。虽然存在发生这种情况的概率，但**大部分时候都是能够成功分配担保**的，这样就避免了过于频繁执行 Full GC。

**1.6 之后：只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行 Minor GC，否则进行 Full GC**， HandlePromotionFailure 不在有用。

##  九、CMS 垃圾收集器（Concurrent Mark Sweep）

CMS 垃圾收集器属于老年代的收集器

### CMS 垃圾回收器

CMS 收集器是一种以**获取最短回收停顿时间为目标的收集器**。目前很大一部分的 Java 应用集中在互联网网站或者 B/S 系统的服务端上，这类应用尤其**重视服务的响应速度**，希望系统停顿时间最短，以给用户带来较好的体验。CMS 收集器就非常符合这类应用的需求。CMS 收集器是**基于「标记－清除」算法**实现的，整个过程分为４个步骤，包括：

- **初始标记**（CMS initial mark）：初始标记仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快；

- **并发标记**（CMS concurrent mark）：该阶段就是进行 GC Roots Tracing 的过程；==Tracing 过程是什么？==

- **重新标记**（CMS remark）：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短；

- **并发清除**（CMS concurrent sweep）

其中，**初始标记、重新标记这两个步骤仍然需要 「Stop The World」**。由于整个过程中**耗时最长的并发标记和并发清除过程收集器收集线程都可以与用户线程一起工作**，所以，从总体上来说，CMS 收集器的内存回收过程是与用户线程一起并发执行的。 

![426b97e3848](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/426b97e3848.png)

- 优点：并发收集、低停顿；

- CMS 收集器有３个明显的缺点：
    - **CMS 收集器对 CPU 资源非常敏感**。在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。默认启动的回收线程数为：（CPU 数量 + 3）/ 4，CPU 数量=4，回收线程占用 25 % 左右的 CPU 资源，CPU 数量越多占用率越低。数量很小使用可以采用增量式并发收集器 i-CMS（Incremental Concurrent Mark Sweep），即在并发标记和清理的时候让 GC 线程和用户线程交替运行，减少独占，但是手机收集时间变长了，不建议使用。
    
    - **CMS 收集器无法处理浮动垃圾，可能出现「Concurrent Mode Failure」失败而导致另一次 Full GC 的产生**。由于 CMS 并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS 无法在当次收集中处理掉它们，只好留待下一次 GC 时再清理掉。这一部分垃圾就称为「浮动垃圾」。
        
        因为垃圾收集阶段用户线程也在运行，就会不断产生垃圾，所以得预留一部分空间给用户线程使用，则不能等到老年代几乎全部被填满之后才进行垃圾回收， 1.6 之后当老年代使用 92%，CMS 就启动了，该值可以通过参数：`-XX:CMSInitiatingOccupancyFraction` 指定，该百分值太小则 GC 过于频繁，太大会导致预留内存无法满足程序需要，出现「Concurrent Mode Failure」，这时候只能采用 Serial Old 收集器来进行老年代垃圾收集，更加浪费时间。
        
    - 空间碎片：CMS 是一款基于标记-清除算法实现的收集器，所有会有空间碎片的现象，当空间碎片过多时，将会给大对象分配带来很大麻烦，往往会出现老年代还有很大空间剩余，但是无法找到足够大的连续空间来分配当前对象，不得不提前触发一次 Full GC。
        - 解决方案：通过参数：`-XX:+UseCMSCompactAtFullCollection` 开关参数（默认开启），用于在 CMS 收集器顶不住要进行 Full GC 时候开启内存碎片合并整理过程，该过程无法并发停顿时间较长。
        - 补充参数：`-XX+CMSFullGCsBeforeCompaction` 用于设置执行多少次不压缩的 Full GC 之后，跟着来一次带压缩的。默认值为 0 ，表示每次进入 Full GC 都进行碎片整理。

###  CMS 处理过程有七个步骤：

- 初始标记（CMS-initial-mark）,会导致 SWT；
- 并发标记（CMS-concurrent-mark），与用户线程同时运行；
- 预清理（CMS-concurrent-preclean），与用户线程同时运行；
- 可被终止的预清理（CMS-concurrent-abortable-preclean） 与用户线程同时运行；
- 重新标记（CMS-remark），会导致 SWT；
- 并发清除（CMS-concurrent-sweep），与用户线程同时运行；
- 并发重置状态等待下次 CMS 的触发（CMS-concurrent-reset），与用户线程同时运行； 

#### 步骤一：初始标记

这是 CMS 中两次 stop-the-world 事件中的一次。这一步的作用是标记存活的对象，有两部分：

- 标记老年代中所有的 GC Roots 对象（即直接被 GC Root 引用的对象），如下图节点 1；

- 标记年轻代中活着的对象引用到的老年代的对象（指的是年轻带中还存活的引用类型对象，引用指向老年代中的对象）如下图节点 2、3；

 ![img](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/20170502172953141.png) 

在 Java 语言里，可作为 GC Roots 对象的包括如下几种：

- 虚拟机栈（栈桢中的本地变量表）中的引用的对象 ；
- 方法区中的类静态属性引用的对象 ；
- 方法区中的常量引用的对象 ；
- 本地方法栈中JNI的引用的对象； 

**ps：为了加快此阶段处理速度，减少停顿时间，可以开启初始标记并行化，`-XX:+CMSParallelInitialMarkEnabled`，同时调大并行标记的线程数，线程数不要超过 CPU 的核数；**

### 阶段二：并发标记（Concurrent Mark）

在这个阶段垃圾收集器会遍历老年代，然后标记所有存活的对象，它会根据上个阶段找到的 GC Roots 遍历查找。并发标记阶段，它会与用户的应用程序并发运行。并不是老年代的所有存活的对象都会被标记，因为在标记期间用户的程序可能会改变一些引用。（例如结点 3 下面结点的引用发生了改变）

从「初始标记」阶段标记的对象开始找出所有存活的对象;

因为是并发运行的，在运行期间会发生新生代的对象晋升到老年代、或者是直接在老年代分配对象、或者更新老年代对象的引用关系等等，对于这些对象，都是需要进行重新标记的，否则有些对象就会被遗漏，发生漏标的情况。为了提高重新标记的效率，该阶段会把上述对象所在的 Card 标识为 Dirty，后续只需扫描这些 Dirty Card 的对象，避免扫描整个老年代；

并发标记阶段只负责将引用发生改变的 Card 标记为 Dirty 状态，不负责处理；

如下图所示，也就是节点1、2、3，最终找到了节点 4 和 5。并发标记的特点是和应用程序线程同时运行。并不是老年代的所有存活对象都会被标记，因为标记的同时应用程序会改变一些对象的引用等。

![并发标记](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/20170502175211859.png)

这个阶段因为是并发的容易导致 concurrent mode failure

### 阶段三：并发预清理阶段

- 这也是一个并发阶段，与应用的线程并发运行，并不会 Stop 应用的线程，在并发运行的过程中，一些对象的引用可能会发生改变，但是这种情况发生时， JVM 会将包含这个对象的区域（Card）标记为 Dirty，这就是 Card marking

- 在 Pre-clean 阶段，那些能够从 Dirty 对象到达的对象也会被标记，这个标记做完之后， Dirty Card 标记就会被清除了。

前一个阶段已经说明，不能标记出老年代全部的存活对象，是因为标记的同时应用程序会改变一些对象引用，这个阶段就是用来处理前一个阶段因为引用关系改变导致没有标记到的存活对象的，它会扫描所有标记为 Direty 的 Card。

如下图所示，在并发清理阶段，节点 3 的引用指向了 6；则会把节点 3 的 card 标记为 Dirty；
![这里写图片描述](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/20170502211600103.png)

最后将 6 标记为存活,如下图所示：

![这里写图片描述](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/20170502211950472.png)

### 阶段四：可终止的预处理

这个阶段尝试着去承担下一个阶段 Final Remark 阶段足够多的工作。这个阶段持续的时间依赖好多的因素，由于这个阶段是重复的做相同的事情直到发生 aboart 的条件（比如：重复的次数、多少量的工作、持续的时间等等）之一才会停止。
**ps:此阶段最大持续时间为5秒，之所以可以持续5秒，另外一个原因也是为了期待这5秒内能够发生一次ygc，清理年轻带的引用，是的下个阶段的重新标记阶段，扫描年轻带指向老年代的引用的时间减少；**

### 阶段五：重新标记

这个阶段会导致第二次 stop the word，**该阶段的任务是完成标记整个年老代的所有的存活对象**。
这个阶段，重新标记的内存范围是整个堆，包含 young_gen 和 old_gen。为什么要扫描新生代呢，因为对于老年代中的对象，如果被新生代中的对象引用，那么就会被视为存活对象，即使新生代的对象已经不可达了，也会使用这些不可达的对象当做 CMS 的「GC Root」，来扫描老年代； 因此对于老年代来说，引用了老年代中对象的新生代的对象，也会被老年代视作「GC ROOTS」:当此阶段耗时较长的时候，可以加入参数 `-XX:+CMSScavengeBeforeRemark`，在重新标记之前，先执行一次 ygc，回收掉年轻带的对象无用的对象，并将对象放入幸存带或晋升到老年代，这样再进行年轻代扫描时，只需要扫描幸存区的对象即可，一般幸存带非常小，这大大减少了扫描时间.

由于之前的预处理阶段是与用户线程并发执行的，这时候可能年轻带的对象对老年代的引用已经发生了很多改变，这个时候，remark 阶段要花很多时间处理这些改变，会导致很长 stop the word，所以通常 CMS 尽量运行 Final Remark 阶段在年轻代是足够干净的时候，是为了减少连续 STW 发生的可能性（年轻代存活对象过多的话，也会导致老年代涉及的存活对象会很多）。

**另外，还可以开启并行收集：`-XX:+CMSParallelRemarkEnabled`**

**至此，老年代所有存活的对象都被标记过了，现在可以通过清除算法去清理老年代不再使用的对象**。

### 阶段六：并发清理

通过以上 5 个阶段的标记，老年代所有存活的对象已经被标记并且现在要通过 Garbage Collector 采用清扫的方式回收那些不能用的对象了。
这个阶段主要是清除那些没有标记的对象并且回收空间；

由于 CMS 并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS 无法在当次收集中处理掉它们，只好留待下一次GC时再清理掉。这一部分垃圾就称为「浮动垃圾」。

![image-20191219091858884](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191219091858884.png)

### 阶段七：并发重置

这个阶段并发执行，重新设置 CMS 算法内部的数据结构，准备下一个 CMS 生命周期的使用。

### CMS 总结

- CMS 通过将大量工作分散到并发处理阶段来减少 STW 时间；
- CMS 缺点：
  - CMS 收集器无法处理浮动垃圾（Floating Garbage），可能出现 Concurrent Mode Failure 失败从而导致另一次的 Full GC 的产生，可能引发串行 Full GC。
  - 空间碎片导致无法分配大对象，CMS 收集器提供了一个 `-XX+UseCMSCompaceAtFullCollection` 开关参数（默认开启），用于在 CMS 收集器顶不住要进行 Full GC 时候开启内存碎片的合并整理过程，内存整理过程是无法并发的，解决了空间碎片问题但是增加了停顿时间；
  - 对于堆比较大的应用， GC 的时间难以预估。

**针对上面步骤的代码验证**

设置虚拟机参数为：`-verbose:gc -Xms20M -Xmx20M -Xmn10M -XX:+PrintGCDetails -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC` 因为 CMS 只能运行到老年代，对应的新生代会自动采用与 CMS 对应的垃圾回收器

程序为：

```java
package com.gjxaiou.gc;

/**
 * @Author GJXAIOU
 * @Date 2019/12/18 13:19
 */
public class MyTest5 {
    public static void main(String[] args) {
        int size = 1024 * 1024;
        byte[] myAlloc1 = new byte[4 * size];
        System.out.println("----111111111----");
        byte[] myAlloc2 = new byte[4 * size];
        System.out.println("----222222222----");
        byte[] myAlloc3 = new byte[4 * size];
        System.out.println("----333333333----");
        byte[] myAlloc4 = new byte[2 * size];
        System.out.println("----444444444----");
    }
}

```

输出结果：

```java
// 前面没有执行任何的垃圾回收，因为 Eden 区域放置 4M 对象可以放下
----111111111----
// 因为第二次 new 又需要分配 4M 空间，Eden 空间不够用，使用垃圾回收，对应新生代是 ParNew 收集器
[GC (Allocation Failure) [ParNew: 5899K->670K(9216K), 0.0016290 secs] 5899K->4768K(19456K), 0.0016630 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----222222222----
    // 新生代垃圾回收
[GC (Allocation Failure) [ParNew: 5007K->342K(9216K), 0.0023932 secs] 9105K->9168K(19456K), 0.0024093 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
// 老年代垃圾回收    老年代存活对象占用空间大小（老年代总的空间大小）
[GC (CMS Initial Mark) [1 CMS-initial-mark: 8825K(10240K)] 13319K(19456K), 0.0003398 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-mark-start]
----333333333----
----444444444----
Heap
 par new generation   total 9216K, used 6780K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  78% used [0x00000000fec00000, 0x00000000ff2499d0, 0x00000000ff400000)
  from space 1024K,  33% used [0x00000000ff400000, 0x00000000ff455a08, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 concurrent mark-sweep generation total 10240K, used 8825K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 3144K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 343K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

另一端代码，将上面代码中的 `byte[] myAlloc4 = new byte[2 * size];` 修改为：`byte[] myAlloc4 = new byte[3 * size];`得到的结果如下：

```java
----111111111----
[GC (Allocation Failure) [ParNew: 5765K->637K(9216K), 0.0024098 secs] 5765K->4735K(19456K), 0.0024726 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----222222222----
[GC (Allocation Failure) [ParNew: 4974K->240K(9216K), 0.0041475 secs] 9072K->9060K(19456K), 0.0041812 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
----333333333----
----444444444----
[GC (CMS Initial Mark) [1 CMS-initial-mark: 8819K(10240K)] 16522K(19456K), 0.0002890 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-mark-start]
Heap
 par new generation   total 9216K, used 7764K [0x00000000fec00000, 0x00000000ff600000, 0x00000000ff600000)
  eden space 8192K,  91% used[CMS-concurrent-mark: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[CMS-concurrent-abortable-preclean-start]
[CMS-concurrent-abortable-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
 [0x00000000fec00000, 0x00000000ff358e70, 0x00000000ff400000)
  from space 1024K,  23% used [0x00000000ff400000, 0x00000000ff43c2d0, 0x00000000ff500000)
  to   space 1024K,   0% used [0x00000000ff500000, 0x00000000ff500000, 0x00000000ff600000)
 concurrent mark-sweep generation total 10240K, used 8819K [0x00000000ff600000, 0x0000000100000000, 0x0000000100000000)
 Metaspace       used 3126K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 338K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```

下面抓取一下 gc 信息，来进行详细分析，首先将  jvm 中加入以下运行参数：

```java
-XX:+PrintCommandLineFlags                  [0]
-XX:+UseConcMarkSweepGC                     [1]   
-XX:+UseCMSInitiatingOccupancyOnly          [2]
-XX:CMSInitiatingOccupancyFraction=80       [3]
-XX:+CMSClassUnloadingEnabled               [4]
-XX:+UseParNewGC                            [5]
-XX:+CMSParallelRemarkEnabled               [6]
-XX:+CMSScavengeBeforeRemark                [7]
-XX:+UseCMSCompactAtFullCollection          [8]
-XX:CMSFullGCsBeforeCompaction=0            [9]
-XX:+CMSConcurrentMTEnabled                 [10]
-XX:ConcGCThreads=4                         [11] 
-XX:+ExplicitGCInvokesConcurrent            [12]
-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses    [13]
-XX:+CMSParallelInitialMarkEnabled          [14]

-XX:+PrintGCDetails                         [15]
-XX:+PrintGCCause                           [16]
-XX:+PrintGCTimeStamps                      [17]
-XX:+PrintGCDateStamps                      [18]
-Xloggc:../logs/gc.log                      [19]
-XX:+HeapDumpOnOutOfMemoryError             [20]
-XX:HeapDumpPath=../dump                    [21]

```

先来介绍下下面几个参数的作用：
\0. [0]打印出启动参数行
\1. [1]参数指定使用CMS垃圾回收器；
\2. [2]、[3]参数指定CMS垃圾回收器在老年代达到80%的时候开始工作，如果不指定那么默认的值为92%；
\3. [4]开启永久带（jdk1.8以下版本）或元数据区（jdk1.8及其以上版本）收集，如果没有设置这个标志，一旦永久代或元数据区耗尽空间也会尝试进行垃圾回收，但是收集不会是并行的，而再一次进行Full GC；
\4. [5] 使用cms时默认这个参数就是打开的，不需要配置，cms只回收老年代，年轻带只能配合Parallel New或Serial回收器；
\5. [6] 减少Remark阶段暂停的时间，启用并行Remark，如果Remark阶段暂停时间长，可以启用这个参数
\6. [7] 如果Remark阶段暂停时间太长，可以启用这个参数，在Remark执行之前，先做一次ygc。因为这个阶段，年轻带也是cms的gcroot，cms会扫描年轻带指向老年代对象的引用，如果年轻带有大量引用需要被扫描，会让Remark阶段耗时增加；
\7. [8]、[9]两个参数是针对cms垃圾回收器碎片做优化的，CMS是不会移动内存的， 运行时间长了，会产生很多内存碎片， 导致没有一段连续区域可以存放大对象，出现”promotion failed”、”concurrent mode failure”, 导致fullgc，启用UseCMSCompactAtFullCollection 在FULL GC的时候， 对年老代的内存进行压缩。-XX:CMSFullGCsBeforeCompaction=0 则是代表多少次FGC后对老年代做压缩操作，默认值为0，代表每次都压缩, 把对象移动到内存的最左边，可能会影响性能,但是可以消除碎片；
106.641: [GC 106.641: [ParNew (promotion failed): 14784K->14784K(14784K), 0.0370328 secs]106.678: [CMS106.715: [CMS-concurrent-mark: 0.065/0.103 secs] [Times: user=0.17 sys=0.00, real=0.11 secs]
(concurrent mode failure): 41568K->27787K(49152K), 0.2128504 secs] 52402K->27787K(63936K), [CMS Perm : 2086K->2086K(12288K)], 0.2499776 secs] [Times: user=0.28 sys=0.00, real=0.25 secs]
\8. [11]定义并发CMS过程运行时的线程数。比如value=4意味着CMS周期的所有阶段都以4个线程来执行。尽管更多的线程会加快并发CMS过程，但其也会带来额外的同步开销。因此，对于特定的应用程序，应该通过测试来判断增加CMS线程数是否真的能够带来性能的提升。如果未设置这个参数，JVM会根据并行收集器中的-XX:ParallelGCThreads参数的值来计算出默认的并行CMS线程数：
ParallelGCThreads = (ncpus <=8 ? ncpus : 8+(ncpus-8)*5/8) ，ncpus为cpu个数，
ConcGCThreads =(ParallelGCThreads + 3)/4
这个参数一般不要自己设置，使用默认就好，除非发现默认的参数有调整的必要；
\9. [12]、[13]开启foreground CMS GC，CMS gc 有两种模式，background和foreground，正常的cms gc使用background模式，就是我们平时说的cms gc；当并发收集失败或者调用了System.gc()的时候，就会导致一次full gc，这个fullgc是不是cms回收，而是Serial单线程回收器，加入了参数[12]后，执行full gc的时候，就变成了CMS foreground gc，它是并行full gc，只会执行cms中stop the world阶段的操作，效率比单线程Serial full GC要高；需要注意的是它只会回收old，因为cms收集器是老年代收集器；而正常的Serial收集是包含整个堆的，加入了参数[13],代表永久带也会被cms收集；
\10. [14] 开启初始标记过程中的并行化，进一步提升初始化标记效率;
\11. [15]、[16]、[17]、[18] 、[19]是打印gc日志，其中[16]在jdk1.8之后无需设置
\12. [20]、[21]则是内存溢出时dump堆

## 十、G1 收集器（Garbage First Collector）

### （一）评价系统的指标

**吞吐量：**

- 吞吐量关注的是，在一个指定的时间内，最大化一个应用的工作量。
- 如下方式来衡量一个系统吞吐量的好坏：
    1、在一个小时内同一个事务（或者任务、请求）完成的次数（tps，实际中还会经常见qps，每秒查询率QPS是对一个特定的查询服务器在规定时间内所处理流量多少的衡量标准）。
    2、数据库一小时可以完成多少次查询。
- 对于关注吞吐量的系统，卡顿是可以接受的，因为这个系统关注长时间的大量任务的执行能力，单次快速的响应并不值得考虑。

**响应能力：**

- 响应能力指一个程序或者系统对请求是否能够及时响应，比如：
    1、一个桌面UI能多快地响应一个事件。
    2、一个网站能够多快返回一个页面请求。
    3、数据库能够多快返回查询的数据。
- 对于这类对响应能力敏感的场景，长时间的停顿是无法接受的。

以上是用来评价一个系统的两个很重要的指标，介绍这两个指标的原因是因为G1就是用来解决这样的问题而应运而生的。

### （二）理论

- g1 收集器是一个**面向服务端**的垃圾收集器，适用于多核处理器、大内存容量的服务端系统。
- 它满足短时间 gc 停顿的同时达到一个较高的吞吐量。
- JDK1.7 以上版本适用【通过配置JVM的参数来指定既可】。

以上可以看到G1在吞吐量和响应能力上都进行了兼顾。



### （三）特点

- 并行与并发：使用多个 CPU 来缩短 STW 停顿的时间；同时可以并发操作；
- 分代收集：使用不同方式处理不同代对象；
- 空间整合：整体基于 标记 - 整理 算法，局部（两个 Region 之间）基于 复制 算法；即**没有内存碎片**
- 可预测的停顿：能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集上的时间不得超过 N 毫秒。

### （三）G1 收集器的设计目标：

- 与应用线程同时工作，几乎【注意措辞】不需要stop the world(与CMS类似)；
- 整理剩余空间，不产生内存碎片（CMS只能在Full GC时，用stop the world整理内存碎片）。
- GC停顿更加可控；【对于CMS来说如果出现了Full GC时，则会对新生代和老年代的堆内存进行完整的整理，停顿时间就不可控了】G1 可以回收部分老年代，剩下的可以在下次 GC 时候再清理；
- 不牺牲系统的吞吐量；
- gc不要求额外的内存空间（CMS需要预留空间存储浮动垃圾【这个在学习CMS中已经阐述过了，其实就是CMS回收的过程跟用户线程是并发进行的，所在在标记或者清除的同时对象的引用还会被改变，使得原来对象本来不是垃圾，当CMS清理时该对象已经变成了垃圾了，但是CMS认为它还不是垃圾，所以该对象的清除工作就会放到下一次了，所以将这种对象则称之为浮动垃圾】）

### （四） G1 的设计规划是要替换掉 CMS

- G1 在某些方面弥补了 CMS 的不足，比如 CMS 使用的是 Mark-sweep 算法，自然会产生内存碎片；然而 G1 基于复制算法，高效的整理剩余内存，而不需要管理内部碎片；
- 同时 G1提供更多的手段来达到对 GC 停顿时间的可控；

### （五）Hotspot 虚拟机主要构成

![image-20191218163635270](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191218163635270.png)

### （六）传统垃圾收集器堆结构

![image-20191218163937612](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191218163937612.png)

###  （七）G1 堆结构

 

![G1](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/G1.png)

- **heap 被划分为一个个相等的不连续的内存区域(regions) ，每个 region 都有一个分代的角色: eden、 survivor、 old** ，新生代和老年代不是物理隔离，而是一部分 Region（不需要连续）的集合。
- 对每个角色的数量并没有强制的限定，也就是说对每种分代内存的大小，可以动态变化
- G1 最大的特点就是高效的执行回收，优先去执行那些大量对象可回收的区域(region)
- G1 使用了 gc 停顿可预测的模型，来满足用户设定的 gc 停顿时间，根据用户设定的目标时间，G1 会自动地选择哪些 region 要清除，一次清除多少个 region。（后台根据各个 Region 回收获取的空间大小和回收所需时间的经验值，存放在一个优先列表中）
- G1 从多个 region 中复制存活的对象，然后集中放入一个 region 中，同时整理、清除内存(copying 收集算法)
- 对比使用 mark-sweep 的 CMS, G1 使用的 copying 算法不会造成内存碎片;
- 对比Parallel Scavenge(基于copying )、Parallel Old 收集器(基于 mark-compact-sweep)，Parallel 会对整个区域做整理导致 gc 停顿会比较长，而 G1 只是特定地整理几个 region。
- G1 并非一个实时的收集器，与 parallelScavenge 一样，对 gc 停顿时间的设置并不绝对生效，只是 G1 有较高的几率保证不超过设定的 gc 停顿时间。与之前的 gc 收集器对比，G1 会根据用户设定的 gc 停顿时间，智能评估哪几个 region 需要被回收可以满足用户的设定

### （八）基本概念

#### 分区(Region):

- G1 采取了不同的策略来解决并行、串行和CMS收集器的碎片、暂停时间不可控等问题-------G1 将 **整个堆分成相同大小的分区**(Region)
- 每个分区都可能是年轻代也可能是老年代，但是在同一时刻只能属于某个代。年轻代、幸存区、老年代这些概念还存在，**成为逻辑上的概念**，这样方便复用之前分代框架的逻辑。
- 在物理上不需要连续，则带来了额外的好处-------有的分区内垃圾对象特别多，有的分区内垃圾对象很少，**G1会优先回收垃圾对象特别多的分区**，这样可以花费较少的时间来回收这些分区的垃圾，这也就是 G1 名字的由来，即首先收集垃圾最多的分区。
- 依然是在新生代满了的时候，对整个新生代进行回收-----------整个新生代中的对象，要么被回收、要么晋升，至于新生代也采取分区机制的原因，则是因为这样跟老年代的策略统一，方便调整代的大小
- G1还是一种带压缩的收集器，在回收老年代的分区时，是将存活的对象从一个分区拷贝到另一个可用分区，这个拷贝的过程就实现了局部的压缩。

#### 收集集合(CSet)

- 一组可被回收的分区的集合。在 CSet 中存活的数据会在 GC过程中被移动到另一个可用分区，CSet中的分区可以来自eden空间、survivor空间、 或者老年代。 Cset 在同一时刻可以拥有以上三种不同类型中的分区；

#### 已记忆集合(RSet：Remembered Set) :

- RSet 记录了其他 Region 中的对象引用本 Region 中对象的关系，属于 points-into 结构( 谁引用了我的区域中的对象)RSet 的价值在于使得垃圾收集器不需要扫描整个堆找到谁引用了当前分区中的对象，只需要扫描 RSet 即可。**每个 Region 都有一个对象的 RSet**。

    示例：Region1 和 Region3 中的对象都引用了 Region2中的对象，因此在 Region2 的 RSet 中记录了这两个引用。

[![img](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/region.png)](https://github.com/weolwo/jvm-learn/blob/master/src/resources/images/region.png)

- G1 GC 是在 points-out 的 card table 之上再加了一层结构来构成 points-into RSet：每个 region 会记录下到底哪些别的 region 有指向自己的指针，而这些指针分别在哪些 card 的范围内。
- 这个 RSet 其实是一个 hash table，key 是别的 region 的起始地址，value 是一个集合，里面的元素是 card table 的index. 举例来说，如果 region A 的 RSet 里有一项的 key 是 region B，value 里有 index 为 1234 的 card,它的意思就是 region B 的 一个 card 里有引用指向 region A。所以对 region  A 来说，该 RSet 记录的是 points-into 的关系;而 card table 仍然记录了 points-out 的关系。
- Snapshot-At-The-Beginning(SATB):SATB 是 G1 GC 在并发标记阶段使用的增量式的标记算法，
- 并发标记是并发多线程的，但并发线程在同一时刻只扫描一个分区

参考链接：https://www.oracle.com/technetwork/tutorials/tutorials-1876574.html

### G1 相对于 CMS 的优势

- G1 在压缩空间方面有优势（因为CMS 是标记清除算法，不带压缩的，造成内存碎片；G1 采用拷贝算法，没有内存碎片）
- G1 通过将内存空间分成区域(Region) 的方式避免内存碎片问题；
- Eden、Survivor、 Old 区不再固定，在内存使用效率上来说更灵活；
- G1 可以通过设置预期停顿时间( Pause Time) 来控制垃圾收集时间，避免应用雪崩现象
- G1 在回收内存后会马上同时做合并空闲内存的工作，而 CMS 默认是在 STW ( stop the world) 的时候做
- G1 会在 Young GC 中使用（也可以使用在 Old 区中），而 CMS 只能在 Old 区使用；

### G1 的适合场景

- 服务端多核 CPU、JVM 内存占用较大的应用
- 应用在运行过程中会产生大量内存碎片、需要经常压缩空间
- 想要更可控、可预期的 GC 停顿周期：防止高并发下应用的雪崩现象

### G1 GC模式

- G1 提供了两种 GC 模式，Young GC 和 Mixed GC, 两种都是完全 Stop The World 的
- Young GC：方式是选定所有年轻代里的 Region。通过控制年轻代的 Region 个数，即年轻代内存大小，来控制 Young GC 的时间开销。
- Mixed GC：选定所有年轻代里的Region，外加根据全局并发标记（global concurrent marking）统计得出收集收益高（垃圾更多）的若干老年代 Region。在用户指定的开销目标范围内尽可能选择收益高的老年代 Region。
- Mixed GC 不是 Full GC（G1 中没有 Full GC），它**只能回收部分老年代的 Region**，如果 Mixed GC 实在无法跟上程序分配内存的速度，导致老年代填满无法继续进行 Mixed GC，就会使用 Serial Old GC (里面有 Full GC)来收集整个 GC heap。 所以本质上，**G1 是不提供 Full GC 的**。

### 全局并发标记（global concurrent marking）

global concurrent marking 的执行过程类似于 CMS，但是不同的是在 G1 GC 中，它**主要是为 Mixed GC 提供标记服务的**（即表示应该回收哪些老年代），并不是一次 GC 过程的一个必须环节。

下面为全局并发标记执行过程

- **初始标记( initial mark, STW)** ：它标记了从 GC Root 开始直接可达的对象。并且修改 TAMS（Next Top at Mark Start）的值，让下一阶段用户程序并发执行时候（因为该阶段需要暂停用户线程），能在正确可用的 Region 中创建新对象。

- **并发标记( Concurrent Marking)** ：这个阶段从GC Root 开始对堆中的对象进行可达性分析标记，标记线程与应用程序线程并发执行，并且收集各个 Region 的存活对象信息。

- **重新标记( Remark, STW)** :标记那些在并发标记阶段发生变化的对象，记录在线程 RSet Logs 中，同时将 RSet Logs 中的数据合并到 RSet 中，将被回收。（该阶段需要停顿但可以并行）

- **清理(Cleanup)** :首先对各个 Region 中的回收价值和成本进行排序，根据用户期望的 GC 时间停顿时间来制定  回收计划，所以可能只清空一部分 Region。清除空 Region (没有存活对象的)，加入到 free list。

    

第一阶段 initial mark 是共用了 Young GC 的暂停，这是因为他们可以复用 rootscan 操作，所以可以说 global concurrent marking 是伴随 Young GC 而发生的；

第四阶段 Cleanup 只是回收了没有存活对象的 Region，所以它并不需要 STW。

### G1在运行过程中的主要模式

- YGC(不同于CMS)
    - G1 YGC 在 Eden 充满时触发，在回收之后所有之前属于 Eden 的区块全部变成空白，即不属于任何一个分区( Eden、Survivor、Old )
    - YGC执行步骤：
        - 阶段1:根扫描 静态和本地对象被描
        - 阶段2:更新RS 处理dirty card队列更新RS
        - 阶段3:处理RS 检测从年轻代指向老年代的对象
        - 阶段4:对象拷贝 拷贝存活的对象到survivor/old区域
        - 阶段5:处理引用队列 软引用，弱引用，虚引用处理
- 并发阶段（global concurrent marking）
- 混合模式
- Full GC (一 般是G1出现问题时发生，本质上不属于G1，G1进行的回退策略（回退为：Serial Old GC）)

### 什么时候发生 Mixed GC?

- 由一些参数控制，另外也控制着哪些老年代 Region 会被选入 CSet  (收集集合)，下面是一部分的参数
    - **G1HeapWastePercent**：在 global concurrent marking 结束之后，我们可以知道 old gen regions 中有多少空间要被回收，在每次 YGC 之后和再次发生 Mixed GC 之前（YGC  和 Mixed GC 之间是交替进行（不是一次一次交替，可能是多次对一次）的），会检查垃圾占比是否达到此参数，只有达到了，下次才会发生 Mixed GC；
    - **G1MixedGCLiveThresholdPercent**： old generation region 中的存活对象的占比，只有在此参数之下，才会被选入CSet
    - **G1MixedGCCountTarget**：一 次 global concurrent marking 之后，最多执行 Mixed GC 的次数
    - **G1OldCSetRegionThresholdPercent**：一次 Mixed GC 中能被选入 CSet 的最多 old generation region 数量
    - 除了以上的参数，G1 GC 相关的其他主要的参数有：
    
    | 参数                               | 含义                                                         |
    | :--------------------------------- | :----------------------------------------------------------- |
    | -XX:G1HeapRegionSize=n             | 设置 Region 大小，并非最终值                                 |
    | -XX:MaxGCPauseMillis               | 设置 G1 收集过程目标时间，默认值200 ms，不是硬性条件         |
    | -XX:G1NewSizePercent               | 新生代最小值，默认值 5%                                      |
    | -XX:G1MaxNewSizePercent            | 新生代最大值，默认值 60%                                     |
    | -XX:ParallelGCThreads              | STW 期间，并行 GC 线程数                                     |
    | -XX:ConcGCThreads=n                | 并发标记阶段，并行执行的线程数                               |
    | -XX:InitiatingHeapOccupancyPercent | 设置触发标记周期的 Java 堆占用率阈值。默认值是 45%。这里的 Java 堆占比指的是 non_young_capacity_bytes，包括old+humongous |

### G1 收集概览

- G1算法将堆划分为若干个区域( Region),它仍然属于分代收集器。不过,这些区域的一部分包含新生代,**新生代的垃圾收集依然采用暂停所有应用线程的方式**，将存活对象拷贝到老年代或者 Survivor 空间。老年代也分成很多区域，**G1收集器通过将对象从一个区域复制到另外一个区域,完成了清理工作**。这就意味着,在正常的处理过程中，G1完成了堆的压缩(至少是部分堆的压缩，因为复制本质上就包括压缩),这样也就不会有CMS内存碎片问题的存在

#### Humongous区域

在G1中,还有一种特殊的区域,叫 Humongous区域。如果一个对象占用的空间达到或是超过了分区容量50%以上,G1收集器就认为这是一个巨型对象。**这些巨型对象,默认直接会被分配在老年代**,但是如果它是一个短期存在的巨型对象就会对垃圾收集器造成负面影响。为了解决这个问题， G1 划分了一个 Humongous 区,它用来专门存放巨型对象。如果一个H 区装不下一个巨型对象,那么 G1 会寻找连续的H分区来存储。为了能找到连续的H区,有时候不得不启动 Full GC



### G1 Yong GC

- Young GC 主要是对Eden区进行GC，**它在Eden空间耗尽时会被触发**。在这种情况下Eden空间的数据移动到 Survivor空间中如果 Survivor 空间不够,Eden空间的部分数据会直接晋升到老年代空间。 Survivor区的数据移动到新的 Survivor 区中,也有部分数据晋升到老年代空间中。最终Eden空间的数据为空,GC完成工作,应用线程继续执行;
- 如果仅仅 GC 新生代对象,我们如何找到所有的根对象呢?老年代的所有对象都是根么?那这样扫描下来会耗费大量的时间。于是，G1引进了 RSet 的概念。它的全称是 Remembered set，作用是跟踪指向某个堆内的对象引用

![image-20191218205128114](JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191218205128114.png)

- 在 CMS 中,也有RSet的概念,**在老年代中有一块区域用来记录指向新生代的引用这是一种 point-out**,在进行 Young Go时扫描根时,仅仅需要扫描这一块区域,而不需要扫描整个老年代

- 但在 G1 中,并没有使用 point-out，这是由于一个分区太小,分区数量太多,如果是用 point-out 的话,会造成大量的扫描浪费,有些根本不需要 GC 的分区引用也扫描了。
- 于是 G1中使用 point-in 来解决。 point-in 的意思是哪些分区引用了当前分区中的对象。这样,仅仅将这些对象当做根来扫描就避免了无效的扫描。
- 由于新生代有多个，那么我们需要在新生代之间记录引用吗?这是不必要的，原因在于每次 GC 时所有新生代都会被扫描，**所以只需要记录老年代到新生代之间的引用即可**

- 需要注意的是，如果引用的对象很多，赋值器需要对每个引用做处理，赋值器开销会很大，为了解决赋值器开销这个问题，在 G1 中又引入了另外一个概念：卡表( Card table)。一个 Card table 将一个分区在逻辑上划分为固定大小的连续区域，每个区域称之为卡。卡通常较小，介于 128 到 512 字节之间。 Card Table通常为字节数组，由 Card 的索引(即数组下标)来标识每个分区的空间地址；

- 默认情况下，每个卡都未被引用，当一个地址空间被引用时候，这个地址空间对应的数组索引的值被标记为 0，即标记为脏被引用，此外 Rset 也将这个数组下标记录下来。一般情况下，**这个 Rset 其实是一个 HashTable，Key 是别的 Region 的起始地址，Value 是一个集合，里面的元素是 Card Table 的 Index**。

- G1 Young GC 过程

  - 阶段一：根扫描；

    静态和本地对象被扫描

  - 阶段二：更新 RS

    处理 Dirty Card 队列，更新 RS

  - 阶段三：处理 RS

    检测从年轻代指向老年代的对象

  - 阶段四：对象拷贝

    拷贝存活的对象到 Survivor/old 区域

  - 阶段五：处理引用队列

    软引用、弱引用、虚引用处理



#### 再谈 Mixed GC

- **Mixed GC 不仅进行正常的新生代垃圾收集，同时也回收部分后台扫描线程（即全局并发标记的线程）标记的老年代分区**；
- Mixed GC 步骤：
  - 全局并发标记（Global concurrent marking）
  - 拷贝存活对象（evacuation）

- 在 G1 GC 中， Global concurrent Marking 主要是为 Mixed GC 提供标记服务的，并不是一次 GC 过程中的一个必须环节。



### 三色标记算法

提到并发标记，我们不得不了解并发标记的三色标记算法。它是描述追踪式回收器的一种有效的方法，利用它可以推演回收器的正确性，**标记表示该对象是可达的，即不应该被当做垃圾回收**

- 我们将对象分成三种类型:
    - **黑色**：根对象，或者该对象与它的子对象（一个对象里面包含或者容纳的成员变量，因为一个对象或者类里可以引用其他的对象）都被扫描过(对象被标记了，且它的所有 field 也被标记完了)
    - **灰色**：对象本身被扫描，但还没扫描完该对象中的子对象( 它的 field 还没有被标记或标记完)
    - **白色**：未被扫描对象，扫描完成所有对象之后，最终为白色的为不可达对象，即垃圾对象(对象没有被标记到)

#### 示例：

遍历了所有可达的对象后，所有可达的对象都变成了黑色。不可达的对象即为白色，需要被清理,如图：

[<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/sanmark.gif" alt="三色标记算法" style="zoom: 50%;" />](https://github.com/weolwo/jvm-learn/blob/master/src/resources/images/sanmark.gif)



- 但是如果在标记过程中，应用程序也在运行，那么对象的指针就有可能改变。这样的话，我们就会遇到一个问题:对象丢失问题

<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/image-20191218210809306.png" alt="image-20191218210809306" style="zoom:50%;" />

这时候应用程序执行了以下操作: A.c=C B.c=null 这样，对象的状态图变成如下情形:

[<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/sans2.png" alt="img" style="zoom:50%;" />](https://github.com/weolwo/jvm-learn/blob/master/src/resources/images/sans2.png)

这时候垃圾收集器再标记扫描的时候就会变成下图这样

[<img src="JVM%20%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6.resource/sans1.png" alt="img" style="zoom:50%;" />

- **很显然，此时C是白色，被认为是垃圾需要清理掉，显然这是不合理的**

### SATB

- 在G1中，使用的是SATB ( Snapshot-At-The- Beginning)的方式，删除的时候记录所有的对象
- 它有3个步骤
    - 在开始标记的时候生成一个快照图，标记存活对象
    - 在并发标记的时候所有被改变的对象入队(在writebarrier里把所有旧的引用所指向的对象都变成非白的（例如上面的 C 颜色会变成灰色或者黑色，不会导致被回收）)
    - 可能存在浮动垃圾，将在下次被收集

### G1混合式回收

- G1到现在可以知道哪些老的分区可回收垃圾最多。当全局并发标记完成后，在某个时刻，就开始了Mixed GC。这些垃圾回收被称作“混合式”是因为他们不仅仅进行正常的新生代垃圾收集，同时也回收部分后台扫描线程标记的分区混合式GC也是采用的复制清理策略，当GC完成后，会重新释放空间



#### G1分代算法

为老年代设置分区的目的是老年代里有的分区垃圾多,有的分区垃圾少,这样在回收的时候可以专注于收集垃圾多的分区这也是G1名称的由来不过这个算法并不适合新生代垃圾收集，**因为新生代的垃圾收集算法是复制算法,但是新生代也使用了分区机制主要是因为便于代大小的调整**。



### SATB详解

- SATB 是维持并发 GC 的一种手段。G1 并发的基础就是 SATB。SATB 可以理解成在 GC 开始之前对堆内存里的对象做次快照，此时活的对象就认为是活的，从而形成了一个对象图。
- 在 GC 收集的时候，新生代的对象也认为是活的对象，除此之外其他不可达的对象都认为是垃圾对象

#### 如何找到在GC过程中分配的对象呢?

- 每个region记录着两个 top-at-mark-start ( TAMS 指针，分别为 prevTAMS 和 nextTAMS。在TAMS以上的对象就是新分配的，因而被视为隐式marked（即默认被标记了）。
- 通过这种方式我们就找到了在GC过程中新分配的对象，并把这些对象认为是活的对象。
- 解决了对象在GC过程中分配的问题，那么在GC过程中引用发生变化的问题怎么解决呢?
    - G1给出的解决办法是通过 WriteBarrier。Write Barrier 就是对引用字段进行赋值做了额外处理。通过Write Barrier就可以了解到哪些引用对象发生了什么样的变化

### 基础知识

- mark的过程就是遍历heap标记live object的过程，采用的是三色标记算法，这三种颜色为white(表示还未访问到)、gray(访问到但是它用到的引用还没有完全扫描、black( 访问到而且其用到的引用已经完全扫描完)

- 整个三色标记算法就是从GCroots出发遍历heap,针对可达对象先标记white为gray,然后再标记gray为black;遍历完成之后所有可达对象都是black的，所有white都是可以回收的
- SATB仅仅对于在marking开始阶段进行快照（"snapshot"(marked all reachable at markstart)），但是concurrent的时候并发修改可能造成对象漏标记
    - 对black新引用了一个white对象，然后又从gray对象中删除了对该white对象的引用，这样会造成了该white对象漏标记
    - 对black新引用了一个white对象，然后从gray对象删了一个引用该white对象的white对象，这样也会造成了该white对象漏标记，
    - 对black新引用了一个刚new出来的white对象，没有其他gray对象引用该white对象，这样也会造成了该white对象漏标记
- 对于三色算法在concurrent的时候可能产生的漏标记问题，SATB在marking阶段中，对于从gray对象移除的目标引用对象将其标记为 gray,对于black引用的新产生的对象将其标记为black;由于是在开始的时候进行snapshot,因而可能存在Floating Garbage

### 漏标与误标

- 误标没什么关系，顶多造成浮动垃圾，在下次gc还是可以回收的，但是漏标的后果是致命的，把本应该存活的对象给回收了，从而影响的程序的正确性
- 漏标的情况只会发生在白色对象中，且满足以下任意一个条件
    - 并发标记时，应用线程给一个黑色对象的引用类型字段赋值了该白色对象
    - 并发标记时，应用线程删除所有灰色对象到该白色对象的引用（示例：几个灰色和一个黑色同时执行该白色）
- 对于第一种情况，利用post-write barrier，记录所有新增的引用关系，然后根据这些引用关系为根重新扫描一遍
- 对于第二种情况，利用pre-write barrier，将所有即将被删除的引用关系的旧引用记录下来，最后以这些旧引用为根重新扫描一遍

### 停顿预测模型

- G1收集器突出表现出来的一点是通过一个停顿预测模型根据用户配置的停顿时间来选择CSet的大小，从而达到用户期待的应用程序暂停时间。
- 通过-XX:MaxGCPauseMillis参数来设置。这一点有点类似于ParallelScavenge收集器。 关于停顿时间的设置并不是越短越好。
- 设置的时间越短意味着每次收集的CSet越小，导致垃圾逐步积累变多，最终不得不退化成SerialGC;停顿时间设置的过长，那么会导致每次都会产生长时间的停顿，影响了程序对外的响应时间

### G1的收集模式

Young GC 和 Mixed GC 是分代 G1 模式下选择 Cset 的两种子模式；

- G1的运行过程是这样的：会在 Young GC 和 Mixed GC 之间不断地切换运行，同时定期地做全局并发标记，在实在赶不上对象创建速度的情况下 使用 Full GC(这时候会回退到 Serial GC)。
- 初始标记是在 Young GC 上执行的，在进行全局并发标记的时候不会做 MixedGC，在做 MixedGC 的时候也不会启动初始标记阶段。
- 当 MixedGC 赶不上对象产生的速度的时候就退化成 FullGC，这一点是需要重点调优的地方。

### G1最佳实践

- 不要设置新生代和老年代的大小
    - G1 收集器在运行的时候会调整新生代和老年代 的大小。通过改变代的大小来调整对象晋升的速度以及晋升年龄，从而达到我们为收集器设置的暂停时间目标。
    - 设置了新生代大小相当于放弃了 G1 为我们做的自动调优。我们需要做的只是设置整个堆内存的大小，剩下的交给 G1 自已去分配各个代的大小即可。
- 不断调优暂停时间指标
    - 通过 `-XX:MaxGCPauseMillis=x` 可以设置启动应用程序暂停的时间，G1 在运行的时候会根据这个参数选择 CSet 来满足响应时间的设置。一般情况下这个值设置到 100ms 或者 200ms 都是可以的(不同情况下会不一样)，但如果设置成 50ms 就不太合理。**暂停时间设置的太短，就会导致出现 G1 跟不上垃圾产生的速度，最终退化成 Full GC**。所以对这个参数的调优是一个持续的过程，逐步调整到最佳状态。
- 关注Evacuation Failure
    - Evacuation（表示 copy） Failure 类似于 CMS 里面的晋升失败，堆空间的垃圾太多导致无法完成 Region之间的拷贝，于是不得不退化成 Full GC 来做一次全局范围内的垃圾收集

### G1日志解析:

程序代码为：

VM 参数为：`-verbose:gc -Xms10m -Xmx10m -XX:+UseG1GC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:MaxGCPauseMillis=200m`

其中：` * -XX:+UseG1GC` 表示指定垃圾收集器使用G1；`-XX:MaxGCPauseMillis=200m ` 表示设置垃圾收集最大停顿时间

```java
package com.gjxaiou.gc.g1;

/**
 * @Author GJXAIOU
 * @Date 2019/12/20 20:59
 */
public class G1LogAnalysis {
    public static void main(String[] args) {
        int size = 1024 * 1024;
        byte[] myAlloc1 = new byte[size];
        byte[] myAlloc2 = new byte[size];
        byte[] myAlloc3 = new byte[size];
        byte[] myAlloc4 = new byte[size];
        System.out.println("hello world");
    }
}
```

日志结果为：

```java
2019-12-20T21:02:10.163+0800: [GC pause (G1 Humongous Allocation【说明分配的对象超过了region大小的50%】) (young) (initial-mark), 0.0015901 secs]
   [Parallel Time: 0.8 ms, GC Workers: 10【GC工作线程数】]
      [GC Worker Start (ms): Min: 90.3, Avg: 90.4, Max: 90.4, Diff: 0.1]【几个垃圾收集工作的相关信息统计】
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.2, Max: 0.3, Diff: 0.1, Sum: 2.1]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.4, Avg: 0.4, Max: 0.5, Diff: 0.1, Sum: 4.4]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 5.4, Max: 8, Diff: 7, Sum: 54]
【上面的几个步骤为YOUNG GC的固定执行步骤】
 * 阶段1:根扫描
 * 静态和本地对象被描
 * 阶段2:更新RS
 * 处理dirty card队列更新RS
 * 阶段3:处理RS
 * 检测从年轻代指向老年代的对象
 * 阶段4:对象拷贝
 * 拷贝存活的对象到survivor/old区域
 * 阶段5:处理引用队列
 * 软引用，弱引用，虚引用处理
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.3]
      [GC Worker Total (ms): Min: 0.7, Avg: 0.7, Max: 0.7, Diff: 0.1, Sum: 6.9]
      [GC Worker End (ms): Min: 91.1, Avg: 91.1, Max: 91.1, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]【 清楚 cardTable所花费时间】
   [Other: 0.7 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.1 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 2048.0K(6144.0K)->0.0B(2048.0K) Survivors: 0.0B->1024.0K Heap: 3725.2K(10.0M)->2836.0K(10.0M)]
 [Times: user=0.01 sys=0.00, real=0.00 secs] 
2019-12-20T21:02:10.165+0800: [GC concurrent-root-region-scan-start]
2019-12-20T21:02:10.165+0800: [GC pause (G1 Humongous Allocation) (young)2019-12-20T21:02:10.165+0800: [GC concurrent-root-region-scan-end, 0.0006999 secs]
2019-12-20T21:02:10.165+0800: [GC concurrent-mark-start]
, 0.0013416 secs]
   [Root Region Scan Waiting: 0.3 ms]
   [Parallel Time: 0.5 ms, GC Workers: 10]
      [GC Worker Start (ms): Min: 92.5, Avg: 92.6, Max: 92.6, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.1, Max: 0.2, Diff: 0.1, Sum: 1.0]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.3, Avg: 0.3, Max: 0.3, Diff: 0.0, Sum: 3.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 4.6, Max: 8, Diff: 7, Sum: 46]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
      [GC Worker Total (ms): Min: 0.4, Avg: 0.4, Max: 0.5, Diff: 0.1, Sum: 4.1]
      [GC Worker End (ms): Min: 93.0, Avg: 93.0, Max: 93.0, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.2 ms]
   [Other: 0.3 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.2 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(2048.0K)->0.0B【新生代清理后】(1024.0K) Survivors: 1024.0K->1024.0K Heap: 3901.0K(10.0M)->4120.5K(10.0M)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
2019-12-20T21:02:10.166+0800: [GC concurrent-mark-end, 0.0012143 secs]
2019-12-20T21:02:10.167+0800: [Full GC (Allocation Failure)  4120K->3676K(10M), 0.0020786 secs]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 1024.0K->0.0B Heap: 4120.5K(10.0M)->3676.9K(10.0M)], [Metaspace: 3091K->3091K(1056768K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
2019-12-20T21:02:10.169+0800: [GC remark, 0.0000082 secs]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
2019-12-20T21:02:10.169+0800: [GC concurrent-mark-abort]
hello world
Heap
 garbage-first heap   total 10240K, used 4700K [0x00000000ff600000, 0x00000000ff700050, 0x0000000100000000)
  region size 1024K【说明region默认大小】, 1 young (1024K), 0 survivors (0K)
 Metaspace       used 3229K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 350K, capacity 388K, committed 512K, reserved 1048576K

Process finished with exit code 0

```











### 补充

## 0. 背景

目前应用多数使用的是Java 7，其中有很多一部分应用是直面客户，对响应时间有一定的要求。

在Java 7/8中，CMS可以说是关注响应时间的、最优秀的垃圾收集器之一。

正确地使用CMS收集器可以显著提升系统的响应性能，本文介绍一下CMS相关的知识。

只有你真的懂了，才有信心使用它。

PS：即便是在Java 8中，CMS也未必比G1的表现差，还是要具体问题具体的看。

## 1. 概念

一般来说，堆空间是JVM内存中最大的空间，老年代又是堆空间中最大的一部分；因此，对老年代的垃圾收集，一般是比较耗时、耗资源的。

Java 7中默认的老年代收集器是Parallel Old，这是个老年代的并行收集器，它的工作效率很好，换句话说，垃圾收集这件事它做的很好，收集工作的吞吐量很高。

所谓并行收集，代表它在做垃圾收集时，使用的是多线程的工作模式，多个线程并行的做垃圾回收，在多核CPU的资源环境下，表现更好。

但是，它并不是并发收集。所谓并发收集，指的是垃圾回收的工作跟系统的正常逻辑同时运行。

Parallel Old不是并发的，所以它在做垃圾回收时，系统的正常工作逻辑是停止的，俗称STW（Stop The World，整个世界停下来...）

对于一个大内存Java应用来说，例如16G的堆空间，如果每次GC花上个5秒钟时间，期间停止响应前端请求和其他系统调用的请求，是否能接受呢...

相比较，CMS则是一个关注响应时间的、并发的收集器，它的收集工作尽量采用并发方式来执行，即与应用的正常逻辑同时存在；即使有部分环节是以STW的方式在执行，也尽量使用了并行（多线程）方式来处理。

## 2. CMS执行过程简介

简单地说，CMS的收集过程是：

- 从GC Root对象开始寻找可以触达的对象，找到之后就打上标记；
- 标记过程之后，剩余未被标记的对象就会被清除；
- GC Root是标记的起点，它不只一个，所以准确的说是GC Roots。能成为GC Roots的对象很多，例如活跃线程调用栈上的引用变量、某些全局变量、某些ClassLoader等

下图是CMS工作的流程，在完成一次GC的过程中，它会经历如下几个阶段：

初始标记 -> 并发标记 -> 并发预清理 -> 重新标记 -> 并发清理 -> 并发重置

其中：

- 名字叫做'并发xxx'的阶段，CMS的收集线程和应用的线程是同时执行的
- 名字没有'并发xxx'的阶段，CMS是STW（独占）执行的
- 对于一些STW的阶段，例如‘重新标记’，CMS虽然是STW，但也是多线程并行执行的



关于CMS中每个阶段所做的事情，下面给出概要的描述。看不懂没关系，看完第3节再回来读一遍这块会好很多。

**初始标记** ：在这个阶段，需要虚拟机停顿正在执行的任务，官方的叫法STW(Stop The Word)。这个过程从垃圾回收的”根对象”开始，只扫描到能够和”根对象”直接关联的对象，并作标记。所以这个过程虽然暂停了整个JVM，但是很快就完成了。

**并发标记** ：这个阶段紧随初始标记阶段，在初始标记的基础上继续向下追溯标记。并发标记阶段，应用程序的线程和并发标记的线程并发执行，所以用户不会感受到停顿。

**并发预清理** ：并发预清理阶段仍然是并发的。在这个阶段，虚拟机查找在执行并发标记阶段新进入老年代的对象(可能会有一些对象从新生代晋升到老年代，或者有一些对象被分配到老年代)。通过重新扫描，减少下一个阶段”重新标记”的工作，因为下一个阶段会Stop The World。

**重新标记** ：这个阶段会暂停虚拟机，收集器线程扫描在CMS堆中剩余的对象。扫描从”根对象”开始向下追溯，并处理对象关联。

**并发清理** ：清理垃圾对象，这个阶段收集器线程和应用程序线程并发执行。

**并发重置** ：这个阶段，重置CMS收集器的数据结构，等待下一次垃圾回收

## 3. CMS执行过程剖析

CMS的全称是Concurrent Mark Sweep，从名字就可以看出：它是一款并发的、使用标记-清除算法的垃圾收集器。

它是老年代收集器，要使用的话，可以通过参数XX:+UseConcMarkSweepGC打开。

CMS是并发的，它极为关注GC对应用线程的影响，避免造成应用线程的长时间停顿。

但因为采用标记-清除算法，长时间使用CMS，会造成很多小内存碎片；直接影响就是，内存还剩余足够空间，但是因为碎片导致不够连续，从而无法为创建大对象服务。

如下图所示，CMS对老年代的回收，会通过一个后台线程来检测是否需要启动回收。



上图中，判断是否启动回收的条件有多种：

a. 如果使用CMS收集器时，配置了参数-XX:+UseCMSInitiatingOccupancyOnly，代表收集工作的启动，要通过预设的内存阈值参数来控制；即，我们需要配置参数CMSInitiatingOccupancyFraction(老年代阈值)，一旦内存使用超过该值，则触发CMS收集，该值的默认参数是92%

b. 如果没有设置参数-XX:+UseCMSInitiatingOccupancyOnly，则CMS会根据内存和收集情况自行判断是否触发回收

c. 避免晋升失败。新生代对象经过多次回收之后，如果还存活，则会晋升到老年代，晋升之前要经历的回收次数是可配置的，收集器会根据该配置、新生代内存目前的空间情况、新生代目前留存的对象年龄（经历过几次回收）、老年代目前剩余的空间，来判断是否要触发一次老年代回收，以免新生代对象满足晋升条件时，老年代没有足够空间承载。

**初始标记**

初始标记阶段是STW运行的，CMS要尽量缩短这种独占处理的时间，所以只对GC Roots可以直接触达的对象进行标记，而不做深层的递归查找

初始标记的起点，有的来自GC Roots，有的来自新生代对老年代对象的引用



**并发标记**

并发标记阶段，CMS的线程是跟应用线程同时工作的，CMS会顺着初始标记阶段的结果，进行递归查找

因为是并发执行，在CMS做标记的过程中，老年代的内存可能又会发生若干变化，例如：

- 有新生代对象晋升到老年代
- 新建某个比较大的对象，eden区空间不够，直接在老年代分配
- 老年代内部对象的引用关系发生了变化

这些并发标记过程中产生的变化，会通过一种叫CardTable的数据结构来记录，当发生上诉变化时，对象所在的Card会被标记为Dirty。



**并发预清理(Precleaning)和可中断的预清理(AbortablePreclean)**

并发预清理阶段也是一个并发执行阶段，它所做的事情其实还是标记，而非真正的清理

之所以要有该阶段，是因为后续的重新标记是个STW阶段，为了减少STW阶段处理的工作量，所以要尽量在并发处理中做更多的事情：

- 在上述并发标记过程中，如果有在新生代分配新对象，且该对象有引用老年代对象，则将该老年代对象标记
- 遍历CardTable，将其中Dirty(有发生改变的老年代对象)进行处理，打上标记

 

可中断的预清理，也是一个并发执行阶段，也是做的标记的事情

它的目标和并发预处理相似，是要尽量在并发处理中做更多的事情，减轻下面重新标记(STW阶段)的工作量

 该阶段的触发有一定条件，需要Eden区的使用量超过CMSScheduleRemarkEdenSizeThreshold的值，该值默认为2M

 该阶段CMS会循环的执行，执行的逻辑跟并发预清理阶段类似

退出循环的条件有2个，无论哪一个达到，都会退出：

- 循环的执行时间达到CMSMaxAbortablePrecleanTime的值，默认为5s
- Eden区的使用率达到CMSScheduleRemarkEdenPenetration的值，默认为50%

**重新标记**

重新标记是一个STW的执行过程，目的是屏蔽应用线程对GC的干扰，独占的执行CMS的标记过程

虽然并发标记、并发预清理、可中断的预清理阶段都尽量的扫描、标记了存活对象，但因为都是并发执行，难免会产生'漏网之鱼'

这个阶段做的事情，与之前的并发标记也相似：

- 遍历新生代对象，重新标记

- 根据GC Roots，重新标记

- 遍历老年代的Dirty Card，重新标记

如上所述，CMS在重新标记阶段会扫描新生代，如果新生代有大量对象存在，则会比较耗时，拉长STW的时间

所以CMS提供了一个参数CMSScavengeBeforeRemark，可以在重新标记之前触发一次young gc，减少重新标记在扫描新生代上花更多时间

**并发清理**

并发清理阶段就是将老年代中，未被标记的对象删除，完成清理工作。

**并发重置**

并发重置就是将上述过程中的一些数据结构、状态值等进行重置，以备下次收集

## 4. 补充内容

a. CMS的收集过程，尽量缩短了STW的时间，以减少应用线程的停顿

但是，从给另一个角度看，在并发执行的过程中，也占用了应用的CPU资源，导致应用的执行效率和时间降低。

默认情况下，CMS的回收线程数是**(CPU个数+3)/4**，如果是2 core的CPU，则在并发阶段会有50%的线程用于垃圾回收，对应用来说影响还是比较大的

b. 理解了上述CMS的收集流程后，不难发现，CMS没办法像Parallel Old一样，等到老年代满了以后启动垃圾回收

因为CMS是并发执行的，执行过程中，应用还在继续运行，还会继续产生垃圾

为了让垃圾回收和应用逻辑都能正常执行，需要留出一些buffer，例如在垃圾占用率达到75%则启动垃圾回收，剩余的25%用来承载GC过程中产生的'浮动垃圾'

通过参数**CMSInitiatingOccupancyFraction**可以设置这个比例值，如果比例值过大，即浮动垃圾满溢，则会抛出Concurrent  Mode Failure错误

同时，JVM会启动Serial Old收集器来完成对老年代的收集，从名字可以看出，这是一个串行收集算法的收集器，会STW导致应用停止运行

c. 如上文所述，CMS是采用标记-清除算法，标记清除算法的一个缺点是会生成内存碎片，长期累积后，会导致内存没办法接受大对象的存储

CMS提供了两个参数用来处理这个问题：

**UseCMSCompactAtFullCollection**和***\*CMSFullGCsBeforeCompaction\****

第一个参数意思是在full gc时候，CMS会启动压缩，把碎片压缩到内存到一端，这个值默认是true，因此不设置也行

第二个参数是一个整型值，表示经过多少次full gc后，CMS会启动压缩，默认值是0，表示每次full gc都会触发压缩

有个容易混淆的概念是，CMS是老年代垃圾回收算法，跟full gc不是一个维度的概念，CMS对应的是major gc阶段

因此，这个逻辑是，某种逻辑触发了full gc，在full gc发生时，CMS会根据上述2个参数判断，是否要启动内存压缩

 



### 常见设置

**JVM配置，更多的是指GC相关的参数。另外，没有通用的JVM参数配置方案。**

不同类型应用中的内存对象，会有不同的生存特征，没办法使用相同的JVM配置来做统一的优化管理。

即使是相同的应用，在不同的硬件环境下、在不同的性能需求下，也都需要有不同的配置来协调，从而将JVM的性能发挥到良好的状态。

通常来说，JVM的参数配置需要遵循：常规参数配置 -》观察/监控 -》发现/定位问题 -》调整/验证 这样的流程，迭代出适合应用 + 场景 + 需求的JVM配置

没有绝对好用的GC收集器，只有相对适合的，要根据应用的特征和对性能的要求，通过调优过程来迭代出适合就具体应用的具体参数。

通常我们评价一个GC收集器，会关注诸如：GC停顿时间、GC吞吐率、是否STW、GC碎片多少等等来综合评价。

针对如上的项目基础条件，**理论上（具体问题具体分析）**，推荐能做如下的参数控制操作：

a. 控制堆大小（xmx + xms）、堆分配（xmn等，G1收集器不配）、perm空间或meta空间、收集器、gc日志、dump日志这几项内容。

b. 指定收集器：例如Java 7的项目，显示指定ParNew + CMS的收集器组合。

c. 如果堆内存大于等于6G，并且是Java 8的项目，可以考虑打开G1收集器。

d. 如果CMS收集器的GC行为表现良好，不需要更换G1收集器。

e. 具体情况和参数还得取决于项目类型，例如关注的是时间、吞吐量或其他，以及硬件环境等。



## 3. Java 7的配置示例说明

-server // 服务器模式，使用C2重量级编译。启动慢，性能高。JDOS或者JONE默认已经开启了server
-Xmx4g //最大允许分配的堆内存，不要占满物理内存
-Xms4g //跟xmx设置一样即可，防止抖动
-XX:PermSize=512m //持久代内存大小（Java8设置MetaSpace）
-XX:MaxPermSize=512m //跟PermSize设置一样即可
-Xmn2g //新生代内存（需要按应用的特点、minor/full gc之后的内存状况来具体调整，官方推荐为堆内存的3/8）
-XX:SurvivorRatio=6 //survivor区的比例，可以不设置，使用默认值8（需要根据应用特点来调整）
-XX:+DisableExplicitGC //禁止手动GC
-XX:+UseParNewGC //新生代打开ParNewGC
-XX:+UseConcMarkSweepGC //老年代打开CMS收集器
-XX:+CMSParallelRemarkEnabled //降低标记停顿
-XX:+UseCMSInitiatingOccupancyOnly //使用手动定义初始化定义开始CMS收集
-XX:CMSInitiatingOccupancyFraction=75 //使用cms作为垃圾回收使用75％后开始CMS
-XX:+ScavengeBeforeFullGC //Full GC之前做一次MiniorGC，按需选择
-XX:+CMSScavengeBeforeRemark //remark阶段做一次MiniorGC，按需选择
-XX:+UseFastAccessorMethods //原始类型的快速优化，按需选择
-XX:+PrintGCDateStamps //GC日志打印日期参数
-XX:+PrintGCDetails //GC日志打印细节
-verbose:gc //打开GC日志
-Xloggc:/tmp/gc.log //GC日志文件地址
-XX:+UseGCLogFileRotation //GC日志文件滚动保存
-XX:NumberOfGCLogFiles=5 //GC日志文件数量
-XX:GCLogFileSize=256m //GC日志文件大小
-XX:+HeapDumpOnOutOfMemoryError //OOM时候把内存快照打印下来
-XX:HeapDumpPath=/tmp //内存快照的地址

## 4. 堆空间的经验值

关于堆大小和对分配的数值，没有绝对的经验值，推荐的经验值：经过Full GC之后内存对象的留存情况，可以估算出来。

**活跃数据的大小**是指，应用程序稳定运行时长期存活对象在堆中占用的空间大小，也就是Full GC后堆中老年代占用空间的大小。可以通过GC日志中Full GC之后老年代数据大小得出，比较准确的方法是在程序稳定后，多次获取GC数据，通过取平均值的方式计算活跃数据的大小。活跃数据和各分区之间的比例关系如下

| 空间   | 倍数                                    |
| :----- | :-------------------------------------- |
| 总大小 | **3-4** 倍活跃数据的大小                |
| 新生代 | **1-1.5** 活跃数据的大小                |
| 老年代 | **2-3** 倍活跃数据的大小                |
| 永久代 | **1.2-1.5** 倍Full GC后的永久代空间占用 |

## 5. G1收集器的配置

网上有很多比较G1和CMS收集器表现优劣的文章，但脱离具体环境和需求来对比，是没有意义的。

虽然Oracle官宣G1有更优秀的算法，但也**有很多人给出了CMS的吞吐和停顿时间更优**的测试结果。

个人理解，G1的优点更多的存在于：

a. 可配置预期的GC停顿时间（软控制 - G1会尽量满足，但不保证）

  -XX:MaxGCPauseMillis=200（默认值也是200）

b. G1是有压缩的算法，可减少内存碎片

c. 可自动调节内存空间的比例，如新生代空间的大小，不用自己配置，Metaspace也是自动扩展的

d. 更优的算法（基于预测）

不必纠结CMS还是G1，如果CMS在你的应用环境中表现尚可，不必换G1；如果你觉得G1的上述特性对你很重要（例如想控制GC停顿时间尽量不要超过多少毫秒），那么按照下面的配置来使用G1收集器：

```
-server 
-Xms4G 
-Xmx4G 
-XX:MetaspaceSize=512m 
-XX:MaxMetaspaceSize=512m 
-XX:+UseG1GC 
```

 -XX:MaxGCPauseMillis=200 //默认值也是200，200可不设置

```
-XX:+DisableExplicitGC //禁止手动GC
```

-verbose:gc //打开GC日志

-Xloggc:/tmp/gc.log //GC日志文件地址

-XX:+UseGCLogFileRotation //GC日志文件滚动保存

-XX:NumberOfGCLogFiles=5 //GC日志文件数量

-XX:GCLogFileSize=256m //GC日志文件大小

-XX:+HeapDumpOnOutOfMemoryError //OOM时候把内存快照打印下来

-XX:HeapDumpPath=/tmp //内存快照的地址

