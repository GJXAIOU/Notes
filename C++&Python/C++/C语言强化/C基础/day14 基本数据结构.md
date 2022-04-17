---
date:`2018-10-15`
---

# day14 基本数据结构


## 冒泡排序与选择排序

- **冒泡排序法：**
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

void swap(int *a,int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

void bubble(int *array,int n)
{
  int i;
  int j;
  for(i = 0; i < n; i++)
  {
    for(j = 1; j < n-i; j++)
      {
        if(array[j -1] > array[j])
          {
            swap(&array[j-1] ,&array[j]);
          }    
      }
  }
}

void print_array(int *array,int n)
{
  int i;
  for(i = 0; i < n; i++)
    {
      printf("%d\n",array[i]);
    }
}


int main()
{
  int array[10] = {1,3,4,7,43,98,4,99,3,8};
  bubble(array,10);
  print_array(array,10);
  return 0;

}
```

- **选择排序法：**
在一个集合中找最小的那个数，放在最前面，一直这样找下去
```c
#include<stdio.h>
#include<string.h>

//查找一个数组中的最小值（每次查找的循环次数依次递减）

int minkey(int *array,int low,int high)
//第一个参数为一个数组，第二个参数为数组的开始下标，第三个参数为数组的终止下标
//函数返回值为最小元素的下标

  {
    int min = low;
    int key = array[low];//在没有查找最小元素之前，第一个元素是最小的
    int i;
    for(i = low +1; i < high;i++)
      {
        if(key > array[i])
          {
            key = array[i];
            min = i;
          }
      }
    return min;
  }


//交换元素
void swap(int *a,int *b)
{
    int tmp = *a;
    *a = *b;
    *b = tmp;
}


//将数组中的值进行排序

void select(int *array, int n)
{
    int i;
    for(i = 0;i < n; i++)
      {
        int j = minkey(array,i,n);
        if(i != j)//即范围内的第一个成员不是最小的
          {
            swap(&array[i],&array[j]);
          }
      }
}

void print_array(int *array,int n)
{
  int i;
  for(i = 0; i < n; i++)
    {
      printf("%d\n",array[i]);
    }
}


int main(void)
{
  int array[10] = {1,2,4,32,87,3,23,98,5,67};
  select(array,10);
  print_array(array,10);
  return 0;
}
```
程序允许的结果：
`1`
`2`
`3`
`4`
`5`
`23`
`32`
`67`
`87`
`98`
注:表格中的数据和程序使用的数据不一致，只是算法一致
![表格表示]($resource/%E8%A1%A8%E6%A0%BC%E8%A1%A8%E7%A4%BA.png)


- **快速排序**==需要测试==
没有程序，可以从视频中大概找找
```c
#include<stdio.h>
#include<string.h>


void q_sort(int *array,int low, int high)//快速排序，low是排序范围的下标，high是上标
{
  if(low < high)
    {
        int povitloc = partition(array,low,high);
 //定义一个变量，指向轴点的位置；
 //partition得到轴点的位置，并且已经根据轴点将数组内容排序了
        q_sort(array,povitloc + 1,high);
      //在轴点的下半部分递归快速
        q_sort(array,low,povitloc -1);
     //在轴点的上半部分递归快速
    }
}


void quick_sort(int *array,int n)
{
  q_sort(array,0,n-1);
}


void print_array(int *array,int n)
{
  int i;
  for(i = 0; i < n; i++)
    {
      printf("%d\n",array[i]);
    }
}


int main(void)
{
  int array[10] = {1,2,4,32,87,3,23,98,5,67};
  quick_sort(array,10);
  print_array(array,10);
  return 0;

}

```

程序有问题，附上其他的链接；
[快速排序](https://blog.csdn.net/jeryjeryjery/article/details/52894756)
![快速排序]($resource/%E5%BF%AB%E9%80%9F%E6%8E%92%E5%BA%8F.png)

**CSDN上快速排序算法：**
- 法一；定轴法；
（一）定轴法:
1.备份对轴(首记录，即第一个元素)

2.取两个指针left和right，初始值分别是序列的第二个元素和最后一个元素,并且left<=right

3.移动两个指针
*从right所指的位置向左搜索，找到第一个小于轴的元素

*从left所指的位置向右搜索，找到第一个大于轴的元素
*找到后如果left<right，那么就交换两个位置的值

4.重复上述过程，直到left>right

5.把轴放到right的位置，并且将right位置的值放到第一位

6.分别将right位置左边的和右边的进行上述的递归

C++代码实现如下：

```c

void quickSort(int* A,int first,int last)
{        //数组A,first是第一个元素下标，last是最后一个元素下标
	if(last<=first)     //到了长度小于1这种情况已经是有序列了
		return;
 
	int pivot=A[first];
	int left=first+1;       //left等于第二个元素
	int right=last;    
        int temp;
	while(left<=right){
		while(A[right]>pivot&&right>=left)//找到一个比first小的,但必须保证left值小于等于right值
			right--;
 
		while(A[left]<pivot&&left<=right) //找到一个比first大的,但得保证left值小于等于right值
			left++;
 
		if(left>=right)   //说明已经是相对有序序列，无需交换
			break;
 
		temp=A[left];                     //交换位置
		A[left]=A[right];
		A[right]=temp;
		left++,right--;                   //相应的进一位
	}
	A[first]=A[right];     //因为right一定是停在从右到左第一个小于first的数上，交换之后，
	//依然能保证first值左边的比first小，右边的比first大
	A[right]=pivot;
 
	quickSort(A,first,right-1);               //左半部分
	quickSort(A,left,last);                   //右半部分


```

二）挖坑法:

1.备份轴记录

2.取两个指针low和high，初始值就是序列的两端下标，保证low<=high

3.移动两个指针

*从high向左找到第一个小于轴的元素, 放在low的位置

*从low向右找到第一个大于轴的元素，放在high的位置

4.重复，直到low=high，

5.把轴放在low所指的位置

6.分别对low所指的位置的左边和右边进行上述的递归

C++实现代码如下：
```c
void quickSort(int s[], int l, int r)
{
    if(l<r)
    {      
        int low=l;           //左边第一个，因为第一个已经用pivot保存了
	    int high=r;          //右边
	    int pivot = s[l];     //第一个，已被保存
        while(low<high)       //当左小于右,当相等的时候会跳出循环
          {
            while(low<high&&s[high]>= pivot)  // 从右向左找第一个小于x的数
				high--; 
            if(low<high)                        
                s[low++] = s[high];
                    
            while(low<high&&s[low]<pivot)     // 从左向右找第一个大于等于x的数
                low++; 
            if(low<high)  
                s[high--] = s[low];
          }
        s[low]=pivot;        
        quickSort(s, l, low - 1);             //low左边递归调用
        quickSort(s, low + 1, r);             //low右边递归调用
    }
}


```
---


## 查找算法

- **顺序查找**

```c
int seq(int *array, int low, int high, ing key)
{
  int i;
  for(i = low; i < high; i++)
    {
      if(array[i] == key)
        {
          return i;
        }else
        {
          return -1;
        }
    }
}


int main()
{
  int array[10] = {32,43,68,43,98,5,67,2,4,3};
  printf("%d\n",seq(array,0,10,89));
  return 0;
}



```



- **二分查找**
**前提：** 是针对已经排好序的数组
**原理：** 每次取数组的中间位置元素，将要查找的数和该元素比较，如果相同则就是该元素，如果大于该元素，则要查找的元素在中间元素之后，反之在前面，然后将可能的范围再次一分为二，进行查找；
```c

#include<stdio.h>
#include<stdlib.h>
#include<string.h>


int bin(int *array,int low,int high,int key)
{
  while(low <= high)
    {
      int mid = (low + high)/2;
        if(key == array[mid])//中间切一刀，正好等于中间数
          {
            return mid;
          }else if(key > array[mid])//如果要找的数大于array[mid]，那么就在下半部分继续查找
          {
            low = mid +1;
          }else//如果要找的数小于array[mid]，那么就在上半部分继续查找
          {
            high = mid -1;
          }
    }
    return -1;//没有找到数据的话返回-1

}


int main()
{
  int array[10] = {32,43,68,43,98,5,67,2,4,3};

  quick_sort(array,10);//这里需要调用快速排序方法，暂时上面的代码有错误
  printf("%d\n",bin(array,0,10,32));
  return 0;

}


```

**以上程序的递归方法实现**
```c
int bin_rec(int *array,int low,int high,int key)
{
  if(low <= high)//递归终止条件
    {
      int mid = (low + high)/2;
        if(key == array[mid])
          {
            return mid;
          }else if(key > array[mid])
          {
            return bin_sec(array,mid +1,high);//在下半部分查找
          }else
          {
            return bin_sec(array,low,mid -1);//在上半部分查找
          }
    }
    return -1;//没有找到数据的话返回-1

}


int main()
{
  int array[10] = {32,43,68,43,98,5,67,2,4,3};
  quick_sort(array,10);//这里需要调用快速排序方法，暂时上面的代码有错误
  printf("%d\n",bin_rec(array,0,10,32));
  return 0;

}


```


## 四、单向链表的实现

引入：在一个数组中插入或者删除一个数据会造成同时移动大量的数据，

 
 1.**单向链表的定义：**


**链式存储：**
- 存储单元可以是连续的或者非连续的
- 链式存储中的元素存储时候除了存储本身信息还要存储一个知识其连接的后面的元素的位置；这两部分的数据成为**结点**；
- **数据域：** 一个结点中存储的数据元素
- **指针域：** 存储连接后面存储位置的域
- **链表：** n个结点的存储映像链接而成

整个链表必须从头结点开始进行，头结点的指针指向了下一个结点的位置，最后一个 结点的指针指向NULL

```c
#include<stdio.h>
#inlcude<stdlib.h>

//一个结点包含数据域和指针域
struct list
{
  int date;//数据域
  struct list *next;//指针域
};


//创建一个结点的函数：
struct list *create_list()
{
  return calloc(szieof(struct list),1);
}；






//循环遍历链表

void traverse(struct list *ls)
{
  struct list *p = ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
    {
      printf("%d\n",p -> date);
      p = p -> next;  //p指向他对应的下一个结点
    }
}




int main(void)
{
  //创建三个结点(可以通过函数)
  struct list *first = calloc(sizeof(struct list), 1);//在堆中间创建一个结点
  struct list *second = calloc(sizeof(struct list), 1);
  struct list *third = create_list();

  //将三个结点连接起来
  first -> next = second;
  second -> next = third;
  third -> next = NULL;

  //数据域进行赋值
  first -> date = 2;
  second -> date = 3;
  third ->date = 4;


  traverse(first);

  return 0;
}
```


2.**在链表中插入元素**
```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}



//在链表中插入元素

struct  list  *insert_list(struct  list  *ls,  int  n,  int  data)
{
  struct  list  *p  =  ls;
  while(p  &&  n--)
    {
      p  =  p  ->  next;
    }

  if(p  ==  NULL)
    {
      return  NULL;//n的位置大于链表节点数
    }

  struct  list  *node  =  create_list();//新建立一个节点
  node  ->  data  =  data;
  node  ->  next  =  p  ->  next;
  p  ->  next  =  node;
  return  node;
};



int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;
  insert_list(first,0,10);

//参数含义：链表头，指针位置，要插入的值

  traverse(first);
  return  0;

}
```
程序运行结果：相当于在第0个位置，即第一的元素的后面插入一个数字`10`

![插入元素的结果]($resource/%E6%8F%92%E5%85%A5%E5%85%83%E7%B4%A0%E7%9A%84%E7%BB%93%E6%9E%9C.png)


3、**在链表中删除指定位置的元素**
```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}



