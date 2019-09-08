# Frame02_01 Spring


**主要内容**
Spring 框架简介及官方压缩包目录介绍
Spring 环境搭建
IoC 详解
Spring 创建 Bean 的三种方式(包含两种工厂方式)
scope 属性讲解(包含单例设计模式)
DI 详解
Spring 中几种注入方式
利用 Spring DI 实现生成 SqlSessionFactory 对象



## 一、Spring 框架简介及官方压缩包目录介绍

- Spring 框架宗旨:不重新发明技术,让原有技术使用起来更加方便.
- **Spring 几大核心功能**
  - IoC/DI：控制反转/依赖注入；
  - AOP：面向切面编程；
  - 声明式事务；

- Spring Framework runtime （系统图见下）
  -  test： spring 提供的测试功能；
  - Core  Container：是核心容器，里面的内容是 Spring 启动最基本的条件；
    - Beans: **Spring中负责创建类对象并管理对象**；
    - Core: 核心类；
    - Context: 上下文参数，**用于获取外部资源或者管理注解等**；
    - SpEl: 对应于 `expression.jar`
  - AOP: 实现 aop 功能需要的依赖
  - Aspects: 切面 AOP 依赖的包
  - Data  Access/Integration：**Spring 封装数据访问层相关内容**
    - JDBC：Spring 对 JDBC 封装后的代码；
    - ORM: 封装了持久层框架的代码，例如 Hibernate
    - transactions：对应 `spring-tx.jar`,声明式事务时使用；
  - WEB：**需要 Spring 完成 web 相关功能时需要**；
    - 例如：由 tomcat 加载 spring 配置文件时需要有 spring-web包

Servlet是具体的业务功能，不能封装

![Spring Framework Runtime]($resource/Spring%20Framework%20Runtime.png)

   - Spring 框架中重要概念
      - 容器(Container): 将 Spring 当作一个大容器.
      - 老版本中的使用 BeanFactory 接口，**新版本中是 ApplicationContext 接口, 是 BeanFactory 子接口，BeanFactory 的功能在 ApplicationContext 中都有**；

- 从 Spring3 开始把 Spring 框架的功能拆分成多个 jar，Spring2 及以前就一个 jar

## 二、IoC：控制反转（Inversion of Control）

- IoC 完成的事情：**原先由程序员主动通过 new来 实例化对象事情，现在转交给 Spring 负责**。
- 控制反转中控制指的是：控制类的对象；
- 控制反转中反转指的是：转交给 Spring 负责；
- **IoC 最大的作用：解耦**
即程序员不需要管理对象，解除了对象管理和程序员之间的耦合。

## 三、Spring 环境搭建

- 导入 jar，包括四个核心包一个日志包(commons-logging)
```mulu
commons-logging.jar
spring-beans.jar
spring-context.jar
spring-core.jar
spring-expression.jar
```

- 在 src 下新建 `applicationContext.xml` 文件
  - 上面的文件名称和路径可以自定义，这么起名称是为了记住 Spring 容器 ApplicationContext,
而 `applicationContext.xml` 中配置的信息最终存储到了 AppliationContext 容器中。

-  spring 配置文件（applicationContext.xml）是基于 schema
  -  schema 文件扩展名 `.xsd`
  - 把 schema 理解成 DTD 的升级版.（DTD是XML的语法检查器）
  - schema 比 DTD 具备更好的扩展性.拓展性体现如下：
    - 每次引入一个 xsd 文件是一个 namespace(即xmlns)
  - 配置文件中一般只需要引入基本 schema
  配置文件的基本内容为：
```java
<?xml version="1.0" encoding="UTF-8"?> <beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
 http://www.springframework.org/schema/beans/spring-beans.xsd">   

 
 </beans>
```
**使用：**
- 首先需要一个 `People.java` 的实体类；
- 然后新建配置文件：`applicationContext.xml`
  - schema 通过 `<bean/>` 创建对象；
  - 默认配置文件被加载时就创建对象；
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- id表示获取到对象标识,是为了以后获取使用这个对象
    class 表示创建哪个类的对象-->
    
    <bean id="peo" class="com.bjsxt.pojo.People"/>
