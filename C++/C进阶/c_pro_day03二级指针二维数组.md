# day03二级指针二维数组


## 一、const
- 修饰变量
将变量变为只读，不能更改，所以定义的时候必须初始化
```c
const int a = 10;
a = 100;//error
```
可以使用指针修改：
```c
const int a = 10;
int *p = &a;
*p = 22;
```
- 修饰指针
**从左往右看,跳过类型,看const修饰哪一个字符，如果是`*`,则表示指针指向的内存不能改变,如果是指针变量,则表示指针的指向不能改变,即指针的值不能改变** 
  - 指向的内存不能变
```c
char buf[100] = "akldfjksdjgf";
const char *p = buf;
//等价于：char const *p = buf;
//只能更改指针值（指向），但是指向的内存数据不能更改

```

  - 指向不能变

```c
char *const p2 = buf;
p2[2] = '4';//这是可以的，但是不能改变指针的值 
```

  - 指向和值都不能改
```c
const char *const p3 = buf;
//指针的值和指向的内存的值都是不可更改
```

注；**如何引入外部文件定义的const变量（同一个项目中的不同c文件）**
例子：在a.c中定义：`const int a = 10;`
在b.c中定义使用：`extern const int a;`//只能声明，不能在进行赋值





## 二、指针作为函数参数


### （一）值传递（形参的修改不会影响到实参的值）
例子：（这里main函数中的p和getMem函数中的p未建立对应关系）
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#pragma warning(disable:4996)

int getMem(char *p)
{
	p = (char *)malloc(sizeof(char) * 100);
	if (p == NULL)
	{
		return -1;
	}
	strcpy(p ,"abdakg");
	printf("p = %s\n",p );
	return 0;
}


int main(void)
{
	char *p = NULL;
	int ret = 0;
	ret = getMem(p);
	if (ret != 0)
	{
		printf("getMem err: %d\n",ret);
		return ret;
	}
	printf("p = %s\n",p);
}
```
程序运行结果：
`p = abdakg`
`p = (null)`
程序运行内存四区图：
![程序运行内存四区图]($resource/%E7%A8%8B%E5%BA%8F%E8%BF%90%E8%A1%8C%E5%86%85%E5%AD%98%E5%9B%9B%E5%8C%BA%E5%9B%BE.png)


---
**1.二级指针做函数参数输出特性**
```c

#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#pragma warning(disable:4996)

int getMem(char **p)
{
	if (p == NULL)
	{
		return -1;
	}

	char *tmp = (char *)malloc(sizeof(char) * 100);
	if (tmp == NULL)
	{
		return -2;
	}
	strcpy(tmp, "abdakg");
	*p = tmp;//即是将tmp赋值给p指向的内存空间
	return 0;
}

int main(void)
{
	char *p = NULL;//这时候并没有分配内存
	int ret = 0;
	ret = getMem(&p);
	if (ret != 0)
	{
		printf("getMem err: %d\n", ret);
		return ret;
	}
	printf("p = %s\n", p);
	if (p != NULL)
	{
		free(p);
		p = NULL;

	}
	return 0;
}
```
程序运行结果：
`p = abdakg`
程序运行对应的内存四区图
![二级指针做参数输出]($resource/%E4%BA%8C%E7%BA%A7%E6%8C%87%E9%92%88%E5%81%9A%E5%8F%82%E6%95%B0%E8%BE%93%E5%87%BA.png)

---

## 指针数组的使用

**1.定义指针：**
**指针数组的本质：** 是一个数组，只是每一个元素都是指针char *

- 第一种内存模型
```c
char *p1 = "dkfjdksjf";
char *p2 = "djfdkshfa";
char *p3 = "dkfjaioerio";
//若简化定义一系列定义，可使用指针数组

char *p[] = {"dkfjdksjf","djfdkshfa","dkfjaioerio"};
int  n = sizoeof(p) /sizeof(p[0]);
//这种方法只能是没有定义数组长度的时候使用，以下方法就不能使用
//char *p[10] = {"dkfjdksjf","djfdkshfa","dkfjaioerio"};
//printf("sizeof(p) = %d,sizeof(p[0]) = %d\n",sizeof(p),sizeof(p[0]));
//输出结果为：40和4（因为32位系统中指针占4个字节）



//打印方式
 int i = 0;
 for(i = 0; i < n ; i++)
   {
       printf("%s\n",p[i]);
   }
