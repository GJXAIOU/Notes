# FrameDay02_2 Spring

[TOC]


## 一、面向切面编程(Aspect Oriented Programming，AOP)
**在程序原有纵向执行流程中，针对某一个或某一些方法添加通知，形成横切面过程就叫做面向切面编程。**

>AOP 可以说是 OOP（Object-Oriented Programing，面向对象编程）的补充和完善。OOP 引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。**当我们需要为分散的对象引入公共行为的时候，OOP 则显得无能为力**。即 OOP 允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。对于其他类型的代码，如安全性、异常处理和透明的持续性也是如此。这种散布在各处的无关的代码被称为横切（cross-cutting）代码，在 OOP 设计中，它导致了大量代码的重复，而不利于各个模块的重用。
>
>AOP 技术利用「横切」技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为「Aspect」，即方面。简而言之：就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP 代表的是一个横向的关系，**如果说「对象」是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，即所谓的「方面」，然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。**
>
>通过「横切」技术，AOP 把软件系统分割为：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处都基本相似。比如权限认证、日志、事务处理。
>
>实现 AOP 的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的 执行；二是采用静态织入的方式，引入特定的语法创建「方面」，从而使得编译器可以在编译期间织入有关“方面”的代码。

- **AOP 和 Filter 区别主要体现在：拦截的对象不同**：
  
    - **AOP 拦截的是切点（即方法），只要该方法可以被 Spring 管理就可以被拦截**；
    - Filter 拦截的是请求；

    主要可以在程序执行的任意一个方法的前面或者后面额外添加或者扩充一些功能；

<img src="FrameDay02_2%20Spring.resource/%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E7%BC%96%E7%A8%8B.png" alt="面向切面编程"  />


- 常用概念
  - **原有功能：称为切点： `pointcut`**；
  - 前置通知：在切点之前执行的功能：`beforeAdvice`
  - 后置通知：在切点之后执行的功能：`afterAdvice`
  - 如果切点执行过程中出现异常，会触发异常通知：`throwsadvice`
  - 所有功能总称叫做切面；
  - **织入: 把切面嵌入到原有功能的过程叫做织入**

##  二、两种 AOP 实现方式

