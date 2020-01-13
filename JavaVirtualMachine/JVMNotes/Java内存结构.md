# JVM 内存结构

- 虚拟内存：将一些磁盘空间当做内存使用；

# 一、 JVM 内存划分

jdk 中 1.7 和 1.8 中间有区别

JVM 在运行 Java 程序的过程中会将其所管理的内存划分为若干个不同的数据区域，JVM 管理的内存包括以下几个运行时数据区域：（下面为 JVM 运行时内存数据区域）

![img](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/166786b5cf6d7f95)

| 区域       | 是否线程共享 | 是否会内存溢出 |
| ---------- | ------------ | -------------- |
| 程序计数器 | 否           | 不会           |
| 虚拟机栈   | 否           | 会             |
| 本地方法栈 | 否           | 会             |
| 堆         | 是           | 会             |
| 方法区     | 是           | 会             |



- 虚拟机栈：每个虚拟机栈都是归属于一个线程的，是线程私有的空间，当一个线程创建的时候，与之对应的虚拟机栈就产生了，线程消亡则对应的虚拟机栈就消失；**生命周期同线程相同**；其描述的是 Java 方法执行的内存模型，每个方法执行的同时都会创建一个栈帧。

    - 虚拟机栈中数据称为：栈帧：每一个方法执行的时候都会创建一个与该方法有关并且独有的栈帧（JVM 是基于栈执行的），里面存储操作数栈中数据，局部变量表（就是该线程可以自己访问到的局部变量信息（包括八种基本数据类型（其中 64 位的 long、double 占用两个局部变量空间（Slot））和对象引用（reference 类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个-  代表对象的句柄或者其他与此对象相关的位置）））、方法的返回地址（出口，即执行一条字节码指令的地址）、动态连接等，即主要存储与方法执行相关的内容；**每个方法从调用到执行完毕，对应一个栈帧在虚拟机栈中的入栈和出栈。** 

    - **通常所说的栈，一般是指虚拟机栈中的局部变量表部分**。局部变量表所需的内存在编译期间完成分配。 栈的大小可以固定也可以动态扩展，当扩展到无法申请足够的内存，则OutOfMemoryError。 当栈调用深度大于JVM所允许的范围，会抛出StackOverflowError的错误，不过这个深度范围不是一个恒定的值
    - 局部变量表所需的内存空间实在编译期间完成分配，运行期不会改变；
    - 异常：如果线程请求的栈深度大于虚拟机所允许的深度，将抛出 StackOverflowError 异常；当虚拟机在动态扩展时候无法申请到足够的内存，将抛出 OutOfMemoryError 异常；

- 程序计数器（Program Counter Register）：是当前线程所执行的字节码的行号指示器，描述字节码解释权工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，线程在执行字节码时候，执行完当前字节码之后，指定下一行字节码的位置在哪（因为执行可以顺序也可以跳转）；
  
    - 是线程所私有的内存空间（或者说是数据结构），针对多线程，本质上是通过线程轮流切换并分配处理器执行时间的方式实现，即在任意一个确定时刻，一个处理器都只会执行一条线程中的指令，因此切换执行下一个线程需要记住上一个线程挂起到什么位置（便于恢复到正确的执行位置），因此**每个线程都有一个独立的程序计数器**，各个线程之间计数器互不影响，独立存储；
    - 如果线程执行的是一个 Java  方法，则计数器记录的是正在执行的虚拟机字节码指令的地址，如果正在执行的是 Native 方法，则计数器值为空（Undefined）；
    
- 本地方法栈（Native Method Stack）：方法上加上 Native 关键字，表示该方法是有 C/C++ 实现，不是 Java 实现的，即主要用于执行本地方法；同样会抛出 StackOverflowError 和 OutOfMemoryError 异常。

    - 虚拟机栈和本地方法栈区别：前者是为虚拟机执行 Java 方法（即字节码）服务，后者是为虚拟机使用到的 Native 方法服务。
    - 虚拟机规范中对该部分没有强制规范， **Hotspot 虚拟机直接将本地方法栈和虚拟机栈合二为一**。

