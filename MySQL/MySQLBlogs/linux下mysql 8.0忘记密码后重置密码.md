# linux下mysql 8.0忘记密码后重置密码



1：//免密码登陆

找到mysql配置文件:my.cnf，
`vi /etc/my.cnf`

在【mysqld】模块添加：skip-grant-tables   保存退出；

2：//使配置生效

重启mysql服务：  service mysqld restart；

3：//将旧密码置空

mysql -u root -p    //提示输入密码时直接敲回车。

//选择数据库

use mysql

//将密码置空

update user set authentication_string = '' where user = 'root';

//退出

quit

4://去除免密码登陆

删掉步骤1的语句  skip-grant-tables

重启服务  service mysqld restart

5：//修改密码

mysql -u root -p  //提示输入密码时直接敲回车，刚刚已经将密码置空了

ALTER USER 'root'@'localhost' IDENTIFIED BY 'abc123@xxx';//'abc123@xxx'  密码形式过于简单则会报错

ps：mysql5.7.6版本后 废弃user表中 password字段 和 password（）方法，所以旧方法重置密码对mysql8.0版本是行不通的，共勉