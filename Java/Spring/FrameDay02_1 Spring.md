# Frame02_01 Spring

[TOC]

## 一、Spring 框架简介及官方压缩包目录介绍

**Spring 几大核心功能**：IoC/DI：控制反转/依赖注入；AOP：面向切面编程；声明式事务；

![Spring Framework Runtime](FrameDay02_1%20Spring.resource/Spring%20Framework%20Runtime.png)

-  test： spring 提供的测试功能；
- Core  Container：是核心容器，里面的内容是 Spring 启动最基本的条件；
  - Beans: **Spring 中负责创建类对象并管理对象**；
  - Core: 核心类；
  - Context: 上下文参数，**用于获取外部资源或者管理注解等**；
  - SpEl: 对应于 `expression.jar`
- AOP：实现 aop 功能需要的依赖
- Aspects: 切面 AOP 依赖的包
- Data  Access/Integration：**Spring 封装数据访问层相关内容**
  - JDBC：Spring 对 JDBC 封装后的代码；
  - **ORM: 封装了持久层框架的代码，例如 Hibernate**
  - **transactions**：对应 `spring-tx.jar`,声明式事务时使用；
- WEB：**需要 Spring 完成 web 相关功能时需要**；

  例如：由 tomcat 加载 spring 配置文件时需要有 spring-web 包

Servlet 是具体的业务功能，不能封装

#### Spring 框架中重要概念

- 容器(Container): 将 Spring 当作一个大容器.
- 老版本中的使用 BeanFactory 接口，**新版本中是 ApplicationContext 接口，是 BeanFactory 子接口，BeanFactory 的功能在 ApplicationContext 中都有**；

- 从 Spring3 开始把 Spring 框架的功能拆分成多个 jar，Spring2 及以前就一个 jar

## 二、IoC：控制反转（Inversion of Control）

- IoC 完成的事情：**原先由程序员主动通过 new 来实例化对象事情，现在转交给 Spring 负责**。
- 控制反转中「控制」的是类的对象；反转指的是：转交给 Spring 负责；
- IoC 最大的作用：解耦
即程序员不需要管理对象，解除了对象管理和程序员之间的耦合。

## 三、Spring 环境搭建与使用

#### （一）搭建环境

- 导入 jar，包括四个核心包（对应四个核心容器）一个日志包(commons-logging)

    ```java
    spring-beans.jar
    spring-core.jar
    spring-context.jar
    spring-expression.jar
    commons-logging.jar
    ```

- 在 src 下新建 spring 配置文件 `applicationContext.xml` 

  上面的文件名称和路径可以自定义，而 `applicationContext.xml` 中配置的信息最终存储到了 AppliationContext 容器中。

- spring 配置文件是基于 schema（MyBatis 是基于 DTD）
  - schema 文件扩展名 `.xsd`

  - DTD 是 XML 的语法检查器。而 schema 是 DTD 的升级版，拥有更好的拓展性，主要体现在每次引入一个 xsd 文件是一个 namespace(即 xmlns)；

  - 配置文件中一般只需要引入基本 schema；其基本内容通常为：

    ```java
    <?xml version="1.0" encoding="UTF-8"?> 
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">   
    
    </beans>
    ```

#### （二）使用

- 首先需要一个 `People.java` 的实体类，一般对应数据库中一个表；

