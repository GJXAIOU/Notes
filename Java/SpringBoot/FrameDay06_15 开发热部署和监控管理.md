# 七、Spring Boot与开发热部署

在开发中我们修改一个 Java 文件后想看到效果不得不重启应用，这导致大量时间花费，我们希望不重启应用的情况下，程序可以自动部署（热部署）。有以下四种情况，如何能实现热部署。



- 方式一：模板引擎

    - 在 Spring Boot 中开发情况下禁用模板引擎的 cache

    - 页面模板改变 ctrl+F9 可以重新编译当前页面并生效

- 方式二：Spring Loaded

    Spring官方提供的热部署程序，实现修改类文件的热部署

    - 下载Spring Loaded（项目地址https://github.com/spring-projects/spring-loaded）
    - 添加运行时参数；`javaagent:C:/springloaded-1.2.5.RELEASE.jar –noverify`

- 方式三：JRebel

    收费的一个热部署软件，安装插件使用即可

- 方式四：Spring Boot Devtools（推荐）

    - 步骤一：引入依赖

        ```xml
        <dependency>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-devtools</artifactId>   
        </dependency> 
        ```

    - 步骤二：IDEA 使用 ctrl+F9 进行重新编译

    

    

# 八、Spring Boot与监控管理

## 一、监控管理

通过引入 spring-boot-starter-actuator，可以使用Spring Boot为我们提供的**准生产环境下的应用监控和管理功能**。我们可以通过 HTTP，JMX，SSH 协议来进行操作，自动得到审计、健康及指标信息等。

使用监控步骤

- 步骤一：引入spring-boot-starter-actuator 依赖

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```

- 为了可以有权限访问项目中的数据，需要在配置文件 `application.properties` 中进行如下配置

    ```properties
    management.security.enabled=false
    ```

- 通过 http 方式访问监控端点

    ![image-20201114200356392](FrameDay06_15%20%E5%BC%80%E5%8F%91%E7%83%AD%E9%83%A8%E7%BD%B2%E5%92%8C%E7%9B%91%E6%8E%A7%E7%AE%A1%E7%90%86.resource/image-20201114200356392.png)

     启动项目之后，根据上面截图显示的项目启动中的配置，然后在浏览器中通过 `http://localhost:8080/beans`（示例）就可以访问。2.0 之后使用 `http://localhost:8080/actuator/beans`

    示例：`http://localhost:8080/health`结果为：

    ```json
    {
        "status": "DOWN",
        "myApp": {
            "status": "DOWN",
            "msg": "服务异常"
        },
        "diskSpace": {
            "status": "UP",
            "total": 1000203087872,
            "free": 335452856320,
            "threshold": 10485760
        },
        "redis": {
            "status": "UP",
            "version": "6.0.9"
        }
    }
    ```

- 监控和管理端点 （可以在项目启动的控制台中看到可用的）

| **端点名**  | **描述**                    |
| ----------- | --------------------------- |
| autoconfig  | 所有自动配置信息            |
| auditevents | 审计事件                    |
| beans       | 所有Bean的信息              |
| configprops | 所有配置属性                |
| dump        | 线程状态信息                |
| env         | 当前环境信息                |
| health      | 应用健康状况                |
| info        | 当前应用信息                |
| metrics     | 应用的各项指标              |
| mappings    | 应用@RequestMapping映射路径 |
| shutdown    | 关闭当前应用（默认关闭）    |
| trace       | 追踪信息（最新的http请求）  |

## 二、定制端点信息

定制端点一般在 `application.properties` 中通过 `endpoints + 端点名 + 属性名` 来设置。常见设置样式如下：

- 修改端点 id：`endpoints.beans.id = mybeans`（endpoints.beans.id=mybeans）

    就是以前通过 	`localhost:8080/beans`，现在通过 `localhost:8080/mybeans`		

- 开启远程应用关闭功能：`endpoints.shutdown.enabled = true`

    远程关闭应用，需要发送 Post 请求（可以通过 postman）。在 Postman 中发送 PUT 请求：	`http://localhost:8080/shutdown`	，就可以远程关闭应用程序。请求返回结果为：

    ```json
    {
        "message": "Shutting down, bye..."
    }
    ```

- 关闭端点远程查看权限：`endpoints.beans.enabled = false`

- 关闭其他所有端点开启所需端点

    ```properties
    endpoints.enabled = false
    endpoints.beans.enabled = true
    ```

- 定制端点访问根路径（可以结合 spring security 对某个路径进行权限认证）

    `management.context-path=/manage` 修改访问根路径为：`/manage`，则原来的 `http://localhost:8080/mybean` 路径修改为：`http://localhost:8080/manage/mybean`。

- 关闭 http 端点

    `management.port = -1`

## 三、自定义健康状态检测

Spring Boot 的 `org.springframework.boot:spring-boot-actuator:1.5.12.RELEASE` Jar 包提供了一系列的健康检测组件。**他们在 pom.xml 文件中有了对应的 starter 之后就会生效**。

![image-20201114202705754](FrameDay06_15%20%E5%BC%80%E5%8F%91%E7%83%AD%E9%83%A8%E7%BD%B2%E5%92%8C%E7%9B%91%E6%8E%A7%E7%AE%A1%E7%90%86.resource/image-20201114202705754.png)

例如导入 Redis 的 starter 之后，然后在 `application.properties`中配置一个错误的 Redis 主机地址，然后访问`locahost:8080/XXX/health` 就可以看到错误。本质上用的就是上面的  `RedisHealthIndicator`。如果导入了 Redis 依赖，同时在属性文件中配置：`spring.redis.host=192.168.238.151`。就可以查看 Redis 的状态了。

![image-20201114222445921](FrameDay06_15%20%E5%BC%80%E5%8F%91%E7%83%AD%E9%83%A8%E7%BD%B2%E5%92%8C%E7%9B%91%E6%8E%A7%E7%AE%A1%E7%90%86.resource/image-20201114222445921.png)





**可以自定义健康指示器，即自定义检测哪些状态**

- 步骤一：自定义一个指示器，实现 HealthIndicator 接口
- 步骤二：指示器名称必须为 XXXHealthIndicator
- 步骤三：将指示器加入容器中

```java
package com.atguigu.springboot08actuator.health;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class MyAppHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {

        // 自定义的检查方法,不同状态返回下面两种不同结果
        // return Health.up().build() 代表健康
        return Health.down().withDetail("msg","服务异常").build();
    }
}

```

