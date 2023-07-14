# 第二章：装配 Bean

分为四个部分：

- 声明 bean
- 构造器注入和 Setter 方法注入
- 装配 bean
- 控制 bean 的创建和销毁



在 Spring 中，对象无需自己**查找或创建**与其相关联的其他对象。容器负责把需要相互协作的对象引用赋予给各个对象。 ==》我们需要标明对象之间的依赖关系。

**创建应用对象之间协作关系的行为称为装配（wiring），这也是依赖注入（DI）的本质**

什么是bean：https://blog.csdn.net/weixin_42594736/article/details/81149379 
         https://www.cnblogs.com/cainiaotuzi/p/7994650.html

## 一、Spring配置的可选方案

Spring 容器负责创建应用程序中的 Bean 并通过 DI 来协调这些对象之间的关系。

开发人员工作：告诉 Spring 要创建哪些 bean 并且如何将其装配在一起。在描述 bean 如何装配时，spring 提供以下三种主要的装配机制：

- 在 XML 中进行显式配置
- 在 Java 中进行显式配置
- 隐式的 bean 发现机制和自动装配

原则：**尽可能的使用自动装配机制，显式配置越少越好，必须使用时优先使用 JavaConfig，最后使用 xml**。同时三种方式可以并行存在并组合使用。

## 二、自动化装配 Bean

Spring 从两个角度来实现自动化装配 bean：

- 组件扫描（component scanning）：Spring 会自动发现应用上下文中所创建的 bean
- 自动装配（autowiring）：Spring 自动满足 bean 之间的依赖

创建 Bean  ==》启动组件扫描

CD 为我们阐述 DI 如何运行提供了一个很好的样例——你不将 CD 插入（注入）到 CD 播放器中，那么播放器其实是没有太大用处的。CD 播放器依赖于 CD 才能完成它的使命

```java
//CompactDisc接口在Java中定义了CD的概念
package org.gjxaiou.beans;

public interface CompactDisc {
    void play();
}

```

CompactDisc 作为接口，它定义了 CD 播放器对一盘 CD 所能进行的操作，**它将 CD 播放器的任意实现与 CD 本身的耦合降低到了最小的程度**
我们还需要一个 CompactDisc 的实现

```java
//带有@Component注解的CompactDisc实现类CompactDiscImpl
package org.gjxaiou.beans;

import org.springframework.stereotype.Component;

@Component("compactDiscImpl")
public class CompactDiscImpl implements CompactDisc {
    private String hello = "hello";
    public void play() {
        System.out.println(hello);
    }
}

```

CompactDiscImpl 类上使用了 **@Component 注解，这个注解表明该类会作为组件类，并告知 Spring 要为这个类创建 bean。**此时不需要再显式的配置该类的 bean。

Spring 上下文中所有的 bean 都会给定一个 ID，默认就是类名第一个字母变为小写，也可以在 @Component 中显示指定。

**启动组件扫描方式一：**

**组件扫描默认是不启用的**。还需要显式配置一下 Spring，从而命令它去寻找带有 @Component 注解的类，并为其创建 bean

```Java
//@Component注解启用了组件扫描
package org.gjxaiou.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
// 本质是给 value 字段赋值
@ComponentScan("org.gjxaiou")
// 含义同上，但是可以指定多个包
@ComponentScan(basePackages = {"org.gjxaiou","org.hello"})
// 以上两种都是 string 形式，如果包名改了无效了
// 可以通过下面指定包中所包含类，即该类所在的包会做为组件扫描的基础包，配置的类都不需要是个 bean，只要是个类或者接口就行
@ComponentScan(basePackageClasses = {CDPlayerConfig.class, CompactDiscImpl.class} )
public class CDPlayerConfig {
}

```

该类通过 Java 代码定义了 Spring 的装配规则。但是 CDPlayerConfig 类并没有显式声明任何 bean，只不过它使用了 @ComponentScan 注解，这个注解能够在 Spring 中启用组件扫描。

如果没有其他配置的话，@ComponentScan 默认会扫描与配置类相同的包。

因为 CDPlayerConfig 类位于 soundsystem 包中，因此**Spring 将会扫描配置类所在包以及这个包下的所有子包**，查找带有 @Component 注解的类。

这样，就能发现 CompactDisc，并且会在 Spring 中自动为其创建一个 bean。

==》但是这里的 CDPlayerConfig 与 CompactDiscImpl 不完全在同一个包下，所以需要声明扫描的包路径。

==》方式二：使用 Java 依赖注入规范提供的 `@Named`，其和 @Component 使用一样；

