---
date:`2018-11-16-2018-11-18`
---

# C_PP_章一   C++对C的拓展
 
@toc

## 一、简单的C++程序

**第一个helloworld 程序：**
```cpp
/*
第一个程序：hello world
*/

#include"pch.h"
#include"iostream" //包含C++的头文件
using namespace std;   //使用命名空间，std：标准命名空间，在这个命名空间中定义了很多的标准定义

void main()
{
	cout << "hello C++ world" << endl;
	//cout :标准输出，输出到窗口
	//<<：左移操作符：在C++中的功能进行了加强，即操作符进行了重载
	//endl : 表示： \n
	system("pause");
}
```


### （一）求圆的周长和面积
==面向过程加工的是：一个一个函数==
==面向对象加工的是；一个一个类==
**使用结构化的方法实现：**
```cpp
/*
求圆的面积：结构化的方法
*/
#include"pch.h"
#include"iostream"
using namespace std;

void main()
{
	double r = 0 , girth = 0, area =0 ;
	const double PI = 3.1415;
	cout << "Please input radius:\n"<<endl; //操作符重载
	
	//cin：标准输入：表示键盘
	cin >> r;  //输入
	girth = 2 * PI * r;
	area = PI * r * r;
	cout << "radius = " << r << endl;
	cout << "girth = " << girth << endl;
	cout << "area = " << area << endl;
}
```
程序运行结果：
`Please input radius:`
`2`
`radius = 2`
`girth = 12.566`
`area = 12.566`


**使用面向对象方法实现：**
```cpp
/*
使用面向对象的方法实现
*/

#include"iostream" 
using namespace std;

/*在C语言中定义结构体；但是不能定义函数
struct Circle
{
	double s;
	int r;
}
*/


//先进行类的抽象

class Circle    
//相当于在C++中定义了一个自定义的数据类型：Circle，只有定义类这种数据类型的变量的时候才分配内存空间
{
public:
	double radius; //成员变量也叫属性
public: //类的访问控制
	void Set_Radius(double r) //成员函数
	{ 
		radius = r; 
	} 

	double Get_Radius() 
	{
		return  radius; 
	} //通过成员函数设置成员变量

	double Get_Girth()
	{ 
		return  2 * 3.14f * radius; 
	} //通过成员函数获取成员变量

	double Get_Area() 
	{ 
		return  3.14f * radius * radius;
	}

};

void main()
{
	//实例化：有这个数据类型定义变量
	Circle A, B; //用类定义对象

	//使用键盘输入的值
	cout << "please input the Radius:";
	double in_Radius;
	cin >> in_Radius;
	A.Set_Radius(in_Radius); //类的调用
	cout << "A.Radius = " << A.Get_Radius() << endl;
	cout << "A.Girth = " << A.Get_Girth() << endl;
	cout << "A.Area = " << A.Get_Area() << endl;

	//直接赋值
	B.Set_Radius(10.5);
	cout << "B.radius = " << B.Get_Radius() << endl;
	cout << "B.Girth=" << B.Get_Girth() << endl;
	cout << "B.Area = " << B.Get_Area() << endl;
}
```



### （二）==初学者容易犯错误类型==
```cpp
/*
常见的错误的程序
*/

#include<iostream>
using namespace std;//c++的命名空间
class circle
{
public:
	double r;
	double pi = 3.1415926;
	double area = pi * r*r;  //这个在初始化的时候就执行了一次，但是当是r为随机数；

};

int main()
{
	circle pi;
	cout << "请输入area" << endl;
	cin >> pi.r;

	cout << pi.area << endl;	//乱码  //这里pi.area只是相当于从变量所表示的内存空间中拿值，并没有执行后面的语句

	system("pause");
	return 0;
}

```


## 二、程序设计方法的发展历程
  
**1.面向过程的结构化程序设计方法**

- 设计思路：
  - 自顶向下、逐步求精。采用模块分解与功能抽象，自顶向下、分而治之。

- 程序结构：
   - 按功能划分为若干个基本模块，形成一个树状结构。
   - 各模块间的关系尽可能简单，功能上相对独立；每一模块内部均是由顺序、选择和循环三种基本结构组成。
    - 其模块化实现的具体方法是使用子程序。

- 优点：
  有效地将一个较复杂的程序系统设计任务分解成许多易于控制和处理的子任务，便于开发和维护。

- 缺点：可重用性差、数据安全性差、难以开发大型软件和图形界面的应用软件
  - 把数据和处理数据的过程分离为相互独立的实体。
  - 当数据结构改变时，所有相关的处理过程都要进行相应的修改。
  - 每一种相对于老问题的新方法都要带来额外的开销。
  - 图形用户界面的应用程序，很难用过程来描述和实现，开发和维护也都很困难。


**2.面向对象的方法**

- ==将数据及对数据的操作方法封装在一起==，作为一个相互依存、不可分离的整体——对象。
- ==对同类型对象抽象出其共性，形成类==。
- 类通过一个简单的外部接口，与外界发生关系。
- 对象与对象之间通过消息进行通信。

**2.1面向对象的基本概念**

**对象**

- 一般意义上的对象：
  - 是现实世界中一个实际存在的事物。
  -  可以是有形的（比如一辆汽车），也可以是无形的（比如一项计划）。
  - 是构成世界的一个独立单位，具有
    - 静态特征：可以用某种数据来描述
    - 动态特征：对象所表现的行为或具有的功能

- 面向对象方法中的对象：
  - 是系统中用来描述客观事物的一个实体，它是用来构成系统的一个基本单位。==对象由一组属性和一组行为构成。==

  -  属性：用来描述对象静态特征的数据项。
  -  行为：用来描述对象动态特征的操作序列。

**类**

- 分类——人类通常的思维方法

- 分类所依据的原则——抽象
  - 忽略事物的非本质特征，只注意那些与当前目标有关的本质特征，从而找出事物的共性，把具有共同性质的事物划分为一类，得出一个抽象的概念。
  - 例如，石头、树木、汽车、房屋等都是人们在长期的生产和生活实践中抽象出的概念。

- 面向对象方法中的"类"
  - 具有相同属性和服务的一组对象的集合
  - 为属于该类的全部对象提供了抽象的描述，包括属性和行为两个主要部分。
  - 类与对象的关系：  犹如模具与铸件之间的关系，一个属于某类的对象称为该类的一个实例。

**封装**

==也就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。==

- 把对象的属性和服务结合成一个独立的系统单元。
- 尽可能隐蔽对象的内部细节。对外形成一个边界（或者说一道屏障），只保留有限的对外接口使之与外部发生联系。

