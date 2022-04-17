# day02 指针强化

- 指针也是一种数据类型
- 指针变量也是一种变量


## 一、指针也是一种数据类型

 1.**指针变量和它指向的指针内存不同点：**
 
-  首先改变指针的值并不会改变指向的内存区域的值，因为指针的值为一个地址变量，而指向的内存中是存储的具体的值；
- 同样改变指针指向内存区域的值并不会改变指针的值；

---

2.**进行写内存时候，首先确保该内存可写（根据内存四区图）**
不允许向NULL和未知地址拷贝内存
```c
char *buf1 = "asdfghjkl";
buf1[2] = 1;
//这个是错误的，因为该字符串是放置在常量区，是不可更改的；

char buf2[100] = "shjfaksjdhgj";
buf2[3] = 3;
//可以更改，因为数组的元素都是放置在栈区，可以直接进行修改；

```

---

3.指针是一种数据类型，指针进行加一或者减一的步长取决于指向的元素的类型所占的字节；




## 二、通过指针进行间接赋值

1.**三大条件：**
- 定义两个变量
- 建立关系
- 通过`*`操作内存

```c
int main(void)
{
    //定义两个变量
    int a  = 100;
    int *p ;

    //建立关系
    p = &a;

    //通过*操作内存
    *p = 20;
    
}
```

2.**如果通过形参改变实参的内存内容（值），必须采用地址传递**

- 一级指针
例子1：将a的地址作为参数传递
```c
void get_str(int *b)
{
  *b = 200;
}

int main(void)
{
    int a = 100;
    get_str(&a);
    printf("a = %d\n",a);
}
```
程序运行结果为：`a = 200`



- 二级指针

例子2：（改变指针的地址）
此程序没有改变地址，本质上就是一个错误的程序，看下一个
```c

void fun(int *p)
{
  p = 0xaabb;
  printf("fun:p = %p\n",p);  
}


int main(void)
{
  int *p = 0x122;
  printf("p1 = %p\n",p);

  fun(p);//本质上是值传递
  printf("p2 = %p\n",p);

  system("pause");
  return 0;
}
```
程序运行结果：
`p1 = 0000000000000122`
`fun:p = 000000000000AABB`
`p2 = 0000000000000122`
由此可见指针p的地址并没有改变



程序3：改变指针的地址（通过二级指针进行修改）

```c
void fun(int **p)
{
  *p = 0xaabb;
  printf("fun:p = %p\n",*p);  
}


int main(void)
{
  int *p = 0x122;  
  printf("p1 = %p\n",p);

  fun(&p);//本质上是地址传递
  printf("p2 = %p\n",p);

  system("pause");
  return 0;
}
```
程序运行结果：
`p1 = 0000000000000122`
`fun:p = 000000000000AABB`
`p2 = 000000000000AABB`
通过二级指针可以更改指针的地址

再加一个示例加强理解：
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

void fun(char **p,int *len)
{
  if(p == NULL)
    {
      return;
    }
    char *tmp = (char *)malloc(100);
    if(tmp == NULL)
      {
        return;
      }
      strcpy(tmp,"adjfkdjfkdjf");

    //间接赋值
    *p = tmp;
    *len = strlen(tmp);
}

int main(void)
{
  char *p = NULL;
  int len = 0;
  fun(&p,&len);
  if(p != NULL)
    {
      printf("p = %s\n len = %d\n",p ,len);
    }
  return 0;
}
```
程序运行结果：
`p = adjfkdjfkdjf`
`len = 12`

![指针以及内存四区图]($resource/%E6%8C%87%E9%92%88%E4%BB%A5%E5%8F%8A%E5%86%85%E5%AD%98%E5%9B%9B%E5%8C%BA%E5%9B%BE.png)

---


3.**指针应该和内存四区相结合进行理解**

- 主调函数和被调函数
  - 主调函数可把堆区、栈区，全局数据内存地址传给被调用函数
  - 被调函数只能返回堆区、全局数据
- 内存分配方式
  - 指针可以作为输入输出
      - 输入：主调函数分配内存
      - 输出：被调用函数分配内存 


4.**区别三个语句**
```c
int fun()
{
  int a = 10;
  return a;
}
//ok，b的值为10

