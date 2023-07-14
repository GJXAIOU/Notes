## 1.1、简化Java开发

创建 Spring 的主要目的是用来替代更加重量级的企业级 Java 技术，尤其是 EJB。相对 EJB 来说，Spring 提供了更加轻量级和简单的编程模型。它增强了简单老式 Java 对象（Plain Old Java Object，POJO），使其具备了之前只有 EJB 和其他企业级 Java 规范才具有的功能。

为了降低 Java 开发的复杂性，Spring 采取了以下 4 种关键策略：

- 基于 POJO 的轻量级和最小侵入性编程

- 通过依赖注入和面向接口实现松耦合

- 基于切面和惯例进行声明式编程

- 通过切面和模板减少样式代码


### 1.1.1、Spring 赋予 POJO 魔力

方式之一就是**通过 DI 来装配它们。帮助应用对象彼此之间保持松散耦合** ==》DI 最核心作用

### 1.1.2、依赖注入

> 依赖注入和 Spring 的依赖注入区别
>
> 依赖注入（Dependency Injection，DI）是一种设计模式，它通过将依赖对象的创建和管理责任转移到外部，从而降低了类之间的耦合度。DI 的目的是通过将依赖关系从代码中移除，使得代码更具可测试性、可维护性和可扩展性。
>
> Spring 是一个广泛使用的 Java 开发框架，其中的依赖注入是其核心特性之一。Spring 的依赖注入是通过容器（ApplicationContext）来实现的，它通过配置文件（如 XML 配置文件）或注解（如 `@Autowired`）的方式来描述组件之间的依赖关系，然后由 Spring 容器在运行时动态地注入依赖对象。
>
> 区别如下：
>
> 1. 实现方式：依赖注入是一种设计模式，可以用多种方式实现，而 Spring 是一个框架，提供了依赖注入的实现。
> 2. 范围：依赖注入是一种概念或设计原则，不限于特定的编程语言或框架，可以应用于各种编程语言和环境。而 Spring 的依赖注入是针对 Java 开发的，特别是针对 Spring 框架。
> 3. 配置方式：依赖注入可以通过手动编码来实现，也可以使用框架或容器来自动完成。Spring 的依赖注入主要通过配置文件或注解来描述依赖关系，由 Spring 容器负责解析配置并完成依赖注入。
> 4. 功能和扩展：Spring 的依赖注入不仅提供了基本的依赖注入功能，还包括了更多的特性和扩展，如依赖的生命周期管理、AOP（面向切面编程）支持等。
>
> 总的来说，依赖注入是一种通用的设计原则，而 Spring 的依赖注入是一种特定的实现方式，它提供了更多功能和便利，帮助开发者更方便地管理和组织代码的依赖关系。

任何一个有实际意义的应用都会由两个或者更多的类组成，这些类相互之间进行协作来完成特定的业务逻辑。

按照传统的做法，每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将导致高度耦合和难以测试的代码，

耦合具有两面性(two-headed beast)。一方面，紧密耦合的代码难以测试、难以复用、难以理解，并且典型地表现出“打地鼠”式的 Bug 特性。另一方面，一定程度的耦合又是必须的——完全没有耦合的代码什么也做不了。总而言之，耦合是必须的，但应当被小心谨慎地管理。

```java
//DamselRescuingKnight只能执行RescueDamselQuest探险任务

package com.springinaction.knights;

public class DameselRecuingKnight implements Knight {
  
  private RescueDamselQuest quest;

  public DamselRescuingKnight() {
    // 与RescueDamselQuest 紧耦合    
    this.request = new RescueDamselQuest();
  }      
  public void embarkOnQuest() {
    quest.embark();  
  }   
}
```

DamselRescuingKnight 与 RescueDamselQuest 耦合到了一起，极大的限制了这个骑士执行探险的能力。

如果一个少女需要救援，这个骑士能够招之即来。但如果一条恶龙需要杀掉，或者一个圆桌需要被滚起来，那么这个骑士就爱莫能助了

