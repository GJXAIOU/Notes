# 05 \| ArrayList还是LinkedList？使用不当性能差千倍

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/e0/4d/e0e7e861db0466a5c60d14fe0b5bfe4d.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/ad/b6/ad7930998d431602b4ca022d551fc5b6.mp3" type="audio/mpeg"></audio>

你好，我是刘超。

集合作为一种存储数据的容器，是我们日常开发中使用最频繁的对象类型之一。JDK为开发者提供了一系列的集合类型，这些集合类型使用不同的数据结构来实现。因此，不同的集合类型，使用场景也不同。

很多同学在面试的时候，经常会被问到集合的相关问题，比较常见的有ArrayList和LinkedList的区别。

相信大部分同学都能回答上：“ArrayList是基于数组实现，LinkedList是基于链表实现。”

而在回答使用场景的时候，我发现大部分同学的答案是：<span class="orange">“ArrayList和LinkedList在新增、删除元素时，LinkedList的效率要高于 ArrayList，而在遍历的时候，ArrayList的效率要高于LinkedList。”</span>

这个回答是否准确呢？今天这一讲就带你验证。

## 初识List接口

在学习List集合类之前，我们先来通过这张图，看下List集合类的接口和类的实现关系：

![](<https://static001.geekbang.org/resource/image/54/09/54f564eb63a2c74723a82540668fc009.jpg?wh=1000x1001>)

我们可以看到ArrayList、Vector、LinkedList集合类继承了AbstractList抽象类，而AbstractList实现了List接口，同时也继承了AbstractCollection抽象类。ArrayList、Vector、LinkedList又根据自我定位，分别实现了各自的功能。

<!-- [[[read_end]]] -->

ArrayList和Vector使用了数组实现，这两者的实现原理差不多，LinkedList使用了双向链表实现。基础铺垫就到这里，接下来，我们就详细地分析下ArrayList和LinkedList的源码实现。

## ArrayList是如何实现的？

ArrayList很常用，先来几道测试题，自检下你对ArrayList的了解程度。

**问题1：**我们在查看ArrayList的实现类源码时，你会发现对象数组elementData使用了transient修饰，我们知道transient关键字修饰该属性，则表示该属性不会被序列化，然而我们并没有看到文档中说明ArrayList不能被序列化，这是为什么？

**问题2：**我们在使用ArrayList进行新增、删除时，经常被提醒“使用ArrayList做新增删除操作会影响效率”。那是不是ArrayList在大量新增元素的场景下效率就一定会变慢呢？

**问题3：**如果让你使用for循环以及迭代循环遍历一个ArrayList，你会使用哪种方式呢？原因是什么？

如果你对这几道测试都没有一个全面的了解，那就跟我一起从数据结构、实现原理以及源码角度重新认识下ArrayList吧。

### 1\.ArrayList实现类

ArrayList实现了List接口，继承了AbstractList抽象类，底层是数组实现的，并且实现了自增扩容数组大小。

ArrayList还实现了Cloneable接口和Serializable接口，所以他可以实现克隆和序列化。

ArrayList还实现了RandomAccess接口。你可能对这个接口比较陌生，不知道具体的用处。通过代码我们可以发现，这个接口其实是一个空接口，什么也没有实现，那ArrayList为什么要去实现它呢？

其实RandomAccess接口是一个标志接口，他标志着“只要实现该接口的List类，都能实现快速随机访问”。

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

### 2\.ArrayList属性

ArrayList属性主要由数组长度size、对象数组elementData、初始化容量default\_capacity等组成， 其中初始化容量默认大小为10。

```
//默认初始化容量
    private static final int DEFAULT_CAPACITY = 10;
    //对象数组
    transient Object[] elementData; 
    //数组长度
    private int size;
```

从ArrayList属性来看，它没有被任何的多线程关键字修饰，但elementData被关键字transient修饰了。这就是我在上面提到的第一道测试题：transient关键字修饰该字段则表示该属性不会被序列化，但ArrayList其实是实现了序列化接口，这到底是怎么回事呢？

这还得从“ArrayList是基于数组实现“开始说起，由于ArrayList的数组是基于动态扩增的，所以并不是所有被分配的内存空间都存储了数据。

如果采用外部序列化法实现数组的序列化，会序列化整个数组。ArrayList为了避免这些没有存储数据的内存空间被序列化，内部提供了两个私有方法writeObject以及readObject来自我完成序列化与反序列化，从而在序列化与反序列化数组时节省了空间和时间。

因此使用transient修饰数组，是防止对象数组被其他外部方法序列化。

### 3\.ArrayList构造函数

ArrayList类实现了三个构造函数，第一个是创建ArrayList对象时，传入一个初始化值；第二个是默认创建一个空数组对象；第三个是传入一个集合类型进行初始化。

当ArrayList新增元素时，如果所存储的元素已经超过其已有大小，它会计算元素大小后再进行动态扩容，数组的扩容会导致整个数组进行一次内存复制。因此，我们在初始化ArrayList时，可以通过第一个构造函数合理指定数组初始大小，这样有助于减少数组的扩容次数，从而提高系统性能。

```
public ArrayList(int initialCapacity) {
        //初始化容量不为零时，将根据初始化值创建数组大小
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {//初始化容量为零时，使用默认的空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    public ArrayList() {
        //初始化默认为空数组
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

### 4\.ArrayList新增元素

ArrayList新增元素的方法有两种，一种是直接将元素加到数组的末尾，另外一种是添加元素到任意位置。

```
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

两个方法的相同之处是在添加元素之前，都会先确认容量大小，如果容量够大，就不用进行扩容；如果容量不够大，就会按照原来数组的1.5倍大小进行扩容，在扩容之后需要将数组复制到新分配的内存地址。

```
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

当然，两个方法也有不同之处，添加元素到任意位置，会导致在该位置后的所有元素都需要重新排列，而将元素添加到数组的末尾，在没有发生扩容的前提下，是不会有元素复制排序过程的。

这里你就可以找到第二道测试题的答案了。如果我们在初始化时就比较清楚存储数据的大小，就可以在ArrayList初始化时指定数组容量大小，并且在添加元素时，只在数组末尾添加元素，那么ArrayList在大量新增元素的场景下，性能并不会变差，反而比其他List集合的性能要好。

### 5\.ArrayList删除元素

ArrayList的删除方法和添加任意位置元素的方法是有些相同的。ArrayList在每一次有效的删除元素操作之后，都要进行数组的重组，并且删除的元素位置越靠前，数组重组的开销就越大。

```
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

### 6\.ArrayList遍历元素

由于ArrayList是基于数组实现的，所以在获取元素的时候是非常快捷的。

```
public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    E elementData(int index) {
        return (E) elementData[index];
    }
```

## LinkedList是如何实现的？

虽然LinkedList与ArrayList都是List类型的集合，但LinkedList的实现原理却和ArrayList大相径庭，使用场景也不太一样。

LinkedList是基于双向链表数据结构实现的，LinkedList定义了一个Node结构，Node结构中包含了3个部分：元素内容item、前指针prev以及后指针next，代码如下。

```
private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

总结一下，LinkedList就是由Node结构对象连接而成的一个双向链表。在JDK1.7之前，LinkedList中只包含了一个Entry结构的header属性，并在初始化的时候默认创建一个空的Entry，用来做header，前后指针指向自己，形成一个循环双向链表。

在JDK1.7之后，LinkedList做了很大的改动，对链表进行了优化。链表的Entry结构换成了Node，内部组成基本没有改变，但LinkedList里面的header属性去掉了，新增了一个Node结构的first属性和一个Node结构的last属性。这样做有以下几点好处：

- first/last属性能更清晰地表达链表的链头和链尾概念；
- first/last方式可以在初始化LinkedList的时候节省new一个Entry；
- first/last方式最重要的性能优化是链头和链尾的插入删除操作更加快捷了。

<!-- -->

这里同ArrayList的讲解一样，我将从数据结构、实现原理以及源码分析等几个角度带你深入了解LinkedList。

### 1\.LinkedList实现类

LinkedList类实现了List接口、Deque接口，同时继承了AbstractSequentialList抽象类，LinkedList既实现了List类型又有Queue类型的特点；LinkedList也实现了Cloneable和Serializable接口，同ArrayList一样，可以实现克隆和序列化。

由于LinkedList存储数据的内存地址是不连续的，而是通过指针来定位不连续地址，因此，LinkedList不支持随机快速访问，LinkedList也就不能实现RandomAccess接口。

```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

### 2\.LinkedList属性

我们前面讲到了LinkedList的两个重要属性first/last属性，其实还有一个size属性。我们可以看到这三个属性都被transient修饰了，原因很简单，我们在序列化的时候不会只对头尾进行序列化，所以LinkedList也是自行实现readObject和writeObject进行序列化与反序列化。

```
transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;
```

### 3\.LinkedList新增元素

LinkedList添加元素的实现很简洁，但添加的方式却有很多种。默认的add (Ee)方法是将添加的元素加到队尾，首先是将last元素置换到临时变量中，生成一个新的Node节点对象，然后将last引用指向新节点对象，之前的last对象的前指针指向新节点对象。

```
public boolean add(E e) {
        linkLast(e);
        return true;
    }

    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

LinkedList也有添加元素到任意位置的方法，如果我们是将元素添加到任意两个元素的中间位置，添加元素操作只会改变前后元素的前后指针，指针将会指向添加的新元素，所以相比ArrayList的添加操作来说，LinkedList的性能优势明显。

```
public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }

    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

