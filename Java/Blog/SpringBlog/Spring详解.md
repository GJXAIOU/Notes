

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




# Spring系列（一）：Spring MVC bean 解析、注册、实例化流程源码剖析

## 1．背景

最近在使用Spring MVC过程中遇到了一些问题，网上搜索不少帖子后虽然找到了答案和解决方法，但这些答案大部分都只是给了结论，并没有说明具体原因，感觉总是有点不太满意。

更重要的是这些所谓的结论大多是抄来抄去，基本源自一家，真实性也有待考证。

> 要成为一名优秀的码农，不仅能熟练的复制粘贴，更要有打破砂锅问到底的精神，达到知其然也知其所以然的境界。

那作为程序员怎么能知其所以然呢？

> 答案就是阅读源代码！

此处请大家内心默读三遍。

用过Spring 的人都知道其核心就是IOC和AOP，因此要想了解Spring机制就得先从这两点入手，本文主要通过对IOC部分的机制进行介绍。

## 2\. 实验环境

在开始阅读之前，先准备好以下实验材料。

*   Spring 5.0源码([github.com/spring-proj…](https://github.com/spring-projects/spring-framework.git))

*   IDE：Intellij IDEA

IDEA 是一个优秀的开发工具，如果还在用Eclipse的建议切换到此工具进行。

IDEA有很多的快捷键，在分析过程中建议大家多用Ctrl+Alt+B快捷键，可以快速定位到实现函数。

## 3\. Spring Bean 解析注册

Spring bean的加载主要分为以下6步：

*   （1）读取XML配置文件
*   （2）XML文件解析为document文档
*   （3）解析bean
*   （4）注册bean
*   （5）实例化bean
*   （6）获取bean

### 3.1 读取XML配置文件

查看源码第一步是找到程序入口，再以入口为突破口，一步步进行源码跟踪。

Java Web应用中的入口就是web.xml。

在web.xml找到ContextLoaderListener ，此Listener负责初始化Spring IOC。

contextConfigLocation参数设置了bean定义文件地址。

```
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath*:spring.xml</param-value>
</context-param>
复制代码
```

下面是ContextLoaderListener的官方定义：

> public class ContextLoaderListener extends ContextLoader implements ServletContextListener

> Bootstrap listener to start up and shut down Spring's root WebApplicationContext. Simply delegates to ContextLoader as well as to ContextCleanupListener.

> [docs.spring.io/spring-fram…](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/ContextLoaderListener.html)

翻译过来ContextLoaderListener作用就是负责启动和关闭Spring root WebApplicationContext。

具体WebApplicationContext是什么？开始看源码。

```
package org.springframework.web.context;
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }
    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }
    //servletContext初始化时候调用
    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext();
    }
    //servletContext销毁时候调用
    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
    }
}
复制代码
```

从源码看出此Listener主要有两个函数，一个负责初始化WebApplicationContext，一个负责销毁。

继续看initWebApplicationContext函数。

```
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
//初始化Spring容器时如果发现servlet 容器中已存在根Spring容根器则抛出异常，证明rootWebApplicationContext只能有一个。
   if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
      throw new IllegalStateException(
            "Cannot initialize context because there is already a root application context present - " +
            "check whether you have multiple ContextLoader* definitions in your web.xml!");
   }
   if (this.context == null) {
	//1.创建webApplicationContext实例
        this.context = createWebApplicationContext(servletContext);
   }
   if (this.context instanceof ConfigurableWebApplicationContext) {
        ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
	 //2.配置WebApplicationContext
        configureAndRefreshWebApplicationContext(cwac, servletContext);
    }
    //3.把生成的webApplicationContext 设置为root webApplicationContext。
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
    return this.context; 

}
复制代码
```

在上面的代码中主要有两个功能：

*   （1）创建WebApplicationContext实例。
*   （2）配置生成WebApplicationContext实例。
*   （3）把生成的webApplicationContext 设置为root webApplicationContext。

#### 3.1.1 创建WebApplicationContext实例

进入CreateWebAPPlicationContext函数

```
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
   //得到ContextClass类,默认实例化的是XmlWebApplicationContext类
   Class<?> contextClass = determineContextClass(sc);
   //实例化Context类
   return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
}
复制代码
```

进入determineContextClass函数。

```
protected Class<?> determineContextClass(ServletContext servletContext) {
   // 此处CONTEXT_CLASS_PARAM = "contextClass"String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
   if (contextClassName != null) {
         //若设置了contextClass则使用定义好的ContextClass。
         return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
      }
   else {
      //此处获取的是在Spring源码中ContextLoader.properties中配置的org.springframework.web.context.support.XmlWebApplicationContext类。
      contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
      return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
}
复制代码
```

#### 3.1.2 配置Web ApplicationContext

进入configureAndReFreshWebApplicaitonContext函数。

```
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac, ServletContext sc) {
  //webapplicationContext设置servletContext.
   wac.setServletContext(sc);
   // 此处CONFIG_LOCATION_PARAM = "contextConfigLocation"，即读即取web.xm中配设置的contextConfigLocation参数值，获得spring bean的配置文件.
   String configLocationParam = sc.getInitParameter(CONFIG_LOCATION_PARAM);
   if (configLocationParam != null) {
      //webApplicationContext设置配置文件路径设。
      wac.setConfigLocation(configLocationParam);
   }
   //开始处理bean
   wac.refresh();
}
复制代码
```

### 3.2 解析XML文件

上面wac变量声明为ConfigurableWebApplicationContext类型，ConfigurableWebApplicationContext又继承了WebApplicationContext。

WebApplication Context有很多实现类。 但从上面determineContextClass得知此处wac实际上是XmlWebApplicationContext类，因此进入XmlWebApplication类查看其继承的refresh()方法。

沿方法调用栈一层层看下去。

```
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      //获取beanFactory
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
     // 实例化所有声明为非懒加载的单例bean 
      finishBeanFactoryInitialization(beanFactory);
    }
}
复制代码
```

获取beanFactory。

```
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
     //初始化beanFactory
     refreshBeanFactory();
     return beanFactory;
}
复制代码
```

beanFactory初始化。

```
@Override
protected final void refreshBeanFactory() throws BeansException {
      DefaultListableBeanFactory beanFactory = createBeanFactory();
      //加载bean定义
      loadBeanDefinitions(beanFactory);
      synchronized (this.beanFactoryMonitor) {
      this.beanFactory = beanFactory;
      }
}
复制代码
```

加载bean。

```
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
   //创建XmlBeanDefinitionReader实例来解析XML配置文件
   XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
   initBeanDefinitionReader(beanDefinitionReader);
   //解析XML配置文件中的bean。
   loadBeanDefinitions(beanDefinitionReader);
}
复制代码
```

读取XML配置文件。

```
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
//此处读取的就是之前设置好的web.xml中配置文件地址
   String[] configLocations = getConfigLocations();
   if (configLocations != null) {
      for (String configLocation : configLocations) {
         //调用XmlBeanDefinitionReader读取XML配置文件
         reader.loadBeanDefinitions(configLocation);
      }
   }
}
复制代码
```

XmlBeanDefinitionReader读取XML文件中的bean定义。

```
public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {
   ResourceLoader resourceLoader = getResourceLoader();
      Resource resource = resourceLoader.getResource(location);
      //加载bean
      int loadCount = loadBeanDefinitions(resource);
      return loadCount;
   }
}
复制代码
```

继续查看loadBeanDefinitons函数调用栈，进入到XmlBeanDefinitioReader类的loadBeanDefinitions方法。

```
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
      //获取文件流
      InputStream inputStream = encodedResource.getResource().getInputStream();
      InputSource inputSource = new InputSource(inputStream);
     //从文件流中加载定义好的bean。
      return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
   }
}
复制代码
```

最终将XML文件解析成Document文档对象。

```
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource)
      //XML配置文件解析到Document实例中
      Document doc = doLoadDocument(inputSource, resource);
      //注册bean
      return registerBeanDefinitions(doc, resource);
   }
复制代码
```

### 3.3 解析bean

上一步完成了XML文件的解析工作，接下来将XML中定义的bean注册到webApplicationContext，继续跟踪函数。

```
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
   BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
   //使用documentRedder实例读取bean定义
   documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
   }
复制代码
```

用BeanDefinitionDocumentReader对象来注册bean。

```
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
   this.readerContext = readerContext;
   //读取document元素
   Element root = doc.getDocumentElement();
   //真正开始注册bean
   doRegisterBeanDefinitions(root);
}
复制代码
```

解析XML文档。

```
protected void doRegisterBeanDefinitions(Element root) {
    //预处理XML 
   preProcessXml(root);
   //解析注册bean
   parseBeanDefinitions(root, this.delegate);
   postProcessXml(root);
}
复制代码
```

循环解析XML文档中的每个元素。

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   //如果该元素属于默认命名空间走此逻辑。Spring的默认namespace为：http://www.springframework.org/schema/beans“
   if (delegate.isDefaultNamespace(root)) {
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (node instanceof Element) {
            Element ele = (Element) node;
            //对document中的每个元素都判断其所属命名空间，然后走相应的解析逻辑
            if (delegate.isDefaultNamespace(ele)) {
               parseDefaultElement(ele, delegate);
            }
            else {
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
      //如果该元素属于自定义namespace走此逻辑 ，比如AOP，MVC等。
      delegate.parseCustomElement(root);
   }
}
复制代码
```

下面是默认命名空间的解析逻辑。

不明白Spring的命名空间的可以网上查一下，其实类似于package，用来区分变量来源，防止变量重名。

```
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
   //解析import元素
   if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
      importBeanDefinitionResource(ele);
   }
   //解析alias元素
   else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
      processAliasRegistration(ele);
   }
   //解析bean元素
   else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
      processBeanDefinition(ele, delegate);
   }
   //解析beans元素
   else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
      // recurse
      doRegisterBeanDefinitions(ele);
   }
}
复制代码
```

这里我们就不一一跟踪，以解析bean元素为例继续展开。

```
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
   //解析bean
   BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
   if (bdHolder != null) {
         // 注册bean
         BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
   }
}
复制代码
```

解析bean元素，最后把每个bean解析为一个包含bean所有信息的BeanDefinitionHolder对象。

```
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
   String id = ele.getAttribute(ID_ATTRIBUTE);
   String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);
   AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
   return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
}
复制代码
```

### 3.4 注册bean

接下来将解析到的bean注册到webApplicationContext中。接下继续跟踪registerBeanDefinition函数。

```
public static void registerBeanDefinition(
      BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
      throws BeanDefinitionStoreException {
      // 获取beanname
      String beanName = definitionHolder.getBeanName();
      //注册bean
      registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
      // 注册bean的别名
      String[] aliases = definitionHolder.getAliases();
      if (aliases != null) {
          for (String alias : aliases) {
             registry.registerAlias(beanName, alias);
     }
   }
}
复制代码
```

跟踪registerBeanDefinition函数，此函数将bean信息保存到到webApplicationContext的beanDefinitionMap变量中，该变量为map类型，保存Spring 容器中所有的bean定义。

```
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
      throws BeanDefinitionStoreException {
	//把bean信息保存到beanDefinitionMap中
        this.beanDefinitionMap.put(beanName, beanDefinition);
	//把beanName 保存到List 类型的beanDefinitionNames属性中
    this.beanDefinitionNames.add(beanName);
   }
复制代码
```

### 3.5 实例化bean

Spring 实例化bean的时机有两个。

一个是容器启动时候，另一个是真正调用的时候。

如果bean声明为scope=singleton且lazy-init=false，则容器启动时候就实例化该bean（Spring 默认就是此行为）。否则在调用时候再进行实例化。

相信用过Spring的同学们都知道以上概念，但是为什么呢？

继续从源码角度进行分析，回到之前XmlWebApplication的refresh（）方法。

```
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      //生成beanFactory,
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
      // 实例化所有声明为非懒加载的单例bean 
      finishBeanFactoryInitialization(beanFactory);
 }
