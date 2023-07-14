# 五、Spring Boot 与安全

## 一、安全

一般使用的安全框架：shiro 或者 Spring Security。

Spring Security 是针对 Spring 项目的安全框架，也是 Spring Boot 底层安全模块默认的技术选型。他可以实现强大的 web 安全控制。对于安全控制，我们仅需引入 spring-boot-starter-security 模块，进行少量的配置，即可实现强大的安全管理。

 主要的几个类：

- `WebSecurityConfigurerAdapter`：自定义Security策略

- `AuthenticationManagerBuilder`：自定义认证策略

- `@EnableWebSecurity`：开启WebSecurity模式



应用程序的两个主要区域是“认证”和“授权”（或者访问控制）。这两个主要区域是 Spring Security 的两个目标。

- 「认证」（Authentication），是**建立**一个他声明的主体的过程（一个“主体”一般是指用户，设备或一些可以在你的应用程序中执行动作的其他系统）。

- 「授权」（Authorization）指确定一个主体是否**允许**在你的应用程序执行一个动作的过程。为了抵达需要授权的店，主体的身份已经有认证过程建立。

这个概念是通用的而不只在 Spring Security 中。

## 二、Web&安全 

1.登陆/注销

–HttpSecurity配置登陆、注销功能

2.Thymeleaf提供的SpringSecurity标签支持

–需要引入thymeleaf-extras-springsecurity4

–sec:authentication=“name”获得当前用户的用户名

–sec:authorize=“hasRole(‘ADMIN’)”当前用户必须拥有ADMIN权限时才会显示标签内容

3.remember me

–表单添加remember-me的checkbox

–配置启用remember-me功能

4.CSRF（Cross-site request forgery）跨站请求伪造

–HttpSecurity启用csrf功能，会为表单添加_csrf的值，提交携带来预防CSRF；





### 使用步骤

- 步骤一：引入 SpringSecurity 依赖

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    ```

- 步骤二：编写SpringSecurity的配置类；

    该类需要继承 `WebSecurityConfigurerAdapter` 并且标注：`@EnableWebSecurity`

- 步骤三：控制请求的访问权限：

    通过重写 `configure()` 方法

    ```java
    configure(HttpSecurity http) {
        http.authorizeRequests().antMatchers("/").permitAll()
            .antMatchers("/level1/**").hasRole("VIP1")
    }
    ```

- 步骤四：定义认证规则：
        configure(AuthenticationManagerBuilder auth){
          auth.inMemoryAuthentication()
           .withUser("zhangsan").password("123456").roles("VIP1","VIP2")
        }
     5、开启自动配置的登陆功能：
        configure(HttpSecurity http){
          http.formLogin();
        }
     6、注销：http.logout();
     7、记住我：Remeberme()；*