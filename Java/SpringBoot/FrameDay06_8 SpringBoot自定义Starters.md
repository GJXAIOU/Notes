# 八、自定义 Starter

### 	编写自动配置

以 `WebMvcAutoConfiguration` 为例看如何实现自动配置，该类主要的注解如下：

- `@Configuration`   // 指定这个类是一个配置类

- `@ConditionalOnXXX` // 在指定条件成立的情况下自动配置类生效

- `@AutoConfigureAfter`  // 指定自动配置类的顺序

- `@Bean`  // 给容器中添加组件

- `@ConfigurationPropertie`  // 结合相关xxxProperties类来绑定相关的配置，都是用于控制 `@Configuration` 是否生效

    源码详细分析：读取配置文件中的值。

    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ ElementType.TYPE, ElementType.METHOD })
    @Documented
    @Conditional(OnPropertyCondition.class)
    public @interface ConditionalOnProperty {
        // 数组，获取对应 property 名称的值，与 name 不可同时使用
    	String[] value() default {};
        //property名称的前缀，可有可无
    	String prefix() default "";
        //数组，property完整名称或部分名称（可与prefix组合使用，组成完整的property名称），与value不可同时使用
    	String[] name() default {};
    //可与name组合使用，比较获取到的属性值与havingValue给定的值是否相同，相同才加载配置
    	String havingValue() default "";
    //缺少该property时是否可以加载。如果为true，没有该property也会正常加载；反之报错
    	boolean matchIfMissing() default false;
    //是否可以松散匹配，至今不知道怎么使用的
    	boolean relaxedNames() default true;
    }
    
    ```

- `@EnableConfigurationProperties` // 让xxxProperties生效加入到容器中

- `@ConditionalOnMissingBean` 一般配合 `@Bean` // 只有特定名称或者类型的Bean不存在于BeanFactory中时才创建某个Bean

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
    
    // XXXXXXXXXXXXX
    	@Bean
		@ConditionalOnMissingBean
		@ConditionalOnProperty(prefix = "spring.mvc", name = "locale")
		public LocaleResolver localeResolver() {
			if (this.mvcProperties
					.getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED) {
				return new FixedLocaleResolver(this.mvcProperties.getLocale());
			}
			AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
			localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
			return localeResolver;
		}
   // XXXXXXXXXXX
}
```

同时自动配置类要能加载生效，需要启动就加载的自动配置类，配置在 META-INF/spring.factories 中，示例如下

![image-20201109194834634](FrameDay06_8%20SpringBoot%E8%87%AA%E5%AE%9A%E4%B9%89Starters.resource/image-20201109194834634.png)





### 模式：**启动器（starter）**

启动器模块是一个空 JAR  文件，仅提供**辅助性依赖管理**，这些依赖可能用于自动装配或者其他类库。

例如 pom 文件中对应的一个关于 Web 的 Starter，`spring-boot-starter-web` 对应的 Jar 包中是不含有任何 Java 代码的。然后启动器依赖点开之后，发现其引入了自动配置依赖 `spring-boot-autoconfigure`。

所以：启动器只用来做依赖导入；同时专门来写一个自动配置模块；其中启动器依赖自动配置模块；别人只需要引入启动器（starter）即可（同时也就引入了自动配置器）。

<img src="FrameDay06_8%20SpringBoot%E8%87%AA%E5%AE%9A%E4%B9%89Starters.resource/image-20201109203509429.png" alt="image-20201109203509429" style="zoom:67%;" />

以 MyBatis 官方提供的 stater 和自动配置类结构为例：

![image-20201112140428678](FrameDay06_8%20SpringBoot%E8%87%AA%E5%AE%9A%E4%B9%89Starters.resource/image-20201112140428678.png)

自定义启动器命名规约：

- 官方命名空间
    - 前缀：`spring-boot-starter-`
    - 模式：`spring-boot-starter-模块名`
    - 举例：`spring-boot-starter-web`、`spring-boot-starter-actuator`、`spring-boot-starter-jdbc`
- 自定义命名空间
    - 后缀：`-spring-boot-starter`
    - 模式：`模块-spring-boot-starter`
    - 举例：`mybatis-spring-boot-starter` 即规则为： `自定义启动器名-spring-boot-starter`

