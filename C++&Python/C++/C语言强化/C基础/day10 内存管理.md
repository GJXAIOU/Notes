# day10 内存管理
@toc


## 一、变量作用域

全局变量和局部变量

一个C语言变量的作用域可以是代码块，作用域就是函数作用域或者文件作用域
代码块：{}之间的一段代码；



### （一）零碎知识点


**1.文件作用域：**

如果一个变量在其他的代码文件中已经定义了，可以在本程序中使用，但是使用前要使用关键字：`extern`
例如：`extern int age` //有一个int型变量age已经在其他文件中定义了，这里就直接使用了；

`static int a  = 0;`//在文件外部不可用

---

**2.auto自动变量**

C语言中所有的局部变量默认中都是auto，，所以`auto`可以省略。不需要关注它在内存中什么时候消失和出现
`auto int i = 0;`等效于`int i = 0;`

----

**3.register变量**
`register int i = 0;`  //建议，如果有寄存器空闲的话，就将这个变量放进寄存器中使用，可以加快读取

但是`int *p = &i`//这个语句就会报错，因为放在寄存器中就没有内存地址了

---

### （二）static使用

#### 1、修饰变量：动态变量和静态变量

- **动态变量**

```c
#include<stdio.h>

void myauto()
{
  int a = 0;
  printf("a = %d\n",a);
  a++;
}


int main()
{
  int i;
  for(i = 0; i < 5; i++)
  {
    myatuo();
 
  }

}
```

程序的输出结果为 : 
`a = 0`
`a = 0`
`a = 0`
`a = 0`
`a = 0`
                          

- **静态变量**
首先只要整个程序开始执行之后，**静态变量是一直存在的**，不消失的；
其次，**静态变量值初始化一次**，即`static int a = 0;`语句只执行一次
```c
#include<s tdio.h>

void mystatic()
{
  static int a = 0;//整个进程运行过程中一直有效，是在静态区，但是只能mystatic函数内部访问使用
  printf("a = %d\n",a);
  a++;

}

int main()
{
  int i = 0;
  for(i = 0; i < 5; i++)
  {
    mystatic();
  }
}
```
程序运行结果为:
`a = 0`
`a = 1`
`a = 2`
`a = 3`
`a = 4`

- **总结**
 **static定义的静态局部变量：**
  - 静态局部变量在函数内存定义的，**其生存周期为整个源程序**，但是作用域同自动变量，**只能在定义该变量的函数内部使用**；退出该函数之后该变量仍然存在只是不能使用；
  - 静态变量未赋初值的系统自动赋值0；其他自动变量赋值则具有随机性；
    
  **static定义的静态全局变量：**
  - 仍然采用静态存储方式，但是**作用域为定义该变量的源文件内部** ，非静态的全局变量的作用域为这个源程序（包括多个源文件）

#### 2.static修饰函数

使用static定义的函数只能在本文件中被调用



### （三）extern使用
**变量和函数是否使用extern的区别:**

- 变量 
  - extern int age;//当age这个变量是在另一个.c文件中时，需要在别的文件中调用这个变量时候使用
  - int age;    //这包含两个含义：声明一个变量或者定义一个变量
  
- 函数（下面两个没有什么区别）
  - extern void age()
  - void age()     


---


## 二、内存四区简介

**研究作用：**
- 生命周期
- 作用域

![内存四区图示]($resource/%E5%86%85%E5%AD%98%E5%9B%9B%E5%8C%BA%E5%9B%BE%E7%A4%BA.png)

**流程说明：**
1、操作系统把物理硬盘代码load到内存
2、操作系统把c代码分成四个区
3、操作系统找到main函数入口执行

**分区：**

- **1.代码区**

存放可执行的代码；程序被操作系统加载到内存中时候，所有可执行的代码都加载到代码区，==这块内存是不可以在运行期间修改的====；



- **2.静态区**

存放所有的==静态变量/常量（即使用static关键字的）和全局变量/常量==


 - **3.栈区**

栈区是一种先进后出的内存结构，所有的==自动变量、指针，数组、函数的形参、局部变量==都是由编译器自动放出栈中，当一个自动变量超出其作用域的时候，自动从栈中弹出；

对于自动变量，什么时候入栈和出栈都是系统自动控制的

栈区的大小是可以动态设置的，不过不会特别大，一般大小为XX k，

C语言中函数的==参数变量是从右往左入栈==的，也就是最右边的参数是最先入栈的
![栈的图示]($resource/%E6%A0%88%E7%9A%84%E5%9B%BE%E7%A4%BA.png)

**栈溢出**
当栈空间已满，但是还往栈内存中压变量，这就是栈溢出；

![堆栈的生长方向]($resource/%E5%A0%86%E6%A0%88%E7%9A%84%E7%94%9F%E9%95%BF%E6%96%B9%E5%90%91.png)


- **4.堆区**

堆和栈一样，是一种在程序运行过程中可以随时修改的内存区域，但是==没有栈那样的先进后出的顺序==，是一个大容器，容量远远大于栈，==堆内存空间的申请和释放需要手工通过代码来完成==；如果没有释放系统会在程序结束的时候回收