- ==继承==对于软件复用有着重要意义，是面向对象技术能够提高软件开发效率的重要原因之一。
  - 定义：特殊类的对象拥有其一般类的全部属性与服务，称作特殊类对一般类的继承。
  - 例如：将轮船作为一个一般类，客轮便是一个特殊类。

**多态**

==多态是指在一般类中定义的属性或行为，被特殊类继承之后，可以具有不同的数据类型或表现出不同的行为。== 这使得同一个属性或行为在一般类及其各个特殊类中具有不同的语义。

**面向对象的软件工程**

- 面向对象的软件工程是面向对象方法在软件工程领域的全面应用。它包括:

  - 面向对象的分析（OOA）

  - 面向对象的设计（OOD）

  - 面向对象的编程（OOP）

  - 面向对象的测试（OOT）

  - 面向对象的软件维护（OOSM）

**总结：**

- 面向过程程序设计：数据结构 + 算法
  主要解决科学计算问题，用户需求简单而固定

  - 特点：
    - 分析解决问题所需要的步骤
    - 利用函数实现各个步骤
    - 依次调用函数解决问题

  - 问题：
    - 软件可重用性差
    - 软件可维护性差
    - 构建的软件无法满足用户需求

- 面向对象程序设计：由现实世界建立软件模型

  将现实世界中的事物直接映射到程序中，可直接满足用户需求

  - 特点：
    - 直接分析用户需求中涉及的各个实体
    - 在代码中描述现实世界中的实体
    - 在代码中关联各个实体协同工作解决问题

  - 优势：
    - 构建的软件能够适应用户需求的不断变化
    - 直接利用面向过程方法的优势而避开其劣势


## 三、C语言和C++语言的关系


- C语言是在实践的过程中逐步完善起来的
  - 没有深思熟虑的设计过程
  - 使用时存在很多“灰色地带” 
  - 残留量过多低级语言的特征
  - 直接利用指针进行内存操作

- C语言的目标是高效
  - 最终程序执行效率的高效

当面向过程方法论暴露越来越多的缺陷的时候，业界开始考虑在工程项目中引入面向对象的设计方法，而第一个需要解决的问题就是：高效的面向对象语言，并且能够兼容已经存在的代码。

**C语言 +面向对象方法论 ===》Objective C /C++**

- C语言和C++并不是对立的竞争关系
  - C++是C语言的加强，是一种更好的C语言
  - C++是以C语言为基础的，并且完全兼容C语言的特性

**学习C++并不会影响原有的C语言知识，相反会根据加深对C的认知；**
**学习C++可以接触到更多的软件设计方法，并带来更多的机会。**

  - 1） C++是一种更强大的C，通过学习C++能够掌握更多的软件设计方法
  - 2） C++是Java/C#/D等现代开发语言的基础，学习C++后能够快速掌握这些语言
  - 3）C++是各大知名软件企业挑选人才的标准之一


## 四、C++对C的加强部分

### （一） namespace命名空间

- **1.C++命名空间基本常识：**

  所谓namespace，是指标识符的各种可见范围。C++标准程序库中的所有标识符都被定义于一个名为std的namespace中。

  - <iostream>和<iostream.h>格式不一样，前者没有后缀，实际上，在你的编译器include文件夹里面可以看到，二者是两个文件，打开文件就会发现，里面的代码是不一样的。后缀为.h的头文件c++标准已经明确提出不支持了，早些的实现将标准库功能定义在全局空间里，声明在带.h后缀的头文件里，c++标准为了和C区别开，也为了正确使用命名空间，规定头文件不使用后缀.h。 因此
    - 1）当使用<iostream.h>时，相当于在c中调用库函数，使用的是全局命名空间，也就是早期的c++实现；
    - 2）**当使用<iostream>的时候，该头文件没有定义全局命名空间，必须使用namespace std；这样才能正确使用cout**。

  - 由于namespace的概念，使用C++标准程序库的任何标识符时，可以有三种选择：
    - 直接指定标识符。例如std::ostream而不是ostream。完整语句如下： std::cout << std::hex << 3.4 << std::endl;
    - 使用using关键字。 using std::cout; using std::endl; using std::cin; 以上程序可以写成 cout << std::hex << 3.4 << endl;
    - 最方便的就是使用using namespace std; 例如： using namespace std;这样命名空间std内定义的所有标识符都有效（曝光）。就好像它们被声明为全局变量一样。那么以上语句可以如下写: cout <<hex << 3.4 << endl;因为标准库非常的庞大，所以程序员在选择的类的名称或函数名 时就很有可能和标准库中的某个名字相同。所以为了避免这种情况所造成的名字冲突，就把标准库中的一切都被放在名字空间std中。但这又会带来了一个新问 题。无数原有的C++代码都依赖于使用了多年的伪标准库中的功能，他们都是在全局空间下的。所以就有了<iostream.h> 和<iostream>等等这样的头文件，一个是为了兼容以前的C++代码，一个是为了支持新的标准。命名空间std封装的是标准程序库的名称，标准程序库为了和以前的头文件区别，一般不加".h"

- **2 C++命名空间定义及使用语法**

  - 在C++中，名称（name）可以是符号常量、变量、宏、函数、结构、枚举、类和对象等等。为了避免，在大规模程序的设计中，以及在程序员使用各种各样的C++库时，这些标识符的命名发生冲突，标准C++引入了关键字namespace（命名空间/名字空间/名称空间/名域），可以更好地控制标识符的作用域。
  - std是c++标准命名空间，c++标准程序库中的所有标识符都被定义在std中，比如标准库中的类iostream、vector等都定义在该命名空间中，使用时要加上using声明(using namespace std) 或using指示(如std::string、std::vector<int>).


  - C中的命名空间
     - 在C语言中只有一个全局作用域
     -  C语言中所有的全局标识符共享同一个作用域
     - 标识符之间可能发生冲突 

  - C++中提出了命名空间的概念
    - 命名空间将全局作用域分成不同的部分
    - **不同命名空间中的标识符可以同名而不会发生冲突**
    - **命名空间可以相互嵌套**
    - 全局作用域也叫默认命名空间

  - C++命名空间的定义：
     - namespace name { … }

  - C++命名空间的使用：
     - 使用整个命名空间：using namespace name;
    - **使用命名空间中的变量：using name::variable**;
    - 使用默认命名空间中的变量：::variable
    - 默认情况下可以直接使用默 认命名空间中的所有标识符

