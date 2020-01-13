## SpringBoot 第二章

- 整合 Servlet 
- 整合 Filter 
- 整合 Listener
- 访问静态资源
- 文件上传

## 一、整合 Servlet

### （一）通过注解扫描完成Servlet 组件的注册

- 编写 servlet

    ```java
    package com.gjxaiou.servlet;
    
    import javax.servlet.ServletException;
    import javax.servlet.annotation.WebServlet;
    import javax.servlet.http.HttpServlet;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import java.io.IOException;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/6 20:51
     * 
     * SpringBoot 整合 Servlet 方式一：通过注解扫描完成 Servlet 组件的注册
     */
    
    
    /**
     * 之前 Spring 整合的时候需要在 web.xml 中配置以下信息
     * <servlet>
     *   <servlet-name>FirstServlet</servlet-name>
     *   <servlet-class>com.gjxaiou.servlet.FirstServlet</servlet-class>
     * </servlet>
     * <servlet-mapping>
     *   <servlet-name>FirstServlet</servlet-name>
     *   <url-pattern>/first</url-pattern>
     * </servlet-mapping>
     */
    
    // 在 springboot 中只要在类上声明即可，无需在 web.xml 中进行配置
    @WebServlet(name = "FirstServlet", urlPatterns = "/first")
    public class FirstServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            // 通过访问 http://localhost:8080/first 然后看控制台输出判断是否执行
            System.out.println("FirstServlet");
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            super.doPost(req, resp);
        }
    }
    
    
    ```

    

- 编写启动类

    ```java
    package com.gjxaiou;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.ServletComponentScan;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/6 21:01
     */
    @SpringBootApplication
    // 在 SpringBoot 启动时候会扫描 @WebServlet 注解，并将该类实例化
    @ServletComponentScan
    public class Start {
        public static void main(String[] args) {
            SpringApplication.run(Start.class, args);
        }
    }
    
    ```

    

### （二）通过方法完成Servlet 组件的注册

- 编写 servlet

    ```java
    package com.gjxaiou;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.ServletComponentScan;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/6 21:01
     */
    @SpringBootApplication
    // 在 SpringBoot 启动时候会扫描 @WebServlet 注解，并将该类实例化
    @ServletComponentScan
    public class Start {
        public static void main(String[] args) {
            SpringApplication.run(Start.class, args);
        }
    }
    ```

    

- 编写启动类

    ```java
    package com.gjxaiou;
    
    import com.gjxaiou.servlet.SecondServlet;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.ServletRegistrationBean;
    import org.springframework.context.annotation.Bean;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/6 21:01
     */
    @SpringBootApplication
    public class Start2 {
        public static void main(String[] args) {
            SpringApplication.run(Start2.class, args);
        }
    
        // 通过 ServletRegistrationBean 方法来注册 Servlet
        // 该方法返回值固定，方法名随便
        @Bean
        public ServletRegistrationBean getServletRegistrationBean() {
            ServletRegistrationBean<SecondServlet> secondServletServletRegistrationBean =
                    new ServletRegistrationBean<>(new SecondServlet());
            // 配置 UrlMapping
            secondServletServletRegistrationBean.addUrlMappings("/second");
            return secondServletServletRegistrationBean;
        }
    }
    
    ```

    

    ## 二、整合 Filter

    ### （一）通过注解扫描完成Filter 组件的注册

- 编写 Filter

    ```java
    package com.gjxaiou.filter;
    
    import javax.servlet.*;
    import javax.servlet.annotation.WebFilter;
    import java.io.IOException;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 21:08
     * <p>
     * SpringBoot 整合 Filter 方式一
     * 之前通过 web.xml 配置 Filter ：同时在下面 xml 中指定拦截 /first 路径下的 Servlet
     * <filter>
     *      <filter-name>FirstFilter</filter-name>
     *      <filter-class>com.gjxaiou.filter.FirstFilter</filter-class>
     * </filter>
     * <filter-mapping>
     *      <filter-name>FirstFilter</filter-name>
     *      <url-pattern>/first</url-pattern>
     * </filter-mapping>
     */
    @WebFilter(filterName = "FirstFilter", urlPatterns = "/first")
    public class FirstFilter implements Filter {
    
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
    
        }
    
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
                             FilterChain filterChain) throws IOException, ServletException {
            System.out.println("进入 filter");
            filterChain.doFilter(servletRequest, servletResponse);
            System.out.println("离开 Filter");
        }
    
        @Override
        public void destroy() {
    
        }
    }
    
    ```

