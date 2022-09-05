# 08 \| 应用1：正则如何处理 Unicode 编码的文本？

作者: 涂伟忠

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/a2/70/a28dyy0f0e02c5c393080feefdf13170.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/61/f1/613871e207af4bcf76d87b23a65be1f1.mp3" type="audio/mpeg"></audio>

你好，我是伟忠。这一节我们来学习，如何使用正则来处理Unicode编码的文本。如果你需要使用正则处理中文，可以好好了解一下这些内容。

不过，在讲解正则之前，我会先给你讲解一些基础知识。只有搞懂了基础知识，我们才能更好地理解今天的内容。一起来看看吧！

## Unicode基础知识

**Unicode**（中文：万国码、国际码、统一码、单一码）是计算机科学领域里的一项业界标准。它对世界上大部分的文字进行了整理、编码。Unicode使计算机呈现和处理文字变得简单。

Unicode至今仍在不断增修，每个新版本都加入更多新的字符。目前Unicode最新的版本为 2020 年3月10日公布的13.0.0，已经收录超过 14 万个字符。

现在的Unicode字符分为17组编排，每组为一个平面（Plane），而每个平面拥有 65536（即2的16次方）个码值（Code Point）。然而，目前Unicode只用了少数平面，我们用到的绝大多数字符都属于第0号平面，即**BMP平面**。除了BMP 平面之外，其它的平面都被称为**补充平面**。

关于各个平面的介绍我在下面给你列了一个表，你可以看一下。

