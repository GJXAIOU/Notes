# C_PP_章四 多态


## 一 、多态

### （一）问题引出

如果子类定义了与父类中原型相同的函数会发生什么？

**函数重写**
* 在子类中定义与父类中原型相同的函数
* 函数重写只发生在父类与子类之间
```cpp

//当类型兼容遇上函数重写（即基类和子类的成员函数相同）
#include "iostream"
using namespace std;

class Parent
{
public:
	void print()
	{
		cout << "Parent:print() do..." << endl;
	}
};

class Child : public Parent
{

public:
	void print()
	{
		cout << "Child:print() do..." << endl;
	}
};



void HowtoPrint1(Parent *base)
{
	base->print();
}

void HowtoPrint2(Parent &base )
{
	base.print();
}


int main()
{

	//使用指针
	Parent *base = NULL;
	Parent p1;
	Child c1;

	base = &p1;
	base->print();  //调用的是父类的打印函数

	base = &c1;
	base->print();  //其实调用的仍然是父类的打印函数

	//使用引用
	Parent &base1 = p1;
	base1.print();   //父类的打印函数

	Parent &base2 = c1;
	base2.print();   //父类的打印函数

	//使用指针和引用作为函数

	HowtoPrint1(&p1);
	HowtoPrint1(&c1);

	HowtoPrint2(p1);
	HowtoPrint2(c1);
	system("pause");

	return 0;

}

```

**结论：** 针对的是使用子类进行定义对象，上面程序是用父类进行定义程序
* 父类中被重写的函数依然会继承给子类
* 默认情况下子类中重写的函数==将隐藏父类中的函数==
* 通过作用域分辨符::可以访问到父类中被隐藏的函数

**解释：**
默认情况下：
C/C++是静态编译型语言
在编译时，编译器自动根据指针的类型判断指向的是一个什么样的对象
但是；
1、在编译此函数的时，编译器不可能知道指针 p 究竟指向了什么。
2、编译器没有理由报错。
3、于是，编译器认为最安全的做法是编译到父类的print函数，因为父类和子类肯定都有相同的print函数。



**因此提出面向对象新需求**
//如果我传一个父类对象，执行父类的print函数
//如果我传一个子类对象，执行子类的printf函数
* //现象产生的原因
  * //赋值兼容性原则遇上函数重写 出现的一个现象
  * //1 没有理由报错
  * //2 对被调用函数来讲，在编译器编译期间，我就确定了，这个函数的参数是p，是Parent类型的。。。
  * //3静态链编

 

### （二）面向对象新需求


编译器的做法不是我们期望的
  * 根据实际的对象类型来判断重写函数的调用
  * 如果**父类指针**指向的是父类对象则调用父类中定义的函数
  * 如果父类指针指向的是子类对象则调用子类中定义的重写函数

![多态1]($resource/%E5%A4%9A%E6%80%811.png)



### （三）解决方案

 * C++中通过virtual关键字对多态进行支持
 * 使用virtual==声明==的函数被重写后即可展现多态特性

最前面的程序的解决方法；根据传递的类的不能执行对应的语句
```cpp

//当类型兼容遇上函数重写（即基类和子类的成员函数相同）
#include "iostream"
using namespace std;

class Parent
{
public:
	virtual void print()
	{
		cout << "Parent:print() do..." << endl;
	}
};

class Child : public Parent
{ 
public:
	virtual void print()
	{
		cout << "Child:print() do..." << endl;
	}
};



void HowtoPrint1(Parent *base)
{
	base->print();
}
void HowtoPrint2(Parent &base )
{
	base.print();
}


int main()
{

	//使用指针
	Parent *base = NULL;
	Parent p1;
	Child c1;

	base = &p1;
	base->print();  //调用的是父类的打印函数

	base = &c1;
	base->print();  

	//使用引用
	Parent &base1 = p1;
	base1.print();   //父类的打印函数

	Parent &base2 = c1;
	base2.print();   

	//使用指针和引用作为函数

	HowtoPrint1(&p1);
	HowtoPrint1(&c1);

	HowtoPrint2(p1);
	HowtoPrint2(c1);
	system("pause");

	return 0;

}

```

