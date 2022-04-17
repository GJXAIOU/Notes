---
date:`2018-11-19-2018-11-19`
---

# C_PP_章二 类和对象

## 一、前言

- C++学习技术路线及目标
   * 研究C++编译器管理类和对象的方法 ===》避免死角
   * c++编译器对类对象的生命周期管理，对象创建、使用、销毁;
   * c++面向对象模型初探;  
   * c++面向对象多态原理探究;
   * 操作符重载;
   
   
- 面向抽象类（接口）编程
![面向抽象类（接口）编程]($resource/%E9%9D%A2%E5%90%91%E6%8A%BD%E8%B1%A1%E7%B1%BB%EF%BC%88%E6%8E%A5%E5%8F%A3%EF%BC%89%E7%BC%96%E7%A8%8B.png)



## 二、类和对象

### （一）基本概念

- 1）类、对象、成员变量、成员函数
- 2）面向对象三大概念
  - 封装、继承、多态
- 3）编程实践
  - 类的定义和对象的定义，对象的使用
  - 求圆形的面积
  - 定义Teacher类，打印Teacher的信息（把类的声明和类的实现分开）



### （二）类的封装

**1.封装（Encapsulation）**

- A）封装，是面向对象程序设计最基本的特性。把数据（属性）和函数（操作）合成一个整体，这在计算机世界中是用类与对象实现的。
- B）封装，把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。

  备注：有2层含义（==把属性和方法进行封装和对属性和方法进行访问控制==）

- C++中类的封装
  - 成员变量，C++中用于表示类属性的变量
  - 成员函数，C++中用于表示类行为的函数

**类的封装的第一层含义：**
```cpp
/*
类的定义以及使用：
*/

//封装的第一层含义：
//将属性和方法进行封装


#include"iostream"
using namespace std;

class my_circle
{
public :
	double m_r;
	double m_s;

public:
	double getR()
	{
		return m_r;
	}

	double setR(double r)
	{
		m_r = r;
		return m_r;
	}

	double getS()
	{
		m_s = 3.14*m_r *m_r;
		return m_s;
	}

};



//使用指针调用类中函数
//通过类的指针可以调用类的成员函数
void printCircle01(my_circle *pc)
{
	cout<<"r"<<pc->getR()<<endl;
	cout << "s" << pc->getS() << endl;

}


void printCircle01(my_circle &pc)  #这里加不加&都行
{
	cout << "r" << pc.getR() << endl;
	cout << "s" << pc.getS() << endl;

}



int main()
{
	my_circle c1, c2;//定义两个对象c1和c2
	c1.setR(10);
	cout << "c1 s: " << c1.getS() << endl;

	//指针

	c1.setR(11);
	printCircle01(&c1);

	c2.setR(20);
	printCircle01(&c2);


	//引用
	printCircle01(c2);

	system("pause");
	return 0;
}

```
程序运行结果：
`c1 s: 314`
`r11`
`s379.94`
`r20`
`s1256`
`r20`
`s1256`

**类的封装的第二层含义：**
- public:修饰的成员变量和成员函数，可以在类的内部和类的外部访问
- private:修饰的成员变量和成员函数，只能在类的内部被访问，不能在类的外部访问
- protected:修饰的成员变量和成员函数，只能在类的内部被访问，不能再类的外部访问;但是可以用在继承之中，子类可以继承父类的protected修饰的变量或者函数
```cpp
/封装的第二层含义：
//对属性和方法进行访问控制

#include"iostream"
using namespace std;

class my_circle
{
private://下面两个成员变量的属性是私有的
	double m_r;
double m_s; //属性没有写（没有权限修饰的）默认等同于私有的

public:
	double setR(double r)
	{
		m_r = r;
		return m_r;
	}

};


int main()
{
	my_circle c1, c2;
	//c1.m_r = 23;   //这个是不可以的

	c1.setR(20);  //这个是可以的
	system("pause");
	return 0;
}
```

**2类成员的访问控制**

- 在C++中可以给成员变量和成员函数定义访问级别
  - Public修饰成员变量和成员函数可以在类的内部和类的外部被访问
  - Private修饰成员变量和成员函数只能在类的内部被访问

* //类是把属性和方法封装 同时对信息进行访问控制
* //类的内部，类的外部
* //我们抽象了一个类，用类去定义对象
* //类是一个数据类型，类是抽象的
* //对象是一个具体的变量。占用内存空间。
```cpp

class Circle
{
public:
	double r;
	double s;

public:
	double getR()
	{
		a++;
		return r;
	}

	void setR(double val)
	{
		r = val;
	}


public:
	double getS() //增加功能时，是在修改类, 修改类中的属性或者是方法
	{
		s = 3.14f*r*r;
		return s;
	}

	//private:

	int a;

};
```



**3.struct和class关键字区别**
- 在用struct定义类时，所有成员的默认属性为public
- 在用class定义类时，所有成员的默认属性为private


**4.类的声明和类的实现分开**
首先在“项目名称”右击，选择“添加”，然后选择“类”
![新建类1]($resource/%E6%96%B0%E5%BB%BA%E7%B1%BB1.png)

在新的弹出窗口内填写类名之后，系统会自动生成两个文件
![添加类]($resource/%E6%B7%BB%E5%8A%A0%E7%B1%BB.png)

Teacher.h
```h
#pragma once//表示给头文件仅包含一次
class Teacher
{
private:
	double m_la;
	int str[39];
public:
	int num_of_class(int i);
	void hahaha();
};

```
Teacher.cpp
//具体实现函数内容，在函数名前面加上`类名::`
```cpp
#include "Teacher.h"

int Teacher::num_of_class(int i)   //这里是具体实现在.h中声明的函数，注意函数名前加上.h的头文件名
{
	i =i + 200;
	return i;
}

void Teacher::hahaha()
{

}
```

在其他函数中使用该类：（相当于主函数）
```cpp
/*

使用Teacher类（类的定义和实现是分来的）
*/

#include"iostream"
#include"Teacher.h"

using namespace std;

int main()
{
	Teacher t1;
	int ans = 0;
	ans = t1.num_of_class(34);
	cout << "ans = " << ans << endl;
	system("pause");
	return 0;
}
```
程序运行结果：`ans = 234`




### （三）C++面向对象程序设计举例

**目标：面向过程向面向对象思想转变**
**初学者要仔细体会类和对象之间的关系，并通过适当练习巩固和提高！**

- 案例1:设计立方体类(cube)，求出立方体的面积和体积
```cpp
/*
案例一：根据长宽高求出长方体的面积与体积
*/

#include"iostream"
using namespace std;

class Cube
{
private:
	int m_a;
	int m_b;
	int m_c;
	int m_s;
	int m_v;

public:
	void set(int a, int b, int c)//因为类的成员变量这里都是private:
	{
		m_a = a;
		m_b = b;
		m_c = c;
	}

	int getS()
	{
		m_s = 2 * (m_a *m_b + m_a * m_c + m_b * m_c);
		return m_s;
	}

	int getV()
	{
		m_v = m_a * m_b *m_c;
		return m_v;
	}

};



int main()
{
	Cube cube;
	cube.set(1, 2, 3);
	int s = 0;
	int v = 0;
	s = cube.getS();
	v = cube.getV();

	cout << "s = " << s << endl;
	cout << "v = " << v << endl;

	system("pause");
	return 0;
}
```
程序运行结果：
`s = 22`
`v = 6`

 - 求两个立方体，是否相等（全局函数和成员函数）
   - 方法一：使用全局函数法
```cpp

/*
判断两个立方体是否相等：

*/

#include"iostream"
using namespace std;

class Cube
{
private:
	int m_a;
	int m_b;
	int m_c;
	int m_s;
	int m_v;

public:
	void set(int a, int b, int c)
	{
		m_a = a;
		m_b = b;
		m_c = c;
	}

	int getS()
	{
		m_s = 2 * (m_a *m_b + m_a * m_c + m_b * m_c);
		return m_s;
	}

	int getV()
	{
		m_v = m_a * m_b *m_c;
		return m_v;
	}

	//因为这里面的m_a,m_b,m_c变量均是private，外部无法直接调用

	int getA()
	{	
		int a = m_a;
		return a;
	}

	int getB()
	{
		int b = m_b;
		return b;
	}

	int getC()
	{
		int c = m_c;
		return c;
	}
};

//使用全局函数法
int judgecube(Cube &cube1, Cube &cube2)
{
	if (cube1.getA()==cube2.getA()  && cube1.getB() == cube2.getB() && cube1.getC() == cube2.getC())
	{
		return 1;
	}
	else
	{
		return 0;
	}

}

int main()
{
	Cube cube1, cube2;
	cube1.set(1, 2, 3);

	cube2.set(1, 2, 4);

	if (judgecube(cube1,cube2) == 0)
	{
		cout << "buxiangdeng " << endl;
	}
	else
	{
		cout << "xiangdeng" << endl;
	}

	system("pause");
	return 0;
}
```
程序运行结果：
`buxiangdeng`

  - 方法二：使用类的成员函数
```cpp

/*
使用类的成员函数进行判别
*/
#include"iostream"
using namespace std;

class Cube
{
private:
	int m_a;
	int m_b;
	int m_c;
	int m_s;
	int m_v;

public:
	void set(int a, int b, int c)
	{
		m_a = a;
		m_b = b;
		m_c = c;
	}

	int getS()
	{
		m_s = 2 * (m_a *m_b + m_a * m_c + m_b * m_c);
		return m_s;
	}

	int getV()
	{
		m_v = m_a * m_b *m_c;
		return m_v;
	}

	//因为这里面的m_a,m_b,m_c变量均是private，外部无法直接调用

	int getA()
	{
		int a = m_a;
		return a;
	}

	int getB()
	{
		int b = m_b;
		return b;
	}

	int getC()
	{
		int c = m_c;
		return c;
	}


	int judgecube(Cube &cube2)
	{
		if (m_a== cube2.getA() && m_b== cube2.getB() &&m_c == cube2.getC()) //因为调用时cube1的类的成员函数，所以他自身的私有成员变量也可以使用
		{
			return 1;
		}
		else
		{
			return 0;
		}
	}

};



int main()
{
	Cube cube1, cube2;
	cube1.set(1, 2, 3);

	cube2.set(1, 2, 4);
	
	int ret = cube1.judgecube(cube2);//使用cube1对象的类的成员函数judgecube去执行

	if ( ret== 0)
	{
		cout << "buxiangdeng " << endl;
	}
	else
	{
		cout << "xiangdeng" << endl;
	}

	system("pause");
	return 0;
}
```
程序运行结果：`buxiangdeng`


案例2 :设计一个圆形类（AdvCircle），和一个点类（Point），计算点在圆内部还是圆外
         即：求点和圆的关系（圆内和圆外）
         
![案例示意图]($resource/%E6%A1%88%E4%BE%8B%E7%A4%BA%E6%84%8F%E5%9B%BE%E2%80%98%E2%80%99.png)

