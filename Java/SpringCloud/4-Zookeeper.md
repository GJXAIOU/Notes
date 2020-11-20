# Zookeeper 实现注册注册发现

Zookeeper 是一个分布式协调工具，可以实现注册中心功能（即可以取代 Eureka 功能）

## 一、配置虚拟机和 zookeeper

首先关闭 Linux 的防火墙：`systemctl stop firewalld` 然后可以使用 `systemctl status firewalld` 进行确认。

如果本机和虚拟机之间的网络连接方式为：NAT，则注意实际上连接

1. 虚拟机终端输入ifconfig查看 ens33 下的端口号
2. 查看主机与虚拟机之间通信是否畅通
    1. 虚拟机端口号：ifconfig查看 ens33 下的端口号
    2. 主机端口号：网络连接下的 VMnet8 的端口号
    3. 使用虚拟机ping主机，使用主机ping虚拟机确保都可以ping通
## 二、新建模块：服务提供者 cloud-provider-payment8004
- 步骤一：pom.xml  文件

1. 排除zookeeper-discovery中自带的 zookeeper，同时引入与linux相同版本的 zookeeper

2. 排除引入 zookeeper 的日志，因为日志会会冲突

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <artifactId>SpringCloud2020</artifactId>
            <groupId>com.gjxaiou.springcloud</groupId>
            <version>1.0-SNAPSHOT</version>
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>cloud-provider-payment8004</artifactId>
    
    
        <dependencies>
            <dependency><!-- 引用自己定义的api通用包，可以使用Payment支付Entity -->
                <groupId>com.gjxaiou.springcloud</groupId>
                <artifactId>cloud-api-commons</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <!--监控-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
            <!--SpringBoot整合Zookeeper客户端-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
                <exclusions>
                    <!--先排除自带的zookeeper3.5.3-->
                    <exclusion>
                        <groupId>org.apache.zookeeper</groupId>
                        <artifactId>zookeeper</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
            <!--添加zookeeper3.4.14版本-->
            <dependency>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
                <version>3.4.14</version>
            </dependency>
            <!--热部署-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <scope>runtime</scope>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
    
    </project>
    ```

- 步骤二：yml 文件

    ```yaml
    # 8004 表示注册到 Zookeeper 服务器的支付服务提供者的端口号
    server:
      port: 8004
    
    spring:
      application:
        # 服务别名
        name: cloud-provider-payment
      cloud:
        zookeeper:
          # ip地址为linux中的网络接口，2181为zookeeper的默认端口
          connect-string: 192.168.238.151:2181
    ```

- 步骤三：Controller 类和主类

    ```java
    package com.gjxaiou.springcloud.controller;
    
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.util.UUID;
    
    @RestController
    @Slf4j
    public class PaymentController {
    
        @Value("${server.port}")
        private String serverPort;
    
        @GetMapping(value = "/payment/zk")
        public String paymentZk() {
            return "Spring Cloud with Zookeeper: " + serverPort + "\t" + UUID.randomUUID().toString();
        }
    }
    ```

    主类需要标注：`@EnableDiscoveryClient`

    ```java
    package com.gjxaiou.springcloud;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    
    @SpringBootApplication
    @EnableDiscoveryClient
    public class PaymentMain8004 {
        public static void main(String[] args) {
            SpringApplication.run(PaymentMain8004.class, args);
        }
    }
    ```

- 然后启动 Zookeeper

### 测试1：系统是否正常

- 从 Linux 层面：能看到服务名即为配置成功

    ```shell
    [zk: localhost:2181(CONNECTED) 3] ls /services
    [cloud-provider-payment]
    ```

- 网址测试：`http://localhost:8004/payment/zk` 可以看到一行 JSON 消息。

### 测试2：Zookeeper 属性查看
- 查看容器是否启动：`docker ps`
- 进入容器：`  docker exec -it zookeeper实例标识 /bin/bash`
-  进入bin目录：`cd bin`
-  登录 server：` zkCli.sh -server 127.0.0.1:2181`
- 查看目录： `ls /`


