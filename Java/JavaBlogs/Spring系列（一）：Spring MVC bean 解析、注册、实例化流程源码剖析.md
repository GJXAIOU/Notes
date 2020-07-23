---
flag: blue
---


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