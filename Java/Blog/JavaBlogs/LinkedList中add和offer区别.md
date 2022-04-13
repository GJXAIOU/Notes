# LinkedList中add和offer区别


offer属于 offer in interface Deque<E>

add 属于 add in interface Collection<E>

当队列为空时候，使用add方法会报错，而offer方法会返回false。

add是list的
offer是queue的
api里说：
add：Inserts the specified element at the specified position in this list
将指定的元素插入到list中指定的的位置。
offer：Inserts the specified element into this queue if it is possible to do so immediately without violating capacity restrictions.
如果在不违反容量限制的情况下，尽可能快的将指定的元素插入到queue中去

通地这个就可以看出区别了，哪个没有任何限制，哪个有限制
