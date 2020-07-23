# Java迭代器Iterator详解

为了方便的处理集合中的元素,Java中出现了一个对象,该对象提供了一些方法**专门处理集合中的元素**.例如删除和获取集合中的元素.该对象就叫做迭代器(Iterator).

对 Collection 进行迭代的类，称其为迭代器。还是面向对象的思想，专业对象做专业的事情，迭代器就是专门取出集合元素的对象。但是该对象比较特殊，不能直接创建对象（通过new），该对象是以内部类的形式存在于每个集合类的内部。

如何获取迭代器？Collection接口中定义了获取集合类迭代器的方法（iterator（）），所以所有的Collection体系集合都可以获取自身的迭代器。



## 1.Iterable

正是由于每一个容器都有取出元素的功能。这些功能定义都一样，只不过实现的具体方式不同（因为每一个容器的数据结构不一样）所以对共性的取出功能进行了抽取，从而出现了Iterator接口。而每一个容器都在其内部对该接口进行了内部类的实现。也就是将取出方式的细节进行封装。


Jdk1.5之后添加的新接口, Collection的父接口. 实现了Iterable的类就是可迭代的.并且支持增强for循环。该接口只有一个方法即获取迭代器的方法iterator（）可以获取每个容器自身的迭代器Iterator。（Collection）集合容器都需要获取迭代器（Iterator）于是在5.0后又进行了抽取将获取容器迭代器的iterator（）方法放入到了Iterable接口中。Collection接口进程了Iterable，所以Collection体系都具备获取自身迭代器的方法，只不过每个子类集合都进行了重写（因为数据结构不同）


## 2.Iterator 

iterator() 返回该集合的迭代器对象


该类主要用于遍历集合对象，该类描述了遍历集合的常见方法

1：java.lang. Itreable  

- Itreable      接口 实现该接口可以使用增强for循环
  - Collection 描述所有集合共性的接口
     - List接口     可以有重复元素的集合
     - Set接口     不可以有重复元素的集合

public interface Iterable<T>

Itreable   该接口仅有一个方法，用于返回集合迭代器对象。

`Iterator<T> iterator()` 返回集合的迭代器对象

 

Iterator接口定义的方法

```language

Itreator	该接口是集合的迭代器接口类，定义了常见的迭代方法
	1：boolean hasNext() 
		判断集合中是否有元素，如果有元素可以迭代，就返回true。
	2： E next()  
		返回迭代的下一个元素，注意： 如果没有下一个元素时，调用next元素会抛出NoSuchElementException
	3： void remove()
		从迭代器指向的集合中移除迭代器返回的最后一个元素（可选操作）。
```
思考：为什么next方法的返回类型是Object的呢？ 

为了可以接收任意类型的对象,那么返回的时候,不知道是什么类型的就定义为object

## 3.迭代器的遍历

第一种方式：while循环
```java
public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
		Iterator it = list.iterator();
		while (it.hasNext()) {
			String next = (String) it.next();
			System.out.println(next);
		}
	}
```
第二种方式:for循环
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		for (Iterator it = list.iterator(); it.hasNext();) {
             //迭代器的next方法返回值类型是Object，所以要记得类型转换。
			String next = (String) it.next();
			System.out.println(next);
		}
	}
}

```

需要取出所有元素时，可以通过循环，java 建议使用for 循环。因为可以对内存进行一下优化。

第三种方式：使用迭代器清空集合
```java
public class Demo1 {
	public static void main(String[] args) {
		Collection coll = new ArrayList();
		coll.add("aaa");
		coll.add("bbb");
		coll.add("ccc");
		coll.add("ddd");
		System.out.println(coll);
		Iterator it = coll.iterator();
		while (it.hasNext()) {
			it.next();
			it.remove();
		}
		System.out.println(coll);
	}
}

```

需要注意的细节如下：
细节一：

如果迭代器的指针已经指向了集合的末尾，那么如果再调用next()会返回NoSuchElementException异常
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		Iterator it = list.iterator();
		while (it.hasNext()) {
			String next = (String) it.next();
			System.out.println(next);
		}
		// 迭代器的指针已经指向了集合的末尾
		// String next = (String) it.next();
		// java.util.NoSuchElementException
	}
}

```
细节二：

 如果调用remove之前没有调用next是不合法的，会抛出IllegalStateException
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		Iterator it = list.iterator();
		while (it.hasNext()) {
			// 调用remove之前没有调用next是不合法的
			// it.remove();
			// java.lang.IllegalStateException
			String next = (String) it.next();
			System.out.println(next);
		}
 
	}
}
```



## 4.迭代器的原理
查看ArrayList源码
```java
private class Itr implements Iterator<E> {
 
