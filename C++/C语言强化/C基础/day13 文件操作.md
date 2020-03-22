# day13 文件操作

# 文本文件

## **（一）fopen**

用于打开文件文件
**1.1将固定数据写入文本文件**
```c
#include<stdio.h>

int  main()
{

  FILE  *p  =  fopen("C:users\\gjx16\\desktop\\a.txt","w");
  //FILE为一个结构体，fopen的第一个参数为文件位置，第二个参数代表读还是写或者其他。因为\为转义字符，因此需要两个\\

  //"w"含义为：如果文件不存在，就建立一个文件，如果已经存在就覆盖

  fputs("hello  world",p);//写入一个字符串，文件位置为指针p指向的位置(一行一行的写入)

  fclose(p);//关闭这个文件

  return  0;

}
```
程序运行结果；在文件a.txt中写入文本`hello world`。


**1.2根据输入写入文本文件** 
可以根据用户输入的数据进行自动写入文件，通过输入`exit`来结束输入

```c
#include<stdio.h>
#include<string.h>

int  main(void)
{

  char  s[1024]  =  {0};
  FILE  *p  =  fopen("C:users\\gjx16\\desktop\\a.txt","w");//以文本文件写的方式打开这个文件

  while(1)
  {
    memset(s,0,sizeof(s));//首先将这个数组清零

    //scanf("%s",s);//使用scanf一行内不能输入空格

    gets(s);//使用gets可以获得行内空格

    if(strcmp(s,"exit")  ==  0)//如果用户输入exit则退出
    //strcmp(str1,str2),比较两个字符串，相等返回0，前小于后返回负数，前大于后返回正数
    break;

    int  len  =  strlen(s);//因为strlen遇到`\0`时候就会结束，相当于统计了这一行的长度。因为如果出现空格数组默认结束，这里是每一次输入回车的同时在文件中同样会输出一个回车

    s[len]  =  '\n';

    fputs(s,p);//将数组s中的数据写入p指向的文件

  }

  fclose(p);//关闭这个文件

  printf("you  have  finish  typing,please  open  the  file.\n");

  return  0;

}
```
程序运行结果：将用户输入的数据写入文件之中，如果用户输入`exit`时候结束输入

---

- **2.读文本文件**
```c
#include<stdio.h>
#include<string.h>

int  main(void)
{

  char  s[1024]  =  {0};

  FILE  *p  =  fopen("C:users\\gjx16\\desktop\\a.txt","r");//以文本文件的格式读取该文件

  printf("The  content  of  the  file  is:\n");

  while(!feof(p))//feof含义为：判断是否到达文件尾，如果到达文件尾返回真
    {
      memset(s,0,sizeof(s));//首先将这个数组清零

      fgets(s,sizeof(s),p);//第一个参数为存放读取内容的内存位置，第二个参数为这块内存的大小，第三个参数为fopen返回的文件指针（同样为一行一行的读取）

      printf("%s",s);//将读取的文件内容打印输出
    }

  fclose(p);//关闭这个文件

  return  0;

}
```
程序输出结果；将文本文件中的内容输出出来


---

- **3.加密文件**
将读取的内容通过一定的运算之后再次写入
```c
#include<stdio.h>
#include<string.h>

void  code(char  *s)
{
  while(*s)//遍历整个数组
    {
      (*s)++;//数组值加一
      s++;
    }
}


int  main(void)
{
  char  s[1024]  =  {0};

  FILE  *p  =  fopen("C:users\\gjx16\\desktop\\a.txt","r");//读文件
  FILE  *p1  =  fopen("C:users\\gjx16\\desktop\\b.txt","w");//写文件

  while(!feof(p))//feof含义为：判断是否到达文件尾，如果到达文件尾返回真
    {

      memset(s,0,sizeof(s));//首先将这个数组清零

      fgets(s,sizeof(s),p);//第一个参数为存放读取内容的内存位置，第二个参数为这块内存的大小，第三个参数为fopen返回的文件指针

      code(s);

      fputs(s,p1);//将读取的文件内容写入b.txt文件
  }

  fclose(p);//关闭这个文件
  fclose(p1);

  return  0;

}
```
程序运行结果：将文本文件a.txt中的数据全部加1之后再次写入文本文档b.txt中，从而实现加密的作用；

---

- **4.fopen 常用的操作方式**

|参数|作用|备注|
|---|---|---|
|r  |以只读的方式打开文件，该文件必须存在|如果打开成功返回打开的 文件指针，如果打开失败（可能文件不存在或者没有权限）返回NULL|
|a|以附加的方式打开只写文件；|若文件不存在，则会建立该文件，如果存在则写入的数据会被加到文件尾后，即文件的原有内容将会被保留（EOF符保留）
|r+|以可读写方式打开文件，该文件必须存在；
|rb+|读写打开一个二进制文件，允许读写数据，文件必须存在；
| rw+|读写打开一个文本文件，允许读和写；
|w|打开只写文件|若文件存在则文件长度清为0，即该文件内容会消失，如文件不存在则建立该文件；
|w+|打开可读写文件;|若文件存在则文件长度清为0，即该文件内容会消失，如文件不存在则建立该文件；
|a+|以附加的方式打开可读写的文件|若文件不存在，则会建立该文件，如果存在则写入的数据会被加到文件尾后，即文件的原有内容将会被保留（EOF符不保留）

