# C_PP_章三 继承和派生


## 一、继承概念

面向对象程序设计有4个主要特点：抽象、封装、继承和多态性。我们已经讲解了类和对象，了解了面向对象程序设计的两个重要特征一数据抽象与封装，已经能够设计出基于对象的程序，这是面向对象程序设计的基础。

要较好地进行面向对象程序设计，还必须了解面向对象程序设计另外两个重要特 征——继承性和多态性。本章主要介绍有关继承的知识，多态性将在后续章节中讲解。

继承性是面向对象程序设计最重要的特征，可以说，如果没有掌握继承性，就等于没有掌握类和对象的精华，就是没有掌握面向对象程序设计的真谛。

### （一）类之间的关系

has-A，uses-A 和 is-A

has-A 包含关系，用以描述一个类由多个“部件类”构成。实现has-A关系用类成员表示，即一个类中的数据成员是另一种已经定义的类。

uses-A 一个类部分地使用另一个类。通过类之间成员函数的相互联系，定义友员或对象参数传递实现。

is-A 机制称为“继承”。关系具有传递性,不具有对称性。

### （二）继承关系举例

万事万物中皆有继承，是重要的现象 

两个案例：1）植物继承图；2）程序员继承图
![1]($resource/1.png)

### （三）继承相关概念

![2]($resource/2.png)

### （四）派生类的定义

![3]($resource/3.png)

注意：C++中的继承方式（public、private、protected）会影响子类的对外访问属性。


### （五）继承重要说明

1、子类拥有父类的所有成员变量和成员函数
2、子类可以拥有父类没有的方法和属性
3、子类就是一种特殊的父类
4、子类对象可以当作父类对象使用

--------

## 二、派生类的访问控制

派生类继承了基类的全部成员变量和成员方法（除了构造和析构之外的成员方法），但是这些成员的访问属性，在派生过程中是可以调整的。



### （一）单个类的访问控制

1、类成员访问级别（public、private、protected）

2、思考：类成员的访问级别只有public和private是否足够？
```cpp
//单个类的访问控制

#include"iostream"
using namespace std;

class TestParent
{
public:  //修饰的成员变量和方法，可以在类的内部和外部使用；
	TestParent();
	~TestParent();
	void print();
	int m_a; 
protected://修饰的成员变量和方法，只能在类的内部使用，在继承的子类中可以使用
	int m_b;
private: //修饰的成员变量和方法，只能在类的内部使用，不能在类的外部使用
	int m_c;

};

TestParent::TestParent()
{
	m_a = 0; 
	m_b = 0;
	m_c = 0;
}

TestParent::~TestParent()
{
	return;
}

void TestParent::print()
{
	cout << "m_a = " << m_a << endl;
	cout << "m_b = " << m_b << endl;
	cout << "m_c = " << m_c << endl;
}



//三种继承方式的子类


//公有继承
class TestChild1:public TestParent  //这里还可以是protected 或者private
{
public:
	void useVar1();
private:

};


void TestChild1::useVar1()
{
	m_a = 1;//ok
	m_b = 2;//ok
	//m_c = 3; //error
}



//保护继承
class TestChild2 :protected TestParent
{
public:
	void useVar2();
private:

};


void TestChild2::useVar2()
{
	m_a = 1;//ok
	m_b = 2;//ok
	//m_c = 3;//error
}



//private 继承
class TestChild3 :private TestParent
{
public:
	void useVar3();
private:

};


void TestChild3::useVar3()
{
	m_a = 1;//ok
	m_b = 2;//ok
	//m_c = 3;//error
}



int main()
{
	
	//public:
	TestChild1 t1;
	t1.m_a = 4;//ok
	//t1.m_b = 5;//error
	//t1.m_c = 6;//error

	//protected:
	TestChild2 t2;
	//t2.m_a = 11;
	//t2.m_b = 22;
	//t2.m_c = 33;

	//private:
	TestChild3 t3;
	//t3.m_a = 111;
	//t3.m_b = 222;
	//t3.m_c = 333;

	return 0;

}
```


