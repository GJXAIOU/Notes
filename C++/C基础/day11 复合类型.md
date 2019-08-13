# day11 复合类型
@toc


## 一、结构体的定义和成员初始化

```c
#include<stdio.h>
#include<string.h>      //这里需要使用strcpy函数

struct student
{
  char name[100];
  int age;
  int sex;
};                   //说明了一个结构体的数据成员类型,注意这里有一个;


int main()
{


//法一：先定义然后初始化

struct student st; //定义了一个结构体的变量（在栈中），名字叫做st

//使用成员变量

st.age = 20;
st.sex = 0;
strcpy(st.name,"zhangsan");
printf("name = %s\n",st.name);



//法二：在定义的同时初始化成员变量

struct stduent st = {"zhangsan",20,1};//这个必须按照顺序依次赋值
struct student st = {.age = 20,.name = "zhangsan",.sex = 0};//此方法允许自由变换成员变量的位置进行赋值

//法三：通过用户赋值进行初始化

scanf("%s",&st.name);
scanf("%d",&st.age);
scanf("%d",&st.sex);

}
```

---

## 二、结构体成员内存对齐详解

结构体在内存之中一定是一个矩形结构

```c

#include<stdio.h>

struct neicun
{
  int a;
  char b;
  char c;
};

int main()
{
struct neicun duiqi;
printf("%d\n",sizeof(duiqi));  //按照定义中最长的那个对齐 
}
```

程序的输出结果为：`8`
![内存空间对齐1]($resource/%E5%86%85%E5%AD%98%E7%A9%BA%E9%97%B4%E5%AF%B9%E9%BD%901.png)

**结构体中定义变量顺序改变之后**
```c
#include<stdio.h>

struct neicun
{
  char a;
  int b;
  char c;
};

int main()
{
struct neicun duiqi;
printf("%d\n",sizeof(duiqi));  //按照定义中最长的那个对齐 
}

```

程序运行结果：`12`

![内存空间对齐2]($resource/%E5%86%85%E5%AD%98%E7%A9%BA%E9%97%B4%E5%AF%B9%E9%BD%902.png)

**示例**

```c
#include<stdio.h>

struct C
{
  char a;  //1个字节
  short b;//空一个字节，然后两个字节
  char c;//一个字节
  int d;//另起一行占四个字节
  long long e;//另起一行占8个字节

}

int main()
{
printf("%d\n",sizeof(struct C));
}

```

程序运行结果为：`24`

**示例**
==**偶数位对齐**==
```c
#include<stdio.h>

struct neicun
{
  char a;
  short b;
  int c;
  short d;
};

int main()
{
struct neicun duiqi;
printf("%d\n",sizeof(duiqi));  //按照定义中最长的那个对齐 
}

```
程序的运行结果：`12`

![内存空间3]($resource/%E5%86%85%E5%AD%98%E7%A9%BA%E9%97%B43.png)

**实例二**
```c
#include<stdio.h>

struct neicun
{
  char array[10];
  char b;
  char c;
  char d;
  char e;
  
};

int main()
{
  struct neicun duiqi;
  printf("%d\n",sizeof(duiqi));  //按照定义中最长的那个对齐 
}
```
程序输出结果为：`14`  ,不会将整个数组当成一个整体，只是10个char型的数据，所以最大的长度仍然为char

**示例三**
```c

#include<stdio.h>

struct neicun
{
  int array[10];
  char b;
  char c;
  char d;
  char e;
  char f;
  
};

int main()
{
  struct neicun duiqi;
  printf("%d\n",sizeof(duiqi));  //按照定义中最长的那个对齐 
}

```
程序输出结果为：`48`
因为数组中为10个int型数据，所以整个结构体中最大的字节长度为int,所以全部按照int 对齐；


---

## 三、结构体成员的位字段
一般用于嵌入式系统中的状态控制

**例子程序**
```c

#include<stdio.h>

struct  D
{
  char  a  :  2;  //定义一个成员类型为char  ,但是这个成员只使用2个bit
  unsigned  char  b  :  4;  
  //a和b在内存中是连续的地址
};

int  main()
{
  struct  D  lizi;
  printf("sizeof(lizi)  =  %d\n",sizeof(lizi));
  lizi.a  =  3;
  printf("a  =  %d\n",lizi.a);
  lizi.b  =  15;
  printf("b  =  %d\n",lizi.b);
  lizi.b  =  16;
  printf("b xiugaihou  =  %d\n",lizi.b);

  return  0;
}

```
程序的输出结果为：
`sizeof(lizi) = 1`//因为内存中最小为一个字节，所以变量lizi还是占一个字节(8位)
`a = -1`//以为2用二进制表示为11，而第一个1代表符号，因此结果为-1
`b = 15`//四位能表示15
`b xiugaihou = 0`//因为16表示为 1 0000 ，可是定义的时候只给b四个字节，因此只能显示0000，结果即为0；