### （四）多态实例

**案例场景：**

英雄战机HeroFighter , AdvHeroFighter 分别和敌机EnemyFighter 战斗.

```cpp

//战机和敌机比较大小，然后进阶版的战机和敌机进行比较大小

#include "iostream"
using namespace std;

class HeroFighter
{
public:
	virtual int ackPower()
	{
		return 10;
	}
};

class AdvHeroFighter : public HeroFighter
{
public:
	virtual int ackPower()
	{
		return HeroFighter::ackPower() * 2;//调用基类的函数
	}
};

class enemyFighter
{
public:
	int destoryPower()
	{
		return 15;
	}
};


void fighter(HeroFighter *fighter, enemyFighter *enemger)
{
	if (fighter->ackPower() > enemger->destoryPower())
	{
		cout << "主角牛X" << endl;
	} 
	else
	{
		cout << "主角 is a 垃圾" << endl;
	}	

}


int main()
{
	HeroFighter hero;
	AdvHeroFighter advhero;
	enemyFighter enemy;
	fighter(&hero, &enemy);
	fighter(&advhero, &enemy);

	system("pause");
	return 0;
}
```

### （五）多态工程意义

**面向对象3大概念**

- 封装
  突破了C语言函数的概念；
  
- 继承
  代码复用  ，即复用原来写好的代码；

- 多态
  多态保证可以使用未来的代码，前面相当于搭建框架，后面可以填充代码；多态是我们软件行业追寻的一个目标。。。



### （六）多态成立的条件

* 多态成立的三个条件
  * 1 要有继承
  * 2 要有函数重写（虚函数）
  * 3 要有父类指针（父类引用）指向子类对象

//多态是设计模式的基础，多态是框架的基础




### （七）多态的理论基础

**1.静态联编和动态联编**

1、联编是指一个程序模块、代码之间互相关联的过程。

2、静态联编（static binding），是程序的匹配、连接在编译阶段实现，也称为早期匹配。
例如不使用virsual关键字使用，重名的成员函数，其实C++编译器已经确定怎么执行了
  重载函数使用静态联编。

3、动态联编是指程序联编推迟到运行时进行，所以又称为晚期联编（迟绑定）。

switch 语句和 if 语句是动态联编的例子。

4、理论联系实际

* 1、C++与C相同，是静态编译型语言
* 2、在编译时，编译器自动根据指针的类型判断指向的是一个什么样的对象；所以编译器认为父类指针指向的是父类对象。
* 3、由于程序没有运行，所以不可能知道父类指针指向的具体是父类对象还是子类对象
* 从程序安全的角度，编译器假设父类指针只指向父类对象，因此编译的结果为调用父类的成员函数。这种特性就是静态联编。


---

### （八）虚析构函数
**使用目的：** 使用基类一次是释放掉所有子类分配的内存空间
示例代码：
```cpp

//注意virsual添加的位置
#include "iostream"
using namespace std;
#pragma warning(disable:4996)


class Father
{
public:
	Father();
	virtual ~Father();  //在这里添加virtual
	char *p;
private:

};

Father::Father()
{
	 p= new char[20];
	strcpy(p, "AAA");
	cout << "Father构造函数" << endl;
}

Father::~Father()  //注意这里不需要添加virtual
{
	delete[]p;
	cout << "Father析构函数" << endl;
}

class Child:public Father
{
public:
	Child();
	~Child();
	char *p;
private:

};

Child::Child()
{
	p = new char[20];
	strcpy(p, "AAA");
	cout << "Child构造函数" << endl;
}

Child::~Child()
{
	delete[]p;
	cout << "Child析构函数" << endl;
}

void howtodelete(Father *base)
{
	delete base;  //因为delete没有加多态的属性所以最终只调用了父类的析构函数
}

int main()
{
	Child *myc = new Child;
	howtodelete(myc);

	system("pause");
	return 0;
}
```



