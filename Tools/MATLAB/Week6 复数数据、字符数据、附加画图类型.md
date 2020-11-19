## 一、复数数据  


### 1.概述    

**1.一般形式：**   

    c=a+bi    

**2.表示形式**   

-  **直角坐标式：**     
    a,b均为实数，以实部和虚部为x,y轴，则复数可以在坐标轴上以一个点来表示。    
-  **极坐标式：** c=z∠θ   其中z表示向量的模，θ代表辐角   
         a=zcosθ，b=zsinθ,z=√（a^2+b^2）,θ=tan^-1 b/a     

**3.复数运算**   

    c1 + c2 = (a1 + a2) + (b1 + b2)i   
    c1 - c2 = (a1 - a2) + (b1 - b2)i   
    c1 × c2 = (a1a2 - b1b2) + (a1b2 + b1a2)i 
    c1
    ---  =
    c2(a1a2+b1b2)   %% 这里公式不对劲   
    (a1  
    2 + b2  
    2)  +    
    (b1a2-a1b2)   
    (a2   
    2+b2   
    2)  i     



### 2.复变量   

**概念：** 复数值赋值给一个变量名，则相当于创建一个复变量。

**关系运算符**   
- == ：判断是否相等（实部和虚部同时）   
- ~= ：判断是否不等 （实部和虚部其一）  
- < > >= <= :只比较复数值的实部部分   




### 3.复函数  
**概念：** 支持复数运算的函数   

- 类型转换函数 ：将复数据转换为实数据   

> **real:** 将复数的实部转换为double型数据   
**image：** 将复数的虚部转换为double型数据    


- 绝对值与辐角函数 ：将函数转换为极坐标形式    

> **abs()：** 公式为：abs(c)=√（a^2+b^2）   

常见的支持复数运算的MATLAB函数：  

函数 | 描述  
|:------:|:---|
conj(c) | 计算 c 的共共轭复数。如果 c=a+bi，那么 conj(c)=a-bi。 
real(c) | 返回复数 c 的实部 
imag(c) | 返回复数 c 的虚部 
isreal(c) | 如果数组 c 中没有一个元素有虚部，函数 isreal(c)将返回 1。所以如果一个数组 c 是复数组成，那么~isreal(c)将返回 1。 
abs(c) | 返回复数 c 模 
angle(c) | 返回复数 c 的幅角，等价于 atan2(imag(c)，real(c))


- 数学函数：支持复数运算的函数   

    例如：指数函数、对数函数、三角函数、平方根函数等等   




### 4.复数数据作图   

     t = 0:pi/20:4*pi; 
    y = exp(-0.2*t) .* (cos(t) + i * sin(t));
    polar(angle(y),abs(y)); 
    title('\bfPlot of Complex Function'); 










## 二、字符串函数   
每一个MATLAB字符串都是一个char型数组，一个字型占两个字节；   
> ischar 函数是专门用于判断当前变量是否为字符数组   



### 1.字符转换函数    

- double：将字符型数据转换为double型，x=double(str)   
- char: 将double型函数转换为字符型数据，x=char(x)   


### 2.创建二维字符数组   
**注意:** 二维数组的两行元素长度必须相等   
使用char函数创建二维数组则不要担心元素的长度问题  
> name=char('hello world','gaojixu')

> name =
hello world
gaojixu  


### 3.字符串的连接   

- strcat函数:  

    可以水平的连接两个字符串，**忽略所有字符串尾端的空格，保留字符串中的空格**；   

**标准格式**     

    result=strcat('string 1','string 2')    

    result =

    string 1string 2



- strvcat函数：  

可以竖直额连接两个字符串，自动的将其转换为二维数组   

**标准格式**     

    result=strvcat('Long String 1','String 2')   

    result =

    Long String 1
    String 2       



### 4.字符串的比较   

常见字符串比较函数   

函数名|功能  
---|---
strcmp |  判断两字符串是否等价 （包括字符串前面或者后面的空格）
strcmpi |  忽略大小写判断两字符串是否等价 （功能同上，但是忽略大小写区别）
strncmp  | 判断两字符串前 n 个字符是否等价 （包括开头的空格）
strncmpi  | 忽略大小写判断两字符串前 n 个字符是否等价 （功能同上，但是忽略大小写区别）    

 **exp:**   
 
    str1='hello';   
    str2='Hello';   
    str3='help';   
    c=strcmp(str1,str2)     //值为逻辑0     
    d=strcmpi(str1,str2)    //值为逻辑1   
    e=strncmp(str1,str3,3)  //值为逻辑1    
    
