# Mysql中数据类型括号中的数字代表的含义
[原文地址链接](https://www.cnblogs.com/loren-Yang/p/7512258.html)

相信大家不管是看别人的代码，还是自己的代码总会在定义表的数据类型时，会需要在数据类型后面加一个括号，里面写一个参数，例如int（3），smallint（5），char（5）等，但是括号里面的数字到底是什么意思呢？我以前也是只会用，但是感觉自己是在瞎用，根本没有注意到重点，今天写个博客记录下重点。

## 字符和字节的概念。

- **字节（Byte）是一种计量单位，表示数据量多少**，它是计算机信息技术用于计量存储容量的一种计量单位。

-  **字符是指计算机中使用的文字和符号**，比如`1、2、3、A、B、C、~！·#￥%……—*（）——+、，中，国`等等。

 字符和字节不存在绝对的关系，只是在不同的编码格式里，对应的比值不一样。比如：

**_1UTF-8编码中，一个英文字符等于一个字节，一个中文（含繁体）字符等于三个字节。

**_2._**Unicode编码中，一个英文等于两个字节，一个中文（含繁体）字符等于两个字节。

符号：英文标点占一个字节，中文标点占两个字节。举例：英文句号“.”占1个字节的大小，中文句号“。”占2个字节的大小。

**_3._**UTF-16编码中，一个英文字母字符或一个汉字字符存储都需要2个字节（Unicode扩展区的一些汉字存储需要4个字节）。

**_4._**UTF-32编码中，世界上任何字符的存储都需要4个字节。

所有你看见的单个字：a,啊，都叫字符。

**char和varchar括号中的数字含义。**

**char**的列长度是固定的,char的长度可选范围在0-255字符之间。也就是char最大能存储255个字符.

**varchar**的列长度是可变的,在mysql5.0.3之前varchar的长度范围为0-255字符，mysql5.0.3之后varchar的长度范围为0-65535个字节.

CHAR(M)定义的列的长度为固定的，M取值可以为0-255之间，当保存CHAR值时，在它们的右边填充空格以达到指定的长度。当检索到CHAR值时，尾部的空格被删除掉。在存储或检索过程中不进行大小写转换。CHAR存储定长数据很方便，CHAR字段上的索引效率级高，比如定义char(10)，那么不论你存储的数据是否达到了10个字符，都要占去10个字符的空间,不足的自动用空格填充。

VARCHAR(M)定义的列的长度为可变长字符串，VARCHAR的最大有效长度由最大行大小和使用的字符集确定。整体最大长度是65,532字节。VARCHAR值保存时不进行填充。当值保存和检索时尾部的空格仍保留，符合标准SQL。varchar存储变长数据，但存储效率没有CHAR高。如果一个字段可能的值是不固定长度的，我们只知道它不可能超过10个字符，把它定义为 VARCHAR(10)是最合算的。VARCHAR类型的实际长度是它的值的实际长度+1。为什么"+1"呢？这一个字节用于保存实际使用了多大的长度。从空间上考虑，用varchar合适；从效率上考虑，用char合适，关键是根据实际情况找到权衡点。

简单来说：varchar(m)里面表示的是长度，例如：varchar（5）表示最大可存储5个中文或5个英文字母。 

**int smallint等数据类型括号中的数字含义。**

| 

类型 

 | 大小 | 范围（有符号） | 范围（无符号） | 用途 |
| TINYINT | 1字节 | （-128，127） | （0，255） | 小整数值 |
| SMALLINT | 2字节 | （-32 768,32 767） | （0，65535） | 大整数值 |
| MEDIUMINT | 3字节 | （-8 388 608，8 388 607） | 

(0，16 777 215)

 | 大整数值 |
| INT | 4字节 | 

(-2 147 483 648，2 147 483 647) (0，4 294 967 295) 

 | 

(0，4 294 967 295) 

 | 大整数值 |
| BIGINT | 8字节 | 

(-9 233 372 036 854 775 808，9 223 372 036 854 775 807)

 | 

(0，18 446 744 073 709 551 615)

 | 极大整数值 |

这些类型，是定长的，其容量是不会随着后面的数字而变化的，比如int(11)和int(8)，都是一样的占4字节。tinyint(1)和tinyint(10)也都占用一个字节。

那么后面的11和8，有啥用呢。

数据类型（m）中的m不是表示的数据长度，而是表示数据在显示时显示的最小长度。tinyint(1) 这里的1表示的是 最短显示一个字符。tinyint(2) 这里的2表示的是 最短显示两个字符。

当字符长度（m）超过对应数据类型的最大表示范围时，相当于啥都没发生；

当字符长度（m）小于对应数据类型的表示范围时，就需要指定拿某个字符来填充，比如zerofill（表示用0填充），

设置tinyint(2) zerofill 你插入1时他会显示01；设置tinyint(4) zerofill 你插入1时他会显示0001。

**即使你建表时，不指定括号数字，mysql会自动分配长度：int(11)、tinyint(4)、smallint(6)、mediumint(9)、bigint(20)。**

**推荐两篇博客：**

[**实际验证varchar和char括号中数字表示的是字符**](http://blog.csdn.net/zyz511919766/article/details/51682407)

[**关于int smallint中括号中数字表示的是显示宽度**](http://www.cnblogs.com/stringzero/p/5707467.html)