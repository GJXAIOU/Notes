[TOC]

# 前言

EJB 规范采用的实体 Bean（Entity Bean）功能太全太笨重。绝大部分项目比较简单，因此 Spring 框架一切从实际出发，使用基于 POJO（Plain Old Java Object，简单 Java 对象）的轻量级编程模型来进行应用的快速开发。 Spring 并不是替代 EJB，而是面对不同应用场景的两种解决方案。

# 第一部分：掀起 Spring 的盖头来

## 第一章：Spring 框架的由来

### ==Spring 框架概述==

![image-20200701081509350](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200701081509350.png)

- Spring 框架构建在 Core 核心模块之上，该模块是整个框架的基础，该模块中 Spring 提供了一个 IOC 容器（IOC Container）实现，用来实现以依赖注入的方式管理对象之间的依赖关系。

- AOP 模块增强了 POJO 的能力，并补充 OOP 的不足。

- 提供了完备的**数据访问和事务管理的抽象和集成服务，**
    - 在数据访问支持方面： 对 JDBC 的 API 进行了极大的简化。同时提供了主流的 ORM 产品：Hibernate、MyBatis 等的形式统一的集成支持。
    - **Spring 的事务管理抽象层是 AOP 的最佳实践，其直接构建在 AOP 的基础之上，提供了编程式事务管理和声明式事务管理的完备支持**，从而简化了开发中的数据访问和事务管理工作。

- Spring 提供了一套自己的 Web MVC 框架。



# 第二部分：Spring 的 IOC 容器

## 第二章：IOC（Inversion of Control,控制反转） 的基本概念

#### 2.1 让别人为你服务

如果 A 依赖 B，原始方式是主动的获取依赖对象，其中最直接方式就是**在类的构造函数中新建对应的依赖类**，即主动的获取依赖对象，从而造成**被注入对象直接依赖于被依赖对象**。但是其实我们最终要做的就是直接调用依赖对象所提供的某种服务，只要用到该依赖对象时候它准备就绪即可，获取方式没有什么关系。

![image-20200613215328795](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200613215328795.png)

==但是在 IOC 中。IOC 中两者通过 IOC Service Provider ，所有的被注入对象和依赖对象都由  IOC Service Provider 统一管理==，被注入对象需要什么高度 IOC  Sp，后者就会将相应的被依赖对象注入。**IOC Service Provider 就是通常 IOC 容器所充当的角色**。**从被注入对象看，依赖关系的取得方式发生反转，控制也才从被注入对象转移到 IOC Service Provider**。

#### 2.2==通知方式==

==作为被注入对象，要想让 IOC Service Provider 为其提供服务并将需要依赖的对象注入，需要以某种方式通知对方。即依赖注入的三种方式：==

- 构造方法注入

    被注入对象可以通过在其构造方法中声明依赖对象的参数列表。IOC Service Provider 会检查被注入对象的构造方法，取得所需要的依赖对象列表，进而为其注入相应的对象。**同一个对象不可能被构造两次，所以被注入对象的构造乃至整个生命周期都有 IOC Sp 来管理**。同时该种方式对象构造完成之后就进入了就绪状态，可以马上使用。

    **优点**：对象构造完成就进入就绪状态，可以马上使用。

    **缺点**：依赖对象比较多时候，构造方法参数列表较长。在通过反射构造对象的时候，对相同类型的参数处理、维护和使用上比较麻烦。同样因为构造方法无法继承、无法设置默认值，所以对于非必须的依赖处理，可能需要引入多个构造方法，维护不便。

- Setter 方法注入

    因为 JavaBean 对象可以通过 set() 和 get() 方法来访问对象即更改或获得相应的对象属性，且 setXXX() 方法统称为：Setter 方法。所以**当前对象只要为其依赖对象所对应的属性添加 setter 方法，就可以通过 setter 方法将对应的依赖对象设置到被注入对象中**。==Setter 不能让对象构造完成之后即可使用，但是相对宽松，可以在对象构造完成之后再进行注入。==

    **优点**：该方法可以命名，比构造方法在描述性上更好，同时 setter 方法可以被继承、设置默认值。

    **缺点**：对象无法在构造完成之后就进入就绪状态。

- 接口注入（**该种方式强制被注入对象实现不必要的接口，带有侵入性**，不推荐。）

    ==被注入对象如果要 IOC Service Provider 为其注入依赖对象，就**必须实现某个接口**==，该接口提供一个方法用来为其注入依赖对象。IOC Sp 通过这些接口来了解应该为被注入对象注入什么依赖对象。

    例如：A 让 IOC SP 为其注入所依赖的 B，所以 A 实现 BCallable 接口，该接口中声明一个方法：injectB(B b)（方法名随意，但是方法参数类型是所依赖对象的类型），这样 InjectionServiceContainer 对象即对应的 IOC Sp 就可以通过这个接口方法将依赖对象注入到被注入对象 A 中。【接口名、方法名都不重要，参数类型重要】

    

## 第三章：掌握大局的 IOC Service Provider

业务对象可以通过  IOC 方式来声明响应的依赖，但是 IOC Service Provider  才能将相互依赖的对象绑定一起。**IOC Service Provider 是一个抽象出来的概念，指代任何将 IOC 场景中的业务对象绑定到一起的实现方式**。可以是代码或者类或者 IOC 容器，Spring IOC 容器就是一个提供依赖注入的 IOC SP。

```java
// IOC Sp 方式之一：代码
A a = new A();
C c = new C();
B b = new B(a,c); // 实现依赖绑定
```

#### 3.1 IOC SP 职责

==主要包括：业务对象的构建管理和业务对象间依赖绑定==。

- 业务对象的构建管理：在 IOC 场景中，业务对象无需关心所依赖的对象是如何构建取得的，该部分由 IOC Sp 完成，IOC SP 将需要的对象的构建逻辑从客户端剥离，避免了该部分逻辑污染业务对象的实现。
- 业务对象间的依赖绑定：IOC SP 通过结合之前构建和管理的所有业务对象，以及各个业务对象间可以识别的依赖关系，将这些对象所依赖的对象注入绑定，从而保证每个业务对象在使用的时候，可以处于就绪状态。

#### 3.2 IOC  SP 记录管理对象之间的依赖关系

第二章中，被注入对象可以通过多种方式**通知** IOC SP 为其注入相应的依赖，但是接收到通知的 IOC SP 需要理解被注入对象的意图并及时提供想要的依赖，即**IOC SP 同样需要知道自己所管理和掌握的被注入对象和依赖对象之间的对应关系**。

**IOC SP 记录对象之间的关系方式**：即注册对象管理信息的方式：

- 直接编码方式

    大部分 IOC 容器均支持直接编码方式，在容器启动之前，我们就可以通过程序编码的方式将被注入对象和依赖对象注册到容器中，并明确它们相互之间的依赖注入关系。

    ```java
    IocContainer container = ....;
    // 注册过程
    container.register(A.class, new A());
    container.register(B.class, new B());
    ... ...
    A a = (A)container.get(A.class);
    a.getAndPersistB();
    ```

    通过对相应的类指定对应的具体实例，可以告知 IOC 容器，当我们要这种类型的对象实例时，请将容器中注册的、对应的那个具体实例返回给我们。

- 配置文件方式

    常见的依赖注入关系管理方式，包括普通文本、properties 文件、XML 文件（最常见）等等。配置完成之后可以从读取配置文件完成对象组装的容器中获取 A 并且使用。

    ```xml
    <bean id = "a" class="..A">
        <property name = "bb">
            <ref bean = "b" />
        </property>    
    </bean>    
    
    <bean id = "b" class = "...B"/>
    ```

    ```java
    // 读取配置方式然后使用
    IOContainer container = ...;
    container.readConfigurationFile("配置文件位置");
    A aa = (A)container.getBean("a");
    aa.getAndPersistB();
    ```

- 元数据方式

    可以直接在类中使用元数据信息来标注各个对象之间的依赖关系，然后由 Google Guice 框架（一般该中方式只针对这个框架）根据这些注解所提供的信息将这些对象组装后交给客户端使用。

    ```java
    // 使用 Guice 的注解标注依赖关系后的 A 定义
    public class A{
    	private B b;
        private C c;
        @Inject // 通过该注解只能需要  IOC SP 通过构造方法注入方式来注入所依赖的对象，绑定关系在下面代码实现
        public A(B b, C c){
        	this.b = b;
            this.c = c;
        }
    }
    
    // 绑定关系，需要自定义类然后继承 AbstractModule,然后重新 configure() 方法
    public class ABindingModule extends AbstractModule{
       @Override
       protected void configure(){
       	bind(A.class).to(B.class).in(Scopes.SINGLETON);
        bind(A.class).to(B.class).in(Scopes.SINGLETON);
       } 
    }
    
    // 使用过程
    Injector injector = Guice.createInjector(new ABindingModule());
    A a = injector.getInstance(A.class);
    ```

    

## 第四章：Spring 的 IOC 容器之 BeanFactory

Spring 的 IOC 容器就是一个 IOC SP，其作为一个提供 IOC 支持的轻量级容器，除了提供基本的 IOC 支持之外，**作为轻量级容器**提供其它支持，包括对相应 AOP 框架支持。两者关系如图所示：

![image-20200614093453318](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200614093453318.png)

==Spring 提供了两种容器类型：BeanFactory 和 ApplicationContext==

- BeanFactory 是基础类型的 IOC 容器，提供完整的 IOC 服务支持，**默认采用延迟初始化策略**，即只有当客户端对象需要**访问**容器中的某个受管理的对象的时候，才对该受管理对象进行**初始化以及依赖注入操作**。因此启动速度较快。
- ApplicationContext 在 BeanFactory 基础上进行构建，相对比较高级的容器实现，拥有 BeanFactory 全部特性之外还包括：事件发布、国际化信息支持等等。**ApplicationContext 管理的对象在该类型容器启动之后，默认全部初始化并绑定完成**。所以需要的系统资源更多，其启动时间较长。

![image-20200614102853097](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200614102853097.png)

​		ApplicationContext 间接继承自 BeanFactory，即是构建在 BeanFactory 之上的 IOC 容器，其它三个接口介绍见章五。

BeanFactory 即生产 Bean 的工厂，因为 Spring 提倡使用 POJO，则会将每个业务对象看成一个 JavaBean 对象，BeanFactory 作为 Spring 提供的基本的 IOC 容器，其可以完成作为 IOC Service Provider 的所有职能，包括业务对象的注册和对象间的依赖关系。将应用所需的业务对象交给 BeanFactory 之后，然后其内部进行组装，最后可以直接从 BeanFactory 提供的接口方法中取得最终组装完成并可用的对象。

```java
public class BeanFactory{
	Object getBean(String name) throws BeansException;
    Object getBean(String name, Class requiredType) throws BeansException;
    Object getBean(String name, Object[] args) throws BeansException;
    boolean containsBean(String name);
    boolean isSingleton(String name);
    ....
}
```

#### 4.1 拥有 BeanFactory 之后

无论是否引入 BeanFactory 之类的轻量级容器，**应用的设计和开发流程实际上没有太大改变**，即针对系统和业务逻辑，该如何设计和实现当前系统不受是否引入轻量级容器的影响，**唯一不同就是对象之间的依赖关系的解决方式改变了**。即在之前我们通常在应用程序的入口类的 main 方法中自己实例化对应的对象然后调用，现在只需要将“生成图纸”给 BeanFactory 就会为我们生成一个对象，我们直接从容器中取生产好的就行。

#### 4.2 BeanFactory 的对象注册与依赖绑定方式

BeanFactory 作为一个 IOC SP，为了能够明确管理各个业务对象以及业务对象之间的依赖绑定关系，需要某种途径来记录和管理这些信息。并且 IOC SP 中三种方式也支持。

- 直接编码方式

    不应该作为一种方式，因为不管什么方式最终都需要编码才能落实所有信息并且使用。下面示例显示注册并绑定过程：

    ```java
    public static void main(String[] args) {
        // 首先构建一个 DefaultListableXX 作为 BeanDefinitionRegister
        DefaultListableBeanFactory beanRegistry = new DefaultListableBeanFactory();
        // 将其交个 binviaCode() 进行具体对象注册和相关依赖管理
        BeanFactory container = (BeanFactory)bindViaCode(beanRegistry);
        // 通过 binViaCode() 返回的 BeanFactory 取得需要的对象
        A a = (A)container.getBean("aa");
    }
    
    public static BeanFactory bindViaCode(BeanDefinitionRegistry registry){
        // 首先针对相应的业务对象构建与之相对应的 BeanDefinition
        AbstractBeanDefiniton a = new RootBeanDefinition(A.class, true);
        AbstractBeanDefiniton b = new RootBeanDefinition(B.class, true);
        AbstractBeanDefiniton c = new RootBeanDefinition(C.class, true);
    
        // 将 Bean 定义注册到容器中
        // 将上面的 BeanDefinition 注册到通过方法参数传进来的 BeanDefinitionRegister 中
        register.registerBeanDefinition("aa", a);
        register.registerBeanDefinition("bb", b);
        register.registerBeanDefinition("cc", c);
    
        // 指定依赖关系
            // 方式一：通过构造函数
        ConstructorArgumentValue argValue = new ConstructorArgumentValue();
        argValue.addIndexedArgumentValue(0, b);
        argValue.addIndexedArgumentValue(1, c);
        a.setConstructorArgumentValues(argValue);
    
            // 方式二：通过 setter 方法注入
        MutablePropertyValues propertyValues = new MutablePropertyValues();
        propertyValues.addPropertyValue(new propertyValue("bb",b));
        propertyValues.addPropertyValue(new propertyValue("cc",c));
        a.setPropertyValues(propertyValues);
    
        // 绑定完成
        return (BeanFactory)register;
    }
    ```

    **详解**：BeanFactory 只是一个接口，必须使用一个该接口的实现来进行实际的 Bean 管理，**通常使用的实现类为：DefaultListableBeanFactory**，该类除了间接实现 BeanFactory 接口，还实现了 BeanDefinitionRegister 接口，该接口才是在 BeanFactory 的实现中担当 Bean 注册管理的角色。==**基本上 BeanFactory 接口只定义了如何访问容器内管理的 Bean 的方法，各个 BeanFactory 的具体实现类负责具体 Bean 的注册一级管理工作**。**BeanDefinitionRegister 接口定义抽象了 Bean 的注册逻辑，通常具体的 BeanFactory 实现类会实现该接口来管理 Bean 的注册**。==

    ==**每一个受管理的对象，在容器内都有一个 BeanDefinition 的实例与之相对应，该实例负责保存对象的所有必要信息，包括其对应的对象的 Class 类型，是否为抽象类、构造方法参数以及其他属性**，当客户端向BeanFactory 请求相应对象的时候，BeanFactory 会通过这些信息为客户端返回一个完备可用的对象实例。BeanDefinition 的两个主要实现类为：RootBeanDefinition 和 ChildBeanDefinition。==

    > BeanDefinitionRegister 像图书馆书架，书放在书架上，虽然借书和还书和图书馆 BeanFactory 打交道，但是书架才是存放各类书的地方。

    ![image-20200614133254286](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200614133254286.png)