- 然后新建配置文件：`applicationContext.xml`

  schema 通过 `<bean/>` 创建对象；同时默认**配置文件被加载时就创建对象**；

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
         http://www.springframework.org/schema/beans/spring-beans.xsd">
      
      <!-- id 是获取该对象的标识，目的是为了以后方便的获取使用该对象； class 表示具体创建的是哪个类对象-->
      <bean id="people" class="com.gjxaiou.pojo.People"></bean>
  
  </beans>
  ```

- 然后编写测试方法:Test.java，在这里可以创建对象
  - `getBean(“<bean>标签的 id 值”,返回值类型);`如果没有第二个参数, 默认是 Object；
  - `getBeanDefinitionNames()`，是 Spring 容器中目前管理的所有对象；

  ```java
  public class SpringTest {
      public static void main(String[] args) {
          // 加载配置文件，同时创建对应类
          ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
          // 使用 getBean() 创建对象
          People people = applicationContext.getBean("people", People.class);
          System.out.println("创建的对象名为：" + people + "\n");
  
          // 查看当前 Spring 容器中管理的所有类对象(以及数量)
          int beanDefinitionCount = applicationContext.getBeanDefinitionCount();
          System.out.println("当前 Spring 容器中管理的类对象数目为：" + beanDefinitionCount + "\n");
          String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
          for (String beanDefinitionName : beanDefinitionNames) {
              System.out.println( "当前 Spring 容器中管理的类对象为：" + beanDefinitionName + "\n");
          }
      }
  }
  ```

  程序运行结果为：

  ```shell
  创建的对象名为：People(id=0, name=null, gender=null, score=0, tel=null)
  
  当前 Spring 容器中管理的类对象数目为：1
  
  当前 Spring 容器中管理的类对象为：people
  ```

## 四、Spring 创建对象的三种方式

首先提供一个实体类：
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class People {
    private Integer id;
    private String name;
    private String gender;
    private Integer score;
    private String tel;
}
```

### 方案一：通过构造方法创建
- 无参构造创建：默认情况
- 有参构造创建：需要明确配置
  - 首先**需要在类中提供有参构造方法**
  - 然后在 applicationContext.xml 中设置调用哪个构造方法创建对象

    如果设定的条件**匹配多个构造方法执行最后的构造方法**，一般index、name、type有可以唯一确定对象即可，也可以多个结合使用来唯一确定
    - index : 参数的索引，从 0 开始
    - name: 参数名
    - type：类型(区分开关键字和封装类如 int 和 Integer)
```xml
<bean id = "people" class= "com.gjxaiou.pojo.People"></bean>
```

```java
public void createObjectByConstructor(){
  // 1.启动 spring 容器
  ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
  // 2.从 spring 容器中取出数据
  People people = (People)context.getBean("people");
}
```

spring 只支持两种工厂：实例工厂和静态工厂

- **工厂设计模式：就是帮助创建类对象**，一个工厂可以生产多个对象；
- **实例工厂：需要先创建工厂，才能生产对象**
- 使用静态工厂不需要创建工厂，可以快速创建对象；

### 方案二：通过实例工厂

首先必须要有一个实例工厂

```java
public class PeopleFactory {
    public  People newInstance() {
        return new People(4, "张三", "男", 98,"12323232");
    }
} 
```
以前常规的使用工厂创建对象方式：
```java
PeopleFactory factory = new PeopleFactory();
People people = factory.newInstance();
```
下面是对应的在 spring 中使用的方式：（factory-bean 对应于 id），即在 applicationContext.xml 中配置
```xml
<!--使用实例工厂创建对象-->
<!--首先创建一个工厂-->
<bean id="factory" class="com.gjxaiou.factory.PeopleFactory"></bean>
<!--根据工厂创建对象，factory-bean 对应工厂 id，factory-method 对应创建对象方法-->
<bean id="factoryPeople" factory-bean="factory" factory-method="newInstance"></bean>
```

### 方案三：通过静态工厂

**spring 容器只负责调用静态工厂方法，而静态工厂方法内存实现需要自己完成；**

- 首先编写一个静态工厂(在方法上添加 static)
```java
public class StaticPeopleFactory {
    public static People newInstance(){
        return new People(5, "李四", "女", 98,"12323232");
    }
}
```

 -  然后在 applicationContext.xml 中进行配置
```xml
<!--使用静态工厂创建对象-->
<!--不需要创建工厂，直接创建对象，只需要指明工厂类可以直接使用工厂中的方法-->
<bean id="staticFactoryPeople" class="com.gjxaiou.factory.StaticPeopleFactory"
      factory-method="newInstance"></bean>
```