```
内存四区图：

![第一种内存模型图]($resource/%E7%AC%AC%E4%B8%80%E7%A7%8D%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E5%9B%BE.png)


**2.排序：**
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>


void print_array(char *p[], int n)
//或者void print_array(char **p,int n)
{
	int i = 0;
	for (i = 0; i < n; i++)
	{

		printf("%s ,", p[i]);
	}
	printf("\n");

}

void  sort_array(char **p,int n)
{
	int i = 0;
	int j = 0;
	int *tmp = NULL;

	//选择法排序
	for (i = 0; i < n - 1; i++)
	{
		for (j = i + 1; j < n; j++)
		{
			if (strcmp(p[i], p[j]) > 0)
			{
				tmp = p[i];
				p[i] = p[j];
				p[j] = tmp;

			}
		}
	}
}



int main(void)
{
	char *p[] = {"1111111","44444444","aaaaaaaaaa","00000000"};

	int  n = sizeof(p) / sizeof(p[0]);
	
	printf("shuruqianshuzu:");

	print_array(p,n);
	
	sort_array(p,n);

	printf("paixuzhihou:");
	print_array(p, n);

	return 0;
}

```
程序运行结果；
`shuruqianshuzu:1111111 ,44444444 ,aaaaaaaaaa ,00000000 ,`
`paixuzhihou:00000000 ,1111111 ,44444444 ,aaaaaaaaaa ,`


**3.实参与形参传值参数的定义方法：**
即形参应该怎么写
例子1：
```c
void array(char a[]//或者char *a)
int main()
{
   char p[] = {"1111","4444","aaaa","0000"};
   array(p);
}
```

例子2：
```c
void array(char *a[]//或者char **a)
int main()
{
   char *p[] = {"1111","4444","aaaa","0000"};
   array(p);
}

```


## 三、二维数组的使用

**1.一维数组 的地址**
```c
char a[10] = "dkfjdkfj";
printf("%d,%d,%d,%d\n",a,a+1,&a,&a+1);//&a表示整个一维数组的地址，a表示首元素的地址
//结果示例：1 ，2，1，11



int i = 0;
for(i = 0; i < 10; i++)
{
  printf("%s\n",a[i]);//这里的a[i] 等于 a+i  或者 *(a+i),因为这里的首行地址和首元素地址是一样的
}




char a[4][30]= {"sfdsfaf","dfdgdfgre","dfdogio"};
printf("%d,%d,%d,%d\n",a,a+1,&a,&a+1);
//结果示例：1，31,1,120


```


**2.将二维数组排序打印输出：**
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#pragma warning(disable:4996)


void print_array(char p[][30], int n)
//或者void print_array(char **p,int n)
{
	int i = 0;
	for (i = 0; i < n; i++)
	{

		printf("%s ,", p[i]);
	}
	printf("\n");

}


void  sort_array(char p[4][30], int n)
{
	int i = 0;
	int j = 0;
	int tmp[30] = {0};

	//选择法排序
	for (i = 0; i < n - 1; i++)
	{
		for (j = i + 1; j < n; j++)
		{
			if (strcmp(p[i], p[j]) > 0)
			{
				//不能使用指针，只能交换内存块
				strcpy(tmp,p[i]);
				strcpy(p[i],p[j]);
				strcpy(p[j],tmp);

			}
		}
	}
}

int main(void)
{
	char p[4][30] = { "1111111","44444444","aaaaaaaaaa","00000000" };

	int  n = sizeof(p) / sizeof(p[0]);//结果应该是4

	printf("shuruqianshuzu:");

	print_array(p, n);

	sort_array(p, n);

	printf("paixuzhihou:");
	print_array(p, n);

	return 0;
}

```

程序运行结果：
`shuruqianshuzu:1111111 ,44444444 ,aaaaaaaaaa ,00000000 ,`
`paixuzhihou:00000000 ,1111111 ,44444444 ,aaaaaaaaaa ,`


**二级指针第三种内存模型**
分配在堆区

```c
#include<stdio.h>
#include<string.h>
#pragma warning(disable:4996)


