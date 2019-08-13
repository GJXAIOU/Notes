# 排序算法


```
以下程序在win10 X64位操作系统，使用VS2017运行验证可行
```
排序是非常重要且很常用的一种操作，有冒泡排序、选择排序、插入排序、希尔排序、快速排序、堆排序等多种方法。




## 实例1 冒泡法排序

**1.前言：** 数组中有N个整数，用冒泡法将它们从小到大（或从大到小）排序。冒泡法较慢；

**2.算法步骤：**
冒泡法效率是最低的，但算法简单：
- 从第一个数开始，相邻两个数两两比较，将大的交换到后面，然后继续比较第2、3个数…..,遍历结束之后最小的数就在最前面了；

- 将最前面的最小的数排除在外，其余数重复步骤1。

- 重复步骤2，直到所有数都排好为止。


**3.示例程序：**
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

---

## 实例2 选择法排序
**1.前言：** 数组中有N个整数，用选择法将它们从小到大排序。其速度比冒泡法快；

**2.算法步骤：**
- 找出一个最小数，一般默认数组中的第一个元素为最小的元素，然后遍历后面的所有的元素，分别与这个最小的元素进行比较，如果比最小的元素还小就交换位置，一轮之后数组中最小的元素排到了最前面；

- 在剩下的数里面，再找一个最小的，交换到剩下数的最前面；

- 重复步骤2 ，直到所有数都已排好；


3.**示例程序**
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

//打印数组中的元素
void print_array(int *array, int num)
{
	int i = 0;
	for (i = 0; i < num; i++)
	{
		printf("%d \n", array[i]);
	}
}


//选择法进行排序

//如果数组作为函数参数，则数组形参退化为指针
void sort_array(int *array, int num)//等价于：void sort_array(int array[],int num)
{
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



	printf("after = \n");
	sort_array(array, num);
	print_array(array, num);

	system("pause");
	return 0;
}
```
程序运行结果：
`after =`
`0`
`1`
`2`
`3`
`4`
`4`
`5`
`6`
`9`


---

## 实例3 插入排序
**1.前言：** 数组中有N个整数，用插入排序实现它们由小到大的排列。算法的效率介于冒泡法和选择法之间

**2.算法过程：** 其基本思想是：将数组分成两个区：前面是已排序的区域（有序区），后面是没有排序的区域（无序区）。每次都从无序区中取第一个数插入到有序区中适当位置，直到所有数据插入完毕为止。例如：待排序的数据存放在数组A[0, 1, ...N-1]中，未排序前，A[0]自己是一个有序区，A[1, 2, ...N-1]是无序区。程序必须从i = 1开始，直到i = N-1为止，每次将A[i]插入到有序区中，而为了找到这个适当位置，需要将A[i]与有序区的数据进行比较。

- 基本的插入排序：

  首先在有序区A[0,1,...i-1]中查找A[i]应该插入的位置k（0 <= k <= i-1），然后将A[k,k+1,...i-1]中的数据各自后移一个位置，腾出位置k插入A[i]。
若有序区所有数据均小于A[i]时，A[i]就应该在原位置不变，不需要插入。

- 改进后的插入排序：

  将待插入的数据A[i]自右至左依次与有序区的数据A[i-1,i-2,...0]进行比较，若A[i]小于某数据A[j]，则A[j]后移一个位置，继续与前面的数据比较......直到遇到比A[i]小的数据或前面已没有数据，则插入位置确定。若碰到一个数据A[j]比A[i]小，则A[i]应插入到位置j+1。若A[i-1]比A[i]小，则A[i]位置不变。若所有数据都比A[i]大，则A[i]应插入到位置0。

下面是改进后插入排序的代码：
```c

#include<stdio.h>
#include<string.h>
#include<stdlib.h>


void print_array(int array[], int num)
{
	int i = 0;
	for (i = 0; i <= num - 1; i++)
	{
		printf("%d  ", array[i]);
	}
	return 0;
}
void sort_array(int array[], int num)
{
	int i = 0;
	int tmp = 0;
	int j = 0;
	for (i = 1; i <= num - 1; i++)
	{
		tmp = array[i];
		for (j = i - 1; array[j] > tmp && j >= 0; j--)
		{
			array[j + 1] = array[j];
		}
		
		array[j + 1] = tmp;
			
	}
}

int main(void)
{
	int  array[] = { 1,4,6,9,0,4,3,5,2 };

	int num = 0;
	num = sizeof(array) / sizeof(int);

	sort_array(array, num);
	print_array(array, num);

	printf("\n");
	system("pause");
	return 0;

}
```
程序运行结果：
`0  1  2  3  4  4  5  6  9`






