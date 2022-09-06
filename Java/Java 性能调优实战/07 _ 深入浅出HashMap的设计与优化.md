# 07 \| 深入浅出HashMap的设计与优化

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/d3/ce/d32b34a63447b3694357ac9173c5ffce.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/52/bc/52d774c8a6b9659ecb2543e3357a58bc.mp3" type="audio/mpeg"></audio>

你好，我是刘超。

在上一讲中我提到过Collection接口，那么在Java容器类中，除了这个接口之外，还定义了一个很重要的Map接口，主要用来存储键值对数据。

HashMap作为我们日常使用最频繁的容器之一，相信你一定不陌生了。今天我们就从HashMap的底层实现讲起，深度了解下它的设计与优化。

## 常用的数据结构

我在05讲分享List集合类的时候，讲过ArrayList是基于数组的数据结构实现的，LinkedList是基于链表的数据结构实现的，而我今天要讲的HashMap是基于哈希表的数据结构实现的。我们不妨一起来温习下常用的数据结构，这样也有助于你更好地理解后面地内容。

**数组**：采用一段连续的存储单元来存储数据。对于指定下标的查找，时间复杂度为O(1)，但在数组中间以及头部插入数据时，需要复制移动后面的元素。

**链表**：一种在物理存储单元上非连续、非顺序的存储结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的。

链表由一系列结点（链表中每一个元素）组成，结点可以在运行时动态生成。每个结点都包含“存储数据单元的数据域”和“存储下一个结点地址的指针域”这两个部分。

由于链表不用必须按顺序存储，所以链表在插入的时候可以达到O(1)的复杂度，但查找一个结点或者访问特定编号的结点需要O(n)的时间。

<!-- [[[read_end]]] -->

**哈希表**：根据关键码值（Key value）直接进行访问的数据结构。通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做哈希函数，存放记录的数组就叫做哈希表。

**树**：由n（n≥1）个有限结点组成的一个具有层次关系的集合，就像是一棵倒挂的树。

## HashMap的实现结构

了解完数据结构后，我们再来看下HashMap的实现结构。作为最常用的Map类，它是基于哈希表实现的，继承了AbstractMap并且实现了Map接口。

哈希表将键的Hash值映射到内存地址，即根据键获取对应的值，并将其存储到内存地址。也就是说HashMap是根据键的Hash值来决定对应值的存储位置。通过这种索引方式，HashMap获取数据的速度会非常快。

例如，存储键值对（x，“aa”）时，哈希表会通过哈希函数f(x)得到"aa"的实现存储位置。

但也会有新的问题。如果再来一个(y，“bb”)，哈希函数f(y)的哈希值跟之前f(x)是一样的，这样两个对象的存储地址就冲突了，这种现象就被称为哈希冲突。<span class="orange">那么哈希表是怎么解决的呢？方式有很多，比如，开放定址法、再哈希函数法和链地址法。</span>

开放定址法很简单，当发生哈希冲突时，如果哈希表未被装满，说明在哈希表中必然还有空位置，那么可以把key存放到冲突位置后面的空位置上去。这种方法存在着很多缺点，例如，查找、扩容等，所以我不建议你作为解决哈希冲突的首选。

再哈希法顾名思义就是在同义词产生地址冲突时再计算另一个哈希函数地址，直到冲突不再发生，这种方法不易产生“聚集”，但却增加了计算时间。如果我们不考虑添加元素的时间成本，且对查询元素的要求极高，就可以考虑使用这种算法设计。

HashMap则是综合考虑了所有因素，采用链地址法解决哈希冲突问题。这种方法是采用了数组（哈希表）+ 链表的数据结构，当发生哈希冲突时，就用一个链表结构存储相同Hash值的数据。

## HashMap的重要属性

从HashMap的源码中，我们可以发现，HashMap是由一个Node数组构成，每个Node包含了一个key-value键值对。

```
transient Node<K,V>[] table;
```

Node类作为HashMap中的一个内部类，除了key、value两个属性外，还定义了一个next指针。当有哈希冲突时，HashMap会用之前数组当中相同哈希值对应存储的Node对象，通过指针指向新增的相同哈希值的Node对象的引用。

```
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
}
```

HashMap还有两个重要的属性：加载因子（loadFactor）和边界值（threshold）。在初始化HashMap时，就会涉及到这两个关键初始化参数。

```
int threshold;

    final float loadFactor;
```

LoadFactor属性是用来间接设置Entry数组（哈希表）的内存空间大小，在初始HashMap不设置参数的情况下，默认LoadFactor值为0.75。**为什么是0.75这个值呢？**

这是因为对于使用链表法的哈希表来说，查找一个元素的平均时间是O(1+n)，这里的n指的是遍历链表的长度，因此加载因子越大，对空间的利用就越充分，这就意味着链表的长度越长，查找效率也就越低。如果设置的加载因子太小，那么哈希表的数据将过于稀疏，对空间造成严重浪费。

那有没有什么办法来解决这个因链表过长而导致的查询时间复杂度高的问题呢？你可以先想想，我将在后面的内容中讲到。

Entry数组的Threshold是通过初始容量和LoadFactor计算所得，在初始HashMap不设置参数的情况下，默认边界值为12。如果我们在初始化时，设置的初始化容量较小，HashMap中Node的数量超过边界值，HashMap就会调用resize()方法重新分配table数组。这将会导致HashMap的数组复制，迁移到另一块内存中去，从而影响HashMap的效率。

