## Spring、Spring MVC、MyBatis 整合文件配置详解


使用 SSM 框架做了几个小项目了，感觉还不错是时候总结一下了。先总结一下 SSM 整合的文件配置。其实具体的用法最好还是看官方文档。

Spring:http://spring.io/docs

MyBatis:http://mybatis.github.io/mybatis-3/

基本的组织结构和用法就不说了，前面的博客和官方文档上都非常的全面。jar 包可以使用 Maven 来组织管理。来看配置文件。

### **1 ****web.xml的配置**

web.xml 应该是整个项目最重要的配置文件了，不过 servlet3.0 中已经支持注解配置方式了。在 servlet3.0 以前每个 servlet 必须要在 web.xml 中配置 servlet 及其映射关系。但是在 spring 框架中就不用了，因为 Spring 中是依赖注入（Dependency Injection）的也叫控制反转（Inversion of Control）。但是也要配置一个重要的 servlet，就是前端控制器（DispatcherServlet）。配置方式与普通的 servlet 基本相似。

配置内容如下：

```
<!-- 配置前端控制器 -->
  <servlet>
      <servlet-name>spring</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
          <!-- ContextconfigLocation配置springmvc加载的配置文件
          适配器、处理映射器等
           -->
          <param-name>contextConfigLocation</param-name>
          <param-value>WEB-INF/classes/spring/springmvc.xml</param-value>
  </init-param>
  </servlet>
  <servlet-mapping>
      <servlet-name>spring</servlet-name>
      <!-- 1、.action访问以.action结尾的  由DispatcherServlet进行解析
           2、/,所有访问都由DispatcherServlet进行解析
       -->
      <url-pattern>/</url-pattern>
  </servlet-mapping>
```

这里需要注意，springmvc.xml 是 spring 配置文件，将在后面讨论。在<servlet-mapping>中 url 如果是.action，前端控制器就只会拦截以.action 结尾的请求，并不会理会静态的文件。对静态页面的控制就要通过其他的手段。以/作为 url 的话就会拦截所有的请求，包括静态页面的请求。这样的话就可以拦截任何想要处理的请求，但是有一个问题。如果拦截了所有的请求，如果不在拦截器中做出相应的处理那么所有静态的 js、css 以及页面中用到的图片就会访问不到造成页面无法正常显示。但这可以通过静态资源的配置来解决这个问题。后面会提到。

配置 spring 容器：

```
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/classes/spring/applicationContext-*.xml</param-value>
</context-param>
```

其中 applicationContext-*.xml 包含 3 个配置文件，是 springIoC 容器的具体配置。后面会提到。

配置一个监听器：

```
<listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

web.xml 的完整配置是这样的：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0"
    xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
 
http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
 
  <display-name></display-name>    
 
  <!-- 404错误拦截 -->
  <error-page>
    <error-code>404</error-code>
    <location>/error404.jsp</location>
  </error-page>
  <!-- 500错误拦截 -->
  <error-page>
    <error-code>500</error-code>
    <location>/error500.jsp</location>
  </error-page>
 
  <!-- 加载spring容器 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/classes/spring/applicationContext-*.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
 
  <!-- 配置前端控制器 -->
  <servlet>
      <servlet-name>spring</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
          <!-- ContextconfigLocation配置springmvc加载的配置文件
          适配器、处理映射器等
           -->
          <param-name>contextConfigLocation</param-name>
          <param-value>WEB-INF/classes/spring/springmvc.xml</param-value>
  </init-param>
  </servlet>
  <servlet-mapping>
      <servlet-name>spring</servlet-name>
      <!-- 1、.action访问以.action结尾的  由DispatcherServlet进行解析
           2、/,所有访问都由DispatcherServlet进行解析
       -->
      <url-pattern>/</url-pattern>
  </servlet-mapping>
 
  <!-- 解决post乱码问题的过滤器 -->
  <filter>
      <filter-name>CharacterEncodingFilter</filter-name>
      <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
      <init-param>
          <param-name>encoding</param-name>
          <param-value>utf-8</param-value>
      </init-param>
  </filter>
  <filter-mapping>
      <filter-name>CharacterEncodingFilter</filter-name>
      <url-pattern>/*</url-pattern>
  </filter-mapping>
  <welcome-file-list>
    <welcome-file>welcome.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```