```cpp

/*
设计一个圆形类（AdvCircle），和一个点类（Point），计算点在圆内部还是圆外
即：求点和圆的关系（圆内和圆外）
*/

#include"iostream"
using namespace std;


class myPoint
{
private:
	int x1;
	int y1;

public:
	int set_point(int _x1, int _y1)
	{
		x1 = _x1;
		y1 = _y1;
		return x1, y1;
	}

	int getX1()
	{
		return x1;
	}

	int getY1()
	{
		return y1;
	}

};


class myCircle
{
private:
	int x0;
	int y0;//圆心坐标
	int r;

public:
	int get_circle(int _x0, int _y0, int _r)
	{
		x0 = _x0;
		y0 = _y0;
		r = _r;
		return x0, y0, r;
	}


	

	int judge(myPoint &point)
	{
		int line = (point.getX1() - x0)*(point.getX1() - x0) + (point.getY1() - y0)*(point.getY1() - y0);
		if (line <= r * r)
		{
			return 1;
		}
		else
		{
			return 0;
		}
	}



};



int main()
{
	myCircle circle;
	circle.get_circle(1, 2, 3);

	myPoint point;
	point.set_point(2, 3);

	int x_point = point.getX1();
	int y_point = point.getY1();

	int ret = circle.judge(point);
	if (ret == 0)
			{
				cout << "buzai " << endl;
			}
			else
			{
				cout << "zai" << endl;
			}
		



}
```
程序运行结果：`zai`
案例3: 对于第二个案例，类的声明和类的实现分开
整个文件分为：main.cpp  myCircle.h  myCircle.cpp   myPoint.h   myPoint.cpp
**main.cpp**
```cpp

/*
上个例子中：
类的实现和声明分开
*/

#include"myCircle.h"
#include"myPoint.h"
#include"iostream"
using namespace std;

int main()
{
	myCircle circle;
	circle.get_circle(1, 2, 3);

	myPoint point;
	point.set_point(2, 3);

	int x_point = point.getX1();
	int y_point = point.getY1();

	int ret = circle.judge(point);
	if (ret == 0)
			{
				cout << "buzai " << endl;
			}
			else
			{
				cout << "zai" << endl;
			}
		
}


```
**myCircle.h**
```h
#pragma once

class myPoint;

class myCircle
{
	private:
		int x0;
		int y0;//圆心坐标
		int r;

	public:
		int get_circle(int _x0, int _y0, int _r);

		int judge(myPoint &point);//这里使用到了myPoint类，所以前面需要类的声明
		
};
```

**myCircle.cpp**
```cpp
#include "myCircle.h"
#include"myPoint.h"

class myPoint;

	int myCircle::get_circle(int _x0, int _y0, int _r)//在函数前面加上类名：：
	{
		x0 = _x0;
		y0 = _y0;
		r = _r;
		return x0, y0, r;
	}

	int myCircle::judge(myPoint &point)
	{
		int line = (point.getX1() - x0)*(point.getX1() - x0) + (point.getY1() - y0)*(point.getY1() - y0);
		if (line <= r * r)
		{
			return 1;
		}
		else
		{
			return 0;
		}
	}


```

**myPoint.h**
```h
#pragma once
class myPoint
{
private:
	int x1;
	int y1;

public:
	int set_point(int _x1, int _y1);
	int getX1();
	int getY1();

};


```

**myPoint.cpp**
```cpp
#include "myPoint.h"

	int  myPoint::set_point(int _x1, int _y1)
	{
		x1 = _x1;
		y1 = _y1;
		return x1, y1;
	}

	int myPoint:: getX1()
	{
		return x1;
	}

	int myPoint::getY1()
	{
		return y1;
	}

```








### （四）作业

作业1：编写C++程序完成以下功能：

1）定义一个Point类，其属性包括点的坐标，提供计算两点之间距离的方法；
2）定义一个圆形类，其属性包括圆心和半径；
3）创建两个圆形对象，提示用户输入圆心坐标和半径，判断两个圆是否相交，并输出结果。

作业2：设计并测试一个名为Rectangle的矩形类，其属性为矩形的左下角与右上角两个点的坐标，根据坐标能计算出矩形的面积

作业3：定义一个Tree类，有成员ages（树龄），成员函数grow（int years）对ages加上years，age（）显示tree对象的ages的值。








## 三、对象的构造和析构

**前言**

- 创建一个对象时，常常需要作某些初始化的工作，例如对数据成员赋初值。注意，类的数据成员是不能在声明类时初始化的。
- 为了解决这个问题，**C++编译器提供了构造函数(constructor)来处理对象的初始化。构造函数是一种特殊的成员函数，与其他成员函数不同，不需要用户来调用它，而是在建立对象时自动执行。**

### （一）构造和析构函数

#### **1.构造函数和析构函数的概念**

**有关构造函数**

- 构造函数的定义：
  * 1）C++中的类可以定义与类名相同的特殊成员函数，这种与类名相同的成员函数叫做构造函数；
  * 2）构造函数在定义时可以有参数； 
  * 3）没有任何返回类型的声明。

- 构造函数的调用：
  * 自动调用：一般情况下C++编译器会自动调用构造函数
  * 手动调用：在一些情况下则需要手工调用构造函数

**有关析构函数**

* 析构函数定义及调用
  * 1）C++中的类可以定义一个特殊的成员函数清理对象，这个特殊的成员函数叫做析构函数
  * 语法：~ClassName()
  * 2）析构函数没有参数也没有任何返回类型的声明
  * 3）析构函数在对象销毁时自动被调用
  * 4）析构函数调用机制
     * C++编译器自动调用

代码演示：
```cpp
/*
构造函数和析构函数的定义和使用 
*/

#include"iostream"
using namespace std;

class  Test	
{
public:
	Test()  //无参数，无返回值，构造函数
	{
		cout << "this is gouzaofunction" << endl;
	}

	~Test() //析构函数
	{
		cout << "this is a xigoufunction" << endl;
	}

protected:
private:
};

void objectplay()  //从这里可以更加直接的看到运行周期
{
	Test t1，t2;
//先构建的后析构
}

int main()
{
	objectplay();

	system("pause");
	return 0;
}
```
程序运行结果：
`this is gouzaofunction`
`this is a xigoufunction`

---
构造函数和析构函数用法：可以用于初始化和释放内存空间
```cpp
/*
构建函数和析构函数的实际使用示例；
*/


#pragma warning(disable:4996)

#include"iostream"
using namespace std;

class  Test	
{
public:
	Test()  //构造函数 ，可以用于一些初始化
	{
		a = 100;
		p = (char *)malloc(150);
		strcpy(p, "aabbccddeeffgg");
		cout << "this is gouzaofunction" << endl;
	}

	void print()
	{
		cout << a << endl;
		cout << p << endl;
	}

	~Test() //析构函数 ,可以用于释放函数内存空间
	{
		if (p != NULL)
		{
			free(p);
		}
		cout << "this is a xigoufunction" << endl;
	}

protected:
private:
	int a;
	char *p;
}; 

void objectplay()  //从这里可以更加直接的看到运行周期
{
	Test t1;
	t1.print();

	Test t2;
	t2.print();
//先构建的后析构
}

int main()
{
	objectplay();
	

	system("pause");
	return 0;
}
```


#### **2 C++编译器构造析构方案  PK 对象显示初始化方案**
（相当于使用构造函数和析构函数与使用原始的初始化方案的区别）

**设计构造函数和析构函数的原因**
面向对象的思想是从生活中来，手机、车出厂时，是一样的。
生活中存在的对象都是被初始化后才上市的；初始状态是对象普遍存在的一个状态的

- 普通方案：
  - 为每个类都提供一个public的initialize函数；
  - 对象创建后立即调用initialize函数进行初始化。

* 优缺点分析
  * 1）initialize只是一个普通的函数，必须显示的调用
  * 2）一旦由于失误的原因，对象没有初始化，那么结果将是不确定的
     **没有初始化的对象，其内部成员变量的值是不定的**
  * 3）不能完全解决问题
```cpp
//为什么对象需要初始化 有什么样的初始化方案

#include "iostream"

using namespace std;

/*
思考为什么需要初始化

  面向对象思想来自生活，手机、车、电子产品，出厂时有初始化

  怎么样进行初始化？

方案1：显示调用方法

缺点：易忘、麻烦；显示调用init，不能完全解决问题

*/

class Test21
{
public:
	int m;
	int getM() const { return m; }
	void setM(int val) { m = val; }
	int n;

	int getN() const { return n; }

	void setN(int val) { n = val; }

public:
	int init(int m, int n)
	{
		this->m = m;
		this->n = n;
		return 0;
	}

protected:

private:

};

int main()
{
	int rv = 0;

	Test21 t1; //无参构造函数的调用方法

	Test21 t2;

	//t1.init(100, 200);

	//t2.init(300, 400);

	cout << t1.getM() << " " << t1.getN() << endl;

	cout << t2.getM() << " " << t2.getN() << endl;

	**//****定义对象数组时，没有机会进行显示初始化**

		Test21 arr[3];

	//Test arr_2[3] = {Test(1,3), Test(), Test()};

	system("pause");

	return rv;

}


```

###  (二)构造函数的分类及调用

C++编译器给程序员提供的对象初始化方案，高端大气上档次。

**1.构造参数的定义和调用方式：**
```cpp
/*
构造函数的分类和使用
*/

#include"iostream"
using namespace std;

class Test
{
private:
	int m_a;
	int m_b;

public:
	Test()  //构造无参函数
	{
		m_a = 1;
		m_b = 2;
	}


	Test(int a)
	{
		m_a = a;
		m_b = 0;
	}


	Test(int a ,int b) //构造有参函数
	{
		m_a = a;
		m_b = b;
	}

	Test(const Test& obj) //赋值构造函数,这里的const可加可不加，后面的obj 自己变换  
	{

	}

};


//下面进行调用函数

int main()
{
	//调用无参数的构造函数，本质上这是C++编译器自动的调用构造函数

	Test t1;



	//调用有参数的构造函数

	//调用只有一个参数的构造函数
	//以下两种调用方式等效
	Test t2(1);

	Test t3 = (1, 2, 3, 4);  //逗号表达式，最后传入的参数为最后一个值


	//调用多个参数的构造函数

	Test t4(1, 2); //相当于调用两个参数的构造函数

	//采用直接调用构造函数，即采用手动调用的方式
	
	Test t5 = Test(1, 2);

	system("pause");
	return 0;
}
```
 
三种构造函数的调用方式：
```cpp
/*

显示初始化方案，相当于从不用初始化方案的角度来突出为什么需要使用构造函数

*/

#include"iostream"
using namespace std;

class Test
{
private:
	int a;
	int b;


public:
	void init(int _a, int _b)  //一个相当于初始化的函数
	{
		a = _a;
		b = _b;
	}

};


int  main()
{
	//由上面的类可以看出，并没有提供构造函数
	// 类没有提供构造函数，C++编译器会自动提供另一个默认的构造函数
	//类没有提供赋值构造   函数，C++编译器也会自动的提供另一个默认的赋值构造函数
	Test t1;

	//如果需要对对象t1进行初始化
	int a = 10;
	int b = 20;
	t1.init(a, b);


	//如果定义的对象是一个数组，就需要挨个的进行初始化
	Test array[3];
	array[0].init(1, 2);
	array[1].init(1, 2);
	array[2].init(1, 2);

	system("pause");
	return 0;

}
```



**2.随机构造函数（copy构造函数）调用时机**

- 赋值构造函数的四种调用场景（调用时机）

  - 第1和第2个调用场景  
```cpp
#include "iostream"
using namespace std;
class AA
{
public:
 AA() //无参构造函数 默认构造函数
 {
 cout<<"我是构造函数，自动被调用了"<<endl;
 }
 
 AA(int _a) //无参构造函数 默认构造函数
 {
 a = _a;
 }

  AA(const AA &obj2)

  {

  cout<<"我也是构造函数，我是通过另外一个对象obj2，来初始化我自己"<<endl;

  a = obj2.a + 10;

  }

 ~AA()

 {

 cout<<"我是析构函数，自动被调用了"<<endl;

 }

 void getA()

 {

 printf("a:%d \n", a);

 }

protected:

private:

 int a;

};

//单独搭建一个舞台

void ObjPlay01()

{

 AA a1; //变量定义

 //赋值构造函数的第一个应用场景

 //用对象1 初始化 对象2

 AA a2 = a1; //定义变量并初始化 //初始化法

 a2 = a1; //用a1来=号给a2 编译器给我们提供的浅copy

}

```

  - 第二个应用场景