- 外部配置文件方式

    Spring 中的 IOC 容器支持两种配置文件格式：Properties 和 XML 格式。 IOC 容器**会根据不同的外部配置文件格式**，==给出相应的 BeanDefinitionReader 实现类，尤其相应的实现类来将配置文件内容读取并且映射到 BeanDefinition，然后将映射之后的 BeanDefinition 注册到一个 BeanDefinitionRegistry  之后，BeanDefinitionRegister 即完成 Bean 的注册和加载。==【大部分工作（包括文件格式、装配 BeanDefinition）均由 BeanDefinitionReder 的实现类完成，**BeanDefinitionRegistry 仅仅负责保管】

    - Properties 配置方式格式，通过 PropertiesBeanDefinitionReader 类用于 Properties 格式配置文件的加载，我们只要提供配置文件即可。
    - XML 配置格式的加载，使用 XSD（XML Schema Definition）约束方式来实现文档格式约束。通过 XMLBeanDefinitionReader 类读取配置文件并且解析，之后将解析后的文件内容映射到相应的 BeanDefinition，并加载到相应的 BeanDefinitionRegistry 中（这里就是 DefaultListableBeanFactory中）。这时候整个 BeanFactory 既可以给客户端使用了。

- 注解方式

    使用方式如下：@Autowired 告知 Spring 容器需要为当前对象注入哪些依赖对象，@Component 配合 classpath-scanning 使用，只需要在 Spring 的配置文件中增加一个“触发器” `<context:component-scan base-package="包名">`,就会扫描该包下面所有标注 @Component  的类然后将其添加到容器中进行管理，并根据它们所标注的 @Autowire 为这些类注入符合条件的依赖对象。

    ```java
    @Component
    public class A{
    	@Autowired
        private B b;
        @Autowired
        private C c;
        
        // 省略构造方法
    }
    
    @Component
    class B{
    	// 省略
    }
    
    @Component
    class B{
    	// 省略
    }
    ```

#### 4.3 BeanFactory 的 XML 之旅

Spring 在 DTD 的基础上引入 XML Schema 的文档声明，最终使用基于 XSD 的文档声明。常见标签如下：

- `<beans>`：包含 0 或 1 个 `<description>` 和 多个 `<bean>` 以及 `<import>` 或者 `<alias>`，一般用于对 `<bean>` 进行统一的默认行为设置。主要包括
    - `default-lazy-init`：默认为 false
    - `default-autowire`：默认为 no，其他包括：byName,byType/constructor 和 autodetect
    - `default-dependency-check`：默认值为 none 即不进行依赖检查，其他包括：objects/simple/all
    - `default-init-method`，这里统一指定初始化方法名，每个 bean 标签内部可以单独指定
    - `default-destroy-method`，这里统一指定销毁方法名，每个 bean 标签内部可以单独指定
- `<import>`：如果配置文件 xml 按照模块或者层次关系分为多个，需要导入，例如 A.xml 中的 `<bean>`  定义依赖 B.xml 中的 `<bean>` 定义，需要在 A.xml 中引入 B.xml 即：`<import resource="B.xml">`

- `<alias>`：为某些 `<bean>` 起别名，一般为了简写，如将 aaa 简写为 a  `<alias name="aaa" alias="a">`

**因为每个业务对象作为一个个体，对应于 XML 配置中的 `<bean>` 元素，单个业务对象配置方式如下：**

- id 属性：通过 id 属性来指定当前注册对象的 beanName，因为注册到容器中的对象需要唯一的标识来区分。当然内部 bean 可以不用指定 id 属性。
- name 属性：为指定的 `<bean>` 取别名，`<bean id="XX" name="/dfdXX" class="YY">`，因为 name 可以使用 id 中不能使用的一些字符，如 `/`，并且可以通过逗号、空格、冒号来分割多个 name 值。
- class 属性：注册到容器中对象需要通过 class 属性指定其类型。

**各个业务对象之间相互协作从而需要相互依赖，即构造方法和 setter 方法对应的 XML 如何表示**：

- 构造方法注入对应的 XML 方式

    ```xml
    <bean id = "a“ class = "xxxxx">
       // 两种方式
       <constructor-arg>
          <ref bean = "b">
          <value>1111<value>
       </constructor-arg>   
       <constructor-arg ref = "b" value = "123"/>                  
    </bean>   
    // 如果需要明确配置项和对象的构造函数参数列表一一对应，可以使用 type 或则 index，例如 Java 代码中有几个构造函数，增加标签即可，其它不变
    <constructor-arg type="int">                            
    <constructor-arg index = "0">                     
    ```

- setter 方法注入对应的 XML 方式

    主要提供了 `<property>` 元素，其中 name 属性表名注入对象所对应的实例变量名称，value 和 ref 属性表示具体的依赖对象引用或者值。==如果使用 property 进行依赖注入，对应必须提供默认构造方法==。

    ```xml
    <bean id = 'a' class = "XXXX">
        // 同样两种方式
        <property name = "bb">
            <ref bean = "b">
        </property>
        <property name = "cc" ref = "c">
    </bean>
    ```

- 注意：`<constructor-arg> ` 与 `<property>` 可以共存在一个 bean 中。

- 上面连个标签内部可用的配置项：bean/ref/idref/value/null/list/set/map/props

    - value 可以包括：基本数据类型 + 对应的包装类 + String

    - ref 来引用容器中其他对象实例，后面选择有：bean/local/parent，bean 通吃

        因为 BeanFactory 是分层次的，容器 A 在初始化时候如果首先加载容器 B 的所有对象定义然后再加载自身中对象，则  B 为 A 的父容器。

    - idref ：为当前对象注入所依赖的对象的名称，不是引用，当然使用 value 将名称注入也行，但是 idref 可以在容器解析配置的时候进行检查这个 beanName 在不在。

    - list：注入 java.util.List 及其子类或者数组类型的依赖对象。list 中虽然值可以是 value,ref,bean 但是一般是一类。

    - set：有序的注入

    - map：对应：`<entry key=“XXX” >   <value> VVV </value>   </entry>`

- depends-on：默认都是显式的指定 bean 直接的依赖关系，如果没有通过 `<ref>` 来明确指定 A 依赖对象 B，怎么能使容器在实例化 A 之前先实例化 B。需要在 A  的 `<bean id = XXX   depends-on = "B 的 bean id 名称">`，通常是拥有静态初始化代码块或者数据库驱动注册之类场景。

- autowire：根据 bean 定义的某些特点将相互依赖的某些 bean 直接自动绑定，无需手工明确该 bean 定义相关的依赖关系。**手工绑定会覆盖自动绑定行为**，同时自动绑定只适用于原生类型、String 类型以及 Class 类型和他们的类型数组==之外==的对象类型。

    - no：默认不进行自动绑定；

    - byName：按照代码类中声明的变量名称与 XML 配置文件中声明的 bean 定义的 beanName 进行匹配，相匹配的 bean 定义自动绑定到当前实例变量上。

        ```java
        public class Foo{
        	private Bar b;
        }
        
        <bean id = "foobean" class = "XXXX" autowire="byName"/>
        <bena id = "b" class = "YYYY"/>    
        ```

    - byType：同上，如果没有找到不设置，找的多个也不会配置。就是按照 bean 中 Class 中找类型。

    - constructor：针对构造方法参数的类型惊喜自动绑定，匹配**构造方法的参数类型**，不是实例类型。

    - autodetect：若对象有无参数的构造方法，按照 bytype，否则按照 constructor 模式。

- dependency-check：通常与自动绑定使用，保证自动绑定完成之后，最终确定每个对象所依赖的**所有对象**是否按照预期进行注入。

- lazy-init：ApplicationContext 容器启动时候就会进行初始化，若不想某个 bean 进行启动就初始化可以指定这个，**但是指定不一定就一定会延迟**。**如果某个非延迟初始化的 bean 定义依赖延迟加载的 bean，还是会直接初始化****。

**针对继承的 Bean**：两个 bean 可以在 XML 中分配单独配置，也可以在子类 `<bean>` 标签中加上 `parent="父类 bean 的 beanName"`，然后只对特定变化的属性进行修改即可。

parent 属性和 abstract 属性结合使用达到 bean 定义模板化。即模板 bean 使用 abstract 关键字，但是可以不指定 class，也可以指定，然后其他 bean 增加 parent 的值指向刚才定义的 bean 即可。

**bean 的 scope**：因为 BeanFactory 作为轻量级容器，可以管理对象的生命周期。

**scope 用来声明容器中对象所对应处的限定场景或者说对象的存活时间，即容器在对象进入其相应的 scope 前生成并装配这些对象，在该对象不再处于这些 scope 的限定之后，容器通常会销毁这些对象。**配置中的 bean 定义可以看做一个模板，容器根据这个模板来构造对象，但是根据这个模板创建多少对象，对象存活多久就看 bean 对应的 scope 决定。

- singleton（默认）：IOC 容器内部只有一个实例，所有对该对象的引用将**共享**该实例，该实例从容器启动，并且因为第一次请求而初始化之后，将一直存活到容器退出。

    它和单例模式不一样，两者语义是不同的，标记为 singleton 的 bean 是由容器来保证这个类型的 bean 在同一个容器中只存在一个共享实例，但是单例模式时保证在同一个 Classloader 中只存在一个该类型实例。

- prototype：容器在接到该类型对象的请求的时候，会每次重新生成一个新的对象实例给请求方。**该类型的对象的实例化和属性设置由容器完成，但是准备完成只有将对象实例返回给请求方之后，容器就不再拥有当前返回对象的引用，请求方需要自己负责当前返回对象的后继生命周期的管理工作（包括对象销毁）。**
- request、session、global session **只能使用于 Web 应用程序，也就是只能在 XSD 中配置**，通常与 XmlWebApplicationContext 共同使用。
    - request ：Spring 的 XmlWebApplicationContext 容器会**为每个 HTTP 请求创建一个全新的 request–Processor 对象供当前请求使用**，请求结束之后该对象实例的生命周期即结束。可以看做特定场景的 prototype。
    - session：生命周期比 request 稍微长点。
    - global session **只能用于基于 portlet 的 Web 应用程序才有意义，在基于 servlet 的 Web 应用在使用它和使用 session 效果一样**。

##### 4.3.7 偷梁换柱：方法注入和方法替换

```java
public class A  {
	private B b;
	public B getNewsBean() {
		return b; 
	}
}
```

如果将上述 A 在 XML 配置为 prototype，但是当将 B 的实例注入 A 之后，A 就一直拥有这个 b实例，所以每次返回的还是同一个 实例。解决方式：每次调用 getNewBean() 能返回一个新的 B 实例。

-  方法一：方法注入

    让 getNewBean() 方法声明符合规定的格式，然后在配置文件中通知容器，当该方法被调用的时候，每次返回指定类型的对象实例即可。该方法必须能被子类重写，**因为容器会为要进行方法注入的对象使用 CGLIB 动态生成一个子类实现，从而替代当前对象**。然后只要在 XML 中为 A 的 bean 标签添加：`<lookup-method name ="方法名 getNewBean" bean = "对象名 b">`。

- 方法二：使用 BeanFactoryAware 接口

    即在实现 getNewBean() 方法时候保证每次调用 BeanFactory 的 getBean(“b”) 时候每次都能取到 B 的新实例即可。所以通过 BeanFactoryAware 接口，容器在实例化实现了该接口的 bean 定义的过程中会自动将容器本身注入该 bean，因此该 bean 就拥有了其所在的 BeanFactory 的引用。所以只需要 A 类实现该接口，然后重新 setBeanFactory() 方法以及更改 getNewBean() 即可。

    ```java
    public class A  implements BeanFactoryAware{
        private BeanFactory beanFactory;
    	private B b;
        public void setBeanFactory(BeanFactory bf){
        	this.beanFactory = bf;
        }
        public B getNewBean(){
        	return beanFactory.getBean("b");
        }
    }
    ```

- 方法替换

    与方法注入只是通过相应方法为主体对象注入依赖对象不同，方法替换更多体现在方法的实现层面上，它可以灵活替换或者说以新的方法实现覆盖掉原来某个方法的实现逻辑。基本上可以认为，方法替换可以帮助我们实现简单的方法拦截功能。
    假设某天我看FXNewsProvider不爽，想替换掉它的getAndPersistNews方法默认逻辑，这时，
    我就可以用方法替换将它的原有逻辑给替换掉。步骤：自定义类继承 `beans.factory.support.MethodReplacer` ，然后重写 reimplement 方法，同时在 A 类的 bean 标签中配置 `<replaced-method name="要替换的方法名" replacer="替换后方法名">`



#### 4.4 容器内部流程

![image-20200614172037054](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200614172037054.png)

IOC 容器会以某种方式加载 Configuration Metadata（通常为 XML 格式的配置信息），然后根据这些信息绑定整个系统的对象，最终组装成一个可用的基于轻量级容器的应用系统。==上述流程可以划分为两个阶段：容器启动阶段和 Bean 实例化阶段==。分别相当于按照图纸装生产线和使用装配好的生产线生产产品。

**容器启动阶段**

主要包括：加载配置、加载配置信息、装备到 BeanDefinition 和其它后处理。

**该阶段首先会通过某种途径加载配置元数据**，除了代码方式之外，一般使用工具类（BeanDefinitionReader）来对加载的配置元数据进行解析和分析，**然后将分析之后的信息编组为对应的 BeanDefinition，最后把这些保存了 bean 定义必要信息的 BeanDefinition 注册到相应的 BeanDefinitionRegistry，容器启动工作就完成了。**

**Bean 实例化阶段**

主要包括：实例化对象，装配依赖、生命周期回调、对象其他处理，注册回调接口等。

**上阶段结束之后，所有的 bean 定义信息都通过 BeanDefinition 的方式注册到了 BeanDefinitionRegistry 中，当某个请求方通过容器的 getBean 方法明确的请求某个对象或者因为依赖关系容器需要隐形的调用 getBean 方法时候，就会触发第二阶段活动。**

该阶段，容器会首先检查所请求的对象之前是否已经初始化。如果没有，就会根据注册的 BeanDefinition所提供的信息来实例化所请求的对象，并为其注入依赖。如果该对象实现了某些回调接口，也会根据回调接口的要求来装配它。当该对象装配完毕之后，容器会立即将其返回请求方使用。



==**插手容器的启动**==

**Spring 提供了 BeanFactoryPostProcessor  的容器拓展机制，该机制允许我们在容器实例化响应对象之前，对注册到容器的 BeanDefinition 所保存的信息做相应的修改**。相当于在第一阶段加入最后一步，可以对最终的 BeanDefinition 做一些额外的操作，如修改其中 bean 定义的某些属性，为 bean 定义添加其他信息等等。

**自定义过程**：如果自定义实现 BeanFactoryPostProcessor，需要实现 BeanFactoryPostProcessor 接口。同时，**因为一个容器可能拥有多个 BeanFactoryPostProcessor**，这时候需要实现类同时实现 Spring 的 `core.Ordered` 接口，保证各个 BFPP 可以按照预定设置顺序执行。同时 Spring 提供了几个常见的 BFPP 实现类：`org.springframework.beans.factory.config.PropertyPlaceholderConfigurer` 和 `PropertyOverriddeConfigurer` 类。同时为了处理配置文件中的数据类型和真正业务对象所定义的数据类型转换，Spring 允许通过 `CustomEditConfigurer`类来注册自定义的 PropertyEditor 以帮助容器中默认的 PropertyEditor。

针对基本的 IOC 容器 BeanFactory 和高级的容器 ApplicationContext，分别有不同方式应用 BeanFactoryPostProcessor。**BeanFactory 中需要用手动方式应用所有的 BFPP，ApplicationContext 会自动识别配置文件中的 BFPP 并应用它**，所以只需要在 XML 配置文件中将  BFPP 的实现类按照 `<bean>` 的方式配置即可。

