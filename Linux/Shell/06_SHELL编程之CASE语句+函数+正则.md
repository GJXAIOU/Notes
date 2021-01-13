[toc]
#课程目标

- 掌握case语句的基本语法结构
- 掌握函数的定义及调用
- 掌握常用的正则表达式元字符含义

# 一、case语句

**关键词：确认过眼神，你是对的人**:couple_with_heart:

1. case语句为多重匹配语句
2. 如果匹配成功，执行相匹配的命令

## 1. 语法结构

~~~powershell
说明：pattern表示需要匹配的模式


case var in             定义变量;var代表是变量名
pattern 1)              模式1;用 | 分割多个模式，相当于or
    command1            需要执行的语句
    ;;                  两个分号代表命令结束
pattern 2)
    command2
    ;;
pattern 3)
    command3
    ;;
		  *)              default，不满足以上模式，默认执行*)下面的语句
    command4
    ;;
esac							esac表示case语句结束
~~~

## 2. 应用案例

### ㈠ 脚本传不同值做不同事

**具体需求：**当给程序传入start、stop、restart三个不同参数时分别执行相应命令

~~~powershell
#!/bin/env bash
case $1 in
        start|S)
        service apache start &>/dev/null && echo "apache 启动成功"
        ;;
        stop|T)
        service apache stop &>/dev/null && echo "apache 停止成功"
        ;;
        restart|R)
        service apache restart &>/dev/null && echo "apache 重启完毕"
        ;;
        *)
        echo "请输入要做的事情..."
        ;;
esac

~~~

### ㈡ 根据用户需求选择做事

**具体需求：**

脚本提示让用户输入需要管理的服务名，然后提示用户需要对服务做什么操作，如启动，关闭等操作

```powershell
#!/bin/env bash
read -p "请输入你要管理的服务名称(vsftpd):" service
case $service in
        vsftpd|ftp)
        read -p "请选择你需要做的事情(restart|stop):" action
        case $action in
                stop|S)
                service vsftpd stop &>/dev/null && echo "该$serivce服务已经停止成功"
                ;;
                start)
                service vsftpd start &>/dev/null && echo "该$serivce服务已经成功启动"
                ;;
        esac
        ;;
        httpd|apache)
        echo "apache hello world"
        ;;
        *)
        echo "请输入你要管理的服务名称(vsftpd)"
        ;;
esac
	
```

###㈢ 菜单提示让用户选择需要做的事

**具体需求：**

模拟一个多任务维护界面;当执行程序时先显示总菜单，然后进行选择后做相应维护监控操作

```powershell
**********请选择*********
h	显示命令帮助
f	显示磁盘分区
d	显示磁盘挂载
m	查看内存使用
u	查看系统负载
q	退出程序
*************************
```

**思路：**

1. 菜单打印出来
2. 交互式让用户输入操作编号，然后做出相应处理

**落地实现：**

1. 菜单打印(分解动作)

```powershell
#!/bin/env bash
cat <<-EOF
	h	显示命令帮助
	f	显示磁盘分区
	d	显示磁盘挂载
	m	查看内存使用
	u	查看系统负载
	q	退出程序
	EOF
```

2. 最终实现

~~~powershell
#!/bin/bash
#打印菜单
cat <<-EOF
	h	显示命令帮助
	f	显示磁盘分区
	d	显示磁盘挂载
	m	查看内存使用
	u	查看系统负载
	q	退出程序
	EOF

#让用户输入需要的操作
while true
do
read -p "请输入需要操作的选项[f|d]:" var1
case $var1 in
	h)
	cat <<-EOF
        h       显示命令帮助
        f       显示磁盘分区
        d       显示磁盘挂载
        m       查看内存使用
        u       查看系统负载
        q       退出程序
	EOF
	;;
	f)
	fdisk -l
	;;
	d)
	df -h
	;;
	m)
	free -m
	;;
	u)
	uptime
	;;
	q)
	exit
	;;
esac
done



