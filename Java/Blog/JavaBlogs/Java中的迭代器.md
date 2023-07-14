# Java中的迭代器

　　迭代器是一种设计模式，它**是一个对象**，它可以遍历并选择序列中的对象，而开发人员不需要了解该序列的底层结构。迭代器通常被称为“轻量级”对象，因为创建它的代价小。

　　Java中的Iterator功能比较简单，并且**只能单向移动**：

　　(1) 使用方法iterator()要求容器返回一个Iterator。第一次调用Iterator的next()方法时，它返回序列的第一个元素。注意：iterator()方法是java.lang.Iterable接口,被Collection继承。

　　(2) 使用next()获得序列中的下一个元素。

　　(3) 使用hasNext()检查序列中是否还有元素。

　　(4) 使用remove()将迭代器新返回的元素删除。

　　Iterator是Java迭代器最简单的实现，为List设计的**ListIterator具有更多的功能，它可以从两个方向遍历List，也可以从List中插入和删除元素**。

迭代器应用：
 list l = new ArrayList();
 l.add("aa");
 l.add("bb");
 l.add("cc");
 for (Iterator iter = l.iterator(); iter.hasNext();) {
  String str = (String)iter.next();
  System.out.println(str);
 }
 /*迭代器用于while循环
 Iterator iter = l.iterator();
 while(iter.hasNext()){
  String str = (String) iter.next();
  System.out.println(str);
 }
 */