//在链表中指定位置删除元素

int delete_list(struct  list  *ls,  int  n)
{
  struct  list  *p  =  ls;
  while(p  &&  n--)
    {
      p  =  p  ->  next;
    }

  if(p  ==  NULL)
    {
      return  NULL;//n的位置大于链表节点数
    }

  struct  list  *tmp  =  p -> next;//将p ->next暂时存储
  p -> next = p -> next -> next;//跳过要删除的元素，要删除元素的前一个元素的下一跳指向删除元素的下一个位置
  free(tmp);
  return  0;//表示删除成功
};



int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 delete_list（first,2);
 //删除首元素之后的第二个元素


  traverse(first);
  return  0;

}
```

4、**返回链表中元素的个数：**

```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}




//返回链表中元素的个数
//相当于加上一个计数器
int count_list(struct list *ls)
{
  struct list *p = ls;
  int count = 0;

  while(p)
    {
      count ++;
      p = p -> next;
    }
    return count;

}



int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 printf("count = %d\n",count_list(first));
  printf("--------------\n");

  traverse(first);
  return  0;

}
```
程序运行结果：
![计数]($resource/%E8%AE%A1%E6%95%B0.png)




5.**清空链表，只保留首节点**

```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}



void clear_list(struct list *ls)//清空链表，只保留首节点
{
    struct list *p = ls -> next;//直接指向下一个结点，避免第一个结点被删除
    while(p)
      {
        struct list *tmp = p ->next;//暂存p的下一级指向的位置
        free(p);
        p = tmp;
      }
      ls -> next = NULL;//只有首节点，那么首节点的next也应该设置为NULL
}