### Spring 容器创建对象的时机

- 默认情况下：启动 spring 容器便创建对象；
- 可以在 spring 配置文件中的 `<bean>` 标签中设置  `lazy-init` 属性
  - 如果 `lazy-init` 为 `default/false` ，即在启动 spring 容器时创建对象（默认）；
  - 如果 `lazy-init` 为 `true` ，则在执行 `context.getBean` 时才要创建对象

注：启动 spring 容器对应于代码执行到 `ApplicationContext context = new ClassPathXmlApplicationContext(“applicationContext.xml”);`时候；
**如果配置了多个相同的 bean，则都会执行构造函数；**

代码示例：
```java
public void People(){
    public People(){
      System.out.println("执行构造函数");
    }  
}
```

```xml
<!-- 在 applicationContext.xml 配置实现分别放行执行其中一个 
<bean id="people" class="com.gjxaiou.pojo.People"></bean>
<bean id="people" lazy-init="false" class="com.gjxaiou.pojo.People"></bean>
<bean id="people" lazy-init="true" class="com.gjxaiou.pojo.People"></bean>
```

测试函数为：
```java
public class Test {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
                "applicationContext.xml");
        System.out.println("这是分割线");
        People people = (People)context.getBean("people");
    }
}
```
测试结果为：
```java
// 下面是上面挨个放行之后执行的结果集
执行构造函数
这是分割线

执行构造函数
这是分割线

这是分割线
执行构造函数
```


## Spring 的 bean 中的 scope 配置

```xml
<!--默认值或者设置为 singleton 都表示产生的对象是单例的-->
<bean id="people" scope="singleton" class="com.gjxaiou.pojo.People"></bean>
<!--prototype 表示多例模式-->
<bean id="people" scope="prototype" class="com.gjxaiou.pojo.People"></bean>
```

测试文件
```java
public class Test {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
            "applicationContext.xml");
        People people1 = (People)context.getBean("people");
        People people2 = (People)context.getBean("people");
        System.out.println(people1.equals(people2));
    }
}
```

测试结果：
```java
// 使用单例模式时，只执行一次构造函数
执行构造函数
true

// 执行多例模式时，执行两次构造函数
执行构造函数
执行构造函数
false
```

**注：在单例模式下，启动 spring 容器，便会创建对象；在多例模式下，启动容器并不会创建对象，获得 bean 的时候才会创建对象**


## 五、Spring 的生命周期
- 步骤一：Spring 容器创建对象
- 步骤二：执行 init 方法
- 步骤三：调用自己的方法
- 步骤四：当 spring 容器关闭的时候执行 destroy 方法

**注意：当 scope 为 "prototype" 时，调用 close（） 方法时是不会调用 destroy 方法的**
下面是测试程序：

```java
public class SpringLifeCycle {
    public SpringLifeCycle(){
        System.out.println("SpringLifeCycle");
    }
    //定义初始化方法
    public void init(){
        System.out.println("init...");
    }
    //定义销毁方法
    public void destroy(){
        System.out.println("destroy...");
    }

    public void sayHello(){
        System.out.println("say Hello...");
    }
}
```

```xml
<bean id="springLifeCycle" init-method="init" destroy-method="destroy" 
          class="com.gjxaiou.pojo.SpringLifeCycle"></bean>
```
测试程序：
```java
public class Test {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
                "applicationContext.xml");
        SpringLifeCycle springLifeCycle = (SpringLifeCycle) context.getBean("springLifeCycle");
        springLifeCycle.sayHello();

        // 销毁 Spring 容器
        ClassPathXmlApplicationContext classContext = (ClassPathXmlApplicationContext) context;
        classContext.close();
    }
}
```

测试结果：
```java
SpringLifeCycle
init...
say Hello...
destroy...
```



第六和第七均属于 DI 依赖注入
第六属于一般属性的注入，第七属于对象的注入；