### 自定义步骤

新建一个空的项目，然后新增两个模块：一个是启动器模块：`gjxaiou-spring-boot-starter`，一个是自动配置模块：`gjxaiou-spring-boot-autoconfigurer`。

![image-20201112141133913](FrameDay06_8%20SpringBoot%E8%87%AA%E5%AE%9A%E4%B9%89Starters.resource/image-20201112141133913.png)

模块一：启动器模块，仅仅用于依赖引入。空项目，只要在 pom.xml 中如下引入自动配置类依赖即可。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.starter</groupId>
    <artifactId>atguigu-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--启动器-->
    <dependencies>
        <!--引入自动配置模块-->
        <dependency>
            <groupId>com.atguigu.starter</groupId>
            <artifactId>atguigu-spring-boot-starter-autoconfigurer</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```

模块二：自动配置模块

首先 pom.xml 文件中依赖如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.atguigu.starter</groupId>
    <artifactId>atguigu-spring-boot-starter-autoconfigurer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>atguigu-spring-boot-starter-autoconfigurer</name>
    <description>Demo project for Spring Boot</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.10.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--首先引入spring-boot-starter；是所有 starter 的基本配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
</project>
```

首先是基本的属性配置

```java
package com.atguigu.starter;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "atguigu.hello")
public class HelloProperties {

    private String prefix;
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}

```



```java
package com.atguigu.starter;

public class HelloService {

    HelloProperties helloProperties;

    public HelloProperties getHelloProperties() {
        return helloProperties;
    }

    public void setHelloProperties(HelloProperties helloProperties) {
        this.helloProperties = helloProperties;
    }

    public String sayHellAtguigu(String name) {
        return helloProperties.getPrefix() + "-" + name + helloProperties.getSuffix();
    }
}
```

自动配置类

```java
package com.atguigu.starter;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 自动配置类
 */
@Configuration
@ConditionalOnWebApplication //web应用才生效
// 使得 HelloProperties 属性文件生效，就可以直接用里面的属性了。
@EnableConfigurationProperties(HelloProperties.class)
public class HelloServiceAutoConfiguration {

    @Autowired
    HelloProperties helloProperties;
    @Bean
    public HelloService helloService(){
        HelloService service = new HelloService();
        service.setHelloProperties(helloProperties);
        return service;
    }
}
```

同时为了让上面的自动配置类生效，需要在 `resources` 文件夹路径下面新建文件 `META-INF/spring.factories`，里面的配置为：

```xml
# 为了让 HelloServiceAutoConfiguration 自动配置类生效
# 这里是配置启动的时候要启动哪些自动配置类
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.atguigu.starter.HelloServiceAutoConfiguration
```



#### 使用

首先将上面两个装入 maven仓库中可以方便别人引用。同时因为 `XXX-starter` 依赖 `XXX-starter-autoConfiguration`，因为需要先将 `autoCongfiguration` 先进行按照，步骤如下：

在 IDEA 右边 「Maven」中选择该模块，然后依次点击 「Lifecycle」-> 「install」进行安装，然后安装 starter，步骤一样。



测试项目

首先在项目的 pom.xml 文件中引入上述启动器的依赖：

```xml
<!--引入自定义的starter，只需要引入启动器的依赖即可-->
<dependency>
    <groupId>org.example</groupId>
    <artifactId>gjxaiou-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

在 Controller 类中直接调用包中的 Service，同时对应前后缀可以直接在 `application.properties`中进行配置

```java
package com.atguigu.springboot.controller;

import com.atguigu.starter.HelloService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    // 导入包中的 helloService
    @Autowired
    HelloService helloService;

    @GetMapping("/hello")
    public String hello() {
        return helloService.sayHellAtguigu("haha");
    }
}
```

属性配置为：

```properties
atguigu.hello.prefix=ATGUIGU
atguigu.hello.suffix=HELLO WORLD
```

启动之后访问对应的路径即可。







更多SpringBoot整合示例

https://github.com/spring-projects/spring-boot/tree/master/spring-boot-samples