#!/bin/bash
#打印菜单
menu(){
cat <<-END
	h	显示命令帮助
	f	显示磁盘分区
	d	显示磁盘挂载
	m	查看内存使用
	u	查看系统负载
	q	退出程序
	END
}
menu
while true
do
read -p "请输入你的操作[h for help]:" var1
case $var1 in
	h)
	menu
	;;
	f)
	read -p "请输入你要查看的设备名字[/dev/sdb]:" var2
	case $var2 in
		/dev/sda)
		fdisk -l /dev/sda
		;;
		/dev/sdb)
		fdisk -l /dev/sdb
		;;
	esac
	;;
	d)
	lsblk
	;;
	m)
	free -m
	;;
	u)
	uptime
	;;
	q)
	exit
	;;
esac
done

~~~

**课堂练习：**

1. 输入一个等级（A-E），查看每个等级的成绩；如：输入A，则显示“90分~100分”，依次类推
2. 判断用户输入的字符串，如果是"hello",则显示"world"；如果是"world",则显示"hello",否则提示"请输入hello或者world，谢谢！"



# 二、==函数==

## 1. 什么是函数？

- shell中允许将**一组命令集合**或**语句**形成一段**可用代码**，这些代码块称为shell函数
- 给这段代码起个名字称为函数名，后续可以直接调用该段代码的功能

## 2. 如何定义函数？

**方法1：**

```powershell
函数名()
{
  函数体（一堆命令的集合，来实现某个功能）   
}
```

**方法2：**

```powershell
function 函数名()
{
   函数体（一堆命令的集合，来实现某个功能）
   echo hello
   echo world
}

```

**函数中==return==说明:**

1. return可以==结束一个函数==。类似于循环控制语句break(结束当前循环，执行循环体后面的代码)。
2. return默认返回函数中最后一个命令状态值，也可以给定参数值，范围是0-256之间。
3. 如果没有return命令，函数将返回最后一个指令的退出状态值。

##3. 函数如何调用？

### ㈠ 当前命令行调用

~~~powershell
[root@MissHou shell04]# cat fun1.sh 
#!/bin/bash
hello(){
echo "hello lilei $1"
hostname
}
menu(){
cat <<-EOF
1. mysql
2. web
3. app
4. exit
EOF
}

[root@MissHou shell04]# source fun1.sh 
[root@MissHou shell04]# . fun1.sh 

[root@MissHou shell04]# hello 888
hello lilei 888
MissHou.itcast.cc
[root@MissHou shell04]# menu
1. mysql
2. web
3. app
4. exit

~~~

### ㈡ 定义到用户的环境变量中

~~~powershell
[root@MissHou shell05]# vim ~/.bashrc 
文件中增加如下内容：
hello(){
echo "hello lilei $1"
hostname
}
menu(){
cat <<-EOF
1. mysql
2. web
3. app
4. exit
EOF
}

注意：
当用户打开bash的时候会读取该文件
~~~

### ㈢ 脚本中调用

~~~powershell
#!/bin/bash
#打印菜单
source ./fun1.sh
menu(){
cat <<-END
	h	显示命令帮助
	f	显示磁盘分区
	d	显示磁盘挂载
	m	查看内存使用
	u	查看系统负载
	q	退出程序
	END
}
menu		//调用函数

~~~

##4. 应用案例

**具体需求：**

1. 写一个脚本==收集用户输入==的基本信息(姓名，性别，年龄)，如不输入==一直提示输入==
2. 最后根据用户的信息输出相对应的内容

**思路：**

1. ==交互式==定义多个变量来保存用户信息  姓名、性别、年龄
2. 如果不输一直提示输入
   - ==循环==直到输入字符串不为空  while  判断输入字符串是否为空
   - 每个信息都必须不能为空，该功能可以定义为一个函数，方便下面脚本调用

3. 根据用户输入信息做出匹配判断

**代码实现：**

~~~powershell
#!/bin/bash
#该函数实现用户如果不输入内容则一直循环直到用户输入为止，并且将用户输入的内容打印出来
input_fun()
{
  input_var=""
  output_var=$1
  while [ -z $input_var ]
	do
	read -p "$output_var" input_var
	done
	echo $input_var
}

input_fun 请输入你的姓名:

