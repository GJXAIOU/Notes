---
tags : 
- java基础

flag: blue
---

@toc

## JavaDay13  抽象类 、接口、多态
## 一、 abstract 抽象类

- 【需求】
==要求继承于该类的子类，【强制】重写这些技能方法==，不写就报错，将业务逻辑的问题提升到语法问题，错误前置

- 【解决】
abstract 修饰抽象类的关键字，也是修饰抽象方法的关键字

- 【注意事项】   
1. 如果一个 ==**方法用abstract修饰，那么这个方法是不能有方法体的**== ，这个称之为【方法的声明】
==**一个方法如果用abstract修饰，【要求继承该类的子类必须重写这个方法】**==

2. 如果 ==**一个类中存在用abstract修饰的方法，那么这个类必须用abstract修饰**==
The type Hero mus t be an abstract class to define abstract methods

3. ==**抽象类不能有自己的类对象**==
因为在用abstract修饰的抽象类当中，有可能存在抽象方法，而抽象方法是没有方法体，不知道应该运行什么代码，而创建抽象类对象之后看，就会存在这样的隐患，所以【抽象类是没有自己的类对象】

4. 一个类用abstract修饰，但是不包含抽象方法，是可以的，但是你觉得有意义吗？
语法没有问题，但是没有实际意义

【总结】
 ==如果一个类继承了用abstract修饰的抽象类，那么**要求该类必须【实现】抽象类中所有抽象方法**==

代码示例一：
```java

abstract class Hero {
	//成员变量
	int blood;
	int power;

    // 在含有继承的父类中一般提供两个构造方法：其中一个为空
	public Hero() {}
	
	public Hero(int blood, int power) {
		this.blood = blood;
		this.power = power;
	}
	
	/*
	 这里【要求】所有的英雄都有三个技能，分别是QWE，而且要求继承该Hero的子类，必须重写这三个方法
	 这里就可以使用abstract来修饰这些方法
	 */
	abstract public void Q();
	abstract public void W();
	abstract public void E();
	
	public void test() {
		System.out.println("测试");
	}
}

abstract class Test {
	public void test() {
		System.out.println("测试");
	}
}

/*
	Fizz类 是继承了Hero类，因为Hero中包含了三个用abstract修饰的方法
 	那么要求在Fizz类必须【实现】这三个方法
 */
class Fizz extends Hero {
	
	//构造方法
	public Fizz() {}
	
	public Fizz(int blood, int power) {
		super(blood, power); //调用父类的构造方法，初始化父类的成员变量
	}
	
	//是实现父类中要求继承该父类的子类必须【实现】的Q方法 implement
	@Override
	public void Q() {
		System.out.println("淘气打击");
	}

	//是实现父类中要求继承该父类的子类必须【实现】的W方法 implement
	@Override
	public void W() {
		System.out.println("海石三叉戟");
	}

	//是实现父类中要求继承该父类的子类必须【实现】的E方法 implement
	@Override
	public void E() {
		System.out.println("古灵精怪");
	}	
}

class Caitlyn extends Hero{
	
	public Caitlyn() {}
	
	public Caitlyn(int blood, int power) {
		this.blood = blood;
		this.power = power;
	}

	@Override
	public void Q() {
		System.out.println("和平使者");
	}
	
	@Override
	public void W() {
		System.out.println("约德尔诱捕器");
	}
	
	@Override
	public void E() {
		System.out.println("90口径绳网");
	}
}

public class Demo1 {
	public static void main(String[] args) {
		Fizz fizz = new Fizz(100, 100);
		
		fizz.Q();
		fizz.W();
		fizz.E();
		fizz.test();
		
		Caitlyn caitlyn = new Caitlyn(100, 100);
		
		caitlyn.Q();
		caitlyn.W();
		caitlyn.E();
		caitlyn.test();
		
		//Hero h = new Hero();
	}
}

```
程序运行结果：
```java
淘气打击
海石三叉戟
古灵精怪
测试
和平使者
约德尔诱捕器
90口径绳网
测试
```