注1： `w`表示在文本模式下写文件，而Windows的文本模式下，文件以`\r\n`代表换行，若用文本文件打开并使用fputs函数写入`\n`时候，系统会自动补齐为`\r\n`。
在Linux中文本模式下以`\n`表示换行。

`wb`表示在二进制模式下写文件，以`\n`作为换行标志，因此Windows的文本模式和二进制模式不一样，但是Linux的一样。

注2：`EOF`表示文件结尾最后一个字符
`feof`表示判断是不是到达文件结尾

---



## **（二）getc和putc**
  - **1.getc**
一个字节的挨个读取或者写入
```c
#include<stdio.h>
#include<string.h>
#include<errno.h>

int  main(void)

{

  FILE  *p  =  fopen("C:users\\gjx16\\desktop\\a.txt","r");

  if(p  ==  NULL)  //文件不存在或者无权限的情况下会打不开，相应的返回值为NULL
    {
        printf("The  file  is  missing  or  you  do  not  have  the  right  to  open  it，%s\n",strerror(errno));//这里后面的部分会打印出具体的错误原因
    }else
    {
      char  c  =  0;
      while((c  =  getc(p))  !=  EOF)//EOF代表文件最后的一个结束标识
        {
          printf("%c",c);
        }
        fclose(p);  //放在可以打开的情况下即可，因为如果打不开就不用关闭了

    }

  return  0;

}
```
程序运行结果：挨个的读取文件a.txt中的内容

---
  - **2.putc**
 运行一次只能写入一个字符

```c
#include<stdio.h>
#include<string.h>

int  main(void)
{
  FILE  *p  =  fopen("C:users\\gjx16\\desktop\\a.txt","w");

  if(p  ==  NULL)  //文件不存在或者无权限的情况下会打不开，相应的返回值为NULL
    {
        printf("The  file  is  missing  or  you  do  not  have  the  right  to  open  it");
    }else
    {
       putc('a',p);//将字符a写入p指向的文件
       putc('b',p);
       fclose(p);  //放在可以打开的情况下即可，因为如果打不开就不用关闭了
    }

  return  0;
}
```
程序运行结果；将字符`a`和`b`写入指针p指向的文件a.txt

---



## **（三）文件排序**
将文件a.txt中的一系列数字排好序会后放在文件b.txt中；
**程序缺点:** 无法读取大的文件，栈的存储空间有限
**注意点:** 读取出来的是字符，要先进行类型转换，使用`atoi`
```c
#include<stdio.h> 
#include<stdlib.h>
#include<string.h>

void  swap  (int  *a,  int  *b)
{
  int  tmp  =  *a;
  *a  =  *b;
  *b  =  tmp;
}


void  bubble(int  *p,int  n)
{
  int  i;
  int  j;
  for(i  =  0;i  <  n;i++)
    {
      for(j  =  1;j  <  n-i;  j++)
    {

    if(p[j-1]  >  p[j])
      {
        swap(&p[j-1],&p[j]);
      }
    }
  }
}



int  main()
{
  int  array[100]  ={0};
  char  buf[100];
  int  index  =  0;//这是一个计数器
  FILE  *p  =  fopen("C:\\users\\gjx16\\desktop\\a.txt","r");

  if(p  ==  NULL)//判断有没有权利打开该文件
  {
    printf("The  file  is  missing  or  you  do  not  have  the  right  to  open  it\n");
  }else
  {
    while(!feof(p))//如果没有到达文件结尾，那么循环继续
  {
    memset(buf,0,sizeof(buf));//每次读取文件一行之前就把这个buffer清空
    fgets(buf,sizeof(buf),p);//从文件中读取一行
    array[index]  =  atoi(buf);//将读取的一行转换为int，赋值给数组成员
    index++;
  }

fclose(p);
}

  bubble(array,index);//将数组排序
  p  =  fopen("C:\\users\\gjx16\\desktop\\b.txt","w");//用写的方式打开b.txt

  int  i;
  for(i  =  0;  i  <  index;i++)
    {
      memset(buf,0,sizeof(buf));
      sprintf(buf,"%d\n",array[i]);//将数组成员转换为字符串
      fputs(buf,p);
    }
  fclose(p);

  return  0;

}

```
程序运行结果：将a.txt中的数据读取之后排序 ，然后写入b.txt中


**改进：**
在堆中建立一个动态数组进行排序

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

void  swap  (int  *a,  int  *b)
{
  int  tmp  =  *a;
  *a  =  *b;
  *b  =  tmp;
}


void  bubble(int  *p,int  n)
{
  int  i;
  int  j;
  for(i  =  0;i  <  n;i++)
    {
      for(j  =  1;j  <  n-i;  j++)
        {
          if(p[j-1]  >  p[j])
            {
              swap(&p[j-1],&p[j]);
            }
        }
    }
}


