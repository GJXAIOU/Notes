# 一个Java对象到底占用多大内存

[原文链接](https://mp.weixin.qq.com/s?__biz=MzIyNzc1ODQ0MQ==&mid=2247484257&idx=1&sn=0bbf06e0ebf5eaafdf0037be03c718d4&chksm=e85d1b67df2a92719d76c1054eaa57883d72a2333d87e9c139b9d985dacce07dad48bf226ac5&scene=0&xtrack=1&key=bed3aaf93eb4dcf3cd77047548cee06735f87ad7bddc3bd156465d031d33b1d435ad27c53a06ad12a961ae6daa2fa6a51d9cdff558c8082b0fca5cf6e14a3bdd17d7dd8b1fc507c6780e6265211609b3&ascene=1&uin=MjY5MDU3MTgyOA%3D%3D&devicetype=Windows+10&version=62060834&lang=zh_CN&pass_ticket=7SrNtMIkbeCWB%2F7LyH9yAqT%2Bc9q9PgWXJoYSePg5XMvunVQe2cdH7Serx6alH8P9)

最近在调研MAT和VisualVM源码实现，遇到一个可疑问题，两者计算出来的对象大小不一致，才有了这样疑惑。

**一个Java对象到底占用多大内存？**


为了复现这个问题，准备了4个最简单类：
```java
class AAAAA {}

class BBBBB {   
  int a = 1;
}

class CCCCC {    
long a = 1L;
}

class DDDDD {    
String s = "hello";
}
```


当然了，再来个主函数：
```java
final  List<AAAAA> aaa = new  ArrayList<>(100000);
final  List<BBBBB> bbb = new  ArrayList<>(100000);
final  List<CCCCC> ccc = new  ArrayList<>(100000);
final  List<DDDDD> ddd = new  ArrayList<>(100000);

for (int i = 0; i < 100000; i++) {
  aaa.add(new AAAAA());
  bbb.add(new BBBBB());
  ccc.add(new CCCCC());
  ddd.add(new DDDDD());
}

```
本地的执行环境是64位的JDK8，且使用默认的启动参数，运行之后通过 `jmap-dump`命令生成dump文件，分别用MAT和VisualVM打开。

### MAT

![640]($resource/640.jpg)

通过MAT打开，可以发现ABD对象大小都是16字节，而C对象大小为24字节

### VisualVM

![641]($resource/641.jpg)

通过Vis打开，可以发现其显示的大小和MAT有蛮大的差别。



### 好奇怪，哪个是对的？

要回答这个问题，首先得清楚的知道JVM中对象的内存布局。

在Hotspot中，一个对象包含3个部分：对象头、实例数据和对齐填充。

#### 对象头

这里不讲对象头是个什么东西，感兴趣的同学可以看我的其它文章。对象头的大小一般和系统的位数有关，也和启动参数 `UseCompressedOops`有关：

*   32位系统，占用 8 字节

*   64位系统，开启 `UseCompressedOops`时，占用 12 字节，否则是16字节

#### 实例数据

原生类型的内存占用情况如下：

*   boolean 1

*   byte 1

*   short 2

*   char 2

*   int 4

*   float 4

*   long 8

*   double 8

引用类型的内存占用和系统位数以及启动参数 `UseCompressedOops`有关

*   32位系统占4字节

*   64位系统，开启 `UseCompressedOops`时，占用4字节，否则是8字节

#### 对齐填充

在Hotspot中，为了更加容易的管理内存，一般会使用8字节进行对齐。

意思是每次分配的内存大小一定是8的倍数，如果对象头+实例数据的值不是8的倍数，那么会重新计算一个较大值，进行分配。

#### 结果

有了对象各部分的内存占用大小，可以很轻松的计算出ABCD各对象在64位系统，且开启 `UseCompressedOops`参数时的大小。

*   A对象只包含一个对象头，大小占12字节，不是8的倍数，需要4字节进行填充，一共占16字节

*   B对象包含一个对象头和int类型，12+4=16，正好是8的倍数，不需要填充。

*   C对象包含一个对象头和long类型，12+8=20，不是8的倍数，需要4个字节进行填充，占24字节

*   D对象包含一个对象头和引用类型，12+4=16，正好是8的倍数，不需要填充。

可以得出，VisualVM的显示结果有点问题，主要因为以下两点：

*   首先，没有考虑是否开启 `UseCompressedOops`

*   其次，没有考虑内存对齐填充的情况

感兴趣的同学，可以动手实践一下，这样可以加深对象内存布局的理解。

经过这段时间对MAT和VisualVM的源码研究，发现MAT的功能不是强大一点点，建议大家以后尽量使用MAT。
