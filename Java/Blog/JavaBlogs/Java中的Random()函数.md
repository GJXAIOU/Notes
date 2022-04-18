# JAVA中的Random()函数


Java 中存在着两种 Random 函数：

## 一、java.lang.Math.Random;

　　调用这个 Math.Random()函数能够返回**带正号的double值，该值大于等于0.0且小于1.0，即取值范围是[0.0,1.0)的左闭右开区间，返回值是一个伪随机选择的数，在该范围内（近似）均匀分布**。例子如下：
```java
package IO;
import java.util.Random;

public class TestRandom {
    public static void main(String[] args) {
        // 结果是个double类型的值，区间为[0.0,1.0）
        System.out.println("Math.random()=" + Math.random());
        // 注意不要写成(int)Math.random()*3，这个肯定为0，因为先执行了强制转换
        int num = (int) (Math.random() * 3); 
        System.out.println("num=" + num);
    }
}
// output: 每次结果都不不一致的
// Math.random()=0.8331445049622747
// num=1
```

## 二、java.util.Random
在 Java 的 API 帮助文档中，总结了一下对这个 Random()函数功能的描述：
1、java.util.Random 类中实现的随机算法是伪随机，也就是**有规则**的随机，所谓有规则的就是在给定种(seed)的区间内随机生成数字；

2、相同种子数的 Random 对象，相同次数生成的随机数字是完全相同的；
```java
// 案例2 :对于种子相同的Random对象，生成的随机数序列是一样的。
// 使用种子为10的Random对象生成[0,20)内随机整数序列
Random ran1 = new Random(10);
System.out.println("序列1: \n");
for (int i = 0; i < 8; i++) {
    System.out.print(ran1.nextInt(20) + " ");
}

Random ran2 = new Random(10);
System.out.println("\n 序列2:\n");
for (int i = 0; i < 8; i++) {
    System.out.print(ran2.nextInt(20) + " ");
}
// output:
//序列1: 
//13 0 13 10 6 16 17 8 
//序列2:
//13 0 13 10 6 16 17 8

```
3、Random 类中各方法生成的随机数字都是均匀分布的，也就是说区间内部的数字生成的几率均等；

下面 Random()的两种构造方法：
- Random()：创建一个新的随机数生成器。
- Random(long seed)：使用单个 long 种子创建一个新的随机数生成器。

我们可以在构造 Random 对象的时候指定种子（这里指定种子有何作用，请接着往下看），如：`Random r1 = new Random(20);`或者**默认当前系统时间的毫秒数**作为种子数:`Random r1 = new Random();`
```java
 // 案例3
        // 在没带参数构造函数生成的Random对象的种子缺省是当前系统时间的毫秒数。
        Random r3 = new Random();
        System.out.println("\n 序列3: \n");
        for (int i = 0; i < 10; i++) {
            System.out.print(r3.nextInt(10)+" ");
        }
```

需要说明的是：你在创建一个 Random 对象的时候可以给定任意一个合法的种子数，**种子数只是随机算法的起源数字，和生成的随机数的区间没有任何关系**。如下面的 Java 代码：
```java
Random rand =new Random(25); 
int i;
i=rand.nextInt(100);
```
初始化时 25 并没有起直接作用（注意：不是没有起作用）,`rand.nextInt(100);`中的 100 是随机数的上限,产生的随机数为 0-100 的整数,不包括 100。

生成不重复的随机数 如下例：
```java
import java.util.ArrayList;
import java.util.Random;

public class TestRandom {
    public static void main(String[] args) {
        // 另外，直接使用Random无法避免生成重复的数字，如果需要生成不重复的随机数序列，需要借助数组和集合类
        ArrayList list=new TestRandom().getDiffNO(10);
        System.out.println();
        System.out.println("产生的n个不同的随机数："+list);
    }
    
    /**
     * 生成n个不同的随机数，且随机数区间为[0,10)
     */
    public ArrayList getDiffNO(int n){
        // 生成 [0-n) 个不重复的随机数
        // list 用来保存这些随机数
        ArrayList list = new ArrayList();
        Random rand = new Random();
        boolean[] bool = new boolean[n];
        int num = 0;
        for (int i = 0; i < n; i++) {
            do {
                // 如果产生的数相同继续循环
                num = rand.nextInt(n);
            } while (bool[num]);
            bool[num] = true;
            list.add(num);
        }
        return list;
    }
    
    
}
```

备注：下面是 Java.util.Random()方法摘要：

1.  protected int next(int bits)：生成下一个伪随机数。
2.  boolean nextBoolean()：返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的 boolean 值。
3.  void nextBytes(byte[] bytes)：生成随机字节并将其置于用户提供的 byte 数组中。
4.  double nextDouble()：返回下一个伪随机数，它是取自此随机数生成器序列的、在 0.0 和 1.0 之间均匀分布的 double 值。
5.  float nextFloat()：返回下一个伪随机数，它是取自此随机数生成器序列的、在 0.0 和 1.0 之间均匀分布 float 值。
6.  double nextGaussian()：返回下一个伪随机数，它是取自此随机数生成器序列的、呈高斯（“正态”）分布的 double 值，其平均值是 0.0 标准差是 1.0。
7.  int nextInt()：返回下一个伪随机数，它是此随机数生成器的序列中均匀分布的 int 值。
8.  int nextInt(int n)：返回一个伪随机数，它是取自此随机数生成器序列的、在 0（包括和指定值和 n（不包括）之间均匀分布的 int 值。
9.  long nextLong()：返回下一个伪随机数，它是取自此随机数生成器序列的均匀分布的 long 值。
10.  void setSeed(long seed)：使用单个 long 种子设置此随机数生成器的种子。

下面给几个例子：

1.  生成[0,1.0)区间的小数：double d1 = r.nextDouble();
2.  生成[0,5.0)区间的小数：double d2 = r.nextDouble() * 5;
3.  生成[1,2.5)区间的小数：double d3 = r.nextDouble() * 1.5 + 1;
4.  生成-231 到 231-1 之间的整数：int n = r.nextInt();
5.  生成[0,10)区间的整数：
`int n2 = r.nextInt(10);//方法一`
`n2 = Math.abs(r.nextInt() % 10);//方法二`