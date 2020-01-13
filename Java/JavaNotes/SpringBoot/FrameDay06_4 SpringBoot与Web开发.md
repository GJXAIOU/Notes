---

style: summer
flag: purple
date: '2019-11-24'
custom: GJXAIOU
---



# Web开发

[TOC]

## 一、简介

- 使用 SpringBoot 进行 WEB 开发步骤：
    - **创建 SpringBoot 应用，选中我们需要的模块；**
    - **SpringBoot 已经默认将这些场景配置好了，只需要在配置文件中指定少量配置就可以运行起来；**
    - **自己编写业务代码；**

- **自动配置原理**

    使用自动配置原理还应该考虑一下这个场景 SpringBoot 帮我们配置了什么？能不能修改？能修改哪些配置？能不能扩展？

    - xxxxAutoConfiguration：帮我们给容器中自动配置组件；
    - xxxxProperties：配置类来封装配置文件的内容；



## 二、Spring Boot 对静态资源的映射规则

在 SpringBoot 中关于 SpringMVC 的所有配置都在 `WebMvcAutoConfiguration.java` （使用 Ctrl + n 全局搜索该文件名即可）中，下面仅仅是静态资源配置的相关代码

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
    } else {
        Integer cachePeriod = this.resourceProperties.getCachePeriod();
        if (!registry.hasMappingForPattern("/webjars/**")) {
            this.customizeResourceHandlerRegistration(
                registry.addResourceHandler(new String[]{"/webjars/**"})
                .addResourceLocations(new String[]{"classpath:/META-INF/resources/webjars/"})
                .setCachePeriod(cachePeriod));
        }

        String staticPathPattern = this.mvcProperties.getStaticPathPattern();
        //静态资源文件夹映射
        if (!registry.hasMappingForPattern(staticPathPattern)) {
            this.customizeResourceHandlerRegistration(
                registry.addResourceHandler(new String[]{staticPathPattern})
                .addResourceLocations(this.resourceProperties.getStaticLocations())
                .setCachePeriod(cachePeriod));
                }
            }
        }
		

//配置欢迎页映射
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(
    ResourceProperties resourceProperties) {
    return new WelcomePageHandlerMapping(resourceProperties.getWelcomePage(),
                                         this.mvcProperties.getStaticPathPattern());
}

//配置喜欢的图标
@Configuration
@ConditionalOnProperty(
    value = {"spring.mvc.favicon.enabled"},
    matchIfMissing = true
)
public static class FaviconConfiguration {
    private final ResourceProperties resourceProperties;

    public FaviconConfiguration(ResourceProperties resourceProperties) {
        this.resourceProperties = resourceProperties;
    }

    @Bean
    public SimpleUrlHandlerMapping faviconHandlerMapping() {
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setOrder(-2147483647);
        mapping.setUrlMap(Collections.singletonMap("**/favicon.ico", this.faviconRequestHandler()));
        return mapping;
    }

    @Bean
    public ResourceHttpRequestHandler faviconRequestHandler() {
        ResourceHttpRequestHandler requestHandler = new ResourceHttpRequestHandler();
        requestHandler.setLocations(this.resourceProperties.getFaviconLocations());
        return requestHandler;
    }
}
```

上面代码中的设置配置在 `resourceProperties.java` 中，例如下面的部分文件设置（点击resourcesProperties 文件即可） ，可以设置其所有的属性值，例如： 可以设置和静态资源有关的参数，缓存时间等

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties implements ResourceLoaderAware {
    private static final String[] SERVLET_RESOURCE_LOCATIONS = { "/" };
	private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
			"classpath:/META-INF/resources/", "classpath:/resources/",
			"classpath:/static/", "classpath:/public/" };

	private static final String[] RESOURCE_LOCATIONS;

	static {
		RESOURCE_LOCATIONS = new String[CLASSPATH_RESOURCE_LOCATIONS.length
				+ SERVLET_RESOURCE_LOCATIONS.length];
		System.arraycopy(SERVLET_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS, 0,
				SERVLET_RESOURCE_LOCATIONS.length);
		System.arraycopy(CLASSPATH_RESOURCE_LOCATIONS, 0, RESOURCE_LOCATIONS,
				SERVLET_RESOURCE_LOCATIONS.length, CLASSPATH_RESOURCE_LOCATIONS.length);
	}

	private String[] staticLocations = RESOURCE_LOCATIONS;
	private Integer cachePeriod;
	private boolean addMappings = true;
	private final Chain chain = new Chain();
	private ResourceLoader resourceLoader;

```



### （一）映射规则一（WebJars）

从第一段代码中得出：==所有 `/webjars/**`的请求 ，都去 `classpath:/META-INF/resources/webjars/` 找资源；==

- webjars：**以 jar 包的方式引入静态资源**，以前是放在 webapp 目录下面即可；

