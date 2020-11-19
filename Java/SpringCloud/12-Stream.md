# 简介
### 官网
https://spring.io/projects/spring-cloud-stream
### 是什么
Spring Cloud Stream是一个框架，用于构建与共享消息传递系统连接的高度可扩展的事件驱动型微服务。

应用程序通过 inputs 和 outputs 来与 Spring Cloud Stream 中的 binder 对象交互。而 Spring Cloud Stream 的 binder 对象负责与消息中间件交互。

==目前仅支持 RabbitMQ,Kafaka==
### 解决了什么
一个系统中采用多个消息中间件，解决不同消息中间件之间通信的问题。
### 消息中间件
###### 种类
1. ActiveMQ
2. RabbitMQ
3. RocketMQ
4. Kafka
###### 标准MQ
1. 生产者和消费者之间靠消息媒介传递消息内容？Message
2. 消息必须走特定通道？MessageChannel
3. 消息通道里的消息如何被消费？
消息通道MessageChannel的子接口SubscribableChanner，由MessageHandler消息处理器所订阅
###### 常用注解
1. @Input
注解标识输入通道
2. @Output
注解标识输出通道
3. @StreamListener
监听队列，用于消费者的队列的消息接收
4. @EnableBinding
channel和exchange绑定在一起
# 实操
### 模块 cloud-stream-rabbitmq-provider8801
1. pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
</dependency>
```
2. yml
```yml
server:
  port: 8801

spring:
  application:
    name: cloud-stream-provider
  cloud:
    stream:
      binders: #在此配置要绑定的 rabbitmq的服务信息
        defaultRabbit:  # 表示定义的名称，用于 binding整合
          type: rabbit  # 消息组件类型
          environment:  # 设置rabbitmq相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        output: # 输出通道的名称
          destination: studyExchange  #表示要使用的 Exchange 名称定义
          content-type: application/json  # 消息类型
          binder: defaultRabbit
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳时间间隔默认30s
    lease-expiration-duration-in-seconds: 5 # 如果超过了5秒的间隔默认90s
    instance-id: send-8001.com  #信息列表显示主机名称
    prefer-ip-address: true # 访问路径变为ip地址
```
3. main
```java
@SpringBootApplication
@EnableEurekaClient
```
4. service
```java
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.support.MessageBuilder;

import javax.annotation.Resource;
import java.util.UUID;

@EnableBinding(Source.class)//定义消息推送管道
@Slf4j
public class IMessageProviderImpl implements IMessageProvider {

    @Resource
    private MessageChannel output;//消息发送通道

    @Override
    public String send() {

        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        log.info(serial+"***********************");
        return serial;
    }
}
```
5. controller
```java
@RestController
public class IMessageController {
    @Resource
    private IMessageProvider provider;

    @GetMapping("/sendMessage")
    public String send(){
        return provider.send();
    }
}
```
6. 测试
    1. 进入rabbitmq 查看Exchanges中有没有studyExchange对应 yml中的自定义名字
    2. 多次访问http://localhost:8801/sendMessage
    3. 查看rabbitmq 中overview中 Message rates 的折线变化
### 消费者模块 cloud-stream-rabbitmq-consumer8802
1. pom
同8801一样
2. yml
```yml
server:
  port: 8802

spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: #在此配置要绑定的 rabbitmq的服务信息
        defaultRabbit:  # 表示定义的名称，用于 binding整合
          type: rabbit  # 消息组件类型
          environment:  # 设置rabbitmq相关的环境配置
            spring:
              rabbitmq:
                host: localhost
                port: 5672
                username: guest
                password: guest
      bindings: # 服务的整合处理
        input: # 输出通道的名称
          destination: studyExchange  #表示要使用的 Exchange 名称定义
          content-type: application/json  # 消息类型
          binder: defaultRabbit
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
  instance:
    lease-renewal-interval-in-seconds: 2 # 设置心跳时间间隔默认30s
    lease-expiration-duration-in-seconds: 5 # 如果超过了5秒的间隔默认90s
    instance-id: receive-8002.com  #信息列表显示主机名称
    prefer-ip-address: true # 访问路径变为ip地址
```
3. main
@SpringBootApplication
4. 业务类
```java
@Component
@Slf4j
@EnableBinding(Sink.class)
public class StreamController {
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String>message){
        log.info("消费者1号接收到消息"+message.getPayload()+"\t port:"+serverPort);
    }

}
```
5. 测试
    1. 依次启动7001，8801,8802
    2. 访问http://localhost:8801/sendMessage
    3. 查看8801控制台是否有输出
### 消息重复消费
1. 新建8803，同8802一样
2. 访问http://localhost:8801/sendMessage
3. 8802与8803都可以访问消息

# 消息分组与持久化
### 消息分组
1. 将两个微服务分到一个组group中，保证消息只能被一个组中的一个应用消费一次，不同的组可以同时消费
2. 配置8802与8803的yml如下
```yml
spring:
  application:
    name: cloud-stream-consumer
  cloud:
    stream:
      binders: #在此配置要绑定的 rabbitmq的服务信息
      bindings: # 服务的整合处理
        input: # 输出通道的名称
          group: group1 # 将 8802与8803分为同一组，这样消息只有一个可以消费
```
### 持久化
1. 停掉8802,8803
2. 使用8801发送消息
3. 删除8803的分组 group 
4. 启动8803与8802
5. 发现8802可以接受到关闭时发送的消息，而8803不能
