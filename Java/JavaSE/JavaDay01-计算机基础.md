---
tags : 
- java基础

flag: blue
---

@toc

主要目标 安装JDK环境的，运行第一个Java代码

## 一、 计算机基础

### （一）计算机是由硬件和软件组成的

- 硬件：
    CPU 内存 主板 显卡 电源  硬盘 散热器 显示器  键盘 鼠标 耳机 音响
    PC Person Computer

- 最主要的是什么
    
    CPU   内存 
CPU：中央处理器 处理计算机的数据  骁龙835 ARM 
     **RAM:随机存储器** 断电数据消失 内存  **Random Access Memory**
    **ROM：永久存储器**   HDD HHD  SSD  固态硬盘  **Read-Only Memory**
    GPU：显卡  比特币

- 硬件提供了软件的运行平台
    - 软件：
      - 操作系统：          
      - 应用程序：

- 软件是用来提供一个用户和计算机交流的方式
    - JavaEE + 云计算 
    - JavaWEB开发方向  Java后台
    - 50PB 50 * 1024TB
               

## (二)用户如何和计算机交流
  1. 图形化界面操作：
      QQ 网页 LOL 操作比较简单 人机交互比较好 友好性  效率低
  2. 命令行操作：
      DOS Linux基本指令 功能强大，效率很高，但是对于普通用户来说，友好性极差
      MS-DOS

  在Windows操作系统下如果打开黑框框：
      命令提示符 命令行     

​      a) 开始按钮 -> 所有程序 -> 附件 -> 命令提示符 
​      b) 开始按钮 -> 搜索程序和文件 ->输入cmd ,回车
​      c) Window + R -> 跳出一个对话框 “运行” -》 输入cmd， 回车
​        

### （三）  ==常用的DOS命令：==【掌握 默写】

- **dir** 查看当前工作路径下的文件和文件夹

- **cd** 切换工作目录：
  例如:
      cd Desktop 把当前的工作路径切换到桌面
      C:\Users\lxl   -> C:\Users\lxl\Desktop
      
  - a) `.` 一个点和  `..` 两个点  windows 和 Linux 通用的 
      . 表示当前的工作目录
      .. 表示当前工作目录的上一级目录/父目录
  
  - b) 相对路径：
就是两个目录或者两个文件之间的路径 
  
  - c) 绝对路径：
      计算机中绝对位置，唯一的位置
      
  
在Linux系统操作当中会遇到大量的路径问题
  
- **md**    make dir
  创建文件夹
  
- **rd**    remove dir 删除一个文件夹，**要求：删除文件夹里面不能有内容，删除的是一个空文件**
  【注意事项】
      rd 删除文件夹不是放入回收站，而是直接在磁盘上抹掉数据，**这个操作是不可逆的**
  
-  **echo**   获取文件名（包括后缀）
  echo 数据 > 文件名.后缀名
  在操作普通文件的时候，必须有文件后缀名，否则这个文件就是一个无效文件

- **del** 删除文件  delete 
  del 文件名.后缀名
  【注意事项】
      这个删除操作也是直接抹掉数据，**不可逆操作**
  
- **cls**    clear screan 清理当前屏幕显示

- **exit**    退出

- `*` 通配符
  任意长度任意类型

- **方向键的上下**，可以回顾之前的命令

- **copy   路径下文件   目标路径**     copy C:\1\mima.txt    C:\2\ 拷贝文件
  
    Linux 基本指令
        ls cd mkdir rm cp mv chmod sudo tar touch vim lsattr 
        
    
    <<鸟叔的私房菜>>
    <<Think in Java>> Java的圣经
    <<Java 核心技术 卷一卷二>>  
    <<Effective Java>>  Oracle 公司自己写的 
            
### （四）计算机的语言

1. 机器语言
    0101010101010101
    计算机之父 冯·诺依曼  二进制思想 文件存储原理
    000 111 001 010 100 101 110 011
    5000次

2. 汇编语言
        add sub mov 助记符来表示机器语言
        贝尔实验室 《太空漫步》 
        UNIX
    
3. 高级语言
    C C++ C# Java PHP Python Perl Go JavaScript
    
    Java 爪哇岛
    MySQL Tomcat Apacha
    
    编译性语言：
        C C++ OC PHP
    解释性语言:
        Java C#

### （五）Java的起源

高斯林 / 高司令
SUN Stanford University NetWork 斯坦福大学网络公司
Java是为了做机顶盒
96年1月  Java 1.0 主要推广技术方向是JavaWEB开发 网站开发 
04年推出跨时代的Java 1.5  JDK1.5  JDK1.8
09年甲骨文公司(Oracle)收购SUN公司

- Java三大语言结构

  JDK1.5之后才有的规范
  
  JavaSE 基本体系 Java个人开发的标准。
  JavaEE 企业级开发 主要针对的是 JavaWEB 方向
  JavaME 手机软件的开发 (MTK)

- Java最明显的一个特征：
        跨平台：一处编译，处处使用
        跨平台的原因：
            在不同的硬件设备上，都有 Java 的虚拟机： JVM 
        

### （六）JDK安装
以后所有的软件都从官网下载

- 获取JDK1.8 Windows 64位安装包
    [link](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
- 安装JDK1.8
        注意事项：
            a)安装到D盘
            b)要求安装的目录不能存在任何中文    jre1.8.0_131
            c)JDK安装完成会跳出第二个安装界面，更改安装目录安装地址和JDK在同一个目录下

**3. 专用名词**
        JRE: Java Runtime Environment Java运行环境
        JRE = Java虚拟机(JVM) + 核心类库(辅助JVM运行的工具) 
        JDK: Java Development Kits  Java 开发开发工具集
        JDK = JRE + Java开发工具 