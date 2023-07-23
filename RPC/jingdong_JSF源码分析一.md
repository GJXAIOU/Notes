> 原文：http://xingyun.jd.com/shendeng/article/detail/4191

## 一、架构设计

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181606819.png)

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181607548.png)

## 二、1.7.4-HOTFIX-T4版本包布局及简要含义

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181606235.png)

看过了全包的简要，那么其核心的功能模块，就从常用的项目xml配置出发，便于我们的理解。如下：

## 三、jsf-provider.xml配置

以我们地址服务的jsf-provider.xml文件为例，即：

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608317.png)

可以看到，在JSF的配置文件中，我们并没有看到任何关于注册中心的内容。说到底，作为（集团自主研发的高效）RPC调用框架，其高可用的注册中心重中之重，所以带着这份疑惑，继续往下探究，没有注册中心地址，这些标签是怎么完成服务的注册，订阅的。

## 四、配置解析

在Spring的体系中，Spring提供了可扩展Schema的支持，即自定义的标签解析。

1、首先我们发现配置文件中自定义的xsd文件，在标签名称上找到NamespaceUri链接http://jsf.jd.com/schema/jsf/jsf.xsd

2、然后根据SPI加载，在META-INF中找到定义好Spring.handlers文件和Spring.schemas文件，一个是具体的解析器的配置，一个是jsf.xsd的具体路径

```java
Spring.handlers文件内容：
http\://jsf.jd.com/schema/jsf=com.jd.jsf.gd.config.spring.JSFNamespaceHandler
--------------------------------------------------------
Spring.schemas文件内容：
http\://jsf.jd.com/schema/jsf/jsf.xsd=META-INF/jsf.xsd
```

3、由此我们进一步查询继承NameSpaceHanderSupport或者实现NameSpaceHandler对应的接口类，在我们的jsf框架中JSFNamespaceHandler是采用继承前者（NameSpaceHanderSupport）去实现的，即：

### （一）com.jd.jsf.gd.config.spring.JSFNamespaceHandler

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608797.png)

【补充】**NamespaceHandler的功能就是解析我们自定义的JSF命名空间的**，为了方便起见，我们实现NamespaceHandlerSupport，其内部通过BeanDefinitionParser对具体的标签进行处理，即对我们定义的标签进行具体处理。

### （二）com.jd.jsf.gd.config.spring.JSFBeanDefinitionParser#parse

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608256.png)

4、**最终这些配置（也就是我们在xml中配置的标签值）会解析成为ServerConfig和ProviderConfig，并且会依据所配置的属性，对相应的类进行属性的赋值**。

### （三）com.jd.jsf.gd.config.ServerConfig

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610390.png)

### （四）com.jd.jsf.gd.config.ProviderConfig

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608285.png)

## 五、初始化

OK，我们回到JSFNamespaceHandler看一下服务是如何暴露的。众所周知，在Spring容器中的bean一定会经历一个初始化的过程。所以通过com.jd.jsf.gd.config.spring.JSFBeanDefinitionParser实现org.springframework.beans.factory.xml.BeanDefinitionParser来进行xml的解析，另外通过ParserContext中封装了BeanDefinitionRegistry对象，用于BeanDefinition的注册，用来初始化各个bean，即（com.jd.jsf.gd.config.spring.JSFBeanDefinitionParser#parse）。

如下：bean类ProviderBean会监听上下文事件，并且当整个容器初始化完毕之后会调用export()方法进行服务的暴露。

### com.jd.jsf.gd.config.spring.ProviderBean

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181607257.png)

## 服务暴露

我们回到源码中，发现其核心代码逻辑如下图：

### com.jd.jsf.gd.config.ProviderConfig#doExport

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608773.png)

com.jd.jsf.gd.config.ProviderConfig#doExport 该方法的整体逻辑如下：

1、首先进行各种基本的校验和拦截，如：

- [JSF-21200]provider的alias不能为空；
- [JSF-21202]providerconfig的server为空；
- [JSF-21203]同一接口+alias的provider配置了多个；
- ...

2、其次获取所有的RegistryConfig，如果获取不到注册的地址，那么就会走默认的注册中心地址：“i.jsf.jd.com”。

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181607234.png)

### com.jd.jsf.gd.config.RegistryConfig

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link.png)

3、然后获取provider中配置的server，如果provider中存在server相关的配置，即是com.jd.jsf.gd.config.ServerConfig，此时会启动server（serverConfig.start()），并且采用默认对应的序列化方式（serverConfig.getSerialization()，默认msgpack）进行注册服务编码。

### com.jd.jsf.gd.config.ServerConfig#start（服务启动）

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181606257.png)

