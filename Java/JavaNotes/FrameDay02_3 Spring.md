# FrameDay02_3 Spring


**主要内容**
- 声明式事务
- 事务传播行为
- 事务隔离级别
- 只读事务
- 事务回滚
- 常用注解
- Ajax 复习


##  一、自动注入

 - 前提：在 Spring 配置文件中对象名和 ref=”id” 即 id 名相同，可以使用自动注入，就是可以不配置`<property/>`

**两种配置办法**
- 在`<bean>`中通过 `autowire=””` 配置，只对这个 <bean> 生效；
- 在`<beans>`中通过 `default-autowire=””`配置，表示当前文件中所有 <bean> 都生效，是全局配置内容；

**参数配置**
- autowire=”” 可取值如下：
  - default: 默认值，根据全局 `default-autowire=””`值。**默认全局和局部都没有配置情况下，相当于 no；**
  - no: 不自动注入；
  - byName: 通过名称自动注入，**在 Spring 容器中找类的 Id**；因为在容器中找，因此就是Teacher不在配置文件中配置，使用component注解也是可以的；
  - byType: 根据类型注入；
    - spring 容器中不可以出现两个相同类型的<bean>，否则不能根据类型注入；
  - constructor: 根据构造方法注入；必须在People类中提供 Teacher类的构造方法；
    - 提供对应参数的构造方法(构造方法参数中包含注入对象)；
    - 底层使用 byName， 构造方法参数名和其他<bean>的 id相同；

代码示例：
```Teacher_java
public class Teacher{
}
```

```People_java
public class People{
    private Teacher teacher;
    public Teacher getTeacher(){
        return teacher;
    }
    public void setTeacher(Teacher teacher){
        this.teacher = teacher;
    }
}
```
原来 Spring 中的配置文件写法
```xml
<bean id = "techer" class = "com.gjxaiou.Teacher"> </bean>
<bean id = "people" class = "com.gjxaiou.People">
    <property name = "teacher" ref = "teacher">
</bean>
```
使用自动注解的写法：
```xml
<bean id = "techer" class = "com.gjxaiou.Teacher"> </bean>
<!--其中 XXX 为上面的选项之一，只要能唯一确定即可-->
<bean id = "people" class = "com.gjxaiou.People" autowire = "XXX"> </bean>
```



## 二、 Spring 中加载 properties 文件
将一些配置写在 properties 属性文件中，然后使用 Spring 进行加载读取；
properties 文件中的后面的值中间不能有空格

- 步骤一：首先在 src 下新建 xxx.properties 文件
- 在 spring 配置文件中先引入 xmlns:context（具体见文档），在下面添加
  如果需要记载多个配置文件逗号分割，每一个都是以classpath开头
`<context:property-placeholder location="classpath:db.properties", location="classpath:abc.properties"/>`
对应的 db.properties 配置示例为：
```db_properties
jdbc.driverClassName = com.mysql.cj.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/lianxi
jdbc.username = root
jdbc.password = GJXAIOU
```
然后对于其中的属性值，可以spring在配置文件的 bean ：id= DataSource中的value= ${key}取值
```applicationContext_xml
<context:property-placeholder location="classpath:db.properties"></context:property-placeholder>
    <bean id="dataSource"
          class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>
```
原来的 Spring 配置方式：
```applicationContext_java
<bean id = "dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
      <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>
      <property name="url" value="jdbc:mysql://localhost:3306/lianxi"></property>
      <property name="username" value="root"></property>
      <property name="password" value="GJXAIOU"></property>
  </bean>
```

- 添加了属性文件加载，并且在<beans>中开启自动注入需要注意的地方
  部分属性配置**当配置全局开启自动注入**之后需要进行限定：当 <beans>使用`default-sutowire = "byName"` 开启自动注入的时候，当同时使用 扫描器的时候；
  - SqlSessionFactoryBean（对象） 的 id 不能叫做 sqlSessionFactory，因为会在加载db.property的属性文件之前，就加载了其他文件，导致 ${jdbc.username}等等无法取到值
  - 修改：把原来通过ref 引用替换成value 赋值，因为自动注入只能影响ref，不会影响 value 赋值
正确的配置为：
```applicationContext_xml
<!-- 扫描器 --> 
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
 <property name="basePackage" value="com.bjsxt.mapper"></property>
 <property name="sqlSessionFactoryBeanName" value="factory"></property> </bean>
```


### Spring 使用注解

