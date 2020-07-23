# Part2：Web 中的 Spring

## 章五：构建 Spring Web 应用程序

本章中，将会接触到Spring MVC基础，以及如何编写控制器来处理web请求，如何通明地绑定请求参数到业务对象上，同时还可以提供数据校验和错误处理的功能。

### 一、 Spring MVC初探
基于模型- 视图-控制（MVC）
spring 将请求在调度 Servlet、处理器映射（handler Mapping）、控制器以及视图解析器（view resolver ）之间移动。

#### （一）跟踪Spring MVC请求
请求会由DispatcherServlet分配给控制器（根据处理器映射来确定），在控制器完成处理后，接着请求会被发送给一个视图来呈现结果
![spring 处理请求过程]($resource/spring%20%E5%A4%84%E7%90%86%E8%AF%B7%E6%B1%82%E8%BF%87%E7%A8%8B.png)

- 在请求离开浏览器时，会带有用户所请求内容的信息，例如请求的URL、用户提交的表单信息。

- 请求旅程的第一站是Spring的DispatcherServlet。Spring MVC所有的请求都会通过一个前端控制器Servlet。前端控制器是常用的Web应用程序模式，在这里**一个单实例的Servlet将请求委托给应用程序的其他组件来执行实际的处理**。在Spring MVC中，DispatcherServlet 就是前端控制器。

- DispatcherServlet的任务是将请求发送给Spring MVC**控制器**。控制器是一个用于处理请求的Spring组件。在典型的应用程序中可能会有多个控制器， Dispatcher Servlet需要知道应该将请求发送给哪个控制器。所以DispatcherServlet会查询一个或多个处理器映射来确定请求的下一站在哪里。**处理器映射**会根据请求所携带的URL信息来进行决策。

- 一旦选择了合适的控制器，DispatcherServlet会将请求发送给选中的控制器。到达了控制器，请求会卸下其负载（用户提交的信息）并等待控制器处理这些信息（实际上，**设计良好的控制器本身只处理很少甚至不处理工作，而是将业务逻辑委托给个或多个服务对象**）。

- 控制器在完成逻辑处理后通常会产生一些信息，这些信息需要返回给用户并在浏览器上显示。这些信息被称为**模型**（Model）。不过仅仅给用户返回原始的信息是不够的–---这些信息需要以用户友好的方式进行格式化，一般是HTML。所以，信息需要发送给—个**视图**（View），通常会是JSP。

- 控制器所做的最后一件事是**将模型数据打包**，并且标示出用于渲染输出的视图名称。**它接下来会将请求连同模型和视图名称发送回DispatcherServlet**。

- 这样，控制器就不会与特定的视图相耦合，传递给DispatcherServlet的视图名称并不直接表示某个特定的JSP。它仅仅传递了一个逻辑名，这个名字将会用来查找用来产生结果的真正视图。DispatcherServlet将会使用**视图解析器**来将逻辑视图名匹配为一个特定的视图实现。

- 既然DispatcherServlet已经知道由哪个视图渲染结果，那么请求的任务基本上也就完成了。它的最后一站是视图的实现（可能是JSP），在这里它交付模型数据。请求的任务就完成了。视图将使用模型数据渲染输出，并通过这个输出将响应对象传递给客户端。

#### （二）搭建Spring MVC

- ##### 配置DispatcherServlet

DispatcherServlet是Spring MVC的核心，**它负责将请求分发到其他各个组件**。

在旧版本中，DispatcherServlet之类的servlet一般在`web.xml`文件中配置，该文件一般会打包进最后的war包中；但是Spring3引入了注解，我们在这一章将展示如何基于注解配置Spring MVC。

**注意：**
在使用maven构建web工程时，由于缺少web.xml文件，可能会出现`web.xml is missing and <failOnMissingWebXml> is set to true`这样的错误，那么可以通过在pom.xml文件中添加如下配置来避免这种错误：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-war-plugin</artifactId>
            <version>2.6</version>
            <configuration>
                <failOnMissingWebXml>false</failOnMissingWebXml>
            </configuration>
        </plugin>
    </plugins>
