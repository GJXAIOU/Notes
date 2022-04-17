# C_PP_章五 纯虚函数和抽象类


## 一、基本概念

**纯虚函数：**
* 纯虚函数声明格式： `virtual 返回值类型   函数名（参数列表）= 0 `
* 纯虚函数是一个在基类中声明的虚函数，但是在基类中并没有定义（即没有具体的代码实现），但是要求任何的派生类都要实现属于自己的这个函数；
* 纯虚函数为各派生类提供一个公共界面（接口的封装和设计、软件的模块功能划分）

**抽象类：**
  一个具有纯虚函数的基类被称为抽象类

**注意点：**
```cpp
class shape  //抽象类
{
public:
  void where()
  {
    return center;
  }
  virtual void rotate(int b) = 0;
  virtual void draw() = 0;

};

int main()
{
  shape s;  //错误：抽象类不能建立对象
  shape *s; //可以：可以声明抽象类的指针
  shape f();//错误：抽象类不能作为返回类型
  vois g(shape); //错误：抽象类不能作为参数类型
  shape &h(shape &);  //可以：可以声明抽象类的引用
  return 0;
}
```

![image073]($resource/image073.png)



## 二、抽象类案例

```cpp
//纯虚函数抽象类语法基础


#include "iostream"
using namespace std;

class Figure
{
public:
	virtual void Area() = 0;   //声明一个纯虚函数，基类中不需要实现，在下面每一个继承类都需要按照各自需求进行实现
protected:
private:
	int a;
	int b;
};

class circle :public Figure
{
public:
	circle(int r,int a, int b)
	{
		this->r = r;
	}

	virtual void Area()  //子类中必须实现这个函数
	{
		cout << "圆的面积 " << 3.14*r*r << endl;
	}
private:
	int r;
};


class rectangle :public Figure
{
public:
	rectangle(int length, int width, int a, int b)
	{
		this->length = length;
		this->width = width;
	}

	virtual void Area()  //子类中必须实现这个函数
	{
		cout << "长方形的面积 " << length * width << endl;
	}
private:
	int length;
	int width;
};

void getArea(Figure *base)
{
	base->Area();
}



int main()
{
	
	//Figure f1;//这是错误的，抽象类不能实例化
	Figure *p = NULL;   //但是可以使用指针和引用
	circle c1(2, 1, 2);
	rectangle r1(1, 2, 3, 4);

	getArea(&c1);
	getArea(&r1);
	system("pause");
	return 0;
} 
```


## 三、抽象类在多继承中的应用

C++中没有Java中的接口概念，抽象类可以模拟Java中的接口类。（接口和协议）



### （一）有关多继承的说明

**工程上的多继承**

* 被实际开发经验抛弃的多继承
* 工程开发中真正意义上的多继承是几乎不被使用的
* 多重继承带来的代码复杂性远多于其带来的便利
* 多重继承对代码维护性上的影响是灾难性的
* 在设计方法上，任何多继承都可以用单继承代替 

**多继承中的二义性和多继承不能解决的问题**
![image076]($resource/image076.png)


### （二）多继承的应用场景

* 绝大多数面向对象语言都不支持多继承
* 绝大多数面向对象语言都支持接口的概念
* C++中没有接口的概念
* C++中可以使用纯虚函数实现接口
* 接口类中只有函数原型定义，没有任何数据的定义


实际工程经验证明
* 多重继承接口不会带来二义性和复杂性等问题 
* 多重继承可以通过精心设计用单继承和接口来代替
* 接口类只是一个功能说明，而不是功能实现。
* 子类需要根据功能说明定义功能实现。

抽象类在多继承中的应用的示例程序：
```cpp
//抽象类在多继承中的应用

#include "iostream"
using namespace std;

class Interface1
{
public:
	virtual void print() = 0;
	virtual int add(int a, int b) = 0;
};

class Interface2
{
public:
	virtual void print() = 0;
	virtual int add(int a, int b) = 0;
	virtual int minus(int a, int b) = 0;
};

class parent
{
public:
	int a;
};

class Child : public parent, public Interface1, public Interface2
{
public:
	void print()
	{
		cout << "Child::print" << endl;
	}

	int add(int a, int b)
	{
		return a + b;
	}

	int minus(int a, int b)
	{
		return a - b;
	}

};

int main()
{

	Child c;
	c.print();
	cout << c.add(3, 5) << endl;
	cout << c.minus(4, 6) << endl;

	Interface1* i1 = &c;
	Interface2* i2 = &c;

	cout << i1->add(7, 8) << endl;
	cout << i2->add(7, 8) << endl;

	system("pause");

}
```




