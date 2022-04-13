# [Java关键字(二)——native](https://www.cnblogs.com/ysocean/p/8476933.html)

**目录**

*   [1、JNI：Java Native Interface](https://www.cnblogs.com/ysocean/p/8476933.html#_label0)
*   [3、用C语言编写程序本地方法](https://www.cnblogs.com/ysocean/p/8476933.html#_label1)
    *   [　　一、编写带有 native 声明的方法的java类](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_0)
    *   [　　二、使用 javac 命令编译所编写的java类，生成.class文件](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_1)
    *   [　　三、使用 javah -jni  java类名 生成扩展名为 h 的头文件](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_2)
    *   [　　四、使用C语言实现本地方法](https://www.cnblogs.com/ysocean/p/8476933.html#_label1_3)
*   [4、JNI调用C的流程图](https://www.cnblogs.com/ysocean/p/8476933.html#_label2)
*   [5、native关键字](https://www.cnblogs.com/ysocean/p/8476933.html#_label3)

* * *

　　本篇博客我们将介绍Java中的一个关键字——native。

　　native 关键字在 JDK 源码中很多类中都有，在 Object.java类中，其 getClass() 方法、hashCode()方法、clone() 方法等等都是用 native 关键字修饰的。

| 

1

2

3

 | 

`public` `final` `native` `Class<?> getClass();`

`public` `native` `int` `hashCode();`

`protected` `native` `Object clone() ``throws` `CloneNotSupportedException;`

 |

　　那么为什么要用 native 来修饰方法，这样做有什么用？

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 1、JNI：Java Native Interface

　　在介绍 native 之前，我们先了解什么是 JNI。

　　一般情况下，我们完全可以使用 Java 语言编写程序，但某些情况下，Java 可能会不满足应用程序的需求，或者是不能更好的满足需求，比如：

　　①、标准的 Java 类库不支持应用程序平台所需的平台相关功能。

　　②、我们已经用另一种语言编写了一个类库，如何用Java代码调用？

　　③、某些运行次数特别多的方法代码，为了加快性能，我们需要用更接近硬件的语言（比如汇编）编写。

　　上面这三种需求，其实说到底就是如何用 Java 代码调用不同语言编写的代码。那么 JNI 应运而生了。

　　从Java 1.1开始，Java Native Interface (JNI)标准就成为java平台的一部分，它允许Java代码和其他语言写的代码进行交互。JNI一开始是为了本地已编译语言，尤其是C和C++而设计 的，但是它并不妨碍你使用其他语言，只要调用约定受支持就可以了。使用java与本地已编译的代码交互，通常会丧失平台可移植性。但是，有些情况下这样做是可以接受的，甚至是必须的，比如，使用一些旧的库，与硬件、操作系统进行交互，或者为了提高程序的性能。JNI标准至少保证本地代码能工作在任何Java 虚拟机实现下。

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180307233528546-1790949526.png)

　　通过 JNI，我们就可以通过 Java 程序（代码）调用到操作系统相关的技术实现的库函数，从而与其他技术和系统交互，使用其他技术实现的系统的功能；同时其他技术和系统也可以通过 JNI 提供的相应原生接口开调用 Java 应用系统内部实现的功能。

　　在windows系统上，一般可执行的应用程序都是基于 native 的PE结构，windows上的 JVM 也是基于native结构实现的。Java应用体系都是构建于 JVM 之上。

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180307235217786-1681048194.png)

　　可能有人会问，Java不是跨平台的吗？如果用 JNI，那么程序不就将失去跨平台的优点?确实是这样的。

　　**JNI 的缺点：**

　　①、程序不再跨平台。要想跨平台，必须在不同的系统环境下重新编译本地语言部分。

　　②、程序不再是绝对安全的，本地代码的不当使用可能导致整个程序崩溃。一个通用规则是，你应该让本地方法集中在少数几个类当中。这样就降低了JAVA和C之间的耦合性。

 　　目前来讲使用 JNI 的缺点相对于优点还是可以接受的，可能后面随着 Java 的技术发展，我们不在需要 JNI，但是目前 JDK 还是一直提供对 JNI 标准的支持。

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 3、用C语言编写程序本地方法

　　上面讲解了什么是 JNI，那么我们接下来就写个例子，如何用 Java 代码调用本地的 C 程序。

　　官方文档如下：[https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html](https://docs.oracle.com/javase/8/docs/technotes/guides/jni/spec/jniTOC.html)

　　步骤如下：

　　①、编写带有 **native** 声明的方法的java类，生成.java文件；**(****注意这里出现了 native 声明的方法关键字）** 

　　②、使用 **javac** 命令编译所编写的java类，生成.class文件；

　　③、使用** javah -jni  java类名** 生成扩展名为 h 的头文件，也即生成.h文件；

　　④、使用C/C++（或者其他编程想语言）实现本地方法，创建.h文件的实现，也就是创建.cpp文件实现.h文件中的方法；

　　⑤、将C/C++编写的文件生成动态连接库，生成dll文件；

　　下面我们通过一个 HelloWorld 程序的调用来完成这几个步骤。

　　注意：下面所有操作都是在所有操作都是在目录：D:\JNI 下进行的。

#### 　　一、编写带有 **native** 声明的方法的java类

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　用 native 声明的方法表示告知 JVM 调用，该方法在外部定义，也就是我们会用 C 语言去实现。

　　System.loadLibrary("helloJNI");加载动态库，参数 helloJNI 是动态库的名字。我们可以这样理解：程序中的方法 helloJNI() 在程序中没有实现，但是我们下面要调用这个方法，怎么办呢？我们就需要对这个方法进行初始化，所以用 static 代码块进行初始化。

　　这时候如果我们直接运行该程序，会报“A Java Exception has occurred”错误：

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308002259932-1479682705.png)

#### 　　二、使用 **javac** 命令编译所编写的java类，生成.class文件

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003043572-1546432391.png)

　　执行上述命令后，生成 HelloJNI.class 文件：

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003151699-1569400870.png)

#### 　　三、使用** javah -jni  java类名** 生成扩展名为 h 的头文件

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003250559-2084750367.png)

　　执行上述命令后，在 D:/JNI 目录下多出了个 HelloJNI.h 文件：

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308003335427-592698657.png)