复制代码
```

可以看到获得beanFactory后调用了 finishBeanFactoryInitialization()方法，继续跟踪此方法。

```
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
  // 初始化非懒加载的单例bean   
  beanFactory.preInstantiateSingletons();
}
复制代码
```

预先实例化单例类逻辑。

```
public void preInstantiateSingletons() throws BeansException {
   // 获取所有注册的bean
   List<String> beanNames = new ArrayList<String>(this.beanDefinitionNames);
   // 遍历bean 
   for (String beanName : beanNames) {
      RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
      //如果bean是单例且非懒加载，则获取实例
      if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            getBean(beanName);
      }
   }
}
复制代码
```

获取bean。

```
public Object getBean(String name) throws BeansException {
   return doGetBean(name, null, null, false);
}
复制代码
```

doGetBean中处理的逻辑很多，为了减少干扰，下面只显示了创建bean的函数调用栈。

```
protected <T> T doGetBean(
      final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
      throws BeansException {
	//创建bean
       createBean(beanName, mbd, args);
}
复制代码
```

创建bean。

```
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    return beanInstance;
}
复制代码
```

```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
      throws BeanCreationException {
      // 实例化bean   
      instanceWrapper = createBeanInstance(beanName, mbd, args);
}
复制代码
```

```
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
   //实例化bean
   return instantiateBean(beanName, mbd);
}
复制代码
```

```
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
      //调用实例化策略进行实例化
      beanInstance = getInstantiationStrategy().instantiate(mbd, beanName,
}
复制代码
```

判断哪种动态代理方式实例化bean。

```
public Object instantiate(RootBeanDefinition bd, String beanName, BeanFactory owner) {
   //使用JDK动态代理
   if (bd.getMethodOverrides().isEmpty()) {
      return BeanUtils.instantiateClass(constructorToUse);
   }
   else {
     //使用CGLIB动态代理
     return instantiateWithMethodInjection(bd, beanName, owner);
   }
}
复制代码
```

不管哪种方式最终都是通过反射的形式完成了bean的实例化。

```
public static <T> T instantiateClass(Constructor<T> ctor, Object... args) 
      ReflectionUtils.makeAccessible(ctor);
      return ctor.newInstance(args);
}
复制代码
```

### 3.6 获取bean

我们继续回到doGetBean函数，分析获取bean的逻辑。

```
protected <T> T doGetBean(
      final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
      throws BeansException {
    //获取beanName
     final String beanName = transformedBeanName(name);
     Object bean
    // 先检查该bean是否为单例且容器中是否已经存在例化的单例类
    Object sharedInstance = getSingleton(beanName);
    //如果已存在该bean的单例类
    if (sharedInstance != null && args == null) {
       bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }
    else{
          // 获取父BeanFactory
          BeanFactory parentBeanFactory = getParentBeanFactory();
          //先判断该容器中是否注册了此bean，如果有则在该容器实例化bean，否则再到父容器实例化bean
          if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
               String nameToLookup = originalBeanName(name);
               // 如果父容器有该bean，则调用父beanFactory的方法获得该bean
               return (T) parentBeanFactory.getBean(nameToLookup, args);
           }
            //如果该bean有依赖bean，先实递归例化依赖bean。    
            String[] dependsOn = mbd.getDependsOn();
            if (dependsOn != null) {
                for (String dep : dependsOn) {
                   registerDependentBean(dep, beanName);
                   getBean(dep);
                }
            }
    	    //如果scope为Singleton执行此逻辑
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
                   @Override
                   public Object getObject() throws BeansException {
    		//调用创建bean方法
                         return createBean(beanName, mbd, args);
                      }

                   }
                });
             }
             //如果scope为Prototype执行此逻辑，每次获取时候都实例化一个bean
             else if (mbd.isPrototype()) {
                Object prototypeInstance = null;
                   prototypeInstance = createBean(beanName, mbd, args);
             }
             //如果scope为Request,Session,GolbalSession执行此逻辑
             else {
                String scopeName = mbd.getScope();
                final Scope scope = this.scopes.get(scopeName);
                   Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                      @Override
                      public Object getObject() throws BeansException {
                            return createBean(beanName, mbd, args);
                      }
                   });
             }
          }
}
      return (T) bean;
}
复制代码
```

上面方法中首先调用getSingleton(beanName)方法来获取单例bean，如果获取到则直接返回该bean。方法调用栈如下：

```
public Object getSingleton(String beanName) {
   return getSingleton(beanName, true);
}
复制代码
```

```
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
//从singletonObjects中获取bean。
   Object singletonObject = this.singletonObjects.get(beanName);
   return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
复制代码
```

getSingleton方法先从singletonObjects属性中获取bean 对象,如果不为空则返回该对象，否则返回null。

那 singletonObjects保存的是什么？什么时候保存的呢？

回到doGetBean（）函数继续分析。如果singletonObjects没有该bean的对象，进入到创建bean的逻辑。处理逻辑如下：

```
//获取父beanFactory
BeanFactory parentBeanFactory = getParentBeanFactory();
//如果该容器中没有注册该bean，且父容器不为空，则去父容器中获取bean后返回
if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      return parentBeanFactory.getBean(nameToLookup, requiredType);
}

复制代码
```

下面是判断容器中有没有注册bean的逻辑，此处beanDefinitionMap相信大家都不陌生，在注册bean的流程里已经说过所有的bean信息都会保存到该变量中。

```
public boolean containsBeanDefinition(String beanName) {
   Assert.notNull(beanName, "Bean name must not be null");
   return this.beanDefinitionMap.containsKey(beanName);
}
复制代码
```

如果该容器中已经注册过bean，继续往下走。先获取该bean的依赖bean，如果镩子依赖bean，则先递归获取相应的依赖bean。

```
String[] dependsOn = mbd.getDependsOn();
if (dependsOn != null) {
    for (String dep : dependsOn) {
         registerDependentBean(dep, beanName);
         getBean(dep);
    }
}   
复制代码
```

依赖bean创建完成后，接下来就是创建自身bean实例了。

获取bean实例的处理逻辑有三种，即Singleton、Prototype、其它(request、session、global session)，下面一一说明。

#### 3.6.1 Singleton

如果bean是单例模式，执行此逻辑。

```
if (mbd.isSingleton()) {
       sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
          @Override
          public Object getObject() throws BeansException {
    	    //创建bean回调
            return createBean(beanName, mbd, args); 
       });
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
复制代码
```

获取单例bean，如果已经有该bean的对象直接返回。如果没有则创建单例bean对象，并添加到容器的singletonObjects Map中，以后直接从singletonObjects直接获取bean。

```
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
   synchronized (this.singletonObjects) {
      Object singletonObject = this.singletonObjects.get(beanName);
	  //如果singletonObjects中没有该bean
      if (singletonObject == null) {
    	//回调参数传进来的ObjectFactory的getObject方法，即调用createBean方法创建bean实例
            singletonObject = singletonFactory.getObject();
        //置新创建单例bean标志位为true。
           newSingleton = true;
         if (newSingleton) {
        //如果是新创建bean，注册新生成bean对象
            addSingleton(beanName, singletonObject);
         }
      }
      //返回获取的单例bean
      return (singletonObject != NULL_OBJECT ? singletonObject : null);
   }
}
复制代码
```

把新生成的单例bean加入到类型为MAP 的singletonObjects属性中，这也就是前面singletonObjects（）方法中获取单例bean时从此Map中获取的原因。

```
protected void addSingleton(String beanName, Object singletonObject) {
   synchronized (this.singletonObjects) {
    //把新生成bean对象加入到singletonObjects属性中。
      this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

      this.registeredSingletons.add(beanName);
   }
}
复制代码
```

#### 3.6.2 Prototype

Prototype是每次获取该bean时候都新建一个bean，因此逻辑比较简单，直接创建一个bean后返回。

```
else if (mbd.isPrototype()) {
    Object prototypeInstance = null;
    //创建bean
    prototypeInstance = createBean(beanName, mbd, args);
    bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
}
复制代码
```

#### 3.6.3 request、session、global session

```
else {
    //获取该bean的scope   
    String scopeName = mbd.getScope();
    //获取相应scope
    final Scope scope = this.scopes.get(scopeName);
    //获取相应scope的实例化对象
    Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
             @Override
             public Object getObject() throws BeansException {
                   return createBean(beanName, mbd, args);
             }
    });
    bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
   }