</build>
```

**这里不使用 web.xml  配置**：既然不使用`web.xml`文件，你**需要在servlet容器中使用Java配置DispatcherServlet**，具体的代码列举如下：

```java
package spittr.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class SpittrWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected String[] getServletMappings() {
        // 将 DispatcherServlet 映射到：/
        return new String[] { "/" };
    }

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { WebConfig.class };
    }

}
```
**从 类名： SpittrWebAppInitializer**，得知要创建的应用名为：Spitter；

任意继承自`AbstractAnnotationConfigDispatcherServletInitializer`的类都会被自动配置DispatcherServlet 和 Spring 应用上下文（Spring 的应用上下文会位于应用程序的 Servlet 上下文中），这个类负责**配置DispatcherServlet**、**初始化Spring MVC容器和Spring容器**。

上面那段的原理：因为容器会在类路径中查找实现 javax.servlet.ServletContainerInitializer 接口的类，如果发现就用它配置 Servlet 容器，而 Spring 中提供了该接口的实现，名为：SpringServletContainerInitializer，该类又会查找实现 WebApplicationInitializer 的类，并将配置的任务交给他们实现，Spring 3.2 中提供了前面那个 WebXXXzer 的实现，就是上面的 AbstractXXXXXzer，同时我们的 SpittrWebXXX 拓展了上面 AbstractXXXzer ，因此当部署到 Servlet 3.0 容器中的时候，容器就会自动发现它，并用它来 配置 Servlet 上下文。

SpittrWebAppInitializer重写了三个方法，`getRootConfigClasses()`方法用于获取Spring应用容器的配置文件，这里我们给定预先定义的RootConfig.class；`getServletConfigClasses()`负责获取SpringMVC应用容器，这里传入预先定义好的WebConfig.class；`getServletMappings()`方法负责指定需要由DispatcherServlet映射的路径，这里给定的是”/”，意思是由DispatcherServlet处理所有向该应用发起的请求。

##### - 两种应用上下文

当DispatcherServlet启动时，会创建一个Spring应用上下文并且会加载配置文件中声明的bean，通过`getServletConfigClasses()`方法，DispatcherServlet会加载`WebConfig`配置类中所配置的bean。

在Spring web应用中，通常还有另外一种应用上下文：`ContextLoaderListener`。

**DispatcherServlet用来加载web组件bean**，如控制器（controllers）、视图解析器（view resolvers）以及处理器映射（handler mappings）等。而**ContextLoaderListener则用来加载应用中的其他bean**，如运行在应用后台的中间层和数据层组件。

AbstractAnnotationConfigDispatcherServletInitializer会同时创建DispatcherServlet和ContextLoaderListener。`getServletConfigClasses()`方法返回的`@Configuration`类会定义DispatcherServlet应用上下文的bean。同时，`getRootConfigClasses()`返回的`@Configuration`类用来配置ContextLoaderListener上下文创建的bean。

相对于传统的`web.xml`文件配置的方式，通过AbstractAnnotationConfigDispatcherServletInitializer来配置DispatcherServlet是一种替代方案。需要注意的是，这种配置只适用于**Servlet 3.0**，例如Apache Tomcat 7或者更高。

##### -  激活Spring MVC

正如有多种方式可以配置DispatcherServlet，激活Spring MVC组件也有不止一种方法。一般的，都会通过XML配置文件的方式来配置Spring，例如可以通过`<mvc:annotation-driven>`来激活基于注解的Spring MVC。

最简单的配置Spring MVC的一种方式是通过`@EnableWebMvc`注解：

```java
package spittr.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc
public class WebConfig {

}
```

`@Configuration`表示这是Java配置类；`@EnableWebMvc`注解用于启动Spring MVC特性。

这样就可以激活Spring MVC了，但是还有其他一些问题：
- 没有配置视图解析器（view resolvers），这种情况下，Spring会默认使用`BeanNameViewResolver`，它会通过寻找那些与视图id匹配的bean以及实现了View接口的类进行视图解析；
- 没有激活组件扫描：这样Spring会寻找配置中明确声明的任意控制器；
- DispatcherServlet会处理所有的请求，包括静态资源请求，如图片和样式（这些往往不是我们想要的）。

因此，需要为WebConfig增加一些配置：

```java
package spittr.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@Configuration
@EnableWebMvc// 激活Spring MVC
@ComponentScan("spitter.web") // 启动组件扫描
public class WebConfig extends WebMvcConfigurerAdapter {

