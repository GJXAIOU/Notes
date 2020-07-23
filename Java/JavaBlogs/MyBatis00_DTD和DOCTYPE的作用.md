
# DTD 简介

*   [DTD 教程](https://www.w3school.com.cn/dtd/index.asp "DTD 教程")
*   [DTD 构建模块](https://www.w3school.com.cn/dtd/dtd_building.asp "DTD - XML 构建模块")

**文档类型定义（DTD）可定义合法的XML文档构建模块。它使用一系列合法的元素来定义文档的结构。**

**DTD 可被成行地声明于 XML 文档中，也可作为一个外部引用。**

## 内部的 DOCTYPE 声明

假如 DTD 被包含在您的 XML 源文件中，它应当通过下面的语法包装在一个 DOCTYPE 声明中：

<!DOCTYPE 根元素 [元素声明]>

带有 DTD 的 XML 文档实例（请在 IE5 以及更高的版本打开，并选择查看源代码）：

<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>

[在您的浏览器中打开此 XML 文件，并选择“查看源代码”命令](https://www.w3school.com.cn/dtd/note_in_dtd.xml)。

### 以上 DTD 解释如下：

_!DOCTYPE note_ (第二行)定义此文档是 _note_ 类型的文档。

_!ELEMENT note_ (第三行)定义 _note_ 元素有四个元素："to、from、heading,、body"

_!ELEMENT to_ (第四行)定义 _to_ 元素为 "#PCDATA" 类型

_!ELEMENT from_ (第五行)定义 _from_ 元素为 "#PCDATA" 类型

_!ELEMENT heading_ (第六行)定义 _heading_ 元素为 "#PCDATA" 类型

_!ELEMENT body_ (第七行)定义 _body_ 元素为 "#PCDATA" 类型

## 外部文档声明

假如 DTD 位于 XML 源文件的外部，那么它应通过下面的语法被封装在一个 DOCTYPE 定义中：

<!DOCTYPE 根元素 SYSTEM "文件名">

这个 XML 文档和上面的 XML 文档相同，但是拥有一个外部的 DTD: （[在 IE5 中打开](https://www.w3school.com.cn/dtd/note_ex_dtd.xml)，并选择“查看源代码”命令。）

<?xml version="1.0"?>
<!DOCTYPE note SYSTEM "note.dtd">
<note>
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note> 

这是包含 DTD 的 "note.dtd" 文件：

<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>

## 为什么使用 DTD？

通过 DTD，您的每一个 XML 文件均可携带一个有关其自身格式的描述。

通过 DTD，独立的团体可一致地使用某个标准的 DTD 来交换数据。

而您的应用程序也可使用某个标准的 DTD 来验证从外部接收到的数据。

您还可以使用 DTD 来验证您自身的数据。







# DTD和DOCTYPE的作用

[原文链接](https://blog.csdn.net/caomiao2006/article/details/9223421)
一直以来写网页，不论用Adobe（以前是Macromedia）的DW，还是Editplus自动生成的初始网页，头部都会加上类似的一句话：

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0Transitional//EN""http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"> 而一直以来本人对他的态度也是忽略的，以为这个东西没什么作用。今天知道他原来在HTML中是很有作用的。

**DTD声明是什么意思呢？**

DTD意为Document Type Definition（文档类型定义），先撇开DTD文件的具体内容不谈。我们看到HTML文档中的DTD声明始终以!DOCTYPE开头，空一格后跟着文档根元素的名称（网页也就是HTML）。如果是内部DTD，则再空一格出现[]，在中括号中是文档类型定义的内容。而对于外部DTD，则又分为私有DTD与公共DTD，私有DTD使用SYSTEM表示，接着是外部DTD的URL。而公共DTD则使用PUBLIC，接着是DTD公共名称，接着是DTD的URL。下面是一些示例。

公共DTD，DTD名称格式为“注册//组织//类型 标签//语言”,“注册”指示组织是否由国际标准化组织(ISO)注册，+表示是，-表示不是。“组织”即组织名称，如：W3C；“类型”一般是DTD，“标签”是指定公开文本描述，即对所引用的公开文本的唯一描述性名称，后面可附带版本号。最后“语言”是DTD语言的ISO 639语言标识符，如：EN表示英文，ZH表示中文。

本文开头的DTD也就是XHTML 1.0 Transitional的DTD。以!DOCTYPE开始，html是文档根元素名称，PUBLIC表示是公共DTD，后面是DTD名称，以-开头表示是非ISO组织，组织名称是W3C，文档类型是XHTML，版本号1.0过渡类型，EN表示DTD语言是英语，最后是DTD的URL。

注意：虽然DTD的文件URL可以使用相对URL也可以使用绝对URL，但推荐标准是使用绝对URL。另一方面，对于公共DTD，如果解释器能够识别其名称，则不去查看URL上的DTD文件。

**下面来说说DTD的作用。**

<P align="center">这是一个居中段落</P>

在XHTML中，标记是区分大小写的，上面的代码毫无意义。可在HTML中它是一个居中段落。浏览器是怎样处理这种情况呢？难道浏览器认为你写的是HTML，然后把它作为一个一个居中段落显示？如是你写的是XHTML呢，它将是一段不可显示的代码！浏览器是怎样知道你用的是什么标记语言然后正确对待这段代码呢？

这就是DTD的工作了。一个DTD应该放在每一个文档的第一行。这样正确地放置，你的DTD才能告诉浏览器的用的是什么标记语言。在通常情况下，如果你编写的是正确代码，并拥有一个合适的DTD，浏览器将会根据W3C的标准显示你的代码。

如果说你没有使用DTD，你将很难预测浏览器是怎样显示你的代码，仅仅在同一浏览器就有不同的显示效果。尽管你的网页做得非常飘亮，要是没有使用DTD，你的努力也是白费的。因此，一个DTD是必不可少的。

**XHTML**较为规范，他的DTD分为几种：Strick、Transitional和Frameset

XHTML1.0 Strict DTD（严格的文档类定义）：要求严格的DTD，你不能使用表现标识和属性，和CSS一同使用。完整代码如下：

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">

XHTML1.0 Transitional DTD（过渡的文档类定义）：要求非常宽松的DTD，它允许你继续使用HTML4.01的标识(但是要符合xhtml的写法)。完整代码如下：

<!DOCTYPE html  PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

XHTML1.0 Frameset DTD（框架集文档类定义）:专门针对框架页面设计使用的DTD，如果你的页面中包含有框架，需要采用这种DTD。完整代码如下：    

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Frameset//EN"  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-frameset.dtd">

**HTML**的语法就非常宽松了，他的DTD也分为一样的三种。

HTML 4.01 Strict DTD （严格的文档类定义）不能包含已过时的元素（或属性）和框架元素。对于使用了这类DTD的文档，使用如下文档声明：

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

HTML 4.01 Transitional DTD（过渡的文档类定义）能包含已过时的元素和属性但不能包含框架元素。对于使用了这类DTD的文档，使用如下文档声明：    

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"  "http://www.w3.org/TR/html4/loose.dtd">

HTML 4.01 Frameset DTD（框架集文档类定义）。能包含已过时的元素和框架元素。对于使用了这类DTD的文档，使用如下文档声明：

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN"  "http://www.w3.org/TR/html4/frameset.dtd">

更早的HTML版本：HTML 3.2   

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">  

更早的HTML版本：HTML 2.0    

<!DOCTYPE html PUBLIC "-//IETF//DTD HTML 2.0//EN">

**我们选择什么样的DOCTYPE**

理想情况当然是严格的DTD，但对于我们大多数刚接触web标准的设计师来说，过渡的DTD是目前理想选择。因为这种DTD还允许我们使用表现层的标识、元素和属性，也比较容易通过W3C的代码校验。

上面说的“表现层的标识、属性”是指那些纯粹用来控制表现的tag，例如用于排版的表格（<table>标签）、换行（<br>标签）和背景颜色标识等。在XHTML中标识是用来表示结构的，而不是用来实现表现形式，我们过渡的目的是最终实现数据和表现相分离。

[http://blog.sina.com.cn/s/blog_44dd2a630100pc6y.html](http://blog.sina.com.cn/s/blog_44dd2a630100pc6y.html)