### 4\.LinkedList删除元素

在LinkedList删除元素的操作中，我们首先要通过循环找到要删除的元素，如果要删除的位置处于List的前半段，就从前往后找；若其位置处于后半段，就从后往前找。

这样做的话，无论要删除较为靠前或较为靠后的元素都是非常高效的，但如果List拥有大量元素，移除的元素又在List的中间段，那效率相对来说会很低。

### 5\.LinkedList遍历元素

LinkedList的获取元素操作实现跟LinkedList的删除元素操作基本类似，通过分前后半段来循环查找到对应的元素。但是通过这种方式来查询元素是非常低效的，特别是在for循环遍历的情况下，每一次循环都会去遍历半个List。

所以在LinkedList循环遍历时，我们可以使用iterator方式迭代循环，直接拿到我们的元素，而不需要通过循环查找List。

## 总结

前面我们已经从源码的实现角度深入了解了ArrayList和LinkedList的实现原理以及各自的特点。如果你能充分理解这些内容，很多实际应用中的相关性能问题也就迎刃而解了。

就像如果现在还有人跟你说，“ArrayList和LinkedList在新增、删除元素时，LinkedList的效率要高于ArrayList，而在遍历的时候，ArrayList的效率要高于LinkedList”，你还会表示赞同吗？

