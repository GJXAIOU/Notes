# 第二章：装配 Bean

分为四个部分：

- 声明bean
- 构造器注入和Setter方法注入
- 装配bean
- 控制bean的创建和销毁



在 Spring中，对象无需自己**查找或创建**与其相关联的其他对象。容器负责把需要相互协作的对象引用赋予给各个对象。

创建应用对象之间协作关系的行为称为装配（wiring），这也是依赖注入（DI）的本质

*什么是bean：https://blog.csdn.net/weixin_42594736/article/details/81149379 
         https://www.cnblogs.com/cainiaotuzi/p/7994650.html*

配置Spring容器最常见的三种方法：

## 一、Spring配置的可选方案

Spring 容器负责创建应用程序中的 Bean 并通过 DI 来协调这些对象之间的关系。

开发人员工作：告诉 Spring 要创建哪些 bean 并且如何将其装配在一起。在描述 bean 如何装配时，spring 提供以下三种主要的装配机制：

- 在XML中进行显式配置
- 在Java中进行显式配置
- 隐式的bean发现机制和自动装配

原则：**尽可能的使用自动装配机制，显式配置越少越好，必须使用时有限使用 javaConfig，最后使用 xml**。同时三种方式可以并行存在并组合使用。

## 二、自动化装配 Bean

Spring 从两个角度来实现自动化装配bean：

- 组件扫描（component scanning）：Spring 会自动发现应用上下文中所创建的 bean
- 自动装配（autowiring）：Spring 自动满足 bean 之间的依赖

创建 Bean  ==》启动组件扫描

CD为我们阐述DI如何运行提供了一个很好的样例——你不将CD插入（注入）到CD播放器中，那么播放器其实是没有太大用处的。CD播放器依赖于CD才能完成它的使命

```java
//CompactDisc接口在Java中定义了CD的概念
package org.gjxaiou.beans;

public interface CompactDisc {
    void play();
}

```

CompactDisc作为接口，它定义了CD播放器对一盘CD所能进行的操作，**它将CD播放器的任意实现与CD本身的耦合降低到了最小的程度**
我们还需要一个CompactDisc的实现

```java
//带有@Component注解的CompactDisc实现类SgtPeppers
package org.gjxaiou.beans;

import org.springframework.stereotype.Component;

@Component
public class CompactDiscImpl implements CompactDisc {
    private String hello = "hello";
    public void play() {
        System.out.println(hello);
    }
}

```

SgtPeppers 类上使用了 **@Component注解，这个注解表明该类会作为组件类，并告知Spring要为这个类创建bean。**此时不需要再显式的配置该类的 bean。

**启动组件扫描方式一：**

**组件扫描默认是不启用的**。还需要显式配置一下 Spring，从而命令它去寻找带有 @Component 注解的类，并为其创建 bean

```Java
//@Component注解启用了组件扫描
package org.gjxaiou.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan("org.gjxaiou.beans.*")
public class CDPlayerConfig {
}

```

该类通过 Java 代码定义了 Spring 的装配规则。但是 CDPlayerConfig 类并没有显式声明任何 bean，只不过它使用了 @ComponentScan 注解，这个注解能够在Spring中启用组件扫描。

如果没有其他配置的话，@ComponentScan 默认会扫描与配置类相同的包。

因为CDPlayerConfig类位于soundsystem包中，因此**Spring将会扫描这个包以及这个包下的所有子包**，查找带有@Component注解的类。

这样，就能发现CompactDisc，并且会在Spring中自动为其创建一个bean。

**启动组件扫描方式二**：

也可以用XML来启用组件扫描，可以使用Spring context命名空间的<context:component-scan>元素（推荐上面的 Java 配置）

```xml
<beans xxxxxx>
  <context:component-scan base-package="soundsystem" />
</beans>
```

下面，我们创建一个简单的JUnit测试，它会创建Spring上下文，并判断CompactDisc是不是真的创建出来了

```java
package org.gjxaiou;

import org.gjxaiou.config.CDPlayerConfig;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(CDPlayerConfig.class);
        System.out.println(applicationContext.getBeanDefinitionNames());
        System.out.println("Hello World!");
    }
}
// 输出如下
[Ljava.lang.String;@7403c468
Hello World!
```

只添加一行@ComponentScan注解就能自动创建无数个bean

2.2.2、为组件扫描的bean命名

