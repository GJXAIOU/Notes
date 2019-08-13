# Day04二级指针与数组


## 三、多级指针的使用

没听懂

## 四、一维数组的使用
**1.包括一维数组的定义和使用**
```c
#include<stdio.h>
#include<string.h>


int main(void)
{
	int a[] = { 1,2,3,4,5,6,7,8 };
	//sizeof() 是测量 变量多占的空间（变量所对应的类型的空间（这里是数组类型，并不是整数类型））
	//数组类型空间：由元素个数和元素类型所决定   8*4 = 32
	//sizeof(a[0]) 是首元素的大小，每个元素占4个字节

	int num = 0;
	num = sizeof(a) / sizeof(a[0]);

	//使用
	int i = 0;
	for (i = 0; i < num; i++)
	{
		printf("%d ", *(a + i));
	//这里a+i ，因为a表示首元素的地址，因此a+i 表示第i个元素的地址，因此 *（a+i）表示第i个元素的内存值；
	}


	//数组类型

	//a代表首元素的地址
	//&a表示整个数组的首地址，它和 首元素地址是一样的，但是步长不同；
	printf("a = %d   a+1 =  %d\n ", a, a + 1);//步长为4
	printf("&a = %d  &a+1  = %d\n", &a, &a + 1);//步长为32
	

	//因为数组类型由元素个数和元素类型决定，因此可以通过typedef定义一个数组类型。
	//如果有typedef则表示是类型，没有的话为变量

	//定义：
	typedef int A[8];//这就表示是数组类型，不是变量，等价于typedef int (A)[8]
	//使用：
	A b;   //怎么看；将typedef去掉，同时将b替换到A的位置，结果为：int b[8],其他使用一样


}
```
**2.使用typedef定义数组类型**
```c
//因为数组类型由元素个数和元素类型决定，因此可以通过typedef定义一个数组类型。
	//如果有typedef则表示是类型，没有的话为变量

	//定义：
	typedef int A[8];//这就表示是数组类型，不是变量，等价于typedef int (A)[8]
	//使用：
	A b;   //怎么看；将typedef去掉，同时将b替换到A的位置，结果为：int b[8],其他使用一样


//数组元素赋值
for(i = 0 ; i < 8 ; i++)
{
  b[i] = i;
}

```


## 五、指针数组

```c
#include<stdio.h>
#include<string.h>

//argc： 传参数的个数（包含可执行程序）
//argv:  指针数组，指向输入的参数内容
int main(int  argc, char *argv[])//这个是系统调用，而里面的参数只能从命令行进行输入
{
	//指针数组：本质上为数组，只是每个元素都是指针
	//[]的优先级是比*优先级高


	char *a[] = { "aaaaa","bbbbbbb","cccccccc" };
	int i = 0;

	printf("argc = %d\n", argc);

	for (i = 0; i < argc; i++)
	{
		printf("%s\n", argv[i]);
	}




}
```

1.**指针数组的定义：**
```c
//指针数组变量
//[]优先级比*高，它是数组，每个元素都是指针（char *）
char *str[] = {"123","578"};
char **str = {"123","578"};//错误的

```
2.**指针数组做形参**
```c
//以下两个等价
void fun(char *str[]);
void fun(char **str);//因为str[] = *str

```

3.**main函数的指针数组**
```c
//argc :传参数的个数（包括可执行程序）
//argv: 指针数组，指向输入的参数

int main(int argc, char *argv[]);

:demo.exe a b test
int argc = 4
char *argv[] = {"demo.exe","a","b","test"}


```

## 六、数组指针


数组指针：本质上为指针，是一个指向数组的指针
定义方法：三种
**定义方法1：**
```c
//方法一；先定义数组类型，根据类型定义指针变量
#include<stdio.h>
#include<string.h>

int main(void)
{
	int a[10] = { 0 };
	//定义数组指针变量


	typedef int A[10];  //A 数组类型，[10]也可以认为是步长
	A *p = NULL;   //p是数组指针类型变量
	p = &a;//因为数组指针是指向一维数组的这个数组，而不是首元素地址，所以使用p = a；的时候会有警告

	printf("p: %d ,p +1: %d\n", p, p + 1);

	//赋值
	for (i = 0; i < 10; i++)
	{
		//因为p = &a;所以*p = *&a,即是a
		(*p)[i] = i + 1;
	}

	system("pause");
	return 0;
}
```