- 编写启动类

    ```java
    package com.gjxaiou;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.ServletComponentScan;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 21:20
     */
    @SpringBootApplication
    @ServletComponentScan
    public class FilterStart {
        public static void main(String[] args) {
            SpringApplication.run(FilterStart.class, args);
        }
    }
    
    ```

    通过访问 `http://localhost:8080/first` ，控制台输出：

    ```java
    进入 filter
    FirstServlet
    离开 Filter
    ```

    

    ### （二）通过方法完成Filter 组件的注册

- 编写 Filter

    ```java
    package com.gjxaiou.filter;
    
    import javax.servlet.*;
    import java.io.IOException;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 21:28
     * SpringBoot 注册 Filter 方式二:
     */
    public class SecondFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
    
        }
    
        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            System.out.println("进入 filter");
            filterChain.doFilter(servletRequest, servletResponse);
            System.out.println("离开 Filter");
        }
    
        @Override
        public void destroy() {
    
        }
    }
    
    ```

- 编写启动类

    ```java
    package com.gjxaiou;
    
import com.gjxaiou.filter.SecondFilter;
    import com.gjxaiou.servlet.SecondServlet;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.FilterRegistrationBean;
    import org.springframework.boot.web.servlet.ServletRegistrationBean;
    import org.springframework.context.annotation.Bean;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 21:48
     */
    @SpringBootApplication
    public class FilterStart2 {
        public static void main(String[] args) {
            SpringApplication.run(FilterStart2.class, args);
        }
        // 通过 ServletRegistrationBean 方法来注册 Servlet
        // 该方法返回值固定，方法名随便
        @Bean
        public ServletRegistrationBean getServletRegistrationBean2() {
            ServletRegistrationBean<SecondServlet> secondServletServletRegistrationBean =
                    new ServletRegistrationBean<>(new SecondServlet());
            // 配置 UrlMapping
            secondServletServletRegistrationBean.addUrlMappings("/second");
            return secondServletServletRegistrationBean;
        }
    
        @Bean
        public FilterRegistrationBean getFilterRegistrationBean(){
            FilterRegistrationBean<SecondFilter> secondFilterFilterRegistrationBean =
                    new FilterRegistrationBean<>(new SecondFilter());
            // 可以拦截多个 URL
            //secondFilterFilterRegistrationBean.addUrlPatterns(new String[]{"*.do", "*.jsp"});
            secondFilterFilterRegistrationBean.addUrlPatterns("/second");
            return secondFilterFilterRegistrationBean;
        }
    }
    
    
    ```
    
    程序执行结果：
    
    ```java
    进入 filter
    SecondServlet
    离开 Filter
    ```
    
    

## 三、整合 Listener

### （一）通过注解扫描完成 Listener 组件的注册

- 编写 Listener

    ```java
    package com.gjxaiou.listener;
    
    import javax.servlet.ServletContextEvent;
    import javax.servlet.ServletContextListener;
    import javax.servlet.annotation.WebListener;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 21:58
     * <p>
     * SpringBoot 整合 Listener 方式一：
     * 之前在 web.xml 中配置方式为：
     * <listener>
     *      <listener-class>com.gjxaiou.listener.FirstListener</listener-class>
     * </listener>
     */
    
    // 具体实现什么接口根据 Listener 作用而定
    @WebListener
    public class FirstListener implements ServletContextListener {
        @Override
        public void contextInitialized(ServletContextEvent sce) {
            System.out.println("Listener ...init....");
        }
    
        @Override
        public void contextDestroyed(ServletContextEvent sce) {
        }
    }
    
    ```