- **3、 C++命名空间编程实践**
使用标准命名空间：
```cpp
/*
标准命名空间：
*/

#include"iostream"
using namespace std;

//首先因为文件iostream没有引入标准的std;需要手工写
//如果不写using namespace std ,需要在显示的时候引入std

void main()
{
	//如果没有之前没有写using namespace std
	std::cout << "namespace test" << std::endl;
	system("pause");
}

```

自定义和使用命名空间：
```cpp

/*
自定义命名空间
*/
#include"iostream"
using namespace std;
namespace NameSpaceA  //定义命名空间
{
	int a = 0;  //定义变量
}

namespace NameSpaceB
{
	int a = 1;
	namespace NameSpaceC //嵌套命名空间
	{
		struct Teacher
		{
			char name[10];
			int age;
		};
	}
}


int main()
{
	using namespace NameSpaceA;
	using NameSpaceB::NameSpaceC::Teacher;//使用命名空间里面的结构体的方法
	printf("a = %d\n", a);
	printf("a = %d\n", NameSpaceB::a);
	NameSpaceB::NameSpaceC::Teacher t2;
		//可以将前面写出来，定义t2，这里可以不写的，因为前面已经使用了using了
	Teacher t1 = { "aaa", 3 };
	//C++中对struct进行了加强，可以直接使用结构体名进行定义变量

	printf("t1.name = %s\n", t1.name);

	printf("t1.age = %d\n", t1.age);

	system("pause");

	return 0;

}
```
程序运行结果：
`a = 0`
`a = 1`
`t1.name = aaa`
`t1.age = 3`