**利用空闲的字节**

```c

#include<stdio.h>

struct A
{
  char a;
  int b;
};

int main()
{
  struct A kongxian = {1,2};
  char *p = &kongxian;
  p++;
  *p = 3;
  p++;
  *p = 4; 
  printf("%d\n",&kongxian);
  return 0;
}
```
可以通过程序运行输出的地址查找变量的位置，通过位置查找出内存中变量的具体值，可以看出3和4是在1和2的后面的；

---


## 四、结构体数组的定义和使用

**例子程序**
```c
#include<stdio.h>

struct  B
{
  char  name[100];
  unsigned  char  age;//使用无符号的字符类型就行，范围为0-255，完全可以满足数据要求
  unsigned  char  score;
};

int  main()
{
  struct  B  jiegoutishuzu[5]  =  {{"zhangsan",12,90},{"lisi",34,98},{"wangwu",23,94},{"zhaoliu",34,99},{"zhouqi",98,100}};  //定义的时候直接赋值

  int  i;
  for(i  =  0;i  <  5;i++)
  {
  printf("xingming  =  %s,nianling  =  %d,fengshu  =  %d\n",jiegoutishuzu[i].name,jiegoutishuzu[i].age,jiegoutishuzu[i].score);
  }

  return  0;
}
```

程序输出结果：
`xingming = zhangsan,nianling = 12,fengshu = 90`
`xingming = lisi,nianling = 34,fengshu = 98`
`xingming = wangwu,nianling = 23,fengshu = 94`
`xingming = zhaoliu,nianling = 34,fengshu = 99`
`xingming = zhouqi,nianling = 98,fengshu = 100`

**和数组相似的用法**
1.交换结构体中定义数组的元素
```c
#include<stdio.h>

struct  B
{
  char  name[100];
  unsigned  char  age;
  unsigned  char  score;
};

int  main()
{
  struct  B  jiegoutishuzu[5]  =  {{"zhangsan",12,90},{"lisi",34,98},{"wangwu",23,94},{"zhaoliu",34,99},{"zhouqi",98,100}}; 

//交换结构体中元素的顺序

  struct B tmp = jiegoutishuzu[0];
  jiegoutishuzu[0] = jiegoutishuzu[1];
  jiegoutishuzu[1] = tmp;
  
  int  i;
  for(i  =  0;i  <  5;i++)
  {
  printf("xingming  =  %s,nianling  =  %d,fengshu  =  %d\n",jiegoutishuzu[i].name,jiegoutishuzu[i].age,jiegoutishuzu[i].score);
  }

  return  0;
}


```
程序运行结果为：
`xingming  =  lisi,nianling  =  34,fengshu  =  98`
`xingming  =  zhangsan,nianling  =  12,fengshu  =  90`
`xingming  =  wangwu,nianling  =  23,fengshu  =  94`
`xingming  =  zhaoliu,nianling  =  34,fengshu  =  99`
`xingming  =  zhouqi,nianling  =  98,fengshu  =  100`


2.通过结构实现数组的功能

```c
#include<stdio.h>

struct  B
{
  char  array[100];
 
};

int  main()
{
  struct  B  a1 ={"hello"};//如果结构体的成员为数组的，通过结构可以变现的实现数组的赋值  
  struct  B  a2 ={0};
  a2 = a1;
  printf("a2.array = %s\n",a2.array);
  return  0;
}
```
程序运行的结果为：`a2.array = hello`


3.将上面第一个程序（例子程序）中的数组元素按照年龄进行排序，如果年龄相同则按照分数排序

```c
/*
将结构体成员st中按照年龄排序，年龄相同安排分数排序
*/

#include<stdio.h>
struct  B
{
  char  name[20];
  unsigned  char  age;
  unsigned  char  score;
};


void  swap(struct  B  *a,  struct  B  *b)
{
  struct  B  tmp  =  *a;
  *a  =  *b;
  *b  =  tmp;
}



int  main()
{
  struct  B  st[5]  =  {  {"zhangsan",12,90},{"lisi",34,98},{"wangwu",23,94},{"zhaoliu",34,99},{"zhouqi",98,100}  };

  int  i;
  int  j;
  for  (i  =  0;  i  <  5;  i++)
  {
    for  (j  =  1;  j  <  5  -  i;  j++)
      {
        if  (st[j].age  <  st[j  -  1].age)
          {
            swap(&st[j],  &st[j  -  1]);
          }
        else  if  (st[j].age  ==  st[j - 1].age)
          {
            if  (st[j].score  <  st[j - 1].score)
              {
                swap(&st[j],  &st[j  - 1]);
              }
          }
      }
  }

  int  k;
  for(k  =  0;  k  <  5;  k++)
    {
      printf("st[%d].name  =  %s,st[%d].age  =  %d,st[%d].score  =  %d\n",  k,  st[k].name,  k,  st[k].age,  k,  st[k].score);
    }

  return  0;

}
```


