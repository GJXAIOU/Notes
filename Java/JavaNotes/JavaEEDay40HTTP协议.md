# JavaEEDay40 HTTP 和 Servlet
@toc

## 一、 HTTP协议

- HTTP是 HyperText Tranfer Protocol 超文本传输协议
  - 是一个规范，是用来约束WEB服务器和浏览器直接的通讯协议
  - 基于TCP/IP的一个协议，用于连接WEB服务器和WEB浏览器
​
- HTTP的协议版本：
 HTTP/1.0 HTTP/1.1

- HTTP协议对应浏览器来说可以分为两大类：
  - HTTP请求:从浏览器发送给服务器的请求内容
  - HTTP响应:浏览器接受服务器发送的数据内容

### （一）HTTP请求：
  - HTTP请求头示例
```http
GET / HTTP/1.1   --请求行
Host: www.baidu.com --请求的服务器地址
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.89 Safari/537.36
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: BAIDUID=2010BC084DE91EB2B0EDD6EE92330BE1:FG=1; BIDUPSID=2010BC084DE91EB2B0EDD6EE92330BE1; PSTM=1499085770; MCITY=-268%3A; BDORZ=B490B5EBF6F3CD402E515D22BCDA1598; BDRCVFR[mkUqnUt8juD]=mk3SLVN4HKm; sugstore=1; BD_HOME=0; H_PS_PSSID=25575_1454_21111_17001_20927; BD_UPN=12314353
```

- 请求方式
    - 使用GET请求方式：
    示例`GET /Day40/TestHttp?name=12345&password=12345678 HTTP/1.1`，其中?之后就是GET请求的数据参数
​
    - 使用POST请求方式：**在URL中没有看到任何的参数**, 而且请求的方式也略有不同
示例：`POST /Day40/TestHttp HTTP/1.1`, 请求的数据在 POST请求特有的请求体内保存，也就是谷歌浏览器 From Data

### （二）HTTP 响应 

- 响应头：
```http
HTTP/1.1 200 OK
 Server: Apache-Coyote/1.1  -- 使用的服务器是什么
 Content-Length: 0          -- 响应得到的数据长度，字节数
 Date: Tue, 16 Jan 2018 03:01:56 GMT  时间
```
格式：
 HTTP/版本号(目前常用的版本号都是1.1) 状态码  原因的简单描述
| 状态码 | 含义 |
| --- | --- |
| 100 ~ 199 | 表示成功接受请求，要求客户端继续提交下一次请求 |
| 200 ~ 299 | 表示成功接受并且处理完成了整个操作流程，常用200 |
| 300 ~ 399 | 为了完成请求，要求客户做出进一步的操作，例如：跳转，资源不存在，跳转到新界面 常用302 304 和 307 |
| 400 ~ 499 | 客户请求错误，常用404 |
| 500 ~ 599 | 服务器GG了！！！ 常用500 |

## 二、Servlet

全称Java Servlet，是用Java编写的服务器端程序。 其主要功能在于交互式地浏览和修改数据，生成动态Web内容。

Tomcat服务器启动之后输入： http://localhost:8080/Day40/TestHttp
```xml
 <servlet>
 <!-- 自定义的名字 -->
 <servlet-name>TestHttp</servlet-name>
 <!-- Servlet程序完整的包名.类名 -->
 <servlet-class>com.qfedu.http.TestHttp</servlet-class>
 </servlet>
 <servlet-mapping>
 <!-- 要求和上面servlet-name完全一致 -->
 <servlet-name>TestHttp</servlet-name>
 <!-- 映射路径 -->
 <url-pattern>/TestHttp</url-pattern>
 </servlet-mapping>
```
- 浏览器访问服务器的URL分析
 1 . 浏览器根据输入的URL来访问Tomcat服务器 http://localhost:8080
 2 . /Day40 访问在 Tomcat 服务器下 webapps 里面的 Day40 项目文件夹
 3 . /TestHttp 用来匹配在【WEB-INF】下的【web.xml】文件里面的 url-pattern,服务器开始工作；
​

### （一）映射路径
​其中 `/TestHttp` 称之为 映射路径
- 映射的流程：
 1 . 在【WEB-INF】下的 web.xml 文件中匹配 url-pattern
 2 . 匹配到就可以获取到 <servlet-mapping>，可以找到 <servlet-name> 标签内容，获取 servlet-name
 3 . 通过匹配 <servlet-name>，找到 <servlet>，也就可以获取到 <servlet-class> 标签内容
 在这个标签中包含了 servlet 程序运行需要执行的 .class 文件
 4 . 通过这个包名.类名就可以加载这个 .class (字节码文件)，运行 servlet 程序，这里用到的就是【反射】的思想

