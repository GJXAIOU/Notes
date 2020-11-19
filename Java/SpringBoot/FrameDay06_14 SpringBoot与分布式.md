# 六、Spring Boot与分布式

## 一、分布式应用

在分布式系统中，国内常用 zookeeper+dubbo 组合，而 Spring Boot 推荐使用全栈的 Spring，Spring Boot+Spring Cloud。

### 分布式系统架构演变

![image-20201114145152241](FrameDay06_14%20SpringBoot%E4%B8%8E%E5%88%86%E5%B8%83%E5%BC%8F.resource/image-20201114145152241.png)

- **单一应用架构**

    当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作量的数据访问框架(ORM)是关键。

- **垂直应用架构**

    当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，**将应用拆成互不相干的几个应用**，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键。

- **分布式服务架构**

    当垂直应用越来越多，应用之间交互不可避免，**将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心**，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。

- **流动计算架构**

    当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时**需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率**。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键。



## 二、Zookeeper 和 Dubbo

- **ZooKeeper**（注册中心）

    ZooKeeper 是一个分布式的，开放源码的**分布式应用程序协调服务**。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

- **Dubbo**

    Dubbo 是 Alibaba 开源的**分布式服务框架，主要解决各个服务之间远程调用问题**，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间解耦合（或者最大限度地松耦合）。从服务模型的角度来看，Dubbo 采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方（Provider）和服务消费方（Consumer）两个角色。

![image-20201114145232770](FrameDay06_14%20SpringBoot%E4%B8%8E%E5%88%86%E5%B8%83%E5%BC%8F.resource/image-20201114145232770.png)

### 项目搭建过程

- 步骤一：安装zookeeper作为注册中心

`docker pull registry.docker-cn.com/library/zookeeper `

启动：`docker run --name zk01 -p 2181:2181 --restart always -d 镜像ID`

- 步骤二：导入相关依赖，整合 Dubbo

    针对服务提供者和服务消费者：引入dubbo和zkclient相关依赖

    ```xml
    <!--引入 dubbo 依赖 -->
    <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>2.7.8</version>
    </dependency>
    <!--引入zookeeper的客户端工具，必须排除 Zookeeper 依赖-->
    <dependency>
        <groupId>com.github.sgroschupf</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.1</version>
        <exclusions>
            <exclusion>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>2.12.0</version>
    </dependency>
    
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>2.8.0</version>
    </dependency>
    ```

- 步骤三：编写服务提供者：即 `provider-ticket` 

    - 在配置文件中配置dubbo的扫描包和注册中心地址

        ```properties
        # 当前应用名称，见 pom.xml 中的 <name>provider-ticket</name>
        dubbo.application.name=provider-ticket
        # 应用发布的注册中心的地址
        dubbo.registry.address=zookeeper://192.168.238.151:2181
        # 哪个包下面的服务进行发布
        dubbo.scan.base-packages=com.gjxaiou.service
        ```

    - 编写一个 Service 以及对应的实现类同时使用 @Service发布服务
        在上面那个实现类上使用注解进行标注。即启动之后会按照属性配置包的位置进行扫描，然后将标有 `@Service` 的服务进行发布。注意，在 2.X 版本中该注解已经被标注了：`@Deprecated` 应该使用`@DubboService`

        ```java
        package com.gjxaiou.service;
        
        public interface TicketService {
            String buyTicket(String userName);
        }
        
        // -----------------------------------------------
        package com.gjxaiou.service.com.gjxaiou.service.impl;
        
        import com.gjxaiou.service.TicketService;
        import org.apache.dubbo.config.annotation.DubboService;
        import org.springframework.stereotype.Component;
        
        // 注入到容器中
        @Component
        //将服务发布出去
        @DubboService
        public class TicketServiceImpl implements TicketService {
            @Override
            public String buyTicket(String userName) {
                return userName + " buy a ticket";
            }
        }
        
        ```

- 步骤四：**消费服务步骤**：

    - 首先进行基本属性配置 	

        ```properties
        # 应用名称见：<name>consumer-user</name>
        dubbo.application.name=consumer-user
        
        dubbo.registry.address=zookeeper://192.168.238.151:2181
        ```

        同时将上面提供注册服务的基本包（`com.gjxaiou.service`）移动到服务消费项目的目录下。「注意」基础包名要完全一样，但是实现类不需要。

- 步骤五：消费使用步骤：

    消费的 `UserService` 类上标注的 `@Service` 是 Spring 的注解，然后里面使用了 `TicketService` 上面使用 `@DubboReference` 进行标注（因为注册和获取服务都是按照服务的全类名进行的）。

    ```java
    package com.gjxaiou.userService;
    
    import com.gjxaiou.service.TicketService;
    import org.apache.dubbo.config.annotation.DubboReference;
    import org.springframework.stereotype.Service;
    
    // 这个 @Service 是 Spring 的
    @Service
    public class UserService {
        @DubboReference
        TicketService ticketService;
    
        public void testMethod(String userName) {
            System.out.println("恭喜：" + ticketService.buyTicket(userName));
        }
    }
    ```

    

