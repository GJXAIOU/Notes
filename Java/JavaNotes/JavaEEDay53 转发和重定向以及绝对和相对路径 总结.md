---
style : summer
tags : [转发和重定向，相对路径和绝对路径]
---

# JavaEEDay 53 转发和重定向以及绝对和相对路径 总结

## 一、转发和重定向区分

- 代码上:转发是通过**request**对象的`getRequestDispatcher().forward()`方法,重定向是 **response**对象的`sendRedirect()`方法；
- 转发和重走向都会发生资源的跳转,但是转发发生的跳转之后的路径还是跳转之前的路径,重定向的之后的路径是跳转之后的路径；（转发路径不变，重定向路径改变）
- **转发发生在服务器,重定向发生在客户端**；
- **转发是一次请求,重定向是两次请求**；(通过浏览器 F12，然后查看 network)
- 发生转发之后与之前的资源共用一个 request 对象,而重定向是两个 request对象；
- 转发和重走向中都可以使用绝对路径和相对路径。如果使用绝对路径转发的`/`表示的是当前 WEB 应用程序的根目录,重定向中的`/`表示的是整个 WEB 站点的根目录
- 转发只能跳转到同一项目下的资源,重定向可以 跨域,也就意味着可以跳转到非本项目的路径；
```java
@WebServlet("/index")
public class ServletTest extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 尝试使用转发访问百度 : 失败
        req.getRequestDispatcher("http://www.baidu.com").forward(req, resp);

        // 尝试使用重定向跨域访问百度 ：成功
        resp.sendRedirect("http://www.baidu.com");
    }
}
```


![转发]($resource/%E8%BD%AC%E5%8F%91.jpg)
![重定向]($resource/%E9%87%8D%E5%AE%9A%E5%90%91.jpg)



比较 |request.getRequestDispatcher("xxx.jsp").forward(request, response); |response.sendRedirect("xxx");
---|---|---
变量作用域 |request中存放的变量不会失效 |request存放的变量失效，进入一个新的request作用域
浏览器的地址 |地址栏保持初值不会改变 |改变为重定向的页面地址
作用范围 |只能是同一个web应用中的组件 |可重定向到其他程序资源或者其他站点
URL以/开头 |相对于web应用程序根目录 |相对于web站点根目录
各自优点 |相对高效，隐藏实际链接地址 |能跳转到其他的服务器上的资源。


我们也可以通过一个借钱的场景来表述转发和重定向的区别：
A 向 B 借钱，B 没有钱，C 有钱；
- 方式一：B 向 C 借钱，然后给 A；因为 A 不知道 C 的存在，认为钱是从 B 来，因此 A 只借了一次钱； -> 转发；
- 方式二：B 反馈给 A 让其向 C 借钱；A 知道 C 的存在，A 一共借了两次钱；  ->重定向；
A相当于客户端，通过一个链接指向 服务器 B（B 可以是 jsp 或者 Servlet），因为方式二中 A 请求了两次，且都是从客户端请求，因此重定向发生在客户端；


## 二、绝对路径和相对路径

示例文件目录：
/index1
/main/index2
/main/main2/index3
/index4

去往  | 相对路径  | 绝对路径
--- | --- | ----
index1 -> index2 | main/index2 | /main/index2
index2 -> index1 | ../index1 | /index1
index1 -> index4 | index4   | /index4
index1 -> index3 | main/main2/index3 | /main/main2/index3
index3 -> index2 | ../index2 | /main/index2     

**相对路径**
- `..`表示上一级目录；
- 优点：路径的字符数较少；
- 缺点：一旦某一个资源路径发生改变，则其他资源就会无法定位到它；

**绝对路径**
- `/`表示根目录，所有的资源相对于根目录都有一个固定的路径；
- 优点：清晰，容易定位；
- 缺点：目录层级较多的时候，字符较长；

==开发过程中建议使用绝对路径==

**关于`/`的使用**：`/`表示根目录；
- 【web.xml 中的`/`表示：`"localhost:8080/projectname/"`】web.xml 中使用`/`，给 Servlet 指定 URL-partner 是为 Servlet 指定通过什么路径可以访问当前 Servlet。比如我们设置为`"/test"`,那其实我们需要通过 `localhost:8080/ projectname/test`才能够访问到 servlet,所以这里的`/`表示的是 `"localhost:8080/ projectname/"`,`"/"`可以理解是一个缩写。

- 【重定向中的`/`表示到：`localhost:8080/`】**转发和重定向他们的路径都可以是相对路径或者绝对路径,如果是相对路径,在转发和重定向中都一样**,但是如果他们使用绝对路径就不在一样了,转发中`"/"`同样也是表示`"localhost8o80/projectname/"`,但是在重走向中使用`"/"`,这个`/`到达的是 `localhost:8080/`。也就是到达 `webapp`

- 【html 中`/`表示 web 目录，即不包括项目名】在html 中绝对路径`/`表示的也是达到 tomcat的 web下,不包括当前项目路径;

从需求角度去记忆,在可以跨域的地方/表示的就是到达 localhost:8080，如果不能踣域的地方就表示localhost: 8080/projectname，例如在 HTML、JSP 等前端语言中，其中`/`均表示：`localhost:8080`，因为`localhost`即表示主机，`8080`即对应 Tomcat，项目名即对应：`web`
