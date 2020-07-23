---
tags:
- 未看
---

# JAVA中几种常用的RPC框架介绍

[原文链接地址](https://zhuanlan.zhihu.com/p/46818259)



RPC是远程过程调用的简称，广泛应用在大规模分布式应用中，作用是有助于系统的垂直拆分，使系统更易拓展。Java中的RPC框架比较多，各有特色，广泛使用的有RMI、Hessian、Dubbo等。RPC还有一个特点就是能够跨语言，本文只以JAVA语言里的RPC为例。

对于RPC有一个逻辑关系图，以RMI为例：

![](https://pic3.zhimg.com/80/v2-f771e70d45bc8b69435363da62adc72a_hd.jpg)

其他的框架结构也类似，区别在于对象的序列化方法，传输对象的通讯协议，以及注册中心的管理与failover设计（利用zookeeper）。

客户端和服务端可以运行在不同的JVM中，Client只需要引入接口，接口的实现以及运行时需要的数据都在Server端，RPC的主要依赖技术是序列化、反序列化和传输协议，JAVA里对应的就是对象的序列化、反序列化以及序列化后数据的传输。RMI的序列化和反序列化是JAVA自带的，Hessian里的序列化和反序列化是私有的，传输协议则是HTTP，Dubbo的序列化可以多种选择，一般使用Hessian的序列化协议，传输则是TCP协议，使用了高性能的NIO框架Netty。对于序列化，我还了解一些，像Google的ProBuffer、JBoss Marshalling和Apache Thrift等，之前有写一篇介绍ProBuffer的博文

1、RMI（远程方法调用）

JAVA自带的远程方法调用工具，不过有一定的局限性，毕竟是JAVA语言最开始时的设计，后来很多框架的原理都基于RMI，RMI的使用如下：

对外接口

```text
<span style="font-size:12px;">public interface IService extends Remote {

    public String queryName(String no) throws RemoteException;

}</span>
```

**服务实现**

```text
import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

// 服务实现
public class ServiceImpl extends UnicastRemoteObject implements IService {

    /**
     */
    private static final long serialVersionUID = 682805210518738166L;

    /**
     * @throws RemoteException
     */
    protected ServiceImpl() throws RemoteException {
        super();
    }

    /* (non-Javadoc)
     *
     */
    @Override
    public String queryName(String no) throws RemoteException {
        // 方法的具体实现
        System.out.println("hello" + no);
        return String.valueOf(System.currentTimeMillis());
    }

}

```

**RMI客户端**

```text
import java.rmi.AccessException;
import java.rmi.NotBoundException;
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

// RMI客户端
public class Client {

    public static void main(String[] args) {
        // 注册管理器
        Registry registry = null;
        try {
            // 获取服务注册管理器
            registry = LocateRegistry.getRegistry("127.0.0.1",8088);
            // 列出所有注册的服务
            String[] list = registry.list();
            for(String s : list){
                System.out.println(s);
            }
        } catch (RemoteException e) {

        }
        try {
            // 根据命名获取服务
            IService server = (IService) registry.lookup("vince");
            // 调用远程方法
            String result = server.queryName("ha ha ha ha");
            // 输出调用结果
            System.out.println("result from remote : " + result);
        } catch (AccessException e) {

        } catch (RemoteException e) {

        } catch (NotBoundException e) {

        }
    }
}

```

**RMI服务端**

```text
import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

// RMI服务端
public class Server {

    public static void main(String[] args) {
        // 注册管理器
        Registry registry = null;
        try {
            // 创建一个服务注册管理器
            registry = LocateRegistry.createRegistry(8088);

        } catch (RemoteException e) {

        }
        try {
            // 创建一个服务
            ServiceImpl server = new ServiceImpl();
            // 将服务绑定命名
            registry.rebind("vince", server);

            System.out.println("bind server");
        } catch (RemoteException e) {

        }
    }
}

```

服务注册管理器写在了Server里，当然也可以抽出来单独作为一个服务，在其他一些框架中，往往用Zookeeper充当注册管理角色。

2、Hessian（基于HTTP的远程方法调用）

基于HTTP协议传输，在性能方面还不够完美，负载均衡和失效转移依赖于应用的负载均衡器，Hessian的使用则与RMI类似，区别在于淡化了Registry的角色，通过显示的地址调用，利用HessianProxyFactory根据配置的地址create一个代理对象，另外还要引入Hessian的Jar包。

![](https://pic1.zhimg.com/80/v2-f20d91abb298359dc64b7c4bc76b578c_hd.jpg)

**3、Dubbo（淘宝开源的基于TCP的RPC框架）**

基于Netty的高性能RPC框架，是阿里巴巴开源的，总体原理如下：

![](https://pic4.zhimg.com/80/v2-58b20e319f8bd0ce40e472315b9d3377_hd.jpg)

在了解Dubbo之前，要先对Zookeeper有深入的理解，当理解了zookeeper后，Dubbo也就了无秘密了。

Dubbo的详细说明在淘宝开源里说的非常详细，在工作中很多生产项目都用了Dubbo，过程中也发现了很多需要注意的地方，尤其是那繁多的配置，设置不当都会让人烦脑，最好能再基于现有开源的Dubbo再定制优化一下。

欢迎工作一到五年的Java工程师朋友们加入Java架构开发：855801563

群内提供免费的Java架构学习资料（里面有高可用、高并发、高性能及分布式、Jvm性能调优、Spring源码，MyBatis，Netty,Redis,Kafka,Mysql,Zookeeper,Tomcat,Docker,Dubbo,Nginx等多个知识点的架构资料）合理利用自己每一分每一秒的时间来学习提升自己，不要再用"没有时间“来掩饰自己思想上的懒惰！趁年轻，使劲拼，给未来的自己一个交代