int  main()
{
 
   int  index  =  0;//这是一个计数器
   char  buf[100];
    FILE  *p  =  fopen("C:\\users\\gjx16\\desktop\\a.txt","r");//第一次打开a.txt目的是知道这个文件有多少行
    while(!feof(p))//如果没有到达文件结尾，那么循环继续，第一次循环仅仅为了统计文件中的行数index
      {
        memset(buf,0,sizeof(buf));//每次读取文件一行之前就把这个buffer清空
        fgets(buf,sizeof(buf),p);//从文件中读取一行
        index++;
      }

  fclose(p);
  int *array = calloc(sizeof(int),index);//在堆中建立了一个动态数组，动态数组的成员数量等于a.txt中的行数

  
  p  =  fopen("C:\\users\\gjx16\\desktop\\a.txt","r");
  index = 0;//计数器从零开始

  if(p  ==  NULL)
  {
    printf("error\n");
}else
{
    while(!feof(p))//如果没有到达文件结尾，那么循环继续
      {
          memset(buf,0,sizeof(buf));//每次读取文件一行之前就把这个buffer清空
          fgets(buf,sizeof(buf),p);//从文件中读取一行
          array[index]  =  atoi(buf);//将读取的一行转换为int，赋值给数组成员
          index++;
      }

fclose(p);
}

  bubble(array,index);//将数组排序
  p  =  fopen("C:\\users\\gjx16\\desktop\\b.txt","w");//用写的方式打开b.txt

  int  i;
  for(i  =  0;  i  <  index;i++)
    {
      memset(buf,0,sizeof(buf));
      sprintf(buf,"%d\n",array[i]);//将数组成员转换为字符串
      fputs(buf,p);
    }
  fclose(p);

  return  0;

}
```
程序运行结果与上面程序一致，但是可以读取较大的文件

---

## **（四）解析文本内容**                                                                          
- 根据已定的表达式来计算

```c
#include<stdio.h>
#include<string.h>

//主要提取三个部分，运算符，运算符前后的数值，需要定义三个变量来存放

int select_string(const char *s)
{
  char num1[100] = {0};
  char num2[100] = {0};
  char operator = 0;

  int i;
  int len = strlen(s);//得到字符串的长度
  for(i = 0;i < len; i++)
    {
      if(s[i] == '+' || s[i] == '-' || s[i] == '*' || s[i] == '/')
        {
         //将运算符前面的数据放入num1中
          strncpy(num1,s,i);//接受地址，原来的地址，复制元素的数目         
          operator = s[i];//s[i] 为运算符
          break;
        }       
    }


  int start = i + 1;//前面的i为+号的位置，设其为起始位置
  for(;i < len; i++)
    {
      if(s[i] == '=') 
      {
        strncpy(num2,&s[start],i-start);  
      }                        
    }
    printf("num1 = %s,operator = %c,num2 = %s\n",num1,operator,num2);

//根据提取出来的元素值以及运算符号进行分类运算

    switch(operator)
    {
      case '+':
      return atoi(num1) + atoi(num2);
      case '-':
      return atoi(num1) - atoi(num2);
      case '*':
      return atoi(num1) * atoi(num2);
      case '/':   //一般在除法的时候进行一个判断，保证程序的健壮性
      {
        int a = atoi(num2);
        if(a)//如果a 有值的话，进行除法运算
          {
            return atoi(num1) / atoi(num2);
          }else
          return 0;
      }  
    }
}


int main(void)
{
  const char *string = "52+36=";
  int result = select_string(string);
  printf("%d\n",result);
  return 0;
}

```
程序运行结果：`88`,就是将固定的一个表达式的值计算出来

---


- 根据文本文件内部的数据，通过读写进行运算
```c
#include<stdio.h>
#include<string.h>

//主要提取三个部分，运算符，运算符前后的数值，需要定义三个变量来存放

int select_string(const char *s)
{
  char num1[100] = {0};
  char num2[100] = {0};
  char operator = 0;

  int i;
  int len = strlen(s);//得到字符串的长度
  for(i = 0;i < len; i++)
    {
      if(s[i] == '+' || s[i] == '-' || s[i] == '*' || s[i] == '/')
        {
         //将运算符前面的i个数据放入num1中
          strncpy(num1,s,i);//接受地址，原来的地址，复制元素的数目
          operator = s[i];//s[i] 为运算符
          break;
        }       
    }


  int start = i +1;//前面的i为+号的位置，设其为起始位置
  for(;i < len; i++)//i的起始位置就是上面i所循环到达的值，这里可以不写
    {
      if(s[i] == '=') 
      {
        strncpy(num2,&s[start],i-start);  
      }                        
    }
   // printf("num1 = %s,operator = %c,num2 = %s\n",num1,operator,num2);



//根据提取出来的元素值以及运算符号进行分类运算

    switch(operator)
    {
      case '+':
      return atoi(num1) + atoi(num2);
      case '-':
      return atoi(num1) - atoi(num2);
      case '*':
      return atoi(num1) * atoi(num2);
      case '/':   //一般在除法的时候进行一个判断，保证程序的健壮性
      {
        int a = atoi(num2);
        if(a)//如果a 有值的话，进行除法运算
          {
            return atoi(num1) / atoi(num2);
          }else
          return 0;
      }  
    
    }
}



//函数功能：删除读取的每行最后的回车符，这样表达式和答案就在同一行
void cutereturn(char *s)
{
  int len = strlen(s);
  if(s[len -1] == '\n')
    s[len -1] = 0;

}