BeanFactory 手动配置为：多个 BFPP 则都要配置

```java
// 声明将被后处理的BeanFactory实例
ConfigurableListableBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("..."));
// 声明要使用的BeanFactoryPostProcessor 
PropertyPlaceholderConfigurer propertyPostProcessor = new PropertyPlaceholderConfigurer();
propertyPostProcessor.setLocation(new ClassPathResource("..."));
// 执行后处理操作 propertyPostProcessor.postProcessBeanFactory(beanFactory);
```

上面说的提供的 BFPP 实现类功能：

- PropertyPlaceholderConfigurer 
    通常我们不想将类似于系统管理相关的信息同业务对象相关的配置信息混杂到XML配置文件中，以免部署或者维护期间因为改动繁杂的XML配置文件而出现问题。我们会将一些数据库连接信息、邮件服务器等相关信息单独配置到一个properties文件中，这样，如果因系统资源变动的话，只需要关注这些简单properties配置文件即可。
    PropertyPlaceholderConfigurer允许我们在XML配置文件中使用占位符（PlaceHolder），并将这些占位符所代表的资源单独配置到简单的properties文件中来加载。以数据源的配置为例，使用了PropertyPlaceholderConfigurer之后（这里沿用代码清单4-42的配置内容），可以在XML配置文件中按照代码清单4-43所示的方式配置数据源，而不用将连接地址、用户名和密码等都配置到XML中。```

    ```xml
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="url">
    		<value>${jdbc.url}</value>
    	</property>
    	<property name="driverClassName">
    		<value>${jdbc.driver}</value>
    	</property>
        ...
    </bean>
    ```

    对应的 jdbc.properties 如：

    ```java
    jdbc.url=jdbc:mysql://server/MAIN?useUnicode=true&characterEncoding=ms932&
    failOverReadOnly=false&roundRobinLoadBalance=true
    jdbc.driver=com.mysql.jdbc.Driver
    ```

    当 BeanFactory 在第一个阶段加载完所有配置信息，其中保存的对象的属性还是以占位符形式存在，当 PPC 作为 BFPP 应用时候，就会使用配置文件 properties 中配置信息来替换相应 BeanDefinition 中占位符所表示的属性值，当进入第二阶段实例化 Bean 时候，该 bean 定义中的属性值都是最终替换完成的。**同样会检查 Java 的 System 类中的 Properties，可以通过 setSystemPropertiesMode() 来控制是否加载或覆盖 System 相应的 Properties 行为。

- PropertyOverrideConfigurer

    ==可以对容器中配置任何想处理地 bean 定义的 property 信息进行覆盖替换==。例如 xml 中配置 dataSource.maxActive = 100，但是可以在 PropertyOverrideConfigurer 加载的 properties 文件中设置 dataSource.maxActive = 200，对 bean 定义的属性直接替换。properties 文件配置格式：`beanName.propertyName=value`

    同样该属性在 Spring的配置文件中进行配置：如果多个对同一个bean的同一个属性处理，最后一个生效。

    ```xml
    <bean class="org.springframework.beans.factory.config.PropertyOverrideConfigurer">
    	<property name="location" value="pool-adjustment.properties(属性文件名称)"/>
    </bean>
    ```

    **该类的父类：PropertyResourceConfigurer 提供一个方法 convertPropertyValue() **,所以子类（包括上面 PPC也是其子类）可以覆盖该方法来惊喜配置项进行转换。如对加密后的字符串解密之后再覆盖相应 bean 定义。

- CustomEditorConfigurer

    **上面两个都是修改 BeanDefinition 中数据变更来达到目的**，CEC 只是**辅助性的将后期会用到的信息注册到容器中**，对 BeanDefinition 没有改动。

    **因为一般对象依赖的对象不管什么类型，配置在 XMl 中记载的都是 String 类型，程序使用的是各种类型的对象。所以要将容器从 XMl 读取的字符串形式转换为具体对象**。Spring 内部通过 JavaBean 和自身实现的的 PropertyEditor 来进行转换，一般为每个对象类型提供一个 PropertyEditor 即可，位于 `org.spXXX.beans.propertyeditors ` 包中，主要包括：String/Byte/CharArrayPropertyEditor，将符合 CSV 格式的字符串/字节/字符转换为 对应的数组格式，默认逗号分隔。ClassEditor，根据String 类型的 class 名称转换得到对应的 class 对象，相当于 Class.forName(String)。FileEditor等等。**这些容器都会默认加载使用，如果不是上面的类型我们需要自定义 PropertyEditor（继承 PropertyEditorSupport 即可），然后通过 CEC 来告诉容器。**



##### 4.4.3 Bean 的一生

容器启动之后不会马上实例化对应的 bean 定义，**第一阶段容器仅仅拥有所有对象的 BeanDefinition 来保存实例化阶段将要使用的必要信息**，只有请求方通过 BeanFactory 的 getBean() 方法来显示请求某个对象实例或者因为依赖而隐式的时候才可能会进行实例化。因为某 bean() 定义的 getBean() 方法第一次调用不论是隐式还是显式，都会调用 createBean() 触发实例化，后面调用 getBean() 就不一定再次实例化了（例如 Prototype类型）。隐式请求主要包括两种：

> getBean() 方法位于：org.spriXXX.beans.factory.support.AbstractBeanFactory 类中
>
> createBean() 位于：org.sprXXX.beans.factory.support.AbstractAutowireCapableBeanFactory 类中，是上面的子类。

- 针对 BeanFactory：因为默认延迟初始化，所以在因为请求对象 A 而自身在实例化时候依赖 B，导致容器内部调用 getBean()，所以相对于 B 就是隐式的。
- 针对 ApplicationContext：默认启动就实例化所有 bean 定义，它的启动和实例两个步骤是连续的，所以相当于隐式调用了 getBean()。

![image-20200615133816256](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200615133816256.png)

- 步骤一：Bean 的实例化和 BeanWrapper

    **容器内部实现采用策略模式来决定用何种方式初始化 bean 实例**。一般就是反射或者 CGLIB 动态字节码生成来初始化相应的 bean 实例或者动态生成其子类。

    `beans.factory.support.InstantiationStrategy` 定义是实例化策略的抽象接口，其直接子类`SimpleInstantiationStrategy` 实现了简单的对象实例化功能，可以通过反射来实例化对象实例，但不支持方法注入方式的对象实例化。`CglibSubclassingInstantiationStrategy` 继承了`SimpleInstantiationStrategy` 的以反射方式实例化对象的功能，并且通过 CGLIB 的动态字节码生成功能，该策略实现类可以动态生成某个类的子类，进而满足了方法注入所需的对象实例化需求。**默认情况下**，容器内部采用的是 `CglibSubclassingInstantiationStrategy`。
    容器只要根据相应 bean 定义的 `BeanDefintion` 取得实例化信息，结合 `CglibSubclassingInstantiationStrategy` 
    以及不同的 bean 定义类型，就可以返回实例化完成的对象实例。但是，不是直接返回构造完成的对象实例，而是以 `BeanWrapper` 对构造完成的对象实例进行包裹，**所以第一步返回相应的BeanWrapper实例**。

    通过 `BeanWrapper` 接口的实现类 `BeanWrapperImpl` 来对某个 bean 进行包裹，然后再进行设置或者获取 bean 相应的属性值。避免直接使用反射 API。

    > 因为该接口继承了 beans.PropertyAccessor 接口，**可以以统一的方式对对象属性进行访问**。同时继承了 PropertyEditorRegistry 接口，从上面可知CustomEditorConfigurer 只是**辅助性的将后期会用到的信息 PropertyEditor 注册到容器中**。后期就是给该接口用的。
    >
    > 所以 BeanWrapperImpl 示例内部包裹了 对象实例，同时拥有 PropertyEditor，这样 BeanWrapper 在转换类型、设置对象属性值的时候就可以正常执行了。

- 步骤二：各种 Aware 接口

    当对象实例化完成，并且相关属性以及依赖设置完成之后， Spring 容器会检查当前对象实例是否实现了一系列以 Aware 命名结尾的接口（全是if 语句: bean instanceof XXXAware），如果是则将这些 Aware 接口定义中的依赖注入给当前对象实例。包括：

    - org.springframework.beans.factory.BeanNameAware。如果Spring容器检测到当前对象实
        例实现了该接口，会将该对象实例的bean定义对应的beanName设置到当前对象实例。

    - BeanClassLoaderAware。会将对应加载当前bean的Classloader注入当前对象实例。

    - BeanFactoryAware。方法注入的时候，使用该接口以便每次获取prototype类型bean的不同实例。如果对象声明实现了BeanFactoryAware接口，BeanFactory容器会将自身设置到当前对象实例。这样，当前对象实例就拥有了一个BeanFactory容器的引用，并且可以对这个容器内允许访问的对象按照需要进行访问。

        

    以上几个Aware接口只是针对BeanFactory类型的容器而言，对于ApplicationContext类型的容器，也存在几个Aware相关接口。不过在检测这些接口并设置相关依赖的实现机理上，与以上几个接口处理方式有所不同，使用的是下面将要说到的BeanPostProcessor方式。不过，设置Aware接口这一步与BeanPostProcessor是相邻的，把这几个接口放到这里一起提及，也没什么不可以的。
    对于ApplicationContext类型容器，容器在这一步还会检查以下几个Aware接口并根据接口定义设置相关依赖。
    
    -  org.springframework.context.ResourceLoaderAware 。ApplicationContext 实现了Spring的ResourceLoader接口（后面会提及详细信息）。当容器检测到当前对象实例实现了ResourceLoaderAware接口之后，会将当前ApplicationContext自身设置到对象实例，这样当前对象实例就拥有了其所在ApplicationContext容器的一个引用。
    - org.springframework.context.ApplicationEventPublisherAware。ApplicationContext作为一个容器，同时还实现了ApplicationEventPublisher接口，这样，它就可以作为ApplicationEventPublisher来使用。所以，当前ApplicationContext容器如果检测到当前实例化的对象实例声明了ApplicationEventPublisherAware接口，则会将自身注入当前对象。
    - org.springframework.context.MessageSourceAware。ApplicationContext通过Message-Source接口提供国际化的信息支持，即I18n（Internationalization）。它自身就实现了Message-Source接口，所以当检测到当前对象实例实现了MessageSourceAware接口，则会将自身注入当前对象实例。
- org.springframework.context.ApplicationContextAware。如果ApplicationContext容器检测到当前对象实现了ApplicationContextAware接口，则会将自身注入当前对象实例
  
- 步骤三：BeanPostProcessor 接口

    **BeanPostProcessor 是存在于对象实例化阶段，而 BeanFactoryPostProcessor 则是存在于容器启动阶段**。

    **该接口作用是处理容器内所有符合条件的实例化后的对象实例**，声明了两个方法：`postProcessBeforeInitialization()`方法和 `postProcessAfterInitialization()` 方法。两个方法中都传入了原来的对象实例的引用，因此可以对传入的对象实例执行任何的操作。

    **主要作用**：处理标记接口实现类或者为当前对象提供代理实现。如上面的 ApplicationContext 中 Aware 接口就是通过它的方式处理的。如果每个对象实例化过程中走到这一步，容器就会检测到之前注册到容器的该接口的实现类 ApplicationAwareProcessor，然后调用其 postPro InitXX() 方法。检查并设置 Aware 相关依赖。

    上面这种方式是用于检查标记接口以便应用自定义逻辑。**还可以对当前对象实例进行更多处理：如替换当前对象实例或者字节码增强当前对象实例等**。同时 AOP 利用 BeanPostProcessor 来为对象生成相应的代理对象。同时可以自定义 BeanPostProcessor。

- 步骤四：InitializingBean 和 init-method

    InitalizingBean 是容器内部的一个对象生命周期标识接口，里面包含 afterPropertiesSet() 方法。

    如果对象实例化过程中调用过 BeanPostProcessor 的前置处理，就会接着检测当前对象是否实现了 InitializingBean 接口，如果是就调用 afterPropertiesSet() 方法来调整对象实例的状态。比如，在有些情况下，某个业务对象实例化完成后，还不能处于可以使用状态。这个时候就可以让该业务对象实现该接口，并在方法afterPropertiesSet()中完成对该业务对象的后续处理。

    但是如果业务对象实现了该接口，则相当于有侵入性，所以在自定义对象初始化方面可以通过在 XML 的 `<bean>` 中指定 init-method 属性即可。

- 步骤五：DisposableBean 与 destroy-method

    调用完成之后，容器将检查 singleton 类型的 bean 实例，看其是否实现`beans.factory.DisposableBean` 接口。或者其对应的 bean 定义是否通过`<bean>`的 `destroy-method` 属性指定了自定义的对象销毁方法。如果是，就会为该实例注册一个用于对象销毁的回调（Callback），以便在这些 singleton 类型的对象实例销毁之前，执行自定义销毁逻辑。什么时候执行需要自己调用。

    常用于 Spring 容器中注册数据库连接池时候，系统退出之后，连接池应该关闭以释放相应资源。

    其中 BeanFactory 容器在主程序退出之前或者其他合理时间调用 `ConfigurableBeanFactory` 提供的 `destroySingletons()` 方法销毁容器中管理的所有单例类型的对象实例。而 ApplicationContext 容器，调用 AbstractApplicationContext 中的 `registerShutdownHook()` 方法，该方法底层就是使用标准 Runtime 类的 `addShutdownHook()` 方法来调用相应 bean 对象的销毁逻辑。

    

## 第五章：Spring IOC 容器 ApplicationContext

**除了 BeanFactory 之外拓展功能**：统一资源加载策略、国际化信息支持、容器内部事件发布、多配置模块加载的简化。BeanFactoryPostProcessor、BeanPostProcessor 以及其他特殊类型 bean 的自动识别，容器启动后 bean 实例的自动初始化。

Spring 为基本的 BeanFactory 类型容器提供了 XmlBeanFactory 实现，同样为 ApplicationContext 类型提供了 `FileSystemXmlApplicationContext`、`ClassPathXmlApplicationContext`、`XmlWebApplicationContext` 常见的实现，分别表示从文件系统、Classpath 加载 bean 定义以及相应资源和用于 web 资源的 ApplicationContext 实现。

### 5.1 统一资源加载策略

标准类 `java.net.URL` 中，URL（Uniform Resource Locator）统一资源定位符，其基本实现通常局限于网络形式发布的资源的查找和定位工作，即基本上只提供了基于 HTTP、FTP、File 等协议的资源定位问题。但是资源可以以任何资源形式存在，包括二进制、字节流、文件等等形式，存放位置也包括文件系统、classpath等位置。**所以 url 类功能职责划分不清、资源查找和表示没有清晰的界限，资源查找后的形式多种多样，没有统一的抽象**。**要实现：资源查找完成返回给客户端的应该是一个统一的资源抽象接口，客户端对资源的处理应该基于资源抽象接口界定，不是资源查找和定位需要关系的事情**。

#### 5.1.1 Spring 中的 Resource 接口

使用 `core.io.Resource` 接口作为所有资源的抽象和访问接口，该接口根据**资源的不同类型或者资源处于不同场合，Spring 给出了不同的实现类**：都在 core.io 包里面：

- `ClassPathResource`：该实现从 Java 程序的 ClassPath 中加载具体资源并进行封装，可以使指定的类加载器或者指定的类进行资源加载。
- `FileSystemResource`：对 Java.io.File 类型进行封装，我们可以以文件或者 URL 的形式对该类型资源进行访问。
- `ByteArrayResource`：将字节（byte）数组提供的数据作为一种资源进行封装。如果用 InputStream 形式访问该类型资源，会根据字节数组的数据构建相对应的 ByteArrayInputStream 返回。
- 可以自定义自己实现，实现该接口就行，该接口中包括：`exists()`/`isOpen()`、`getURL()`/`getFile()`等方法。来查询资源状态、访问资源内容等等。一般不用。

#### 5.1.2 ResourceLoader

上面只是有了资源，但是通过该 `core.io.ResourceLoader` 接口提供了来查找和定位这些资源的统一抽象。当然具体资源查找定位有对应的实现类给出。**该接口中主要包括：`getResource(String location)` 方法根据指定的资源位置来定位到具体的资源实例**。具体实现类包括：

![image-20200615205701238](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200615205701238.png)

- DefaultResourceLoader

    默认实现类，该类首先检查资源路径是否以 classpath:前缀打头，如果是则直接构建 ClassPathResource 类型资源并返回。否则通过 URL，根据资源路径来定位路径，找到就构建 URLResource 类型的资源并返回；还没找到就通过 `getResourceByPath(String)` 方法来定位。 

- FileSystemResourceLoader

    为了避免DefaultResourceLoader在最后getResourceByPath(String)方法上的不恰当处理，我们可以使用org.springframework.core.io.FileSystemResourceLoader，它继承自DefaultResourceLoader，但覆写了getResourceByPath(String)方法，使之从文件系统加载资源并以FileSystemResource类型返回。这样，我们就可以取得预想的资源类型

批量查找的 ResourceLoader 包括：

- ResourcePatternResolver

    上面每次只能根据资源路径返回确定的单个 Resource 实例，而该接口可以根据指定资源路径匹配模式，每次返回多个 Resource 实例。该类继承 ResourceLoader，引入方法 `Resource[] getResource(String);`支持路径匹配模式返回多个 Resource 实例。

    该接口常见实现类：`PathMatchingResourcePatternResolver`，构建该实例的时候，需要指定一个 ResourceLoader，不指定就是默认 DefXX 那个，同样该实现类内部将匹配后确定的资源路径交给 ResourceLoader 来查找和定位。保证加载行为上的一致性。

#### 5.1.3 ApplicationContext 和 ResourceLoader

由 4-2 得知：，ApplicationContext 继承了 ResourcePatternResolver，当然就间接实现了 ResourceLoader 接口。所以，**任何的 ApplicationContext  实现都可以看作是一个ResourceLoader甚至ResourcePatternResolver**。而这就是ApplicationContext支持Spring内统一资源加载策略的真相。

![image-20200615212645810](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200615212645810.png)

通常，所有的ApplicationContext实现类会直接或者间接地继承 `context.support.AbstractApplicationContext`。同时 AbstractApplicationContext 继承了`DefaultResourceLoader`，那么，它的 `getResource(String)` 当然就直接用 `DefaultResourceLoader` 的了。剩下需要它“效劳”的，就是ResourcePatternResolver的Resource[]getResources (String)，当然，AbstractApplicationContext也不负众望，当即拿下。AbstractApplicationContext类的内部声明有一个resourcePatternResolver，类型是ResourcePatternResolver，对应的实例类型为 `PathMatchingResourcePatternResolver` 。之前我们说过 `PathMatchingResourcePatternResolver` 构造的时候会接受一个 `ResourceLoader`，而 AbstractApplicationContext本身又继承自DefaultResourceLoader，当然就直接把自身给“贡献”了。这样，整个 ApplicationContext的实现类就完全可以支持ResourceLoader或者ResourcePatternResolver 接口，你能说 Application-
Context 不支持 Spring 的统一资源加载吗？说白了，ApplicationContext 的实现类在作为 ResourceLoader 或者ResourcePatternResolver 时候的行为，完全就是委派给了 PathMatchingResourcePatternResolver 和 DefaultResourceLoader 来做。图5-2给出了AbstractApplicationContext与ResourceLoader和ResourcePatternResolver之间的类层次关系。

所以 ApplicationContext 可以作为 ResourceLoader 或者 ResourcePatternResolver 使用。

```java
ResourceLoader resourceLoader = new ClassPathXmlApplicationContext("配置文件路径");
// 或者
// ResourceLoader resourceLoader = new FileSystemXmlApplicationContext("配置文件路径");
```



### 5.2 国际化信息支持

包括货币形式、时间表现形式、地区语言等。Java 提供两个类：`java.util.Locale` 和 `java.util.ResourceBundle`。`Locale china = new Locale("zh","CN");` 前面是语言代码，后面是国家代码。ResourceBundle 可以读取配置文件来分别设置。

Spring 提供了进一步抽象的国际化信息的访问接口：`context.MessageSource`，里面包含多个不同参数的 `getMessage()` 方法，统一访问方式。



### 5.3 容器内部事件发布

P109

### 5.4 多配置模块加载的简化

程序会按照功能模块或者层次将配置文件进行分割为多个，在加载整个系统的 bean 的时候，需要让容器同时读入划分到不同配置文件的信息。相比 BeanFactory，可以直接以 String[] 数组形式引入配置文件路径即可。



## 第六章：IOC 容器拓展

主要是基于注解的依赖注入

### 自动绑定：autowire => @Autowired

可以通过default-autowire来指定默认的自动绑定方式，也可以通过每个bean定义上的autowire来指定每个bean定义各自的自动绑定方式，它们都是触发容器对相应对象给予依赖注入的标志。而将自动绑定的标志用注解来表示时，也就得到了基于注解的依赖注入（自动绑定）。

**@Autowired 和 autowire=”byType” 一样都是按照类型匹配进行依赖注入**，但是更加强大。

@Autowired 可以标注类定义中的域（属性）【不管属性用什么修饰符都行】或者构造方法定义【相当于之前自动绑定的 constructor 方式，即根据构造方法参数类型来注入】或者方法定义【不局限于 setter 方法，只要**该方法定义了需要被注入的参数**】。

> 标注之后， Spring 容器需要某种方式了解哪些对象标注了这个注解，哪些对象可以作为依赖对象。

**原有的自动绑定和 @Autowired 区别**：原来是将所有对象相关的 bean 定义追加到容器的配置文件中，然后使用 autowire 告知容器，按照该属性指定的绑定方式，将容器中各个对象绑定到一起。使用 @Autowired 之后，容器的配置文件中只有简单的 bean 定义，**为了给容器中定义的每个 bean 定义对应的实例注入依赖，可以遍历它们，然后通过反射，检查每个 bean 定义对应的类上各种可能位置上的 @Autowired。如果存在就从当前容器管理的对象中获取符合条件的对象，设置给 @Autowired 所标注的属性域、构造方法或者方法定义**。上述步骤示例代码

```java
Object[] beans = ...; 
for(Object bean:beans){
	if(autowiredExistsOnField(bean)){
        Field f = getQulifiedField(bean)); 
		setAccessiableIfNecessary(f);
		f.set(getBeanByTypeFromContainer());
	} 
	if(autowiredExistsOnConstructor(bean)){
		... 
    } 
	if(autowiredExistsOnMethod(bean)){
		... 
	}
}
```

Spring 中提供了一个 BeanPostProcessor 的实现 `beans.factory.annotation.AutowireAnnotationBeanPostProcessor` ,在实例 bean 定义的过程中来检查当前对象是否有 @Autowired 标注的依赖需要注入。我们需要做的就上将上面这个类也作为一个 bean 配置到配置文件中即可。

### @Qualifier

如果使用 @Autowired 找到多个统一类型的对象实例，就需要 @Qualifier 进一步区分。@Qualifier 就是 byName 自动绑定的注解版。如 A 依赖的 B 有两个实现，分别为 B1impl 和 B2impl，配置如下：

```xml
<beans>
	<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor"/>
	<bean id="newsA" class="..A"/>
	<bean id="b1" class="..B1impl"/>
	<bean id="b2" class="..B2impl"/>
