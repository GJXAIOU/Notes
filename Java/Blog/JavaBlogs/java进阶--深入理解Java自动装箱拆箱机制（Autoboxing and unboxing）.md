# java进阶--深入理解Java自动装箱拆箱机制（Autoboxing and unboxing）
 https://blog.csdn.net/u013309870/article/details/70229983

## **基础类型与对象类型的区别**
在 Java 中一切皆为对象，可唯独基础类型很奇怪，对于从 C 语言过渡过来的同学可能很容易就接受了，但是却并不一定能搞懂这两者之间的关系。 
对于对象类型，是在程序运行时，通过 new 动态在堆上创建的对象，而基础类型，是在编译的时候就将值分配到数据栈中。通过基础类型，变量地址所存储的值恰好为存储的值；而对象类型，你持有的仅仅是引用而并不是真实数据存储的地址。 
基础类型，是将数据分配到了数据栈中，而对象类型，则将对象分配到了对象堆中。据说这也是 java 保留基础类型的原因，因为性能！保存在数据栈中避免了运行时动态分配内存的操作。

另外一点，还需要注意的是，java 中有 8 种基础类型： 
byte\short\int\long\double\float\boolean\char 
这里面并没有 String!!!





## **1.自动装箱与拆箱的定义**

装箱就是自动将基本数据类型转换为包装器类型；拆箱就是 自动将包装器类型转换为基本数据类型。

Java 中的数据类型分为两类：一类是基本数据类型，另一类是引用数据类型。如下图：

