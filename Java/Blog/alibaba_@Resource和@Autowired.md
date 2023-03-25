# Spring探索丨既生@Resource，何生@Autowired？

提到 Spring 依赖注入，大家最先想到应该是 @Resource 和 @Autowired，很多文章只是讲解了功能上的区别，对于Spring为什么要支持两个这么类似的注解却未提到，属于知其然而不知其所以然。不知大家在使用这两个注解的时候有没有想过，@Resource 又支持名字又支持类型，还要 @Autowired 干嘛，难道是 Spring 官方没事做了？

真的是没事做了吗？读了本文你将会了解到：

1. @Resource和@Autowired来源
2. Spring官方为什么会支持这两个功能如此相似的注解？
3. 为什么@Autowired属性注入的时候Idea会曝出黄色的警告？
4. @Resource和@Autowired推荐用法

### 一、来源

既然要弄清楚，就要先了解他们的身世。

@Resource 于 2006年5月11日随着JSR 250 发布 ，官方解释是：

> Resource 注释标记了应用程序需要的资源。该注解可以应用于应用程序组件类，或组件类的字段或方法。当注解应用于字段或方法时，容器将在组件初始化时将所请求资源的实例注入到应用程序组件中。如果注释应用于组件类，则注释声明应用程序将在运行时查找的资源。

可以看到它类似一个定义，而由其他的组件或框架自由实现。

@Autowired 于 2007年11月19日随着Spring2.5发布，同时官方也对@Resource进行了支持。@Autowired的官方解释是：

> 将构造函数、字段、设置方法或配置方法标记为由 Spring 的依赖注入工具自动装配。

可以看到，@Autowired 是 Spring 的亲儿子，而 @Resource 是 Spring 对它定义的一种实现，它们的功能如此相似。那么为什么要支持了 @Resource，又要自己搞个 @Autowired呢？

对此专门查了一下Spring2.5的官方文档，文档中有一段这么说到：

> However, Spring 2.5 dramatically changes the landscape. As described above, the autowiring choices have now been extended with support for the JSR-250 **@Resource** annotation to enable autowiring of named resources on a per-method or per-field basis. However, the **@Resource** annotation alone does have some limitations. Spring 2.5 therefore introduces an **@Autowired** annotation to further increase the level of control.

大概的意思是说，Spring2.5 支持注解自动装配啦，现已经支持JSR-250 @Resource 基于每个方法或每个字段的命名资源的自动装配，但是只有 @Resource是不行的，我们还推出了“粒度”更大的@Autowired，来覆盖更多场景了。

嗯哼，那么官方说的“粒度”就是关键了，那“粒度”指的是什么呢”？

## 二、既生“@Resource”，何生“@Autowired”

要想找到粒度是什么，我们先从两个注解的功能下手

@Autowired：类型注入

@Resource：名字注入优先，找不到名字找类型

论功能的“粒度”，@Resource 已经包含 @Autowired 了啊，“粒度”更大啊，难道是Spring2.5的时候还不是这样？我又去翻了下Spring2.5文档，上面明确的写到：

> When using **@Resource** without an explicitly provided name, if no Spring-managed object is found for the default name, the injection mechanism will fallback to a type-match.

这不是和现在一样的吗，我此时凌乱了。那么“粒度”到底指的是什么？在混迹众多论坛后，其中stackoverflow的一段话引起了我的注意：

> Both **@Autowired** and **@Resource** work equally well. But there is a conceptual difference or a difference in the meaning.
>
> - **@Resource** means get me **a known resource by name**. The name is extracted from the name of the annotated setter or field, or it is taken from the name-Parameter.
> - **@Inject** or  **@Autowired** try to wire in **a suitable other component by type**.
>
> So, basically these are two quite distinct concepts. Unfortunately the Spring-Implementation of **@Resource** has a built-in fallback, which kicks in when resolution by-name fails. In this case, it falls back to the **@Autowired**-kind resolution by-type. While this fallback is convenient, IMHO it causes a lot of confusion, because people are.

大概的意思是：Spring虽然实现了两个功能类似的，但是存在概念上的差异或含义上的差异：

- @Resource 这按名称给我一个确定已知的资源。
- @Autowired 尝试按类型连接合适的其他组件。

但是 @Resource 当按名称解析失败时会启动。在这种情况下，它会按类型解析，引起概念上的混乱，因为开发者没有意识到概念上的差异，而是倾向于使用 @Resource基于类型的自动装配。

