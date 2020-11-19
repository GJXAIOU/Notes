# 概述
### 是什么？
Spring Cloud Ribbon 是基于 Netflix Ribbon 实现的一套==客户端负载均衡的工具==。
简单的说，Ribbonn是Netflix的开源项目，主要功能 是提供==客户端的软件负载均衡算法和服务调用。==Ribbon客户端组件提供一系列完善的配置项，如连接超时，重试等。就是在配置文件中列出 Loa Balancer后面所有机器，Ribbon会自动帮助你基于某种规则 (如简单轮询，随机连接等)去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。
### 官网资料，停更
https://github.com/Netflix/ribbon
### 能干什么？
1. 负载均衡
    * 负载均衡(Load Balance)是什么
        将用户的请求平摊的分配到多个服务上，从而达到HA(高可用)，常见的负载均衡有 Nginx,LVS,硬件 F5等。
    * Ribbon 本地负载均衡客户端 VS Nginx 服务端负载均衡
        Nginx 是服务器 负载均衡，客户端所有请求都会交给 nginx，然后由 nginx实现请求转发。即负载均衡是由服务端实现的。
        Ribbon 是本地负载均衡，在微服务调用接口时，在注册中心上获取注册信息服务列表 之后缓存在JVM本地，从而实现本地RPC远程服务调用技术。
2. 实现
负载均衡+RestTemplate 调用

<img src="imgs/Ribbon.png">

* Ribbon工作时有两步
    1. 第一步先选择 EurekaServer，优先选择统一区域负载较少的 server
    2. 第二部再根据用户指定的策略，从server取到的服务注册列表中选择一个地址。其中 Riibon 提供了多种策略（轮询，随机，根据响应时间加权）。

### 引入依赖
不需要
spring-cloud-starter-netflix-eureka-client 已经引入了 Ribbon-Balance的依赖

# RestTemplate 使用
1. getForObject 返回json
2. getForEntity 返回ResponseEnity对象，包括响应头，响应体等信息。
3. postForObject
与 get 方法一样，不同的是传进去的参数是对象
4. postForEntity
5. GET 请求方法
6. POST请求方法

# Ribbon 自带的负载均衡
### 核心组件 IRule
###### IRule默认自带的负载规则
1. RoundRobinRule   轮询
2. RandomRule   随机
3. RetryRule    先按照RoundRobinRule的 策略获取服务，如果获取服务失败则在指定时间里进行重试，获取可用服务
4. WeightedResponseTimeRule 对RoundRobinRule的扩展，响应速度越快，实例选择权重越大 ，越容易被选择
5. BestAvailableRule    会先过滤掉由于多次访问故障而处于断路器 跳闸状态的服务，然后选择一个并发一个最小的服务
6. BestAvaibilityFilteringRule  先过滤掉故障实例，再选择并发量较小的实例
7. ZoneAvoidanceRule    默认规则，符合server所在区域的性能和server的可用性选择服务器
###### 如何替换
1. 注意：IRule配置类不能放在@ComponentSan 的包及子包下，因为默认的扫描会变成全局负载均衡都按照这样的规则。
2. 新建包 com.wxh.myRule
3. 新建类 
    ```java
    public class MySelfRule {
        @Bean
        public IRule myRule(){
            return new RandomRule();//定义为随机
        }
    }
    ```
4. 主类添加注解
```java
// 选择要接收的服务和配置类
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
```
### 默认负载均衡轮回算法原理
###### 负载均衡算法
rest 接口 第几次请求数 % 服务器集群=实际调用服务器位置下标，每次服务重启后rest接口计数从1开始

总台数：2台

请求数  调用下标
1       1%2=1       
2       2%2=0
3       3%2=1
4       4%2=0
######  RoundRobinRule源码分析
跳过41集
###### 手写轮回算法
跳过42集