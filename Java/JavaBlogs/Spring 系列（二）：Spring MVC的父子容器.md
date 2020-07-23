---
flag: blue
---
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