```cpp
//单独搭建一个舞台

void ObjPlay02()
{

 AA a1(10); //变量定义

 //赋值构造函数的第一个应用场景

 //用对象1 初始化 对象2

 AA a2(a1); //定义变量并初始化 //括号法

 //a2 = a1; //用a1来=号给a2 编译器给我们提供的浅copy

 a2.getA();
}
```
//注意：初始化操作 和 等号操作 是两个不同的概念

**第一个和第二个场景：**
```cpp

/*
赋值构造函数四种使用时机
*/

#include"iostream"
using namespace std;

class Test
{
private:
	int m_a;
	int m_b;

public:
	Test()  //构造无参函数
	{
		m_a = 1;
		m_b = 2;
	}


	Test(int a)
	{
		m_a = a;
		m_b = 0;
	}


	Test(int a ,int b) //构造有参函数
	{
		m_a = a;
		m_b = b;
	}

	Test(const Test& obj) //赋值构造函数
	{
		m_b = obj.m_b + 100;
		m_a = obj.m_a + 100;

	}

	void my_print()
	{
		cout << "m_a" << m_a << "m_b" << m_b << endl;

	}
};



int main()
{
	//第一种调用方法：
	Test t1(1,2);
	Test t2 = t1;  //用t1来初始化t2
	t2.my_print();

	//第二种调用机制：
	Test t3(t1);
	t3.my_print();


	system("pause");
	return 0;
}

```
  - 第3个调用场景
使用实参去初始化形参，调用形参的copy构造函数
```cpp
/*

第三种应用场景
*/
#include "iostream"
using namespace std;

class Location
{
public:

	Location(int xx = 0, int yy = 0)
	{
		X = xx; Y = yy; cout << "Constructor Object.\n";
	}

	Location(const Location & obj)      //拷贝构造函数
	{
		X = obj.X; Y = obj.Y;
		cout << "Copy_constructor called." << endl;
	}

	~Location()
	{
		cout << X << "," << Y << " Object destroyed." << endl;
	}
	int GetX() { return X; } int GetY() { return Y; }

private: int X, Y;
};

//这是业务函数，形参是一个元素
void f(Location p)
{
	cout << "Funtion:" << p.GetX() << "," << p.GetY() << endl;
}

void mainobjplay()
{
	Location A(1, 2);  //形参是一个元素，函数调用，会执行实参变量初始化形参变量

	    f(A);
}

void main()
{
	mainobjplay();

	system("pause");
}
```

  - 第4个调用场景


```cpp
#第四个应用场景

#include "iostream"
using namespace std;

class Location
{
public:

	Location(int xx = 0, int yy = 0)
	{
		X = xx; Y = yy; cout << "Constructor Object.\n";
	}

	Location(const Location & p)      //复制构造函数
	{
		X = p.X; Y = p.Y; cout << "Copy_constructor called." << endl;
	}

	~Location()
	{
		cout << X << "," << Y << " Object destroyed." << endl;
	}

	int GetX() { return X; } int GetY() { return Y; }

private: int X, Y;
};

//alt + f8 排版

void f(Location p)
{
	cout << "Funtion:" << p.GetX() << "," << p.GetY() << endl;
}

//结论一：函数的返回值是一个元素（复杂类型的），返回的是一个新的匿名对象（所以会调用匿名对象类的copy构造函数）
Location g()
{
	Location A(1, 2);
	return A;
}

//对象初始化操作 和 =等号操作 是两个不同的概念

//匿名对象的去和留，关键看，返回时如何接

void mainobjplay()
{
	//若返回的匿名对象，赋值给另外一个同类型的对象，那么匿名对象会被析构

	//Location B;

	//B = g();  //用匿名对象赋值给B对象，然后匿名对象析构

	//若返回的匿名对象，来初始化另外一个同类型的对象，那么匿名对象会直接转成新的对象B，不会被析构
	Location B = g();

	cout << "传智扫地僧测试" << endl;
}

void main()
{
	mainobjplay();

	system("pause");
}


```

![全为]($resource/%E5%85%A8%E4%B8%BA.png)

**3.默认构造函数**

- 二个特殊的构造函数
  - 1）默认无参构造函数
  当类中没有定义构造函数时，编译器默认提供一个无参构造函数，并且其函数体为空

  - 2）默认拷贝构造函数
  当类中没有定义拷贝构造函数时，编译器默认提供一个默认拷贝构造函数，简单的进行成员变量的值复制




###  (三)构造函数调用规则研究

* 1）当类中没有定义任何一个构造函数时，c++编译器会提供默认无参构造函数和默认拷贝构造函数；
* 2）当类中定义了拷贝构造函数时，c++编译器不会提供无参数构造函数；需要自己加上
* 3） 当类中定义了任意的非拷贝构造函数（即：当类中提供了有参构造函数或无参构造函数），c++编译器不会提供默认无参构造函数；需要自己加上 
* 4 ）默认拷贝构造函数成员变量简单赋值；

总结：只要你写了构造函数，那么你必须用。

- **构造析构阶段性总结**
  * 1）构造函数是C++中用于初始化对象状态的特殊函数
  * 2）构造函数在对象创建时自动被调用
  * 3）构造函数和普通成员函数都遵循重载规则
  * 4）拷贝构造函数是对象正确初始化的重要保证
  * 5）必要的时候，必须手工编写拷贝构造函数

========》1个对象的初始化讲完了，增加一个案例。

### （四）深拷贝和浅拷贝

- 默认复制构造函数可以完成对象的数据成员值简单的复制
- 对象的数据资源是由指针指示的堆时，默认复制构造函数仅作指针值复制

**1浅拷贝问题抛出和分析**

深拷贝浅拷贝现象出现的原因
```cpp
/*
如果未定义copy函数，使用C++默认的copy函数，实质上是一种浅拷贝
*/

#define _CRT_SECURE_NO_WARNINGS
#include"iostream"
using namespace std;

class Name
{
public:

	Name(const char *pname)
	{
		size = strlen(pname);

		pName = (char *)malloc(size + 1);

		strcpy(pName, pname);
	}

	//析构函数
	~Name()
	{
		cout << "开始析构" << endl;

		if (pName != NULL)
		{
			free(pName);
			pName = NULL;
			size = 0;
		}
	}

	void operator=(Name &obj3)
	{
		if (pName != NULL)
		{
			free(pName);

			pName = NULL;

			size = 0;
		}

		cout << "测试有没有调用我。。。。" << endl;

		//用obj3来=自己

		pName = (char *)malloc(obj3.size + 1);

		strcpy(pName, obj3.pName);

		size = obj3.size;

	}

protected:

private:

	char *pName;

	int size;

};

//对象的初始化 和 对象之间=号操作是两个不同的概念

void playObj()
{

	Name obj1("obj1.....");

	Name obj2 = obj1; //obj2创建并初始化
	//C++中提供的默认copy 构造函数是浅拷贝

	Name obj3("obj3...");

	//需要重载=号操作符，才能解决带来的浅拷贝问题

	obj2 = obj3; //=号操作，也是一种浅拷贝

	cout << "业务操作。。。5000" << endl;
}

void main()
{
	playObj();

	system("pause");
}
```
//程序在运行的时候理会出错，因为本质上是进行的浅拷贝，一开始释放obj2时候是正常析构的，但是析构obj1的时候就会出错

示意图：
![4]($resource/4.png)

**2浅拷贝程序C++提供的解决方法**

显示提供copy构造函数

显示操作重载=号操作，不使用编译器提供的浅copy

```cpp

/*

C++的默认copy构造函数是浅拷贝的解决方法 
*/


#define _CRT_SECURE_NO_WARNINGS
#include"iostream"
using namespace std;

class Name
{
public:

	Name(const char *pname)
	{
		size = strlen(pname);

		pName = (char *)malloc(size + 1);

		strcpy(pName, pname);
	}
    //解决方法：手工编写拷贝函数，使用深copy
	Name(Name &obj)
	{
		//用obj来初始化自己

		pName = (char *)malloc(obj.size + 1);

		strcpy(pName, obj.pName);

		size = obj.size;
	}

	//析构函数
	~Name()
	{

		cout << "开始析构" << endl;

		if (pName != NULL)

		{

			free(pName);

			pName = NULL;

			size = 0;

		}

	}

	void operator=(Name &obj3)

	{

		if (pName != NULL)

		{

			free(pName);

			pName = NULL;

			size = 0;

		}

		cout << "测试有没有调用我。。。。" << endl;

		//用obj3来=自己

		pName = (char *)malloc(obj3.size + 1);

		strcpy(pName, obj3.pName);

		size = obj3.size;

	}

protected:

private:

	char *pName;

	int size;

};

//对象的初始化 和 对象之间=号操作是两个不同的概念

void playObj()
{

	Name obj1("obj1.....");

	Name obj2 = obj1; //obj2创建并初始化
	//C++中提供的默认copy 构造函数是浅拷贝

	Name obj3("obj3...");

	//重载=号操作符

	obj2 = obj3; //=号操作

	cout << "业务操作。。。5000" << endl;
}

void main()
{
	playObj();

	system("pause");
}

```

![3]($resource/3.png)

---


### （五）多个对象构造和析构

**1.对象初始化列表**

- 1）对象初始化列表出现原因
程序本身是有问题的
```cpp

/*

构造函数的初始化列表
*/

#include"iostream"
using namespace std;

class A 
{
public:
	A(int _a)
	{
		a = _a;
	}

protected:
private:
	int a;
	int b;
};


class B   //在一个类里面组合了一个带有参数的构造函数的类，因为C++编译器要确定分配给B多大的内存，但是没法初始化A定义的两个对象（因为需要调用有参的构造函数）
{
public:   //没有写B的构造函数，按理说应该调用默认的构造函数

protected:
private:
	int b1;
	int b2;
	A a1;  //主要是因为没有方法和没有机会去初始化这两个A定义的对象
	A a2;
};

int main()
{
	A a1(10);
	B objB;
	return 0;

}

```

解决方法：
```cpp
/*
构造函数的初始化列表
*/

#include"iostream"
using namespace std;

class A 
{
public:
	A(int _a)
	{
		a = _a;
	}

protected:
private:
	int a;
};


class B  
{
public:   
//含义：前面就是定义B的构造函数，：后面试两个需要初始化的对象，括号里面是初始化的时候需要传递的参数（初始化的函数在class A中）
	B(int _a, int _b) : a1(1), a2(3)  
	{

	}
//第二种参数传递的方法：
/*
B(int _a, int _b,int m,int n) : a1(m), a2(n)  
	{
      a = _a;
      b = _b;
	}
	
*/
protected:
private:
	int b1;
	int b2;
	A a1;  
	A a2;
};

int main()
{
	A a1(10);
	B objB(1,2);
	//第二种方式：B objB(1,2,4,3)
	return 0;

}

```

**以上程序的操作顺序：**
首先执行被组合对象的构造函数（a1,a2），如果有多个组合对象，则按照定义的顺序，而不是按照初始化列表的顺序（：后面的顺序）
析构函数：和构造函数的调用顺序相反

  - 1.必须这样做：
    - 如果我们有一个类成员，它本身是一个类或者是一个结构，而且这个成员它只有一个带参数的构造函数，没有默认构造函数。这时要对这个类成员进行初始化，就必须调用这个类成员的带参数的构造函数，
    - 如果没有初始化列表，那么他将无法完成第一步，就会报错。

  - 2、类成员中若有const修饰，必须在对象初始化的时候，给const int m 赋值
    - 当类成员中含有一个const对象时，或者是一个引用时，他们也必须要通过成员初始化列表进行初始化，
    - 因为这两种对象要在声明后马上初始化，而在构造函数中，做的是对他们的赋值，这样是不被允许的。