- 步骤六：**测试步骤**

    首先启动服务提供者的项目，然后运行服务消费者的项目，测试程序在测试类：`ConsumerUserApplicationTests`中。

    ```java
    package com.gjxaiou;
    
    import com.gjxaiou.userService.UserService;
    import org.junit.jupiter.api.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest
    class ConsumerUserApplicationTests {
        @Autowired
        UserService userService;
    
        @Test
        void contextLoads() {
            userService.testMethod("GJXAIOU");
        }
    }
    ```

    先启动 ProviderTicketApplication，然后启动测试类 ConsumerUserApplicationTests，结果如下：

    `恭喜：GJXAIOU buy a ticket`



#### 报错整理：

- `java.lang.NoClassDefFoundError: org/apache/curator/framework/CuratorFrameworkFactory`

    解决：缺少curator依赖

    ```xml
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
        <version>2.12.0</version>
    </dependency>
    ```

- 报错如下：

    ```java
    ***************************
    APPLICATION FAILED TO START
    ***************************
    
    Description:
    
    An attempt was made to call a method that does not exist. The attempt was made from the following location:
    
        org.apache.curator.utils.DefaultZookeeperFactory.newZooKeeper(DefaultZookeeperFactory.java:29)
    
    The following method did not exist:
    
        org.apache.zookeeper.ZooKeeper.<init>(Ljava/lang/String;ILorg/apache/zookeeper/Watcher;Z)V
    
    The method's class, org.apache.zookeeper.ZooKeeper, is available from the following locations:
    
        jar:file:/D:/javaHj/maven/maven-repository/org/apache/zookeeper/zookeeper/3.3.3/zookeeper-3.3.3.jar!/org/apache/zookeeper/ZooKeeper.class
    
    It was loaded from the following location:
    
        file:/D:/javaHj/maven/maven-repository/org/apache/zookeeper/zookeeper/3.3.3/zookeeper-3.3.3.jar
    ```

    解决方案：只需要在zkclient中去除zookeeper依赖

    ```xml
    <dependency>
        <groupId>com.github.sgroschupf</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.1</version>
        <exclusions>
            <exclusion>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    ```

- 报错：

    ```java
    java.lang.NoClassDefFoundError: org/apache/curator/framework/recipes/cache/TreeCacheListener
    	at java.lang.ClassLoader.defineClass1(Native Method) ~[na:1.8.0_221]
    	at java.lang.ClassLoader.defineClass(ClassLoader.java:763) ~[na:1.8.0_221]
    	at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:142) ~[na:1.8.0_221]
    	at java.net.URLClassLoader.defineClass(URLClassLoader.java:468) ~[na:1.8.0_221]
    ```

    解决：导入 curator-recipes jar 包

    ```xml
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>2.8.0</version>
    </dependency>
    ```

## 三、Spring Boot 和 Spring Cloud

Spring Cloud 是一个**分布式的整体解决方案**。Spring Cloud 为开发者提供了在分布式系统（配置管理，服务发现，熔断，路由，微代理，控制总线，一次性 Token ，全局琐，Leader 选举，分布式 Session，集群状态）中快速构建的工具，使用 Spring Cloud 的开发者可以快速的启动服务或构建应用、同时能够快速和云平台资源进行对接。



**SpringCloud分布式开发五大常用组件**

- 服务发现——Netflix Eureka

- 客服端负载均衡——Netflix Ribbon

- 断路器——Netflix Hystrix

- 服务网关——Netflix Zuul

- 分布式配置——Spring Cloud Config



### 微服务

![image-20201114145410569](FrameDay06_14%20SpringBoot%E4%B8%8E%E5%88%86%E5%B8%83%E5%BC%8F.resource/image-20201114145410569.png)