</beans>
```
 
- 然后编写测试方法:Test.java，在这里可以创建对象
  - `getBean(“<bean>标签 id 值”,返回值类型);`如果没有第二个参数, 默认是 Object
  -  `getBeanDefinitionNames()`,是Spring 容器中目前管理的所有对象。
```java
public interface Test {
   public static void main(String[] args) {
      // 这里配置文件加载的时候类就被创建了
  ApplicationContext applicationContext =  new ClassPathXmlApplicationContext("applicationContext.xml");
  // 使用 getBean（）创建对象
  People people = applicationContext.getBean("peo",People.class);
  System.out.println(people);   
  
  // 查看当前 spring 容器中管理的所有类对象
  String[] names = applicationContext.getBeanDefinitionNames();
  for (String string : names) {
         System.out.println(string);
    }
  }
}
```


## 四、Spring 创建对象的三种方式

首先提供一个实体类：
```com_gjxaiou_pojo_People_java
public class People {
    private int age;
    private String name;
 // 省略 set 和 get 方法
```

### 方案一：通过构造方法创建
- 无参构造创建:默认情况
- 有参构造创建:需要明确配置
  - 首先需要在类中提供有参构造方法
  - 然后在 applicationContext.xml 中设置调用哪个构造方法创建对象
    - 如果设定的条件匹配多个构造方法执行最后的构造方法，一般index、name、type有可以唯一确定对象即可，也可以多个结合使用来唯一确定
      - index : 参数的索引,从 0 开始
      - name: 参数名
      - type：类型(区分开关键字和封装类如 int 和 Integer)

### 方案二：通过实例工厂
spring只支持两种工厂：实例工厂和静态工厂

- **工厂设计模式：就是帮助创建类对象**，一个工厂可以生产多个对象；
-  **实例工厂：需要先创建工厂,才能生产对象**

**实现步骤:**
- 必须要有一个实例工厂
```java
public class PeopleFactory {
    public People newInstance(){
        return new People(1,"测试");
    }
}    
```
以前常规的使用工厂创建对象方式：
```Test_java
PeopleFactory factory = new PeopleFactory();
People people = factory.newInstance();
```
下面是对应的在spring中使用的方式：（factory-bean 对应于 id），即在 applicationContext.xml 中配置
```applicationContext_java
// 创建一个工厂
<bean id="factory" class="com.bjsxt.pojo.PeopleFactory"></bean>
// 创建一个 People 对象
<bean id="peo1" factory-bean="factory" factory-method = "newInstance"></bean>
```

### 方案三：通过静态工厂

- 使用静态工厂不需要创建工厂，可以快速创建对象；

**实现步骤**
- 首先编写一个静态工厂(在方法上添加 static)
```staticPeopleFactory_java
public class StaticPeopleFactory {
    public static People newInstance() {
        return new People(1,"lisi");
  }
}
```

 -  然后在 applicationContext.xml 中
```applicationContext_xml
// 直接创建一个对象，不需要先创建工厂，只需要指明工厂类，直接使用工厂中的方法
<bean id="peo2" class="com.bjsxt.pojo.PeopleFactory" factory-method="newInstance"> </bean>
```


## 五、如何给 Bean 的属性赋值(注入)
**就是给对象的属性进行赋值**
- 方法一： 通过构造方法设置属性值；
- 方法二：设置注入(本质上通过对象的 get 和 set 方法，因此一定要生成 get 和 set 方法)
首先基本的数据类如下：
```java
public class People{
    private int id;
    private String name;
    private Set<String> strs;
    private List<String> list;
    private String[] strs;
    private Map<String, String> map;
    // 省略对应的 set 和 get 方法
}
```
  -  如果属性是基本数据类型或 String 等简单的，首先创建People类对象，下面是给对象的属性进行赋值
```java
<bean id="peo" class="com.bjsxt.pojo.People">
    <property name="id" value="222"></property>
    <property name="name" value="张三"></property>
</bean>
```
上面代码等效于：
```java
<bean id="peo" class="com.bjsxt.pojo.People">
    <property name="id">
        <value>456</value>
    </property>
    <property name="name">
        <value>zhangsan</value>
    </property>
</bean>
```

  - 如果属性是 `Set<?>`
这里 set 里面存放的基本数据类型，如果存放的是对象，则需要将 <value> 标签更换为  <ref> 标签，中间的值设置为对象即可；
```xml
<property name="sets">
   <set>
     // 这里存放的类型虽然是 String，但是值不能加 ""
      <value>1</value>
      <value>2</value>
      <value>3</value>
      <value>4</value>
   </set>
</property>
```

- 如果属性是 `List<?>`
```xml
<property name="list">
  <list>
    <value>1</value>
    <value>2</value>
    <value>3</value>
  </list>
</property>
```

- 如果 list 中就只有一个值，直接在里面使用 value 即可；
```xml
<property name="list" value="1">
</property>
```

- 如果属性是数组
当然如果数组中就只有一个值,可以直接通过 value 属性赋值
```xml
<property name="strs" >
  <array>
      <value>1</value>
      <value>2</value>
      <value>3</value>
  </array>
</property>
```

- 如果属性是 `map<key, value>`
```xml
<property name="map">
  <map>
    <entry key="a" value="b" >
    </entry>
    <entry key="c" value="d" >
    </entry>
  </map>
</property>
```

- 如果属性是 Properties 类型，下面代码作用是对应于从 XXX.properties文件中取值
```xml
<property name="demo">
  <props>
    <prop key="key">value</prop>
    <prop key="key1">value1</prop>
  </props>
</property>
```


## 六、DI（Dependency Injection）：依赖注入

- DI 和 IoC 是一样的，当一个类(A)中需要依赖另一个类(B)对象时,把 B 赋值给 A 的过程就叫做依赖注入。
解释：就是在A中的属性是B类对象，现在要将A类中的B属性赋值； 
```java
public class  A{
    int id; 
    string name; 
    private B b;
}
public class B{ 
    int score;
}
```

代码体现：
```xml
// 先对 A 对象进行实例化
<bean id="a" class="com.bjsxt.pojo.A">
    <property name="b" ref="b"></property> </bean>
    // 然后对 A 类中的 B 对象进行实例化
    <bean id="b" class="com.bjsxt.pojo.B">
      <property name="id" value="1"></property>
      <property name="price" value="12"></property>
</bean>
```

## 七、使用 Spring 简化 MyBatis

- 需要导入 mybatis 所 有 jar 和 spring 基 本包，以及 spring-jdbc,spring-tx,spring-aop,spring-web,spring 整合 mybatis 的包等；
```lib
asm-3.3.1.jar
cglib-2.2.2.jar
commons-logging-1.1.1.jar
commons-logging-1.1.3.jar
javassist-3.17.1-GA.jar
jstl-1.2.jar
LIST.TXT
log4j-1.2.17.jar
log4j-api-2.0-rc1.jar
log4j-core-2.0-rc1.jar
mybatis-3.2.7.jar
mybatis-spring-1.2.3.jar
mysql-connector-java-5.1.30.jar
slf4j-api-1.7.5.jar
slf4j-log4j12-1.7.5.jar
spring-aop-4.1.6.RELEASE.jar
spring-beans-4.1.6.RELEASE.jar
spring-context-4.1.6.RELEASE.jar
spring-core-4.1.6.RELEASE.jar
spring-expression-4.1.6.RELEASE.jar
spring-jdbc-4.1.6.RELEASE.jar
spring-tx-4.1.6.RELEASE.jar
spring-web-4.1.6.RELEASE.jar
standard-1.1.2.jar
```

- 先配置 web.xml
这里为了让 Tomcat 在启动时候自动加载 Spring 的配置文件，需要在 web.xml 中告诉 Tomcat 怎么加载；
```xml
<?xml version="1.0" encoding="UTF-8"?>
  <web-app version="3.0"
      xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
      