- 需要使用什么，从[官方网站](http://www.webjars.org/)中引入响应的 jar 包即可，这里以 jQuery 为例，需要引入下面的jar包，下图为该jar 包中具体的内容；

```xml
<!--引入jquery-webjar-->在访问的时候只需要写 webjars 下面资源的名称即可
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

<img src="FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180203181751.png" style="zoom:80%;" />

测试：如果访问：localhost:8080/webjars/jquery/3.3.1/jquery.js 就能访问这里面的 js 了

### （二）映射规则二：（项目任意路径）

- 上述代码块第一个第13行中：mvcProperties 文件中的 staticPathPattern 值为：`/**`（源代码为  WebMvcProperties.java 中对应的代码：`private String staticPathPattern = "/**";`），这就是第二种映射规则；

- =="/**" 访问当前项目的任何资源，都去（静态资源的文件夹）找映射== 即如果该路径没有人进行处理就去下面文件夹中找（"/"：表示当前项目的根路径）

```
"classpath:/META-INF/resources/", 
"classpath:/resources/",
"classpath:/static/", 
"classpath:/public/" 
```

<img src="FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/image-20191122111352927.png" alt="image-20191122111352927" style="zoom: 80%;" />

例如访问：localhost:8080/static/asserts/css/signin.css  就是可以访问到资源的；

### （三）映射规则三：欢迎页

==欢迎页； 静态资源文件夹下的所有index.html页面；被"/**"映射；==

​	localhost:8080/   找index页面

### （四）映射规则四：网页标签的小图标

==所有的 **/favicon.ico  都是在静态资源文件下找；==

### （五）自定义映射规则

当然可以自己在 application.properties 中通过 `spring.resource.static-locations=classpath:/hello/`设置路径在 /hello/下面



## 三、模板引擎

因为 springboot 使用内嵌的 tomcat，不支持 jsp 等等，因此可以使用模板引擎。

现有的模板引擎包括：JSP、Velocity、Freemarker、Thymeleaf

<img src="FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/template-engine.png" style="zoom:80%;" />



==**SpringBoot 推荐的 Thymeleaf**==，因为其语法更简单，功能更强大；



### （一）引入thymeleaf

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!--切换 thymeleaf 版本-->
<properties>
    <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
    <thymeleaf-extras-java8time.version>3.0.4.RELEASE</thymeleaf-extras-java8time.version>
    <thymeleaf-extras-springsecurity.version>3.0.4.RELEASE</thymeleaf-extras-springsecurity.version>
      <!-- 下面是布局功能的支持程序，针对 thymeleaf3 主程序需要 layout2 以上版本 -->
    <thymeleaf-layout-dialect.version>2.4.1</thymeleaf-layout-dialect.version>
</properties>
```

### （二）Thymeleaf 使用

自动配置规则在 `spring-boot-autoconfigure-2.2.1.RELEASE.jar!\org\springframework\boot\autoconfigure\thymeleaf\ThymeleafAutoConfiguration.java `中，下面是其对应的 ThymeleafProperties.java 中封装着其默认规则，下面是部分代码

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {

	private static final Charset DEFAULT_ENCODING = Charset.forName("UTF-8");

	private static final MimeType DEFAULT_CONTENT_TYPE = MimeType.valueOf("text/html");
	// 默认的前缀和后缀
	public static final String DEFAULT_PREFIX = "classpath:/templates/";

	public static final String DEFAULT_SUFFIX = ".html";
```

上面的含义：只要我们把HTML页面放在 `classpath:/templates/`目录下，thymeleaf 就能自动渲染；

例如在 helloController.java 中输入：

```java
package com.gjxaiou.springboot.controller;

@Controller
public class HelloController {

    @RequestMapping("/success")
    public String success(){
        return "success";
    }
}
```

上面代码在 templates文件夹下面新建：success.html ，访问：`localhost:8080/success`就可以到达该页面；

- 在 HTML 中的使用
    -  导入thymeleaf的名称空间（下面两段代码都在在 sucess.html 中）	

        ```html
        <html lang="en" xmlns:th="http://www.thymeleaf.org">
        ```

    - 使用thymeleaf语法；

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>成功！</h1>
    <!--th:text 将div里面的文本内容设置为 -->
    <div th:text="${hello}">这是显示欢迎信息</div>
    <!--这里直接打开 html 文件显示内容为：这是显示欢迎信息，如果通过模板解析访问显示内容为：你好，会代替后面的值-->
</body>
</html>
```

对应的controller 修改为：

```java
package com.gjxaiou.springboot.controller;

@Controller
public class HelloController {

    //查出用户数据，在页面展示
    @RequestMapping("/success")
    public String success(Map<String,Object> map){
        map.put("hello","<h1>你好</h1>");
        return "success";
    }
}
```



### （三）语法规则

- `th：任意html属性`：来替换原生属性（即 HTML 中默认的属性）的值

    示例：th:text：改变当前元素里面的文本内容；

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/2018-02-04_123955.png)



- 支持的表达式语法：

```properties
Simple expressions:（表达式语法）
    Variable Expressions: ${...}：获取变量值；底层就是OGNL；
    		1）、可以获取对象的属性、调用方法
    		2）、使用内置的基本对象：（即在 ${}中可以加入下面这些）
    			#ctx : the context object.
    			#vars: the context variables.
                #locale : the context locale.
                #request : (only in Web Contexts) the HttpServletRequest object.
                #response : (only in Web Contexts) the HttpServletResponse object.
                #session : (only in Web Contexts) the HttpSession object.
                #servletContext : (only in Web Contexts) the ServletContext object.
				示例：${session.foo}
            3）、内置的一些工具对象：
#execInfo : information about the template being processed.
#messages : methods for obtaining externalized messages inside variables expressions, in the same way as they would be obtained using #{…} syntax.
#uris : methods for escaping parts of URLs/URIs
#conversions : methods for executing the configured conversion service (if any).
#dates : methods for java.util.Date objects: formatting, component extraction, etc.
#calendars : analogous to #dates , but for java.util.Calendar objects.
#numbers : methods for formatting numeric objects.
#strings : methods for String objects: contains, startsWith, prepending/appending, etc.
#objects : methods for objects in general.
#bools : methods for boolean evaluation.
#arrays : methods for arrays.
#lists : methods for lists.
#sets : methods for sets.
#maps : methods for maps.
#aggregates : methods for creating aggregates on arrays or collections.
#ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).

    Selection Variable Expressions: *{...}：选择表达式：和${}在功能上是一样；
    	补充功能：配合 th:object="${session.user}：
           <div th:object="${session.user}">
           		 <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
           		 <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
                  <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
            </div>
         对应于使用上面的 ${...}写法为：
          <div>
             <p>Name: <span th:text="${session.user.firstName}">Sebastian</span>.</p>
             <p>Surname: <span th:text="${session.user.lastName}">Pepper</span>.</p>
             <p>Nationality: <span th:text="${session.user.nationality}">Saturn</span>.</p>
          </div>
    
    Message Expressions: #{...}：获取国际化内容
    Link URL Expressions: @{...}：定义URL；
    Fragment Expressions: ~{...}：片段引用表达式

    		
Literals（字面量）
      Text literals: 'one text' , 'Another one!' ,…
      Number literals: 0 , 34 , 3.0 , 12.3 ,…
      Boolean literals: true , false
      Null literal: null
      Literal tokens: one , sometext , main ,…
Text operations:（文本操作）
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:（数学运算）
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:（布尔运算）
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:（比较运算）
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:条件运算（三元运算符）
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:
    No-Operation: _ 
```



使用示例：

```java
package com.gjxaiou.springboot.controller;

@Controller
public class HelloController {

    //查出用户数据，在页面展示
    @RequestMapping("/success")
    public String success(Map<String,Object> map){
        map.put("hello","<h1>你好</h1>");
        map.put("users",Arrays.asList("zhangsan","lisi","wangwu"));
        return "success";
    }

}

```

对应的 success.html 文件为：

```java
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
    
<body>
    <div th:text="${hello}"></div>
    <div th:utext="${hello}"></div>
    <hr/>

    <!-- th:each每次遍历都会生成当前这个标签： 最终形成3个h4 -->
    <h4 th:text="${user}"  th:each="user:${users}"></h4>
    <hr/>
        
        <!--每个结果放在 span 中，最终是 1 个<h4>里面 3个 span 标签-->
    <h4>
        <span th:each="user:${users}"> [[${user}]] </span>
    </h4>
         
</body>
</html>
```



## 四、SpringMVC 自动配置

[关于springMVC 的说明](https://docs.spring.io/spring-boot/docs/2.2.1.RELEASE/reference/htmlsingle/#boot-features-spring-mvc  )  

![image-20191224164615565](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/image-20191224164615565.png)

### （一）Spring MVC 的 auto-configuration

Spring Boot 自动配置好了 SpringMVC以下是SpringBoot 对 SpringMVC的默认配置（见 Reference 文档 P107/519）:**下面的所有自动配置都在（WebMvcAutoConfiguration）文件中**

- Inclusion（包含） of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.

  - 自动配置了ViewResolver（视图解析器：根据方法的返回值得到视图对象（View），视图对象决定如何渲染（转发？重定向？））

  - ContentNegotiatingViewResolver：这个类是组合所有的视图解析器的；

  - 如何定制视图解析器：==我们可以自己给容器中添加一个视图解析器；然后ContentNegotiatingViewResolver 自动的将其组合进来；==

    验证：在主类中自定义视图解析器，然后使用 @Bean 就将其加入容器中；

  ```java
  package com.gjxaiou.springboot;
  
  @SpringBootApplication
  public class SpringBoot04WebRestfulcrudApplication {
  
  	public static void main(String[] args) {
  		SpringApplication.run(SpringBoot04WebRestfulcrudApplication.class, args);
  	}
  
  	// 添加到容器中
  	@Bean
  	public ViewResolver myViewReolver(){
  		return new MyViewResolver();
  	}
  
  	public static class MyViewResolver implements ViewResolver{
  		@Override
  		public View resolveViewName(String viewName, Locale locale) throws Exception {
  			return null;
  		}
  	}
  }
  
  ```
  
  然后全局搜索：`DispatcherServlet`，来到 1000行（ctrl + G ） 的 doDispatch() 方法上打断点然后调试，访问任意路径看自定义的视图解析器是否在里面；
  
  查看 Valiables 中的 viewResolvers 中就应该有自己定义的视图解析器
  
  ![image-20191123214440623](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/image-20191123214440623.png)
  
- Support for serving static resources, including support for WebJars (see below).静态资源文件夹路径,webjars

- Static `index.html` support. 静态首页访问

- Custom `Favicon` support (see below).  支持 favicon.ico 自定义

- Automatic use of `Converter`, `GenericConverter`, `Formatter` beans；就是页面带来的数据格式要转换成其他格式，例如页面带来一个 18（文本类型）要转换为 integer 类型等等；

  - `Converter`：转换器；  public String hello(User user)：类型转换使用 Converter
  - `Formatter`  格式化器；  2017.12.17===Date；

```java
@Bean
@ConditionalOnProperty(prefix = "spring.mvc", name = "date-format")//在文件中配置日期格式化的规则
public Formatter<Date> dateFormatter() {
    return new DateFormatter(this.mvcProperties.getDateFormat());//日期格式化组件
}
```

​	==自己添加的格式化器转换器，我们只需要放在容器中即可==，因为它也是遍历 Converter 类型；

- Support for `HttpMessageConverters` (see below)。

  - HttpMessageConverter：是SpringMVC用来转换Http请求和响应的；如方法返回User对象，想以Json数据形式写出去；

  - `HttpMessageConverters` 是从容器中确定；获取所有的HttpMessageConverter；

    ==自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中（@Bean,@Component）==

- Automatic registration of `MessageCodesResolver` (see below).定义错误代码生成规则

- Automatic use of a `ConfigurableWebBindingInitializer` bean (see below).

  ==我们可以配置一个ConfigurableWebBindingInitializer来替换默认的；（需要添加到容器中）== ，

  

### （二）扩展SpringMVC

Spring MVC 中拓展方式： 

```xml
<mvc:view-controller path="/hello" view-name="success"/>
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

Spring Boot 中方式：**==编写一个配置类（@Configuration），该类是 WebMvcConfigurerAdapter 类型；但是不能标注 @EnableWebMvc==**;

既保留了所有的自动配置，也能用我们扩展的配置；

```java
// 使用 WebMvcConfigurerAdapter 可以来扩展 SpringMVC 的功能
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
       // super.addViewControllers(registry);
        //浏览器发送 /gjxaiou 请求来到 success
        registry.addViewController("/gjxaiou").setViewName("success");
    }
}
```

原理：

- WebMvcAutoConfiguration 是 Spring MVC 的自动配置类

- 在做其他自动配置时会导入：@Import(**EnableWebMvcConfiguration**.class)

```java
@Configuration
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration {
    private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();

    //从容器中获取所有的WebMvcConfigurer
    @Autowired(required = false)
    public void setConfigurers(List<WebMvcConfigurer> configurers) {
        if (!CollectionUtils.isEmpty(configurers)) {
            this.configurers.addWebMvcConfigurers(configurers);
            //一个参考实现；将所有的WebMvcConfigurer相关配置都来一起调用；  
            @Override
            // public void addViewControllers(ViewControllerRegistry registry) {
            //    for (WebMvcConfigurer delegate : this.delegates) {
            //       delegate.addViewControllers(registry);
            //   }
        }
    }
}
```

- 容器中所有的 WebMvcConfigurer 都会一起起作用；

- 我们的配置类也会被调用；

​	效果：SpringMVC 的自动配置和我们的扩展配置都会起作用；

### （三）全面接管SpringMVC（不推荐）

SpringBoot 对 SpringMVC 的自动配置都不需要，所有都是我们自己配置；例如基本的静态资源不能使用了，所有路径都需要自己进行配置；

**我们需要在配置类中添加@EnableWebMvc即可；**

```java
//使用WebMvcConfigurerAdapter可以来扩展SpringMVC的功能
@EnableWebMvc
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        // 浏览器发送 /gjxaiou 请求来到 success
        registry.addViewController("/gjxaiou").setViewName("success");
    }
}
```

原理：

为什么@EnableWebMvc自动配置就失效了；

- @EnableWebMvc 的代码为：（点击）

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

- 由上面的 @Import() 中的 DelegatingWebMvcConfiguration.class ，其代码为：

```java
@Configuration
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
```

- 下面是 Spring Boot 的自动配置类：WebMvcAutoConfiguration

```java
@Configuration
@ConditionalOnWebApplication
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class,
		WebMvcConfigurerAdapter.class })
//容器中没有这个组件的时候，这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class,
		ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {
```

- 总结：`@EnableWebMvc` 里面的 `@Import(DelegatingWebMvcConfiguration.class)` 将 里面类的父类：`WebMvcConfigurationSupport` 组件导入进来，所有相当于`@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)` 就是不符合的，因此自动配置类就失效了 ；

- 导入的WebMvcConfigurationSupport 只是 SpringMVC 最基本的功能；



## 五、如何修改 SpringBoot 的默认配置

- 方式一：SpringBoot 在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean、@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件可以有多个（ViewResolver）将用户配置的和自己默认的组合起来；

- 方式二：在 SpringBoot 中会有非常多的 xxxConfigurer 帮助我们进行扩展配置；

- 方式三：在 SpringBoot 中会有很多的 xxxCustomizer 帮助我们进行定制配置；



## 六、RestfulCRUD 项目

### （一）设置默认访问首页

项目中有两个 index.html 文件，位置分别为：`resources.public.index.html` 和`resources.templates.index.html`，要求默认是访问后者的 index.html 文件；

**下面的两种方式都是默认使用了 thymeleaf 模板引擎**；

- 方法一：编写单独的控制器进行路径映射

```java
@Controller
public class HelloController {
	// 不管是访问当前项目还是访问当前项目的 index.html  
	@RequestMapping({"/","/index.html"})
    	public String index(){
        	return "index";
   		}
}    
```

- 方法二：添加视图映射（只需要将组件注册到容器中即可）

```java
@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    //所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean //将组件注册在容器
    public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
        WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("index");
                registry.addViewController("/index.html").setViewName("index");
            }
        };
        return adapter;
    }
}

```

### （二）国际化

- 以前springmvc 配置方式：
    - 编写国际化配置文件；
    - 使用ResourceBundleMessageSource管理国际化资源文件；
    - 在页面使用fmt:message取出国际化内容；

- 使用 Spring Boot 实现步骤：
    - 首先编写国际化配置文件，抽取页面需要显示的国际化消息（然后分别配置属性文件）

    ![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180211130721.png)

    ![image-20191122184748011](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/image-20191122184748011.png)

    - SpringBoot自动配置好了管理国际化资源文件的组件；

    ```java
    @ConfigurationProperties(prefix = "spring.messages")
    public class MessageSourceAutoConfiguration {
         //我们的配置文件可以直接放在类路径下叫messages.properties；（因为默认名为 message）
    	private String basename = "messages";  
        
        @Bean
    	public MessageSource messageSource() {
    		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    		if (StringUtils.hasText(this.basename)) {
                //设置国际化资源文件的基础名（去掉语言国家代码的）
    			messageSource.setBasenames(StringUtils.commaDelimitedListToStringArray(
    					StringUtils.trimAllWhitespace(this.basename)));
    		}
    		if (this.encoding != null) {
    			messageSource.setDefaultEncoding(this.encoding.name());
    		}
    		messageSource.setFallbackToSystemLocale(this.fallbackToSystemLocale);
    		messageSource.setCacheSeconds(this.cacheSeconds);
    		messageSource.setAlwaysUseMessageFormat(this.alwaysUseMessageFormat);
    		return messageSource;
    	}
    ```

    因为没有使用系统默认的文件名，因此需要在 application.properties 中配置：`spring.messages.basename=i18n.login`，注意后面是去掉国家代码的部分（即取所有配置文件去掉后面的语言和国家的前面公共部分）；

    - 在页面获取国际化的值；

    在页面中（login.html）取值，例如 19行的：`th:text="#{login.username}"`（如果出现乱码，注意配置在 IDEA 中配置编码）

```html
<!DOCTYPE html>
<html lang="en"  xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
		<meta name="description" content="">
		<meta name="author" content="">
		<title>Signin Template for Bootstrap</title>
		<!-- Bootstrap core CSS -->
		<link href="asserts/css/bootstrap.min.css" th:href="@{/webjars/bootstrap/4.0.0/css/bootstrap.css}" rel="stylesheet">
		<!-- Custom styles for this template -->
		<link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
	</head>

	<body class="text-center">
		<form class="form-signin" action="dashboard.html">
			<img class="mb-4" th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
			<h1 class="h3 mb-3 font-weight-normal"  th:text="#{login.tip}" >Please sign in</h1>
			<label class="sr-only" th:text="#{login.username}">Username</label>
			<input type="text" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus="">
			<label class="sr-only" th:text="#{login.password}">Password</label>
			<input type="password" class="form-control" placeholder="Password" th:placeholder="#{login.password}" required="">
			<div class="checkbox mb-3">
				<label>
          		<input type="checkbox" value="remember-me"/> [[#{login.remember}]]
        </label>
			</div>
			<button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
			<p class="mt-5 mb-3 text-muted">© 2017-2018</p>
			<a class="btn btn-sm">中文</a>
			<a class="btn btn-sm">English</a>
		</form>

	</body>

</html>
```

效果：根据浏览器语言设置的信息切换了国际化；



- **国际化实现原理**：

​	国际化 Locale（区域信息对象）；springMVC 中的LocaleResolver（获取区域信息对象）；

springboot 默认也配置了区域信息解析器，如下：默认的就是根据请求头（浏览器 F12 中的Request Header 中的 Accept-Language中可以看出）带来的区域信息获取 Locale 进行国际化

```java
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

```

4）、点击链接切换国际化，需要自己写一个 LocalResolver ，以下面的为例：

```java
/**
 * 可以在连接上携带区域信息
 */
public class MyLocaleResolver implements LocaleResolver {
    
    @Override
    public Locale resolveLocale(HttpServletRequest request) {
        String l = request.getParameter("local");
        Locale locale = Locale.getDefault();
        if(!StringUtils.isEmpty(l)){
            String[] split = l.split("_");
            // split[0]是语言代码，后面那个是国家代码
            locale = new Locale(split[0],split[1]);
        }
        return locale;
    }

    @Override
    public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

    }
}


// 这里的方法名不能改变
 @Bean
    public LocaleResolver localeResolver(){
        return new MyLocaleResolver();
    }
}


```

对应的 html 也要更改：实现点击 中文显示中文，点击 English 显示英文

```html
		<button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
			<p class="mt-5 mb-3 text-muted">© 2017-2018</p>
			<a class="btn btn-sm" th:href="@{/index.html(local='zh_CN')}">中文</a>
			<a class="btn btn-sm" th:href="@{/index.html(local='en_US')}">English</a>
		</form>
```

### （三）登陆

首先修改 html 中的路径，下面是从 index.html 的15行开始；

```java
             
<body class="text-center">
        <!--当有一个请求为：/user/login，并且请求方式为：POST -->
		<form class="form-signin" action="dashboard.html" th:action="@{/user/login}" method="post">
			<img class="mb-4" th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" alt="" width="72" height="72">
			<h1 class="h3 mb-3 font-weight-normal" th:text="#{login.tip}">Please sign in</h1>
			<!--判断-->
			<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
			<label class="sr-only" th:text="#{login.username}">Username</label>
			<input type="text"  name="username" class="form-control" placeholder="Username" th:placeholder="#{login.username}" required="" autofocus="">
			<label class="sr-only" th:text="#{login.password}">Password</label>
			<input type="password" name="password" class="form-control" placeholder="Password" th:placeholder="#{login.password}" required="">
			<div class="checkbox mb-3">
				<label>
          			<input type="checkbox" value="remember-me"/> [[#{login.remember}]]
        		</label>
			</div>
			<button class="btn btn-lg btn-primary btn-block" type="submit" th:text="#{login.btn}">Sign in</button>
			<p class="mt-5 mb-3 text-muted">© 2017-2018</p>
			<a class="btn btn-sm" th:href="@{/index.html(l='zh_CN')}">中文</a>
			<a class="btn btn-sm" th:href="@{/index.html(l='en_US')}">English</a>
		</form>
	</body>                
```



开发期间模板引擎页面修改以后，要实时生效

- 禁用模板引擎的缓存（applicationl.properties），否则前端页面修改之后也没有变化

```
# 禁用缓存
spring.thymeleaf.cache=false 
```

- 页面修改完成以后ctrl+f9：重新编译；因为idea在运行期间不会重新编译页面



登陆错误消息的显示

```html
<p style="color: red" th:text="${msg}" th:if="${not #strings.isEmpty(msg)}"></p>
```

控制器代码：

```java
package com.gjxaiou.springboot.controller;

@Controller
public class LoginController {
    
    // 默认使用下面的方式，但是 springboot 中可以使用 @PostMapping 表示POST 请求，其本质上是里面声明了一个 @RequestMapping(method = {RequestMethod.POST})
    //@RequestMapping(value = "/user/login",method = RequestMethod.POST)
    @PostMapping(value = "/user/login")
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password,
                        Map<String,Object> map, HttpSession session){
        if(!StringUtils.isEmpty(username) && "123456".equals(password)){
            //登陆成功，防止表单重复提交，可以重定向到主页
            session.setAttribute("loginUser",username);
            return "redirect:/main.html";
        }else{
            //登陆失败
            map.put("msg","用户名密码错误");
            // 因为要去的页面在 templates 下面，因此有模板引擎的解析，因此前后缀不需要写
            return  "login";
        }
    }
}

```

为了能够解析出上面的 redirect:/main.html，需要在上面的自定义视图配置中添加上对应的语句；

```java
package com.gjxaiou.springboot.config;

@Configuration
public class MyMvcConfig extends WebMvcConfigurerAdapter {

    //所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean //将组件注册在容器
    public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
        WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/main.html").setViewName("dashboard");
            }

        } 

    @Bean
    public LocaleResolver localeResolver(){

        return new MyLocaleResolver();
    }

}

```



### （四）拦截器进行登陆检查

首先需要编写拦截器：

```java
package com.gjxaiou.springboot.component;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 登陆检查，
 */
public class LoginHandlerInterceptor implements HandlerInterceptor {
    //目标方法执行之前
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("loginUser");
        if(user == null){
            //未登陆，返回登陆页面
            request.setAttribute("msg","没有权限请先登陆");
            request.getRequestDispatcher("/index.html").forward(request,response);
            return false;
        }else{
            //已登陆，放行请求
            return true;
        }

    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {

    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {

    }
}

```



注册拦截器

```java
  //所有的WebMvcConfigurerAdapter组件都会一起起作用
    @Bean //将组件注册在容器
    public WebMvcConfigurerAdapter webMvcConfigurerAdapter(){
        WebMvcConfigurerAdapter adapter = new WebMvcConfigurerAdapter() {
            @Override
            public void addViewControllers(ViewControllerRegistry registry) {
                registry.addViewController("/").setViewName("login");
                registry.addViewController("/index.html").setViewName("login");
                registry.addViewController("/main.html").setViewName("dashboard");
            }

            //注册拦截器
            @Override
            public void addInterceptors(InterceptorRegistry registry) {
                //super.addInterceptors(registry);
                //静态资源；  *.css , *.js
                //SpringBoot已经做好了静态资源映射
                registry.addInterceptor(new LoginHandlerInterceptor()).addPathPatterns("/**")
                        .excludePathPatterns("/index.html","/","/user/login");
            }
        };
        return adapter;
    }
```

### （五）CRUD-员工列表

实验要求：

- RestfulCRUD：CRUD要满足Rest风格；

Rest风格为：URI：  /资源名称/资源标识      并且使用 HTTP请求方式区分对资源CRUD操作

|      | 普通CRUD（使用urL来区分操作） | RestfulCRUD       |
| ---- | ----------------------------- | ----------------- |
| 查询 | getEmp                        | emp---GET         |
| 添加 | addEmp?xxx                    | emp---POST        |
| 修改 | updateEmp?id=xxx&xxx=xx       | emp/{id}---PUT    |
| 删除 | deleteEmp?id=1                | emp/{id}---DELETE |

2）、实验的请求架构;

| 实验功能                             | 请求URI  | 请求方式 |
| ------------------------------------ | -------- | -------- |
| 查询所有员工                         | emps     | GET      |
| 查询某个员工(来到修改页面)           | emp/{id} | GET      |
| 来到添加页面                         | emp      | GET      |
| 添加员工                             | emp      | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/{id} | GET      |
| 修改员工                             | emp      | PUT      |
| 删除员工                             | emp/{id} | DELETE   |

功能一：员工列表：

首先 dashboard.html中部分代码修改如下：

```html
	<li class="nav-item">
						<a class="nav-link" href="#" th:href="@{/emps}">
							<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-users">
								<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path>
								<circle cx="9" cy="7" r="4"></circle>
								<path d="M23 21v-2a4 4 0 0 0-3-3.87"></path>
								<path d="M16 3.13a4 4 0 0 1 0 7.75"></path>
							</svg>
							Customers
						</a>
					</li>
```

然后对应 /emps 请求需要对应的处理，EmployeeController.java

通过访问：` http://localhost:8083/crud/main.html ` 点击左边的 Employee 就能跳转到员工列表面；

#### thymeleaf 公共页面元素抽取

例如主页面和员工列表页面的左边和上面都是一样的

```html
1、抽取公共片段
<div th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</div>

2、引入公共片段（重用）相当于有一个 footer 页面中引入 copy 片段；
<div th:insert="~{footer :: copy}"></div>
insert 之后表达式有两种：这里是第二种；
~{templatename::selector}：模板名::选择器
~{templatename::fragmentname}:模板名::片段名

3、默认效果：
insert的公共片段在div标签中
如果使用th:insert等属性进行引入，可以不用写~{}：
行内写法可以加上：[[~{}]];[(~{})]；
```

以顶部为例，使用 F12 看到该部分对应的代码：为 html  中的 <nav > 标签，然后在该标签后面添加：`th:fragment="topbar"`，具体名字自定

```html
<body>
		<nav class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-0" th:fragment="topbar">
			<a class="navbar-brand col-sm-3 col-md-2 mr-0" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Company name</a>
			<input class="form-control form-control-dark w-100" type="text" placeholder="Search" aria-label="Search">
			<ul class="navbar-nav px-3">
				<li class="nav-item text-nowrap">
					<a class="nav-link" href="http://getbootstrap.com/docs/4.0/examples/dashboard/#">Sign out</a>
				</li>
			</ul>
		</nav>
```

然后在 List.html 将上面这个一样的代码删除，只要引入抽取的 topbar 即可；

```html
<!--引入抽取的topbar-->
<!--模板名：会使用thymeleaf的前后缀配置规则进行解析，因此只需要写 dashboasrd 即可，会自动加上 /templs 和后缀 .html-->
<div th:replace="dashboard::topbar"></div>
```



- 三种方法引入公共片段的 th 属性：
    - **th:insert**：将公共片段整个插入到声明引入的元素中
    - **th:replace**：将声明引入的元素替换为公共片段
    - **th:include**：将被引入的片段的内容包含进这个标签中

下面代码中比较了三种方式的区别：

```html
<!--首先是抽取公共片段-->
<footer th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

<!--分别采用三种不同的方式比较-->
<div th:insert="footer :: copy"></div>
<div th:replace="footer :: copy"></div>
<div th:include="footer :: copy"></div>

<!--下面是对应的三种效果-->
<div>
    <footer>
    &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>

<footer>
&copy; 2011 The Good Thymes Virtual Grocery
</footer>

<div>
&copy; 2011 The Good Thymes Virtual Grocery
</div>
```



引入片段的时候传入参数： 这里使用的是选择器（从第一行中加上：`id="sidebar"`）

```html
<nav class="col-md-2 d-none d-md-block bg-light sidebar" id="sidebar">
    <div class="sidebar-sticky">
        <ul class="nav flex-column">
            <li class="nav-item">
                <a class="nav-link active"
                   th:class="${activeUri=='main.html'?'nav-link active':'nav-link'}"
                   href="#" th:href="@{/main.html}">
                    <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-home">
                        <path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"></path>
                        <polyline points="9 22 9 12 15 12 15 22"></polyline>
                    </svg>
                    Dashboard <span class="sr-only">(current)</span>
                </a>
            </li>

<!--引入侧边栏;同时传入参数-->
<div th:replace="commons/bar::#sidebar(activeUri='emps')"></div>
```

如果仅仅是在 list.html 中引入侧边栏没有参数的话，使用 `#id`

`<div th:replace="dashbar::#sidebar"></div>`

### 6）、CRUD-员工添加

添加页面

```html
<form>
    <div class="form-group">
        <label>LastName</label>
        <input type="text" class="form-control" placeholder="zhangsan">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input type="email" class="form-control" placeholder="zhangsan@gjxaiou.com">
    </div>
    <div class="form-group">
        <label>Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender"  value="1">
            <label class="form-check-label">男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender"  value="0">
            <label class="form-check-label">女</label>
        </div>
    </div>
    <div class="form-group">
        <label>department</label>
        <select class="form-control">
            <option>1</option>
            <option>2</option>
            <option>3</option>
            <option>4</option>
            <option>5</option>
        </select>
    </div>
    <div class="form-group">
        <label>Birth</label>
        <input type="text" class="form-control" placeholder="zhangsan">
    </div>
    <button type="submit" class="btn btn-primary">添加</button>
</form>
```

提交的数据格式不对：生日：日期；

2017-12-12；2017/12/12；2017.12.12；

日期的格式化；SpringMVC将页面提交的值需要转换为指定的类型;

2017-12-12---Date； 类型转换，格式化;

默认日期是按照/的方式；

### 7）、CRUD-员工修改

修改添加二合一表单

```html
<!--需要区分是员工修改还是添加；-->
<form th:action="@{/emp}" method="post">
    <!--发送put请求修改员工数据-->
    <!--
1、SpringMVC中配置HiddenHttpMethodFilter;（SpringBoot自动配置好的）
2、页面创建一个post表单
3、创建一个input项，name="_method";值就是我们指定的请求方式
-->
    <input type="hidden" name="_method" value="put" th:if="${emp!=null}"/>
    <input type="hidden" name="id" th:if="${emp!=null}" th:value="${emp.id}">
    <div class="form-group">
        <label>LastName</label>
        <input name="lastName" type="text" class="form-control" placeholder="zhangsan" th:value="${emp!=null}?${emp.lastName}">
    </div>
    <div class="form-group">
        <label>Email</label>
        <input name="email" type="email" class="form-control" placeholder="zhangsan@gjxaiou.com" th:value="${emp!=null}?${emp.email}">
    </div>
    <div class="form-group">
        <label>Gender</label><br/>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender" value="1" th:checked="${emp!=null}?${emp.gender==1}">
            <label class="form-check-label">男</label>
        </div>
        <div class="form-check form-check-inline">
            <input class="form-check-input" type="radio" name="gender" value="0" th:checked="${emp!=null}?${emp.gender==0}">
            <label class="form-check-label">女</label>
        </div>
    </div>
    <div class="form-group">
        <label>department</label>
        <!--提交的是部门的id-->
        <select class="form-control" name="department.id">
            <option th:selected="${emp!=null}?${dept.id == emp.department.id}" th:value="${dept.id}" th:each="dept:${depts}" th:text="${dept.departmentName}">1</option>
        </select>
    </div>
    <div class="form-group">
        <label>Birth</label>
        <input name="birth" type="text" class="form-control" placeholder="zhangsan" th:value="${emp!=null}?${#dates.format(emp.birth, 'yyyy-MM-dd HH:mm')}">
    </div>
    <button type="submit" class="btn btn-primary" th:text="${emp!=null}?'修改':'添加'">添加</button>
</form>
```

### 8）、CRUD-员工删除

```html
<tr th:each="emp:${emps}">
    <td th:text="${emp.id}"></td>
    <td>[[${emp.lastName}]]</td>
    <td th:text="${emp.email}"></td>
    <td th:text="${emp.gender}==0?'女':'男'"></td>
    <td th:text="${emp.department.departmentName}"></td>
    <td th:text="${#dates.format(emp.birth, 'yyyy-MM-dd HH:mm')}"></td>
    <td>
        <a class="btn btn-sm btn-primary" th:href="@{/emp/}+${emp.id}">编辑</a>
        <button th:attr="del_uri=@{/emp/}+${emp.id}" class="btn btn-sm btn-danger deleteBtn">删除</button>
    </td>
</tr>


<script>
    $(".deleteBtn").click(function(){
        //删除当前员工的
        $("#deleteEmpForm").attr("action",$(this).attr("del_uri")).submit();
        return false;
    });
</script>
```



## 错误处理机制

### （一）SpringBoot 默认的错误处理机制

Spring Boot 报错的默认效果：

- 浏览器：返回一个默认的错误页面

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180226173408.png)

  浏览器发送请求的请求头：

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180226180347.png)