或者
#!/bin/bash
fun()
{
	read -p "$1" var
	if [ -z $var ];then
		fun $1
	else
		echo $var
	fi
}


#调用函数并且获取用户的姓名、性别、年龄分别赋值给name、sex、age变量
name=$(input_fun 请输入你的姓名:)
sex=$(input_fun 请输入你的性别:)
age=$(input_fun 请输入你的年龄:)

#根据用户输入的性别进行匹配判断
case $sex in
			man)
			if [ $age -gt 18 -a $age -le 35 ];then
				echo "中年大叔你油腻了吗？加油"
			elif [ $age -gt 35 ];then
				echo "保温杯里泡枸杞"
			else
				echo "年轻有为。。。"
			fi
			;;
			woman)
			xxx
			;;
			*)
			xxx
			;;
esac

~~~

**扩展延伸：**

```powershell
描述以下代码含义：	
:()
{
   :|:&
}
:
```

#三、综合案例

## 1. 任务背景

现有的跳板机虽然实现了统一入口来访问生产服务器，yunwei用户权限太大可以操作跳板机上的所有目录文件，存在数据被误删的安全隐患，所以希望你做一些安全策略来保证跳板机的正常使用。

## 2. 具体要求

1. 只允许yunwei用户通过跳板机远程连接后台的应用服务器做一些维护操作
2. 公司运维人员远程通过yunwei用户连接跳板机时，跳出以下菜单供选择：

~~~powershell
欢迎使用Jumper-server，请选择你要操作的主机：
1. DB1-Master
2. DB2-Slave
3. Web1
4. Web2
h. help
q. exit
~~~

3. 当用户选择相应主机后，直接**免密码登录**成功
4. 如果用户不输入一直提示用户输入，直到用户选择退出

## 3. 综合分析

1. 将脚本放到yunwei用户家目录里的.bashrc文件里（/shell05/jumper-server.sh）
2. 将菜单定义为一个函数[打印菜单]，方便后面调用
3. 用case语句来实现用户的选择【交互式定义变量】
4. 当用户选择了某一台服务器后，进一步询问用户需要做的事情  case...esac  交互式定义变量
5. 使用循环来实现用户不选择一直让其选择
6. 限制用户退出后直接关闭终端  exit 

## 4. 落地实现

~~~powershell
#!/bin/bash
# jumper-server
# 定义菜单打印功能的函数
menu()
{
cat <<-EOF
欢迎使用Jumper-server，请选择你要操作的主机：
1. DB1-Master
2. DB2-Slave
3. Web1
4. Web2
h. help
q. exit
	EOF
}
# 屏蔽以下信号
trap '' 1 2 3 19
# 调用函数来打印菜单
menu
#循环等待用户选择
while true
do
# 菜单选择，case...esac语句
read -p "请选择你要访问的主机:" host
case $host in
	1)
	ssh root@10.1.1.1
	;;
	2)
	ssh root@10.1.1.2
	;;
	3)
	ssh root@10.1.1.3
	;;
	h)
	clear;menu
	;;
	q)
	exit
	;;
esac
done


将脚本放到yunwei用户家目录里的.bashrc里执行：
bash ~/jumper-server.sh
exit

~~~

**进一步完善需求**

为了进一步增强跳板机的安全性，工作人员通过跳板机访问生产环境，但是不能在跳板机上停留。

```powershell
#!/bin/bash
#公钥推送成功
trap '' 1 2 3 19
#打印菜单用户选择
menu(){
cat <<-EOF
欢迎使用Jumper-server，请选择你要操作的主机：
1. DB1-Master
2. DB2-Slave
3. Web1
4. Web2
h. help
q. exit
EOF
}

#调用函数来打印菜单
menu
while true
do
read -p "请输入你要选择的主机[h for help]：" host

#通过case语句来匹配用户所输入的主机
case $host in
	1|DB1)
	ssh root@10.1.1.1
	;;
	2|DB2)
	ssh root@10.1.1.2
	;;
	3|web1)
	ssh root@10.1.1.250
	;;
	h|help)
	clear;menu
	;;
	q|quit)
	exit
	;;
esac
done

自己完善功能：
1. 用户选择主机后，需要事先推送公钥；如何判断公钥是否已推
2. 比如选择web1时，再次提示需要做的操作，比如：
clean log
重启服务
kill某个进程
```