int *fun()
{
  int a = 10;
  return &a;
}
//本质上第一个和第二个是等价的
//p确实保存了fun()内部a 的地址，但是如果fun完毕之后，a就释放，p就指向未知区域

int *fun()
{
  static int a = 10;
  return &a;

}
//a放在全局区，函数运行完毕，a的空间不释放

int main()
{
  int b = fun();
}


```

## 三、字符串

### （一）基本操作

1.**初始化**
C语言没有字符串类型，只能通过字符数组模拟,本质上是字符数组
C语言字符串，以字符`\0`，或者数字0结尾

两个区别：
```c
#include<stdio.h>
#include<string.h>

int main(void)
{
  //不指定长度，而且没有0结束符
  char buf1[] = {'a','b','c'};
  printf("buf1= %s\n",buf1);


  //指定长度，当后面没有赋值的元素自动的补0
  char buf2[100] = {'a','b','c'};
  printf("buf2 = %s\n",buf2);

  //元素含有字符0
  char buf3[20] = {'a','b','0','4','5'};
  printf("buf3 = %s\n",buf3);
  
  //元素含有数字0
  char buf4[20] = {'a','b','0','4','5'};
  printf("buf4 = %s\n",buf4);

  //元素含有\0
  char buf5[20] = {'a','b','\0','4','5'};
  printf("buf5 = %s\n",buf5);


  //直接批量赋值
  char buf6[] = "asfsjgkjsdl";
  //strlen：测字符串长度，不包括数字0和字符\0
  //sizeof:测数组长度，包括数字0和字符\0
  printf("strlen = %d , sizeof = %d\n",strlen(buf6),sizeof(buf6));

  char buf7[100] = "asfsjgkjsdl";
  //strlen：测字符串长度，不包括数字0和字符\0
  //sizeof:测数组长度，包括数字0和字符\0
  printf("strlen = %d , sizeof = %d\n",strlen(buf7),sizeof(buf7));
  

  return 0;
}
```
程序运行结果：
`buf1= abc`
`buf2 = abc`
`buf3 = ab045`
`buf4 = ab045`
`buf5 = ab`
`strlen = 11 , sizeof = 12`
`strlen = 11 , sizeof = 100`




2.**使用字符串**
一般都是字符数组，所以就当做数组来用
- 使用[]方式
```c
#include<stdio.h>

int main()
{
  char buf[] = "abcdef";
  int i = 0;
  for(i = 0; i < strlen(buf) ; i++)
    {
      printf("buf[%d] =  %c", i ,buf[i]);
    }
    return 0;

}
```

- 使用指针方式
```c
#include<stdio.h>

int main()
{
  char buf[] = "abcdef";
  int i = 0;
  char *p = buf; 
  for(i = 0; i < strlen(buf) ; i++)
    {
       //以下表达式是等价的
      // printf("%c", p[i]);
       //printf("%c",*(p + i));
       printf("%c",*(buf + i));
      
    }
    return 0;

}
```


注：***和++的使用说明：**

```c
while(*a++ = *b++)
{
  NULL;
}
//首先执行：*a = *b;然后a++; b++; 然后判断*a是否为0，不为0则执行，为0跳出循环
```


注：**建议**
- 1.判断形参指针是否为空，然后在选择是否执行下面的操作
- 2.不要直接使用形参，先将形参赋值给另一个 变量，然后使用新的变量，这样可以保证形参指针一直执行原来的位置


3.**使用strstr查找字符串中的特定数组**
```c
#include<stdlib.h>
#include<string.h>
#include<stdio.h>


int main(void)
{
  char str[100] = "1234abcd3454343abcd678769878abcd9900abcd77637";
  char *p  = str;
  int num = 0;
  do
  {
    int num = 0;
    p = strstr(p ,"abcd");
    if(p != NULL)
      {
          num++;
          p = p + strlen("abcd");
      
      } else
      {
          break;
      }
  }while(p != NULL);

  printf("num =%d\n",num);
  return 0;
}
```
程序运行结果：`n = 4`


**使用while**
```c
int my_strstr(char *p ,int *n)
{
  int i = 0;
  char *tmp = p ;
  while((tmp = strstr(tmp,"abcd")) != NULL)
    {
      //能进来就代表肯定有匹配的字符串

      //重新设置起点位置
      tmp = tmp + strlen("abcd");
      i++;

      if(*tmp == 0)//如果到结束符
        {
          break;
        }
    }
    //间接赋值
    *n = i;
    return 0;

}