Spring应用上下文中所有的bean都会给定一个ID。默认会根据类名为其指定一个ID，如上面例子中SgtPeppersbean的ID为sgtPeppers，**也就是将类名的第一个字母变小写
**

```
//设定bean的ID
@Component("lonelyHeartsClub")
//也可以用@Named注解来为bean设置ID
@Named("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc{...}
```

2.2.3、设置组件扫描的基础包

**默认会以配置类所在的包作为基础包(basepackage)来扫描组件**。如果想扫描不同的包，或者扫描多个包，又该如何呢？

```
//指明包的名称
@Configuration
@ComponentScan("soundsystem")
public class CDPlayerConfig() {}
```

如果想更清晰地表明你所设置的是基础包，可以通过basePackages属性进行配置：

```
@Configuration
@ComponentScan(basePackages="soundsystem")
public class CDPlayerConfig() {}
```

多个基础包：basePackages={"soundsystem", "video"}
上面所设置的基础包是以String类型表示的，这种方法是类型不安全（not type-safe）的

```
//另外一种方法，将其指定为包种所包含的类或接口：
@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class })
public class CDPlayerConfig {}
```

如果所有的对象都是独立的，彼此之间没有任何依赖，就像SgtPeppersbean这样，那你所需要的可能就是组件扫描而已
但是很多对象会依赖其他的对象才能完成任务，这就需要一种方法能够将组件扫描得到的bean和它们的依赖装配在一起——Spring的自动装配

2.2.4、通过为bean添加注解实现自动装配

**自动装配就是让Spring自动满足bean依赖的一种方法**，在满足依赖的过程种，会在Spring应用上下文种寻找匹配某个bean需求的其他bean
为了声明要进行自动装配，借助Spring的@Autowired注解

@Autowired注解不仅能用在构造器上，还能用在属性的Setter方法上：

```
@Autowired
public void setCompactDisc(CompactDisc cd) {
  this.cd = cd;  
}
```

在Spring初始化bean之后，它会尽可能得去满足bean的依赖，在上面的例子中，依赖是通过带有@Autowired注解的方法进行声明
假如有且只有一个bean匹配依赖需求的话，那这个bean将会被装配进来
如果没有匹配的bean，那么在应用上下文创建的时候，Spring会抛出一个异常，为避免这个异常，可以：@Autowired(required=false)
当required设置为false时，Spring会尝试执行自动装配，但如果没有匹配的bean的话，Spring将会让这个bean处于未装配状态
如果你的代码没有进行null检查的话，这个处于未装配状态的属性有可能会出现NullPointerException

如果有多个bean满足依赖关系的话，Spring会抛出一个异常，表明没有明确指定要选择哪个bean进行自动装配

@Autowired还可以替换为@Inject——不推荐

2.3、通过Java代码装配bean

比如你想要将第三方库中的组件装配到你的应用中，这种情况下，是没有办法在它的类上添加@Component和@Autowired注解的，因此就不能使用自动化装配的方案了
要采用显式装配的方式。两种方案：Java和XML。
进行显示配置时，JavaConfig是更好的方案。因为它更强大、类型安全且对重构友好
JavaConfig是配置代码，意味着它不应该包含任何业务逻辑，JavaConfig也不应该侵入到业务逻辑代码中，**通常会将JavaConfig放到单独的包中**

2.3.1、创建配置类

**创建JavaConfig类的关键在于为其添加@Configuration注解，表明这个类是一个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节**
到此为止，我们都是依赖组件扫描来发现Spring应该创建的bean，下面将关注于显示配置

 2.3.2、声明简单的bean

要在JavaConfig中声明bean，需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加@Bean注解，如下面的代码声明了CompactDisc bean：

```
@Bean
public CompactDisc sgtPepper() {
  return new SgtPeppers();  
}
```

 @Bean注解会告诉Spring这个方法将会返回一个对象，该对象要注册为Spring应用上下文中的bean

2.3.3、借助JavaConfig实现注入

在JavaConfig中装配bean的最简单的方式就是引用创建bean的方法，如下就是一种声明CDPlayer的可行方案：

```
@Bean
public CDPlayer cdPlayer() {
  return new CDPlayer(sgtPeppers());  
}
```

在这里并没使用默认的构造器构造实例，而是调用了需要传入CompactDisc对象的构造器来创建CDPlayer实例

看起来，CompactDisc是通过调用sgtPeppers()得到的，但情况并非完全如此。
因为sgtPeppers()方法添加了@Bean注解，Spring将会拦截所有对它的调用，并确保直接返回该方法所创建的bean，而不是每次都对其进行实际的调用