## 四、抽象类知识点强化

/*
编写一个C++程序, 计算程序员( programmer )工资 
 1 要求能计算出初级程序员( junior_programmer ) 中级程序员 ( mid_programmer )高级程序员( adv_programmer)的工资
 2 要求利用抽象类统一界面,方便程序的扩展, 比如:新增, 计算 架构师 (architect ) 的工资 
*/
~~调用的时候参数有点问题~~
```cpp
//抽象类知识点的强化


#include "iostream"
#include "sstream"
using namespace std;

class programmer
{
public:
	virtual void getSal() = 0;
protected:
private:
};

class junior_programmer :public programmer
{
public:
	junior_programmer(char *name, char *job, int sale)
	{
		this->name = name;
		this->job = job;
		this->sale = sale;
	}

	virtual void getSal()
	{
		cout << "junior_programmer:" << endl;
		cout << "name = " << name << "  job:" << job << " sale :" << sale << endl;
	}
protected:
private:
	char *name;
	char *job;
	int sale;
};

class mid_programmer :public programmer
{
public:
	mid_programmer(char *name, char *job, int sale)
	{
		this->name = name;
		this->job = job;
		this->sale = sale;
	}

	virtual void getSal()
	{
		cout << "mid_programmer:" << endl;
		cout << "name = " << name << "  job:" << job << " sale :" << sale << endl;
	}
protected:
private:
	char *name;
	char *job;
	int sale;
};



class adv_programmer :public programmer
{
public:
	adv_programmer(char *name, char *job, int sale)
	{
		this->name = name;
		this->job = job;
		this->sale = sale;
	}

	virtual void getSal()
	{
		cout << "adv_programmer:" << endl;
		cout << "name = " << name << "  job:" << job << " sale :" << sale << endl;
	}
protected:
private:
	char *name;
	char *job;
	int sale;
};


//接下来为后期可以拓展的部分


class architect :public programmer
{
public:
	architect(char *name, char *job, int sale)
	{
		this->name = name;
		this->job = job;
		this->sale = sale;
	}

	virtual void getSal()
	{
		cout << "architect:" << endl;
		cout << "name = " << name << "  job:" << job << " sale :" << sale << endl;
	}
protected:
private:
	char *name;
	char *job;
	int sale;
};

//以上为后期拓展部分

void pro_sale(programmer *base)
{
	base->getSal();
}


int main()
{
	junior_programmer jp("张三","初级",2000);
	mid_programmer mp("小张", "中级", 8000);
	adv_programmer ap("小李", "高级", 20394);

	pro_sale(&jp);
	pro_sale(&mp);
	pro_sale(&ap);


	//后期新增程序的调用

	architect ar("架构师", "牛", 121332);
	pro_sale(&ar);
	system("pause");
	return 0; 

}
```




## 五、面向抽象类编程思想强化

理论知识

Ø 虚函数和多态性使成员函数根据调用对象的类型产生不同的动作

Ø  多态性特别适合于实现分层结构的软件系统，便于对问题抽象时  定义共性，实现时定义区别

Ø **面向抽象类编程（面向接口编程）**是项目开发中重要技能之一。


### （一）案例：socket库c++模型设计和实现

**企业信息系统框架集成第三方产品**

**案例背景**：一般的企业信息系统都有成熟的框架。软件框架一般不发生变化，能自由的集成第三方厂商的产品。

**案例需求**：请你在**企业信息系统框架**中集成第三方厂商的Socket通信产品和第三方厂商加密产品。

第三方厂商的Socket通信产品：完成两点之间的通信；

第三方厂商加密产品：完成数据发送时加密；数据解密时解密。

![image077]($resource/image077.png)

**案例要求**： 1）能支持多个厂商的Socket通信产品入围

 2）能支持多个第三方厂商加密产品的入围

 3）企业信息系统框架不轻易发生框架