int main(void)
{
  //const char *string = "52+36=";
  //int result = select_string(string);
  FILE *pa = fopen("C:\\users\\gjx16\\desktop\\a.txt","r");//以读的方式打开文件a.txt
  FILE *pb = fopen("C:\\users\\gjx16\\desktop\\b.txt","w");//以写的方式打开文件b.txt
  
  int buf1[1024];//存放从a.txt中读取的内容
  int buf2[1024]; 
  while(!feof(pa))
  {
    memset(buf1,0,sizeof(buf1));//将buf1里面的内容清空
    memset(buf2,0,sizeof(buf2));//将buf2里面的内容清空
    
    fgets(buf1,sizeof(buf1),pa);//从文件中读取一行记录，字符串最后以'\n'结尾的；
    
    cutereturn(buf1);//将后面的换行符即回车进行删除
    
    int value = select_string(buf1);//调用函数select_string将存放a.txt内容的buf1进行元素提取以及计算结果；
    sprintf(buf2,"%s%d\n",buf1,value);//将字符串buf1和value打印输出至buf2中
    //使用示例：
    //sprintf(s, "%4d%4d", 123, 4567); //产生：" 1234567"(这里指定了长度)
    
    
    //printf("%s\n",buf2);//可以使用该语句将值进行输出，下面采用将表达式和结果一起输入到新的文件中；
    
    //将a.txt中输出的结果存储到文件b.txt中

  fputs(buf2,pb);//将buf2中的内容放入b.txt中
 
  }
  fclose(pa);
  fclose(pb);
  printf("the result is on the b.txt");
  //printf("result = %d\n",result);
  return 0;
}

```
程序输出结果为：
![读取文件中的数据计算]($resource/%E8%AF%BB%E5%8F%96%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E8%AE%A1%E7%AE%97.png)



---


## **（五）fscanf函数使用**
fprintf,fscanf,fgets,fputs都是针对文本文件的一行操作；

fscanf用法和scanf用法基本一致，fscanf是从一个文件读取输入，scanf是从键盘读取输入值

示例：
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main()
{
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\c.txt","r");

  while(!feof(p))
    {
      char buf[100] = {0};
      //fgets(buf,sizeof(buf),p);//可以将文件中的内容读取存放在数组buf中，读取长度为sizeof（buf）

//使用方法一：
      //fscanf(p,"%s",buf);//从文件p中读取存入数组buf中

      int a= 0;
      int b = 0;
      fscanf(p,"%d + %d = ",&a,&b);//根据固定的格式解析相应的元素
      printf("a = %d , b = %d\n",a,b);
     
    }
    fclose(p);
    return 0;
}
```

---

## **（六）fprintf函数使用**
本质上和printf功能相似，只是fprintf是输出到文件之中
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

int main()
{
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\d.txt","w");
  char buf[100] = "hello world";
  int a = 7;
  int b= 8;
  fprintf(p,"%s,%d,%d",buf,a,b);//将buf中元素以及a,b中的元素输出至指针p指向的文件b.txt中
 fclose(p);
 return 0;
}
```


---

## 二、二进制文件的读写



### **（一）fread**
既能读取文本文件，也可以读取二进制文件

根据buf分长度可以采取两种读取方式：一次性读取或者使用循环每次读取一部分

示例；
 注：这里读取的字节包括`\r`和`\n`之内的都在内
```c
#include<stdio.h>

int main(void)
{
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\c.txt","rb");
  char buf[1024] = {0};
  fread(buf,sizeof(char),1,p);//第一个参数为缓冲区，第二个参数为读取的时候最小的单位大小（字节），第三个参数为一次读取几个单位，第四个参数为打开的文件指针；
  //读取指针p指向的文件中的元素，按照最小单位字节的大小，且一次只读取一个字节，放入数组buf中
  printf("buf = %s",buf);
  fclose(buf);
  return 0;
}
```

---

因为fread是按照命令中所设定的字节数来读取，并不是按照一行一行的读取，因此如果buf很小的情况下，只能按照循环来读取文件中的全部字节；
**当buf很小的时候的程序：（使用循环部分部分读取）**
```c
#include<stdio.h>

int main(void)
{
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\c.txt","rb");//Windows下文本文件以\r\n结尾，自动添加\r
  char buf[10] = {0};
  while(!feof(p))
    {
      fread(buf,sizeof(char),sizeof(buf) - 1,p);//按照buf的大小进行读取，这里减1是因为使每一次buf读取的时候都留下最后一位放置\0;不然每次buf中都会读满十个成员
      printf("buf = %s",buf);
    }
    fclose(buf);
  return 0;
}
```

---

## **（二）fwrite**
既能读取文本文件，也可以读取二进制文件
```c
#include<stdio.h>