下面被 Spring 管理的类使用注解，首先需要在 Spring 中使用 <bean> 新建对象。
在**被Spring 管理的类中**通过 `@Value(“${key}”)` 取出properties 中内容，就不需要全部在Spring配置文件中使用 `value=  “${}”`进行取值；**Servlet 没有被 Spring 容器管理**

- 步骤一：添加注解扫描（URL 为所有包含注解的位置以及配置文件的位置）:在Spring配置文件中配置
```applicationContext_java
<context:property-placeholder location="classpath:second.properties"/>
<context:component-scan base-package="com.gjxaiou.service.impl">
</context:component-scan>
```

- 步骤二：在类中添加
  - key 和变量名可以不相同
  - 变量类型任意，只要保证 key 对应的 value **能转换成这个类型**就可以.
```Demo_java
package com.gjxaiou.service.impl
@service
public class Demo{
    @Value("${my.demo}")
    private String test;

    // 引用对象的时候
    @Resource
    private UserMapper userMapper;
}
```

- 步骤三：配置 second.properties 文件
```properties
my.demo = 123
```


## 三、scope 属性

scope 是<bean>的属性，**作用:控制对象有效范围(例如单例，多例等)**

- <bean/> 标签对应的对象**默认是单例的**；即bean声明一次之后，在类中即使使用getBean多次，但是最终对象是同一个。
无论获取多少次，都是同一个对象（**单例就是多次获取某一个对象的时候，不是每次都实例化**）

- scope 可取值（字符串）如下：【scope 是在 bean标签中配置】
  - singleton 默认值，单例
  - prototype 多例，每次获取重新实例化
  - request 每次请求重新实例化（同一次请求，无论多少对象仅仅实例化一次）
  - session 每个会话对象内，对象是单例的.
  - application 在 application 对象内是单例
  - global session 是spring 推 出 的 一 个 对 象 ， 依 赖 于 spring- webmvc-portlet.jar  ，类似于 session


## 四、单例设计模式

- 作用: 在应用程序中保证最多只能有一个实例；

- 好处:
  - 提升运行效率; 因为只需要 new 一次；
  - 实现数据共享. 案例:application 对象；

### （一）懒汉式
对象只有被调用时才去创建，同时由于添加了锁，所以导致效率低
**示例代码**
```SingleTon_java
public class SingleTon {
    //由于对象需要被静态方法调用,因此把方法 getInstance 设置为static
    //由于对象是static,必须要设置访问权限修饰符为private ,
    如果是public 可以直接调用对象,不执行访问入口
    private static SingleTon singleton;
   
     /** 单例设计模式首先要求其构造方法的私有化，这样其他类不能实例化这
    个类对象，同时需要对外提供提供入口
    */
    private SingleTon(){}
   
    // 实例方法,实例方法必须通过对象调用，因此需要设置方法为静态方法，才能调用
    public static SingleTon getInstance(){
    // 添加逻辑如果实例化过,直接返回
    if(singleton==null){
        /*
        * 多线程访问下,可能出现if 同时成立的情况（同时
        执行到if语句）,需要添加锁*/
        synchronized (SingleTon.class) {
            //双重验证
            if(singleton==null){
                singleton = new SingleTon();
              }
          }
      }
    return singleton;
    }
}
```
测试方法（使用方法）
```Test_java
public class Test{
    public static void main(){
        SingleTon singleTon1 = singleTon.getInstance();
        SingleTon singleTon12= singleTon.getInstance();
        System.out.println(singleTon1 == singleTon2);
     }
}
// output：true
```

注：构造方法特征方法名和类名相同且无返回值， 因为它必须返回当前对象的引用，所以没有返回值；


###  （二）饿汉式
解决了懒汉式中多线程访问可能出现同一个对象和效率低问题
```SingleTon
public class SingleTon {
    //在类加载时进行实例化.
    private static SingleTon singleton=new SingleTon();
    private SingleTon(){}
    public static SingleTon getInstance(){
        return singleton;
    }
}
```


##  五、声明式事务

- 编程式事务:
  - 由程序员编程事务控制代码，比如：OpenSessionInView 属于编程式事务（事务控制自己写）事务控制主要指就是事务的提交、回滚等等；

- 声明式事务:
 **事务控制代码已经由 spring 写好，程序员只需要声明出哪些方法需要进行事务控制和如何进行事务控制**然后 Spring 会帮助我们管理；