- 堆（Heap）：在虚拟机启动时候创建，对所有线程共享，存放绝大部分的对象实例（部分会使用栈上分配，标量替换 技术存放在其他位置），Java 中不能直接使用对象，只能通过引用方式获取该对象然后使用它，引用作为一个变量是在栈中。

    - 线程共享的 Java 堆中可以划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer,TLAB）

    - Java 堆可以处于物理上不连续的内存空间中，只要逻辑上连续即可，一般都是可以扩展的；

- 方法区：对所有线程共享，存储元信息，包括已被虚拟机加载的类信息、常量、静态变量、即使编译器编译后的代码、类中（Class ）固有的信息；在 Hotspot 中 **永久代（Permanent Generation）从 JDK 1.8 中已经废弃 **，并且永久代不等于方法区，Hotspot 虚拟机（其他虚拟机不存在永久代概念）使用永久代来实现方法区，即将 GC 分代收集器拓展到方法区，使得垃圾收集器可以像管理 Java 堆一样管理该部分内存，省去专门为方法区编写内存管理代码的工作。
    - 运行时常量池（Runtime Constant Pool）：方法区的一部分，Class 文件中的常量池会存储编译期生成的字面值和符号引用，该部分内容在类加载后进入方法区的运行时常量池中存放，**运行时常量池相比 Class 文件常量池而言具有动态性**，因为 Java 并不要求常量一定只有编译期才能产生即并非只有预置在 Class 文件中常量池部分的内容才可以进入方法区运行时常量池，**运行期间也可以将新的常量池放入池中**，例如：String 类的 intern（） 方法；
    
- 直接内存：不是虚拟机运行时数据区的一部分，也不是 Java 虚拟机规范中定义的内存区域，即不是 JVM 管理的内存，与 Java NIO（New Input/Output） 密切相关，通过使用 Native 函数库直接分配堆外内存，由操作系统进行管理， JVM 通过存储在堆上的 DirectByteBuffer 对象作为该内存的引用来操作直接内存；

# 二、Java 对象创建过程

注：这里对象指普通 Java 对象，不包括数组和 Class 对象

- 创建对象的方式
    - 使用 new 关键字
    - 使用 clone
    - 通过反射
    - 通过反序列化