int main()
{
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\e.bat","wb");
  char buf[10] = {0};
  buf[0] = 'a';
  buf[1] = 'b';
  buf[2] = 'c';
  buf[4] = 'd';
  fwrite(buf,sizeof(char),5,p);//将数组buf中的内容，以sizeof（char）为最小单位，将5个这样单位的数据写入文件中；
  //此时文件e.bat的大小为5个字节    

/*//当写入的是一个整数的时候
*  int a = 10;
*  fwrite(&a,sizeof(int),1,p);
*  当然这里的额第二个和第三个参数的位置都是可以互换的，因为最终一次性写入的数目是元素二和元素三的积
*/
  
  fclose(p);

  return 0;
  
}
```


---

### (三）二进制文件的复制
使用`fread`和`fwrite`实现文件的拷贝
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>


int main(void)
{
  FILE *pa =fopen("c:\\users\\gjx16\\desktop\\a.bat","rb");
  //这里只要是一个二进制文件都可以
  FILE *pb = fopen("c:\\users\\gjx16\\desktop\\b.bat","wb");

char buf[1024*4];//暂时存放的数组空间不能太小，太小造成读取速度太慢，但是如果太大的话容易造成溢出，这个大小一般也就是一次性读取的字节数目

while(!feof(pa))
  {
    memset(buf,0,sizeof(buf));
    size_t res = fread(buf,sizeof(char),sizeof(buf),pa);//返回从源文件中读取的字节数目
    fwrite(buf,sizeof(char),res,pb);//从源文件中读取多少字节，就玩目标文件中写入多少字节
   //这里的写入字节数一定是res*sizeof(char),不能使用sizeof(buf)*sizeof(char),这样会多读入很多字节，造成文件拷贝之后和源文件并不一致；
   
  }

fclose(pa);
fclose(pb);
return 0;

}


```


----


## （四）二进制文件的加密与解密
可以应用于任何文类型，不仅仅包括二进制文件
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

void code(char *p,size_t n)
{
  size_t i;
  for(i = 0; i < n; i++)
    {
       p[i] +=3;     
    }
}


int main(void)
{
  FILE *pa =fopen("c:\\users\\gjx16\\desktop\\a.txt","rb");
  //这里只要是一个二进制文件都可以
  FILE *pb = fopen("c:\\users\\gjx16\\desktop\\b.txt","wb");

char buf[1024*4];

while(!feof(pa))
  {
    memset(buf,0,sizeof(buf));
    size_t res = fread(buf,sizeof(char),sizeof(buf),pa);
    code(buf,res);
    fwrite(buf,sizeof(char),res,pb);
   
  }

fclose(pa);
fclose(pb);
return 0;

}
```
程序执行结果：（这里以文本文件为例）
![二进制文件加密]($resource/%E5%A4%A7%E5%B9%85%E5%BA%A6.png)

**解密的程序如下**
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

void decode(char *p,size_t n)
{
  size_t i;
  for(i = 0; i < n; i++)
    {
       p[i] -=3;     
    }
}


int main(void)
{
  FILE *pa =fopen("c:\\users\\gjx16\\desktop\\b.txt","rb");
  //这里只要是一个二进制文件都可以
  FILE *pb = fopen("c:\\users\\gjx16\\desktop\\c.txt","wb");

char buf[1024*4];

while(!feof(pa))
  {
    memset(buf,0,sizeof(buf));
    size_t res = fread(buf,sizeof(char),sizeof(buf),pa);
    code(buf,res);
    fwrite(buf,sizeof(char),res,pb);
   
  }

fclose(pa);
fclose(pb);
return 0;

}
```



## **（五）stat函数**
可以获取有关一个文件的很多信息
例如文件创建时间，修改时间，文件大小等等内容；

**拷贝一个大文件**
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<sys/stat.h> //调用stat函数的时候必须包含此头文件
#include<time.h>//这个头文件中包含了记录当前时间的clock函数

int main(void)
{
  clock_t c1  = clock();//得到系统的当前时间，单位：毫秒
  
  struct stat st ={0};  //定义一个名为st的结构
  stat("C:\\users\\gjx16\\desktop\\a.txt",&st);
  //调用完stat函数之后，文件的相关信息就存储在st的结构体中了

  //st.st_size可以得到文件的大小
  //printf("%u\n",st.st_size);//将文件的大小打印出来，单位为字节

  char *array = malloc(st.st_size);//根据文件大小在堆中动态的分派一块内存
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","rb");
  fread(array,sizeo(char),st.st_size,p);//相当于一次性将整个文件放入了内存
  fclose(p);
   
  p = fopen("C:\\users\\gjx16\\desktop\\b.txt","wb");
  fwrite(array,sizeo(char),st.st_size,p);
  fclose(p);

  clock_t c2 = clock();//得到运行完时候的系统时间
  printf("end,%u\n",c2-c1);
  return 0;
}

```

---

## （六）结构体与二进制文件

  **1.将结构体中的内容写入文件中**
针对于二进制文件
```c
#include<stdio.h>
#include<string.h>

struct student
{
  char name;
  int age;
};

int main()
{
   struct student st = {"xiaoming",34};
   FILE  *p = fopen("c:\\users\\gjx16\\desktop\\a.dat","wb");   
   fwrite(&st,sizeof(st),1,p);
   
   fclose(p);
   return 0;  
}
```
程序运行结果：在桌面上创建了一个a.bat的文件，文件大小为8个字节，因为该结构体大小为8个字节

**2.读取文件中的数据**

```c
#include<stdio.h>
#include<string.h>

struct student
{
  char name;
  int age;
};

int main()
{
   struct student st = {0};
   FILE  *p = fopen("c:\\users\\gjx16\\desktop\\a.dat","rb");   
   fread(&st,sizeof(st),1,p);
   fclose(p);
   printf("name = %d,age = %d\n",st.name,st.age);
   return 0;  
}

