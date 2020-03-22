# day05 结构体


## 一、结构体的基础操作

- 结构体类型定义
```c
#include<stdio.h>

//结构体类型的定义
	/*
	首先struct为关键字
	struct Teacher 结合在一起才是类型
	{}后面有；
	*/
	
	struct Teacher
	{
		char name[50];
		int age;
	};


int main()
{
	return 0;
}
```

- 结构体变量定义
  - 方法一： 
```c
#include<stdio.h>

struct Teacher    // 先定义类型
{
	char name[50];
	int age;
};


//结构体变量的定义
//第一种：先定义类型，再定义变量--最常用的方法
struct Teacher t1;  //这里定义的就是全局变量

int main()
{
	struct Teacher t2;//这里定义的就是局部变量

	return 0;
}
```
   
  - 方法二：方法三：
```c
#include<stdio.h>
//第2种：定义类型的时候同时定义变量
struct Teacher
{
	char name[50];
	int age;
}t2,t3;

//第三种：可以不用名字
struct 
{
	char name[50];
	int age;
}t2, t3;


int main()
{

	return 0;
}
```

- 结构体变量的初始化
```c
#include<stdio.h>
//定义的同时进行初始化
struct Teacher
{
	char name[50];
	int age;
}t1 = {"hello",18};

//或者定义同时进行初始化，这里对应的是另一种定义方法：
struct Teacher t2 = { "hello",19 };

int main()
{
	//使用
	printf("%s,%d\n", t2.name, t2.age);
	system("pause");
	return 0;
}
```

- 使用typedef该类型名
```c
#include<stdio.h>

typedef struct Teacher2
{
	char name[3];
	int age;
}Teacher2;//将struct Teacher 定义为Teacher2
//对应的使用
Teacher2 t3;

int main()
{
	//使用
	printf("%s,%d\n", t3.name, t3.age);
	system("pause");
	return 0;
}
```

- 点运算符和指针法操作结构体
```c
#include<stdio.h>

typedef struct Teacher2
{
	char name[40];
	int age;
}Teacher2;//将struct Teacher 定义为Teacher2
//对应的使用
Teacher2 t3;

int main()
{
	//使用点运算符进行操作赋值
	strcpy(t3.name, "xiaoming");
	t3.age = 28;
	printf("%s,%d\n", t3.name, t3.age);

	//使用指针操作赋值
	struct Teacher2 *p = NULL; //不能直接向指针赋值，因为这个指针只占四个字节
	p = &t3;
	strcpy(p->name, "xiaoli");
	p->age = 25;
	printf("%s,%d\n", p->name, p->age);
	return 0;
}
```

- 结构体也是一种数据类型，复合类型，自定义类型


- 结构体变量相互赋值
```c
//在main函数内部，不需要调用函数
#include<stdio.h>
//结构体只是另一个类型，还没有分配空间，
//只有根据其类型定义变量的时候，才进行分配空间，有空间之后才能赋值
typedef struct Teacher2
{
	char name[40];
	int age;
}Teacher2;

int main()
{
	Teacher2 t1 = { "zhanger",15 };
	Teacher2 t2 = t1;//相同类型的两个结构体之间可以进行相互赋值
	//把t1成员变量内存的值拷贝给t2成员变量的内存，本质上t1和t2已经没有关系了
	printf("%s,%d\n", t2.name, t2.age);
	return 0;
}
```
程序运行结果：`zhanger,15`
```c
#include<stdio.h>
//结构体只是另一个类型，还没有分配空间，
//只有根据其类型定义变量的时候，才进行分配空间，有空间之后才能赋值
typedef struct Teacher2
{
	char name[40];
	int age;
}Teacher2;

void copyTeacher(Teacher2 *to, Teacher2 *from)
{
	*to = *from;
	printf("[copyTeacher] %s ,%d\n", to->name, to->age);
}

int main()
{
	Teacher2 t1 = { "zhanger",15 };
	Teacher2 t2;
	memset(&t2, 0, sizeof(t2));
	copyTeacher(&t2, &t1);//这里需要传址
	return 0;
}
```

