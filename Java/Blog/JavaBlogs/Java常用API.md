---
tags: 
- API
style: summer
flag: red
date: '2019-6-30'
---

# Java常用API
@toc

# 一、Object对象

面向对象的核心思想:“找合适的对象，做适合的事情”。

合适的对象：
1. 自己描述类，自己创建对象。
2. sun已经描述了好多常用的类，可以使用这些类创建对象。
    API（Application Program Interface）

 sun定义的那么多类的终极父类是Object。Object描述的是所有类的通用属性与方法。

## （一） toString方法以及重写

```java
public static void main(String[] args){
  Object o = new Object();
  System.out.println(o); //java.lang.Object@de6ced
}
```
toString() 返回对象的描述信息 java.lang.Object@de6ced 类名@哈希码值的十六进制形式。
直接输入一个对象的时候，会调用对象的toString方法。

练习：自定义一个Person类，打印该对象的描述信息，要求描述信息为：姓名 — 年龄

```java
class Person{
	int age;
	String name;

	public Person() {
	}
	
	public Person(int age, String name){
		this.age = age;
		this.name = name;
	}
	
}

public class Demo {
	public static void main(String[] args) {
		Person person = new Person();
		System.out.println(person); //对象的描述信息 study.Person@311d617d
	}
}
```

问题：调用person 的toString方法时，打印出来的信息是`类名+内存地址值`。不符合要求。根据我们之前学的继承，假如父类的指定的功能不能满足要求，那么子类可以复写父类的功能函数。那么该对象再调用toString()方法时，则会调用子类复写的toString方法。

```java
//tostring方法重写
class Person{
	int age;
	String name;

	public Person() {
	}
	
	public Person(int age, String name){
		this.age = age;
		this.name = name;
	}
	
  //可以使用Alt + shift + s 然后generate toString 
	@Override
	public String toString() {
		return "Person [age=" + this.age + ", name=" + this.name + "]";
	}
	
}
```


**编程习惯:** 开发者要对自定义的类重写toString()，对对象做详细的说明



## （二） equals方法

**equals()** 返回的是比较的结果  如果相等返回true，否则false，比较的是对象的内存地址值。
```java
public static void main(String[] args){
  Object o1 = new Object();
  Object o2 = new Object();
  System.out.println(o1.equals(o2)); //false
}

public static void main(String[] args){
  Object o1 = new Object();
  Object o2 = o1;
  System.out.println(o1.equals(o2)); //true
}
```

问题：比较两个人是否是同一个人，根据两个人的名字判断。

**问题：** 如果根据名字去作为判断两个人是否是同一个时，明显p与p1是同一个人，但是程序输入却不是同一个人。不符合我们现实生活的要求。

**解决:** 根据我们学的继承中的函数复写，如果父类的函数不能满足我们目前的要求，那么就可以在子类把该功能复写，达到复合我们的要求。
```java
@Override
public boolean equals(Object arg0) {
	Person person = (Person)arg0; //强制类型转换
	return this.name.equals(person.name);//判断name的属性，使用的string中的equals方法
}

```
**编程习惯：** 开发者要对自定义的类重写equals()，使得比较两个对象的时候比较对象的属性是否相等，而不是内存地址。



## （三） hashCode方法

**hashCode()** 返回该对象的哈希码值： 采用操作系统底层实现的哈希算法。 同一个对象的哈希码值是唯一的。

java规定如果两个对象equals返回true，那么这两个对象的hashCode码必须一致。(使用默认的 equals 方法的时候)
**一般情况下：重写了 equals 方法就得重写 hashCode 方法**
```java
@Override
public boolean equals(Object arg0) {
	Person person = (Person)arg0; //强制类型转换
	return this.name.equals(person.name);//判断name的属性，使用的string中的equals方法
}

@Override
public int hashCode() {
	return this.name.hashCode();
}
```


# 二、String类

String类描述的是文本字符串序列。例如： 留言 、QQ、 写日志。

**创建String类的对象的两种方式：**
1. 使用“ ”直接赋值法
2. new关键字法



## （一）字符串对象的比较

```java
public class Demo {
	public static void main(String[] args) {
		String string1 = "hello";
		String string2 = "hello";
		String string3 = new String("hello");
		String string4 = new String("hello");
		
		System.out.println(string1 == string2); //true
		System.out.println(string1 == string3); //false
		System.out.println(string3 == string4); //false
}
```


String string = “hello” 这个语句会先检查字符串常量池是否存放这个”string1”这个字符串对象，如果没有存在，那么就会在字符串常量池中创建这个字符串对象，如果存在直接返回该字符串的内存地址值。