#### 　　四、使用C语言实现本地方法

　　如果不想安装visual studio 的，我们需要在 windows平台安装 gcc。

　　安装教程如下：http://blog.csdn.net/altland/article/details/63252757

　　注意安装版本的选择，根据系统是32位还是64位来选择。[64位点击下载](https://sourceforge.net/projects/mingw-w64/files/latest/download?source=files)。

　　安装完成之后注意配置环境变量，在 cmd 中输入 g++ -v，如果出现如下信息，则安装配置完成：

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308013743763-382483401.png)

　　接着输入如下命令：

![](https://images.cnblogs.com/OutliningIndicators/ContractedBlock.gif) View Code

　　-m64表示生成dll库是64位的。后面的路径表示本机安装的JDK路径。生成之后多了一个helloJNI.dll 文件

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308215629249-636146276.png)

　　最后运行 HelloJNI：输出 Hello JNI! **大功告成。**

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308014528999-1880600112.png)

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 4、JNI调用C的流程图

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180308220935516-668760521.png)

　　图片引用自：https://www.cnblogs.com/Qian123/p/5702574.html

[回到顶部](https://www.cnblogs.com/ysocean/p/8476933.html#_labelTop)

### 5、native关键字

　　通过上面介绍了那么多JNI的知识，终于到介绍本篇文章的主角——native 关键字了。相信大家看完上面的介绍，应该也是知道什么是 native 了吧。

　　native 用来修饰方法，用 native 声明的方法表示告知 JVM 调用，该方法在外部定义，我们可以用任何语言去实现它。 简单地讲，一个native Method就是一个 Java 调用非 Java 代码的接口。

　　native 语法：

　　①、修饰方法的位置必须在返回类型之前，和其余的方法控制符前后关系不受限制。

　　②、不能用 abstract 修饰，也没有方法体，也没有左右大括号。

　　③、返回值可以是任意类型

　　我们在日常编程中看到native修饰的方法，只需要知道这个方法的作用是什么，至于别的就不用管了，操作系统会给我们实现。