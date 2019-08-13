# C++编译出错分析

所谓编译错误就是我们平常所说的bug。对于初级程序员来说，bug是常伴身边的，不夸张地说，写代码和改bug耗费的时间是一样的。而看完此系列文章后，便能读懂这些英文编译错误，有针对性地修改程序，大大缩短改bug的时间，从而省出更多时间学习新知识。



**1 fatal error C1003: error count exceeds number; stopping compilation**

中文对照：（编译错误）错误太多，停止编译

分析：修改之前的错误，再次编译

**2 fatal error C1004: unexpected end of file found**

中文对照：（编译错误）文件未结束

分析：一个函数或者一个结构定义缺少“}”、或者在一个函数调用或表达式中括号没有配对出现、或者注释符“/*…*/”不完整等

**3 fatal error C1083: Cannot open include file: 'xxx': No such file ordirectory**

中文对照：（编译错误）无法打开头文件xxx：没有这个文件或路径

分析：头文件不存在、或者头文件拼写错误、或者文件为只读

**4 fatal error C1903: unable to recover from previous error(s); stoppingcompilation**

中文对照：（编译错误）无法从之前的错误中恢复，停止编译

分析：引起错误的原因很多，建议先修改之前的错误

**5 error C2001: newline in constant**

中文对照：（编译错误）常量中创建新行

分析：字符串常量多行书写

**6 error C2006: #include expected a filename, found 'identifier'**

中文对照：（编译错误）#include命令中需要文件名

分析：一般是头文件未用一对双引号或尖括号括起来，例如“#include stdio.h”

**7 error C2007: #define syntax**

中文对照：（编译错误）#define语法错误

分析：例如“#define”后缺少宏名，例如“#define”

**8 error C2008: 'xxx' : unexpected in macro definition**

中文对照：（编译错误）宏定义时出现了意外的xxx

分析：宏定义时宏名与替换串之间应有空格，例如“#define TRUE"1"”

**9 error C2009: reuse of macro formal 'identifier'**

中文对照：（编译错误）带参宏的形式参数重复使用

分析：宏定义如有参数不能重名，例如“#define s(a,a) (a*a)”中参数a重复

**10 error C2010: 'character' : unexpected in macro formal parameter list**

中文对照：（编译错误）带参宏的形式参数表中出现未知字符

分析：例如“#define s(r|) r*r”中参数多了一个字符‘|’

**11 error C2014: preprocessor command must start as first nonwhite space**

中文对照：（编译错误）预处理命令前面只允许空格

分析：每一条预处理命令都应独占一行，不应出现其他非空格字符

**12 error C2015: too many characters in constant**

中文对照：（编译错误）常量中包含多个字符

分析：字符型常量的单引号中只能有一个字符，或是以“\”开始的一个转义字符，例如“charerror = 'error';”

**13 error C2017: illegal escape sequence**

中文对照：（编译错误）转义字符非法

分析：一般是转义字符位于 ' ' 或 "" 之外，例如“char error = ' '\n;”

**14 error C2018: unknown character '0xhh'**

中文对照：（编译错误）未知的字符0xhh

分析：一般是输入了中文标点符号，例如“char error = 'E'；”中“；”为中文标点符号

**15 error C2019: expected preprocessor directive, found 'character'**

中文对照：（编译错误）期待预处理命令，但有无效字符

分析：一般是预处理命令的#号后误输入其他无效字符，例如“#!define TRUE 1”

**16 error C2021: expected exponent value, not 'character'**

中文对照：（编译错误）期待指数值，不能是字符

分析：一般是浮点数的指数表示形式有误，例如123.456E

**17 error C2039: 'identifier1' : is not a member of 'identifier2'**

中文对照：（编译错误）标识符1不是标识符2的成员

分析：程序错误地调用或引用结构体、共用体、类的成员

**18 error C2041: illegal digit 'x' for base 'n'**

中文对照：（编译错误）对于n进制来说数字x非法

分析：一般是八进制或十六进制数表示错误，例如“int i = 081;”语句中数字‘8’不是八进制的基数

