# JavaEEDay45 过滤器

@toc

##  一、复习
### （一）WEB 基础：
  - HTML + CSS + js : 做一个前端页面展示
  - XML：做一个配置文件
  - MySQL：数据库
  - Servlet：运行在服务器的 Java 小程序，用于处理客户端请求，产生相应；
  - Tomcat：所有想要被客户端访问的资源都需要部署到 Tomcat 下：包括：html/js/css/servlet/jsp/图片；

### （二）Servlet生命周期

1.什么时候初始化：
在客户端第一次访问某一个Servlet的时候，此Servlet会初始化，并且调用init方法； 
注：init在一个生命周期只会调用一次，每次访问该Servlet重复执行的是service方法。

2.什么时候销毁：
直到服务器(tomcat)关闭的时候Servlet才会销毁。


## 二、过滤器
本质上是一个接口，用于拦截客户端的请求，对请求进行过滤，还可以对响应进行处理；
- 过滤器的用途：拦截、验证、统一管理
  - 用户认证权限管理
  - 统计访问量和访问命中率，形成访问报告
  - 请求编码转换
  - 数据加密
  - 日志记录和审核过滤器
  - 拦截敏感词汇

### （一）前言引导
场景设置：部分网页需要登录之后才可以访问；
思路：
1.用户填写用户名、密码进行判断，根据判断结果进行页面跳转；
2.用户登录成功之后，再 session 中保存一个变量，要表示用户是否已经登录；
3.在登录成功之后才能够访问的资源中判断 session 中变量；

代码：
index.jsp  :页面主页
web.xml   :配置文件
login.jsp   :登录页面
index2.jsp :备用主页
loginServlet.java 程序

index.jsp
```jsp
<body>
<%--  在jsp中判断session中是否存在islogin，并且还为true--%>

  <%
    // 因为后面返回值为Object类型，因此强转应该是Boolean类型，不是boolean
    Boolean islogin = (Boolean)session.getAttribute("islogin");
    if (islogin == null || !islogin) {
      // 用户未登录
      response.sendRedirect("login.jsp");
    }
  %>

  </body>
```
web.xml
```xml
    <servlet>
        <servlet-name>login</servlet-name>
        <servlet-class>gjxaiou.LoginServlet</servlet-class>

    </servlet>

    <servlet-mapping>
        <servlet-name>login</servlet-name>
        <url-pattern>/login</url-pattern>
    </servlet-mapping>
```
login.jsp
```jsp
<body>
    <%--这里点击登录离职后跳转到Servlet--%>
    <form action="login" method="post">
        请输入账号密码：
        <input type="text" name="username"/>
        <input type="text" name="password"/>
        <input type="submit" value="登录">


    </form>

</body>
```

index2.jsp
```jsp
<body>
<%--  在jsp中判断session中是否存在islogin，并且还为true--%>

<%
    // 因为后面返回值为Object类型，因此强转应该是Boolean类型，不是boolean
    Boolean islogin = (Boolean)session.getAttribute("islogin");
    if (islogin == null || !islogin) {
        // 用户未登录
        response.sendRedirect("login.jsp");
    }
%>

</body>
```

LoginServlet.java
```java
/**
 * @author GJXAIOU
 * @create 2019-08-12-18:46
 */
public class LoginServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 1.获取前端传送过来的数据
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        // 2.判断数据是否正确，可以采用数据库验证
        if ("gjxaiou".equals(username) && "123456".equals(password)){
            // 登录成功，跳转到首页
            // 这里使用重定向
            resp.sendRedirect("index.jsp");

            // 在回话中保存一个变量，来表示用户已经登录
            HttpSession session = req.getSession();
            // 在session中保存一个变量
            session.setAttribute("islogin", true);
        }else {
            // 登录失败，重新回到登录页面，即回到上一页面
            resp.getWriter().print("<script>window.history.back()</script>");
        }

        // 3. 根据判断结果进行页面跳转，见上；
    }
}
```