* new关键字创建对象的3个步骤:

     * 在堆内存中创建出对象的实例

        当虚拟机遇到一条 new 指令时候，首先虚拟机会检查该指令的参数能否在常量池中定位到一个类的符号引用，然后检查这个符号引用所代表的类是不是被正确的加载、连接、初始化，如果没有首先进行类加载过程。

        当上述过程完成之后，虚拟机开始为新生对象分配内存（实际分配的内存空间在对象加载完成之后就确定了），为对象分配内存的任务相当于将一块确定大小的内存从 Java 堆中划分出来；在堆中为对象分配内存分为两种情况（堆内存空间分为已经被占用和未被占用两部分），【情况一（指针碰撞）：如果占用和未占用分别是两块连续空间，中间存放一个指针作为分界点的指示器】如果在未被占用的空间中为对象分配了一段的内存空间，则原来指向未被占用空间位置的指针发生偏移，指向下一个未被占用的空间位置（指针挪动的距离等价于分配的内存），这样对象就创建完成了；【情况二（空闲列表）：两块空间不连续】虚拟机会记录已被使用和未被使用的地址列表，以及未被使用的内存地址大小，如果需要为对象分配内存空间，则需要在未被使用的地址列表中选择一块可以容纳该对象的内存空间放置对象，然后在列表中将记录修改；

        - 指针碰撞（Bump the Pointer）(前提是堆中的空间通过一个指针进行分割，一侧是已经被占用的空间，另一侧是未被占用的空间)
        - 空闲列表(Free List)(前提是堆内存空间中已被使用与未被使用的空间是交织在一起的，这时，虚拟机就需要通过一个列表来记录哪些空间是可以使用的，哪些空间是已被使用的，接下来找出可以容纳下新创建对象的且未被使用的空间，在此空间存放该对象，同时还要修改列表上的记录)
        - 为什么堆不确定是否平整：取决于堆所采用的垃圾收集器是否带有压缩整理功能；
        - 针对并发情况下频繁创建对象可能带来的线程不安全问题（分配了内存但是指针没来得及修改，其他对象同时使用了原来的指针进行分配内存）：方法一：对分配内存空间的动作进行同步处理（虚拟机中采用 CAS 加上失败重试保证更新操作的原子性）；方法二：将内存分配的动作按照线程划分在不同的空间中进行，即每个线程在 Java 堆中预先分配一小块内存称为本地线程分配缓存（TLAB），哪个线程要分配内存时就在哪个线程的 TLAB 上进行分配，只有 TLAB 用完并分配新的 TLAB 时候才需要进行同步锁定，可以使用 `-XX:+/-UseTLAB`参数设定。

     * 为对象的实例成员变量赋初值（对于静态变量在加载阶段就进行了赋初值），因为虚拟机在内存分配完成之后就会将分配到的内存空间都初始化为零值（不包括对象头）（若使用 TLAB，在分配 TLAB 时就执行该步骤），保证对象的实例字段可以在不赋初值情况下就可以使用。

     * 虚拟机对对象进行必要的设置，如该对象为哪个类的实例、怎么找到类的元数据，对象的 Hash 码等，这些信息都存放在**对象的对象头**中，可以进行不同设置。至此对于虚拟机来说一个对象已经产生了，但是对于 Java 程序而言对象创建才刚刚开始，还需要执行 `<init>`方法，同时对字段进行赋值。

     * 将对象的引用返回

 * 对象在内存中的布局（即对象包含的信息）
     * 对象头（Header）：例如对象的 Hash 码以及分代信息
       * 一部分称为（Mark Word）用于存储自身的运行时的数据，如哈希码、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID，根据虚拟机位数不同占 32 / 64 bit，该部分数据结构不固定，会根据对象的状态复用自己的存储空间。
       * 另一部分为：类型指针，即对象指向它的类元数据的指针（虚拟机通过该指针来确定这个对象是哪个类的实例），但是不是所有的虚拟机实现都必须在对象数据上保留类型指针；
  * 另一部分：**只有数组对象有**，用于记录数组长度的数据，因为虚拟机可以通过普通 Java 对象的元数据信息确定 Java 对象的大小，但是从数据的元数据中却无法确定数组的大小。
     * 实例数据（Instance Data）：即对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容（无论是从父类继承或者子类中定义的）。这里信息存储的顺序受虚拟机分配策略参数和字段在 Java 源码中定义顺序的影响；Hotspot 虚拟机中默认的分配策略为：longs/doubles、ints、shorts/chars、bytes/booleans，oops(Ordinary Object Pointers)，其次父类中定义的变量在子类之前；
     * 对齐填充（Padding）（非必须）：起到占位符作用，因为 Hotspot 中自动内存管理系统要求对象起始地址必须是 8 字节的整倍数（即对象的大小必须是 8 字节的整数倍）。
     
 * 对象的访问定位（引用访问对象的方式）
   

Java 程序需要通过栈上的 reference 数据来操作堆上的具体对象；
     
