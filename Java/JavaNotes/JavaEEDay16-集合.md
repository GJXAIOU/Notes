---
tags : 
- java基础

flag: yellow
---
@toc

# JavaEEDay16-集合

## 一、背景：
- 数组：用于保存大量数据采取的方式；
  - 可以保存的数据类型：
    - 8 大基本数据类型 byte/short/int /long/float/double/char/boolean
    - 自定义类对象：例如 Student Player
    - Java 中的类对象
  - 局限性：
    - 只能保存一种数据类型的数据；
    - 数组的元素个数创建时是确定的，无法直接对元素个数进行修改；
    - 数组里面保存的元素内容空间是连续的；

- 问题：==如果需要保存任意类型的一个数组==，所有的数据类型都可以在这个数组中保存；
- 答案：==使用 Object 数据类型==，因为 Object 是 Java 中所有的类的直接父类或者间接父类 基类/根类；

代码示例：
```java
package study;
import java.util.Arrays;

public class Demo1 {
	public static void main(String[] args) {
		//创建一个数据类型为Object数组，元素个数为5
		
		Object[] arr = new Object[5];
		arr[0] = "hello";
		arr[1] = 23;
		arr[2] = 2.3f;
		arr[3] = new Demo1();
		arr[4] = true;
		
	System.out.println(Arrays.toString(arr));
	}
}

```
程序输出结果：
`[hello, 23, 2.3, study.Demo1@21588809, true]`

---

## 二、集合

### （一）概念
集合就是存储对象的【容器】
主要介绍：Collection、List、Set 接口
- Collection： 集合的总【接口】，规定了很多方法，要求所有【遵从】该接口的类，全部实现；
  - List： 接口，如果【遵循】了 List 接口，要求实现所有 List 中的方法，具有 ==List 集合的特征：**有序、可重复**==
  - Set：接口，如果【遵循】了 Set 接口，要求实现所有 Set 中的方法，具有 ==Set 集合的特征：**无序，不可重复**==

首先从 Collection 开始学习，学习之后基本上 List 和 Set 就是在此基础上添加操作
### （二）Collction 基本方法：
- 增：
  - add(Object o); //添加一个元素，任意类型的
  - addAll(Collection c); //添加另一个集合
- 删：
  - clear(); //清空整个集合
  - remove(Object o); //删除该集合中指定的元素
  - removeAll(Colletcion c); //删除两个集合中的交集
  - retainAll(Collection c); //保留两个集合中的交集，删除其他元素
- 查：
  - size(); //获取当前集合有效元素的个数
  - toArray(); //把当前集合中所有的元素转换成为 Object 类型的数组返回
  
```java
package study;

import java.util.ArrayList; //注意导包：java.util.ArrayList包
import java.util.Arrays;
import java.util.Collection;

public class Demo2 {
	public static void main(String[] args) {
		//Collection 是一个接口，而接口是没有自己的类对象的，但是可以指向【遵从】该接口的类对象
		//这里借助于常用集合ArrayList来完成；
		Collection collection  = new ArrayList();
		
		//测试添加方法： add(Object 0);
		collection.add("天气变热了");
		collection.add("明天30度啦");
		collection.add("测试");
		System.out.println(collection);
		
		//测试添加方法：addAll(Collection c);
		Collection collection2 = new ArrayList();
		collection2.add("测试一");
		collection2.add("好热");
		
		collection.addAll(collection2);
		System.out.println(collection);
		
		
		//测试clear() : 清空集合中所有的元素
		System.out.println(collection);
		collection.clear();
		System.out.println(collection);
		
		//测试remove(Object o)
		collection.remove("测试");
		System.out.println(collection);
		
		//测试removeAll
		collection.removeAll(collection2);
		System.out.println(collection);
		
		//测试retainAll(Collection c)
		collection.retainAll(collection2);
		System.err.println(collection);
		
		//测试size()
		System.out.println(collection.size());
		
		//测试Arrays.toString
		System.out.println(Arrays.toString(collection.toArray()));		
	}	
}

```