</beans>
```

```java
public class A{
	@Autowired
    @Qualifier("b2")
    private B b;
    @Autowired
    private B bb;
}
```

同样可以标注于方法或者构造方法定义上：

```java
@Autowired
public void setUp(@Qualifier("b2") B b, B bb){
}
```

**解决现在还要将 bean 添加到 IOC 容器中，即还需要在 XML 中配置**：只需要在 XML 中配置：`<context:component-scan base-package="包名"/>` 则 classpath-scanning 从某一顶层包开始扫描。当扫描到某个类标注了相应的注解之后，就会提取该类的相关信息，构建对应的BeanDefinition，然后把构建完的BeanDefinition注册到容器。那么后面BeanPostProcessor为@Autowired或者@Resource所提供的注入肯定是有东西拿咯！

该配置默认扫描的注解类型是 @Component，同样在 @Component 语义上细化的 @Repository、@Service、@Controller 也可以扫描到。（他们都是配置在类上）。同样该配置扫描相关类定义并将它们添加到容器时候，使用默认的命名规则来生成那些添加到容器的 bean 定义的名称（beanName），一般就是开头首字母大写，当然可以指定名称：`@Component("自定义名字")`。当然该配置内部同时将AutowiredAnnotationBeanPostProcessor和
CommonAnnotationBeanPostProcessor一并注册到了容器中，所以，依赖注入的需求得以满足。



# 第三部分 Spring AOP 框架

## 第七章：一起来看 AOP

软件开发需要高效、容易维护、容易拓展。开发是为了解决需求，需求可以划分为业务需求和系统需求，**业务需求通过面向对象 OOP 对业务需求进行抽象和封装并且使之模块化，但是对于系统需求不能满足**。

> 两者区别：以贷款业务管理系统为例：从业务角度分为贷款申请、贷款信息管理、顾客管理等，按照功能划分模块开发即可。一般业务需求与实现是一一对应的，比较方便。
>
> 系统需求：开发过程中需要测试或者对系统进行监控，需要为业务需求的实现对象添加日志功能或者方法需要权限控制从而需要进行安全检查。虽然需求比较明确，但是如果以 OOP 集成到系统，就不是一个需求一个实现了，系统中每个业务对象都需要加入日志和安全检查，会遍及所有的业务对象。

**AOP（Aspect-Oriented Programing）将日志记录、安全检查、事务管理等系统需求进行模块化**，简化系统需求和实现之间的对比关系。AOP 引入 Aspect 来将系统中的横切关注点（就是上面的系统需求）进行模块化，Aspect 对于 AOP 类似于 Class 对于 OOP。AOP 是 OOP 的补充，即把以 Class 形式模块化的业务需求和以 Aspect 形式模块化的系统需求拼合就实现了整个系统。

<img src="Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200616094033696.png" alt="image-20200616094033696" style="zoom: 80%;" />



### 7.1 AOP 的尴尬与现实

==AOP 是一种理念，发展过程分为静态 AOP 和动态 AOP。==

- 静态 AOP：相应的横切关注点以 Aspect 形式实现之后，**会通过特定的编译器将其编译并且织入系统的静态类中，即直接以 Java 字节码形式编译到 Java 类中**，JVM 可以像之前一样加载 Java 类运行。缺点就是灵活性不够，如果横切关注点需要改变织入到系统的位置，需要重新修改 Aspect 定义文件，然后使用编译器重新编译并重新织入到系统。
- 动态 AOP：利用 Java 语言提供的动态特性， AOP 的织入过程是系统运行开始之后进行，而不是预先编译到系统类中，而且织入的信息大多采用外部 XML 文件格式保存，可以调整植入点和织入逻辑单元不改变系统其它模块，同时系统运行时候也可以动态改变织入逻辑。缺点就是性能降低：因为动态 AOP 的实现产品大都在类加载或者系统运行期间，采用对系统字节码进行操作方式来完成 Aspect 到系统的织入，但是反射和字节码技术提升正在改善。

### 7.3 Java 平台的 AOP 实现机制

#### 7.3.1 方式一：动态代理（Spring 默认实现）

动态代理(Dynamic Proxy)机制，可以在运行期间为相应的接口(Interface)动态生成对应的代理对象。所以可以将横切关注点逻辑封装到动态代理的InvocationHandler 中，然后在系统运行期间，根据横切关注点需要织入的模块位置， 将横切逻辑织入到相应的代理类中。以动态代理类为载体的横切逻辑，现在当然就可以与系统其他实现模块一起工作了。
这种方式实现的唯一缺点或者说优点就是，所有需要织入横切关注点逻辑的模块类都得实现相应的接口，因为动态代理机制只针对接口有效。 当然动态代理是在运行期间使用反 射，相对于编译后的静态类的执行，性能上可能稍逊一些。

#### 7.3.2 方式二：动态字节码增强

Java虚拟机加载的class文件都是符合一定规范的 ， 所以，只要交给Java虚拟机运行的文件符合Java class规范， 程序的运行就没有问题。 通常的class文件都是从Java源代码文件使用Javac编译器编译而成的， 但只要符合Java class规范， **我们也可以使用ASM或者CGLIB等Java工具库， 在程序 运行期间，动态构建字节码的class文件**。
在这样的前提下，我们可以为需要织入横切逻辑的模块类在运行期间， 通过动态字节码增强技术，
为这些系统模块类生成相应的子类，而将横切逻辑加到这些子类中，让应用程序在执行期间使用的是这些动态生成的子类，从而达到将横切逻辑织入系统的目的。
使用动态字节码增强技术，即使模块类没有实现相应的接口，我们依然可以对其进行扩展，而不 用像动态代理那样受限千接口。不过，这种实现机制依然存在不足，如果需要扩展的类以及类中的实 例方法等声明为final的话，则无法对其进行子类化的扩展。
Spring AOP在无法采用动态代理机制进行AOP功能扩展的时候，会使用CGLIB库的动态字节码增 强支持来实现AOP的功能扩展。

#### 7.3.4 自定义类加载器

所有的Java 程序的class都要通过相应的类加载器(Classloader)加载到Java虚拟机之后才可以运行。
默认的类加载器会读取class字节码文件， 然后按照class字节码规范， 解析并加载这些class文件到虚拟
机运行。如果我们能够在这个class文件加载到虚拟机运行期间， 将横切逻辑织入到class文件的话就完成了AOP和OOP的融合
我们可以通过自定义类加载器的方式完成横切逻辑到系统的织入， 自定义类加载器通过读取外部
文件规定的织入规则和必要信息， 在加载class文件期间就可以将横切逻辑添加到系统模块类的现有逻
辑中，然后将改动后的class交给Java虚拟机运行。偷梁换柱得漂亮， 不是吗？
通过类加载器，我们基本可以对大部分类以及相应的实例进行织入， 功能与之前的儿种方式相比
当然强大很多。不过这种方式最大的问题就是类加载器本身的使用。某些应用服务器会控制整个的类
加载体系，所以， 在这样的场景下使用可能会造成一定的问题。
JBoss AOP和已经并入AspectJ项目的AspectWer厄框架都是采用自定义类加载器的方式实现。



### 7.4 AOP 成员

- Joinpoint

    AOP 的功能模块织入 OOP 功能模块中，在织入过程前需要进行织入操作的系统执行点就称为 Jointpoint。常见的 Joinpoint 点包括：

    - 方法调用：当某个方法被调用的时候所处的程序执行点。
    - 方法调用执行（方法执行）：即某个方法内部执行开始时点。
    - 构造方法调用：程序执行过程中对某个对象调用其构造方法进行初始化的时点，一般是 new 语句。
    - 构造方法执行：同上；
    - 字段设置、字段获取：一般就是 setter 和 Getter 方法设定；
    - 异常处理执行：在某些类型异常抛出之后，对应的异常处理逻辑执行的时点。
    - 类初始化：一般是类中某些静态类型或者静态块的初始化时间点。

    一般必要的执行时点都可以作为 Joinpoint，但是像程序中**某个循环开始的时点可以作为一种 Joinpoint，但是较难捕捉，所以一般不支持**。

- Pointcut

    到 139 页开始没看





































## 第八章：Spring AOP 概述及其实现

Spring AOP 采用 Java 作为 AOP的实现语言（AOL），相比 AspectJ 那种语言拓展型实现，可以更快融入开发过程。Spring  AOP 没有提供 AOP 全部需求，如果无法满足可以求助于 AspectJ，两者有很好的集成关系。

Spring AOP 属于第二代AOP, 采用动态代理机制和字节码生成技术实现。 与最初的AspectJ采用编译器将横切逻辑织入目标对象不同，动态代理机制和字节码生成都是在运行期间为目标对象生成一个代理对象，而将横切逻辑织入到这个代理对象中， 系统最终使用的是织入了横切逻辑的代理对象， 而 不是真正的目标对象。

### 代理模式

**这里相当于一种静态代理** =》为目标对象创建一个静态代理。

访问者 -》代理-》被访问者；代理通常几乎会全权拥有被代理者的职能，代理能处理的就不劳烦被代理者，所以**代理可以减少被访问者的负担**，即是需要将代理请求转发给真正被访问者，但是在**转发请求之前或者之后加上特定逻辑，比如安全访问限制**。代理模式中四种角色如图所示：

![image-20200616153214306](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200616153214306.png)

- ISubject。 该接口是对被访问者或者被访问资源的抽象。 在严格的设计模式中， 这样的抽象 接口是必须的，但往宽了说， 某些场景下不使用类似的统一抽象接口也是可以的。
- Subjectimpl。 被访问者或者被访问资源的**具体实现类**。 如果你要访问某位明星， 那么Subjectimpl就是你想访问的明星；如果你要买房子， 那么Subjectimpl就是房主...…
- SubjectProxy，被访间者或者被访问资源的**代理实现类**， 该类持有一个ISubject接口的具体 实例。 在这个场条中，我们要对Subjectimpl进行代理， 那么SubjectPro对现在持有的就是Subjectimpl的实例．
- Client。 代表访问者的抽象角色， Client将会访问!Subject类型的对象或者资源。 在这个场景中， Client将会请求具体的Subjectimpl实例， 但Client无法直接请求其真正要访问的资源 Subjectimpl, 而是必须要通过!Subject资源的访问代理类SubjectPro对进行。

Subjectlmpl 和 SubjectProxy 都实现了相同 的接口 ISubject, 而 SubjectProxy 内部持有 Subjectimpl 的引用。 当Client通过request()请求服务的时候， Subject Proxy 将转发该请求给 Subjectlmpl 。 从这个角度来说，SubjectProxy 反而有多此一举之嫌了。 不过，SubjectProxy 的作 用不只局限千请求的转发，**更多时候是对请求添加更多访问限制**。Subjectlmpl和SubjectProxy之间 的调用关系。

在将请求转发给被代理对象Subjectimpl之前或者之后， 都可以根据悄况插入其他处理逻辑， 比 如在转发之前记录方法执行开始时间， 在转发之后记录结束时间， 这样就能够对Subjectimpl的 request ()执行的时间进行检测。 或者， 可以只在转发之后对Subjectimpl的request()方法返回结 果进行覆盖， 返回不同的值。甚至， 可以不做诮求转发， 这样， 就不会 有Subjectimpl的访问发生。 如果你不希望某人访问你的Subjectimpl, 这种场景正好适合。
代理对象SubjectProxy就像是Subjectimpl的影子，只不过这个影子通常拥有更多的功能。如果 Subjectimpl是系统中的Joinpoint所在的对象，即目标对象，那么就可以为这个目标对象创建一个代 理对象，然后将横切逻辑添加到这个代理对象中 。 当系统使用这个代理对象运行的时候，原有逻辑的 实现和横切逻辑就完全融合到一个系统中

![image-20200620105905739](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200620105905739.png)

Spring AOP 本质上采用该代理模式，但是实现细节上有所区别。

**问题**：不仅仅ISubjec七的实现类有request()方法，IRequestable接口以及相应实 现类可能也有request ()方法，它们也是我们需要横切的关注点.为了能够为IRequestable相应实现类也织入以上的横切逻辑， 我们又得提供对应的代理对象。所以虽然Joinpoint相同 (request() 方法的执行）， 但是对应的目标对象类型是不一样的．针对不一样的目标对象类型， 我们要为其单独实现一个代理对象。 而实际上， 这些代理对象
所要添加的横切逻辑是一样的。 当系统中存在成百上于的符合Pointcut匹配条件的目标对象时（悲观点说 目标对象类型都不同）， 我们就要为这成百上千的目标对象创建成百上千的代理对象，



**动态代理**：

可以为指定的接口在系统运行期间动态的生成代理对象。动态代理机制的实现主要由一个类和一个接口组成， 即 java.lang.reflect.Proxy类和java. lang. reflect. InvocationHandler接口。 下面， 让我们看一下， 如何使用动态代理来实现之前的 "request服务时间控制” 功能。虽然要为!Subject和IRequestable两种类型提供代理对象，但因为代理对象中要添加的横切逻辑是一样的， 所以， 我们只需要实现一个InvocationHandler 类（该类继承 InvocationHandler 接口，同时你们包含 invoke 方法，又是明显使用了 Proxy  类）就可以了。在其中的 invoke 方法中通过 `if(method.getName().equals("request"))` 来匹配所有包括 request 的方法。这样当 Proxy 动态生成的代理对象上相应的接口方法被调用的时候，对应的 InvocationHandler 就会拦截相应的方法调用，并进行相应处理。



InvocationHandler就是我们实现横切逻辑的地方， 它是横切逻辑的载体， 作用跟Advice是一样的，因此使用动态代理机制来实现 AOP 的过程中，可以在 Invocation Handler 基础上细化程序结构，并根据Advice的类型，分化出对应不同Advice类型的程序结构。我们将在稍后看到SpringAOP 中的不同Advice类型实现以及结构规格。
动态代理虽好，但不能满足所有的衙求。因为动态代理机制只能对实现了相应Interface的类使用， 如果某个类没有实现任何的Interface, 就无法使用动态代理机制为其生成相应的动态代理对象。 虽然 面向接口编程应该是提倡的做法，但不排除其他的编程实践。对千没有实现任何Interface的目标对象， 我们许哟啊寻找其他方式为其动态的生成代理对象。
默认情况下，如果Spring AOP发现目标对象实现了相应Interface, 则采用动态代理机制为其生成 代理对象实例。而如果目标对象没有实现任何Interface, Spring AOP会尝试使用一个称为CGLIB (Code Generation Library)的开源的动态字节码生成类库， 为目标对象生成动态的代理对象实例。



#### 8.2.3 动态字节码生成

使用动态字节码生成技术扩展对象行为的原理是， 我们可以对目标对象进行继承扩展， 为其生成相应的子类， 而子类可以通过覆写来扩展父类的行为， 只要将横切逻辑的实现放到子类中， 然后让系统使用扩展后的目标对象的子类， 就可以达到与代理模式相同的效果了。SubClass instanceof Superclass == true, 不是吗？ 

但是，使用继承的方式来扩展对象定义， 也不能像静态代理模式那样， **为每个不同类型的目标对 象都单独创建相应的扩展子类**。所以， 我们要借助于CGLIB这样的动态字节码生成库， 在系统运行期间动态地为目标对象生成相应的扩展子类。

CGLIB可以对实现了某种接口的类， 或者没有实现任何接口的类进行扩展。但我们已经说过， 可 以使用动态代理机制来扩展实现了某种接口的目标类，所以， 这里主要演示没有实现任何接口的目标 类是如何使用CGLIB来进行扩展的。

如果要对没有实现接口的类进行拓展，首先需要实现一个 net.sf. cg lib. proxy. Callback。 不过可以直接使用 net.sf.cglib.proxy.MethodInterceptor接口(Methodinterceptor 扩展了 Calssback 接口）。代码上就是新建一个类然后继承 MethodInterceptor 接口。然后在 intecept 方法将上面 invoke 方法中逻辑再重写一遍即可。从而该类就是西安类对 request 方法请求进行访问控制的逻辑，现在我们要通过CGLIB的Enhancer为目标对象动态地生成一个子类，并将RequestCtrlCallback中的横切逻辑 附加到该子类中，代码如下所示：

```java
Enhancer enhancer = new Enhancer() ; 
enhancer.setSuperclass(Requestable.class); 
enhancer.setCallback(new RequestCtrlCallback()); 
Requestable proxy = (Requestable)enhancer.create(); proxy.reques仁（）；
```

通过为enhancer指定需要生成的子类对应的父类，以及Callback实现，，enhancer最终为我们生成 了需要的代理对象实例。**使用CGLIB对类进行扩展的唯一限制就是无法对final方法进行覆写。**



## 第九章、第十章：AOP 一世和二世

#### 9.1 Spring AOP 中的 Joint point

AOP 底层实现机制没有改变，改变的是 AOP 概念实体的表现形式和使用方式。

Spring AOP 中仅仅支持方法级别的 AOP，即只支持方法执行类型的 Joinpoint。就是仅仅提供方法的拦截。不提供 AOP 本身的构造方法调用、字段设置以及获取、方法调用的 jointpoint。因为如果提供类中属性的级别的 jointpoint，这破坏了面向对象的封装，同时可以通过 setter 和 getter 方法拦截来实现。如果实在需求特殊，可以通过其它 AOP 实现产品，如 AspectJ。（Spring AOP 也提供对 AspectJ 的支持）。

#### 9.2 Spring AOP 中的 Pointcut

Spring中以接口定义org.springframework.aop.Pointcut作为其AOP框架中所有Pointcut的最顶层抽象，该接口定义了两个方法用来帮助捕捉系统中的相应Joinpoint, 并提供了一个TruePointcut 类型实例。 如果Pointcut类型为TruePointcut, 默认会对系统中的所有对象，以及对象上所有被支持 的Joinpoin进行匹配。 org.springframework.aop.Pointcut接口定义如以下代码所示：P 153

![image-20200620155351106](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200620155351106.png)

后面没看

255



## 第十一章：AOP 应用案例







## 第十二章：AOP 拓展篇







# 第四部分：使用 Spring 访问数据

Spring 提供了数据访问层，该层划分为三个部分：

该数据访问层以统一的数据访问异常层次体系为核心，接下来两个为两翼。

- 统一的数据访问异常层次体系

    Spring 将**特定**的数据访问技术相关的异常进行转译，然后封装为一套标准的异常层次体系。（客户端不再需要因为数据访问技术变更而改动代码）

- JDBC（Java Database Connectivity，Java 数据库连接）API 的最佳实践。

    **JDBC 是一套数据访问标准，规范了各个数据库厂商之间的数据访问接口**。但是具体的异常信息全部由各个 RDBMS（Relational Database Management System 关系数据库管理系统）来**自行界定**。从而导致应用程序需要根据具体的数据库提供商来判断异常中所提供的信息的含义。同时 JDBC API 更加贴近底层，使用起来比较繁琐。

- 统一的方式对各种 ORM 方案进行集成。

    除了使用标准的 JDBC 进行数据库访问，通常使用 ORM（Object Relational Mapping 对象-关系映射），ORM **主要用来屏蔽对象与关系数据库之间接口的非一致性**。所以 Spring 集成了各个 ORM，包括将它们特定的异常纳入 Spring 统一的异常层次体系。

## 十三章：统一的数据访问异常层次体系

### 13.1 DAO 模式

为了简化和统一不同应用场景下的数据存储机制和访问方式相关操作，J2EE 核心模式提出 DAO （Data Access Object，数据访问对象）模式，使用 DAO 模式**可以分离数据的访问和存储，从而屏蔽各种数据访问方式的差异性**。即无论数据存储在文本、CSV 或者数据库，使用 DAO 模式访问数据的***客户端代码可以完全忽略这种差异，以统一的接口来访问相关数据**。

**使用**：该模式首先定义一个数据访问对象 Dao 接口，你们包括操作数据的方法。客户端代码（通常为服务层 service 代码）**只需要声明依赖该数据访问接口，所有的数据访问全通过该接口进行**，而针对每个存储机制通过该接口的实现类来具体处理，所以无论数据存储机制怎么改变，只会导致 DAO 接口的实现类发生变化（可能是 Factory 对象的几行代码或者 IOC 容器配置文件的 class 类型也需要变化），但是客户端代码不变。

### 13.2 DAO 缺点

但是当引入每个实现类的特定数据访问机制的代码时候，每种特定的数据访问机制的异常处理都是特定的。

例如 JDBC 进行数据访问出问题时候会抛出 SQLException（属于 checked exception），所以 DAO 的实现类需要进行捕捉该异常并处理。但是这样客户端代码就不知道数据访问期间发生了什么问题，所以只能抛出客户端，所以该 DAO 实现类的中方法签名需要更改（加上 throws Exception），这 DAO 接口定义的方法声明也得改。**这时候因为使用 JDBC 做数据访问，所以针对抛出的特定的 SQLException，客户端代码需要捕捉异常并且处理**。同样当引入另一个 DAO 实现类，抛出异常又是不一样，DAO 接口和客户端又需要更改。

### 13.3 解决问题

不能在实现类上处理异常，就将**特定的数据访问异常进行封装之后然后抛出**，因为大多数（所有）数据访问操作抛出的异常对于客户端来说都是系统的错误，客户端无法处理只能重试，所以应该使用 unchecked exception 形式封装然后抛出，因为该方式不需要编译器检查。例如 `throw new RuntimeException(e);`

**虽然统一了数据访问接口定义**，但是以  SQLException 为例，不同厂商可能通过 ErrorCode 作为具体错误信息标准，有的使用 SQLState 返回详细的错误信息，返回给客户端之后，**客户端仍然需要针对不同的提供商采用不同的信息提取方式**。**通过 异常的分类转义来屏蔽差异性**：

- 不将对特定信息提取留给客户端，应该由 DAO 的实现类或者统一的工具类完成。【在 catch 中，如果是MySQL，分析错误信息然后抛出异常，如果是 Oracle。。。】
- 对不同异常类型进行区分，不能只是一个 RuntimeException，**按照类型分**，如连接失败，连不上属于获取资源失败，等划分为一个个子类型。

### 13.4 Spring 提供了一套 unchecked exception 类型的面向数据访问领域的异常层次体系

Spring 提供的统一异常层次体系中大部分异常类型定义在 `org.springframework.dao` 包中，**所有异常类型均以 `dao.DataAccessException` 为统领，然后根据职能划分为不同异常子类型。**

![image-20200622084440036](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200622084440036.png)

包括：访问结束进行清理失败异常、无法访问响应的数据资源异常等等。

## 十四章：JDBC 的最佳实践

因为 **JDBC 作为 Java 平台访问关系数据库的标准 API**，但是该标准主要面向底层的数据库操作，导致使用上简单的查询也需要写一大堆重复的代码。**重复（繁琐）、易出错**。

Spring 提供两种使用 JDBC  API 的最佳实践

- 以 jdbcTemplate 为核心的基于 Template 的 JDBC 使用方式
- 在 JDBCTemplate 基础上构建的基于操作对象的 JDBC 使用方式

### 14.1 基于 Template 的 JDBC 使用方式

项目中通常数据访问逻辑在 DAO 层中，不同开发人员自定义类来实现 DAO 接口，每个人都要在实现类中今次那个重复操作，如数据库连接开关、statement 等等。同时多个实现类会出现 statement 使用完成未关闭，connection 未释放等问题。

同样 JDBC 规范在数据访问异常处理时候不规范，仅仅通过 SQLException 类型来包括一切数据访问异常情况。并且将 ErrorCode 的规范制定交个各个数据库厂商，这样捕捉到 SQLException 之后需要在客户端判断是什么数据库，然后采用对应的 getErrorCode() 来取出 ErrorCode，然后和提供商提供的表格对比看问题。

#### JDBCTemplate

spring 提供 `org.springXXX.jdbc.core.jdbcTemplate` 作为数据访问的 Helper 类，该类是 Spring 数据抽象层提供的所有 JDBC API 最佳实践的基础，其他 helper 类或者更高的抽象都依赖该类。JDBCTemplate 主要关注：

- 封装所有基于 JDBC 的数据访问代码，然后以统一的格式和规范来使用 JDBC API。即所有基于 JDBC 的数据访问需求全部通过 JDBCTemplate 进行。
- 对 SQLException 所提供的异常信息在框架内部进行统一的转译，统一数据访问异常层次接口。

JDBCTemplate 主要通过模板方法对基于 JDBC 的数据访问代码进行统一的封装。

**模板方法模式**

主要用于对算法或者行为逻辑进行封装，即如果多个类中存在某些相似的算法逻辑或者行为逻辑，可以将这些相似的逻辑提取到模板方法类中实现，然后让相应的子类根据需要实现某些自定义的逻辑。一般定义一个抽象模板类，然后定义一个 final 的模板方法，表示该方法内部的逻辑是不可变的，然后方法内部调用其它方法，其它方法如果是所有子类都一样就可以实现，如果每个子类不一样，就声明为抽象方法定义即可。这样就不需要每个子类中都声明并且实现共有的逻辑，只需要每个子类实现特有的逻辑即可。

jdbcTemplate 抽象类中的 `execute(String sql)` 方法中就定义了逻辑，包括 getConnect()/createStatement()【抽象的 executeWithStatement(statement, sql)】，然后 `executeWithStatement()` 方法定义为抽象方法，由各个子类进行自定义。同时为了去除模板类的 abstract 关键字（有关键字的话每次进行数据访问的时候都需要给定相应的子类实现），从而引入 StatementCallback 接口（里面包括一个 doWithStatement()），就是将里面方法变为 `execute(StatementCallback callback) `。如果在 DAO 实现类中使用 JDBCTemplate，只要提供参数和相应的 Callback 即可，只需要关心与数据库访问逻辑相关东西即可。

> Callback 接口和模板方法类之间可以看成服务于被服务关系，模板方法类相应 Callback 做事，需要提供相应的资源，Callback 做完之后模板方法来处理公开资源，Callback 不用管。一般使用匿名内部类实现
>
> StatementCallback callback = new StatementCallback(){
>
> ​			public Object doWithStatement(Statement stmt){
>
> ​						return new Integer(stmt.executeUpdate(sql));
>
> ​			}
>
> }



**jdbcTemplate 实现结构**

该类继承 `core.jdbcOperations `接口和 `jdbc.support.jdbcAccsessor` 类。

接口界定了 JDBCTemplate 可以使用的 JDBC的操作集合包括查询到更新。jdbcAccessor 抽象类为 JDBCTemplate 的直接父类，主要为子类提供一些属性，包括 DataSource 接口（该接口是替代 DriverManager 接口的数据库连接方式）

同样上面提到的 Callback 接口有很多，按照接口公开的 API 自由度有面向 Connection 的 ConnectionCallback 回调接口，自由度最大，还有 StatementCallback 接口，处理静态 SQL 访问，还有 PreparedStatement 的可以处理包含查询参数的 SQL  请求，最后包括上面那个。



### 14.2 基于操作对象的 JDBC 使用方式

将数据库操作进行面向对象的形式进行建模，即将查询、更新、调用存储过程等数据操作抽象为操作对象。对象统一定义在 `jdbc.object` 包下面，以 `RdbmsOperation` 类为抽象顶层定义。

![image-20200622133614548](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200622133614548.png)



## 十五章：对各种 ORM 集成

Spring 对 ORM 集成主要体现在：

- 统一的资源管理方式

    相当于不管使用 JDBC 还是 ORM，使用方式和资源管理方式都是统一的。

- 特定于 ORM 的数据访问异常到 Spring 统一异常体系的转译

    转译特定的数据访问异常，并纳入 Spring 统一的异常层次体系中。

- 统一的数据访问事务管理和控制方式

    Spring 为所有的数据访问方式提供了公共统一的事务管理抽象层，来接管各种特定的事务管理方式，然后以统一的方式来管理和控制。



### 15.2 Spring 对 MyBatis 的集成

MyBatis 的自动化介于 JDBC 和 Hibernate 之间，但是上手快，因为 MyBatis 和 JDBC 一样都是一种依赖于数据资源的访问技术，所以也会遇到资源管理、异常处理等问题。

#### MyBatis 前生

使用 `com.mybatis.sqlmap.client.SqlMapClient` 进行数据访问，该方式需要中的配置文件来获取具体的 SQL 映射文件，使用的时候除了自动提交外，每个实例里面都要 `startTransaction()` ，执行语句，`commitTransaction()`，然后异常处理和抛出异常。所以在引入事务管理和异常处理以及资源管理时候，数据访问代码难以管理，即每次实现 DAO 的时候或者在 service 层管理事务的时候缺乏统一管理。

#### MyBatis 今生

为了将 MyBatis 的事务控制纳入 Spring 统一的事务管理，Spring 使用基于 SqlMapSession 的数据访问方式对 MyBatis 集成，可以将 MyBatis 内部直接指定的数据源已事务管理器等方式转由外部提供，如使用 IOC 容器为其注入。

Spring 为 MyBatis 提供 `SqlMapClientTemplate` 模板方法类和 `SqlMapClientCallback`回调接口来完成所有基于 MyBatis 的数据访问操作。前者管理事务、异常处理、资源管理，后者使开发人员专注于具体的数据访问逻辑。





# 第五部分 事务管理

## 第 17 章：有关事务的楔子

### 17.1 认识事务本身

软件系统中需要相应的**数据资源**（如 数据库或者文件系统）来 **保存系统状态**，我们必须对访问操作通过限定来当对系统状态所依托的数据资源进行访问的时候，保证系统处于”正确的完整状态。

事务本身的 4 个限定属性：ACID 

- 原子性要求事务所包含的全部操作是一个不可分割的整体， 这些操作要么全部提交成功，要么只要其中一个操作失败， 就全部失败。
- 一致性要求事务所包含的操作不能违反数据资源的一致性检查，数据资源在事务执行之前处于某个数据一致性状态， 那么， 事务执行之后也依然需要保持数据间的一致性状态。【如转账前后总钱数不变】
-  事务的隔离性主要规定了各个事务之间相互影响的程度。隔离性概念主要面向对数据资源的并发访问(Concurrency) , 并兼顾影响事务的一致性。 当两个事务或者更多事务同时访问同一个数据资源 的时候， 不同的隔离级别决定了各个事务对该数据资源访问的不同行为。

隔离级别分析见其他笔记

EJB 、 Spring、 JDBC等数据访问方式， 都允许我们为事务指 定以上提到的4种隔离级别， 但最终祁务是否以指定的隔离级别执行，则由底层的数据资源来决定。【因为向 Oracle 只支持 ReadCommit 和 Serialiable，指定其他的数据库不支持的，采用数据库默认的隔离级别】。

- 持久性：一旦整个事务操作成功提交，对数据所做的变更将被记载并不可逆转，数据库等数据资源管理系统会通过冗余存储或者多数据网络备份等方式，来保证事 务的持久性。

### 17.2 初识事务家族成员

在一个典型的事务处理场景中，有以下几个参与者

Resource Manager。 简称RM, 它负责存储并管理系统数据资源的状态， 比如数据库服务器、
JMS消息服务器等都是相应的ResourceManager。

![image-20200626115927955](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200626115927955.png)

![image-20200626120008272](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200626120008272.png)

## 第 18 章：Java 事务管理

在Java的局部事务场景中，系统里事务管理的具体处理方式，会随省所使用的数据访问技术的不同而各异。我们不是使用专用的事务API来管理事务，而是通过当前使用的数据访问技术所提供的基于程序与事务资源之间的通信通道的API来管理事务。例如 JDBC 通过 java.sql.Connection 以及 Hibernate 的 Session等技术。

**问题**：

- 局部事务的管理绑定到了具体的数据访问方式。导致事务管理代码和数据访问代码以及业务逻辑代码混杂，同时更换数据访问方式则要变动很多代码。
- 事务的异常处理没有统一的事务相关异常体系。

## 第 19 章：Spring 事务王国的架构

Spring 将**事务管理相关的关注点进行适当的分离和抽象**，体现在：

- 通过 Spring 事务框架，可以按照统一的编程模型来进行事务编程，不用关心具体的数据访问技术和需要访问的事务资源类型。
- spring 事务框架和 spring 提供的数据访问支持紧密结合，更好处理事务管理和数据访问。
- spring 提供的 AOP 使得事务框架实现声明式事务管理。

Spring 的事务框架设计理念的基本原则是：让事务管理的关注点与数据访问关注点相分离。

- 当在业务层使用事务的抽象API进行事务界定的时候，不需要关心事务将要加诸于上的事务资源是什么，对不同的事务资源的管理将由相应的框架实现类来操心。
- 当在数据访问层对可能参与邓务的数据资源进行访问的时候，只需要使用相应的数据访问API 进行数据访问，不需要关心当前的事务资源如何参与事务或者是否需要参与事务。这同样将 由事务框架类来打理。

对于开发人员只需要关系：通过抽象后的事务管理 API 对当前事务进行界定。



Spring 事务抽象架构的核心接口（也是整个事务抽象决策的顶层接口）：`org.springframework.transaction.PlatformTransactionManager`，作用是**为应用程序提供事务界定的统一方式**。接口定义为：

```java
public interface PlatformTransactionManager{
TransactionsStatus getTransaction (TransactionDefinition definition) throws Transaction Exception; 
void comrnit(TransactionStatus status) throws Transaction眨ception;
void rollback(TransactionStatus status) throws TransactionException; 
}
```

**具体由该接口针对不同的数据访问方式的对应的实现类执行**。因为一般事务管理放在 service 层，数据访问逻辑放在 dao 层。而 service 层对象会在同一业务方法中调用多个数据访问对象的方法。



#### 以 JDBC 自身的事务管理为例

JDBC的局部事务控制是由同一个java.sql.Connectio过长完成的，所以要保证两个DAO的 数据访问方法处千一个事务中，我们就得保证它们使用的是同一个java.sql.Connection。要做到这 一点 ， 通常会采用称为connection-passing的方式，即为同一个事务中的各个dao的数据访问方法传递当前事务对应的同一个 connection。**事务管理代码和数据访问代码之间通过 connection 直接耦合了**。



Spring 事务抽象包括三个接口：PTM 参照 TD中的属性定义来开启相关事务，事务开启之后到事务结束期间的事务状态由 TS 负责。

![image-20200626145858608](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200626145858608.png)

### TransactionDefinition

该接口定义了哪些事务属性可以指定，包括：事务隔离级别（5 种）、事务传播行为（表示整个事务处理过程中所跨越的业务对象，将以什么样的行为参与事务）、事务的超时时间、时候为只读事务。

需要该接口的实现类来给 PTM 创建事务提供信息。其实现类可以按照编程式事务和声明式事务分为两类。

![image-20200626150758837](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200626150758837.png)

- DTD 是该接口**默认实现类**，提供了各个事务属性的默认值，但是可以通过 setter 方法自定义。而 TransactionTemplate 是 Spring 提供的进行编程式事务管理的模板方法类。
- TransactionAttribute 主要面向**使用 AOP 进行声明式事务管理的场合**。添加了一个 rollbackOn(Throwable ex); 方法，**可以通过声明的方式指定业务方法在抛出哪些异常的情况下进行回滚事务**。 

### PlatformTransactionManager

该接口的整个抽象体系基于 **策略模式**，由该接口对事务界定进行统一的抽象，而具体的界定策略的实现则交给具体的实现类。给接口针对**各种数据访问技术都提供对应的实现类**。例如针对 JDBC 和 MyBatis 的 DataSourceTransactionManager 类，针对 Hibernate 的 hibernateTransactionManager 类，针对 JDO、JPA、JMS 等等都有。所以使用 Spring 的事务抽象框架进行事务管理的时候，根据数据访问技术选择对应是实现类即可。

**整个框架是以策略模式为基础的，但是从各个类的基础层次上看，实现更多依赖与模板方法模式**。

![image-20200626152531120](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200626152531120.png)

 AbstractPlatforrnTransactiont-lanager作为 DataSourceTransactio啦anager的父类，以模板方法的形式封装了固定的事务处理逻辑，而只将与 事务资源相关的操作以protected或者abstract方法的形式留给oataSourceTransactionManager 来实现。作为模板方法 父类，AbstractPlatformTransactionManager 替各子类实现了以下固定的事 务内部处理逻辑：

- 判定是否存在当前事务，然后根据判断结果执行不同的处理逻辑：` final TransactionStatus getTransaction(Transaction.Definitiondefinition) `。如果存在即看 TrXXDefinXXX 中定义的传播行为来决定挂起事务还是抛出异常，如果不存在事务也需要根据传播行为具体语义来决定处理方式。
- 结合是否存在当前事务的情况， 根据TransactionDefinition中指定的传播行为的不同语义执行后继逻辑
- 根据情况挂起或者恢复事务：
- 提交事务之前检查readOnly字段是否被设置，如果是的话，以事务的回滚代替平务的提交；`commit(TransactionStatus status)`
- 在事务回滚的悄况下，洁理并恢复事务状态；`rollback (TransactionStatus status)`
- 如果事务的Synchonization 处于active状态， 在事务处理的规定时点触发注册的 Synchonization 回调接口。



### TransactionStatus

事务处理过程中，通过该接口中的 setRollbackOnly ()方法标记事务回滚， 所以， Commit(TransactionStatus)在具体提交事务之前会检查rollBackOnly状态。如 果该状态被设惯， 那么转而执行事务的回滚操作。



## 第 20 章：使用 Spring 进行事务管理

### 编程式事务管理

一共两种方式：要么直接使用PlatformTransactionManager, 要么使用更方便的TransactionTemplate 。

- 方式一：接口= New 具体实现类，然后调用各种方法即可。

    优点：使用该方式可以**完整的控制整个事务处理过程，同时抽象了事务操作从而屏蔽了不同事务管理 API 差异**。

    缺点：比较偏底层，同时因为事务管理流程比较固定，所以推荐方式二

- 方式二：该方式通过**模板方法模式和 Callback 结合**对方式一的相关事务界定操作和相关的异常处理进行了模板化的封装，开发只要关注通过相应的 Callback 接口提供的具体事务界定内容即可。Spring针对TransactionTemplate 提供了两个Callback接口，TransactionCallback和TransactionCallbackWithoutResult, 二者的唯一区别就是是否需要返回执行结果。

    同时：TransactionTemplate会捕捉TransactionCallback或者TransactionCallbackWithout­Result邓务操作中抛出的unchecked. exceptjon并回滚事务 ， 然后将unchecked exception抛给上层处理。 所以， 现在我们只需要处理特定于应用程序的异常即可， 而不用像直接使用PlatformTran­sactionManager那样，对所有可能的异常都进行处理。

### 声明式事务

分离编程式事务中的事务管理代码和业务逻辑代码相互混杂问题。

因为事务管理本身就是一种横切关注点，可以通过 Spring AOP 来为其提供相应的 Advice 实现，然后织入到系统中需要该横切逻辑的 Jointpoint 出，从而将业务逻辑从事务管理逻辑中剥离出来。只需要提供一个**拦截器**，在业务方法执行开始之前开启 一个事务，当方法执行完成或者异常退出的时候就提交事务或者回滚事务。

同样需要某种方式来记录业务方法与对应事务信息之间的映射关系，来确定该方法时候需要事务支持。拦截器只需要查询这种映射关系就可以知道时候为当前业务方法创建事务，如果需要创建事务，就以业务方法作为标志，到映射关系中查找创建事务所需要的信息，然后创建事务。映射关系一般叫做**驱动事务的元数据**，一般采用外部配置文件（如 properties 或者 XML）。

Spring 提供了声明式事务一切设施，：`transaction.interceptor.TransactionInterceptor`，我们只需要提供 XML 或者注解元数据即可。

- XML：专门为事务管理提供了一个单独的命名空间：tx 命名空间。`<tx:advice>` 是专门为声明事务 Advice 而设置的配置元素。还有 `<tx:method>` 指定每条映射关系。
- 注解：原理是将对应业务方法的事务元数据，直接通过注解 标注到业务方法或者业务方法所在的对象上，然后在业务方法执行期间，通过反射读取标注在该业务 方法上的注解所包含的元数据信息，最终将根据读取的信息为业务方法构建事务管理的支持。`@Transactional` 配置信息和 `<tx:method>` 几乎相同。

## 第 21 章：Spring 事务管理之拓展篇

P423/ 673 需要看





# 第六部分：Spring 的 Web MVC 框架



## 第 22 章：迈向 Spring MVC 的路程

Servlet -> JSP -> 两者联合

Servlet 运行于 Web 容器之内 ， 提供了Session和对象生命周期管理等功能。 Servlet依然是Java类， 从中直接访问Java平台的 各种服务， 并使用相应的API支持是很自然的事情， 比如调用业务对象， 使用JDBC进行数据访问。

Servlet 强项在于无缝地衔接业务对象与 Web 层对象之间的调用。但是后来 Servlet 将流程逻辑控制、视图显示逻辑、业务逻辑、数据访问逻辑都混合了，造成后期维护困难。

JSP 将视图逻辑和 Servlet 分离出来形成模板化视图标准，这样 Servlet 中就不需要在 `doPost()` 方法中使用 `out.println()` 来输出 HTML 文件了，只需要直接使用 `forword(request, response, “XX.jsp”);` 即可。同时 JSP 最终编译称为 Servlet 来运行，同时针对使用 Servlet  处理 Web 请求，只需要在 web.xml 中注册相应的请求 URL 和具体的处理 Servlet 之间的映射关系。虽然可以通过引入 JavaBean 对业务逻辑进行了封装，但是 JSP 也越来越臃肿。

最初 MVC 格式：控制器、视图、模型：Servlet 进行 web 请求处理流程管理，JSP 进行视图渲染逻辑，JavaBean 进行业务逻辑处理和与数据层进行交互。

- 控制器负责接收视图发送的请求并进行处理， 它会根据请求条件通知模型进行应用程序状态的更新，之后选择合适的视图显示给用户。
- 模型通常封装了应用的逻辑以及数据状态。 当控制器通知模型进行状态更新的时候， 模型封装的相应逻辑将被调用。执行完成后，模型通常会通过事件机制通知视图状态更新完毕，从而视图可以显示最新的数据状态。
- 视图是面向用户的接口。当用户通过视图发起某种诮求的时候，视图将这些请求转发给控制器进行处理。 处理流程流经控制器和模型之后，最终视图将接收到模型的状态更新通知， 然后视图将结合模型数据，更新自身的显示。

**缺点：**在视图与模型间的数据同步工作是采用**从模型Push到视图**的形 式完成的。但是对于 web 应用来说， 局限千所用的协议以及使用场景， 无法实现这样的功能。所以，我们只能对MVC中的组件的最初作用定义做适当的调整，由控制器与模型进行交互 在原来通知模型更新应用程序状态的基础上，还要获取模型更新的结果数据，然后将更新的模型数据一并转发给视图，即由控制器从模型中 Pull 数据给视图。=》该样式称为 Web MVC。

**模式二缺点**：没有指定具体应用需要**几个控制器**。

- 情况一：Web 应用程序使用多个 Servlet 作为控制器，即一个 Servlet 对应一个 web 请求处理。通过 web.xml 来进行两者映射。**但是所有 web 请求的处理流程比较分散管理，以及不用**。

- 情况二：使用单一 Servlet 作为集中控制器，便于集中管理。

    **缺点**：因为所有请求都映射到该控制器，所以控制器类需要自己来根据Web请求的URL信息进行分析， 以判断处理流程的流向。 显然， 无法再借助Web容器 的URL映射匹配能力来完成这个工作了。同时一般逻辑是**硬编码到 控制器中，逻辑不能重用**。

### 22.4 Web 框架

Web框架存在的意义在千， 它们为Web应用程序的开发提供了一套可复用的基础设施， 这样开发 人员只需要关注特定于每个应用的逻辑开发工作， 而不需要每次都重复那些可以统一处理的通用逻 辑。 当前的Web开发框架有如下两种类型。

- 请求驱动类型（如 SpringMVC）：基千Servlet的请求／响应 (request/response) 处理 模型构建的。一般基于上面的模式二；
- 事件驱动类型：是基于组件的 Web 框架，如 Swing 等 GUI 框架， 将视图 组件化， 由视图中的相应组件触发事件，进而驱动整个处理流程。

请求驱动型框架通常会结合Front Controller 以及Page Controller模式。， 对单一Servlet控制器做进一步的改进，对原先过千耦合的各种控制器逻辑 进行逐步的分离。具体来说，就是由原来的单一Servlet作为整个应用程序的Front Controller 。 该Servlet 接收到具体的Web处理请求之后， 会参照预先可配置的映射信息， 将待处理的Web处理请求转发给次 一级的控制器(sub-controller)来处理，两级控制器共同组成了程序的控制器。

![image-20200627095057419](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627095057419.png)

## 第 23 章：Spring MVC 初体验

- 在Web层，即在框架的控制器的实现方面，SpringMVC对谓求处理期间涉及的各种关注点进行了合理而完全的分离，并且明确设置了相应的角色（进行了明确的划分，从而可拓展、可替换）用于建模并处理整个生命周期中的各个关注点， 这包括：
    ．设贸HandlerMapping用千处理Web请求与具体请求处理控制器之间的映射匹配；
    ．设置LocaleResolver用千国际化处理；
    ．设置ViewResolver用千灵活的视图选择等．
- 从表现(presentation)层面来看，Spring运用逻辑命名视图(logicalnamed view)策略，通过引入ViewResolver和View, 消晰地分离了视图类型的选择和渲染(Render)与具体控制器之间的耦合，使得各种视图技术可以很容易地集成到SpringMVC框架中。
- Spring MVC 可以更好的得到 IOC 容器提供的依赖注入支持，以及 AOP、数据访问层、事务管理层的支持。即 Spring 提供了良好的中间层支持。

### 23.1 总览 Spring MVC

Spring MVC 中也是通过引入 Front Controller 和 Page Controller 的**概念**来分离控制流程逻辑和具体的 Web 请求处理逻辑。 FC 就是 `web.servlet.DispatcherServlet`，负责接收从处理所有所有的 web 请求，只不过针对具体的处理逻辑会委派给下一级控制器去实现。PC 就是对应 `web.servlet.mvc.Controller` 。

**方式二：Servlet 控制器的工作**

- 获取请求信息，如请求的路径、各种参数值等
- 根据请求信息，调用具体的服务对象处理具体的 web 请求
- 处理完成之后，将要在视图中显示的模型数据通过 request 进行传递，最后通过 request-dispatcher 选择具体的 jsp 视图并显示。

### DispatcherServlet 处理流程

- HandlerMapping（web 请求的处理协调人）

    当将作为 FC 的 DS 注册到 web.xml 中时候，DS 就要服务于规定的一组 web 请求。因为是 一对多所以 DS 只能自己处理具体的 web 请求和具体的处理类之间的映射关系匹配。Spring MVC 中使用`web.servlet.HandlerMapping` 来**专门管理 web 请求到具体处理类之间的映射关系**。在Web请求到达DispatcherServlet之后 ， Dispatcher将寻求具体的HandlerMapping实例，以获取对应当前Web请求的具体处理类，即 org.springframework.web.servlet.Controller。

    ```xml
    <servlet>
     // 配置DS 
    </servlet>
    <servlet-mapping> 
    	<servlet-name>dispatcher</Servlet-name> 
        <Url-pattern>*.do</url-pattern> 
    </servlet-mapping> 
    ```

    

- `web.servlet.Controller` (Web请求的具体处理者）

    Controller是对应DispatcherServlet的次级控制器，它本身实现了对应某个具体Web诮求的处理逻辑。使用 handleMapping 查找 web 请求对应的 Controller 实例，查找结果返回给 DS，然后调用对应 Controller 执行处理请求。Controller的处理方法执行完毕之后，将返回一个 `Web.servlet.Controller` 的处理方法执行完毕之后，将返回一个 `web.servlet.ModeAndView` 实例，该实例里面包括两个内容。

    - 视图的逻辑名称（或者具体的视图实例）。DS 将根据该视图的逻辑名称来决定为用户显示哪个视图。
    - 模型数据。视图渲染过程中需要将模型数据并入视图的显示中。

- viewResolver 和 View（视图独立战争的领导者）

    SpringMVC 提出了一套基于 ViewResolver和View接口的Web视图处理抽象层，以屏蔽Web框架在使用不同的Web视图技术时候的差异性。【就是将相同的模型数据纳入不同的视图形式并且显示】

    Spring MVC通过引入org.springframework.Web.servlet.View接口定义， 来统一地抽象视图的生成策略。 之后，
    DispatcherServlet只需要根据Spring Controller处理完毕后通过ModelAndView 返回回的逻辑视图名称
    查找到具体的View实现，然后委派该具体的V iew 实现类来根据模型数据，输出具体的视图内容。实现类结构：

    ![image-20200627111804513](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627111804513.png)



现在， 视图模板与模型数据的合并逻辑， 以及合并后的视图结果的输出逻辑， 全部封装到了相应 的View实现类中。DispatcherServlet只需要根据Model知dView返回的信息， 选择具体的View实现类做最终的具体工作即可。不过， 与HandlerMapping帮助DispatcherServlet查找具体的Spring Controller以处理Web请求类似，DispatcherServlet现在需要依赖千某 个org.springfrarnework.web.servlet.ViewResolver来帮它处理逻辑视图名与具体的View实例之间的映射对应关系． ViewResolver将根据ModelAndView中的逻辑视图名查找相应的View实现类，然后将查找的结果返回 DispatcherServlet。DispatcherServlet最终会将ModelAndView中的模型数据交给返回的View 来处理最终的视图渲染工作．

![image-20200627112039709](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627112039709.png)

## 第 24 章：接触 Spring MVC 主要角色

### 24.1 忙碌的协调人 HandlerMapping

HandlerMapping帮助DispatcherServlet进行Web请求的URL到具体处理类的匹配。但是具体的次级处理器统称为 Handler（通常为 controller，但是不限于）。HandlerMapping要处理的也就是Web诮求到相应Handler之间的 映射关系。HandlerMapping 接口定义很简洁，具体工具由其对应的实现子类完成。主要方法为：`HandlerExecutionChain getHandler(HttpServle匕Request request) throws Exception;` 返回的 Handler­ExecutionChain中确实包含了用千处理具体Web请求的Handler。

**可用的实现类**

- BeanNarneurlHandlerMapping：必须保证视图模板中的请求路径`<a href="path1"></a>` 与容器中对应 Handler 的 beanName 一致。`<bean name ="/path" class = "">`		
- simpleUrlHandlerMapping：解除了两者必须一直的限定，更加灵活。在 value 属性中两者使用等号匹配。
- DefaultAnnotationHandlerMapping：用于基于注解的配置方式

**HandlerMapping 执行序列**

可以为DispatcherServlet提供多个Handler­Mpping供其使用，按照优先级进行选用，只要有一个返回可用的 Handler，则 DS 使用该 Handler 进行 web 请求的处理同时不再询问其他的 HandlerMapping。**因为上面的几个实现类都实现了 Ordered 接口，所以在web.xml 中配置实现类的 bean 标签时候，设置 `<peroperty name ' "order" value = "1">`，如果不设置 value 值，默认为 Integer.MAX_VALUE 。**

