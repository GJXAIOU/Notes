# JavaEEDay48 监听器


## 概念
- 监听你的 web 应用，监听许多信息的初始化、销毁、增加、修改、删除值等；
- 监听器用于监听一些重要事件的发生，监听器对象可以在事情发生前、发生后可以做一些必要的处理；
- 主要使用场景：实现系统的日志；
-  针对三大域对象：request/session/application 进行监听


## 类型分类
- 按监听的对象划分:
  - 用于监听应用程序环境对象( ServletContext)的事件监听器
  - 用于监听用户会话对象( HttpSession)的事件监听器
  - 用于监听请求消息对象( ServletRequest)的事件监听器

- 按监听的事件划分：
  - 用于监听域对象自身的创建和销毁的事件监听器
  - 用于监听域对象中的属性的增加和删除的事件监听器
  - 用于监听绑定到 HttpSession域中的某个对象的状态的事件监听器

## 监听使用到的类
- 监听域对象的创建和销毁
  - ServletContextListener
  - HttpSessionListener
  - ServletRequestListener

- 监听域中属性值 的增加和删除 
  - ServletContextAttributeListener
  - HttpSessionAttributeListener
  - ServletRequestAttributeListener 
  - 这个三个接口都需要实现以下方法
attributeAdded    |    setAttribute时触发
attrⅰbuteRemoved  |  removeAttribute触发
attributeReplaced   |  为已存在的键赋值新的值触发
- 监听绑定到 HttpSession域中的某个对象的状态的事件
  - HttpsessionBindingListener
  - HttpSessionActivationListener (略)



## 监听器使用步骤示例
1.自定义类实现对应的 Listener，同时重写方法；
2.在 web.xml 中配置
```xml
<listener>
  <listener-class>完整的类名</listener-class>
</listener>
```


## js 中事件监听
- 对鼠标的监听：`onclick`、`onmouseout`、`onmouseover`
- 对键盘的监听：`onkeyDown()`
- 对表单事件的监听：`onblur`、`onfocuse`、`onchange`


## 代码示例

- 以 application 域对象为例，实现 ServletContextLister
```java
/**
 * @author GJXAIOU
 * @create 2019-08-15-18:18
 */
public class ApplicationListener implements ServletContextListener {
    private long beginTime;
    private long endTime;
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        long begin = System.currentTimeMillis();
        System.out.println("contextInitialized");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        endTime = System.currentTimeMillis();
        System.out.println("contextDestroyed");
        System.out.println("系统共运行了" + (endTime - beginTime) + "毫秒");
    }
}
```
对应的 web.xml 配置
```xml
    <!--配置监听器，只需配置实现了监听器接口的类的全称，不需要配置范围，因为针对所有的监听对象-->
    <listener>
        <listener-class>gjxaiou.ApplicationListener</listener-class>
    </listener>
```

其余的域对象相似；下面接着以 request 为例
```java
/**
 * @author GJXAIOU
 * @create 2019-08-15-18:48
 */
public class requestListener implements ServletRequestListener {
    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {

    }

    // 每个请求创建的时候就会执行，可以用作计数，和过滤器功能相似；
    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {

    }
}
```
同样需要配置 XML 文件，配置同上；

- 监听 session 域对象的创建和销毁
```java
/**
 * @author GJXAIOU
 * @create 2019-08-15-18:39
 */
public class SessionListener implements HttpSessionListener {

    @Override
    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
        System.out.println("sessionCreated");

    }

    // 浏览器关闭不会执行该方法，因为session并没有销毁，重写打开浏览器不能使用上次创建的session而是创建了新的session是因为
    // sessionId丢失，只有第一个session的浏览器关闭30min之后才会执行第一个session的sessionDestroy方法
    // 或者主动调用invalidate()方法销毁session
    @Override
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
        System.out.println("sesseionDestroyed");
    }
}
```

因为 session 对象默认关闭浏览器 30min 之后才会销毁，因此可以手动销毁，在访问的页面中可以以超链接的方式，设置一个 注销的按钮(button)并将其指向销毁页面`<a href="sessionDestroy.jsp"> 注销登录，销毁session</a>`，在销毁页面(sessionDestroy.jsp)中实现销毁：
sessionDestroy.jsp
```jsp
<body>
  <%
  // 注销的时候可以主动调用销毁
  session.invalidate();
  %>  
</body>
```



- 以 application 域对象为例，监听其域对象的变化
```java
/**
 * @author GJXAIOU
 * @create 2019-08-15-18:57
 */
public class ApplicationAttributeListener implements ServletContextAttributeListener {
    // 监听application对象，其中增加属性的时候触发，对应方法：setAttributes()
    @Override
    public void attributeAdded(ServletContextAttributeEvent servletContextAttributeEvent) {

    }

    // 监听application对象，其中删除属性的时候触发，对应方法：removeAttributes()
    @Override
    public void attributeRemoved(ServletContextAttributeEvent servletContextAttributeEvent) {

    }

    // 监听application对象，其中重新赋值属性，覆盖之前数据的时候触发。
    @Override
    public void attributeReplaced(ServletContextAttributeEvent servletContextAttributeEvent) {

    }
}
```








