# 02丨量词与贪婪：小小的正则，也可能把CPU拖垮！

作者: 涂伟忠

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/8a/9a/8a7c66b3f05b67a1c4135ce26656be9a.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/ab/c3/abcd1e70644114b1366756966c5e75c3.mp3" type="audio/mpeg"></audio>

你好，我是涂伟忠。在上一讲中，我们已经学习了正则中和一些元字符相关的内容。这一节我们讲一下正则中的三种模式，贪婪匹配、非贪婪匹配和独占模式。

这些模式会改变正则中量词的匹配行为，比如匹配一到多次；在匹配的时候，匹配长度是尽可能长还是要尽可能短呢？如果不知道贪婪和非贪婪匹配模式，我们写的正则很可能是错误的，这样匹配就达不到期望的效果了。

## 为什么会有贪婪与非贪婪模式？

由于本节内容和量词相关的元字符密切相关，所以我们先来回顾一下正则中表示量词的元字符。

![](<https://static001.geekbang.org/resource/image/2b/c3/2b03098dcc203c648a40f89a0ba77fc3.png?wh=1626*950>)

在这6种元字符中，我们可以用 {m,n} 来表示 （\*）（+）（?） 这3种元字符：

![](<https://static001.geekbang.org/resource/image/38/74/38ceb28add7794fe9ed069e08fb1b374.jpg?wh=1285*569>)

表示量词的星号（\*）和 加号（+）可能没你想象的那么简单，我用一个例子给你讲解一下。我们先看一下加号（+），使用 a+ 在 aaabb 中查找，可以看到只有一个输出结果：

![](<https://static001.geekbang.org/resource/image/2b/08/2b3e3f549e69fdd398c15d6b0bd44e08.png?wh=1038*590>)

对应的Python代码如下：

```
>>> import re
>>> re.findall(r'a+', 'aaabb')
['aaa']
```

加号应该很容易理解，我们再使用 a\* 在 aaabb 这个字符串中进行查找，这次我们看到可以找到4个匹配结果。

![](<https://static001.geekbang.org/resource/image/b0/4c/b0c582cbf8ec081bc798296b5471804c.png?wh=1032*614>)

使用Python示例如下，我们可以看到输出结果，也是得到了4个匹配结果：

```
>>> import re
>>> re.findall(r'a*', 'aaabb')
['aaa', '', '', '']
```

但这一次的结果匹配到了三次空字符串。为什么会匹配到空字符串呢？因为星号（\*）代表0到多次，匹配0次就是空字符串。到这里，你可能会有疑问，如果这样，aaa 部分应该也有空字符串，为什么没匹配上呢？

<!-- [[[read_end]]] -->

这就引入了我们今天要讲的话题，贪婪与非贪婪模式。这两种模式都必须满足匹配次数的要求才能匹配上。贪婪模式，简单说就是尽可能进行最长匹配。非贪婪模式呢，则会尽可能进行最短匹配。正是这两种模式产生了不同的匹配结果。

## 贪婪、非贪婪与独占模式

### 贪婪匹配（Greedy）

首先，我们来看一下贪婪匹配。在正则中，表示次数的量词默认是贪婪的，在贪婪模式下，会尝试尽可能最大长度去匹配。

首先，我们来看一下在字符串 aaabb 中使用正则 a\* 的匹配过程。

![](<https://static001.geekbang.org/resource/image/a7/ca/a7d62eee986938327d31e170cdd3caca.jpg?wh=1308*124>)

![](<https://static001.geekbang.org/resource/image/63/97/63e5c750b66f6eb914c73befdba43f97.jpg?wh=1311*446>)

a\* 在匹配开头的 a 时，会尝试尽量匹配更多的 a，直到第一个字母 b 不满足要求为止，匹配上三个a，后面每次匹配时都得到了空字符串。

相信看到这里你也发现了，贪婪模式的特点就是尽可能进行最大长度匹配。所以要不要使用贪婪模式是根据需求场景来定的。如果我们想尽可能最短匹配呢？那就要用到非贪婪匹配模式了。

### 非贪婪匹配（Lazy）

那么如何将贪婪模式变成非贪婪模式呢？我们可以在量词后面加上英文的问号(?)，正则就变成了 a\*?。此时的匹配结果如下：<br>

![](<https://static001.geekbang.org/resource/image/10/bc/10e40baa1194b17dcc57a089524a37bc.png?wh=1036*548>)

```
>>> import re
>>> re.findall(r'a*', 'aaabb')  # 贪婪模式
['aaa', '', '', '']
>>> re.findall(r'a*?', 'aaabb') # 非贪婪模式
['', 'a', '', 'a', '', 'a', '', '', '']
```

这一次我们可以看到，这次匹配到的结果都是单个的a，就连每个a左边的空字符串也匹配上了。

到这里你可能就明白了，非贪婪模式会尽可能短地去匹配，我把这两者之间的区别写到了下面这张图中。

![](<https://static001.geekbang.org/resource/image/3f/d1/3f95a3648980c1eb3c550fb34b46fad1.png?wh=2092*618>)

为了让你加深理解，我们再来看一个示例，这一次让我们查找一下引号中的单词。

从下面这个示例中，我们可以很容易看出两者对比上的差异。左右的文本是一样的，其中有两对双引号。不同之处在于，左边的示例中，不加问号时正则是贪婪匹配，匹配上了从第一个引号到最后一个引号之间的所有内容；而右边的图是非贪婪匹配，找到了符合要求的结果。

![](<https://static001.geekbang.org/resource/image/40/79/40c03d7a2cb990b35e4801589eca1379.png?wh=2112*790>)

### 独占模式（Possessive）

不管是贪婪模式，还是非贪婪模式，都需要发生回溯才能完成相应的功能。但是在一些场景下，我们不需要回溯，匹配不上返回失败就好了，因此正则中还有另外一种模式，独占模式，它类似贪婪匹配，但匹配过程不会发生回溯，因此在一些场合下性能会更好。

你可能会问，那什么是回溯呢？我们来看一些例子，例如下面的正则：

> regex = “xy{1,3}z”

> text = “xyyz”

在匹配时，y{1,3}会尽可能长地去匹配，当匹配完 xyy 后，由于 y 要尽可能匹配最长，即三个，但字符串中后面是个 z 就会导致匹配不上，这时候正则就会**向前回溯**，吐出当前字符 z，接着用正则中的 z 去匹配。

![](<https://static001.geekbang.org/resource/image/7a/88/7a9636b588963e5af9619837fe5a6888.png?wh=1204*458>)

如果我们把这个正则改成非贪婪模式，如下：

> regex = “xy{1,3}?z”

> text = “xyyz”

由于 y{1,3}? 代表匹配1到3个 y，尽可能少地匹配。匹配上一个 y 之后，也就是在匹配上 text 中的 xy 后，正则会使用 z 和 text 中的 xy 后面的 y 比较，发现正则 z 和 y 不匹配，这时正则就会**向前回溯**，重新查看 y 匹配两个的情况，匹配上正则中的 xyy，然后再用 z 去匹配 text 中的 z，匹配成功。

![](<https://static001.geekbang.org/resource/image/21/0c/2177c740a2d5dd805f3157d54636500c.png?wh=1002*378>)

了解了回溯，我们再看下独占模式。

独占模式和贪婪模式很像，独占模式会尽可能多地去匹配，如果匹配失败就结束，不会进行回溯，这样的话就比较节省时间。具体的方法就是在量词后面加上加号（+）。

> regex = “xy{1,3}+yz”

> text = “xyyz”

![](<https://static001.geekbang.org/resource/image/96/cb/96635e198c2ff6cf7b8ea2a0d18f8ecb.png?wh=1000*366>)

需要注意的是 Python 和 Go 的标准库目前都不支持独占模式，会报错，如下所示：

```
>>> import re
>>> re.findall(r'xy{1,3}+yz', 'xyyz')
error: multiple repeat at position 7
```

报错显示，加号（+）被认为是重复次数的元字符了。如果要测试这个功能，我们可以安装 PyPI 上的 regex 模块。

```
注意：需要先安装 regex 模块，pip install regex

>>> import regex
>>> regex.findall(r'xy{1,3}z', 'xyyz')  # 贪婪模式
['xyyz']
>>> regex.findall(r'xy{1,3}+z', 'xyyz') # 独占模式
['xyyz']
>>> regex.findall(r'xy{1,2}+yz', 'xyyz') # 独占模式
[]
```

你也可以使用 Java 或 Perl 等其它语言来测试独占模式，查阅相关文档，看一下你所用的语言对独占模式的支持程度。

如果你用 a{1,3}+ab 去匹配 aaab 字符串，a{1,3}+ 会把前面三个 a 都用掉，并且不会回溯，这样字符串中内容只剩下 b 了，导致正则中加号后面的 a 匹配不到符合要求的内容，匹配失败。如果是贪婪模式 a{1,3} 或非贪婪模式 a{1,3}? 都可以匹配上。

![](<https://static001.geekbang.org/resource/image/1d/b7/1dbf7d9fed42390edb3bf9ef9e0da7b7.jpg?wh=1619*372>)

这里我简单总结一下，独占模式性能比较好，可以节约匹配的时间和CPU资源，但有些情况下并不能满足需求，要想使用这个模式还要看具体需求（比如我们接下来要讲的案例），另外还得看你当前使用的语言或库的支持程度。

## 正则回溯引发的血案

学习到了这里，你是不是觉得自己对贪婪模式、非贪婪模式，以及独占模式比较了解了呢？其实在使用过程中稍不留神，就容易出问题，在网上可以看到不少因为回溯引起的线上问题。

这里我们挑选一个比较出名的，是阿里技术微信公众号上的发文。Lazada卖家中心店铺名检验规则比较复杂，名称中可以出现下面这些组合：

1. 英文字母大小写；

2. 数字；

3. 越南文；

4. 一些特殊字符，如“&”，“-”，“\_”等。


<!-- -->

负责开发的小伙伴在开发过程中使用了正则来实现店铺名称校验，如下所示：

```
^([A-Za-z0-9._()&'\- ]|[aAàÀảẢãÃáÁạẠăĂằẰẳẲẵẴắẮặẶâÂầẦẩẨẫẪấẤậẬbBcCdDđĐeEèÈẻẺẽẼéÉẹẸêÊềỀểỂễỄếẾệỆfFgGhHiIìÌỉỈĩĨíÍịỊjJkKlLmMnNoOòÒỏỎõÕóÓọỌôÔồỒổỔỗỖốỐộỘơƠờỜởỞỡỠớỚợỢpPqQrRsStTuUùÙủỦũŨúÚụỤưƯừỪửỬữỮứỨựỰvVwWxXyYỳỲỷỶỹỸýÝỵỴzZ])+$
```

这个正则比较长，但很好理解，中括号里面代表多选一，我们简化一下，就成下面这样：

```
^([符合要求的组成1]|[符合要求的组成2])+$
```

脱字符（^）代表以这个正则开头，美元符号（$）代表以正则结尾，我们后面会专门进行讲解。这里可以先理解成整个店铺名称要能匹配上正则，即起到验证的作用。

你需要留意的是，正则中有个加号（+），表示前面的内容出现一到多次，进行贪婪匹配，这样会导致大量回溯，占用大量CPU资源，引发线上问题，我们只需要将贪婪模式改成独占模式就可以解决这个问题。

我之前说过，要根据具体情况来选择合适的模式，在这个例子中，匹配不上时证明店铺名不合法，不需要进行回溯，因此我们可以使用独占模式，但要注意并不是说所有的场合都可以用独占模式解决，我们要首先保证正则能满足功能需求。

仔细再看一下 这个正则，你会发现 “组成1” 和 “组成2” 部分中，A-Za-z 英文字母在两个集合里面重复出现了，这会导致回溯后的重复判断。这里要强调一下，并不是说有回溯就会导致问题，你应该尽量减少回溯后的计算量，这些在后面的原理讲解中我们会进一步学习。

另外，腾讯云技术社区​也有类似的技术文章，你如果感兴趣，可以点击这里[进行](<https://zhuanlan.zhihu.com/p/38229530>)查看。

说到这里，你是不是想起了课程开篇里面提到的一句话：

> 如果你有一个问题，你想到可以用正则来解决，那么你有两个问题了。

> Some people, when confronted with a problem, think “I know, I’ll use regular expressions.” Now they have two problems.

所以一个小小的正则，有些时候也可能会把CPU拖垮，这也提醒我们在写正则的时候，一定要思考下回溯问题，避免使用低效的正则，引发线上问题。

## 最后总结

最后我来给你总结一下：正则中量词默认是贪婪匹配，如果想要进行非贪婪匹配需要在量词后面加上问号。贪婪和非贪婪匹配都可能会进行回溯，独占模式也是进行贪婪匹配，但不进行回溯，因此在一些场景下，可以提高匹配的效率，具体能不能用独占模式需要看使用的编程语言的类库的支持情况，以及独占模式能不能满足需求。

![](<https://static001.geekbang.org/resource/image/1a/75/1ad3eb0d011ba4fc972b9e5191a9f275.png?wh=1394*948>)

## 课后思考

最后，我们来做一个小练习吧。

有一篇英文文章，里面有很多单词，单词和单词之间是用空格隔开的，在引号里面的一到多个单词表示特殊含义，即引号里面的多个单词要看成一个单词。现在你需要提取出文章中所有的单词。我们可以假设文章中除了引号没有其它的标点符号，有什么方法可以解决这个问题呢？如果用正则来解决，你能不能写出一个正则，提取出文章中所有的单词呢（不要求结果去重）？

> we found “the little cat” is in the hat, we like “the little cat”

> 其中 the little cat 需要看成一个单词

好了，今天的课程就结束了，希望可以帮助到你，也希望你在下方的留言区和我参与讨论，并把文章分享给你的朋友或者同事，一起交流一下。

## 精选留言(125)

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34.jpeg)

  Geek.S.

  以前只知道贪婪模式和懒惰模式，原来还有一个独占模式，贪婪和非贪婪都会发生回溯，结合文中给的案例链接，知道了 NFA 和 DFA (这个老师应该在后面讲匹配原理时会讲到)。难怪余晟老师说学会正则后务必要学会克制。 如果只是判断文本是否符合规则，则可以使用独占模式; 如果需要获取匹配的结果，则根据需要使用贪婪或非贪婪。 作业题: (这里需要获取单词，不能使用独占模式) \w+|“[^”]+”  (注意: 例句中的双引号是中文状态下的) 结果(10 次匹配, 48 步): ['we', 'found', '"the little cat"', 'is', 'in', 'the', 'hat', 'we', 'like', '"the little cat"']  相应的 Python 代码: >>> import re >>> text = '''we found “the little cat” is in the hat, we like “the little cat”'''  # 注意: 例句中的双引号是中文状态下的 >>> pattern = re.compile(r'''\w+|“[^”]+”''') >>> pattern.findall(text) ['we', 'found', '"the little cat"', 'is', 'in', 'the', 'hat', 'we', 'like', '"the little cat"']  示例: https://regex101.com/r/l8hkqi/1

  作者回复: 对的，务必克制，搞懂原理才能用的更好。 答案是对的👍🏻

  2020-06-15

  **8

  **41

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056841-453.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  一步![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  老师，对于文中的这个语句 regex.findall(r'xy{1,3}+z', 'xyyz') 这里是独占模式，不进行回溯。这里在尽可能多的匹配第三个 y的时候匹配失败，不应该是直接匹配失败 返回空数组吗？ 怎么是返回 xyyz 呢？ 如果返回 xyyz 不就进行回溯了吗？

  作者回复: y出现一到三次，匹配量词以后，没有y了，就接着用正则中的z去匹配，没有回溯。如果+后面还有个y就会匹配不上，如果是贪婪或非贪婪就可以匹配上。 独占模式不回溯这个说法其实不够准确，可以理解成“独占模式不会交还已经匹配上的字符”，这样应该就能理解了。 独占模式比较复杂，实际场景中用的其实不多，先了解就好了

  2020-06-15

  **4

  **15

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056842-454.jpeg)

  Robot

  测试结果跟文中的不符 >>> import re >>> re.findall(r'a*?','aaabb') ['', '', '', '', '', '']

  2020-06-16

  **3

  **14

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056843-455.jpeg)

  BillionY

  \w+|“[^”]*”    \w+|“[\w\s]+? \w+|“.+?” 还有第四种方法吗?

  作者回复: 建议第一种，写得方式有很多，但思路都是一样的

  2020-06-15

  **5

  **11

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/ibZVAmmdAibBeVpUjzwId8ibgRzNk7fkuR5pgVicB5mFSjjmt2eNadlykVLKCyGA0GxGffbhqLsHnhDRgyzxcKUhjg/132)

  pyhhou

  思考题： ".+?"|[^\s|,]+ 关于回溯，是不是就是递归调用函数栈的原理？拿 xy{1,3}z 匹配 xyyz 举例，步骤如下： 1. 正则中的 x 入栈，匹配上 text 的第一个字符 x 2. 正则中的 y 入栈，匹配上 text 中的第二个字符 y 3. 因为这里没有加问号，属于贪婪匹配，正则中的 y 继续入栈，匹配上 text 中的第三个字符 y 4. 正则中的 y 继续入栈，但是这个时候 y 和 z 不匹配，执行回溯，就是当前正则的第三个 y 出栈 5. 用范围量词后的字符 z 继续入栈匹配，匹配上 text 的最后一个字符，完成匹配

  作者回复: 👍🏻思路很新颖，你说的是匹配的过程，并不是回溯的过程，正则回溯是基于状态机的

  2020-06-15

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/10/1a/66/2d9db9ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  苦行僧

  w+|“[^”]+”, w+ 看懂了, 但 后面的没看懂?

  作者回复: 引号里面是非引号出现一到多次

  2020-08-13

  **

  **5

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056843-457.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  William![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  js版 ```javascript let str = `we found "the little cat" is in the hat, we like "the little cat"` let re = new RegExp(/"[^"]+"|\w+/, 'g') let res = str.match(re) console.log(res) ```

  作者回复: 没问题，认真学习的同学，赞，多动手练习才能更好地掌握

  2020-06-23

  **

  **4

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056842-454.jpeg)

  Robot

  建议老师统一下正则的运行环境。

  作者回复: 文章中用的都是Python3，毕竟2已经不维护了

  2020-06-16

  **

  **4

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056843-458.jpeg)

  飞

  [a-z]+|“[^“]+”

  作者回复: 思路没问题，可以再考虑下单词中有大写字母呢？尽量考虑全面些

  2020-06-15

  **2

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/c4/06/c08eca4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Zion W.

  引号是全角还是半角？

  作者回复: 全角。 这个例子本来是想着半角的，但默认的字体下没看出来写的是全角。捂脸ớ ₃ờ

  2020-06-27

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/33/07/8f351609.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  JustDoDT

  我总结一些，正则这东西： 1、掌握基本规则 2、多用，多试验 我的运行环境：MacOS，python 3.6.2 我匹配的英文引号 " 例子 text = 'we found "the little cat" is in the hat, we like "the little cat"' re.findall(r'"(.*?)"', text) 输出：['the little cat', 'the little cat'] 中文引号版本 “ ” text = 'we found “the little cat” is in the hat, we like “the little cat”' re.findall(r'“(.*?)”', text) 输出：['the little cat', 'the little cat'] so easy

  作者回复: 👍🏻，就是要有这种自信。思考题目里面如果要查找所有单词，引号外面的也找出来呢？

  2020-06-18

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/db/64/06d54a80.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  中年男子

  还有就是文章中的例子： xy{1,3}+yz 去匹配 xyyz，我的理解是用y{1,3}尽可能多的去匹配， 也就是 xyy之后，用第三个y 去匹配z，因为不回溯，到这里就失败了， 还没到正则中z前面那个y。 还请解惑。

  作者回复: y一到三次独占模式，虽然只匹配到了两个，但还是满足了次数要求，这时候没失败，继续看下一个，后面的y匹配不上z才失败的

  2020-06-15

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/db/64/06d54a80.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  中年男子

  老师， 我有个问题， 既然是独占模式了， 那{1,3} 这种量词是不是就没什么意义？直接{3}不就行了

  作者回复: 不是，独占模式类似于贪婪模式，如果只有两个a也能匹配上，直接写3只能切配3个a了，你可以理解成正则中a匹配不上字符串中的b，会接着用正则后面的内容去匹配b

  2020-06-15

  **2

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/25/83/28/ee3d71cc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  蓝猫冬天变肥猫

  老师， [a-zA-Z]+|“.+? 匹配出来10个 ”但是为什么我用.+|“.+?”把整个句子都匹配上了呢？

  作者回复: 因为.+匹配范围很广，包括空格，[a-zA-Z]+这个不包含空格

  2021-07-27

  **

  **1

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056843-463.jpeg)

  lhs

  老师，不知道可不可以这样来理解文中提到的内容：把回溯理解成一个“动态适配”的过程，贪婪和非贪婪都会根据正则表达式的规则去适配出一个符合的结果，而独占模式只是尝试一次去匹配，比较硬气，all in之后没有结果了，就放弃掉了，不会去动态调整找到符合规则的结果。 比如对于regex.findall(r'a{1,3}ab', 'aaab') 这个例子，因为表示量词是默认开启贪婪模式的，所以a{1,3}会尽可能地匹配三个aaa,剩下的表达式ab会去匹配字符串'aaab'中b，发现ab和b对不上，这个时候由于贪婪模式会发生回溯，它就会去上次匹配的结果"aaa"中借一个a过来，被借了一个a之后上次的匹配结果变为aa,发现也是符合a{1,3}这个规则的(可匹配1-3个a),借到a之后,刚好可以匹配到ab字符串，匹配结束。 再来理解regex.findall(r'a{1,3}?ab', 'aaab') 这个例子,加?号之后，表示非贪婪模式,a{1,3}会尽可能少匹配a,第一次会匹配到一个a,接下来正则剩ab,用a去匹配'aaab'中的第二个a，最后正则剩一个b，它会去先匹配'aaab'中第三个a，发现对不上，因为非贪婪模式也会发生回溯，所以会尝试把匹配不上的a送到上次的匹配结果中，由于上次匹配结果经历了第一次和第二次的匹配，为aa，再加上第三次倒贴过来的a，就变成了aaa，还是符合a{1,3}这个规则的，最后面剩下的规则b刚好匹配上最后一个字符b，匹配结束。 对于regex.findall(r'a{1,3}+ab', 'aaab')这个例子，由于是独占模式，有贪婪的特性，最开始就会去尝试匹配一个最大的结果，a{1,3}会去匹配‘aaab’中的三个a，字符串'aaab'在经历最大匹配之后就只剩下一个b了，跟剩下的规则ab匹配不上,最终匹配结果为空。由于独占模式不发生回溯，也就不会像贪婪模式一样会去上次的匹配结果中借一个a来和ab匹配，而是发现对不上之后就马上放弃结束掉。 上面就是我的理解，再结合老师文章中总结的要点，我觉得会更明白。 贪婪匹配中的回溯：后面匹配不上，会吐出已匹配的再尝试 非贪婪匹配中的回溯：后面匹配不上，会匹配更长再接着尝试 独占模式：不会发生回溯，匹配不上即失败。

  2021-02-02

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/18/a6/d4/f0207473.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  简简单单

  老师我有个疑问求解答:  正则: \w+|“.+?” 字符串 : we found “the little cat” is in the hat, we like “the little cat” 结果的确是把每个单词单独匹配到了, 并且 引号的也当成一个单词了, 那么请问 为什么 \w+ 不会把 引号内的单词匹配到呢, 为什么不会出现 the 与 “the little cat” 共存呢 正则匹配的过程是怎么样的 ?

  作者回复: 引号也匹配上可以用断言解决。 不会共存可以看看后面的匹配原理部分，一个分支能匹配上就不会看另在一个分支，如果失败了看另外一个分支

  2020-07-21

  **2

  **1

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056844-465.jpeg)

  洛奇

  本人仅看这篇文章后，还是不明白为什么“匹配越南店铺名”的例子会有回溯？从正则表达式xy*yz匹配xyz的例子可以大概明白什么是回溯，但是匹配店铺名的例子中，两个符合条件的组合是“或”的关系，而不是xy*yz里的y*和y的前后关系，而且正则头尾已经用^和$包裹了，为什么也会产生回溯呢？

  作者回复: 多分支结构，在一个分支不满足时，就会再次尝试另外一个分支，会造成回溯。也就是一个字符会被正则匹配两次，这主要是因为我们日常用的大多数正则都是基于 NFA 实现的。这些内容会在原理篇中进一步讲解，到时候看了如果还有问题可以再留言。

  2020-07-03

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/bd/ec/cc7abf0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  L

  老师我完全没有办法理解这个空白符。 确实将 aaabb 拆成了 空白符+a+空白符+... 可以理解 但是这样做的意义是什么？？？  此外 我想问一下对于同一个正则表达式 不同的语言引擎的解释是可以不一样的吗？ 一样用这个进行举例。 const regex1 = new RegExp('a*?','g') const str1 = 'aaabb' console.log(str1.match(regex1))  // ['','','','','','',''] 预期的结果不是 a a a b b 吗？ 而对于这个结果我做了一个尝试性的解读。 macth返回的不是字符串的子串，而是正则的结果 因此 返回的结果应该为 ''  a aa a* 因此即使是a匹配到了 返回的也是对应的正则的匹配形式 空字符 因为非贪婪模式下并不需要到a就可以返回了 可是这样得话 为什么会多出来一个 空白符 呢？ 对于这个空白符 个人的理解是 当输出一个空字符的时候 a* 也是可以匹配成功的，因此需要对这种情况进行处理，那么是在什么时候进行处理的呢？在一开始的时候进行处理的，还是在结尾的时候进行处理的？ 老师在讲原理的时候会聊这块吗？ 我的问题主要有三点 1、不同的语言对正则的理解是不是可能不一样 2、为什么python要对字符串进行加上空字符的处理 3、为什么JavaScript的match会返回这样的结果 希望老师能给一个解答 还有一个请求 老师在在讲原理的时候，能不能提一下这块

  作者回复: 感谢你写这么长的问题，一看就是非常热心学习的同学 1. 不同语言实现确实可能不一样，这也是正则学起来比较麻烦的地方，在Python2和Python3，甚至 Python3.6 和 Python 3.7 都不一样。但也不要担心，绝大部分特性都是一样的，后面讲解流派的时候会涉及到其它的一些原因。 2. 在正则的流派，匹配的原理相关章节中会有一些讲解，但语言和工具太多，也不太可能讲的非常全面。 3. 不要过于纠结细小的点，就像你说的，如果尝试去理解，也能大概猜测出它是怎么匹配的。 另外，在用到相关的工具或语言的时候，自己要去测试看下结果，千万不要不做任何测试，直接就使用。

  2020-06-19

  **4

  **1

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056842-454.jpeg)

  Robot

  课后思考： /"[^"]+"|[a-z]+/g

  作者回复: 思路没问题，可以考虑下包含大写字母的情况，可以看看其他同学的答案

  2020-06-16

  **

  **1

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056844-467.jpeg)

  卡尔

  ^([a-zA-Z]+|"[^"]+")$

  作者回复: 查找所有单词，不需要^和$

  2020-06-16

  **

  **1

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34.jpeg)

  Geek.S.

  独占模式是贪婪的，很好奇有没有非贪婪的独占模式？老师可不可以分析一下这个问题？👻

  作者回复: 没有呢，独占模式其实日常用的不是很多，使用场景有限，了解一下就行了

  2020-06-15

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/78/62/354b1873.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  clever_P

  [a-zA-Z]+|"[a-zA-Z ]+"

  作者回复: 思路没问题，单词建议用\w 表示

  2020-06-15

  **

  **1

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056844-469.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  程志强

  \w+|“(\w{1,10}|\s)+”

  2022-07-25

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056844-470.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  可乐加冰

  老师，我也能写正则了，交作业：[a-zA-Z]+|“.+”

  2022-07-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2c/19/9a/c9fd018e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  ZhuSP

  ([a-z]+)|("[a-z|\s]+") https://regex101.com/r/kjCnDq/1

  2022-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2c/19/9a/c9fd018e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  ZhuSP

  ([a-z]+)|("[a-z|\s]+")

  2022-07-10

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/132-1662309056844-472.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  奔波儿灞

  “[a-z ]+”|[a-z]+

  2022-07-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/27/4d/15/4d13caf5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  哆啦A靖

  \w+|“[\w ]+” 10次匹配，48步

  2022-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/2c/d8/442c13dc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  李小文

  \w+|“.+?”

  2022-05-11

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_deab10

  贪婪模式：“由于 y{1,3}? 代表匹配 1 到 3 个 y，尽可能少地匹配。匹配上一个 y 之后，也就是在匹配上 text 中的 xy 后，正则会使用 z 和 text 中的 xy 后面的 y 比较，发现正则 z 和 y 不匹配，这时正则就会向前回溯” 如果是y{1,1000}，匹配 xyyz 是从1000,999,998,997.....这样一直回溯吗？

  2022-05-11

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_deab10

  贪婪模式：“在匹配时，y{1,3}会尽可能长地去匹配，当匹配完 xyy 后，由于 y 要尽可能匹配最长，即三 个，”， 独占模式既然也是尽可能多的匹配，为什么：“y{1,3}+尽可能多的匹配了两个y” 为什么不是3个y?

  2022-05-11

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_deab10

  贪婪模式：“在匹配时，y{1,3}会尽可能长地去匹配，当匹配完 xyy 后，由于 y 要尽可能匹配最长，即三 个，”，

  2022-05-11

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_deab10

  没理解作者的表述。 1.贪婪模式：“...但字符串中后面是个 z 就会导致匹配不上，这时候正则就会向前回溯，吐出当前字符 z，”，z在哪？被谁“吐”出来？既然是“吐出来”，它又是被谁、在什么时候“吃进去”的？

  2022-05-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2a/8e/30/bd1129f0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  BIT-312

  (\w++)|(\“.+?\”) 这种情况下非贪婪，因此判断出最短的就退出 (\w++)|(\“[^\”]++\”) 这种情况，判断出下一个不是 "\”" 就行，找到第一个是"\”"的位置退出，然后匹配完成，前提条件就是不能乱用格式，还用独占模式加快匹配 这俩都在那个网页上通过了，不过感觉上只要格式不是错误的比如 ““” 这种或者 “””，应该都是可行的

  2022-05-10

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83eoib6BjEV4KPEaibP0MSA3f9VA21H2h4P8uIaatDqW7krBqJF6YH136TCNKGSbERHscOIEp0TlYlcbw/132)

  若失

  “[a-zA-Z\s]+”|[a-zA-Z]+ 10matches 32steps

  2022-05-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/34/0e/018f88b0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Alun.

  /“.+?”|\w+

  2022-04-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/60/41/8f9210f6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Gecho

  思考题： [\w]+|“[\w| ]+”

  2022-04-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/20/c8/b16eb6ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程同学

  老师好，这篇文章我看了很多遍，发现在贪婪模式下的“尽可能多地匹配”，与独占模式下的“尽可能多地匹配”是不一样的，是这样吗

  作者回复: 是的，后面的文章有更详细的解释

  2022-03-15

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/cf/dc/f119f657.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  0ck0

  ab 之间有空白字符？  a* 匹配ab 2次

  2022-03-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ef/a2/6ea5bb9e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  LEON

  aaabb 对应下标012345  怎么对应的，没看明白a前面是0吗？

  2022-02-25

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Geek_039a5c

  这里举例有点让人不明白。  对于独占模式的举例，为啥不是用xy{1,3}+z ？

  2022-02-13

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056844-465.jpeg)

  洛奇

  老师，用"[^"]*?"是否比".*?"性能更好？因为我认为前者已经先用引号和相应字符比较过了，直觉告诉我前者可能性能更好。

  作者回复: 对的

  2022-01-30

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056845-482.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Roway

  xy{1,3}+yz    改成独占模式 xy{3}yz 不可以吗？

  作者回复: 自己多测试，根据结果反复思考，这样才能记得牢

  2021-12-09

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/5OwNu4ibknPOExCibicJib5rB0Wib1EV7sMd2OxcZFu58hc28UTeGvAVGwtrn4k9XZxe3DicITUU8SV0AgxatTiaH06mw/132)

  anine

  在regex101上 ,关于a*?匹配aaabb,只有php语言给出了预期的答案   其他语言均为空格 。能解释下吗 ？实在很困惑

  2021-12-01

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056846-484.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  咱是吓大的![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  贪婪模式是从多到少进行匹配，非贪婪模式是从少到多进行匹配，如果匹配不成功则回溯 独占模式从多到少进行匹配，且只匹配一次 老师我这么理解对吗？

  作者回复: 关键是回溯，后面课程有进一步讲解

  2021-11-11

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056846-485.jpeg)

  风中劲草

  作业： “[\s\S]+?”

  2021-10-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/29/9c/a2/960e58ce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  娄

  (".+?")|\w+

  2021-10-29

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/iaU7yVx9WnPfnpJBwGCCLWBy9SvkVQRMxmOBxyZTqRYFgPEEYa0pfUHAek6TNEp6SdFRwumUycSeCL7DaM6FotQ/132)

  DMY

  作业： “.+?”|\w+

  2021-10-16

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056846-488.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Johnny

  \w+|“[^”]+”

  2021-09-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b2/2a/4d908144.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  月

  老师你好，请问javascript有独占模式吗？一种语言有没有这种模式，怎么查啊？

  2021-09-13

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056846-490.jpeg)

  李建飞

  打卡：("([a-z]+[ ]?)+"?)|([a-z]+)

  2021-09-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/ce/cc/0ac98381.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bug管理员

  作业练习搞了半天终于弄出来了，“(.+?)”|[a-z]{2,}+

  作者回复: 赞，动手练习认真思考

  2021-09-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/29/db/3a/a9113de0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  向死而生

  a?是贪婪模式还是非贪婪模式

  作者回复: 贪婪模式。表示次数的，后面再加个问号才是非贪婪模式

  2021-08-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/29/db/3a/a9113de0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  向死而生

  非贪婪匹配的最后一个例题没懂，”不是也能被.给匹配吗，为什么非贪婪模式下不匹配？

  作者回复: 非贪婪是尽可能少的匹配，即使能匹配上，也尽量选最短的

  2021-08-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/aa/62/78b45741.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Morty

  贪婪模式的回溯让我想到了状态机的 reconsume

  2021-07-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/ec/ff/b268bea7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  king

  https://regex101.com/r/mlQIun/1 “.*?”|\w+

  2021-04-18

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/132-1662309056846-495.jpeg)

  SHIFT

  \w+[^\s”,]|“[\w\s]+”

  2021-04-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/42/90/1b402e4a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  熊大

  aaabb后面为啥还有个空字符？

  2021-03-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/21/d8/0f/b97bb632.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Invincible_JM

  \b\w+\b|“.+?”

  2021-02-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/ef/59/6eda594b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  octopus

  \w+\s?|“.+?”虽然这个可以匹配出来，但是不知道思路对不对

  2021-01-25

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056847-499.jpeg)

  取悦

  看了其他人答案，应该是\w+|“[^”]+”。 我之前写的是\w+|“[A-Za-z\s]+”，会有个问题，如果双引号之间有其他字符，就匹配失败了。

  2020-12-27

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056847-499.jpeg)

  取悦

  思考题： \w+|“[A-Za-z\s]+”

  2020-12-27

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/132-1662309056847-500.png)

  Geek_be6cee

  "\w+(?:\s+\w+){0,}"|\w+

  2020-12-04

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056847-501.jpeg)

  每天晒白牙

  \w+| “[\w ]+” 我这种解析出来的 “the little cat” 和 “the little cat” 都带着引号，老师的意思应该是把引号都过滤掉吧，没找过突破口

  2020-11-27

  **1

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Wm89LvRU2Ns0MMGYDXwBkeKvID7hxtypUzoPuAB0PDBDriauQEBwiaVDgXgz2TBANmH44w2GSibjlcmuicq8YnJKTA/132)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  sspantz

  老师，我想了好久，文章来回看了好多遍也不知怎么解决，又不想跳出文章的学习去查资料，毕竟是作业。思考了半小时后，我被自己蠢哭了，我以为要匹配"xxx"里面的xxx, 原来只要匹配"xxx"就可以了。 于是答案不是很明显是: \w+|"[^"]+"。但问题来了，如果我想单独匹配"xxx"里面的xxx怎么办呢？

  作者回复: 认真思考一定会有收获，我刚开始接触也是想了好久的。 至于后面的问题，等你学习了后面断言相关的知识，你就有答案了

  2020-11-17

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056847-503-1662309056883-604)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  IT蜗壳-Tango

  打卡学习，每天学习一点点。加油＾０＾~

  作者回复: 加油

  2020-11-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/9c/fa/0b9d9832.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  松鼠工作室

  来一个正则 (([a-z]|[A-Z])+)|(".*?") 不知道对不对 望请老师解答啊

  作者回复: 中括号里面可以有多个范围，所以分支写法没必要，可以参考一下其他同学答案

  2020-11-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/52/0e/c5ff46d2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Condor Hero

  老师您讲解非贪婪模式的例子，我用 python3 运行没有疑问，但是用 JS 的执行效果如下������： ```js > "aaabb".match(/a*/g) < (4) ["aaa", "", "", ""] > "aaabb".match(/a*?/g) < (6) ["", "", "", "", "", ""] ``` 非贪婪模式的匹配结果，JS 和 python3 的结果为啥不一样？

  2020-11-06

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/EoiaYtjYmiaRal5Of1a05z3klUibJczR6DfvkXdQsRicamxUYMibf8icAKQ5wo1kkXEzfgZRaTmBCxAO2KMQ1AJjaRDQ/132)

  风一样的小雅静

  +号也是量词，那量词后面再加独占符+，是要用两个+号么

  作者回复: 对的

  2020-11-06

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTKvnkXZiaop0hbe9w5kuAvf6d17suDPB6W1j2hlYPJ43eRogABUic5pUV7ia5rPHjXLWEfDZLHiafMUiaQ/132)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  marsly

  老师，我用\w+|".+?"匹配出来之后， "the little cat"，我想直接匹配到the little cat不带引号怎么办

  作者回复: 后面有讲，可以用环视实现

  2020-10-22

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056847-508.jpeg)

  陈迪

  \# Py3 import re def method1(s):    # print("running method1: ")    rexp = r'''\w+|".*?"'''     res = re.findall(rexp, s)    # print(res) def method2(s):    # print("running method2: ")    rexp = r'''\w+|"[^"]*"'''     res = re.findall(rexp, s)    # print(res) s = '''we found "the little cat" is in the hat, we like "the little cat"''' method1(s) print("-------------") method2(s) # method1的关键点：加了个问号变成了非贪婪匹配，不加的话会贪婪匹配，直接匹配到最后一个"，得出错误结果 # method2的关键点：用了[^"]，很巧妙，直接使用默认的贪婪匹配即可 # 两者都会回溯，但是method2的效率会更高；method1匹一下就不匹了，每次都得回溯一下，面对""内是长单词就很吃亏了；而method2基于贪婪，每次几乎一遍就能“做对”  # 下面验证一下： # 把函数体内的print禁止 from timeit import timeit short_s = '''we found "the little cat" is in the hat, we like "the little cat"''' t1 = timeit(lambda: method1(short_s), number=100000) # 0.37051227 t2 = timeit(lambda: method2(short_s), number=100000) # 0.33821018 # 差距不明显 print(f"t1:{t1}, t2:{t2}") long_s = '''we found "the little cat the little cat the little...把它搞得很长很长来试验..." is in "" the hat, we like "the little cat"''' t1 = timeit(lambda: method1(long_s), number=100000) # 0.745851991 t2 = timeit(lambda: method2(long_s), number=100000) # 0.374672004 # 差距变明显了。。印证了之前的分析 print(f"t1:{t1}, t2:{t2}") # 补充： # 这里应该用*，不是+，用来""里头内容为空的情况 # 总感觉正则很危险，当前写的正则，在将来面对哪个意料之外的输入的时候，就不好使了 # 而要命的是输入往往是用户输入的，当前能想到的解决办法就是多加测试用例。

  2020-10-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/bd/7a/37df606b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  乔帆 Kayla

  老师您好，请问独占模式如何不回溯，实现{1,3}范围的选取的呢？例如 xy{1,3}z，xyyz和xyyyz都可以满足，那么如何做到尽可能多的匹配y，又不回溯呢？

  作者回复: 其实不用过分担心回溯问题，而且需要注意回溯后的工作量，后面的原理篇有进一步讲解，可以去看看

  2020-09-27

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1c/bd/7a/37df606b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  乔帆 Kayla

  课后题： \w+|".+?" 问题1： 独占模式 a{1,3}+ab 是不是和 a{3}ab 等价? 问题2： ^([符合要求的组成1]|[符合要求的组成2])+$ ，怎么改为独占模式，是^([符合要求的组成1]|[符合要求的组成2])++$ 吗？ 望老师解答。

  作者回复: 问题1：不等价，考虑一下 a{1,3}+xy 能匹配上 aaxy 不一定是3个a 问题2，对的，后面原理篇会有进一步讲解

  2020-09-27

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/62/a5/43aa0c27.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  TKbook

  这课真买对了。以前虽然会用。但是真不知道还有独占模式。。

  作者回复: 感谢支持，这个问题，后面原理那儿还有更详细的讲解

  2020-08-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/39/8c/089525aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小乙哥

  (“([a-zA-Z]+\s?)+”|[a-zA-Z]+)

  2020-08-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/ce/5d/224b08b1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  坐观云起

  非贪婪模式的例子： 左边的例子，如果真的换行啦，匹配的是2个结果。 不换行，匹配的才是1个结果。

  作者回复: 嗯，根据测试结果自己思考下原因，搞懂了就行

  2020-08-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1d/3a/8a/76b03c2f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  南辞

  答案：\w+|“\w*”   这样匹配的是不是就是指定引号里面只能是字母。 我这里有个问题，就是说\s代表任意的空白符，那么怎么样匹配abc god  ghs这种形式的呢？我尝试了.*\s. *这样没有成功，就是把上面所有的看成一个整体的形式？老师帮解答一下，谢谢😜。

  作者回复: 空白和后面的字符得括号括起来，重复多次就可以了 .+(\s+.+)*

  2020-07-15

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056848-514.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  大神博士

  交作业 /“.+?”+|\w+/g

  作者回复: 没问题

  2020-07-13

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056848-515.jpeg)

  吕伟

  \w+|".+?"

  作者回复: 可以的

  2020-07-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/62/ef/6937b490.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ElliON

  “[a-z\s]*?”|[a-z]{1,} 结果： we found “the little cat” is in the hat we like “the little cat”

  作者回复: 可以的，[a-z]{1,} 一到多次也可以直接使用加号，[a-z]+

  2020-07-06

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056849-517.jpeg)

  D![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  \w+|“.*?”

  作者回复: 对的

  2020-07-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/60/07/d9a81a7b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小熊阿牛

  请教一个问题，\w+|“[^"]+"，按照“[^"]+"匹配为一个短语，而不按照\w+匹配为多个单词，应该如何理解？是因为贪婪模式，|的内在逻辑，还是其他原因呢？

  作者回复: 因为\w不能匹配引号，所以就走了第二个分支，当第二个分支把字符串（你说的短语部分）用掉之后，后面再匹配就不会尝试 \w+ 部分了。

  2020-07-04

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  岁月轻狂

  \w+|“.+?”

  作者回复: 可以的

  2020-07-03

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056849-519.jpeg)

  Crystal

  老师好，请问贪婪模式非贪婪模式中空字符的监测是什么规律呢，没搞懂，贪婪模式中 ‘aaabb’只有 01234 位，为什么会出来第5位呢；非贪婪模式中又为什么会出现 a 左侧的空字符，难道字符串中每一位左右都存在空字符么

  作者回复: 对的，你可以这么理解，匹配开始的时候是空，匹配完了剩下一个空字符。

  2020-07-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/c8/67/5489998a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  John Bull![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/8ac15yy848f71919ef784a1c9065e495.png)

  ECMAScript现在也不支持独占模式！

  作者回复: 独占模式只有少数语言支持，平时用的确实用的不多，不要太纠结哈

  2020-06-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c4/06/c08eca4c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Zion W.

  ".+?"|[a-z]+ 两部分： 前面一部分用lazy模式匹配所有在两个引号之间的单词。 后面一部分匹配任意连续的字母。

  作者回复: 思路正确，问题不大，单词一般用 \w+ 来表示，[a-z]只有小写。

  2020-06-27

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Milen

  \>>> import re >>> text = '''we found “the little cat” is in the hat, we like “the little cat”'''  >>> pattern = re.compile(r'''\w+|“[\w ]+”''') >>> pattern.findall(text) ['we', 'found', '"the little cat"', 'is', 'in', 'the', 'hat', 'we', 'like', '"the little cat"'] 但是匹配到的the little cat 带有引号，如何将引号去掉呢？

  作者回复: 后面的断言一节有讲，可以看看

  2020-06-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/22/82/36e3c5d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Adams

  [a-z]{1,}+|".+?"

  作者回复: 可以，你这个只包含了小写字母，但给的这个示例可以用，单词一般使用 \w表示，这样大写字母也能涵盖了

  2020-06-23

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056849-522.jpeg)

  Peace

  更正，[^“][\w ]+?[”$]|\w+

  作者回复: 为什么要在中括号中使用 ^ 和 $ 呢？

  2020-06-23

  **2

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056849-522.jpeg)

  Peace

  "[\w ]+?"|\w

  作者回复: 注意一下，想匹配的是单词，不是单个字母哈

  2020-06-23

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056849-523.jpeg)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  盘胧

  \>>> re.findall(r'a*?', 'aaabb') ['', '', '', '', '', ''] python3.6 老师为啥会这样。。。你的什么版版本

  作者回复: 我用的是Python 3.7

  2020-06-22

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/5f/dc/2a8440ed.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  ꯭小꯭凯꯭![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/d57fddfa5b2c2ce3071e92304da8af62.png)

  r'".*?"|[a-zA-Z]+'

  作者回复: 单词一般用\w+来表示

  2020-06-22

  **2

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_zy1991

  贪婪模式回溯的是测试字符串，而非贪婪模式回溯的是正则表达式，是这么理解吗？

  作者回复: 正则引擎有两种，DFA和NFA，我们面对的一般是 NFA。 DFA 从匹配文本入手，从左到右，每个字符不会匹配两次，时间复杂度是线性的，更加稳定，所以通常情况下，它的速度更快，但支持的特性很少、功能受限，不支持捕获组、各种引用等等；而 NFA 则是从正则表达式入手，不断读入字符，尝试是否匹配当前正则，不匹配则吐出字符重新尝试，通常它的速度比较不稳定，有时候很好，有时候不怎么好，好不好取决于你写的正则表达式，最优时间复杂度为线性，最差情况为指数级的。但 NFA 支持更多的特性，因而绝大多数编程场景下（包括 Java 、.NET、Perl、Python、Ruby、PHP等）。 所以文中讲的贪婪和非贪婪都是从正则表达式入手。

  2020-06-21

  **2

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/132-1662309056850-525.jpeg)

  Geek_fbbf2c

  运行环境：Win10,，Python 3.8，Pycharm， 代码： p= re.compile(r'a*?') print (p.match('aaabb')) 结果：<re.Match object; span=(0, 0), match=''> 什么都没有找到？哪位老师能给解释一下？先谢谢！

  作者回复: 这个不是什么都没找到，是匹配上了，0开始，0结束，也就是第一个空字符串。 如果你想更清晰可以用findall

  2020-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/07/2a/27ba7492.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ZL

  ^([符合要求的组成1]|[符合要求的组成2])+$ 这个要怎样改成独占模式

  作者回复: 改成两个加号，第一个是重复一到多次，第二个是独占模式

  2020-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/33/07/8f351609.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  JustDoDT

  尴尬了，没有把题目看全，想要把提到的引号去掉，分两步走了 环境 python3.7.4 第一步： tmp = re.findall(r'\w+|“.+?”', text)  tmp ['we', 'found', '“the little cat”', 'is', 'in', 'the', 'hat', 'we', 'like', '“the little cat”'] 第二步：去掉引号 res = [re.sub('”|“', '',  i) for i in tmp] res ['we', 'found', 'the little cat', 'is', 'in', 'the', 'hat', 'we', 'like', 'the little cat']

  作者回复: 第二步其实可以用普通字符串替换两次也行。 等后面学习了断言，一个正则就可以直接提取出不带信号的了

  2020-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/f9/5f/a0a882a0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  纳川

  [a-z|A-Z]+|“[a-z|A-Z|\s]+”

  作者回复: 思路没问题，注意中括号里面不需要加管道符，另外单词一般用\w

  2020-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/bd/ec/cc7abf0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  L

  为什么要有独占模式？ 比如regex = “xy{1,3}?z”  直接使用regex = “xy{3}z” 不好吗？独占模式的意义在哪里？

  作者回复: 你发的这个是非贪婪模式，独占模式是加号。 独占模式确实平时用的不多，java等语言里面支持。 独占模式意义在于不匹配时不回溯，直接停止，具体的有没有用得看工作场景，联系你看看阿里技术上那篇正则的文章，看一下作者如何用独占模式解决问题的

  2020-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/33/07/8f351609.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  JustDoDT

  我python 3.6.2 运行下面的代码和老式结果不一样啊 In [5]: re.findall(r'a*?', 'aaabb') Out[5]: ['', '', '', '', '', '']

  作者回复: 我用的Python3.7

  2020-06-18

  **5

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056850-528.jpeg)

  🐒🐱🐭🐮🐯🐰🐶

  之前发现并没有理解题目的意思：重新写一遍： js :'we found "the little cat" is in the hat, we like "the little cat"'.match(/".+?"|\w+/g) ["we", "found", ""the little cat"", "is", "in", "the", "hat", "we", "like", ""the little cat""]

  作者回复: 赞👍🏻

  2020-06-18

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056850-529.jpeg)

  我母鸡啊！

  js  \w+|“.+?”

  作者回复: 对的

  2020-06-17

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056850-530.jpeg)

  宁悦

  我写的就能匹配文中的例子，看了看其他同学的答案，学到了。 ".*?"|\w{2,6}

  作者回复: 多看看其它同学的，看看哪个答案更好。 \w{2,6} 这个，也不一定啊，单词长度一定是2到6个么？

  2020-06-17

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056850-528.jpeg)

  🐒🐱🐭🐮🐯🐰🐶

  js    'we found "the little cat" is in the hat, we like "the little cat"'.match(/\"([\w\s]*)*\"/) 打印结果：["the little cat","the little cat"]

  作者回复: 为什么要用两个星号呢，一个不行么？题目是想找出所有单词，包括那些不在引号中的，如果想不出来可以参考下其它同学的答案哈

  2020-06-17

  **2

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056850-531.jpeg)

  Juntíng

  练习题： let str = 'we found "the little cat" is in the hat, we like "the little cat"'; let reg = /".+?"|[^\s,]+/gm; console.log(str.match(reg)); JS 语言下，'aaabb'.match(/a*?/gm) 匹配结果是：["", "", "", "", "", ""]  这里产生结果是什么原因导致的呢？

  作者回复: 不同的正则实现会有一些小差异，这个问题Python2和Python3也不一样。不要过于纠结，一般情况下，也不建议直接用 a*?，太容易出问题，还有就是写完了之后一定要测试好。

  2020-06-17

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erbY9UsqHZhhVoI69yXNibBBg0TRdUVsKLMg2UZ1R3NJxXdMicqceI5yhdKZ5Ad6CJYO0XpFHlJzIYQ/132)

  饭团

  老师，感觉那个(把z吐出来)是不是应该是把y吐出来！因为这个时候正则是在匹配第三个y

  作者回复: 吐出的是文本，不是匹配的正则，后面的原理篇会进一步讲解。

  2020-06-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  梅花五瓣

  我可以理解为独占模式是匹配量词{m,n}中的n个a吗？ a{1,2}+b匹配aaab 结果匹配了后面2个a，是它先匹配了前面两个a发现第三个a和正则中的b不匹配，然后又从第二个a开始匹配两个a发现第四个b和正则表达式中的b匹配得出结果，是这样子的一个流程吗？

  作者回复: 不是，独占类似于贪婪，意思是尽量多地去匹配，不是非得匹配n次。 比如 a{1,3}+b 去匹配 aab，a期望是1到3个，实际只有两个，匹配到两个后，接着看加号后面的正则，没有回溯。

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/1d/17/94e4c63a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  咩啊

  “.+?”|\w+ 我还以为要把筛选出来后的双引号也要去掉，小白想到头晕o(╥﹏╥)o

  作者回复: 双引号也可以去掉，等学了后面的断言你就有答案了

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/97/75/d73a7828.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  c1rew

  \w+|“.+?”    (10 matches, 74 steps) https://regex101.com/r/eLihse/1/ \w+|“(.+?)”    (10 matches, 104 steps) 只要引号内的the little cat，加括号分组提取   https://regex101.com/r/eLihse/2 对比下留言区内的答案 \w+|“[^”]+”      (10matches, 48steps) 优秀啊

  作者回复: 👍🏻，多看看他人的答案，总结一下写的方式，套路其实就那些

  2020-06-16

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056850-535.jpeg)

  m

  /\w+|“.+?”/g 虽然不知道自己写的什么，但是好像匹配上了... the little cat 需要看成一个单词，但是匹配上的有“”

  作者回复: 思路是对的，正则其实不难。等后面学了断言，你就知道如何去了引号

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/31/7a/f8b0a282.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Januarius

  import regex regex.findall(r'''\w+|“[^”]+''','we found “the little cat” is in the hat, we like “the little cat”')

  作者回复: 👍🏻

  2020-06-16

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLaoiaerNMy7eh7wscKYHoqbohHhFgeia8kY2nkm65dbnwVIpFWibDaiaGlmYcZGKVPS3iaSIVDEdqEI9g/132)

  明明不平凡

  [^\“\s]+|“.+?” 文本中有可能出现非\w的情况吧？

  作者回复: 思路清晰，思考全面👍🏻

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/1d/15/8ad4e24a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yaomon

  \w+|“.+?”

  作者回复: 思路没问题

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/a4/39/6a5cd1d8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  sotey

  "[a-z\s]+"|[a-z]+

  作者回复: 思路是正确的，单词一般用\w来表示，你这个忽略了大写的情况

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ba/01/5ce8ce0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Leoorz

  之前一直对 贪婪和非贪婪模式马马虎虎，至今才知道居然还有独占模式....收获很大 思考题： \w+|“.*?” https://regex101.com/r/PnzZ4k/48 后面部分的 “.*?”主要是想练习一下 非贪婪模式的  最小限度匹配引号中的单词

  作者回复: 没问题，👍🏻

  2020-06-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/e9/6c/00668d9d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  爱乐之城

  请问老师，回溯和重复判断那个例子，回溯是吐出之前已匹配的最后一个字符，这和重复判断有什么关系呢？

  作者回复: 回溯之后，吐出的字符就要再次判断，判断了多次

  2020-06-16

  **2

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056843-458.jpeg)

  飞

  老师有地方没理解，在阿里案例中，正则中有个加号（+），独占模式匹配失败就会停止匹配，为何您说这样会导致大量回溯呢？

  作者回复: 这里指的那个加号是表示次数的，重复一到多次，不是独占模式的加号。 加号在量词后面才表示独占模式

  2020-06-15

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/af/32/74465c5e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Karl

  老师，请问那个“空字符串”怎么理解？就拿那个用a*? 测试 aaabb 的例子来说，明明 aaabb 就是一个字母挨一个字母，为什么还要加进“空字符串”这个实体呢？

  作者回复: 因为星号是0到多次，如果非贪婪，就是优先匹配0次，也就是空字符串了。建议多看看文章中讲的内容，多看几遍就可以理解了

  2020-06-15

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/132-1662309056851-543.jpeg)

  张谋

  “.+?”|\w+

  作者回复: 思路没问题

  2020-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/46/19/827d1ce0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张和祥

  \w+|“(.+?)” 另外想问一下，+和?可以有多个意思，如果放在一起使用，改怎么区分分别代表什么？ 比如说xy{1,3}+z应该可以替换成xy++z。

  作者回复: 看它所处的位置，是不是在量词后面。 不在量词后面的表示次数

  2020-06-15

  **

  **

