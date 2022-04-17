I’m the king of myself.
# day8 函数整体

@toc

## 字符串函数



## 整数转换为字符串

```c
#include<stdio.h>

void  zhuanhuan(int  a,char  s[])

{

  int  i  =  0;

//将整数的每一位都取出并且放入字符串之中

  while(a)
  {
    int  c  =  a  %  10;  //取出整数的个位
    char  b  =  c  +  '0';  //  ==将整数转换为字符==
    s[i]  =  b;//将转换后的char一次放入字符串s中
    i++;
    a/=  10;//相当于每次将最后的一位去掉
  }


//因为放入字符串的字符顺序与之前的整数数字的顺序相反，因此将整个数组的前后部分互换位置；

  int  min  =  0;
  int  max  =  i-1;
  
  while(min  <  max)
  {
  char  tmp  =  s[min];
  s[min]  =  s[max];
  s[max]  =  tmp;
  min  ++;
  max  --;
  }

}


int  main()
{
  int  i  =200;
  char  s[100]  =  {0};
  zhuanhuan(i,s);
  printf("%s\n",s);
}
```




## C语言多文件编译

- #include 与#define的意义
  - #include就是简单的文件内容替换
  - #define就是简单的文本替换而已

- #ifndef 与#endif
   #ifndef 的意义就是条件预编译，如果#ifndef后面的条件成立，那么就预编译#ifndef开始到#endif之间的代码，否则不会去预编译这段代码； 



==实现多文件编译：==

通过自定义头文件`#include "a.h"` 实现多文件编译，冒号表示为自定义的头文件

在编写宏定义`a.h`的时候，规范如下：
（含义：如果没有_GJX_A_这个宏，那么就编译#endif之间的这些代码，如果有的话就不执行）
```c
#ifndef _GJX_A_     //文件名定义的要几乎确保唯一
#define _GJX_A_     //具体宏的名字是自定义的

//具体的函数声明
int max(int a,int b);
int add(int a,int b);

#endif

//防止源文件中多次include的同一个头文件的时候，重复预编译头文件内容

```


## 函数递归分析
函数的递归分析：本质上就是函数调用自身，不是简单的循环，必须有条件，否则为死循环；

先序递归和后续递归的结果是逆置的；
```c
#include<stdio.h>

void test(int n)
{

  if(n > 0)
  {  
    n--;
    printf("n = %d\n",n); //先序递归(函数语句在再次调用命令之前)，如果是先序递归则程序是顺序执行的；2 1 0
    test(n);  //函数递归
    printf("n= %d\n",n);  //后续递归，那么代码是逆序执行 0 1 2   
  }

}


int main()
{
  int i = 3;
  test(i);
  return 0;
}

```


## 函数复习

实现strcat的功能，将字符串str1和str2连接起来
```c
#include<stdio.h>
#include<stdlib.h>

void mystrcat(char s1[],char s2[])
{
  int len1 = 0;
  while(s1[len1++]);
  len1--;

  int len2 = 0;
  while(s2[len2++]);
  len2--;

  int i;
  for(i = 0; i < len2; i++)
  {
    s1[len1 + i] = s2[i];  
  }

}



int main()
{
  char str1[20] = "hello";
  char str2[20] = "world";
  mystrcat(str1,str2);
  printf("%s\n",str1);


}
```
程序运行的结果为：`helloworld`