[可以阅读参考的博客](https://blog.csdn.net/liuhaiabc/article/details/52597204)

**方式一：Schema-based**

- 每个通知都需要实现接口或类（`implement MethodBeforeAdvice` 或者 `implement  AfterReturningAdvice`）
- 配置 spring 配置文件时在`<aop:config>`配置

**方式二： AspectJ**

- 每个通知不需要实现接口或类
- 配置 spring 配置文件是在`<aop:config>` 的子标签 `<aop:aspect>` 中配置

### （一）Schema-based 实现步骤
**本质上即在切点方法之前和之后分别执行了一个方法**

- **步骤一：首先导入 jar**，除了 spring 核心功能包外，注意添加两个额外的包：aopalliance.jar 和 aspectjweaver.jar

- **步骤二：然后新建通知类并重写 before() 或者 afterReturning() 方法**（对应前置通知类和后置通知类，根据需要使用）

  - 新建前置通知类：实现 MethodBeforeAdvice 接口

    `before()` 方法三个参数含义：切点方法对 Method 对象、切点方法参数、切点在哪个对象中；

    ```java
    // 前置通知类实现 MethodBeforeAdvice 接口
    public class MyBeforeAdvice implements MethodBeforeAdvice {
        @Override
        public  void  before(Method method, Object[] objects, Object o) throws Throwable {
            System.out.println("执行前置通知");
        }
    }
    ```

  - 新建后置通知类：实现 AfterReturningAdvice 接口

    `afterReturning()` 方法四个参数含义：切点方法的返回值、切点方法的对象、切点方法参数、切点方法所在类的对象；

    ```java
    public class MyAfterAdvice implements AfterReturningAdvice {
        @Override
        public void afterReturning(Object o, Method method, Object[] objects, Object o1) throws Throwable {
            System.out.println("执行后置通知");
        }
    }
    ```

- **步骤三：配置 spring 配置文件**

  - 首先引入 aop 命名空间(通过查询文档即可)
  
  - 然后配置通知类的 `<bean>`，同样可以使用注解来表示。
  
  - 配置切面
    - `*` 为通配符,匹配任意方法名,任意类名,任意一级包名
    - 如果希望匹配任意方法参数 `(..)`，就是下面 expression中的最后demo2(..)
    通配符的使用范例：
    `<aop:pointcut expression="execution(* com.gjxaiou.test.*.*(..))" id="mypoint"/>` 表示test 这个包下面的任意类的任意方法的任意参数都需要形成切面，本质上任意位置都可以使用 * 表示任意；
  
    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans
            xmlns="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:aop="http://www.springframework.org/schema/aop"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/aop
            http://www.springframework.org/schema/aop/spring-aop.xsd">
        <!--步骤一：需要在上面导入 AOP 命名空间-->
    
        <!--步骤二：首配置通知类对象，在切面中进行引入-->
        <bean id="myBefore" class="com.gjxaiou.advice.MyBeforeAdvice"></bean>
        <bean id="myAfter" class="com.gjxaiou.advice.MyAfterAdvice"></bean>
    
        <!--步骤三：配置切面-->
        <aop:config>
            <!--配置切点，里面的路径是该方法的完整路径（包括参数）-->
            <aop:pointcut id="mypoint"
                          expression="execution(* com.gjxaiou.pointcut.AOPpointcut.AopPointcut())"/>
            <!--为切面添加通知，pointcut-ref 为切点的 id-->
            <aop:advisor advice-ref="myBefore" pointcut-ref="mypoint"></aop:advisor>
            <aop:advisor advice-ref="myAfter" pointcut-ref="mypoint"></aop:advisor>
        </aop:config>
    
    
        <bean id = "pointcut" class="com.gjxaiou.pointcut.AOPpointcut"></bean>
    
    </beans>
    ```
  
- 步骤四：编写测试代码

    ```java
    public class AOPTest {
        public static void main(String[] args) {
            ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
            AOPpointcut pointcut = applicationContext.getBean("pointcut", AOPpointcut.class);
            pointcut.AopPointcut();
        }
    }
    ```

- 运行结果:

    ```java
    执行前置通知
    这是切点
    执行后置通知
    ```

### （二）配置异常通知的步骤(AspectJ 方式)

- **只有当切点报异常才能触发异常通知**
- 在 Spring 中有 AspectJ 方式提供了异常通知的办法
 如果希望通过 schema-base 实现，需要按照特定的要求自己编写方法。

**实现步骤:**

- 首先新建自定义类和自定义方法

    ```java
    public class MyThrowAdvice {
        public void MyException(Exception e){
            System.out.println("执行异常通知" + e.getMessage());
        }
    }
    ```

- 然后在 spring 配置文件中配置【配置和上面不一样，不要看错】
  - `<aop:aspect>`的 ref 属性表示:方法在哪个类中.
  - `<aop: xxxx/>` XXX表示什么通知
  -  method: 当触发这个通知时,调用哪个方法
  -  throwing: 表示异常对象名,必须和通知中方法参数名相同(可以不在通知中声明异常对象)

```xml
<!--配置异常通知-->
<bean id="exceptionAdvice" class="com.gjxaiou.exception.MyThrowAdvice"></bean>
<aop:config>
    <!-- 这里 ref 告诉 spring，这个 method 是哪一个类的，同时上面也要配 bean -->
    <aop:aspect ref="exceptionAdvice">
        <aop:pointcut id="mypoint"
                      expression="execution(* com.gjxaiou.pointcut.AOPpointcut.AopPointcut())"/>
        <!-- 这里的 method 是当告诉 spring，触发异常的时候调用的是哪一个方法,后面是针对于哪一个切点的;最后的 throwing 值为上面声明中的异常名-->
        <aop:after-throwing method="MyException" pointcut-ref="mypoint"
                            throwing="e"></aop:after-throwing>
    </aop:aspect>
</aop:config>

```
如果不在这里面声明通知的话，另一种方式是：
在 Test 类中，通过 getBean 获取对象，然后通过对象调用方法的时候使用 try-catch，在上面的类中使用 throws抛出，不能使用 try-catch,否则 spring 接收不到异常

## 三、异常通知(Schema-based 方式)

-  步骤一：首先新建一个类实现 throwsAdvice 接口
  - **必须自己写方法,且方法名必须为 afterThrowing ，都写的话会报下面那一个**
  - 有两种参数方式：必须是 1 个或 4 个
- **异常类型要与切点报的异常类型一致**
```java
public class MyThrow implements ThrowsAdvice{
   public void afterThrowing(Method m, Object[] args,Object target, Exception ex) {
       System.out.println("执行异常通知");
   }
   public void afterThrowing(Exception ex) throws Throwable {
      System.out.println("执行异常通过-schema-base 方式");
    }
}
```

- 步骤二：然后在 ApplicationContext.xml 配置
```xml
<bean id="demo" class="com.gjxaiou.test.Demo"></bean>
<bean id="mythrow" class="com.gjxaiou.advice.MyThrow"></bean>

<aop:config>
    <aop:pointcut expression="execution(*com.gjxaiou.test.Demo.demo1())" id="mypoint"/>
    <aop:advisor advice-ref="mythrow"  pointcut-ref="mypoint" />
</aop:config>
```


## 四、环绕通知(Schema-based 方式)

- 把前置通知和后置通知都写到一个通知中,组成了环绕通知

**实现步骤**
- 首先新建一个类实现 MethodInterceptor，并且重写 Invoke 方法
```java
public class MyArround implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation arg0) throws Throwable {
        System.out.println("环绕-前置");
        // 放行,调用切点方式，语句前就相当于前置通知，语句后为后置通知
        Object result = arg0.proceed();
        System.out.println("环绕-后置");
        return result;
    }
}
```

- 步骤二：配置 applicationContext.xml
```xml
<bean id="demo" class="com.gjxaiou.test.Demo"></bean>
<bean id="myarround" class="com.gjxaiou.advice.MyArround"></bean>