## HashMap添加元素优化

初始化完成后，HashMap就可以使用put()方法添加键值对了。从下面源码可以看出，当程序将一个key-value对添加到HashMap中，程序首先会根据该key的hashCode()返回值，再通过hash()方法计算出hash值，再通过putVal方法中的(n - 1) & hash决定该Node的存储位置。

```
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

```
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

```
if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //通过putVal方法中的(n - 1) & hash决定该Node的存储位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```

<span class="orange">如果你不太清楚hash()以及(n-1)&amp;hash的算法，就请你看下面的详述。</span>

我们先来了解下hash()方法中的算法。如果我们没有使用hash()方法计算hashCode，而是直接使用对象的hashCode值，会出现什么问题呢？

假设要添加两个对象a和b，如果数组长度是16，这时对象a和b通过公式(n - 1) & hash运算，也就是(16-1)＆a.hashCode和(16-1)＆b.hashCode，15的二进制为0000000000000000000000000001111，假设对象 A 的 hashCode 为 1000010001110001000001111000000，对象 B 的 hashCode 为 0111011100111000101000010100000，你会发现上述与运算结果都是0。这样的哈希结果就太让人失望了，很明显不是一个好的哈希算法。

但如果我们将 hashCode 值右移 16 位（h >>> 16代表无符号右移16位），也就是取 int 类型的一半，刚好可以将该二进制数对半切开，并且使用位异或运算（如果两个数对应的位置相反，则结果为1，反之为0），这样的话，就能避免上面的情况发生。这就是hash()方法的具体实现方式。**简而言之，就是尽量打乱hashCode真正参与运算的低16位。**

我再来解释下(n - 1) & hash是怎么设计的，这里的n代表哈希表的长度，哈希表习惯将长度设置为2的n次方，这样恰好可以保证(n - 1) & hash的计算得到的索引值总是位于table数组的索引之内。例如：hash=15，n=16时，结果为15；hash=17，n=16时，结果为1。

在获得Node的存储位置后，如果判断Node不在哈希表中，就新增一个Node，并添加到哈希表中，整个流程我将用一张图来说明：

