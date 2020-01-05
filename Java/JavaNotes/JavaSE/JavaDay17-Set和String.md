---
tags : 
- java基础

flag: yellow
---

@toc

# JavaDay17 Set 和 String

## 一、复习

- Collection 集合的总接口
      add(Object o)   addAll(Collection c)    remove(Object o)    clear() removeAll(Collection c)   retainAll(Collection c)    contains(Object o)    containsAll(Collection c)   isEmpty()  equals(Collection c)    iterator()    size()    toArray()
- List 接口  有序 可重复
          add(int index, Object o) addAll(int index, Collection c) indexOf(Object o)
          lastIndexOf(Object o) get(int index) subList(int fromIndex, int toIndex)
          set(int index, Object o) listIterator() 
- ArrayList  
    底层维护的是一个Object类型的数组，默认的元素个数为10 
    特征： 增删慢，查找快
    ensureCapacity(int minCapacity);
    trimToSize();
- LinkedList 
  底层维护的是一个链表
  特征： 增删快，查找慢
   addFrist(Object o) addLast(Object o) getFrist() getLast() removeFirst() 
    removeLast()  
-  Vector (了解)
    和ArrayList是类似的，基本上常用的方法也是一致
                 线程安全，效率低！！！
- Set
  
  
- 迭代的方法：
   Iterator 
      共有的：  next() hasNext() remove()
   ListIterator
       特有的： add(Object o) set(Object o) nextIndex()

---

## 二、 Set
是一个接口、无序（指添加顺序和展示顺序可能不一致）；没有特有的方法；
使用代码示例：
```java
package study;
import java.util.HashSet;
import java.util.Set;

public class Set_Use {
	public static void main(String[] args) {
		Set set = new HashSet();
		
		set.add("A");
		set.add("B");
		set.add("E");
		set.add("B");
		set.add("C");
		System.out.println(set);
	}
}
```
结果：`[A, B, C, E]`



### （一）set 两个实现类
#### 1. HashSet：
在添加元素的时候，每一次都会调用 hashCode 方法；
下面代码通过重写 hashCode 和 equals 方法，也是首先调用 hashCode 方法，然后调用 equals 方法;

- HashSet 存储原理：
  - 当向 HashSet 集合中添加元素，HashSet 首先会调用该元素 hashCode 方法，获取该对象的 Hash 值（因为这里进行了 hashCode 方法的重写，所以获取的不再是该对象的地址）。然后通过【移位】计算，计算出该元素应该保存在【哈希】表中的哪一个位置；
  - 情况一：该位置没有任何元素，直接插入；
  - 情况二：该位置存在其他元素，哈希表就会调用该元素的 equals 方法，和已经保存在哈希表里面的元素进行比较；
    - 如果比较的结果为 true，表示相同的元素，无法添加；
    - 如果比较的结果为 false,表示为不同的元素，可以添加；
- 哈希表中每一个单元都是一个桶式结构，可以保存多个元素，允许元素共存；

代码实现：
```java
package study;
import java.util.HashSet;
/**
 * class detail:hashSet
 * @author GJXAIOU
 * @since  2019年6月29日
 */

class StudentT{
	private int id;
	private String name;
	
	public StudentT() {	}
	
	public StudentT(int id, String name) {
		this.id = id;
		this.name = name;
	}
	
	@Override
	public String toString() {
		return "[ ID : " + id  + "  Name : " + name + "]";
	}
	//重写HashCode和equals方法
	@Override
	public boolean equals(Object object) {
		System.out.println("********equals*********");
		StudentT studentT = (StudentT) object;
		//return this.id == studentT.id;
		return this.name == studentT.name;
	}
		
	@Override
	public int hashCode() {
		System.out.println("**重写hashCode，根据id判断值是否相等*");
		return this.id;
	}
}	


public class HashSet_Use {
	public static void main(String[] args) {
		HashSet hashSet = new HashSet();
		
		hashSet.add(new StudentT(1,"A"));
		hashSet.add(new StudentT(2,"C"));
		hashSet.add(new StudentT(3,"B"));
		hashSet.add(new StudentT(2,"D"));
		System.out.println(hashSet);
	}	
		
}


```
重写 equals 方法中，如果返回为：`return this.id == studentT.id`程序运行结果为：
```
*********hashCode***********
*********hashCode***********
*********hashCode***********
*********hashCode***********
********equals*********
[[ ID : 1  Name : A], [ ID : 2  Name : C], [ ID : 3  Name : B]]
```
如果返回为：`return this.name == studentT.name`程序运行结果为：
```
*********hashCode***********
*********hashCode***********
*********hashCode***********
*********hashCode***********
********equals*********
[[ ID : 1Name : A], [ ID : 2Name : C], [ ID : 2Name : D], [ ID : 3Name : B]]
```