### 24.2亲密伙伴 Controller

Controller 是用于处理具体Web请求的handler类型之一。

**实现一个具体 Controller 方式**：

- 方式一：直接实现 Controller 接口，然后重写：`ModelAndView handleRequest(HttpServletRequest request, HttpServletReponse response)`。该方式虽然灵活但是需要关注底层细节，包括请求参数抽取，请求编码的设置、session 数据管理等等。

- 方式二：SpringMVC 提供的 Controller 实现体系，帮助我们处理了 web 请求过程中某些通用关注点。

    ![image-20200627122816820](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627122816820.png)

- AbstractController 类通过模板方法模式帮我们解决了如下儿个通用关注点：我们只需要在该类公开的：`handleRequestInternal(request, response)` 模板方法中实现具体的 web 请求处理过程中的其他逻辑。

    - 管理当前Controller所支持的请求方法类型(GET/POST) 
    - 管理页面的缓存设置，即是否允许浏览器缓存当前页面； 
    - 管理执行流程在会话(Session)上的同步

- SimpleFormController 类继承了BaseCommandController 的**自动生成数据绑定和通过 Validator 的数据验证功能**。也继承了 AbstractFormController 的**模板化处理流程**。主要针对单一表单处理。

    - 数据绑定：在Web应用程序中使用数据绑定的最主要好处就是 ， 我们再也不用自己通过request .getParameter(String)方法遍历获取每个请求参数， 然后根据需要转型为自己需要的类型了。 Spring MVC提供的数据绑定功能帮助我们自动提取HttpServletRequest中的相应参数，然后转型为需要的对象类型。 我们唯一需要做的， 就是为数据绑定提供一个目标对象， 这个目标对象在Spring中称为 Cormnand 对象， 此后的Web处理逻辑直接同数据绑定完成的Command 对象打交道即可。

        有关数据绑定的过程，我们可以简单概括如下。
        (1)在Web请求到达之后， Spring MVC某个框架类将提取当前Web请求中的所有参数名称， 然后遍历它，以获取对应每个参数的值，获取的参数名与参数值通常放入一个值对象(PropertyValue) 中 娥终我们将拥有所有需要绑定的参数和参数值的一个集合(Collection) 。

        (2)有了即将绑定到目标Cornmand 对象的数据来源之后， 我们即可将这些数据根据Command对象象中 各个域属性定义的类型进行数据转型， 然后设置到Command对象上。
        在这个过程中我们将碰到我们的老朋友BeanWrapperimpl, 还记得loC容器讲解内容中 Beanwrapper第一次登场的情景吗? Beanwrapperimpl会将Command对象纳入自身管理范围 ， 如 下所示：
        BeanWrapper beanWrapper = new BeanWrapperimpl(command); 
        然后比照参数名与Command对象的屈性对应关系，以进行参数值到Command对象屈性的设置， 而参数值与Command对象屈性间类型差异性的转换工作，则由BeanWrapperimpl所依赖的一系列自定义PropertyEd让or负责° 如果BeanWrapperimpl所使用的默认的PropertyEditor没有提供对某一种类型的支持，我们也可以添加自定义的PropertyEditor。这些内容你应该已经熟悉了，而我要澄淌的是，在绑定过程中，参数名称与Command对象屈性之间的对应关系是如何确 定的

    - 数据验证：核心类为 `validation.Validator`（不局限于 MVC 中使用，Spring中都可以）负责实现具体的验证逻辑。`validation.Errors` 负责承载验证过程中出现的错误信息。

    整个表单的处理流程，即 SimpleFormController 流程

    ![image-20200627125830489](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627125830489.png)

    - 在Web请求到达SimpleFormController之 后，SimpleFormController 将首先通过方法 isForrnSubmission(request)判明当前请求是否为表单提交请求。默认该方法实现就是 `return “POST".equals(request.getMethod())` 即只要以POST形式发送的Web请求就是表单提交，当然可以重写。
        - 如果为 false，表示为初次的 web 请求，需要为用户显示相应的交互表单，即**表单显示阶段**。
        - 如果为 true，则表示已经在表单中编辑完数据，准备提交，则启动**处理表单提交阶段**。