- **4.结论**

    - 1） 当使用<iostream>的时候，该[头文件](http://baike.baidu.com/view/668911.htm)没有定义全局命名空间，必须使用namespace std；这样才能正确使用cout。若不引入using namespace std ,需要这样做。std::cout。
    -  2） c++标准为了和C区别开，也为了正确使用命名空间，规定头文件不使用后缀.h。
    - 3） C++命名空间的定义： namespace name { … }
    - 4） using namespace NameSpaceA;//使用方式
    - 5） namespce定义可嵌套。

---

### （二）“实用性”增加（二-六点是语法增强）

```cpp
#include "iostream"
using namespace std;

//C语言中的变量都必须在作用域开始的位置定义！！

//C++中更强调语言的“实用性”，所有的变量都可以在需要使用时再定义。

int main()
{
 int i = 0;
 printf("ddd");
 int k;
 system("pause");

 return 0;

}
```

---

### （三）register关键字增强

- **含义：**
**register关键字**： 请求编译器让变量a直接放在寄存器里面，这样运行的速度快

- **C与C++中作用的区别**
==在c语言中 register修饰的变量不能取地址==，但是在c++里面做了更改

```c
#include"iostream"
using namespace std;

int main()
{
	register int a = 0;
	cout<<"&a = "<< &a<<endl;  //在C语言中一旦变量放在寄存器中就不能取地址，这样写会报错，但是在C++中可以实现返回内存地址
	system("pause");
	return 0;
}
```


  - register关键字的变化
    - register关键字请求“编译器”将局部变量存储于寄存器中
    - C++编译器有自己的优化方式，不使用register也可能做优化
    - C++中可以取得register变量的地址
    - ==C++编译器发现程序中需要取register变量的地址时，register对变量的声明变得无效==。
    - 早期C语言编译器不会对代码进行优化，因此register变量是一个很好的补充。
    

### （四）变量检测增强

- 在C语言中，重复定义多个同名的全局变量是合法的
- 在C++中，不允许定义多个同名的全局变量
- C语言中多个同名的全局变量最终会被链接到全局数据区的同一个地址空间上
```c
int g_var;
int g_var = 1;
```

- C++直接拒绝这种二义性的做法。


### （五） struct类型加强

- C语言的struct定义了一组变量的集合，C编译器并不认为这是一种新的类型
- ==struct关键字可以完成和class关键字一样的作用，但是还是有区别的==
- C++中的struct是一个新类型的定义声明
==C++中不需要使用typedef，可以直接使用结构体名字定义新的变量==
```cpp
/*
C++对struct进行了功能加强
*/

#include"iostream"
using namespace std;
struct Student
{
	char name[100];
	int age;
};

int main(void)
{
	Student s1 = { "wang", 1 };
	Student s2 = { "wang2", 2 };
	cout << "ans  = " << s2.name << endl;

	return 0;
}
```
程序运行结果：
`ans  = wang2`


### （六）C++中所有的变量和函数都必须有类型

- C++中所有的变量和函数都必须有类型
-  C语言中的默认类型在C++中是不合法的
- ==C++更加强调类型，任意的程序元素都必须显示指明类型==

- 总结：
  - 在C语言中
     - int f( )；表示返回值为int，接受任意参数的函数
     -  int f(void)；表示返回值为int的无参函数
  -  在C++中
     -  int f( );和int f(void)具有相同的意义，都表示返回值为int的无参函数


### （七）新增Bool类型关键字

- C++中的布尔类型
  - C++在C语言的基本类型系统之上增加了bool
   - ==C++中的bool可取的值只有true和false==
   - 理论上bool只占用一个字节，`sizeof(bool)`
   - 如果多个bool变量定义在一起，可能会各占一个bit，这取决于编译器的实现
  -  true代表真值，编译器内部用1来表示
  -  false代表非真值，编译器内部用0来表示
  -  bool类型只有true（非0）和false（0）两个值
   - ==C++编译器会在赋值时将非0值转换为true，0值转换为false==
```cpp

/*
C++中新增bool类型关键字
*/

#include"iostream"
using namespace std;

int main(int argc, char *argv[])
{
	int a;
	bool b = true;//告诉C++编译器分配一个字节空间
	printf("b = %d, sizeof(b) = %d\n", b, sizeof(b));
	b = 4;
	a = b;//切记b只有0或者1两个值
	printf("a = %d, b = %d\n", a, b);
	b = -4;
	a = b;//切记b只有0或者1两个值
	printf("a = %d, b = %d\n", a, b);
	a = 10;
	b = a;
	printf("a = %d, b = %d\n", a, b);
	b = 0;
	printf("b = %d\n", b);
	system("pause");

	return 0;

}

```
程序运行结果；
`b = 1, sizeof(b) = 1`
`a = 1, b = 1`
`a = 1, b = 1`
`a = 10, b = 1`
`b = 0`

---

###  (八) 三目运算符功能增强

**1、三目运算符在C和C++编译器的表现**

```cpp

/*
c++对于三目运算符的加强
*/

#include"iostream"
using namespace std;

int main()
{
	int a = 10;
	int b = 20;
	//返回一个最小数 并且给最小数赋值成30
	//在C语言中三目运算符是一个表达式 ，表达式不可能做左值，但是在C++可以作为左值进行赋值
	(a < b ? a : b) = 30;
	cout<<"a = "<< a<<endl;
	cout << "\n b =" << b << endl;
	system("pause");
	return 0;
}
```
程序运行结果：
`a = 30` 
`b =20`

**2、结论**
- 1）==C语言返回变量的值 ，C++语言是返回变量本身====；
  - C语言中的三目运算符返回的是变量值，不能作为左值使用，放在寄存器中；
  - C++中的三目运算符可直接返回变量本身，因此可以出现在程序的任何地方；

- 2）注意：三目运算符可能返回的值中如果有一个是常量值，则不能作为左值使用
  - (a < b ? 1 : b )= 30;

- 3）C语言如何支持类似C++的特性呢？
  - ==>当左值的条件：要有内存空间；C++编译器帮助程序员取了一个地址而已
  C语言中可以这样来使用：`*(a <b ? &a:&b)=30`


## 五、C/C++中的const

### （一）const基础知识（用法、含义、好处）

**1.const的用法：**

- const用法：
```cpp
/*
const增强：
*/
#include"iostream"
using namespace std;

int main()
{
	//C++中常量对象必须初始化

	const int a =0;
	int const b =1;//第一个第二个意思一样 代表一个常整形数

	const int *c = NULL;//第三个 c是一个指向常整形数的指针(所指向的内存数据不能被修改，但是本身可以修改)

	int * const d = NULL;//第四个 d 常指针（指针变量不能被修改，但是它所指向内存空间可以被修改）

	const int * const e = NULL;//第五个 e一个指向常整形的常指针（指针和它所指向的内存空间，均不能被修改）

	return 0;

}
```

初级理解：const是定义常量==》const意味着只读

- const好处：
  - 合理的利用const，
    - 1指针做函数参数，可以有效的提高代码可读性，减少bug；
    - 2清楚的分清参数的输入和输出特性
    - int setTeacher_err( const Teacher *p)
    - Const修改形参的时候，在利用形参不能修改指针所向的内存空间


**2. C中const是“冒牌货”**
C语言中使用const定义变量，可以使用指针进行间接修改
```c
int main()
{
 const int a = 10;
 int *p = (int*)&a;
 printf("a===>%d\n", a);
 *p = 11;
 printf("a===>%d\n", a);
 return 0;
}
```
程序运行结果；
C语言中：`11`
C++语言中：`10`

- 解释：
  - C++编译器对const常量的处理
  - 当碰见常量声明时，在符号表中放入常量
  - 编译过程中若发现使用常量则直接以符号表中的值替换
  - ==编译过程中若发现对const使用了extern或者&操作符，则给对应的常量分配存储空间（兼容C）==

  - ？联想： int &a = 1(err) & const int &a = 10(ok)?


- C++中const符号表原理图
![cout符号原理图]($resource/cout%E7%AC%A6%E5%8F%B7%E5%8E%9F%E7%90%86%E5%9B%BE%202.png)



- 注意：
  - C++编译器虽然可能为const常量分配空间，但不会使用其存储空间中的值。

- 结论：
  - C语言中的const变量
    - C语言中const变量是只读变量，有自己的存储空间

  - C++中的const常量
    - 可能分配存储空间,也可能不分配存储空间 
    - 当const常量为全局，并且需要在其它文件中使用
    - 当使用&操作符取const常量的地址


**3 、const和#define相同之处**
```cpp

//#define N 10

int main()
{
 const int a = 1;

 const int b = 2;

 int array[a + b ] = {0};

 int i = 0;

 for(i=0; i<(a+b); i++)
 {
   printf("array[%d] = %d\n", i, array[i]);
 }

 getchar();

 return 0;

}
```

C++中的const修饰的，是一个真正的常量，而不是C中变量（只读）。在const修饰的常量编译期间，就已经确定下来了。

**4 const和#define的区别**
- 对比加深
  - C++中的const常量类似于宏定义
    - const int c = 5; ≈ #define c 5
    
  - C++中的const常量与宏定义不同
    - const常量是由编译器处理的，提供类型检查和作用域检查
    - 宏定义由预处理器处理，单纯的文本替换
   
//在func1定义a，在func2中能使用吗？
//在func1中定义的b，在func2中能使用吗？

练习
```cpp
void fun1()
{
 #define a 10  #在fun1()函数中define定义的变量a，可以在fun2()中使用

 const int b = 20; #在fun1()函数中const定义的变量b，不可以在fun2()中使用

//如果想让fun1()中使用define定义的变量a不可以在fun2()中使用，需要卸载宏，
 //#undef a   //只卸载a的宏定义
 //# undef   //卸载fun1()中所有的宏定义

}

void fun2()
{
 printf("a = %d\n", a);

 //printf("b = %d\n", b);
}

int main()
{
 fun1();

 fun2();

 return 0;
}

```


**5、结论**

- C语言中的const变量
  - C语言中const变量是只读变量，有自己的存储空间

- C++中的const常量
  - 可能分配存储空间,也可能不分配存储空间 
  - 当const常量为全局，并且需要在其它文件中使用，会分配存储空间
  - 当使用&操作符，取const常量的地址时，会分配存储空间
  - 当const int &a = 10; const修饰引用时，也会分配存储空间



## 六、引用专题的讲座

### （一）引用（普通应用）

- 变量名回顾
  - 变量名实质上是一段连续存储空间的别名，是一个标号(门牌号)
  - 程序中通过变量来申请并命名内存空间  
  - 通过变量的名字可以使用存储空间

问题1：对一段连续的内存空间只能取一个别名吗？

**1、引用概念**

- a) 在C++中新增加了引用的概念
- b) ==引用可以看作一个已定义变量的别名==
- c) 引用的语法：`Type& name = var`;
- d） 引用做函数参数那？（引用作为函数参数声明时不进行初始化）

```cpp
void main()
{
 int a = 10; //c编译器分配4个字节内存。。。a内存空间的别名
 int &b = a; //b就是a的别名。。。

 a =11; //直接赋值
 {
   int *p = &a;
   *p = 12;
   printf("a %d \n",a);
 }

 b = 14;
 printf("a:%d b:%d", a, b);

 system("pause");

}
```


**2、引用是C++的概念**

属于C++编译器对C的扩展
问题：C中可以编译通过吗？
```cpp
int main()
{
 int a = 0;
 int &b = a; //int * const b = &a
 b = 11; //*b = 11;

 return 0;
}

```
结论：请不要用C的语法考虑 b=11


