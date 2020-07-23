---
tags: 朕已阅
---

# Spring详解（一）------概述

[原文地址](https://www.cnblogs.com/ysocean/p/7466191.html)

## 1、什么是 Spring ?

　　Spring是一个开源框架，Spring是于2003 年兴起的一个轻量级的Java 开发框架，由Rod Johnson 在其著作Expert One-On-One J2EE Development and Design中阐述的部分理念和原型衍生而来。它是为了解决企业应用开发的复杂性而创建的。**框架的主要优势之一就是其分层架构，分层架构允许使用者选择使用哪一个组件，同时为 J2EE 应用程序开发提供集成的框架**。Spring使用基本的JavaBean来完成以前只可能由EJB完成的事情。然而，Spring的用途不仅限于服务器端的开发。从简单性、可测试性和松耦合的角度而言，任何Java应用都可以从Spring中受益。Spring的核心是控制反转（IoC）和面向切面（AOP）。

　　**简单来说，Spring是一个分层的JavaSE/EE full-stack(一站式) 轻量级开源框架。**



## 3、Spring 特点

　　**①、方便解耦，简化开发**
　　通过Spring提供的IoC容器，我们可以**将对象之间的依赖关系交由Spring进行控制**，避免硬编码所造成的过度程序耦合。有了Spring，用户不必再为单实例模式类、属性文件解析等这些很底层的需求编写代码，可以更专注于上层的应用。

　　**②、AOP编程的支持**
　　通过Spring提供的AOP功能，方便进行面向切面的编程，许多不容易用传统OOP实现的功能可以通过AOP轻松应付。

　　**③、声明式事务的支持**
　　在Spring中，我们可以从单调烦闷的事务管理代码中解脱出来，**通过声明式方式灵活地进行事务的管理**，提高开发效率和质量。

　　**④、方便程序的测试**
　　可以用非容器依赖的编程方式进行几乎所有的测试工作，在Spring里，测试不再是昂贵的操作，而是随手可做的事情。例如：Spring对Junit4支持，可以通过注解方便的测试Spring程序。

　　**⑤、方便集成各种优秀框架**
　　Spring不排斥各种优秀的开源框架，相反，Spring可以降低各种框架的使用难度，Spring提供了对各种优秀框架（如Struts,Hibernate、Hessian、Quartz）等的直接支持。

　　**⑥、降低Java EE API的使用难度**
　　Spring对很多难用的Java EE API（如JDBC，JavaMail，远程调用等）提供了一个薄薄的封装层，通过Spring的简易封装，这些Java EE API的使用难度大为降低。

　　**⑦、Java 源码是经典学习范例**
　　Spring的源码设计精妙、结构清晰、匠心独运，处处体现着大师对Java设计模式灵活运用以及对Java技术的高深造诣。Spring框架源码无疑是Java技术的最佳实践范例。如果想在短时间内迅速提高自己的Java技术水平和应用开发水平，学习和研究Spring源码将会使你收到意想不到的效果。


## 4、Spring 框架结构

　　![](https://images2017.cnblogs.com/blog/1120165/201709/1120165-20170902113444874-1912798255.png)

 　　1、核心容器：核心容器提供 Spring 框架的基本功能(Spring Core)。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。**BeanFactory 使用控制反转（IOC） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开**。

　　2、Spring 上下文：Spring 上下文是一个配置文件，向 Spring框架提供上下文信息。Spring 上下文包括企业服务，例如JNDI、EJB、电子邮件、国际化、校验和调度功能。

　　3、Spring AOP：通过配置管理特性，Spring AOP 模块直接将面向切面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

　　4、Spring DAO：JDBCDAO抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。

　　5、Spring ORM：Spring 框架插入了若干个ORM框架，从而提供了 ORM 的对象关系工具，其中包括JDO、Hibernate和iBatisSQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。

　　6、Spring Web 模块：Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

　　7、Spring MVC 框架：MVC框架是一个全功能的构建 Web应用程序的 MVC 实现。通过策略接口，MVC框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。模型由javabean构成，存放于Map；视图是一个接口，负责显示模型；控制器表示逻辑代码，是Controller的实现。Spring框架的功能可以用在任何J2EE服务器中，大多数功能也适用于不受管理的环境。Spring 的核心要点是：支持不绑定到特定 J2EE服务的可重用业务和数据访问对象。毫无疑问，这样的对象可以在不同J2EE 环境（Web 或EJB）、独立应用程序、测试环境之间重用。


## 5、Spring 框架特征 

　　轻量——从大小与开销两方面而言Spring都是轻量的。完整的Spring框架可以在一个大小只有1MB多的JAR文件里发布。并且Spring所需的处理开销也是微不足道的。此外，Spring是非侵入式的：典型地，Spring应用中的对象不依赖于Spring的特定类。

　　控制反转——Spring通过一种称作控制反转（IoC）的技术促进了低耦合。**当应用了IoC，一个对象依赖的其它对象会通过被动的方式传递进来，而不是这个对象自己创建或者查找依赖对象**。你可以认为IoC与JNDI相反——不是对象从容器中查找依赖，而是容器在对象初始化时不等对象请求就主动将依赖传递给它。

　　面向切面——Spring提供了面向切面编程的丰富支持，允许通过分离应用的业务逻辑与系统级服务（例如审计（auditing）和事务（transaction）管理）进行内聚性的开发。应用对象只实现它们应该做的——完成业务逻辑——仅此而已。它们并不负责（甚至是意识）其它的系统级关注点，例如日志或事务支持。

　　容器——Spring包含并管理应用对象的配置和生命周期，在这个意义上它是一种容器，你可以配置你的每个bean如何被创建——基于一个可配置原型（prototype），你的bean可以创建一个单独的实例或者每次需要时都生成一个新的实例——以及它们是如何相互关联的。然而，Spring不应该被混同于传统的重量级的EJB容器，它们经常是庞大与笨重的，难以使用。

　　框架——Spring可以将简单的组件配置、组合成为复杂的应用。在Spring中，应用对象被声明式地组合，典型地是在一个XML文件里。Spring也提供了很多基础功能（事务管理、持久化框架集成等等），将应用逻辑的开发留给了你。

　　MVC——Spring的作用是整合，但不仅仅限于整合，Spring 框架可以被看做是一个企业解决方案级别的框架。客户端发送请求，服务器控制器（由DispatcherServlet实现的)完成请求的转发，控制器调用一个用于映射的类HandlerMapping，该类用于将请求映射到对应的处理器来处理请求。HandlerMapping 将请求映射到对应的处理器Controller（相当于Action）在Spring 当中如果写一些处理器组件，一般实现Controller 接口，在Controller 中就可以调用一些Service 或DAO 来进行数据操作 ModelAndView 用于存放从DAO 中取出的数据，还可以存放响应视图的一些数据。 如果想将处理结果返回给用户，那么在Spring 框架中还提供一个视图组件ViewResolver，该组件根据Controller 返回的标示，找到对应的视图，将响应response 返回给用户。（典型例子是 SpringMVC 的实现，可以参考 **[SpringMVC详解](http://www.cnblogs.com/ysocean/tag/SpringMVC%E8%AF%A6%E8%A7%A3%E7%B3%BB%E5%88%97/)**）


## 6、Spring 优点

　　Spring能有效地组织你的中间层对象，无论你是否选择使用了EJB。如果你仅仅使用了Struts或其他的包含了J2EE特有APIs的framework，你会发现Spring关注了遗留下的问题。Spring能消除在许多工程上对Singleton的过多使用。根据我的经验，这是一个主要的问题，它减少了系统的可测试性和面向对象特性。

　　Spring能消除使用各种各样格式的属性定制文件的需要，在整个应用和工程中，可通过一种一致的方法来进行配置。曾经感到迷惑，一个特定类要查找迷幻般的属性关键字或系统属性，为此不得不读Javadoc乃至源编码吗？有了Spring，你可很简单地看到类的JavaBean属性。

　　Spring能通过接口而不是类促进好的编程习惯，减少编程代价到几乎为零。

　　Spring被设计为让使用它创建的应用尽可能少的依赖于他的APIs。在Spring应用中的大多数业务对象没有依赖于Spring。所以使用Spring构建的应用程序易于单元测试。

　　Spring能使EJB的使用成为一个实现选择，而不是应用架构的必然选择。你能选择用POJOs或local EJBs来实现业务接口，却不会影响调用代码。

　　Spring帮助你解决许多问题而无需使用EJB。Spring能提供一种EJB的替换物，它们适于许多web应用。例如，Spring能使用AOP提供声明性事务而不通过使用EJB容器，如果你仅仅需要与单个的数据库打交道，甚至不需要JTA实现。

　　Spring为数据存取提供了一致的框架，不论是使用JDBC或O/R mapping产品（如Hibernate）。

**总结**：
- 1.低侵入式设计，代码污染极低
- 2.独立于各种应用服务器，基于Spring框架的应用，可以真正实现Write Once,Run Anywhere的承诺
- 3.Spring的DI机制降低了业务对象替换的复杂性，提高了组件之间的解耦
- 4.Spring的AOP支持允许将一些通用任务如安全、事务、日志等进行集中式管理，从而提供了更好的复用
- 5.Spring的ORM和DAO提供了与第三方持久层框架的良好整合，并简化了底层的数据库访问
- 6.Spring并不强制应用完全依赖于Spring，开发者可自由选用Spring框架的部分或全部