### （二）不同的继承方式会改变继承成员的访问属性

**1.C++中的继承方式会影响子类的对外访问属性**

public继承：父类成员在子类中保持原有访问级别

private继承：父类成员在子类中变为private成员

protected继承：父类中public成员会变成protected

  父类中protected成员仍然为protected

  父类中private成员仍然为private

**2。private成员在子类中依然存在，但是却无法访问到。不论种方式继承基类，派生类都不能直接使用基类的私有成员**。

**3.C++中子类对外访问属性表**
|    |父类 |成员  |访问|级别|
|---|---|---|---|---|
|继	| |public|	proteced|	private|
|承 | public	|public	|proteced|	private|
|方 | proteced	|proteced|	proteced|	private|
|式 |private	|private|	private	|Private|


**4.继承中的访问控制**
![4]($resource/4.png)

### （三）“三看”原则**

C++中的继承方式（public、private、protected）会影响子类的对外访问属性    

判断某一句话，能否被访问

 1）看调用语句，这句话写在子类的内部、外部

 2）看子类如何从父类继承（public、private、protected）    

 3）看父类中的访问级别（public、private、protected）

### （四）派生类类成员访问级别设置的原则

思考：如何恰当的使用public，protected和private为成员声明访问级别？

1、需要被外界访问的成员直接设置为public

2、只能在当前类中访问的成员设置为private

3、只能在当前类和子类中访问的成员设置为protected，protected成员的访问权限介于public和private之间。

### （五）综合训练**

练习：

public继承不会改变父类对外访问属性；

private继承会改变父类对外访问属性为private；

protected继承会部分改变父类对外访问属性。

结论：一般情况下class B : public A

```cpp

//类的继承方式对子类对外访问属性影响

#include"iostream"
using namespace std;

class A
{
private:
	int a;
protected:
	int b;
public:
	int c;
	A();
	void set(int a, int b, int c);
};

A::A()
{
	a = 0;
	b = 0;
	c = 0;
}

void A::set(int a, int b, int c)
{
	this->a = a;
	this->b = b;
	this->c = c;
}


//public:

class B : public A
{
public:
	void print()
	{
		//cout<<"a = "<<a<<endl; //err
		cout << "b = " << b<<endl;
		cout << "c = " << c << endl;
	}
};


//protected:
class C : protected A
{
public:
	void print()
	{
		//cout<<"a = "<<a<<endl; //err
		cout << "b = " << b << endl;
		cout << "c = " << c << endl;
	}
};

//private
class D : private A
{
public:
	void print()
	{
		//cout<<"a = "<<a; //err
		cout << "b = " << b << endl;//保护的在子类中变成私有的，但是在子类中仍可使用
		cout << "c = " << c << endl;//保护的在子类中变成私有的，但是在子类中仍可使用
	}
};



int main()
{
	A aa;
	B bb;
	C cc;
	D dd;
	aa.c = 100; //ok
	bb.c = 100; //ok
	//cc.c = 100; //err 类的外部是什么含义
	//dd.c = 100; //err
	aa.set(1, 2, 3);
	bb.set(10, 20, 30);
	//cc.set(40, 50, 60); //ee

	//dd.set(70, 80, 90); //ee
	bb.print();
	cc.print();
	dd.print();
	system("pause");
	return 0;
}
```


## 三、继承中的构造和析构

### （一）类型兼容性原则

类型兼容规则是指在需要基类对象的任何地方，都可以使用公有派生类的对象来替代。通过公有继承，派生类得到了基类中除构造函数、析构函数之外的所有成员。这样，公有派生类实际就具备了基类的所有功能，凡是基类能解决的问题，公有派生类都可以解决。类型兼容规则中所指的替代包括以下情况：

* 子类对象可以当作父类对象使用
* 子类对象可以直接赋值给父类对象
* 子类对象可以直接初始化父类对象
* 父类指针可以直接指向子类对象
* 父类引用可以直接引用子类对象
在替代之后，派生类对象就可以作为基类的对象使用，但是只能使用从基类继承的成员。
类型兼容规则是多态性的重要基础之一。