- 结构体静态数组
```c
#include<stdio.h>
//结构体只是另一个类型，还没有分配空间，
//只有根据其类型定义变量的时候，才进行分配空间，有空间之后才能赋值
typedef struct Teacher
{
	char name[40];
	int age;
}Teacher;


int main()
{
	//定义静态数组和使用静态数组

	Teacher a[5] = { {"zhangsan",34},{"xiaoli",23},{"zhaoliu",12} };
	Teacher b[5] = { "wangqi",23,"qianba",11,"liuliuliu",94 };

	//使用
	int i = 0;
	for (i = 0; i < 5; i++)
	{
		printf("%s,%d\n", a[i].name, a[i].age);

	}
	return 0;
}
```

- 结构体动态数组
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char name[40];
	int age;
}Teacher;


int main()
{
	//定义和使用动态数组
	Teacher *p = (Teacher *)malloc(3 * sizeof(Teacher));
	if (p == NULL)
	{
		return -1;
	}

	char buf[10];
	int i = 0;
	for (i = 0; i < 3; i++)
	{
		sprintf(buf, "name %d%d%d", i, i, i);
		strcpy(p[i].name, buf);
		p[i]. age = 20 + i;
	}

	for (i = 0; i < 3; i++)
	{
		printf("num of %d:%s,%d\n", i + 1, p[i].name, p[i].age);

	}
	printf("\n");


	if (p != NULL)
	{
		free(p);
		p = NULL;
	}


	return 0;
}
```
程序运行结果：
`num of 1:name 000,20`
`num of 2:name 111,21`
`num of 3:name 222,22`

- 结构体嵌套一级指针问题
```c
//结构体变量为普通变量名的时候：

#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char name1[40];//这里name已经分配了40个字节的空间了
	char *name;//这时候并没有分配空间
	int age;
}Teacher;


int main()
{
	Teacher t;
	t.name = (char *)malloc(30); //必须先分配空间
	strcpy(t.name, "lily");
	t.age = 22;
	printf("name = %s ,age = %d\n", t.name, t.age);

	if (t.name != NULL)
	{
		free(t.name);
		t.name = NULL;
	}

	return 0;
}

```
程序运行结果：`name = lily ,age = 22`
```c
//结构体变量为指针的时候

#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char name1[40];//这里name已经分配了40个字节的空间了
	char *name;//这时候并没有分配空间
	int age;
}Teacher;


int main()
{
	Teacher *p = NULL;
	p = (Teacher *)malloc(sizeof(Teacher));//首先为p分配空间
	p->name = (char *)malloc(30); //再为p.name 分配空间
	strcpy(p->name, "lilei");
	p->age = 22;
	printf("name = %s ,age = %d\n", p->name, p->age);

	if (p->name != NULL)
	{
		free(p->name);
		p->name = NULL;
	}

	if (p != NULL)
	{
		free(p);
		p = NULL;


	}
	return 0;
}
```
程序运行结果为：`name = lilei ,age = 22`

程序运行内存结构图：
![搜狗截图20181109154732]($resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20181109154732.png)

```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char name1[40];//这里name已经分配了40个字节的空间了
	char *name;//这时候并没有分配空间
	int age;
}Teacher;


int main()
{
	Teacher *p = NULL;
	p = (Teacher *)malloc(sizeof(Teacher)*3);
	
	int i = 0;
	char buf[30];
	for (i = 0; i < 3; i++)
	{
		p[i].name = (char *)malloc(30);
		sprintf(buf, "name %d%d%d", i, i, i);
		strcpy(p[i].name, buf);
		p[i].age = 20 + 2 * i;

	}


	for (i = 0; i < 3; i++)
	{
		printf("%s,%d\n", p[i].name, p[i].age);
	}

	for (i = 0; i < 3; i++)
	{
		if (p[i].name != NULL)
		{
			free(p[i].name);
			p[i].name = NULL;
		}


	}



	if (p != NULL)
	{
		free(p);
		p = NULL;
	}

	return 0;
}
```
程序运行结果：
`name 000,20`
`name 111,22`
`name 222,24`


- 结构体做函数参数(例子等于上面程序进行函数封装)
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char name1[40];//这里name已经分配了40个字节的空间了
	char *name;//这时候并没有分配空间
	int age;
}Teacher;


//就是打印的功能
void showTeacher(Teacher *p, int n)
{
	int i = 0;
	for (i = 0; i < n; i++)
	{
		printf("%s,%d\n", p[i].name, p[i].age);//虽然P是指针，但是这里使用的是里面的元素，这里的p[i].name 等价于  (*(p+i)).name
	}

}

//释放功能的实现
void freeTeacher(Teacher *p, int n)
{
	int i = 0;
	for (i = 0; i < n; i++)
	{
		if (p[i].name != NULL)
		{
			free(p[i].name);
			p[i].name = NULL;
		}
	}

	if (p != NULL)
	{
		free(p);
		p = NULL;
	}
}


  
int main()
{
	Teacher *p = NULL;
	p = (Teacher *)malloc(sizeof(Teacher)*3);
	
	int i = 0;
	char buf[30];
	for (i = 0; i < 3; i++)
	{
		p[i].name = (char *)malloc(30);
		sprintf(buf, "name %d%d%d", i, i, i);
		strcpy(p[i].name, buf);
		p[i].age = 20 + 2 * i;

	}



	showTeacher(p, 3);//传地址

	//释放空间
	freeTeacher(p, 3);

	p = NULL;
	
	return 0;
}
```

