# 01 | Java代码是怎么运行的？

我们学院的一位教授之前去美国开会，入境的时候海关官员就问他：既然你会计算机，那你说说你用的都是什么语言吧？

教授随口就答了个 Java。海关一看是懂行的，也就放行了，边敲章还边说他们上学那会学的是 C+。我还特意去查了下，真有叫 C+ 的语言，但是这里海关官员应该指的是 C++。

事后教授告诉我们，他当时差点就问海关，是否知道 Java 和 C++ 在运行方式上的区别。但是又担心海关官员拿他的问题来考别人，也就没问出口。那么，下次你去美国，不幸地被海关官员问这个问题，你懂得如何回答吗？

作为一名 Java 程序员，你应该知道，Java 代码有很多种不同的运行方式。比如说可以在开发工具中运行，可以双击执行 jar 文件运行，也可以在命令行中运行，甚至可以在网页中运行。当然，这些执行方式都离不开 JRE，也就是 Java 运行时环境。

实际上，JRE 仅包含运行 Java 程序的必需组件，包括 Java 虚拟机以及 Java 核心类库等。我们 Java 程序员经常接触到的 JDK（Java 开发工具包）同样包含了 JRE，并且还附带了一系列开发、诊断工具。

然而，运行 C++ 代码则无需额外的运行时。我们往往把这些代码直接编译成 CPU 所能理解的代码格式，也就是机器码。

比如下图的中间列，就是用 C 语言写的 Helloworld 程序的编译结果。可以看到，C 程序编译而成的机器码就是一个个的字节，它们是给机器读的。那么为了让开发人员也能够理解，我们可以用反汇编器将其转换成汇编代码（如下图的最右列所示）。

```shell

; 最左列是偏移；中间列是给机器读的机器码；最右列是给人读的汇编代码
0x00:  55                    push   rbp
0x01:  48 89 e5              mov    rbp,rsp
0x04:  48 83 ec 10           sub    rsp,0x10
0x08:  48 8d 3d 3b 00 00 00  lea    rdi,[rip+0x3b] 
                                    ; 加载"Hello, World!\n"
0x0f:  c7 45 fc 00 00 00 00  mov    DWORD PTR [rbp-0x4],0x0
0x16:  b0 00                 mov    al,0x0
0x18:  e8 0d 00 00 00        call   0x12
                                    ; 调用printf方法
0x1d:  31 c9                 xor    ecx,ecx
0x1f:  89 45 f8              mov    DWORD PTR [rbp-0x8],eax
0x22:  89 c8                 mov    eax,ecx
0x24:  48 83 c4 10           add    rsp,0x10
0x28:  5d                    pop    rbp
0x29:  c3                    ret
```

既然 C++ 的运行方式如此成熟，那么你有没有想过，为什么 Java 要在虚拟机中运行呢，Java 虚拟机具体又是怎样运行 Java 代码的呢，它的运行效率又如何呢？

今天我便从这几个问题入手，和你探讨一下，Java 执行系统的主流实现以及设计决策。

## 为什么 Java 要在虚拟机里运行？

Java 作为一门高级程序语言，它的语法非常复杂，抽象程度也很高。因此，直接在硬件上运行这种复杂的程序并不现实。所以呢，在运行 Java 程序之前，我们需要对其进行一番转换。

这个转换具体是怎么操作的呢？当前的主流思路是这样子的，设计一个面向 Java 语言特性的虚拟机，并通过编译器将 Java 程序转换成该虚拟机所能识别的指令序列，也称 Java 字节码。这里顺便说一句，之所以这么取名，是因为 Java 字节码指令的操作码（opcode）被固定为一个字节。

举例来说，下图的中间列，正是用 Java 写的 Helloworld 程序编译而成的字节码。可以看到，它与 C 版本的编译结果一样，都是由一个个字节组成的。

并且，我们同样可以将其反汇编为人类可读的代码格式（如下图的最右列所示）。不同的是，Java 版本的编译结果相对精简一些。这是因为 Java 虚拟机相对于物理机而言，抽象程度更高。

```java
# 最左列是偏移；中间列是给虚拟机读的机器码；最右列是给人读的代码
0x00:  b2 00 02         getstatic java.lang.System.out
0x03:  12 03            ldc "Hello, World!"
0x05:  b6 00 04         invokevirtual java.io.PrintStream.println
0x08:  b1               return
```

Java 虚拟机可以由硬件实现[1]，但更为常见的是在各个现有平台（如 Windows_x64、Linux_aarch64）上提供软件实现。这么做的意义在于，一旦一个程序被转换成 Java 字节码，那么它便可以在不同平台上的虚拟机实现里运行。这也就是我们经常说的“一次编写，到处运行”。

虚拟机的另外一个好处是它带来了一个托管环境（Managed Runtime）。这个托管环境能够代替我们处理一些代码中冗长而且容易出错的部分。其中最广为人知的当属自动内存管理与垃圾回收，这部分内容甚至催生了一波垃圾回收调优的业务。

除此之外，托管环境还提供了诸如数组越界、动态类型、安全权限等等的动态检测，使我们免于书写这些无关业务逻辑的代码。

## Java 虚拟机具体是怎样运行 Java 字节码的？

下面我将以标准 JDK 中的 HotSpot 虚拟机为例，从虚拟机以及底层硬件两个角度，给你讲一讲 Java 虚拟机具体是怎么运行 Java 字节码的。

从虚拟机视角来看，执行 Java 代码首先需要将它编译而成的 class 文件加载到 Java 虚拟机中。加载后的 Java 类会被存放于方法区（Method Area）中。实际运行时，虚拟机会执行方法区内的代码。

如果你熟悉 X86 的话，你会发现这和段式内存管理中的代码段类似。而且，Java 虚拟机同样也在内存中划分出堆和栈来存储运行时数据。

不同的是，Java 虚拟机会将栈细分为面向 Java 方法的 Java 方法栈，面向本地方法（用 C++ 写的 native 方法）的本地方法栈，以及存放各个线程执行位置的 PC 寄存器。

