简单概括就是：**xml本身是一种格式规范，是一种包含了数据以及数据说明的文本格式规范**。

**举例说明：**

比如，我们要给对方传输一段数据，数据内容是“too young,too simple,sometimes naive”，要将这段话按照属性拆分为三个数据的话，就是，年龄too young，阅历too simple，结果sometimes naive。

我们都知道程序不像人，可以体会字面意思，并自动拆分出数据，因此，我们需要帮助程序做拆分，因此出现了各种各样的数据格式以及拆分方式。

比如，可以是这样的
数据为“too young,too simple,sometimes naive”
然后按照逗号拆分，第一部分为年龄，第二部分为阅历，第三部分为结果。

也可以是这样的
数据为`“too_young**too_simple*sometimes_naive”`
从数据开头开始截取前面十一个字符，去掉`*`号并把下划线替换为空格作为第一部分，再截取接下来的十一个字符同样去掉`*`并替换下划线为空格作为第二部分，最后把剩下的字符同样去`*`号体会空格作为第三部分。

这两种方式都可以用来容纳数据并能够被解析，但是不直观，通用性也不好，而且如果出现超过限定字数的字符串就容纳不了，也可能出现数据本身就下划线字符导致需要做转义。

基于这种情况，出现了xml这种数据格式， 上面的数据用XML表示的话
可以是这样
`<person age="too young" experience="too simple" result="sometimes naive" />`
也可以是这样
```xml
<person>
    <age value="too young" />
    <experience value="too simple" />
    <result value="sometimes naive" />
</person>
```

两种方式都是xml，**都很直观，附带了对数据的说明，并且具备通用的格式规范可以让程序做解析。**

如果用json格式来表示的话，就是下面这样
```text
{
    "age":"too young",
    "experience":"too simple",
    "result":"sometimes naive"
}
```

看出来没，**其实数据都是一样的，不同的只是数据的格式而已**，同样的数据，我用xml格式传给你，你用xml格式解析出三个数据，用json格式传给你，你就用json格式解析出三个数据，还可以我本地保存的是xml格式的数据，我自己先解析出三个数据，然后构造成json格式传给你，你解析json格式，获得三个数据，再自己构造成xml格式保存起来，说白了，**不管是xml还是json，都只是包装数据的不同格式而已，重要的是其中含有的数据，而不是包装的格式**。