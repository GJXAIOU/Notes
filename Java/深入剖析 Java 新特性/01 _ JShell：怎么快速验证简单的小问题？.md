# 01 \| JShell：怎么快速验证简单的小问题？

作者: 范学雷

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/7a/40/7a59b91ae303335c8da7357936a9d540.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/d7/bf/d74fc51aa625cd46b04bd4d9200e03bf.mp3" type="audio/mpeg"></audio>

你好，我是范学雷。今天，我们聊一聊Java的交互式编程环境，JShell。

JShell这个特性，是在JDK 9正式发布的。从名字我们就能想到，JShell是Java的脚本语言。一门编程语言，为什么还需要支持脚本语言呢？编程语言的脚本语言，会是什么样子的？它又能够给我们带来什么帮助呢？

让我们一起来一层一层地拆解这些问题，弄清楚Java语言的脚本工具是怎么帮助我们提高生产效率的。我们先从阅读案例开始。

## 阅读案例

学习编程语言的时候，我们可能都是从打印“Hello, world!”这个简单的例子开始的。一般来说，Java语言的教科书也是这样的。今天，我们也从这个例子开始，温习一下Java语言第一课里面涉及的知识。

```plain
class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

好了，有了这段可以拷贝的代码，接下来我们该怎么办呢？

首先，我们需要一个文本编辑器，比如vi或者类似于IDEA这样的集成编辑环境，把这段代码记录下来。文本编辑器，每个人都有不同的偏好，每个系统都有不同的偏好。一个软件工程师，可能需要很长时间，才能找到自己顺手的编辑器。就我自己而言，我使用了二十多年vi编辑器，直到这两年才发现IDEA的好。但是使用IDEA的时候，我还是会不自主地敲击vi的命令。不得不说，顺手，确实是一个很顽固、难改的行为习惯。

<!-- [[[read_end]]] -->

回到刚才的正题。有了文本编辑器，接下来，我们要把这段源代码编译成Java的字节码。编译器会帮助我们评估这段代码，看看有没有错误，有没有需要警示的地方。通常，我们使用javac命令行，或者通过集成编辑环境自动编译。

```plain
$ javac HelloWorld.java
```

编译完成之后，我们要运行编译好的字节码，把程序的结果显示出来。在这里，我们一般使用java命令行，或者通过集成编辑环境来运行。

```plain
$ java HelloWorld
```

最后一步，我们要观察运行的结果，检查一下是不是我们期望的结果。

```plain
Hello, world!
```

如果让我去教授Java语言，教到这里，我会让同学们小小地庆祝一下：我们完成了Java语言的第一个程序。

万事开头难，完成Java语言的第一个小程序，尤其难！ 你要学习使用编辑器、使用编译器、使用运行环境。对于一个编程语言的初学者而言，这是迈入Java语言世界的第一步，也是很大的一步。这当然是巨大的收获，一个小小的庆祝当然是应得的也是值得的！

当然，会有同学试着改动这段代码，享受创造的乐趣。比如说，把“Hello, world!”改成“世界你好”或者“How are you?”。 这样一来，我们就还要经历编辑、编译、运行、观察这样的过程。

```plain
class HowAreYou {
    public static void main(String[] args) {
        System.out.println("How are you?");
    }
}
```

毫不意外，对Java的了解更深之后，还会有同学继续修改代码，把System.out换成System.err。然后，同样的过程还要再来一遍：编辑、编译、运行、观察。

其实，编辑、编译、运行、观察这四个步骤，就是我们学习一门新语言或者一项新特性的常规过程。如果你已经有多年的Java语言使用经验，想一想吧，你是怎么学习JDK 7的try-with-resource语句，又是怎么学习JDK 8的Lambda表达式的？是不是也是类似的过程？

也许，你已经习惯了这样的过程，并没有感觉得到有什么不妥当的地方。不过，如果我们看看bash脚本语言的处理，也许你会发现问题所在。

```plain
bash $ echo Hello，World！
Hello, world!
bash $
```

显然，使用bash编写的“Hello, world!”要简单得多。你只需要在命令行输入代码，bash就会自动检查语法，立即打印出结果；它不需要我们调用额外的编辑器、编译器以及解释器。当然，这并不是说bash不需要编译和运行过程。bash只是把这些过程处理得自动化了，不再需要我们手动处理了。

## 拖后腿的学习效率

没有对比，就没有伤害。一般来说，不管是初学者还是熟练的程序员，使用bash都可以快速编写出“Hello, world!”，不到一分钟，我们就可以观察到结果了。但是如果使用Java，一个初学者，也许需要半个小时甚至半天才能看到输出结果；而一个熟练的程序员，也需要几分钟甚至十几分钟才能完成整个过程。

这样的学习效率差异并不是无关紧要的。有来自学校的反馈表明，老师和学生放弃Java的最重要的原因，就是学习Java的门槛太高了，尤其是入门第一课。上面的这个小小的“Hello, world!”程序，需要极大的耐心，才能看到最后的结果。这当然影响了新的小伙伴们学习Java的热情。而且，老朋友们学习Java新技术的热情，以及深入学习现有技术的热情，也会受到了极大的阻碍。

JDK 17发布的时候，我们经常可以看到这样的评论，“然而，我还是在使用JDK 8”。确实，没有任何人，也没有任何理由责怪这样的用户。除非有着严格的自律和强烈的好奇心，没有人喜欢学习新东西，尤其是学习门槛比较高的时候。

如果需要半个小时，我们才能看一眼一个新特性的样子，重点是，这个新特性还不一定能对我们有帮助，那很可能我们就懒得去看了。或者，我们也就是看一眼介绍新特性的文档，很难有动手试一试的冲动。最后，我们对它的了解也就仅仅停留在“听过”或者“看过”的程度上，而不是进展到“练过”或者“用过”的程度。

那你试想一下，如果仅仅需要一分钟，我们就能看到一个新特性的样子呢？我想，在稍纵即逝的好奇心消逝之前，我们很有可能会尝试着动动手，看一看探索的成果。

实际上，学习新东西，及时的反馈能够给我们极大的激励，推动着我们深入地探索下去。那Java有没有办法，变得像bash那样，一分钟内就可以展示学习、探索的成果呢？

## 及时反馈的JShell

办法是有的。JShell，也就是Java的交互式编程环境，是Java语言给出的其中一个答案。

JShell API和工具提供了一种在 JShell 状态下交互式评估 Java 编程语言的声明、语句和表达式的方法。JShell 的状态包括不断发展的代码和执行状态。为了便于快速调查和编码，语句和表达式不需要出现在方法中，变量和方法也不需要出现在类中。

我们还是通过例子来理解上面的表述。

### 启动JShell

JShell的工具，是以Java命令的形式出现的。要想启动JShell的交互式编程环境，在控制台shell的命令行中输入Java的脚本语言命令 “ jshell ” 就可以了。

下面的这个例子，显示的就是启动JShell这个命令，以及JShell的反馈结果。

```shell
$ jshell
|&nbsp; Welcome to JShell -- Version 17
|&nbsp; For an introduction type: /help intro

