# Spring Framework Overview

Version 5.3.13

Spring makes it easy to create Java enterprise applications. It provides everything you need to embrace the Java language in an enterprise environment, with support for Groovy and Kotlin as alternative languages on the JVM, and with the flexibility to create many kinds of architectures depending on an application’s needs. As of Spring Framework 5.1, Spring requires JDK 8+ (Java SE 8+) and provides out-of-the-box support for JDK 11 LTS. Java SE 8 update 60 is suggested as the minimum patch release for Java 8, but it is generally recommended to use a recent patch release.

Spring supports a wide range of application scenarios. In a large enterprise, applications often exist for a long time and have to run on a JDK and application server whose upgrade cycle is beyond developer control. Others may run as a single jar with the server embedded, possibly in a cloud environment. Yet others may be standalone applications (such as batch or integration workloads) that do not need a server.

Spring is open source. It has a large and active community that provides continuous feedback based on a diverse range of real-world use cases. This has helped Spring to successfully evolve over a very long time.

Spring 使创建 Java 企业应用程序变得容易。它提供了在企业环境中使用 Java 语言所需的一切，支持 Groovy 和 Kotlin 作为 JVM 上的替代语言，并具有根据应用程序需要创建多种体系结构的灵活性。**从 Spring Framework 5.1 开始，Spring 需要 JDK 8+（JavaSE8+）并为 JDK 11LTS 提供现成的支持。建议将 JavaSE8 Update60 作为 Java8 的最低补丁版本，但通常建议使用最新的补丁版本。**

Spring 支持广泛的应用场景。在大型企业中，应用程序通常存在很长时间，并且必须在 JDK 和应用程序服务器上运行，其升级周期超出了开发人员的控制。另一些可能作为嵌入服务器的单个 jar 运行，可能在云环境中。还有一些可能是不需要服务器的独立应用程序（如批处理或集成工作负载）。

Spring 是开源的。它有一个庞大而活跃的社区，根据各种各样的实际用例提供持续的反馈。这帮助 Spring 在很长一段时间内成功地进化。

## 1. What We Mean by "Spring"

The term "Spring" means different things in different contexts. It can be used to refer to the Spring Framework project itself, which is where it all started. Over time, other Spring projects have been built on top of the Spring Framework. Most often, when people say "Spring", they mean the entire family of projects. This reference documentation focuses on the foundation: the Spring Framework itself.

The Spring Framework is divided into modules. Applications can choose which modules they need. At the heart are the modules of the core container, including a configuration model and a dependency injection mechanism. Beyond that, the Spring Framework provides foundational support for different application architectures, including messaging, transactional data and persistence, and web. It also includes the Servlet-based Spring MVC web framework and, in parallel, the Spring WebFlux reactive web framework.

A note about modules: Spring’s framework jars allow for deployment to JDK 9’s module path ("Jigsaw"). For use in Jigsaw-enabled applications, the Spring Framework 5 jars come with "Automatic-Module-Name" manifest entries which define stable language-level module names ("spring.core", "spring.context", etc.) independent from jar artifact names (the jars follow the same naming pattern with "-" instead of ".", e.g. "spring-core" and "spring-context"). Of course, Spring’s framework jars keep working fine on the classpath on both JDK 8 and 9+.

「Spring」一词在不同的语境中意味着不同的事物。它可以用来指代 Spring 框架项目本身，它就是从这里开始的。随着时间的推移，**其他 Spring 项目已经建立在 Spring 框架之上**。通常，当人们说「Spring」时，他们指的是整个项目家族。本参考文档的重点是基础：Spring 框架本身。

Spring 框架分为多个模块。应用程序可以选择所需的模块。核心是核心容器的模块，包括配置模型和依赖项注入机制。除此之外，Spring 框架还为不同的应用程序体系结构提供了基础支持，包括消息传递、事务数据和持久性以及 web 应用程序。它还包括基于 Servlet 的 spring mvc web 框架，以及并行的 spring web flux 反应式 web 框架。