![这里写图片描述](https://img-blog.csdn.net/20170418171248279?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzMwOTg3MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由上可知 Java 中的基本数据类型有八种分别是：int（4 字节） byte（1 字节） short（2 字节） long（8 字节） float （4 字节） double（8 字节） char（2 字节） boolean（1byte）基本数据类型不是对象，不能使用对象的方法。将基本数据类型转换为对象就是自动装箱的过程。下面是基本数据类型与封装器类之间对应的关系。

| 简单类型 | 二进制位数 | 封装器类 |
| --- | --- | --- |
| int | 32 | Integer |
| byte | 8 | Byte |
| long | 64 | Long |
| float | 32 | float |
| double | 64 | Double |
| char | 16 | Character |
| boolean | 1 | Boolean |


- 自动装箱，底层其实执行了 Integer a=Integer.valueOf(1);
  - 自动拆箱，底层其实执行了 int b=a.intValue();

## **自动装箱**

先看下面这段代码：

```
public static void main(String[] args) {
        // TODO Auto-generated method stub
        int a=3;
        //定义一个基本数据类型的变量a赋值3
        Integer b=a;
        //b是Integer 类定义的对象，直接用int 类型的a赋值    
        System.out.println(b);
        //打印结果为3
    }
```

上面的代码中 `Integer b=a;` 非常的奇怪，一个对象怎么赋值成了基本数据类型的变量，其实这就是自动装箱的过程，上面程序在执行`Integer b=a;`的时候调用了 Integer.valueOf ( int i ) 方法，`Integer b=a; 这段代码等同于：Integer b=Integer.valueOf ( a ) 。下面是对 Integer.valueOf ( int i ) 方法简化后的源码：

```
public static Integer valueOf(int i) {       
        if (i >= -128 && i <= 127)
            return IntegerCache.cache[i + 127];
            //如果i的值大于-128小于127则返回一个缓冲区中的一个Integer对象
        return new Integer(i);
        //否则返回 new 一个Integer 对象
    }
```

可以看到 Integer.valueOf ( a )其实是返回了一个 Integer 的对象。因此由于自动装箱的存在 Integer b=a 这段代码是没有毛病的。其实更简化的来写可以这样：Integer b=3，同样这段代码等价于：Integer b=Integer.valueOf (3 ) 。

## **自动拆箱**

先看下面的这段代码：

```
public static void main(String[] args) {
        // TODO Auto-generated method stub

        Integer b=new Integer(3);
        //b为Integer的对象
        int a=b;
        //a为一个int的基本数据类型
        System.out.println(a);
        //打印输出3。
    }
```

上面的代码：int a=b,很奇怪，怎么把一个对象赋给了一个基本类型呢？其实 int a=b,这段代码等价于：int a=b.intValue(),来看看 inValue()方法到底是什么，下面是源码：

```
public int intValue() {
        return value;
    }
```

这个方法很简单嘛，就是返回了 value 值，然而 value 又是什么，继续找到了一个代码：

```
public Integer(int value) {
        this.value = value;
    }

```

原来 value，就是定义 Integer b=new Integer(3) ; 赋的值。所以上面的代码其实是这样的：

```
public static void main(String[] args) {
        // TODO Auto-generated method stub

        Integer b=new Integer(3);
        //b为Integer的对象
        int a=b.intValue();
        //其中b.intValue()返回实例化b时构造函数new Integer(3);赋的值3。
        System.out.println(a);
        //打印输出3。
    }
```

## **相关题目**

自动装箱和拆箱已经解决了，看看下面的代码输出什么：

```
public static void main(String[] args) {        
        //1
        Integer a=new Integer(123);
        Integer b=new Integer(123);
        System.out.println(a==b);//输出 false,因为是两个对象

        //2 
        Integer c=123;
        Integer d=123;  
        System.out.println(c==d);//输出 true

        //3
        Integer e=129;
        Integer f=129;
        System.out.println(e==f);//输出 false
        //4
        int g=59;
        Integer h=new Integer(59);
        System.out.println(g==h);//输出 true
    }
```

上面的三段代码：代码 1 输出为 true 还是比较好理解的：如下图：

![这里写图片描述](https://img-blog.csdn.net/20170419095258057?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzMwOTg3MA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

a，b 是栈中对象的引用分别指向堆中两个不同的对象，而 a==b 这条语句就是判断 a , b 在堆中指向的对象是不是同一个，因此 代码 1 输出 false。

然而代码 2 中的输出为什么会是 true 呢？由上面的自动装箱知道

```
Integer c=123;
Integer d=123;  
```

不也是生成了两个对象 c 和 d 吗？ 为什么输出结果却为 true。这个其实跟自动装箱有关，再来看一下自动装箱的源代码：

```
public static Integer valueOf(int i) {       
        if (i >= -128 && i <= 127)
            return IntegerCache.cache[i + 127];
            //如果i的值大于-128小于127则返回一个缓冲区中的一个Integer对象
        return new Integer(i);
        //否则返回 new 一个Integer 对象
    }
```

上面的这段代码中：IntegerCache.cache[i + 127]; 又是什么？下面是简化后的源码：

```
 private static class IntegerCache {

        static final Integer cache[];
        //定义一个Integer类型的数组且数组不可变
        static {  
        //利用静态代码块对数组进行初始化。                     
            cache = new Integer[256];
            int j = -128;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

  //cache[]原来是一个Integer 类型的数组（也可以称为常量池），value 从-128到127，
    public static Integer valueOf(int i) {   
        if (i >=-128 && i <= 127)        
            return IntegerCache.cache[i + (-IntegerCache.low)];
            //如果装箱时值在-128到127之间，之间返回常量池中的已经初始化后的Integer对象。
        return new Integer(i);
        //否则返回一个新的对象。
    }
}
```

由上面的代码很好理解，原来 IntegerCache 类在初始化的时候，生成了一个大小为 256 的 integer 类型的常量池，并且 integer.val 的值从-128~127，当我们运行 Integer c=a ;时，如果 -128<=a<=127 时，不会再生成新的 integer 对象，直接从常量池中找到对应的已经初始化后的对象。当 a<-128||a>127 时会生成一个新的对象。因此不难理解代码 2 为什么会输出 true 了。因为 c 和 d 指向的是同一个对象，并不是生成了两个不同对象。同样不难理解代码 3 为什么会输出 false 。但是代码 4 中明明 g 指向的是栈中的变量，而 h 指向的是堆中的对象，为什么 `g==h` 是 true，这就是自动拆箱，`g==h` 这代码执行时其实是：`g==h.IntValue(）`，而 h.IntValue()=59,所以两边其实是两个 int 在比较。

## 总结

上面讲解了 int 基本类型的自动装箱和拆箱，其实**int byte short long float double char boolean** 这些基本类型的自动装箱和拆箱过程都是差不多的。

简单一句话：**装箱就是自动将基本数据类型转换为包装器类型；拆箱就是 自动将包装器类型转换为基本数据类型。**