---
flag: blue
---
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