**定义方法2：**
```c
//先定义数组指针类型，然后根据了类型定义变量，写法比指针数组多了一个（）
#include<stdio.h>
#include<string.h>

int main(void)
{
	int a[10] = { 0 };
	
	//（）和[]的优先级是一样的，均是从左往右
	//（）里面有指针，所以这是一个指针，[]表示数组，因此这是一个指向数组的指针，又因为前面有typedef,因此这是一个数组指针类型
	typedef int(*P)[10];
	P q;//q是数组指针变量
	q = &a;

	
	//赋值
	int i = 0;
	for (i = 0; i < 10; i++)
	{
		(*q)[i] = i + 1;
	}

	//打印输出
	for (i = 0; i < 10; i++)
	{
		printf("%d ", (*q)[i]);


	}
	printf("\n");

	system("pause");
	return 0;
}


```
**定义方法3：**
```c

//直接定义数组指针变量
#include<stdio.h>
#include<string.h>

int main(void)
{
	int a[10] = { 0 };
	
	//（）和[]的优先级是一样的，均是从左往右
	//（）里面有指针，所以这是一个指针，[]表示数组，因此这是一个指向数组的指针，没有typedef,因此这是一个数组指针变量
	int(*p)[10];//p是数组指针变量
	p = &a;

	
	//赋值
	int i = 0;
	for (i = 0; i < 10; i++)
	{
		(*p)[i] = i + 1;
	}

	//打印输出
	for (i = 0; i < 10; i++)
	{
		printf("%d ", (*p)[i]);


	}
	printf("\n");

	system("pause");
	return 0;
}

```



## 七、二维数组

1.**定义与使用**
```c
#include<stdio.h>
#include<string.h>

int main(void)
{
	//二维数组的定义：主要这两种方式

	int a[4][8] = { 1,2,3,4,5,6,7,8,9,10 };
	int a1[4][8] = { {1,2,3},{4,5,6,7},{8,9,10,11} };
	
	//打印输出：

	int i = 0;
	int j = 0;
	for (i = 0; i < 4; i++)
	{
		for (j = 0; j < 8; j++)
		{
			printf("a[%d][%d] = ", a[i][j]);
		}
		printf("\n");

	}
	printf("\n");


	//地址：二维数组中数组名表示第0行首地址
	printf("%d,%d\n", a, a + 1);//第0行首地址，第一行首地址
	printf("%d,%d\n", a[0], a[1]);//含义同上
	printf("%d ,%d\n", *(a + 0), *(a + 1));//第0行首元素地址，第1行首元素地址
	printf("%d,%d\n", *(a + 0) + 1);//表示第0行第二个元素的地址。
	printf("%d,%d\n", a[0] + 1);//含义同上 

  /*
  以int a[][4] = {1,2,3,4,5,6,7,8,9,10,11,12};为例
  //a:代表第0行的首地址
  //a + i 等价于 &a[i]  ，代表第i行首地址
  //*(a + i) 等价于 a[i]: 表示第i行首元素地址
  //*(a + i) + j  等价于 &a[i][j]  :表示第i行第j列元素的地址
  //*(*(a +i) + j) 等价于 a[i][j]  :表示第i行第j列元素的值
 
  */

	return 0;

}

```

2.**求二维数组元素个数**
```c
int a[][8] = {1,2,4,3,6,7,98,5,6,87,9,5,3,45,7,98,6,4,3};
int num = 0;
num = sizeof(a)/sizeof(int);
//等价于：num = sizeof(a)/sizeof(a[0][0]);
```


