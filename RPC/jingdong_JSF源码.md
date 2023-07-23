> 原文链接：http://xingyun.jd.com/shendeng/article/detail/8594

jsf源码学习分享（一）

本文主要是以一个初学者的身份对京东内部rpc框架jsf进行一个学习梳理。

本文所研究的版本是 jsf 1.7.4HOT_FIX_T4,让我们从源码走起，一步步来剖析其中的原理。

首先是从我们平常写的最多的jsf.xml，在jsf与spring的集成中，jsf的jar包的spring.handlers定义了解析jsf的xml文件的类JSFNamespaceHandler 。在该类中定义jsf的标签将被解析成什么样的类。

```
public class JSFNamespaceHandler extends NamespaceHandlerSupport {
    public JSFNamespaceHandler() {
    }

    public void init() {
        this.registerBeanDefinitionParser("provider", new JSFBeanDefinitionParser(ProviderBean.class, true));
        this.registerBeanDefinitionParser("consumer", new JSFBeanDefinitionParser(ConsumerBean.class, true));
        this.registerBeanDefinitionParser("consumerGroup", new JSFBeanDefinitionParser(ConsumerGroupBean.class, true));
        this.registerBeanDefinitionParser("server", new JSFBeanDefinitionParser(ServerBean.class, true));
        this.registerBeanDefinitionParser("registry", new JSFBeanDefinitionParser(RegistryConfig.class, true));
        this.registerBeanDefinitionParser("annotation", new JSFBeanDefinitionParser(AnnotationBean.class, true));
        this.registerBeanDefinitionParser("parameter", new JSFParameterDefinitionParser(ParameterConfig.class));
        this.registerBeanDefinitionParser("filter", new JSFBeanDefinitionParser(FilterBean.class, true));
        this.registerBeanDefinitionParser("connStrategy", new JSFBeanDefinitionParser(ConnStrategyBean.class, true));
    }
}
```

### ﻿ 

## 入口：

我们来关心下最核心的几个。首先是解析procider标签的ProviderBean，是用来提供服务的。

```
public class ProviderBean<T> extends ProviderConfig<T> implements InitializingBean, DisposableBean, ApplicationContextAware, ApplicationListener, BeanNameAware {
    private static final long serialVersionUID = -6685403797940153883L;
    private static final Logger LOGGER = LoggerFactory.getLogger(ProviderBean.class);
    private transient ApplicationContext applicationContext;
    protected transient String beanName;
    private transient boolean supportedApplicationListener;

    protected ProviderBean() {
    }

    public void setBeanName(String name) {
        this.beanName = name;
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        if (applicationContext != null) {
            try {
                Method method = applicationContext.getClass().getMethod("addApplicationListener", ApplicationListener.class);
                method.invoke(applicationContext, this);
                this.supportedApplicationListener = true;
            } catch (Throwable var5) {
                if (applicationContext instanceof AbstractApplicationContext) {
                    try {
                        Method method = AbstractApplicationContext.class.getDeclaredMethod("addListener", ApplicationListener.class);
                        if (!method.isAccessible()) {
                            method.setAccessible(true);
                        }

                        method.invoke(applicationContext, this);
                        this.supportedApplicationListener = true;
                    } catch (Throwable var4) {
                    }
                }
            }
        }

    }

    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ContextRefreshedEvent && this.isDelay() && !this.exported && !CommonUtils.isUnitTestMode()) {
            LOGGER.info("JSF export provider with beanName {} after spring context refreshed.", this.beanName);
            if (this.delay < -1) {
                Thread thread = new Thread(new Runnable() {
                    public void run() {
                        try {
                            Thread.sleep((long)(-ProviderBean.this.delay));
                        } catch (Throwable var2) {
                        }

                        ProviderBean.this.export();
                    }
                });
                thread.setDaemon(true);
                thread.setName("DelayExportThread");
                thread.start();
            } else {
                this.export();
            }
        }

    }

    private boolean isDelay() {
        return this.supportedApplicationListener && this.delay < 0;
    }

    public void afterPropertiesSet() throws Exception {
        if (this.applicationContext != null) {
            Map registryMaps;
            ArrayList registryLists;
            Collection registryConfigs;
            if (this.getServer() == null) {
                registryMaps = this.applicationContext.getBeansOfType(ServerConfig.class, false, false);
                registryLists = null;
                if (CommonUtils.isNotEmpty(registryMaps)) {
                    registryConfigs = registryMaps.values();
                    if (CommonUtils.isNotEmpty(registryConfigs)) {
                        registryLists = new ArrayList(registryConfigs);
                    }
                }

                super.setServer(registryLists);
            }

            if (this.getRegistry() == null) {
                registryMaps = this.applicationContext.getBeansOfType(RegistryConfig.class, false, false);
                registryLists = null;
                if (CommonUtils.isNotEmpty(registryMaps)) {
                    registryConfigs = registryMaps.values();
                    if (CommonUtils.isNotEmpty(registryConfigs)) {
                        registryLists = new ArrayList(registryConfigs);
                    }
                }

                super.setRegistry(registryLists);
            }

            registryMaps = this.applicationContext.getBeansOfType(FilterBean.class, false, false);
            Iterator i$ = registryMaps.entrySet().iterator();

            while(i$.hasNext()) {
                Entry<String, FilterBean> entry = (Entry)i$.next();
                FilterBean filterBean = (FilterBean)entry.getValue();
                if (filterBean.containsProvider(this.beanName)) {
                    List<AbstractFilter> filters = this.getFilter();
                    if (filters == null) {
                        List<AbstractFilter> filters = new ArrayList();
                        filters.add(filterBean.getRef());
                        this.setFilter(filters);
                    } else {
                        filters.add(filterBean.getRef());
                    }
                }
            }
        }

        if (!this.isDelay() && !CommonUtils.isUnitTestMode()) {
            LOGGER.info("JSF export provider with beanName {} after properties set.", this.beanName);
            this.export();
        }

    }

    public void destroy() throws Exception {
        LOGGER.info("JSF destroy provider with beanName {}", this.beanName);
        this.unexport();
    }
}
```

