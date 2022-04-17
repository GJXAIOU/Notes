---
1.概述：学习要求，学习标准
2.数据类型、变量
3.内存四区（栈、堆、全局、代码区）
4.说明：本系列代码使用VS2017进行测试运行，代码位置为：
E:\Program\C++\C++\practice\programs\Cenhance\Cenhance_1\Cenhance_1
---

# day01 数据类型和内存四区


## 一、数据类型


1.**学习历程**

![学习历程]($resource/%E5%AD%A6%E4%B9%A0%E5%8E%86%E7%A8%8B.png)

2.**如何看懂带算法的程序**

- 看懂流程
- 每个语句的功能
- 试数
- 调试
- 模仿改
- 不看代码自己写


----
3.**排序方法：**
- **冒泡排序**
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>


//将数组中的元素进行排序
void sort_array(int array[], int num)
{
	int i;
	int j;
	for (i = 0; i < num; i++)//这里等价于：for(i = 0; i < num -1; i++)
	{
		for (j = 1; j < num - i; j++)//这里等价于：for(j = 0; i < num - i -1; j++)
		{
			if (array[j - 1] > array[j])
			{
				int tmp;
				tmp = array[j];
				array[j] = array[j - 1];
				array[j - 1] = tmp;
			}
		}
	}
}



//将排序之后的数组打印出来

void print_array(int array[], int num)
{
	int i = 0;
	for (i = 0; i < num; i++)
	{
		printf("array[%d] = %d\n", i, array[i]);
	}

}


int main()
{
	int array[] = { 1,4,8,0,9,34,98,100,5,3 };
	int num = sizeof(array) / sizeof(int);//求出数组元素的长度

	sort_array(array, num);
	print_array(array, num);
	
	system("pause");
	return 0;
}
```
程序运行结果：
`array[0] = 0`
`array[1] = 1`
`array[2] = 3`
`array[3] = 4`
`array[4] = 5`
`array[5] = 8`
`array[6] = 9`
`array[7] = 34`
`array[8] = 98`
`array[9] = 100`


- **选择法排序**

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main(void)
{
	int array[] = { 1,4,6,9,0,4,3,5,2 };
	int num = 0;
	num = sizeof(array) / sizeof(array[0]);//求数组元素个数,只能使用在定义的时候没有确定数组长度的情况




	//将排序前的数组打印出来进行对比
	int i = 0;
	for (i = 0; i < num; i++)
	{
		printf("before = %d \n", array[i]);
	}
	printf("------------------\n");



	//选择法进行排序

	int j = 0;
	int tmp = 0;
	for (i = 0; i < num - 1; i++)
	{
		for (j = i + 1; j < num; j++)
		{
			if (array[i] > array[j])
			{
				tmp = array[i];
				array[i] = array[j];
				array[j] = tmp;
			}
		}
	}
	for (i = 0; i < num; i++)
	{
		printf("after = %d \n", array[i]);
	}

	system("pause");
	return 0;
}
```
程序运行结果；
`before = 1`
`before = 4`
`before = 6`
`before = 9`
`before = 0`
`before = 4`
`before = 3`
`before = 5`
`before = 2`

------------------
`after = 0`
`after = 1`
`after = 2`
`after = 3`
`after = 4`
`after = 4`
`after = 5`
`after = 6`
`after = 9`

---

4.**将上面的程序进行函数封装**

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>


void printf_array(int *array, int num)
{
	int i = 0;
	for (i = 0; i < num; i++)
	{
		printf("%d \n", array[i]);
	}
}


//如果数组作为函数参数，则数组形参退化为指针
//下面的第一个参数可以是*array，或者为：array[],或者为：array[1]，或者array[10]等等
void sort_array(int *array, int num)
{
	//选择法进行排序
	int i = 0;
	int j = 0;
	int tmp = 0;
	for (i = 0; i < num - 1; i++)
	{
		for (j = i + 1; j < num; j++)
		{
			if (array[i] > array[j])
			{
				tmp = array[i];
				array[i] = array[j];
				array[j] = tmp;
			}
		}
	}

}


int main(void)
{
	int array[] = { 1,4,6,9,0,4,3,5,2 };
	int num = 0;
	num = sizeof(array) / sizeof(array[0]);

	printf("before = \n");
	printf_array(array, num);
	printf("------------------\n");
	printf("after = \n");
	sort_array(array, num);
	printf_array(array, num);
	system("pause");
	return 0;
}
```
程序运行结果同上；


----

5.**void 无类型**

- 函数参数为空，定义函数时，可以用void修饰：例如：`int fun(void)`
- 函数没有返回值：`void fun（int a )`
- 不能定义void类型的普通变量
- 可以使用定义void *变量 ： `void *p`,其中在32位系统中为4个字节，在64位系统中占64字节
- 数据类型本质：固定内存块大小别名
- void *p  万能指针，可以作为函数返回值，函数参数

**例子程序:**  void *p的万能指针，在实际使用中一定要变换成实际类型的指针

```c
#include<stdio.h>
#include<string.h>