int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 clear_list(first);

  traverse(first);
  return  0;

}



```
程序运行结果：只保留了首元素，清空了其他元素
![清空链表]($resource/%E6%B8%85%E7%A9%BA%E9%93%BE%E8%A1%A8.png)


6.**返回链表是否为空**

```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


int empty_list(struct list *ls)
{
    if(ls -> next)//如果链表不为空，因为如果这个链表是空的，这ls->next指向NULL；
      {
        return 0;
      }else
      return -1;

}




int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 empty_list(first);
  return  0;

}


```

7.**返回链表指定位置的节点**

```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


//返回链表指定位置的节点
struct list *locale_list(struct list *ls,int n)
{
   struct list *p = ls;
   while(p && n--)//因为只能从表头挨个循环查找
   {
       p = p -> next;
   }
   if(p == NULL)
     return NULL;

  return p;
}




int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 printf("%d\n",locale_list(first,2) -> data);
  return  0;

}


```
程序运行结果：
![返回链表指定位置的节点]($resource/%E8%BF%94%E5%9B%9E%E9%93%BE%E8%A1%A8%E6%8C%87%E5%AE%9A%E4%BD%8D%E7%BD%AE%E7%9A%84%E8%8A%82%E7%82%B9.png)


8.**返回数据域等于data的节点**
```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