关于模块的注意事项：Spring 的框架 JAR 允许部署到 JDK9 的模块路径（“Jigsaw”）。为了在支持 Jigsaw 的应用程序中使用，Spring Framework 5 jar 附带了 “Automatic Module Name” 清单条目，这些条目定义了稳定的语言级模块名（“Spring.core”、“Spring.context”等），独立于 jar 工件名（jar 遵循相同的命名模式，用“-”代替“.”，e、 g.“spring核心”和“spring上下文”）。当然，Spring 的框架 JAR 在 JDK 8 和 9+ 上的类路径上都能正常工作。

## 2. History of Spring and the Spring Framework

Spring came into being in 2003 as a response to the complexity of the early [J2EE](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition) specifications. While some consider Java EE and Spring to be in competition, Spring is, in fact, complementary to Java EE. The Spring programming model does not embrace the Java EE platform specification; rather, it integrates with carefully selected individual specifications from the EE umbrella:

- Servlet API ([JSR 340](https://jcp.org/en/jsr/detail?id=340))
- WebSocket API ([JSR 356](https://www.jcp.org/en/jsr/detail?id=356))
- Concurrency Utilities ([JSR 236](https://www.jcp.org/en/jsr/detail?id=236))
- JSON Binding API ([JSR 367](https://jcp.org/en/jsr/detail?id=367))
- Bean Validation ([JSR 303](https://jcp.org/en/jsr/detail?id=303))
- JPA ([JSR 338](https://jcp.org/en/jsr/detail?id=338))
- JMS ([JSR 914](https://jcp.org/en/jsr/detail?id=914))
- as well as JTA/JCA setups for transaction coordination, if necessary.

The Spring Framework also supports the Dependency Injection ([JSR 330](https://www.jcp.org/en/jsr/detail?id=330)) and Common Annotations ([JSR 250](https://jcp.org/en/jsr/detail?id=250)) specifications, which application developers may choose to use instead of the Spring-specific mechanisms provided by the Spring Framework.

As of Spring Framework 5.0, Spring requires the Java EE 7 level (e.g. Servlet 3.1+, JPA 2.1+) as a minimum - while at the same time providing out-of-the-box integration with newer APIs at the Java EE 8 level (e.g. Servlet 4.0, JSON Binding API) when encountered at runtime. This keeps Spring fully compatible with e.g. Tomcat 8 and 9, WebSphere 9, and JBoss EAP 7.

Over time, the role of Java EE in application development has evolved. In the early days of Java EE and Spring, applications were created to be deployed to an application server. Today, with the help of Spring Boot, applications are created in a devops- and cloud-friendly way, with the Servlet container embedded and trivial to change. As of Spring Framework 5, a WebFlux application does not even use the Servlet API directly and can run on servers (such as Netty) that are not Servlet containers.

Spring continues to innovate and to evolve. Beyond the Spring Framework, there are other projects, such as Spring Boot, Spring Security, Spring Data, Spring Cloud, Spring Batch, among others. It’s important to remember that each project has its own source code repository, issue tracker, and release cadence. See [spring.io/projects](https://spring.io/projects) for the complete list of Spring projects.

Spring 于 2003 年诞生，作为对早期[J2EE](https://en.wikipedia.org/wiki/Java_Platform) 复杂性的响应企业版规范。虽然有些人认为 javaEE 和 Spring 将处于竞争状态，但实际上**，Spring 是 java EE 的补充。Spring 编程模型不包含JavaEE 平台规范；相反，它与精心挑选的 EE 保护伞中的单个规范集成：**

- Servlet API（[JSR340](https://jcp.org/en/jsr/detail?id=340))
- WebSocket API（[JSR356](https://www.jcp.org/en/jsr/detail?id=356))
- 并发实用程序（[JSR236](https://www.jcp.org/en/jsr/detail?id=236))
- JSON绑定API（[JSR367](https://jcp.org/en/jsr/detail?id=367))
- Bean验证（[JSR303](https://jcp.org/en/jsr/detail?id=303))
- JPA（[JSR338](https://jcp.org/en/jsr/detail?id=338))
- JMS（[JSR 914](https://jcp.org/en/jsr/detail?id=914))
- 以及用于事务协调的JTA/JCA设置（如有必要）。

Spring 框架还支持依赖项注入（[JSR330](https://www.jcp.org/en/jsr/detail?id=330))和常见注释（[JSR 250](https://jcp.org/en/jsr/detail?id=250))规范，**应用程序开发人员可以选择使用这些规范来代替 Spring 框架提供的特定于 Spring 的机制。**

从 Spring Framework 5.0 开始，Spring 至少需要 Java EE 7 级别（例如 Servlet 3.1+、JPA 2.1+），同时在运行时遇到 Java EE 8 级别的新 API（例如Servlet 4.0、JSON绑定API）时提供开箱即用的集成。这使 Spring 与 Tomcat 8 和 9、WebSphere 9 和 JBoss EAP 7 完全兼容。

随着时间的推移，javaee 在应用程序开发中的角色已经发生了变化。在 JavaEE 和 Spring 的早期，创建应用程序是为了部署到应用服务器。今天，在 SpringBoot 的帮助下，应用程序是以 devops 和云友好的方式创建的，嵌入了 Servlet 容器，可以进行简单的更改。**从 SpringFramework5 开始，WebFlux 应用程序甚至不直接使用 ServletAPI，可以在非 Servlet 容器的服务器（如 Netty）上运行。**

Spring 继续创新和发展。除了 Spring 框架之外，还有其他项目，如 Spring Boot、Spring Security、Spring Data、Spring Cloud、Spring Batch 等。重要的是要记住，每个项目都有自己的源代码存储库、问题跟踪器和发布 cadence。参见[spring.io/projects](https://spring.io/projects)查看Spring 项目的完整列表。

## 3. Design Philosophy

When you learn about a framework, it’s important to know not only what it does but what principles it follows. Here are the guiding principles of the Spring Framework:

- Provide choice at every level. Spring lets you defer design decisions as late as possible. For example, you can switch persistence providers through configuration without changing your code. The same is true for many other infrastructure concerns and integration with third-party APIs.
- Accommodate diverse perspectives. Spring embraces flexibility and is not opinionated about how things should be done. It supports a wide range of application needs with different perspectives.
- Maintain strong backward compatibility. Spring’s evolution has been carefully managed to force few breaking changes between versions. Spring supports a carefully chosen range of JDK versions and third-party libraries to facilitate maintenance of applications and libraries that depend on Spring.
- Care about API design. The Spring team puts a lot of thought and time into making APIs that are intuitive and that hold up across many versions and many years.
- Set high standards for code quality. The Spring Framework puts a strong emphasis on meaningful, current, and accurate javadoc. It is one of very few projects that can claim clean code structure with no circular dependencies between packages.

**当您了解一个框架时，重要的是不仅要知道它做什么，还要知道它遵循什么原则。以下是 Spring 框架的指导原则**：

- 在各个级别提供选择。Spring 允许您尽可能推迟设计决策。例如，您可以通过配置切换持久化提供者，而无需更改代码。许多其他基础设施问题以及与第三方 API 的集成也是如此。

- 适应不同的观点。Spring 具有灵活性，对于事情应该如何做并不固执己见。它以不同的视角支持广泛的应用程序需求。

- 保持强大的向后兼容性。Spring 的发展经过了精心管理，在版本之间几乎没有突破性的变化。Spring 支持一系列精心挑选的JDK 版本和第三方库，以便于维护依赖 Spring 的应用程序和库。

- 关心 API 设计。Spring 团队投入了大量的精力和时间来制作直观的 API，这些 API 可以跨多个版本和多年使用。

- 为代码质量设定高标准。Spring 框架非常强调有意义、最新和准确的 javadoc。它是极少数几个可以声称代码结构干净、包之间没有循环依赖关系的项目之一。

## 4. Feedback and Contributions

For how-to questions or diagnosing or debugging issues, we suggest using Stack Overflow. Click [here](https://stackoverflow.com/questions/tagged/spring+or+spring-mvc+or+spring-aop+or+spring-jdbc+or+spring-r2dbc+or+spring-transactions+or+spring-annotations+or+spring-jms+or+spring-el+or+spring-test+or+spring+or+spring-remoting+or+spring-orm+or+spring-jmx+or+spring-cache+or+spring-webflux+or+spring-rsocket?tab=Newest) for a list of the suggested tags to use on Stack Overflow. If you’re fairly certain that there is a problem in the Spring Framework or would like to suggest a feature, please use the [GitHub Issues](https://github.com/spring-projects/spring-framework/issues).

If you have a solution in mind or a suggested fix, you can submit a pull request on [Github](https://github.com/spring-projects/spring-framework). However, please keep in mind that, for all but the most trivial issues, we expect a ticket to be filed in the issue tracker, where discussions take place and leave a record for future reference.

For more details see the guidelines at the [CONTRIBUTING](https://github.com/spring-projects/spring-framework/tree/main/CONTRIBUTING.md), top-level project page.

对于操作问题或诊断或调试问题，我们建议使用 Stack Overflow。点击[here](https://stackoverflow.com/questions/tagged/spring+or+spring-mvc+or+spring-aop+or+spring-jdbc+or+spring-r2dbc+or+spring-transactions+or+spring-annotations+or+spring-jms+or+spring-el+or+spring-test+or+spring+or+spring-remoting+or+spring-orm+or+spring-jmx+or+spring-cache+or+spring-webflux+or+spring-rsocket?tab=Newest)以获得建议用于 stack overflow 的标记列表。如果您相当确定 Spring 框架中存在问题，或者希望推荐一个功能，请使用[GitHub Issues](https://github.com/spring-projects/spring-framework/issues).

如果您考虑了解决方案或建议的修复方案，您可以在[Github](https://github.com/spring-projects/spring-framework)上提交拉取请求。但是，请记住，对于除最琐碎的问题外的所有问题，我们希望在 issue 追踪器中提交一份记录单，在那里进行讨论，并留下记录以供将来参考。

有关更多详细信息，请参见项目页面的最上面的  [CONTRIBUTING](https://github.com/spring-projects/spring-framework/tree/main/CONTRIBUTING.md) 指导。

## 5. Getting Started

If you are just getting started with Spring, you may want to begin using the Spring Framework by creating a [Spring Boot](https://projects.spring.io/spring-boot/)-based application. Spring Boot provides a quick (and opinionated) way to create a production-ready Spring-based application. It is based on the Spring Framework, favors convention over configuration, and is designed to get you up and running as quickly as possible.

You can use [start.spring.io](https://start.spring.io/) to generate a basic project or follow one of the ["Getting Started" guides](https://spring.io/guides), such as [Getting Started Building a RESTful Web Service](https://spring.io/guides/gs/rest-service/). As well as being easier to digest, these guides are very task focused, and most of them are based on Spring Boot. They also cover other projects from the Spring portfolio that you might want to consider when solving a particular problem.

如果您刚刚开始使用 Spring，您可能希望通过创建一个基于**[Spring Boot](https://projects.spring.io/spring-boot/)**的应用程序来开始使用 Spring 框架。SpringBoot 提供了一种快速（且自以为是）的方法来创建一个基于 Spring 的应用程序。它**基于 Spring 框架，支持约定而非配置**，旨在让您尽快启动并运行。

您可以使用[start.spring.io](https://start.spring.io/)生成基本项目或遵循[“入门”指南](https://spring.io/guides)，例如[开始构建RESTful Web服务](https://spring.io/guides/gs/rest-service/).这些指南不仅易于理解，而且非常注重任务，而且大多数都基于Spring Boot。它们还涵盖了在解决特定问题时可能需要考虑的Spring文件夹中的其他项目。

Version 5.3.13
Last updated 2021-11-11 07:31:24 UTC