---

## 五、结构嵌套以及结构和指针

### 1.结构的嵌套

```c
#include<stdio.h>
#include<stdlib.h>

struct  A
{
  int  a;
  char  b;
};

struct  B
{
  struct  A  a;
  char  b;
};

struct  C
{
  struct  A  a;
};

int  main()
{
  struct  B  a2;
  a2.a.a  =  200;  //访问A结构中的a
  
  struct  C  *ap  =  malloc(sizeof(struct  C));//这个结构体成员是在堆里面
  ap->a.a  =  20;

  printf("ap->a.a  =  %d\n",ap->a.a);
  printf("sizeof(struct  B)  =  %d\n",sizeof(struct  B));

  free(ap);

  return  0;
}
```

 程序运行结果：
 `ap->a.a = 20`
`sizeof(struct B) = 12`
 
![结构嵌套]($resource/%E7%BB%93%E6%9E%84%E5%B5%8C%E5%A5%97.png)


**结构体之间的赋值**

```c
#include<stdio.h>

struct A 
{
  int a;
  char b;
};

int main()
{
 struct A a1,a2;
  a1.a = 1;
  a1.b = 2;

  a2 = a1;//结构体赋值，其实就是结构体之间内存的拷贝
  //等效于：memcpy(&a2,&a1,sizeof(a1));//目标地址，原地址，拷贝长度
  return 0;
}
```

**指向结构体的指针**
```c
#include<stdio.h>

struct A 
{
  int a;
  char b;
};

int main()
{
 struct A a1;
 struct A *p = &a1;

//对指针进行赋值
//方法一：
//(*p).a = 10;
//(*p).b = 20;

//方法二：
p->a = 10;
p->b = 20; 

  return 0;
}
```



### 结构体中的数组成员和指针成员

**1.结构体中的数组成员**
```c
#include<stdio.h>
#include<string.h>

struct student
{
  char name[100];
  int age;
};

int main()
{
  struct student st;
  st.age = 10;
  strcpy(st.name,"zhangsan");
  printf("%d,%s\n",st.age,st.name);

  return 0;
}
```

**2.结构体中的指针成员**
此程序只能在VS中跑，在Qt中一直报错
```c
#include<stdio.h>
#include<string.h>
#pragma warning(disable : 4996)

struct student_a
{ 
  char *name;
  int age;
};


int main()
{
  struct student_a st = {NULL,0};//必须赋初值
  st.age = 10;
  st.name = malloc(100);//因为内容为空值，所以没有空间，需要单独划分空间
  strcpy(st.name,"zhangsan");
  printf("st.age = %d,st,name%s\n",st.age,st.name);

  free(st.name);

  return 0;
}
```
程序允许结果：`st.age = 10,st.name = zhangsan`


##  六、结构作为函数的参数

**例子程序1**

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

struct student 
{
  char name[20];
  int age;
};


void printf_student(struct student a)
{
  printf("name = %s,age = %d\n",a.name,a.age);
}

int main()
{
  struct student st = {"tom",20};
  printf_student(st);//调用函数
  return 0;
}
```

程序运行结果：`name = tom,age = 20`

**例子程序2**

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

struct student 
{
  char name[20];
  int age;
};


void set_student(struct student *a,const char *name, int age)  
{
  strcpy(a->name,name);
  a->age = age;
}


int main()
{
  struct student st;
  set_student(&st,"mike",100);  //只有传递地址才能将形参的变化反映到实参这边
  printf("name = %s,age = %d\n",st.name,st.age);
  return 0;
}
```
程序运行结果：`name = mike,age = 100`

==尽量不要直接使用结构作为函数参数，如果使用尽量采用指针的形式==
将以上程序融合修改为：

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

struct student 
{
  char name[20];
  int age;
};


void printf_student(const struct student *b)  //加入const之后就不会改变它的值，需要改变值时候去掉const
{
  printf("name = %s,age = %d\n",b->name,b->age);
}


void set_student(struct student *a,const char *name, int age)  
{
  strcpy(a->name,name);
  a->age = age;
}


int main()
{
  struct student st;
  set_student(&st,"mike",100);  //只有传递地址才能将形参的变化反映到实参这边
  printf_student(&st);
  return 0;
}
```
程序运行结果：`name = mike,age = 100`








