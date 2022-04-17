---
date:`2018-12-19`复习
---


# day6字符串与字符数组

- **基础知识的比较**
  - **'\0'与ASCII中 space 的区别：**
  ASCII中0~31为“非打印控制字符”，其中例如 0号对应的’\0‘ ，13号对应的回车键，将它们按字符的形式打印到屏幕上的时候均为“ ”，它和空格 “ ” 是有本质上的区别。**空格**对应的ASCII为32号，**它属于“打印字符”**，只不过空格在屏幕上的表现形式也为 “ ” ，与无法打印的非打印控制字符显示的效果一样罢了。**’\0‘仅仅作为一个字符串结束标志存在着，它的作用就是一个标志**！


**1.字符数组的定义**

```c
#include<stdio.h>
int main()
{
 //初始化的方法
   char array1[100] = {'a','b','c','d'};//法一
   char array2[100] = "abcd";  //法二
   char array3[] = "abcd";//数组根据后面的元素个数自动分配空间

  printf("%s\n",array);  //输出数组值
  printf("%d\n",sizeof(array2));//得到的值为100
  printf("%d\n",sizeof(array3));//得到的值为5
   
  return 0；
}
```

**2.1将一个字符串里面的元素进行排序**
（使用sizeof函数，当且仅当定义数组的大小正好包含完整元素的时候才可以）
```c

#include<stdio.h>
#include"stdlib.h"

int  main()
{
	int  i;
	int  j;
	char  array[5] = "acdb";

	for (i = 0; i < (sizeof(array) - 1); i++)
	{
		for (j = 1; j < ((sizeof(array) - 1) - i); j++)
		{
			char  tmp;

			if (array[j - 1] > array[j])
			{
				tmp = array[j];
				array[j] = array[j - 1];
				array[j - 1] = tmp;
			}
		}
	}

	printf("%s\n", array);

	return  0;

}

```

**2.2  法二：根据len的大小，这时候可以任意更改数组大小**
```c
#include<stdio.h>
#include"stdlib.h"

int  main()
{
	int  i;
	int  j;
	char  array[15] = "acdb";

	//通过这种方法得到数组长度
	int  len = 0;
	while (array[len++]);
	len--;  

	//排序算法
	for (i = 0; i < len; i++)
	{
		for (j = 1; j < (len - i); j++)
		{
			char  tmp;

			if (array[j - 1] > array[j])
			{
				tmp = array[j];
				array[j] = array[j - 1];
				array[j - 1] = tmp;

			}
		}
	}

	printf("(%s)\n", array);

	system("pause");
	return  0;

}
```

**2.3 sizeof和len的区别**

==根据以下的程序可以得到：sizeof得到的是数组定义的长度，而len得到的是数组中元素的个数(当是汉字的时候，就要取决于编码来看)==

```c
#include<stdio.h>
int  main()
{
  char  s[10]  =  "1234";
  int  len;
  while(s[len++]);   //字符串是以0结尾，即遇到0即视为结尾
  len  --;

  /*这里的while语句等效为：
  *  while（str[i]）
  *  {
  *    i++;
  *  }
  *   i--；
  */

  printf("sizeof  =  %d\n",sizeof(s));
  printf("len  =  %d\n",len);
  return  0;

}
```

**3.1 将字符串进行逆置**
1.直接进行逆置,即将后面的0也逆置到最前面（一般不会输出结果因为后面很多0）
```c
#include<stdio.h>
int  main()
{
  char  str[15]  =  "hello  world";
  int  min  =  0;
  int  max  =  14;
  while(min  <  max)
  {
  char  tmp;
  tmp  =  str[max];
  str[max]  =  str[min];
  str[min]  =  tmp;
  min++;
  max--;
  }

  printf("%s\n",str);
  return  0;
}
```

2.如果仅将“hello world”，这几个字符串进行倒置，就要先计算数组长度；
```c
#include<stdio.h>
int  main()
{
  char  str[15]  =  "hello  world";
  
  //求数组的长度
  int len  =  0;
  while(str[len++])；
  len --;

  int  min  =  0;
  int  max  =  len-1;
  while(min  <  max)
  {
    char  tmp;
    tmp  =  str[max];  
    str[max]  =  str[min];  
    str[min]  =  tmp;  
    min++;  
    max--;

  }
  printf("%s\n",str);
  return  0;

}
```

**3.2如何将汉字进行倒置**
在ASCII码中1个字节存放一个字符；
在GBK编码中2个字节存放一个汉字；倒置的时候需要两个字节共同互换位置，因此需要两个中间变量
在UTF-8编码中3个字节存放一个汉字；

以下的示例是以GBK编码为例；(Qt 中默认是UTF-8编码)

(工具->选项 -> 文本编辑器  -> 行为 ->文件编码：将UTF-8的编码更改为GBK编码)
```c
#include<stdio.h>

int  main()
{
  char  str[50]  =  "你好世界";

  int  len  =  0;
  while(str[len++]);
  len--;


  int  min  =  0;
  int  max  = len-1;
  while(min  <  max)
  {
    char  tmp;
    tmp  =  str[max  -1];
    str[max  -1]  =  str[min];
    str[min]  =  tmp;
    
    char  tmp1;
    tmp1  =  str[max];
    str[max]  =  str[min  +  1];
    str[min  +  1]  =  tmp1;
    min  +=  2;
    max  -=2;
  }
  printf("%s\n",str);
  return  0;

}
```