3.**数组指针** 
```c
#include<stdio.h>
#include<string.h>

int main(void)
{
	
	int a[][5] = { 1,2,4,3,6,7,98,5,6,87,9,5,3,45,7,98,6,4,3 };

	//直接定义数组指针变量

	//定义数组指针变量,这个指针变量应该指向一维数组的整个数组的首地址

	int(*p)[5];
	//错误定义：p = &a;  这是整个二维数组的首地址
	p = a;//表示第0行首地址，每次跳4*5个字节，这里p等价于二维数组名


	//先定义数组指针类型，然后定义变量
	typedef int(*P)[10];
	P p;
	p = a;




	//测二维数组的函数和列数
	int num1 = sizeof(a) / sizeof(a[0]);//获取行数
	int num2 = sizeof(a[0]) / sizeof(a[0][0]);//获取列数



}
```

3.**数组指针做形参**

```c
#include<stdio.h>
#include<string.h>


void printarray(int a[][4])//第二维为指针+1 的步长
//等价于 void printarray(int a[3][4])
//等价于 void printarray(int (*a)[4])//数组做形参则退化成指针，注意每一次跳的步长是不同的
{
	int i = 0;
	int j = 0;
	for (i = 0; i < 3; i++)
	{
		for (j = 0; j < 4; j++)
		{
			printf("%d  ", a[i][j]);
		}
		printf("\n");
	}
	printf("\n");
}

int main(void)
{

	int a[][4] = { 1,2,3,4,5,6,7,8,9,10,11,12 };

	printarray(a);

	system("pause");
	return 0;

}


```

程序运行结果：
`1  2  3  4`
`5  6  7  8`
`9  10  11  12`


## 作业

### 作业一： 程序有点问题
```c
#include<stdio.h>
#include<string.h>


#define NUM(a) (sizeof(a)/sizeof(*a))

/*
函数功能：找到数组中指定字符串的位置

参数说明：
table ：字符串数组（指针数组）首地址
size ： 数组元素个数
key : 匹配字符串，如："do"
pos :匹配字符串在数组中的位置，如果“do”在keywords[]中的位置为4

返回值：
成功：0
失败：非0

*/


int  searchKeyTable(const char* table[] ,const int size,const char *key,int *pos)
{
	if (table == NULL || key == NULL || pos == NULL)
	{
		return -1;
	}

	int i = 0; 
	int index = -1;  //因为可能正好为0，所以这里不能赋值为0，就随便选一个负数即可
	for (i = 0; i < size; i++)
	{
		if (strcmp(table[i], key) == 0)
		{
			index = i; 
			break;
		}

	}
	if (index == -1)//没有匹配的字符串的时候
	{
		return -2;

	}

	*pos = index + 1;

	return 0;
}

int main(void)
{
	const char* keywords[] = { "while","case","static","do" };

	int pos = 0;
	int ret = 0;

	ret = searchKeyTable(keywords, NUM(keywords), "do", &pos);

	if (ret != 0)
	{
		printf("searchKeyTable err : %d \n", ret);
		return ret;
	}

	printf("%s located in keyword is : %d\n", pos);



	system("pause");
	return 0;
}

```




### 作业二：
```c

#include<stdio.h>
#include<string.h>

int main()
{
	int ret = 0;//这是表示错误码
	char *p1[] = { "aa","ccccccc","bbbbb" };
	char buf2[][30] = { "111111","3333333","222222" };
	char **p3 = NULL;
	int len1, len2, len3, i = 0;

	len1 = sizeof(p1) / sizeof(*p1);
	len2 = sizeof(buf2) / sizeof(buf2[0]);

	/*
	功能：
	1.把指针数组p1的字符串取出来
	2.把二维数组buf2的字符取出来
	3.上面的字符串放在p3.p3是在堆区分配的二维内存
	4.对p3中字符串进行排序，通过strcmp（）进行排序

	参数：
	p1:指针数组首地址，char *p1[] = { "aa","ccccccc","bbbbb" };
	len1 :p1元素个数
	buf2:二维数组首元素地址，char buf2[][30] = { "111111","3333333","222222" };
	len2:buf2字符串的行数
	p3:二级指针的地址，需要在函数内分配二维内存，保存p1和buf2的字符串，还需要排序
	len3：保存p3中字符串的个数

	返回值：

	*/

	ret = sort(p1, len1, buf2, len2, &p3, &len3);


	//释放p3所指向内存
	//在函数内部把p3的值赋值为NULL

	free_buf(&p3);



	system("pause");
	return 0;




}

```