原来 Spring 官方说的“粒度”是指“资源范围”，**@Resource 找寻的是确定的已知的资源，相当于给你一个坐标，你直接去找。@Autowired 是在一片区域里面尝试搜索合适的资源。**

所以上面的问题答案已经基本明确了。

Spring 为什么会支持两个功能相似的注解呢？

- 它们的概念不同，@Resource 更倾向于找已知资源，而 Autowired 倾向于尝试按类型搜索资源。
- 方便其他框架迁移，@Resource 是一种规范，只要符合JSR-250规范的其他框架，Spring 就可以兼容。

既然 @Resource 更倾向于找已知资源，为什么也有按类型注入的功能？

- 个人猜测：可能是为了兼容从 Spring 切换到其他框架，开发者就算只使用Resource 也是保持 Spring 强大的依赖注入功能。

## 三、Spring 的区别对待

看到这相信大家对使用 @Resource还是@Autowired有了自己的见解。在日常写代码中有个小细节不知道大家有没有注意到，**使用 @Autowired在属性上的时候Idea会曝出黄色的警告，并且推荐我们使用构造方法注入**，而Resource就不会，这是为什么呢？警告如下：

![图片](alibaba_@Resource%E5%92%8C@Autowired.resource/640-20230323202122728.png)

#### 为什么@Autowired在属性上的时候Idea会曝出黄色的警告，并且推荐我们使用构造方法注入？

其实Spring文档中已经给出了答案，主要有这几点：

**1、声明不了常量的属性**

基于属性的依赖注入不适用于声明为 final 的字段，因为此字段必须在类实例化时去实例化。声明不可变依赖项的唯一方法是使用基于构造函数的依赖项注入。

**2、容易忽视类的单一原则**

一个类应该只负责软件应用程序功能的单个部分，并且它的所有服务都应该与该职责紧密结合。如果使用属性的依赖注入，在你的类中很容易有很多依赖，一切看起来都很正常。但是如果改用基于构造函数的依赖注入，随着更多的依赖被添加到你的类中，构造函数会变得越来越大，代码开始就开始出现“异味”，发出明确的信号表明有问题。具有超过十个参数的构造函数清楚地表明该类有太多的依赖，让你不得不注意该类的单一问题了。因此，属性注入虽然不直接打破单一原则，但它却可以帮你忽视单一原则。

**3、循环依赖问题**

A类通过构造函数注入需要B类的实例，B类通过构造函数注入需要A类的实例。如果你为类 A 和 B 配置 bean 以相互注入，使用构造方法就能很快发现。

**4、依赖注入强依赖Spring容器**

如果您想在容器之外使用这的类，例如用于单元测试，不得不使用 Spring 容器来实例化它，因为没有其他可能的方法（除了反射）来设置自动装配的字段。

#### 为什么@Resource没有呢？

在官方文档中，我没有找到答案，查了一些资料说是：@Autowired 是 Spring 提供的，一旦切换到别的 IoC 框架，就无法支持注入了. 而@Resource 是 JSR-250 提供的，它是 Java 标准，我们使用的 IoC 容器应该和它兼容，所以即使换了容器，它也能正常工作。

## 三、@Autowired 和 @Resource 推荐用法

### **1. 什么场景用什么合适**

记住一句话就行，@Resource倾向于**确定性的单一资源**，@Autowired**为类型去匹配符合此类型所有资源**。

如集合注入，@Resource也是可以的，但是建议使用@Autowired。idea左侧的小绿标可以看出来，不建议使用@Resource注入集合资源，本质上集合注入不是单一，也是不确定性的。

![图片](alibaba_@Resource%E5%92%8C@Autowired.resource/640.png)



### 2. @Autowired推荐用法

方法1 ：使用构造函数注入（推荐）

原生版：

![图片](alibaba_@Resource%E5%92%8C@Autowired.resource/640-20230323202122907.png)

优雅版：使用lombok的@RequiredArgsConstructor+private final

![图片](alibaba_@Resource%E5%92%8C@Autowired.resource/640-20230323202122718.png)

方法2：set注入

原生版：

![图片](alibaba_@Resource%E5%92%8C@Autowired.resource/640-20230323202122750.png)

优雅版：使用lombok的@Setter

![图片](alibaba_@Resource%E5%92%8C@Autowired.resource/640-20230323202122780.png)