   <!-- 上下文参数-->
   <context-param>
      <param-name>contextConfigLocation</param-name>
      <!-- spring 配置文件-->
      <param-value>classpath:applicationContext.xml</param-value>
   </context-param>
<!-- 内部封装了一个监听器,可以将帮助加载Spring 的配置文件 -->

<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener          </listener-class>
</listener>
</web-app>
```

- 然后编写 spring 配置文件，对应于实现 myBatis.xml 配置文件中的内容；
```applicationContext_xml
<?xml version="1.0" encoding="UTF-8"?>
<beans
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/sc hema/beans
    http://www.springframework.org/schema/beans/spring-be ans.xsd">

    <!-- 数据源封装类 .数据源:获取数据库连接,spring-jdbc.jar 中(类名知道即可)，代替类MyBatis中的dataSource配置功能-->
<bean id="dataSouce" class="org.springframework.jdbc.datasource.DriverMana gerDataSource">
  <property name="driverClassName" value="com.mysql.jdbc.Driver"> </property>
  <property name="url" value="jdbc:mysql://localhost:3306/ssm"> </property>
  <property name="username" value="root"></property>
  <property name="password" value="smallming"></property>
</bean>

  <!-- 创建SqlSessionFactory 对象-->
  <!--这个类是专门在 Spring 中生成 sqlSessionFactory 对象的类-->
  <bean id="factory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- 数据库连接信息来源于dataSource -->
  <property name="dataSource" ref="dataSouce"></property>
  </bean>
  