**[Martin ](https://martinfowler.com/)[Fowler](https://martinfowler.com/)** 微服务原文[https://](https://martinfowler.com/articles/microservices.html)[martinfowler.com/articles/microservices.html](https://martinfowler.com/articles/microservices.html)

### •Spring Cloud 入门

–1、创建provider

–2、创建consumer

–3、引入Spring Cloud

–4、引入Eureka注册中心

–5、引入Ribbon进行客户端负载均衡



- 步骤一：`provider-ticket`、`consumer-user`、`eureka-server` 三个模块中导入相关依赖

    ```xml
    <!--首先三个都需要导入 SpringCloud 依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Hoxton.SR9</version>
        <type>pom</type>
        <scope>runtime</scope>
    </dependency>
    
    <!-- 注册中心导入 Eureka-server，生产者和消费者导入 Eureka-client-->
    
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        <version>2.2.6.RELEASE</version>
    </dependency>
    ```

- 步骤二：配置注册中心 Eureka

    - 在配置文件 `application.yml` 中配置相关属性

        ```yaml
        server:
          port: 8761
        eureka:
          instance:
            hostname: eureka-server  # eureka实例的主机名
          client:
            register-with-eureka: false #不把自己注册到 eureka 上
            fetch-registry: false #不从eureka上来获取服务的注册信息
            service-url:
              defaultZone: http://localhost:8761/eureka/
        ```

    - 在主类上使用 `@EnableRurkaServer` 进行标注。

        ```java
        package com.gjxaiou;
        
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
        
        @EnableEurekaServer
        @SpringBootApplication
        public class EurekaServerApplication {
        
            public static void main(String[] args) {
                SpringApplication.run(EurekaServerApplication.class, args);
            }
        }
        ```

- 步骤三：配置服务提供者

    - 基本属性配置

        ```yaml
        server:
          port: 8001
        spring:
          application:
            # 见 pom.xml 文件中的 <name>consumer-user</name>
            name: provider-ticket
        
        eureka:
          instance:
            prefer-ip-address: true # 注册服务的时候使用服务的ip地址
          client:
            service-url:
              defaultZone: http://localhost:8761/eureka/
        ```

    - 因为 Spring Cloud 使用 轻量级的 HTTP 进行通信，所以服务提供者也需要提供一个 Controller 用于外部访问。Service 和 Controller 代码为：

        ```java
        package com.gjxaiou.service;
        
        import org.springframework.stereotype.Service;
        
        @Service
        public class TicketService {
            // 比较简单就不分接口和实现类了
            public String getTicket(String userName){
                System.out.println("服务提供者：8001");
                return "恭喜：" + userName + " 买了票";
            }
        }
        
        // -------------------------------------------
        
        package com.gjxaiou.controller;
        
        import com.gjxaiou.service.TicketService;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.stereotype.Controller;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        @RestController
        public class TicketController {
            @Autowired
            TicketService ticketService;
        
            @GetMapping("/ticket")
            public String buyTicket(String userName) {
                return ticketService.getTicket(userName);
            }
        }
        ```

    - 补充：同一个引用的两个实例同时运行

        将服务提供者右边的 Maven 中执行  「Lifecycle」->「package」进行打包，然后将上面的端口改为 8002，再进行 「package」打包，将两个 jar 包放在一个文件中，分别开启两个 cmd 窗口使用 `java -jar jar包名称`进行执行。这样在注册中心（使用 `localhost:8761`进行查看）就能看到多个服务实例了。

    

- 步骤四：创建消费者和消费

    首先还是 yaml 中进行配置

    ```yaml
    spring:
      application:
        name: consumer-user
    server:
      port: 8200
    
    eureka:
      instance:
        prefer-ip-address: true # 注册服务的时候使用服务的ip地址
      client:
        service-url:
          defaultZone: http://localhost:8761/eureka/
    
    ```

    同时在主类上使用 `@EnableDiscoveryClient`用于开启发现服务功能。

    - 编写一个 Controller 来访问生成者提供的服务

        ```java
        package com.gjxaiou.controller;
        
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        import org.springframework.web.client.RestTemplate;
        
        @RestController
        public class UserController {
        
            @Autowired
            RestTemplate restTemplate;
        
            @GetMapping("/buyticket")
            public String buyTicket(String userName) {
                // URL 为 服务名称/地址
                String s = restTemplate.getForObject("http://PROVIDER-TICKET/ticket", String.class);
                return userName + s;
            }
        }
        
        ```

        

    - 同时在主类中提供访问 HTTP 方式

        ```java
        package com.gjxaiou;
        
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
        import org.springframework.cloud.client.loadbalancer.LoadBalanced;
        import org.springframework.context.annotation.Bean;
        import org.springframework.web.client.RestTemplate;
        
        @EnableDiscoveryClient //开启发现服务功能
        @SpringBootApplication
        public class ConsumerUserApplication {
        
            public static void main(Strijavang[] args) {
                SpringApplication.run(ConsumerUserApplication.class, args);
            }
        
            // 用于发送 HTTP 请求
            @LoadBalanced //使用负载均衡机制
            @Bean
            public RestTemplate restTemplate() {
                return new RestTemplate();
            }
        }
        ```

#### 报错整理

- 项目模块进行打包的时候

    ```java
    Failed to execute goal org.apache.maven.plugins:maven-surefire-plugin:2.22.2:test (default-test) on project provider-ticket: There are test failures.
    
    Please refer to E:\Program\Java\Project\VideoClass\FramePromotion\SpringBoot\SpringBoot-06-SpringCloud\provider-ticket\target\surefire-reports for the individual test results.
    Please refer to dump files (if any exist) [date].dump, [date]-jvmRun[N].dump and [date].dumpstream.
    
    ```

    