**3、引用做函数参数**
- ==普通引用在声明时必须用其它的变量进行初始化==，`int &b ;`是错误的，引用定义的时候必须初始化；

- ==引用作为函数参数声明时不进行初始化==

```cpp
//05复杂数据类型 的引用

struct Teacher
{
	char name[64];

	int age;
};

//原本代码
void printfT(Teacher *pT)
{
	cout << pT->age << endl;
}


//使用引用
//pT是t1的别名 ,相当于修改了t1

void printfT2(Teacher &pT)
{
	cout<<pT.age<<endl;

	pT.age = 33;
}

//pT和t1的是两个不同的变量

void printfT3(Teacher pT)

{

	cout << pT.age << endl;

	pT.age = 45; //只会修改pT变量 ,不会修改t1变量

}

void main()

{

	Teacher t1;

	t1.age = 35;

	printfT(&t1);//调用第一个函数

	printfT2(t1); //pT是t1的别名，在函数中对PT修改相当于修改了t1

	printf("t1.age:%d \n", t1.age); //33

	printfT3(t1);// pT是形参 ,t1 copy一份数据 给pT //---> pT = t1

	printf("t1.age:%d \n", t1.age); //35

	cout << "hello..." << endl;

	system("pause");

	return;

}
```

**4、引用的意义**

- 1）引用作为其它变量的别名而存在，因此在一些场合可以代替指针

- 2）引用相对于指针来说具有更好的可读性和实用性

![引用的意义]($resource/%E5%BC%95%E7%94%A8%E7%9A%84%E6%84%8F%E4%B9%89.png)



**5、引用本质思考**

- 思考1：C++编译器背后做了什么工作？
```cpp

int main()

{

 int a = 10;

 int &b = a;

 //b是a的别名，请问c++编译器后面做了什么工作？

 b = 11;

 cout<<"b--->"<<a<<endl;

 printf("a:%d\n", a);

 printf("b:%d\n", b);

 printf("&a:%d\n", &a);

 printf("&b:%d\n", &b); //两个地址是一样的,a,b都是同一块内存空间的门牌号

 system("pause");

 return 0;

}
```

单独定义的引用时，必须初始化；说明很像一个常量


- 思考2：普通引用有自己的空间吗？
```cpp
struct Teacer 
{
	int &a;
	int &b;//结构体中可以这样定义，所占内存空间为4个字节，和指针所占空间一致
};

int main()
{

	printf("sizeof(Teacher) %d\n", sizeof(Teacer));

	system("pause");

	return 0;
}

```

引用是一个有地址，引用是常量；

char *const p

 

**6、引用的本质**

- 1）引用在C++中的内部实现是 一个常指针

    Type& name  <<-->> Type* const name

- 2）C++编译器在编译过程中使用常指针作为引用的内部实现，因此引用所占用的空间大小与指针相同。

- 3）从使用的角度，引用会让人误会其只是一个别名，没有自己的存储空间。这是C++为了实用性而做出的细节隐藏
```cpp
void func(int &a)                                             void func(int *const a)                                                                              
{                                                               { 
  a = 5;                                                          *a = 5;
}                                                               }
左边的程序在C++中执行的时候，C++编译器相当于将其翻译成右边的代码
```


- 4） 请仔细对比间接赋值成立的三个条件

  - 1定义两个变量 （一个实参一个形参）

  - 2建立关联 实参取地址传给形参

  - 3*p形参去间接的修改实参的值


**7、引用结论**

- 1）引用在实现上，只不过是把：间接赋值成立的三个条件的后两步和二为一

  当实参传给形参引用的时候，只不过是c++编译器帮我们程序员手工取了一个实参地址，传给了形参引用（常量指针）

- 2）当我们使用引用语法的时，我们不去关心编译器引用是怎么做的

  当我们分析奇怪的语法现象的时，我们才去考虑c++编译器是怎么做的


**8、函数返回值是引用(引用当左值)**

- C++引用使用时的难点：

  - 当函数返回值为引用时

    - 若返回栈变量（因为内存空间已经被释放了）

    - 不能成为其它引用的初始值

    - 不能作为左值使用

 - 若返回静态变量或全局变量

    - 可以成为其他引用的初始值

    - 即可作为右值使用，也可作为左值使用

 - C++链式编程中，经常用到引用，运算符重载专题

**返回值是基础类型，当引用**

```cpp
*

函数返回值为一个引用

*/
int getAA1()
{
	int a;
	a = 10;
	return a;
}

//基础类型a返回的时候，也会有一个副本

int& getAA2()
{
	int a;
	a = 10;
	return a;
}


int* getAA3()
{
	int a;
	a = 10;
	return &a;
}


void main()
{
	int a1 = 0;
	int a2 = 0;
	a1 = getAA1();
	a2 = getAA2();//当使用一个变量接受的时候，系统将引用的值相当于创建一个副本然后给变量a2，所以a2值为10；
	int &a3 = getAA2();//当使用一个引用来接受的时候，系统将a的地址返回，这时候相当于a3指向那块内存，但是调用完成之后，内存已经释放了，所以会出现乱码；
}
```
程序运行示意图；

![引用]($resource/%E5%BC%95%E7%94%A8.png)


**返回值是static变量，当引用**

```cpp


/*

返回值是static变量，当引用
*/

//static修饰变量的时候，变量是一个状态变量

#include"iostream"
using namespace std;

//返回的是a的值
int j()
{
	static int a = 10;
	a++;
	cout << "a : " << a << endl;
	return a;
}


//返回的是a这个变量本身
int& j1()
{
	static int a = 10;
	a++;
	cout << "a : " << a << endl;

	return a;
}



int *j2()
{	
	static int a = 10;
	a++;
	printf("a:%d \n", a);

	return &a;
}





void main()
{

	// j()的运算结果是一个数值，没有内存地址，不能当左值。。。。。

	//11 = 100;

	//*(a>b?&a:&b) = 111;

	//当被调用的函数当左值的时候，必须返回一个引用。。。。。

	j1() = 100; //编译器帮我们打造了环境

	j1();

	*(j2()) = 200; //相当于我们程序员手工的打造 做左值的条件

	j2();

	system("pause");

}

```


**返回值是形参，当引用**