复制代码
```

从相应scope获取对象实例。

```
public Object get(String name, ObjectFactory<?> objectFactory) {
   RequestAttributes attributes = RequestContextHolder.currentRequestAttributes();
   //先从指定scope中获取bean实例，如果没有则新建，如果已经有直接返回
   Object scopedObject = attributes.getAttribute(name, getScope());
   if (scopedObject == null) {
      //回调函数调用createBean创建实例
      scopedObject = objectFactory.getObject();
      //创建实例后保存到相应scope中
      attributes.setAttribute(name, scopedObject, getScope());
      }
   }
   return scopedObject;
}
复制代码
```

判断scope，获取实例函数逻辑。

```
public Object getAttribute(String name, int scope) {
   //scope是request时
   if (scope == SCOPE_REQUEST) {
      //从request中获取实例
      return this.request.getAttribute(name);
   }
   else {
      PortletSession session = getSession(false);
      if (session != null) {
         //scope是globalSession时，从application中获取实例
         if (scope == SCOPE_GLOBAL_SESSION) {
            //从globalSession中获取实例
            Object value = session.getAttribute(name, PortletSession.APPLICATION_SCOPE);
            return value;
         }
         else {
            //从session中获取实例
            Object value = session.getAttribute(name);
            return value;
         }
      }
      return null;
   }
}
复制代码
```

在相应scope中设置实例函数逻辑。

```
public void setAttribute(String name, Object value, int scope) {
   if (scope == SCOPE_REQUEST) {
      this.request.setAttribute(name, value);
   }
   else {
      PortletSession session = getSession(true);
      if (scope == SCOPE_GLOBAL_SESSION) {
         session.setAttribute(name, value, PortletSession.APPLICATION_SCOPE);
      }
      else {
         session.setAttribute(name, value);
      }
   }
}
复制代码
```

以上就是Spring bean从无到有的整个逻辑。

## 4\. 小结

从源码角度分析 bean的实例化流程到此基本接近尾声了。

回到开头的问题，ContextLoaderListener中初始化的WebApplicationContext到底是什么呢？

通过源码的分析我们知道WebApplicationContext负责了bean的创建、保存、获取。其实也就是我们平时所说的IOC容器，只不过名字表述不同而已。

## 5\. 尾声

本文主要是讲解了XML配置文件中bean的解析、注册、实例化。对于其它命名空间的解析还没有讲到，后续的文章中会一一介绍。

希望通过本文让大家在以后使用Spring的过程中有“一切尽在掌控之中”的感觉，而不仅仅是稀里糊涂的使用。



# Spring 系列（二）：Spring MVC的父子容器

## 1.背景

在使用Spring MVC时候大部分同学都会定义两个配置文件，一个是Spring的配置文件spring.xml，另一个是Spring MVC的配置文件spring-mvc.xml。

在这里给大家抛个问题，如果在spring.xml和spring-mvc.xml文件中同时定义一个相同id的单例bean会怎样呢？大家可以先思考一下再继续往下看。

我做了个实验，结论是：容器中会同时存在两个相同id 的bean，而且使用起来互不干扰。

这是为什么呢？学过Spring的同学肯定会质疑，众所周知id是bean的唯一标示，怎么可能同时存在两个相同id的bean呢？是不是我在胡扯呢？

原谅我在这和大家卖了个关子，其实大家说的都没错，因为这里涉及到Spring MVC父子容器的知识点。

这个知识点是：在使用Spring MVC过程中会存在Spring MVC 、Spring两个IOC容器，且Spring MVC是Spring的子容器。

那这个父子容器到底是什么呢？

为了保证我所说的权威性，而不是知识的二道贩子，我将从Spring 官方文档和源码两方面展开介绍。

## 2.Spring MVC父子容器

### 2.1 web.xml配置

还是先找程序入口，查看web.xml配置文件，找到Spring MVC相关配置。

```
<servlet>
        <servlet-name>spring-mvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
</servlet>
复制代码
```

配置很简单，只是配置了一个类型为DispatcherServlet类型的Servlet,并设置了初始化参数。那DispatcherServlet是什么呢？

### 2.2 DispatcherServlet类介绍

查看[API文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)

![](https://user-gold-cdn.xitu.io/2019/4/21/16a3ec2252cfac9b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

从继承图看出最终继承自HttpServlet，其实就是一个普通的Servlet。那为什么这个Servlet就能完成Spring MVC一系列复杂的功能呢？继续往下看。

### 2.3 DispatcherServlet工作流程

![](https://user-gold-cdn.xitu.io/2019/4/21/16a3ecb865659338?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

DispatcherServlet工作流程如下：

*   (1) 所有请求先发到DispacherServlet
*   (2) DispacherServlet根据请求地址去查询相应的Controller，然后返回给DispacherServlet。
*   (3) DispacherServlet得到Controller后，让Controler处理相应的业务逻辑。
*   (4) Controler处理处理完后将结果返回给DispacherServlet。
*   (5) DispacherServlet把得到的结果用视图解析器解析后获得对应的页面。
*   (6) DispacherServlet跳转到解析后的页面。

在整个过程中DispatcherServlet承当了一个中心控制器的角色来处理各种请求。

### 2.4 DispatcherServlet上下文继承关系

![](https://user-gold-cdn.xitu.io/2019/4/21/16a3ede6206d0521?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图来自Spring官网：

```
https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html
复制代码
```

从图中可以看到DispatcherServlet里面有一个 Servlet WebApplicationContext，继承自 Root WebApplicationContext。

从[上篇文章](https://juejin.im/user/5895420861ff4b006b040369)中我们知道WebApplicationContext其实就是一个IOC容器，root WebApplicationContext是Spring容器。

这说明DispatcherServlet中里创建了一个IOC容器并且这个容器继承了Spring 容器，也就是Spring的子容器。

而且官方文档中还有如下一段文字描述：

```
For many applications, having a single WebApplicationContext is simple and suffices. It is also possible to have a context hierarchy where one root WebApplicationContext is shared across multiple DispatcherServlet (or other Servlet) instances, each with its own child WebApplicationContext configuration. See Additional Capabilities of the ApplicationContext for more on the context hierarchy feature.

The root WebApplicationContext typically contains infrastructure beans, such as data repositories and business services that need to be shared across multiple Servlet instances.
Those beans are effectively inherited and can be overridden (that is, re-declared) in the Servlet-specific child WebApplicationContext, which typically contains beans local to the given Servlet.

复制代码
```

结合图和上述文字我们可以得出以下信息：

> 1.  应用中可以包含多个IOC容器。

> 1.  DispatcherServlet的创建的子容器主要包含Controller、view resolvers等和web相关的一些bean。

> 1.  父容器root WebApplicationContex主要包含包含一些基础的bean，比如一些需要在多个servlet共享的dao、service等bean。

> 1.  如果在子容器中找不到bean的时候可以去父容器查找bean。

看到这里也许大家心中也许就明白文章开头中我说的Spring MVC中的父子容器了，对那个问题也有了自己的判断和答案。

当然文章还没有结束，毕竟这还仅限于对官方文档的理解，为了进一步验证，我们拿出终极武器：

> 阅读源码！

### 2.5 DispatcherServlet源码分析

本小节我们分为Spring MVC容器的创建和bean的获取两部分进行分析。

#### 2.5.1 Spring MVC容器的创建

前面分析到DispatcherServlet本质上还是一个Servlet ，既然是Servlet ，了解Servlet生命周期的同学都知道Web 容器装载Servlet第一步是执行init（）函数,因此以DispatcherServlet 的init函数为突破口进行分析。

```
@Override
public final void init() throws ServletException {
   // 1.读取init parameters 等参数，其中就包括设置contextConfigLocation 
    PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
   //2.初始化servlet中使用的bean
   initServletBean();
}
复制代码
```

在第1步读取init parameter的函数最终会调用setContextConfigLocation（）设置配置文件路径。此处重点介绍initServletBean（），继续跟踪。

```
Override
protected final void initServletBean() throws ServletException {
      //初始化webApplicationContext
      this.webApplicationContext = initWebApplicationContext();
}
复制代码
```

```
protected WebApplicationContext initWebApplicationContext() {
    //1.获得rootWebApplicationContext
    WebApplicationContext rootContext =
            WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;
    //2.如果还没有webApplicatioinContext，创建webApplicationContext
    if (wac == null) {
	//创建webApplicationContext
        wac = createWebApplicationContext(rootContext);
    }
   return wac;
}

