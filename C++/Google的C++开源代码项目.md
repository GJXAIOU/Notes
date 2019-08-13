# Google的C++开源代码项目

v8  -  V8 JavaScript Engine
V8 是 Google 的开源 JavaScript 引擎。
V8 采用 C++ 编写，可在谷歌浏览器（来自 Google 的开源浏览器）中使用。
V8 根据 ECMA-262 第三版中的说明使用 ECMAScript，并在使用 IA-32 或 ARM 处理器的 Windows XP 和 Vista、Mac OS X 10.5 (Leopard) 以及 Linux 系统中运行。
V8 可以独立运行，也可以嵌入任何 C++ 应用程序中。

nativeclient  -  Native code for web apps
Native Client是一个在Web应用程序中运行本地代码（目前只支持x86架构）的开源的研究性技术，提供更好的“富客户端”用户体验。它允许网络开发者编写更强大的Web程序，这些程序直接通过系统运行而不用通过浏览器来进行,据Google称，它到最后将允许网络开发者开发和桌面软件一样的的web程序，这些程序将带来更快的速度。Native Client类似于微软的ActiveX技术，它还能在Linux和Mac OS X下运行。目前它尚未支持IE，仅支持Google Chrome, Firefox, Safari 和Opera。

tesseract-ocr - An OCR Engine that was developed at HP Labs between 1985 and 1995... and now at Google.
OCR(Optical Character Recognition):光学字符识别,是指对图片文件中的文字进行分析识别，获取的过程。
Tesseract：开源的OCR识别引擎，初期Tesseract引擎由HP实验室研发，后来贡献给了开源软件业，后经由Google进行改进，消除bug，优化，重新发布。当前版本为3.01.

google-glog  -  Logging library for C++
Google glog是一个基于程序级记录日志信息的c++库，编程使用方式与c++的stream操作类似，例：
LOG(INFO) << "Found " << num_cookies << " cookies";

double-conversion  -  Binary-decimal and decimal-binary routines for IEEE doubles.
从V8引擎中抽出的有关数值计算相关的代码，包括大数计算，数值到字符串转换等

googletest  -  Google C++ Testing Framework
gtest测试框架[1]是在不同平台上（Linux，Mac OS X，Windows，Cygwin，Windows CE和Symbian）为编写C++测试而生成的。它是基于xUnit架构的测试框架，支持自动发现测试，丰富的断言集，用户定义的断言，death测试，致命与非致命的失败，类型参数化测试，各类运行测试的选项和XML的测试报告。

googlemock  -  Google C++ Mocking Framework
googlemock　　mock技术，在c++单元测试可以随意修改函数行为的技术。 　　
googlemock是google基于gtest开发的mock框架，适用于c++单元测试。

libphonenumber  -  Google's phone number handling library, powering Android and more
一个专门用于处理电话号码的库

google-diff-match-patch  -  Diff, Match and Patch libraries for Plain Text
google-diff-match-patch这个类库提供了强大的算法用于纯文本内容的差异比较，匹配，打补丁，实现同步纯文本所需要执行一些操作。支持多种语言包括：Java、JavaScript、C++、C#、Objective C、Lua和Python。

libkml  -  a KML library written in C++ with bindings to other languages
libKML是解析，生成和操作KML的库。使用OGC KML2.2标准。
KML，是 Keyhole 标记语言（Keyhole Markup Language）的缩写，是一种采用 XML 语法与格式的语言，用于描述和保存地理信息（如点、线、图像、多边形和模型等），可以被 Google Earth 和 Google Maps 识别并显示。您可以使用 KML 来与其他 Google Earth 或 Google Maps 用户分享地标与信息。当然，您也可以从 Google Earth 社区 等相关网站获得有趣的 KML 文件。Google Earth 和 Google Maps 处理 KML 文件的方式与网页浏览器处理 HTML 和 XML 文件的方式类似。像 HTML 一样，KML 使用包含名称、属性的标签（tag）来确定显示方式。因此，您可将 Google Earth 和 Google Maps 视为 KML 文件浏览器。单击此处可获得更多信息。

