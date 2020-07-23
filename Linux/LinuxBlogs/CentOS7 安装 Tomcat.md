# CentOS安装Tomcat


1.安装tomcat前，需要安装JDK，请参考博主另外一篇文章[CentOS使用yum安装jdk](https://segmentfault.com/a/1190000015389941)

2.本次安装tomcat，使用wget下载命令安装，需要先安装wget命令

```
yum -y install wget
```

3.下载tomcat

```
cd /home/GJXAIOU/Tomcat/
```

```
wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.47/bin/apache-tomcat-8.5.47.tar.gz
```

**前面三步可以自己到 Tomcat 官网下载安装包 ，然后上传到 服务器，最后使用下面的解压。**


4.解压tomcat

```
tar -zxvf apache-tomcat-8.5.47.tar.gz
```

```
mv apache-tomcat-8.5.31 tomcat(修改名称)
```

5.tomcat常用命令

```
/usr/local/tomcat/bin/startup.sh(启动命令)
/usr/local/tomcat/bin/shutdown.sh(关闭命令)
ps -ef|grep java(查看tomcat进程)
kill -9 进程号(杀死经常)
tail -f /usr/local/tomcat/logs/catalina.out(查看tomcat日志)
```