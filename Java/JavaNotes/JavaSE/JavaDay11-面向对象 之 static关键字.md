---
tags : 
- java基础

flag: blue
---

@toc

# JavaDay11 static关键字

## 一、static关键字的引入
- 生活情况：
  饮水机为例，通常是放在一个公共的环境中，大家共同使用，不会说每个人入学发一个饮水机。
  如果每一个人都有一个饮水机，过多电线，过多的水管，占用了大量的空间

- 代码中的问题：
  发现在代码中，存在某一些成员变量的数值是一样的，大量重复，在每一个对象中都有存在，例如：
  当前代码中的国家，都是中国。这样会占用过多的内存资源，甚至于硬盘资源。

- 期望：
 	把国家中国属性放到一个共享的区域里，然后让每一个对象使用就好
 	
- 解决问题：
 	使用static关键字

---

## 二、static修饰成员变量

### （一）static关键字的使用：
- 【重点】
1. 如果使用static修饰的成员变量，这些成员变量称之为【静态成员变量】，这个 ==【静态成员变量】实际存放的内存空间在【内存数据区】，和当前【类对象内存】没有任何关系== 。也就是说，这个【静态成员变量】使用内存空间不再【堆区】对象内存中

2. ==用static修饰的成员变量，这个【静态成员变量】可以提供给多个类对象使用==。

3. 什么时候用static，真正意义上存在大量重复，并且存在一定共享基础的数据，这种情况下，可以使用static修饰
  例如：country属性就适合用static修饰，但是name属性就不适合
  
 -  小问题:
    发现通过类对象来调用【静态成员变量】，报警告
    The static field Person.country should be accessed in a static way
    ==用static修饰的【静态成员变量】应该用【静态】的方式访问==
  
- 【重点】
   ==用static修饰的成员变量，这个成员变量会【早于】类对象的创建而创建，而且【晚于】类对象的销毁而销毁==。所以，用static修饰的【静态成员变量】是和类**对象"无关**的"，这里的“无关的”指的是：首先内存上是无关的，然后和对象的创没创建没有关系。
  	
   严格来说：类对象和【静态成员变量】无关，那么通过类对象来调用【静态成员变量】是"非法的"
   Java语言期望的是：更加严谨的调用方式，因为和对象"无关"，所以不希望用调用来调用  
  ==【推荐调用/使用成员变量的方式】`类名.成员变量;`==
   没用警告，也是让你记得static修饰的【静态成员变量】和类对象无关
  
  【修改问题】
   ==用static修饰的成员变量，不管通过哪一种方式修改了内存，那么所有用到这个【静态成员变量】的数据，都会发生变动==
  	
   因为【静态成员变量】是一个【隐含的共享资源】
   	 例如：
  		井里放块糖，有甜大家尝

示例代码：
```java
class Person {
	//成员变量
	private String name;    //姓名
	
	//这里就是用static修饰的成员变量
	static String country = "中国"; //国家
	
	//构造方法
	public Person() {}
	
	public Person(String name) {
		this.name = name;
	}
	
	//setter和getter方法
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
}


public class Demo1 {
	public static void main(String[] args) {
		Person p1 = new Person("叶问"); //4个		
		Person p2 = new Person("陈华顺"); //4个 		
		Person p3 = new Person("陈真"); // 4个		     
		Person p4 = new Person("李云龙"); // 4个
		
		System.out.println("p1.country:" + p1.country);
		System.out.println("p2.country:" + p2.country);
		Person.country = "中国"; //推荐方式
		p3.country = "People Republic of China";
		System.out.println("p3.country:" + p3.country); // PRC 
		System.out.println("p4.country:" + p4.country); // 中国
	}
}
```
    		

---

## 三、static修饰成员方法

static修饰成员方法，这个方法称之为【静态成员方法】

格式:
```java
权限修饰符  static 返回值类型   方法名(形式参数列表) {
           方法体;
}
```
- 【重点】
  	用static修饰的成员方法，称之为【静态成员方法】，这个 ==【静态成员方法】是早于对象的创建而【加载】，	对象销毁之后依然存在==。因为方法和函数是放在代码区，是加载进去的，所以说：【静态成员方法】和对象"无关"

