---
style: summer
flag: red
---

# Part1：Spring 核心知识
@toc

## 章一：Spring 之旅

Spring 实现简化企业级开发（EJB）的底层功能依赖于两大核心特性：依赖注入（Dependency Injection，DI）和面向切面编程（aspect-oriented programming，AOP）

### 一、简化 Java 开发
使用的四种关键策略：
- 基于 POJO 的轻量级和最小侵入性编程；
spring 不会强迫实现 Spring 规范的接口或者继承 Spring 规范的类，从而避免框架通过强迫应用继承他们的类或者实现他们的接口从而导致应用和框架绑死；非侵入式编程意味着该类在 Spring 应用中和非 Spring 应用中都可以发挥同样的作用，Spring 仅仅通过 DI 来装配 POJO；
- 通过依赖注入和面向接口实现松耦合；
用于实现保持对象直接的松耦合，
- 基于切面和惯例进行声明式编程；
DI 能够让相互协作的软件组件（Bean）保持松散耦合，而面向切面编程允许将遍布应用各处的功能分离出来形成可以重用的组件。通常针对跨越系统的多个组件，例如日志、事务管理、安全等系统服务，不应该融入到系统自身的核心业务逻辑中。原来做法是各个模块需要上述功能就实现对应的方法，或者上述方法抽象出来再需要的地方进行调用，本质都是重复出现。应该只关注核心功能，不关注上述功能；
![采用AOP功能前后对比]($resource/%E9%87%87%E7%94%A8AOP%E5%8A%9F%E8%83%BD%E5%89%8D%E5%90%8E%E5%AF%B9%E6%AF%94.png)

- 通过切面和模板减少样板式代码；
例如使用 JDBC 来访问数据库查询数据，需要进行样板式的配置（连接、查询、释放。。）。Spring 通过模板封装来消除样板式代码，例如可以使用 Spring 的 JdbcTemplate 代替 JDBC API，专注于书写逻辑 SQL ；

### 二、容纳你的 Bean

在基于 Spring 应用中，**应用对象是生存于 Spring 容器中**，Spring 容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期。spring 容器使用 DI 管理构成应用的组件，它会创建相互协作的组件之间的关联。
**Spring 自带多个容器实现**，主要是两类：
- Bean 工程：由`org.springframework.beans.factory.BeanFactory`接口定义，提供基本的 DI 支持；
- 应用上下文：由`org.springframework.context.ApplicationContext`接口定义，基于 BeanFactory 构建，提供应用框架级别的服务，通常使用这里；自带的多种应用上下文见下：
  *   AnnotationConfigApplicationContext;：一个或多个基于 Java 的配置类中加载 Spring 应用上下文；
  *   AnnotationConfigWebApplicationContext;一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文；
  *   ClassPathXmlApplicationContext; 在**类路径**下一个或多个 XML 配置文件中加载上下文定义，把应用上下文的定义作为类资源；
  *   FileSystemXmlApplicationContext;：在**文件系统**下一个或多个 XML 配置文件中加载上下文定义；
  *   XmlWebApplicationContext；从 web 应用下的一个或多个 XML 配置文件中加载上下文定义；

- FileSystemXmlApplicationContext和ClassPathXmlApplicationContext的区别在于：FileSystemXmlApplicationContext在**指定文件系统路径下查找**，而ClassPathXmlApplicationContext在**所有的类路径（包括JAR文件）下**查找。

如果想从Java配置中加载应用上下文，可以使用AnnotationConfigApplicationContext。

- 示例：
如何加载一个FileSystemXmlApplicationContext（从文件路径加载）：
 `ApplicationContext context =  new  FileSystemXmlApplicationContext("c:/knight.xml");`

使用ClassPathXmlapplicationContext加载（从类路径加载）：
 `ApplicationContext context =  new  ClassPathXmlapplicationContext("knight.xml");`