![](<https://static001.geekbang.org/resource/image/8c/61/8c1c6b9b87f10eec04dbc2224f755d61.png?wh=1660*684>)

Unicode标准也在不断发展和完善。目前，使用4个字节的编码表示一个字符，就可以表示出全世界所有的字符。那么Unicode在计算机中如何存储和传输的呢？这就涉及编码的知识了。

<!-- [[[read_end]]] -->

Unicode相当于规定了字符对应的码值，这个码值得编码成字节的形式去传输和存储。最常见的编码方式是UTF-8，另外还有UTF-16，UTF-32 等。UTF-8 之所以能够流行起来，是因为其编码比较巧妙，采用的是变长的方法。也就是一个Unicode字符，在使用UTF-8编码表示时占用1到4个字节不等。最重要的是Unicode兼容ASCII编码，在表示纯英文时，并不会占用更多存储空间。而汉字呢，在UTF-8中，通常是用三个字节来表示。

```
>>> u'正'.encode('utf-8')
b'\xe6\xad\xa3'
>>> u'则'.encode('utf-8')
b'\xe5\x88\x99'
```

下面是 Unicode 和 UTF-8 的转换规则，你可以参考一下。

![](<https://static001.geekbang.org/resource/image/c8/ed/c8055321ed7e4782b3d862f5d06297ed.png?wh=852*570>)

## Unicode中的正则

在你大概了解了Unicode的基础知识后，接下来我来给你讲讲，在用Unicode中可能会遇到的坑，以及其中的点号匹配和字符组匹配的问题。

### 编码问题的坑

如果你在编程语言中使用正则，编码问题可能会让正则的匹配行为很奇怪。先说结论，在使用时一定尽可能地使用Unicode编码。

如果你需要在Python语言中使用正则，我建议你使用Python3。如果你不得不使用Python2，一定要记得使用 Unicode 编码。在Python2中，一般是以u开头来表示Unicode。如果不加u，会导致匹配出现问题。比如我们在“极客”这个文本中查找“时间”。你可能会很惊讶，竟然能匹配到内容。

下面是Python语言示例：

```
# 测试环境 macOS/Linux/Windows， Python2.7
>>> import re
>>> re.search(r'[时间]', '极客') is not None
True
>>> re.findall(r'[时间]', '极客')
['\xe6']
# Windows下输出是 ['\xbc']
```

通过分析原因，我们可以发现，不使用Unicode编码时，正则会被编译成其它编码表示形式。比如，在macOS或Linux下，一般会编码成UTF-8，而在Windows下一般会编码成GBK。

下面是我在macOS上做的测试，“时间”这两个汉字表示成了UTF-8编码，正则不知道要每三个字节看成一组，而是把它们当成了6个单字符。

```
# 测试环境 macOS/Linux，Python 2.7
>>> import re
>>> re.compile(r'[时间]', re.DEBUG)
in
  literal 230
  literal 151
  literal 182
  literal 233
  literal 151
  literal 180
<_sre.SRE_Pattern object at 0x1053e09f0>
>>> re.compile(ur'[时间]', re.DEBUG)
in
  literal 26102
  literal 38388
<_sre.SRE_Pattern object at 0x1053f8710>
```

我们再看一下 “极客” 和 “时间” 这两个词语对应的UTF-8编码。你可以发现，这两个词语都含有 16进制表示的e6，而GBK编码时都含有16进制的bc，所以才会出现前面的表现。

下面是查看文本编码成UTF-8或GBK方式，以及编码的结果：

```
# UTF-8
>>> u'极客'.encode('utf-8')
'\xe6\x9e\x81\xe5\xae\xa2'  # 含有 e6
>>> u'时间'.encode('utf-8')
'\xe6\x97\xb6\xe9\x97\xb4'  # 含有 e6

# GBK
>>> u'极客'.encode('gbk')
'\xbc\xab\xbf\xcd'  # 含有 bc
>>> u'时间'.encode('gbk')
'\xca\xb1\xbc\xe4'  # 含有 bc
```

这也是前面我们花时间讲编码基础知识的原因，只有理解了编码的知识，你才能明白这些。在学习其它知识的时候也是一样的思路，不要去死记硬背，搞懂了底层原理，你自然就掌握了。因此在使用时，一定要指定 Unicode 编码，这样就可以正常工作了。

```
# Python2 或 Python3 都可以
>>> import re
>>> re.search(ur'[时间]', u'极客') is not None
False
>>> re.findall(ur'[时间]', u'极客')
[]
```

### 点号匹配

之前我们学过，**点号**可以匹配除了换行符以外的任何字符，但之前我们接触的大多是单字节字符。在Unicode中，点号可以匹配上Unicode字符么？这个其实情况比较复杂，不同语言支持的也不太一样，具体的可以通过测试来得到答案。

下面我给出了在Python和JavaScript测试的结果：

```
# Python 2.7
>>> import re
>>> re.findall(r'^.$', '学')
[]
>>> re.findall(r'^.$', u'学')
[u'\u5b66']
>>> re.findall(ur'^.$', u'学')
[u'\u5b66']

# Python 3.7
>>> import re
>>> re.findall(r'^.$', '学')
['学']
>>> re.findall(r'(?a)^.$', '学')
['学']
```

```
/* JavaScript(ES6) 环境 */
> /^.$/.test("学")
true
```

至于其它的语言里面能不能用，你可以自己测试一下。在这个课程里，我更多地是希望你掌握这些学习的方法和思路，而不是单纯地记住某个知识点，一旦掌握了方法，之后就会简单多了。

### 字符组匹配

之前我们学习了很多字符组，比如\d表示数字，\w表示大小写字母、下划线、数字，\s表示空白符号等，那 Unicode 下的数字，比如全角的1、2、３等，算不算数字呢？全角的空格算不算空白呢？同样，你可以用我刚刚说的方法，来测试一下你所用的语言对这些字符组的支持程度。

## Unicode 属性

在正则中使用Unicode，还可能会用到Unicode的一些属性。这些属性把Unicode字符集划分成不同的字符小集合。

在正则中常用的有三种，分别是**按功能划分**的Unicode Categories（有的也叫 Unicode Property），比如标点符号，数字符号；按**连续区间划分**的Unicode Blocks，比如只是中日韩字符；按**书写系统划分**的Unicode Scripts，比如汉语中文字符。

![](<https://static001.geekbang.org/resource/image/2y/ae/2yy1c343b4151d14e088a795c4ec77ae.jpg?wh=1542*653>)

在正则中如何使用这些Unicode属性呢？在正则中，这三种属性在正则中的表示方式都是\p{属性}。比如，我们可以使用 Unicode Script 来实现查找连续出现的中文。

![](<https://static001.geekbang.org/resource/image/38/9c/383a10b093d483c095603930f968c29c.png?wh=1076*562>)

你可以在[这里](<https://regex101.com/r/Bgt4hl/1>)进行测试。

其中，Unicode Blocks在不同的语言中记法有差异，比如Java需要加上In前缀，类似于 \p{**<span class="orange">In</span>

****Bopomofo**} 表示注音字符。

知道Unicode属性这些知识，基本上就够用了，在用到相关属性的时候，可以再查阅一下参考手册。如果你想知道Unicode属性更全面的介绍，可以看一下维基百科的对应链接。

- [Unicode Property](<https://en.wikipedia.org/wiki/Unicode_character_property>)
- [Unicode Block](<https://en.wikipedia.org/wiki/Unicode_block>)
- [Unicode Script](<https://en.wikipedia.org/wiki/Script_(Unicode)>)

<!-- -->

## 表情符号

表情符号其实是“图片字符”，最初与日本的手机使用有关，在日文中叫“绘文字”，在英文中叫emoji，但现在从日本流行到了世界各地。不少同学在聊天的时候喜欢使用表情。下面是办公软件钉钉中一些表情的截图。

![](<https://static001.geekbang.org/resource/image/0e/e8/0ee6f3c217a13337b46c0ff41dc866e8.png?wh=958*202>)

在2020 年 3 月 10 日公布的Unicode标准 13.0.0 中，新增了55个新的emoji表情，完整的表情列表你可以在这里查看[这个链接](<http://www.unicode.org/emoji/charts/full-emoji-list.html>)。

这些表情符号有如下特点。

1. 许多表情不在BMP内，码值超过了 FFFF。使用 UTF-8编码时，普通的 ASCII 是1个字节，中文是3个字节，而有一些表情需要4个字节来编码。
2. 这些表情分散在BMP和各个补充平面中，要想用一个正则来表示所有的表情符号非常麻烦，即便使用编程语言处理也同样很麻烦。
3. 一些表情现在支持使用颜色修饰（Fitzpatrick modifiers），可以在5种色调之间进行选择。这样一个表情其实就是8个字节了。

<!-- -->

在这里我给出了你有关于表情颜色修饰的5种色调，你可以看一看。

![](<https://static001.geekbang.org/resource/image/2e/75/2e74dd14262807c7ab80c4867c3a8975.png?wh=944*390>)

下面是使用IPython测试颜色最深的点赞表情，在macOS上的测试结果。你可以发现，它是由8个字节组成，这样用正则处理起来就很不方便了。因此，在处理表情符号时，我不建议你使用正则来处理。你可以使用专门的库，这样做一方面代码可读性更好，另一方面是表情在不断增加，使用正则的话不好维护，会给其它同学留坑。而使用专门的库可以通过升级版本来解决这个问题。

![](<https://static001.geekbang.org/resource/image/cf/69/cf9fbeddf035820a9303512dbedb2969.png?wh=946*250>)

## 总结

好了，讲到这，今天的内容也就基本结束了。最后我来给你总结一下。

今天我们学习了Unicode编码的基础知识、了解了UTF-8编码、变长存储、以及它和Unicode的关系。Unicode字符按照功能，码值区间和书写系统等方式进行分类，比如按书写系统划分 \p{Han} 可以表示中文汉字。

在正则中使用Unicode有一些坑主要是因为编码问题，使用的时候你要弄明白是拿Unicode去匹配，还是编码后的某部分字节去进行匹配的，这可以让你避开这些坑。

而在处理表情时，由于表情比较复杂，我不建议使用正则来处理，更建议使用专用的表情库来处理。

![](<https://static001.geekbang.org/resource/image/76/3f/76924343bfb8d3f1612b92b6cab4703f.png?wh=1852*1468>)

## 课后思考

最后，我们来做一个小练习吧。在正则 xy{3} 中，你应该知道， y是重复3次，那如果正则是“极客{3}”的时候，代表是“客”这个汉字重复3次，还是“客”这个汉字对应的编码最后一个字节重复3次呢？如果是重复的最后一个字节，应该如何解决？

```
'极客{3}'
```

你可以自己来动动手，用自己熟悉的编程语言来试一试，经过不断练习你才能更好地掌握学习的内容。

今天的课程就结束了，希望可以帮助到你，也希望你在下方的留言区和我参与讨论，同时欢迎你把这节课分享给你的朋友或者同事，一起交流一下。

## 精选留言(19)

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34.jpeg)

  小乙哥

  有点看着蒙圈了，unicode和utf-8不都是编码吗？不都是对字符的编码？他们之间的关系没有搞太清楚

  作者回复: Unicode只规定了字符对应的码值，如果要进行存储或者网络传输等，这个码值得编码成字节，常见的编码方式是 UTF-8。 在内存中使用的时候可以使用unicode，但存储到硬盘里时就得转换成字节编码形式。

  2020-08-25

  **6

  **10

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293469-1390.jpeg)

  虹炎

  我的答案： 客重复3次，如果重复的是最后一个字节，就这样‘极(客){3}’, 给客加个括号分组。 请老师指正？

  作者回复: 对的，没问题👍🏻

  2020-06-29

  **12

  **9

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293469-1391.jpeg)

  lhs

  解释一下UTF-8的编码规则，会比较好理解文章中提到的Unicode和UTF-8的转化： 对于单个字节的字符，第一位设为 0，后面的 7 位对应这个字符的 Unicode 码点。因此，对于英文中的 0 - 127 号字符，与 ASCII 码完全相同。这意味着 ASCII 码那个年代的文档用 UTF-8 编码打开完全没有问题。 对于需要使用 N 个字节来表示的字符（N > 1），第一个字节的前 N 位都设为 1，第 N + 1 位设为 0，剩余的 N - 1 个字节的前两位都设位 10，剩下的二进制位则使用这个字符的 Unicode 码点来填充。

  作者回复: 感谢分享，大家这部分查一下资料了解下

  2021-03-24

  **

  **5

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293469-1392.jpeg)

  天蓬太帅

  请问例子里有一个表达式是findall(r’(?a)^.$’, ‘学’)，这里面的?a是啥意思？

  作者回复: 这是一种匹配模式，可以查询下 Python 文档，看下 re 模块的说明。指的是ASCII模式，可以让 \w 等只匹配 ASCII https://docs.python.org/3.8/library/re.html#module-re

  2020-07-19

  **

  **3

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293469-1393.jpeg)

  jooe

  感谢老师加了 js 的代码示例，对于不会后端语言的还是很友好的

  作者回复: 嗯，看到不少同学留言回复的是 js 代码，就加上了

  2020-07-03

  **2

  **1

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293469-1394.jpeg)![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,w_14.png)

  好运来![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  ​    print(u'正'.encode('utf-8'))    # 全角正则和全角字符串，不能匹配到    print(re.findall(r'＼ｗ＋', 'ａｂｃｄｅｆ')) # 输出[]    # 半角正则和全角字符串，能够匹配到    print(re.findall(r'\w+', 'ａｂｃｄｅｆ')) # 输出['ａｂｃｄｅｆ']    # 半角正则和半角字符串，能够匹配到    print(re.findall(r'\w+', 'abcdef')) # 输出['abcdef']    # 同上    print(re.findall(r'＼ｄ＋', '１２３４５６')) # 输出[]    print(re.findall(r'\d+', '１２３４５６')) # 输出['１２３４５６']    print(re.findall(r'\d+', '123456')) # 输出['123456']    # 同上    print(re.findall(r'＼ｓ＋', '　　')) # 输出[]    print(re.findall(r'\s+', '　　')) # 输出['\u3000\u3000']    print(re.findall(r'\s+', '  ')) # ['  ']     # “极客{3}”的时候，代表是“客”这个汉字重复 3 次    re.compile(r'极客{3}', re.DEBUG)    print(re.findall(r'极客{3}', '极客客客'))     # 思考题没有想到好的解决方法    print(re.compile(str(u'极客'.encode('utf-8')[-1:]).replace('b', '')), re.DEBUG)

  作者回复: 动手测试很赞，知识就是在练习中学习和掌握的。 思考题不难，可以参考下其它同学的答案，如果不支持，可以用括号把汉字括起来，这样就是整体重复了

  2020-06-29

  **

  **1

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293469-1395.jpeg)

  宁悦

  py3下测试 re.findall('极客{3}', '极客客客客气气气') 结果：['极客客客']

  作者回复: Python3对unicode支持还是挺好的

  2020-06-29

  **

  **1

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1396.jpeg)![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,w_14.png)

  Ocean D

  python2中的str等同于python3中的bytes类型， 而python2中的unicode等于python3中的str类型

  作者回复: 是的

  2021-10-29

  **

  **1

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1397.jpeg)

  octopus

  客重复三次，如果要极客重复三次‘(极客){3}’

  作者回复: 对的，用括号括起来，但也要注意子组要不要的问题

  2021-02-01

  **2

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1398.jpeg)

  追风筝的人

  Unicode（中文：万国码、国际码、统一码、单一码）是计算机科学领域里的一项业界标准。它对世界上大部分的文字进行了整理、编码。Unicode 使计算机呈现和处理文字变得简单。

  2020-10-24

  **

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1399.jpeg)

  gsz

  Unicode和gbk编码时，即刻，时间两个词都包含相同的16位数，为什么unicode匹配不了，而gbk的可以匹配？

  作者回复: 本质在于，正则是不是能知道哪几个字节要看成一个整体，如果它能认识，就不会识别出错；如果当成了多个单字节字符，就可能出错。

  2020-08-20

  **

  **1

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1400.jpeg)

  Juntíng

  JS 语言里, 客重复三次， 重复的是最后一个字节的话,将“客”独立为组再进行

  作者回复: 可以的

  2020-08-03

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  岁月轻狂

  老师，点好通配那里，（？a）是什么意思？

  作者回复: 这是一种匹配模式，可以查下Python文档，是以 ASCII 模式匹配

  2020-07-20

  **

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1401.jpeg)

  吕伟

  老师，为什么在sublime text3的正则搜索中输入\p{Han}，没有显示匹配结果。

  作者回复: Unicode属性很多地方不支持，你可以用go语言等试试

  2020-07-20

  **

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1401.jpeg)

  吕伟

  极(客){3}

  作者回复: 对的

  2020-07-19

  **

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1402.jpeg)![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,w_14.png)

  一步![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  对于 emoji 表情 ， unicode 没有一个属性可以表示的吗？ 比如 \p{P} 标点符号

  作者回复: 没有呢

  2020-07-01

  **

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1403.jpeg)

  Robot

  本来想把python3、pyhon2统一处理为 re.findall(ur'极客{3}', u'极客客客') 碰到Unicode类型的统一加u 结果在python3下报错 >>> re.findre.findall(ur'极客{3}', u'极客客客')  File "<stdin>", line 1    re.findre.findall(ur'极客{3}', u'极客客客')                        ^ SyntaxError: invalid syntax

  作者回复: 看上去像是你自己打错了，re.findall

  2020-06-29

  **3

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1403.jpeg)

  Robot

  课后思考：re.findall(ur'极{3}', u'极极极')

  作者回复: 对的，Python中是支持的

  2020-06-29

  **2

  **

- ![img](08%20_%20%E5%BA%94%E7%94%A81%EF%BC%9A%E6%AD%A3%E5%88%99%E5%A6%82%E4%BD%95%E5%A4%84%E7%90%86%20Unicode%20%E7%BC%96%E7%A0%81%E7%9A%84%E6%96%87%E6%9C%AC%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662309293470-1404.jpeg)

  盘胧

  对编码知识一直模棱两可的，顺着这个再梳理一下吧

  作者回复: 嗯嗯，编码的知识很基础很重要。搞懂了编码的知识，其他的也容易理解了。

  2020-06-29

  **

  **1