int main(void)
{
	//只有一个指针的情况下进行分配堆区
	char *p0 = NULL;
	p0 = (char *)malloc(sizeof(char) * 100);
	strcpy(p0,"sfjkdjfkld");


	//如果有10个同类型的指针进行分配堆区
	int i = 0;
	char *p[10] = { 0 };
	for (i = 0; i < 10; i++)
	{
		p[i] = (char *)malloc(sizeof(char) * 100);
		strcpy(p[i], "sdfkdjsfk");
	}
	
	
	//数组
	int a[10];
	int *q = (int *)malloc(10 * sizeof(int));//q[10]


	//动态分配一个数组，每个元素都是char *
	int n = 10;
	char **buf = (char **)malloc(n * sizeof(char *));
	if (buf == NULL)
	{
		return -1;
	}

	for (i = 0; i < n; i++)
	{
		buf[i] = (char *)malloc(30 * sizeof(char));
		char str[30];
		sprintf(str, "test%d%d", i, i);
		strcpy(buf[i], str);
	}

	for (i = 0; i < n; i++)
	{
		printf("%s, ",buf[i]);
	}

	printf("\n");

	for (i = 0; i < n; i++)
	{
		free(buf[i]);
		buf[i] = NULL;
	}

	if (buf != NULL)
	{
		free(buf);
	}
	return 0;
}
```
二级指针的第三种内存模型为：

![二级指针的第三种内存类型]($resource/%E4%BA%8C%E7%BA%A7%E6%8C%87%E9%92%88%E7%9A%84%E7%AC%AC%E4%B8%89%E7%A7%8D%E5%86%85%E5%AD%98%E7%B1%BB%E5%9E%8B.png)


**将上面的程序进行函数封装：**
```c
#include<stdio.h>
#include<string.h>
#pragma warning(disable:4996)


char *getMem(int n)
{
	int i = 0;
	char **buf = (char **)malloc(n * sizeof(char *));
	if (buf == NULL)
	{
		return NULL;
	}

	for (i = 0; i < n; i++)
	{
		buf[i] = (char *)malloc(30 * sizeof(char));
		char str[30];
		sprintf(str, "test%d%d", i, i);
		strcpy(buf[i], str);
	}
	return buf;
}

void print_buf(char **buf,int n)
{
	int i = 0;
	for (i = 0; i < n; i++)
	{
		printf("%s, ", buf[i]);
	}

	printf("\n");
}

void free_buf(char **buf, int n)
{
	int i = 0;
	for (i = 0; i < n; i++)
	{
		free(buf[i]);
		buf[i] = NULL;
	}

	if (buf != NULL)
	{
		free(buf);
		buf = NULL;
	}

}

int main(void)
{
	char **buf = NULL;
	int n = 3;
	buf = getMem(n);
	if (buf == NULL)
	{
		printf("getMem err\n");
		return -1;
	}

	print_buf(buf,n);
	free_buf(buf, n);
	buf = NULL;
	return 0;
}
```
程序运行结果为：
`test00, test11, test22,`


## 作业

1.**作业1：**
已知：`char buf[] = “abcdef”`,下面表达式的区别？
- `const char *p = buf`
- `char const *p = buf`
- `char *const p = buf`
- `const char *const  *p = buf`

2.**求下列表达式的值：**
- `char *p1[] = {“1111”,“2222”,“3333”};`
   `sizeof(p1) /sizeof(p1[0])`= 3*4 （指针占4个字节）/ 4 = 3;
```c
#include<stdio.h>

int main(void)
{
	char *p1[] = {"1111","2222","3333"}; 
	printf("%d,%d,%d\n", sizeof(p1), sizeof(p1[0]), sizeof(p1) / sizeof(p1[0]));
	system("pause");
	return 0;
}
```
//结果：12，4, 3

- `char *p2[10] = {“1111”,“2222”,“3333”};`
   `sizeof(p2) /sizeof(p2[0])`= 10*4 /4 = 10;
```c
#include<stdio.h>

int main(void)
{
	char *p2[10] = {"1111","2222","3333"}; 
	printf("%d,%d,%d\n", sizeof(p2), sizeof(p2[0]), sizeof(p2) / sizeof(p2[0]));
	system("pause");
	return 0;
}

```
//结果：40,4,10

- `char p3[][30] = {“1111”,“2222”,“3333”};`
   `sizeof(p3) /sizeof(p3[0])`=  3*30 / 30 = 3;
```c
#include<stdio.h>

int main(void)
{
	char p3[][30] = { "1111","2222","3333" };
	printf("%d,%d,%d\n", sizeof(p3), sizeof(p3[0]), sizeof(p3) / sizeof(p3[0]));
	system("pause");
	return 0;
}
```
//结果：90,30,3


- `char p4[10][30] = {“1111”,“2222”,“3333”};`
   `sizeof(p4) /sizeof(p4[0])`= 10*30 /30 = 10;

```c
#include<stdio.h>