**需求实现**

  思考1：企业信息系统框架、第三方产品如何分层

思考2：企业信息系统框架，如何自由集成第三方产品

（软件设计：模块要求松、接口要求紧）

  思考3：软件分成以后，开发企业信息系统框架的程序员，应该做什么？

  第三方产品入围应该做什么？

**编码实现**

分析有多少个类 CSocketProtocol  CSckFactoryImp1  CSckFactoryImp2

CEncDesProtocol  HwEncdes  ciscoEncdes

1、 定义 CSocketProtocol 抽象类

2、 编写框架函数

3、 编写框架测试函数

4、 厂商1（CSckFactoryImp1）实现CSocketProtocol、厂商2（CSckFactoryImp1）实现CSocketProtocol

5、 抽象加密接口（CEncDesProtocol）、加密厂商1(CHwImp)、加密厂商2(CCiscoImp))，集成实现业务模型

6、 框架（c语言函数方式，框架函数；c++类方式，框架类）

~~des.cpp程序没有，所以第7天的第17-20视频没有看~~

**几个重要的面向对象思想**

继承-组合（强弱）

注入

控制反转 IOC

MVC

  面向对象思想扩展aop思想

 aop思想是对继承编程思想的有力的补充


### （二）案例：计算员工工资


### （三）案例：计算几何体的表面积和体积
~~第21个视频上有~~










## 六、面向接口编程和C多态（在C语言中是实现与C++多态相同的功能）-贴近实战、较难


**前言：数组指针语法梳理 **
* 数组类型语法
* 数组指针类型
* 数组指针变量
```cpp
#include<stdio.h>

//数组类型基本语法的知识梳理

//1.定义一个数组类型


//2.定义一个指针数组类型



//3.定义一个指向 数组类型的指针（即 数组类指针）


int  main()
{

	//默认定义数组方式
	int  a[10];   //其中a代表数组首元素的地址，&a代表整个数组的地址， a+1 的步长为4，&a+1的步长为10


	//使用typedef定义数组类型
	typedef int(MyTypeArray)[10];
	//定义变量
	MyTypeArray myArray;
	myArray[0] = 10;


	//定义一个指针数组类型
	typedef int (*PTypeArray)[10];
	PTypeArray *myPArray;
	myPArray = &a;   //指向a数组的首地址
	(*myPArray)[0] = 20;  //操作元素
	a[0] = 10;

	//3.定义一个指向 数组类型的指针（即 数组类指针） 相当于直接定义变量而不是先定义类型在定义指针

	int(*myPString)[10];
	myPString = &a;
	(*myPString)[0] = 10;


}
```



### （一）函数类型语法基础

**基础概念：**
- 函数三要素：  名称、参数、返回值
- C语言中的函数有自己特定的类型
- C语言中通过typedef为函数类型重命名
  * typedef type name(parameter list)
  * typedef int f(int, int);
  * typedef void p(int);

**函数指针**

* 函数指针用于指向一个函数

* 函数名是函数体的入口地址

  * 1)可通过函数类型定义函数指针: FuncType* pointer;
  * 2)也可以直接定义：type (*pointer)(parameter list);

    * pointer为函数指针变量名
    * type为指向函数的返回值类型
    * parameter list为指向函数的参数类型列表



**函数指针语法梳理**
- 函数类型
- 函数指针类型
- 函数指针变量
以上三个的实现程序如下：
```cpp
//函数类型

//1.如何定义一个函数类型
//2.如何定义一个函数指针类型
//3.如何定义一个函数指针（指向函数的入口）

int add(int a, int b)
{
	printf("func is add a and b");
	return a + b;
}

void main()
{
	//直接调用
	add(1, 2);

	//定义一个函数类型
	typedef int (MyFuncType)(int a, int b);//定义了一个类型
	MyFuncType *myPointerFunc = NULL;//定义了一个指针，指向某一种类的函数，这里的某一类是根据定义来判断：
	//这里某一类是指含有两个参数，返回值为int 的类型

	myPointerFunc = &add;
	myPointerFunc(3, 4); //间接调用
	myPointerFunc = add;  //这里是否取地址都可以
	myPointerFunc(3, 4); //间接调用


	//定义一个函数指针类型
	typedef int(*MyPointerFuncType)(int a, int b);   //相当于定义：int *a = NULL;

	MyPointerFuncType MyPointer;//定义一个指针
	MyPointer = add;
	MyPointer(5, 6);


	//直接定义一个函数指针(相当于直接定义了一个变量)
	int(*MyPointerFunc)(int a, int b);//定义了一个变量
	MyPointerFunc = add;
	MyPointerFunc(7, 8);


}
```


