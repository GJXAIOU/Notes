# CentOS 7 安装  JDK 1.8

==所有过程都是自己亲测可以，如果安装过程中有任何疑问可以私聊或者邮箱联系即可==

## 一、打开url选择jdk1.8下载

[下载链接](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)

这里选择linux x64版本：

![jdk 版本]($resource/jdk%20%E7%89%88%E6%9C%AC.png)

## 二、下载

下载压缩包之后可以使用 `rz` 命令上传到虚拟机中；上传目录为：`/home/GJXAIOU/Java/`

## 三、安装

- 切换到刚才的上传目录，然后解压压缩包
```
tar -zxvf jdk-8u231-linux-x64.tar.gz 
```

## 4.设置环境变量

- 打开文件
**因为 profile 文件权限为：644，因此这里需要使用  root 用户才能编辑，或者使用 chmod 赋权限**
```
sudo vim /etc/profile
```

- 在配置文件的末尾添加
```
export JAVA_HOME=/home/GJXAIOU/Java/jdk1.8.0_231
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

- 使环境变量生效
```
source /etc/profile
```

- 检查
```
java -version
```
结果显示：
```java
[GJXAIOU@localhost o2o]$ java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```

==安装完成==