- **判断单个字符是否相等**   
**exp：**   

    a='fate';  
    b='cake';  
    result=a==b
    
    result=   
          1×4 logical 数组    

            0   1   0   1       
            


**PS:**    

    所有的关系运算符都是对字符所对应的ASCII值进行比较


- **在一字符串内对字符进行判断**   

函数名|功能   
---|---
isletter  | 用来判断一个字符是否为字母 
isspace  | 判断一个字符是否为空白字符（空格，tab，换行符）



**exp**   
    
    str='Rome 23s'    
    result=isletter(str)    
    
    result =   
 
        1×8 logical 数组   

        1   1   1   1   0   0   0   1      
        


- **在一个字符中查找和替换字符**   

    - 函数findstr返回短字符在长字符串中所有的开始位置；     
    
         test='This is a test!';   
         position=findstr(test,'is')      
    
        position =     
           3    6   
          //字符串'is'在test中一共出现两次，开始位置分别为3和6.    
    ---
    
    - 函数strmatch查看二维数组行开头的字符，并且返回那些以指定的字符序列为开头行号；
        
        
        exp：  
        
        str=strvcat('maxarry','min value','max value');        
        result=strmatch('max',str)   

            result =   

             1    
             3      
    
    ---

    - 函数strrep用于标准的查找和替换工作，他能找到一个字符串中所有另一个字符串，并被第三个字符串替换；   
        
        exp：   
        
        
        str='This is a test';   
        result= strrep(str,'test','pest')   
        
        result =   

        This is a pest      

    -函数strtok返回输入字符串中第一次出现在分隔符前面的所有字符；   
    默认格式为:[token,remainder]=strtok(string,delim)  
    //token代表输入字符串中第一次出现在分隔符前面的所有字符
    //remainder代表这一行的其他部分    
    //string是输入字符串   
    //delim是可选择的分隔符   
    
        exp:    
        
        [token,remainder]=strtok('This is a test！')   
            
            token=    
            This     
            remainder=    
            is a test!    
    
    
    
    
- **大小写转换 **    
    - 函数upper和lower分别将一个字符串中所转换为大写和小写(数字和符号不受影响)         
    
    exp：   
        
        result =upper('This is test 1!')   
        result =   
        THIS IS TEST 1!    

        result =lower('This is test 2')   
        result=   
        this is test 2!      
        
        
        

- **字符串转换为数字**   
    - 函数eval将数字组成的字符串转换为数字     
    
        exp：    

        a='3.141592';   
        b=eval(a)   
    
        b=   
        3.1416    
    
    - 函数sscanf将字符串转换为数字，根据格式化转义字符转化为相应的数字
        
        **标准格式**   
        value=sscanf(string,format)    
        //string是要转换的字符串，format是相应的转换字符   
        
        **exp:**    
        value1=sscanf('3.141593','%g')    
        
        value1 =    

                3.1416     
 
 
 
                
- **数字转换成字符串**    
    - 函数int2num将一个标量数字转换为对应长度的字符数组；   
    
         exp：   
            x = 5317;      
            y = int2str(x)      
        
             y =    

                 5317    

        >> whos
          Name      Size            Bytes  Class     Attributes    

             x         1x1                 8  double                 
             y         1x4                 8  char          
             
             
    
    - 函数num2str可以对输出的字符串进行更多的控制    
    
        exp1：   
        p=num2str(pi,7)     
        
             p =    
                3.141593     
                
                
        exp2:      
        p=num2str(pi,'%10.5e')    
        
            p =    

                3.14159e+00     
                
                

        
        
        
        
        
## 多维数组     

