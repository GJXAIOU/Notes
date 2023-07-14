# Week 3 选择结构
**自上而下的编程思想:**    
1.首先可以清晰并且精确的陈述你的问题  
2.定义程序所需的输入量和输出量  
3.设计程序所需的算法（将过程分解并且逐步求精）    
4.把算法转换为代码  
5.检测产生的MATLAB程序(先进行单元检测，然后组合进行整体检测)  
# 运算符
## 关系运算符  
==  ~=  >  <  >=  <=  
结果为true or false  
> 关系运算在所有数学计算之后进行，因此关系运算符两边加不加括号都行  
---
**注意**  
~=和==在比较字符串时候是安全的，但是比较数据时候当心，因为精确度的问题会产生round off错误，例如0==sin(pi)的值为0  
---
## 逻辑运算符  
& 逻辑与  | 逻辑或  xor 逻辑与或  ~ 逻辑非  

**PS:标量与数组之间同样可以进行关系运算与逻辑运算（标量与数组元素挨个进行比较）**  
**PS：最好在一个if结构里面使用多个elseif结构，尽量少用if的嵌套结构  

## 逻辑函数  
常见的逻辑函数  
函数  |用途
---|---|---
ischar(a) | a 是字符数组返回 1，否则返回 0 
isempty(a) | a 是空数组返回 1，否则返回 0 
isinf(a)  |a 是无穷大，则返回 1，否则返回 0 
isnan(a) | a 不是一个数则返 1，否则返回 0 
isnumeric(a)|  a 是一个数值数组返回 1，否则返回 0 
---
## 选择结构  
> ps:blocks(语句块)  
1.if结构体经常缩进2~3个空格  

18712112422

## 数据输入
### 方法一 使用内置函数初始化
### 方法二 使用关键字input初始化
使用方法：A=input(需要输入的语句)  

## 数据存储
MATLAB以列为主导顺序存储数组中的元素，多维数组在内存中的顺序为数组第一个下标增长最快，其次为第二个。。。。  



## 撰写一个程序
1.程序以.m后缀存储

2.左上角，new script 新建一个脚本
执行：点击运行或者F5

3.保存文件名，不能数字开头，区分大小写

4.注解为%，多行注解为选中多行，然后点击comment（注解）按钮
%%可以把程序分节，然后选择运行节，可以运行部分程序

5.在行数旁边的短横线上点击一些，或者点击设置断点,  
-再选择run时候，在断点处出结果， 然后点击continue时候可以继续运行
程序缩排，ctrl+l快捷键，或者全选右击然后选择
## 结构化程序
1.逻辑结构  
== equal to  ~= not equal to && and  || or  
%% rem(a, 2)==0 表示a与2的余数是不是0  
disp('')显示''中结果    

2.if elseif else  
if condition1  
statement 1  
elseif condition2  
statement 2  
else statement3  
end    
---

3.switch expression  根据expression结果找case  
case value1  
statement 2    
...
otherwise statement    
end  
---

4.try/catch结构
try  
statement 1  
statement 2  
...
catch  
statement 1
statement 2  
...
end  

**先执行try语句块的语句，没错误则跳过catch语句块，有错误的话，则停止执行下面的try语句块，并且立即执行catch语句块**  

---

4.while  
while expression  
statement  
end  
---

%% prod（1:n）即是n!  
1e100即是10^100  

5.for variable=start:increment:end
commands
end
  s(n)=1:n   disp(s)
  ---
3.
## 撰写函数