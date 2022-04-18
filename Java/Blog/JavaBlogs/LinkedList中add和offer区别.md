# LinkedList中add和offer区别


offer 属于 offer in interface Deque<E>

add 属于 add in interface Collection<E>

当队列为空时候，使用 add 方法会报错，而 offer 方法会返回 false。

add 是 list 的
offer 是 queue 的
api 里说：
add：Inserts the specified element at the specified position in this list
将指定的元素插入到 list 中指定的的位置。
offer：Inserts the specified element into this queue if it is possible to do so immediately without violating capacity restrictions.
如果在不违反容量限制的情况下，尽可能快的将指定的元素插入到 queue 中去

通地这个就可以看出区别了，哪个没有任何限制，哪个有限制