### （三）Collection 其他方法：判断方法
- isEmpty(); //是否为空
- contains(Object o); //是否包含当前元素
- containsAll(Colllection c); //是否包含指定集合里面的所有元素
- equals(Object o); //判断是否相等

代码示例：
```java
package study;

import java.util.ArrayList;
import java.util.Collection;

public class Demo3 {
	public static void main(String[] args) {
		Collection collection = new ArrayList();
		collection.add("A");
		collection.add("B");
		collection.add("C");
		collection.add("D");
		
		Collection collection2 = new ArrayList();
		collection2.add("A");
		collection2.add("B");
		collection2.add("D");
		collection2.add("C");
		
		//测试equal()方法 ； //比较这两个集合之间的元素是否完全相同，放入的位置也相同；
		collection.equals(collection2);  //因为顺序不同，结果为false
		
		//测试isEmpty方法
		collection.isEmpty(); //结果为false
		
		//测试 contains(Object o);
		System.out.println(collection.contains("A"));
		System.out.println(collection.contains("E"));
		
		//测试containAll(Collection c); //判断传入的集合是不是当前集合的子集
		System.out.println(collection.containsAll(collection2));
	}

}

```



### （四）collection 中判断方法的重写

- contains, containAll,equals 方法；
  - 发现：
    - Java 语言中默认判断两个对象是否相同的方式是：判断这两个对象的地址是否相同；
    - 在这里 student1 对象和 new Student（1, “成龙”）;是两个完全不同的对象，因此判断结果为 false；
  - 问题：
    - 因为上面两个对象里面保存的数据其实是一样的，也是符合业务逻辑的，或者是符合生活逻辑的，因此想实现在符合语法的前提下，也符合生活逻辑（即当两个对象中保存的数据一致时，判为相等）；
  - 解决方法：
    - 重写 equals 和 hashCode 方法；
    - 默认情况下：
      - **hashCode 方法在系统默认情况下，是当前类的对象在内存中地址的十进制数**；（地址默认为 16 进制显示，hashCode 为 10 进制显示）
      - equals 方法是两个对象相互比较的法则；

代码示例：
```java
package study;

import java.util.ArrayList;
import java.util.Collection;

class Student{
	private int  id;
	private String name;

	public Student() {	
		
	}
	
	public  Student(int id, String name) {
		this.id = id;
		this.name = name;
	}
	

	public void setId(int id) {
		this.id = id;
	}
	

	public int getId() {
		return id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
	
	@Override
	//描述当前类对象，当通过打印方法的时候自动调用
	public String toString() {
		return "[ ID : " + id + "    Name : " + name + "]";
	}
	
	//重写equals方法和hashCode方法
	@Override
	public boolean equals(Object obj) {
		//这里的equals方法是Student类重写的方法，当集合调用contains、containsAll、equals方法的时候
		//都会调用这里的Student类中的equals方法进行比较，比较的对象是Student对象；
		System.out.println("Student的equals方法");
		
		//原来的equals方法不符合生活逻辑，仅仅是判断两个对象的地址是否相同，不判断里面的内容是否一致，
		//重写改变为判断对象中的数据是否一致
		
		//1.首先进行强制类型转换
		Student student = (Student)obj;
		System.out.println(this.name + " 和 " + student.name + "进行比较");
		
		//这里的this.name.equals(student.name)中的equals方法是调用String类型的equals方法，用于判断两字符串是否相等；
		return this.id == student.id && this.name.equals(student.name);
	}
	
	@Override
	public int hashCode() {
		System.out.println("Student 的 HashCode 方法");
		//如果重写了equals方法，同时也要重写hashCode方法
		//因为hashCode值要确定【唯一性】，只要满足自己逻辑就OK，这里认为id是唯一的
		return this.id;
	}	
}


public class Demo1 {
	public static void main(String[] args) {
		Collection collection  =  new ArrayList();
		Student student1 = new Student(1,"张三");
		Student student2 = new Student(2,"李四"	);
		Student student3 = new Student(3,"王五");
		
		collection.add(student1);
		collection.add(student2);
		collection.add(student3);
		
		System.out.println(collection);
		
		boolean ret = collection.contains(new Student(1, "张三")); //
		System.out.println(ret);
	} 
}

```
程序运行结果：
```java
[[ ID : 1    Name : 张三], [ ID : 2    Name : 李四], [ ID : 3    Name : 王五]]
Student的equals方法
张三 和 张三进行比较
true
```