总结：子类就是特殊的父类 (**base *p = &child;)**
这是源文档的示例程序：
```cpp
#include <cstdlib>
#include <iostream>
using namespace std;

/*
子类对象可以当作父类对象使用
  子类对象可以直接赋值给父类对象
  子类对象可以直接初始化父类对象
  父类指针可以直接指向子类对象
  父类引用可以直接引用子类对象
 */

//子类就是特殊的父类

class Parent03

{

protected:

 const char* name;

public:

 Parent03()

 {

 name = "Parent03";

 }

 void print()

 {

 cout<<"Name: "<<name<<endl;

 }

};

class Child03 : public Parent03

{

protected:

 int i;

public:

 Child03(int i)

 {

 this->name = "Child2";

 this->i = i;

 }

};

int main()

{

 Child03 child03(1000);

 //分别定义父类对象 父类指针 父类引用 child

 Parent03 parent = child03;

 Parent03* pp = &child03;

 Parent03& rp = child03;

 parent.print();

 pp->print();

 rp.print();

 system("pause");

 return 0;

}

```

这是亲自敲得示例程序：
```cpp
// 类型兼容性原则

/*
子类对象可以当作父类对象使用
  子类对象可以直接赋值给父类对象
  子类对象可以直接初始化父类对象
  父类指针可以直接指向子类对象
  父类引用可以直接引用子类对象
 */
#include "iostream"
using namespace std;

class Parent
{
public:
	Parent();
	Parent(const Parent &obj);
	void print_Parent();
private:

};

Parent::Parent()
{
	cout << "这是父类构造函数" << endl;
}
Parent::Parent(const Parent &obj)
{
	cout << "这是父类的拷贝构造函数" << endl;
}

void Parent::print_Parent()
{
	cout << "这是父类" << endl;
}




class Child :public Parent
{
public:
	void print_Child();
private:

};

void Child::print_Child()
{
	cout << "这是子类" << endl;
}


void howtoprint1(Parent *p)  //父类指针做函数参数
{
	p->print_Parent();  //调用的是父类的成员函数
}

void howtoprint2(Parent &p)  
{
	p.print_Parent();  
}



int main()
{
	Parent p1;
	p1.print_Parent();

	Child c1;
	c1.print_Child();
	c1.print_Parent();//子类对象可以调用父类的方法



	//赋值兼容性原则
	//1.1基类指针（引用）指向 子类对象

	Parent *p = NULL;
	p = &c1;
	p->print_Parent();


	//1.2 指针做函数参数
	howtoprint1(&p1);  //传递父类对象可以
	howtoprint1(&c1);  //传递子类对象也可以，因为赋值兼容性原则

	//1.3引用做函数参数
	howtoprint2(p1);
	howtoprint2(c1);



	//2.可以使用子类对象初始父类对象
	//因为子类就是一种特殊的父类
	Parent p2 = c1;

	system("pause");
	return 0;
}

```

### （二）继承中的对象模型

类在C++编译器的内部可以理解为结构体

子类是由父类成员叠加子类新成员得到的


![5]($resource/5.png)
![6]($resource/6.png)


**继承中构造和析构**

**问题：如何初始化父类成员？父类与子类的构造函数有什么关系**

  在子类对象构造时，需要**调用父类构造函数对其继承得来的成员进行初始化**

  在子类对象析构时，需要**调用父类析构函数对其继承得来的成员进行清理**