复制代码
```

可以看到上面初始化webApplicationContext分为2步。

*   （1）获取父容器rootWebApplicationContext。
*   （2）创建子容器。

我们先看看rootWebApplicationContext是如何获取的。

```
public static WebApplicationContext getWebApplicationContext(ServletContext sc) {
   return getWebApplicationContext(sc, WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
}

public static WebApplicationContext getWebApplicationContext(ServletContext sc, String attrName) {
   Object attr = sc.getAttribute(attrName);
   return (WebApplicationContext) attr;
}
复制代码
```

从上面代码中我没看到是从ServletContext获取了名为“WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE”的webApplicationContext。

认真看过[上篇文章](https://juejin.im/user/5895420861ff4b006b040369)的同学应该记得这个属性是在Spring初始化 容器initWebApplicationContext（）函数中的第3步设置进去的，取得的值即Spring IOC容器。

继续看如何创建webApplicationContext。

```
protected WebApplicationContext createWebApplicationContext(WebApplicationContext parent) {
   return createWebApplicationContext((ApplicationContext) parent);
}
复制代码
```

```
createWebApplicationContext(ApplicationContext parent) {
  //1.获取WebApplicationContext实现类，此处其实就是XmlWebApplicationContext
  Class<?> contextClass = getContextClass();
  //生成XmlWebApplicationContext实例
  ConfigurableWebApplicationContext wac =
         (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
  //2.设置rootWebApplicationContext为父容器 
   wac.setParent(parent);
  //3.设置配置文件
   wac.setConfigLocation(getContextConfigLocation());
  //4.配置webApplicationContext.
   configureAndRefreshWebApplicationContext(wac);
   return wac;
}
复制代码
```

```
protected void configureAndRefreshWebApplicationContext(ConfigurableWebApplicationContext wac) {
   //开始处理bean
   wac.refresh();
}
复制代码
```

看到这里同学们有没有是曾相识的感觉。是的，这段逻辑和[上篇文章](https://juejin.im/user/5895420861ff4b006b040369)创建Spring IOC的逻辑类似。

唯一不同的是在第2步会把Spring容器设置为自己的父容器。至于新建容器中bean的注册、解析、实例化等流程和Spring IOC容器一样都是交给XmlWebApplicationContext类处理，还没有掌握的同学可以看[上篇文章](https://juejin.im/user/5895420861ff4b006b040369)。

#### 2.5.2 Spring MVC Bean的获取

Spring MVC bean的获取其实我们在上篇文章已经介绍过，这次再单拎出来介绍一下，加深记忆。

```
protected <T> T doGetBean(
    // 获取父BeanFactory
    BeanFactory parentBeanFactory = getParentBeanFactory();
    //如果父容器不为空，且本容器没有注册此bean就去父容器中获取bean
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
         // 如果父容器有该bean，则调用父beanFactory的方法获得该bean
         return (T) parentBeanFactory.getBean(nameToLookup,args);
    }
    //如果子容器注册了bean，执行一系列实例化bean操作后返回bean.
    //此处省略实例化过程
    .....
    return (T) bean;
}
复制代码
```

上面代码就可以对应官方文档中“如果子容器中找不到bean,就去父容器找”的解释了。

## 3.小结

看完上面的介绍，相信大家对Spring MVC父子容器的概念都有所了解，现在我们分析文章开头的问题。

如果spring.xml和spring-mvc.xml定义了相同id的bean会怎样？假设id=test。

1.首先Spring 初始化，Spring IOC 容器中生成一个id为test bean实例。

2.Spring MVC开始初始化，生成一个id为test bean实例。

此时，两个容器分别有一个相同id的bean。那用起来会不会混淆？

答案是不会。

当你在Spring MVC业务逻辑中使用该bean时，Spring MVC会直接返回自己容器的bean。

当你在Spring业务逻辑中使用该bean时，因为子容器的bean对父亲是不可见的，因此会直接返回Spring容器中的bean。

虽然上面的写法不会造成问题。但是在实际使用过程中，建议大家都把bean定义都写在spring.xml文件中。

因为使用单例bean的初衷是在IOC容器中只存在一个实例，如果两个配置文件都定义，会产生两个相同的实例，造成资源的浪费，也容易在某些场景下引发缺陷。

## 4.尾声

现在大家基本都不使用在xml文件中定义bean的形式，而是用注解来定义bean，然后在xml文件中定义扫描包。如下：

```
<context:component-scan base-package="xx.xx.xx"/>
复制代码
```

那如果在spring.xml和spring-mvc.xml配置了重复的包会怎样呢？

如果本文看明白的同学此时已经知道了答案。

答案是会在两个父子IOC容器中生成大量的相同bean，这就会造成内存资源的浪费。

也许有同学想到，那只在spring.xml中设置扫描包不就能避免这种问题发生了吗，答案是这样吗？

大家可以试试，这样会有什么问题。如果不行，那是为什么呢？

欲知分晓，敬请期待下篇分解！





# Spring 系列（三）：你真的懂@RequestMapping吗？

## 1.前言

[上篇](https://juejin.im/post/5cbc10b46fb9a0689f4c2c22)给大家介绍了Spring MVC父子容器的概念，主要提到的知识点是:

```
Spring MVC容器是Spring容器的子容器，当在Spring MVC容器中找不到bean的时候就去父容器找.
复制代码
```

在文章最后我也给大家也留了一个问题，既然子容器找不到就去父容器找，那干脆把bean定义都放在父容器不就行了？是这样吗，我们做个实验。

我们把<context:component-scan base-package="xx.xx.xx"/> 这条语句从spring-mvc.xml文件中挪到spring.xml中，重启应用。会发现报404，如下图：

![](https://user-gold-cdn.xitu.io/2019/4/24/16a4cf72528d985c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

404说明请求的资源没有找到，为什么呢？

使用Spring MVC的同学一般都会以下方式定义请求地址:

```
@Controller
@RequestMapping("/test")
public class Test {
   @RequestMapping(value="/handle", method=RequestMethod.POST)
   public void handle();
}
复制代码
```

@Controller注解用来把一个类定义为Controller。

@RequestMapping注解用来把web请求映射到相应的处理函数。

@Controller和@RequestMapping结合起来完成了Spring MVC请求的派发流程。

为什么两个简单的注解就能完成这么复杂的功能呢？又和<context:component-scan base-package="xx.xx.xx"/>的位置有什么关系呢？

让我们开始分析源码。

## 2.@RequestMapping流程分析

@RequestMapping流程可以分为下面6步：

*   1.注册RequestMappingHandlerMapping bean 。
*   2.实例化RequestMappingHandlerMapping bean。
*   3.获取RequestMappingHandlerMapping bean实例。
*   4.接收requst请求。
*   5.在RequestMappingHandlerMapping实例中查找对应的handler。
*   6.handler处理请求。

为什么是这6步，我们展开分析。

### 2.1 注册RequestMappingHandlerMapping bean

第一步还是先找程序入口。

使用Spring MVC的同学都知道，要想使@RequestMapping注解生效，必须得在xml配置文件中配置< mvc:annotation-driven/>。因此我们以此为突破口开始分析。

在[Spring系列（一）：bean 解析、注册、实例化](https://juejin.im/post/5cb89dae6fb9a0686b47306d) 文中我们知道xml配置文件解析完的下一步就是解析bean。在这里我们继续对那个方法展开分析。如下：

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   //如果该元素属于默认命名空间走此逻辑。Spring的默认namespace为：http://www.springframework.org/schema/beans“
   if (delegate.isDefaultNamespace(root)) {
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (node instanceof Element) {
            Element ele = (Element) node;
            //对document中的每个元素都判断其所属命名空间，然后走相应的解析逻辑
            if (delegate.isDefaultNamespace(ele)) {
               parseDefaultElement(ele, delegate);
            }
            else {
              //如果该元素属于自定义namespace走此逻辑 ，比如AOP，MVC等。
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
      //如果该元素属于自定义namespace走此逻辑 ，比如AOP，MVC等。
      delegate.parseCustomElement(root);
   }
}
复制代码
```

方法中根据元素的命名空间来进行不同的逻辑处理，如bean、beans等属于默认命名空间执行parseDefaultElement()方法，其它命名空间执行parseCustomElement()方法。

< mvc:annotation-driven/>元素属于mvc命名空间，因此进入到 parseCustomElement()方法。

```
public BeanDefinition parseCustomElement(Element ele) {
    //解析自定义元素
    return parseCustomElement(ele, null);
}
复制代码
```

进入parseCustomElement(ele, null)方法。

```
public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
    //获取该元素namespace url
    String namespaceUri = getNamespaceURI(ele);
    //得到NamespaceHandlerSupport实现类解析元素
    NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
    return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
复制代码
```

进入NamespaceHandlerSupport类的parse()方法。

```
@Override
public BeanDefinition parse(Element element, ParserContext parserContext) {
    //此处得到AnnotationDrivenBeanDefinitionParser类来解析该元素
    return findParserForElement(element, parserContext).parse(element, parserContext);
}
复制代码
```

上面方法分为两步，（1）获取元素的解析类。（2）解析元素。

（1）获取解析类。

```
private BeanDefinitionParser findParserForElement(Element element, ParserContext parserContext) {
    String localName = parserContext.getDelegate().getLocalName(element);
    BeanDefinitionParser parser = this.parsers.get(localName);
    return parser;
}
复制代码
```

Spring MVC中含有多种命名空间，此方法会根据元素所属命名空间得到相应解析类，其中< mvc:annotation-driven/>对应的是AnnotationDrivenBeanDefinitionParser解析类。

（2）解析< mvc:annotation-driven/>元素

进入AnnotationDrivenBeanDefinitionParser类的parse（）方法。

```
@Override
public BeanDefinition parse(Element element, ParserContext context) {
    Object source = context.extractSource(element);
    XmlReaderContext readerContext = context.getReaderContext();
    //生成RequestMappingHandlerMapping bean信息
    RootBeanDefinition handlerMappingDef = new RootBeanDefinition(RequestMappingHandlerMapping.class);
    handlerMappingDef.setSource(source);
    handlerMappingDef.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
    handlerMappingDef.getPropertyValues().add("order", 0);
    handlerMappingDef.getPropertyValues().add("contentNegotiationManager", contentNegotiationManager);
    //此处HANDLER_MAPPING_BEAN_NAME值为:RequestMappingHandlerMapping类名
    //容器中注册name为RequestMappingHandlerMapping类名
    context.registerComponent(new BeanComponentDefinition(handlerMappingDef, HANDLER_MAPPING_BEAN_NAME));
}
复制代码
```

可以看到上面方法在Spring MVC容器中注册了一个名为“HANDLER_MAPPING_BEAN_NAME”,类型为RequestMappingHandlerMapping的bean。

至于这个bean能干吗，继续往下分析。

### 2.2\. RequestMappingHandlerMapping bean实例化

bean注册完后的下一步就是实例化。

在开始分析实例化流程之前，我们先介绍一下RequestMappingHandlerMapping是个什么样类。

#### 2.2.1 RequestMappingHandlerMapping继承图

![](https://user-gold-cdn.xitu.io/2019/4/24/16a4d2126e4259ad?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图信息比较多，我们查找关键信息。可以看到这个类间接实现了HandlerMapping接口，是HandlerMapping类型的实例。

除此之外还实现了ApplicationContextAware和IntitalzingBean 这两个接口。

在这里简要介绍一下这两个接口：

#### 2.2.2 ApplicationContextAware接口

下面是[官方介绍](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/ApplicationContextAware.html)：

```
public interface ApplicationContextAware extends Aware

Interface to be implemented by any object that wishes to be notified of the ApplicationContext that it runs in.
复制代码
```

该接口只包含以下方法：

```
void setApplicationContext(ApplicationContext applicationContext)
throws BeansException

Set the ApplicationContext that this object runs in. Normally this call will be used to initialize the object.
复制代码
```

概括一下上面表达的信息：如果一个类实现了ApplicationContextAware接口，Spring容器在初始化该类时候会自动回调该类的setApplicationContext()方法。这个接口主要用来让实现类得到Spring 容器上下文信息。

#### 2.2.3 IntitalzingBean接口

下面是[官方介绍](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html)：

```
public interface InitializingBean

Interface to be implemented by beans that need to react once all their properties have been set by a BeanFactory: e.g. to perform custom initialization, or merely to check that all mandatory properties have been set.

复制代码
```

该接口只包含以下方法：

```
void afterPropertiesSet() throws Exception

Invoked by the containing BeanFactory after it has set all bean properties and satisfied BeanFactoryAware, ApplicationContextAware etc.
复制代码
```

概括一下上面表达的信息：如果一个bean实现了该接口，Spring 容器初始化bean时会回调afterPropertiesSet()方法。这个接口的主要作用是让bean在初始化时可以实现一些自定义的操作。

介绍完RequestMappingHandlerMapping类后我们开始对这个类的源码进行分析。

#### 2.2.2.4 RequestMappingHandlerMapping类源码分析

既然RequestMappingHandlerMapping实现了ApplicationContextAware接口，那实例化时候肯定会执行setApplicationContext方法，我们查看其实现逻辑。

```
@Override
public final void setApplicationContext(ApplicationContext context) throws BeansException {
   if (this.applicationContext == null) {
  	this.applicationContext = context;
   }
}
复制代码
```

可以看到此方法把容器上下文赋值给applicationContext变量，因为现在是Spring MVC容器创建流程，因此此处设置的值就是Spring MVC容器 。

RequestMappingHandlerMapping也实现了InitializingBean接口，当设置完属性后肯定会回调afterPropertiesSet方法，再看afterPropertiesSet方法逻辑。

```
@Override
public void afterPropertiesSet() 
   super.afterPropertiesSet();
}
复制代码
```

上面调用了父类的afterPropertiesSet()方法，沿调用栈继续查看。

```
@Override
public void afterPropertiesSet() {
	//初始化handler函数
   initHandlerMethods();
}
复制代码
```

进入initHandlerMethods初始化方法查看逻辑。

```
protected void initHandlerMethods() {
    //1.获取容器中所有bean 的name。
    //根据detectHandlerMethodsInAncestorContexts bool变量的值判断是否获取父容器中的bean，默认为false。因此这里只获取Spring MVC容器中的bean，不去查找父容器
    String[] beanNames = (this.detectHandlerMethodsInAncestorContexts ?
         BeanFactoryUtils.beanNamesForTypeIncludingAncestors(getApplicationContext(), Object.class) :
         getApplicationContext().getBeanNamesForType(Object.class));
    //循环遍历bean
    for (String beanName : beanNames) {
	//2.判断bean是否含有@Controller或者@RequestMappin注解
        if (beanType != null && isHandler(beanType)) {
            //3.对含有注解的bean进行处理，获取handler函数信息。
              detectHandlerMethods(beanName);
      }
}
复制代码
```

上面函数分为3步。

（1）获取Spring MVC容器中的bean。

（2）找出含有含有@Controller或者@RequestMappin注解的bean。

（3）对含有注解的bean进行解析。

第1步很简单就是获取容器中所有的bean name，我们对第2、3步作分析。

查看isHandler()方法实现逻辑。

```
@Override
protected boolean isHandler(Class<?> beanType) {
   return (AnnotatedElementUtils.hasAnnotation(beanType, Controller.class) ||
         AnnotatedElementUtils.hasAnnotation(beanType, RequestMapping.class));
}
复制代码
```

上面逻辑很简单，就是判断该bean是否有@Controller或@RequestMapping注解，然后返回判断结果。

如果含有这两个注解之一就进入detectHandlerMethods（）方法进行处理。

查看detectHandlerMethods（）方法。

```
protected void detectHandlerMethods(final Object handler) {
    //1.获取bean的类信息
    Class<?> handlerType = (handler instanceof String ?
         getApplicationContext().getType((String) handler) : handler.getClass());
    final Class<?> userType = ClassUtils.getUserClass(handlerType);
    //2.遍历函数获取有@RequestMapping注解的函数信息
   Map<Method, T> methods = MethodIntrospector.selectMethods(userType,
         new MethodIntrospector.MetadataLookup<T>() {
            @Override
            public T inspect(Method method) {
               try {
                //如果有@RequestMapping注解，则获取函数映射信息
                return getMappingForMethod(method, userType);
               }
         });
    //3.遍历映射函数列表，注册handler
    for (Map.Entry<Method, T> entry : methods.entrySet()) {
      Method invocableMethod = AopUtils.selectInvocableMethod(entry.getKey(), userType);
      T mapping = entry.getValue();
      //注册handler函数
      registerHandlerMethod(handler, invocableMethod, mapping);
   }
}
复制代码
```

上面方法中用了几个回调，可能看起来比较复杂，其主要功能就是获取该bean和父接口中所有用@RequestMapping注解的函数信息，并把这些保存到methodMap变量中。

我们对上面方法进行逐步分析，看看如何对有@RequestMapping注解的函数进行解析。

先进入selectMethods()方法查看实现逻辑。

```
public static <T> Map<Method, T> selectMethods(Class<?> targetType, final MetadataLookup<T> metadataLookup) {
   final Map<Method, T> methodMap = new LinkedHashMap<Method, T>();
   Set<Class<?>> handlerTypes = new LinkedHashSet<Class<?>>();
   Class<?> specificHandlerType = null;
    //把自身类添加到handlerTypes中
    if (!Proxy.isProxyClass(targetType)) {
        handlerTypes.add(targetType);
        specificHandlerType = targetType;
    }
    //获取该bean所有的接口，并添加到handlerTypes中
    handlerTypes.addAll(Arrays.asList(targetType.getInterfaces()));
    //对自己及所有实现接口类进行遍历
   for (Class<?> currentHandlerType : handlerTypes) {
      final Class<?> targetClass = (specificHandlerType != null ? specificHandlerType : currentHandlerType);
      //获取函数映射信息
      ReflectionUtils.doWithMethods(currentHandlerType, new ReflectionUtils.MethodCallback() {
	    //循环获取类中的每个函数，通过回调处理
            @Override
            public void doWith(Method method) {
            //对类中的每个函数进行处理
            Method specificMethod = ClassUtils.getMostSpecificMethod(method, targetClass);
            //回调inspect（）方法给个函数生成RequestMappingInfo  
            T result = metadataLookup.inspect(specificMethod);
            if (result != null) {
                //将生成的RequestMappingInfo保存到methodMap中
                methodMap.put(specificMethod, result);
            }
         }
      }, ReflectionUtils.USER_DECLARED_METHODS);
   }
    //返回保存函数映射信息后的methodMap
    return methodMap;
}
复制代码
```

上面逻辑中doWith()回调了inspect(),inspect()又回调了getMappingForMethod（）方法。

我们看看getMappingForMethod()是如何生成函数信息的。

```
protected RequestMappingInfo getMappingForMethod(Method method, Class<?> handlerType) {
    //创建函数信息
    RequestMappingInfo info = createRequestMappingInfo(method);
    return info;
}
复制代码
```

查看createRequestMappingInfo()方法。

```
private RequestMappingInfo createRequestMappingInfo(AnnotatedElement element) {
    //如果该函数含有@RequestMapping注解,则根据其注解信息生成RequestMapping实例，
    //如果该函数没有@RequestMapping注解则返回空
    RequestMapping requestMapping = AnnotatedElementUtils.findMergedAnnotation(element, RequestMapping.class);
    //如果requestMapping不为空，则生成函数信息MAP后返回
    return (requestMapping != null ? createRequestMappingInfo(requestMapping, condition) : null);
}
复制代码
```

看看createRequestMappingInfo是如何实现的。

```
protected RequestMappingInfo createRequestMappingInfo(
      RequestMapping requestMapping, RequestCondition<?> customCondition) {
         return RequestMappingInfo
         .paths(resolveEmbeddedValuesInPatterns(requestMapping.path()))
         .methods(requestMapping.method())
         .params(requestMapping.params())
         .headers(requestMapping.headers())
         .consumes(requestMapping.consumes())
         .produces(requestMapping.produces())
         .mappingName(requestMapping.name())
         .customCondition(customCondition)
         .options(this.config)
         .build();
}
复制代码
```

可以看到上面把RequestMapping注解中的信息都放到一个RequestMappingInfo实例中后返回。

当生成含有@RequestMapping注解的函数映射信息后，最后一步是调用registerHandlerMethod 注册handler和处理函数映射关系。

```
protected void registerHandlerMethod(Object handler, Method method, T mapping) {
   this.mappingRegistry.register(mapping, handler, method);
}
复制代码
```

看到把所有的handler方法都注册到了mappingRegistry这个变量中。

到此就把RequestMappingHandlerMapping bean的实例化流程就分析完了。

### 2.3 获取RequestMapping bean

这里我们回到Spring MVC容器初始化流程，查看initWebApplicationContext方法。

```
protected WebApplicationContext initWebApplicationContext() {
    //1.获得rootWebApplicationContext
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;
    if (wac == null) {
        //2.创建 Spring 容器 
        wac = createWebApplicationContext(rootContext);
    }
    //3.初始化容器
    onRefresh(wac)；
    return wac;
}
复制代码
```

前两步我们在[Spring 系列（二）：Spring MVC的父子容器](https://juejin.im/post/5cbc10b46fb9a0689f4c2c22)一文中分析过，主要是创建Spring MVC容器，这里我们重点看第3步。

进入onRefresh()方法。

```
@Override
protected void onRefresh(ApplicationContext context) {
    //执行初始化策略 
    initStrategies(context);
}
复制代码
```

进入initStrategies方法，该方法进行了很多初始化行为，为减少干扰我们只过滤出与本文相关内容。

```
protected void initStrategies(ApplicationContext context) {
   //初始化HandlerMapping
   initHandlerMappings(context);
}
复制代码
```

进入initHandlerMappings()方法。

```
private void initHandlerMappings(ApplicationContext context) {
    //容器中查找name为"ANDLER_MAPPING_BEAN_NAME"的实例
    HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
    //把找到的bean放到hanlderMappings中。
    this.handlerMappings = Collections.singletonList(hm);
}
复制代码
```

此处我们看到从容器中获取了name为 “HANDLER_MAPPING_BEAN_NAME”的bean，这个bean大家应该还记得吧，就是前面注册并实例化了的RequestMappingHandlerMapping bean。

### 2.4 接收请求

DispatchServlet继承自Servlet，那所有的请求都会在service()方法中进行处理。

查看service()方法。

```
@Override
protected void service(HttpServletRequest request, HttpServletResponse response) {
    //获取请求方法
    HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
    //若是patch请求执行此逻辑
    if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
        processRequest(request, response);
   }
    //其它请求走此逻辑
   else {
      super.service(request, response);
   }
}
复制代码
```

我们以get、post请求举例分析。查看父类service方法实现。

```
protected void service(HttpServletRequest req, HttpServletResponse resp){
    String method = req.getMethod();
    if (method.equals(METHOD_GET)) {
        //处理get请求
        doGet(req, resp);
    } else if (method.equals(METHOD_POST)) {
        //处理post请求
        doPost(req, resp)
    }
} 
复制代码
```

查看doGet()、doPost()方法实现。

```
@Override
protected final void doGet(HttpServletRequest request, HttpServletResponse response){
    processRequest(request, response);
}
@Override
protected final void doPost(HttpServletRequest request, HttpServletResponse response {
    processRequest(request, response);
}
复制代码
```

可以看到都调用了processRequest（）方法，继续跟踪。

```
protected final void processRequest(HttpServletRequest request, HttpServletResponse response){
    //处理请求
    doService(request, response);
}
复制代码
```

查看doService()方法。

```
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) {
    //处理请求
    doDispatch(request, response);
}
复制代码
```

### 2.5 获取handler

最终所有的web请求都由doDispatch()方法进行处理，查看其逻辑。

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) {
    HttpServletRequest processedRequest = request;
    // 根据请求获得真正处理的handler
    mappedHandler = getHandler(processedRequest);
    //用得到的handler处理请求，此处省略
	。。。。
}
复制代码
```

查看getHandler()。

```
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    //获取HandlerMapping实例
    for (HandlerMapping hm : this.handlerMappings) {
        //得到处理请求的handler
        HandlerExecutionChain handler = hm.getHandler(request);
        if (handler != null) {
            return handler;
        }
   }
   return null;
}
复制代码
```

这里遍历handlerMappings获得所有HandlerMapping实例，还记得handlerMappings变量吧，这就是前面initHandlerMappings()方法中设置进去的值。

可以看到接下来调了用HandlerMapping实例的getHanlder()方法查找handler，看其实现逻辑。

```
@Override
public final HandlerExecutionChain getHandler(HttpServletRequest request) {
    Object handler = getHandlerInternal(request);
}
复制代码
```

进入getHandlerInternal()方法。

```
@Override
protected HandlerMethod getHandlerInternal(HttpServletRequest request) {
    //获取函数url
    String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
    //查找HandlerMethod 
    handlerMethod = lookupHandlerMethod(lookupPath, request);
}
复制代码
```

进入lookupHandlerMethod()。

```
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) {
    this.mappingRegistry.getMappingsByUrl(lookupPath);
}
复制代码
```

可以看到上面方法中从mappingRegistry获取handler，这个mappingRegistry的值还记得是从哪里来的吗？

就是前面RequestMappingHandlerMapping 实例化过程的最后一步调用registerHandlerMethod()函数时设置进去的。

### 2.6 handler处理请求

获取到相应的handler后剩下的事情就是进行业务逻辑。处理后返回结果，这里基本也没什么好说的。

到此整个@RequestMapping的流程也分析完毕。

## 3.小结

认真读完上面深入分析@RequestMapping注解流程的同学，相信此时肯定对Spring MVC有了更深一步的认识。

现在回到文章开头的那个问题，为什么把<context:component-scan base-package="xx.xx.xx"/>挪到spring.xml文件中后就会404了呢？

我想看明白此文章的同学肯定已经知道答案了。

答案是：

当把<context:component-scan base-package="xx.xx.xx"/>写到spring.xml中时，所有的bean其实都实例化在了Spring父容器中。

但是在@ReqestMapping解析过程中，initHandlerMethods()函数只是对Spring MVC 容器中的bean进行处理的，并没有去查找父容器的bean。因此不会对父容器中含有@RequestMapping注解的函数进行处理，更不会生成相应的handler。

所以当请求过来时找不到处理的handler，导致404。

## 4.尾声

从上面的分析中，我们知道要使用@RequestMapping注解，必须得把含有@RequestMapping的bean定义到spring-mvc.xml中。

这里也给大家个建议：

因为@RequestMapping一般会和@Controller搭配使。为了防止重复注册bean，建议在spring-mvc.xml配置文件中只扫描含有Controller bean的包，其它的共用bean的注册定义到spring.xml文件中。写法如下：

spring-mvc.xml

```
<!-- 只扫描@Controller注解 -->
<context:component-scan base-package="com.xxx.controller" use-default-filters="false"
 >
    <context:include-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
