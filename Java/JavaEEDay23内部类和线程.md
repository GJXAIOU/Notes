---
tags: 
- 内部类
- 线程
- 进程
style: summer
---

# JavaDay23 内部类和线程

@toc

## 一、复习
### （一）Comparable 和 Comparator
两者都是接口
- Comparable 是一个接口，给自定义类提供比较方式，遵从 Comparable<T>的实现类，必须实现 compareTo（T o）方法；即进行两个对象的比较；
- Comparator 是一个接口，用来做自定义比较器，遵从 Comparator<T>接口的实现类，必须使用 compare(T o1, T o2)；

方法1.实现自定义比较器
```java
class MyCompare implements Comparator<Student>{
  @override
  public int compare(Student o1, Student o2){
    return o1.getAge() - o2.getAge();
  }
}
```
上面这种方式：好理解但是需要重新定义一个类，而且可能会统一放在一个包里面，略显复杂；

方法 2：使用匿名内部类的匿名对象
格式：`Arrays.sort(T[] t, Comparator<? extends T> com)`
```java
Student[] array = {stu1, stu2, stu3};

Arrays.sort(array.new Comparator<Student>{
  @Override
  public int compare(Student o1, Student o2){
    return o1.getAge() - o2.getAge();
  }
});
```


## 二、内部类

- 生活实例：
在人类中，有些东西，比如内脏，用成员方法或者成员变量描述都显着有点不太合适，因为这些内脏首先是**属于【人体的一部分】，而且会使用【人体的一些属性】，但是又拥有自己的一些【特征】**；能不能把这些器官，认为是一个类，一个属于人类内部的一个类。

- 内部类分类：
	1.成员内部类
	2.局部内部类
	3.匿名内部类


### （一）成员内部类
- 内部类和外部类比较：
  - 1.**成员内部类可以使用外部类的所有成员变量和成员方法**, 不管用什么权限修饰，不管是private还是public都可以使用，因为，这是在内部类的内部使用。
  - 2.【在Outer类的外部创建Outer的Inner对象】
	格式如下:
		`外部类名.内部类名  内部类对象名 = new 外部类名().new 内部类名();`
	例如：`Outer.inner inner = new Outer().new Inner();`
【第一个知识点】:普通的成员变量和成员方法，在没有对象的情况下，不能再类外使用
  - 3.**如果内部类和外部类存在同名的成员变量，这里默认是就近原则，使用的是内部类的成员变量**
如果想要**使用外部类的成员变量和成员方法**的格式： 
`外部类名.this.同名成员变量;`
`外部类名.this.同名成员方法(参数列表);`
  -  4.在外部类的类内方法中，可以创建内部类的对象
 				
```java
package com.qfedu.a_innnerclass;

class Outer {
	int num = 100; //外部类的成员变量

	private static int s = 10;

	class Inner { //这是Outer类的一个成员内部类
		int i = 10; //内部类的成员变量
		int num = 50;
		public void testInner() {
			System.out.println("内部类的成员方法");
			testOuter();
			System.out.println("内部类同名成员变量:" + num); // 50 就近原则
			//使用  外部类名.this.成员变量名  获取同名外部成员变量
			System.out.println("外部类同名成员变量:" + Outer.this.num);
			
			//分别调用内部和外部成员方法
			test();
			Outer.this.test();

			System.out.println(s);
			testStatic();
		}

		public void test() {
			System.out.println("内部类的test方法");
		}
	}//inner

	
	public void testOuter() {
		System.out.println("外部类的成员方法");
	}

	public void test() {
		System.out.println("外部类的test方法");
	}

	//为了将内部类的语句输出
	//在【外部类】中定义【内部类的类对象】
	public void createInnnerClass() {
		//外部类的成员方法中调用内部类的构造方法，通过new关键字来创建内部类类对象使用
		Inner inner = new Inner();
		inner.testInner();
	}

	public static void testStatic() {
		System.out.println("外部类的静态成员方法~");
	}
}

//☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
public class Demo1 {
	public static void main(String[] args) {
		//创建内部类对象的方法一：在外部类中创建内部类对象
//		Outer outer = new Outer();
//		outer.createInnnerClass();

		//创建内部类对象的方法一：在类外创建一个Inner类对象
		//数据类型是Outer.Inner 表示是外部类里面的内部类数据类型
		Outer.Inner inner = new Outer().new Inner();
		inner.testInner();
	}
}
```
运行结果：
```java
内部类的成员方法
外部类的成员方法
内部类成员变量:50
外部类成员变量:100
内部类的test方法
外部类的test方法
10
外部类的静态成员方法~
```

