---
tags : 
- java基础

flag: blue
---

@toc

# JavaDay12  继承

## 一、继承初步
在代码中继承使用的关键字是extends，如果一个类通过extends关键字继承了其他类，那么就可以说，当前类是其他类的子类，或者其他类是当前类父类

- 【发现】
1.  在==创建子类对象的时候，会首先自动调用父类的构造方法==，这里是为了初始化属于父类的成员变量。
2. 父类中的【非私有化】成员变量，子类可以通过继承之后得到使用的权限 
3. 父类中的【非私有化】成员方法，子类可以通过继承之后得到使用的权限
4. 在父类中用private修饰的私有化成员方法，这个私有化成员方法不能通过继承该类的子类对象来调用
       原因:私有化的成员方法，只能在当前类的内部使用，外部没有任何使用权限
5. 父类中用private修饰的私有化成员变量，这个私有化的成员变量不能通过继承该类的子类对象来调用
     原因：私有化的成员变量，只能在当前类的内部使用，外部没有任何使用权限
     
- 【总结】
在继承中，父类里没有私有化的成员变量和成员方法都可以被子类继承，但是一旦私有化，子类就无法继承。这些属性或者方法， 原理是封装思想。

- 【使用继承的注意事项】
继承可以节省代码，提高开发效率，但是在使用继承的时候，前提条件就是，两个类之间的确包含有继承的关系

- 示例代码：
```java
package lianxi;

class Hero {
	int blood;
	//父类中私有化的成员变量
	private int power;
	
	public Hero() {
		System.out.println("父类 Hero类的无参构造方法");
	}
	
	public Hero(int blood, int power) {
		this.blood = blood;
		this.power = power;
		System.out.println("父类 Hero类的有参构造方法");
	}
	
	public Hero(String name) {
		System.out.println("父类有参构造方法");
		
	}
	
	
	//父类Hero类里面成员方法
	public void D() {
		System.out.println("闪现~~~");
	}
	
	public void F() {
		System.out.println("屏障~~~");
	}
	
	private void test() {
		System.out.println("父类中私有化的方法");
	}
}

//extends 继承的关键字，表示当前VN类是Hero类的一个子类，或者说Hero类是VN类的父类
class VN extends Hero {
	String name; //子类的成员变量 英雄名字
	
	public VN() {
		System.out.println("子类 VN类的无参构造方法");
	}
	
	public VN(String name) {
		this.name = name;
		System.out.println("子类VN类的有参构造方法");
	}
	
	//子类自己的方法
	public void R() {
		System.out.println("终极时刻");
	}
	
}

public class Demo2 {
	public static void main(String[] args) {
		//创建一个父类的对象
//		Hero h = new Hero();
//		System.out.println(h);
		
		//创建一个继承于Hero类的一个子类 VN类对象
		VN vn = new VN();
		System.out.println(vn);
		
		VN cVn = new VN("zhangsan");
		System.out.println(cVn);
		
		//子类对象使用继承于父类得来的【非私有化】成员变量
		vn.blood = 100;
		
		//子类不能使用父类中的私有化成员变量
		//vn.power = 200;
		
		//子类使用自己的成员变量
		vn.name = "暗夜猎手";
		
		//子类自己的成员方法
		vn.R();
		
		//子类通过继承获取到使用父类【非私有化】成员方法的权限
		vn.D();
		vn.F();
		
		//子类不能使用父类的私有化的成员方法，也就说没有继承的权限
		//vn.test();
	}
}
```
程序结果：
```java
父类 Hero类的无参构造方法
子类 VN类的无参构造方法
lianxi.VN@7c53a9eb
父类 Hero类的无参构造方法
子类VN类的有参构造方法
lianxi.VN@ed17bee
终极时刻
闪现~~~
屏障~~~
```

![继承内存分析图]($resource/%E7%BB%A7%E6%89%BF%E5%86%85%E5%AD%98%E5%88%86%E6%9E%90%E5%9B%BE%202.jpg)

---

## 二、super关键字 
  ==子类在创建对象时候，首先会自动调用父类的构造方法==
==【前提】父类的构造方法，不能被子类继承，但是子类可以使用==

[问题]
在子类中如果通过构造方法初始化从父类继承而来的成员变量，可能会存在一定的隐患，如果直接使用this.成员变量赋值操作，会导致父类中的成员变量不符合一些业务逻辑，或者生活逻辑。

[考虑]
**能否借助于父类的构造方法，来初始化原本属于父类的成员变量**

[解决]
借助于 super 关键字 ： 调用父类方法的关键字

super关键字的注意事项：
1.  ==super关键字可以在子类中直接调用父类的成员方法==；`super.成员方法;`

2. 【重点】使用super调用父类的构造方法：`super(实际参数); `
Java编译器会根据不同的参数类型，来调用不同的父类中的构造方法

3. 使用super关键字调用父类的构造方法的时候，要求必须当前代码块的第一行

4. this关键字调用构造方法，和super关键字调用构造方法，不能同时出现在一个代码块里面