更糟糕的是，为 DamselRescuingKnight 编写单元测试将出奇的困难。

在这样的一个测试中，你必须保证当骑士的 embarkOnQuest()方法被调用的时候，探险的 embark()方法也要被调用。但是没有一个简明的方式能够实现这一点。DamselRescuingKnight 将无法测试。

**通过 DI，对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或者管理他们的依赖关系**
![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720144744148-394392221.png)

**依赖注入会将所依赖的关系自动交给目标对象，而不是让对象自己去获取依赖**

```java
//BraveKnight足够灵活可以接受任何赋予它的探险任务
public class BraveKnight implements Knight {

  private Quest quest;
  
  public BraveKnight(Quest quest) { //Quest被注入进来
    this.quest = quest;  
  }       
  
  public void embarkOnQuest() {
    quest.embark();
  } 
}
```

BraveKnight 没有自行创建探险任务，而是在构造的时候把探险任务作为构造参数传入。

这是依赖注入的方式之一，即构造器注入（constructor injection）

**更主要的是，传入的是探险类型 Quest，**也就是所有探险任务都必须实现的一个接口，这里的要点是 BraveKnight 没有与任何特定的 Quest 实现发生耦合。

**DI 所带来的最大收益——松耦合。如果一个对象只通过接口（而不是具体实现或初始化过程）来表面依赖关系，那么这种依赖就能在对象本身毫不知情的情况下，用不同的具体实现进行替换**

对依赖进行替换的一个最常用方法就是在测试的时候使用 mock。我们无法充分测试 DamselRescuingKniht，因为它是紧耦合的；
但是可以轻松地测试 BraveKnight，只需要给它一个 Quest 的 mock 实现即可

```java
//为了测试BraveKnight，需要注入一个mock Quest
pakeage com.springinaction.knights;
import static org.mockito.Mockito.*;
import org.junit.Test;

public class BraveKnightTest {
  @Test
  public void knightShouldEmbarkOnQuest() {
    Quest mockQuest = mock(Quest.class); //创建mock Quest
    BraveKnight knight = new BraveKnight(mockQuest); //注入mock Quest
    knight.embarkOnQuest();
    verify(mockQuest, times(1)).embark();
  }
}
```

现在 BraveKnight 类可以接受你传递给它的任意一种 Quest 的实现，但是该如何把特定的 Quest 实现传递给它呢？

```java
//SlayDragonQueset是要注入到BraveKnight中的Quest实现
import java.io.PrintStream;

public class SlayDragonQuest implements Quest {
  private PrintStream stream;
  public SlayDragonQuest(PrintStream stream) {
    this.stream = stream;
  }

  public void embark() {
    stream.println("Embarking on quest to slay the dragon!");
  }
}
```

上面的代码中，我们该如何将 SlayDragonQuest 交给 BraveKnight 呢？又如何将 PrintStream 交给 SlayDragonQuest 呢？

创建应用组件之间协作的行为通常称为装配（wiring）。下面是 XML 装配 bean 的方式：

```java
//使用Spring将SlayDragonQuest注入到BraveKnight中

<bean id="knight" class="com.springinaction.knights.BraveKnight">
  <constructor-arg ref="quest" /> /*注入Quest bean*/
<bean>

/*创建SlayDragonQuest*/
<bean id="quest" class="com.springinaction.knights.SlayDragonQuest">
  <constructor-arg value="#{T(System).out}" /> 
</bean>
```

在这里，BraveKnight 和 SlayDragonQuest 被声明为 Spring 中的 bean。就 BraveKnight 来讲，它在构造时传入了对 SlayDragonQuest bean 的引用，将其作为构造器参数,同时 SlayDragonQuest bean 的声明使用了 Spring 表达式语言(Spring Expression Language),将 System.out(这是一个 PrintStream)传入到了 SlayDragonQuest 的构造器中

Spring 提供了基于 Java 的配置，可作为 XML 的替代方案