- 2）C++中提供初始化列表对成员变量进行初始化

语法规则
```cpp
Constructor::Contructor() : m1(v1), m2(v1,v2), m3(v3)
{
 // some other assignment operation
}
```


- 3）注意概念
  - 初始化：被初始化的对象正在创建
  - 赋值：被赋值的对象已经存在

- 4）注意：
  - 成员变量的初始化顺序与声明的顺序相关，与在初始化列表中的顺序无关
  - 初始化列表先于构造函数的函数体执行

**说明：**

* 1 C++中提供了初始化列表对成员变量进行初始化
* 2 使用初始化列表出现原因：
  * 1.必须这样做：
    * 如果我们有一个类成员，它本身是一个类或者是一个结构，而且这个成员它只有一个带参数的构造函数，
    * 而没有默认构造函数，这时要对这个类成员进行初始化，就必须调用这个类成员的带参数的构造函数，
    * 如果没有初始化列表，那么他将无法完成第一步，就会报错。
  * 2、类成员中若有const修饰，必须在对象初始化的时候，给const int m 赋值
    * 当类成员中含有一个const对象时，或者是一个引用时，他们也必须要通过成员初始化列表进行初始化，
    * 因为这两种对象要在声明后马上初始化，而在构造函数中，做的是对他们的赋值，这样是不被允许的。

```cpp
//总结 构造和析构的调用顺序

#include "iostream"
using namespace std;

class ABC
{
public:
 ABC(int a, int b, int c)
 {
 this->a = a;
 this->b = b;
 this->c = c;

 printf("a:%d,b:%d,c:%d \n", a, b, c);
 printf("ABC construct ..\n");
 }

 ~ABC()
 {
 printf("a:%d,b:%d,c:%d \n", a, b, c);
 printf("~ABC() ..\n");
 }

protected:
private:
 int a;
 int b;
 int c;
};

class MyD
{
public:
 MyD():abc1(1,2,3),abc2(4,5,6),m(100)

 //MyD()
 {
 cout<<"MyD()"<<endl;
 }

 ~MyD()
 {
 cout<<"~MyD()"<<endl;
 }

protected:
private:
 ABC abc1; //c++编译器不知道如何构造abc1
 ABC abc2;
 const int m;
};

int run()
{
 MyD myD;
 return 0;
}

int main_dem03()
{
 run();
 system("pause");
 return 0;
}

```




### （六）构造函数和析构函数的调用顺序研究

**构造函数与析构函数的调用顺序**

- 1）当类中有成员变量是其它类的对象时，首先调用成员变量的构造函数，调用顺序与声明顺序相同；之后调用自身类的构造函数

- 2）析构函数的调用顺序与对应的构造函数调用顺序相反

### （七）构造函数和析构函数综合练习**

通过训练，把所学知识点都穿起来

**1构造析构综合训练**
```cpp

/*

总结 构造和析构的调用顺序

*/

#include "iostream"
using namespace std;

class ABC
{
public:
	ABC(int a, int b, int c)   //构造函数  //所以最先执行的构造函数是这个abc1，然后再次执行这个，对应于;abc2
	{
		this->a = a;
		this->b = b;
		this->c = c;

		cout << "ABC construct ..\n" << this->a << this->b << this->c << endl;

	}

	~ABC()
	{
		cout << "ABC construct ..\n" << this->a << this->b << this->c << endl;
	}

	int getA()
	{
		return this->a;
	}

protected:
private:
	int a;
	int b;
	int c;
};


class MyD 
{
public:
	MyD() :abc1(1, 2, 3), abc2(4, 5, 6), m(100)  //因此必须使用构造函数的初始化列表    //定义的时候，先定义的abc1，所以应该先指向ABC的构造函数
	{
		cout << "MyD()" << endl;
	}

	~MyD()
	{
		cout << "~MyD()" << endl;
	}

	MyD(const MyD & obj) :abc1(7, 8, 9), abc2(10, 11, 12), m(100)  //copy函数后面也可以跟初始化列表
	{
		cout << "MyD(const MyD &obj)" << endl;
	}

protected:
private:
	ABC abc1; //c++编译器不知道如何构造abc1//这里组合类ABC类，又因为ABC类含有有参构造函数 
	ABC abc2;
	const int m;//因为是const m,所以上面所有的m都进行了赋值
};


int doThing(MyD myel)   //myel是一个元素
{
	cout << "dothing() myel.abc1.a:" << myel.abc1.getA() << endl;
}


int run2()//二
{
	MyD myD;//定义成员变量，这时候会执行MyD的构造函数
	doThing(myD);//因为myel是一个元素，这里是使用实参来初始化形参，调用形参的copy构造函数
	return 0;
}

int run3()
{
	cout << "run3 start.." << endl;

	cout << "run3 end,," << endl;
}


int main()
{
	run2();//一
	//run3();
	system("pause");
	return 0;
}




/*
构造函数运行顺序
1.执行abc1构造函数：cout << "ABC construct ..\n" << this->a << this->b << this->c << endl; 结果为：1 2 3
2.执行abc2构造函数：cout << "ABC construct ..\n" << this->a << this->b << this->c << endl; 结果为：4 5 6
3.执行MyD自己的构造函数：cout << "MyD()" << endl;

至此：MyD myD;这句话执行完毕

doThing(myD);//因为myel是一个元素，这里是使用实参来初始化形参，调用形参的copy构造函数
然后因为这还是一个组合对象，所以对于copy构造函数仍然要向上面一样，挨个执行
1.执行abc1的copy构造函数：cout << "ABC construct ..\n" << this->a << this->b << this->c << endl;7 8 9
2.执行abc2的copy构造函数：cout << "ABC construct ..\n" << this->a << this->b << this->c << endl;10 11  12
3.执行MyD自己的构造函数：cout << "MyD()" << endl;

然后指向doThing的内部函数：cout << "dothing() myel.abc1.a:" << myel.abc1.getA() << endl;  7

*/

/*
析构的过程
首先析构形参myel:
先析构自己：cout << "~MyD()" << endl; 
然后析构10 ,11,12 :
然后析构7,8,9；
至此形参的析构结束；

然后开始析构myD
*/


```


**2匿名对象强化训练**

 demo10_构造析构练习强化.cpp

1） 匿名对象生命周期

2） 匿名对象的去和留

**3匿名对象强化训练**

3） 构造中调用构造

```cpp
/*

构造中调用构造
*/

#include"iostream"
using namespace std;

class MyTest
{
public:
	MyTest(int a, int b, int c)
	{
		this->a = a;
		this->b = b;
		this->c = c;

	}

	MyTest(int a, int b)
	{
		this->a = a;
		this->b = b;

		MyTest(a, b, 10);
	}

	~MyTest()
	{
		cout << "MyTest is ..." << endl;
	}

private:
	int a;
	int b;
	int c;


public:
	int getC() const { return c; }

	void setC(int val) { c = val; }
};

int main()
{
	MyTest t1(1, 2);
	cout << t1.getC() << endl;
	system("pause");
	return 0;
	
}


```

 结论： 构造函数中调用构造函数，是一个蹩脚的行为。
程序内存图：
![搜狗截图20181124125452]($resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20181124125452.png)

### （八）对象的动态建立和释放

**1.new和delete基本语法**
```cpp
/*

new 和delete的使用

*/

#include"iostream"
using namespace std;

void main()
{
	//先分配内存然后赋值
	int *p = new int;  //分配基础类型
	*p = 20;  

	//在定义的时候直接分配内存
	int *p1 = new int(30);
	cout << *p1 << endl;

	system("pause");
	return; 
}

```

- 1）在软件开发过程中，常常需要动态地分配和撤销内存空间，例如对动态链表中结点的插入与删除。在C语言中是利用库函数malloc和free来分配和撤销内存空间的。C++提供了较简便而功能较强的运算符new和delete来取代malloc和free函数。

  **注意： new和delete是运算符，不是函数，因此执行效率高。**

- 2）虽然为了与C语言兼容，C++仍保留malloc和free函数，但建议用户不用malloc和free函数，而用new和delete运算符。new运算符的例子： 
new int;  //开辟一个存放整数的存储空间，返回一个指向该存储空间的地址(即指针)
new int(100);  //开辟一个存放整数的空间，并指定该整数的初值为100，返回一个指向该存储空间的地址 
new char[10];  //开辟一个存放字符数组(包括10个元素)的空间，返回首元素的地址    new int[5][4];  //开辟一个存放二维整型数组(大小为5*4)的空间，返回首元素的地址 float *p=new float (3.14159);  //开辟一个存放单精度数的空间，并指定该实数的初值为//3.14159，将返回的该空间的地址赋给指针变量p

- 3）new和delete运算符使用的一般格式为：
  用new分配数组空间时不能指定初值。如果由于内存不足等原因而无法正常分配空间，则new会返回一个空指针NULL，用户可以根据该指针的值判断分配空间是否成功。
![new运算符动态]($resource/new%E8%BF%90%E7%AE%97%E7%AC%A6%E5%8A%A8%E6%80%81.png)
```cpp

/*
使用malloc /free 与new /delete的区别
*/

#include"iostream"
using namespace std;

class Test
{
public:
	Test();
	~Test();

private:
	int a;
	int b;
	int c;
};

Test::Test()
{
	cout << "执行了构造函数" << endl;
}

Test::~Test()
{
	cout << "执行了析构函数" << endl;
}



int main()
{
	//基础类型
	int *p1 = (int *)malloc(sizeof(int));
	*p1 = 10;
	delete p1;

	int *p2 = new int;
	*p2 = 20;
	free(p2);



	//数组类型（元素为基础类型）
	int *p3 = (int *)malloc(sizeof(int)*10);
	p3[0] = 1;
	delete[]p3;

	int *p4 = new int[10];
	p4[1] = 2;
	free(p4);


	//类
	Test *p5 = (Test *)malloc(sizeof(Test));
	delete p5;

	Test *p6 = new Test(10);
	free(p6);
}


```
**结论：** 
new：不仅分配内存，而且调用了构造函数
delete:不仅释放了内存，而且调用了析构函数

- 4） 应用举例

![5]($resource/5.png)

**2类对象的动态建立和释放**

- 使用类名定义的对象都是静态的，在程序运行过程中，对象所占的空间是不能随时释放的。但有时人们希望在需要用到对象时才建立对象，在不需要用该对象时就撤销它，释放它所占的内存空间以供别的数据使用。这样可提高内存空间的利用率。

-  C++中，可以用new运算符动态建立对象，用delete运算符撤销对象

  比如：

  Box *pt;  //定义一个指向Box类对象的指针变量pt
      pt=new Box;  //在pt中存放了新建对象的起始地址  在程序中就可以通过pt访问这个新建的对象。如 cout<<pt->height;  //输出该对象的height成员 cout<<pt->volume( );  //调用该对象的volume函数，计算并输出体积 C++还允许在执行new时，对新建立的对象进行初始化。如 Box *pt=new Box(12,15,18);

   这种写法是把上面两个语句(定义指针变量和用new建立新对象)合并为一个语句，并指定初值。这样更精炼。

  新对象中的height，width和length分别获得初值12,15,18。调用对象既可以通过对象名，也可以通过指针。

   在执行new运算时，如果内存量不足，无法开辟所需的内存空间，目前大多数C++编译系统都使new返回一个0指针值。只要检测返回值是否为0，就可判断分配内存是否成功。

  ANSI C++标准提出，在执行new出现故障时，就“抛出”一个“异常”，用户可根据异常进行有关处理。但C++标准仍然允许在出现new故障时返回0指针值。当前，不同的编译系统对new故障的处理方法是不同的。