### （二）使用迭代器实现
- 步骤：
  - 1.新建类并实现 Filter 接口；
  - 2.重写 init()、doFilter()、destroy()方法；
  - 3.在 doFilter()中进行过滤；
  - 4.在 web.xml 中配置 Filter；

首先可以将 index.jsp 和 index2.jsp 放在 main 文件下，即将需要保护的资源放入过滤器，在过滤器中判断是否已经登录；
判断是否已经登录方法：因为在登录成功之后，在 session 中保存了 islogin，通过判断 islogin 的值是否为空即可
AuthorFilter.java
```java
@Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest= (HttpServletRequest)servletRequest;
        HttpServletResponse httpServletResponse = (HttpServletResponse) servletResponse;

        HttpSession session = httpServletRequest.getSession();

        if (session.getAttribute("islogin") == null){
            // 没有登录，跳转回登录界面重新登录
            // 参数里面的相对路径：相对于过滤器当前处理的请求的路径，当前为 main/*
            httpServletResponse.sendRedirect("../login.jsp");
        }else {
            // 用户登录，放行
            filterChain.doFilter(servletRequest, servletResponse);
        }
    }
```

对应的 web.xml 中只要设置 url-pattern 值为：`/main/*`其他一样；



### 使用过滤器实现将所有文件编码统一改为 utf-8
EncodingFilter.java
```java
package filter.demo;

import javax.servlet.*;
import java.io.IOException;

/** 实现编码的过滤器
 * @author GJXAIOU
 * @create 2019-08-12-20:12
 */
public class EncodingFilter implements Filter {

    // 使用全局变量，因为所有的都要改变编码
    String encode = null;
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
       encode = filterConfig.getInitParameter("encoding");

    }

    /** 书写一个过滤器考虑问题：
     * 1.过滤器的范围；
     * 2.过滤器拦截之后需要做什么事情
     * 这里需求是做一个统一编码的过滤器
     * @param servletRequest
     * @param servletResponse
     * @param filterChain
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //  这里设置代码的编码是写死的，可以在xml中设置，方便更改
        //        // 1.设置请求编码
        //servletRequest.setCharacterEncoding("utf-8");

        servletRequest.setCharacterEncoding(encode);

        // 2.放行
        filterChain.doFilter(servletRequest, servletResponse);

    }

    @Override
    public void destroy() {

    }
}

```