gdata-cpp-util  -  Google Data APIs C++ utility library
一个Google Data APIs 的工具库，可以GET/POST/PUT/DELETE

lutok  -  Lightweight C++ API for Lua
是一个 Lua 的 C++ wrapper
Lua 是一个小巧的脚本语言。是巴西里约热内卢天主教大学（Pontifical Catholic University of Rio de Janeiro）里的一个研究小组，由Roberto Ierusalimschy、Waldemar Celes 和 Luiz Henrique de Figueiredo所组成并于1993年开发。 其设计目的是为了嵌入应用程序中，从而为应用程序提供灵活的扩展和定制功能。Lua由标准C编写而成，几乎在所有操作系统和平台上都可以编译，运行。Lua并没有提供强大的库，这是由它的定位决定的。所以Lua不适合作为开发独立应用程序的语言。Lua 有一个同时进行的JIT项目，提供在特定平台上的即时编译功能。

dcs-bwt-compressor  -  Data compressor program and library
dcsbwt是一个基于Burrower-Wheeler变换的数据压缩程序库

treetree  -  generic n-ary trees for C++
TreeTree （http://code.google.com/p/treetree/）是一个只包含头文件的 C++ Library。它实现了一个通用的 树形结构容器类（遵守 STL约定），并且实现了 operator >> 和 operator <<。
它的底层包含一个双向链表。在前项指针和后项指针以外，每个树节点还包含第三个指针--指向所有子节点的列表。这个实现高效，并且 API 非常清晰。TreeTree 可以表达任何使用Lisp的S表达式能表达的东西（例如推理树(inference trees, programs)等)。
可以使用前序和后序遍历，只遍历某个节点的子节点，或者只是在叶子节点遍历。示例的选项还包含遍历子树（如f(g(x,y),z) 前序遍历，是f(g(x,y),z), g(x,y),x,y和z.

ctemplate  -  Powerful but simple template language for C++
ctemplate (Google-ctemplate)的设计哲学是轻量级，快速，且逻辑和界面分离，因此和ClearSilver和Teng是有一些差异的。比如Ctemplate就没有模板函数，没有条件判断和循环语句（当然，它可以通过变通的方式来实现）。 　　ctemplate大体上分为两个部分，一部分是模板，另一部分是数据字典。模板定义了界面展现的形式（V），数据字典就是填充模板的数据（M），你自己写业务逻辑去控制界面展现（C），典型的MVC模型。

sparsehash  -  An extremely memory-efficient hash_map implementation
Google Sparse Hash 是 Google 一个很节省内存的 hash map 实现

gflags  -  Commandline flags module for C++
Google GFlags 是一个命令行标记的处理库，它可以替代“getopt()”，其内置对C++的支持比如string。

protobuf  -  Protocol Buffers - Google's data interchange format
Google Protocol Buffer 是一个平台无关、语言无关的结构化数据的序列化与反序列化工具。
protocol buffer，可以用来在跨进程、跨机器，不同操作系统，不同编程语言之间进行数据交换。类似于微软的COM IDL或者XML，但是解析速度更快，需要传输字节数更少。(c+
+, java, python)

gperftools  -  Fast, multi-threaded malloc() and nifty performance analysis tools 
TCMalloc，heap检测，是一个google用于性能检测的工具。(c++)

google-breakpad  -  Crash reporting

breakpad，一个项目的开始需要做一些什么样的基础设施，crash dump和运行logging毫无疑问都是应该有的，这个项目就是负责在crash的时候收集信息，发出crash dump报告的。

# 经典的C++库

STLport-------SGI STL库的跨平台可移植版本，在以前有些编译器离符合标准比较远的情况下那时还是有用的，当然目前vc71已经比较接近标准了，故目前不怎么用它了。

