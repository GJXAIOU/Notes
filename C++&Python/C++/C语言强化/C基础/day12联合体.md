---
date:`2018-10-06-2018-10-08`
---
# day12 联合体

联合体是一个能在同一个存储空间存储不同类型数据的类型


## 一、联合体的定义与使用
- 联合体中定义的各个成员的内存地址完全一致；
- 联合体的长度等于成员中最长的一个的长度；
- 联合体虽然可以有多个成员，但是同一时间只能存放其中一种；
```c
#include<stdio.h>

union student//关键字为：union
{
  int age;
  char name;
  short score;
};

int main()
{
  union student st;//新建联合体变量st
  st.age = 18;
  st.name =liuda;

  printf("The length of the union = %d\n",sizeof(union student));
  printf("The address of each element is = %p,%p,%p\n",&st.age,&st.name,&st.score);

  return 0;
}
```
程序运行结果：
`The length of the union = 4`
`The address of each element is = 00000045ED11FB90,00000045ED11FB90,00000045ED11FB90`

---
## 二、联合体的指针成员
**1.错误程序**

//如果联合体中有指针成员，那么一定要使用完这个指针，并且free指针之后才能使用其他成员,因为联合体同一时刻只能存放一种成员。
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

union student 
{
  int age;
  char *name;

};

int main(void)
{
  union student st;
  st.name = malloc(100);//name 指向了一个堆的地址
  st.age = 10;//因为age和name公用一块内存地址，所以这时候name的值也指向了10；
  printf("%s\n",*st.name);
  free(st.name);//这时候name已经不指向name的地址，因此也无法释放了；
}
```


---

**2.正确程序**

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

union  student
{
  int  age;
  char  *name;
};


int  main(void)
{
  union  student  st;

  //定义指针

  st.name  =  malloc(100);//name指向了一个堆的地址

  strcpy(st.name,"hello");  //赋值方式

  //使用指针

  printf("st.name = %s\n",st.name);

  //释放堆

  free(st.name);//这时候name已经不指向name的地址，因此也无法释放了；

  //使用其他成员

  st.age  =  10;//因为age和name公用一块内存地址，所以这时候name的值也指向了10；

  return  0;

}
```
程序运行结果为：`st.name = hello`

---

## 三、枚举与typedef
- **枚举**
**1.枚举定义**
```c
#include<stdio.h>

//里面的成员均为int型常量，默认值分别为0,1,2,3.。。。。

/*相当于以下的定义：
* #define red 0;
* #define black 1;
*.....
*/
enum A
{
  red,black,yellow,green
};


int main(void)
{
  int color = yellow;
  printf("color = %d\n",color);
  return 0;
}
```
程序运行结果为：`color = 2`

---

**2.改变默认常量的值**
```c
#include<stdio.h>

//里面的成员均为int型常量，可以自定义成员的值

enum A
{
  red = 5,black= 2,yellow,green
};


int main(void)
{
  int color = yellow;
  printf("color = %d\n",color);
  return 0;
}
```
程序运行的结果为：`color = 3`
注：没有赋值的成员会根据前一个成员的值自动往后续值

---

- **typedef**

**1.定义与使用**
```c
#include<stdio.h>

typedef char BYTE;//定义了一个新的数据类型，名字叫做BYTE，类型为char

int main(void)
{
  BYTE a = 19;
  printf("%d\n",a);
  return 0;
  
}
```
程序运行结果为：`19`

---

**2.和define的区别**
define做的只是简单地替换，不是创造了新的数据类型
```c
#include<stdio.h>

#define BYTE char

int main(void)
{
  BYTE a = 19;
  printf("%d\n",a);
  return 0;  
}
```
程序运行结果：`19`

**总结：**
- 1.#define定义表达式或者具体的值
- 2.typedef定义仅限于数据类型

---

**3.应用在结构体中**

```c
#include<stdio.h>

struct abc
{
  int a;
  char b;
};

typedef struct abc BYTE;


//将以上代码简化为：（abc同时可以删除）
//typedef  struct  abc
//  {
//    int  a;
//    char  b;
//  }  BYTE；


int main(void)
{
  BYTE st;
  st.a = 23;
  printf("st.a = %d\n",st.a);
  return 0;
  
}



```
程序输出结果为：`st.a = 23`

----

**4.应用在指针中(指向函数的指针)**

- 未作修改的源代码
```c
#include<stdio.h>
#include<string.h>

char *mystrcat(char *s1,char *s2)
{
  strcat(s1,s2);
  return s1;

}

char *test(char *(*p)(char *,char *),char *s3,char *s4)
{
  return p(s3,s4);
}


int main(void)
{
  char s1[100] = "hello";
  char s2[100] = "world";
  char *s = test(mystrcat,s1,s2);
  printf("s = %s\n",s);
  return 0;

}
```

- 使用typeof简化程序

```c
#include<stdio.h>
#include<string.h>

char *mystrcat(char *s1,char *s2)
{
  strcat(s1,s2);
  return s1;

}

typedef char *(*STRCAT)(char *,char *);


char *test(STRCAT p ,char *s3,char *s4)
{
  return p(s3,s4);
}


int main(void)
{
  char s1[100] = "hello";
  char s2[100] = "world";
  char *s = test(mystrcat,s1,s2);
  printf("s = %s\n",s);
  return 0;

}
```
程序运行结果：`s = helloworld`

**应用在指针中**

```c
#include<stdio.h>
#include<string.h>

char *mystrcat(char *s1,char *s2)
{
  strcat(s1,s2);
  return s1;

}

typedef char *(*STRCAT)(char *,char *);


char *test(STRCAT p ,char *s3,char *s4)
{
  return p(s3,s4);
}


int main(void)
{

  STRCAT array[10];//定义一个包含是个成员的数组，每个都是指向char *mystrcat(char *s1,char *s2) 这种数据类型函数的指针
 
   //上面语句等效为
  // char *(*p[10])(char *s1,char *s2); 
  
  char s1[100] = "hello";
  char s2[100] = "world";
  char *s = test(mystrcat,s1,s2);
  printf("s = %s\n",s);
  return 0;

}
```






 