- 如果是其他客户端：默认响应一个json数据

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180226173527.png)

​		![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180226180504.png)

能够报错的原理：

​	可以参照 `org.springframework.boot.autoconfigure.web.ErrorMvcAutoConfiguration`这是错误处理的自动配置；

  	该自动配置列给容器中添加了以下组件

​	1、DefaultErrorAttributes：（L97）帮我们在页面共享信息；

```java
@Override
	public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes,
			boolean includeStackTrace) {
		Map<String, Object> errorAttributes = new LinkedHashMap<String, Object>();
		errorAttributes.put("timestamp", new Date());
		addStatus(errorAttributes, requestAttributes);
		addErrorDetails(errorAttributes, requestAttributes, includeStackTrace);
		addPath(errorAttributes, requestAttributes);
		return errorAttributes;
	}
```



​	2、BasicErrorController：处理用来默认 `/error`路径请求（L103），下面是其代码

```java
@Controller
// 如果我们没有配置 server.error.path:，就默认处理 /error 的请求
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    
    
// BasicErrorController 是怎么处理这个请求呢？见这里的代码  
    // 它有两种处理 /error 路径的请求的方式，分别产生 HTML 类型数据和 json数据；（可以通过上面图片中的请求头中内容判断是否为浏览器）
    @RequestMapping(produces = "text/html")//产生html类型的数据；浏览器发送的请求来到这个方法处理
	public ModelAndView errorHtml(HttpServletRequest request,
			HttpServletResponse response) {
		HttpStatus status = getStatus(request);
		Map<String, Object> model = Collections.unmodifiableMap(getErrorAttributes(
				request, isIncludeStackTrace(request, MediaType.TEXT_HTML)));
		response.setStatus(status.value());
        
        //去哪个页面作为错误页面由这里得到，具体的 resolverErrorView 的代码见下面流程；其中 modelAndView 中包含页面地址和页面内容
		ModelAndView modelAndView = resolveErrorView(request, response, status, model);
		return (modelAndView == null ? new ModelAndView("error", model) : modelAndView);
	}

	@RequestMapping
	@ResponseBody    //产生json数据，其他客户端来到这个方法处理；
	public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
		Map<String, Object> body = getErrorAttributes(request,
				isIncludeStackTrace(request, MediaType.ALL));
		HttpStatus status = getStatus(request);
		return new ResponseEntity<Map<String, Object>>(body, status);
	}
```