jshell>
```

我们可以看到，JShell启动后，Java的脚本语言就接管了原来的控制台。这时候，我们就可以使用JShell的各种功能了。

另外，JShell的交互式编程环境，还有一个详细模式，能够提供更多的反馈结果。启用这个详尽模式的办法，就是使用“-v”这个命令行参数。我们使用JShell工具的主要目的之一，就是观察评估我们编写的代码片段。因此，我一般倾向于启用详细模式。这样，我就能够观察到更多的细节，有助于我更深入地了解我写的代码片段。

```shell
$ jshell -v
|&nbsp; Welcome to JShell -- Version 17
|&nbsp; For an introduction type: /help intro
```

### 退出JShell

JShell启动后，就接管了原来的控制台。要想重新返回原来的控制台，我们就要退出JShell。退出JShell，需要使用JShell的命令行。

下面的这个例子，显示的就是怎么使用JShell的命令行，也就是“exit”，退出java的交互式编程环境。需要注意的是，JShell的命令行是以斜杠开头的。

```shell
jshell> /exit
|&nbsp; Goodbye
```

### JShell的命令

除了退出命令，我们还可以使用帮助命令，来查看JShell支持的命令。比如，在JDK 17里，帮助命令的显示结果，其中的几行大致是下面这样：

```shell
jshell> /help
|&nbsp; Type a Java language expression, statement, or declaration.
|&nbsp; Or type one of the following commands:
|&nbsp; /list [<name or id>|-all|-start]
|&nbsp; 	list the source you have typed
... snipped ...
|&nbsp; /help [<command>|<subject>]
|&nbsp; 	get information about using the jshell tool
... snipped ...
```

熟悉JShell支持的命令，能给我们带来很大的便利。限于篇幅，我们这里不讨论JShell支持的命令。但是，我希望你可以通过帮助命令，或者其他的文档，了解这些命令。它们可以帮助你更有效率地使用这个工具。

我相信你肯定会对帮助命令显示的第一句话非常感兴趣：输入Java语言的表达式、语句或者声明。下面我们就来重点了解一下这一部分。

## 立即执行的语句

首先，我们来看一看使用JShell来评估Java语言的语句。比如，我们可以使用JShell来完成打印“Hello, world!”这个例子。

```shell
jshell> System.out.println("Hello, world!");
Hello, world!

