## 动图+源码，演示Java中常用数据结构执行过程及原理
[原文地址](https://mp.weixin.qq.com/s?__biz=MzA5MTMyNTI0Nw==&mid=2649790185&idx=1&sn=9499ecec9ac3a6fd6e629d7f42b7b08c&chksm=887a20b9bf0da9afe61e07e0ae59070cd803bacc8a11ada858952e1d49863b912635875b9b6a&mpshare=1&scene=1&srcid=&sharer_sharetime=1566873960883&sharer_shareid=9c5a3d8a04ee886a1949cf0328e23b60&key=830c786f292784e502d3eafe4db833514953659b87085aaa7053d9a1be520f5e520bf62625266786eec36cd7f00d9b00e2fcd85d441179d55577e8b1f0e341d0068ffda072f0a8de4d6ce0dd56c9f0f2&ascene=1&uin=MjY5MDU3MTgyOA%253D%253D&devicetype=Windows+10&version=62060834&lang=zh_CN&pass_ticket=fNjZortHO40n28xX9%252BKe8AW44q39U%252ByJjbxCKYNe1KFVMVOHt%252BEmFdNdbZJ82SGT)

最近在整理数据结构方面的知识, 系统化看了下Java中常用数据结构, 突发奇想用动画来绘制数据流转过程.

主要基于jdk8, 可能会有些特性与jdk7之前不相同, 例如LinkedList LinkedHashMap中的双向列表不再是回环的.

HashMap中的单链表是尾插, 而不是头插入等等, 后文不再赘叙这些差异, 本文目录结构如下:

![](https://mmbiz.qpic.cn/mmbiz_png/eQPyBffYbueecg88Kqx7gericIfNReBpzqfVessv7CtgibRJMvaBW03DKkLlGukQuTC0Pz29b5GgufiaKJdGttASA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* * *

## LinkedList

经典的双链表结构, 适用于乱序插入, 删除. 指定序列操作则性能不如ArrayList, 这也是其数据结构决定的.

**add(E) / addLast(E)**

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzRtCEIr9xcwpAg1j8vG0eodrR89XJvJcKDYgRS1tUqh52Wd6FBD4N7w/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**add(index, E)**

这边有个小的优化, 他会先判断index是靠近队头还是队尾, 来确定从哪个方向遍历链入.

```
if (index < (size >> 1)) {            Node<E> x = first;            for (int i = 0; i < index; i++)                x = x.next;            return x;        } else {            Node<E> x = last;            for (int i = size - 1; i > index; i--)                x = x.prev;            return x;        }
```

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzuMmd8dxu4MFy4fGh3tzV7KOoqsT7Y7lymbzbIp2SU5lEAcmB2qNcHQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**靠队尾**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzhzYMCxWuXRcibHIcrDdWM5o5DH92p0YkLYyrF1aNibTQXk9EdAw5MtUQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

**get(index)**

也是会先判断index, 不过性能依然不好, 这也是为什么不推荐用for(int i = 0; i < lengh; i++)的方式遍历linkedlist, 而是使用iterator的方式遍历.

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpz6Z7vSYcIPyCxyoCEGxw825T1k5sVOzULgJxrIAvMOCp0FrVKyLqlKQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzIwxzpqdn562f6kOJ1hfAYgO7LKBIticQAQSWYyFdyhVWxl4zU2xg1qA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**remove(E)**

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzPUWN0djCuHWOYfZiaopzsH1Fia2T8uFibxzm8ib17ibbibvYEASGs58ZiaSvA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

* * *

## ArrayList

底层就是一个数组, 因此按序查找快, 乱序插入, 删除因为涉及到后面元素移位所以性能慢.

**add(index, E)**

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzht01LgeJWAWyqMl45SBtBicktmajY8f2QnDt8ibdy8D6VqbmulGVkiarQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**扩容**

一般默认容量是10, 扩容后, 会length*1.5.

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzTnP21gsv7lVT6d6B9azVictmCYRgMsyHbcxaJj0Zgc9bxO2YTKOODuw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**remove(E)**

循环遍历数组, 判断E是否equals当前元素, 删除性能不如LinkedList.

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzPp9ODFIGAF1a6sKlO3slX0VXlDGG8y6qFOA8g5ce34UPKF1Jemib36A/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

* * *

## Stack

经典的数据结构, 底层也是数组, 继承自Vector, 先进后出FILO, 默认new Stack()容量为10, 超出自动扩容.

**push(E)**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzx2gudUsjZguVtvL8baicDicUgSP7Q7vdQ5Pn6Dibft4PbAMicYDPp8BMvw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

**pop()**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzTp7crXsJW3OvBicaX9BbILJLkyyyhFYlT66pTA2BwPjQQWLGJz1I5xw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

* * *

## 后缀表达式

Stack的一个典型应用就是计算表达式如 9 + (3 - 1) * 3 + 10 / 2, 计算机将中缀表达式转为后缀表达式, 再对后缀表达式进行计算.

### 中缀转后缀

*   数字直接输出

*   栈为空时，遇到运算符，直接入栈

*   遇到左括号, 将其入栈

*   遇到右括号, 执行出栈操作，并将出栈的元素输出，直到弹出栈的是左括号，左括号不输出。

*   遇到运算符(加减乘除)：弹出所有优先级大于或者等于该运算符的栈顶元素，然后将该运算符入栈

*   最终将栈中的元素依次出栈，输出。

### ![](https://mmbiz.qpic.cn/mmbiz_gif/XaklVibwUKn5XmeIJ2TrXaKy1nOpDVgafRKzcSkm0GSRZYyneQTkVLmGg2HNz1HkMrGmUjNEsfdjKaOTSjBJdUA/640?tp=webp&wxfrom=5&wx_lazy=1)

### 计算后缀表达

*   遇到数字时，将数字压入堆栈

*   遇到运算符时，弹出栈顶的两个数，用运算符对它们做相应的计算, 并将结果入栈

*   重复上述过程直到表达式最右端

*   运算得出的值即为表达式的结果

![](https://mmbiz.qpic.cn/mmbiz_gif/XaklVibwUKn5XmeIJ2TrXaKy1nOpDVgafyP080icsRJsDtIlicTnfPTYBubtGRB4PSX8ib5LjJ6OGNibibhicNBFNMnIw/640?tp=webp&wxfrom=5&wx_lazy=1)

* * *

## 队列

与Stack的区别在于, Stack的删除与添加都在队尾进行, 而Queue删除在队头, 添加在队尾.

**ArrayBlockingQueue**

生产消费者中常用的阻塞有界队列, FIFO.

**put(E)**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzuK1CrppWWQTD5GkyicZkuozt6n3VjATbAA9eSmX3niaxTMmuEKnm8RaA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

**put(E) 队列满了**

```
final ReentrantLock lock = this.lock;        lock.lockInterruptibly();        try {            while (count == items.length)                notFull.await();            enqueue(e);        } finally {            lock.unlock();        }
```

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzHfz4pR5XIfoGSR7Sf2Mk8t2mXuwvZsXogicr2H3ic5OTvAYicEhXgtVJQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

**take()**

当元素被取出后, 并没有对数组后面的元素位移, 而是更新takeIndex来指向下一个元素.

takeIndex是一个环形的增长, 当移动到队列尾部时, 会指向0, 再次循环.

```
private E dequeue() {        // assert lock.getHoldCount() == 1;        // assert items[takeIndex] != null;        final Object[] items = this.items;        @SuppressWarnings("unchecked")        E x = (E) items[takeIndex];        items[takeIndex] = null;        if (++takeIndex == items.length)            takeIndex = 0;        count--;        if (itrs != null)            itrs.elementDequeued();        notFull.signal();        return x;    }
```

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzm1yic97HyvwkvHgNUAq1YQNdWv5RUTC7nD74de917wDpG42waD6tfcA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

* * *

## HashMap

最常用的哈希表, 面试的童鞋必备知识了, 内部通过数组 + 单链表的方式实现. jdk8中引入了红黑树对长度 > 8的链表进行优化, 我们另外篇幅再讲.

**put(K, V)**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpz8e1Tsaib3RooISJ0SgIOoyMbdfnErNPRRbzNdgs4eyWuybcEicebGJyQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

**put(K, V) 相同hash值**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzj9iaMMC3dQEgcD7SpOql1GUv8wZsGKZTIngIBBsRRyoRFJOIHUUqDwQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

### resize 动态扩容

当map中元素超出设定的阈值后, 会进行resize (length * 2)操作, 扩容过程中对元素一通操作, 并放置到新的位置.

**具体操作如下:**

*   在jdk7中对所有元素直接rehash, 并放到新的位置.

*   在jdk8中判断元素原hash值新增的bit位是0还是1, 0则索引不变, 1则索引变成"原索引 + oldTable.length".

```
//定义两条链    //原来的hash值新增的bit为0的链，头部和尾部    Node<K,V> loHead = null, loTail = null;    //原来的hash值新增的bit为1的链，头部和尾部    Node<K,V> hiHead = null, hiTail = null;    Node<K,V> next;    //循环遍历出链条链    do {        next = e.next;        if ((e.hash & oldCap) == 0) {            if (loTail == null)                loHead = e;            else                loTail.next = e;            loTail = e;        }        else {            if (hiTail == null)                hiHead = e;            else                hiTail.next = e;            hiTail = e;        }    } while ((e = next) != null);    //扩容前后位置不变的链    if (loTail != null) {        loTail.next = null;        newTab[j] = loHead;    }    //扩容后位置加上原数组长度的链    if (hiTail != null) {        hiTail.next = null;        newTab[j + oldCap] = hiHead;    }
```

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpz4ea7sh8zrBEWodNsl9mZ9yIl1RztA1RF1yrddQa1Vb7dvMUGRjGeCw/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

* * *

## LinkedHashMap

继承自HashMap, 底层额外维护了一个双向链表来维持数据有序. 可以通过设置accessOrder来实现FIFO(插入有序)或者LRU(访问有序)缓存.

**put(K, V)**

**![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpz09rsBPpGFQ5ym38YhPoOOrcYmjaY1ko7P9iaRfLORn1r1hicN1rjicXPQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)**

**get(K)**

accessOrder为false的时候, 直接返回元素就行了, 不需要调整位置.

accessOrder为true的时候, 需要将最近访问的元素, 放置到队尾.

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzqhuXLlFm66waokOg3ssC1GejticyrQoicWZwHkSTNBTLUDzpsxrQuNBA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**removeEldestEntry 删除最老的元素**

![](https://mmbiz.qpic.cn/mmbiz_gif/eQPyBffYbueecg88Kqx7gericIfNReBpzEYlwicbvDrv1HLNVB5BmMicE8J1GIqMwoJllNoqRRctyC6Ugsiby4NtmA/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)