如果采用类对象调用【静态成员方法】会报警告：
	The static method sleep() from type Dog should be accessed in a static way
	在Dog类里面的【静态成员方法】sleep()应该用静态的方式来调用

- 【注意事项】
1. ==在用static修饰的【静态成员方法】中不能使用this关键字==
结果推论:
因为【静态成员方法】可以通过类名来调用，用类名调用的情况下，是不存在对象的，而this关键字是用来表示调用该方法的对象关键字，这里不存在对象，所以不能用。
原理推论：
因为【静态成员方法】是早于对象加载，晚于对象销毁，所以和对象"无关"，有没有对象，该方法都存在，所以this关键字在这里无法使用

2. ==在static修饰的【静态成员方法】中，不能使用【非静态成员变量】==，因为【非静态成员变量】是保存在类对象的【堆区】内存中，和类对象共存，而在【静态成员方法】的调用过程中是没有对象的，所以不能使用【非静态成员变量】

3. 【静态成员方法】中可以直接使用【静态成员变量】

4. 想在【静态成员方法】里面使用【非静态成员变量】怎么办???
可以在当前【静态成员方法】中用new关键字，调用构造方法，创建一个对象，使用该对象。
案例：
单例设计模式

【总结】
静态对静态，非静态对非静态

【静态成员方法用途】
  	1. 调用方便，通过类名直接调用该方法，不需要借助于对象
  	2. 用来完成一些【工具类】	
  		Arrays 工具类

示例代码：
```java
class Dog {
	//成员变量
	private String name;
	
	//【静态成员变量】
	static String country = " JP ";
	
	//构造方法
	public Dog() {}
	
	public Dog(String name) {
		this.name = name; 
	}
	
	//setter 和 getter方法
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
	
	//成员方法
	public void run() {
		this.name = "ll"; //用this关键字来调用非静态成员变量
						  //this表示调用该方法的对象
		name = "lxl"; //非静态成员变量
		country = "Janp"; //静态成员变量
		System.out.println("跑来跑去~~~");
	}
	
	//静态成员方法
	public static void sleep() {
		//this.name = "ll"; Cannot use this in static context  WHY
		country = "RB"; //静态成员变量
		//name = "gz"; // WHY
		System.out.println("睡大觉~~~~");
	}
	
	// 在静态成员方法中使用非静态成员变量
	public static void test() {
		Dog dog = new Dog("狗子");		
		System.out.println(dog.name);
	}
}

public class Demo2 {
	public static void main(String[] args) {
		Dog dog = new Dog("Bobo");
		
		dog.sleep();
		Dog.sleep();
		
		Dog.test();
	}
}
```

---

## 四、 静态成员变量的使用
   详见 Demo3.java 
        一般用于计数器
```java
package com.qfedu.a_static;

/*
	静态成员变量的使用案例：
  		统计一个类创建了多少个对象
  		人口统计，数据统计，ID的自动生成
  		
  	ID用创建对象之后，自动赋值，并且每一个对象的ID号是唯一的，递增的
 */

class Student {
	private int id; //这个用来统计学生 的ID号 这个ID考虑自动生成
	private String name;
	private char sex;
	
	//在数据区的一个【静态成员变量】用于保存count的数据，而且私有化，不给类外提供任何的
	//修改或者获取该数据的方式
	//如果程序停止，这里就会重新计数，没 有做数据【持久化】操作
	private static int count = 1;
	
	//构造代码块，任何的一个对象都会执行的代码
	{
		this.id = count++;
	}
	
	public Student() {}
	
	//因为ID是通过当前的程序自动生成的，不需要通过构造方法传参赋值
	public Student(String name, char sex) {
		this.name = name;
		this.sex = sex;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
	
	public void setSex(char sex) {
		this.sex = sex;
	}
	
	public char getSex() {
		return sex;
	}
	//ID不希望外部能有修改的方式或者权限，所以封装之后不提供setter方法，只提供一个查看功能getter方法
	public int getId() {
		return id;
	}
	
	
}

public class Demo3 {
	public static void main(String[] args) {
		Student stu1 = new Student();
		System.out.println(stu1.getId());
		
		Student stu2 = new Student();
		System.out.println(stu2.getId());
		
	}
}

```

--- 

## 五、静态成员方法的使用