​	3、ErrorPageCustomizer：（L110）系统出现错误以后来到 error 请求进行处理；（类似于之前使用 web.xml 注册的错误页面规则）

```java
// 点击 ErrorPageCustomizer之后在 L312 中的有个 getpath() 方法，该方法内容如下	
public class ErrorProperties {

	/**
	 * Path of the error controller.
	 */
	@Value("${error.path:/error}")
	private String path = "/error";

	public String getPath() {
		return this.path;
	} 
```



​	4、DefaultErrorViewResolver：（L134）

```java
@Override
	public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status,
			Map<String, Object> model) {
		ModelAndView modelAndView = resolve(String.valueOf(status), model);
		if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
			modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
		}
		return modelAndView;
	}

	private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //默认SpringBoot可以去找到一个页面？ 找到：error/404（不仅这一个状态码，这里这是一个示例）
		String errorViewName = "error/" + viewName;
        
        // 如果模板引擎可以解析这个页面地址就用模板引擎解析
		TemplateAvailabilityProvider provider = this.templateAvailabilityProviders
				.getProvider(errorViewName, this.applicationContext);
		if (provider != null) {
            // 模板引擎可用的情况下返回到 errorViewName 指定的视图地址
			return new ModelAndView(errorViewName, model);
		}
        // 模板引擎不可用，就在静态资源文件夹下找 errorViewName 对应的页面   error/404.html（状态码.html）
		return resolveResource(errorViewName, model);
	}
```



