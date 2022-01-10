# 用 Navicat for MySQL 把本地数据库上传到服务器

服务器系统基本都是基于 linux 的，这个数据库上传的方式适用于 linux 的各种版本，当然本地数据库上传到服务器的前提是，服务器也已经安装好了MySQL数据库

## 一、在服务器端：

给远程访问设置权限

```sql
#其中123456是用于连接的密码，根据自己定义
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

FLUSH PRIVILEGES; #设置密码，如果是新安装的mysql需要在这里把密码设置了，如果已经有密码了就不用了
set password =password('123456');
flush privileges;
```

## 二、在本地：

### 建立连接

先新建连接，跟本地数据库连上，连接名随便起一个就可以，如图

![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508232234684-1286027701.png)



![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508232526306-1092687796-16417174208654.png)



![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508232725823-684455099.png)

再新建连接，跟服务器数据库连上，连接名也随便起一个就可以，如图

 ![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508233336230-1031165498.png)



![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508233404808-663892391.png)

数据传输，如图

![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508233604890-1805995381.png)

![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508234118218-1284811614.png)

图3

![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508234339746-1370258325.png)

至此，完成了本地数据库传输到服务器的过程

可以到服务器端，进入mysql查看一下，是否已经上传成功：

![](使用Navicat for MySQL把本地数据库上传到服务器.resource/1282071-20180508234559565-984976539.png)

