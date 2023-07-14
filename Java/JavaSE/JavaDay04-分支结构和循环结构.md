---
tags : 
- java基础

flag: blue
---

@toc


# JavaDay04 分支结构和循环结构
## 一、分支结构
  代码中有三大结构：
      顺序结构，分支结构和循环结构
      
  if (判断语句 true/false) {
      //语句体
  }
  运行流程：
      当程序运行到if语句，首先判断if 之后的括号里面的内容是否为true，如果为true执行语句体，如果为false，
      直接执行大括号之后的语句

```java
  
if (判断语句 true/false) {
    //true 语句体
} else {
    //false 语句体
}
```
  运行流程：
      当程序运行到if-else语句，首先判断if之后的括号里面的内容是否为true，如果为true执行true 语句体，
      如果为false 执行false语句体
        
- 闰年条件：
  能被400整除。或者能被4整除但是不能被100整除。
  int year = 2000;
  
  //闰年条件的代码展示
 ` year % 400 == 0 || (year % 4 == 0 && year % 100 != 0) `

```JAVA
  if (条件1) {
      //语句体1
  } else if (条件2) {
      //语句体2
  } else if (条件3) {
      //语句体3
  ……………………
      
  } else {
      //语句体n
  }
```

  运行流程：
      当程序运行到if - else if结构，首先进行匹配if之后括号里面的条件，找到第一个匹配的选项，执行
      当前if对应的语句体，跳出大括号。如果没有任何一个条件匹配，执行else里面的语句体n
      
  学生成绩：
      90以上  优 关系运算符  score >= 90
      80以上  良
      70以上  中
      60以上  及格
      60以下  叫家长过来~~~
          
- Scanner的使用：
      从键盘上获取用户输入的数据的一种方式：
          是目前没有办法的办法，正常的做开发是通过前端页面发送数据到后台
1. 导包，给当前程序提供技能点
    在程序的开头 clase之前写上
    import java.util.Scanner;
2. 创建Scanner扫描器的变量
    Scanner sc = new Scanner(System.in); 
3. 使用Scanner里面的nestInt();方法获取用户在键盘上输入的int类型数据
    num = sc.nextInt();
        sc.nextFloat();
        sc.nextShort();
        sc.nextLine();
        

**参数合法性判断：**
一般程序的运行首先要符合正常的Java语法要求，但是代码是要提供给用户使用的，也要符合生活逻辑，所以在运行的过程中，要添加一些判断，让代码更加符合生活的情况

例如：学生成绩 满分100分的情况下，分数的范围在 0 ~ 100之间，如果超过了这个范围
这个数据就是一个在生活逻辑上非法数据。所以这里要做参数合法性判断。

让代码更加健壮，思维更加严谨
                
switch - case:匹配的操作
```java
switch (变量) {
    case 确定值1: //冒号
        处理方式1;
        break;
        
    case 确定值2: //冒号
        处理方式2;
        break;
    ………………………………
    
    default: //冒号
        最终的处理方式;
        break;
  }
```


执行流程：
      当程序运行到switch - case 结构，直接利用switch之后的变量去case 中做匹配，找到完全匹配的case选项，执行case之后的处理方式，如果没有任何的一个case匹配执行default里面的最终处理方式
        
【注意事项】
  1. 在switch - case结构中，运行的代码只能是在case 到 break之间的代码！！！
  2. break可以省略，但是之前的case会继续往下执行，直到遇到break跳出switch - case 结构
  3. 在case之外的代码是不会被运行的，在Java中这个会被认为是一个无效代码，如果是Eclipse提示是一个unreachable code 无法触及的代码
  4. 在switch-case中不能出现相同的case选项
  5. default可以省略