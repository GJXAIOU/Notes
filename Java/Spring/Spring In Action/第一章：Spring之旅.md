1.1、简化Java开发

创建Spring的主要目的是用来替代更加重量级的企业级Java技术，尤其是EJB。相对EJB来说，Spring提供了更加轻量级和简单的编程模型。

它增强了简单老式Java对象（Plain Old Java Object，POJO），使其具备了之前只有EJB和其他企业级Java规范才具有的功能

为了降低Java开发的复杂性，Spring采取了以下4种关键策略：

- 基于POJO的轻量级和最小侵入性编程

- 通过依赖注入和面向接口实现松耦合

- 基于切面和惯例进行声明式编程

- 通过切面和模板减少样式代码

  

1.1.1、Spring赋予POJO魔力的方式之一就是通过DI来装配它们。帮助应用对象彼此之间保持松散耦合

1.1.2、依赖注入
　　  任何一个有实际意义的应用都会由两个或者更多的类组成，这些类相互之间进行协作来完成特定的业务逻辑。
　　  按照传统的做饭，每个对象负责管理与自己相互协作的对象（即它所依赖的对象）的引用，这将导致高度耦合和难以测试的代码

 　　 耦合具有两面性(two-headed beast)。一方面，紧密耦合的代码难以测试、难以复用、难以理解，并且典型地表现出“打地鼠”式的Bug特性
 　 另一方面，一定程度的耦合又是必须的——完全没有耦合的代码什么也做不了。
　　　总而言之，耦合是必须的，但应当被小心谨慎地管理

```
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

DamselRescuingKnight与RescueDamselQuest耦合到了一起，极大的限制了这个骑士执行探险的能力。
如果一个少女需要救援，这个骑士能够招之即来。但如果一条恶龙需要杀掉，或者一个圆桌需要被滚起来，那么这个骑士就爱莫能助了

更糟糕的是，为DamselRescuingKnight编写单元测试将出奇的困难。
在这样的一个测试中，你必须保证当骑士的embarkOnQuest()方法被调用的时候，探险的embark()方法也要被调用。但是没有一个简明的方式能够实现这一点。
DamselRescuingKnight将无法测试

通过DI，对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。
对象无需自行创建或者管理他们的依赖关系
![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720144744148-394392221.png)

依赖注入会将所依赖的关系自动交给目标对象，而不是让对象自己去获取依赖

```
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

BraveKnight没有自行创建探险任务，而是在构造的时候把探险任务作为构造参数传入。
这是依赖注入的方式之一，即**构造器注入（constructor injection）

**更主要的是，传入的是探险类型Quest，**也就是所有探险任务都必须实现的一个接口
这里的要点是BraveKnight没有与任何特定的Quest实现发生耦合。
DI所带来的最大收益——松耦合。如果一个对象只通过接口（而不是具体实现或初始化过程）来表面依赖关系，那么这种依赖就能在对象本身毫不知情的情况下，用不同的具体实现进行替换**

对依赖进行替换的一个最常用方法就是在测试的时候使用mock。我们无法充分测试DamselRescuingKniht，因为它是紧耦合的；
但是可以轻松地测试BraveKnight，只需要给它一个Quest的mock实现即可

```
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

现在BraveKnight类可以接受你传递给它的任意一种Quest的实现，但是该如何把特定的Quest实现传递给它呢？

```
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

上面的代码中，我们该如何将SlayDragonQuest交给BraveKnight呢？又如何将PrintStream交给SlayDragonQuest呢？

创建应用组件之间协作的行为通常称为装配（wiring）。下面是XML装配bean的方式：

```
//使用Spring将SlayDragonQuest注入到BraveKnight中

<bean id="knight" class="com.springinaction.knights.BraveKnight">
  <constructor-arg ref="quest" /> /*注入Quest bean*/
<bean>

/*创建SlayDragonQuest*/
<bean id="quest" class="com.springinaction.knights.SlayDragonQuest">
  <constructor-arg value="#{T(System).out}" /> 
</bean>
```

在这里，BraveKnight和SlayDragonQuest被声明为Spring中的bean。就BraveKnight来讲，它在构造时传入了对SlayDragonQuest bean的引用，将其作为构造器参数
同时SlayDragonQuest bean的声明使用了Spring表达式语言(Spring Expression Language),将System.out(这是一个PrintStream)传入到了SlayDragonQuest的构造器中

Spring提供了基于Java的配置，可作为XML的替代方案

```
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

*关于@Configuration，可参考这篇文章：https://www.cnblogs.com/duanxz/p/7493276.html*

*
*现在已经声明了BraveKnigt和Quest的关系，接下来只需要装载XML配置文件，并把应用启动起来
Spring通过应用上下文（Application Context）装载bean的定义并把它们组装起来。
Spring应用上下文全权负责对象的创建和组装。Spring自带了多种应用上下文的实现，它们之间主要的区别仅仅在于如何加载配置

因为knights.xml中的bean是使用XML文件进行配置的，所以选择ClassPathXmlApplicationContext作为应用上下文。

```
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

