# FrameDay03_2 SpringMVC 

**主要内容：**

* JSP 九大内置对象和四大作用域复习
* SpringMVC 作用域传值
* 文件下载
* 文件上传



## 一、 JSP 九大内置对象和四大作用域复习

### （一）九大内置对象
名称 | 类型 | 含义 | 获取方式
---|---|---|---
request |HttpSevletRequest |封装所有请求信息| 在方法参数
response|HttpServletResponse|封装所有响应信息|在方法参数
session |HttpSession |封装所有会话信息|req.getSession()
application| ServletContext |所有信息|getServletContext();和 request.getServletContext();
out |PrintWriter |输出对象 |response.getWriter()
exception |Exception |异常对象 |
page| Object |当前页面对象 |
pageContext | PageContext | 作用是获取其他对象 |
config |ServletConfig | 配置信息 | 


### （二）四大作用域

- page：在当前页面不会重新实例化；
- request：在一次请求中同一个对象，下次请求重新实例化一个request 对象；
- session：作用域在一次会话，只要客户端 Cookie 中传递的 Jsessionid 不变，Session 不会重新实例化(不超过默认时间)；
- application：只有在 tomcat 启动项目时才实例化，关闭 tomcat 时销毁application，所以在整个应用程序内都是单例的。

**实际有效时间:**
- 浏览器关闭则 Cookie 失效；
- 默认时间，在时间范围内无任何交互；可以在 tomcat 的web.xml 中配置
```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

## 二、SpringMVC 作用域传值的几种方式

### （一）使用原生 Servlet

- 首先在 HanlderMethod 参数中添加作用域对象（即在 Controller 中配置）
```java
@Controller
public class DemoController {
	@RequestMapping("demo10")
	public String demo10(HttpServletRequest abc, HttpSession sessionParam){
		// request 作用域
		abc.setAttribute("req", "req 的值");

		// session 作用域
		HttpSession session = abc.getSession();
		session.setAttribute("session", "session 的值");

		// 下面这句话是结合方法中的 HTTPSession 参数进行使用，两个加起来等效于上面两句
		sessionParam.setAttribute("sessionParam","sessionParam的值");

		// appliaction 作用域
		ServletContext application =  abc.getServletContext();
		application.setAttribute("application","application的值");
		return  "/index.jsp";
	}
}
```
执行之后的对应参数值在 index.jsp 中取出进行显示：
```jsp
<body>
    request:${requestScope.req} <br/>
    session:${sessionScope.sessio
    n} <br/>
    sessionParam:${sessionScope.sessionParam}<br/>
    application:${applicationScope.application}<br/>
</body>
```
通过访问：`http://localhost:8080/SpringMVC03_war_exploded/demo10` 得到结果为：
```java
request:req 的值 
session:session 的值 
sessionParam:sessionParam的值
application:application的值
```

### （二）使用 Map 集合

- 本质上是把 map 中内容放在 request 作用域中；
- spring 会对 map 集合通过 BindingAwareModelMap 类进行实例化；
```java
@Controller
public class DemoController {
	@RequestMapping("demo11")
    public String demo11(Map<String,Object> map){
        System.out.println(map.getClass());
        map.put("map","map 的值");
        return "/index.jsp";
    }
}
```
执行之后的对应参数值在 index.jsp 中取出进行显示：
```jsp
<body>
   map:${requestScope.map}
</body>
```
通过访问：`http://localhost:8080/SpringMVC03_war_exploded/demo10`得到结果为：`map:map 的值`


### （三）使用 SpringMVC 中 Model 接口 

该接口的本质把内容最终放入到 request 作用域中。
```java
@Controller
public class DemoController {
	@RequestMapping("demo12")
    public String demo12(Model model){
        model.addAttribute("model", "model 的值");
        return "/index.jsp";
    }
}
```
执行之后的对应参数值在 index.jsp 中取出进行显示：
```jsp
<body>
   <!--下面两种方式都是可以的-->
   model:${requestScope.model}
   model:${model}
</body>
```


### （四）使用 SpringMVC 中 ModelAndView 类

```java
@Controller
public class DemoController {
	@RequestMapping("demo13")
	public ModelAndView demo13(){
        //参数表示跳转视图
        ModelAndView mav = new ModelAndView("/index.jsp");
        mav.addObject("mav", "mav 的值");
        return mav;
    }
}
```
执行之后的对应参数值在 index.jsp 中取出进行显示：
```jsp
<body>
   mav:${requestScope.mav}
</body>
```

## 三、文件下载