web.xml
```xml
  <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>filter.demo.EncodingFilter</filter-class>
        <!-- 这里将需要制定的字符集编码当做初始化参数传入过滤器 ，在过滤器的init方法中读取-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

### 过滤器 URL 路径 设置

1。常见的设置：
- 映射到具体的某一个资源：`<url-pattern>/contact.jsp</url-pattern>`
- 映射到某一路径下所有请求：`<url-pattern>/*</url-pattern>`或`<url-pattern>/main/*</url-pattern>`
- 映射到结尾相同的一类请求：`<url-pattern>*.jsp</url-pattern>`
- 不支持路径下的某一类请求这种形式：例如：`/main/*.jsp`是不支持的；
2.代码示例
web.xml
```xml
<filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>filter.demo.EncodingFilter</filter-class>
        <!-- 这里将需要制定的字符集编码当做初始化参数传入过滤器 ，在过滤器的init方法中读取-->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
        <!-- 一个filter可以对应多个 filter-mapping,多个 url-pattern直接是并集的关系，满足其中任意一个即可  -->
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/main/*</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>*.html</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/hello/*</url-pattern>
    </filter-mapping>
```
Encoding.java
```java
 @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        //  这里设置代码的编码是写死的，可以在xml中设置，方便更改
        //        // 1.设置请求编码
        //servletRequest.setCharacterEncoding("utf-8");

        servletRequest.setCharacterEncoding(encode);

        // 2.只针对hello文件下的.jsp文件放行
        // 思路：获取当前请求路径，通过判断路径后缀的方式，判断后缀为.jsp的文件
        // 2-1: 因为ServletRequest是HttpServletRequest的父接口，先进行强转
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

        // 2-2：获取当前请求完整路径
        String requestURI = httpServletRequest.getRequestURI();

        // 2-3: 判断文件结尾
        if (requestURI.endsWith(".jsp")){
            filterChain.doFilter(servletRequest, servletResponse);
        }


    }

```





### 多个过滤器
先后通过，不是同时通过；
如果第一个通过的拦截器选择拦截，那么后面所有的都没用了；
dofilter 是将请求交个下一个过滤器进行处理；

一个应用可以有多个过滤器，一个过滤器执行完毕放行之后到下一个过滤器，过滤器执行的顺序是按照在 web.xml 中声明的先后顺序；

建议范围较大的过滤器优先通过，因为最外层的过滤器拦截，后续所有的过滤器都会拦截，只有所有的过滤器都放行才会到达资源；
过滤器实现方法 doFilter()方法转给你的 chain.doFilter()实际上是将其交给下一个过滤器使用，只有所有的过滤器都放行才会到达具体的资源；



### 属性：dispatcher
访问一个页面的三种方式：
直接访问
通过转发
通过包含
通过失败
目前是通过转发和包含，仍然 能够通过过滤器；

例如：including.jsp 里面包含 index.html 文件
```jsp
<body>
这里是include.jsp文件，下面是引用
  <jsp:include page="index.html"></jsp:include>
</body>
```

转发方式：myServlet.java
```java
// 通过注解指定Servlet路径，效果和在xml中配置Servlet,servletmappring效果一样
@WebServlet("/my")
public class myServlet extends HttpServlet{
  @override
  protected void doget(){
    req.getRequesDispatcher("index.html").forward(req, resp)

}


}
```
将以上两种方式也进行过滤，在 web.xml 中设置为：
```xml
<filter-mapping>
  <!--dispatcher 默认值即为REQUEST-->
  <dispatcher>REQUEST</dispatcher>
  <!--转发的方式进入资源也会被过滤-->
  <dispatcher>FORWARD</dispatcher>
  <!--引用方式进入资源也会被过滤-->
  <dispatcher>INCLUDE</dispatcher>
</filter-mapping>
```

- ERROR 方式，通过 ERROR 方式触发<error-page>，注意：使用此属性，<error-page>就不能正常使用；
### 统计 main 文件夹所有页面访问量
包含：index.jsp 和 index2.jsp
页面的访问量其实就是在已经访问过的次数上+1

之前访问的次数应该是多个客户端共享的，这里使用 application 作用域，在其中保存数值，所有的人都可以访问和进行操作

- 方案一：在每一个页面中都加入统计,在每一个页面中加入下面代码
```java
<%
  Integer in = (Integer)application.getAttribute("count");
  if(in == null){
    // 即从没有在application中保存该键值对，即没有登录过
    // 第一次访问就添加一个键值对
    application.setAttribute("conut", 1);
  } else {
    // 有人访问过，在此基础上+1
    application.setAttribute("count", ++in);
  }
%>
```

- 方案二：使用迭代器，因为所有页面位于 main 文件夹下：
web.xml 配置
```xml
<filter>
        <filter-name>countFilter</filter-name>
        <filter-class>filter.demo.CountFilter</filter-class>
</filter>
<filter-mapping>
        <filter-name>countFilter</filter-name>
        <url-pattern>/main/*</url-pattern>
</filter-mapping>
```

CountFilter.java 中 doFilter 中代码
```java
HttpServletRequest request = (HttpServletRequest)servletRequest;
// 在代码中获取application对象
ServletContext application = request.getServletContext();

 Integer in = (Integer)application.getAttribute("count");
  if(in == null){
    // 即从没有在application中保存该键值对，即没有登录过
    // 第一次访问就添加一个键值对
    application.setAttribute("conut", 1);
  } else {
    // 有人访问过，在此基础上+1
    application.setAttribute("count", ++in);
  }
```