```

**3.例子：生成一个超大型的文件**
生成的一个大型文件，可能达到上G大小，这时候堆中也不可能放得下；
此例子为生成一个文本文件，该文本文件的每行代表一个整数，整数是从0到512之间的一个随机数，现在要求随该文件中的数字进行排序，不能使用堆内存，只能使用栈内存；

- 此例子为将一大串的随机数写入文件
```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<time.h>

int main(void)
{
   //生成一堆随机数
  srand((unsigned int)time(NULL));
  int
i;}  
  //将随机数写入该文件
  FILE *p = fopen(C:\\users\\gjx16\\desktop\\a.txt,"w");
  for(i = 0 ; i < 10000000; i++)
    {
      fprintf(p,"%d\n",(int)rand() % 513);//将生成的随机数（范围为0-512）写入文件指针p指向的文件之中
      
    }
     fclose(p);
     printf("end of file");
     return 0;

```


- 此例子为将写入的随机数进行排序（使用栈，不能使用堆）

```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
#include<time.h>

int main(void)
{
   //生成一堆随机数
  srand((unsigned int)time(NULL));
  int i;
  
  //将随机数写入该文件
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","w");
  for(i = 0 ; i < 10000000; i++)//测试的时候写入的数目可以适当的小一点，这样运行起来方便
    {
      fprintf(p,"%d\n",(int)rand() % 513);//将生成的随机数（范围为0-512）写入文件指针p指向的文件之中
      
    }
     fclose(p);

//下面将这个文件进行排序

    p = fopen("C:\\users\\gjx16\\desktop\\a.txt","r");
    int array[513] = {0};

    while(!feof(p))
    {
      char buf[100] = {0};//buf代表的是一行
      fgets(buf,sizeof(buf),p);//得到一行
      if(buf[0] != 0)//看buf的 第一个元素是否为0，为0则表示为空行，就什么都不做
      {
        int value = atoi(buf);//将得到的数转换为整数
        array[value]++;
      }
    
    }
    fclose(p);


    p = fopen("C:\\users\\gjx16\\desktop\\b.txt","w");
    int j;
    for(i = 0; i < 513; i++)
      {
        for(j = 0; j < array[i]; j++)
          {
            fprintf(p,"%d\n",i);
          }
      }
      
    printf("end of file\n");
    return 0;
}
```
程序运行结果：首先将随机数写入a.txt中，然后将a.txt中的数字排序之后放入b.txt中

---
==上面排序方法解释==
有点类似于索引排序，将数组的下标当做一种索引
这里以十个数为蓝本
```c
 if(buf[0] != 0)
      {
        int value = atoi(buf);//将得到的数转换为整数
        array[value]++;
      }     
```
a.txt中的数字为：(共12个数)
`4`
`3`
`4`
`6`
`9`
`3`
`2`
`4`
`1`
`7`
`8`
`5`
刚开始的时候，数组array[]全为0；
读取第一个值的时候，value = 4
然后array[value]++相当于array[4]++,即此时array[4] 的值为1，同理可得读取完之后各个array的值分别如下：
`array[0] = 0`
`array[1] = 1`
`array[2] = 1`
`array[3] = 2`
`array[4] = 3`
`array[5] = 1`
`array[6] = 1`
`array[7] = 1`
`array[8] = 1`
`array[9] = 1`
然后循环
```c
    int j;
    for(i = 0; i < 10; i++)
      {
        for(j = 0; j < array[i]; j++)
          {
            fprintf(p,"%d\n",i);
          }
      }
```
当i = 0 的时候，j = 0,而此时array[i] = array[0] = 0;所以不满足要求退出循环；
当i = 1 的时候，j = 0,而此时array[i] = array[1] = 1;所以循环一次，打印一次1；
当i = 2 的时候，j = 0,而此时array[i] = array[2] = 1;所以循环一次，打印一次2；
等等等等；

---



## **（七）文件位置操作**
- **fseek函数**
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

struct student
  {
    char name[10];
    int age;   
  };

int main(void)
{
  struct student st[10] ={0};
  int i = 0;
  for(i = 0 ; i < 10; i++)
  {
     printf("please input the name:");
     scanf("%s",st[i].name);//这里前面不需要加上&
     printf("please input the age:");
     scanf("%s",&st[i].age);//这里前面不需要加上&
     printf("\n");
  }
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.bat","wb");
  fwrite(st,sizeof(struct student),10,p);
  fclose(p);
  return 0;
}


```
程序输出结果：将输入的值均放入文件a.bat中

**下面将读写这个文件中的内容**
```c

#include<stdio.h>
#include<string.h>
#include<stdlib.h>

struct student
  {
    char name[10];
    int age;   
  };


int main()
{
  struct student st = {0};
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\a.bat","rb");

  //两种遍历读取方法；
  //方法一：该方法用的最多，但是这里会将最后一行的空行读取，造成输出程序最后一行中对应元素的值均为0
  while(!feof(p))
    {
      memset(&st,0,sizeof(struct student));
      fread(&st,sizeof(struct student),1,p);//一次读取一行的结构
      printf("name = %s, age = %d\n",st.name,st.age); 
    }

//第二种读取方法
    while(1)
      {
          memset(&st,0,sizeof(struct student));
          if(fread(&st,sizeof(struct student),1,p) == 0)//判断这一行是否为空，如果为空则直接不读
            break;
            printf("name = %s,age = %d\n",st.name,st.age);      
      }


    fclose(p);
    return 0;

}
```
程序输出结果：~~这里的年龄输出一直有问题，程序有待改进~~