**19 error C2048: more than one default**

中文对照：（编译错误）default语句多于一个

分析：switch语句中只能有一个default，删去多余的default

**20 error C2050: switch expression not integral**

中文对照：（编译错误）switch表达式不是整型的

分析：switch表达式必须是整型（或字符型），例如“switch ("a")”中表达式为字符串，这是非法的

**21 error C2051: case expression not constant**

中文对照：（编译错误）case表达式不是常量

分析：case表达式应为常量表达式，例如“case "a"”中“"a"”为字符串，这是非法的

**22 error C2052: 'type' : illegal type for case expression**

中文对照：（编译错误）case表达式类型非法

分析：case表达式必须是一个整型常量（包括字符型）

**23 error C2057: expected constant expression**

中文对照：（编译错误）期待常量表达式

 分析：一般是定义数组时数组长度为变量，例如“int n=10; int a[n];”中n为变量，这是非法的

**24 error C2058: constant expression is not integral**

中文对照：（编译错误）常量表达式不是整数

分析：一般是定义数组时数组长度不是整型常量

**25 error C2059: syntax error : 'xxx'**

中文对照：（编译错误）‘xxx’语法错误

分析：引起错误的原因很多，可能多加或少加了符号xxx

**26 error C2064: term does not evaluate to a function**

中文对照：（编译错误）无法识别函数语言

分析：1、函数参数有误，表达式可能不正确，例如“sqrt(s(s-a)(s-b)(s-c));”中表达式不正确 2、变量与函数重名或该标识符不是函数，例如“int i,j; j=i();”中i不是函数

**27 error C2065: 'xxx' : undeclared identifier**

中文对照：（编译错误）未定义的标识符xxx

分析：1、如果xxx为cout、cin、scanf、printf、sqrt等，则程序中包含头文件有误 2、未定义变量、数组、函数原型等，注意拼写错误或区分大小写。

**28 error C2078: too many initializers**

中文对照：（编译错误）初始值过多

分析：一般是数组初始化时初始值的个数大于数组长度，例如“int b[2]={1,2,3};”

**29 error C2082: redefinition of formal parameter 'xxx'**

中文对照：（编译错误）重复定义形式参数xxx

分析：函数首部中的形式参数不能在函数体中再次被定义

**30 error C2084: function 'xxx' already has a body**

中文对照：（编译错误）已定义函数xxx

分析：在VC++早期版本中函数不能重名，6.0版本中支持函数的重载，函数名可以相同但参数不一样

**31 error C2086: 'xxx' : redefinition**

中文对照：（编译错误）标识符xxx重定义

分析：变量名、数组名重名

**32 error C2087: '<Unknown>' : missing subscript**

中文对照：（编译错误）下标未知

分析：一般是定义二维数组时未指定第二维的长度，例如“int a[3][];”

**33 error C2100: illegal indirection**

中文对照：（编译错误）非法的间接访问运算符“*”

分析：对非指针变量使用“*”运算

**34 error C2105: 'operator' needs l-value**

中文对照：（编译错误）操作符需要左值

分析：例如“(a+b)++;”语句，“++”运算符无效

**35 error C2106: 'operator': left operand must be l-value**

中文对照：（编译错误）操作符的左操作数必须是左值 分析：

例如“a+b=1;”语句，“=”运算符左值必须为变量，不能是表达式

**36 error C2110: cannot add two pointers**

中文对照：（编译错误）两个指针量不能相加

分析：例如“int *pa,*pb,*a; a = pa + pb;”中两个指针变量不能进行“+”运算

**37 error C2117: 'xxx' : array bounds overflow**

中文对照：（编译错误）数组xxx边界溢出

分析：一般是字符数组初始化时字符串长度大于字符数组长度，例如“char str[4] = "abcd";”

**38 error C2118: negative subscript or subscript is too large**

中文对照：（编译错误）下标为负或下标太大

分析：一般是定义数组或引用数组元素时下标不正确 error C2124: divide or mod by zero 中文对照：（编译错误）被零除或对0求余 分析：例如“int i = 1 / 0;”除数为0