**回顾信号：**

~~~powershell
1) SIGHUP 			重新加载配置    
2) SIGINT			键盘中断^C
3) SIGQUIT      	键盘退出
9) SIGKILL		 	强制终止
15) SIGTERM	    	终止（正常结束），缺省信号
18) SIGCONT	   	继续
19) SIGSTOP	   	停止
20) SIGTSTP     	暂停^Z
~~~

# 四、正则表达式

##1. 正则表达式是什么？

**正则表达式**（Regular Expression、regex或regexp，缩写为RE），也译为正规表示法、常规表示法，是一种字符模式，用于在查找过程中==匹配指定的字符==。

许多程序设计语言都支持利用正则表达式进行**字符串操作**。例如，在Perl中就内建了一个功能强大的正则表达式引擎。

正则表达式这个概念最初是由Unix中的工具软件（例如sed和grep）普及开的。

支持正则表达式的程序如：locate |find| vim| grep| sed |awk

## 2. 正则能干什么？

1. 匹配邮箱、匹配身份证号码、手机号、银行卡号等
2. 匹配某些特定字符串，做特定处理等等

## 3. 正则当中名词解释

- **元字符**

  指那些在正则表达式中具有**特殊意义的==专用字符==**,如:点(.) 星(*) 问号(?)等

- **前导字符**

  位于**元字符**前面的字符.	ab**==c==***   aoo**==o==.**

##4. 第一类正则表达式

### ㈠ 正则中普通常用的元字符

| 元字符 | 功能                                         | 备注      |
| ------ | -------------------------------------------- | --------- |
| .      | 匹配除了换行符以外的==任意单个==字符         |           |
| *      | ==前导字符==出现==0==次或==连续多次==        |           |
| .*     | 任意长度字符                                 | ab.*      |
| ^      | 行首(以...开头)                              | ^root     |
| $      | 行尾(以...结尾)                              | bash$     |
| ^$     | 空行                                         |           |
| []     | 匹配括号里任意单个字符或一组单个字符         | [abc]     |
| [^]    | 匹配不包含括号里任一单个字符或一组单个字符   | [^abc]    |
| ^[]    | 匹配以括号里任意单个字符或一组单个字符开头   | ^[abc]    |
| \^[\^] | 匹配不以括号里任意单个字符或一组单个字符开头 | \^\[^abc] |

- 示例文本

~~~powershell
# cat 1.txt
ggle
gogle
google
gooogle
goooooogle
gooooooogle
taobao.com
taotaobaobao.com

jingdong.com
dingdingdongdong.com
10.1.1.1
Adfjd8789JHfdsdf/
a87fdjfkdLKJK
7kdjfd989KJK;
bSKJjkksdjf878.
cidufKJHJ6576,

hello world
helloworld yourself
~~~

- 举例说明

```powershell

```



### ㈡ 正则中其他常用元字符

| 元字符    | 功能                                    | 备注         |
| --------- | --------------------------------------- | ------------ |
| \\<       | 取单词的头                              |              |
| \\>       | 取单词的尾                              |              |
| \\<  \\>  | 精确匹配                                |              |
| \\{n\\}   | 匹配前导字符==连续出现n次==             |              |
| \\{n,\\}  | 匹配前导字符==至少出现n次==             |              |
| \\{n,m\\} | 匹配前导字符出现==n次与m次之间==        |              |
| \\(   \\) | 保存被匹配的字符                        |              |
| \d        | 匹配数字（**grep -P**）                 | [0-9]        |
| \w        | 匹配字母数字下划线（**grep -P**）       | [a-zA-Z0-9_] |
| \s        | 匹配空格、制表符、换页符（**grep -P**） | [\t\r\n]     |

**举例说明：**

~~~powershell
需求：将10.1.1.1替换成10.1.1.254

1）vim编辑器支持正则表达式
# vim 1.txt
:%s#\(10.1.1\).1#\1.254#g 
:%s/\(10.1.1\).1/\1.254/g 