```java
package com.qfedu.a_static;

/*
	静态成员变量的使用案例：
  		统计一个类创建了多少个对象
  		人口统计，数据统计，ID的自动生成
  		
  	ID用创建对象之后，自动赋值，并且每一个对象的ID号是唯一的，递增的
 */

class Student {
	private int id; //这个用来统计学生 的ID号 这个ID考虑自动生成
	private String name;
	private char sex;
	
	//在数据区的一个【静态成员变量】用于保存count的数据，而且私有化，不给类外提供任何的
	//修改或者获取该数据的方式
	//如果程序停止，这里就会重新计数，没 有做数据【持久化】操作
	private static int count = 1;
	
	//构造代码块，任何的一个对象都会执行的代码
	{
		this.id = count++;
	}
	
	public Student() {}
	
	//因为ID是通过当前的程序自动生成的，不需要通过构造方法传参赋值
	public Student(String name, char sex) {
		this.name = name;
		this.sex = sex;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return name;
	}
	
	public void setSex(char sex) {
		this.sex = sex;
	}
	
	public char getSex() {
		return sex;
	}
	//ID不希望外部能有修改的方式或者权限，所以封装之后不提供setter方法，只提供一个查看功能getter方法
	public int getId() {
		return id;
	}
	
	
}

public class Demo3 {
	public static void main(String[] args) {
		Student stu1 = new Student();
		System.out.println(stu1.getId());
		
		Student stu2 = new Student();
		System.out.println(stu2.getId());
		
	}
}


```
  详见 Demo4.java 和 Demo5.java
      通常会做成工具类
```java
package com.qfedu.a_static;

import java.util.Arrays;

/*
  静态成员方法的使用
  	
  	做成工具类，提供给开发者使用，更加便利，调用方便，摆脱类对象的约束，而且所有的的数据是
  	调用者传入和工具类里面的成员变量无关，甚至于某些工具类是没有任何成员变量的
  	
  	Arrays 工具类，提供了大量的数组操作的方法，而且在Arrays工具类里面所有的方法
  	都是用public static修饰的
  	
  	sort  toString  binarySearch
 */

public class Demo4 {
	public static void main(String[] args) {
		int[] arr = {1, 9, 3, 4, 2, 8, 10, 5, 7, 6};
		
		//使用以下Arrays工具类里面的sort方法，排序算法，默认是从小到大，升序排列
		Arrays.sort(arr);
		
		for (int i = 0; i < arr.length; i++) {
			System.out.println(arr[i]);
		}
		
		//Arrays工具类里面的toString方法，方法的返回值是一个String类型
		//例如:{1, 2, 3} => [1, 2, 3]
		String arrString = Arrays.toString(arr);
		System.out.println(arrString);
		
		//Arrays 工具里面的binarySearch 是一个查找数据的方法，要求查找的数组数据必须是通过排序的数组
		//而且返回的是当前要找的数据所在下标，如果没有找到，返回负数，找到了返回有效下标值
		int index = Arrays.binarySearch(arr, 5);
		System.out.println("index =" + index);
	}
}

```

---

## 六、自定义工具类
   完成MyToString(int[] arr) 和 MyReverse(int[] arr)

---
## 七、和main函数聊聊天

```java
class Demo6 {
      public static void main(String[] args) {
	
  	}
}
```
	
- public：权限访问修饰符，公开，权限最高的，自由度也是最高的，方便JVM在任何情况下都可以直接找到main函数
开始当前程序的运行，main函数程序的唯一入口

- static: 静态修饰关键字，调用main函数不需要对象的参与，直接用类名调用即可，方便了JVM调用main函数的过程
    实际情况:   
        Demo6.main(null); //JVM做的事情
        
- void：没有返回值，main函数比较特殊，返回值对于main函数没有意义，因为main函数调用者是JVM，JVM并不需要
这个返回值

- main：函数名，众所周知的函数名，大多数语言中都有这个函数名，表示当前程序的唯一入口 支持C++ C OC 

- (String[] args): main函数运行需要的参数列表，里面的参数类型是一个String[] 是一个字符串数组，
args 是 arguments的缩写 一般不用~~~

---
## 八、数组补充知识
   ArraryIndexOutOfBoundsException 数组下标越界【异常】，通常是操作数组下标超出了数组的有效下标范围
    NullPointerException 空指针异常，操作null内存空间异常