int main()
{
  char str[100] = "1234abcd3454343abcd678769878abcd9900abcd77637";
  int ret = 0;
  int num = 0;

  ret = my_strstr(p,&n);
  if(ret != 0)
  {
    return ret;
  }
  printf("num = %d\n",num);
  return 0;
}
```



4.**一个字符串两头均为空格，求中间一部分的字符长度**
分别同时从左边和从右边向中间查找
```c
#include<stdio.h>
#include<string.h>
#include<ctype.h>

int fun(char *p, int *n)
{
	if (p == NULL || n == NULL)
	{
		return -1;
	}

	int begin = 0;
	int end = (int)strlen(p) - 1;

	//从左边开始
	//如果当前字符为空，而且没有结束
	while (isspace(p[begin]) && p[begin] != 0)
	{
		begin++;//位置往后移动一位
	}

	//如果当前字符为空，而且没有结束
	while (isspace(p[end]) && p[end] != 0)
	{
		end--;//往左移动
	}
	*n = end - begin + 1;
	return 0;

}



int main(void)
{
	char *p = "   abcdefg     ";
	int ret = 0;
	int n = 0;

	ret = fun(p, &n);
	if (ret != 0)
	{
		return ret;
	}

	printf("n = %d\n", n);
	return 0;


}

```

4.2**将中间有元素的那一部分打印出来**
```c
#include<stdio.h>
#include<string.h>
#include<ctype.h>

int fun(char *p, int *buf)
{
	if (p == NULL || buf == NULL)
	{
		return -1;
	}

	int begin = 0;
	int end = (int)strlen(p) - 1;
	int num = 0;

	//从左边开始
	//如果当前字符为空，而且没有结束
	while (isspace(p[begin]) && p[begin] != 0)
	{
		begin++;//位置往后移动一位
	}

	//如果当前字符为空，而且没有结束
	while (isspace(p[end]) && p[end] != 0)
	{
		end--;//往左移动
	}
	num = end - begin + 1;

	strncpy(buf, p + begin, num);
	buf[num] = 0;
	return 0;

}



int main(void)
{
	char *p = "   abcdefg     ";
	char buf[100] = { 0 };
	int ret = 0;
	int n = 0;

	ret = fun(p, buf);
	if (ret != 0)
	{
		return ret;
	}


	printf("buf = %s", buf);

	return 0;
	system("pause");
}

```

## 复习以及作业

1.**.c可程序程序过程**
**预处理：** 宏定义展开、头文件展开、条件编译、这里不会检查语法
**编译：** 检查语法，将预处理后的文件编译生成汇编文件
**汇编:** 将汇编文件生成目标文件（二进制文件）
**链接：** 将目标文件链接称为可执行程序

程序只有在运行才加载到内存（由系统完成 ），但是某个变量具体分配多大空间在编译的时候已经确定，当实际运行时候正式分配



- 作业
1.**根据元素位置的奇偶性挑出字符串中的对应元素**

```c
/*
有一个字符串“1a2b3c4d”
实现以下的功能：
功能1：把偶数位字符挑选出来，组成字符串1；
功能2：把奇数位字符挑选出来，组成字符串2；
功能3：把字符串1和字符串2通过函数参数返回至main，并且打印
功能4：主函数能测试通过
*/

#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int getstr1str2(char *p, char *buf1, char *buf2)//返回值为int,不是void
{
	char *str = p;
	//因为参数赋值时候为 char *p = buf,等价于 char *p ;  p = buf
	
		if (str == NULL || buf1 == NULL || buf2 == NULL)//当这三个全为空的时候
		{
			return -1;
		}
		else
		{
			int i = 0;
			for (i = 0; i < strlen(str); i++)
			{
			if (i % 2 != 0)//当为奇数位置的时候
			{
				*buf1 = str[i];
				buf1++;
			}
			else
			{
				*buf2 = str[i];
				buf2++;

			}

		}

	}
	return 0;

}


