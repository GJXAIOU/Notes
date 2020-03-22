# c_pro_day06 文件操作

**day5 的作业视频没有看**

## 一、基础知识点：

- 数据都是先放入缓冲区，只有当缓冲区满了，或者程序正常结束，或者文件关闭的时候，数据才正式的写入文件中。

- 句柄：用于标识作用


C语言中有三个特殊的指针无需定义，打开就可使用
 - stdin :标准输入：默认为当前终端（键盘 ）
 - stdout :标准输出：默认为当前终端（屏幕）
 - stderr :标准出错：默认为当前终端（屏幕）

示例：
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

int main(void)
{
	fputc('a', stdout);//因为stdout对应于屏幕：这是系统文件，缓冲区不适用

	char ch;
	ch = fgetc(stdin);
	printf("ch = %c\n", ch);

	system("pause");
	return 0;

}

```


- 文件的绝对路径与相对路径
  - 绝对路径只能在Windows中使用，而且文件路径中的`\` 需要更换为`\\`
  - 相对路径都可以使用，只需要使用`./` 后面加文件名即可
 注：针对于VS：相对路径：在编译代码的时候该路径为相对于项目工程；在直接运行可执行程序的时候，该路径是相对于可执行程序（一般需要将文件拷贝到这个目录下）


- 字符串一行无法写下可以使用续行符：`\`,使用示例:
“sdfdf” \
“dfjkdlfjdskl” 




## 二、具体函数的使用

![API]($resource/API.png)

### (一)按字节读取
- fputc
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

void my_fputc(char *path)
{
	FILE *fp = NULL;
	fp = fopen(path, "w+");
	if (fp == NULL)
	{
		perror("my_putc fopen");
		return -1;
	}
	
	char buf[] = "this is  a test program";
	
	int i = 0;
	int n = strlen(buf);
	for (i = 0; i < n; i++)
		{
			//fputc的返回值为成功写入文件的字符
			int ch = fputc(buf[i], fp);
			printf("ch = %c\n", ch);
		}

		if (fp != NULL)
		{
			fclose(fp);
			fp = NULL;
		}

}

int main(void)
{
	my_fputc("../ lianxi.txt");

	system("pause");
	return 0;

}
    
```


- fgetc

```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

void my_fgetc(char *path)
{
	FILE *fp = NULL;
	fp = fopen(path, "r+");
	if (fp == NULL)
	{
		perror("my_fgetc fopen");
		return -1;
	}else
	{

		char buf;
		while ((buf = fgetc(fp))  !=  EOF)
			{
				printf("%c", buf);
			}
			printf("\n");

		fclose(fp);
		fp = NULL;
	}

}

int main(void)
{
	my_fgetc("./lianxi.txt");

	system("pause");
	return 0;

}
    
```

- fputs
```c
#include<stdio.h>
#include<stdlib.h>
#pragma warning(disable:4996)

void my_fputs(char *path)
{
	FILE *fp = NULL;
	fp = fopen(path, "w+");
	if (fp == NULL)
	{
		perror("my_puts fopen");
		return -1;
	}

	char *buf[] = { {"this is  a test program"},{"hello"},{"python"} };

	int i = 0;
	int n = strlen(buf);
	for (i = 0; i < n; i++)
	{
		//fputc的返回值为成功写入文件的字符
		int ch = fputs(buf[i], fp);
		printf("ch = %c\n", ch);
	}

	if (fp != NULL)
	{
		fclose(fp);
		fp = NULL;
	}

}

int main(void)
{
	my_fputs("../ lianxi.txt");

	system("pause");
	return 0;

}
```

- fgets原理同上
按照`\n`作为标志进行换行



### （二）按块进行读取写入

- fwrite
```c
/*

按照块进行写入文件，使用fwrite
*/



#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#pragma warning(disable:4996)

typedef struct stu
{
	char name[50];
	int id;
}stu;

void my_fwrite(char *path)
{
	FILE *fp = NULL;
	//读写的方式打开，如果文件不存在则创建
	fp = fopen(path, "w+");

		if (fp == NULL)
		{
			perror("my_fwrite fopen");
			return;
        }


		stu s[3];
		int i = 0;
		char buf[50];
		for (i = 0; i < 3; i++)
		{
			sprintf(buf, "stu%d%d%d", i, i, i);
			strcpy(s[i].name, buf);

			s[i].id = i + 1;

		}
		
		
		//写文件，按照块来写
		//s,表示需要写入文件的内存首地址
		//sizeof(stu)表示每次写入的块的大小，
		//3，表示一共需要写入的块数，所以整个写入的文件大小为：sizeof(stu)*3
		//fp,为文件指针
		//函数的返回值为成功写入文件的块的数目

		int ret = fwrite(s, sizeof(stu), 3, fp);



		if (fp != NULL)
		{
			fclose(fp);
			fp = NULL;
		}
}


int main(void)
{
	
	
	
	my_fwrite("../05.txt");
	system("pause");

	return 0;

}
```
程序运行结果：在程序所在的上一级目录之下创建一个05.txt文件，并将内容写入