### 24.3 ModelAndView

Controller在将Web请求处理完成后，会返回一个ModelAndView实例。。该ModelAndView 实例将包含两部分内容，一部分为视困相关内容，可以是逻辑视图名称，也可以是具体的View实例；另一部分则是模型数据，视图渲染过程中将会把这些模型数据合并入最终的视图输出．所以，简单来说，ModelA ndView实际上就是一个数据对象。

**视图信息**

ModelAndView可以以逻辑视图名的形式或者Vi至w实例的形式来保存视图信息。如果ModelAndView中直接返回了具体的View实例，那么，Dispatcher­Servlet将直接从ModelAndView中获取该View实例并渲染视图。如果是返回逻辑视图名称就需要 ViewResolver 帮助，根据ModelAndView中的逻辑试图名称获取一个可用的View实例，然后再渲染视图。【该方式更加推荐，视图选择灵活性好】

**模型数据**

ModelAndView 中以 `ui.ModelMap`形式存放模型数据，通过构造方法或者其他方法将模型数据加入该 map 中，为所有的模型数据添加 key，这样就可以在视图渲染极端，具体的 view 实现类就可以获取数据使用了。一般键和视图模板中标识符一致。

### 24.4 视图定位器 ViewResolver

ViewResolver 接口的主要职责是，根据Controller所返回的 ModelAndView中的逻辑 视图名， 为DispatcherServlet返回一个可用的View实例。当然具体处理还是实现类实现。接口实现类只需要根据该接口中唯一方法 resolveViewName (String viewName, Locale locale)方法中以参数形式传入的逻辑视图名(viewName) 和当前Locale的值， 返回相应的View实例即可。