```cpp

//继承中构造和析构

#include "iostream"
using namespace std;

class Parent
{
public:
	Parent(int a, int b);
	~Parent();
	void print_Parent();
private:
	int a;
	int b;
};

Parent::Parent(int a, int b)
{
	this->a = a;
	this->b = b;
	cout << "这是父类构造函数" << endl;
}

Parent::~Parent()
{
	cout << "这是父类析构函数" << endl;
}


void Parent::print_Parent()
{
	cout << "这是父类的方法" << endl;
}




class Child :public Parent
{
public:
	Child(int a, int b, int c);    //声明的时候不需要加上后缀
	~Child();
	void print_Child();
private:
	int c;

};

Child::Child(int a, int b, int c) : Parent(a, b)   //但是实现的时候需要在后面加上继承的成员变量
{
	this->c = c;
	cout << "这是子类的构造函数" << endl;
}

Child::~Child()
{
	cout << "这是子类的析构函数" << endl;
}

void Child::print_Child()
{
	cout << "这是子类方法" << endl;
}


void playobj()
{
	cout << "下面是完整的生命周期" << endl;
	Child c2(3, 4, 5);
}

int main()
{
	Parent p1(1, 2);  //构造函数写了之后必须调用
	Child c1(1,2,3);

	//为了能够完整的展现生命周期
	playobj();


	system("pause");
	return 0;
}
```




### （三）继承中的构造析构调用原则

1、子类对象在创建时会首先调用父类的构造函数

2、父类构造函数执行结束后，执行子类的构造函数

3、当父类的构造函数有参数时，需要在子类的初始化列表中显示调用

4、析构函数调用的先后顺序与构造函数相反

### （四）继承与组合混搭情况下，构造和析构调用原则

  原则：  先构造父类，再构造成员变量、最后构造自己

  先析构自己，在析构成员变量、最后析构父类

 //先构造的对象，后释放

练习：demo05_extend_construct_destory.cpp

原来的示例程序：
```cpp

//子类对象如何初始化父类成员

//继承中的构造和析构

//继承和组合混搭情况下，构造函数、析构函数调用顺序研究

#include <iostream>

using namespace std;

class Object

{

public:

 Object(const char* s)

 {

 cout<<"Object()"<<" "<<s<<endl;

 }

 ~Object()

 {

 cout<<"~Object()"<<endl;

 }

};

class Parent : public Object

{

public:

 Parent(const char* s) : Object(s)

 {

 cout<<"Parent()"<<" "<<s<<endl;

 }

 ~Parent()

 {

 cout<<"~Parent()"<<endl;

 }

};

class Child : public Parent

{

protected:

 Object o1;

 Object o2;

public:

 Child() : o2("o2"), o1("o1"), Parent("Parameter from Child!")

 {

 cout<<"Child()"<<endl;

 }

 ~Child()

 {

 cout<<"~Child()"<<endl;

 }

};

void run05()

{

 Child child;

}

int main05(int argc, char *argv[])

{

 cout<<"demo05_extend_construct_destory.cpp"<<endl;

 run05();

 system("pause");

 return 0;

}

```

自己写的示例程序
~~调用child对象的时候有点问题，里面传递的参数应该是什么~~
```cpp
//继承与组合混搭情况下，构造和析构调用原则

#include "iostream"
using namespace std;

class Object
{
public:
	Object(int a, int b);
	~Object();

private:
	int a;
	int b;

};

Object::Object(int a, int b)
{
	this->a = a;
	this->b = b;
	cout << "这是祖宗类构造函数" << endl;
}

Object::~Object()
{
	cout << "这是祖宗类的析构函数" << endl;
}                                                                                                                                                                                                                                                                                                                                                                                                                                                               


class Parent :public Object
{
public:
	Parent(char *p);
	~Parent();
private:
	char *p;
};

Parent::Parent(char *p) : Object(1 , 2)
{
	this->p = p;
	cout << "这是父类构造函数  " << p<< endl;
}

Parent::~Parent()
{
	cout << "这是父类析构函数" << endl;
}




class Child :public Parent
{
public:
	Child(char *p);    
	~Child();
private:
	char *myp;
	Object obj1;//增加两个老祖宗类的成员变量
	Object obj2;

};

Child::Child(char *p) : Parent(p)  , obj1(3,4),obj2(5,6)
{
	this->myp = p;
	cout << "这是子类的构造函数  " << myp<<endl;
}

Child::~Child()
{
	cout << "这是子类的析构函数" << endl;
}

void objplay()
{
	Child c1();
}

int main()
{
	objplay();
	system("pause");
	return 0;
}

```


### （五）继承中的同名成员变量处理方法

1、当子类成员变量与父类成员变量同名时

2、子类依然从父类继承同名成员