**39 error C2133: 'xxx' : unknown size**

中文对照：（编译错误）数组xxx长度未知

分析：一般是定义数组时未初始化也未指定数组长度，例如“int a[];”

**40 error C2137: empty character constant****。**

中文对照：（编译错误）字符型常量为空

分析：一对单引号“''”中不能没有任何字符

**41 error C2143: syntax error : missing 'token1' before 'token2'**

**42 error C2146: syntax 4error : missing 'token1' before identifier 'identifier'**

中文对照：（编译错误）在标识符或语言符号2前漏写语言符号1

分析：可能缺少“{”、“)”或“；”等语言符号

**43 error C2144: syntax error : missing ')' before type 'xxx'**

中文对照：（编译错误）在xxx类型前缺少‘）’

分析：一般是函数调用时定义了实参的类型

**44 error C2181: illegal else without matching if**

中文对照：（编译错误）非法的没有与if相匹配的else

分析：可能多加了“；”或复合语句没有使用“{}”

**45 error C2196: case value '0' already used**

中文对照：（编译错误）case值0已使用

分析：case后常量表达式的值不能重复出现

**46 error C2296: '%' : illegal, left operand has type 'float'**

**47 error C2297: '%' : illegal, right operand has type 'float'**

中文对照：（编译错误）%运算的左(右)操作数类型为float，这是非法的

分析：求余运算的对象必须均为int类型，应正确定义变量类型或使用强制类型转换

**48 error C2371: 'xxx' : redefinition; different basic types**

中文对照：（编译错误）标识符xxx重定义；基类型不同

分析：定义变量、数组等时重名

**49 error C2440: '=' : cannot convert from 'char [2]' to 'char'**

中文对照：（编译错误）赋值运算，无法从字符数组转换为字符

分析：不能用字符串或字符数组对字符型数据赋值，更一般的情况，类型无法转换

**50 error C2447: missing function header (old-style formal list?)**

**51 error C2448: '<Unknown>' : function-style initializer appears to be a function definition**

中文对照：（编译错误）缺少函数标题(是否是老式的形式表？)

分析：函数定义不正确，函数首部的“( )”后多了分号或者采用了老式的C语言的形参表

**52 error C2450: switch expression of type 'xxx' is illegal**

中文对照：（编译错误）switch表达式为非法的xxx类型

分析：switch表达式类型应为int或char

**53 error C2466: cannot allocate an array of constant size 0**

中文对照：（编译错误）不能分配长度为0的数组

分析：一般是定义数组时数组长度为0

**54 error C2601: 'xxx' : local function definitions are illegal**

中文对照：（编译错误）函数xxx定义非法

分析：一般是在一个函数的函数体中定义另一个函数

**55 error C2632: 'type1' followed by 'type2' is illegal**

中文对照：（编译错误）类型1后紧接着类型2，这是非法的

分析：例如“int float i;”语句

**56 error C2660: 'xxx' : function does not take n parameters**

中文对照：（编译错误）函数xxx不能带n个参数

分析：调用函数时实参个数不对，例如“sin(x,y);”

**57 error C2664: 'xxx' : cannot convert parameter n from 'type1' to 'type2'**

中文对照：（编译错误）函数xxx不能将第n个参数从类型1转换为类型2

分析：一般是函数调用时实参与形参类型不一致

**58 error C2676: binary '<<' : 'class istream_withassign' does not define this operator or a conversion to a type acceptable to the predefined operator error C2676: binary '>>' : 'class ostream_withassign' does not define this operator or a conversion to a type acceptable to the predefined operator**

分析：“>>”、“<<”运算符使用错误，例如“cin<<x; cout>>y;”

**59 error C4716: 'xxx' : must return a value**

中文对照：（编译错误）函数xxx必须返回一个值

分析：仅当函数类型为void时，才能使用没有返回值的返回命令。

**60 fatal error LNK1104: cannot open file "Debug/Cpp1.exe"**

中文对照：（链接错误）无法打开文件Debug/Cpp1.exe

分析：重新编译链接