现在我们不妨通过几组测试来验证一下。这里因为篇幅限制，所以我就直接给出测试结果了，对应的测试代码你可以访问[Github](<https://github.com/nickliuchao/collection>)查看和下载。

**1\.ArrayList和LinkedList新增元素操作测试**

- 从集合头部位置新增元素
- 从集合中间位置新增元素
- 从集合尾部位置新增元素

<!-- -->

测试结果(花费时间)：

- ArrayList>LinkedList
- ArrayList<LinkedList
- ArrayList<LinkedList

<!-- -->

通过这组测试，我们可以知道LinkedList添加元素的效率未必要高于ArrayList。

由于ArrayList是数组实现的，而数组是一块连续的内存空间，在添加元素到数组头部的时候，需要对头部以后的数据进行复制重排，所以效率很低；而LinkedList是基于链表实现，在添加元素的时候，首先会通过循环查找到添加元素的位置，如果要添加的位置处于List的前半段，就从前往后找；若其位置处于后半段，就从后往前找。因此LinkedList添加元素到头部是非常高效的。

同上可知，ArrayList在添加元素到数组中间时，同样有部分数据需要复制重排，效率也不是很高；LinkedList将元素添加到中间位置，是添加元素最低效率的，因为靠近中间位置，在添加元素之前的循环查找是遍历元素最多的操作。

而在添加元素到尾部的操作中，我们发现，在没有扩容的情况下，ArrayList的效率要高于LinkedList。这是因为ArrayList在添加元素到尾部的时候，不需要复制重排数据，效率非常高。而LinkedList虽然也不用循环查找元素，但LinkedList中多了new对象以及变换指针指向对象的过程，所以效率要低于ArrayList。

说明一下，这里我是基于ArrayList初始化容量足够，排除动态扩容数组容量的情况下进行的测试，如果有动态扩容的情况，ArrayList的效率也会降低。

**2\.ArrayList和LinkedList删除元素操作测试**

- 从集合头部位置删除元素
- 从集合中间位置删除元素
- 从集合尾部位置删除元素

<!-- -->

测试结果(花费时间)：

- ArrayList>LinkedList
- ArrayList<LinkedList
- ArrayList<LinkedList

<!-- -->

ArrayList和LinkedList删除元素操作测试的结果和添加元素操作测试的结果很接近，这是一样的原理，我在这里就不重复讲解了。

**3\.ArrayList和LinkedList遍历元素操作测试**

- for(;;)循环
- 迭代器迭代循环

<!-- -->

测试结果(花费时间)：

- ArrayList<LinkedList
- ArrayList≈LinkedList

<!-- -->

我们可以看到，LinkedList的for循环性能是最差的，而ArrayList的for循环性能是最好的。

这是因为LinkedList基于链表实现的，在使用for循环的时候，每一次for循环都会去遍历半个List，所以严重影响了遍历的效率；ArrayList则是基于数组实现的，并且实现了RandomAccess接口标志，意味着ArrayList可以实现快速随机访问，所以for循环效率非常高。

LinkedList的迭代循环遍历和ArrayList的迭代循环遍历性能相当，也不会太差，所以在遍历LinkedList时，我们要切忌使用for循环遍历。

## 思考题

我们通过一个使用for循环遍历删除操作ArrayList数组的例子，思考下ArrayList数组的删除操作应该注意的一些问题。

```
public static void main(String[] args)
    {
        ArrayList<String> list = new ArrayList<String>();
        list.add("a");
        list.add("a");
        list.add("b");
        list.add("b");
        list.add("c");
        list.add("c");
        remove(list);//删除指定的“b”元素

        for(int i=0; i<list.size(); i++)("c")()()(s : list) 
        {
            System.out.println("element : " + s)list.get(i)
        }
    }
```

从上面的代码来看，我定义了一个ArrayList数组，里面添加了一些元素，然后我通过remove删除指定的元素。请问以下两种写法，哪种是正确的？

写法1：

```
public static void remove(ArrayList<String> list) 
    {
        Iterator<String> it = list.iterator();
        
        while (it.hasNext()) {
            String str = it.next();
            
            if (str.equals("b")) {
                it.remove();
            }
        }

    }
```

写法2：

```
public static void remove(ArrayList<String> list) 
    {
        for (String s : list)
        {
            if (s.equals("b")) 
            {
                list.remove(s);
            }
        }
    }
```

期待在留言区看到你的答案。也欢迎你点击“请朋友读”，把今天的内容分享给身边的朋友，邀请他一起学习。

![unpreview](<https://static001.geekbang.org/resource/image/bb/67/bbe343640d6b708832c4133ec53ed967.jpg>)

## 精选留言(75)

- ![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,m_fill,h_34,w_34.jpeg)

  陆离

  置顶

  对于arraylist和linkedlist的性能以前一直都是人云亦云，大家都说是这样那就这样吧，我也从来没有自己去验证过，没想过因操作位置的不同差异还挺大。 当然这里面有一个前提，那就是arraylist的初始大小要足够大。 思考题是第一个是正确的，第二个虽然用的是foreach语法糖，遍历的时候用的也是迭代器遍历，但是在remove操作时使用的是原始数组list的remove，而不是迭代器的remove。 这样就会造成modCound != exceptedModeCount，进而抛出异常。

  作者回复: 陆离同学一直保持非常稳定的发挥，答案非常准确！

  2019-05-30

  **2

  **95

- ![img](https://static001.geekbang.org/account/avatar/00/12/91/50/e576a068.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  刘天若Warner

  置顶

  老师，为什么第二种就会抛出`ConcurrentModificationException`异常呢，我觉得第一种迭代器会抛这个异常啊

  作者回复: for(:)循环[这里指的不是for(;;)]是一个语法糖，这里会被解释为迭代器，在使用迭代器遍历时，ArrayList内部创建了一个内部迭代器iterator，在使用next()方法来取下一个元素时，会使用ArrayList里保存的一个用来记录List修改次数的变量modCount，与iterator保存了一个expectedModCount来表示期望的修改次数进行比较，如果不相等则会抛出异常； 而在在foreach循环中调用list中的remove()方法，会走到fastRemove()方法，该方法不是iterator中的方法，而是ArrayList中的方法，在该方法只做了modCount++，而没有同步到expectedModCount。 当再次遍历时，会先调用内部类iteator中的hasNext(),再调用next(),在调用next()方法时，会对modCount和expectedModCount进行比较，此时两者不一致，就抛出了ConcurrentModificationException异常。 所以关键是用ArrayList的remove还是iterator中的remove。

  2019-05-30

  **

  **67

- ![img](https://static001.geekbang.org/account/avatar/00/10/e9/52/aa3be800.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Loubobooo

  这一道我会。如果有看过阿里java规约就知道，在集合中进行remove操作时，不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator方式，如果并发操作，需要对 Iterator 对象加锁。 <!-- 规约第七条 -->

  作者回复: 👍

  2019-06-01

  **2

  **31

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/WtHCCMoLJ2DvzqQwPYZyj2RlN7eibTLMHDMTSO4xIKjfKR1Eh9L98AMkkZY7FmegWyGLahRQJ5ibPzeeFtfpeSow/132)

  脱缰的野马__

  老师您好，在我的认知里面，之所以数组遍历比链表要快，应该还有一个底层的原因，就是源于数组的实现是在内存当中是一块连续的内存空间，而链表所有元素可能分布在内存的不同位置，对于数组这种数据结构来说对CPU读是非常友好的，不管是CPU从内存读数据读到高速缓存还是线程从磁盘读数据到内存时，都不只是读取需要的那部分数据，而是读取相关联的某一块地址数据，这样的话对于在遍历数组的时候，在一定程度上提高了CPU高速缓存的命中率，减少了CPU访问内存的次数从而提高了效率，这是我结合计算机相关原理的角度考虑的一点。

  作者回复: 赞，这是计算机底层访问数组和链表的实现原理，数组因为存储地址是连续的，所以在每次访问某个元素的时候，会将某一块连续地址的数据读取到CPU缓存中，这也是数组查询快于链表的关键。

  2020-03-15

  **

  **26

- ![img](https://static001.geekbang.org/account/avatar/00/11/a6/10/3ff2e1a5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  皮皮

  第一种写法正确，第二种会报错，原因是上述两种写法都有用到list内部迭代器Iterator，而在迭代器内部有一个属性是exceptedmodcount，每次调用next和remove方法时会检查该值和list内部的modcount是否一致，不一致会报异常。问题中的第二种写法remove（e），会在每次调用时modcount++，虽然迭代器的remove方法也会调用list的这个remove（e）方法，但每次调用后还有一个exceptedmodcount=modcount操作，所以后续调用next时判断就不会报异常了。

  作者回复: 关键在用谁的remove方法。

  2019-05-30

  **

  **17

- ![img](https://static001.geekbang.org/account/avatar/00/0f/92/e4/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  TerryGoForIt

  老师您好，我比较好奇的是为什么 ArrayList 不像 HashMap 一样在扩容时需要一个负载因子呢？

  作者回复: HashMap有负载因子是既要考虑数组太短，因哈希冲突导致链表过长而导致查询性能下降，也考虑了数组过长，新增数据时性能下降。这个负载因子是综合了数组和链表两者的长度，不能太大也不能太小。而ArrayList不需要这种考虑。

  2019-05-31

  **

  **14

- ![img](https://static001.geekbang.org/account/avatar/00/10/db/b2/29b4f22b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  JasonZ

  linkedlist使用iterator比普通for循环效率高，是由于遍历次数少，这是为什么？有什么文档可以参考么？

  作者回复: 因为for循环需要遍历链表，每循环一次就需要遍历一次指定节点前的数据，源码如下: // 获取双向链表中指定位置的节点        private Entry<E> entry(int index) {            if (index < 0 || index >= size)                throw new IndexOutOfBoundsException("Index: "+index+                                                    ", Size: "+size);            Entry<E> e = header;            // 获取index处的节点。            // 若index < 双向链表长度的1/2,则从前先后查找;            // 否则，从后向前查找。            if (index < (size >> 1)) {                for (int i = 0; i <= index; i++)                    e = e.next;            } else {                for (int i = size; i > index; i--)                    e = e.previous;            }            return e;        } 而iterator在第一次拿到一个数据后，之后的循环中会使用Iterator中的next()方法采用的是顺序访问。

  2019-06-09

  **8

  **11

- ![img](https://static001.geekbang.org/account/avatar/00/0f/4d/bb/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  csyangchsh

  测试代码不严谨，建议使用JMH。

  作者回复: 厉害了，感谢建议。这里很多同学没有了解过JMH测试框架，所以没有使用。

  2019-05-30

  **

  **10

- ![img](https://static001.geekbang.org/account/avatar/00/12/f6/df/a576bfce.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  建国

  老师，您好，linkList查找元素通过分前后半段，每次查找都要遍历半个list，怎么就知道元素是出于前半段还是后半段的呢？

  作者回复: 这个是随机的，因为分配的内存地址不是连续的。

  2019-06-10

  **4

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/10/18/ee/a1ed60d1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ABC

  谢谢老师明白了，如果第二种写法换成for(;;)就会直接调用ArrayList的remove()方法就不会报错了。 在第二种写法里面用foreach相当于是使用ArrayList内部的Itr类进行遍历，但删除数据又是用的ArrayList里面的remove()方法。从而导致状态不一致，引发报错。

  2019-05-31

  **

  **6

- ![img](https://static001.geekbang.org/account/avatar/00/10/64/86/f5a9403a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  多襄丸

  modCount属于ArrayList expectedModCount属于Iterator 增强for循环  本质是iterator遍历 iterator循环  iterator遍历 增强for循环  调用list.remove() 不会修改到iterator的expectedModCount, 从而导致 迭代器的expectedModCount != ArrayList的modCound; 迭代器会抛出 concurrentModifiedException 而iterator遍历 的时候 用iterator. remove(); modCount 会被同步到expectedModCount中去，ArrayList的modCount == Iterator的exceptedModCount，所以不会抛出异常。  老师对其他同学的评论以及我的理解就是这样。

  作者回复: 赞

  2019-12-24

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/12/07/cf/b0d6fe74.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  L.

  老师，随机访问到底是什么意思？怎么个随机法？谢谢～

  作者回复: 这里指的是不需要通过遍历寻址，可以通过index直接访问到内存地址。

  2019-08-06

  **2

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/11/30/8a/b5ca7286.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  业余草

  请问：List<A> list = new ArrayList<>(); for(int i=0;i++;i<1000){ A a = new A(); list.add(a); } 和 和  这个  List<A> list = new ArrayList<>(); A a; for(int i=0;i++;i<1000){ a = new A(); list.add(a); } 效率上有差别吗？不说new ArrayList<>(); 初始化问题。单纯说创建对象这一块。谢谢！

  作者回复: 没啥区别的，可以实际操作试试

  2019-05-31

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/13/03/f7/3a493bec.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  老杨同志

  写法一正确，写法二会快速失败

  2019-05-30

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/10/90/5cb92311.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  麦兜布熊

  老师，什么场景会用到linkedlist呢？我好像只见过Arraylist的代码呢

  作者回复: 在做一些业务功能开发时，我们平常用的最多的是ArrayList，因为ArrayList就能满足我们的业务需求，单次填充列表以及单次全部读取列表。 到经常有删除/插入操作的情况下适合使用LinkedList，例如我们要写一个类似LRU算法的缓存功能，就可以用到LinkedList。

  2020-03-31

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/a0/cb/aab3b3e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张三丰

  first/last 方式可以在初始化 LinkedList 的时候节省 new 一个 Entry； 来回看老师的专栏，这是第三遍了，每次都会有新的理解，同时也有新的疑问产生，比如上边的那句话今天一直没有想明白。。。。麻烦老师详细解答一下。

  2019-11-06

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/03/c9/9a9d82ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Aaron_涛

  arrayList，for循环访问快是因为内存连续，可以整个缓存行读取进cpu缓存中，遍历下个的时候无需去内存中获取。并不是实现什么随机获取接口

  作者回复: 是的，是因为连续内存。在代码中，程序是不知道底层开辟的内存情况，所以需要一个类似序列化的接口标志，这个接口仅仅是一个标志，并不是实现。

  2019-07-17

  **2

  **2

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/U5tJTyH25kJA3eAtK0jKTmiaDGkFx4O1yOVjKnbnEQukTjDJCqhlKvLFaIZ6UVp3HcJK3GllMCRfDPU7wodslLQ/132)

  gavin

  老师好，怎么确定操作集合是从头部、中间、还是尾部操作的呢？

  作者回复: arraylist的add方法默认是从尾部操作，delete方法就是根据自己指定的位置来删除；linkedlist的add方法也是默认从尾部插入元素，delete方法也是根据指定的元素来删除。

  2019-06-11

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/0f/ae/d6/fbb8236d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  DebugDog

  写法一正确。 虽然都是调用了remove方法，但是两个remove方法是不同的。 写法二是有可能会报ConcurrentModificationException异常。 所以在ArrayList遍历删除元素时使用iterator方式或者普通的for循环。

  作者回复: 对的，使用普通循环也需要注意。

  2019-05-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/12/55/55/19ec7b0e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  mickle

  第二种不行吧，会报并发修改异常的

  2019-05-30

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/1f/4c/dd/c6035349.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  Bumblebee

  分析第二个方法会抛ConcurrentModificationException异常，我们这么入手应该好理解一点，将生成的class文件反编译等到如下代码：  public static void remove(List<String> list) {        Iterator var1 = list.iterator(); // ①         while(var1.hasNext()) {            String s = (String)var1.next(); // ②            if (s.equals("b")) {                list.remove(s); // ③            }        }     } 首先明确 1）modCound是ArrayList的成员变量继承自AbstractList； 2）expectedModCount是ArrayList内部迭代器Itr的成员变量； 执行到①处时会new一个迭代器对象，迭代器对象创建时会将list对象的成员变量modCound赋值给expectedModCount； 执行到②处时会检查list对象中的成员变量modCound与迭代器var1对象中的成员变量expectedModCount是否相等，不相等则抛ConcurrentModificationException异常； 执行到③处时对象list的成员变量modCound会被修改，但是迭代器var1对象中的成员变量expectedModCount不会被同步修复； 循环执行回到②处时就抛ConcurrentModificationException异常；

  2022-05-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/24/79/cf/211f0208.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郝希军

  第一个是正确的写法，第二种写法会有异常，因为直接把list的某个位置删除，但是for循环编辑器又不知道已经被删除，然后遍历到最后一个的时候肯定异常了，这题还是挺简单的

  2021-03-23

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/f0/07/92445721.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  李德强

  自己掉坑里了! ArrayList<String> list = new ArrayList<String>();         list.add("a");        list.add("b");        list.add("c");        list.add("d");        list.add("e");         for (String item :  list) {            if (item.equals("d")) {                list.remove(item);            }        } 这种移除唯一且倒数第二个的不会报错。 原因时判断hasNext时就返回了false，没有机会去校验modCount了。

  作者回复: 对的

  2019-08-19

  **4

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/15/ff/0a/12faa44e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓杰

  写法2不正确，使用for循环遍历元素的过程中，如果删除元素，由于modCount != expectedModCount，会抛出ConcurrentModificationException异常

  作者回复: 对的！

  2019-05-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/54/9a/76c0af70.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  每天晒白牙

  需要用迭代器方式删除 for循环遍历删除会抛并发修改异常

  作者回复: 是的，不要使用迭代器循环时用ArrayList的remove方法，具体分析可以看留言区。

  2019-05-30

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/1a/d8/5d/07dfb3b5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  杯莫停

  一般remove都是对某个元素进行操作，我们遍历删除的时候做个判断，命中了之后就停止循环，modCount++是先加后用，因此不会影响remove操作，这样就不存在抛异常的问题了。

  2022-06-28

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/5d/82/81b2ba91.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  keep_it_real

  一、新增 1.ArrayList： 是末尾添加elementData[size++] = input操作，所以在不需要扩容的情况下，add的时间复杂度是O(1)，而随机添加就需要对添加位置之后的所有元素进行reHash，重新分配空间，会造成性能损失，插入元素越靠前越严重。而且如果数组长度达到阈值还要扩容。 2.LinkedList  add默认末尾添加，而且随机添加只是改变前后node的后指针和前继指针的指向即可，由于它的特殊的结构我们还可以选择头部添加addFirst()和尾部添加linkLast() 由于链表在空间上不是连续的，而是靠指针连接，所以没必要重新分配空间，所以不管是末尾添加还是随机添加时间复杂度都是O（1） 二、删除 1.ArrayList： (1.)随机访因为下标的缘故时间复杂度是O（1）但删除就需要找到目标元素，就需要遍历数组，时间复杂度是O(n) (2)每次对元素修改操作都会modCount++，是为了判断是否存在并发操作，但在调用Itr的next()方法的时候会调用checkForComodification()对这个参数进行检测是否做了修改，是则抛异常。 (3)正确的remove姿势是遍历到目标删除元素删除后终止循环 2.LinkedList  可以头部/尾部添加，也可以头部尾部（removeFirst()/removeFirst()）删除，删除原理就是把头部和尾部节点都设为null方便GC回收，通俗点讲就是把要删除的元素择出来。 remove()有两种一种是删除目标元素，一种是根据下标删除（下标删除当链表为空是会抛异常） 三、遍历 1.ArrayList： 实现了RandomAccess接口标识遍历是根据index随机访问，性能相对要好 2.LinkedList  在使用 for 循环的时候，每一次 for 循环都会去遍历半个 List，所以严重影响了遍历的效率

  2022-05-13

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/25/4e/87/e78b0f25.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  Ywis

  remove方法既然不能用，那他存在的意义是什么呢？

  2022-04-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/e3/a1/d50f2188.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  BeerBear

  最后一道思考题，如果做法一种没有指定传入的List是什么类型，应该优先传入LinkedList，因为LinkedList在remove时性能更高。

  2022-03-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/28/91/91/428a27a3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  平民人之助

  迭代器会有一个数组下标，直接删除会导致空指针。如果走其他方法，可以搞成奶牛数组list，也能直接删除。

  2021-07-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/27/ac/29/8e75c8af.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  夏日炎炎

  Fast fail机制

  2021-06-02

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJeVG7hJqOKibpfMzpnoVIfwDzoqAlbGM7RdQZ1nMTxN2BLRHdZejA19nLEBrLJyUR4eiavjm7wYt8g/132)

  Zuul

  老师你好，有一个疑问，ArrayList是支持排序的，那如果 list里的元素是 【2,5,8,12,14】 如果用默认的add添加一个10，不是也会重排的吗？

  2021-04-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/81/d4/e92abeb4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jecy-8

  如果使用for(;;)，不会报错，但是如果有多个需要删除的元素时会漏掉或错删，因此需要在删除之后i需要重置一下位置 for (int i =0; i < list.size(); i ++) {            if (list.get(i).equals("b")) {                list.remove(list.get(i));                i --;            }        }

  2021-04-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/40/cc/9ff93968.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  🎄J

  3.LinkedList 新增元素：“之前的 last 对象的前指针指向新节点对象”描述不对吧，我看源码是： if (l == null)            first = newNode;        else            l.next = newNode; 所以应该是：如果添加的是第一个元素(l=null)，则将last引用指向新节点对象之前的 last 对象的下一个（next）指针指向新节点对象”

  2021-04-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ba/cd/1f91aa44.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  KK

  老师您好：怎么知道linkedlist采用for（：）循环调用哪个方法呢？

  2021-03-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/49/69/c0fcf4e8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  def

  原来forEach 语法糖写法也是用到了内部的Iterator去遍历的，这个地方之前还真的没有留意到...

  2021-03-01

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erHVh5AkbfJMK2xQlM0vow6UlsOAUQI47tia6SnQsQAujd0yGwRnOtibrEevkzEcdatzBdnCPnd8GyA/132)

  Geek_d2186f

  第一种可以，for-each不适用于add和remove等操作，导致list 元素变化。

  2020-09-15

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erHVh5AkbfJMK2xQlM0vow6UlsOAUQI47tia6SnQsQAujd0yGwRnOtibrEevkzEcdatzBdnCPnd8GyA/132)

  Geek_d2186f

  第一种方法和第三种方法都会导致list中元素变化，不适用于add和remove操作，只有第二种请求可以用于add和remove操作。

  2020-09-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/43/79/18073134.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  test

  总结： 1. 头插居多且元素不固定，使用LinkedList，其他情况使用ArrayList； 2. 不要用for来遍历LinkedList. 思考题: foreach使用迭代器,第一个使用了迭代器的remove，可行。第二个使用了list的remove，modCount会变，再次调用next时会报错。迭代器使用modCount实现fail-fast，以保证迭代过程中如果多线程改变了数据则直接跑错。

  2020-08-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1a/71/e7/39cf45cc.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  别过来你丑到我了(¬_¬)ﾉ

  public static void main(String[] args) {        List<String> var0 = new ArrayList<String>() {            {                this.add("1");                this.add("2");                this.add("3");                this.add("4");            }        };        Iterator var1 = var0.iterator();         while(var1.hasNext()) {            String var2 = (String)var1.next();            if (var2.equals("3")) {                // 此时这里并不会抛异常，但当 var2 = 1 or 2 or 4 时，则都会抛异常                var0.remove(var2);                // break;            }        }         System.out.println(var0);    } 背景：JDK1.8，以上代码为增强 for 循环 remove 元素反编译后得到的代码。 如代码所示，当元素值等于 3 时，remove掉改元素，按道理应该抛异常，但是没有，换成等于 1 或者 2 或者 4 的时候，都会抛异常，debug 的时候发现，等于 3 remove 掉元素之后，直接退出 while 循环了，但是按道理 hasNext 应该是 true 才对呀，有点困惑，求老师解答~

  2020-08-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/38/ca/7a151456.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  虞淇淇妈妈

  中间位置添加元素，ArryList的代码有问题啊

  2020-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1b/ab/76/38975d8c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  guohuibo

  public class LinkedList<E>    extends AbstractSequentialList<E>    implements List<E>, Deque<E>, Cloneable, java.io.Serializable 老师想问个问题，LinkedList继承了AbstractSequentialList,但是AbstractSequentialList已经实现了List接口，那么为啥LinkedList还要再次实现List接口了?有什么好处吗?

  2020-04-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/a0/cb/aab3b3e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张三丰

  只在数组末尾添加元素，那么 ArrayList 在大量新增元素的场景下，性能并不会变差，反而比其他 List 集合的性能要好。 这种情况比linklist性能好的依据是什么，毕竟linkedlist是双向链表，时间复杂度也是O(1)啊。

  2020-04-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/f5/57/ce10fb1b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  天天向上

  对于linkedList的遍历、remove、get方法，都会用到java.util.LinkedList#node方法，而老师讲到的删除位置的查找，遍历的问题，都是因为这个方法中的内容。

  2019-12-22

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/4d/5b/3ed7df22.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  思忆

  这两种都不正确，因为增强for循环的底层使用的是迭代器，再删除过程中，指针发生变化，所以会异常

  2019-11-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/15/03/c0fe1dbf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  考休

  思考题是初级开发者在使用remove的时候普遍遇到的问题，第一个使用迭代器成功，第二个会报错！

  2019-11-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/a0/cb/aab3b3e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张三丰

  看了老师如下的回答不是很明白，为什么数组过长新增数据的性能就下降了呢？因为数组越长hash碰撞的几率越小，那么性能越高才对。 老师您好，我比较好奇的是为什么 ArrayList 不像 HashMap 一样在扩容时需要一个负载因子呢？ 作者回复: HashMap有负载因子是既要考虑数组太短，因哈希冲突导致链表过长而导致查询性能下降，也考虑了数组过长，新增数据时性能下降。这个负载因子是综合了数组和链表两者的长度，不能太大也不能太小。而ArrayList不需要这种考虑。

  作者回复: 因为数组是连续性的存储结构，在新增数据时，如果不是想数组尾部顺序插入元素，则会涉及到其他数据的重新排列，性能就会差了。虽然查询性能很优秀，但在新增和删除数据时，性能就会变差了。 第二个问题已经回答了。

  2019-10-21

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/77/423345ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Sdylan

  2019.10.09 打卡 开眼界了 从源头分析 得到自己的结论 值得学习一把

  2019-10-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  课后思考及问题 请问老师你仔细研究过ArrayList和LinkedList之后，你觉得他们在性能上是否做到了极致？是否还存在优化的空间？

  作者回复: 性能应该对应具体场景来分析，同理，优化空间也是根据具体场景来进行优化。目前，大部分通用的源码都是兼容了大部分场景来实现的。

  2019-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/71/05/db554eba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  👽

  先说思考题   写法1正确，在ForEach遍历，或者fori遍历时，是禁止在内部删除数据的，会报错。 还有就是添加元素和删除元素的问题。这个测试不严谨。没有写明测试环境和测试代码。也没有测试数据。 个人测试的结果是，全都是ArrayList更快。个人表示，也出乎我的意料。最起码在我个人的理解里，链表插入数据明显会更快。 因为不需要考虑数据扩容的问题。

  作者回复: 具体的测试代码可以在github中查看，如果对测试结果有疑问，可以深入源码查看相应的实现，分析性能。

  2019-09-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/29/06/0b327738.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Gankki

  老师，您好。LinkedList 中 Node 是私有的静态内部类，除了防止内存泄露吗？还有其他的设计考虑吗？

  作者回复: 把老师考到了，现在数据结构这块很多都是以这种内部类来设计的，没有发现有特别的考虑。

  2019-07-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/60/81/cfc17578.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我戒酒了

  ArrayListTest 的这个测试方法笔误写错了吧	 public static void addFromMidTest(int DataNum) { 	ArrayList<String> list = new ArrayList<String>(DataNum); 	int i = 0; 	 	long timeStart = System.currentTimeMillis(); 	while (i < DataNum) { 		int temp = list.size(); 		list.add(temp/2+"aaavvv"); //正确写法list.add(temp/2, +"aaavvv"); 		i++; 	} 	long timeEnd = System.currentTimeMillis(); 		System.out.println("ArrayList从集合中间位置新增元素花费的时间" + (timeEnd - timeStart)); }

  作者回复: 对的，感谢细心的你提醒

  2019-07-02

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  吃胖了再减肥再吃再健身之谜之不...

  老师，我用的测试代码试了好多次，在“从集合尾部位置新增元素”这个场景下，我测试的结果是“ArrayList>LinkedList”，你代码里面的1000000，一百万次遍历时，会有少量的情况出现“ArrayList<LinkedList”，所以我讲遍历次数增加到10000000，一千万次遍历，测试结果如下 第一次： ArrayList从集合尾部位置新增元素花费的时间4690 LinkedList从集合尾部位置新增元素花费的时间2942 第二次： ArrayList从集合尾部位置新增元素花费的时间4655 LinkedList从集合尾部位置新增元素花费的时间2798 第三次： ArrayList从集合尾部位置新增元素花费的时间5126 LinkedList从集合尾部位置新增元素花费的时间2960 从这个场景看来，大数据量遍历的情况下，LinkedList新增数据比较快，不知道我这个验证的结果是否正确，期待老师指教，谢谢！

  作者回复: 老师没有在查询时同时新增测试，考虑的只是单个场景下的测试。ArrayList新增需要固定一个初始化大小，如果默认初始化大小，则会在新增时出现扩容的情况，这样性能反而会降低。

  2019-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/7f/15/7d670139.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  无忧

  老师，你好。ArrayList删除元素时将对应位置元素设置为null。代码注释说GC会回收，请问回收是什么时候触发呢。 如果元素为空就会被回收的话，AreayList在扩容时未使用的数组部分会不会也是空的被回收呢？ public E remove(int index) {        rangeCheck(index);         modCount++;        E oldValue = elementData(index);         int numMoved = size - index - 1;        if (numMoved > 0)            System.arraycopy(elementData, index+1, elementData, index,                             numMoved);        elementData[--size] = null; // clear to let GC do its work         return oldValue;    }

  2019-06-20

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/7f/15/7d670139.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  无忧

  老师你好。ArrayLists删除元素时 public E remove(int index) {        rangeCheck(index);         modCount++;        E oldValue = elementData(index);         int numMoved = size - index - 1;        if (numMoved > 0)            System.arraycopy(elementData, index+1, elementData, index,                             numMoved);        elementData[--size] = null; // clear to let GC do its work         return oldValue;    }

  2019-06-20

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/88/0a/31e6d5bb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  码德纽@宝

  我个人感觉arraylist和linkedlist性能最大的区别在于arraylist需要重新组排和扩容的开销。

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/71/05/db554eba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  👽

  源码粘贴不完。大概描述一下 方法1 最后是通过调用迭代器remove(int index)，是直接删除对应下标的元素。 方法2 最终是  如果b存在，那么调用list的remove（Object o），list的remove是删除指定对象equlse为true的第一个元素。 方法2 其实是转换为迭代器遍历，迭代器遍历的过程中使用了list的删除，导致迭代器下标越界。 顺便说下，自认为还有方法3 public static void remove3(ArrayList<String> list){    while (list.remove("b")){    }  } 这种方式删除元素 好像也是可以的。

  2019-06-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/db/74/d513e7d7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Bruce

  ”之前的 last 对象的前指针指向新节点对象。” 这句话  为什么是前指针呢 代码里写的是  l.next = newNode;

  2019-06-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/ac/15/935acedb.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  iusugar

  老师，可以在代码块多加一些注释吗？有些变量和方法不是很明白。原谅我比较菜...

  编辑回复: 收到，和老师说过了，这讲的会尽快加上，感谢你的建议！

  2019-06-06

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/05/7f/d35ab9a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  z.l

  第二种remove后加个return就不报错了吧

  作者回复: 是不会报错了，但剩余的业务就无法进行下去了

  2019-06-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/3b/36/2d61e080.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  行者

  所以好像遇到这种问题，不能一上来就定性，ArrayList就是擅长随即访问，LinkedList就是擅长增加、删除。要基于不同的情况去分析问题，会让你有新的发现。

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/f8/ba/14e05601.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  约书亚

  实际场景使用中linked list的效率应该还要更低吧？因为要考虑到内存结构紧凑的问题。array list在删除时候移动元素，很大可能是在一个cache line上操作，会很快，但linked list就未必了：写测试代码，linked list的元素总是连贯的。但实际使用场景一定是不连贯的。

  作者回复: 很赞成，需要根据不同的业务场景考虑，实际场景中的问题更复杂。

  2019-06-01

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTLbBZ9iaHfebHH4kOzFvxvs8Hx5iaUruAZvE8Dj4nia0mk4uxLc2rRUZD0ic9uKdxLibib0dGSaibL6NGRUg/132)

  清风拂面

  文稿关于从头部和尾部插入新元素所用时间那一块反了

  编辑回复: 同学你好～你是想说ArrayList从头插入元素和从尾部插入元素的速度问题吗？从头部插入，存在数组复制，从尾部不存在。所以从尾部插入的速度要比头部快。

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/27/1d/1cb36854.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小辉辉

  第一种是对的，第二种情况说的就是我之前干过的事情，而且当时出现有时报错，有时不报错。自己踩过的坑，记忆深刻😂😂😂

  作者回复: 希望在这个专栏后面更多的帮助到你，避免踩坑！

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/b0/35/44e5516e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  阳阳

  写法一是正确的，写法二会报错。

  作者回复: 答对了

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/5f/73/bb3dc468.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  拒绝

  老师，用foreach循环删除集合里倒数第二个元素，为什么不会报错？

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fe/44/3e3040ac.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员人生

  第一种是对的。第二种会报java.util.ConcurrentModificationException

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/16/65/c3/5324b326.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  半清醒

  收获颇多！！

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/48/75/02b4366a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  乘坐Tornado的线程魔法师

  请问下文中提到有关LinkedList在1.7之后的改动：first/last 方式可以在初始化 LinkedList 的时候节省 new 一个 Entry。原因是？作者可否指名下，感谢！

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/79/4b/740f91ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  -W.LI-

  老师好!这个我测试过。增加元素的话，就算不设置初始值，array也比linked快。随机，增删的话，增删元素靠近前部的话linked比array快。list总元素越大，linked比array快的比例越少。根据元素的位子不array的性能会比较稳定，linked差别挺大的。循环的话都用增强型for循环，基本看不出多大差别。测试结果和老师讲的符合。老师我有个问题:网上都说增强型for循环只是个语法糖底层还是迭代器遍历。课后习题第二种会报错的吧?

  作者回复: 如果存在大量扩容，我相信你的测试不准确。for(:)这种写法就是增强型，是一个语法糖。

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/8c/f7/a4de6f64.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  大卫

  为什么迭代器方式比for效率要高呢? 最后的问题是使用迭代方式去remove，直接for循环遍历remove可能会导致ConcurrentModifedException异常

  作者回复: 因为可以减少遍历的发生。

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/b3/c5/7fc124e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Liam

  为什么直接使用迭代器会比使用for循环遍历方式高呢，另外，如果是遍历的话，randomaccess并不重要吧，链表也能通过指针一次拿到下个元素的地址

  作者回复: 你是说LinkedList使用迭代器比for循环高吗，使用for需要遍历list，而使用iterator可以减少遍历。第二问题，你说的对的，这个接口只是一个标准罢了。

  2019-05-30

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEIvUlicgrWtibbDzwhLw5cQrDSy2JuE1mVvmXq11KQIwpLicgDuWfpp9asE0VCN6HhibPDWn7wBc2lfmA/132)

  a、

  删除arrayList应该倒序删除，这样就不会出现少删除的情况，因为arraylist每删除一个元素，arraylist都会把内部数组元素从新排列一次，第一个删除是对的，第二个会出现ConcurrentModificationException

  作者回复: 这个理解有点偏差，建议看留言区的正确解释。

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/1d/13/31ea1b0b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](05%20_%20ArrayList%E8%BF%98%E6%98%AFLinkedList%EF%BC%9F%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%BD%93%E6%80%A7%E8%83%BD%E5%B7%AE%E5%8D%83%E5%80%8D.resource/resize,w_14.png)

  峰

  第一个正确，最大感受时间复杂度这个东东要充分考虑场景，系数，io复杂度等等才能在实际工程有其意义。

  2019-05-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/18/ee/a1ed60d1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ABC

  思考: 我觉得第二种写法是对的，之前也一直是用的这种写法。而且，删除之后，数据的改变会反应在list上。 第一种写法没怎么用过，看写法，在删除元素之后。数据的变更并不会体现在list上。 如有不正确，请指正。谢谢。

  作者回复: 答案已经给出，希望能帮助你减少平时使用arraylist的潜在问题。

  2019-05-30