## 二、多态相关面试题

### **面试题1：请谈谈你对多态的理解**

- **多态的实现效果**
  多态：同样的调用语句有多种不同的表现形态；

- **多态实现的三个条件**
   有继承、有virtual重写、有父类指针（引用）指向子类对象。

- **多态的C++实现**
   virtual关键字，告诉编译器这个函数要支持多态；不是根据指针类型判断如何调用；而是==要根据指针所指向的实际对象类型来判断如何调用 == 
 
- **多态的理论基础**
  动态联编PK静态联编。根据实际的对象类型来判断重写函数的调用。

- **多态的重要意义**
  是设计模式的基础 是框架的基石。

- **实现多态的理论基础**
  函数指针做函数参数
  C函数指针是C++至高无上的荣耀。C函数指针一般有两种用法（正、反）。

- **多态原理探究**
  与面试官展开讨论

### **面试题2：谈谈C++编译器是如何实现多态**
   c++编译器多态实现原理

### **面试题3：谈谈你对重写，重载理解**

- 函数重载
  * 必须在同一个类中进行
  * **子类无法重载父类的函数**，父类同名函数将被名称覆盖
      因为认为是函数重载，所以调用的时候只在子类中查找，则无法调用父类的同名函数，
      本质上：默认情况下调用的是子类的函数，只有在使用域作用符情况下才调用父类中的同名函数
  * 重载是在编译期间根据参数类型和个数决定函数调用

- 函数重写
  * 必须发生于父类与子类之间
  * 并且父类与子类中的函数必须有完全相同的原型
  * 使用virtual声明之后能够产生多态(如果不使用virtual，那叫重定义)
  * 多态是在运行期间根据具体对象的类型决定函数调用

```cpp
#include <cstdlib>
#include"iostream"
using namespace std;

class Parent01
{
public:
	Parent01()
	{
		cout << "Parent01:printf()..do" << endl;
	}

public:
//这三个之间关系是：函数重载
	virtual void func()
	{
		cout << "Parent01:void func()" << endl;
	}

	virtual void func(int i)
	{
		cout << "Parent:void func(int i)" << endl;
	}

	virtual void func(int i, int j)
	{
		cout << "Parent:void func(int i, int j)" << endl;
	}
};

class Child01 : public Parent01
{
public:
	void func(int i, int j)
	{
		cout << "Child:void func(int i, int j)" << " " << i + j << endl;
	}

	void func(int i, int j, int k)
	{
		cout << "Child:void func(int i, int j, int k)" << " " << i + j + k << endl;
	}

};


void run01(Parent01* p)
{
	p->func(1, 2);
}


int main()
{

	Parent01 p;
	p.func();
	p.func(1);
	p.func(1, 2);
	
	Child01 c;
	//c.func(); //问题1

	c.Parent01::func();
	c.func(1, 2);
	run01(&p);
	run01(&c);
	system("pause");
	return 0;

}
```
//问题1：child对象继承父类对象的func，请问这句话能运行吗？why
不能：因为使用c.func(); 的话，因为名称覆盖，C++编译器不会去父类中寻找0个参数的func函数，只会在子类中找func函数。

//1子类里面的func无法重载父类里面的func
//2当父类和子类有相同的函数名、变量名出现，发生名称覆盖（子类的函数名，覆盖了父类的函数名。）
//3 应该是使用：c.Parent::func();




### **面试题4：是否可类的每个成员函数都声明为虚函数，为什么。**
c++编译器多态实现原理
可以，但是每次执行都需要虚函数表中寻址，降低执行效率

