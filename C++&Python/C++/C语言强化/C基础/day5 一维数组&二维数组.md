---
date:'2018-9-9'
nandian:循环的嵌套
date:`2018-12-18`复习
---

# day5
@toc

==笔记的day3和day4本身即为空缺==

## 一、一维数组

**1.定义数组**

```c
#include<stdio.h>

int main()
{
  int array[10];  //定义了一个数组，数组名为array，里面有十个元素,从0-9；
 
  array[0] = 20;
  array[9] = 100;

  printf("%d\n",array[0]);
  printf("%d\n",sizeof(array));//一个整数四个字节，所以这里显示是40
  
  return 0;
}
```
程序输出结果：
`20`
`40`

数组本质就是：在内存中就是一个连续的空间；而且每个元素的类型是一样的；

---

**2.连续的赋值操作**

```c
#include<stdio.h>
int main()
{
  int i;
  int array[10];
  for(i = 0;i < 10;i++)
  {
    array[i] = i;  
    printf("array[%d] = %d\n",i,array[i]);
  }
  return 0;
}
```

程序输出结果为：
`array[0] = 0`
`array[1] = 1`
`array[2] = 2`
`array[3] = 3`
`array[4] = 4`
`array[5] = 5`
`array[6] = 6`
`array[7] = 7`
`array[8] = 8`
`array[9] = 9`

---

**3.数组的初始化**


```c
#include<stdio.h>
int main()
{

//数组初始化的三种方式(挨个赋值，部分赋值，统一赋值)
  
  //1.全部进行赋值
  int array[10] = {1,1,3,5,5,6,9,6,8,5};

  //2.定义一个数组的同时进行赋值初始化,这是部分初始化
  int array[10] = {100,2,3,3,4,9};

  //3-1.将所有元素置为0
  int array[10] = {0};  
  
  //3-2.另一种置为0 的方法
  int i;
  for(i = 0; i < 10; i++)
 {
   array[i] = 0;
 }



  int i;
  for(i = 0;i < 10;i++)
  {
    printf("array[%d] = %d\n",i,array[i]);
  }
  return 0;
}

```

---
**4.求数组中最大元素的值**

将数组中的第一个元素默认设置成最大的元素，然后让各个元素进行比较；
```c
#include<stdio.h>
int main()
{
  int array[5] = {2,3,5,6,7};
  int max = array[0];
  
  
  int i;
  for(i = 1;i < 5;i++) 
  //等效于：for(i = 1; i < sizeof(array)/sizeof(int);i++)
  //想找最大的先遍历一遍,因为第一的元素默认设置成最大值，所以这里的循环从1开始；
  {
    if(max < array[i])
     {
        max = array[i];
     }
  }

  printf("max = %d\n",max);

return 0;

}
```
程序运行结果：`max = 7`

---

**5.求数组中的最小值，并求出最小值的编号**
```c
#include<stdio.h>
int main()
{
  int array[5] = {6,3,5,6,7};
  int min = array[0];
  int index = 0;//在没有遍历数组之前，默认数组的第0号元素就是最小的元素；所以默认的地址也是0；
  
  int i;
  for(i = 1;i < 5;i++) //想找最小的先遍历一遍
    {
      if(min > array[i])
       {
          index = i;
          min = array[i];
       }
    }

  printf("min = %d,index = %d\n",min,index);

return 0;

}


```
程序输出结果：`min = 3,index = 1`

---

**6.求数组元素和**

```c
#include<stdio.h>
int main()
{
  int array[10] = {1,2,3,4,5,6,7,8,9,0};
  int i;
  int sum = 0;
  for(i = 0; i < 10; i++)
  {
    sum += array[i];
  }

  printf("sum = %d\n",sum);

  return 0;

}
```
程序运行结果:`sum = 45`

---