String string3 = new String(“hello”) 该语句会创建两个对象,首先会先检查字符串常量池中存不存在 `hello` 这个字符串对象，如果不存在就会创建，如果存在就返回内存地址值。创建了出来之后，new String这个语句就会在堆内存中开辟一个字符串对象。总共两个对象。

![string数组]($resource/string%E6%95%B0%E7%BB%84.png)



## （二）获取方法

返回值 |命令 | 含义
---|---|---
int |  length() | 获取字符串的长度
char | charAt(int index) | 获取特定位置的字符 (角标越界)
int  |  indexOf(String str) | 获取特定字符的位置(overload)
int  |  lastIndexOf(int ch)  |   获取最后一个字符的位置
```java
String string = "hello world";
      System.out.println("length : " + string.length());       				  //11
      System.out.println("I 的位置 ：" + string.indexOf("o")); 				 //4
      System.out.println("I的最后位置  ："  + string.lastIndexOf("o"));    // 7
      System.out.println("P的位置 ： " + string.indexOf("P"));				 //-1 不存在下标就是-1
      System.out.println("获取下标为1的元素：" + string.charAt(1));      //e
      System.out.println("获取下表为12的元素：" + string.charAt(12));  // 数组越界java.lang.StringIndexOutOfBoundsException
```


## （三）判断方法

返回值 |命令 | 含义
---|---|---
boolean  | endsWith(String str)  | 是否以指定字符结束
boolean |  isEmpty()  | 是否长度为0 如：“” null V1.6
boolean  | contains(CharSequences) | 是否包含指定序列 应用：搜索
boolean | equals(Object anObject) | 是否相等
boolean |equalsIgnoreCase(String anotherString) |忽略大小写是否相等
```java
String string1 = " ";
String string2 = "";
System.out.println("string1长度：" + string1.length()); //string1的长度：1
System.out.println("string2长度：" + string2.length()); //string2的长度：0
System.out.println("string1是否为空：" + string1.isEmpty()); //false
System.out.println("string2是否为空：" + string2.isEmpty()); //true

string1 = "hello java world";
System.out.println("是否包含 java 字符串：" +string1.contains("java")); //true
System.out.println("abc".equals("abc")); //true

System.out.println(new String("abc") .equals(new String("abc"))); //true
System.out.println(new String("abc") .equals(new String("ABC"))); //false
System.out.println(new String("abc") .equalsIgnoreCase(new String("ABC"))); //true

string1 = "Demo.java";
System.out.println("是不是以 .java 结尾 " + string1.endsWith(".java")); //是不是以 .java 结尾 true
```


## （四）转换方法

方法| 含义
---|---
String(char[] value) | 将字符数组转换为字符串
String(char[] value, int offset, int count) |
Static String valueOf(char[] data)|
static String valueOf(char[] data, int offset, int count) |
char[] toCharArray() | 将字符串转换为字符数组

```java
String str = new String(new char[]{'h', 'e', 'l', 'l', '0'});
System.out.println(str);

char[] chars = str.toCharArray();
for (int i = 0; i < chars.length; i++) {
    System.out.println(chars[i]);
}

byte[] bytes = {97, 98, 99};
String s = new String(bytes);
System.out.println(s);

byte[] strBytes = str.getBytes();
for (int i = 0; i < strBytes.length; i++) {
    System.out.println(strBytes[i]);
 }
```
程序运行结果：
```java
hell0
h
e
l
l
0
abc
104
101
108
108
48
```




## （五）其他方法
 返回值  | 方法| 说明
 ---|---|---
String |replace(char oldChar, char newChar)| 替换
String[] | split(String regex) | 切割
String| substring(int beginIndex)
String| substring(int beginIndex, int endIndex) |截取字串
String | toUpperCase() |转大写
String  | toLowerCase() |转小写
String |  trim() |去除空格



## （六）练习

- 去除字符串两边空格的函数。

```java
public class Demo {

    // 定义一个祛除字符串两边空格的函数
    public static void main(String[] args) {
        String string = Demo.trim("  hell  0  ");
        System.out.println(string.toCharArray());
    }
    public static String trim(String str) {

        // 0、定义求字串需要的起始索引变量
        int start = 0;
        int end = str.length() - 1;
        // 1. for循环遍历字符串对象的每一个字符
        for (int i = 0; i < str.length(); i++) {
            if (str.charAt(i) == ' ') {
                start++;
            } else {
                break;
            }
        }
        System.out.println(start);

        for (; end < str.length() && end >= 0; ) {
            if (str.charAt(end) == ' ') {
                end--;
            } else {
                break;
            }
        }
        System.out.println(end);
        // 2. 求子串
        if (start < end) {
            return str.substring(start, (end + 1));
        } else {
            return "_";
        }
    }
}

```
程序执行结果：
```java
2
8
hell  0
```

