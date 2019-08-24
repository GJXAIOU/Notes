# JSP

@toc


## 一、JSP 含义

### （一）基本含义

- Java Server Pages, 它和servlet技术一样，都是Java中**用于开发动态WEB资源的技术**；
- JSP 相等于 Java + HTML；
- JSP最大的特点就是：你有一种写 HTML 代码感觉，但是 HTML 只能提供静态的 WEB 资源，而   **JSP 技术允许在 HTML 页面中嵌套 Java 代码**；
- JSP 对比Servlet有一个特别好的地方，可以对前端页面进行排版；

### （二）JSP 的执行过程

**JSP本质就是一个Servlet程序**

 1.  第一次访问 JSP 文件时，会在 Tomcat 的 work 目录下生成对应的  Java 程序；例如:  `01hello.jsp ==> _01hello_jsp.java`；
 2. 然后根据这个 Java 程序生成对应的 class 字节码文件；
 3. Tomcat 服务器会加载这个 class 字节码文件，执行里面的代码，执行的本质其实是一个  Servlet 程序；
 4. 如果是第二次访问这个 JSP 文件，不会再重新生成对应 Java 程序，以及 class 文件，直接执行对应class文件；

注：这里Tomcat服务器会根据JSP文件和Class文件的修改时间，来判断是否要重新生成Java和编译之后的Class文件

## 二、JSP语法规范

- JSP脚本
```jsp
<%
   这里只能Java代码
%>
```
 在JSP脚本中只能出现Java代码，不能出现其他的任何的东西，例如：html代码
 在JSP脚本中代码会被JSP编译器原封不动的放到Servlet程序中的_jspService方法中

- JSP表达式
 `<%=Java的表达式 %>`
 例如：`<%= new Date()%>`
 Java的表达式就是一条Java语句，但是在JSP表达式语法中不允许出现分号 ;

- JSP声明
```jsp
<%!
 定义变量，方法
 %>
```
 JSP声明中的Java代码不会被编译到_jspService方法中，而是定义在这个JSP文件对应的class中，**JSP声明中的变量是成员变量，方法可以认为是成员方法**

- JSP注释
 `<%-- 注释内容 --%>` 唯一推荐格式

示例代码：
```jsp
<%@page import="java.text.SimpleDateFormat"%>
<%@page import="java.util.Date"%>
<%@page language="java" contentType="text/html; charset=utf-8"
        pageEncoding="UTF-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
    <title>My JSP '02script.jsp' starting page</title>

</head>
<body>
    这里可以直接书写JSP！
<%--
 下面就是JSP脚本，在脚本中只能书写Java代码，
 并且这部分Java代码会出现在 _jspService方法中
 --%>
<%
    int num = 10;
    System.out.println(num);
%>
<%--
单个的JSP脚本中Java代码可以不完整，可以在多个脚本直接嵌套其他语言，
但是要求JSP脚本前后组合必须形成一个完成的Java语句。
--%>

<%
    for (int i = 0; i < 10; i++) {
%>

<h2>雷猴</h2>

<%
    }
%>



<%-- JSP表达式 --%>
<%--
    下面的语句相对于这三句：
        out.print(new Date() );
        out.print(name );
 --%>
<%=new Date() %>
<%=num %>



<%-- JSP声明 --%>
<%!
    //定义了一个成员变量
    private int aa = 10;

    //定义了一个方法
    public int add(int a, int b) {
        return a + b;
    }

    int sum = add(10, 12);
%>
<%
    System.out.println(aa);
    System.out.println(sum);
    System.out.println(add(20, 40));

%>
</body>
</html>

```

## 三、JSP三大指令

**JSP指令不是直接生成可视化文件，而且告诉JSP引擎如果处理JSP页面中的其他部分**;
​
- JSP指令的基本语法：
 `<%@ 指令名 属性名="值" %>`
​
### （一）include指令

 include指令用于**引入其他JSP页面**，如果使用include 指令引入的其他JSP页面，JSP引擎会把这两个JSP文件翻译成一个servlet，所以include指令引入通常称之为静态引入；即被引入的文件不会生成 class 文件，最终只会生成该文件的 class 文件。

- 格式：
 `<%@include file="relativeURL"%>`
 这里要求file属性使用URL用来引入其他JSP文件，这里URL必须是一个相对路径，如果使用 `/`，表示当前WEB项目的根目录

- 注意：
 1 . 被引入的文件必须符合JSP语法
 2 . 被引入的文件可以使用HTML,JSP这些文件格式，JSP引擎也会按照JSP语法规范处理
- 小规范：
 如果是静态的引入文件，通常会使用.jspf(JSP Fragments)作为文件后缀名
​Instruction.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>instruction</title>
</head>
<body>
<%@include file="Destination.jsp"%>

</body>
</html>
```
Destination.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<ul>
    <li>这里不保留其他修饰</li>
    <li>这里的所有内容将返回到调用处</li>
    <li>将调用的语句覆盖掉，执行这里的所有内容</li>
</ul>
```

### （二）page指令
用于定义JSP页面的各种属性，无论是page执行在JSP文件的哪一个位置出现，都是作用于整个JSP页面，这里书写要求：page指令最好放在页面的开始
- page指令的常用语法：
```jsp
<%@page
 [language="java"],
 [import="pageName.{* | className}"],  <%--可以导入包名.*或者包名.类名--%>
 [session="true | false"],
 [buffer="none | 8kb | sizekb"],
 [autoFlush="true | false"],
 [errorPage="relativeURL"],
 [isErrorPage="true | false"],
 [pagaEncode="UTF-8 | characterSet"],
 [contentType="text/html; charset="UTF-8"]
 %>
```
 
 注意： 使用errorPage设置当前页面的错误异常处理页面，有错误时候就跳转到错误页面中，在错误处理页面中，如果需要使用exception对象，必须设置 `isErrorPage="true"`，然后才能使用 exception 对象；

 一般情况下不会给每一个页面都设置ErrorPage, 会在 web.xml 文件中进行配置，作全局异常处理；