jshell>
```

可以看到，一旦输入完成，JShell立即就能返回执行的结果，而不再需要编辑器、编译器、解释器。

更方便的是，我们可以使用键盘的上移箭头，编辑上一次或者更前面的内容。如果我们想评估System.out其他的方法，比如不追加行的打印，我们编辑上一次的输入命令，把上面例子中的“println”换成“print”。就像下面这样就可以了。

```shell
jshell> System.out.print("Hello, world!");
Hello, world!
jshell>
```

如果我们使用了错误的方法，或者不合法的语法，JShell也能立即给出提示。

```shell
jshell> System.out.println("Hello, world\!");
|&nbsp; Error:
|&nbsp; illegal escape character
|&nbsp; System.out.println("Hello, world\!");
| &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ^
```

JShell的这种立即执行、及时反馈的特点，毫无疑问地，加快了我们学习和评估简单Java代码的速度，激励着我们去学习更多的东西，更深入的技能。

## 可覆盖的声明

另外，JShell还有一个特别好用的功能。那就是，它支持变量的重复声明。JShell是一个有状态的工具，这样我们就能够很方便地处理多个有关联的语句了。比如说，我们可以先试用一个变量来指代问候语，然后再使用标准输出打印出问候语。

```shell
jshell> String greeting;
greeting ==> null
|&nbsp; created variable greeting : String

jshell> String language = "English";
language ==> "English"
|&nbsp; created variable language : String

jshell> greeting = switch (language) {
&nbsp;&nbsp; ...> &nbsp; &nbsp; case "English" -> "Hello";
&nbsp;&nbsp; ...> &nbsp; &nbsp; case "Spanish" -> "Hola";
&nbsp;&nbsp; ...> &nbsp; &nbsp; case "Chinese" -> "Nihao";
&nbsp;&nbsp; ...> &nbsp; &nbsp; default -> throw new RuntimeException("Unsupported language");
&nbsp;&nbsp; ...> };
greeting ==> "Hello"
|&nbsp; assigned to greeting : String

jshell> System.out.println(greeting);
Hello