    // 配置一个JSP视图解析器
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB_INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }

    // 配置静态资源处理
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```

首先需要注意的是，`WebConfig`使用了`@ComponentScan`注解，因此会在`spitter.web`包下扫描寻找组件，这些组件包括使用`@Controller`进行注解的控制器。这样就不再需要在配置类中显式地声明其他控制器。

接下来，添加了一个`ViewResolver`bean，即`InternalResourceViewResolver`。它通过匹配符合设置的前缀和后缀的视图来用来**寻找对应的JSP文件**，比如视图home会被解析为/WEB-INF/views/home.jsp。这里的三个函数的含义依次是：`setPrefix()`方法用于设置视图路径的前缀；`setSuffix()`用于设置视图路径的后缀，即如果给定一个逻辑视图名称——”home”，则会被解析成”/WEB-INF/views/home.jsp”； `setExposeContextBeansAsAttributes(true)`使得可以在JSP页面中通过${}访问容器中的bean。

然后，`WebConfig`继承自`WebMvcConfigurerAdapter`，并且重写了`configureDefaultServletHandling()`方法，通过调用`enable()`方法从而可以让DispatcherServlet将静态资源的请求转发给默认的servlet。

```java
package spittr.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@ComponentScan(basePackages = { "spitter" }, excludeFilters = {
        @Filter(type = FilterType.ANNOTATION, value = EnableWebMvc.class) })
public class RootConfig {

}
```

需要注意的一点是，RootConfig 使用了`@ComponentScan`注解。

#### （三）Spittr应用介绍

这一章要用的例子应用，从Twitter获取了一些灵感，因此最开始叫Spitter；然后又借鉴了最近比较流行的网站Flickr，因此我们也把e去掉，最终形成Spittr这个名字。这也有利于区分领域名称（类似于twitter，这里用spring实现，因此叫spitter）和应用名称。

Spittr类似于Twitter，用户可以通过它添加一些推文。Spittr有两个重要的概念：_spitter_（应用的用户）和_spittle_（用户发布简单状态）。本章将会构建该应用的web层、创建用于展示spittle的控制器以及用户注册流程。

### 二、编写简单的控制器

Spring MVC中，**控制器仅仅是拥有`@RequestMapping`注解方法的类，从而可以声明它们可以处理何种请求**。

在开始之前，我们先假设一个控制器，它可以处理匹配`/`的请求并会跳转到主页面。

```java
package spittr.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller // 声明一个控制器，基于@Component 注解，用于辅助实现组件扫描
public class HomeController {

    @RequestMapping(value = "/", method = RequestMethod.GET) // 处理对"/" 的GET请求
    public String home() {
        return "home";// 视图名为 home
    }

}
```

`@Controller`是一个构造型注解，它基于`@Component`，组件扫描器会自动地将HomeController声明为Spring上下文的一个bean。

home()方法采用了`@RequestMapping`注解，**属性`value`指定了该方法处理的请求路径，`method`方法指定了可以处理的HTTP方法。这种情况下，当一个来自`/`的GET方法请求时，就会调用home()方法**。

home()方法仅仅返回了一个”home”的String值，Spring MVC会对这个String值进行解析并跳转到指定的视图上。`DispatcherServlet`则会请求视图解析器将这个逻辑视图解析到真实视图上。

我们已经配置了InternalResourceViewResolver，“home”视图会被解析到`/WEB-INF/views/home.jsp`。

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>

<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>

<html>
<head>
<meta charset="utf-8">
<title>Spittr</title>
<link rel="stylesheet" type="text/css"
    href="<c:url value="/resources/style.css" />">
</head>
<body>
    <h1>Welcome to Spittr</h1>
    <a href="<c:url value="/spittles" />">Spittles</a> |
    <a href="<c:url value="/spitter/register" />">Register</a>
</body>
</html>
```

下面对HomeController进行测试。

#### （一）测试控制器