//返回数据域等于data的节点
struct list *elem_locale(struct list *ls,int data)
{
   struct list *p = ls;
   while(p)
   {
       if(p -> data == data)
       return p;
       p = p -> next;
   }
  return NULL;//没有找到数据域等于data的节点
}




int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 printf("%d\n",elem_locale(first,3));
  return  0;

}

```

程序运行结果：~~暂不清楚~~


9、**返回数据域等于data的节点位置**

```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


//返回数据域等于data的节点位置
int elem_pos(struct list *ls,int data)
{
   int index = 0;
   struct list *p = ls;
   while(p)
   {
       index++;
       if(p -> data == data)
       return index;
       p = p -> next;
   }
  return NULL;//没有找到数据域等于data的节点
}




int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 printf("%d\n", elem_pos(first,3));
  return  0;

}

```
程序运行结果：返回data数据元素在链表中的位置；

![返回数据域等于data的节点位置]($resource/%E8%BF%94%E5%9B%9E%E6%95%B0%E6%8D%AE%E5%9F%9F%E7%AD%89%E4%BA%8Edata%E7%9A%84%E8%8A%82%E7%82%B9%E4%BD%8D%E7%BD%AE.png)


10.**得到链表最后一个结点**

==链表的最后一个结点指向NULL，但是不代表最后一个结点为NULL，只是 last -> next = NULL==

```c
#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


//返回链表的最后一个节点
struct list *last_list(struct list *ls)
{
   struct list *p = ls;
   while(p -> next)//当p的next为空时候，表示p为最后一个节点
   {
       p = p -> next;
   }
  return p;
}




int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;

//函数引用
 printf("%d\n", last_list(first) -> data);
  return  0;

}

```
程序输出结果；

![查找列表中最后一个节点]($resource/%E6%9F%A5%E6%89%BE%E5%88%97%E8%A1%A8%E4%B8%AD%E6%9C%80%E5%90%8E%E4%B8%80%E4%B8%AA%E8%8A%82%E7%82%B9.png)


11.**合并两个列表，结果放在list1中**

```c

#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


//返回链表的最后一个节点
struct list *last_list(struct list *ls)
{
   struct list *p = ls;
   while(p -> next)//当p的next为空时候，表示p为最后一个节点
   {
       p = p -> next;
   }
  return p;
}

struct  list  *insert_list(struct  list  *ls,  int  n,  int  data)
{
  struct  list  *p  =  ls;
  while(p  &&  n--)
    {
      p  =  p  ->  next;
    }

  if(p  ==  NULL)
    {
      return  NULL;//n的位置大于链表节点数
    }

  struct  list  *node  =  create_list();//新建立一个节点
  node  ->  data  =  data;
  node  ->  next  =  p  ->  next;
  p  ->  next  =  node;
  return  node;
};


