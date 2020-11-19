# 概述
1. 分布式系统面临的配置问题
每建一个微服务都需要一次配置，例如10个微服务访问相同的数据库，如果数据库名更改了，要改十次。
2. 是什么
SpringCloud Config 为微服务架构中的微服务提供几种化的外部配置支持，将不同微服务应用提供一个中心化外部配置。
3. 怎么用
服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器 并为客户端提供配置信息。加密解密信息接口。
客户端则是通过指定的配置中心来管理应用资源。并在启动的时候从配置中心获取和加载配置信息配置服务器默认采用git来存储配置信息。这样有助于对环境配置进行版本管理，并且可以通过git客户端方便管理和配置服务内容。
将配置信息以REST接口的形式暴露。通过 post curl 刷新
4. 与 github整合
# 服务端配置与整合
### github
1. 新建仓库springcloud-config
2. 获取新建的地址git@github.com:OT-mt/springcloud-config.git
3. 本地硬盘目录新建 git仓库并clone
### 建模块
1. pom
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
2. yml
```yml
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  cloud:
    config:
      server:
        git:
          #uri: git@github.com:EiletXie/config-repo.git #Github上的git仓库名字
          uri: https://github.com/OT-mt/springcloud-config.git
          ##搜索目录.这个目录指的是github上的目录
          search-paths:
            - springcloud-config
      ##读取分支
      label: master

eureka:
  client:
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/

```
3. 主启动
```java
@SpringBootApplication
@RestController
@EnableConfigServer
```
4. 测试
http://localhost:3344/springcloud-config/blob/master/config-prod.yml
# 客户端配置与测试
1. 建 mouble 
cloud-config-client3355
2. pom
```xml
<!-- 注意与上述不同 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```
3. yml
```yml
```
4. 主启动
# 客户端动态刷新
1. 避免每次更新配置都要更新客户端
2. 步骤
  1. 添加 actuator 依赖
  2. 修改yml暴露端口
  3. @RefreshScope业务类controller修饰
  4. 刷新
    ```
    curl -X POST "http://localhost:3355/actuator/refresh"
    ```
  5. 测试