- 结构体嵌套二级指针
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char **stu;//二维内存
}Teacher;




int main()
{
	char **name = NULL;
	int n = 3;
	int i = 0;
	name = (char **)malloc(n * sizeof(char *));
	//下面为name 中每一个变量进行赋值
	for (i = 0; i < n; i++)
	{
		name[i] = (char *)malloc(30);
		strcpy(name[i], "lily");
	}

	for (i = 0; i < n; i++)
	{
		printf("%s\n", name[i]);

	}

	//用完需要释放
	//先释放里面的
	for (i = 0; i < n; i++)
	{
		if (name[i] != NULL)
		{
			free(name[i]);
			name[i] = NULL;
		}
	}

	//释放外面的
	if (name != NULL)
	{
		free(name);
		name = NULL;
	}


	return 0;
}
```
//使用第一个
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char **stu;//二维内存
}Teacher;




int main()
{
	

	//使用结构体的时候

	//1.
	Teacher t;
	//t.stu[3]  //t里面有三个成员

	int n = 3;
	int i = 0;
	t.stu = (char **)malloc(n * sizeof(char *));
	//下面为name 中每一个变量进行赋值
	for (i = 0; i < n; i++)
	{
		t.stu[i] = (char *)malloc(30);
		strcpy(t.stu[i], "lily");
	}

	for (i = 0; i < n; i++)
	{
		printf("%s\n", t.stu[i]);

	}

	//用完需要释放
	//先释放里面的
	for (i = 0; i < n; i++)
	{
		if (t.stu[i] != NULL)
		{
			free(t.stu[i]);
			t.stu[i] = NULL;
		}
	}

	//释放外面的
	if (t.stu != NULL)
	{
		free(t.stu);
		t.stu = NULL;
	}
	return 0;
}
```

//使用第二个
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char **stu;//二维内存
}Teacher;




int main()
{
//相当于将1中的t.stu 改为p -> stu，然后增加释放P即可

	//2.
	Teacher *p = NULL;
	//p->stu[3]

	p = (Teacher *)malloc(sizeof(Teacher));


	int n = 3;
	int i = 0;
	p->stu = (char **)malloc(n * sizeof(char *));
	//下面为name 中每一个变量进行赋值
	for (i = 0; i < n; i++)
	{
		p->stu[i] = (char *)malloc(30);
		strcpy(p->stu[i], "lily");
	}

	for (i = 0; i < n; i++)
	{
		printf("%s\n", p->stu[i]);

	}

	//用完需要释放
	//先释放里面的
	for (i = 0; i < n; i++)
	{
		if (p->stu[i] != NULL)
		{
			free(p->stu[i]);
			p->stu[i] = NULL;
		}
	}

	//释放外面的
	if (p->stu != NULL)
	{
		free(p->stu);
		p->stu = NULL;
	}
	//上面只是释放了stu,还需要释放P

	if (p != NULL)
	{
		free(p);
		p = NULL;
	}

	return 0;
}

