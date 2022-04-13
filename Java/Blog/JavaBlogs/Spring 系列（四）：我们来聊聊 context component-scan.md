---
flag: blue
---
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