- **声明式事务都是针对于 ServiceImpl 类下方法的**；
- **事务管理器基于通知(advice)的**，即本质上就是通知；

- 在 spring 配置文件中配置声明式事务
需要引入 tx 命名空间
```xml
xmlns:tx="http://www.springframework.org/schema/tx" 
http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd"  
```
因此配置文件 applicationContext.xml 中的内容为：
```
<context:property-placeholder location="classpath:db.properties,classpath:second.properties"/>

<!-- 因为事务最终都是要提交给数据库的，因此需要 DataSource -->
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverMa nagerDataSource">
    <property name="driverClassName" value="${jdbc.driver}"></property>
    <property name="url" value="${jdbc.url}"></property>
    <property name="username" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>
</bean>

<!-- 通知类在spring-jdbc.jar 中 --> 
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSour
ceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>

<!-- 配置声明式事务，确认具体哪些方法有事务 -->
<tx:advice id="txAdvice" transaction-manager="txManager">
    <tx:attributes>
        <!-- 哪些方法需要有事务控制 -->
         <!-- 名字里面可以是用 *，表示方法以ins 开头事务管理 -->
        <tx:method name="ins*" />
        <tx:method name="del*" />
        <tx:method name="upd*" />
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>

<aop:config>
<!--切点范围设置大一些，仅仅设置哪些有切点即可，和事务控制没有关系 -->
<aop:pointcut expression="execution(* com.bjsxt.service.impl.*.*(..))"
id="mypoint" />
<aop:advisor advice-ref="txAdvice" pointcut-ref="mypoint" />
</aop:config>
```

 使用：
```com_gjxaiou_pojo_People_java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@toString
public class People{
    int id;
    String username;
    String password;
}
```

```com_gjxaiou_service_UsersService_java
public interface UsersService{
    int insert(Users user);
}
```

```com_gjxaiou_service_UsersServiceImpl_java
public class UsersServiceImpl implements UsersService{
    @override
    // 声明式事务是基于通知的，要将下面的方法实现事务，要配置其切点
    public int insert(Users users){
    return 0;
    }
}
```




## 六、声明式事务中属性解释

- `name=””` 表示哪些方法需要有事务控制
支持*通配符

- `readonly=”boolean”` 表示是否是只读事务.
  - 如果为 true，告诉数据库此事务为只读事务.是一种数据库优化，会对性能有一定提升，所以只要是查询的方法，建议使用此属性.
  - 如果为 false(默认值)，表示事务需要提交的事务.建议新增，删除，修改情况下使用.

- propagation 控制事务传播行为.
  - 当一个具有事务控制的方法被另一个有事务控制的方法调用后，需要如何管理事务(新建事务?在事务中执行?把事务挂起?报异常?)  
  - REQUIRED (默认值): （针对被调用的）如果当前有事务，就在事务中执行，如果当前没有事务，新建一个事务.

  -  SUPPORTS:如果当前有事务就在事务中执行，如果当前没有事务，就在非事务状态下执行.

  - MANDATORY:必须在事务内部执行，如果当前有事务，就在事务中执行，如果没有事务，报错.

  - REQUIRES_NEW:必须在事务中执行，如果当前没有事务，新建事务，如果当前有事务，把当前事务挂起（执行自己的事务）.
  - NOT_SUPPORTED:必须在非事务下执行，如果当前没有事务，正 常执行，如果当前有事务，把当前事务挂起.

  - NEVER:必须在非事务状态下执行，如果当前没有事务，正常执行，如果当前有事务，报错.

  - NESTED:必须在事务状态下执行.如果没有事务，新建事务，如果当前有事务，创建一个嵌套事务.


- `isolation=””` 事务隔离级别（4.5开始是其值的可选项）
在多线程或并发访问下如何保证访问到的数据具有完整性的.
- DEFAULT: 默认值，由底层数据库自动判断应该使用什么隔离级别

- READ_UNCOMMITTED: 可以读取未提交数据，可能出现脏读，不重复读，幻读.
 效率最高.

-  READ_COMMITTED:只能读取其他事务已提交数据.可以防止脏读，可能出现不可重复读和幻读.

- REPEATABLE_READ: 读取的数据会被添加锁，防止其他事务修改此数据，可以防止不可重复读.脏读，可能出现幻读.

- SERIALIZABLE: 排队操作，对整个表添加锁.一个事务在操作数据时，另一个事务等待事务操作完成后才能操作这个表.
  -  最安全的
  - 效率最低的.