**如果想从任意地方进行读取则必须采用`fseek` 函数**
知识点注释；
|参数|意义|
---|---|
SEEK_SET|从文件开始|
SEEK_CUR|从文件当前位置|
SEEK_END|从文件结尾位置|


```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>

struct student
  {
    char name[10];
    int age;   
  };


int main()
{
  struct student st = {0};
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\a.bat","rb");

//位置一：以下所有的fseek不重复，这里举例程序中只有一个在起作用
  fseek(p,sizeof(struct student)*2,SEEK_SET);
  //参数含义；文件地址、偏移的量、从哪里开始
  //偏移了两个student结构体那么长



//这时候读取应该是从第三个结构体开始读取，跳过了前两个  
  memset(&st,0,sizeof(struct student));
  fread(&st,sizeof(struct student),1,p);//一次读取一行的结构
  printf("name = %s, age = %d\n",st.name,st.age); 

//位置二：
 fseek(p,sizeof(struct student),SEEK_SET);//从当前位置跳过一个结构长度，相当于直接跳过第二个结构，读取第三个结构

//位置三；
 fseek(p,0 - sizeof(struct student),SEEK_SET);//从当前位置往回跳一个结构，结果就是读取了两个第一个结构，然后读取第二个结构
 
  memset(&st,0,sizeof(struct student));
  fread(&st,sizeof(struct student),1,p);//一次读取一行的结构
  printf("name = %s, age = %d\n",st.name,st.age); 

  memset(&st,0,sizeof(struct student));
  fread(&st,sizeof(struct student),1,p);//一次读取一行的结构
  printf("name = %s, age = %d\n",st.name,st.age); 
  
    fclose(p);
    return 0;

}
```
以上程序主要针对于二进制文件，而此函数同样适用于文本文件

---

**以文本文件为例：**
```c
#include<stdio.h>
#include<stdlib.h>

int main()
{
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","rb");//虽然是文本文件，这里仍然采用二进制的方式进行读取
  
//采用此种方式见第一个截图
fseek(p,-5,SEEK_END);//从文件结尾位置回滚5个字节再行读取

//采用此种方式见第二个截图
fseek(p,5,SEEK_SET);//从文件开始位置之后跳过5个字节之后在进行读取

char buf[100] = {0};//定义一个数组用于存放从文件中读取的数值
fgets(buf,sizeof(buf),p);
printf("buf = %s\n",buf);
fclose(p);
return 0;

}
```
程序运行结果为；
![进行回滚操作时]($resource/1.png)

![从开始位置跳过五个字节之后]($resource/%E4%BB%8E%E5%BC%80%E5%A7%8B%E4%BD%8D%E7%BD%AE%E8%B7%B3%E8%BF%87%E4%BA%94%E4%B8%AA%E5%AD%97%E8%8A%82%E4%B9%8B%E5%90%8E.png)


**使用ftell函数可以获得当前读取的位置（第几个字节）**
函数`ftell`用于得到文件位置指针当前位置相对于文件首的偏移字节数
```c
#include<stdio.h>
#include<stdlib.h>

int main()
{
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","rb");
  fseek(p,5,SEEK_SET);
  char buf[100] = {0};//定义一个数组用于存放从文件中读取的数值
  fgets(buf,sizeof(buf),p);
  printf("buf = %s\n",buf);
  printf("ftell = %d\n",ftell);
  fclose(p);
  return 0;
}
```
程序运行结果为：（以文本a.txt为例）
![ftell确定目前为止]($resource/ftell%E7%A1%AE%E5%AE%9A%E7%9B%AE%E5%89%8D%E4%B8%BA%E6%AD%A2.png)
注释：跳过开始两个开始读取，然后`abcdef`共6个字节，但是一行读完之后，后面还有`\r`和`\n`因此一共显示为8个字节。  

---

**函数 `fflush` 的使用：**
函数功能为将缓冲区的内容立刻写入磁盘文件（其实该函数内部也有一个很小的缓冲区，并未实现真正的时时写入，只是这个缓冲区并不大）

- 不加函数`fflush`时候
```c
#include<stdio.h>

int main()
{
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\a.txt","w");//以文本文件的方式写文件,没文件先创文件，有文件先清空然后重新写入内容

  while(1)
  {
    char buf[100] = {0};
    printf("please input the content:\n");
    scanf("%s",buf);
    if(strcmp(buf,"exit") == 0 )//如果用户输入exit，则退出
    break;
    fprintf(p,"%s\n",buf);//将buf中的数据写入文件
  }
  fclose(p);
  return 0;
}
```
注释：这里程序运行时候，假设该文件已存在，则会首先清空该文件，
当用户输入数据的时候，并没有时时的写入进文件，而是存入缓冲区之中，只有在运行到`fclose`之后才真正的将数据写入文件之中，或者当缓冲区满的时候写入；
这样避免了多次读写对于硬盘寿命的影响

![函数执行数据读写操作的流程图]($resource/%E7%BC%93%E5%86%B2%E5%8C%BA.png)

- 加入函数之后