### （二）函数指针做函数参数

**函数指针做函数参数概念**

当函数指针 做为函数的参数，传递给一个被调用函数，

被调用函数就可以通过这个指针调用外部的函数，这就形成了回调

练习
```cpp
int add(int a, int b) 
int libfun( int (*pDis)(int a, int b) );

int main(void)
{
 int (*pfun)(int a, int b);  
 pfun = add;
 libfun(pfun);
}

int add(int a, int b)
{
 return a + b;
}

int libfun( int (*pDis)(int a, int b) )
{
  int a, b;
  a = 1;
  b = 2;
add(1,3) //直接调用add函数

printf("%d", pDis(a, b)); //通过函数指针做函数参数,间接调用add函数

//思考 这样写 pDis(a, b)有什么好处?

}
```


**剖析思路**

//1函数的调用 和 函数的实现  有效的分离
//2 C++的多态,可扩展

现在这几个函数是在同一个文件当中，但是假如int libfun(int (*pDis)(int a, int b))是一个库中的函数，就只有使用回调了，通过函数指针参数将外部函数地址传入来实现调用，即使函数 add 的代码作了修改，也不必改动库的代码，就可以正常实现调用，这样便于程序的维护和升级

回调函数思想：
![image078]($resource/image078.png)
**结论：回调函数的本质：提前做了一个协议的约定（把函数的参数、函数返回值提前约定）**

请思考：C编译器通过那个具体的语法，实现解耦合的？

 C++编译器通过多态的机制(提前布局vptr指针和虚函数表,找虚函数入口地址来实现)


**函数指针做函数参数思想剖析：**
```cpp
//函数指针做函数参数的思想剖析
#include<stdio.h>


int add(int a, int b) //自任务的实现者
{
	printf("func is add a and b");
	return a + b;
}

//假设这是后来实现的代码：
int add2(int a, int b) //自任务的实现者
{
	printf("func2 is add a and b");
	return a + b;
}




//定义一个函数指针类型
typedef int(*MyPointerFuncType)(int a, int b);   //相当于定义：int *a = NULL;


//函数指针做函数参数
int MainOp(MyPointerFuncType myFuncAdd)
{
	 int c = myFuncAdd(5, 6);
	 return c;
}
//函数指针做函数参数的第二种方式
int MainOp2(int(*MyPointerFuncType)(int a, int b))
{
	int d = MyPointerFuncType(5, 6);
	return d;
}



void main()
{

	MyPointerFuncType myFuncAdd = NULL;
	add(1, 2);


	MyPointerFuncType MyPointer;//定义一个指针
	MyPointer = add;
	MyPointer(5, 6);


	//具体的调用
	MainOp(add);
	MainOp2(add);

	//加上后来实现者之后，在MainOP框架没有任何改变的情况下实现对新增函数的调用
	MainOp2(add2);

	return 0;
}

```
          


### （三）函数指针正向调用

1、 函数指针做函数参数，调用方式

被调用函数和主调函数在同一文件中（用来教学，没有任何意义）


2、函数指针做函数参数

被调用函数和主调函数不在同一个文件中、模块中。

难点：理解被调用函数是什么机制被调用起来的。框架

框架提前设置了被调用函数的入口（框架提供了第三方模块入口地址的集成功能）

框架具备调用第三方模块入口函数

3、 练习
```cpp

```

typedef int (*EncDataFunc)(unsigned char *inData,int inDataLen,unsigned char *outData,int *outDataLen,void *Ref, int RefLen);

int MyEncDataFunc(unsigned char *inData,int inDataLen,unsigned char *outData,int *outDataLen,void *Ref, int RefLen)

{

 int rv = 0;

 char *p = "222222222222";

 strcpy(outData, p);

 *outDataLen = strlen(p);

 return rv;

}

int Send_Data(EncDataFunc encDataFunc, unsigned char *inData, int inDataLen, unsigned char *outData, int *outDatalen)