4、start方法中会调用ServerFactory中的方法产生相对应的对象，然后会调用Server中的方法去启动Server，而在这个过程中，最终会对应到ServerTransportFactory产生相应的传输层，即是定位到JSFServerTransport的start（）方法，在这里我们可以看到，该部分实现了netty框架的transport层，在进入这个方法的时候，会根据是否配置使用epoll模型来选择所生成的对象是EpollServerSocketChannel或者NioServerSocketChannel，然后在ServerBootStrap中初始化相关的参数，直到最后绑定好端口号。

### com.jd.jsf.gd.server.ServerFactory#initServer

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181606180.png)

### com.jd.jsf.gd.transport.JSFServerTransport#start

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181606688.png)

5、最后通过（this.register();）服务注册并且暴露出去，其中JsfRegistry这个类的对象，在该类的构造函数中会连接jsf的注册中心，如果注册中心不可用的话，会生成并使用本地的文件并且开始守护线程，并使用两个线程池去发送心跳检测以及重试机制，另外一个线程池去检测连接是否成功（com.jd.jsf.gd.registry.JSFRegistry#addRegistryStatListeners）。

### com.jd.jsf.gd.registry.JSFRegistry

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181607858.png)

6、服务注册（com.jd.jsf.gd.registry.JSFRegistry#register）的过程会将对应的ProviderConfig转换为JsfUrl类，这里JsfUrl是整个框架的核心，他保存了一系列的配置，并且和其同样重要的还有订阅Url类SubscribeUrl，这里JsfUrl属于服务Url，服务Url中保存了协议，端口号，ip地址等相关重要的信息，并且回到上层JsfContext会将配置信息维护起来（JSFContext.cacheProviderConfig(this);）。到这里（this.exported = true;）provider的服务从配置装配到服务暴露就完成了。

### com.jd.jsf.gd.registry.JSFRegistry#register

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181606986.png)

### com.jd.jsf.vo.JsfUrl

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608490.png)

## jsf-consumer.xml配置

以上是完成了provider服务的暴露，那么我们回到consumer中，看一下，如下：

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181609522.png)

我们在上方的配置文件中发现到了注册中心地址i.jsf.jd.com，也就是说服务注册相关的配置没有写到jsf-provider.xml端，只是配置到了jsf-consumer.xml中而已。

## 配置解析&初始化

配置解析过程是同上，就不多做赘述了，最终这些配置会解析成为ConsumerConfig和RegistryConfig，并且会依据所配置的属性，对相应的类进行属性的赋值。

### com.jd.jsf.gd.config.ConsumerConfig->AbstractConsumerConfig

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610147.png)

最终初始化映射到ConsumerBean类。

### com.jd.jsf.gd.config.spring.ConsumerBean

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181608914.png)

## 服务订阅

不过我们发现其实现了FactoryBean，如我们所了解的，如果一个 bean 实现了该接口（FactoryBean），它被用作一个对象的工厂来暴露，而不是直接作为一个将自己暴露的 bean 实例。这也就意味着需要调用getObject（）来获取真正的实例化对象（可能是共享的或独立的）。之所以这样使用的原因在于我们的Consumer端只能调用接口，接口是无法直接使用的，它需要被动态代理封装，产生代理对象，再把它放入Spring容器中。因此使用FactoryBean其实只是为了方便创建代理对象而已。

在getObject（）方法中ConsumerBean会调用子类consumerConfig的refer（）方法，从而开始了客户端的初始化的过程。在refer（）过程中，consumer会去订阅相关的provider的服务。核心代码如下：

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610012.png)

### com.jd.jsf.gd.config.ConsumerConfig#refer

该方法的整体逻辑为，如下：

1、首先进行各种基本的校验和拦截，如：

- consumer的alias不能为空，请检查配置
- 同一个接口+alias+protocol 本地配置的超过三次，抛出启动异常
- ...

2、一些不同配置的逻辑，如是否泛化调用，是否走injvm调用等

3、通过工厂模式生成一个Client的实例，由于上面ConsumerConfig->AbstractConsumerConfig默认的集群策略failover，所以在没有配置的情况下会生成FailoverClient，然后进行相关的Invoke操作（this.proxyInvoker = new ClientProxyInvoker(this);）。

### com.jd.jsf.gd.client.ClientFactory#getClient

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610047.png)

【补充】目前JSF支持的集群策略有，failover：失败重试（默认）；failfast：失败忽略；pinpoint：定点调用；

4、在client中，先定义其负载均衡，然后判断是否需延迟建立长链接。否的话，会直接进行初始化连接。其次如果未定义路由规则，或者不存在连接时，Client会先初始化相关的路由以及初始化连接，如下：

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610008.png)