- url-pattern 匹配形式：
   - 精确匹配：
 url-pattern规定的是写什么样的匹配，就在URL中写什么
 例如：
 /TestHttp 在浏览器中只能输入：http://localhost:8080/Day40/TestHttp
 /TestHttp/hi 在浏览器中只能输入：http://localhost:8080/Day40/TestHttp/hi
 一个字符都不能差！！！

   - 模糊匹配：
 `/*`   http://localhost:8080/Day40/任意内容      都可以访问当前的Servlet程序
 `*.do` 格式： http://localhost:8080/Day40/任意内容.do
 `*.html (伪静态)` 格式： http://localhost:8080/Day40/任意内容.html
 `*.action` 同上
 `*.jsp` 同上

 注意事项：
 1 . 模糊匹配不能同时使用\和* 不允许
 2 . 如果同时存在模糊匹配和精确匹配，精确匹配的优先级更高
 3 . 使用后缀名的模糊匹配优先级最低
 4 . 要求所有的Servlet程序的url-pattern都不能为/，不允许和Tomcat默认Servlet冲突，下面 DefaultServlet 倒数第二行的中间就是一个 /

### （二）Tomcat默认的Servlet程序 

在URL中输入 http://localhost:8080/Day40/有以下的流程

  1 . 在Day40的项目目录下【WEB-INF】里面的web.xml中匹配 url-pattern为 / 的标签 【精确匹配】
  2 . 如果没有匹配到这个 / 映射路径，这个时候Tomcat会把这个映射路径交给 在Tomcat中
  默认的Servlet程序中，程序名为： DefaultServlet，文件见下面的代码；
  3 . DefaultServlet会首先在Day40的项目目录下找有没有对应的index.**文件 
      这里可以支持index.html index.htm index.jsp
  4 . 如果有，将对应的index.html发送给浏览器
  5 . 如果没有报状态码 404 跳转到默认的404页面
```html
<servlet>
      <servlet-name>default</servlet-name>
      <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
      <init-param>
          <param-name>debug</param-name>
          <param-value>0</param-value>
      </init-param>
      <init-param>
          <param-name>listings</param-name>
          <param-value>false</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
      <servlet-name>default</servlet-name>
      <url-pattern>/</url-pattern>
	</servlet-mapping>
```

 
### （三）Servlet生命周期【重点】

之前提到的生命周期有：成员变量，局部变量，类对象，线程的生命周期；

- Servlet 程序的生命周期
是由 Tomcat 服务器控制的，因为 Servlet 程序，目前只能在 Tomcat 服务器上运行；

- Servlet 生命周期的四个主要方法：
  - 构造方法：用于创建 Servlet 对象:有且只执行一次;
  - init 方法 : 初始化构造方法创建好的 Servlet 对象;有且只执行一次；
  -  service 方法：提供 Servlet 程序的服务 :想用几次用几次
  当指定的 Servlet 程序被创建，初始化之后，并没有销毁，而且从任何的浏览器任何的IP地址访问
  当前的 Servlet 程序，都不会重新创建，而是始终执行这一个 Servlet 程序，所以 **Servlet就是一个单例对象**。
   - destroy 方法 :销毁 Servlet 对象，在 Tomcat 服务器关闭时执行 :有且只执行一次

![Servlet生命周期时序图]($resource/Servlet%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E6%97%B6%E5%BA%8F%E5%9B%BE.png)


### （四）Servlet 的自动加载

在Tomcat服务器上，假如存在Servlet程序代码量巨大，操作的数据很恐怖，并没有自动启动，而是在用户访问的时候才启动当前servlet，这样会导致用户访问的友好性，时间效率很低；

如果说一个servlet程序运行启动过程比较长，操作复杂度的较大，为了提高用户访问体验，会设置自动加载；

需要修改web.xml文件
```xml
 <servlet>
    <servlet-name>LifeDemo</servlet-name>
    <servlet-class>d_life.LifeDemo</servlet-class>
    <!-- 这里就是负责自动加载的XML语句，中间的数字从1开始，数字越小，优先级越高 -->
    <load-on-startup>2</load-on-startup>
  </servlet>
```
用来在Tomcat服务器上提前加载一些重要的servlet