### **面试题5：构造函数中调用虚函数能实现多态吗？为什么？**
 c++编译器多态实现原理
不能
### **面试题6：虚函数表指针（VPTR）被编译器初始化的过程，你是如何理解的？**
 c++编译器多态实现原理

### **面试题7：父类的构造函数中调用虚函数，能发生多态吗？c++编译器多态实现原理**

### **面试题8：为什么要定义虚析构函数？**

- 在什么情况下应当声明虚函数

  * **构造函数不能是虚函数**。建立一个派生类对象时，必须从类层次的根开始，沿着继承路径逐个调用基类的构造函数
  * **析构函数可以是虚函数**。虚析构函数用于指引 delete 运算符正确析构动态对象

![多态2]($resource/%E5%A4%9A%E6%80%812.png)

![多态3]($resource/%E5%A4%9A%E6%80%813.png)



### **其他**

**父类指针和子类指针的步长**

* 1） 铁律1：指针也只一种数据类型，c++类对象的指针 使用`p++`或者`p--`，仍然可用。
* 2） 指针运算是按照指针所指的类型进行的。
  p++等价于p=p+1 //p = (unsigned int)basep + sizeof(*p) 步长。
* 3） 结论：父类`p++`与子类`p++`步长不同；不要混搭，**不要用父类指针++方式操作数组**。

```cpp
//父类指针和子类指针的步长

#include"iostream"
using namespace std;

class Father
{
public:
	Father(int a);
	virtual void print();
private:
	int a;
};

Father::Father(int a = 1)
{
	this->a = a;
}

 void Father::print()
{
	cout << "这是父类" << endl;
}


class Child :public Father
{
public:
	Child(int a, int b);
	virtual void print();
protected:
private:
	int b;
};


Child::Child(int a = 0, int b = 1) :Father(a)
{
	this->b = b;
}

void Howtoplay(Father *base)
{
	base->print();
}

 void Child::print()
{
	cout << "这是子类" << endl;
}


int main()
{			
	Child array[] = { Child(1),Child(2),Child(3) };

	Father *p = NULL;
	Child  *c = NULL;
	p = array;
	c = array;//父类和子类的指针都指向数组的首地址
	
	p->print();
	c->print();  
	//以上都是可以正常输出的

	p++;
	c++;
	p->print();
	c->print();
	//因为Child 中多了一个元素b，造成Child的定义的对象的长度比Father长，
	//所以同时+1的步长是不一样的，造成程序出错
	//如果子类和父类的长度一样则可以

	system("pause");
	return 0;
}
```
![多态4]($resource/%E5%A4%9A%E6%80%814.png)

 

----

## 三、多态原理探究

- 理论知识：
  * 当类中声明虚函数时，编译器会在类中生成一个虚函数表
  * 虚函数表是一个存储类成员函数指针的数据结构
  * 虚函数表是由编译器自动生成与维护的
  * virtual成员函数会被编译器放入虚函数表中
  * 当存在虚函数时，每个对象中都有一个指向虚函数表的指针（C++编译器给父类对象、子类对象提前布局vptr指针；当进行`howToPrint(Parent *base)`函数是，C++编译器不需要区分子类对象或者父类对象，只需要再base指针中，找vptr指针即可。）
  * vptr指针一般作为类对象的第一个成员
在定义对象的时候，C++编译器会在对象中自动的添加一个vptr指针，这个指针的目的是将虚函数做成虚函数表，而虚函数表中存储的是虚函数的入口地址，因此这样每个对象中都有虚函数表