web.xml中配置示例：
可以定义 `except-code` 或者 `except-type` 来区分错误类型；
```jsp
<error-page>
 <!-- error-code 是要捕获的错误编号 -->
 <error-code>404</error-code>
 <!-- location是异常处理页面所在位置 -->
 <location>/common/404.jsp</location>
 </error-page>
 
 <error-page>
 <!-- error-code 是要捕获的错误编号 -->
 <error-code>500</error-code>
 <!-- location是异常处理页面所在位置 -->
 <location>/common/500.jsp</location>
 </error-page>
 
 <error-page>
 <!-- 这里是要捕获的异常类型，完整的包名.类名 -->
 <exception-type>java.lang.NullPointerException</exception-type>
 <!-- location是异常处理页面所在位置 -->
 <location>/common/error.jsp</location>
 </error-page>
```
然后在 404.jsp、500.jsp、error.jsp 文件中写错误的处理方式

 

###  （三）tablib 指令
稍等再说

## 四、JSP和Servlet

Servlet用来处理响应的数据，这些数据用于发送到前端；
JSP用户处理Servlet转发过来的响应数据；

## 五、内置对象 （将红色第六模块面试）

每一个JSP页面在第一次访问的时候， WEB服务器都会把JSP文件交给JSP引擎处理，处理之后会出现一个非常重要的方法 `_jspService`，本质就是Servlet里面的Service方法；
​
JSP引擎在调用JSP对应的`_jspService`方法时， 会提供给`_jspService`方法9个内置对象，它们不用声明，不用创建，直接使用；
​
- 九大内置对象

| 内置对象 | 对应的Servlet技术 |
| --- | --- |
| request | HttpServletRequest |
| response | HttpServletResponse |
| application | ServletContext |
| Session | HttpSession |
| out | JspWriter 类似于PrintWriter |
| pageContext | ServletContext |
| exception | Throwable |
| config | ServletConfig |
| page | Object(this) 很少使用 |

- 部分内置对象说明：
  -  out 
 out 用于给客户端发送文本数据；
 out对象是通过pageContext对象里面的getOut获取的，类似于response.getWriter()获取到的PrintWriter对象
```jsp
<%
    /*
    相对于是一个带有缓冲区的PrintWriter对象，
    如果想要JspWriter对象展示内容，
    展示在浏览器上有三种条件
    1.页面加载完成
    2.缓冲区已满
    3.调用flush方法
    注：缓冲区大小默认是8KB 是在page指令里面的buffer属性设置的
    */
    response.getWriter().write("Test PrintWriter");
    out.write("Test JspWriter Out");
    out.flush();
%>
```

  - pageContext
 pageContext对象是JSP技术最重要的一个对象，这个pageContext表示当前JSP页面的运行环境,**这个对象不仅封装了其他的8大内置(隐式)对象的引用**，而且**pageContext本身就是一个域对象**，可以用来保存数据。而且这个pageContext还封装了WEB开发中经常涉及到的一些常用操作
 例如：
 跳转其他资源，检索其他域对象的属性

```jsp
<%
    //可以获取其他内置对象
    pageContext.getRequest();
    pageContext.getResponse();
    pageContext.getServletContext();
    pageContext.getServletConfig();
    pageContext.getException();
    pageContext.getOut();
    pageContext.getPage();
%>
<%
    /*
    pageContext 是域对象
    pageContext.setAttribute(String name, Object value, int 域对象标号);
    第一个参数：属性名
    第二个参数：属性值
    第三个参数：域对象编号（给哪个域使用）
    */
    //request域
    pageContext.setAttribute("MSG", "request_msg",
            pageContext.REQUEST_SCOPE);

    //Session域
    pageContext.setAttribute("MSG", "session_msg",
            pageContext.SESSION_SCOPE);

    //整个WEB项目域对象 在JSP中叫Application 在Servlet称之为ServletContext
    pageContext.setAttribute("MSG", "application_msg",
            pageContext.APPLICATION_SCOPE);

    //当前页面page域对象，只在当前页面管用
    pageContext.setAttribute("MSG", "page_msg",
            pageContext.PAGE_SCOPE);

%>

<%
    /*
    从域对象中拿出数据，分别从各种域对象调用getAttribute(String name)方法
    */
    out.write("" + request.getAttribute("MSG") + "<br>");
    out.write("" + session.getAttribute("MSG") + "<br>");
    out.write("" + application.getAttribute("MSG") + "<br>");
    out.write("" + pageContext.getAttribute("MSG") + "<br>");

    //从域对象中查找属性 就近原则
    //pageContext -> request -> session -> appliction
    out.write("" + pageContext.findAttribute("MSG") + "<br>");
    // 获取request对象，因为06和07两个request，不能直接从06中获取07的request，，需要重定向
    request.getRequestDispatcher("07getattribute.jsp").forward(request, response);
%>
```

  - 四个域对象
 request
 session
 application
 pageContext

## 三层结构

三层架构
表示层：包含JSP和Servlet等与WEB相关的内容； 
业务逻辑层：业务层中不包含JavaWeb API，它只关心业务逻辑 
数据访问层（持久化层）：封装了对数据库的访问细节,比如针对数据库表的增删改查操作
​
练习：修改通讯录管理程序，通过jsp实现界面