```
//第三种方式
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char **stu;//二维内存
}Teacher;


int main()
{

	//之前都是一个元素，现在是三个元素
	//3.
	Teacher *q = NULL;
	//相当于 Teacher q[3]  ,然后 q[i].stu[3]
	q = (Teacher *)malloc(sizeof(Teacher) * 3);

	int i = 0;
	int j = 0;
	for (i = 0; i < 3; i++)
	{
		q[i].stu = (char **)malloc(3 * sizeof(char *));
		//相当于现在有三个指针，需要分别进行分配空间
		for (j = 0; j < 3; j++)
		{
			q[i].stu[j] = (char *)malloc(30);
			char buf[30];
			sprintf(buf, "name %d%d%d%d", i, i, j, j);
			strcpy(q[i].stu[j], buf);
		}

	}

	//将结果进行打印输出
	for (i = 0; i < 3; i++)
	{
		printf("%s,%s,%s\n", q[i].stu[0], q[i].stu[1], q[i].stu[2]);
	}

	printf("\n");

	//释放
	//先释放上面的三个指针，然后释放stu,最后释放q

	for (i = 0; i < 3; i++)
	{
		for (j = 0; j < 3; j++)
		{
			//先释放三个指针
			if (q[i].stu[j] != NULL)
			{
				free(q[i].stu[j]);
				q[i].stu[j] = NULL;
			}
		}

		//然后释放stu
		if (q[i].stu != NULL)
		{
			free(q[i].stu);
			q[i].stu = NULL;
		}

	}

	if (q != NULL)
	{
		free(q);
		q = NULL;
	}

	return 0;
}
```

//将上面的第三个程序进行函数封装得到
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char **stu;//二维内存
}Teacher;



//n1表示老师的个数，n2表示学生的个数
int createTeacher(Teacher **tmp, int n1 ,int n2)
{
	//先做一个判断
	if(tmp == NULL)
	{
		return -1;
	}
	
	Teacher *q = (Teacher *)malloc(sizeof(Teacher) * n1);

	int i = 0;
	int j = 0;
	for (i = 0; i < n1; i++)
	{
		q[i].stu = (char **)malloc(n2 * sizeof(char *));
		//相当于现在有三个指针，需要分别进行分配空间
		for (j = 0; j < n2; j++)
		{
			q[i].stu[j] = (char *)malloc(30);
			char buf[30];
			sprintf(buf, "name %d%d%d%d", i, i, j, j);
			strcpy(q[i].stu[j], buf);
		}

	}

	//间接赋值是指针存在的最大意义
	*tmp = q;
	

	return 0;
}



//打印函数

void showTeacher(Teacher *q, int n1, int n2)
{

	if (q == NULL)
	{
		return;
	}
	int i = 0;
	int j = 0;
	//将结果进行打印输出
	for (i = 0; i < n1; i++)
	{
		for(j= 0; j< n2; j++)
		{
			printf("%s\n", q[i].stu[j]);
		}
		printf("\n");
	}



}


//释放
//先释放上面的三个指针，然后释放stu,最后释放q

void freeTeacher(Teacher **tmp, int n1, int n2)
{
	int i = 0;
	int j = 0;
	if (tmp == NULL)
	{
		return;
	}
	Teacher *q = *tmp;

	for (i = 0; i < n1; i++)
	{
		for (j = 0; j < n2; j++)
		{
			//先释放三个指针
			if (q[i].stu[j] != NULL)
			{
				free(q[i].stu[j]);
				q[i].stu[j] = NULL;
			}
		}

		//然后释放stu
		if (q[i].stu != NULL)
		{
			free(q[i].stu);
			q[i].stu = NULL;
		}

	}

	if (q != NULL)
	{
		free(q);
		q = NULL;
		*tmp = NULL;
	}




}