<aop:config>
    <aop:pointcut expression="execution(*com.gjxaiou.test.Demo.demo1())" id="mypoint"/>
    <aop:advisor advice-ref="myarround" pointcut-ref="mypoint" />
</aop:config>
```

## 五、使用 AspectJ 方式实现

- 步骤一：新建类,不用实现
```java
public  class  MyAdvice  {
    public  void  mybefore(String  name1,int  age1){
       System.out.println("前置"+name1 );
    }
    public void mybefore1(String name1){
        System.out.println("前置:"+name1);
    }
    public void myaftering(){
        System.out.println("后置2");
    }
    public void myafter(){
        System.out.println("后置1");
    }
    public void mythrow(){
        System.out.println("异常");
    }
    public Object myarround(ProceedingJoinPoint p) throws Throwable{
        System.out.println("执行环绕");
        System.out.println("环绕-前置");
        Object result = p.proceed();
        System.out.println("环绕后置");
        return result;
    }
}
```

- 步骤二：配置 spring 配置文件
  - `<aop:after/>` 后置通知,是否出现异常都执行 
  -  `<aop:after-returing/>` 后置通知,只有当切点正确执行时执行
  -  `<aop:after/>` 和 `<aop:after-returing/>` 和`<aop:after-throwing/>`执行顺序都和在 Spring 配置文件中的配置顺序有关 
  -  `execution()` 括号不能扩上 args  
  -  中间使用 and 不能使用&& 由 spring 把 and 解析成&&
  -  args(名称) 名称自定义的.顺序和 demo1(参数,参数)对应
  -  `<aop:before/>  arg-names=” 名  称 ”` 名 称 来 源  于 expression=”” 中 args(),名称必须一样
  - args() 有几个参数,arg-names 里面必须有几个参数
  - `arg-names=””` 里面名称必须和通知方法参数名对应
```xml
<aop:config>
    <aop:aspect  ref="myadvice">
    <!-- 这里的name1 和 age1 仅仅是对参数进行赋值，然后将这些值赋值给通知，因此上面 advice 参数名称和他们相同-->
        <aop:pointcut  expression="execution(* com.gjxaiou.test.Demo.demo1(String,int)) and args(name1,age1)"  id="mypoint"/>
      <aop:pointcut  expression="execution(* com.gjxaiou.test.Demo.demo1(String))  and  args(name1)" id="mypoint1"/>
      
      <aop:before method="mybefore"pointcut-ref="mypoint" arg-names="name1,age1"/>
      <aop:before method="mybefore1" pointcut-ref="mypoint1" arg-names="name1"/>
     <aop:after method="myafter" pointcut-ref="mypoint"/>
     <aop:after-returning method="myaftering" pointcutref="mypoint"/>
     <aop:after-throwing method="mythrow" pointcutref="mypoint"/>
     <aop:around method="myarround" pointcut-ref="mypoint"/>