3、在子类中通过作用域分辨符::进行同名成员区分（**在派生类中使用基类的同名成员，显式地使用类名限定符**）

4、同名成员存储在内存中的不同位置

![7]($resource/7.png)
![8]($resource/8.png)

```cpp
//继承中的同名成员变量和成员函数处理方法

#include "iostream"
using namespace std;

class Object
{
public:
	void GetObject();
	int a;
	int b;
	void Printla();
};


void Object::GetObject()
{
	cout << "b " << b << endl;
}

void Object::Printla()
{
	cout << "这是父类的成员函数" << endl;
}


class Parent :public Object
{
public:
	void GetParent();
	int b;
	int c;
	void Printla();
};


void Parent::GetParent()
{
	cout << "b " << b << endl;
	cout << "c " << c << endl;
}

void Parent::Printla()
{
	cout << "这是子类的成员函数" << endl;
}


int main()
{
	Parent p1;

	//成员变量的调用比较
	//默认是调用子类的成员变量；
	//以下的两个语句等价
	p1.b = 2;
	p1.Parent::b = 2;
	//调用父类中的成员变量
	p1.Object::b = 3;
	p1.GetObject();
	p1.GetParent();


	//成员函数的调用比较
	p1.Printla();
	p1.Parent::Printla();//以上这两句等价，默认调用的也是子类的成员函数
	p1.Object::Printla();  //调用父类的成员函数

	system("pause");
	return 0;
}


```

总结：同名成员变量和成员函数通过作用域分辨符进行区分





### （六）派生类中的static关键字

继承和static关键字在一起会产生什么现象哪？

理论知识

Ø 基类定义的静态成员，将被所有派生类共享

Ø 根据静态成员自身的访问特性和派生类的继承方式，在类层次体系中具有不同的访问性质  （遵守派生类的访问控制）

Ø  派生类中访问静态成员，用以下形式显式说明：

_类名_ :: _成员_

  或通过对象访问  _对象名_ . _成员_

![aa]($resource/aa.png)
![bb]($resource/bb.png)
![cc]($resource/cc.png)

总结：

1> static函数也遵守3个访问原则

2> static易犯错误（不但要初始化，更重要的显示的告诉编译器分配内存）

3> 构造函数默认为private
```cpp
//继承中的static关键字

#include "iostream"
using namespace std;

class Object
{
public:
	 static int a;
};

int Object::a = 2;  //static关键字必须初始化，更重的是C++编译器会根据初始化进行内存分配

class Parent :public Object
{
public:
	int b;
	int c;
};


int main()
{
	Parent p1;
	p1.a = 3;
	system("pause");
	return 0;
}
```



## 四、多继承

### （一）多继承的应用

**1.多继承概念**

Ø 一个类有多个直接基类的继承关系称为多继承

Ø   多继承声明语法

class _派生类名_ : _访问控制_ _基类名1_ , _访问控制_ _基类名2_ , … , _访问控制_ _基类名n_

 {

  _数据成员和成员函数声明_

 }；

Ø   类 C 可以根据访问控制同时继承类 A 和类 B 的成员，并添加

  自己的成员

![9]($resource/9.png)

**2.多继承的派生类构造和访问**

Ø 多个基类的派生类构造函数可以用初始式调用基类构造函数初始化数据成员

Ø 执行顺序与单继承构造函数情况类似。多个直接基类构造函数执行顺序取决于定义派生类时指定的各个继承基类的顺序。

Ø  一个派生类对象拥有多个直接或间接基类的成员。不同名成员访问不会出现二义性。如果不同的基类有同名成员，派生类对象访问时应该加以识别。

**3.多继承简单应用**