一般的web测试需要将工程发布到一个web容器中，启动后才能观察运行结果。

从另外的角度来看，HomeController其实是一个简单的POJO对象，那么可以使用下面的方法对其进行测试：

```java
package spittr.web;

import org.junit.Test;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

public class HomeControllerTest {

    @Test
    public void testHomePage() throws Exception {
        HomeController controller = new HomeController();
        // 设置MockMvc
        MockMvc mockMvc = MockMvcBuilders.standaloneSetup(controller).build();
        mockMvc.perform(MockMvcRequestBuilders.get("/")).andExpect(MockMvcResultMatchers.view().name("home"));
    }

}
```

相对于直接调用home()方法测试它的返回值，上面的测试中发起了一个来自`/`的 GET 请求，并且对其结果视图进行断言。将HomeController的实例传送给`MockMvcBuilders.standaloneSetup`，并且调用`build()`方法来创建一个`MockMvc`实例。然后，使用`MockMvc`实例产生了一个GET请求，并且设置了视图的期望。

#### （二）定义类层级的请求处理

```java
package spittr.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller // 声明一个控制器
@RequestMapping("/") // 控制器匹配路径
public class HomeController {

    @RequestMapping(method = RequestMethod.GET) // 处理GET请求
    public String home() {
        return "home";// 视图名称
    }

}
```

在这个新版的HomeController中，将请求匹配路径移到了类层级，HTTP方法的匹配仍处在方法层级。当有控制类中有一个类层级的`@RequestMapping`，该类中所有的用`@RequestMapping`注解的处理方法共同组成了类层级的`@RequestMapping`。

`@RequestMapping`的value属性接受String数组，那么就可以使用如下配置：

```java
@Controller // 声明一个控制器
@RequestMapping("/", "/homepage") // 控制器匹配路径
public class HomeController {
...
}
```

这种情况下，home()方法就可以处理来自`/`和`/homepage`的GET请求。

#### （三）将model数据传送给视图

在Spittr应用中，需要一个页面，用来显示最近提交的spittle清单。首先需要定义一个数据访问的仓库，用来抓取spittle：

```java
package spittr.data;

import java.util.List;
import spittr.Spittle;

public interface SpittleRepository {
    /**
     * @param max
     *            待返回的最大的Spittle ID
     * @param count
     *            返回Spittle对象的个数
     * @return
     */
    List<Spittle> findSpittles(long max, int count);
}
```

如果要获取最近的20个Spittle对象，那么只需调用这样调用：
`List<Spittle> recent = spittleRepository.findSpittles(Long.MAX_VALUE, 20);`

下面对Spittle进行定义：

```java
package spittr;

import java.util.Date;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Spittle {
    private final Long id;
    private final String message;// 消息
    private final Date time;// 时间戳
    private Double latitude;
    private Double longitude;

    public Spittle(String message, Date time) {
        this(message, time, null, null);
    }

    public Spittle(String message, Date time, Double latitude, Double longitude) {
        this.id = null;
        this.message = message;
        this.time = time;
        this.latitude = latitude;
        this.longitude = longitude;
    }

    @Override
    public boolean equals(Object that) {
        return EqualsBuilder.reflectionEquals(this, that, "id", "time");
    }

    @Override
    public int hashCode() {
        return HashCodeBuilder.reflectionHashCode(this, "id", "time");
    }
}
```

Spittle对象中现在包含信息、时间戳、位置这几个属性。

下面利用Spring的`MockMvc`来断言新的控制器的行为是否正确：

上面的测试通过创建一个SpittleRepository接口的mock实现，该实现会通过findSpittles()方法返回一个包含20个Spittle对象的集合。然后将这个bean注入到SpittleController实例中，并设置MockMvc使用该实例。

不同于HomeControllerTest，该测试使用了`setSingleView()`，发起一个`/spittles`的GET请求，并断言视图是否为spittles以及model是否含有一个spittleList的属性值。

当然，现在运行这个测试代码肯定是会出错的，因为还没有SpittleController。