![img](https://static001.geekbang.org/resource/image/ab/77/ab5c3523af08e0bf2f689c1d6033ef77.png)

在运行过程中，每当调用进入一个 Java 方法，Java 虚拟机会在当前线程的 Java 方法栈中生成一个栈帧，用以存放局部变量以及字节码的操作数。这个栈帧的大小是提前计算好的，而且 Java 虚拟机不要求栈帧在内存空间里连续分布。

当退出当前执行的方法时，不管是正常返回还是异常返回，Java 虚拟机均会弹出当前线程的当前栈帧，并将之舍弃。

从硬件视角来看，Java 字节码无法直接执行。因此，Java 虚拟机需要将字节码翻译成机器码。

在 HotSpot 里面，上述翻译过程有两种形式：第一种是解释执行，即逐条将字节码翻译成机器码并执行；第二种是即时编译（Just-In-Time compilation，JIT），即将一个方法中包含的所有字节码编译成机器码后再执行。

![img](https://static001.geekbang.org/resource/image/5e/3b/5ee351091464de78eed75438b6f9183b.png)

前者的优势在于无需等待编译，而后者的优势在于实际运行速度更快。HotSpot 默认采用混合模式，综合了解释执行和即时编译两者的优点。它会先解释执行字节码，而后将其中反复执行的热点代码，以方法为单位进行即时编译。

## Java 虚拟机的运行效率究竟是怎么样的？

HotSpot 采用了多种技术来提升启动性能以及峰值性能，刚刚提到的即时编译便是其中最重要的技术之一。

即时编译建立在程序符合二八定律的假设上，也就是百分之二十的代码占据了百分之八十的计算资源。

对于占据大部分的不常用的代码，我们无需耗费时间将其编译成机器码，而是采取解释执行的方式运行；另一方面，对于仅占据小部分的热点代码，我们则可以将其编译成机器码，以达到理想的运行速度。

理论上讲，即时编译后的 Java 程序的执行效率，是可能超过 C++ 程序的。这是因为与静态编译相比，即时编译拥有程序的运行时信息，并且能够根据这个信息做出相应的优化。

举个例子，我们知道虚方法是用来实现面向对象语言多态性的。对于一个虚方法调用，尽管它有很多个目标方法，但在实际运行过程中它可能只调用其中的一个。

这个信息便可以被即时编译器所利用，来规避虚方法调用的开销，从而达到比静态编译的 C++ 程序更高的性能。

为了满足不同用户场景的需要，HotSpot 内置了多个即时编译器：C1、C2 和 Graal。Graal 是 Java 10 正式引入的实验性即时编译器，在专栏的第四部分我会详细介绍，这里暂不做讨论。

之所以引入多个即时编译器，是为了在编译时间和生成代码的执行效率之间进行取舍。C1 又叫做 Client 编译器，面向的是对启动性能有要求的客户端 GUI 程序，采用的优化手段相对简单，因此编译时间较短。

C2 又叫做 Server 编译器，面向的是对峰值性能有要求的服务器端程序，采用的优化手段相对复杂，因此编译时间较长，但同时生成代码的执行效率较高。

从 Java 7 开始，HotSpot 默认采用分层编译的方式：热点方法首先会被 C1 编译，而后热点方法中的热点会进一步被 C2 编译。

为了不干扰应用的正常运行，HotSpot 的即时编译是放在额外的编译线程中进行的。HotSpot 会根据 CPU 的数量设置编译线程的数目，并且按 1:2 的比例配置给 C1 及 C2 编译器。

在计算资源充足的情况下，字节码的解释执行和即时编译可同时进行。编译完成后的机器码会在下次调用该方法时启用，以替换原本的解释执行。

## 总结与实践

今天我简单介绍了 Java 代码为何在虚拟机中运行，以及如何在虚拟机中运行。

之所以要在虚拟机中运行，是因为它提供了可移植性。一旦 Java 代码被编译为 Java 字节码，便可以在不同平台上的 Java 虚拟机实现上运行。此外，虚拟机还提供了一个代码托管的环境，代替我们处理部分冗长而且容易出错的事务，例如内存管理。

Java 虚拟机将运行时内存区域划分为五个部分，分别为方法区、堆、PC 寄存器、Java 方法栈和本地方法栈。Java 程序编译而成的 class 文件，需要先加载至方法区中，方能在 Java 虚拟机中运行。

为了提高运行效率，标准 JDK 中的 HotSpot 虚拟机采用的是一种混合执行的策略。

它会解释执行 Java 字节码，然后会将其中反复执行的热点代码，以方法为单位进行即时编译，翻译成机器码后直接运行在底层硬件之上。

HotSpot 装载了多个不同的即时编译器，以便在编译时间和生成代码的执行效率之间做取舍。

下面我给你留一个小作业，通过观察两个条件判断语句的运行结果，来思考 Java 语言和 Java 虚拟机看待 boolean 类型的方式是否不同。

下载 asmtools.jar(详细介绍 https://www.cnblogs.com/yelongsan/p/9674723.html)  ，并在命令行中运行下述指令（不包含提示符 $）：

```java

$ echo '
public class Foo {
 public static void main(String[] args) {
  boolean flag = true;
  if (flag) System.out.println("Hello, Java!");
  if (flag == true) System.out.println("Hello, JVM!");
 }
}' > Foo.java
$ javac Foo.java
$ java Foo
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class > Foo.jasm.1
$ awk 'NR==1,/iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.1 > Foo.jasm
$ java -cp /path/to/asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm
$ java Foo

```

## 精选留言(203)

- ![img](https://static001.geekbang.org/account/avatar/00/11/12/da/a3ea305f.jpg)

  jiaobuchongจุ๊บ

  置顶

  对老师写的那段 awk 不懂得可参考： https://blog.csdn.net/jiaobuchong/article/details/83037467

  2018-10-14

  **

  **93

- ![img](https://static001.geekbang.org/account/avatar/00/11/3b/65/203298ce.jpg)

  小名叫大明

  置顶

  受益匪浅，多谢老师。  请教老师一个问题，网上我没有搜到。  服务器线程数爆满，使用jstack打印线程堆栈信息，想知道是哪类线程数太多，但是堆栈里全是一样的信息且没有任何关键信息，是哪个方法创建的，以及哪个线程池的都看不到。  如何更改打印线程堆栈信息的代码（动态）让其打印线程池信息呢？

  2018-07-26

  **4

  **47

- ![img](https://static001.geekbang.org/account/avatar/00/11/f8/db/c4edf697.jpg)

  曲东方

  jvm把boolean当做int来处理 flag = iconst_1 = true awk把stackframe中的flag改为iconst_2 if（flag）比较时ifeq指令做是否为零判断，常数2仍为true，打印输出 if（true == flag）比较时if_cmpne做整数比较，iconst_1是否等于flag，比较失败，不再打印输出

  作者回复: 字节码高手！

  2018-07-20

  **9

  **409

- ![img](https://static001.geekbang.org/account/avatar/00/10/e9/a1/2fe5b97a.jpg)

  novembersky

  文中提到虚拟机会把部分热点代码编译成机器码，我有个疑问，为什么不把java代码全部编译成机器码？很多服务端应用发布频率不会太频繁，但是对运行时的性能和吞吐量要求较高。如果发布或启动时多花点时间编译，能够带来运行时的持久性能收益，不是很合适么？

  作者回复: 问得好！事实上JVM确实有考虑做AOT (ahead of time compilation) 这种事情。AOT能够在线下将Java字节码编译成机器码，主要是用来解决启动性能不好的问题。 对于这种发布频率不频繁(也就是长时间运行吧？)的程序，其实选择线下编译和即时编译都一样，因为至多一两个小时后该即时编译的都已经编译完成了。另外，即时编译器因为有程序的运行时信息，优化效果更好，也就是说峰值性能更好。

  2018-07-20

  **7

  **140

- ![img](https://static001.geekbang.org/account/avatar/00/11/2c/69/021420f0.jpg)

  醉人

  解释执行 执行时才翻译成机器指令，无需保存不占内存。但即时编译类似预编译，编译之后的指令需要保存在内存中，这种方式吃内存，按照二八原则这种混合模式最恰当的，热点代码编译之后放入内存避免重复编译，而其他运行次数较少代码则解释执行，避免占用过多内存

  2018-07-20

  **1

  **123

- ![img](https://static001.geekbang.org/account/avatar/00/11/fc/40/e0d86fd7.jpg)

  かっこいすぎる郑一凡

  我想问下，JVM是怎么区别出热点代码和非热点代码的？

  2018-07-20

  **12

  **100

- ![img](https://static001.geekbang.org/account/avatar/00/11/31/f4/467cf5d7.jpg)

  MARK

  作业终于做出来~\(≧▽≦)/~喜大普奔 asmtools下载地址： https://adopt-openjdk.ci.cloudbees.com/view/OpenJDK/job/asmtools/lastSuccessfulBuild/artifact/asmtools-6.0.tar.gz； 先是在window环境里，awk不能使用，看https://zh.wikipedia.org/wiki/Awk，AWK是一种优良的文本处理工具，Linux及Unix环境中现有的功能最强大的数据处理引擎之一,于是转战Linux， [root@localhost cqq]# javac Foo.java [root@localhost cqq]# java Foo Hello,Java Hello,JVM [root@localhost cqq]# java -cp /cqq/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class>Foo.jasm.1 [root@localhost cqq]# ls asmtools.jar  Foo.class  Foo.jasm.1  Foo.java [root@localhost cqq]# vi Foo.jasm.1 [root@localhost cqq]# awk 'NR==1,/iconst_1/{sub(/iconst_1/,"iconst_2")} 1' Foo.jasm.1>Foo.jasm [root@localhost cqq]# java -cp /cqq/asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm [root@localhost cqq]# java Foo Hello,Java 结果为啥是这个看点赞第一的高手； 另外asmtools使用方式还可以这样子： java -jar asmtools.jar jdis Foo.class>Foo.jasm.1 java -jar asmtools.jar jasm Foo.jasm

  2018-07-25

  **5

  **90

- ![img](https://static001.geekbang.org/account/avatar/00/0f/cc/43/59a9b4ae.jpg)

  之外^Excepts

  我只想问，你就没有教授的担忧？万一我拿今天的知识点去问面试者，答不上来咋办？

  2018-07-21

  **

  **65

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg)

  钱

  1:为什么使用JVM？ 1-1:可以轻松实现Java代码的跨平台执行 1-2:JVM提供了一个托管平台，提供内存管理、垃圾回收、编译时动态校验等功能 1-3:使用JVM能够让我们的编程工作更轻松、高效节省公司成本，提示社会化的整体快发效率，我们只关注和业务相关的程序逻辑的编写，其他业务无关但对于编程同样重要的事情交给JVM来处理 2:听完此节的课程的疑惑（之前就没太明白，原期待听完后不再疑惑的） 2-1:Java源代码怎么就经过编译变成了Java字节码？ 2-2:JVM怎么就把Java字节码加载进JVM内了？先加载那个类的字节码？它怎么定位的？拿到后怎么解析的？不会整个文件放到一个地方吧？使用的时候又是怎么找到的呢？这些感觉还是黑盒 2-3:JVM将内存区域分成堆和栈，然后又将栈分成pc寄存器、本地方法栈、Java方法栈，有些内存空间是线程可共享的，有些是线程私有的。现在也了解不同的内存区块有不同的用处，不过他们是怎么被划分的哪？为什么是他们，不能再多几种或少几种了吗？共享的内存区和私有的又是怎么控制的哪？

  作者回复: 总结得非常细致！ 2-1 其实是这样的，JVM接收字节码，要运行在JVM上只能选择转化为字节码。要是不想在JVM上跑，可以选择直接转化为机器码。 2-2 类加载会在第三篇详细介绍。 2-3 具体的划分都是实现细节。你也可以选择全部冗杂在一起。但是这样子做性能较高，因为线程私有的可以不用同步。

  2018-07-24

  **

  **50

- ![img](https://static001.geekbang.org/account/avatar/00/14/79/a2/18815f9c.jpg)

  J

  在Windows使用不了awk工具（貌似有代替方案），所以结合其他小伙伴的答案和自己的思考，答案整理如下： 小作业的过程是： 1、写Java代码，生成java文件 2、将java文件编译成class文件(字节码) 3、执行字节码，输出两个Hello,world! 4、使用asmtool工具将class文件生成jasm文件 5、使用awt工具将jasm文件stackframe的flag改为iconst_2 6、再次使用asmtool工具将jasm文件恢复成class文件 7、执行字节码，输出一个Hello, world！ 由于Java虚拟机将boolean类型看成0或者1，在步骤5中将源代码中的flag修改成2，于是在步骤7中的运行过程中，if(2)，true,执行输出；if(2 == 1)，结果为false，不执行输出。

  2018-12-25

  **3

  **48

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5d/10/0acf7cbc.jpg)

  Ryan-Hou

  在为什么Java要在虚拟机里执行这一节您提到，java语法复杂，抽象度高，直接通过硬件来执行不现实，但是同样作为高级语言为什么C++就可以呢？这个理由作为引入虚拟机这个中间层的原因不是很充分吧

  作者回复: 多谢指出！这里的直接运行指的是不经过任何转换(编译)，直接在硬件上跑。即便是C++，也不可以直接运行。 C++的策略是直接编译成目标架构的机器码，Java的策略是编译成一个虚拟架构的机器码。这个虚拟架构可以有物理实现(可以搜Java processor)，也可以是软件实现，也就是我们经常接触到的JRE。

  2018-07-20

  **2

  **41

- ![img](https://static001.geekbang.org/account/avatar/00/12/04/4d/bd86bdc2.jpg)

  周仕林

  看到有人说热点代码的区别，在git里面涉及到的热点代码有两种算法，基于采样的热点探测和基于计数器的热点探测。一般采用的都是基于计数器的热点探测，两者的优缺点百度一下就知道了。基于计数器的热点探测又有两个计数器，方法调用计数器，回边计数器，他们在C1和C2又有不同的阈值。😂😂

  作者回复: 谢谢！

  2018-07-23

  **2

  **36

- ![img](https://static001.geekbang.org/account/avatar/00/11/fb/9b/4a4893c1.jpg)

  kernel

  评论比文章精彩，学习到更多

  2018-08-13

  **

  **24

- ![img](https://static001.geekbang.org/account/avatar/00/14/c3/bd/ec8b3044.jpg)

  尔东

  1.Java代码首先编译为class文件，然后通过java虚拟机加载到方法区。Java虚拟机是一个独立的进程，执行方法区的代码。 2.Java虚拟机把内存分为堆栈两种形式来存储运行时数据，包括线程共有的方法区和堆，以及线程私有的pc计数器，方法栈，naive方法栈 3.Java虚拟机将字节码翻译成机器码执行的方法有两种，一种是解释执行，即逐条将字节码翻译成机器码并执行；第二种是即时编译，即将一个方法包含的所有字节码编译成机器码后再执行 4.解释执行的好处是无需等待编译，即时编译的好处是速度更快。这里编译的概念并不是代码编译为字节码，而是字节码编译为机器码，字节码编译为机器码是由java虚拟机在运行程序的时候才会去做的，所以是运行时的开销。热点代码会通过即时编译来执行。 5.HotSpot内置了多个即时编译器，包括C1、C2和Graal。 6.Asmtools.jar下载地址https://ci.adoptopenjdk.net/view/Dependencies/job/asmtools/lastSuccessfulBuild/ 7.JVM将Boolean类型看作是int类型，true就是1，false就是0，flag如果改成2第二个判1等式就不成立，所以只有第一个判0等式通过。

  2018-12-30

  **6

  **22

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/ed/d50de13c.jpg)

  mj4ever

  Java语言规范中，boolean类型的只有两种 true和false（false是默认值）； Java虚拟机规范中，boolean类型要转换为int类型，true => 1，false => 0； 在编译而成的class文件中，除了字段和传入参数外，基本看不出boolean类型： 比如，第一次时为： public class Foo {    public Foo() {    }    public static void main(String[] var0) {        byte flag = 1;        if (flag != 0) {            System.out.println("Hello, Java!");        }        if (flag == 1) {            System.out.println("Hello, JVM!");        }    } } 第2次执行时，正则表达式将flag替换为2： public class Foo {    public Foo() {    }    public static void main(String[] var0) {        byte flag = 2;        if (flag != 0) {            System.out.println("Hello, Java!");        }        if (flag == 1) {            System.out.println("Hello, JVM!");        }    } }

  2018-10-05

  **3

  **20

- ![img](https://static001.geekbang.org/account/avatar/00/11/12/18/83293985.jpg)

  笨笨蛋

  什么时候使用C1，什么时候使用C2，他是怎么区分热点方法的呢？

  作者回复: 刚刚看到一个同学总结了。JVM会统计每个方法被调用了多少次，超过多少次，那就是热点方法。(还有个循环回边计数器，用来编译热循环的。) 默认的分层编译应该是达到两千调C1，达到一万五调C2。

  2018-07-24

  **4

  **19

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLLEr7QAYMfmD6IZxSEHKXTRVGyibI92MLJVmibyQIhwa6yVUtDz9ibsy9kECCSiaaKiaDrjp3tgZkS9Dg/132)

  ace

  最后awk那段分析了下。希望分析正确 也对大家有帮助 1、NR==1,/iconst_1/ 是用于匹配行 匹配第一行到第一个出现iconst_1的行 2、{}进行脚本执行。针对第一步中匹配的行执行内置的字符串函数sub 做替换 3、1 都会被执行。在awk 1被计算为true，表现为默认操作 print $0 也就是打印整行 整体效果是打印所以文本行，但第一个出现iconst_1的做替换。

  2018-08-05

  **

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/11/06/1b/ecd5fffe.jpg)

  那我懂你意思了

  老师，那个pc寄存器，本地方法栈，以及方法栈，java方法栈这三个组成的就是我们常统称的栈吧，然后也叫栈帧？

  作者回复: JVM里的栈指的应该是Java方法栈和本地方法栈。每个方法调用会在栈上划出一块作为栈帧(stack frame)。栈是由多个栈帧构成的，就好比电影是由一个个帧构成的。

  2018-07-20

  **3

  **16

- ![img](https://static001.geekbang.org/account/avatar/00/11/33/58/1f1e33d5.jpg)

  踏雪无痕

  您好，我现在所在的项目经常堆外内存占用非常多，超过总内存的70%，请问一下有没有什么方法能观察一下堆外内存有什么内容？

  作者回复: 堆外内存的话，就把JVM当成普通进程来查找内存泄漏。可以看下Google Performance Tools相关资料

  2018-07-20

  **

  **15

- ![img](https://static001.geekbang.org/account/avatar/00/10/6c/28/a1f9f0ad.jpg)

  陈树义

  asmtools.jar 是在哪里下载的，怎么在给的链接页面没找到。

  2018-07-21

  **1

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/0f/57/38/ba6a106f.jpg)

  Phoenix

  解释执行是将字节码翻译为机器码，JIT也是将字节码翻译为机器码，为什么JIT就比解释执行要快这么多？ 如果说JIT检测到是热点代码并且进行优化，那么为什么解释执行不直接就用这种优化去解释字节码？ 一些比较浅的问题，希望老师能指点一二

  作者回复: 1. 就单条加法字节码而言，解释执行器需要识别字节码，然后将两个操作数从Java方法栈上读取出来并相加，最后将结果存入Java方法栈中。而JIT生成的机器码就只是一个CPU加法指令。 2. 因为JIT比较费时。如果字节码需要JIT后才跑，那么启动性能会很糟糕

  2018-09-16

  **

  **12

- ![img](https://static001.geekbang.org/account/avatar/00/10/1d/6f/0e552a48.jpg)

  志文

  Java 作为一门高级程序语言，它的语法非常复杂，抽象程度非常高，所以不能直接在硬件上执行。所以要引入JAVA虚拟机。 我觉得理由不充分，JAVA为什么不能像c++一样直接转成机器码呢？从理论上是可以用编译器来实现这个的功能的。问题在于直接像c++那样编译成机器码，就实现不了跨平台了。那么是不是跨平台才是引入JAVA虚拟机的重要原因呢 。请老师解答

  2018-07-22

  **3

  **12

- ![img](https://static001.geekbang.org/account/avatar/00/10/00/bd/c9b8252a.jpg)

  suzuiyue

  JIT程序重启之后还需要再来一遍吗？

  作者回复: 程序关闭后，即时编译的结果就没了，因此需要再来一遍

  2018-09-18

  **

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/12/07/89/c493c1a0.jpg)

  欲风

  方法区和元空间是一个概念吧，能不能统一说法到jdk8之后的版本～

  2018-07-20

  **1

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/29/3806fe23.jpg)

  临风

  我跟楼上的novembersky同学一样疑惑，对于性能要求高的web应用，为什么不直接使用即时编译器在启动时全部编译成机器码呢？虽然启动耗时，但是也是可以接受的

  作者回复: 通常，对于长时间运行的程序来说，大部分即时编译就发生在前几个小时。 再之后的即时编译主要是一些非热点代码，以及即时编译器中的bug造成的反复去优化重新编译。

  2018-07-20

  **

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/12/be/d4/ff1c1319.jpg)

  金龟

  感觉看完后，解释执行和jit的区别还是有点没搞懂。解释执行的意思是:直接将整个字节码码文件转化成机器码，jit的意思是:用到哪段编译哪段?

  作者回复: 反过来。解释执行相当于同声传译，你说一句我翻一句给观众(CPU)听。JIT是线下翻译，可以花时间精简掉你的口语话表达(做编译优化)

  2018-11-24

  **2

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/11/fe/fb/ae860669.jpg)

  Kouichi

  为啥是"理论"上比cpp快...这样看起来 如果都编译成机器码了 应该就是挺快的呀... 那干啥不像Go一样 直接编译成目标平台的机器码... 咋感觉绕了一圈..

  作者回复: 因为实际上会插入一些虚拟机相关的代码，稍微拉低了运行效率。 至于为什么不采用直接编译的方法，在峰值性能差不多的这个前提下，线下编译和即时编译就是两种选项，各有优缺点。JVM这样做，主要也是看重字节码的可移植性，而牺牲了启动性能。 另外呢，现代工程语言实现都是抄来抄去的。JVM也引入了AOT编译，在线下将Java代码编译成可链接库。

  2018-07-20

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/13/66/98/95433f67.jpg)

  shuwei

  留言超精彩，一定要看!!!

  2018-11-28

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/61/677e8f92.jpg)

  xianhai

  即时编译生成的代码是只保存在内存中吗？会不会写到磁盘上？如果我怀疑优化后的代码有bug，有办法debug吗？

  2018-07-21

  **1

  **7

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  雾里听风

  理论上讲，即时编译后的 Java 程序的执行效率，是可能超过 C++ 程序的。 我们导师当时是这么解释的，c是所有CPU指令集的交集，而jit可以根据当前的CPU进行优化，调用交集之外的CPU指令集，往往这部分指令集效率很高。 作者如何看待这句话？

  作者回复: 这句话不准确。现代编译器一般都分为平台无关的前端和平台相关的后端。如果你要生成某个平台的代码，编译器会选择相应的后端。因此，无论是C编译器还是JIT编译器，都是基于目标CPU的指令集来做优化的。

  2018-07-20

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/10/da/d9/f051962f.jpg)

  曾泽浩

  解释执行和即时执行这里听得有点蒙

  2018-07-20

  **1

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8e/14/7281bdfa.jpg)

  Wi1ls努力努力再努力

  也可以二进制打开.class文件，直接修改字节码文件041b99修改为051b99，与 awk 一样的效果

  2019-04-22

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/11/ff/1e/2a40a5ca.jpg)

  大明

  对不起，听了29篇文章了，至今不太清楚hotspot和openjdk两者之间的关系。

  作者回复: HotSpot是JVM里的引擎，可以理解为JDK中用C++写的部分。Oracle JDK/OpenJDK包括HotSpot。

  2018-09-26

  **3

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/12/0f/66/b188ad18.jpg)

  Bert.zhu

  老师，对于jvm的即时编译，当方法里有很多if,elseif这样的判断，jvm也是整个方法进行编译，还是只部分编译？

  作者回复: JVM有两种编译方式，整个方法进行编译，或者对热循环进行编译。后面那种涉及到一个叫on stack replacement的技术。不论是那种，都要比if else的粒度大。

  2018-07-20

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/11/f8/f3/2f4f8eb7.jpg)

  Fyypumpkin

  老师，问一下这个asmtools是做什么用的

  作者回复: 就是Java字节码的反汇编器和汇编器。

  2018-07-20

  **

  **5

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/SttQqfuIiazh8ZISZjWibV5fQk67T0fVBwmDKuHicWBEBiaBhHzXUs9IGBI3gyljEAM96X5aibTpVdTALNpIbxPUFCg/132)

  世界和平

  你好  我想问下  解释执行 Java 字节码后，再次执行到这里，还需要再次编译吗

  作者回复: 在即时编译器完成该方法的编译前，每次执行这里都需要解释执行。编译完成后，也有可能因为种种原因导致虚拟机抛弃这一编译结果。这时候就有可能触发再次编译

  2020-01-09

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/13/2f/93/0d4a353b.jpg)

  YJ

  测试了一下，从iconst_1, ..., iconst_5都可以编译成功，但是iconst_6以上就会报错 ``` ➜  jvm-learn grep -n "iconst_6" Foo.jasm 18:		iconst_6; ➜  jvm-learn java -cp asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm jasm: null null fatal exception Foo.jasm: [18, 253] 1 error ``` 请问有人知道为什么吗？

  作者回复: 因为6以上没有对应的字节码了，需要用bipush Sipush ldc 了

  2018-11-13

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c3/df/a704066e.jpg)

  崔龙龙

  对于占据大部分的不常用的代码，我们无需耗费时间将其编译成机器码，而是采取解释执行的方式运行； 这是否意味着不常用的代码的多次调用就要多次进行解释执行

  作者回复: 调用到一定次数就会触发即时编译的

  2018-11-12

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/08/4e/87e40222.jpg)

  Yoph

  awk 'NR==1,/iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.1 > Foo.jasm是将字节码中的 iconst_1修改为了iconst_2，如下代码： iconst_2;//原本为iconst_1 istore_1; iload_1; ifeq	L14; //ifeq当栈顶数值等0时就跳转L14 getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;"; ldc	String "Hello, Java!"; invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V"; L14:	stack_frame_type append; locals_map int; iload_1; iconst_1; if_icmpne	L27; //比较栈顶两数值结果不等0时就跳转L27 getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;"; ldc	String "Hello, JVM!"; invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V"; L27:	stack_frame_type same;

  2018-10-30

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/40/01/de9f91d8.jpg)

  spectator

  按照这个说法，编译为机器码的方法都在JVM外执行，那么这部分的方法是在内存的什么位置，又是通过何种方式被JVM调用的

  2018-10-26

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/12/09/2d/ba294b1c.jpg)

  laoyaozhang

  有一点看了好几个相关文章都没提到，即时编译生成的机器码一般是存到哪里的？是内存吗？ 如果是内存的话，是在给JvM分配的内存呢？还是再单独开辟一块内存？

  2018-07-22

  **1

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/e4/8d/66eb10ca.jpg)

  mingrui

  安卓ART虚拟机和其他Java虚拟机有什么区别

  2018-07-21

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/d0/00/9d05af66.jpg)

  加多

  方法区是不是属于堆的一部分？

  作者回复: 不属于。JVM中的堆是用来存放Java对象的。

  2018-07-20

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/fd/a1/de24483a.jpg)

  ᠬᠢᠶᠠᠨ 祁颜·恩和ᠡᠩᠬᠡ

  java10的javafx能ppapi吗？我网上没查到该资料。告诉我呗

  2018-07-20

  **

  **4

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKlUTMg2fe8GicddkOJz7MDCt078opia2sPEM2p8X1yE2icMDziaScL1NjFu00uurj05rwwxP34Fn3mTw/132)

  小寅

  评论太精彩了，多谢大佬们提的高质量问题

  2020-01-03

  **

  **3

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q3auHgzwzM7CIlLxbun5LFhKnPM82kMdxdDqp5qXPgStIdrBsq0G54hVLAVCiaDsuH56x63oG8RVp9zz926LxhA/132)

  Geek_15237d

  Java虚拟机是如何执行Java字节码的？ 答:从虚拟机的视角看，执行Java代码首先会将它编译好的以.class结尾的字节码文件加载到虚拟机中并存放在方法区中，虚拟机实际执行的是方法区内的代码。虚拟会把栈细分为运行Java方法的Java方法区，运行本地代码的本地方法区和记录各个线程运行位置的PC寄存，当调用Java方法时，会在Java方法栈中创建一个栈桢，存放运行时的局部变量和字节码的操作数，Java虚拟机不要求栈桢是一个连续的内存结构。当程序结束，无论正常还是异常结束，虚拟机都会弹出栈桢元素，删除栈桢。 从硬件角度来看，Java字节码并不能在硬件环境上执行，需要将字节码转换为机器码。有两种转换方式，一是解释执行，逐行解释并执行，二是即时编译，即把Java方法作为整天进行编译然后再执行，前者不需要等待编译后者运行效率高。hotspot采用了混合方式，即先把字节码逐行解释并执行，然后随遇一些热点代码在进行即时编译。

  2019-04-25

  **

  **3

