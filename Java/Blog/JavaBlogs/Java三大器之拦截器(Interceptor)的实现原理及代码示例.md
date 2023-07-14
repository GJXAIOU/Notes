# Java拦截器（Interceptor）的实现原理及代码示例

[原文链接](https://blog.csdn.net/reggergdsg/article/details/52962774)

## 一、Java 中的过滤器、监听器、拦截器比较
![过滤器、监听器、拦截器比较]($resource/%E8%BF%87%E6%BB%A4%E5%99%A8%E3%80%81%E7%9B%91%E5%90%AC%E5%99%A8%E3%80%81%E6%8B%A6%E6%88%AA%E5%99%A8%E6%AF%94%E8%BE%83.png)

## 二、拦截器的概念
**Java 里的拦截器是动态拦截 Action 调用的对象，它提供了一种机制可以使开发者在一个 Action 执行的前后执行一段代码，也可以在一个 Action 执行前阻止其执行，同时也提供了一种可以提取Action 中可重用部分代码的方式。在 AOP 中，拦截器用于在某个方法或者字段被访问之前，进行拦截然后再之前或者之后加入某些操作**。目前，我们需要掌握的主要是 Spring 的拦截器，Struts2的拦截器不用深究，知道即可。

## 二、拦截器的原理
 大部分时候，拦截器方法都是通过代理的方式来调用的。Struts2 的拦截器实现相对简单。当请求到达 Struts2 的 ServletDispatcher 时，Struts2 会查找配置文件，并根据配置实例化相对的拦截器对象，然后串成一个列表（List），最后一个一个的调用列表中的拦截器。Struts2 的拦截器是可
插拔的，拦截器是 AOP 的一个实现。Struts2 拦截器栈就是将拦截器按一定的顺序连接成一条链。在访问被拦截的方法或者字段时，Struts2 拦截器链中的拦截器就会按照之前定义的顺序进行调用。

## 三、自定义拦截器的步骤
* 第一步：自定义一个实现了 Interceptor 接口的类，或者继承抽象类 AbstractInterceptor。
* 第二步：在配置文件中注册定义的拦截器。
* 第三步：在需要使用 Action 中引用上述定义的拦截器，为了方便也可以将拦截器定义为默认的拦截器，这样在不加特殊说明的情况下，所有的 Action 都被这个拦截器拦截。

## 四、过滤器与拦截器的区别
**过滤器可以简单的理解为“取你所想取”，过滤器关注的是web请求；拦截器可以简单的理解为“拒你所想拒”，拦截器关注的是方法调用，比如拦截敏感词汇。**
- 拦截器是基于Java反射机制来实现的，而过滤器是基于函数回调来实现的。（有人说，拦截器是基于动态代理来实现的）
- 拦截器不依赖servlet容器，过滤器依赖于servlet容器。
- 拦截器只对Action起作用，过滤器可以对所有请求起作用。
- 拦截器可以访问Action上下文和值栈中的对象，过滤器不能。
- 在Action的生命周期中，拦截器可以多次调用，而过滤器只能在容器初始化时调用一次。

## 五、Spring拦截器

- 抽象类HandlerInterceptorAdapter
我们如果在项目中使用了Spring框架，那么，我们可以直接继承`HandlerInterceptorAdapter.Java` 这个抽象类，来实现我们自己的拦截器。

Spring 框架，对 Java 的拦截器概念进行了包装，这一点和 Struts2 很类似。HandlerInterceptorAdapte 继承了抽象接口 HandlerInterceptor。

```
package org.springframework.web.servlet.handler;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
public abstract class HandlerInterceptorAdapter implements HandlerInterceptor{
    // 在业务处理器处理请求之前被调用
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
        return true;
    }
    // 在业务处理器处理请求完成之后，生成视图之前执行
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
      throws Exception{
    }
    // 在DispatcherServlet完全处理完请求之后被调用，可用于清理资源
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
      throws Exception{
    }
}
```

   接下来我们看一下Spring框架实现的一个简单的拦截器UserRoleAuthorizationInterceptor，UserRoleAuthorizationInterceptor继承了抽象类HandlerInterceptorAdapter，实现了用户登录认证的拦截功能，如果当前用户没有通过认证，会报403错误。

```
package org.springframework.web.servlet.handler;
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
public class UserRoleAuthorizationInterceptor extends HandlerInterceptorAdapter{
    // 字符串数组，用来存放用户角色信息
    private String[] authorizedRoles;
    public final void setAuthorizedRoles(String[] authorizedRoles){
        this.authorizedRoles = authorizedRoles;
    }
    public final boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
      throws ServletException, IOException{
        if (this.authorizedRoles != null) {
            for (int i = 0; i < this.authorizedRoles.length; ++i) {
                if (request.isUserInRole(this.authorizedRoles[i])) {
                    return true;
                }
            }
        }
        handleNotAuthorized(request, response, handler);
        return false;
    }
    protected void handleNotAuthorized(HttpServletRequest request, HttpServletResponse response, Object handler)
      throws ServletException, IOException{
          // 403表示资源不可用。服务器理解用户的请求，但是拒绝处理它，通常是由于权限的问题
          response.sendError(403);
    }
}
```

   下面，我们利用Spring框架提供的HandlerInterceptorAdapter抽过类，来实现一个自定义的拦截器。我们这个拦截器叫做UserLoginInterceptorBySpring，进行登录拦截控制。工作流程是这样的：如果当前用户没有登录，则跳转到登录页面；登录成功后，跳转到
之前访问的URL页面。

```
import java.util.HashMap;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.HandlerInterceptorAdapter;
/**
 * @description 利用spring框架提供的HandlerInterceptorAdapter，实现自定义拦截器
 */
public class UserLoginInterceptorBySpring extends HandlerInterceptorAdapter{
    // 在业务处理器处理请求之前被调用
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception{
        // equalsIgnoreCase 与 equals的区别？
        if("GET".equalsIgnoreCase(request.getMethod())){
            //RequestUtil.saveRequest();
        }
        System.out.println("preHandle...");
        String requestUri = request.getRequestURI();
        String contextPath = request.getContextPath();
        String url = requestUri.substring(contextPath.length());
        System.out.println("requestUri" + requestUri);
        System.out.println("contextPath" + contextPath);
        System.out.println("url" + url);
        String username = (String) request.getSession().getAttribute("username");
        if(null == username){
            // 跳转到登录页面
            request.getRequestDispatcher("/WEB-INF/login.jsp").forward(request, response);
            return false;
        }
        else{
            return true;
        }
    }
    // 在业务处理器处理请求完成之后，生成视图之前执行
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception{
        System.out.println("postHandle...");
        if(modelAndView != null){
            Map<String, String> map = new HashMap<String, String>();
            modelAndView.addAllObjects(map);
        }
    }
    // 在DispatcherServlet完全处理完请求之后被调用，可用于清理资源
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception{
        System.out.println("afterCompletion...");
    }
}
```

  拦截器是依赖Java反射机制来实现的。拦截器的实现，用到的是JDK实现的动态代理，我们都知道，JDK实现的动态代理，需要依赖接口。拦截器
是在面向切面编程中应用的，就是在你的service或者一个方法前调用一个方法，或者在方法后调用一个方法。拦截器不是在web.xml，比如struts在
struts.xml中配置。

```
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {  
    Object result = null;  
    System.out.println("方法调用前，可以执行一段代码" + method.getName());  
    result = method.invoke(this.targetObj, args);  
    System.out.println("方法调用后，可以执行一段代码 " + method.getName());  
    return result;  
}  
```

## 六、总结

- 过滤器（Filter）：所谓过滤器顾名思义是用来过滤的，Java的过滤器能够为我们提供系统级别的过滤，也就是说，能过滤所有的web请求，这一点，是拦截器无法做到的。在Java Web中，你传入的request,response提前过滤掉一些信息，或者提前设置一些参数，然后再传入servlet或者struts的action进行业务逻辑，比如过滤掉非法url（不是login.do的地址请求，如果用户没有登陆都过滤掉）,或者在传入servlet或者struts的action前统一设置字符集，或者去除掉一些非法字符（聊天室经常用到的，一些骂人的话）。filter 流程是线性的，url传来之后，检查之后，可保持原来的流程继续向下执行，被下一个filter, servlet接收。

- 监听器（Listener）：Java的监听器，也是系统级别的监听。监听器随web应用的启动而启动。Java的监听器在c/s模式里面经常用到，它会对特定的事件产生产生一个处理。监听在很多模式下用到，比如说观察者模式，就是一个使用监听器来实现的，在比如统计网站的在线人数。又比如struts2可以用监听来启动。Servlet监听器用于监听一些重要事件的发生，监听器对象可以在事情发生前、发生后可以做一些必要的处理。

- 拦截器（Interceptor）：Java里的拦截器提供的是非系统级别的拦截，也就是说，就覆盖面来说，拦截器不如过滤器强大，但是更有针对性。
Java中的拦截器是基于Java反射机制实现的，更准确的划分，应该是基于JDK实现的动态代理。它依赖于具体的接口，在运行期间动态生成字节码。
拦截器是动态拦截Action调用的对象，它提供了一种机制可以使开发者在一个Action执行的前后执行一段代码，也可以在一个Action执行前阻止其执行，同时也提供了一种可以提取Action中可重用部分代码的方式。在AOP中，拦截器用于在某个方法或者字段被访问之前，进行拦截然后再之前或
者之后加入某些操作。Java的拦截器主要是用在插件上，扩展件上比如 Hibernate Spring Struts2等，有点类似面向切片的技术，在用之前先要在配置文件即xml，文件里声明一段的那个东西。