**4.1.分配堆的方法：**
```c
#include<stdio.h>
#include<stdlib.h>   //需要加一个头文件

int main()
{
    int *p = malloc(sizeof(int)*10);  //语句含义为：在堆中间申请内存，函数的返回值为无类型的指针，参数为size_t无符号的整数   ，现在这个语句的含义是：申请一个内存大小为10个int大小的空间。指针p指向分配出来的堆的地址；

    memset(p ,0,sizeof(int) * 10);  //将分配的这个空间置零初始化
    free(p);//释放通过malloc分配的堆内存
    p = NULL;
    return 0;
}
```


---


**4.2.堆和栈的区别**

```c
#include<stdio.h>
#include<stdlib.h>

void *geta()   //这是错误的，不能将一个栈变量的地址通过函数的返回值返回,因为函数执行结束之后变量就完全释放了
{
  int a = 0;
  return &a;
}


void *geta1() //可以通过函数的返回值返回一个堆地址，但是一定要free
{
  int  *p = malloc(sizeof(int)); //申请了一个堆空间
  return p;  //返回值为p 即是分配的堆空间的地址
}



void *gata2()   //这也是合法的，只要将main函数中的geta1改为geta2,然后将free(getp)去掉即可
{
  static int a = 0;  //使用static即使a为静态变量，存在于静态区，会一直存在不会被释放
  return &a;
}




int main()

{
  int *getp = geta1();
  *getp = 100;
  free(getp);   //getp指向了划分的堆区域，因此释放即可
}


```

**理解下面程序**
~~这个程序本质上就是一个错误程序例子，具体的修改见下面的一个程序~~
```c
#include<stdio.h>
#include<stdlib.h>

void getheap(int  *a)
{
  a = malloc(sizeof(int) * 10);
}

int main()
{
  int *p =NULL;
  gatheap(p);  //实参没有任何改变，相当于这里的p一直没有任何改变

  p[0] = 1;
  p[1] = 2;
  printf("p[0] = %d ,p[1] = %d\n",p[0],p[1]);
  free(p);
}
```

![函数图形化解析]($resource/%E5%87%BD%E6%95%B0%E5%9B%BE%E5%BD%A2%E5%8C%96%E8%A7%A3%E6%9E%90.png)

从这程序中可以看出：也就是说a的值改了，a本身是在栈里面的，它只是指向堆里面的一个空间地址；然后实参p的值一直没有变化
getheap在执行完之后，a就消失了，导致他指向的具体堆空间的地址编号也随之消失了；

**修改**

```c
#include<stdio.h>
#include<stdlib.h>

void  getheap(int  **a)
  {
     *a  =  malloc(sizeof(int) * 10);  //这里的*a就是指针p中的值了，
  }

/*
  int **a = &p;等价于
  int **a;
  a = &p

*/



int  main()
{
  int  *p  =NULL;
  getheap(&p);  //传递的是P的地址
  p[0]  =  1;
  p[1]  =  2;
  printf("p[0]  =  %d  ,p[1]  =  %d\n",p[0],p[1]);
  free(p);
  return  0;

}
```

程序的分析图片：

![堆和栈地址]($resource/%E5%A0%86%E5%92%8C%E6%A0%88%E5%9C%B0%E5%9D%80.png)



### 三、内存模型详解以及Linux系统堆内存大小的分析

**1.栈和堆的比较**

- 栈(stack)
 - 明确知道数据占用多好内存
 - 数据很少 
 - 变量离开作用范围后，栈上的数据会自动释放
 - 栈的最大尺寸固定，超过则引起栈溢出
  
- 堆(heap)
  - 需要大量内存的时候
  - 不知道需要多少内存 

```c
//在堆中可以建立一个动态的数组

#include<stdio.h>

int main()
{

  int i ;
  scanf("%d",&i);
  int *array = malloc(sizeof(int) * i);
  free(array);


}
```

---

**2.堆的分配和释放**

内存的最小单位是字节，但是操作系统在管理内存的时候，最小的单位是内存页（32位操作系统中每一页为4k ）

---

**3.malloc和calloc和realloc**

- **malloc** 这里分配的空间里面可能有数据，需要手动清空
```c
#include<stdio.h>
#include<stdio.h>
int main()
{
  char *p = malloc(10);//分配一个10个字节的空间,但是这10个空间没有清理过，所以每次使用可能都是10个随机数

  memset(p,0,10); //这里含义是将p 中的10个元素全部置零
  int i = 0;
  for(i = 0; i < 10; i++)
  {
    printf("%d\n",p[i]);
  }
  free(p);
  return 0;
}
```

- **calloc**分配的堆空间已经自动的清空（置零）

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h >