在不再需要使用由new建立的对象时，可以用delete运算符予以释放。如

delete pt; //释放pt指向的内存空间

这就撤销了pt指向的对象。此后程序不能再使用该对象。

如果用一个指针变量pt先后指向不同的动态对象，应注意指针变量的**当前指向**，以免删错了对象。在执行delete运算符时，在释放内存空间之前，自动调用析构函数，完成有关善后清理工作。

**3.编程实践**

//1 malloc free函数 c关键字

// new delete 操作符号 c++的关键字

//2 new 在堆上分配内存 delete

//分配基础类型 、分配数组类型、分配对象

//3 new和malloc 深入分析

混用测试、异同比较

结论： malloc不会调用类的构造函数

 Free不会调用类的析构函数




## 四、静态成员变量和成员函数


思考：每个变量，拥有属性。有没有一些属性，归所有对象拥有？

### （一）静态成员变量

- **1定义静态成员变量**
  - 关键字 **static**  可以用于说明一个类的成员，
    静态成员提供了一个同类对象的共享机制

  - 把一个类的成员说明为 **static**  时，这个类无论有多少个对象被创建，这些对象共享这个 **static**  成员

  - 静态成员局部于类，它不是对象成员

![1]($resource/1.png)

例如：

```cpp

/*
静态成员变量
*/

#include<iostream>
using namespace std;

class counter
{

	static int num; //**声明与定义静态数据成员**

public:

	void setnum(int i) 
	{ 
		num = i; 
	} //成员函数访问静态数据成员

	void shownum() 
	{ 
		cout << num << endl;
	}

};

int counter::num = 0;//声明与定义静态数据成员

void main()
{
	counter a, b;

	a.shownum(); //调用成员函数访问私有静态数据成员
	b.shownum();
	a.setnum(10);
	a.shownum();
	b.shownum();

}

```

![6]($resource/6.png)

**从结果可以看出，访问的是同一个静态数据成员**

- **2使用静态成员变量**
```cpp
//例5-14_使用公有静态数据成员

#include<iostream.h>

class counter
{ 
public :

 counter (int a) { mem = a; }

 int mem; **_//_****_公有数据成员_**

 static int Smem ; **_//_****_公有静态数据成员_**

} ;

int counter :: Smem = 1 ; **_//_****_初始值为1_**

void main()

{  counter c(5);

 int i ;

 for( i = 0 ; i < 5 ; i ++ )

 { **counter::Smem** += i ;

 cout << **counter::Smem** << '\t' ; //访问静态成员变量方法2

 }

 cout<<endl;

 cout<<"c.Smem = "<<**c.Smem**<<endl; //访问静态成员变量方法1

 cout<<"c.mem = "<<**_c.mem_**<<endl;

}

```
使用静态成员函数调用静态成员变量
```cpp
/*
静态成员函数
*/
// 结论：在静态成员函数中可以使用静态成员变量，但是不能使用类中普通的成员变量

#include"iostream"
using namespace std;

class BB
{
public:
	int printC()
	{
		cout << "c:" << c << endl;
		return c;
	}

	int addC()
	{
		c = c+1;
	}


	static void getC()
	{
		cout << "c" << c << endl;  //可以直接在静态成员函数中使用静态成员变量，但是不能直接使用一般变量
	}
private:
	int a;
	int b;
	static int c;

};


int BB::c = 10;  //初始化成员变量值

void main()
{
	BB b1, b2, b3;
	b1.printC();//10
	b2.addC();//11
	b3.printC();//11

	//调用成员函数两种方法；
	b3.getC();//使用对象.
	BB::getC();//使用类::



}

```

### (二)静态成员函数

- **1）概念**

  - 静态成员函数数冠以关键字static

  - 静态成员函数提供不依赖于类数据结构的共同操作，它没有this指针
  - 在类外调用静态成员函数用 `_类名_ ::`作限定词，或通过对象调用

- **2）案例**
![7]($resource/7.png)

- **3）疑难问题：**
静态成员函数中，不能使用普通变量。
//静态成员变量属于整个类的，分不清楚，是那个具体对象的属性。
![成员函数中的疑难问题]($resource/%E6%88%90%E5%91%98%E5%87%BD%E6%95%B0%E4%B8%AD%E7%9A%84%E7%96%91%E9%9A%BE%E9%97%AE%E9%A2%98.png)

**4.3 综合训练**



## 五、 C++面向对象模型初探

**前言**

- C++对象模型可以概括为以下2部分：

  - 1. 语言中直接支持面向对象程序设计的部分，主要涉及如构造函数、析构函数、虚函数、继承（单继承、多继承、虚继承）、多态等等。

  - 2. 对于各种支持的底层实现机制。

  在c语言中，“数据”和“处理数据的操作（函数）”是分开来声明的，也就是说，语言本身并没有支持“数据和函数”之间的关联性。在c++中，通过抽象数据类型（abstract data type，ADT），在类中定义数据和函数，来实现数据和函数直接的绑定。

  概括来说，在C++类中有两种成员数据：static、nonstatic；三种成员函数：static、nonstatic、virtual。

![C++类的组成]($resource/C++%E7%B1%BB%E7%9A%84%E7%BB%84%E6%88%90.png)

### （一）基础知识

C++中的class从面向对象理论出发，将变量(属性)和函数(方法)集中定义在一起，用于描述现实世界中的类。从计算机的角度，程序依然由数据段和代码段构成。

**C++编译器如何完成面向对象理论到计算机程序的转化？**
换句话：C++编译器是如何管理类、对象、类和对象之间的关系
具体的说：具体对象调用类中的方法，那，c++编译器是如何区分，是那个具体的类，调用这个方法那?

思考一下程序结果
```cpp
/*
面向对象模型初探
*/

#include "iostream"
using namespace std;

class C1
{
public:

	int i; //4
	int j; //4
	int k; //4

protected:

private:

}; //12

class C2
{
public:

	int i; //4
	int j; //4
	int k; //4

	static int m; 

public:

	int getK() const { return k; } //方法是属于代码，应该放在代码区
	void setK(int val) { k = val; } 

protected:

private:
}; 

struct S1
{
	int i;
	int j;
	int k;
}; 

struct S2
{
	int i;
	int j;
	int k;
	static int m;
}; 

int main()
{

	printf("c1:%d \n", sizeof(C1));
	printf("c2:%d \n", sizeof(C2));
	printf("s1:%d \n", sizeof(S1));
	printf("s2:%d \n", sizeof(S2));
	system("pause");

}
```
程序运行结果：
`c1:12`
`c2:12`
`s1:12`
`s2:12`

 
### (二)编译器对属性和方法的处理机制

通过上面的案例，我们可以的得出：
1）C++类对象中的成员变量和成员函数是分开存储的
- 成员变量：
  - 普通成员变量：存储于对象中，与struct变量有相同的内存布局和字节对齐方式
  - 静态成员变量：存储于全局数据区中

- 成员函数：存储于代码段中。
问题出来了：很多对象共用一块代码？代码是如何区分具体对象的那？
换句话说：int getK() const { return k; }，代码是如何区分，具体obj1、obj2、obj3对象的k值？

2）C++编译器对普通成员函数的内部处理

![C++编译器对普通成员函数的内部处理]($resource/C++%E7%BC%96%E8%AF%91%E5%99%A8%E5%AF%B9%E6%99%AE%E9%80%9A%E6%88%90%E5%91%98%E5%87%BD%E6%95%B0%E7%9A%84%E5%86%85%E9%83%A8%E5%A4%84%E7%90%86.png)

请仔细思考，并说出你的总结！



### （三）总结

1、C++类对象中的成员变量和成员函数是分开存储的。C语言中的内存四区模型仍然有效！
**2、C++中类的普通成员函数都隐式包含一个指向当前对象的this指针。**
3、静态成员函数、成员变量属于类

静态成员函数与普通成员函数的区别

静态成员函数不包含指向具体对象的指针

普通成员函数包含一个指向具体对象的指针


### （四）this指针

![this指针图示]($resource/this%E6%8C%87%E9%92%88%E5%9B%BE%E7%A4%BA.png)

实验1：若类成员函数的形参 和 类的属性（成员变量）名字相同，通过this指针来解决。
```cpp

/*
this 指针
*/

#include"iostream"
using namespace std;


class Test
{
public:
	Test( int a, int b)//Test(Test *this, int a, int b)//因为这里成员函数的形参和成员变量一样，不用this指针会出现：a = a ,b = b;
	{
		this->a = a;
		this->b = b;
	}

	void printT()
	{
		cout << "a; " << a << endl;
		cout << "b: " << this->b << endl;
	}


    const void hello(int a,int b )
    {
      //下面这两个语句都是错误的
      this->a = 100;
      this ->b = 200;
    }
   //const修饰的this指针，即相当于this指针指向的内存空间不能被修改
   //因为上面的函数会别C++编译器解析为:void hello(const Test *this, int a ,int b)
   //同时上面的const其实下载函数（）外面的任何位置都行
   
protected:
private:
	int a;
	int b;
};
int  main()
{
	Test t1(1, 2);
	t1.printT();
	system("pause");
	return 0;
}
```
程序运行结果：
`a:1`
`b:2`


实验2：类的成员函数可通过const修饰，请问const修饰的是谁


### （五）全局函数PK成员函数

 1、把全局函数转化成成员函数，通过this指针隐藏左操作数

   Test add(Test &t1, Test &t2)===》Test add( Test &t2)
```cpp
/*
全局函数与成员函数
*/


//使用全局函数

#include"iostream"
using namespace std;

class Test	
{

public:
	Test(int a, int b)
	{
		this->a = a;
		this->b = b;
	}

protected:
private:
	int a;
	int b;
};


Test Testadd(Test &t1, Test &t2)
{
	Test tmp(3,4);
	return tmp;
}

int main()
{
	Test t1(1, 2);
	Test t2(3, 4);

	Test t3(2, 3);
	t3= Testadd(t1, t2);
}
```

```cpp
//使用成员函数

#include"iostream"
using namespace std;

class Test
{

public:
	Test(int a, int b)
	{
		this->a = a;
		this->b = b;
	}
	Test Testadd(Test &t2)
	{
		Test tmp(this->a + t2.a, this->b + t2.b);
		return tmp;
	}

protected:
private:
	int a;
	int b;
};



int main()
{
	Test t1(1, 2);
	Test t2(3, 4);

	Test t3 = t1.Testadd(t2);//匿名对象直接转化成t4
	Test t4(3,5);
	t4 = t1.Testadd(t2);  //匿名对象复制给t5
}


```
 2、把成员函数转换成全局函数，多了一个参数

   void printAB()===》void printAB(Test *pthis)

 3、函数返回元素和返回引用
```cpp
Test& add(Test &t2) //*this //函数返回引用,相当于返回自身，谁调用返回谁
 {      
   this->a = this->a + t2.getA();

   this->b = this->b + t2.getB();

   return *this; //*操作让this指针回到元素状态

 }

Test add2(Test &t2) //*this //函数返回元素
 {

 //t3是局部变量

 Test t3(this->a+t2.getA(), this->b + t2.getB()) ;

 return t3;

 }

 void add3(Test &t2) //*this //函数返回元素

 {

 //t3是局部变量

 Test t3(this->a+t2.getA(), this->b + t2.getB()) ;

 //return t3;

 }
  

```