以上四个主要组件执行步骤：

​		一但系统出现4xx或者5xx之类的错误；ErrorPageCustomizer就会生效（用来定制错误的响应规则），就会来到 /error 请求；该请求就会被**BasicErrorController**处理，结果就是两种：响应页面或者响应json数据；

​		1）响应页面；去哪个页面作为错误页面是由**DefaultErrorViewResolver**解析得到的（见上面代码 BasicXXXX）；

```java
// 这里是解析页面的方法：resolveErrorView 的具体代码
protected ModelAndView resolveErrorView(HttpServletRequest request,
      HttpServletResponse response, HttpStatus status, Map<String, Object> model) {
    // 通过调用 ErrorViewResolver（异常视图解析器），通过拿到所有的异常视图解析器 得到ModelAndView，有就放回，没有就返回 null，这里的 ErrorViewResolver 就是上面注入的DefaultErrorViewResolver，所以具体解析过程见它
   for (ErrorViewResolver resolver : this.errorViewResolvers) {
      ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
      if (modelAndView != null) {
         return modelAndView;
      }
   }
   return null;
}
```

### （二）如果定制错误响应：

#### 	1.如何定制错误的页面（针对浏览器）

- 有模板引擎的情况下，视图地址为：error/状态码;** 【将错误页面命名为  `错误状态码.html` 放在模板引擎文件夹（templates）里面的 error文件夹下】，发生此状态码的错误就会来到  对应的页面；
    - 我们可以使用4xx和5xx作为错误页面的文件名来匹配这种类型的所有错误，精确优先（优先寻找精确的状态码.html，如果没有就是 4 开头的就都到 4XX.html）；		

    - 页面能获取的信息；（因为返回的是 modelAndView，其中能使用的是后面参数中的model ，而model 是通过getErrorAttributes 来获取（其又调用了 errorAttributes.getErrorAttributes() 方法）最终的实现类就是上面第一个代码：DefaultErrorAttributes），然后在跳转的页面中可以使用：`status值：[[${status}]]`等等取出；
        - timestamp：时间戳
        - status：状态码
        - error：错误提示
        - exception：异常对象
        - message：异常消息
        - errors：JSR303数据校验的错误都在这里