### （二）局部内部类
放在方法内或者函数内的类，称之为局部内部类

- 【第二个知识点】
  - 1.局部变量的
    - 【生存期】
从声明定义位置开始，到代码块结束
    - 【作用域】
只能在当前代码块内部
  - 2.类对象的
    - 【生存期】
通过new关键字创建的时候开始，JVM垃圾回收机制调用时，销毁该对象，结束
    - 【作用域】
那个引用变量拥有这个对象的首地址，哪里就是他的作用域

- 说明：
**局部内部类只能在函数或者方法中使用，方法或者函数外不能使用**

- 发现：
**局部内部类中，貌似不能【修改】所在方法或者函数中的局部变量**，原因如下：

局部内部类的对象是在函数或者说方法的内部通过JVM借助于new关键字，和局部内部类的构造方法，创建的一个类对象，并且该对象是由JVM的垃圾回收机制回收的；
但是局部变量n是在testInner()方法中，而这个局部变量的n的生存周期是和testInner()该方法的大括号有关，生存期和作用域都是在大括号以内；
如果在testInner()方法的内部，MethodInner()这个类是方法中的局部内部类，而创建的对象在使用testInner()方法中的局部变量时，因为对象的销毁时间不确定，但是一定是晚于局部变量的销毁的，这里隐含了一个类对象【延长了局部变量生存期】，这个是不符合Java原理的；
这就是为什么不能修改，因为有可能在这个对象被使用的时候，局部变量的内存空间已经被内存收回，换而言之这个局部变量不存在了。

- 【解决方法】
如果是在局部内部类中使用的所在函数或者方法的局部变量，该变量用final修饰

```java
package com.qfedu.a_innnerclass;

class Test {
	int num = 100;
	
	public void testInner() {
		//这里是在方法中定义了一个类,这个类只能在当前函数或者方法中使用
		final int n = 10;
		class MethodInner {
			int i = 10;
			
			public void function() {
				System.out.println(i);
				num = 20;
				System.out.println(num);
				//n = 20;//使用final之后就可以了
				System.out.println(n);
				System.out.println("局部内部类的成员方法");
			}
		}
		
		MethodInner methodInner = new MethodInner();
		methodInner.function();
	}
	
}

public class Demo2 {
	public static void main(String[] args) {
//		for (int i = 0; i < 10; i++) {
//			System.out.println(i);
//		}
		
//		System.out.println(i);
		new Test().testInner();
	}
}
```
程序运行结果：
```java
10
20
10
局部内部类的成员方法
```

### （三）匿名内部类		
只是没有了名字，但是类本体中该实现的还得实现；
且**匿名内部类必须继承一个类或者实现一个接口**；
【第四个知识点】
	类的本体:在类声明部分，大括号里面的内容就是类 的本体！！！
```java
class Test {
	//成员变量
	//成员方法
}
```
	
匿名内部类就是没有名字的类，但是有类的本体！！！

```java
package com.qfedu.a_innnerclass;

import java.util.Arrays;
import java.util.Comparator;

abstract class Animal {
	int age;

	public abstract void test();
	abstract public void jump();
}

class CodingMonkey extends Animal {

	//以下两个方法是类的本体！！！
	@Override
	public void test() {
		System.out.println("这是类本体中的一个成员方法");
	}

	@Override
	public void jump() {
		System.out.println("Hello");
	}
}



interface A {
	int i = 0; // public static final
	public void testA(); //abstract
}

public class Demo3 {
	public static void main(String[] args) {
		//方法一：正常方法： 可以创建CodingMonkey即Animal的子类的对象，进行方法的操作
		CodingMonkey cm = new CodingMonkey();

		//Animal a = cm; 多态，父类的引用指向子类的对象~
		cm.test();
		cm.jump();


		//方法二：匿名对象类： 也可以直接通过匿名对象类进行直接操作

     /*
		Animal是一个抽象类，ani 是抽象类的一个引用类型变量

		new Animal() {
			发现这里面的内容和继承该抽象类的子类内容一致，都是抽象类，要求子类实现的方法；同时这里是类的本体，而这里可以看做是一个类，但是这个类没有名字，所以：这个就是【匿名内部类】；本质上这里创建了一个【匿名内部类】的对象，赋值给了Animal的引用数据类型 ani，同时这里隐含了一个【继承】关系
			 };
			 */
		Animal ani = new Animal() {
			//同样需要实现继承的相关功能
			@Override
			public void test() {
				System.out.println("匿名内部类的里面的test方法");
			}

			@Override
			public void jump() {
				System.out.println("匿名内部类的Hello");
			}
		};

		ani.jump();
		ani.test();


		//以上更加简洁的使用方式：

		//匿名内部类的匿名对象直接调用方法
		new A() {
			//这里是一个类【遵从】接口之后，要求实现接口中的抽象方法，这里也是一个【匿名内部类】
			@Override
			public void testA() {
				System.out.println("匿名内部类实现interface A中的testA方法");
			}
		}.testA();//new A(){}可以看成匿名对象
		//Java Android开发

		Integer[] arr = {3, 2, 4, 5, 6, 1};

		//匿名内部类的匿名对象作为方法的参数
		Arrays.sort(arr, new Comparator<Integer>() {

			@Override
			public int compare(Integer o1, Integer o2) {
				return o1 - o2;
			}
		});

		System.out.println(Arrays.toString(arr));
	}

}
```
程序运行结果：
```java
这是类本体中的一个成员方法
Hello
匿名内部类的Hello
匿名内部类的里面的test方法
匿名内部类实现interface A中的testA方法
[1, 2, 3, 4, 5, 6]
```