代码示例二：
```java
package com.qfedu.a_abstract;

/*
 需求：
  	描述一个图形类 Shape类，要求所有继承
  	于该类的子类都要有 计算周长 和 面积的方法
  	
  	实现shape类
  	实现继承于shape类的 圆类 方块 三角
 */

abstract class Shape {
	abstract float perimeter();
	abstract float square();
}

class MyCircle extends Shape {
	
	private float r;
	private float PI = 3.1415926f;
	
	public MyCircle() {}
	
	public MyCircle(float r) {
		if (r <= 0) {
			this.r = 1;
		} else {
			this.r = r;
		}
	}

	@Override
	public float perimeter() {
		return 2 * r * PI;
	}

	@Override
	public float square() {
		return r * r * PI;
	}	
}

class Rect extends Shape {

	float length;
	float width;
	
	public Rect() {}
	
	public Rect(int length, float width) {
		this.length = length;
		this.width = width;
	}
	
	@Override
	public float perimeter() {
		// TODO Auto-generated method stub
		return 0;
	}

	@Override
	public float square() {
		// TODO Auto-generated method stub
		return 0;
	}
	
}

class Triangle extends Shape {
	float l1;
	float l2;
	float l3;
	
	public Triangle() {}
	
	public Triangle(float l1, float l2, float l3) {
		if (l1 < (l2 + l3) && l2 < (l1 + l3) && l3 < (l1 + l2)) {
			this.l1 = l1;
			this.l2 = l2;
			this.l3 = l3;
		} else {
			throw new RuntimeException("错了！！！");
		}
	}
	
	
	@Override
	public float perimeter() {
		return l1 + l2 + l3;
	}

	@Override
	public float square() {
		float p = (l1 + l2 + l3) / 2;
		
		return (float) Math.sqrt(p * (p - l1) * (p - l2) * (p - l3));
		
	}
	
}

public class Demo2 {
	public static void main(String[] args) {
		
	}
}

```

---


## 二、接口

- [功能]
代码中的接口：
拓展当前类的功能，或者用来打补丁

接口用到的关键字 `interface`     UI：User Interface

- 格式：
```java
interface 接口名 {
//拓展的内容
成员变量 和 成员方法
}
```
接口名采用的命名法是大驼峰命名法

【遵从】接口的关键字 `implements`



- [补充知识]
==final关键字==：
  * ==final关键字可以用来修饰成员变量和局部变量，用final修饰的变量在赋值之后不能发生改变==；
  * ==final 如果修饰一个类,该类不能被继承==；
  * ==final 修饰一个方法，该方法不能被重写==；

 ==接口中的成员方法是没有方法体==

- 【接口中的缺省属性】
  - 1.==在接口中成员变量默认的缺省属性是public static final修饰，要求在定义成员变量时直接赋值==
例如：USB接口在定义标准的时候，就确定USB接口的尺寸
  - 2.==在接口中成员方法的【缺省属性】是abstract，这里是要求【遵从】接口的类来完成的方法==
例如：USB接口规定了连接的方式和硬件要求

- 【注意事项】
  - ==1.在interface中定义方法，都是一个abstract修饰的方法，要求【遵从】接口的类要实现这些方法==
  - ==2.在interface中定义法成员变量都是用final修饰，只有使用权，没有赋值权（修改权）==
  - ==3.一个类可以【遵从】多个接口，不同的接口用逗号隔开，使用**implements**关键字来遵从接口==

Java是一门单继承，多实现/遵从的语言；


```java

interface A {
	//成员变量
	int num = 10; //【缺省属性】public static final
	//成员方法
	void testA(); //【缺省属性】abstract
}

interface B { 
	void testB();
}

//这里使用implements关键字来遵从接口
public class Demo1 implements A, B {

	@Override
	public void testA() {
		System.out.println("实现接口中要求完成的testA方法");
	}

	@Override
	public void testB() {
		System.out.println("遵从接口B实现testB方法");
		
	}
	public static void main(String[] args) {
		Demo1 d1 = new Demo1();
		
		d1.testA(); //【遵从】接口，并且实现接口中规定的方法，调用该方法
		System.out.println(d1.num); //【遵从】接口，使用接口中的成员变量
		//d1.num = 200; 因为在街口定义的成员变量默认的缺省属性都是public static final 
		//而final修饰的变量，里面的数据是不能发生改变的
		d1.testB();
	}

	
}
```


---

## 三、多态

- 概念如下：
==父类的引用指向子类的对象，或者接口的引用指向【遵从】接口类对象==

- 多态的使用注意事项：
  1. 多态情况下，==父类的引用调用父类和子类同名的普通成员方法，那么调用是子类的方法==
  2. 多态情况下，==父类的引用调用父类和子类同名的普通成员变量，那么调用是父类的成员变量==
  3. 多态情况下，==父类的引用调用父类和子类同名的【静态】成员方法，那么调用的是父类的【静态】成员方法==，没什么用
  4. 多态情况下，==父类的引用【不能】调用子类特有的成员变量==