- fread
```c
/*

使用fread进行按块读取文件
*/


#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#pragma warning(disable:4996)

typedef struct stu
{
	char name[50];
	int id;
}stu;

void my_fread(char *path)
{
	FILE *fp = NULL;
	//读写的方式打开，如果文件不存在则打开失败
	fp = fopen(path, "w+");

	if (fp == NULL)
	{
		perror("my_fread fopen");
		return;
	}


	stu s[3];
	int i = 0;
	char buf[50];
	for (i = 0; i < 3; i++)
	{
		sprintf(buf, "stu%d%d%d", i, i, i);
		strcpy(s[i].name, buf);

		s[i].id = i + 1;

	}


	//读文件，按照块来读
	//s,表示需要读取文件的内存首地址
	//sizeof(stu)表示每次读取的块的大小，
	//3，表示一共需要读取的块数，所以整个读取的文件大小为：sizeof(stu)*3
	//fp,为文件指针
	//函数的返回值为成功读取文件的块的数目

	int ret = fread(s, sizeof(stu), 3, fp);

	
	for (i = 0; i < 3; i++)
	{
		printf("%s,%d\n", s[i].name, s[i].id);
	}


	if (fp != NULL)
	{
		fclose(fp);
		fp = NULL;
	}
}


int main(void)
{

	my_fread("../05.txt");
	system("pause");

	return 0;

}

```
程序运行结果：
`stu000,1`
`stu111,2`
`stu222,3`


### (三)格式化输入输出

- fprintf
```c

/*
使用fprintf ,进行格式化输出
*/

#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#pragma warning(disable:4996)

typedef struct stu
{
	char name[50];
	int id;
}stu;

void my_fprintf(char *path)
{
	FILE *fp = NULL;
	//读写的方式打开
	fp = fopen(path, "w+");

	if (fp == NULL)
	{
		perror("my_fread fopen");
		return;
	}


	//打印到屏幕（以下两种等价）
	printf("hello i am ,mial is %d\n", 333);

	fprintf(stdout, "hello i am ,mial is %d\n", 333);

	//打印到文件
	fprintf(fp, "hello i am ,mial is %d\n", 333);


	if (fp != NULL)
	{
		fclose(fp);
		fp = NULL;
	}
}


int main(void)
{
	my_fprintf("../06.txt");
	system("pause");

	return 0;

}

```


- fscanf 进行格式化输入
```c
/*

使用fscanf ,进行格式化输入
*/

#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#pragma warning(disable:4996)

typedef struct stu
{
	char name[50];
	int id;
}stu;

void my_fscanf(char *path)
{
	FILE *fp = NULL;
	//读写的方式打开
	fp = fopen(path, "r+");

	if (fp == NULL)
	{
		perror("my_fscanf fopen");
		return;
	}

	int a = 0;
	fscanf(fp, "hello i am ,mial is %d\n", &a);  //相当于从fp 中将%d位置的元素取出来
	printf("a = %d\n", a);





	if (fp != NULL)
	{
		fclose(fp);
		fp = NULL;
	}
}


int main(void)
{

	my_fscanf("../06.txt");
	system("pause");

	return 0;

}
```



### (四) 进行文件随机读取
```c


/*

随机位置读文件
*/


#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#pragma warning(disable:4996)

typedef struct stu
{
	char name[50];
	int id;
}stu;

void my_fread(char *path)
{
	FILE *fp = NULL;
	//读写的方式打开，如果文件不存在则打开失败
	fp = fopen(path, "w+");

	if (fp == NULL)
	{
		perror("my_fread fopen");
		return;
	}


	stu s[3];

	stu tmp;


	//先只读第三个结构体的内容，以下两种方式等价

	fseek(fp, 2 * sizeof(stu), SEEK_SET);  //从开头读，跳过两个结构体大小
	//fseek(fp, 0 - sizeof(stu), SEEK_END);//从结果读，跳过一个结构体大小

	//判断是否读取到该结构体，同时打印输出

	int ret = 0;
	ret = fread(&tmp, sizeof(stu), 1, fp);
	if (ret == 1)
	{

		printf("[tmp] %s,%d\n", tmp.name, tmp.id);

	}

	//将文件的光标移动到最开始的地方，从新将这个文件的中三个结构体都读取（以下方式等价）
	//fseek(fp, 0, SEEK_SET);
	rewind(fp);

		int i = 0;
		char buf[50];
		for (i = 0; i < 3; i++)
		{
			sprintf(buf, "stu%d%d%d", i, i, i);
			strcpy(s[i].name, buf);
	
			s[i].id = i + 1;
	
		}
	
	
	    ret = fread(s, sizeof(stu), 3, fp);
	
		
		for (i = 0; i < 3; i++)
		{
			printf("%s,%d\n", s[i].name, s[i].id);
		}
	
	

	if (fp != NULL)
	{
		fclose(fp);
		fp = NULL;
	}
}


int main(void)
{

	my_fread("../05.txt");
	system("pause");

	return 0;

}

```


~~加密解密视频没有看：17-22没看~~