---

## 三、 迭代器 Itreator

- 常用方法：

返回值类型 | 方法名  | 含义
---| -- |---
boolean | hasNext();|  //判断当前迭代器是否有下一个元素
Object  | next();  | //获取当前迭代器指向的元素，并且获取之后，指向下一个元素；
void | remove(); | //删除当前迭代器通过next获取到的对象


注意: 在==通过迭代器调用remove方法时候，之前必须调用过next方法==，否则会报异常：java.lang.IllegalAtateException


代码示例
```java
package lianxi ;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;


/**
 * class detail:迭代器
 * @author GJXAIOU
 * @since  2019年6月28日
 */
public class Demo2 {
	public static void main(String[] args) {
		Collection collection = new ArrayList();
		collection.add("张三");
		collection.add("李四");
		collection.add("王五");
		
		//第一种遍历方式：将集合转化为数组，然后按照数组的方式进行遍历
		//缺点：需要拷贝一份完整的集合数据，如果集合的数据过多，会造成内存的极大浪费，甚至超过内存的最大值
		Object[] array = collection.toArray(); 
		//toArray()是转换为Object类型的数组
		
		for (int i = 0; i < array.length; i++) {
			System.out.println(array[i]);
		}
		
		System.out.println("-------------------------------------");
		
		//第二种方式：使用迭代器
		
		//上述方法使用示例
		Iterator iterator = collection.iterator(); //返回当前集合的一个迭代器
		System.out.println("当前元素有没有下一个元素：" + iterator.hasNext());
		System.out.println("当前迭代器指向的元素：" + iterator.next());
		System.out.println("当前迭代器指向的元素：" + iterator.next());
		System.out.println("调用了一下删除的方法");
		iterator.remove(); //这里删除的是上一个指向的next元素；
		System.out.println(collection);
		
		System.out.println("---------------------------------------");
		
		//使用迭代器，借助hasNext和next方法，完成对整个集合的遍历
		while (iterator.hasNext()) {
			System.out.println("迭代器操作：" + iterator.next());
			//将整个集合清空
			//iterator.remove();
		}
		
	}
}
```

程序运行结果：
```java
张三
李四
王五
-------------------------------------
当前元素有没有下一个元素：true
当前迭代器指向的元素：张三
当前迭代器指向的元素：李四
调用了一下删除的方法
[张三, 王五]
---------------------------------------
//注：这里因为上面迭代器已经指向了李四，因此结果为
迭代器操作：王五

//如果将上面的语句都注释掉的话，结果应该为：
迭代器操作：张三
迭代器操作：李四
迭代器操作：王五
```

---

## 四、共享资源问题

针对同一个资源，同时两个主体在使用，容易产生冲突；
代码示例：集合对象和迭代器同时处理集合中的元素，造成冲突；
```Java
package study;

import java.util.ArrayList;
import java.util.Collection;
import java.util.Iterator;

/**
 * class detail:共享资源处理问题
 * @author GJXAIOU
 * @since  2019年6月28日
 */
public class ShareSources {
	public static void main(String[] args) {
		Collection collection = new ArrayList();
		collection.add("张三");
		collection.add("李四");
		collection.add("王五");
		
		Iterator iterator = collection.iterator();
		
		//下面代码会抛出异常： java.util.ConcurrentModificationException
		/*
		 * 这里存在两种操作集合数据的方式：
		 第一种方式：集合的对象collection；第二种方式：集合的迭代器iterator
		 * 这里两种操作都有查看和删除的权限，但是一方操作的同时，
		 另一方也在操作，会造成共享资源问题；
		 * 
		 * 解决方法：
		 *  	1.使用迭代器操作放弃集合对象操作
		 *  	2.使用集合对象操作放弃迭代器操作
		 */
		while (iterator.hasNext()) {
					System.out.println(iterator.next());
					collection.remove("李四");	
		}
	}
}

```