**说明：推荐使用 basePackageClasses 的方式，在包中创建一个用于扫描的空标记接口，这样后续不耽误其他类和包名的重构。**

**启动组件扫描方式二**：

也可以用 XML 来启用组件扫描，可以使用 Spring context 命名空间的 `<context:component-scan>` 元素（推荐上面的 Java 配置）

```xml
<beans xxxxxx>
  <context:component-scan base-package="soundsystem" />
</beans>
```

创建 Spring 上下文，并判断 CompactDisc 是不是真的创建出来了

```java
package org.gjxaiou.config;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(CDPlayerConfig.class);
        System.out.println(applicationContext.getBean("compactDiscImpl"));
        System.out.println("Hello World!");
    }
}

// 输出如下
org.gjxaiou.beans.CompactDiscImpl@6fb0d3ed
Hello World!
```

只添加一行 @ComponentScan 注解就能自动创建无数个 bean

#### 2.2.2、为组件扫描的 bean 命名

Spring 应用上下文中所有的 bean 都会给定一个 ID。默认会根据类名为其指定一个 ID，如上面例子中 SgtPeppersbean 的 ID 为 sgtPeppers，**也就是将类名的第一个字母变小写**

```java
//设定bean的ID
@Component("lonelyHeartsClub")
//也可以用@Named注解来为bean设置ID
@Named("lonelyHeartsClub")
public class SgtPeppers implements CompactDisc{...}
```

#### 2.2.3、设置组件扫描的基础包

**默认会以配置类所在的包作为基础包(basepackage)来扫描组件**。如果想扫描不同的包，或者扫描多个包，又该如何呢？

```java
//指明包的名称
@Configuration
@ComponentScan("soundsystem")
public class CDPlayerConfig() {}
```

如果想更清晰地表明你所设置的是基础包，可以通过 basePackages 属性进行配置：

```java
@Configuration
@ComponentScan(basePackages="soundsystem")
public class CDPlayerConfig() {}
```

多个基础包：basePackages={"soundsystem", "video"}
上面所设置的基础包是以 String 类型表示的，这种方法是类型不安全（not type-safe）的

```java
//另外一种方法，将其指定为包种所包含的类或接口：
@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class })
public class CDPlayerConfig {}
```

如果所有的对象都是独立的，彼此之间没有任何依赖，就像 SgtPeppersbean 这样，那你所需要的可能就是组件扫描而已。

但是很多对象会依赖其他的对象才能完成任务，这就需要一种方法能够将组件扫描得到的 bean 和它们的依赖装配在一起——Spring 的自动装配

### 2.2.4、通过为 bean 添加注解实现自动装配

**自动装配就是让 Spring 自动满足 bean 依赖的一种方法**，在满足依赖的过程种，会在 Spring 应用上下文种寻找匹配某个 bean 需求的其他 bean。

为了声明要进行自动装配，借助 Spring 的 @Autowired 注解

@Autowired 注解可以放在属性、构造器、属性的 Setter 方法，甚至类中任何方法上；

```java
package org.gjxaiou.beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer {
    private CompactDisc compactDisc;

    // 构造器上加上 @Autowired,表示当 spring 创建 CDPlayer bean 的时候，会通过这个构造器来进行实例化，并且会传入一个可设置给 CompactDisc 类型的 bean
    @Autowired
    public CDPlayer(CompactDisc cd) {
        this.compactDisc = cd;
    }
    
    public void testMethod() {
        compactDisc.play();
    }
}
```

@Autowird 还可以放在属性的 setter 方法上来实现自动装配

```java
package org.gjxaiou.beans;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class CDPlayer {
    private CompactDisc compactDisc;

    // 也可以放在属性的 setter 方法上来实现自动装配
    @Autowired
    public void setCompactDisc(CompactDisc cd ){
        this.compactDisc = cd;
    }

    // 实际上可以放在任意方法上
    @Autowired
    public void insertCompact(CompactDisc cd){
        this.compactDisc = cd;
    }

    public void testMethod() {
        compactDisc.play();
    }
}
```

在 Spring 初始化 bean 之后，它会尽可能得去满足 bean 的依赖，在上面的例子中，依赖是通过带有 @Autowired 注解的方法进行声明的，即上面的两个方法都行。

不管是构造器、setter 方法还是其他方法，spring 都会尝试满足方法参数上所声明的依赖，假如有且只有一个 bean 匹配依赖需求的话，那这个 bean 将会被装配进来。