```cpp
int g1(int *p)

{

 *p = 100;

 return *p;

}

int& g2(int *p) //

{

 *p = 100;

 return *p;

}

//当我们使用引用语法的时候 ，我们不去关心编译器引用是怎么做的

//当我们分析乱码这种现象的时候，我们才去考虑c++编译器是怎么做的。。。。

void main23()

{

 int a1 = 10;

 a1 = g2(&a1);

 int &a2 = g2(&a1); //用引用去接受函数的返回值，是不是乱码，关键是看返回的内存空间是不是被编译器回收了。。。。

 printf("a1:%d \n", a1);

 printf("a2:%d \n", a2);

 system("pause");

}
```


**返回值非基础类型**
```cpp
struct Teachar

{

 char name[64];

 int age;

};
```
//如果返回引用不是基础类型，是一个类，那么情况非常赋值。。涉及到copy构造函数和=操作重载，抛砖。。。。

```cpp

struct Teachar

{

 char name[64];

 int age;

};
```
//如果返回引用不是基础类型，是一个类，那么情况非常赋值。。涉及到copy构造函数和=操作重载，抛砖。。。。
```cpp
struct Teachar & OpTeacher(struct Teachar &t1)
{

}
```


**9、指针引用**
```cpp
/*
指针引用

*/
#include "iostream"

using namespace std;

struct Teacher
{
	char name[64];

	int age;
};



//getTe对应的是从语言中的使用方法
int getTe(Teacher **myp)
{
	Teacher *p = (Teacher *)malloc(sizeof(Teacher));
	 
	if (p == NULL)

	{

		return -1;

	}

	memset(p, 0, sizeof(Teacher));

	p->age = 33;

	*myp = p; //

	return 0;

}


//指针的引用而已

int getTe2(Teacher* &myp)
{
	//给myp赋值相当于给main函数中的pt1进行赋值
	myp = (Teacher *)malloc(sizeof(Teacher));

	myp->age = 34;

	return 0;

}


void Freeteacher(Teacher *pT1)
{
	if (pT1 == NULL)
	{
		return;
	}
	free(pT1);

}

void main()
{
	Teacher *p = NULL;
	Teacher *pT1 = NULL;

	//getTe(&p);

	getTe2(p);
	Freeteacher(pT1);

	printf("age:%d \n", p->age);

	system("pause");

}
```



### （二）常引用

下面开始进入const引用难点

**1、使用变量初始化const引用**

在C++中可以声明const引用
`const Type& name = var；`const引用让变量拥有只读属性
常引用的初始化有两种情况：
```cpp

/*

常引用的初始化；两种方式
*/
#include"iostream"
using namespace std;

int main()
{
	//使用变量进行初始化常引用
	int x = 20;
	const int &y = x;  //使用x变量去初始化常引用


	//使用字面量初始化常量引用

	const int a = 40;  //定义一个字面量，首先C++将a是放在符号表中的，所以是没有内存地址的
	//int &m = 41;//这是一个普通引用，引用一个字面量，但是引用本质上就是给内存取别名，而这里的a是没有内存的，所以这句话会报错
	const int &b = 42;//这时候C++编译器会自动的分配内存空间
	const int &c = a;//这也是正确的

	system("pause");
	return 0;

}

```
案例1：
```cpp
/*
常引用：案例1
*/

#include"iostream"
using namespace std;

int main()
{

	int a = 10;

	const int &b = a;//常引用  作用：让变量拥有只读属性，不能使用b来修改a

	//int *p = (int *)&b;

	b = 11; //err

	//*p = 11; //只能用指针来改变了

	cout << "b--->" << a << endl;

	printf("a:%d\n", a);

	printf("b:%d\n", b);

	printf("&a:%d\n", &a);

	printf("&b:%d\n", &b);

	system("pause");

	return 0;

}
```

案例2：
```cpp
void main()

{

 int a = 10;

 const int &b = a; //const引用  使用变量a初始化

 a = 11;

 //b = 12; //通过引用修改a,对不起修改不了

 system("pause");

}

struct Teacher1

{

 char name[64];

 int age;

};

void printTe2(const Teacher1 *const pt)

{

}

//const引用让变量(所指内存空间)拥有只读属性

void printTe(const Teacher1 &t)

{

 //t.age = 11;

}

void main42()

{

 Teacher1 t1;

 t1.age = 33;

 printTe(t1);

 system("pause");

}

```


**2使用字面量常量初始化const引用**
思考：
1、用变量对const引用初始化，const引用分配内存空间了吗？
2、用常量对const引用初始化，const引用分配内存空间了吗？
```cpp
void main()
{

 const int b = 10;

 printf("b:%d", &b);

 //int &a1 = 19; 如果不加const编译失败

 const int &a = 19;

 printf("&a:%d \n", &a);

 system("pause");

}
```


**3综合案例**
```cpp
void main()

{

 //普通引用

 int a = 10;

 int &b = a;

 //常量引用  ：让变量引用只读属性

 const int &c = a;

 //常量引用初始化  分为两种

 //1 用变量 初始化 常量引用

 {

 int x = 20;

 const int& y = x;

 printf("y:%d \n", y);

 }

 //2 用常量 初始化 常量引用

 {

 //int &m = 10; //引用是内存空间的别名 字面量10没有内存空间 没有方法做引用

 const int &m = 10;

 }

 cout<<"hello..."<<endl;

 system("pause");

 return ;

}
```


###   (三) const引用结论

- 1）Const & int e 相当于 const int * const e

- 2）普通引用 相当于 int *const e1

- 3）==当使用常量（字面量）对const引用进行初始化时，C++编译器会为常量值分配空间，并将引用名作为这段空间的别名==

- 4）==使用字面量对const引用初始化后，将生成一个只读变量==

### （四）const修饰类

  后续课程介绍
  
### （五）综合练习

```cpp
int& j()
{
 static int a = 0;
 return a;
}

int& g()
{
 int a = 0;
 return a;
}

int main()
{
 int a = g();
 int& b = g();

 j() = 10;

 printf("a = %d\n", a);
 printf("b = %d\n", b);
 printf("f() = %d\n", f());
 system("pause");

 return 0;

}
```
 
## 七、C++对C的函数拓展


### （一）inline内联函数

C++中的const常量可以替代宏常数定义，如：const int A = 3;   #define A 3

C++中是否有解决方案替代宏代码片段呢？（替代宏代码片段就可以避免宏的副作用！）

- C++中推荐使用内联函数替代宏代码片段 
- C++中使用inline关键字声明内联函数

内联函数声明时inline关键字必须和函数定义结合在一起，否则编译器会直接忽略内联请求。