ProviderBean继承了providerConfig 并且实现了InitializingBean接口，Spring初始化bean的时候,如果bean实现了InitializingBean接口,会自动调用afterPropertiesSet方法.此方法会初始化容器。我们可以看到在afterPropertiesSet方法中。它首先会获取服务端列表和注册地址列表，之后会调用export()方法进行服务的暴露。

在export（）方法中，

1.先获取provider中配置的serverConfig，serverConfig也就是记录了提供服务的ip和端口，如果没有服务配置就抛出异常。再获取RegistryConfig，如果获取不到注册的地址，那么就会走默认的注册地址，，默认的索引地址就是“[i.jsf.jd.com](http://i.jsf.jd.com/)“

2.启动server，并且采用默认msgpack的方式进行注册服务编码，并且在服务端注册进程。

3.服务注册并且暴露出去。

## 启动服务

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-16-57njLiXswIxxw8qdT.png)

﻿﻿



serverConfig.start()首先创建服务端并启动。在这个过程中也会进行传输层的创建。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-17-26yOALiUMSlAxEV7g.png)

﻿﻿



首先会根据ServiceConfig去创建服务端，在getServer（）方法中，会去根据ServiceConfig转化成服务端传输层的配置。根据配置的端口寻找服务，如果没找到则根据不同的协议类型创建服务。。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-17-29dkE8W8ZPAINWjXH.png)

﻿﻿



在Server.start(）方法中，它会去获取传输层，根据传输协议的不同创建不同的传输层。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-17-44XS127xInNk72727XAH.png)

﻿﻿



### 服务传输层启动

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-17-33tPKUJzAA51HjEagC.png)

﻿﻿



启动传输层，可以看到使用的是netty,服务端会绑定端口， Netty中的IO操作是异步的,包括bind, write, connect等操作会简单的返回一个ChannelFuture。调用者不能立刻获得结果, 而是通过Future-listener机制, 用户可以方便的主动获取或者通过通知机制获得IO操作结果

我们应该重点关注Channel上的Handler，用于处理心跳包，客户端请求，加解密等

```
((ServerBootstrap)this.serverBootstrap
.group(this.config.getParentEventLoopGroup(),
 this.config.getChildEventLoopGroup())
.channel(clazz))
.childHandler(new ServerChannelInitializer(this.config));
```

### **向服务容器注册服务**

在ServiceConfig中，我们提到了在服务端注册进程，Server.registerProcessor(）以服务的interfaceId和alias组成实例名，向map中存入对应的invoker，

等到收到消费者消息后用invoker调用服务。

## 注册服务

在ProviderConfig中，会调用register（）方法。向注册中心注册服务，并且监听服务配置。有多个registryConfig,多实例部署保证高可用性(容灾)﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-17-52R0kXTVbomb12nLkT.png)

﻿注册的过程中会将配置类ProviderConfig转化为jsfUrl，jsfUrl是jsf的核心类，包含了一个提供的服务的ip,port，Interface等重要信息，。一个提供者如果有多台服务，那么就会产生多个jsfUrl。jsfUrl在我们的应用中是常见日志。﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-11-17-55HTGw11qE8RtmC55Jw.png)