如果没有匹配的 bean，那么在应用上下文创建的时候，Spring 会抛出一个异常，为避免这个异常，可以：@Autowired(required=false)
当 required 设置为 false 时，Spring 会尝试执行自动装配，但如果没有匹配的 bean 的话，Spring 将会让这个 bean 处于未装配状态
如果你的代码没有进行 null 检查的话，这个处于未装配状态的属性有可能会出现 NullPointerException。

如果有多个 bean 满足依赖关系的话，Spring 会抛出一个异常，表明没有明确指定要选择哪个 bean 进行自动装配。

@Autowired 还可以替换为 @Inject ——不推荐

==>上面就是通过组件扫描和自动装配实现 spring 的自动化配置。

### 2.3、通过 Java 代码装配 bean ==》显式装配 bean

比如你想要将第三方库中的组件装配到你的应用中，这种情况下，是没有办法在它的类上添加 @Component 和 @Autowired 注解的，因此就不能使用自动化装配的方案了。

要采用显式装配的方式。两种方案：Java 和 XML。

进行显示配置时，JavaConfig 是更好的方案。因为它更强大、类型安全且对重构友好，因为其就是 Java 代码。

JavaConfig 与其他 Java 代码不同点：JavaConfig 是配置代码，意味着它不应该包含任何业务逻辑，JavaConfig 也不应该侵入到业务逻辑代码中，**通常会将JavaConfig放到单独的包中**

#### 2.3.1、创建配置类

**创建 JavaConfig 类的关键在于为其添加 @Configuration 注解，表明这个类是一个配置类，该类应该包含在 Spring 应用上下文中如何创建bean的细节**。

之前我们都是依赖组件扫描来发现 Spring 应该创建的 bean，尽管我们可以同时使用组件扫描和显式配置。因为这里主要关注显式配置，所以去除了 @ComponentScan。

```java
package org.gjxaiou.config;

import org.springframework.context.annotation.Configuration;

@Configuration
public class CDPlayerConfig {
}

```

此时没有组件扫描了，则上面的程序在执行就失败了，以为扫描不到上面的那些 bean 了。

####  2.3.2、声明简单的 bean

要在 JavaConfig 中声明 bean，需要编写一个方法，这个方法会创建所需类型的实例，然后给这个方法添加 @Bean 注解，如下面的代码声明了 CompactDisc bean：

```java
@Bean
public CompactDisc sgtPepper() {
  return new SgtPeppers();  
}
```

@Bean 注解会告诉 Spring 这个方法将会返回一个对象，该对象要注册为 Spring 应用上下文中的 bean。这个方法体中就包含了最终产生 bean 实例的逻辑。   ==》这个 bean 就不需要使用组件扫描也能加到 spring 中了。

默认生成的 bean 名称和该方法名一样，或者在 @Bean(name = “xxx”) 显式指定 bean 名称。

==》原则，只要这个方法最终生成一个类实例即可。

#### 2.3.3、借助 JavaConfig 实现注入

上面是声明一个 bean，现在要将两个 bean 进行装配。

在 JavaConfig 中装配 bean 的最简单的方式就是引用创建 bean 的方法，如下就是一种声明 CDPlayer 的可行方案：

```java
@Bean
public CDPlayer cdPlayer() {
  return new CDPlayer(sgtPeppers());  
}
```

cdPlayer() 方法也使用了 @Bean，表示这个方法会创建一个 bean 实例并将其注册到 Spring 的应用上下文中。

cdPlayer() 方法并没使用默认的构造器构造实例，而是调用了需要传入 CompactDisc 对象的构造器来创建 CDPlayer 实例

看起来，CompactDisc 是通过调用 sgtPeppers()得到的，但情况并非完全如此。

**因为 sgtPeppers()方法添加了@Bean 注解，Spring 将会拦截所有对它的调用，并确保直接返回该方法所创建的 bean，而不是每次都对其进行实际的调用。所以如果多个 bean 的创建中都调用 sgtPeppers() 方法，返回的其实都是同一个 bean**。

通过调用方法来引用 bean 的方式有点令人困惑。还有一种理解起来更简单的方式：

```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
  return new CDPlayer(compactDisc)  
}
```

在这里，cdPlayer()方法请求一个 CompactDisc 作为参数。当 Spring 调用 cdPlayer()创建 CDPlayer bean 的时候，它会自动装配一个 CompactDisc 到配置方法中。然后，方法体就可以按照合适的方式来使用它了。借助这种技术，不用明确引用 CompactDisc 的@Bean 方法。

通过这种方式引用其他的 bean 通常是最佳的选择，因为它不会要求将 CompactDisc 声明到同一个配置类中。在这里甚至没有要求 CompactDisc 必须要在 JavaConfig 中声明，实际上它可以通过组件扫描功能自动发现或者通过 xml 来进行配置。不管 CompactDisc 是采用什么方式创建出来的，Spring 都会将其传入到配置方法中。