* 使用句柄的方式
  
    - 首先在堆中划分出一块内存来作为句柄池，reference 中存储的是对象的句柄地址，句柄分为两部分，一部分为该对象实例真正的指针，执行真正的对象实例数据信息，第二部分为类型数据各自的具体地址信息，元数据信息放置在方法区。
    - 优势：reference 中存储的是稳定的句柄地址，当对象移动（如垃圾回收时候）时只会改变句柄中的实例数据指针，而 reference 本身不需要修改。
    
    ![通过句柄方式访问对象](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/955ac97ce62d2deb57356f1aee43f33a.jpeg)
    
     * 使用直接指针的方式（Hotspot 使用方式）
    
        - Java 堆对象中放置访问类型数据的相关信息，reference 中存储的是对象地址。
    
        - 优势：速度更快，节省一次指针定位的时间开销（并且对象的访问在 Java 中非常频繁）。
    
        ![通过直接指针方式访问对象](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/ee322420543cd38485ba6e1ae665ac82.jpeg)

## 一、虚拟机堆内存溢出测试

因为堆用于存储对象实例，所以通过不断的创建对象实例，并且保证 GC Roots 到对象之间有科大路径来避免垃圾回收机制清除这些对象。

```java
  //-Xms5m -Xmx5m -XX:+HeapDumpOnOutOfMemoryError 设置jvm对空间最小和最大值（如果两值相同则堆不会自动扩展）以及遇到内存溢出异常时 Dump 出当前的内存堆转储快照，便于以后分析。
package com.gjxaiou.memory;

import java.util.ArrayList;
import java.util.List;

public class MyTest1 {
    public static void main(String[] args) {
      
        //打开jvisualvm 装在磁盘上的转存文件
        List<MyTest1> list = new ArrayList<>();
        while (true) {
            list.add(new MyTest1());
        }
    }
}
```

报错结果：

```java
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid20108.hprof ...
Heap dump file created [2076951 bytes in 0.060 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.gjxaiou.memory.MyTest1.main(MyTest1.java:12)
```

对 Dump 出来的堆转存储快照进行分析，判断内存中对象是否是必要的，即首先确定是内存泄漏（Memory Leak）还是内存溢出（Memory Overflow）

- 如果是内存泄漏，查看泄漏对象到 GC Roots 的引用链，就可以找到泄漏对象是通过怎样的路径与 GC Roots 相关联并且导致垃圾回收器无法回收他们，从而定位泄漏代码的位置；
- 反之则表示内存中的对象确实必须保持存活，则应当检查虚拟机堆参数（-Xmx 和 -Xms），是否可以增大；另一方面检查代码上是否存在某些对象生命周期过长，保持状态时间过长的情况，尝试减少程序运行期的内存消耗

### （一）JVisualVM 使用

直接在 cmd 控制台中输入 jvisualvm 即可开启

![image-20191211162736253](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211162736253.png)

![image-20191211163216509](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211163216509.png)

如果在上面的代码中：` list.add(new MyTest1());` 调用 `System.gc()`;然后再次执行该程序，这时会在 JVisualVM 的左边本地进程中多一个该程序的进程，点击打开之后

首先可以看到概述以及 JVM 参数

![image-20191211164809109](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211164809109.png)

然后可以在监视中查看，其他线程和抽样器均可以可视化的查看程序运行信息；

![image-20191211164902563](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211164902563.png)

# 二、虚拟机栈内存溢出测试

Hotspot 虚拟机不区分虚拟机栈和本地方法栈，因此通过（-Xoss）设置本地方法栈大小是无效的，栈容量只能通过 `-Xss` 参数设置

- 如果线程请求的栈深度大于虚拟机所允许的最大深度，抛出 `StackOverflowError`异常；
- 如果虚拟机在拓展栈时无法申请到足够的内存空间，则抛出 `OutOfMemoryError` 异常；
- 以上两种异常会互相重叠，本质是对同一件事情的两种描述，因为栈空间无法继续分配的时候，可能是内存太小，也可能为已使用的栈空间过大。

在下面**单线程**的情况下，无论是使用 `-Xss` 参数减少栈内存容量或者是定义了大量的本地变量从而增加此方法帧中本地变量表的长度，**只能**抛出 `StackOverflowError`，出异常的时候输出堆栈深度相应减小；

