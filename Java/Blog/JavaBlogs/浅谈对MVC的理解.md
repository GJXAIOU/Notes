---
flag: red
---

# 浅谈对MVC的理解

[原文地址](https://www.cnblogs.com/lk0823/p/6753586.html)

## 一、MVC设计模式理解

MVC 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种**业务逻辑、数据、界面显示分离**的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。MVC 被独特的发展起来用于映射传统的输入、处理和输出功能在一个逻辑的图形化用户界面的结构中。

- **Model（模型）：** 数据模型，提供要展示的数据，因此包含**数据和行为**，主要提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。主要使用的技术：数据模型：实体类（JavaBean），数据访问：JDBC，Hibernate 等。

- **View（视图）**：负责进行模型的展示，一般就是我们见到的用户界面，比如 JSP，Html 等。

- **Controller（控制器）：** 接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。主要使用的技术：servlet，Struts 中的 Action 类等。

   MVC 是一个框架模式，它强制性的使应用程序的输入、处理和输出分开。使用 MVC 应用程序被分成三个核心部件：模型、视图、控制器。它们各自处理自己的任务。最典型的 MVC 就是 JSP + servlet + javabean 的模式。

![MVC流程]($resource/MVC%E6%B5%81%E7%A8%8B.jpg)


## 二、Java web 应用程序的常用组件

JAVA web 一般叫做 J2EE，java 2 企业级版本，组件包含很多：JSP/SERVLET， Web Service, Message, EJB 等等。

- **JSP**
JSP 是一种动态网页技术。它把 HTML 页面中加入 Java 脚本，以及 JSP 标签构成 JSP 文件。当浏览器请求某个 JSP 页面时，Tomcat 会把 JSP 页面翻译为 Java 文件。

- **servlet**
**Servlet运行于Web容器中**，如 Tomcat，它可以被 Web 容器动态加载，接收浏览器请求，调用其他组件处理请求，然后把处理结果返回。**当浏览器访问某个Servlet时，Web容器将会创建一个ServletRequest对象和ServletResponse对象，并且把用户的请求信息封装在ServletRequest对象中。然后把这两个对象作为参数传输给Servlet的特定方法中。在该方法中处理请求，把处理结果封装在ServletResponse对象中，返回给Web容器。最后Web容器把结果返回到浏览器去解析、显示**。

- **EJB**
企业级 JavaBean（Enterprise JavaBean, EJB）是一个用来构筑企业级应用的服务器端可被管理组件。Java 企业版 API（Java Enterprise Edition）中提供了对 EJB 的规范。EJB 是一个封装有某个应用程序之业务逻辑**服务器端组件**。

## （二）Java web 的解决方案(开发方法)

- **JSP+JavaBean**
该模式将业务逻辑与页面表现进行分离，在一定程度上增加了程序的可调试性和维护性。简单，适合小型项目的快速构建与运行。

- **JSP+javaBean+servlet**
JSP 作为视图，来表现页面；Servlet 作为控制器，控制程序的流程并调用业务进行处理；JavaBean 封装了业务逻辑。遵循了 MVC 设计模式。也是最为基础的一种构思方式。

- **Spring**
建立在核心模块之上，能够适应于多种多视图、模板技术、国际化和验证服务,实现控制逻辑和业务逻辑清晰的分离。                                                          

- **JSP+Struts+Hibernate**
利用 Struts 的 MVC 设计模式，与 Hibernate 持久化对象组成的开发方案。

- **JSP+Struts+Spring+Hibernate**
Struts 负责表示层，Spring 负责逻辑层的业务， Hibernate 持久层中数据库的操作，组成的开发方案。

## （二）常用的Java web 的MVC框架

- **Struts2**
 Struts2 是流行和成熟的基于 MVC 设计模式的 Web 应用程序框架。 Struts2 不只是 Struts1 下一个版本，它是一个完全重写的 Struts 架构。Struts 对 Model，View 和 Controller 都提供了对应的组件。但是在 ssh 开发过程中主要用 Struts 作为三层架构中的表现层，也就是 MVC 中的 View 和 Control 层。

  Struts2 提供了表单提交参数封装成 POJO 类，提交参数的类型转换，输入校验，文件的上传下载，程序的国际化，Struts2 标签，以及对 AJAX 的支持。

- **Hibernate**
     Hibernate 是一个开放源代码的对象关系映射框架，它对 JDBC 进行了非常轻量级的对象封装，使得 Java 程序员可以随心所欲的使用对象编程思维来操纵数据库。 Hibernate 可以应用在任何使用 JDBC 的场合，既可以在 Java 的客户端程序使用，也可以在 Servlet/JSP 的 Web 应用中使用，说的简单点：就是功能更加强大的 JDBC。

   Hibernate 实现了对象到数据库端的封装。就是常说的 ORM（Object Relation Mapping）,它的出现使得编程更加的面向对象，在传统的编程上，我们要将对象存储到关系数据库中，需要写很多代码来实现，而且需要考虑跨数据库的平台的问题。有了 Hibernate 可以方便的实现从对象转换到关系数据库。这就是对象持久化。

- **Spring**
主要包含两个重要功能：IOC 和 AOP，也就是常说的依赖注入和面向切面编程。当然还有 Spring 的事务功能，不过这一功能是在结合前面两者的功能实现的。

  -   **IOC** 依赖注入（控制反转），是一种设计模式。一层含义是**控制权的转移**：由传统的在程序中控制依赖转移到由容器来控制；第二层是**依赖注入**：将相互依赖的对象分离，**在 spring 配置文件中描述他们的依赖关系**。他们的依赖关系只在使用的时候才建立。简单来说就是不需要 NEW 一个对象了。

  -    **AOP** 这是一种面向切面的编程思想，这种思想使得编程思想上得到了历史性的进步。它将程序的执行过程切割成不同的面，在面之间可以插入我们想执行的逻辑。