上面都是使用构造器实现 DI，其实也可以使用 setter 方法实现依赖注入，本质上只要能返回一个实例，方式随便选；

```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
    CDPlayer cdPlayer =  new CDPlayer();
    cdPlayer.setCompactDisc(compactDisc);
    return cdPlayer;
}
```



### 2.4、通过XML装配bean

> xml 可以参考：https://www.cnblogs.com/riches/p/11520090.html

Spring 现在有了强大的自动化配置和基于 Java 的配置，XML 不应该在是你的第一选择了

#### 2.4.1、创建XML配置规范

在使用 XML 为 Spring 装配 bean 之前，你需要创建一个新的配置规范。
在使用 JavaConfig 的时候，这意味着要创建一个带有@Configuration 注解的类，而在 XML 配置中，意味着要创建一个以 `<beans>`元素为根的 XML 文件

#### 2.4.2、声明一个简单的 `<bean>`：

`<bean>`元素类似于 JavaConfig 中的@Bean 注解

```
// 创建这个bean的类通过class属性类指定的，并且要使用全限定的类名
// 建议设置id，不要自动生成，如果不指定会根据全限定类名生成，这里就会生成：soundsystem.SgtPeppers#0  后面的 0 用于区分同类型的其他 bean，例如如果在声明一个同类型没有指定 id 的，则对应生成 bean 为 xxxx#1
<bean id="compactDisc" class="soundsystem.SgtPeppers" />
```

此时无需像 JavaConfig 一样直接创建该类的实例，Spring 发现这个 `<bean>` 元素时会调用该类的默认构造器来创建 Bean。因此使用 XMl 方式创建 Bean 更加被动，同时不像 JavaConfig 中有很多方式都可以创建 Bean，xml 中只有这一种方式。

#### 2.4.3、借助构造器注入初始化bean

XML 中实现 DI 有多种配置方式，例如使用构造器注入，其中构造器注入又有两种：

- `<constructor-arg>` 元素；
- 使用 spring 3.0 说引入的 c- 命名空间（不介绍了）；

前种方式配置冗长但是功能更加完整。

1.构造器注入 bean 引用

前提：CDPlayerXml 有一个入参为 CompactDisc 类型的构造器，同时 Id 为 compactDiscImpl 的 bean 是 CompactDisc 类型的。

```java
package org.gjxaiou.beans;

// 就是一个普通的 POJO，但是因为使用构造器注入，所以需要提供构造器
public class CDPlayerXml {
    private CompactDisc compactDisc;

    public CDPlayerXml(CompactDisc compactDisc){
        this.compactDisc = compactDisc;
    }

    public void plan(){
        compactDisc.play();
    }
}

```

在 resources 目录下新建一个 xml 文件，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="compactDiscImpl" class="org.gjxaiou.beans.CompactDiscImpl">
    </bean>
    
    <bean id="xmlCd" class="org.gjxaiou.beans.CDPlayerXml">
        <constructor-arg ref="compactDiscImpl"/>
    </bean>
</beans>
```

当 Spring 遇到这个<bean>元素时，会创建一个 CDPlayerXml 实例。<constructor-arg>元素会告诉 Spring 要将一个 ID 为 compactDiscImpl 的 bean 引用传递到 CDPlayerXml 的构造器中

```java
public class XmlApp {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring-xml.xml");
        CDPlayerXml xmlCd = context.getBean("xmlCd", CDPlayerXml.class);
        xmlCd.plan();
    }
}
// 输出：hello
```

==>说明：虽然 CompactDiscImpl 类上使用 @Component，但是如果显式的通过 new ClassPath 和 new Annotation 加载 xml 和配置类，会导致构建出两个上下文，如何将两者结合下面会说明。

#### 2.将字面量注入到构造器中

上面的 DI 都是按照类型装配，即将对象的引用装配到依赖于他们的对象中。这里是只有一个字面量来配置对象。

```xml
//通过ID引用SgtPeppers
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <constructor-arg value="xxxx" />
     <constructor-arg value="yyyy" />
</bean>
```

即将 CDPlayer 中的构造方法的第一个参数设置为 xxx，第二个参数设置为 yyyy，使用 value 属性即表示给定的值要以字面量的形式注入到构造器中。

当构造器参数的类型是 java.util.List 时，使用<list>元素。set 也一样

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

#### 2.4.4、使用XML实现属性注入

==》上面依赖注入都是通过构造器，下面使用 xml 实现属性注入（即使用属性的 setter 方法）；

**通用规则：对强依赖使用构造器注入，对可选性依赖使用属性注入**

**<property>元素**为属性的 Setter 方法所提供的功能与<constructor-arg>元素为构造器所提供的功能是一样的

```java
public class CDPlayerXml {
    private CompactDisc compactDisc;

