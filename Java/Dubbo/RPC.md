https://www.bilibili.com/video/BV17Z4y1s7cG?from=search&seid=1171719910651571127



# RPC

## 前言

 RPC 是一种概念，是一种远程通信方式，具体实现有很多种。

## 一、名词解析

从单机走向分布式产生了很多分布式的通信方式

- 最古老也是最有效,并且永不过时的,TCP/DP的二进制传输。事实上所有的通信方式归根结底都是TCP/UDP

- CORBA (Common Object Request Broker Architecture)。古老而复杂的,支持面向对象的通信协议

- Web Service(SOA SOAP RDDI WSDL…)

    基于 http+xml 的标准化 Web API，服务端提供服务的接口是 xml 格式，并且该格式在 HTTP 上进行传输。因为 xml 本身大，就是纯文本， HTTP 本身传输也是纯文本，所以慢。

- RestFul ( Representational State Transfer)

    回归简单化本源的 Web API 的事实标准h ttp://mashibing.com/product/java

    http + json

- RMI Remote Method InvocationJava

    内部的分布式通信协议，只支持 Java 语言。

- JMS (Java Message Service)

    JavaEE 中的消息框架标准,为很多MQ所支持

- RPC(Remote Procedure Call) 

    远程方法调用,这只一个统称,重点在于方法调用(不支持对象的概念),具体实现甚至可以用 RMI RestFul 等去实现,但一般不用.因为 RMI 不能跨语言.而 RestFul效率太低。多用于服务器集群间的通信,因此常使用更加高效短小精悍的传输模式以提高效率。



## RPC组成 

一个 RPC 的核心功能主要有 5 个部分组成，分别是：客户端、客户端 Stub、网络传输模块、服务端 Stub、服务端等。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912232743848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)

- 下面分别介绍核心 RPC 框架的重要组成：
    客户端(Client)：服务调用方。
    客户端存根(Client Stub)：存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端。
    服务端存根(Server Stub)：接收客户端发送过来的请求消息并进行解包，然后再调用本地服务进行处理。
    服务端(Server)：服务的真正提供者。
    Network Service：底层传输，可以是 TCP 或 HTTP。

### TCP/IP 模拟RPC

由服务的调用方与服务的提供方建立 Socket 连接，并由服务的调用方通过 Socket 将需要调用的接口名称、方法名称和参数序列化后传递给服务的提供方，服务的提供方反序列化后再利用反射调用相关的方法。

***将结果返回给服务的调用方，整个基于 TCP 协议的 RPC 调用大致如此。



## 项目搭建

### 基本包：common

主要包括类 User、Product，用于后续传输，同时提供两个访问方法，就是通过 id 返回对象（这里 id 限定为 123）

- 实体类

    - User

        ```java
        package com.gjxaiou;
        
        import java.io.Serializable;
        
        public class User implements Serializable {
            private static final long serialVersionUID = 1L;
            int id;
            String name;
            // 省略构造方法、Getter、Setter 和 toString 方法。
        }
        ```

    - Product

        ```java
        package com.gjxaiou;
        
        import java.io.Serializable;
        
        public class Product implements Serializable {
            private static final long serialVersionUID = 1L;
            int id;
            String name;
            int count;
            // 省略构造方法、Getter、Setter 和 toString 方法。
        }
        ```

- 提供对外接口，获取这两个对象

    - IUserService

        ```java
        package com.gjxaiou;
        
        public interface IUserService {
            User findUserById(int id);
        }
        ```

    - IProductService

        ```java
        package com.gjxaiou;
        
        public interface IProductService {
            Product findProductByName(String name);
        }
        ```

## 01 最基础二进制传递

首先提供一个类 User，然后暴露一个访问接口 IUserService，供别人访问



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912215655826.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)

## 02 简化客户端的流程 引入stub(客户端存根)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912220900187.png#pic_center)

#### 客户端存根(Client Stub)：存放服务端地址信息，将客户端的请求参数数据信息打包成网络消息，再通过网络传输发送给服务端。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912221405888.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)

## 03 使用动态代理生成service类供客户端调用 彻底隐藏所有网络细节

首先使用一个类实现服务器端暴露的 service 接口，然后在这个实现类中添加网络访问的步骤。

但是如果每次暴露一个接口就需要一个实现类，所以能不能动态的产生这个类。





- 在02的client中 用stub调用方法 底层暴露了 希望用封装的Service调用 而且隐藏底层细节
    于是引入动态代理 生成service类给client使用 在stub中也不用写每一个业务函数的细节
- 在02的stub中 随着业务的增多方法会越来越多 那就会堆积很多函数 需要反射来使手写的函数减少
    当外部使用返回的动态代理对象去调用某个功能的时候 内部就会method反射

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912224808534.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912225423727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912225834865.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912232049227.png#pic_center)

## 04 引入客户端存根的真正含义-支持多个方法的打包 服务端利用反射解析打包过来的消息并invoke执行

client还是调用希望调用的函数 stub却对函数进行了规则化的传递 不在在自己这里处理了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912232022499.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)
server:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912231952580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)
现在支持不同功能的函数经过存根打包 在网络中传输给服务端 那服务端也要对相应的服务进行解析并且返回函数

## 05 服务端支持不同参数的函数的返回结果

服务端写回一个对象 而不是在客户端哪里获得属性然后组装对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914154503178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)

在客户端存根当中 服务端写回的结果进行了改变
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914154353867.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)

## 06 服务端支持对不同类的不同函数、不同参数的结果返回

存根就是网络消息打包存放的地方
现在已经升级为通过动态代理 支持所有的类的所有的函数并且根据类型和参数能够区别重写的函数
返回结果也支持所有的类型
目前就是客户端做调用 然后存根进行打包运输给服务端
服务端解析并且返回对象 存根再将对象写回
![ ](https://img-blog.csdnimg.cn/20200914160007120.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091416021859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)
server 获取名字
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200914160556927.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NzMTIzbWxr,size_16,color_FFFFFF,t_70#pic_center)

### 总结

一次 RPC 调用流程如下：

服务消费者(Client 客户端)通过本地调用的方式调用服务。
客户端存根(Client Stub)接收到调用请求后负责将方法、入参等信息序列化(组装)成能够进行网络传输的消息体。
客户端存根(Client Stub)找到远程的服务地址，并且将消息通过网络发送给服务端。
服务端存根(Server Stub)收到消息后进行解码(反序列化操作)。
服务端存根(Server Stub)根据解码结果调用本地的服务进行相关处理
服务端(Server)本地服务业务处理。
处理结果返回给服务端存根(Server Stub)。
服务端存根(Server Stub)序列化结果。
服务端存根(Server Stub)将结果通过网络发送至消费方。
客户端存根(Client Stub)接收到消息，并进行解码(反序列化)。
服务消费方得到最终结果

–
马老师讲的是本机的rpc调用 在微服务框架中微服务应用都是在不同的JVM、不同的内存中
这时候简单的动态代理和反射就可能调用到其他微服务的内存空间里了 甚至会造成并发的一个错乱
所以就引入了 相关的注册中心（Eureka zookeeper） 服务发现 根据ID

- 在 RPC 中，所有的函数都必须有自己的一个 ID。这个 ID 在所有进程中都是唯一确定的。客户端和服务端分别维护一个函数和Call ID的对应表。
- 客户端想要调用函数A 就查找自己所的对应表把A对应的ID通过存根传输 服务端根据ID在自己这边的表中找到函数 并执行
- 网络传输层需要把 Call ID 和序列化后的参数字节流传给服务端，然后再把序列化后的调用结果传回客户端。