HashSet 小代码实验：
```java
package study;
import java.util.HashSet;
import java.util.Scanner;

/**
 * class detail: 从键盘上接收用户输入（账号和密码），如果账号相同则认为是相同用户，则无法添加，使用HashSet；
 * @author GJXAIOU
 * @since  2019年6月29日
 */
class Account{
	private String userName;
	private String password;
	
	public Account() {}
	
	public Account(String userName, String password) {
		this.userName = userName;
		this.password = password;
	}
	

	@Override
	public int hashCode() {
		//这里使用用户的hashCode作为当前对象hashCode值，因为字符串只要是不同的字符串，就会有不同的hashCode
		return this.userName.hashCode();
	}
	
	@Override
	public boolean equals(Object arg0) {
		Account account = (Account)arg0;
		return this.userName.equals(account.userName);
	}
}


public class Practice {
	public static void main(String[] args) {
		HashSet hashset = new HashSet();
		
		Scanner scanner = new Scanner(System.in);
		while(true) {
			System.out.println("请输入用户名：");
			String userName = scanner.nextLine();
			
			System.out.println("请输入密码：");
			String password = scanner.nextLine();

			Account account = new Account(userName, password);
			if (hashset.add(account)) {
				System.out.println("注册成功");
			}else {
				System.out.println("用户名已存在，请重新输入");
			}
		}
		
	}
}

```

#### 2.TreeSet 实现类
TreeSet 是一个树形结构的 Set 结构；
  发现：==TreeSet 中添加元素原本是字符串类型，当添加自定义对象或者数字的时候，会报异常==；
  原因：因为 TreeSet 是一个树形结构，所有的元素都需要进行比较之后才可以放入到 Set 集合中，而字符串和自定义类对象是没有比较的方式和方法的；
  也就是说【要求】在 TreeSet 里面的所有的元素都要有【比较的方式】或者有【自然顺序】

TreeSet 中添加自定义元素的方式：
  - 方式一：让这个自定义类【遵从】Comparable 接口，实现 compareTo 方法；
  	compareTo 的返回值有三种：0、正整数、负整数
	   返回值  |  含义
	   ---| ---
	   0       |     表示相同；
	   正整数  |  表示【调用这个方法的对象】大于传值的对象；
	   负整数  |  表示【调用这个方法的对象】小于传值的对象；
```java
package study;

import java.util.TreeSet;

/**
 * class detail:通过在自定义类中实现compareTo方法，实现可以在TreeSet中添加自定义类元素
 * @author GJXAIOU
 * @since  2019年6月29日
 */

class Person implements Comparable{
	private int id;
	private String name;
	private int salary;
	
	public Person(){ }
	
	public Person(int id, String name, int salary) {
		this.id = id;
		this.name = name;
		this.salary = salary;
	}
	
	@Override
	public int compareTo(Object arg0) {
		//这里以工资为标准
		Person person = (Person)arg0;
		return this.salary  -  person.salary;
	}
	
	@Override
	public String toString() {
		return "[ ID : " + id + "  Name : "  + name + "  Salary : " + salary + " ]";
	}
	
}


public class TreeSet_Use {
	public static void main(String[] args) {
		TreeSet treeSet = new TreeSet();
		
//		treeSet.add("1");
//		treeSet.add("3");
//		treeSet.add("5");
//		treeSet.add("1");
		
		treeSet.add(new Person(1, "A", 20));
		treeSet.add(new Person(2,"B",24));
		treeSet.add(new Person(3, "C", 20));
		
		System.err.println(treeSet);
	}
}

```
程序运行结果：
`[[ ID : 1  Name : A  Salary : 20 ], [ ID : 2  Name : B  Salary : 24 ]]`

  - 方法二：自定义比较器，这个比较器是【遵从】Comparator 接口，实现 `int compare(Object o1,Object o2)`
可以在创建 TreeSet 对象的时候，传入比较器对作为比较方式；

```java
package study;

import java.util.Comparator;
import java.util.TreeSet;

/**
 * class detail:通过在自定义类中实现compareTo方法，实现可以在TreeSet中添加自定义类元素
 * @author GJXAIOU
 * @since  2019年6月29日
 */

class PersonP{
	private int id;
	private String name;
	 int salary;
	
	public PersonP(){ }
	
	public PersonP(int id, String name, int salary) {
		this.id = id;
		this.name = name;
		this.salary = salary;
	}

	@Override
	public String toString() {
		return "[ ID : " + id + "  Name : "  + name + "  Salary : " + salary + " ]";
	}	
}


//自定义比较器
class MyCompare implements Comparator{

	@Override
	public int compare(Object arg0, Object arg1) {
		PersonP person0 = (PersonP) arg0;
		PersonP person1 = (PersonP) arg1;
		return person0.salary - person1.salary;
	}
}


	
public class ZiDingYiCompare {
	public static void main(String[] args) {
		
		//创建TreeSet时候，传入自定义比较器对象
		TreeSet treeSet = new TreeSet(new MyCompare());
		
//		treeSet.add("1");
//		treeSet.add("3");
//		treeSet.add("5");
//		treeSet.add("1");
		
		treeSet.add(new PersonP(1, "A", 20));
		treeSet.add(new PersonP(2, "B", 24));
		treeSet.add(new PersonP(3, "C", 20));
		
		System.err.println(treeSet);
	}
}

```
程序运行结果：`[[ ID : 1  Name : A  Salary : 20 ], [ ID : 2  Name : B  Salary : 24 ]]`