## 六、如何给 Bean（对象）的属性赋值(注入)
### 方法一： 通过构造方法设置属性值；

首先类中提供有参和无参两个构造方法：

```java
public class People{
    public People(){
    }

    public People(int id, Student student){
        this.id = id;
        this.student = student;
    }
}
```

然后在 Spring 配置文件中进行注入

```java
<!--
   index：代表参数的位置，从 0 开始计算；
   type：指的是参数的类型，在有多个构造函数的时候使用 type 进行区分，如果可以区分哪一个构造函数就可以不用写 type；
   value：给基本类型赋值；
   ref：给引用类型赋值；
-->
<bean id = "people" class = "com.gjxaiou.pojo.People">
    <constructor-arg index = "0", type = "java.lang.Integer" value = "1"></constructor-arg>
    <constructor-arg index = "1", type = "com.gjxaiou.pojo.Student" ref = "student"></constructor-arg>
</bean>

<bean id = "student" class = "com.gjxaiou.pojo.Student"></bean>
```

测试：利用构造函数进行赋值

```java
package com.gjxaiou.pojo;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext(
				"applicationContext.xml");
		People people = (People) context.getBean("people");
		System.out.println(people.getId());
	}
}
```

**总结：** 

如果 spring 的配置文件中的 bean 中没有 `<constructor-arg>` 该元素，则调用默认的构造函数。有则该元素确定唯一的构造函数。

### 方法二：设置注入

**本质上通过对象的 get 和 set 方法，因此一定要生成 get 和 set 方法，包括引用的对象的 get 和 set 方法；**)

首先基本的数据类如下：

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class People {
    private Integer id;
    private String name;
    private String tel;
    private List<String> list1;
    private Map<String, String> map1;
    private Set<String> set1;
    private String[] string;
    private Properties properties;
    private Student student;
}
```
  -  如果属性是基本数据类型或 String 等简单的，首先创建 People 类对象，下面是给对象的属性进行赋值
 **`<property>` 标签用来描述一个类的属性，基本类型封装类、 String 等需要值的类型使用 value 赋值，引用类型使用 ref 赋值；** 

 ```xml
 <bean id="people" class="com.gjxaiou.pojo.People">
     <property name="id" value="12"></property>
     <property name="name" value="chenliu"></property>
     <property name="tel" value="123243"></property>
 </bean>
 ```

 上面代码等效于：（一般使用上面方式）	

 ```xml
 <bean id="peo" class="com.gjxaiou.pojo.People">
     <property name="id">
         <value>12</value>
     </property>
     <!-- 。。。。。。-->
 </bean>
 ```

  - 如果属性是 `Set<?>`
    这里 set 里面存放的基本数据类型，如果存放的是对象，则需要将 `<value>` 标签更换为  `<ref>` 标签，中间的值设置为对象即可；

  ```xml
  <property name="set1">
      <set>
          <value>1</value>
          <value>2</value>
      </set>
  </property>
  ```

- 如果属性是 `List<?>`

     ```xml
     <property name="list1">
         <list>
             <!--虽然这里存放 String 数据类型，但是值不使用 ""-->
             <value>1</value>
             <value>2</value>
         </list>
     </property>
     ```

     如果 list 中就只有一个值，直接在里面使用 value 即可；

     `<property name="list" value="1"></property>`

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

- 如果属性是 `map<key, value>` 以及对象类型

     ```xml
     <property name="map">
         <map>
             <entry key="a" value="b"></entry>
             <entry key="c" value="d"></entry>
             <entry key="d">
                 <ref bean="student"/>
             </entry>
         </map>
     </property>
     ```

     备注：引用上面的 student，需要同时配置 `<bean id = student class = "com.gjxaiou.pojo.Student"></bean>`

- 如果属性是 Properties 类型，下面代码作用是对应于从 XXX.properties文件中取值

    ```xml
    <property name="demo">
      <props>
        <prop key="key">value</prop>
        <prop key="key1">value1</prop>
      </props>
    </property>
    ```

==至此实现了对一个类中属性的赋值，但是怎么对该类中依赖的另一个类的属性进行赋值呢，见下==

## 七、DI（Dependency Injection）：依赖注入

**spring 动态的向某个对象提供它所需要的其他对象**。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象 A 需要操作数据库，以前我们总是要在 A 中自己编写代码来获得一个 Connection 对象，有了 spring 我们就只需要告诉 spring，A 中需要一个 Connection，至于这个 Connection 怎么构造，何时构造，A 不需要知道。在系统运行时，spring 会在适当的时候制造一个 Connection，然后像打针一样，注射到 A 当中，这样就完成了对各个对象之间关系的控制。A 需要依赖 Connection 才能正常运行，而这个 Connection 是由 spring 注入到 A 中的，依赖注入的名字就这么来的。那么DI 是如何实现的呢？ Java 1.3 之后一个重要特征是**反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性**，==**spring 就是通过反射来实现注入的**==。

**依赖注入，就是给属性赋值（包括基本数据类型和引用数据类型）**

DI 和 IoC 是一样的，当一个类 A 中需要依赖另一个类 B 对象时,把 B 赋值给 A 的过程就叫做依赖注入。
解释：就是在 A 中的属性是 B 类对象，现在要将A类中的 B 属性赋值； 

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
<bean id="a" class="com.gjxaiou.pojo.A">
    <property name="b" ref="b"></property> </bean>
    // 然后对 A 类中的 B 对象进行实例化
    <bean id="b" class="com.gjxaiou.pojo.B">
      <property name="id" value="1"></property>
      <property name="price" value="12"></property>
</bean>
```