</aop:aspect>
</aop:config>
```



## 六、 使用注解(基于 Aspect) 实现 AOP

- spring 不会自动去寻找注解,必须告诉 spring 哪些包下的类中可能有注解

  在 spring 配置（applicationContext.xml）文件中引入 xmlns:context 命名空间

  ```java
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd">
  ```


 **实现步骤:**

- 步骤一：在 spring 配置文件中设置注解在哪些包中（使用组件扫描）
`<context:component-scan base-package="com.gjxaiou.advice,com.gjxaiou.test"></context:component-scan>`

同时要添加动态代理： proxy-target-class值为 true表示使用 cglib动态代理，值为 false 表示使用 jdk 动态代理；
`<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>`

- 步骤二：在 Demo.java 类中添加 @Componet,可以加参数用于别名，相当于直接替代了配置文件中的 bean 标签
  - 在方法上添加 `@Pointcut(“”)` 定义切点（必要步骤）
```java
@Component（"dd"）
public class Demo {
    @Pointcut("execution(* com.gjxaiou.test.Demo.demo1())")
    public void demo1() throws Exception{
        // int i = 5/0;
      System.out.println("demo1");
    }
}
```

- 步骤三：在通知类中配置MyAdvice.java 中
  -  @Component 类被 spring 管理
  - @Aspect 相当于 `<aop:aspect/>` 这个标签，表示通知方法在当前类中
```java
@Component
@Aspect // 表示该类为通知切面类
public class MyAdvice {
    @Before("com.gjxaiou.test.Demo.demo1()")
    public void mybefore(){
        System.out.println("前置");
    }
    @After("com.gjxaiou.test.Demo.demo1()")
    public void myafter(){
        System.out.println("后置通知");
    }
    @AfterThrowing("com.gjxaiou.test.Demo.demo1()")
    public void mythrow(){
        System.out.println("异常通知");
    }
    @Around("com.gjxaiou.test.Demo.demo1()")
    public Object myarround(ProceedingJoinPoint p) throws Throwable{
        System.out.println("环绕-前置");
        Object result = p.proceed();
        System.out.println("环绕-后置");
        return result;
    }
}
```

## 七、代理设计模式

代理设计模式主要涉及三大对象：真实对象、代理对象和抽象对象。其优点是：保护真实对象、让真实对象职责更加明确同时增加系统拓展性。

### （一）静态代理

- 由代理对象代理所有真实对象的功能
  - 需要自己编写代理类
  - 每个代理的功能需要单独编写

- 静态代理设计模式的缺点:

  当代理功能比较多时，代理类中方法需要写很多

### （二）动态代理

- 为了解决静态代理频繁编写代理功能缺点

- 分为：JDK 提供和 CGLib 动态代理

#### 1.JDK 动态代理
- 优点：JDK 自带，不需要额外导入 jar 包；

- 缺点:
  - 真实对象必须实现接口
  - **利用反射机制**，效率不高

- 使用 JDK 动态代理时可能出现下面异常【当将接口对象转换为具体真实对象时候】

  `Excepion in thread “main” java.lang.ClassCastException: com.sun.proxy.$Proxy0 cannot be cast to com.gjxaiou.TrueObject at com.gjxaiou.ProxyObject.main(ProxyObject.java:14)`


#### 2.cglib 动态代理

- cglib 优点:
  - **基于字节码**，生成真实对象的子类
   - 运行效率高于 JDK  动态代理
  - 不需要实现接口
- cglib 缺点:
  - 非 JDK 功能，需要额外导入 jar

- 使用 spring  aop 时,只要出现 Proxy  和真实对象转换异常
设置为 true 使用 cglib
设置为 false 使用 jdk(默认值)
`<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>`

