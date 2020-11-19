# Spring Boot 入门

前八章：springboot 的基本使用和内部原理

后八章：springboot 的高级场景整合

## 一、Spring Boot 简介

是简化 Spring 应用开发的一个框架；同时也是整个 Spring 技术栈的一个大整合；提供了 J2EE 开发的一站式解决方案；

## 二、微服务

- 微服务：架构风格（服务微化）

- 一个应用应该是一组小型服务，之间可以通过 HTTP 的方式进行互通；

- 之前：单体应用：ALL IN ONE

- 微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

[详细参照微服务文档](https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa)

## 三、环境准备

- jdk1.8：Spring Boot 推荐 jdk1.7 及以上；java version "1.8.0_112"

- maven3.6：maven 3.3 以上版本；Apache Maven 3.3.9

- IntelliJ IDEA2019：

- SpringBoot 2.2.1.RELEASE；

    注：在 `spring-boot-dependencies-2.2.1.RELEASE.pom `中 可以看到 ==内嵌的 Tomcat 版本为：9.0.27==（L222）

### （一）MAVEN 设置

给 maven 的 settings.xml 配置文件的 `<profiles></profiles>`标签添加

```xml
<profile>
  <id>jdk-1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

## 四、Spring Boot： HelloWorld 实现

实现的功能：浏览器发送 hello 请求，服务器接受请求并处理，响应 Hello World 字符串；

### （一）项目搭建

- 方式一：首先创建一个 maven 工程；（jar），然后导入 spring boot 相关的依赖（maven.xml）

```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
    </parent>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```

- 方式二：File -> New Project -> Spring initializr (选择jdk版本) -> next -> 填写信息（注意修改最后的 package，同时 Artifact 不能有大写字母 ），然后默认 maven 会自动导入相关的依赖。



### （二）在主程序中启动Spring Boot应用

```java
package com.gjxaiou;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {
        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

### （三）编写相关的 Controller

```java
package com.gjxaiou.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "hello world";
    }
}
```

### （四）运行主程序测试

只要直接执行上面的主程序，就可以直接在浏览器中访问：`localhost:8080/hello`，结果图如下：

![image-20191223163759281](FrameDay06_1%20SpringBoot%E5%85%A5%E9%97%A8.resource/image-20191223163759281.png)

### （五）简化部署（直接将项目打成 jar，然后在命令行运行即可）

**打成 Jar 包**

不在需要将项目导出为 war 包然后部署到服务器中，只需要在 maven 中导入下面插件，然后使用右边的 `maven Project -> lifecycle -> pageage(右击:run 项目名[package])` 命令就可以直接将项目打成可执行的 jar 包，默认的 jar 包位置在 项目的 target 文件夹下面；可以直接进入该文件夹，然后在文件路径框输入 cmd;

```xml
 <!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

**运行 Jar 包**：将这个应用打成 jar 包之后，直接使用 java -jar 的命令进行执行；`java -jar Jar包位置和名称`

## 五、Hello World 原理探究

### （一）POM文件

#### 1、父项目：作为版本仲裁作用

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.1.RELEASE</version>
</parent>

<!--点击上面之后可以看到他的父项目是：spring-boot-dependencies-2.2.1.RELEASE.pom，下面是部分-->

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.2.1.RELEASE</version>
    <relativePath>../spring-boot-dependencies</relativePath>
  </parent>
<!--再次点击<artifactId> 里面内容 打开之后，可以看到下面有属性<properties>，因此他来真正管理 Spring Boot 应用里面的所有依赖 版本,下面为部分-->

 <properties>
    <activemq.version>5.15.10</activemq.version>
    <spring-hateoas.version>1.0.1.RELEASE</spring-hateoas.version>
    <spring-integration.version>5.2.1.RELEASE</spring-integration.version>
    <sqlite-jdbc.version>3.28.0</sqlite-jdbc.version>
    <sun-mail.version>${jakarta-mail.version}</sun-mail.version>
  </properties>
```

上面的就是 Spring Boot 的版本仲裁中心；

**以后我们导入依赖默认是不需要写版本**；（没有在 dependencies 里面管理的依赖自然需要声明版本号）

#### 2、启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

- 上面的父项目只是作为版本仲裁，真正的 jar 包导入是由下面这个`spring-boot-starter-web`完成的：

- spring-boot-starter：spring-boot 场景启动器，是所有 spring-boot-starter-XXX 的父项目；
- spring-boot-starter-XXX ：作用是：帮我们导入了 XXX（这里是 web） 模块正常运行所依赖的组件即 Jar 包；（点击进去就能看见 dependency了） 

- **Spring Boot 将所有的功能场景都抽取出来，做成一个个的 starters（启动器），只需要在项目里面引入这些 starter 相关场景，则所有依赖都会导入进来。要用什么功能就导入什么场景的启动器，所有 starter 见官网**。

[所有 Starter 介绍见官网 1.5. Starters，下面为部分](https://docs.spring.io/spring-boot/docs/2.2.1.RELEASE/reference/html/using-spring-boot.html#using-boot-starter)

| Name                           | Description                                                  | Pom                                                          |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter`          | Core starter, including auto-configuration support, logging and YAML | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.1.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq` | Starter for JMS messaging using Apache ActiveMQ              | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.1.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp`     | Starter for using Spring AMQP and Rabbit MQ                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.1.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop`      | Starter for aspect-oriented programming with Spring AOP and AspectJ | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.1.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |

### （二）主程序类，主入口类

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个 Spring Boot 应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        // Spring 应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

- `@SpringBootApplication`:    该注解标注在某个类上说明这个类是 SpringBoot 的主配置类，SpringBoot 就应该运行这个类的 `main()` 方法来启动 SpringBoot 应用；

- 该注解本质上是一个组合注解，由下面一系列注解组成（点击即可查看）：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration // SpringBoot 的配置类；
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    // 此处省略：XXXXXXX
}
```

下面分别解释一下上面所有的注解含义：

- 注解一：`@SpringBootConfiguration`：SpringBoot 的配置类的注解；

    - 标注在某个类上，表示这是一个 SpringBoot 的配置类；

      其本质上是 spring 定义的注解（点击内容见下），其中 `@Configuration`：即使用该注解来标注配置类；（为了简化 spring 通常将配置文件替换为配置类来使用，然后使用上述的标识来标识为配置类）。

    ```java
    @Target({ElementType.TYPE})
    @Retention(RetentionPolicy.RUNTIME)
    @Documented
    @Configuration  // spring 定义的表示配置类的注解
    public @interface SpringBootConfiguration {
        @AliasFor(
            annotation = Configuration.class
        )
        boolean proxyBeanMethods() default true;
    }
    
    ```
    - 配置类对应于配置文件；配置类也是容器中的一个组件；@Component

- 注解二：`@EnableAutoConfiguration`：开启自动配置功能；

    以前我们需要配置的东西，SpringBoot 帮我们自动配置；使用 `@EnableAutoConfiguration`告诉 SpringBoot 开启自动配置功能；

    - 注解二-1：该注解点击之后是一个组合注解（如下），其本质是使用 `@AutoConfigurationPackage` 这个注解进行自动配置（点击）；

        ```java
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @AutoConfigurationPackage
        @Import(AutoConfigurationImportSelector.class)
        public @interface EnableAutoConfiguration {
        ```

        而点击 `@AutoConfigurationPackage` 得到，其中主要是 `@Import(AutoConfigurationPackages.Registrar.class)`

        ```java
        @Target(ElementType.TYPE)
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Inherited
        @Import(AutoConfigurationPackages.Registrar.class)  // spring 的底层注解
        public @interface AutoConfigurationPackage {
        
        }
        ```

        `@Import(AutoConfigurationPackages.Registrar.class)`：

        `@Import()`属于 Spring 的底层注解给容器中导入一个组件；导入的组件由`AutoConfigurationPackages.Registrar.class` 类来指定；点击之后在 `registrar` 类中的 `registerBeanDefinitions()` 方法（L122）

        ```java
        static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
        
        @Override
        public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
            // 给容器中导入组件的具体执行方法， metadata 就是这个注解标注的元信息
        	register(registry, new PackageImport(metadata).getPackageName());
        }
        ```

        通过在 register 那行打断点然后 debug 之后，因为 metadata 是这个注解的元信息，然后打开这个注解的元信息，其中元数据 metadata 中的 注解（看调试窗口的 Variables 中 的 metadata 的 annotations ）是标注在 SPringBootApplication 中的，然后下面的 introspectedClass 可以看出是标注在 HelloXXXX () 方法上面的，然后选择`new PackageImpot(metadata).getPackageName()`，右击选择`evalate expression`，计算包值得到 `result = com.gjxiaou`

        因此 `@AutoConfigurationPackage` 真正的是含义是：==将主配置类（用 @SpringBootApplication 标注的类）的所在包及下面所有子包里面的所有组件扫描到 Spring 容器；==，所以这里可以将 controller 类扫描进去。

    - 注解二 - 2：`@Import(AutoConfigurationImportSelector.class)`；`AutoConfigurationImportSelector` 类是导入哪些组件的选择器；该类中的主要方法是：`public String[] selectImports(AnnotationMetadata annotationMetadata) {XXX}`,上面方法的父类本质上是将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；

        最终作用是会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；自动配置类的作用就是给容器中导入这个场景需要的所有组件，并配置好这些组件；		![自动配置类](FrameDay06_1%20SpringBoot%E5%85%A5%E9%97%A8.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180129224104.png)

        有了自动配置类，免去了我们手动编写配置和注入功能组件等的工作；

        主要是通过方法		SpringFactoriesLoader.loadFactoryNames(EnableAutoConfiguration.class,classLoader)；作用就是：==Spring Boot 在启动的时候从类路径下的 META-INF/spring.factories （该文件在 org.springframework.boot:spring-boot-autoconfigure:jar）中获取 EnableAutoConfiguration 指定的值，将这些值作为自动配置类导入到容器中，这样自动配置类就生效，从而帮我们进行自动配置工作;==以前我们需要自己配置的东西，自动配置类都帮我们做了；

    J2EE的整体整合解决方案和自动配置都在spring-boot-autoconfigure-2.2.1.RELEASE.jar 中；

## 六、使用 Spring Initializer 快速创建 Spring Boot 项目

上面的项目需要创建项目，然后导包并且配置 controller 和启动器，可以使用第四点项目搭建中的方式二实现：

### IDEA：使用 Spring Initializer 快速创建项目

默认生成的 Spring Boot 项目特点；

- 主程序已经生成好了，我们只需要完成我们自己的逻辑；
- resources 文件夹中目录结构
  - static：保存所有的静态资源，包括 js、css、images；
  - templates：保存所有的模板页面；（因为 Spring Boot 默认 jar 包使用嵌入式的 Tomcat，默认不支持JSP 页面）；可以使用模板引擎（freemarker、thymeleaf）来支持 JSP；
  - application.properties：Spring Boot 应用的配置文件，可以修改一些默认设置；