int main(void)
{
	char p4[10][30] = { "1111","2222","3333" };
	printf("%d,%d,%d\n", sizeof(p4), sizeof(p4[0]), sizeof(p4) / sizeof(p4[0]));
	system("pause");
	return 0;
}
```
//结果：300,30,10

----

3.`char buf[][30] = {“1111”,“2222”,“3333”};`
二维数组做参数的时候，为什么不能写成`void fun(char **buf);`
为什么可以写成：`void fun(char buf[][30])`

因为第一个的buf +1的步长为4，而原来的+1步长为30；

---

4.//画出三种二级指针内存模型图
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

int main(void)
{
	//指针数组
	char *p1[] = {"123","456","789"};//三个元素，每个占四个字节

	//二维数组
	char p2[3][4] = { "123","456","789" };

	//手工二维内存
	char **p3 = (char **)malloc(3 * sizeof(char *));//char *array[3];

	int i = 0;
	for (i = 0; i < 3; i++)
	{
		p3[i] = (char  *)malloc(10 * sizeof(char));//char buf[10]
		sprintf(p3[i], "%d%d%d", i, i, i);
	}
	system("pause");
	return 0;
}
```

![作业四：三种二级指针内存模型图]($resource/%E4%BD%9C%E4%B8%9A%E5%9B%9B%EF%BC%9A%E4%B8%89%E7%A7%8D%E4%BA%8C%E7%BA%A7%E6%8C%87%E9%92%88%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E5%9B%BE.png)


**作业5** ~~程序有点问题，输出结果与预期不同~~
```c
 /*
有字符串有以下特征（"abcd11111abcd222222abcdqqqqqqqq"）,请写一个函数接口，输出一下结果：
把字符串替换为（dcba11111dcba222222dcbaqqqqqqqq），并把结果传出
要求：
1.正确实现接口和功能
2.编写测试用例

src: 原字符串
dst:生成的或需要填充的字符串
sub:需要查找的字符串
new_sub :替换的新字符串

return : 0 成功
		-1 失败

*/

#include<stdio.h>
#include<string.h>
#pragma warning(disable:4996)

int replaceSubstr(char *src, char **dst, char *sub, char *new_sub)/*in ,out ,in,in*/
{
	if (src == NULL || dst == NULL || sub == NULL || new_sub == NULL)
	{
		return -1;
		
	}
	/*
	src = "dddabcd11111abcd222222abcdqqqqqqqq"
	sub = "abcd"
	new_sub = "lalalala"
	
	
	*/

	char *start = src;
	char *p;
	char tmp[512] = { 0 };
	int len = 0;

	do
	{
		p = strstr(start, sub);
		if (p != NULL)
		{
			len = 0;
			len = p - start;

			if (len > 0)
			{
				strcat(tmp, start, len);//tmp = "ddd"

			}

			strncat(tmp, new_sub, strlen(new_sub));//tmp = "dddlalala"

			//重新设置起点位置
			start = p + strlen(sub);
		}
		else
		{
			strcat(tmp, start);
			break;
		}
		}while (*start != '\0');//start[i] != 0
		char *buf = (char *)malloc(strlen(tmp) + 1);
		strcpy(buf, tmp);

		//间接赋值是指针存在的最大意义
		*dst = buf;



	return 0;
}

void freeBuf(char *buf)
{
		free(buf);
		buf = NULL;
}


int main(void)
{
	char *p = "dddabcd11111abcd222222abcdqqqqqqqq";
	char *buf = NULL;//在replaceSubstr函数里面分配空间
	int ret = 0;

	ret = replaceSubstr(p, &buf, "abcd", "lalalala");
	
	if (ret != 0)
	{
		printf("replaceSubstr err :%d \n", ret);
		system("pause");
		return ret;
	}

	printf("p = %s\n", p);
	printf("buf = %s\n", buf);

	if(buf != NULL)
	{
		freeBuf(buf);
	}
	buf = NULL;

	system("pause");
	return 0;

}

```


