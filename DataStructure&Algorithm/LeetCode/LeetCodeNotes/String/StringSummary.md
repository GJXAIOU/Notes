# StringSummary

### 常用 API

http://www.51gjie.com/java/596.html

### 常见注意点

- 字符串的合法性判断
- 空字符串返回值是什么？
- 多个解的输出顺序？

==字符串列表转换为 String 数组==

`res.toArray(new String[res.size()]);`

```java
List<String> res = new ArrayList<>();

public String[] permutation(String s) {
    if (s.length() == 0) {
        return res.toArray(new String[res.size()]);
    }
}
```



- **判断字符是否为空格**

    ```java
    for (int i = 0; i < s.length(); i++) {
        char temp = s.charAt(i);
        // 单引号直接使用 == 即可，同时 ' ' 只占一个字符
        if (temp == ' ') {
    
        } 
    }
    ```

    

- 数组转为字符串不使用 `数组名.toString();`，应该使用 `Arrays.toString(数组名)`**也是不可以，还是得用 StringBuilder 的 toString。

- 数字转字符串：`Integer.toString(a);`
- StringBuilder 转 Int，需要首先使用 `.toString()`方法转换为 String，然后使用 `Integer.paseInt(XX)`转换为 Int 类型。
- 字符串要是排序，需要先使用 `XXX.toCharArray()`转换为字符数组，然后使用 `Arrays.sort()` 进行排序。

- 数组转为字符串：`String.valueOf(XXX);`





### String 和 StringBuffer、StringBuilder 的区别是什么 String 为什么是不可变的

#### 可变性：以为存储时的区别

  **String 类中使用 final 关键字字符数组保存字符串，`private final　char　value[]`，所以 String 对象是不可变的**。而StringBuilder 与 StringBuffer 都继承自 AbstractStringBuilder 类，在 AbstractStringBuilder 中也是使用字符数组保存字符串 `char[]value` 但是没有用 final 关键字修饰，所以这两种对象都是可变的。

- String 类的源码

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
```

- StringBuilder 和 StringBuffer 源码

    ```java
     /* @author      Arthur van Hoff
     * @see     java.lang.StringBuilder
     * @see     java.lang.String
     * @since   JDK1.0
     */
     public final class StringBuffer
        extends AbstractStringBuilder
        implements java.io.Serializable, CharSequence
    {
    
        public StringBuffer() {
            super(16);
        }
    ```

- StringBuilder 源码

    ```java
    /* @author      Michael McCloskey
     * @see         java.lang.StringBuffer
     * @see         java.lang.String
     * @since       1.5
     */
    public final class StringBuilder
        extends AbstractStringBuilder
        implements java.io.Serializable, CharSequence
    {
    
        public StringBuilder() {
            super(16);
        }
    
       
        public StringBuilder(int capacity) {
            super(capacity);
        }
    
    ```

    **可以看出其构造函数中都是使用了其父类 AbstractStringBuilder,自身没有定义存储的容器,而是继承了其父类的容器 **，代码见下：可以看出就是 `char[] value`

    ```java
    abstract class AbstractStringBuilder implements Appendable, CharSequence {
        /**
         * The value is used for character storage.
         */
        char[] value;
        /**
         * Creates an AbstractStringBuilder of the specified capacity.
         */
        AbstractStringBuilder(int capacity) {
            value = new char[capacity];
        }
    
    ```

    > 从上面看出,**String的字符是存储在一个被final修饰的char数组(类似于c中的指针常量)中的,而StringBuilder的字符是存储在一个普通的char数组**中的

#### 线程安全性

 String 中的对象是不可变的，也就可以理解为常量，线程安全。AbstractStringBuilder 是 StringBuilder 与 StringBuffer 的公共父类，定义了一些字符串的基本操作，如 expandCapacity、append、insert、indexOf 等公共方法。**StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的**。StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。 

#### 性能　　

**每次对 String 类型进行改变的时候，都会生成一个新的 String 对象，然后将指针指向新的 String 对象**。**StringBuffer 每次都会对 StringBuffer 对象本身进行操作，而不是生成新的对象并改变对象引用**。相同情况下使用 StirngBuilder 相比使用 StringBuffer 仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

 

### 运算时的区别

#### String的运算

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513113503305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoZW5rZWRpbmc5MzUw,size_16,color_FFFFFF,t_70)

这两个相加的操作可以看作这样

```java
final char c1[] = {'第','一','个','字','符','串'};

final char c2[] = {'第','二','个','字','符','串'};

final char c3[] = new char[12];

c3[] =  {'第','一','个','字','符','串','第','二','个','字','符','串'};

c1 = c3

//这段只做理解
```

String在运算的时候都会**创建一个大小合适的char数组[]**,所以当下次再拼接的时候都要进行重新分配.

#### StringBuilder的运算

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513113553460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoZW5rZWRpbmc5MzUw,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513113643860.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NoZW5rZWRpbmc5MzUw,size_16,color_FFFFFF,t_70)
**StringBuilder特征**

- StringBuilder初始化容量是16(无参构造)
- 追加之前会计算一次容量,大于所需容量则会重新创建一个char[]数组,计算规则是 **newCapacity = (value.length << 1) + 2;** 也就是 原来长度*2+2
- StringBuilder在运算的时候每次会计算容量是否足够,如果所需容量不小于自身容量,那么就会重新分配一个自身容量两倍+2的char[].所以省去了很多创建char[]的次数.

### 大白话结论

- String之所以慢是因为,大部分cpu资源都被浪费在分配资源,拷贝资源的部分了,相比StringBuilder有更多的内存消耗.
- StringBuilder快就快在,相比String,他在运算的时候分配内存次数小,所以拷贝次数和内存占用也随之减少.
- 由于GC的机制,即便原来的char[]没有引用了,那么也得等到GC触发的时候才能回收,String运算过多的时候就会产生大量垃圾,消耗内存.

### 建议

- 如果目标字符串需要大量拼接的操作,那么这个时候应当使用StringBuilder.
- 反之,如果目标字符串操作次数极少,或者是常量,那么就直接使用String.



### JVM 优化

从jdk 5开始，Java就对String字符串的`+`操作进行了优化，该操作编译成字节码文件后会被优化为StringBuilder的append操作。但是，**我们不能一味地把String的`+` 操作等同于append操作**。

可以看这篇文章的对反编译字节码文件的分析：
[jdk不同版本对String拼接的优化分析](http://blog.csdn.net/kingszelda/article/details/54846069)

这里只做总结：

1. 字符串拼接从jdk5开始就已经完成了优化，并且没有进行新的优化。
2. 循环内String+常量的话会每次new一个StringBuilder，再调用append方法。
3. 循环外字符串拼接可以直接使用String的+操作，没有必要通过StringBuilder进行append.
4. 有循环体的话，好的做法是在循环外声明StringBuilder对象，在循环内进行手动append。不论循环多少层都只有一个StringBuilder对象。









