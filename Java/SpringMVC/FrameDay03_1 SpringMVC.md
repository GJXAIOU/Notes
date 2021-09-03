# FrameDay03_1 SpringMVC

## 一、SpringMVC 简介

### （一）SpringMVC 中重要组件

- DispatcherServlet：前端控制器，**接收所有请求**(如果配置`/`就除了jsp 之外都拦截)；
- HandlerMapping：解析请求格式的，**判断希望要执行哪个具体的方法**；
- HandlerAdapter：负责**调用具体的方法**；
- ViewResovler：视图解析器，**负责解析结果，准备跳转到具体的物理视图**（即页面文件），即无需再写跳转语句了；

### （二）SpringMVC 运行原理图

<img src="FrameDay03_1%20SpringMVC.resource/SpringMVC%20%E8%BF%90%E8%A1%8C%E5%8E%9F%E7%90%86%E5%9B%BE.png" alt="SpringMVC 运行原理图" style="zoom: 67%;" />

### （三）Spring 容器和 SpringMVC 容器的关系

- 代码
![Spring与SpringMVC](FrameDay03_1%20SpringMVC.resource/Spring%E4%B8%8ESpringMVC.jpg)

- Spring 容器和 SpringMVC 容器是父子容器。

  **SpringMVC 容器中能够调用 Spring 容器的所有内容。**
  ![关系](FrameDay03_1%20SpringMVC.resource/%E5%85%B3%E7%B3%BB.png)


## 二、SpringMVC 环境搭建【使用注解方式】

### （一）导入 jar

```jar
// 日志包
commons-logging.jar

// spring
spring-aop.jar
spring-aspects.jar
spring-beans.jar
spring-context.jar
spring-core.jar
spring-expression.jar
spring-jdbc.jar
spring-tx.jar

// web 模块
spring-web.jar
spring-wermvc
```

### （二）配置 web.xml

- 首先在 web.xml 中配置前端控制器 DispatcherServlet
```xml
<servlet>
	<servlet-name>springmvc123</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<!-- 修改 SpringMVC 配置文件路径和名称，value 值是指在 src 目录下 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springmvc.xml</param-value>
	</init-param>
	<!-- 设置自启动，不设置的话只有 Tomcat 访问的时候才会加载这个类 -->
	<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>springmvc123</servlet-name>
	<!--下面含义：拦截除了 Jsp 之外所有的请求-->
	<url-pattern>/</url-pattern>
</servlet-mapping>
```
如果不配置 `<init-param>` 默认会在`/WEB-INF/<servlet-name>-servlet.xml`找默认位置的配置文件名，这里使用 `<init-param>` 指定 SpringMVC 配置文件为：src 目录下的 springmvc.xml。

### （三）在 src 下新建 springmvc.xml