- 编写启动类

    ```java
    package com.gjxaiou;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.ServletComponentScan;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 22:04
     */
    @SpringBootApplication
    @ServletComponentScan
    public class ListenerStart {
        public static void main(String[] args) {
            SpringApplication.run(ListenerStart.class, args);
        }
    }
    
    ```
    
    在程序运行过程中会在控制台输出：`Listener ...init....`



### （二）通过方法完成 Listener 组件注册

- 编写 Listener

    ```java
    package com.gjxaiou.listener;
    
    import javax.servlet.ServletContextEvent;
    import javax.servlet.ServletContextListener;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 22:07
     */
    public class SecondListener implements ServletContextListener {
        @Override
        public void contextInitialized(ServletContextEvent sce) {
            System.out.println("SecondListener ...init....");
        }
    
        @Override
        public void contextDestroyed(ServletContextEvent sce) {
        }
    }
    
    ```

- 编写启动类

    ```java
    package com.gjxaiou;
    
import com.gjxaiou.listener.SecondListener;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.web.servlet.ServletListenerRegistrationBean;
    import org.springframework.context.annotation.Bean;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 22:08
     */
    @SpringBootApplication
    public class ListenerStart2 {
        public static void main(String[] args) {
            SpringApplication.run(ListenerStart2.class, args);
        }
    
        /**
         * 注册 Listener
         */
        @Bean
        public ServletListenerRegistrationBean<SecondListener> getServletListenerRegistrationBean() {
            ServletListenerRegistrationBean<SecondListener> bean =
                    new ServletListenerRegistrationBean<SecondListener>(new SecondListener());
            return bean;
        }
    
    }
    
    ```
    
    

## 四、访问静态资源

springBoot 默认从以下位置查找静态资源：

- SpringBoot 从 classpath/static 的目录（resources 下面建立 static 文件夹），下面可以建立自己的子文件夹
    注意目录名称必须是 static
- ServletContext 根目录下（对于 Maven 项目就是 src/main/webapp 目录）
    在 src/main/webapp 目录名称必须要 webapp

## 五、文件上传

- 提供一个上传页面便于上传（src/main/resources/static/upload.html）

    ```java
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>文件上传</title>
    </head>
    <body>
    <form action="fileUploadController" method="post" enctype="multipart/form-data">
        请点击上传文件：<input type="file" name="filename"/> <br>
        <input type="submit">
    </form>
    
    </body>
    </html>
    ```

    

- 编写 Controller

    ```java
    package com.gjxaiou.controller;
    
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.multipart.MultipartFile;
    
    import java.io.File;
    import java.io.IOException;
    import java.util.HashMap;
    import java.util.Map;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 22:23
     */
    // 表示对该类下面所有方法的返回值做 JSON 格式转换，约等于 @Controller + @ResponseBody
    @RestController
    public class FileUploadController {
        /**
         * 处理文件上传
         */
        @RequestMapping("/fileUploadController")
        public Map<String,Object> fileUpload(MultipartFile filename) throws IOException {
            System.out.println("上传文件名为：" + filename.getOriginalFilename());
            // 将文件保存到 XXX
            filename.transferTo(new File("d:/" + filename.getOriginalFilename()));
            Map<String, Object> map = new HashMap<>();
            map.put("msg","ok");
            return map;
        }
    }
    
    ```

- 编写启动类

    ```java
    package com.gjxaiou;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    /**
     * @Author GJXAIOU
     * @Date 2020/1/7 22:35
     */
    @SpringBootApplication
    public class FileStart {
        public static void main(String[] args) {
            SpringApplication.run(FileStart.class,args);
        }
    }
    
    ```

    访问：`http://localhost:8080/upload.html`，点击上传一个文件，提交之后，页面上显示 ok，表示提交成功，同时 D 盘根目录下有一个同名文件。

    

- 设置上传文件大小的默认值
    
    - 需要添加一个 springBoot 的配置文件： `application.properties`
    - 设置单个上传文件的大小：`spring.http.multipart.maxFileSize=200MB`
    - 设置一次请求上传文件的总容量： `spring.http.multipart.maxRequestSize=200MB`