```java
package com.gjxaiou.memory;

/**
 * 虚拟机栈溢出测试(使用递归)
 * @Author GJXAIOU
 * @Date 2019/12/11 16:53
 */

public class MyTest2 {
    // 查看一共递归了多少层
    private int length;
    public int getLength() {
        return length;
    }

    public void test() throws InterruptedException {
        length++;
        Thread.sleep(300);
        test();
    }

    public static void main(String[] args) {
        //测试调整虚拟机栈内存大小为：  -Xss160k，此处除了可以使用JVisuale监控程序运行状况外还可以使用jconsole
        MyTest2 myTest2 = new MyTest2();
        try {
            myTest2.test();
            // 注意：catch 捕捉的是 Throwable，不是 Exception，因为 STackOverflow 和 OutOfMemoryError 都不是 Exception 的子类
        } catch (Throwable e) {
            //打印最终的最大栈深度为：2581
            System.out.println(myTest2.getLength());
            e.printStackTrace();
        }
    }
}

```

程序报错：

```java
java.lang.StackOverflowError
	at com.gjxaiou.memory.MyTest2.test(MyTest2.java:18)
	at com.gjxaiou.memory.MyTest2.test(MyTest2.java:19)
	at com.gjxaiou.memory.MyTest2.test(MyTest2.java:19)
	at com.gjxaiou.memory.MyTest2.test(MyTest2.java:19)
	at com.gjxaiou.memory.MyTest2.test(MyTest2.java:19)
	at com.gjxaiou.memory.MyTest2.test(MyTest2.java:19)
    ......
    
```



程序运行时候同时打开 JvisualVM ，在 线程 选项右上角有一个 线程 Dump，可以查看所有线程的状态，这里主要看 Main 线程，可以由下图中看出该线程一直在调用 19行的 test() 方法，然后最后 返回了 26 行的调用方法，其他的监视、线程等等也可以查看；

![image-20191211171420096](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211171420096.png)

**测试多线程情况**

首先操作系统对于分配给每个进程的内存是有限制的，为 总内存 - 最大堆容量 - 最大方法区容量（程序计数器忽略），剩余的内存由











### （一）JConsole 使用

同样在控制台中使用 `jconsole`命名来启动（提前启动项目），然后本地连接到该项目即可监控程序，**特色**：可以在线程选项框最下面检查程序是否存在死锁；

```java
package com.gjxaiou.memory;

/**
 * @Author GJXAIOU
 * @Date 2019/12/11 18:03
 */
public class MyTest3 {
    public static void main(String[] args) {
        // 构造两个线程
        // 步骤一：Thread-A 线程启动，执行 A.method（）方法，然后就会拿到类 A 对应的 Class 对象的锁，同时执行方法，睡眠，当执行到 B.method() 方法时候，发现该方法也是 synchronized 的，所以会尝试获取类 B 对应的 Class 对象对应的锁；
        new Thread(() -> A.method(), "Thread-A").start();
        //步骤二：同时 Thread-B 线程启动，同上步骤就会形成死锁
        new Thread(() -> B.method(), "Thread-B").start();
    }
}

class A{
    // 线程进入到 synchronized 修饰的方法后，并且该方法是由 static 修饰的，则持有的不是当前类（Class A）对应的锁，而是当前类所对应的 Class
    // 对象的锁，所以不管该类有多少个实例或者对象，持有的都是一把锁
    public static synchronized  void method(){
        System.out.println("method from A");
        try {
            Thread.sleep(5000);
            B.method();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class B{
    public static synchronized void method(){
        System.out.println("method from B");
        try {
            Thread.sleep(5000);
            A.method();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



程序对应的监测结果为：首先通过线程栏正下方的 “检测死锁” 之后结果如下：

状态可以看出：`java.lang.Class`上的 Blocked，拥有者是 Thread-B，说明线程 Thread-B 已经持有了 `java.lang.Class@77552c4c` 这个对象的锁，所以 Thread-A 在这个对象上处于阻塞状态；因为调用的是 `B.method()`所以等待的是 B 类对应的 Class 对象的锁。

![image-20191211182158268](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211182158268.png)

同样在 JVisualVM 中会自动提示检测到死锁，并且按照提示在线程选项中生成一个线程 Dump，然后查看上面的两个线程，发现他们分别已经锁定了自己的 Class 对象，想锁定对方的 Class 对象；

![image-20191211183809571](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211183809571.png)



## 三、方法区元空间溢出测试

因为从 1.8 开始废除永久代，使用元空间，因为元空间采用的是操作系统本地的内存，初始内存大小为 21 M，并且如果不断占用达到空间最大内存大小则元空间虚拟机会进行垃圾回收，如果回收还是不够就会进行内存扩展，最大可以扩展到物理内存最大值；

首先需要显式的设定初始元空间大小，同时因为元空间中存放一个类的 Class 的元信息（并不存放最占空间的对象实例）， 因此需要不断将 Class 信息不断的增加到元空间中，例如在 Spring （jsp 会动态转为 Servlet，CGlib 等等同理）中会在运行期动态的生成类（就是该类在编译时候是不存在的，在运行期动态创建），这些动态创建类的元信息就要放在元空间中，因此需要不断的动态创建类。

```java
package com.gjxaiou.memory;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
/**
 * @Author GJXAIOU
 * @Date 2019/12/11 19:00
 */

