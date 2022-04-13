## 让我再撸一次HashMap



## 引言

其实我很早以前就想写一篇关于HashMap的面试专题。对于JAVA求职者来说，HashMap可谓是集合类的重中之重，甚至你在复习的时候，其他集合类都不用看，专攻HashMap即可。
然而，鉴于网上大部分的关于HashMap的面试方向文章，烟哥看过后都不是太满意。因此，斗胆尝试也写一篇关于HashMap的面试专题文章!

## 正文

**(1)HashMap的实现原理?**
此题可以组成如下连环炮来问

*   你看过HashMap源码嘛，知道原理嘛?

*   为什么用数组+链表？

*   hash冲突你还知道哪些解决办法？

*   我用LinkedList代替数组结构可以么?

*   既然是可以的,为什么HashMap不用LinkedList,而选用数组?

_你看过HashMap源码嘛，知道原理嘛?_
针对这个问题，嗯，当然是必须看过HashMap源码。至于原理，下面那张图很清楚了:

![](https://mmbiz.qpic.cn/mmbiz_png/SYoYmIOcI5qop4BNzyRA8z6iaGz8vLwwdZqgcQ98PgwuI71aoQWzTH9LreuY78oyMfwkkTh5PrC6j3ibM7AUdfog/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HashMap采用Entry数组来存储key-value对，每一个键值对组成了一个Entry实体，Entry类实际上是一个单向的链表结构，它具有Next指针，可以连接下一个Entry实体。
只是在JDK1.8中，链表长度大于8的时候，链表会转成红黑树！
_为什么用数组+链表？_
数组是用来确定桶的位置，利用元素的key的hash值对数组长度取模得到.
链表是用来解决hash冲突问题，当出现hash值一样的情形，就在数组上的对应位置形成一条链表。

`ps`:这里的hash值并不是指hashcode，而是将hashcode高低十六位异或过的。至于为什么要这么做，继续往下看。

_hash冲突你还知道哪些解决办法？_
比较出名的有四种(1)开放定址法(2)链地址法(3)再哈希法(4)公共溢出区域法

`ps:`大家有兴趣拓展的，自己去搜一下就懂了，这个就不拓展了！
_我用LinkedList代替数组结构可以么?_
这里我稍微说明一下，此题的意思是，源码中是这样的

```
Entry[] table = new Entry[capacity];
```

`ps：`Entry就是一个链表节点。
那我用下面这样表示

```
List<Entry> table = new LinkedList<Entry>();  
```

是否可行?
答案很明显，必须是可以的。
_既然是可以的,为什么HashMap不用LinkedList,而选用数组?_
因为用数组效率最高！
在HashMap中，定位桶的位置是利用元素的key的哈希值对数组长度取模得到。此时，我们已得到桶的位置。显然数组的查找效率比LinkedList大。

_那ArrayList，底层也是数组，查找也快啊，为啥不用ArrayList?_
(烟哥写到这里的时候，不禁觉得自己真有想法，自己把自己问死了，还好我灵机一动想出了答案)
因为采用基本数组结构，扩容机制可以自己定义，HashMap中数组扩容刚好是2的次幂，在做取模运算的效率高。
而ArrayList的扩容机制是1.5倍扩容，那ArrayList为什么是1.5倍扩容这就不在本文说明了。

**(2)HashMap在什么条件下扩容?**
此题可以组成如下连环炮来问

*   HashMap在什么条件下扩容?

*   为什么扩容是2的n次幂?

*   为什么为什么要先高16位异或低16位再取模运算?

_HashMap在什么条件下扩容?_
如果bucket满了(超过load factor*current capacity)，就要resize。
load factor为0.75，为了最大程度避免哈希冲突
current capacity为当前数组大小。

_为什么扩容是2的次幂?_
HashMap为了存取高效，要尽量较少碰撞，就是要尽量把数据分配均匀，每个链表长度大致相同，这个实现就在把数据存到哪个链表中的算法；这个算法实际就是取模，hash%length。
但是，大家都知道这种运算不如位移运算快。
因此，源码中做了优化hash&(length-1)。
也就是说hash%length==hash&(length-1)
那为什么是2的n次方呢？
因为2的n次方实际就是1后面n个0，2的n次方-1，实际就是n个1。
例如长度为8时候，3&(8-1)=3  2&(8-1)=2 ，不同位置上，不碰撞。
而长度为5的时候，3&(5-1)=0  2&(5-1)=0，都在0上，出现碰撞了。
所以，保证容积是2的n次方，是为了保证在做(length-1)的时候，每一位都能&1  ，也就是和1111……1111111进行与运算。

_为什么为什么要先高16位异或低16位再取模运算?_
我先晒一下，jdk1.8里的hash方法。1.7的比较复杂，咱就不看了。

![](https://mmbiz.qpic.cn/mmbiz_png/SYoYmIOcI5qop4BNzyRA8z6iaGz8vLwwdfpmHEQ3W9icdygIOH2zvN9edP2YN5PAXjK3FcsRiaDwhxTlVfZfxTGqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

hashmap这么做，只是为了降低hash冲突的几率。

打个比方，当我们的length为16的时候，哈希码(字符串“abcabcabcabcabc”的key对应的哈希码)对(16-1)与操作，对于多个key生成的hashCode，只要哈希码的后4位为0，不论不论高位怎么变化，最终的结果均为0。
如下图所示

![](https://mmbiz.qpic.cn/mmbiz_jpg/SYoYmIOcI5p2hxQpunvxUZoX63YdX3drZzU8MtMBibXyJXDyafRZeyo0PUwbIXqHhqA4gfny8faswzKIwtdVI2g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而加上高16位异或低16位的“扰动函数”后，结果如下

![](https://mmbiz.qpic.cn/mmbiz_png/SYoYmIOcI5p2hxQpunvxUZoX63YdX3dr7w9B5C1h59515iaF9vraZkVmYyDznkic8mCk5ibUEyzzJliadBYRTMsJiag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到: 扰动函数优化前：1954974080 % 16 = 1954974080 & (16 - 1) = 0 扰动函数优化后：1955003654 % 16 = 1955003654 & (16 - 1) = 6 很显然，减少了碰撞的几率。

**(3)讲讲hashmap的get/put的过程?**
此题可以组成如下连环炮来问

*   知道hashmap中put元素的过程是什么样么?

*   知道hashmap中get元素的过程是什么样么？

*   你还知道哪些hash算法？

*   说说String中hashcode的实现?(此题很多大厂问过)

_知道hashmap中put元素的过程是什么样么?_
对key的hashCode()做hash运算，计算index;
如果没碰撞直接放到bucket里；
如果碰撞了，以链表的形式存在buckets后；
如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树(JDK1.8中的改动)；
如果节点已经存在就替换old value(保证key的唯一性)
如果bucket满了(超过load factor*current capacity)，就要resize。

_知道hashmap中get元素的过程是什么样么?_
对key的hashCode()做hash运算，计算index;
如果在bucket里的第一个节点里直接命中，则直接返回；
如果有冲突，则通过key.equals(k)去查找对应的Entry;

*   若为树，则在树中通过key.equals(k)查找，O(logn)；

*   若为链表，则在链表中通过key.equals(k)查找，O(n)。

_你还知道哪些hash算法？_
先说一下hash算法干嘛的，Hash函数是指把一个大范围映射到一个小范围。把大范围映射到一个小范围的目的往往是为了节省空间，使得数据容易保存。
比较出名的有MurmurHash、MD4、MD5等等

_说说String中hashcode的实现?(此题频率很高)_

```
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

String类中的hashCode计算方法还是比较简单的，就是以31为权，每一位为字符的ASCII值进行运算，用自然溢出来等效取模。

哈希计算公式可以计为**s[0]31^(n-1) + s[1]31^(n-2) + … + s[n-1]**
那为什么以31为质数呢?
主要是因为31是一个奇质数，所以`31*i=32*i-i=(i<<5)-i`，这种位移与减法结合的计算相比一般的运算快很多。

**(4)为什么hashmap的在链表元素数量超过8时改为红黑树?**
此题可以组成如下连环炮来问

*   知道jdk1.8中hashmap改了啥么?

*   为什么在解决hash冲突的时候，不直接用红黑树?而选择先用链表，再转红黑树?

*   我不用红黑树，用二叉查找树可以么?

*   那为什么阀值是8呢?

*   当链表转为红黑树后，什么时候退化为链表?

_知道jdk1.8中hashmap改了啥么?_

*   由**数组+链表**的结构改为**数组+链表+红黑树**。

*   优化了高位运算的hash算法：h^(h>>>16)

*   扩容后，元素要么是在原位置，要么是在原位置再移动2次幂的位置，且链表顺序不变。

最后一条是重点，因为最后一条的变动，hashmap在1.8中，不会在出现死循环问题。

_为什么在解决hash冲突的时候，不直接用红黑树?而选择先用链表，再转红黑树?_
因为红黑树需要进行左旋，右旋，变色这些操作来保持平衡，而单链表不需要。
当元素小于8个当时候，此时做查询操作，链表结构已经能保证查询性能。当元素大于8个的时候，此时需要红黑树来加快查询速度，但是新增节点的效率变慢了。

因此，如果一开始就用红黑树结构，元素太少，新增效率又比较慢，无疑这是浪费性能的。

_我不用红黑树，用二叉查找树可以么?_
可以。但是二叉查找树在特殊情况下会变成一条线性结构（这就跟原来使用链表结构一样了，造成很深的问题），遍历查找会非常慢。

_那为什么阀值是8呢?_
不知道，等jdk作者来回答。
这道题，网上能找到的答案都是扯淡。
我随便贴一个牛客网的答案，如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/SYoYmIOcI5p2hxQpunvxUZoX63YdX3drsVk8THH2ial0ibZ7uR2UauzT18e66KAEOkZXgU81YcBXxvT4xC4EcOAQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看出bug没？交点是6.64？交点分明是4，好么。
log4=2，4/2=2。
jdk作者选择8，一定经过了严格的运算，觉得在长度为8的时候，与其保证链表结构的查找开销，不如转换为红黑树，改为维持其平衡开销。

_当链表转为红黑树后，什么时候退化为链表?_
为6的时候退转为链表。中间有个差值7可以防止链表和树之间频繁的转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。

**(5)HashMap的并发问题?**
此题可以组成如下连环炮来问

*   HashMap在并发编程环境下有什么问题啊?

*   在jdk1.8中还有这些问题么?

*   你一般怎么解决这些问题的？

_HashMap在并发编程环境下有什么问题啊?_

*   (1)多线程扩容，引起的死循环问题

*   (2)多线程put的时候可能导致元素丢失

*   (3)put非null元素后get出来的却是null

_在jdk1.8中还有这些问题么?_
在jdk1.8中，死循环问题已经解决。其他两个问题还是存在。

_你一般怎么解决这些问题的？_
比如ConcurrentHashmap，Hashtable等线程安全等集合类。

**(6)你一般用什么作为HashMap的key?**
此题可以组成如下连环炮来问

*   健可以为Null值么?

*   你一般用什么作为HashMap的key?

*   我用可变类当HashMap的key有什么问题?

*   如果让你实现一个自定义的class作为HashMap的key该如何实现？

_健可以为Null值么?_
必须可以，key为null的时候，hash算法最后的值以0来计算，也就是放在数组的第一个位置。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

_你一般用什么作为HashMap的key?_
一般用Integer、String这种不可变类当HashMap当key，而且String最为常用。

*   (1)因为字符串是不可变的，所以在它创建的时候hashcode就被缓存了，不需要重新计算。这就使得字符串很适合作为Map中的键，字符串的处理速度要快过其它的键对象。这就是HashMap中的键往往都使用字符串。

*   (2)因为获取对象的时候要用到equals()和hashCode()方法，那么键对象正确的重写这两个方法是非常重要的,这些类已经很规范的覆写了hashCode()以及equals()方法。

_我用可变类当HashMap的key有什么问题?_
hashcode可能发生改变，导致put进去的值，无法get出，如下所示

```
HashMap<List<String>, Object> changeMap = new HashMap<>();
List<String> list = new ArrayList<>();
list.add("hello");
Object objectValue = new Object();
changeMap.put(list, objectValue);
System.out.println(changeMap.get(list));
list.add("hello world");//hashcode发生了改变
System.out.println(changeMap.get(list));
```

输出值如下

```
java.lang.Object@74a14482
null
```

_如果让你实现一个自定义的class作为HashMap的key该如何实现？_
此题考察两个知识点

*   重写hashcode和equals方法注意什么?

*   如何设计一个不变类

**针对问题一，记住下面四个原则即可**
(1)两个对象相等，hashcode一定相等
(2)两个对象不等，hashcode不一定不等
(3)hashcode相等，两个对象不一定相等
(4)hashcode不等，两个对象一定不等
**针对问题二，记住如何写一个不可变类**
(1)类添加final修饰符，保证类不被继承。
如果类可以被继承会破坏类的不可变性机制，只要继承类覆盖父类的方法并且继承类可以改变成员变量值，那么一旦子类以父类的形式出现时，不能保证当前类是否可变。

(2)保证所有成员变量必须私有，并且加上final修饰
通过这种方式保证成员变量不可改变。但只做到这一步还不够，因为如果是对象成员变量有可能再外部改变其值。所以第4点弥补这个不足。

(3)不提供改变成员变量的方法，包括setter
避免通过其他接口改变成员变量的值，破坏不可变特性。

(4)通过构造器初始化所有成员，进行深拷贝(deep copy)
如果构造器传入的对象直接赋值给成员变量，还是可以通过对传入对象的修改进而导致改变内部变量的值。例如：

```
public final class ImmutableDemo {  
    private final int[] myArray;  
    public ImmutableDemo(int[] array) {  
        this.myArray = array; // wrong  
    }  
}
```

这种方式不能保证不可变性，myArray和array指向同一块内存地址，用户可以在ImmutableDemo之外通过修改array对象的值来改变myArray内部的值。
为了保证内部的值不被修改，可以采用深度copy来创建一个新内存保存传入的值。正确做法：

```
public final class MyImmutableDemo {  
    private final int[] myArray;  
    public MyImmutableDemo(int[] array) {  
        this.myArray = array.clone();   
    }   
}
```

(5)在getter方法中，不要直接返回对象本身，而是克隆对象，并返回对象的拷贝
这种做法也是防止对象外泄，防止通过getter获得内部可变成员对象后对成员变量直接操作，导致成员变量发生改变。