![10]($resource/10.png)
![11]($resource/11.png)
```cpp
//多继承

#include "iostream"
using namespace std;

class Object1
{
public:
	Object1(int a);
	 int a;
};

Object1::Object1(int a)
{
	this->a = a;
	cout << "这是Object1的构造函数" << endl;
}


class Object2
{
public:
	Object2(int b);
	 int b;
};

Object2::Object2(int b)
{
	this->b = b;
	cout << "这是Object2的构造函数" << endl;
}


class Parent :public Object1,Object2
{
public:
	Parent(int a, int b, int c);
	int c;
};

Parent::Parent(int a, int b, int c) : Object1(a), Object2(b)
//Parent::Parent(int c) : Object1(1), Object2(2)  //这种相当于将后两个的初始化写死了
{
	this->c = c;
}


int main()
{
	Parent p1(1,2,3);

	system("pause");
	return 0;
}
```



### （二）虚继承

  如果一个派生类从多个基类派生，而这些基类又有一个共同的基类，则在对该基类中声明的名字进行访问时，可能产生二义性

![12]($resource/12.png)


分析：

![13]($resource/13.png)

**总结：**

Ø   如果一个派生类从多个基类派生，而这些基类又有一个共同

  的基类，则在对该基类中声明的名字进行访问时，可能产生

  二义性

Ø   如果在多条继承路径上有一个公共的基类，那么在继承路径的某处

  汇合点，这个公共基类就会在派生类的对象中产生多个基类子对象

Ø   要使这个公共基类在派生类中只产生一个子对象，必须对这个基类

  声明为虚继承，使这个基类成为虚基类。

Ø   虚继承声明使用关键字 virtual


![14]($resource/14.png)
```cpp
//虚继承（继承的二义性）只能解决有共同老祖宗的场景

#include "iostream"
using namespace std;

class Object
{
public:
	int obj;
};


class Object1:virtual public Object  //使用virtual使其形成虚继承，从而可以在Parent类中直接使用Object类中的成员变量
{
public:
	 int obj1;
};

class Object2 : virtual public Object
{
public:
	 int obj2;
};

//class Parent :public Object1,Object2//这样继承Object2为private
class Parent :public Object1, public Object2
{
public:
	int c;
};


int main()
{
	Parent p1;
	p1.obj = 1;
	p1.obj1 = 2;
	p1.obj2 = 3;


	system("pause");
	return 0;
}
```

```cpp
//虚继承不是万能的，当继承两个相互独立的类的时候，virsual 关键字并不能解决成员变量相同的问题
#include "iostream"
using namespace std;

class Parent1
{
public:
	int a;
};



class Parent2
{
public:
	int a;
};

class Child :virtual public Parent1,virtual public Parent2
{
public:
	int b;
};

int main()
{
	Child c1;
	//c1.a = 1;    //即使加上virtual关键字之后仍然不能直接使用
	c1.Parent1::a = 4;  //只能这样使用
	c1.b = 2;
}
```

实验：注意增加virtual关键字后，构造函数调用的次数。
- 增加virtual关键字之后，构造函数只调用一次

使用virtual之后每个类的大小也会发生变化
```cpp
//使用virtual关键字，C++编译器会给变量偷偷的增加属性，使类的大小变大
#include "iostream"
using namespace std;

class Parent
{
public:
	int a;
};



class Child1:virtual public Parent
{
public:
	int b;
};

class Child2:public Parent
{
public:
	int b;
};

int main()
{
	cout << sizeof(Parent) << endl;
	cout << sizeof(Child1) << endl;
	cout << sizeof(Child2) << endl;
	system("pause");
	return 0;
}
```

## 五、继承总结

Ø 继承是面向对象程序设计实现软件重用的重要方法。程序员可以在已有基类的基础上定义新的派生类。

Ø  单继承的派生类只有一个基类。多继承的派生类有多个基类。

Ø  派生类对基类成员的访问由继承方式和成员性质决定。

Ø  创建派生类对象时，先调用基类构造函数初始化派生类中的基类成员。调用析构函数的次序和调用构造函数的次序相反。

Ø  C++提供虚继承机制，防止类继承关系中成员访问的二义性。

Ø  多继承提供了软件重用的强大功能，也增加了程序的复杂性。

![派生类的访问控制]($resource/%E6%B4%BE%E7%94%9F%E7%B1%BB%E7%9A%84%E8%AE%BF%E9%97%AE%E6%8E%A7%E5%88%B6.png)

