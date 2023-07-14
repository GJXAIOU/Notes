# Spring Boot与消息

## 一、概述

- 大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力
- 消息服务中两个重要概念：
    消息代理（message broker）和目的地（destination）
    当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。
- 消息队列主要有两种形式的目的地
    - 队列（queue）：点对点消息通信（point-to-point）
    - 主题（topic）：发布（publish）/订阅（subscribe）消息通信

### 消息队列的作用

作用一：异步处理

![image-20201114091420666](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114091420666.png)

作用二：应用解耦

![image-20201114091509171](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114091509171.png)

作用三：流量削峰

![image-20201114091528627](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114091528627.png)

### 消息队列分类：

- 点对点式：
    - 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，**消息读取后被移出队列**。
    - **消息只有唯一的发送者和接受者，但并不是说只能有一个接收者**。（即可以有多个接收，但是最终只有一个可以收到，收到该消息就从队列中移除了）
- 发布订阅式：
    发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息。

### 消息队列规范

- JMS（Java Message Service）JAVA消息服务：
    基于JVM消息代理的规范。ActiveMQ、HornetMQ是JMS实现
- AMQP（Advanced Message Queuing Protocol）
    - 高级消息队列协议，也是一个消息代理的规范，兼容JMS
    - RabbitMQ是AMQP的实现

| 区别         | JMS          | AMQP         |
| ------------ | --------------- | --------- |
| 定义         | Java api     | 网络线级协议      |
| 跨语言       | 否      | 是         |
| 跨平台       | 否       | 是         |
| Model    | 提供两种消息模型：（1）、Peer-2-Peer（2）、Pub/sub    | 提供了五种消息模型：direct exchange/fanout exchange/topic change/headers exchange /system exchange 本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 多种消息类型：TextMessage/MapMessage/BytesMessage/ StreamMessage/ ObjectMessage/ Message （只有消息头和属性） | byte[] 当实际应用时，有复杂的消息，可以将消息序列化后发送。 |
| 综合评价   | JMS 定义了JAVA API层面的标准；在java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差； | AMQP定义了wire-level层的协议标准；天然具有跨平台、跨语言特性。 |



- Spring 提供的支持
    - spring-jms提供了对JMS的支持
    - spring-rabbit提供了对AMQP的支持
    - 需要ConnectionFactory的实现来连接消息代理
    - 提供JmsTemplate、RabbitTemplate来发送消息
    - @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
    - @EnableJms、@EnableRabbit开启支持
- Spring Boot自动配置
    - JmsAutoConfiguration
    - RabbitAutoConfiguration

## 二、RabbitMQ简介

RabbitMQ 是一个由 erlang 开发的 AMQP(Advanved Message Queue Protocol)的开源实现。核心概念如下：

- Message
    消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。
- Publisher
    消息的生产者，也是一个向交换器发布消息的客户端应用程序。
- Exchange
    交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
    Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别
- Queue
    消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。
- Binding
    绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。
    Exchange 和Queue的绑定可以是多对多的关系。
- Connection
    网络连接，比如一个 TCP 连接。
- Channel
    信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP 连接。
- Consumer
    消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。
- Virtual Host
    虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个vhost 本质上就是一个mini 版的RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的vhost 是 `/` 。 一个 Broker 包括多个 Virtual Host，各个虚拟主机之间完全独立。
- Broker
    表示**消息队列服务器实体**

![image-20201114093641479](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114093641479.png)

## 三、RabbitMQ 运行机制

### AMQP 中的消息路由

- AMQP 中消息的路由过程和 Java 开发者熟悉的JMS 存在一些差别，AMQP 中增加了 Exchange 和 Binding 的角色。**生产者把消息发布到 Exchange 上**，消息最终到达队列并被消费者接收，而 **Binding  决定交换器的消息应该发送到那个队列**。

<img src="FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114094547027.png" alt="image-20201114094547027" style="zoom:80%;" />





### Exchange 类型

Exchange 分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers 。headers 匹配 AMQP 消息的 header 而不是路由键，headers 交换器和direct 交换器完全一致，但性能差很多，目前几乎用不到了，所以直接看另外三种类型：

- Direct Exchange

    **消息中的路由键（routing key）如果和 Binding 中的 binding key 一致，交换器就将消息发到对应的队列中**。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为 「dog」，则只转发 routing key 标记为 「dog」的消息，不会转发 「dog.puppy」，也不会转发 「dog.guard」等等。它是完全匹配、单播的模式。

<img src="FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114094704194.png" alt="image-20201114094704194" style="zoom:67%;" />

- Fanout Exchange

    每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。**fanout 交换器不处理路由键**，只是简单的将队列绑定到交换器上，**每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上**。很像子网广播，每台子网内的主机都获得了一份复制的消息。**fanout 类型转发消息是最快的**。

<img src="FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114094746294.png" alt="image-20201114094746294" style="zoom: 67%;" />

- Topic Exchange

    **topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上**。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号 `#` 和符号 `*`。`#` 匹配 0 个或多个单词，`*` 匹配一个单词。

<img src="FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114094820742.png" alt="image-20201114094820742" style="zoom:67%;" />

## 四、RabbitMQ整合

- 步骤一：虚拟机安装 MQ：

    - `docker pull registry.docker-cn.com/library/rabbitmq:3-management`
    - `docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq 加上IMageID`
    - 使用 `主机IP地址:15672`就进入管理页面了。

- 步骤二：管理页面配置新增 Exchange，如下图所示，分别配置：`exchange.direct`、`exchange.fanout`、`exchange.topic`。

    ![image-20201114094916462](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114094916462.png)

    ![image-20201114102719735](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114102719735.png)

- 步骤三：添加消息队列：Queues，将下图四个都添加上去

    ![image-20201114102841323](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114102841323.png)

- 步骤四：将 Exchange 和 Queues 按照下图进行绑定

    ![image-20201114103023744](FrameDay06_10%20SpringBoot%E4%B8%8E%E6%B6%88%E6%81%AF.resource/image-20201114103023744.png)

- 引入spring-boot-starter-amqp

    对应的自动配置类：`RabbitAutoConfiguration`

    - 通过方法 `public CachingConnectionFactory rabbitConnectionFactory(RabbitProperties config){}` 可以获取连接工厂，从中可以获取和 RabbitMq 的连接。其中参数  RabbitProperties 封装了 RabbitMq 的所有配置。

        所以在 `application.properties` 配置为：

        ```properties
        spring.rabbitmq.host=118.24.44.169
        spring.rabbitmq.username=guest
        spring.rabbitmq.password=guest
        # 如果不写也是默认访问 / ，见 RabbitProperties 类
        #spring.rabbitmq.virtual-host=
        ```
        
-  通过方法 `public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {}` 获取 RabbitTemplate，来给 RabbitMQ 发送和接受消息。
    
- 通过方法 `public AmqpAdmin amqpAdmin(ConnectionFactory connectionFactory) {}` 获取 AmqpAdmin，是 RabbitMQ 的系统管理功能组件。
    
- application.yml配置

- 测试RabbitMQ

    - AmqpAdmin：管理组件
    - RabbitTemplate：消息发送处理组件

    收发测试见：`Springboot02AmqpApplicationTests` 类



如果没有在 MQ 后台创建好 Exchange、Queues 以及绑定规则，需要**在程序中**临时创建一些使用，使用：`AmqpAdmin`，可以用来创建和删除 Queues、Exchange、Binding 等等。

测试程序见测试类。