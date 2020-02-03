---
tags : 
- java基础

flag: blue
---
@toc
# JavaDay10 构造方法和 this 关键字

## 一、开发工具 Eclipse

- 准备工作：
打开Eclipse 
  - 第一步，会弹出 select a directory as workspace ，选择一个没有中文的路径，要保证自己能够找到；

  - 第二步：反选欢迎界面；

  - 第三步：设置字体大小
    菜单栏 -> Window -> Perferences -> 搜索栏 搜索 font -> 找到 Colors and Fonts -> 右侧 找到 Basic -> Text Font -> 只设置 字体大小；
    同时也可以设置 主体 Theme；

- 创建第一个Java Project
1. 在Package Explorer  鼠标右键 -> new -> Java Project -> 起一个项目名(不能有中文)
2. 在Java Project 当前是Day10 找到 src文件夹 -> 鼠标右键 -> new -> Package ->  packageName要求全小写，通常在项目中，包名是项目域名或者公司域名的倒序
3. 在package 上 当前是com.qfedu 鼠标右键 -> new -> class -> 起一个Class名字 


- 异常处理
1. 界面初始化 菜单栏 -> Window -> perspective -> Reset perspective
   
- 其他 IDE
Eclipse  MyEclipse  IDEA   Hadoop

---

## 二、面向对象 之 构造方法

###   (一)构造方法的功能：
  【初始化类对象】，即给成员变量进赋值操作；

### (二)构造方法的格式：
```java
修饰符 类名(用于初始化的参数列表) {
    //初始化语句体
}
```

###   (三)构造方法需要注意的一些细节;
  1. 构造方法，==没有返回值！！！也不需要写void==
  2. 构造方法的==方法名必须是类名==
  3. 代码中并没有实现【无参数的构造方法】， 但是 Java 编译器会帮助我们自动生成一个无参数的构造方法，前提条件是在类中没有自定义任何的构造方法。
  4. 如果在类中自定义了不管是有参还是无参的构造方法，那么Java编译器就不会再给我们提供无参的构造方法了
  5. 一个类中可以存在多个构造方法，来满足不同的需求和使用情况，不同参数，不同的初始化对象

### (四)反编译工具：javap
  javap -c -l -private XXX.class
  -c: 反编译的选项
  -l: 显示行号
  -private:显示所有的类好成员
  XXX.class：是要进行反编译的class字节码文件
      
### (五)构造方法和成员方法的区别
区别| 构造方法 | 成员方法
---|---|---
返回值区别 | 构造方法没有返回值，也不需要void填充 | 必须有返回值，如果确实没有返回值也要用void填充
方法名区别 | 构造方法的名字是和类名一致的，不能是其他名字 | 符合标识符规范，见名知意，动宾结构
作用的区别 | 构造方法是用来初始化对象 | 是对类对象的一个行为描述，也是完成某些特定的功能
调用者区别 | 构造方法的真实的使用者是JVM，==通过new关键字来使用构造方法== | 成员方法目前是使用对象来调用的，后面会有使用【类名】来调用的

---
## 三、构造代码块

- 代码块种类：
  1. 构造代码块
  2. 局部代码块 (基本不用)
  3. 静态代码块！！！(后期用的还是蛮多的~~~ static JDBC)

- 基本格式
```java
{
  构造代码块；
}
```

【要求】
小孩子出生，创建对象之后，必须得哭

【问题】
现行的处理方式，不太符合生活逻辑，是对象创建完之后来调用cry()方法完成，但是我们期望的是对象和cry()方法是同时的

【瓶颈】
目前处理方式是将cry()方法放到每一个，构造方法中，通过调用构造方法，创建并初始化对象同时调用cry()方法，的确可以解决问题

极端情况，如果有10000个或则非常多的构造方法，但是每一个构造方法都要有这cry()方法

**作用**：
	==不管是通过调用哪一个该类的构造方法来初始化对象，都会自动执行构造代码快里面的代码==
	==等于说构造代码块里面的代码会对【当前类所有的对象】进行初始化操作==

要求：
	==构造代码块放在成员变量之后，构造方法之前==

---

## 四、this 关键字

==代码中：表示调用该方法的【对象】==

### （一）发现
在Eclipse自动生成的Setter和Getter方法中，我们发现，Eclipse会**使用this关键字来区分成员变量还是参数列表**，而且形式参数的名字和成员变量的名字一致 

setter和getter方法，都是通过当前【类的对象】来调用的，而这个this表示的就是调用该方法的【类对象】

 **Java中规定，哪一个对象调用这个方法，哪一个对象就是this** 【80%严谨】

###  （二）this关键字的作用：
1. 解决就近原则，在Java中如果存在成员变量和局部变量的名字是一致的，就可以通过this来约束条件，用this.成员变量名 来区分到底是成员变量还是局部变量

2. ==this可以在类内用来调用其他方法，this.方法名(参数)== 【80%严谨】

### （ 三） this关键字的特殊用法【难点】

#### 1.调用成员方法、成员变量
 this关键字可以调用成员变量，可以使用成员方法。都是通过`.`运算符及：`this.成员参数/成员方法`来调用的

#### 2.调用构造方法
- this关键字调用构造方法的格式:`this(实际参数列表);`
- 用this关键字调用构造方法的注意事项:
  - 参数类型，参数的顺序以及参数的数量要和构造方法中一一对应，这样Java编辑器才能知道到底调用的是哪一个构造方法，这里采用方式是【函数/方法的重载】
  - **在一个构造方法中通过this关键字调用另一个构造方法，那么这条语句必须在当前代码块的第一行**
  - 两个构造方法，不能通过this关键字相互调用。

---

##   五、重载
- 函数/方法的重载：
      前提条件：在同一个类内
      必须条件：同一个函数/方法名
      不同条件：参数类型和参数数量不同，还有返回值不同
  
- 总结：
      重载就是在同一个类当中，同名的函数/方法，但是参数，返回值可以不同，这叫做函数/方法的重载
      这样使用可以减少代码的复杂度，减少函数名的使用
  

示例代码：
```java
/**
 * @since  2019年4月26日
 */

class People {
	private String name;
	private int id;
	private int age;	
	{
		cry();
	}
	
	public People() {
		System.out.println("空构造方法");
	}
	
	public People(String name) {
		this();
		this.name = name;
		System.out.println("含有姓名的构造方法");
	}
	public People(String name,int age) {
		this.name = name;
		this.age = age;
	}
	
	public void cry() {
		System.out.println("哇哇哇哇~~~~");
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}			
}


public class Practice {
	public static void main(String[] args) {
		People child = new People("李四");		
	}
}

```
输出结果：
```
哇哇哇哇~~~~
空构造方法
含有姓名的构造方法
```