复制代码
```

spring.xml

```
<!-- 配置扫描注解,不扫描@Controller注解 -->
<context:component-scan base-package="com.xxx">
    <context:exclude-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
复制代码
```

use-default-filters属性默认为true，会扫描所有注解类型的bean 。如果配置成false，就只扫描白名单中定义的bean注解。





# Spring 系列（四）：我们来聊聊<context:component-scan/>

## 1.背景

[上篇](https://juejin.im/post/5cbeadb96fb9a031ff0d18b5)最后给大家了一个建议，建议配置bean扫描包时使用如下写法：

spring-mvc.xml

```
<!-- 只扫描@Controller注解 -->
<context:component-scan base-package="com.xxx.controller" use-default-filters="false"
 >
    <context:include-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
复制代码
```

spring.xml

```
<!-- 配置扫描注解,不扫描@Controller注解 -->
<context:component-scan base-package="com.xxx">
    <context:exclude-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
复制代码
```

文中提到通过以上配置，就可以在Spring MVC容器中只注册有@Controller注解的bean,Spring容器注册除了@Controller的其它bean。

有的同学留言问为什么这样写就达到这种效果了呢？

也有人可能认为我是无脑从网上抄来的，我有什么依据，凭什么这么说？经过ISO 9000认证了吗？

为了维护文章的权威性以及我的脸面，本篇我就继续带大家从官网和源码两方面进行分析。

## 2\. < context:component-scan/>流程分析

### 2.1 Java注解

不是说好的讲< context:component-scan>吗，怎么注解乱入了。

放心，虽然看源码累，写让大家看懂的文章更累，但是我还没疯。

为什么讲注解，因为Spring中很多地方用到注解，本文及前几篇文章大家或多或少也都有看到。

因此在这里加个小灶，和大家一起回顾一下注解的知识点。

先查看[官方文档](https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html)：

```
Annotations, a form of metadata, provide data about a program that is not part of the program itself. Annotations have no direct effect on the operation of the code they annotate.
Annotations have a number of uses, among them:
*  Information for the compiler — Annotations can be used by the compiler to detect errors or suppress warnings.
*  Compile-time and deployment-time processing — Software tools can process annotation information to generate code, XML files, and so forth.
*  Runtime processing — Some annotations are available to be examined at runtime.
复制代码
```

上面一段话翻译过来：

```
注解是原数据的一种形式，对标注的代码逻辑上没有直接的影响，只是用来提供程序的一些信息。
主要用处如下：
*  为编译器提供信息，比如错误检测或者警告提示。
*  在编译和部署期处理期，程序可以根据注解信息生成代码、xml文件。
*  在程序运行期用来做一些检查。
复制代码
```

### 2.2 Java元注解

JAVA为了开发者能够灵活定义自己的注解，因此在java.lang.annotation包中提供了4种元注解，用来注解其它注解。

查看[官方文档](https://docs.oracle.com/javase/tutorial/java/annotations/predefined.html)对这4种元注解的介绍：

*   1.  @Retention

```
@Retention annotation specifies how the marked annotation is stored:
*   RetentionPolicy.SOURCE – The marked annotation is retained only in the source level and is ignored by the compiler.
*   RetentionPolicy.CLASS – The marked annotation is retained by the compiler at compile time, but is ignored by the Java Virtual Machine (JVM).
*   RetentionPolicy.RUNTIME – The marked annotation is retained by the JVM so it can be used by the runtime environment.
复制代码
```

翻译：指定标记的注解存储范围。可选范围是原文件、class文件、运行期。

*   1.  @Documented

```
@Documented annotation indicates that whenever the specified annotation is used those elements should be documented using the Javadoc tool. (By default, annotations are not included in Javadoc.) For more information, see the Javadoc tools page.
复制代码
```

翻译：因为注解默认是不会被JavaDoc工具处理的，因此@Documented用来要求注解能被JavaDoc工具处理并生成到API文档中 。

*   1.  @Target

```
@Target annotation marks another annotation to restrict what kind of Java elements the annotation can be applied to. A target annotation specifies one of the following element types as its value:
*   ElementType.ANNOTATION_TYPE can be applied to an annotation type.
*   ElementType.CONSTRUCTOR can be applied to a constructor.
*   ElementType.FIELD can be applied to a field or property.
*   ElementType.LOCAL_VARIABLE can be applied to a local variable.
*   ElementType.METHOD can be applied to a method-level annotation.
*   ElementType.PACKAGE can be applied to a package declaration.
*   ElementType.PARAMETER can be applied to the parameters of a method.
*   ElementType.TYPE can be applied to any element of a class.