- ![img](http://thirdwx.qlogo.cn/mmopen/eKfCXGrAiaUgRmHMr1q0rDOAibCpy6p7bFQkjWEooOMQNv4JUcic1X6XoQDNO3eWK7QlzSzKwxRDmzffKlHic4rQxRLFawiaICQ7N/132)

  jikelinj

  老师我看的疑问更多了，我看留言中也有一部分和我一样的疑惑。我使用jdk1.7和1.8都试过了，两个都可以打印，真心求一下困惑解答

  2018-08-01

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/73/01598bd5.jpg)

  Lilee

  你好，可以问一下虚拟机是怎么判断某一段代码是否为热点代码的吗？

  2018-07-20

  **1

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/cb/ce/d9e00eb5.jpg)

  undefined

  编译了一个 asmtools.jar 下面链接可取 https://pan.baidu.com/s/1n8G2Hpbowd0soMIutnFWSw 提取码: 8fqv

  2021-03-16

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/67/17/942e5115.jpg)

  wuyong

  asmtools.jar 下载地址： https://github.com/eclipse/openj9/pull/1152 https://ci.adoptopenjdk.net/view/Dependencies/job/asmtools/lastSuccessfulBuild/artifact/ 

  2020-08-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/14/06eff9a4.jpg)

  Jerry银银

  文章中，有这样一句话：“如果你熟悉 X86 的话，你会发现这和段式内存管理中的代码段类似。而且，Java 虚拟机同样也在内存中划分出堆和栈来存储运行时数据”。 我的理解是是跟操作系统的内存管理相似，但为什么说是“x86"呢？然后，我也知道，当然操作系统都是使用的分页的方式，这里的“段式内存管理”具体是怎么样的概念？

  2019-12-21

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/12/37/2f671271.jpg)

  shyan1

  为什么 Java 要在虚拟机里运行？ Java 作为一门高级程序语言，它的语法非常复杂，抽象程度也很高。因此，直接在硬件上运行这种复杂的程序并不现实。所以呢，在运行 Java 程序之前，我们需要对其进行一番转换。 这个理由太牵强了吧 

  2019-03-15

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/27/54/d38c34a0.jpg)

  小泷哥

  两点∶ 1.无论是c1,c2,graal都只编译方法，不是整个类 2.编译后的机器码放在内存 疑惑： 1.为什么不保存下来？ 2.如果说是因为虚函数导致每次都需要重新编译，那没有设计虚函数的方法是否能保存下来？

  作者回复: 可以保存下来的，你可以查查jaotc这个工具的资料。第36篇也有部分介绍。 这样做的话，JVM需要确保你所执行的字节码和你编译后的机器码是否吻合。主要有两种情况会造成不吻合，一是换了库版本造成的代码本身不同，二是动态字节码注入造成实际运行的字节码和原字节码不同

  2018-12-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/6e/b6/fe81c712.jpg)

  菩提树下的灵猴

  这么多留言，就没有想要这位老师的微信号？联系方式？老师方便留个联系方式吗？

  2018-10-25

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/0e/61/ae68f8eb.jpg)

  dream

  请问一下这段话是什么意思：$ awk 'NR==1,/iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.1 > Foo.jasm，我知道awk是类似于vim的记事本工具，但是这段代码到底做了什么不理解

  作者回复: 你可以理解为把文件中第一次出现的iconst_1改为iconst_2

  2018-10-11

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/85/7c/03a268fe.jpg)

  leohuachao

  java -cp /path/to/asmtools.jar org.openjdk.asmtools.jdis.Main Foo.class > Foo.jasm.1 可以简写为 java -jar /path/to/asmtools.jar jdis Foo.class > Foo.jasm.1

  2018-09-19

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/ff/d1/28adb620.jpg)

  蒙奇•D•273°

  静态方法和实例方法都是如何分配的

  2018-07-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/12/18/83293985.jpg)

  笨笨蛋

  搞不懂，没有讲清楚堆栈到底如何共享？有些文章说栈数据共享，但又说每个线程都会有一个堆栈，那堆栈的数据还如何共享？还有堆有时候说数据不共享，但又说线程间数据共享？这老师能解答一下吗？

  作者回复: 线程各自的栈空间是不共享的，但可以通过堆空间来共享数据。如果只有一个线程知道某个数据存放在堆的哪个位置，那也相当于不共享。注意不是等同于不共享，因为其它线程可以扫描整个堆，来找到这个位置。

  2018-07-24

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/81/d4/e92abeb4.jpg)

  Jecy-8

  我一直以为方法区是从推划出的一部分，用来存放类和静态信息等，方法区中又包含常量池😅

  作者回复: 这种理解也没啥问题，都是概念上的。说不定哪天我司里的架构师就说方法区属于堆了

  2018-07-24

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/07/ea/b82fd545.jpg)

  abs

  我懂了，最后一个命令，又重新编译为class文件了

  2018-07-20

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/07/ea/b82fd545.jpg)

  abs

  请问下练习中生成了jasm文件，没有把更改写入Foo.class是怎么对结果造成影响的呢？

  2018-07-20

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  风到哪里去了

  asmtools.jar 无法下载 可以用什么代替么

  2019-05-24

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/2b/58/11c05ccb.jpg)

  布衣骇客

  来晚了 ，还是很期待，JVM一直是可望而不可即的，现在被老师讲得通透易懂，不那么怕了，目标更加明确了

  2018-12-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/7f/88/c4263c58.jpg)

  五年

  练习题的代码命令可以给注释解释就好了,百度也百度不到什么意思...

  作者回复: 那条awk命令是找到文本中第一条iconst1，替换成iconst2

  2018-11-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/4f/28/1f086e9a.jpg)

  rachel

  java程序需要运行在虚拟机的原因？   我觉得主要是为了避免系统语言（例如C语言、C++等）与平台的强耦合性，不易移植的缺点，从而实现“一次编写，到处运行”的平台无关性。

  2018-09-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/ff/b0/dca98a0d.jpg)

  G1

  元空间和永久代只是不同版本下的方法区实现，所以原文中用方法区称呼是正确的

  2018-08-20

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  网络已断开

  ​                iconst_2;                istore_1;                iload_1;                ifeq    L14;                getstatic       Field java/lang/System.out:"Ljava/io/PrintStream;";                ldc     String "Hello,Java!";                invokevirtual   Method java/io/PrintStream.println:"(Ljava/lang/String;)V";        L14:    stack_frame_type append;                locals_map int;                iload_1;                iconst_1;                if_icmpne       L27;                getstatic       Field java/lang/System.out:"Ljava/io/PrintStream;";                ldc     String "Hello,JVM!";                invokevirtual   Method java/io/PrintStream.println:"(Ljava/lang/String;)V";        L27:    stack_frame_type same;                return;  line1:入栈2 line2:栈顶int存入第二个局部变量(第一个存放的是this) line3:局部变中量表第二个局部变量入栈 line4:判断栈顶是否为零(为2不为0) line10:局部变中量表第二个局部变量入栈 line11:入栈1 line12:比较栈顶是否相等(1和2自然是不等的)

  2018-08-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/d7/cd/6ebfc468.jpg)

  王刚

  老师我想问下，Java虚拟机指的就是jre吗？ 如果不是Java虚拟机和jre有啥区别嘛？

  2018-07-27

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/e2/89/2c07a3c1.jpg)

  Hizkijah

  1楼说的第二个无法输出，我验证了可以输出啊。

  2018-07-23

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/bb/38/2fb5c4ce.jpg)

  、

  字节码不懂  不会题目

  2018-07-22

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/fd/19/068c5e8c.jpg)

  韶华易逝~

  老师，hotspot分层编译，并使用额外编译线程。那么c1和c2是在什么时候进行编译的啊，应用程序启动时还是启动后的运行时啊？

  2018-07-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/66/f0/830d3c1f.jpg)

  kmax

  开门红，期待后面有一些具体案例实践分析

  2018-07-20

  **

  **1

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  mover

  你好，在回答Ryan-hou时你的回答：这里的直接运行指的是不经过任何转换(编译)，直接在硬件上跑。即便是C++，也不可以直接运行。 这个地方可以展开说一下吗？c++不可以直接执行，why？c++不是编译成机器码了吗？JAVA像c++那样直接编译成目标机器的机器码为什么不现实？

  2018-07-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/03/c9/9a9d82ab.jpg)

  Aaron_涛

  我用jdk8运行了下，两个都输出了呀，我javap看了下是两条ifeq，没有得出上面那个人说的下面没有输出这是为啥，不过我看到了虚拟机对boolean是看成整形

  2018-07-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/1a/2c364284.jpg)

  隔离样

  初中还是高中时真的学的是c+

  作者回复: 哈哈，一般初高中应该使用工程性没那么强的语言吧？

  2018-07-20

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/91/8a/e7d6c8c3.jpg)

  SirNicholas![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  18年刚出来极客时间买了这门课程,记得文中老师说JVM时为了让程序员只需要关注业务代码无需关注虚拟机实现,那请问雨笛老师：现在国内后端面试java基本上大小公司都会面试gc,jvm的底层知识。那对于程序员发展来说,大家把重心或多或少都转移到JVM上，这是不是违背了JVM的初衷呢？本来是解放程序员的事情，现在大家又重新来研究JVM是不是对技术发展起到负反馈作用了呢？我认识的人有大牛但不会专门研究JVM这些，而且大部分人也很难达到提出虚拟机规范实现的标准。如果真要研究JVM最终肯定得回到操作系统，因为os是比JVM还要底层的也是JVM当初发展的基石。那现在大家研究JVM,面试也会问JVM有什么意义呢？直接研究操作系统不是更加直接有效的方法吗？

  2022-06-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/4c/dd/2c0b093f.jpg)

  [嘿哈]

  总结 一、jvm能做什么？        ①  提供Java程序一次编译到处运行的平台；        ②  自动化管理内存；        ③  程序编译成字节码后，虚拟机能够识别字节码程序，将字节码转换成机器码； 二、Jvm内存模型有那些部分？       ①  程序计数器（记录程序执行的下一条指令）       ②  方法区（主要存在类信息）       ③  堆  （通一jvm内的所有线程共享，主要存储运行时产生的对象）       ④  虚拟机栈  （线程独享，主要存储Java方法调用的局部变量）       ⑤  本地方法栈 三、jvm翻译字节码成机器码的方式有那些？       ①  解释执行（将字节码逐条翻译成机器码并执行）       ②  及时编译（JIT，即将一个方法中的所有字节码编译成机器码，在执行） 四、 解释执行与及时编译有那些区别？      ①  及时编译能结合程序运行时的数据，做到最优，但是首次执行时需要等待编译；      ②  解释执行无需等待编译； 总结的不对的，希望老师与同学们多多指正！

  2022-05-10

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  吴晓江

  章尾的小作业，看这篇很有帮助 https://wenjie.store/archives/ant-asmtools-awk

  2022-04-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/b8/96/716ba431.jpg)

  苏成![img](https://static001.geekbang.org/resource/image/bb/65/bba6d75e3ea300b336b4a1e33896a665.png)

  提问，为什么我生成的不是int类型而是byte类型呢？

  2022-04-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/b8/96/716ba431.jpg)

  苏成![img](https://static001.geekbang.org/resource/image/bb/65/bba6d75e3ea300b336b4a1e33896a665.png)

  public class Foo {  public static void main(String[] paramArrayOfString) {    byte b = 2;    if (b != 0)      System.out.println("Hello, Java!");     if (b == 2)      System.out.println("Hello, JVM!");   } } 为什么我运行之后生生产的class是这样啊

  2022-04-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2a/10/83/0facd0eb.jpg)

  利威尔兵长

  .class文件就是Java字节码文件，Java字节码相当于JVM的机器语言。 操作码（opcode）被固定为一个字节。一个字节8位，理论上最多支持256个操作码。java实际上使用了200左右的字节码。还有一些留给调试操作。 虚方法到目标方法的映射，解释执行可能要老老实实再找一遍，而即时编译后已经将目标方法编译好放在那里，无需再进行一次转换。 虚函数的开销之一是要使用函数指针来定位函数入口，比起直接获取函数入口来说，多了一次间接寻址，此时，CPU的指令缓存会被刷新，调用时程序跳转，再一次刷线缓存。 虚函数的另一个开销是，无法进行inline，是性能损失较大的地方，无法inline，就意味着每次函数调用，都得要建立和销毁一次函数栈帧，费时间又费空间。

  2022-03-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2a/e0/0b/6f667b2c.jpg)

  枫林血舞

  特意写了这段代码，通过jclasslib看了下字节码发现：Java中将boolean值看成了常量：iconst_1（常量值int类型，且值为1）。

  2022-03-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/4e/85/ef0108cd.jpg)

  herish

  感兴趣的朋友可以阅读一下我的文章：https://juejin.cn/post/7057503611248443400

  2022-03-01

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTL3xax4aG4h59x50C7LQ5K7BicvIEicakyfE0lV4Pyib6OsYc1jC7Qa37g2v8qhib5BQiaB2DfB4DMG5Cw/132)

  花花世界小人物

  学习总结： 为什么使用jvm 1.一次编译多处运行 2.托管服务 编译方式 1.解释执行 优点：程序执行中无需再编译；缺点：启动时间长。 2.及时编译 优点：编译是有程序的信息，可以做优化峰值性能比较友好；缺点：执行时需要花时间编译 java 7 HotSpot 采用混合编译,及时编译器：c1,c2,graal

  2022-02-21

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_0b457c

  不想开虚拟机不想远程的可以使用 Cmder https://cmder.net/

  2022-01-12

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/1mN1YORUe7JOc4lFnSB0zfuO9KtpHCCwKD6YdlKb8Ncora1t4icf8wCb3owMx0T5M6YRz4cLOge8pQIRgjAapYA/132)

  Geek_885d39

  我在windows电脑上使用命令行javac Foo.java得到的Foo.class文件如下，boolean为啥没有变成int? public class Foo {  public static void main(String[] paramArrayOfString) {    boolean bool = true;    if (bool)      System.out.println("Hello, Java!");     if (bool == true)      System.out.println("Hello, JVM!");   } }

  2021-12-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/6e/8c/49d94d09.jpg)

  和光同尘

  java设计之初不是因为跨平台才加的JVM嘛

  2021-12-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  sotondolphin

  在 HotSpot 里面，上述翻译过程有两种形式：第一种是解释执行，即逐条将字节码翻译成机器码并执行；第二种是即时编译（Just-In-Time compilation，JIT），即将一个方法中包含的所有字节码编译成机器码后再执行。 第一种和第二种的解释是不是颠倒了?

  2021-09-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/24/b6/31/2c102457.jpg)

  徐奔

  作者发的网址我貌似不能访问了，我是去maven仓库里下载的 https://mvnrepository.com/search?q=asmtools

  2021-08-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/20/3a/03/aba3c9b5.jpg)

  迪

  1、Java虚拟机的意义 1.1、“一次编写，到处运行”，一旦一个程序被转换成 Java 字节码，那么它便可以在不同平台上的虚拟机实现里运行； 1.2、虚拟机带来了一个托管环境（Managed Runtime）。这个托管环境能够代替我们处理一些代码中冗长而且容易出错的部分。其中最广为人知的当属自动内存管理与垃圾回收，这部分内容甚至催生了一波垃圾回收调优的业务。除此之外，托管环境还提供了诸如数组越界、动态类型、安全权限等等的动态检测，使我们免于书写这些无关业务逻辑的代码。 2、Java字节码 从虚拟机视角来看，执行 Java 代码首先需要将它编译而成的 class 文件加载到 Java 虚拟机中。加载后的 Java 类会被存放于方法区（Method Area）中。实际运行时，虚拟机会执行方法区内的代码。执行时需要将字节码翻译成机器码，有【解释执行】和【即时编译】(JIT)两种，线程共享： 方法区、堆； 线程私有： PC寄存器、Java方法栈、本地方法栈； 3、Java虚拟机的运行效率 为了提高运行效率，标准 JDK 中的 HotSpot 虚拟机采用的是一种混合执行的策略。它会解释执行 Java 字节码，然后会将其中反复执行的热点代码，以方法为单位进行即时编译，翻译成机器码后直接运行在底层硬件之上。HotSpot 装载了多个不同的即时编译器，以便在编译时间和生成代码的执行效率之间做取舍。

  2021-06-16

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  von

  前者的优势在于无需等待编译，而后者的优势在于实际运行速度更快（为什么后者更快呢？）。

  2021-04-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6e/4c/494b2907.jpg)

  ck

  在windows上下载了gawk. 然后将该文件中awk路径设置为环境变量, 但在执行awk 'NR==1,/iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.1 > Foo.jasm 后抛出如下错误: awk: 'NR==1,/iconst_1/{sub(/iconst_1/, awk: ^ invalid char ''' in expression 将命令中的单引号和双引号改为如下即可: awk "NR==1,/iconst_1/{sub(/iconst_1/, \"iconst_2\")} 1" Foo.jasm.1 > Foo.jasm 

  2021-03-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_86221b

  问个小白问题，字节码解释执行的具体过程是什么，难道是读一个字节码，然后翻译成机器码然后再执行机器码，然后再解释下一个字节码吗

  2020-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/de/6b/adee88bb.jpg)

  宿臾洛城

  因为编译成class文件以后，true被编译成了1，在ams文件中展示的就是iconst_1。 而awk那句话的意思是将第一次出现的iconst_1修改为iconst_2，也就是将被编译成的1改成了2。 所以在转为class文件的时候，boolean值被修改成了2。而其他的没有改变。因为2!=0，所以只输出了 Hello，Java！

  2020-06-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0f/9f/f4b06bd5.jpg)

  见南山

  jvm提供解释执行和即时编译来将java高级语言编译成java字节码class文件，将内存划分为堆栈，栈分为标记线程运行位置的pc计数器，存储方法调用次序的方法栈和本地方法栈。堆则是方法区和堆，方法区存储的就是class文件，堆里面则是申请的对象。

  2020-04-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0f/9f/f4b06bd5.jpg)

  见南山

  记录下: java作为高级语言是无法直接运行在硬件，必须编译为机器码才行。jvm出现提供将java语言转换为中间语言java字节码，更接近机器码，只要编译成字节码就能在各个硬件机器上执行，同时提供了优化，垃圾回收等机制。而像c++则不同直接转换为机器码执行，我理解是与硬件相关的，无法在多平台执行。

  2020-04-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/6d/09/ffeabc27.jpg)

  任鑫

  ”即时编译拥有程序的运行时信息，并且能够根据这个信息做出相应的优化。“，老师以及各位同学，这段应该怎么理解呢？

  2020-04-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  叶中柱

  我理解解释编译同即时编译是不是类比请求是查库跟查内存。    接口性能优化:第一次请求回源查库并写入内存并返回，第二次直接查内存并返回 热点代码:先解释编译，程序执行，若方法执行过于频繁则被即时编译成机器码，再次执行方法时不用编译直接执行机器码 如果理解有偏差还望指正

  2020-04-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/88/58/3e19586a.jpg)

  晓双

  java代码运行是由编译器编译成.class文件。然后由虚拟机读取编译成汇编，等到操作系统来执行的时候会把汇编编译成机器码来执行。不知道是这样不？

  2020-04-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/18/be/ad3127e0.jpg)

  Judy

  jvm像是一个适配器，连接java程序和机器。从而保证java程序的运行。并且为java程序提供了托管环境。

  2020-04-15

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/E3XKbwTv6WTssolgqZjZCkiazHgl2IdBYfwVfAcB7Ff3krsIQeBIBFQLQE1Kw91LFbl3lic2EzgdfNiciaYDlJlELA/132)

  rike

  我的按照步骤执行的，到执行10行java Foo命令时，提示Error: Could not find or load main class Foo，在windows上把编译后能执行的字节码放在centos上也不行，不知怎么回事？

  2020-01-14

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/55/28/31b0cf2f.jpg)

  黑色毛衣

  java 有两种编译方式：JIT 和 AOT，AOT 是没用吗？是因为他是启动的时候全部编译，效率太低？

  2019-10-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/16/e0/7abad3cc.jpg)

  星期八![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  再回头来看，发现全是干货，得好好再看一次了

  2019-10-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/7a/e8/0930b207.jpg)

  文文小杰

  为啥我的用JD-GUI打开class文件都是下面这样的T_T不是你们说的int： public class Foo { public static void main(String[] paramArrayOfString) { boolean bool = true; if (bool) System.out.println("Hello, Java!");  if (bool == true) System.out.println("Hello, JVM!");  } }

  2019-10-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c3/e0/3db22579.jpg)

  技术骨干

  什么是虚方法呢？

  2019-10-21

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0c/52/f25c3636.jpg)

  长脖子树

  第二遍读专栏, 想问个问题, java 解释执行时, 是否会缓存解释后生成的机器码? 或者说为什么不缓存? 和解释性的语言比如 python 是否类似

  2019-10-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c0/6c/29be1864.jpg)

  随心而至

  老师java1.8 jdk 带的javac生成的字节码，由IDEA反编译之后如下，导致我测试的时候两句都没有打印，有什么方法禁用javac的优化吗? public class Foo {    public Foo() {    }     public static void main(String[] var0) {        boolean var1 = true;        if (var1) {            System.out.println("Hello, Java!");        }         if (var1) {            System.out.println("Hello, JVM!");        }     } }

  2019-09-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0d/e9/2f02a383.jpg)

  冬瓜蔡

  编译器将Java文件编译成字节码文件，然后不同平台对应的jvm将字节码转换成平台可识别的机器码，这样就可以将同一套字节码运行在不同平台上，相当于jvm帮助我们屏蔽了不同平台之间的差异。

  2019-08-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/27/06/7ab75a5b.jpg)

  色即是空

  HotSpot，不知道是什么？

  2019-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/a5/db/e4773f61.jpg)

  搬砖小少年

  我这边显示的是下面这一整块东东，哪位道友解释一下，谢谢 $ java Foo ▒▒▒▒: ▒Ҳ▒▒▒▒▒▒޷▒▒▒▒▒▒▒▒▒ Foo

  2019-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5f/27/a6873bc9.jpg)

  我知道了嗯

  字节码是不是就是class文件？如果是的话，那么机器码又是怎么样的

  2019-05-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/cd/a4/bffb956c.jpg)

  余章胜

  为什么 Java 要在虚拟机里运行？这个问题能再讲彻底一年么？

  2019-05-19

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  风到哪里去了

  对于JVM的编译机制会把部分代码编译成机器码，Android从dalvik到ART的虚拟机优化过程中，完全采取了AOT的编译形式。

  2019-05-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/f6/32/c9d8fd60.jpg)

  Snail

  就这么简单的一段Foo.java代码，编译成功之后竟然运行不起来。。。 我是完全照搬文章最后面的操作

  2019-04-26

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q3auHgzwzM7CIlLxbun5LFhKnPM82kMdxdDqp5qXPgStIdrBsq0G54hVLAVCiaDsuH56x63oG8RVp9zz926LxhA/132)

  Geek_15237d

  笔记:Java语言为什么需要再Java虚拟机上运行？答:Java语言是一中高级比较抽象的语言，不能像c++那样直接运行在机器的硬件上，需要经过一系列的转换操作才能运行，而Java虚拟机就是提供这个转换操作的。Java代码首先别编译为.class文件的字节码，这样Java代码就可以在任何平台的虚拟机上运行了。

  2019-04-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/5b/72/4f8a4297.jpg)

  阿May的海绵宝宝

  请问老师,是只有即时编译中才存在指令重排序吧？

  2019-04-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f9/3d/abb7bfe3.jpg)

  Jthan

  【在计息资源充足的情况下，字节码的解释执行和即使编译可同时进行】 1.请问计息资源充足是指？ 2.可以在程序启动时指定某些方法进行JIT吗？ 多谢！

  2019-04-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/57/bf/656a57c6.jpg)

  换个名字

  我把改完后的字节码文件，用反编译工具打开，发现原来的boolean类型已经变成byte了

  2019-03-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/87/4b/16ea3997.jpg)

  tiankonghewo

  栈帧在内存空间都是连续分布的吧？不然怎么重叠呢？

  2019-03-26

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/7d/2e/2bc2ba6f.jpg)

  Prince Baron

  老师，JNI有两款编译器C1和C2，解释器有名字吗？

  2019-03-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/0e/5a/617f28bf.jpg)

  Paul

  Jvm是如何判断哪些代码是解释执行，哪些代码是即时编译的呢？有相应的标准吗？代码全部采用解释执行，性能是不是更好？

  2019-03-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/61/ba/fec75d3a.jpg)

  一去二三里

  看不懂 jasm里面得jvm指令集，这个影响后续得课程吗？

  2019-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/06/63/e75c2ac5.jpg)

  

  public class Foo {  public static void main(String[] paramArrayOfString)  {    int i = 2;    if (i != 0) {      System.out.println("Hello, Java!");    }    if (i != 0) {      System.out.println("Hello, JVM!");    }  } } 为什么jasm反向生成的class，是这样的啊，boolean类型变成了int类型

  2019-03-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/19/49/9452b2fd.jpg)

  Theodore![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  Oracle jdk怎么做这个实验？idea 有对应asm插件怎么用？

  2019-03-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/56/15/45becc7f.jpg)

  ‭vayi

  asmtools下载下来后自己要用ant构建一下才能得到jar的。。。

  2019-03-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/2c/62/b5fcbf2b.jpg)

  Mark

  后期调试里不顺辣么多线程，线程诞生的方式是不是有点问题 TreadFactory 了解下

  2019-03-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/7b/57/a9b04544.jpg)

  QQ怪

  栈祯中存放操作数，这个操作数是什么？

  2019-03-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/48/b0/f9ad2d76.jpg)

  Abc简简简简

  小结：java代码是怎样运行的？可以从两个角度回答，一个人虚拟机的角度加载运行，一个是硬件角度，解释或编译

  2019-03-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/57/6e/b6795c44.jpg)

  夜空中最亮的星

  国外的海关官员知识面都这么广吗？还好订阅了老师的课，否则就拒签了，😄

  2019-02-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/df/9f/6e3e1b77.jpg)

  阿U

  老师 我执行完那段 java -jar asmtools.jar jdis Foo.class > Foo.jasm.1之后报错 Exception in thread "main" java.lang.UnsupportedClassVersionError:org/openjdk/asmtools/Main : Unsupported major.minor version 52.0 我网上查了一下说是  jkd和jre版本不一致 我看了 我的是一样的

  2019-02-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ec/13/49e98289.jpg)

  neohope

  刚开始看您这一系列的文章，感觉写的真棒！ 我想请教几个问题： 1、在JVM里面，代码被JIT编译后，JVM是如何切换到机器码的呢？ 2、C1编译后，字节码是不会被清理出内存的，这样理解对吗？ 3、那C2编译后，C1编译的代码是否会被清理出内存呢？ 感谢！

  2019-02-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/a9/90/0c5ed3d9.jpg)

  颇忒妥

  老师，您在文章里讲到C1（client）和C2（server）两种编译器，那么在启动JVM的时候添加了-server参数，那么JVM是依然会同时使用C1、C2，还是说只会使用C2呢？

  2019-01-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/46/c4/128338f8.jpg)

  阿姐的阿杰

  请教老师：「栈帧的大小是提前计算好的」请问是如何确定栈帧的大小的呢？困扰了好久。

  2019-01-16

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/b4/82/62548de6.jpg)

  jack Wong

  郑老师你好，关于翻译出来的jasm文件有个问题想请教一下 super public class Foo version 52:0 {  public Method "<init>":"()V" stack 1 locals 1 { 	aload_0; 	invokespecial	Method java/lang/Object."<init>":"()V"; 	return; } public static Method main:"([Ljava/lang/String;)V" stack 2 locals 2 { 	iconst_1; 	istore_1; 	iload_1; 	ifeq	L14; 	getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;"; 	ldc	String "Hello, Java!"; 	invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V"; L14:	stack_frame_type append; 	locals_map int; 	iload_1; 	iconst_1; 	if_icmpne	L27; 	getstatic	Field java/lang/System.out:"Ljava/io/PrintStream;"; 	ldc	String "Hello, JVM!"; 	invokevirtual	Method java/io/PrintStream.println:"(Ljava/lang/String;)V"; L27:	stack_frame_type same; 	return; } } // end Class Foo 有两个iconst_1,我尝试两个都改成iconst_2,发现两个if都可以进入，如果我就将第一个iconst_1改成iconst_2那就可以，其实这两个iconst_1有什么不一样呢

  2019-01-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/ef/4f/8b7e1f1e.jpg)

  roie

  我在这节老师练习代码，最后一个指令java Foo 操作爆出这个错误，不知道什么意思？大佬帮忙解决下，错误如下： Error: A JNI error has occurred, please check your installation and try again Exception in thread "main" java.lang.VerifyError: Operand stack underflow Exception Details:  Location:    Foo.main([Ljava/lang/String;)V @0: istore_1  Reason:    Attempt to pop empty stack.  Current Frame:    bci: @0    flags: { }    locals: { '[Ljava/lang/String;' }    stack: { }  Bytecode:    0x0000000: 3c1b 9900 0bb2 0003 1201 b600 0504 1ba0    0x0000010: 000b b200 0312 02b6 0005 b1  Stackmap Table:    append_frame(@13,Integer)    same_frame(@26)         at java.lang.Class.getDeclaredMethods0(Native Method)        at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)        at java.lang.Class.privateGetMethodRecursive(Class.java:3048)        at java.lang.Class.getMethod0(Class.java:3018)        at java.lang.Class.getMethod(Class.java:1784)        at sun.launcher.LauncherHelper.validateMainClass(LauncherHelper.java:544 )        at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:526)

  2019-01-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/b4/82/62548de6.jpg)

  jack Wong

  想请教一下作者大大，文中提及到的栈帧是不是jvm 中每个线程的私有内存空间呢？主内存的各个副本

  2019-01-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/f3/69/7039d03f.jpg)

  书策稠浊

  Java是一次编写，一次编译，到处运行，c++是一次编写，多次编译，到处运行，不同的只是编译而已，常用的操作系统就这么几种，根本不需要编译多少次。所以我觉得用可移植性来说Java的优势有点说不过去。

  2019-01-08

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLwSoTjHPX5tm4whBSfoZLX6toZxrZGUaLABQywKNf4MDc9toK3QSV7Z99ATcGicFCysoleQ5ISzmw/132)

  乘风

  老师问个问题：  jvm编译过程有两个：1.解释执行 2.即时编译，即时编译将直接将字节码翻译为机器码，那为什么不在启动时直接将jar里的文件字节码翻译为机器码，然后在做执行，这样效率会更快吗？

  2018-12-25

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/a6/ff/4a8c99f1.jpg)

  贾鹤鸣   

  文中提到 `从 Java 7 开始，HotSpot 默认采用分层编译的方式` 我网上找了下貌似，Java 8 默认是关闭这种方式的，有没有直接展示各版本差异对比，以及验证方法的地方呢。

  2018-12-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/5b/73/23040664.jpg)

  summer_Day

  厉害

  2018-12-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/71/e0/ebe7d07c.jpg)

  easy

  老师，您好，堆内存dump出来百分之98都是不可达对象，怎么回事？

  2018-12-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/9c/c5/3de92354.jpg)

  Spark

  每当调用一个Java方法，虚拟机会在该线程的Java方法栈中生成一个栈帧，用以存放局部变量和字节码的操作数。请问，字节码的操作数是什么？

  2018-12-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/89/09/d660509d.jpg)

  intuition

  老师你好，我有个地方还是想不通，为什么java采用一次编译，到处运行的这种方式，而不是C++的不同平台都进行编译， java这样设计 加了中间层 反而执行效率降低，那这种设计的初衷是什么呢？  

  作者回复: 个人感觉应该是静态编译的各种语言中C++比较突出，一次编译到处运行的各种语言中Java比较典型。 你可以用LLVM把C++编译成bitcode到处运行，也可以用AOT把Java编译成机器码。只不过不是那么”流行”

  2018-11-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/e3/1a/17d8ec44.jpg)

  Rophie

  为什么asmtools 下载不了？ 那位能够共享一下？

  2018-10-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/10/bb/f1061601.jpg)

  Demon.Lee![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  “举例来说，下图的中间列，正是用Java写的Helloworld程序编译而成的字节码。” ...... “# 最左列是偏移；中间列是虚拟机读的机器码；最右侧是给人读的代码 0x00:  b2 00 02     getstatic java.lang.System.out” ...... ------------------- 这里是否有误，一个写的是字节码，一个又说是机器码，上面那句话里的是机器码吧？还是虚拟机读的是字节码？ 

  作者回复: 如果把虚拟机也当作一个机器，它所接收的代码格式就可以叫做”机器码”。Java虚拟机的机器码，我们有一个约定俗成的名字叫Java字节码

  2018-10-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/5c/85/921f4f99.jpg)

  有凤来仪

  老师问个问题，java代码最终运行在jvm还是操作系统呢？如果是jvm，为啥还要转为机器码呢

  2018-10-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/0e/61/ae68f8eb.jpg)

  dream

  最后小作业中：$ awk 'NR==1,/iconst_1/{sub(/iconst_1/, "iconst_2")} 1' Foo.jasm.1 > Foo.jasm  这个命令，我只知道awk 在linux中是类似于vim的与记事本相关的命令，但是中间的命令到底做了什么一直不明白，经过查资料归纳如下，不对的希望多加指正： awk是linux下一个强大的文本分析工具 NR(Number of Record)：行号，当前处理的文本行的行号。 sub(regex,sub,string)sub 函数执行一次子串替换。它将第一次出现的子串用 regex 替换。第三个参数是可选的，默认为 $0。 $0	完整的输入记录 $n	当前记录的第n个字段，字段间由FS分隔 iconst_1 为 pattern /iconst_1/   两个/表示正则表达式的开始结束符号 最后那个1为定值，非0 参考资料如下： https://blog.csdn.net/xp5xp6/article/details/50531396 http://www.runoob.com/linux/linux-comm-awk.html http://www.runoob.com/w3cnote/awk-built-in-functions.html 但是把第一个iconst_1替换为iconst_2有什么用呢？有点不理解。

  作者回复: 第二篇有讲解。boolean在字节码中被映射为int类型。这里我们在尝试整数2是true还是false，还是it depends。

  2018-10-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5a/d7/2bcc13a5.jpg)

  chenfei

  你好，既然类都被加载到方法区了，反射调用是否有可能直接访问方法区，还是要必须访问类型类对象？

  2018-10-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/30/af/86684a0e.jpg)

  maytwo

  老师，运行在x86的java程序，怎样才能运行在arm架构

  作者回复: 下AArch64版本的OpenJDK

  2018-10-01

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  兰传科安卓青铜

  第一种是解释执行，即逐条将字节码翻译成机器码并执行；第二种是即时编译（Just-In-Time compilation... JIT和ART的区别我没太理解，边翻译边执行不是JIT吗？

  2018-09-27

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/11/11/52a78856.jpg)

  D→_→M

  做作业的时候发现自己了解的真的是太少了，又去看了几篇java字节码个awk的文章才勉强把作业搞懂。看了一些awk的用法文章，了解了一些语法，但还不是很懂老师的那天awk命令，老师能给讲解一下吗？

  作者回复: awk就是一个文本处理修改工具，要用时查查man文档即可。 这一条awk指令相当于用记事本打开，搜索iconst1，替换为iconst2。你可以就在记事本这样的GUI程序里操作，也挺方便的。

  2018-09-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/a4/a2/87654b24.jpg)

  L

  既然解释执行的时候都已经将字节码翻译成机器码了，为什么还要再对热点代码进行即时编译呢？为什么不能在解释执行的同时将已经翻译成机器码的代码保存起来，下次执行的时候如果是已经翻译过的代码就直接执行机器码，如果是未翻译过的那就进行解释执行？

  2018-09-18

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/a6/43/cb6ab349.jpg)

  Spring

  即时编译器收集信息会不会产生额外的开销？这个开销会不会随着单位时间内访问次数的增大而增大？Java的动态编译和C++的静态编译都是转换成机器码，Java的优势在哪，运行期间的优化方案有哪些？ 阅读完后我还有这些疑问，麻烦老师解答一下😃

  作者回复: 可以直接看第16 17篇，你的问题基本上都覆盖到了

  2018-09-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9b/24/1e4883c6.jpg)

  dingwood

  “栈帧存储局部变量以及字节码的操作数，栈帧的大小是提前计算好的 ”    ，郑老师，这段话中 字节码的操作数 是指什么？不明白这个概念。 另外 栈帧的大小 在哪可以看到，在linux 用ulimit -a ，显示的stack大小即为栈帧大小？ 两个问题，盼复，谢谢！

  2018-08-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/26/00/2de4f71e.jpg)

  D.Onlyone

  看来我还是太菜，能看懂，就是作业完成不了，😂，还是看的评论才弄好，为啥我就看不懂这些字节码?求老师给推荐个书.

  2018-08-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/29/5c/6e2e1ffd.jpg)

  蓝白之间

  C++如何运行在不同操作系统的呢？比如Linux 和windows 

  2018-08-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/1c/a1/4e063c44.jpg)

  匿名小板凳

  老师好，Java虚拟机在执行时怎么定义热点方法和热点代码的？

  2018-08-06

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eqckmvZia5LKRafjqOOBxk6LmJn273RLnw7giaibD1SsN7o01LKfpRiaUCKH6u7hPyOujT6Jspg3z19lg/132)

  三分热狗

  思路清晰 通俗易懂 受益匪浅

  2018-08-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/85/d0/71bc9d31.jpg)

  hacker time

  解释器的程序能讲解一下吗？

  2018-08-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/38/e6/fa815274.jpg)

  三思

  PC寄存器和栈什么关系啊，一直对这儿不太理解

  2018-08-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/36/3b/20731848.jpg)

  张皮皮

  听之后又重新了解了一些，之前看过，不过忘了，作业暂时还没做

  2018-08-01

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/eKfCXGrAiaUgRmHMr1q0rDOAibCpy6p7bFQkjWEooOMQNv4JUcic1X6XoQDNO3eWK7QlzSzKwxRDmzffKlHic4rQxRLFawiaICQ7N/132)

  jikelinj

  这个平台还不够完善，只能给作者留言。希望老师一定要看到留言给解答一下吧

  2018-08-01

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/eKfCXGrAiaUgRmHMr1q0rDOAibCpy6p7bFQkjWEooOMQNv4JUcic1X6XoQDNO3eWK7QlzSzKwxRDmzffKlHic4rQxRLFawiaICQ7N/132)

  jikelinj

  老师，我的demo怎么两个都可以打印输出？ hello java 和hello jvm都可以，但是我看到留言中有位高手的详解中说第二个不打印，有点郁闷

  作者回复: 在汇编后，也就是调用jasm.Main后，用javap查看一下Foo类的字节码，看有没有iconst_2

  2018-08-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7f/ca/ea85bfdd.jpg)

  helloworld

  终于把JVM中的解释执行和JIT及时编译分清楚了！讲得浅显易懂！

  2018-07-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/7f/ca/ea85bfdd.jpg)

  helloworld

  终于把JVM中的解释执行和JIT及时编译分清楚了！讲得浅显易懂！

  2018-07-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/25/e5/921492c1.jpg)

  Bale

  老实……原谅我做作业慢了 我这边windows是可以做做作业的，首先把.gz里的.zip打开然后进入asmtool6.0/lib中，把asmtools.jar解压到指定位置，然后将作业代码，asmtools的路径对应修改即可。 原理请看评论第一高手

  2018-07-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/2e/a2/cdd182e5.jpg)

  迈克擂

  想请问老师:静态常量池和方法区是怎样的关系呢？Java类加载到方法区，和类中的静态常量和静态块加载的顺序是怎样的呢？希望老师看到能够解答一下。

  2018-07-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/70/7a/64f1c75a.jpg)

  红烧清蒸

  我按作业执行最后还是都打印了。我原来Foo.class的字节码是0:iconst_1，Foo.jasm.1的字节码是iconst_1，Foo.jasm的字节码变成了iconst_2，运行java cp asmtools.jar org.openjdk.asmtools.jasm.Main Foo.jasm后Foo.class字节码变成了0:iconst_2，我想知道wsm没有成功，我用的是jdk1.8，awk版本是20070501，asmtools.jar是通过hg clone拉下来有ant编译的

  2018-07-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5f/83/bb728e53.jpg)

  Douglas

  老师，Java 虚拟机会在当前线程的 Java 方法栈中生成一个栈帧，用以存放局部变量以及字节码的操作数。这个局部变量应该是一个引用吧，真正的值是存储在堆里面。这个帧栈是提前计算好的，那具体有多大，对变量有什么限制吗？

  2018-07-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/5f/83/bb728e53.jpg)

  Douglas

  老师，Java 虚拟机会在当前线程的 Java 方法栈中生成一个栈帧，用以存放局部变量以及字节码的操作数。我想知道这个局部变量可以有多大，如果是个大对象，比如一个含有几百兆的字节数组的对象，比如文件数据，是不是对系统的性能有很大影响？

  2018-07-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6e/64/456c280d.jpg)

  镰仓

  非常好，简洁，打击准确。对于我意愿深入理解Android 虚拟机提供了学习指导。继续学习

  2018-07-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/d1/15/7d47de48.jpg)

  咖啡猫口里的咖啡猫🐱

  老师怎么好编译jdk8，或以后的，，两天了，能不能介绍的文章出处啊😊，我还是喜欢自己有代码调试的感觉

  2018-07-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/04/7a/a15ca9a8.jpg)

  lubibo

  想问一下java方法栈和本地方法栈有什么区别

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/5e/ff/295bcf2c.jpg)

  vimfun

  先编译为Java bytecode 运行时 即时编译 分C1. C2, G. 多种可能 先C1 再 C2  java9有了aot解决启动时间问题 asmtools  jasm/jdis

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/93/64dc7d2a.jpg)

  bosiam

  对于占据大部分的不常用的代码，我们无需耗费时间将其编译成机器码，而是采取解释执行的方式运行；另一方面，对于仅占据小部分的热点代码，我们则可以将其编译成机器码，以达到理想的运行速度。 这句话有个疑问： 解释执行的方式运行也是需要编译成机器码的对吧，只不过是逐条编译？

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/b5/b0/7f350c5a.jpg)

  Desperado

  希望案列在手

  2018-07-23

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIyhbzdkFM64HvRATbWjc3hkic7icUszl9hU9hpIMZcibKH4wWam4SHfkcvM7MUoKjGDRrvYGXuvR91Q/132)

  性能

  老师，即时编译是啥算法？编译哪些代码？何时编译完成？为啥我每次压测启动后，top命令查看，同样的代码编译线程工作时长不太一样？

  作者回复: 即时编译就是一个编译器，里面有很多不同的优化，对应不同的算法。触发即时编译用的是JVM维护的统计方法调用次数的计数器。编译时间取决于编译器自己的效率。由于程序的不确定性，在多线程环境下即时编译器干的活可能多可能少。

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/f8/e1/bffdc547.jpg)

  ybin

  想问下，方法中的局部变量是在方法结束时出栈回收，那方法中的循环或是if块中的变量是在循环或是if结束后被出栈被回收呢，还是在方法结束后再回收呢，毕竟if中定义的变量还是不能被if外的代码使用？

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/44/d3c21481.jpg)

  李子

  听上去，栈溢出除了跟方法调用层级有关，也跟方法的局部变量多少大小有关。 有些性能调优会让把线程栈内存调低一点，这是为什么？只是为了能有更多的线程么？

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/dc/7b/3f8d9fc9.jpg)

  秋

  \>> Java 作为一门高级程序语言，它的语法非常复杂，抽象程度也很高..直接在硬件上运行不现实 c++的语法和抽象程度不比Java低，但是在本地跑，感觉这句话不是很准确，逻辑可以调整下

  2018-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/06/fd/494803f2.jpg)

  孙歌

  老师，您好！想问一下方法区存储Java类是采用什么结构形式来存储的呢，还有方法区的Java类会不会被垃圾回收呢？

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/10/44/e7437824.jpg)

  和风暖林

  写得很好呀，期待后面的内容

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/fa/7c/f8f38ad0.jpg)

  小奶狗

  老师，关于题目，为什么执行awk后，反编译Foo.class，反编译后的代码和Foo.java都不同了，表现在“初始化变量为2”了（所以改动后只会打印Hello,Java!），可我们没直接对Foo.class做改动啊。还有用javap -c Foo.class，得到的结果有一段为“Compiled from Foo.jasm”，我觉得应该是"Compiled from Foo.java"才对，到底awk这一步做了什么，好疑惑。

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/d0/00/9d05af66.jpg)

  加多

  请问Jdk8里面的方法区也不属于堆的吗？

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0f/b1/2bf45bc3.jpg)

  xzchaoo

  对于小作业，为何编译器对这两种if产生了不一样的字节码呢？就这个例子而言，两者应该是等价的。

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/04/03/8fde8434.jpg)

  Andy

  很不错，解决了我许多疑问！大概多久更新一集，希望每天更新一集

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/e2/58/8c8897c8.jpg)

  杨春鹏

  老师，解释执行是逐行将字节码编译成机器码。如何理解为这里的“逐行”。是所谓的一行代码吗，也就是按照分号来划分？

  2018-07-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/02/23/bd476313.jpg)

  小菜鸟

  老师好！问下！何时将字节码加载到jvm呢

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/02/23/bd476313.jpg)

  小菜鸟

  你好！有个疑问问下哈！ 什么时候会将字节码加载到jvm的方法区呢？ 另外您说的编译是指什么？当java文件变为class文件的时候不就是已经编译过了吗？这个时候在jvm中存放的不就是class文件了吗

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f8/87/0491e9e5.jpg)

  Lisa Li

  我可以理解为class文件先被解释执行，等收集到足够的信息之后，C1会对其中的的热点代码进行优化（方法为单位），C2又会对C1中的热点代码(等收集到足够信息）进行优化（方法为单位).对吗？

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/2c/68/c299bc71.jpg)

  天敌

  老师，我的代码报错 java.lang.UnsupportedClassVersionError, 请问是因为我使用jdk1.7的原因吗？如果是，请问我需要采用什么版本的jdk？

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/0d/2b/4814d3db.jpg)

  阿土

  老师你好，看了你的讲解之后终于明白为什么服务刚启动的时候响应速度比较慢，是因为这个时候还没有即时编译完成，请求过一段时间之后即时编译差不多完成性能就有了很大提升。那么除了这个原因还有别的原因导致刚启动的服务比较慢么？

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/61/61/677e8f92.jpg)

  xianhai

  运行asmtools那一步时出错。Foo.class：1 Error invalid character in input

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9d/3b/bc67bfbe.jpg)

  Bin

  hotspot长时间运行后，字节码都会变成机器码吗？还是说有一个比例或者虚拟机只即时编译热点代码，那么虚拟机以什么样的标准去判断热点代码呢？

  2018-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/fd/19/068c5e8c.jpg)

  韶华易逝~

  老师，上述中提到即时编译有c1和c2，主要应对client端型程序和server端长时间运行型程序。7版本的hotspot的又使用额外线程进行分层编译。我迷惑的是c1和c2是什么时候编译，应用启动还是启动后的运行时？

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e7/87/9ad59b98.jpg)

  程序设计的艺术

  在方法内实例化了很多个对象，调用其方法，会不会因为对象过多引起堆内存泄漏或者溢出？谢谢

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e7/87/9ad59b98.jpg)

  程序设计的艺术

  我现在在后台应用中new了很多对象，都是方法内实例化的，如果大并发的话，会不会有问题？

  2018-07-20

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  mover

  理论上讲，即时编译后的 Java 程序的执行效率，是可能超过 C++ 程序的。 你好，理论上讲，可能超过，那么实际上呢，是什么情况？从统计上来看，实现同样的功能，JAVA程序比c++慢多少？慢一倍？10%？

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/62/59/a01a5ddd.jpg)

  ProgramGeek

  关于热点代码通过C2编译器编译，那怎么检测热点代码呢？其次即时编译在一些场景会逆优化退回解释执行，那一般在哪些场景会逆优化呢？

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c3/56/0ad8772a.jpg)

  Axis

  我希望能深度介绍下c2编译器  尤其是sea of node  谢谢

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/c3/56/0ad8772a.jpg)

  Axis

  我希望能介绍一下c2 特别是sea of node的相关论文

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e8/54/a650a973.jpg)

  kyrano

  “Java 虚拟机不要求栈帧在内存空间里连续分布” 能说下底层是如何实现不连续的吗？

  2018-07-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/0d/5c/ef42b1c1.jpg)

  包子

  不全部jit编译的原因，个人觉得是由于边运行边编译可以采集运行时信息用于优化编译。还有其他原因么

  2018-07-20

  **

  **