- 没有模板引擎（即模板引擎找不到这个对应的错误页面），还是在静态资源文件夹下找页面，就是上面的值不能获取了；

- 以上都没有（模板引擎和静态资源里面都没有）对应的错误页面，就是默认来到SpringBoot默认的错误提示页面；

**定制示例**：

自定义异常：

```java
package com.atguigu.springboot.exception;

public class UserNotExistException extends RuntimeException {

    public UserNotExistException() {
        super("用户不存在");
    }
}
```

对应的controller：访问路径：`localhost:8083/crud/hello?user=aaa`

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public  String hello(@RequestParam("user") String user){
        if(user.equals("aaa")){
            throw new UserNotExistException();
        }
        return "Hello World";
    }
}    
```



#### 	2.如何定制错误的json数据（针对其他客户端）

- 方式一：使用 @ExceptionHandler 捕获指定的异常，然后给出自定义异常处理&返回定制json数据（不返回默认的json数据）；缺点：没有自适应效果（浏览器和其他设备返回的都是纯 JSON 数据）

```java
@ControllerAdvice
// 异常处理类
public class MyExceptionHandler {
	// 因为返回值要将 Map 转换为 JSON 数据
    @ResponseBody
    // 指明要处理的异常（该异常同上），可以写 Exception.class 就会处理所有异常
    @ExceptionHandler(UserNotExistException.class)
    public Map<String,Object> handleException(Exception e){
        Map<String,Object> map = new HashMap<>();
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        return map;
    }
}

```

- 转发到 /error 请求使用 BasicErrorController 进行自适应响应效果处理

```java
 @ExceptionHandler(UserNotExistException.class)
    public String handleException(Exception e, HttpServletRequest request){
        Map<String,Object> map = new HashMap<>();
        // 传入我们自己的错误状态码例如4xx 5xx，因为默认是 2XX，这样就不会进入定制错误页面的解析流程
        /** 下面是获取状态码代码
         * Integer statusCode = (Integer) request
         .getAttribute("javax.servlet.error.status_code");
         */
        request.setAttribute("javax.servlet.error.status_code",500);
        map.put("code","user.notexist");
        map.put("message",e.getMessage());
        // 转发到 /error ，使用 BasicErrorController,因为它是有自适应的返回页面
        return "forward:/error";
    }
```

#### 	3.将我们的定制数据携带出去（带到页面上或者 JSON 中）

借助于 Spring Boot 的默认错误请求方式：出现错误以后，会来到 /error 请求，会被BasicErrorController处理，其做了自适应效果，返回了页面/ Json 数据，即返回 modelAndView (主要是后面的 model 参数)或者 Map 数据（主要是后面的 body 参数中内容）；但是两者（model 和 body 都是通过 getErrorAttributes()方法得到），因此响应出去可以获取的数据是由getErrorAttributes得到的（该方法是AbstractErrorController（ErrorController）规定的方法），其本质上是通过调用：`errorAttributes.getErrorAttributes方法得到数据`；

- 方式一：编写一个ErrorController的实现类【或者是编写AbstractErrorController的子类】，当然可能只需要重写返回浏览器、其他设备的自适应方法那两块代码即可，然后放在容器中；

- 方式二：因为页面上能用的数据或者是json返回能用的数据都是通过 errorAttributes.getErrorAttributes 得到；因为 DefaultErrorAttributes 是 errorAttributes 的实现类，而且通过 errorAttributes 上面注解可以看出，如果容器中没有 errorAttributes 就新建一个DefaultErrorAttributes，因此本质上是使用容器中DefaultErrorAttributes.getErrorAttributes()；默认进行数据处理的；

    

    实现：首先自定义 ErrorAttributes

    ```java
    // 给容器中加入我们自己定义的ErrorAttributes
    @Component
    public class MyErrorAttributes extends DefaultErrorAttributes {
    // 通过下面代码的 return "forward:/error"; 最终转发给错误处理器，而错误处理器是从该方法中获取 map，该 map 就是页面和 json 能获取到的所有字段，
            @Override
        public Map<String, Object> getErrorAttributes(RequestAttributes requestAttributes, boolean includeStackTrace) {
            Map<String, Object> map = super.getErrorAttributes(requestAttributes, includeStackTrace);
            map.put("company","atguigu");
    
            //我们的异常处理器携带的数据，getAttribute()中两个参数，第一个为下面放入request 作用域的 key 值，第二个表示是什么请求：其中request对应0（完整的看该方法代码即可）；最后放入 map 中
            Map<String,Object> ext = (Map<String, Object>) requestAttributes.getAttribute("ext", 0);
            map.put("ext",ext);
            return map;
        }
    }
    ```

    如果想将之前自己定制的异常处理类（MyExceptionHandler）中的参数也传递过来，只需要将整个 map 值放在 request 作用域中即可：

    ```java
    package com.atguigu.springboot.controller;
    
    @ControllerAdvice
    public class MyExceptionHandler {
    
        @ExceptionHandler(UserNotExistException.class)
        public String handleException(Exception e, HttpServletRequest request){
            Map<String,Object> map = new HashMap<>();
          
            request.setAttribute("javax.servlet.error.status_code",500);
            map.put("code","user.notexist");
            map.put("message","用户出错啦");
    		// 将整个 map 值放在 request 作用域中即可
            request.setAttribute("ext",map);
            return "forward:/error";
        }
    }
    ```

    

最终的效果：响应是自适应的，可以通过定制ErrorAttributes改变需要返回的内容，

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180228135513.png)



## 配置嵌入式Servlet容器

之前项目：写好 web 项目之后打成 war 包，然后放入 Tomcat（就是一个 Servlet 容器）；

SpringBoot 默认使用 Tomcat 作为嵌入式的 Servlet 容器：下面是 pom 文件中的依赖关系

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180301142915.png)



问题：外部的 Tomcat 可以修改 config 配置来修改，内置怎么修改；

### （一）如何定制和修改Servlet容器的相关配置；

- 方式一：修改和server有关的配置（点击下面的属性跳转到对应的配置类中，可以看到有个 ServerProperties.java 里面都是可以配置的属性【继承了EmbeddedServletContainerCustomizer】）；

```properties
server.port=8081
server.context-path=/crud

server.tomcat.uri-encoding=UTF-8

//通用的Servlet容器设置
server.xxx
//Tomcat的设置
server.tomcat.xxx
```

- 方式二：编写一个**EmbeddedServletContainerCustomizer**：嵌入式的Servlet容器的定制器；来修改Servlet容器的配置

```java
// 一定要将这个定制器加入到容器中
@Bean  
public EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
    return new EmbeddedServletContainerCustomizer() {

        // 定制嵌入式的Servlet容器相关的规则
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.setPort(8083);
        }
    };
}
```

### （二）注册 Servlet 三大组件【Servlet、Filter、Listener】

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件（以前的 Web 应用是通过 web.xml 注册组件）。

注册三大组件用以下方式

- ServletRegistrationBean ：注册 Servlet

    首先自己写一个 servlet

    ```java
    package com.atguigu.springboot.servlet;
    
    public class MyServlet extends HttpServlet {
    
        //处理get请求
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            doPost(req,resp);
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            resp.getWriter().write("Hello MyServlet");
        }
    }
    ```

    让自定义的 Servlet 起作用，即注册 Servlet

    ```java
    //注册三大组件
    @Configuration
    public class MyServerConfig {
        @Bean
        public ServletRegistrationBean myServlet(){
            // 参数：传入自己的 Servlet ，该Servlet 要映射的路径（即发送 /myServlet 请求就会到 MyServlet）
            ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet(),"/myServlet");
            return registrationBean;
        }
    }
    ```

    

    - FilterRegistrationBean

        首先自定义 Filter

        ```java
        package com.atguigu.springboot.filter;
        
        import javax.servlet.*;
        import java.io.IOException;
        
        public class MyFilter implements Filter {
            @Override
            public void init(FilterConfig filterConfig) throws ServletException {
            }
        
            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
                System.out.println("MyFilter process...");
               // 将该请求放行
                chain.doFilter(request,response);
            }
        
            @Override
            public void destroy() {
            }
        }
        
        ```

        然后将自定义的 Servlet 注册

        ```java
        @Configuration
        public class MyServerConfig {
            @Bean
            public FilterRegistrationBean myFilter(){
                FilterRegistrationBean registrationBean = new FilterRegistrationBean();
                registrationBean.setFilter(new MyFilter());
                // 该 Filter 拦截哪些请求，这里参数要求是 List，因此使用 Arrays.asList 进行转换
                registrationBean.setUrlPatterns(Arrays.asList("/hello","/myServlet"));
                return registrationBean;
            }
        }
        ```

    - ServletListenerRegistrationBean

        首先自定义自己的 Listener				

        ```java
        package com.atguigu.springboot.listener;
        
        import javax.servlet.ServletContextEvent;
        import javax.servlet.ServletContextListener;
        
        public class MyListener implements ServletContextListener {
            @Override
            public void contextInitialized(ServletContextEvent sce) {
                System.out.println("contextInitialized...web应用启动");
            }
        
            @Override
            public void contextDestroyed(ServletContextEvent sce) {
                System.out.println("contextDestroyed...当前web项目销毁");
            }
        }
        ```

        将 Listener 注册

        ```java
        @Configuration
        public class MyServerConfig {
            @Bean
            public ServletListenerRegistrationBean myListener(){
                ServletListenerRegistrationBean<MyListener> registrationBean = new ServletListenerRegistrationBean<>(new MyListener());
                return registrationBean;
            }
        }
        ```



SpringBoot帮我们自动SpringMVC的时候，自动的注册SpringMVC的前端控制器：DIspatcherServlet；

具体的配置见：DispatcherServletAutoConfiguration.java

```java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
public ServletRegistrationBean dispatcherServletRegistration(
      DispatcherServlet dispatcherServlet) {
   ServletRegistrationBean registration = new ServletRegistrationBean(
         dispatcherServlet, this.serverProperties.getServletMapping());
    //默认拦截：从serverProperties里面 getServletMapping() ServletPath 的值，就是 `/` ，即所有请求；包括静态资源，但是不拦截jsp请求；   /*会拦截jsp
    //可以通过server.servletPath来修改SpringMVC前端控制器默认拦截的请求路径
    
   registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
   registration.setLoadOnStartup(
         this.webMvcProperties.getServlet().getLoadOnStartup());
   if (this.multipartConfig != null) {
      registration.setMultipartConfig(this.multipartConfig);
   }
   return registration;
}