通过调用方法来引用bean的方式有点令人困惑。还有一种理解起来更简单的方式：

```
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
  return new CDPlayer(compactDisc)  
}
```

2.4、通过XML装配bean——Spring现在有了强大的自动化配置和基于Java的配置，XML不应该在是你的第一选择了

2.4.1、创建XML配置规范

在使用XML为Spring装配bean之前，你需要创建一个新的配置规范。
在使用JavaConfig的时候，这意味着要创建一个带有@Configuration注解的类，而在XML配置中，意味着要创建一个以<beans>元素为根的XML文件

2.4.2、声明一个简单的<bean>：<bean>元素类似于JavaConfig中的@Bean注解

```
//创建这个bean的类通过class属性类指定的，并且要使用全限定的类名
//建议设置id，不要自动生成
<bean id="compactDisc" class="soundsystem.SgtPeppers" />
```

2.4.3、借助构造器注入初始化bean

```
//通过ID引用SgtPeppers
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <constructor-arg ref="compactDisc" />
</bean>
```

当Spring遇到这个<bean>元素时，会创建一个CDPlayer实例。<constructor-arg>元素会告诉Spring要将一个ID为compactDisc的bean引用传递到CDPlayer的构造器中

当构造器参数的类型是java.util.List时，使用<list>元素。set也一样

```
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="The Beatles" />
    <constructor-arg>
        <list>
          <ref bean="sgtPeppers />
          <ref bean="whiteAlbum />      
        </list>
    </constructor-arg>
</bean>
```

2.4.4、使用XML实现属性注入

**通用规则：对强依赖使用构造器注入，对可选性依赖使用属性注入**

**<property>元素**为属性的Setter方法所提供的功能与<constructor-arg>元素为构造器所提供的功能是一样的

```
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <property name="compactDisc" ref="compactDisc" />
</bean>
```

借助<property>元素的value属性注入属性

```
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <property name="title" value="Sgt. Pepper's Lonely Hearts Club Band" />
    <property name="tracks">
        <list>
            <value>Getting Better</value>
            <value>Fixing a Hole</value>
        </list>
    </property>
</bean>    
```

2.5、导入和混合配置

2.5.1、在JavaConfig中引用XML配置

我们假设CDPlayerConfig已经很笨重，要将其拆分，方案之一就是将BlankDisc从CDPlayerConfig拆分出来，定义到它自己的CDConfig类中：

```
package soundsystem;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class CDConfig {
  @Bean 
  public CompactDisc compactDisc() {
    return new SgtPeppers();    
  }    
}
```

然后将两个类在CDPlayerConfig中使用@Import注解导入CDConfig：

```
package soundsystem;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;

@Configuration
@Import(CDConfig.class)
public class CDPlayerConfig {
  @Bean
  public CDPlayer cdPlayer(CompactDisc compactDisc) {
    return new CDPlayer(compactDisc);    
  }
}
```

**更好的办法是：创建一个更高级别的SoundSystemConfig，在这个类中使用@Import 将两个配置类组合在一起：**

```
package soundsystem;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import({CDPlayerConfig.class, CDConfig.class})
public class SoundSystemConfig { 
}
```

如果有一个配置类配置在了XML中，则使用@ImportResource注解

```
@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```

2.5.2、在XML配置中引用JavaConfig

**在JavaConfig配置中，使用@Import和@ImportResource来拆分JavaConfig类**
**在XML中，可以使用import元素来拆分XML配置**

```
<beans  ....>
  <bean class="soundsystem.CDConfig" />
  <import resource="cdplayer-config.xml" />
</beans>
```

**不管使用JavaConfig还是使用XML进行装配，通常都会创建一个根配置，这个配置会将两个或更多的装配类/或XML文件组合起来**
**也会在根配置中启用组件扫描(通过<context:component-scan>或@ComponentScan)**

**小结：**

Spring框架的核心是Spring容器。容器负责管理应用中组件的生命周期，它会创建这些组件并保证它们的依赖能够得到满足，这样的话，组件才能完成预定的任务
Spring中装配bean的三种主要方式：自动化配置、基于java的显式配置以及基于XML的显示配置。这些技术描述了Spring应用中的组件以及这些组件之间的关系
尽可能使用自动化装配，避免显式装配所带来的维护成本。基于Java的配置，它比基于XML的配置更加强大、类型安全并且易于重构