**注意在 springmvc.xml 中配置放行 files 文件夹**
`<mvc:resources location="/files/" mapping="/files/**"></mvc:resources>`
- 访问资源时，响应头如果没有设置 Content-Disposition 的值，浏览器默认按照 inline 值进行处理；
  - inline 作用是能显示就显示,不能显示就下载；
- 只需要修改相应头中 `Context-Disposition= ”attachment;filename=文件名”`；
  - 其中 `attachment` 表示下载，以附件形式下载；
  - `filename=值` 中的值就是下载时显示的下载文件名；

**实现步骤**

- 导入 apache 的两个 jar
  - commons-fileupload.jar
  - commons-io.jar
- 在 jsp 中添加超链接，设置要下载文件，实现点击超链接就下载对应的文件；
  - 在 springmvc.xml 中放行静态资源 files 文件夹，文件夹中放置的是要下载的文件；
```xml
<a  href="download?fileName=a.rar">下载</a>
```

- 编写对应的控制器方法
```java
@Controller
public class DemoController {
	@RequestMapping("download")
	public void download(String fileName,HttpServletResponse res,HttpServletRequest req) throws IOException{
		// 设置响应流中文件进行下载
		res.setHeader("Content-Disposition","attachment;filename="+fileName);
		// 把二进制流放入到响应体中
		ServletOutputStream os = res.getOutputStream();
		String path = req.getServletContext().getRealPath("files");
		System.out.println(path);
		File file = new File(path, fileName);
		// 把这个文件直接读成字节数组
		byte[] bytes = FileUtils.readFileToByteArray(file);
		os.write(bytes);
		os.flush();
		os.close();
	}
}
```


## 四、文件上传

- 首先是基于 apache 的 commons-fileupload.jar 完成文件上传；
- 使用 MultipartResovler，是SpringMVC中负责文件上传的组件，
  - 作用: 把客户端上传的文件流转换成 MutipartFile 封装类；
  - 然后可以通过 MutipartFile 封装类获取到文件流；
- 表单数据类型分类：
在<form>的 enctype 属性是控制表单类型的；
  * 默认值为 `application/x-www-form-urlencoded`，表示普通表单数据(即少量文字信息)；
  * `text/plain` 表示大文字量时使用的类型.例如邮件,论文
  * `multipart/form-data` 表单中包含二进制文件内容；

**实现步骤:**
- 首先导入 springmvc 包和 apache 文件上传 commons-fileupload 和commons-io 两个 jar；

- 然后编写 JSP 页面
```jsp
<body>
<form action="upload" enctype="multipart/form-data" method="post">
	姓名:<input type="text" name="name"/><br/>
	文件:<input type="file" name="file"/><br/>
	<input type="submit" value="提交"/>
</form>
</body>
```

- 然后配置 springmvc.xml；主要是增加了 MultipartResovler 解析器和异常解析器
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
    <!-- 扫描注解 -->
	<context:component-scan base-package="com.gjxaiou.controller"></context:component-scan>
	<!-- 注解驱动 -->
	<!-- org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping -->
	<!-- org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter -->
	<mvc:annotation-driven></mvc:annotation-driven>
	<!-- 静态资源 -->
	<mvc:resources location="/js/" mapping="/js/**"></mvc:resources>
	<mvc:resources location="/css/" mapping="/css/**"></mvc:resources>
	<mvc:resources location="/images/" mapping="/images/**"></mvc:resources>
	<mvc:resources location="/files/" mapping="/files/**"></mvc:resources>


	<!-- MultipartResovler解析器 -->
	<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<property name="maxUploadSize" value="50"></property>
	</bean>
	<!-- 异常解析器 -->
	<bean id="exceptionResolver" class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
		<property name="exceptionMappings">
			<props>
				<prop key="org.springframework.web.multipart.MaxUploadSizeExceededException">/error.jsp</prop>
			</props>
		</property>
	</bean>
</beans>
```

- 最后编写控制器类
 MultipartFile 对象名必须和`<input type="file"/>` 的 name 属性值相同
```java
@Controller
public class DemoController {
	@RequestMapping("upload")
    public String upload(MultipartFile file,String name)throws IOException{
        String fileName = file.getOriginalFilename();
        String suffix = fileName.substring(fileName.lastIndexOf("."));
        //判断上传文件类型
        if(suffix.equalsIgnoreCase(".png")){
            String uuid = UUID.randomUUID().toString();
            FileUtils.copyInputStreamToFile(file.getInputStream(), new File("E:/"+uuid+suffix));
             return "/index.jsp";
            }else{
             return "error.jsp";
        }
    }
}
```