    // 提供一个 setter 方法
    public void setCompactDisc(CompactDisc compactDisc) {
        this.compactDisc = compactDisc;
    }

    public void plan(){
        compactDisc.play();
    }
}
```

同时 xml 中配置如下：

```xml
 <bean id="xmlCd" class="org.gjxaiou.beans.CDPlayerXml">
 </bean>
```

则 CDPlayerXml 这个 bean 是可以被正常创建的，因为从 CDPlayerXml 类代码可以看出该类除了默认的构造器没有任何构造器，也没有任何强依赖。但是创建完成之后 compactDisc 属性其实是空的。

将 xml 修改如下实现属性 setter 方法注入。

```
    <bean id="compactDiscImpl" class="org.gjxaiou.beans.CompactDiscImpl">
    </bean>

    <bean id="xmlCd" class="org.gjxaiou.beans.CDPlayerXml">
    <!--引用 id 为 compactDiscImpl 的 bean，并通过 setCompactDisc() 方法将其注入到 compactDisc 属性中-->
        <property name="compactDisc" ref="compactDiscImpl"/>
    </bean>
```

借助<property>元素的 value 属性注入属性

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

### 2.5、导入和混合配置

可以将 JavaConfig 的组件扫描和自动装配和/或 xml 配置混合在一起。其实本质上至少需要一些显式配置来启动组件扫描和自动装配。

#### 2.5.1、在JavaConfig中引用XML配置

我们假设 CDPlayerConfig 已经很笨重，要将其拆分，方案之一就是将 BlankDisc 从 CDPlayerConfig 拆分出来，定义到它自己的 CDConfig 类中：

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

方式一：在类在 CDPlayerConfig 中使用@Import 注解导入 CDConfig：

==该配置类中的 bean 需要 CDConfig 中的 bean，所以使用 @Import 注解

```java
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

方式二：**更好的办法是：创建一个更高级别的SoundSystemConfig，在这个类中使用@Import 将两个配置类组合在一起：**

```java
package soundsystem;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import({CDPlayerConfig.class, CDConfig.class})
public class SoundSystemConfig { 
}
```

如果有一个配置类配置在了 XML 中，则使用@ImportResource 注解

```java
@Configuration
@Import(CDPlayerConfig.class)
@ImportResource("classpath:cd-config.xml")
public class SoundSystemConfig {
}
```

2.5.2、在 XML 配置中引用 JavaConfig

- xml 中引入 xml

**在JavaConfig配置中，使用@Import和@ImportResource来拆分JavaConfig类**
**在XML中，可以使用import元素来拆分XML配置**，即这里就是将部分 bean 配置到 cdplayer-config.xml 中，然后在当前 xml 中引用该 xml；

在 xml 中，可以使用 <bean> 引入其他 JavaConfig 配置类，下面就是在当前 xml 中引入  CDConfig 配置类；

```
<beans  ....>
  <bean id = 'cdPlayer' class = "xxxx">

  <bean class="soundsystem.CDConfig" />
  <import resource="cdplayer-config.xml" />
</beans>
```

方式二：同样可以引入一个更高层级的 xml，该 xml 中就是汇总各种其他 xml/JavaConfig，不在单独配置 bean；bean 的配置都在引用的 xml 和 JavaConfig 中进行配置。

```xml
<beans  ....>
  <bean class="soundsystem.CDConfig" />
  <import resource="cdplayer-config.xml" />
</beans>
```

**不管使用JavaConfig还是使用XML进行装配，通常都会创建一个根配置，这个配置会将两个或更多的装配类/或XML文件组合起来**
**也会在根配置中启用组件扫描(通过<context:component-scan>或@ComponentScan)**

**小结：**

Spring 框架的核心是 Spring 容器。容器负责管理应用中组件的生命周期，它会创建这些组件并保证它们的依赖能够得到满足，这样的话，组件才能完成预定的任务
Spring 中装配 bean 的三种主要方式：自动化配置、基于 java 的显式配置以及基于 XML 的显示配置。这些技术描述了 Spring 应用中的组件以及这些组件之间的关系
尽可能使用自动化装配，避免显式装配所带来的维护成本。基于 Java 的配置，它比基于 XML 的配置更加强大、类型安全并且易于重构