## 八、使用 Spring 简化 MyBatis

- 步骤一：需要导入 MyBatis 所 有 jar 和 Spring 基本包，以及 spring-jdbc，spring-tx，spring-aop，spring-web 以及 Spring 整合 MyBatis 的包等；

    ```xml
    asm-3.3.1.jar
    cglib-2.2.2.jar
    slf4j-api-1.7.5.jar
    slf4j-log4j12-1.7.5.jar
    commons-logging-1.1.3.jar
    javassist-3.17.1-GA.jar
    jstl-1.2.jar
    log4j-1.2.17.jar
    log4j-api-2.0-rc1.jar
    log4j-core-2.0-rc1.jar
    
    mybatis-3.2.7.jar
    mybatis-spring-1.2.3.jar
    mysql-connector-java-5.1.30.jar
    
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

- 步骤二：配置 web.xml
  **为了让 Tomcat 在启动时候自动加载 Spring 的配置文件，需要在 web.xml 中告诉 Tomcat 怎么加载；**

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           version="4.0">
  
      <!--上下文参数，配置 Spring 配置文件位置，告诉 Tomcat 启动时候加载 Spring 配置文件路径-->
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <!-- Spring 配置文件目录-->
          <param-value>classpath:applicationContext.xml</param-value>
      </context-param>
      
      <!--内部封装了一个监听器，用于帮助加载 Spring 配置文件-->
      <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
  </web-app>
  ```

