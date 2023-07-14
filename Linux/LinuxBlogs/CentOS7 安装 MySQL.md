# CentOS7 安装 MySQL

## 引言


Linux上安装软件常见的几种方式：

*   源码编译
*   压缩包解压（一般为tar.gz）
*   编译好的安装包（RPM、DPKG等）
*   在线安装（YUM、APT等）

以上几种方式便捷性依次增加，但通用性依次下降，比如直接下载压缩包进行解压，这种方式一般需要自己做一些额外的配置工作，但只要掌握了方法，各个平台基本都适用，YUM虽然简单，但是平台受限，网络受限，必要的时候还需要增加一些特定YUM源。

几种安装方式最好都能掌握，原则上能用简单的就用简单的：YUM>RPM>tar.gz>源码

本文是介绍MySQL在CentOS上的安装，主要步骤都是参考了MySQL官方文档：[dev.mysql.com/doc/refman/…](https://dev.mysql.com/doc/refman/5.7/en/installing.html)


## 一、YUM

#### 0、删除已安装的MySQL

##### 检查MariaDB

```
shell> rpm -qa|grep mariadb
mariadb-server-5.5.60-1.el7_5.x86_64
mariadb-5.5.60-1.el7_5.x86_64
mariadb-libs-5.5.60-1.el7_5.x86_64
```

##### 删除mariadb

如果不存在（上面检查结果返回空）则跳过步骤
```
shell> rpm -e --nodeps mariadb-server
shell> rpm -e --nodeps mariadb
shell> rpm -e --nodeps mariadb-libs
```
其实yum方式安装是可以不用删除mariadb的，安装MySQL会覆盖掉之前已存在的mariadb_

##### 检查MySQL
```
shell> rpm -qa|grep mysql
```

##### 删除MySQL

如果不存在（上面检查结果返回空）则跳过步骤
```
shell> rpm -e --nodeps xxx
```

### 1、添加MySQL Yum Repository

 从CentOS 7开始，MariaDB成为Yum源中默认的数据库安装包。也就是说在CentOS 7及以上的系统中使用yum安装MySQL默认安装的会是MariaDB（MySQL的一个分支）。如果想安装官方MySQL版本，需要使用MySQL提供的Yum源。

##### 下载MySQL源

官网地址：[dev.mysql.com/downloads/r…](https://dev.mysql.com/downloads/repo/yum/)

查看系统版本：

```
shell> cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
```

选择对应的版本进行下载，例如CentOS 7当前在官网查看最新Yum源的下载地址为： [dev.mysql.com/get/mysql80…](https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm)

```
shell> wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

##### 安装MySQL源
例如CentOS7当前最新MySQL源安装：
```
shell> sudo rpm -Uvh mysql80-community-release-el7-3.noarch.rpm

```

##### 检查是否安装成功

执行成功后会在`/etc/yum.repos.d/`目录下生成两个repo文件`mysql-community.repo`及 `mysql-community-source.repo`

并且通过`yum repolist`可以看到mysql相关资源

```
shell> yum repolist enabled | grep "mysql.*-community.*"
!mysql-connectors-community/x86_64 MySQL Connectors Community                108
!mysql-tools-community/x86_64      MySQL Tools Community                      90
!mysql80-community/x86_64          MySQL 8.0 Community Server                113
```

### 2、选择MySQL版本

使用MySQL Yum Repository安装MySQL，默认会选择当前最新的稳定版本，例如通过上面的MySQL源进行安装的话，默安装会选择MySQL 8.0版本，**如果就是想要安装该版本，可以直接跳过此步骤**，如果不是，比如我这里希望安装MySQL5.7版本，就需要“切换一下版本”：

##### 查看当前MySQL Yum Repository中所有MySQL版本（每个版本在不同的子仓库中）

```
shell> yum repolist all | grep mysql
```

##### 切换版本
```
shell> sudo yum-config-manager --disable mysql80-community
shell> sudo yum-config-manager --enable mysql57-community
```

除了使用yum-config-manager之外，还可以直接编辑`/etc/yum.repos.d/mysql-community.repo`文件

enabled=0禁用
```
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

enabled=1启用
```
# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

##### 检查当前启用的MySQL仓库
```
shell> yum repolist enabled | grep mysql
```

如果同时启用了多个仓库，安装时会选择最新版本_

### 3、安装MySQL
```
shell> sudo yum install mysql-community-server
```

该命令会安装MySQL服务器 (mysql-community-server) 及其所需的依赖、相关组件，包括mysql-community-client、mysql-community-common、mysql-community-libs等

如果带宽不够，这个步骤时间会比较长，请耐心等待~

### 4、启动MySQL

CentOS 6：
```
shell> sudo service mysqld start
```

##### 查看状态

CentOS 6：
```
shell> sudo service mysqld status

```

##### 停止

CentOS 6：

```
shell> sudo service mysqld stop
复制代码
```

##### 重启

```
shell> sudo systemctl restart mysqld.service
复制代码
```

CentOS 6：

```
shell> sudo service mysqld restart
复制代码
```

### 5、修改密码

##### 初始密码

MySQL第一次启动后会创建超级管理员账号`root@localhost`，初始密码存储在日志文件中：

```
shell> sudo grep 'temporary password' /var/log/mysqld.log
```

##### 修改默认密码

```
shell> mysql -uroot -p
```

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```


出现上面的提示是因为密码太简单了，解决方法如下：
**修改 my.cnf 前要设置权限，设置时候将权限返回回去**
1.  使用复杂密码，MySQL默认的密码策略是要包含数字、字母及特殊字符；
2.  如果只是测试用，不想用那么复杂的密码，可以修改默认策略，即`validate_password_policy`（以及`validate_password_length`等相关参数），使其支持简单密码的设定，具体方法可以自行百度；
**具体方法**：
想设置符合的密码，然后设置上面两个策略

```
mysql>ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root_12root';
mysql>set global validate_password.policy=0;
mysql>set global validate_password.length=1;
mysql>ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```


### 6、允许root远程访问

```
mysql> create user 'GJXAIOU'@'%'identified by 'gjx';
mysql> grant all privileges on *.* to 'GJXAIOU'@'%' with grant option;
mysql> FLUSH PRIVILEGES;

```




### 7、设置编码为utf8

##### 查看编码

```
mysql> SHOW VARIABLES LIKE 'character%';
```

##### 设置编码

编辑/etc/my.cnf，[mysqld]节点增加以下代码：

```
[mysqld]
character_set_server=utf8
init-connect='SET NAMES utf8'
```

### 8、设置开机启动

```
shell> systemctl enable mysqld
shell> systemctl daemon-reload
复制代码
```

## 二、RPM

> 除安装过程外，其他步骤和yum方式安装相同，不再赘述

### 0、删除已旧版本

略

### 1、下载MySQL安装包

下载地址：[dev.mysql.com/downloads/m…](https://dev.mysql.com/downloads/mysql/)

选择对应的版本：

![](https://user-gold-cdn.xitu.io/2019/6/18/16b66894c80e9b32?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```
shell> wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
复制代码
```

### 2、安装MySQL

##### 解压（解打包）

```
shell> tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar
mysql-community-embedded-devel-5.7.26-1.el7.x86_64.rpm
mysql-community-libs-5.7.26-1.el7.x86_64.rpm
mysql-community-embedded-5.7.26-1.el7.x86_64.rpm
mysql-community-test-5.7.26-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.26-1.el7.x86_64.rpm
mysql-community-common-5.7.26-1.el7.x86_64.rpm
mysql-community-devel-5.7.26-1.el7.x86_64.rpm
mysql-community-client-5.7.26-1.el7.x86_64.rpm
mysql-community-server-5.7.26-1.el7.x86_64.rpm
复制代码
```

我们主要安装的是这四个（如果有需要也可以一并安装其它的）：

```
mysql-community-libs-5.7.26-1.el7.x86_64.rpm
mysql-community-common-5.7.26-1.el7.x86_64.rpm
mysql-community-client-5.7.26-1.el7.x86_64.rpm
mysql-community-server-5.7.26-1.el7.x86_64.rpm
复制代码
```

如果不想下载rpm-bundle，官网也提供单独的rpm下载链接

##### 安装

各rpm包是有依赖关系的，所以需要按照一定顺序进行安装，安装期间如果提示缺少哪些依赖也要先安装相应的包：

```
shell> rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm
shell> rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm
shell> rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm
shell> rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm
复制代码
```

还有一种简单的方式，可以自动处理各个包之间的依赖关系并自动下载缺少的依赖：

```
shell> yum install mysql-community-{server,client,common,libs}-*
复制代码
```

_注意：上面的`yum install`命令需要在tar解压之后的各个rpm包所在目录内执行，否则就变成yum方式安装了，需要配置MySQL的yum源并且速度很慢，还要当前机器支持外网访问_

### 3、设置

略

## 三、tar.gz

### 0、删除旧版本

略

### 1、下载

下载地址：[dev.mysql.com/downloads/m…](https://dev.mysql.com/downloads/mysql/)

选择对应的版本：

![](https://user-gold-cdn.xitu.io/2019/6/18/16b668a1b3c258aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```
shell> wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz
复制代码
```

### 2、安装&配置：

##### 依赖

MySQL依赖libaio库，如果没有先安装一下：

```
shell> yum install libaio
复制代码
```

##### 创建mysql用户

不需要登录的一个系统账号，启动MySQL服务时会使用该账号

```
shell> groupadd mysql
shell> useradd -r -g mysql -s /bin/false mysql
复制代码
```

##### 解压并创建链接

```
shell> cd /usr/local
shell> tar zxvf /path/to/mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz
shell> ln -s mysql-5.7.26-linux-glibc2.12-x86_64/ mysql
复制代码
```

##### 创建mysql-files目录

这一步并不是必须的，可以设置secure_file_priv的值指向该目录（用于限制数据导入导出操作的目录）

```
shell> cd mysql
shell> mkdir mysql-files
shell> chown mysql:mysql mysql-files
shell> chmod 750 mysql-files
复制代码
```

##### 初始化

```
shell> bin/mysqld --initialize --user=mysql
复制代码
```

如果初始化时报错如下：

```
error while loading shared libraries: libnuma.so.1: cannot open shared object file: No such file or directory
复制代码
```

是因为libnuma没有安装（或者默认安装的是32位），我们这里需要64位的：

```
shell> yum install numactl.x86_64
复制代码
```

执行完后重新初始化即可 初始化成功后返回结果中有一行包含初始密码，第一次登录时要用到它：

```
A temporary password is generated for root@localhost: 8M0ary878s*U
复制代码
```

##### 启用SSL（非必须）

```
shell> bin/mysql_ssl_rsa_setup
复制代码
```

##### 启动

```
shell> bin/mysqld_safe --user=mysql &
复制代码
```

查看进程可以看到一些默认参数，可以在配置文件中修改这些参数

```
shell> ps -ef | grep mysql
root     14604 12719  0 00:03 pts/0    00:00:00 /bin/sh bin/mysqld_safe --user=mysql
mysql    14674 14604  0 00:03 pts/0    00:00:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=VM_2_24_centos.err --pid-file=VM_2_24_centos.pid
复制代码
```

##### 设置环境变量

避免每次执行mysql命令都要加上路径，在`/etc/profile`中添加：

```
export PATH=$PATH:/usr/local/mysql/bin
复制代码
```

##### 设置为服务

```
shell> cp support-files/mysql.server /etc/init.d/mysqld
shell> service mysqld start|stop|restart|status
复制代码
```

##### 开机启动

```
shell> chkconfig --add mysqld
shell> chkconfig --list mysqld
mysqld         	0:关	1:关	2:开	3:开	4:开	5:开	6:关
复制代码
```

_其他配置与yum、rpm相同，不再赘述_

## 四、源码安装

就别费这个劲了吧...

## 结束语

我们不是Linux运维专家，也不是MySQL专家，生在这个年代也不知算是幸福还是不幸，线上的环境已经越来越少有人（主要指平时写代码的人）手动去搞这些数据库、中间件的安装配置了，为什么呢？因为各种云产品实在是太方便了呀，一般的公司也不会差这几个钱，既方便又稳定，何乐而不为呢~但是我们自己搞一搞用于自己测试还是必要的，而且还有不少公司的开发环境、测试环境偶尔还是需要手动搞一下的，当然，还有那些个自己搞机房的巨头们。

那我们既然不是专家，上面所写的内容如果有纰漏也是在所难免的，如果被看到了还希望能够及时批评指正~