```

2）、SpringBoot能不能支持其他的Servlet容器；

### （三）替换为其他嵌入式Servlet容器

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180302114401.png)

默认支持下面三个：

- Tomcat（默认使用）

默认使用 Tomcat 是因为引入web的模块默认含有：spring-boot-starter-tomcat，因此就是使用嵌入式的Tomcat作为Servlet容器；

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   
</dependency>
```

- Jetty（适合长连接）

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-jetty</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

- Undertow（不支持 JSP）

```xml
<!-- 引入web模块 -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
   <exclusions>
      <exclusion>
         <artifactId>spring-boot-starter-tomcat</artifactId>
         <groupId>org.springframework.boot</groupId>
      </exclusion>
   </exclusions>
</dependency>

<!--引入其他的Servlet容器-->
<dependency>
   <artifactId>spring-boot-starter-undertow</artifactId>
   <groupId>org.springframework.boot</groupId>
</dependency>
```

### （四）嵌入式Servlet容器自动配置原理；

嵌入式的Servlet容器自动配置类见：`org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration`，下面是其部分代码：

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
@Configuration
@ConditionalOnWebApplication
//导入BeanPostProcessorsRegistrar；作用是给容器中导入一些组件
//利用 registerBeanDefinitions() 方法导入了EmbeddedServletContainerCustomizerBeanPostProcessor这个组件：具体分析见下面
//这是一个后置处理器：作用是在bean初始化前后（即创建完对象，还没赋值赋值）执行初始化工作
@Import(BeanPostProcessorsRegistrar.class)

public class EmbeddedServletContainerAutoConfiguration {
   
   
// -------到底怎么能实现自动切换的呢------------------------  
    // 通过下面三个方法上的 @ConditionalOnClass 来判断具体导入的是什么依赖，导入什么依赖就符合那个条件，就返回对应的嵌入式 Servlet 容器
    
    @Configuration
    //判断当前是否引入了Tomcat依赖；
	@ConditionalOnClass({ Servlet.class, Tomcat.class })
    //判断当前容器没有用户自己定义的嵌入式的Servlet容器工厂：EmbeddedServletContainerFactory；该工场作用：创建嵌入式的Servlet容器；具体解释见下面
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedTomcat { 
		@Bean
		public TomcatEmbeddedServletContainerFactory tomcatEmbeddedServletContainerFactory() {
			return new TomcatEmbeddedServletContainerFactory();
		}
	}   