```java
@Configuration
public class KnightConfig {
  @Bean
  public Knight knight() {
    return new BraveKnight(quest());
  }

  @Bean
  public Quest quest() {
    return new SlayDragonQuest(System.out);
  }
}
```

关于@Configuration，可参考这篇文章：https://www.cnblogs.com/duanxz/p/7493276.html

现在已经声明了 BraveKnigt 和 Quest 的关系，接下来只需要装载 XML 配置文件，并把应用启动起来。

**Spring 通过应用上下文（Application Context）装载 bean 的定义并把它们组装起来。**

**Spring 应用上下文全权负责对象的创建和组装。Spring 自带了多种应用上下文的实现，它们之间主要的区别仅仅在于如何加载配置。**

因为 knights.xml 中的 bean 是使用 XML 文件进行配置的，所以选择 ClassPathXmlApplicationContext 作为应用上下文。

```java
//KnightMain.java加载包含Knight的Spring上下文
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class KnightMain {
  public static void main(String[] args) throws Exception {
    //加载Spring上下文
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("META-INF/spring/knight.xml");
    //获取knight bean
    Knight knight = context.getBean(Knight.class);  
    knight.embarkOnQuest();
    context.close();
  }
}
```

main()方法基于 knight.xml 文件创建了 Spring 应用上下文。随后它调用该应用上下文获取一个 ID 为 knight 的 bean
得到 Knight 对象的引用后，只需要简单调用 embarkOnQuest()方法即可。

这个类完全不知道我们的英雄骑士接受哪种探险任务，而且完全没有意识到这是由 BraveKnight 来执行的。只有 knights.xml 文件知道哪个骑士执行哪种任务。

### （三）应用切面

**DI 能够让相互协作的软件组件保持松散耦合，而面向切面编程(aspect-oriented programming,AOP)允许你把遍布应用各处的功能分离出来形成可重用的组件。**

**AOP 实现关注点分离。如日志、事务管理和安全这样的系统服务，通常被称为横切关注点，因为它们会跨越系统的多个组件**
![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720175133582-1299182761.png)

**AOP 能够使这些服务模块化，并以声明的方式将它们应用到它们需要影响的组件中去**。这样，这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务带来的复杂性。总之，AOP 能够确保 POJO 的简单性。

如下图所示，我们可以把切面想象为覆盖在很多组件上的一个外壳。使用 AOP 可以使用各种功能层去包裹核心业务层。这些层以声明的方式灵活地应用到系统中，你的核心应用甚至根本不知道它们的存在。

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720185348783-1095501477.png)

```java
//使用吟游诗人这个服务类来记载骑士的所有事迹
import java.io.PrintStream;

public class Minstrel {
  private PrintStream stream;
  public Minstrel(PrintStream stream) {
    this.stream = stream;
  }
  public void singBeforeQuest() {
    stream.println("Fa la la,the knight is so brave!");    
  }  
  public void singAfterQuest()  {
    stream.println("Tee hee hee, the brave knight " + 
        "did embark on a quest!");
  }
}
//BraveKnight必须要调用Minstrel的方法
public class BraveKnight implements Knight {
  private Quest quest;
  private Minstrel minstrel;
  public BraveKnight(Quest quest, Minstrel minstrel) {
    this.quest = quest;
    this.minstrel = minstrel;
  }
  public void embarkOnQuest() throws QuestException {
    //Knight应该管理它的Minstrel吗？
    minstrel.singBeforeQuest();
    quest.embark();
    minstrel.singAfterQuest(); 
  }  
}    
```

将 Minstrel 抽象为一个切面，所需要做的事情就是在一个 Spring 配置文件中声明它

```java
/*声明Minstrel bean*/
<bean id="minstrel" class="com.springinaction.knights.Minstrel">
    <constructor-arg value="#{T(System).out}" />
</bean>

<aop:config>
    <aop:aspect ref="minstrel">
        <aop:pointcut id="embark"
                expression="execution(* *.embarkOnQuest(...))" /> //定义切点
        <aop:before pointcut-ref="embark" method="singBeforeQuest" /> //声明前置通知
        <aop:after pointcut-ref="embark" method="singAfterQuest" /> //声明后置通知

    </aop:aspect>
</aop:config>
```