- 获取上传文件名 "D:\\20120512\\day12\\Demo1.java"。
```java
public static  String getFileName2( String path ){
    return path.substring( path.lastIndexOf("\\") + 1 );
     }
}
```

- 将字符串对象中存储的字符反序。
```java
// 将字符串对象中存储的字符反序
    public static String reaverseString(String src) {

        // 1. 将字符串转换为字符数组
        char chs[] = src.toCharArray();
        // 2. 循环交换
        for (int start = 0, end = chs.length - 1; start < end; start++, end--) {
            // 3. 数据交换
            char temp = chs[end];
            chs[end] = chs[start];
            chs[start] = temp;
        }
        // 4. 将字符数组转换为字符串
        return new String(chs);
    }
```

- 求一个子串在整串中出现的次数

```java
 public static int getCount(String src, String tag) {
        // 0. 定义索引变量和统计个数的变量
        int index = 0;
        int count = 0;
        // 1. 写循环判断
        while ((index = src.indexOf(tag)) != -1)   // jackjava
        {
            // 2. 求字串
            System.out.println(src);
            src = src.substring(index + tag.length());   // index 4 + 4 = 8
            System.out.print(src.length() + " : " + index + " :  " + tag.length());
            // 3. 累加
            count++;
        }
        return count;
    }
```

# 三、StringBuffer
**StringBuffer** : 由于String是不可变的，所以导致String对象泛滥，在频繁改变字符串对象的应用中，需要使用可变的字符串缓冲区类。

- 特点：
  - 默认缓冲区的容量是16。
  - StringBuffer ： 线程安全的所有的缓冲区操作方法都是同步的。效率很低。

```java
public static void main(String[] args) {
        String string = "";
        for (int i = 0; i < 10; i++) {
            string += 1;
        }
        System.out.println(string);
    }
```



## （一）添加方法
| 方法名 | 说明
---|---|
StringBuffer("jack")  |   在创建对象的时候赋值
append()         |     在缓冲区的尾部添加新的文本对象
insert()          |      在指定的下标位置添加新的文本对象

```java
StringBuffer sb = new StringBuffer("jack");
sb.append(true);
sb.append('a');
// 链式编程
sb.append(97).append(34.0).append(new char[]{
     'o', 'o'
 });   
// 输出缓冲区的中文本数据
System.out.println(sb.toString());          
sb = new

StringBuffer("jack");
// jajavack
sb.insert(2, "java");                       
System.out.println(sb.toString());
```

## （二）查看
| 方法 | 说明
|---|---
toString()  | 返回这个容器的字符串
indexOf(String str)  | 返回第一次出现的指定子字符串在该字符串中的索引。
substring(int start)  |从开始的位置开始截取字符串

```java
public static void main(String[] args) {
    StringBuffer stringBuffer = new StringBuffer("jackc");
  System.out.println(stringBuffer.indexOf("c"));
  System.out.println(stringBuffer.lastIndexOf("c")); }
```



## （三）修改(U)
|方法 | 说明
| ---|---
replace(int start int endString str) | 使用给定 `String`  中的字符替换此序列的子字符串中的字符。该子字符串从指定的 `start`  处开始，一直到索引 `end - 1`  处的字符
setCharAt(int index char ch) | 指定索引位置替换一个字符

```java
public static void main(String[] args) {
        StringBuffer stringBuffer = new StringBuffer("helloworld");
        System.out.println(stringBuffer.replace(2, 6, "javaEE"));
        stringBuffer.setCharAt(8,'Q');
        System.out.println(stringBuffer);
    }
```
程序运行结果：
```java
hejavaEEorld
hejavaEEQrld
```




## （四）删除(D)

方法 | 说明
---|---
delete(int start, int end)|删除指定区域元素，start <= char < end
delete(0, sb.length) | 清空整个缓冲区
deleteCharAt(int index) | 删除某个下标的元素   

```java
public static void main(String[] args) {
        StringBuffer stringBuffer = new StringBuffer("helloworld");
        System.out.println(stringBuffer.delete(2, 5));

        System.out.println(stringBuffer.deleteCharAt(3));
    }
```
程序运行结果：
```java
heworld
hewrld
```


## （五）反序

reverse() 把字符串反序输出。
```java
public static void main(String[] args) {
  String string = "helloworld";
  StringBuffer stringBuffer = new StringBuffer(string);
  System.out.println(stringBuffer.reverse()); }
```
程序运行结果：
`dlrowolleh`



# 四、StringBuilder

StringBuilder 是JDK1.5之后提出的，线程不安全，但是效率要高。用法与StringBuffer类似。

# 五、System