2）sed支持正则表达式【后面学】
# sed -n 's#\(10.1.1\).1#\1.254#p' 1.txt
10.1.1.254

说明：
找出含有10.1.1的行，同时保留10.1.1并标记为标签1，之后可以使用\1来引用它。
最多可以定义9个标签，从左边开始编号，最左边的是第一个。


需求：将helloworld yourself 换成hellolilei myself

# vim 1.txt
:%s#\(hello\)world your\(self\)#\1lilei my\2#g

# sed -n 's/\(hello\)world your\(self\)/\1lilei my\2/p' 1.txt 
hellolilei myself

# sed -n 's/helloworld yourself/hellolilei myself/p' 1.txt 
hellolilei myself
# sed -n 's/\(hello\)world your\(self\)/\1lilei my\2/p' 1.txt 
hellolilei myself

Perl内置正则：
\d      匹配数字  [0-9]
\w      匹配字母数字下划线[a-zA-Z0-9_]
\s      匹配空格、制表符、换页符[\t\r\n]

# grep -P '\d' 1.txt
# grep -P '\w' 2.txt
# grep -P '\s' 3.txt
~~~

### ㈢ 扩展类正则常用元字符

**==丑话说在前面：==**

我说我比较特殊，你要相信！否则我错给你看:smirk:

- grep你要用我，必须加 **==-E==**  或者  让你兄弟`egrep`来找我

- sed你要用我，必须加 **==-r==**

| 扩展元字符 | 功能                   | 备注                                         |
| ---------- | ---------------------- | -------------------------------------------- |
| +          | 匹配一个或多个前导字符 | bo+ 匹配boo、 bo                             |
| ?          | 匹配零个或一个前导字符 | bo? 匹配b、 bo                               |
| \|         | 或                     | 匹配a或b                                     |
| ()         | 组字符（看成整体）     | (my\|your)self：表示匹配myself或匹配yourself |
| {n}        | 前导字符重复n次        |                                              |
| {n,}       | 前导字符重复至少n次    |                                              |
| {n,m}      | 前导字符重复n到m次     |                                              |

**举例说明：**

~~~powershell
# grep "root|ftp|adm" /etc/passwd
# egrep "root|ftp|adm" /etc/passwd
# grep -E "root|ftp|adm" /etc/passwd

# grep -E 'o+gle' test.txt 
# grep -E 'o?gle' test.txt 

# egrep 'go{2,}' 1.txt
# egrep '(my|your)self' 1.txt


使用正则过滤出文件中的IP地址：
# grep '[0-9]\{2\}\.[0-9]\{1\}\.[0-9]\{1\}\.[0-9]\{1\}' 1.txt 
10.1.1.1
# grep '[0-9]{2}\.[0-9]{1}\.[0-9]{1}\.[0-9]{1}' 1.txt 
# grep -E '[0-9]{2}\.[0-9]{1}\.[0-9]{1}\.[0-9]{1}' 1.txt 
10.1.1.1
# grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' 1.txt 
10.1.1.1
# grep -E '([0-9]{1,3}\.){3}[0-9]{1,3}' 1.txt 
10.1.1.1

~~~

##5. 第二类正则

| 表达式    | 功能                             | 示例            |
| --------- | -------------------------------- | --------------- |
| [:alnum:] | 字母与数字字符                   | [[:alnum:]]+    |
| [:alpha:] | 字母字符(包括大小写字母)         | [[:alpha:]]{4}  |
| [:blank:] | 空格与制表符                     | [[:blank:]]*    |
| [:digit:] | 数字                             | [[:digit:]]?    |
| [:lower:] | 小写字母                         | [[:lower:]]{4,} |
| [:upper:] | 大写字母                         | [[:upper:]]+    |
| [:punct:] | 标点符号                         | [[:punct:]]     |
| [:space:] | 包括换行符，回车等在内的所有空白 | [[:space:]]+    |

~~~powershell
[root@server shell05]# grep -E '^[[:digit:]]+' 1.txt
[root@server shell05]# grep -E '^[^[:digit:]]+' 1.txt
[root@server shell05]# grep -E '[[:lower:]]{4,}' 1.txt
~~~