/**
 * 元空间内存溢出测试(使用 cglib,需要导入对应 jar 包和 asm.jar)
 * 设置元空间最大大小（不让其扩容）：-XX:MaxMetaspaceSize=200m
 * 关于元空间参考：https://www.infoq.cn/article/java-permgen-Removed
 */
public class MyTest4 {
    public static void main(String[] args) {
        //使用动态代理动态生成类（不是实例）
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(MyTest4.class);
            enhancer.setUseCache(false);
            enhancer.setCallback((MethodInterceptor) (obj, method, ags, proxy) -> proxy.invokeSuper(obj, ags));
            System.out.println("Hello World");
            // 在运行期不断创建 MyTest 类的子类
            enhancer.create();
        }
    }
}
/** output:
 * Caused by: java.lang.OutOfMemoryError: Metaspace
 */

```

从 Jconsole 中可以看出，只有类是不断增加的

![image-20191211193600512](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211193600512.png)

使用 JVisualVM 可以查看元空间增长情况

![image-20191211193944968](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211193944968.png)

### 四、JVM命令使用

查看当成程序进程号： ps -ef | grep java（获取所有包含 java 的进程及其 id）**建议使用**：`jsp -l`

```java
package com.gjxaiou.memory;

/**
 * @Author GJXAIOU
 * @Date 2019/12/11 20:20
 */
public class MyTest5 {
    public static void main(String[] args) {
        while (true) {
            System.out.println("hello world");
        }
    }
}
```

- 使用 `jmap -clstats` + pid 结果如下：

![image-20191211205050818](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211205050818.png)

```java
C:\Users\gjx16>jmap -clstats 17992
Attaching to process ID 17992, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.221-b11
finding class loader instances ..done.
computing per loader stat ..done.
please wait.. computing liveness.liveness analysis may be inaccurate ...
class_loader    classes bytes   parent_loader   alive?  type

<bootstrap>     606     1134861   null          live    <internal>
0x00000006c24ba258      0       0       0x00000006c2404b38      dead    java/util/ResourceBundle$RBClassLoader@0x00000007c00648a8
0x00000006c2404b38      4       5070    0x00000006c2404ba8      live    sun/misc/Launcher$AppClassLoader@0x00000007c000f958
0x00000006c2404ba8      0       0         null          live    sun/misc/Launcher$ExtClassLoader@0x00000007c000fd00