int main()
{

	//之前都是一个元素，现在是三个元素
	//3.
	Teacher *q = NULL;
	//相当于 Teacher q[3]  ,然后 q[i].stu[3]
	int ret = 0;

	ret = createTeacher(&q, 3,3);
	
	if (ret != 0)
	{
		return ret;
	}


	showTeacher(q, 3, 3);


	freeTeacher(&q, 3, 3);




	



	
	return 0;
}
```

- 结构体的点运算符和指针法操作的区别
普通变量后面用`.`加上成员名
指针后面使用`->` 加上成员名


- 结构体数组的排序
将结构体中元素按照从小到大进行来排序
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	int age;
	char **stu;//二维内存
}Teacher;



//n1表示老师的个数，n2表示学生的个数
int createTeacher(Teacher **tmp, int n1 ,int n2)
{
	//先做一个判断
	if(tmp == NULL)
	{
		return -1;
	}
	
	Teacher *q = (Teacher *)malloc(sizeof(Teacher) * n1);

	int i = 0;
	int j = 0;
	for (i = 0; i < n1; i++)
	{
		q[i].stu = (char **)malloc(n2 * sizeof(char *));
		//相当于现在有三个指针，需要分别进行分配空间
		for (j = 0; j < n2; j++)
		{
			q[i].stu[j] = (char *)malloc(30);
			char buf[30];
			sprintf(buf, "name %d%d%d%d", i, i, j, j);
			strcpy(q[i].stu[j], buf);
		}
		//首先对age进行赋值
		q[i].age = 20 + i;
	}



	//间接赋值是指针存在的最大意义
	*tmp = q;
	


	return 0;
}



//打印函数

void showTeacher(Teacher *q, int n1, int n2)
{

	if (q == NULL)
	{
		return;
	}
	int i = 0;
	int j = 0;
	//将结果进行打印输出
	for (i = 0; i < n1; i++)
	{
		printf("[age= %d]\t ", q[i].age);
		for(j= 0; j< n2; j++)
		{
			printf("%s\n", q[i].stu[j]);
		}
		printf("\n");
	}



}


void sortTeacher(Teacher *p, int n)
{
	if (p == NULL)
	{
		return;
	}

	int i = 0;
	int j = 0;
	Teacher tmp;

	for (i = 0; i < n - 1; i++)
	{

		for (j = i + 1; j < n; j++)
		{
			if (p[i].age < p[j].age)
			{
				tmp = p[i];
				p[i] = p[j];
				p[j] = tmp;
			}

		}
	}
}





//释放
//先释放上面的三个指针，然后释放stu,最后释放q

void freeTeacher(Teacher **tmp, int n1, int n2)
{
	int i = 0;
	int j = 0;
	if (tmp == NULL)
	{
		return;
	}
	Teacher *q = *tmp;

	for (i = 0; i < n1; i++)
	{
		for (j = 0; j < n2; j++)
		{
			//先释放三个指针
			if (q[i].stu[j] != NULL)
			{
				free(q[i].stu[j]);
				q[i].stu[j] = NULL;
			}
		}

		//然后释放stu
		if (q[i].stu != NULL)
		{
			free(q[i].stu);
			q[i].stu = NULL;
		}

	}

	if (q != NULL)
	{
		free(q);
		q = NULL;
		*tmp = NULL;
	}




}




int main()
{

	//之前都是一个元素，现在是三个元素
	//3.
	Teacher *q = NULL;
	//相当于 Teacher q[3]  ,然后 q[i].stu[3]
	int ret = 0;

	ret = createTeacher(&q, 3,3);
	
	if (ret != 0)
	{
		return ret;
	}

	//排序前
	printf("before:\n");

	showTeacher(q, 3, 3);
	
	sortTeacher(q, 3);

	//排序后
	printf("after:\n");
	showTeacher(q, 3, 3);


	freeTeacher(&q, 3, 3);


	return 0;
}
```


- 结构体的深拷贝和浅拷贝
//浅拷贝
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char *name;
	int age;
}Teacher;



int main(void)
{

	Teacher t1;
	t1.name = (char *)malloc(30);
	strcpy(t1.name, "lily");
	t1.age = 22;

	Teacher t2;
	t2 = t1;

	printf("[t2]= %s, %d\n", t2.name, t2.age);


	//浅拷贝的内存为同一块，所以只需要释放一次
	if (t1.name == NULL)
	{
		free(t1.name);
		t1.name = NULL;

	}


	system("pause");

	return 0;

}


```

![浅拷贝示例图]($resource/%E6%B5%85%E6%8B%B7%E8%B4%9D%E7%A4%BA%E4%BE%8B%E5%9B%BE.png)

//深拷贝：认为的增加一块内存，相当于再重新拷贝一份
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

typedef struct Teacher
{
	char *name;
	int age;
}Teacher;



int main(void)
{

	Teacher t1;
	t1.name = (char *)malloc(30);
	strcpy(t1.name, "lily");
	t1.age = 22;

	Teacher t2;
	t2 = t1;//已经复制了一遍

	//在人为的进行重新拷贝一次
	t2.name = (char *)malloc(30);
	strcpy(t2.name, t1.name);

	printf("[t2]= %s, %d\n", t2.name, t2.age);


	
	if (t1.name == NULL)
	{
		free(t1.name);
		t1.name = NULL;
	}


	if (t2.name == NULL)
	{
		free(t2.name);
		t2.name = NULL;
	}

	system("pause");

	return 0;

}


```