## 6. 正则表达式总结

**把握一个原则，让你轻松搞定可恶的正则符号：**

1. 我要找什么？
   - 找数字  		                  [0-9]
   - 找字母                                    [a-zA-Z]
   - 找标点符号                            [[:punct:]]
2. 我要如何找？看心情找
   - 以什么为首                           ^key
   - 以什么结尾                           key$
   - 包含什么或不包含什么        [abc]  \^[abc]   [\^abc]   \^[\^abc]
3. 我要找多少呀？
   - 找前导字符出现0次或连续多次             ab==*==
   - 找任意单个(一次)字符                             ab==.==
   - 找任意字符                                               ab==.*==
   - 找前导字符连续出现几次                        {n}  {n,m}   {n,}
   - 找前导字符出现1次或多次                      go==+==
   -  找前到字符出现0次或1次                       go==?==                  

# 五、正则元字符一栏表

**元字符**：在正则中，具有特殊意义的专用字符，如: 星号(*)、加号(+)等

**前导字符**：元字符前面的字符叫前导字符

| 元字符           | 功能                                     | 示例              |
| ---------------- | ---------------------------------------- | ----------------- |
| *                | 前导字符出现0次或者连续多次              | ab*  abbbb        |
| .                | 除了换行符以外，任意单个字符             | ab.   ab8 abu     |
| .*               | 任意长度的字符                           | ab.*  adfdfdf     |
| []               | 括号里的任意单个字符或一组单个字符       | [abc]\[0-9]\[a-z] |
| [^]              | 不匹配括号里的任意单个字符或一组单个字符 | [^abc]            |
| ^[]              | 匹配以括号里的任意单个字符开头           | ^[abc]            |
| \^[^]            | 不匹配以括号里的任意单个字符开头         |                   |
| ^                | 行的开头                                 | ^root             |
| $                | 行的结尾                                 | bash$             |
| ^$               | 空行                                     |                   |
| \\{n\\}和{n}     | 前导字符连续出现n次                      | [0-9]\\{3\\}      |
| \\{n,\\}和{n,}   | 前导字符至少出现n次                      | [a-z]{4,}         |
| \\{n,m\\}和{n,m} | 前导字符连续出现n-m次                    | go{2,4}           |
| \\<\\>           | 精确匹配单词                             | \\<hello\\>       |
| \\(\\)           | 保留匹配到的字符                         | \\(hello\\)       |
| +                | 前导字符出现1次或者多次                  | [0-9]+            |
| ?                | 前导字符出现0次或者1次                   | go?               |
| \|               | 或                                       | \^root\|\^ftp     |
| ()               | 组字符                                   | (hello\|world)123 |
| \d               | perl内置正则                             | grep -P  \d+      |
| \w               | 匹配字母数字下划线                       |                   |

# 六、正则练习作业

## 1. 文件准备

```powershell
# vim test.txt 
Aieur45869Root0000
9h847RkjfkIIIhello
rootHllow88000dfjj
8ikuioerhfhupliooking
hello world
192.168.0.254
welcome to uplooking.
abcderfkdjfkdtest
rlllA899kdfkdfj
iiiA848890ldkfjdkfj
abc
12345678908374
123456@qq.com
123456@163.com
abcdefg@itcast.com23ed
```

## 2. 具体要求