    /**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Server.class, Loader.class,
			WebAppContext.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedJetty {

		@Bean
		public JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory() {
			return new JettyEmbeddedServletContainerFactory();
		}
	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Servlet.class, Undertow.class, SslClientAuthMode.class })
	@ConditionalOnMissingBean(value = EmbeddedServletContainerFactory.class, search = SearchStrategy.CURRENT)
	public static class EmbeddedUndertow {

		@Bean
		public UndertowEmbeddedServletContainerFactory undertowEmbeddedServletContainerFactory() {
			return new UndertowEmbeddedServletContainerFactory();
		}
	}
```

- EmbeddedServletContainerFactory（嵌入式Servlet容器工厂）【接上面的，具体解释如下：】

```java
public interface EmbeddedServletContainerFactory {

   // 获取嵌入式的Servlet容器
   EmbeddedServletContainer getEmbeddedServletContainer(
         ServletContextInitializer... initializers);

}
```

嵌入式的 Servlet 容器工厂有：

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180302144835.png)

- EmbeddedServletContainer：（嵌入式的Servlet容器）

    对应的三种嵌入式容器：

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180302144910.png)



- 以**TomcatEmbeddedServletContainerFactory**为例，具体说明如果导入对应的依赖，怎么实现自动切换的；下面为其部分代码

```java
@Override
public EmbeddedServletContainer getEmbeddedServletContainer(
      ServletContextInitializer... initializers) {
    //创建一个Tomcat
   Tomcat tomcat = new Tomcat();
    
    //配置Tomcat的基本环节
   File baseDir = (this.baseDirectory != null ? this.baseDirectory
         : createTempDir("tomcat"));
   tomcat.setBaseDir(baseDir.getAbsolutePath());
   Connector connector = new Connector(this.protocol);
   tomcat.getService().addConnector(connector);
   customizeConnector(connector);
   tomcat.setConnector(connector);
   tomcat.getHost().setAutoDeploy(false);
    // 配置引擎
   configureEngine(tomcat.getEngine());
   for (Connector additionalConnector : this.additionalTomcatConnectors) {
      tomcat.getService().addConnector(additionalConnector);
   }
   prepareContext(tomcat.getHost(), initializers);
    
    //将配置好的Tomcat传入进去，调用getTomcatEmbeddedServletContainer返回一个EmbeddedServletContainer；
    // 同时 getTomcatXXXX 判断只要端口 > 0 就自动启动Tomcat服务器；
   return getTomcatEmbeddedServletContainer(tomcat);
}
```

- 我们对嵌入式容器的配置修改是怎么生效？

    - 通过 application.properties 属性文件修改的方式是通过：ServerProperties.java 来生效的；

    - 通过自己设置一个 Servlet 容器定制器：EmbeddedServletContainerCustomizer





**EmbeddedServletContainerCustomizer**：该定制器帮我们修改了Servlet容器的配置（因为ServerProperties 也是实现了该定制器）即ServerProperties也是定制器

怎么修改的原理？

- 容器中导入了**EmbeddedServletContainerCustomizerBeanPostProcessor**这个组件（是上面定制器的对应的后置处理器）（由上上面的@import 导入），下面代码只有第一部分是该组件的，剩余的都是调用方法的具体内容代码；

```java
//初始化之前
@Override
public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
    //如果当前初始化的是一个ConfigurableEmbeddedServletContainer类型的组件
   if (bean instanceof ConfigurableEmbeddedServletContainer) {
       // 下面这个方法见下面
      postProcessBeforeInitialization((ConfigurableEmbeddedServletContainer) bean);
   }
   return bean;
}

// 这是上面的那个方法
private void postProcessBeforeInitialization(
			ConfigurableEmbeddedServletContainer bean) {
    //获取所有的定制器，调用每一个定制器的customize方法来给Servlet容器进行属性赋值；
    for (EmbeddedServletContainerCustomizer customizer : getCustomizers()) {
        customizer.customize(bean);
    }
}

// 下面是 getCustomizers 具体代码
private Collection<EmbeddedServletContainerCustomizer> getCustomizers() {
    if (this.customizers == null) {
        // Look up does not include the parent context
        this.customizers = new ArrayList<EmbeddedServletContainerCustomizer>(
            this.beanFactory
            //从容器中获取所有这个类型的组件：EmbeddedServletContainerCustomizer
            //因此定制Servlet容器，需要给容器中可以添加一个EmbeddedServletContainerCustomizer类型的组件
            .getBeansOfType(EmbeddedServletContainerCustomizer.class,
                            false, false)
            .values());
        Collections.sort(this.customizers, AnnotationAwareOrderComparator.INSTANCE);
        this.customizers = Collections.unmodifiableList(this.customizers);
    }
    return this.customizers;
}


```

上面总结步骤：

- SpringBoot根据导入的依赖情况，给容器中添加相应的嵌入式容器工厂XXXEmbeddedServletContainerFactory【比如TomcatEmbeddedServletContainerFactory】

- 容器中某个组件要创建对象（就是上面工厂创建对象）就会惊动后置处理器：EmbeddedServletContainerCustomizerBeanPostProcessor；后置处理器会判断是不是 ConfigurableEmbeddedServletContaioner（在 postProcessBeforeInitialization()方法中进行判断），而三大嵌入式 Servlet 容器工厂都是 ConfigurableEmbeddedServletContaioner（见下面图片）

    ![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180302144835.png)

    所以只要是嵌入式的Servlet容器工厂，后置处理器就工作；

- 后置处理器，是从容器中获取所有的 **EmbeddedServletContainerCustomizer**，调用定制器的定制方法（定制端口号啥的等等）；



### （五）嵌入式Servlet容器启动原理

因为从上面的步骤可以得出，一起都是从创建嵌入式 Servlet 容器工厂开始的，那什么时候创建嵌入式的Servlet容器工厂？和什么时候获取嵌入式的Servlet容器并启动Tomcat；

下面过程通过 debug 方式得到：

分别在：`EnbeddedServletContainerAutoConfiguration`的 `TomcatEmbeddedServletContainerFactory()`方法的 `return new tomcatEmbeddedServletContainerFactory`前面和 `TomcatEmbeddedServletContainerFactory`类的 `getEmbeddedServletContainer()`方法前面打断点；

获取嵌入式的Servlet容器工厂：

- 步骤一：SpringBoot应用启动运行run方法（在主配置类中的 run(）方法，下一步：run方法分为两步：首先创建一个 spring 应用，然后执行run 方法，然后调用run 的时候，在代码（SpringApplication.java）的 L303  使用了 refreshContext(context)刷新容器，而其中的参数 context是由 L299 的 createApplicationContext() 方法创建的，该方法会判断是否是 web 环境，如果是创建一个 web的 IOC 容器，反之创建一个默认的IOC 容器；

- 接上面的 refreshContext(context);SpringBoot刷新IOC容器【即创建IOC容器对象，并初始化容器，包括创建容器中的每一个组件】；如果是web应用创建**AnnotationConfigEmbeddedWebApplicationContext**容器，否则：**AnnotationConfigApplicationContext**

- 然后调用 refresh(context);**刷新刚才创建好的ioc容器；**本质上调用父类的refresh() 方法下面是该父类具体的刷新过程（AbstractApplicationContext.java）

```java
public void refresh() throws BeansException, IllegalStateException {
   synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      try {
         // Allows post-processing of the bean factory in context subclasses.
         postProcessBeanFactory(beanFactory);

         // Invoke factory processors registered as beans in the context.
         invokeBeanFactoryPostProcessors(beanFactory);

         // Register bean processors that intercept bean creation.
         registerBeanPostProcessors(beanFactory);

         // Initialize message source for this context.
         initMessageSource();

         // Initialize event multicaster for this context.
         initApplicationEventMulticaster();

         // Initialize other special beans in specific context subclasses.
         onRefresh();

         // Check for listener beans and register them.
         registerListeners();

         // Instantiate all remaining (non-lazy-init) singletons.
         finishBeanFactoryInitialization(beanFactory);

         // Last step: publish corresponding event.
         finishRefresh();
      }

      catch (BeansException ex) {
         if (logger.isWarnEnabled()) {
            logger.warn("Exception encountered during context initialization - " +
                  "cancelling refresh attempt: " + ex);
         }

         // Destroy already created singletons to avoid dangling resources.
         destroyBeans();

         // Reset 'active' flag.
         cancelRefresh(ex);

         // Propagate exception to caller.
         throw ex;
      }

      finally {
         // Reset common introspection caches in Spring's core, since we
         // might not ever need metadata for singleton beans anymore...
         resetCommonCaches();
      }
   }
}
```

- onRefresh(); web的ioc容器重写了onRefresh方法（在 EmbeddedWebApplication.java 中）

- web的ioc容器会创建嵌入式的Servlet容器（就在上面的 onRefresh() 方法中）；**createEmbeddedServletContainer**();
    - **创建的第一步：获取嵌入式的Servlet容器工厂：**

    `EmbeddedServletContainerFactory containerFactory = getEmbeddedServletContainerFactory();`getEmbeddedServletContainerFactory() 就是从ioc容器中获取EmbeddedServletContainerFactory 组件；	**TomcatEmbeddedServletContainerFactory**创建对象，后置处理器一看是这个对象，就获取所有的定制器来先定制Servlet容器的相关配置；
    
-  **使用上面的容器工厂获取嵌入式的Servlet容器**：`this.embeddedServletContainer = containerFactory.getEmbeddedServletContainer(getSelfInitializer());`该方法中就创建了 Tomcat 对象同时
   
- 同时（接上）嵌入式的Servlet容器创建对象并启动Servlet容器；

**先启动嵌入式的Servlet容器，再将ioc容器中剩下没有创建出的对象获取出来；**

**==IOC容器启动创建嵌入式的Servlet容器==**



## 九、使用外置的Servlet容器

- 嵌入式Servlet容器：应用打成可执行的jar
    - 优点：简单、便携；
    - 缺点：默认不支持JSP、优化定制Servlet 容器比较复杂（
        - 方法一：使用定制器【ServerProperties、自定义EmbeddedServletContainerCustomizer】，
        - 方法二：自己编写嵌入式Servlet容器的创建工厂【EmbeddedServletContainerFactory】）；

- 外置的Servlet容器：外面安装Tomcat---应用war包的方式打包；

### （一）步骤

==P51==

1）、必须创建一个war项目；（利用idea创建好目录结构）

2）、将嵌入式的Tomcat指定为provided；

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

3）、必须编写一个**SpringBootServletInitializer**的子类，并调用configure方法

```java
public class ServletInitializer extends SpringBootServletInitializer {

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
       //传入SpringBoot应用的主程序
      return application.sources(SpringBoot04WebJspApplication.class);
   }

}
```

4）、启动服务器就可以使用；

### （二）原理

- jar包：执行SpringBoot主类的main方法，启动ioc容器，创建嵌入式的Servlet容器；

- war包：首先启动服务器，**服务器启动SpringBoot应用**【SpringBootServletInitializer】，启动ioc容器；



在 servlet3.0 文档的 8.2.4章： Shared libraries / runtimes pluggability中定义了一些规则：

- 服务器启动（web应用启动）会创建当前web应用里面每一个jar包里面ServletContainerInitializer实例：

- ServletContainerInitializer的实现放在jar包的META-INF/services文件夹下，该文件夹下必须有一个名为javax.servlet.ServletContainerInitializer的文件，该文件内容就是ServletContainerInitializer的实现类的全类名

- 还可以使用@HandlesTypes，在应用启动的时候加载我们感兴趣的类；



### （三）流程：

- 启动Tomcat

-  在org\springframework\spring-web\4.3.14.RELEASE\spring-web-4.3.14.RELEASE.jar!\META-INF\services\javax.servlet.ServletContainerInitializer：Spring的web模块里面有这个文件：**org.springframework.web.SpringServletContainerInitializer**

- SpringServletContainerInitializer将@HandlesTypes(WebApplicationInitializer.class)标注的所有这个类型的类都传入到onStartup方法的Set<Class<?>>；为这些WebApplicationInitializer类型的类创建实例；

- 每一个WebApplicationInitializer都调用自己的onStartup；

![](FrameDay06_4%20SpringBoot%E4%B8%8EWeb%E5%BC%80%E5%8F%91.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180302221835.png)

- 相当于我们的SpringBootServletInitializer的类会被创建对象，并执行onStartup方法

- SpringBootServletInitializer实例执行onStartup的时候会createRootApplicationContext；创建容器

```java
protected WebApplicationContext createRootApplicationContext(
      ServletContext servletContext) {
    //1、创建SpringApplicationBuilder
   SpringApplicationBuilder builder = createSpringApplicationBuilder();
   StandardServletEnvironment environment = new StandardServletEnvironment();
   environment.initPropertySources(servletContext, null);
   builder.environment(environment);
   builder.main(getClass());
   ApplicationContext parent = getExistingRootWebApplicationContext(servletContext);
   if (parent != null) {
      this.logger.info("Root context already created (using as parent).");
      servletContext.setAttribute(
            WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, null);
      builder.initializers(new ParentContextApplicationContextInitializer(parent));
   }
   builder.initializers(
         new ServletContextApplicationContextInitializer(servletContext));
   builder.contextClass(AnnotationConfigEmbeddedWebApplicationContext.class);
    
    //调用configure方法，子类重写了这个方法，将SpringBoot的主程序类传入了进来
   builder = configure(builder);
    
    //使用builder创建一个Spring应用
   SpringApplication application = builder.build();
   if (application.getSources().isEmpty() && AnnotationUtils
         .findAnnotation(getClass(), Configuration.class) != null) {
      application.getSources().add(getClass());
   }
   Assert.state(!application.getSources().isEmpty(),
         "No SpringApplication sources have been defined. Either override the "
               + "configure method or add an @Configuration annotation");
   // Ensure error pages are registered
   if (this.registerErrorPageFilter) {
      application.getSources().add(ErrorPageFilterConfiguration.class);
   }
    //启动Spring应用
   return run(application);
}
```

- Spring的应用就启动并且创建IOC容器

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting();
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
      Banner printedBanner = printBanner(environment);
      context = createApplicationContext();
      analyzers = new FailureAnalyzers(context);
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
       
       //刷新IOC容器
      refreshContext(context);
      afterRefresh(context, applicationArguments);
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
```

**==启动Servlet容器，再启动SpringBoot应用==**