![](<https://static001.geekbang.org/resource/image/eb/d9/ebc8c027e556331dc327e18feb00c7d9.jpg?wh=1454*1380>)

**从图中我们可以看出：**在JDK1.8中，HashMap引入了红黑树数据结构来提升链表的查询效率。

这是因为链表的长度超过8后，红黑树的查询效率要比链表高，所以当链表超过8时，HashMap就会将链表转换为红黑树，这里值得注意的一点是，这时的新增由于存在左旋、右旋效率会降低。讲到这里，我前面我提到的“因链表过长而导致的查询时间复杂度高”的问题，也就迎刃而解了。

以下就是put的实现源码:

```
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
//1、判断当table为null或者tab的长度为0时，即table尚未初始化，此时通过resize()方法得到初始化的table
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
//1.1、此处通过（n - 1） & hash 计算出的值作为tab的下标i，并另p表示tab[i]，也就是该链表第一个节点的位置。并判断p是否为null
            tab[i] = newNode(hash, key, value, null);
//1.1.1、当p为null时，表明tab[i]上没有任何元素，那么接下来就new第一个Node节点，调用newNode方法返回新节点赋值给tab[i]
        else {
//2.1下面进入p不为null的情况，有三种情况：p为链表节点；p为红黑树节点；p是链表节点但长度为临界长度TREEIFY_THRESHOLD，再插入任何元素就要变成红黑树了。
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
//2.1.1HashMap中判断key相同的条件是key的hash相同，并且符合equals方法。这里判断了p.key是否和插入的key相等，如果相等，则将p的引用赋给e

                e = p;
            else if (p instanceof TreeNode)
//2.1.2现在开始了第一种情况，p是红黑树节点，那么肯定插入后仍然是红黑树节点，所以我们直接强制转型p后调用TreeNode.putTreeVal方法，返回的引用赋给e
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
//2.1.3接下里就是p为链表节点的情形，也就是上述说的另外两类情况：插入后还是链表/插入后转红黑树。另外，上行转型代码也说明了TreeNode是Node的一个子类
                for (int binCount = 0; ; ++binCount) {
//我们需要一个计数器来计算当前链表的元素个数，并遍历链表，binCount就是这个计数器

                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) 
// 插入成功后，要判断是否需要转换为红黑树，因为插入后链表长度加1，而binCount并不包含新节点，所以判断时要将临界阈值减1
                            treeifyBin(tab, hash);
//当新长度满足转换条件时，调用treeifyBin方法，将该链表转换为红黑树
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

## HashMap获取元素优化

当HashMap中只存在数组，而数组中没有Node链表时，是HashMap查询数据性能最好的时候。一旦发生大量的哈希冲突，就会产生Node链表，这个时候每次查询元素都可能遍历Node链表，从而降低查询数据的性能。

特别是在链表长度过长的情况下，性能将明显降低，红黑树的使用很好地解决了这个问题，使得查询的平均复杂度降低到了O(log(n))，链表越长，使用黑红树替换后的查询效率提升就越明显。

我们在编码中也可以优化HashMap的性能，例如，重写key值的hashCode()方法，降低哈希冲突，从而减少链表的产生，高效利用哈希表，达到提高性能的效果。

## HashMap扩容优化

HashMap也是数组类型的数据结构，所以一样存在扩容的情况。

在JDK1.7 中，HashMap整个扩容过程就是分别取出数组元素，一般该元素是最后一个放入链表中的元素，然后遍历以该元素为头的单向链表元素，依据每个被遍历元素的 hash 值计算其在新数组中的下标，然后进行交换。这样的扩容方式会将原来哈希冲突的单向链表尾部变成扩容后单向链表的头部。

而在 JDK 1.8 中，HashMap对扩容操作做了优化。由于扩容数组的长度是 2 倍关系，所以对于假设初始 tableSize = 4 要扩容到 8 来说就是 0100 到 1000 的变化（左移一位就是 2 倍），在扩容中只用判断原来的 hash 值和左移动的一位（newtable 的值）按位与操作是 0 或 1 就行，0 的话索引不变，1 的话索引变成原索引加上扩容前数组。

之所以能通过这种“与运算“来重新分配索引，是因为 hash 值本来就是随机的，而hash 按位与上 newTable 得到的 0（扩容前的索引位置）和 1（扩容前索引位置加上扩容前数组长度的数值索引处）就是随机的，所以扩容的过程就能把之前哈希冲突的元素再随机分布到不同的索引中去。

## 总结

HashMap通过哈希表数据结构的形式来存储键值对，这种设计的好处就是查询键值对的效率高。

我们在使用HashMap时，可以结合自己的场景来设置初始容量和加载因子两个参数。当查询操作较为频繁时，我们可以适当地减少加载因子；如果对内存利用率要求比较高，我可以适当的增加加载因子。

<span class="orange">我们还可以在预知存储数据量的情况下，提前设置初始容量（初始容量=预知数据量/加载因子）。</span>

这样做的好处是可以减少resize()操作，提高HashMap的效率。

HashMap还使用了数组+链表这两种数据结构相结合的方式实现了链地址法，当有哈希值冲突时，就可以将冲突的键值对链成一个链表。

但这种方式又存在一个性能问题，如果链表过长，查询数据的时间复杂度就会增加。HashMap就在Java8中使用了红黑树来解决链表过长导致的查询性能下降问题。以下是HashMap的数据结构图：

![](<https://static001.geekbang.org/resource/image/c0/6f/c0a12608e37753c96f2358fe0f6ff86f.jpg?wh=1304*904>)

## 思考题

实际应用中，我们设置初始容量，一般得是2的整数次幂。你知道原因吗？

期待在留言区看到你的答案。也欢迎你点击“请朋友读”，把今天的内容分享给身边的朋友，邀请他一起学习。

## 精选留言(62)

- ![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,m_fill,h_34,w_34.jpeg)

  陆离

  2的幂次方减1后每一位都是1，让数组每一个位置都能添加到元素。 例如十进制8，对应二进制1000，减1是0111，这样在&hash值使数组每个位置都是可以添加到元素的，如果有一个位置为0，那么无论hash值是多少那一位总是0，例如0101，&hash后第二位总是0，也就是说数组中下标为2的位置总是空的。 如果初始化大小设置的不是2的幂次方，hashmap也会调整到比初始化值大且最近的一个2的幂作为capacity。

  作者回复: 回答正确，就是减少哈希冲突，均匀分布元素。

  2019-06-04

  **6

  **113

- ![img](https://static001.geekbang.org/account/avatar/00/10/49/28/4dcfa376.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,w_14.png)

  giserway

  1）通过将 Key 的 hash 值与 length-1 进行 & 运算，实现了当前 Key 的定位，2 的幂次方可以减少冲突（碰撞）的次数，提高 HashMap 查询效率； 2）如果 length 为 2 的次幂，则 length-1  转化为二进制必定是 11111…… 的形式，在于 h 的二进制与操作效率会非常的快，而且空间不浪费；如果 length 不是 2 的次幂，比如 length 为 15，则 length-1 为 14，对应的二进制为 1110，在于 h 与操作，最后一位都为 0，而 0001，0011，0101，1001，1011，0111，1101 这几个位置永远都不能存放元素了，空间浪费相当大，更糟的是这种情况中，数组可以使用的位置比数组长度小了很多，这意味着进一步增加了碰撞的几率，减慢了查询的效率！这样就会造成空间的浪费。

  作者回复: 回答非常到位

  2019-06-04

  **4

  **53

- ![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425613967-1426.jpeg)

  Sdylan

  装载因子0.75是怎么算出来了，是经验值还是什么？ 另外为什么链表长度8就要转红黑树呢

  作者回复: 加载因子过高，虽然提高了空间的利用率，但增加了查询时间的成本；加载因子过低，虽然减少查询时间的成本，但是空间利用率又很低了。所以0.75是一个折中的选择。 链表长度为8转为红黑树的原因是，官方根据泊松分布实验发现，假设hashmap长度length为16，假设放入12（0.75*16）个数据到hashmap中，链表中存放8个节点的概率仅为0.00000006，而链表中存放1~7节点的概率为： 0: 0.60653066 1: 0.30326533 2: 0.07581633 3: 0.01263606 4: 0.00157952 5: 0.00015795 6: 0.00001316 7: 0.00000094 从以上可知，实际上一个链表被放满8个节点的概率非常小，实际上链表转红黑树是非常耗性能的，而链表在8个节点以内的平均查询时间复杂度与黑红树相差无几，超过8个节点，黑红树的查询复杂度会好一些。所以，当链表的节点大于等于8个的时候，转为红黑树的性价比比较合适。

  2019-10-12

  **4

  **37

- ![img](https://static001.geekbang.org/account/avatar/00/10/2b/bb/5cf70df8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  嘉嘉☕

  加载因子那块儿，感觉有点跳跃，为什么加载因子越大，对空间利用越充分呢？

  作者回复: 加载因子是扩容的参考标准，如果加载因子越大，例如默认数组初始化大小为16，加载因子由0.75改成1.0，原来数组长度到了12就扩容，变成数组大小为16，也就是说被占满了，才会进行扩容，这也可能意味着已经发生了很多哈希冲突，这样导致某些数组中的链表长度增加，影响查询效率。

  2019-06-13

  **3

  **30

- ![img](https://static001.geekbang.org/account/avatar/00/10/e4/64/6e458806.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大虫子

  老师您好，能解答下，为什么JDK1.8之前，链表元素增加采用的是头插法，1.8之后改成尾插法了。1.8之前采用头插法是基于什么设计思路呢？

  作者回复: 你好，JDK1.7是考虑新增数据大多数是热点数据，所以考虑放在链表头位置，也就是数组中，这样可以提高查询效率，但这种方式会出现插入数据是逆序的。在JDK1.8开始hashmap链表在节点长度达到8之后会变成红黑树，这样一来在数组后节点长度不断增加时，遍历一次的次数就会少很多，相比头插法而言，尾插法操作额外的遍历消耗已经小很多了。 也有很多人说避免多线程情况下hashmap扩容时的死循环问题，我个人觉得避免死循环的关键不在尾插法的改变，而是扩容时，用了首尾两个指针来避免死循环。这个我会在后面的多线程中讲到hashmap扩容导致死循环的问题。

  2019-06-04

  **4

  **29

- ![img](https://static001.geekbang.org/account/avatar/00/14/3b/ad/31193b83.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,w_14.png)

  孙志强

  以前看源码，我记得好像链表转换红黑树不光链表元素大于8个，好像还有一个表的大小大于64

  作者回复: 对的，有这样一个条件。

  2019-06-05

  **

  **21

- ![img](https://static001.geekbang.org/account/avatar/00/14/bc/52/52745d32.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小小征

  0 的话索引不变，1 的话索引变成原索引加上扩容前数组。  这句有点不理解 老师

  作者回复: 以下是resize中判断是否位移的部分代码，我们可以看到元素的hash值与原数组容量运算，如果运算结果为0，保持原位，如果运算结果为1，则意向扩容的高位。 		if ((e.hash & oldCap) == 0) {                             if (loTail == null)                                 loHead = e;                             else                                 loTail.next = e;                             loTail = e;                         }                         else {                             if (hiTail == null)                                 hiHead = e;                             else                                 hiTail.next = e;                             hiTail = e;                         } 假设链表中有4、8、12，他们的二进制位00000100、00001000、00001100，而原来数组容量为4，则是 00000100，以下与运算： 00000100 & 00000100 = 0 保持原位 00001000 & 00000100 = 1 移动到高位 00001100 & 00000100 = 1 移动到高位

  2019-06-05

  **10

  **12

- ![img](https://static001.geekbang.org/account/avatar/00/12/79/4b/740f91ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  -W.LI-

  老师好。hashmap的put和get的时间复杂度算多少啊?最好O(1)。最坏复杂度是O(log(n))平均是O(1)么?。。。treeMap的,treeMap，putO(n)，getO(1)?之前面试被问了，不晓得哪错了

  作者回复:  面试的时候，问到时间复杂度，大部分是考察你对数据结构的了解程度。建议可以多复习下数据结构的知识。 hashmap的最优时间复杂度是O（1），而最坏时间复杂度是O(n)。 在没有产生hash冲突的情况下，查询和插入的时间复杂度是O(1)； 而产生hash冲突的情况下，如果是最终插入到链表，链表的插入时间复杂度为O(1)，而查询的时候，时间复杂度为O(n)； 在产生hash冲突的情况下，如果最终插入的是红黑树，插入和查询的平均时间复杂度是O（logn）。 而TreeMap是基于红黑树实现的，所以时间复杂度你也清楚了吧。

  2019-06-04

  **2

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/78/3e/c39d86f1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Chocolate

  老师，您好，请教一个问题，为什么 HashMap 的容量等于数组长度？但是扩容的时候却是根据 Map 里的所有元素总数去扩容，这样会不会导致数组中的某一个 node 有很长的链表或红黑树，数组中的其他位置都没有元素？谢谢

  作者回复: 这种数据倾斜的可能性比较小

  2019-06-05

  **2

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/15/ff/0a/12faa44e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓杰

  初始容量2的n次方是偶数，在计算key的索引位置时，是通过(n-1)&hash计算的，这样n-1得到的奇数，那么通过在进行与操作时，如果hash的第一位是0，那么(n-1)&hash得到的是偶数，如果hash的第一位是1，那么(n-1)&hash得到的是奇数，因此可以让数据分布更加均匀，减少hash冲突，相反如果n-1是偶数，那无论hash的第一位是偶数还是奇数，(n-1)&hash得到的都是偶数，不利于数据的分布

  2019-06-05

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/13/17/27/ec30d30a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jxin

  一下子追到最新了。老师集合这块讲得是真不错。弱弱补个小点。空集合或空map返回不要new，走常量不可变集合或map。尽量减少对象的创建。毕竟创建和回收都是有开销的。

  2019-06-04

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/14/56/4e/60e50534.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  M.c

  “这是因为链表的长度超过 8 后，红黑树的查询效率要比链表高，所以当链表超过 8 时，HashMap 就会将链表转换为红黑树”，此处转换为红黑树少了个条件吧？MIN_TREEIFY_CAPACITY要同时大于64

  作者回复: 是的，在源码中可以查到对于的代码，链表长度大于8而且整个map中的键值对大于等于MIN_TREEIFY_CAPACITY (64)时，才进行链表到红黑树的转换。

  2020-02-16

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/2b/fa/1cde88d4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大俊stan

  作者如果把扩容的源码贴出来，可能更好理解是如何扩容，以及为什么多线程hashmap会产生闭环

  作者回复: 好的，收到建议

  2019-08-23

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/2a/54/c9990105.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bro.

  编程中有遇到这种情况,当判断一个数n 是否为偶数使用n%2 == 0 ,计算机语言已经做了优化为 n&(2-1) = n & 1 == 0,对于二进制来说与操作只需要一个电路即可完成比取余速度快多了,相同的对于hash扩容,只需要判断前一位hash值是0还是1,如果是0保持数组位置不变,如果为1增加原来扩容前数组长度即可,而且由于hash值计算每一位都是平均分配0或者1,所以保持均匀分布

  2019-06-10

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/d7/df/fc0a6709.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  WolvesLeader

  hashmap在多线程情况下数据丢失，大师，能不能分析分析原因

  作者回复: 你好，这个在后面多线程中会讲到，可以留意关注下。

  2019-06-04

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/15/f6/ea/9b41ce10.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陆劲

  要是早一天发这个就好了，昨天面试被问到哈希函数给问懵了。老师很棒！这几乎是我在极客跟的最紧的课。

  作者回复: 哈希表在面试中是出现较频繁的，不仅因为它在平时工作中用的比较多，还由于它优秀的设计和优化思想。

  2019-06-04

  **

  **3

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  小白

  老师你好，请教一个问题。HashMap 长度等于数组长度吗？ 那么它的链表下面存的那些还算吗？ 如果下标为1的位置下挂了 3个node ？ 那么它的数组长度还是16 吗？  还是 13 ？

  2020-07-14

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/14/bd/13/6424f528.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  luzero

  使用2的次幂是因为进行&运算的时候，每次都能落到tables数组内，并且2的次幂进行&运算和直接模运算（%）的值是一样的，也就是说（n-1）&hash==hash%n，如果直接使用（%）模运算最终也会转换成二进制的去计算性能不如&运算，还有就是&计算分布均匀，减少哈希冲突，如果是2的次幂,假设n=16，（16-1）&0==0、（16-1）&1==1、16-1)&2==2、........、（16-1)&19==3、（16-1)&20==4、（16-1)&21==5、。如果不是2的次幂的话，假设是n=15，（15-1）&1==0、（15-1）&1==2、（15-1)&3==2、......、（15-1)&4==4、（15-1)&5==4.    哈哈、不知道我回答的沾不沾边？望老师您点评一下哈

  作者回复: 理解正确，赞

  2019-06-28

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/10/b3/c5/7fc124e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Liam

  Hash字典发生扩容时，需要复制元素，请问这个过程是一次完成的吗？redis里的字典是准备了两个字典，一个原始字典，一个rehash字典，扩容后，不是一次完成数据迁移的，每次操作字典都考虑两个数组并复制数据，扩容完毕后交换两个数组指针

  作者回复: HashMap的扩容是一次性完成的。

  2019-06-05

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/10/e0/c5/0a727162.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  曾泽伟

  再哈希--如果我们不考虑添加元素的时间成本，且对查询元素的要求极高，就可以使用此法 我没太明白，为什么再哈希会提高查询？如果再哈希了，我第一次哈希的位置没查到，得再次哈希查找位置，如果还没有又要再哈希，循环往复，感觉查询性能并不高啊，老师能解释一下吗？

  作者回复: 我们知道再hash也是放在数组上的，这样比放在链表或红黑树上来说，查询性能是会提升的，一个简单的hash计算的速度远远要比遍历链表或红黑树要快。

  2019-06-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/26/38/ef063dc2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,w_14.png)

  Darren

  因为选择下标和扩容的时候都依赖和(n-1）按位与，2的幂次方满足n-1都是1的情况，因此必须选择2的幂次方，且如果使用时传入的参数不是2的幂次方，HashMap会自动转成大于传入参数的最小的2的幂次方来确保下标和扩容机制的正常运行

  2019-06-04

  **

  **2

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  suke

  内容讲的很跳跃,factor和threshold怎么参与计算的也不说,这思路一点也不严谨啊,付费内容能不能有点耐心

  2019-11-05

  **1

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/5d/57/82f1a3d4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  吾爱有三

  老师您好，1.8中hashmap解决了并发场景的死循环，那么为什么在高并发还是不建议使用hashmap？

  作者回复: 因为存在线程安全性问题，例如put数据丢失，size()最终的数据不对等等。

  2019-07-08

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/54/9a/76c0af70.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  每天晒白牙

  static int indexFor(int h, int length) {      return h & (length-1); } 我们知道对于HashMap的table而言，数据分布需要均匀（最好每项都只有一个元素，这样就可以直接找到），不能太紧也不能太松，太紧会导致查询速度慢，太松则浪费空间。计算hash值后，怎么才能保证table元素分布均与呢？我们会想到取模，但是由于取模的消耗较大，HashMap是这样处理的：调用indexFor方法。 HashMap源码中有一个indexFor方法，返回的是key的hashcode跟初始容量-1做与运算。 首先length为2的整数次幂的话，h&(length-1)就相当于对length取模，这样便保证了散列的均匀，同时也提升了效率； 其次，length为2的整数次幂的话，为偶数。这样length-1为奇数，奇数的最后一位为1，这样便保证了h&(length-1)的最后一位为0，也可能为1（这取决于h的值），即与后的结果可能为偶数也可能是奇数。这样便可以保证散列的均匀性， 而如果length为奇数的话，很明显length-1为偶数，它的最后一位是0，这样h&(length-1)的最后一位肯定为0，即只能为偶数，这样任何hash值都只会被散列到数组的偶数下标位置上，这便浪费了近一半的空间。所以，length取2的整数次幂，是为了使不同hash值发生碰撞的概率较小，这样就能使元素在哈希表中均匀地散列。 ---------------------  作者：每天晒白牙  来源：CSDN  原文：https://blog.csdn.net/dam454450872/article/details/80376661  版权声明：本文为博主原创文章，转载请附上博文链接！

  2019-06-18

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/49/7d/7b9fd831.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Fever

  Hash冲突，那么可以把 key 存放到冲突位置的空位置上去，这里是不是写错了，应该是冲突位置后面的空位置吧？

  作者回复: 是的，描述不清楚，感谢提醒。

  2019-06-12

  **

  **1

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/8ibhsSDmHmichVKzxW6vN6Ln2sy8ibOcG8O8akwPDnia98cqLwO29qk4mUqNScwMSlVgOAlgUNw0YnbyosjIJGAibIg/132)

  jimmy

  请问下为什么链表转换成红黑树的阈值是8？

  作者回复: 这是兼容链表查询和转红黑树两者性能一起考虑的，这个值经过不是绝对的最佳值，是基于大数据求得比较合适的阈值。

  2019-06-10

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/17/11/6b/8034959a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  迎风劲草

  主要是为了减少hash冲突，hash & (n-1)  n为2的n次幂，n-1换成比特位都是1

  2019-06-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/69/c6/513df085.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  强哥

  最主要的原因是位运算替代%取模操作，提高运算性能，说什么降低冲突的，是否比较过质数和偶数冲突概率呢？

  作者回复: 用与运算是提高了运算性能，而容量大小为2的幂次方是为了降低哈希冲突。

  2019-06-04

  **

  **1

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/sQV132p5s2Nicia1wJPiafJtMP5ic0kwpzntdkibInNScCeic0ZvDE7y90nCM7rGDaZ2DiaRlYjhoplOpJVGlbibkToAPg/132)

  Geek_964b0f

  hashMap中计算索引时使用（n-1)&hash，当n为2的幂次方时，n-1的二进制数为*111的形式，能更好的减少哈希冲突，jdk8中有对初始容量设置为2的幂次方的操作，例如输入9，会将其转化为16

  2020-12-17

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  Geek_a8b86d

  例如：hash=15，n=16 时，结果为 15；hash=17，n=16 时，结果为 1。这个是怎么算的？

  2020-11-18

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/81/e6/6cafed37.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  旅途

  老师 对于使用2的幂次方减一然后与hash进行与的位运算这个我懂了,但是好像没理解透彻,假如十进制8 二进制1000 对hash使用位或（|）操作,这样效果一样吗

  2020-08-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/df/bf/96b50d1e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  😚 46

  在 HashMap 初始化时，threshold = 初始容量，默认是16。 当插入第一个元素后，会进入 resize 方法，此时 threshold = 容量 * 加载因子。 

  2020-07-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/f4/8f/6b3d4370.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  瑶老板的小弟

  1.8优化，扩容时候链表尾节点插入，避免环链。

  2020-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/c5/2d/1eebfc3c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  GaGi

  老师你好，文中说jdk1.8的扩容优化：“在扩容中只用判断原来的 hash 值和左移动的一位（newtable 的值）按位与操作是 0 或 1 就行”， 查看了1.8的源码，发现是hash&oldCap，也就是说跟原本的数组长度进行比较

  2020-05-01

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/aEvdVcM1ESQUrsiaYdwAqhqcOfhZ9dxtEetUwm4tfhHbhRUspIyvrFjibQkicUL3N03e5pmSGTfz2OzZ7ibZMn4k2Q/132)

  其实还OK

  如果每个元素的hash值都是唯一，特别是低位的最大程度不同时，初始容量设置为2的n次幂可以保持与运算后结果与元素的hash值低位一样，就可以达到最大程度的错开元素的存储

  2020-04-02

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  赵玉闯

  老师，我想问一下，如果数组扩容，是不是应该是基于原来数组尾部继续申请连续的内存空间，也就是原来的二倍，那如果基于原来数组的尾部没有连续的这么大的空间了，怎么办。会去一个新的地方申请原来的二倍空间，然后把这些数据都挪过去吗？

  作者回复: 是的，arraylist的扩容方式则是直接重新申请一个新的空间，将原来的数组中的元素复制过去

  2020-03-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  很棒 1.就是被hashmap中链表的 头部 和 尾部搞晕了，老师，是数组里的元素是链表头部还是尾部啊？ 2.扩容方法，呜呜呜… 有点想老师也能贴一下扩容源码分析。

  作者回复: 1、只是一个首尾数据调换的过程； 2、后面补上，评论里面字数有限，无法将完整源码贴上。

  2019-12-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b5/0d/0e65dee6.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,w_14.png)

  FelixFly

  老师，实际应用中，我们设置初始容量，一般得是 2 的整数次幂，这个HashMap在初始化容量的时候会重新计算，会重新计算为2 的整数次幂 我们设置初始容量的要考虑加载因子，设置的初始容量是预知数据量 / 加载因子，假如预知容量是20，算出来的结果是26.6，这样设置的初始容量应该为27，但是计算出来的容量是32（也就是2的5次幂），也就是说这时候的默认边界值为32*0.75=24 还有另一种方法是设置加载因子改为1.0，预售容量是20，设置初始容量为20，加载因子为1.0，这样计算出来的容量是32，也就是说默认边界值是32 这两种方法在实际应用中哪种比较好点

  作者回复: 第一种，加载因子考虑的更多的是扩容

  2019-11-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/e3/41/bd0e3a04.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](07%20_%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAHashMap%E7%9A%84%E8%AE%BE%E8%AE%A1%E4%B8%8E%E4%BC%98%E5%8C%96.resource/resize,w_14.png)

  雷小鸿

  2次幂就是hash%length和(n一1)&hash相等。用&运算效率高。不是2次幂不保证和取余相等。就不会均匀分配。

  2019-11-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/a0/cb/aab3b3e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张三丰

  ”这种方法存在着很多缺点，例如，查找、扩容等，所以我不建议你作为解决哈希冲突的首选。” 老师能否讲讲为什么扩容时候有缺点？

  作者回复: 扩容需要迁移数据，消耗性能

  2019-10-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/37/d0/26975fba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  aof

  老师，这句"因此加载因子越大，对空间的利用就越充分，这就意味着链表的长度越长”，为什么加载因子越大链表长度越长？

  作者回复: 因为将加载因子调大，填满的元素越多，空间利用率越高，但冲突的机会加大了

  2019-10-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  课后思考及问题 假设初始容量设置为4，4是2的幂次方， 4(十进制)=100(二进制)，100-1=011，假设是四个字节的整型数字，一个字节八位4*8=32位，则在前面再补29个0。则不管计算的哈希值是什么和011相与后，000/001/010/011(前面还有29个0)这四种情况都是可能存在的，所以他们的位置是都能被占用的，当然上，如果数据超过四个就必然会发生多个数字落到同一个位置的情况啦！这就是哈希冲突啦！ 再假设初始容量为5， 5(十进制)=101(二进制)，101-1=100，同样假设是四个字节的整型数字，一个字节八位4*8=32位，则在前面再补29个0。则不管计算的哈希值是什么和100相与后，只能得到000/100这两个位置(前面还有29个0)，其中这三个位置001/010/011，无论如何都是不会被占用的，五个位置即使超过两个就必然会发生哈希冲突，并且还有三个位置永远用不了。 很明显通过一正一反两个极其简单的例子，就能反应出，如果初始容量不是2的幂次方，则会造成更多的空间浪费和更高的冲突率。 当然，位运算的性能总是最佳的，这么操作性能也会很棒! 如果把初始容量从1到100都推演一下，估计效果会更好，印象更深刻。

  2019-09-07

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/08/92f42622.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  尔冬橙

  老师，hashmap里判断两个对象相不相等先是比较hash函数，而hash函数是hashcode右移16位异或得到的，那么hash函数相同，hashcode一定相同么

  作者回复: 不一定，这里只是一种算hash值的方式，与key的hashcode并不存在比较。

  2019-08-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/8b/4b/fa52d222.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  行则将至

  作者回复: 假设链表中有4、8、12，他们的二进制位00000100、00001000、00001100，而原来数组容量为4，则是 00000100，以下与运算： 00000100 & 00000100 = 0 保持原位 00001000 & 00000100 = 1 移动到高位 00001100 & 00000100 = 1 移动到高位 老师，您这个例子的结果说错了吧？ 结果应该是1   0   1  吧？ 00000100这个也不代表1吧。换算成十进制是4啊。 这个问题，一直不理解，希望老师

  作者回复: 纠正下按位“与”运算，这里应该是： 00000100 & 00000100 = 非0 移动到高位 00001000 & 00000100 = 0 保持原位 00001100 & 00000100 = 非0 移动到高位

  2019-08-10

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/b3/798a4bb2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  帽子丨影

  文中最后说到扩容是从新分配索引是使用hash值与newCap进行&操作，但是看源码是和oldCap进行&操作的，我觉得也应该是与oldCap进行比较：因为这一步主要是看hash值在oldCap那个1的位置是不是1，如果是1则表示在newCap上也是1(而在oldCap这个1被忽略了)，需要放在高位上

  2019-08-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/52/37/13b4c8aa.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Vincent

  不管你初始容量是否设置为2的整数次幂，hashmap 通过tableSizeFor 方法转换成了2的整数次幂。至于为什么要2的整数次幂 是因为hash 函数计算hash值时候使用了(n - 1) & hash, 如果n是2的整数次幂，那么n-1 就是所有二进制位都是1，可以保证所有函数都在数组table 的某一个索引位置。所以这个时候我们只要保证hash函数返回的hash值区分度大，就可以减少冲突的可能。

  2019-07-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/50/96/18612c89.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  克

  总不能等到每个数组idx都有元素才扩容吧。那样鬼知道什么时候填满，按照元素总数初以加载因子就是为了减少链表长度。

  2019-07-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/bd/13/6424f528.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  luzero

  老师请教一个问题，HashMap 获取元素优化这个章节中的 “例如，重新 key 值的 hashCode() 方法”，是重写？还是重新？ 哈哈哈、我有点搞蒙了哦

  作者回复: 是重写

  2019-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/81/0c/22cf60a9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Forwardジ

  老师，hashmap的大小看文章里没讲解，这块怎么总结？

  作者回复: 是指hashmap扩容吗？

  2019-06-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/60/6d/e2576fda.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Rancood

  (n-1)&hash得到的是在数组下标在map的数组的范围之内

  2019-06-26

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b4/3b/a1f7e3a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ZOU志伟

  假设链表中有4、8、12，他们的二进制位00000100、00001000、00001100，而原来数组容量为4，则是 00000100，以下与运算： 00000100 & 00000100 = 0 保持原位 00001000 & 00000100 = 1 移动到高位 00001100 & 00000100 = 1 移动到高位 这段不理解，为何&运算是0或者1？

  作者回复: 纠正下按位“与”运算，这里应该是： 00000100 & 00000100 = 非0 移动到高位 00001000 & 00000100 = 0 保持原位 00001100 & 00000100 = 非0 移动到高位

  2019-06-19

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/b4/3b/a1f7e3a4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ZOU志伟

  元素获取和扩容讲得有点模糊，建议加点图标说明

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/bc/6d/f6f0a442.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  汤小高

  老师，有个疑问： (e.hash & oldCap)，它的两种结果，一个是0，另一个应该是oldCap，老师为什么说一种结果是0，一种结果是1呢？另一种结果不一定是1吧？比如hash为3，二进制是0011，oldCap为4，二进制为0100  它们相与的结果为0   ，但是比如hash为4，二进制为0100  oldCap为4，二进制为为0100，它们相与的结果不还是0100吗，十进制不是4吗，怎么会是0了

  作者回复: 假设链表中有4、8、12，他们的二进制位00000100、00001000、00001100，而原来数组容量为4，则是 00000100，以下与运算： 00000100 & 00000100 = 非0 移动到高位 00001000 & 00000100 = 0 保持原位 00001100 & 00000100 = 非0 移动到高位

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/27/1d/1cb36854.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小辉辉

  HashMap源码看过几遍，跟着老师的思路来又学到了新东西

  2019-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/9c/35/9dc79371.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  你好旅行者

  对于老师提到的hash函数的目的是为了尽量打乱hashcode真正参与运算的低16位。我想问为什么只考虑了低16位，假设出现一种情况，hashcode的低位全部相等，高位不相等，那岂不是会大大加大哈希冲突吗？

  作者回复: 一个开源框架更多是考虑一个大环境下的优化场景，所以对于某些特殊场景，比如你说的这种低位全相等，高位不相等的情况，自己需要考虑如何去优化key值。

  2019-06-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/ff/0a/12faa44e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓杰

  用与运算代替取模运算，可以提高运算的效率，而且当n是2的幂次方时：hash & (n- 1) == hash % n

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/8e/cf0b4575.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郑晨Cc

  思考题：因为计算机是二进制 数组大小设置为2的整数次幂有利于使用位运算代替取模求余，就像在人类的思维中很容易看出一个数字对10的正数次幂求余数一样。扩容中的位运算也是类似的道理

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/69/c6/513df085.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  强哥

  我继续之前的问题，如果容量不是2的幂次方，那还能用位运算代替模运算吗？这个是否有人思考过呢？

  2019-06-05

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  胖妞

  平常基本会用就满足了，所以，一般不会设置map的初始值，底层算法有点复杂，尤其位运算，我都需要好好想想，才能明白干啥！从文中看，开始map创建就是一个node数组，只是，在计算存放node位置的时候，如果node位置计算出现冲突(一致)了，就将把冲突的后一个node转存到前一个node尾部，变成一个链表，当链表长度超过8后，这种链表又会重新转成树结构！？是吗？这里面的几次转变，产生的一些问题，也不是很明白！第一，hase算法，计算位置的时候，什么情况下会出现结果一致，第二，链表转换为树为啥定位为8和长度限制，第三，转换为树的时候，肯定会把以前的链表元素全部遍历后，重新创建数据结构，那么，这种重新创建数据结构的时间话费难道还比在后面继续追加来的快吗？算法小白一枚！尤其这种涉及位运算的！唉！看来要好好看一下基础了！

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fe/44/3e3040ac.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员人生

  降低hash碰撞，增加空间利用率

  2019-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/26/d9/f7e96590.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  yes

  1.8那个流程图中不转树的话，node是插入尾部吧不是头部

  作者回复: 是的，感谢提醒！~

  2019-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f1/c9/adc1df03.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  哲

  主要是为了方便位运算吧

  作者回复: 对的，具体是为了降低哈希冲突，均匀分布新增元素，提高查询性能。

  2019-06-04

