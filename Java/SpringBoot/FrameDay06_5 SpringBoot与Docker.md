# 五、Spring Boot 与 Docker

[TOC]



## 一、Docker 简介

- Docker 是一个开源的**应用容器引擎**，是一个轻量级容器技术；

- Docker 支持**将软件编译成一个镜像**，然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；

- **运行中的这个镜像称为容器**，容器启动是非常快速的。

![](FrameDay06_5%20SpringBoot%E4%B8%8EDocker.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180303145531.png)

## 二、核心概念

- docker 主机(Host)：安装了 Docker 程序的机器（Docker 直接安装在操作系统之上）；

- docker 客户端(Client)：连接 Docker 主机进行操作；

- docker 仓库(Registry)：用来保存各种打包好的软件镜像；

- docker 镜像(Images)：软件打包好的镜像，放在 Docker 仓库中；

- docker 容器(Container)：**镜像启动后的实例称为一个容器**，容器是独立运行的一个或一组应用。

<img src="FrameDay06_5%20SpringBoot%E4%B8%8EDocker.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180303165113.png" style="zoom:80%;" />

使用 Docker 的步骤：

- 安装 Docker

- 去 Docker 仓库找到这个软件对应的镜像；

- 使用 Docker 运行这个镜像，这个镜像就会生成一个 Docker 容器；

- 对容器的启动停止就是对软件的启动停止；

## 三、安装 Docker

### （一）安装 linux 虚拟机

- VMWare、VirtualBox（安装）；
- 导入虚拟机文件 centos7-atguigu.ova；
- 双击启动 linux 虚拟机;使用  `root/ 123456` 登陆。
- 使用客户端连接 linux 服务器进行命令操作；

- 设置虚拟机网络；

​		桥接网络 -> ==选好网卡== -> 接入网线；

- 设置好网络以后使用命令重启虚拟机的网络 `service network restart`

- 查看 linux 的 ip 地址 `ip addr`

- 使用客户端连接 linux；

### （二）在 linux 虚拟机上安装 docker

- 步骤一：检查内核版本，必须是3.10及以上。命令： `uname -r`

- 步骤二：安装 docker `yum install docker`

- 步骤三：输入 `y` 确认安装。

- 步骤四：启动 docker

    ```shell
    [root@localhost ~]# systemctl start docker
    [root@localhost ~]# docker -v
    Docker version 1.12.6, build 3e8e77d/1.12.6
    ```

- 步骤五：开机启动 Docker

    ```shell
    [root@localhost ~]# systemctl enable docker
    ```

- 步骤六：停止 docker

    ```shell
    systemctl stop docker
    ```

## 四、Docker 常用命令与操作

### （一）针对镜像操作

[在线镜像库](https://hub.docker.com/)

| 操作 | 命令                                            | 说明                                                     |
| ---- | ----------------------------------------------- | -------------------------------------------------------- |
| 检索 | docker  search 关键字  eg：docker  search redis | 我们经常去docker  hub上检索镜像的详细信息，如镜像的TAG。 |
| 拉取 | docker pull  镜像名:tag                         | :tag是可选的，tag表示标签，多为软件的版本，默认是latest  |
| 列表 | docker images                                   | 查看所有本地镜像                                         |
| 删除 | docker  rmi   image-id                          | 删除指定的本地镜像                                       |

### （二）容器操作

软件镜像（QQ安装程序）----运行镜像----产生一个容器（正在运行的软件，运行的QQ）；

步骤：

- 搜索镜像

    ````shell
    [root@localhost ~]# docker search tomcat
    ````

- 拉取镜像

    ```shell
    [root@localhost ~]# docker pull tomcat
    ```

- 根据镜像启动容器(--name 自定义名字 -d:后台运行 运行的镜像名称)

    ```shell
    docker run --name mytomcat -d tomcat:latest
    ```

- 查看运行中的容器：`docker ps`

- 停止运行中的容器： `docker stop  容器的id`

- 查看所有的容器： `docker ps -a`

- 启动容器：`docker start 容器的id`

- 删除一个容器（rmi 是删除镜像的）：`docker rm 容器的id`

- 启动一个做了端口映射的tomcat：将虚拟机的 8888 映射到tomcat 的8080

    ```shell
    [root@localhost ~]# docker run -d -p 8888:8080 tomcat
    -d：后台运行
    -p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口
    ```

- 为了演示简单关闭了linux的防火墙
    `service firewalld status` ；查看防火墙状态
    `service firewalld stop`：临时关闭防火墙

- 查看容器的日志：`docker logs container-name/container-id`

更多命令参看每一个镜像的[文档](https://docs.docker.com/engine/reference/commandline/docker/)。

### （三）安装 MySQL 容器示例

步骤一：`docker pull mysql`

错误的启动方式

```shell
[root@localhost ~]# docker run --name mysql01 -d mysql
42f09819908bb72dd99ae19e792e0a5d03c48638421fa64cce5f8ba0f40f5846

# mysql退出了
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
42f09819908b        mysql               "docker-entrypoint.sh"   34 seconds ago      Exited (1) 33 seconds ago                            mysql01
538bde63e500        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       compassionate_
goldstine
c4f1ac60b3fc        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       lonely_fermi
81ec743a5271        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       sick_ramanujan


//错误日志
[root@localhost ~]# docker logs 42f09819908b
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD；# 这个三个参数必须指定一个
```

正确的启动

```shell
[root@localhost ~]# docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
b874c56bec49fb43024b3805ab51e9097da779f2f572c22c695305dedd684c5f
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b874c56bec49        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysql01
```

做了端口映射（应该使用这个）

```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql02 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```

对应启动 Redis 命令：`docker run --name redis -d -p 6379:6379 redis` 然后使用 `docker ps` 查看状态。

几个其他的高级操作

```shell
# 自定义 mysql 配置文件
docker run --name mysql03 -v /conf/mysql:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
把主机的/conf/mysql文件夹挂载到 mysqldocker容器的 /etc/mysql/conf.d文件夹里面
这样改mysql的配置文件就只需要把mysql配置文件放在自定义的文件夹下（/conf/mysql）

docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
不使用配置文件，直接指定mysql的一些配置参数
```

