# Spring Cloud Config 3.1.0

Spring Cloud Config provides server and client-side support for externalized configuration in a distributed system. With the Config Server you have a central place to manage external properties for applications across all environments. The concepts on both client and server map identically to the Spring `Environment` and `PropertySource` abstractions, so they fit very well with Spring applications, but can be used with any application running in any language. As an application moves through the deployment pipeline from dev to test and into production you can manage the configuration between those environments and be certain that applications have everything they need to run when they migrate. The default implementation of the server storage backend uses git so it easily supports labelled versions of configuration environments, as well as being accessible to a wide range of tooling for managing the content. It is easy to add alternative implementations and plug them in with Spring configuration.

Spring Cloud Config 为分布式系统中的外部化配置提供服务器端和客户端支持。使用配置服务器，您可以在中心位置管理所有环境中应用程序的外部属性。客户机和服务器上的概念与 Spring 的  `Environment` 和 `PropertySource` 抽象完全相同，因此它们非常适合 Spring 应用程序，但可以用于以任何语言运行的任何应用程序。当应用程序在部署管道中从开发人员移动到测试人员并进入生产环境时，您可以管理这些环境之间的配置，并确保应用程序在迁移时拥有运行所需的一切。**服务器存储后端的默认实现使用 git**，因此它可以轻松地支持配置环境的标记版本，并且可以访问用于管理内容的各种工具。添加替代实现并使用 Spring 配置插入它们是很容易的。

## Features

Spring Cloud Config Server features:

- HTTP, resource-based API for external configuration (name-value pairs, or equivalent YAML content)
- Encrypt and decrypt property values (symmetric or asymmetric)
- Embeddable easily in a Spring Boot application using `@EnableConfigServer`

Config Client features (for Spring applications):

- Bind to the Config Server and initialize Spring `Environment` with remote property sources
- Encrypt and decrypt property values (symmetric or asymmetric)

Spring Cloud Config 服务端功能：

- HTTP，用于外部配置的基于资源的 API（名称-值对或等效的 YAML 内容）
- 加密和解密属性值（对称或非对称）
- 使用`@EnableConfigServer` 可轻松嵌入 Spring 启动应用程序

Config 客户端功能（用于 Spring 应用程序）：

- 绑定到配置服务器并使用远程属性源初始化 Spring`Environment`

- 加密和解密属性值（对称或非对称）

## Getting Started

As long as Spring Boot Actuator and Spring Config Client are on the classpath any Spring Boot application will try to contact a config server on `http://localhost:8888`, the default value of `spring.cloud.config.uri`. If you would like to change this default, you can set `spring.cloud.config.uri` in `bootstrap.[yml | properties]` or via system properties or environment variables.

```java
@Configuration
@EnableAutoConfiguration
@RestController
public class Application {

  @Value("${config.name}")
  String name = "World";

  @RequestMapping("/")
  public String home() {
    return "Hello " + name;
  }

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```

The value of `config.name` in the sample (or any other values you bind to in the normal Spring Boot way) can come from local configuration or from the remote Config Server. The Config Server will take precedence by default. To see this look at the `/env` endpoint in the application and see the `configServer` property sources.

To run your own server use the `spring-cloud-config-server` dependency and `@EnableConfigServer`. If you set `spring.config.name=configserver` the app will run on port 8888 and serve data from a sample repository. You need a `spring.cloud.config.server.git.uri` to locate the configuration data for your own needs (by default it is the location of a git repository, and can be a local `file:..` URL).

示例中的 `config.name` 值（或以正常 Spring 引导方式绑定到的任何其他值）可以来自本地配置或远程配置服务器。默认情况下，配置服务器将优先。要查看此内容，请查看应用程序中的`/env`端点，并查看`configServer`属性源。

要运行自己的服务器，请使用 `spring-cloud-config-server` 依赖项和 `@EnableConfigServer`。如果设置 `spring.config.name=configserver`，应用程序将在端口 8888 上运行，并提供来自示例存储库的数据。您需要一个 `spring.cloud.config.server.git.uri` 来根据自己的需要定位配置数据（默认情况下，它是 git 存储库的位置，可以是本地的  `file:..` URL）。

## Quickstart Your Project

Bootstrap your application with [Spring Initializr](https://start.spring.io/).