jshell>&nbsp;
```

为了更方便地评估，你可以使用JShell运行变量的重复声明和类型变更。比如说，我们可以再次声明只带问候语的变量。

```shell
jshell> String greeting = "Hola";
greeting ==> "Hola"
|&nbsp; modified variable greeting : String
|&nbsp; &nbsp; update overwrote variable greeting : String
```

或者，把这个变量声明成一个其他的类型，以便后续的代码使用。

```shell
jshell> Integer greeting;
greeting ==> null
|&nbsp; replaced variable greeting : Integer
|&nbsp; &nbsp; update overwrote variable greeting : String
```

变量的声明可以重复，也可以转换类型，就像上一个声明并不存在一样。这样的特点和Java的可编译代码有所不同，在可编译的代码里，在一个变量的作用域内，这个变量的类型是不允许转变的，也不允许重复声明。

JShell支持可覆盖的变量，主要是为了简化代码评估，解放我们的大脑。要不然，我们还得记住以前输入的、声明的变量，这可不是一个简单的任务。

也正是因为JShell支持可覆盖的变量，我们才能说JShell支持不断发展的代码，JShell才能够更有效地处理多个关联的语句。

## 独白的表达式

前面我们说过，JShell工具可以接受的输入包括Java语言的表达式、语句或者声明。刚才讨论了语句和声明的例子，现在我们来看看输入表达式是什么样子的。

我们知道，在Java程序里，语句是最小的可执行单位，表达式并不能单独存在。但是，JShell却支持表达式的输入。比如说，输入“1+1”，JShell会直接给出正确的结果。

```shell
jshell> 1 + 1
$1 ==> 2
|&nbsp; created scratch variable $1 : int
```

有了独立的表达式，我们就可以直接评估表达式，而不再需要把它附着在一个语句上了。毫无疑问，这简化了表达式的评估工作，使得我们可以更快地评估表达式。下面的例子，就可以用来探索字符串常量和字符串实例的联系和区别，而不需要复杂的解释性代码。

```shell
jshell> "Hello, world" == "Hello, world"
$2 ==> true
|&nbsp; created scratch variable $2 : boolean