```c
#include<stdio.h>

int main()
{
  FILE *p = fopen("c:\\users\\gjx16\\desktop\\a.txt","w");

  while(1)
  {
    char buf[100] = {0};
    printf("please input the content:\n");
    scanf("%s",buf);
    if(strcmp(buf,"exit") == 0 )//如果用户输入exit，则退出
    break;
    fprintf(p,"%s\n",buf);//将buf中的数据写入文件
    fflush(p);
  }
  fclose(p);
  return 0;
}
```
程序运行结果；只要用户输入数据，该数据就会时时的写入文件

**fflush的优劣：**
- 优势：
    不会因为断电或者死机等等故障造成缓冲区的数据的丢失
    
- 劣势：
    硬盘读写次数增加，导致程序效率低下并且硬盘使用寿命变短
    
- 作用：
    用于修改配置文件的时候，用于一些不常修改的数据但是很重要的数据的时候


## （八）文件删除以及重命名

- **remove函数**
```c
#include<stdlio.h>

int main()
{
  remove("C:\\users\\gjx16\\desktop\\a.txt");//删除该文件
  retrurn 0;
}
```
   
- **rename函数**
```c
#include<stdlio.h>

int main()
{
 rename("C:\\users\\gjx16\\desktop\\a.txt","C:\\users\\gjx16\\desktop\\b.txt");//将a.txt修改为b.txt
  retrurn 0;
}

```

**二进制文件排序：**
```c
#include<stdio.h>
#include<string.h>

struct  student
{
  char  name[5];
  int  age;
};

void  swap(struct  student  *a,struct  student  *b)
{
  struct  student  tmp  =  *a;
  *a  =  *b;
  *b  =  tmp;
}


void  bubble(struct  student  *p,int  n)
{
  int  i;
  int  j;
  for(i  =  0;  i  <  n;  i++)
    {
      for(j  =  0;  j  <  n-i;  j++)
        {
          if(p[j-1].age  >p[j].age)
            {
              swap(&p[j-1],&p[j]);
            }
        }
    }
}



int  main(void)
{
  struct  student  st[5]  ={0};
  FILE  *p  =  fopen("c:\\users\\gjx16\\desktop\\a.txt","wb");
  int  i  =  0;
  for(i  =  0  ;  i  <  5;  i++)
     {
        printf("please  input  the  name:");
        scanf("%s",st[i].name);//这里前面不需要加上&
        printf("please  input  the  age:");
        scanf("%s",&st[i].age);//这里前面不需要加上&
        printf("\n");
     }

  fwrite(st,sizeof(struct  student),5,p);

  fclose(p);

  //下面读取文件，因为这是小文件，可以使用循环读取或者一次性读取

  int  i1;
  for(i1  =  0;  i1  <5;  i1++)
    {
        fread(&st[i1],sizeof(struct  student),1,p);
    }

//一次性的读取

//fread(st,sizeof(struct  student),5,p);
  fclose(p);

  bubble(st,5);

//输出到新的文件

  /*  for(i  =  0  ;  i  <5;  i++)
  *      {
  *        printf("name  =  %s,age  =  %d\n",st[i].name,st[i].age);
  *      }
  */

  FILE  *p1  =  fopen("c:\\users\\gjx16\\desktop\\b.txt","wb");
  fwrite(st,sizeof(struct  student),5,p1);

  fclose(p1);
  return  0;

}

```



总结；
将结构体写入二进制文件
```c
#include<stdio.h>
#include<string.h>


struct student
{
  char  name[100];
  int age;
};


int main()
{
  struct student st[3] = {{"aa",11},{"bb",22},{"cc",33}};
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","wb");
  fwrite(st,sizeof(struct student),3,p);
  fclose(p);
  return 0;

}
```
用结构体读出二进制文件
```c
#include<stdio.h>
#include<string.h>


struct student
{
  char  name[100];
  int age;
};


int main()
{
  struct student st[3] = {0};
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","rb+");//以二进制同时进行读写
  fseek(p,sizeof(struct student),SEEK_SET);//从开始位置后移一个结构的大小
  struct student s = {"tom",55};
  fwrite(&s,sizeof(struct student),1,p);//将这个结构体的元素写入
  fseek(p,0,SEEK_SET);//返回到结构体的最开头指针，否则下面的循环不能从头开始
   fread(st,sizeof(struct student),3,p);

int i;
for(i = 0; i< 3; i++)
{
  printf("name = %s,age = %d\n",st[i].name,st[i].age);
}
  fclose(p);
  return 0;

}
```
程序输出结果为：
![结构体以及二进制文件]($resource/%E7%BB%93%E6%9E%84%E4%BD%93%E4%BB%A5%E5%8F%8A%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%96%87%E4%BB%B6.png)

**一般读取为一行一行的读取和写入，也可以实现一次性的写入**
因为数组所能放置的元素数量有限，因此只能是在数量较少的时候使用
```c
#include<stdio.h>
#include<string.h>

int main()
{
  FILE *p = fopen("C:\\users\\gjx16\\desktop\\a.txt","r");
  char content[1024] ={0};
  while(!feof(p))
    {
      char buf[100] = {0};
      fgets(buf,sizeof(buf),p);//将文件中读取的内容放入buf中，因为是按照一行一行的读取
      strcat(content,buf);//将数组中的元素连接起来
    }
    printf("%s",content);
    fclose(p);
    return 0;
  
  }

}
```