---

## 五、List
首先 Collection 接口中的所有方法在这里仍然可以使用，下面是 List 接口中【特有的】方法：
- 添加：
  - add(int index, Object o); //指定位置上放入元素
  - addAll(int index, Collection c); //在指定位置上添加一个集合
- 获取：
  - Object    get(int index); //获取下标的元素
  - int     indexOf(Object o); //获取某个元素的下标位置
  - int    lastIndexOf(Object o); //找到指定元素最后一个出现在集合中的位置
  - List    subList(int fromIndex, int toIndex); //获取子 List 集合
- 修改：
  - set(int index, Object o); //设置指定下标上的元素
- 迭代：
  - ListIterator();

方法的使用示例程序：
```java
package study;

import java.util.ArrayList;
import java.util.List;

public class Demo2 {
	public static void main(String[] args) {
		List list = new ArrayList();  
		//导包一定要导入java.util.List
		list.add("张三");
		list.add("李四");
		list.add("王五");
		
		System.out.println(list);
		System.out.println("------------------------");
		
		list.add(1, "赵六"); //在元素1后面插入“赵六"
		System.out.println(list);		
		System.out.println("------------------------");
		
		List list2 = new ArrayList();
		list2.add("陈七");
		list2.add("朱九");
		list2.add("陈七");
		list.addAll(2,list2);
		System.out.println(list);
		System.out.println("------------------------");
		
		//get方法
		System.out.println(list.get(1)); 
		System.out.println("------------------------");
		
		//indexOf和lastIndexOf
		System.out.println(list2.indexOf("陈七"));
		System.out.println(list2.lastIndexOf("陈七"));
		System.out.println("------------------------");
		
		//subList(int fromIndex, int toIndex);
		//在Java中，所有使用到区间范围的操作，全是要头不要尾；
		List sublist = list.subList(0, 5);
		System.out.println(sublist);
		
		list.set(list.indexOf("张三"), "六六六");
		System.out.println(list);				
	}
}
```
程序运行结果：
```java
[张三, 李四, 王五]
------------------------
[张三, 赵六, 李四, 王五]
------------------------
[张三, 赵六, 陈七, 朱九, 陈七, 李四, 王五]
------------------------
赵六
------------------------
0
2
------------------------
[张三, 赵六, 陈七, 朱九, 陈七]
[六六六, 赵六, 陈七, 朱九, 陈七, 李四, 王五]
```

### （一）List 特有迭代器- ListIterator()

- 方法：
  - hasNext();
  - next();
  - remove();
  - add(Object 0); //在当前迭代器指向的位置上，添加元素，其他元素向后移动；
  - set(Object 0); //替换 next 获取到的元素；
  - nextIndex(); //下一个元素的下标；


代码示例：
```java
package study;

import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

/**
 * class detail:ListIterator
 * @author GJXAIOU
 * @since  2019年6月28日
 */
public class Listitetator_use {
	public static void main(String[] args) {
		List list = new ArrayList();
		list.add("小米");
		list.add("华为");
		list.add("苹果");
		
		//获取List的特有迭代器
	ListIterator iterator = list.listIterator();
	
	System.out.println("下一个元素" + iterator.hasNext());
	System.out.println("next获取数据" + iterator.next());
	System.out.println("set方法进行替换");
	iterator.set("mi"); //替换next获取到的元素
	
	System.out.println("添加内容：");
	iterator.add("VO"); //在当前迭代器位置上增加元素
	System.out.println(list);
	
	System.out.println(iterator.nextIndex());
	}
}

```
程序运行结果：
```java
下一个元素true
next获取数据小米
set方法进行替换
添加内容：
[mi, VO, 华为, 苹果]
2
```