## 四、统一接口
示例代码一：==在方法中如果需要的参数是一个类对象，那么在这个参数中传入该类的【子类】对象也可以==
```java
package lianxi;

/*
一个动物园：
Animal
	Monkey
	Tiger
	Snake
	
现在需要对所有的动物进行喂食

发现，每一个东西都有一个喂食的方式，而且喂食得方式其实都一样的
归纳总结的思想：能不能把这些方法放到一起啊

思考：能不能实现一个喂食动物的方法

喂食动物，要确定操作的是哪一个类，只能使用Animal类

发现：
	这里要求传入对象是Animal类对象的方法，传入Animal的子类对象也可以，没有任何的报错和提示，直接可以运行
	在方法中如果需要的参数是一个类对象，那么在这个参数中传入该类的【子类】对象也可以
	例如：
	eat(Animal a) 中，将Animal a替换成 Monkey m 方法也正常运行
*/


class Animal {
	int age; 
	
	public Animal() {}
	public Animal(int age) {
		this.age = age;
	}	
	
	public void eat(Animal a) {
		System.out.println(a.getClass() + "在吃东西"); // getclass表示获取类名
	}
	
	public Animal tellMyWhoAreYou(Animal a) {
		System.out.println(a.getClass());
		return a;
	}
}

class Monkey extends Animal {	
}

class Tiger extends Animal {
}

class Snake extends Animal {	
}

public class Demo1 {
	public static void main(String[] args) {
		Monkey m = new Monkey();
		m.eat(m);
	
		Tiger t = new Tiger();
		t.eat(t);
		
		Snake s = new Snake();
		s.eat(s);

        System.out.println("---------------------------------------");
        
		Animal a = new Animal();
		
		a.eat(m); 
		//这里需求的是一个Animal 的类对象，但传入是Monkey类对象，
	   //Monkey类和Animal类直接关系是，Monkey 是 Animal的一个子类
		a.eat(t);
		a.eat(s);

        System.out.println("---------------------------------------");
        
		//原来的表达式：Animal ma = a.tellMyWhoAreYou(s); 
		//**************重要******************
		Monkey ma = (Monkey) a.tellMyWhoAreYou(m);
		System.out.println(ma);
		
		Snake ms = (Snake) a.tellMyWhoAreYou(s);
		Tiger mt = (Tiger) a.tellMyWhoAreYou(t);
	}
	
}

```
程序运行结果：
```java
class lianxi.Monkey在吃东西
class lianxi.Tiger在吃东西
class lianxi.Snake在吃东西
---------------------------------------
class lianxi.Monkey在吃东西
class lianxi.Tiger在吃东西
class lianxi.Snake在吃东西
---------------------------------------
class lianxi.Monkey
lianxi.Monkey@2a33fae0
class lianxi.Snake
class lianxi.Tiger
```



示例代码二：==在一个方法中，定义的参数格式是一个接口，传入的参数是【遵从】该接口的对象，没有问题==
```java
package com.qfedu.c_duotai;

//规定一个USB接口。在USB接口中要求所有的USB设备要完成connect()
interface USB {
	void connect();
}

//U盘【遵从】了USB接口，实现了connect方法
class UPan implements USB {
	@Override
	public void connect() {
		System.out.println("U盘连接电脑，传输数据");
	}
}

//键盘【遵从】了USB接口，实现了connect方法
class Keyboard implements USB {
	@Override
	public void connect() {
		System.out.println("推荐filco键盘，有且是茶轴");
	}
}

class Computer {
	//电脑上留有一个USB接口，但是是什么设备来使用USB不确定，根据谁要连接来确定，但是连接的设备
	//必须【遵从】USB接口
	public void USBConnect(USB usb) {
		usb.connect();
	}
}

public class Demo2 {
	public static void main(String[] args) {
		Computer MBP = new Computer();
		
		//传入的是一个【遵从】USB接口的 U盘匿名对象
		MBP.USBConnect(new UPan());
		
		//传入的是一个【遵从】USB接口的键盘匿名对象
		MBP.USBConnect(new Keyboard());
		
		/*
		 定义的形式参数是一个接口，但是传入的是【遵从】接口的类对象 
		 */
	}
}



```



示例代码三：多态：父类引用指向子类对象；

```java

package com.qfedu.c_duotai;

class Father {
	String name; //父类中的成员变量
	int weight = 90; //和子类同名的成员变量
	
	public Father() {}
	 
	public Father(String name) {
		this.name = name;
	}
	
	public void game() {
		System.out.println("曼切斯特");
	}
	
	public static void work() {
		System.out.println("机械工程师");
	}
}

class Son extends Father {
	int age = 16; //子类自己的成员变量 
	int weight = 74; // 和父类同名的成员变量
	
	public Son(String name) {
		super(name);
	}
	
	@Override
	public void game() {
		System.out.println("Housten Rocket");
	}
	
	public static void work() {
		System.out.println("逗比程序猿导师");
	}
}

public class Demo3 {
	public static void main(String[] args) {
//		Son s = new Son("David");
//		s.game();
//		s.work();
//		
//		Father f = new Father("Jack");
//		f.game();
//		f.work();
		
		//父类引用指向子类的对象 【多态】
		Father ftoS = new Son("23333");
		
		ftoS.game();
		
		System.out.println(ftoS.weight);
		
		ftoS.work();
		
//		System.out.println(ftoS.age);
	}
}

```
程序运行结果：
```java
Housten Rocket
90
机械工程师
```
