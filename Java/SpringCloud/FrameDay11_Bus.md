# Bus 消息总线

[TOC]



## 一、概述

Bus 支持两种消息代理：RabbitMQ 和 Kafka

### 是什么？
Spring Cloud Bus 是将分布式系统的节点与轻量级消息代理链接的框架，整合了 Java 的事件处理机制和消息中间件的功能。

**消息总线**

在微服务架构的系统中，通常使用**轻量级的消息代理**来构建一个**共用的消息主题**。并让系统中所有微服务实例都连接上了，由于**该主题中产生的消息会被所有实例监听和消费，因此称为消息总线**。

### 干什么
这可以用于广播状态更改（例如配置更改）或其他管理指令。一个关键的想法是，Bus就像一个扩展的Spring Boot应用程序的分布式执行器，但也可以用作应用程序之间的通信渠道。当前唯一的实现是使用AMQP代理作为传输，但是相同的基本功能集（还有一些取决于传输）在其他传输的路线图上。
### 官网资料
https://www.springcloud.cc/spring-cloud-bus.html

## 安装 RabbitMQ
1. 下载 ErLang
https://www.erlang.org/downloads
按照默认安装即可
2. 下载RabbitMQ：https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.7/rabbitmq-server-3.7.7.exe
按默认安装
3. 进入软件安装的 sbin目录输入命令：rabbitmq-plugins enable rabbitmq_management 进行安装
4. 通过安装的可视化插件 RabbitMQ-start 启动之后。查看是否安装成功：http://localhost:15672/
5. 默认登录账号密码： guest guest
```
rabbitmq-server -detached 后台启动

Rabbitmq-server 直接启动，如果你关闭窗口或者需要在改窗口使用其他命令时应用就会停止

 关闭:rabbitmqctl stop
```
## 使用 Bus 进行动态刷新全局广播
### 新建项目 cloud-config-client3366，与 3355 完全一样
### 设计思想
- 方式一：利用消息总线触发一个客户端/bus/refresh从而刷新所有客户端配置

- 方式二：利用消息总线触发一个服务端 ConfigServer 的/bus/refresh端点从而刷新所有客户端

3. 明显二更合适
    1. 打破了微服务职责单一性
    2. 破坏了微服务各节点的对等性
    3. 有一定局限性 ，微服务歉意时网络地址常常发生变化
### 实现
###### 给cloud-config-center3344 配置中心提供消息总线支持
- 步骤一：pom.xml 新增如下 MQ 依赖

    ```xml
    <!-- 添加消息总线RabbitMQ支持 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bus-amqp</artifactId>
    </dependency>
    ```

- 步骤二：yml 中也增加 MQ的支持

```yml
# 增加对 RabbitMQ 的配置
rabbitmq:
  host: localhost
  port: 5672
  username: guest
  password: guest

# RabbitMQ 相关配置，暴露 BUS 刷新配置的端点
management:
  endpoints:
    web:
      exposure:
        include: "bus-refresh"
```
###### 给cloud-config-client3355客户端提供消息总线支持
1. pom
```xml
<!-- 添加消息总线RabbitMQ支持 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

```
2. yml
```yml 
spring:
  application:
    name: cloud-client
  cloud:
    config:
      label: master
      name: config
      profile: dev
      uri: http://localhost:3344
  #rabbitmq配置,注意与服务端不同这个在spring下面
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```
###### 给cloud-config-client3366客户端提供消息总线支持
###### 测试

分别启动 7001/3344/3355/3366 

1. 改变 github 内容

2. 在发送 post 请求前面和后面分别访问

    `http://config-3344.com:3344/config-dev.yml` 和 `http://localhost:3355/configInfo` 和 `http://localhost:3366/configInfo`

3. 发送post请求：

    `curl -X POST "http://localhost:3344/actuator/bus-refresh"`

这样 3344/3355、3366 上获取的配置都变了

## 使用 Bus 进行动态刷新定点通知

用于指定具体某一个实例生效而不是全部。

1. 使用
    `curl -X POST "http://localhost:配置中心端口号/actuator/bus-refresh/{destination}"`
    
    其中 destination 就是`微服务名称+:端口号`
    
2. 本例中
    `curl -X POST "http://localhost:3344/actuator/bus-refresh/cloud-client:3355"`
    代表只通知3355