这里需要新引入 xmlns:mvc 命名空间
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!-- 步骤一：扫描注解，必须在 SpringMVC 的配置文件中配置，不能在 web.xml 中配置 -->
	<context:component-scan base-package="com.gjxaiou.controller"></context:component-scan>
	<!-- 注解驱动 -->
	<!-- org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping -->
	<!-- org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter -->
	<!--下面配置相当于配置了上面两个-->
	<mvc:annotation-driven></mvc:annotation-driven>
	<!-- 配置不要拦截静态资源（js/css/html/图片文件），location 为路径， mapping 只是配置匹配规则-->
    <!-- /js/*： 表示 JS文件下所有文件，/js/**：表示js文件夹下面所有文件及其子文件-->
	<mvc:resources location="/js/" mapping="/js/**"></mvc:resources>
	<mvc:resources location="/css/" mapping="/css/**"></mvc:resources>
	<mvc:resources location="/images/" mapping="/images/**"></mvc:resources>
</beans>
```

### （四）编写控制器类

```java
package com.gjxaiou.controller;

import com.gjxaiou.pojo.Demo;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import java.util.List;

// 通过 Controller注解，将该类交给容器去管理
@Controller
public class DemoController {
	@RequestMapping("demo")
	public String demo(){
		System.out.println("执行demo");
		return "main.jsp";
	}
    
	@RequestMapping("demo2")
	public String demo2(){
		System.out.println("demo2");
		return "main1.jsp";
	}
}	
```

## 三、字符编码过滤器

**用于解决在下面传参时候，传入的中文参数显示为乱码**
只要在 web.xml 中配置 Filter
```xml
<!--配置一个字符编码过滤器-->
<filter>
	<filter-name>MyEncoding</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	<init-param>
		<!--下面的 name 是固定的，需要对这个参数进行赋值-->
		<param-name>encoding</param-name>
		<param-value>utf-8</param-value>
	</init-param>
</filter>

<filter-mapping>
	<filter-name>MyEncoding</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

## 四、传参

大原则：只要把内容写到方法参数中，SpringMVC 只要有这个对应内容，就会注入内容；

### （一）传输的是基本数据类型参数
#### 1. 默认保证参数名称和请求中传递的参数名相同

- 请求中传入的参数：在 index.jsp 配置内容为：下面传入的参数名为：name、age
```jsp
<body>
<form action="demo3" method="post">
    <input type="text" name = "name"/>
    <input type="text" name = "age"/>
    <input type="submit" name = "提交">
</form>
</body>
```
- 对应的 Controller 中的内容为：参数名称为：name、age
```java
@Controller
public class DemoController {
  	@RequestMapping("demo3")
	public String demo3(String name, int age){
		System.out.println("demo2 " + name + " "+age);
		return "main.jsp";
	}
}	
```
然后使用 Tomcat 在页面中输入相应数据之后，页面跳转到 main.jsp，同时在控制窗口输出：`demo3 GJXAIOU 123`

#### 2.如果请求参数名和方法参数名不对应，则使用@RequestParam()赋值

如果 index.jsp 内容为：传入的参数为：name1、age1，但是对应的 Controller 中方法参数名称为：name、age

```java
@Controller
public class DemoController {
  	@RequestMapping("demo3")
    public String demo3(@RequestParam(value="name1")String name, @RequestParam(value="age1") int age){
        System.out.println("执行demo3"+" "+name+" "+age);
        return "main.jsp";
    }
}	
```

如果以上**参数中的基本数据类型**（不是对应的封装类，因为封装类不会出现这种情况），例如 int，如果在页面中没有输入参数直接提交之后会报错，为了防止出现这种情况；

#### 3.通过 @RequestParam 设置默认值：defaultValue = “XXX”

假设 index.jsp 中传入的参数为：name、age，对应的 Controller 中方法参数名称为：name、age

```java
@Controller
public class DemoController {
  	@RequestMapping("demo4")
    public String demo4(@RequestParam(defaultValue="1")int name, @RequestParam(defaultValue="2") int age){
        System.out.println("执行demo4"+" "+name+" "+age);
        return "main.jsp";
    }
}	
```

#### 4.设置强制要求必须有某个参数：required = true

对应的 Controller 中的内容为：下面要求 name 属性必须进行赋值；

```java
@Controller
public class DemoController {
  	@RequestMapping("demo5")
    public String demo5(@RequestParam(required = true)String name, int age){
        System.out.println("执行demo5"+" "+name+" "+age);
        return "main.jsp";
    }
}	
```

这里可以使用 Index.jsp 中使用输入框进行赋值，也可以在 URL 输入框中直接追加：`demo5?name=gjx&age=23`，中间不能有空格，完整路径为：`http://localhost:8080/springmvc02_war_exploded/demo3?name=GJX&age=3`，同样上面的所有都可以使用这种方式进行赋值；

#### **5.请求的参数中包含多个同名参数的获取方式**

- 这里的 index.jsp 以复选框为例，所有的 name 值均为：hover
```jsp
<!--对应于复选框-->
<form action="demo6" method="post">
    <input type="text" name = "name"/>
    <input type="text" name = "age"/>
    <input type="checkbox" name = "hover" value="学习"> 学习
    <input type="checkbox" name = "hover" value="笔记"> 笔记
    <input type="checkbox" name = "hover" value="视频"> 视频
    <input type="submit" name = "提交">
</form>
```
- 对应的 Controller 中获取方式
```java
@Controller
public class DemoController {
  	@RequestMapping("demo6")
  // @RequestParam("hover") List<String> lis 表示将所有的 hover 都放在 list中
    public String demo6(String name, int age, @RequestParam("hover") List<String> list){
   System.out.println(name + "  " + age + " " + list);
 return "main.jsp"; }
}	
```
示例结果为：`GJXAIOU  23 [学习, 哈哈, 视频]`

###  （二）HandlerMethod 中参数是对象类型

**只需要请求参数名和对象中属性名对应(同时有 get/set 方法)**
- 对应的 Controller 中内容为：
```java
@Controller
public class DemoController {
  @RequestMapping("demo7")
  public String demo7(Demo demo){
     System.out.println("demo7" + " " + demo);
     return "main.jsp"; 
  }
}	
```
前提条件是实现 People 实体类，以及对应的 get/set 、构造方法、toString 方法==一定要有无参构造方法==：
```java
public class People {
    private String name;
    private int age;
    // 省略 get、set方法以及构造方法、toString方法
}
```
对应的 demo 类的代码为：**类对象名和下面 jsp 文件中参数的 点 的前面的名称对应**
```java
public class Demo {
     private People peo;

    @Override
    public String toString() {
        // 这里传参的时候 peo 值为 null，因此这里不能写 peo.getName等等，否则报NullPointerException
        return "Demo{" + "peo=" + peo  + '}';
    }
}
```
- 对应的 index.jsp 代码为：
```jsp
<form action="demo7" method="post"> demo7，测试传入为对象
    <input type="text" name = "peo.name"/>
    <input type="text" name = "peo.age"/>
    <input type="submit" name = "提交">
</form>
```
控制窗口输出为：`demo7 Demo{peo=null}`

**在请求参数中传递集合对象类型参数**
- 对应的 index.jsp 中代码为：
```java
<form action="demo8" method="post"> demo8，测试传入为集合对象
    <input type="text" name = "peoList[0].name"/>
    <input type="text" name = "peoList[0].age"/>
    <input type="text" name = "peoList[1].name"/>
    <input type="text" name = "peoList[1].age"/>
    <input type="submit" name = "提交">
</form>
```
具体里面传递的类 People 没有改变。
同时新建类：Demo2.java
```java
public class Demo2 {
    private List<People> peopleList;

    @Override
    public String toString() {
        return "Demo2{" +
                "peopleList=" + peopleList +
                '}';
    }
}
```
- 控制器 Controller 中代码
```java
@Controller
public class DemoController {
  @RequestMapping("demo8")
  public String demo8(Demo2 demo2){
      System.out.println("demo8" + " " + demo2);
      return "main.jsp"; }
}
```




###  （三）restful 传值方式，正常使用在超链接中

用于简化 jsp 中参数编写格式

- 首先在 jsp 中设定特定的格式：
`<a href="demo9/123/abc">跳转</a>`

- 然后在控制器中设置如下：
  - 在@RequestMapping 中一定要和**请求格式**对应
  - `{名称}` 中名称是自定义名称
  - @PathVariable  获取@RequestMapping 中内容，默认按照方法参数名称去寻找。
  控制器 Controller 中代码
```java
@Controller
public class DemoController {
  	@RequestMapping("demo9/{id}/{name}")
	public String demo9(@PathVariable String name, @PathVariable("id") int age){
		System.out.println(name + "  " + age);
		return "main.jsp";
	}
}
```
根据上面设置，需要在 WebContent 目录下新建 demo9/123/main.jsp，才能正确的跳转。

## 四、跳转方式

默认跳转方式请求转发。

设置返回值字符串内容

- 添加 redirect:资源路径 重定向
- 添加 forward:资源路径  或省略 forward: 转发

## 五、视图解析器

SpringMVC 会提供默认视图解析器，但是我们也可以自定义视图解析器。

```xml
<bean id="viewResolver"
    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```

如果希望不执行自定义视图解析器, 在方法返回值前面添加 forward: 或 redirect:

```java
@RequestMapping("demo10")
public String demo10(){
    return "forward:demo11";
}    

@RequestMapping("demo11")
pulic String demo11(){
	System.out.println("demo11");
    return "main";
}
```

## 六、@ResponseBody

- 在方法上只有@RequestMapping 时,无论方法返回值是什么都认为需要跳转
- 在方法上添加@ResponseBody(恒不跳转)
  -  如果返回值满足 key-value 形式(即返回值为对象或 map)
     - 会把响应头设置为 application/json;charset=utf-8
     -  把转换后的内容以输出流的形式响应给客户端
  - 如果返回值不满足 key-value,例如返回值为 String
     - 把相应头设置为 text/html
     - 把方法返回值以流的形式直接输出
     - 如果返回值包含中文,出现中文乱码

        produces 表示设置响应头中 Content-Type 的值
```java
// @ResponseBody // 将返回值转换为 json字符串 ，同时设置响应头类型为Application/json，同时不再跳转，表示该内容转化为json字符串之后以流的形式输出

@RequestMapping(value="demo12",produces="text/html;charset=utf-8")
public String demo12() throws IOException{
    People p = new People();
    p.setAge(12);
    p.setName("张三");
    return "中文";
}
```

底层使用 Jackson 进行 json 转换，因此需要在项目中一定要导入 jackson 的 jar。