﻿并且JsfContext会将配置信息维护起来。至此，服务端的启动与注册基本完成。



# jsf源码学习分享（二）

上文中我们聊到了服务端的发布与注册。接下来我们来聊聊消费端的启动。

ConsumerBean继承了ConsumerConfig类，ConsumerBean实现了FactoryBean接口，这就意味着需要调用getObject（）来获取真正的对象，这是因为我们的Consumer端只能调用接口，接口是无法直接使用的，它需要被动态代理封装，产生代理对象，再把代理对象放入Spring容器中供我们使用。这也就是平常为什么写个xml文件，spring容器就创建了实例了。

在getObject（）方法中ConsumerBean会调用父类consumerConfig的refer（）方法从而开始了客户端的初始化的过程。

在refer（）过程中，主要做了三件事情：

1.会产生一个Client的实例。 在这个过程中，consumer会去订阅相关的provider的服务。

2产生一个客户端代理invoker来对服务端进行实际的调用。

3.默认通过javassist动态代理对接口产生一个代理实例。因为消费端是一个接口，没有办法进行实际的调用。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-18-18-100dmsUD618IdTgcUE.png)

﻿﻿



在创建client中，如果不存在连接时，Client会先初始化连接。在初始化连接的过程中，首先要找到服务提供者列表，也就是进行订阅处理，然后建立连接。

### 订阅服务：

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-19-00A7tIalNintN14TaZ.png)

﻿﻿



﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-13ABItnZLkZrqKfOJ.png)

﻿﻿



获得所有的服务提供者:调用一次订阅方法，同步返回可用的provider列表，并且进行连接，然后开启心跳检测,

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-19-47TrIufyGtMqcvqYb.png)

﻿﻿



获取所有的registryConfig，因为注册中心地址有多个，获得所有服务需向所有注册中心去订阅.

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-19-48VI7dmVbkyGqbzy7.png)

﻿﻿



注册订阅核心接口

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-19-50IwGZR7vr8bYtCRo.png)

﻿﻿



注册订阅核心类可注册provider变更的监听器，实现增量添加、删除与全量更新的provider变更事件。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-19-55XxO59Y9TBzoFMrKG.png)

﻿﻿



订阅url包含了相关的回调事件

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-01119kFfkoB19714A68b.png)

﻿﻿



### 初始化连接：

我们可以看到这里会初始化一个名为JSF-CLI-CONN-#interfaceId的线程池，这个线程池中执行任务：获取到一个ClientTransport对象

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-14fkTLxZmvJe14j14Xp.png)

﻿﻿



由于有多个服务提供者，因此创建多个连接，使用连接池进行连接复用，如果不存在连接，则初始化连接

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-17vNmKMDlW8ERZa8v.png)

﻿﻿



和服务端同样，根据协议的不同创建不同的传输层

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-18xCM0Uob814Bpq78L.png)

﻿﻿



这里可以看到也是通过netty进行网络通信，会根据服务端的相关参数建立连接，绑定到相应的ip，并监听服务器端的端口号，通过对应的通道做数据传输。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-19Bp9HAidJSHTcJUq.png)

﻿﻿



和服务端的pipeline类似，可以看到JSFEncoder/JSFDecoder用来解码，编码具体的协议。责任链尾部是ClientChannelHandler，用于发送请求和接受服务端的响应。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-194512N7PmwbUk814A14V.png)

﻿﻿



### 调用服务：

   在前面我讲到，是ClientProxyInvoker对服务进行的真正的调用，由于动态代理的缘故，调用消费端的接口就是调用ClientProxyInvoker的invoke方法。

   现在服务端和客户端已经能够建立网络连接，此时可以发起请求调用。invoke方法首先根据ConsumerConfig构建RequestMessage。最终核心调用是ResponseMessage response =this.filterChain.invoke(requestMessage);    filterChain是一条链条，在ClientProxyInvoker的初始化中，会形成一条链条会根据消费者不同的配置情况下进行依次处理，链条的尾端是ConsumerInvokeFilter，即最终会调用Client进行发送消息。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-20-20QX9t9JlFyb9m77q.png)

﻿﻿



由于作者水平有限，难免有不足乃至错误之处，敬请斧正。

# jsf源码学习分享（三）

在本文中，我们主要来分析，发起调用的过程究竟发生了什么。

### 1.客户端动态代理发送消息

上文我们说到调用消费端的接口就是调用ClientProxyInvoker的invoke方法。

在实例化ClientProxyInvoker的过程中，会初始化ClientProxyInvoker的三个成员变量，我们重点关注下成员变量filterChain。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-22-22-33RARhcPTMFXIu9aA.png)

﻿﻿