{

 int rv = 0;

 if (encDataFunc != NULL)

 {

 rv = encDataFunc(inData, inDataLen, outData, outDatalen, NULL, 0);

 if (rv != 0)

 {

 printf("func encDataFunc() err.\n");

 return rv;

 }

 }

 return rv;

}

int main()

{

 int rv = 0;

 EncDataFunc encDataFunc = NULL;

 encDataFunc = MyEncDataFunc;

 // 第一个调用

 {

 unsigned char inData[2048];

 int inDataLen;

 unsigned char outData[2048];

 int outDatalen;

 strcpy(inData, "1111");

 inDataLen = strlen(inData);

 rv = encDataFunc(inData,inDataLen, outData, &outDatalen, NULL, 0);

 if (rv != 0)

 {

 printf("edf err .....\n");

 }

 else

 {

 printf("edf ok \n");

 printf("%s \n", outData);

 }

 }

 {

 unsigned char inData[2048];

 int inDataLen;

 unsigned char outData[2048];

 int outDatalen;

 strcpy(inData, "3333");

 inDataLen = strlen(inData);

 rv = Send_Data(MyEncDataFunc, inData, inDataLen, outData, &outDatalen);

 if (rv != 0)

 {

 printf("func Send_Data err:%d", rv);

 return rv;

 }

 printf("%s \n", outData);

 }

 getchar();

}

 |

### （四）函数指针反向调用

回调函数效果展示。


### （五）C动态库升级成框架案例

**C****语言版本Socket****动态库升级成框架集成第三方产品**

**简称：C****动态库升级成框架案例**

**名字解释       **

  动态库：抽象类一个套接口，单独封装成模块，供别人调用；无法扩展。

  框架：能自由的扩展 

**案例背景**：一般的企业信息系统都有成熟的框架，可以有C语言写，也可以由C++语言。软件框架一般不发生变化，能自由的集成第三方厂商的产品。

**案例需求：**在socket通信库中，完成数据加密功能，有n个厂商的加密产品供你选择，如何实现动态库和第三个厂商产品的解耦合。

提醒：C++通过抽象类，也就是面向抽象类编程实现的（相当于C++编译器通过多态机制，已经很好用了。提前布局vptr指针、虚函数表；调用是迟绑定完成。），

**C语言**中如何实现哪？

**案例要求**： 1）能支持多个第三方厂商加密产品的入围

 2）企业信息系统框架不轻易发生框架

**需求实现思路分析**
![image079]($resource/image80.png) 
  思考1：企业信息系统框架、第三方产品如何分层 

思考2：企业信息系统框架，如何自由集成第三方产品

（软件设计：模块要求松、接口要求紧）

  思考3：软件分层确定后，动态库应该做什么？产品入围厂商应该做什么？

以后，开发企业信息系统框架的程序员，应该做什么？

  第三方产品入围应该做什么？

**编码实现**

1、 动态库中定义协议，并完成任务的调用

typedef int (*EncData)(unsigned char *inData,int inDataLen,unsigned char *outData,int *outDataLen,void *Ref, int RefLen);

typedef int (*DecData)(unsigned char *inData,int inDataLen,unsigned char *outData,int *outDataLen,void *Ref, int RefLen);

2、 加密厂商完成协议函数的编写

3、 对接调试。

4、 动态库中可以缓存第三方函数的入口地址，也可以不缓存，两种实现方式。

**案例总结**

  回调函数：利用函数指针做函数参数，实现的一种调用机制，具体任务的实现者，可以不知道什么时候被调用。

  回调机制原理：

  当具体事件发生时，调用者通过函数指针调用具体函数

  回调机制的将调用者和被调函数分开，两者互不依赖

  任务的实现 和 任务的调用 可以耦合  （提前进行接口的封装和设计）

### （六）附录：诸葛亮的锦囊妙计**

诸葛亮对刘备，暗暗关照赵云道：“我这里有三个锦囊，内藏三条妙计。到南徐时打开第一个，到年底时打开第二个，危急无路时打开第三个。”

第一个锦囊

  一到东吴就拜会乔国老

第二个锦囊

  刘备被孙权设计留下就对他谎称曹操大军压境

第三个锦囊

被东吴军队追赶就求孙夫人解围
