# JavaEEDay39 Tomcat
## 一、Tomcat从入门到熟悉

### （一） B/S 和 C/S

#### B/S：浏览器和服务器架构

示例：www.baidu.com  www.taobao.com

好处：

- 不需要符合各种平台环境的客户端，有浏览器就可以
- 更新方便，服务器更新，浏览器只要刷新就可以获取到最新的信息
    ​

#### C/S: 客户端和服务器架构

如：QQ 与微信

好处：

- 用户体验比 B/S 略好

弊端：

- 如果要使用服务 ，必须装软件
- 服务器更新之后，要求客户端页随之更新


### （二） 什么是服务器

服务器除了硬件之外，还有软件：
这个软件是用来提供共享资源能力的，让网络端的电脑能够访问服务器 。例如 Tomcat  Nginx  apche 组织
产品有：
 WebLogic：BEA公司，收费的，完全支持JavaEE规范
 WebSphere：IBM公司，收费的，完全支持JavaEE规范
 JBoss: RedHat公司，收费的，完全支持JavaEE规范
 Tomcat：Apche组织，完全免费开源的，支持我们能够使用到的JavaEE 部分规范,例如 Servlet  JSP  JDBC

数据库服务器：安装了数据库软件的一个电脑，MySQL Oracle SQLServer
WEB 服务器：提供 WEB 服务器的一台电脑，安装 WEB 服务器软件

## 二、Tomcat的基本使用

Tomcat 服务器是一个免费的开放源代码的 Web 应用服务器，Tomcat 是 Apache 软件基金会（Apache Software Foundation）的 Jakarta 项目中的一个核心项目，它**早期的名称为 catalina**，后来由 Apache、Sun 和其他一些公司及个人共同开发而成，并更名为 Tomcat。Tomcat  是一个小型的轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试 JSP 程序的首选，因为 Tomcat  技术先进、性能稳定，成为目前比较流行的 Web 应用服务器。Tomcat 是应用（java）服务器，它**只是一个 servlet 容器**，是 Apache 的扩展，但它是独立运行的。

Tomcat 不是一个完整意义上的 Jave EE 服务器，它甚至都没有提供对哪怕是一个主要 Java EE API 的实现；但由于遵守 apache 开源协议，tomcat 却又为众多的 java 应用程序服务器嵌入自己的产品中构建商业的 java 应用程序服务器，如JBoss和JOnAS。尽管Tomcat对Jave EE API的实现并不完整，然而很企业也在渐渐抛弃使用传统的Java EE技术（如EJB）转而采用一些开源组件来构建复杂的应用。这些开源组件如 Structs、Spring 和Hibernate，而 Tomcat 能够对这些组件实现完美的支持。

 * 1 . 从官网下载Tomcat服务器软件： https://tomcat.apache.org/download-80.cgi#8.0.48

 * 2 . 下载 Tomcat 8.0.48：这里建议使用压缩版：64-bit window.zip

 * 3 . 下载完成：解压，解压之后的文件夹放到一个完全没有中文的目录下
  
 * 4 . 启动 Tomcat 服务器
   *  安装路径下 `/bin/startup.bat`  双击，弹出黑框：黑框如果存在，表示 Tomcat 服务器正在运行
   要求：**绝对不允许用右上角的X号关闭黑框** 
   * 在浏览器上输入http://localhost:8080
   * 页面可以加载表示 Tomcat 服务器启动成功
   
 * 5 . 关闭 Tomcat 服务器：在安装路径 `/bin/shutdown.bat`

### Tomcat 使用常见问题：

- 1 . 闪退
考虑 JDK 环境变量的配置，Tomcat 服务器使用 Java 写的，需要 JVM 支持，需要配置 `JAVA_HOME CLASSPATH`；

-  2 . 端口被占用
 [以下操作，请先关闭 Tomcat 服务器]
  修改 Tomcat 服务器的端口号
  修改 Tomcat 的配置文件（在安装路径 `/conf/server.xml`)
  `<Connector port="8080" protocol="HTTP/1.1"`
  `connectionTimeout="20000"`
  `redirectPort="8443" />`
  例如：修改为 8081，保存修改，重启 Tomcat 服务器就可以使用了

- **Tomcat 的目录结构**

  - bin：存放 Tomcat 程序的二进制文件目录
  - conf：存放 Tomcat 的配置信息，主要操作对象是 `server.xml`
  - lib: 存放 Tomcat 运行需要的 JAR 包，这里面需要关注的是 `servlet-api.jar`
  - logs: 存放运行时日志的临时文件夹
  - webapps：共享资源路径，WEB 应用保存的目录
  - work: Tomcat 的运行目录，JSP 运行的临时目录。JSP 生成的临时文件都会放在这里