我们可以看到这条链条尾端的Filter是ConsumerInvokeFilter。在链条的构建过程中，主要是根据consumerConfig的属性来决定加入不同的过滤器

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-22-22-34TV12hMx0jVwwMbXv.png)

﻿﻿



在ConsumerInvokeFilter的invoke方法中，它会调用客户端Client.sendMsg方法。从而进入到传输层进行发送消息。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-29-23-14ZZn19uQ29s8b612Q299.png)

﻿﻿



ConsumerInvokeFilter是通过Client发送消息的。首先会通过Client的实现类中，发起调用。客户端如果调用失败会有不同的容错策略。默认的客户端是FailOverClient.，于是在默认的情况下会是下生成FailOverClient做相关的发送消息操作.

failover：发送消息成功获得结果，则直接返回。否则重试至最大次数。

重试过程中会将已经调用过的的服务加入到一个集合中，然后将可连接健康状态的服务中移除已经调用过的的服务，利用负载均衡算法在健康服务中重新选出一个服务发送。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-40DPte43R0HAng12RhG.png)

﻿﻿



﻿

获取到的是所有提供这个服务的服务端信息列表，我们就需要从中选择一个，这便涉及到客户端侧的负载均衡策略

| 负载均衡算法              | 作用                                                         |
| :------------------------ | :----------------------------------------------------------- |
| RandomLoadbalance(默认）  | 按权重的进行随机，可以方便的调整,调用量越大分布越均匀        |
| ConsistentHashLoadbalance | 一致性哈希算法                                               |
| LeastActiveLoadbalance    | 在调用前后增加计数器，并发小的Provider认为是快的Provider，并发大的Provider认为是慢的Provider。并发数相同再比较权重。这样快的Provider收到更多请求。 |

 **FailFastClient：**调用失败则抛出异常信息。

**客户端发送消息**

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-16-00-10LpOBdgQab7lJaq8.png)

﻿﻿



在Client的sendMsg0方法中，它会去调用transport.send方法，最终会通过传输层发送消息。

具体的调用顺序，我们也可以在jsf客户端的报错日志发现端倪。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-29-22-59FZEc0M29JnDTXArb.png)

﻿﻿



我们先看看AbstractTCPClientTransport，根据每个请求唯一的requestId将返回结果放入对应的future中，然后通过future.get方法获取结果。

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-44EdLieghzPyjvsAL.png)

﻿﻿



JSFClientTransport将future放入concurrentHashMap中，

并且根据传输层不同的协议构建request

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-45ZmYxU146CGWU0Wot.png)

﻿﻿



将调用数据写入到bytebuf，buyebuf可以理解为netty的字节容器，然后封装到requestMessage

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-46T6CQ14lZd7uY0SFa.png)

﻿﻿



JSFClientTransport通过channel通道发送消息

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-46XasT0ALPsxLDY6X.png)

﻿﻿



### 2.服务端通过反射调用获得结果

ServerChannelHandler 分别位于服务端的首部，channelRead方法中用于接收从客户端发送过来的消息，将处理rpcRequest的逻辑包装成一个任务，扔给线程池来处理

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-47m14PPu14d12rigGdnW.png)

﻿﻿



JSFTask实现了runnable()接口，在run（）方法中首先调用了JSFTask的doRun（）方法，使用ProviderProxyInvoker处理消息，invoker是远程调用的抽象，根据接口名和alias从map中得到ProviderProxyInvoker进行调用，得到responseMessage

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/image2022-11-27_10-26-20.png)

﻿﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-47iVDcGnTYq106Triz.png)

﻿

﻿



ProviderProxyInvoker返回结果同样是经过一段链条，最终走到ProviderInvokeFilter 使用反射调用获得最终结果

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-48cKBDnWE14QWe10LVp.png)

﻿﻿



﻿

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-29-22-51dW68LuYmO29ZxjWv.png)

﻿﻿



Run()方法中在通过JSFTask的doRun（）方法得到responseMessage后,调用onResponse()方法，在该方法中调用write（）方法将对responseMessage构建并通过channel返回给客户端

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-48ulfEmbwxHDde6v7.png)

﻿﻿



客户端ClientChannelHandler收到响应消息后，取出其中的requestId，根据requestId找到对应的future,j将msg放入future中，将相应的键值对从concurrentHashMap中移除。然后在future.get()中得到responseMessage，这样也就得到了最终结果！

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-48489l46AE7lif6mZ10a.png)

﻿﻿



﻿

﻿

![img](jingdong_JSF%E6%BA%90%E7%A0%81.resource/2022-12-14-22-48PcT6li53B10NKzeih.png)

﻿



﻿﻿