- 结构体偏移
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)


//结构体类型定义下来之后，内部的成员变量的内存布局就已经确定了
typedef struct Teacher
{
	char name[64];//64
	int age;//4
	int id;//4
}Teacher;



int main(void)
{

	Teacher t1;
	Teacher *p = NULL;
	p = &t1;

	int n1 = (int)(&p->age) - (int)p;  //这里求的是相对于结构体的首地址的偏移，使用int进行强制转换是因为整数便于比较偏移量
	printf("n1 = %d\n", n1);


	int n2 = (int)&((Teacher *)0)->age;  //这是相对于绝对0 地址的偏移量
	printf("n2 = %d\n", n2);
	
	system("pause");

	return 0;

}

```
程序运行结果：
`n1 = 64`
`n2 = 64`

- 结构体字节对齐（内存对齐相当于使用空间换取时间）
  - 理论性知识点：
  
![结构体对齐1]($resource/%E7%BB%93%E6%9E%84%E4%BD%93%E5%AF%B9%E9%BD%901.png)

![结构体对齐2]($resource/%E7%BB%93%E6%9E%84%E4%BD%93%E5%AF%B9%E9%BD%902.png)

![结构体对齐3]($resource/%E7%BB%93%E6%9E%84%E4%BD%93%E5%AF%B9%E9%BD%903.png)


   - 单一结构体内存大小
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

//内存对齐：相当于使用空间换时间
/*
	默认情况下：对齐单位以最长的那个，例如double为8，但是数组类型：char[39] 本质上长度为1，只是连起来放置39个，

	第一个的偏移量为0，相当于是从头开始排

*/
typedef struct Teacher
{
	
	int age;
	double haa;
	char name[10];
	int id;
}Teacher;

/*分析：
首先最长的单位为8
首先age本身就是4个字节，相当于占4个位置,
然后haa本身为8个字节，偏移即8*0 = 0,可是0有元素了，所以8*1 = 8 ，可以使用，则中间这4个位置为空
然后name本身为1个字节，同时以8个长度对齐
然后id为4个字节，正好补全



age:      1 1 1 1 * * * *
haa:      1 1 1 1 1 1 1 1
name：    1 1 1 1 1 1 1 1
          1 1 * * 1 1 1 1

最终结构体占32字节
*/



int main(void)
{
	Teacher A;
	printf("%d\n", sizeof(A));

	system("pause");
	return 0;

}


```
程序运行结果：`32`

  - 当结构体中嵌套结构体的时候
  - 
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

//内存对齐：相当于使用空间换时间
/*
	默认情况下：对齐单位以最长的那个，例如double为8，但是数组类型：char[39] 本质上长度为1，只是连起来放置39个，

	第一个的偏移量为0，相当于是从头开始排

*/
typedef struct Teacher
{
	
	int age;
	double haa;
	char name[10];

}Teacher;


typedef struct Teacher1
{

	int age;
	double haa;
	char name[10];

	//这前面所占的空间，最后一排就是没有占满也要自动的填满，这下面嵌套的结构体从下一行开始的时候添加
	Teacher a;
}Teacher1;


int main(void)
{
	Teacher1 A;
	printf("%d\n", sizeof(A));

	system("pause");
	return 0;

}

```
程序运行结果：`64`


- 根据指定的长度进行对齐(指定的长度必须为2的倍数)
  - 如果指定长度小于结构体中最长的长度：按照指定长度对齐
  - 如果指定的长度大于结构体中最长的长度：按照结构体中最长的长度进行对齐 
**示例程序：**
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)


# pragma pack(2)  //以2个字节进行对齐（指定的长度必须为2的倍数）

typedef struct Teacher
{
	
	int age;
	double haa;
	char name[10];

}Teacher;


int main(void)
{
	Teacher A;
	printf("%d\n", sizeof(A));

	system("pause");
	return 0;

}

```