System 可以获取系统的属性。
```java
public static void main(String[] args) {
        // 获取系统属性
        Properties properties = System.getProperties();
        // 输出系统属性
        properties.list(System.out);
        System.out.println("*********************");

        // 获取操作系统名称
        String osName = System.getProperty("os.name");
        System.out.println(osName);
        System.out.println("*********************");

        // 检测操作系统系统支持该软件
        if ("Windows 10".equals(osName)){
            System.out.println("符合");
        }else{
            System.out.println("不符合");
        }
        System.out.println("*********************");

        // 获取path环境变量值
        System.out.println(System.getenv("path"));
    }

```
输出结果：
```java
-- listing properties --
java.runtime.name=Java(TM) SE Runtime Environment
sun.boot.library.path=E:\Program\Java\JDK1.8\jre\bin
java.vm.version=25.221-b11
## 等等等等
*********************
Windows 10
*********************
符合
*********************
C:\Program Files (x86)\Common Files\Oracle\Java\javapath； ## 等等等等
```



# 六、Runtime

Runtime 类主要描述的是应用程序运行的环境。

返回值|方法名 | 说明|
---|---|---|
 无  |exit() | 退出虚拟机|
long| freeMemory() | 获取可用内存数目
无 | gc() | 调用垃圾回收器程序，但是调用该方法不会马上就运行 gc
long | maxMemory() | 获取 JVM 最大内存容量
long | totalMemory() | 获取总内存
Process | exec(String command) | 启动一个字符串命令的进程  

```java
public static void main(String[] args) {
        // 获取应用运行环境的对象
        Runtime runtime = Runtime.getRuntime();

        // 获取可用的内存数
        System.out.println(runtime.freeMemory());

        // 获取JVM试图使用的内存总容量
        System.out.println(runtime.maxMemory());

        // 获取JVM只能使用的总容量
        System.out.println(runtime.totalMemory());

        // 启动程序：
        try {
            Process screentoGif = runtime.exec("D:\\Screentogif\\screentogif.exe");
            Process typora = runtime.exec("typora demo.md");
            
            Thread.sleep(1000*10);
            typora.destroy();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

# 七、Date

Date 类封装的是系统的当前时间.。但是Date已经过时了，sun推荐使用Calendar类。

Calendar: 该类是一个日历的类，封装了年月日时分秒时区。
```java
public static void main(String[] args) {
        Calendar calendar = Calendar.getInstance();
        // 获取年、月、日、时、分、秒
        int year = calendar.get(Calendar.YEAR);
        int month = calendar.get(Calendar.MONTH) + 1;
        int day = calendar.get(Calendar.DAY_OF_MONTH);
        int dayOfWeek = calendar.get(Calendar.DAY_OF_WEEK);

        int hour = calendar.get(Calendar.HOUR_OF_DAY);
        int minute = calendar.get(Calendar.MINUTE);
        int second = calendar.get(Calendar.SECOND);

        System.out.println("year : " + year + " month : " + month + " day : " + day +  "\n" +
                "dayOfWeek : " + dayOfWeek + "\n" +
                "hour : " + hour + " minute : " + minute + " second : " + second);

    }
```
程序运行结果：
```java
year : 2019 month : 8 day : 8
dayOfWeek : 5
hour : 11 minute : 40 second : 42
```

日期格式化类：SimpleDateFormat
```java
public static void main(String[] args) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy年MM日dd日 E a hh时mm分ss秒");
        System.out.println(simpleDateFormat.format(new Date()));
    }
```
程序结果：`2019年08日08日 星期四 上午 11时43分53秒`



# 八、Math

Math：类封装了很多数学的功能。
方法| 说明
---|---
static double ceil(double a)|返回大于等于指定小数的最小整数
static double floor(double a) | 返回小于等于指定小数的最大整数
static long round(double a) | 四舍五入
static double random() | 返回大于等于 0.0 小于 1.0 的小数 1.0 <= x <11.0   

```java
public static void main(String[] args) {
        System.out.println(Math.PI);
        System.out.println(Math.ceil(12.3));
        System.out.println(Math.ceil(12.5));
        System.out.println(Math.ceil(-12.5));

        System.out.println(Math.floor(-15.1));
        System.out.println(Math.floor(15.1));

        System.out.println(Math.round(15.1));
        System.out.println(Math.round(15.5));

        System.out.println(Math.random());
    }
```
程序输出结果：
```java
3.141592653589793
13.0
13.0
-12.0
-16.0
15.0
15
16
0.1833227126599336
```


练习：生成一个随机码
```java
public static void main(String[] args) {
        // 生成一个随机码
        Random random = new Random();
        char[] chars = {'a', 'b', 'c', 'd', 'e', '你', '好', '@'};

        StringBuilder stringBuilder = new StringBuilder("");

        for (int i = 0; i < 4; i++) {
            stringBuilder.append(chars[random.nextInt(chars.length)]);

        }
        System.out.println("随机码：" + stringBuilder.toString());
    }
```