```java
package spittr.web;

import java.util.ArrayList;
import java.util.Date;
import java.util.List;

import org.hamcrest.core.IsCollectionContaining;
import org.junit.Test;
import org.mockito.Mockito;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.servlet.view.InternalResourceView;

import spittr.Spittle;
import spittr.data.SpittleRepository;

public class SpittleControllerTest {

    @Test
    public void shouldShowRecentSpittles() throws Exception {
        List<Spittle> expectedSpittles = createSpittleList(20);
        // 首先创建 SpittleRepository 接口的 mock 实现，这个实现从 findSpittles() 方法中返回 20 个 Spittle 对象；
        SpittleRepository mockRepository = Mockito.mock(SpittleRepository.class);
        Mockito.when(mockRepository.findSpittles(Long.MAX_VALUE, 20)).thenReturn(expectedSpittles);

        SpittleController controller = new SpittleController(mockRepository);
        MockMvc mockMvc = MockMvcBuilders.standaloneSetup(controller)
                .setSingleView(new InternalResourceView("/WEB_INF/views/spittles.jsp")).build();

        // 调用MockMvc.perform(RequestBuilder requestBuilder)发起一个http请求，然后将得到ResultActions
        mockMvc.perform(MockMvcRequestBuilders.get("/spittles"))// 添加验证断言来判断执行请求后的结果是否是预期的；
                .andExpect(MockMvcResultMatchers.view().name("spittles"))// view()：得到视图验证器；
                // 得到相应的***ResultMatchers后，接着再调用其相应的API得到ResultMatcher，
                // 如ModelResultMatchers.attributeExists(final String... names)判断Model属性是否存在。
                .andExpect(MockMvcResultMatchers.model().attributeExists("spittleList"))// model()：得到模型验证器；
                .andExpect(MockMvcResultMatchers.model().attribute("spittleList", IsCollectionContaining.hasItems(expectedSpittles.toArray())));
    }

    private List<Spittle> createSpittleList(int count) {
        List<Spittle> spittles = new ArrayList<Spittle>();
        for (int i = 0; i < count; i++) {
            spittles.add(new Spittle("Spittle ", new Date()));
        }
        return spittles;
    }

}
```

SpittleController中，使用@Autowired注解注入了spittleRepository属性。



==到P151 页了，少了一个 Controller 方法==



需要注意的是`spittles()`方法使用了Model（_控制器和视图之间传递的数据_）作为入参，Model本质上是一个map，它会被传送至view，因此数据可以提供给客户端。如果在调用`addAttribute()`方法时没有指定key，那么就会从传入的对象中获取，比如代码中传入的参数属性是List，那么key就是spittleList。最后，该方法返回spittles作为传动给model的视图名称。

也可以显示的指定key：

```java
model.addAttribute(spittleRepository.findSpittles(Long.MAX_VALUE, 20));
```

也可以直接采用map的方式：

```java
@RequestMapping(method = RequestMethod.GET)
public String spittles(Map model) {
    // 将spittles添加到model中
    model.put("spittles", spittleRepository.findSpittles(Long.MAX_VALUE, 20));

    // 返回视图名称
    return "spittles";
}
```

不管采用何种方式实现spittles()方法，结果都是一样的。一个Spittle对象集合会存储在model中，并分配到名为spittles的view中，根据测试方法中的配置，该视图就是/WEB-INF/views/spittles.jsp。

现在model已经有数据了，那么JSP页面中如何获取数据呢？当视图是一个JSP页面时，model数据会作为请求属性被拷贝到请求中，因此可以通过JSTL（JavaServer Pages Standard Tag Library）`<c:forEach>`来获取：

```html
<c:forEach items="${spittleList}" var="spittle">
    <li id="spittle_<c:out value="spittle.id"/>">
        <div class="spittleMessage">
            <c:out value="${spittle.message}" />
        </div>
        <div>
            <span class="spittleTime"><c:out value="${spittle.time}" /></span>
            <span class="spittleLocation"> (<c:out
                    value="${spittle.latitude}" />, <c:out
                    value="${spittle.longitude}" />)
            </span>
        </div>
    </li>
</c:forEach>
```

下面对SpittleController进行扩展，让它可以处理一些输入。

# 接受输入请求

Spring MVC提供了如下方式供客户端传递数据到控制器处理方法：
- Query parameters
- Form parameters
- Path variables

