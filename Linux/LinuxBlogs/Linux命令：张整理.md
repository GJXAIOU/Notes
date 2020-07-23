# **第一部分 Linux的规则与安装**

## 第0章

所谓的位：指的是CPU一次读取数据的最大量

磁盘最小存储物理单位——扇区（sector）

## 第1章

## 第2章

## 第3章

## 第4章

### 基础命令

显示具体时间与日期：date

显示日历：cal+[month]+[year]

进入计算器：bc 退出计算器：quit

命令求助：[command]+ --help

关机：shutdown；poweroff；halt

重启：reboot

# **第二部分 Linux文件、目录、与磁盘格式**

## 第5章 Linux的文件权限与目录配置

修改文件权限：chmod, 9个基本权限，拥有者（owner）、所属群组（group）、其他人（others）

数字：chmod 777 文件名

读r:4； 写w:2 ； 执行x:1

符号：chmod u=rwx,g=rwx,o=rwx

切换目录：cd / （change directory）

cd /var/spool/main后执行cd ../cron进入/var/spool/cron/ （绝对路径与相对路径）

回到上层目录：cd ..

回到上一个操作的目录：cd -

回到使用者家目录（等同于cd）：cd ~

显示目前所在目录：pwd

新建空目录：mkdir (make directory)

删除空目录：rmdir

新建文件：touch

执行文件：./run.sh:执行本目录下名为run.sh的文件

查看Linux内核版本：uname -r

查看Linux操作系统的架构版本：uname -m

## 第6章 Linux文件与目录管理

### 查看当前文件夹的目录：ls

### 复制文件或目录：cp

cp -i ~/ .bashrc /tmp/bashrc:将家目录下的 .bashrc文件复制到/tmp目录下，并更名为 bashrc

-i的意思是目标文件夹下已存在该文件，询问是否覆盖

### 删除文件或目录：rm

rm -i .bashrc:删除.bashrc文件

-i的意思是确认操作

### 移动文件或目录：mv

mv bashrc1 bashrc2 mvtest:将bashrc1文件和bashrc2文件移动到mvtest目录下

### 直接查看文件内容：cat（concatenate）

### more

### less

### 数据截取：head

head -n 20 /etc/man_db.conf: 显示/etc/man_db.conf文件的前20行

tail -n 20 /etc/man_db.conf: 显示/etc/man_db.conf文件的后20行

### 查看非纯文本文件：od

### 观察文件类型：file

### 脚本文件的查找：which

which [-a] ifconfig :查找ifconfig这个命令的完整文件名

[-a] : 将所有由PATH目录中找到的命令均可以列出，而不止第一个被找到的命令名称

### 文件的查找：whereis；locate；find

whereis和locate直接加文件名，速度快

find / -mtime 0 :将过去系统上面24小时内有修改过的内容（mtime）的文件列出

find / etc -newer /etc/passwd :寻找/etc下面的文件，如果文件日期比 /etc/passwd新就列出

### 打包命令：tar

压缩： tar -jcv -f finename.tar.bz2 (finename.tar.bz2是文件名)

查询： tar -jtv -f finename.tar.bz2

解压缩： tar -jxv -f finename.tar.bz2 -C 解压目录

## 第9章 VIM程序编辑器

vi welcome.text 新建welxome.text文档并进入一般命令模式

按下 i 进入编辑模式

：wq表示保存并退出

：q表示退出

：q!表示强制退出但不保存

：qw!表示强制保存后退出

# **第三部分 学习shell与shell script**

name Vbird : 设置变量name，且内容为Vbird

unset name : 取消设置的name这个变量内容

env : 列出目前的shell环境中的所有环境变量与其内容 （environment）

set ： 列出所有变量，包括环境变量和自定义变量等

history n : 列出目前最近的n条数据

source ~ /. baashrc或者 . ~/.bashrc意思是读入环境配置文件的命令

## **set,env和export**

set 用来显示本地变量

env 用来显示环境变量

export 用来显示和设置环境变量

set 显示当前shell的变量，包括当前用户的变量

env 显示当前用户的变量

export 显示当前导出成用户变量的shell变量

### 正则表达式

dmesg | grep 'qxl'意思是用dmesg列出内核信息，再以grep找出内含qxl那行

grep -n 'the' regular_express.txt 从文件regular_express.txt中找出含有the字符串的行列并显示

grep -vn 'the' regular_express.txt 从文件regular_express.txt中找出不含the字符串的行列并显示

grep -in 'the' regular_express.txt 从文件regular_express.txt中找出含有the字符串的行列并显示，不区分大小写

### 文件比对

diff passwd.old passwd.new 比对文件passwd.old和passwd.new的差异

### 文件打印

pr /etc/man_db.conf 打印文件/etc/man_db.conf

## 账号管理

useradd vbirdl 新增用户vbirdl

su切换成root身份

sudo + command 以root身份执行