- 宏替换和函数调用区别
```cpp

/*

带参数的宏和函数调用的区别
*/

#include "iostream"
using namespace std;

//带参数的宏
#define MYFUNC(a, b) ((a) < (b) ? (a) : (b)) 

inline int myfunc(int a, int b)
{
	return a < b ? a : b;
}

int main()
{

	int a = 1;
	int b = 3;
	int c = myfunc(++a, b); 
	/*
	这里相当于将上面的函数这个拿下来，当然先算++a,和b的值
	return a < b ? a : b;
	
	*/
	cout << "a = " << a << " b  =" << b << " c =" << c << endl;

	int x = 1;
	int y = 3;
	int z = MYFUNC(++x, y);
	/*
	这里就是最直接的函数替换
	MYFUNC(++x, y) 相当于：(++x) < (y) ? (++x) : (y)
	*/
	cout << "x = " << x << " y  =" << y << " z =" << z << endl;

	system("pause");

	return 0;
}

```
程序运行结果：
`a = 2 b  =3 c =2`
`x = 3 y  =3 z =3`

- 说明1：
  - 必须inline int myfunc(int a, int b)和函数体的实现，写在一块

- 说明2:
  - C++编译器可以将一个函数进行内联编译
  - 被C++编译器内联编译的函数叫做内联函数
  - 内联函数在最终生成的代码中是没有定义的
  - C++编译器直接将函数体插入在函数调用的地方
  - 内联函数没有普通函数调用时的额外开销(压栈，跳转，返回)
 
- 说明3：C++编译器不一定准许函数的内联请求！

- 说明4:
  - 内联函数是一种特殊的函数，具有普通函数的特征（参数检查，返回类型等）
  - 内联函数是对编译器的一种请求，因此编译器可能拒绝这种请求
  - ==内联函数由 编译器处理，直接将编译后的函数体插入调用的地方==
  - ==宏代码片段 由预处理器处理， 进行简单的文本替换，没有任何编译过程==

- 说明5：
  - 现代C++编译器能够进行编译优化，因此一些函数即使没有inline声明，也可能被编译器内联编译
  - 另外，一些现代C++编译器提供了扩展语法，能够对函数进行强制内联
  如：g++中的__attribute__((always_inline))属性


- 说明6：
  - C++中内联编译的限制：
  - 不能存在任何形式的循环语句   
  - 不能存在过多的条件判断语句
  - 函数体不能过于庞大
  - 不能对函数进行取址操作
  - 函数内联声明必须在调用语句之前


- 编译器对于内联函数的限制并不是绝对的，==内联函数相对于普通函数的优势只是省去了函数调用时压栈，跳转和返回的开销。==
  因此，当函数体的执行开销远大于压栈，跳转和返回所用的开销时，那么内联将无意义。

- 结论：
  -  1）==内联函数在编译时直接将函数体插入函数调用的地方==
  - 2）inline只是一种请求，==编译器不一定允许这种请求==
  - 3）内联函数省去了普通函数调用时压栈，跳转和返回的开销




###   (二)默认参数

1.**概念：**
-  C++中可以在函数声明时为参数提供一个默认值
- 当函数调用时没有指定这个参数的值，编译器会自动用默认值代替
```cpp
/*
默认参数
*/

#include "iostream"
using namespace std;
void myPrint(int x = 3)
{
	cout << " x = " << x << endl;
}


int main()
{
	myPrint(4);
	myPrint();

	system("pause");
	return 0;
}
```
程序运行结果：
`x = 4`
 `x = 3`


- 函数默认参数的规则
  - 只有参数列表后面部分的参数才可以提供默认参数值
 -  ==一旦在一个函数调用中开始使用默认参数值，那么这个参数后的所有参数都必须使用默认参数值==
```cpp
//默认参数

#include"iostream"
using namespace std;

void printAB(int x = 3)
{
	cout << "x = " << x << endl;
}

//在默认参数规则 ，如果默认参数出现，那么右边的都必须有默认参数

void printABC(int a, int b, int x = 3, int y = 4, int z = 5)
{
	cout << "x = " << x << endl;
}

int main()
{
	printAB(2);
	printAB();
	system("pause");
	return 0;

}
```
程序运行结果：
`x = 2`
`x = 3`
 

---


### （三）函数占位参数

- 函数占位参数 
  - 占位参数只有参数类型声明，而没有参数名声明
  - 一般情况下，在函数体内部无法使用占位参数
```cpp
/*
占位参数
*/

#include "iostream"
using namespace std;

int func(int a, int b, int)  //最后一个为占位参数 
{
	return a + b;
}

int main()
{
	//func(1, 2); //这样是错误的，因为即使是占位参数也要写全

	printf("func(1, 2, 3) = %d\n", func(1, 2, 3));

	return 0;
}

```
程序运行结果：`func(1, 2, 3) = 3`



### (四)默认参数和占位参数

可以将占位参数与默认参数结合起来使用

 -  意义
    - 为以后程序的扩展留下线索 
    - 兼容C语言程序中可能出现的不规范写法

//C++可以声明占位符参数，占位符参数一般用于程序扩展和对C代码的兼容

```cpp
/*
占位参数和默认参数
*/

#include "iostream"
using namespace std;

int func2(int a, int b, int = 0)//第三个为将占位参数赋默认值
{
	return a + b;
}

void main()
{
	//如果默认参数和占位参数在一起，都能调用起来
	int c = func2(1, 2);//ok
	int d = func2(1, 2, 3);//ok
	
	cout << "c = " << c << " d = " << d << endl;
	system("pause");
}
```
程序运行结果：
`c = 3 d = 3`

**结论：**//如果默认参数和占位参数在一起，都能调用起来



---

### (五) 函数重载（Overroad）

**1.函数重载概念**
函数重载(Function Overload)：
  用同一个函数名定义不同的函数，当函数名和不同的参数搭配时函数的含义不同

**2 函数重载的判断标准**
函数重载至少满足下面的一个条件：
  - 参数个数不同
  - 参数类型不同
  - 参数顺序不同

**3 函数返回值不是函数重载的判断标准**
//两个难点：重载函数和默认函数参数混搭； 重载函数和函数指针

**函数重载调用方式：**
```cpp
/*
函数重载
*/
#include"iostream"
using namespace std;

int func(int x)
{
	return x;
}

int func(int a, int b)
{
	return a + b;
}

int func(const char* s)
{
	return strlen(s);
}



int main()
{
	int c = 0;

	c = func(1);
	cout << "c = " <<c << endl;


	c = func(1, 2);
	cout << "\nc = " << c << endl;


	c = func("12345");
	cout << "\nc = " << c << endl;

	return 0;

}

```
程序运行结果；
`c = 1`
`c = 3`
`c = 5`


**4.函数重载的调用准则**