**61 fatal error LNK1168: cannot open Debug/Cpp1.exe for writing**

中文对照：（链接错误）不能打开Debug/Cpp1.exe文件，以改写内容。

分析：一般是Cpp1.exe还在运行，未关闭

**62 fatal error LNK1169: one or more multiply defined symbols found**

中文对照：（链接错误）出现一个或更多的多重定义符号。

分析：一般与error LNK2005一同出现

**63 error LNK2001: unresolved external symbol _main**

中文对照：（链接错误）未处理的外部标识main

分析：一般是main拼写错误，例如“void mian()”

**64 error LNK2005: _main already defined in Cpp1.obj**

中文对照：（链接错误）main函数已经在Cpp1.obj文件中定义

分析：未关闭上一程序的工作空间，导致出现多个main函数

**65 warning C4003: not enough actual parameters for macro 'xxx'**

中文对照：（编译警告）宏xxx没有足够的实参

分析：一般是带参宏展开时未传入参数

**66 warning C4067: unexpected tokens following preprocessor directive - expected a newline**

中文对照：（编译警告）预处理命令后出现意外的符号 - 期待新行

分析：“#include<iostream.h>;”命令后的“；”为多余的字符

**67 warning C4091: '' : ignored on left of 'type' when no variable is declared**

中文对照：（编译警告）当没有声明变量时忽略类型说明

分析：语句“int ;”未定义任何变量，不影响程序执行

**68 warning C4101: 'xxx' : unreferenced local variable**

中文对照：（编译警告）变量xxx定义了但未使用

分析：可去掉该变量的定义，不影响程序执行

**69 warning C4244: '=' : conversion from 'type1' to 'type2', possible loss of data**

中文对照：（编译警告）赋值运算，从数据类型1转换为数据类型2，可能丢失数据

分析：需正确定义变量类型，数据类型1为float或double、数据类型2为int时，结果有可能不正确，数据类型1为double、数据类型2为float时，不影响程序结果，可忽略该警告

**70 warning C4305: 'initializing' : truncation from 'const double' to 'float'**

中文对照：（编译警告）初始化，截取双精度常量为float类型

分析：出现在对float类型变量赋值时，一般不影响最终结果

**71 warning C4390: ';' : empty controlled statement found; is this the intent?**

中文对照：（编译警告）‘；’控制语句为空语句，是程序的意图吗？

分析：if语句的分支或循环控制语句的循环体为空语句，一般是多加了“；”

**72 warning C4508: 'xxx' : function should return a value; 'void' return type assumed**

中文对照：（编译警告）函数xxx应有返回值，假定返回类型为void

分析：一般是未定义main函数的类型为void，不影响程序执行 c语言的错误对照表———— 在遇到错误时可以对照查看

**73 warning C4552: 'operator' : operator has no effect; expected operator with side-effect**

中文对照：（编译警告）运算符无效果；期待副作用的操作符

分析：例如“i+j;”语句，“+”运算无意义

**74 warning C4553: '==' : operator has no effect; did you intend '='?**

中文对照：（编译警告）“==”运算符无效；是否为“=”？

分析：例如 “i==j;” 语句，“==”运算无意义

**75 warning C4700: local variable 'xxx' used without having been initialized**

中文对照：（编译警告）变量xxx在使用前未初始化

分析：变量未赋值，结果有可能不正确，如果变量通过scanf函数赋值，则有可能漏写“&”运算符，或变量通过cin赋值，语句有误

**76 warning C4715: 'xxx' : not all control paths return a value**

中文对照：（编译警告）函数xxx不是所有的控制路径都有返回值

分析：一般是在函数的if语句中包含return语句，当if语句的条件不成立时没有返回值

**77 warning C4723: potential divide by 0**

中文对照：（编译警告）有可能被0除

分析：表达式值为0时不能作为除数

**78 warning C4804: '<' : unsafe use of type 'bool' in operation**

中文对照：（编译警告）‘<’：不安全的布尔类型的使用

分析：例如关系表达式“0<=x<10”有可能引起逻辑错误