看到配置文件中多了两块内容，一个是 error page 是用来友好的处理错误的，可以使用错误代码来区别并跳转到相应的处理页面。这段配置代码最好放到最前面，在前端控制器拦截之前处理。

还有一块内容是一个解决 post 乱码问题的过滤器，拦截 post 请求并编码为 utf8。

**2 springmvc.xml的配置**

视图解析器的配置：

```
<!-- 配置视图解析器 -->
     <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
         <!-- 使用前缀和后缀 -->
         <property name="prefix" value="/"></property>
         <property name="suffix" value=".jsp"></property>
</bean>
```

在 Controller 中设置视图名的时候会自动加上前缀和后缀。

Controller 的配置

自动扫描方式，扫描包下面所有的 Controller，可以使用注解来指定访问路径。

```
<!-- 使用组件扫描的方式可以一次扫描多个Controller -->
<context:component-scan base-package="com.wxisme.ssm.controller">
```

也可以使用单个的配置方式，需要指定 Controller 的全限定名。

```
<bean name="/queryUser.action" class="com.wxisme.ssm.controller.Controller1"/>
```

配置注解的处理器适配器和处理器映射器：

```
<!-- 注解的处理器适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>
<!-- 注解的处理器映射器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
```

也可以使用下面的简化配置：

```
<!-- 配置注解的处理器映射器和处理器适配器 -->
<mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
```

配置拦截器，可以直接定义拦截所有请求，也可以自定义拦截路径。

```
<mvc:interceptors>
    <!-- 直接定义拦截所有请求 -->
    <bean class="com.wxisme.ssm.interceptor.IdentityInterceptor"></bean>
        <!-- <mvc:interceptor>
            拦截所有路径的请求   包括子路径
            <mvc:mapping path="/**"/>
            <bean class="com.wxisme.ssm.interceptor.IdentityInterceptor"></bean>
        </mvc:interceptor> -->
    </mvc:interceptors>
```

配置全局异常处理器

```
<!-- 定义全局异常处理器 -->
    <!-- 只有一个全局异常处理器起作用 -->
    <bean id="exceptionResolver" class="com.wxisme.ssm.exception.OverallExceptionResolver"></bean>
```

配置文件上传数据解析器，在上传文件时需要配置。

```
<!--配置上传文件数据解析器  -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize">
            <value>9242880</value>
        </property>
    </bean>
```

还可以配置一些自定义的参数类型，以日期类型绑定为例。

```

<!-- 自定义参数类型绑定 -->
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
     <property name="converters">
         <list>
             <!-- 日期类型绑定 -->
             <bean class="com.wxisme.ssm.controller.converter.DateConverter"/>
         </list>
     </property>
    </bean>
```

上面提到过如果在配置前端控制器时拦截了所有的请求，不做特殊处理就会导致部分静态资源无法使用。如果是这种情况就可以使用下面的配置来访问静态资源文件。

```
<mvc:resources mapping="/images/**" location="/images/" />
<mvc:resources mapping="/css/**" location="/css/" />  
<mvc:resources mapping="/js/**" location="/js/" />
<mvc:resources mapping="/imgdata/**" location="/imgdata/" />
```

也可以使用默认，但是需要在 web.xml 中配置。

```
<!-- 访问静态资源文件 -->
<!-- <mvc:default-servlet-handler/> 需要在web.xml中配置-->
```

