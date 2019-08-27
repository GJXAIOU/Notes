#  JavaEEDay41 Servlet 与session

## 一、Servlet
注意事项：
 1 . 防止线程安全问题
 2 . 在使用同步代码块选择锁对象，通常会使用当前servlet程序对象

```java
package a_thread;
import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 统计你是当前页面的第几个访问者
 * 利用一个计数器，每一次有一个人访问就 + 1
 */
public class VisitedCount extends HttpServlet {
    int count = 1;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
		/*int count = 1;
		局部变量，每一次调用doGet方法，都会重新创建一个count变量, 无法作为计数器*/
        //告诉浏览器解析数据的方式和使用的字符集
        resp.setContentType("text/html;charset=utf-8");
        //服务器发送数据使用的字符集
        resp.setCharacterEncoding("utf-8");
		
        synchronized (this) { 
			/*
			 这里因为操作了共享资源计数器count，为了避免线程的安全问题，这里采用加锁的方式(同步代码块)
			锁对象用this，this表示当前Servlet程序类VisitedCount的对象，因为在Tomcat服务器
			上当VisitedCount对象被创建和init之后，不会在创建新的VisitedCount对象，满足锁对象
			的基本要求
			 */
            resp.getWriter().write("你是第" + count + "访问者");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            count++;
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

### ServletConfig对象
在 web.xml 中补充以下信息：
```xml
<servlet>
 <servlet-name>ConfigServlet</servlet-name>
 <servlet-class>b_servletconfig.ConfigServlet</servlet-class>
 
 <!-- servlet程序的初始化参数 -->
 <init-param>
 <param-name>user</param-name>
 <param-value>root</param-value>
 </init-param>
 
 <init-param>
 <param-name>password</param-name>
 <param-value>123456</param-value>
 </init-param>
 </servlet>
```
在web.xml文件中利用<init-param>标签来定义servlet初始化参数，主要包含两个子标签
<param-name> <param-value>
​
当Tomcat服务器程序启动，WEB容器在创建servlet时，会把初始化参数封装到ServletConfig对象中，这个ServletConfig对象，会作为Servlet对象的初始化方法init的参数
​
【注意】
 在一个Servlet 对应一个ServletConfig
```java
package b_servletconfig;
import java.io.IOException;
import java.util.Enumeration;
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class ConfigServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        //因为获取的是当前ConfigServlet程序的ServletConfig对象,因此使用this调用getServletConfig()方法
        ServletConfig config = this.getServletConfig();

        //获取当前Servlet程序的名字
        System.out.println(config.getServletName());
		
        //根据param-name获取对应的param-value值
        String value = config.getInitParameter("user");
        System.out.println(value);
		
        //获取到所有初始化参数的枚举类型
        Enumeration<String> names = config.getInitParameterNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            System.out.println(name + ":" + config.getInitParameter(name));
        }

    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        doGet(req, resp);
    }
}
```
用于获取参数值以及属性；



## ServletContext

表示当前整个WEB应用，一个WEB应用有且只有一个ServletContext对象
​
1 获取WEB应用的路径
 ServletContext context = this.getServletContext();
 context.getContextPath();
 request.getContextPath();
​
2.WEB应用的全局参数
 <!-- 这是针对整个WEB项目的初始化参数 -->
 <context-param>
 <param-name>AAA</param-name>
 <param-value>aaa</param-value>
 </context-param>
 context.getInitParameter(Stirng paramName);

3.ServletContext 是一个域对象
 request，session 也是域对象
​
 可以利用ServletContext域对象特征，完成在整个WEB项目中的数据共享传递
 不单单能传递基本数据类型，也可以传递类对象类型
 setAttribute(String name, Object value);
 Object    getAttribute(String name);
​
4.转发和重定向
转发会携带数据，但是重定向没有携带数据；
 WEB服务器接受到客户端的请求之后，让Tomcat服务器去调用另一个WEB资源，这称之为转发


-  request域转发
   - request.setAttribute(String name, Object value);

   - 转发的方法 `req.getRequestDispatcher(String uri).forward(HttpServletRequest req, HttpServletResponse resp);`
       - uri：服务器端要访问别的WEB资源路径，这里是服务器端的地址，一般用 / 开头，表示WEB项目的根目录
       - HttpServletRequest req: 原本用户请求当前页面的request对象
       - HttpServletResponse resp: 原本用户请求当前页面生成对应的response对象

 - 重定向
 response.sendRedirect(String url);
 这个URL是相对于webapps的，是提供给浏览器使用的URL

 推荐写法是：
 response.sendRedirect(request.getContextPath() + "重定向地址");

 【注意】
 在重定向的情况下，request域不能使用

## 会话控制

1. 软件程序中的会话  一次会话：  打开浏览器 -> 访问服务器上的内容 -> 关闭浏览器

登录业务场景：  打开浏览器 -> 访问登录页面 -> 输入 用户名 密码 验证码 -> 发送给服务器 -> 服务器验证消息  -> 返回给浏览器 -> 浏览器加载主页

  发现：在百度首页登录了用户名，在所有的百度服务的页面下都是一个登录的状态

  购物车场景：  在手机淘宝APP选择了一个商品，加入购物车
  打开浏览器 -> 淘宝网 -> 登录账号 -> 查看购物车，也可以看到手机APP添加商品
  这里的购物车数据在那里保存？？？

  会话控制技术：  管理 浏览器客户端 和 服务器 之间进行会话过程生产的会话数据

## 会话控制技术

Cookie技术： 会话数据保存在浏览器客户端，也就是用户本地
Session技术：会话数据保存在服务器上

### Cookie技术

- 核心方法：
 1 . 创建Cookie对象
 Cookie(String name, String value); 
 2 . 设置Cookie
 setValue(String value) //通过name来设置Cookie的Value
 setPath(String path); //设置Cookie的有效范围 
 setMaxAge(int time);     //设置最大有效时间
 3 . 发送Cookie到浏览器保存
 response.addCookie(Cookie c); //发送Cookie给浏览器
 4 . 服务器接受浏览器访问时带来的Cookie数据
 Cookie[] request.getCookies();

- Cookie有一定的局限性
 1 . 限制数据类型 必须是String
 2 . 不能保存中文
 3 . Cookie保存数据容量较小，只有4KB

 如果要保存4KB以上数据，或者要保存中文数据，就不能使用Cookie技术，只能用Session

 【但是有一个前提】
 如果浏览器没有打开Cookie功能， Session也无法使用

 Sesion的特点：
 会话数据保存在 服务器 上，可以生成一个临时或者永久的temp文件

### Session技术

HttpSession
 1 . 获取Session对象
 HttpSession getSession();
 HttpSession getSession(boolean create);  // 再创建一个 session

 2 . 设置Session对象
 void   setMaxInactiveInterval(int interval); //设置Session 的有效时间
 void   invalidate(); //销毁Session
 String   getId();    //获取Session的ID

 3 . Session也是一个域对象，这里可以保存数据属性到Session
 void setAttribute(String name, Object value); //设置Session里面保存是会话数据内容，可以保存一个对象
 Object getAttribute(String name); //获取到Session里面数据
 void removeAttribute(String name); //清空Session里面的数据