int main(void)
{
	char *p = "1a2b3c4d";
	char buf1[50] = { 0 };
	char buf2[50] = { 0 };
	int ret = 0;

	ret = getstr1str2(p,buf1,buf2);
	if (ret != 0)
	{
		printf("getdtr1str2 err: %d\n", ret);
		return ret;
	}

	printf("buf1 = %s\n",buf1);
	printf("buf2 = %s\n",buf2);
	system("pause");
	return 0;

}
```
程序运行结果：
`buf1 = abcd`
`buf2 = 1234`


2.**查找键值**
如果查到键值，就返回键值对应等号后面的值以及字符串的长度
```c
/*
查找键值（“key = value”）字符串，在开发中经常使用
要求1：请自己定义一个接口，实现根据key获取
要求2：编写测试用例
要求3：键值对中间可能有n多空格，请去除空格


注意：键值对字符串格式可能如下：
"key1 = value1"
"key2 =       value2"
"key3   = valude3"
"key4             = value4"
"key5   =     "
"key6   ="

*/


#include<stdio.h>
#include<string.h>
#include<ctype.h>
#include<stdlib.h>
#define _CRT_SECURE_NO_WARNINGS 
#pragma warning(disable:4996)


//这是两边夹的函数，去除空格的作用
int  trimSpace(char *inbuf, char *outbuf)
{
	if (inbuf == NULL || outbuf == NULL)
	{
		return -1;
	}
	char *p = inbuf;
	int begin = 0;
	int end = strlen(p) - 1;
	int n = 0;

	if (end < 0)
	{
		return -2;
	}

	while (isspace(p[begin]) && p[begin] != 0)
	{
		begin++;
	}
	
	while (isspace(p[end]) && end > 0)//保证空字符串的时候跳出来
	{
		end--;
	}

    if(end == 0)
      {
        return -2;
      }
      
	n = end - begin + 1;//非空元素的个数
	strncpy(outbuf, p + begin, n);
	outbuf[n] = 0;
	return 0;
}





//查找key值的函数，并且获取后面的字符串、
int getKeyByValue(char *keyvaluebuf, char *keybuf, char *valuebuf, int *valuebuflen)
{
	/*
	keyvaluebuf = "key4 =     value4"
	keybuf = "key4"
	*/

	if (keybuf == NULL || keyvaluebuf == NULL || valuebuf == NULL || valuebuflen == NULL)
	{
		return -1;
	}

	char *p = NULL;
	int ret = 0;
	//查找匹配键值
	//“key4 =     value4”,找key4，找到返回首地址
	p = strstr(keyvaluebuf, keybuf);
		if(p == NULL)
		{
			return -2;
		}

	//如果找到，重新设置起点位置，跳过“key4”
	//"key4 =   value4"  -> "=   value4"
	p = p + strlen(keybuf);


	//查到“=”
	p = strstr(p, "=");
	if (p == NULL)
	{
		return -3;
	}

	//如果找到，重新设置起点位置，跳过“=”
	//"=     value4" ->  "   value4"
	p = p + strlen("=");


	//去非空字符
	ret = trimSpace(p, valuebuf);
	if (ret != 0)
	{
		printf("trimSpace err : %d\n", ret);
		return ret;
	}

	//获取长度，通过*间接赋值
	*valuebuflen = strlen(valuebuf);
	return 0;
}

int main()
{
	char *keyVal = "key4             = value4";
	char *key = "key4";//想要查找的key
	char value[100] = { 0 };
	int len = 0;
	int ret = 0;

	ret = getKeyByValue(keyVal, key, value, &len);
	if (ret != 0)
	{
		printf("getKeyByValue err : %d \n", ret);
		return ret;
	}

	printf("Val:%s\n", value);
	printf("len = %d\n",len );

	system("pause");
	return 0;

}

```
程序运行结果：
`Val:value4`
`len = 6`





