# 简介
### 是什么？
Spring Cloud Bus将分布式系统的节点与轻量级消息代理链接
### 干什么
这可以用于广播状态更改（例如配置更改）或其他管理指令。一个关键的想法是，Bus就像一个扩展的Spring Boot应用程序的分布式执行器，但也可以用作应用程序之间的通信渠道。当前唯一的实现是使用AMQP代理作为传输，但是相同的基本功能集（还有一些取决于传输）在其他传输的路线图上。
### 官网资料
https://www.springcloud.cc/spring-cloud-bus.html

# 安装 RabbitMQ
1. 下载 ErLang
https://www.erlang.org/downloads
按照默认安装即可
2. 下载RabbitMQ：https://dl.bintray.com/rabbitmq/all/rabbitmq-server/3.7.7/rabbitmq-server-3.7.7.exe
按默认安装
3. 进入sbin目录输入命令：rabbitmq-plugins enable rabbitmq_management 进行安装
4. 查看是否安装成功：http://localhost:15672/
5. 登录 guest guest
```
rabbitmq-server -detached 后台启动

Rabbitmq-server 直接启动，如果你关闭窗口或者需要在改窗口使用其他命令时应用就会停止

 关闭:rabbitmqctl stop
```
# 全局广播
### 新建项目 cloud-config-client3366，与3355一样
### 设计思想
1. 利用消息总线触发一个客户端/bus/refresh从而刷新所有客户端配置
2. 利用消息总线触发一个服务端 ConfigServer 的/bus/refresh端点从而刷新所有客户端
3. 明显二更合适
    1. 打破了微服务职责单一性
    2. 破坏了微服务各节点的对等性
    3. 有一定局限性 ，微服务歉意时网络地址常常发生变化
### 实现
###### 给cloud-config-center3344 配置中心提供消息总线支持
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
#rabbitmq配置
rabbitmq:
  host: localhost
  port: 5672
  username: guest
  password: guest
#暴露bus刷新配置端点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
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
1. 改变github内容
2. 发送post请求：
    ```
    curl -X POST "http://localhost:3344/actuator/bus-refresh"
    ```
###### 动态刷新定点通知
1. 使用
    curl -X POST "http://localhost:配置中心端口号/actuator/bus-refresh/{destination}"
2. 本例中
    curl -X POST "http://localhost:3344/actuator/bus-refresh/cloud-client:3355"
    代表只通知3355