main()方法基于knight.xml文件创建了Spring应用上下文。随后它调用该应用上下文获取一个ID为knight的bean
得到Knight对象的引用后，只需要简单调用embarkOnQuest()方法即可
这个类完全不知道我们的英雄骑士接受哪种探险任务，而且完全没有意识到这是由BraveKnight来执行的。只有knights.xml文件知道哪个骑士执行哪种任务

 

1.1.3、应用切面

DI能够让相互协作的软件组件保持松散耦合，而面向切面编程(aspect-oriented programming,AOP)允许你把遍布应用各处的功能分离出来形成可重用的组件
AOP实现关注点分离。如日志、事务管理和安全这样的系统服务，通常被称为横切关注点，因为它们会跨越系统的多个组件
![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720175133582-1299182761.png)

AOP能够使这些服务模块化，并以声明的方式将它们应用到它们需要影响的组件中去。这样，这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务带来的复杂性。总之，AOP能够确保POJO的简单性

如下图所示，我们可以把切面想象为覆盖在很多组件上的一个外壳。使用AOP可以使用各种功能层去包裹核心业务层。这些层以声明的方式灵活地应用到系统中，你的核心应用甚至根本不知道它们的存在

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720185348783-1095501477.png)

 

```
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

将Minstrel抽象为一个切面，所需要做的事情就是在一个Spring配置文件中声明它

```
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

1.1.4、使用模板消除样板式代码

许多JavaAPI，如JDBC，会涉及编写大量的样板式代码：如首先需要创建数据连接，然后再创建一个语句对象，然后才能进行查询
JMS、JNDI和使用REST服务通常也涉及大量的重复代码
**模板能够让你的代码关注于自身的职责**

*Spring的JdbcTemplate，参考：https://www.cnblogs.com/caoyc/p/5630622.html*
Spring对数据库的操作在jdbc上面做了深层次的封装，使用spring的注入功能，可以把DataSource注册到JdbcTemplate之中。
主要提供以下五类方法：

- execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句；
- update方法及batchUpdate方法：update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句；
- query方法及queryForXXX方法：用于执行查询相关语句；
- call方法：用于执行存储过程、函数相关语句。

 

1.2、容纳你的Bean

在基于Spring的应用中，你的应用对象生存于Spring容器（container）中。
Spring容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡（在这里可能就是new到finalize()）

容器是Spring框架的核心。容器并不是只有一个。Spring自带来多个容器实现，**可以归为两种不同类型**：
**bean工厂**(由org.springframework.beans.factory.eanFactory接口定义)是最简单的容器，提供基本的DI支持
**应用上下文**(由org.springframework.context.ApplicationContext接口定义)**基于BeanFactory构建**，并提供应用框架级别的服务，例如从属性文件解析文本信息以及发布应用事件给感兴趣的事件监听者

1.2.1、使用应用上下文

- **AnnotationConfigApplicationContext**：从一个或多个基于Java的配置类中加载Spring应用上下文
- **AnnotationConfigWebApplicationContext**：从一个或多个基于Java的配置类中加载Spring Web应用上下文
- **ClassPathXmlApplicationContext**：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源（在所有的类路径——包含JAR文件下查找xml）
- **FileSystemXmlApplicationConext**：从文件系统下的一个或多个XML配置文件中加载上下文定义
- **XMLWebApplicationContext**：从web应用下的一个或多个XML配置文件中加载上下文定义

无论从文件系统还是从类路径下装载应用上下文，将bean加载到bean工厂的过程都是类似的

```
ApplicationContext context = new FileSystemXmlApplicationContext("c:/knight.xml");
ApplicationContext context = new AnnotationConfigApplicationContext(com.springinaction.knights.config.KnightConfig.class);
```

在应用上下文准备就绪之后，我们就可以调用上下文的getBean()方法从Spring容器中获取bean

1.2.2 bean的生命周期

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720204426088-810251844.png)

*Bean的生命周期，参考：https://www.cnblogs.com/xujian2014/p/5049483.html*

1.3.1、Spring模块
![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720205822427-567801302.png)

![img](%E7%AC%AC%E4%B8%80%E7%AB%A0%EF%BC%9ASpring%E4%B9%8B%E6%97%85.resource/286343-20190720205924661-1265651631.png)

Spring Portfolio几乎为每一个领域的Java开发都提供了Spring编程模型，如：
Spring Web Flow：为基于流程的会话式Web应用（可以想一下购物车或者向导功能）提供了支持
Spring Web Service
Spring Security：利用AOP，Spring Security为Spring应用提供了声明式安全机制
Spring Integration：提供了多种通用应用集成模式的Spring声明式风格实现
Spring Batch：需要对数据进行大量操作时，如需要开发一个批处理应用。
**Spring Data：JPA
**Spring Social:
Spring Boot