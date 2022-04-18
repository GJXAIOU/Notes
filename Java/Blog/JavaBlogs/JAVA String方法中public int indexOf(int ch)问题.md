# JAVA String方法中public int indexOf(int ch)问题

在 JAVA 中返回一个字符在字符串的位置首次出现的位置时候,String 给我们提供几个有效的 API。


```java
//返回指定字符在此字符串中第一次出现处的索引。
 int indexOf(int ch) 

//返回在此字符串中第一次出现指定字符处的索引，从指定的索引开始搜索。     
 int indexOf(int ch, int fromIndex) 
 
 //返回指定子字符串在此字符串中第一次出现处的索引。     
 int indexOf(String str) 
 
 //返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始。   
 int indexOf(String str, int fromIndex) 
      
```

举个例子：
```java
class Demo1{
	public static void main(String[] args){
		System.out.println("hello world".indexOf('l'));
		//返回值为2
		System.out.println("hello world".indexOf('o',5));
		//返回值为7
	}
}
```


我们总是很习以为常的取用这个函数，传入一个字符去寻找这个字符首次出现的位置，但是我们仔细的看下 JDK 的函数声明 public int indexOf(int ch)，请注意这里的函数的局部参数是数据类型是 int，而不是我们认为是 char，在 JAVA 中 int 类型定义为 4 个字节，而 char 类型定义为 2 个字节，虽然我们可以将 char 自动转换为 int，但是 JDK 为什么不直接声明为 public int indexOf(char ch)，这就是我们今天要讨论的问题。

  首先 JAVA 使用的 Unicode 编码长度是是 4 个字节，也就是说一个 int 大小为是可以容纳一个 Unicode 的编码的长度的。Unicode 的编码中第一字节称为组，第二字节称为面，第三字节称为行，第四字节称为点。第 0 组第 0 面里的字符可以只用 2 个字节表示，且涵盖了绝大部分的常用字。为了方便称呼，Unicode 给它了一个名称——**基本多文种平面**（BMP  Basic Multilingual Plane）。基本多文种平面值域和上域都是 0 到 FFFF，共计 65535 个码点。并且 ASCII 中有的字符 Unicode 中都有，并且对应相同的编码数字，并不是我们简单的认为 Unicode 编码一个 char 就可以存下数据。

我们看下 String 的 indexOf(int ch)的源代码:

```java
public int indexOf(int ch, int fromIndex) {
        final int max = value.length;
        if (fromIndex < 0) {
            fromIndex = 0;
        } else if (fromIndex >= max) {
            // Note: fromIndex might be near -1>>>1.
            return -1;
        }
 
        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
            // handle most cases here (ch is a BMP code point or a
            // negative value (invalid code point))
            final char[] value = this.value;
            for (int i = fromIndex; i < max; i++) {
                if (value[i] == ch) {
                    return i;
                }
            }
            return -1;
        } else {
            return indexOfSupplementary(ch, fromIndex);
        }
    }
```

@param   ch character (Unicode code point)  参数 ch 是一个 Unicode 的代码点

什么是代码点？**代码点 Code Point：与一个Unicode编码表中的某个字符对应的代码值**。Java 中用 char 来表示 Unicode 字符，由于刚开始 Unicode 最多使用 16bit 表示。因此 char 能够表示全部的 Unicode 字符。**后来由于Unicode4.0规定Unicode支持的字符远远超过65536个字符。因此char现在不能表示所有的unicode字符。仅仅能表示0x000000到0x00FFFF(00 代表的就是拉丁文及其符号)之间的字符。也就是说，char不能表示增补字符**。

Java 中用 int 表示所有 Unicode 代码点。int 的 21 个低位（最低有效位）用于表示 Unicode 代码点，并且 11 个高位（最高有效位）必须为零。也就是说，int 能表示出 char 不能表示的增补字符。我们还可以看到,indexOf()有一个 if{}else{}语句当超过 Unicode 的代码补充范围时候,就会调用 indexOfSupplementartary()方法。他是处理超过范围的问题的。这里其实我们就可以记住**indexOf(int ch)其实是传入的Unicode的代码点，不是传入的真正的字符，而且Java中的代码点是用32为数据表示的,因次是用int而不是char**