total = 4       610     1139931     N/A         alive=3, dead=1     N/A
```

- 使用 `jmap -heap` + pid 查看堆中状况

```java
C:\Users\gjx16>jmap -heap 5816
Attaching to process ID 5816, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.221-b11

using thread-local object allocation.
Parallel GC with 10 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 4257218560 (4060.0MB)
   NewSize                  = 88604672 (84.5MB)
   MaxNewSize               = 1418723328 (1353.0MB)
   OldSize                  = 177733632 (169.5MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 48758784 (46.5MB)
   used     = 11702160 (11.160049438476562MB)
   free     = 37056624 (35.33995056152344MB)
   24.000106319304436% used
From Space:
   capacity = 524288 (0.5MB)
   used     = 0 (0.0MB)
   free     = 524288 (0.5MB)
   0.0% used
To Space:
   capacity = 1572864 (1.5MB)
   used     = 0 (0.0MB)
   free     = 1572864 (1.5MB)
   0.0% used
PS Old Generation
   capacity = 177733632 (169.5MB)
   used     = 1155216 (1.1016998291015625MB)
   free     = 176578416 (168.39830017089844MB)
   0.6499704006498894% used

3158 interned Strings occupying 259480 bytes.
```

- 使用 `jstat -gc` + pid 查看元空间容量和被使用量

![image-20191211205017818](Java%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84.resource/image-20191211205017818.png)

```java
C:\Users\gjx16>jstat -gc 14320
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
1536.0 1536.0  0.0    0.0   48640.0   8755.1   173568.0    1061.0   4864.0 3763.1 512.0  409.7      19    0.013   0      0.000    0.013
```

其中 MC表示元空间总大小，MU表示元空间已使用的大小；

- jcmd (从JDK 1. 7开始增加的命令)

| 命令                                             | 含义                                                 |
| ------------------------------------------------ | ---------------------------------------------------- |
| jcmd pid VM.flags                                | 查看该线程的JVM 的启动参数                           |
| jcmd pid help                                    | 列出当前运行的 Java 进程可以执行的操作               |
| jcmd pid help 具体命令                           | 查看具体命令的选项                                   |
| jcmd pid PerfCounter.print                       | 查看具体命令的选项                                   |
| jcmd pid VM.uptime                               | 查有JVM的启动时长                                    |
| jcmd pid GC.class_ histogram                     | 查看系统中类的统计信息                               |
| jcmd pid Thread.print                            | 查看线程堆栈信息                                     |
| jcmd pid GC.heap_dump filename.hprof(可以加路径) | 导出 Heap dump文件， 导出的文件可以通过jvisualvm查看 |
| jcmd pid VM.system_ properties                   | 查看 JVM 的属性信息                                  |
| jcmd pid VM.version                              | 查看目标 JVM 进程的版本信息                          |
| jcmd pid VM.command_line                         | 查看 JVM 启动的命令行参数信息                        |

- jstack ：可以查看或者导出 Java 应用程序中线程的堆栈信息  `jstack pid`

- **jmc**（Java Mission Control）:页面式的查看工具，可以安装插件

    - 使用命令行开启
    - 功能更加齐全，界面更加优秀

    注：jfr（Java Flight Recoder）Java 飞行记录器：可以实时获取 Java 进程的统计数据

- JVisualVM 中有 OQL 对象查询语言，类似于 SQL 语句，可以查询一些值；

### JVM内存举例说明

```java
public void method() {
    Object object = new Object();

    /*生成了2部分的内存区域，1)object这个引用变量，因为
        是方法内的变量，放到JVM Stack里面,2)真正Object
        class的实例对象，放到Heap里面
        上述 的new语句一共消耗12个bytes, JVM规定引用占4
        个bytes (在JVM Stack)， 而空对象是8个bytes(在Heap)
        方法结束后，对应Stack中的变量马上回收，但是Heap
        中的对象要等到GC来回收、*/
}
```