复制代码
```

翻译：用来标识注解的应用范围。可选的范围是注解、构造函数、类属性、局部变量、包、参数、类的任意元素。

*   1.  @Inherited

```
 @Inherited annotation indicates that the annotation type can be inherited from the super class. (This is not true by default.) When the user queries the annotation type and the class has no annotation for this type, the class' superclass is queried for the annotation type. This annotation applies only to class declarations.
复制代码
```

翻译：默认情况下注解不会被子类继承，被@Inherited标示的注解可以被子类继承。

上面就是对4种元注解的介绍，其实大部分同学都知道，这里只是一起做个回顾，接下来进入正体。

### 2.3 @Controller介绍

查看[官方文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Controller.html)

```
Indicates that an annotated class is a "Controller" (e.g. a web controller).
This annotation serves as a specialization of @Component, allowing for implementation classes to be autodetected through classpath scanning. It is typically used in combination with annotated handler methods based on the RequestMapping annotation.
复制代码
```

翻译一下： @Controller注解用来标明一个类是Controller，使用该注解的类可以在扫描过程中被检测到。通常@Controller和@RequestMapping注解一起使用来创建handler函数。

我们在来看看源码，在org.springframework.stereotype包下找到Controller类。

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
   String value() default "";
}
复制代码
```

可以看到Controller声明为注解类型，类上的@Target({ElementType.TYPE}) 注解表明@Controller可以用到任意元素上，@Retention(RetentionPolicy.RUNTIME)表明注解可以保存到运行期，@Documented表明注解可以被生成到API文档里。