## 处理查询参数：@RequestParam

Spittr应用的一个需求就是要对spittle列表分页展示，但是SpittleController仅仅展示最近的spittle。如果要让用户可以每次得到一页的spittle记录，那么就需要让用户可以通过某种方式将他们想看的spittle记录的参数传递到后台。

在浏览spittle时，如果想要查看下一页的spittle，那么就需要传递比当前页的最后一个spittle的id小一位的id，也可以传递想要展示的spittle的数量。

为了实现分页，需要编写一个控制器满足：
- `before`参数，结果中的spittle的id都要在这个参数之前；
- `count`参数，结果中要包含的spittle的个数

下面我们对上面的`spittles()`方法进行小小的改动，让它可以使用before和count参数。首先对测试方法进行改动：

```java
    @Test
    public void shouldShowRecentSpittles() throws Exception {
        List<Spittle> expectedSpittles = createSpittleList(20);
        SpittleRepository mockRepository = Mockito.mock(SpittleRepository.class);
        Mockito.when(mockRepository.findSpittles(238900, 50)).thenReturn(expectedSpittles);

        SpittleController controller = new SpittleController(mockRepository);
        MockMvc mockMvc = MockMvcBuilders.standaloneSetup(controller)
                .setSingleView(new InternalResourceView("/WEB_INF/views/spittles.jsp")).build();

        // 调用MockMvc.perform(RequestBuilder requestBuilder)发起一个http请求，然后将得到ResultActions
        mockMvc.perform(MockMvcRequestBuilders.get("/spittles?max=238900&count=50"))// 添加验证断言来判断执行请求后的结果是否是预期的；
                .andExpect(MockMvcResultMatchers.view().name("spittles"))// view()：得到视图验证器；
                // 得到相应的***ResultMatchers后，接着再调用其相应的API得到ResultMatcher，
                // 如ModelResultMatchers.attributeExists(final String... names)判断Model属性是否存在。
                .andExpect(MockMvcResultMatchers.model().attributeExists("spittleList"))// model()：得到模型验证器；
                .andExpect(MockMvcResultMatchers.model().attribute("spittleList", IsCollectionContaining.hasItems(expectedSpittles.toArray())));
    }
```

这个测试方法的主要改动就是它发起的GET请求传递了两个参数：max和count。对`spittles()`进行修改：

```java
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
        @RequestParam(value="max", defaultValue=MAX_LONG_AS_STRING) long max, 
        @RequestParam(value="count", defaultValue="20") int count) {
    return spittleRepository.findSpittles(max, count);
}
```

这种情况下，如果没有max参数没有指定，那么就会使用默认的设置。由于查询参数是String类型的，因此`defaultValue`属性值也需要设置为String类型，需要对Long.MAX_VALUE进行设置：
`private static final String MAX_LONG_AS_STRING = "9223372036854775807";`

虽然，这里defaultValue的属性为String类型，当运行到函数时，将会根据函数的参数类型进行转换。

查询参数是请求中传送信息给控制器的最常用方式，另外一种流行的方式就是将参数作为请求路径的一部分。

## 通过路径参数传递数据：@PathVariable

假设现在应用需要展示单独的一篇Spittle，那么就需要一个id作为查询参数，对应的处理方法可以是：

```java
@RequestMapping(value="show", method=RequestMethod.GET)
public String showSpittle(
        @RequestParam("spittle_id") long spittleId,
        Model model
        ) {
    model.addAttribute(spittleRepository.findOne(spittleId));
    return "spittle";
}
```

这个handler方法将会处理形如`/spittles/show?spittle_id=12345`的请求，但是这并不符合资源导向的观点。理想情况下，应该使用URL路径对资源进行区分，而不是查询参数，即应该使用`/spittles/12345`这种形式。

为了实现资源导向的控制器，我们先在测试中获得这个需求（使用了静态引入）：

