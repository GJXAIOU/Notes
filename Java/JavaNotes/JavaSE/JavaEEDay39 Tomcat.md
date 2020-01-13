# JavaEEDay39 Tomcat
@toc

## 一、Tomcat从入门到熟悉

### （一） B/S 和 C/S

- B/S：浏览器和服务器架构
 www.baidu.com  www.taobao.com
  -  好处：
 1 . 不需要符合各种平台环境的客户端，有浏览器就可以
 2 . 更新方便，服务器更新，浏览器只要刷新就可以获取到最新的信息
​
- C/S: 客户端和服务器架构
 QQ 微信 快手  LOL PUBG
  -  好处：
 1 . 用户体验比B/S略好
  -  弊端：
 1 . 如果要使用服务，必须装软件
 2 . 服务器更新之后，要求客户端页随之更新
​
对于中小企业更多的会选择使用B/S
HTML CSS JavaScript JavaEE MySQL/Oracle Tomcat Nginx

### （二） 什么是服务器
计算机：CPU  内存  硬盘  带宽
流量(带宽)最重要：

服务器除了硬件之外，还有软件：
这个软件是用来提供共享资源能力的，让网络端的电脑能够访问服务器 。例如 Tomcat  Nginx  apche 组织
产品有：
 WebLogic：BEA公司，收费的，完全支持JavaEE规范
 WebSphere：IBM公司，收费的，完全支持JavaEE规范
 JBoss: RedHat公司，收费的，完全支持JavaEE规范
 Tomcat：Apche组织，完全免费开源的，支持我们能够使用到的JavaEE 部分规范,例如 Servlet  JSP  JDBC

数据库服务器：安装了数据库软件的一个电脑，MySQL Oracle SQLServer
WEB服务器：提供WEB服务器的一台电脑，安装WEB服务器软件

## 二、Tomcat的基本使用

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，它早期的名称为catalina，后来由Apache、Sun 和其他一些公司及个人共同开发而成，并更名为Tomcat。Tomcat 是一个小型的轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选，因为Tomcat 技术先进、性能稳定，成为目前比较流行的Web 应用服务器。Tomcat是应用（java）服务器，它只是一个servlet容器，是Apache的扩展，但它是独立运行的。目前最新的版本为Tomcat 8.0.24 Released。

Tomcat不是一个完整意义上的Jave EE服务器，它甚至都没有提供对哪怕是一个主要Java EE API的实现；但由于遵守apache开源协议，tomcat却又为众多的java应用程序服务器嵌入自己的产品中构建商业的java应用程序服务器，如JBoss和JOnAS。尽管Tomcat对Jave EE API的实现并不完整，然而很企业也在渐渐抛弃使用传统的Java EE技术（如EJB）转而采用一些开源组件来构建复杂的应用。这些开源组件如Structs、Spring和Hibernate，而Tomcat能够对这些组件实现完美的支持。

 * 1 . 从官网下载Tomcat服务器软件： https://tomcat.apache.org/download-80.cgi#8.0.48

 * 2 . 下载Tomcat 8.0.48：这里建议使用压缩版：64-bit window.zip

 * 3 . 下载完成：解压，解压之后的文件夹放到一个完全没有中文的目录下
  
 * 4 . 启动Tomcat服务器
   *  安装路径下/bin/startup.bat  双击，弹出黑框：黑框如果存在，表示Tomcat服务器正在运行
要求：**绝对不允许用右上角的X号关闭黑框** 
   * 在浏览器上输入http://localhost:8080
   * 页面可以加载表示Tomcat服务器启动成功
   
 * 5 . 关闭Tomcat服务器：在安装路径/bin/shutdown.bat

- Tomcat使用常见问题：
   - 1 . 闪退
   考虑JDK环境变量的配置
 Tomcat服务器使用Java写的，需要JVM支持
 需要配置 JAVA_HOME CLASSPATH Path

  -  2 . 端口被占用
 [以下操作，请先关闭Tomcat服务器]
 修改Tomcat服务器的端口号
 修改Tomcat的配置文件，Tomcat配置文件在安装路径/conf/server.xml
 `<Connector port="8080" protocol="HTTP/1.1"`
 `connectionTimeout="20000"`
 `redirectPort="8443" />`
 例如：修改为8081，保存修改，重启Tomcat服务器就可以使用了