5、在初始化连接（initConnections）中会进行调用ConsumerConfig中的subscribe（）进行服务的订阅，并且在初始化的过程中，consumer会连接相应的Providers。

### com.jd.jsf.gd.client.Client#initConnections

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181609425.png)

6、我们看到在获取连接过程（connectToProviders（））中，如果连接池中已经存在则直接返回，不存在则需要重新建立连接。如下：会初始化一个名为JSF-CLI-CONN-#interfaceId的线程池，在线程池中会执行对应任务，即获取到一个ClientTransport对象。

### com.jd.jsf.gd.client.Client#connectToProviders

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610733.png)

如果连接（com.jd.jsf.gd.transport.ClientTransportFactory#initTransport），则：1）首先判断相关协议；

### com.jd.jsf.gd.transport.ClientTransportFactory#initTransport

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610593.png)

2）然后BuildChannel建立连接（com.jd.jsf.gd.transport.ClientTransportFactory#BuildChannel），在这里会对应到Server端启动代码的相关参数，即是该处会监听服务器端的端口号，绑定到相应的ip，并根据对应的通道做数据传输。

### com.jd.jsf.gd.transport.ClientTransportFactory#BuildChannel

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181610947.png)

3）并且，对于transport层，我们应该关注他注入了哪些pipeline：可以看到JSFEncoder/JSFDecoder用来解码，编码具体的协议。handler则是具体的channel处理器，用于处理心跳包，客户端请求，和消息路由。最后回到最上层（com.jd.jsf.gd.config.ConsumerConfig#refer）将配置文件维护起来（JSFContext.cacheConsumerConfig(this)；）。

### com.jd.jsf.gd.transport.ClientChannelInitializer#initChannel

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181612038.png)

综上简单的过了一遍各自功能的初始化加载方式等简单阐述了一下，不过阅读起来肯定是偏零散的，需要我们根据以上步骤和源码包进行进一步探索。另外我们也可以用服务器启动日志上，看到JSF核心流程的日志记录，部分截图如下（感兴趣可以在预发机器的启动日志中翻阅）：

# 服务器启动日志（JSF）

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181613870.png)

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181613502.png)

总的来说，其提供者Provider，消费者Consumer，注册Registry等关系应该如下图所示：

# Architecture

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181612285.png)

第一部分：Provider端启动服务向注册中心Session注册自己的服务，注册服务的形式如：[jsf://192.168.124.73:22000/?safVersion=210&jsfVersion=1691&interface=com.jd.wxt.material.service.DspTaskQueueService&alias=chengzhi36_42459] 此时在注册服务，暴露服务的过程中，此时JsfRegistry会进行初始化连接。在此过程中，如果有配置对应的Server端，那么还会对Server进行初始化.

第二部分：Consumer在第一次获取实体信息时，由于其是FactoryBean，故必须调用getObject（）方法去获取实体，在此过程会调ConsumerConfig的refer（）方法，即可将Client端注册到jsf注册中心并订阅对应alias下Provider的变化。在这个过程中会初始化Consumer以及Provider端的监听器，对相关的Consumer和Provider做事件监听。此过程会产生Client对象，并且在初始化该对象时，会默认的去使用随机的负载均衡策略，并且会初始化路由，初始化路由后，会调用ConsumerConfig的subscribe方法，然后会初始化客户端对之前启动的Server进行连接。

第三部分：注册中心会异步通知Consumer是否需要重新订阅Provider。

第四部分：就是直接调用方法invoke。

第五部分：Monitor对服务进行监控，治理或者降级容灾。

# 流程图

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181612700.png)

最终调用，如下：

com.jd.jsf.gd.filter.FilterChain#invoke

```
ResponseMessage response = this.filterChain.invoke(requestMessage);
```

![img](jingdong_JSF%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E4%B8%80.resource/link-20230712181612039.png)

# 参考资料

SPI： http://jnews.jd.com/circle-info-detail?postId=215333

NamespaceHandler：https://www.yisu.com/zixun/447867.html

jsf：https://cf.jd.com/pages/viewpage.action?pageId=132108361

netty：https://www.cnblogs.com/jing99/p/12515149.html

------

**本次就先写到这，文章中如有问题，欢迎留言斧正。也希望能和更多志同道合的伙伴沟通交流。后续会再更新关于JSF心跳检测、服务治理、服务反注册、钩子工具等模块细化的分析以及当前我们平台系统初步接入wormhole平台（中间件mesh化）的一些经验分享。欢迎大家点赞关注。**