```java
@Test
public void testSpittle() throws Exception {
    Spittle expectedSpittle = new Spittle("Hello", new Date());
    SpittleRepository mockRepository = Mockito.mock(SpittleRepository.class);
    when(mockRepository.findOne(12345)).thenReturn(expectedSpittle);

    SpittleController controller = new SpittleController(mockRepository);
    MockMvc mockMvc = standaloneSetup(controller).build();

    mockMvc.perform(get("/spittles/12345"))
    .andExpect(view().name("spittle"))
    .andExpect(model().attributeExists("spittle"))
    .andExpect(model().attribute("spittle", expectedSpittle));
}
```

该测试中发起了一个`/spittles/12345`的GET请求，并且对其返回结果视图进行断言。

为了满足路径参数，Spring MVC允许在`@RequestMapping`路径中使用占位符（需要用大括号包围），下面是使用占位符来接受一个id作为路径的一部分：

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
        @PathVariable("spittleId") long spittleId,
        Model model
        ) {
    model.addAttribute(spittleRepository.findOne(spittleId));
    return "spittle";
}
```

spittle()方法的spittleId入参使用了`@PathVariable("spittleId")`注解，表明请求中占位符位置的值都会被传送到handler的spittleId参数。_@RequestMapping中value属性的占位符必须和@PathVariable包裹的参数一致_。如果@PathVariable中没有给定参数，那么将默认使用入参的册数参数名。即可以使用下面的方法：

```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
        @PathVariable long spittleId,
        Model model
        ) {
    model.addAttribute(spittleRepository.findOne(spittleId));
    return "spittle";
}
```

spittle()方法会将接收的参数值传递给spittleRepository的findOne()方法并查找到一个Spittle，并将其放置到model中，model的key值会是spittle，接下来就可以在视图中引用这个Spittle：

```html
<div class="spittleView">
    <div class="spittleMessage">
        <c:out value="${spittle.message}" />
    </div>
    <div>
        <span class="spittleTime"><c:out value="${spittle.time}" /></span>
    </div>
</div>
```

查询参数和路径参数可以处理一些少量的请求数据，但是当请求数据过大时，它们就不再适用，下面就来讲解一下如何处理表单数据。

# 处理表单

Web应用不仅仅是将内容推送给用户，它同时也会让用户填写表单并将数据提交给应用。

对于表单有两种处理方式：展示表单以及处理用户提交的表单数据。在Spittr中，需要提供一个供新用户进行注册的表单。

`SpitterController`：展示用户注册表单

```java
package spittr.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
@RequestMapping("/spitter")
public class SpitterController {

    // 处理来自/spitter/register的get请求
    @RequestMapping(value = "/register", method = RequestMethod.GET)
    public String showRegistrationForm() {
        return "registerForm";
    }

}
```

`showRegistrationForm`方法的`@RequestMapping`注解，以及类级别的注解`@RequestMapping`，表明了这个方法会处理来自/spitter/register的get请求，该方法仅仅返回了一个名为registerForm的逻辑视图。根据之前在`InternalResourceViewResolver`中的配置，这个逻辑视图会导向到`/WEB-INF/views/registerForm.jsp`该界面。

对应的测试方法：

```java
package spittr.web;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*;

import org.junit.Test;
import org.springframework.test.web.servlet.MockMvc;

public class SpitterControllerTest {

    @Test
    public void shouldShowRegistration() throws Exception {
        SpitterController controller = new SpitterController();
        MockMvc mockMvc = standaloneSetup(controller).build();
        mockMvc.perform(get("/spitter/register")).andExpect(view().name("registerForm"));
    }
}
```

也可以通过启动项目访问界面的方式验证：

```html
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
  <head>
    <title>Spitter</title>
    <link rel="stylesheet" type="text/css" href="<c:url value="/resources/style.css" />" >
  </head>
  <body>
    <h1>Register</h1>

    <form method="POST">
      First Name: <input type="text" name="firstName" /><br/>
      Last Name: <input type="text" name="lastName" /><br/>
      Username: <input type="text" name="username" /><br/>
      Password: <input type="password" name="password" /><br/>
      <input type="submit" value="Register" />
    </form>
  </body>
