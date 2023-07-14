# 13 \| 外部函数接口，能不能取代Java本地接口？

作者: 范学雷

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/37/64/37abd32152335e651d120eb4c0927f64.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/d8/06/d83f92afa31ed2c309be5ee28eb47706.mp3" type="audio/mpeg"></audio>

你好，我是范学雷。今天，我们一起来讨论Java的外部函数接口。

Java的外部函数接口这个新特性，我写这篇文章的时候，还在孵化期，还没有发布预览版。由于孵化期的特性还不成熟，不同的版本之间的差异可能会很大。我建议你使用最新版本，现在来说就是JDK 17来体验孵化期的特性。

Java的外部函数接口这个特性，有可能会是Java自诞生以来最重要的两个特性之一，它和外部内存接口一起，会极大地丰富Java语言的生态环境。提前了解一下这样的新特性，有助于我们思考现在的技术手段和未来的技术规划。

我们从阅读案例开始，来看一看Java的外部函数接口为什么可能会带来这么大的影响，以及它能够给我们的代码带来什么样的变化吧。

## 阅读案例

我们知道，像Java或者Go这样的通用编程语言，都需要和其他的编程语言或者环境打交道，比如操作系统或者C语言。Java是通过Java本地接口（Java Native Interface, JNI）来支持这样的做法的。 本地接口，拓展了一门编程语言的生存空间和适用范围。有了本地接口，就不用所有的事情都在这门编程语言内部实现了。

比如下面的代码，就是一个使用Java本地接口实现的“Hello, world!"的小例子。其中的sayHello这个方法，使用了修饰符native，这表明它是一个本地的方法。

<!-- [[[read_end]]] -->

```java
public class HelloWorld {
    static {
        System.loadLibrary("helloWorld");
    }

    public static void main(String[] args) {
        new HelloWorld().sayHello();
    }

    private native void sayHello();
}
```

这个本地方法，可以使用C语言来实现。然后呢，我们需要生成这个本地方法对应的C语言的头文件。

```bash
$ javac -h . HelloWorld.java
```

有了这个自动生成的头文件，我们就知道了C语言里这个方法的定义。然后，我们就能够使用C语言来实现这个方法了。

```c++
#include "jni.h"
#include "HelloWorld.h"
#include <stdio.h>

JNIEXPORT void JNICALL Java_HelloWorld_sayHello(JNIEnv *env, jobject jObj) {
&nbsp; &nbsp; printf("Hello World!\n");
}
```

下一步，我们要把C语言的实现编译、链接放到它的动态库里。这时候，就要使用C语言的编译器了。

```bash
$ gcc -I$(JAVA_HOME)/include -I$(JAVA_HOME)/include/darwin \
      -dynamiclib HelloWorld.c -o libhelloWorld.dylib
```

完成了这一步，我们就可以运行这个Hello World的本地实现了。

```bash
java -cp . -Djava.library.path=. HelloWorld
```

你看，一个简单的“Hello, world!"的本地接口实现，需要经历下面这些步骤：

1. 编写Java语言的代码（HelloWorld.java）；
2. 编译Java语言的代码（HelloWorld.class）；
3. 生成C语言的头文件（HelloWorld.h）；
4. 编写C语言的代码（HelloWorld.c）;
5. 编译、链接C语言的实现（libhelloWorld.dylib）；
6. 运行Java命令，获得结果。

<!-- -->

其实，在Java本地接口的诸多问题中，像代码实现的过程不简洁这样的问题，还属于可以克服的小问题。

Java本地接口面临的比较大的问题有两个。

一个是C语言编译、链接带来的问题，因为Java本地接口实现的动态库是平台相关的，所以就没有了Java语言“一次编译，到处运行”的跨平台优势；另一个问题是，因为逃脱了JVM的语言安全机制，JNI本质上是不安全的。

Java的外部函数接口，是Java语言的设计者试图解决这些问题的一个探索。

## 外部函数接口

Java的外部函数接口是什么样子的呢？下面的代码，就是一个使用Java的外部函数接口实现的“Hello, world!"的小例子。我们来一起看看，Java的外部函数接口是怎么工作的。

```bash
import java.lang.invoke.MethodType;
import jdk.incubator.foreign.*;

public class HelloWorld {
    public static void main(String[] args) throws Throwable {
        try (ResourceScope scope = ResourceScope.newConfinedScope()) {
            CLinker cLinker = CLinker.getInstance();
            MemorySegment helloWorld =
                    CLinker.toCString("Hello, world!\n", scope);
            MethodHandle cPrintf = cLinker.downcallHandle(
                    CLinker.systemLookup().lookup("printf").get(),
                    MethodType.methodType(int.class, MemoryAddress.class),
                    FunctionDescriptor.of(CLinker.C_INT, CLinker.C_POINTER));
            cPrintf.invoke(helloWorld.address());
        }
    }
}
```

在这段代码里，try-with-resource语句里使用的ResourceScope这个类，定义了内存资源的生命周期管理机制。

第8行代码里的CLinker，实现了C语言的应用程序二进制接口（Application Binary Interface，ABI）的调用规则。这个接口的对象，可以用来链接C语言实现的外部函数。

接下来，也就是第12行代码，我们使用CLinker的函数标志符（Symbol）查询功能，查找C语言定义的函数printf。在C语言里，printf这个函数的定义就像下面的代码描述的样子。

```c++
int printf(const char *restrict format, ...);
```

C语言里，printf函数的返回值是整型数据，接收的输入参数是一个可变长参数。如果我们要使用C语言打印“Hello, world!”，这个函数调用的形式就像下面的代码。

```c++
printf("Hello World!\n");
```