---

## 六、友元

### （一）友元函数
- 首先友元函数是全局函数，在友元函数中可以修改类的私有属性；
- 友元函数在类中的声明位置位于public或者private只下均可；
![8]($resource/8.png)

```cpp
//友元函数：

#include"iostream"
using namespace std;

class A
{
public:
	friend void modify(A *PA,int _a);  //友元函数的声明

	A(int a, int b)
	{
		this->a = a;
		this->b = b;
 	}

	int getA()
	{
		return this->a;
	}


private:
	int a;
	int b;
};

void modify(A *PA,int _a)//后面的参数可加可不加
{
	//PA->a = 100;//可以这样直接修改值
	PA->a = _a;  //通过传递的值进行修改
}


int main()
{
	A a1(1, 2);
	cout << "a:" << a1.getA() << endl;
	modify(&a1,20);

	int aa = a1.getA();

	cout << "aa:" << aa << endl;
}
```
![9]($resource/9.png)


### （二）友元类

- 若B类是A类的友员类，则B类的所有成员函数都是A类的友员函数
- 若B类是A类的友元类，则B类可以直接修改A的成员变量和成员函数
- 友员类通常设计为一种对数据操作或类之间传递消息的辅助类
程序示例：
```cpp
//友元类
 #include"iostream"
using namespace std;

class A
{
public:
	friend class B;//将B 声明为A的友元类，这样B 可以直接修改A的成员变量和成员函数 

	A(int a=0, int b=0)
	{
		this->a = a;
		this->b = b;
 	}

	int getA()
	{
		return this->a;
	}

private:
	int a;
	int b;
};


class B
{
public:

	void set(int a)
	{
		objA.a = a;
	}

	void printa()
	{
		cout << objA.a << endl;
	}

private:
	A objA;
};


int main()
{
	B b1;
	b1.set(200);
	b1.printa();
	system("pause");
	return 0;
}
```
程序运行结果：200

![10]($resource/10.png)



## 七、强化训练

**1 static关键字强化训练题**

某商店经销一种货物。货物购进和卖出时以箱为单位，各箱的重量不一样，因此，商店需要记录目前库存的总重量。现在用C++模拟商店货物购进和卖出的情况。
```cpp
#include "iostream"
using namespace std;

class Goods
{
public:

	Goods(int w) { weight = w; total_weight += w; }

	~Goods() { total_weight -= weight; }

	int Weight() { return weight; };

	static int TotalWeight() { return total_weight; }

	Goods *next;

private:

	int weight;
	static int total_weight;

};

int Goods::total_weight = 0;

//r尾部指针

void purchase(Goods * &f, Goods *& r, int w)
{

	Goods *p = new Goods(w);

	p->next = NULL;

	if (f == NULL) f = r = p;

	else { r->next = p; r = r->next; } //尾部指针下移或新结点变成尾部结点

}

void sale(Goods * & f, Goods * & r)
{

	if (f == NULL) { cout << "No any goods!\n"; return; }

	Goods *q = f; f = f->next; delete q;

	cout << "saled.\n";

}

void main()
{
	Goods * front = NULL, *rear = NULL;

	int w; int choice;

	do
	{
		cout << "Please choice:\n";

		cout << "Key in 1 is purchase,\nKey in 2 is sale,\nKey in 0 is over.\n";

		cin >> choice;

		switch (choice) // 操作选择
		{
		case 1: // 键入1，购进1箱货物
		{ cout << "Input weight: ";
		cin >> w;

		purchase(front, rear, w); // 从表尾插入1个结点

		break;
		}

		case 2:              // 键入2，售出1箱货物
		{ sale(front, rear); break; } // 从表头删除1个结点

		case 0: break;              // 键入0，结束
		}

		cout << "Now total weight is:" << Goods::TotalWeight() << endl;

	} while (choice);
}
```
**2** **数组类封装**

目标：解决实际问题，训练构造函数、copy构造函数等，为操作符重载做准备

数组类的头文件:`Array.h`
```cpp
#pragma once
class Array
{
public:
	Array(int length);
	Array(const Array&obj);
	~Array();
public:
	int length();

	void setData(int index, int value);

	int getData(int index);
private:
	int m_length;
	char *m_space;
};
```

数组类的头文件的实现：`Array.cpp`
```cppp
#include "Array.h"
using namespace std;
#include"iostream"

Array::Array(int length)
{
	if (length < 0)
	{
		length = 0;
	}
	else
	{
		m_length = length;
		m_space = new char[m_length];
	}
}

Array::Array(const Array&obj)
{
   this->m_length = obj.m_length;
   this->m_space = new char [this->m_length];//进行分配内存空间

  for(int i= 0; i < obj.m_length;i++)//数组元素复制， 这里的obj.m_length可以直接替换成m_length
  {
    this->m_space[i] = obj.m_space[i];
  }
}

Array::~Array()
{
	if (m_space != NULL)
	{
		delete[] m_space;
		m_length = 0;
	}
}

void Array::setData(int index, int valude)
{
	m_space[index] = valude;
}

int Array::getData(int index)
{
	return m_space[index];
}

int Array::length()
{
	return m_length; 
}
```

数组类的测试
```cpp
#include "iostream"
#include "Array.h"
using namespace std;

int main()
{
	Array a1(10);
	for (int i = 0; i < a1.length(); i++)
	{
		a1.setData(i, i);
	}
	for (int i = 0; i <  a1.length(); i++)
	{
		printf("array %d: %d\n", i, a1.getData(i));
	}

	Array a2 = a1;

	for (int i = 0; i < a2.length(); i++)
	{
		printf("array %d: %d\n", i, a2.getData(i));
	}
	system("pause");
	return 0;
}
```

**3.小结**

* 类通常用关键字class定义。类是数据成员和成员函数的封装。类的实例称为对象。
* 结构类型用关键字struct定义，是由不同类型数据组成的数据类型。
* 类成员由private, protected, public决定访问特性。public成员集称为接口。
* 构造函数在创建和初始化对象时自动调用。析构函数则在对象作用域结束时自动调用。
* 重载构造函数和复制构造函数提供了创建对象的不同初始化方式。
* 静态成员是局部于类的成员，提供一种同类对象的共享机制。
* 友员用关键字friend声明。友员是对类操作的一种辅助手段。一个类的友员可以访问该类各种性质的成员
* 链表是一种重要的动态数据结构，可以在程序运行时创建或撤消数据元素。



---


## 八、运算符重载

### （一）概念

#### **1.什么是运算符重载**

![11]($resource/11.png)
所谓重载，就是重新赋予新的含义。函数重载就是对一个已有的函数赋予新的含义，使之实现新功能，因此，一个函数名就可以用来代表不同功能的函数，也就是”一名多用”。

运算符也可以重载。实际上，我们已经在不知不觉之中使用了运算符重载。例如，大 家都已习惯于用加法运算符”+”对整数、单精度数和双精度数进行加法运算，如5+8， 5.8 +3.67等，其实计算机对整数、单精度数和双精度数的加法操作过程是很不相同的， 但由于C++已经对运算符”+”进行了重载，所以就能适用于int, float, doUble类型的运算。