同样 DispatchServlet 接收多个 ViewResolver 处理视图的查找，和上面一样使用优先级。

### 24.5 各司其职 View



## 第 25 章：认识更多 SpringMVC 家族成员

![image-20200627154117815](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627154117815.png)

![image-20200627154138511](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627154138511.png)

### 25.1 文件上传与 MultipartResolver

MultipartResolver 是服务器端处理通过表单上传文件的主要组件。使用比较多的类库是 Commons FileUpload ，Spring MVC 底层也是这些，但是通过 MultipartResolver 接口的抽象使得我们拥有使用哪一种类库权利。

当Web请求到达DispatcherServlet并等待处理的时候，Dispatcherservlet首先会检查能否从自己的WebApplicationContext中找到一个名称为multipartResolver (由DispatcherServlet的常量（MULTIPART_RESOLVER_BEAN_NAME 所决定）的MultipartResolver实例。如果能够获得一个Multi­partResolver的实例， DispatcherServlet将通过MultipartResolver的isMultipart (request) 方法检查当前Web请求是否为multipart类型。 如果是， DispatcherServlet将调用Multipart­Resolver的resolveMultipart(request)方法，并返回一个MultipartHt tpServ letRequest供后继处理流程使用，否则， 直接返回最初的HttpServletRequest 。

具体执行还需要其对应的实现类完成，包含两个实现类：`web.multipart.commons.CommonsMultipartResolver` 和另一个，该实现类使用 Commons FileUpload 类库的实现，要启用 SpringMVC框架内的文件上传支持，本质上讲，就是选择这两个实现类中的哪一个， 然后将最终的选 择添加到Dispatcherservlet的WebApplicationContext（就是配置一个 bean 标签）。

### 25.2 Handler 和 HandlerAdaptor

HandlerMapping将会通过HandlerExecutionChain返回一 个 Object 类型的 Handler 对象（通常为 controller）用于具体Web请求的处理。因为不仅仅支持 Controller 这一种 Handler，不过， 对于DispatcherServlet来说，这就有点儿问题了，它如何来判断我们到底使用的是什么 类型的Handler , 又如何决定调用Handler对象的哪个方法来处理Web 请求呢？所以 DS  将不同 Handler 的调用职责转交给 HandlerAdaptor 。 该接口作为一个 **适配器**屏蔽不同 Handler 类型给 DS 带来的困扰。相当于充当了 DS 和 Handler 的中间人。

DispatcherServlet从HandlerMapping获得一个Handler之后，将询问 Handler Adaptor的supports(..)方法， 以便了解当前HandlerAdaptor是否支持HandlerMapping 刚刚返回的Handler类型的调用 。 如果supports (..)返回true, Dispa七cherServlet则调用 HandlerAdaptor的handle (.. l方法， 同时将刚才的Handler作为参数传入。 方法执行后将返回 ModelAndView, 之后的工作就由ViewResolver接手了。 所以 DS 只需要对 HA 提供统一接口即可。新增 Handler 只要实现 HA即可。

为具体的 Handler 类型提供一个 HandlerAdaptor 实现类比较简单，主要工作只是调用这个HandlerAdaptor "认识 ” 的Handler的Web请求处理方法，然后将处理结果转换为Dispatcher­Servlet统一使用的ModelAn扣iew就行。

- SimpleControllerHandlerAdapter是Controller对应的HandlerAdaptor实现。support() 方法决定它只认识 Controller，所以在 handler() 方法中直接将 Object 类型的 Handler 强制转换为 Controller，然后然后调用其handleRequest (..)即可。 因为Controller的handle­Request (..)方法可以返回已经组装好的ModelAndView, 所以， 就直接返回了．

#### 25.2.2 深入了解 Handler

除了 Controller，Spring MVC 提供了其他 Handler：`web.servlet.throwaway.ThrowawayController`。该 Handler 不依赖于任何 Servlet API，并且能够为他们定义状态，所以拥有很好的可测试性。

### 25.3 框架内处理流程拦截与 HandlerInterceptor

HandlerMapping返回的用于处理具体Web请求的Handler对象，是通过一个 HandlerExecutionChain 对象进行封装的。HandlerExecutionChain就是一个数据载体，它包含了两方面的数据，一个就是用于处理Web请求的Handler,一个则是 一组随同Handler 一起返回的HandlerInterceptor。这组 Handler Interceptor可以在Handler的执行前后对处理流程进行拦截操作。该 HandlerInterceptor 接口包括三个方法：

- boolean preHandle()。该拦截方法将在相应的HandlerAdaptor调用具体的Handler处理Web请求之前执行。如果返回 false 则后续所以流程不执行。
- void postHandler(..) 。 该拦截方法的执行时机为HandlerAdaptor调用具体的Handler处理完Web诮求之后，并且在视图的解析和渲染之前。通过该方法我们可以获取Handler执行后的结果，即ModelAndView。 我们可以在原处理结果的基础上对其进行进一步的后处理，比如添加新的统一的模型数据，或者对ModelAndView中的数据进行变更等。postHandle(..)返回类型为void, 不可以阻断后继处理流程 。
- void afterCompletion(..) 。 在框架内整个处理流程结束之后，或者说视图都渲染完了的时候，不管是否发生异常，aftercompletion(.. J拦截方法将被执行。同样如果 web 请求中包含资源也是在这里进行清理。

具体实现类包括：`web.servlet.handler.UserRoleAuthorizationInterceptor`，`web.servlet.mvc.WebContentInterceptor`。

**大多数情况需要自定义实现类**，一般不直接实现该接口，而是继承 HandlerInterceptorAdaptor 类。然后使用 bean 标签添加到 WebApplicationContext 中即可。

Handler Interceptor和Handler实际上 “本是同根生” 。如果我们从Handlerlnterceptor所处 的位置溯源而上（按照HandlerInterceptor -HandlerExecutionChain - HandlerMapping的顺 序）， 则会发现HandlerMapping是其最终的发源地， AbstractHandlerMapping作为几乎所有HandlerMapping实现类的父类， 提供了setInterceptors (..)方法以接受一组指定的Handler­Interceptor实例。 所以， 要使我们的Handlerinterceptor发挥作用， 只要将它添加到相应的 HandlerMapping即可。这些 指定的Handlerinterceptor将随同处理具体Web谐求的Handler 一起返回（以 HandlerExecutionChain的形式）， 并对Web谓求的处理流程进行拦截。

>对于 那些需要经常被扩展， 而又包含多个方法需要实现的接口声明，通常情况下，使用者并非每次 都希望提供所有接口方法的实现逻抖． 为了避免在实现这些接口的时候，每次都要去实现接口 定义中的所有方法， 对应API的设计者通常都会提供一个XXXAdaptor类专门用于子类化的需 要，避免实现所有接口方法的烦琐．

**其他选择**

因为 Spring MVC 是基于 Servlet API 构建，所以 Servlet 提供的规范也可以在 Spring MVC 中的 web 程序中使用，包括Servlet 的拦截组件 Filter。

![image-20200627173153959](Spring%20%E6%8F%AD%E7%A7%98.resource/image-20200627173153959.png)

Handler Interceptor位千DispatcherServlet 之后， 指定给HandlerMapping的它， 可以对 HandlerMapping所管理的多组映射处理关系进行拦截。最主要的是，Handlerinterceptor拥有更细粒度的拦截点。 我们可以在Handler执行之前，Handler执行之后， 以及整个DispatcherServlet内部 处理流程完成时点插入必要的拦截逻辑。 通过结合HandlerMapping的Chaining(t寺性，我们可以对不同HandlerMapping管理的多组不同的Handler, 应用不同的Handlerinterceptor进行处理流程的拦截处理，所以**针对细粒度的拦截请求**。

Filter通常被映射到Java Web应用程序中的某个servlet, 或者一组符合某 种URL匹配模式的访问资源上。 所以，从SpringMVC应用的结构上看，Filter位千DispatcherServlet 之前。 如果把Filter和Handlerinterceptor看作同一类型的拦截器， Filter将比Handlerinterceptor 拥有更高的执行优先级。不过，二者提供的拦截功能所加诸千上的目标对象却完全是不同级别： Filter 序列在servlet层面对 DispatcherSer vlet进行拦截， 而HandlerInterceptor则位千Dispatcher­Servlet内部，对Handler的执行进行拦截。 Filter的应用位置注定了它不能够提供细粒度的拦截时点， 所以， 通常情况下， 使用Filter对千Web应用程序中的一些普遍关注点进行统一处理是比较适合的， 一 旦需要细化处理流程的拦截逻辑， 可以再转而求助千Handlerinterceptor。

同样因为 Filter 最为 Servlet 的标准组件，需要在 web.xml 中配置，其生命周期更多的有 web 容器管理，为了避免过去耦合的绑定，能够采用依赖注入实现，Spring MVC 引入了 `web.filter.DelegatingFilterProxy` 作为—个Filter的Proxy对象， 当真正需要执行拦截操作的时候， 它将把具体的工作委派给它所对应的一个Filter委派对象。



#### 25.4 框架内的异常处理和 HandlerExceptionResolver



## 第 26 章：SpringMVC 中基于注解的 Controller

传统的Handler类型， 比如Controller或者ThrowawayController, 都需要具体实现类继承某个基类或者实现某个接口。 而使用基于注解的Controller的话， 则没有这样的限制。 实际上， 基千注解的controller就是一个普通的POJO, 只是使用某些注解附加了一些相关的元数据信息而已。 通常就是加上：

```java
@controller 
@RequestMapping (“/helloAnnoController.sanno”)
```

传统的Controller或者ThrowawayController通过接口类型作为标志的方式 ， 基于注解的 Controller则采用标注千具体实现类上的某种类型的注解作为标志。只需要在 WebApplicationContext 中加上`<context:component-scan base-package=•your.controller.package"/> ` 即可。从而找到处理上面路径对应的处理请求的 Handler，并调用其标注了 @RequestMapping 的 方法进行当前 web 请求的处理。



**怎么知道当前请求对应的 Handler，然后怎么知道调用哪个具体方法来处理**。即需要为基千注解的Controller提供相应的HandlerMapping以处 理Web请求到Handler的映射关系 ， 需要提供相应的HandlerAdaptor以调用并执行自定义handler的 处理逻辑

#### 自定义基于注解的 Controller 的 HandlerMapping

@RequestMapping 中的注解信息需要通过 Java 反射机制获取，但是上面的 BeanNameUrlHandlerMapping还是SimpleUrl­HandlerMapping, 显然都没有提供通过反射读取注解中映射信息的功能。需要自定义。

自定义实现类大概逻辑：所要作的只是遍历所有可用的基千注解的Con七roller实现类，然后根据诮求的路径信息，与实 现类中的注解所标注的请求处理映射信息进行比对。如果当前基于注解的Controller实现类的注解 所包含的信息与请求的路径信息相匹配，那么就返回当前这一基千注解的Controller实现类即可。

官方提供了一个 HandlerMapping 结构的 `web.annotation.DefaultAnnotationHandlerMapping` 实现类，该来首先扫描应该程序的 Classpath（通过 `<context:component-scan/>` 完成），然后通过反射获取所有标注了 @Controller 的对象，然后就是 HandlerMapping 其他实现类一样的功能了。

#### 自定义基于注解的 Controller 的 HandlerAdaptor

HandlerMapping返回了处理Web请求的某个基千注解的Controller实例， 而Dispatcher­Servlet并不知道应该去调用该实例的哪个方法来处理Web请求。 为了能够以统一的方式调用各种类 型的Handler, DispatcherSe切let需要一个针对基千注解的Controller的HandlerAdaptor实现。 该类 通过反射查找标注了@RequestMapping的方法定义， 然后通过反射调用 该方法， 并返回DispatcherServlet所需要的ModelAndView即可。

## 第 27 章：SpringMVC 拓展篇