- **Tomcat的目录结构**
  - bin：存放Tomcat程序的二进制文件 目录
  - conf: 存放Tomcat的配置信息，主要操作对象是server.xml
  - lib: 存放Tomcat运行需要的JAR包，这里面有一个需要关注的是servlet-api.jar
  - logs: 存放运行时日志的临时文件夹
  - webapps：共享资源路径，WEB应用保存的目录
  - work: Tomcat的运行目录， JSP运行的临时目录。JSP生成的临时文件都会放在这里

- Tomcat共享文件演示示例： 
 主要操作的是webapps这个文件夹
  -  首先在webapps这个文件下创建一个目录：mywebs 【在 Tomcat 没有启动的状态下】
   - 其次在mywebs下写一个html文件： index.html
  - 最后在浏览器输入：http://10.8.156.34:8080/mywebs/index.html就可以访问
 http: 这是HTTP协议
 10.8.156.34: 服务器的IP地址
 8080：服务器Tomcat软件的端口号
 mywebs：在Tomcat服务器下共享目录webapps里面的一个JavaWEB项目目录名
 index.html 是一个HTML文件
小知识：
 index.html 因为他的名字是index,在没有确定申请访问哪一个文件时，会默认打开index.html

## 三、WEB应用目录结构
- WebRoot  ：WEB项目/应用的根目录(这个名字没有中文即可)。里面有两大列
  - 静态资源 (例如 HTML CSS JavaScript img vedio audio)文件；名字是 WEB-INF：【固定写法】
     - classes: 存放二进制字节码文件的目录 放入.class文件【固定写法，可选】
     -  lib: 存放当前WEB项目运行需要的JAR包 【固定写法，可选】
  - web.xml 非常重要【现在非常重要】
​
注意事项：
 1 . **WEB-INF目录的下内容不能通过浏览器目录方式访问**
 2 . **如果需要访问WEB-INF里面的资源内容，需要配置web.xml**



## 四、Servlet入门

### （一）Servlet 含义

- 静态资源和动态资源的区别
  - 静态资源：
用户在多次访问的情况下，但是每一次页面的源代码没有发生任何的改变
  - 动态资源:
 用户每一次访问，申请到的页面源代码都是不一样的

- Servlet是Java语言实现动态资源的开发技术
要求：
 1 . Servlet程序只能在Tomcat服务器上运行  【记住】
 2 . Servlet其实是一个非常普通的类，只不过继承了HttpServlet，覆盖了doGet和doPost方法

### （二）手动书写一个Servlet程序

- 1 . 定义一个类，要求继承HttpServlet
 HttpServlet并不存在于当前我们使用的JDK1.8中，因为 Servlet程序是要交给Tomcat运行的，所以Tomcat是支持HttpServlet，那么在Tomcat下必须有HttpServlet当前这个代码，在Tomcat安装路径/lib/servlet-api.jar
其次需要把添加 servlet-api.jar
- 2 . 书写Servlet程序
   - HttpServletRquest    是  servlet请求
   - HttpServletResponse   是 servlet响应

  - 因为当前是使用响应给浏览器发送数据：
 要求是：设置响应在浏览器上展示的方式和编码集 使用HTML展示，字符集为utf-8
 resp.setContentType("text/html;charset=utf-8");
 PrintWrite resp.getWriter(); 专门用来给浏览器写入数据的对象 基于IO流的

- 3 . 执行Servlet程序
 这里servlet程序不能再交给Eclipse执行了，而是要在Tomcat服务器上执行Servlet程序
 在Tomcat服务器软件根目录下webapps中创建一个Web项目文件夹，要按照Web项目要求创建
  - 项目目录如下：


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

 
