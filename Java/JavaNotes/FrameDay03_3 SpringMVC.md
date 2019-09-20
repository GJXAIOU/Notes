# FrameDay03_3 SpringMVC


**重要知识点**
* 自定义拦截器
* 登录状态验证


## 一、自定义拦截器

**AOP拦截的是方法，SpringMVC 拦截器拦截的是请求**

- 跟过滤器比较像的技术；
- 发送**请求**时被拦截器拦截，拦截之后可以在控制器的前后添加额外功能；
-  跟 AOP 区分开
  - AOP 在特定方法前后进行扩充(主要针对 ServiceImpl 进行扩充)，其中特定方法指的是：只要这个方法可以被spring管理，就可以在这个方法前后进行扩充；
  - SpringMVC 拦截器：请求的拦截，针对点是控制器方法(主要针对 Controller)；

- SpringMVC 拦截器和Filter 的区别
  - 拦截器只能拦截 Controller；
  - Filter 可以拦截任何请求；
  
**实现自定义拦截器的步骤:**
【通过下面的拦截器，URL 中输入 demo2 无法访问，demo3 可以访问】
- 首先新建类并实现 HandlerInterceptor（使用 Ctrl + O 重写未实现方法）
```java
package com.gjxaiou.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/** 自定义类，同时实现 HandlerInterceptor，同时添加未实现的方法
 * @author GJXAIOU
 * @create 2019-09-20-19:34
 */
public class InterceptorDemo implements HandlerInterceptor {
    /**
     * 该方法在进入控制器之前执行
     * 所有的控制代码都写在这里，什么情况下可以访问路径，什么情况下不能访问路径；
     * @param request
     * @param response
     * @param handler
     * @return true 表示进入拦截器，false 表示阻止进入拦截器
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 参数 Object handler 表示拦截器拦截的方法的全称
        System.out.println("拦截的方法为：handeler = " + handler);
        System.out.println("preHandle");
        return true;
    }

    /**
     * 该方法在控制器执行完毕，进入到 JSP 之前执行
     * 作用：可以用于日志记录以及敏感词语过滤
     * @param request
     * @param response
     * @param handler
     * @param modelAndView
     * @throws Exception
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        // 参数 ModelAndView modelAndView 可以取到视图的值，以及取到或修改视图中参数名；
        System.out.println("往视图名：" + modelAndView.getViewName() + "跳转；");
        // 取出视图中参数值，因为 getModel() 返回值为 Map，因此使用 .get(key值)取到对应值；
        String modelString = modelAndView.getModel().get("model").toString();
        System.out.println(modelAndView.getModel().put("model", "修改后的值").toString());
        // 替换视图中参数值
        String modelStringReplace = modelString.replace("值", "替换的值");
        modelAndView.getModel().put("model", modelStringReplace);

        System.out.println("postHandle");
    }

    /**
     * 该方法在 JSP 执行完成之后执行
     * 作用：用于记录执行过程中出现的异常，并且可以将异常日志记录到日志中；
     * @param request
     * @param response
     * @param handler
     * @param ex
     * @throws Exception
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        // 参数 Exception ex 值为 NULL 表示没有异常，反之有异常；
        System.out.println("是否有异常：ex = " + ex + "   异常信息为：" + ex.getMessage());
        System.out.println("afterCompletion");
    }
}
```

-  然后在springmvc.xml 配置拦截器需要拦截哪些控制器
分为拦截所有控制器和拦截特定 URL 的控制器；
```xml
<!--扫描注解-->
    <context:component-scan base-package="com.gjxaiou.controller"></context:component-scan>

<!--注解驱动-->
    <mvc:annotation-driven></mvc:annotation-driven>

<!--第一种表示所有控制器全部拦截-->  
<mvc:interceptors>      
    <bean class="com.gjxaiou.interceptor.InterceptorDemo"></bean>   </mvc:interceptors>-->   
    
<!-- 第二种表示只拦截以下路径的控制器-->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/demo1"/>
        <mvc:mapping path="/demo2"/>
        <bean class="com.gjxaiou.interceptor.InterceptorDemo"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

- 控制器中实现的 DemoController 为：
```java
package com.gjxaiou.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

/**
 * @author GJXAIOU
 * @create 2019-09-20-19:50
 */
@Controller
public class ControllerDemo {
    @RequestMapping("demo1")
    public String demo1(){
        System.out.println("执行 demo1");
        return "index.jsp";
    }

    @RequestMapping("demo2")
    public ModelAndView demo2(){
        System.out.println("执行 demo2");
        ModelAndView modelAndView = new ModelAndView("/index.jsp").addObject("modelAndView",
                "modelAndView 的值");
        return modelAndView;
    }

    @RequestMapping("demo3")
    public String demo3(Model model){
        System.out.println("执行 demo3");
        model.addAttribute("model", "model 的值");
        return "index.jsp";
    }
}
```

- 在 index.jsp 中接收值
```jsp
<body>
  modelAndView:${modelAndView} <br>

  model:${model}