//这里只合并链表的节点，删除第二个链表头

void merge_list(struct list *ls1, struct list *ls2)
{
  last_list(ls1) -> next = ls2 -> next;//这里调用上一个函数，获取第一个链表的最后一个节点
  free(ls2);//后面一个链表的链表头不要了

}





int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second  =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();

  //将三个结点连接起来

  first  ->  next  =  second;
  second  ->  next  =  third;
  third  ->  next  =  NULL;

  //数据域进行赋值

  first  ->  data  =  2;
  second  ->  data  =  3;
  third  ->data  =  4;


//快速创建链表first1

struct list *first1 = create_list();
int i;
for(i = 0; i < 10; i++)
  {
    insert_list(first1,0,i);
  }


merge_list(first,first1);
traverse(first);
  return  0;

}

```
程序运行结果：

![连接两个链表，结果放在第一个链表中]($resource/%E8%BF%9E%E6%8E%A5%E4%B8%A4%E4%B8%AA%E9%93%BE%E8%A1%A8%EF%BC%8C%E7%BB%93%E6%9E%9C%E6%94%BE%E5%9C%A8%E7%AC%AC%E4%B8%80%E4%B8%AA%E9%93%BE%E8%A1%A8%E4%B8%AD.png)

12.**单项链表的逆置**

```c

#include<stdio.h>
#include<stdlib.h>

//一个结点包含数据域和指针域

struct  list
{
   int  data;//数据域
   struct  list  *next;//指针域
};


//创建一个结点的函数：

struct  list  *create_list()
{
  return  calloc(sizeof(struct  list),1);
};



//循环遍历链表

void  traverse(struct  list  *ls)
{
  struct  list  *p  =  ls;
  while(p)//要保证最后一个结点指向NULL，否则循环无法结束
  {
  printf("%d\n",p  ->  data);
  p  =  p  ->  next;  //p指向他对应的下一个结点
  }
}


//返回链表的最后一个节点
struct list *last_list(struct list *ls)
{
   struct list *p = ls;
   while(p -> next)//当p的next为空时候，表示p为最后一个节点
   {
       p = p -> next;
   }
  return p;
}

struct  list  *insert_list(struct  list  *ls,  int  n,  int  data)
{
  struct  list  *p  =  ls;
  while(p  &&  n--)
    {
      p  =  p  ->  next;
    }

  if(p  ==  NULL)
    {
      return  NULL;//n的位置大于链表节点数
    }

  struct  list  *node  =  create_list();//新建立一个节点
  node  ->  data  =  data;
  node  ->  next  =  p  ->  next;
  p  ->  next  =  node;
  return  node;
};




void reverse(struct list *ls)
{
  if(ls -> next == NULL)
    return;//只有一个节点不需要逆置

  if(ls -> next -> next ==NULL)
    return;//只有两个节点也不需要逆置

  struct list *last = ls ->next;//逆置后ls ->next就成了最后一个节点了；

//逆置之前需要保存三个结点：当前结点的指针，上一个结点的指针，下一个结点的指针

  struct list *pre = ls;//上一个结点的指针
  struct list *cur = ls -> next;//当前节点的指针
  struct list *next = NULL;//下一个节点的指针
  while(cur)
    {
      next = cur -> next;
      cur -> next = pre;
      pre = cur;
      cur = next;
    }
    
    ls -> next = pre;
    last ->next = NULL;
}




int  main(void)
{
  //创建三个结点(可以通过函数)

  struct  list  *first  =  calloc(sizeof(struct  list),  1);//在堆中间创建一个结点
  struct  list  *second =  calloc(sizeof(struct  list),  1);
  struct  list  *third  =  create_list();
  struct  list  *fourth =  create_list();
  //将三个结点连接起来

  first   ->  next  =  second;
  second  ->  next  =  third;
  third   ->  next  =  fourth;
  fourth  ->  next  =  NULL;

  //数据域进行赋值

  first   ->  data  =  2;
  second  ->  data  =  3;
  third   ->  data  =  4;
  fourth  ->  data  =  5;

  reverse(first);
  traverse(first);

  return  0;

}

```

程序运行结果：

![链表元素逆置]($resource/%E9%93%BE%E8%A1%A8%E5%85%83%E7%B4%A0%E9%80%86%E7%BD%AE.png)