- 在使用匿名内部类的过程中，我们需要注意如下几点：
1、使用**匿名内部类时，我们必须是继承一个类或者实现一个接口**，但是两者不可兼得，同时也只能继承一个类或者实现一个接口。
2、**匿名内部类中是不能定义构造函数的**。
3、**匿名内部类中不能存在任何的静态成员变量和静态方法**。
4、匿名内部类为局部内部类，所以局部内部类的所有限制同样对匿名内部类生效。
5、匿名内部类不能是抽象的，它必须要实现继承的类或者实现的接口的所有抽象方法。

## 三、多线程与多进程

- 多线程和多进程：
  - 线程：
在一个软件中，负责不同功能的子程序，称之为线程。
  - 进程：
是在计算机系统中所有正在运行的程序，都可以看做是一个进程。多进程的操作系统

- 从实际出发：
每一个CPU的内核在一个时间片内只能执行一件事情。但是这个时间片非常的短，不同的程序会出现在不同的时间片上，不断的切换，所以你感觉是在同时运行,现在的CPU基本上都是多核多线程的。


- 面试题：
请问一个Java程序再运行的时候，最少有几个线程？？？
最少是两个线程：1. main线程，即主线程   2. JVM的垃圾回收机制

- **多线程的好处**：
1.可以让一个程序同时执行不同的任务
2.可以提高资源的利用率

- **多线程的弊端**：
1.线程安全问题 之前的迭代器
2.增加了CPU负担
3.降低了其他程序CPU的执行概率
4.容易出现死锁行为

- **如何创建一个线程**：
两种方式：
1.自定义一个类继承Thread类，那么这个类就可以看做是一个多线程类
2.要求【重写】Thread类里面的run方法，把需要执行的自定义线程代码放入这个run方法


下面是使用迭代器出现的同时操作问题；
```java
package com.qfedu.b_thread;

import java.util.ArrayList;
import java.util.Iterator;

class MyThread extends Thread {
	//要求重写【run】方法
	@Override
	public void run() {
		for (int i = 0; i < 10; i++) {
			System.out.println("自定义Thread类:" + i);
		}
	}
}

public class Demo1 {
	public static void main(String[] args) {
		
		//1. 创建自定义线程类的类对象
		MyThread mt = new MyThread();
		
		//2. 启动线程 。
		//mt.run();尝试调用n方法 相当于调用了一个普通的方法，而不是一个线程
		mt.start(); //启动线程的方式，会发现，MyThread线程和main线程在抢占CPU资源
		
		for (int i = 0; i < 10; i++) {
			System.out.println("main线程：" + i);
		}




		/*
		ArrayList<String> list = new ArrayList<String>();
		
		list.add("hello, how do you do");
		list.add("I'm fine, Thank you , and you");
		list.add("I'm fine, too");

        //三种遍历方式：
		for (int i = 0; i < list.size(); i++) {
			System.out.println(list.get(i));
		}
		
		for (String string : list) {
			System.out.println(string);
		}
		
		Iterator<String> it = list.iterator();
		while (it.hasNext()) {
			System.out.println(it.next());
		}
		*/
	}
}
```