接下来的两行代码（第13行和第14行代码），就是要把这个调用形式，表达成Java语言外部函数接口的形式。这里使用了JDK 7引入的MethodType，以及尚处于孵化期的FunctionDescriptor。MethodType定义了后面的Java代码必须遵守的调用规则。而FunctionDescriptor则描述了外部函数必须符合的规范。

好了，到这里，我们找到了C语言定义的函数printf，规定了Java调用代码要遵守的规则，也有了外部函数的规范。调用一个外部函数需要的信息就都齐全了。接下来，我们生成一个Java语言的方法句柄（MethodHandle）（第11行），并且按照前面定义的Java调用规则，使用这个方法句柄（第15行），这样我们就能够访问C语言的printf函数了。

对比阅读案例里使用JNI实现的代码，使用外部函数接口的代码，不再需要编写C代码。当然，也不再需要编译、链接生成C的动态库了。所以，由动态库带来的平台相关的问题，也就不存在了。

## 提升的安全性

更大的惊喜，来自于外部函数接口在安全性方面的提升。

从根本上说，任何Java代码和本地代码之间的交互，都会损害Java平台的完整性。链接到预编译的C函数，本质上是不可靠的。Java运行时，无法保证C函数的签名和Java代码的期望是匹配的。其中一些可能会导致JVM崩溃的错误，这在Java运行时无法阻止，Java代码也没有办法捕获。

而使用JNI代码的本地代码则尤其危险。这样的代码，甚至可以访问JDK的内部，更改不可变数据的数值。允许本地代码绕过Java代码的安全机制，破坏了Java的安全性赖以存在的边界和假设。所以说，JNI本质上是不安全的。

遗憾的是，这种破坏Java为台完整系的风险，对于应用程序开发人员和最终用户来说，几乎是无法察觉的。因为，随着系统的不断丰富，99%的代码来自于夹在JDK和应用程序之间的第三方、第四方、甚至第五方的类库里。

相比之下，大部分外部函数接口的设计则是安全的。一般来说，使用外部函数接口的代码，不会导致JVM的崩溃。也有一部分外部函数接口是不安全的，但是这种不安全性并没有到达JNI那样的严重性。可以说，使用外部函数接口的代码，是Java代码，因此也受到Java安全机制的约束。

## JNI退出的信号

当出现了一个更简单、更安全的方案后，原有的方案很难再有竞争力。外部函数接口正式发布后，JNI的退出可能也就要提上议程了。

在外部函数接口的提案里，我们可以看到这样的描述：

> JNI 机制是如此危险，以至于我们希望库在安全和不安全操作中都更喜欢纯Java的外部函数接口，以便我们可以在默认情况下及时全面禁用JNI。这与使Java平台开箱即用、缺省安全的更广泛的Java路线图是一致的。

安全问题往往具有一票否决权，所以，JNI的退出很可能比我们预期的还要快！

## 总结

好，到这里，我来做个小结。前面，我们讨论了Java的外部函数接口这个尚处于孵化阶段的新特性，对外部函数接口这个新特性有了一个初始的印象。外部内存接口和外部函数接口联系在一起，为我们提供了一个崭新的不同语言之间的协作方案。

如果外部函数接口正式发布出来，我们可能需要考虑切换到外部函数接口，逐步退出传统的、基于JNI的解决方案。

这一次学习的主要目的，就是让你对外部函数接口有一个基本的印象。由于外部函数接口尚处于孵化阶段，所以我们不需要学习它的API。只要知道Java有这个发展方向，目前来说就足够了。

如果面试中聊到了Java的未来，你不妨聊一聊外部内存接口和外部函数接口，它们要解决的问题，以及能带来的变化。

## 思考题

其实，今天的这个新特性，也是练习使用JShell快速学习新技术的一个好机会。我们在前面的讨论里，分析了下面这段代码。为了方便你阅读，我把这段代码重新拷贝到下面了。

```java
try (ResourceScope scope = ResourceScope.newConfinedScope()) {
    CLinker cLinker = CLinker.getInstance();
    MemorySegment helloWorld =
            CLinker.toCString("Hello, world!\n", scope);
    MethodHandle cPrintf = cLinker.downcallHandle(
            CLinker.systemLookup().lookup("printf").get(),
            MethodType.methodType(int.class, MemoryAddress.class),
            FunctionDescriptor.of(CLinker.C_INT, CLinker.C_POINTER));
    cPrintf.invoke(helloWorld.address());
}
```

你能不能找一个你熟悉的C语言标准函数，试着修改上面的代码，快速地验证一下外部函数接口能不能按照你的预期工作？

需要注意的是，要想使用孵化期的JDK技术，需要在JShell里导入孵化期的JDK模块。就像下面的例子这样。

```java
$ jshell --add-modules jdk.incubator.foreign -v
|&nbsp; Welcome to JShell -- Version 17
|&nbsp; For an introduction type: /help intro

jshell> import jdk.incubator.foreign.*;
```

欢迎你在留言区留言、讨论，分享你的阅读体验以及你的设计和代码。我们下节课见！

注：本文使用的完整的代码可以从[GitHub](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/foreign>)下载，你可以通过修改[GitHub](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/foreign>)上[review template](<https://github.com/XueleiFan/java-up/blob/main/src/main/java/co/ivi/jus/foreign/review/xuelei/foreignMemory.jsh>)代码，完成这次的思考题。如果你想要分享你的修改或者想听听评审的意见，请提交一个 GitHub的拉取请求（Pull Request），并把拉取请求的地址贴到留言里。这一小节的拉取请求代码，请在[外部函数接口专用的代码评审目录](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/foreign/review>)下，建一个以你的名字命名的子目录，代码放到你专有的子目录里。比如，我的代码，就放在memory/review/xuelei的目录下面。