</html>
```

![该界面提供了用户注册的功能](http://hoxis-github-io.qiniudn.com/160215-spring-in-action-rigister.png)

接下来需要对提交的表单进行处理。

## 编写表单处理控制器

在处理POST请求时，控制器需要接受表单数据并且将这些数据存储为一个Spitter对象。为了避免重复提交，应该重定向到一个新的界面：用户信息页。在处理post请求时，一个聪明的做法就是在处理完成后发送一个重定向的请求，从而可以避免重复提交。

下面来实现控制器方法，从而可以处理注册请求。

```java
    private SpitterRepository spitterRepository;

    public SpitterController() {
    }

    // 注入SpitterRepository
    @Autowired
    public SpitterController(SpitterRepository spitterRepository) {
        this.spitterRepository = spitterRepository;
    }

    public String processRegistration(Spitter spitter) {
        // 保存Spitter
        spitterRepository.save(spitter);
        // 重定向到新的页面
        return "redirect:/spitter/" + spitter.getUsername();
    }
````
processRegistration方法使用Spitter对象作为入参，该对象的属性会从请求中填充。该方法中调用了spitterRepository的save方法对Spitter对象进行存储。最后返回了一个带有`redirect:`的字符串。

当InternalResourceViewResolver遇到`redirect:`时，它会自动地将其当做一个重定向请求，从而可以重定向到用户详情页面，如/spitter/xiaoming。

同时，InternalResourceViewResolver也可以识别前缀`forward:`，这种情况下，请求会被转向到给定的URL地址。

下面需要编写处理处理用户详情页面的方法：

<div class="se-preview-section-delimiter"></div>
```java
    @RequestMapping(value = "/{username}", method = RequestMethod.GET)
    public String showSpitterProfile(@PathVariable("username") String username, Model model) {
        Spitter spitter = spitterRepository.findByUsername(username);
        model.addAttribute(spitter);
        return "profile";
    }
```

## 参数校验

从Spring3.0开始，Spring支持Java校验api，从而可以从而可以不需要添加其他配置，仅仅需要有一个Java API 的实现，如Hibernate Validator。

Java Validation API定义了许多注解，可以使用这些注解来约束参数的值，所有的注解都在包`javax.validation.constraints`中。

| 注解 | 描述 |
| --- | --- |
| @AssertFalse（@AssertTrue） | 对象必须是布尔类型，并且必须为false（true） |
| @DecimalMax(value)、@DecimalMin(value) | 限制对象必须是一个数字，其值不大于（不小于）指定的BigDecimalString值 |
| @Digits(integer,fraction) | 对象必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction |
| @Future | 必须是一个将来的日期 |
| @Max(value)、@Min(value) | 必须为一个不大于（不小于）指定值的数字 |
| @NotNull | 限制对象不能为空 |
| @Null | 限制对象必须为空 |
| @Past | 必须是一个过去的日期 |
| @Pattern(value) | 必须符合指定的正则表达式 |
| @Size(min,max) | 限制字符长度必须在min到max之间 |

使用示例：

```java
public class Spitter {

    private Long id;

    @NotNull
    @Size(min = 5, max = 16)
    private String username;

    @NotNull
    @Size(min = 5, max = 25)
    private String password;

    @NotNull
    @Size(min = 2, max = 30)
    private String firstName;

    ...
```

既然已经对Spitter的参数添加了约束，那么就需要改动processRegistration方法来应用校验：

```java
    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public String processRegistration(@Valid Spitter spitter, Errors errors) {
        // 若校验中出现错误，那么就返回到注册界面
        if (errors.hasErrors()) {
            return "registerForm";
        }
        // 保存Spitter
        spitterRepository.save(spitter);
        // 重定向到新的页面
        return "redirect:/spitter/" + spitter.getUsername();
    }
```

# 总结

这一章比较适合Spring MVC的入门学习资料。主要涵盖了Spring MVC处理web请求的处理过程、如何写简单的控制器和控制器方法来处理Http请求、如何使用mockito框架测试控制器方法。

基于Spring MVC的应用有三种方式读取数据：查询参数、路径参数和表单输入。本章用两节介绍了这些内容，并给出了类似错误处理和参数验证等关键知识点。

由于缺少真正的入库操作，因此本章节的一些方法不能真正的运作。

在接下来的章节中，我们会对Spring视图进行深入了解，对如何在JSP页面中使用Spring标签库进行展开。