除定义的几个元注解外我们还看到有个@Component注解，这个注解是干什么的呢？

查看[官方文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)

```
Indicates that an annotated class is a "component". Such classes are considered as candidates for auto-detection when using annotation-based configuration and classpath scanning.
复制代码
```

翻译一下：被@Component注解标注的类代表该类为一个component，被标注的类可以在包扫描过程中被检测到。

再看源码：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Component {
   String value() default "";
}
复制代码
```

可以看到@Component注解可以用在任意类型上，保留在运行期，能生成到API文档中。

再回到@Controller注解，正是因为@Controller被@Component标注，因此被@Controller标注的类也能在类扫描的过程中被发现并注册。

另外Spring中还用@Service和@Repositor注解定义bean，@Service用来声明service类，@Repository用来声明DAO累。

其源码如下：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
   String value() default "";
}
复制代码
```

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
   String value() default "";
}
复制代码
```

### 2.4 源码剖析

铺垫都结束了，现在开始重头戏。

和< annotation-driven/>元素一样， < component-scan/>也属于自定义命名空间，对应的解析器是ComponentScanBeanDefinitionParser。

自定义命名空间的解析过程可以参考[上篇](https://juejin.im/post/5cbeadb96fb9a031ff0d18b5)，此处不再介绍。

我们进入ComponentScanBeanDefinitionParser类的parse()方法。

```
@Override
public BeanDefinition parse(Element element, ParserContext parserContext) {
    //此处 BASE_PACKAGE_ATTRIBUTE = "base-package";
    //1.获取要扫描的包
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
    //此处CONFIG_LOCATION_DELIMITERS = ",; \t\n"，
    //把,或者;分割符分割的包放到数组里面
   String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
         ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);
    //2.创建扫描器
   ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
    //3.扫描包并注册bean
   Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
   return null;
}
复制代码
```

上面扫描注册过程可以分为3步。

（1）获取要扫描的包。

（2）创建扫描器。

（3）扫描包并注册bean。

第1步逻辑比较简单，就是单纯的读取配置文件的"base-package"属性得到要扫描的包列表。

我们从第2步开始分析。

#### 2.4.1 创建扫描器

进入configureScanner方法()。

```
protected ClassPathBeanDefinitionScanner configureScanner(ParserContext parserContext, Element element) {
    //useDefaultFilters默认为true，即扫描所有类型bean
    boolean useDefaultFilters = true;
    //1.此处USE_DEFAULT_FILTERS_ATTRIBUTE = "use-default-filters"，获取其XML中设置的值
    if (element.hasAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE)) {
        useDefaultFilters = Boolean.valueOf(element.getAttribute(USE_DEFAULT_FILTERS_ATTRIBUTE));
    }
   //2.创建扫描器
    ClassPathBeanDefinitionScanner scanner = createScanner(parserContext.getReaderContext(), useDefaultFilters);
    //3.解析过滤类型
    parseTypeFilters(element, scanner, parserContext);
    //4.返回扫描器
    return scanner;
}
复制代码
```

创建扫描器的方法分为4步。

（1）获取扫描类范围。

（2）根据扫描范围初始化扫描器。

（3）设置扫描类的过滤器。

（4）返回创建的扫描器。

第1步也比较简单，从配置文件中获得“use-default-filters”属性的值，默认是true，即扫描所有类型的注解。

我们进入第2步的createScanner（）方法，看看如何创建扫描器。

```
protected ClassPathBeanDefinitionScanner createScanner(XmlReaderContext readerContext, boolean useDefaultFilters) {
    //新建一个扫描器
    return new ClassPathBeanDefinitionScanner(readerContext.getRegistry(), useDefaultFilters,
        readerContext.getEnvironment(),readerContext.getResourceLoader());
}
复制代码
```

沿调用栈进入ClassPathBeanDefinitionScanner()方法。

```
public ClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry, boolean useDefaultFilters,
//如果useDefaultFilters为true，注册默认过滤器
    if (useDefaultFilters) {
        //注册默认过滤器
        registerDefaultFilters();
   }
}
复制代码
```

进入registerDefaultFilters()方法。

```
protected void registerDefaultFilters() {
    this.includeFilters.add(new AnnotationTypeFilter(Component.class));
}
复制代码
```

可以看到上面方法把Component注解类型加入到了扫描白名单中，因此被@Component标注的类都会被扫描注册。

在此，大家也明白为什么@Controller、@service、@Repository标注的类会被注册了吧，因为这些注解都用@Component标注了。

我们再进入第3步的parseTypeFilters()方法，看如何设置过滤器。

```
protected void parseTypeFilters(Element element, ClassPathBeanDefinitionScanner scanner, ParserContext parserContext) {
    //解析exclude-filter和include-filter元素
    //获取元素所有子节点
    NodeList nodeList = element.getChildNodes();
    //遍历元素子节点
    for (int i = 0; i < nodeList.getLength(); i++) {
        Node node = nodeList.item(i);
        if (node.getNodeType() == Node.ELEMENT_NODE) {
        String localName = parserContext.getDelegate().getLocalName(node);
        //解析include-filter元素 ,此处 INCLUDE_FILTER_ELEMENT = "include-filter"
        if (INCLUDE_FILTER_ELEMENT.equals(localName)) {
            //创建类型过滤器
            TypeFilter typeFilter = createTypeFilter((Element) node, classLoader, parserContext);
            //把解析出来的类型加入白名单
            scanner.addIncludeFilter(typeFilter);
        }
        //解析exclude-filter元素，此处EXCLUDE_FILTER_ELEMENT = "exclude-filter"
        else if (EXCLUDE_FILTER_ELEMENT.equals(localName)) {
            //创建类型过滤器
            TypeFilter typeFilter = createTypeFilter((Element) node, classLoader, parserContext);
            //把解析出来的类型加入黑名单
            scanner.addExcludeFilter(typeFilter);
        }
    }
}
复制代码
```

进入createTypeFilter()方法查看实现逻辑。

```
protected TypeFilter createTypeFilter(Element element, ClassLoader classLoader, ParserContext parserContext) {
    //获取xml中type属性值，此处FILTER_TYPE_ATTRIBUTE = "type"   
    String filterType = element.getAttribute(FILTER_TYPE_ATTRIBUTE);
    //获取xml中expression属性值，此处FILTER_EXPRESSION_ATTRIBUTE = "expression"，获取xml中该属性值
    String expression = element.getAttribute(FILTER_EXPRESSION_ATTRIBUTE);
    expression = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(expression);
    //如果是注解类型，创建注解类型过滤器，并把需要过滤的注解类设置进去
    if ("annotation".equals(filterType)) {
        return new AnnotationTypeFilter((Class<Annotation>) ClassUtils.forName(expression, classLoader));
    }
}
复制代码
```

上面就是创建扫描器的过程，主要是将XML文件中设置的类型添加到白名单和黑名单中。

#### 2.4.2 扫描注册bean

得到扫描器后，开始扫描注册流程。

进入doScan()方法。

```
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
    Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<BeanDefinitionHolder>();
    //遍历所有需要扫描的包
    for (String basePackage : basePackages) {
        //1.在该包中找出用@Component注解的类，放到候选列表中
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
        for (BeanDefinition candidate : candidates) {
        //2.判断容器中是否已经有bean信息，如果没有就注册
        if (checkCandidate(beanName, candidate)) {
            //生成bean信息
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            //添加bean信息到bean定义列表中
            beanDefinitions.add(definitionHolder);
            //3.把bean注册到IOC容器中
            registerBeanDefinition(definitionHolder, this.registry);
        }
    }
}
复制代码
```

扫描注册过程分为3步。

（1）从包中找出需要注册的bean并放到候选列表中。

（2）遍历候选列表中的所有bean，判断容器中是否已经存在bean。

（3）如果不存在bean，就把bean信息注册到容器中。

接下来依次分析上面扫描注册流程。

##### 2.4.2.1 查找候选bean

我们先看第1步，查找候选bean的过程。进入findCandidateComponents()方法。

```
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
    Set<BeanDefinition> candidates = new LinkedHashSet<BeanDefinition>();
    //1.获取包的classpath
    String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
        resolveBasePackage(basePackage) + '/' + this.resourcePattern;
    //2.把包下的所有class解析成resource资源   
    Resource[] resources = this.resourcePatternResolver.getResources(packageSearchPath);
    //遍历所有类resource
    for (Resource resource : resources) {
        if (resource.isReadable()) {
            //3.获取类的元信息
            MetadataReader metadataReader = this.metadataReaderFactory.getMetadataReader(resource);
            //4.判断是否候选component
            if (isCandidateComponent(metadataReader)) {
                //5.根据类元信息生成beanDefinition
                ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                sbd.setResource(resource);
                sbd.setSource(resource);
                //6.判断该bean是否能实例化
                if (isCandidateComponent(sbd)) {
                    //7.加入候选类列表
                    candidates.add(sbd);
                 }
    //8.返回候选components选列表
    return candidates;
}
复制代码
```

查找bean的流程比较繁琐，可以分为以下8步。

（1）获取包扫描路径。

（2）把包路径下的所有类解析成resource类。

（3）解析resource类，获取类的元信息。

（4）根据类元信息判断该类是否在白名单中。

（5）如果在白名单中，生成beanDefinition信息。

（6）根据beanDefinition信息判断类是否能实例化。

（7）如果可以实例化，将beanDefinition信息加入到候选列表中。

（8）返回保存beanDefinition信息的候选列表。

还记得BeanDefinition是什么吧，主要是保存bean的信息。如果不记得看看[Spring注册流程](https://juejin.im/post/5cb89dae6fb9a0686b47306d)。

因为其它逻辑比较简单，在此我们重点分析第4步和第6步。

先看第4步，进入isCandidateComponent()方法。

```
protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
    //1.遍历黑名单，若传入的类元信息在黑名单中返回false
    for (TypeFilter tf : this.excludeFilters) {
        //判断是否和传入的类匹配
        if (tf.match(metadataReader, this.metadataReaderFactory)) {
            return false;
        }
    }
    //2.遍历白名单，若传入的类元信息在白名单中返回true
    for (TypeFilter tf : this.includeFilters) {
        if (tf.match(metadataReader, this.metadataReaderFactory)) {
            //根据@Conditional注解判断是否注册bean，如果没有@Conditional注解，返回true.
            return isConditionMatch(metadataReader);
        }
    }
    return false;
}
复制代码
```

可以看到上面主要逻辑是判断该类是否在白名单或黑名单列表中，如果在白名单，则返回true，在黑名单返回false。黑、白名单的值就是创建扫描流程中通过parseTypeFilters()方法设置进去的。

再稍微提一下上面@Conditional注解，此注解是Spring 4中加入的，作用是根据设置的条件来判断要不要注册bean，如果没有标注该注解，默认注册。我们在这里不展开细说，有兴趣的同学可以自己查阅相关资料。

我们再看第6步，进入isCandidateComponent()方法。

```
protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
    //获取元类信息
    AnnotationMetadata metadata = beanDefinition.getMetadata();
    //判断是否可以实例化
    return (metadata.isIndependent() && (metadata.isConcrete() ||
        (metadata.isAbstract() && metadata.hasAnnotatedMethods(Lookup.class.getName()))));
}
复制代码
```

可以看到上面是根据该类是不是接口、抽象类、嵌套类等信息来判断能否实例化的。

##### 2.4.2.2 判断bean是否已经注册

候选bean列表信息已经得到，再看看如何对列表中的bean做进一步判断。

进入checkCandiates()方法。

```
protected boolean checkCandidate(String beanName, BeanDefinition beanDefinition) {
    if (!this.registry.containsBeanDefinition(beanName)) {
        return true;
   }
    return false;
}
复制代码
```

上面方法比较简单，主要是查看容器中是否已经有bean的定义信息。

##### 2.4.2.3 注册bean

对bean信息判断完成后，如果bean有效，就开始注册bean。

进入registerBeanDefinition()方法。

```
protected void registerBeanDefinition(BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry) {
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, registry);
}
复制代码
```

再进入registerBeanDefinition()方法。

```
public static void registerBeanDefinition(
    BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry) {
    //得到beanname
    String beanName = definitionHolder.getBeanName();
    //注册bean信息
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
    //注册bean的别名
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
    for (String alias : aliases) {
        registry.registerAlias(beanName, alias);
    }

}
复制代码
```

上面流程大家有没有似曾相识，和[Spring解析注册流程](https://juejin.im/post/5cb89dae6fb9a0686b47306d)文中注册bean的逻辑一样。

到此就完成了扫描注册bean流程的分析。接下来就是bean的实例化等流程，大家可以参考[Spring解析注册流程](https://juejin.im/post/5cb89dae6fb9a0686b47306d)一文。

## 3.小结

看完上面的分析，相信大家对< context:component-scan/>有了深入的了解。

现在回到开头的那段代码。会不会有“诚不我欺也”的感觉。

最后，我再把那段代码贴出来，大家对着代码在脑海里想象一下其解析流程，检验一下掌握程度。

如果有哪一步卡住了，建议再回头看看我的文章，直至能在脑海中有一个完整的流程图，甚至能想到对应的源代码段。

如果能做到这样，说明你真正理解了< context:component-scan/>，接下来就可以愉快的和小伙伴炫技或者和面试官去侃大山了。

spring-mvc.xml

```
<!-- 只扫描@Controller注解 -->
<context:component-scan base-package="com.xxx.controller" use-default-filters="false"
 >
    <context:include-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
复制代码
```

spring.xml

```
<!-- 配置扫描注解,不扫描@Controller注解 -->
<context:component-scan base-package="com.xxx">
    <context:exclude-filter type="annotation"
        expression="org.springframework.stereotype.Controller" />
</context:component-scan>
复制代码
```

本文完。