从java配置类加载应用上下文：
 `ApplicationContext context =  new  AnnotationConfigApplicationContext(com.springinaction.knights.config.KnightConfig.class);`

 应用上下文准备就绪后，可以调用上下文的getBean()方法从Spring容器中获取Bean
**所以说应用上下文就是一个 Spring 容器**


#### （二）**bean的生命周期**

使用new对bean进行实例化，一旦不再被使用，将由Java自动进行垃圾回收。
![Bean 的生命周期]($resource/Bean%20%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

1.  Spring对Bean进行实例化；
2.  Spring将值和Bean的引用注入进Bean对应的属性中；
3.  如果Bean实现了`BeanNameAware`接口，Spring将Bean的ID传递给`setBeanName()`接口方法；
4.  如果Bean实现了`BeanFactoryAware`接口，Spring将调`setBeanFactory()`接口方法，将BeanFactory容器实例传入；
5.  如果Bean实现了`ApplicationContextAware`接口，Spring将调用`setApplicationContext()`接口方法，将应用上下文的引用传入；
6.  如果Bean实现了`BeanPostProcessor`接口，Spring将调用`postProcessBeforeInitialization()`接口方法；
7.  如果Bean实现了`InitializationBean`接口，Spring将调用`afterPropertiesSet()`方法。类似的如果Bean使用了`init-method`声明了初始化方法，该方法也会被调用；
8.  如果Bean实现了`BeanPostProcessor`接口，Spring将调用`ProcessAfterInitialization()`方法；
9.  此时此刻，Bean已经准备就绪，可以被应用程序使用了，它们将一直`驻留在应用上下文中`，直到该应用上下文被销毁；
10.  如果Bean实现了`DisposableBean`接口，Spring将调用`destory()`方法，同样的，如果Bean中使用了`destroy-method`声明了销毁方法，也会调用该方法；

### 三、俯瞰 Spring 整体

![Spring 模块]($resource/Spring%20%E6%A8%A1%E5%9D%97.png)

1.  Spring核心容器
    是Spring框架中最核心的部分，管理Spring应用中bean的创建、配置和管理。在此模块中包含了Spring bean工厂，为Spring提供了DI功能。
2.  Spring的AOP模块
    是Spring应用系统中开发切面的基础。与DI一样，AOP可以帮助应用对象解耦。借助AOP可以将遍布系统的关注点（如事务和安全）从应用对象中解耦出来。
3.  数据访问与继承
    使用JDBC编写代码通常导致大量的样板式代码。Spring的JDBC和DAO（Data Access Object）模块抽象了这些样板式代码。还在多种数据库服务的错误信息上构建了一个语义丰富的异常层。同样 Spring 的 ORM 模块是建立在 DAO 的支持之上的，Spring 的事务管理支持所有的 ORM 框架和 JDBC。
4.  Web和远程调用：
    MVC（Model-View-Controller）模式榜知用户将界面逻辑和应用逻辑分离。（SpringMVC）
5.  Instrumentation模块提供了为JVM添加代理的功能。
6.  测试
    Spring为使用JNDI、Servlet和Portlet编写单元测试提供了一系列的mock对象实现。


### 四、Spring 新功能
。。。。




## 章二：装配 Bean

原来创建应用对象之间的关联关系通常使用构造器或者查找，导致代码结构复杂。**在Spring中，对象无需自己查找或创建与其所关联的其他对象。相反，容器负责把需要相互协作的对象引用赋予各个对象。**

**创建应用对象之间协作关系的行为通常称为装配**（wiring），这也是依赖注入的本质。

###  一、 Spring配置的可选方案
Spring 容器负责创建应用程序中的 Bean 并通过 DI 来协调这些对象之间的关系，但是得告诉 Spring **要创建哪些 Bean 并且如何将他它们装配到一起**。
Spring提供了三种主要的装配机制（可以互相搭配使用）：

*   在XML中进行显式配置；
*   在Java中进行显式配置；
*   隐式的bean发现机制和自动装配。

建议**尽可能的使用自动装配的机制**。显式配置越少越好，**当必须要显式配置bean的时候**（比如，有些源码不是由你来维护的，而当你需要为这些代码配置bean的时候），**推荐使用类型安全并且比XML更加强大的JavaConfig**。最后，只有当你想要使用便利的XML命名空间，并且在JavaConfig中没有同样的实现是才应该使用XML。

### 二、 自动化装配bean

Spring从两个角度实现自动化装配：
*   组件扫描（component scanning）：Spring会自动发现应用上下文中所创建的bean；
*   自动装配（autowiring）：Spring自动满足bean之间的依赖。

#### （一）创建可被发现的bean

在类上使用@Component注解，这个注解表明该类会作为组件类，并告知Spring要为这个类创建bean。
示例：
定义一个CD机的类：
```java
package a_autoConfig;

/**
 * CD 接口，定义 CD 播放器对一盘 CD 能做的操作，降低 CD播放器的任意实现对 CD 本身的耦合
 */
public interface CompactDisk {
    void play();
}
```
带有@Component注解的CompactDisc实现类SgtPeppers
```java
package a_autoConfig;

import org.springframework.stereotype.Component;

/**
 *  @Component 注解告诉 Spring 该类会作为组件类，同时要为该类创建 Bean
 */
@Component
public class SgtPeppersDisk implements CompactDisk {

    private String title = "Sgt. Pepper's  Club Band";
    private String artist = "The Beatles";

    @Override
    public void play() {
        System.out.println("通过a_autoConfig显示：Playing " + title + " by " + artist);
    }
}
```
组件扫描默认是不启用的，还需要显式配置一下Spring，从而命令它去寻找带有@Component注解的类并为其创建bean。

```
package a_autoConfig;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * 因为默认组件扫描不启用，需要显式配置 Spring 命令其需要带有 @Component 注解的类，并且为其创建 Bean
 * 这里 CDPlayerConfig 类没有显式的声明任何 Bean，仅仅使用 @ComponentScan 注解来在 Spring 中启动组件扫描，
 *      这里扫描注解中是有参数的，如果没有配置则默认扫描：与配置类相同的包以及该包下所有的子包中带 @Component 的类
 */
@Configuration
@ComponentScan("a_autoConfig")
public class CDPlayerConfig {
}
```
 上面是使用 Java 代码定义 Spring 的装配规则， 如果没有其他配置，**@ComponentScan默认会扫描与配置类相同的包以及该包下所有子包，查找所有带有 @Component 注解的类**。

上面对应的，如果想使用XML来启用组件扫描，可以使用Spring context命名空间的`<context:component-scan>`元素。
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:Context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframeword.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context.xsd">

        <context:component-scan base-package="soundsystem"/>
</beans>

```

为了配合测试的一些类
```MediaPlay_java
package a_autoConfig;

public interface MediaPlayer {
    void play();
}
```
上面接口的实现类
```java
package a_autoConfig;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer implements MediaPlayer {

    @Autowired
    private CompactDisk cd;

    @Override
    public void play() {
        cd.play();
    }
}
```

测试组件扫描代码（即看配置是否起作用）：
```
package a_autoConfig;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static org.junit.Assert.assertNotNull;

/**
 * CDPlayerTest 使用了 Spring 的 `SpringJUnit4ClassRunner`，以便在测试开始的时候自动创建Spring的应用上下文。
 * 注解@ContextConfiguration会告诉它需要在CDPlayerConfig中加载配置。
 * 因为 CDPlayerConfig 类中包含了 @ComponentScan，因此最终的应用上下文中应该包含 CompactDisc  Bean；
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CDPlayerConfig.class)
public class A_CDPlayerTest {

    // MediaPlay 的实现类中包含 CompactDisc 的一个属性
    // @Autowired 会将 CompactDisc bean 注入到该测试代码中
    @Autowired
    private MediaPlayer mediaPlayer;

    @Test
    public void playCD(){
        mediaPlayer.play();
    }

    // 测试方法二：直接看 CompactDisc 对象有没有被注入
    @Autowired
    private CompactDisk compactDisk;
    @Test
    public void compactDiskShouldNotBeNull(){
        assertNotNull(compactDisk);
    }
}

```

CDPlayerTest使用了Spring的 `SpringJUnit4ClassRunner`，以便在测试开始的时候自动创建Spring的应用上下文。注解@ContextConfiguration会告诉它需要在CDPlayerConfig中加载配置。因为 CDPlayerConfig 类中包含了 @ComponentScan，因此最终的应用上下文中应该包含 CompactDisc  Bean；

#### （二） 为组件扫描的bean命名

在组件扫描中如果没有明确的设置ID，Spring会根据类名为其指定一个ID。**这个bean所给定的ID是首字母变为小写的类名**。即下面类 SgtPeppers 如果使用 `@Component` 和使用 `@Component(“sgtPeppers”)` 一样。

如果想为这个bean设置不同的ID，需要将其作为值传递给@Component注解。

```
@Component("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc {
}

```

另外一种bean命名方式使用Java依赖注入规范中提供的@Named注解：**不推荐**，就是将上面的 @Component 换成 @Named 即可；

Spring支持将@Named作为@Component注解的替代方案。两者之间由一些细微的差异，但在大多数场景中可以互换。


#### （三） 设置组件扫描的基础包

如果没有为@ComponentScan设置属性，按照默认规则，它会以配置类所在的包作为基础包来扫描逐渐。

如果**想扫描不同的包，在@ComponentScan的value属性中指定包名即可**。

```
@Configuraion
@ComponentScan("soundsystem")
public class CDPlayerConfig{
}

```

如果想更加清晰的表明所设置的是基础包，可以通过basePackages属性进行配置：

```
@Configuration
@ComponentScan(basePackages="soundsystem")
public class CDPlayerConfig{}

```

basePackages属性可以设置多个包，赋值一个数组即可：例如：`@ComponentScan(basePackages={"soundsystem", "video"})`

除了将包设置为简单的String类型外，@ComponentScan 还提供了另外一种方法，将**其指定为包中所包含的类或接口**：**这样也会扫描该类（接口）所在包**，即该类所在的包将成为组件扫描的基础包。推荐在扫描的包中新建一个空的接口，用于被扫描；
**使用下面这种**
```
@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})
public class CDPlayerConfig { }

```

#### （四） 通过为bean添加注解实现自动装配

自动装配：让 Spring 自动满足 Bean 依赖的一种方法，在满足依赖的过程中，会在 Spring 应用上下文中寻找匹配某个 Bean 需求的其他 Bean；使用 @Autowired 来声明要进行自动装配；

```java
package a_autoConfig;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer implements MediaPlayer {
    pricate CompactDisc cd;

    // 该注解表明：当 Spring 创建 CDPlayer 时候，会通过该构造器进行实例化并且传入一个可以设置给 CompactDisc 类型的 Bean
    @Autowired
    public CDPlayer(CompactDisc cd){
      this.cd = cd;
    }

    @Override
    public void play() {
        cd.play();
    }
}
```

* **可以在构造器、setter方法、任何方法上使用@Autowired注解**，Spring会尝试满足方法参数上所声明的依赖。假如有且只有一个bean匹配依赖需求，那么这个bean将会被装配进来。

* 如果没有匹配的bean，那么在应用上下文创建的时候会抛出异常。可以将@Autowired的required属性设置为false`@Autowired(required=false)`，此时Spring会尝试自动装配，但是如果没有匹配的bean的话该属性将会为null。就是不装配也不会报异常，这样使用时候可能出错 NPE 问题。

* 如果有多个bean都满足依赖关系，Spring将会抛出异常，因为没有指定那个 bean 。

* @Autowired是Spring特有的注解。可以使用@Inject注解替代（但是不建议使用）。


### 三、 通过Java代码装配Bean

**为什么进行显式装配**
主要针对将第三方库中的组件装配到自己的应用中，是没有办法在他们的类中加上 @Component 和 @Autowired 注解的，就不能使用自动化装配了。

显式的装配方式有两种：Java和XML。一般用于将第三方库中的组件装配到自己的应用中。

**推荐 Java 配置，因为类型安全且对重构友好**
同时 JavaConfig 是配置代码，因此不能有任何的业务逻辑，不能侵入业务逻辑代码中，一般放在单独的包中，使得和应用程序逻辑相分离。


**配置过程如下：**
#### （一）创建配置类

@Configuration注解表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节。
```java
package soundsystem
import org.springframework.context.annotation.Configuration;

@Configuration
public class CDPlayerConfig{
}
```

前面都是使用组件扫描（即在上面类上加上 @ComponentScan 注解）来发现 Spring 应该创建的 Bean，下面使用显示配置方式；

#### （二）声明简单bean

要在JavaConfig中声明bean，需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加@Bean注解。
```java
@Bean(name="设置另一个名字")
public CompactDisc sgtPeppers(){
    // 这里只要确认该方法体返回一个新的 SgtPerppers() 实例结课，可以添加其他的逻辑
    return new SgtPeppers();
}
```


#### （三） 借助JavaConfig实现注入
**这里就是将 CompactDisc 注入到 CDPlayer 中**，因为这里的 CDPlayer bean 依赖于 CompactDisc。

在JavaConfig中装配bean的最简单方式就是引用创建bean的方法。在方法上添加了@Bean注解，Spring将会拦截所有对它的调用，并确保直接返回该方法所创建的bean，而不是每次都对其进行实际的调用。

通过调用方法引用bean的方式有点令人困惑。还有一种理解起来更简单的方式：

```
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
    return new CDPlayer(compactDisc);
}
```

cdPlaye()方法请求一个CompactDisc作为参数，当Spring调用cdPlayer()创建CDPlayer bean的时候，它会自动装配一个CompactDisc到配置方法之中。

通过这种方式引用其他的bean通常是最佳的选择，因为它不会要求将CompactDisc声明到同一个配置类之中。在这里甚至没有要求CompactDisc必须要在JavaConfig中声明，实际上它可以通过组件扫描功能自动发现或者通过XML来进行配置。可以将配置分散到多个配置类、XML文件以及自动扫面和装配bean之中。

代码示例：
```CompactDisk_java
package b_javaConfig;

public interface CompactDisk {
    void play();
}
```

```SgtPeppersDisk_java
package b_javaConfig;

import lombok.Getter;
import lombok.Setter;
import org.springframework.stereotype.Component;

@Getter
@Setter
@Component
public class SgtPeppersDisk implements CompactDisk {

    private String title = "Sgt. Pepper's Lonely Hearts Club Band";
    private String artist = "The Beatles";

    @Override
    public void play() {
        System.out.println("通过b_javaConfig显示：Playing " + title + " by " + artist);
    }
}
```

```Media_java
package b_javaConfig;

public interface MediaPlayer {
    void play();
}
```

```CDPlayer_java
package b_javaConfig;

import lombok.Getter;
import lombok.Setter;

@Setter
@Getter
public class CDPlayer implements MediaPlayer {

    private CompactDisk cd;

    public CDPlayer(CompactDisk cd) {
        this.cd = cd;
    }

    @Override
    public void play() {
        cd.play();
    }

    public void setCompactDisc(CompactDisk compactDisk) {
    }
}
```

```CDPlayerConfig_java
package b_javaConfig;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by yangjing on 2018/1/2
 */
@Configuration
public class CDPlayerConfig {

    /**
     * @Bean 注解会告诉Spring这个方法将返回一个对象，该对象要注册为Spring应用上下文中的bean。方法体中包含了最终产生bean实例的逻辑。
     * 默认情况下，bean的ID与方法名一致。如果想为其设置一个不同的名字，可以重命名该方法，也可以通过@Bean的name属性指定。
     * @return
     */
    @Bean(name = "hello")
    public CompactDisk sgtPerppersDisk(){
        return new SgtPeppersDisk();
    }

    /**
     * 方案一：将 CompactDisk 装配到 CDPlayer 中最简单方式就是 引用创建 Bean 的方法;
     * 该方法也使用 @Bean 表示该方法会创建一个 Bean 实例并将其注册到 Spring 上下文中，其 BeanID 即为类名；
     * 这里调用了需要传入 CompactDisk 对象（sgtPerppersDisk 是 CompactDisc 的实现类）的构造器 来创建 CDPlayer 实例，
     *      但是这里的 CompactDisc 对象不是通过 SgtPeppersDisk() 得到的，因为 SgtPeppersDisk() 方法上也有 @Bean，
     *      因此 Spring 会拦截所有对他的调用，并确保直接返回该方法创建的 bean，而不是每次都对其进行实际的调用。
     * @return
     */
    @Bean
    public MediaPlayer mediaPlayer(){
        return new CDPlayer(sgtPerppersDisk());
    }

    /**
     * 默认 Spring 中的 bean 都是单例的，
     * 接上面：这里不会每次都调用 sgtPerppersDisk()，Spring 会拦截对 sgtPerppersDisk() 的调用，并确保返回的是 Spring 所创建的 Bean，
     *   即 Spring 本身在调用 sgtPerppersDisk() 时所创建的 CompactDisc bean，因此两个 MediaPlayer bean 会得到相同的
     *   SgtPeppersDisk 实例。
     * @return
     */
    @Bean
    public MediaPlayer anotherMediaPlayer(){
        return new CDPlayer(sgtPerppersDisk());
    }

    /**
     * 方式二：当 Spring 调用 cdPlayer() 创建 CDPlayer bean 的时候，会自动装配一个 CompactDisc
     * 到配置方法中，然后方法体就可以按照合适的方法使用它；
     *  这样 cdPlayer() 方法也能将 CompactDisc 注入到 CDPlayer 的构造器中，不用明确引用 CompactDisc 的 @Bean 方法；
     *  优势：ComponentDisc 不要求将其声明到同一个配置类，甚至不一定要求使用 JavaConfig 声明，可以采用组件扫描自动发现或者xml配置都行。
     * @param compactDisk
     * @return
     */
    @Bean
    public MediaPlayer mediaPlayer(CompactDisk compactDisk) {
        return new CDPlayer(compactDisk);
    }

    /**
     * 方式二的另一种 DI方式，上面通过构造器实现 DI 功能，下面通过 Setter 方法实现注入 CompactDisc
     * 重点：带有 @Bean 注解的方法可以采用任何必要的 Java 功能产生 bean 实例即可，不一定仅仅使用构造器和 setter 方法；
     * @param compactDisk
     * @return
     */
    @Bean
    public MediaPlayer mediaPlayer(CompactDisk compactDisk){
        CDPlayer cdPlayer = new CDPlayer(compactDisk);
        cdPlayer.setCompactDisc(compactDisk);
        return cdPlayer;
    }
}


```

### 四、通过XML装配Bean

#### （一）创建XML配置规范

在XML配置中，要创建一个XML文件，并且要以<beans>元素为根。

```
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframeword.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context">

        <!-- 配置详情 -->
</beans>

```

#### （二）声明一个简单的<bean>
**Spring XML 配置中只有一种 ==声明==  bean 的方式，使用 <bean> 元素并指定 class 属性；**
<bean>元素类似于JavaConfig中的@Bean注解。
`<bean id="compactDisc" class="soundsystem.SgtPeppers">`

- 如果没有给定ID，所以这个bean将会根据全限定类名来进行命名。例如上面的 Bean Id 就是：`soundsystem.SgtPeppers#0`，其中`#0`是一个计数形式，用来区分相同类型的其他bean，例如声明了另一个 SgtPeppers，且没有设置 Id 属性，则会自动设置 Id 为：`soundsystem.SgtPeppers#1`。

- 这里不再需要直接负责创建 SgtPeppers 的实例（基本的 JavaConfig 中需要），这里 Spring 发现 `<bean>`这个元素时候，会调用 SgtPeppers 的默认构造器来创建 bean，虽然更加被动但是不能像 JavaConfig 中一样以自己的方式创建 Bean 实例。

#### （三）借助构造器注入初始化bean

构造器注入有两种基本的配置方案可供选择：

*   `<constructon-arg>` 元素；
*   使用Spring 3.0 所引入的c-命名空间。

-  构造器注入bean引用
**这里针对的是类型的装配**，即将对象的引用装配到依赖于他们的对象中；
上面通过`<bean>` 标签声明了 CompactDisc 接口的实现类 SgtPeppersDisk，下面将 CompactDisc 依赖注入到 CDPlayer 中
当 Spring 遇到 <bean> 元素之后，会创建一个 CDPlayer 实例，<constructor-arg> 元素会告知 Spring 要将一个 Id 为 compactDisc 的 bean 引用传递到 CDPlayer 的构造器中。
```
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <constructor-arg ref="compactDisc"/>
</bean>

```

c-命名空间的schema:加上下面这个：`xmlns:c="http://www.springframework.org/schema/c"`
然后在 xml 文件中配置：
```
<bean id="cdPlayer" class="soundsystem.CDPlayer" 
    c:cd-ref="compactDisc" />
// 含义： c-命名空间的前缀:装配的构造器的参数名-表示装配一个 bean 引用="要注入的 bean 的 ID"
```

可以使用参数（针对构造器的参数名）在整个参数列表中的位置来替代参数名，这里使用 `0`表示第一个参数：因为在XML中不允许数字作为属性的第一个字符，因此必须要添加一个下划线作为前缀。`c:_0-ref="compactDisc"`

如果只有一个参数，不用标示参数：`c:_-ref="compactDisc" `


- 将字面量注入到构造器中

这里示例是 CompactDisc 的新的实现类，里面的参数不是直接附字符串的，因此需要将值注入

```BlankDisk_java
package c_xmlConfig;

import java.util.List;

public class BlankDisk implements CompactDisk {

    private String title;
    private String artist;
    private List<String> tracks;

    public BlankDisk(String title, String artist, List<String> tracks) {
        this.title = title;
        this.artist = artist;
        this.tracks = tracks;
    }

    @Override
    public void play() {
        System.out.println("通过c_xmlConfig显示：");
        System.out.println("Playing " + title + " by " + artist);
        for (String track : tracks) {
            System.out.println("-Track：" + track);
        }
    }
}

```

使用构造器进行注入：
```
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <!--value 属性表示给定的 value 值将以字面量的形式注入到构造器中-->
    <constructor-arg value="sgt.Pepper" />
    <constructor-arg value="sgtbags" />
</bean>
```

使用c-命名空间装配字面量与装配引用的区别在于属性名中去掉了-ref后缀。
方案一：引用构造器参数名字
方案二：将参数名换成数字 0/1.....(利用位置)
```
<bean id="compactDisc" class="soundsystem.BlankDisc"
    c:_title="sgt. Pepper"
    c:_artist="sgtbags"/>
```

- 装配集合

**<constructor-arg>可以将集合装配到构造器参数重，而c-命名空间不能**。

可以使用<null/>元素将null传递给构造器，方式是：`<constructor-arg><null/></constructor-org>`。

可以使用<list>元素声明一个列表：
```
<constructor-arg>
    <list>
        <value>"CCCC"</value>
        <value>"张三"</value>
        ....
    </list>
</constructor-arg>
```

<list>元素是<constructor-arg>的子元素，这表明一个包含值的列表将会传递到构造器中。其中<value>元素用来指定列表中的每个元素。

与之类似，可以使用<ref>元素代替<value>，**实现bean引用列表的装配**。例如 参数是`List<CompactDisc> cd`，则引用是
```xml
<!--前面配置省略-->
<constructor-arg>
    <list>
      <ref-bean="sgtPeppersDisk"/>
      <ref-bean="blackDisk"/>
      ....
    </list>
</constructor-arg>
```

可以用相同的方式使用<set>元素。<set>和<list>元素的区别不大，无论在哪种情况下，<set>或<list>都可以用来装配List、Set甚至是数组。只是 set 会忽略重复的值，同时保证存放顺序。

#### （四）设置属性

新的代码示例：

```java
package c_xmlConfig;

public class CDPlayer implements MediaPlayer {

    private CompactDisk cd;

    public CDPlayer(CompactDisk cd) {
        this.cd = cd;
    }

    @Override
    public void play() {
        cd.play();
    }
}
```

**使用构造器注入还是使用属性注入**

- 强依赖，例如 BlankDisc 中都是强依赖，使用构造器注入；
- 弱依赖：移入 CompactDisc 属于弱依赖，因为即使没有该依赖注入，CDPlayer 类仍具备一定的功能；

注入方式：

```xml
<bean id ="cdPlayer" class="soundsystem.CDPlayer">
	<property name = "cd" ref="compactDisc"/>
</bean>
```

<property>元素为属性的setter方法所提供的功能与<constructor-arg>元素为构造器提供的功能是一样的，本例子中通过引用 id 为 compactDisc 的 bean 来注入到 cd 属性中。**通过 setCompactDisc() 方法**

Spring提供了p-命名空间作为<property>元素的替代方案。p-命名空间的schema：

```xml-dtd
xmlns:p="http://www.springframework.org/schema/p"
```

对应的装配方式：

```xml
<bean id ="cdPlayer" class="soundsystem.CDPlayer" 
    p:cd-ref="compactDisc"/>

```

#### (五)将字面量注入属性中

新的代码示例为：

```java
package c_xmlConfig;

import java.util.List;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class BlankDisk implements CompactDisk {

    private String title;
    private String artist;
    private List<String> tracks;
   
    @Override
    public void play() {
        System.out.println("通过c_xmlConfig显示：");
        System.out.println("Playing " + title + " by " + artist);
        for (String track : tracks) {
            System.out.println("-Track：" + track);
        }
    }
}
```

注入方式：

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
	<property name="title" value="sdfjkla"></property>
    <property name="artist" value="sdfjkla"></property>
    <property name="tracks">
    	<list>
        	<value>slkgdjlksd</value>
            <value>jfldskjga</value>
        </list>
    </property>
</bean>
```

如果使用 P 标签的：

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc"
      p:title="dfjdsljgf"
      p:artist="kdjgfdsjg">
    <property name="tracks">
    	<value>djfjdsfj</value>
        <value>djgfgjkdajg</value>
    </property>
</bean>
```

util-命名空间所提供的功能之一是[util:list](util:list)元素，它会创建一个列表的bean。

```xml-dtd
xmlns:util="http://www.springframework.org/schema/util"
```

```
<util:list id="trackList">
    <value>......</value>
    <value>.....</value>
</util:list>
```

下表展示了Spring util-命名空间中的元素

| 元素 | 描述 |
| --- | --- |
| [util:constant](util:constant) | 引用某个类型的public static域，并将其暴露为bean |
| [util:list](util:list) | 创建一个java.util.List类型的bean，其中包含值或引用 |
| [util:map](util:map) | 创建一个java.util.Map类型的bean，其中包含值或引用 |
| [util:properties](util:properties) | 创建一个java.util.Propertie类型的bean |
| [util:property-path](util:property-path) | 引用一个bean的属性（或内嵌属性），并将其暴露为bean |
| [util:set](util:set) | 创建一个java.util.Set类型的bean，其中包含值或引用 |

### 五、导入和混合配置

#### （一）在JavaConfig中引用XML配置

使用@Import注解引用其他JavaConfig配置，使用@ImportResource注解引入XML配置。

```
@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {

}

```

#### （二）在XML配置中引用JavaConfig

在XML中可以使用<import>元素来引用其他XML配置。

将JavaConfig作为bean使用<bean>元素导入XML配置文