- 步骤三：编写 spring 配置文件 applicationContext.xml，对应于实现 myBatis.xml 配置文件中的内容；

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans
        xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-be ans.xsd">
    
        <!-- 数据源封装类 .数据源:获取数据库连接,spring-jdbc.jar 中(类名知道即可)，代替类MyBatis中的dataSource配置功能-->
    <bean id="dataSouce" class="org.springframework.jdbc.datasource.DriverMana gerDataSource">
      <property name="driverClassName" value="com.mysql.jdbc.Driver"> </property>
      <property name="url" value="jdbc:mysql://localhost:3306/ssm"> </property>
      <property name="username" value="root"></property>
      <property name="password" value="smallming"></property>
    </bean>
    
      <!-- 创建 SqlSessionFactory 对象-->
      <!--这个类是专门在 Spring 中生成 sqlSessionFactory 对象的类-->
      <bean id="factory" class="org.mybatis.spring.SqlSessionFactoryBean">
      <!-- 数据库连接信息来源于dataSource -->
      <property name="dataSource" ref="dataSouce"></property>
      </bean>
      
      <!-- 扫描器相当于 mybatis.xml 中 mappers 下 package 标签,扫描 com.gjxaiou.mapper 包后会给对应接口创建对象-->
      <bean
          class="org.mybatis.spring.mapper.MapperScannerConfigurer">
      <!-- 要扫描哪个包-->
          <property name="basePackage" value="com.gjxaiou.mapper"></property>
      <!-- 和factory 产生关系-->
          <property name="sqlSessionFactory" ref="factory"></property>
      </bean>
    
      <!-- 由 spring 管理 service 实现类-->
      <bean id="airportService"
          class="com.gjxaiou.service.impl.AirportServiceImpl">
          <property name="airportMapper" ref="airportMapper"></property>
      </bean>
    </beans>
    ```

- 编写代码
  - 首先提供 POJO 类
  - 编写 mapper  包下时，**必须使用接口绑定方案或注解方案(必须有接口)**
  - 正常编写 Service 接口和 Service 实现类

    需要在 Service 实现类中声明 Mapper 接口对象,并生成get/set 方法
  - spring 无法管理 Servlet,在 service 中取出 Service 对象（下面是Servlet中代码）

```java
@WebServlet("/airport")
public class AirportServlet extends HttpServlet{
    private AirportService airportService;
    