- rollback-for=”异常类型全限定路径”
表示当出现什么异常时需要进行回滚

  - 建议:给定该属性值.
  - 手动抛异常一定要给该属性值.

- no-rollback-for=””
 当出现什么异常时不滚回事务.


**补充知识**

- 脏读:
一个事务(A)读取到另一个事务(B)中未提交的数据，另一 个事务中数据可能进行了改变，此时A 事务读取的数据可能和数据库中数据是不一致的，此时认为数据是脏数据，读取脏数据过程叫做脏读.

- 幻读:
事务 A 按照特定条件查询出结果，事务 B 新增了一条符合条件的数据.事务 A 中查询的数据和数据库中的数据不一致的，事务 A 好像出现了幻觉，这种情况称为幻读.
  - 主要针对的操作是新增或删除
  - 两次事务的结果.

- 不可重复读:
当事务 A 第一次读取事务后，事务 B 对事务 A 读取的数据进行修改，事务 A 中再次读取的数据和之前读取的数据不一致，这个过程称为不可重复读.
  - 主要针对的是某行数据.(或行中某列)
  - 主要针对的操作是修改操作.
  -  两次读取在同一个事务内

## 七、Spring 中常用注解

- @Component 创建类对象，相当于配置<bean/>

- @Service 与@Component 功能相同.
 但是@service写在 ServiceImpl 类上.

- @Repository 与@Component 功能相同.
 写在数据访问层类上.

- @Controller 与@Component 功能相同.
 写在控制器类上.

- @Resource(不需要写对象的 get/set)
是 java 中自带的注解
  - 默认按照 byName 注入，如果没有名称对象，按照 byType  注入
    -  建议把对象名称和 spring 容器中对象名相同

- @Autowired(不需要写对象的 get/set)
是 spring 的注解
  - 默认按照 byType  注入.

- @Value() 获取 properties 文件中内容

* @Pointcut() 定义切点

* @Aspect() 定义切面类

* @Before() 前置通知

* @After 后置通知

* @AfterReturning 后置通知，必须切点正确执行

* @AfterThrowing 异常通知

* @Arround 环绕通知

## 八、Ajax
使用Ajax绝不会有跳转语句，都是写的输出语句，即响应回来的结果是什么

- 标准请求响应时浏览器的动作(同步操作)
  - 浏览器请求什么资源，跟随显示什么资源

* ajax:异步请求.【有请求的时候，浏览器开启一个子线程进行数据请求，获取到数据之后，根据脚本对主线程中东西进行修改，主线程在子线程进行请求的过程中是不发生改变的；】

  - 局部刷新，通过异步请求，请求到服务器资源数据后，通过脚本修改页面中部分内容.

* ajax 由 javascript 推出的.
  *  由 jquery 对 js 中 ajax 代码进行的封装，达到使用方便的效果.

###   jquery 中 ajax 分类
 
*  第一层 $.ajax({ 属性名:值，属性名:值})
  * 是 jquery 中功能最全的.代码写起来相对最麻烦的.

示例代码
url:  请求服务器地址
data:请求参数
dataType:服务器返回数据类型
error 请求出错执行的功能
success 请求成功执行的功能，表达式function(data)中的
data是服务器返回的数据
type:请求方式
```index_js
// 这里配置 script type 等等
$(function(){
    $("a").click(function(){
        $.ajax({
            url:'demo',
            data:{"name":"张三"},
            dataType:'html',
            error:function(){
                alert("请求出错.")
            },
            success:function(data){
                alert("请求成功"+data)
            },
            type:'POST'
        });
        return false;
    })
});
```

- 第二层(简化$.ajax)
  相当于上面的代码中 设置好了type，只有成功返回等
  - $.get(url,data,success,dataType))
  - $.post(url,data,success,dataType)

- 第三层(简化$.get())
  - $.getJSON(url,data,success). 相 当 于 设  置  $.get 中dataType=”json”
  -  $.getScript(url,data,success) 相 当 于 设  置  $.get 中dataType=”script”

- 如果服务器返回数据是从表中取出.为了方便客户端操作返回的数据，服务器端返回的数据设置成 json
  - 客户端把 json 当作对象或数组操作.

- json：是一种数据格式.
  - JsonObject ： json 对象，理解成 java  中对象
 格式为： `{“key”:value,”key”:value}`
  - JsonArray:json 数组
格式为： `[{“key”:value,”key”:value},{}]`