程序示例：
```cpp
//vptr指针

#include"iostream"
using namespace std;
class Father
{
public:
	Father(int a);
	void print();
private:
	int a;
};

Father::Father(int a = 1)
{
	this->a = a;
}

void Father::print()
{
	cout << "这是父类" << endl;
}


class Child :public Father
{
public:
	Child(int a,int b);
	void print();
protected:
private:
	int b;
};


Child::Child(int a = 0 ,int b = 1):Father(a)
{
	this->b = b;
}

void Howtoplay(Father *base)
{
	base->print();
}

void Child::print()
{
	cout << "这是子类" << endl;
}


int main()
{
	Father p1(1); //提前布局
			//用类定义对象的时候，C++ 编译器会在对象中添加一个vptr指针
	Child c1(1, 2); //同样的子类中也有一个vptr指针

	Howtoplay(&p1);
	Howtoplay(&c1);
	system("pause");
	return 0;
}

```

### （一）多态的实现原理

**C++中多态的实现原理**

![多态111]($resource/%E5%A4%9A%E6%80%81111.png)
![多态222]($resource/%E5%A4%9A%E6%80%81222.png)
![多态333]($resource/%E5%A4%9A%E6%80%81333.png)
 

**说明1：虚函数执行效率低**
通过虚函数表指针VPTR调用重写函数是在程序运行时进行的，因此需要通过寻址操作才能确定真正应该调用的函数。而普通成员函数是在编译时就确定了调用的函数。在效率上，虚函数的效率要低很多。

**说明2：**
出于效率考虑，没有必要将所有成员函数都声明为虚函数

**说明3 ：** C++编译器，执行HowToPrint函数，不需要区分是子类对象还是父类对象
![多态555]($resource/%E5%A4%9A%E6%80%81555.png)



### （二）如何证明vptr指针的存在

证明vptr指针存在示例程序：（加virtual关键字和不加导致类的大小不同）
```cpp

//证明vptr指针的存在（通过验证类的大小）

#include "iostream"
using namespace std;

class A
{
public:
	void printf()
	{
		cout << "aaa" << endl;
	}

protected:

private:

	int a;

};

class B :public A
{

public:
	virtual void printf()
	{
		cout << "aaa" << endl;
	}

protected:

private:
	int a;
};

void main()
{
	//加上virtual关键字 c++编译器会增加一个指向虚函数表的指针  。。。
	cout << "sizeof(a):" << sizeof(A) << " sizeof(b): " << sizeof(B) << endl;
	cout << "hello..." << endl;
	system("pause");
	return;
}

```


### (三)构造函数中能调用虚函数，实现多态吗
不能
1）**对象中的VPTR指针什么时候被初始化？**

* 对象在创建的时,由编译器对VPTR指针进行初始化
* 只有当对象的构造完全结束后VPTR的指向才最终确定
* 父类对象的VPTR指向父类虚函数表
* 子类对象的VPTR指向子类虚函数表

vptr指针的分步初始化示例程序：
```cpp
//vptr的分步初始化

#include"iostream"
using namespace std;

class Father
{
public:
	Father(int a);
	 virtual void print();
private:
	int a;
};

Father::Father(int a = 1)
{
	this->a = a;
	print();   //在父类构造函数中调用一个虚函数
}

 void Father::print()
{
	cout << "这是父类" << endl;
}


class Child :public Father
{
public:
	Child(int a, int b);
	virtual void print();
protected:
private:
	int b;
};


Child::Child(int a = 0, int b = 1) :Father(a)
{
	this->b = b;
	print();   //在子类构造函数中调用一个虚函数
}

void Howtoplay(Father *base)
{
	base->print();
}

 void Child::print()
{
	cout << "这是子类" << endl;
}


int main()
{			
	Child c1(1, 2);  //定义子类的对象，如果只有父类的构造函数中调用了其中的虚函数，则最终调用的是父类的函数，
	//如果子类的构造函数中也调用了这个虚函数，则最终执行的是子类的虚函数，（这两个虚函数函数名一样）

	system("pause");
	return 0;
}

```

2）分析过程

  画图分析
  红色代表vptr指针的指向过程：首先指向父类的虚函数表，然后指向子类的虚函数表
 
![多态666]($resource/%E5%A4%9A%E6%80%81666.png)

