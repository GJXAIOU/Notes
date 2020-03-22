---
date:`2018-12-19`复习 161行
---

# day 7 函数



**1.定义构造函数**
示例：使用系统自带的strlen函数求解数组的长度
```c
#include<stdio.h>
#include<string.h>

int main()
{
  char s[] = "hello world";
  int len = strlen(s);
  printf("%d\n",len);

}

```

使用自己构造的函数求解数组长度

```c
#include<stdio.h>
int main()
{
  char s[] = "hello world";
  int len = 0;
  while(s[len++]);
  len--;

printf("len = %d\n",len);

}

```
将以上的程序转换为函数；

```c
#include<stdio.h>

int mystrlen(char a[])   //函数返回值类型   函数名（参数类型  形式参数）
{
  int len = 0;
  while(a[len++]);
  len --;
  return len;    //函数的返回值一定要和函数定义的时候一致

}

int main()
{
  char s[] ="hello world";
  int len = mystrlen(s); //实参可以是变量或者是常量，而形参只能是变量；
  printf("len = %d\n",len);


}
```

当自定义的函数在main函数之后时候，前面要加函数声明
```c

#include<stdio.h>

int mystrlen(char a[])   //函数声明



int main()
{
  char s[] ="hello world";
  int len = mystrlen(s); //实参可以是变量或者是常量，而形参只能是变量；
  printf("len = %d\n",len);
}



int mystrlen(char a[])   //函数返回值类型   函数名（参数类型  形式参数）
{
  int len = 0;
  while(a[len++]);
  len --;
  return len;    //函数的返回值一定要和函数定义的时候一致
}


```


求a.b中较大的一个数

```c
#include<stdio.h>

int max(int a,int b)
{
  int c = (a < b)? a : b; 
}
```

**形参与实参传递**
C语言中实参到形参是“值传递”，是一种单向传递，只能由实参传递给形参，但是当函数的参数为数组的时候，可以通过形参修改实参的值



**实现strcpy功能的函数**

```c

#include<stdio.h>

char  mystrcat(char  a[],  char  b[])
int  mystrlen(char  c[])
int  main()
{

char  str1[]  =  "hello  ";
char  str2[]  =  "world";
mystrcat(str1,str2);

printf("%s\n",str1);

return  0;
}

void  mystrcpy(char  a[],char  b[])
{

  int  len1  =  mystrlen(str1);
  int  len2  =  mystrlen(str2);

  int  i  =  0;
for(i  =  1;  i  <  len2  +  1;  i++)
  {
  a[len1  +  i]  =  b[i-1];

  }

}

int  mystrlen(char  c[])

{

  int  len  =  0;

  while(c[len++]);

  len  --;

  return  len;

}
```

**随机数产生函数rand和srand**
**rand 是伪随机数产生器，每次调用rand产生的随机数是一样的；**
如果在调用rand之前先调用srand就出现任意的随机数，而且要保证每次调用srand函数的时候，参数的值是不同的，那么rand函数就一定会产生不同的随机数

- 仅有 rand 的时候
```c
#include<stdio.h>
#include<stdlib.h>


int main()
{

  int i = 0;
  for(i = 0; i < 10; i++)
  {
    int value = rand();
    printf("%d\n",value);
  
  }
  return 0;
}


```

- 同时使用`rand`和`srand`的时候
因为要求`srand`的参数时刻变化，一般选择将系统时间转换为整数赋值作为参数

```c
#include<stdio.h>
#include<time.h>
#include<stdlib.h>

int main()
{
  time_t tm = time(NULL);   //得到系统时间
  srand(tm);    //随机数种子发生器
  int i;
  for(i = 0; i < 10; i++)
    {
      int value = rand();
      printf("%d\n",value);
    }
  return 0;
}

```


**字符串数组的输入与输出**

- 输入函数：scanf  、gets 、fgets

 **scanf :**  可以使用转义输出，既可以输出整数或者字符串，存在缓冲区溢出的问题；遇到空格和回车均认为输入结束；
  **gets ：** 只能输入字符串，存在缓冲区溢出的问题；只有遇到回车的时候才认为输出结束；
  **fgets ：** 通过设置第二个参数的大小可以防止缓冲区溢出的出现；

  - scanf
```c
#include<stdio.h>

int  main()

{

  char  s1[20]  =  {0};

  scanf("%s",s1);

  printf("%s\n",s1);

  return  0;

}

```

  - gets 
     
```c
#include<stdio.h>

int main()
{
  char ss[20] = {0};
  gets(ss);
  printf("%s\n",ss);

return 0;
}
```

局限性：gets仅能使用获得字符串的值，但是一般使用过程中需要使用整数，因此需要借助其他函数将其转换为整数   `atio` 函数将字符串转换为对应的整数值
```c
#include<stdio.h>
#include<stdlib.h>

int  main()
{

  char  ss[20]  =  {0};
  gets(ss);
  int  sss;
  sss  =  atoi(ss);
  printf("%d\n",sss);

return  0;

}
```

  - fgets 
  fgets是安全的，不存在缓冲区溢出的问题，只要第二个参数值的大小小于数组的长度