Boost---------准标准库，功能强大涉及能想的到的大部分非特别领域的[算法](http://lib.csdn.net/base/datastructure "算法与数据结构知识库")，有一个大的C++社区支持

WxWindows-----功能强大的跨平台GUI库，它的功能和结构都类似MFC，故原则上可以通过WxWindows把现有MFC程序移植到非Win平台下

Blitz---------高效率的数值计算函数库 ,你可以订制补充你需要的算法

Log4cpp-------日志处理，功能类似[Java](http://lib.csdn.net/base/java "Java 知识库")中的log4j

ACE-----------自适应通讯环境，重量级的通讯环境库。

Crypto++ -----加/解密算法库, 非常专业的C++ 密码学函式库

CppUnit --- 一个c++的单元[测试](http://lib.csdn.net/base/softwaretest "软件测试知识库")框架  类似 java  的JUnit

Loki ------- 一个实验性质的库，尝试把类似设计模式这样思想层面的东西通过库来提供,他是C++的一个模板库,系C++"贵族"，它把C++模板的功能发挥到了极致

学术性的C++库:

FC++ --------The Functional [C++](http://lib.csdn.net/base/cplusplus "C++知识库")Library  ,用库来扩充语言的一个代表作 ,模板库

CGAL ------- Computational GeometryAlgorithms Library计算几何方面的大部分重要的解决方案和方法以C++库的形式提供给工业和学术界的用户。

其它目前我感觉还不是很爽的C++库： 

Doxygen ----注释文档生成工具 ,可恨的是我找不到 windows版本

QT ----------大名顶顶的一个多平台的C++图形用户界面应用程序框架（GUI库）可气的是他的Windows版是商业发布的要付费

xml4c--------IBM开发的XML Parser，系超重量级的，适用大型应用中，其DLL有 12M，恐怖吧，轻量级的有TinyXml

Xerces c++ --Apache的XML项目， 但 只支持少数的字符编码，如ASCII，UTF-8，UTF-16等，不能处理包含中文字符的XML文档

XMLBooster -----  也是一种  XML的 解析工具

Fox  -------又一种开放源代码（C++）的GUI库，功能不是很强

C++开发环境(Win平台下除了 VisualC++ 和 Borland C++以外的)：

Cygwin --------Windows下的一个Unix仿真环境

MinGW  --------GCC的一个Windows移植版本

Dev C++ -------- 一个C/C++ 的集成开发环境，在Windows上的C++编译器一直和标准有着一段距离的时候，GCC就是一个让Windows下开发者流口水的编译器。

Eclipse-CDT  ----IMB 开发的一个集成开发环境，一般用来作为Java 开发环境，但由于Eclipse 是通过插件体系来扩展功能，这里我们 安装 CDT插件后，就可以用来作为C++集成开发环境工具。

# 50个知名的开源网站

1、http://snippets.dzone.com/tag/c/--数以千计的有用的[C语言](http://lib.csdn.net/base/c "C语言知识库")源代码片段

2、http://www.hotscripts.com/category/c-cpp/scripts-programs/Hotscripts --提供数以百计的C和C++脚本和程序。所有程序都分为不同的类别。

3、http://www.planetsourcecode.com/vb/default.asp?lngWId=3--超过万行C和C++免费的源代码

4、http://freshmeat[.NET](http://lib.csdn.net/base/dotnet ".NET知识库")/browse/164/--超过9000个C编写的项目。

5、http://www.daniweb.com/code/c.html--DANIWEB提供的实用代码段。

6、http://www.programmersheaven.com/tags/C/--programmersheaven.com上的C编程资源。

7、http://www.ddj.com/code/ddj.html--Dr. Dobb’s Journal的源代码。

8、http://www.cprogramming.com/cgi-bin/source/source.cgi--C和C + +编程资源。

9、http://www.codecogs.com/--CodeCogs是一项协作的开放源码库，C/C++的数值方面的组件。

10、http://www.google.com/codesearch?q=programming++lang:c&cs_r=lang:c--谷歌代码的C源代码。

11、http://www.codepedia.com/1/C--CodePedia是一个开放的关于系统编程和其他与电脑有关的议题。

12、http://www.cis.temple.edu/~ingargio/cis71/code/--为学生提供的一个简单的[c语言](http://lib.csdn.net/base/c "C语言知识库")程序的列表。

13、http://www.codeproject.com/?cat=2--codeproject提供的C/C++资源代码项目。

14、http://www.thefreecountry.com/sourcecode/cpp.shtml--以下是一些C和C++库的DLL，VCLs，源代码，元件，模块，应用程序框架，类库，源代码片段等，你可以在您的项目中使用而不需要支付费用和版税。

15、http://people.sc.fsu.edu/~burkardt/cpp_src/cpp_src.html--这是一个全面的关于C++的345个源代码清单。

16、http://www.cplusplus.com/src/--C++写的通用控制台程序和Windows程序代码清单。

17、http://users.cs.fiu.edu/~weiss/dsaa_c++/code/--C++语言[数据结构](http://lib.csdn.net/base/datastructure "算法与数据结构知识库")与算法分析（第二版）的源代码。

18、http://c.snippets.org/--C源代码片段。

19、http://www.bbdsoft.com/downloads.html--C++源代码。

20、http://www.moshier[.net](http://lib.csdn.net/base/dotnet ".NET知识库")/天文学和数值软件源代码

21、http://cplus.about.com/od/cgames/C_Games_with_Source_Code.htm--游戏有关的C++源代码。

22、http://cliodhna.cop.uop.edu/~hetrick/c-sources.html--免费的C/C++数值计算源代码。

23、http://www.mathtools[.Net](http://lib.csdn.net/base/dotnet ".NET知识库")/C_C__/Utilities/index.html--C/C++工具。

24、http://www.programmerworld.net/resources/c_library.htm--免费C++源代码和其它有用的工具。

25、http://www.cmcrossroads.com/bradapp/links/cplusplus-links.html--布拉德阿普尔顿的C++链接-资源，项目，图书馆，教学和编码。

26、http://www.robertnz.net/cpp_site.html--这是一个收集了数C/C++网站链接列表的网页。

27、http://www.josuttis.com/libbook/examples.html--在这里，你可以看到并下载所有的本书的C++标准库例子。

28、ftp://66.77.27.238/sourcecode/cuj/--C/C++用户杂志

29、ftp://66.77.27.238/sourcecode/wd/--Windows开发者网络

30、http://www.einet.net/directory/65892/Developers.htm--C程序

31、http://www.daniweb.com/code/cplusplus.html--实用代码段。

32、http://snippets.dzone.com/tag/c--C++源代码

33、http://www.programmersheaven.com/tags/C--C++编程资源，programmersheaven.com

34、http://www.google.com/codesearch?hl=en&lr=&q=programming--谷歌代码搜索-C++编程语言

35、http://www.codepedia.com/1/Cpp--CodePedia是一个开放的关于系统编程和其他与电脑有关的议题的网站。

36、http://www.codebeach.com/index.asp?TabID=1&CategoryID=3--C++源代码，Codebeach提供

37、http://freshmeat.net/browse/165/--5000项目写的C++编程语言

38、http://cplus.about.com/od/codelibrary/Code_Library_for_C_C_and_C.htm--代码库C、C + +和C＃。

39、http://www.c.happycodings.com/--Visual Basic、[PHP](http://lib.csdn.net/base/php "PHP知识库")、ASP技术、C、C++大全。

40、http://www.blueparrots.com/--Borland C游戏，图像和声音源代码范例。

41、http://www.java2s.com/Code/Cpp/CatalogCpp.htm--C++源代码。

42、http://www.yeohhs.com/modules/mydownloads/--C与C++电子书和源代码示例。

43、http://www.brpreiss.com/books/opus4/programs/index.htmlC++的数学方程和公式源代码。

44、http://users.cs.fiu.edu/C++。

45、http://www.josuttis.com/libbook/examples.html--C++标准库-教程和参考资料。

46、http://emr.cs.uiuc.edu/~reingold/calendars.shtmlEdward M. Reingold's Calendar Book, Papers, and Code。

47、http://cpp.snippets.org/--c++源代码档案。

48、http://ubiety.uwaterloo.ca/~tveldhui/papers/techniques/--用C和C++的解决科学问题。

49、http://c.ittoolbox.com/topics/core-c/--C/C++的IT工具框。

50、http://www.le.ac.uk/cc/tutorials/c/ccccdbas.html--本文件中包含有大量的C示例程序

下面结合自己多年的开发经验，想到哪里写到哪里，希望对新人有一定的帮助。

一、网络

网络库必须掌握 ACE 和 libevent， 一个是重量级的网络库， 一个是轻量级的网络库。仔细想想，现在那个程序不用网络啊。不懂网络，你将寸步难行啊。熟悉这两个开源库的　　　　前提是你必须懂socket的原理，给大家推荐的好书就是[《UNIX网络编程》](https://www.baidu.com/s?wd=%E3%80%8AUNIX%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E3%80%8B&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，看懂这本书就可以了，其他的什么[《windows网络编程》](https://www.baidu.com/s?wd=%E3%80%8Awindows%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B%E3%80%8B&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)这些都不用看，因为网络编程，你学会了伯克利的套接字，你就可以在任何平台上进行网络编程了，不需要学什么windows下的网络编程，因为windows下的网络也是从伯克利套接字搞过来的，如果你学习《windows网络编程》，那么你那天在[Linux](http://lib.csdn.net/base/linux "Linux知识库")，unix下进行网络编程，你还得在学习一下。没有那个必要。

二、[数据库](http://lib.csdn.net/base/mysql "MySQL知识库")

数据库嘛，开源的[MySQL](http://lib.csdn.net/base/mysql "MySQL知识库")和开源的PostgreSQL只要懂其中一个就可以了，商业数据库在掌握一个[Oracle](http://lib.csdn.net/base/oracle "Oracle知识库")就可以了，文件数据库掌握 sqlite。不过请大家注意，不要被上面数据库名字给迷惑了，数据库的本质是SQL语句，一定要懂数据库的基本原理，熟练应用SQL语言，懂数据库的优化，存储过程等。数据库的原理搞懂了，拿什么数据库过来都轻松掌握，就不会在乎是[mysql](http://lib.csdn.net/base/mysql "MySQL知识库")还是[oracle](http://lib.csdn.net/base/oracle "Oracle知识库")了。

三、日志操作

日志操作推荐大家熟悉 log4cpp这个日志库， 支持多线程， 日志重定向到网络等都有， 反正你能想到的日志的功能都有。日志嘛，一个是方便查找问题，方便记录程序运行的一些情况。这是必须的。

四、管理后台

[众所周知](https://www.baidu.com/s?wd=%E4%BC%97%E6%89%80%E5%91%A8%E7%9F%A5&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，写程序是给其他人用的，不是自己用，所以在程序的易用性上，多给对方提供一些可以直接查看的管理界面，显得分成重要。为程序提供一个web的管理界面，方便用户登录上去查看程序的各种信息，很有必要。

五、读取配置文件

所有的程序都必须有配置文件，方便配置一些项目，为程序提供灵活性。 所以写程序，必须有读取配置文件的封装类。

六、内存池

所有的进程都需要分配内存，对C/C++来说，分配和管理内存是已经很有挑战性的工作。给大家推荐 nedmalloc 这个开源的内存池库。nedmalloc是一个跨平台的高性能多线程内存分配库，很多库都使用它。

七、缓存库

众所周知，缓存库用得最多的就是memcache了。在做数据库开发的时候特别有用。

八、脚本

脚本是一个很有意思的东西，很多功能，其实我们只要写个脚本就可以完成，代码量少，开发速度快。必须掌握的脚本，比较通用的要算 perl 了，很古老的语言，但是功能太强大了。我可以保证的说，.net，java能干的工作，肯定可以让perl来干。C能干的， perl不一定能干。perl作为[linux](http://lib.csdn.net/base/linux "Linux知识库")，unix的系统集成的脚本语言，必须学会。

lua 语言，在游戏行业用得比较多。

[Python](http://lib.csdn.net/base/python "Python知识库") 脚本，功能很强大，推荐学。

上面这些是所有程序都会用到的比较通用的功能。

在不同的应用领域，需要掌握不同开源库，比如搞游戏开发的，可能需要掌握开源的UI库CEGUI、duilib, 开源的3D引擎OGRE等