int main()
{
  char *p = calloc(10,sizeof(char));//分配10个大小为sizeof（char）的空间，并且自动全部置零
  int i = 0;
  for(i = 0; i < 10; i++)
  {
    printf("%d\n",p[i]);
  }
  return 0;
  free(p);
}
```

- **realloc** 划分的空间与已有的空间地址是连续的

如果已分配的空间不够用，需要划分出一块新的内存空间，要求是这块内存空间和原来已划分的空间是连续的；
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main()
{
  char *p = calloc(10,sizeof(char));//这10个空间是置零了的
  char *p1 = realloc(p,20);
  //参数含义：需要扩展的空间名，需要扩展后的空间为20个字节
  //语句含义：在原有内存基础之上，在堆中间增加连续的内存；如果原有的内存没有连续的空间可拓展，那么会分配一个空间，将原有的内存copy到新空间，然后释放
  //新开辟的这10个字节空间是没有置零的

  int i = 0;
  for(i = 0; i < 20; i++)
  {
    printf("%d\n",p1[i]); //这里的p也需要改为p1
  }
  return 0;
  free(p1);  //这里释放的是p1
}
```

当要减少原来已划分的空间的时候
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main()
{
  char *p = calloc(10,sizeof(char));//这10个空间是置零了的
  char *p1 = realloc(p,5);
  
  int i = 0;
  for(i = 0; i < 5; i++)
  {
    printf("%d\n",p1[i]); //这里的p也需要改为p1
  }
  return 0;
  free(p1);  //这里释放的是p1
}
```

当`realloc`的第一个参数为空的时候，即：`realloc (NULL,5)`等效于`malloc(5)`


**4.二级指针分派堆空间**

如果是通过一个函数的参数给实参分配堆空间内存，那么一定是二级指针的形式
**正确模型1**
```c
#include<stdio.h>

void getheap1(int  **p)
{
  *p = malloc(100);

}
int main()
{
  int *p = NULL;
  getheap(&p);//如果是通过一个函数的参数给实参分配堆空间内存，肯定使用二级指针

  free(p);

}

```

**正确模型2**

```c
#include<stdio.h>

int *getheap2()
{
  return = malloc(100);

}
int main()
{
  int *p = NULL;

  p = getheap2();

  free(p);

}
```

**正确模型3：**
常量、静态变量、全局变量都在都在静态区域，一直是有效的
```c
#include<stdio.h>

int *getstring2()
{
  return = "hello";//可以将一个常量的地址作为函数返回值返回

}
int main()
{
 const char *ss = getstring2();
 printf("ss = %c\n",c);
 return 0;

}

```

**正确模型4**

```c
#include<stdio.h>

int *getstring4()
{
  static char array[10] = "hello";  //仍然在静态区
  return array;

}
int main()
{
 const char *s = getstring4();
 printf("s = %s\n",s);
 return 0;

}
```

程序输出结果为：`s  = hello`

**错误的模型1**

```c
#include<stdio.h>

int *getstring()
{
  char array[10] = "hello";
  return array;

}
int main()
{
  char *s = getstring();//当getarray()这个语句执行完之后，因为数组array是存放在栈中，地址就释放了
  printf("s = %s\n",s);
  return 0;
}
```
程序运行结果为乱码



**函数调用模型：**
下面我们具体总结一下，各个函数的变量的生命周期

![函数调用模型1]($resource/%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E6%A8%A1%E5%9E%8B1.png)

main里面的变量分配内存，函数fa(),函数fb()中的变量分配的内存空间它们的生命周期都是多长呢？

上述图1，已经说明了内存主要分为四区，因此每个函数中变量在堆栈的生命周期是不同的，

同时在函数调用的时候，先执行的函数最后才执行完毕

![函数调用模型2]($resource/%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E6%A8%A1%E5%9E%8B2.png)

例子程序：

```c

char*fa()
{

char*pa = "123456";//pa指针在栈区，“123456”在常量区，该函数调用完后指针变量pa就被释放了
 char*p = NULL;     //指针变量p在栈中分配4字节
 p=(char*)malloc(100);//本函数在这里开辟了一块堆区的内存空间，并把地址赋值给p
 strcpy(p, "wudunxiong 1234566");//把常量区的字符串拷贝到堆区
 return p;//返回给主调函数fb()，相对fa来说fb是主调函数，相对main来说，fa(),fb()都是被调用函数
}
char*fb()
{
 char*pstr = NULL;
 pstr = fa();
 return pstr;//指针变量pstr在这就结束
}

void main()
{ 
 char*str = NULL;
 str = fb();
 printf("str = %s\n",str);
 free(str);    //防止内存泄露，被调函数fa()分配的内存存的值通过返回值传给主调函数，然后主调函数释放内存
 str = NULL;//防止产生野指针
 system("pause");
}
```

**总结：**

1、主调函数分配的内存空间（堆，栈，全局区）可以在被调用函数中使用，可以以指针作函数参数的形式来使用

2、被调用函数分配的内存空间只有堆区和全局区可以在主调函数中使用（返回值和函数参数），而栈区却不行，因为栈区函数体运行完之后

这个函数占用的内存编译器自动帮你释放了。

3、一定要明白函数的主被调关系以及主被调函数内存分配回收，也就是后面接下几篇总结的函数的输入输出内存模型