```shell
# 获取服务名
[zk: localhost:2181(CONNECTED) 0] ls /services
[cloud-provider-payment]
# 获取流水号
[zk: localhost:2181(CONNECTED) 1] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
# 获取详细信息
[zk: localhost:2181(CONNECTED) 2] get /services/cloud-provider-payment/efc76371-522d-4d5d-8f56-f8fe4deb7a47
{"name":"cloud-provider-payment","id":"efc76371-522d-4d5d-8f56-f8fe4deb7a47","address":"WINDOWS-N0GUAG7","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1590232919360,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
```
### 服务节点是临时的
关闭 8004 后在linux终端中,一段时间后失去连接
```
[zk: localhost:2181(CONNECTED) 18] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
[zk: localhost:2181(CONNECTED) 19] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
[zk: localhost:2181(CONNECTED) 20] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
[zk: localhost:2181(CONNECTED) 21] ls /services/cloud-provider-payment
[]
[zk: localhost:2181(CONNECTED) 22] 
```
再开启8004，再次查看流水号可以发现流水号跟之前的不一样 ，所以服务节点是临时的，在关闭服务后完全删除。
## 三、消费者模块：订单服务 cloud-consumerzk-order80 入住 zookeeper

- 步骤一：pom.xml 文件

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <artifactId>SpringCloud2020</artifactId>
            <groupId>com.gjxaiou.springcloud</groupId>
            <version>1.0-SNAPSHOT</version>
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>cloud-consumerzk-order80</artifactId>
    
        <dependencies>
            <dependency><!-- 引用自己定义的api通用包，可以使用Payment支付Entity -->
                <groupId>com.gjxaiou.springcloud</groupId>
                <artifactId>cloud-api-commons</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>
            <!--监控-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-actuator</artifactId>
            </dependency>
            <!--SpringBoot整合Zookeeper客户端-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
                <exclusions>
                    <!--先排除自带的zookeeper3.5.3-->
                    <exclusion>
                        <groupId>org.apache.zookeeper</groupId>
                        <artifactId>zookeeper</artifactId>
                    </exclusion>
                </exclusions>
            </dependency>
            <!--添加zookeeper3.4.14版本-->
            <dependency>
                <groupId>org.apache.zookeeper</groupId>
                <artifactId>zookeeper</artifactId>
                <version>3.4.14</version>
            </dependency>
            <!--热部署-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-devtools</artifactId>
                <scope>runtime</scope>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <optional>true</optional>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </project>
    ```

- 步骤二：新建并配置 `application.yml`

    ```yaml
    server:
      port: 80
    spring:
      application:
        name: cloud-consumerzk-order80
      cloud:
        zookeeper:
          connect-string: 192.168.238.151:2181
    ```

- 步骤三：写主类

    ```java
    package com.gjxaiou.springcloud;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    
    @SpringBootApplication
    @EnableDiscoveryClient
    public class OrderZKMain80{
        public static void main(String[] args) {
            SpringApplication.run(OrderZKMain80.class, args);
        }
    }
    ```

- 步骤四：自定义配置类生成 RestTemplate

    ```java
    package com.gjxaiou.springcloud.config;
    
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.client.RestTemplate;
    
    @Configuration
    public class ApplicationContextConfig {
        @Bean
        public RestTemplate getRestTemplate() {
            return new RestTemplate();
        }
    }
    ```

- 步骤五：Controller 调用 8004 服务

    ```java
    package com.gjxaiou.springcloud.controller;
    
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.client.RestTemplate;
    
    import javax.annotation.Resource;
    
    @RestController
    @Slf4j
    public class OrderZKController {
        public static final String INVOKE_URL = "http://cloud-provider-payment";
    
        @Resource
        private RestTemplate restTemplate;
    
        @GetMapping(value = "/consumer/payment/zk")
        public String paymentInfo(){
            String result = restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
            return  result;
        }
    }
    ```

#### 测试

7. linux 输入，查看节点是否注册上
```
[zk: localhost:2181(CONNECTED) 1] ls /services
[cloud-provider-payment, cloud-consumerzk-order80]
```
2.网址登陆查看
http://localhost:8004/payment/zk
http://localhost/consumer/payment/zk

