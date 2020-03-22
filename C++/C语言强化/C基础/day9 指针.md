# day9 指针
@toc

这里稍微留心一下：[指针难点](#四、指针难点)

## 一、指针的含义和定义

==指针存放的内容是一个地址，该地址指向一个内存空间；==

**1.指针的定义**
```c
#include<stdio.h>
int main()
{
  int a = 0;
  int b = 10;
  char buf[10];
  printf("%p,%p,%p\n",&a,&b,buf);   //&即是取变量地址，数组名表地址

  int *p = &a; //得到变量a的地址，将这个地址赋值给指针变量p
  //地址是一个整数，但是特殊性在于这个整数不能直接通过整数来操作


  //上面的赋值等效于以下语句：
  
  //int *p; //定义一个指针，名字叫做p，它指向于一个int的地址
  //p = &a;  //指针变量的值一般不能赋值一个整数，而是通过取变量地址的方式进行赋值。
  
  
  return 0;
}
```

通过指针获得指针所指内存的值，可以通过指针直接更改值的大小

---

**2.指针的使用**
```c
#include<stdio.h>

int main()
{
  int a = 3;
  int *p;
  p = &a;    //定义指针之后应该初始化或者赋值，否则会成野指针

  int b = *p;  //*p 表示指针变量指向内存的内容
  printf("b = %d\n",b);

  *p = 10;//通过指针间接的修改变量的值，这时候a的值为10； 
  printf("a = %d\n",a);
  return 0;

}
```

程序的运行结果为：`b = 3;` `a = 10;`  

---

**3.空指针与野指针的区别**

- **空指针：** 指向NULL的指针，当一个指针不指向任何一个有效的内存地址的时候，应该把指针设置成NULL
- **野指针 ：** 没有具体指向任何变量地址的指针（程序中应该避免野指针的存在）
  但是在VS中野指针是不报错的
  - **空指针程序例子**
```c
#include<stdio.h>

int main()
{
  int i = 0;
  void *p;  //定义无类型指针，意思是这只是一个指针变量，而不指向任何具体的数据类型；
  p = NULL; //将指针赋值NULL，值为NULL的指针，我们俗称为空指针，NULL其实就是0

  return 0;
}
```


---


## 二、指针与数组的关系

**1.指针的兼容性**

指针之间赋值要根据双方的数据类型来判断，==原则上一定是相同类型的指针指向相同类型的变量地址==，不能用一种类型的指针指向另一种类型的变量地址。

```c
#include<stdio.h>

int main()
{
  float a = 3.14;  
  int i = a;  //这是可以的：C语言会自动进行数据类型转换，将浮点数后面的小数部分舍弃；

  //int *p = &a;  //这个一般不可以：这是很严重的错误，指针类型不兼容

  printf("i = %d\n",i);
 // printf("*p = %d\n",*p);

  return 0;
}
```
程序运行结果：`i = 3`

---

**2.特殊的两类指针**

- **指向常量的指针**

定义：  const  数据类型   指针 

特点：==不能通过修改指针值（*p）来改变所指向内存空间存放的值==，但是可以通过变量赋值修改指向的值，可以修改指针指向的位置

```c
#include<stdio.h>

int main()
{
  int a = 10;
  int b = 20;

  const int *p = &a;  //p这个指针只能指向一个常量，可以直接改变a的赋值来改变a的值，但是不能通过改变*p的值来改变a的值；

  //*p = 30;  //这样不能修改，只会报错；

  a = 30;   //这样是可以的

  p = &b;   //这样也是可以的

  printf("%d\n",*p);
  return 0;

}
```

程序运行结果：`*p = 20`

---

- **常量指针**

定义：数据类型   *const   指针名  

特点：==不能修改指针指向的内存位置==，但是可以修改指向的内存的值以及读取这个变量的值

```c
#include<stdio.h>
int main()
{
  int a = 10;
  int b = 20;
  int *const p = &a;  //定义一个常量指针，可以通过修改常量指针修改或者读取一个变量的值
  *p = 30;
  // p = &b;   //这是错误的，常量指针一旦定义了，那么就不能修改其指向的变量
  printf("a = %d\n",*p);

  return 0;

}
```

程序运行结果：`a = 30`

---

**3.指针的加减对应指向内存的位置的改变**

```c
#include<stdio.h>

int main()
{
  int array[100] = {1,2,3,4,5,6,7,8,9,0};
  int *p = array;
  p += 5;   // ==这里p在内存中相当于移动了5*4（8）个字节==
  p -= 3;

  p[3] = 100;   //这时候实际上相当于array[5] 的值为100

  int i = 0;
  for(i = 0; i < 10; i++)
  {
  printf("array[%d] = %d\n",i,array[i]);
  
  }

  return 0;
}


```
程序运行的结果：
`array[0] = 1 `
`array[1] = 2`
`array[2] = 3`
`array[3] = 4`
`array[4] = 5`
`array[5] = 100`
`array[6] = 7`
`array[7] = 8`
`array[8] = 9`
`array[9] = 0`

---

**4.通过指针相减可以得到数组之间两个元素之间的距离差**

```c
#include<stdio.h>

int main()
{
  int array[20] = {1,23,4,78,9,2};
  int *p1 = &array[3];
  int *p2 = &array[7];
  printf("juli = %d\n",p2 -p1);
  return 0;
}


```

程序输出结果：`juli = 4`

---


**5.指针与数组在地址上的关系**
数组名等效于数组名取地址，等效于数组首元素的地址

```c
#include<stdio.h>

int main()
{

char buf[10] = {0,1,2,3,4};
char *p = buf;
char *p1 = &buf;
char *p2 = &buf[0];
//前面的这三个是等效的，都表示数组首元素的地址

char *p3 = &buf[1];
char *p4 = &buf[2];

p3 ++;   //改变了p3所指的地址，变成了p4指向的buf[2]
printf("%d,%d,%d,%d,%d\n",p,p1,p2,p3,p4);
return 0;

}
```

调试的结果为：
`343472120,343472120,343472120,343472122,343472122`

可见前面三个是等效的；后面可以通过指针的加减实现指向位置的改变，从而改变输出值的大小

**示例1：用指针来求一个字符串的长度**，不可以使用数组下标的方式

```c
#include<stdio.h>

int main()
{
  char s1[100] = "hello";
  char s2[100] = "world";
  char *p1 = s1;
  int len = 0;
  while(*p1)  //*p1的内容为0的时候，表示字符串就结束了
  {
    p1++;
    len++;
  }
  printf("len = %d\n",len);
  return 0;
}
```
~~结果有点问题：如果元素是整数，且后几位全为0.则会造成计数不准确。~~


**示例2：用指针来将s1和s2合并为一个字符串，结果放入s1中，不可以使用数组下标的方式；**
结构：先获取字符串一的长度，循环导致此时指向字符串一的指针正好指在字符串最后一个位置
然后将字符串二的指针指向之前末尾，实现合并；


```c
#include<stdio.h>

int main()
{
  char s1[100] = "hello";
  char s2[100] = "world";



  //获取字符串s1的长度
  
  char *p1 = s1;
  int len = 0;
  while(*p1)  //*p1的内容为0的时候，表示字符串就结束了
  {
    p1++;
    len++;
  }
  printf("len = %d\n",len);   



  //将s2元素连接到s1后面,这时候p1已经指向了s1的最后名的\0
  
  char *p2 = s2;
  while(*p2)
  {
    *p1 = *p2; //从s1数组最后开始，从s2的首元素开始，因为根据上面的语句，在求出s1长度的同时，p1也指向了s1的末尾位置；
    p2++;
    p1++;
  }

//以上的while语句等效为：

  /*
    while(*p2)
    {
      *p1++ = *p2++;
      //先++在取值
    }
  */


  printf("s1 = %s\n",s1);
  return 0;
}
```

----

**6.使用指针来访问数组成员**

```c
#include<stdio.h>

int main()
{
  int array[100] = { 0 };
  int *p = array;
  p[0] = 100; //这是允许的，从语法上可以和数组方式一样，就是通过下标的方式进行访问

 printf("sizeof(array) = %d\n",sizeof(array));
 printf("sizeof(p) = %d",sizeof(p));
 printf("sizeof(char *) = %d",sizeof(char *));  //char * 表示一个指向char的指针
  return 0;
} 


```
运行结果为：
`sizeof(array) = 400`
`sizeof(p) = 8`(在32位的操作系统中值为4),因为本质上是一个int型整数
`sizeof(char *) = 8`

---

**7.游戏外挂：针对于单机游戏（以植物大战僵尸为例）**

- 首先打开游戏，开始游戏之后，比如现在阳光的数量为175，这时候打开`cheat Engine`软件，在值中输入175，然后会出现所有变量值为175的变量及其地址；
- 这时候继续游戏，消耗阳光之后，比如值变为75，继续在软件中输入变量值为75，然后再次查找就会得到唯一的变量地址，即为存放阳光变量的真正地址

- 创建工程开始写程序，将变量值进行修改

```c
_declpec(dllexport)  //加上这个关键字，代表go函数是可以在其他程序中调用dll函数

void go()
{
  int *p = 0x1CF88808;  //得到阳光变量的地址，改地址可以通过以上的软件得到

  while（1）
  {
    if(*p < 100)
    {
      *p = 300;   //应当避免一次性赋值很大，防止触发系统的反作弊机制
    }
  }
}
```

- 然后将项目属性修改为动态库.dll

- 然后使用软件dllinject 将这个程序添加进游戏之后就可以了；


---





## 三、指针数组以及多级指针

**1.两种相似定义的区别**

```c

#include<stdio.h>

int main()
{
  int *s[10] = {0};  //本质上定义了一个数组，指向int*型变量
  int (*p)[10] = 0;  //本质上仅仅定义了一个指针变量，指向int  [10]这么大的一种数据类型
  
  printf("sizeof(s) = %d\n",sizeof(s));
  printf("sizeof(p) = %d\n",sizeof(p));
}

```
程序运行结果：`sizeof(s) = 80`
                       `sizeof(p) = 8`

---

==**2.指针数组**==
定义和使用
```c
#include<stdio.h>

int main()
{
  int *a[10];//定义了一个指针数组，一共10个成员，其中每个成员都是int *类型；
  printf("%d,%d\n",sizeof(a),sizeof(a[0]));

  short *b[10]; //定义了一个含有十个成员的指针数组，其中每个成员都是short *类型
  printf("%d,%d\n",sizeof(b),sizeof(b[0]));



  //具体使用方法

  int a1;
  int a2;
  a[0] = &a1;
  a[1] = &a2;

  return 0;
}
```

---

**3.指向指针的指针**

因为指针本身也为变量，也存在地址，相当于用指针指向指针的地址

```c
#include<stdio.h>
int main()
{
  int a = 10;
  int *p = &a;    //定义一个指针p存放变量a的地址
  int **pp = &p;  //定义一个二级指针，指向了一个一级指针的地址
  //这里等效于以下代码
  int **pp;
  pp = &p;

  //使用
  **pp = 100;  //通过二级指针修改内存的地址
  
  //*pp = 10;  这相当于将p指向了编号为10的这块内存，pp还是正常的指针，但是p被修改成野指针了；

  printf("a = %d\n",a);

return 0;

}
```
程序运行结果：`a = 100`

----

**4.指向多维数组的指针**
——以二维数组为例

- **定义：**
```c
#include<stdio.h>

int  main()

{

  int  buf[2][3]  =  {{1,2,3},{4,5,6}};
  int  (*p)[3];  //定义了一个指针，只有一个成员变量，指向int  [3]  这种数据类型，是一种指向二维数组的指针；

  p  =  buf;  //表示指向二维数组的第一行
  //如果想要表示数组的第二行，可以使用P++

//同理  p[0]  =  buf的首地址

  printf("%d\n",sizeof(p));  //sizeof(p)在32为的操作系统中是4，在64位操作系统中值为8
  printf("%d,%d\n",p,p  +  1);  //这里的p+1实际上平移了1  *  sizeof(int  [3])大小，即平移了24个字节；

  int  i  =  0;
  for(i  =  0;  i  <  2;  i++)
    {
      printf("%d\n",*p[i]);//*p[i] 表示i行首元素的值
   }

  return  0;

}
```
程序运行结果：`1` ,`4`


- **使用：**
功能：将二维数组中所有的元素打印出来

```c
#include<stdio.h>

int main()
{
  int buf[2][3] = {{1,2,3},{4,5,6}};
  
  int (*p)[3];   
  p = buf; 

  int i;
  int j;
  for(i = 0; i < 2; i++)
  {
    for(j = 0; j < 3; j++)
    {
      printf("%d\n",p[i][j]);
    
    }
  }


  return 0;
}
```
程序运行结果：
`1`
`2`
`3`
`4`
`5`
`6`

上面程序中`printf(“%d\n”,p[i][j]);` 等效于 `printf(“%d\n”,*(*(p+i)+j));`
注释：`(p+i)`表示每一行的首地址,等效于`p[i] `

---


**5.各种地址的表示方法**

| 表示方法                                 | 含义                             |
| ------------------------------------    | --------------------             |
| int buf[3][5]                           | 二维数组名称，buf代表数组首地址     |
| int (*a)[5]                             | 定义一个指向int[5]类型的指针变量a   |
| a[0]  ,  (a+0)  ,a                      | 表示第0行第0列元素的地址            |
| a+1                                     | 表示第一行首地址                   |
| a[1]   ,  *(a +1)                       | 表示第一行第0列元素的地址           |
| a[1] + 2   ,*(a+1)+2  ,  &a[1][2]       | 表示第一行第二列元素的地址          |
| (a[1] + 2) , (*(a+1)+2) , a[1][2]       | 表示第一行第二列元素的值            |

---

**6.练习**
目标：在不使用数组下标的情况下，只能通过指向二维数组的指针求出数组中每行和每列的平均值

~~程序不完整~~

```c
#include<stdio.h>

int  main()

{

  int  buf[3][5]  =  {{2,4,3,5,3},{7,2,6,8,1},{7,3,5,0,2}};
  int  (*p)[5];
  p  =  buf;
  
  int  i;
  int  j;
  float  _aver_hang  =  0;
  float  _aver_lie  =  0;
  float  _sum_hang  =  0;
  float  _sum_lie  =  0;
  for(i  =  0;  i  <  3;  i++)
  {
    for(j  =  0;  j  <  5;  j++)
      {

        _sum_hang  +=  p[i][j];
        _aver_hang  =  _sum_hang  /  3.0;
      }

  printf("%d\n",_aver_hang);

  }

  return  0;

}
```

---

**7.const保护函数参数以及返回值为指针的函数**

通过函数的指针参数可以间接的实现形参修改实参的值；
- **无法传值的程序**
```c
#include<stdio.h>

void test(int p)
{
  p++;  
}

int main()
{
  int i = 0;
  test(i); 
  printf("i = %d\n",i);
  return 0;

}
```

程序运行结果:`i = 0`；
因为形参的值不能反过来传给实参

- **可以传值的程序**
**想改变实参的值需要使用指针；**
```c
#include<stdio.h>

void test(int *p)
{
  (*p)++;  //*p为i的值，则对应的是i+1；

}

int main()
{
  int i = 0;
  test(&i);  //传给形参的是实参i的地址
  printf("i = %d\n",i);
  return 0;

}
```

程序结果为：`i=1`;

---

**8.交换两个数据的值：**

```c
#include<stdio.h>

void swap(int *c,int *d)
{
  int tmp = *c; //*c即是a的值
  *c = *d;
  *d = tmp;
}

int main()
{
  int a = 10;
  int b = 20;
  swap(&a,&b);  //传递的是地址
  printf("a = %d, b = %d",a,b);
}
```
程序运行结果：`a = 20, b = 10`


---



**9.将一维数组名作为函数参数**
数组名代表数组的首地址

```c
#include<stdio.h>

void set_array(int buf1[])  //当一维数组名作为函数的参数的时候，数组名其实就是一个指针变量
 //上面那行代码等效为：void set_array(int *buf1)
  {
    buf1[0] = 100;
    buf1[1] = 200;
  }


void print_array(int buf2[])
{
  int i;
  for(i = 0;i < 10; i++)
  {
    printf("buf[%d] = %d\n",i,buf2[i]);
  }

}


int main()
{
  int buf[10] = {1,2,3,4,5,6,7,8,9,10};
  set_array(buf);  //放的是数组的首地址
  print_array(buf);
  return 0;
}
```

当参数为指针的时候，应该同时定义指针的维度

```c
#include<stdio.h>

void print_array(int *buf2 , int n)
{
  int i;
  for(i = 0;i < n; i++)
  {
    printf("buf[%d] = %d\n",i,buf2[i]);
  }

}


int main()
{
  int buf[] = {1,2,3,4,5,6,7,8,9,10};
  print_array(buf,sizeof(buf)/sizeof(int)); //自动将buf的长度获取
  return 0;
}
```

---
**10.将二维数组名作为函数参数**

```c
#include<stdio.h>

void print_array(int (*p)[3],int a,int b)  //这里的维度3一般不可更改，而且实际中二维数组作为函数的参数的示例很少使用
//等效写法 void print_array(int p[][3],int a,int b)
{
   int i;
   int j;
   for(i = 0;i < a; i++)
   {
     for(j = 0; j < b; j++)
     {
       printf("p[%d][%d] = %d\n",i,j,p[i][j]);
     
     }   
   
   }
 
}

int main()
{
  int buf[2][3] = {{1,2,3},{4,5,6}}; 
 
// print_array(buf,2,3);  这是行数和列数都是写死的
 //如果想将二维数组的行数和列数改变，仍然不妨碍其他的程序进行
  print_array(buf,sizeof(buf)/sizeof(buf[0]),sizeof(buf[0])/sizeof(int));


//以下程序是常见的打印行列数的代码
printf("shuzuchangdu  =  %d\n",sizeof(buf));

printf("diyihangchangdu  =  %d\n",sizeof(buf[0]));

printf("hangshu  =  %d\n",sizeof(buf)/sizeof(buf[0]));

printf("lieshu  =  %d\n",sizeof(buf[0])/sizeof(int));
 
   return 0;
}
```

---

**11.const保护函数参数**
当部分的函数参数不需要被修改的时候，可以使用const进行保护
如果将一个数组作为函数的形参传递，那么数组的内容可以在被调用的函数内部进行修改，如果不希望被修改，可以对形参采用const参数

**示例：** 将数组s1和s2进行合并，则对于传过来的s2的值是不应该改变的

```c
#include<stdio.h>

void mystrcat(char *a, const char *b)   //第二个形参的值不希望被改变，使用const可以避免被改变

{

  int len = 0;
  while(b[len])
  {
    len++;
  }

  while(*a)
  {
    a++;
  }


//*b = 10;    这个语句会被报错
  int i;
  for(i = 0; i < len; i++)
    {
      *a = *b;
      a++;
      b++;
    }

}

int main()
{
  char s1[10] = "abc";
  char s2[10] = "def";
  mystrcat(s1,s2);
  printf("s1 = %s\n",s1);
  return 0;

}
```

---

**12.指针作为函数的返回值**

**示例：** 实现查找数组中是否含有某个字符

```c
#include<stdio.h>

char *mystrchar(char *s,char b)
{
  while(*s)
  {
    if(*s == b)
    return s;
    s++;
  }

  return NULL;
}

int main()
{
   char str[20] ="hello world";  
   char *la = mystrchar(str,'o');
   printf("str = %s\n",la);
   return 0;

}
```

---

**13.char指针与字符串以及返回值为指针的函数**
通过指针访问字符串数组

```c
#include<stdio.h>

int main()
{
  int buf[20] = "hello world";

  char *p = buf;
  //以下的代码之间是相互等效的
  *(p + 5) = 'a';
  p[5] = 'b';   //上面这两个中p 的大小没变

  p +=5;
  *p = 'c';   //p变成了5

  printf("buf = %s\n",buf);
  

}
```

数组做参数和字符串做参数的区别

```c
#include<stdio.h>

void print_array(int *p ,int n)  //如果参数为一个int数组，那么必须传递第二个参数用来表示维度
{
  int i;
  for(i = 0; i < n; i++)
  {
      printf("p[%d] = %d\n",i,p[i]);
  }
}

void print_str(char *s)
{
  int i = 0;
  while(s[i])
  {
    printf("%c",s[i++]);  //这句要记住
  }

}

int main()
{
  int a[10] = {1,2,3,4,5};
  print_array(a,10);

  char b[20] = "hello world";
  print_str(b);

  return 0;

}
```

--- 


**14.两种打印字符串的方法：**
遇`\0`是否继续打印后面的字符

- 正常情况下遇到`\0`即默认字符串到达结尾，停止打印，程序为：

```c
#include<stdio.h>

void print_str(char *p)
{
  //下面有两种写法：
  while(*p)
  {
    printf("%c",*p);
    p++;
  }

  int i = 0;
  while(s[i])
  {
    printf("%c",s[i++]);
  }


}


int main()
{
char buf[20] = "hello world";
buf[3] = 0;

print_str(buf);
return 0;

}
```

该程序的结果为：`hel`

- 如果想让该字符串仅仅按照字符串的长度来输出，不受\\0的约束

```c
#include<stdio.h>

void print_str(char *s,int len)
{
  int i = 0;
  for(i = 0; i < len; i++)
  {
    printf("%c",s[i]);
  }

}

int main()
{
  char s[20] = "hello world";
  s[3] = 0;
  print_str(s,sizeof(s));   //打印的长度是数组定义的长度
  return 0;
  
}
```

程序运行结果：hel o world

---


**15.main函数含参数**
操作系统调用主函数
![111]($resource/111.png)


## 四、指针难点
==1.void 与void *区别==

- **void**表示无类型，一般用于对函数返回的限定以及对函数参数的限定
  - 当函数不需要返回值时，必须使用void限定。例如： `void func(int a, int b);` 
  - 当函数不允许接受参数时，必须使用void限定。例如： `int func(void)`或者`int main()`。


- void * 表示无类型指针，可以指向任何数据类型,同理，可以使用任意数据类型的指针对void指针赋值； 
  - 由于void指针可以指向任意类型的数据，亦即可用任意数据类型的指针对void指针赋值，因此还可以用void指针来作为函数形参，这样函数就可以接受任意数据类型的指针作为参数。例如：
```c
void * memcpy( void *dest, const void *src, size_t len );
void * memset( void * buffer, int c, size_t num);

```
例子：
```c
int * pint;
void *pvoid;
pvoid = pint; 
//不过不能 pint= pvoid; 如果要将pvoid赋给其他类型指针，则需要强制类型转换如：pint= (int *)pvoid;

```
  - 在ANSIC标准中，不允许对void指针进行算术运算如pvoid++或pvoid+=1等，而在GNU中则允许


==2.void (*p)() 、void *p()和void *（*p）(void)的区别==

- void (*p)()是一个指向函数的指针，表示是一个指向函数入口的指地变量，该函数的返回类型是void类型。它的用法可参看下例:
p指向一个无参数返回值为void函数的首地址
例如：有一返加void值的函数swap，(swap用来交换两个数)
```c

void (*p)();    /*定义指向函数的指针变量p*/
p=swap;    /*使指针变量p指向函数max*/
(*p)(a,b);   /*通过指针变量p调用函数max*/
它等价于:
swap(a,b)
```
- void *p()是一个指针型函数，它的函数名为p，返回了一个指针，因为是void，这个指针没有定义类型，所以返回的是一个通用型指针。
给你举一个例子：
```c


#include<stdio.h>
int *max(int *p);
void main()
{
    int a[10]={96,23,45,86,79,63,58,36,29,95};
    int *p;
    p=max(a);
    printf(“max=%d\n”,*p);
}
int *max(int *p)
{
     int i,*q=p;
     for(i=1;i<10;i++)
     if(*(p+i)>*q)
          q=p+i;
     return q;
}
```

- void *（*p）(void)：函数返回值是void *指针类型。*p则是一个函数指针。括号中的void表示函数指针指向的函数参数为void。






**函数返回值为指针：**
```c
#include<stdio.h>

#include<string.h>

char  *compare(char  *str1,char  *str2){

  if(strlen(str1)>strlen(str2)){

  return  str1;

  }else  if(strlen(str1)<strlen(str2)){

  return  str2;

  }else{

  char  *r  =  "yiyangchang";

  return  r;

  }

}

void  main(){

  char  *str1  =  "123";

  char  *str2  =  "1234";

  char  *r=compare(str1,str2);

  printf("%s\n",r);

}


```

程序运行结果：`1234`