    @Override
    public void init() throws ServletException {
     // 对 service 实例化
     // ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");

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


## 九、使用注解
Annotation(注解) 是 JDK1.5 及以后版本引入的。它可以用于创建文档，跟踪代码中的依赖性，甚至执行基本编译时检查。注解主要用于简化前面的 XML 配置方式。

### （一）注解的基本使用：@Component

**示例**：People 实体类

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class People(){
    private integer id;  
    private String name;
    private String gender;
}
```
不使用注解需要在 XML 中配置：`<bean id = "people" class = "com.gjxaiou.pojo.People></bean>`
==使用注解的步骤：==

- **步骤一：在 applicationContext.xml 中导入命名空间**（用于约束 xml 文件格式的，下面的第一个标签为：xmls:context，表示下面的标签的格式为：`<context:标签名>`）

    ```xml
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd"
    ```

- **步骤二：引入组件扫描器**
  通过组件扫描，扫描所有含有注解的类：`<context:component-scan base-package = "com.gjxaiou.annotation"></context:component-scan>`，这里使用 base-package：表示含有注解类的包名，如果需要扫描多个包，则上面的代码需要书写多行改变 base-package 里面内容即可；

- **步骤三：在 People 类中添加注解**：@Component

```java
@Component
public class People{
  private Integer id;
  private String name;
  private String gender;
}
```

测试程序
```java
@test
public void annotationTest(){
  // 启动 Spring 容器；从 Spring 容器中取出数据；通过对象调用方法；
    ApplicationContext context = new ClassPathXmlApplicationContext(applicationContext.xml);
    People people = (People)context.getBean("people");
    System.out.println(people.getId());
}
```

**注：** 如果 @Component 后面没有参数，则 getBean();中使用该类的类型首字母小写即可；
即上面的等价于：`<bean id = "people" class = "com.gjxaiou.annotation.People">`
如果后面有参数，例：@Component(“dd”)，相当于上面的 id 等于 dd；

### （二）@Component 衍生注解
下面的三个注解是 @Component 的衍生注解，功能是一样的，只是对应的层不同；

- @Repository：Dao 层
- @Service：service 层
- @Controller：web 层

### （三）@Resource
该注解用于对类成员变量、方法及其构造方法进行标注，完成自动装配工作。即可以用来消除 set、get 方法；

代码示例：
```java
public class Student{
    pubic void descStudent(){
      System.out.println("执行 descStudent()");
    }
}
```

```java
public class People{
    private Student student;
    public void showStudent(){
        this.student.descStudetn();
    }
}
```

如果想要获取 People 对象，同时调用 showStudent(）方法：即是给属性 student 进行实例化，即是依赖注入
- 不使用注解（在 applicationContext.xml 中配置）：

    ```xml
    <property name = "students">
        <ref bean = "student">
    </property>
    
    <bean id = "student" class = "com.gjxaiou.annotation.Student"></bean>
    ```

- 使用注解：不需要在 applicationContext.xml 中进行配置，直接在实体类添加注解即可；只需要保证 People 中 @Resource()中 name 的值和 Student 中的 @Component 的值相同；

    ```java
    @Compontent("people")
    public class People{
        @Resource(name="student")
        public Student student;
        public void showStudent(){
            this.student.descStudent();
        }
    }
    
    @Compontent("student")
    public class Student{
        pubic void descStudent(){
          System.out.println("执行 descStudent()");
        }
    }
    ```

    @Resource 注解以后，判断该注解 name 的属性是否为""(name 没有写)
    - 如果没有写 name 属性，则会让属性的名称的值和 spring 配置文件 bean 中 ID 的值做匹配（如果没有进行配置，也和注解 @Component 进行匹配），如果匹配成功则赋值，如果匹配不成功，则会按照 spring 配置文件 class 类型进行匹配，如果匹配不成功，则报错

    - 如果有 name 属性，则会按照 name 属性的值和 spring 的 bean 中 ID 进行匹配，匹配成功，则赋值，不成功则报错


### （四）@Autowired 自动装配

功能和注解 @Resource 一样，可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。只不过注解==**@Resource 是按照名称来进行装配**，而**@Autowired 则是按照类型来进行装配**==；

- 首先创建一个接口和实现类 
```java
// 接口：PeopleDao
public interface PeopleDao{
    public void savePeople();
}

// 对应的实现类 PeopleDaoImpl
@Component("peopleDaoImpl")
public class PeopleDaoImpl implements PeopleDao{
    @Override
    public void savePeople(){
        System.out.println("save People");
    }
}
```
- 创建 PeopleService 使用上面的接口
```java
@Service("peopleService")
public class PeopleService(){
    @Autowired
    private PeopleDao peopleDao;
    public void savePeople(){
        this.peopleDao.savePeople();
    }
}
```

注意：这里我们在 `private PeopleDao peopleDao` 上面添加了注解 @Autowired，它**首先会根据类型去匹配**，PeopleDao 是一个接口，它的实现类是 PeopleDaoImpl，那么这里的意思就是：
 `PeopleDao peopleDao = new PeopleDaoImpl();`

如果 PeopleDao 接口的实现类多个，如：
```java
@Component("personDaoImplTwo")
public class PersonDaoImplTwo implements PersonDao{
    @Override
    public void savePerson() {
        System.out.println("save Person Two");        
    } 
}
```

- 方法一：更改对象名称：改为：`private PeopleDao peopleDaoImpl`，因为 ==@Autowired 注解默认是按照类型进行匹配，如果类型匹配有多个，则按照名称匹配，这里就是将创建的实例名称改为实现类 PeopleDaoImpl 的 @Component 中的值；==

- 方法二：配合 `@Qualifier(“名称”)` 使用
即在@Autowird 下面增加一行：`@Qualifier("peopleDaoImpl")`，参数值为对应实现类 @Component 中的值；

**总结**

- 在使用 @Autowired 时，首先在容器中查询对应**类型**的 bean

- 如果查询结果刚好为一个，就将该 bean 装配给 @Autowired 指定的数据

- 如果查询的结果不止一个，那么 @Autowired 会根据名称来查找。

- 如果查询的结果为空，那么会抛出异常。解决方法时，使用 required=false