- 多维数组创建      
    - 将二维数组拓展成为三维数组      
    
    exp:   
    a=[1 2 3 4;5 6 7 8]    
        //建立一个2x4的数组    
    //下面将其拓展为2x4x3数组    

    a(:,:,2)=[9 10 11 12;13 14 15 16]   
    a(:,:,3)=[17 18 19 20;21 22 23 24]    
    
        
        a(:,:,1) =   

     1     2     3     4   
     5     6     7     8   


        a(:,:,2) =   

     9    10    11    12   
    13    14    15    16   


        a(:,:,3) =   
          
    17    18    19    20   
    21    22    23    24     
    
    
    - 直接生成多维数组    
    exp：   
    
    b=ones(4,5,3)     
    
        b(:,:,1) =    

     1     1     1     1     1   
     1     1     1     1     1   
     1     1     1     1     1   
     1     1     1     1     1   


        b(:,:,2) =   

     1     1     1     1     1   
     1     1     1     1     1   
     1     1     1     1     1   
     1     1     1     1     1   


        b(:,:,3) =    

     1     1     1     1     1    
     1     1     1     1     1    
     1     1     1     1     1    
     1     1     1     1     1      
     
     
     
     
- 多维数组的维数和大小     

    多维数组的维数可以使用ndims函数得到，数组大小合一通过size函数得到    
    
    exp：   
    
        ndims(b)   
        
        ans =

             3
        
        size(b)   
        
        ans =   

        4     5     3    
        
        
        
- 多维数组的访问    

    b(2,2,2)   
    
        ans =

        1







## 二维作图补充    


### 二维作图的附加类型     

- 常见图形创建函数    


函数名|二维图像    
---|---    
stair  |  针头图   
bar  |条形图   
barh & compass  |  罗盘图   
pie |饼图    

**PS：**pie函数支持扇区分离 ,需要加上函数legend一起使用   


**exp:**   
    
    data = [10 37 5 6 6]; //饼状图比例是该元素占整个数组和的比例      
    explode = [0 1 0 0 0]; // 值为1所对应的数据会从整体的饼状图中分离出来           
    pie(data, explode);                     
    title('\bfExample of a Pie Plot');            
    legend('One','Two','Three','Four','Five');                



- 附加画图类型具体   

函数 | 描述    
---|---
bar(x, y) | 这个函数用于创建一个水平的条形图，x 代表第一个 X 轴的取值，y 代表对应于 Y 的取值 
barh(x, y) | 这个函数用于创建一个竖直的条形图，x 代表第一个 X 轴的取值，y 代表对应于 Y 的取值 
compass(x, y) | 这个函数用于创建一个极坐标图，它的每一个值都用箭头表示，从原点指向（x，y），注意：（x，y）是直角坐标系中的坐标。 
pie(x)            pie(x, explode) | 这个函数用来创建一个饼状图，x 代表占总数的百分数。explode 用来判断是否还有剩余的百分数 
stairs(x, y) | 用来创建一个阶梯图，每一个阶梯的中心为点(x, y) 
stem(x, y) | 这个函数可以创建一个针头图，它的取值为(x,y)




### 作图函数    

 这里不需要创建数组再将数组传递给作图函数，而是直接创建图像，函数分别为ezplot和fplot;    
 
 - ezplot函数调用形式：   
    
    - ezplot( fun);      

    - ezplot( fun, [xmin xmax]);      
    
    - ezplot( fun, [xmin xmax], figure);           
        //fun为要画的基本表达式         
        //xmin和xmax默认情况下为-2pi和2pi       
        
        
        
    **exp：**   
        ezplot('sin(x)/x',[-4*pi 4*pi]);     
        title('Plot of sinx/x');     
        grid on;      
        
        
        
        

- fplot函数：    

    功能上以及使用上与函数ezplot基本一致，但是表现的更加精确；     
    
    **优点：**     
    - 函数fplot是适应性的，即在函数突然变化时候相应的自变量会显示更多的点；   
    - 可以指定坐标图的标题与坐标轴的标签；    
    
    **一般情况下画函数图像时候，使用fplot函数。**     
    
    
    
    
    


### 柱状图    

    使用函数hist画柱状图       
    
    
    **函数的标准形式**      
    
    hist (y)            //创建并画出一个10等分的柱状图；       
    hist(y, nbins)      //创建以nbins为宽度的柱状图；   
    his(y, x);          //允许用户使用数组X指定柱状图中长条的中心
    [n, xout] =hist(y, ...)    //创建一个柱状图并返回一个数组xcout,在数组n中的每一长条的数目，但是实际上并没有创建一个图像。  
    











## 三维作图    

- 作用：   
    - 两个变量是同一自变量的函数，当你希望显示自变量重要性时候，可以使用三维作图；   
    - 一个变量是另外两个变量的函数；   



