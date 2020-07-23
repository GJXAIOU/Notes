# CentOS7 服务器导入执行 SQL 文件


==所有过程都是自己亲测可以，如果安装过程中有任何疑问可以私聊或者邮箱联系即可==

## 一、首先使用 Navicat 导出 SQL 文件
![Navicat 导出 SQL 文件]($resource/Navicat%20%E5%AF%BC%E5%87%BA%20SQL%20%E6%96%87%E4%BB%B6.png)

## 二、将文件上传到服务器

- 上传位置：`/home/GJXAIOU/Project/o2o/`
- 使用命令：`rz`

## 三、将数据导入
- 首先进入数据库：`mysql -uroot -p`
- 正式导入数据
```sql
# 创建数据库
show databases;
create database 数据库名; # 该数据库名和导出文件的数据库名相同
use 数据库名;

# 执行数据库文件
source /home/GJXAIOU/Project/o2o/o2o.sql

# 查看导入是否成功
show tables;
```

## 四、补充

- 如果已经有同名的数据库，可以使用 ：`drop database 数据库名;` 删除原来的数据库；

**如果可以欢迎点赞呀**