		int cursor = 0;
		int lastRet = -1;
		int expectedModCount = modCount;
 
		public boolean hasNext() {
			return cursor != size();
		}
 
		public E next() {
			checkForComodification();
			try {
				E next = get(cursor);
				lastRet = cursor++;
				return next;
			} catch (IndexOutOfBoundsException e) {
				checkForComodification();
				throw new NoSuchElementException();
			}
		}
 
		public void remove() {
			if (lastRet == -1)
				throw new IllegalStateException();
			checkForComodification();
 
			try {
				AbstractList.this.remove(lastRet);
				if (lastRet < cursor)
					cursor--;
				lastRet = -1;
				expectedModCount = modCount;
			} catch (IndexOutOfBoundsException e) {
				throw new ConcurrentModificationException();
			}
		}
 
		
	}
	
```

注意：
1.在对集合进行迭代过程中，不允许出现迭代器以外的对元素的操作，因为这样会产生安全隐患，java会抛出异常并发修改异常（ConcurrentModificationException），普通迭代器只支持在迭代过程中的删除动作。

2.ConcurrentModificationException: 当一个集合在循环中即使用引用变量操作集合又使用迭代器操作集合对象， 会抛出该异常。
```java
public class Demo1 {
	public static void main(String[] args) {
		Collection coll = new ArrayList();
		coll.add("aaa");
		coll.add("bbb");
		coll.add("ccc");
		coll.add("ddd");
		System.out.println(coll);
		Iterator it = coll.iterator();
		while (it.hasNext()) {
			it.next();
			it.remove();
			coll.add("abc"); // 出现了迭代器以外的对元素的操作
		}
		System.out.println(coll);
	}
}

```
如果是List集合，想要在迭代中操作元素可以使用List集合的特有迭代器ListIterator，该迭代器支持在迭代过程中，添加元素和修改元素。

## 5.List特有的迭代器ListIterator

public interface ListIterator extends Iterator

ListIterator<E> listIterator()

-  Iterator
  - hasNext()
  - next()
  - remove()
    -  ListIterator Iterator子接口 List专属的迭代器
	   - add(E e)    将指定的元素插入列表（可选操作）。该元素直接插入到 next 返回的下一个元素的前面（如果有）
	   - void set(E o)   用指定元素替换 next 或 previous 返回的最后一个元素
	   - hasPrevious()    逆向遍历列表，列表迭代器有多个元素，则返回 true。
	   - previous()       返回列表中的前一个元素。
Iterator在迭代时，只能对元素进行获取(next())和删除(remove())的操作。

对于 Iterator 的子接口ListIterator 在迭代list 集合时，还可以对元素进行添加

(add(obj))，修改set(obj)的操作。
```java
public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
         // 获取List专属的迭代器
		ListIterator lit = list.listIterator();
 
		while (lit.hasNext()) {
			String next = (String) lit.next();
			System.out.println(next);
		}
 
	}
}
```
倒序遍历
```java


public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
        // 获取List专属的迭代器
		ListIterator lit = list.listIterator();
		while (lit.hasNext()) {
			String next = (String) lit.next();
			System.out.println(next);
		}
		System.out.println("***************");
		while (lit.hasPrevious()) {
			String next = (String) lit.previous();
			System.out.println(next);
		}
 
	}
}

```


Set方法：用指定元素替换 next 或 previous 返回的最后一个元素
```java


public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		ListIterator lit = list.listIterator();
		lit.next(); // 计算机网络
		lit.next(); // 现代操作系统
		System.out.println(lit.next()); // java编程思想
		//用指定元素替换 next 或 previous 返回的最后一个元素
		lit.set("平凡的世界");// 将java编程思想替换为平凡的世界
		System.out.println(list);
 
	}
}

```
add方法将指定的元素插入列表，该元素直接插入到 next 返回的元素的后面
```java


public class Demo2 {
	public static void main(String[] args) {
		ArrayList list = new ArrayList();
		// 增加：add() 将指定对象存储到容器中
		list.add("计算机网络");
		list.add("现代操作系统");
		list.add("java编程思想");
		list.add("java核心技术");
		list.add("java语言程序设计");
		System.out.println(list);
 
		ListIterator lit = list.listIterator();
		lit.next(); // 计算机网络
		lit.next(); // 现代操作系统
		System.out.println(lit.next()); // java编程思想
		// 将指定的元素插入列表，该元素直接插入到 next 返回的元素的后
		lit.add("平凡的世界");// 在java编程思想后添加平凡的世界
		System.out.println(list);
 
	}
}
```