**作业六：**
```c
/*
有一个字符串符合以下特征（"abcdef,acccd,eeee,aaaa,e3eeee,ssss"）
写两个函数（API），输出以下结果：
第一个API（第二种内存模型：二维数组）
- 以逗号分割字符串，形成二维数组，并把结果传出
- 把二维数组行数运算结果也传出
int spitString(const char *str ,char c,char buf[10][30] ,int *count);   //in


第二个API（第三种内存模型：动态生成二维内存）
- 以逗号分隔字符串，形成一个二级指针；
- 把一共拆分多少行字符串个数传出；
int spitString2(const char *str ,char c,char **myp ,int *count);   //in



要求：
- 能正确表达功能的要求，定义出接口
- 正确实现接口和功能
- 编写正确的测试用例

*/


//这是第一个API

#include<stdio.h>
#include<string.h>
#pragma warning(disable:4996)

int spitString(const char *str, char c, char buf[10][30], int *count)
{
	if (str == NULL || count == NULL)
	{
		return -1;
	}

	const char *start = str;
	char *p = NULL;
	int i = 0;

	do
	{
		p = strchr(start, c);
		if (p != NULL)
		{
			int len = p - start;
			strncpy(buf[i], start, len);
			//结束符
			buf[i][len] = 0;

			i++;

			//重新设置起点位置

			start = p + 1;
		}
		else
		{
		    strcpy(buf[i] ,start);
		    i++;
			break;

		}

	} while (*start != 0);

	if (i == 0)
	{
		return -2;
	}

	*count = i;

	return 0;



}


int main(void)
{
	const char *p = "abcdef,acccd,eeee,aaaa,e3eeee,ssss";
	char buf[10][30] = { 0 };
	int n = 0;
	int  i = 0;
	int ret = 0;
	ret = spitString(p, ',', buf, &n);
	if (ret != 0)
	{
		printf("spitString err : %d\n", ret);
		system("pause");
		return ret;
	}

	for (i = 0; i < n; i++)
	{
		printf("%s\n", buf[i]);

	}
	printf("\n");
	system("pause");
	return 0;
}

```

**第二个API**
```c

/*
有一个字符串符合以下特征（"abcdef,acccd,eeee,aaaa,e3eeee,ssss"）
写两个函数（API），输出以下结果：
第一个API（第二种内存模型：二维数组）
- 以逗号分割字符串，形成二维数组，并把结果传出
- 把二维数组行数运算结果也传出
int spitString(const char *str ,char c,char buf[10][30] ,int *count);   //in


第二个API（第三种内存模型：动态生成二维内存）
- 以逗号分隔字符串，形成一个二级指针；
- 把一共拆分多少行字符串个数传出；
int spitString2(const char *str ,char c,char **myp ,int *count);   //in



要求：
- 能正确表达功能的要求，定义出接口
- 正确实现接口和功能
- 编写正确的测试用例

*/




#include<stdio.h>
#include<string.h>
#pragma warning(disable:4996)

int spitString(const char *str, char c, char **buf, int *count)
{
	if (str == NULL || count == NULL)
	{
		return -1;
	}

	const char *start = str;
	char *p = NULL;
	int i = 0;

	do
	{
		p = strchr(start, c);
		if (p != NULL)
		{
			int len = p - start;
			strcpy(buf[i], start, len);
			//结束符
			buf[i][len] = 0;

			i++;

			//重新设置起点位置

			start = p + 1;
		}
		else
		{
			break;

		}

	} while (*start != 0);

	if (i == 0)
	{
		return -2;
	}

	*count = i;

	return 0;



}

char **getMem(int n)
{
	char **buf = NULL;
	buf = (char **)malloc(n * sizeof(char *));
	if (buf == NULL)
	{
		return NULL;
	}

	int i = 0;
	for (i = 0; i < n; i++)
	{
		buf[i] = (char *)malloc(30);
	}
	return buf;
}






int main(void)
{
	const char *p = "abcdef,acccd,eeee,aaaa,e3eeee,ssss";
	char **buf = NULL;
	int n = 0;
	int  i = 0;
	int ret = 0;

	buf = getMem(6);//6个

	if (buf == NULL)
	{

		return -1;
	}

	ret = spitString(p, ',', buf, &n);
	if (ret != 0)
	{
		printf("spitString err : %d\n", ret);
		system("pause");
		return ret;
	}

	for (i = 0; i < n; i++)
	{
		printf("%s\n", buf[i]);

	}


	//释放
	for (i = 0; i < n; i++)
	{
		free(buf[i]);
		buf[i] = NULL;


	}

	if (buf != NULL)
	{
		free(buf);
		buf = NULL;

	}

	printf("\n");
	system("pause");
	return 0;
}
```