</body>
```


## 二、拦截器栈

- 多个拦截器同时生效时,组成了拦截器栈
- 执行顺序：整体标准为先进后出；
- 执行顺序和在 springmvc.xml 中配置顺序有关
- 设置先配置拦截器A 在配置拦截器B 执行顺序为
preHandle(A)  --> preHandle(B)  -->  控制器方法  -->  postHandle(B)-->  postHanle(A)  -->  JSP  -->  afterCompletion(B)  -->  afterCompletion(A)

**代码示例：**
同时在包 com.bjsxt.interceptor 包下面实现两个拦截器：interceptor1 和 interceptor2，两者的示例代码相同，但是 print 函数结果不同作为区分；
```java
package com.bjsxt.interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class DemoInterceptor1 implements HandlerInterceptor {
	@Override
	public boolean preHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2) throws Exception {
		System.out.println("preHandle1");
		return true;
	}

	@Override
	public void postHandle(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, ModelAndView arg3)
			throws Exception {
		System.out.println("postHandle1");
	}

	@Override
	public void afterCompletion(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2, Exception arg3)
			throws Exception {
		System.out.println("afterCompletion1");
	}
}

```

然后在 springmvc.xml 配置拦截器为：
```xml
<!-- 拦截器 -->
<mvc:interceptors>
	<!--拦截顺序和配置顺序有关-->
	<bean class="com.bjsxt.interceptor.DemoInterceptor1"></bean>
	<bean class="com.bjsxt.interceptor.DemoInterceptor2"></bean>
</mvc:interceptors>
```
假设 controller 中只有一个控制器：
```java
@Controller
public class DemoController {
	@RequestMapping("demo")
	public String demo( ){
		System.out.println("执行demo");
		return "index.jsp";
	}
}
```
当通过 URL 访问：`http://localhost:8080/springmvc07_war_exploded/demo`时候，结果为：
```java
preHandle1
preHandle2
执行demo
postHandle2
postHandle1
index.jsp
afterCompletion2
afterCompletion1
```


## 三、SpringMVC 运行原理

如果在  web.xml 中设置  DispatcherServlet 的<url-pattern>为/时，当用户发起请求， 请求一个控制器时， 首先会执行  DispatcherServlet。 由DispatcherServlet 调  用  HandlerMapping 的DefaultAnnotationHandlerMapping 解  析  URL， 解  析  后  调  用HandlerAdatper 组  件  的  AnnotationMethodHandlerAdapter 去  调  用Controller 中的  HandlerMethod。当  HandlerMethod 执行完成后会返回View，返回的view会被  ViewResovler 进行视图解析，解析后调用  jsp 对应的.class 文件并运行，最终把运行.class 文件的结果响应给客户端。


## 四、SpringMVC 对 Date 类型转换

- 只需要在springmvc.xml 中配置,代码中不需要做任何修改；
  * 必须额外导入 joda-time.jar
  * 时间类型 java.sql.Date
springmvc.xml 内容为：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:mvc="http://www.springframework.org/schema/mvc"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
	<!-- 扫描注解 -->
	<context:component-scan base-package="com.bjsxt.controller"></context:component-scan>
	<!-- 注解驱动 -->
	<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>


<bean id="conversionService"
		class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
		<property name="registerDefaultFormatters" value="false" />
		<property name="formatters">
			<set>
				<bean
					class="org.springframework.format.number.NumberFormatAnnotationFormatterFactory" />
			</set>
		</property>
		<property name="formatterRegistrars">
			<set>
				<bean
					class="org.springframework.format.datetime.joda.JodaTimeFormatterRegistrar">
					<property name="dateFormatter">
						<bean
							class="org.springframework.format.datetime.joda.DateTimeFormatterFactoryBean">
							<property name="pattern" value="yyyy-MM-dd" />
						</bean>
					</property>
				</bean>
			</set>
		</property>
	</bean>
</beans>
```


- 使用注解：在需要转换的参数或实体类属性上添加`@DateTimeFormatter(pattern=”表达式”)`
  - 使用Date 参数接收（在控制器中）
```java
@Controller
public class DemoController {
	@RequestMapping("demo")
    public String demo(@DateTimeFormat(pattern="yyyy-MM-dd") Date time){
        System.out.println(time);
        return "main.jsp";
    }
}
```

- 具体的实体类为：
```java
public class Demo1 {
	@DateTimeFormat(pattern="yyyy/MM/dd")
	private Date time;

	public Date getTime() {
		return time;
	}

	public void setTime(Date time) {
		this.time = time;
	}

	@Override
	public String toString() {
		return "Demo1 [time=" + time.toLocaleString() + "]";
	}
}
```

- 注意地方:
  * 不需要导入额外jar
  * Date 是java.util.Date