### （二）List 集合的三种遍历方式：
- for 循环
- 迭代器
- 增强 for 循环

```java
package study;

import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListBianLi {
	public static void main(String[] args) {
		List list = new ArrayList();
		
		list.add("A");
		list.add("B");
		list.add("C");
		list.add("D");
		list.add("E");
		
		//使用for循环进行遍历：使用get(int index)  和size()方法
		for(int i = 0; i < list.size(); i++) {
			System.out.println(list.get(i));
		}
		
		System.out.println("*********************");
		
		//使用ListIterator迭代器
		ListIterator iterator = list.listIterator();
		
		while (iterator.hasNext()) {
			System.out.println(iterator.next());
			}
		
		System.out.println("*********************");

        //逆向  
        //hasPrevious 判断前一个元素有没有 
          while(iterator.hasPrevious()){ 
               System.out.println(iterator.previous());     
            } 
           }
		
		//增强for循环：实质为迭代器的实现
		for(Object object : list) {  //数据类型  变量名
			System.out.println(object);
		}	
	}
} 
```
程序运行结果：
```java
A
B
C
D
E
*********************
A
B
C
D
E
*********************
A
B
C
D
E
```



### （三）ArrayList 【重点】
**ArrayList 是底层维护了一个 Object 类型的数组**，这样的话这个 ArrayList 既可以保存任意类型的数据，又有数组的优势；

- 特征：
  - 当调用无参构造方法 ArrayList，这里创建的底层 Object 类型的数组元素个数默认为 10 ，即 DEFAULT_CAPACITY = 10；
  - 数组查询快、增删慢的特征；
- 开发中使用较多的场景：
  - 查询场景较多但是增删较少的程序：例如：图书馆管理系统，人员管理系统；


- ArrayList 特有的方法：都不太常用
  - ensureCapacity(int minCapacity); //判断当前 ArrayList 里面保存元素内容 Object 数组，元素个数是否大于 minCapacity;
  - trimToSize(); //截断底层维护的 Object 类型的数组，让数组容量变成当前 ArrayList 的 size 值（有效元素个数）

- 查询快、增删慢的原理：
  - 查询快：因为底层维护的是一个 Object 类型的数组，可以完全使用数组的下标机制来访问数据，这种访问的形式是非常快的；
  - 增加慢：增加数据时候，可能导致 ArrayList 底层的 Object 数组的元素不够用，那么会调用数组的扩容方法 grow，而扩容方法会创建一个新的数组，数组的元素大于原来数组，同时会将源数据拷贝到新数组中，数据量较大的时候， 拷贝过程耗费时间；
  - 删除慢：因为删除一个数据，会导致数组中该元素之后的所有数据做一个整体的左移，这也是一次数组的拷贝，浪费时间；

问： ArrayList 是一个可以自增长的空间，请问，增加的原理是什么？增长的长度是多少？
ArrayList 底层维护的是一个 Object 数组，默认的元素个数为 10，如果添加元素，当前需求的元素空间超过了 Object 数组的元素个数，会调用底层的 grow 方法，进行数组的扩容和拷贝；
其中扩容量大约为 1.5 倍；：newCapacity = oldCapacity +(oldCapacity >> 1);
即 新元素个数 = 老元素个数 + （老元素个数 >> 1）

### （四）LinkedList
底层维护的是一个链表，特征：增删快、查找慢

LinkedList 特有的方法：
- addFirst(Object o);
- addLast(Object o);
- getFirst();
- getLast();
- removeFirst();
- removeLast();


![LinkedList和ArrayList继承区别]($resource/LinkedList%E5%92%8CArrayList%E7%BB%A7%E6%89%BF%E5%8C%BA%E5%88%AB.png)