  <!-- 扫描器相当于mybatis.xml 中mappers 下package 标签,扫描com.bjsxt.mapper 包后会给对应接口创建对象-->
  <bean
      class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <!-- 要扫描哪个包-->
      <property name="basePackage" value="com.bjsxt.mapper"></property>
  <!-- 和factory 产生关系-->
      <property name="sqlSessionFactory" ref="factory"></property>
  </bean>

  <!-- 由spring 管理service 实现类-->
  <bean id="airportService"
      class="com.bjsxt.service.impl.AirportServiceImpl">
      <property name="airportMapper" ref="airportMapper"></property>
  </bean>
</beans>

```

- 编写代码
  - 首先正常编写：pojo
  - 编写 mapper 包下时，**必须使用接口绑定方案或注解方案(必须有接口)**
  - 正常编写 Service 接口和 Service 实现类
    - 需要在 Service 实现类中声明 Mapper 接口对象,并生成get/set 方法
  - spring 无法管理 Servlet,在 service 中取出 Servie 对象（下面是Servlet中代码）
```airMapper_xml
@WebServlet("/airport")
public class AirportServlet extends HttpServlet{
    private AirportService airportService;
    
    @Override
    public void init() throws ServletException {
// 对service 实例化
// ApplicationContext ac = new
ClassPathXmlApplicationContext("applicationContext.xm l");

// spring 和web 整合后所有信息都存放在webApplicationContext
// 下面语句作用：因为现在在web.xml中配置spring配置文件，也就是说当Tomcat 启动后会将spring配置文件中所有东西启动，启动完成之后会把信息放在webApplicationContext容器中，下面是往外取东西

  ApplicationContext ac = WebApplicationContextUtils.getRequiredWebApplicationContext(getServletContext());
  airportService=ac.getBean("airportService",AirportServiceImpl.class);
  }
  
  @Override
  protected void service(HttpServletRequest req,HttpServletResponse resp) throws ServletException,IOException {
      req.setAttribute("list", airportService.show());
      req.getRequestDispatcher("index.jsp").forward(req,resp);
  }
}
```