5. ==在子类的构造方法中，如果没有通过super指定调用父类的构造方法，那么Java编译器会帮我们自动调用父类的无参构造方法来使用==。
                     
[建议]**存在继承关系下， 父类中最好提供一个无参的构造方法供子类使用**


[回顾]
	this关键字调用构造方法的格式：
1. this(实际参数)  Java编译器会根据不同的参数类型，来调用不同的构造方法
2. 如果用this调用构造方法，必须在当前代码块的第一行
3. this关键字调用构造方法的时候，不能相互调用


```java
class Fu {
	int age;
	String name;
	
	public Fu() {
		System.out.println("父类无参的构造方法");
	}
	
	public Fu(int age, String name) {
		if (age < 0) {
			this.age = 30;
		} else {
			this.age = age;
		}
		 
		this.name = name;
		System.out.println("父类有参数的构造方法");
	}
	
	public void work() {
		System.out.println("父亲是机械工程师~~~");
	}
}

class Zi extends Fu {
	String hobby;
	
	public Zi() {
		//这里没有用super来调用任何的父类中的构造方法
		System.out.println("子类无参构造方法");
	}
	
	public Zi(String hobby) {
		this.hobby = hobby;
	}
	
	public Zi(String name, int age, String hobby) {
		//name 属性，和age属性是从父类继承而来，既然是父类的属性，就用父类的方法来初始化
		super(age, name); //这里相对于调用父类中两个参数的的构造方法，来初始化父类中的成员变量

		this.hobby = hobby;
	}
	
	public void play() {
		work();
		System.out.println("玩吃鸡~~~");
	}	
}

public class Demo3 {
	public static void main(String[] args) {
		//这里是匿名对象调用方法
		new Zi().play();
	}
	
}

```
程序结果：
```java
父类无参的构造方法
子类无参构造方法
父亲是机械工程师~~~
玩吃鸡~~~
```

---

## 三、重写
[问题]
父类中存在一个playGame()的方法，这个方法是比较符合父类的实际情况
子类继承之后，也能调用playGame()，但是这个方法不太符合子类的情况

[期望]
需要让继承而来的方法， 更加适合子类的实际情况

【重写】
==子类中存在和父类同名，同参数，同返回值的方法，这种情况称之为【重写】==
通常会使用【注解】@Override 开启严格的重写检查
如果父类中没有这个方法，使用@Override检查会报错

重写代码示例一：
```java
class Father {
	int age;
	String name;
	
	public Father() {}
	
	public Father(int age, String name) {
		if (age < 0) {
			this.age = 30;
		} else {
			this.age = age;
		}
		this.name = name;
	}
	
	public void playGame() {
		System.out.println(this.name + "喜欢钓鱼");
	}
}

class Son extends Father {
	int id;
	
	public Son() {}
	
	public Son(int age, String name, int id) {
		//age和name是父类的成员变量，所以使用父类的构造方法来初始化
		super(age, name);
		//子类的成员变量，自己处理
		this.id = id;
	}
	
	@Override
	public void playGame() {
		System.out.println(this.name + "大吉大利，今晚吃鸡");
	}
	
}

public class Demo4 {
	public static void main(String[] args) {
		Father f = new Father(60, "王健林");
		
		f.playGame();
		
		Son s = new Son(29, "王思聪", 666);
		
		s.playGame();
	}
}


```
程序运行结果：
```java
王健林喜欢钓鱼
王思聪大吉大利，今晚吃鸡
```

重写代码示例二：

```java
package com.qfedu.a_extends;

/*
 	定义游戏人物类
 	 都有QWE方法
 	 
 	 不同的英雄有不同的QWE方法
 	 
 	 皮城女警 Q 和平使者 W 约德尔诱捕器  E 90口径绳网
 	 维鲁斯  Q 穿刺之箭 W 枯萎箭袋  E 恶灵箭雨
 */

class LOLHero {
	String name;
	
	public LOLHero() {}
	
	public LOLHero(String name) {
		this.name = name;
	}
	
	public void Q() {
		System.out.println("Q技能");
	}
	
	public void W() {
		System.out.println("W技能");
	}
	
	public void E() {
		System.out.println("E技能");
	}
}

class Caitlyn extends LOLHero{
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

class Verus extends LOLHero{
	@Override
	public void Q() {
		System.out.println("穿刺之箭");
	}
	
	@Override
	public void W() {
		System.out.println("腐败箭袋");
	}
	
	@Override
	public void E() {
		System.out.println("恶灵箭雨");
	}	
}

public class Demo5 {
	public static void main(String[] args) {
		Verus nimingjun = new Verus();
		
		nimingjun.Q();
		nimingjun.E();
		nimingjun.W();
		
		Caitlyn yuye = new Caitlyn();
		
		yuye.Q();
		yuye.E();
		yuye.W();
		
	}
}

```
程序运行结果：
```java
穿刺之箭
恶灵箭雨
腐败箭袋
和平使者
90口径绳网
约德尔诱捕器
```