int  main(void)
{
  void  *p  =  0x1222;//因为指针固定占4个字节，所以定义的时候什么数据类型都行
  char  buf[100]  =  "djfkdjhf";
  p  =  buf;
  printf("  p  =  %s\n",(char  *)p);//实际使用时候转换成需要的类型的指针
 
  
  int  a[100]  =  {1,2,3,4};
  p =  a;
  int  i  =  0;
  for(i  =  0;  i  <  4;  i++)
    {
      printf("%d\n",*((int  *)p  +  i));//实际使用的时候还是转换成需要的类型的指针
    }

  return  0;

}

```
程序运行结果：
`p = djfkdjhf`
`1`
`2`
`3`
`4`



- 指针没有指向变量的话会造成没法分配内存空间
```c
#include<stdio.h>
#include<string.h>

int  main(void)
{
  char  *p  =  NULL;
  char  str[100]  =  {0};
  p  =  str;
  strcpy(p,"nihao");//拷贝是将这个字符串拷贝到该指针指向的内存区域，而不是给这个指针
  printf("%s",p);

  return  0;

}

```

如果没有定义`char str[100]`就会导致指针P没有被分配空间，造成程序错误


---

6.**程序分文件**

- **要求：** 按照函数的功能分文件
- 分文件时候，防止头文件重复包含，只需要在所有的头文件（.h文件）里面最前面加上：`#pragma once`
- 让C语言代码可以在C++的编译器中编译运行，需要加上以下代码：
```c
#ifdef_cplusplus
extern "C" {
#endif

//函数的声明

#ifdef_cplusplus
}
#endif
```

7.**数据类型**
 - 类型的本质：固定内存大小的别名
 - 数据类型的作用：编译器预算对象（变量）分配的内存空间大小，只有当变量定义的时候才真正的将这个空间分配出去；
 - 数据类型可以通过typedef其别名



## 变量


## 内存四区

此程序说明内存四区划分

```c
#include<stdio.h>
#include<stdlib.h>

char *get_str1()
{
  int *str1 = "abd";
  return str1;
}

char *get_str2()
{
  int *str2 = "abd";
  return str2;
}



int main(void)
{
  char *p = NULL;
  char *q = NULL;

  p = get_str1();
  q = get_str2();

  printf("p = %s,&p = %d\n",p ,p);
  printf("q = %s ,&q = %d\n",q,q);
  system("pause");
  return 0;

}

```
程序运行结果：

![内存四区调试结果]($resource/%E5%86%85%E5%AD%98%E5%9B%9B%E5%8C%BA%E8%B0%83%E8%AF%95%E7%BB%93%E6%9E%9C.png)

虽然p和q分别是两个函数的返回值，但是从结果可以看出他们指向的地址都是同一个
**解析本函数的内存四区图：**

![程序内存四区分析]($resource/%E7%A8%8B%E5%BA%8F%E5%86%85%E5%AD%98%E5%9B%9B%E5%8C%BA%E5%88%86%E6%9E%90.png)

首先在栈中划定四个字节的区域放置指针p(因为指针在32位中都是四个字节)，因为值为NULL,所以并没有值；同时指针q也是这样；
然后调用`get_str1`函数，在函数中定义的指针`str1`（在另一块栈区）指向字符串“abd”（存在全局区）,函数返回值为str1，即为该字符串的地址，而在main函数中该地址赋值给了指针p（调用完之后指针str1就消失了）,因此指针p指向了该字符串，最后打印输出
同样因为该字符串并没有改变，因此系统不会将他存储两遍，所以地址是一样的；



**分析栈区**
以上的程序容易出现乱码的情况，因为栈中使用结束之后就会自动释放，释放之后无法确定系统中存储的是什么元素


**分析堆区：**

```c
#include<stdio.h>
#include<string.h>

char *get_str()
{
  char *tmp = (char *)malloc(100);
  if(tmp = NULL)
    {
      return NULL;
    }

  strcpy(tmp ,"abcdefghijklmnopqrst");
  return tmp;
}

int main(void)
{
    char buf[100] = {0};
    char *p = NULL;
    p = get_str();
    if(p != NULL)
      {
        printf("p = %s\n",p);
        free(p);//释放掉这个堆区，但是释放本质上并不是清空堆区中的内容，而只是取消p对于这个内存区域的使用权，本质上这个字符串仍然存储在堆区中；
        p =NULL;//将这块堆区置零，防止下次使用的时候数据有问题；
      }

return 0 ;
    

}
```
![本程序对应内存四区示意图 2]($resource/%E6%9C%AC%E7%A8%8B%E5%BA%8F%E5%AF%B9%E5%BA%94%E5%86%85%E5%AD%98%E5%9B%9B%E5%8C%BA%E7%A4%BA%E6%84%8F%E5%9B%BE%202.png)



**函数调用模型：**

![函数调用模型]($resource/%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8%E6%A8%A1%E5%9E%8B.png)


**栈的生长方向：**

![栈的生长方向]($resource/%E6%A0%88%E7%9A%84%E7%94%9F%E9%95%BF%E6%96%B9%E5%90%91.png)