- Tomcat 共享文件演示示例： 
  主要操作的是 webapps 这个文件夹

  -  首先在 webapps 这个文件下创建一个目录：mywebs 【在 Tomcat 没有启动的状态下】
   - 其次在 mywebs 下写一个 html 文件： index.html
  - 最后在浏览器输入：http://10.8.156.34:8080/mywebs/index.html就可以访问
   http: 这是 HTTP 协议
    10.8.156.34: 服务器的 IP 地址
    8080：服务器 Tomcat 软件的端口号
    mywebs：在 Tomcat 服务器下共享目录 webapps 里面的一个 JavaWEB 项目目录名
    index.html 是一个 HTML 文件

    index.html 因为他的名字是 index，在没有确定申请访问哪一个文件时，会默认打开 index.html

## 三、WEB应用目录结构

- WebRoot  ：WEB 项目/应用的根目录（这个名字没有中文即可）。里面有两大列
  - 静态资源 (例如 HTML CSS JavaScript img vedio audio)文件；名字是 WEB-INF：【固定写法】
     - classes: 存放二进制字节码文件的目录 放入 `.class` 文件【固定写法，可选】
     -  lib: 存放当前 WEB 项目运行需要的 JAR 包 【固定写法，可选】
  - web.xml 非常重要【现在非常重要】

注意事项：
**WEB-INF 目录的下内容不能通过浏览器目录方式访问，如果需要访问 WEB-INF 里面的资源内容，需要配置 web.xml**

## 四、Servlet入门

### （一）Servlet 含义

- 静态资源和动态资源的区别
  - 静态资源：
  用户在多次访问的情况下，但是每一次页面的源代码没有发生任何的改变
  - 动态资源:
   用户每一次访问，申请到的页面源代码都是不一样的

- Servlet 是 Java 语言实现动态资源的开发技术
- Servlet程序只能在Tomcat服务器上运行；
- Servlet其实是一个非常普通的类，只不过继承了 HttpServlet，覆盖了 doGet 和 doPost 方法；

### （二）手写一个 Servlet 程序

- 定义一个继承 HttpServlet 的类
 HttpServlet 并不存在于 JDK1.8 中，因为 Servlet 程序是要交给 Tomcat 运行的，所以 Tomcat 是支持 HttpServlet，那么在 Tomcat 下必须有HttpServlet 当前这个代码，在 Tomcat 安装路径 `/lib/servlet-api.jar`
 其次需要把添加 servlet-api.jar
 
- 书写 Servlet 程序
   - HttpServletRquest：是 servlet 请求
   
   - HttpServletResponse：是 servlet 响应
  
   - 因为当前是使用响应给浏览器发送数据：
    要求是：设置响应在浏览器上展示的方式和编码集：使用 HTML 展示，字符集为 utf-8

    ```java
    resp.setContentType("text/html;charset=utf-8");
    // 专门用来给浏览器写入数据的对象 基于IO流的
    PrintWrite resp.getWriter(); 
    ```
   
- 执行 Servlet 程序
 要在 Tomcat 服务器上执行 Servlet 程序，首先在 Tomcat 服务器软件根目录下 webapps 中创建一个 Web 项目文件夹，要按照 Web 项目要求创建


  WEB-INF：【固定写法】
          classes:    找到当前Servlet程序的class文件，要把这个文件放入到classes文件中
 【注意】要求放入的class文件要带有完整的包名
           web.xml 按照Servlet程序的规范书写XML文件
```xml
# 这里还有其他的文件
<!--4 . WEB.xml文件中的内容 -->
 <servlet>
     <!-- servlet的内部名称 这个名字可以自定义-->
     <servlet-name>HelloServlet</servlet-name>
     <!-- servlet程序的class文件 要求是完整的类名，也就是包名.类名 -->
     <servlet-class>a_firstServlet.HelloServlet</servlet-class>
 </servlet>
<servlet-mapping>
     <!-- servlet的名字 【要求和上边的名字必须一模一样】-->
     <servlet-name>HelloServlet</servlet-name>
     <!-- servlet的访问名称: http://localhost:8080/mywebs/访问名 -->
     <url-pattern>/hello</url-pattern>
 </servlet-mapping>
```

 