完全可以不拦截所有路径，大可避免这个问题的发生。完整的配置大概是这样的，需要注意 xml 文件的命名空间，有时候会有影响的。

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xmlns:jdbc="http://www.springframework.org/schema/jdbc"
xmlns:jee="http://www.springframework.org/schema/jee"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context.xsd  
http://www.springframework.org/schema/mvc  
http://www.springframework.org/schema/mvc/spring-mvc.xsd  
http://www.springframework.org/schema/tx  
http://www.springframework.org/schema/tx/spring-tx.xsd 
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop.xsd">
 
     <!-- 配置视图解析器 -->
     <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
         <!-- 使用前缀和后缀 -->
         <property name="prefix" value="/"></property>
         <property name="suffix" value=".jsp"></property>
     </bean>
 
     <!-- 使用组件扫描的方式可以一次扫描多个Controller -->
     <context:component-scan base-package="com.wxisme.ssm.controller">
     </context:component-scan>
     <!-- 配置注解的处理器映射器和处理器适配器 -->
    <mvc:annotation-driven conversion-service="conversionService"></mvc:annotation-driven>
 
    <!-- 自定义参数类型绑定 -->
    <bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
     <property name="converters">
         <list>
             <!-- 日期类型绑定 -->
             <bean class="com.wxisme.ssm.controller.converter.DateConverter"/>
         </list>
     </property>
    </bean>
 
    <!-- 访问静态资源文件 -->
    <!-- <mvc:default-servlet-handler/> 需要在web.xml中配置-->
 
    <mvc:resources mapping="/images/**" location="/images/" />
    <mvc:resources mapping="/css/**" location="/css/" />  
    <mvc:resources mapping="/js/**" location="/js/" />
    <mvc:resources mapping="/imgdata/**" location="/imgdata/" />
 
    <!-- 定义拦截器 -->
    <mvc:interceptors>
    <!-- 直接定义拦截所有请求 -->
    <bean class="com.wxisme.ssm.interceptor.IdentityInterceptor"></bean>
        <!-- <mvc:interceptor>
            拦截所有路径的请求   包括子路径
            <mvc:mapping path="/**"/>
            <bean class="com.wxisme.ssm.interceptor.IdentityInterceptor"></bean>
        </mvc:interceptor> -->
    </mvc:interceptors>
 
    <!--配置上传文件数据解析器  -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="maxUploadSize">
            <value>9242880</value>
        </property>
    </bean>
 
    <!-- 定义全局异常处理器 -->
    <!-- 只有一个全局异常处理器起作用 -->
    <bean id="exceptionResolver" class="com.wxisme.ssm.exception.OverallExceptionResolver"></bean>
 
 </beans>
```

applicationContext-*.xml 的配置

applicationContext-*.xml 包括三个配置文件，分别对应数据层控制、业务逻辑 service 控制和事务的控制。

数据访问层的控制，applicationContext-dao.xml 的配置：

配置加载数据连接资源文件的配置，把数据库连接数据抽取到一个 properties 资源文件中方便管理。

配置为：

```
<!-- 加载数据库连接的资源文件 -->
<context:property-placeholder location="/WEB-INF/classes/jdbc.properties"/>
```

其中 jdbc.properties 文件的内容如下：配置内容如下：

```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/database
jdbc.username=root
jdbc.password=1234
```

配置数据库连接池，这里使用的是 dbcp，别忘了添加 jar 包！

```

<!-- 配置数据源   dbcp数据库连接池 -->
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

**3 ****Spring和MyBatis整合配置，jar包由MyBatis提供**

配置 sqlSessionFactory

```

<!-- 配置sqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 数据库连接池 -->
    <property name="dataSource" ref="dataSource"/>
    <!-- 加载Mybatis全局配置文件 -->
    <property name="configLocation" value="/WEB-INF/classes/mybatis/SqlMapConfig.xml"/>
</bean>
```

SqlMapConfig.xml 文件是 MyBatis 的配置文件，后面会提到。

配置 Mapper 扫描器，扫描 mapper 包下的所有 mapper 文件和类，要求 mapper 配置文件和类名需要一致。

```
<!-- 配置mapper扫描器 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- 扫描包路径，如果需要扫描多个包中间用半角逗号隔开 -->
    <property name="basePackage" value="com.wxisme.ssm.mapper"></property>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

整个 applicationContext-dao.xml 配置文件应该是这样的：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xmlns:jdbc="http://www.springframework.org/schema/jdbc"
xmlns:jee="http://www.springframework.org/schema/jee"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context.xsd
http://www.springframework.org/schema/mvc
http://www.springframework.org/schema/mvc/spring-mvc.xsd
http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx.xsd
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop.xsd">
 
<!-- 加载数据库连接的资源文件 -->
<context:property-placeholder location="/WEB-INF/classes/jdbc.properties"/>
 
<!-- 配置数据源   dbcp数据库连接池 -->
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driver}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
 
<!-- 配置sqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 数据库连接池 -->
    <property name="dataSource" ref="dataSource"/>
    <!-- 加载Mybatis全局配置文件 -->
    <property name="configLocation" value="/WEB-INF/classes/mybatis/SqlMapConfig.xml"/>
</bean>
 
<!-- 配置mapper扫描器 -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!-- 扫描包路径，如果需要扫描多个包中间用半角逗号隔开 -->
    <property name="basePackage" value="com.wxisme.ssm.mapper"></property>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
 
</beans>
```