**4.字符串和字符数组的差别**

- 定义及其使用上的区别
  - 字符串：`char *string = “andnf”;`
  - 字符数组：`char a[100] = "andnf";`或者`char a[100] = {a,n,d,n,f};`
==使用字符串初始化字符数组的时候要比逐个赋值多占一个字节，用于存放 \0 ,但是字符数组的长度不会计算\0，而数组占有的字节数会计算\0 ==，
     - 输入输出：输出的时候只要遇到‘\0’就会停止输出；
     - 定义：定义字符数组的时候，如果不赋值则必须规定数组的大小；但是如果初始化了，就可以选择规定大小或者不规定；如果不规定则系统会根据数组的大小进行自动分配；

```c
int main()
{
  char s[] = "abcd";
  char s1[] = {'a','b','c','d'};
  printf("%d\n",sizeof(s));
  printf("%d\n",sizeof(s1));
  return 0;
}
```
==得到的结果分别是5和4.==

字符串在内存中是以0结尾；以下程序可以正常的输出字符串“abcd”，因为s[4]的值为0，所以这是一个字符串；
```c
#include<stdio.h>
int main()
{
  char s[5] = {0};
  s[0] = 'a';
  s[1] = 'b';
  s[2] = 'c';
  s[3] = 'd';
  printf("%s\n",s);
  return 0;
}
```

当将s[4]同样赋值之后，在以%s输出就是乱码了,程序如下：

```c
#include<stdio.h>
int main()
{
  char s[5] = {0};
  s[0] = 'a';
  s[1] = 'b';
  s[2] = 'c';
  s[3] = 'd';
  s[4] = 'e';
  printf("%s\n",s);
  return 0;
}
```

**当一个数组之中既含有汉字又含有字母的时候**

==如果将一个字符串当做char处理，那么标准的ASACII字符一定是个整数，而汉字的第一个字节一定是负数==





## 字符串的使用


**1.去掉字符串右边的空格**

```c
#include<stdio.h>
int main()
{  
  char s[20] = "hello world   ";
  //首先获取字符串的长度
  int len = 0;
  while(s[len++]);
    len --;
    
  //因为是删除字符串右边的空格，因此需要从最右边进行遍历；

  int i = 0;
  for(i = len - 1;i >= 0;i--)
    {
      if(s[i] != ' ')
       { 
         s[i + 1] = 0;
          break;
       }
    }
    printf("(%s)\n",s);
    return 0;

}
```

**2.去掉字符串左边的空格**

```c
#include<stdio.h>

int  main()

{

  char  s[20]  =  "  hello  world";

  //首先获取字符串的长度

  int  len  =  0;

  while(s[len++]);

  len  --;  

  //因为是删除字符串右边的空格，因此需要从最右边进行遍历；

  int  i  =  0;

  int  j  =  0;

  for(i  =  0;i  <=  len  -1;i++)

  {
    if(s[i]  !=  '  ')

    {
      for(j  =  0;  j  <  len  -1;j++)    //当是s[i]不等于空格的时候，将空格元素要替换掉

        {
        s[j]  =  s[i  +  j];
        }
        break;

    }

  }

  printf("(%s)\n",s);

  return  0;

}

```
法二：

```c
#include<stdio.h>

int  main()

{

  char  s[20]  =  "  hello  world";

  //首先获取字符串的长度

  int  len  =  0;

  while(s[len++] ==' ');

  len  --; //得到了字符串前面的空格数量 

   int i = len;
   while(s[i])
   {
     s[i-len] = s[i];
     i++;
   }
   s[i - len] = 0; 

  printf("(%s)\n",s);

  return  0;

}

```

**3.字符串到整数的变化**

步骤：1.知道字符串的长度
2.将每个字符读取出来，转化为整数后*10的长度-1次方；
3.将每个位计算和加起来

```c
int main()
{
  char s[100] = "1234";
  int len = 0;
  while(s[len++]);
  len --;

  int value = 0;//存放变量，为将字符串转换为整数后的变量
  int i;
  int tmp = len;
  for (i = 0;i < len; i++) //遍历字符串
  {
    int base = 10; //求10的n次方
    if((tmp - i -1) == 0 )
    {
      base = 1;          //字符串的最后一个时候乘1
    }
     else
     {
       int j;
       for(j = 1; j < (tmp -i -1);j++)
       {
         base *= 10;
       }
    
     }
      value += (base * (s[i] - '0'));  
  }
   printf("%d\n",value);
   return 0;
}



```

~~下面的程序有问题，输出的值不对~~
```c
#include<math.h>

#include<stdio.h>

int  main()

{

  char  s[10]  =  "1234";

  int  len  =  0;

  while(s[len++]);

  len  --;

  double  result  =  0;

  int  i;

  int  j;

  int  wei  =  0;

  for(i  =  0;i  <  len;  i++)

  {

  if(len  -  i  -1  ==  0)

  {

  wei  =  s[i];

  }

  else

  {

  for(j  =  0  ;j  <  len  -  1;j++)

  {

  double  x  =  10;

  double  y  =  len  -1  -j;

  result  +=  (s[j])*(pow(x,y));

  }

  }

  result  +=  wei;

  printf("%d\n",result);

  return  0;

}

}
```






 