又如`<<`是C的位运算中的位移运算符（左移），但在输出操作中又是与流对 象cout 配合使用的流插入运算符，`>>`也是位移运算符(右移），但在输入操作中又是与流对象 cin 配合使用的流提取运算符。这就是运算符重载(operator overloading)。C系统对`<<`和`>>`进行了重载，用户在不同的场合下使用它们时，作用是不同 的。对`<<`和`>>`的重载处理是放在头文件stream中的。因此，如果要在程序中用`<<`和`>>`作流插入运算符和流提取运算符，必须在本文件模块中包含头文件stream(当然还应当包括”using namespace std“)。

现在要讨论的问题是：用户能否根据自己的需要对C++已提供的运算符进行重载，赋予它们新的含义，使之一名多用？

#### **2.运算符重载入门技术推演**

1为什么会用运算符重载机制
//原因 Complex是用户自定义类型，编译器根本不知道如何进行加减，但是编译器给提供了一种机制，让用户自己去完成，自定义类型的加减操作。。。这个机制就是运算符重载机制

用复数类举例，定义一个Complex类,有两个对象，每个对象由两个属性，要求使用对象之间的加法实现对应属性的相加，示例程序如下：
```cpp

/*对于基础数据类型，C++编译器知道如何进行运算，但是对于用户自定义的类型编译器提供了一种
让自定义数据类型进行运算符操作的机制,=》运算符重载机制
*/

#include "iostream"
using namespace std;

class Complex
{
public:
	friend Complex operator+(Complex &c1, Complex &c2);//友元函数声明

	Complex(int a = 0, int b = 0)//构造函数
	{
		this->a = a;
		this->b = b;
	}

	void printCom()
	{
		cout << a << " + " << b << "i " << endl;
	}

private:
  int a;
  int b;
};

/*使用全局函数，通过调用全局函数实现
Complex myAdd(Complex &c1, Complex &c2)
{
 Complex tmp(c1.a+ c2.a, c1.b + c2.b);

 return tmp;
}
*/


//可以简单的想象为将上面全局函数的函数名替换为operator+
Complex operator+(Complex &c1, Complex &c2)
{
	Complex tmp(c1.a + c2.a, c1.b + c2.b);
	return tmp;
}



int  main()
{
	Complex c1(1, 2), c2(3, 4);

	//方法1： 通过调用普通函数实现
	//Complex c3 = myAdd(c1, c2);
	//c3.printCom();


	//方法2  将函数名称替换为：operator+ 
	//使用以下方式进行调用
	//Complex c3 = operator+(c1, c2);
	//c3.printCom();


	//方法3：最常用的调用方式
	Complex c3 = c1 + c2; 
	c3.printCom()

	system("pause");
	return;
}
```
程序运行结果：`4+6i`


---

### （二）运算符重载的限制
![12]($resource/12.png)

重载运算符函数可以对运算符进行新的解释，但是原有的基本语义不变：
* 不改变运算符的优先级
* 不改变运算符的结合性
* 不改变运算符所需要的操作数
* 不能创建新的运算符

---

### （三）运算符重载编程基础
![14]($resource/14.png)
例如:
   //全局函数  完成 +操作符  重载  
    Complex operator+(Complex &c1, Complex &c2)

  //类成员函数  完成 -操作符  重载
    Complex operator-(Complex &c2)

#### **1.运算符重载的两种方法:**
- 运算符可以重载为成员函数了或者友元函数
- 关键区别在于成员函数具有this指针，友元函数没有this 指针
- 不管是成员函数还是友元函数重载，运算符的使用方法相同
- 但是两种的传递参数不同，实现代码也不同，应用场合也不同

- **二元操作符重载的实现：**
![16]($resource/16.png)
 
```cpp
//二元函数操作符的重载的两种方法


/*全局函数、类成员函数方法实现运算符重载步骤
1）要承认操作符重载是一个函数，写出函数名称operator + ()
2）根据操作数，写出函数参数
3）根据业务，完善函数返回值(看函数是返回引用 还是指针 元素)，及实现函数业务
*/

#include "iostream"
using namespace std;

class Complex
{
private:
	int a;
	int b;

	friend Complex operator+(Complex &c1, Complex &c2); //这里是针对全局函数

public:
	Complex(int a = 0, int b = 0)
	{
		this->a = a;
		this->b = b;
	}

	void printCom()
	{
		cout << a << " + " << b << "i " << endl;
	}

	//成员函数实现 - 运算符重载 
	Complex operator-(Complex &c2)//因为是c1调用的，因此这里的this 指向c1
	{
		Complex tmp(this->a - c2.a, this->b -  c2.b);//通过构造函数实现 tmp中成员变量的变化 
		return tmp;
	}

};


//使用全局函数实现  + 运算符重载
Complex operator+(Complex &c1, Complex &c2)
{
	Complex tmp(c1.a + c2.a, c1.b + c2.b);

	return tmp;
}


int  main()
{
	Complex c1(1, 2), c2(3, 4);

	//使用 全局函数
	Complex c3 = c1 + c2;
	c3.printCom();

	//使用成员函数
	Complex c4 = c1.operator-(c2);
	c4.printCom();

	system("pause");
	return 0;
}
```
程序运行结果：
`4 + 6i`
`-2 + -2i`


- **一元函数操作符重载的实现：**
![17]($resource/17.png)

- 实现前置++和前置--
```cpp
//使用成员函数和全局函数实现一元函数运算符的重载

//实现前置++和前置--

#include "iostream"
using namespace std;

class Complex
{
private:
	int a;
	int b;

	friend Complex& operator++(Complex &c1); //这里主要是针对全局函数

public:
	Complex(int a = 0, int b = 0)
	{
		this->a = a;
		this->b = b;
	}

	void printCom()
	{
		cout << a << " + " << b << "i " << endl;
	}

	Complex& operator--()
	{
		this->a--;
		this->b--;
		return *this;   //因为this指针代表c2的地址，所以*this则表示c2本身；
	}

};


//全局函数实现  + 运算符重载
Complex& operator++(Complex &c1)
{
	c1.a++;
	c1.b++;
	return c1;
}

int  main()
{
	Complex c1,c2;

	//使用全局函数实现前置++操作符的重置
	++c1;
	c1.printCom();


	//使用成员函数实现前置--操作符的重置
	--c2;
	c2.printCom();
	
	system("pause");
	return 0;
}

```
程序运行结果：
`1 + 1i`
`-1 + -1i`

- 实现后置++和后置--
```cpp
//使用成员函数和全局函数实现一元函数运算符的重载

//实现后置++ 和后置--

#include "iostream"
using namespace std;

class Complex
{
private:
	int a;
	int b;

	friend Complex operator++(Complex &c1, int); //这里主要是针对全局函数

public:
	Complex(int a = 0, int b = 0)
	{
		this->a = a;
		this->b = b;
	}

	void printCom()
	{
		cout << a << " + " << b << "i " << endl;
	}

	//成员函数实现后置--
	Complex operator--(int)
	{
		Complex tmp = *this;
		this->a--;
		this->b--;
		return tmp;   //因为this指针代表c2的地址，所以*this则表示c2本身；
	}

};


//全局函数实现  ++ 运算符重载
Complex operator++(Complex &c1,int)  //为了和前置++的函数能够共同存在，参数中加了一个占位符从而实现函数重载
{
	//前置++是先使用后++ ，所以应该先返回c1,但是直接使用return c1.会造成程序的直接退出，后面的++无法实现，所以使用临时变量

	Complex tmp = c1;
	c1.a++;
	c1.b++;
	return tmp;
	return c1;
}

int  main()
{
	Complex c1, c2;
	//使用全局函数实现后置++操作符的重置
	c1++;
	c1.printCom();

	//使用成员函数实现后置--操作符的重置
	c2--;
	c2.printCom();


	system("pause");
	return 0;
}

```


**前置和后置运算符总结:**

C++中通过一个占位参数来区分前置运算和后置运算
![18]($resource/18.png)



#### **2.定义运算符重载函数名的步骤**
首先写运算符重载函数的调用，然后根据调用的形式写具体的函数实现；
**一般情况下函数重载使用成员函数实现**
全局函数、类成员函数方法实现运算符重载步骤
 * 1）要承认操作符重载是一个函数，写出函数名称operator+ ()
 * 2）根据操作数，写出函数参数
 * 3）根据业务，完善函数返回值(看函数是返回引用 还是指针 元素)，及实现函数业务

#### **3.友元函数实现操作符重载的应用场景**

**1）友元函数和成员函数选择方法**
- 当无法修改左操作数的类时，使用全局函数进行重载,因为成员函数需要在左边的类中进行修改；
- =, [], ()和->操作符只能通过成员函数进行重载

**2）用友元函数重载 <<和 >>操作符**

* istream 和 ostream 是 C++ 的预定义流类，都不让修改的222222222222222222222222222222222222222222222
* cin 是 istream 的对象，cout 是 ostream 的对象
* 运算符 << 由ostream 重载为插入操作，用于输出基本类型数据
* 运算符 >> 由 istream 重载为提取操作，用于输入基本类型数据
* 用友员函数重载 << 和 >> ，输出和输入用户自定义的数据类型

a）用全局函数方法实现 << 操作符
```pp
//实现<<运算符的重载

//注释内部是一套完整的程序，但是只能实现一次打印输出，新的一套程序可以实现链式输出

#include "iostream"
using namespace std;

class Complex
{
private:
	int a;
	int b;

	//friend  void operator<<(ostream &cout, Complex &c1); //这里主要是针对全局函数

public:
	Complex(int a = 0, int b = 0)
	{
		this->a = a;

		this->b = b;
	}

	void printCom()
	{
		cout << a << " + " << b << "i " << endl;
	}
};

/*这里类似于cout只能有全局函数加上友元函数实现，因为如果要使用成员函数实现，需要在cout 的类：ostream中定义成员函数，
但是这个ostream类是系统隐藏的，不现实；
void operator<<(ostream &cout, Complex &c1)
{
	cout << c1.a << " + " << c1.b << "i" << endl;
}
*/

ostream& operator<<(ostream &cout, Complex &c2)  //要想实现函数返回值当左值，需要返回一个引用
{
	cout << c2.a << " + " << c2.b << "i" << endl;
	return cout;
}


 
int  main()
{
	Complex c1, c2;
	//cout << c1;  //实现将c1中两个元素以a+bi的形式直接打印输出；
	/*
	首先承认运算符重载是函数，所以函数名为：operator<<
	其次因为有左右参数，分别为ostream 和Complex类，所有函数头为：operator<<(ostream&cout,Complex c1)
	然后根据返回值确定最前面的返回值类型
	*/

	cout << c2 << "kdjfkdjfkd";
	//因为<<操作符是从左到右，所以左边执行返回值要当左值再次执行这个函数
	
	system("pause");
	return 0;
}

```

**3）友元函数重载操作符使用注意点**

- a） 友员函数重载运算符常用于运算符的左右操作数类型不同的情况
![19]($resource/19.png)
- b）其他
  - 在第一个参数需要隐式转换的情形下，使用友员函数重载运算符是正确的选择
  - 友员函数没有 this 指针，所需操作数都必须在参数表显式声明，很容易实现类型的隐式转换
  - C++中不能用友员函数重载的运算符有
     = （）  ［］  －>

**4）友元函数案例vector类**
```cpp
#include <iostream>
using namespace std;

//为vector类重载流插入运算符和提取运算符

class vector
{
public:
	vector(int size = 1);

	~vector();

	int & operator[](int i);

	friend ostream & operator << (ostream & output, vector &);

	friend istream & operator >> (istream & input, vector &);

private:
	int * v;

	int len;

};

vector::vector(int size)
{
	if (size <= 0 || size > 100)
	{
		cout << "The size of " << size << " is null !\n"; abort();
	}

	v = new int[size]; len = size;
}

vector :: ~vector()
{
	delete[] v;

	len = 0;
}

int &vector::operator[](int i)
{
	if (i >= 0 && i < len) return v[i];

	cout << "The subscript " << i << " is outside !\n"; abort();
}

ostream & operator << (ostream & output, vector & ary)
{
	for (int i = 0; i < ary.len; i++)
		output << ary[i] << " ";

	output << endl;
	return output;
}

istream & operator >> (istream & input, vector & ary)
{
	for (int i = 0; i < ary.len; i++)
		input >> ary[i];

	return input;
}

void main()
{
	int k;
	cout << "Input the length of vector A :\n";
	cin >> k;
	vector A(k);
	cout << "Input the elements of vector A :\n";
	cin >> A;
	cout << "Output the elements of vector A :\n";
	cout << A;
	system("pause");
}

```

### （四）运算符重载提高

#### **1.运算符重载机制**

C++编译器是如何支持操作符重载机制的?

#### **2.重载赋值运算符=**

* 赋值运算符重载用于对象数据的复制
* operator= 必须重载为成员函数
* 重载函数原型为：
  类型&类名:: operator= ( const  类名 & ) ;

案例：完善Name类，支持=号操作。
![22]($resource/22.png)

  结论:
 1 先释放旧的内存
 2 返回一个引用
 3 =操作符 从右向左
```cpp
//重载 = 操作符

#define _CRT_SECURE_NO_WARNINGS
#include"iostream"
using namespace std;

class Name
{
public:

	Name(const char *pname)
	{
		size = strlen(pname);

		pName = (char *)malloc(size + 1);

		strcpy(pName, pname);
	}

	Name(Name &obj)
	{
		//用obj来初始化自己

		pName = (char *)malloc(obj.size + 1);

		strcpy(pName, obj.pName);

		size = obj.size;
	}

	//析构函数
	~Name()
	{

		cout << "开始析构" << endl;

		if (pName != NULL)

		{

			free(pName);

			pName = NULL;

			size = 0;

		}

	}

	//obj3 = obj1; // C++编译器提供的  等号操作  也属  浅拷贝\
	// obj4 = obj3 = obj1//如果要是实现这种连续的赋值，需要函数返回一个引用
	//obj3.operator=(obj1)//这种是成员函数的调用方法

//具体的实现步骤，先释放obj3的内存,后根据obj1进行分配内存大小，最后将obj1赋值给obj3
	Name& operator=(Name &obj1)
	{
		//1 先释放obj3旧的内存

		if (this->pName != NULL)
		{
			delete[] pName;
			size = 0;

		}

		//2 根据obj1分配内存大小

		this->size = obj1.size;
		this->pName = new char[size + 1];

		//3把obj1赋值给obj3

		strcpy(pName, obj1.pName);
		return *this;

	}

protected:

private:

	char *pName;

	int size;

};

//对象的初始化 和 对象之间=号操作是两个不同的概念

void playObj()
{

	Name obj1("obj1.....");

	Name obj2 = obj1; //obj2创建并初始化
	//C++中提供的默认copy 构造函数是浅拷贝

	Name obj3("obj3...");

	//重载=号操作符

	obj2 = obj3; //=号操作

	cout << "业务操作。。。5000" << endl;
}

int main()
{
	playObj();

	system("pause");
}

```


#### **3.重载数组下表运算符[]** ==这里的视频再看一遍==
- 重载[]和()运算符
  - 运算符 [] 和 () 是二元运算符
  - [] 和 () 只能用成员函数重载，不能用友元函数重载

- 重载下标运算符 []
  - [] 运算符用于访问数据对象的元素
  重载格式：  返回值类型 类 :: operator[] ( 类型 ) ；
###   设 x 是类 X 的一个对象，则表达式 `x[y]` 可被解释为  `x.operator[](y)`
- 两种使用方法：
  - 返回值当右值，相当于返回一个元素；
  - 返回值当左值，相当于返回一个引用；
  
![33]($resource/33.png)
![44]($resource/44.png)


#### **4.重载函数调用符()**

() 运算符用于函数调用

重载格式:  返回值类型 类:: operator() ( 表达式表 ) ；

例1
设 x是类 X的一个对象，则表达式*
x ( arg1, arg2, … )
可被解释为
x . operator () (arg1, arg2, … )

**案例：**
例2：用重载()运算符实现数学函数的抽象
```cpp
#include <iostream>
using namespace std;
class F
{
public:

	double operator ( )  (double x, double y);
};

double F :: operator ( )  (double x, double y)
{
	return x * x + y * y;
}

void main()
{
	F f;
	//对象名后面（参数）这种形式要不就是调用了构造函数，要不就是运算符重载

	cout << f(2, 4) << endl;  // f.operator()(5.2, 2.5)
}
```
比较普通成员函数

**//例3用重载()运算符实现 pk成员函数**
```cpp
#include <iostream.h>

class F

 { public :

 double memFun ( double x , double y ) ;

 } ;

double F :: memFun ( double x , double y )

 { return x * x + y * y ; }

void main ( )                     

{

F f ;

 cout << f.memFun ( 5.2 , 2.5 ) << endl ;

}
```

#### **5.为什么不要重载&&和||操作符**

- 理论知识：
  - 1）&&和||是C++中非常特殊的操作符
  - 2）&&和||内置实现了短路规则
  - 3）操作符重载是靠函数重载来完成的
  - 4）操作数作为函数参数传递
  - 5）C++的函数参数都会被求值，无法实现短路规则
```cpp
#include <cstdlib>
#include <iostream>
using namespace std;

class Test
{
	int i;
public:
	Test(int i)
	{
		this->i = i;
	}

	Test operator+ (const Test& obj)
	{
		Test ret(0);
		cout << "执行+号重载函数" << endl;
		ret.i = i + obj.i;
		return ret;
	}

	bool operator&& (const Test& obj)
	{
		cout << "执行&&重载函数" << endl;
		return i && obj.i;
	}

};

// && 从左向右
int  main()
{
	int a1 = 0;
	int a2 = 1;
	cout << "注意：&&操作符的结合顺序是从左向右" << endl;

	if (a1 && (a1 + a2))
	{
		cout << "有一个是假，则不在执行下一个表达式的计算" << endl;
	}

	Test t1 = 0;
	Test t2 = 1;

	if(t1 && (t1 + t2))
	{
		t1.operator&&(t1 + t2);
		t1.operator&&(t1.operator+(t2));

			//t1 && t1.operator+(t2)//首先执行加号运算符重载
			// t1.operator( t1.operator(t2) )//再执行&&的运算符重载
			cout << "两个函数都被执行了，而且是先执行了+" << endl;
	}
	system("pause");
	return 0;
}

```
程序运行结果：
`注意：&&操作符的结合顺序是从左向右`
`执行+号重载函数`
`执行&&重载函数`


### （五）运算符重载在项目开发中的应用

#### **1实现一个数组类**
添加<< >>

#### **2实现一个字符串类**
//构造函数要求
//C语言中 没有字符串这种类型，是通过数组来模拟字符串
//C++中 我们来设计一个字符串类 以零结尾的字符串

具体函数实现：
- 类的头文件：
```cpp
#pragma once
#include "iostream"
using namespace std;

class MyString
{
public:
	MyString();
	MyString(const char *p);
	MyString(const MyString &s);
	~MyString();
private:
	int m_len;
	char* m_p;

public:
	MyString& operator=(const char *p);
	MyString& operator= (const MyString &s);

	char& operator[](int index);

	friend ostream & operator<<(ostream &out, MyString &s);

	bool operator==(const char *p);
	bool operator!=(const char *p);
	bool operator==(const MyString&s);
	bool operator!=(const MyString &s);

	int operator<(const char *p);
	int operator<(const MyString &s);

	char *c_str()
	{
		return m_p;
	}
	//这个函数相比上面的区别只是得到的指针不能改变，在下面具体调用中嗲用
	const char *c_str2()
	{
		return m_p;
	}

	int length()
	{
		return m_len;
	}
};

```

头文件类的实现：
(函数前面加的是域作用符)
```cppp
#include "MyString.h"
#include"math.h"
#include "iostream"
#pragma warning(disable:4996)


MyString::MyString() //构造函数：将默认的字符串进行默认为空字符串
{
	m_len = 0;
	m_p = new char[m_len + 1];
	strcpy(m_p, " ");
}

MyString::MyString(const char *p)
{
	if (p == NULL)
	{
		m_len = 0;
		m_p = new char[m_len + 1];
		strcpy(m_p, "");
	}
	else
	{
		m_len = strlen(p);
		m_p = new char[m_len + 1];
		strcpy(m_p, p);
	}
}

//拷贝构造函数
//实现MyString s3 = s2;
 MyString::MyString(const MyString &s)
{
	m_len = s.m_len;   
	m_p = new char[m_len + 1];  //分配空间
	strcpy(m_p, s.m_p);
}


//析构函数的实现
 MyString::~MyString()
 {
	 if (m_p != NULL)
	 {
	、
	
		 delete[] m_p;
		 m_p = NULL;
		 m_len = 0;
	 }
 }

//下面进行的是操作符重载
	//等号=操作符重载
 MyString&MyString:: operator=(const char *p)
	{
		//用于实现s4 = "s2222"
		//因为s4已经分配内存，应该先将旧的内存空间删掉然后再分配新的
		
		//释放旧内存
		if (m_p != NULL)
		{
			delete[] m_p;
			m_len = 0;
		}

		//分配新的内存
		if (p == NULL)
		{
			m_len = 0;
			m_p = new char[m_len + 1];
			strcpy(m_p, "");
		}
		else
		{
			m_len = strlen(p);
			m_p = new char[m_len + 1];
			strcpy(m_p, p);
		}
		return *this;
	}



 MyString&MyString:: operator= (const MyString &s)
	{
		//用于实现s4 = s2
		if (m_p != NULL)
		{
			delete[] m_p;
			m_len = 0;
		}

		//根据s分配新的内存
		m_len = s.m_len;
		m_p = new char[m_len + 1];
		strcpy(m_p, s.m_p);
	
		return *this;
	}



 char&MyString::operator[](int index)
 {
	 return m_p[index];
 }


 //注意这个是全局函数，所以函数名前面不能加上MyString::  
 ostream& operator<<(ostream &out, MyString &s)
 {
	 cout << s.m_p;
	 return out;
 }



 //下面是实现==和!= 的重载，其中分为类和字符串的比较与类和类的比较
 bool MyString::operator==(const char *p)
 {
	 if (p == NULL)
	 {
		 if (m_len == 0)
		 {
			 return true;
		 }
		 else
		 {
			 return false;
		 }
	 }
	 else
	 {
		 if (m_len == strlen(p))
		 {
			 return !strcmp(m_p, p);
		 } 
		 else
		 {
			 return false;
		 }
	 }
	 return true;
 }

 bool MyString::operator!=(const char *p)
 {
	 return !(*this == p);
 }


 //两个类之间的比较
 bool MyString::operator==(const MyString&s)
 {
	if (m_len != s.m_len)
	{
		return false;
	}
	return !strcmp(m_p, s.m_p);
 }

 bool MyString::operator!=(const MyString &s)
 {
	 return !(*this == s);
 }






 //实现  <  的重载

 int MyString::operator<(const char *p)
 {
	 return strcmp(this->m_p, p);
 }
 int MyString::operator<(const MyString &s)
 {
	 return strcmp(this->m_p, s.m_p);
 }
```

函数调用实现：
```cppp
// 实现一个字符串类


//C语言中 没有字符串这种类型，是通过数组来模拟字符串

//C++中 我们来设计一个字符串类 以零结尾的字符串

//若len为0,表示空串

#include "iostream"
#include "MyString.h"
using namespace std;
#pragma  warning (disable: 4996)


int main()
{
	MyString s1;
	MyString s2("s2");
	MyString s2_2 = NULL;
	MyString s3 = s2;




	//下面进行操作符重载
//=操作符
	//两种调用方式；
	MyString s4 = "adfdfdn";
	
	s4 = "s2222"; 

	//调用方式二；
	s4 = s2;




//实现[]重载
  //当[]当右值的时候
	s4[1] = 'a';
	cout << "s4[1] = " << s4[1] << endl;
	
	

//实现<<操作符的重载
	cout << s4 << endl;   //相当于实现字符串的整体输出



//实现== 和!= 的重载
	MyString s5 = "ahhhh";
	
	if (s5 == "shhsk")
	{
		cout << "true" << endl;
	}
	else
	{
		cout << "false" << endl;
	}

	if (s5 != "sjfddsj")
	{
		cout << "false" << endl;
	}
	else
	{
		cout << "true" << endl;
	}
	
	//两个类之间做判断
	
	if (s5 == s2)
	{
		cout << "true" << endl;
	}
	else
	{
		cout << "false" << endl;
	}

	if (s5 != s2)
	{
		cout << "false" << endl;
	}
	else
	{
		cout << "true" << endl;
	}



//实现大于小于号的符号重载

	MyString s6 = "skdjfkld";
	if (s6 < "kdjfkdj")
	{
		cout << "s6 小于 skdjfkld" << endl;
	} 
	else
	{
		cout << "s6 大于 skdjfkld" << endl;
	}


	if (s6 < s5)
	{
		cout << "s6 小于 s5" << endl;
	}
	else
	{
		cout << "s6 大于 s5" << endl;
	}





	//使用类中的private:的指针

	MyString s7 = "jdkfjdklfjdl";
	strcpy(s7.c_str(), "lskjdfkljdklf");
	cout << s7 << endl;
}

```


#### **3智能指针类编写**

1问题抛出
  指针使用过程中，经常会出现内存泄漏和内存多次被释放常

2解决方案：例如：boost库的智能指针
  项目开发中，要求开发者使用预先编写的智能指针类对象代替C语言中的原生指针

3智能指针思想
  工程中的智能指针是一个类模板
  通过构造函数接管申请的内存
  通过析构函数确保堆内存被及时释放
  通过重载指针运算符* 和 -> 来模拟指针的行为
  通过重载比较运算符 == 和 != 来模拟指针的比较

```cpp
class Test
{
public:
 Test()
 {
   this->a = 10;
 }

 void printT()
 {
   cout<<a<<endl;
 }

private:
 int a;
};

class MyTestPointer
{
public:

public:
 MyTestPointer()
 {
   p = NULL;
 }

 MyTestPointer(Test* p)
 {
   this->p = p;
 }

 ~MyTestPointer()
 {
   delete p;
 }

 Test* operator->()
 {
   return p;
 }

 Test& operator*()
 {
   return *p;
 }

protected:
 Test *p;
};

void main01_classp()
{
 Test *p = new Test;
 p->printT();
 delete p;
 MyTestPointer myp = new Test; //构造函数
 myp->printT(); //重载操作符 ->
};

```

```cpp
class MyIntPointer
{
public:
 MyIntPointer()
 {
   p = NULL;
 }

 MyIntPointer(int* p)
 {
   this->p = p;
 }

 ~MyIntPointer()
 {
   delete p;
 }

 int* operator->()
 {
   return p;
 }

 int& operator*()
 {
   return *p;
 }

protected:
 int *p;
};

void main02_intp()
{
 int *p = new int(100);
 cout<<*p<<endl;
 delete p;
 MyIntPointer myp = new int(200);
 cout<<*myp<<endl; //重载*操作符
};
```

**8.7 附录：运算符和结合性**

![55]($resource/55.png)
![66]($resource/66.png)

- 总结
  * 操作符重载是C++的强大特性之一
  * 操作符重载的本质是通过函数扩展操作符的语义
  * operator关键字是操作符重载的关键
  * friend关键字可以对函数或类开发访问权限
  * 操作符重载遵循函数重载的规则
  * 操作符重载可以直接使用类的成员函数实现
  * =, [], ()和->操作符只能通过成员函数进行重载
  * ++操作符通过一个int参数进行前置与后置的重载
  * C++中不要重载&&和||操作符