```powershell
1、查找不以大写字母开头的行（三种写法）。
grep '^[^A-Z]' 2.txt
grep -v '^[A-Z]' 2.txt
grep '^[^[:upper:]]' 2.txt
2、查找有数字的行（两种写法）
grep '[0-9]' 2.txt
grep -P '\d' 2.txt
3、查找一个数字和一个字母连起来的
grep -E '[0-9][a-zA-Z]|[a-zA-Z][0-9]' 2.txt
4、查找不以r开头的行
grep -v '^r' 2.txt
grep '^[^r]' 2.txt
5、查找以数字开头的
grep '^[0-9]' 2.txt
6、查找以大写字母开头的
grep '^[A-Z]' 2.txt
7、查找以小写字母开头的
grep '^[a-z]' 2.txt
8、查找以点结束的
grep '\.$' 2.txt
9、去掉空行
grep -v '^$' 2.txt
10、查找完全匹配abc的行
grep '\<abc\>' 2.txt
11、查找A后有三个数字的行
grep -E 'A[0-9]{3}' 2.txt
grep  'A[0-9]\{3\}' 2.txt
12、统计root在/etc/passwd里出现了几次
grep -o 'root' 1.txt |wc -l

13、用正则表达式找出自己的IP地址、广播地址、子网掩码
ifconfig eth0|grep Bcast|grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}'
ifconfig eth0|grep Bcast| grep -E -o '([0-9]{1,3}.){3}[0-9]{1,3}'
ifconfig eth0|grep Bcast| grep -P -o '\d{1,3}.\d{1,3}.\d{1,3}.\d{1,3}'
ifconfig eth0|grep Bcast| grep -P -o '(\d{1,3}.){3}\d{1,3}'
ifconfig eth0|grep Bcast| grep -P -o '(\d+.){3}\d+'

# egrep --color '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' /etc/sysconfig/network-scripts/ifcfg-eth0
IPADDR=10.1.1.1
NETMASK=255.255.255.0
GATEWAY=10.1.1.254

# egrep --color '[[:digit:]]{1,3}\.[[:digit:]]{1,3}\.[[:digit:]]{1,3}\.[[:digit:]]{1,3}' /etc/sysconfig/network-scripts/ifcfg-eth0 
IPADDR=10.1.1.1
NETMASK=255.255.255.0
GATEWAY=10.1.1.254


14、找出文件中的ip地址并且打印替换成172.16.2.254
grep -o -E '([0-9]{1,3}\.){3}[0-9]{1,3}' 1.txt |sed -n 's/192.168.0.\(254\)/172.16.2.\1/p'

15、找出文件中的ip地址
grep -o -E '([0-9]{1,3}\.){3}[0-9]{1,3}' 1.txt

16、找出全部是数字的行
grep -E '^[0-9]+$' test
17、找出邮箱地址
grep -E '^[0-9]+@[a-z0-9]+\.[a-z]+$'


grep --help:
匹配模式选择：
Regexp selection and interpretation:
  -E, --extended-regexp     扩展正则
  -G, --basic-regexp        基本正则
  -P, --perl-regexp         调用perl的正则
  -e, --regexp=PATTERN      use PATTERN for matching
  -f, --file=FILE           obtain PATTERN from FILE
  -i, --ignore-case         忽略大小写
  -w, --word-regexp         匹配整个单词
  
```

#七、课后作业

## 脚本搭建web服务

**要求如下**：

1. 用户输入web服务器的IP、域名以及数据根目录
2. 如果用户不输入则一直提示输入，直到输入为止
3. 当访问www.test.cc时可以访问到数据根目录里的首页文件“this is test page” 

**参考脚本：**

~~~powershell
参考：
#!/bin/bash
conf=/etc/httpd/conf/httpd.conf
input_fun()
{
  input_var=""
  output_var=$1
  while [ -z $input_var ]
	do
	read -p "$output_var" input_var
	done
	echo $input_var
}
ipaddr=$(input_fun "Input Host ip[192.168.0.1]:")
web_host_name=$(input_fun "Input VirtualHostName [www.test.cc]:")
root_dir=$(input_fun "Input host Documentroot dir:[/var/www/html]:")

[ ! -d $root_dir ] && mkdir -p $root_dir
chown apache.apache $root_dir && chmod 755 $root_dir
echo this is $web_host_name > $root_dir/index.html
echo "$ipaddr $web_host_name" >> /etc/hosts

[ -f $conf ] && cat >> $conf <<end
NameVirtualHost $ipaddr:80
<VirtualHost $ipaddr:80>
	ServerAdmin webmaster@$web_host_name
	DocumentRoot $root_dir
	ServerName $web_host_name
	ErrorLog logs/$web_host_name-error_log
	CustomLog logs/$web_host_name-access_loh common
</VirtualHost>
end
~~~