**7.打印数组中大于平均值的所有的值与其下标**
```c
#include<stdio.h>

int  main()
{
  int  array[10]  =  {1,2,3,4,5,6,7,8,9,0};
  
  int  i;
  float  aver;
  int  sum  =  0;
  int  index  =  0;
  for(i  =  0;  i  < sizeof(array)/sizeof(int);i++)
    {
      sum  +=array[i];
    }

  aver  =  sum/(float)(sizeof(array)/sizeof(int));
  printf("the aver of the array is = %f\n",aver);
 

  printf("The  following  elements  are  greater  than  the  average  value  of  the  array.  The  addresses  are:\n");


  for(i  =  0;i  <  10;i++)
  {
    if(array[i]  >aver)
      {
        index  =  i;
        printf("array[%d]  =  %d,index  =  %d\n",i,array[i],index);
      }
  }

  return 0;
}
```

程序输出的结果为：
`the aver of the array is = 4.500000`
`The  following  elements  are  greater  than  the  average  value  of  the  array.  The  addresses  are:`
`array[4]  =  5,index  =  4`
`array[5]  =  6,index  =  5`
`array[6]  =  7,index  =  6`
`array[7]  =  8,index  =  7`
`array[8]  =  9,index  =  8`

---

**8.数组逆置**
- 实现单个元素的互换
```c
#include<stdio.h>

int  main()
{
  int  array[5]  =  {1,2,3,4,5};

//部分交换

  int  i;
  int  tmp  =  array[0];
  array[0]  =  array[1];
  array[1]  =  tmp;

  for(i  =  0;i < 5;i++)
  {
    printf("array[%d]=%d\n",i,array[i]);
  }

  return  0;
}

```
程序运行结果为：
`array[0]=2`
`array[1]=1`
`array[2]=3`
`array[3]=4`
`array[4]=5`

- 实现数组的前半部分和后半部分的互换
  - 法一：直接换元素
 ==注意这里的是i 和 个数-1-i进行交换==
```c
#include<stdio.h>

int  main()

{
  int  array[10]  =  {1,2,3,4,5,6,7,8,9,0};
  int  i;
  for(i  =  0;  i  <=  4;  i++)
  {
    int  tmp  =  array[i];
    array[i]  =  array[9-i];
    array[9-i]  =  tmp;
  }

  for(i  =  0;i  <  10;i++)
    {
      printf("array[%d]=%d\n",i,array[i]);
    }

  return  0;

}
```
程序运行结果为：
`array[0]=0`
`array[1]=9`
`array[2]=8`
`array[3]=7`
`array[4]=6`
`array[5]=5`
`array[6]=4`
`array[7]=3`
`array[8]=2`
`array[9]=1`

  - ==法二：通过换下标==
```c
#include<stdio.h>

int  main()
{
  int  array[10]  =  {1,2,3,4,5,6,7,8,9,0};
  int  i;
  int num = sizeof(array)/sizeof(int);

  int  min  =  0;//数组的最小下标
  int  max  =  num -1;//数组的最大下标

  while(min < max)
   {
      int  tmp  =  array[min];
      array[min]  =  array[max];
      array[max]  =tmp;
      min++;
      max--;
   }

  for(i  =  0;i  <  num;i++)
  {
    printf("array[%d]=%d\n",i,array[i]);
  }

  return  0;

}

```
程序运行结果为：
`array[0]=0`
`array[1]=9`
`array[2]=8`
`array[3]=7`
`array[4]=6`
`array[5]=5`
`array[6]=4`
`array[7]=3`
`array[8]=2`
`array[9]=1`

---

==**9.水仙花数**==
查找100都1000之间的水仙花数；
```c
#include<stdio.h>

int  main()
{
  int  i;
  for(i  =  100;  i  <  1000;i++)
    {
      int  i1  =  i%10;//求个位上的数字
      int  i2  =  i/10%10;//求十位上的数字
      //或者为：int i2 = i%100/10
      int  i3  =  i/100;//求百位上数字
      
      if(i1*i1*i1  +  i2*i2*i2  +  i3*i3*i3  ==  i)
        {
          printf("%d\n",i);
        }
   }

  return  0;

}
```
程序运行结果；
`153`
`370`
`371`
`407`


---