jshell> "Hello, world" == new String("Hello, world")
$3 ==> false
|&nbsp; created scratch variable $3 : boolean
```

## 总结

好，到这里，今天的课程就要结束了，我来做个小结。从前面的讨论中，我们了解了JShell的基本概念、它的表达形式以及编译的过程。

JShell提供了一种在 JShell 状态下交互式评估 Java 编程语言的声明、语句和表达式的方法。JShell 的状态包括不断发展的代码和执行状态。为了便于快速调查和编码，语句和表达式不需要出现在方法中，变量和方法也不需要出现在类中。

JShell的设计并不是为了取代IDE。JShell在处理简单的小逻辑，验证简单的小问题时，比IDE更有效率。如果我们能够在有限的几行代码中，把要验证的问题表达清楚，JShell就能够快速地给出计算的结果。这一点，能够极大地提高我们的工作效率和学习热情。

但是，对于复杂逻辑的验证，使用JShell也许不是一个最优选择。这时候，也许使用IDE或者可编译的代码更合适。

我还拎出了几个技术要点，这些都可能在你的面试中出现。通过这一次学习，你应该能够：

- 了解JShell的基本概念，知道JShell有交互式工具，也有API； - 面试问题：你使用过JShell吗？

    <!-- -->

- 知道JShell能够接收Java编程语言的声明、语句和表达式，以及命令行； - 面试问题：JShell的代码和普通的可编译代码，有什么不一样？

    <!-- -->


<!-- -->

这一次的讨论，主要是想让你认识到JShell能给我们带来的便利，知道简单的使用方法。这样，当后面我们想要讨论更多的话题时，你就可以使用JShell快速验证你的小问题、小想法。 要想掌握JShell更复杂的用法，请参考相关的文档或者材料。

## 思考题

在前面的讨论里，我们使用了一个例子，来说明Java处理字符串常量的方式。

```shell
jshell> "Hello, world" == "Hello, world"
$2 ==> true
|&nbsp; created scratch variable $2 : boolean
```

对于精通Java语言的同学，这个例子也许是直观的。但对部分同学来说，这个例子也许过于隐晦。过于隐晦的代码不是好的代码。同样地，过于隐晦的JShell片段也不是好的片段。

你有没有办法，让这个例子更容易理解？使用多个JShell片段，是不是更好理解？这就是我们今天的思考题。

欢迎你在留言区留言、讨论，分享你的阅读体验以及你对这个思考题的处理办法。

注：本文使用的完整的代码可以从[GitHub](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/jshell>)下载，你可以通过修改[GitHub](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/instance>)上[review template](<https://github.com/XueleiFan/java-up/blob/main/src/main/java/co/ivi/jus/jshell/review/xuelei/string-literal.jsh>)代码，完成这次的思考题。如果你想要分享你的修改或者想听听评审的意见，请提交一个 GitHub的拉取请求（Pull Request），并把拉取请求的地址贴到留言里。这一小节的拉取请求代码，请放在[实例匹配专用的代码评审目录](<https://github.com/XueleiFan/java-up/blob/main/src/main/java/co/ivi/jus/jshell/review>)下，建一个以你的名字命名的子目录，代码放到你专有的子目录里。比如，我的代码，就放在jshell/review/xuelei的目录下面。

## 精选留言(19)

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34.jpeg)

  kimoti

  老板知道Java有这些新特性他会要求你的效率提高20%。所以不要指望每周能多休一天,码农永远是被压榨的对象。

  作者回复: 也许可以换个角度看， 如果我们不能提高，还能不能保得住工作； 如果老板不去提高，公司会不会黄掉。 技术进步的大潮下， 有些人的影响是负面的，有些人的影响是正面的， 不过我们应该都有选择权。

  2021-11-17

  **2

  **6

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779711-1.jpeg)

  aoe

  错过11直接搭上17顺风车

  作者回复: ;-) 只要在路上，永远都不晚。

  2021-11-16

  **

  **6

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  王国建

  github代码库地址在哪里啊

  作者回复: 没一讲的最后，都有一个注。 这个注里，会说明GitHub的代码，和提交拉去请求的目录。

  2021-11-17

  **

  **5

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779711-2.jpeg)

  light

  老师，有什么介绍java语言设计的历史的书或者文章吗？从这些特性的变迁可以看到程序设计方式、理论的变化。

  作者回复: 有过有这样的书，真是值得看一看，应该可以看透很多东西。 可惜，我不知道市面上有没有这样的书或者文章。简单的历史很容易找到，要深入介绍的变迁背后的逻辑的，我还没有看到。看到过的小伙伴，给介绍下？

  2021-11-15

  **

  **5

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-3.jpeg)![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,w_14.png)

  jjn0703![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  现在公司开发还是以Java8为准，如果想不影响日常开发环境的情况下，安装JDK17尝试新特性，可以做到么

  作者回复: 可以做到啊，你可以安装多个版本，配置/运行时，选择需要的版本就可以了。 如果你还需要进一步的信息，可以说一下你的开发环境，看看小伙伴们是不是有类似的经验。

  2021-11-22

  **

  **1

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-4.jpeg)

  面白i小黄毛

  老师，可不可以讲讲各类市面上的jdk的异同，比如oracle jdk，openjdk，还有最近m1芯片比较多用的zulu jdk等等。公司一直在用jdk8，之后的oracle jdk不是要收费吗？我们使用更高版本jdk的时候应该如何选择各类的jdk呢？希望老师可以讲解讲解。谢谢老师

  作者回复: 我不是律师，我的评论不构成建议。 我想，基本上都是一样的。 Oracle JDK和OpenJDK的差异，几乎可以忽略不计。选择的主要依据，就是是否需要技术支持，以及对安全问题响应的及时程度。 很多公司需要付费的技术支持的，也有很多公司不需要技术支持。 需要技术支持的，就选择Oracle JDK；不需要的，就选择OpenJDK。对安全敏感的，也选择Oracle JDK。Oracle JDK 17是免费的，直到下一个LTS发布。 我在InfoQ写作平台有一篇文章（https:&#47;&#47;xie.infoq.cn&#47;article&#47;a72ca12f97838e16ba2c91937），你可以看一看。 稍微有点老了，没有反应JDK17的变化，但是基本的精神还在。 另外，我不是律师，我的评论不构成建议。关于许可证的问题，只有公司的律师有资格回答这个问题。

  2021-11-20

  **

  **1

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-5.jpeg)

  meanless

  "所以，当时主流的客户端 - 服务器的设计，是使用阻塞式的套接字接口编程。现在，如果淘宝、京东还使用阻塞式的套接字接口，那是没有一点希望支持双十一的巨大流量的。" 那么现在大厂们用的都是什么技术,用到了Java的什么新特性呢?

  作者回复: NIO和反应式是其中的两个。

  2021-11-20

  **

  **1

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-6.jpeg)

  镞砺括羽

  会不会讲lamdba和stream

  作者回复: lamdba和stream是JDK 8的新特性，我们只讲JDK 8以后发布的新特性。不过，我们会讲Flow。 小伙伴们，有没有看到过比较好的lamdba和stream文章或者书籍，推荐下？

  2021-11-16

  **5

  **1

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-7.jpeg)![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,w_14.png)

  码小呆

  目前公司的新项目用了jdk17,不过,我感觉和其他版本并没有什么区别,出现这种的情况是因为,我不懂17有哪些新的特性,希望学习完这门课程后,对jdk17有一个全新的认识~~

  2022-06-29

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-7.jpeg)![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,w_14.png)

  码小呆

  github的仓库地址在哪里啊,老师

  作者回复: 看思考题的链接。

  2022-06-29

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779712-8.jpeg)![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,w_14.png)

  二饼![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  好实用的课程！

  2022-06-23

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  hhhhhh

  老师 git地址在哪里

  作者回复: 看思考题里提供的链接。

  2022-06-21

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779713-9.jpeg)

  song

  高版本jdk 收费 影响了推广

  作者回复: 可以使用OpenJDK，区别细微。

  2022-06-01

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779713-10.jpeg)

  曾子良

  公司使用的是jdk8，没有实战场景啊

  作者回复: 会有的。曾经的JDK 1.4.2，比JDK 8还长寿。

  2022-04-30

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/132.jpeg)![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,w_14.png)

  jiangb

  关键词20%，期待20%

  2022-01-26

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779713-11.jpeg)![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,w_14.png)

  橙子

  老师有没有计划搞一期ZGC的专题？

  作者回复: 很遗憾，目前还没有计划。 留言里点名了不少特性，新的旧的都有。 我要看看专栏写完后，我还有没有时间加一些续篇进来。目前看，我的时间确实很难再挤得出来了，除非等到下一次休假的时候。

  2021-11-30

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779713-12.jpeg)

  LiuHu

  老师关于升级收益能够相对具体说明一下评估基础吗？

  作者回复: 文章中提到的是一个粗暴的评估，基础是经验，而不是具体的基准测试。每一个Java的新技术，都会评估它能带来的性能和效率。从JDK 9到JDK 17，这样的小升级有很多。你如果看每一个小升级的的性能提升和效率，并且能够切实地使用起来，这些小的升级累计起来，你就能看到可观的收益。曾经有Flink的开发者，做了JDK 8切换到JDK 11之后的基准测试，我忘记数字了，只记得大致的结果是： 哇！ 我们后面讲到具体的新技术的时候，会聊到这些受益。这里的数字，并不能解决为什么、怎么办这样的问题。这些问题的讨论，会在后面的新特性讨论中出现。到时候，我们就会有更深入的了解了。

  2021-11-28

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779713-13.jpeg)

  Calvin

  课程中说是“JDK 8 以来重量级的 Java 新特性清单”，看了整个课程目录，好像没有 JDK 8 的 lamdba 表达式、函数式接口之类的，我觉得这些老师反而应该重点讲下。^_^

  作者回复: Lambda表达式和函数式接口，是JDK 8以及之前的内容。这个专栏讲解JDK 8以后的新特性。已经看到多次这样的留言了，好像JDK 8的新技术还没有普及。我们以后有机会再说吧。

  2021-11-27

  **

  **

- ![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662398779713-14.jpeg)

  giteebravo![img](01%20_%20JShell%EF%BC%9A%E6%80%8E%E4%B9%88%E5%BF%AB%E9%80%9F%E9%AA%8C%E8%AF%81%E7%AE%80%E5%8D%95%E7%9A%84%E5%B0%8F%E9%97%AE%E9%A2%98%EF%BC%9F.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  平时工作中主要使用 Java 8 还在和 lamda stream 较着劲 以为专栏会包含 Java 8 的特性 不过呢了解一下新版本的特性也好

  作者回复: 嗯， 很多人都还在学习Lamda和stream。 😄不耽误也看看更新的。

  2021-11-25

  **

  **