- 三维曲线作图：          
    - 标准最简单的样式：
        
        plot(x,y,z);    



    **exp:**    
                
                
        t = 0:0.1:10;           
        x = exp(-0.2*t) .* cos(2*t);     
        y = exp(-0.2*t) .* sin(2*t);     
        plot3(x,y,t);     
        title('\bfThree-Dimensional Line Plot');     
        xlabel('\bfx');    
        ylabel('\bfy');    
        zlabel('\bfTime');    
        axis square;    
        grid on;    


---



    - 三维表面，网格，等高线图像：   
    
    
函数  |  描述    
---|---
mesh(x, y, z) | 这个函数创建一个三维网格图象。数组 x 包括要画得每一点的 x 值，数组y 包括要画得每一点的 y 值，数组 z 包括要画得每一点的 z 值。 
surf(x, y, z) | 这个函数创建一个三维表面图象。x，y，z 代表的意义与上式相同。 
contour(x, y, z) | 这个函数创建一个三维等高线图象。x，y，z 代表的意义与上式相同。 
    
    
    **PS:**用户必须创建三个等大小的数组，这里可以使用meshgrid函数，函数形式为：   
    
    [x,y]=meshgrid(xstart:xinc:xend,ystart:yinc:yend);    
    
    
    **exp**    
    
        [x,y] = meshgrid(-4:0.2:4,-4:0.2:4);         
        z = exp(-0.5*(x.^2+y.^2));     
        mesh(x,y,z);     
        xlabel('\bfx');     
        ylabel('\bfy');     
        zlabel('\bfz');     
        
        
        
        
        
        
        
        
        
        
        
        
## 整章总结     


函数 | 描述     
      ---|---
char     | (1)把数字转化为相应的字符值 
char     | (2)把二维数组转化相应的字符串 
double   | 把字符转化为相应的 double 值 
blanks   | 创建一个由空格组成的字符串 
deblanks | 去除字符串末端的空格 
strcat   | 连接字符串 
strvcat  | 竖直地连接字符串 
strcmp   | 如果两字符串相等，那么函数将会返回 1 
stricmp  | 忽略大小写如果两字符串相等，那么函数将会返回 1 
strncmp  | 如果两字符串的前 n 个字母相等，那么函数将会返回 1 
strncmpi | 忽略大小，如果两字符串的前 n 个字母相同，那么数将会返回 1 
findstr  | 在一个字符串中寻找另一个字符串 
strfind  | 在一个字符串中寻找另一个字符串（版本 6。1 或以后的版本） 
strjust  | 对齐字符串 
strmatch | 找字符串的区配 
strrep   | 用一个字符串去替代另一个字符串 
strtok   | 查找一字符串 
upper    | 把字符串的所有字符转化为大写 
lower    | 把字符串的所有字符转化为小写 
int2str  | 把整数转化为相应的字符串形式 
num2str  | 把数字转化为相应的字符串形式 
mat2str  | 把矩阵转化为相应的字符串形式 
sprintf  | 对一字符串进行格式化输出 
str2double | 把字符串转化相应的 double 型数据 
str2num  | 把字符转化成数字 
sscanf   | 从字符串中读取格式化数据 
hex2num  | 把 IEEE 十六进制字符型型数据转化为 double 形数据 
hex2dec  | 把十六制字符串转化为相应的十进制整数 
dec2hex  | 把十进制数转化为相应的十六制字符串 
bin2dec  | 把二进制字符串转化为相应的十进制整数 
base2dec | 把 baseb 转化为相应的十进制数据 
dec2base |  把十进制转化为相应的 baseb 
bar(x, y) | 这个函数用于创建一个水平的条形图，x 代表第一个 X 轴的取值，y 代表对应于 Y 的取值 
barh(x, y)|  这个函数用于创建一个竖直的条形图，x 代表第一个 X 轴的取值，y 代表对应于 Y 的取值 
compass(x, y)  |这个函数用于创建一个极坐标图，它的每一个值都用箭头表示，从原点指向（x，y），注意：（x，y）是直角坐标系中的坐标。 
pie(x) pie(x, explode) |这个函数用来创建一个饼状图，x 代表占总数的百分数。explode 用来判断是否还有剩余的百分数 
stairs(x, y) | 用来创建一个阶梯图，每一个阶梯的中心为点(x, y) 
stem(x, y) | 这个函数可以创建一个针头图，它的取值为(x,y)





 






    
        
    

    
    