---
tags: [未看]
---
# Linux查看日志命令




   当日志文件存储日志很大时，我们就不能用vi直接进去查看日志，需要Linux的命令去完成我们的查看任务.



```
Log位置：

/var/log/message    系统启动后的信息和错误日志，是Red Hat Linux中最常用的日志之一   
/var/log/secure 与安全相关的日志信息   
/var/log/maillog    与邮件相关的日志信息   
/var/log/cron   与定时任务相关的日志信息   
/var/log/spooler    与UUCP和news设备相关的日志信息   
/var/log/boot.log   守护进程启动和停止相关的日志消息 
```

* * *

*   **tail** 

 参数： 
tail [ -f ] [ -c Number | -n Number | -m Number | -b Number | -k Number ] [ File ] 
参数解释： 
-f 该参数用于监视File文件增长。 
-c Number 从 Number 字节位置读取指定文件 
-n Number 从 Number 行位置读取指定文件。 
-m Number 从 Number 多字节字符位置读取指定文件，比方你的文件假设包括中文字，假设指定-c参数，可能导致截断，但使用-m则会避免该问题。 
-b Number 从 Number 表示的512字节块位置读取指定文件。 
-k Number 从 Number 表示的1KB块位置读取指定文件。 
File 指定操作的目标文件名称 
上述命令中，都涉及到number，假设不指定，默认显示10行。Number前面可使用正负号，表示该偏移从顶部还是从尾部開始计算。 
tail可运行文件一般在/usr/bin/以下。

```
实例：  
1、tail -f filename  
说明：监视filename文件的尾部内容（默认10行，相当于增加参数 -n 10），刷新显示在屏幕上。退出，按下CTRL+C。  

2、tail -n 20 filename  
说明：显示filename最后20行。  

3、tail -r -n 10 filename  
说明：逆序显示filename最后10行。
```

*   **head**

```
head 仅仅显示前面几行  

head -n 10  test.log   查询日志文件中的头10行日志;  

head -n -10  test.log   查询日志文件除了最后10行的其他所有日志;  
```

*   **grep**

```
grep [options]  
主要参数:  
[options]主要参数：  
－c：只输出匹配行的计数。  
－I：不区分大 小写(只适用于单字符)。  
－h：查询多文件时不显示文件名。  
－l：查询多文件时只输出包含匹配字符的文件名。  
－n：显示匹配行及 行号。  
－s：不显示不存在或无匹配文本的错误信息。  
－v：显示不包含匹配文本的所有行。  
pattern正则表达式主要参数：  
： 忽略正则表达式中特殊字符的原有含义。  
^：匹配正则表达式的开始行。  
$: 匹配正则表达式的结束行。  
<：从匹配正则表达 式的行开始。  
>：到匹配正则表达式的行结束。  
[ ]：单个字符，如[A]即A符合要求 。  
[ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求 。  
。：所有的单个字符。  
 - ：有字符，长度可以为0。
```

*   **sed**

```
用sed命令  
sed -n '5,10p' filename 这样你就可以只查看文件的第5行到第10行。  
```

*   **cat**

```
cat主要有三大功能：  
1.一次显示整个文件。$ cat filename  
2.从键盘创建一个文件。$ cat > filename   
  只能创建新文件,不能编辑已有文件.  
3.将几个文件合并为一个文件： $cat file1 file2 > file  

参数：  
-n 或 --number 由 1 开始对所有输出的行数编号  
-b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号  
-s 或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行  
-v 或 --show-nonprinting  
例：  
把 textfile1 的档案内容加上行号后输入 textfile2 这个档案里  
cat -n textfile1 > textfile2  

把 textfile1 和 textfile2 的档案内容加上行号（空白行不加）之后将内容附加到 textfile3 里。  
cat -b textfile1 textfile2 >> textfile3  

把test.txt文件扔进垃圾箱，赋空值test.txt  
cat /dev/null > /etc/test.txt   
注意：>意思是创建，>>是追加。千万不要弄混了。  
```

*   **tac (反向列示)**

```
tac 是将 cat 反写过来，所以他的功能就跟 cat 相反， cat 是由第一行到最后一行连续显示在萤幕上，  
而 tac 则是由最后一行到第一行反向在萤幕上显示出来！ 
```

*   **混合使用命令**

```
A.  tail web.2016-06-06.log -n 300 -f  
    查看底部即最新300条日志记录，并实时刷新      

B.  grep 'nick' | tail web.2016-04-04.log -C 10   
    查看字符‘nick’前后10条日志记录, 大写C  

C.  cat -n test.log |tail -n +92|head -n 20  
    tail -n +92表示查询92行之后的日志  
    head -n 20 则表示在前面的查询结果里再查前20条记录  
```