**10.求一个数组中所有奇数元素之和**
```c

#include<stdio.h>
int  main()
{
	int  array[10] = { 1,3,4,7,8,0,9 };

	int  i;

	int  sum = 0;

	printf("The  odd  numbers  in  the  array  are  as  follows:\n");

	for (i = 0; i < 10; i++)
	{

		if (array[i] % 2 != 0)

			//if(array[i]%  2  ==  1)
		{
			sum += array[i];

			printf("array[%d]  =  %d\n", i, array[i]);
		}
	}

	printf("sum  =  %d\n", sum);

	return  0;
}
```

**11.求3-100中的素数**

注：素数：除了1和本身均不能被任何数整除；(也就是相当于不能被2到i-1之间的任何数整除)
```c
#include<stdio.h>
#include"stdlib.h"
int  main()
{
	int  i;
	int  j;

	for (i = 3; i < 101; i++)
	{
		int  status = 0;

		for (j = 2; j < i; j++)
		{
			if (i%j == 0)
			{
				status = 1;
				break;
			}
	    }

		if (status == 0)
		{
			printf("%d\n", i);
	
		}
}

	system("pause");
	return  0;

}
```
![素数]($resource/%E7%B4%A0%E6%95%B0.png)













# 二维数组
可以近似理解为表格；

**1.二维数组的定义**

```c
#include<stdio.h>

int main()
{

//先定义后赋值
  int array[2][3];//定义了一个二维数组，相当于有两个array[3] 
  array[0][0] = 0;//给第0行第0列赋值
  array[1][2] = 0;//给第1行第2列赋值

//定义的同时进行赋值
  int a[3][4] = {{1,2,3,4,},{5,6,7,8},{9,10,11,12}};
  int a[3][4] = {0};//将二维数组中所有值全部置0
  
  return 0;
}
```

**2.数组大小**

```c
int main()
{
  int array[2][3] = {{1,2,3},{4,5,6}};
  printf("%d\n",sizeof(array));//整个数组的大小为24
  printf("%d\n",sizeof(array[0])); //相当于第0行的大小，为12
  printf("%d\n",sizeof(array[0][0]));//相当于第0行第0列元素的大小，为4
 return 0;
}
```

**3.数组的值输出**
```c
#include<stdio.h>
#include"stdlib.h"

int  main()
{
	int  array[2][3] = { {1,2,3},{4,5,6} };
	int  i, j;
	for (i = 0; i < 2; i++)
	{
		for (j = 0; j < 3; j++)
		{
			printf("array[%d][%d]=  %d\n", i, j, array[i][j]);
		}
	}
	return  0;
}
```


**4.求一个二维数组的每行每列的和**

==注意printf的位置==
每行的和：
```c
int main()
{
  int array[3][5] = {{1,3,4,5,9},{4,9,5,0,8},{9,3,4,23,4}};
  int i,j;
  for(i = 0; i <3;i++)
  {
    int sum = 0;
    for(j = 0;j < 5;j++)
    {
      sum += array[i][j];
    }
    printf("%d\n",sum);
  }
  return 0;
}
```

每列的和：

```c
int main()
{
   int array[3][5] = {{1,3,4,5,9},{4,9,5,0,8},{9,3,4,23,4}};
   int i,j;
   for(j = 0; j < 5; j++);
   {
     int sum = 0;
     for(i = 0; i < 3;i++)
     {
       sum +=array[i][j];
     }
   printf("%d\n",array[i][j]);
   }
  return 0;
}
```


**5.冒泡排序**
```c
int main()
{
  int array[10] = {1,4,8,0,9,34,98,100,5,3};
  int i;
  int j;
  for(i = 0;i < 10;i++)
    {
      for(j = 1; j < 10-i;j++)
      {
        if(array[j-1] >array[j])
          {
            int tmp;
            tmp = array[j];
            array[j] = array[j-1];
            array[j-1] = tmp;
          }
      }
    }
    //打印程序
    for(i = 0;i < 10;i++)
    {
    printf("array[%d] = %d\n",i,array[i]);
    }
  return 0;
}
```