```c
#include<stdio.h>
#include<stdlib.h>

int  main()
{

  char  s[20]  =  {0};

  fgets(s,(sizeof(s)  -  1),stdin);
  //参数名称：char型数组名，存储长度（小于数组的长度，单位为：字节），stdin表示标准输入输出

  printf("%s\n",s);

  return  0;

}

```

**字符串操作函数**
 sizeof 返回的只是数组的大小，而strlen返回的是字符串的有效元素的长度（不包括\0）

- strlen:
```c
#include<stdio.h>
#include<string.h>

int main()
{
  char s[100] = "hello world";
  int len = strlen(s);    
  printf("len = %d\n",len);

  return 0;
}

```
- strcat:

```c
#include<stdio.h>

#include<string.h>

int  main()

{

  char  s[100]  =  "hello  world";
  char  s1[100]  =  "nihao";
  strcat(s,s1);
  //两个字符串合并，结果放入第一个参数之中，但是如果第一个字符串的大小无法容纳两个字符串的实际元素大小，也会产生溢出问题。

  printf("%s\n",s);
  return  0;

}
```

- strncat 

```c
#include<stdio.h>
#include<string.h>

int  main()

{

  char  s[100]  =  "hello  world";
  char  s1[100]  =  "nihao";
  strncat(s,s1,2);//只复制前两个字符；

  printf("%s\n",s);
  return  0;

}

```

- strcmp

```c
#include<stdio.h>

#include<string.h>

int  main()
{

  char  s1[]  =  "hello";
  char  s2[]  =  "hello  world";
  if(strcmp(s1,s2)  ==  0)  //返回值为0表示两个字符串相同
  {
    printf("相同\n");
  }

  else
  {
    printf("不相同\n");
  }

}
```


- strncmp  
只比较前n个字符

```c
#include<stdio.h>
#include<string.h>
int main()
{
  char s1[] = "hello";
  char s2[] = "hello world";
  if(strcmp(s1,s2,5) == 0)
    {
      printf("相同\n");
    }
  else
    {
       printf("不相同\n"); 
    }

  return 0;
}
```

- strcpy 
拷贝

```c
#include<stdio.h>
#include<string.h>
int main()
{
  char s1[] = "abcdefg";
  char s2[] = "12345";
  strcpy(s1,s2);
  printf("%s\n",s1);
  
  return 0;
}
```

- strncpy 

```c
#include<stdio.h>
#include<string.h>
int main()
{
  char s1[] = "abcdefg";
  char s2[] = "1234567";
  strncpy(s1,s2,5);
  printf("%s\n",s1);
  
  return 0;
}
```

- sprintf
将格式化后的字符串输出到第一个参数指定的字符串中去，而不是直接的输出到屏幕上

```c
#include<stdio.h>
#include<string.h>

int main()
{
  int i = 100;
  char s[100] = {0};
  sprintf(s,"i = %d", i);
  printf("%s\n",s);

  return 0;
}
```

应用：将整数转换为字符串
```c
int main()
{
  int i = 100;
  char s[100] = {0};
  sprintf(s,"%d", i);  //将i 的值放在字符数组s 中
  printf("%s\n",s);

  return 0;
}
```

- sscanf
sscanf是从字符串中得到一个输入值
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

int  main()
{

  char  s[100]  =  "5+6=";
  int  a;
  int  b;
  sscanf(s,"%d+%d",&a,&b); //相当于将字符转换为整数
  printf("%d\n",a+b);

  return  0;

}
```

- strchr
返回值是指针，查找数组中的元素，会返回数组中查到到的该元素之后的所有元素值；
```c
#include<stdio.h>
#include<string.h>

int  main()
{

  char  s[100]  = {0};
  strcpy(s,"hello world");  //相当于为数组s赋值
  const char *buf = strchr(s,'o'); //在s这个字符串之间，查找第二个参数指定的字符
  printf("%s",buf);

  return  0;

}

```

- strstr
查找该字符串中是否含有某些连续的字符元素
```c
#include<stdio.h>
#include<string.h>

int  main()
{

  char  s[100]  = {0};
  strcpy(s,"hello world");  //相当于为数组s赋值
  const char *buf = strstr(s,"wo"); //在s这个字符串之间，查找第二个参数指定的字符
  printf("%s",buf);

  return  0;

}
```

- strtok 
可以根据特定的字符进行分割字符串
```c
#include<stdio.h>
#include<string.h>

int  main()
{

  char  s[100]  = {0};
  strcpy(s,"hello_world_ni_hao");  //相当于为数组s赋值
  const char *buf;
  buf = strtok(s,"_"); //strtok第一次调用的时候，第一个参数是字符串，但是第二次调用的时候第一个参数应该是NULL；
  while(buf)
  {
    printf("%s\n",buf);
    buf = strtok(NULL,"_");
  
  }

  printf("%s",buf);

  return  0;

}
```

- atoi
将字符串转换为整数（int），同样的函数还有：atof ,atol

```c
#include<stdio.h>
#include<string.h>

int  main()
{

  char  s[100]  = "200";
  int i = atoi(s);
  printf("%d",i);

  return  0;

}
```


