-  编译器调用重载函数的准则
    * 将所有同名函数作为候选者
    * 尝试寻找可行的候选函数
    * 精确匹配实参
    * 通过默认参数能够匹配实参
    * 通过默认类型转换匹配实参
    * 匹配失败
      * 最终寻找到的可行候选函数不唯一，则出现二义性，编译失败。
      * 无法匹配所有候选者，函数未定义，编译失败。

- 函数重载的注意事项
    * 重载函数在本质上是相互独立的不同函数（静态链编）
    * 重载函数的函数类型是不同的
    * 函数返回值不能作为函数重载的依据
    * 函数重载是由函数名和参数列表决定的。


- ==函数重载是发生在一个类中里面==


**5.函数重载遇上函数默认参数**
 
```cpp
//当函数默认参数遇上函数重载会发生什么

/*
函数重载和默认参数
*/

#include"iostream"
using namespace std;

int func(int a, int b, int c = 0)
{
	return a * b * c;
}

int func(int a, int b)
{
	return a + b;
}

//1个参数的允许吗 可以

int func(int a)
{
	return a; 
}

int main()
{
	int c = 0;
	//c = func(1, 2); // 存在二义性，调用失败，编译不能通过
	
	c = func(1);
	cout << "c = " << c << endl;

	return 0;
}
```
程序运行结果：`c = 1`


**6.函数重载和函数指针结合**

- 函数重载与函数指针（表示函数入口地址）
    - 当使用重载函数名对函数指针进行赋值时
    - 根据重载规则挑选与函数指针参数列表一致的候选者
    - 严格匹配候选者的函数类型与函数指针的函数类型

```cpp
int func(int x) // int(int a)
{
 return x;
}

int func(int a, int b)
{
 return a + b;
}

int func(const char* s)
{
 return strlen(s);
}

typedef int(*PFUNC)(int a); // int(int a)

int main()
{
 int c = 0;
 PFUNC p = func;
 c = p(1);//根据参数确定最终调用哪一个函数

 printf("c = %d\n", c);
 return 0;

}
```



**7.函数重载、重写、重定义**

后续课程。


## 八、附录

**附录1：C++语言对C语言扩充和增强的几点具体体现**
![附录1-1]($resource/%E9%99%84%E5%BD%951-1.png)
![附录1-2]($resource/%E9%99%84%E5%BD%951-2.png)
![附录1-3]($resource/%E9%99%84%E5%BD%951-3.png)
![附录1-4]($resource/%E9%99%84%E5%BD%951-4.png)
![附录1-5]($resource/%E9%99%84%E5%BD%951-6.png)

**附录2：C语言register关键字—最快的关键字**

register：这个关键字请求编译器尽可能的将变量存在CPU内部寄存器中，而不是通过内存寻址访问，以提高效率。**注意是尽可能，不是绝对**。你想想，一个CPU 的寄存器也就那么几个或几十个，你要是定义了很多很多register 变量，它累死也可能不能全部把这些变量放入寄存器吧，轮也可能轮不到你。

**一、皇帝身边的小太监----寄存器**

~~不知道什么是寄存器？那见过太监没有？没有？其实我也没有。没见过不要紧，见过就麻烦大了。^_^，大家都看过古装戏，那些皇帝们要阅读奏章的时候，大臣总是先将奏章交给皇帝旁边的小太监，小太监呢再交给皇帝同志处理。这个小太监只是个中转站，并无别的功能。  好，那我们再联想到我们的CPU。CPU 不就是我们的皇帝同志么？大臣就相当于我们的内存，数据从他这拿出来。那小太监就是我们的寄存器了（这里先不考虑CPU 的高速缓存区）。数据从内存里拿出来先放到寄存器，然后CPU 再从寄存器里读取数据来处理，处理完后同样把数据通过寄存器存放到内存里，CPU 不直接和内存打交道。这里要说明的一点是:小太监是主动的从大臣手里接过奏章，然后主动的交给皇帝同志，但寄存器没这么自觉，它从不主动干什么事。一个皇帝可能有好些小太监，那么一个CPU 也可以有很多寄存器，不同型号的CPU 拥有寄存器的数量不一样。  为啥要这么麻烦啊？速度！就是因为速度。寄存器其实就是一块一块小的存储空间，只不过其存取速度要比内存快得多。进水楼台先得月嘛，它离CPU 很近，CPU 一伸手就拿到数据了，比在那么大的一块内存里去寻找某个地址上的数据是不是快多了？那有人问既然它速度那么快，那我们的内存硬盘都改成寄存器得了呗。我要说的是：你真有钱！~~


**二、举例**

   register修饰符暗示编译程序相应的变量将被频繁地使用，如果可能的话，应将其保存在CPU的寄存器中，以加快其存储速度。例如下面的内存块拷贝代码，

```cpp
#ifdef NOSTRUCTASSIGN

memcpy(d, s, l)
{
	register char *d;
	register char *s;
	register int i;

	while (i--)
	*d++ = *s++;
}

#endif
```




**三、使用register 修饰符的注意点**

 但是使用register修饰符有几点限制。

　　首先，register**变量必须是能被CPU所接受的类型**。这通常意味着register变量**必须是一个单个的值**，并且**长度应该小于或者等于整型的长度**。不过，有些机器的寄存器也能存放浮点数。

　　其次，因为register变量可能不存放在内存中，所以不能用“&”来获取register变量的地址。

　~~由于寄存器的数量有限，而且某些寄存器只能接受特定类型的数据（如指针和浮点数），因此真正起作用的register修饰符的数目和类型都依赖于运行程序的机器，而任何多余的register修饰符都将被编译程序所忽略。~~

　　~~在某些情况下，把变量保存在寄存器中反而会降低程序的运行速度。因为被占用的寄存器不能再用于其它目的；或者变量被使用的次数不够多，不足以装入和存储变量所带来的额外开销。~~

　　早期的C编译程序不会把变量保存在寄存器中，除非你命令它这样做，这时register修饰符是C语言的一种很有价值的补充。然而，随着编译程序设计技术的进步，在决定那些变量应该被存到寄存器中时，现在的C编译环境能比程序员做出更好的决定。实际上，许多编译程序都会忽略register修饰符，因为尽管它完全合法，但它仅仅**是暗示而不是命令**。

 |

## 九、作业及强化训练

1 复杂数据类型引用做函数参数

  分析内存四区变化图

2 代码敲一遍

3 设计一个类, 求圆形的周长

4 设计一个学生类,属性有姓名和学号,

可以给姓名和学号赋值

可以显示学生的姓名和学号