业务逻辑控制，applicationContext-service.xml 的配置：

这个文件里暂时只需要定义 service 的实现类即可。

```
<!-- 定义service -->
<bean id="userService" class="com.wxisme.ssm.service.impl.UserServiceImpl"/>
```

事务控制，applicationContext-transaction.xml 的配置

配置数据源，使用 JDBC 控制类。

```

<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 配置数据源 -->
    <property name="dataSource" ref="dataSource"/>
</bean>
```

配置通知，事务控制。

```
<!-- 通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 传播行为 -->
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
 
        </tx:attributes>
    </tx:advice>
```

配置 AOP 切面

```
<!-- 配置aop  -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.wxisme.ssm.service.impl.*.*(..))"/>
    </aop:config>
```

整个事务控制的配置是这样的：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xmlns:jdbc="http://www.springframework.org/schema/jdbc"
xmlns:jee="http://www.springframework.org/schema/jee"
xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:tx="http://www.springframework.org/schema/tx"
xsi:schemaLocation="http://www.springframework.org/schema/beans 
http://www.springframework.org/schema/beans/spring-beans.xsd 
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context.xsd  
http://www.springframework.org/schema/mvc  
http://www.springframework.org/schema/mvc/spring-mvc.xsd  
http://www.springframework.org/schema/tx 
http://www.springframework.org/schema/tx/spring-tx.xsd  
http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop.xsd">
 
<!-- 事务控制  对MyBatis操作数据库  spring使用JDBC事务控制类 -->
 
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <!-- 配置数据源 -->
    <property name="dataSource" ref="dataSource"/>
</bean>
 
    <!-- 通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 传播行为 -->
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
 
        </tx:attributes>
    </tx:advice>
 
    <!-- 配置aop  -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.wxisme.ssm.service.impl.*.*(..))"/>
    </aop:config>
 
</beans>
```

**4 ****MyBatis的配置**

SqlMapConfig.xml 的配置   全局 setting 配置这里省略，数据库连接池在 spring 整合文件中已经配置，具体 setting 配置参考官方文档。

别名的定义：

```

<typeAliases>
    <!-- 批量定义别名 ，指定包名，自动扫描包中的类，别名即为类名，首字母大小写无所谓-->
    <package name="com.wxisme.ssm.po"/>
</typeAliases>
```

mapper 映射文件的配置：

```

<mappers>
    <!-- 加载映射文件 -->
    <!-- 这里也可以使用class来加载映射文件，前提是：使用mapper代理的方法，遵循规范，
    并且两个文件必须同名且在同一目录
    <mapper class="com.wxisme.mybatis0100.mapper.UserMapper"/>
    基于class加载，可以进行批量加载
    -->
    <!-- 通过扫描包的方式来进行批量加载映射文件 -->
    <package name="com.wxisme.ssm.mapper"/>
 
</mappers>
```

整个文件的配置应该是这样的：

```

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
<!-- 将数据库连接数据抽取到属性文件中方便测试 -->
<!-- <properties resource="/WEB-INF/classes/jdbc.properties"></properties> -->
<!-- 别名的定义 -->
<typeAliases>
    <!-- 批量定义别名 ，指定包名，自动扫描包中的类，别名即为类名，首字母大小写无所谓-->
    <package name="com.wxisme.ssm.po"/>
</typeAliases>
 
<!-- 数据库连接用数据库连接池 -->
 
<mappers>
    <!-- 通过扫描包的方式来进行批量加载映射文件 -->
    <package name="com.wxisme.ssm.mapper"/>
</mappers>
</configuration>
```

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<mapper namespace="com.wxisme.ssm.mapper.AlbumMapper" >
</mapper>

```

以上只是对 SSM 框架简单使用时的配置文件，如果需要深入使用或者需要理解其内部机理需要参考官方文档和其源代码。