- ![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,m_fill,h_34,w_34-1662309056843-455.jpeg)

  BillionY

  如果你用 a{1,3}+ab 去匹配 aaab 字符串，a{1,3}+ 会把前面三个 a 都用掉，并且不会回溯，这样字符串中内容只剩下 b 了，导致正则中加号后面的 a 匹配不到符合要求的内容，匹配失败。如果是贪婪模式 a{1,3} 或非贪婪模式 a{1,3}? 都可以匹配上。 --- 我想知道, 如果使用: r = regex.findall(r'a{1,3}+b', 'aaab')  这种独占模式, 效率是否高于前两种?

  作者回复: 你这个例子贪婪和独占模式效率一样，你可以在前面发的测试链接网站上试一试，看具体尝试了多少次，自己在想一想原因。

  2020-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/a8/df/f454e4b7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  轻北

  老师，如果用\w+|“.+?” 那the little cat两边的双引号怎么去掉呢

  作者回复: 后面会讲，分组那一讲里面会有

  2020-06-15

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/bb/a8/5e1b5726.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ♠️K

  （\w+）+？

  作者回复: 可以测试一下，看看能不能正常工作。

  2020-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/eb/09/ba5f0135.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Chao

  \w+|“.+?”

  作者回复: 思路是对的

  2020-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/07/8c/0d886dcc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Shopee内推码：NTAGxyl

  老师您好，请问python的正则引擎是不是和js的有不同，我尝试了非贪婪模式，您的代码，在js中虽然也报错了，但是错误的内容表达的意思似乎不同。如您文章中那一处说：『报错显示，加号（+）被认为是重复次数的元字符了。』，而我在js下的报错信息是：『Uncaught SyntaxError: Invalid regular expression: /xy{1,3}+z/: Nothing to repeat at <anonymous>:1:1』。 请问不同平台下的引擎是否有差异，这部分日后会有解释吗。谢谢老师。

  作者回复: js这个报错说没有东西可以重复，也是把加号认为是表示一到多次的元字符了，也就是说js也不支持这个。

  2020-06-15

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f7/17/f94e987f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](02%E4%B8%A8%E9%87%8F%E8%AF%8D%E4%B8%8E%E8%B4%AA%E5%A9%AA%EF%BC%9A%E5%B0%8F%E5%B0%8F%E7%9A%84%E6%AD%A3%E5%88%99%EF%BC%8C%E4%B9%9F%E5%8F%AF%E8%83%BD%E6%8A%8ACPU%E6%8B%96%E5%9E%AE%EF%BC%81.resource/resize,w_14.png)

  Shawn.C

  要引号： (\w+)|“([\w\s]+)” 不要引号 \w+|“.+?”

  作者回复: 要引号和不要引号写反了吧，你这个要借助替换来去引号，如果使用断言可以直接提取的时候就把引号去了，后面会讲解断言

  2020-06-15