### （四）使用模板消除样板式代码

许多 JavaAPI，如 JDBC，会涉及编写大量的样板式代码：如首先需要创建数据连接，然后再创建一个语句对象，然后才能进行查询
JMS、JNDI 和使用 REST 服务通常也涉及大量的重复代码
**模板能够让你的代码关注于自身的职责**

Spring的JdbcTemplate，参考：https://www.cnblogs.com/caoyc/p/5630622.html

Spring 对数据库的操作在 jdbc 上面做了深层次的封装，使用 spring 的注入功能，可以把 DataSource 注册到 JdbcTemplate 之中。
主要提供以下五类方法：

- execute 方法：可以用于执行任何 SQL 语句，一般用于执行 DDL 语句；
- update 方法及 batchUpdate 方法：update 方法用于执行新增、修改、删除等语句；batchUpdate 方法用于执行批处理相关语句；
- query 方法及 queryForXXX 方法：用于执行查询相关语句；
- call 方法：用于执行存储过程、函数相关语句。

## 二、容纳你的 Bean

在基于 Spring 的应用中，你的应用对象生存于 Spring 容器（container）中。

**Spring 容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡（在这里可能就是 new 到 finalize()）**

容器是 Spring 框架的核心。容器并不是只有一个。Spring 自带来多个容器实现，**可以归为两种不同类型**：

- **bean工厂**(由 org.springframework.beans.factory.eanFactory 接口定义)是最简单的容器，提供基本的 DI 支持。
- **应用上下文**(由 org.springframework.context.ApplicationContext 接口定义)**基于 BeanFactory 构建**，并提供应用框架级别的服务，例如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者。

### （一）使用应用上下文

- **AnnotationConfigApplicationContext**：从一个或多个基于 Java 的配置类中加载 Spring 应用上下文
- **AnnotationConfigWebApplicationContext**：从一个或多个基于 Java 的配置类中加载 Spring Web 应用上下文
- **ClassPathXmlApplicationContext**：从类路径下的一个或多个 XML 配置文件中加载上下文定义，把应用上下文的定义文件作为类资源（在所有的类路径——包含 JAR 文件下查找 xml）
- **FileSystemXmlApplicationConext**：从文件系统下的一个或多个 XML 配置文件中加载上下文定义
- **XMlWebApplicationContext**：从 web 应用下的一个或多个 XML 配置文件中加载上下文定义

无论从文件系统还是从类路径下装载应用上下文，将 bean 加载到 bean 工厂的过程都是类似的

```java
ApplicationContext context = new FileSystemXmlApplicationContext("c:/knight.xml");
ApplicationContext context = new AnnotationConfigApplicationContext(com.springinaction.knights.config.KnightConfig.class);
```

在应用上下文准备就绪之后，我们就可以调用上下文的 getBean()方法从 Spring 容器中获取 bean

### （二）bean 的生命周期

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720204426088-810251844.png)

Bean的生命周期，参考：https://www.cnblogs.com/xujian2014/p/5049483.html

## 三、Spring 模块

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720205822427-567801302.png)

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720205924661-1265651631.png)

Spring Portfolio 几乎为每一个领域的 Java 开发都提供了 Spring 编程模型，如：
Spring Web Flow：为基于流程的会话式 Web 应用（可以想一下购物车或者向导功能）提供了支持
Spring Web Service
Spring Security：利用 AOP，Spring Security 为 Spring 应用提供了声明式安全机制
Spring Integration：提供了多种通用应用集成模式的 Spring 声明式风格实现
Spring Batch：需要对数据进行大量操作时，如需要开发一个批处理应用。
Spring Data：JPA

Spring Social:
Spring Boot